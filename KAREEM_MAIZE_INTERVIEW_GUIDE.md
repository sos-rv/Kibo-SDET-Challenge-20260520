# Kareem Maize — SDET Interview Guide

**Candidate:** Kareem Maize  
**Position:** Senior Development Engineer in Test (SDET)  
**Interview Date:** [To be scheduled]  
**Interviewer:** [Your name]

---

## Executive Summary

**Resume Profile:** 12 years experience, Senior Full Stack Engineer at Walmart (2022-2025), claims expertise in Playwright, CI/CD, AI tools, and test automation.

**Take-Home Assessment:** 3.4/5 — Conditional Hire (Mid-Level)
- ✅ **Strengths:** Exceptional polling utility, clean architecture, good AI workflow
- ❌ **Critical Gaps:** Task 6 (Observability) incomplete, only 4 tests, weak test quality

**Interview Goal:** Determine if gaps are **time constraints** or **fundamental understanding issues**. Verify resume claims match demonstrated skills.

---

## 🚨 CRITICAL RED FLAGS TO INVESTIGATE

### 1. Resume vs. Assignment Mismatch

**Resume Claims:**
> "12 years designing and implementing automated unit and integration tests"  
> "Proven experience building and maintaining Playwright-based UI automation"  
> "Integrated automated suites into CI/CD... owning pipeline reliability"

**Assignment Reality:**
- Only **4 tests** (vs. requirement of 5+ edge cases)
- **No security tests** despite resume claiming experience with "reducing flaky test failures by 30%"
- **Task 6 (Observability) incomplete** — logs captured but never shown when tests fail
- **Test quality issues** — useless assertions, brittle tests

**⚠️ Interview Focus:** Probe depth of testing experience. Is the resume inflated?

---

### 2. Task 6 Observability — Built But Never Used

**What He Did:**
```csharp
public class ApiResponse<T>
{
    public string RequestLog { get; init; }  // ✅ Captured
    public string ResponseLog { get; init; }  // ✅ Captured
    public long ElapsedMs { get; init; }
}
```

**What He Didn't Do:**
```csharp
// Tests fail with ZERO diagnostic context
Assert.Equal(HttpStatusCode.Created, resp.StatusCode);
// Failure shows: "Expected: Created, Actual: 500"
// RequestLog and ResponseLog are INVISIBLE ❌
```

**Critical Question:** Does he understand the **PURPOSE** of observability, or just the mechanics of capturing logs?

**Resume Claims:**
> "Implemented observability pipelines via OpenTelemetry and Grafana, reducing MTTR by 40%"

**Reality Check:** If he truly owned observability pipelines, why didn't he connect captured logs to test failures?

---

### 3. Edge Case Coverage — 4 Tests vs. Resume Claims

**Resume Claims:**
> "Increased automated test coverage to 93%"  
> "Leveraged Generative AI to auto-generate test cases... improving test coverage completeness"

**Assignment Reality:**
- **4 tests total** (far below 5+ requirement)
- **No security tests** (SQL injection, XSS, DoS)
- **No destructive tests** (empty cart, negative pricing)
- **Brainstormed edge cases but didn't implement them**

**Critical Question:** If he's experienced at building comprehensive test suites, why so few tests?

---

## 📋 INTERVIEW STRUCTURE (45-60 minutes)

### Phase 1: Warm-Up & Ownership Verification (5-10 min)

**Goal:** Verify he wrote the code and understands his decisions

**Questions:**

1. **"Walk me through your TestingFramework architecture. What are the main classes and how do they relate?"**
   - ✅ **Strong Answer:** Fluently describes ApiClient → ApiResponse → OrderBuilder → Poller without hesitation
   - ❌ **Red Flag:** Needs to look at code, can't explain design decisions

2. **"What was the hardest decision you made during the take-home?"**
   - ✅ **Strong Answer:** Discusses trade-offs (e.g., "Should I use DelegatingHandler vs. HttpClient wrapper?")
   - ❌ **Red Flag:** "Everything was straightforward" (suggests shallow thinking or copy-paste)

3. **"Is there anything you'd refactor if you had another hour?"**
   - ✅ **Strong Answer:** Volunteers observability gap, test isolation, edge case coverage
   - ❌ **Red Flag:** "It's production-ready as-is" (lack of self-awareness)

