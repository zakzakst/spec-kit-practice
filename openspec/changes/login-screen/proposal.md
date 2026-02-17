## Why

We need a login screen to allow users to authenticate before accessing protected parts of the application. Currently there is no mechanism for users to sign in, which limits the app to an unauthenticated state. Adding a login screen is the first step toward user-specific data and functionality.

## What Changes

- Introduce a new login UI view containing fields for username/email and password.
- Include client-side validation and error display for incorrect credentials.
- Implement authentication flow against the existing API (or a stub if none exists).
- Update navigation to redirect unauthenticated users to the login screen.
- **BREAKING**: Routes previously accessible without auth will now require a valid session.

## Capabilities

### New Capabilities

- `user-login`: provide a login screen and authentication flow for users.

### Modified Capabilities

- None

## Impact

Affected code will include UI components, routing logic, and any existing pages that should be protected. Dependencies may include authentication libraries or API endpoints. The change will require updates to navigation and potentially backend auth endpoints if they are added or modified.
