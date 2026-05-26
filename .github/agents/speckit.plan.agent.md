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

## Outline

1. **Setup**: リポジトリルートで `.specify/scripts/powershell/setup-plan.ps1 -Json` を実行し、`FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH` を解析する。シングルクォートを含む引数は必要に応じてエスケープすること。

2. **Load context**: `FEATURE_SPEC` と `.specify/memory/constitution.md` を読み、コピー済みの `IMPL_PLAN` テンプレートを読み込む。

3. **Execute plan workflow**: `IMPL_PLAN` の構造に従って次を行う。
   - Technical Context を埋める（不明点は `NEEDS CLARIFICATION`）
   - constitution から Constitution Check を埋める
   - ゲートを評価する（正当化のない違反は ERROR）
   - Phase 0: `research.md` を生成し、全 `NEEDS CLARIFICATION` を解消する
   - Phase 1: `data-model.md`, `contracts/`, `quickstart.md` を生成する
   - Phase 1: agent script を実行して agent context を更新する
   - 設計後に Constitution Check を再評価する

4. **Stop and report**: Phase 2 の planning 完了で終了し、branch、`IMPL_PLAN` パス、生成成果物を報告する。

## Phases

### Phase 0: Outline & Research

1. Technical Context から不明点を抽出する。
   - 各 `NEEDS CLARIFICATION` -> research task
   - 各 dependency -> best practices task
   - 各 integration -> patterns task

2. 調査エージェントを生成 / 発行する。

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. 結果を `research.md` に統合する。形式:
   - Decision
   - Rationale
   - Alternatives considered

**Output**: `research.md`。すべての `NEEDS CLARIFICATION` が解消されていること。

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` が完成していること

1. 機能 spec からエンティティを抽出し、`data-model.md` を作成する。
   - Entity name
   - fields
   - relationships
   - validation rules
   - state transitions（必要なら）

2. 機能要件から API contracts を生成する。
   - 各 user action -> endpoint
   - 標準的な REST / GraphQL パターンを使う
   - 出力先は `/contracts/`

3. **Agent context update**
   - `.specify/scripts/powershell/update-agent-context.ps1 -AgentType copilot` を実行
   - 実行中の AI agent をスクリプト側で検出する
   - 適切な agent-specific context ファイルを更新する
   - 現在 plan で追加された技術のみを追記する
   - マーカー間の手動追記は保持する

**Output**: `data-model.md`, `/contracts/*`, `quickstart.md`, agent-specific file

## Key rules

- 絶対パスを使う
- gate failure や未解決 clarification は ERROR にする