---

### Phase 2: Task A — Custom Assertion Helper (10-15 min) ⭐ CRITICAL

**Goal:** Determine if Task 6 gap is **time** or **understanding**

**Setup (say to candidate):**
> "Your tests assert on order status using `Assert.Equal(HttpStatusCode.Created, resp.StatusCode)`. When this fails in CI, the output is just 'Expected: Created, Actual: 500' with no diagnostic context. You built a great observability infrastructure (RequestLog, ResponseLog, ElapsedMs) but it's never shown when assertions fail. Can you build a custom assertion helper that outputs diagnostics on failure?"

**What You're Looking For:**

**✅ STRONG HIRE Signal (solves in <5 min with 0-1 hints):**
> "Oh! I see the problem — the logs exist but aren't part of the assertion failure message. I need to wrap `Assert.Equal()` in a helper that outputs RequestLog and ResponseLog when the assertion fails."

```csharp
public static class OrderAssert
{
    public static void HasStatusCode(ApiResponse<OrderDto> response, HttpStatusCode expected)
    {
        if (response.StatusCode != expected)
        {
            var diagnostics = $@"
Expected: {expected}
Actual: {response.StatusCode}
Elapsed: {response.ElapsedMs}ms

Request:
{response.RequestLog}

Response:
{response.ResponseLog}";
            
            throw new XunitException(diagnostics);
        }
    }
}
```

**✅ HIRE Signal (solves with 1-2 hints):**
- After hint: *"What happens when `Assert.Equal()` fails? What information does it show?"*
- Response: "Ah, just 'Expected X, Actual Y' with no context. I should output the diagnostics in the exception message."

**❌ NO HIRE Signal (struggles even with coaching):**
- Doesn't connect "captured logs" to "usable on failure"
- Suggests logging to console instead of assertions
- Can't implement even after full explanation

**Decision Point:**
- **Pass Task A quickly** → Gaps were time constraint → **HIRE**
- **Fail Task A** → Fundamental understanding gap → **NO HIRE**

---

### Phase 3: Resume Deep Dive (15-20 min)

**Goal:** Verify resume claims match demonstrated skills

#### A. Testing Experience Claims

**Resume:**
> "Implemented and maintained automated test suites (Playwright, Cypress, Jest) integrated into CI/CD pipelines; increased automated test coverage to 93%"

**Questions:**

1. **"You mention 93% test coverage at Walmart. Walk me through how you achieved that. What was your testing strategy?"**
   - ✅ **Strong:** Describes test pyramid, unit vs integration split, coverage tools
   - ❌ **Red Flag:** Vague answer, can't explain methodology

2. **"Your take-home had 4 tests. You brainstormed 5 edge cases (SQL injection, empty cart, etc.) but didn't implement them. Walk me through your prioritization."**
   - ✅ **Strong:** Honest about time constraint, explains priority (happy path → negative cases)
   - ❌ **Red Flag:** Defensive, blames assignment, doesn't acknowledge gap

3. **"You claim experience with 'reducing flaky test failures by 30%'. What causes flaky tests, and how did you fix them?"**
   - ✅ **Strong:** Discusses timing issues, polling vs. sleep, test isolation, environment setup
   - ❌ **Red Flag:** Generic answer, can't give specific examples

---

#### B. CI/CD & Observability Claims

**Resume:**
> "Integrated automated UI and API test suites into GitHub Actions/Jenkins pipelines, owning pipeline reliability... reduced CI pipeline failures by 32%"

**Questions:**

1. **"Your assignment captured logs but didn't output them when tests failed. In a real CI environment, how would you make test failures self-diagnosing?"**
   - ✅ **Strong:** Discusses custom assertions, xUnit output helper, log aggregation
   - ❌ **Red Flag:** "I'd re-run the test locally" (defeats purpose of CI diagnostics)

2. **"You mention 'owning pipeline reliability' at Walmart. Walk me through a time when a CI pipeline was failing intermittently. How did you diagnose and fix it?"**
   - ✅ **Strong:** Specific story with root cause analysis, fix implementation, metrics
   - ❌ **Red Flag:** Vague answer, can't provide specifics

