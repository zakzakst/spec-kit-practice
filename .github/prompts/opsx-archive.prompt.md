---
description: 実験的ワークフローで完了済み change を archive する
---

実験的ワークフローで完了した change を archive する。

**Input**: `/opsx:archive` の後に変更名を省略可能（例: `/opsx:archive add-auth`）。省略時は文脈から推定を試みてよいが、曖昧なら必ず選択を求める。

**Steps**

1. **変更名がなければ選択させる**

   `openspec list --json` を実行して利用可能な change を取得し、**AskUserQuestion tool** で選ばせる。

   表示するのはアクティブな change のみ（archive 済みは除外）。
   可能なら各 change の schema も見せる。

   **IMPORTANT**: 推測や自動選択はしない。必ずユーザーに選ばせる。

2. **成果物の完了状況を確認する**

   `openspec status --change "<name>" --json` を実行する。

   解析対象:
   - `schemaName`
   - `artifacts` とその status

   **`done` でない artifact がある場合**
   - 未完了成果物を列挙して警告する
   - 続行確認を取る
   - 承認があれば進む

3. **タスク完了状況を確認する**

   tasks ファイル（通常は `tasks.md`）を読み、`- [ ]` と `- [x]` を数える。

   **未完了タスクがある場合**
   - 件数を示して警告する
   - 続行確認を取る
   - 承認があれば進む

   **tasks ファイルがない場合**: タスク警告なしで続行する。

4. **delta spec の同期状態を評価する**

   `openspec/changes/<name>/specs/` に delta specs があるか確認する。なければ同期確認は不要。

   **delta specs がある場合**
   - 各 delta spec を `openspec/specs/<capability>/spec.md` と比較する
   - 追加 / 変更 / 削除 / リネームの差分を把握する
   - まとめて概要を示してから選択を求める

   **Prompt options**
   - 変更が必要: `Sync now (recommended)`, `Archive without syncing`
   - 既に同期済み: `Archive now`, `Sync anyway`, `Cancel`

   ユーザーが同期を選んだら `/opsx:sync` 相当のロジックを実行する。選択にかかわらず archive 処理自体は続行してよい。

5. **archive を実行する**

   archive ディレクトリがなければ作る:
   ```bash
   mkdir -p openspec/changes/archive
   ```

   ターゲット名は当日の日付を使って `YYYY-MM-DD-<change-name>` とする。

   **ターゲットが既に存在する場合**
   - エラーで失敗させる
   - 既存 archive のリネームや別日での実行を提案する

   **存在しない場合**
   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **要約を表示する**

   次を含める:
   - Change name
   - 使用 schema
   - Archive location
   - Spec sync status（synced / skipped / no delta specs）
   - 警告事項（未完了 artifact / tasks など）

**Output On Success**

```text
## Archive Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** Synced to main specs

All artifacts complete. All tasks complete.
```

**Output On Success (No Delta Specs)**

```text
## Archive Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** No delta specs

All artifacts complete. All tasks complete.
```

**Output On Success With Warnings**

```text
## Archive Complete (with warnings)

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** Sync skipped (user chose to skip)

**Warnings:**
- Archived with 2 incomplete artifacts
- Archived with 3 incomplete tasks
- Delta spec sync was skipped (user chose to skip)

Review the archive if this was not intentional.
```

**Output On Error (Archive Exists)**

```text
## Archive Failed

**Change:** <change-name>
**Target:** openspec/changes/archive/YYYY-MM-DD-<name>/

Target archive directory already exists.

**Options:**
1. Rename the existing archive
2. Delete the existing archive if it's a duplicate
3. Wait until a different date to archive
```

**Guardrails**
- 変更名未指定時は必ず選択させる
- 完了判定には `openspec status --json` を使う
- 警告があっても archive 自体は妨げず、確認のうえ進める
- ディレクトリ移動時に `.openspec.yaml` も保持される
- 起きたことを明確に要約する
- sync が必要なら `/opsx:sync` の流儀に従う
- delta specs がある場合は、必ず同期評価と要約提示を先に行う
