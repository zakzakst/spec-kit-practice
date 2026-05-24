---
description: コードレビュー。ローカルの未コミット変更または GitHub PR を対象にします（PR モードでは PR 番号 / URL を渡します）
argument-hint: [pr-number | pr-url | blank for local review]
---

# コードレビュー

> PR レビューモードは Wirasm の PRPs-agentic-eng をもとにしたものです。PRP ワークフロー群の一部です。

**Input**: $ARGUMENTS

---

## モード選択

`$ARGUMENTS` に PR 番号、PR URL、または `--pr` が含まれている場合:
- 下の **PR Review Mode** へ進む

それ以外の場合:
- **Local Review Mode** を使う

---

## ローカルレビューモード

未コミット変更に対する包括的なセキュリティ / 品質レビューを行います。

### Phase 1 - 収集

```bash
git diff --name-only HEAD
```

変更ファイルがなければ停止: `"Nothing to review."`

### Phase 2 - レビュー

変更された各ファイルを全文読む。以下を確認する:

**Security Issues (CRITICAL):**

- ハードコードされた認証情報、API キー、トークン
- SQL インジェクション脆弱性
- XSS 脆弱性
- 入力検証不足
- 安全でない依存関係
- パストラバーサルの危険

**Code Quality (HIGH):**

- 関数が 50 行超
- ファイルが 800 行超
- ネスト深度が 4 段超
- エラーハンドリング不足
- `console.log` 文
- `TODO` / `FIXME` コメント
- 公開 API に JSDoc がない

**Best Practices (MEDIUM):**

- 変更可能パターン（可能なら immutable を使う）
- コード / コメント内の絵文字使用
- 新規コードに対するテスト不足
- アクセシビリティ問題（a11y）

### Phase 3 - レポート

以下を含むレポートを生成する:

- Severity: CRITICAL, HIGH, MEDIUM, LOW
- ファイル位置と行番号
- 問題の説明
- 修正案

CRITICAL または HIGH が見つかった場合はコミットをブロックする。
セキュリティ脆弱性のあるコードは決して承認しない。

---

## PR レビューモード

GitHub PR に対する包括的レビュー。差分取得、全文読解、検証実行、レビュー投稿まで行う。

### Phase 1 - 取得

入力から PR を特定する:

| Input                          | Action                                   |
| ------------------------------ | ---------------------------------------- |
| Number (e.g. `42`)             | そのまま PR 番号として使う               |
| URL (`github.com/.../pull/42`) | PR 番号を抽出する                        |
| Branch name                    | `gh pr list --head <branch>` で PR を探す |

```bash
gh pr view <NUMBER> --json number,title,body,author,baseRefName,headRefName,changedFiles,additions,deletions
gh pr diff <NUMBER>
```

PR が見つからなければエラーで停止する。後続フェーズ用に PR メタデータを保持する。

### Phase 2 - 文脈整理

レビュー文脈を構築する:

1. **Project rules** - `CLAUDE.md`、`.claude/docs/`、および contributing ガイドラインを読む
2. **Planning artifacts** - `.claude/prds/`、`.claude/plans/`、`.claude/reviews/`、および旧式の `.claude/PRPs/{prds,plans,reports,reviews}/` を確認し、この PR に関係する文脈を探す
3. **PR intent** - PR 説明から目的、関連 issue、テスト計画を読み取る
4. **Changed files** - 変更された全ファイルを列挙し、種別（source、test、config、docs）に分類する

### Phase 3 - レビュー

変更された各ファイルを **全文** 読む（差分ハンクだけではなく、その周辺文脈も必要）。

PR レビューでは、PR ヘッドのリビジョンにおける全文を取得する:

```bash
gh pr diff <NUMBER> --name-only | while IFS= read -r file; do
  gh api "repos/{owner}/{repo}/contents/$file?ref=<head-branch>" --jq '.content' | base64 -d
done
```

以下 7 カテゴリのチェックリストを適用する:

| Category               | What to Check                                                                 |
| ---------------------- | ----------------------------------------------------------------------------- |
| **Correctness**        | ロジックエラー、off-by-one、null 処理、エッジケース、レース条件              |
| **Type Safety**        | 型不一致、危険な cast、`any` の使用、ジェネリクス不足                        |
| **Pattern Compliance** | プロジェクト慣習への準拠（命名、構造、エラーハンドリング、import）          |
| **Security**           | インジェクション、認可漏れ、秘密情報露出、SSRF、パストラバーサル、XSS        |
| **Performance**        | N+1 クエリ、インデックス不足、無制限ループ、メモリリーク、大きな payload     |
| **Completeness**       | テスト不足、エラーハンドリング不足、未完成 migration、文書不足               |
| **Maintainability**    | デッドコード、マジックナンバー、深いネスト、不明瞭な命名、型不足             |

各指摘に重大度を割り当てる:

