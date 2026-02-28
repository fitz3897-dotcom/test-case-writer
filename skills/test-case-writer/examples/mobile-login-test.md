# Example: Mobile App Login Test Suite

## Test Target
- **Feature**: User Login (Mobile App)
- **Platform**: iOS & Android
- **Auth methods**: Email/password, biometric (Face ID / fingerprint), third-party (Google, Apple Sign-In)
- **Requirements**: REQ-AUTH-001 through REQ-AUTH-008

## Test Design Methods Applied
- Equivalence Partitioning (valid/invalid credentials)
- Boundary Value Analysis (password length limits)
- Error Guessing (common login failures)
- State Transition (account lock after failed attempts)
- Scenario-Based (end-to-end login flows)

---

## Functional Tests — Happy Path

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| LOGIN-FUNC-001 | Verify login succeeds when valid email and password are entered | P0 | Registered user exists; app on login screen | Email: `alice@example.com`, Password: `Test@1234` | 1. Enter email 2. Enter password 3. Tap "Login" | User redirected to home screen; welcome message shows user name | REQ-AUTH-001 |
| LOGIN-FUNC-002 | Verify login succeeds when using Face ID after enabling biometric login | P0 | User previously enabled biometric login; Face ID enrolled on device | Enrolled face | 1. Open app 2. Biometric prompt appears 3. Authenticate with Face ID | User logged in directly to home screen | REQ-AUTH-005 |
| LOGIN-FUNC-003 | Verify login succeeds when using fingerprint authentication | P0 | User previously enabled biometric login; fingerprint enrolled | Enrolled fingerprint | 1. Open app 2. Biometric prompt appears 3. Authenticate with fingerprint | User logged in directly to home screen | REQ-AUTH-005 |
| LOGIN-FUNC-004 | Verify login succeeds when using Google Sign-In | P1 | Google account available; Google app installed | Google account: `testuser@gmail.com` | 1. Tap "Continue with Google" 2. Select Google account 3. Authorize | User logged in; profile synced from Google | REQ-AUTH-006 |
| LOGIN-FUNC-005 | Verify login succeeds when using Apple Sign-In | P1 | Apple ID available; iOS device | Apple ID | 1. Tap "Sign in with Apple" 2. Authenticate with Face ID/passcode 3. Choose email sharing preference | User logged in; account linked to Apple ID | REQ-AUTH-006 |
| LOGIN-FUNC-006 | Verify "Remember Me" keeps user logged in after app restart | P1 | User on login screen | Email: `alice@example.com`, Password: `Test@1234` | 1. Enter credentials 2. Enable "Remember Me" toggle 3. Tap "Login" 4. Force quit app 5. Reopen app | User is still logged in on app reopen | REQ-AUTH-007 |

## Functional Tests — Negative Scenarios

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| LOGIN-NEG-001 | Verify error displays when incorrect password is entered | P0 | Registered user exists | Email: `alice@example.com`, Password: `WrongPass1` | 1. Enter email 2. Enter wrong password 3. Tap "Login" | Error: "Invalid email or password. Please try again." Login button re-enabled. | REQ-AUTH-002 |
| LOGIN-NEG-002 | Verify error displays when non-registered email is entered | P0 | Email not registered | Email: `nobody@example.com`, Password: `Test@1234` | 1. Enter unregistered email 2. Enter password 3. Tap "Login" | Error: "Invalid email or password. Please try again." (same message as wrong password — no user enumeration) | REQ-AUTH-002 |
| LOGIN-NEG-003 | Verify error displays when email field is empty | P1 | App on login screen | Email: empty, Password: `Test@1234` | 1. Leave email empty 2. Enter password 3. Tap "Login" | Inline error below email field: "Email is required" | REQ-AUTH-003 |
| LOGIN-NEG-004 | Verify error displays when password field is empty | P1 | App on login screen | Email: `alice@example.com`, Password: empty | 1. Enter email 2. Leave password empty 3. Tap "Login" | Inline error below password field: "Password is required" | REQ-AUTH-003 |
| LOGIN-NEG-005 | Verify error displays when both fields are empty | P2 | App on login screen | Both fields empty | 1. Leave both fields empty 2. Tap "Login" | Inline errors below both fields | REQ-AUTH-003 |
| LOGIN-NEG-006 | Verify error displays when email format is invalid | P1 | App on login screen | Email: `not-an-email`, Password: `Test@1234` | 1. Enter invalid email format 2. Enter password 3. Tap "Login" | Inline error: "Please enter a valid email address" | REQ-AUTH-003 |
| LOGIN-NEG-007 | Verify account locks after 5 consecutive failed login attempts | P0 | Registered user; 0 failed attempts | Email: `alice@example.com`, Password: `Wrong1` through `Wrong5` | 1. Enter correct email 2. Enter wrong password 3. Tap "Login" 4. Repeat steps 2-3 four more times (5 total) | After 5th attempt: "Account temporarily locked. Try again in 15 minutes or reset your password." | REQ-AUTH-004 |
| LOGIN-NEG-008 | Verify locked account cannot login even with correct password | P0 | Account locked (5 failed attempts within lockout window) | Email: `alice@example.com`, Password: `Test@1234` (correct) | 1. Enter correct credentials 2. Tap "Login" | Error: "Account temporarily locked. Try again in 15 minutes or reset your password." | REQ-AUTH-004 |