3. **"You implemented observability pipelines via OpenTelemetry and Grafana. How would you apply that experience to this testing framework?"**
   - ✅ **Strong:** Discusses correlation IDs, distributed tracing, log aggregation
   - ❌ **Red Flag:** Can't connect resume experience to assignment

---

#### C. AI Tools & Proactive Learning

**Resume:**
> "Leveraged Generative AI (GitHub Copilot/OpenAI) to auto-generate test cases and accelerate test script development, cutting test authoring time by 25%"

**Questions:**

1. **"Your PROMPT_LOG shows good AI usage. Walk me through Prompt 5 where you used ChatGPT for HttpClient diagnostics. Why did you choose that approach?"**
   - ✅ **Strong:** Discusses prompting strategy, evaluation criteria, what was rejected
   - ❌ **Red Flag:** Can't remember, suggests logs were fabricated

2. **"If you were building this framework from scratch tomorrow using AI tools, what would you do differently? Where would AI save the most time?"**
   - ✅ **Strong:** Specific ideas (e.g., "AI for edge case generation, boilerplate reduction")
   - ❌ **Red Flag:** Generic answer, "AI can do everything"

3. **"You claim 25% time savings from AI test generation. How do you verify AI-generated tests actually test what they claim to?"**
   - ✅ **Strong:** Discusses code review, mutation testing, coverage analysis
   - ❌ **Red Flag:** "I just run the tests" (no critical evaluation)

---

### Phase 4: Technical Probes (10-15 min)

**Goal:** Assess depth of SDET knowledge beyond the assignment

#### Scenario 1: Test Isolation

**Question:**
> "I notice all your tests use the same tenant 'tenant-abc-123'. If I run your tests in parallel using xUnit's default parallelization, what happens? How would you fix it?"

**✅ Strong Answer:**
- Identifies the problem: shared tenant = test pollution
- Proposes solution: unique tenant per test via `IAsyncLifetime`
- Bonus: Discusses `IClassFixture` vs `IAsyncLifetime` trade-offs

**❌ Weak Answer:**
- "Just disable parallel execution" (avoids the problem)
- Can't explain why shared state is problematic

---

#### Scenario 2: Security Testing

**Resume Claims:**
> "12 years experience", "Senior engineer"

**Question:**
> "You didn't implement any security tests in the assignment. If I asked you to add 3 security tests right now, what would you test and why?"

**✅ Strong Answer:**
- SQL injection in tenant header
- XSS via email field (Unicode, script tags)
- DoS via oversized payloads
- Explains OWASP Top 10 awareness

**❌ Weak Answer:**
- Generic answers ("test for hacking")
- Can't name specific attack vectors

---

#### Scenario 3: CI/CD Integration

**Question:**
> "Walk me through how you'd integrate this test suite into a GitHub Actions pipeline. What stages would you define? How would you handle flaky tests?"

**✅ Strong Answer:**
- Build → Unit Tests → Integration Tests → E2E Tests
- Test categorization (`[Trait]`) for selective execution
- Retry logic for flaky tests, but identifies root cause
- Environment-specific configuration

**❌ Weak Answer:**
- "Just run `dotnet test`"
- No mention of stages, parallelization, or failure handling

---

## 🎯 DECISION MATRIX

### STRONG HIRE (Upgrade from Mid-Level to Senior)

**Criteria:**
- ✅ Solves Task A **immediately** (0 hints, <5 minutes)
- ✅ Volunteers observability gap before you mention it
- ✅ Can fluently explain all resume claims with specific examples
- ✅ Demonstrates senior-level thinking (test isolation, security, CI/CD strategy)
- ✅ Shows proactive learning (discusses latest testing trends, AI tools)

**Outcome:** Offer at Senior SDET level

---

### HIRE (Mid-Level as Scorecard Suggests)

**Criteria:**
- ✅ Solves Task A with **1-2 hints** (<10 minutes)
- ✅ Honest about time constraint causing gaps
- ✅ Can explain resume claims but with less depth than expected
- ✅ Shows coachability (takes hints well, iterates quickly)
- ✅ Demonstrates growth potential

**Outcome:** Offer at Mid-Level SDET with mentorship plan

---

### BORDERLINE (Additional Interview or Rejection)

**Criteria:**
- ⚠️ Solves Task A with **full coaching ladder** (>10 minutes)
- ⚠️ Resume claims don't match demonstrated skills
- ⚠️ Can't explain specific examples from resume
- ⚠️ Defensive about gaps rather than reflective

