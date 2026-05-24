---
name: evolve
description: instinct を分析し、進化した構造を提案または生成する
command: true
---

# Evolve コマンド

## 実装

プラグインルートのパスを使って instinct CLI を実行します:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

または `CLAUDE_PLUGIN_ROOT` が設定されていない場合（手動インストール時）は:

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

instinct を分析し、関連するものをより高レベルな構造へ束ねます:

- **Commands**: instinct がユーザー起点の操作を表している場合
- **Skills**: instinct が自動発火すべき振る舞いを表している場合
- **Agents**: instinct が複雑で多段な処理を表している場合

## 使い方

```text
/evolve                    # すべての instinct を分析して進化候補を提案
/evolve --generate         # さらに evolved/{skills,commands,agents} 以下にファイルを生成
```

## 進化ルール

### -> Command（ユーザー起動）

ユーザーが明示的に依頼しそうな操作を instinct が表している場合:

- 「ユーザーが ... と頼んだら」のような instinct が複数ある
- 「新しい X を作るとき」のようなトリガーを持つ instinct
- 毎回同じような手順をたどる instinct

例:

- `new-table-step1`: "when adding a database table, create migration"
- `new-table-step2`: "when adding a database table, update schema"
- `new-table-step3`: "when adding a database table, regenerate types"

-> **new-table** コマンドを作る

### -> Skill（自動発火）

自動的に起きてほしい振る舞いを instinct が表している場合:

- パターン一致型のトリガー
- エラーハンドリングの反応
- コードスタイルの強制

例:

- `prefer-functional`: "when writing functions, prefer functional style"
- `use-immutable`: "when modifying state, use immutable patterns"
- `avoid-classes`: "when designing modules, avoid class-based design"

-> `functional-patterns` スキルを作る

### -> Agent（深さ / 分離が必要）

複雑で多段な処理で、分離した方がよい instinct の場合:

- デバッグワークフロー
- リファクタリング手順
- リサーチタスク

例:

- `debug-step1`: "when debugging, first check logs"
- `debug-step2`: "when debugging, isolate the failing component"
- `debug-step3`: "when debugging, create minimal reproduction"
- `debug-step4`: "when debugging, verify fix with test"

-> **debugger** エージェントを作る

## やること

1. 現在のプロジェクト文脈を検出する
2. project + global の instinct を読む（ID 衝突時は project 優先）
3. トリガー / ドメインのパターンで instinct をグループ化する
4. 以下を特定する:
   - Skill 候補（2 個以上の instinct を含むトリガークラスタ）
   - Command 候補（高信頼のワークフロー instinct）
   - Agent 候補（より大きく高信頼なクラスタ）
5. 必要に応じて昇格候補（project -> global）を表示する
6. `--generate` が渡された場合、以下にファイルを書き出す:
   - Project scope: `~/.claude/homunculus/projects/<project-id>/evolved/`
   - Global fallback: `~/.claude/homunculus/evolved/`

## 出力形式

```text
============================================================
  EVOLVE ANALYSIS - 12 instincts
  Project: my-app (a1b2c3d4e5f6)
  Project-scoped: 8 | Global: 4
============================================================

High confidence instincts (>=80%): 5

## SKILL CANDIDATES
1. Cluster: "adding tests"
   Instincts: 3
   Avg confidence: 82%
   Domains: testing
   Scopes: project

## COMMAND CANDIDATES (2)
  /adding-tests
    From: test-first-workflow [project]
    Confidence: 84%

## AGENT CANDIDATES (1)
  adding-tests-agent
    Covers 3 instincts
    Avg confidence: 82%
```

## フラグ

- `--generate`: 分析結果に加え、evolved ファイルも生成する

## 生成ファイル形式

### Command

```markdown
---
name: new-table
description: Create a new database table with migration, schema update, and type generation
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table Command

[クラスタ化された instinct に基づく生成内容]

## Steps

1. ...
2. ...
```

### Skill

```markdown
---
name: functional-patterns
description: Enforce functional programming patterns
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns Skill

[クラスタ化された instinct に基づく生成内容]
```

### Agent

```markdown
---
name: debugger
description: Systematic debugging agent
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger Agent

[クラスタ化された instinct に基づく生成内容]
```
