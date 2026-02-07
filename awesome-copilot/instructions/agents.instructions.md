---
description: "GitHub Copilot のカスタムエージェントファイルを作成するためのガイドライン"
applyTo: "**/*.agent.md"
---

# カスタムエージェントファイルのガイドライン

GitHub Copilot の特定の開発タスクに関する専門知識を提供する、効果的で保守可能なカスタム エージェント ファイルを作成する手順。

## プロジェクトの背景

- 対象者: GitHub Copilot 用のカスタムエージェントを作成する開発者
- ファイル形式: YAML フロントマター付き Markdown
- ファイル名の命名規則: 小文字とハイフン (例: `test-specialist.agent.md`)
- 場所: `.github/agents/` ディレクトリ (リポジトリ レベル) または `agents/` ディレクトリ (組織/エンタープライズ レベル)
- 目的: 特定のタスクに合わせた専門知識、ツール、指示を備えた専門エージェントを定義する
- 公式ドキュメント: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents

## 必須の前文

すべてのエージェントファイルには、次のフィールドを含むYAMLフロントマターを含める必要があります。:

```yaml
---
description: "エージェントの目的と機能の簡単な説明"
name: "Agent Display Name"
tools: ["read", "edit", "search"]
model: "Claude Sonnet 4.5"
target: "vscode"
infer: true
---
```

### コアフロントマタープロパティ

#### **description** (必須)

- エージェントの目的とドメインの専門知識を明確に示す一重引用符で囲まれた文字列
- 簡潔（50～150文字）で実用的な内容であること
- 例: 「テストの範囲、品質、テストのベストプラクティスに焦点を当てます」

#### **name** (オプション)

- UI でのエージェントの表示名
- 省略された場合は、デフォルトでファイル名（`.md` または `.agent.md` なし）になります。
- タイトルを大文字にして説明的にする
- 例: `'Testing Specialist'`

#### **tools** (オプション)

- エージェントが使用できるツール名またはエイリアスのリスト
- カンマ区切りの文字列またはYAML配列形式をサポート
- 省略した場合、エージェントは利用可能なすべてのツールにアクセスできます
- 詳細については、以下の「ツール設定」セクションを参照してください。

#### **model** (STRONGLY RECOMMENDED)

- エージェントが使用するAIモデルを指定します
- VS Code、JetBrains IDE、Eclipse、Xcode でサポートされています
- 例: `'Claude Sonnet 4.5'`, `'gpt-4'`, `'gpt-4o'`
- エージェントの複雑さと必要な機能に基づいて選択する

#### **target** (オプション)

- ターゲット環境を指定します: `'vscode'` または `'github-copilot'`
- 省略した場合、エージェントは両方の環境で使用可能です
- エージェントに環境固有の機能がある場合に使用します

#### **infer** (オプション)

- Copilot がコンテキストに基づいてこのエージェントを自動的に使用できるかどうかを制御するブール値
- デフォルト: 省略した場合は `true`
- エージェントを手動で選択する必要がある場合は `false` に設定します

#### **metadata** (オプション, GitHub.comのみ)

- エージェント注釈の名前と値のペアを持つオブジェクト
- 例: `metadata: { category: 'testing', version: '1.0' }`
- VS Codeではサポートされていません

#### **mcp-servers** (オプション, Organization/Enterpriseのみ)

- このエージェントのみが利用できるMCPサーバーを構成する
- 組織/エンタープライズレベルのエージェントのみサポートされます
- 下記の「MCPサーバーの構成」セクションを参照してください。

#### **handoffs** (オプション, VS Codeのみ)

- エージェント間の遷移と次のステップの提案を行うガイド付きシーケンシャルワークフローを有効にする
- ハンドオフ設定のリスト（それぞれターゲットエージェントとオプションのプロンプトを指定）
- チャットの応答が完了すると、ハンドオフボタンが表示され、ユーザーは次のエージェントに移動できます。
- VS Code（バージョン 1.106 以降）でのみサポートされます
- 詳細については、以下の「ハンドオフ設定」セクションを参照してください。

## ハンドオフ設定

ハンドオフを使用すると、カスタムエージェント間をシームレスに遷移するガイド付きのシーケンシャルワークフローを作成できます。これは、ユーザーが次のステップに進む前に各ステップをレビューして承認できる、複数ステップの開発ワークフローをオーケストレーションするのに役立ちます。

### 一般的なハンドオフパターン

- **計画 → 実行**: 計画エージェントで計画を生成し、実装エージェントに渡してコーディングを開始します。
- **実装 → レビュー**: 実装を完了したら、コードレビューエージェントに切り替えて品質とセキュリティの問題をチェックします。
- **失敗するテストを書く → 成功するテストを書く**: 失敗するテストを生成し、それらのテストをパスさせるコードを実装するために引き渡す
- **研究 → 文書化**: トピックを調査し、その後、ガイドを作成するためにドキュメンテーションエージェントに移行します

