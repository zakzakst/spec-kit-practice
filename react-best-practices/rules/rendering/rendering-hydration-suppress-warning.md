---
title: 予期される hydration mismatch を抑制する
impact: LOW-MEDIUM
impactDescription: 既知の差分に対する不要な hydration 警告を防ぐ
tags: rendering, hydration, ssr, nextjs
---

## 予期される hydration mismatch を抑制する

SSR フレームワーク（たとえば Next.js）では、サーバーとクライアントで意図的に異なる値（ランダム ID、日付、ロケール/タイムゾーンのフォーマットなど）があります。こうした *予期された* mismatch では、動的なテキストを `suppressHydrationWarning` 付きの要素で囲み、不要な警告を防ぎます。実際のバグを隠すためには使わないでください。乱用もしないでください。

**不適切（既知の mismatch 警告が出る）:**

```tsx
function Timestamp() {
  return <span>{new Date().toLocaleString()}</span>
}
```

**適切（予期された mismatch のみを抑制する）:**

```tsx
function Timestamp() {
  return (
    <span suppressHydrationWarning>
      {new Date().toLocaleString()}
    </span>
  )
}
```
