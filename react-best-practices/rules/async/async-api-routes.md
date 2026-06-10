---
title: API ルートでウォーターフォール連鎖を防ぐ
impact: CRITICAL
impactDescription: 2〜10倍の改善
tags: api-routes, server-actions, waterfalls, parallelization
---

## API ルートでウォーターフォール連鎖を防ぐ

API ルートや Server Action では、まだ `await` しない独立した処理でも、すぐに開始してください。

**誤り（config が auth を待ち、data が両方を待つ）:**

```typescript
export async function GET(request: Request) {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchData(session.user.id)
  return Response.json({ data, config })
}
```

**正しい例（auth と config をすぐに開始する）:**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id)
  ])
  return Response.json({ data, config })
}
```

より複雑な依存関係がある処理では、`better-all` を使うと並列性を自動で最大化できます（[Dependency-Based Parallelization](./async-dependencies.md) を参照）。
