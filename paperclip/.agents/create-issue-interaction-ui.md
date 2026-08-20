---
name: create-issue-interaction-ui
description: >
  新しい Paperclip issue-thread interaction kind を end-to-end で追加します。
  repo 作業で request_confirmation、checkbox confirmation、ask_user_questions、
  suggest_tasks などの interaction card を導入・拡張するときに使います。
---

# 新しい issue-thread interaction UI を作る（開発者・メンテナー向け skill）

開発者・メンテナー向けの skill です。production の Paperclip agents には install しないでください。

この skill は、Paperclip contributor が新しい issue-thread interaction kind を
shared contract から issue-detail wiring、helper、doc まで導入する流れを案内します。
意図的に開発者・メンテナー向けです。対象は、デプロイ済みの Paperclip company 内で
動く operational agent ではなく、`paperclipai/paperclip` 内で code change を行う
human または coding agent です。

production の Paperclip agents に install しないでください。このガイドは
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

## メンタルモデル

すべての issue-thread interaction には4つの構成要素があります:

| 層           | 担当範囲                                                                                           |
|--------------|---------------------------------------------------------------------------------------------------|
| Shared       | kind 定数、payload/result interface、Zod validator、export された型、shared test の coverage |
| Server       | service の create/accept/reject/respond、staleness、supersede、idempotency、activity log、wake 送信 |
| UI           | card の pending/resolved/stale state、fixture、Storybook、issue-thread/IssueDetail の wiring |
| Helpers/Docs | CLI command、MCP tool、plugin SDK の type・host・testing path、`skills/paperclip` のガイダンス |

既存の4種類が標準的な先行実装です。いちばん近いものを選び、
並行した仕組みを新しく作るのではなく、その配線をコピーしてください:

- `request_confirmation` — `target` に紐づく単一の yes/no。stale/supersede に対応。
- `request_checkbox_confirmation` — 変更できない option 集合に対する上限付き multi-select。
- `ask_user_questions` — target binding のない小規模な型付き form。
- `suggest_tasks` — board が個別に受け入れられる task を提案。

新しい card が target binding と yes/no-style の resolution を必要とするなら、
2つの `request_*` kind を手本にします。structured form なら
`ask_user_questions`、作成可能な child entity を生むなら `suggest_tasks` を
手本にします。

## 代表的な実装例

現在の最良の end-to-end の参考実装は checkbox confirmation の展開
（`4d5322c82` で merge、GitHub PR `#7649`）です。始める前にその diff を読んでください:

```sh
git show --stat 4d5322c82
```

