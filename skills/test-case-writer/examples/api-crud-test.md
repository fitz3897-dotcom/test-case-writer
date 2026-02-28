# Example: REST API User CRUD Test Suite

## Test Target
- **Feature**: User Management REST API
- **Base URL**: `https://api.example.com/v1`
- **Endpoints**:
  - `POST /users` — Create user
  - `GET /users/:id` — Get user by ID
  - `GET /users` — List users (paginated)
  - `PUT /users/:id` — Update user
  - `PATCH /users/:id` — Partial update user
  - `DELETE /users/:id` — Delete user
- **Auth**: Bearer token (JWT)
- **Requirements**: REQ-USER-001 through REQ-USER-012

## Test Design Methods Applied
- Equivalence Partitioning (valid/invalid field values)
- Boundary Value Analysis (field length limits, pagination)
- Decision Table (auth × role × resource ownership)
- Error Guessing (common API mistakes)

---

## Create User — POST /users

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-API-001 | Verify create user returns 201 when all required fields are provided | P0 | Valid admin auth token | `{"name": "Alice Smith", "email": "alice@example.com", "password": "Secure@123", "role": "user"}` | 1. Send POST /users with valid body and auth header | 201 Created; response body contains `id`, `name`, `email`, `role`, `created_at`; password NOT in response | REQ-USER-001 |
| USER-API-002 | Verify create user returns 400 when required field "email" is missing | P1 | Valid admin auth token | `{"name": "Alice Smith", "password": "Secure@123"}` | 1. Send POST /users without email field | 400; error response: `{"error": {"code": "VALIDATION_ERROR", "details": [{"field": "email", "message": "Email is required"}]}}` | REQ-USER-002 |
| USER-API-003 | Verify create user returns 400 when email format is invalid | P1 | Valid admin auth token | `{"name": "Alice", "email": "not-an-email", "password": "Secure@123"}` | 1. Send POST /users with invalid email | 400; error details: field "email", message "Must be a valid email address" | REQ-USER-002 |
| USER-API-004 | Verify create user returns 409 when email already exists | P1 | User `alice@example.com` already exists | `{"name": "Alice Two", "email": "alice@example.com", "password": "Secure@456"}` | 1. Send POST /users with duplicate email | 409 Conflict; error code: "DUPLICATE_EMAIL" | REQ-USER-002 |
| USER-API-005 | Verify create user returns 400 when password is shorter than 8 characters | P1 | Valid admin auth token | `{"name": "Alice", "email": "a2@example.com", "password": "Short1"}` | 1. Send POST /users with 6-char password | 400; error details: field "password", message "Must be at least 8 characters" | REQ-USER-002 |
| USER-API-006 | Verify create user returns 400 when name exceeds 100 characters | P2 | Valid admin auth token | `{"name": "A" × 101, "email": "long@example.com", "password": "Secure@123"}` | 1. Send POST /users with 101-char name | 400; error details: field "name", message "Must be 100 characters or fewer" | REQ-USER-002 |
| USER-API-007 | Verify create user returns 401 when no auth token is provided | P0 | No auth header | Valid user body | 1. Send POST /users without Authorization header | 401 Unauthorized | REQ-USER-003 |
| USER-API-008 | Verify create user returns 403 when non-admin user attempts creation | P1 | Auth token for role "user" (not admin) | Valid user body | 1. Send POST /users with non-admin token | 403 Forbidden; error code: "INSUFFICIENT_PERMISSIONS" | REQ-USER-003 |

## Get User — GET /users/:id

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-API-009 | Verify get user returns 200 with correct user data | P0 | User with ID `usr_001` exists | — | 1. Send GET /users/usr_001 with valid auth | 200; response contains `id`, `name`, `email`, `role`, `created_at`, `updated_at` | REQ-USER-004 |
| USER-API-010 | Verify get user returns 404 when user ID does not exist | P1 | No user with ID `usr_999` | — | 1. Send GET /users/usr_999 with valid auth | 404 Not Found; error code: "USER_NOT_FOUND" | REQ-USER-004 |
| USER-API-011 | Verify get user returns 403 when requesting another user's data (non-admin) | P1 | Auth token for user `usr_002`; requesting `usr_001` | — | 1. Send GET /users/usr_001 with usr_002's token | 403 Forbidden (IDOR protection) | REQ-USER-003 |
| USER-API-012 | Verify get user does not return password field in response | P0 | User exists | — | 1. Send GET /users/usr_001 2. Inspect response body | Response does NOT contain `password` or `password_hash` field | REQ-USER-004 |

## List Users — GET /users

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-API-013 | Verify list users returns paginated results with default page size | P1 | 25 users exist in database | — | 1. Send GET /users (no pagination params) | 200; returns 20 users (default page_size); response includes `total`, `page`, `page_size`, `has_next: true` | REQ-USER-005 |
| USER-API-014 | Verify list users returns correct page when page parameter is specified | P1 | 25 users exist | Query: `?page=2&page_size=10` | 1. Send GET /users?page=2&page_size=10 | 200; returns users 11-20; `page: 2`, `has_next: true` | REQ-USER-005 |
| USER-API-015 | Verify list users returns empty array when page exceeds total | P2 | 25 users exist | Query: `?page=100` | 1. Send GET /users?page=100 | 200; `data: []`, `has_next: false` (not 404) | REQ-USER-005 |
| USER-API-016 | Verify list users filters by role parameter | P2 | Users with roles "admin" and "user" exist | Query: `?role=admin` | 1. Send GET /users?role=admin | 200; all returned users have `role: "admin"` | REQ-USER-005 |
| USER-API-017 | Verify list users returns 400 when page_size is negative | P2 | Valid auth | Query: `?page_size=-1` | 1. Send GET /users?page_size=-1 | 400; error: "page_size must be a positive integer" | REQ-USER-005 |

