---
description: plan テンプレートに従って設計成果物を生成し、実装計画ワークフローを実行する。
handoffs:
  - label: Create Tasks
    agent: speckit.tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: speckit.checklist
    prompt: Create a checklist for the following domain...
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## 概要 (Outline)

1. **セットアップ (Setup)**: リポジトリルートで `.specify/scripts/powershell/setup-plan.ps1 -Json` を実行し、`FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH` を解析する。シングルクォートを含む引数は必要に応じてエスケープすること。

2. **コンテキストの読み込み (Load context)**: `FEATURE_SPEC` と `.specify/memory/constitution.md` を読み、コピー済みの `IMPL_PLAN` テンプレートを読み込む。

3. **設計ワークフローの実行 (Execute plan workflow)**: `IMPL_PLAN` の構造に従って次を行う。
   - 技術的コンテキスト (Technical Context) を埋める（不明な点は `NEEDS CLARIFICATION` (要確認) とする）
   - constitution（憲章）から憲章チェック (Constitution Check) セクションを埋める
   - ゲート (Gate) を評価する（正当な理由のない違反は `ERROR` (エラー) とする）
   - フェーズ 0 (Phase 0): `research.md` を生成し、すべての `NEEDS CLARIFICATION` を解消する
   - フェーズ 1 (Phase 1): `data-model.md`, `contracts/`, `quickstart.md` を生成する
   - フェーズ 1 (Phase 1): エージェントスクリプトを実行してエージェントコンテキストを更新する
   - 設計完了後に憲章チェック (Constitution Check) を再評価する

4. **停止と報告 (Stop and report)**: フェーズ 2 (Phase 2) の設計 (planning) 完了時に処理を終了し、ブランチ名 (branch)、`IMPL_PLAN` パス、生成された成果物を報告する。

## フェーズ (Phases)

### フェーズ 0: 概要 & 調査 (Phase 0: Outline & Research)

1. 技術的コンテキスト (Technical Context) から不明な点を抽出する。
   - 各 `NEEDS CLARIFICATION` -> 調査タスク (research task)
   - 各 依存関係 (dependency) -> ベストプラクティスタスク (best practices task)
   - 各 外部連携 (integration) -> パターン調査タスク (patterns task)

2. 調査エージェントを生成 / 発行する。

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. 結果を `research.md` に統合する。形式は以下の通り:
   - 意思決定 (Decision)
   - 根拠 (Rationale)
   - 検討された代替案 (Alternatives considered)

**出力 (Output)**: `research.md`。すべての `NEEDS CLARIFICATION` が解消されていること。

### フェーズ 1: 設計 & コントラクト (Phase 1: Design & Contracts)

**前提条件 (Prerequisites):** `research.md` が完成していること

1. 機能仕様 (spec) からエンティティを抽出し、`data-model.md` を作成する。
   - エンティティ名 (Entity name)
   - フィールド (fields)
   - 関係性 (relationships)
   - 検証ルール (validation rules)
   - 状態遷移 (state transitions)（必要なら）

2. 機能要件から API コントラクト (API contracts) を生成する。
   - 各ユーザーアクション (user action) -> エンドポイント (endpoint)
   - 標準的な REST / GraphQL パターンを使う
   - 出力先は `/contracts/`

3. **エージェントコンテキストの更新 (Agent context update)**
   - `.specify/scripts/powershell/update-agent-context.ps1 -AgentType copilot` を実行
   - 実行中の AI エージェントをスクリプト側で検出する
   - 適切なエージェント固有のコンテキストファイル (agent-specific context file) を更新する
   - 現在設計 (plan) で追加された技術のみを追記する
   - マーカー間の手動追記内容は保持する

**出力 (Output)**: `data-model.md`, `/contracts/*`, `quickstart.md`, およびエージェント固有のコンテキストファイル

## 重要なルール (Key rules)

- 絶対パスを使うこと
- ゲート違反 (gate failure) や未解決の確認事項 (clarification) は `ERROR` (エラー) にすること
