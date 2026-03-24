---
name: automated-testing
description: Sets up Vitest, writes unit and integration tests, and configures GitHub Actions CI. Use when building a test suite from scratch, testing AI output parsers, or blocking broken deploys with automated pipelines.
---
# Skill: Automated Testing

```json
{
  "skill_id": "automated-testing",
  "category": "Testing & Reliability",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Automated Testing — Writing Tests That Actually Protect Your AI Product</title>

<overview>

## What This Skill Enables
- Write unit, integration, and AI-output tests that run automatically on every code change
- Set up a CI pipeline that blocks broken deployments before they reach users
- Test AI-specific behaviours — structured output parsing, prompt reliability, and workflow state — without burning API credits on every test run

## Why It Matters for Vibe Coders
Automated tests are the difference between deploying with confidence and deploying with dread. Once written, they run in seconds every time you save a file or push to GitHub — catching regressions you would not notice for days otherwise. For AI products, automated tests are doubly important because AI calls are expensive, slow, and non-deterministic. This skill shows you how to test around AI without actually calling it in every test run.

## When to Use This Skill
- When setting up your test pipeline for the first time
- When adding a new feature to an existing, tested codebase
- When you need to test AI-dependent code without making real AI calls
- When setting up CI/CD to block bad deploys automatically

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "test_framework": "[REPLACE: e.g. Vitest / Jest / Pytest]",
    "ci_platform": "[REPLACE: e.g. GitHub Actions / Vercel CI / none yet]",
    "test_database": "[REPLACE: e.g. Supabase test project / local PostgreSQL / SQLite for tests]",
    "existing_test_files": "[REPLACE: list paths of existing test files]",
    "feature_to_test_today": "[REPLACE: ONE feature only]",
    "ai_calls_in_this_feature": "[REPLACE: yes/no — does this feature call an LLM?]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Automated Testing

### Mental Model 1: Tests as a Safety Net
Imagine you are a trapeze artist. A safety net does not make the performance better — it makes it safer to try new things. Automated tests are your safety net. They do not make your app better. They let you change your app confidently, knowing the net will catch you if you fall.

The value of tests compounds over time. The first test you write has limited value. The 50th test you write is enormously valuable because it protects 49 other things simultaneously.

### Mental Model 2: The Mock Hierarchy for AI Products
```
ALWAYS MOCK:                    NEVER MOCK:
─────────────────────────────   ─────────────────────────────
AI provider API calls           Your own business logic
External service calls          Database queries (in integration tests)
  (Stripe, Resend, Twilio)      Auth checks
Email/SMS sending               Input validation
File system operations          Response shaping
```

The goal: tests that are fast (no network calls), cheap (no API credits), and deterministic (same result every run).

### Mental Model 3: The Arrange-Act-Assert Pattern
Every well-written test has exactly three sections:

```typescript
it("creates a reminder with valid input", async () => {
  // ARRANGE — set up everything the test needs
  const userId = "test-user-id";
  const input = { text: "Call mum", scheduledAt: "2025-01-15T10:00:00Z" };
  mockDb.reminder.create.mockResolvedValueOnce({ id: "new-id", ...input, userId });

  // ACT — do the one thing being tested
  const result = await createReminder(userId, input);

  // ASSERT — verify the outcome
  expect(result.success).toBe(true);
  expect(result.data.id).toBe("new-id");
  expect(mockDb.reminder.create).toHaveBeenCalledWith(
    expect.objectContaining({ data: expect.objectContaining({ userId, text: input.text }) })
  );
});
```

Never mix arranging and asserting. The act should be one line.

</mental_models>

---

<system_design_breakdown>

## Test File Organisation

```
/tests
  /unit                        ← Pure function tests (no network, no DB)
    parse-date.test.ts
    calculate-score.test.ts
    format-reminder.test.ts

  /integration                 ← API route tests (real DB, mocked external services)
    reminders.test.ts
    lessons.test.ts
    auth-webhook.test.ts

  /ai                          ← AI output validation tests (mocked AI provider)
    intent-extraction.test.ts
    lesson-parser.test.ts
    output-schema.test.ts

  /e2e                         ← Browser-based tests (Playwright)
    auth-flow.spec.ts          ← sign up + first action
    critical-path.spec.ts      ← most important user journey

  /utils                       ← Shared test helpers
    setup.ts                   ← mocks, fixtures, helpers
    db.ts                      ← test database helpers
    factories.ts               ← test data factories
