---
name: api-and-interface-design
description: 安定した API と interface の設計を支援します。API、module boundary、公開 interface を設計するときに使います。REST や GraphQL endpoint、module 間の type contract、frontend/backend の境界を作るときに使います。
---

# API と Interface の設計

## 概要

誤用しにくい、安定した、よく文書化された interface を設計します。良い interface は、正しいことを簡単にし、間違ったことを難しくします。これは REST API、GraphQL schema、module boundary、component props など、ある code 片が別の code 片と話す surface に適用されます。

## 使う場面

- 新しい API endpoint を設計するとき
- module boundary や team 間の contract を定義するとき
- component prop interface を作るとき
- API 形状に影響する database schema を決めるとき
- 既存の public interface を変更するとき

## コア原則

### Hyrum's Law

> 十分な数の API 利用者がいると、契約で約束していなくても、システムの観測可能な振る舞いのすべてが誰かに依存されるようになる。

つまり、公開された振る舞い - 文書化されていない癖、エラーメッセージの文面、タイミング、順序を含む - は、利用者が依存し始めた瞬間に事実上の contract になります。設計上の示唆は次の通りです。

- **何を公開するかは意図的に決める。** 観測可能な振る舞いはすべて約束候補です。
- **実装詳細を漏らさない。** 観測できれば、利用者は依存します。
- **非推奨化は設計時から考える。** 利用者が依存するものを安全に外す方法は `deprecation-and-migration` を参照してください。
- **テストだけでは足りない。** どれだけ contract test が完璧でも、Hyrum's Law により、"安全な" 変更が未文書の振る舞いに依存している実利用者を壊すことがあります。

### One-Version Rule

同じ dependency や API の複数 version を consumer に選ばせないようにします。Diamond dependency 問題は、異なる consumer が同じものの異なる version を必要とするときに起こります。1 度に 1 つの version だけが存在する世界を前提に設計し、fork ではなく extension を選びます。

### 1. Contract First

実装より先に interface を定義します。contract が spec であり、実装はその後です。

```typescript
// まず contract を定義する
interface TaskAPI {
  // task を作成し、server-generated fields を含む作成済み task を返す
  createTask(input: CreateTaskInput): Promise<Task>;

  // filter に一致する paginated task を返す
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // 単一 task を返すか、NotFoundError を投げる
  getTask(id: string): Promise<Task>;

  // partial update - 指定された field だけを変更する
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // idempotent delete - すでに削除済みでも成功する
  deleteTask(id: string): Promise<void>;
}
```

### 2. 一貫した Error Semantics

1 つの error strategy を選び、全体で統一します。

```typescript
// REST: HTTP status codes + structured error body
// すべての error response は同じ shape にする
interface APIError {
  error: {
    code: string;        // machine-readable: "VALIDATION_ERROR"
    message: string;     // human-readable: "Email is required"
    details?: unknown;   // 必要なら追加情報
  };
}

// status code の対応
// 400 -> client が不正なデータを送った
// 401 -> 未認証
// 403 -> 認証済みだが権限なし
// 404 -> resource not found
// 409 -> conflict（duplicate、version mismatch）
// 422 -> validation failed（semantic 的に不正）
// 500 -> server error（内部詳細は絶対に出さない）
```

**混在させないでください。** endpoint によって throw したり、null を返したり、`{ error }` を返したりすると、consumer は挙動を予測できません。

### 3. 境界で検証する

内部コードは信頼します。外部入力が入る system edge で検証します。

```typescript
// API boundary で検証する
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid task data',
        details: result.error.flatten(),
      },
    });
  }

  // 以降は internal code が型を信頼できる
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

検証を置く場所:
- API route handler（ユーザー入力）
- form submission handler（ユーザー入力）
- external service response の parsing（第三者データ - **常に未信頼**）
- environment variable の読み込み（設定）

> **第三者 API response は未信頼データです。** logic、rendering、decision-making に使う前に、shape と内容を検証してください。侵害された、またはおかしな external service は、予期しない型、悪意ある content、指示らしい文言を返すことがあります。

検証を置かない場所:
- 既に type contract を共有している internal function 間
- 既に検証済みの code から呼ばれる utility
- 自分の database から来たばかりの data

### 4. 変更より追加を優先する

既存 consumer を壊さずに interface を拡張します。

```typescript
// Good: optional field を追加
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // 後から追加、任意
  labels?: string[];                     // 後から追加、任意
}

