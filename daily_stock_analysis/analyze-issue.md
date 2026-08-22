# Issue 分析

GitHub Issue を分析し、その妥当性、優先度、リポジトリの責任範囲、および推奨アクションを判断します。

**Repository**: https://github.com/ZhuLinsen/daily_stock_analysis/issues

## 使用方法

```text
/analyze-issue <issue_number>
```

## 手順

分析時は簡潔な日本語を使用し、リポジトリのルートにある `AGENTS.md` を優先して遵守してください。

### Step 1: 最新のコードベースを同期

Issue の分析前に、必ずリモートの状態を更新し、可能な限りローカルを安全に最新のベースラインへ進めてください。

```bash
git status --short
git fetch --all --prune
# 仅当工作区干净且当前分支可 fast-forward 时执行：
git pull --ff-only
```

- ワークツリーがクリーンで、現在のブランチに fast-forward 可能な上流ブランチがある場合のみ、`git pull --ff-only` を実行し、その結果を受け入れてください。
- ローカルの変更、競合状態、未追跡のリスクファイル、上流ブランチの不在、または fast-forward 不可のいずれかに該当する場合は、`stash`、`reset`、強制的なブランチ切り替え、ローカル状態の上書きを行わないでください。fetch 済みの `origin/main` または関連するリモート refs を使用して分析してください。
- 出力文書の `Evidence` に同期結果として、ローカル HEAD、使用したリモートのベースライン、ローカルのワークツリーを更新しなかった理由（該当する場合）を記録してください。

### Step 2: Issue 情報を取得

```bash
gh issue view <issue_number> --repo ZhuLinsen/daily_stock_analysis
gh issue view <issue_number> --repo ZhuLinsen/daily_stock_analysis --comments
```

バグの場合は、Issue テンプレートに次の情報が記載されているかを優先的に確認してください。

- 最新バージョンに同期済みか
- commit hash / バージョンのベースライン
- 実行環境と再現手順
- ログまたはエラー情報

### Step 3: 4 つの主要な問いに回答

1. バージョンが明確か
2. 問題が実在し、検証可能か
3. リポジトリの責任範囲に含まれるか
4. 直ちに対応する価値があるか

### Step 4: リポジトリの現状に基づく証拠確認

- 関連するコード、設定、テスト、スクリプト、ワークフロー、ドキュメントを確認してください。
- 問題が API、データソースのフォールバック、レポート生成、通知送信、認証、デスクトップ版、リリース手順に関係する場合は、影響範囲を明確に記載してください。
- 実際のバグ、環境設定の問題、使用方法の問題、外部依存関係の問題のいずれに該当するかを判断してください。
- すでに修正されている可能性がある場合は、Issue の説明だけでなく現在のコードを確認してください。

### Step 5: 結論をまとめる

少なくとも次の項目を記載してください。

- `バージョンのベースライン`：最新 / 最新ではない / 未提供
- `妥当性`：はい/いいえ + 理由
- `Issue に該当するか`：はい/いいえ + 理由
- `解決しやすいか`：はい/いいえ + 難所
- `結論`：`成立 / 部分的に成立 / 不成立`
- `分類`：`bug / feature / docs / question / external`
- `優先度`：`P0 / P1 / P2 / P3`
- `難易度`：`easy / medium / hard`
- `推奨アクション`：`直ちに修正 / 対応予定に入れて修正 / ドキュメントを明確化 / クローズ`

### Step 6: 分析文書を作成

`.claude/reviews/issues/issue-<number>.md` に保存してください。

## 出力文書の形式

```markdown
# Issue #<number> Analysis

**日付**: YYYY-MM-DD
**ステータス**: レビュー待ち

## 概要

- バージョンのベースライン：
- 妥当性：
- Issue に該当するか：
- 解決しやすいか：
- 結論：
- 分類：
- 優先度：
- 難易度：
- 推奨アクション：

## 根拠

- コード同期のベースライン：
- Issue の主要情報：
- 主要なコード/スクリプト/ワークフローの根拠：

## 影響範囲

- 影響を受けるモジュール：
- 影響を受ける実行経路（ローカル / Docker / GitHub Actions / API / Web / Desktop）：

## 根本原因 / 主な判断理由

<根本原因または主な判断根拠>

## 対応案

<修正、明確化、またはクローズの方法を提案>

後続で PR を作成することを提案する場合、PR title は `AGENTS.md` に従い、`<種類>: <変更内容>` の形式にしてください。`[codex]`、`codex`、`autocode`、`copilot`、その他のツールや agent の出所を示すプレフィックスは付けないでください。この規約は協業上の一貫性を保つための注意事項であり、それだけを理由にレビューを妨げてはいけません。

## リスクとロールバック

- リスク：
- 修正した場合のロールバック方法：

## 返信案

<返信内容の案>
```

## Allowed Auto-Actions (No Confirmation Needed)

- 拉取 issue 详情与评论
- 执行 `git fetch --all --prune`，并在工作区干净且可 fast-forward 时执行 `git pull --ff-only`
- 阅读相关代码、配置、脚本、工作流和文档
- 生成分析文档

## Actions Requiring Confirmation

执行以下动作前，先询问用户：

1. 添加或修改标签
2. 在 issue 下评论
3. 关闭 issue
4. 开始修复 issue