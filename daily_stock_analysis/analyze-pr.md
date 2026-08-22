# PR 分析

GitHub Pull Request を分析し、必要性、説明の完全性、検証エビデンス、主なリスク、およびそのままマージできるかを評価します。

**Repository**: https://github.com/ZhuLinsen/daily_stock_analysis/pulls

## 使用方法

```text
/analyze-pr <pr_number>
```

## 手順

分析時は簡潔な日本語を使用し、リポジトリのルートにある `AGENTS.md` と `.github/PULL_REQUEST_TEMPLATE.md` を優先して遵守してください。

### Step 1: 最新のコードベースを同期

PR の分析前に、必ずリモートの状態を更新し、可能な限りローカルを安全に最新のベースラインへ進めてください。

```bash
git status --short
git fetch --all --prune
# 仅当工作区干净且当前分支可 fast-forward 时执行：
git pull --ff-only
```

- ワークツリーがクリーンで、現在のブランチに fast-forward 可能な上流ブランチがある場合のみ、`git pull --ff-only` を実行し、その結果を受け入れてください。
- ローカルの変更、競合状態、未追跡のリスクファイル、上流ブランチの不在、または fast-forward 不可のいずれかに該当する場合は、`stash`、`reset`、強制的なブランチ切り替え、ローカル状態の上書きを行わないでください。fetch 済みの `origin/main`、PR head、または GitHub diff を使用して分析してください。
- 出力文書の `Validation Evidence` に同期結果として、ローカル HEAD、使用したリモートのベースライン、ローカルのワークツリーを更新しなかった理由（該当する場合）を記録してください。

### Step 2: PR の基本情報を取得

```bash
gh pr view <pr_number> --repo ZhuLinsen/daily_stock_analysis
gh pr view <pr_number> --repo ZhuLinsen/daily_stock_analysis --comments
gh pr checks <pr_number> --repo ZhuLinsen/daily_stock_analysis
gh pr diff <pr_number> --repo ZhuLinsen/daily_stock_analysis
```

CI に失敗がある場合は、すぐにローカルですべてのチェックを再実行するのではなく、まず失敗ログを確認してください。

```bash
gh run view <run_id> --log-failed
```

### Step 3: タイトルと説明の完全性を確認

まず PR title が `AGENTS.md` の非ブロッキングの推奨事項に従っているか確認してください。

- 形式は `<種類>: <変更内容>` とし、例として `fix: 大規模分析の履歴が失われる問題を修正` のようにしてください。
- 種類は `fix`/`feat`/`refactor`/`docs`/`chore`/`test`/`ci` を優先してください。
- `[codex]`、`codex`、`autocode`、`copilot`、その他のツールや agent の出所を示すプレフィックスは含めないでください。
- タイトルは実際の変更内容を説明する必要があります。タイトルと diff が一致しない場合は説明の完全性で指摘しますが、それだけを理由にレビューを妨げてはいけません。

`.github/PULL_REQUEST_TEMPLATE.md` と照合し、次の項目が含まれているか確認してください。

- `PR Type`
- `Background And Problem`
- `Scope Of Change`
- `Issue Link`
- `Verification Commands And Results`
- `Visual Evidence`（PR がレポート形式、レポートのレンダリング結果、または Web UI を変更する場合のみ、スクリーンショットまたは代替となる視覚的エビデンスを要求）
- `Compatibility And Risk`
- `Rollback Plan`

PR がサードパーティモデル / API の互換性に関する仕様、リクエストパラメーターの固定値、OpenAI-compatible ルーティング、YAML alias、フォールバック動作、または実行時設定の保存 / クリーンアップ / 移行ロジックに関係する場合は、説明に次の事項が明記されているか追加で確認してください。

- 公式の情報源リンクまたは告知
- 現在固定されている依存関係 / 実行時の互換性範囲（例：LiteLLM のバージョン範囲）
- 検証済みの呼び出し経路のカバレッジ
- 旧設定が暗黙的に書き換えられる、消去される、移行される、または変更されないか
- 最小限のロールバック方法（通常はこの PR の revert）

PR がレポート形式、レポートのレンダリング結果、または Web UI を変更する場合は、`Visual Evidence` に影響を受けるレポート / ページのスクリーンショットが添付されているかも確認してください。変更前後の差異がある場合は、前後比較を優先して確認します。スクリーンショットを用意できない場合は、説明に理由と代替となる視覚的エビデンスを記載する必要があります。

### Step 4: CI / Diff のエビデンスを優先

- まず `gh pr checks`、PR diff、既存のテスト、ワークフローログに基づいて問題を判断してください。
- CI が変更範囲をカバーしていない場合、CI の結果だけでは問題を確定できない場合、または重要なリグレッションリスクを検証する必要がある場合に限り、ローカルで最小限の検証を追加してください。
- デフォルトで現在のブランチを切り替えたり、`gh pr checkout` を実行したりしないでください。

ローカル検証の追加が必要な場合は、変更範囲に最も近いチェックを選択してください。例：

- 后端：`./scripts/ci_gate.sh` 或 `python -m py_compile <changed_python_files>`
- 前端：`cd apps/dsa-web && npm ci && npm run lint && npm run build`
- 桌面端：先构建 Web，再构建 Electron

### Step 5: 正確性とリスクを評価

重点的に確認する項目：

- 明確な問題を解決しており、無関係な変更が含まれていないか
- API / Schema / Web / Desktop の互換性を壊していないか
- フォールバック、縮退経路、通知経路、またはリリース手順を壊していないか
- 明らかなロジックエラー、例外の握りつぶし、セキュリティ上の問題、設定の意味の変更がドキュメントに反映されていない問題がないか

### Step 6: レビュー文書を作成

`.claude/reviews/prs/pr-<number>.md` に保存してください。

## 出力文書の形式

```markdown
# PR #<number> Analysis

**日付**: YYYY-MM-DD
**ステータス**: レビュー待ち

## 指摘事項

- [重大度] file:line - 問題の説明

## 概要

- 必要性：
- 対応する Issue の有無：
- PR の種類：
- PR title：
- description の完全性：
- 検証状況：
- 主なリスク：
- そのままマージできるか：

## 検証エビデンス

- コード同期のベースライン：
- CI の結論：
- ローカルで追加した検証（該当する場合）：

## 互換性とリスク

- API / Web / Desktop：
- 設定 / Docker / GitHub Actions：
- フォールバック / 通知 / レポート構造：
- サードパーティ依存関係 / 公式制約の情報源：
- 実行時の互換性範囲 / カバー済みの経路：
- 旧設定の移行または暗黙的な書き換えのリスク：

## レビューコメント案

<コメント内容の案>
```

## Allowed Auto-Actions (No Confirmation Needed)

- 拉取 PR 元数据、diff、评论和 CI 状态
- 执行 `git fetch --all --prune`，并在工作区干净且可 fast-forward 时执行 `git pull --ff-only`
- 阅读相关代码、模板、工作流与文档
- 在必要时执行最小化本地验证
- 生成评审文档

## Actions Requiring Confirmation

执行以下动作前，先询问用户：

1. 发布评论
2. Approve PR
3. Request changes
4. Merge PR
5. 关闭 PR