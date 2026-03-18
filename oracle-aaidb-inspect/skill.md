# DATABASE_INSPECT 検証スキル

## Overview

Oracle ADB の `DATABASE_INSPECT` PL/SQL パッケージを MCP ツールとして検証するスキル。
Select AI Agent は使わず、`database_inspect_tool.sql` のみインストールし PL/SQL 関数を直接呼び出す構成。

- ベクトル検索 + Oracle Text のハイブリッド検索に対応
- マルチプロバイダー対応（OCI, OpenAI, Cohere, Azure, Google, AWS）

## SQL 実行手順

| Step | ファイル | 内容 | 実行ユーザー |
|------|---------|------|-------------|
| 1 | `sql/00_prerequisites.sql` | ADB バージョン・権限の確認 | ADMIN |
| 2 | `sql/01_create_ai_profile.sql` | エンベディングモデル付き AI プロファイル作成 | ADMIN |
| 3 | `sql/02_install_tool.sql` | DATABASE_INSPECT パッケージインストール | ADMIN |
| 4 | `sql/03_test_queries.sql` | パッケージの動作確認 | ADMIN |
| 5 | `sql/04_register_mcp_tools.sql` | 基本 MCP ツール登録（SQLcl で実行） | ADMIN |
| 6 | `sql/05_register_inspect_tools.sql` | DATABASE_INSPECT 全9関数の MCP 登録（SQLcl） | ADMIN |
| 7 | MCP サーバー再起動 | VSCode で MCP 再接続 | — |

> チームIDが 4 以外の場合は `05_register_inspect_tools.sql` 内の `_4` サフィックスを変更すること。
> 確認: `SELECT id#, agent_team_name FROM HAMORITA.DATABASE_INSPECT_AGENT_TEAMS$;`

## MCP ツール一覧

### 基本ツール（04_register_mcp_tools.sql）

`LIST_SCHEMAS` / `LIST_OBJECTS` / `GET_OBJECT_DETAILS` / `EXECUTE_SQL`

### DATABASE_INSPECT ツール（05_register_inspect_tools.sql）

| ツール名 | 機能 | AI活用 |
|---|---|---|
| INSPECT_LIST_OBJECTS | チーム内オブジェクト一覧 | — |
| INSPECT_LIST_INCOMING_DEPS | 被依存オブジェクト一覧 | — |
| INSPECT_LIST_OUTGOING_DEPS | 依存先オブジェクト一覧 | — |
| INSPECT_GET_DDL | 完全な DDL 取得 | — |
| INSPECT_SEARCH_CODE | 自然言語でコード検索 | ベクトル検索 |
| INSPECT_EXPAND_CHUNK | 検索結果の周辺コード展開 | — |
| INSPECT_SUMMARIZE | オブジェクトを AI 要約 | LLM |
| INSPECT_GENERATE_CHART | Mermaid/HTML 図表生成 | LLM |
| INSPECT_GENERATE_PLDOC | PLDoc 自動生成 | LLM |

## トラブルシューティング

- **パッケージが INVALID**: ADB 26ai 未満 or 権限不足 → `SHOW ERRORS PACKAGE BODY DATABASE_INSPECT;`
- **AI プロファイル失敗**: OCI クレデンシャル・compartment_id・リージョン対応を確認
- **ベクトル検索が動かない**: `embedding_model` 設定済みか、ベクトルテーブル存在するか確認

## Oracle Version Notes (19c vs 26ai)

- DATABASE_INSPECT は **26ai 専用**（DBMS_CLOUD_AI_AGENT, DBMS_VECTOR_CHAIN を使用）
- 19c / 23ai では動作しない

## Sources

- [GitHub: database_inspect](https://github.com/oracle-devrel/oracle-autonomous-database-samples/tree/main/autonomous-ai-agents/database_inspect)
- [DBMS_CLOUD_AI ドキュメント](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/dbms-cloud-ai-package.html)
- [Select AI ドキュメント](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/sql-generation-ai-autonomous.html)
