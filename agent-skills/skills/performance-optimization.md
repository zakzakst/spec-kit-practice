---
name: performance-optimization
description: frontend、backend、query、database 全体の performance を最適化します。性能要件があるとき、性能 regression を疑うとき、Core Web Vitals や load time を改善したいとき、N+1 query を直したいとき、profiling で bottleneck が見つかったときに使います。
---

# Performance Optimization

## 概要

最適化の前に計測します。計測なしの performance work は推測です。推測は、何にも効かないのに複雑さだけ増やす premature optimization を招きます。まず profile し、実際の bottleneck を特定し、それだけを直し、もう一度計測します。測定で効くと証明されたものだけを最適化します。

## 使う場面

- spec に performance 要件があるとき（load time budget、response time SLA など）
- ユーザーや monitoring が遅さを報告したとき
- Core Web Vitals が基準未満のとき
- 変更が regression を入れたと疑うとき
- 大きな dataset や高トラフィックを扱う機能を作るとき

**使わない場面:** 問題の証拠がないうちに最適化しないこと。premature optimization は、得られる performance より多くの複雑さを払います。

## Core Web Vitals の目標

| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| **LCP**（Largest Contentful Paint） | 2.5s 以下 | 4.0s 以下 | 4.0s 超 |
| **INP**（Interaction to Next Paint） | 200ms 以下 | 500ms 以下 | 500ms 超 |
| **CLS**（Cumulative Layout Shift） | 0.1 以下 | 0.25 以下 | 0.25 超 |

## Optimization Workflow

```text
1. MEASURE   -> 実測で baseline を作る
2. IDENTIFY  -> 実際の bottleneck を見つける（思い込みではなく）
3. FIX       -> その bottleneck を直す
4. VERIFY    -> 再計測し、残すか戻すか決める
5. GUARD     -> 回帰防止の monitoring や test を追加する
```

### Step 1: Measure

補完的な 2 つの手法を使います。両方使ってください。

- **Synthetic（Lighthouse、DevTools Performance tab）:** 条件が一定で再現性がある。CI の regression 検出や、特定問題の切り分けに向いています。
- **RUM（web-vitals library、CrUX）:** 実ユーザーの実環境データです。修正が本当に user experience を改善したか確認するには必須です。

**Frontend:**

```bash
# Synthetic: Chrome DevTools の Lighthouse（または CI）
# Chrome DevTools -> Performance tab -> Record
# Chrome DevTools MCP -> Performance trace

# RUM: code に web-vitals library
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

**Backend:**

```bash
# response time logging
# Application Performance Monitoring (APM)
# timing 付きの database query logging

# 単純な timing
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

### どこから計測を始めるか

症状から、まず何を計測すべきかを決めます。

```text
何が遅い？
  First page load
    -> Large bundle? bundle size を計測し、code splitting を確認
    -> Slow server response? DevTools Network waterfall で TTFB を測る
      -> DNS が長い？ 知っている origin に dns-prefetch / preconnect を追加
      -> TCP/TLS が長い？ HTTP/2、edge deployment、keep-alive を確認
      -> Waiting（server）が長い？ backend を profile し、query と caching を確認
    -> Render-blocking resources? CSS/JS の blocking を network waterfall で確認
  Interaction feels sluggish
    -> UI が click で固まる？ main thread を profile し、long task（50ms 超）を探す
    -> Form input が遅い？ re-render や controlled component の負荷を確認
    -> Animation が引っかかる？ layout thrashing、forced reflow を確認
  Page after navigation
    -> Data loading? API response time を測り、waterfall を確認
    -> Client rendering? component render time を profile し、N+1 fetch を確認
  Backend / API
    -> 単一 endpoint が遅い？ database query を profile し、index を確認
    -> 全 endpoint が遅い？ connection pool、memory、CPU を確認
    -> ときどき遅い？ lock contention、GC pause、external dependency を確認
```

### Step 2: ボトルネックを特定する

カテゴリごとの一般的な bottleneck です。

**Frontend:**

| 症状 | あり得る原因 | 調べ方 |
|---|---|---|
| LCP が遅い | 大きな画像、render-blocking resource、遅い server | network waterfall、image size を確認 |
| CLS が大きい | サイズ未指定の画像、遅れて入る content、font shift | layout shift attribution を確認 |
| INP が悪い | main thread 上の重い JavaScript、巨大 DOM 更新 | Performance trace で long task を確認 |
| 初回 load が遅い | 大きな bundle、多数の network request | bundle size、code splitting を確認 |

**Backend:**

| 症状 | あり得る原因 | 調べ方 |
|---|---|---|
| API response が遅い | N+1 query、index 不足、最適化不足の query | database query log を確認 |
| memory が増え続ける | reference の漏れ、無制限 cache、大きな payload | heap snapshot を解析 |
| CPU が急上昇する | 同期的な重い計算、regex backtracking | CPU profiling |
| latency が高い | caching 不足、重複計算、network hop | stack を通して trace する |

