---
title: 必要になるまで await を遅らせる
impact: HIGH
impactDescription: 使われないコードパスをブロックしない
tags: async, await, conditional, optimization
---

## 必要になるまで await を遅らせる

`await` を実際に使う分岐の中へ移動し、必要のないコードパスをブロックしないようにします。

**誤り（両方の分岐をブロックする）:**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // すぐに返るが、userData を待ってしまっている
    return { skipped: true }
  }
  
  // この分岐だけが userData を使う
  return processUserData(userData)
}
```

**正しい例（必要なときだけブロックする）:**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // 待たずにすぐ返す
    return { skipped: true }
  }
  
  // 必要なときだけ取得する
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

**別の例（早期 return の最適化）:**

```typescript
// 誤り: 常に permissions を取得する
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId)
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}

// 正しい例: 必要なときだけ取得する
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  const permissions = await fetchPermissions(userId)
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}
```

この最適化は、スキップされる分岐が頻繁に選ばれる場合や、遅延した処理が高コストな場合に特に有効です。

`await getFlag()` と安価な同期ガード（`flag && someCondition`）を組み合わせる場合は、[非同期フラグより先に安価な条件を確認する](./async-cheap-condition-before-await.md) を参照してください。
