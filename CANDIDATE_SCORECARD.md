# Kibo SDET Challenge — Candidate Assessment Scorecard

**Candidate:** [Submission from Downloads folder]
**Date:** 2026-03-30
**Reviewer:** Automated Assessment
**Assessment Type:** Take-Home Submission Review

---

## Executive Summary

**Overall Rating:** ⭐⭐⭐ **HIRE** (Mid-level SDET) — **with reservations on Task 6**

**Key Strengths:**
- ✅ Clean, functional Testing SDK architecture
- ✅ Proper elimination of Thread.Sleep() with generic polling utility
- ✅ Good Builder pattern implementation with fluent API
- ✅ Thoughtful AI usage documentation with critical evaluation
- ✅ Environment-driven configuration support

**Critical Weaknesses:**
- ❌ **Task 6 (Observability) incomplete** — logs captured but never shown when tests fail, defeating the entire purpose
- ❌ Only 4 tests total — minimal edge case coverage
- ❌ No test isolation strategy (shared tenant across tests)
- ⚠️ Missing validation in builder (accepts invalid inputs silently)
- ⚠️ No custom assertions — standard xUnit assertions provide zero diagnostic context

---

## Take-Home Scoring (1–5 scale)

### 1. Code Reuse & Architecture: **4/5** ✅ Strong

**Evidence:**
- ✅ Clean separation: `ApiClient` in `Kibo.TestingFramework` project
- ✅ Generic `ApiResponse<T>` wrapper with metadata (status, timing, logs)
- ✅ Reusable client with configurable base URL and default tenant
- ✅ Proper `IDisposable` implementation for HttpClient lifecycle
- ✅ Could be published as NuGet package with minor additions (interfaces)

**Deductions:**
- ⚠️ No interface abstraction (`IApiClient`) for dependency injection
- ⚠️ Models duplicated between framework and API project (acceptable but noted)

**Code Quality Indicators:**
```csharp
// Good: Generic, reusable client pattern
public class ApiClient : IDisposable
{
    private readonly HttpClient _http;
    public Uri BaseUri { get; }

    public async Task<ApiResponse<OrderDto>> CreateOrderAsync(...)
    public async Task<ApiResponse<OrderDto>> GetOrderAsync(...)
}

// Good: Observable responses with diagnostics
public class ApiResponse<T>
{
    public T? Body { get; init; }
    public HttpStatusCode StatusCode { get; init; }
    public string RequestLog { get; init; } = string.Empty;
    public string ResponseLog { get; init; } = string.Empty;
    public long ElapsedMs { get; init; }
}
```

---

### 2. Design Patterns: **4/5** ✅ Strong

**Evidence:**
- ✅ **Builder Pattern:** `OrderBuilder` with fluent API and method chaining
- ✅ **Fluent API design:** Chainable methods as required (`.WithCustomerEmail().WithItems().ForTenant()`)
- ✅ Sensible defaults: Default email and single line item
- ✅ Multiple build methods: `WithCustomerEmail()`, `WithItems(int)`, `WithLineItems()`, `ForTenant()`
- ✅ Clean, readable builder interface

**Note:** Instructions asked for "Fluent API" (method chaining design pattern), NOT "FluentAssertions" library. Candidate correctly implemented fluent builder pattern. Standard xUnit assertions are perfectly acceptable.

**Deductions:**
- ⚠️ Builder doesn't validate inputs (e.g., `WithItems(0)` or `WithCustomerEmail("")` accepted)
- ⚠️ No interface or base class for future builders (e.g., `LineItemBuilder`)

**Code Quality Indicators:**
```csharp
// Good: Fluent builder with sensible defaults
public class OrderBuilder
{
    public OrderBuilder()
    {
        _order.CustomerEmail = "test@kibo.local";
        _order.LineItems = new List<LineItemDto>
        {
            new LineItemDto { ProductCode = "SKU-001", Quantity = 1, UnitPrice = 9.99m }
        };
    }

    public OrderBuilder WithCustomerEmail(string email) { ... return this; }
    public OrderBuilder WithItems(int count) { ... return this; }
    public OrderDto Build() => _order;
}

// Usage in tests is clean:
var builder = new OrderBuilder()
    .WithCustomerEmail("john.doe@example.com")
    .WithItems(1)
    .ForTenant("tenant-abc-123");
```

