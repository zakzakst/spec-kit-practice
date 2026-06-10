---
title: ユーザー意図に基づいてプリロードする
impact: MEDIUM
impactDescription: 体感レイテンシを下げる
tags: bundle, preload, user-intent, hover
---

## ユーザー意図に基づいてプリロードする

重いバンドルは、必要になる前にプリロードして体感レイテンシを下げます。

**例（ホバー/フォーカス時にプリロード）:**

```tsx
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== 'undefined') {
      void import('./monaco-editor')
    }
  }

  return (
    <button
      onMouseEnter={preload}
      onFocus={preload}
      onClick={onClick}
    >
      エディタを開く
    </button>
  )
}
```

**例（機能フラグが有効なときにプリロード）:**

```tsx
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== 'undefined') {
      void import('./monaco-editor').then(mod => mod.init())
    }
  }, [flags.editorEnabled])

  return <FlagsContext.Provider value={flags}>
    {children}
  </FlagsContext.Provider>
}
```

`typeof window !== 'undefined'` のチェックにより、プリロード対象のモジュールが SSR 向けにバンドルされるのを防ぎ、サーバーバンドルサイズとビルド速度を最適化できます。
