---
title: Storage API 呼び出しをキャッシュする
impact: LOW-MEDIUM
impactDescription: 高コストな I/O を減らす
tags: javascript, localStorage, storage, caching, performance
---

## Storage API 呼び出しをキャッシュする

`localStorage`、`sessionStorage`、`document.cookie` は同期的で高コストです。読み取り結果はメモリにキャッシュしてください。

**誤り（呼び出しのたびに storage を読む）:**

```typescript
function getTheme() {
  return localStorage.getItem('theme') ?? 'light'
}
// 10 回呼ばれる = storage の読み取りも 10 回
```

**正しい例（Map でキャッシュする）:**

```typescript
const storageCache = new Map<string, string | null>()

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key))
  }
  return storageCache.get(key)
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value)
  storageCache.set(key, value)  // キャッシュを同期しておく
}
```

React コンポーネントだけでなく、ユーティリティやイベントハンドラなどどこでも使えるように、フックではなく `Map` を使ってください。

**Cookie のキャッシュ:**

```typescript
let cookieCache: Record<string, string> | null = null

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split('; ').map(c => c.split('='))
    )
  }
  return cookieCache[name]
}
```

**重要（外部変更時は無効化する）:**

Storage が外部から変更される可能性がある場合（別タブ、サーバー設定の cookie など）は、キャッシュを無効化してください。

```typescript
window.addEventListener('storage', (e) => {
  if (e.key) storageCache.delete(e.key)
})

document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible') {
    storageCache.clear()
  }
})
```
