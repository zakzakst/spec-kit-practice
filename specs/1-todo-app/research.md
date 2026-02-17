# Research for シンプルなTodoアプリ

## Decision: Frontend-only React application with localStorage persistence

**Rationale**:

- The feature description specifies a simple web app; no mention of user accounts or server state.
- A single-page application (SPA) written in React can deliver the required interactions quickly and keeps complexity low.
- Using `localStorage` satisfies the persistence requirement (FR-006) without needing a backend, simplifying development and deployment.
- React ecosystems (Vite, Create React App, Next.js) have well-known toolchains, good testing support, and are appropriate for a small project.

**Alternatives considered**:

- **Vanilla JS/HTML/CSS**: This would minimize dependencies but offers less structure and harder scaling even for a tiny app. For this exercise, React provides a clearer data-driven model.
- **Backend-backed application (Node/Express, etc.)**: Introducing a server would satisfy persistence but adds unnecessary infrastructure for a local todo list. Could be revisited if requirements expand.
- **Use a wrapper like Electron or mobile**: out of scope; spec targets web.

## Decision: TypeScript for type safety

**Rationale**:

- TypeScript catches common errors at compile time and improves developer experience when modelling entities like Todo Item.
- The overhead is small for a new project and aligns with modern frontend best practices.

**Alternatives considered**:

- Plain JavaScript: quicker setup but lacks type guarantees. For a simple app TS is still beneficial.

## Decision: Testing with Vitest (unit) and Playwright (integration)

**Rationale**:

- Vitest pairs naturally with Vite for unit/component tests.
- Playwright provides cross-browser automation for end-to-end tests, allowing independent verification of acceptance scenarios.

**Alternatives considered**:

- Jest for testing: widely used but would require additional configuration alongside Vite.
- Cypress for e2e: powerful, but Playwright is also lightweight and supports headless automation.

## Decision: Project scaffolding with Vite

**Rationale**:

- Vite offers fast build times, simple configuration, and official templates for React + TypeScript.

**Alternatives considered**:

- Create React App: stable but slower dev server, more boilerplate.

## Storage Mechanism: Browser `localStorage`

**Rationale**:

- Requirement FR-006 only states persistence across reloads/sessions; localStorage fulfills this without server dependency.
- Data volume is small, so storage quotas aren't an issue.

**Alternatives considered**:

- IndexedDB: overkill for simple objects.
- Server API: complexity not justified.

## Summary

The app will be a single React+TypeScript SPA bootstrapped with Vite, persisting todo items in localStorage. Testing will use Vitest and Playwright to cover unit and end-to-end scenarios.