そこで実装された plan は、[PAP-10415](/PAP/issues/PAP-10415#document-plan) の
issue document として保存されています。Paperclip 自体でこの作業を進めるなら、
自分の plan document のテンプレートとして使ってください。

## 作業順序

最初に shared contract を実装します。UI が固まる前でも着地できる最小の正しい変更で、
後続の層はすべてその型と validator を利用します。

### 1. shared contract（最小、最初に実装）

修正する箇所:

- `packages/shared/src/constants.ts` — `ISSUE_THREAD_INTERACTION_KINDS` に kind の文字列と
  サイズ定数を追加する（`REQUEST_CHECKBOX_CONFIRMATION_OPTION_LIMIT = 200` を参考にする）。
- `packages/shared/src/types/issue.ts` — `Option`、`Payload`、`Result`、`Interaction` interface を追加する。
  ファイル末尾の `IssueThreadInteraction` と payload/result の union type を拡張する。
- `packages/shared/src/types/index.ts` — 新しい型を再 export する。
- `packages/shared/src/validators/issue.ts` — payload、result、create-input variant の Zod schema を追加する。
  target binding を使う場合は既存の `requestConfirmationTargetSchema` を再利用する。
- `packages/shared/src/validators/index.ts` — 新しい schema を再 export する。
- `packages/shared/src/index.ts` — package root から再 export する。
- `packages/shared/src/issue-thread-interactions.test.ts` — 新しい payload variant の table test を拡張する。

すでに議論されていて、必ず守るべき validation の不変条件:

- option list には上限を設ける（checkbox kind は 200。UX がコンパクトに描画できる数を選ぶ）。
- option id は payload 内で一意にし、default selection は既知の id を参照しなければならない。
- label と description の長さは既存の question option に合わせて制限する。より緩い上限を独自に作らない。
- target binding には共有の `RequestConfirmationTarget` schema を使い、stale の期限切れ処理を
  1つの code path に通す。

### 2. server service と routes

修正する箇所:

- `server/src/services/issue-thread-interactions.ts` — 次の箇所に kind を追加する:
  - 対応 kind の一覧（上部付近の `SUPPORTED_KINDS`）
  - `mapInteractionRow`（payload/result parser を処理する `switch (row.kind)`）
  - create-input validation（`switch (data.kind)`）
  - accept/reject/respond/stale-expiration の分岐
  - activity-log payload と continuation wake payload
- `server/src/routes/issues.ts` — kind 固有の分岐を拡張する（特に checkbox PR の
  6096 行付近にある response shape の分岐）。
- `server/src/__tests__/issue-thread-interactions-service.test.ts` — create、accept-with-result、
  reject-with-reason、stale-target expiration、supersede-on-user-comment、idempotency conflict、
  wake payload shape を対象にする。
- `server/src/__tests__/issue-thread-interaction-routes.test.ts` — create と respond/accept/reject の
  HTTP 動作、company scoping、authorization を対象にする。

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

- `ui/src/components/IssueThreadInteractionCard.tsx` — card component（例: `RequestCheckboxConfirmationCard`）と
  resolution component（例: `RequestCheckboxConfirmationResolution`）を追加する。既存の switch を
  `interaction.kind` で分岐する。card shell を再利用し、並行した card frame は導入しない。
- `ui/src/lib/issue-thread-interactions.ts` — `getCheckboxConfirmationSelectedLabels` のような型付き helper を追加し、
  card を宣言的に保つ。
- `ui/src/lib/issue-thread-interactions.test.ts` — helper をテストする。
- `ui/src/components/IssueThreadInteractionCard.test.tsx` — pending、resolved、stale、disabled/submitting、
  validation-error state をテストする。
- `ui/src/fixtures/issueThreadInteractionFixtures.ts` — 新しい kind の pending fixture と resolved fixture を
  少なくとも1つずつ用意する。
- `ui/src/stories/issue-thread-interactions.stories.tsx` —主要 state の Storybook entry を追加する。
- `ui/src/pages/IssueDetail.tsx` — card の描画元となる kind ごとの分岐を拡張する（callback wiring、response submission）。
- `ui/src/components/IssueChatThread.tsx` — kind が thread-level rendering（badge、summary、count）に影響する場合は、
  ここにある kind ごとの switch を更新する。
- `ui/src/api/issues.ts` — 型付きの accept/reject/respond body を拡張する。

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

- `cli/src/commands/client/issue.ts` — CLI sub-command を追加するか、generic interaction create path を拡張する。
- `cli/src/__tests__/issue-subresources.test.ts` —新しい flag set を対象にする。
- `packages/mcp-server/src/tools.ts` —新しい payload shape を受け取る MCP tool を追加し、既存の
  `createIssueThreadInteraction` codepath を再利用する。
- `packages/mcp-server/src/tools.test.ts` — tool の payload shape を対象にする。
- `packages/plugins/sdk/src/types.ts` — plugin author が autocomplete を使えるよう、型付きの
  `CreateIssueThreadInteraction` variant を追加する。
- `packages/plugins/sdk/src/worker-rpc-host.ts` — create call の kind switch を拡張する。
- `packages/plugins/sdk/src/testing.ts` — plugin が新しい kind を end-to-end でシミュレートできるよう test harness を拡張する。
- `packages/plugins/sdk/tests/testing-actions.test.ts` — test harness を通じた新しい kind の往復テストを追加する。

### 5. agent へのガイダンス

修正する箇所:

- `skills/paperclip/SKILL.md` — interaction-kinds table に行を追加する:
  *使う場面*、*使わない場面*、コピー可能な payload 例。
- `skills/paperclip/references/api-reference.md` — payload と result の完全な schema、validation の上限、
  create/respond body、error code を記載する。

skills の文章は runtime agent が読みます。簡潔に保ち、兄弟 kind との違いが
1〜2文で明確に伝わるようにしてください。

## review を依頼する前に実行するテスト

checkbox PR では `NODE_ENV=test` で、次の焦点を絞ったセットを実行しました。
新しい kind でも同じ形を使い、新しい test file に差し替えてください:

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
になっています（React の production build を拾っています）。`NODE_ENV=test` を明示して再実行してください。

## pre-merge チェックリスト

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
- [ ] CLI、MCP、plugin SDK の helper が新しい payload shape を受け入れ、テスト coverage がある。
- [ ] `skills/paperclip/SKILL.md` と `references/api-reference.md` が更新されている。
- [ ] 上の絞り込みテストセットがすべて通り、CI gate も通過している。

## review で見つかったアンチパターン

これは checkbox PR の review thread から出てきたもので、次回は避けるべきものです:

- 別の kind の accept-payload field 名を流用すること（たとえば
  `selectedOptionIds` を導入せずに `selectedClientKeys` に相乗りする）。
  各 kind は自分自身の field 名を持ちます。
- 既存の `request_confirmation` helper に流さず、staleness や supersede の logic を
  並列に書くこと。これは気づかないうちに挙動をずらします。
- resolved state で何百もの selected-option chip を描画すること。
  大きな選択は、まず count で要約しなければなりません。
- 「API が generic だから大丈夫」という理由で plugin SDK / MCP / CLI coverage を
  省くこと。typed helper がないと外部 caller は新しい kind を利用できず、後で agent flow が
  壊れます。
- server route が受け付ける前に skills guidance に kind を追加すること。
  agent は新しい kind を試して、本番で 400 を受けます。