| Severity     | Meaning                                  | Action                  |
| ------------ | ---------------------------------------- | ----------------------- |
| **CRITICAL** | セキュリティ脆弱性またはデータ損失の危険 | マージ前に必ず修正      |
| **HIGH**     | 問題化しやすいバグまたはロジックエラー   | マージ前に修正推奨      |
| **MEDIUM**   | コード品質問題またはベストプラクティス不足 | 修正推奨              |
| **LOW**      | スタイル上の軽微な指摘                   | 任意                    |

### Phase 4 - 検証

利用可能な検証コマンドを実行する:

`package.json`、`Cargo.toml`、`go.mod`、`pyproject.toml` などの設定ファイルからプロジェクト種別を判定し、適切なコマンドを実行する。

**Node.js / TypeScript**（`package.json` がある場合）:

```bash
npm run typecheck 2>/dev/null || npx tsc --noEmit 2>/dev/null  # Type check
npm run lint                                                    # Lint
npm test                                                        # Tests
npm run build                                                   # Build
```

**Rust**（`Cargo.toml` がある場合）:

```bash
cargo clippy -- -D warnings  # Lint
cargo test                   # Tests
cargo build                  # Build
```

**Go**（`go.mod` がある場合）:

```bash
go vet ./...    # Lint
go test ./...   # Tests
go build ./...  # Build
```

**Python**（`pyproject.toml` / `setup.py` がある場合）:

```bash
pytest  # Tests
```

検出したプロジェクト種別に適用できるものだけを実行し、各項目の pass / fail を記録する。

### Phase 5 - 判定

指摘内容に応じて推奨判断を行う:

| Condition                                    | Decision                   |
| -------------------------------------------- | -------------------------- |
| CRITICAL / HIGH が 0 件で検証が通過         | **APPROVE**                |
| MEDIUM / LOW のみで検証も通過               | **APPROVE** with comments  |
| HIGH がある、または検証失敗がある           | **REQUEST CHANGES**        |
| CRITICAL がある                             | **BLOCK** - 修正必須       |

特殊ケース:

- Draft PR -> 常に **COMMENT** を使う（approve / block はしない）
- docs / config だけの変更 -> 軽めのレビューにし、正確性を中心に見る
- 明示的な `--approve` または `--request-changes` フラグ -> 判断自体は上書きするが、指摘はすべて報告する

### Phase 6 - レポート

この作業系が旧式の `.claude/PRPs/reviews/` を使っていない限り、`.claude/reviews/pr-<NUMBER>-review.md` にレビュー成果物を作成する:

```markdown
# PR Review: #<NUMBER> - <TITLE>

**Reviewed**: <date>
**Author**: <author>
**Branch**: <head> -> <base>
**Decision**: APPROVE | REQUEST CHANGES | BLOCK

## Summary

<全体評価を 1〜2 文で>

## Findings

### CRITICAL

<指摘、または "None">

### HIGH

<指摘、または "None">

### MEDIUM

<指摘、または "None">

### LOW

<指摘、または "None">

## Validation Results

| Check      | Result                |
| ---------- | --------------------- |
| Type check | Pass / Fail / Skipped |
| Lint       | Pass / Fail / Skipped |
| Tests      | Pass / Fail / Skipped |
| Build      | Pass / Fail / Skipped |

## Files Reviewed

<変更種別付きのファイル一覧: Added / Modified / Deleted>
```

### Phase 7 - 公開

レビューを GitHub へ投稿する:

```bash
# APPROVE の場合
gh pr review <NUMBER> --approve --body "<summary of review>"

# REQUEST CHANGES の場合
gh pr review <NUMBER> --request-changes --body "<summary with required fixes>"

# COMMENT only の場合（Draft PR または情報提供のみ）
gh pr review <NUMBER> --comment --body "<summary>"
```

特定行へのインラインコメントには GitHub review comments API を使う:

```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/comments" \
  -f body="<comment>" \
  -f path="<file>" \
  -F line=<line-number> \
  -f side="RIGHT" \
  -f commit_id="$(gh pr view <NUMBER> --json headRefOid --jq .headRefOid)"
```

または複数のインラインコメントを一括投稿する:

```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/reviews" \
  -f event="COMMENT" \
  -f body="<overall summary>" \
  --input comments.json  # [{"path": "file", "line": N, "body": "comment"}, ...]
```

### Phase 8 - 出力

ユーザーへの報告:

```text
PR #<NUMBER>: <TITLE>
Decision: <APPROVE|REQUEST_CHANGES|BLOCK>

Issues: <critical_count> critical, <high_count> high, <medium_count> medium, <low_count> low
Validation: <pass_count>/<total_count> checks passed

Artifacts:
  Review: .claude/reviews/pr-<NUMBER>-review.md
  GitHub: <PR URL>

Next steps:
  - <decision に応じた提案>
```

---

## エッジケース

- **`gh` CLI がない**: ローカルのみのレビューへフォールバックする（差分を読んで GitHub 投稿は省略）。ユーザーへ警告する。
- **ブランチが乖離している**: レビュー前に `git fetch origin && git rebase origin/<base>` を提案する。
- **大規模 PR（50 ファイル超）**: レビュー範囲を警告し、まず source、次に test、その後 config / docs を優先して確認する。
