---
title: RSC 境界でのシリアライズを最小化する
impact: HIGH
impactDescription: データ転送量を減らす
tags: server, rsc, serialization, props
---

## RSC 境界でのシリアライズを最小化する

React の Server / Client 境界では、すべてのオブジェクトプロパティが文字列にシリアライズされ、HTML レスポンスと、その後の RSC リクエストに埋め込まれます。このシリアライズ済みデータはページの重さと読み込み時間に直接影響するため、サイズは非常に重要です。クライアントが実際に使うフィールドだけを渡してください。

**悪い例（50 個のフィールドすべてをシリアライズする）:**

```tsx
async function Page() {
  const user = await fetchUser()  // 50 個のフィールド
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // 1 フィールドだけ使う
}
```

**良い例（1 個のフィールドだけをシリアライズする）:**

```tsx
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} />
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}
```
