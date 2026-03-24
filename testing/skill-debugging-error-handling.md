---
name: debugging-error-handling
description: Fixes bugs systematically using a structured diagnosis process and adds production error handling. Use when stuck on any bug, when setting up Sentry, when adding structured logging, or when errors are silently swallowed.
---
# Skill: Debugging & Error Handling

```json
{
  "skill_id": "debugging-error-handling",
  "category": "Testing & Reliability",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Debugging & Error Handling — Fixing Bugs Systematically Without Losing Hours</title>

<overview>

## What This Skill Enables
- Debug any error in your app using a repeatable, time-boxed process — not random trial-and-error
- Add error handling to your backend and frontend that makes bugs self-reporting (you find out before your users do)
- Build a logging and monitoring setup so production issues are visible the moment they happen

## Why It Matters for Vibe Coders
Debugging is where most vibe coders lose entire days. The AI agent changes one thing, breaks another, the vibe coder pastes the error back, the AI changes three more things, and now three new things are broken. This skill gives you a systematic process: isolate, understand, fix one thing, verify. It also gives your app the ability to tell you when something goes wrong, rather than waiting for users to report it.

## When to Use This Skill
- Any time you are stuck on a bug for more than 15 minutes
- When setting up error tracking for the first time (do this before launch)
- When adding a new feature to make sure errors are handled gracefully
- When a production bug appears and you need to understand what happened

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "error_tracking": "[REPLACE: e.g. Sentry / none yet / Axiom]",
    "logging_setup": "[REPLACE: e.g. console.log only / Pino / structured logging]",
    "current_bug_description": "[REPLACE: describe the bug you are debugging right now, or 'none — setting up error handling']",
    "error_message_if_any": "[REPLACE: paste the exact error message]",
    "file_where_error_occurs": "[REPLACE: file path if known]",
    "what_changed_before_bug_appeared": "[REPLACE: what was the last thing you changed?]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Debugging

### Mental Model 1: The Scientific Method
Every bug is a hypothesis to be tested:

```
1. OBSERVE   → What exactly is wrong? (exact error, exact steps to reproduce)
2. HYPOTHESIZE → What is the most likely cause?
3. TEST       → Change ONE thing to test that hypothesis
4. CONCLUDE   → Did it fix it? If yes: done. If no: new hypothesis.
```

The most common debugging failure is changing multiple things at once. You fix the bug but do not know which change fixed it — and you may have introduced a new bug with the other changes.

### Mental Model 2: The Error Is Pointing at Something
Most error messages are not vague — they are specific. Developers (and vibe coders) often skim the error and immediately start changing code. Read the error completely before touching anything.

```
Error: Cannot read properties of undefined (reading 'scheduledAt')
         │
         └─→ Something is undefined that you expected to have a value
             The property 'scheduledAt' is what you tried to access
             The CAUSE is one level up: what returned undefined?
```

### Mental Model 3: The Layer System for API Bugs
When an API feature breaks, it can break at any layer. Check them in order:

```
Layer 1: Network    → Is the request reaching the server? (Check Network tab)
Layer 2: Auth       → Is the user authenticated? (Check for 401)
Layer 3: Validation → Is the input valid? (Check for 400)
Layer 4: Business   → Is the logic correct? (Check server logs)
Layer 5: Database   → Is the query correct? (Check DB logs or query output)
Layer 6: Response   → Is the response parsed correctly? (Check client-side handling)
```

Start at Layer 1, not Layer 4. Most vibe-coder bugs are at Layers 1–3.

</mental_models>

---

<system_design_breakdown>

## Error Architecture for AI Products

### Where Errors Should Be Caught

```
USER ACTION
    │
    ▼
FRONTEND COMPONENT
    │ try/catch around API calls
    │ shows user-facing error message
    ▼
API ROUTE
    │ try/catch wraps all logic
    │ returns structured error response
    │ logs to server console
    ▼
BUSINESS LOGIC / SERVICE
    │ throws typed errors
    │ never swallows errors silently
    ▼
DATABASE / AI PROVIDER
    │ errors bubble up as thrown exceptions
    │ never returns undefined silently
    ▼
ERROR TRACKING (Sentry / Axiom)
    │ captures all unhandled exceptions
    │ alerts on error rate spikes
    ▼
YOU (developer)
    receives Slack/email alert within 60 seconds of production error
