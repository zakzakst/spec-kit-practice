---
title: 複数の配列走査をまとめる
impact: LOW-MEDIUM
impactDescription: 反復回数を減らす
tags: javascript, arrays, loops, performance
---

## 複数の配列走査をまとめる

複数の `.filter()` や `.map()` は、配列を何度も走査します。1 回のループにまとめてください。

**誤り（3 回の反復）:**

```typescript
const admins = users.filter(u => u.isAdmin)
const testers = users.filter(u => u.isTester)
const inactive = users.filter(u => !u.isActive)
```

**正しい例（1 回の反復）:**

```typescript
const admins: User[] = []
const testers: User[] = []
const inactive: User[] = []

for (const user of users) {
  if (user.isAdmin) admins.push(user)
  if (user.isTester) testers.push(user)
  if (!user.isActive) inactive.push(user)
}
```
