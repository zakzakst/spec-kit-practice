---
title: SVG の精度を最適化する
impact: LOW
impactDescription: ファイルサイズを削減する
tags: rendering, svg, optimization, svgo
---

## SVG の精度を最適化する

SVG の座標精度を下げてファイルサイズを小さくします。最適な精度は viewBox のサイズに依存しますが、一般には精度を下げることを検討すべきです。

**不適切（精度が高すぎる）:**

```svg
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />
```

**適切（小数 1 桁）:**

```svg
<path d="M 10.3 20.8 L 30.9 40.2" />
```

**SVGO で自動化する:**

```bash
npx svgo --precision=1 --multipass icon.svg
```
