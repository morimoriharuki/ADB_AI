# Oracle Autonomous AI Database - DATABASE_INSPECT 検証

## 概要

Oracle Autonomous AI Database (ADB) の `DATABASE_INSPECT` PL/SQL パッケージを検証するプロジェクト。
自然言語でデータベースオブジェクトを探索・分析できるツール群を提供する。

参照元: https://github.com/oracle-devrel/oracle-autonomous-database-samples/tree/main/autonomous-ai-agents/database_inspect

## 構成

Select AI Agent は使わず、`DATABASE_INSPECT` パッケージのみをインストールし、
PL/SQL 関数を直接呼び出す構成。

## 環境

- リージョン: 大阪 (ap-osaka-1)
- ADB バージョン: 26ai（推奨）
- 接続方法: VS Code の Oracle SQL Developer Extension（接続名: `aiidb-mcp-server`）

## セットアップ手順

SQL ファイルはすべて VS Code で開き、Oracle SQL Developer Extension の `aiidb-mcp-server` 接続で実行する。

### 1. 前提条件確認
`sql/00_prerequisites.sql` を実行し、環境が要件を満たすか確認する。

### 2. AI プロファイル作成
`sql/01_create_ai_profile.sql` を参考に、エンベディングモデル付きの AI プロファイルを作成する。
（ベクトル検索機能を使う場合に必要）

### 3. パッケージインストール
`sql/02_install_tool.sql` を ADMIN ユーザーで実行する。
実行時に `SCHEMA_NAME` の入力を求められるので、対象スキーマ名を指定する。

### 4. 動作確認
`sql/03_test_queries.sql` のクエリを実行して、パッケージが正常に動作するか確認する。

### 5. MCP 基本ツール登録
`sql/04_register_mcp_tools.sql` を ADMIN ユーザーで SQLcl から実行する。
汎用的な SQL クエリツール（スキーマ一覧、オブジェクト一覧、メタデータ取得、SQL実行）が登録される。

### 6. DATABASE_INSPECT 全関数の MCP ツール登録
`sql/05_register_inspect_tools.sql` を ADMIN ユーザーで SQLcl から実行する。

```bash
sql admin/<パスワード>@<接続文字列>
@sql/05_register_inspect_tools.sql
```

DATABASE_INSPECT パッケージの全9関数がラッパー関数経由で MCP ツールとして登録される。
ラッパー関数は `tool_name`（エージェントチームID）を内部で自動設定するため、MCP クライアント側ではチームIDを意識する必要がない。

> **注意**: チームIDが 4 以外の場合は、`05_register_inspect_tools.sql` 内の `_4` サフィックスを実際のIDに変更すること。
> チームIDの確認: `SELECT id#, agent_team_name FROM HAMORITA.DATABASE_INSPECT_AGENT_TEAMS$;`

### 7. MCP サーバー設定
`.mcp.json` に adb-mcp サーバーを設定する。Bearer トークンはADBコンソールから取得する。

```json
{
  "mcpServers": {
    "adb-mcp": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote",
        "https://dataaccess.adb.<リージョン>.oraclecloudapps.com/adb/mcp/v1/databases/<ADB OCID>",
        "--allow-http",
        "--header", "Authorization:${AUTH_HEADER}"
      ],
      "env": {
        "AUTH_HEADER": "Bearer <トークン>"
      }
    }
  }
}
```

ツール登録後、MCP サーバーを再起動（VSCode で MCP サーバー再接続）すると新ツールが使えるようになる。

## MCP ツール一覧

### 基本ツール（04_register_mcp_tools.sql）

| ツール名 | 機能 |
|---|---|
| `LIST_SCHEMAS` | スキーマ一覧を取得 |
| `LIST_OBJECTS` | 指定スキーマのオブジェクト一覧 |
| `GET_OBJECT_DETAILS` | カラム・索引・制約の詳細 |
| `EXECUTE_SQL` | 読み取り専用SQLの実行 |

### DATABASE_INSPECT ツール（05_register_inspect_tools.sql）

| ツール名 | 元の関数 | 機能 | AI活用 |
|---|---|---|---|
| `INSPECT_LIST_OBJECTS` | list_objects | チーム内オブジェクト一覧 | - |
| `INSPECT_LIST_INCOMING_DEPS` | list_incoming_dependencies | 被依存オブジェクト一覧 | - |
| `INSPECT_LIST_OUTGOING_DEPS` | list_outgoing_dependencies | 依存先オブジェクト一覧 | - |
| `INSPECT_GET_DDL` | retrieve_object_metadata | 完全なDDL取得 | - |
| `INSPECT_SEARCH_CODE` | retrieve_object_metadata_chunks | 自然言語でコード検索 | ベクトル検索 |
| `INSPECT_EXPAND_CHUNK` | expand_object_metadata_chunk | 検索結果の周辺コード展開 | - |
| `INSPECT_SUMMARIZE` | summarize_object | オブジェクトをAI要約 | LLM |
| `INSPECT_GENERATE_CHART` | generate_chart | Mermaid/HTML図表生成 | LLM |
| `INSPECT_GENERATE_PLDOC` | generate_pldoc | PLDoc自動生成 | LLM |

## 提供ツール（PL/SQL 関数）

| 関数名 | 機能 |
|---|---|
| `list_objects` | データベースオブジェクト一覧を取得 |
| `list_incoming_dependencies` | 入力依存関係を取得 |
| `list_outgoing_dependencies` | 出力依存関係を取得 |
| `generate_graph` | Mermaid/HTML で依存関係を可視化 |
| `retrieve_object_metadata` | DDL メタデータを取得 |
| `retrieve_object_metadata_chunks` | ハイブリッド検索（ベクトル+テキスト） |
| `expand_object_metadata_chunk` | チャンクのコンテキストを拡張 |
| `summarize_object` | オブジェクトを要約 |
| `generate_pldoc` | PLDoc ドキュメントを自動生成 |

## 検証結果

`docs/setup_log.md` に記録。