```

### Error Response Standard
Every error from your API must follow this shape:

```json
{
  "success": false,
  "error": "MACHINE_READABLE_CODE",
  "message": "Human-readable explanation of what went wrong",
  "details": {} // optional: validation errors, field-specific messages
}
```

This means your frontend always knows what to display, and your logs always have a searchable error code.

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Diagnose before changing. Fix one thing at a time. Verify the fix. -->

## The Universal Debugging Process (Use Every Time)

### Step 1 — Read the Full Error (2 minutes max)
Before touching any code:

```
1. Read the full error message — all of it, including the stack trace
2. Identify: what type of error is it?
   - TypeError (wrong type/undefined) → something is null/undefined
   - SyntaxError → code is not valid syntax
   - Network Error → request did not reach the server
   - 4xx HTTP → request reached server but was rejected (check why)
   - 5xx HTTP → server crashed (check server logs)
3. Identify: which FILE and LINE number is mentioned first in the stack trace?
   → That is where the bug lives (usually)
4. Ask: "What was the LAST thing I changed?" → 90% of bugs are in the last change
```

---

### Step 2 — Reproduce the Bug Reliably (5 minutes)
You cannot fix a bug you cannot reproduce consistently.

```
Write down (literally):
- What I did: [exact steps]
- What I expected: [expected behaviour]
- What happened: [actual behaviour]
- Does it happen every time? Y/N

If N: what makes it intermittent? (specific data? specific user? specific timing?)
Intermittent bugs are usually: race conditions, async timing, or data-dependent edge cases.
```

---

### Step 3 — Isolate the Problem (10 minutes)
**Prompt your AI agent:**
```
I have this error: [PASTE FULL ERROR MESSAGE AND STACK TRACE]
It occurs in this file: [FILE PATH]
Here is the relevant code: [PASTE THE FUNCTION OR ROUTE — keep it small]
The last change I made was: [DESCRIBE]

Do NOT fix anything yet.
Tell me:
1. What is causing this specific error
2. Which line is the root cause (not just the symptom)
3. What the minimal fix would be
4. What other code might break if we apply that fix
```

---

### Step 4 — Fix ONE Thing
After the AI identifies the cause:

**Prompt your AI agent:**
```
Root cause identified: [DESCRIBE WHAT AI TOLD YOU]
Fix ONLY this: [THE SPECIFIC ISSUE]
Do not refactor, do not change any other files, do not improve unrelated code.
Show me the before and after of the changed lines only.
```

---

### Step 5 — Verify the Fix
```
1. Reproduce the original bug steps → does the error still occur? (should be: NO)
2. Test adjacent functionality → did the fix break anything nearby?
3. If new errors appear: STOP. Do not chase them yet. 
   Revert and start the process again with the new full error.
```

---

## Adding Proactive Error Handling (Infrastructure Setup)

### Step 6 — Add Structured Error Logging

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add structured error logging to my app.
Requirements:
1. Every API route's catch block logs: timestamp, route path, userId, error message, stack trace
2. Log format: JSON (machine-readable for log aggregation tools)
3. Log levels: error (unexpected crashes), warn (handled errors worth monitoring), info (significant events)
4. In development: pretty-print to console
5. In production: output raw JSON (for Axiom/Datadog ingestion)

Use Pino (recommended) or console with JSON.stringify.
Apply to all routes in /app/api/ — show me the middleware pattern to avoid repeating this.
```

---

### Step 7 — Add Sentry Error Tracking (Production)

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add Sentry error tracking to my [FRAMEWORK] app.
Requirements:
1. Install: @sentry/nextjs (or @sentry/python for FastAPI)
2. Capture all unhandled exceptions automatically
3. Add user context to every error (userId, email) so I know which user was affected
4. Set up source maps so stack traces show my original code, not minified code
5. Create a /api/sentry-example-api route that intentionally throws an error to verify setup

Environment variables: SENTRY_DSN (from Sentry dashboard)
Do not add Sentry to tests — only development and production environments.
```

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start — Active Bug
```
I am debugging a bug in my app.
Project context: [PASTE context_anchor JSON]

Error: [PASTE EXACT ERROR MESSAGE]
File: [WHERE IT OCCURS]
Steps to reproduce: [DESCRIBE]
Last change made: [DESCRIBE]

