---
name: check-pr
description: >
  GitHub、GitLab、Perforce の PR / MR / CL を対象に、review comment、
  failing check、PR 本文の不足を確認します。変更を確認・修正・提出準備したいときに
 使います。
license: MIT
compatibility: git と gh（GitHub CLI）、glab（GitLab CLI）、または p4（Perforce CLI）がインストールされ、認証済みであることが必要です。
metadata:
  author: greptileai
  version: "1.3"
allowed-tools: Bash(gh:*) Bash(glab:*) Bash(git:*) Bash(p4:*)
---

# PR をチェックする

pull request（GitHub）、merge request（GitLab）、または shelved changelist（Perforce）を
対象に、レビューコメント、ステータスチェック、説明文の完成度を調べ、見つかった問題への
対応を手伝います。

## 入力

- **PR/MR/CL 番号**（任意）: 指定がなければ、現在のブランチの PR/MR、または p4 の既定の保留中 changelist を検出します。

## 手順

### 0. プラットフォームを判定する

まず、`.p4config` ファイルや `P4CLIENT` / `P4PORT` 環境変数を見て、ユーザーが Perforce depot で作業しているか確認します:

```bash
# Perforce 環境を確認する
if p4 info >/dev/null 2>&1; then
  VCS="perforce"
else
  # git のリモート検出にフォールバックする
  REMOTE_URL=$(git remote get-url origin)
  if echo "$REMOTE_URL" | grep -qi "gitlab"; then
    VCS="gitlab"
  else
    VCS="github"
  fi
fi
```

ホスト名に "gitlab" を含まない self-hosted GitLab の場合は、ユーザーが入力として `--vcs gitlab` を渡して上書きできます。Perforce の場合は `--vcs perforce` で上書きできます。

### 1. PR/MR/CL を特定する

番号が与えられていればそれを使います。なければ検出します:

**GitHub:**
```bash
gh pr view --json number,headRefName,headRefOid -q '{number: .number, branch: .headRefName, head: .headRefOid}'
```

**GitLab:**
```bash
glab mr view --output json | jq '.iid'
```

**Perforce:**
```bash
# 現在のユーザー／クライアントの保留中 changelist を一覧表示する
p4 changes -s pending -u $P4USER -c $P4CLIENT
```

プラットフォームごとの主要なフィールドの違い:
- GitHub: `number`, `headRefName`, `headRefOid`
- GitLab: `iid`, `source_branch`, `sha`
- Perforce: changelist 番号（CL）、レビュー中 CL の `shelved` ファイル

### 2. PR/MR/CL の詳細を取得する

**GitHub:**
```bash
gh pr view <PR_NUMBER> --json title,body,state,reviews,comments,headRefName,headRefOid,statusCheckRollup
OWNER_REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
gh api "repos/$OWNER_REPO/pulls/<PR_NUMBER>/comments"
gh api --paginate "repos/$OWNER_REPO/issues/<PR_NUMBER>/comments?per_page=100"
```

GitHub の PR は issue でもあるため、一般的な PR コメントは issue comments endpoint にあります。Greptile は、毎回新しいレビューやコメントを作る代わりに、一般コメント 1 件を更新することがあります。PR が問題ないと判断する前に、`updated_at` で並べた最新の Greptile 作成一般コメントを、"Prompt to fix all with AI" セクションを含めて必ず確認してください。

**GitLab:**
```bash
glab mr view <MR_IID> --output json
# discussion を取得する（インライン差分コメントの type は "DiffNote"、一般コメントの type は null）
glab api "projects/:fullpath/merge_requests/<MR_IID>/discussions"
```

GitLab では必要に応じて discussion を paginate してください（`?per_page=100&page=N` を付けます）。

**Perforce:**
```bash
# changelist の説明、ファイル、ステータスを取得する
p4 describe -s <CL_NUMBER>

# shelved ファイルを取得する（レビュー中 CL 用）
p4 describe -S <CL_NUMBER>

# shelved changelist の差分を取得する
p4 diff2 //...@=<CL_NUMBER> //...@=<CL_NUMBER>

# レビューコメントを一覧表示する（p4 review ワークフローを使用する場合）
p4 review -c <CL_NUMBER>
```

Perforce CL の主要フィールド:
- `Change`: changelist number
- `Status`: `pending`, `submitted`, `shelved`
- `Description`: CL の説明／コミットメッセージ
- `Files`: CL に含まれるファイルの一覧