### ハンドオフ前文構造

エージェントファイルのYAMLフロントマターで「handoffs」フィールドを使用してハンドオフを定義します:

```yaml
---
description: "エージェントの簡単な説明"
name: "Agent Name"
tools: ["search", "read"]
handoffs:
  - label: 実装を開始する
    agent: implementation
    prompt: "次に、上記で概説した計画を実行します。"
    send: false
  - label: コードレビュー
    agent: code-review
    prompt: "品質とセキュリティの問題がないか実装を確認してください。"
    send: false
---
```

### ハンドオフプロパティ

リスト内の各ハンドオフには次のプロパティが含まれている必要があります:

| プロパティ | タイプ  | 必須 | 説明                                                                                        |
| ---------- | ------- | ---- | ------------------------------------------------------------------------------------------- |
| `label`    | string  | Yes  | チャットインターフェースのハンドオフボタンに表示される表示テキスト                          |
| `agent`    | string  | Yes  | 切り替え先のターゲットエージェント識別子（名前または`.agent.md` のないファイル名）          |
| `prompt`   | string  | No   | 対象エージェントのチャット入力に事前入力するプロンプトテキスト                              |
| `send`     | boolean | No   | `true` の場合、プロンプトをターゲットエージェントに自動的に送信します (デフォルト: `false`) |

### ハンドオフ動作

- **ボタン表示**: チャットの応答が完了すると、対話型の提案としてハンドオフ ボタンが表示されます。
- **コンテキストの保存**: ユーザーがハンドオフボタンを選択すると、会話のコンテキストを維持したままターゲットエージェントに切り替わります。
- **事前入力されたプロンプト**: `prompt` が指定されている場合は、ターゲットエージェントのチャット入力に事前に入力されて表示されます。
- **手動 vs 自動**: `send: false` の場合、ユーザーは事前に入力されたプロンプトを確認して手動で送信する必要があります。`send: true` の場合、プロンプトは自動的に送信されます。

### ハンドオフ構成ガイドライン

#### ハンドオフを使用するタイミング

- **複数ステップのワークフロー**: 複雑なタスクを専門のエージェントに分割する
- **品質ゲート**: 実装フェーズ間のレビュー手順の確保
- **ガイド付きプロセス**: 構造化された開発プロセスを通じてユーザーを誘導する
- **スキルの移行**: 計画・設計から実装・テストの専門家へ

#### ベストプラクティス

- **クリアラベル**: 次のステップを明確に示すアクション指向のラベルを使用する
  - ✅ Good: 「実装の開始」、「セキュリティのレビュー」、「テストの作成」
  - ❌ Avoid: "Next", 「エージェントに行く」、「何かする」

- **関連するプロンプト**: 完了した作業を参照するコンテキスト認識プロンプトを提供する
  - ✅ Good: `'次に、上記で概説した計画を実行します。'`
  - ❌ Avoid: 文脈のない一般的なプロンプト

- **選択的使用**: あらゆるエージェントに引き継ぎを作成せず、論理的なワークフローの遷移に焦点を当てます
  - エージェントごとに最も関連性の高い次のステップを 2～3 つに制限します
  - ワークフローに自然に従うエージェントにのみハンドオフを追加します

- **エージェントの依存関係**: ハンドオフを作成する前にターゲットエージェントが存在することを確認する
  - 存在しないエージェントへのハンドオフは無視されます
  - ハンドオフをテストして期待通りに動作するか確認する

- **プロンプト内容**: プロンプトは簡潔かつ実行可能なものにする
  - コンテンツを重複させずに現在のエージェントの作業を参照する
  - 対象エージェントが必要とする可能性のあるコンテキストを提供する

### 例: 完成したワークフロー

3人のエージェントがハンドオフを行い、完全なワークフローを作成する例を示します:

**計画エージェント** (`planner.agent.md`):

```yaml
---
description: '新機能やリファクタリングの実装計画を作成する'
name: 'Planner'
tools: ['search', 'read']
handoffs:
  - label: 計画を実行する
    agent: implementer
    prompt: '上記の計画を実行します。'
    send: false
---
# 計画エージェント
あなたはプランニングスペシャリストです。あなたの仕事は:
1. 要件を分析する
2. 作業を論理的なステップに分解する
3. 詳細な実装計画を作成する
4. テスト要件を特定する

コードを書かずに、計画だけに集中してください。
```

**実装エージェント** (`implementer.agent.md`):