```

## CI Pipeline Architecture

```
Git push to branch
      │
      ▼
┌─────────────────────────────────────┐
│  GitHub Actions / Vercel CI         │
│                                     │
│  Step 1: Install dependencies       │
│  Step 2: Type check (tsc --noEmit)  │
│  Step 3: Lint (ESLint)              │
│  Step 4: Unit tests (fast, ~10s)    │
│  Step 5: Integration tests (~30s)   │
│  Step 6: Build (next build)         │
│                                     │
│  If any step fails → block PR/deploy│
└─────────────────────────────────────┘
      │
      ▼ (only if all pass)
    Deploy to preview environment
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Set up infra → write one test → get it green → add the next. -->

## Step 1 — Set Up Test Infrastructure

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Set up Vitest (recommended) for my Next.js project.
Generate:
1. npm install command: vitest @vitest/coverage-v8 @testing-library/react jsdom
2. vitest.config.ts — configured for Next.js App Router with proper module aliases
3. /tests/utils/setup.ts — exports: mockDb, mockAI, createTestRequest(method, body, userId?)
4. /tests/utils/factories.ts — exports: createTestUser(), createTestReminder(), createTestLesson()
   These return realistic fake data objects matching my TypeScript types
5. package.json scripts:
   "test": "vitest run"
   "test:watch": "vitest"
   "test:coverage": "vitest run --coverage"

Do not write any feature tests yet. Infrastructure only.
```

**Verify:** Run `npm test`. Should output "No test files found" (not an error — just no tests yet).

---

## Step 2 — Write Your First Unit Test

Pick the single most complex pure function in your app (typically date parsing, scoring logic, or AI response parsing).

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Here is the function I want to test:
[PASTE THE FUNCTION — keep it under 30 lines]

Write unit tests in /tests/unit/[function-name].test.ts
Follow the Arrange-Act-Assert pattern.
Cover:
1. The normal/happy path (typical input → expected output)
2. Boundary cases (minimum valid input, maximum valid input)
3. Invalid/null input (should return null or throw a descriptive error)
4. At least one case that was a real or likely bug (based on the function's logic)

Each test name should complete the sentence: "it should..."
No mocks needed — this is a pure function.
```

**Verify:** `npm test tests/unit/[function-name].test.ts` — all tests pass.

---

## Step 3 — Write Integration Tests for Your Most Critical Route

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Test utilities are set up at /tests/utils/setup.ts

Write integration tests for: [ROUTE PATH]
File: /tests/integration/[feature].test.ts

For each test:
- Import the route handler directly (not via HTTP)
- Use mockDb from setup.ts to intercept database calls
- Mock auth to return a test userId
- Mock any AI calls — do not make real ones

Required test scenarios:
1. Happy path → 201/200 with correct response shape
2. Unauthenticated → 401
3. Missing required field → 400 with descriptive error
4. Resource not owned by requesting user → 404
5. Database error → 500 with generic message (not the DB error)
6. [Any domain-specific edge case relevant to this route]
```

**Verify:** `npm test tests/integration/[feature].test.ts` — all tests pass green.

---

## Step 4 — Write AI Output Tests

For any function that parses or validates AI responses.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Here is the function that parses AI output:
[PASTE PARSER FUNCTION]

Write tests in /tests/ai/[feature]-parser.test.ts

Test with these simulated AI output scenarios:
1. Perfect output — exactly what the AI should return
2. Output with extra fields — parser should ignore them
3. Output missing an optional field — parser should use default
4. Output missing a required field — parser should return null or throw
5. Completely malformed output (plain text, empty string, truncated JSON)
6. Output with the wrong types ("scheduledAt": 12345 instead of ISO string)

These tests should NOT call the real AI — use hardcoded string fixtures.
The goal: verify the parser handles every realistic AI failure mode.
```

**Verify:** `npm test tests/ai/` — all pass, including the failure case tests.

---

## Step 5 — Set Up GitHub Actions CI

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
My tests run with: npm test