Before suggesting any fix:
1. Identify the root cause (not just the symptom)
2. Tell me which single line is responsible
3. List any risks of fixing it
Then wait for my confirmation before suggesting the code change.
```

### Graceful Error Handling Addition Prompt
```
Add error handling to this [route / function / component]:
[PASTE CODE]

Requirements:
1. Wrap all logic in try/catch
2. On error: log { timestamp, context: "[ROUTE NAME]", error: error.message, stack: error.stack }
3. Return/render a user-facing message that is helpful but does not expose internals
4. For API routes: return structured { success: false, error: "ERROR_CODE", message: "..." }
5. For components: show an inline error message with a retry button
6. Do not change any business logic — only add error handling around it
```

### Error Boundary Addition Prompt
```
Add a React Error Boundary to my [PAGE / FEATURE SECTION].
Project context: [PASTE context_anchor JSON]

Requirements:
1. Catch rendering errors in [COMPONENT OR PAGE]
2. Display a fallback UI: "Something went wrong" + a "Try again" button that reloads
3. Log the error to Sentry (if configured) or console
4. Do not catch errors outside of this boundary — let them bubble to the global handler

File: /components/error-boundary.tsx (reusable) + usage in [TARGET FILE]
```

### Production Bug Investigation Prompt
```
A production bug was reported:
User impact: [DESCRIBE — e.g. "Users cannot create reminders"]
Error from Sentry / logs: [PASTE ERROR]
Affected users: [e.g. all users / specific users / since yesterday]
Project context: [PASTE context_anchor JSON]

Help me investigate:
1. What could cause this specific error?
2. Which files should I check first?
3. What information do I need from the logs to confirm the cause?
4. What is the safest minimal fix if the cause is [YOUR BEST GUESS]?

Do not write any code yet. Give me the investigation plan.
```

### Add Health Check Endpoint Prompt
```
Add a /api/health endpoint to my app that:
1. Checks: database connection is alive
2. Checks: AI provider API key is set (not necessarily valid — just present)
3. Returns: { status: "ok" | "degraded" | "down", checks: { db: "ok"|"error", ai: "ok"|"error" }, timestamp }
4. Returns 200 if all checks pass, 503 if any critical check fails
5. Does NOT require authentication — this is a public monitoring endpoint

This endpoint will be called by my hosting provider's health checks every 30 seconds.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "My AI agent keeps changing things and making more errors. What do I do?"

Stop. This is the most important thing: **stop giving the AI new instructions when you have unresolved errors.**

The AI agent does not have memory of what it changed last session. If you keep saying "fix this error," it will keep changing code, each change potentially introducing new issues. You end up with a codebase that is progressively more broken.

**The reset procedure:**
1. Use `git diff` to see every change from the current session
2. If things are worse than before: run `git stash` or `git checkout .` to revert
3. Start fresh with ONE very specific problem and the debugging process in this skill
4. Never let an AI agent make more than 3 file changes before you verify the app still works

---

### "What is Sentry and do I need it before launch?"

Sentry is a service that automatically captures and organises every error that happens in your production app. Without it, you only know about bugs when users tell you.

**What it gives you:**
- Instant Slack/email notification when a new error occurs
- The full stack trace, the user affected, the exact URL, and the request data
- Error frequency (is this happening to 1 user or 1,000 users?)
- Error trends (did a deploy cause a spike in errors?)

**Do you need it before launch?** Yes, for any app with real users. The free tier handles up to 5,000 errors per month — enough for most early-stage products.

**Setup time:** 20–30 minutes using the prompt in this skill. The cost of not having it is finding out about production bugs from angry users instead of an instant notification.

---

### "What is the difference between a 4xx and 5xx error?"

**4xx (Client Error):** The request was wrong in some way. The server understood it but could not fulfil it.
- 400: Your input was invalid
- 401: You are not logged in
- 403: You are logged in but not allowed
- 404: The thing you asked for does not exist
- 429: You are sending too many requests

**5xx (Server Error):** Your server crashed or failed. The request was fine — the server could not handle it.
- 500: Something in your code threw an unexpected error
- 502/503: Your server is down or overloaded
- 504: The request timed out

