---
name: pr-gardening
description: >
  この Paperclip instance が開いた pull request（community contribution は除く）を
  見つけ、それぞれの目的と merge-ready かどうかの確度を報告し、
  ready でないものは `/prepare-paperclip-pr` で自動的に green に戻します。
  ただし merge はしません。
compatibility: Node.js 20 以上、GitHub 読み取りアクセスで認証済みの gh、Paperclip run credentials が必要です。
allowed-tools: Bash(node:*) Bash(gh:*) Bash(curl:*)
---

# PR の整理

最近の window（既定 14 日）で active な Paperclip issue に参照されている、**この Paperclip instance が開いた** pull request を能動的に世話します。candidate の発見と ready 判定は script で行い、LLM の分析ではありません。この workflow 中の GitHub access は read-only です。

## 対象範囲 - 自分たちの PR のみ

既定では、この workflow は instance の GitHub identity（`gh` の認証済み login、例: `cryppadotta`）が author の pull request だけを世話します。community contribution と dependabot PR は Stage A で author login により機械的に除外され、`droppedCommunityPullRequests` に入ります。手で再追加しないでください。scope を広げるのは、呼び出し側が明示的に `--authors` または `--include-community` を渡したときだけです。

## 厳格なガードレール

- **pull request を merge、approve、close しないこと。**
- **他人や agent に merge、approve、close するよう指示しないこと。**
- **community PR に対して gardening、comment、`/prepare-paperclip-pr` 実行をしないこと。** Stage A の author allowlist に入っている PR だけが対象です。
- mutating な `gh` command や GitHub API request は使わないこと。script は `gh pr view` と read-only の `gh api` GET request だけを使います。
- draft pull request は report-only です。draft に gardening comment を投稿しません。
- comment は既存の originating issue のみに行います。pull request ごとに gardening issue を
  新規作成しないでください。
- `--dry-run` は gardening comment、prepare task、inbox archive を含むすべての Paperclip mutation を抑止します。discovery と GitHub inspection はどの mode でも read-only のままです。

## 入力

- `--days <N>`: activity window。既定 `14`。mention された issue の activity と PR 自身の `updatedAt` の両方に適用され、window 内に activity がない open PR は stale として落とします。
- `--authors <logins>`: 対象にする PR の GitHub login をカンマ区切りで指定します。既定は `gh` の認証済み user です。
- `--include-community`: author filter を完全に無効化します。明示的な呼び出しがある場合のみ。
- `--repo <owner/repo>`: GitHub repository。既定は `gh repo view` で検出します。
- `--dry-run`: comment 投稿、prepare task 作成、inbox archive を行わずに discovery / verify / report だけを行います。
- `--archive-inbox`: GitHub が candidate PR を現在の head で merged と確認した後、Stage D で responsible user の inbox から originating issue を archive します。
- `--cooldown-hours <N>`: repeat-gardening の cooldown。既定 `48`。
- `--max-rounds <N>`: PR ごとの gardening round の最大回数。既定 `3`。

生成ファイルには、`$PAPERCLIP_RUN_SCRATCH_DIR/pr-gardening` のような run 所有ディレクトリを使います。

## Stage A - candidate を見つける

extract-search path を実行します。これは各 result page を走査し、PR URL を正規化し、PR number を重複排除し、参照された issue をすべて記録し、issue work product を確認して origin を特定し、GitHub 上で merged / closed とされた PR を落とし、author が allowlist 外（community contribution）の PR を `droppedCommunityPullRequests` に入れ、open PR でも自分自身の `updatedAt` が window 外なら `droppedStalePullRequests` に落とします。1 issue あたりの extract match cap を超えた issue（通常は大量の PR URL を列挙する digest や QA issue）は、run を止める代わりに `source.truncatedIssues` に記録し、report に注記します。

```bash
node .agents/skills/pr-gardening/scripts/find-candidates.mjs \
  --days 14 \
  --dry-run \
  --output "$RUN_DIR/candidates.json"
```

script は `GET /api/companies/:companyId/search/extract` を `kind=url`、`scope=all`、`updatedWithin=<N>d` で呼び出し、`--authors` または `--include-community` で上書きされない限り、author allowlist を `gh api user` から解決します。full issue-list fetching や LLM scanning に置き換えないでください。

## Stage B - 現在の head の ready 状態を確認する