// Bad: 既存 field の型変更や削除
interface CreateTaskInput {
  title: string;
  // description: string;  // 削除 - 既存 consumer を壊す
  priority: number;        // string から変更 - 既存 consumer を壊す
}
```

### 5. 予測可能な命名

| パターン | 規約 | 例 |
|---|---|---|
| REST endpoint | 複数形 noun、verb なし | `GET /api/tasks`, `POST /api/tasks` |
| Query param | camelCase | `?sortBy=createdAt&pageSize=20` |
| Response field | camelCase | `{ createdAt, updatedAt, taskId }` |
| Boolean field | is/has/can prefix | `isComplete`, `hasAttachments` |
| Enum value | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

### 6. Idempotency Key をきちんと扱う

`Idempotency-Key` を受け取るのは contract です。それを honour するのは implementation であり、そこが金を失う場所です。サーバーが受け取るだけで雑に扱う key は、ないより悪いです。再試行しても安全だと client が信じてしまうからです。

**意図から key を導き、試行からは導かない。** key は 1 つの意図に対する retry では同じで、別の意図では違う必要があります。

```typescript
crypto.randomUUID()                    // 笨・試行ごとに新しい key - retry ごとに別 charge になる
`${userId}:${amount}`                  // 笨・同じ $50 の正当な charge が 1 つに潰れる
`${orderId}:${Date.now()}`             // 笨・timestamp は randomUUID を被ったもの

req.headers['idempotency-key']         // 笨・client が 1 回作って retry で再利用する
`charge:v1:${orderId}`                 // 笨・不変 ID から導く
```

key は client か、意図を起こした event から来るべきであり、retry を担当する layer から来てはいけません。

**チェックと実行は一緒にする。分けると race です。**

```typescript
// 笨・TOCTOU: 同時 retry が両方 "not seen" を読んで、両方 charge する
if (!(await db.exists(key))) {
  await chargeCard(amount);
  await db.insert(key);
}

// 笨・unique constraint に勝負を任せる
try {
  await db.insert({ key, state: 'in_progress', requestHash });
} catch (e) {
  if (isUniqueViolation(e)) return replayOrReject(key);
  throw;
}
const result = await chargeCard(amount);
await db.update({ key, state: 'succeeded', response: result });
```

unique constraint 自体が mechanism です。1 回で uniqueness を強制できない store では、この契約を支えられません。

**payload を守る。** 同じ key で body が違うなら client bug なので、静かに最初の response を返すのではなく、はっきり失敗させます。

```typescript
if (existing.requestHash !== hash(req.body)) {
  return res.status(422).json({ error: 'idempotency key reused with a different payload' });
}
```

**in-flight duplicate の扱いを決める。** 1 回目がまだ実行中に 2 回目が来るのは、retry storm で普通に起こります。

| Strategy | Response | 使う場面 |
|---|---|---|
| Reject | `409 Conflict` | 後で retry できる。最も単純で安全 |
| Wait | 結果を bounded に待つ | 同期的に必要 |
| Return pending | `202` + status URL | 長時間かかる効果 |

1 回目が "詰まって見える" からといって 2 回目を通してはいけません。止まっている attempt の fate が分からないときこそ、重複コストが最も大きいです。

**各呼び出しの outcome は 2 つではなく 3 つです。** success、failure、そして _unknown_ です。timeout は effect が適用されたか何も教えてくれません。call out する前に intent を記録しておけば、call と response の間で crash しても、「あとで解決すべき何か」が残ります。黙って再送される charge にはなりません。

**保有期間は最長の retry chain から決める。** disk cost ではありません。key は、同じ意図を再送できるすべての経路より長く生きる必要があります。dead-letter queue が 1 週間後に再送される場合や、provider の dispute window も含みます。24 時間の key TTL では、7 日の DLQ の後で duplicate が起きます。

## REST API パターン

### Resource Design

```text
GET    /api/tasks              -> task を一覧表示（filter は query params）
POST   /api/tasks              -> task を作成
GET    /api/tasks/:id          -> task 1 件を取得
PATCH  /api/tasks/:id          -> task を部分更新
DELETE /api/tasks/:id          -> task を削除

