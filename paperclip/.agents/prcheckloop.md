---
name: prcheckloop
description: >
  GitHub PR の latest-head の checks がすべて green になるか、具体的な
  blocker が明示されるまで反復します。review 修正後も check が失敗 / 保留の
  ままの PR、greploop 後の PR に使います。
---

# PRCheckloop

GitHub PR の check をすべて green の状態にするか、具体的な blocker を示して終了します。

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

PR number が与えられていない場合は、現在の branch から検出します:

```bash
gh pr view --json number,headRefName,headRefOid,url,isDraft
```

必要なら、変更前に PR の branch へ切り替えます。

次のいずれかなら早めに終了します:

- `gh` が認証されていない
- その branch に PR がない
- repo が GitHub でホストされていない

### 2. 最新の head SHA を追跡する

常に現在の PR head SHA を対象にします:

```bash
PR_JSON=$(gh pr view "$PR_NUMBER" --json number,headRefName,headRefOid,url)
HEAD_SHA=$(echo "$PR_JSON" | jq -r .headRefOid)
PR_URL=$(echo "$PR_JSON" | jq -r .url)
```

古い SHA の失敗 check は無視します。push のたびに `HEAD_SHA` を更新し、確認ループを最初からやり直します。

### 3. その SHA の check を洗い出す

GitHub check run と legacy commit status context の両方を取得します:

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

### 4. check が実際に実行されるまで待つ

新しい push の直後は check が表示されるまで少し時間がかかることがあります。
15〜30 秒ごとに poll し、次のいずれかになるまで待ちます:

- check が表示され、すべての項目が最終状態になった
- check が表示され、少なくとも1つが失敗した
- 通常は2分程度の妥当な待機時間を過ぎても check が表示されない

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

失敗した各 check を分類します:

| 失敗の種類 | 対応 |
|---|---|
| コード / test の regression | ローカルで再現し、修正して検証する |
| lint / type / build の不一致 | workflow と一致するローカル command を実行して修正する |
| flaky または一時的な infra issue | flaky である証拠があれば1回だけ再実行する |
| 外部 service / status app の失敗 | details URL と担当者の候補を添えてエスカレーションする |
| secret / permission / branch protection の不足 | 直ちにエスカレーションする |

コード変更なしで失敗 job を再実行するのは1回だけにします。再実行をループさせてはいけません。

### 6. 対応可能な失敗を修正する

チェックアウトしたコードから対応できる失敗の場合:

1. workflow または失敗した command を読み、実際の gate を特定する。
2. 可能であればローカルで再現する。
3. 正しい最小限の修正を行う。
4. まず対象を絞った検証を実行し、必要に応じてより広い検証を行う。
5. 論理的にまとまった commit を作成する。
6. PR を再確認する前に push する。

Do not stop at a local fix. The loop is only complete when the remote PR checks
for the new head SHA are green.

### 7. push して繰り返す

修正するたびに:

```bash
git push
sleep 5
```

その後 PR metadata を更新し、新しい `HEAD_SHA` を取得して手順3から再開します。

次のいずれかになった場合だけ loop を終了します:

- 最新の head SHA に対するすべての check が green になった
- 妥当な修正を試みても blocker が残っている
- 最大反復回数に達した

### 8. blocker を正確にエスカレーションする

PR を green にできない場合は、次を報告します:

- PR URL
- 最新の head SHA
- 失敗または欠落している check の正確な名前
- details URL
- すでに試したこと
- blocker となっている理由
- blocker を解消すべき担当者またはチーム
- 次に行う具体的なアクション

適切な blocker の例:

- 外部 status app の障害
- GitHub secret または permission の不足
- branch protection における required check 名の不一致
- 1回再実行しても継続する flaky failure
- 保有していない認証情報またはインフラへのアクセスが必要な失敗

## Output

スキルの完了時に、次を報告します:

- PR URL と branch
- 最終的な head SHA
- green / pending / failing となった check の概要
- 実施した修正と検証内容
- 変更を push したかどうか
- 完全に green でない場合は blocker の概要

## Notes

- このスキルは意図的に `check-pr` より対象を絞っています。PR check の修復ループです。
- このスキルは `greploop` を補完します。Greptile が完全に問題なくても、CI が red のままの場合があります。
