### 検索を行う
ここまでで検索を実行する準備はすでに整いました。
Agentic検索を行う前に、通常の検索を行う方法を説明します。

- 検索クエリをつくる

```python
search_client = SearchClient(
    endpoint=SEARCH_ENDPOINT,
    index_name=INDEX_NAME,
    credential=credential
)
```

- 全文検索
https://learn.microsoft.com/ja-jp/azure/search/search-query-create?tabs=portal-text-query

```python
query = "摂氏ではマイナス273.15度にあたる、全ての原子の振動が停止する最も低い温度を何というでしょう?"
results = search_client.search(
        search_text=query,
        query_type=QueryType.SIMPLE,
        top=1,
        select=["raw_id", "title", "text"],
    )
```

- ベクトル検索
https://learn.microsoft.com/ja-jp/azure/search/vector-search-how-to-query?tabs=query-2025-09-01%2Cbuiltin-portal
```python
vq = VectorizableTextQuery(
    text=query,
    k_nearest_neighbors=topk,
    fields="text_vector",
)

results =  await async_search_client.search(
    search_text=None, 
    vector_queries=[vq], 
    select=["raw_id"]
)
```

- ハイブリッド検索
https://learn.microsoft.com/ja-jp/azure/search/hybrid-search-how-to-query?tabs=portal

```python
```

- semantic renrakを有効化
https://learn.microsoft.com/ja-jp/azure/search/semantic-how-to-query-request?tabs=portal-query
https://learn.microsoft.com/ja-jp/azure/search/semantic-how-to-query-rewrite
```python

```

