---
description: スクリプト、スキーマ、ルート、エクスポートなど、真実の源泉となるファイルからドキュメントを同期します。
---

# ドキュメントを更新する

真実の源泉となるファイルから生成する形で、ドキュメントをコードベースと同期させます。

## ステップ 1: 真実の源泉を特定する

| Source                              | Generates                         |
| ----------------------------------- | --------------------------------- |
| `package.json` scripts              | 利用可能コマンドのリファレンス    |
| `.env.example`                      | 環境変数ドキュメント              |
| `openapi.yaml` / route files        | API エンドポイントのリファレンス  |
| Source code exports                 | 公開 API ドキュメント             |
| `Dockerfile` / `docker-compose.yml` | インフラ設定ドキュメント          |

## ステップ 2: Script Reference を生成する

1. `package.json`（または `Makefile`, `Cargo.toml`, `pyproject.toml`）を読む
2. すべての script / command を説明付きで抽出する
3. リファレンステーブルを生成する:

```markdown
| Command         | Description                              |
| --------------- | ---------------------------------------- |
| `npm run dev`   | Start development server with hot reload |
| `npm run build` | Production build with type checking      |
| `npm test`      | Run test suite with coverage             |
```

## ステップ 3: 環境変数ドキュメントを生成する

1. `.env.example`（または `.env.template`, `.env.sample`）を読む
2. すべての変数と用途を抽出する
3. 必須 / 任意に分類する
4. 期待される形式と有効値を記録する

```markdown
| Variable       | Required | Description                       | Example                             |
| -------------- | -------- | --------------------------------- | ----------------------------------- |
| `DATABASE_URL` | Yes      | PostgreSQL connection string      | `postgres://user:pass@host:5432/db` |
| `LOG_LEVEL`    | No       | Logging verbosity (default: info) | `debug`, `info`, `warn`, `error`    |
```

## ステップ 4: Contributing Guide を更新する

`docs/CONTRIBUTING.md` を生成または更新し、以下を含める:

- 開発環境セットアップ（前提条件、インストール手順）
- 利用可能なスクリプトと用途
- テスト手順（実行方法、新規テストの書き方）
- コードスタイルの強制手段（linter、formatter、pre-commit hook）
- PR 提出チェックリスト

## ステップ 5: Runbook を更新する

`docs/RUNBOOK.md` を生成または更新し、以下を含める:

- デプロイ手順（ステップバイステップ）
- ヘルスチェックエンドポイントと監視
- よくある問題と修正方法
- ロールバック手順
- アラートとエスカレーション経路

## ステップ 6: 鮮度チェック

1. 90 日以上更新されていないドキュメントファイルを探す
2. 最近のソースコード変更と照合する
3. 古くなっている可能性が高い文書を手動レビュー対象としてフラグ付けする

## ステップ 7: 要約を表示する

```text
Documentation Update
----------------------------------------------
Updated:  docs/CONTRIBUTING.md (scripts table)
Updated:  docs/ENV.md (3 new variables)
Flagged:  docs/DEPLOY.md (142 days stale)
Skipped:  docs/API.md (no changes detected)
----------------------------------------------
```

## ルール

- **唯一の真実の源泉**: 常にコードから生成し、生成セクションを手で編集しない
- **手書きセクションは保持**: 生成対象部分だけ更新し、手書き prose は残す
- **生成コンテンツを明示**: 生成セクションは `<!-- AUTO-GENERATED -->` で囲う
- **頼まれていない新規文書は作らない**: 明示要求がある場合だけ新規 doc ファイルを作る
