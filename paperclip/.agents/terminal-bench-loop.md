---
name: terminal-bench-loop
description: >
  1つの Terminal-Bench task を、上限付きの Paperclip smoke / diagnosis / fix loop
  に通します。Terminal-Bench が通るまで回す、Terminal-Bench loop を再実行する、
  board gate 付きの修正と診断を繰り返す、といった依頼に使います。
---

# Terminal-Bench Loop

Paperclip を使って 1つの Terminal-Bench 問題を smoke pass に到達させるための、再現可能な運用 skill です。issue topology を明示し、run に上限を設け、board gate で product fix を管理し、worktree の継続性を保ちます。

この skill は engineering ではなく、**運用 + 診断** のためのものです。Terminal-Bench loop に関する issue、artifact、approval を調整します。コード変更を許可するものではありません。承認された product fix は、board confirmation の後に別の implementation child issue として着地します。

正規の実行モデル: loop を始める前、または loop issue を動かす前に `doc/execution-semantics.md` を読んでください。すべての loop issue は、その document が許す state に置く必要があります: terminal（`done` / `cancelled`）、明示的な live（active run / queued wake）、明示的な waiting（participant / interaction / approval を伴う `in_review`）、または明示的な recovery / blocker（`blockedByIssueIds` と named owner を伴う `blocked`）です。

## 使う場面

次のいずれかの title または body に一致する assignment で使います:

- "run Terminal-Bench in a loop", "loop \<task-name\> through Paperclip"
- "drive Terminal-Bench fix-git", "iterate on Terminal-Bench until it passes"
- "Terminal-Bench smoke loop", "bench loop", "smoke loop on \<task-name\>"
- Terminal-Bench loop の parent issue へのリンクが添えられ、さらに次の iteration を求められているもの

また、既存の top-level loop issue を渡され、次の iteration、diagnosis、rerun を求められたときにも使います。

## 使わない場面

- assignment が `paperclip-bench` 自体（Harbor adapter、wrapper、telemetry）の作成や変更である場合。そちらは通常の engineering flow で進めます。
- assignment がランキング用の benchmark result 提出である場合。この skill は設計上 smoke / non-comparable run を出すものであり、full-suite や comparable run は BenchmarkQualityManager にエスカレーションします。
- assignment が Terminal-Bench loop から表面化していない通常の Paperclip product bug である場合。通常の investigation を使います。
- company skill の install や assign の権限がなく、相手が実際には library mutation を求めている場合。その step は権限のある skill-library owner に任せます。

## 守るべき3つの不変条件

すべての loop iteration と提案する product fix は、次の3つの不変条件を同時に満たす必要があります。これらは `/diagnose-why-work-stopped` に由来し、ユーザーは liveness work で繰り返し示しています:

1. **生産的な作業を継続する。** 各 loop issue には常に、agent、board、user、または名前付き blocker のいずれかによる明確な次のアクション担当者が必要です。待機対象がないまま `in_review` にする、暗黙の停止は禁止です。
2. **実際の blocker だけが作業を停止させる。** 停止は、何かが本当に進行不能な場合（board confirmation、QA、認証情報不足、予算枯渇）に発生します。見かけ上の停止は検出して適切な経路へ送ります。
3. **無限ループを作らない。** iteration 数、実時間予算、product fix 適用前の board gate によって loop を上限内に保ちます。

提案した iteration が3つのどれかに反するなら、捨てるか作り直します。loop issue に、
その iteration で各 invariant をどう守るかを明記してください。

## 入力

iteration 1 の前に、これらを top-level loop issue に記録します。提供できない入力がある場合は blocker とし、解除担当者を明記して停止します。

