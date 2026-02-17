# Tasks: シンプルなTodoアプリ

**Input**: Design documents from `/specs/1-todo-app/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create `frontend/` directory and bootstrap a React+TypeScript Vite project (run `npm create vite@latest frontend --template react-ts`)
- [ ] T002 Add dependencies in `frontend/package.json`: `react`, `react-dom`, `uuid`, `vitest`, `@testing-library/react`, `playwright` and related type packages
- [ ] T003 [P] Configure linting and formatting in `frontend/.eslintrc.js` and `frontend/.prettierrc`
- [ ] T004 [P] Initialize Git ignore, TypeScript configuration (`tsconfig.json`), and basic project metadata under `frontend/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

- [ ] T005 [P] Create `frontend/src/types/todo.ts` defining `TodoItem` interface with `id`, `title`, `completed`, `createdAt`
- [ ] T006 [P] Implement storage helper `frontend/src/hooks/useLocalStorage.ts` providing get/save methods for todos
- [ ] T007 [P] Create base component skeleton `frontend/src/App.tsx` rendering placeholder UI
- [ ] T008 [P] Add test configuration files:
  - `frontend/vitest.config.ts`
  - `frontend/playwright.config.ts`
- [ ] T009 [P] Add base styles `frontend/src/index.css` and import in `frontend/src/main.tsx`

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Add a task (Priority: P1) 🎯 MVP

**Goal**: Allow users to enter a title and save a new todo item which then appears in the list.

**Independent Test**: Start with an empty browser state, add a task through the UI, and verify it appears in the list and persists after reload.

### Tests for User Story 1

- [ ] T010 [P] [US1] Write a Vitest unit/component test in `frontend/tests/unit/add-todo.spec.tsx` verifying that submitting the form with a title adds an item to state

### Implementation for User Story 1

- [ ] T011 [P] [US1] Create `frontend/src/components/AddTodo.tsx` with a controlled input and submit handler
- [ ] T012 [P] [US1] Create `frontend/src/components/TodoList.tsx` that renders an array of todos passed via props
- [ ] T013 [US1] Implement add‑todo logic and state management in `frontend/src/App.tsx` using `useLocalStorage` hook
- [ ] T014 [US1] Add validation in `AddTodo.tsx` to prevent empty titles and display an error message

**Checkpoint**: User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - Mark task as complete (Priority: P2)

**Goal**: Provide a control to toggle a todo item's completed state and reflect the change visually and in storage.

**Independent Test**: With at least one unfinished task, toggle its completion state and verify the UI updates and the change persists after reload.

### Tests for User Story 2

- [ ] T015 [P] [US2] Write a unit test `frontend/tests/unit/toggle-complete.spec.tsx` ensuring toggling the checkbox updates the todo's `completed` field via `useLocalStorage`

### Implementation for User Story 2

- [ ] T016 [P] [US2] Ensure `TodoItem` interface in `frontend/src/types/todo.ts` includes `completed` boolean (already defined in foundational)
- [ ] T017 [P] [US2] Add a checkbox and label in `frontend/src/components/TodoItem.tsx` (create this component) to display and toggle completion
- [ ] T018 [US2] Implement `toggleTodo` handler in `frontend/src/App.tsx` that flips the `completed` flag and saves via the hook
- [ ] T019 [US2] Update `TodoList.tsx` to use `TodoItem` component and propagate toggle handler

**Checkpoint**: User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Delete a task (Priority: P3)

**Goal**: Allow users to remove a todo item from the list via a delete control.

**Independent Test**: With a task present, use the delete button and verify the item is removed and not present after reload.

### Tests for User Story 3

- [ ] T020 [P] [US3] Add a unit test `frontend/tests/unit/delete-todo.spec.tsx` verifying that the delete action removes the item from storage

### Implementation for User Story 3

- [ ] T021 [P] [US3] Add a delete button/icon to `frontend/src/components/TodoItem.tsx`
- [ ] T022 [US3] Implement `deleteTodo` handler in `frontend/src/App.tsx` that removes the item and updates storage
- [ ] T023 [US3] Update `TodoList.tsx` / `TodoItem.tsx` to call the delete handler when invoked

**Checkpoint**: All user stories should now be independently functional

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T024 [P] Update README and documentation in `frontend/README.md` and `specs/1-todo-app/quickstart.md`
- [ ] T025 [P] Run `quickstart.md` validation by following its steps in a clean clone
- [ ] T026 Code cleanup and refactoring across `frontend/src` components and hooks
- [ ] T027 [P] Add end-to-end Playwright tests under `frontend/tests/e2e/` covering add, toggle, and delete flows

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies – can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion – BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion; can run in parallel thereafter
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Begins after Phase 2; no dependencies on other stories
- **User Story 2 (P2)**: Begins after Phase 2; may integrate with US1 components but is independently testable
- **User Story 3 (P3)**: Begins after Phase 2; independent of earlier stories

### Within Each User Story

- Tests must be written and fail before implementation
- Components/hooks before integration
- Core story logic before UI enhancements

### Parallel Opportunities

- Setup tasks (T003, T004) can run in parallel
- Foundational tasks (T005–T009) are all parallelizable
- User stories can be worked on concurrently once foundation is complete
- Tests within a story are parallelizable
- Different stories can proceed in parallel by separate team members

---

## Parallel Example: User Story 1

```bash
# while foundation completes
cd frontend
# Developer A works on AddTodo component
# Developer B writes the unit test
# Developer C updates App state handling
```

_The above illustrates independent workstreams that may overlap once the project skeleton exists._