### 3. 保留中の check を待つ

分析する前に、すべてのステータスチェックが完了していることを確認します。GitHub で `PENDING` / `IN_PROGRESS`、GitLab で `running` / `pending` のチェックがあれば、すべてが終端状態になるまで 30 秒ごとにポーリングします。

**GitHub:** `gh pr view` の `statusCheckRollup` を poll します。

**GitLab:**
```bash
glab api "projects/:fullpath/merge_requests/<MR_IID>/pipelines"
```
パイプラインのステータス: `running`, `pending`, `success`, `failed`, `canceled`, `skipped`。`running` または `pending` のパイプラインがなくなるまでポーリングします。

**Perforce:** Perforce には標準の CI チェック機能がありません。チームがレビュー ツール（Swarm など）や shelve イベントで起動する外部 CI を使用している場合は、該当するシステムを確認します。それ以外の場合は、すぐに分析へ進みます。

### 4. 現在の head に対する新しい Greptile review を要求する

GitHub PR では、既存の Greptile レビュー、コメント、サマリーを、PR の現在の `headRefOid` に結びついていない限り最新とはみなさないでください。これは既存 PR に新しいコミットを push した直後に特に重要です。古いコミットに対する Greptile レビューは古いものであり、PR に Greptile コメントや過去のレビューが残っていても無効です。

Greptile の判定直前に、現在の head SHA を取得します:

```bash
HEAD_SHA=$(gh pr view <PR_NUMBER> --json headRefOid -q .headRefOid)
```

そのコミットの check-run を調べ、完了済みの Greptile run を要求します:

```bash
OWNER_REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
GREPTILE_CHECKS=$(gh api "repos/$OWNER_REPO/commits/$HEAD_SHA/check-runs?per_page=100" \
  --jq '[.check_runs[] | select(.name | test("greptile"; "i"))]')

# run が有効な最新の合格結果とみなされるのは、完了していて、かつ正常終了している場合だけです。
# GitHub check-run の conclusion: success, neutral, skipped, failure, timed_out,
# cancelled, action_required, stale。success / neutral だけを clean とみなし、
# それ以外（特に failure と action_required）はすべてブロックします。
FRESH_GREPTILE_COMPLETED=$(echo "$GREPTILE_CHECKS" \
  | jq '[.[] | select(.status == "completed")] | length')
FRESH_GREPTILE_CLEAN=$(echo "$GREPTILE_CHECKS" \
  | jq '[.[] | select(.status == "completed" and (.conclusion | IN("success","neutral")))] | length')
FRESH_GREPTILE_BLOCKING=$(echo "$GREPTILE_CHECKS" \
  | jq '[.[] | select(.status == "completed" and ((.conclusion | IN("success","neutral")) | not))] | length')

if [ "$FRESH_GREPTILE_COMPLETED" = "0" ]; then
  echo "Blocked: current PR head $HEAD_SHA に結びついた完了済み Greptile review/check がありません。"
  echo "PR check を done にする前に、この head に対する新しい Greptile review を依頼してください。"
  echo "推奨トリガー: gh pr comment <PR_NUMBER> --body \"@greptile review\""
  exit 1
fi

if [ "$FRESH_GREPTILE_BLOCKING" != "0" ] || [ "$FRESH_GREPTILE_CLEAN" = "0" ]; then
  echo "ブロック: Greptile は head $HEAD_SHA で完了しましたが、正常終了していません。"
  echo "(conclusion が failure/action_required/timed_out/cancelled/stale だったか、clean run が存在しません。)"
  echo "指摘に対応して push し、新しい head で success / clean に終わるまで Greptile を再実行してください。"
  exit 1
fi
```

現在の head に対する Greptile チェックが存在するものの `still pending` / `in progress` の場合は、古いレビュー資料に進まず、`greploop` と同じポーリングパターンで待ちます。妥当な待機時間の後でも現在の head に対する Greptile チェックが出てこない場合は、`HEAD_SHA` に対する新しい Greptile レビュー待ちとして PR をブロックし、そこで停止します。現在の head SHA に結びつけられない PR コメント、PR レビュー、Greptile サマリーからチェック完了とはみなしてはいけません。

Greptile 統合がある GitLab 環境でも、MR の現在の head SHA に対して同じ最新性ルールを適用します:

