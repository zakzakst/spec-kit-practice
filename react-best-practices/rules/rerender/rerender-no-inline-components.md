---
title: コンポーネントをコンポーネント内で定義しない
impact: HIGH
impactDescription: 毎回のレンダーでの再マウントを防ぐ
tags: rerender, components, remount, performance
---

## コンポーネントをコンポーネント内で定義しない

**影響: 高（毎回のレンダーで再マウントを防ぐ）**

別のコンポーネントの中でコンポーネントを定義すると、レンダーのたびに新しいコンポーネント型が作られます。React は毎回別物として扱い、完全に再マウントするため、すべての state と DOM が失われます。

親の変数を props なしで使いたいことが、この書き方を選ぶよくある理由です。代わりに、必ず props を渡してください。

**誤り（毎回のレンダーで再マウントされる）:**

```tsx
function UserProfile({ user, theme }) {
  // `theme` を使うために内側で定義している - NG
  const Avatar = () => (
    <img
      src={user.avatarUrl}
      className={theme === 'dark' ? 'avatar-dark' : 'avatar-light'}
    />
  )

  // `user` を使うために内側で定義している - NG
  const Stats = () => (
    <div>
      <span>{user.followers} followers</span>
      <span>{user.posts} posts</span>
    </div>
  )

  return (
    <div>
      <Avatar />
      <Stats />
    </div>
  )
}
```

`UserProfile` がレンダーされるたびに、`Avatar` と `Stats` は新しいコンポーネント型になります。React は古いインスタンスを unmount して新しいものを mount するため、内部 state は失われ、effect は再実行され、DOM ノードも作り直されます。

**正しい例（代わりに props を渡す）:**

```tsx
function Avatar({ src, theme }: { src: string; theme: string }) {
  return (
    <img
      src={src}
      className={theme === 'dark' ? 'avatar-dark' : 'avatar-light'}
    />
  )
}

function Stats({ followers, posts }: { followers: number; posts: number }) {
  return (
    <div>
      <span>{followers} followers</span>
      <span>{posts} posts</span>
    </div>
  )
}

function UserProfile({ user, theme }) {
  return (
    <div>
      <Avatar src={user.avatarUrl} theme={theme} />
      <Stats followers={user.followers} posts={user.posts} />
    </div>
  )
}
```

**このバグの症状:**
- 入力欄が文字入力のたびにフォーカスを失う
- アニメーションが予期せず最初から再生される
- 親のレンダーのたびに `useEffect` の cleanup / setup が走る
- コンポーネント内のスクロール位置がリセットされる
