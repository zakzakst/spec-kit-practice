---
applyTo: "*"
description: "あらゆる言語、フレームワーク、スタックに対応した、エンジニアが執筆した、最も包括的で実践的なパフォーマンス最適化の手順です。実用的なガイダンス、シナリオベースのチェックリスト、トラブルシューティング、プロのヒントなど、フロントエンド、バックエンド、データベースのベストプラクティスを網羅しています。"
---

# パフォーマンス最適化のベストプラクティス

## 導入

パフォーマンスは単なる流行語ではありません。人々に愛される製品と、愛されなくなる製品を分けるものです。遅いアプリがユーザーを苛立たせ、クラウド料金を膨らませ、顧客を失うことさえあるのを、私は身をもって経験してきました。このガイドは、私が実践し、レビューしてきた最も効果的で実践的なパフォーマンスプラクティスを、フロントエンド、バックエンド、データベース層に加え、高度なトピックも網羅した、生きたコレクションです。高速で効率的、そしてスケーラブルなソフトウェアを構築するための参考資料、チェックリスト、そしてインスピレーションの源としてご活用ください。

---

## 一般原則

- **まず測定し、次に最適化する:** 最適化を行う前に、必ずプロファイリングと測定を実施してください。ベンチマーク、プロファイラー、監視ツールなどを活用し、真のボトルネックを特定してください。推測はパフォーマンスの敵です。
  - _プロのヒント:_ Chrome DevTools、Lighthouse、New Relic、Datadog、Py-Spy、または言語に組み込まれているプロファイラーなどのツールを使用します。
- **一般的なケースに最適化:** 最も頻繁に実行されるコードパスの最適化に重点を置きましょう。稀なエッジケースについては、それが極めて重要でない限り、時間を無駄にしないでください。
- **早すぎる最適化を避ける:** まず明確で保守しやすいコードを書き、必要な場合にのみ最適化してください。時期尚早な最適化は、コードの可読性と保守性を低下させる可能性があります。
- **リソース使用量を最小限に抑える:** メモリ、CPU、ネットワーク、ディスクリソースを効率的に使用してください。常に「より少ないリソースでこれを行うことはできないか？」と自問してください。
- **シンプルさを好む:** シンプルなアルゴリズムとデータ構造は、多くの場合、より高速で最適化が容易です。過剰なエンジニアリングは避けましょう。
- **パフォーマンスの想定を文書化する:** パフォーマンスが重要なコードや、分かりにくい最適化が施されているコードには、明確なコメントを付けてください。将来のメンテナー（あなた自身も含む）にとって、きっと感謝されるでしょう。
- **プラットフォームを理解する:** 言語、フレームワーク、ランタイムのパフォーマンス特性を理解しましょう。Python では高速なものが JavaScript では遅い場合があり、その逆も同様です。
- **パフォーマンステストの自動化:** パフォーマンステストとベンチマークをCI/CDパイプラインに統合し、リグレッションを早期に発見しましょう。
- **パフォーマンス予算を設定する:** 読み込み時間、メモリ使用量、API レイテンシなどの許容制限を定義します。自動チェックでそれらを強制します。

---

## フロントエンドパフォーマンス

### レンダリングとDOM

- **DOM操作を最小限に抑える:** 可能な場合はバッチ更新を行ってください。DOM の頻繁な変更はコストがかかります。
  - _アンチパターン:_ DOM をループで更新する代わりに、ドキュメントフラグメントを構築して一度だけ追加します。
- **仮想DOMフレームワーク:** React、Vue などを効率的に使用し、不要な再レンダリングを回避します。
  - _React の例:_ 不要なレンダリングを防ぐには、`React.memo`、`useMemo`、および `useCallback` を使用します。
- **リスト内のキー:** 仮想DOMの差分を容易にするため、リストでは常に安定したキーを使用してください。リストが静的でない限り、配列のインデックスをキーとして使用しないでください。
- **インラインスタイルを避ける:** インラインスタイルはレイアウトの乱れを引き起こす可能性があります。CSSクラスの使用を推奨します。
- **CSSアニメーション:** よりスムーズな GPU アクセラレーション効果を得るには、JavaScript ではなく CSS トランジション/アニメーションを使用します。
- **重要でないレンダリングを延期する:** ブラウザがアイドル状態になるまで作業を延期するには、`requestIdleCallback` などを使用します。

