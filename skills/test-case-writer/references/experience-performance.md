# Experience and Performance Testing Reference

This reference covers UX experience testing and performance testing. Use the relevant section based on the test target.

---

# §UX — User Experience Testing

## 1. Consistency (一致性测试)

### Visual Consistency

| Test Point | Check |
|---|---|
| Typography | Same font family, sizes, and weights used for same-level headings, body text, labels across all screens |
| Color palette | Primary, secondary, error, warning, success colors match design system |
| Spacing | Consistent margins and padding between similar elements (e.g., all card components have same padding) |
| Icons | Same icon style (line vs filled) used throughout; same icon for same action |
| Button styles | Primary, secondary, tertiary buttons are visually distinct and used consistently |
| Component variants | Same component looks and behaves the same everywhere (e.g., all modal dialogs have same close behavior) |

### Behavioral Consistency

| Test Point | Check |
|---|---|
| Navigation patterns | Back behavior is predictable across all flows |
| Confirmation dialogs | Destructive actions always require confirmation |
| Error handling | All errors display in the same format (inline, toast, modal) |
| Loading patterns | Same loading indicator style (skeleton, spinner, shimmer) across similar views |
| Gesture behavior | Same swipe action produces same result across similar list items |
| Form behavior | Tab order, auto-focus, and submit behavior consistent across all forms |

---

## 2. Loading States and Feedback (加载状态与反馈)

### Loading State Coverage

Every network request or async operation should have these states tested:

| State | Test Points |
|---|---|
| Initial load | Skeleton screen or spinner shown; content replaces loader smoothly |
| Pull to refresh | Refresh indicator shown; new data appears; indicator dismisses |
| Load more / infinite scroll | Footer loader shown; new items appended without scroll jump |
| Action in progress | Button shows loading state; button is disabled to prevent double-tap |
| Background refresh | Data updates without disrupting user's current scroll position |
| Partial load | Some content loads while other sections show individual loaders |

### Feedback Completeness

| User Action | Expected Feedback |
|---|---|
| Tap a button | Visual press state (color change, scale) within 100ms |
| Submit a form | Loading indicator → success message OR error message |
| Delete an item | Confirmation dialog → item removed with animation → undo option (if applicable) |
| Copy to clipboard | Toast: "Copied" appears for 2-3 seconds |
| Save / favorite | Icon state changes immediately (optimistic UI); reverts on failure |
| Network error | Clear error message with retry option; not a blank screen |
| Empty state | Friendly illustration/message + action CTA (e.g., "No results — try a different search") |

---

## 3. Error Message Readability (错误信息可读性)

### Error Message Quality Checklist

| Criterion | Good Example | Bad Example |
|---|---|---|
| User-friendly language | "That email is already registered. Try logging in instead." | "Error: UNIQUE_CONSTRAINT_VIOLATION on users.email" |
| Actionable | "Password must be at least 8 characters" | "Invalid password" |
| Specific | "File must be a PNG or JPEG under 5MB" | "Upload failed" |
| Non-technical | "Something went wrong. Please try again." | "500 Internal Server Error" |
| Non-blaming | "We couldn't find that page" | "You entered an invalid URL" |
| Accessible | Error text has 4.5:1 contrast; announced by screen reader | Error shown only as red border |

### Error Scenarios to Test

- Validation errors on each form field (shown inline, not just as a top banner)
- Network timeout → message + retry button
- Server error (500) → generic friendly message + support contact option
- Session expired → redirect to login with "session expired" message
- Permission denied → explain what permission is needed and how to get it
- Rate limited → "Too many attempts. Please wait [N] minutes."
- Maintenance mode → "We're performing maintenance. Back by [time]."

---

## 4. Navigation Flow (导航流合理性)

### Navigation Test Cases

| Scenario | Verify |
|---|---|
| Back button behavior | Returns to previous screen with preserved state; doesn't loop |
| Deep navigation (3+ levels deep) | User can easily return to root |
| Tab bar state | Switching tabs preserves each tab's navigation state |
| Login redirect | After login, user returns to the screen they intended to visit |
| Link to non-existent page | 404 page with link back to home |
| Browser back/forward (web) | Correct page and state restored; no broken states |
| Breadcrumb accuracy | Each breadcrumb segment links to the correct parent screen |
| Modal / sheet dismiss | Data entered in modal is handled (saved or discard confirmed) |