- **Source issue。** loop を依頼した Paperclip issue。loop parent からこの issue にリンクします。
- **Terminal-Bench task name。** 単一 task の識別子（例: `terminal-bench/fix-git`）。複数 task の suite はこの skill の対象外です。
- **Iteration budget。** 追加の fix なしで loop を停止するまでの iteration 最大数（通常は 3～5）。iteration ごとの実時間上限も記録します。
- **Paperclip App worktree issue。** 分離 worktree を所有する execution workspace を持つ、Paperclip App project 配下の実装側 issue。初回 iteration で作成し、後続 iteration では `inheritExecutionWorkspaceFromIssueId` または同等の仕組みで再利用します。
- **Benchmark command。** テスト対象の Paperclip App worktree に固定した `PAPERCLIPAI_CMD`（または同等の command binding）を含む、正確な `paperclip-bench` 呼び出し。loop issue にそのまま記録します。
- **Dispatch runner config。** smoke で Paperclip heartbeat を実際に開始するために必要な Harbor/Paperclip runner dispatch 設定。現在の Harbor wrapper では、`assignee`、`heartbeat_strategy`、`agent_adapter` / `agent_adapters`、ローカル認証情報が意図的に必要な場合の `reuse_host_home`、停止予算を保持できる形で `PAPERCLIP_HARBOR_RUNNER_CONFIG` JSON（または同等の設定ファイル）を記録します。`BEN-1` を未割り当ての `todo` として作成し、heartbeat 対応 agent が 0 のままになる Harbor コマンド単体は harness/setup failure であり、有効な product diagnosis ではありません。
- **Latest artifact root。** `paperclip-bench` が run artifact（manifest、`results.jsonl`、Harbor raw job folder、redacted telemetry）を書き込む filesystem または storage のパス。iteration ごとに追記し、上書きはしません。
- **Approval policy。** 実装前に提案された product fix を承認する必要がある者（デフォルトは `request_confirmation` 経由の board、委任された場合は CTO。loop driver 単独での承認は不可）。

各入力を top-level loop issue（description または専用の `inputs` document）に記録します。loop の途中で入力が変わった場合は、変更内容と適用された iteration を記録します。

## issue topology

loop はコメント内の文章ではなく、ツリーとして表現できなければなりません:

- **Top-level loop issue。** 長期間存続します。入力、iteration counter、現在の state、すべての iteration child へのリンク、product rule の履歴を保持します。iteration 実行中は `in_progress`、loop parent に typed waiter（execution-policy participant、`request_confirmation` / `ask_user_questions` / `suggest_tasks` interaction、approval、または named human owner）が直接存在する場合のみ `in_review`、child issue が gating work の場合（fix-proposal の `request_confirmation` を持つ iteration child、または implementation、QA、CTO review child）は `blockedByIssueIds` 付きの `blocked`、pass 時は `done`、board rejection / budget exhaustion 時は `cancelled` にします。
- **Iteration child issue。** iteration ごとに 1 つ作成します。それぞれに、上限付きの run issue（smoke）、diagnosis issue（`/diagnose-why-work-stopped` を適用）、`request_confirmation` interaction 付きの fix-proposal document、そして承認後に限り implementation、QA、CTO review、rerun child を持たせます。executor が順番どおりに起動できるよう、iteration child は前の iteration に依存させます。
- **Paperclip App implementation issue。** 初回 iteration では、project policy により分離 worktree が作成される新しい Paperclip App child を作ります。後続 iteration の implementation/rerun child は `inheritExecutionWorkspaceFromIssueId` 経由で同じ execution workspace を参照し、同じ worktree を修正してテストします。

依存関係は「blocked by X」のような文章ではなく、`blockedByIssueIds` で接続します。依存先の child が `done` になると、executor が次の child を自動的に起動します。

## 手順

### 0. 現在の execution contract を読む

loop を開くか進める前に `doc/execution-semantics.md` を読みます。loop issue の state を分類するときは、この文書の用語をそのまま使います: live path / waiting path / recovery path、post-run disposition、bounded continuation、productivity review、pause-hold、watchdog。新しい state を作ってはいけません。

### 1. top-level loop issue を開くか再利用する

- 既存の loop issue が渡された場合は、inputs、iteration counter、前回 iteration の stop reason、現在の Paperclip App worktree pointer、最新の benchmark command を読みます。
- loop issue がない場合は、Paperclip App project（または source issue が指す project）配下に作成します。タイトルは `Terminal-Bench loop: <task-name>` とし、description に上記の入力、iteration budget、source issue へのリンクを記録します。
- worktree pointer が引き続き解決できることを確認します。記録された execution workspace が破棄されている場合（worktree の prune、project の変更など）は、解除担当者（CodexCoder または Paperclip App owner）を明記して loop を blocked にし、停止します。

### 2. iteration child を開く

