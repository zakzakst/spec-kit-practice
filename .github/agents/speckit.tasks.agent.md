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

## 概要 (Outline)

1. **セットアップ (Setup)**: リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json` を実行し、`FEATURE_DIR` と `AVAILABLE_DOCS` を取得する。パスは絶対パスにする。シングルクォート入り引数は必要に応じてエスケープする。

2. **設計ドキュメントの読み込み (Load design documents)**: `FEATURE_DIR` から次を読む。
   - **必須 (Required)**: `plan.md`（技術スタック、ライブラリ、構造）、`spec.md`（優先度付き ユーザーストーリー (user stories)）
   - **任意 (Optional)**: `data-model.md`, `contracts/`, `research.md`, `quickstart.md`
   - すべての文書があるとは限らない。存在するものに基づいてタスクを生成する。

3. **タスク生成ワークフローの実行 (Execute task generation workflow)**:
   - `plan.md` から技術スタック (tech stack)、ライブラリ (libraries)、プロジェクト構造 (project structure) を抽出
   - `spec.md` からユーザーストーリー (user stories) と優先度（P1, P2, P3...）を抽出
   - `data-model.md` があればエンティティ (entities) を抽出してストーリーに対応付け
   - `contracts/` があればエンドポイント (endpoints) をストーリーに対応付け
   - `research.md` があればセットアップタスク (setup task) に反映
   - ユーザーストーリー単位でタスクを生成
   - ユーザーストーリーの完了順を示す依存関係グラフ (dependency graph) を作成
   - ストーリーごとに並列実行の例を作成
   - 各ストーリーが独立にテスト可能な粒度になっているか検証する

4. **tasks.md の生成 (Generate tasks.md)**: `.specify/templates/tasks-template.md` を骨組みとして使い、次を埋める。
   - `plan.md` から取得した正しい機能名
   - フェーズ 1 (Phase 1): セットアップ (Setup)
   - フェーズ 2 (Phase 2): 基盤開発 (Foundational)
   - フェーズ 3+ (Phase 3+): ユーザーストーリーごとのフェーズ（優先順）
   - 各フェーズにゴール (goal)、独立テスト基準 (independent test criteria)、必要ならテスト (tests)、実装タスク (implementation tasks) を記述する
   - 最終フェーズ (Final Phase): 磨き込み & 横断的関心事 (Polish & cross-cutting concerns)
   - 全タスクが厳密なチェックリスト形式であること
   - 明確なファイルパス (file paths)
   - ストーリー完了順を示す依存関係 (dependencies)
   - ストーリーごとの並列実行の例 (parallel execution examples)
   - MVPファーストの導入戦略 (MVP first implementation strategy)

5. **レポート (Report)**: 生成した `tasks.md` のパスと次を要約する。
   - 総タスク数 (Total task count)
   - ユーザーストーリーごとのタスク数 (Task count per user story)
   - 並列処理可能なタスク (Parallel opportunities)
   - ストーリーごとの独立テスト基準 (Independent test criteria per story)
   - 推奨される MVP スコープ (Suggested MVP scope)（通常は ユーザーストーリー 1）
   - 形式検証結果（checkbox, ID, labels, file paths）

Context for task generation: $ARGUMENTS

`tasks.md` は、そのまま LLM が実行できる粒度で具体的にすること。

## タスク生成ルール (Task Generation Rules)

**CRITICAL**: タスクはユーザーストーリー単位で整理し、独立実装・独立テストを可能にすること。

**テストは任意 (Tests are OPTIONAL)**: feature spec（機能仕様）またはユーザーが明示要求した場合のみテストタスクを生成する。

### チェックリスト形式（必須） (Checklist Format - REQUIRED)

すべてのタスクは次の形式に**厳密に**従うこと。

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**フォーマット構成要素 (Format Components)**

1. **チェックボックス (Checkbox)**: 常に `- [ ]` で始める
2. **タスク ID (Task ID)**: 実行順の連番 `T001`, `T002`, ...
3. **並列マーカー [P] (parallel marker)**: 並列可能な場合のみ付ける
4. **[Story] ラベル (label)**: ユーザーストーリーフェーズ (user story phase) のみ必須
   - `[US1]`, `[US2]`, `[US3]` 形式
   - Setup / Foundational / Polish には付けない
5. **説明 (Description)**: 明確な動作 + 正確なファイルパス (file path)

**例 (Examples)**

- 正: `- [ ] T001 Create project structure per implementation plan`
- 正: `- [ ] T005 [P] Implement authentication middleware in src/middleware/auth.py`
- 正: `- [ ] T012 [P] [US1] Create User model in src/models/user.py`
- 正: `- [ ] T014 [US1] Implement UserService in src/services/user_service.py`
- 誤: `- [ ] Create User model`
- 誤: `T001 [US1] Create model`
- 誤: `- [ ] [US1] Create User model`
- 誤: `- [ ] T001 [US1] Create model`

### タスクの整理 (Task Organization)

1. **ユーザーストーリー (spec.md) 由来** - 主軸
   - 各ユーザーストーリー（P1, P2, P3...）ごとに phase（フェーズ）を作る
   - 関連する models / services / endpoints / UI / tests をそのストーリーに紐付ける
   - ストーリーの依存関係を明示する

2. **コントラクト (Contracts) 由来**
   - 各コントラクト / エンドポイントを対応するストーリーへ紐付ける
   - テストが必要なら、そのストーリーのフェーズでコントラクトテストタスク `[P]` を実装タスクの前に配置する

3. **データモデル (Data Model) 由来**
   - 各エンティティ (entity) を必要とするストーリーへ紐付ける
   - 複数ストーリーに共通する場合は、最も早いストーリーまたは Setup フェーズに置く
   - リレーションシップ (relationships) は適切なストーリーのサービスレイヤータスク (service layer task) に落とし込む

4. **セットアップ / インフラ (Setup / Infrastructure) 由来**
   - 共有インフラ (Shared infrastructure) -> Setup フェーズ
   - 基盤となるブロックタスク (Foundational / blocking tasks) -> Foundational フェーズ
   - ストーリー固有のセットアップ -> 各ストーリーのフェーズ内

### フェーズ構造 (Phase Structure)

- **フェーズ 1 (Phase 1)**: セットアップ (Setup)
- **フェーズ 2 (Phase 2)**: 基盤開発 (Foundational)（すべてのストーリーより前に完了必須）
- **フェーズ 3+ (Phase 3+)**: 優先順のユーザーストーリー (User Stories)
  - 各ストーリー内の流れ: テスト（必要なら）-> モデル -> サービス -> エンドポイント -> 統合
  - 各フェーズは独立してテスト可能な増分であること
- **最終フェーズ (Final Phase)**: 磨き込み & 横断的関心事 (Polish & Cross-Cutting Concerns)
