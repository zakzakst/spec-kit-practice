---
name: create-agent-adapter
description: >
  server、UI、CLI にまたがる Paperclip agent adapter を作成または変更します。
  新しい CLI agent、API agent、カスタム process、adapter package を追加するときに
  使います。
---

# Paperclip Agent Adapter の作成

adapter は、Paperclip の orchestration layer を特定の AI agent runtime
（Claude Code、Codex CLI、カスタム process、HTTP endpoint など）につなぎます。
各 adapter は self-contained な package で、**3つの consumer** である server、
UI、CLI 向けの実装を提供します。

---

## 1. アーキテクチャ概要

```
packages/adapters/<name>/
  src/
    index.ts            # Shared metadata (type, label, models, agentConfigurationDoc)
    server/
      index.ts          # Server exports: execute, sessionCodec, parse helpers
      execute.ts        # Core execution logic (AdapterExecutionContext -> AdapterExecutionResult)
      parse.ts          # Stdout/result parsing for the agent's output format
    ui/
      index.ts          # UI exports: parseStdoutLine, buildConfig
      parse-stdout.ts   # Line-by-line stdout -> TranscriptEntry[] for the run viewer
      build-config.ts   # CreateConfigValues -> adapterConfig JSON for agent creation form
    cli/
      index.ts          # CLI exports: formatStdoutEvent
      format-event.ts   # Colored terminal output for `paperclipai run --watch`
  package.json
  tsconfig.json
```

adapter module は、次の3つの registry から消費されます:

| Registry | Location | Interface |
|----------|----------|-----------|
| Server | `server/src/adapters/registry.ts` | `ServerAdapterModule` |
| UI | `ui/src/adapters/registry.ts` | `UIAdapterModule` |
| CLI | `cli/src/adapters/registry.ts` | `CLIAdapterModule` |

---

## 2. 共有型（`@paperclipai/adapter-utils`）

すべての adapter interface は `packages/adapter-utils/src/types.ts` にあります。
`@paperclipai/adapter-utils`（型）または `@paperclipai/adapter-utils/server-utils`
（runtime helper）から import します。

### 中核インターフェース

```ts
// The execute function signature — every adapter must implement this
interface AdapterExecutionContext {
  runId: string;
  agent: AdapterAgent;          // { id, companyId, name, adapterType, adapterConfig }
  runtime: AdapterRuntime;      // { sessionId, sessionParams, sessionDisplayId, taskKey }
  config: Record<string, unknown>;  // The agent's adapterConfig blob
  context: Record<string, unknown>; // Runtime context (taskId, wakeReason, approvalId, etc.)
  onLog: (stream: "stdout" | "stderr", chunk: string) => Promise<void>;
  onMeta?: (meta: AdapterInvocationMeta) => Promise<void>;
  authToken?: string;
}

interface AdapterExecutionResult {
  exitCode: number | null;
  signal: string | null;
  timedOut: boolean;
  errorMessage?: string | null;
  usage?: UsageSummary;           // { inputTokens, outputTokens, cachedInputTokens? }
  sessionId?: string | null;      // Legacy — prefer sessionParams
  sessionParams?: Record<string, unknown> | null;  // Opaque session state persisted between runs
  sessionDisplayId?: string | null;
  provider?: string | null;       // "anthropic", "openai", etc.
  model?: string | null;
  costUsd?: number | null;
  resultJson?: Record<string, unknown> | null;
  summary?: string | null;        // Human-readable summary of what the agent did
  clearSession?: boolean;         // true = tell Paperclip to forget the stored session
}

interface AdapterSessionCodec {
  deserialize(raw: unknown): Record<string, unknown> | null;
  serialize(params: Record<string, unknown> | null): Record<string, unknown> | null;
  getDisplayId?(params: Record<string, unknown> | null): string | null;
}
```

### モジュールインターフェース

