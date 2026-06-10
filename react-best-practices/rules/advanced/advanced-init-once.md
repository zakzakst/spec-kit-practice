---
title: マウントごとではなく、アプリ起動時に 1 回だけ初期化する
impact: LOW-MEDIUM
impactDescription: 開発環境での重複初期化を防ぐ
tags: initialization, useEffect, app-startup, side-effects
---

## マウントごとではなく、アプリ起動時に 1 回だけ初期化する

アプリ全体で 1 回だけ実行されるべき初期化処理を、コンポーネントの `useEffect([])` の中に置かないでください。コンポーネントは再マウントされることがあり、effect も再実行されます。代わりに、モジュールレベルのガードか、エントリーモジュールでのトップレベル初期化を使ってください。

**誤り: 開発環境では 2 回実行され、再マウント時にも再実行される**

```tsx
function Comp() {
  useEffect(() => {
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}
```

**正しい例: アプリの読み込みごとに 1 回だけ実行する**

```tsx
let didInit = false

function Comp() {
  useEffect(() => {
    if (didInit) return
    didInit = true
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}
```

参考: [Initializing the application](https://react.dev/learn/you-might-not-need-an-effect#initializing-the-application)