### Asset Optimization

- **Image Compression:** Use tools like ImageOptim, Squoosh, or TinyPNG. Prefer modern formats (WebP, AVIF) for web delivery.
- **SVGs for Icons:** SVGs scale well and are often smaller than PNGs for simple graphics.
- **Minification and Bundling:** Use Webpack, Rollup, or esbuild to bundle and minify JS/CSS. Enable tree-shaking to remove dead code.
- **Cache Headers:** Set long-lived cache headers for static assets. Use cache busting for updates.
- **Lazy Loading:** Use `loading="lazy"` for images, and dynamic imports for JS modules/components.
- **Font Optimization:** Use only the character sets you need. Subset fonts and use `font-display: swap`.

### Network Optimization

- **Reduce HTTP Requests:** Combine files, use image sprites, and inline critical CSS.
- **HTTP/2 and HTTP/3:** Enable these protocols for multiplexing and lower latency.
- **Client-Side Caching:** Use Service Workers, IndexedDB, and localStorage for offline and repeat visits.
- **CDNs:** Serve static assets from a CDN close to your users. Use multiple CDNs for redundancy.
- **Defer/Async Scripts:** Use `defer` or `async` for non-critical JS to avoid blocking rendering.
- **Preload and Prefetch:** Use `<link rel="preload">` and `<link rel="prefetch">` for critical resources.

### JavaScript Performance

- **Avoid Blocking the Main Thread:** Offload heavy computation to Web Workers.
- **Debounce/Throttle Events:** For scroll, resize, and input events, use debounce/throttle to limit handler frequency.
- **Memory Leaks:** Clean up event listeners, intervals, and DOM references. Use browser dev tools to check for detached nodes.
- **Efficient Data Structures:** Use Maps/Sets for lookups, TypedArrays for numeric data.
- **Avoid Global Variables:** Globals can cause memory leaks and unpredictable performance.
- **Avoid Deep Object Cloning:** Use shallow copies or libraries like lodash's `cloneDeep` only when necessary.

### Accessibility and Performance

- **Accessible Components:** Ensure ARIA updates are not excessive. Use semantic HTML for both accessibility and performance.
- **Screen Reader Performance:** Avoid rapid DOM updates that can overwhelm assistive tech.

### Framework-Specific Tips

#### React

- Use `React.memo`, `useMemo`, and `useCallback` to avoid unnecessary renders.
- Split large components and use code-splitting (`React.lazy`, `Suspense`).
- Avoid anonymous functions in render; they create new references on every render.
- Use `ErrorBoundary` to catch and handle errors gracefully.
- Profile with React DevTools Profiler.

#### Angular

- Use OnPush change detection for components that don't need frequent updates.
- Avoid complex expressions in templates; move logic to the component class.
- Use `trackBy` in `ngFor` for efficient list rendering.
- Lazy load modules and components with the Angular Router.
- Profile with Angular DevTools.

#### Vue

- Use computed properties over methods in templates for caching.
- Use `v-show` vs `v-if` appropriately (`v-show` is better for toggling visibility frequently).
- Lazy load components and routes with Vue Router.
- Profile with Vue Devtools.

### Common Frontend Pitfalls

- Loading large JS bundles on initial page load.
- Not compressing images or using outdated formats.
- Failing to clean up event listeners, causing memory leaks.
- Overusing third-party libraries for simple tasks.
- Ignoring mobile performance (test on real devices!).

### Frontend Troubleshooting

- Use Chrome DevTools' Performance tab to record and analyze slow frames.
- Use Lighthouse to audit performance and get actionable suggestions.
- Use WebPageTest for real-world load testing.
- Monitor Core Web Vitals (LCP, FID, CLS) for user-centric metrics.

---

## Backend Performance

### Algorithm and Data Structure Optimization

- **Choose the Right Data Structure:** Arrays for sequential access, hash maps for fast lookups, trees for hierarchical data, etc.
- **Efficient Algorithms:** Use binary search, quicksort, or hash-based algorithms where appropriate.
- **Avoid O(n^2) or Worse:** Profile nested loops and recursive calls. Refactor to reduce complexity.
- **Batch Processing:** Process data in batches to reduce overhead (e.g., bulk database inserts).
- **Streaming:** Use streaming APIs for large data sets to avoid loading everything into memory.