```ts
// Server — registered in server/src/adapters/registry.ts
interface ServerAdapterModule {
  type: string;
  execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult>;
  testEnvironment(ctx: AdapterEnvironmentTestContext): Promise<AdapterEnvironmentTestResult>;
  sessionCodec?: AdapterSessionCodec;
  supportsLocalAgentJwt?: boolean;
  models?: { id: string; label: string }[];
  agentConfigurationDoc?: string;
}

// UI — registered in ui/src/adapters/registry.ts
interface UIAdapterModule {
  type: string;
  label: string;
  parseStdoutLine: (line: string, ts: string) => TranscriptEntry[];
  ConfigFields: ComponentType<AdapterConfigFieldsProps>;
  buildAdapterConfig: (values: CreateConfigValues) => Record<string, unknown>;
}

// CLI — registered in cli/src/adapters/registry.ts
interface CLIAdapterModule {
  type: string;
  formatStdoutEvent: (line: string, debug: boolean) => void;
}
```

---

## 2.1 Adapter Environment Test の契約

すべての server adapter は `testEnvironment(...)` を実装する必要があります。
これは agent 設定画面の board UI にある "Test environment" ボタンの動作に使われます。

```ts
type AdapterEnvironmentCheckLevel = "info" | "warn" | "error";
type AdapterEnvironmentTestStatus = "pass" | "warn" | "fail";

interface AdapterEnvironmentCheck {
  code: string;
  level: AdapterEnvironmentCheckLevel;
  message: string;
  detail?: string | null;
  hint?: string | null;
}

interface AdapterEnvironmentTestResult {
  adapterType: string;
  status: AdapterEnvironmentTestStatus;
  checks: AdapterEnvironmentCheck[];
  testedAt: string; // ISO timestamp
}

interface AdapterEnvironmentTestContext {
  companyId: string;
  adapterType: string;
  config: Record<string, unknown>; // runtime-resolved adapterConfig
}
```

ガイドライン:

- Return structured diagnostics, never throw for expected findings.
- Use `error` for invalid/unusable runtime setup (bad cwd, missing command, invalid URL).
- Use `warn` for non-blocking but important situations.
- Use `info` for successful checks and context.

severity policy は product-critical です。warning は save blocker ではありません。
例: `claude_local` で `ANTHROPIC_API_KEY` が見つかった場合は、Claude は
まだ実行できるので `error` ではなく `warn` にします（subscription auth の代わりに
API-key auth を使うだけです）。

---

## 3. 新しい adapter を作る手順

### 3.1 package を作成する

```
packages/adapters/<name>/
  package.json
  tsconfig.json
  src/
    index.ts
    server/index.ts
    server/execute.ts
    server/parse.ts
    ui/index.ts
    ui/parse-stdout.ts
    ui/build-config.ts
    cli/index.ts
    cli/format-event.ts
```

**package.json** — four-export convention を使います:

```json
{
  "name": "@paperclipai/adapter-<name>",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "exports": {
    ".": "./src/index.ts",
    "./server": "./src/server/index.ts",
    "./ui": "./src/ui/index.ts",
    "./cli": "./src/cli/index.ts"
  },
  "dependencies": {
    "@paperclipai/adapter-utils": "workspace:*",
    "picocolors": "^1.1.1"
  },
  "devDependencies": {
    "typescript": "^5.7.3"
  }
}
```

### 3.2 ルートの `index.ts` - adapter metadata

この file は **3つすべて** の consumer（server、UI、CLI）から import されます。
dependency-free に保ってください（Node API も React も使わない）。

```ts
export const type = "my_agent";        // snake_case, globally unique
export const label = "My Agent (local)";

export const models = [
  { id: "model-a", label: "Model A" },
  { id: "model-b", label: "Model B" },
];

export const agentConfigurationDoc = `# my_agent agent configuration
...document all config fields here...
`;
```

**必須 export:**
- `type` — `agents.adapter_type` に保存される adapter type key
- `label` — UI 向けの人間が読める名前
- `models` — agent 作成フォームで選べる model option
- `agentConfigurationDoc` — すべての `adapterConfig` field を説明する markdown
  （他の agent を設定する LLM agent が使います）

**`agentConfigurationDoc` を routing logic として書く:**

`agentConfigurationDoc` は、他の agent を作る Paperclip agents を含む LLM agents に
読まれます。marketing copy ではなく、**routing logic** として書いてください。
具体的な "use when" と "don't use when" を入れ、LLM がこの adapter が特定の task に
適切かどうか判断できるようにします。

```ts
export const agentConfigurationDoc = `# my_agent agent configuration

Adapter: my_agent