---

### 3. Resiliency (Polling): **5/5** ⭐ Exceptional

**Evidence:**
- ✅ Generic `Poller.WaitUntilAsync<T>` utility — fully reusable
- ✅ Configurable interval and timeout with sensible defaults
- ✅ Returns last observed state in timeout exception message
- ✅ Handles condition exceptions gracefully (try-catch in polling loop)
- ✅ Uses `Task.Delay` instead of `Thread.Sleep` (async-friendly)
- ✅ Uses `Stopwatch` for accurate timing

**Outstanding Implementation:**
```csharp
public static class Poller
{
    public static async Task<T> WaitUntilAsync<T>(
        Func<Task<T>> action,
        Func<T, bool> condition,
        TimeSpan? interval = null,
        TimeSpan? timeout = null)
    {
        var pollInterval = interval ?? TimeSpan.FromMilliseconds(500);
        var pollTimeout = timeout ?? TimeSpan.FromSeconds(15);
        var sw = Stopwatch.StartNew();
        T last = default!;

        while (sw.Elapsed < pollTimeout)
        {
            last = await action();
            try { if (condition(last)) return last; }
            catch { /* swallow condition exceptions */ }
            await Task.Delay(pollInterval);
        }

        throw new TimeoutException($"Polling timed out after {pollTimeout}. Last observed state: {last}");
    }
}
```

**Usage in tests:**
```csharp
var ready = await Poller.WaitUntilAsync(
    action: async () => await client.GetOrderAsync(createdId, tenant: builder.TenantId),
    condition: r => r.Body != null && r.Body.Status == "ReadyForFulfillment",
    interval: TimeSpan.FromMilliseconds(500),
    timeout: TimeSpan.FromSeconds(15)
);
```

**Potential enhancements (not required but would be exceptional):**
- Exponential backoff support
- Pluggable strategies (fixed vs. exponential vs. jitter)
- Max attempts parameter

---

### 4. CI/CD Readiness: **3/5** ✅ Competent

**Evidence:**
- ✅ Environment variable support: `KIBO_BASE_URL` env var with fallback
- ✅ Tests are deterministic (no hardcoded timing assumptions)
- ✅ Configurable base URL per test or globally

**Missing CI/CD features:**
- ❌ No test categorization/traits (`[Trait("Category", "Smoke")]` for selective CI execution)
- ❌ No configuration file support (appsettings.json, .env files)
- ⚠️ Base URL fallback to `TestServerFixture` — unclear if this runs in-memory or requires server

**Code:**
```csharp
// Good: Environment-driven configuration
private string BaseUrl => Environment.GetEnvironmentVariable("KIBO_BASE_URL") ?? _server.BaseUrl;
```

**Recommendations for improvement:**
- Add `[Trait("Category", "Integration")]` to polling test
- Add `[Trait("Category", "Smoke")]` to fast tests
- Document `dotnet test --filter "Category=Smoke"` usage

---

### 5. Observability: **2/5** ⚠️ Needs Significant Work

**Evidence:**
- ✅ Request/response logging **captured** in `ApiResponse.RequestLog` and `ApiResponse.ResponseLog`
- ✅ Timing metrics: `ElapsedMs` property for performance validation
- ✅ Full request details: method, URL, headers, body
- ✅ Full response details: status, headers, body

**Critical Gap - Task 6 NOT completed:**
- ❌ **Logs are captured but NEVER shown when tests fail**
- ❌ **No custom assertions that include diagnostic context in failure messages**
- ❌ Tests use standard xUnit `Assert.Equal()` which shows "Expected: Created, Actual: 500" with **zero diagnostic context**
- ❌ When assertion fails, the captured `RequestLog` and `ResponseLog` are **invisible** - making failures useless to diagnose

