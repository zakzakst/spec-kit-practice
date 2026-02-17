## 1. Setup

- [ ] 1.1 Create `frontend/src/components/LoginForm.tsx` component file
- [ ] 1.2 Add authentication utility `frontend/src/hooks/useAuth.ts` for token storage and state
- [ ] 1.3 Update `frontend/src/types/auth.ts` to include `AuthToken` type definition
- [ ] 1.4 Add route `/login` in application router configuration (`frontend/src/App.tsx` or equivalent)

## 2. Login Form and Validation

- [ ] 2.1 Implement JSX for username/email and password fields in `LoginForm.tsx`
- [ ] 2.2 Add client-side validation logic to `LoginForm.tsx` that prevents submission with empty fields and displays inline errors
- [ ] 2.3 Add submit button and loading indicator to `LoginForm.tsx`

## 3. Authentication Flow

- [ ] 3.1 Implement `login` function in `useAuth.ts` that posts credentials to `/api/login` and handles response
- [ ] 3.2 Store returned token in `localStorage` under key `authToken` within `useAuth.ts`
- [ ] 3.3 Update `useAuth` to provide `isAuthenticated`, `login`, and `logout` methods
- [ ] 3.4 Handle authentication errors in `LoginForm.tsx`, displaying a banner message on failure

## 4. Routing Protection

- [ ] 4.1 Create a `RequireAuth` higher-order component or hook that checks `isAuthenticated` and redirects to `/login` if false
- [ ] 4.2 Wrap protected route components (e.g. `Dashboard`, `Profile`) with `RequireAuth` in router configuration
- [ ] 4.3 Ensure `/login` route is excluded from protection logic to avoid redirect loops

## 5. Testing

- [ ] 5.1 Write unit tests for `LoginForm.tsx` validating behavior of form fields and submission (frontend/tests/unit/login-form.spec.tsx)
- [ ] 5.2 Add unit tests for `useAuth.ts` verifying token storage and auth state changes
- [ ] 5.3 Create Playwright e2e test `frontend/tests/e2e/login-flow.spec.ts` covering successful login, invalid credentials, and redirect behavior

## 6. Polish & Documentation

- [ ] 6.1 Update `frontend/README.md` with instructions for running auth-related tests
- [ ] 6.2 Add doc snippet to `specs/login-screen/quickstart.md` (or global quickstart) describing the login screen
- [ ] 6.3 Ensure linting and formatting pass across new files