### Step 3: よくあるアンチパターンを直す

#### N+1 Query（Backend）

```typescript
// BAD: task ごとに owner を 1 query ずつ取りに行く N+1
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// GOOD: join / include を使って 1 回で取る
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

#### 無制限のデータ取得

```typescript
// BAD: 全件取得
const allTasks = await db.tasks.findMany();

// GOOD: limit 付き pagination
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

#### 画像最適化不足（Frontend）

```html
<!-- BAD: dimensions も format 最適化もない -->
<img src="/hero.jpg" />

<!-- GOOD: Hero / LCP image - art direction + resolution switching, high priority -->
<!--
  2 つの技術を組み合わせる:
  - Art direction（media）: breakpoint ごとに crop / composition を変える
  - Resolution switching（srcset + sizes）: 画面密度に合う file size を返す
-->
<picture>
  <!-- Mobile: portrait crop (8:10) -->
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.avif 400w, /hero-mobile-800.avif 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/avif"
  />
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.webp 400w, /hero-mobile-800.webp 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/webp"
  />
  <!-- Desktop: landscape crop (2:1) -->
  <source
    srcset="/hero-800.avif 800w, /hero-1200.avif 1200w, /hero-1600.avif 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/avif"
  />
  <source
    srcset="/hero-800.webp 800w, /hero-1200.webp 1200w, /hero-1600.webp 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/webp"
  />
  <img
    src="/hero-desktop.jpg"
    width="1200"
    height="600"
    fetchpriority="high"
    alt="Hero image description"
  />
</picture>

<!-- GOOD: fold の下の image - lazy load + async decoding -->
<img
  src="/content.webp"
  width="800"
  height="400"
  loading="lazy"
  decoding="async"
  alt="Content image description"
/>
```

#### 不要な Re-render（React）

```tsx
// BAD: 毎回新しい object を作るので child が再レンダーされる
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// GOOD: 安定した reference
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}

// 重い component には React.memo を使う
const TaskItem = React.memo(function TaskItem({ task }: Props) {
  return <div>{/* expensive render */}</div>;
});

// 重い計算には useMemo を使う
function TaskStats({ tasks }: Props) {
  const stats = useMemo(() => calculateStats(tasks), [tasks]);
  return <div>{stats.completed} / {stats.total}</div>;
}
```

#### Bundle Size が大きい

```typescript
// Modern bundler（Vite, webpack 5+）は、dependency が ESM で package.json に `sideEffects: false` があるなら、
// named import を tree-shaking で自動処理します。import style を変える前に profile してください。
// 本当に効くのは split と lazy loading です。

// GOOD: 重くてまれにしか使わない機能は dynamic import
const ChartLibrary = lazy(() => import('./ChartLibrary'));

// GOOD: route-level code splitting を Suspense で包む
const SettingsPage = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <SettingsPage />
    </Suspense>
  );
}
```

#### Caching 不足（Backend）

```typescript
// 頻繁に読むが滅多に変わらない data を cache する
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes
let cachedConfig: AppConfig | null = null;
let cacheExpiry = 0;

async function getAppConfig(): Promise<AppConfig> {
  if (cachedConfig && Date.now() < cacheExpiry) {
    return cachedConfig;
  }
  cachedConfig = await db.config.findFirst();
  cacheExpiry = Date.now() + CACHE_TTL;
  return cachedConfig;
}

// static asset 用の HTTP caching header
app.use('/static', express.static('public', {
  maxAge: '1y',           // 1 年 cache
  immutable: true,        // 再検証しない（filename に content hash を使う）
}));

// API response の Cache-Control
res.set('Cache-Control', 'public, max-age=300'); // 5 minutes
```

### Step 4: 検証する（残すか戻すか）

fix は、再計測するまでは仮説です。この段階で残すか戻すかを決めます。

**baseline を測ったのと同じ方法で再計測する:** 同じ command、同じ条件、同じ固定 budget（wall-clock、sample count、request count）。cold cache で測った baseline と warm cache で測った結果を比べると、変えたものではなく cache を測ったことになります。

**1 度に 1 つだけ変える。** 3 つの最適化をまとめて入れると 1 つの数字しか得られず、どれが効いたか分かりません。まとめて出す必要があるなら、先に 1 つずつ単独で測ります。

**平均だけでなく noise に勝つ。** 繰り返し計測し、delta が run-to-run variance を超えるか確認します。5% の variance の中の 3% 改善は改善ではありません。別の sample です。

その後、次のように厳密に判断します。