**Example of the problem:**
```csharp
// Current code:
var resp = await client.CreateOrderAsync(order);
Assert.Equal(HttpStatusCode.Created, resp.StatusCode);  // ❌ Failure shows no context

// Test fails with:
// Expected: Created
// Actual: InternalServerError
// ^^^ The RequestLog and ResponseLog exist but are never seen!
```

**What's missing:**
- ❌ No correlation IDs for request tracking
- ❌ No custom assertions that output logs on failure (e.g., `AssertStatusCode()` helper)
- ❌ No xUnit output helper integration to write diagnostics
- ❌ No automatic log dumping on test failure
- ❌ No structured logging (JSON format for log aggregation)

**Task 6 Goal:** "Make test failures easy to diagnose without re-running" - **NOT ACHIEVED**. The infrastructure exists but is completely unused in actual test assertions.

**Code:**
```csharp
// Good: Captured diagnostics available for test assertions
private static string BuildRequestLog(HttpRequestMessage req, string? body)
{
    var sb = new StringBuilder();
    sb.AppendLine($"{req.Method} {req.RequestUri}");
    foreach (var h in req.Headers)
        sb.AppendLine($"{h.Key}: {string.Join(',', h.Value)}");
    if (!string.IsNullOrEmpty(body))
        sb.AppendLine(body);
    return sb.ToString();
}
```

**Potential improvements:**
- Add correlation ID generation and header injection
- Add `ILogger` integration for structured logging
- Add toggleable console output for debugging

---

### 6. GenAI Fluency: **4/5** ✅ Strong

**Evidence:**
- ✅ **5 documented prompts** with tool, prompt summary, and outcome
- ✅ Critical evaluation: "Used the output as checklist", "Modified the generated example"
- ✅ Diverse tools: GitHub Copilot, ChatGPT, Copilot Chat
- ✅ Specific use cases: anti-patterns, builder scaffold, poller pattern, edge case brainstorming, diagnostics
- ✅ Clear workflow: scaffold → adapt → modify → integrate

**Deductions:**
- ⚠️ Prompts are summarized, not verbatim (harder to evaluate prompt quality)
- ⚠️ Limited iteration evidence (mostly one-shot prompts → modification)

**AI Usage Examples:**
```markdown
## Prompt 2 — Task 2 (Builder)
**Tool:** ChatGPT
**Prompt (summary):** "Generate a fluent OrderBuilder with sensible defaults and the ability to add random line items."
**Outcome:** Accepted the skeleton and adapted the `WithItems(int)` method to provide deterministic, simple random data suited for tests. Kept chaining style and sensible defaults.

## Prompt 5 — Task 6 (HttpClient diagnostics)
**Tool:** ChatGPT
**Prompt (summary):** "How to capture request/response and timing for HttpClient calls in tests?"
**Outcome:** Used the suggested approach (capture request/response bodies around `SendAsync` and record elapsed time) in `ApiClient`. Logging is off by default but exposed via `ApiResponse.RequestLog` and `ApiResponse.ResponseLog`.
```

**Assessment:**
- Shows **practical AI workflow** as a scaffolding and brainstorming tool
- Demonstrates **critical judgment** by modifying outputs
- Could improve by showing **iterative prompting** and **rejection examples**

---

### 7. Code Quality: **4/5** ✅ Strong

**Evidence:**
- ✅ Clean build with no warnings (verified via dotnet build)
- ✅ Consistent naming conventions (PascalCase for public, _camelCase for private)
- ✅ Proper async/await usage throughout
- ✅ Modern C# features: init-only properties, nullable reference types
- ✅ Reasonable project structure
- ✅ Good encapsulation (private fields, public properties)

