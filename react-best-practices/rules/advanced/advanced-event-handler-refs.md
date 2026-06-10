---
title: イベントハンドラーを Ref に保持する
impact: LOW
impactDescription: 安定したサブスクリプション
tags: advanced, hooks, refs, event-handlers, optimization
---

## イベントハンドラーを Ref に保持する

コールバックの変更で再サブスクライブしたくない effect では、コールバックを ref に保持してください。

**誤り: 毎回のレンダーで再サブスクライブしてしまう**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}
```

**正しい例: サブスクリプションを安定させる**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  const handlerRef = useRef(handler)
  useEffect(() => {
    handlerRef.current = handler
  }, [handler])

  useEffect(() => {
    const listener = (e) => handlerRef.current(e)
    window.addEventListener(event, listener)
    return () => window.removeEventListener(event, listener)
  }, [event])
}
```

**代替案: 最新の React なら `useEffectEvent` を使う**

```tsx
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e) => void) {
  const onEvent = useEffectEvent(handler)

  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}
```

`useEffectEvent` は同じパターンをより簡潔に書ける API です。常に最新の handler を呼び出す、安定した関数参照を作成します。
