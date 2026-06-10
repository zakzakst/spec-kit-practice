---
title: インタラクションロジックはイベントハンドラに置く
impact: MEDIUM
impactDescription: effect の再実行と重複した副作用を避ける
tags: rerender, useEffect, events, side-effects, dependencies
---

## インタラクションロジックはイベントハンドラに置く

副作用が特定のユーザー操作（送信、クリック、ドラッグ）で発火するなら、そのイベントハンドラの中で実行してください。操作を state + effect として表現しないでください。無関係な変更で effect が再実行され、同じ操作が重複することがあります。

**誤り（イベントを state + effect で表現している）:**

```tsx
function Form() {
  const [submitted, setSubmitted] = useState(false)
  const theme = useContext(ThemeContext)

  useEffect(() => {
    if (submitted) {
      post('/api/register')
      showToast('登録しました', theme)
    }
  }, [submitted, theme])

  return <button onClick={() => setSubmitted(true)}>送信</button>
}
```

**正しい例（ハンドラの中で実行する）:**

```tsx
function Form() {
  const theme = useContext(ThemeContext)

  function handleSubmit() {
    post('/api/register')
    showToast('登録しました', theme)
  }

  return <button onClick={handleSubmit}>送信</button>
}
```

参考: [このコードはイベントハンドラへ移すべきか？](https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler)