Use when:
- The agent needs to run MyAgent CLI locally on the host machine
- You need session persistence across runs (MyAgent supports thread resumption)
- The task requires MyAgent-specific tools (e.g. web search, code execution)

Don't use when:
- You need a simple one-shot script execution (use the "process" adapter instead)
- The agent doesn't need conversational context between runs (process adapter is simpler)
- MyAgent CLI is not installed on the host

Core fields:
- cwd (string, required): absolute working directory for the agent process
...
`;
```

明示的な否定条件を加えると、adapter 選択の精度が上がります。具体的な anti-pattern
1つは、説明文3段落より価値があります。

### 3.3 server module

#### `server/execute.ts` - 中核

これは最も重要な file です。`AdapterExecutionContext` を受け取り、
`AdapterExecutionResult` を返さなければなりません。

**必須の動作:**

1. **Read config** — extract typed values from `ctx.config` using helpers (`asString`, `asNumber`, `asBoolean`, `asStringArray`, `parseObject` from `@paperclipai/adapter-utils/server-utils`)
2. **Build environment** — call `buildPaperclipEnv(agent)` then layer in `PAPERCLIP_RUN_ID`, context vars (`PAPERCLIP_TASK_ID`, `PAPERCLIP_WAKE_REASON`, `PAPERCLIP_WAKE_COMMENT_ID`, `PAPERCLIP_APPROVAL_ID`, `PAPERCLIP_APPROVAL_STATUS`, `PAPERCLIP_LINKED_ISSUE_IDS`), user env overrides, and auth token
3. **Resolve session** — check `runtime.sessionParams` / `runtime.sessionId` for an existing session; validate it's compatible (e.g. same cwd); decide whether to resume or start fresh
4. **Render prompt** — use `renderTemplate(template, data)` with the template variables: `agentId`, `companyId`, `runId`, `company`, `agent`, `run`, `context`
5. **Call onMeta** — emit adapter invocation metadata before spawning the process
6. **Spawn the process** — use `runChildProcess()` for CLI-based agents or `fetch()` for HTTP-based agents
7. **Parse output** — convert the agent's stdout into structured data (session id, usage, summary, errors)
8. **Handle session errors** — if resume fails with "unknown session", retry with a fresh session and set `clearSession: true`
9. **Return AdapterExecutionResult** — populate all fields the agent runtime supports

**Environment variables the server always injects:**

| Variable | Source |
|----------|--------|
| `PAPERCLIP_AGENT_ID` | `agent.id` |
| `PAPERCLIP_COMPANY_ID` | `agent.companyId` |
| `PAPERCLIP_API_URL` | Server's own URL |
| `PAPERCLIP_RUN_ID` | Current run id |
| `PAPERCLIP_TASK_ID` | `context.taskId` or `context.issueId` |
| `PAPERCLIP_WAKE_REASON` | `context.wakeReason` |
| `PAPERCLIP_WAKE_COMMENT_ID` | `context.wakeCommentId` or `context.commentId` |
| `PAPERCLIP_APPROVAL_ID` | `context.approvalId` |
| `PAPERCLIP_APPROVAL_STATUS` | `context.approvalStatus` |
| `PAPERCLIP_LINKED_ISSUE_IDS` | `context.issueIds` (comma-separated) |
| `PAPERCLIP_API_KEY` | `authToken` (if no explicit key in config) |

#### `server/parse.ts` - output parser

agent の stdout format を structured data に parse します。次を扱える必要があります:

- **Session identification** — init event から session / thread ID を抽出します
- **Usage tracking** — token count（input / output / cached）を抽出します
- **Cost tracking** — 利用可能なら cost を抽出します
- **Summary extraction** — agent の最終テキスト応答を取り出します
- **Error detection** — error state を判定し、error message を抽出します
- **Unknown session detection** — retry logic 用に `is<Agent>UnknownSessionError()`
  function を export します

**agent の出力は untrusted として扱ってください。** 解析する stdout は、任意の tool call を
実行し、外部コンテンツを取得し、読んだ file の prompt injection の影響を受けている
可能性がある LLM 駆動 process から来ます。防御的に parse します:
- output から来たものを `eval()` したり、動的に実行したりしないでください
- safe extraction helper（`asString`、`asNumber`、`parseJson`）を使ってください。
  想定外の type では fallback を返します
