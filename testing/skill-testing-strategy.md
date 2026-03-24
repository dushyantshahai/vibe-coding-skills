# Skill: Testing Strategy

```json
{
  "skill_id": "testing-strategy",
  "category": "Testing & Reliability",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Testing Strategy — What to Test, When, and How Much for AI Products</title>

<overview>

## What This Skill Enables
- Decide which parts of your app genuinely need tests and which do not — without over-engineering
- Build a minimal, high-value test suite that catches real bugs without slowing you down
- Understand the three types of tests and when each one earns its cost for an AI product

## Why It Matters for Vibe Coders
Most vibe coders either write zero tests (and break things constantly in production) or ask their AI agent to "write tests for everything" (and get hundreds of tests that test nothing meaningful). The right answer is in between: a small, strategic set of tests that protect the parts of your app most likely to break and most expensive to break. This skill gives you that strategy.

## When to Use This Skill
- Before writing any test — to decide what is worth testing
- When production bugs keep slipping through your manual QA
- When adding tests to a codebase that currently has none
- When your AI agent generates a new feature and you need to know what to verify

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "test_framework": "[REPLACE: e.g. Vitest / Jest / Pytest / none yet]",
    "current_test_coverage": "[REPLACE: e.g. none / only unit tests / some integration tests]",
    "most_critical_features": [
      "[REPLACE: e.g. reminder scheduling — if this breaks, users miss reminders]",
      "[REPLACE: e.g. lesson generation — most expensive AI call in the app]",
      "[REPLACE: e.g. auth — if this breaks, users cannot log in]"
    ],
    "recent_production_bugs": "[REPLACE: describe any bugs that reached users recently]",
    "feature_to_test_today": "[REPLACE: ONE feature or function only]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Testing

### Mental Model 1: The Testing Trophy
For AI products, prioritise tests in this order — not the classic pyramid:

```
        ╔═══════════════╗
        ║  E2E Tests    ║  ← Few, slow, expensive — only for critical user journeys
        ╠═══════════════╣
        ║  Integration  ║  ← Most valuable — test your API routes + DB together
        ║  Tests        ║
        ╠═══════════════╣
        ║  Unit Tests   ║  ← For pure logic only — date parsing, data transforms
        ╠═══════════════╣
        ║  Static Types ║  ← Free tests — TypeScript catches entire bug classes
        ╚═══════════════╝
