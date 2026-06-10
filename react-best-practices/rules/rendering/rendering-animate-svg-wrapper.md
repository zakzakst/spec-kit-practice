---
title: SVG 要素ではなくラッパーをアニメーションする
impact: LOW
impactDescription: ハードウェアアクセラレーションを有効にする
tags: rendering, svg, css, animation, performance
---

## SVG 要素ではなくラッパーをアニメーションする

多くのブラウザでは、SVG 要素に対する CSS3 アニメーションはハードウェアアクセラレーションされません。SVG を `<div>` で包み、代わりにラッパーをアニメーションさせます。

**不適切（SVG を直接アニメーションする - ハードウェアアクセラレーションなし）:**

```tsx
function LoadingSpinner() {
  return (
    <svg 
      className="animate-spin"
      width="24" 
      height="24" 
      viewBox="0 0 24 24"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  )
}
```

**適切（ラッパーの div をアニメーションする - ハードウェアアクセラレーションあり）:**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg 
        width="24" 
        height="24" 
        viewBox="0 0 24 24"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  )
}
```

これは、すべての CSS の transform と transition（`transform`, `opacity`, `translate`, `scale`, `rotate`）に当てはまります。ラッパーの div を使うと、ブラウザは GPU アクセラレーションを利用でき、より滑らかにアニメーションできます。
