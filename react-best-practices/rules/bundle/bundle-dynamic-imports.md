---
title: 重いコンポーネントには動的インポートを使う
impact: CRITICAL
impactDescription: TTI と LCP に直接影響する
tags: bundle, dynamic-import, code-splitting, next-dynamic
---

## 重いコンポーネントには動的インポートを使う

初回レンダリングで不要な大きなコンポーネントは、`next/dynamic` を使って遅延読み込みします。

**誤り（Monaco がメインチャンクに約300KB含まれる）:**

```tsx
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

**正しい例（Monaco を必要時に読み込む）:**

```tsx
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```