**Debugging rule:** 4xx errors are usually fixable in the frontend or the request. 5xx errors require you to check the server logs.

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## Error Handling
- Error class: AppError with factory errors — `lib/errors/index.ts`
- Error handler: handleError() — used in all route catch blocks
- Error boundary: React ErrorBoundary with Sentry — `components/error-boundary.tsx`
- Logger: structured JSON in prod, coloured in dev — `lib/logger.ts`
- Request correlation: requestId in all error responses (from X-Request-ID header)
- Async safety: all route handlers wrapped in try/catch
- AI failure checklist: token limit, content policy, rate limit, JSON parse, streaming timeout, model pinning
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Error Handling Is Working

### Error Handling Checklist
```
□ Every API route has a try/catch wrapping all logic
□ Every catch block logs the error (not swallowed silently)
□ Every catch block returns a structured error response (not a 200 with error inside)
□ Every component that fetches data handles the error state (not just loading + success)
□ The /api/health endpoint returns correctly when DB is down
□ Sentry (or equivalent) is configured and receiving test errors
□ You have verified Sentry works by deliberately triggering a test error
```

### Test Your Error Handling Intentionally
```bash
# Simulate a database outage — temporarily set wrong DB URL
DATABASE_URL="postgresql://wrong-host/wrong-db" npm run dev
# → Your app should show a graceful error, not a crash

# Simulate an AI provider failure — temporarily set wrong API key
OPENAI_API_KEY="invalid-key-test" npm run dev
# → AI-dependent features should show a user-friendly error

# Test your 404 handling — visit a non-existent route
curl http://localhost:3000/api/this-does-not-exist
# → Should return 404 JSON, not an HTML error page or a crash
```

### Common Error Handling Mistakes

| Issue | What You See | Fix |
|---|---|---|
| Swallowed error | Feature silently fails, no error shown | Add `console.error(error)` in every catch block |
| Error exposes internals | `"PrismaClientKnownRequestError: ..."` shown to user | Catch Prisma errors and return a generic "Database error" message |
| No user-facing message | API returns 500 but frontend shows blank | Add error state handling to the component using the API |
| Error in error handler | App crashes trying to report an error | Wrap Sentry calls in try/catch too |
| All errors look the same | Hard to search logs | Add a unique error code to every catch block |

### Debugging Loop Reference
```
Bug appears →
  1. READ full error message + stack trace
  2. IDENTIFY layer: network? auth? validation? logic? database?
  3. REPRODUCE reliably — exact steps
  4. ISOLATE — find the root cause line, not the symptom line
  5. FIX ONE THING — one change, one file
  6. VERIFY — reproduce original bug steps → should not occur
  7. REGRESSION — test adjacent functionality → should still work
  8. If stuck at step 4 for >10 minutes: add console.log to the suspect line,
     re-run, see what the actual value is vs what you expect
```

</testing_and_qa>

---

<common_patterns>

## Reusable Error Handling Patterns

### Pattern 1: Standard API Route Error Handler
```typescript
// /lib/api-error.ts
export class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number = 500,
    public details?: object
  ) {
    super(message);
    this.name = "AppError";
  }
}

// Reusable common errors
export const Errors = {
  UNAUTHORIZED: new AppError("UNAUTHORIZED", "Authentication required", 401),
  FORBIDDEN: new AppError("FORBIDDEN", "Access denied", 403),
  NOT_FOUND: (resource: string) => new AppError("NOT_FOUND", `${resource} not found`, 404),
  VALIDATION: (message: string, details?: object) =>
    new AppError("VALIDATION_ERROR", message, 400, details),
  INTERNAL: new AppError("INTERNAL_ERROR", "An unexpected error occurred", 500),
};

// In API routes:
export function handleError(error: unknown, context: string) {
  if (error instanceof AppError) {
    console.error(`[${context}] AppError:`, error.code, error.message);
    return NextResponse.json(
      { success: false, error: error.code, message: error.message, details: error.details },
      { status: error.statusCode }
    );
  }
  // Unknown error — log full details server-side, generic message to client
  console.error(`[${context}] Unexpected error:`, error);
  return NextResponse.json(
    { success: false, error: "INTERNAL_ERROR", message: "An unexpected error occurred" },
    { status: 500 }
  );
}
```