- session ID やその他の structured data は、通す前に validate してください
- output に URL、file path、command が含まれていても、adapter では action せず、
  記録だけに留めてください

#### `server/index.ts` - server の export

```ts
export { execute } from "./execute.js";
export { testEnvironment } from "./test.js";
export { parseMyAgentOutput, isMyAgentUnknownSessionError } from "./parse.js";

// Session codec — session persistence に必要です
export const sessionCodec: AdapterSessionCodec = {
  deserialize(raw) { /* raw DB JSON -> typed params or null */ },
  serialize(params) { /* typed params -> JSON for DB storage */ },
  getDisplayId(params) { /* -> human-readable session id string */ },
};
```

#### `server/test.ts` - environment diagnostics

UI のテストボタンで使われる、adapter 固有の preflight check を実装します。

最低限の期待値:

1. 必須の config primitive（path、command、URL、auth 前提）を validate します
2. 決定論的な `code` 値を持つ check object を返します
3. severity を一貫して割り当てます（`info` / `warn` / `error`）
4. 最終 status を計算します:
   - 1つでも `error` があれば `fail`
   - `error` はなく、warning が 1つ以上あれば `warn`
   - それ以外は `pass`

この操作は軽量で、side effect がないことが必要です。

### 3.4 UI module

#### `ui/parse-stdout.ts` - transcript parser

個々の stdout line を、run detail viewer 用の `TranscriptEntry[]` に変換します。
agent の streaming output format を扱い、次の kind の entry を生成できる必要があります:

- `init` — model/session initialization
- `assistant` — agent text responses
- `thinking` — agent thinking/reasoning (if supported)
- `tool_call` — tool invocations with name and input
- `tool_result` — tool results with content and error flag
- `user` — user messages in the conversation
- `result` — final result with usage stats
- `stdout` — fallback for unparseable lines

```ts
export function parseMyAgentStdoutLine(line: string, ts: string): TranscriptEntry[] {
  // Parse JSON line, map to appropriate TranscriptEntry kind(s)
  // Return [{ kind: "stdout", ts, text: line }] as fallback
}
```

#### `ui/build-config.ts` - config builder

UI form の `CreateConfigValues` を、agent に保存される `adapterConfig` JSON blob に
変換します。

```ts
export function buildMyAgentConfig(v: CreateConfigValues): Record<string, unknown> {
  const ac: Record<string, unknown> = {};
  if (v.cwd) ac.cwd = v.cwd;
  if (v.promptTemplate) ac.promptTemplate = v.promptTemplate;
  if (v.model) ac.model = v.model;
  ac.timeoutSec = 0;
  ac.graceSec = 15;
  // ... adapter-specific fields
  return ac;
}
```

#### UI config fields component

`ui/src/adapters/<name>/config-fields.tsx` に、`AdapterConfigFieldsProps` を
実装する React component を作成します。これは agent の作成 / 編集 form に
adapter 固有の field を表示します。

`ui/src/components/agent-config-primitives` の共有 primitive を使います:
- `Field` — ラベル付き form field wrapper
- `ToggleField` — label と hint 付きの boolean toggle
- `DraftInput` — draft / commit 振る舞いを持つ text input
- `DraftNumberInput` — draft / commit 振る舞いを持つ number input
- `help` — 一般的な field 用の標準 hint text

component は `create` mode（`values` / `set` を使う）と `edit` mode（`config` /
`eff` / `mark` を使う）の両方をサポートしなければなりません。

### 3.5 CLI module

#### `cli/format-event.ts` - terminal formatter

`paperclipai run --watch` 用に stdout line を見やすく整形します。色付けには
`picocolors` を使います。

```ts
import pc from "picocolors";

export function printMyAgentStreamEvent(raw: string, debug: boolean): void {
  // Parse JSON line from agent stdout
  // Print colored output: blue for system, green for assistant, yellow for tools
  // In debug mode, print unrecognized lines in gray
}
```

---

## 4. 登録チェックリスト

adapter package を作成したら、3つすべての consumer に登録します:

### 4.1 server の登録（`server/src/adapters/registry.ts`）

