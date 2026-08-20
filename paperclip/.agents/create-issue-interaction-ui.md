---
name: create-issue-interaction-ui
description: >
  新しい Paperclip issue-thread interaction kind を end-to-end で追加します。
  repo 作業で request_confirmation、checkbox confirmation、ask_user_questions、
  suggest_tasks などの interaction card を導入・拡張するときに使います。
---

# 新しい issue-thread interaction UI を作る（Developer/maintainer skill）

Developer/maintainer skill です。production の Paperclip agents には install しないでください。

この skill は、Paperclip contributor が新しい issue-thread interaction kind を
shared contract から issue-detail wiring、helper、doc まで導入する流れを案内します。
意図的に developer/maintainer 向けです。対象は、デプロイ済みの Paperclip company 内で
動く operational agent ではなく、`paperclipai/paperclip` 内で code change を行う
human または coding agent です。

production の Paperclip agents に install しないでください。この guide は
Paperclip 自体を変更する repository contributor 向けです。

## 使う場面

- 新しい interaction kind を導入するとき（compact picker、structured rating、
  in-thread approval card など）。
- 既存 interaction の並列 variant が必要で、payload の shape、validation、
  resolution outcome が異なるとき（`ask_user_questions` は option 数、target binding、
  result shape が違うので不適切）。
- 新しい card に対して reviewer が「他の interaction と同じ audit、staleness、
  supersede、continuation semantics」を求めているとき。

## 使わない場面

- 新しい payload schema を必要としない既存 interaction kind に field を追加するだけのとき。
  既存の validator / UI をそのまま patch してください。
- Paperclip agents が interaction を *呼び出す方法* を変えるとき。
  その場合は `skills/paperclip` または `references/api-reference.md` を更新します。
  それは agent guidance であって card work ではありません。
- non-thread UI（issue detail sidebar、project board widget など）を作るとき。
  それぞれ独自の component convention があります。

## mental model

すべての issue-thread interaction には4つの moving part があります:

| Layer        | Owns                                                                                              |
|--------------|---------------------------------------------------------------------------------------------------|
| Shared       | Kind constant, payload/result interfaces, Zod validators, exported types, shared-test coverage.    |
| Server       | Service create/accept/reject/respond, staleness, supersede, idempotency, activity log, wake send. |
| UI           | Card pending/resolved/stale states, fixtures, Storybook, issue-thread/IssueDetail wiring.         |
| Helpers/Docs | CLI command, MCP tool, plugin SDK type+host+testing path, `skills/paperclip` guidance.            |

既存の4種類が canonical prior art です。いちばん近いものを選び、
並行した仕組みを新しく作るのではなく、その plumbing をコピーしてください:

- `request_confirmation` — single yes/no bound to a `target` with stale/supersede.
- `request_checkbox_confirmation` — bounded multi-select against an immutable option set.
- `ask_user_questions` — small typed form, no target binding.
- `suggest_tasks` — proposes tasks the board can accept individually.

新しい card が target binding と yes/no-style の resolution を必要とするなら、
2つの `request_*` kind を手本にします。structured form なら
`ask_user_questions`、作成可能な child entity を生むなら `suggest_tasks` を
手本にします。

## 代表的な worked example

現在の最良の end-to-end reference は checkbox confirmation の rollout
（`4d5322c82` で merge、GitHub PR `#7649`）です。始める前にその diff を読んでください:

```sh
git show --stat 4d5322c82
```

