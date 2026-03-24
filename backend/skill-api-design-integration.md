---
name: api-design-integration
description: Connects apps to third-party services including AI providers, Stripe, email, and SMS. Use when integrating any external API, building webhook handlers, or wrapping a third-party service for reliability.
---
# Skill: API Design & Third-Party Integration

```json
{
  "skill_id": "api-design-integration",
  "category": "Architecture & Backend",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>API Design & Third-Party Integration — Consuming External Services Reliably</title>

<overview>

## What This Skill Enables
- Connect your app to third-party services (AI providers, payment processors, email, SMS, calendar) without breaking when those services behave unexpectedly
- Design clean data contracts between your frontend and backend
- Handle webhooks — the mechanism most external services use to notify your app of events

## Why It Matters for Vibe Coders
Most AI products are not built in isolation — they connect to OpenAI, Stripe, Resend, Twilio, Google Calendar, and more. Each integration is a potential failure point. This skill gives you a consistent pattern for every external service call so that a flaky third-party API never silently breaks your app or your users' experience.

## When to Use This Skill
- Whenever you add a new third-party service to your app
- When an external API call is occasionally failing and you do not know why
- When you need to handle events from an external service (payment completed, email bounced, calendar event created)
- When designing your own API's request/response contracts for your frontend team (or your AI agent's frontend skill)

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "third_party_services_in_use": [
      "[REPLACE: e.g. OpenAI for lesson generation]",
      "[REPLACE: e.g. Resend for email notifications]",
      "[REPLACE: e.g. Stripe for subscriptions]"
    ],
    "service_to_integrate_today": "[REPLACE: ONE service only, e.g. Resend email]",
    "what_this_service_does_in_my_app": "[REPLACE: plain English, e.g. sends reminder notification emails to users]",
    "existing_integration_files": "[REPLACE: e.g. /lib/openai.ts, /lib/stripe.ts]",
    "webhook_endpoints_needed": "[REPLACE: e.g. /api/webhooks/stripe for payment events, or none]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Third-Party Integrations

### Mental Model 1: Every External Call Is a Risk
When your app calls an external API, you are depending on another company's infrastructure. It can be:
- Slow (network latency, their server load)
- Down (outages happen — even OpenAI and Stripe have them)
- Changed (they deprecate an API version)
- Expensive (you get charged per call)

**Design principle:** Every external call must have a timeout, error handling, and a fallback behaviour. Never assume it will succeed.

### Mental Model 2: Push vs Pull
There are two ways external services communicate with your app:

| Mode | How It Works | Example |
|---|---|---|
| **You call them (Pull)** | Your code makes an HTTP request to their API | Calling OpenAI to generate text |
| **They call you (Push/Webhook)** | They send an HTTP request to your endpoint when something happens | Stripe notifying you when a payment succeeds |

Most integrations use both. You call their API to initiate actions; they call your webhook to notify you of results or events you did not trigger.

### Mental Model 3: The Integration Wrapper Pattern
Never scatter third-party API calls throughout your codebase. Wrap every external service in a single file:

```
/lib/
  openai.ts      ← all OpenAI calls go here
  stripe.ts      ← all Stripe calls go here
  resend.ts      ← all email calls go here
```

If the external service changes their API or you switch providers, you change one file — not 15 scattered call sites.

</mental_models>

---

<system_design_breakdown>

## Integration Architecture

```
YOUR API ROUTE
      │
      ▼
┌─────────────────────────────────────────┐
│         INTEGRATION WRAPPER             │
│  /lib/[service].ts                      │
│  - API key from env vars only           │
│  - Timeout set on every call            │
│  - Retry logic for transient failures   │
│  - Error normalised to your format      │
│  - Usage logged to ai_generations table │
└──────────────────┬──────────────────────┘
                   │
                   ▼
         THIRD-PARTY API
         (OpenAI / Stripe / etc.)
                   │
          (async event)
                   │
                   ▼
