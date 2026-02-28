# Test Case Best Practices Reference

This reference covers principles, conventions, and processes for writing high-quality test cases.

---

## 1. Test Case Atomicity (测试用例原子性)

### Principles

- **One test, one objective**: Each test case verifies exactly one behavior or condition. Do not bundle multiple checks into a single test case.
- **Independent execution**: A test case must not depend on the outcome of another test case. Each test case sets up its own preconditions.
- **Deterministic result**: Given the same preconditions and input, the test case always produces the same result. Avoid dependencies on time, randomness, or external state.

### Examples

**Bad — non-atomic:**
> TC-001: Verify user can register, log in, and update profile

This tests three features. If login fails, you don't know if update profile works.

**Good — atomic:**
> TC-001: Verify user registration with valid data succeeds
> TC-002: Verify user login with valid credentials succeeds
> TC-003: Verify user profile update with valid name succeeds

### When to Break Atomicity

End-to-end scenario tests intentionally cover a flow across features. These are acceptable as scenario tests (labeled as such) but should complement, not replace, atomic unit test cases.

---

## 2. Naming Conventions (命名规范)

### TC-ID Format

```
[MODULE]-[TYPE]-[NNN]
```

| Component | Options | Examples |
|---|---|---|
| MODULE | Feature or module abbreviation (2-6 chars, uppercase) | LOGIN, USER, CART, PAY, SRCH, NOTIF |
| TYPE | Test type code | FUNC, NEG, SEC, PERF, UX, EDGE, COMPAT, A11Y |
| NNN | 3-digit sequential number | 001, 002, ... 999 |

Full examples: `LOGIN-FUNC-001`, `PAY-SEC-003`, `CART-PERF-001`

### Title Convention

Use the pattern: **Verify [action] when [condition] results in [expected outcome]**

| Component | Description | Examples |
|---|---|---|
| Action | What the user/system does | "login succeeds", "error message displays", "API returns 400" |
| Condition | Under what circumstances | "valid credentials are entered", "password field is empty", "rate limit is exceeded" |
| Outcome | What should happen | "user redirected to dashboard", "inline error shown below field", "429 response with Retry-After header" |

### Title Examples

- "Verify login succeeds when valid email and password are entered"
- "Verify error message displays when password is less than 8 characters"
- "Verify account locks when 5 consecutive failed login attempts occur"
- "Verify API returns 201 when user is created with all required fields"

### Naming Anti-Patterns

- Too vague: "Test login" — what about login?
- Too long: "Verify that when a user enters an incorrect password for the fifth time in a row on the login page, the system locks their account and sends a notification email" — extract the condition to preconditions
- Imperative: "Enter wrong password and check error" — use declarative "Verify" style

---

## 3. Priority and Severity Classification (优先级与严重性分类)

### Priority (testing order)

| Level | Definition | Examples |
|---|---|---|
| P0 — Blocker | Cannot release without passing; core functionality broken | Login broken, payment fails, data loss, security vulnerability |
| P1 — Critical | Major feature broken; workaround may exist | Search returns wrong results, notification not delivered, API returns wrong data |
| P2 — Major | Feature works but with significant issues | Slow performance, UI misalignment on major screen, missing validation |
| P3 — Minor | Cosmetic or low-impact issues | Typo, color slightly off, tooltip missing, edge case with workaround |

### Severity (impact assessment)

| Level | User Impact | System Impact |
|---|---|---|
| Critical | User cannot complete primary task | Data corruption, security breach, system crash |
| High | User experience significantly degraded | Feature unavailable, incorrect data displayed |
| Medium | User annoyed but can work around it | Performance degradation, intermittent errors |
| Low | User unlikely to notice | Cosmetic, rare edge case |

### Priority vs Severity Matrix

Sometimes severity and priority differ. A high-severity bug in a rarely used feature might be lower priority than a medium-severity bug in the checkout flow.

| | High Severity | Medium Severity | Low Severity |
|---|---|---|---|
| High Priority | Fix immediately (P0) | Fix before release (P1) | Schedule for release (P2) |
| Medium Priority | Fix before release (P1) | Schedule for release (P2) | Backlog (P3) |
| Low Priority | Schedule (P2) | Backlog (P3) | Backlog (P3) |

---

## 4. Requirements Traceability (需求可追溯性)

### Why Trace

- Verify every requirement has at least one test case (no gaps)
- Verify every test case maps to a requirement (no orphan tests)
- Impact analysis: when a requirement changes, identify affected test cases
- Coverage reporting: percentage of requirements with passing tests

### Traceability Matrix

| Requirement ID | Requirement | Test Cases | Status |
|---|---|---|---|
| REQ-001 | User can register with email | REG-FUNC-001, REG-FUNC-002, REG-NEG-001 | Covered |
| REQ-002 | Password must be 8-20 chars | REG-FUNC-003, REG-NEG-002, REG-EDGE-001 | Covered |
| REQ-003 | User can reset password | — | NOT Covered |

### Best Practices

- Include a `Traceability` field in every test case linking to the requirement ID or user story
- Review the traceability matrix during sprint planning to identify uncovered requirements
- Update the matrix when requirements change or are removed
- Use it to prioritize test execution: highest-priority requirements first

