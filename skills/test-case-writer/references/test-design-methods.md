# Test Design Methods Reference

This reference covers 9 core test design methodologies. Apply them in combination for thorough coverage.

---

## §1 Equivalence Partitioning (等价类划分)

### Core Concept

Divide input data into groups (partitions) where all values in a partition are expected to produce the same behavior. Test one representative value from each partition instead of exhaustively testing every value.

### When to Use

- Input fields accept a range of values (age, price, quantity)
- Dropdown or radio selections with defined options
- Any parameter with identifiable valid and invalid groups

### How to Apply

1. Identify all input parameters
2. For each parameter, determine valid partitions and invalid partitions
3. Select one representative value from each partition
4. Create test cases: one per valid partition, one per invalid partition

### Example: Age Field (accepts 18-65)

| Partition | Range | Representative Value | Expected |
|---|---|---|---|
| Invalid (below min) | < 18 | 10 | Rejected |
| Valid | 18 - 65 | 30 | Accepted |
| Invalid (above max) | > 65 | 80 | Rejected |
| Invalid (non-numeric) | Letters | "abc" | Rejected |
| Invalid (empty) | Empty | "" | Rejected |
| Invalid (negative) | < 0 | -5 | Rejected |

---

## §2 Boundary Value Analysis (边界值分析)

### Core Concept

Defects cluster at the edges of equivalence partitions. Test the exact boundary values: minimum, minimum-1, minimum+1, maximum, maximum-1, maximum+1.

### When to Use

- Numeric input ranges
- String length limits (min/max characters)
- Date ranges
- Array/list size limits
- Pagination offsets

### How to Apply

1. Identify boundaries for each equivalence partition
2. For each boundary, test: boundary value, boundary-1, boundary+1
3. Include the zero point and negative values when applicable

### Example: Password Length (8-20 characters)

| Test Point | Value | Length | Expected |
|---|---|---|---|
| Below min boundary | "abcdefg" | 7 | Rejected |
| At min boundary | "abcdefgh" | 8 | Accepted |
| Above min boundary | "abcdefghi" | 9 | Accepted |
| Mid-range | "abcdefghij12" | 12 | Accepted |
| Below max boundary | "a" × 19 | 19 | Accepted |
| At max boundary | "a" × 20 | 20 | Accepted |
| Above max boundary | "a" × 21 | 21 | Rejected |
| Empty | "" | 0 | Rejected |

---

## §3 Decision Table (判定表)

### Core Concept

Map combinations of conditions to their resulting actions. Each column represents one rule — a unique combination of condition states and the expected outcome.

### When to Use

- Business rules with multiple conditions (e.g., discount eligibility)
- Form submissions where multiple fields interact
- Workflows where the combination of flags/toggles affects the result
- Authorization logic (role + resource + action)

### How to Apply

1. List all conditions (boolean or multi-valued)
2. List all possible actions/outcomes
3. Build a table with all condition combinations
4. For each combination, mark the expected action
5. Simplify by merging rules where conditions don't matter (use "—")

### Example: Free Shipping Eligibility

| Rule | R1 | R2 | R3 | R4 |
|---|---|---|---|---|
| **Conditions** | | | | |
| Order ≥ $50 | Y | Y | N | N |
| Premium member | Y | N | Y | N |
| **Actions** | | | | |
| Free shipping | ✓ | ✓ | ✓ | ✗ |
| Standard rate | | | | ✓ |

This produces 4 test cases covering every condition combination.

---

## §4 State Transition (状态迁移)

### Core Concept

Model the system as a finite state machine. Identify all states, transitions (events/triggers), and guards (conditions). Test every transition path and verify invalid transitions are blocked.

### When to Use

- Order workflows (created → paid → shipped → delivered → returned)
- User account states (active, suspended, locked, deleted)
- Document lifecycle (draft → review → approved → published)
- Connection states (disconnected → connecting → connected → error)

### How to Apply

