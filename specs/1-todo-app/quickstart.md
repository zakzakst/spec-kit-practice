# Quickstart for シンプルなTodoアプリ

These steps initialize and run the todo web application locally.

## Prerequisites

- Node.js 18+ installed (or compatible LTS)
- npm or yarn available in PATH

## Setup

1. **Clone the repository** (if not already):

   ```bash
   git clone <repo-url> spec-kit-practice
   cd spec-kit-practice
   ```

2. **Ensure you are on the feature branch**:

   ```bash
   git checkout -b 1-todo-app
   ```

3. **Install dependencies** (project root will contain frontend package):
   ```bash
   cd frontend
   npm install
   # or yarn install
   ```

## Running in Development

Launch the Vite development server:

```bash
npm run dev
# or yarn dev
```

Open `http://localhost:5173` (port may vary) to view the app. Changes will hot-reload.

## Testing

- **Unit / component tests**:

  ```bash
  npm run test:unit
  ```

- **End-to-end / integration tests**:
  ```bash
  npm run test:e2e
  ```

## Building for Production

```bash
npm run build
```

The production-ready static files will be emitted under `frontend/dist`.

## Notes

- Todo items are saved in browser `localStorage`; clearing the storage will reset the list.
- No backend service is required. The app can be deployed as static assets to any web host.
