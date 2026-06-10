---
title: レイアウトスラッシングを避ける
impact: MEDIUM
impactDescription: 強制同期レイアウトを防ぎ、性能上のボトルネックを減らす
tags: javascript, dom, css, performance, reflow, layout-thrashing
---

## レイアウトスラッシングを避ける

スタイルの書き込みとレイアウトの読み取りを交互に行わないでください。`offsetWidth`、`getBoundingClientRect()`、`getComputedStyle()` などのレイアウトプロパティをスタイル変更の合間に読むと、ブラウザは強制的に同期再計算を実行させられます。

**これは問題ありません（ブラウザがスタイル変更をまとめて処理するため）:**
```typescript
function updateElementStyles(element: HTMLElement) {
  // 各行でスタイルは無効化されるが、ブラウザが再計算をまとめる
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
}
```

**誤り（読み取りと書き込みが交互だと再計算が強制される）:**
```typescript
function layoutThrashing(element: HTMLElement) {
  element.style.width = '100px'
  const width = element.offsetWidth  // 再計算を強制する
  element.style.height = '200px'
  const height = element.offsetHeight  // さらに再計算を強制する
}
```

**正しい例（書き込みをまとめ、その後に 1 回だけ読む）:**
```typescript
function updateElementStyles(element: HTMLElement) {
  // すべての書き込みをまとめる
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
  
  // すべての書き込みが終わってから読む（再計算は 1 回）
  const { width, height } = element.getBoundingClientRect()
}
```

**正しい例（読み取りをまとめてから書き込む）:**
```typescript
function avoidThrashing(element: HTMLElement) {
  // 読み取りフェーズ - まずレイアウト問い合わせをすべて行う
  const rect1 = element.getBoundingClientRect()
  const offsetWidth = element.offsetWidth
  const offsetHeight = element.offsetHeight
  
  // 書き込みフェーズ - その後にすべてのスタイル変更を行う
  element.style.width = '100px'
  element.style.height = '200px'
}
```

**より良い方法: CSS クラスを使う**
```css
.highlighted-box {
  width: 100px;
  height: 200px;
  background-color: blue;
  border: 1px solid black;
}
```
```typescript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')
  
  const { width, height } = element.getBoundingClientRect()
}
```

**React の例:**
```tsx
// 誤り: スタイル変更とレイアウト問い合わせが交互になっている
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  const ref = useRef<HTMLDivElement>(null)
  
  useEffect(() => {
    if (ref.current && isHighlighted) {
      ref.current.style.width = '100px'
      const width = ref.current.offsetWidth // レイアウトを強制する
      ref.current.style.height = '200px'
    }
  }, [isHighlighted])
  
  return <div ref={ref}>Content</div>
}

// 正しい例: クラスを切り替える
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  return (
    <div className={isHighlighted ? 'highlighted-box' : ''}>
      Content
    </div>
  )
}
```

可能であれば、インラインスタイルよりも CSS クラスを使ってください。CSS ファイルはブラウザにキャッシュされ、クラスのほうが責務分離もしやすく、保守も簡単です。

レイアウトを強制する操作の詳細は、[この gist](https://gist.github.com/paulirish/5d52fb081b3570c81e3a) と [CSS Triggers](https://csstriggers.com/) を参照してください。