### Concurrency and Parallelism

- **Asynchronous I/O:** Use async/await, callbacks, or event loops to avoid blocking threads.
- **Thread/Worker Pools:** Use pools to manage concurrency and avoid resource exhaustion.
- **Avoid Race Conditions:** Use locks, semaphores, or atomic operations where needed.
- **Bulk Operations:** Batch network/database calls to reduce round trips.
- **Backpressure:** Implement backpressure in queues and pipelines to avoid overload.

### Caching

- **Cache Expensive Computations:** Use in-memory caches (Redis, Memcached) for hot data.
- **Cache Invalidation:** Use time-based (TTL), event-based, or manual invalidation. Stale cache is worse than no cache.
- **Distributed Caching:** For multi-server setups, use distributed caches and be aware of consistency issues.
- **Cache Stampede Protection:** Use locks or request coalescing to prevent thundering herd problems.
- **Don't Cache Everything:** Some data is too volatile or sensitive to cache.

### API and Network

- **Minimize Payloads:** Use JSON, compress responses (gzip, Brotli), and avoid sending unnecessary data.
- **Pagination:** Always paginate large result sets. Use cursors for real-time data.
- **Rate Limiting:** Protect APIs from abuse and overload.
- **Connection Pooling:** Reuse connections for databases and external services.
- **Protocol Choice:** Use HTTP/2, gRPC, or WebSockets for high-throughput, low-latency communication.

### Logging and Monitoring

- **Minimize Logging in Hot Paths:** Excessive logging can slow down critical code.
- **Structured Logging:** Use JSON or key-value logs for easier parsing and analysis.
- **Monitor Everything:** Latency, throughput, error rates, resource usage. Use Prometheus, Grafana, Datadog, or similar.
- **Alerting:** Set up alerts for performance regressions and resource exhaustion.

### Language/Framework-Specific Tips

#### Node.js

- Use asynchronous APIs; avoid blocking the event loop (e.g., never use `fs.readFileSync` in production).
- Use clustering or worker threads for CPU-bound tasks.
- Limit concurrent open connections to avoid resource exhaustion.
- Use streams for large file or network data processing.
- Profile with `clinic.js`, `node --inspect`, or Chrome DevTools.

#### Python

- Use built-in data structures (`dict`, `set`, `deque`) for speed.
- Profile with `cProfile`, `line_profiler`, or `Py-Spy`.
- Use `multiprocessing` or `asyncio` for parallelism.
- Avoid GIL bottlenecks in CPU-bound code; use C extensions or subprocesses.
- Use `lru_cache` for memoization.

#### Java

- Use efficient collections (`ArrayList`, `HashMap`, etc.).
- Profile with VisualVM, JProfiler, or YourKit.
- Use thread pools (`Executors`) for concurrency.
- Tune JVM options for heap and garbage collection (`-Xmx`, `-Xms`, `-XX:+UseG1GC`).
- Use `CompletableFuture` for async programming.

#### .NET

- Use `async/await` for I/O-bound operations.
- Use `Span<T>` and `Memory<T>` for efficient memory access.
- Profile with dotTrace, Visual Studio Profiler, or PerfView.
- Pool objects and connections where appropriate.
- Use `IAsyncEnumerable<T>` for streaming data.

### Common Backend Pitfalls

- Synchronous/blocking I/O in web servers.
- Not using connection pooling for databases.
- Over-caching or caching sensitive/volatile data.
- Ignoring error handling in async code.
- Not monitoring or alerting on performance regressions.

### Backend Troubleshooting

- Use flame graphs to visualize CPU usage.
- Use distributed tracing (OpenTelemetry, Jaeger, Zipkin) to track request latency across services.
- Use heap dumps and memory profilers to find leaks.
- Log slow queries and API calls for analysis.

---

## Database Performance

### Query Optimization

- **Indexes:** Use indexes on columns that are frequently queried, filtered, or joined. Monitor index usage and drop unused indexes.
- **Avoid SELECT \*:** Select only the columns you need. Reduces I/O and memory usage.
- **Parameterized Queries:** Prevent SQL injection and improve plan caching.
- **Query Plans:** Analyze and optimize query execution plans. Use `EXPLAIN` in SQL databases.
- **Avoid N+1 Queries:** Use joins or batch queries to avoid repeated queries in loops.
- **Limit Result Sets:** Use `LIMIT`/`OFFSET` or cursors for large tables.

