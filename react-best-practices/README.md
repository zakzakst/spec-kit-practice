# React ベストプラクティス

エージェントと LLM 向けに最適化された、React ベストプラクティスを作成・管理するための構造化リポジトリです。

## 構成

- `rules/` - 個別ルールファイル（1 ルールにつき 1 ファイル）
  - `_sections.md` - セクションのメタデータ（タイトル、影響度、説明）
  - `_template.md` - 新しいルールを作成するためのテンプレート
  - `area-description.md` - 個別ルールファイル
- `src/` - ビルド用スクリプトとユーティリティ
- `metadata.json` - ドキュメントのメタデータ（バージョン、組織、概要）
- __`AGENTS.md`__ - コンパイル済み出力（生成物）
- __`test-cases.json`__ - LLM 評価用テストケース（生成物）

## はじめに

1. 依存関係をインストールします:
   ```bash
   pnpm install
   ```

2. ルールから `AGENTS.md` をビルドします:
   ```bash
   pnpm build
   ```

3. ルールファイルを検証します:
   ```bash
   pnpm validate
   ```

4. テストケースを抽出します:
   ```bash
   pnpm extract-tests
   ```

## 新しいルールを作成する

1. `rules/_template.md` を `rules/area-description.md` にコピーします
2. 適切なエリア接頭辞を選びます:
   - `async-` - Waterfalls の排除（セクション 1）
   - `bundle-` - バンドルサイズの最適化（セクション 2）
   - `server-` - サーバーサイド性能（セクション 3）
   - `client-` - クライアントサイドのデータ取得（セクション 4）
   - `rerender-` - 再レンダー最適化（セクション 5）
   - `rendering-` - レンダリング性能（セクション 6）
   - `js-` - JavaScript 性能（セクション 7）
   - `advanced-` - 高度なパターン（セクション 8）
3. frontmatter と本文を埋めます
4. 説明付きの明確な例があることを確認します
5. `pnpm build` を実行して `AGENTS.md` と `test-cases.json` を再生成します

## ルールファイルの構造

各ルールファイルは次の構造に従います:

```markdown
---
title: ルールのタイトル
impact: MEDIUM
impactDescription: 影響の説明（任意）
tags: tag1, tag2, tag3
---

## ルールのタイトル

ルールの簡潔な説明と、その重要性を記述します。

**Incorrect（何が問題かの説明）:**

```typescript
// 悪いコード例
```

**Correct（何が正しいかの説明）:**

```typescript
// 良いコード例
```

必要に応じて、例の後に補足説明を加えます。

Reference: [リンク](https://example.com)

## ファイル命名規則

- `_` で始まるファイルは特別扱いです（ビルド対象外）
- ルールファイル: `area-description.md`（例: `async-parallel.md`）
- セクションはファイル名の接頭辞から自動推定されます
- ルールは各セクション内でタイトル順にソートされます
- ID（例: 1.1, 1.2）はビルド時に自動生成されます

## 影響度レベル

- `CRITICAL` - 最優先、非常に大きな性能向上
- `HIGH` - 大きな性能改善
- `MEDIUM-HIGH` - 中程度から高めの改善
- `MEDIUM` - 中程度の性能改善
- `LOW-MEDIUM` - 小から中程度の改善
- `LOW` - 段階的な改善

## スクリプト

- `pnpm build` - ルールを AGENTS.md にコンパイルします
- `pnpm validate` - すべてのルールファイルを検証します
- `pnpm extract-tests` - LLM 評価用のテストケースを抽出します
- `pnpm dev` - ビルドと検証を実行します

## 貢献

ルールを追加または変更する際は:

1. セクションに対応した正しいファイル名の接頭辞を使います
2. `_template.md` の構造に従います
3. 説明付きの明確な良/悪の例を含めます
4. 適切なタグを追加します
5. `pnpm build` を実行して `AGENTS.md` と `test-cases.json` を再生成します
6. ルールはタイトル順に自動ソートされるため、番号管理は不要です

## 謝辞

[@shuding](https://x.com/shuding) により [Vercel](https://vercel.com) で最初に作成されました。
