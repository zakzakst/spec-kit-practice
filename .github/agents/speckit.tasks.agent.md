---
description: 利用可能な設計成果物に基づき、実行可能で依存順に並んだ tasks.md を生成する。
handoffs:
  - label: Analyze For Consistency
    agent: speckit.analyze
    prompt: Run a project analysis for consistency
    send: true
  - label: Implement Project
    agent: speckit.implement
    prompt: Start the implementation in phases
    send: true
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## Outline

1. **Setup**: リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json` を実行し、`FEATURE_DIR` と `AVAILABLE_DOCS` を取得する。パスは絶対パスにする。シングルクォート入り引数は必要に応じてエスケープする。

2. **Load design documents**: `FEATURE_DIR` から次を読む。
   - **Required**: `plan.md`（技術スタック、ライブラリ、構造）、`spec.md`（優先度付き user stories）
   - **Optional**: `data-model.md`, `contracts/`, `research.md`, `quickstart.md`
   - すべての文書があるとは限らない。存在するものに基づいてタスクを生成する

3. **Execute task generation workflow**
   - `plan.md` から tech stack、libraries、project structure を抽出
   - `spec.md` から user stories と優先度（P1, P2, P3...）を抽出
   - `data-model.md` があれば entities を抽出して story に対応付け
   - `contracts/` があれば endpoints を story に対応付け
   - `research.md` があれば setup task に反映
   - ユーザーストーリー単位でタスクを生成
   - user story 完了順を示す dependency graph を作る
   - story ごとに並列実行例を作る
   - 各 story が独立にテスト可能な粒度になっているか検証する

4. **Generate tasks.md**: `.specify/templates/tasks-template.md` を骨組みとして使い、次を埋める。
   - `plan.md` から取得した正しい機能名
   - Phase 1: Setup
   - Phase 2: Foundational
   - Phase 3+: user story ごとの phase（優先順）
   - 各 phase に goal、独立テスト基準、必要なら tests、implementation tasks
   - Final Phase: Polish & cross-cutting concerns
   - 全タスクが厳密なチェックリスト形式であること
   - 明確な file paths
   - story 完了順を示す dependencies
   - story ごとの parallel execution examples
   - MVP first の implementation strategy

5. **Report**: 生成した `tasks.md` のパスと次を要約する。
   - Total task count
   - Task count per user story
   - Parallel opportunities
   - Independent test criteria per story
   - Suggested MVP scope（通常は User Story 1）
   - 形式検証結果（checkbox, ID, labels, file paths）

Context for task generation: $ARGUMENTS

`tasks.md` は、そのまま LLM が実行できる粒度で具体的にすること。

## Task Generation Rules

**CRITICAL**: タスクはユーザーストーリー単位で整理し、独立実装・独立テストを可能にすること。

**Tests are OPTIONAL**: feature spec またはユーザーが明示要求した場合のみテストタスクを生成する。

### Checklist Format (REQUIRED)

すべてのタスクは次の形式に**厳密に**従うこと。

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**Format Components**

1. **Checkbox**: 常に `- [ ]` で始める
2. **Task ID**: 実行順の連番 `T001`, `T002`, ...
3. **[P] marker**: 並列可能な場合のみ付ける
4. **[Story] label**: user story phase のみ必須
   - `[US1]`, `[US2]`, `[US3]` 形式
   - Setup / Foundational / Polish には付けない
5. **Description**: 明確な動作 + 正確な file path

**Examples**

- 正: `- [ ] T001 Create project structure per implementation plan`
- 正: `- [ ] T005 [P] Implement authentication middleware in src/middleware/auth.py`
- 正: `- [ ] T012 [P] [US1] Create User model in src/models/user.py`
- 正: `- [ ] T014 [US1] Implement UserService in src/services/user_service.py`
- 誤: `- [ ] Create User model`
- 誤: `T001 [US1] Create model`
- 誤: `- [ ] [US1] Create User model`
- 誤: `- [ ] T001 [US1] Create model`

### Task Organization

1. **From User Stories (spec.md)** - 主軸
   - 各 user story（P1, P2, P3...）ごとに phase を作る
   - 関連する models / services / endpoints / UI / tests をその story に紐付ける
   - story 依存を明示する

2. **From Contracts**
   - 各 contract / endpoint を対応する story へ紐付ける
   - テストが必要なら、その story phase で contract test task `[P]` を実装前に置く

3. **From Data Model**
   - 各 entity を必要とする story へ紐付ける
   - 複数 story 共通なら、最も早い story か Setup phase に置く
   - relationships は適切な story の service layer task に落とし込む

4. **From Setup / Infrastructure**
   - Shared infrastructure -> Setup phase
   - Foundational / blocking tasks -> Foundational phase
   - Story-specific setup -> 各 story phase 内

### Phase Structure

- **Phase 1**: Setup
- **Phase 2**: Foundational（すべての story より前に完了必須）
- **Phase 3+**: 優先順の User Stories
  - 各 story 内の流れ: Tests（必要なら）-> Models -> Services -> Endpoints -> Integration
  - 各 phase は独立テスト可能な増分であること
- **Final Phase**: Polish & Cross-Cutting Concerns