---

## 5. Test Data Management (测试数据管理)

### Principles

- **Use concrete values**: Never write "enter valid email" — write `test.user@example.com`
- **Document data dependencies**: If test data requires pre-existing records, state it in preconditions
- **Isolate test data**: Each test case should create its own data or use a known fixture; never depend on data from another test
- **Sensitive data**: Never use real user data. Use realistic but fictional data

### Test Data Categories

| Category | Examples | Storage |
|---|---|---|
| Static reference data | Country codes, currencies, categories | Seed scripts |
| Dynamic test data | User accounts, orders | Created in test setup, deleted in teardown |
| Boundary values | Min/max values, empty strings | Defined inline in test case |
| Invalid data | SQL injection strings, XSS payloads | Shared security test data file |
| Large datasets | 10K users, 100K orders | Generated by data factory scripts |

### Test Data Checklist

- [ ] All test cases specify exact input values, not descriptions
- [ ] Preconditions state required database state
- [ ] No test depends on data created by a previous test
- [ ] Sensitive fields use masked/fictional data
- [ ] Test data is environment-specific (dev, staging, prod have different data)
- [ ] Data cleanup is documented (how to reset after test run)

---

## 6. Review Process (评审流程)

### Self-Review Checklist

Before submitting test cases for review, verify:

- [ ] Every test case has a unique TC-ID following naming convention
- [ ] Titles follow the "Verify [action] when [condition] results in [outcome]" pattern
- [ ] Preconditions are complete and explicit
- [ ] Steps are atomic (one action per step)
- [ ] Expected results are observable and verifiable
- [ ] Test data uses concrete values
- [ ] Priority assignment is justified
- [ ] Both positive and negative scenarios exist
- [ ] Edge cases and boundary values are covered
- [ ] No duplicate or overlapping test cases

### Peer Review Focus Areas

| Reviewer Focus | Questions to Ask |
|---|---|
| Completeness | Are all requirements covered? Any missing scenarios? |
| Correctness | Do expected results match the specification? |
| Clarity | Can another tester execute this without asking questions? |
| Efficiency | Are there redundant test cases that can be merged? |
| Maintainability | Will these test cases break with minor UI/API changes? |

### Review Metrics

- Defects found per test case (indicates test quality)
- Test case review cycle time (target: < 2 days)
- Review comments per test suite (aim to reduce over time)

---

## 7. Regression Strategy (回归策略)

### Regression Test Selection

Not all test cases need to run in every regression cycle. Prioritize:

| Tier | Criteria | Run Frequency |
|---|---|---|
| Smoke tests | Core flows (login, key feature, checkout) | Every build |
| Critical regression | P0 + P1 test cases | Every release |
| Full regression | All test cases | Major releases, quarterly |
| Targeted regression | Tests for changed modules + dependent modules | Per feature branch |

### Regression Suite Maintenance

- Remove test cases for deleted features
- Update test cases when requirements change (don't just add new ones)
- Mark flaky tests and fix root causes (don't just re-run)
- Track regression pass rate over time (target: > 95% pass rate)
- Review and prune the suite quarterly: if a test hasn't found a bug in 6 months, consider if it's still valuable

---

## 8. Automation vs Manual Decision (自动化 vs 手工决策)

### Automate When

| Criterion | Reasoning |
|---|---|
| Test runs frequently (every build) | Manual execution cost too high |
| Test is deterministic | Same input always → same output |
| Test checks data / calculations | Computers are better at comparing values |
| Test has many data combinations | Parameterized tests are efficient |
| Test is part of regression suite | Automation pays off over many runs |
| Test checks API contracts | API tests are highly automatable |

### Keep Manual When

| Criterion | Reasoning |
|---|---|
| UX / visual verification needed | Human eye catches subtle layout issues |
| Exploratory testing | Creative, unscripted investigation |
| One-time test | Automation cost not justified |
| Test requires physical device interaction | Hardware-specific gestures, camera, sensors |
| Test involves subjective judgment | "Does this animation feel smooth?" |
| Requirements are changing rapidly | Automation would need constant updates |

### Automation Candidate Checklist

For each test case, rate automation suitability:

| Factor | Score (1-5) |
|---|---|
| Execution frequency | 1 (rarely) to 5 (every build) |
| Stability of feature | 1 (changing weekly) to 5 (stable for months) |
| Determinism | 1 (flaky/random) to 5 (fully deterministic) |
| Setup complexity | 1 (complex environment) to 5 (simple API call) |
| Value of fast feedback | 1 (low) to 5 (blocking other work) |

Total score > 18: strong automation candidate
Total score 12-18: consider automating
Total score < 12: keep manual

### Tagging Convention

In the test case summary, mark automation candidates:

| TC-ID | Title | Priority | Automation |
|---|---|---|---|
| LOGIN-FUNC-001 | Verify login with valid credentials | P0 | Candidate |
| LOGIN-UX-001 | Verify login animation smoothness | P3 | Manual |
| USER-API-001 | Verify create user API returns 201 | P0 | Automated |
