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
対象に、review comment、status check、description の完成度を調べ、見つかった問題の
対応を手伝います。

## 入力

- **PR/MR/CL number**（任意）: 指定がなければ、現在の branch の PR/MR、または p4 の既定 pending changelist を検出します。

## 手順

### 0. プラットフォームを判定する

まず、`.p4config` ファイルや `P4CLIENT` / `P4PORT` 環境変数を見て、ユーザーが Perforce depot で作業しているか確認します:

```bash
# Check for Perforce environment
if p4 info >/dev/null 2>&1; then
  VCS="perforce"
else
  # Fall back to git remote detection
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
# List pending changelists for the current user/client
p4 changes -s pending -u $P4USER -c $P4CLIENT
```

プラットフォームごとの主要な field の違い:
- GitHub: `number`, `headRefName`, `headRefOid`
- GitLab: `iid`, `source_branch`, `sha`
- Perforce: changelist number (CL), `shelved` files for in-review CLs

### 2. PR/MR/CL の詳細を取得する

**GitHub:**
```bash
gh pr view <PR_NUMBER> --json title,body,state,reviews,comments,headRefName,headRefOid,statusCheckRollup
OWNER_REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
gh api "repos/$OWNER_REPO/pulls/<PR_NUMBER>/comments"
gh api --paginate "repos/$OWNER_REPO/issues/<PR_NUMBER>/comments?per_page=100"
```

GitHub の PR は issue でもあるため、一般的な PR コメントは issue comments endpoint にあります。Greptile は、毎回新しい review や comment を作る代わりに、一般コメント1件を更新することがあります。PR が問題ないと判断する前に、`updated_at` で並べた最新の Greptile 作成 general comment を、"Prompt to fix all with AI" section を含めて必ず確認してください。

**GitLab:**
```bash
glab mr view <MR_IID> --output json
# Fetch discussions (inline diff comments are type "DiffNote"; general comments have null type)
glab api "projects/:fullpath/merge_requests/<MR_IID>/discussions"
```

GitLab では必要に応じて discussion を paginate してください（`?per_page=100&page=N` を付けます）。

**Perforce:**
```bash
# Get changelist description, files, and status
p4 describe -s <CL_NUMBER>

# Get shelved files (for in-review CLs)
p4 describe -S <CL_NUMBER>

# Get the diff of the shelved changelist
p4 diff2 //...@=<CL_NUMBER> //...@=<CL_NUMBER>

# List review comments (if using p4 review workflow)
p4 review -c <CL_NUMBER>
```

Perforce CL の主要 field:
- `Change`: changelist number
- `Status`: `pending`, `submitted`, `shelved`
- `Description`: the CL description / commit message
- `Files`: list of files in the CL

### 3. 保留中の check を待つ

分析する前に、すべての status check が完了していることを確認します。GitHub で `PENDING` / `IN_PROGRESS`、GitLab で `running` / `pending` の check があれば、すべてが終端状態になるまで 30 秒ごとに poll します。

**GitHub:** `gh pr view` の `statusCheckRollup` を poll します。

**GitLab:**
```bash
glab api "projects/:fullpath/merge_requests/<MR_IID>/pipelines"
```
Pipeline statuses: `running`, `pending`, `success`, `failed`, `canceled`, `skipped`. Poll until no pipeline has `running` or `pending` status.

**Perforce:** Perforce doesn't have built-in CI checks natively. If the team uses a review tool (Swarm, etc.) or an external CI triggered by shelve events, check the relevant system. Otherwise, proceed to analysis immediately.

### 4. 現在の head に対する新しい Greptile review を要求する

GitHub PR では、既存の Greptile review、comment、summary を、PR の現在の `headRefOid` に結びついていない限り current とみなさないでください。これは既存 PR に新しい commit を push した直後に特に重要です。古い commit に対する Greptile review は stale であり、PR に Greptile comment や過去の review が残っていても無効です。

Greptile gate の直前に、current head SHA を取得します:

```bash
HEAD_SHA=$(gh pr view <PR_NUMBER> --json headRefOid -q .headRefOid)
```

その commit の check-run を調べ、完了済みの Greptile run を要求します:

```bash
OWNER_REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
GREPTILE_CHECKS=$(gh api "repos/$OWNER_REPO/commits/$HEAD_SHA/check-runs?per_page=100" \
  --jq '[.check_runs[] | select(.name | test("greptile"; "i"))]')

# run が有効な fresh pass とみなされるのは、完了していて、かつ clean に終わっている場合だけです。
# GitHub check-run の conclusion: success, neutral, skipped, failure, timed_out,
# cancelled, action_required, stale。success / neutral だけを clean とみなし、
# それ以外（特に failure と action_required）はすべて block します。
FRESH_GREPTILE_COMPLETED=$(echo "$GREPTILE_CHECKS" \
  | jq '[.[] | select(.status == "completed")] | length')
FRESH_GREPTILE_CLEAN=$(echo "$GREPTILE_CHECKS" \
  | jq '[.[] | select(.status == "completed" and (.conclusion | IN("success","neutral")))] | length')
FRESH_GREPTILE_BLOCKING=$(echo "$GREPTILE_CHECKS" \
  | jq '[.[] | select(.status == "completed" and ((.conclusion | IN("success","neutral")) | not))] | length')

if [ "$FRESH_GREPTILE_COMPLETED" = "0" ]; then
  echo "Blocked: current PR head $HEAD_SHA に結びついた完了済み Greptile review/check がありません。"
  echo "PR check を done にする前に、この head に対する新しい Greptile review を依頼してください。"
  echo "Suggested trigger: gh pr comment <PR_NUMBER> --body \"@greptile review\""
  exit 1
fi

if [ "$FRESH_GREPTILE_BLOCKING" != "0" ] || [ "$FRESH_GREPTILE_CLEAN" = "0" ]; then
  echo "Blocked: Greptile は head $HEAD_SHA で完了しましたが、clean に終わっていません。"
  echo "(conclusion が failure/action_required/timed_out/cancelled/stale だったか、clean run が存在しません。)"
  echo "指摘に対応して push し、新しい head で success / clean に終わるまで Greptile を再実行してください。"
  exit 1
fi
```

