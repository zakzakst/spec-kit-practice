---
title: Min/Max には sort ではなくループを使う
impact: LOW
impactDescription: O(n log n) ではなく O(n) にする
tags: javascript, arrays, performance, sorting, algorithms
---

## Min/Max には sort ではなくループを使う

最小値や最大値を求めるだけなら、配列を 1 回走査すれば十分です。ソートは無駄が多く、遅くなります。

**誤り（O(n log n) - 最新の値を探すために sort する）:**

```typescript
interface Project {
  id: string
  name: string
  updatedAt: number
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt)
  return sorted[0]
}
```

配列全体をソートしているだけで、最大値を 1 つ見つけたいだけです。

**誤り（O(n log n) - 最古と最新の両方を sort で求める）:**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt)
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] }
}
```

最小値と最大値だけ必要な場面でも、不要にソートしています。

**正しい例（O(n) - 1 回のループ）:**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null
  
  let latest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i]
    }
  }
  
  return latest
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null }
  
  let oldest = projects[0]
  let newest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i]
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i]
  }
  
  return { oldest, newest }
}
```

配列を 1 回走査するだけで、コピーもソートも不要です。

**代替案（小さな配列なら Math.min / Math.max も可）:**

```typescript
const numbers = [5, 2, 8, 1, 9]
const min = Math.min(...numbers)
const max = Math.max(...numbers)
```

これは小さな配列なら使えますが、スプレッド構文の制限により、大きな配列では遅くなったりエラーになったりします。最大配列長は Chrome 143 ではおよそ 124000、Safari 18 では 638000 です。正確な値は環境によって変わります。詳細は [この fiddle](https://jsfiddle.net/qw1jabsx/4/) を参照してください。信頼性を重視するなら、ループ方式を使ってください。
