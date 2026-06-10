---
title: ネストしたデータ取得を並列化する
impact: CRITICAL
impactDescription: サーバー側のウォーターフォールをなくす
tags: server, rsc, parallel-fetching, promise-chaining
---

## ネストしたデータ取得を並列化する

ネストしたデータを並列で取得するときは、各アイテムの Promise 内で依存する fetch をつなげて、1 件の遅いアイテムが他を止めないようにします。

**悪い例（1 件の遅いアイテムが、すべてのネストした fetch を止める）:**

```tsx
const chats = await Promise.all(
  chatIds.map(id => getChat(id))
)

const chatAuthors = await Promise.all(
  chats.map(chat => getUser(chat.author))
)
```

100 件のうち 1 件の `getChat(id)` が非常に遅い場合でも、その 99 件の author は準備できているのに読み込みを始められません。

**良い例（各アイテムが自分のネストした fetch をつなげる）:**

```tsx
const chatAuthors = await Promise.all(
  chatIds.map(id => getChat(id).then(chat => getUser(chat.author)))
)
```

各アイテムが `getChat` から `getUser` へ独立してつながるため、1 件の遅い chat があっても他の author の fetch は止まりません。
