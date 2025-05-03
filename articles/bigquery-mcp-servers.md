---
title: "BigQueryのMCPサーバー実装の比較調査"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["BigQuery", "MCP", "AI", "データベース", "LLM"]
published: false
---

# BigQuery MCPサーバーの調査結果

BigQueryのMCPサーバーについて調査した結果、主に2つの実装が見つかりました。どちらもGoogle BigQueryデータベースにLLM（大規模言語モデル）からアクセスするためのMCPサーバーです。

## 1. ergut/mcp-bigquery-server

**リポジトリ**: https://github.com/ergut/mcp-bigquery-server

**特徴**:
- Node.jsで実装
- BigQueryデータセットへの安全な読み取り専用アクセスを提供
- 自然言語でSQLクエリを実行可能（英語でデータについて質問するだけでSQLクエリを実行）
- テーブルとマテリアライズドビューの両方にアクセス可能
- データセットスキーマの探索機能
- 安全な制限内でのデータ分析（デフォルトで1GBクエリ制限）

**インストール方法**:
```
npx -y @ergut/mcp-bigquery-server --project-id <your-project-id> --location <location> [--key-file <path-to-key-file>]
```

**Claude Desktop設定例**:
```json
{
  "mcpServers": {
    "bigquery": {
      "command": "npx",
      "args": [
        "-y",
        "@ergut/mcp-bigquery-server",
        "--project-id",
        "your-project-id",
        "--location",
        "us-central1"
      ]
    }
  }
}
```

## 2. LucasHild/mcp-server-bigquery

**リポジトリ**: https://github.com/LucasHild/mcp-server-bigquery

**特徴**:
- Pythonで実装
- 以下の3つのツールを提供:
  - execute-query: BigQueryのSQLクエリを実行
  - list-tables: BigQueryデータベース内のすべてのテーブルをリスト
  - describe-table: 特定のテーブルのスキーマを説明

**設定オプション**:
- --project (必須): GCPプロジェクトID
- --location (必須): GCPロケーション（例：europe-west9）
- --dataset (オプション): 特定のBigQueryデータセットのみを考慮
- --key-file (オプション): BigQuery用のサービスアカウントキーファイルへのパス

## 使用方法

どちらのMCPサーバーも、Claude Desktopなどのサポートされているインターフェースで使用できます。設定後は、自然言語でBigQueryデータに対して質問したり、クエリを実行したりすることができます。

これらのMCPサーバーを使用することで、LLMがBigQueryデータベースに直接アクセスし、データの分析や探索を行うことが可能になります。

どちらを選ぶかは、使用言語の好み（Node.js vs Python）や必要な機能によって異なります。ergutの実装は自然言語からSQLへの変換に重点を置いているようですが、LucasHildの実装はより直接的なSQLクエリ実行に焦点を当てています。
