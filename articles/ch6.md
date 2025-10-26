#### インデックスを作成する
環境が構築できたら、検索エンジンのインデックスの作成に映ります。

まずインデックスのフィールドを以下のように定義します。

```python
fields = [
    SearchField(name="id", type="Edm.String", key=True, filterable=True, sortable=True),
    SearchField(name="raw_id", type="Edm.String", filterable=True, sortable=True),
    SearchField(name="title", type="Edm.String", searchable=True, analyzer_name=LexicalAnalyzerName.JA_LUCENE),
    SearchField(name="text", type="Edm.String", searchable=True, analyzer_name=LexicalAnalyzerName.JA_LUCENE),
    SearchField(
        name="text_vector",
        type="Collection(Edm.Single)",
        searchable=True,
        vector_search_dimensions=DIM,
        vector_search_profile_name="aoai-hnsw",
    ),
]
```

- ここでidとraw_idを分けているのは、JQaRAのidは`#`を含むためです。Azure AI Searchのキーには特殊文字が使えないため、raw_idとして保存しています。idはraw_idを変換して保存しています。
- titleとtextフィールドには日本語用のアナライザーを設定しています
- text_vectorはtextフィールドの埋め込みを保存するためのフィールドです。DIMは埋め込みの次元数を指定します。vector_search_profile_nameには後ほど定義するベクトル検索のプロファイル名を指定します。


次にベクトル検索の設定を行います。

```python
vector_search = VectorSearch(
    algorithms=[
        HnswAlgorithmConfiguration(name="hnsw")
    ],
    # クエリ時の自動ベクトル化
    vectorizers=[
        AzureOpenAIVectorizer(
            vectorizer_name="aoai-vectorizer",
            parameters=AzureOpenAIVectorizerParameters(
                resource_url=AOAI_ENDPOINT,
                deployment_name=AOAI_EMBEDDING_DEPLOYMENT,
                model_name=AOAI_EMBEDDING_MODEL,
            ),
        ),
    ],
    profiles=[
        VectorSearchProfile(
            name="aoai-hnsw",
            algorithm_configuration_name="hnsw",
            vectorizer_name="aoai-vectorizer",
        ),
    ]
)
```

- algorithmsでは`exhaustive KNN`か`HNSW`を選択できます。
- vectorizerはテキストや画像を埋め込みベクトルに自動で変換するコンポーネントです。
    - Azure OpenAI または Azure AI Vision にデプロイされた埋め込みモデルを使用できます        
    - https://learn.microsoft.com/ja-jp/azure/search/vector-search-how-to-configure-vectorizer
    - クエリの実行時にテキスト (または画像) クエリ入力の埋め込みを生成するために使用される

- profilesでは、アルゴリズムとcompressionおよびvectorizerの組み合わせを定義します



さらにセマンティックリランカーの設定を行います。

```python
semantic_search = SemanticSearch(
    default_configuration_name="semantic_config",
    configurations=[
        SemanticConfiguration(
            name="semantic_config",
            prioritized_fields=SemanticPrioritizedFields(
                title_field=SemanticField(field_name="title"),
                content_fields=[SemanticField(field_name="text")]
            ),
        )
    ],
)
```



最後に上記の設定でインデックスの作成を実行します。

```python
index_client = SearchIndexClient(endpoint=SEARCH_ENDPOINT, credential=credential)

index = SearchIndex(
    name=INDEX_NAME,
    fields=fields,
    vector_search=vector_search,
    semantic_search=semantic_search,
)

index_client.create_or_update_index(index)
```