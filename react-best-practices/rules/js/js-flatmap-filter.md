---
title: flatMap で map と filter を 1 回で行う
impact: LOW-MEDIUM
impactDescription: 中間配列をなくす
tags: javascript, arrays, flatMap, filter, performance
---

## flatMap で map と filter を 1 回で行う

**影響: LOW-MEDIUM（中間配列をなくす）**

`.map().filter(Boolean)` をつなげると中間配列が作られ、2 回走査されます。`.flatMap()` を使って、変換とフィルタリングを 1 回で行ってください。

**誤り（2 回の反復、中間配列あり）:**

```typescript
const userNames = users
  .map(user => user.isActive ? user.name : null)
  .filter(Boolean)
```

**正しい例（1 回の反復、中間配列なし）:**

```typescript
const userNames = users.flatMap(user =>
  user.isActive ? [user.name] : []
)
```

**追加例:**

```typescript
// レスポンスから有効な email を抽出する
// Before
const emails = responses
  .map(r => r.success ? r.data.email : null)
  .filter(Boolean)

// After
const emails = responses.flatMap(r =>
  r.success ? [r.data.email] : []
)

// 有効な数値を解析して抽出する
// Before
const numbers = strings
  .map(s => parseInt(s, 10))
  .filter(n => !isNaN(n))

// After
const numbers = strings.flatMap(s => {
  const n = parseInt(s, 10)
  return isNaN(n) ? [] : [n]
})
```

**使いどころ:**
- アイテムを変換しつつ、一部を除外したいとき
- 条件付きで map し、入力によっては出力が 0 件のとき
- 無効な入力をスキップしたい解析・検証処理