```ts
import { execute as myExecute, sessionCodec as mySessionCodec } from "@paperclipai/adapter-my-agent/server";
import { agentConfigurationDoc as myDoc, models as myModels } from "@paperclipai/adapter-my-agent";

const myAgentAdapter: ServerAdapterModule = {
  type: "my_agent",
  execute: myExecute,
  sessionCodec: mySessionCodec,
  models: myModels,
  supportsLocalAgentJwt: true,  // true if agent can use Paperclip API
  agentConfigurationDoc: myDoc,
};

// Add to the adaptersByType map
const adaptersByType = new Map<string, ServerAdapterModule>(
  [..., myAgentAdapter].map((a) => [a.type, a]),
);
```

### 4.2 UI の登録（`ui/src/adapters/registry.ts`）

```ts
import { myAgentUIAdapter } from "./my-agent";

const adaptersByType = new Map<string, UIAdapterModule>(
  [..., myAgentUIAdapter].map((a) => [a.type, a]),
);
```

With `ui/src/adapters/my-agent/index.ts`:

```ts
import type { UIAdapterModule } from "../types";
import { parseMyAgentStdoutLine } from "@paperclipai/adapter-my-agent/ui";
import { MyAgentConfigFields } from "./config-fields";
import { buildMyAgentConfig } from "@paperclipai/adapter-my-agent/ui";

export const myAgentUIAdapter: UIAdapterModule = {
  type: "my_agent",
  label: "My Agent",
  parseStdoutLine: parseMyAgentStdoutLine,
  ConfigFields: MyAgentConfigFields,
  buildAdapterConfig: buildMyAgentConfig,
};
```

### 4.3 CLI の登録（`cli/src/adapters/registry.ts`）

```ts
import { printMyAgentStreamEvent } from "@paperclipai/adapter-my-agent/cli";

const myAgentCLIAdapter: CLIAdapterModule = {
  type: "my_agent",
  formatStdoutEvent: printMyAgentStreamEvent,
};

// Add to the adaptersByType map
```

---

## 5. セッション管理 - 長時間実行を前提に設計する

session によって、agent は run をまたいで conversation context を維持できます。
この system は **codec-based** です。各 adapter が session state の
serialize / deserialize 方法を定義します。

**最初から長時間実行を前提に設計してください。** session reuse は後で足す
optimization ではなく、既定の primitive として扱います。issue に取り組む agent は、
最初の割り当て、approval callback、再割り当て、手動の促しなどで何十回も wake される
可能性があります。各 wake では既存の conversation を再開し、すでに何をしたか、
どの file を読んだか、どんな決定をしたかを agent が保持できるようにします。
毎回最初から始めると、同じ file の再読みに token を無駄にし、矛盾した決定の
リスクが高まります。

**重要な概念:**
- `sessionParams` は task ごとに DB に保存される不透明な `Record<string, unknown>` です
- adapter の `sessionCodec.serialize()` は execution result data を保存可能な params に変換します
- `sessionCodec.deserialize()` は保存された params を次の run 用に戻します
- `sessionCodec.getDisplayId()` は UI 用の人間が読める session ID を取り出します
- **cwd-aware resume**: session が現在の config とは異なる cwd で作られた場合は
  resume しない（プロジェクトをまたいだ session 汚染を防ぐため）
- **Unknown session retry**: resume が "session not found" エラーで失敗したら、
  fresh session で再試行し、Paperclip が stale session を消すよう `clearSession: true` を返す

agent runtime が何らかの context compaction や conversation compression
（例: Claude Code の automatic context management、Codex の `previous_response_id`
 chaining）をサポートしているなら、それを使います。session resume をサポートする
adapter は compaction を自動で得られます。agent runtime が resume をまたいで
context window 管理を内部で扱ってくれるからです。

**pattern**（claude-local と codex-local の両方から）:

```ts
const canResumeSession =
  runtimeSessionId.length > 0 &&
  (runtimeSessionCwd.length === 0 || path.resolve(runtimeSessionCwd) === path.resolve(cwd));
const sessionId = canResumeSession ? runtimeSessionId : null;

// ... run attempt ...

// If resume failed with unknown session, retry fresh
if (sessionId && !proc.timedOut && exitCode !== 0 && isUnknownSessionError(output)) {
  const retry = await runAttempt(null);
  return toResult(retry, { clearSessionOnMissingSession: true });
}
```

---

## 6. Server-Utils helpers

`@paperclipai/adapter-utils/server-utils` から import します:

