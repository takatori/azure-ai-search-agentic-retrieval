### Agentic Retrieval検索を行う
ここまでで、Agentic Retrieval検索を行う準備が整いました。
以下のコードで、実際にAgentic Retrieval検索を実行します。
まずは、KnowledgeAgentが参照するKnowledge Sourceを作成します。

```python
ks = SearchIndexKnowledgeSource(
    name=KNOWLEDGE_SOURCE_NAME,
    description="Knowledge source for Earth at night data",
    search_index_parameters=SearchIndexKnowledgeSourceParameters(
        search_index_name=INDEX_NAME,
        source_data_select="id,raw_id,title,text",
    ),
)
index_client.create_or_update_knowledge_source(
    knowledge_source=ks, api_version=SEARCH_API_VERSION
)
```

つぎに、Knowledge Agentを作成します。

```python
aoai_params = AzureOpenAIVectorizerParameters(
    resource_url=AOAI_ENDPOINT,
    deployment_name=AOAI_GPT_DEPLOYMENT,
    model_name=AOAI_GPT_MODEL,
)

output_cfg = KnowledgeAgentOutputConfiguration(
    modality=KnowledgeAgentOutputConfigurationModality.ANSWER_SYNTHESIS,
    include_activity=True,
)

agent = KnowledgeAgent(
    name=KNOWLEDGE_AGENT_NAME,
    models=[KnowledgeAgentAzureOpenAIModel(azure_open_ai_parameters=aoai_params)],
    knowledge_sources=[
        KnowledgeSourceReference(
            name=KNOWLEDGE_SOURCE_NAME,
            reranker_threshold=2.5,
        )
    ],
    output_configuration=output_cfg,
)

index_client.create_or_update_agent(agent, api_version=SEARCH_API_VERSION)

```

実際にAgentic Retrieval検索を実行するには以下のように実行します。

```python
agent_client = KnowledgeAgentRetrievalClient(
    endpoint=SEARCH_ENDPOINT, 
    agent_name=KNOWLEDGE_AGENT_NAME, 
    credential=credential
)

query = """
    目の愛護デーは何月何日でしょう?また、耳の日は何月何日でしょう?その理由も教えてください。
    When is the International Day of Nonviolence?
    さらに、それぞれの日に何をすれば良いか教えてください
"""
messages.append({"role": "user", "content": query})

req = KnowledgeAgentRetrievalRequest(
    messages=[
        KnowledgeAgentMessage(
            role=m["role"],
            content=[KnowledgeAgentMessageTextContent(text=m["content"])],
        )
        for m in messages
        if m["role"] != "system"
    ],
    knowledge_source_params=[
        SearchIndexKnowledgeSourceParams(
            knowledge_source_name=KNOWLEDGE_SOURCE_NAME, kind="searchIndex"
        )
    ],
)

result = agent_client.retrieve(retrieval_request=req, api_version=SEARCH_API_VERSION)
```


```python
print("[Response]")
print(textwrap.fill(result.response[0].content[0].text, width=120))

print("[Activity]")
print(json.dumps([a.as_dict() for a in result.activity], indent=2, ensure_ascii=False))

print("[Results]")
print(
    json.dumps([r.as_dict() for r in result.references], indent=2, ensure_ascii=False)
)
```

実行した結果が以下です。`[Response]`が最終的な応答、`[Activity]`がエージェントのアクティビティログ、`[Results]`が検索インデックスから取得した

```text
== Response ==
目の愛護デーは毎年10月10日です。[ref_id:9][ref_id:0] 理由は、数字の「10」を横に倒すと眉と目に見えることからと言われています。[ref_id:0][ref_id:3]
当日は目の健康を維持するための各種啓発行事が行われ、厚生労働省主催の活動や眼科医による早期受診の呼びかけなどが行われますので、定期検診や目の異常があれば早めに受診するなどの参加が推奨されます。[ref_id:0][ref_id:3]
耳の日は毎年3月3日です。[ref_id:2] 理由は「3」の字が耳の形に似ていることと、「み(3)み(3)」の語呂合わせによるものです。[ref_id:8][ref_id:2]
当日は耳や耳の病気への関心を高め、健康な耳への感謝や良い音楽で耳を楽しませること、耳の不自由な人々への社会的関心を高めるといった啓発や配慮を行うのが適切です。[ref_id:8]
国際非暴力デーは毎年10月2日です。[ref_id:4][ref_id:15] これはマハトマ・ガンディーの誕生日に因んで設けられた日です。[ref_id:15][ref_id:17]
当日はガンディーの非暴力の理念を称え、非暴力や対話による紛争解決を促進する啓発・行事などに参加したり、非暴力を訴える活動を行うことが適切です。[ref_id:4][ref_id:15]

== Activity == 
[
  {
    "id": 0,
    "type": "modelQueryPlanning",
    "elapsed_ms": 11455,
    "input_tokens": 2075,
    "output_tokens": 1189
  },
  {
    "id": 1,
    "type": "searchIndex",
    "elapsed_ms": 1044,
    "knowledge_source_name": "jqara-knowledge-source",
    "query_time": "2025-10-26T08:03:18.165Z",
    "count": 5,
    "search_index_arguments": {
      "search": "目の愛護デー 日付 理由 当日にできること"
    }
  },
  {
    "id": 2,
    "type": "searchIndex",
    "elapsed_ms": 1073,
    "knowledge_source_name": "jqara-knowledge-source",
    "query_time": "2025-10-26T08:03:19.239Z",
    "count": 4,
    "search_index_arguments": {
      "search": "耳の日 日付 理由 当日にできること"
    }
  },
  {
    "id": 3,
    "type": "searchIndex",
    "elapsed_ms": 937,
    "knowledge_source_name": "jqara-knowledge-source",
    "query_time": "2025-10-26T08:03:20.176Z",
    "count": 9,
    "search_index_arguments": {
      "search": "International Day of Nonviolence date reason what to do on that day"
    }
  },
  {
    "id": 4,
    "type": "semanticReranker",
    "input_tokens": 21534
  },
  {
    "id": 5,
    "type": "modelAnswerSynthesis",
    "elapsed_ms": 14974,
    "input_tokens": 5243,
    "output_tokens": 1603
  }
]

== Results ==
[
  {
    "type": "searchIndex",
    "id": "0",
    "activity_source": 1,
    "reranker_score": 2.8578153,
    "doc_key": "QA20QBIK-1731_5495013"
  },
  {
    "type": "searchIndex",
    "id": "3",
    "activity_source": 1,
    "reranker_score": 2.4909399,
    "doc_key": "QA20CAPR-1851_184480"
  },
  ...
]
```