```

**For most AI products at MVP stage:** Focus 80% of your test effort on integration tests. They test the most realistic scenarios with the least setup.

### Mental Model 2: The Cost-Benefit Test for Every Test
Before writing a test, ask:
1. **How often will this break?** (Higher frequency = higher value test)
2. **How bad is it when it breaks?** (User data loss > wrong count display)
3. **How long does it take to detect manually?** (Long detection time = higher value test)
4. **How hard is it to fix after the fact?** (Hard to fix = protect it with a test)

If the answer to all four is "low / not bad / quickly / easy" — skip the test. Write it later if it actually breaks.

### Mental Model 3: The AI Test Problem
AI outputs are non-deterministic — the same input can produce different outputs on different runs. This means:
- You **cannot** write traditional equality assertions for AI outputs (`expect(output).toBe("exact string")`)
- You **can** write structural assertions (`expect(output).toHaveProperty("scheduledAt")`)
- You **can** write constraint assertions (`expect(output.length).toBeLessThan(500)`)
- You **can** write semantic evaluations using a second AI call as a judge

</mental_models>

---

<system_design_breakdown>

## The Three Test Types for AI Products

### Type 1: Unit Tests
Test a single pure function in complete isolation.

```
Input → Function → Output
No database. No network. No AI calls. No side effects.
```

**Worth writing for:**
- Date/time parsing and formatting
- Data transformation functions
- Validation logic
- Price calculation
- Any function with complex conditional logic

**Not worth writing for:**
- Simple getters/setters
- Functions that just call another function
- UI component rendering (use visual review instead)

---

### Type 2: Integration Tests
Test one API route end-to-end — real database, real auth check, mocked external services.

```
HTTP Request → Auth → Validation → DB Query → Response
             (real)   (real)       (real DB)   (assert shape + status)
             AI calls → MOCKED (don't want to pay per test run)
```

**Worth writing for:**
- Every API route that handles user data
- Webhook handlers (idempotency is critical to test)
- Background job handlers
- Auth flows

---

### Type 3: End-to-End (E2E) Tests
Simulate a real user clicking through the browser.

```
Browser → Click → Form → API → DB → Response rendered in browser
```

**Only worth writing for:**
- The most critical user journey (signup → first core action)
- Payment flow (Stripe checkout to subscription active)
- Auth flow (sign up, log in, log out)

**Not worth writing for:** Every page, every button, every edge case. E2E tests are slow and brittle — they break when you change a CSS class or move a button.

---

## Test Priority Matrix for AI Products

| Feature | Test Type | Priority | Why |
|---|---|---|---|
| Auth (login, signup) | Integration | 🔴 Critical | Breaks for all users if wrong |
| API route input validation | Integration | 🔴 Critical | Security + data integrity |
| Core domain logic (scheduling, scoring) | Unit | 🔴 Critical | Pure logic, easy to test |
| AI output parsing | Unit | 🟠 High | Non-determinism makes bugs subtle |
| Webhook handler idempotency | Integration | 🟠 High | Duplicate processing = bad data |
| AI generation endpoint | Integration (mocked) | 🟠 High | Expensive per real call |
| UI component rendering | Visual review | 🟡 Medium | Better caught by eyes than tests |
| Complete user journey | E2E (1–2 only) | 🟡 Medium | Slow, only for critical path |
| Static pages | None needed | ⚪ Low | Nothing to break |

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Set up test infrastructure once. Then add tests one feature at a time. -->

## Step 1 — Set Up Your Test Framework (One Time)

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Set up [Vitest / Jest / Pytest] for my [FRAMEWORK] project.

Generate:
1. Install command for test framework + testing utilities
2. Config file: [vitest.config.ts / jest.config.ts / pytest.ini]
3. A test utilities file at /tests/utils/setup.ts with:
   - A mock for the database client (intercept DB calls in tests)
   - A mock for the AI provider (return predictable fake responses)
   - A helper to create an authenticated test request
4. A sample passing test to confirm setup works: tests/sanity.test.ts

Do not write tests for any real features yet — just the infrastructure.
```

**Verify:** Run `npm test` (or `pytest`). The sample test passes. The framework is installed.

---

## Step 2 — Identify Your Top 3 Critical Features to Test

Before writing any real test, answer these questions:

```
1. What is the ONE feature that, if broken, makes your app completely unusable?
   → This gets a test first.

2. What feature has broken in production most recently?
   → Write a regression test that would have caught it.

3. What feature involves the most complex logic (date parsing, scoring, scheduling)?
   → Unit test the core logic function.
```

Write these down. They are your first three tests.

---

## Step 3 — Write Your First Integration Test
For the most critical API route.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Test framework: [FRAMEWORK]
Test infrastructure is set up at /tests/utils/setup.ts

Write an integration test for: [ROUTE PATH AND METHOD]
File: /tests/integration/[feature].test.ts

Test these scenarios:
1. Happy path — valid input, authenticated user → expect 200/201 with correct response shape
2. Missing required field → expect 400 with error message
3. Unauthenticated request → expect 401
4. Resource not found → expect 404
5. [Any domain-specific edge case for this route]

Use the mock DB and mock auth from /tests/utils/setup.ts.
Do NOT make real AI calls — mock the AI provider to return a fixed response.
```

**Verify:** `npm test tests/integration/[feature].test.ts` — all 5 scenarios pass.

---

## Step 4 — Write Unit Tests for Core Logic Functions

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Write unit tests for this function: [FUNCTION NAME] at [FILE PATH]
Here is the function: [PASTE FUNCTION CODE]

Test these cases:
1. [Normal input] → expect [output]
2. [Edge case: empty/null input] → expect [output or error]
3. [Edge case: boundary value] → expect [output]
4. [Invalid input] → expect [thrown error or returned null]

Each test should be independent — no shared state between tests.
File: /tests/unit/[function-name].test.ts
```

**Verify:** Tests run and pass. Each test name clearly describes what it is checking.

---

## Step 5 — Add Tests as You Build (The Ongoing Practice)

After the infrastructure is set up, add to your AI agent build prompts:

```
Standard addition to every feature build prompt:
"After generating the [FEATURE] code, also generate:
1. One integration test for the happy path of the main route
2. One unit test for any pure logic functions
3. Add both to /tests/[unit|integration]/[feature].test.ts"
```

This ensures tests grow with the codebase rather than being retrofitted later.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am adding tests to my app.
Project context: [PASTE context_anchor JSON]
Test framework already set up: [YES/NO]
Tests already written: [LIST FILES]

Today I am writing tests for: [ONE FEATURE]
Before writing any test, confirm:
1. What file contains the code being tested
2. What mock utilities are available in /tests/utils/
3. Whether this requires a unit test, integration test, or both
```

### Test Infrastructure Setup Prompt
```
Set up a complete testing infrastructure for my [FRAMEWORK] project.
Project context: [PASTE context_anchor JSON]

Framework: [Vitest / Jest / Pytest]
Include:
1. Config file with correct paths for my project structure
2. /tests/utils/db.ts — mock database that records calls and returns configurable responses
3. /tests/utils/auth.ts — helper to create fake authenticated sessions
4. /tests/utils/ai.ts — mock AI provider that returns a fixed string (configurable per test)
5. /tests/utils/server.ts — test server setup for integration tests (using supertest or similar)
6. /tests/sanity.test.ts — one trivial passing test to confirm setup works

Install commands for all dependencies.
```

### Critical Path Test Prompt
```
Write tests for the most critical user journey in my app.
Project context: [PASTE context_anchor JSON]
Critical journey: [DESCRIBE — e.g. "user signs up → creates first reminder → receives confirmation"]

Write:
1. Integration test: each API call in the journey works correctly
2. Test the journey in sequence: each test's output becomes the next test's input
3. Cover: happy path + what happens if step 2 fails (does it clean up correctly?)

Mock: AI calls, email sends, any external services
Use: real database operations (test DB, rolled back after each test)
```

### Regression Test Prompt
```
A bug reached production: [DESCRIBE THE BUG]
It was caused by: [DESCRIBE ROOT CAUSE IF KNOWN]
It was fixed by: [DESCRIBE THE FIX]

Write a regression test that would have caught this bug before it shipped.
The test should:
1. Reproduce the exact conditions that caused the bug
2. Assert the correct behaviour (that the bug fix produces)
3. Have a descriptive name explaining what it guards against

File: /tests/regression/[descriptive-name].test.ts
```

### Test Coverage Audit Prompt
```
Audit my codebase for missing critical tests.
Project context: [PASTE context_anchor JSON]
Existing tests: [LIST OR DESCRIBE]

For each of these high-risk areas, tell me if a test exists and if not, what test to write:
1. All API routes in /app/api/ — are input validation and auth tested?
2. Webhook handlers — is idempotency tested?
3. Core business logic functions — are edge cases covered?
4. AI output parsing — is malformed output handled?

Return: a table of [File/Function, Test Status, Recommended Test if Missing]
Do not write any tests yet — audit only.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Do I really need tests if I am moving fast?"

Short answer: you need a small number of the right tests, not a large number of any test.

Without any tests, every deploy is a gamble. You change the reminder scheduling function for a new feature and accidentally break it for existing users — and you only find out when users complain. A single integration test on that function would have caught it in 2 seconds.

**The minimum viable test suite for an AI product:**
1. One integration test per API route (happy path + auth failure)
2. Unit tests for any function with date/time logic, scoring, or complex conditionals
3. One E2E test for sign-up through first core action

That is probably 15–25 tests total. Runs in under 30 seconds. Catches 80% of production bugs.

---

### "Should I write tests before or after the code?"

For vibe coders using AI agents: **write tests after the code is working but before moving to the next feature.** Here is why:

- Writing tests before (TDD) is valuable but requires understanding the code deeply — AI agents write TDD poorly
- Writing tests after a feature works lets you test the actual behaviour, not the theoretical behaviour
- The key is: **test before the next feature touches the same code**

The dangerous gap is "I'll add tests later" — later never comes. Lock it in before the next session.

---

### "My AI agent wrote 50 tests for a simple function. Are they all valuable?"

Probably not. AI agents have a tendency to write exhaustive tests for trivial cases (`expect(add(1, 2)).toBe(3)`) while missing the real edge cases.

After your AI agent generates tests, review them with this filter:
- Would this test actually catch a real bug? If the answer is "the function would have to be completely broken for this to fail" — delete it.
- Is there a real edge case that is NOT tested? (null inputs, empty arrays, timezone edge cases, very long strings) — add those.

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Test Suite Is Working

### Quick Health Checks
```bash
# Run all tests and see pass/fail summary
npm test                    # Vitest / Jest
pytest                      # Python

# Run tests in watch mode during development
npm test -- --watch         # Re-runs on file save

# See which lines of code are covered
npm test -- --coverage
# Aim for >70% coverage on /lib/ and /api/ directories
# Do not chase 100% — it leads to testing trivial code
```

### Test Quality Checklist
```
□ Each test has a clear, descriptive name (describes what it guards against)
□ Each test is independent (does not depend on another test running first)
□ No real AI API calls in tests (always mocked — slow + costs money)
□ No real external service calls (email, SMS, payment — always mocked)
□ Database is reset between tests (no shared state between test runs)
□ Tests run in under 60 seconds total (if slower, find what is causing it)
□ A failing test tells you exactly what broke (error message is descriptive)
```

### Common Test Infrastructure Errors

| Error | Meaning | Fix |
|---|---|---|
| `Cannot find module` in test | Import path wrong or module not installed | Check the import path; run `npm install` |
| Tests pass locally, fail in CI | Environment variable missing | Add required env vars to your CI config |
| `Database already exists` error | Test DB not reset between runs | Add `beforeEach` that truncates test tables |
| All tests fail after one fails | Tests share state (one test's output affects another) | Make each test independent; use `beforeEach` setup |
| AI mock returning wrong format | Mock not configured for this test's expected output | Check mock setup in `/tests/utils/ai.ts` — make the response configurable |

</testing_and_qa>

---

<common_patterns>

## Reusable Test Patterns

### Pattern 1: Test Utilities Setup
```typescript
// /tests/utils/setup.ts
import { vi } from "vitest";

// Mock database — records all calls, returns configurable responses
export const mockDb = {
  reminder: {
    findMany: vi.fn().mockResolvedValue([]),
    findUnique: vi.fn().mockResolvedValue(null),
    create: vi.fn().mockImplementation((args) => ({
      id: "test-id-123",
      createdAt: new Date(),
      ...args.data,
    })),
    update: vi.fn().mockImplementation((args) => args.data),
    delete: vi.fn().mockResolvedValue({ id: "test-id-123" }),
  },
};

// Mock AI provider — returns predictable fixed response
export const mockAI = {
  generateText: vi.fn().mockResolvedValue({
    success: true,
    data: { content: "Mocked AI response for testing" },
    usage: { promptTokens: 10, completionTokens: 20 },
  }),
  generateStructuredOutput: vi.fn().mockResolvedValue({
    success: true,
    data: { action: "create_reminder", task: "test task", scheduledAt: "2025-12-01T10:00:00Z" },
  }),
};

// Mock authenticated request
export function createAuthenticatedRequest(
  body: object,
  userId = "test-user-id"
): Request {
  return new Request("http://localhost/api/test", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer test-token-for-${userId}`,
    },
    body: JSON.stringify(body),
  });
}
```

### Pattern 2: Standard Integration Test Structure
```typescript
// /tests/integration/reminders.test.ts
import { describe, it, expect, beforeEach, vi } from "vitest";
import { POST } from "@/app/api/reminders/create/route";
import { mockDb, createAuthenticatedRequest } from "../utils/setup";

