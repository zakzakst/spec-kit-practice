---
name: continuous-learning-v2
description: hook でセッションを観測し、信頼度付きの atomic instinct を作り、それらを skill / command / agent へ進化させる instinct-based learning system。v2.1 ではプロジェクト単位の instinct を追加し、プロジェクト間汚染を防ぎます。
origin: ECC
version: 2.1.0
---

# Continuous Learning v2.1 - Instinct ベースアーキテクチャ

Claude Code セッションを、信頼度付きの小さな learned behavior である atomic "instincts" へ変換し、再利用可能知識へ育てる高度な学習システムです。

**v2.1** では **project-scoped instincts** を追加しました。React の流儀は React プロジェクトに、Python の慣習は Python プロジェクトにとどまり、"always validate input" のような普遍的パターンだけが global に共有されます。

## 有効化するタイミング

- Claude Code セッションから自動学習を設定したいとき
- hook による instinct-based な振る舞い抽出を構成するとき
- 学習された振る舞いの confidence threshold を調整するとき
- instinct library を確認、export、import したいとき
- instinct を skill、command、agent へ進化させたいとき
- project-scoped と global の切り分けを管理したいとき
- instinct を project から global へ昇格させたいとき

## v2.1 の新要素

| Feature       | v2.0                             | v2.1                                                                                |
| ------------- | -------------------------------- | ----------------------------------------------------------------------------------- |
| Storage       | Global (`~/.claude/homunculus/`) | Project-scoped (`${XDG_DATA_HOME:-~/.local/share}/ecc-homunculus/projects/<hash>/`) |
| Scope         | All instincts apply everywhere   | Project-scoped + global                                                             |
| Detection     | None                             | git remote URL / repo path                                                          |
| Promotion     | N/A                              | Project -> global when seen in 2+ projects                                          |
| Commands      | 4 (status/evolve/export/import)  | 6 (+promote/projects)                                                               |
| Cross-project | Contamination risk               | Isolated by default                                                                 |

## v2 の新要素（vs v1）

| Feature     | v1                      | v2                                          |
| ----------- | ----------------------- | ------------------------------------------- |
| Observation | Stop hook (session end) | PreToolUse/PostToolUse (100% reliable)      |
| Analysis    | Main context            | Background agent (Haiku)                    |
| Granularity | Full skills             | Atomic "instincts"                          |
| Confidence  | None                    | 0.3-0.9 weighted                            |
| Evolution   | Direct to skill         | Instincts -> cluster -> skill/command/agent |
| Sharing     | None                    | Export/import instincts                     |

## Instinct モデル

instinct は小さな learned behavior:

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
scope: project
project_id: "a1b2c3d4e5f6"
project_name: "my-react-app"
---

# Prefer Functional Style

## Action
Use functional patterns over classes when appropriate.

## Evidence
- Observed 5 instances of functional pattern preference
- User corrected class-based approach to functional on 2025-01-15
```

**特性:**

- **Atomic** - 1 trigger, 1 action
- **Confidence-weighted** - 0.3 は tentative、0.9 はほぼ確実
- **Domain-tagged** - code-style, testing, git, debugging, workflow など
- **Evidence-backed** - どの観測で作られたかを追う
- **Scope-aware** - `project`（既定）または `global`

## 仕組み

```text
Session Activity (in a git repo)
      |
      | Hooks capture prompts + tool use (100% reliable)
      | + detect project context (git remote / repo path)
      v
projects/<project-hash>/observations.jsonl
      |
      | Observer agent reads (background, Haiku)
      v
PATTERN DETECTION
  * User corrections -> instinct
  * Error resolutions -> instinct
  * Repeated workflows -> instinct
  * Scope decision: project or global?
      |
      v
projects/<project-hash>/instincts/personal/
instincts/personal/  (GLOBAL)
      |
      | /evolve clusters + /promote
      v
projects/<hash>/evolved/  and  evolved/
```

## Project Detection

現在の project は自動検出される:

1. **`CLAUDE_PROJECT_DIR` env var**（最優先）
2. **`git remote get-url origin`** - portable な project ID を作るために hash 化
3. **`git rev-parse --show-toplevel`** - repo path ベースの fallback
4. **Global fallback** - project が検出できなければ global scope へ保存

各 project には 12 文字の hash ID（例: `a1b2c3d4e5f6`）が付く。`${XDG_DATA_HOME:-~/.local/share}/ecc-homunculus/projects.json` に ID と人間向け名称の対応を持つ。

### Data Directory

background instinct write が Claude Code の sensitive-path guard に引っかからないよう、observer data は `~/.claude` の外へ保存する:

1. `CLV2_HOMUNCULUS_DIR` が絶対パスで設定されている場合
2. `$XDG_DATA_HOME/ecc-homunculus`
3. `$HOME/.local/share/ecc-homunculus`

既存ユーザーで `~/.claude/homunculus` にデータがある場合は、一度だけ移行できる:

```bash
bash skills/continuous-learning-v2/scripts/migrate-homunculus.sh
```

## Quick Start

### 1. Observation Hook を有効化する

**plugin としてインストールした場合**（推奨）:

追加の `settings.json` hook block は不要。Claude Code v2.1+ は plugin の `hooks/hooks.json` を自動読込し、`observe.sh` はそこで登録済み。

以前に `observe.sh` を `~/.claude/settings.json` に手で入れていたなら、その重複した `PreToolUse` / `PostToolUse` block を削除する。plugin hook を重複登録すると二重実行と `${CLAUDE_PLUGIN_ROOT}` 解決エラーを招く。

**手動で** `~/.claude/skills` に入れた場合は、`~/.claude/settings.json` に以下を追加する:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
          }
        ]
      }
    ]
  }
}
```

