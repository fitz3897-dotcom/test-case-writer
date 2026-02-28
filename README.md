# test-case-writer

**[English](#english) | [中文](#中文)**

---

<a id="english"></a>

## English

### Overview

**test-case-writer** is a Claude Code plugin that generates comprehensive, well-structured test cases for mobile apps, backend services, APIs, and other software systems. It acts as a senior QA engineer and test architect, applying proven test design methodologies to produce thorough, actionable test suites — not generic checklists, but targeted, high-quality test cases with concrete data and verifiable outcomes.

### Features

- **9 Test Design Methodologies** — Applies equivalence partitioning, boundary value analysis, decision tables, state transition, pairwise testing, error guessing, scenario-based testing, exploratory testing, and risk-based testing in combination
- **Multi-Domain Coverage** — Specialized reference guides for mobile app testing (iOS/Android), backend API testing, UX experience testing, and performance testing
- **Standardized Output** — Every test case follows a consistent template with TC-ID, title, priority, preconditions, test data, steps, expected result, and traceability
- **Bilingual Support** — Generates test cases in the same language as your request (English or Chinese), while keeping TC-IDs and Markdown structure in English for tooling compatibility
- **Iterative Refinement** — Asks clarifying questions for ambiguous requests; offers to expand coverage, build traceability matrices, or generate automation-ready scripts after initial delivery

### Installation

```bash
claude /plugin install test-case-writer
```

### Quick Start

Once installed, the skill is automatically triggered when you ask Claude Code to:

- "Write test cases for the login feature"
- "Design test scenarios for the user management API"
- "Create a test plan for the checkout flow"
- "Generate QA tests for the push notification module"
- "Test coverage analysis for the search feature"
- "Write regression tests for the payment service"

### How It Works

The plugin follows a **5-step workflow** for every test case writing request:

| Step | Action | Description |
|------|--------|-------------|
| **1. Understand** | Gather Context | Identify the test target, system type, requirements, user roles, input constraints, integration points, and known edge cases. Ask clarifying questions if context is incomplete. |
| **2. Select Methods** | Choose Design Methods | Pick 2–3 test design methodologies based on feature characteristics (e.g., input ranges → EP + BVA; multi-condition logic → Decision Table; workflows → State Transition). |
| **3. Write** | Generate Test Cases | Produce test cases using the standard template with concrete test data, atomic steps, and verifiable expected results. |
| **4. Organize** | Categorize | Group into Functional (happy path, negative, boundary, edge) and Non-Functional (performance, security, UX, compatibility, reliability). Flag coverage gaps. |
| **5. Review** | Quality Check | Run a 12-point quality checklist to ensure completeness, correctness, and consistency before delivery. |

### Test Design Methodologies

The plugin applies 9 core methodologies, selected based on the feature under test:

| # | Method | When to Use | Example |
|---|--------|-------------|---------|
| 1 | **Equivalence Partitioning** | Input fields with valid/invalid ranges | Age field: test one value from each partition (below min, valid, above max, non-numeric, empty) |
| 2 | **Boundary Value Analysis** | Numeric ranges, string length limits, date ranges | Password 8–20 chars: test 7, 8, 9, 19, 20, 21 characters |
| 3 | **Decision Table** | Multiple conditions affecting outcome | Free shipping: order amount × membership status → 4 rules |
| 4 | **State Transition** | Workflows with distinct states | Order: New → Confirmed → Shipped → Delivered; test valid + invalid transitions |
| 5 | **Pairwise Testing** | Many parameter combinations | OS × Browser × Resolution: reduce 27 combos to ~9 tests |
| 6 | **Error Guessing** | History of bugs, incomplete specs | Null inputs, SQL injection, double-submit, timezone mismatches |
| 7 | **Scenario-Based Testing** | End-to-end user journeys | Complete purchase flow from search to order confirmation |
| 8 | **Exploratory Testing** | New features, low confidence areas | Charter: "Explore checkout with multiple payment methods to discover edge cases" |
| 9 | **Risk-Based Testing** | Limited time, safety-critical features | Payment = high risk (full coverage); color theme = low risk (happy path only) |

### Output Format

Test cases are delivered as Markdown tables with 8 columns:

```
| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
```

**TC-ID Convention:**

```
[MODULE]-[TYPE]-[NNN]
```

- **MODULE**: Feature abbreviation in uppercase (e.g., `LOGIN`, `USER`, `CART`, `PAY`, `SRCH`, `NOTIF`)
- **TYPE**: Test type code — `FUNC` (functional), `NEG` (negative), `SEC` (security), `PERF` (performance), `UX` (experience), `EDGE` (edge case), `COMPAT` (compatibility), `A11Y` (accessibility)
- **NNN**: 3-digit sequential number within each module-type pair

**Title Convention:**

```
Verify [action] when [condition] results in [outcome]
```

Examples:
- "Verify login succeeds when valid credentials are entered"
- "Verify API returns 429 when rate limit is exceeded"
- "Verify error message displays when password field is empty"

**Priority Levels:**

| Level | Definition | Examples |
|-------|-----------|----------|
| P0 — Blocker | Core business flow broken; cannot release | Login broken, payment fails, data loss, security vulnerability |
| P1 — Critical | Major feature broken; workaround may exist | Search returns wrong results, notifications not delivered |
| P2 — Major | Feature works but with significant quality issues | Slow performance, validation gaps, UI misalignment |
| P3 — Minor | Low-impact issues with easy workarounds | Typos, cosmetic inconsistencies, rare edge cases |

**Summary Table:**

Every test suite includes a summary at the end:

```
| Metric             | Count |
|--------------------|-------|
| Total Test Cases   | NN    |
| P0 (Blocker)       | N     |
| P1 (Critical)      | N     |
| P2 (Major)         | N     |
| P3 (Minor)         | N     |
| Functional         | N     |
| Non-Functional     | N     |
| Automation Candidates | N  |
```

### Domain-Specific Testing Coverage

The plugin includes specialized reference guides loaded on-demand based on the system type:

#### Mobile App Testing (iOS / Android)

Covers 10 testing domains:

1. **UI and Interaction** — Layout, text rendering, touch targets (44×44pt / 48×48dp), keyboard interaction, orientation, dark/light mode, scroll behavior
2. **Device Compatibility** — Screen sizes, OS versions, manufacturers, hardware variants (notch, foldable), memory tiers
3. **Network Conditions** — Wi-Fi, 4G, 3G, offline, Wi-Fi↔cellular switch, intermittent connectivity, high latency
4. **Push Notifications** — Permission flow, delivery in foreground/background/killed, deep links, rich notifications, grouping, DND, scheduling
5. **Permission Management** — Camera, location, photos, microphone, notifications, contacts, Bluetooth; grant/deny/revoke flows
6. **App Lifecycle** — Cold/warm start, background↔foreground, interruptions, memory pressure, force quit, split screen
7. **Install / Upgrade / Uninstall** — Fresh install, version upgrades, data migration, schema migration, uninstall cleanup
8. **Deep Linking** — Universal/App Links, fallback when app not installed, auth-gated links, deferred deep links
9. **Gesture Testing** — Tap, long press, swipe, pinch zoom, pull-to-refresh, drag-and-drop, multi-finger, edge swipe
10. **Accessibility** — VoiceOver/TalkBack, dynamic type, color contrast (WCAG AA), color independence, reduced motion, touch accommodation

#### Backend API / Service Testing

Covers 11 testing domains:

1. **API Functional** — REST (HTTP methods, status codes, headers, idempotency) and GraphQL (queries, mutations, variables, N+1)
2. **Authentication & Authorization** — Credentials, tokens (expired/malformed/refresh), IDOR, privilege escalation, security headers
3. **Input Validation** — Required fields, string/numeric/email/date/enum/array/nested object validation, consistent error responses
4. **Database CRUD & Data Integrity** — Unique constraints, foreign keys, cascades, transactions, concurrent writes, soft delete
5. **Concurrency & Race Conditions** — Simultaneous updates, double-submit, limited resource claims, parallel transfers
6. **Idempotency** — GET/PUT/DELETE idempotency, idempotency keys for POST, concurrent requests with same key
7. **Rate Limiting** — Below/at/above limits, cooldown, tier-based limits, per-endpoint isolation, rate limit headers
8. **Pagination & Filtering** — Page navigation, cursor-based pagination, data changes during pagination, filter+sort+pagination combos
9. **Webhook & Callback** — Event delivery, payload validation, signature verification, retry logic, SSRF prevention
10. **Microservice Integration** — Inter-service calls, graceful degradation, circuit breaker, contract testing, correlation ID propagation
11. **Message Queue** — Producer (publish, backpressure, ordering, transactions), Consumer (processing, duplicates, poison messages, ordering)

#### UX Experience Testing

Covers 5 testing domains:

1. **Consistency** — Visual consistency (typography, colors, spacing, icons, buttons) and behavioral consistency (navigation, confirmations, errors, loading patterns, gestures, forms)
2. **Loading States & Feedback** — Initial load, pull-to-refresh, infinite scroll, action in progress, background refresh, partial load; feedback for every user action
3. **Error Message Readability** — User-friendly, actionable, specific, non-technical, non-blaming, accessible; tested across all error scenarios
4. **Navigation Flow** — Back button, deep navigation, tab state, login redirect, 404, browser back/forward, breadcrumbs, modal dismiss
5. **Animation & Motion** — 60fps frame rate, 200–500ms duration, easing curves, interruption handling, reduced motion support

#### Performance Testing

Covers 9 testing domains:

1. **Response Time Baselines** — API (<200ms P95 simple, <500ms complex), web (FCP <1.5s, TTI <3.5s), mobile cold start (<2s)
2. **Memory Leak Detection** — Navigation loops, modal open/close, long list scroll, image load/dismiss; backend GC, connections, payloads
3. **CPU & Battery** — Idle <3% CPU, scrolling <30% CPU, background ~0%; battery drain comparison, wake lock verification
4. **API Latency Under Load** — Baseline, normal, peak (2–3×), stress, spike (10×), soak (4–24h); P50/P95/P99 metrics
5. **Load / Stress / Soak Testing** — Ramp-up/hold/ramp-down patterns, breaking point identification, graceful degradation verification
6. **Database Query Performance** — PK lookup <5ms, indexed <20ms, JOINs <50ms, FTS <200ms, aggregation <500ms
7. **Cache Performance** — Hit ratio >80%, latency <5ms in-memory, invalidation, stampede prevention, eviction policies
8. **App Startup Time** — Cold <2s, warm <500ms, hot <200ms; test on lowest-spec device, with large DB, expired token
9. **Scroll & Animation Frame Rate** — 60fps target, jank detection, GPU profiling on low-end devices

### Best Practices Built In

The plugin enforces these quality standards automatically:

- **Atomicity** — One test, one objective; independent execution; deterministic results
- **Concrete Test Data** — `email: "test@example.com"`, not `email: "valid email"`; exact boundary values, realistic injection payloads
- **Priority Justification** — P0/P1 reserved for critical paths; priority follows a documented matrix
- **Requirements Traceability** — Every test case links to a requirement ID; traceability matrix included for full suites
- **Coverage Gap Detection** — Flags missing negative tests, boundary values, concurrency tests, offline tests, accessibility tests, security tests
- **Automation Guidance** — Each test case scored for automation suitability (frequency × stability × determinism × setup × feedback value)
- **Regression Strategy** — Smoke (every build) → Critical regression (every release) → Full (major releases) → Targeted (per branch)

### Examples

The `skills/test-case-writer/examples/` directory contains two complete worked examples:

#### Mobile Login Test Suite — 19 test cases

- **Coverage**: Happy path (6), Negative scenarios (8), Edge cases (5), Network/Reliability (3), Security (3)
- **Methods Applied**: Equivalence Partitioning, Boundary Value Analysis, Error Guessing, State Transition, Scenario-Based
- **Auth Methods Covered**: Email/password, Face ID, fingerprint, Google Sign-In, Apple Sign-In
- **Highlights**: Account lockout after 5 failed attempts, double-tap prevention, SQL injection in email field, password not in network logs

#### REST API CRUD Test Suite — 28 test cases

- **Coverage**: Create (8), Get (4), List (5), Update (4), Delete (4), Security (3), Performance (2)
- **Methods Applied**: Equivalence Partitioning, Boundary Value Analysis, Decision Table, Error Guessing
- **Endpoints**: POST /users, GET /users/:id, GET /users, PUT /users/:id, PATCH /users/:id, DELETE /users/:id
- **Highlights**: IDOR protection, privilege escalation prevention, idempotent delete, pagination edge cases, traceability matrix mapping 10 requirements

### Project Structure

```
test-case-writer/
├── .claude-plugin/
│   └── plugin.json                      # Plugin manifest
├── skills/
│   └── test-case-writer/
│       ├── SKILL.md                     # Core skill definition (5-step workflow)
│       ├── examples/
│       │   ├── mobile-login-test.md     # Mobile login test suite (19 cases)
│       │   └── api-crud-test.md         # REST API CRUD test suite (28 cases)
│       └── references/
│           ├── test-design-methods.md   # 9 test design methodologies
│           ├── best-practices.md        # Quality standards & conventions
│           ├── mobile-app-testing.md    # 10 mobile-specific testing domains
│           ├── backend-testing.md       # 11 backend-specific testing domains
│           └── experience-performance.md # UX (5 domains) + Performance (9 domains)
├── CLAUDE.md                            # Claude Code guidance
└── README.md                            # This file
```

**Architecture**: SKILL.md is the entry point that defines the workflow. Reference files are standalone, loaded on-demand by domain. Example files demonstrate expected output format and patterns.

### License

MIT

---

<a id="中文"></a>

## 中文

### 概述

**test-case-writer** 是一个 Claude Code 插件，用于为移动应用、后端服务、API 及其他软件系统生成全面、结构化的测试用例。它扮演资深 QA 工程师和测试架构师的角色，运用经过验证的测试设计方法论，产出针对性强、高质量的测试套件——包含具体的测试数据和可验证的预期结果，而非泛泛的检查清单。

### 功能特性

- **9 大测试设计方法论** —— 综合运用等价类划分、边界值分析、判定表、状态迁移、配对测试、错误猜测、场景化测试、探索性测试和基于风险的测试
- **多领域覆盖** —— 提供移动应用测试（iOS/Android）、后端 API 测试、用户体验测试和性能测试的专项参考指南
- **标准化输出** —— 每条测试用例均遵循统一模板，包含用例编号、标题、优先级、前置条件、测试数据、操作步骤、预期结果和需求追溯
- **双语支持** —— 根据你的请求语言生成测试用例（中文或英文），同时保持用例编号和 Markdown 结构使用英文以兼容工具链
- **迭代优化** —— 对模糊需求主动提出澄清问题；初始交付后可继续扩展覆盖范围、构建追溯矩阵或生成自动化脚本

### 安装

```bash
claude /plugin install test-case-writer
```

### 快速开始

安装后，当你向 Claude Code 提出以下类型的请求时，技能会自动触发：

- "为登录功能编写测试用例"
- "设计用户管理 API 的测试场景"
- "创建结账流程的测试计划"
- "生成推送通知模块的 QA 测试"
- "分析搜索功能的测试覆盖率"
- "编写支付服务的回归测试"

### 工作原理

插件对每个测试用例编写请求遵循 **5 步工作流**：

| 步骤 | 动作 | 说明 |
|------|------|------|
| **1. 理解** | 收集上下文 | 识别测试目标、系统类型、需求、用户角色、输入约束、集成点和已知边界情况。上下文不完整时主动提问。 |
| **2. 选择** | 确定设计方法 | 根据功能特征选择 2–3 种测试设计方法（如：输入范围 → 等价类+边界值；多条件逻辑 → 判定表；工作流 → 状态迁移）。 |
| **3. 编写** | 生成测试用例 | 使用标准模板生成测试用例，包含具体测试数据、原子化步骤和可验证的预期结果。 |
| **4. 组织** | 分类整理 | 分为功能测试（正向、反向、边界、边缘）和非功能测试（性能、安全、体验、兼容性、可靠性）。标记覆盖缺口。 |
| **5. 审查** | 质量检查 | 执行 12 项质量检查清单，确保交付前的完整性、正确性和一致性。 |

### 测试设计方法论

插件根据被测功能的特征选择和应用 9 种核心方法论：

| # | 方法 | 适用场景 | 示例 |
|---|------|---------|------|
| 1 | **等价类划分** | 输入字段有有效/无效范围 | 年龄字段：从每个分区取一个代表值（低于最小值、有效值、超出最大值、非数字、空值） |
| 2 | **边界值分析** | 数值范围、字符串长度限制、日期范围 | 密码 8–20 字符：测试 7、8、9、19、20、21 个字符 |
| 3 | **判定表** | 多个条件影响结果 | 免运费：订单金额 × 会员状态 → 4 条规则 |
| 4 | **状态迁移** | 具有明确状态的工作流 | 订单：新建 → 已确认 → 已发货 → 已送达；测试有效和无效的状态转换 |
| 5 | **配对测试** | 大量参数组合 | 操作系统 × 浏览器 × 分辨率：27 种组合缩减为约 9 条测试 |
| 6 | **错误猜测** | 有缺陷历史、规格不完整 | 空输入、SQL 注入、重复提交、时区不匹配 |
| 7 | **场景化测试** | 端到端用户旅程 | 从搜索到下单确认的完整购买流程 |
| 8 | **探索性测试** | 新功能、信心不足的区域 | 章程："用多种支付方式探索结账流程，发现边界情况" |
| 9 | **基于风险的测试** | 时间有限、安全关键功能 | 支付 = 高风险（全面覆盖）；主题颜色 = 低风险（仅正向路径） |

### 输出格式

测试用例以 8 列 Markdown 表格交付：

```
| TC-ID | Title | Priority | Preconditions | Test Data | Steps | Expected Result | Traceability |
| 用例编号 | 标题 | 优先级 | 前置条件 | 测试数据 | 操作步骤 | 预期结果 | 需求追溯 |
```

**用例编号规范：**

```
[模块]-[类型]-[编号]
```

- **模块**：功能缩写，大写字母（如 `LOGIN`、`USER`、`CART`、`PAY`、`SRCH`、`NOTIF`）
- **类型**：测试类型代码 —— `FUNC`（功能）、`NEG`（反向）、`SEC`（安全）、`PERF`（性能）、`UX`（体验）、`EDGE`（边缘）、`COMPAT`（兼容性）、`A11Y`（无障碍）
- **编号**：每个模块-类型对内的 3 位顺序编号

**标题规范：**

```
Verify [动作] when [条件] results in [结果]
验证 当[条件]时 [动作] 的结果为[预期结果]
```

示例：
- "Verify login succeeds when valid credentials are entered"（验证输入有效凭据时登录成功）
- "Verify API returns 429 when rate limit is exceeded"（验证超出限流时 API 返回 429）
- "Verify error message displays when password field is empty"（验证密码为空时显示错误提示）

**优先级定义：**

| 级别 | 定义 | 示例 |
|------|------|------|
| P0 — 阻塞 | 核心业务流程中断，无法发布 | 登录失败、支付异常、数据丢失、安全漏洞 |
| P1 — 严重 | 主要功能异常，可能有变通方案 | 搜索返回错误结果、通知未送达 |
| P2 — 重要 | 功能可用但存在明显质量问题 | 性能缓慢、校验缺失、UI 错位 |
| P3 — 次要 | 低影响问题，有简单变通方案 | 错别字、样式细微偏差、罕见边缘情况 |

**汇总表：**

每个测试套件末尾包含汇总统计：

```
| 指标           | 数量 |
|----------------|------|
| 测试用例总数   | NN   |
| P0（阻塞）     | N    |
| P1（严重）     | N    |
| P2（重要）     | N    |
| P3（次要）     | N    |
| 功能测试       | N    |
| 非功能测试     | N    |
| 自动化候选     | N    |
```

### 领域专项测试覆盖

插件包含按系统类型按需加载的专项参考指南：

#### 移动应用测试（iOS / Android）

涵盖 10 个测试领域：

1. **UI 与交互测试** —— 布局、文本渲染、触摸目标（44×44pt / 48×48dp）、键盘交互、屏幕方向、深色/浅色模式、滚动行为
2. **设备兼容性矩阵** —— 屏幕尺寸、操作系统版本、制造商、硬件差异（刘海/折叠屏）、内存等级
3. **网络条件测试** —— Wi-Fi、4G、3G、离线、Wi-Fi↔蜂窝切换、间歇性连接、高延迟
4. **推送通知测试** —— 权限流程、前台/后台/杀死状态下的送达、深链接、富通知、分组、免打扰、定时发送
5. **权限管理测试** —— 相机、位置、相册、麦克风、通知、通讯录、蓝牙；授权/拒绝/撤销流程
6. **应用生命周期测试** —— 冷启动/热启动、后台↔前台、中断（来电/闹钟）、内存压力、强制退出、分屏
7. **安装/升级/卸载测试** —— 全新安装、版本升级、数据迁移、数据库架构迁移、卸载清理
8. **深链接测试** —— Universal Link/App Link、未安装应用时的降级、需认证的链接、延迟深链接
9. **手势测试** —— 点击、长按、滑动、双指缩放、下拉刷新、拖放、多指手势、边缘滑动
10. **无障碍测试** —— VoiceOver/TalkBack、动态字体、色彩对比度（WCAG AA）、色彩无关性、减少动效、触控辅助

#### 后端 API / 服务测试

涵盖 11 个测试领域：

1. **API 功能测试** —— REST（HTTP 方法、状态码、请求头、幂等性）和 GraphQL（查询、变更、变量、N+1 问题）
2. **认证与授权** —— 凭据验证、令牌（过期/格式错误/刷新）、IDOR、权限提升、安全响应头
3. **输入校验** —— 必填字段、字符串/数字/邮箱/日期/枚举/数组/嵌套对象校验、统一错误响应格式
4. **数据库 CRUD 与数据完整性** —— 唯一约束、外键、级联操作、事务、并发写入、软删除
5. **并发与竞态条件** —— 同时更新、重复提交、有限资源抢占、并行转账
6. **幂等性测试** —— GET/PUT/DELETE 的幂等性、POST 的幂等键、相同键的并发请求
7. **限流测试** —— 低于/等于/超出限制、冷却恢复、分级限制、按端点隔离、限流响应头
8. **分页与过滤** —— 页面导航、游标分页、分页过程中数据变化、过滤+排序+分页组合
9. **Webhook 与回调** —— 事件送达、载荷校验、签名验证、重试逻辑、SSRF 防护
10. **微服务集成** —— 服务间调用、优雅降级、熔断器、契约测试、关联 ID 传播
11. **消息队列** —— 生产者（发布、背压、顺序、事务）、消费者（处理、去重、毒消息、顺序保证）

#### 用户体验测试

涵盖 5 个测试领域：

1. **一致性测试** —— 视觉一致性（字体、颜色、间距、图标、按钮）和行为一致性（导航、确认、错误、加载、手势、表单）
2. **加载状态与反馈** —— 初始加载、下拉刷新、无限滚动、操作进行中、后台刷新、部分加载；每个用户操作都有反馈
3. **错误信息可读性** —— 用户友好、可操作、具体明确、非技术性、非指责性、无障碍；覆盖所有错误场景
4. **导航流合理性** —— 返回按钮、深层导航、标签页状态、登录重定向、404 页面、浏览器前进/后退、面包屑、弹窗关闭
5. **动效流畅度** —— 60fps 帧率、200–500ms 时长、缓动曲线、中断处理、减少动效支持

#### 性能测试

涵盖 9 个测试领域：

1. **响应时间基准** —— API（简单查询 P95 <200ms，复杂查询 <500ms）、Web（FCP <1.5s，TTI <3.5s）、移动端冷启动（<2s）
2. **内存泄漏检测** —— 页面循环导航、弹窗反复打开关闭、长列表滚动、图片加载释放；后端 GC、连接池、大载荷
3. **CPU 与电池消耗** —— 空闲 <3% CPU、滚动 <30% CPU、后台约 0%；电池消耗对比、唤醒锁验证
4. **负载下的 API 延迟** —— 基线、正常负载、峰值（2–3×）、压力测试、尖峰（10×）、浸泡（4–24 小时）；P50/P95/P99 指标
5. **负载/压力/浸泡测试** —— 爬坡/保持/下降模式、断点识别、优雅降级验证
6. **数据库查询性能** —— 主键查询 <5ms、索引查询 <20ms、JOIN <50ms、全文搜索 <200ms、聚合 <500ms
7. **缓存性能** —— 命中率 >80%、内存缓存延迟 <5ms、缓存失效、缓存击穿防护、淘汰策略
8. **应用启动时间** —— 冷启动 <2s、温启动 <500ms、热启动 <200ms；在最低配置设备、大型数据库、过期令牌条件下测试
9. **滚动与动画帧率** —— 60fps 目标、卡顿检测、低端设备 GPU 分析

### 内置最佳实践

插件自动执行以下质量标准：

- **原子性** —— 一条用例验证一个行为；独立执行；结果确定性
- **具体测试数据** —— 使用 `email: "test@example.com"` 而非 `email: "有效邮箱"`；精确的边界值，真实的注入载荷
- **优先级有据可依** —— P0/P1 保留给关键路径；优先级遵循文档化的矩阵
- **需求可追溯** —— 每条用例关联需求编号；完整套件包含追溯矩阵
- **覆盖缺口检测** —— 标记缺失的反向测试、边界值、并发测试、离线测试、无障碍测试、安全测试
- **自动化指导** —— 每条用例按自动化适用性评分（执行频率 × 稳定性 × 确定性 × 环境复杂度 × 反馈价值）
- **回归策略** —— 冒烟测试（每次构建）→ 关键回归（每次发布）→ 全量回归（大版本）→ 定向回归（按分支）

### 示例

`skills/test-case-writer/examples/` 目录包含两个完整的示例：

#### 移动端登录测试套件 — 19 条测试用例

- **覆盖范围**：正向路径 (6)、反向场景 (8)、边缘情况 (5)、网络/可靠性 (3)、安全 (3)
- **应用方法**：等价类划分、边界值分析、错误猜测、状态迁移、场景化测试
- **认证方式覆盖**：邮箱密码、Face ID、指纹、Google 登录、Apple 登录
- **亮点**：5 次失败后账户锁定、防重复点击、邮箱字段 SQL 注入、密码不出现在网络日志中

#### REST API CRUD 测试套件 — 28 条测试用例

- **覆盖范围**：创建 (8)、查询 (4)、列表 (5)、更新 (4)、删除 (4)、安全 (3)、性能 (2)
- **应用方法**：等价类划分、边界值分析、判定表、错误猜测
- **端点**：POST /users、GET /users/:id、GET /users、PUT /users/:id、PATCH /users/:id、DELETE /users/:id
- **亮点**：IDOR 防护、权限提升防御、幂等删除、分页边界情况、10 条需求的追溯矩阵

### 项目结构

```
test-case-writer/
├── .claude-plugin/
│   └── plugin.json                      # 插件清单
├── skills/
│   └── test-case-writer/
│       ├── SKILL.md                     # 核心技能定义（5 步工作流）
│       ├── examples/
│       │   ├── mobile-login-test.md     # 移动端登录测试套件（19 条用例）
│       │   └── api-crud-test.md         # REST API CRUD 测试套件（28 条用例）
│       └── references/
│           ├── test-design-methods.md   # 9 种测试设计方法论
│           ├── best-practices.md        # 质量标准与规范
│           ├── mobile-app-testing.md    # 10 个移动端专项测试领域
│           ├── backend-testing.md       # 11 个后端专项测试领域
│           └── experience-performance.md # 用户体验 (5) + 性能测试 (9) 领域
├── CLAUDE.md                            # Claude Code 指导文件
└── README.md                            # 本文件
```

**架构**：SKILL.md 是入口，定义工作流。参考文件独立存在，按领域按需加载。示例文件展示预期输出格式和模式。

### 许可证

MIT
