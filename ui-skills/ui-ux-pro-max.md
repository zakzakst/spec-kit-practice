---
name: ui-ux-pro-max
description: "UI/UXデザインインテリジェンス。50種類のスタイル、21種類のパレット、50種類のフォントペア、20種類のチャート、9種類のスタック（React、Next.js、Vue、Svelte、SwiftUI、React Native、Flutter、Tailwind、shadcn/ui）。アクション：計画、構築、作成、設計、実装、レビュー、修正、改善、最適化、強化、リファクタリング、UI/UXコードのチェック。プロジェクト：ウェブサイト、ランディングページ、ダッシュボード、管理パネル、eコマース、SaaS、ポートフォリオ、ブログ、モバイルアプリ、.html、.tsx、.vue、.svelte。要素：ボタン、モーダル、ナビゲーションバー、サイドバー、カード、表、フォーム、チャート。スタイル：グラスモーフィズム、クレイモーフィズム、ミニマリズム、ブルータリズム、ニューモーフィズム、ベントグリッド、ダークモード、レスポンシブ、スキューモーフィズム、フラットデザイン。トピック：カラーパレット、アクセシビリティ、アニメーション、レイアウト、タイポグラフィ、フォントの組み合わせ、間隔、ホバー、シャドウ、グラデーション。統合：コンポーネント検索とサンプル用のshadcn/ui MCP。"
---

# UI/UX Pro Max - デザインインテリジェンス

Webおよびモバイルアプリケーション向けの包括的なデザインガイド。9つのテクノロジースタックに対応した、50種類以上のスタイル、97種類のカラーパレット、57種類のフォントペア、99種類のUXガイドライン、25種類のチャートタイプを収録。優先度に基づいた推奨事項を含む検索可能なデータベースです。

## いつ利用するか

以下のガイドラインを参考にしてください:

- 新しいUIコンポーネントまたはページの設計
- カラーパレットとタイポグラフィの選択
- UXの問題に対するコードのレビュー
- ランディングページやダッシュボードの構築
- アクセシビリティ要件の実装

## 優先度別ルールカテゴリ

| 優先度 | カテゴリ                 | 影響度 | 領域                  |
| ------ | ------------------------ | ------ | --------------------- |
| 1      | アクセシビリティ         | 重要   | `ux`                  |
| 2      | タッチとインタラクション | 重要   | `ux`                  |
| 3      | パフォーマンス           | 高     | `ux`                  |
| 4      | レイアウトとレスポンシブ | 高     | `ux`                  |
| 5      | タイポグラフィと色       | 中     | `typography`, `color` |
| 6      | アニメーション           | 中     | `ux`                  |
| 7      | スタイル選択             | 中     | `style`, `product`    |
| 8      | チャートとデータ         | 低     | `chart`               |

## クイックリファレンス

### 1. アクセシビリティ（重要）

- `色のコントラスト` - 通常のテキストの最小比率は4.5:1
- `フォーカス状態` - インタラクティブな要素上の目に見えるフォーカスリング
- `代替テキスト` - 意味のある画像の説明的な代替テキスト
- `aria-labels` - アイコンのみのボタンのaria-label
- `キーボードナビゲーション` - タブ順序は視覚的な順序と一致する
- `フォームラベル` - for属性でラベルを使用する

### 2. タッチとインタラクション（重要）

- `タッチターゲットのサイズ` - 最小44x44ピクセルのタッチターゲット
- `ホバー vs タップ` - 主なインタラクションにはクリック/タップを使用する
- `読み込みボタン` - 非同期操作中にボタンを無効にする
- `エラーフィードバック` - 問題の近くのエラーメッセージをクリアする
- `カーソルポインタ` - クリック可能な要素にカーソルポインターを追加する

### 3. パフォーマンス（高）

- `画像最適化` - WebP、srcset、遅延読み込みを使用する
- `動きを制限した` - prefers-reduced-motion をチェック
- `コンテンツジャンプ` - 非同期コンテンツ用のスペースを予約する

### 4. レイアウトとレスポンシブ（高）

- `ビューポートメタ` - width=device-width initial-scale=1
- `読みやすいフォントサイズ` - モバイルでは本文テキストは最低16ピクセル
- `水平スクロール` - コンテンツがビューポートの幅に収まるようにする
- `Zインデックス管理` - Zインデックススケールを定義する（10、20、30、50）

### 5. タイポグラフィと色（中）

- `行の高さ` - 本文には1.5～1.75を使用します
- `行の長さ` - 1行あたり65～75文字までに制限
- `フォントペアリング` - 見出しと本文のフォントの個性を合わせる

### 6. アニメーション（中）