1. Draw a state diagram: list all states and transitions
2. Identify valid transitions and the events that trigger them
3. Identify invalid transitions (should be blocked or produce error)
4. Design test cases for:
   - Every valid transition (0-switch coverage)
   - Every pair of consecutive transitions (1-switch coverage)
   - Invalid transition attempts from each state

### Example: Order Status

```
States: [New] → [Confirmed] → [Shipped] → [Delivered]
                    ↓                          ↓
               [Cancelled]              [Returned]
```

| TC | Start State | Event | End State | Valid? |
|---|---|---|---|---|
| 1 | New | Confirm | Confirmed | Yes |
| 2 | Confirmed | Ship | Shipped | Yes |
| 3 | Shipped | Deliver | Delivered | Yes |
| 4 | Confirmed | Cancel | Cancelled | Yes |
| 5 | Delivered | Return | Returned | Yes |
| 6 | New | Ship | — | No (blocked) |
| 7 | Cancelled | Confirm | — | No (blocked) |
| 8 | Delivered | Cancel | — | No (blocked) |

---

## §5 Pairwise Testing (配对测试)

### Core Concept

Most defects are triggered by the interaction of at most two parameters. Instead of testing all combinations (which grows exponentially), test every pair of parameter values at least once. This dramatically reduces test count while maintaining high defect detection.

### When to Use

- Configuration testing (OS × browser × resolution × language)
- API calls with many optional parameters
- Form submissions with multiple independent fields
- Feature flag combinations

### How to Apply

1. List all parameters and their possible values
2. Use an orthogonal array or pairwise generation tool (e.g., PICT, AllPairs)
3. Generate the minimum set of test configurations covering all pairs
4. Add specific high-risk combinations manually if needed

### Example: Browser Compatibility

Parameters:
- OS: Windows, macOS, Linux
- Browser: Chrome, Firefox, Safari
- Resolution: 1080p, 1440p, 4K

Full combinations = 3 × 3 × 3 = 27. Pairwise reduces to ~9-12 tests:

| TC | OS | Browser | Resolution |
|---|---|---|---|
| 1 | Windows | Chrome | 1080p |
| 2 | Windows | Firefox | 1440p |
| 3 | Windows | Safari | 4K |
| 4 | macOS | Chrome | 1440p |
| 5 | macOS | Firefox | 4K |
| 6 | macOS | Safari | 1080p |
| 7 | Linux | Chrome | 4K |
| 8 | Linux | Firefox | 1080p |
| 9 | Linux | Safari | 1440p |

---

## §6 Error Guessing (错误猜测)

### Core Concept

Leverage testing experience and domain knowledge to anticipate where defects are most likely to hide. This is an experience-based technique that complements systematic methods.

### When to Use

- After applying systematic techniques, to catch what they miss
- When time is limited and you need high-value test cases fast
- For areas with a history of bugs
- When specifications are incomplete or ambiguous

### Common Error Categories

**Data-related errors:**
- Null, empty, whitespace-only inputs
- Special characters: `< > & " ' / \ ; -- {} [] ()`
- Unicode and emoji: `中文`, `العربية`, `🎉`
- Extremely long strings (10,000+ characters)
- SQL injection patterns: `' OR 1=1 --`
- XSS patterns: `<script>alert(1)</script>`
- Leading/trailing whitespace
- Zero, negative numbers, MAX_INT, MIN_INT
- Floating point precision: 0.1 + 0.2

**Timing and sequence errors:**
- Double-click / double-submit
- Back button after form submission
- Concurrent modifications to the same resource
- Actions during loading state
- Session timeout during operation

**State and environment errors:**
- Low disk space / low memory
- Timezone mismatches
- Daylight saving time transitions
- Leap year dates (Feb 29)
- Year boundary (Dec 31 → Jan 1)
- Clock skew between client and server

---

## §7 Scenario-Based Testing (场景化测试)

### Core Concept

Test complete user journeys end-to-end, simulating real-world usage patterns. Each scenario represents a story from the user's perspective, covering multiple features in sequence.

### When to Use

- Acceptance testing for user stories
- Integration testing across multiple components
- Critical business flows (checkout, onboarding, payment)
- When you need stakeholder-readable test cases