┌─────────────────────────────────────────┐
│         WEBHOOK ENDPOINT                │
│  /api/webhooks/[service]/route.ts       │
│  - Verify signature FIRST               │
│  - Idempotency check                    │
│  - Process event                        │
│  - Return 200 fast                      │
└─────────────────────────────────────────┘
```

### Common Integrations for AI Products

| Service | Purpose | Integration Complexity |
|---|---|---|
| OpenAI / Anthropic / Google AI | Core AI features | Medium — streaming + error handling |
| Resend / SendGrid | Transactional email | Low |
| Stripe | Subscriptions + payments | High — webhooks critical |
| Twilio / Vonage | SMS notifications | Low |
| Google Calendar API | Calendar integration | Medium — OAuth per user |
| Upstash | Redis caching + rate limiting + queues | Low |
| Inngest / Trigger.dev | Background job scheduling | Medium |

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: One integration at a time. Test the wrapper before using it in routes. -->

## Step 1 — Create the Integration Wrapper File
Before writing any route that calls the external service.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
I am integrating [SERVICE NAME] into my app.
Its purpose: [WHAT IT DOES IN YOUR APP]

Create /lib/[service-name].ts with:
1. Client initialisation using environment variable: [ENV_VAR_NAME]
2. One function for each action I need: [LIST ACTIONS e.g. sendEmail, generateText]
3. Each function: has a 10-second timeout, catches all errors, returns { success, data, error }
4. A comment above each function explaining when to call it
5. Export a named client instance for reuse

Do not call this from any route yet. Just the wrapper file.
```

**Verify:** Import the wrapper in a test script. Call one function with test data. Confirm it returns the expected `{ success, data }` shape.

---

## Step 2 — Add the Integration to a Route
Only after the wrapper is tested.

**Prompt your AI agent:**
```
The [SERVICE] wrapper at /lib/[service].ts is working.
Now use it in this route: [ROUTE FILE PATH]

Add a call to [FUNCTION NAME] at [DESCRIBE WHERE IN THE ROUTE FLOW].
If the integration call fails: [DESCRIBE FALLBACK — e.g. log error and continue, or return 503]
Do not change any other logic in the route.
```

**Verify:** Call the route end-to-end. Confirm the integration fires. Check service dashboard for confirmation (e.g. email delivered, AI call logged).

---

## Step 3 — Set Up Webhook Endpoint (If Service Uses Push Events)
For Stripe, Clerk, auth providers, and any service that sends events to your app.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Set up a webhook endpoint for [SERVICE NAME].

Create /api/webhooks/[service]/route.ts that:
1. Verifies the webhook signature using [SERVICE]'s signing secret from env var [ENV_VAR]
2. Returns 400 immediately if signature is invalid
3. Parses the event type
4. Handles these specific events: [LIST EVENTS e.g. payment.succeeded, user.created]
5. Returns 200 immediately after receiving (do heavy processing async)
6. Is idempotent — processing the same event twice should not cause duplicate actions

Include a comment with the URL to register in [SERVICE]'s dashboard: [YOUR_DOMAIN]/api/webhooks/[service]
```

**Verify:** Use the service's webhook test tool (Stripe CLI, Clerk dashboard, etc.) to send a test event. Confirm your endpoint receives it and returns 200.

---

## Step 4 — Test Failure Scenarios
**Prompt your AI agent:**
```
The [SERVICE] integration is working for the happy path.
Now make the wrapper resilient to failures.

Add to /lib/[service].ts:
1. Retry logic: retry up to 2 times on network errors or 429/503 responses, with 1-second delay
2. Circuit breaker: if 3 consecutive calls fail, throw a clear error immediately without retrying
3. Timeout: 10 seconds max per call, then throw TimeoutError
4. Log every call: { service, function, success, latency_ms, error_if_any }

