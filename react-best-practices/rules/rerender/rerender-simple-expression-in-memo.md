---
title: プリミティブ結果の単純な式を useMemo で包まない
impact: LOW-MEDIUM
impactDescription: 毎回のレンダーで無駄な計算が走る
tags: rerender, useMemo, optimization
---

## プリミティブ結果の単純な式を useMemo で包まない

式が単純で（論理演算子や算術演算子が少ない）、結果の型がプリミティブ（boolean, number, string）なら、`useMemo` で包まないでください。
`useMemo` の呼び出しと依存関係の比較のほうが、その式自体よりコストが高くなることがあります。

**誤り:**

```tsx
function Header({ user, notifications }: Props) {
  const isLoading = useMemo(() => {
    return user.isLoading || notifications.isLoading
  }, [user.isLoading, notifications.isLoading])

  if (isLoading) return <Skeleton />
  // some markup を返す
}
```

**正しい例:**

```tsx
function Header({ user, notifications }: Props) {
  const isLoading = user.isLoading || notifications.isLoading

  if (isLoading) return <Skeleton />
  // some markup を返す
}
```
