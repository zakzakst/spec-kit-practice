---
title: 戦略的な Suspense 境界
impact: HIGH
impactDescription: 初期描画を高速化
tags: async, suspense, streaming, layout-shift
---

## 戦略的な Suspense 境界

async コンポーネント内で JSX を返す前にデータを `await` する代わりに、Suspense 境界を使って、データ読み込み中でも外側の UI をより早く表示してください。

**誤り（外側の UI がデータ取得でブロックされる）:**

```tsx
async function Page() {
  const data = await fetchData() // ページ全体をブロックする
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

中央部分だけがデータを必要としているのに、レイアウト全体がそのデータを待ってしまいます。

**正しい例（外側の UI をすぐ表示し、データは後から流し込む）:**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

async function DataDisplay() {
  const data = await fetchData() // このコンポーネントだけをブロックする
  return <div>{data.content}</div>
}
```

Sidebar、Header、Footer はすぐに描画されます。データを待つのは `DataDisplay` だけです。

**代替案（コンポーネント間で Promise を共有する）:**

```tsx
function Page() {
  // 取得はすぐに開始するが、await はしない
  const dataPromise = fetchData()
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  )
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // Promise を展開する
  return <div>{data.content}</div>
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // 同じ Promise を再利用する
  return <div>{data.summary}</div>
}
```

両方のコンポーネントが同じ Promise を共有するため、取得は 1 回だけです。レイアウトはすぐに描画され、両方のコンポーネントが同時に待機します。

**このパターンを使わない方がよい場合:**

- レイアウトの判断に必要な重要データがある場合（配置に影響する）
- ファーストビュー上の SEO 重要コンテンツ
- 小さくて高速なクエリで、Suspense のオーバーヘッドに見合わない場合
- ローディングからコンテンツへの切り替えでレイアウトシフトを避けたい場合

**トレードオフ:** 初期描画の高速化と、レイアウトシフトの可能性のどちらを優先するかです。UX の優先順位に応じて選んでください。
