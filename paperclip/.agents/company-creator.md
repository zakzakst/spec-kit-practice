---
name: company-creator
description: >
  agentcompanies/v1 に準拠した agent company package を作成します。
  company を作る、agent team を scaffold する、agent を採用する、repo / skills の集合を
  company package に変えるときに使います。
---

# Company Creator

Agent Companies specification に準拠した agent company package を作成します。

Spec references:

- Normative spec: `docs/companies/companies-spec.md` (read this before generating files)
- Web spec: https://agentcompanies.io/specification
- Protocol site: https://agentcompanies.io/

## 2つのモード

### Mode 1: ゼロから company を作る

The user describes what they want. Interview them to flesh out the vision, then generate the package.

### Mode 2: repo から company を作る

The user provides a git repo URL, local path, or tweet. Analyze the repo, then create a company that wraps it.

See [references/from-repo-guide.md](references/from-repo-guide.md) for detailed repo analysis steps.

## 手順

### Step 1: 文脈を集める

Determine which mode applies:

- **From scratch**: What kind of company or team? What domain? What should the agents do?
- **From repo**: Clone/read the repo. Scan for existing skills, agent configs, README, source structure.

### Step 2: ヒアリングする（AskUserQuestion を使う）

この step を省略しないでください。file を書く前に AskUserQuestion でユーザーと
認識を合わせます。

**ゼロから company を作る場合** は、次を聞きます:

- Company purpose and domain (1-2 sentences is fine)
- What agents they need - propose a hiring plan based on what they described
- Whether this is a full company (needs a CEO) or a team/department (no CEO required)
- Any specific skills the agents should have
- How work flows through the organization (see "Workflow" below)
- Whether they want projects and starter tasks

**repo から company を作る場合** は、分析を示したうえで次を確認します:

- Confirm the agents you plan to create and their roles
- Whether to reference or vendor any discovered skills (default: reference)
- Any additional agents or skills beyond what the repo provides
- Company name and any customization
- Confirm the workflow you inferred from the repo (see "Workflow" below)

**Workflow — この company では work がどう流れるか?**

company は、skills を持つ agent の一覧ではありません。アイデアを work product に
変える組織です。各 agent が次を理解できるように workflow を把握する必要があります:

- Who gives them work and in what form (a task, a branch, a question, a review request)
- What they do with it
- Who they hand off to when they're done, and what that handoff looks like
- What "done" means for their role

**すべての company が pipeline とは限りません。** 文脈から適切な workflow pattern を
推論してください:

- **Pipeline** — sequential stages, each agent hands off to the next. Use when the repo/domain has a clear linear process (e.g. plan → build → review → ship → QA, or content ideation → draft → edit → publish).
- **Hub-and-spoke** — a manager delegates to specialists who report back independently. Use when agents do different kinds of work that don't feed into each other (e.g. a CEO who dispatches to a researcher, a marketer, and an analyst).
- **Collaborative** — agents work together on the same things as peers. Use for small teams where everyone contributes to the same output (e.g. a design studio, a brainstorming team).
- **On-demand** — agents are summoned as needed with no fixed flow. Use when agents are more like a toolbox of specialists the user calls directly.

ゼロから company を作る場合は、説明に基づいて workflow pattern を提案し、
それでよいか確認します。

repo から company を作る場合は、repo の構造から pattern を推論します。skills に
明確な順次依存（たとえば `plan-ceo-review → plan-eng-review → review → ship → qa`）が
あれば、それは pipeline です。skills が独立した能力なら、hub-and-spoke か
on-demand の可能性が高くなります。ヒアリングでは、その推論を伝えてユーザーに
確認または修正してもらいます。

**ヒアリングの基本原則:**

- Propose a concrete hiring plan. Don't ask open-ended "what agents do you want?" - suggest specific agents based on context and let the user adjust.
- Keep it lean. Most users are new to agent companies. A few agents (3-5) is typical for a startup. Don't suggest 10+ agents unless the scope demands it.
- From-scratch companies should start with a CEO who manages everyone. Teams/departments don't need one.
- Ask 2-3 focused questions per round, not 10.