### Pattern 2: Frontend Error Boundary
```typescript
// /components/error-boundary.tsx
"use client";
import { Component, ReactNode } from "react";

interface Props { children: ReactNode; fallback?: ReactNode; }
interface State { hasError: boolean; error?: Error; }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: { componentStack: string }) {
    console.error("[ErrorBoundary]", error, info);
    // Sentry.captureException(error); // uncomment when Sentry is set up
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="flex flex-col items-center justify-center py-16 text-center">
          <p className="text-gray-900 font-medium">Something went wrong.</p>
          <p className="text-gray-500 text-sm mt-1">We have been notified and are looking into it.</p>
          <button
            className="mt-4 text-sm text-indigo-600 underline"
            onClick={() => this.setState({ hasError: false })}
          >
            Try again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### Pattern 3: Structured Server Logger
```typescript
// /lib/logger.ts
type LogLevel = "info" | "warn" | "error";

function log(level: LogLevel, context: string, message: string, data?: object) {
  const entry = {
    timestamp: new Date().toISOString(),
    level,
    context,
    message,
    ...data,
  };

  if (process.env.NODE_ENV === "development") {
    const colour = { info: "\x1b[36m", warn: "\x1b[33m", error: "\x1b[31m" }[level];
    console[level](`${colour}[${context}]\x1b[0m ${message}`, data ?? "");
  } else {
    console[level](JSON.stringify(entry)); // JSON for log aggregation in production
  }
}

export const logger = {
  info:  (context: string, message: string, data?: object) => log("info",  context, message, data),
  warn:  (context: string, message: string, data?: object) => log("warn",  context, message, data),
  error: (context: string, message: string, data?: object) => log("error", context, message, data),
};

// Usage:
// logger.error("[POST /api/reminders]", "Failed to create reminder", { userId, error: err.message });
```

### Pattern 4: Health Check Endpoint
```typescript
// /app/api/health/route.ts
import { db } from "@/lib/db";

export async function GET() {
  const checks: Record<string, "ok" | "error"> = {};

  // Database check
  try {
    await db.$queryRaw`SELECT 1`;
    checks.database = "ok";
  } catch {
    checks.database = "error";
  }

  // AI provider check (key presence only — not a live call)
  checks.ai_provider = process.env.OPENAI_API_KEY ? "ok" : "error";

  const allOk = Object.values(checks).every(v => v === "ok");

  return Response.json(
    { status: allOk ? "ok" : "degraded", checks, timestamp: new Date().toISOString() },
    { status: allOk ? 200 : 503 }
  );
}
```

### Pattern 5: AI-Specific Failure Debug Checklist

AI failures look different from regular API failures. When AI features aren't working, work through this checklist:

```
AI Feature Not Working — Debug in this order:

□ 1. TOKEN LIMIT EXCEEDED
   Symptom: Response cuts off mid-sentence, or error "context_length_exceeded"
   Check: Log prompt token count before sending. Use tiktoken to count:
     import { encoding_for_model } from "tiktoken"
     const enc = encoding_for_model("gpt-4o")
     const tokens = enc.encode(prompt).length
   Fix: Reduce system prompt, trim conversation history, chunk input

□ 2. CONTENT POLICY / MODERATION BLOCK
   Symptom: API returns 400 with "content_policy_violation" or empty response
   Check: Log the exact API error code and message
   Fix: Review prompt for inadvertent policy triggers, run through moderation API first

□ 3. RATE LIMIT (OpenAI/Anthropic)
   Symptom: 429 errors, intermittent failures under load
   Check: Check API dashboard for rate limit tier; look for 429s in Sentry
   Fix: Implement exponential backoff (already in skill-api-design-integration);
        request tier upgrade from provider

□ 4. INVALID JSON IN STRUCTURED OUTPUT
   Symptom: JSON.parse fails on model response, or Zod validation fails
   Check: Log raw response before parsing
   Fix: Use response_format: { type: "json_object" } + Zod validation wrapper;
        add "respond only with valid JSON" to system prompt

□ 5. STREAMING DISCONNECT
   Symptom: Stream starts but stops before completion, client shows partial response
   Check: Check Vercel function timeout (default 10s — AI routes need maxDuration: 60)
          Check if edge runtime is set on streaming routes
   Fix: Add export const runtime = "edge" to streaming routes;
        set maxDuration: 60 in vercel.json for AI paths

