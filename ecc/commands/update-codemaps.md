---
description: プロジェクト構造を走査し、トークン効率のよいアーキテクチャ codemap を生成します。
---

# Codemap を更新する

コードベース構造を分析し、AI コンテキスト向けに軽量化したアーキテクチャ文書を生成します。

## ステップ 1: プロジェクト構造を走査する

1. プロジェクト種別（monorepo、単一アプリ、ライブラリ、マイクロサービス）を特定する
2. すべてのソースディレクトリ（`src/`, `lib/`, `app/`, `packages/`）を見つける
3. エントリポイント（`main.ts`, `index.ts`, `app.py`, `main.go` など）をマップする

## ステップ 2: Codemap を生成する

`docs/CODEMAPS/`（または `.reports/codemaps/`）に codemap を新規作成または更新する:

| File              | Contents                                                      |
| ----------------- | ------------------------------------------------------------- |
| `architecture.md` | 高レベルのシステム図、サービス境界、データフロー             |
| `backend.md`      | API ルート、middleware チェーン、service -> repository 対応  |
| `frontend.md`     | ページツリー、コンポーネント階層、状態管理フロー             |
| `data.md`         | データベーステーブル、関連、マイグレーション履歴             |
| `dependencies.md` | 外部サービス、サードパーティ統合、共有ライブラリ             |

### Codemap 形式

各 codemap は、AI コンテキスト消費に向けてトークン効率よく保つ:

```markdown
# Backend Architecture

## Routes

POST /api/users -> UserController.create -> UserService.create -> UserRepo.insert
GET /api/users/:id -> UserController.get -> UserService.findById -> UserRepo.findById

## Key Files

src/services/user.ts (business logic, 120 lines)
src/repos/user.ts (database access, 80 lines)

## Dependencies

- PostgreSQL (primary data store)
- Redis (session cache, rate limiting)
- Stripe (payment processing)
```

## ステップ 3: 差分検出

1. 既存 codemap がある場合、差分率を計算する
2. 変更が 30% を超える場合、差分を表示して上書き前にユーザー承認を求める
3. 変更が 30% 以下ならその場で更新する

## ステップ 4: メタデータを追加する

各 codemap に鮮度ヘッダーを追加する:

```markdown
<!-- Generated: 2026-02-11 | Files scanned: 142 | Token estimate: ~800 -->
```

## ステップ 5: 分析レポートを保存する

`.reports/codemap-diff.txt` に要約を書き出す:

- 前回走査以降に追加 / 削除 / 変更されたファイル
- 新たに検出した依存関係
- アーキテクチャ変更（新ルート、新サービスなど）
- 90 日以上更新されていないドキュメントへの鮮度警告

## ヒント

- 実装詳細ではなく **高レベル構造** に集中する
- フルコードブロックより **ファイルパスと関数シグネチャ** を優先する
- 各 codemap は効率的に読み込めるよう **1000 トークン未満** に保つ
- 冗長な文章より ASCII 図でデータフローを示す
- 大きな機能追加やリファクタリング後に実行する