```yaml
---
description: '計画や仕様に基づいてコードを実装する'
name: 'Implementer'
tools: ['read', 'edit', 'search', 'execute']
handoffs:
  - label: 実装のレビュー
    agent: reviewer
    prompt: 'コードの品質、セキュリティ、ベスト プラクティスの遵守については、この実装を確認してください。'
    send: false
---
# 実装エージェント
あなたは実装スペシャリストです。あなたの仕事は:
1. 提供された計画または仕様に従う
2. クリーンで保守しやすいコードを書く
3. 適切なコメントとドキュメントを含める
4. プロジェクトのコーディング標準に従う

ソリューションを完全に徹底的に実装します。
```

**レビューエージェント** (`reviewer.agent.md`):

```yaml
---
description: '品質、セキュリティ、ベストプラクティスについてコードをレビューする'
name: 'Reviewer'
tools: ['read', 'search']
handoffs:
  - label: 計画に戻る
    agent: planner
    prompt: '上記のフィードバックを確認し、新しい計画が必要かどうかを判断します。'
    send: false
---
# コードレビューエージェント
あなたはコードレビューの専門家です。あなたの仕事は:
1. コードの品質と保守性をチェックする
2. セキュリティ上の問題と脆弱性を特定する
3. プロジェクト標準の遵守を確認する
4. 改善を提案する

実装に関する建設的なフィードバックを提供します。
```

このワークフローにより、開発者は:

1. プランナーエージェントを使って詳細な計画を作成しましょう
2. 計画に基づいてコードを書くために実装エージェントに引き渡す
3. 実装を確認するためにレビュー担当者に引き継ぎます
4. 重大な問題が見つかった場合は、必要に応じて計画に引き継ぐ

### バージョンの互換性

- **VS Code**: ハンドオフはVS Code 1.106以降でサポートされています
- **GitHub.com**: 現在サポートされていません。エージェント移行ワークフローでは異なるメカニズムが使用されます
- **Other IDEs**: サポートは限定的または全くなく、最大限の互換性のために VS Code 実装に重点を置いています

## ツール構成

### ツール仕様戦略

**すべてのツールを有効にする** (default):

```yaml
# toolsプロパティを完全に省略するか、下記の記述をする:
tools: ["*"]
```

**特定のツールを有効にする**:

```yaml
tools: ["read", "edit", "search", "execute"]
```

**MCPサーバーツールを有効にする**:

```yaml
tools: ["read", "edit", "github/*", "playwright/navigate"]
```

**すべてのツールを無効にする**:

```yaml
tools: []
```

### 標準ツールエイリアス

すべてのエイリアスは大文字と小文字を区別しません:

| エイリアス | 別名                                 | カテゴリ         | 説明                                         |
| ---------- | ------------------------------------ | ---------------- | -------------------------------------------- |
| `execute`  | shell, Bash, powershell              | Shell execution  | 適切なシェルでコマンドを実行する             |
| `read`     | Read, NotebookRead, view             | File reading     | ファイルの内容を読み取る                     |
| `edit`     | Edit, MultiEdit, Write, NotebookEdit | File editing     | ファイルの編集と変更                         |
| `search`   | Grep, Glob, search                   | Code search      | ファイルまたはファイル内のテキストを検索する |
| `agent`    | custom-agent, Task                   | Agent invocation | 他のカスタムエージェントを呼び出す           |
| `web`      | WebSearch, WebFetch                  | Web access       | ウェブコンテンツを取得して検索する           |
| `todo`     | TodoWrite                            | Task management  | タスク リストの作成と管理 (VS Code のみ)     |

### 組み込みのMCPサーバーツール

**GitHub MCP Server**:

```yaml
tools: ['github/*']  # すべてのGitHubツール
tools: ['github/get_file_contents', 'github/search_repositories']  # 特定のツール
```

- すべての読み取り専用ツールはデフォルトで使用可能
- ソースリポジトリにスコープされたトークン

**Playwright MCP Server**:

```yaml
tools: ['playwright/*']  # すべてのPlaywrightツール
tools: ['playwright/navigate', 'playwright/screenshot']  # 特定のツール
```

- ローカルホストのみにアクセスするように設定
- ブラウザの自動化とテストに役立ちます

### ツール選択のベストプラクティス

- **最小権限の原則**: エージェントの目的に必要なツールのみを有効にする
- **セキュリティ**: 明示的に要求されない限り、`execute` アクセスを制限する
- **集中**: ツールが少ない = エージェントの目的が明確になり、パフォーマンスが向上する
- **ドキュメント化**: 複雑な構成に特定のツールが必要な理由をコメントしてください

## サブエージェントの呼び出し（エージェントオーケストレーション）

エージェントは、**エージェント呼び出しツール** (`エージェント` ツール) を使用して他のエージェントを呼び出し、複数ステップのワークフローを調整できます。

推奨されるアプローチは**プロンプトベースのオーケストレーション**です:

- オーケストレーターは、自然言語でステップバイステップのワークフローを定義します。
- 各ステップは専門のエージェントに委任されます。
- オーケストレーターは、必須のコンテキスト (基本パス、識別子など) のみを渡し、各サブエージェントにツール/制約の独自の `.agent.md` 仕様を読み取ることを要求します。

