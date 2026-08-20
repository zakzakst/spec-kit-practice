---
name: prcheckloop
description: >
  GitHub PR の latest-head の checks がすべて green になるか、具体的な
  blocker が明示されるまで反復します。review 修正後も check が失敗 / 保留の
  ままの PR、greploop 後の PR に使います。
---

# PRCheckloop

GitHub PR を完全に green な check 状態へ持っていくか、具体的な blocker を
示して終了します。

## 対象範囲

- GitHub PR のみ。repo が GitLab なら止めて `check-pr` を使ってください。
- 古い commit ではなく、最新の PR head SHA の check に注目します。
- review comment や PR template の整理ではなく、CI / status check に注目します。
- review comment の整理も必要なら、`check-pr` と組み合わせて使います。

## 入力

- **PR number**（任意）: 指定がなければ現在の branch から PR を検出します。
- **最大反復回数**: 既定値は `5`。

## ワークフロー

### 1. PR を特定する

If no PR number is provided, detect it from the current branch:

```bash
gh pr view --json number,headRefName,headRefOid,url,isDraft
```

必要なら、変更前に PR の branch へ切り替えます。

次のいずれかなら早めに終了します:

- `gh` is not authenticated
- there is no PR for the branch
- the repo is not hosted on GitHub

### 2. 最新の head SHA を追跡する

Always work against the current PR head SHA:

```bash
PR_JSON=$(gh pr view "$PR_NUMBER" --json number,headRefName,headRefOid,url)
HEAD_SHA=$(echo "$PR_JSON" | jq -r .headRefOid)
PR_URL=$(echo "$PR_JSON" | jq -r .url)
```

古い SHA の失敗 check は無視します。push のたびに `HEAD_SHA` を更新し、
確認ループを最初からやり直します。

### 3. その SHA の check を洗い出す

Fetch both GitHub check runs and legacy commit status contexts:

```bash
gh api "repos/{owner}/{repo}/commits/$HEAD_SHA/check-runs?per_page=100"
gh api "repos/{owner}/{repo}/commits/$HEAD_SHA/status"
```

PR 全体をコンパクトに見るには、次の GraphQL payload が便利です:

```bash
gh api graphql -f query='
query($owner:String!, $repo:String!, $pr:Int!) {
  repository(owner:$owner, name:$repo) {
    pullRequest(number:$pr) {
      headRefOid
      url
      statusCheckRollup {
        contexts(first:100) {
          nodes {
            __typename
            ... on CheckRun { name status conclusion detailsUrl workflowName }
            ... on StatusContext { context state targetUrl description }
          }
        }
      }
    }
  }
}' -F owner=OWNER -F repo=REPO -F pr="$PR_NUMBER"
```

### 4. check が実際に走るまで待つ

新しい push の直後は check が表示されるまで少し時間がかかることがあります。
15〜30 秒ごとに poll し、次のいずれかになるまで待ちます:

- checks have appeared and every item is in a terminal state
- checks have appeared and at least one failed
- no checks appear after a reasonable wait, usually 2 minutes

次は最終的な成功状態として扱います:

- check runs: `SUCCESS`, `NEUTRAL`, `SKIPPED`
- status contexts: `SUCCESS`

次は保留として扱います:

- check runs: `QUEUED`, `PENDING`, `WAITING`, `REQUESTED`, `IN_PROGRESS`
- status contexts: `PENDING`

次は失敗として扱います:

- check runs: `FAILURE`, `TIMED_OUT`, `CANCELLED`, `ACTION_REQUIRED`, `STARTUP_FAILURE`, `STALE`
- status contexts: `FAILURE`, `ERROR`

最新 SHA に対する check が一切出てこない場合は、`.github/workflows/`、
workflow の path filter、branch protection の期待値を確認します。足りない
check を repo から発生させたり修正したりできない場合は、エスカレーションします。

### 5. 失敗した check を調べる

GitHub Actions の失敗については、現在の SHA の run と失敗 log を確認します:

```bash
gh run list --commit "$HEAD_SHA" --json databaseId,workflowName,status,conclusion,url,headSha
gh run view <RUN_ID> --json databaseId,name,workflowName,status,conclusion,jobs,url,headSha
gh run view <RUN_ID> --log-failed
```

For each failing check, classify it:

| Failure type | Action |
|---|---|
| Code/test regression | Reproduce locally, fix, and verify |
| Lint/type/build mismatch | Run the matching local command from the workflow and fix it |
| Flake or transient infra issue | Rerun once if evidence supports flakiness |
| External service/status app failure | Escalate with the details URL and owner guess |
| Missing secret/permission/branch protection issue | Escalate immediately |

Only rerun a failed job once without code changes. Do not loop on reruns.

### 6. Fix actionable failures

If the failure is actionable from the checked-out code:

1. Read the workflow or failing command to identify the real gate.
2. Reproduce locally where reasonable.
3. Make the smallest correct fix.
4. Run focused verification first, then broader verification if needed.
5. Commit in a logical commit.
6. Push before re-checking the PR.

Do not stop at a local fix. The loop is only complete when the remote PR checks
for the new head SHA are green.

### 7. Push and repeat

After each fix:

```bash
git push
sleep 5
```

Then refresh the PR metadata, get the new `HEAD_SHA`, and restart from Step 3.

Exit the loop only when:

- all checks for the latest head SHA are green, or
- a blocker remains after reasonable repair effort, or
- the max iteration count is reached

### 8. Escalate blockers precisely

If you cannot get the PR green, report:

- PR URL
- latest head SHA
- exact failing or missing check names
- details URLs
- what you already tried
- why it is blocked
- who should likely unblock it
- the next concrete action

Good blocker examples:

- external status app outage
- missing GitHub secret or permission
- required check name mismatch in branch protection
- persistent flake after one rerun
- failure needs credentials or infrastructure access you do not have

## Output

When the skill completes, report:

- PR URL and branch
- final head SHA
- green/pending/failing check summary
- fixes made and verification run
- whether changes were pushed
- blocker summary if not fully green

## Notes

- This skill is intentionally narrower than `check-pr`: it is a repair loop for
  PR checks.
- This skill complements `greploop`: Greptile can be perfect while CI is still
  red.