// Replace real DB with mock before tests run
vi.mock("@/lib/db", () => ({ db: mockDb }));
vi.mock("@clerk/nextjs/server", () => ({
  auth: () => ({ userId: "test-user-id" }),
}));

describe("POST /api/reminders/create", () => {
  beforeEach(() => {
    vi.clearAllMocks(); // Reset mock call counts between tests
  });

  it("creates a reminder with valid input", async () => {
    const req = createAuthenticatedRequest({
      text: "Call mum",
      scheduledAt: "2025-12-01T10:00:00Z",
    });

    const res = await POST(req);
    const body = await res.json();

    expect(res.status).toBe(201);
    expect(body.success).toBe(true);
    expect(body.data.id).toBeDefined();
    expect(mockDb.reminder.create).toHaveBeenCalledOnce();
  });

  it("returns 400 when text is missing", async () => {
    const req = createAuthenticatedRequest({ scheduledAt: "2025-12-01T10:00:00Z" });
    const res = await POST(req);
    expect(res.status).toBe(400);
    expect((await res.json()).success).toBe(false);
  });

  it("returns 401 when not authenticated", async () => {
    vi.mocked(require("@clerk/nextjs/server").auth).mockReturnValueOnce({ userId: null });
    const req = new Request("http://localhost/api/test", { method: "POST" });
    const res = await POST(req);
    expect(res.status).toBe(401);
  });
});
```

### Pattern 3: Unit Test for Pure Logic
```typescript
// /tests/unit/parse-reminder-date.test.ts
import { describe, it, expect } from "vitest";
import { parseReminderDate } from "@/lib/utils/parse-reminder-date";