### 仕組み

1. オーケストレーターのツールリストに `agent` を含めることでエージェントの呼び出しを有効にします:

```yaml
tools: ["read", "edit", "search", "agent"]
```

2. 各ステップで、以下を指定してサブエージェントを起動します。:

- **エージェント名** （ユーザーが選択/呼び出す識別子）
- **エージェント仕様パス** （読んで従うべき `.agent.md` ファイル）
- **最小限の共有コンテキスト** （例: `basePath`、`projectName`、`logFile`）

### プロンプトパターン（推奨）

サブエージェントが予測通りに動作するように、すべてのステップで一貫した「ラッパープロンプト」を使用します:

```text
このフェーズは、「<AGENT_SPEC_PATH>」で定義されたエージェント「<AGENT_NAME>」として実行する必要があります。

重要:
- .agent.md 仕様全体 (ツール、制約、品質基準) を読んで適用します。
- ベース パスが「<BASE_PATH>」である「<WORK_UNIT_NAME>」で作業します。
- この基本パスの下で必要な読み取り/書き込みを実行します。
- 明確な概要を返します (実行されたアクション + 作成/変更されたファイル + 問題)。
```

オプション: 追跡可能性のために軽量で構造化されたラッパーが必要な場合は、プロンプトに小さな JSON ブロックを埋め込みます (人間が読めるツールに依存しないまま):

```text
{
  "step": "<STEP_ID>",
  "agent": "<AGENT_NAME>",
  "spec": "<AGENT_SPEC_PATH>",
  "basePath": "<BASE_PATH>"
}
```

### オーケストレーターの構造（汎用性を保つ）

保守可能なオーケストレーターのために、これらの構造要素を文書化する:

- **動的パラメータ**: ユーザーから抽出される値 （例: `projectName`、`fileName`、`basePath`）。
- **サブエージェントレジストリ**: 各ステップを `agentName` + `agentSpecPath` にマッピングするリスト/テーブル。
- **ステップの順序**: 明示的なシーケンス（ステップ 1 → ステップ N）。
- **トリガー条件** （オプションですが推奨）: ステップが実行されるタイミングとスキップされるタイミングを定義します。
- **ログ戦略** （オプションですが推奨）: 各ステップの後に更新される単一のログ/レポート ファイル。

オーケストレータープロンプト内にオーケストレーション「コード」（JavaScript、Python など）を埋め込むことは避け、決定論的なツール駆動型の調整を優先します。

### 基本パターン

各ステップの呼び出しを構成する:

1. **ステップの説明**: 明確な一行の目的（ログとトレーサビリティに使用）
2. **エージェントID**: `agentName` + `agentSpecPath`
3. **コンテクスト**: 小さな明示的な変数セット（パス、ID、環境名）
4. **期待される出力**: 作成/更新するファイルとその書き込み先
5. **返却概要**: サブエージェントに短く構造化された要約を返すように依頼する

### 例: マルチステップ処理

```text
Step 1: 生の入力データを変換する
Agent: data-processor
Spec: .github/agents/data-processor.agent.md
Context: projectName=${projectName}, basePath=${basePath}
Input: ${basePath}/raw/
Output: ${basePath}/processed/
Expected: `${basePath}/processed/summary.md`に書き込む

Step 2: 処理されたデータを分析する（ステップ1の出力に依存）
Agent: data-analyst
Spec: .github/agents/data-analyst.agent.md
Context: projectName=${projectName}, basePath=${basePath}
Input: ${basePath}/processed/
Output: ${basePath}/analysis/
Expected: `${basePath}/analysis/report.md`に書き込む
```

### 要点

- **プロンプトで変数を渡す**: すべての動的な値には `${variableName}` を使用します
- **プロンプトを集中させる**: 各サブエージェントの明確で具体的なタスク
- **返却概要**: 各サブエージェントは達成したことを報告する必要がある
- **順次実行**: 出力と入力の間に依存関係がある場合にステップを順番に実行する
- **エラー処理**: 依存する手順に進む前に結果を確認する

### ⚠️ ツールの可用性要件

**クリティカル**: サブエージェントが特定のツール（例：`edit`、`execute`、`search`）を必要とする場合、オーケストレーターはそれらのツールを自身の`tools`リストに含める必要があります。サブエージェントは、親オーケストレーターが利用できないツールにアクセスできません。

**例**:

```yaml
# サブエージェントがファイルを編集したり、コマンドを実行したり、コードを検索したりする必要がある場合
tools: ["read", "edit", "search", "execute", "agent"]
```

オーケストレーターのツール権限は、呼び出されるすべてのサブエージェントの上限として機能します。すべてのサブエージェントに必要なツールが確実に提供されるよう、ツールリストを慎重に計画してください。

