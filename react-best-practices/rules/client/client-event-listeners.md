---
title: グローバルイベントリスナーを重複登録しない
impact: LOW
impactDescription: N 個のコンポーネントで 1 つのリスナーを共有
tags: client, swr, event-listeners, subscription
---

## グローバルイベントリスナーを重複登録しない

`useSWRSubscription()` を使って、グローバルなイベントリスナーをコンポーネントインスタンス間で共有します。

**誤り（N 個のインスタンス = N 個のリスナー）:**

```tsx
function useKeyboardShortcut(key: string, callback: () => void) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && e.key === key) {
        callback()
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  }, [key, callback])
}
```

`useKeyboardShortcut` フックを複数回使うと、各インスタンスが新しいリスナーを登録してしまいます。

**正解（N 個のインスタンス = 1 個のリスナー）:**

```tsx
import useSWRSubscription from 'swr/subscription'

// キーごとのコールバックを追跡するためのモジュールレベルの Map
const keyCallbacks = new Map<string, Set<() => void>>()

function useKeyboardShortcut(key: string, callback: () => void) {
  // このコールバックを Map に登録する
  useEffect(() => {
    if (!keyCallbacks.has(key)) {
      keyCallbacks.set(key, new Set())
    }
    keyCallbacks.get(key)!.add(callback)

    return () => {
      const set = keyCallbacks.get(key)
      if (set) {
        set.delete(callback)
        if (set.size === 0) {
          keyCallbacks.delete(key)
        }
      }
    }
  }, [key, callback])

  useSWRSubscription('global-keydown', () => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && keyCallbacks.has(e.key)) {
        keyCallbacks.get(e.key)!.forEach(cb => cb())
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  })
}

function Profile() {
  // 複数のショートカットが同じリスナーを共有する
  useKeyboardShortcut('p', () => { /* ... */ }) 
  useKeyboardShortcut('k', () => { /* ... */ })
  // ...
}
```
