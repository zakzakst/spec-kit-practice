---
title: React DOM の Resource Hint を使う
impact: HIGH
impactDescription: 重要なリソースの読み込み時間を短縮する
tags: rendering, preload, preconnect, prefetch, resource-hints
---

## React DOM の Resource Hint を使う

**影響: 高（重要なリソースの読み込み時間を短縮する）**

React DOM には、ブラウザに必要なリソースを予告するための API があります。これは、クライアントが HTML を受け取る前にリソースの読み込みを始められるため、サーバーコンポーネントで特に有用です。

- **`prefetchDNS(href)`**: 今後接続する可能性があるドメインの DNS を解決する
- **`preconnect(href)`**: サーバーとの接続（DNS + TCP + TLS）を確立する
- **`preload(href, options)`**: 近いうちに使うリソース（stylesheet、font、script、image）を取得する
- **`preloadModule(href)`**: 近いうちに使う ES モジュールを取得する
- **`preinit(href, options)`**: stylesheet や script を取得して評価する
- **`preinitModule(href)`**: ES モジュールを取得して評価する

**例（外部 API への preconnect）:**

```tsx
import { preconnect, prefetchDNS } from 'react-dom'

export default function App() {
  prefetchDNS('https://analytics.example.com')
  preconnect('https://api.example.com')

  return <main>{/* content */}</main>
}
```

**例（重要なフォントとスタイルを preload する）:**

```tsx
import { preload, preinit } from 'react-dom'

export default function RootLayout({ children }) {
  // フォントファイルを preload する
  preload('/fonts/inter.woff2', { as: 'font', type: 'font/woff2', crossOrigin: 'anonymous' })

  // 重要な stylesheet を即座に取得して適用する
  preinit('/styles/critical.css', { as: 'style' })

  return (
    <html>
      <body>{children}</body>
    </html>
  )
}
```

**例（コード分割されたルート用のモジュールを preload する）:**

```tsx
import { preloadModule, preinitModule } from 'react-dom'

function Navigation() {
  const preloadDashboard = () => {
    preloadModule('/dashboard.js', { as: 'script' })
  }

  return (
    <nav>
      <a href="/dashboard" onMouseEnter={preloadDashboard}>
        Dashboard
      </a>
    </nav>
  )
}
```

**使い分け:**

| API | 用途 |
|-----|------|
| `prefetchDNS` | 後で接続する外部ドメイン |
| `preconnect` | すぐに取得する API や CDN |
| `preload` | 現在のページに必要な重要リソース |
| `preloadModule` | 次の遷移で使われそうな JS モジュール |
| `preinit` | 早期に実行が必要な stylesheet / script |
| `preinitModule` | 早期に実行が必要な ES モジュール |

Reference: [React DOM Resource Preloading APIs](https://react.dev/reference/react-dom#resource-preloading-apis)
