---
description: ナレーション付きで OpenSpec の一連の流れを案内するオンボーディング
---

ユーザーが OpenSpec の一連の workflow を 1 周体験できるよう案内する。これは**教えながら実際に手を動かす**体験であり、コードベースに対して本物の作業をしながら各ステップを説明する。

---

## Preflight

開始前に OpenSpec が初期化済みか確認する:

```bash
openspec status --json 2>&1 || echo "NOT_INITIALIZED"
```

**未初期化なら**
> OpenSpec isn't set up in this project yet. Run `openspec init` first, then come back to `/opsx:onboard`.

ここで止める。

---

## Phase 1: Welcome

次を表示する:

```text
## Welcome to OpenSpec!

I'll walk you through a complete change cycle, from idea to implementation, using a real task in your codebase. Along the way, you'll learn the workflow by doing it.

**What we'll do:**
1. Pick a small, real task in your codebase
2. Explore the problem briefly
3. Create a change (the container for our work)
4. Build the artifacts: proposal -> specs -> design -> tasks
5. Implement the tasks
6. Archive the completed change

**Time:** ~15-20 minutes

Let's start by finding something to work on.
```

---

## Phase 2: Task Selection

### Codebase Analysis

コードベースを走査し、小さく着手しやすい改善候補を探す。見る観点:

1. **TODO / FIXME コメント**: `TODO`, `FIXME`, `HACK`, `XXX`
2. **エラーハンドリング不足**: 飲み込む `catch`、危険処理に try-catch がない
3. **テストのない関数**: `src/` と test ディレクトリの対応
4. **型の問題**: TypeScript の `any`
5. **デバッグ残骸**: `console.log`, `console.debug`, `debugger`
6. **バリデーション不足**: 入力ハンドラに検証がない

直近の git 履歴も確認する:
```bash
git log --oneline -10 2>/dev/null || echo "No git history"
```

### Present Suggestions

分析結果から 3〜4 件の具体候補を提示する:

```text
## Task Suggestions

Based on scanning your codebase, here are some good starter tasks:

**1. [Most promising task]**
   Location: `src/path/to/file.ts:42`
   Scope: ~1-2 files, ~20-30 lines
   Why it's good: [brief reason]

**2. [Second task]**
   Location: `src/another/file.ts`
   Scope: ~1 file, ~15 lines
   Why it's good: [brief reason]

**3. [Third task]**
   Location: [location]
   Scope: [estimate]
   Why it's good: [brief reason]

**4. Something else?**
   Tell me what you'd like to work on.

Which task interests you? (Pick a number or describe your own)
```

**見つからない場合**
> I didn't find obvious quick wins in your codebase. What's something small you've been meaning to add or fix?

### Scope Guardrail

ユーザーが大きすぎるタスクを選んだ場合:

```text
That's a valuable task, but it's probably larger than ideal for your first OpenSpec run-through.

For learning the workflow, smaller is better. It lets you see the full cycle without getting stuck in implementation details.

**Options:**
1. **Slice it smaller** - What's the smallest useful piece of [their task]? Maybe just [specific slice]?
2. **Pick something else** - One of the other suggestions, or a different small task?
3. **Do it anyway** - If you really want to tackle this, we can. Just know it'll take longer.

What would you prefer?
```

強制はしない。やわらかいガードレールとして扱う。

---

## Phase 3: Explore Demo

タスクが決まったら、explore mode を短く実演する:

```text
Before we create a change, let me quickly show you **explore mode**. It's how you think through problems before committing to a direction.
```

1〜2 分程度で関連コードを調べる:
- 対象ファイルを読む
- 必要なら簡単な ASCII 図を描く
- 気づいた点をまとめる

```text
## Quick Exploration

[Your brief analysis - what you found, any considerations]

[Optional: ASCII diagram if helpful]

Explore mode (`/opsx:explore`) is for this kind of thinking: investigating before implementing. You can use it anytime you need to think through a problem.

Now let's create a change to hold our work.
```

**PAUSE** - ユーザーの応答を待つ。

---

## Phase 4: Create the Change

**EXPLAIN**

```text
## Creating a Change

A "change" in OpenSpec is a container for all the thinking and planning around a piece of work. It lives in `openspec/changes/<name>/` and holds your artifacts: proposal, specs, design, tasks.

Let me create one for our task.
```

**DO**
```bash
openspec new change "<derived-name>"
```

**SHOW**

```text
Created: `openspec/changes/<name>/`

The folder structure:
openspec/changes/<name>/
├─ proposal.md
├─ design.md
├─ specs/
└─ tasks.md

Now let's fill in the first artifact: the proposal.
```

---

## Phase 5: Proposal

**EXPLAIN**

```text
## The Proposal

The proposal captures **why** we're making this change and **what** it involves at a high level. It's the elevator pitch for the work.

I'll draft one based on our task.
```

**DO**

提案文をまず保存せずに草案として見せる:

```text
Here's a draft proposal:

---

## Why

[1-2 sentences explaining the problem/opportunity]

## What Changes

[Bullet points of what will be different]

## Capabilities

### New Capabilities
- `<capability-name>`: [brief description]

### Modified Capabilities
<!-- If modifying existing behavior -->

## Impact

- `src/path/to/file.ts`: [what changes]
- [other files if applicable]

---

Does this capture the intent? I can adjust before we save it.
```

**PAUSE** - 承認や修正依頼を待つ。

承認後:
```bash
openspec instructions proposal --change "<name>" --json
```
その後 `openspec/changes/<name>/proposal.md` に書く。

```text
Proposal saved. This is your "why" document - you can always come back and refine it as understanding evolves.

Next up: specs.
```