describe("parseReminderDate", () => {
  it("parses 'tomorrow at 3pm' relative to a known date", () => {
    const baseDate = new Date("2025-01-15T12:00:00Z"); // Wednesday
    const result = parseReminderDate("tomorrow at 3pm", baseDate);
    expect(result.toISOString()).toBe("2025-01-16T15:00:00.000Z");
  });

  it("parses 'next Monday' correctly", () => {
    const baseDate = new Date("2025-01-15T12:00:00Z"); // Wednesday
    const result = parseReminderDate("next Monday", baseDate);
    expect(result.toISOString()).toBe("2025-01-20T00:00:00.000Z");
  });

  it("returns null for unparseable input", () => {
    const result = parseReminderDate("banana", new Date());
    expect(result).toBeNull();
  });

  it("handles empty string gracefully", () => {
    const result = parseReminderDate("", new Date());
    expect(result).toBeNull();
  });
});
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Use Real API Keys in Tests
```typescript
// ❌ Real key in test environment
process.env.OPENAI_API_KEY = "sk-real-key-abc123";

// ✅ Always mock — never call real services from tests
vi.mock("@/lib/openai", () => ({ generateText: vi.fn().mockResolvedValue(...) }));
```

### Rule 2: Use a Separate Test Database
Never run tests against your production or development database. Tests create and delete data — this is destructive.
```
# .env.test
DATABASE_URL="postgresql://localhost:5432/myapp_test"
# This DB is wiped clean before every test run
```

