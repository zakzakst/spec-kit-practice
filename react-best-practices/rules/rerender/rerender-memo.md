---
title: メモ化コンポーネントへ抽出する
impact: MEDIUM
impactDescription: 早期 return を可能にする
tags: rerender, memo, useMemo, optimization
---

## メモ化コンポーネントへ抽出する

高コストな処理はメモ化コンポーネントに切り出して、計算前に早期 return できるようにしてください。

**誤り（loading 中でも avatar を計算する）:**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user)
    return <Avatar id={id} />
  }, [user])

  if (loading) return <Skeleton />
  return <div>{avatar}</div>
}
```

**正しい例（loading 中は計算をスキップする）:**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user])
  return <Avatar id={id} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  )
}
```

**注:** プロジェクトで [React Compiler](https://react.dev/learn/react-compiler) を有効にしている場合、`memo()` や `useMemo()` を手動で書く必要はありません。コンパイラが再レンダリングを自動で最適化します。
