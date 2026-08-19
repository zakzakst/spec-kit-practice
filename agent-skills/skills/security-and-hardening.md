---
name: security-and-hardening
description: 脆弱性に対してコードを堅くします。ユーザー入力、認証、データ保存、外部連携を扱うときに使います。未信頼データを受け取る機能、セッションを管理する機能、第三者サービスと連携する機能にも使います。個人データやプライバシー規制（GDPR、CCPA など）が絡むときにも使います。
---

# Security and Hardening

## 概要

Web application 向けの security-first な開発実践です。外部入力はすべて hostile、秘密情報は sacred、認可チェックは必須だとみなします。Security はフェーズではありません。ユーザーデータ、認証、外部システムに触れるすべての行にかかる制約です。

## 使う場面

- ユーザー入力を受け付けるものを作るとき
- 認証や認可を実装するとき
- 機密データを保存・送信するとき
- 外部 API や service と連携するとき
- file upload、webhook、callback を追加するとき
- payment や PII を扱うとき

## プロセス: まず脅威モデリング

脅威モデルなしに付けた制御は推測です。hardening の前に、攻撃者の視点で 5 分考えます。

1. **信頼境界を洗い出す。** どこで未信頼データが system に入るか？ HTTP request、form field、file upload、webhook、第三者 API、message queue、そして **LLM output** です。各境界は attack surface です。
2. **資産を名付ける。** 何が盗まれたり壊されたりするとまずいか？ credential、PII、payment data、admin action、money movement。
3. **各境界に STRIDE を当てる** - 儀式ではなく、軽い見方です。

| Threat | 問い | 一般的な対策 |
|---|---|---|
| **S**poofing | 誰かが user / service を偽装できるか？ | 認証、署名検証 |
| **T**ampering | データが転送中や保存時に改ざんされるか？ | 整合性チェック、parameterized query、HTTPS |
| **R**epudiation | 後でその action を否認できるか？ | security event の audit logging |
| **I**nformation disclosure | データが漏れるか？ | 暗号化、field allowlist、一般化された error |
| **D**enial of service | 圧倒されるか？ | rate limiting、入力サイズ制限、timeout |
| **E**levation of privilege | 本来持たない権限を得られるか？ | 認可チェック、least privilege |

4. **use case の隣に abuse case を書く。** 各機能について、「どう悪用するか？」と自問し、それを最初の test にします。

信頼境界を言えない機能は、まだ secure にする準備ができていません。これは OWASP **A04: Insecure Design** です。多くの breach は code ではなく design から始まります。

## Three-Tier Boundary System

### Always Do（例外なし）

- **すべての外部入力を検証する**（API route、form handler などの system boundary）
- **すべての database query を parameterize する** - user input を SQL に連結しない
- **出力を encode する** - XSS を防ぐ（framework の auto-escaping を使い、迂回しない）
- **すべての外部通信で HTTPS を使う**
- **password は bcrypt / scrypt / argon2 で hash する**（plaintext は保存しない）
- **security header を設定する**（CSP、HSTS、X-Frame-Options、X-Content-Type-Options）
- **session には httpOnly、secure、sameSite cookie を使う**
- **検出された package manager の native audit を、毎回 release 前に実行する**

### Ask First（人間の承認が必要）

- 新しい認証フローの追加や auth logic の変更
- 新しい種類の機微データ（PII、payment info など）の保存
- 新しい外部 service integration の追加
- CORS 設定の変更
- file upload handler の追加
- rate limiting や throttling の変更
- 権限や role の昇格

### Never Do

- **秘密情報を version control に commit しない**（API key、password、token）
- **機微データを log しない**（password、token、クレジットカード番号全文）
- **client-side validation を security boundary として信頼しない**
- **便宜のために security header を無効化しない**
- **ユーザー提供データに `eval()` や `innerHTML` を使わない**
- **認証 token を client-accessible storage（localStorage など）に保存しない**
- **stack trace や内部 error 詳細をユーザーに露出しない**

## OWASP Top 10 Prevention Patterns

これらは順位ではなく防止パターンです。2021 の順番は `../../references/security-checklist.md` の quick-reference table を参照してください。

### Injection（SQL、NoSQL、OS Command）