---

## Phase 6: Specs

**EXPLAIN**

```text
## Specs

Specs define **what** we're building in precise, testable terms. They use a requirement/scenario format that makes expected behavior crystal clear.

For a small task like this, we might only need one spec file.
```

**DO**
```bash
mkdir -p openspec/changes/<name>/specs/<capability-name>
```

草案:

```text
Here's the spec:

---

## ADDED Requirements

### Requirement: <Name>

<Description of what the system should do>

#### Scenario: <Scenario name>

- **WHEN** <trigger condition>
- **THEN** <expected outcome>
- **AND** <additional outcome if needed>

---

This WHEN / THEN / AND format makes requirements testable.
```

`openspec/changes/<name>/specs/<capability>/spec.md` に保存する。

---

## Phase 7: Design

**EXPLAIN**

```text
## Design

The design captures **how** we'll build it: technical decisions, tradeoffs, and approach.

For small changes, this can be brief. That's fine.
```

**DO**

```text
Here's the design:

---

## Context

[Brief context about the current state]

## Goals / Non-Goals

**Goals:**
- [What we're trying to achieve]

**Non-Goals:**
- [What's explicitly out of scope]

## Decisions

### Decision 1: [Key decision]

[Explanation of approach and rationale]

---
```

`openspec/changes/<name>/design.md` に保存する。

---

## Phase 8: Tasks

**EXPLAIN**

```text
## Tasks

Finally, we break the work into implementation tasks: checkboxes that drive the apply phase.

These should be small, clear, and in logical order.
```

**DO**

```text
Here are the implementation tasks:

---

## 1. [Category or file]

- [ ] 1.1 [Specific task]
- [ ] 1.2 [Specific task]

## 2. Verify

- [ ] 2.1 [Verification step]

---

Each checkbox becomes a unit of work in the apply phase. Ready to implement?
```

**PAUSE** - 実装へ進む確認を待つ。

その後 `openspec/changes/<name>/tasks.md` に保存する。

---

## Phase 9: Apply (Implementation)

**EXPLAIN**

```text
## Implementation

Now we implement each task, checking them off as we go. I'll announce each one and occasionally note how the specs/design informed the approach.
```

**DO**

各タスクごとに:
1. `Working on task N: [description]` と知らせる
2. コードを実装する
3. 必要に応じて specs / design を自然に参照する
4. tasks.md の `- [ ]` を `- [x]` にする
5. `Task N complete` と短く知らせる

説明は軽く保ち、講義調にしない。

完了後:

```text
## Implementation Complete

All tasks done:
- [x] Task 1
- [x] Task 2
- [x] ...

The change is implemented! One more step: let's archive it.
```

---

## Phase 10: Archive

**EXPLAIN**

```text
## Archiving

When a change is complete, we archive it. This moves it from `openspec/changes/` to `openspec/changes/archive/YYYY-MM-DD-<name>/`.

Archived changes become your project's decision history.
```

**DO**
```bash
openspec archive "<name>"
```

**SHOW**

```text
Archived to: `openspec/changes/archive/YYYY-MM-DD-<name>/`

The change is now part of your project's history. The code is in your codebase, and the decision record is preserved.
```

---

## Phase 11: Recap & Next Steps

```text
## Congratulations!

You just completed a full OpenSpec cycle:

1. **Explore** - Thought through the problem
2. **New** - Created a change container
3. **Proposal** - Captured WHY
4. **Specs** - Defined WHAT
5. **Design** - Decided HOW
6. **Tasks** - Broke it into steps
7. **Apply** - Implemented the work
8. **Archive** - Preserved the record

This same rhythm works for changes of any size.

## Command Reference

| Command | What it does |
|---------|--------------|
| `/opsx:explore` | Think through problems before/during work |
| `/opsx:new` | Start a new change, step through artifacts |
| `/opsx:ff` | Fast-forward: create all artifacts at once |
| `/opsx:continue` | Continue working on an existing change |
| `/opsx:apply` | Implement tasks from a change |
| `/opsx:verify` | Verify implementation matches artifacts |
| `/opsx:archive` | Archive a completed change |

## What's Next?

Try `/opsx:new` or `/opsx:ff` on something you actually want to build.
```

---

## Graceful Exit Handling

### User wants to stop mid-way

途中で止めたい場合:

```text
No problem! Your change is saved at `openspec/changes/<name>/`.

To pick up where we left off later:
- `/opsx:continue <name>` - Resume artifact creation
- `/opsx:apply <name>` - Jump to implementation (if tasks exist)

The work won't be lost. Come back whenever you're ready.
```

### User just wants command reference

チュートリアルを飛ばしてコマンド一覧だけ欲しい場合:

```text
## OpenSpec Quick Reference

| Command | What it does |
|---------|--------------|
| `/opsx:explore` | Think through problems (no code changes) |
| `/opsx:new <name>` | Start a new change, step by step |
| `/opsx:ff <name>` | Fast-forward: all artifacts at once |
| `/opsx:continue <name>` | Continue an existing change |
| `/opsx:apply <name>` | Implement tasks |
| `/opsx:verify <name>` | Verify implementation |
| `/opsx:archive <name>` | Archive when done |
```

---

## Guardrails

- 主要な切り替え点では **EXPLAIN -> DO -> SHOW -> PAUSE** を守る
- 実装中の説明は軽く保つ
- change が小さくても phase を飛ばさない
- 指定された pause では確認を待つが、止まりすぎない
- 離脱したいユーザーを無理に引き留めない
- 実際のコードベース上の本物のタスクを使う
- スコープはやわらかく小さめへ導くが、最終判断はユーザーに任せる
