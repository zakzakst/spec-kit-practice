# [Custom Agents (.agent.md)](https://code.visualstudio.com/docs/copilot/customization/custom-agents)

特定のツール、指示、動作を持つカスタムペルソナ。役割ベースのツール制限を伴うオーケストレーションされたワークフローに使用します。

## ロケーション

| パス | スコープ |
|------|-------|
| `.github/agents/*.agent.md` | ワークスペース |
| `<profile>/agents/*.agent.md` | ユーザープロファイル |

## Frontmatter

```yaml
---
description: "<必須>"          # エージェントピッカーおよびサブエージェントの発見用
name: "Agent Name"           # 任意、デフォルトはファイル名
tools: [search, web]         # 任意: エイリアス、MCP (<server>/*)、拡張機能ツール
model: "Claude Sonnet 4"     # 任意、ピッカーのデフォルトを使用します。フォールバック用の配列をサポート
argument-hint: "タスク..."     # 任意、入力のガイダンス
agents: [agent1, agent2]     # 任意、許可されるサブエージェントを名前で制限（省略=すべて、[]=なし）
user-invocable: true         # 任意、エージェントピッカーに表示（デフォルト: true）
disable-model-invocation: false  # 任意、サブエージェントの呼び出しをブロック（デフォルト: false）
handoffs: [...]              # 任意、他のエージェントへのトランジション
hooks:                       # 任意、このエージェントのライフサイクルイベントに対するインラインフック
  PreToolUse:
    - type: command
      command: "./scripts/validate.sh"
  PostToolUse:
    - type: command
      command: "./scripts/format.sh"
---
```

### 呼び出しの制御

| 属性 | デフォルト | 効果 |
|-----------|---------|--------|
| `user-invocable: false` | `true` | エージェントピッカーから非表示にし、サブエージェントとしてのみアクセス可能にする |
| `disable-model-invocation: true` | `false` | 他のエージェントがサブエージェントとして呼び出すのを防ぐ |

### モデルのフォールバック

```yaml
model: ['Claude Sonnet 4.5 (copilot)', 'GPT-5 (copilot)']  # 最初に利用可能なモデルが使用されます
```

## ツール

ソース: 組み込みのエイリアス、特定のツール、MCPサーバー (`<server>/*`)、拡張機能ツール。

**特別**: `[]` = ツールなし、省略 = デフォルト。本文での参照: `#tool:<name>`

### ツールエイリアス

| エイリアス | 目的 |
|-------|---------|
| `execute` | シェルコマンドの実行 |
| `read` | ファイル内容の読み取り |
| `edit` | ファイルの編集 |
| `search` | ファイルまたはテキストの検索 |
| `agent` | サブエージェントとしてカスタムエージェントを呼び出す |
| `web` | URLの取得とWeb検索 |
| `todo` | タスクリストの管理 |

### 一般的なパターン

```yaml
tools: [read, search]             # 読み取り専用の調査
tools: [myserver/*]               # MCPサーバーのみ
tools: [read, edit, search]       # ターミナルアクセスなし
tools: []                         # 会話のみ
```

利用可能なツールを見つけるには、現在のツールリストを確認するか、本文で `#tool:` 構文を使用して特定のツールを参照してください。

## テンプレート

```markdown
---
description: "{使用するタイミング... サブエージェント発見のためのトリガーフレーズ}"
tools: [{ツールエイリアスの最小限のセット}]
user-invocable: false
---
あなたは {特定のタスク} のスペシャリストです。あなたの仕事は {明確な目的} を行うことです。

## 制約
- DO NOT {このエージェントが決して行うべきではないこと}
- DO NOT {別の制限事項}
- ONLY {このエージェントが行う唯一のこと}

## アプローチ
1. {このエージェントの仕組みのステップ1}
2. {ステップ2}
3. {ステップ3}

## 出力フォーマット
{このエージェントが正確に返すべきもの}
```

## 呼び出し方法

- **手動**: チャットでのエージェントセレクタ
- **サブエージェント**: `description` の一致に基づいて親エージェントが委任します（`infer` が許可する場合）

## 基本原則

1. **単一の役割**: エージェントごとに焦点を絞った責任を持つ1つのペルソナ
2. **最小限のツール**: その役割に必要なものだけを含めます。過剰なツールは焦点をぼかします
3. **明確な境界**: エージェントが実行しては*いけない*ことを定義します
4. **キーワードが豊富な説明**: 親エージェントがいつ委任すべきかを認識できるように、トリガーワードを含めます

## アンチパターン

- **スイスアーミーナイフ型エージェント**: ツールが多すぎ、すべてをやろうとする
- **曖昧な説明**: 「役立つエージェント」では委任の指針になりません。具体的に記述してください
- **役割の混乱**: 説明が本文のペルソナと一致しない
- **循環するハンドオフ**: 進捗の基準なしに A → B → A と循環する

## インラインフック

カスタムエージェントは、frontmatter 内のインライン `hooks` をサポートしています。これらのフックはエージェントのライフサイクルポイントでシェルコマンドを実行し、このエージェントのみにスコープされます。形式はスタンドアロンのフックファイル（[hooks リファレンス](../hooks.md)を参照）と同じです。

### サポートされているイベント

`SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PreCompact`, `SubagentStart`, `SubagentStop`, `Stop`

### 例

```yaml
---
description: "危険なコマンドをブロックするセキュアなコードレビュアー"
tools: [read, search, execute]
hooks:
  PreToolUse:
    - type: command
      command: "./scripts/block-dangerous-cmds.sh"
      timeout: 10
  PostToolUse:
    - type: command
      command: "./scripts/auto-lint.sh"
---
```

各フックコマンドは以下をサポートします: `type`（`command` である必要があります）、`command`、プラットフォームのオーバーライド（`windows`、`linux`、`osx`）、`cwd`、`env`、`timeout`。