```bash
node .agents/skills/pr-gardening/scripts/check-readiness.mjs \
  --input "$RUN_DIR/candidates.json" \
  --output "$RUN_DIR/readiness.json" \
  --dry-run
```

candidate ごとに、script は current head SHA を再取得し、次を記録します:

- open/draft state と mergeability/conflicts
- `statusCheckRollup` の check-run と legacy status inventory
- exact head 上で完了した Greptile check-run。`success` または `neutral` の場合のみ clean とみなします
- `reviewDecision`
- base branch との差分コミット数

判定は `ready`、`needs_gardening`、draft の場合は `report_only` です。wake 後や PR が修正されたという主張の後は、必ずこの stage を再実行してください。issue comment を ready の証拠として信じてはいけません。

## follow-up の Create-PR task 重複排除

gardening の結果、branch に単一の pull request を作る follow-up task が必要なら、
何かを作る前に重複排除します。

branch ごとに1つずつ、順番に処理します:

1. `backlog`、`todo`、`in_progress`、`in_review`、`blocked` の status を持つ open Paperclip issue から、branch 名が完全一致するものを検索します。
2. 一致した issue の title、description、最近の comment を確認し、同じ branch 用の同等な open「この branch から PR を作成する」task がないか調べます。
3. 同等の open task があれば再利用します。現在の PR、head、理由の context を簡潔な comment に記載し、gardening issue または blocker list からリンクします。別の task は作成しません。
4. 同等の open task がない場合に限り、その branch 用の follow-up task を1つだけ作成します。

follow-up task 作成を並列にばらまかないでください。create-PR または prepare-PR task に対して同時に `POST /api/companies/:companyId/issues` を投げてはいけません。P1 の issue-create idempotency support が使えるようになったら、すべての create-PR follow-up task 作成に `idempotencyKey: "pr-gardening:create-pr:{branch}"` を、すべての prepare-PR task に `idempotencyKey: "pr-gardening:prepare-pr:{owner/repo}#{number}"` を含めます。

## Stage C - `/prepare-paperclip-pr` で自分たちの PR を ready にする

`--dry-run` mode と、`ready` または `report_only` の entry ではこの stage を省略します。

ここに出てくる `needs_gardening` PR はすべてこの instance が開いたものです（Stage A が保証します）。
なので、報告するだけではなく、`/prepare-paperclip-pr` skill を使って実際に merge-ready にします。
PR は1つずつ処理します:

1. **Cooldown と round。** `candidates.json` から `originatingIssue` を特定します（選択優先順位は、正確な PR URL を `pull_request` work product として持つ issue、PR に言及する comment がある issue、最後に最も最近 active だった言及 issue の順）。その comment を取得し、次の marker を検索します:

   ```text
   <!-- pr-gardening:<owner/repo>#<number> -->
   ```

   一致する最新 marker が cooldown より新しければ、その PR はスキップします。
   matching marker から round を数え、3 round を超えたら止めて
   `not converging; recommend close or human decision` と報告します。
   これは human の判断材料であり、PR を閉じろという指示ではありません。

2. **重複排除。** open Paperclip issues から PR number / branch を探します。
   同等の open prepare-PR task がすでにあるなら、新しく作らず concise な status comment を
  付けて再利用します（上の重複排除セクションを参照）。

3. **prepare skill を実行する。** PR ごとに1つ、coder agent（できれば CodexCoder）に割り当てた
  対象を絞った child task を作り、その PR に対して `/prepare-paperclip-pr` を実行するよう指示します。
   PR URL、branch、current head SHA、`readiness.json` から得た machine-detected な
   `reasons[]` を含めます。gardener であるあなたがすでにその PR の branch を worktree で
   checkout しているなら、delegation せず直接 `/prepare-paperclip-pr` を実行しても構いません。
   どちらの場合でも、prepare work が PR を merge、approve、close してはいけません。

4. **marker comment を残す。** originating issue に、上の marker、current head SHA、
  コピーした `reasons[]`、round counter、prepare task への link を comment します。
  issue が terminal の場合は `resume: true` を含め、comment によって live continuation が作られるようにします。

Suggested body:

```markdown
<!-- pr-gardening:paperclipai/paperclip#1234 -->
Gardening: dispatched `/prepare-paperclip-pr` for https://github.com/paperclipai/paperclip/pull/1234 via PAP-XXXX.

Current-head verification at `abc123` found:
- failing check: test
- Greptile missing at current head

Gardening round 1/3. Re-verification is required after changes; do not merge based on this comment.
```