Create /.github/workflows/ci.yml that:
1. Triggers on: push to any branch, pull request to main
2. Runs on: ubuntu-latest, Node.js 20
3. Steps in order:
   a. Checkout code
   b. Setup Node.js with npm cache
   c. npm ci (clean install)
   d. npx tsc --noEmit (type check)
   e. npm test (run all tests)
   f. npm run build (verify build succeeds)
4. Environment variables needed: [LIST — use GitHub Secrets for sensitive values]
5. On failure: annotate the PR with which step failed

Test database: use a test Supabase project URL from GitHub Secrets.
Do not include any real secrets in the workflow file — reference them as ${{ secrets.NAME }}.
```

**Verify:** Push a commit to a branch. Open GitHub → Actions tab. Confirm the workflow runs and passes.

---

## Step 6 — Add Coverage Gating (Optional but Recommended)
Block merges if test coverage drops below a threshold.

**Prompt your AI agent:**
```
Add coverage reporting and gating to my CI pipeline.
Threshold: fail if coverage of /lib/ and /app/api/ directories drops below 70%.
Add to vitest.config.ts:
  coverage: {
    include: ["lib/**", "app/api/**"],
    thresholds: { lines: 70, functions: 70 }
  }
Add coverage step to /.github/workflows/ci.yml after the test step.
```

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am writing automated tests for my app.
Project context: [PASTE context_anchor JSON]
Test infrastructure at: /tests/utils/setup.ts [EXISTS/NEEDS SETUP]

Today I am testing: [ONE FEATURE]
Type of test needed: [unit / integration / AI output / E2E]

Before writing tests:
1. Show me the function/route signature I will be testing
2. List the edge cases you think are worth covering
3. Identify which dependencies need to be mocked
Wait for my confirmation before writing any test code.
```

### AI Output Schema Test Prompt
```
Write tests for this AI output parser:
[PASTE PARSER CODE]

The AI is supposed to return JSON matching this schema:
[PASTE ZOD SCHEMA OR TYPE DEFINITION]

Test with these fixtures (I will write the actual AI output strings):
1. Valid complete output
2. Output with snake_case instead of camelCase (AI sometimes does this)
3. Output wrapped in markdown code fences ```json ... ``` (AI often adds these)
4. Truncated output (stream cut off mid-JSON)
5. Pure text explanation instead of JSON (AI refused or misunderstood)
6. Empty string

For cases 3 and above: parser should return null and log a warning.
File: /tests/ai/[feature]-output.test.ts
```

### Test Data Factory Prompt
```
Create a test data factory for my app.
File: /tests/utils/factories.ts

Generate factory functions for:
- createTestUser(overrides?) → User object
- createTestReminder(overrides?) → Reminder object
- createTestLesson(overrides?) → Lesson object
- createTestAIGeneration(overrides?) → AIGeneration object

Each factory:
- Returns a complete object with all required fields filled with realistic fake data
- Accepts an optional overrides object to customise specific fields
- Uses deterministic fake data (no Math.random — use fixed seeds or sequences)
- Matches the TypeScript types in /types/

Use: @faker-js/faker for realistic data OR hardcoded sensible defaults (simpler).
```