### ⚠️ 重要な制限事項

**サブエージェント オーケストレーションは、大規模なデータ処理には適していません。** マルチステップサブエージェントパイプラインの使用を避ける:

- 数百または数千のファイルの処理
- 大規模データセットの取り扱い
- 大規模なコードベースでの一括変換の実行
- 5～10以上の連続ステップのオーケストレーション

サブエージェントの呼び出しごとにレイテンシとコンテキストのオーバーヘッドが増加します。大量の処理が必要な場合は、単一のエージェントに直接ロジックを実装してください。オーケストレーションは、集中管理可能なデータセットにおける特殊なタスクの調整にのみ使用してください。

## Agent Prompt Structure

The markdown content below the frontmatter defines the agent's behavior, expertise, and instructions. Well-structured prompts typically include:

1. **Agent Identity and Role**: Who the agent is and its primary role
2. **Core Responsibilities**: What specific tasks the agent performs
3. **Approach and Methodology**: How the agent works to accomplish tasks
4. **Guidelines and Constraints**: What to do/avoid and quality standards
5. **Output Expectations**: Expected output format and quality

### Prompt Writing Best Practices

- **Be Specific and Direct**: Use imperative mood ("Analyze", "Generate"); avoid vague terms
- **Define Boundaries**: Clearly state scope limits and constraints
- **Include Context**: Explain domain expertise and reference relevant frameworks
- **Focus on Behavior**: Describe how the agent should think and work
- **Use Structured Format**: Headers, bullets, and lists make prompts scannable

## Variable Definition and Extraction

Agents can define dynamic parameters to extract values from user input and use them throughout the agent's behavior and sub-agent communications. This enables flexible, context-aware agents that adapt to user-provided data.

### When to Use Variables

**Use variables when**:

- Agent behavior depends on user input
- Need to pass dynamic values to sub-agents
- Want to make agents reusable across different contexts
- Require parameterized workflows
- Need to track or reference user-provided context

**Examples**:

- Extract project name from user prompt
- Capture certification name for pipeline processing
- Identify file paths or directories
- Extract configuration options
- Parse feature names or module identifiers

### Variable Declaration Pattern

Define variables section early in the agent prompt to document expected parameters:

```markdown
# Agent Name

## Dynamic Parameters

- **Parameter Name**: Description and usage
- **Another Parameter**: How it's extracted and used

## Your Mission

Process [PARAMETER_NAME] to accomplish [task].
```

### Variable Extraction Methods

#### 1. **Explicit User Input**

Ask the user to provide the variable if not detected in the prompt:

```markdown
## Your Mission

Process the project by analyzing your codebase.

### Step 1: Identify Project

If no project name is provided, **ASK THE USER** for:

- Project name or identifier
- Base path or directory location
- Configuration type (if applicable)

Use this information to contextualize all subsequent tasks.
```

#### 2. **Implicit Extraction from Prompt**

Automatically extract variables from the user's natural language input:

```javascript
// Example: Extract certification name from user input
const userInput = "Process My Certification";

// Extract key information
const certificationName = extractCertificationName(userInput);
// Result: "My Certification"

const basePath = `certifications/${certificationName}`;
// Result: "certifications/My Certification"
```

#### 3. **Contextual Variable Resolution**

Use file context or workspace information to derive variables:

```markdown
## Variable Resolution Strategy

1. **From User Prompt**: First, look for explicit mentions in user input
2. **From File Context**: Check current file name or path
3. **From Workspace**: Use workspace folder or active project
4. **From Settings**: Reference configuration files
5. **Ask User**: If all else fails, request missing information
```

### Using Variables in Agent Prompts

#### Variable Substitution in Instructions

Use template variables in agent prompts to make them dynamic:

```markdown
# Agent Name

## Dynamic Parameters

- **Project Name**: ${projectName}
- **Base Path**: ${basePath}
- **Output Directory**: ${outputDir}

## Your Mission

Process the **${projectName}** project located at `${basePath}`.

## Process Steps

1. Read input from: `${basePath}/input/`
2. Process files according to project configuration
3. Write results to: `${outputDir}/`
4. Generate summary report

## Quality Standards

- Maintain project-specific coding standards for **${projectName}**
- Follow directory structure: `${basePath}/[structure]`
```

#### Passing Variables to Sub-Agents

When invoking a sub-agent, pass all context through substituted variables in the prompt. Prefer passing **paths and identifiers**, not entire file contents.

Example (prompt template):

```text
This phase must be performed as the agent "documentation-writer" defined in ".github/agents/documentation-writer.agent.md".

IMPORTANT:
- Read and apply the entire .agent.md spec.
- Project: "${projectName}"
- Base path: "projects/${projectName}"
- Input: "projects/${projectName}/src/"
- Output: "projects/${projectName}/docs/"

Task:
1. Read source files under the input path.
2. Generate documentation.
3. Write outputs under the output path.
4. Return a concise summary (files created/updated, key decisions, issues).
```

