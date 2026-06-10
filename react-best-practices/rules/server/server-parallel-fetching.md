---
title: コンポーネント合成でデータ取得を並列化する
impact: CRITICAL
impactDescription: サーバー側のウォーターフォールをなくす
tags: server, rsc, parallel-fetching, composition
---

## コンポーネント合成でデータ取得を並列化する

React Server Components は、ツリー内では順番に実行されます。コンポジションを組み替えて、データ取得を並列化してください。

**悪い例（Sidebar は Page の fetch 完了を待つ）:**

```tsx
export default async function Page() {
  const header = await fetchHeader()
  return (
    <div>
      <div>{header}</div>
      <Sidebar />
    </div>
  )
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}
```

**良い例（両方を同時に fetch する）:**

```tsx
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  )
}
```

**children prop を使う代替例:**

```tsx
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

function Layout({ children }: { children: ReactNode }) {
  return (
    <div>
      <Header />
      {children}
    </div>
  )
}

export default function Page() {
  return (
    <Layout>
      <Sidebar />
    </Layout>
  )
}
```
