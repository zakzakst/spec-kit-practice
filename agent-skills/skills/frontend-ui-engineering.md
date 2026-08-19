---
name: frontend-ui-engineering
description: 本番品質で、アクセシブルで、レスポンシブなユーザー向け UI を構築します。インターフェースやページを作る、コンポーネントを作る、レイアウトを実装する、WCAG のアクセシビリティ要件を満たす、state を扱う、あるいは出力に本番品質の見た目と手触りが求められるときに使います。
---

# フロントエンド UI エンジニアリング

## 概要

アクセシブルで、パフォーマンスが高く、見た目も整った、本番品質のユーザーインターフェースを作ります。目標は、優れたデザインを理解するエンジニアが作ったように見える UI であって、AI が生成したように見える UI ではありません。つまり、本物の design system に従い、適切なアクセシビリティを確保し、意図のある interaction pattern を使い、ありがちな "AI aesthetic" を避けます。

## 使う場面

- 新しい UI component や page を作るとき
- 既存のユーザー向け interface を変更するとき
- レスポンシブ layout を実装するとき
- interactivity や state management を追加するとき
- 見た目や UX の問題を直すとき

## Component Architecture

### File Structure

component に関係するものは、近くにまとめます。

```text
src/components/
  TaskList/
    TaskList.tsx          # Component implementation
    TaskList.test.tsx     # Tests
    TaskList.stories.tsx  # Storybook stories (使っている場合)
    use-task-list.ts      # Custom hook (state が複雑な場合)
    types.ts              # Component-specific types (必要なら)
```

### Component Patterns

**configuration より composition を優先する:**

```tsx
// Good: Composable
<Card>
  <CardHeader>
    <CardTitle>Tasks</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// Avoid: 設定が多すぎる
<Card
  title="Tasks"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**component は焦点を絞る:**

```tsx
// Good: 1 つのことだけをする
export function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li className="flex items-center gap-3 p-3">
      <Checkbox checked={task.done} onChange={() => onToggle(task.id)} />
      <span className={task.done ? 'line-through text-muted' : ''}>{task.title}</span>
      <Button variant="ghost" size="sm" onClick={() => onDelete(task.id)}>
        <TrashIcon />
      </Button>
    </li>
  );
}
```

**data fetching と presentation を分ける:**

```tsx
// Container: data を扱う
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="Failed to load tasks" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="No tasks yet" />;

  return <TaskList tasks={tasks} />;
}

// Presentation: rendering を扱う
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## State Management

**いちばん単純でうまくいく方法を選ぶ:**

```text
Local state (useState)           -> コンポーネント固有の UI state
Lifted state                     -> 2〜3 個の sibling component で共有する state
Context                          -> Theme, auth, locale (読むことが多く、書くことは少ない)
URL state (searchParams)         -> Filter, pagination, 共有可能な UI state
Server state (React Query, SWR)  -> キャッシュ付きの remote data
Global store (Zustand, Redux)    -> app 全体で共有する複雑な client state
```

**3 層を超える prop drilling は避ける。** 使っていない component を通して props を渡しているなら、context を導入するか component tree を組み替えます。

## Design System Adherence

### AI Aesthetic を避ける

AI 生成 UI には、見分けやすいパターンがあります。以下は避けてください。

| AI Default | 問題点 | Production Quality |
|---|---|---|
| 何でも紫 / インディゴ | モデルは visually "safe" な palette に寄りがちで、どの app も同じに見える | プロジェクト本来の color palette を使う |
| 過剰な gradient | 視覚ノイズを増やし、多くの design system とぶつかる | design system に合う flat か控えめな gradient にする |
| 全部丸い（rounded-2xl） | "friendly" に見えるが、実際の design で重要な corner radius の hierarchy を無視する | design system の border-radius に合わせる |
| ありがちな hero section | 実際の content や user need とつながりのない template 的 layout | content-first layout にする |
| Lorem ipsum 風の文言 | placeholder text は、実コンテンツで初めて見える length、折り返し、overflow の問題を隠す | 現実的な placeholder content を使う |
| どこもかしこも過剰な padding | 余白を均等に取りすぎると hierarchy が壊れ、画面を無駄に使う | 一貫した spacing scale を使う |
| 量産型 card grid | 情報の優先度やスキャンパターンを無視したレイアウトの近道 | 目的に沿った layout を使う |
| shadow-heavy design | 深い shadow は content と競合し、低スペック端末では rendering も重くなる | design system で指定がない限り、控えめか shadow なし |

### Spacing and Layout

spacing scale は一貫して使い、勝手に値を足さないでください。

```css
/* scale を使う: 0.25rem 単位（または project の規約に従う） */
/* Good */  padding: 1rem;      /* 16px */
/* Good */  gap: 0.75rem;       /* 12px */
/* Bad */   padding: 13px;      /* scale にない */
/* Bad */   margin-top: 2.3rem; /* scale にない */
```