□ 6. WRONG MODEL BEHAVIOUR (seems like a different model)
   Symptom: Model stopped following instructions it used to follow
   Check: Are you using a pinned model version or a floating one ("gpt-4o" vs "gpt-4o-2024-08-06")?
   Fix: Pin to a dated snapshot in lib/ai/models.ts
```

### Pattern 6: Error Correlation with Request IDs — Help Users Help You Debug

When a user reports "I got an error", you need a way to find the exact server-side log entry. Return a `requestId` in every error response so users can report it to support.

```typescript
// Update your AppError and handleError to include requestId:
export function handleError(err: unknown, requestId?: string): Response {
  if (err instanceof AppError) {
    return Response.json({
      success: false,
      error: err.code,
      message: err.message,
      requestId,  // include so user can report it
    }, { status: err.statusCode })
  }

  const id = requestId ?? crypto.randomUUID()
  Sentry.withScope((scope) => {
    scope.setTag("requestId", id)
    Sentry.captureException(err)
  })

  return Response.json({
    success: false,
    error: "INTERNAL_ERROR",
    message: "Something went wrong. Reference ID: " + id,
    requestId: id,
  }, { status: 500 })
}

// In your frontend ErrorBoundary or error handler:
// Show: "Error reference: {requestId} — copy this when contacting support"
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Expose Internal Error Details to Users
```typescript
// ❌ Exposes your database schema, query structure, internal paths
return NextResponse.json({ error: error.message }, { status: 500 });
// e.g. error.message = "PrismaClientKnownRequestError at /var/task/node_modules/.prisma..."

// ✅ Log full details server-side, return generic message to client
console.error("[ROUTE]", error); // Full details for you
return NextResponse.json(
  { success: false, error: "INTERNAL_ERROR", message: "Something went wrong. Please try again." },
  { status: 500 }
);
```

### Rule 2: Never Log Sensitive Data
```typescript
// ❌ Logs user's full data object — may contain PII, hashed passwords
logger.error("User lookup failed", { user });

// ✅ Log only identifiers and safe fields
logger.error("User lookup failed", { userId: user.id, route: "/api/profile" });
```

### Rule 3: Sentry Must Be Configured to Scrub PII
When setting up Sentry, configure data scrubbing to remove passwords, tokens, and personal data from error reports before they are sent:
```typescript
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  beforeSend(event) {
    // Remove sensitive request headers
    if (event.request?.headers) {
      delete event.request.headers["authorization"];
      delete event.request.headers["cookie"];
    }
    return event;
  },
});
```

### Rule 4: Rate Limit Error Reporting Endpoints
If you have a client-side error reporting endpoint, rate limit it. Without this, a malicious actor can flood your error tracking with fake events, hiding real ones and potentially incurring costs.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Catching Errors and Returning 200
```typescript
// ❌ Frontend thinks the request succeeded — it did not
try {
  await db.reminder.create(data);
} catch (error) {
  return NextResponse.json({ success: false, error: "Failed" }, { status: 200 }); // 200 is WRONG
}

// ✅ Return the appropriate error status code
return NextResponse.json({ success: false, error: "INTERNAL_ERROR" }, { status: 500 });
```

### ❌ Empty Catch Blocks
```typescript
// ❌ Error disappears — you have no idea what went wrong
try {
  await doSomething();
} catch (error) {
  // nothing here
}

// ✅ Always log at minimum
} catch (error) {
  logger.error("[CONTEXT]", "doSomething failed", { error: String(error) });
  // Then decide: rethrow? return error response? fallback?
}
```

### ❌ Fixing Symptoms Instead of Root Causes
```
Bug: component crashes with "Cannot read properties of undefined"
Symptom fix: add `data?.field ?? ""` everywhere (defensive programming)
Root cause fix: find out WHY data is undefined — is an API call failing silently?
```
**Always ask: "Why is this value undefined?" before adding `?.` guards everywhere.**

### ❌ Making Multiple Changes While Debugging
You change 3 things to fix 1 bug. Two of those changes cause new bugs. Now you have 3 bugs instead of 1.  
**Fix:** One change per debug iteration. Verify before the next change.

### ❌ No Error State in Frontend Components
The API call fails. The component shows a blank screen (or crashes). The user has no idea what happened.
**Fix:** Every component that calls an API must handle three states: loading, success, and error. Always. No exceptions.

### ❌ Unhandled Promise Rejections in Route Handlers

