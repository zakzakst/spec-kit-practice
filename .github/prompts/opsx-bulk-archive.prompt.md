---
description: 複数の完了済み change をまとめて archive する
---

複数の完了済み change を 1 回で archive する。

このプロンプトは、コードベースを確認して実際にどこまで実装済みかを判断しつつ、spec conflict も考慮して賢く一括 archive するためのもの。

**Input**: 不要（選択式）

**Steps**

1. **アクティブな change を取得する**

   `openspec list --json` を実行して全 active changes を取得する。
   1 件もなければその旨を伝えて終了する。

2. **archive 対象を選択させる**

   **AskUserQuestion tool** の multi-select で選ばせる。
   - 各 change の schema を表示
   - `All changes` オプションを含める
   - 1 件以上選択可能

   **IMPORTANT**: 自動選択しない。必ずユーザーに選ばせる。

3. **各 change の状態をまとめて検証する**

   change ごとに次を集める:

   a. **Artifact status**
   - `openspec status --change "<name>" --json`
   - `schemaName` と `artifacts`

   b. **Task completion**
   - `openspec/changes/<name>/tasks.md`
   - `- [ ]` と `- [x]` の件数
   - なければ `No tasks`

   c. **Delta specs**
   - `openspec/changes/<name>/specs/`
   - 各 capability spec の有無
   - `### Requirement: <name>` 行から要件名を抽出

4. **spec conflict を検出する**

   `capability -> [changes that touch it]` の対応を作る。

   ```text
   auth -> [change-a, change-b]  <- CONFLICT
   api  -> [change-c]            <- OK
   ```

   同一 capability に対して 2 件以上の delta spec があれば conflict とみなす。

5. **agent 的に conflict を解消する**

   各 conflict についてコードベースを調べる:

   a. 各 conflicting change の delta spec を読み、何を追加 / 変更しようとしているか理解する

   b. コードベースを検索して実装証拠を探す
   - 要件に対応しそうなコード
   - 関連ファイル、関数、テスト

   c. 解決方針を決める
   - 片方だけ実装済み -> その change の spec だけ sync
   - 両方実装済み -> 作成日時順に適用（古い方から、新しい方が上書き）
   - どちらも未実装 -> spec sync をスキップし、警告

   d. 解決結果を記録する
   - どの change の spec を適用するか
   - 順番
   - 根拠

6. **統合ステータス表を表示する**

   ```text
   | Change | Artifacts | Tasks | Specs | Conflicts | Status |
   |--------|-----------|-------|-------|-----------|--------|
   | schema-management | Done | 5/5 | 2 delta | None | Ready |
   | project-config | Done | 3/3 | 1 delta | None | Ready |
   | add-oauth | Done | 4/4 | 1 delta | auth (!) | Ready* |
   | add-verify-skill | 1 left | 2/5 | None | None | Warn |
   ```

   conflict がある場合は解決方針も表示する:

   ```text
   * Conflict resolution:
     - auth spec: Will apply add-oauth then add-jwt (both implemented, chronological order)
   ```

   未完了 change があれば警告も出す:

   ```text
   Warnings:
   - add-verify-skill: 1 incomplete artifact, 3 incomplete tasks
   ```

7. **一括操作の確認を取る**

   **AskUserQuestion tool** で単一確認を行う。

   例:
   - `Archive all N changes`
   - `Archive only N ready changes (skip incomplete)`
   - `Cancel`

   未完了 change が含まれる場合は、警告付きで archive されることを明確に伝える。

8. **確定した change を順に archive する**

   conflict 解決順を尊重して処理する。

   a. **spec sync**
   - delta specs があれば `openspec-sync-specs` 相当で同期する
   - conflict があれば解決済み順序で適用する

   b. **archive 実行**
   ```bash
   mkdir -p openspec/changes/archive
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

   c. **結果を記録**
   - Success
   - Failed
   - Skipped

9. **最終サマリを表示する**

   ```text
   ## Bulk Archive Complete

   Archived 3 changes:
   - schema-management-cli -> archive/2026-01-19-schema-management-cli/
   - project-config -> archive/2026-01-19-project-config/
   - add-oauth -> archive/2026-01-19-add-oauth/

   Skipped 1 change:
   - add-verify-skill (user chose not to archive incomplete)

   Spec sync summary:
   - 4 delta specs synced to main specs
   - 1 conflict resolved (auth: applied both in chronological order)
   ```

   失敗があれば:

   ```text
   Failed 1 change:
   - some-change: Archive directory already exists
   ```

**Conflict Resolution Examples**

Example 1: 片方だけ実装済み

```text
Conflict: specs/auth/spec.md touched by [add-oauth, add-jwt]

Checking add-oauth:
- Delta adds "OAuth Provider Integration" requirement
- Searching codebase... found src/auth/oauth.ts implementing OAuth flow

Checking add-jwt:
- Delta adds "JWT Token Handling" requirement
- Searching codebase... no JWT implementation found

Resolution: Only add-oauth is implemented. Will sync add-oauth specs only.
```

Example 2: 両方実装済み

```text
Conflict: specs/api/spec.md touched by [add-rest-api, add-graphql]

Checking add-rest-api (created 2026-01-10):
- Delta adds "REST Endpoints" requirement
- Searching codebase... found src/api/rest.ts

Checking add-graphql (created 2026-01-15):
- Delta adds "GraphQL Schema" requirement
- Searching codebase... found src/api/graphql.ts

Resolution: Both implemented. Will apply add-rest-api specs first,
then add-graphql specs (chronological order, newer takes precedence).
```

**Output On Success**

```text
## Bulk Archive Complete

Archived N changes:
- <change-1> -> archive/YYYY-MM-DD-<change-1>/
- <change-2> -> archive/YYYY-MM-DD-<change-2>/

Spec sync summary:
- N delta specs synced to main specs
- No conflicts (or: M conflicts resolved)
```

**Output On Partial Success**

```text
## Bulk Archive Complete (partial)

Archived N changes:
- <change-1> -> archive/YYYY-MM-DD-<change-1>/

Skipped M changes:
- <change-2> (user chose not to archive incomplete)

Failed K changes:
- <change-3>: Archive directory already exists
```

**Output When No Changes**

```text
## No Changes to Archive

No active changes found. Use `/opsx:new` to create a new change.
```

**Guardrails**
- 任意件数を許可する
- 必ず選択を促し、自動選択しない
- spec conflict は早期検出し、コードベースを見て解消する
- 両方実装済みなら時系列順に適用する
- 実装がない場合のみ sync をスキップし、警告する
- 確定前に per-change status を明確に見せる
- 確認は一括で 1 回にまとめる
- success / skip / fail をすべて追跡して報告する
- `.openspec.yaml` は archive 時に保持される
- archive 先は `YYYY-MM-DD-<name>`
- archive 先が既存ならその change だけ失敗として他は続行する
