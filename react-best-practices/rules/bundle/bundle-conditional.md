---
title: 条件付きモジュール読み込み
impact: HIGH
impactDescription: 大きなデータは必要なときだけ読み込む
tags: bundle, conditional-loading, lazy-loading
---

## 条件付きモジュール読み込み

機能が有効になったときだけ、大きなデータやモジュールを読み込みます。

**例（アニメーションフレームを遅延読み込み）:**

```tsx
function AnimationPlayer({ enabled, setEnabled }: { enabled: boolean; setEnabled: React.Dispatch<React.SetStateAction<boolean>> }) {
  const [frames, setFrames] = useState<Frame[] | null>(null)

  useEffect(() => {
    if (enabled && !frames && typeof window !== 'undefined') {
      import('./animation-frames.js')
        .then(mod => setFrames(mod.frames))
        .catch(() => setEnabled(false))
    }
  }, [enabled, frames, setEnabled])

  if (!frames) return <Skeleton />
  return <Canvas frames={frames} />
}
```

`typeof window !== 'undefined'` のチェックにより、このモジュールが SSR 向けにバンドルされるのを防ぎ、サーバーバンドルサイズとビルド速度を最適化できます。
