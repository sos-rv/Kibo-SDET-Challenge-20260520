# Eric Syed — Senior SDET Interview Guide

**Candidate:** Eric Syed  
**Position:** Senior Development Engineer in Test (SDET)  
**Interview Date:** [To be scheduled]  
**Interviewer:** [Your name]  
**Take-Home Score:** 4.9/5 ⭐⭐⭐⭐⭐ **STRONG HIRE**

---

## 🎯 Executive Summary

**Profile:** Exceptional take-home submission demonstrating senior-level SDET capabilities

**Key Achievements:**
- ✅ **DelegatingHandler pattern** — Industry-standard HttpClient middleware (not just wrapper)
- ✅ **12 comprehensive tests** — 6 destructive edge cases including SQL injection with professional bug reports
- ✅ **Complete Task 6** — Full observability with correlation IDs, timing, toggleable logging
- ✅ **Security-first mindset** — CRITICAL bug report with Risk/Impact/Recommendation structure
- ✅ **Exceptional AI fluency** — 6 prompts showing iteration, critical evaluation, rejection examples

**Interview Goal:** 
1. **Verify ownership** (not copy-paste from AI/senior engineer)
2. **Probe depth beyond breadth** (can he architect at scale?)
3. **Assess senior-level thinking** (trade-offs, team leadership, production mindset)
4. **Confirm AI fluency is genuine** (not performative for assessment)

**Expected Outcome:** **Strong Hire** → Senior SDET with potential for tech lead/principal track

---

## 🚀 INTERVIEW STRATEGY (Different from Mid-Level)

### Eric vs. Kareem: Different Approaches

| Aspect | Kareem (Mid-Level) | Eric (Senior) |
|--------|-------------------|---------------|
| **Goal** | Verify gaps are time, not understanding | Verify ownership and probe depth |
| **Focus** | Task A is make-or-break | Architecture, scaling, leadership |
| **Tasks** | Custom assertions (observability gap) | Schema validation, backoff strategies |
| **Discussion** | Basic CI/CD, test isolation | Team frameworks, API versioning, production debugging |
| **Outcome** | Hire if passes Task A | Confirm senior level, explore principal potential |

**Why Different:**
- Eric's take-home is **already exceptional** — no critical gaps to fix
- Focus on **how he thinks**, not what he knows
- Probe **architectural decisions** and **team impact**
- Assess **leadership potential** and **production experience**

---

## 📋 SESSION STRUCTURE (50-60 minutes)

| Phase | Duration | Focus |
|-------|----------|-------|
| **Warm-Up & Ownership** | 5-10 min | Verify he wrote it, understand decisions |
| **Live Task 1** | 10-12 min | **Task D: Schema Validation** (security depth) |
| **Live Task 2** | 10-12 min | **Task B: Test Isolation** OR **Task C: Backoff** (systems thinking) |
| **Architecture Discussion** | 15-20 min | Scaling, versioning, team frameworks, production |
| **AI Fluency Deep Dive** | 5-8 min | Verify iterative workflow is genuine |

**Note:** Skip Task A (Custom Assertions) — he already has exceptional observability with DelegatingHandler.

---

## PHASE 1: Warm-Up & Ownership Verification (5-10 min)

**Goal:** Confirm he wrote the code and made deliberate architectural choices

### Question 1: Architecture Walkthrough

**Ask:**
> "Walk me through your TestingFramework architecture. I see you used DelegatingHandler for observability — why that pattern instead of a simpler HttpClient wrapper?"

**What You're Looking For:**

**✅ STRONG (confirms ownership):**
> "I chose DelegatingHandler because it's the standard .NET middleware pattern for HttpClient. It sits in the pipeline between the application and the HTTP call, so I can inject diagnostics without changing the client API. The alternative was wrapping every `SendAsync` call manually, but that couples observability to the client implementation. DelegatingHandler is composable — you can chain multiple handlers for auth, retry, logging without tight coupling."

**Probing Questions:**
- "What trade-offs did DelegatingHandler introduce?"
- "When would you NOT use DelegatingHandler?"

**❌ RED FLAG (possible copy-paste):**
- Can't explain why DelegatingHandler vs. wrapper
- Vague answer: "It was just the right pattern"
- Defensive: "The AI suggested it"

---

### Question 2: Hardest Decision

