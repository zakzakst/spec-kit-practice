---
title: 結合された Hook 計算を分割する
impact: MEDIUM
impactDescription: 独立した処理の再計算を避ける
tags: rerender, useMemo, useEffect, dependencies, optimization
---

## 結合された Hook 計算を分割する

1 つの hook の中に、依存値の異なる独立した処理が複数あるなら、別々の hook に分けてください。結合された hook は、ある依存値が変わるたびに、他の処理で使っていなくてもすべて再実行されます。

**誤り（`sortOrder` が変わるたびに filtering まで再計算される）:**

```tsx
const sortedProducts = useMemo(() => {
  const filtered = products.filter((p) => p.category === category)
  const sorted = filtered.toSorted((a, b) =>
    sortOrder === "asc" ? a.price - b.price : b.price - a.price
  )
  return sorted
}, [products, category, sortOrder])
```

**正しい例（filtering は products または category が変わったときだけ再計算される）:**

```tsx
const filteredProducts = useMemo(
  () => products.filter((p) => p.category === category),
  [products, category]
)

const sortedProducts = useMemo(
  () =>
    filteredProducts.toSorted((a, b) =>
      sortOrder === "asc" ? a.price - b.price : b.price - a.price
    ),
  [filteredProducts, sortOrder]
)
```

このパターンは、関係のない副作用を 1 つにまとめた `useEffect` にも当てはまります。

**誤り（どちらか一方の依存値が変わるだけで両方の effect が走る）:**

```tsx
useEffect(() => {
  analytics.trackPageView(pathname)
  document.title = `${pageTitle} | My App`
}, [pathname, pageTitle])
```

**正しい例（effect を独立して走らせる）:**

```tsx
useEffect(() => {
  analytics.trackPageView(pathname)
}, [pathname])

useEffect(() => {
  document.title = `${pageTitle} | My App`
}, [pageTitle])
```

**注:** プロジェクトで [React Compiler](https://react.dev/learn/react-compiler) を有効にしている場合、依存関係の追跡はコンパイラが自動で最適化し、これらのケースの一部は自動で扱われます。