### Rule 3: Never Commit Test Credentials
Even fake/test credentials should not be committed. Use placeholder values:
```typescript
// ✅ Obvious placeholders — safe to commit
const TEST_USER_ID = "test-user-id-do-not-use-in-production";
const TEST_API_KEY = "test-key-placeholder";
```

### Rule 4: Verify Auth Is Tested in Every Integration Test
Every integration test suite for a protected route must include at minimum one test that verifies the route returns 401 when called without authentication. If this test is missing, the route may not actually be protected.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Writing Tests That Test the Mock, Not the Code
```typescript
// ❌ This tests nothing — the mock always returns what you told it to
mockDb.reminder.findMany.mockResolvedValue([{ id: "1", text: "test" }]);
const result = await mockDb.reminder.findMany();
expect(result[0].text).toBe("test"); // You just tested your own mock setup
```
**Fix:** Tests should assert behaviour of your code (the route, the function) — not the mock itself.

### ❌ Shared State Between Tests
```typescript
// ❌ Test 2 depends on data created by Test 1
it("test 1", async () => { await createReminder(...); });
it("test 2", async () => {
  const reminders = await getReminders(); // Expects Test 1's reminder to exist
  expect(reminders.length).toBe(1);      // Breaks if Test 1 doesn't run first
});
```
**Fix:** Every test sets up its own data. `beforeEach` resets state.

