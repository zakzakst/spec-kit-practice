---
title: 重い派生レンダーには useDeferredValue を使う
impact: MEDIUM
impactDescription: 重い計算中も入力の応答性を保つ
tags: rerender, useDeferredValue, optimization, concurrent
---

## 重い派生レンダーには useDeferredValue を使う

ユーザー入力が高コストな計算やレンダーを引き起こすなら、`useDeferredValue` を使って入力の応答性を保ってください。deferred な値は少し遅れて更新されるので、React は入力更新を優先し、重い結果は空き時間に描画できます。

**誤り（フィルタリング中に入力がもたつく）:**

```tsx
function Search({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('')
  const filtered = items.filter(item => fuzzyMatch(item, query))

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <ResultsList results={filtered} />
    </>
  )
}
```

**正しい例（入力は軽快に保ち、結果は準備できたら描画する）:**

```tsx
function Search({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('')
  const deferredQuery = useDeferredValue(query)
  const filtered = useMemo(
    () => items.filter(item => fuzzyMatch(item, deferredQuery)),
    [items, deferredQuery]
  )
  const isStale = query !== deferredQuery

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <div style={{ opacity: isStale ? 0.7 : 1 }}>
        <ResultsList results={filtered} />
      </div>
    </>
  )
}
```

**使う場面:**

- 大きなリストのフィルタリング / 検索
- 入力に反応する重い可視化（チャート、グラフ）
- 目に見える遅延を引き起こす派生 state 全般

**注:** 重い計算は、`useDeferredValue` を依存値にして `useMemo` で包んでください。そうしないと、毎回のレンダーで計算され続けます。

参考: [React useDeferredValue](https://react.dev/reference/react/useDeferredValue)
