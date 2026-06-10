---
title: スクロール性能のために passive イベントリスナーを使う
impact: MEDIUM
impactDescription: イベントリスナーによるスクロール遅延をなくす
tags: client, event-listeners, scrolling, performance, touch, wheel
---

## スクロール性能のために passive イベントリスナーを使う

touch と wheel のイベントリスナーには `{ passive: true }` を付けて、スクロールを即時に開始できるようにします。ブラウザは通常、`preventDefault()` が呼ばれるか確認するためにリスナーの完了を待つので、スクロール遅延が発生します。

**誤り:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch)
  document.addEventListener('wheel', handleWheel)
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

**正解:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch, { passive: true })
  document.addEventListener('wheel', handleWheel, { passive: true })
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

**passive を使う場面:** 計測/分析、ログ出力、`preventDefault()` を呼ばないすべてのリスナー。

**passive を使わない場面:** カスタムのスワイプ操作、独自のズーム操作、`preventDefault()` が必要なリスナー。