The sub-agent receives all necessary context embedded in the prompt. Variables are resolved before sending the prompt, so the sub-agent works with concrete paths and values, not variable placeholders.

### Real-World Example: Code Review Orchestrator

Example of a simple orchestrator that validates code through multiple specialized agents:

1. Determine shared context:

- `repositoryName`, `prNumber`
- `basePath` (e.g., `projects/${repositoryName}/pr-${prNumber}`)

2. Invoke specialized agents sequentially (each agent reads its own `.agent.md` spec):

```text
Step 1: Security Review
Agent: security-reviewer
Spec: .github/agents/security-reviewer.agent.md
Context: repositoryName=${repositoryName}, prNumber=${prNumber}, basePath=projects/${repositoryName}/pr-${prNumber}
Output: projects/${repositoryName}/pr-${prNumber}/security-review.md

Step 2: Test Coverage
Agent: test-coverage
Spec: .github/agents/test-coverage.agent.md
Context: repositoryName=${repositoryName}, prNumber=${prNumber}, basePath=projects/${repositoryName}/pr-${prNumber}
Output: projects/${repositoryName}/pr-${prNumber}/coverage-report.md

Step 3: Aggregate
Agent: review-aggregator
Spec: .github/agents/review-aggregator.agent.md
Context: repositoryName=${repositoryName}, prNumber=${prNumber}, basePath=projects/${repositoryName}/pr-${prNumber}
Output: projects/${repositoryName}/pr-${prNumber}/final-review.md
```

#### Example: Conditional Step Orchestration (Code Review)

This example shows a more complete orchestration with **pre-flight checks**, **conditional steps**, and **required vs optional** behavior.

**Dynamic parameters (inputs):**

- `repositoryName`, `prNumber`
- `basePath` (e.g., `projects/${repositoryName}/pr-${prNumber}`)
- `logFile` (e.g., `${basePath}/.review-log.md`)

**Pre-flight checks (recommended):**

- Verify expected folders/files exist (e.g., `${basePath}/changes/`, `${basePath}/reports/`).
- Detect high-level characteristics that influence step triggers (e.g., repo language, presence of `package.json`, `pom.xml`, `requirements.txt`, test folders).
- Log the findings once at the start.

**Step trigger conditions:**

| Step                   | Status       | Trigger Condition                                                 | On Failure    |
| ---------------------- | ------------ | ----------------------------------------------------------------- | ------------- |
| 1: Security Review     | **Required** | Always run                                                        | Stop pipeline |
| 2: Dependency Audit    | Optional     | If a dependency manifest exists (`package.json`, `pom.xml`, etc.) | Continue      |
| 3: Test Coverage Check | Optional     | If test projects/files are present                                | Continue      |
| 4: Performance Checks  | Optional     | If perf-sensitive code changed OR a perf config exists            | Continue      |
| 5: Aggregate & Verdict | **Required** | Always run if Step 1 completed                                    | Stop pipeline |

**Execution flow (natural language):**

1. Initialize `basePath` and create/update `logFile`.
2. Run pre-flight checks and record them.
3. Execute Step 1 → N sequentially.
4. For each step:

- If trigger condition is false: mark as **SKIPPED** and continue.
- Otherwise: invoke the sub-agent using the wrapper prompt and capture its summary.
- Mark as **SUCCESS** or **FAILED**.
- If the step is **Required** and failed: stop the pipeline and write a failure summary.

5. End with a final summary section (overall status, artifacts, next actions).

**Sub-agent invocation prompt (example):**

```text
This phase must be performed as the agent "security-reviewer" defined in ".github/agents/security-reviewer.agent.md".

IMPORTANT:
- Read and apply the entire .agent.md spec.
- Work on repository "${repositoryName}" PR "${prNumber}".
- Base path: "${basePath}".

Task:
1. Review the changes under "${basePath}/changes/".
2. Write findings to "${basePath}/reports/security-review.md".
3. Return a short summary with: critical findings, recommended fixes, files created/modified.
```

**Logging format (example):**

```markdown
## Step 2: Dependency Audit

**Status:** ✅ SUCCESS / ⚠️ SKIPPED / ❌ FAILED
**Trigger:** package.json present
**Started:** 2026-01-16T10:30:15Z
**Completed:** 2026-01-16T10:31:05Z
**Duration:** 00:00:50
**Artifacts:** reports/dependency-audit.md
**Summary:** [brief agent summary]
```

This pattern applies to any orchestration scenario: extract variables, call sub-agents with clear context, await results.

### Variable Best Practices

#### 1. **Clear Documentation**

Always document what variables are expected:

