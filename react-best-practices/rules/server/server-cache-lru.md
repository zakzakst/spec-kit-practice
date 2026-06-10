---
title: リクエストをまたぐ LRU キャッシュ
impact: HIGH
impactDescription: リクエスト間でキャッシュする
tags: server, cache, lru, cross-request
---

## リクエストをまたぐ LRU キャッシュ

`React.cache()` は 1 リクエスト内でしか機能しません。連続したリクエスト間で共有したいデータ（たとえば、ユーザーがボタン A を押したあとにボタン B を押すようなケース）には、LRU キャッシュを使います。

**実装:**

```typescript
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000  // 5 分
})

export async function getUser(id: string) {
  const cached = cache.get(id)
  if (cached) return cached

  const user = await db.user.findUnique({ where: { id } })
  cache.set(id, user)
  return user
}

// 1 回目のリクエスト: DB クエリ、結果をキャッシュ
// 2 回目のリクエスト: キャッシュヒット、DB クエリなし
```

数秒以内に同じデータへアクセスする複数のユーザー操作がある場合に使います。

**Vercel の [Fluid Compute](https://vercel.com/docs/fluid-compute) を使う場合:** LRU キャッシュは特に効果的です。複数の同時リクエストが同じ関数インスタンスとキャッシュを共有できるためです。つまり、Redis のような外部ストレージがなくても、キャッシュはリクエスト間で保持されます。

**従来型のサーバーレスの場合:** 各実行は独立しているため、プロセスをまたぐキャッシュには Redis を検討してください。

Reference: [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)