- `持続時間とタイミング` - マイクロインタラクションには150～300msを使用する
- `変換パフォーマンス` - width/heightではなく、transform/opacityを使用します
- `読み込み状態` - スケルトンスクリーンまたはスピナー

### 7. スタイル選択（中）

- `スタイルマッチ` - 製品タイプに合わせてスタイルを合わせる
- `一貫性` - すべてのページで同じスタイルを使用する
- `絵文字アイコンなし` - 絵文字ではなくSVGアイコンを使用する

### 8. チャートとデータ（低）

- `チャート型` - グラフの種類とデータの種類を一致させる
- `カラーガイダンス` - アクセシブルなカラーパレットを使用する
- `データテーブル` - アクセシビリティのためにテーブルの代替を提供する

## 使い方

以下の CLI ツールを使用して特定のドメインを検索します。

---

## 前提条件

Pythonがインストールされているかどうかを確認する:

```bash
python3 --version || python --version
```

Pythonがインストールされていない場合は、ユーザーのOSに基づいてインストールしてください:

**macOS:**

```bash
brew install python3
```

**Ubuntu/Debian:**

```bash
sudo apt update && sudo apt install python3
```

**Windows:**

```powershell
winget install Python.Python.3.12
```

---

## このスキルの使い方

ユーザーがUI/UX作業（設計、構築、作成、実装、レビュー、修正、改善）をリクエストした場合は、このワークフローに従ってください:

### Step 1: ユーザー要件を分析する

ユーザーリクエストから重要な情報を抽出する:

- **製品タイプ**: SaaS、eコマース、ポートフォリオ、ダッシュボード、ランディングページなど。
- **スタイルキーワード**: ミニマル、遊び心、プロフェッショナル、エレガント、ダークモードなど。
- **業界**: ヘルスケア、フィンテック、ゲーム、教育など。
- **スタック**: React、Vue、Next.js、またはデフォルトの `html-tailwind`

### Step 2: デザインシステムを生成する（必須）

**常に `--design-system` から始める** ことで、根拠のある包括的な推奨事項が得られます:

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<product_type> <industry> <keywords>" --design-system [-p "Project Name"]
```

このコマンド:

1. 5つのドメイン（製品、スタイル、色、ランディング、タイポグラフィ）を並行して検索します
2. `ui-reasoning.csv` からの推論ルールを適用して最適な一致を選択します
3. パターン、スタイル、色、タイポグラフィ、効果など、完全なデザインシステムを返します
4. 回避すべきアンチパターンを含む

**例:**

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "beauty spa wellness service" --design-system -p "Serenity Spa"
```

### Step 2b: 永続的なデザイン システム (マスター + オーバーライド パターン)

**セッション間で階層的に検索**するためにデザインシステムを保存するには、`--persist` を追加します。:

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<query>" --design-system --persist -p "Project Name"
```

これにより:

- `design-system/MASTER.md` — すべての設計ルールを備えたグローバルな正しいソース
- `design-system/pages/` — ページ固有のオーバーライド用のフォルダ

**ページ固有のオーバーライド:**

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<query>" --design-system --persist -p "Project Name" --page "dashboard"
```

これにより:

- `design-system/pages/dashboard.md` — マスターからのページ固有の逸脱

**階層的検索の仕組み:**

1. 特定のページ（例：「チェックアウト」）を構築するときは、まず `design-system/pages/checkout.md` を確認します。
2. ページファイルが存在する場合、そのルールがマスターファイルを**上書き**します。
3. そうでない場合は、`design-system/MASTER.md` のみを使用してください。

**コンテキスト認識検索プロンプト:**

```
[Page Name] ページを作成しています。design-system/MASTER.md をお読みください。
また、design-system/pages/[Page Name].md が存在するかどうかも確認してください。
ページファイルが存在する場合は、そのルールを優先します。
存在しない場合は、マスタールールのみを使用します。
では、コードを生成してください…
```

### Step 3: 詳細な検索を補足する（必要に応じて）

デザインシステムを入手したら、ドメイン検索を使用して詳細を取得します:

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

**詳細検索を使用する場合:**

| 必要                         | 領域         | 例                                      |
| ---------------------------- | ------------ | --------------------------------------- |
| より多くのスタイルオプション | `style`      | `--domain style "glassmorphism dark"`   |
| チャートの推奨事項           | `chart`      | `--domain chart "real-time dashboard"`  |
| UXのベストプラクティス       | `ux`         | `--domain ux "animation accessibility"` |
| 代替フォント                 | `typography` | `--domain typography "elegant luxury"`  |
| ランディング構造             | `landing`    | `--domain landing "hero social-proof"`  |