### E2E Test Prompt (Playwright)
```
Write a Playwright E2E test for the critical user journey in my app.
Project context: [PASTE context_anchor JSON]

Journey: [DESCRIBE — e.g. "User signs up with email → creates first reminder → sees it in the list"]

File: /tests/e2e/critical-path.spec.ts
Requirements:
1. Use Playwright's test and expect
2. Use data-testid attributes for element selection (not CSS classes)
3. Add these data-testid attributes to the relevant elements: [LIST]
4. Test on: Chromium only (fastest for CI)
5. Viewport: 1280x720 (desktop)
6. Handle: page navigation, form submission, loading states
7. Use a fresh test account on each run (email: test+[timestamp]@yourdomain.com)

Include the Playwright install command and playwright.config.ts.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is the difference between mocking and real integration tests?"

**Mocking** means replacing a real dependency (database, AI provider, email service) with a fake one that you control. The fake records what was called and returns whatever you configure it to return.

**Real integration** means using the actual dependency — usually the real database, but with a dedicated test database that gets wiped after each test.

**When to mock:**
- AI provider calls (expensive, slow, non-deterministic)
- External services like email, SMS, Stripe (you cannot easily test these end-to-end)
- File system operations

**When to use real (test) dependencies:**
- Database queries — using the real query language catches bugs mocks would miss
- Auth logic — close to real behaviour but with test users

**The rule:** mock external services you do not own. Use real implementations of things you do own (your database schema, your business logic).

---

### "GitHub Actions keeps failing — what should I check first?"

In order of frequency:
1. **Missing environment variable** — your test needs `DATABASE_URL` but it is not set in GitHub Secrets. Check Settings → Secrets in your repo.
2. **Test passes locally but fails in CI** — timing issue (CI is slower) or environment difference. Add `--timeout=10000` to your test command.
3. **Build fails but tests pass** — a TypeScript error that only shows at build time. Run `tsc --noEmit` locally to find it.
4. **Flaky test** — passes sometimes, fails sometimes. Usually an async timing issue. Add `await` before the assertion or increase a timeout.

---

### "How do I test AI-generated content without calling the AI?"

You test the code around the AI, not the AI itself:

1. **Test the parser** — does your code correctly handle the AI's response? Feed in hardcoded response strings as fixtures.
2. **Test the prompt builder** — does your code build the correct prompt from user input? Assert the prompt string matches expected patterns.
3. **Test the workflow** — does the orchestrator correctly pass state between steps? Mock the AI step to return a fixed output.
4. **Test the output schema** — does the response match the expected structure? Use Zod validation as an assertion.

The AI model's quality is not your responsibility to test. Your responsibility is: does your code behave correctly given any plausible AI output?

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## Testing
- Test runner: Vitest — config at vitest.config.ts
- Test structure: tests/unit/, tests/integration/, tests/ai/, tests/e2e/
- Coverage thresholds: 70% lines/functions on lib/ and app/api/
- Mock rule: always mock AI/external services; never mock own business logic
- Test factories: tests/utils/factories.ts — createUser(), createDocument(), etc.
- CI: GitHub Actions — .github/workflows/ci.yml — runs on every PR
- E2E: Playwright — tests/e2e/ — runs on deploy to staging
- AI tests: structural/constraint assertions, not exact string matching
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Running and Maintaining Your Test Suite

### Essential Commands
```bash
# Run all tests once
npm test

# Run tests in watch mode (re-runs on file save)
npm test -- --watch

# Run only tests matching a pattern
npm test -- --grep "reminder"

# Run a specific file
npm test tests/integration/reminders.test.ts

# Run with coverage report
npm test -- --coverage

# Run tests for changed files only (fast during development)
npm test -- --changed