### ❌ Testing Implementation, Not Behaviour
```typescript
// ❌ Tests that the function was called — not that the outcome was correct
expect(mockDb.reminder.create).toHaveBeenCalled();

// ✅ Tests the actual outcome the user cares about
expect(res.status).toBe(201);
expect(body.data.reminderId).toBeDefined();
```

### ❌ Making Real AI Calls in Tests
Real AI calls in tests cost money, are slow (3–10s each), return non-deterministic results, and fail when you are offline. Always mock.

### ❌ Chasing 100% Test Coverage
100% coverage means you wrote tests for `return null` functions and one-line getters. It does not mean your app is reliable. Focus on covering the code that is complex, frequently modified, or user-facing.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Test Suite

### Add Contract Testing for AI Outputs
As your app grows, use a lightweight AI judge to evaluate whether AI outputs meet your quality bar:
```typescript
// /tests/ai-quality/lesson-generation.test.ts
it("generates a lesson with correct structure", async () => {
  const lesson = await generateLesson({ topic: "fractions", level: "beginner" });

  // Structural assertions (deterministic)
  expect(lesson).toHaveProperty("title");
  expect(lesson.sections.length).toBeGreaterThanOrEqual(3);
  expect(lesson.estimatedMinutes).toBeLessThan(30);

  // Quality assertion (use AI judge — only run in CI, not every local run)
  if (process.env.CI) {
    const evaluation = await evaluateWithAI(lesson.content, {
      criteria: "Is this appropriate for a beginner learning fractions?"
    });
    expect(evaluation.score).toBeGreaterThan(7); // out of 10
  }
});
```

### Add Visual Regression Testing
When your UI reaches a stable state, screenshot testing catches unintended visual changes:
```
Tools: Playwright (built-in screenshot comparison), Chromatic (for Storybook)
Ask your AI agent: "Add Playwright visual regression tests for the 
[reminder list / lesson card / chat interface] component."
```

### Add Load Testing Before Launch
Before your first public launch, verify your backend handles concurrent users:
```
Tool: k6 (free, scriptable)
Ask your AI agent: "Write a k6 load test that simulates 50 concurrent users 
each creating one reminder via POST /api/reminders/create over 30 seconds.
Assert: p95 response time < 500ms, error rate < 1%."
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — The Bug That Motivated Testing
**Bug:** A timezone handling bug caused reminders to be scheduled 5.5 hours late for users in IST (India Standard Time).

**Root cause:** `new Date(scheduledAt)` was being parsed as UTC and then not converted to the user's local timezone before saving.

**Test written after the fix:**
```typescript
describe("parseScheduledAt — timezone handling", () => {
  it("preserves the user's intended local time regardless of server timezone", () => {
    // User in IST says "3pm" — should be saved as 3pm IST, not 3pm UTC
    const result = parseScheduledAt("2025-01-15T15:00:00", "Asia/Kolkata");
    expect(result.toISOString()).toBe("2025-01-15T09:30:00.000Z"); // 3pm IST = 9:30am UTC
  });
});
```

**Impact:** This test ran on every subsequent deploy. The timezone bug never returned.

---

### Case Study 2: EdTech App — Minimum Test Suite That Covered 80% of Production Issues
```
After launch, 6 production bugs were tracked over 3 months.
Retrospective showed 5 of the 6 would have been caught by:

1. Integration test: POST /api/lessons/generate
   - Missing input validation test → caught "undefined topic" 500 error

2. Unit test: calculateReadingTime(content: string)
   - Missing empty string test → caught NaN display bug

3. Integration test: POST /api/auth/webhook (Clerk user.created)
   - Missing idempotency test → caught duplicate user creation on webhook retry

4. Integration test: GET /api/lessons/[id] ownership check
   - Missing cross-user access test → would have caught a IDOR vulnerability

5. Unit test: parseAILessonResponse(rawOutput: string)
   - Missing malformed JSON test → caught silent failure when AI returned partial JSON

Total tests added: 12
Time to write: ~2 hours
Production bugs prevented: 5 out of 6 (83%)
```

</real_world_examples>

</skill_document>
