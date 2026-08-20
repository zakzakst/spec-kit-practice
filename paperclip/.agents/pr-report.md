---
name: pr-report
description: >
  maintainer 級の PR / contribution report を HTML または Markdown で作成します。
  PR を深くレビューしたいとき、設計を説明したいとき、システム比較をしたいとき、
  あるいは merge recommendation を準備したいときに使います。
---

# PR レポート Skill

PR、branch、または大きな contribution の maintainer 級レビューを作成します。

基本姿勢:

- まず変更を理解してから判断する
- diff だけでなく、実際に組み上がった system を説明する
- architecture の問題と product scope の異議を切り分ける
- あいまいな印象ではなく、具体的な recommendation を出す

## 使う場面

次のような依頼があったときに使います:

- "review this PR deeply"
- "explain this contribution to me"
- "make me a report or webpage for this PR"
- "compare this design to similar systems"
- "should I merge this?"

## 出力

よくある出力:

- standalone HTML report in `tmp/reports/...`
- Markdown report in `report/` or another requested folder
- short maintainer summary in chat

ユーザーが webpage を求めたら、見出しが明確で視認性の高い、洗練された
standalone HTML artifact を作ります。

この skill に同梱されている resources:

- `references/style-guide.md` for visual direction and report presentation rules
- `assets/html-report-starter.html` for a reusable standalone HTML/CSS starter

## ワークフロー

### 1. 対象を把握し、枠組みを作る

可能なら GitHub の PR ページだけでなく、ローカルコードから作業します。

集めるもの:

- target branch or worktree
- diff size and changed subsystems
- relevant repo docs, specs, and invariants
- contributor intent if it is documented in PR text or design docs

最初に答えるべき問いは、「この変更は *何になろうとしているのか*？」です。

### 2. system の mental model を作る

ファイルごとのメモで止まらず、設計全体を再構成します:

- what new runtime or contract exists
- which layers changed: db, shared types, server, UI, CLI, docs
- lifecycle: install, startup, execution, UI, failure, disablement
- trust boundary: what code runs where, under what authority

大きな contribution では、first principles から system を教える tutorial-style の
section を入れます。

### 3. maintainer としてレビューする

finding は最初に出します。severity の高い順に並べます。

Prioritize:

- behavioral regressions
- trust or security gaps
- misleading abstractions
- lifecycle and operational risks
- coupling that will be hard to unwind
- missing tests or unverifiable claims

可能なら、必ず具体的な file reference を示します。

### 4. 異議の種類を区別する

Be explicit about whether a concern is:

- product direction
- architecture
- implementation quality
- rollout strategy
- documentation honesty

architecture への異議を scope への異議の中に隠さないでください。

### 5. 必要なら外部の先例と比較する

If the contribution introduces a framework or platform concept, compare it to
similar open-source systems.

When comparing:

- prefer official docs or source
- focus on extension boundaries, context passing, trust model, and UI ownership
- extract lessons, not just similarities

比較のための良い問い:

- Who owns lifecycle?
- Who owns UI composition?
- Is context explicit or ambient?
- Are plugins trusted code or sandboxed code?
- Are extension points named and typed?

### 6. recommendation を実行可能にする

Do not stop at "merge" or "do not merge."

Choose one:

- merge as-is
- merge after specific redesign
- salvage specific pieces
- keep as design research

If rejecting or narrowing, say what should be kept.

役立つ recommendation の分類:

- keep the protocol/type model
- redesign the UI boundary
- narrow the initial surface area
- defer third-party execution
- ship a host-owned extension-point model first

### 7. artifact を作る

推奨する report 構成:

1. Executive summary
2. What the PR actually adds
3. Tutorial: how the system works
4. Strengths
5. Main findings
6. Comparisons
7. Recommendation

HTML report の場合:

- use intentional typography and color
- make navigation easy for long reports
- favor strong section headings and small reference labels
- avoid generic dashboard styling

ゼロから作る前に `references/style-guide.md` を読みます。
素早く洗練された starter が役立つなら、`assets/html-report-starter.html` から始めます。
and replace the placeholder content with the actual report.

### 8. 引き渡し前に検証する

Check:

- artifact path exists
- findings still match the actual code
- any requested forbidden strings are absent from generated output
- if tests were not run, say so explicitly

## レビューの観点

### プラグインと platform 作業

特に注意する点:

- docs claiming sandboxing while runtime executes trusted host processes
- module-global state used to smuggle React context
- hidden dependence on render order
- plugins reaching into host internals instead of using explicit APIs
- "capabilities" that are really policy labels on top of fully trusted code

### 良い兆候

- typed contracts shared across layers
- explicit extension points
- host-owned lifecycle
- honest trust model
- narrow first rollout with room to grow

## 最終応答

chat では次を要約します:

- where the report is
- your overall call
- the top one or two reasons
- whether verification or tests were skipped

Keep the chat summary shorter than the report itself.
