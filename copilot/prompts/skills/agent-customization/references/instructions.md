# [Instructions (.instructions.md)](https://code.visualstudio.com/docs/copilot/customization/custom-instructions#instructions-files)

特定のファイルセットまたはディレクトリに自動的に適用される的を絞ったルール。

## ロケーション

| パス | スコープ |
|------|-------|
| `.github/instructions/*.instructions.md` | ワークスペース |
| `<profile>/instructions/*.instructions.md` | ユーザープロファイル |
| `.agents/instructions/*.instructions.md` | 代替のワークスペースパス |

## Frontmatter

```yaml
---
description: "Reactコンポーネント用" # 任意: 人間が読むための説明
applyTo: # 必須: Globパターンの配列
  - "src/components/**/*.tsx"
  - "src/hooks/**/*.ts"
---
```

## テンプレート

```markdown
---
description: "Reactのベストプラクティス"
applyTo:
  - "src/components/**/*.tsx"
---

## 状態管理
- クラスコンポーネントではなく、Hooks（`useState`、`useEffect`）を使用します
- コンポーネントツリーの奥深くにある状態には `useContext` を使用します

## スタイリング
- インラインスタイルではなく、CSSモジュールを使用します
- 共有変数については `styles/theme.css` を参照してください
```

## 使用するタイミング

- 特定のテクノロジーに関するガイドライン（例: React、TypeScript、Go）
- テストフレームワークのルール（例: Cypress、Jest）
- 特定のディレクトリでのディレクトリ固有の規約（例: `api/` 対 `ui/`）
- API統合パターン

## 基本原則

1. **的を絞る**: `applyTo` を使用して関連するファイルに限定します。`**/*.ts`（すべて）のような指定は避け、`src/api/**/*.ts`（特定のもの）のようにします
2. **具体的**: 漠然とした原則ではなく、正確なパターンと例を提供します
3. **直交性**: それぞれの `.instructions.md` ファイルが重なり合うことなく、個別の懸念事項（テスト、スタイリング、APIなど）をカバーするようにします
4. **見せる、語らない**: 長い説明よりも簡潔なコード例を提供します

## アンチパターン

- **曖昧な説明**: 「役立つコーディングのヒント」では発見しにくくなります
- **広すぎる適用**: 適用先を `"**"` にしつつ、特定のファイルにしか関連しないコンテンツを含める
- **ドキュメントの複製**: リンクする代わりに README をコピーする
- **関心事の混在**: テスト + API設計 + スタイリングを1つのファイルにまとめる