```typescript
// BAD: 文字列連結による SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// GOOD: parameterized query
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// GOOD: parameterized input を使う ORM
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### Broken Authentication

```typescript
// Password hashing
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// Session management
app.use(session({
  secret: process.env.SESSION_SECRET,  // code ではなく environment から
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // JavaScript から読めない
    secure: true,       // HTTPS only
    sameSite: 'lax',    // CSRF protection
    maxAge: 24 * 60 * 60 * 1000,  // 24 hours
  },
}));
```

### Cross-Site Scripting（XSS）

```typescript
// BAD: user input を HTML として描画する
element.innerHTML = userInput;

// GOOD: framework の auto-escaping を使う（React はデフォルトでそう）
return <div>{userInput}</div>;

// HTML をどうしても描画するなら、先に sanitize する
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### Broken Access Control

```typescript
// 認証だけでなく、認可も必ず確認する
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // この resource を authenticated user が所有しているか確認する
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Not authorized to modify this task' }
    });
  }

  // 更新へ進む
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### Security Misconfiguration

```typescript
// Security header（Express では helmet を使う）
import helmet from 'helmet';
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // 可能ならさらに絞る
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS - 知っている origin のみに限定する
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### Sensitive Data Exposure

```typescript
// API response で機微な field を返さない
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// 秘密情報には environment variable を使う
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');
```

### Server-Side Request Forgery（SSRF）

server が user の影響を受けた URL を fetch するとき - webhook、"import from URL"、image proxy、link preview など - 攻撃者は internal service（cloud metadata、`localhost`、private IP）を狙えます。

```typescript
// BAD: user が与えたものをそのまま fetch する
await fetch(req.body.webhookUrl);

// GOOD: scheme + host を allowlist し、解決された IP が private なら拒否、redirect も禁止
import { lookup } from 'node:dns/promises';
import ipaddr from 'ipaddr.js';

const ALLOWED_HOSTS = new Set(['hooks.example.com']);

async function assertSafeUrl(raw: string): Promise<URL> {
  const url = new URL(raw);
  if (url.protocol !== 'https:') throw new Error('https only');
  if (!ALLOWED_HOSTS.has(url.hostname)) throw new Error('host not allowed');
  // すべての record を解決する。1 つでも private/reserved があれば失敗。
  const addrs = await lookup(url.hostname, { all: true });
  if (addrs.some((a) => ipaddr.parse(a.address).range() !== 'unicast')) {
    throw new Error('private/reserved IP');
  }
  return url;
}

await fetch(await assertSafeUrl(req.body.webhookUrl), { redirect: 'error' });
```

`range() !== 'unicast'` のチェックは、loopback、link-local `169.254.169.254`（cloud metadata、SSRF の代表的標的）、private、IPv4/IPv6 の unique-local range を含みます。

**注意 - それでも TOCTOU gap は残ります。** `fetch` は check 後に DNS を再解決するため、短い TTL の record を使う攻撃者は、validation と connection の間で internal IP に rebinding できます。高リスク surface では、1 度解決した IP に接続するか、filtering agent（`request-filtering-agent` / `ssrf-req-filter`）を前段に置いてください。

## Input Validation Patterns

### Schema Validation at Boundaries

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// route handler で検証する
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid input',
        details: result.error.flatten(),
      },
    });
  }
  // result.data は型付けされ、検証済み
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### File Upload Safety

```typescript
// file type と size を制限する
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('File type not allowed');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('File too large (max 5MB)');
  }
  // file extension を信用しない - critical なら magic bytes を確認する
}
```

## Dependency Audit の結果をどう扱うか

package manager の audit は既知の advisory を報告するだけで、package が信頼できることも、脆弱な code が到達可能であることも証明しません。次の decision tree を使います。

```text
native package-manager audit が vulnerability を報告した
  Severity: critical or high
    vulnerable code は runtime / build / test / deployment path で到達可能？
      YES -> 直ちに修正（dependency を更新 / パッチ / 置き換え）
      NO（これらの path では未使用が確認済み） -> 早めに修正するが blocker ではない
    fix はある？
      YES -> patched version に更新
      NO -> workaround を探す、置き換えを検討する、または review date 付きで allowlist
  Severity: moderate
    production で到達可能？ -> 次の release cycle で修正
    dev-only？ -> 都合のよいときに修正、backlog で追跡
  Severity: low
    通常の dependency update で追う
```