## Update User — PUT /users/:id

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-API-018 | Verify update user returns 200 when valid data is provided | P0 | User `usr_001` exists | `{"name": "Alice Updated", "email": "alice.new@example.com"}` | 1. Send PUT /users/usr_001 with updated body | 200; response reflects updated `name` and `email`; `updated_at` changed | REQ-USER-006 |
| USER-API-019 | Verify partial update via PATCH updates only specified fields | P1 | User `usr_001` exists; current name: "Alice Updated" | `{"name": "Alice Final"}` | 1. Send PATCH /users/usr_001 with only name field | 200; `name` updated to "Alice Final"; `email` unchanged | REQ-USER-007 |
| USER-API-020 | Verify update user returns 409 when changing email to existing email | P1 | Users `usr_001` and `usr_002` exist; `usr_002` has email `bob@example.com` | `{"email": "bob@example.com"}` | 1. Send PATCH /users/usr_001 with usr_002's email | 409 Conflict; error code: "DUPLICATE_EMAIL" | REQ-USER-006 |
| USER-API-021 | Verify user cannot escalate own role via update | P0 | Auth token for user `usr_001` with role "user" | `{"role": "admin"}` | 1. Send PATCH /users/usr_001 with role "admin" using own token | 403 Forbidden or role field ignored; user role remains "user" | REQ-USER-003 |

## Delete User — DELETE /users/:id

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-API-022 | Verify delete user returns 204 when valid user is deleted | P0 | User `usr_001` exists; admin auth | — | 1. Send DELETE /users/usr_001 with admin token | 204 No Content; subsequent GET /users/usr_001 returns 404 | REQ-USER-008 |
| USER-API-023 | Verify delete user returns 404 when user ID does not exist | P2 | No user with ID `usr_999` | — | 1. Send DELETE /users/usr_999 | 404 Not Found | REQ-USER-008 |
| USER-API-024 | Verify delete user returns 403 when non-admin attempts deletion | P1 | Auth token for role "user" | — | 1. Send DELETE /users/usr_001 with non-admin token | 403 Forbidden | REQ-USER-003 |
| USER-API-025 | Verify repeated delete of same user is idempotent | P2 | User `usr_001` already deleted | — | 1. Send DELETE /users/usr_001 again | 404 Not Found (not 500) | REQ-USER-008 |

## Security Tests

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-SEC-001 | Verify SQL injection in name field is handled safely | P1 | Valid admin auth | `{"name": "'; DROP TABLE users; --", "email": "inject@example.com", "password": "Secure@123"}` | 1. Send POST /users with SQL injection in name | 201 (name stored as literal string) or 400 (rejected); no SQL execution | REQ-USER-009 |
| USER-SEC-002 | Verify expired JWT token returns 401 | P1 | Token expired 1 hour ago | — | 1. Send GET /users/usr_001 with expired token | 401; error code: "TOKEN_EXPIRED" | REQ-USER-003 |
| USER-SEC-003 | Verify malformed JWT token returns 401 | P1 | Token: `invalid.token.here` | — | 1. Send GET /users/usr_001 with malformed token | 401; error code: "INVALID_TOKEN" | REQ-USER-003 |

## Performance Tests

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
|---|---|---|---|---|---|---|---|
| USER-PERF-001 | Verify GET /users/:id responds within 200ms under normal load | P2 | User exists; system under normal load (< 100 RPS) | — | 1. Send 100 sequential GET requests 2. Measure P95 response time | P95 response time < 200ms | REQ-USER-010 |
| USER-PERF-002 | Verify GET /users list responds within 500ms with 10,000 users | P2 | 10,000 users in database | Query: `?page=1&page_size=20` | 1. Send GET /users 2. Measure response time | Response time < 500ms; pagination works correctly | REQ-USER-010 |

---

## Test Case Summary

| Metric | Count |
|---|---|
| Total Test Cases | 28 |
| P0 (Blocker) | 6 |
| P1 (Critical) | 12 |
| P2 (Major) | 10 |
| P3 (Minor) | 0 |
| Functional | 25 |
| Non-Functional | 3 |
| Automation Candidates | 28 |

## Traceability Matrix

| Requirement | Description | Test Cases | Coverage |
|---|---|---|---|
| REQ-USER-001 | Create user with valid data | USER-API-001 | Covered |
| REQ-USER-002 | Input validation for user creation | USER-API-002 through USER-API-006 | Covered |
| REQ-USER-003 | Authentication and authorization | USER-API-007, 008, 011, 021, 024, USER-SEC-002, 003 | Covered |
| REQ-USER-004 | Get user by ID | USER-API-009, 010, 012 | Covered |
| REQ-USER-005 | List users with pagination/filtering | USER-API-013 through USER-API-017 | Covered |
| REQ-USER-006 | Full update user | USER-API-018, 020 | Covered |
| REQ-USER-007 | Partial update user | USER-API-019 | Covered |
| REQ-USER-008 | Delete user | USER-API-022 through USER-API-025 | Covered |
| REQ-USER-009 | Security: injection prevention | USER-SEC-001 | Covered |
| REQ-USER-010 | Performance benchmarks | USER-PERF-001, 002 | Covered |
