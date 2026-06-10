---
title: 不変性のために sort() ではなく toSorted() を使う
impact: MEDIUM-HIGH
impactDescription: React の state で mutation バグを防ぐ
tags: javascript, arrays, immutability, react, state, mutation
---

## 不変性のために sort() ではなく toSorted() を使う

`.sort()` は配列をその場で変更するため、React の state や props でバグの原因になります。`.toSorted()` を使うと、元の配列を変更せずに新しいソート済み配列を作れます。

**誤り（元の配列を変更する）:**

```typescript
function UserList({ users }: { users: User[] }) {
  // users prop の配列を変更してしまう
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**正しい例（新しい配列を作る）:**

```typescript
function UserList({ users }: { users: User[] }) {
  // 新しいソート済み配列を作り、元の配列は変更しない
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**React で重要な理由:**

1. props / state の mutation は React の不変性モデルを壊す - React は props と state を読み取り専用として扱うことを前提にしている
2. stale closure のバグを招く - コールバックや effect の中で配列を変更すると、予期しない挙動になることがある

**ブラウザ対応（古い環境向けのフォールバック）:**

`.toSorted()` は主要な最新ブラウザで利用できます（Chrome 110+、Safari 16+、Firefox 115+、Node.js 20+）。古い環境では、スプレッド構文を使ってください:

```typescript
// 古いブラウザ向けのフォールバック
const sorted = [...items].sort((a, b) => a.value - b.value)
```

**他の不変配列メソッド:**

- `.toSorted()` - 不変な sort
- `.toReversed()` - 不変な reverse
- `.toSpliced()` - 不変な splice
- `.with()` - 要素の不変置換