Do not change any route files.
```

**Verify:** Temporarily point the wrapper at a fake URL. Confirm retry fires and the circuit breaker eventually stops retrying.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on a third-party integration for my app.
Project context: [PASTE context_anchor JSON]
Service: [SERVICE NAME]
Files already created: [LIST]
Today I am building: [ONE SPECIFIC TASK]
Before writing code, confirm the env var names I need to set up for this integration.
```

### OpenAI Integration Wrapper Prompt
```
Create a production-ready OpenAI integration wrapper for my [FRAMEWORK] app.
Project context: [PASTE context_anchor JSON]

File: /lib/openai.ts
Functions needed:
1. generateText(prompt, systemPrompt, options?) → { content: string }
2. generateStructuredOutput(prompt, schema, systemPrompt?) → { data: T }
3. generateEmbedding(text) → { embedding: number[] }
4. streamText(prompt, systemPrompt) → ReadableStream

Each function:
- Uses gpt-4o for text, text-embedding-3-small for embeddings
- Reads API key from process.env.OPENAI_API_KEY
- Has 30-second timeout for generation, 10 seconds for embeddings
- Returns { success, data, error, usage: { promptTokens, completionTokens } }
- Logs to console with format: [OpenAI][functionName] latency: Xms tokens: Y
```

### Stripe Integration Prompt
```
Set up Stripe subscriptions for my [APP TYPE] app.
Project context: [PASTE context_anchor JSON]
Plans: [LIST PLANS — e.g. Free, Pro ($9/mo), Team ($29/mo)]

Generate:
1. /lib/stripe.ts — wrapper with: createCustomer, createCheckoutSession, createPortalSession, getSubscription
2. /api/webhooks/stripe/route.ts — handles: checkout.session.completed, customer.subscription.updated, customer.subscription.deleted
3. Database updates triggered by each webhook event
4. A checkSubscription(userId) utility that returns the user's current plan

Environment variables needed: STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, STRIPE_[PLAN]_PRICE_ID
```

### Email Integration Prompt (Resend)
```
Set up transactional email using Resend for my app.
Project context: [PASTE context_anchor JSON]

Generate:
1. /lib/email.ts — wrapper with: sendReminderEmail, sendWelcomeEmail, sendWeeklySummaryEmail
2. Each function takes { to, subject, [template-specific fields] }
3. Uses React Email templates or simple HTML (your preference)
4. Returns { success, messageId, error }
5. Reads API key from RESEND_API_KEY
6. From address: [YOUR FROM ADDRESS e.g. notifications@yourapp.com]

Do not set up React Email templates yet — use simple HTML strings for now.
```