| baseline との結果 | 対応 |
|---|---|
| threshold を超え、テストも green | **Keep.** before/after の数字を commit message に書く |
| noise の範囲（測定可能な変化なし） | **Revert.** |
| 悪化した | **Revert.** |
| 改善したが test が red | **Revert.** 勝利の衣を着た regression です |

**"Neutral" は keep ではなく revert です。** チームがよく飛ばすのがこの段階です。変更はもう書いたし、捨てるのはもったいないと感じて、そのまま置いてしまう。そうすると、何も得ていないのに複雑さだけが残ります。keep した code は、ずっと保守することになります。ちゃんと元を取らせてください。

**正しさが metric を支配します。** suite が green であることと、数字が動くことの両方が必要です。必要な work を削ることで勝ったように見える "最適化"（validation を飛ばす、fresh である必要のある cache をキャッシュする、load-bearing な `await` を消す）は regression であって勝利ではありません。

#### 失敗した試みも含めて、すべて記録する

revert した work は git history に残りません。だからこそ、同じ死んだアイデアが次の四半期にまた試されます。短い ledger を残し、却下された案を再提案させないようにします。

| Idea | Baseline -> Result | Verdict | Why |
|---|---|---|---|
| row component を memoize する | INP 240ms -> 235ms | reverted | noise の範囲（15ms）。row は bottleneck ではなかった |
| list を virtualize する | INP 240ms -> 90ms | kept | long task が trace から消えた |
| API origin に preconnect する | LCP 2.8s -> 2.8s | reverted | すでに same-origin |

PR description の一節でも、repo に `PERF.md` を置くのでも構いません。重要なのは、次の人（あるいは次の agent）がそれを読んでから experiment を提案し、すでに失敗したものを再実行しないことです。

## Performance Budget

budget を決めて強制します。

```text
JavaScript bundle: < 200KB gzipped（initial load）
CSS: < 50KB gzipped
Images: < 200KB per image（above the fold）
Fonts: < 100KB total
API response time: < 200ms（p95）
Time to Interactive: < 3.5s on 4G
Lighthouse Performance score: 90 以上
```

**CI で enforce する:**

```bash
# Bundle size check
npx bundlesize --config bundlesize.config.json

# Lighthouse CI
npx lhci autorun
```

## See Also

詳細な performance checklist、optimization command、anti-pattern の参照は `../../references/performance-checklist.md` を見てください。

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「後で最適化する」 | performance debt は複利で増えます。明らかな anti-pattern は今直し、micro-optimization は後回しにしてください。 |
| 「自分のマシンでは速い」 | あなたのマシンはユーザーのマシンではありません。代表的な hardware と network で profile してください。 |
| 「この最適化は明らか」 | 計測していなければ分かっていません。まず profile してください。 |
| 「100ms くらい誰も気にしない」 | 研究では 100ms の遅延でも conversion rate に影響します。ユーザーは思っている以上に気づきます。 |
| 「framework が performance を面倒見てくれる」 | framework は一部の問題は防げますが、N+1 query や大きすぎる bundle は直せません。 |
| 「あまり効かなかったけど、害もない」 | neutral 変更は revert です。そこには永遠に保守コストを払うだけで、見返りがありません。 |
| 「もう書いたんだから、残すしかない」 | sunk cost です。計測は書くのにかかった時間を気にしません。 |
| 「改善は明らかだから再計測は不要」 | なら再計測は安いですし、本当に証明できます。未計測の勝利が、neutral な複雑さを呼び込みます。 |

## レッドフラグ

- profiling data なしの最適化
- data fetching における N+1 query
- pagination のない一覧 endpoint
- dimensions、lazy loading、responsive size のない image
- レビューなしに増え続ける bundle size
- production での performance monitoring がない
- `React.memo` と `useMemo` を everywhere で使う（使いすぎも使わなさすぎもダメ）
- 再計測なしで残した最適化
- 複数の最適化を 1 つの計測にまとめ、個別に帰属できない
- test を変えたり飛ばしたり消したりしないと勝てない "win"
- 最初の失敗を記録しなかったため、同じ失敗した最適化を何度も試す

## 検証

performance 関連の変更のあとに確認すること。

- [ ] before / after の計測がある（具体的な数字）
- [ ] baseline と同じ方法（同じ command、同じ条件）で再計測した
- [ ] 改善が mean だけでなく run-to-run variance を超えている
- [ ] baseline に勝てなかった変更は、neutral のまま残さず revert した
- [ ] 試みを kept / reverted ともに記録し、死んだアイデアを再実行しないようにした
- [ ] 具体的な bottleneck を特定して対処した
- [ ] Core Web Vitals が "Good" の範囲にある
- [ ] bundle size が大きく増えていない
- [ ] 新しい data fetching code に N+1 query がない
- [ ] performance budget が CI で通る（設定している場合）
- [ ] 既存テストが通る（最適化で振る舞いを壊していない）
