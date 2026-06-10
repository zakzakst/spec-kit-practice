---
title: 表示/非表示には Activity コンポーネントを使う
impact: MEDIUM
impactDescription: 状態と DOM を保持する
tags: rendering, activity, visibility, state-preservation
---

## 表示/非表示には Activity コンポーネントを使う

React の `<Activity>` を使って、表示/非表示を頻繁に切り替える高コストなコンポーネントの状態と DOM を保持します。

**使用例:**

```tsx
import { Activity } from 'react'

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? 'visible' : 'hidden'}>
      <ExpensiveMenu />
    </Activity>
  )
}
```

高コストな再レンダリングと状態消失を避けられます。