### Common Navigation Defects

- Back button after form submission shows stale form data
- Deep link opens a screen with no back navigation to the main app
- Tab badge count doesn't clear after visiting the tab
- Swipe-back gesture dismisses a modal and the parent screen simultaneously

---

## 5. Animation and Motion (动效流畅度)

### Animation Quality Tests

| Test Point | Criteria |
|---|---|
| Frame rate | Animations maintain 60fps (no jank or dropped frames) |
| Duration | Transitions are 200-500ms; not too fast or too slow |
| Easing | Natural easing curves (ease-in-out); no linear motion for UI elements |
| Interruption | Animations can be interrupted (e.g., swipe during a transition) without glitch |
| Reduced Motion | All non-essential animations disabled when "Reduce Motion" is enabled |
| Loading animations | Loaders animate smoothly even on low-end devices |
| Scroll performance | 60fps scrolling; no layout shifts during scroll |

---

# §Performance — Performance Testing

## 1. Response Time Baselines (响应时间基准)

### Target Benchmarks

| Metric | Target | Measurement Method |
|---|---|---|
| API response (simple query) | < 200ms (P95) | Load testing tool or APM |
| API response (complex query) | < 500ms (P95) | Load testing tool or APM |
| Page load (web, first contentful paint) | < 1.5s | Lighthouse, WebPageTest |
| Page load (web, time to interactive) | < 3.5s | Lighthouse |
| App cold start (mobile) | < 2s | Profiler, manual timing |
| App warm start (mobile) | < 500ms | Profiler |
| Search results | < 1s | End-to-end timing |
| File upload (1MB) | < 3s on 4G | Network-conditioned test |

### Test Cases

- Measure response time for each critical API endpoint under normal load
- Verify P50, P95, P99 response times against benchmarks
- Test response time with increasing payload sizes (100 items, 1000 items, 10000 items)
- Compare response times with and without caching

---

## 2. Memory Leak Detection (内存泄漏检测)

### Mobile App Memory Tests

| Scenario | Check |
|---|---|
| Navigate back and forth between screens 50 times | Memory returns to baseline |
| Open and close modals/sheets repeatedly | No monotonic memory growth |
| Scroll through a long list (10,000+ items) | Memory stays bounded (virtualized list) |
| Load and dismiss images repeatedly | Image memory released after dismissal |
| Background the app for 1 hour | Memory not significantly higher than at launch |

### Backend Memory Tests

| Scenario | Check |
|---|---|
| Process 10,000 requests | Memory returns to baseline after GC |
| Long-running connections (WebSocket, SSE) | Memory grows linearly with connections, not exponentially |
| Large payload processing | Memory released after response sent |
| Error handling paths | Memory released even on error/exception paths |

---

## 3. CPU and Battery Consumption (CPU/电池消耗)

### Mobile App CPU Tests

| Scenario | Acceptable |
|---|---|
| App idle in foreground | < 3% CPU |
| Scrolling a list | < 30% CPU; smooth 60fps |
| Background (no active tasks) | ~ 0% CPU |
| Location tracking in background | < 5% CPU; uses significant location changes, not continuous GPS |
| Video playback | Hardware decoder used; CPU < 15% |

### Battery Impact Tests

- Run core workflow for 1 hour and measure battery drain
- Compare battery drain with competitor apps for similar functionality
- Verify no wake locks held after background tasks complete
- Check that network requests are batched (not one-at-a-time)

---

## 4. API Latency Under Load (API 延迟测试)

### Load Profiles

| Profile | Description | Key Metric |
|---|---|---|
| Baseline | 1 concurrent user | P50 response time |
| Normal load | Expected daily active users | P95 response time |
| Peak load | 2-3× normal (e.g., sale event) | P99 response time, error rate |
| Stress test | Gradually increase until failure | Breaking point, failure mode |
| Spike test | Sudden 10× traffic spike | Recovery time, error rate during spike |
| Soak test | Normal load for 4-24 hours | Memory leaks, connection pool exhaustion |

### Metrics to Capture

- Response time distribution (P50, P95, P99)
- Throughput (requests/second)
- Error rate (% of 5xx responses)
- CPU utilization
- Memory utilization
- Database connection pool usage
- Thread pool / goroutine count

---