- loop issue の iteration counter を増やします。
- `Iteration N: <task-name>` というタイトルの iteration child を作成します。description には入力を再掲し、loop parent を参照します。executor が 2 つの iteration を並行して開始できないよう、前回 iteration の terminal child（存在する場合）に依存させます。
- iteration counter が budget を超える場合は child を作成しません。loop issue を `cancelled`（budget exhausted）にするか、budget 延長の判断をユーザーに求める場合は `in_review` にします。

### 3. 上限付き smoke を実行する

- benchmark command はテスト対象の Paperclip App worktree を使わなければなりません。`PAPERCLIPAI_CMD`（または同等の command binding）を、その worktree 内の CLI entrypoint に設定します。operator の現在の Paperclip checkout に対して smoke を実行してはいけません。
- 同じ command block に、benchmark issue を実行可能にする runner dispatch config も含めます。現在の Harbor wrapper では、意図した assignee、heartbeat strategy、agent adapter、credential/home mode、stop budget を含む `PAPERCLIP_HARBOR_RUNNER_CONFIG` を export します。dispatch config を省略した `uvx harbor run ...` 単体を canonical smoke として扱わず、harness/setup miss として記録し、記録済み config で再実行します。
- wall-clock と Paperclip の run-budget controls の両方で run に上限を設けます。smoke が iteration ごとの上限を超えそうな場合は kill し、打ち切り理由を記録します。
- iteration child または専用の `run` document に次を記録します:
  - Paperclip run id と heartbeat run ids
  - benchmark run id、manifest、`results.jsonl` の行、Harbor raw job folder
  - 使用した dispatch config（`PAPERCLIP_HARBOR_RUNNER_CONFIG` または同等のもの）。assignee と adapter type を含めます
  - harness が報告した正確な stop reason（pass、harness fail、verifier fail、timeout、agent gave up、infrastructure error）
  - Paperclip telemetry が出力する場合は、heartbeat-enabled agent 数と heartbeat-observed agent 数
  - failure taxonomy bucket（task/model、Paperclip product、harness/setup、verifier/infrastructure、security、unclear）
  - latest artifact root 配下の artifact path
- iteration には **smoke / non-comparable** のラベルを付けます。comparable run はこの skill の対象外です。

### 4. 正確な停止点を診断する

この loop の範囲だけを対象として、iteration の run に `/diagnose-why-work-stopped` パターンを適用します。無関係な forensic boilerplate は持ち込まないでください。具体的には:

- Paperclip App worktree 配下で smoke が生成した Paperclip issue tree を node ごとにたどり、進行を停止させた正確な `(issue, status)` の組み合わせを見つけます。run id、comment timestamp、status transition の証拠を引用します。
- subtree 内の進行していない issue をすべて、**本当に human/board の介入が必要**、**agent が実行可能だが現在 route されていない**、**すでに対応済み** のいずれかに分類します。
- failure が task/model、Paperclip product、harness/setup、verifier/infrastructure、security、unclear のどれかを明示します。証拠から推論した場合（例: cross-company API boundary により直接読み取れない場合）は、そのことも明記します。
- failure が Paperclip product gap の場合は、fix を contract として表現した **一般的な product rule** とし、上記 3 つの不変条件で確認します。その rule が最近の productive run を妨げていた場合は、適用範囲を狭めます。

診断結果を iteration child 上の `diagnosis` document に記録します。ここではまだコードを提案しません。

### 5. 次の動きを決める

diagnosis に基づき、iteration は次の iteration 用 terminal state のいずれか 1 つで終了します:

- **Pass。** Smoke verifier が pass を報告した場合。iteration child と loop parent を QA/CTO review に進めます（Step 8）。
- **Product fix proposed。** Paperclip product gap が特定された場合。fix proposal を iteration child 上の `plan` document に書き、Step 6 に進みます。
- **Non-product failure with retry。** failure が harness/setup/infrastructure または model flakiness で、iteration budget が尽きておらず、コード変更なしの rerun に意味があると loop driver が判断する場合（例: 一時的な infra 障害）。iteration child に理由を記録し、implementation step なしで Step 7 に進みます。
- **Real blocker。** 名前付きの外部 blocker（credentials、quota、third-party outage、security review）がある場合。loop issue を `blocked` に移し、`blockedByIssueIds` に blocker issue を設定し（必要なら作成）、解除担当者を明記します。停止します。
- **Budget or board stop。** iteration budget に達した場合、または board が次の fix proposal を拒否した場合。run history と停止理由を要約した comment を付け、loop issue を `cancelled` に移します。

