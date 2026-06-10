---
title: 非同期の後処理に `after()` を使う
impact: MEDIUM
impactDescription: レスポンス時間を短縮する
tags: server, async, logging, analytics, side-effects
---

## 非同期の後処理に `after()` を使う

Next.js の `after()` を使って、レスポンス送信後に実行したい処理を予約します。これにより、ログ記録、分析、その他の副作用がレスポンスをブロックするのを防げます。

**悪い例（レスポンスをブロックする）:**

```tsx
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // 変更処理を実行する
  await updateDatabase(request)
  
  // ログ記録がレスポンスをブロックする
  const userAgent = request.headers.get('user-agent') || 'unknown'
  await logUserAction({ userAgent })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

**良い例（非ブロッキング）:**

```tsx
import { after } from 'next/server'
import { headers, cookies } from 'next/headers'
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // 変更処理を実行する
  await updateDatabase(request)
  
  // レスポンス送信後にログを記録する
  after(async () => {
    const userAgent = (await headers()).get('user-agent') || 'unknown'
    const sessionCookie = (await cookies()).get('session-id')?.value || 'anonymous'
    
    logUserAction({ sessionCookie, userAgent })
  })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

レスポンスはすぐに送信され、ログ記録はバックグラウンドで行われます。

**よくある用途:**

- 分析トラッキング
- 監査ログ
- 通知の送信
- キャッシュ無効化
- 後片付け処理

**重要な注意点:**

- `after()` は、レスポンスが失敗したりリダイレクトしたりしても実行されます
- Server Actions、Route Handlers、Server Components で利用できます

Reference: [https://nextjs.org/docs/app/api-reference/functions/after](https://nextjs.org/docs/app/api-reference/functions/after)
