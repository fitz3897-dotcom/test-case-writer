# Backend Testing Reference

This reference covers 11 backend-specific testing domains. Use it when the test target is a backend API, microservice, or server-side system.

---

## 1. API Functional Testing (API 功能测试)

### REST API Testing

For each endpoint, verify:

| Aspect | Test Points |
|---|---|
| HTTP method correctness | GET does not modify data; POST creates; PUT/PATCH updates; DELETE removes |
| Status codes | 200/201/204 for success; 400/401/403/404/409/422/429/500 for errors |
| Response body | Correct schema, data types, field presence, null handling |
| Request headers | Content-Type, Accept, Authorization, custom headers |
| Query parameters | Filtering, sorting, field selection; invalid parameters handled |
| Request body | Valid JSON/XML; empty body; malformed body |
| Idempotency | Repeated identical requests produce the same result (GET, PUT, DELETE) |

### GraphQL API Testing

| Aspect | Test Points |
|---|---|
| Query | Correct fields returned; nested objects; aliases; fragments |
| Mutation | Data modified correctly; return value matches |
| Variables | Valid and invalid variable types; missing required variables |
| Errors | Partial errors (some fields fail); query complexity limits |
| Introspection | Schema query returns correct type definitions (or is disabled in production) |
| N+1 prevention | Verify DataLoader or batching is in place for nested queries |

### Common Defects

- Missing Content-Type header causes 415 instead of a clear error
- PATCH endpoint replaces entire resource instead of partial update
- DELETE returns 200 with empty body instead of 204
- GraphQL mutation returns stale cached data

---

## 2. Authentication and Authorization (认证与授权测试)

### Authentication Test Cases

| Scenario | Expected |
|---|---|
| Valid credentials | 200 + token/session returned |
| Invalid password | 401 with generic message (not "password incorrect") |
| Non-existent user | 401 with same generic message (no user enumeration) |
| Expired token | 401 with "token expired" error code |
| Malformed token | 401 with "invalid token" error code |
| Missing Authorization header | 401 |
| Refresh token flow | New access token issued; old token invalidated (rotation) |
| Logout | Token invalidated; subsequent requests return 401 |
| Multi-device session | Login from device B doesn't invalidate device A (unless policy requires it) |

### Authorization Test Cases

| Scenario | Expected |
|---|---|
| User accesses own resource | 200 |
| User accesses another user's resource | 403 |
| User accesses admin-only endpoint | 403 |
| Admin accesses user resource | 200 (if policy allows) |
| Deleted/suspended user's token | 401 or 403 |
| IDOR: replace resource ID in URL with another user's ID | 403 (not 200 with another user's data) |
| Privilege escalation: change role in request body | Ignored or 403 |
| Horizontal privilege: user A modifies user B's data | 403 |

### Security Headers to Verify

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY` or `SAMEORIGIN`
- `Strict-Transport-Security` (HSTS)
- `Cache-Control: no-store` for sensitive responses
- No sensitive data in response headers or error stack traces

---

## 3. Input Validation and Error Handling (输入校验与错误处理)

### Input Validation Matrix

| Input Type | Test Values |
|---|---|
| Required fields | Missing, null, empty string, whitespace only |
| String fields | Min length, max length, max+1, special characters, Unicode, emoji, HTML tags, SQL injection patterns |
| Numeric fields | 0, negative, MAX_INT+1, floating point, NaN, Infinity, string "abc" |
| Email | Valid formats, missing @, double @, special chars in local part, IDN domains |
| Date/time | Valid ISO 8601, invalid format, future dates, past dates, timezone handling, leap seconds |
| Enum fields | Valid values, invalid value, empty, case mismatch |
| Arrays | Empty array, single item, max items, max+1, nested arrays, duplicate items |
| Nested objects | Missing required nested fields, extra fields, deep nesting (10+ levels) |
| File upload | Valid type, invalid type, empty file, oversized file, filename with special chars |

### Error Response Format

Verify the API returns consistent error responses:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable message",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "value": "not-an-email"
      }
    ]
  }
}
```

Test that:
- Error codes are consistent across all endpoints
- Messages are helpful but don't leak internal details
- Multiple validation errors are returned together (not one at a time)
- Error responses always include the correct HTTP status code

---