### Webhook Debug Prompt
```
My webhook endpoint for [SERVICE] is not receiving events.
Project context: [PASTE context_anchor JSON]
Webhook URL registered: [YOUR URL]
Expected events: [LIST EVENTS]
What I see: [DESCRIBE WHAT IS OR IS NOT HAPPENING]

Check for:
1. Is the URL reachable? (local dev needs tunnelling via ngrok or Cloudflare tunnel)
2. Is the signature verification failing silently?
3. Is the endpoint returning a non-200 status?
4. Is there a timeout before the endpoint responds?

Give me a step-by-step debug checklist.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is a webhook and why is it different from a normal API call?"

**Normal API call (you initiate):** You ask a question, you wait for the answer.  
→ "Hey Stripe, has this payment succeeded?" → Stripe answers immediately.

**Webhook (they initiate):** You give them your address; they knock on your door when something happens.  
→ Stripe knocks on `/api/webhooks/stripe` and says "That payment just succeeded."

Webhooks are essential because you cannot keep asking "did anything happen?" every second — that is slow, expensive, and unreliable. Instead, the service tells you the moment something happens.

**The catch with webhooks in development:** They need a publicly accessible URL, but your laptop is not public. Use **ngrok** or **Cloudflare Tunnel** to create a temporary public URL pointing to your local server.

---

### "Why do I need to verify webhook signatures?"

Anyone can send a POST request to your webhook endpoint pretending to be Stripe or Clerk. Without signature verification, an attacker can fake a "payment succeeded" event and get premium access for free.

Your auth provider gives you a signing secret. Every real webhook request includes a signature generated using that secret. You verify the signature before trusting the event. This takes 5 lines of code and prevents a serious vulnerability.

---

### "Should I call the AI API directly in my route or use a wrapper?"

Always use a wrapper (`/lib/openai.ts`). Here is why this matters practically:

- When OpenAI releases a new, cheaper model, you change one line in the wrapper — not 15 routes
- When you want to add logging or cost tracking to all AI calls, you add it once in the wrapper
- When the API changes (it does), you fix it in one place
- Your AI coding agent will generate more consistent code because every prompt can reference the same wrapper pattern

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## API Integration Patterns
- External API wrappers: `lib/integrations/` — one file per service
- Circuit breakers: `lib/api/circuit-breaker.ts` — openAI, stripe, etc.
- Retry strategy: exponential backoff with jitter — `lib/api/retry.ts`
- Pagination: cursor-based on all list endpoints
- Webhook verification: HMAC-SHA256 signature check on all inbound webhooks
- Idempotency: idempotency key header on all mutating external API calls
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing Integrations

### Integration Test Template
```typescript
// /tests/integrations/openai.test.ts
import { generateText, generateEmbedding } from "@/lib/openai";

describe("OpenAI integration", () => {
  it("generates text successfully", async () => {
    const result = await generateText(
      "Say exactly: test successful",
      "You are a test assistant."
    );
    expect(result.success).toBe(true);
    expect(result.data?.content).toContain("test successful");
    expect(result.usage?.completionTokens).toBeGreaterThan(0);
  });

  it("handles network error gracefully", async () => {
    // Temporarily override API key to force failure
    process.env.OPENAI_API_KEY = "invalid-key";
    const result = await generateText("test", "test");
    expect(result.success).toBe(false);
    expect(result.error).toBeDefined();
    // Restore
    process.env.OPENAI_API_KEY = "real-key";
  });
});
```

### Webhook Testing Without Production Events
```bash
# Stripe CLI — send test events to your local endpoint
stripe listen --forward-to localhost:3000/api/webhooks/stripe
stripe trigger checkout.session.completed

# Clerk — use dashboard webhook testing tab
# Inngest — use dev server: npx inngest-cli@latest dev
```

### Debugging Loop for Integration Failures
```
Integration call failing →
  1. Is it a network error? (ECONNREFUSED, timeout) → check if service is reachable
  2. Is it a 4xx error? → check your request format and API key
  3. Is it a 5xx error? → the external service has a problem; check their status page
  4. Is it a parsing error? → the response shape changed; log raw response
  5. Paste the raw error + wrapper code to AI agent:
     "This integration call fails with: [ERROR]
      Raw response if available: [RESPONSE]
      Here is my wrapper: [PASTE CODE]
      Fix only the error handling. Do not change the function signature."
```

### Common Integration Errors

| Error | Service | Meaning | Fix |
|---|---|---|---|
| `401 Unauthorized` | Any | API key wrong or missing | Check env var name and value |
| `429 Too Many Requests` | OpenAI, Stripe | Rate limit hit | Add retry with backoff; implement rate limiting |
| `400 Bad Request` | Any | Request format wrong | Log the full request body; check API docs |
| `Webhook signature verification failed` | Stripe, Clerk | Wrong secret or raw body parsed early | Ensure raw body is passed to verify, not parsed JSON |
| `ECONNREFUSED` | Any | Cannot reach the service | Check URL; check if service is down; check timeout |
| `Context length exceeded` | OpenAI | Prompt too long | Truncate input or use a model with larger context |

</testing_and_qa>

---

<common_patterns>

## Reusable Integration Patterns

### Pattern 1: Universal Integration Wrapper Shape
```typescript
// Every integration wrapper returns this consistent type
type IntegrationResult<T> = {
  success: boolean;
  data?: T;
  error?: string;
  usage?: Record<string, number>; // tokens, cost, etc.
};