# Python (pytest)
pytest                            # all tests
pytest tests/unit/                # specific folder
pytest -k "reminder"              # tests matching keyword
pytest --cov=app --cov-report=html # with coverage
```

### Test Health Checklist
```
□ All tests pass locally before pushing
□ No tests use real AI API calls (check for actual fetch calls to AI providers)
□ No tests share state — each test sets up and tears down its own data
□ Test names describe the expected behaviour, not the implementation
□ CI pipeline runs on every push and blocks merges on failure
□ Coverage stays above your threshold (70% for /lib/ and /app/api/)
□ Tests run in under 60 seconds total
□ No test files have TODO comments for missing test cases (write them or delete the TODO)
```

### Common CI/Test Errors

| Error | Cause | Fix |
|---|---|---|
| `Cannot find module '@/lib/...'` in CI | Path alias not configured in vitest | Add `resolve.alias` to vitest.config.ts matching tsconfig paths |
| Tests pass locally, fail in CI | Missing env var in GitHub Secrets | Add the variable to repo Settings → Secrets → Actions |
| `ECONNREFUSED` in integration tests | Test DB not accessible in CI | Use GitHub Actions service containers for PostgreSQL |
| `Test timeout after 5000ms` | Async operation too slow | Increase test timeout: `it("test", { timeout: 15000 }, async () => {})` |
| Type errors in test files | Test types not included in tsconfig | Add `/tests/**` to `include` in tsconfig.json |
| `vi is not defined` | Vitest not imported | Add `import { vi, describe, it, expect } from "vitest"` |

</testing_and_qa>

---

<common_patterns>

## Reusable Automated Test Patterns

### Pattern 1: Test Data Factories
```typescript
// /tests/utils/factories.ts
import type { User, Reminder, Lesson } from "@/types";

let counter = 0;
const nextId = () => `test-id-${++counter}`;

export function createTestUser(overrides: Partial<User> = {}): User {
  return {
    id: nextId(),
    email: `user-${counter}@test.com`,
    name: `Test User ${counter}`,
    createdAt: new Date("2025-01-01T00:00:00Z"),
    updatedAt: new Date("2025-01-01T00:00:00Z"),
    ...overrides,
  };
}

export function createTestReminder(overrides: Partial<Reminder> = {}): Reminder {
  return {
    id: nextId(),
    userId: "test-user-id",
    text: "Test reminder text",
    scheduledAt: new Date("2025-12-01T10:00:00Z"),
    isComplete: false,
    deletedAt: null,
    createdAt: new Date("2025-01-01T00:00:00Z"),
    updatedAt: new Date("2025-01-01T00:00:00Z"),
    ...overrides,
  };
}
```

### Pattern 2: Vitest Configuration for Next.js
```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./tests/utils/setup.ts"],
    coverage: {
      provider: "v8",
      include: ["lib/**", "app/api/**"],
      thresholds: { lines: 70, functions: 70 },
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./"),
    },
  },
});
```

### Pattern 3: GitHub Actions CI Workflow
```yaml
# /.github/workflows/ci.yml
name: CI

on:
  push:
    branches: ["*"]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpassword
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci

      - name: Type check
        run: npx tsc --noEmit

      - name: Run tests
        run: npm test
        env:
          DATABASE_URL: postgresql://postgres:testpassword@localhost:5432/testdb
          # Add other non-sensitive test env vars here
          # Sensitive values → GitHub Secrets: ${{ secrets.SECRET_NAME }}

      - name: Build
        run: npm run build
```

### Pattern 4: AI Output Parser Test with Fixtures
```typescript
// /tests/ai/intent-parser.test.ts
import { describe, it, expect } from "vitest";
import { parseReminderIntent } from "@/lib/ai/parsers/reminder-intent";

// Realistic AI output fixtures — no real AI calls
const FIXTURES = {
  perfect: `{"action":"create_reminder","task":"Call mum","scheduledAt":"2025-12-01T15:00:00Z"}`,
  withCodeFence: "```json\n{\"action\":\"create_reminder\",\"task\":\"Call mum\",\"scheduledAt\":\"2025-12-01T15:00:00Z\"}\n```",
  missingOptionalField: `{"action":"create_reminder","task":"Call mum"}`, // no scheduledAt
  snakeCaseKeys: `{"action":"create_reminder","task_text":"Call mum","scheduled_at":"2025-12-01T15:00:00Z"}`,
  plainTextRefusal: "I cannot understand this request. Please rephrase.",
  truncatedJson: `{"action":"create_reminder","task":"Call m`,
  emptyString: "",
  wrongTypes: `{"action":"create_reminder","task":123,"scheduledAt":"not-a-date"}`,
};

describe("parseReminderIntent", () => {
  it("parses a perfect JSON response", () => {
    const result = parseReminderIntent(FIXTURES.perfect);
    expect(result?.action).toBe("create_reminder");
    expect(result?.task).toBe("Call mum");
    expect(result?.scheduledAt).toBe("2025-12-01T15:00:00Z");
  });

  it("strips markdown code fences before parsing", () => {
    const result = parseReminderIntent(FIXTURES.withCodeFence);
    expect(result?.action).toBe("create_reminder");
  });

  it("returns null for a plain text refusal", () => {
    const result = parseReminderIntent(FIXTURES.plainTextRefusal);
    expect(result).toBeNull();
  });

  it("returns null for truncated JSON", () => {
    const result = parseReminderIntent(FIXTURES.truncatedJson);
    expect(result).toBeNull();
  });

  it("returns null for empty string", () => {
    const result = parseReminderIntent(FIXTURES.emptyString);
    expect(result).toBeNull();
  });

  it("returns null when required field has wrong type", () => {
    const result = parseReminderIntent(FIXTURES.wrongTypes);
    expect(result).toBeNull();
  });
});
```

### Pattern 5: Coverage Thresholds — Enforce Minimums in CI

Without a coverage floor, coverage will silently regress over time as new code is added without tests.

```typescript
// vitest.config.ts — add coverage thresholds
import { defineConfig } from "vitest/config"
import react from "@vitejs/plugin-react"
import path from "path"

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./tests/setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov", "html"],
      include: ["lib/**/*.ts", "app/api/**/*.ts"],  // only measure what matters
      exclude: ["lib/db/index.ts", "**/*.test.ts", "**/*.d.ts"],
      thresholds: {
        lines: 70,        // CI fails if overall line coverage drops below 70%
        functions: 70,
        branches: 60,
        statements: 70,
      }
    }
  },
  resolve: {
    alias: { "@": path.resolve(__dirname, "./") }
  }
})
```

**Coverage targets by code type:**
| Code | Target | Reason |
|---|---|---|
| Auth middleware | 90%+ | High security impact |
| Payment flows | 90%+ | Business critical |
| AI prompt builders | 80%+ | Hard to debug in prod |
| API route handlers | 70%+ | Core business logic |
| UI components | 40%+ | Lower ROI, harder to test |
| Config files | 0% | Not worth testing |

Run with: `npx vitest run --coverage` — CI fails automatically if thresholds not met.

### Pattern 6: Flaky Test Detection — Find Unreliable Tests Before They Block Shipping

AI-dependent tests and timing-sensitive integration tests are prone to intermittent failures. A test that fails 10% of the time will block CI roughly 1 in 10 PRs.

```yaml
# .github/workflows/flaky-detector.yml — run weekly to find flaky tests
name: Flaky Test Detector
on:
  schedule:
    - cron: "0 6 * * 1"  # every Monday at 6am