### Step 4: スタックガイドライン（デフォルト：html-tailwind）

実装固有のベストプラクティスを取得します。ユーザーがスタックを指定しない場合は、**デフォルトで `html-tailwind` が使用されます**。

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "<keyword>" --stack html-tailwind
```

利用可能なスタック: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`

---

## 検索リファレンス

### 利用可能な領域

| 領域         | 用途                                 | キーワード例                                             |
| ------------ | ------------------------------------ | -------------------------------------------------------- |
| `product`    | 製品タイプの推奨事項                 | SaaS, e-commerce, portfolio, healthcare, beauty, service |
| `style`      | UIスタイル、色、効果                 | glassmorphism, minimalism, dark mode, brutalism          |
| `typography` | フォントの組み合わせ、Google Fonts   | elegant, playful, professional, modern                   |
| `color`      | 製品タイプ別のカラーパレット         | saas, ecommerce, healthcare, beauty, fintech, service    |
| `landing`    | ページ構造、CTA戦略                  | hero, hero-centric, testimonial, pricing, social-proof   |
| `chart`      | チャートの種類、ライブラリの推奨事項 | trend, comparison, timeline, funnel, pie                 |
| `ux`         | ベストプラクティス、アンチパターン   | animation, accessibility, z-index, loading               |
| `react`      | React/Next.js のパフォーマンス       | waterfall, bundle, suspense, memo, rerender, cache       |
| `web`        | Webインターフェースガイドライン      | aria, focus, keyboard, semantic, virtualize              |
| `prompt`     | AIプロンプト、CSSキーワード          | (style name)                                             |

### 利用可能なスタック

| スタック          | 焦点                                                  |
| ----------------- | ----------------------------------------------------- |
| `html-tailwind`   | Tailwind utilities, responsive, a11y (DEFAULT)        |
| `react`           | State, hooks, performance, patterns                   |
| `nextjs`          | SSR, routing, images, API routes                      |
| `vue`             | Composition API, Pinia, Vue Router                    |
| `svelte`          | Runes, stores, SvelteKit                              |
| `swiftui`         | Views, State, Navigation, Animation                   |
| `react-native`    | Components, Navigation, Lists                         |
| `flutter`         | Widgets, State, Layout, Theming                       |
| `shadcn`          | shadcn/ui components, theming, forms, patterns        |
| `jetpack-compose` | Composables, Modifiers, State Hoisting, Recomposition |

---

## ワークフローの例

**ユーザーリクエスト:** "プロフェッショナルスキンケアサービスのランディングページを作成します。"

### Step 1: 要件を分析する

- 商品タイプ: 美容/スパサービス
- スタイルキーワード：エレガント、プロフェッショナル、ソフト
- 業界: 美容・ウェルネス
- スタック: html-tailwind (デフォルト)

### Step 2: デザインシステムを生成する（必須）

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "beauty spa wellness service elegant" --design-system -p "Serenity Spa"
```

**出力:** パターン、スタイル、色、タイポグラフィ、効果、アンチパターンを備えた完全なデザイン システム。

### Step 3: 詳細な検索を補足する（必要に応じて）

```bash
# アニメーションとアクセシビリティに関するUXガイドラインを入手する
python3 skills/ui-ux-pro-max/scripts/search.py "animation accessibility" --domain ux

# 必要に応じて代替のタイポグラフィオプションを取得します
python3 skills/ui-ux-pro-max/scripts/search.py "elegant luxury serif" --domain typography
```

### Step 4: スタックガイドライン

```bash
python3 skills/ui-ux-pro-max/scripts/search.py "layout responsive form" --stack html-tailwind
```

**それから:** デザインシステム＋詳細調査を統合し、デザインを実装します。

---

## 出力形式

`--design-system`フラグは2つの出力形式をサポートします:

```bash
# ASCII ボックス (デフォルト) - 端末の表示に最適
python3 skills/ui-ux-pro-max/scripts/search.py "fintech crypto" --design-system