```markdown
## Required Variables

- **projectName**: The name of the project (string, required)
- **basePath**: Root directory for project files (path, required)

## Optional Variables

- **mode**: Processing mode - quick/standard/detailed (enum, default: standard)
- **outputFormat**: Output format - markdown/json/html (enum, default: markdown)

## Derived Variables

- **outputDir**: Automatically set to ${basePath}/output
- **logFile**: Automatically set to ${basePath}/.log.md
```

#### 2. **Consistent Naming**

Use consistent variable naming conventions:

```javascript
// Good: Clear, descriptive naming
const variables = {
  projectName, // What project to work on
  basePath, // Where project files are located
  outputDirectory, // Where to save results
  processingMode, // How to process (detail level)
  configurationPath, // Where config files are
};

// Avoid: Ambiguous or inconsistent
const bad_variables = {
  name, // Too generic
  path, // Unclear which path
  mode, // Too short
  config, // Too vague
};
```

#### 3. **Validation and Constraints**

Document valid values and constraints:

```markdown
## Variable Constraints

**projectName**:

- Type: string (alphanumeric, hyphens, underscores allowed)
- Length: 1-100 characters
- Required: yes
- Pattern: `/^[a-zA-Z0-9_-]+$/`

**processingMode**:

- Type: enum
- Valid values: "quick" (< 5min), "standard" (5-15min), "detailed" (15+ min)
- Default: "standard"
- Required: no
```

## MCP Server Configuration (Organization/Enterprise Only)

MCP servers extend agent capabilities with additional tools. Only supported for organization and enterprise-level agents.

### Configuration Format

```yaml
---
name: my-custom-agent
description: "Agent with MCP integration"
tools: ["read", "edit", "custom-mcp/tool-1"]
mcp-servers:
  custom-mcp:
    type: "local"
    command: "some-command"
    args: ["--arg1", "--arg2"]
    tools: ["*"]
    env:
      ENV_VAR_NAME: ${{ secrets.API_KEY }}
---
```

### MCP Server Properties

- **type**: Server type (`'local'` or `'stdio'`)
- **command**: Command to start the MCP server
- **args**: Array of command arguments
- **tools**: Tools to enable from this server (`["*"]` for all)
- **env**: Environment variables (supports secrets)

### Environment Variables and Secrets

Secrets must be configured in repository settings under "copilot" environment.

**Supported syntax**:

```yaml
env:
  # Environment variable only
  VAR_NAME: COPILOT_MCP_ENV_VAR_VALUE

  # Variable with header
  VAR_NAME: $COPILOT_MCP_ENV_VAR_VALUE
  VAR_NAME: ${COPILOT_MCP_ENV_VAR_VALUE}

  # GitHub Actions-style (YAML only)
  VAR_NAME: ${{ secrets.COPILOT_MCP_ENV_VAR_VALUE }}
  VAR_NAME: ${{ var.COPILOT_MCP_ENV_VAR_VALUE }}
```

## File Organization and Naming

### Repository-Level Agents

- Location: `.github/agents/`
- Scope: Available only in the specific repository
- Access: Uses repository-configured MCP servers

### Organization/Enterprise-Level Agents

- Location: `.github-private/agents/` (then move to `agents/` root)
- Scope: Available across all repositories in org/enterprise
- Access: Can configure dedicated MCP servers

### Naming Conventions

- Use lowercase with hyphens: `test-specialist.agent.md`
- Name should reflect agent purpose
- Filename becomes default agent name (if `name` not specified)
- Allowed characters: `.`, `-`, `_`, `a-z`, `A-Z`, `0-9`

## Agent Processing and Behavior

### Versioning

- Based on Git commit SHAs for the agent file
- Create branches/tags for different agent versions
- Instantiated using latest version for repository/branch
- PR interactions use same agent version for consistency

### Name Conflicts

Priority (highest to lowest):

1. Repository-level agent
2. Organization-level agent
3. Enterprise-level agent

Lower-level configurations override higher-level ones with the same name.

### Tool Processing

- `tools` list filters available tools (built-in and MCP)
- No tools specified = all tools enabled
- Empty list (`[]`) = all tools disabled
- Specific list = only those tools enabled
- Unrecognized tool names are ignored (allows environment-specific tools)

### MCP Server Processing Order

1. Out-of-the-box MCP servers (e.g., GitHub MCP)
2. Custom agent MCP configuration (org/enterprise only)
3. Repository-level MCP configurations

Each level can override settings from previous levels.

## Agent Creation Checklist

### Frontmatter

- [ ] `description` field present and descriptive (50-150 chars)
- [ ] `description` wrapped in single quotes
- [ ] `name` specified (optional but recommended)
- [ ] `tools` configured appropriately (or intentionally omitted)
- [ ] `model` specified for optimal performance
- [ ] `target` set if environment-specific
- [ ] `infer` set to `false` if manual selection required

### Prompt Content

