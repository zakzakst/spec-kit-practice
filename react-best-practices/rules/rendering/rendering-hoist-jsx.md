---
title: 静的な JSX 要素は外へ持ち上げる
impact: LOW
impactDescription: 再生成を避ける
tags: rendering, jsx, static, optimization
---

## 静的な JSX 要素は外へ持ち上げる

静的な JSX をコンポーネントの外に切り出し、再生成を避けます。

**不適切（毎回のレンダーで要素を再生成する）:**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />
}

function Container() {
  return (
    <div>
      {loading && <LoadingSkeleton />}
    </div>
  )
}
```

**適切（同じ要素を再利用する）:**

```tsx
const loadingSkeleton = (
  <div className="animate-pulse h-20 bg-gray-200" />
)

function Container() {
  return (
    <div>
      {loading && loadingSkeleton}
    </div>
  )
}
```

これは、サイズの大きい静的な SVG ノードに特に有効です。毎回のレンダーで再生成するとコストが高くなります。

**注:** プロジェクトで [React Compiler](https://react.dev/learn/react-compiler) を有効にしている場合、コンパイラが静的な JSX 要素を自動的に持ち上げ、コンポーネントの再レンダーを最適化するため、手動での持ち上げは不要です。