**Minor issues:**
- ⚠️ No XML documentation comments (acceptable for internal framework)
- ⚠️ `default!` null-forgiving operator in Poller (acceptable but noted)
- ⚠️ No explicit null checks in builder methods

**Code Quality Indicators:**
```csharp
// Good: Modern C# with init-only properties
public class ApiResponse<T>
{
    public T? Body { get; init; }
    public HttpStatusCode StatusCode { get; init; }
    public long ElapsedMs { get; init; }
}

// Good: Proper async patterns
public async Task<ApiResponse<OrderDto>> CreateOrderAsync(OrderDto order, string? tenant = null)
{
    var sw = Stopwatch.StartNew();
    var resp = await _http.SendAsync(req);
    sw.Stop();
    // ...
}
```

---

### 8. Edge Case Tests: **2/5** ⚠️ Needs Significant Work

**Critical Gap:** Only **4 tests total**, far below the expected 5+ edge case tests.

**Tests Implemented (with quality issues):**
1. ⚠️ `CreateOrder_ReturnsSuccess` — Happy path, but has useless assertion (`ElapsedMs >= 0` always true) and incomplete validation
2. ❌ `CreateOrder_WithoutTenantHeader_Returns401` — **Accidentally works but doesn't verify header is absent**
3. ✅ `GetOrder_AfterCreation_StatusBecomesReadyForFulfillment` — Polling behavior (correct)
4. ✅ `GetOrder_WithInvalidId_Returns404` — Not found case (correct)

**Test Quality Issues:**
- ❌ **Test 2 is brittle:** Relies on `null` being whitespace in `AddTenantHeader()` check, doesn't verify header absence
- ⚠️ **Test 1 has meaningless assertion:** `Assert.True(resp.ElapsedMs >= 0)` — ElapsedMs is `long`, always ≥ 0
- ⚠️ **Incomplete validation:** Tests don't verify response body fields match request (ID, tenant, lineItems, email)

**Missing Edge Cases (from PROMPT_LOG brainstorming but not implemented):**
- ❌ Empty line items array
- ❌ Negative pricing
- ❌ Long strings / large payloads
- ❌ Unicode/encoding edge cases
- ❌ Malformed JSON
- ❌ Concurrent requests (race conditions)
- ❌ Boundary values (e.g., max items, zero quantity)
- ❌ Invalid email formats
- ❌ Schema validation (unexpected fields in response)

**Documentation:**
```markdown
## Prompt 4 — Task 4 (Edge Case Brainstorm)
**Tool:** GitHub Copilot Chat
**Prompt (summary):** "List 5 destructive or edge-case tests for a simple orders API (headers, payloads, sizes, encodings)."
**Outcome:** Generated ideas (missing tenant header, empty line items, negative pricing, long strings, unicode). Used these to plan additional tests (not all implemented due to time).
```

**Assessment:**
- AI was used to **brainstorm** edge cases (strong signal)
- Ideas were **documented** but **not implemented**
- Suggests time constraint rather than lack of understanding
- Need to see live coding to confirm edge case thinking

---

## Take-Home Average: **3.4/5** ✅ Competent (Hire Tier)

| Dimension | Score | Weight | Notes |
|-----------|-------|--------|-------|
| Code Reuse & Architecture | 4/5 | High | Clean SDK design |
| Design Patterns | 4/5 | High | Fluent builder implemented correctly |
| Resiliency (Polling) | 5/5 | High | Exceptional implementation |
| CI/CD Readiness | 3/5 | Medium | Env vars supported, missing test traits |
| Observability | 2/5 | Medium | **Logs captured but never shown on failure** |
| GenAI Fluency | 4/5 | High | Thoughtful usage with critical evaluation |
| Code Quality | 4/5 | High | Clean, modern C# |
| Edge Case Tests | 2/5 | High | Only 4 tests, minimal coverage |
| **Weighted Average** | **3.4/5** | — | Adjusted from 3.6 due to observability gap |

---

## Red Flags / Concerns

