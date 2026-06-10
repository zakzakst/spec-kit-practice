---
name: vercel-react-best-practices
description: Vercel Engineering による React / Next.js のパフォーマンス最適化ガイドライン。React / Next.js のコードを作成・レビュー・リファクタリングする際に、最適なパフォーマンスパターンを確実に適用したいときに使用します。React コンポーネント、Next.js ページ、データ取得、バンドル最適化、性能改善に関する作業でトリガーされます。
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# Vercel React ベストプラクティス

React と Next.js のアプリケーション向けに、Vercel が保守している包括的なパフォーマンス最適化ガイドです。69 個のルールを 8 カテゴリに整理し、影響度の高い順に並べることで、自動リファクタリングとコード生成を支援します。

## 適用する場面

次のようなケースでこのガイドラインを参照してください:
- 新しい React コンポーネントや Next.js ページを作成する
- データ取得を実装する（クライアント側・サーバー側の両方）
- コードをパフォーマンス観点でレビューする
- 既存の React / Next.js コードをリファクタリングする
- バンドルサイズや読み込み時間を最適化する

## 優先度別のルールカテゴリ

| 優先度 | カテゴリ | 影響度 | 接頭辞 |
|----------|----------|--------|--------|
| 1 | Waterfalls の排除 | CRITICAL | `async-` |
| 2 | バンドルサイズの最適化 | CRITICAL | `bundle-` |
| 3 | サーバーサイド性能 | HIGH | `server-` |
| 4 | クライアントサイドのデータ取得 | MEDIUM-HIGH | `client-` |
| 5 | 再レンダー最適化 | MEDIUM | `rerender-` |
| 6 | レンダリング性能 | MEDIUM | `rendering-` |
| 7 | JavaScript 性能 | LOW-MEDIUM | `js-` |
| 8 | 高度なパターン | LOW | `advanced-` |

## クイックリファレンス

### 1. Waterfalls の排除（CRITICAL）

- `async-cheap-condition-before-await` - フラグやリモート値を待つ前に、安価な同期条件を確認する
- `async-defer-await` - 実際に使う分岐まで `await` を遅らせる
- `async-parallel` - 独立した処理には `Promise.all()` を使う
- `async-dependencies` - 部分的な依存関係には `better-all` を使う
- `async-api-routes` - API ルートでは早めに Promise を開始し、await は遅らせる
- `async-suspense-boundaries` - Suspense を使ってコンテンツをストリーミングする

### 2. バンドルサイズの最適化（CRITICAL）

- `bundle-barrel-imports` - バレルファイルを避け、直接インポートする
- `bundle-dynamic-imports` - 重いコンポーネントには `next/dynamic` を使う
- `bundle-defer-third-party` - 分析・ログ系は hydration 後に読み込む
- `bundle-conditional` - 機能が有効になったときだけモジュールを読み込む
- `bundle-preload` - ホバーやフォーカスで事前読み込みして体感速度を上げる

### 3. サーバーサイド性能（HIGH）

- `server-auth-actions` - Server Action も API ルートと同様に認証する
- `server-cache-react` - リクエスト内の重複排除には `React.cache()` を使う
- `server-cache-lru` - リクエストをまたぐキャッシュには LRU キャッシュを使う
- `server-dedup-props` - RSC props で重複シリアライズを避ける
- `server-hoist-static-io` - フォントやロゴなどの静的 I/O はモジュールレベルへ移す
- `server-no-shared-module-state` - RSC / SSR で可変なリクエスト状態をモジュールに置かない
- `server-serialization` - クライアントコンポーネントへ渡すデータ量を最小化する
- `server-parallel-fetching` - コンポーネント構成を見直してフェッチを並列化する
- `server-parallel-nested-fetching` - item ごとのネストしたフェッチを `Promise.all` でつなぐ
- `server-after-nonblocking` - ブロックしない処理には `after()` を使う

### 4. クライアントサイドのデータ取得（MEDIUM-HIGH）

- `client-swr-dedup` - 自動重複排除には SWR を使う
- `client-event-listeners` - グローバルイベントリスナーは重複登録しない
- `client-passive-event-listeners` - スクロールには passive リスナーを使う
- `client-localstorage-schema` - localStorage データはバージョン管理し、最小化する

### 5. 再レンダー最適化（MEDIUM）