| Helper | Purpose |
|--------|---------|
| `asString(val, fallback)` | Safe string extraction |
| `asNumber(val, fallback)` | Safe number extraction |
| `asBoolean(val, fallback)` | Safe boolean extraction |
| `asStringArray(val)` | Safe string array extraction |
| `parseObject(val)` | Safe `Record<string, unknown>` extraction |
| `parseJson(str)` | Safe JSON.parse returning `Record` or null |
| `renderTemplate(tmpl, data)` | `{{path.to.value}}` template rendering |
| `buildPaperclipEnv(agent)` | Standard `PAPERCLIP_*` env vars |
| `redactEnvForLogs(env)` | Redact sensitive keys for onMeta |
| `ensureAbsoluteDirectory(cwd)` | Validate cwd exists and is absolute |
| `ensureCommandResolvable(cmd, cwd, env)` | Validate command is in PATH |
| `ensurePathInEnv(env)` | Ensure PATH exists in env |
| `runChildProcess(runId, cmd, args, opts)` | Spawn with timeout, logging, capture |

---

## 7. 規約とパターン

### 命名
- Adapter type: `snake_case` (e.g. `claude_local`, `codex_local`)
- Package name: `@paperclipai/adapter-<kebab-name>`
- Package directory: `packages/adapters/<kebab-name>/`

### config の parse
- `config` 値を直接信じないでください。必ず `asString`、`asNumber` などを使います。
- 任意 field にはすべて妥当な default を用意します。
- すべての field を `agentConfigurationDoc` に記載します。

### prompt template
- すべての run で `promptTemplate` をサポートします
- 標準の variable set とともに `renderTemplate()` を使います
- default prompt には `@paperclipai/adapter-utils/server-utils` の
  `DEFAULT_PAPERCLIP_AGENT_PROMPT_TEMPLATE` を使い、local adapter が Paperclip の
  execution contract を共有するようにします。つまり、同じ heartbeat で行動し、
  要求されていない限り plan-only の終了を避け、持続的な進捗と次の action を残し、
  polling ではなく child issue を使い、blocker には owner / action を付け、
  governance boundary を尊重します。

### error handling
- timeout、process error、parse failure を区別します
- failure では常に `errorMessage` を入れます
- parse に失敗したときは raw stdout/stderr を `resultJson` に含めます
- agent CLI が入っていない場合（command not found）を処理します

### logging
- すべての process output に対して `onLog("stdout", ...)` と `onLog("stderr", ...)` を呼び、
  real-time run viewer に流します
- spawn 前に `onMeta(...)` を呼んで invocation details を記録します
- meta に env を含めるときは `redactEnvForLogs()` を使います

### Paperclip Skills の注入

Paperclip は、runtime 中に agent が必要とする共有 skill を
（repo のトップレベル `skills/` ディレクトリに）同梱しています。
たとえば `paperclip` API skill や `paperclip-create-agent` workflow skill です。
各 adapter は、agent の working directory を汚さずに、これらの skill を
agent runtime から見つけられるようにする責任があります。

**制約:** skill を agent の `cwd` に copy したり symlink したりしないでください。
cwd はユーザーの project checkout です。そこへ `.claude/skills/` や他の file を
書き込むと、repo を Paperclip internals で汚し、git status を壊し、commit に
入り込む危険があります。

**pattern:** skill 用に clean で isolated な場所を作り、agent runtime にそこを
見るよう伝えます。

**claude-local のやり方:**

1. At execution time, create a fresh tmpdir: `mkdtemp("paperclip-skills-")`
2. Inside it, create `.claude/skills/` (the directory structure Claude Code expects)
3. Symlink each skill directory from the repo's `skills/` into the tmpdir's `.claude/skills/`
4. Pass the tmpdir to Claude Code via `--add-dir <tmpdir>` — this makes Claude Code discover the skills as if they were registered in that directory, without touching the agent's actual cwd
5. Clean up the tmpdir in a `finally` block after the run completes

