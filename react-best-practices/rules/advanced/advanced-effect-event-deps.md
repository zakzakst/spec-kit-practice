---
title: Effect Event を依存配列に入れない
impact: LOW
impactDescription: 不要な effect の再実行と lint エラーを防ぐ
tags: advanced, hooks, useEffectEvent, dependencies, effects
---

## Effect Event を依存配列に入れない

Effect Event の関数は安定した identity を持ちません。identity は意図的に毎回のレンダーで変わります。`useEffectEvent` が返す関数を `useEffect` の依存配列に含めないでください。実際に反応すべき値だけを依存関係として残し、Effect Event は effect 本体か、その effect が作成したサブスクリプションの中から呼び出してください。

**誤り: Effect Event を依存関係に追加している**

```tsx
import { useEffect, useEffectEvent } from 'react'

function ChatRoom({ roomId, onConnected }: {
  roomId: string
  onConnected: () => void
}) {
  const handleConnected = useEffectEvent(onConnected)

  useEffect(() => {
    const connection = createConnection(roomId)
    connection.on('connected', handleConnected)
    connection.connect()

    return () => connection.disconnect()
  }, [roomId, handleConnected])
}
```

Effect Event を依存配列に入れると、毎回のレンダーで effect が再実行され、React Hooks の lint ルールにも引っかかります。

**正しい例: Effect Event ではなく、反応的な値に依存する**

```tsx
import { useEffect, useEffectEvent } from 'react'

function ChatRoom({ roomId, onConnected }: {
  roomId: string
  onConnected: () => void
}) {
  const handleConnected = useEffectEvent(onConnected)

  useEffect(() => {
    const connection = createConnection(roomId)
    connection.on('connected', handleConnected)
    connection.connect()

    return () => connection.disconnect()
  }, [roomId])
}
```

参考: [React useEffectEvent: Effect Event in deps](https://react.dev/reference/react/useEffectEvent#effect-event-in-deps)
