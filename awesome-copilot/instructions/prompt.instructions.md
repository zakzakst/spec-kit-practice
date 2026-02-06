---
description: "GitHub Copilot 用の高品質なプロンプトファイルを作成するためのガイドライン"
applyTo: "**/*.prompt.md"
---

# Copilot プロンプトファイルのガイドライン

あらゆるリポジトリで一貫した高品質の結果を提供できるように GitHub Copilot をガイドする、効果的で保守可能なプロンプト ファイルを作成する手順。

## 範囲と原則

- 対象者: Copilot Chat の再利用可能なプロンプトを作成するメンテナーと貢献者。
- 目標: 予測可能な動作、明確な期待、最小限の権限、リポジトリ間の移植性。
- 主な参考資料: プロンプト ファイルと組織固有の規則に関する VS Code ドキュメント。

## 前書きの要件

すべてのプロンプト ファイルには、次のフィールドを含む YAML フロントマターが含まれている必要があります。

### Required/Recommended Fields

| フィールド      | 必須 | 説明                                                                                                       |
| --------------- | ---- | ---------------------------------------------------------------------------------------------------------- |
| `description`   | 推奨 | プロンプトの簡単な説明（1文、実行可能な結果）                                                              |
| `name`          | 任意 | チャットで「/」を入力した後に表示される名前。指定されていない場合はファイル名がデフォルトになります。      |
| `agent`         | 推奨 | 使用するエージェント: `ask`、`edit`、`agent`、またはカスタムエージェント名。デフォルトは現在のエージェント |
| `model`         | 任意 | 使用する言語モデル。デフォルトは現在選択されているモデルです。                                             |
| `tools`         | 任意 | このプロンプトで使用できるツール/ツールセット名のリスト                                                    |
| `argument-hint` | 任意 | チャット入力時にユーザーインタラクションをガイドするためのヒントテキストが表示されます                     |

### ガイドライン

- 読みやすさとバージョン管理の明確さを保つために、一貫した引用符（一重引用符を推奨）を使用し、1 行につき 1 つのフィールドを維持します。
- `tools` が指定されていて、現在のエージェントが `ask` または `edit` の場合、デフォルトのエージェントは `agent` になります。
- 組織で必要な追加のメタデータ（`language`, `tags`, `visibility`など）を保存します

## ファイルの命名と配置

- ワークスペース標準で別のディレクトリが指定されていない限り、`.prompt.md` で終わるケバブケースのファイル名を使用し、`.github/prompts/` の下に保存します。
- アクションを伝える短いファイル名を指定します (たとえば、`prompt1.prompt.md` ではなく `generate-readme.prompt.md`)。

## 内容の構造

- プロンプトの意図に一致する `#` レベルの見出しから始めると、クイックピック検索で適切に表示されます。
- コンテンツを予測可能なセクションで構成します。推奨されるベースラインは、「ミッション」または「主要指令」、「スコープと前提条件」、「入力」、「ワークフロー」（ステップバイステップ）、「出力の期待値」、「品質保証」です。
- セクション名はドメインに合わせて調整しますが、論理的な流れを維持します: 理由 → コンテキスト → 入力 → アクション → 出力 → 検証。
- 発見しやすくするために、相対リンクを使用して関連するプロンプトまたは説明ファイルを参照します。

## 入力とコンテキスト処理

- 必須値には `${input:variableName[:placeholder]}` を使用し、ユーザーが入力しなければならない場合について説明します。可能な場合は、デフォルト値または代替値を提供してください。
- `${selection}`、`${file}`、`${workspaceFolder}` などのコンテキスト変数は、必要な場合にのみ呼び出し、Copilot がそれらをどのように解釈するかを説明します。
- 必須のコンテキストが欠落している場合の続行方法を文書化します (たとえば、「ファイル パスを要求し、未定義のままの場合は停止する」)。

## ツールと権限のガイダンス

- `tools` は、タスクを実行できる最小限のツールセットに限定してください。順序が重要な場合は、優先実行順にリストしてください。
- プロンプトがチャット モードからツールを継承する場合は、その関係について言及し、重要なツールの動作や副作用を述べます。
- 破壊的な操作 (ファイルの作成、編集、ターミナル コマンド) について警告し、ワークフローにガードレールまたは確認手順を含めます。

## 指導のトーンとスタイル

- Copilot を対象とした直接的で命令形の文章で記述します (例:「分析する」、「生成する」、「要約する」)。
- ローカリゼーションをサポートするには、Google デベロッパー ドキュメントの翻訳のベスト プラクティスに従って、文章を短く明確にしてください。
- 慣用句、ユーモア、または文化特有の言及は避け、中立的で包括的な言語を優先します。

## Output Definition

- Specify the format, structure, and location of expected results (for example, “Create `docs/adr/adr-XXXX.md` using the template below”).
- Include success criteria and failure triggers so Copilot knows when to halt or retry.
- Provide validation steps—manual checks, automated commands, or acceptance criteria lists—that reviewers can execute after running the prompt.

## Examples and Reusable Assets

- Embed Good/Bad examples or scaffolds (Markdown templates, JSON stubs) that the prompt should produce or follow.
- Maintain reference tables (capabilities, status codes, role descriptions) inline to keep the prompt self-contained. Update these tables when upstream resources change.
- Link to authoritative documentation instead of duplicating lengthy guidance.

## Quality Assurance Checklist

- [ ] Frontmatter fields are complete, accurate, and least-privilege.
- [ ] Inputs include placeholders, default behaviours, and fallbacks.
- [ ] Workflow covers preparation, execution, and post-processing without gaps.
- [ ] Output expectations include formatting and storage details.
- [ ] Validation steps are actionable (commands, diff checks, review prompts).
- [ ] Security, compliance, and privacy policies referenced by the prompt are current.
- [ ] Prompt executes successfully in VS Code (`Chat: Run Prompt`) using representative scenarios.

## Maintenance Guidance

- Version-control prompts alongside the code they affect; update them when dependencies, tooling, or review processes change.
- Review prompts periodically to ensure tool lists, model requirements, and linked documents remain valid.
- Coordinate with other repositories: when a prompt proves broadly useful, extract common guidance into instruction files or shared prompt packs.

## Additional Resources

- [Prompt Files Documentation](https://code.visualstudio.com/docs/copilot/customization/prompt-files#_prompt-file-format)
- [Awesome Copilot Prompt Files](https://github.com/github/awesome-copilot/tree/main/prompts)
- [Tool Configuration](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode#_agent-mode-tools)
