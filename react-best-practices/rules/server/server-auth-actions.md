---
title: API ルートと同様に Server Action を認証する
impact: CRITICAL
impactDescription: サーバー側の変更処理への不正アクセスを防ぐ
tags: server, server-actions, authentication, security, authorization
---

## API ルートと同様に Server Action を認証する

**Impact: CRITICAL (サーバー側の変更処理への不正アクセスを防ぐ)**

`"use server"` を含む Server Actions は、API ルートと同様に公開エンドポイントとして扱われます。認証と認可は必ず各 Server Action の**内部**で検証してください。middleware、レイアウトのガード、ページレベルのチェックだけに頼ってはいけません。Server Actions は直接呼び出せるためです。

Next.js のドキュメントでも、Server Actions は公開 API エンドポイントと同じセキュリティ上の注意を払うべきであり、ユーザーがその変更を実行できるかを確認するよう明記されています。

**悪い例（認証チェックがない）:**

```typescript
'use server'

export async function deleteUser(userId: string) {
  // だれでも呼び出せる。認証チェックがない
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}
```

**良い例（アクション内部で認証する）:**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { unauthorized } from '@/lib/errors'

export async function deleteUser(userId: string) {
  // 必ずアクションの内部で認証を確認する
  const session = await verifySession()
  
  if (!session) {
    throw unauthorized('ログインが必要です')
  }
  
  // 認可も確認する
  if (session.user.role !== 'admin' && session.user.id !== userId) {
    throw unauthorized('他のユーザーは削除できません')
  }
  
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}
```

**入力検証つき:**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { z } from 'zod'

const updateProfileSchema = z.object({
  userId: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email()
})

export async function updateProfile(data: unknown) {
  // 先に入力を検証する
  const validated = updateProfileSchema.parse(data)
  
  // その後で認証する
  const session = await verifySession()
  if (!session) {
    throw new Error('認証されていません')
  }
  
  // 次に認可する
  if (session.user.id !== validated.userId) {
    throw new Error('自分のプロフィールのみ更新できます')
  }
  
  // 最後に変更処理を実行する
  await db.user.update({
    where: { id: validated.userId },
    data: {
      name: validated.name,
      email: validated.email
    }
  })
  
  return { success: true }
}
```

Reference: [https://nextjs.org/docs/app/guides/authentication](https://nextjs.org/docs/app/guides/authentication)
