---
description: "GitHub Copilot 用の高品質なカスタム指示ファイルを作成するためのガイドライン"
applyTo: "**/*.instructions.md"
---

# カスタム指示ファイルのガイドライン

GitHub Copilot がドメイン固有のコードを生成し、プロジェクトの規則に従うようにガイドする、効果的で保守可能なカスタム指示ファイルを作成する手順。

## プロジェクトの背景

- 対象読者: ドメイン固有のコードを扱う開発者とGitHub Copilot
- ファイル形式: YAML フロントマターを使用した Markdown
- ファイルの命名規則: ハイフン付きの小文字（例：`react-best-practices.instructions.md`）
- 場所: `.github/instructions/` 配下
- 目的: コード生成、レビュー、ドキュメント化のためのコンテキスト認識ガイダンスを提供する

## 必須の前文

すべての指示ファイルには、次のフィールドを含むYAMLフロントマターを含める必要があります。:

```yaml
---
description: "指導の目的と範囲の簡単な説明"
applyTo: "対象ファイルの glob パターン (例: **/*.ts、**/*.py)"
---
```

### 前書きガイドライン

- **description**: 目的を明確に示す、1～500文字の単一引用符で囲まれた文字列
- **applyTo**: これらの命令が適用されるファイルを指定する glob パターン
  - 単一パターン: `'**/*.ts'`
  - 複数パターン: `'**/*.ts, **/*.tsx, **/*.js'`
  - 特定のファイル: `'src/**/*.py'`
  - すべてのファイル: `'**'`

## ファイル構造

適切に構成された指示ファイルには、以下のセクションが含まれている必要があります:

### 1. タイトルと概要

- `#` 見出しを使用した明確で説明的なタイトル
- 目的と範囲を説明する簡単な紹介
- オプション: 主要なテクノロジーとバージョンを含むプロジェクトコンテキストセクション

### 2. コアセクション

ドメインに基づいてコンテンツを論理的なセクションに整理する:

- **一般的な指示**: 高レベルのガイドラインと原則
- **ベストプラクティス**: 推奨されるパターンとアプローチ
- **コード標準**: 命名規則、フォーマット、スタイルルール
- **建築/構造**: プロジェクトの構成と設計パターン
- **一般的なパターン**: よく使用される実装
- **セキュリティ**: セキュリティに関する考慮事項（該当する場合）
- **パフォーマンス**: 最適化ガイドライン（該当する場合）
- **テスト**: テスト基準とアプローチ（該当する場合）

### 3. 例とコードスニペット

明確なラベルを付けて具体的な例を挙げる:

```markdown
### 良い例

\`\`\`language
// 推奨されるアプローチ
コード例はこちら
\`\`\`

### 悪い例

\`\`\`language
// このパターンを避ける
コード例はこちら
\`\`\`
```

### 4. 検証と確認（オプションだが推奨）

- Build commands to verify code
- Linting and formatting tools
- Testing requirements
- Verification steps

## Content Guidelines

### Writing Style

- Use clear, concise language
- Write in imperative mood ("Use", "Implement", "Avoid")
- Be specific and actionable
- Avoid ambiguous terms like "should", "might", "possibly"
- Use bullet points and lists for readability
- Keep sections focused and scannable

### Best Practices

- **Be Specific**: Provide concrete examples rather than abstract concepts
- **Show Why**: Explain the reasoning behind recommendations when it adds value
- **Use Tables**: For comparing options, listing rules, or showing patterns
- **Include Examples**: Real code snippets are more effective than descriptions
- **Stay Current**: Reference current versions and best practices
- **Link Resources**: Include official documentation and authoritative sources

### Common Patterns to Include

1. **Naming Conventions**: How to name variables, functions, classes, files
2. **Code Organization**: File structure, module organization, import order
3. **Error Handling**: Preferred error handling patterns
4. **Dependencies**: How to manage and document dependencies
5. **Comments and Documentation**: When and how to document code
6. **Version Information**: Target language/framework versions

## Patterns to Follow

### Bullet Points and Lists

```markdown
## Security Best Practices

- Always validate user input before processing
- Use parameterized queries to prevent SQL injection
- Store secrets in environment variables, never in code
- Implement proper authentication and authorization
- Enable HTTPS for all production endpoints
```

### Tables for Structured Information

```markdown
## Common Issues

| Issue            | Solution            | Example                       |
| ---------------- | ------------------- | ----------------------------- |
| Magic numbers    | Use named constants | `const MAX_RETRIES = 3`       |
| Deep nesting     | Extract functions   | Refactor nested if statements |
| Hardcoded values | Use configuration   | Store API URLs in config      |
```

### Code Comparison

```markdown
### Good Example - Using TypeScript interfaces

\`\`\`typescript
interface User {
id: string;
name: string;
email: string;
}

function getUser(id: string): User {
// Implementation
}
\`\`\`

### Bad Example - Using any type

\`\`\`typescript
function getUser(id: any): any {
// Loses type safety
}
\`\`\`
```

### Conditional Guidance

```markdown
## Framework Selection

- **For small projects**: Use Minimal API approach
- **For large projects**: Use controller-based architecture with clear separation
- **For microservices**: Consider domain-driven design patterns
```

## Patterns to Avoid

- **Overly verbose explanations**: Keep it concise and scannable
- **Outdated information**: Always reference current versions and practices
- **Ambiguous guidelines**: Be specific about what to do or avoid
- **Missing examples**: Abstract rules without concrete code examples
- **Contradictory advice**: Ensure consistency throughout the file
- **Copy-paste from documentation**: Add value by distilling and contextualizing

## Testing Your Instructions

Before finalizing instruction files:

1. **Test with Copilot**: Try the instructions with actual prompts in VS Code
2. **Verify Examples**: Ensure code examples are correct and run without errors
3. **Check Glob Patterns**: Confirm `applyTo` patterns match intended files

## Example Structure

Here's a minimal example structure for a new instruction file:

```markdown
---
description: "Brief description of purpose"
applyTo: "**/*.ext"
---

# Technology Name Development

Brief introduction and context.

## General Instructions

- High-level guideline 1
- High-level guideline 2

## Best Practices

- Specific practice 1
- Specific practice 2

## Code Standards

### Naming Conventions

- Rule 1
- Rule 2

### File Organization

- Structure 1
- Structure 2

## Common Patterns

### Pattern 1

Description and example

\`\`\`language
code example
\`\`\`

### Pattern 2

Description and example

## Validation

- Build command: `command to verify`
- Linting: `command to lint`
- Testing: `command to test`
```

## Maintenance

- Review instructions when dependencies or frameworks are updated
- Update examples to reflect current best practices
- Remove outdated patterns or deprecated features
- Add new patterns as they emerge in the community
- Keep glob patterns accurate as project structure evolves

## Additional Resources

- [Custom Instructions Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Awesome Copilot Instructions](https://github.com/github/awesome-copilot/tree/main/instructions)
