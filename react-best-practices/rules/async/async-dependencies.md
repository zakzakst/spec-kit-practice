---
title: 依存関係ベースの並列化
impact: CRITICAL
impactDescription: 2〜10倍の改善
tags: async, parallelization, dependencies, better-all
---

## 依存関係ベースの並列化

一部だけ依存関係がある処理では、`better-all` を使って並列性を最大化してください。各タスクを可能な限り早いタイミングで自動的に開始してくれます。

**誤り（profile が不要に config を待つ）:**

```typescript
const [user, config] = await Promise.all([
  fetchUser(),
  fetchConfig()
])
const profile = await fetchProfile(user.id)
```

**正しい例（config と profile を並列実行する）:**

```typescript
import { all } from 'better-all'

const { user, config, profile } = await all({
  async user() { return fetchUser() },
  async config() { return fetchConfig() },
  async profile() {
    return fetchProfile((await this.$.user).id)
  }
})
```

**追加依存なしの代替案:**

すべての Promise を先に作成し、最後に `Promise.all()` でまとめて待つこともできます。

```typescript
const userPromise = fetchUser()
const profilePromise = userPromise.then(user => fetchProfile(user.id))

const [user, config, profile] = await Promise.all([
  userPromise,
  fetchConfig(),
  profilePromise
])
```

参考: [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
