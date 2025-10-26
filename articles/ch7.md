#### indexerを作成する
つぎに、indexerを作成してデータをAzure AI Searchに取り込みます。

まず、indexerが参照するデータソースを作成します。
- storageの接続先をazure portal上から取得し、connection_stringに設定します

```python
# indexerクライアントを作成
indexer_client = SearchIndexerClient(endpoint=SEARCH_ENDPOINT, credential=credential)

container = SearchIndexerDataContainer(
    name=BLOB_CONTAINER,
    query=BLOB_PREFIX,  # "docs" 以下だけ取り込む。コンテナ全体なら None
)

# データソースを作成
data_source = SearchIndexerDataSourceConnection(
    name=DATA_SOURCE_NAME,
    type=SearchIndexerDataSourceType.AZURE_BLOB,
    connection_string=AZURE_STORAGE_CONNECTION_STRING,
    container=container,
    description="JQaRA JSONs in Blob Storage",
)

# indexerにデータソースを登録
indexer_client.create_or_update_data_source_connection(data_source)
```

つぎに、skillsetを設定します。
今回は、Azure OpenAIの埋め込みモデルを使ってテキストのベクトル化を行うskillsetを作成します。
- TODO: contextの話

```python
embedding_skill = AzureOpenAIEmbeddingSkill(
    description="Skill to generate embeddings via Azure OpenAI",
    context="/document",
    resource_url=AOAI_ENDPOINT,
    deployment_name=AOAI_EMBEDDING_DEPLOYMENT,
    model_name=AOAI_EMBEDDING_MODEL,
    dimensions=DIM,
    inputs=[
        InputFieldMappingEntry(name="text", source="/document/content"),
    ],
    outputs=[
        OutputFieldMappingEntry(name="embedding", target_name="text_vector")
    ],
)

# skillsetを作成
skillset = SearchIndexerSkillset(
    name=SKILLSET_NAME,
    skills=[embedding_skill],
    description="JQaRA index-time embedding skillset",
)

# skillsetを登録
indexer_client.create_or_update_skillset(skillset)
```


- インデクサーを作成して実行する

```python
indexer = SearchIndexer(
    name=INDEXER_NAME,
    data_source_name=DATA_SOURCE_NAME,
    target_index_name=INDEX_NAME,
    skillset_name=SKILLSET_NAME, 
    # ドキュメント -> インデックス のフィールドマッピング
    field_mappings=[
        FieldMapping(source_field_name="id", target_field_name="id"),
        FieldMapping(source_field_name="raw_id", target_field_name="raw_id"),
        FieldMapping(source_field_name="title", target_field_name="title"),
        FieldMapping(source_field_name="text", target_field_name="text"),
    ],
    # スキル出力 -> インデックス のフィールドマッピング
    output_field_mappings=[
        FieldMapping(
            source_field_name="/document/text_vector",
            target_field_name="text_vector",
        ),
    ],    
    parameters=IndexingParameters(configuration={"parsingMode": "jsonLines"})
)
# インデクサーを登録
indexer_client.create_or_update_indexer(indexer)
```

これでindexerの作成が完了しました。

indexerを実行してデータが取り込まれるかを確認します。
indexerの実行はportal上からも可能ですが、今回はコードで実行します。

```python
indexer_client.run_indexer(INDEXER_NAME)
```

ここまででデータの取り込みが完了し、検索が可能になりました。