### Schema Design

- **Normalization:** Normalize to reduce redundancy, but denormalize for read-heavy workloads if needed.
- **Data Types:** Use the most efficient data types and set appropriate constraints.
- **Partitioning:** Partition large tables for scalability and manageability.
- **Archiving:** Regularly archive or purge old data to keep tables small and fast.
- **Foreign Keys:** Use them for data integrity, but be aware of performance trade-offs in high-write scenarios.

### Transactions

- **Short Transactions:** Keep transactions as short as possible to reduce lock contention.
- **Isolation Levels:** Use the lowest isolation level that meets your consistency needs.
- **Avoid Long-Running Transactions:** They can block other operations and increase deadlocks.

### Caching and Replication

- **Read Replicas:** Use for scaling read-heavy workloads. Monitor replication lag.
- **Cache Query Results:** Use Redis or Memcached for frequently accessed queries.
- **Write-Through/Write-Behind:** Choose the right strategy for your consistency needs.
- **Sharding:** Distribute data across multiple servers for scalability.

### NoSQL Databases

- **Design for Access Patterns:** Model your data for the queries you need.
- **Avoid Hot Partitions:** Distribute writes/reads evenly.
- **Unbounded Growth:** Watch for unbounded arrays or documents.
- **Sharding and Replication:** Use for scalability and availability.
- **Consistency Models:** Understand eventual vs strong consistency and choose appropriately.

### Common Database Pitfalls

- Missing or unused indexes.
- SELECT \* in production queries.
- Not monitoring slow queries.
- Ignoring replication lag.
- Not archiving old data.

### Database Troubleshooting

- Use slow query logs to identify bottlenecks.
- Use `EXPLAIN` to analyze query plans.
- Monitor cache hit/miss ratios.
- Use database-specific monitoring tools (pg_stat_statements, MySQL Performance Schema).

---

## Code Review Checklist for Performance

- [ ] Are there any obvious algorithmic inefficiencies (O(n^2) or worse)?
- [ ] Are data structures appropriate for their use?
- [ ] Are there unnecessary computations or repeated work?
- [ ] Is caching used where appropriate, and is invalidation handled correctly?
- [ ] Are database queries optimized, indexed, and free of N+1 issues?
- [ ] Are large payloads paginated, streamed, or chunked?
- [ ] Are there any memory leaks or unbounded resource usage?
- [ ] Are network requests minimized, batched, and retried on failure?
- [ ] Are assets optimized, compressed, and served efficiently?
- [ ] Are there any blocking operations in hot paths?
- [ ] Is logging in hot paths minimized and structured?
- [ ] Are performance-critical code paths documented and tested?
- [ ] Are there automated tests or benchmarks for performance-sensitive code?
- [ ] Are there alerts for performance regressions?
- [ ] Are there any anti-patterns (e.g., SELECT \*, blocking I/O, global variables)?

---

## Advanced Topics

### Profiling and Benchmarking

- **Profilers:** Use language-specific profilers (Chrome DevTools, Py-Spy, VisualVM, dotTrace, etc.) to identify bottlenecks.
- **Microbenchmarks:** Write microbenchmarks for critical code paths. Use `benchmark.js`, `pytest-benchmark`, or JMH for Java.
- **A/B Testing:** Measure real-world impact of optimizations with A/B or canary releases.
- **Continuous Performance Testing:** Integrate performance tests into CI/CD. Use tools like k6, Gatling, or Locust.

### Memory Management

- **Resource Cleanup:** Always release resources (files, sockets, DB connections) promptly.
- **Object Pooling:** Use for frequently created/destroyed objects (e.g., DB connections, threads).
- **Heap Monitoring:** Monitor heap usage and garbage collection. Tune GC settings for your workload.
- **Memory Leaks:** Use leak detection tools (Valgrind, LeakCanary, Chrome DevTools).

### Scalability

- **Horizontal Scaling:** Design stateless services, use sharding/partitioning, and load balancers.
- **Auto-Scaling:** Use cloud auto-scaling groups and set sensible thresholds.
- **Bottleneck Analysis:** Identify and address single points of failure.
- **Distributed Systems:** Use idempotent operations, retries, and circuit breakers.

### Security and Performance

