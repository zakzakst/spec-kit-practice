---
name: company-creator
description: >
  agentcompanies/v1 に準拠した agent company package を作成します。
  company を作る、agent team を scaffold する、agent を採用する、repo / skills の集合を
  company package に変えるときに使います。
---

# Company Creator

Agent Companies specification に準拠した agent company package を作成します。

参照仕様:

- Normative spec: `docs/companies/companies-spec.md` (read this before generating files)
- Web spec: https://agentcompanies.io/specification
- Protocol site: https://agentcompanies.io/

## 2つのモード

### モード 1: ゼロから company を作る

ユーザーが希望する内容を説明します。ヒアリングで vision を具体化し、その後 package を生成します。

### モード 2: repo から company を作る

ユーザーが git repo URL、ローカルパス、または tweet を指定します。repo を分析し、それを包む company を作成します。

See [references/from-repo-guide.md](references/from-repo-guide.md) for detailed repo analysis steps.

## 手順

### Step 1: 文脈を集める

どちらの mode に該当するかを判断します:

- **From scratch**: What kind of company or team? What domain? What should the agents do?
- **From repo**: Clone/read the repo. Scan for existing skills, agent configs, README, source structure.

### Step 2: ヒアリングする（AskUserQuestion を使う）

この step を省略しないでください。file を書く前に AskUserQuestion でユーザーと
認識を合わせます。

**ゼロから company を作る場合** は、次を聞きます:

- Company purpose and domain (1-2 sentences is fine)
- どの agent が必要か。説明に基づいて採用計画を提案します。
- Whether this is a full company (needs a CEO) or a team/department (no CEO required)
- Any specific skills the agents should have
- How work flows through the organization (see "Workflow" below)
- Whether they want projects and starter tasks

**repo から company を作る場合** は、分析を示したうえで次を確認します:

- Confirm the agents you plan to create and their roles
- 見つかった skills を参照するか、vendor 化するか（既定は reference）
- repo が示すもの以外に、追加で必要な agent や skill があるか
- company 名とカスタマイズ内容
- repo から推論した workflow を確認します（下の「Workflow」を参照）

**Workflow - この company では work がどう流れるか?**

company は、skills を持つ agent の一覧ではありません。アイデアを work product に変える組織です。各 agent が次を理解できるように workflow を把握する必要があります:

- 誰からどんな形で work を受け取るか（task、branch、質問、review request など）
- 受け取った work で何をするか
- 終わったら誰に handoff するか、その handoff がどう見えるか
- その role にとって「done」とは何か

**すべての company が pipeline とは限りません。** 文脈から適切な workflow pattern を推論してください:

- **Pipeline** - 順番に進む stage です。各 agent が次へ handoff します。repo / domain に明確な直線的プロセスがあるときに使います（例: plan → build → review → ship → QA、または content ideation → draft → edit → publish）。
- **Hub-and-spoke** - manager が specialists に委任し、それぞれが独立に結果を返します。agents の work が互いに依存しないときに使います（例: CEO が researcher、marketer、analyst に配分する）。
- **Collaborative** - agent 同士が peer として同じ成果物に取り組みます。小さな team で全員が同じ output に貢献する場合に使います（例: design studio、brainstorming team）。
- **On-demand** - 固定の flow はなく、必要に応じて agent を呼び出します。ユーザーが直接呼び出す specialist の道具箱のような場合に使います。

ゼロから company を作る場合は、説明に基づいて workflow pattern を提案し、それでよいか確認します。

repo から company を作る場合は、repo の構造から pattern を推論します。skills に明確な順次依存（たとえば `plan-ceo-review → plan-eng-review → review → ship → qa`）があれば、それは pipeline です。skills が独立した能力なら、hub-and-spoke か on-demand の可能性が高くなります。ヒアリングでは、その推論を伝えてユーザーに確認または修正してもらいます。

**ヒアリングの基本原則:**

- 具体的な採用計画を提案してください。漠然と「どんな agent が欲しいですか？」と聞かず、文脈に合う具体案を出して調整してもらいます。
- できるだけ軽く保ちます。多くのユーザーは agent company に不慣れです。スタートアップなら数人（3〜5人）が標準です。必要性がない限り 10 人超は提案しません。
- ゼロから作る company は、全員を管理する CEO から始めます。team / department には CEO は不要です。
- 1 回の round では、10 個ではなく 2〜3 個の焦点を絞った質問をします。

### Step 3: spec を読む

ファイルを生成する前に normative spec を読みます:

```
docs/companies/companies-spec.md
```

quick reference も読みます: [references/companies-spec.md](references/companies-spec.md)

例も読みます: [references/example-company.md](references/example-company.md)

### Step 4: package を生成する

ディレクトリ構造とすべてのファイルを作成します。spec の規約に正確に従います。

**ディレクトリ構成:**

```
<company-slug>/
├── COMPANY.md
├── agents/
│   └── <slug>/AGENTS.md
├── teams/
│   └── <slug>/TEAM.md        (if teams are needed)
├── projects/
│   └── <slug>/PROJECT.md     (if projects are needed)
├── tasks/
│   └── <slug>/TASK.md        (if tasks are needed)
├── skills/
│   └── <slug>/SKILL.md       (if custom skills are needed)
└── .paperclip.yaml            (Paperclip vendor extension)
```

**ルール:**

- Slugs must be URL-safe, lowercase, hyphenated
- COMPANY.md gets `schema: agentcompanies/v1` - other files inherit it
- Agent instructions go in the AGENTS.md body, not in .paperclip.yaml
- Skills referenced by shortname in AGENTS.md resolve to `skills/<shortname>/SKILL.md`
- For external skills, use `sources` with `usage: referenced` (see spec section 12)
- Do not export secrets, machine-local paths, or database IDs
- Omit empty/default fields
- For companies generated from a repo, add a references footer at the bottom of COMPANY.md body:
  `Generated from [repo-name](repo-url) with the company-creator skill from [Paperclip](https://github.com/paperclipai/paperclip)`