### How to Apply

1. Identify key user personas (new user, returning user, admin)
2. Map their most common journeys through the system
3. For each journey, write a scenario with:
   - Context: who the user is and what they want
   - Flow: step-by-step actions across the system
   - Verification points: what to check at each stage
4. Add alternative paths: what happens when something goes wrong mid-journey

### Example: E-Commerce Purchase Scenario

**Scenario**: Returning customer purchases item using saved payment method.

1. User opens app → home screen loads within 2s
2. User searches for "wireless headphones" → results display within 1s
3. User selects first result → product page shows correct price, stock status
4. User taps "Add to Cart" → cart badge updates, shows correct count
5. User navigates to cart → correct item, quantity, subtotal displayed
6. User proceeds to checkout → saved address and payment method pre-filled
7. User confirms order → order confirmation displayed with order number
8. User receives order confirmation email within 5 minutes

**Alternative path**: item goes out of stock between step 4 and step 6 → verify user sees "item unavailable" message at checkout.

---

## §8 Exploratory Testing (探索性测试)

### Core Concept

Simultaneous learning, test design, and test execution. The tester explores the application with a time-boxed charter, using observations to guide further testing. Complements scripted testing by finding unexpected defects.

### When to Use

- New or unfamiliar features (to build understanding)
- After scripted tests pass but confidence is low
- When specification is incomplete or evolving
- To find usability issues that scripted tests miss
- During bug bashes or timed testing sessions

### How to Structure

Use the **charter format**: "Explore [target] with [resources] to discover [information]."

Example charters:
- "Explore the checkout flow with multiple payment methods to discover payment processing edge cases"
- "Explore the user profile page with slow network simulation to discover loading and error handling issues"
- "Explore the admin dashboard with a non-admin account to discover authorization gaps"

### Session Notes Template

```
Charter: [statement]
Duration: [30/60/90 minutes]
Environment: [device, OS, network]
Areas Explored: [list]
Observations:
  - [finding 1 — bug / question / concern]
  - [finding 2]
Issues Found:
  - [bug description, severity, steps to reproduce]
Coverage Gaps:
  - [areas not yet explored]
```

---

## §9 Risk-Based Testing (基于风险的测试)

### Core Concept

Prioritize test effort based on risk = likelihood of failure × impact of failure. Allocate more test cases and deeper coverage to high-risk areas. Reduce testing in low-risk, stable areas.

### When to Use

- Limited time or resources for testing
- Large systems where full coverage is impractical
- After a major refactoring or dependency upgrade
- When deciding test priorities for a release
- Compliance or safety-critical systems

### How to Apply

1. **Identify risks**: List features/modules and rate each on:
   - Likelihood: How likely is a defect? (complexity, recent changes, historical bugs)
   - Impact: How severe if it fails? (revenue, data loss, user safety, reputation)
2. **Score risks**: Use a simple matrix:

| | Low Impact | Medium Impact | High Impact |
|---|---|---|---|
| **High Likelihood** | Medium | High | Critical |
| **Medium Likelihood** | Low | Medium | High |
| **Low Likelihood** | Low | Low | Medium |

3. **Allocate testing effort**:
   - **Critical**: Full coverage — all methods, all edge cases, stress tests
   - **High**: Thorough coverage — systematic methods + key edge cases
   - **Medium**: Standard coverage — equivalence partitioning + boundary values
   - **Low**: Minimal coverage — happy path + one negative test

4. **Review and adjust**: Update risk assessments as testing reveals actual defect distribution.

### Example: E-Commerce Risk Matrix

| Module | Likelihood | Impact | Risk Level | Test Depth |
|---|---|---|---|---|
| Payment processing | Medium | High | High | Full coverage + security tests |
| User registration | Low | Medium | Low | Happy path + validation |
| Product search | Medium | Medium | Medium | Standard + performance |
| Admin bulk delete | Low | High | Medium | Standard + confirmation flows |
| Color theme toggle | Low | Low | Low | Happy path only |
