---
title: メモ化コンポーネントの非プリミティブなデフォルト値は定数に切り出す
impact: MEDIUM
impactDescription: デフォルト値を定数にしてメモ化を保つ
tags: rerender, memo, optimization
---

## メモ化コンポーネントの非プリミティブなデフォルト値は定数に切り出す

メモ化コンポーネントに配列・関数・オブジェクトのような非プリミティブな省略可能パラメータのデフォルト値がある場合、そのパラメータを渡さずに呼び出すとメモ化が壊れます。毎回の再レンダリングで新しい値インスタンスが作られ、`memo()` の厳密比較に通らないためです。

この問題を避けるには、デフォルト値を定数に切り出してください。

**誤り（`onClick` の値が毎回変わる）:**

```tsx
const UserAvatar = memo(function UserAvatar({ onClick = () => {} }: { onClick?: () => void }) {
  // ...
})

// 省略可能な onClick を渡さずに使う
<UserAvatar />
```

**正しい例（安定したデフォルト値）:**

```tsx
const NOOP = () => {};

const UserAvatar = memo(function UserAvatar({ onClick = NOOP }: { onClick?: () => void }) {
  // ...
})

// 省略可能な onClick を渡さずに使う
<UserAvatar />
```
