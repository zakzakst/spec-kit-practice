---
title: 静的 I/O をモジュールレベルへ移動する
impact: HIGH
impactDescription: リクエストごとのファイル / ネットワーク I/O を防ぐ
tags: server, io, performance, next.js, route-handlers, og-image
---

## 静的 I/O をモジュールレベルへ移動する

**Impact: HIGH (リクエストごとのファイル / ネットワーク I/O を防ぐ)**

ルートハンドラやサーバー関数でフォント、ロゴ、画像、設定ファイルなどの静的アセットを読み込む場合は、その I/O をモジュールレベルに持ち上げてください。モジュールレベルのコードは、モジュールが最初に import されたときに 1 回だけ実行され、毎リクエストでは実行されません。これにより、毎回の呼び出しで発生していた不要なファイル読み込みやネットワーク取得をなくせます。

**悪い例（毎リクエストでフォントファイルを読む）:**

```typescript
// app/api/og/route.tsx
import { ImageResponse } from 'next/og'

export async function GET(request: Request) {
  // 毎回のリクエストで実行される。高コスト
  const fontData = await fetch(
    new URL('./fonts/Inter.ttf', import.meta.url)
  ).then(res => res.arrayBuffer())

  const logoData = await fetch(
    new URL('./images/logo.png', import.meta.url)
  ).then(res => res.arrayBuffer())

  return new ImageResponse(
    <div style={{ fontFamily: 'Inter' }}>
      <img src={logoData} />
      Hello World
    </div>,
    { fonts: [{ name: 'Inter', data: fontData }] }
  )
}
```

**良い例（モジュール初期化時に 1 回だけ読み込む）:**

```typescript
// app/api/og/route.tsx
import { ImageResponse } from 'next/og'

// モジュールレベル: モジュールが最初に import されたときに 1 回だけ実行される
const fontData = fetch(
  new URL('./fonts/Inter.ttf', import.meta.url)
).then(res => res.arrayBuffer())

const logoData = fetch(
  new URL('./images/logo.png', import.meta.url)
).then(res => res.arrayBuffer())

export async function GET(request: Request) {
  // すでに開始済みの Promise を await する
  const [font, logo] = await Promise.all([fontData, logoData])

  return new ImageResponse(
    <div style={{ fontFamily: 'Inter' }}>
      <img src={logo} />
      Hello World
    </div>,
    { fonts: [{ name: 'Inter', data: font }] }
  )
}
```

**良い例（モジュールレベルの同期 fs 読み込み）:**

```typescript
// app/api/og/route.tsx
import { ImageResponse } from 'next/og'
import { readFileSync } from 'fs'
import { join } from 'path'

// モジュールレベルで同期読み込みする。ブロックされるのはモジュール初期化時のみ
const fontData = readFileSync(
  join(process.cwd(), 'public/fonts/Inter.ttf')
)

const logoData = readFileSync(
  join(process.cwd(), 'public/images/logo.png')
)

export async function GET(request: Request) {
  return new ImageResponse(
    <div style={{ fontFamily: 'Inter' }}>
      <img src={logoData} />
      Hello World
    </div>,
    { fonts: [{ name: 'Inter', data: fontData }] }
  )
}
```

**悪い例（設定を毎回読む）:**

```typescript
import fs from 'node:fs/promises'

export async function processRequest(data: Data) {
  const config = JSON.parse(
    await fs.readFile('./config.json', 'utf-8')
  )
  const template = await fs.readFile('./template.html', 'utf-8')

  return render(template, data, config)
}
```

**良い例（設定とテンプレートをモジュールレベルに持ち上げる）:**

```typescript
import fs from 'node:fs/promises'

const configPromise = fs
  .readFile('./config.json', 'utf-8')
  .then(JSON.parse)
const templatePromise = fs.readFile('./template.html', 'utf-8')

export async function processRequest(data: Data) {
  const [config, template] = await Promise.all([
    configPromise,
    templatePromise,
  ])

  return render(template, data, config)
}
```

このパターンを使う場面:

- OG 画像生成用のフォント読み込み
- 静的なロゴ、アイコン、透かしの読み込み
- 実行時に変化しない設定ファイルの読み込み
- メールテンプレートなどの静的テンプレートの読み込み
- すべてのリクエストで同じ静的アセット

このパターンを使わない場面:

- リクエストやユーザーごとに変わるアセット
- 実行中に変更される可能性があるファイル（その場合は TTL 付きキャッシュを使う）
- メモリに保持すると大きすぎるファイル
- メモリ上に残したくない機密データ

Vercel の [Fluid Compute](https://vercel.com/docs/fluid-compute) では、複数の同時リクエストが同じ関数インスタンスを共有するため、モジュールレベルのキャッシュは特に効果的です。静的アセットは、コールドスタートのペナルティなしでリクエスト間にわたってメモリに保持されます。

従来のサーバーレスでは、各コールドスタートでモジュールレベルのコードが再実行されますが、ウォームな呼び出しではインスタンスが再利用されるまで読み込んだアセットが使われ続けます。
