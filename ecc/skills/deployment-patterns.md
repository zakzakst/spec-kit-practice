---
name: deployment-patterns
description: Web アプリケーション向けのデプロイワークフロー、CI/CD パイプラインパターン、Docker コンテナ化、ヘルスチェック、ロールバック戦略、本番準備チェックリスト。
origin: ECC
---

# デプロイパターン

本番デプロイのワークフローと CI/CD のベストプラクティスです。

## 有効化するタイミング

- CI/CD パイプラインを組むとき
- アプリケーションを Docker 化するとき
- デプロイ戦略（blue-green、canary、rolling）を決めるとき
- ヘルスチェックや readiness probe を実装するとき
- 本番リリースの準備をするとき
- 環境別設定を整えるとき

## デプロイ戦略

### Rolling Deployment（標準）

インスタンスを段階的に置き換える。ロールアウト中は旧版と新版が同時に動く。

```text
Instance 1: v1 -> v2  (update first)
Instance 2: v1        (still running v1)
Instance 3: v1        (still running v1)

Instance 1: v2
Instance 2: v1 -> v2  (update second)
Instance 3: v1

Instance 1: v2
Instance 2: v2
Instance 3: v1 -> v2  (update last)
```

**Pros:** ダウンタイムなし、段階的ロールアウト
**Cons:** 2 バージョンが同時稼働するため後方互換が必要
**Use when:** 標準的なデプロイ、後方互換がある変更

### Blue-Green Deployment

同一の環境を 2 つ持ち、トラフィックを原子的に切り替える。

```text
Blue  (v1) -> traffic
Green (v2)   idle, running new version

# After verification:
Blue  (v1)   idle (becomes standby)
Green (v2) -> traffic
```

**Pros:** 即時ロールバック可能、切り替えが明快
**Cons:** デプロイ中は 2 倍のインフラが必要
**Use when:** 重要サービス、問題許容度が低いとき

### Canary Deployment

まず少量のトラフィックだけを新バージョンへ流す。

```text
v1: 95% of traffic
v2:  5% of traffic  (canary)

# If metrics look good:
v1: 50% of traffic
v2: 50% of traffic

# Final:
v2: 100% of traffic
```

**Pros:** 本番トラフィックで問題を早期検知できる
**Cons:** トラフィック分割基盤と監視が必要
**Use when:** 高トラフィック、リスクの高い変更、feature flag 利用時

## Docker

### Multi-Stage Dockerfile（Node.js）

```dockerfile
# Stage 1: Install dependencies
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --production=false

# Stage 2: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build
RUN npm prune --production

# Stage 3: Production image
FROM node:22-alpine AS runner
WORKDIR /app

RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001
USER appuser

COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/package.json ./

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

### Multi-Stage Dockerfile（Go）

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /server ./cmd/server

FROM alpine:3.19 AS runner
RUN apk --no-cache add ca-certificates
RUN adduser -D -u 1001 appuser
USER appuser

COPY --from=builder /server /server

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:8080/health || exit 1
CMD ["/server"]
```

### Multi-Stage Dockerfile（Python/Django）

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
RUN pip install --no-cache-dir uv
COPY requirements.txt .
RUN uv pip install --system --no-cache -r requirements.txt

FROM python:3.12-slim AS runner
WORKDIR /app

RUN useradd -r -u 1001 appuser
USER appuser

COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY . .

ENV PYTHONUNBUFFERED=1
EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health/')" || exit 1
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

### Docker ベストプラクティス

```text
# GOOD practices
- 具体的な version tag を使う（node:22-alpine, not node:latest）
- image を小さくする multi-stage build
- non-root user で実行
- dependency file を先に copy（layer cache）
- .dockerignore で node_modules, .git, tests を除外
- HEALTHCHECK を入れる
- docker-compose / k8s で resource limit を設定

# BAD practices
- root 実行
- :latest tag の使用
- repo 全体を 1 回の COPY で突っ込む
- production image に dev dependency を入れる
- image 内へ secret を埋め込む
```

## CI/CD パイプライン

### GitHub Actions（標準パイプライン）

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: coverage/

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - name: Deploy to production
        run: |
          # Platform-specific deployment command
          # Railway: railway up
          # Vercel: vercel --prod
          # K8s: kubectl set image deployment/app app=ghcr.io/${{ github.repository }}:${{ github.sha }}
          echo "Deploying ${{ github.sha }}"
```

### Pipeline Stages

```text
PR opened:
  lint -> typecheck -> unit tests -> integration tests -> preview deploy

Merged to main:
  lint -> typecheck -> unit tests -> integration tests -> build image -> deploy staging -> smoke tests -> deploy production
```

## Health Checks

### Health Check Endpoint

```typescript
// Simple health check
app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok" });
});

// Detailed health check (for internal monitoring)
app.get("/health/detailed", async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalApi: await checkExternalApi(),
  };

  const allHealthy = Object.values(checks).every((c) => c.status === "ok");

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? "ok" : "degraded",
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || "unknown",
    uptime: process.uptime(),
    checks,
  });
});
```

### Kubernetes Probes

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 30
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 2

startupProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 0
  periodSeconds: 5
  failureThreshold: 30
```

## 環境設定

### Twelve-Factor App パターン

```bash
# すべての設定を environment variable で渡す。コードに埋め込まない
DATABASE_URL=postgres://user:pass@host:5432/db
REDIS_URL=redis://host:6379/0
API_KEY=${API_KEY}
LOG_LEVEL=info
PORT=3000

# Environment-specific behavior
NODE_ENV=production
APP_ENV=production
```

### 設定値のバリデーション

```typescript
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "staging", "production"]),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
});

export const env = envSchema.parse(process.env);
```

## ロールバック戦略

### Instant Rollback

```bash
kubectl rollout undo deployment/app
vercel rollback
railway up --commit <previous-sha>
npx prisma migrate resolve --rolled-back <migration-name>
```

### Rollback Checklist

- [ ] 以前の image / artifact が残っていて tag もある
- [ ] DB migration が後方互換を保っている
- [ ] feature flag だけで新機能を止められる
- [ ] error rate spike 向けの監視アラートがある
- [ ] 本番前に staging で rollback を試した

## 本番準備チェックリスト

### Application

- [ ] unit / integration / E2E を含め、すべての test が通る
- [ ] コードや設定に hardcoded secret がない
- [ ] エラーハンドリングが edge case を網羅している
- [ ] logging が構造化され、PII を含まない
- [ ] health check endpoint が意味ある状態を返す

### Infrastructure

- [ ] Docker image が再現可能に build できる
- [ ] 環境変数が文書化され、起動時に検証される
- [ ] CPU / memory の resource limit がある
- [ ] 水平スケール設定がある
- [ ] 全 endpoint で SSL/TLS が有効

### Monitoring

- [ ] request rate、latency、error などの指標を出している
- [ ] error rate 閾値超えの alert がある
- [ ] 構造化ログの集約と検索ができる
- [ ] health endpoint の uptime monitoring がある

### Security

- [ ] 依存関係の CVE スキャンがある
- [ ] CORS が許可 origin のみに限定されている
- [ ] 公開 endpoint に rate limiting がある
- [ ] 認証 / 認可が検証されている
- [ ] CSP, HSTS, X-Frame-Options などの security header がある

### Operations

- [ ] rollback plan が文書化され、試されている
- [ ] production 相当データ量で migration を試した
- [ ] よくある障害シナリオ向け runbook がある
- [ ] on-call rotation と escalation path が定義されている
