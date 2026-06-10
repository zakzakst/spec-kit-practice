---
title: 繰り返し検索用にインデックス Map を作る
impact: LOW-MEDIUM
impactDescription: 100 万回の操作を 2000 回に減らす
tags: javascript, map, indexing, optimization, performance
---

## 繰り返し検索用にインデックス Map を作る

同じキーで `.find()` を何度も行うなら、`Map` を使ってください。

**誤り（1 回の検索あたり O(n)）:**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map(order => ({
    ...order,
    user: users.find(u => u.id === order.userId)
  }))
}
```

**正しい例（1 回の検索あたり O(1)）:**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map(u => [u.id, u]))

  return orders.map(order => ({
    ...order,
    user: userById.get(order.userId)
  }))
}
```

`Map` を 1 回だけ作成し（O(n)）、その後の検索をすべて O(1) にします。
1000 件の注文 × 1000 件のユーザーなら、100 万回の操作が 2000 回で済みます。