## Edge Cases

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| LOGIN-EDGE-001 | Verify login handles email with leading/trailing whitespace | P2 | Registered user | Email: `  alice@example.com  `, Password: `Test@1234` | 1. Enter email with spaces 2. Enter password 3. Tap "Login" | Whitespace trimmed; login succeeds | REQ-AUTH-001 |
| LOGIN-EDGE-002 | Verify login handles email with different casing | P2 | Registered user with `alice@example.com` | Email: `Alice@Example.COM`, Password: `Test@1234` | 1. Enter email in mixed case 2. Enter password 3. Tap "Login" | Login succeeds (email is case-insensitive) | REQ-AUTH-001 |
| LOGIN-EDGE-003 | Verify password field masks input characters | P1 | App on login screen | Password: `Test@1234` | 1. Enter password 2. Observe password field | Characters are masked (dots/bullets); "show password" toggle reveals text | REQ-AUTH-001 |
| LOGIN-EDGE-004 | Verify login with password at minimum length (8 chars) | P2 | Registered user with 8-char password | Email: `alice@example.com`, Password: `Abcd@123` | 1. Enter credentials 2. Tap "Login" | Login succeeds | REQ-AUTH-001 |
| LOGIN-EDGE-005 | Verify login with password at maximum length (20 chars) | P2 | Registered user with 20-char password | Email: `alice@example.com`, Password: `Abcdefgh@12345678!!` | 1. Enter credentials 2. Tap "Login" | Login succeeds | REQ-AUTH-001 |

## Network and Reliability Tests

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| LOGIN-EDGE-006 | Verify login shows error when device is offline | P1 | Device in airplane mode | Valid credentials | 1. Enable airplane mode 2. Enter credentials 3. Tap "Login" | Error: "No internet connection. Please check your network and try again." | REQ-AUTH-008 |
| LOGIN-EDGE-007 | Verify login handles server timeout gracefully | P1 | Server response delayed > 30s | Valid credentials | 1. Enter credentials 2. Tap "Login" 3. Wait for timeout | Loading spinner appears; after timeout: "Request timed out. Please try again." | REQ-AUTH-008 |
| LOGIN-EDGE-008 | Verify double-tap on login button does not send duplicate requests | P1 | App on login screen | Valid credentials | 1. Enter credentials 2. Rapidly tap "Login" twice | Login button disables after first tap; only one login request sent; user logged in once | REQ-AUTH-008 |

## Security Tests

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| LOGIN-SEC-001 | Verify SQL injection in email field is handled safely | P1 | App on login screen | Email: `' OR 1=1 --`, Password: `anything` | 1. Enter SQL injection in email 2. Enter password 3. Tap "Login" | Error: "Please enter a valid email address" or "Invalid email or password." No data leak. | REQ-AUTH-002 |
| LOGIN-SEC-002 | Verify XSS payload in email field is sanitized | P2 | App on login screen | Email: `<script>alert(1)</script>@test.com`, Password: `Test@1234` | 1. Enter XSS payload in email 2. Tap "Login" | Script not executed; treated as invalid email or safe string | REQ-AUTH-002 |
| LOGIN-SEC-003 | Verify password is not visible in app logs or network traffic | P0 | Network proxy (Charles/mitmproxy) configured | Valid credentials | 1. Enter credentials 2. Tap "Login" 3. Inspect network request | Password is not in plaintext in URL or logs; HTTPS encrypts payload; no password in local logs | REQ-AUTH-002 |

---

## Test Case Summary

| Metric | Count |
|---|---|
| Total Test Cases | 19 |
| P0 (Blocker) | 6 |
| P1 (Critical) | 8 |
| P2 (Major) | 5 |
| P3 (Minor) | 0 |
| Functional | 14 |
| Non-Functional | 5 |
| Automation Candidates | 14 |
