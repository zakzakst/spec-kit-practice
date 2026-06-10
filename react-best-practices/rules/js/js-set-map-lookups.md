---
title: O(1) 検索には Set/Map を使う
impact: LOW-MEDIUM
impactDescription: O(n) を O(1) にする
tags: javascript, set, map, data-structures, performance
---

## O(1) 検索には Set/Map を使う

繰り返し membership チェックをするなら、配列を `Set` / `Map` に変換してください。

**誤り（1 回のチェックあたり O(n)）:**

```typescript
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))
```

**正しい例（1 回のチェックあたり O(1)）:**

```typescript
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))
```