### 2. ディレクトリ構造を初期化する

初回使用時に自動生成されるが、手動でも作成できる:

```bash
mkdir -p "${XDG_DATA_HOME:-$HOME/.local/share}/ecc-homunculus"/{instincts/{personal,inherited},evolved/{agents,skills,commands},projects}
```

project ディレクトリは git repo 内で hook が最初に走ると自動生成される。

### 3. Instinct コマンドを使う

```bash
/instinct-status
/evolve
/instinct-export
/instinct-import
/promote
/projects
```

## Commands

| Command                   | Description                                                        |
| ------------------------- | ------------------------------------------------------------------ |
| `/instinct-status`        | confidence 付きで全 instinct（project + global）を表示            |
| `/evolve`                 | 関連 instinct を cluster 化して skill / command を提案            |
| `/instinct-export`        | instinct を export（scope / domain で絞り込み可能）               |
| `/instinct-import <file>` | scope を制御しながら instinct を import                           |
| `/promote [id]`           | project instinct を global scope へ昇格                            |
| `/projects`               | 既知 project 一覧と instinct 件数を表示                           |

## Configuration

background observer の制御には `config.json` を編集する:

```json
{
  "version": "2.1",
  "observer": {
    "enabled": false,
    "run_interval_minutes": 5,
    "min_observations_to_analyze": 20
  }
}
```

| Key                                    | Default | Description                                  |
| -------------------------------------- | ------- | -------------------------------------------- |
| `observer.enabled`                     | `false` | background observer agent を有効化           |
| `observer.run_interval_minutes`        | `5`     | 観測分析の実行間隔                           |
| `observer.min_observations_to_analyze` | `20`    | 分析前に必要な最小観測数                     |

その他の挙動（observation capture、閾値、project scoping、promotion 条件など）は `instinct-cli.py` と `observe.sh` のコード既定値で管理される。

## File Structure

```text
${XDG_DATA_HOME:-~/.local/share}/ecc-homunculus/
+-- identity.json
+-- projects.json
+-- observations.jsonl
+-- instincts/
|   +-- personal/
|   +-- inherited/
+-- evolved/
|   +-- agents/
|   +-- skills/
|   +-- commands/
+-- projects/
    +-- a1b2c3d4e5f6/
    |   +-- project.json
    |   +-- observations.jsonl
    |   +-- observations.archive/
    |   +-- instincts/
    |   |   +-- personal/
    |   |   +-- inherited/
    |   +-- evolved/
    |       +-- skills/
    |       +-- commands/
    |       +-- agents/
```

## Scope Decision Guide

| Pattern Type                   | Scope       | Examples                                                 |
| ------------------------------ | ----------- | -------------------------------------------------------- |
| Language/framework conventions | **project** | "Use React hooks", "Follow Django REST patterns"         |
| File structure preferences     | **project** | "Tests in `__tests__`/", "Components in src/components/" |
| Code style                     | **project** | "Use functional style", "Prefer dataclasses"             |
| Error handling strategies      | **project** | "Use Result type for errors"                             |
| Security practices             | **global**  | "Validate user input", "Sanitize SQL"                    |
| General best practices         | **global**  | "Write tests first", "Always handle errors"              |
| Tool workflow preferences      | **global**  | "Grep before Edit", "Read before Write"                  |
| Git practices                  | **global**  | "Conventional commits", "Small focused commits"          |

## Instinct Promotion（Project -> Global）

同じ instinct が複数 project で高 confidence で観測されると、global への昇格候補になる。

**自動昇格条件:**

- 同じ instinct ID が 2 つ以上の project に存在
- 平均 confidence >= 0.8

**昇格方法:**

```bash
python3 instinct-cli.py promote prefer-explicit-errors
python3 instinct-cli.py promote
python3 instinct-cli.py promote --dry-run
```

`/evolve` でも昇格候補を提案する。

## Confidence Scoring

confidence は時間とともに変化する:

| Score | Meaning      | Behavior                      |
| ----- | ------------ | ----------------------------- |
| 0.3   | Tentative    | 提案はするが強制しない        |
| 0.5   | Moderate     | relevant なときに適用         |
| 0.7   | Strong       | 自動適用してよい水準          |
| 0.9   | Near-certain | 中核行動                      |

**Confidence が上がる条件**

- パターンが繰り返し観測される
- ユーザーが提案行動を修正しない
- 他の source と整合する instinct がある

**Confidence が下がる条件**

- ユーザーが明示的にその行動を修正する
- 長期間観測されない
- 反証が現れる

## なぜ Observation は Skill ではなく Hook か

Hook は **100%** 決定論的に発火するため:

- すべての tool call が観測される
- パターンを取りこぼさない
- 学習が包括的になる

## Backward Compatibility

v2.1 は v2.0 と v1 に完全互換:

- 既存 global instinct は `scripts/migrate-homunculus.sh` で移行可能
- v1 の `~/.claude/skills/learned/` もそのまま動く
- Stop hook も引き続き動く（ただし今は v2 にも流し込まれる）
- 段階的移行として並行運用もできる

## Privacy

- 観測データは **ローカルマシン内** に残る
- project-scoped instinct は project ごとに分離される
- export できるのは **instinct（パターン）だけ** で、生の観測ではない
- 実際のコードや会話内容は共有されない
- export や promote の範囲は自分で制御できる

## 関連

- [ECC-Tools GitHub App](https://github.com/apps/ecc-tools) - repo history から instinct を生成
- Homunculus - v2 instinct-based architecture の着想元となったコミュニティプロジェクト
- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - continuous learning セクション

---

_Instinct-based learning: Claude にあなたのパターンを、1 プロジェクトずつ教えていく。_
