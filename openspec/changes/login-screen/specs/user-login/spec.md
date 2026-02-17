## ADDED Requirements

### Requirement: Login screen exists

The system SHALL display a login page when an unauthenticated user navigates to a protected route or explicitly visits the login URL.

#### Scenario: Navigate to protected route

- **WHEN** an unauthenticated user attempts to visit `/dashboard` (or other protected URL)
- **THEN** they are redirected to the login page with an empty username and password fields

#### Scenario: Direct login URL

- **WHEN** a user enters `/login` in the browser
- **THEN** the login page loads with appropriate form elements

### Requirement: User can submit credentials

The login page SHALL allow the user to enter a username/email and password and submit them for validation.

#### Scenario: Submit valid credentials

- **WHEN** the user fills both fields and clicks the submit button
- **THEN** the credentials are sent to the authentication endpoint and the system displays a loading indicator

#### Scenario: Submit empty fields

- **WHEN** the user clicks submit with one or both fields empty
- **THEN** inline validation errors appear and the request is not sent

### Requirement: Authentication feedback

The system SHALL inform the user of success or failure of credential validation.

#### Scenario: Correct credentials

- **WHEN** the authentication endpoint responds with success
- **THEN** the user is redirected to the originally requested protected page or a default post-login page

#### Scenario: Incorrect credentials

- **WHEN** the authentication endpoint responds with failure
- **THEN** an error message is displayed indicating the credentials are invalid and the form remains populated

### Requirement: Session token handling

Upon successful login, the system SHALL store an authentication token in localStorage (or cookie) and include it in subsequent API requests.

#### Scenario: Token stored

- **WHEN** login succeeds
- **THEN** a token value is saved under `authToken` in localStorage

#### Scenario: Missing token

- **WHEN** a protected API request is made without a token
- **THEN** the request is blocked and the user is redirected to the login screen
