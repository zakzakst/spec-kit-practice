---
title: ちらつきなしで hydration mismatch を防ぐ
impact: MEDIUM
impactDescription: 見た目のちらつきと hydration エラーを防ぐ
tags: rendering, ssr, hydration, localStorage, flicker
---

## ちらつきなしで hydration mismatch を防ぐ

`localStorage` や cookie などのクライアント側ストレージに依存する内容を描画する場合は、React が hydrate する前に DOM を更新する同期スクリプトを注入することで、SSR の破綻と hydration 後のちらつきの両方を避けます。

**不適切（SSR が壊れる）:**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage はサーバーでは使えないため、エラーになる
  const theme = localStorage.getItem('theme') || 'light'
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}
```

サーバーサイドレンダリングは `localStorage` が undefined なので失敗します。

**不適切（見た目がちらつく）:**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState('light')
  
  useEffect(() => {
    // hydration 後に実行されるため、見えるちらつきが発生する
    const stored = localStorage.getItem('theme')
    if (stored) {
      setTheme(stored)
    }
  }, [])
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}
```

コンポーネントは最初にデフォルト値（`light`）で描画され、その後 hydration 後に更新されるため、誤った内容が一瞬表示されます。

**適切（ちらつきなし、hydration mismatch なし）:**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">
        {children}
      </div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  )
}
```

このインラインスクリプトは要素が表示される前に同期的に実行されるため、DOM にはすでに正しい値が入っています。ちらつきも hydration mismatch も発生しません。

このパターンは、テーマ切り替え、ユーザー設定、認証状態、そしてデフォルト値をちらつかせずに即座に描画したいクライアント専用データに特に有効です。
