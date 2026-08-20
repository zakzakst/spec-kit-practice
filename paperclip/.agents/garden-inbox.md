---
name: garden-inbox
description: Paperclip ユーザーの Mine inbox をスキャンし、取り消し可能な archive 候補を分類し、checkbox confirmation を求め、承認された項目だけを archive します。issue、branch、workspace を変更せずに inbox を整理・剪定・清掃したいときに使います。
---

# Garden Inbox

すべての段階で bundled script を使います。ワークフローの順序は厳密に `scan` → `confirm` → `apply` です。

## Safety contract

- `scan` は読み取り専用として扱います。読み取ってよいのは inbox、workspace、close-readiness、ローカル Git メタデータのみです。
- `confirm` は confirmation card への書き込み専用です。inbox entry の archive や、issue フィールド、branch、workspace の変更をしてはいけません。
- `apply` は、解決済みの `request_checkbox_confirmation` interaction の後にのみ実行します。元の `candidates.json` に含まれる、承認された option ID だけを archive します。
- inbox の archive 状態はユーザーごとの表示状態です。元の issue は変わらず、取り消し可能です。
- issue status の変更、branch の削除、workspace の掃除、issue の削除を inbox archive の代わりに使ってはいけません。

## Inputs

`PAPERCLIP_API_URL` と `PAPERCLIP_API_KEY` が必要です。script は URL 末尾の `/api` を取り除きます。対象ユーザーは run JWT payload の `responsible_user_id` から解決します。`--user-id <uuid>` は明示的な対象上書きにのみ使ってください。

利用可能なら run 所有の出力ディレクトリを使います:

```bash
RUN_DIR="${PAPERCLIP_RUN_SCRATCH_DIR:-${PAPERCLIP_TASK_SCRATCH_DIR:-.}}/garden-inbox"
mkdir -p "$RUN_DIR"
```

## 1. Scan

```bash
node .agents/skills/garden-inbox/scripts/garden-inbox.mjs scan \
  --output-dir "$RUN_DIR" \
  --stale-days 60
```

`garden-inbox-report.md` と `candidates.json` を確認します。レポートは各 inbox row を必ず1つの bucket に分類します:

scan は、endpoint の全体 500 行上限で古い row が黙って欠落しないよう、含める issue status ごとに Mine を別々に読みます。いずれかの単一 status の query がその上限に達した場合、両方の出力は coverage が途中までの可能性を示します。その scan を完了扱いにしないでください。

- A: merged / archived workspace で、関連 work がすべて terminal。既定で選択済み。
- B: terminal または workspace 消失済みで、しきい値を超えて idle な work。既定で選択済み。
- C: base より ahead だが stale な work。既定では絶対に選択しない。
- D: keep。archive の候補としては出しません。

D bucket の entry を手動で candidate file に昇格させないでください。

## 2. Confirm

driving issue に checkbox interaction を投稿します:

```bash
node .agents/skills/garden-inbox/scripts/garden-inbox.mjs confirm \
  --issue-id "$PAPERCLIP_TASK_ID" \
  --candidates "$RUN_DIR/candidates.json"
```

scan の候補が 200 件を超える場合、script は順番に card を投稿します。`confirm` を同じ scan file で再実行しても idempotent です。driving issue は、周囲の Paperclip heartbeat workflow が求める waiting posture のままにしてください。

以前のパスでユーザーが候補を却下した場合は、`--unselect <issueId>` を付けます（複数回指定可）。そうすると初期状態は未選択になり、説明には以前の却下が記録されます。過去に却下された項目を、デフォルトでチェック済みにして再提示しないでください。

開発や payload 確認では、POST を抑止します:

```bash
node .agents/skills/garden-inbox/scripts/garden-inbox.mjs confirm \
  --issue-id "$PAPERCLIP_TASK_ID" \
  --candidates "$RUN_DIR/candidates.json" \
  --dry-run
```

## 3. Apply accepted selections

interaction が解決して wake された後は、wake payload から resolved interaction ID を取り出して次を実行します:

```bash
node .agents/skills/garden-inbox/scripts/garden-inbox.mjs apply \
  --issue-id "$PAPERCLIP_TASK_ID" \
  --interaction-id "$INTERACTION_ID" \
  --candidates "$RUN_DIR/candidates.json"
```

reject、expiry、または空の選択が承認された場合、この script は何も archive しません。apply は、明示的な `--user-id` override を含めて scan file の対象ユーザーを保持します。summary には、各 archive row について API の undo path と target-user body が含まれます。

保存済みの interaction response を使えば、API 書き込みなしで `apply` をテストできます:

```bash
node .agents/skills/garden-inbox/scripts/garden-inbox.mjs apply \
  --issue-id "$PAPERCLIP_TASK_ID" \
  --interaction-file resolved-interaction.json \
  --candidates "$RUN_DIR/candidates.json" \
  --dry-run
```

## Verify the bundled logic

分類や selection safety を変えた後は、依存なしの Node テストを実行します:

```bash
node --test .agents/skills/garden-inbox/scripts/garden-inbox.test.mjs
```