**Ask:**
> "What was the hardest technical decision you made during the take-home? Walk me through your thought process."

**What You're Looking For:**

**✅ STRONG:**
- Specific decision with trade-offs discussed
- Examples:
  - "Choosing where to attach diagnostics — headers vs. in-memory vs. global state"
  - "Deciding whether to validate builder inputs or let API reject them"
  - "Whether to make `WithScenarioEmail()` a switch statement or dictionary lookup"

**Probing:**
- "What alternatives did you consider?"
- "If you had to redo that decision, would you change it?"

**❌ RED FLAG:**
- "Everything was straightforward" (lack of critical thinking)
- Can't articulate trade-offs

---

### Question 3: What Would You Refactor?

**Ask:**
> "If you had another 2 hours, what would you refactor or add?"

**What You're Looking For:**

**✅ STRONG (self-awareness):**
- Volunteers improvements without being asked:
  - "Base URL is still hardcoded — I'd add `KIBO_API_BASE_URL` env var"
  - "Test isolation could be better with `IAsyncLifetime` for unique tenants per test"
  - "I'd add test categorization (`[Trait]`) for selective CI execution"
  - "Schema validation could be extracted to a reusable helper"

**⭐ EXCEPTIONAL (senior thinking):**
- Discusses team impact:
  - "I'd create a getting-started guide for junior engineers"
  - "I'd add examples showing common patterns"
  - "I'd extract the observability handler to a separate NuGet package so other teams could reuse it"

**❌ RED FLAG:**
- "It's production-ready as-is" (lack of humility)
- Mentions trivial refactors (variable naming)

---

### Question 4: AI Usage Deep Dive

**Ask:**
> "Your Prompt Log shows 6 AI interactions with great detail. Walk me through Prompt 1 where you debugged the API client. You mentioned 'rejected PostAsJsonAsync' — tell me about that decision."

**What You're Looking For:**

**✅ STRONG (genuine AI workflow):**
> "The AI suggested three approaches. I kept `StringContent` because the legacy tests used specific serialization, and I wanted backward compatibility. I rejected `PostAsJsonAsync` because it uses different JSON serialization defaults — it would have worked, but if the API had specific casing expectations, it might break. I tested both locally and confirmed `StringContent` matched the legacy behavior exactly."

**Probing:**
- "How do you decide when to trust AI output vs. when to validate it?"
- "Have you ever had AI generate code that looked perfect but was subtly wrong?"

**⭐ EXCEPTIONAL:**
- Gives specific example of AI hallucination and how they caught it
- Discusses testing AI output: "I always write a failing test first to verify the AI's approach actually works"

**❌ RED FLAG:**
- Can't remember details (suggests fabricated prompt log)
- Defensive about AI usage

---

## PHASE 2: Live Task 1 — Schema Validation (10-12 min)

**Why This Task for Eric:**
- He has **security mindset** (SQL injection test) — probe depth
- **Task D (Schema Validation)** tests security awareness + architectural thinking
- More challenging than Task A (which he already mastered)

**Setup:**

> "Your tests validate specific fields like `status` and `id`, but they don't validate the response *shape*. If the API suddenly started returning an extra field like `internalDebugToken` containing sensitive data (SSN, API keys), your tests would pass but you'd have a security leak. How would you add response schema validation to catch unexpected fields?"

**What They Should Build:**

```csharp
public static void AssertSchema(JsonElement actual, params string[] expectedFields)
{
    var actualFields = actual.EnumerateObject().Select(p => p.Name).ToHashSet();
    var expectedSet = expectedFields.ToHashSet();
    
    var missing = expectedSet.Except(actualFields);
    var unexpected = actualFields.Except(expectedSet);
    
    if (missing.Any() || unexpected.Any())
        throw new XunitException(
            $"Schema mismatch.\n" +
            $"Missing: [{string.Join(", ", missing)}]\n" +
            $"Unexpected: [{string.Join(", ", unexpected)}]");
}

// Usage:
var responseJson = JsonDocument.Parse(responseBody).RootElement;
SchemaAssert.Matches(responseJson, "id", "status", "customerEmail", "lineItems", "tenantId");
```

**Evaluation:**

