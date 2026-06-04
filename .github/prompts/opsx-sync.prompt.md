---
description: change 側の delta specs を main specs へ同期する
---

change の delta specs を main specs へ同期する。

これは **agent-driven** な操作であり、delta specs を読んで main specs を直接編集する。単純コピーではなく、シナリオ追加のような部分更新を賢く反映できるようにする。

**Input**: `/opsx:sync` の後に変更名を省略可能（例: `/opsx:sync add-auth`）。省略時は文脈から推定してよいが、曖昧なら必ず選択を求める。

**Steps**

1. **変更名がなければ選択させる**

   `openspec list --json` を実行し、**AskUserQuestion tool** で change を選ばせる。
   `specs/` 配下に delta specs を持つ change のみ表示する。

   **IMPORTANT**: 推測や自動選択はしない。

2. **delta specs を見つける**

   `openspec/changes/<name>/specs/*/spec.md` を探す。

   各 delta spec には次のようなセクションが含まれる:
   - `## ADDED Requirements`
   - `## MODIFIED Requirements`
   - `## REMOVED Requirements`
   - `## RENAMED Requirements`

   見つからなければユーザーへ伝えて停止する。

3. **delta spec ごとに main spec へ適用する**

   capability ごとに:

   a. `openspec/changes/<name>/specs/<capability>/spec.md` を読む

   b. `openspec/specs/<capability>/spec.md` を読む（存在しない場合あり）

   c. **賢く適用する**

   **ADDED Requirements**
   - main spec に存在しなければ追加
   - 既に存在するなら実質 MODIFIED として更新

   **MODIFIED Requirements**
   - 対応する requirement を探す
   - delta に書かれた変更だけを反映する
   - 触れられていない既存シナリオ / 内容は保持する

   **REMOVED Requirements**
   - requirement block ごと削除

   **RENAMED Requirements**
   - FROM を見つけ TO へ改名

   d. capability の main spec がまだない場合:
   - `openspec/specs/<capability>/spec.md` を新規作成
   - 簡単な Purpose セクション（必要なら TBD）を入れる
   - ADDED requirements を追加する

4. **要約を表示する**

   次をまとめる:
   - 更新した capabilities
   - 追加 / 変更 / 削除 / 改名した requirements

**delta spec の形式参照**

```markdown
## ADDED Requirements

### Requirement: New Feature
The system SHALL do something new.

#### Scenario: Basic case
- **WHEN** user does X
- **THEN** system does Y

## MODIFIED Requirements

### Requirement: Existing Feature
#### Scenario: New scenario to add
- **WHEN** user does A
- **THEN** system does B

## REMOVED Requirements

### Requirement: Deprecated Feature

## RENAMED Requirements

- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**重要原則: 賢いマージ**

プログラム的な単純マージではなく、**部分更新** を扱う:
- シナリオ追加だけなら MODIFIED のそのシナリオだけでよい
- delta は全文置換ではなく **意図** を表す
- 常識的な判断で自然にマージする

**成功時の出力**

```text
## Specs 同期完了: <change-name>

更新された main specs:

**<capability-1>**:
- 追加した requirement: "New Feature"
- 変更した requirement: "Existing Feature"（scenario を 1 件追加）

**<capability-2>**:
- 新しい spec ファイルを作成
- 追加した requirement: "Another Feature"

main specs の更新が完了しました。change はまだ active のままです。実装が完了したら archive してください。
```

**Guardrails**
- 変更前に delta と main の両方を読む
- delta に触れていない既存内容は保持する
- 不明点は確認する
- 何を変えているかを作業中に示す
- 2 回実行しても結果が安定する idempotent な操作にする
