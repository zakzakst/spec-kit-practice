---
title: 関数は早期リターンする
impact: LOW-MEDIUM
impactDescription: 不要な計算を避ける
tags: javascript, functions, optimization, early-return
---

## 関数は早期リターンする

結果が確定したらすぐに返して、不要な処理をスキップしてください。

**誤り（答えが見つかっても全件を処理し続ける）:**

```typescript
function validateUsers(users: User[]) {
  let hasError = false
  let errorMessage = ''
  
  for (const user of users) {
    if (!user.email) {
      hasError = true
      errorMessage = 'Email required'
    }
    if (!user.name) {
      hasError = true
      errorMessage = 'Name required'
    }
    // エラーを見つけても全ユーザーの確認を続ける
  }
  
  return hasError ? { valid: false, error: errorMessage } : { valid: true }
}
```

**正しい例（最初のエラーで即座に返す）:**

```typescript
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: 'Email required' }
    }
    if (!user.name) {
      return { valid: false, error: 'Name required' }
    }
  }

  return { valid: true }
}
```
