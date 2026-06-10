---
title: 安定したコールバック参照には useEffectEvent を使う
impact: LOW
impactDescription: effect の再実行を防ぐ
tags: advanced, hooks, useEffectEvent, refs, optimization
---

## 安定したコールバック参照には useEffectEvent を使う

依存配列に追加せずに、コールバック内で最新の値へアクセスできます。古い closure を避けながら、effect の再実行も防げます。

**誤り: コールバックが変わるたびに effect が再実行される**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300)
    return () => clearTimeout(timeout)
  }, [query, onSearch])
}
```

**正しい例: React の `useEffectEvent` を使う**

```tsx
import { useEffectEvent } from 'react';

function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const onSearchEvent = useEffectEvent(onSearch)

  useEffect(() => {
    const timeout = setTimeout(() => onSearchEvent(query), 300)
    return () => clearTimeout(timeout)
  }, [query])
}
```
