---
title: 重要でないサードパーティライブラリの読み込みを遅延する
impact: MEDIUM
impactDescription: ハイドレーション後に読み込む
tags: bundle, third-party, analytics, defer
---

## 重要でないサードパーティライブラリの読み込みを遅延する

分析、ログ、エラートラッキングはユーザー操作をブロックしません。ハイドレーション後に読み込みます。

**誤り（初期バンドルをブロックする）:**

```tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

**正しい例（ハイドレーション後に読み込む）:**

```tsx
import dynamic from 'next/dynamic'

const Analytics = dynamic(
  () => import('@vercel/analytics/react').then(m => m.Analytics),
  { ssr: false }
)

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```
