
## ドキュメント
- [クイックスタート: Azure AI 検索でエージェント取得を使用する](https://learn.microsoft.com/ja-jp/azure/search/search-get-started-agentic-retrieval?tabs=search-perms,foundry-endpoint&pivots=programming-language-python)

- [Azure AI Search でナレッジ エージェントを作成する](https://learn.microsoft.com/ja-jp/azure/search/search-agentic-retrieval-how-to-create)

- トークンの取得方法

```
az account get-access-token --scope https://search.azure.com/.default --query accessToken --output tsv
```


## デプロイの種類

openaiのmodelはglobal standardでのデプロイしかできない.
glbal standardでのデプロイは、勝手にai foundryのprojectが作成される。

https://zenn.dev/incudata/articles/azure-japane-openai-deploy


### 
https://github.com/Azure-Samples/azure-ai-search-multimodal-sample/blob/main/src/backend/data_injestion/indexer_img_verbalize_strategy.py

## 評価