## 5. Load, Stress, and Soak Testing (负载/压力/浸泡测试)

### Load Test Design

1. Identify critical user journeys (login → browse → search → checkout)
2. Define realistic user behavior (think time between actions: 3-10s)
3. Set load levels: ramp up over 5 minutes → hold for 15 minutes → ramp down
4. Measure: response times, error rates, resource utilization at each level

### Stress Test Design

1. Start at normal load
2. Increase by 20% every 5 minutes until system degrades
3. Record: at what load does P95 exceed SLA? When do errors start? What fails first?
4. Verify graceful degradation: system should return errors, not crash

### Soak Test Design

1. Run at 70-80% of peak capacity for 4-24 hours
2. Monitor for: memory leaks, connection leaks, disk space growth, log file growth
3. Verify: response times stable over time (no degradation)

---

## 6. Database Query Performance (数据库查询性能)

### Test Scenarios

| Scenario | Threshold |
|---|---|
| Simple SELECT by primary key | < 5ms |
| SELECT with index | < 20ms |
| JOIN across 2-3 tables | < 50ms |
| Full-text search | < 200ms |
| Aggregation on large dataset | < 500ms |
| Complex report query | < 2s |

### Performance Checks

- Verify slow query log is enabled and monitored
- Check that all WHERE clauses use indexes (EXPLAIN ANALYZE)
- Test query performance with realistic data volume (not just dev seed data)
- Verify N+1 queries are resolved (ORM eager loading)
- Test with 10× current data volume to predict future performance

---

## 7. Cache Performance (缓存命中率)

### Cache Test Scenarios

| Scenario | Expected |
|---|---|
| First request (cold cache) | Cache miss → fetch from origin → store in cache |
| Second identical request | Cache hit → return cached data (faster response) |
| Data updated | Cache invalidated → next request fetches fresh data |
| Cache expiry (TTL) | After TTL expires, next request fetches fresh data |
| Cache stampede | Only one request fetches from origin; others wait for cached result |
| Cache eviction (memory full) | LRU/LFU eviction; most-used items remain cached |

### Metrics to Monitor

- Cache hit ratio: target > 80% for read-heavy endpoints
- Cache latency: < 5ms for in-memory (Redis), < 50ms for distributed
- Origin load reduction: with caching, origin requests should drop significantly
- Stale data incidents: how often do users see outdated cached data

---

## 8. App Startup Time (App 启动时间)

### Mobile App Startup Metrics

| Metric | Target | How to Measure |
|---|---|---|
| Cold start (first launch) | < 2s to interactive | Time from tap to first usable frame |
| Warm start (from background) | < 500ms | Time from resume to screen rendered |
| Hot start (task switcher) | < 200ms | Time from switch to responsive |

### Startup Performance Tests

- Measure startup time on lowest-spec supported device
- Measure with large local database (10,000+ records)
- Measure with expired auth token (requires token refresh)
- Measure first launch after install (no cache, need initial data)
- Compare startup time across OS versions

### Common Startup Bottlenecks

- Synchronous network requests on main thread
- Large database migrations
- Excessive dependency initialization
- Unoptimized image loading
- Analytics SDK initialization blocking UI

---

## 9. Scroll and Animation Frame Rate (滚动/动画帧率)

### Frame Rate Test Scenarios

| Scenario | Target | How to Measure |
|---|---|---|
| Scroll a list of 100 items | 60fps (16.6ms/frame) | GPU profiler, systrace |
| Scroll list with images | 60fps | Check for image decode on main thread |
| Page transition animation | 60fps | Frame time histogram |
| Complex layout scroll | > 45fps minimum | Identify jank frames (> 33ms) |
| Parallax or physics animation | 60fps | GPU rendering profiler |

### Common Jank Causes

- Image decoding on main/UI thread
- Layout recalculation during scroll (forced reflow)
- Unnecessary re-renders (React, Flutter)
- Excessive overdraw (multiple overlapping layers)
- GC pauses during animation
- Shadow/blur effects without hardware acceleration

### How to Test

1. Enable GPU rendering overlay on device (Android: Developer Options; iOS: Core Animation instrument)
2. Scroll through target screens and observe frame time bars
3. Record a trace with systrace (Android) or Instruments (iOS)
4. Identify frames exceeding 16.6ms and their root cause
5. Repeat on lowest-spec supported device
