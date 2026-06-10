---
title: 状態の読み取りは使用時まで遅らせる
impact: MEDIUM
impactDescription: 不要な購読を避ける
tags: rerender, searchParams, localStorage, optimization
---

## 状態の読み取りは使用時まで遅らせる

コールバックの中でしか読まないなら、`searchParams` や `localStorage` のような動的な状態を購読しないでください。

**誤り（searchParams の変化すべてを購読する）:**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams()

  const handleShare = () => {
    const ref = searchParams.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```

**正しい例（必要なときだけ読み取り、購読しない）:**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search)
    const ref = params.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```
