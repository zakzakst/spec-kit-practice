---
title: リクエストデータに共有モジュール状態を使わない
impact: HIGH
impactDescription: 並行処理のバグとリクエストデータ漏えいを防ぐ
tags: server, rsc, ssr, concurrency, security, state
---

## リクエストデータに共有モジュール状態を使わない

React Server Components と、SSR される client component では、リクエストスコープのデータを共有するために変更可能なモジュールレベル変数を使わないでください。サーバー描画は同じプロセス内で並行実行されることがあります。1 つの描画が共有モジュール状態に書き込み、別の描画がそれを読み取ると、競合状態、リクエスト間の汚染、そして 1 人のユーザーのデータが別のユーザーのレスポンスに現れるようなセキュリティバグが起こり得ます。

サーバー上のモジュールスコープは、リクエストごとのローカル状態ではなく、プロセス全体で共有されるメモリとして扱ってください。

**悪い例（並行描画でリクエストデータが漏れる）:**

```tsx
let currentUser: User | null = null

export default async function Page() {
  currentUser = await auth()
  return <Dashboard />
}

async function Dashboard() {
  return <div>{currentUser?.name}</div>
}
```

2 つのリクエストが重なると、リクエスト A が `currentUser` を設定し、その直後にリクエスト B が上書きして、リクエスト A が `Dashboard` の描画を終える前に値が変わってしまうことがあります。

**良い例（リクエストデータは描画ツリー内に閉じ込める）:**

```tsx
export default async function Page() {
  const user = await auth()
  return <Dashboard user={user} />
}

function Dashboard({ user }: { user: User | null }) {
  return <div>{user?.name}</div>
}
```

安全な例外:

- モジュールスコープで 1 回だけ読み込む不変の静的アセットや設定
- リクエスト間の再利用を意図し、かつ適切にキー付けされた共有キャッシュ
- リクエストやユーザー固有の可変データを保持しないプロセス全体のシングルトン

静的アセットや設定については、[静的 I/O をモジュールレベルへ移動する](./server-hoist-static-io.md) を参照してください。