In Next.js App Router, an async route handler that throws an unhandled error may return a 500 with no body — or in some edge cases, silently return a 200 with an empty body. Users see a broken experience with no error message.

```typescript
// ❌ This can silently fail:
export async function POST(req: Request) {
  const data = await req.json()
  const result = await db.document.create({ data })  // if this throws, what happens?
  return Response.json({ success: true, data: result })
}

// ✅ Always wrap async route handlers:
export async function POST(req: Request) {
  try {
    const data = await req.json()
    const result = await db.document.create({ data })
    return Response.json({ success: true, data: result })
  } catch (err) {
    return handleError(err)  // your AppError handler from this skill
  }
}
```

**Global unhandled rejection guard** — add to your app entry point:

```typescript
// app/api/_init.ts (imported once in root layout or middleware)
if (process.env.NODE_ENV === "production") {
  process.on("unhandledRejection", (reason, promise) => {
    console.error("[unhandledRejection]", { reason: String(reason) })
    // Sentry will catch this automatically if initialized, but belt-and-suspenders:
    Sentry.captureException(reason)
  })
}
```

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Error Handling

### Add Distributed Tracing
When requests span multiple services (your backend + AI provider + database), trace the full journey:
```
Tool: Sentry Performance Monitoring (included with Sentry SDK)
Ask your AI agent: "Add Sentry performance tracing to all API routes.
Capture: database query duration, AI provider call duration, total route duration."
```

### Add Alerting Rules in Sentry
```
Configure Sentry alerts for:
1. New error: any error type not seen before → Slack alert immediately
2. Error rate spike: >10 errors in 5 minutes → Slack alert immediately
3. P95 response time: any route >3 seconds → Slack alert daily digest
```

### Add Custom Error Pages in Next.js
```
/app/not-found.tsx      → custom 404 page (shown for missing routes)
/app/error.tsx          → custom error page (shown for unhandled exceptions)
/app/global-error.tsx   → custom error page (root layout errors)

Ask your AI agent: "Create custom error pages for 404 and 500 that match 
my app's design and provide a link back to the dashboard."
```

### Add Canary Deployments to Catch Bugs Early
Instead of deploying to all users at once, deploy to 5% of users first and monitor Sentry for new errors before rolling out fully. Vercel supports this natively with their "Edge Config" and deployment protection features.

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — The Silent Failure Bug
**Bug:** Reminders were not being scheduled. Users created them, saw confirmation, but the reminder never fired.

**How it was found:** A user complained 3 days after the bug was introduced. No error was logged because the bug was a swallowed catch block.

**Root cause:**
```typescript
// The original code — error completely hidden
try {
  await scheduleInngestJob(reminder);
} catch (error) {
  // TODO: handle this — never implemented
}
```

**Fix applied:**
```typescript
try {
  await scheduleInngestJob(reminder);
} catch (error) {
  logger.error("[createReminder]", "Failed to schedule Inngest job", {
    reminderId: reminder.id,
    error: String(error),
  });
  // Re-throw so the API route returns 500 instead of false success
  throw error;
}
```

**Prevention added:** Sentry + a `/api/health` check that verified Inngest connectivity on every deploy.

---

### Case Study 2: EdTech App — Systematic Debugging That Saved 3 Hours
**Bug:** Lesson generation was returning a 500 error for some users but not others.

**Debug process that worked:**
```
Step 1 — Read the Sentry error: 
  "JSON.parse: unexpected character at line 1 column 1 of the JSON data"
  In file: /lib/ai/parse-lesson-response.ts, line 23

Step 2 — Checked what line 23 does: JSON.parse(aiResponse.content)

Step 3 — Added a log: logger.warn("[parseLessonResponse]", "Raw AI output", { content: aiResponse.content })

Step 4 — Reproduced with a user's topic that was failing: "What is 0÷0?"

Step 5 — Log showed the AI returned:
  "I cannot provide a lesson on division by zero as it is mathematically undefined..."
  (plain text refusal — not the JSON structure the parser expected)

Step 6 — Root cause: the system prompt did not handle edge cases where the AI refuses
  Fix: added a check for refusals before attempting JSON.parse
  Added a user-facing message: "This topic cannot be taught. Please try a different topic."

Total time: 45 minutes (vs an estimated 3+ hours of random changes)
```

</real_world_examples>

</skill_document>
