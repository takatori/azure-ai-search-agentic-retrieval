## 実践

ここからは、Azure AI Searchを用いて、実際にAgentic検索を実装し、評価する手順を説明します。
完全なサンプルコードのは以下のGitHubリポジトリで公開しています。
https://github.com/takatori/azure-ai-search-agentic-rag

### データの準備
はじめに、検索対象となるデータを準備します。
今回はテストデータとして、JQaRAデータセットを使用します。
https://github.com/hotchpotch/JQaRA
JQaRAは、日本語の質問応答データセットであり、様々なドメインにわたる質問とその回答が含まれています。
今回は評価のために使用するので、`test` splitのみを使用します。

まず、JQaRAデータセットをダウンロードし、JSON形式で保存します。
つぎに、Azure Blob Storageにデータをアップロードします。


### 環境準備
つぎに、Azure AI Searchを利用するための環境を準備します。
https://learn.microsoft.com/ja-jp/azure/search/search-get-started-agentic-retrieval?tabs=search-perms%2Csearch-endpoint&pivots=programming-language-python

- Search Serviceを作成する
  - Identityを有効化
  - Roleを有効化
  - 自分に権限を付与する
  - Foundryに権限を付与する