- `rerender-defer-reads` - コールバック内でしか使わない state には購読しない
- `rerender-memo` - 高コストな処理はメモ化コンポーネントへ切り出す
- `rerender-memo-with-default-value` - 非プリミティブなデフォルト props は外へ出す
- `rerender-dependencies` - effect ではプリミティブな依存値を使う
- `rerender-derived-state` - 生の値ではなく、派生した boolean などを購読する
- `rerender-derived-state-no-effect` - effect ではなく render 中に状態を導出する
- `rerender-functional-setstate` - 安定したコールバックには関数型 setState を使う
- `rerender-lazy-state-init` - 高コストな初期値には `useState` に関数を渡す
- `rerender-simple-expression-in-memo` - 単純なプリミティブ式に `useMemo` は使わない
- `rerender-split-combined-hooks` - 依存関係が独立した hooks は分割する
- `rerender-move-effect-to-event` - 交互ロジックはイベントハンドラへ移す
- `rerender-transitions` - 非緊急の更新には `startTransition` を使う
- `rerender-use-deferred-value` - 重い派生レンダーは `useDeferredValue` で遅延させる
- `rerender-use-ref-transient-values` - 一時的で頻繁に変わる値は refs に入れる
- `rerender-no-inline-components` - コンポーネントの内側で別コンポーネントを定義しない

### 6. レンダリング性能（MEDIUM）

- `rendering-animate-svg-wrapper` - SVG 要素そのものではなく wrapper をアニメートする
- `rendering-content-visibility` - 長いリストには `content-visibility` を使う
- `rendering-hoist-jsx` - 静的な JSX はコンポーネント外へ切り出す
- `rendering-svg-precision` - SVG の座標精度を下げる
- `rendering-hydration-no-flicker` - クライアント専用データにはインラインスクリプトを使う
- `rendering-hydration-suppress-warning` - 予期された不一致は suppress する
- `rendering-activity` - 表示/非表示には Activity コンポーネントを使う
- `rendering-conditional-render` - 条件分岐には `&&` より三項演算子を使う
- `rendering-usetransition-loading` - ローディング状態には `useTransition` を優先する
- `rendering-resource-hints` - 事前読み込みには React DOM の resource hints を使う
- `rendering-script-defer-async` - script タグには `defer` または `async` を使う

### 7. JavaScript 性能（LOW-MEDIUM）

- `js-batch-dom-css` - CSS の変更は class や `cssText` でまとめる
- `js-index-maps` - 繰り返しの検索には `Map` を作る
- `js-cache-property-access` - ループ内ではオブジェクトのプロパティをキャッシュする
- `js-cache-function-results` - 関数結果はモジュールレベルの `Map` にキャッシュする
- `js-cache-storage` - `localStorage` / `sessionStorage` の読み取りをキャッシュする
- `js-combine-iterations` - 複数の filter / map は 1 回のループにまとめる
- `js-length-check-first` - 配列比較では先に長さを確認する
- `js-early-exit` - 関数は早めに return する
- `js-hoist-regexp` - `RegExp` の生成はループ外へ移す
- `js-min-max-loop` - min / max は sort ではなくループで求める
- `js-set-map-lookups` - O(1) の検索には Set / Map を使う
- `js-tosorted-immutable` - 不変性が必要なら `sort()` ではなく `toSorted()` を使う
- `js-flatmap-filter` - map と filter を 1 パスでやるには `flatMap` を使う
- `js-request-idle-callback` - 重要でない作業は `requestIdleCallback` に回す

### 8. 高度なパターン（LOW）

- `advanced-effect-event-deps` - `useEffectEvent` の結果を effect の依存配列に入れない
- `advanced-event-handler-refs` - イベントハンドラは refs に保持する
- `advanced-init-once` - アプリの初期化はマウントごとではなく 1 回だけ行う
- `advanced-use-latest` - 安定したコールバック参照には `useLatest` を使う

## 使い方

各ルールファイルを読んで、詳細な説明とコード例を確認してください:

```
rules/async-parallel.md
rules/bundle-barrel-imports.md
```

各ルールファイルには次が含まれます:
- そのルールが重要な理由の簡潔な説明
- 問題点を示す incorrect のコード例
- 正しい実装を示す correct のコード例
- 追加の背景情報と参照リンク

## 完全版ドキュメント

全ルールを展開した完全なガイドは `AGENTS.md` を参照してください
