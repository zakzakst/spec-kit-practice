---
title: 関数形式の setState 更新を使う
impact: MEDIUM
impactDescription: stale closure と不要なコールバック再生成を防ぐ
tags: react, hooks, useState, useCallback, callbacks, closures
---

## 関数形式の setState 更新を使う

現在の state をもとに state を更新する場合は、state 変数を直接参照するのではなく、`setState` の関数形式を使ってください。これにより stale closure を防ぎ、不要な依存関係をなくし、安定したコールバック参照を作れます。

**誤り（state を依存値として必要とする）:**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)

  // このコールバックは items に依存し、items が変わるたびに再生成される
  const addItems = useCallback((newItems: Item[]) => {
    setItems([...items, ...newItems])
  }, [items])  // items への依存で再生成が発生する

  // 依存値を忘れると stale closure になる
  const removeItem = useCallback((id: string) => {
    setItems(items.filter(item => item.id !== id))
  }, [])  // items への依存がないため、古い items を使ってしまう

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

最初のコールバックは `items` が変わるたびに再生成されるため、子コンポーネントが不要に再レンダリングされることがあります。2 つ目のコールバックには stale closure のバグがあり、常に初期の `items` を参照してしまいます。

**正しい例（安定したコールバック、stale closure なし）:**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)

  // 安定したコールバックで、再生成されない
  const addItems = useCallback((newItems: Item[]) => {
    setItems(curr => [...curr, ...newItems])
  }, [])  // 依存値は不要

  // 常に最新の state を使うので、stale closure の心配がない
  const removeItem = useCallback((id: string) => {
    setItems(curr => curr.filter(item => item.id !== id))
  }, [])  // 安全で安定している

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

**利点:**

1. **安定したコールバック参照** - state が変わってもコールバックを再生成する必要がない
2. **stale closure がない** - 常に最新の state 値に対して動作する
3. **依存値が少ない** - 依存配列が簡潔になり、メモリリークのリスクも減る
4. **バグを防ぐ** - React でよくある closure バグの原因を取り除ける

**関数形式の更新を使う場面:**

- 現在の state 値に依存するあらゆる `setState`
- state が必要な `useCallback` / `useMemo` の中
- state を参照するイベントハンドラ
- state を更新する非同期処理

**直接更新で問題ない場面:**

- 静的な値を設定する場合: `setCount(0)`
- props / 引数のみから値を設定する場合: `setName(newName)`
- state が前の値に依存しない場合

**注:** プロジェクトで [React Compiler](https://react.dev/learn/react-compiler) を有効にしている場合、いくつかのケースはコンパイラが自動で最適化できますが、正しさを保ち stale closure のバグを防ぐために、関数形式の更新は今でも推奨されます。