### Typography

type hierarchy を守ります。

```text
h1    -> Page title（ページに 1 つ）
h2    -> Section title
h3    -> Subsection title
body  -> Default text
small -> Secondary/helper text
```

heading level を飛ばしてはいけません。heading style を見出し以外に使ってはいけません。

### Color

- semantic color token を使う: `text-primary`, `bg-surface`, `border-default` - 生の hex 値ではない
- 十分な contrast を確保する（通常 text は 4.5:1、large text は 3:1）
- 色だけで情報を伝えない（icon、text、pattern も使う）

## Accessibility（WCAG 2.1 AA）

すべての component は次を満たさなければなりません。

### Keyboard Navigation

```tsx
// すべての interactive element は keyboard accessible であること
<button onClick={handleClick}>Click me</button>        // 笨・デフォルトで focusable
<div onClick={handleClick}>Click me</div>               // 笨・focusable ではない
<div role="button" tabIndex={0} onClick={handleClick}    // 笨・ただし <button> を優先
     onKeyDown={e => {
       if (e.key === 'Enter') handleClick();
       if (e.key === ' ') e.preventDefault();
     }}
     onKeyUp={e => {
       if (e.key === ' ') handleClick();
     }}>
  Click me
</div>
```

### ARIA Labels

```tsx
// visible text がない interactive element には label を付ける
<button aria-label="Close dialog"><XIcon /></button>

// form input には label を付ける
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// visible label がないなら aria-label を使う
<input aria-label="Search tasks" type="search" />
```

### Focus Management

```tsx
// content が変わったら focus を移す
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // open 中は dialog 内に focus を閉じ込める
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>Close</button>
      {/* dialog content */}
    </dialog>
  );
}
```

### Meaningful Empty and Error States

```tsx
// 空画面を出さない
function TaskList({ tasks }: { tasks: Task[] }) {
  if (tasks.length === 0) {
    return (
      <div role="status" className="text-center py-12">
        <TasksEmptyIcon className="mx-auto h-12 w-12 text-muted" />
        <h3 className="mt-2 text-sm font-medium">No tasks</h3>
        <p className="mt-1 text-sm text-muted">Get started by creating a new task.</p>
        <Button className="mt-4" onClick={onCreateTask}>Create Task</Button>
      </div>
    );
  }

  return <ul role="list">...</ul>;
}
```

## Responsive Design

mobile first で作り、そこから広げます。

```tsx
// Tailwind: mobile-first responsive
<div className="
  grid grid-cols-1      /* Mobile: single column */
  sm:grid-cols-2        /* Small: 2 columns */
  lg:grid-cols-3        /* Large: 3 columns */
  gap-4
">
```

次の breakpoint でテストします: 320px、768px、1024px、1440px。

## Loading and Transitions

```tsx
// Skeleton loading（content に spinner を使わない）
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="Loading tasks">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// perception speed を上げる optimistic update
function useToggleTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: toggleTask,
    onMutate: async (taskId) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previous = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[]) =>
        old.map(t => t.id === taskId ? { ...t, done: !t.done } : t)
      );

      return { previous };
    },
    onError: (_err, _taskId, context) => {
      queryClient.setQueryData(['tasks'], context?.previous);
    },
  });
}
```

## See Also

詳細な accessibility 要件と testing tool は `../../references/accessibility-checklist.md` を参照してください。

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「Accessibility はあればいい程度」 | 多くの法域で法的要件であり、エンジニアリング品質基準でもあります。 |
| 「レスポンシブは後でやる」 | 後付けの responsive design は、最初から作るより 3 倍難しいです。 |
| 「デザインが確定していないから styling は飛ばす」 | design system の default を使ってください。unstyled UI はレビューで悪い第一印象になります。 |
| 「これはただの prototype」 | prototype は production code になります。土台を正しく作ってください。 |
| 「AI aesthetic でも今は十分」 | 低品質のサインです。最初から本来の design system を使ってください。 |

## レッドフラグ

- 200 行を超える component（分割する）
- inline style や任意の pixel 値
- error / loading / empty state がない
- keyboard navigation のテストがない
- state の唯一の指標が色だけ（文字や icon なしの red/green）
- 典型的な "AI look"（紫の gradient、巨大 card、量産 layout）

## 検証

UI を作ったあとに確認すること。

- [ ] Component が console error なしで render される
- [ ] すべての interactive element が keyboard accessible（Tab でたどれる）
- [ ] screen reader が page の内容と structure を伝えられる
- [ ] responsive - 320px、768px、1024px、1440px で動く
- [ ] loading / error / empty state をすべて扱っている
- [ ] プロジェクトの design system（spacing、colors、typography）に従っている
- [ ] DevTools または axe-core に accessibility warning がない
