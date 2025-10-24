# Azure AI Searchで実践AgenticRAG

## Agentic RAGとは？

静的なデータやカットオフといったLLMの限界の解決策として、RAGがある。
RAGは検索して外部のデータをLLMに与えることで上記の課題を克服する。
しかしながら、単純なRAGでは解決できない問題がある。


具体的には、現実世界の複雑なクエリに対処できない
静的なワークフロー、多段階の推論や複雑なタスク管理に必要な適応性に欠ける
https://arxiv.org/html/2501.09136v3


Agentic RAGの特徴
- リフレクション
- 動的なプランニング・医師けってい
- 適応的な検索戦略

## なぜAzure AI Searchを使うのか
- 課題
    - インフラ構築が大変
    - 運用も大変
    - 検索専門のチームがなく、他の開発の間で実装しないといけない
    - エンジンの運用だけで手一杯
    - ビジネス的に価値があるのか不明
    - コストもかかる
    - いらないものは作らないほうが良い
    - 特定のケース
    - ローカルで処理できないほどのデータ量で実験したい
    - 短い開発期間で新しいRAGアプリを素早く立ち上げる
  
    - 検索だけでも色々作らないといけない
        - indexing pipline
            - embedding
            - analyze
            - feed
        - engine
            - model
            - full-tesxt search
            - vector search
            - hybrid search
            - reranking
        - searcher
            - query rewrite
            - embedding

- Azure AI Searchをつかメリット
    - ベースラインとして機能する
    - 設定だけでさくっとAgentic検索ができる
    - Agentic RAGも簡単にできる
    Cognitive SearchやAzure OpenAI Serviceと組み合わせることで、構造化・非構造化データの統合検索、RAGパイプラインのクラウド実装が容易にできる。


## 実践: Agentic RAGのアーキテクチャ

TODO: 図

- Indexer
   - ソースデータと検索絵インデックスのフィールド間のマッピングを行う
   - クローラー
    - Azure内外のデータストアをクロール可能
   - プルモデル
   - サポートされるデータソースを対象とする
   - スキルセットにより追加処理を統合的に実施
   - オンデマンドで実行すること重、定期的にスケジューリング実行もできる
   - 更新頻度が高い場合はindexerではなくプッシュモデルを実装する必要がある
   - ターゲットインデックスとデータソースの組み合わせごとに一つのインデクサーを作成する
   - 複数のインでクサーが同じインデックスに書き込み可能
   - 複数のインでクサーに同じデータソースを再利用可能
     - ただし、インデクサーが一回に利用できるデータソースは1つだけ
     - 書き込めるインデックスも一つだけ
   - インデックス作成のステージ
     - 最初の実行時にインデックスが空の場合、すべてのデータがインデクサーによって読み取られる
     - その後の実行では、通常、変更されたデータのみがインデクサーによって検出され取得される
     - BLOBデータの場合自動で検出、Azure SQLやAzure Cosmos DBなどの他のデータソースでは変更の検出を有効化する必要がある
     - stage1: Documet cracking
        - ファイルを開いてコンテンツを抽出する
        - データソースに応じて、インデックス付が可能なコンテンツを抽出するために、インでクサーによってさまざまな操作が施行される
          - PDFなどの画像が埋め込まれたファイルである場合、出来kスト、画像、メタデータを抽出する
          - SQLレコードやCosmosDBの場合、各レコードの各フィールドからバイナリ以外のコンテンツが抽出される          
     - stage2: Field mapping
        - ソースフィールドから宛先フィールドをマッピングする
        - フィールドをマップする方法をインでクサーにいんでくさー定義で指示
        - ソースフィールドの値が変換されずにそのまま、送信先フィールドに送信される
     - stage3: Skillset execution
        - 組み込みまたはカスタムAI処理を呼び出すステップ
        - 省略可能    
     - stage4: Outpput Field Mapping
        - スキルセットを含める場合の、出力フィールドのマッピング
        - スキルセットの出力はエンリッチされたドキュメントとよばれるツリー構造として内部的に示される
        - 出力フィールドマッピングでは、このツリーの部分を選択してインデックス内のフィールドにマッピングする
        - 出力フィールドマッピングとフィールドマッピングは、異なるソースからマッピングする
        - インデックスに存在する必要がある変換されたすべてのコンテンツには出力フィールドマッピングが必要
           - フィールドマッピングは省略可能
        
     - stage5: push index
        - 

