# スキル ルーティング

タスクに応じて、該当するスキルファイルを読み込んでから作業すること。

## ルーティング表

| やりたいこと | 読むスキル | サブエージェント |
|---|---|---|
| OCI 構成図を draw.io で作成 | `skills/drawio-oci/skill.md` | 推奨（XML組み立てが重い） |
| DATABASE_INSPECT の検証・調査 | `oracle-aaidb-inspect/skill.md` | 推奨（SQL実行・ドキュメント参照） |
| OCI Object Storage の検証・調査 | `oracle-aaidb-objstore/skill.md` | 推奨（SQL実行・ドキュメント参照） |
| Oracle Database 全般の質問 | oracle-db-skills（外部） | 任意 |

## 外部スキル: oracle-db-skills

Oracle Database 全般（SQL, PL/SQL, パフォーマンス, セキュリティ, ORDS, SQLcl 等）の
リファレンスガイド集（128個）。Oracle 関連の作業で不明点があれば参照する。

- インストール: `npx skills add krisrice/oracle-db-skills`
- リポジトリ: https://github.com/krisrice/oracle-db-skills
- 特に関連が深いスキル:
  - `skills/sqlcl/sqlcl-mcp-server.md` — SQLcl MCP サーバーの設定
  - `skills/architecture/oracle-cloud-oci.md` — OCI 上の ATP/ADW ガイド

## サブエージェントの使い方

以下のタスクはサブエージェントに委譲し、メインコンテキストの消費を抑えること:

- **ドキュメント参照**: GitHub リポジトリや Oracle 公式ドキュメントの調査
- **SQL の試行錯誤**: ADB MCP サーバー経由での SQL 実行・デバッグ
- **構成図の XML 組み立て**: OCI Library のデコード・レイアウト計算
- **外部スキルの検索**: oracle-db-skills 内の該当スキルファイルの特定・要約

サブエージェントには該当するスキルファイルの内容を渡して起動すること。

## スキルファイルの共通ルール

- 各スキルファイルは 150 行以内に収める
- 構成: Overview → 手順 → トラブルシューティング → Version Notes → Sources
- 公式ドキュメントの参照リンクを必ず Sources に記載する