// Wrap every external call in this pattern
async function callExternalService<T>(
  fn: () => Promise<T>,
  serviceName: string
): Promise<IntegrationResult<T>> {
  const start = Date.now();
  try {
    const data = await Promise.race([
      fn(),
      new Promise<never>((_, reject) =>
        setTimeout(() => reject(new Error("Timeout")), 10000)
      ),
    ]);
    console.log(`[${serviceName}] success in ${Date.now() - start}ms`);
    return { success: true, data };
  } catch (error) {
    console.error(`[${serviceName}] failed in ${Date.now() - start}ms:`, error);
    return { success: false, error: String(error) };
  }
}
```

### Pattern 2: Idempotent Webhook Handler
```typescript
// /api/webhooks/stripe/route.ts
export async function POST(req: NextRequest) {
  // 1. Get raw body BEFORE parsing (required for signature verification)
  const rawBody = await req.text();
  const signature = req.headers.get("stripe-signature")!;

  // 2. Verify signature
  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(rawBody, signature, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch {
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }

  // 3. Idempotency — check if already processed
  const existing = await db.webhookEvent.findUnique({ where: { stripeEventId: event.id } });
  if (existing) {
    return NextResponse.json({ received: true }); // Already processed, return 200
  }

  // 4. Record event immediately (before processing)
  await db.webhookEvent.create({ data: { stripeEventId: event.id, type: event.type } });

  // 5. Process event (async — do not let this delay the 200 response)
  processStripeEvent(event).catch(console.error); // Fire and forget

  // 6. Return 200 immediately
  return NextResponse.json({ received: true });
}
```

### Pattern 3: Retry with Exponential Backoff
```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 2,
  baseDelayMs = 1000
): Promise<T> {
  let lastError: Error;
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      if (attempt < maxRetries) {
        const delay = baseDelayMs * Math.pow(2, attempt); // 1s, 2s, 4s
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  throw lastError!;
}

// Usage:
const result = await withRetry(() => openai.chat.completions.create({...}));
```

### Circuit Breaker — Stop Hammering a Failing Dependency

Retrying endlessly against a failing service (e.g., OpenAI during an outage) blocks threads and cascades failures across your app. A circuit breaker detects the failure and stops trying until the service recovers.

```typescript
// lib/api/circuit-breaker.ts
type CircuitState = "closed" | "open" | "half-open"

interface CircuitBreakerConfig {
  failureThreshold: number  // how many failures before opening
  resetTimeoutMs: number    // how long to wait before trying again
  name: string
}

class CircuitBreaker {
  private state: CircuitState = "closed"
  private failureCount = 0
  private lastFailureTime = 0

  constructor(private config: CircuitBreakerConfig) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === "open") {
      const timeSinceFailure = Date.now() - this.lastFailureTime
      if (timeSinceFailure < this.config.resetTimeoutMs) {
        throw new Error(`Circuit breaker OPEN for ${this.config.name}. Service unavailable.`)
      }
      this.state = "half-open"
    }

    try {
      const result = await fn()
      this.onSuccess()
      return result
    } catch (err) {
      this.onFailure()
      throw err
    }
  }

  private onSuccess() {
    this.failureCount = 0
    this.state = "closed"
  }

  private onFailure() {
    this.failureCount++
    this.lastFailureTime = Date.now()
    if (this.failureCount >= this.config.failureThreshold) {
      this.state = "open"
      console.error(`[circuit-breaker:${this.config.name}] OPENED after ${this.failureCount} failures`)
    }
  }
}

// Usage — one breaker per external dependency
export const openAIBreaker = new CircuitBreaker({
  name: "openai",
  failureThreshold: 5,
  resetTimeoutMs: 60_000,  // try again after 60 seconds
})

