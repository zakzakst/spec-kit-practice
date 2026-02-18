# Tasks: シンプルなTodoアプリ

**入力**: `/specs/1-todo-app/` からの設計ドキュメント
**前提条件**: plan.md, spec.md, research.md, data-model.md, contracts/

## フェーズ 1: セットアップ（共有インフラストラクチャ）

**目的**: プロジェクトの初期化と基本構造

- [ ] T001 `frontend/` ディレクトリを作成し、React+TypeScript Vite プロジェクトをブートストラップします (`npm create vite@latest frontend --template react-ts` を実行)
- [ ] T002 `frontend/package.json` に依存関係を追加します: `react`、`react-dom`、`uuid`、`vitest`、`@testing-library/react`、`playwright` および関連タイプのパッケージ
- [ ] T003 [P] `frontend/.eslintrc.js` と `frontend/.prettierrc` で linting と formatting を設定する
- [ ] T004 [P] Git ignore、TypeScript 設定 (`tsconfig.json`)、および `frontend/` 下の基本的なプロジェクトメタデータを初期化します。

---

## フェーズ 2: 基礎（ブロックの前提条件）

**目的**: あらゆるユーザーストーリーを実装する前に完了する必要があるコアインフラストラクチャ

- [ ] T005 [P] `id`、`title`、`completed`、`createdAt` を持つ `TodoItem` インターフェースを定義する `frontend/src/types/todo.ts` を作成します。
- [ ] T006 [P] ToDo の get/save メソッドを提供するストレージヘルパー `frontend/src/hooks/useLocalStorage.ts` を実装する
- [ ] T007 [P] ベースコンポーネントスケルトン `frontend/src/App.tsx` レンダリングプレースホルダーUIを作成する
- [ ] T008 [P] テスト構成ファイルを追加する:
  - `frontend/vitest.config.ts`
  - `frontend/playwright.config.ts`
- [ ] T009 [P] 基本スタイル `frontend/src/index.css` を追加し、 `frontend/src/main.tsx` にインポートします。

**チェックポイント**: 基盤の準備完了 - ユーザーストーリーの実装を並行して開始できます

---

## フェーズ 3: ユーザーストーリー 1 - タスクを追加する (優先度: P1) 🎯 MVP

**ゴール**: ユーザーがタイトルを入力して新しい ToDo 項目を保存し、リストに表示されるようにします。

**独立テスト**: 空のブラウザ状態から開始し、UI を通じてタスクを追加し、それがリストに表示され、リロード後も維持されることを確認します。

### ユーザーストーリー1のテスト

- [ ] T010 [P] [US1] `frontend/tests/unit/add-todo.spec.tsx` に Vitest ユニット/コンポーネントテストを記述し、タイトル付きのフォームを送信すると状態にアイテムが追加されることを確認します。

### ユーザーストーリー1の実装

- [ ] T011 [P] [US1] 制御された入力と送信ハンドラを持つ `frontend/src/components/AddTodo.tsx` を作成します。
- [ ] T012 [P] [US1] props経由で渡されたTodoの配列をレンダリングする `frontend/src/components/TodoList.tsx` を作成します。
- [ ] T013 [US1] `useLocalStorage` フックを使用して `frontend/src/App.tsx` に add-todo ロジックと状態管理を実装する
- [ ] T014 [US1] `AddTodo.tsx` に検証を追加して、タイトルが空にならないようにし、エラーメッセージを表示します。

**チェックポイント**: ユーザーストーリー1は完全に機能し、独立してテスト可能である必要があります。

---

## フェーズ 4: ユーザーストーリー 2 - タスクを完了としてマークする (優先度: P2)

**ゴール**: ToDo 項目の完了状態を切り替え、その変更を視覚的およびストレージに反映するコントロールを提供します。

**独立テスト**: 少なくとも 1 つの未完了のタスクがある場合は、その完了状態を切り替えて、UI が更新され、リロード後も変更が維持されることを確認します。

### ユーザーストーリー2のテスト

- [ ] T015 [P] [US2] チェックボックスを切り替えると、`useLocalStorage` を介して ToDo の `completed` フィールドが更新されることを確認するユニット テスト `frontend/tests/unit/toggle-complete.spec.tsx` を記述します。

### ユーザーストーリー2の実装

