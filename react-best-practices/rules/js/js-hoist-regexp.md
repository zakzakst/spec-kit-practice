---
title: RegExp の生成を外に出す
impact: LOW-MEDIUM
impactDescription: 再生成を避ける
tags: javascript, regexp, optimization, memoization
---

## RegExp の生成を外に出す

`render` の中で `RegExp` を生成しないでください。モジュールスコープに外出しするか、`useMemo()` でメモ化してください。

**誤り（毎回新しい RegExp を作る）:**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**正しい例（メモ化するか外出しする）:**

```tsx
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function Highlighter({ text, query }: Props) {
  const regex = useMemo(
    () => new RegExp(`(${escapeRegex(query)})`, 'gi'),
    [query]
  )
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**注意（グローバル正規表現は可変状態を持つ）:**

グローバル正規表現 (`/g`) は `lastIndex` という可変状態を持ちます:

```typescript
const regex = /foo/g
regex.test('foo')  // true, lastIndex = 3
regex.test('foo')  // false, lastIndex = 0
```
