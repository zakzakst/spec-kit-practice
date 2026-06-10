---
title: Script タグには defer または async を使う
impact: HIGH
impactDescription: レンダーブロックをなくす
tags: rendering, script, defer, async, performance
---

## Script タグには defer または async を使う

**影響: 高（レンダーブロックをなくす）**

`defer` や `async` のない script タグは、スクリプトのダウンロードと実行中に HTML の解析をブロックします。これにより、First Contentful Paint と Time to Interactive が遅れます。

- **`defer`**: 並列でダウンロードし、HTML の解析完了後に実行し、実行順序を保つ
- **`async`**: 並列でダウンロードし、準備ができ次第すぐ実行し、順序は保証しない

DOM や他のスクリプトに依存するものには `defer` を使います。analytics のような独立したスクリプトには `async` を使います。

**不適切（描画をブロックする）:**

```tsx
export default function Document() {
  return (
    <html>
      <head>
        <script src="https://example.com/analytics.js" />
        <script src="/scripts/utils.js" />
      </head>
      <body>{/* content */}</body>
    </html>
  )
}
```

**適切（ブロックしない）:**

```tsx
export default function Document() {
  return (
    <html>
      <head>
        {/* 独立したスクリプト - async を使う */}
        <script src="https://example.com/analytics.js" async />
        {/* DOM 依存のスクリプト - defer を使う */}
        <script src="/scripts/utils.js" defer />
      </head>
      <body>{/* content */}</body>
    </html>
  )
}
```

**注:** Next.js では、素の script タグよりも `strategy` プロパティ付きの `next/script` コンポーネントを優先してください。

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script src="https://example.com/analytics.js" strategy="afterInteractive" />
      <Script src="/scripts/utils.js" strategy="beforeInteractive" />
    </>
  )
}
```

Reference: [MDN - Script element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#defer)