**重要な質問:**
- vulnerable function は実際に code path から呼ばれているか？
- dependency は runtime か、dev-only か？
- deployment context を考えると exploit 可能か？（例: client-only app における server-side vulnerability）

保留するなら、理由を文書化し、review date を設定します。

### Supply-Chain Hygiene

npm だと決めつけたり、いちばん近い manifest を install root とみなしたりしないでください。次の順で進めます。

1. **installation boundary と manager を見つける。** lockfile を持つ workspace root を使うか、独立した nested project が workspace の外にある場合だけそちらを使います。その場合は `packageManager`（あれば）、lockfile、CI を照合し、食い違いか competing lockfile があれば止まります。manager version を固定し、`../../references/security-checklist.md` の matrix を使ってください。
2. **最初の実行前に dependency scripts を止める。** scripts を無効にした状態か、fail-closed を文書化した policy で bootstrap し、pending script の source を確認し、必要最小限の package だけを承認し、policy を commit してから、clean で frozen / immutable な install で再検証します。全部を blanket approve してはいけません。

audit は既知の advisory しか見つけません。新しく悪意を持った package や typosquat は検出できません。したがって:

- **forced audit remediation を自動適用しない**（`npm audit fix --force` など）。修正案を事前確認し、changelog を読み、結果として上がる各 upgrade をテストしてください。強制修正は declared dependency range を飛び越えることがあります。
- **対応していれば registry signature と provenance を確認する**（`npm audit signatures`、`pnpm audit signatures`）。未対応なら、無条件の安全証明ではなく調査のサインとして扱います。
- **新しい dependency、lockfile diff、script policy の変更は一緒に確認する** - ownership、maintenance、release age、provenance、transitive graph、`cross-env` と `crossenv` のような typosquat に注意します（OWASP **A06**、**LLM03**）。

## Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

// 一般 API の rate limit
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                 // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
}));

// auth endpoint はより厳しく
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 15 分で 10 回
}));
```

## Secrets Management

```text
.env files:
  .env.example  -> Committed（placeholder values の template）
  .env          -> NOT committed（real secrets を含む）
  .env.local    -> NOT committed（local overrides）

.gitignore must include:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**commit 前に必ず確認する:**

