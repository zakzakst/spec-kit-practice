---
name: ci-cd-and-automation
description: CI/CD パイプラインの設定を自動化します。ビルドやデプロイのパイプラインを設定・変更するとき、品質ゲートを自動化するとき、CI でテストランナーを設定するとき、デプロイ戦略を整えるときに使います。
---

# CI/CD と自動化

## 概要

テスト、lint、型チェック、ビルドを通過した変更だけが本番に届くよう、品質ゲートを自動化します。CI/CD は他のすべての skill を実際に効かせるための強制機構です。人間やエージェントが見落とすものを、毎回一貫して検出します。

**Shift Left:** 問題はできるだけパイプラインの早い段階で見つけます。lint で見つかるバグは数分で済みますが、本番で見つかる同じバグは何時間もかかります。チェックを上流へ移動しましょう。静的解析をテストより前に、テストを staging より前に、staging を本番より前に置きます。

**速いほど安全:** 小さなバッチで頻繁にリリースするほうが、リスクは下がります。30 変更のデプロイより、3 変更のデプロイのほうがデバッグしやすいです。頻繁なリリースは、リリースプロセス自体への信頼も高めます。

## 使う場面

- 新しいプロジェクトの CI パイプラインを作るとき
- 自動チェックを追加・変更するとき
- デプロイパイプラインを設定するとき
- 変更をきっかけに自動検証を走らせたいとき
- CI の失敗を調べるとき

## 品質ゲートのパイプライン

すべての変更は、マージ前に次のゲートを通過します。

```
Pull Request Opened
    ↓
  LINT CHECK     eslint, prettier
  ↓ pass
  TYPE CHECK     tsc --noEmit
  ↓ pass
  UNIT TESTS     jest/vitest
  ↓ pass
  BUILD          npm run build
  ↓ pass
  INTEGRATION    API/DB tests
  ↓ pass
  E2E (optional) Playwright/Cypress
  ↓ pass
  SECURITY AUDIT npm audit
  ↓ pass
  BUNDLE SIZE    bundlesize check
    ↓
  Ready for review
```

**ゲートは 1 つも飛ばせません。** lint が失敗したら lint を直します。ルールを無効化してはいけません。テストが失敗したらコードを直します。テストを飛ばしてはいけません。

## GitHub Actions の設定

### 基本の CI パイプライン

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npx tsc --noEmit

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Security audit
        run: npm audit --audit-level=high
