---
title: 派生ブール状態を購読する
impact: MEDIUM
impactDescription: 再レンダリング頻度を減らす
tags: rerender, derived-state, media-query, optimization
---

## 派生ブール状態を購読する

連続的な値ではなく、派生したブール状態を購読して再レンダリング頻度を減らしてください。

**誤り（ピクセル単位の変化で毎回再レンダリングされる）:**

```tsx
function Sidebar() {
  const width = useWindowWidth()  // 継続的に更新される
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

**正しい例（ブール値が変わったときだけ再レンダリングされる）:**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```