```bash
# accidentally staged した secret を確認
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

**secret が 1 度でも commit されたら、必ず rotate する。** 行を削除したり履歴を書き換えたりするだけでは不十分です。remote に到達した瞬間に compromise されたと考え、先に revoke / reissue し、その後で history から消します。

## Data Privacy & Compliance

データの security は「攻撃者が読めるか？」です。privacy は「そもそも持つべきか？ いつまで持つべきか？」です。hardening では答えられない別の問いです。守るのが安い、漏えいさせるのが安い、compliance が楽なのは、最初から集めなかった data です。個人データは、ため込む資産ではなく、減らすべき liability と考えます。

**何を持っているかを把握する。** 見つけられない data は、守ることも削除要求に応えることもできません。追加するたびに field を分類します。

| Class | 例 | 扱い |
|---|---|---|
| **Non-personal** | 集計、匿名化された count | 通常の扱い |
| **Personal (PII)** | 名前、email、IP、device / user ID | 最小化し、access-control し、export / delete に含める |
| **Sensitive** | 健康、金融、位置情報、生体認証、gov ID、未成年に関する情報 | 追加の収集根拠、より厳しい access、しばしば暗号化 + audit logging |

**運用ルール:**
- **最小化して目的を定める。** field は stated use に対してのみ収集する。「あとで役立つかも」は目的ではありません。latent breach scope です。telemetry に PII を流さないでください（`observability-and-instrumentation` も ops 側から同じことを言っています）。
- **最初に retention を決めて、実際に削除する。** すべての personal-data store には TTL と working deletion path が必要です。backup、cache、search index、analytics copy も含みます。期限のない data は、後で起こる breach の予約券です。
- **自分の jurisdiction が求める data-subject rights を支援する**（GDPR / CCPA など）。export、correct、delete の要求は engineering feature です。schema は、ユーザーの data を *見つけられて* *消せる* ように設計し、system 全体に不可逆にばら撒かないでください。
- **収集や third-party sharing の前に consent を得て、監査可能にする。** PII を analytics / ad / LLM vendor に送るのは "sharing" です。ユーザーの選択が gate であり、vendor には data-processing agreement が必要です。
- **defaults は localize し、1 つの region の law を hardcode しない。** data residency や規則は user location によって異なります。policy は前提ではなく、設定可能な boundary にしてください。

data が信頼境界を越えるときは、未信頼として検証します（上の Input Validation を参照）。privacy incident で個人データが漏れたときは、breach notification の時計も postmortem の一部です。`debugging-and-error-recovery` に従ってください。

## Securing AI / LLM Features

アプリが LLM を呼ぶなら - chatbot、summarizer、agent、RAG など - 新しい attack surface を背負います。[OWASP Top 10 for LLM Applications (2025)](https://genai.owasp.org/llm-top-10/) に対応付けます。

- **モデル出力はすべて未信頼入力として扱う（LLM05: Improper Output Handling）。** LLM output を `eval`、SQL、shell、`innerHTML`、file path にそのまま流さないでください。生の user input と同じように検証し、encode します。
- **prompt は hijack されうると考える（LLM01: Prompt Injection）。** context window の未信頼テキスト - user message、取得した web page、PDF など - には instruction が含まれます。system prompt は security boundary ではありません。permissions は prompt ではなく code で enforcement します。
- **秘密や他ユーザーのデータを prompt に入れない（LLM02 / LLM07）。** context に入ったものは返されうると思ってください。API key、cross-tenant data、全文の system prompt を、モデルが反復できる場所に置かないでください。
- **tool と agent の権限を絞る（LLM06: Excessive Agency）。** tool は最小限にし、破壊的・不可逆な action には確認を求め、すべての tool argument を検証します。
- **消費を制限する（LLM10: Unbounded Consumption）。** token、request rate、loop / recursion depth を制限し、crafted input でコスト増大や hang を起こせないようにします。
- **retrieval data を分離する（LLM08: Vector and Embedding Weaknesses）。** RAG では vector store を信頼境界として扱います。embedding を tenant ごとに分けて他人の data を取れないようにし、index 前に document を検証して poisoned content が回答を誘導しないようにします。

```typescript
// BAD: model output を command や markup として信頼する
const sql = await llm.generate(`Write SQL for: ${userQuestion}`);
await db.query(sql);                                   // arbitrary query execution
container.innerHTML = await llm.reply(userMessage);   // stored XSS, via the model

// GOOD: model output は data として扱い、parse -> validate -> encode
let intent;
try {
  intent = CommandSchema.parse(JSON.parse(await llm.replyJson(userMessage)));
} catch {
  throw new ValidationError('unexpected model output'); // JSON.parse か schema が失敗
}
await runAllowlistedAction(intent.action, intent.params);
container.textContent = await llm.reply(userMessage);
```

## Security Review Checklist

```markdown
### Authentication
- [ ] Password は bcrypt / scrypt / argon2 で hash されている（salt rounds 12 以上）
- [ ] Session token は httpOnly、secure、sameSite
- [ ] Login に rate limiting がある
- [ ] Password reset token に期限がある

### Authorization
- [ ] すべての endpoint が user permissions を確認する
- [ ] ユーザーは自分の resource だけにアクセスできる
- [ ] admin action に admin role の確認が必要

### Input
- [ ] すべての user input が boundary で検証されている
- [ ] SQL query が parameterized である
- [ ] HTML output が encode / escape されている
- [ ] server-side の URL fetch が allowlist されている（内部 service への SSRF なし）

### Data
- [ ] code や version control に秘密情報がない
- [ ] 機微な field が API response から除外されている
- [ ] PII が at rest で暗号化されている（必要な場合）
- [ ] 個人データが分類され、明示した目的のために最小化されている
- [ ] 個人データに retention limit と working deletion path がある（backup / index も含む）
- [ ] 必要な場合、export / delete 要求を支援し、第三者共有には consent がある