- **Efficient Crypto:** Use hardware-accelerated and well-maintained cryptographic libraries.
- **Validation:** Validate inputs efficiently; avoid regexes in hot paths.
- **Rate Limiting:** Protect against DoS without harming legitimate users.

### Mobile Performance

- **Startup Time:** Lazy load features, defer heavy work, and minimize initial bundle size.
- **Image/Asset Optimization:** Use responsive images and compress assets for mobile bandwidth.
- **Efficient Storage:** Use SQLite, Realm, or platform-optimized storage.
- **Profiling:** Use Android Profiler, Instruments (iOS), or Firebase Performance Monitoring.

### Cloud and Serverless

- **Cold Starts:** Minimize dependencies and keep functions warm.
- **Resource Allocation:** Tune memory/CPU for serverless functions.
- **Managed Services:** Use managed caching, queues, and DBs for scalability.
- **Cost Optimization:** Monitor and optimize for cloud cost as a performance metric.

---

## Practical Examples

### Example 1: Debouncing User Input in JavaScript

```javascript
// BAD: Triggers API call on every keystroke
input.addEventListener("input", (e) => {
  fetch(`/search?q=${e.target.value}`);
});

// GOOD: Debounce API calls
let timeout;
input.addEventListener("input", (e) => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    fetch(`/search?q=${e.target.value}`);
  }, 300);
});
```

### Example 2: Efficient SQL Query

```sql
-- BAD: Selects all columns and does not use an index
SELECT * FROM users WHERE email = 'user@example.com';

-- GOOD: Selects only needed columns and uses an index
SELECT id, name FROM users WHERE email = 'user@example.com';
```

### Example 3: Caching Expensive Computation in Python

```python
# BAD: Recomputes result every time
result = expensive_function(x)

# GOOD: Cache result
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_function(x):
    ...
result = expensive_function(x)
```

### Example 4: Lazy Loading Images in HTML

```html
<!-- BAD: Loads all images immediately -->
<img src="large-image.jpg" />

<!-- GOOD: Lazy loads images -->
<img src="large-image.jpg" loading="lazy" />
```

### Example 5: Asynchronous I/O in Node.js

```javascript
// BAD: Blocking file read
const data = fs.readFileSync("file.txt");

// GOOD: Non-blocking file read
fs.readFile("file.txt", (err, data) => {
  if (err) throw err;
  // process data
});
```

### Example 6: Profiling a Python Function

```python
import cProfile
import pstats

def slow_function():
    ...

cProfile.run('slow_function()', 'profile.stats')
p = pstats.Stats('profile.stats')
p.sort_stats('cumulative').print_stats(10)
```

### Example 7: Using Redis for Caching in Node.js

```javascript
const redis = require("redis");
const client = redis.createClient();

function getCachedData(key, fetchFunction) {
  return new Promise((resolve, reject) => {
    client.get(key, (err, data) => {
      if (data) return resolve(JSON.parse(data));
      fetchFunction().then((result) => {
        client.setex(key, 3600, JSON.stringify(result));
        resolve(result);
      });
    });
  });
}
```

---

## References and Further Reading

- [Google Web Fundamentals: Performance](https://web.dev/performance/)
- [MDN Web Docs: Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [OWASP: Performance Testing](https://owasp.org/www-project-performance-testing/)
- [Microsoft Performance Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/performance)
- [PostgreSQL Performance Optimization](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [MySQL Performance Tuning](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
- [Node.js Performance Best Practices](https://nodejs.org/en/docs/guides/simple-profiling/)
- [Python Performance Tips](https://docs.python.org/3/library/profile.html)
- [Java Performance Tuning](https://www.oracle.com/java/technologies/javase/performance.html)
- [.NET Performance Guide](https://learn.microsoft.com/en-us/dotnet/standard/performance/)
- [WebPageTest](https://www.webpagetest.org/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/)
- [k6 Load Testing](https://k6.io/)
- [Gatling](https://gatling.io/)
- [Locust](https://locust.io/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Jaeger](https://www.jaegertracing.io/)
- [Zipkin](https://zipkin.io/)

---

## Conclusion

Performance optimization is an ongoing process. Always measure, profile, and iterate. Use these best practices, checklists, and troubleshooting tips to guide your development and code reviews for high-performance, scalable, and efficient software. If you have new tips or lessons learned, add them here—let's keep this guide growing!

---

<!-- End of Performance Optimization Instructions -->
