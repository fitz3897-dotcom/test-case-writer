# test-case-writer

A Claude Code plugin that generates comprehensive, well-structured test cases for mobile apps, backend services, APIs, and other software systems.

## Features

- Applies proven test design methodologies: equivalence partitioning, boundary value analysis, decision tables, state transition, pairwise testing, and more
- Supports mobile app testing (iOS/Android), backend API testing, and general software testing
- Generates test cases with proper TC-IDs, priorities, preconditions, and traceability
- Includes domain-specific references for mobile, backend, UX, and performance testing
- Follows industry best practices for test case writing and organization

## Installation

```bash
claude /plugin install test-case-writer
```

## Usage

Once installed, the skill is automatically triggered when you ask Claude Code to:

- "Write test cases for [feature]"
- "Design test scenarios for [module]"
- "Create a test plan for [system]"
- "Generate QA tests for [API endpoint]"
- "Test coverage analysis for [feature]"

## Test Design Methods

The plugin applies 9 core test design methodologies:

1. **Equivalence Partitioning** - Divide inputs into valid/invalid groups
2. **Boundary Value Analysis** - Test at the edges of input ranges
3. **Decision Table** - Map condition combinations to outcomes
4. **State Transition** - Model and test system state changes
5. **Pairwise Testing** - Cover parameter pair combinations efficiently
6. **Error Guessing** - Anticipate common defect patterns
7. **Scenario-Based Testing** - Test end-to-end user journeys
8. **Exploratory Testing** - Time-boxed, charter-driven exploration
9. **Risk-Based Testing** - Prioritize by likelihood x impact

## Output Format

Test cases are delivered as Markdown tables with:

| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |

Priority levels: P0 (Blocker), P1 (Critical), P2 (Major), P3 (Minor)

## Examples

See the `skills/test-case-writer/examples/` directory for complete worked examples:

- **Mobile Login Test Suite** - 19 test cases covering happy path, negative, edge, network, and security scenarios
- **REST API CRUD Test Suite** - 28 test cases covering all CRUD operations with auth, pagination, and security

## License

MIT
