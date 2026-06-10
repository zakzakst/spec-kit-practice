---
title: 一時的な値には useRef を使う
impact: MEDIUM
impactDescription: 頻繁な更新で不要な再レンダリングを避ける
tags: rerender, useref, state, performance
---

## 一時的な値には useRef を使う

値が頻繁に変わり、更新のたびに再レンダリングしてほしくない場合（マウストラッカー、interval、一時的なフラグなど）は、`useState` ではなく `useRef` に保存してください。コンポーネント state は UI 用に使い、ref は一時的な DOM 近接値に使います。ref の更新では再レンダリングは発生しません。

**誤り（更新のたびに再描画される）:**

```tsx
function Tracker() {
  const [lastX, setLastX] = useState(0)

  useEffect(() => {
    const onMove = (e: MouseEvent) => setLastX(e.clientX)
    window.addEventListener('mousemove', onMove)
    return () => window.removeEventListener('mousemove', onMove)
  }, [])

  return (
    <div
      style={{
        position: 'fixed',
        top: 0,
        left: lastX,
        width: 8,
        height: 8,
        background: 'black',
      }}
    />
  )
}
```

**正しい例（トラッキングで再レンダリングしない）:**

```tsx
function Tracker() {
  const lastXRef = useRef(0)
  const dotRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const onMove = (e: MouseEvent) => {
      lastXRef.current = e.clientX
      const node = dotRef.current
      if (node) {
        node.style.transform = `translateX(${e.clientX}px)`
      }
    }
    window.addEventListener('mousemove', onMove)
    return () => window.removeEventListener('mousemove', onMove)
  }, [])

  return (
    <div
      ref={dotRef}
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        width: 8,
        height: 8,
        background: 'black',
        transform: 'translateX(0px)',
      }}
    />
  )
}
```