## 4. Database CRUD and Data Integrity (数据库 CRUD 与数据完整性)

### CRUD Test Matrix

| Operation | Test Points |
|---|---|
| Create | Correct data stored; auto-generated fields (ID, timestamps) populated; unique constraints enforced; foreign keys valid |
| Read | Correct data returned; soft-deleted records excluded; joins return correct related data |
| Update | Only specified fields changed; updated_at timestamp refreshed; version/etag incremented |
| Delete | Record removed (or soft-deleted); cascading deletes work correctly; orphaned records handled |

### Data Integrity Tests

- **Unique constraints**: Attempt to create duplicate records on unique fields → 409 Conflict
- **Foreign key constraints**: Create record with non-existent foreign key → 400 or 422
- **Cascade behavior**: Delete parent record → verify child records are handled per policy (cascade delete, set null, or block)
- **Transaction rollback**: If multi-step operation fails midway, all changes are rolled back
- **Concurrent writes**: Two simultaneous updates to the same record → one succeeds, one gets 409 (optimistic locking) or waits (pessimistic locking)
- **Data type boundaries**: Store MAX_INT, very long strings, null in nullable fields, empty strings
- **Timezone consistency**: Timestamps stored in UTC; displayed in user's timezone
- **Soft delete**: Deleted records not returned in queries; can be restored by admin

---

## 5. Concurrency and Race Conditions (并发与竞态测试)

### Test Scenarios

| Scenario | Expected Behavior |
|---|---|
| Two users update the same record simultaneously | Optimistic lock: second request gets 409 with stale version error |
| Double-click on "Place Order" button | Only one order created; second request returns existing order or 409 |
| Two users claim the same limited resource (e.g., last ticket) | One succeeds, one gets "unavailable" error; no overselling |
| Parallel transfers from the same account | Balance never goes negative; total debits = sum of successful transfers |
| Concurrent file uploads to the same path | Last write wins with versioning, or reject with 409 |

### How to Test

1. Use load testing tools (k6, JMeter, Locust) to send concurrent requests
2. Send identical requests with minimal delay (< 10ms apart)
3. Verify final database state is consistent
4. Check for: duplicate records, incorrect balances, lost updates, deadlocks

---

## 6. Idempotency (幂等性测试)

### Idempotent Methods

| Method | Expected Behavior on Repeat |
|---|---|
| GET | Always returns same response (for same state) |
| PUT | Same result whether called once or multiple times |
| DELETE | First call deletes; subsequent calls return 404 or 204 (both acceptable) |
| POST | NOT inherently idempotent — use idempotency keys |

### Idempotency Key Testing

For non-idempotent operations (POST), test the idempotency key mechanism:

| Scenario | Expected |
|---|---|
| Same idempotency key, same body | Returns cached result from first request |
| Same idempotency key, different body | 422 (conflict) — key already used with different payload |
| Missing idempotency key | Request processed normally (or 400 if key is required) |
| Expired idempotency key (after TTL) | Request processed as new |
| Concurrent requests with same key | One processed, others wait and return same result |

---

## 7. Rate Limiting (限流测试)

### Test Scenarios