jobs:
  detect-flaky:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20" }
      - run: npm ci

      # Run the test suite 5 times — flag any test that fails at least once
      - name: Run tests 5x
        run: |
          for i in {1..5}; do
            echo "=== Run $i ==="
            npx vitest run --reporter=json >> test-results.json || true
          done

      - name: Report flaky tests
        run: |
          # Parse test-results.json and list tests that had mixed pass/fail
          node scripts/detect-flaky.js
```

```typescript
// scripts/detect-flaky.ts — parses results and reports flaky tests
// A test is "flaky" if it passed in some runs and failed in others
// Add flaky tests to a quarantine list and fix them before they hit CI
```

**Quarantine pattern:** Add `it.skip("FLAKY: ...")` to mark known-flaky tests so they don't block CI. Create a GitHub issue for each flaky test and fix them in the next sprint.

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: No Real Credentials in Test Files or CI Logs
```yaml
# ✅ Correct — GitHub Secrets never appear in logs
env:
  OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

# ❌ Never hardcode even test keys
env:
  OPENAI_API_KEY: "sk-test-abc123" # Visible in repo history forever
```

### Rule 2: Isolate Test Database Completely
The test database must never contain real user data. Use a dedicated test Supabase project (free tier) or a local PostgreSQL instance. Never point `DATABASE_URL` in CI at your production or development database.

### Rule 3: Never Test Security by Checking It Is Not Applied
```typescript
// ❌ This test only passes when auth is broken
it("returns data without auth token", async () => {
  const res = await GET(unauthenticatedRequest);
  expect(res.status).toBe(200); // THIS IS A SECURITY BUG, NOT A PASSING TEST
});

// ✅ Security tests verify that protection IS applied
it("returns 401 without auth token", async () => {
  const res = await GET(unauthenticatedRequest);
  expect(res.status).toBe(401); // Correct — protection is working
});
```

### Rule 4: Rotate Any Secret That Appears in a Test Failure Log
If a real secret accidentally appears in a test output or CI log, treat it as compromised. Rotate it immediately regardless of whether the log is private.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Writing Tests After Every Feature Without Running the Suite
You write 20 tests but never run `npm test`. Three of them have syntax errors. You only find out in CI 2 weeks later.  
**Fix:** Run `npm test -- --watch` during development. See tests go green as you write them.

### ❌ Testing the Mock Instead of the Code
```typescript
// ❌ This entire test proves nothing about your application
mockAI.generateText.mockResolvedValueOnce({ content: "test output" });
const result = await mockAI.generateText("prompt");
expect(result.content).toBe("test output"); // You tested the mock, not your code
```

### ❌ One Giant Test That Checks Everything
```typescript
// ❌ Hard to read, hard to debug when it fails
it("the entire reminder feature works", async () => {
  // Creates user, creates reminder, updates it, deletes it...
  // 80 lines of test code
});

