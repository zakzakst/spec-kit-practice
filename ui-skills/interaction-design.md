---
name: interaction-design
description: マイクロインタラクション、モーションデザイン、トランジション、ユーザーフィードバックパターンを設計・実装します。UIインタラクションの洗練度を高めたり、読み込み状態を実装したり、快適なユーザーエクスペリエンスを実現したりする際に活用できます。
---

# インタラクションデザイン

モーション、フィードバック、思慮深い状態遷移を通じて魅力的で直感的なインタラクションを作成し、使いやすさを向上させてユーザーを満足させます。

## このスキルを使うタイミング

- ユーザーフィードバックを強化するためのマイクロインタラクションの追加
- スムーズなページとコンポーネントの遷移を実装する
- 読み込み状態とスケルトン画面の設計
- ジェスチャーベースのインタラクションの作成
- 通知とトーストシステムの構築
- ドラッグアンドドロップインターフェースの実装
- スクロールトリガーアニメーションの追加
- ホバー状態とフォーカス状態の設計

## コア原則

### 1. 意図的な動き

動きは装飾ではなくコミュニケーションであるべき:

- **フィードバック**: ユーザーアクションが発生したことを確認する
- **方向**: 要素の出所/行き先を表示する
- **フォーカス**: 重要な変更に直接注意を払う
- **連続**: 遷移中にコンテキストを維持する

### 2. タイミングガイドライン

| Duration  | 使用事例                                         |
| --------- | ------------------------------------------------ |
| 100-150ms | マイクロフィードバック（ホバー、クリック）       |
| 200-300ms | 小さな遷移（トグル、ドロップダウン）             |
| 300-500ms | 中程度のトランジション（モーダル、ページの変更） |
| 500ms+    | 複雑に振り付けられたアニメーション               |

### 3. イージング関数

```css
/* Common easings */
--ease-out: cubic-bezier(0.16, 1, 0.3, 1); /* Decelerate - entering */
--ease-in: cubic-bezier(0.55, 0, 1, 0.45); /* Accelerate - exiting */
--ease-in-out: cubic-bezier(0.65, 0, 0.35, 1); /* Both - moving between */
--spring: cubic-bezier(0.34, 1.56, 0.64, 1); /* Overshoot - playful */
```

## クイックスタート: ボタンのマイクロインタラクション

```tsx
import { motion } from "framer-motion";

export function InteractiveButton({ children, onClick }) {
  return (
    <motion.button
      onClick={onClick}
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.98 }}
      transition={{ type: "spring", stiffness: 400, damping: 17 }}
      className="px-4 py-2 bg-blue-600 text-white rounded-lg"
    >
      {children}
    </motion.button>
  );
}
```

## インタラクションパターン

### 1. 読み込み状態

**スケルトンスクリーン**: 読み込み中にレイアウトを維持する

```tsx
function CardSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-48 bg-gray-200 rounded-lg" />
      <div className="mt-4 h-4 bg-gray-200 rounded w-3/4" />
      <div className="mt-2 h-4 bg-gray-200 rounded w-1/2" />
    </div>
  );
}
```

**進捗インジケーター**: 明確な進捗状況を示す

```tsx
function ProgressBar({ progress }: { progress: number }) {
  return (
    <div className="h-2 bg-gray-200 rounded-full overflow-hidden">
      <motion.div
        className="h-full bg-blue-600"
        initial={{ width: 0 }}
        animate={{ width: `${progress}%` }}
        transition={{ ease: "easeOut" }}
      />
    </div>
  );
}
```

### 2. 状態遷移

**スムーズな切り替え**:

```tsx
function Toggle({ checked, onChange }) {
  return (
    <button
      role="switch"
      aria-checked={checked}
      onClick={() => onChange(!checked)}
      className={`
        relative w-12 h-6 rounded-full transition-colors duration-200
        ${checked ? "bg-blue-600" : "bg-gray-300"}
      `}
    >
      <motion.span
        className="absolute top-1 left-1 w-4 h-4 bg-white rounded-full shadow"
        animate={{ x: checked ? 24 : 0 }}
        transition={{ type: "spring", stiffness: 500, damping: 30 }}
      />
    </button>
  );
}
```

