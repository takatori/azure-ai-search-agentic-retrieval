# Azure AI Search Agentic Retrieval

このリポジトリは「LayerX TechBook 1」の「Azure AI Search で実践 Agentic Retrieval」で使用したサンプル（ノートブック中心）を公開するものです。Azure AI Search と LLM を組み合わせ、エージェントが自律的に検索・判断しながら回答を生成する「Agentic Retrieval」の最小実装を示します。

関連リンク（書籍の紹介）: https://techbookfest.org/product/5zbWgnL90mCvjt7MmnwxW2

---

## 構成

- `azure-ai-search-jqara.ipynb`: 実行用ノートブック。検索インデックスの準備、ドキュメント投入、検索・生成、簡易評価（`ranx`）までを順に実行します。
- `pyproject.toml`: 依存関係（Python 3.12 以上）を管理します。
- `uv.lock`: `uv` 用ロックファイル。再現性の高い環境構築に使用します。

---

## 必要要件

- Python 3.12 以上
- Azure サブスクリプションおよび Azure AI Search（Cognitive Search）インスタンス
  - サービスのエンドポイントと管理キー（Admin Key）
- LLM 推論先のいずれか
  - Azure OpenAI（推奨）: エンドポイント、API キー、デプロイ名
  - もしくは OpenAI API: API キー
- 推奨: パッケージマネージャー `uv`（https://docs.astral.sh/uv/）

---

## セットアップ

### 1) リポジトリの取得

```bash
git clone <このリポジトリのURL>
cd azure-ai-search-agentic-retrieval
```

### 2) 仮想環境の作成と依存関係のインストール（uv）

```bash
# macOS/Linux
uv venv
source .venv/bin/activate

# 依存関係を同期
uv sync
```
---

## 環境変数の設定（.env 推奨）

`python-dotenv` を利用して、プロジェクト直下に `.env` を用意します。必要な値はご利用の構成に合わせて調整してください。

```env
SEARCH_ENDPOINT=https://<your-search-service>.search.windows.net
AOAI_ENDPOINT=https://<your-aoai>.openai.azure.com/
AZURE_STORAGE_CONNECTION_STRING=<your-connection-string>
AZURE_AI_FOUNDRY_PROJECT_ENDPOINT=<your-foundry-project-endpoint>
```
## 使用データセット (Datasets)

本プロジェクトの評価には、以下のデータセットを使用しました。

* **JQaRA**: [hotchpotch/JQaRA](https://huggingface.co/datasets/hotchpotch/JQaRA)
    * ライセンス: CC-BY-SA 4.0
    * 本データセットは「AI王 公式配布データセット(JAQKET)」および「Wikipedia」を元に構築されています。

## 参考リンク

- Azure AI Search 公式ドキュメント: https://learn.microsoft.com/azure/search/
- ranx（評価）: https://github.com/AmenRa/ranx
- TechBook 紹介ページ: https://techbookfest.org/product/5zbWgnL90mCvjt7MmnwxW2