- [ ] Clear agent identity and role defined
- [ ] Core responsibilities listed explicitly
- [ ] Approach and methodology explained
- [ ] Guidelines and constraints specified
- [ ] Output expectations documented
- [ ] Examples provided where helpful
- [ ] Instructions are specific and actionable
- [ ] Scope and boundaries clearly defined
- [ ] Total content under 30,000 characters

### File Structure

- [ ] Filename follows lowercase-with-hyphens convention
- [ ] File placed in correct directory (`.github/agents/` or `agents/`)
- [ ] Filename uses only allowed characters
- [ ] File extension is `.agent.md`

### Quality Assurance

- [ ] Agent purpose is unique and not duplicative
- [ ] Tools are minimal and necessary
- [ ] Instructions are clear and unambiguous
- [ ] Agent has been tested with representative tasks
- [ ] Documentation references are current
- [ ] Security considerations addressed (if applicable)

## Common Agent Patterns

### Testing Specialist

**Purpose**: Focus on test coverage and quality
**Tools**: All tools (for comprehensive test creation)
**Approach**: Analyze, identify gaps, write tests, avoid production code changes

### Implementation Planner

**Purpose**: Create detailed technical plans and specifications
**Tools**: Limited to `['read', 'search', 'edit']`
**Approach**: Analyze requirements, create documentation, avoid implementation

### Code Reviewer

**Purpose**: Review code quality and provide feedback
**Tools**: `['read', 'search']` only
**Approach**: Analyze, suggest improvements, no direct modifications

### Refactoring Specialist

**Purpose**: Improve code structure and maintainability
**Tools**: `['read', 'search', 'edit']`
**Approach**: Analyze patterns, propose refactorings, implement safely

### Security Auditor

**Purpose**: Identify security issues and vulnerabilities
**Tools**: `['read', 'search', 'web']`
**Approach**: Scan code, check against OWASP, report findings

## Common Mistakes to Avoid

### Frontmatter Errors

- ❌ Missing `description` field
- ❌ Description not wrapped in quotes
- ❌ Invalid tool names without checking documentation
- ❌ Incorrect YAML syntax (indentation, quotes)

### Tool Configuration Issues

- ❌ Granting excessive tool access unnecessarily
- ❌ Missing required tools for agent's purpose
- ❌ Not using tool aliases consistently
- ❌ Forgetting MCP server namespace (`server-name/tool`)

### Prompt Content Problems

- ❌ Vague, ambiguous instructions
- ❌ Conflicting or contradictory guidelines
- ❌ Lack of clear scope definition
- ❌ Missing output expectations
- ❌ Overly verbose instructions (exceeding character limits)
- ❌ No examples or context for complex tasks

### Organizational Issues

- ❌ Filename doesn't reflect agent purpose
- ❌ Wrong directory (confusing repo vs org level)
- ❌ Using spaces or special characters in filename
- ❌ Duplicate agent names causing conflicts

## Testing and Validation

### Manual Testing

1. Create the agent file with proper frontmatter
2. Reload VS Code or refresh GitHub.com
3. Select the agent from the dropdown in Copilot Chat
4. Test with representative user queries
5. Verify tool access works as expected
6. Confirm output meets expectations

### Integration Testing

- Test agent with different file types in scope
- Verify MCP server connectivity (if configured)
- Check agent behavior with missing context
- Test error handling and edge cases
- Validate agent switching and handoffs

### Quality Checks

- Run through agent creation checklist
- Review against common mistakes list
- Compare with example agents in repository
- Get peer review for complex agents
- Document any special configuration needs

## Additional Resources

### Official Documentation

- [Creating Custom Agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
- [Custom Agents Configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Custom Agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [MCP Integration](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp)

### Community Resources

- [Awesome Copilot Agents Collection](https://github.com/github/awesome-copilot/tree/main/agents)
- [Customization Library Examples](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents)
- [Your First Custom Agent Tutorial](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents/your-first-custom-agent)

### Related Files

- [Prompt Files Guidelines](./prompt.instructions.md) - For creating prompt files
- [Instructions Guidelines](./instructions.instructions.md) - For creating instruction files

## Version Compatibility Notes

### GitHub.com (Coding Agent)

- ✅ Fully supports all standard frontmatter properties
- ✅ Repository and org/enterprise level agents
- ✅ MCP server configuration (org/enterprise)
- ❌ Does not support `model`, `argument-hint`, `handoffs` properties

### VS Code / JetBrains / Eclipse / Xcode

- ✅ Supports `model` property for AI model selection
- ✅ Supports `argument-hint` and `handoffs` properties
- ✅ User profile and workspace-level agents
- ❌ Cannot configure MCP servers at repository level
- ⚠️ Some properties may behave differently

When creating agents for multiple environments, focus on common properties and test in all target environments. Use `target` property to create environment-specific agents when necessary.