### 3. ページ遷移

**Framer Motionレイアウトアニメーション**:

```tsx
import { AnimatePresence, motion } from "framer-motion";

function PageTransition({ children, key }) {
  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={key}
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.3 }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}
```

### 4. フィードバックパターン

**クリックによる波及効果**:

```tsx
function RippleButton({ children, onClick }) {
  const [ripples, setRipples] = useState([]);

  const handleClick = (e) => {
    const rect = e.currentTarget.getBoundingClientRect();
    const ripple = {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top,
      id: Date.now(),
    };
    setRipples((prev) => [...prev, ripple]);
    setTimeout(() => {
      setRipples((prev) => prev.filter((r) => r.id !== ripple.id));
    }, 600);
    onClick?.(e);
  };

  return (
    <button onClick={handleClick} className="relative overflow-hidden">
      {children}
      {ripples.map((ripple) => (
        <span
          key={ripple.id}
          className="absolute bg-white/30 rounded-full animate-ripple"
          style={{ left: ripple.x, top: ripple.y }}
        />
      ))}
    </button>
  );
}
```

### 5. ジェスチャーインタラクション

**スワイプして閉じる**:

```tsx
function SwipeCard({ children, onDismiss }) {
  return (
    <motion.div
      drag="x"
      dragConstraints={{ left: 0, right: 0 }}
      onDragEnd={(_, info) => {
        if (Math.abs(info.offset.x) > 100) {
          onDismiss();
        }
      }}
      className="cursor-grab active:cursor-grabbing"
    >
      {children}
    </motion.div>
  );
}
```

## CSSアニメーションパターン

### Keyframe Animations

```css
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.animate-fadeIn {
  animation: fadeIn 0.3s ease-out;
}
.animate-pulse {
  animation: pulse 2s ease-in-out infinite;
}
.animate-spin {
  animation: spin 1s linear infinite;
}
```

### CSS Transitions

```css
.card {
  transition:
    transform 0.2s ease-out,
    box-shadow 0.2s ease-out;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
}
```

## アクセシビリティに関する考慮事項

```css
/* ユーザーのモーション設定を尊重する */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

```tsx
function AnimatedComponent() {
  const prefersReducedMotion = window.matchMedia(
    "(prefers-reduced-motion: reduce)",
  ).matches;

  return (
    <motion.div
      animate={{ opacity: 1 }}
      transition={{ duration: prefersReducedMotion ? 0 : 0.3 }}
    />
  );
}
```

## ベストプラクティス

1. **パフォーマンス第一**: スムーズな60fpsを実現するには、`transform`と`opacity`を使用します。
2. **モーションサポートの削減**: 常に`prefers-reduced-motion`を尊重する
3. **一貫したタイミング**: アプリ全体でタイミングスケールを使用する
4. **自然物理学**: 線形アニメーションよりもスプリングアニメーションを好む
5. **中断可能**: 長いアニメーションをユーザーがキャンセルできるようにする
6. **プログレッシブエンハンスメント**: JSアニメーションなしで動作する
7. **デバイスでのテスト**: パフォーマンスは大きく異なる

## よくある問題

- **ぎこちないアニメーション**: `width`、`height`、`top`、`left` のアニメーションを避ける
- **オーバーアニメーション**: 動きすぎると疲労を引き起こす
- **ブロッキングインタラクション**: アニメーション中にユーザー入力を妨げない
- **メモリリーク**: アンマウント時にアニメーションリスナーをクリーンアップする
- **コンテンツのフラッシュ**: 最適化のために `will-change` を控えめに使用してください

## リソース

- [Framer Motion Documentation](https://www.framer.com/motion/)
- [CSS Animation Guide](https://web.dev/animations-guide/)
- [Material Design Motion](https://m3.material.io/styles/motion/overview)
- [GSAP Animation Library](https://greensock.com/gsap/)