current head に対する Greptile check が存在するが still pending / in progress の場合は、古い review material に進まず、`greploop` と同じ polling pattern で待ちます。reasonable wait の後でも current head に対する Greptile check が出てこない場合は、`HEAD_SHA` に対する新しい Greptile review 待ちとして PR を blocked にし、そこで止めます。current head SHA に結びつけられない PR comment、PR review、Greptile summary から check complete だとみなしてはいけません。

Greptile integration がある GitLab 環境でも、MR の current head SHA に対して同じ freshness rule を適用します:

```bash
# 1. Get the MR's current head SHA
MR_SHA=$(glab mr view <MR_IID> --output json | jq -r '.sha // .diff_refs.head_sha')

# 2. Find the latest pipeline for that EXACT sha, then the Greptile job within it.
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

# 3. If Greptile integrates via MR notes instead of a CI job, require the newest
#    Greptile note to reference the current head sha and report a clean review.
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

Block completion if, for `MR_SHA`, there is (a) no Greptile job or note at all (missing), (b) the newest Greptile job/note is tied to a different sha (stale), or (c) the Greptile job status is not `success`. A completed Greptile result for a different SHA is stale and must block completion.

For Perforce installations with Greptile integration, apply the same rule using the CL's current shelved-revision identity and the Greptile webhook/review artifact tied to it; a Greptile result for an earlier shelf is stale and must block completion.

### 5. Analyze the PR/MR

Once all checks are complete, evaluate these areas:

#### A. Status Checks

- Are all CI checks passing?
- If any are failing, identify which ones and the failure reason.

#### B. PR/MR Description

- Is the description complete and follows team conventions?
- Are all required sections filled in?
- Are there TODOs or placeholders that need updating?

#### C. Review Comments

- Inline code review comments that need addressing
- Look for bot review comments (e.g. from `greptile-apps[bot]` on GitHub, or the Greptile bot user on GitLab, linters, etc.)
- Human reviewer comments
- **Perforce:** review comments from `p4 review` or external review tools

#### D. General Comments

- Discussion comments on the PR/MR
- For GitHub, check the issue comments endpoint and use `updated_at` to catch bot comments edited in place. Greptile's latest edited summary can contain actionable items even when there are no new inline comments.
- Bot comments (deploy previews, etc.) - usually informational
- **Perforce:** CL description should include a clear summary, affected files rationale, and testing notes

### 6. Categorize issues

For each issue found, categorize as:

| Category | Meaning |
|---|---|
| **Actionable** | Code changes, test improvements, or fixes needed |
| **Informational** | Verification notes, questions, or FYIs that don't require changes |
| **Already addressed** | Issues that appear to be resolved by subsequent commits |

### 7. Report findings

Present a summary table:

| Area | Issue | Status | Action Needed |
|------|-------|--------|---------------|
| Status Checks | CI build failing | Failing | Fix type error in `src/api.ts` |
| Review | "Add null check" - @reviewer | Actionable | Add guard clause |
| Description | TODO placeholder in test plan | Actionable | Fill in test plan |
| Review | "Looks good" - @teammate | Informational | None |

### 8. Fix issues (if requested)

If there are actionable items:

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

After addressing comments, resolve the corresponding review threads.

**Perforce** - Perforce does not have a native "resolve thread" concept. Instead, mark comments as addressed by updating the CL description or by responding in the review tool being used (Swarm, etc.). If using `p4 review`:

```bash
# Mark files as reviewed after addressing feedback
p4 review -c <CL_NUMBER>
```

**GitHub** - fetch unresolved thread IDs (paginate if needed - see [the GraphQL reference](references/graphql-queries.md)):

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

If `hasNextPage` is true, repeat with `-f cursor=ENDCURSOR` to get remaining threads.

Then resolve threads that have been addressed or are informational:

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "THREAD_ID"}) {
    thread { isResolved }
  }
}'
```

Batch multiple resolutions into a single mutation using aliases (`t1`, `t2`, etc.).

**GitLab** - fetch unresolved discussions (see [the GitLab API reference](references/gitlab-api.md)):

```bash
glab api "projects/:fullpath/merge_requests/<MR_IID>/discussions?per_page=100"
```

Filter for discussions where `"resolved": false`. Collect each discussion's `id`.

Resolve each discussion individually (GitLab has no batch resolution):

```bash
glab api --method PUT \
  "projects/:fullpath/merge_requests/<MR_IID>/discussions/<DISCUSSION_ID>" \
  --field resolved=true
```

Repeat for each unresolved discussion ID.

### 10. Multiple PRs/MRs/CLs

If checking a chain of PRs/MRs/CLs, process them sequentially.

**Perforce** - to check multiple changelists at once:
```bash
p4 changes -s pending -u $P4USER -c $P4CLIENT -l
```

## Output format

Summarize:
- PR/MR/CL title or description and current state
- Platform detected (GitHub / GitLab / Perforce)
- Status checks summary (passing/failing/pending) - or N/A for Perforce
- Total issues found
- Actionable items with descriptions
- Items that can be ignored with reasons
- Recommended next steps