```

### データベース統合テスト付き

```yaml
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: ci_user
          POSTGRES_PASSWORD: ${{ secrets.CI_DB_PASSWORD }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
      - name: Integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
```

> **注:** CI 専用のテストデータベースであっても、認証情報はハードコードせず GitHub Secrets を使ってください。よい習慣になり、他の場面でテスト用認証情報を誤用するのを防げます。

### E2E テスト

```yaml
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Install Playwright
        run: npx playwright install --with-deps chromium
      - name: Build
        run: npm run build
      - name: Run E2E tests
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## CI 失敗をエージェントに戻す

AI エージェントと CI を組み合わせる強みは、フィードバックループにあります。CI が失敗したら次の流れで進めます。

```
CI fails
    ↓
失敗出力をコピーする
    ↓
エージェントに渡す:
"CI pipeline failed with this error:
[paste specific error]
Fix the issue and verify locally before pushing again."
    ↓
エージェントが修正 → push → CI が再実行される
```

**重要なパターン:**

```
Lint failure     → エージェントが `npm run lint --fix` を実行してコミットする
Type error       → エージェントがエラー箇所を読んで型を直す
Test failure     → エージェントが debugging-and-error-recovery skill に従う
Build error      → エージェントが設定と依存関係を確認する
```

## デプロイ戦略

### Preview デプロイ

すべての PR に、手動確認用の preview デプロイを用意します。

```yaml
# PR ごとの preview デプロイ（Vercel/Netlify など）
deploy-preview:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - name: Deploy preview
      run: npx vercel --token=${{ secrets.VERCEL_TOKEN }}
```

### Feature Flag

Feature flag はデプロイとリリースを分離します。未完成またはリスクの高い機能は flag の背後に置くことで、次のことができます。

- **コードだけ先に出す。** main に早めにマージし、準備ができたら有効化します。
- **再デプロイなしでロールバックする。** コードを戻す代わりに flag を無効にします。
- **新機能を canary で出す。** 1% のユーザー、その後 10%、その後 100% に広げます。
- **A/B テストを行う。** 機能あり・なしで挙動を比較します。

```typescript
// シンプルな feature flag パターン
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckout();
}
return renderLegacyCheckout();
```

**flag のライフサイクル:** 作成 → テスト向けに有効化 → canary → 全面展開 → flag と不要コードを削除する。ずっと残る flag は技術的負債になります。作成時に片付け日を決めてください。

### 段階的リリース

```
PR が main にマージされる
    ↓
  staging デプロイ（自動）
    ↓ 手動確認
  production デプロイ（手動トリガー、または staging 後に自動）
    ↓
  エラーを監視（15 分ウィンドウ）
    ↓
  エラー検出 → Rollback
  問題なし    → 完了
```

### ロールバック計画

すべてのデプロイは元に戻せる必要があります。

```yaml
# 手動ロールバックワークフロー
name: Rollback
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'ロールバック先のバージョン'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - name: Rollback deployment
        run: |
          # 指定された前のバージョンをデプロイする
          npx vercel rollback ${{ inputs.version }}
```

## 環境管理

```
.env.example       → コミットする（開発者向けテンプレート）
.env               → コミットしない（ローカル開発用）
.env.test          → コミットする（テスト環境、実秘密情報なし）
CI secrets         → GitHub Secrets / vault に保存
Production secrets → デプロイ先プラットフォーム / vault に保存
```

CI に本番秘密情報を持たせてはいけません。CI テスト用には別の secret を使ってください。

## CI 以外の自動化

### Dependabot / Renovate

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
```

### Build Cop の役割

CI を green に保つ担当者を決めておきます。ビルドが壊れたとき、Build Cop の仕事は直すか revert することです。壊した本人の担当ではありません。これにより、誰かが直すだろうと思って壊れたビルドが積み上がるのを防げます。

### PR チェック

- **必須レビュー:** マージ前に少なくとも 1 件の承認が必要
- **必須ステータスチェック:** マージ前に CI が通る必要がある
- **ブランチ保護:** main への force-push を禁止する
- **自動マージ:** すべてのチェックが通って承認済みなら、自動でマージする

## CI の最適化

パイプラインが 10 分を超える場合は、影響の大きい順に次の戦略を適用します。

```
CI パイプラインが遅い？
  ↓
依存関係をキャッシュする
  ↓   `actions/cache` か setup-node の cache オプションで node_modules を使う
ジョブを並列化する
  ↓   lint, typecheck, test, build を別々の並列ジョブに分割する
変更されたものだけを走らせる
  ↓   path filter を使って無関係なジョブを飛ばす（例: docs-only PR では e2e を飛ばす）
matrix build を使う
  ↓   テストスイートを複数 runner に分割する
テストスイートを最適化する
  ↓   重要経路から遅いテストを外し、スケジュール実行に回す
より大きい runner を使う
  ↓   CPU 重めのビルドには GitHub-hosted の大きい runner か self-hosted を使う
```

**例: キャッシュと並列化**

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npx tsc --noEmit

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage
```

## よくある言い訳

| 言い訳 | 現実 |
|---|---|
| 「CI が遅すぎる」 | スキップするのではなく、パイプラインを最適化してください（下の CI 最適化を参照）。5 分のパイプラインは、何時間ものデバッグを防ぎます。 |
| 「この変更は trivial だから CI は飛ばしていい」 | trivial な変更でも build は壊れます。しかも trivial な変更なら CI はすぐ終わります。 |
| 「テストが flaky だから再実行すればいい」 | flaky test は本当のバグを隠し、みんなの時間を無駄にします。flaky さを直してください。 |
| 「CI はあとで入れる」 | CI のないプロジェクトは壊れた状態が積み上がります。初日から入れてください。 |
| 「手動テストで十分」 | 手動テストはスケールせず、再現性もありません。できることは自動化してください。 |

## 危険信号

- プロジェクトに CI パイプラインがない
- CI の失敗が無視されている、または黙殺されている
- パイプラインを通すために CI 上でテストを無効化している
- staging 確認なしで本番デプロイしている
- ロールバック機構がない
- secret がコードや CI 設定ファイルに置かれている（secret manager ではない）
- CI 時間が長いのに最適化の努力がない

## 検証

CI を設定または変更したあとに、次を確認してください。

- [ ] すべての品質ゲートが存在する（lint、types、tests、build、audit）
- [ ] パイプラインがすべての PR と main への push で動く
- [ ] 失敗がマージをブロックする（ブランチ保護が設定されている）
- [ ] CI の結果が開発ループに戻ってくる
- [ ] secret はコードではなく secret manager に保存されている
- [ ] デプロイにロールバック機構がある
- [ ] テストスイートのパイプラインが 10 分未満で完了する
