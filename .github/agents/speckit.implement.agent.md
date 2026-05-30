---
description: tasks.md に定義された全タスクを順に処理し、実装計画を実行する。
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## 概要 (Outline)

1. リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` を実行し、`FEATURE_DIR` と `AVAILABLE_DOCS` を解析する。パスはすべて絶対パスとする。引数中のシングルクォートは必要に応じてエスケープすること。

2. **チェックリストのステータス確認 (Check checklists status)**（`FEATURE_DIR/checklists/` がある場合）
   - すべてのチェックリストファイルを走査する
   - 各ファイルについて集計する
     - 総項目数 (Total items): `- [ ]`, `- [X]`, `- [x]`
     - 完了項目数 (Completed items): `- [X]`, `- [x]`
     - 未完了項目数 (Incomplete items): `- [ ]`
   - 次のようなステータス表を作る

     ```text
     | チェックリスト (Checklist) | 合計 (Total) | 完了 (Completed) | 未完了 (Incomplete) | ステータス (Status) |
     |-----------|-------|-----------|------------|--------|
     | ux.md | 12 | 12 | 0 | PASS |
     | test.md | 8 | 5 | 3 | FAIL |
     | security.md | 6 | 6 | 0 | PASS |
     ```

   - 全体判定:
     - **PASS**: すべての未完了項目 (incomplete) が 0 の場合
     - **FAIL**: 1 つ以上未完了項目 (incomplete) がある場合

   - 未完了が 1 つでもあれば:
     - 表を表示する
     - `Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no) (一部のチェックリストが未完了です。このまま実装を進めますか？(yes/no))` と聞いて**処理を停止**する
     - `no`, `wait`, `stop`, `いいえ` なら中止する
     - `yes`, `proceed`, `continue`, `はい` なら次へ進む

   - すべて完了なら:
     - PASS 表示後、自動で次へ進む

3. 実装コンテキストを読み、分析する。
   - **必須 (REQUIRED)**: `tasks.md`
   - **必須 (REQUIRED)**: `plan.md`
   - **存在する場合のみ (IF EXISTS)**: `data-model.md`
   - **存在する場合のみ (IF EXISTS)**: `contracts/`
   - **存在する場合のみ (IF EXISTS)**: `research.md`
   - **存在する場合のみ (IF EXISTS)**: `quickstart.md`

4. **プロジェクトセットアップの検証 (Project Setup Verification)**
   - 実際のプロジェクト構成に応じて ignore 系ファイルを作成 / 検証する

   判定ロジック:
   - Git リポジトリなら `.gitignore` を作成 / 検証
     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```
   - `Dockerfile*` または plan.md で Docker 言及あり -> `.dockerignore`
   - `.eslintrc*` -> `.eslintignore`
   - `eslint.config.*` -> config の `ignores` に必要パターンがあるか確認
   - `.prettierrc*` -> `.prettierignore`
   - `.npmrc` または `package.json` -> `.npmignore`（公開時）
   - `*.tf` -> `.terraformignore`
   - Helm chart があれば `.helmignore`

   既存ファイルがある場合:
   - 必須パターンが入っているか確認し、不足分のみ追記

   ない場合:
   - 検出技術に応じた標準パターンで新規作成

   技術ごとの一般的な共通パターン (Common patterns by technology):
   - Node.js / JavaScript / TypeScript: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - Python: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - Java: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - C# / .NET: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - Go: `*.exe`, `*.test`, `vendor/`, `*.out`
   - Ruby: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - PHP: `vendor/`, `*.log`, `*.cache`, `*.env`
   - Rust: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - Kotlin: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - C++: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - C: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `Makefile`, `config.log`, `.idea/`, `*.log`, `.env*`
   - Swift: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - R: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - ユニバーサル (Universal): `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   ツール固有のパターン (Tool-specific patterns):
   - Docker: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - ESLint: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - Prettier: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - Terraform: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - Kubernetes / k8s: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. `tasks.md` を解析して以下を抽出する。
   - タスクフェーズ (Task phases): セットアップ (Setup)、テスト (Tests)、コア実装 (Core)、統合 (Integration)、磨き込み (Polish)
   - タスク依存関係 (Task dependencies): 逐次 / 並列
   - タスク詳細: ID、説明、パス、並列マーカー `[P]`
   - 実行フロー: 順序と依存関係

6. タスク計画に従って実装する。
   - フェーズ単位で進める
   - 依存関係を守る
   - TDD 方針なら対応する実装前にテストを書く
   - 同じファイルに触るタスクは逐次実行
   - 各フェーズ完了時に検証する

7. 実装ルール (Implementation Rules):
   - セットアップを最優先 (Setup first)
   - コードの前にテストを作成（必要な場合） (Tests before code)
   - コア機能開発 (Core development)
   - 統合ワーク (Integration work)
   - 磨き込みと検証 (Polish and validation)

8. 進捗追跡とエラーハンドリング (Progress Tracking & Error Handling):
   - タスク完了ごとに進捗報告
   - 非並列タスクが失敗したら停止
   - 並列タスク `[P]` は成功分を継続し、失敗分を報告
   - デバッグに十分な文脈付きエラーを出す
   - 進められない場合は次の一手を提案
   - **重要**: 完了したタスクは tasks ファイルで `[X]` または `[x]` にする

9. 完了時検証 (Final Verification):
   - 必須タスクがすべて完了しているか
   - 実装が元 spec と一致しているか
   - テストとカバレッジが要件を満たすか
   - 技術計画に従っているか
   - 最終ステータスと完了内容を要約する

注記: このコマンドは、完全なタスク分解が `tasks.md` に存在することを前提とする。不足している場合は `/speckit.tasks` の再生成を勧めること。
