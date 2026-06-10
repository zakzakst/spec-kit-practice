---
title: バレルファイルのインポートを避ける
impact: CRITICAL
impactDescription: 200〜800msのインポートコスト、ビルドが遅くなる
tags: bundle, imports, tree-shaking, barrel-files, performance
---

## バレルファイルのインポートを避ける

バレルファイルではなくソースファイルから直接インポートして、使っていない何千ものモジュールを読み込まないようにします。**バレルファイル**とは、複数のモジュールを再エクスポートするエントリポイントのことです（例: `index.js` で `export * from './module'` を行うもの）。

人気のあるアイコンライブラリやコンポーネントライブラリでは、エントリファイルに**最大10,000件の再エクスポート**が含まれることがあります。多くの React パッケージでは、**インポートするだけで200〜800ms**かかり、開発速度と本番のコールドスタートの両方に影響します。

**なぜ tree-shaking では改善しないのか:** ライブラリが external（バンドル対象外）として扱われている場合、バンドラは最適化できません。tree-shaking を有効にするためにバンドルすると、今度はモジュールグラフ全体の解析でビルドが大幅に遅くなります。

**誤り（ライブラリ全体を読み込む）:**

```tsx
import { Check, X, Menu } from 'lucide-react'
// 1,583個のモジュールを読み込むため、開発時に約2.8秒余計にかかる
// 実行時コスト: コールドスタートごとに200〜800ms

import { Button, TextField } from '@mui/material'
// 2,225個のモジュールを読み込むため、開発時に約4.2秒余計にかかる
```

**正しい例 - Next.js 13.5+（推奨）:**

```js
// next.config.js - ビルド時にバレルインポートを自動で最適化する
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}
```

```tsx
// 標準のインポートをそのまま使う - Next.js が直接インポートに変換する
import { Check, X, Menu } from 'lucide-react'
// TypeScript を完全にサポートし、パスを手作業で調整する必要がない
```

この方法は、バレルインポートのコストを解消しつつ、TypeScript の型安全性とエディタの補完を維持できるため推奨されます。

**正しい例 - 直接インポート（Next.js 以外のプロジェクト）:**

```tsx
import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// 使用するものだけを読み込む
```

> **TypeScript の注意:** 一部のライブラリ（特に `lucide-react`）は、深いインポートパス向けの `.d.ts` ファイルを提供していません。`lucide-react/dist/esm/icons/check` からインポートすると暗黙の `any` 型になり、`strict` や `noImplicitAny` でエラーになります。利用可能なら `optimizePackageImports` を優先し、直接インポートを使う前にサブパスで型が提供されているか確認してください。

これらの最適化により、開発起動が15〜70%高速化し、ビルドが28%高速化し、コールドスタートが40%高速化し、HMR も大幅に速くなります。

影響を受けやすいライブラリ: `lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`。

参考: [Next.js のパッケージインポート最適化の取り組み](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
