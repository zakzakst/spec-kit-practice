---
title: Effect の依存関係を絞る
impact: LOW
impactDescription: effect の再実行を最小化する
tags: rerender, useEffect, dependencies, optimization
---

## Effect の依存関係を絞る

オブジェクトではなくプリミティブな依存値を指定して、effect の再実行を最小限にしてください。

**誤り（user のどのフィールドが変わっても再実行される）:**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user])
```

**正しい例（id が変わったときだけ再実行される）:**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user.id])
```

**派生状態は effect の外で計算する:**

```tsx
// 誤り: width=767, 766, 765... で毎回走る
useEffect(() => {
  if (width < 768) {
    enableMobileMode()
  }
}, [width])

// 正しい例: ブール値の切り替わり時だけ走る
const isMobile = width < 768
useEffect(() => {
  if (isMobile) {
    enableMobileMode()
  }
}, [isMobile])
```