| Score | Behavior |
|-------|----------|
| **5 — Exceptional** | Solves independently in <5 min, discusses security implications ("unexpected fields are the risk"), makes it generic for any endpoint |
| **4 — Strong** | Solves with 1 hint, implements both missing and unexpected field checks |
| **3 — Competent** | Needs 2 hints, only checks missing fields (not unexpected) |
| **2 — Weak** | Needs full coaching ladder, manual property checks instead of systematic |

**Probing Questions (if solved quickly):**
- "Which is more important for security — missing fields or unexpected fields?" (Answer: unexpected — leaking data)
- "How would you make this work for nested objects?" (Tests JSON traversal knowledge)
- "Should this be in the framework or test project?" (Answer: framework, reusable)

---

## PHASE 3: Live Task 2 — Choose Based on Task 1 Performance

### Option A: Task B (Test Isolation) — If Task 1 was 4-5/5

**Why:**
- Eric already has `IClassFixture` — but tests share tenant "t1"
- Tests **parallel execution safety** understanding
- Probes **xUnit lifecycle knowledge** (`IAsyncLifetime`)

**Setup:**

> "Your tests use `IClassFixture` with a shared tenant 't1'. If xUnit runs test classes in parallel (which it does by default), orders from one test class could leak into another if they share the same tenant. How would you add test isolation?"

**Expected Solution:**

```csharp
public class OrderTestBase : IAsyncLifetime
{
    protected KiboApiClient Client { get; private set; }
    protected string TenantId { get; private set; }

    public Task InitializeAsync()
    {
        TenantId = $"test-{Guid.NewGuid():N}";
        Client = new KiboApiClient(BaseUrl, TenantId, enableLogging: false);
        return Task.CompletedTask;
    }

    public Task DisposeAsync()
    {
        Client?.Dispose();
        return Task.CompletedTask;
    }
}

public class OrderTests : OrderTestBase
{
    [Fact]
    public async Task CreateOrder_ReturnsSuccess()
    {
        var order = OrderBuilder.Default
            .WithScenarioEmail("happy-path")
            .WithItems(1)
            .Build();
        
        // TenantId is unique per test class
        var response = await Client.CreateOrderAsync(order);
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

**Evaluation:**

| Score | Behavior |
|-------|----------|
| **5 — Exceptional** | Immediately identifies problem, proposes `IAsyncLifetime`, discusses trade-offs (class-level vs. test-level isolation) |
| **4 — Strong** | Identifies problem with 1 hint, implements `IAsyncLifetime` correctly |
| **3 — Competent** | Needs hint about tenant collisions, implements solution with guidance |

**Follow-Up Discussion:**
- "What's the trade-off between test-level and class-level isolation?" 
  - (Class-level is faster but less isolated; test-level is safer but slower)
- "How would you handle tests that NEED to share state?" 
  - (Use `IClassFixture` with unique tenant, or sequential execution)

---

### Option B: Task C (Exponential Backoff) — If Task 1 was 3/5 or below

**Why:**
- Eric's polling is already excellent — but adding backoff tests **extensibility thinking**
- Probes **algorithm design** and **API design** (pluggable strategies)

**Setup:**

> "Your polling utility uses a fixed 500ms interval. In a real CI environment with 50 parallel tests hitting a shared staging API, that fixed rate could overload the server. Can you add exponential backoff?"

**Expected Solution:**

```csharp
public static async Task<T> WaitUntilAsync<T>(
    Func<Task<T>> operation,
    Func<T, bool> condition,
    TimeSpan? interval = null,
    TimeSpan? timeout = null,
    double backoffMultiplier = 1.0,  // 1.0 = fixed, 2.0 = exponential
    TimeSpan? maxInterval = null)
{
    interval ??= TimeSpan.FromMilliseconds(500);
    timeout ??= TimeSpan.FromSeconds(15);
    maxInterval ??= TimeSpan.FromSeconds(5);
    
    var stopwatch = Stopwatch.StartNew();
    var currentInterval = interval.Value;
    T? lastResult = default;

    while (stopwatch.Elapsed < timeout)
    {
        lastResult = await operation();
        if (condition(lastResult)) return lastResult;
        
        await Task.Delay(currentInterval);
        
        // Apply backoff with cap
        currentInterval = TimeSpan.FromMilliseconds(
            Math.Min(
                currentInterval.TotalMilliseconds * backoffMultiplier, 
                maxInterval.Value.TotalMilliseconds
            )
        );
    }

    var lastStateJson = JsonSerializer.Serialize(lastResult);
    throw new TimeoutException($"Polling failed after {timeout}. Last state: {lastStateJson}");
}
```

**Evaluation:**

| Score | Behavior |
|-------|----------|
| **5 — Exceptional** | Solves independently, discusses load implications ("reduces API load by 60%"), suggests pluggable strategy pattern |
| **4 — Strong** | Implements with 1 hint, adds cap correctly, handles edge cases (multiplier = 1) |
| **3 — Competent** | Needs 2 hints, implements basic backoff without cap |

**Advanced Follow-Up (if 5/5):**
- "How would you make the backoff strategy pluggable so users can choose fixed OR exponential OR custom?"
  - (Answer: `Func<int, TimeSpan>` or strategy interface)

---

## PHASE 4: Architecture Discussion (15-20 min) ⭐ CRITICAL

**Goal:** Assess **senior-level systems thinking** and **team leadership** potential

### Discussion Area 1: Scaling to 20 Endpoints

**Ask:**
> "You built this framework for 2 endpoints (Orders, potentially Users). If Kibo had 20 API endpoints across 5 domains (Orders, Inventory, Shipping, Payments, Users), how would your framework scale? What would you change?"

**What You're Looking For:**

**✅ STRONG (Senior):**
- **Client per domain:** `OrderApiClient`, `InventoryApiClient`, `PaymentApiClient`
- **Shared base class:** `KiboApiClientBase` with DelegatingHandler, common headers
- **Interface abstraction:** `IKiboApiClient<TEntity>` for dependency injection
- **Shared builder library:** `EntityBuilder<T>` base class

**⭐ EXCEPTIONAL (Principal/Lead):**
- **Package structure:** Discusses breaking into NuGet packages
  - `Kibo.TestingFramework.Core` (base client, observability, poller)
  - `Kibo.TestingFramework.Orders` (domain-specific client + builders)
- **Versioning strategy:** How to handle v1 vs v2 endpoints
- **Team adoption:** How to onboard new engineers, documentation, examples

**Probing Questions:**
- "How would you handle different auth schemes across domains?" (e.g., Orders uses API key, Payments uses OAuth)
- "What if Inventory team wants custom observability beyond your handler?"

---

### Discussion Area 2: API Versioning

**Ask:**
> "Kibo decides to launch v2 of the Orders API alongside v1 for backward compatibility. How would you handle testing both versions without duplicating your framework?"

**What You're Looking For:**

**✅ STRONG:**
```csharp
public class OrderApiClient
{
    private readonly ApiVersion _version;
    