- データソース

- SkillSet
    - 
- Index
    - ドキュメントの構造はインデックススキーマできまる
    - 
- Knowledge Source
    - https://learn.microsoft.com/ja-jp/javascript/api/@azure/search-documents/knowledgesourcereference?view=azure-node-preview
    - https://learn.microsoft.com/ja-jp/azure/search/agentic-knowledge-source-overview
    - エージェント取得用の追加のプロパティで検索インデックスをラップしたもの
      - 複数のナレッジ ソースがある場合は、プロパティを設定して、クエリ計画を特定のナレッジ ソースに偏らせることができる
        - alwaysQuerySource: クエリ計画に常にそのナレッジソースが含まれるようになる
        - retrievalInstructions: ナレッジソースを含めるか除外するかのガイダンスを提供する
            - クエリ計画に使用されるLLMにユーザ定義プロンプトとして送信される
            - 例えば、"求人応募に関する質問の場合のみ求人インデックスを使用する"

    - 以下が可能
      - 各ナレッジソースは1つのインデックスのみを指す。エージェントによる検索の条件を満たしている必要がある
      - ナレッジエージェント内で一つ以上のナレッジソースを参照する
        - エージェント検索パイプラインでは、1つの要求Dえ複数のナレッジソースに対してクエリを実行可能
        - サブクエリはナレッジソースごとに生成される
      - ナレッジソース定義を使用して、


- Knowledge Agent
    - https://learn.microsoft.com/ja-jp/javascript/api/@azure/search-documents/knowledgeagent?view=azure-node-preview
    - https://learn.microsoft.com/ja-jp/azure/search/agentic-retrieval-overview
    - https://learn.microsoft.com/ja-jp/azure/search/agentic-retrieval-how-to-create-knowledge-base?tabs=rbac%2Cpython-get-agents%2Cpython-create-agent%2Cpython-query-agent%2Cpython-delete-agent
    - 
    - https://learn.microsoft.com/ja-jp/javascript/api/@azure/search-documents/searchindexknowledgesourceparams?view=azure-node-preview
    - 履歴は保持しない、履歴の管理は自前でやる

- Agentic retrieval
  - LLMを用いて、複雑なクエリをより小さい、サブクエリに分割し、インデックス付きのコンテンツのカバレッジを向上する
    - サブクエリには、追加のコンテキストのチャット履歴を含めることができる　
  - サブクエリを並列で実行。各サブクエリは、最も関連性の高い一致を昇格させるためにセマンティックリランキングされる
  - 最適な結果をLLMが独自のコンテンツで回答を生成するために使用できる統一された応答にマージする
  - アーキテクチャとワークフロー
    - How it works
        - 1.ワークフローの開始
            - アプrケーションは、クエリと会話履歴を含んだリクエストでナレッジエージェントを呼びだす
        - 2.クエリの計画
            - ナレッジエージェントは、クエリと会話履歴をLLMに送信し、コンテキストを分析、複雑な質問を焦点に絞ったサブクエリに分割する
            - この手順はカスタマイズできない
        - 3. クエリの実行
            - ナレッジエージェントは、サブクエリをナレッジソースに送信する
            - すべてのサブクエリは同時に実行され、キーワード、ベクター、ハイブリッド検索を使用できる
            - 各サブクエリには、セマンティックリランキングが行われ、最も関連性の高い一致が検出される
            - 引用のために参照元が抽出され保持される
        - 4. 結果の合成
            - 「システムはすべての結果を統合し、3 つの部分からなる統一された応答を生成します。
            - それは、① 結合されたコンテンツ、② 情報源の参照、③ 実行の詳細 の3要素で構成されています。

- semantic ranker
  - 
    

## 実際の構築ステップ
### indexing
- blob storageを用意する
- データをアップロードする
- Azure AI Searchを立てる
- データソースを作成する
- インデックスを作成する
- インデクサーを作成して実行する

### 検索
- 検索クエリをつくる
- Knoledge Sourceをつくる
- Knowledge Agentを作る

### Agent
- TODO


## まとめ