# Markdown - ドキュメント作成に最適
python3 skills/ui-ux-pro-max/scripts/search.py "fintech crypto" --design-system -f markdown
```

---

## より良い結果を得るためのヒント

1. **キーワードを具体的にする** - 「ヘルスケアSaaSダッシュボード」>「アプリ」
2. **複数回検索** - キーワードが異なれば、洞察も異なる
3. **ドメインを結合する** - スタイル + タイポグラフィ + カラー = 完全なデザインシステム
4. **常にUXをチェックする** - よくある問題については、「アニメーション」、「Zインデックス」、「アクセシビリティ」を検索してください
5. **スタックフラグを使用する** - 実装固有のベストプラクティスを取得する
6. **反復する** - 最初の検索が一致しない場合は、別のキーワードを試してください

---

## プロフェッショナルUIの共通ルール

これらは、UIをプロフェッショナルに見えないようにする、見落とされがちな問題です:

### アイコンと視覚要素

| ルール                           | する                                                     | しない                                                           |
| -------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------- |
| **絵文字アイコンは利用しない**   | SVGアイコン（Heroicons、Lucide、Simple Icons）を使用する | 🎨 🚀 ⚙️ のような絵文字を UI アイコンとして使用する              |
| **安定したホバリング状態**       | ホバー時に色/不透明度の遷移を使用する                    | レイアウトをシフトするスケール変換を使用する                     |
| **正しいブランドロゴ**           | Simple Iconsの公式SVGを調べる                            | ロゴのパスを推測したり、間違ったパスを使用したりしないでください |
| **アイコンのサイズを一定にする** | w-6 h-6の固定viewBox（24x24）を使用する                  | 異なるアイコンサイズをランダムに組み合わせる                     |

### インタラクションとカーソル

| ルール                   | する                                                                    | しない                                           |
| ------------------------ | ----------------------------------------------------------------------- | ------------------------------------------------ |
| **カーソルポインタ**     | クリック可能/ホバー可能なすべてのカードに `cursor-pointer` を追加します | インタラクティブ要素にデフォルトのカーソルを残す |
| **ホバーフィードバック** | 視覚的なフィードバックを提供する（色、影、境界線）                      | 表示要素がインタラクティブではない               |
| **スムーズなtransition** | `transition-colors duration-200` を使用します                           | 状態が瞬時に変化するか、遅すぎる（>500ms）       |

### ライト/ダークモードのコントラスト

| ルール                           | する                                              | しない                                          |
| -------------------------------- | ------------------------------------------------- | ----------------------------------------------- |
| **ガラスカードライトモード**     | `bg-white/80` 以上の不透明度を使用してください    | `bg-white/10` を使用する（透明すぎる）          |
| **テキストコントラストライト**   | テキストには `#0F172A` (slate-900) を使用します   | 本文には `#94A3B8` (slate-400) を使用します     |
| **ミュートされたテキストライト** | 最低でも `#475569` (slate-600) を使用してください | グレー400またはそれ以下の明るさのものを使用する |
| **境界の可視性**                 | ライトモードでは `border-gray-200` を使用します   | `border-white/10` （非表示）を使用する          |

### レイアウトと間隔

| ルール                               | する                                             | しない                                                   |
| ------------------------------------ | ------------------------------------------------ | -------------------------------------------------------- |
| **フローティングナビゲーションバー** | `top-4 left-4 right-4`の間隔を追加               | ナビゲーションバーを `top-0 left-0 right-0` に固定します |
| **コンテンツのパディング**           | 固定されたナビゲーションバーの高さを考慮する     | コンテンツを固定要素の後ろに隠す                         |
| **一貫した最大幅**                   | 同じ `max-w-6xl` または `max-w-7xl` を使用します | 異なるコンテナ幅を混在させる                             |

---

## 納品前チェックリスト

UIコードを配信する前に、以下の項目を確認してください:

### 視覚品質

- [ ] アイコンとして絵文字は使用しないでください（代わりに SVG を使用してください）
- [ ] すべてのアイコンは一貫したアイコン セット（Heroicons/Lucide）から取得されます
- [ ] ブランドロゴは正しいです（Simple Icons から検証済み）
- [ ] ホバー状態ではレイアウトシフトが発生しない
- [ ] var() ラッパーではなく、テーマカラーを直接使用 (bg-primary)

### 交流

- [ ] クリック可能な要素はすべて `cursor-pointer` を持ちます
- [ ] ホバー状態は明確な視覚的フィードバックを提供する
- [ ] 遷移はスムーズです（150～300ms）
- [ ] キーボードナビゲーションでフォーカス状態を表示

### ライト/ダークモード

- [ ] ライトモードのテキストには十分なコントラストがあります（最低4.5:1）
- [ ] ライトモードで表示されるガラス/透明要素
- [ ] 両方のモードで境界線が表示されます
- [ ] 配送前に両方のモードをテストする

### レイアウト

- [ ] 浮動要素は端から適切な間隔をあける
- [ ] 固定ナビゲーションバーの背後に隠れたコンテンツはありません
- [ ] 375px、768px、1024px、1440pxでレスポンシブ
- [ ] モバイルでは横スクロールはできません

### アクセシビリティ

- [ ] すべての画像には代替テキストがあります
- [ ] フォーム入力にはラベルがある
- [ ] 色だけが指標ではない
- [ ] `prefers-reduced-motion` を尊重
