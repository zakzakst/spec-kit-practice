---
description: OpenSpec の変更に含まれるタスクを実装する（Experimental）
---

OpenSpec の変更に含まれるタスクを実装する。

**Input**: 変更名を省略可能（例: `/opsx:apply add-auth`）。省略時は会話文脈から推定してよいが、曖昧な場合は利用可能な変更を必ず確認させること。

**Steps**

1. **変更を選ぶ**

   名前が指定されていればそれを使う。そうでなければ:
   - 会話内で変更名に言及があるか推定する
   - アクティブな変更が 1 件だけなら自動選択する
   - 曖昧なら `openspec list --json` を実行し、**AskUserQuestion tool** で選ばせる

   常に `Using change: <name>` を案内し、上書き方法（例: `/opsx:apply <other>`）も示すこと。

2. **ステータスを確認してスキーマを把握する**
   ```bash
   openspec status --change "<name>" --json
   ```
   JSON を解析して次を把握する:
   - `schemaName`: 使用中のワークフロー（例: `spec-driven`）
   - どの成果物がタスクを保持しているか

3. **apply 用 instructions を取得する**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   ここから取得できるもの:
   - Context file paths
   - Progress（total, complete, remaining）
   - Task list with status
   - 現在状態に応じた動的 instruction

   **状態別の扱い**
   - `state: "blocked"`: 不足成果物があるのでメッセージを出し、`/opsx:continue` を勧める
   - `state: "all_done"`: 完了を祝福し、archive を勧める
   - それ以外: 実装へ進む

4. **context files を読む**

   instructions 出力の `contextFiles` に列挙されたファイルを読む。
   スキーマにより内容は異なる:
   - **spec-driven**: proposal, specs, design, tasks
   - その他: CLI 出力の `contextFiles` をそのまま信頼する

5. **現在の進捗を表示する**

   表示内容:
   - 使用中 schema
   - Progress: `N/M tasks complete`
   - Remaining tasks の概要
   - CLI からの動的 instruction

6. **タスクを実装する（完了またはブロックされるまでループ）**

   各 pending task について:
   - 取り組むタスクを表示
   - 必要なコード変更を行う
   - 変更は最小限かつ焦点を絞る
   - tasks ファイルで `- [ ]` を `- [x]` に更新する
   - 次のタスクへ進む

   **次の場合は一時停止**
   - タスク内容が不明瞭 -> clarification を求める
   - 実装中に設計上の問題が見つかる -> 成果物更新を提案する
   - エラーや blocker が出る -> 報告して案内を待つ
   - ユーザーが中断する

7. **完了時または一時停止時に状態を表示する**

   表示内容:
   - 今回セッションで完了したタスク
   - Overall progress: `N/M tasks complete`
   - 全完了なら archive を勧める
   - 一時停止なら理由を説明して案内を待つ

**Output During Implementation**

```text
## Implementing: <change-name> (schema: <schema-name>)

Working on task 3/7: <task description>
[...implementation happening...]
Task complete

Working on task 4/7: <task description>
[...implementation happening...]
Task complete
```

**Output On Completion**

```text
## Implementation Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** 7/7 tasks complete
### Completed This Session
- [x] Task 1
- [x] Task 2
...

All tasks complete! You can archive this change with `/opsx:archive`.
```

**Output On Pause (Issue Encountered)**

```text
## Implementation Paused

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** 4/7 tasks complete

### Issue Encountered
<description of the issue>

**Options:**
1. <option 1>
2. <option 2>
3. Other approach

What would you like to do?
```

**Guardrails**
- 完了またはブロックまでタスクを進め続ける
- 開始前に必ず context files を読む
- 曖昧なタスクは実装前に確認する
- 実装で問題が見えたら成果物更新を提案して止まる
- コード変更は各タスクに最小限で絞る
- タスク完了直後にチェックを更新する
- エラー、blocker、要件不明時は推測で進めない
- 固定のファイル名を仮定せず、CLI の `contextFiles` を使う

**Fluid Workflow Integration**

このプロンプトは「change に対して随時 action する」モデルを支える:

- **いつでも呼べる**: 全成果物完成前でも、実装途中でもよい
- **成果物更新も許容**: 実装で設計課題が見つかったら、phase 固定にせず柔軟に更新を提案する