そこで実装された plan は、[PAP-10415](/PAP/issues/PAP-10415#document-plan) の
issue document として保存されています。Paperclip 自体でこの作業を進めるなら、
自分の plan document の template として使ってください。

## 作業順序

最初に shared contract をやります。UI が固まる前でも着地できる最小の正しい変更で、
後続の layer はすべてその型と validator を読みます。

### 1. shared contract（最小、最初に着地）

修正する箇所:

- `packages/shared/src/constants.ts` — add the kind string to
  `ISSUE_THREAD_INTERACTION_KINDS` and any size constant (mirror
  `REQUEST_CHECKBOX_CONFIRMATION_OPTION_LIMIT = 200`).
- `packages/shared/src/types/issue.ts` — add `Option`, `Payload`, `Result`,
  and `Interaction` interfaces. Extend the `IssueThreadInteraction` and
  payload/result union types at the bottom of the file.
- `packages/shared/src/types/index.ts` — re-export the new types.
- `packages/shared/src/validators/issue.ts` — add Zod schemas for payload,
  result, and the create-input variant. Reuse the existing
  `requestConfirmationTargetSchema` when target binding applies.
- `packages/shared/src/validators/index.ts` — re-export the new schemas.
- `packages/shared/src/index.ts` — re-export at the package root.
- `packages/shared/src/issue-thread-interactions.test.ts` — extend the table
  tests for the new payload variant.

すでに議論されていて、必ず守るべき validation invariant:

- Option lists are bounded (the checkbox kind uses 200; pick a number the UX
  can render compactly).
- Option ids are unique within a payload and any default selection must
  reference known ids.
- Labels and descriptions are length-capped to match existing question
  options. Do not invent looser caps.
- Target binding uses the shared `RequestConfirmationTarget` schema so stale
  expiration runs through one code path.

### 2. server service と routes

修正する箇所:

- `server/src/services/issue-thread-interactions.ts` — add the kind to:
  - the supported-kinds list (`SUPPORTED_KINDS` near the top),
  - `mapInteractionRow` (the `switch (row.kind)` over payload/result parsers),
  - create-input validation (`switch (data.kind)`),
  - the accept/reject/respond/stale-expiration branches,
  - the activity-log payload and the continuation wake payload.
- `server/src/routes/issues.ts` — extend any kind-specific branches (notably
  the response-shape branch around line 6096 in the checkbox PR).
- `server/src/__tests__/issue-thread-interactions-service.test.ts` — cover
  create, accept-with-result, reject-with-reason, stale-target expiration,
  supersede-on-user-comment, idempotency conflict, and wake payload shape.
- `server/src/__tests__/issue-thread-interaction-routes.test.ts` — cover
  create + respond/accept/reject HTTP behavior, company scoping, and
  authorization.

サーバー側の不変条件:

- Board 限定の解決にします。agent が作成した accept/reject は既存の 403
  経路で拒否し、kind ごとの bypass は追加しないでください。
- company スコープを守ります。読み取り、書き込み、期限切れ、上書きのすべてで
  `companyId` を必ず条件に含めます。`issueId` 単独では信じません。
- stale target を扱います。作成時に `target` が指定されていて、新しい revision が
  到着したら、interaction は `outcome: "stale_target"` で期限切れになります。
  独自の stale 判定は書かず、他の `request_*` kind が使う同じ helper を呼んでください。
- user comment による上書きです。payload schema が別途明記していない限り、
  `supersedeOnUserComment: true` を既定にします。
- idempotency を守ります。既存 kind と同じ決定論的な `idempotencyKey`
  形式 (`<kind>:<issueId>:<decisionKey>:<revisionId>`) を必ず守り、重複 POST は
  新しい card を積まず、既存の card を返してください。
- continuation policy をサポートします。`none`、`wake_assignee`、
  `wake_assignee_on_accept` に対応し、質問者が答え待ちで *blocked* なのか
  (`wake_assignee`)、それとも acceptance だけ気にするのか
  (`wake_assignee_on_accept`) に応じて適切な default を選んでください。

### 3. UI card と issue-thread wiring

修正する箇所:

- `ui/src/components/IssueThreadInteractionCard.tsx` — add a card component
  (e.g. `RequestCheckboxConfirmationCard`) and a resolution component
  (e.g. `RequestCheckboxConfirmationResolution`). Branch the existing
  switch by `interaction.kind`. Reuse the card shell — do not introduce a
  parallel card frame.
- `ui/src/lib/issue-thread-interactions.ts` — add typed helpers like
  `getCheckboxConfirmationSelectedLabels` so the card stays declarative.
- `ui/src/lib/issue-thread-interactions.test.ts` — test the helpers.
- `ui/src/components/IssueThreadInteractionCard.test.tsx` — pending,
  resolved, stale, disabled/submitting, and validation-error states.
- `ui/src/fixtures/issueThreadInteractionFixtures.ts` — seed at least one
  pending and one resolved fixture for the new kind.
- `ui/src/stories/issue-thread-interactions.stories.tsx` — Storybook entries
  for the key states.
- `ui/src/pages/IssueDetail.tsx` — extend the per-kind branches the card is
  rendered from (callback wiring, response submission).
- `ui/src/components/IssueChatThread.tsx` — if the kind affects thread-level
  rendering (badge, summary, count), update the per-kind switches here.
- `ui/src/api/issues.ts` — extend the typed accept/reject/respond bodies.

UI 側の不変条件:

- コンパクトに描画します。約100件の option でも無理なく表示できるようにし
  （スクロール領域を制限し、resolved state の要約は count を先頭に置きます）、
  選択済みの項目をすべて inline で chip 化しないでください。
- Select all と clear selection は global menu ではなく card 内に置きます。
- accept payload には kind 固有の field 名を使います（たとえば
  `selectedOptionIds`。suggest-tasks の `selectedClientKeys` は使いません）。
  他の kind の field 名を流用しないでください。
- stale / superseded / accepted の各 state は、それぞれ別の文面を表示します。
  既存の resolution-component shell を再利用してください。

### 4. CLI、MCP、plugin SDK の helper

外部 caller が、新しい interaction を JSON を手書きせずに作成できるようにする
必要があります。修正する箇所:

- `cli/src/commands/client/issue.ts` — add a CLI sub-command or extend the
  generic interaction create path.
- `cli/src/__tests__/issue-subresources.test.ts` — cover the new flag set.
- `packages/mcp-server/src/tools.ts` — add an MCP tool that accepts the new
  payload shape; reuse the existing `createIssueThreadInteraction` codepath.
- `packages/mcp-server/src/tools.test.ts` — cover the tool's payload shape.
- `packages/plugins/sdk/src/types.ts` — add the typed
  `CreateIssueThreadInteraction` variant so plugin authors get autocomplete.
- `packages/plugins/sdk/src/worker-rpc-host.ts` — extend the kind switch in
  the create call.
- `packages/plugins/sdk/src/testing.ts` — extend the test harness so plugins
  can simulate the new kind end-to-end.
- `packages/plugins/sdk/tests/testing-actions.test.ts` — round-trip test for
  the new kind through the test harness.

### 5. agent へのガイダンス

修正する箇所:

- `skills/paperclip/SKILL.md` — add a row to the interaction-kinds table:
  *when to use*, *when not to use*, plus a copyable payload example.
- `skills/paperclip/references/api-reference.md` — full payload and result
  schemas, validation limits, create/respond bodies, error codes.

skills の文章は runtime agent が読みます。簡潔に保ち、兄弟 kind との違いが
1〜2文で明確に伝わるようにしてください。

## review を依頼する前に実行する test

checkbox PR では `NODE_ENV=test` で、この焦点を絞ったセットを実行しました。
新しい kind でも同じ形を使い、あなたの新しい test file に差し替えてください:

```sh
NODE_ENV=test pnpm run preflight:workspace-links
NODE_ENV=test pnpm exec vitest run \
  packages/shared/src/issue-thread-interactions.test.ts \
  server/src/__tests__/issue-thread-interaction-routes.test.ts \
  server/src/__tests__/issue-thread-interactions-service.test.ts \
  ui/src/components/IssueThreadInteractionCard.test.tsx \
  ui/src/lib/issue-thread-interactions.test.ts \
  cli/src/__tests__/issue-subresources.test.ts \
  packages/mcp-server/src/tools.test.ts \
  packages/plugins/sdk/tests/testing-actions.test.ts
```

UI vitest が `act is not a function` で失敗する場合、shell が `NODE_ENV=production`
になっています（React の production build を拾っています）。`NODE_ENV=test`
を明示して再実行してください。

## pre-merge checklist

- [ ] 新しい kind が `ISSUE_THREAD_INTERACTION_KINDS` に追加され、export されている。
- [ ] payload と result の interface が versioned になっている
      （`version: 1` から開始）。
- [ ] Zod validator が option / label / description の上限と id の一意性を
      強制している。
- [ ] target binding（ある場合）が共有の `RequestConfirmationTarget` path を使っている。
- [ ] service が create、accept、reject/respond、stale-target、
      supersede-on-user-comment、idempotency、activity log、continuation wake を
      すべて処理できる。
- [ ] route が board-only resolution と company scoping を守っている。
- [ ] UI が pending、resolved、stale、disabled/submitting、validation-error を
      表示し、resolved state の大きな selection を count-first で要約している。
- [ ] 新しい kind 用の fixture と Storybook entry がある。
- [ ] CLI、MCP、plugin SDK の helper が新しい payload shape を受け入れ、test coverage がある。
- [ ] `skills/paperclip/SKILL.md` と `references/api-reference.md` が更新されている。
- [ ] 上の focused test set が green で、CI gate も通っている。

## review で見つかった anti-pattern

これは checkbox PR の review thread から出てきたもので、次回避ける価値があります:

- 別の kind の accept-payload field 名を流用すること（たとえば
  `selectedOptionIds` を導入せずに `selectedClientKeys` に相乗りする）。
  各 kind は自分自身の field 名を持ちます。
- 既存の `request_confirmation` helper に流さず、staleness や supersede の logic を
  並列に書くこと。これは気づかないうちに挙動をずらします。
- resolved state で何百もの selected-option chip を描画すること。
  大きな選択は、まず count で要約しなければなりません。
- "API が generic だから大丈夫" という理由で plugin SDK / MCP / CLI coverage を
  省くこと。typed helper がないと外部 caller は新しい kind を拾えず、後で agent flow が
  壊れます。
- server route が受け付ける前に skills guidance に kind を追加すること。
  agent は新しい kind を試して、本番で 400 を受けます。