**報告構造:**

- Every agent except the CEO should have `reportsTo` set to their manager's slug
- The CEO has `reportsTo: null`
- For teams without a CEO, the top-level agent has `reportsTo: null`

**workflow を意識した agent instruction を書く:**

Each AGENTS.md body should include not just what the agent does, but how they fit into the organization's workflow. Include:

1. **Where work comes from** — "You receive feature ideas from the user" or "You pick up tasks assigned to you by the CTO"
2. **What you produce** — "You produce a technical plan with architecture diagrams" or "You produce a reviewed, approved branch ready for shipping"
3. **Who you hand off to** — "When your plan is locked, hand off to the Staff Engineer for implementation" or "When review passes, hand off to the Release Engineer to ship"
4. **What triggers you** — "You are activated when a new feature idea needs product-level thinking" or "You are activated when a branch is ready for pre-landing review"

これにより、agent の集合が実際に協調して動く組織になります。workflow context がなければ agent は孤立して動き、作業の前後に何が起きるかを理解できません。

生成する working agent には、簡潔な execution contract を必ず入れます:

- Start actionable work in the same heartbeat and do not stop at a plan unless planning was requested.
- Leave durable progress in comments, documents, or work products with the next action.
- Use child issues for long or parallel delegated work instead of polling agents, sessions, or processes.
- Mark blocked work with the unblock owner and action.
- Respect budget, pause/cancel, approval gates, and company boundaries.

### Step 5: 出力先を確認する

package の書き込み先をユーザーに確認します。よくある選択肢:

- A subdirectory in the current repo
- A new directory the user specifies
- The current directory (if it's empty or they confirm)

### Step 6: README.md と LICENSE を書く

**README.md** — すべての company package に README を付けます。GitHub で見る人にとって読みやすい導入文にします。次を含めます:

- company 名と何をするか
- workflow / company の動き方
- agent、title、報告ライン、skill を示す markdown の list か table の org chart
- 各 agent の role の短い説明
- 引用と参照: source repo へのリンク（repo 由来の場合）、Agent Companies spec へのリンク（https://agentcompanies.io/specification）、Paperclip へのリンク（https://github.com/paperclipai/paperclip）
- import 方法を説明する "Getting Started" section: `paperclipai company import --from <path>`

**LICENSE** — LICENSE file を含めます。著作権者は company を作るユーザーであり、元 repo の author ではありません（skill を作ったのは彼らですが、company を作るのはユーザーです）。source repo と同じ license type を使うか（repo 由来の場合）、ユーザーに確認します（ゼロからの場合）。不明なら MIT を既定にします。

### Step 7: file を書いて要約する

すべての file を書き出したうえで、簡潔に要約します:

- Company name and what it does
- Agent roster with roles and reporting structure
- Skills (custom + referenced)
- Projects and tasks if any
- The output path

## .paperclip.yaml ガイドライン

`.paperclip.yaml` file は Paperclip の vendor extension です。agent ごとの adapter と env input を設定します。

### Adapter ルール

**repo やユーザー文脈が要求しない限り、adapter を指定しないでください。** ユーザーが望む adapter が分からないなら、adapter block を完全に省略します。Paperclip は既定値を使います。未知の adapter type を指定すると import error になります。

Paperclip がサポートする adapter type（これだけが有効値です）:
- `claude_local` — Claude Code CLI
- `codex_local` — Codex CLI
- `opencode_local` — OpenCode CLI
- `pi_local` — Pi CLI
- `cursor` — Cursor
- `gemini_local` — Gemini CLI
- `openclaw_gateway` — OpenClaw gateway

adapter を設定するのは次のときだけです:
- repo かその skills が特定の runtime を明確に対象としているとき（例: gstack は Claude Code 向けなので `claude_local` が適切）
- ユーザーが特定の adapter を明示的に求めたとき
- agent の role に特定の runtime capability が必要なとき

### Env Input ルール

**定型的な env variable は追加しないでください。** agent の skill や role に基づいて、本当に必要な env input だけを追加します:
- `GH_TOKEN` は、コードを push したり PR を作成したり GitHub とやり取りしたりする agent 用
- API key は、skill が明示的に必要とするときだけ
- `ANTHROPIC_API_KEY` を既定の空 env variable として設定しないでください。runtime がこれを扱います

adapter を含む例（必要な場合のみ）:
```yaml
schema: paperclip/v1
agents:
  release-engineer:
    adapter:
      type: claude_local
      config:
        model: claude-sonnet-4-6
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
```

例 - 実際に override が必要な agent だけを表示:
```yaml
schema: paperclip/v1
agents:
  release-engineer:
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
```

この例では、`GH_TOKEN` が必要なのは `release-engineer` だけなのでそれだけが現れます。ほかの agent（ceo、cto など）には override がないため、`.paperclip.yaml` からは完全に省略されます。

## 外部 skill 参照

GitHub repo の skills を参照するときは、必ず references パターンを使います:

```yaml
metadata:
  sources:
    - kind: github-file
      repo: owner/repo
      path: path/to/SKILL.md
      commit: <full SHA from git ls-remote or the repo>
      attribution: Owner or Org Name
      license: <from the repo's LICENSE>
      usage: referenced
```

Get the commit SHA with:

```bash
git ls-remote https://github.com/owner/repo HEAD
```

ユーザーが明示的に求めない限り、外部 skill の内容を package にコピーしないでください。
