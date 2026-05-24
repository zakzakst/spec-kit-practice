---
name: search-first
description: コーディング前に調査するワークフロー。カスタム実装を書く前に既存のツール、ライブラリ、パターンを探します。researcher agent を呼び出します。
origin: ECC
---

# /search-first - コードを書く前に調べる

「実装する前に既存解を探す」というワークフローを体系化します。

## トリガー

次の場面で使う:

- 既存解がありそうな新機能に着手するとき
- 依存関係や統合を追加するとき
- ユーザーが「X の機能を追加して」と言い、これからコードを書こうとしているとき
- 新しい utility、helper、abstraction を作る前

## ワークフロー

```text
0. TOOL AVAILABILITY PREFLIGHT
   使える検索チャネルを先に確認し、使えなかったものは正直に報告する

1. NEED ANALYSIS
   必要な機能を定義し、言語 / フレームワーク制約を把握する

2. PARALLEL SEARCH (researcher agent)
   npm / PyPI, MCP / Skills, GitHub / Web を並列探索する

3. EVALUATE
   候補を機能、保守性、コミュニティ、docs、license、依存関係で評価する

4. DECIDE
   Adopt / Extend / Build Custom のどれかを決める

5. IMPLEMENT
   package install / MCP 設定 / 最小の custom code を行う
```

## Decision Matrix

| Signal                                   | Action                                                  |
| ---------------------------------------- | ------------------------------------------------------- |
| Exact match, well-maintained, MIT/Apache | **Adopt** - install してそのまま使う                   |
| Partial match, good foundation           | **Extend** - install して薄い wrapper を書く           |
| Multiple weak matches                    | **Compose** - 小さな package を 2〜3 個組み合わせる   |
| Nothing suitable found                   | **Build** - 調査結果を踏まえて custom 実装を書く      |

## 使い方

### Step 0: Tool Availability Preflight

これは実行スクリプトではなく agent guidance。今目の前のタスクとプロジェクトに relevant なチャネルだけ確認する。

| Channel           | Check                                                                  | If missing                                               |
| ----------------- | ---------------------------------------------------------------------- | -------------------------------------------------------- |
| Repository search | `rg --files` と対象を絞った `rg`                                      | 見えているファイルしか調べていないと明示する            |
| Package registry  | `npm --version`, `python -m pip --version` など                        | web/docs search に切り替え、registry 全体を見たとは言わない |
| GitHub CLI        | `gh auth status`                                                       | public web または local git history のみに頼る          |
| MCP/docs tools    | 利用可能ツール一覧または local MCP config                             | official docs / web search にフォールバックする          |
| Skills directory  | 必要に応じて `ls ~/.claude/skills ~/.codex/skills`                    | local skill catalog がないと伝える                      |

### Quick Mode（頭の中で回す簡易版）

utility や新機能を書く前に、頭の中で次を確認する:

0. これは repo 内に既にあるか? -> まず relevant module / test を `rg` で探す
1. よくある問題か? -> npm / PyPI を探す
2. これ用の MCP はあるか? -> `~/.claude/settings.json` を見て探す
3. 対応する skill はあるか? -> `~/.claude/skills/` を見る
4. GitHub に実装 / template はあるか? -> net-new code の前に保守されている OSS を探す

### Full Mode（agent）

自明でない機能なら researcher agent を起動する:

```text
Agent(subagent_type="general-purpose", prompt="
  Research existing tools for: [DESCRIPTION]
  Language/framework: [LANG]
  Constraints: [ANY]

  Search: npm/PyPI, MCP servers, Claude Code skills, GitHub
  Return: Structured comparison with recommendation
")
```

古い Claude Code 文書では `Task(...)` と書かれている場合があるが、実際には現在の harness が公開している agent / subagent 名を使う。

## カテゴリ別検索ショートカット

### Development Tooling

- Linting - `eslint`, `ruff`, `textlint`, `markdownlint`
- Formatting - `prettier`, `black`, `gofmt`
- Testing - `jest`, `pytest`, `go test`
- Pre-commit - `husky`, `lint-staged`, `pre-commit`

### AI / LLM Integration

- Claude SDK - 最新 docs は Context7
- Prompt management - MCP servers を確認
- Document processing - `unstructured`, `pdfplumber`, `mammoth`

### Data & APIs

- HTTP clients - `httpx` (Python), `ky` / `undici` (Node)
- Validation - `zod` (TS), `pydantic` (Python)
- Database - まず MCP server の有無を確認

### Content & Publishing

- Markdown processing - `remark`, `unified`, `markdown-it`
- Image optimization - `sharp`, `imagemin`

## 統合ポイント

### planner agent と組み合わせる場合

planner は Phase 1（Architecture Review）の前に researcher を呼ぶべき:

- researcher が使えるツールを洗い出す
- planner がそれを実装計画に織り込む
- 「車輪の再発明」を plan の時点で防げる

### architect agent と組み合わせる場合

architect は次の判断で researcher を参照する:

- 技術スタック選定
- 統合パターンの発見
- 既存 reference architecture の探索

### iterative-retrieval skill と組み合わせる場合

段階的発見として組み合わせる:

- Cycle 1: 広い探索（npm、PyPI、MCP）
- Cycle 2: 上位候補の詳細評価
- Cycle 3: プロジェクト制約との適合性確認

## 例

### Example 1: "Add dead link checking"

```text
Need: Check markdown files for broken links
Search: npm "markdown dead link checker"
Found: textlint-rule-no-dead-link (score: 9/10)
Action: ADOPT - npm install textlint-rule-no-dead-link
Result: Zero custom code, battle-tested solution
```

### Example 2: "Add HTTP client wrapper"

```text
Need: Resilient HTTP client with retries and timeout handling
Search: npm "http client retry", PyPI "httpx retry"
Found: got (Node) with retry plugin, httpx (Python) with built-in retry
Action: ADOPT - use got/httpx directly with retry config
Result: Zero custom code, production-proven libraries
```

### Example 3: "Add config file linter"

```text
Need: Validate project config files against a schema
Search: npm "config linter schema", "json schema validator cli"
Found: ajv-cli (score: 8/10)
Action: ADOPT + EXTEND - install ajv-cli, write project-specific schema
Result: 1 package + 1 schema file, no custom validation logic
```

## アンチパターン

- **いきなりコードへ飛ぶ**: 既存物を調べず utility を書く
- **MCP を無視する**: すでに capability を持つ MCP server を確認しない
- **黙って省略する**: チャネル未利用なのに "nothing found" と報告する
- **過剰カスタマイズ**: library の利点が消えるほど厚く wrapper する
- **依存膨張**: 小機能のために巨大 package を入れる