```ts
// From claude-local execute.ts
async function buildSkillsDir(): Promise<string> {
  const tmp = await fs.mkdtemp(path.join(os.tmpdir(), "paperclip-skills-"));
  const target = path.join(tmp, ".claude", "skills");
  await fs.mkdir(target, { recursive: true });
  const entries = await fs.readdir(PAPERCLIP_SKILLS_DIR, { withFileTypes: true });
  for (const entry of entries) {
    if (entry.isDirectory()) {
      await fs.symlink(
        path.join(PAPERCLIP_SKILLS_DIR, entry.name),
        path.join(target, entry.name),
      );
    }
  }
  return tmp;
}

// In execute(): pass --add-dir to Claude Code
const skillsDir = await buildSkillsDir();
args.push("--add-dir", skillsDir);
// ... run process ...
// In finally: fs.rm(skillsDir, { recursive: true, force: true })
```

**codex-local のやり方:**

Codex には、グローバルな personal skills directory
（`$CODEX_HOME/skills` または `~/.codex/skills`）があります。adapter は、
Paperclip skill がまだ存在しない場合にそこへ symlink します。これは agent tool
自身の config directory であり、ユーザーの project ではないため許容されます。

```ts
// From codex-local execute.ts
async function ensureCodexSkillsInjected(onLog) {
  const skillsHome = path.join(codexHomeDir(), "skills");
  await fs.mkdir(skillsHome, { recursive: true });
  for (const entry of entries) {
    const target = path.join(skillsHome, entry.name);
    const existing = await fs.lstat(target).catch(() => null);
    if (existing) continue;  // Don't overwrite user's own skills
    await fs.symlink(source, target);
  }
}
```

**新しい adapter では:** agent runtime が skills / plugins をどう見つけるかを調べ、
いちばん clean な注入 path を選びます:

1. **最善: tmpdir + flag**（claude-local と同様）- runtime が "追加ディレクトリ" flag を
   サポートしているなら、tmpdir を作り、skill を symlink し、flag を渡して、後始末します。
   副作用はありません。
2. **許容: global config dir**（codex-local と同様）- runtime に project とは別の
   global skills/plugins directory があるなら、そこへ symlink します。既存 entry は飛ばし、
   ユーザーの custom 設定を上書きしないようにします。
3. **許容: env var** - runtime が environment variable から skills/plugin path を読むなら、
   repo の `skills/` ディレクトリを直接指します。
4. **最後の手段: prompt injection** - runtime に plugin system がないなら、
   skill の内容を prompt template 自体に含めます。token は消費しますが、filesystem への
   副作用は避けられます。

**skill は prompt の膨らみではなく、読み込まれる procedure として扱います。**
Paperclip skill（`paperclip` や `paperclip-create-agent` など）は on-demand の
procedure として設計されています。agent は context の中で skill metadata
（name + description）を見るだけで、skill を使うと決めたときにのみ完全な
SKILL.md を読み込みます。これにより base prompt を小さく保てます。
adapter 用の `agentConfigurationDoc` や prompt template を書くときは、skill content を
inline しないでください。agent runtime の skill discovery に任せます。
各 SKILL.md の frontmatter にある description は routing logic として働きます。
skill の中身ではなく、いつ full skill を読むべきかを agent に伝えます。

**明示的な skill 呼び出しと、あいまいな呼び出し。** 信頼性が重要な production
workflow（たとえば、status を報告するために必ず Paperclip API を呼ぶ必要がある agent）
では、prompt template に明示的な指示を書きます:
"Use the paperclip skill to report your progress."  description の一致に任せる
fuzzy routing は探索的な task には十分ですが、必須手順には不安定です。

---

## 8. セキュリティ上の考慮事項

adapter は Paperclip の orchestration layer と任意の agent execution の境界にあります。
これは high-risk な surface です。

### agent output は untrusted として扱う

agent process は、外部 file を読み、URL を取得し、tool を実行する LLM 駆動 code を
走らせます。その出力は、処理した content からの prompt injection の影響を受ける
可能性があります。adapter の parse layer は trust boundary です。すべてを検証し、
何も実行しないでください。

### secret は prompt ではなく environment で注入する

secret（API key、token）を、LLM を通る prompt template や config field に
入れないでください。代わりに、agent の tool が直接読める environment variable として
注入します:

- `PAPERCLIP_API_KEY` は prompt ではなく server が process environment に注入します
- `config.env` の user-provided secret は env var として渡し、`onMeta` logs では redaction します
- `redactEnvForLogs()` helper は `/(key|token|secret|password|authorization|cookie)/i`
  に一致する key を自動で mask します