```bash
# 1. MR の現在の head SHA を取得する
MR_SHA=$(glab mr view <MR_IID> --output json | jq -r '.sha // .diff_refs.head_sha')

# 2. その正確な SHA の最新パイプラインを探し、その中の Greptile ジョブを取得する
LATEST_PIPELINE_ID=$(glab api "projects/:fullpath/merge_requests/<MR_IID>/pipelines" \
  | jq -r --arg sha "$MR_SHA" '[.[] | select(.sha == $sha)] | sort_by(.id) | last | .id // empty')

if [ -n "$LATEST_PIPELINE_ID" ]; then
  GREPTILE_JOBS=$(glab api "projects/:fullpath/pipelines/$LATEST_PIPELINE_ID/jobs" \
    | jq --arg sha "$MR_SHA" '[.[] | select(.name | test("greptile"; "i"))
        | {name, status, pipeline_sha: $sha}]')
else
  GREPTILE_JOBS='[]'
fi

GREPTILE_JOB_SUCCESS=$(echo "$GREPTILE_JOBS" \
  | jq '[.[] | select(.status == "success")] | length')
GREPTILE_JOB_BLOCKING=$(echo "$GREPTILE_JOBS" \
  | jq '[.[] | select(.status != "success")] | length')

# 3. Greptile が CI ジョブではなく MR ノートと統合されている場合は、最新の Greptile ノートが
#    現在の head SHA を参照し、問題のないレビューを報告していることを要求する
GREPTILE_NOTES=$(glab api "projects/:fullpath/merge_requests/<MR_IID>/discussions?per_page=100" \
  | jq --arg sha "$MR_SHA" '[.[].notes[]
      | select(.author.username | test("greptile"; "i"))]
      | sort_by(.updated_at // .created_at)')
GREPTILE_NOTE_CLEAN=$(echo "$GREPTILE_NOTES" \
  | jq --arg sha "$MR_SHA" 'if length == 0 then 0
      elif ((last.body // "") | contains($sha) and test("Confidence Score:[[:space:]]*5/5|Confidence:[[:space:]]*5/5|\\b5/5\\b"; "i") and (test("Prompt To Fix|blocking issue|failed|action required"; "i") | not)) then 1
      else 0 end')

if [ "$GREPTILE_JOB_SUCCESS" = "0" ] && [ "$GREPTILE_NOTE_CLEAN" = "0" ]; then
  echo "Blocked: no successful Greptile job or completed-clean current-head Greptile note is tied to MR head $MR_SHA."
  exit 1
fi

if [ "$GREPTILE_JOB_BLOCKING" != "0" ]; then
  echo "Blocked: at least one Greptile job for MR head $MR_SHA did not succeed."
  exit 1
fi
```

`MR_SHA` について、(a) Greptile ジョブまたはノートがまったくない（欠落）、(b) 最新の Greptile ジョブ／ノートが別の SHA に紐づいている（古い）、または (c) Greptile ジョブのステータスが `success` ではない場合は完了をブロックします。別の SHA に対する完了済み Greptile 結果も古いため、完了をブロックする必要があります。

Greptile 統合がある Perforce 環境では、CL の現在の shelved revision 識別子と、それに紐づく Greptile webhook／レビュー成果物を使って同じルールを適用します。以前の shelf に対する Greptile 結果は古いため、完了をブロックする必要があります。

### 5. Analyze the PR/MR

すべてのチェックが完了したら、次の項目を評価します:

#### A. Status Checks

- すべての CI チェックに合格しているか
- 失敗しているものがあれば、対象と失敗理由を特定する

#### B. PR/MR Description

- 説明文が完全で、チームの規約に従っているか
- 必須セクションがすべて記入されているか
- 更新が必要な TODO やプレースホルダーがないか

#### C. Review Comments

- 対応が必要なインラインコードレビューコメント
- ボットのレビューコメント（GitHub の `greptile-apps[bot]`、GitLab の Greptile ボットユーザー、リンターなど）を確認する
- 人間のレビュアーによるコメント
- **Perforce:** review comments from `p4 review` or external review tools

#### D. General Comments

- PR/MR のディスカッションコメント
- GitHub では issue comments endpoint を確認し、`updated_at` を使ってボットがその場で編集したコメントも見つける。新しいインラインコメントがなくても、Greptile が編集した最新サマリーに対応事項が含まれている場合がある
- ボットコメント（デプロイプレビューなど）。通常は情報提供のみ
- **Perforce:** CL description should include a clear summary, affected files rationale, and testing notes

