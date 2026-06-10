---
title: 遅延 state 初期化を使う
impact: MEDIUM
impactDescription: 毎回のレンダリングで無駄な計算が走る
tags: react, hooks, useState, performance, initialization
---

## 遅延 state 初期化を使う

高コストな初期値には `useState` に関数を渡してください。関数形式を使わないと、値が 1 回しか使われないにもかかわらず、initializer が毎回のレンダリングで実行されます。

**誤り（毎回のレンダリングで実行される）:**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() は初期化後も毎回のレンダリングで実行される
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items))
  const [query, setQuery] = useState('')

  // query が変わるたびに buildSearchIndex が不要に再実行される
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse が毎回のレンダリングで実行される
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem('settings') || '{}')
  )

  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

**正しい例（1 回だけ実行される）:**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() は初回レンダリングでのみ実行される
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items))
  const [query, setQuery] = useState('')

  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse は初回レンダリングでのみ実行される
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {}
  })

  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

`localStorage` / `sessionStorage` から初期値を計算するとき、データ構造（インデックス、Map など）を構築するとき、DOM を読むとき、重い変換を行うときは遅延初期化を使ってください。

`useState(0)` のような単純なプリミティブ、`useState(props.value)` のような直接参照、`useState({})` のような軽いリテラルでは、関数形式は不要です。