### 6. product fix 前に board confirmation を要求する

iteration が **product fix proposed** で終了した場合:

- iteration child の `plan` document を、提案する contract、3 つの不変条件の確認、影響を受ける Paperclip surface、段階的な subtask（implementation、QA、CTO review、rerun）で更新します。ただし、これらの subtask はまだ作成しません。
- **iteration child**（`plan` document を所有する同じ issue）に `request_confirmation` interaction を開き、最新の plan revision を対象にします。Idempotency key は `confirmation:{iterationIssueId}:plan:{revisionId}`。`continuationPolicy` は `wake_assignee` に設定します。
- **iteration child** を `in_review` に移します。typed waiter である `request_confirmation` interaction が直接存在するため、この `in_review` は正常です。comment から plan document にリンクし、保留中の confirmation を明記します。
- **loop parent** を `blockedByIssueIds: [iterationChildId]` 付きの `blocked` に移し、board（または approval policy が指定する approver）を解除担当者として明記した comment を投稿します。ここで loop parent を `in_review` にしてはいけません。typed waiter は parent ではなく iteration child に存在するため、parent の wait path は child blocker です。これは typed waiter が parent に直接付いている場合のみ loop parent を `in_review` にする topology rule に一致します。
- 承認を待ちます。board が plan を変更する superseding comment を投稿した場合は document を修正し、iteration child 上で新しい revision に紐づく新しい confirmation を開きます。以前の confirmation は無効になります。loop parent の `blockedByIssueIds` はすでに iteration child を指しているため、変更は不要です。
- 拒否された場合は **Budget or board stop** のルールに従って loop を終了します。同じ proposal を黙って再試行してはいけません。
- 承認された場合は、`blockedByIssueIds` を順番どおりに接続した implementation、QA、CTO review、rerun child issue を作成します。loop parent の `blockedByIssueIds` は新しい gating child（通常は implementation child）を指すように更新し、実際の downstream work に対して parent を `blocked` に保ちます。implementation child は Paperclip App execution workspace を継承し（worktree-owning issue への `inheritExecutionWorkspaceFromIssueId`）、smoke を実行したのと同じ分離 worktree に fix を適用しなければなりません。

### 7. 同じ worktree に対して rerun する

implementation と QA が完了した後（または **non-product failure with retry** の場合は直ちに）、rerun child が同じ `paperclip-bench` invocation を実行します。このときも `PAPERCLIPAI_CMD` はテスト対象の Paperclip App worktree に固定します。

- rerun は fix を適用したのと同じ worktree を使わなければなりません。iteration 間に workspace が reset されていた場合、loop は無効です。loop issue に blocker を開いて停止します。
- 完了時、rerun child が次の iteration の run record になります。smoke が pass した場合は Step 8 に進みます。それ以外は iteration budget の範囲内で新しい iteration child を作成し、Step 4 に戻ります。

### 8. Pass: QA, CTO review, close

smoke が pass した場合:

- 依存 chain にまだ存在しない場合は QA と CTO review child を作成します（CTO review は QA に blocked とし、chain を順番どおりに起動します）。loop parent の `blockedByIssueIds` を QA / CTO review chain に設定して `blocked` に移し、QA と CTO を解除担当者として明記し、child にリンクする comment を投稿します。typed waiter は parent ではなく child に存在するため、loop parent は `in_review` ではなく `blocked` のままにします。
- この段階で loop parent 自体を `in_review` にしたい場合（例: board user が review の実施を明示的に引き受けた場合）は、execution-policy participant、`request_confirmation` / `ask_user_questions` / `suggest_tasks` interaction、approval、または named human owner という typed waiter を parent に直接設定します。child chain だけに依存してはいけません。parent の `in_review` と QA/CTO child による blocker を組み合わせてはいけません。この skill は、その曖昧な review 状態を防ぐために存在します。
- QA は artifact（manifest、`results.jsonl`、Harbor raw job、redacted telemetry）と、同じ worktree での rerun の再現性を検証します。
- CTO は loop 中に適用された product fix の技術的な範囲を review します。
- QA と CTO が承認したら、task name、iteration count、stop reason（pass）、worktree pointer、最終 artifact root へのリンク、承認された product fix の一覧（それぞれの implementation issue id を含む）を記載した board-level summary comment で loop issue を終了します。

