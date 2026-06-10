---
title: 派生状態をレンダリング中に計算する
impact: MEDIUM
impactDescription: 冗長な再レンダリングと state のずれを防ぐ
tags: rerender, derived-state, useEffect, state
---

## 派生状態をレンダリング中に計算する

値が現在の props/state から計算できるなら、それを state に保存したり effect で更新したりしないでください。レンダリング中に派生させれば、余計な再レンダリングと state のずれを避けられます。props の変化に応じるためだけに effect で state を更新するのではなく、派生値やキー付きリセットを優先してください。

**誤り（冗長な state と effect）:**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const [fullName, setFullName] = useState('')

  useEffect(() => {
    setFullName(firstName + ' ' + lastName)
  }, [firstName, lastName])

  return <p>{fullName}</p>
}
```

**正しい例（レンダリング中に派生させる）:**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const fullName = firstName + ' ' + lastName

  return <p>{fullName}</p>
}
```

参考: [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
