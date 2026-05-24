---
name: verification-loop
description: "Claude Code セッション向けの包括的な検証システム。"
origin: ECC
---

# Verification Loop スキル

Claude Code セッション向けの包括的な検証システムです。

## 使うタイミング

このスキルを使うのは次のとき:

- 機能実装や大きなコード変更を終えた後
- PR を作成する前
- 品質ゲートを通ることを確認したいとき
- リファクタリング後

## 検証フェーズ

### Phase 1: Build Verification

```bash
# プロジェクトがビルドできるか確認
npm run build 2>&1 | tail -20
# OR
pnpm build 2>&1 | tail -20
```

ビルドに失敗したら、先へ進まず修正する。

### Phase 2: Type Check

```bash
# TypeScript projects
npx tsc --noEmit 2>&1 | head -30

# Python projects
pyright . 2>&1 | head -30
```

型エラーはすべて報告する。重要なものは続行前に修正する。

### Phase 3: Lint Check

```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### Phase 4: Test Suite

```bash
# カバレッジ付きでテスト実行
npm run test -- --coverage 2>&1 | tail -50

# カバレッジ閾値確認
# 目標: 最低 80%
```

報告項目:

- Total tests: X
- Passed: X
- Failed: X
- Coverage: X%

### Phase 5: Security Scan

```bash
# シークレット検出
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# console.log 検出
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### Phase 6: Diff Review

```bash
# 何が変わったか確認
git diff --stat
git diff HEAD~1 --name-only
```

変更ファイルごとに確認すること:

- 意図しない変更
- 不足しているエラーハンドリング
- 潜在的なエッジケース

## 出力形式

すべてのフェーズを実行した後、以下のような検証レポートを出す:

```text
VERIFICATION REPORT
==================

Build:     [PASS/FAIL]
Types:     [PASS/FAIL] (X errors)
Lint:      [PASS/FAIL] (X warnings)
Tests:     [PASS/FAIL] (X/Y passed, Z% coverage)
Security:  [PASS/FAIL] (X issues)
Diff:      [X files changed]

Overall:   [READY/NOT READY] for PR

Issues to Fix:
1. ...
2. ...
```

## Continuous Mode

長いセッションでは、15 分ごと、または大きな変更のたびに検証する:

```markdown
頭の中でチェックポイントを置く:

- 各関数を終えた後
- コンポーネントを終えた後
- 次のタスクへ移る前

Run: /verify
```

## Hooks との統合

このスキルは PostToolUse hook を補完するが、それより深い検証を提供する。
hook は即時検知向きであり、このスキルは包括レビュー向きである。