**Outcome:** Second interview with another engineer OR polite rejection

---

### NO HIRE

**Criteria:**
- ❌ **Cannot solve Task A even with coaching**
- ❌ Resume appears inflated (can't substantiate claims)
- ❌ Lacks fundamental SDET understanding (test isolation, observability purpose)
- ❌ Defensive, uncoachable, or dishonest

**Outcome:** Polite rejection

---

## 📊 EVALUATION SCORECARD

Use this during the interview:

| Dimension | Score (1-5) | Notes |
|-----------|-------------|-------|
| **Ownership Verification** | ___ | Can explain code fluently? |
| **Task A Performance** | ___ | Hints needed: 0 / 1-2 / >3 |
| **Resume Accuracy** | ___ | Claims match demonstrated skills? |
| **Testing Depth** | ___ | Test strategy, coverage, flaky tests |
| **CI/CD Knowledge** | ___ | Pipeline design, failure handling |
| **Security Awareness** | ___ | Can name attack vectors, OWASP |
| **AI Fluency** | ___ | Practical usage, critical evaluation |
| **Coachability** | ___ | Takes hints well, iterates quickly |
| **Self-Awareness** | ___ | Volunteers gaps, seeks feedback |
| **Proactive Learning** | ___ | Stays current with tools, trends |

**Overall Recommendation:**
- [ ] **Strong Hire** (Senior SDET)
- [ ] **Hire** (Mid-Level SDET)
- [ ] **Borderline** (Second interview needed)
- [ ] **No Hire** (Polite rejection)

---

## 🔍 SPECIFIC QUESTIONS BY FOCUS AREA

### A. Smart & Curious Engineer

**Goal:** Assess problem-solving and learning mindset

1. **"What's the most interesting testing problem you've solved in the last year? Walk me through your thought process."**
   - Looking for: Curiosity, systematic approach, learning from challenges

2. **"What testing tools or techniques have you learned in the past 6 months? How are you staying current?"**
   - Looking for: Continuous learning, follows industry trends, experiments with new tools

3. **"If you could redesign your take-home framework, what would you change and why?"**
   - Looking for: Self-reflection, improvement mindset, awareness of trade-offs

---

### B. Independent Engineer

**Goal:** Assess autonomy and decision-making

1. **"Tell me about a time when you had to make a significant technical decision without much guidance. How did you approach it?"**
   - Looking for: Independent research, weighs trade-offs, documents decisions

2. **"Your assignment had time constraints. How did you prioritize what to implement vs. skip?"**
   - Looking for: Clear decision framework, risk-based prioritization, honest assessment

3. **"When you encounter a flaky test in CI, what's your debugging process?"**
   - Looking for: Systematic troubleshooting, doesn't need hand-holding, documents findings

---

### C. Works Closely with Dev Engineers

**Goal:** Assess collaboration and communication skills

1. **"Describe a time when you had to convince a dev team to improve test coverage. How did you approach it?"**
   - Looking for: Persuasion through data, collaborative approach, builds relationships

2. **"How do you balance being a quality advocate vs. not blocking development velocity?"**
   - Looking for: Pragmatic trade-offs, understands business context, partner mindset

3. **"If a developer pushes back on writing tests for their feature, how do you handle it?"**
   - Looking for: Empathy, education, finds win-win solutions

---

### D. Automation & CI/CD Expertise

**Goal:** Assess pipeline ownership and automation skills

1. **"Walk me through how you'd integrate this test suite into a CI/CD pipeline. What would your GitHub Actions workflow look like?"**
   - Looking for: Stages, parallelization, env vars, failure handling, test categorization

2. **"You mention 'owning pipeline reliability' on your resume. What metrics do you track to measure pipeline health?"**
   - Looking for: Flakiness rate, execution time, failure rate, coverage trends

3. **"How do you handle environment-specific configuration in your tests (local vs. CI vs. staging)?"**
   - Looking for: Environment variables, config files, Docker, test categories

---

### E. Proactive with Latest Tech & AI

**Goal:** Assess staying current and AI fluency

1. **"How are you using AI tools in your day-to-day testing work? Give me specific examples."**
   - Looking for: Practical usage (not hype), critical evaluation, workflow integration

2. **"If I gave you access to GPT-4 and asked you to generate 20 edge case tests for this orders API, how would you prompt it? How would you verify the output?"**
   - Looking for: Specific prompting strategy, validation approach, doesn't blindly trust AI

3. **"What's your take on AI-powered test generation tools like GitHub Copilot for Tests, Diffblue, or Codium AI? Have you tried them?"**
   - Looking for: Awareness of tools, experimentation mindset, nuanced opinion (not "AI will replace SDETs")

---

## 🚩 RED FLAGS DURING INTERVIEW

Watch for these warning signs:

### Critical Red Flags (Immediate No Hire)
- [ ] Cannot explain their own take-home code
- [ ] Resume claims demonstrably false (e.g., "93% coverage" but can't explain how)
- [ ] Fails Task A even with full coaching
- [ ] Defensive or dishonest when discussing gaps
- [ ] Plagiarized AI prompt log (can't explain prompts)

### Warning Signs (Probe Deeper)
- [ ] Vague answers without specific examples
- [ ] Blames tools/time/requirements instead of reflecting
- [ ] Can't articulate trade-offs or design decisions
- [ ] No questions for interviewer (lack of curiosity)
- [ ] Over-reliance on buzzwords without substance

---

## ✅ GREEN FLAGS (POSITIVE SIGNALS)

Watch for these strong signals:

- [ ] Volunteers gaps before you mention them ("I wish I'd added...")
- [ ] Asks clarifying questions before coding (Task A)
- [ ] Discusses trade-offs unprompted ("I chose X over Y because...")
- [ ] Demonstrates continuous learning (mentions recent experiments)
- [ ] Shows empathy for users/developers ("From a dev perspective...")
- [ ] Iterates quickly when given hints (coachability)
- [ ] Can articulate when **not** to use AI (critical thinking)

---

## 📝 POST-INTERVIEW ACTIONS

### If STRONG HIRE:
1. Extend offer at **Senior SDET** level
2. Highlight observability and test coverage as growth areas
3. Pair with senior engineer for first 2 weeks

### If HIRE (Mid-Level):
1. Extend offer at **Mid-Level SDET**
2. Create mentorship plan focusing on:
   - Test observability best practices
   - Security testing (OWASP Top 10)
   - Test isolation strategies
3. Set 3-month goals for edge case coverage improvement

### If BORDERLINE:
1. Schedule second interview with another engineer
2. Focus on:
   - Task isolation (IAsyncLifetime implementation)
   - Security test implementation (live coding)
   - Resume verification (specific examples)

### If NO HIRE:
1. Send polite rejection email
2. Provide constructive feedback if requested:
   - "Focus on building comprehensive test suites"
   - "Deepen understanding of test observability"
   - "Practice connecting infrastructure to actionable outputs"

---

## 🎬 INTERVIEW CLOSING

**Always ask the candidate:**

1. **"What questions do you have for me about the role, team, or Kibo's testing practices?"**
   - Strong candidates ask thoughtful questions about:
     - Team structure and collaboration
     - Testing philosophy and coverage expectations
     - Tools and tech stack
     - Growth opportunities

2. **"If you join Kibo, what would you want to work on first?"**
   - Looking for: Proactive thinking, alignment with role, realistic expectations

3. **"Is there anything from your take-home you'd like to walk me through that we haven't discussed?"**
   - Gives candidate chance to highlight strengths or explain gaps

---

## FINAL RECOMMENDATION

Based on take-home assessment (3.4/5) and resume claims:

**Primary Concern:** Resume claims ("12 years", "93% coverage", "observability pipelines") don't match assignment execution (4 tests, Task 6 incomplete).

**Interview Mission:** Determine if this is:
- **Scenario A:** Time constraint (capable engineer, just rushed) → **HIRE**
- **Scenario B:** Inflated resume (skills don't match claims) → **NO HIRE**

**Decision Point:** **Task A is make-or-break**. If solved quickly, gaps were time. If struggled, gaps are fundamental.

**Recommended Outcome:** 
- 60% probability: **HIRE at Mid-Level** (time constraint confirmed)
- 30% probability: **NO HIRE** (fundamental gaps revealed)
- 10% probability: **STRONG HIRE** (exceeds expectations in interview)

---

**Good luck with the interview!**