### 6. Categorize issues

見つかった各問題を次のように分類します:

| 分類 | 意味 |
|---|---|
| **対応が必要** | コード変更、テスト改善、または修正が必要 |
| **情報提供** | 変更を必要としない確認事項、質問、参考情報 |
| **対応済み** | 後続のコミットで解決済みと思われる問題 |

### 7. Report findings

サマリー表を提示します:

| 領域 | 問題 | 状態 | 必要な対応 |
|------|-------|--------|---------------|
| ステータスチェック | CI ビルド失敗 | 失敗 | `src/api.ts` の型エラーを修正 |
| レビュー | 「null チェックを追加」- @reviewer | 対応が必要 | ガード節を追加 |
| 説明 | テスト計画に TODO プレースホルダーがある | 対応が必要 | テスト計画を記入 |
| レビュー | 「問題ありません」- @teammate | 情報提供 | なし |

### 8. Fix issues (if requested)

対応可能な項目がある場合:

1. Switch to the PR/MR's branch (git) or ensure files are open in the correct CL (Perforce) if not already.
2. Ask the user if they want to fix the issues.
3. If yes, make the fixes, then:

**GitHub/GitLab:** commit and push:
```bash
git add <files>
git commit -m "address review feedback"
git push
```

**Perforce:** open files for edit, make changes, and re-shelve:
```bash
p4 edit <file>
# make changes
p4 shelve -f -c <CL_NUMBER>
```

### 9. Resolve review threads

コメントに対応したら、対応するレビュースレッドを解決します。

**Perforce** - Perforce にはネイティブな「スレッド解決」の概念がありません。代わりに、CL の説明を更新するか、使用中のレビュー ツール（Swarm など）で返信してコメントに対応済みの印を付けます。`p4 review` を使用する場合:

```bash
# フィードバックに対応した後、ファイルをレビュー済みとして記録する
p4 review -c <CL_NUMBER>
```

**GitHub** - 未解決スレッドの ID を取得する（必要に応じてページネーションする。[GraphQL リファレンス](references/graphql-queries.md)を参照）:

```bash
gh api graphql -f query='
query($cursor: String) {
  repository(owner: "OWNER", name: "REPO") {
    pullRequest(number: PR_NUMBER) {
      reviewThreads(first: 100, after: $cursor) {
        pageInfo { hasNextPage endCursor }
        nodes {
          id
          isResolved
          comments(first: 1) {
            nodes { body path }
          }
        }
      }
    }
  }
}'
```

`hasNextPage` が true の場合は、`-f cursor=ENDCURSOR` を指定して残りのスレッドを取得する操作を繰り返します。

対応済みまたは情報提供のみのスレッドを解決します:

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "THREAD_ID"}) {
    thread { isResolved }
  }
}'
```

エイリアス（`t1`、`t2` など）を使い、複数の解決処理を 1 つの mutation にまとめます。

**GitLab** - 未解決のディスカッションを取得する（[GitLab API リファレンス](references/gitlab-api.md)を参照）:

```bash
glab api "projects/:fullpath/merge_requests/<MR_IID>/discussions?per_page=100"
```

`"resolved": false` であるディスカッションを絞り込み、各ディスカッションの `id` を収集します。

各ディスカッションを個別に解決します（GitLab には一括解決機能がありません）:

```bash
glab api --method PUT \
  "projects/:fullpath/merge_requests/<MR_IID>/discussions/<DISCUSSION_ID>" \
  --field resolved=true
```

未解決の discussion ID ごとに繰り返します。

### 10. Multiple PRs/MRs/CLs

PR/MR/CL のチェーンを確認する場合は、順番に処理します。

**Perforce** - to check multiple changelists at once:
```bash
p4 changes -s pending -u $P4USER -c $P4CLIENT -l
```

## 出力形式

次の内容を要約します:
- PR/MR/CL のタイトルまたは説明と現在の状態
- 検出したプラットフォーム（GitHub / GitLab / Perforce）
- ステータスチェックの概要（合格／失敗／保留）。Perforce の場合は N/A
- 見つかった問題の総数
- 説明付きの対応事項
- 無視できる項目とその理由
- 推奨する次のステップ
