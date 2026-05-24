---
name: eval-harness
description: Claude Code セッション向けの正式な評価フレームワーク。eval-driven development (EDD) の原則を実装します
origin: ECC
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Eval Harness スキル

Claude Code セッション向けの正式な評価フレームワークであり、eval-driven development（EDD）の原則を実装します。

## 有効化するタイミング

- AI 支援ワークフローに EDD を導入するとき
- Claude Code タスク完了の pass / fail 基準を定義するとき
- pass@k 指標で agent の信頼性を測るとき
- prompt や agent 変更向けの回帰テスト群を作るとき
- モデルバージョン間で agent 性能を比較するとき

## 哲学

Eval-Driven Development は、eval を「AI 開発における unit test」として扱う:

- 実装前に期待動作を定義する
- 開発中も継続的に eval を回す
- 変更ごとに回帰を追跡する
- 信頼性測定には pass@k を使う

## Eval の種類

### Capability Evals

Claude が以前できなかったことをできるか確認する:

```markdown
[CAPABILITY EVAL: feature-name]
Task: Description of what Claude should accomplish
Success Criteria:

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3
      Expected Output: Description of expected result
```

### Regression Evals

変更で既存機能が壊れていないことを確認する:

```markdown
[REGRESSION EVAL: feature-name]
Baseline: SHA or checkpoint name
Tests:

- existing-test-1: PASS/FAIL
- existing-test-2: PASS/FAIL
- existing-test-3: PASS/FAIL
  Result: X/Y passed (previously Y/Y)
```

## Grader の種類

### 1. Code-Based Grader

コードで決定論的に確認する:

```bash
# Check if file contains expected pattern
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# Check if tests pass
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# Check if build succeeds
npm run build && echo "PASS" || echo "FAIL"
```

### 2. Model-Based Grader

オープンエンドな出力を Claude に評価させる:

```markdown
[MODEL GRADER PROMPT]
Evaluate the following code change:

1. Does it solve the stated problem?
2. Is it well-structured?
3. Are edge cases handled?
4. Is error handling appropriate?

Score: 1-5 (1=poor, 5=excellent)
Reasoning: [explanation]
```

### 3. Human Grader

手動レビューが必要なものとしてフラグを立てる:

```markdown
[HUMAN REVIEW REQUIRED]
Change: Description of what changed
Reason: Why human review is needed
Risk Level: LOW/MEDIUM/HIGH
```

## 指標

### pass@k

「k 回の試行で少なくとも 1 回成功する」

- pass@1: 初回成功率
- pass@3: 3 回以内に成功する率
- 一般的な目標: pass@3 > 90%

### pass^k

「k 回すべて成功する」

- 信頼性に対するより高い基準
- pass^3: 3 回連続成功
- 重要経路で使う

## Eval ワークフロー

### 1. Define（コーディング前）

```markdown
## EVAL DEFINITION: feature-xyz

### Capability Evals

1. Can create new user account
2. Can validate email format
3. Can hash password securely

### Regression Evals

1. Existing login still works
2. Session management unchanged
3. Logout flow intact

### Success Metrics

- pass@3 > 90% for capability evals
- pass^3 = 100% for regression evals
```

### 2. Implement

定義した eval を通すためのコードを書く。

### 3. Evaluate

```bash
# Run capability evals
[Run each capability eval, record PASS/FAIL]

# Run regression evals
npm test -- --testPathPattern="existing"

# Generate report
```

### 4. Report

```markdown
# EVAL REPORT: feature-xyz

Capability Evals:
create-user: PASS (pass@1)
validate-email: PASS (pass@2)
hash-password: PASS (pass@1)
Overall: 3/3 passed

Regression Evals:
login-flow: PASS
session-mgmt: PASS
logout-flow: PASS
Overall: 3/3 passed

Metrics:
pass@1: 67% (2/3)
pass@3: 100% (3/3)

Status: READY FOR REVIEW
```

## 統合パターン

### 実装前

```text
/eval define feature-name
```

`.claude/evals/feature-name.md` に eval 定義ファイルを作る。

### 実装中

```text
/eval check feature-name
```

現在の eval を実行して状態を報告する。

### 実装後

```text
/eval report feature-name
```

完全な eval レポートを生成する。

## Eval 保存場所

プロジェクト内に保存する:

```text
.claude/
  evals/
    feature-xyz.md      # Eval definition
    feature-xyz.log     # Eval run history
    baseline.json       # Regression baselines
```

## ベストプラクティス

1. **コーディング前に eval を定義する** - 成功条件の思考が明確になる
2. **頻繁に eval を回す** - 回帰を早期検知できる
3. **pass@k を時系列で追う** - 信頼性のトレンドが見える
4. **可能なら code grader を使う** - 決定論的な方が強い
5. **セキュリティは human review** - 完全自動化しない
6. **eval は速く保つ** - 遅い eval は回されなくなる
7. **コードと一緒に version 管理する** - eval も第一級の成果物

## 例: 認証機能を追加する

```markdown
## EVAL: add-authentication

### Phase 1: Define (10 min)

Capability Evals:

- [ ] User can register with email/password
- [ ] User can login with valid credentials
- [ ] Invalid credentials rejected with proper error
- [ ] Sessions persist across page reloads
- [ ] Logout clears session

Regression Evals:

- [ ] Public routes still accessible
- [ ] API responses unchanged
- [ ] Database schema compatible

### Phase 2: Implement (varies)

[Write code]

### Phase 3: Evaluate

Run: /eval check add-authentication

### Phase 4: Report

# EVAL REPORT: add-authentication

Capability: 5/5 passed (pass@3: 100%)
Regression: 3/3 passed (pass^3: 100%)
Status: SHIP IT
```

## Product Evals（v1.8）

unit test だけでは捉えられない振る舞い品質には product eval を使う。

### Grader Types

1. Code grader（決定論的アサーション）
2. Rule grader（regex / schema 制約）
3. Model grader（LLM-as-judge rubric）
4. Human grader（曖昧な出力の手動裁定）

### pass@k Guidance

- `pass@1`: 直接的な信頼性
- `pass@3`: 制御された retry を含む実用的な信頼性
- `pass^3`: 安定性試験（3 回すべて通る）

推奨閾値:

- Capability evals: pass@3 >= 0.90
- Regression evals: release-critical path では pass^3 = 1.00

### Eval Anti-Patterns

- 既知の eval 例に prompt を過剰適合させること
- happy path だけを測ること
- pass 率を追うあまりコストや遅延の悪化を無視すること
- flaky grader を release gate に入れること

### 最小 Eval Artifact Layout

- `.claude/evals/<feature>.md` definition
- `.claude/evals/<feature>.log` run history
- `docs/releases/<version>/eval-summary.md` release snapshot