### ⚠️ Moderate Concerns (address in live session):
- [ ] **Test coverage is minimal** — only 4 tests, edge cases brainstormed but not implemented
- [ ] **Test quality issues** — Test 2 is brittle (doesn't verify header absence), Test 1 has useless assertion
- [ ] **Incomplete validation** — Tests don't verify response body matches request data
- [ ] **No test isolation strategy** — all tests use same tenant "tenant-abc-123" (parallel execution risk)
- [ ] **Builder validation missing** — can create invalid orders (e.g., empty email, 0 items)
- [ ] **Limited defensive testing** — no schema validation, malformed JSON handling, or injection tests
- [ ] **Observability incomplete (Task 6)** — logs captured but never output on test failure; defeats entire purpose

### ✅ No Critical Red Flags:
- [x] Code compiles and tests pass
- [x] Prompt log is genuine and thoughtful (not fabricated)
- [x] AI usage shows critical evaluation, not blind copy-paste
- [x] Can likely explain their code fluently (good architecture)
- [x] No Thread.Sleep() — resiliency requirement met

---

## Green Flags / Positive Signals

- [x] ✅ **Exceptional polling implementation** — generic, configurable, includes last state in timeout
- [x] ✅ **Clean architecture** — separation of concerns, reusable SDK
- [x] ✅ **Modern C# patterns** — async/await, nullable reference types, init-only properties
- [x] ✅ **Thoughtful AI usage** — scaffold, evaluate, modify workflow clearly documented
- [x] ✅ **Environment-driven config** — supports CI/CD override via env vars
- [x] ✅ **Observable responses** — request/response logging and timing available
- [x] ✅ **Fluent builder API** — easy to use, sensible defaults
- [x] ✅ **Honest documentation** — acknowledges time constraints and unimplemented ideas

---

## Live Session Recommendations

### Focus Areas (prioritize these tasks):

1. **Task A: Custom Assertion Helper** ⭐⭐⭐ **HIGHEST PRIORITY**
   - **Critical gap:** Logs captured but never shown on test failure
   - Task 6 (Observability) infrastructure exists but completely unused
   - Need to build assertions that output `RequestLog` and `ResponseLog` on failure
   - This directly tests understanding of "self-diagnosing failures" requirement
   - **Perfect live coding task to reveal if they understand the problem**

2. **Task B: Test Isolation** ⭐ CRITICAL
   - Candidate used same tenant across all tests
   - Assess understanding of parallel execution risks
   - Evaluate ability to implement `IAsyncLifetime` pattern

3. **Task H: AI-Assisted Implementation** ⭐ CRITICAL
   - Strong AI usage in take-home, verify it's natural in real-time
   - Watch for: prompt quality, critical evaluation, integration with existing patterns
   - Confirm AI is part of workflow, not just for assessment

4. **Task G: Negative Builder Validation** ⭐ IMPORTANT
   - Builder accepts invalid inputs silently
   - Probe thinking: "Should builder validate or let API reject?"
   - Assess engineering judgment and trade-off analysis

### Skip These Tasks (already demonstrated):
- ~~Task C: Exponential Backoff~~ — Polling is already exceptional, unnecessary
- ~~Task A: Custom Assertion Helper~~ — Defer unless extra time remains

### Discussion Questions (must ask):

#### CI/CD Integration:
- "You have environment variable support — how would you integrate this into GitHub Actions?"
- "If tests grow to 5 minutes, what would you do?"

#### Scale & Maintenance:
- "If Kibo had 20 endpoints instead of 2, how would your framework scale?"
- "You have `ApiClient` — how would you add `InventoryClient` or `ShipmentClient`?"

#### Test Isolation:
- "I see all tests use 'tenant-abc-123'. What happens if I run tests in parallel?"
- "How would you isolate tests from each other?"

#### Edge Case Thinking:
- "You brainstormed 5 edge cases but implemented 1. Walk me through your prioritization."
- "If you had 30 more minutes, which edge case would you implement first and why?"

#### AI Workflow:
- "Your prompt log shows ChatGPT for the builder. If you had to redo that prompt, what would you change?"
- "Have you ever had AI generate code that looked good but was subtly wrong? What happened?"
- "If a teammate submitted 100% AI-generated test code, what would you say in code review?"

---

## Preliminary Rating: **CONDITIONAL HIRE** → Live Session Required

### Decision: **PROCEED TO LIVE INTERVIEW**

**This is a "make or break" live session.** The take-home shows both exceptional strengths (polling) and critical gaps (observability purpose). The live session will definitively determine if this is a time constraint or fundamental understanding gap.

### Rationale:
- **Technical ceiling is HIGH:** The polling utility (5/5) shows senior-level engineering thinking
- **AI fluency is genuine:** Practical workflow with critical evaluation, not performative
- **Critical unknown:** Does candidate understand the PURPOSE of observability, or just the mechanics?
- **Low risk:** 45-60 minutes of Task A will immediately reveal the answer

### Live Session = HIGH STAKES Decision Point

**🎯 Task A (Custom Assertion Helper) is the CRITICAL probe:**

**If they immediately understand the problem:** ✅ **→ STRONG HIRE**
> "Oh! The logs are captured but never shown when assertions fail. I need to build an `OrderAssert.HasStatus()` helper that wraps `Assert.Equal()` and outputs `RequestLog` and `ResponseLog` in the exception message."

**If they need 1-2 hints to connect the dots:** ✅ **→ HIRE**
> *After hint: "What happens when Assert.Equal fails?"* → "Ah, just 'Expected X, Actual Y' with no context. I should output the diagnostics too."

**If they struggle even with coaching:** ❌ **→ NO HIRE**
> *After multiple hints, still doesn't understand why logs should be in assertions* → Fundamental SDET understanding gap

### Live Session Success Criteria:

**MUST PASS (Non-negotiable):**
- ✅ **Solves Task A with ≤2 hints** — Connects "captured logs" to "usable on failure"
- ✅ Can explain their code without hesitation (ownership verification)
- ✅ Uses AI naturally during live coding (workflow verification)

**SHOULD PASS (Hire tier):**
- ✅ Identifies test isolation problem when prompted (systems thinking)
- ✅ Takes hints well and iterates (coachability)
- ✅ Considers edge cases and trade-offs proactively (engineering maturity)

### Upgrade to "Strong Hire" if:
- ✅ Solves Task A **independently with zero hints** (immediately sees the observability gap)
- ✅ Proactively suggests improvements: "We could also add correlation IDs to the logs"
- ✅ Solves test isolation independently with minimal hints
- ✅ Demonstrates exceptional AI prompting and critical evaluation in real-time
- ✅ Shows architectural thinking ("If I were building this for 20 endpoints...")

### Downgrade to "No Hire" if:
- ❌ **Cannot solve Task A even with full coaching ladder** (fundamental gap)
- ❌ Cannot explain why test isolation matters even with coaching
- ❌ AI usage feels forced or performative rather than natural
- ❌ Resists coaching or becomes defensive when gaps are identified
- ❌ Cannot explain their own take-home code fluently

### Expected Outcome Probability:
- **Strong Hire:** 30% (if Task A is solved immediately + test isolation understood)
- **Hire:** 50% (if Task A solved with 1-2 hints + demonstrates coachability)
- **No Hire:** 20% (if Task A requires full coaching ladder or ownership concerns)

**Bottom line:** The polling utility proves they have senior-level potential. The observability gap tests if they understand SDET fundamentals or just implement requirements mechanically. **Live session will reveal which.**

---

## Notes for Interviewer

**Candidate Strengths to Leverage:**
- Excellent polling implementation shows strong engineering fundamentals
- Clean architecture suggests good code organization instincts
- AI usage documentation shows metacognition and honesty

**Candidate Gaps to Probe:**
- Test coverage gap is significant but documented as time constraint
- Test isolation missing entirely — need to assess if this is knowledge gap or oversight
- Builder validation missing — assess engineering judgment about where validation belongs

**Coaching Opportunities:**
- Test isolation via tenant-per-test pattern
- Defensive testing and security mindset (schema validation, injection)
- Test categorization for CI/CD selective execution

**Overall Assessment:**
This is a **solid mid-level SDET submission** with strong fundamentals but gaps in defensive testing and scale thinking. The polling utility alone demonstrates senior-level thinking. The test coverage gap is concerning but documented as time-limited — live session will reveal whether this is genuine time constraint or lack of edge case thinking.

**Recommendation:** Proceed to live interview with focus on test isolation, defensive testing, and AI workflow verification.

---

## Appendix: Code Samples for Reference

### ApiClient.cs (Lines 43-72)
```csharp
public async Task<ApiResponse<OrderDto>> CreateOrderAsync(OrderDto order, string? tenant = null)
{
    var json = JsonSerializer.Serialize(order, _jsonOptions);
    var req = new HttpRequestMessage(HttpMethod.Post, "v1/orders")
    {
        Content = new StringContent(json, Encoding.UTF8, "application/json")
    };
    AddTenantHeader(req, tenant);

    var sw = Stopwatch.StartNew();
    var resp = await _http.SendAsync(req);
    sw.Stop();

    var body = await resp.Content.ReadAsStringAsync();
    OrderDto? parsed = null;
    try { parsed = JsonSerializer.Deserialize<OrderDto>(body, _jsonOptions); } catch { }

    var requestLog = BuildRequestLog(req, json);
    var responseLog = BuildResponseLog(resp, body);

    return new ApiResponse<OrderDto>
    {
        Body = parsed,
        StatusCode = resp.StatusCode,
        RequestLog = requestLog,
        ResponseLog = responseLog,
        ElapsedMs = sw.ElapsedMilliseconds
    };
}
```

### Poller.cs (Lines 9-32)
```csharp
public static async Task<T> WaitUntilAsync<T>(Func<Task<T>> action, Func<T, bool> condition,
    TimeSpan? interval = null, TimeSpan? timeout = null)
{
    var pollInterval = interval ?? TimeSpan.FromMilliseconds(500);
    var pollTimeout = timeout ?? TimeSpan.FromSeconds(15);

    var sw = Stopwatch.StartNew();
    T last = default!;
    while (sw.Elapsed < pollTimeout)
    {
        last = await action();
        try
        {
            if (condition(last))
                return last;
        }
        catch
        {
            // swallow condition exceptions — keep polling until timeout
        }
        await Task.Delay(pollInterval);
    }
    throw new TimeoutException($"Polling timed out after {pollTimeout}. Last observed state: {last}");
}
```

### OrderTests.cs (Lines 46-67) — Polling Usage
```csharp
[Fact]
public async Task GetOrder_AfterCreation_StatusBecomesReadyForFulfillment()
{
    using var client = new ApiClient(BaseUrl, defaultTenant: "tenant-abc-123");

    var builder = new OrderBuilder().WithCustomerEmail("status-check@example.com").WithItems(1).ForTenant("tenant-abc-123");
    var order = builder.Build();

    var create = await client.CreateOrderAsync(order, tenant: builder.TenantId);
    Assert.Equal(HttpStatusCode.Created, create.StatusCode);
    var createdId = create.Body!.Id;

    // Poll until the API reports the expected status instead of sleeping
    var ready = await Poller.WaitUntilAsync(
        action: async () => await client.GetOrderAsync(createdId, tenant: builder.TenantId),
        condition: r => r.Body != null && r.Body.Status == "ReadyForFulfillment",
        interval: TimeSpan.FromMilliseconds(500),
        timeout: TimeSpan.FromSeconds(15)
    );

    Assert.Equal(HttpStatusCode.OK, ready.StatusCode);
    Assert.Equal("ReadyForFulfillment", ready.Body?.Status);
}
```

---

**End of Assessment**
