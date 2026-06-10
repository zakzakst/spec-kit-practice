---
title: 手動のローディング状態より useTransition を使う
impact: LOW
impactDescription: 再レンダーを減らし、コードをわかりやすくする
tags: rendering, transitions, useTransition, loading, state
---

## 手動のローディング状態より useTransition を使う

ローディング状態には、手動の `useState` ではなく `useTransition` を使います。組み込みの `isPending` 状態が使え、トランジションも自動で管理されます。

**不適切（手動のローディング状態）:**

```tsx
function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isLoading, setIsLoading] = useState(false)

  const handleSearch = async (value: string) => {
    setIsLoading(true)
    setQuery(value)
    const data = await fetchResults(value)
    setResults(data)
    setIsLoading(false)
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isLoading && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

**適切（`useTransition` と組み込みの pending 状態）:**

```tsx
import { useTransition, useState } from 'react'

function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()

  const handleSearch = (value: string) => {
    setQuery(value) // 入力をすぐに更新する
    
    startTransition(async () => {
      // 結果を取得して更新する
      const data = await fetchResults(value)
      setResults(data)
    })
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

**利点:**

- **pending 状態を自動管理**: `setIsLoading(true/false)` を手動で扱う必要がない
- **エラーに強い**: トランジションが例外を投げても pending 状態は正しくリセットされる
- **応答性が高い**: 更新中も UI の応答性を保てる
- **割り込みを処理できる**: 新しいトランジションが古い pending 状態を自動で打ち消す

Reference: [useTransition](https://react.dev/reference/react/useTransition)