### Step 3: spec を読む

Before generating any files, read the normative spec:

```
docs/companies/companies-spec.md
```

Also read the quick reference: [references/companies-spec.md](references/companies-spec.md)

And the example: [references/example-company.md](references/example-company.md)

### Step 4: package を生成する

Create the directory structure and all files. Follow the spec's conventions exactly.

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

This turns a collection of agents into an organization that actually works together. Without workflow context, agents operate in isolation — they do their job but don't know what happens before or after them.

生成する working agent には、簡潔な execution contract を必ず入れます:

- Start actionable work in the same heartbeat and do not stop at a plan unless planning was requested.
- Leave durable progress in comments, documents, or work products with the next action.
- Use child issues for long or parallel delegated work instead of polling agents, sessions, or processes.
- Mark blocked work with the unblock owner and action.
- Respect budget, pause/cancel, approval gates, and company boundaries.

### Step 5: 出力先を確認する

Ask the user where to write the package. Common options:

- A subdirectory in the current repo
- A new directory the user specifies
- The current directory (if it's empty or they confirm)

### Step 6: README.md と LICENSE を書く

**README.md** — すべての company package に README を付けます。GitHub で見る人にとって
読みやすい導入文にします。次を含めます:

- Company name and what it does
- The workflow / how the company operates
- Org chart as a markdown list or table showing agents, titles, reporting structure, and skills
- Brief description of each agent's role
- Citations and references: link to the source repo (if from-repo), link to the Agent Companies spec (https://agentcompanies.io/specification), and link to Paperclip (https://github.com/paperclipai/paperclip)
- A "Getting Started" section explaining how to import: `paperclipai company import --from <path>`

**LICENSE** — LICENSE file を含めます。著作権者は company を作るユーザーであり、
元 repo の author ではありません（skill を作ったのは彼らですが、company を作るのは
ユーザーです）。source repo と同じ license type を使うか（repo 由来の場合）、
ユーザーに確認します（ゼロからの場合）。不明なら MIT を既定にします。

### Step 7: file を書いて要約する

Write all files, then give a brief summary:

- Company name and what it does
- Agent roster with roles and reporting structure
- Skills (custom + referenced)
- Projects and tasks if any
- The output path

## .paperclip.yaml Guidelines

The `.paperclip.yaml` file is the Paperclip vendor extension. It configures adapters and env inputs per agent.

### Adapter Rules

**Do not specify an adapter unless the repo or user context warrants it.** If you don't know what adapter the user wants, omit the adapter block entirely — Paperclip will use its default. Specifying an unknown adapter type causes an import error.

Paperclip's supported adapter types (these are the ONLY valid values):
- `claude_local` — Claude Code CLI
- `codex_local` — Codex CLI
- `opencode_local` — OpenCode CLI
- `pi_local` — Pi CLI
- `cursor` — Cursor
- `gemini_local` — Gemini CLI
- `openclaw_gateway` — OpenClaw gateway

Only set an adapter when:
- The repo or its skills clearly target a specific runtime (e.g. gstack is built for Claude Code, so `claude_local` is appropriate)
- The user explicitly requests a specific adapter
- The agent's role requires a specific runtime capability

### Env Inputs Rules

**Do not add boilerplate env variables.** Only add env inputs that the agent actually needs based on its skills or role:
- `GH_TOKEN` for agents that push code, create PRs, or interact with GitHub
- API keys only when a skill explicitly requires them
- Never set `ANTHROPIC_API_KEY` as a default empty env variable — the runtime handles this

Example with adapter (only when warranted):
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

Example — only agents with actual overrides appear:
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

In this example, only `release-engineer` appears because it needs `GH_TOKEN`. The other agents (ceo, cto, etc.) have no overrides, so they are omitted entirely from `.paperclip.yaml`.

## External Skill References

When referencing skills from a GitHub repo, always use the references pattern:

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

Do NOT copy external skill content into the package unless the user explicitly asks.