- [ ] T016 [P] [US2] `frontend/src/types/todo.ts` の `TodoItem` インターフェースに `completed` ブール値が含まれていることを確認します (foundational で既に定義されています)
- [ ] T017 [P] [US2] `frontend/src/components/TodoItem.tsx` (このコンポーネントを作成) にチェックボックスとラベルを追加して、完了の表示と切り替えを行います。
- [ ] T018 [US2] `frontend/src/App.tsx` に `toggleTodo` ハンドラーを実装し、`completed` フラグを反転してフック経由で保存します。
- [ ] T019 [US2] `TodoList.tsx` を更新して `TodoItem` コンポーネントを使用し、トグル ハンドラーを伝播します。

**チェックポイント**: ユーザーストーリー1と2はどちらも独立して機能する必要がある

---

## フェーズ 5: ユーザーストーリー 3 - タスクを削除する (優先度: P3)

**ゴール**: ユーザーが削除コントロールを使用してリストから ToDo 項目を削除できるようにします。

**独立テスト**: タスクが存在する場合は、削除ボタンを使用して、リロード後にアイテムが削除され、存在しないことを確認します。

### ユーザーストーリー3のテスト

- [ ] T020 [P] [US3] 削除アクションによってアイテムがストレージから削除されることを確認するユニットテスト `frontend/tests/unit/delete-todo.spec.tsx` を追加します。

### ユーザーストーリー3の実装

- [ ] T021 [P] [US3] `frontend/src/components/TodoItem.tsx` に削除ボタン/アイコンを追加します。
- [ ] T022 [US3] `frontend/src/App.tsx` に、アイテムを削除してストレージを更新する `deleteTodo` ハンドラーを実装します。
- [ ] T023 [US3] `TodoList.tsx` / `TodoItem.tsx` を更新して、呼び出されたときに削除ハンドラを呼び出すようにします。

**チェックポイント**: すべてのユーザーストーリーは独立して機能するようになった

---

## フェーズ N: 研磨と横断的な懸念

**目的**: 複数のユーザーストーリーに影響する改善

- [ ] T024 [P] `frontend/README.md` と `specs/1-todo-app/quickstart.md` の README とドキュメントを更新します。
- [ ] T025 [P] クリーンクローンで手順に従って `quickstart.md` 検証を実行します。
- [ ] T026 `frontend/src` コンポーネントとフック全体のコードのクリーンアップとリファクタリング
- [ ] T027 [P] `frontend/tests/e2e/` の下に、追加、切り替え、削除フローをカバーするエンドツーエンドの Playwright テストを追加します。

---

## 依存関係と実行順序

### フェーズの依存関係

- **Setup (Phase 1)**: 依存関係なし – すぐに開始できます
- **Foundational (Phase 2)**: セットアップの完了に依存 – すべてのユーザーストーリーをブロックします
- **User Stories (Phase 3+)**: すべては基礎フェーズの完了に依存し、その後は並行して実行できます。
- **Polish (Final Phase)**: 必要なユーザーストーリーがすべて完了していることが条件

### ユーザーストーリーの依存関係

- **User Story 1 (P1)**: フェーズ 2 の後に開始します。他のストーリーに依存しません。
- **User Story 2 (P2)**: フェーズ2の後に開始します。US1コンポーネントと統合できますが、独立してテスト可能です。
- **User Story 3 (P3)**: フェーズ2の後に始まります。以前のストーリーとは独立しています。

### 各ユーザーストーリー内

- 実装前にテストを書いて失敗する必要がある
- 統合前のコンポーネント/フック
- UI 強化前のコアストーリーロジック

### 並行した機会

- セットアップタスク（T003、T004）は並列実行できます
- 基礎タスク（T005～T009）はすべて並列化可能
- 基盤が完成すれば、ユーザーストーリーは同時に作業できる
- ストーリー内のテストは並列化可能
- 異なるストーリーを別々のチームメンバーが並行して進めることができる

---

## 並列例: ユーザーストーリー 1

```bash
# 基礎工事が完了するまで
cd frontend
# 開発者AはAddTodoコンポーネントに取り組んでいます
# 開発者Bがユニットテストを書く
# 開発者Cがアプリの状態処理を更新
```

_上記は、プロジェクトの骨組みが存在すると重複する可能性がある独立したワークストリームを示しています。_