## Stage D - 任意の inbox tidy-up

この stage は、呼び出し側が明示的に `--archive-inbox` を指定したときだけ実行します。
`--dry-run` は `--archive-inbox` と一緒でも inbox mutation を常に抑止します。
flag がなければ、何も archive しません。

Stage D は、以前に監視していた candidate が merged に移行したときに適用されます。
この stage の直前に Stage B を再実行し、その verification で記録した同じ current head SHA について
GitHub が `state: merged` を返すことを要求します。新しい Stage A discovery は、
すでに merged / closed だった PR を意図的に落とします。

条件を満たす candidate ごとに、`POST /api/issues/:issueId/inbox-archive` と空の JSON body で
responsible user の inbox から originating issue を archive します。`userId` は渡しません。
Paperclip API が gardener の run context から responsible user を解決し、その user の
inbox-agent policy を強制します。GitHub access は read-only のままです。

単に `ready` なだけの PR、merge せずに閉じられた PR、draft、pending、または
未検証 / stale head で merged になった PR は archive しません。user が review、
decision、approval、その他の action を待っている間に originating issue を archive しないでください。
responsible user が未解決、inbox management が無効、gardener が allowlist 外、
または trust boundary があるなどの理由で Paperclip が mutation を拒否したら、
拒否を報告して policy を迂回せずに続行します。

archive 成功後は、archived issue と merged PR を明示する standard gardening marker comment を
originating issue に残します:

```markdown
<!-- pr-gardening:paperclipai/paperclip#1234 -->
Inbox tidy-up: archived [PAP-310](/PAP/issues/PAP-310) from the responsible user's inbox after confirming https://github.com/paperclipai/paperclip/pull/1234 merged at current head `abc123`.

This archive is audited and reversible; later issue activity may resurface the item.
```

`POST /api/issues/:issueId/comments` を使い、archive request と comment request の両方に
`X-Paperclip-Run-Id` を含めます。`--dry-run` では、書かれるはずだった archive と marker comment を
報告するだけで、どちらの mutation も実行しません。

## Stage E - 終了まで監視する

gardening run issue の `blockedByIssueIds` を、Stage C で関わっている非 terminal の prepare task と
originating issue に設定し、blocker 解消で gardener が wake されるようにします。
scheduled または manual rerun が fallback です。

wake のたびに、まず Stage B を再実行します。PR が active gardening から終了するのは、
次のいずれかが機械的に観測されたときだけです:

- 現在の head で `ready` が検証された;
- 外部で merged または closed になった;
- 最大 round 数に達し、収束しない状態として報告された。

terminal issue に gardening issue を blocked のままにしないでください。
agent や長時間セッションを poll しないでください。

## Stage F - レポートを生成して公開する

```bash
node .agents/skills/pr-gardening/scripts/render-report.mjs \
  --input "$RUN_DIR/readiness.json" \
  --output "$RUN_DIR/gardening-report.md"
```

report には scope（authors + window）を記載し、すべての open PR について、
author、PR description から取った1行の purpose summary、readiness confidence bucket を示します:

- **High:** current head の check が green、conflict なし、Greptile clean、base が最新、originating issue が terminal。
- **Medium:** その他は green だが base が古い、review が未完了、または originating issue が active。
- **Low:** check が failing/pending、Greptile がない、draft / 修正直後で未検証、または origin を特定できない。

PR の生成された purpose line が空または役に立たないなら、report を公開するときに
PR title と diff summary から1文で説明を書きます。

`candidates.json`、`readiness.json`、`gardening-report.md` を gardening issue に upload し、
Markdown body で `gardening-report` issue document を作成 / 更新し、artifact への link を
含む summary comment を残します。report が deliverable であり、merge の許可にはなりません。

## Verification

焦点を絞った script test を実行します:

```bash
node --test .agents/skills/pr-gardening/scripts/pr-gardening.test.mjs
```

live dry run では、Stages A、B、F を `--dry-run` で実行し、named PR はまだ open の場合にのみ
sanity-check します。merged / closed の例は readiness results ではなく
`droppedClosedPullRequests` に現れるべきで、community 作成 PR は
`droppedCommunityPullRequests` にしか現れてはいけません。
allowlist 外の author を持つ candidate や report entry は scope failure です。
`--archive-inbox` も試すなら、report に抑止された Stage D action が記載され、
Paperclip archive や marker-comment mutation が起きていないことを確認してください。