| Scenario | Expected |
|---|---|
| Below rate limit | All requests succeed (200) |
| At rate limit boundary | Last allowed request succeeds |
| Exceed rate limit | 429 Too Many Requests with `Retry-After` header |
| After cooldown period | Requests succeed again |
| Different rate limit tiers | Free tier vs premium tier limits enforced correctly |
| Rate limit per endpoint | Hitting limit on /api/search doesn't affect /api/users |
| Rate limit headers | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` present and accurate |

### How to Test

1. Send requests at a controlled rate and verify headers
2. Burst requests to trigger the limit
3. Wait for the reset window and verify recovery
4. Test from different API keys / IP addresses to verify isolation

---

## 8. Pagination and Filtering (分页与过滤测试)

### Pagination Tests

| Scenario | Expected |
|---|---|
| First page | Correct page_size items; has_next = true (if more exist) |
| Last page | Remaining items; has_next = false |
| Page beyond total | Empty results, not an error |
| page_size = 0 | 400 or default page size applied |
| page_size = -1 | 400 error |
| page_size exceeds max | Capped to max (e.g., 100) or 400 |
| Cursor-based pagination | next_cursor is valid; prev_cursor works for backward |
| Data changes during pagination | New items don't cause duplicates or missing items (cursor > offset) |
| Total count accuracy | total_count matches actual record count after filtering |

### Filtering and Sorting Tests

| Scenario | Expected |
|---|---|
| Single filter | Correct results returned |
| Multiple filters (AND) | Intersection of filter criteria |
| Filter with no matches | Empty results array, not 404 |
| Invalid filter field | 400 with helpful error message |
| Sort ascending / descending | Order is correct |
| Sort by non-sortable field | 400 or ignored with warning |
| Filter + sort + pagination | All three work together correctly |
| Search with special characters | Properly escaped, no injection |

---

## 9. Webhook and Callback Testing (Webhook/回调测试)

### Webhook Delivery Tests

| Scenario | Expected |
|---|---|
| Event triggers webhook | POST request sent to registered URL within SLA (e.g., < 5s) |
| Webhook payload | Correct event type, data, timestamp, signature |
| Signature verification | HMAC signature matches; invalid signature rejected by consumer |
| Retry on failure | If endpoint returns 5xx, retry with exponential backoff (e.g., 3 retries) |
| Endpoint timeout | If endpoint doesn't respond within timeout, mark as failed, schedule retry |
| Duplicate delivery | Consumer can handle duplicate events (idempotent processing) |
| Out-of-order delivery | Consumer handles events arriving out of sequence |
| Webhook registration | CRUD operations on webhook subscriptions work correctly |
| Event filtering | Only subscribed event types are delivered |

### Security Tests

- Webhook URL validation (no internal IP addresses — SSRF prevention)
- Signature includes timestamp to prevent replay attacks
- HTTPS required for webhook endpoints
- Secret rotation: old and new secrets both valid during rotation window

---

## 10. Microservice Integration (微服务集成测试)

### Inter-Service Communication Tests

| Scenario | Expected |
|---|---|
| Service A calls Service B successfully | Correct data flows end-to-end |
| Service B is down | Service A returns degraded response or cached data, not 500 |
| Service B responds slowly | Service A times out gracefully; circuit breaker opens after threshold |
| Service B returns unexpected schema | Service A handles gracefully (defensive parsing) |
| Circular dependency detected | No infinite loops; request ID / correlation ID propagated |
| Service discovery | New instance registered and receives traffic; deregistered instance stops receiving |

### Contract Testing

- Consumer-driven contracts: consumer specifies expected request/response; provider verifies
- Schema compatibility: adding fields is backward-compatible; removing/renaming fields is breaking
- Version negotiation: Accept-Version header or URL versioning works correctly

### Observability Tests

- Correlation ID propagated across all service calls in a request chain
- Distributed tracing: complete trace visible in tracing tool (Jaeger, Zipkin)
- Logs include correlation ID, service name, request method, response status
- Health check endpoints return correct status for dependencies

---

## 11. Message Queue Testing (消息队列测试)

### Producer Tests

| Scenario | Expected |
|---|---|
| Publish message | Message appears in queue with correct payload and metadata |
| Publish to non-existent queue/topic | Error handled gracefully (auto-create or clear error) |
| Queue is full / backpressure | Producer receives backpressure signal; retries or buffers |
| Publish with ordering key | Messages with same key delivered in order |
| Transactional publish | Message published only if database transaction commits |

### Consumer Tests

| Scenario | Expected |
|---|---|
| Consume message | Processed correctly; acknowledged |
| Duplicate message | Idempotent processing; no side effects on re-delivery |
| Poison message (invalid payload) | Moved to dead-letter queue after N retries; not blocking other messages |
| Consumer crashes mid-processing | Message re-delivered to another consumer (at-least-once) |
| Consumer lag | Alert triggered when consumer falls behind; no message loss |
| Ordering guarantee | Messages processed in correct order within partition/key |
| Delayed / scheduled messages | Delivered at the scheduled time, not before |

### Infrastructure Tests

- Queue failover: primary broker down → failover to replica with no message loss
- Scaling: adding consumers increases throughput; rebalancing doesn't duplicate processing
- Message TTL: expired messages removed or moved to dead-letter queue
- Monitoring: queue depth, consumer lag, error rate visible in dashboards