export const stripeBreaker = new CircuitBreaker({
  name: "stripe",
  failureThreshold: 3,
  resetTimeoutMs: 30_000,
})

// Wrap external calls:
// const result = await openAIBreaker.execute(() => openai.chat.completions.create(...))
```

### Safe Pagination Pattern — Cursor-Based Pagination

Offset pagination (`LIMIT 10 OFFSET 100`) is slow on large tables and returns inconsistent results when rows are inserted between pages. Use cursor-based pagination instead.

```typescript
// app/api/documents/route.ts
export async function GET(req: Request) {
  const { userId } = await requireAuth()
  const { searchParams } = new URL(req.url)
  const cursor = searchParams.get("cursor")  // ID of last item from previous page
  const limit = Math.min(parseInt(searchParams.get("limit") ?? "20"), 100)  // max 100

  const documents = await db.document.findMany({
    where: { userId, deletedAt: null },
    take: limit + 1,  // fetch one extra to determine if there's a next page
    cursor: cursor ? { id: cursor } : undefined,
    skip: cursor ? 1 : 0,  // skip the cursor item itself
    orderBy: { createdAt: "desc" },
    select: { id: true, title: true, createdAt: true },
  })

  const hasNextPage = documents.length > limit
  const items = hasNextPage ? documents.slice(0, -1) : documents

  return Response.json({
    success: true,
    data: {
      items,
      nextCursor: hasNextPage ? items[items.length - 1].id : null,
      hasNextPage,
    }
  })
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: All API Keys in Environment Variables
```typescript
// ❌ Never
const stripe = new Stripe("sk_live_actual_key_here");

// ✅ Always
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
if (!process.env.STRIPE_SECRET_KEY) throw new Error("STRIPE_SECRET_KEY not set");
```

### Rule 2: Never Log API Keys, Webhook Bodies Containing PII, or Full Request/Response
```typescript
// ❌ Logs sensitive data
console.log("Stripe event:", event); // May contain card details, email, etc.

// ✅ Log only safe identifiers
console.log("Stripe event:", { type: event.type, id: event.id });
```

### Rule 3: Always Verify Webhook Signatures (No Exceptions)
Every webhook endpoint must verify the signature before processing the payload. See Pattern 2 above.

### Rule 4: Use Live vs Test Keys Correctly
```
Development: STRIPE_SECRET_KEY=sk_test_...
Production:  STRIPE_SECRET_KEY=sk_live_...

Never use live keys in development. Never commit either to git.
Add to .gitignore: .env, .env.local, .env.production
```

### Rule 5: Scope API Keys to Minimum Permissions
When creating API keys in third-party dashboards:
- Read-only keys for operations that only read data
- Restricted permissions — do not use a full-access key for an endpoint that only sends email
- Rotate keys immediately if they appear in logs, errors, or any output visible outside your server

### Rule 6: Rate Limit Your Own Endpoints That Call Paid APIs
Without rate limiting, one user can trigger thousands of OpenAI calls, running up a large bill:
```
Ask your AI agent: "Add rate limiting to /api/ai/generate using Upstash Redis.
Max 20 requests per user per hour. Return 429 with retry-after header if exceeded."
```

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Calling External APIs Directly in Routes Without a Wrapper
You have `fetch("https://api.openai.com/...")` in 8 different route files. When OpenAI changes a parameter name, you fix it in 8 places and miss 2.  
**Fix:** All calls to a given service go through one wrapper file. Always.

### ❌ Parsing the Webhook Body Before Signature Verification
Stripe and other services require the raw, unparsed request body to verify the signature. If you call `await req.json()` before verifying, the signature check will always fail.  
**Fix:** Use `await req.text()` to get the raw body first, verify, then parse.

### ❌ Not Handling Webhook Duplicate Delivery
External services guarantee at-least-once delivery — meaning the same event can arrive twice. Processing a "payment succeeded" webhook twice could grant premium access twice or send two welcome emails.  
**Fix:** Always implement idempotency. Store the event ID and check for it before processing.

### ❌ Blocking the Webhook Response with Heavy Processing
Your webhook handler does a lot of work (generate AI content, send emails, update multiple tables) before returning 200. The external service waits, times out, and retries — creating duplicate processing.  
**Fix:** Return 200 immediately. Move all processing to a background job triggered by the webhook.

### ❌ Using the Same API Key for Development and Production
A bug in your dev environment makes 10,000 test API calls on your production OpenAI key. Your bill is enormous.  
**Fix:** Separate API keys per environment. Many services (OpenAI, Stripe) have explicit test mode keys.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Integrations

### Add an Integration Health Dashboard
```sql
-- Query to see integration health over the last 24 hours
SELECT 
  feature as integration,
  COUNT(*) as total_calls,
  SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) as successes,
  AVG(latency_ms) as avg_latency_ms,
  SUM(total_tokens) as total_tokens_used
FROM ai_generations
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY feature
ORDER BY total_calls DESC;
```

### Add Cost Tracking Per Feature
```typescript
// After each AI call, calculate and store cost
const COST_PER_TOKEN = {
  "gpt-4o": { input: 0.0000025, output: 0.00001 },
  "text-embedding-3-small": { input: 0.00000002, output: 0 },
};

const cost =
  (usage.promptTokens * COST_PER_TOKEN[model].input) +
  (usage.completionTokens * COST_PER_TOKEN[model].output);

await db.aiGeneration.create({ data: { ...logData, estimatedCostUsd: cost } });
```

### Implement a Provider Fallback
When your primary AI provider is down, automatically fail over to a backup:
```typescript
export async function generateTextWithFallback(prompt: string) {
  const primary = await callOpenAI(prompt);
  if (primary.success) return primary;

  console.warn("OpenAI failed, falling back to Anthropic");
  return await callAnthropic(prompt);
}
```

### Add a Unified Notification Service
Instead of calling Resend directly from multiple places, create a notification abstraction:
```typescript
// /lib/notifications.ts
export async function notify(userId: string, type: NotificationType, data: object) {
  const user = await getUserPreferences(userId);
  if (user.emailEnabled) await sendEmail(user.email, type, data);
  if (user.smsEnabled) await sendSMS(user.phone, type, data);
  if (user.pushEnabled) await sendPush(user.pushToken, type, data);
}
// Routes call notify() — not individual providers
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Multi-Channel Notification System
**Services integrated:** Resend (email) + Twilio (SMS) + Expo (push notifications)

```
Integration order:
1. Built /lib/resend.ts wrapper → tested with one email send
2. Added to reminder scheduler: on reminder trigger → call sendReminderEmail
3. Built /lib/twilio.ts wrapper → tested with one SMS send
4. Extended scheduler: if user has phone_number → also call sendReminderSMS
5. Built /lib/notifications.ts abstraction → all scheduler code now calls notify()
6. Switching providers later: only change the wrapper, not the scheduler

Key lesson: building the abstraction layer (notify()) before having 3 providers
would have been premature. Built it after the second provider made the pattern clear.
```

---

### Case Study 2: EdTech App — Stripe Subscription with Lesson Gating
**Integration:** Stripe for subscriptions, lesson generation gated behind Pro plan

```
Webhook flow:
1. User clicks "Upgrade to Pro" → route calls createCheckoutSession → redirect to Stripe
2. User pays on Stripe → Stripe sends checkout.session.completed to /api/webhooks/stripe
3. Webhook handler verifies signature → checks idempotency → calls processSubscription()
4. processSubscription() → updates user.plan to "pro" in DB → sends welcome email via Resend
5. User returns to app → lesson generation route checks user.plan === "pro" before allowing AI call

Common failure point: webhook not received in development.
Fix: used Stripe CLI locally: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`
```

</real_world_examples>

</skill_document>
