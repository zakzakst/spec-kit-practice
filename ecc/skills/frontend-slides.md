---
name: frontend-slides
description: ゼロから、または PowerPoint 変換から、見栄えのするアニメーション豊かな HTML プレゼンテーションを作成します。発表資料、ピッチ、PPT/PPTX の web 化などで使います。抽象的なデザイン選択ではなく、視覚的な試作を通して非デザイナーが好みを見つけられるよう支援します。
origin: ECC
---

# フロントエンドスライド

ブラウザだけで動く、依存ゼロのアニメーション豊かな HTML プレゼンテーションを作成します。

zarazhangrui の仕事に見られる visual exploration のアプローチに着想を得ています（credit: @zarazhangrui）。

## 有効化するタイミング

- トーク deck、pitch deck、workshop deck、社内プレゼンを作るとき
- `.ppt` または `.pptx` を HTML プレゼンへ変換するとき
- 既存 HTML プレゼンのレイアウト、モーション、タイポグラフィを改善するとき
- まだ自分のデザイン嗜好が定まっていないユーザーと一緒にスタイル探索するとき

## 絶対条件

1. **依存ゼロ**: 原則として、インライン CSS / JS を含む自己完結型 HTML 1 ファイルにする。
2. **ビューポートに収めることは必須**: すべてのスライドは 1 画面に収まり、内部スクロールを持たない。
3. **見せて判断してもらう**: 抽象的な好みアンケートではなく視覚プレビューを使う。
4. **個性のあるデザイン**: 汎用的な紫グラデ、白背景に Inter、テンプレ感のある deck を避ける。
5. **実運用品質**: コメント、アクセシビリティ、レスポンシブ性、性能を保つ。

生成前に `STYLE_PRESETS.md` を読み、viewport-safe な CSS ベース、密度制限、プリセット一覧、CSS の注意点を確認する。

## ワークフロー

### 1. モード判定

次のどれかを選ぶ:

- **New presentation**: ユーザーがトピック、メモ、または原稿を持っている
- **PPT conversion**: ユーザーが `.ppt` または `.pptx` を持っている
- **Enhancement**: すでに HTML スライドがあり改善したい

### 2. コンテンツ把握

必要最小限だけ確認する:

- 目的: pitch、教育、カンファレンストーク、社内共有
- 長さ: short（5〜10）、medium（10〜20）、long（20+）
- コンテンツの状態: 完成済みコピー、粗いメモ、トピックのみ

コンテンツがあるなら、スタイル検討前に貼ってもらう。

### 3. スタイル把握

既定は visual exploration。

すでに希望 preset が決まっているならプレビューを飛ばして直接使う。

そうでなければ:

1. deck が与えるべき感情を聞く: impressed, energized, focused, inspired
2. `.ecc-design/slide-previews/` に **1 枚スライドのプレビュー 3 種** を生成する
3. 各プレビューは自己完結型で、タイポグラフィ / 色 / モーションが分かりやすく、内容は概ね 100 行未満に保つ
4. どれを採用するか、またはどの要素を混ぜるかをユーザーに選んでもらう

気分からスタイルへ対応づけるときは `STYLE_PRESETS.md` の preset guide を使う。

### 4. プレゼンを構築する

出力先は次のいずれか:

- `presentation.html`
- `[presentation-name].html`

抽出画像やユーザー提供画像がある場合だけ `assets/` フォルダを使う。

必須構造:

- セマンティックな slide section
- `STYLE_PRESETS.md` にある viewport-safe CSS base
- テーマ値用の CSS custom properties
- キーボード、ホイール、タッチ操作に対応する presentation controller class
- reveal animation 用の Intersection Observer
- reduced-motion 対応

### 5. ビューポート適合を強制する

これは hard gate として扱う。

ルール:

- すべての `.slide` は `height: 100vh; height: 100dvh; overflow: hidden;` を使う
- 文字サイズと余白は `clamp()` でスケールさせる
- 内容が収まらない場合は複数スライドに分割する
- 可読性を壊すほど文字を小さくして overflow を解決しない
- スライド内部にスクロールバーを出さない

密度制限と必須 CSS ブロックは `STYLE_PRESETS.md` を使う。

### 6. 検証

完成 deck を次のサイズで確認する:

- 1920x1080
- 1280x720
- 768x1024
- 375x667
- 667x375

ブラウザ自動化が使えるなら、各スライドで overflow がないことと、キーボード操作が機能することを確認する。

### 7. 納品

引き渡し時:

- ユーザーが残したいと言わない限り、一時プレビューは削除する
- 有用なら OS に応じた opener で deck を開く
- ファイルパス、使用 preset、スライド枚数、簡単なテーマ変更ポイントを要約する

現在の OS に応じた opener を使う:

- macOS: `open file.html`
- Linux: `xdg-open file.html`
- Windows: `start "" file.html`

## PPT / PPTX 変換

PowerPoint 変換時:

1. まず `python3` と `python-pptx` でテキスト、画像、ノートを抽出する
2. `python-pptx` がない場合、インストールするか手動 / export ベース手順へ切り替えるか確認する
3. スライド順、speaker notes、抽出アセットを維持する
4. 抽出後は新規プレゼンと同じ style-selection workflow を適用する

変換はクロスプラットフォームで保つ。Python でできるなら macOS 専用ツールに依存しない。

## 実装要件

### HTML / CSS

- 明示要求がない限り CSS / JS はインラインにする
- フォントは Google Fonts または Fontshare を使ってよい
- 雰囲気のある背景、強いタイプ階層、はっきりした視覚方向を優先する
- イラストより、抽象形状、グラデーション、グリッド、ノイズ、幾何で組む

### JavaScript

含めるもの:

- キーボードナビゲーション
- タッチ / スワイプナビゲーション
- マウスホイールナビゲーション
- 進捗表示またはスライド番号
- 進入時に発火する reveal animation

### アクセシビリティ

- セマンティック構造を使う（`main`, `section`, `nav`）
- コントラストを読みやすく保つ
- キーボードだけで操作できるようにする
- `prefers-reduced-motion` を尊重する

## コンテンツ密度の上限

可読性が保てるならという条件つきで、ユーザーがもっと密にしてほしいと明示しない限り、以下の上限を使う:

| Slide type   | Limit                                         |
| ------------ | --------------------------------------------- |
| Title        | 1 heading + 1 subtitle + optional tagline     |
| Content      | 1 heading + 4-6 bullets or 2 short paragraphs |
| Feature grid | 6 cards max                                   |
| Code         | 8-10 lines max                                |
| Quote        | 1 quote + attribution                         |
| Image        | 1 image constrained by viewport               |

## アンチパターン

- 視覚的アイデンティティのない generic startup gradient
- 意図がない system font deck
- 長い bullet wall
- スクロールが必要な code block
- 低い画面で壊れる固定高コンテンツボックス
- `-clamp(...)` のような無効な negated CSS function

## 関連 ECC スキル

- `frontend-patterns` - deck 周辺のコンポーネントやインタラクションパターン
- `liquid-glass-design` - Apple 的 glass aesthetics を意図的に取り込む場合
- `e2e-testing` - 最終 deck の自動ブラウザ検証が必要な場合

## 納品チェックリスト

- presentation がブラウザでローカルファイルから動く
- すべての slide がスクロールなしでビューポートに収まる
- スタイルに個性と意図がある
- アニメーションがうるさくなく意味を持っている
- reduced motion が尊重されている
- 引き渡し時にファイルパスとカスタマイズポイントが説明されている