### 9. 停止ルール

次のいずれかに該当する場合、loop issue に state を明示的に記録して loop を **必ず** 停止します:

- **Pass。** Smoke verifier が pass を報告し、QA と CTO が承認した場合（Step 8）。Loop issue → `done`。
- **Board rejection。** Board が fix proposal を拒否し、revision を求めない場合。Loop issue → `cancelled`。comment に拒否された proposal と理由を記載します。
- **Iteration budget reached。** pass しないまま iteration counter が budget に達した場合。Loop issue → `cancelled`（budget 延長の判断をユーザーに求める場合は `in_review`）。iteration N+1 を黙って開始してはいけません。
- **Real blocker named。** 外部 blocker（credentials、quota、infra、security、missing skill）を loop driver が解決できない場合。blocker issue を `blockedByIssueIds` に設定し、解除担当者を明記して Loop issue → `blocked`。

loop は文章 comment だけで終了させてはいけません。すべての停止には、名前付きの次のアクション担当者を伴う status transition が必要です。

## Worktree rule

loop は heartbeat に使われている現在の Paperclip checkout を無作為にテストしてはいけません。提案した fix を適用するのと同じ分離 Paperclip App worktree をテストしなければなりません。

- 初回 iteration で Paperclip App implementation child を作成します。その project の git-worktree policy により、新しい worktree が作成されます。
- loop issue に worktree-owning issue id と workspace path（または workspace id）を記録します。
- 後続の implementation、QA、rerun child はすべて `inheritExecutionWorkspaceFromIssueId` にその worktree-owning issue を設定し、以降の loop 作業で同じ workspace を共有します。
- benchmark command は常に `PAPERCLIPAI_CMD`（または同等の command binding）をその worktree 内の CLI entrypoint に設定し、benchmark issue を割り当て heartbeat を開始するために記録済みの dispatch runner config（`PAPERCLIP_HARBOR_RUNNER_CONFIG` または同等のもの）を含めます。loop issue に保存した benchmark command が source of truth です。別の shell から smoke を実行する必要がある場合は、Harbor invocation の行だけでなく、記録済みの command block 全体をそのままコピーします。
- workspace が prune された場合、または worktree path を解決できなくなった場合、再構築するまで loop は無効です。loop を `blocked` にし、解除担当者（通常は CodexCoder または Paperclip App owner）を明記します。

## Liveness rule

すべての heartbeat の終了時に、各 loop issue は必ず次のいずれかの状態になっていなければなりません:

- **Terminal:** `done` または `cancelled`。追加の action はありません。
- **明示的な live:** active run、今後の queued wake、または配下で実行中の child issue を伴う `in_progress`。
- **明示的な waiting:** typed waiter（execution-policy participant、`request_confirmation` / `ask_user_questions` / `suggest_tasks` interaction、approval、または named human owner）を伴う `in_review`。
- **明示的な recovery / blocker:** 実際の blocking issue を `blockedByIssueIds` に設定し、解除担当者と必要な action を comment に記載した `blocked`。

終了時に loop issue がこれらのいずれにも当てはまらない場合、heartbeat は完了していません。終了前に state を修正します。

## Pitfalls

