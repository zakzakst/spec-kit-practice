---
title: `React.cache()` によるリクエスト単位の重複排除
impact: MEDIUM
impactDescription: 1 リクエスト内で重複排除する
tags: server, cache, react-cache, deduplication
---

## `React.cache()` によるリクエスト単位の重複排除

サーバー側のリクエスト内重複排除には `React.cache()` を使います。認証とデータベースクエリで特に効果があります。

**使い方:**

```typescript
import { cache } from 'react'

export const getCurrentUser = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return await db.user.findUnique({
    where: { id: session.user.id }
  })
})
```

1 つのリクエスト内では、`getCurrentUser()` を複数回呼んでもクエリは 1 回しか実行されません。

**引数にインラインオブジェクトを使わない:**

`React.cache()` はキャッシュヒット判定に浅い比較 (`Object.is`) を使います。インラインオブジェクトは呼び出しごとに新しい参照を作るため、キャッシュヒットしません。

**悪い例（常にキャッシュミス）:**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } })
})

// 毎回新しいオブジェクトが作られるため、絶対にヒットしない
getUser({ uid: 1 })
getUser({ uid: 1 })  // キャッシュミス。クエリが再実行される
```

**良い例（キャッシュヒット）:**

```typescript
const getUser = cache(async (uid: number) => {
  return await db.user.findUnique({ where: { id: uid } })
})

// プリミティブ引数は値の等価性で比較される
getUser(1)
getUser(1)  // キャッシュヒット。保存済みの結果を返す
```

オブジェクトを渡す必要がある場合は、同じ参照を渡してください。

```typescript
const params = { uid: 1 }
getUser(params)  // クエリが実行される
getUser(params)  // キャッシュヒット（同じ参照）
```

**Next.js 特有の注意:**

Next.js では、`fetch` API にリクエストメモ化が自動で追加されています。同じ URL とオプションを持つリクエストは、1 リクエスト内で自動的に重複排除されるため、`fetch` 呼び出しに `React.cache()` は不要です。ただし、`React.cache()` は次のような他の非同期処理には引き続き重要です。

- データベースクエリ（Prisma、Drizzle など）
- 重い計算処理
- 認証チェック
- ファイルシステム操作
- `fetch` 以外のあらゆる非同期処理

これらの処理をコンポーネントツリー全体で重複排除するために `React.cache()` を使います。

Reference: [React.cache documentation](https://react.dev/reference/react/cache)
