---
title: 非緊急な更新には Transition を使う
impact: MEDIUM
impactDescription: UI の応答性を保つ
tags: rerender, transitions, startTransition, performance
---

## 非緊急な更新には Transition を使う

頻繁に発生する非緊急な state 更新は transition として扱い、UI の応答性を保ってください。

**誤り（スクロールのたびに UI をブロックする）:**

```tsx
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => setScrollY(window.scrollY)
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}
```

**正しい例（ブロックしない更新）:**

```tsx
import { startTransition } from 'react'

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => {
      startTransition(() => setScrollY(window.scrollY))
    }
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}
```