- **operator の Paperclip checkout に対して smoke を実行する。** worktree rule の目的は、bench が fix を適用する worktree をテストすることです。run の開始前に必ず `PAPERCLIPAI_CMD` を設定し、path を確認します。
- **dispatch config を落とす。** `PAPERCLIP_HARBOR_RUNNER_CONFIG`（または同等のもの）を省略した Harbor run は Paperclip を起動して `BEN-1` を作成できても、未割り当てで heartbeat-enabled agent が 0 のままになることがあります。これは Terminal-Bench の product signal ではありません。assignee と adapter config を含む command block 全体を保持して再実行します。
- **承認前に coding する。** board confirmation が iteration の `plan` document を承認するまで、implementation child は存在しません。diagnostic phase でコードを push してはいけません。
- **最近の作業調査を省略する。** Paperclip product rule を提案するときは、影響を受ける liveness/execution 領域で直近数日以内に何が出荷されたかを確認します。先週承認された contract と矛盾する rule は手戻りになります。
- **`in_review` を done とみなす。** participant、interaction、approval、human owner のいずれもないまま loop または iteration child が `in_review` にある場合、それは進行ではなく停止です。liveness violation として扱い、適切な経路へ送ります。
- **iteration N+1 を黙って開始する。** iteration budget に達した場合、loop issue に明示的な budget extension を記録せず、別の iteration を開始してはいけません。
- **Comparable-run drift。** この skill が生成するのは smoke run のみです。依頼者が comparable benchmark submission を求める場合は BenchmarkQualityManager と BenchmarkForensics に引き継ぎ、smoke を comparable と再分類してはいけません。
- **再帰的 recovery。** 自身の recovery issue を recovery する stranded-work recovery は典型的な無限ループです。smoke の subtree 内で診断に現れた場合は深掘りを拒否し、product-rule fix のため `/diagnose-why-work-stopped` に送ります。
- **Skill-library mutation。** この skill は loop iteration の一環として company skill を install、編集、assign しません。library の変更は別 issue を通じて、権限のある skill-library owner に委ねます。
- **chain を隠す。** 失敗した iteration child、撤回した proposal、拒否された confirmation を黙って削除または隠してはいけません。audit trail が loop の証拠です。

## 検証チェックリスト（loop に触れた heartbeat を終了する前）

- [ ] 正確な benchmark command、`PAPERCLIPAI_CMD` binding、dispatch runner config を含むすべての入力が top-level loop issue に記録されている。
- [ ] Iteration counter が最新で、budget 内に収まっている。
- [ ] Paperclip App worktree pointer を引き続き解決でき、iteration の run/implementation/rerun child がその workspace を共有している。
- [ ] smoke run に run id、manifest、`results.jsonl`、Harbor raw job folder、stop reason が記録されている。
- [ ] Paperclip telemetry で benchmark issue の割り当てと heartbeat の enabled/observed が確認できるか、iteration が harness/setup no-dispatch と明示的に分類されている。
- [ ] Diagnosis が `/diagnose-why-work-stopped` パターンを適用し、進行していない issue をすべて分類し、3 つの不変条件を確認している。
- [ ] 未承認の fix proposal に対する implementation child が存在しない。proposal がある場合は、最新の plan revision に対する `request_confirmation` が開いている。
- [ ] すべての loop issue と iteration issue が terminal、明示的な live、明示的な waiting、または名前付き blocker の state にある。
- [ ] この heartbeat で loop が停止した場合、stop reason が pass、board rejection、budget exhausted、名前付きの real blocker のいずれかである。
- [ ] この heartbeat で company-skill library mutation が発生していない。

## Deterministic smoke

skill を install または変更した後、live Terminal-Bench loop で operational とみなす前に、この smoke を実行します:

```sh
pnpm smoke:terminal-bench-loop-skill
```

このコマンドは `PAPERCLIP_API_URL`、`PAPERCLIP_API_KEY`、`PAPERCLIP_COMPANY_ID` から現在の Paperclip API token と company を取得します。`PAPERCLIP_TASK_ID` が設定されている場合は、smoke issue をその source issue 配下に追加し、project/goal context を継承します。デフォルトでは検証後に短命な smoke issue を cancel します。`-- --keep` を渡すと、検証済みの `blocked` loop parent、`in_review` iteration child、保留中の confirmation を手動確認用に残します。

この smoke は deterministic かつ意図的に non-comparable です。Terminal-Bench、Harbor、agent model、provider runtime は起動しません。control-plane の形だけを検証します:

- ローカルの `.agents/skills/terminal-bench-loop/SKILL.md` に loop contract の用語が含まれていること。
- top-level loop issue を作成し、blocker posture に更新できること。
- loop parent 配下に iteration child issue を作成できること。
- モックの benchmark artifact path が `run` document に記録されること。
- `diagnosis` document に正確な stop point と次のアクション担当者が記載されること。
- `request_confirmation` interaction が作成され、iteration child が silent review ではなく typed waiting path を伴う `in_review` に留まること。