### Infrastructure
- [ ] Security header（CSP、HSTS など）が設定されている
- [ ] CORS が既知の origin に限定されている
- [ ] 依存関係が脆弱性監査を通っている
- [ ] error message が内部詳細を露出しない

### Supply Chain
- [ ] 単一の authoritative lockfile が commit され、CI はその manager の frozen / immutable install を使う
- [ ] native audit が reachability と修正リスクで triage され、dependency install script は明示的に承認されるまで止められている
- [ ] 新しい dependency が ownership、provenance、release age、transitive graph の観点でレビューされている

### AI / LLM（使う場合）
- [ ] model output を未信頼として扱っている（eval / SQL / innerHTML / shell なし）
- [ ] secrets と他ユーザーの data を prompt に入れていない
- [ ] tool / agent の権限が絞られ、破壊的 action には確認が必要
```

## See Also

詳細な security checklist と pre-commit verification steps は `../../references/security-checklist.md` を参照してください。

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「これは internal tool だから security は大事じゃない」 | internal tool も侵害されます。攻撃者は最も弱い link を狙います。 |
| 「security は後で付ける」 | security の後付けは、最初から入れるより 10 倍難しいです。今入れてください。 |
| 「こんなものを exploit する人はいない」 | 自動 scanner は見つけます。security by obscurity は security ではありません。 |
| 「framework が security を面倒見てくれる」 | framework は tool を提供するだけで、保証はしません。正しく使うのは自分です。 |
| 「ただの prototype だ」 | prototype は production になります。最初の日から security habits を持ちましょう。 |
| 「脅威モデリングはやりすぎ」 | "どう攻撃するか" を 5 分考えるだけで、あとから control では直せない design flaw を防げます。 |
| 「LLM output はただの text だ」 | その "text" は SQL statement にも script tag にも shell command にもなります。未信頼入力として扱ってください。 |
| 「audit が通ったから dependency は安全」 | audit は既知の advisory を見るだけです。新しく悪意を持った package や unreviewed install script の安全は保証しません。 |
| 「今集めておけば、あとで使うかもしれない」 | 持っていない data は breach も subpoena も誤削除もされません。"かもしれない" は目的ではなく breach scope です。 |
| 「delete request は手作業でやる」 | 手作業の削除では backup、cache、analytics copy を取りこぼします。schema から見つからない data は、要求に応えられません。 |
| 「compliance は legal の仕事でしょ」 | export、削除、retention、consent は schema と code の仕事です。PII を 10 個の system に散らした後では、legal では後付けできません。 |

## レッドフラグ

- ユーザー入力がそのまま database query、shell command、HTML rendering に入る
- source code や commit history に秘密情報がある
- authentication / authorization なしの API endpoint
- CORS 設定なし、または wildcard (`*`) origin
- auth endpoint に rate limiting がない
- stack trace や内部 error がユーザーに見えている
- 既知の critical vulnerability を持つ dependency、1 つの installation boundary に competing lockfile、reproducible でない install、または blanket-approved script
- server がユーザー指定 URL を allowlist なしで fetch する（SSRF）
- LLM / model output を query、DOM、shell、`eval` に流している
- secrets、PII、全文の system prompt を LLM context window に入れている
- 個人データを目的も retention limit も削除 path もなく収集している
- consent なし / data-processing agreement なしで PII を analytics / ad / LLM vendor に送っている
- `Delete my account` が flag を変えるだけで、個人データが store や backup に残り続ける

## 検証

security 関連の code を実装したあと、次を確認します。

- [ ] native audit に未解決の reachable critical / high はない。CI は authoritative lockfile を保ち、未レビューの dependency script を止めている
- [ ] source code や git history に秘密情報がない
- [ ] すべての user input が system boundary で検証されている
- [ ] 認証と認可が保護されたすべての endpoint で確認されている
- [ ] security header が response にある（browser DevTools で確認）
- [ ] error response が内部詳細を露出しない
- [ ] auth endpoint に rate limiting がある
- [ ] server-side URL fetch が allowlist で検証されている（SSRF なし）
- [ ] LLM / model output が使う前に検証・encode されている（AI 機能がある場合）
- [ ] 個人データが分類され、明示した目的のために最小化され、retention limit がある
- [ ] deletion / export 要求が end-to-end で動く（backup、cache、analytics copy を含む）