これは "sidecar injection" pattern に従っています。model は real secret value を
見ませんが、呼び出した tool は environment から読めます。

### network access

agent runtime が network access control（sandboxing、allowlist）をサポートしているなら、
adapter で設定します:

- open internet access よりも最小限の allowlist を優先します。Paperclip API と GitHub
  だけ呼べればよい agent に、任意の host へのアクセス権は不要です。
- Skills + network はリスクを増幅します。HTTP request を教える skill と無制限の
  network access を組み合わせると、exfiltration の経路になります。どちらかを制限してください。
- runtime が layered policy（org-level defaults + per-request overrides）をサポートしているなら、
  org-level policy を adapter config に組み込み、per-agent config でさらに絞り込みます。

### process isolation

- CLI ベースの adapter は server の user permission を継承します。`cwd` と `env`
  config が、agent process が filesystem 上で何にアクセスできるかを決めます。
- `dangerouslySkipPermissions` / `dangerouslyBypassApprovalsAndSandbox` flag は
  開発の便宜のために存在しますが、`agentConfigurationDoc` で危険だと明記する必要があります。
  production deployment では使わないでください。
- timeout と grace period（`timeoutSec`、`graceSec`）は safety rail です。必ず適用してください。
  timeout のない runaway agent process は無制限に resource を消費できます。

---

## 9. TranscriptEntry kind の参考

UI run viewer は次の entry kind を表示します:

| Kind | Fields | 用途 |
|------|--------|-------|
| `init` | `model`, `sessionId` | agent 初期化 |
| `assistant` | `text` | agent のテキスト応答 |
| `thinking` | `text` | agent の reasoning / thinking |
| `user` | `text` | user メッセージ |
| `tool_call` | `name`, `input` | tool 呼び出し |
| `tool_result` | `toolUseId`, `content`, `isError` | tool の結果 |
| `result` | `text`, `inputTokens`, `outputTokens`, `cachedTokens`, `costUsd`, `subtype`, `isError`, `errors` | usage を含む最終結果 |
| `stderr` | `text` | stderr 出力 |
| `system` | `text` | system メッセージ |
| `stdout` | `text` | raw stdout の fallback |

---

## 10. テスト

`server/src/__tests__/<adapter-name>-adapter.test.ts` に test を作成します。テストするのは:

1. **Output parsing** — サンプル stdout を parser に通し、structured output を確認します
2. **Unknown session detection** — `is<Agent>UnknownSessionError` function を確認します
3. **Config building** — `buildConfig` が form values から正しい adapterConfig を作ることを確認します
4. **Session codec** — serialize / deserialize の往復を確認します

---

## 11. 最小 adapter チェックリスト

- [ ] `packages/adapters/<name>/package.json` に four exports（`.`, `./server`, `./ui`, `./cli`）がある
- [ ] Root の `index.ts` に `type`、`label`、`models`、`agentConfigurationDoc` がある
- [ ] `server/execute.ts` が `AdapterExecutionContext -> AdapterExecutionResult` を実装している
- [ ] `server/test.ts` が `AdapterEnvironmentTestContext -> AdapterEnvironmentTestResult` を実装している
- [ ] `server/parse.ts` に output parser と unknown-session detector がある
- [ ] `server/index.ts` が `execute`、`testEnvironment`、`sessionCodec`、parse helper を export している
- [ ] `ui/parse-stdout.ts` に run viewer 用の `StdoutLineParser` がある
- [ ] `ui/build-config.ts` に `CreateConfigValues -> adapterConfig` builder がある
- [ ] `ui/src/adapters/<name>/config-fields.tsx` に agent form 用 React component がある
- [ ] `ui/src/adapters/<name>/index.ts` で `UIAdapterModule` を組み立てている
- [ ] `cli/format-event.ts` に terminal formatter がある
- [ ] `cli/index.ts` が formatter を export している
- [ ] `server/src/adapters/registry.ts` に登録されている
- [ ] `ui/src/adapters/registry.ts` に登録されている
- [ ] `cli/src/adapters/registry.ts` に登録されている
- [ ] root の `pnpm-workspace.yaml` で workspace に追加されている
  （glob ですでに含まれていない場合）
- [ ] parsing、session codec、config building のテストがある