    public OrderApiClient(string baseUrl, string tenant, ApiVersion version = ApiVersion.V1)
    {
        _version = version;
        _httpClient.BaseAddress = new Uri($"{baseUrl}/{VersionPath(_version)}");
    }
    
    private string VersionPath(ApiVersion v) => v switch
    {
        ApiVersion.V1 => "v1",
        ApiVersion.V2 => "v2",
        _ => throw new ArgumentException($"Unsupported version: {v}")
    };
}

// Usage:
var v1Client = new OrderApiClient(baseUrl, tenant, ApiVersion.V1);
var v2Client = new OrderApiClient(baseUrl, tenant, ApiVersion.V2);
```

**⭐ EXCEPTIONAL:**
- Discusses **contract testing** (v2 should pass v1 tests + new tests)
- **Shared assertions** for common behavior
- **Deprecation strategy** (how to sunset v1 tests)

---

### Discussion Area 3: Production Debugging

**Ask:**
> "A test passed in CI yesterday but fails today with a 500 error. The CI log shows your correlation ID but no other context. Walk me through your triage process using your framework's observability features."

**What You're Looking For:**

**✅ STRONG:**
1. "Find correlation ID in logs: `[a3f7c1d2]`"
2. "Check `X-Kibo-Request-Log` header: see exactly what request was sent"
3. "Check `X-Kibo-Response-Log` header: see error body"
4. "Check `X-Kibo-Elapsed-Ms`: timing might indicate backend timeout"
5. "If logging is enabled, console shows full diagnostic trace"
6. "Reproduce locally with same tenant/data using correlation ID"

**⭐ EXCEPTIONAL:**
- "Correlation ID should be searchable in centralized logs (Grafana, Splunk)"
- "I'd add tracing integration (OpenTelemetry) to link test → API → database"
- "For flaky tests, I'd log the correlation ID to a test results file for post-mortem"

**Follow-Up:**
- "How would your framework help differentiate between API regression vs. environment issue?" 
  - (Timing patterns, error patterns, retry behavior)

---

### Discussion Area 4: Team Onboarding

**Ask:**
> "A junior QA engineer joins your team tomorrow. They've never used C# or your framework. How quickly could they write a new test? What would you add to help them?"

**What You're Looking For:**

**✅ STRONG:**
- **Documentation:** README with examples
- **Templates:** Test class template with common patterns
- **Naming conventions:** Clear patterns (`CreateOrder_WithInvalidEmail_Returns400`)
- **Error messages:** Builder validation guides them to correct usage

**⭐ EXCEPTIONAL (Leadership signal):**
- "I'd pair-program their first test"
- "I'd create a getting-started workshop with 3 exercises"
- "I'd add IntelliSense XML docs on every public method"
- "I'd record a 5-minute video walkthrough"
- "I'd set up a #testing-framework Slack channel for questions"

**This reveals:** Senior engineers enable others; leads build teams.

---

### Discussion Area 5: Test Automation Strategy

**Ask:**
> "How do you decide what to automate vs. what to test manually? Walk me through your decision framework."

**What You're Looking For:**

**✅ STRONG:**
- **Automate:** Regression tests, happy paths, API contracts, performance baselines
- **Manual:** Exploratory testing, UX/visual design, one-off scenarios
- **Criteria:** ROI (execution frequency × time saved vs. maintenance cost)

**⭐ EXCEPTIONAL:**
- **Test Pyramid:** Unit (70%) → Integration (20%) → E2E (10%)
- **Risk-based:** Automate critical user journeys first
- **Maintenance burden:** "I avoid automating tests that change frequently"
- **Team skill:** "If only I can maintain the test, it's not a good automation candidate"

---

## PHASE 5: AI Fluency Deep Dive (5-8 min)

**Goal:** Verify his AI workflow is **genuine**, not performative for assessment

### Question 1: Iteration Example

**Ask:**
> "Your Prompt 6 shows 'first pass' and 'second pass' iteration. Walk me through what was wrong with the first pass and how you refined your prompt."

**What You're Looking For:**

**✅ STRONG:**
> "The first prompt was 'Move bug reports to XML docs'. The AI gave inconsistent formatting — some had 'KNOWN ISSUE:', others had 'BUG REPORT:'. I refined the prompt to 'Enforce uniform structure with Expected/Actual/Known Issue format' and gave it an example. Second pass was consistent."

**Probing:**
- "How many iterations do you typically need before accepting AI output?"
- "When do you stop iterating and just fix it manually?"

**⭐ EXCEPTIONAL:**
- Discusses **prompt engineering patterns** (few-shot examples, chain-of-thought)
- "I keep a prompt library for common patterns"

---

### Question 2: AI Trust Boundaries

**Ask:**
> "Where do you trust AI completely, and where do you always verify? Give me specific examples."

**What You're Looking For:**

**✅ STRONG:**
- **Trust:** Boilerplate code (models, builders), documentation templates, edge case brainstorming
- **Verify:** Assertions (AI hallucinates expected values), async patterns (AI forgets `await`), security logic

**Example:**
> "I trust AI for builder scaffolding — it's repetitive, hard to get wrong. I NEVER trust AI for assertions without verifying. I've had AI generate tests that looked great but asserted the wrong value or used incorrect comparison operators."

**⭐ EXCEPTIONAL:**
- "I use AI to generate test cases, then I review for business logic correctness"
- "AI is great for suggesting attack vectors (XSS payloads), but I verify against OWASP"

---

### Question 3: 100% AI-Generated PR

**Ask:**
> "A teammate submits a PR where 100% of the test code is AI-generated with no modifications. What would you say in code review?"

**What You're Looking For:**

**✅ STRONG:**
- "I'd ask them to explain the tests. If they can't, I'd reject it."
- "I'd check if assertions actually verify behavior or just check for truthy values."
- "I'd ask why they didn't modify anything — did they verify it works?"

**⭐ EXCEPTIONAL (Leadership):**
- "I'd use it as a teaching moment — walk them through reviewing AI output."
- "I'd ask them to add comments explaining WHY each assertion exists."
- "I'd pair with them on reviewing AI-generated code together."

**❌ RED FLAG:**
- "I'd approve it if the tests pass" (no critical thinking)
- "AI code is fine as long as it works" (misses quality/maintainability)

---

## 🎯 SCORING RUBRIC

### Take-Home (Already Scored: 4.9/5)

| Dimension | Eric's Score | Notes |
|-----------|--------------|-------|
| Code Reuse & Architecture | 5/5 | DelegatingHandler, production-ready |
| Design Patterns | 5/5 | Multiple patterns, scenario-driven builder |
| Resiliency (Polling) | 5/5 | JSON serialization in timeout |
| CI/CD Readiness | 4/5 | Env vars supported, missing test traits |
| Observability | 5/5 | Complete Task 6, correlation IDs |
| GenAI Fluency | 5/5 | 6 prompts, iteration, critical evaluation |
| Code Quality | 5/5 | Zero warnings, XML docs, modern C# |
| Edge Case Tests | 5/5 | 12 tests, security-focused, professional bug reports |
| **Average** | **4.9/5** | **Strong Hire** |

---

### Live Session Scoring

| Task | Score (1-5) | Notes |
|------|-------------|-------|
| **Warm-Up** | ___ | Ownership verified? Design rationale clear? |
| **Task D: Schema Validation** | ___ | Security awareness? Implementation quality? |
| **Task B/C: Isolation or Backoff** | ___ | Systems thinking? xUnit knowledge? |
| **Live Average** | ___ | — |

---

### AI Fluency Live Assessment

| Dimension | Score (1-5) | Notes |
|-----------|-------------|-------|
| Comfort with AI tools | ___ | Natural or performative? |
| Prompt quality | ___ | Contextual & iterative? |
| Critical evaluation | ___ | Reviews & modifies output? |
| AI workflow articulation | ___ | Can explain when/why/how? |
| **AI Fluency Average** | ___ | — |

---

### Architecture Discussion

| Area | Score (1-5) | Notes |
|------|-------------|-------|
| Scaling to 20 endpoints | ___ | Abstraction thinking? |
| API versioning strategy | ___ | Contract testing awareness? |
| Production debugging | ___ | Observability utilization? |
| Team onboarding | ___ | Leadership signals? |
| Test automation strategy | ___ | ROI/risk framework? |
| **Discussion Average** | ___ | — |

---

## 🏆 DECISION MATRIX

### STRONG HIRE (Senior SDET) — Expected Outcome

**Criteria:**
- ✅ Take-Home: 4.9/5 (already met)
- ✅ Live Tasks: ≥ 4.0/5 average
- ✅ Fluently explains architectural decisions
- ✅ AI workflow is genuine (can discuss failures, iteration)
- ✅ Discussion shows senior systems thinking

**Offer:** Senior SDET, top 25% of salary band for exceptional work

---

### PRINCIPAL/LEAD UPGRADE (Exceptional)

**Criteria:**
- ✅ All Strong Hire criteria met
- ✅ Live Tasks: 5/5 (solves independently, suggests improvements)
- ✅ Architecture discussion shows **team framework thinking**
- ✅ Demonstrates **leadership signals** (onboarding, mentoring, documentation)
- ✅ Discusses **production debugging at scale** (tracing, observability platforms)

**Offer:** Senior SDET with fast-track to Principal/Lead (6-12 months)

---

### HIRE (Senior with Growth Areas) — Unlikely

**Criteria:**
- ✅ Take-Home: 4.9/5
- ⚠️ Live Tasks: 3.0-3.9/5 (needs hints but reaches solution)
- ⚠️ Architecture discussion shows gaps (can't discuss versioning, scaling)

**Offer:** Senior SDET, standard salary band, mentorship in production systems

---

### BORDERLINE (Ownership Concerns) — Red Flag

**Criteria:**
- ❌ Cannot explain architectural decisions (suggests didn't write it)
- ❌ Live Tasks: < 3.0/5 (struggles with coaching)
- ❌ AI workflow is performative (can't discuss specifics)

**Action:** Additional interview to verify ownership OR polite rejection

---

## 🚨 RED FLAGS TO WATCH FOR

**Critical (Immediate Concern):**
- [ ] Cannot explain DelegatingHandler choice
- [ ] Can't recall details from PROMPT_LOG (suggests fabricated)
- [ ] Defensive when asked about decisions
- [ ] Struggles with basic xUnit concepts (`IAsyncLifetime`)
- [ ] Copy-paste AI output without understanding

**Warning Signs (Probe Deeper):**
- [ ] Vague answers about trade-offs
- [ ] Can't discuss production debugging
- [ ] No questions for interviewer
- [ ] Over-reliance on buzzwords

---

## ✅ GREEN FLAGS (CONFIRM SENIOR LEVEL)

**Strong Signals:**
- [ ] Volunteers improvements unprompted
- [ ] Asks clarifying questions before coding
- [ ] Discusses team impact ("How would junior engineers use this?")
- [ ] Mentions production implications ("This would help with incident triage")
- [ ] Shows iterative AI workflow with rejection examples
- [ ] Thinks about next developer ("I'd add XML docs for IntelliSense")

**Exceptional Signals (Principal/Lead Potential):**
- [ ] Discusses breaking framework into NuGet packages
- [ ] Mentions team onboarding strategy (workshops, videos)
- [ ] Talks about observability platforms (OpenTelemetry, Grafana)
- [ ] Shows awareness of testing economics (ROI, maintenance burden)

---

## 📝 POST-INTERVIEW ACTIONS

### If STRONG HIRE (Expected):
1. ✅ **Immediate offer** at Senior SDET level
2. ✅ Highlight observability work as exceptional
3. ✅ Discuss potential tech lead track in 12-18 months
4. ✅ Pair with architecture team for first project

---

### If PRINCIPAL/LEAD UPGRADE:
1. ✅ **Senior SDET offer** with accelerated promotion track
2. ✅ Assign to **framework/platform project** immediately
3. ✅ Discuss **team lead responsibilities** in 6 months
4. ✅ Budget for conference speaking (build profile)

---

### If HIRE (Senior with Growth):
1. ✅ Senior SDET offer, standard band
2. ⚠️ Mentorship plan: pair with principal engineer
3. ⚠️ Focus areas: production debugging, team frameworks
4. ⚠️ 3-month checkpoint on growth

---

### If BORDERLINE (Ownership Concerns):
1. ⚠️ Schedule second interview with different engineer
2. ⚠️ Live coding focus: implement test from scratch
3. ⚠️ Ask to explain specific code sections line-by-line

---

## 🎬 INTERVIEW OPENING SCRIPT

**Start of Interview:**

> "Hi Eric, thanks for joining. I reviewed your take-home submission and I'm really impressed — the DelegatingHandler pattern, the security tests, the observability work is exceptional. Today we're going to dive deeper into your thinking and probe how you'd approach this framework at scale.
>
> **AI Tools:** You're welcome to use any AI tools during the live tasks — Copilot, ChatGPT, Claude, whatever's part of your workflow. We're not testing whether you can code without help; we want to see how you work in practice.
>
> Let's start with you walking me through your framework architecture..."

---

## 🎬 INTERVIEW CLOSING SCRIPT

**End of Interview:**

> "Thanks Eric, this was a great conversation. Your take-home was exceptional, and seeing how you think about scaling and team impact confirms that. 
>
> **What questions do you have for me** about the role, team, or Kibo's testing practices?"

**Listen for:**
- Team structure questions
- Growth opportunities
- Technical challenges
- Tools and tech stack

**Final Question:**
> "If you join Kibo, what would you want to work on first in the testing platform?"

**Looking for:**
- Proactive thinking
- Alignment with senior role
- Team mindset ("I'd pair with engineers to understand pain points first")

---

## 📊 EXPECTED OUTCOME PROBABILITIES

| Outcome | Probability | Rationale |
|---------|-------------|-----------|
| **Strong Hire (Senior)** | **70%** | Take-home is exceptional; ownership likely confirmed |
| **Principal/Lead Upgrade** | **20%** | If architecture discussion shows team leadership |
| **Hire (Senior w/ Growth)** | **8%** | If live tasks show gaps in depth |
| **Borderline/No Hire** | **2%** | Only if ownership cannot be verified |

**Bottom Line:** Eric's take-home is **already Strong Hire level**. Live interview is to **confirm ownership** and **assess leadership potential**. Very low risk of rejection.

---

**Good luck with the interview! This is a rare senior-level candidate — focus on confirming they're real, then explore principal track potential.**