// ✅ One thing per test
it("creates a reminder successfully", ...)
it("returns 400 when text is missing", ...)
it("soft deletes a reminder", ...)
```

### ❌ Brittle E2E Tests That Break on Every UI Change
```typescript
// ❌ Breaks when you change the button's CSS class
await page.click(".btn-primary.mt-4.font-semibold");

// ✅ Stable selector that only changes when the feature intentionally changes
await page.click('[data-testid="create-reminder-button"]');
```

### ❌ Skipping the Build Step in CI
Tests passing does not mean the app builds successfully. TypeScript errors that only appear at build time (unused exports, type mismatches across files) are caught by `npm run build` but not by tests alone. Always include the build step in CI.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Automated Test Suite

### Add Mutation Testing
Verify your tests actually catch bugs by introducing intentional bugs and checking if tests fail:
```
Tool: Stryker Mutator (JavaScript) or mutmut (Python)
Ask your AI agent: "Set up Stryker mutation testing for my /lib/ directory.
Run it and show me which mutations survived (tests that didn't catch the bug)."
```

### Add Property-Based Testing for Complex Logic
Instead of writing individual test cases, generate hundreds of random inputs automatically:
```typescript
// fast-check library — generates random inputs and finds edge cases you missed
import fc from "fast-check";
import { parseReminderDate } from "@/lib/utils/parse-date";

it("never throws on any string input", () => {
  fc.assert(fc.property(
    fc.string(),
    fc.date(),
    (input, baseDate) => {
      // Should never throw — only return null or a valid date
      expect(() => parseReminderDate(input, baseDate)).not.toThrow();
    }
  ));
});
```

### Add Test Sharding for Large Test Suites
When your test suite grows beyond 200 tests, split it across multiple CI runners to keep total time under 2 minutes:
```yaml
# GitHub Actions matrix strategy — runs tests in 4 parallel shards
strategy:
  matrix:
    shard: [1, 2, 3, 4]
steps:
  - run: npm test -- --shard=${{ matrix.shard }}/4
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — AI Output Tests That Caught 4 Production Bugs Pre-Launch

The reminder intent parser had a systematic weakness: it was only tested with "perfect" AI output during development. Before launch, a set of AI output fixture tests was written with 8 different malformed response patterns.

```
Fixtures that revealed bugs before launch:
1. Code fence wrapping → parser threw SyntaxError (fixed: strip fences before parse)
2. Past date scheduling → parser accepted "yesterday at 3pm" (fixed: validate date is future)
3. Missing scheduledAt for recurring reminders → parser returned partial object (fixed: Zod validation)
4. AI returning a question instead of JSON → parser returned garbage (fixed: detect non-JSON responses)

Zero of these 4 bugs required calling the real AI to discover.
All were caught by hardcoded fixture strings in < 1 second per test.
```

---

### Case Study 2: EdTech App — CI Pipeline That Paid for Itself in Week 1

The CI pipeline was set up with: type check + unit tests + integration tests + build check.

In the first week after setup:
```
Monday: Developer pushed a refactor. 
  Type check caught: 3 TypeScript errors in the lesson API types.
  Fixed before merge: 15 minutes.
  If reached production: lesson generation would have broken for all users.

Wednesday: AI agent generated a new feature.
  Integration test caught: the new route was missing an auth check.
  Fixed before merge: 5 minutes.
  If reached production: any user could access any other user's lessons.

Friday: Routine deploy.
  Build check caught: an import from a deleted file.
  Fixed before deploy: 10 minutes.
  If deployed: the app would not start.

Cost of CI setup: ~2 hours.
Bugs prevented in week 1: 3 (one of which was a security issue).
```

</real_world_examples>

</skill_document>