GET    /api/tasks/:id/comments -> task の sub-resource として comments を一覧
POST   /api/tasks/:id/comments -> task に comment を追加
```

### Pagination

一覧 endpoint は paginate します。

```typescript
// Request
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// Response
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### Filtering

filter には query parameter を使います。

```text
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### Partial Updates（PATCH）

部分オブジェクトを受け取り、渡されたものだけ更新します。

```typescript
// title だけ変わり、他は維持される
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## TypeScript Interface パターン

### Discriminated Union を使う

```typescript
// Good: 各 variant が明示的
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// Consumer は type narrowing できる
function getStatusLabel(status: TaskStatus): string {
  switch (status.type) {
    case 'pending': return 'Pending';
    case 'in_progress': return `In progress (${status.assignee})`;
    case 'completed': return `Done on ${status.completedAt}`;
    case 'cancelled': return `Cancelled: ${status.reason}`;
  }
}
```

### Input / Output を分ける

```typescript
// Input: caller が提供するもの
interface CreateTaskInput {
  title: string;
  description?: string;
}

// Output: system が返すもの（server-generated fields を含む）
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### ID には Branded Type を使う

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// TaskId を期待する場所に UserId を誤って渡すのを防ぐ
function getTask(id: TaskId): Promise<Task> { ... }
```

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「API はあとで文書化する」 | types が文書です。先に定義してください。 |
| 「今は pagination いらない」 | 100 件を超えた瞬間に必要になります。最初から入れてください。 |
| 「PATCH は難しいから PUT でいい」 | PUT は毎回完全 object が必要。client が実際に欲しいのは PATCH です。 |
| 「必要になったら versioning する」 | versioning なしの breaking change は consumer を壊します。最初から拡張前提で設計してください。 |
| 「あの undocumented behavior を使う人はいない」 | Hyrum's Law: 観測できるなら誰かが依存します。公開振る舞いはすべて約束として扱ってください。 |
| 「2 つの version を保守すればいい」 | 複数 version は保守コストを倍増させ、diamond dependency 問題を生みます。One-Version Rule を優先してください。 |
| 「内部 API には contract なんていらない」 | 内部 consumer も consumer です。contract は結合を防ぎ、並列作業を可能にします。 |
| 「Idempotency-Key ヘッダーを受け取るだけで十分」 | ヘッダーは contract であり、key を result に紐づけて保存するのが implementation です。受け取るだけで honour しない key は、retry して安全だと client に誤認させます。 |
| 「queue は exactly-once を保証する」 | consumer crash をまたいで保証する queue はありません。broker の ack と side effect は 1 transaction ではありません。at-least-once + idempotent processing で設計してください。 |
| 「duplicate request はまれ」 | それらは相関します。依存先が劣化したときに retry が増え、まさに duplicate が最も起きやすく、最も高くつきます。 |

## レッドフラグ

- 条件によって違う shape を返す endpoint
- endpoint 間で error format が不一致
- 検証が internal code 全体に散らばっている
- 既存 field への破壊的変更（type change、削除）
- pagination のない一覧 endpoint
- REST URL に verb が入っている（`/api/createTask`、`/api/getUsers`）
- third-party API response を検証やサニタイズなしで使っている
- idempotency key を探してから insert する - それは guard ではなく race です
- UUID、timestamp、または試行ごとに再生成されるもので idempotency key を作っている
- 同じ key で別 payload が来ても、静かに最初の response を返している
- key retention が、request を再送しうる最長経路より短い

## 検証

API を設計したあと、次を確認します。

- [ ] すべての endpoint に typed input / output schema がある
- [ ] error response が 1 つの一貫した format に従っている
- [ ] 検証は system boundary でのみ行われている
- [ ] 一覧 endpoint は pagination に対応している
- [ ] 新しい field は追加的で optional（後方互換）
- [ ] 命名がすべての endpoint で一貫している
- [ ] API documentation または types が実装と一緒にコミットされている
- [ ] state-changing endpoint は idempotency key を honour するか、再試行 unsafe として文書化されている
- [ ] key は unique constraint で 1 回の atomic operation で claim される
- [ ] 異なる payload で再利用された key は、間違った response を replay せず、はっきり失敗する
- [ ] in-flight duplicate の扱いは、409 / wait / 202 のどれかに明示的に決まっている
- [ ] key retention は dead-letter replay を含む最長 retry path を超えている
