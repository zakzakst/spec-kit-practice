---
title: 長いリストには CSS の content-visibility を使う
impact: HIGH
impactDescription: 初期描画を高速化する
tags: rendering, css, content-visibility, long-lists
---

## 長いリストには CSS の content-visibility を使う

画面外のレンダリングを遅延させるために `content-visibility: auto` を適用します。

**CSS:**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}
```

**例:**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}
```

1000 件のメッセージがある場合、ブラウザは画面外の約 990 項目について layout/paint をスキップします（初期描画が約 10 倍高速化）。
