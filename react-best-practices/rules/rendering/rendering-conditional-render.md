---
title: 明示的な条件付きレンダリングを使う
impact: LOW
impactDescription: 0 や NaN の描画を防ぐ
tags: rendering, conditional, jsx, falsy-values
---

## 明示的な条件付きレンダリングを使う

条件が `0`、`NaN`、または描画されてしまう他の falsy 値になり得る場合は、条件付きレンダリングに `&&` ではなく明示的な三項演算子（`? :`）を使います。

**不適切（count が 0 のときに "0" が描画される）:**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count && <span className="badge">{count}</span>}
    </div>
  )
}

// count = 0 のとき: <div>0</div> が描画される
// count = 5 のとき: <div><span class="badge">5</span></div> が描画される
```

**適切（count が 0 のときは何も描画しない）:**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count > 0 ? <span className="badge">{count}</span> : null}
    </div>
  )
}

// count = 0 のとき: <div></div> が描画される
// count = 5 のとき: <div><span class="badge">5</span></div> が描画される
```
