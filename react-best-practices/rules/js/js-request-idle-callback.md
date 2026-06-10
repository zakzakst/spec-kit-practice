---
title: requestIdleCallback で非重要処理を遅延する
impact: MEDIUM
impactDescription: バックグラウンド処理中も UI の応答性を保つ
tags: javascript, performance, idle, scheduling, analytics
---

## requestIdleCallback で非重要処理を遅延する

**影響: MEDIUM（バックグラウンド処理中も UI の応答性を保つ）**

ブラウザがアイドル状態のときに非重要な処理をスケジュールするには、`requestIdleCallback()` を使ってください。これによりメインスレッドをユーザー操作やアニメーションのために空けておけるので、カクつきが減り、体感性能が向上します。

**誤り（ユーザー操作中にメインスレッドを塞ぐ）:**

```typescript
function handleSearch(query: string) {
  const results = searchItems(query)
  setResults(results)

  // これらはすぐにメインスレッドを塞ぐ
  analytics.track('search', { query })
  saveToRecentSearches(query)
  prefetchTopResults(results.slice(0, 3))
}
```

**正しい例（非重要処理をアイドル時間に遅らせる）:**

```typescript
function handleSearch(query: string) {
  const results = searchItems(query)
  setResults(results)

  // 非重要処理はアイドル時間に回す
  requestIdleCallback(() => {
    analytics.track('search', { query })
  })

  requestIdleCallback(() => {
    saveToRecentSearches(query)
  })

  requestIdleCallback(() => {
    prefetchTopResults(results.slice(0, 3))
  })
}
```

**必須処理にタイムアウトを付ける例:**

```typescript
// ブラウザが忙しい状態でも 2 秒以内に analytics を送る
requestIdleCallback(
  () => analytics.track('page_view', { path: location.pathname }),
  { timeout: 2000 }
)
```

**大きな処理を分割する:**

```typescript
function processLargeDataset(items: Item[]) {
  let index = 0

  function processChunk(deadline: IdleDeadline) {
    // アイドル時間がある間だけ処理する（50ms 未満のチャンクを目標にする）
    while (index < items.length && deadline.timeRemaining() > 0) {
      processItem(items[index])
      index++
    }

    // まだ項目が残っていれば次のチャンクを予約する
    if (index < items.length) {
      requestIdleCallback(processChunk)
    }
  }

  requestIdleCallback(processChunk)
}
```

**未対応ブラウザ向けのフォールバック付き:**

```typescript
const scheduleIdleWork = window.requestIdleCallback ?? ((cb: () => void) => setTimeout(cb, 1))

scheduleIdleWork(() => {
  // 非重要処理
})
```

**使いどころ:**

- Analytics とテレメトリ
- localStorage / IndexedDB への状態保存
- 次の操作で使いそうなリソースの先読み
- 緊急でないデータ変換の処理
- 非重要機能の遅延初期化

**使わないほうがよい場面:**

- すぐに反応が必要なユーザー操作
- ユーザーが待っている描画更新
- 時間に敏感な処理
