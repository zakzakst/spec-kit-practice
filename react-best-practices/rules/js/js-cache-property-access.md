---
title: ループ内のプロパティ参照をキャッシュする
impact: LOW-MEDIUM
impactDescription: 参照回数を減らす
tags: javascript, loops, optimization, caching
---

## ループ内のプロパティ参照をキャッシュする

ホットパスでは、オブジェクトのプロパティ参照をキャッシュしてください。

**誤り（3 回の参照 × N 回の反復）:**

```typescript
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value)
}
```

**正しい例（参照は合計 1 回）:**

```typescript
const value = obj.config.settings.value
const len = arr.length
for (let i = 0; i < len; i++) {
  process(value)
}
```
