---
title: 配列比較では先に長さを確認する
impact: MEDIUM-HIGH
impactDescription: 長さが異なるときに高コストな処理を避ける
tags: javascript, arrays, performance, optimization, comparison
---

## 配列比較では先に長さを確認する

ソート、深い比較、シリアライズのような高コストな操作で配列を比較する場合は、先に長さを確認してください。長さが違えば、配列が等しいことはありません。

実際のアプリケーションでは、この最適化はホットパス（イベントハンドラ、レンダリングループ）で特に効果的です。

**誤り（毎回高コストな比較を行う）:**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 長さが違っても、毎回ソートして join する
  return current.sort().join() !== original.sort().join()
}
```

`current.length` が 5 で `original.length` が 100 でも、O(n log n) のソートを 2 回実行しています。さらに配列を join して文字列比較するコストもあります。

**正しい例（先に O(1) の長さチェックをする）:**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 長さが違えばすぐに返す
  if (current.length !== original.length) {
    return true
  }
  // 長さが一致した場合だけソートする
  const currentSorted = current.toSorted()
  const originalSorted = original.toSorted()
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true
    }
  }
  return false
}
```

この新しい方法が効率的なのは次の理由です:
- 長さが違うときに、ソートや join のオーバーヘッドを避けられる
- 連結済み文字列のためのメモリ消費を避けられる（大きな配列では特に重要）
- 元の配列を変更しない
- 差分が見つかった時点で早期に返せる
