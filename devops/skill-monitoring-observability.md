---
name: monitoring-observability
description: Sets up Sentry, uptime monitoring, AI cost tracking, and daily cost alerts. Use before any public launch, when AI bills are unexpectedly high, or when production bugs are discovered by users instead of by you.
---
# Skill: Monitoring & Observability

```json
{
  "skill_id": "monitoring-observability",
  "category": "Deployment & DevOps",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Monitoring & Observability — Knowing When Your AI Product Breaks Before Users Do</title>

<overview>

## What This Skill Enables
- Know instantly when something breaks in production — before users report it
- Track AI API costs per feature so you are never surprised by a large bill
- Measure what matters: response times, error rates, and the health of your AI calls
- Build dashboards that show real-time system health without complex infrastructure

## Why It Matters for Vibe Coders
An unmonitored AI product is a liability. AI API calls can silently fail, costs can spike overnight, and performance can degrade gradually until it becomes user-impacting. Most vibe coders find out about production problems from user complaints. This skill means you find out first — via an alert, a dashboard, or a cost anomaly — and fix it before it becomes a support ticket.

## When to Use This Skill
- Before your first public launch — monitoring should be in place before users arrive
- When you notice users are churning but cannot identify why
- When your AI API bill is higher than expected and you do not know which feature is responsible
- When a production bug occurred and you had no data to diagnose it

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "hosting_platform": "[REPLACE: e.g. Vercel / Railway]",
    "ai_provider": "[REPLACE: e.g. OpenAI / Anthropic]",
    "current_monitoring": "[REPLACE: e.g. none / Sentry errors only / Vercel analytics only]",
    "most_expensive_ai_features": "[REPLACE: e.g. lesson generation, chat completions]",
    "monitoring_task_today": "[REPLACE: e.g. set up error tracking / add cost monitoring / build health dashboard]",
    "monthly_ai_budget_usd": "[REPLACE: e.g. $50 / $200 / unknown]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Monitoring

### Mental Model 1: The Three Pillars
Good observability has three components:

```
LOGS        → What happened? (event records)
             "User X created reminder Y at 14:32:01"
             "OpenAI call failed: rate limit exceeded"

METRICS     → How is it performing? (numbers over time)
             API response time: p50=180ms, p95=420ms, p99=1200ms
             Error rate: 0.3% of requests
             AI cost: $0.82 today

TRACES      → Why did it happen? (request flow through system)
             Request → Auth (12ms) → DB query (45ms) → AI call (1.2s) → Response
```

For AI products, **logs and metrics** are the most important. Distributed tracing is nice-to-have.

### Mental Model 2: Alert on Symptoms, Not Causes
The right things to alert on are the things users directly experience:

```
✅ ALERT ON (user-facing symptoms):
- Error rate > 2% (users are getting errors)
- p95 response time > 3s (users are waiting too long)
- /api/health returning non-200 (core services are down)
- AI cost spike > 2x daily average (potential abuse or infinite loop)

❌ DO NOT ALERT ON (internal metrics unless they cause the above):
- CPU usage > 80% (unless causing slowdowns)
- Specific function call counts
- Cache hit rates
```

### Mental Model 3: The Observability Stack for AI Products
You do not need one complex tool — you need three simple, focused ones:

| Tool | Purpose | Cost |
|---|---|---|
| **Sentry** | Error tracking + alerts | Free up to 5k errors/month |
| **Vercel Analytics** | Web vitals + page performance | Free on hobby plan |
| **Custom AI cost table** | Track AI spend per feature | Free (your own DB table) |

Add these in order. Each one gives you increasing visibility. All three together give you full coverage for most AI products.

</mental_models>

---

<system_design_breakdown>

## Monitoring Architecture for AI Products

```
USER REQUEST
     │
     ▼
API ROUTE
     │ ──────────────────→ SENTRY (errors, performance)
     │
     ├── AI call ─────────→ AI_GENERATIONS table (cost, latency, model)
     │
     ├── DB query ─────────→ Vercel/Railway logs (query time)
     │
     └── Response ─────────→ Vercel Analytics (response time, status code)

BACKGROUND (async, continuous):
  Uptime monitor (Better Uptime / UptimeRobot) → checks /api/health every 60s
  Cost alert (OpenAI dashboard) → email when spend > $X/day
  Sentry weekly digest → summary of top errors
```

## The AI Cost Tracking Table

Every AI call should log to this table in your database:

```sql
-- Already covered in skill-database-storage.md
-- Referenced here because it is critical for monitoring

ai_generations:
  id, user_id, feature, model,
  prompt_tokens, completion_tokens, total_tokens,
  estimated_cost_usd, latency_ms,
  status (success/failed/timeout),
  created_at
```

This table gives you:
- Cost breakdown by feature (which feature is most expensive?)
- Cost breakdown by user (is one user abusing your system?)
- Latency trends by model (is GPT-4o getting slower?)
- Failure rates by feature (which AI feature is least reliable?)

## SLO Definitions — What Does "Working Correctly" Mean?

Before you can monitor, you need to define what success looks like in measurable terms. A Service Level Objective (SLO) is a target you commit to.

**Recommended SLOs for AI products:**

| Service | SLO | Alert Threshold |
|---|---|---|
| Web app availability | 99.5% uptime (< 3.6 hrs/month downtime) | Alert if < 99% in rolling 24h |
| AI generation (p95 latency) | p95 < 10 seconds | Alert if p95 > 15 seconds over 5 min |
| AI generation (error rate) | < 2% errors | Alert if > 5% errors in 5 min window |
| API response (non-AI, p95) | p95 < 500ms | Alert if p95 > 1 second |
| Background job success rate | > 95% | Alert if < 90% in rolling 1 hour |

```sql
-- Query p95 AI latency from your ai_generations table (run weekly):
SELECT
  DATE_TRUNC('day', created_at) as day,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY latency_ms) as p50_ms,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY latency_ms) as p95_ms,
  COUNT(*) as request_count,
  AVG(cost_usd) as avg_cost_usd
FROM ai_generations
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY 1
ORDER BY 1;
```

Review your SLOs monthly. If you're consistently at 98% when your SLO is 99.5%, investigate before it becomes a customer problem.

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Add one monitoring layer at a time. Verify each before adding the next. -->

## Step 1 — Add Sentry Error Tracking (First Priority)
The most important monitoring layer — catches all unhandled errors automatically.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add Sentry error tracking to my [FRAMEWORK] app.
1. Install: @sentry/nextjs (or @sentry/python for FastAPI)
2. Configure in sentry.config.ts with:
   - dsn: process.env.SENTRY_DSN
   - environment: process.env.VERCEL_ENV ?? "development"
   - tracesSampleRate: 0.1 (10% of requests traced — not 100%, too expensive)
   - beforeSend: scrub Authorization headers and cookie values
3. Add user context to errors: when a user is logged in, attach { id: userId } to Sentry scope
4. Add a test route: GET /api/sentry-test that throws a deliberate error
5. Add SENTRY_DSN to .env.example

Do not add Sentry to test environment (NEXT_PUBLIC_VERCEL_ENV === "test").
```

**Verify:** Deploy. Hit `/api/sentry-test`. Open Sentry dashboard — the error should appear within 30 seconds with the user context attached.

---

## Step 2 — Add Uptime Monitoring
Know immediately when your app goes down — before users do.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

I want to set up uptime monitoring using Better Uptime (betteruptime.com — free tier).

Generate the /api/health endpoint if it does not exist:
1. GET /api/health
2. Checks: database connection, AI provider key presence
3. Returns: { status: "ok" | "degraded", checks: {...}, timestamp: ISO string }
4. HTTP 200 if all ok, HTTP 503 if degraded

Also give me the configuration to enter in Better Uptime:
- Monitor URL: https://[myapp.com]/api/health
- Check frequency: every 1 minute
- Alert channels: [email / Slack — describe how to configure]
- Expected response: status 200, response contains "ok"
```

**Verify:** Set up the monitor. Check the Better Uptime dashboard shows "Online." Temporarily break the health endpoint (return 503) and confirm you receive an alert within 2 minutes.

---

## Step 3 — Add AI Cost Tracking
Track exactly which features are costing what.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add AI cost tracking to every AI call in my app.
The ai_generations table already exists (from skill-database-storage.md).

Create /lib/ai-tracker.ts with a function: trackAIGeneration(params)
params: { userId, feature, model, promptTokens, completionTokens, latencyMs, status }

This function should:
1. Calculate estimated_cost_usd using current token pricing for the model
2. Insert a row into ai_generations table
3. Be called after EVERY AI API call completes (or fails)
4. Never throw — if tracking fails, log the error but don't break the main flow

Then update these files to call trackAIGeneration:
[LIST YOUR AI WRAPPER FILES — e.g. /lib/openai.ts]
```

**Verify:** Make 3 AI calls through your app. Check the `ai_generations` table — 3 rows should exist with correct token counts and cost estimates.

---

## Step 4 — Add a Cost Alert

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
ai_generations tracking is working.

Create a daily cost alert that:
1. Runs as a cron job at midnight UTC: GET /api/cron/cost-check
2. Queries: total AI spend in the last 24 hours grouped by feature
3. If total_cost > [YOUR DAILY BUDGET e.g. $10]: 
   - Sends an alert email via Resend with the breakdown
   - Logs a warning to Sentry
4. Protects the endpoint with CRON_SECRET env var header check

Also add to vercel.json:
{
  "crons": [{ "path": "/api/cron/cost-check", "schedule": "0 0 * * *" }]
}
```

**Verify:** Manually trigger the endpoint (curl with your CRON_SECRET header). Confirm it returns the cost breakdown. Test the alert by temporarily lowering the threshold.

---

## Step 5 — Add Performance Monitoring (Vercel Analytics)
Track web vitals and identify slow pages.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add Vercel Analytics and Speed Insights to my Next.js app.
1. Install: @vercel/analytics @vercel/speed-insights
2. Add <Analytics /> to /app/layout.tsx
3. Add <SpeedInsights /> to /app/layout.tsx
4. Add custom event tracking for key user actions:
   - Track: reminder_created, lesson_generated, chat_message_sent
   - Use: import { track } from "@vercel/analytics"
   - Call: track("reminder_created", { userId: userId }) after each action

No API key needed — works automatically when deployed to Vercel.
```

**Verify:** Deploy. Perform each tracked action. Check Vercel dashboard → Analytics → Events. Events should appear within a few minutes.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am adding monitoring to my app.
Project context: [PASTE context_anchor JSON]

Current monitoring in place: [DESCRIBE]
Today I am adding: [ONE MONITORING LAYER]

Before writing code:
1. List exactly which files will be created or modified
2. Confirm no business logic will change — only observability additions
3. List any new environment variables needed
```

### Sentry Setup Prompt
```
Set up Sentry for my [Next.js / FastAPI] app.
Project context: [PASTE context_anchor JSON]

Complete setup including:
1. Install command
2. sentry.config.ts (or sentry.server.config.ts for Next.js)
3. Instrumentation: add to instrumentation.ts for Next.js App Router
4. User context: attach userId to every Sentry event when user is logged in
5. PII scrubbing: remove auth headers, cookies, and password fields from events
6. Source maps: upload during build so stack traces show original code
7. Test route: /api/sentry-example that throws on purpose
8. .env.example addition: SENTRY_DSN, SENTRY_ORG, SENTRY_PROJECT

Environment: capture errors in "production" and "preview", suppress in "development".
```

### AI Cost Dashboard Prompt
```
Create a simple AI cost dashboard page for my app.
Project context: [PASTE context_anchor JSON]
ai_generations table is set up and populated.

Build a protected admin page at /admin/ai-costs that shows:
1. Total spend today / this week / this month (in USD)
2. Spend breakdown by feature (bar chart or table)
3. Top 10 most expensive requests today (model, feature, tokens, cost, timestamp)
4. Requests by status (success/failed/timeout) as percentages
5. Average latency by model over last 7 days

Data: query the ai_generations table directly (server component)
Access: only users with role "admin" can view this page
Style: simple table + numbers — no need for fancy charting libraries

Add to /app/(app)/admin/ai-costs/page.tsx
```

### Structured Logging Addition Prompt
```
Add structured logging to all API routes in /app/api/.
Project context: [PASTE context_anchor JSON]

For every request to any API route, log:
{ timestamp, method, path, status, durationMs, userId, error? }

Using: the logger from /lib/logger.ts (already created in skill-debugging-error-handling.md)
Pattern: middleware that wraps every route handler — not inline in each route

Also add:
- Log "slow request" warning when any route takes > 2000ms
- Log "ai_call" info event after every AI call with { feature, model, tokens, latencyMs }
- Never log: request bodies (may contain PII), Authorization headers, cookie values
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Do I need all of this monitoring for an MVP?"

**Minimum you need before any real users:**
1. Sentry error tracking — 20 minutes to set up, catches all crashes automatically
2. Uptime monitoring (Better Uptime free tier) — 10 minutes, alerts when app is down

**Add before paying customers:**
3. AI cost tracking — prevents surprise bills
4. Cost alerts — automated notification when spend spikes

**Add when you have 100+ daily active users:**
5. Vercel Analytics — understand which features users actually use
6. Performance monitoring — identify and fix slow pages

You do not need a full observability stack for an MVP. You need Sentry and uptime monitoring. Everything else is additive value.

---

### "My OpenAI bill is higher than expected. How do I find out why?"

If you have the `ai_generations` table set up, run this query in your database dashboard:

```sql
-- Cost breakdown by feature, last 7 days
SELECT
  feature,
  COUNT(*) as calls,
  SUM(total_tokens) as total_tokens,
  SUM(estimated_cost_usd) as total_cost_usd,
  AVG(latency_ms) as avg_latency_ms
FROM ai_generations
WHERE created_at > NOW() - INTERVAL '7 days'
  AND status = 'success'
GROUP BY feature
ORDER BY total_cost_usd DESC;
```

This tells you exactly which feature is most expensive and how many calls it is making. Common surprises:
- A feature is being called more than expected (user is in a loop)
- `max_tokens` is set too high on a feature that rarely needs that many tokens
- A streaming route is making multiple AI calls per request due to a bug

---

### "What should I actually get alerted on?"

Alert fatigue is real. If every alert is not immediately actionable, you will start ignoring alerts. Alert only on these:

**Page immediately (Slack/email):**
- App is down (health check failing)
- Error rate > 5% in last 5 minutes
- AI cost spike > 3x normal daily spend

**Notify daily (digest):**
- New error types seen for the first time
- p95 response time > 3 seconds on any route
- Daily cost summary

**Review weekly (no alert — just look):**
- Most common error types
- Slowest routes
- Highest-cost features

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## Observability
- Error tracking: Sentry — DSN in SENTRY_DSN env var, PII scrubbing enabled
- Uptime monitoring: Better Uptime — health endpoint at /api/health
- AI cost tracking: ai_generations table + daily alert cron at /api/cron/cost-alert
- Analytics: Vercel Analytics (web vitals) + PostHog (custom events)
- Request correlation: X-Request-ID header in middleware — `lib/logger.ts`
- SLOs: p95 AI generation < 10s, error rate < 2%, uptime > 99.5%
- Runbooks: docs/runbooks/
- Alert thresholds: DAILY_AI_BUDGET_USD=[X] env var
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Monitoring Setup

### Monitoring Verification Checklist
```
□ Sentry is receiving errors (hit /api/sentry-test → verify in Sentry dashboard)
□ Sentry has user context on errors (userId appears in error details)
□ Uptime monitor is active and showing "Online"
□ Uptime monitor sends alert when app is down (test by returning 503 from /api/health)
□ ai_generations table is populating (make an AI call → check table)
□ Cost tracking calculates USD correctly (verify calculation against OpenAI pricing)
□ Cost alert fires when threshold is exceeded (manually test with low threshold)
□ Vercel Analytics shows events (perform tracked actions → verify in dashboard)
```

### Test Each Alert Works
```bash
# Test 1: Sentry error capture
curl https://yourapp.com/api/sentry-test
# → Check Sentry dashboard for the error within 30 seconds

# Test 2: Uptime alert (temporarily break health endpoint)
# In health route: return new Response(null, { status: 503 });
# → Deploy → Wait for alert → Fix → Verify "recovered" alert

# Test 3: Cost alert (lower threshold temporarily)
# Set DAILY_AI_BUDGET_USD=0.01 → Make one AI call → Run cron manually
# curl -H "Authorization: Bearer $CRON_SECRET" https://yourapp.com/api/cron/cost-check
# → Should receive alert email

# Test 4: Structured logs (check in Vercel Functions)
# Make a request → Open Vercel dashboard → Functions → find the log entry
```

### Common Monitoring Issues

| Issue | Meaning | Fix |
|---|---|---|
| Sentry not receiving errors | SENTRY_DSN not set or wrong | Check env var; verify DSN from Sentry project settings |
| Too many Sentry events | Noisy errors filling up free quota | Add `ignoreErrors` to Sentry config for known ignoreable errors |
| ai_generations not tracking | trackAIGeneration not called | Verify wrapper is called in every AI function |
| Cost estimates wildly off | Using outdated token prices | Update COST_PER_TOKEN in ai-tracker.ts with current OpenAI pricing |
| Uptime alert not firing | Alert contact not configured | Verify email/Slack alert channel in Better Uptime settings |
| Vercel Analytics not showing | Package not installed or component missing | Verify `<Analytics />` in layout.tsx; check deployment |

</testing_and_qa>

---

<common_patterns>

## Reusable Monitoring Patterns

### Pattern 1: AI Cost Tracker
```typescript
// /lib/ai-tracker.ts
import { db } from "@/lib/db";
import { logger } from "@/lib/logger";

// Current pricing (update quarterly)
const TOKEN_COST_USD: Record<string, { input: number; output: number }> = {
  "gpt-4o":                  { input: 0.0000025,  output: 0.00001 },
  "gpt-4o-mini":             { input: 0.00000015, output: 0.0000006 },
  "claude-3-5-sonnet":       { input: 0.000003,   output: 0.000015 },
  "claude-3-haiku":          { input: 0.00000025, output: 0.00000125 },
  "text-embedding-3-small":  { input: 0.00000002, output: 0 },
};

interface TrackParams {
  userId?: string;
  feature: string;
  model: string;
  promptTokens: number;
  completionTokens: number;
  latencyMs: number;
  status: "success" | "failed" | "timeout";
}

export async function trackAIGeneration(params: TrackParams): Promise<void> {
  try {
    const pricing = TOKEN_COST_USD[params.model] ?? { input: 0, output: 0 };
    const estimatedCostUsd =
      params.promptTokens * pricing.input +
      params.completionTokens * pricing.output;

    await db.aiGeneration.create({
      data: {
        userId: params.userId ?? null,
        feature: params.feature,
        model: params.model,
        promptTokens: params.promptTokens,
        completionTokens: params.completionTokens,
        totalTokens: params.promptTokens + params.completionTokens,
        estimatedCostUsd,
        latencyMs: params.latencyMs,
        status: params.status,
      },
    });

    logger.info("[ai-tracker]", `AI call tracked`, {
      feature: params.feature,
      model: params.model,
      tokens: params.promptTokens + params.completionTokens,
      costUsd: estimatedCostUsd.toFixed(6),
      latencyMs: params.latencyMs,
    });
  } catch (error) {
    // Never let tracking failure break the main flow
    logger.error("[ai-tracker]", "Failed to track AI generation", { error: String(error) });
  }
}
```

### Pattern 2: Enhanced Health Check Endpoint
```typescript
// /app/api/health/route.ts
import { db } from "@/lib/db";

export const dynamic = "force-dynamic";

export async function GET() {
  const start = Date.now();
  const checks: Record<string, { status: "ok" | "error"; latencyMs?: number }> = {};

  // Database check
  const dbStart = Date.now();
  try {
    await db.$queryRaw`SELECT 1`;
    checks.database = { status: "ok", latencyMs: Date.now() - dbStart };
  } catch {
    checks.database = { status: "error" };
  }

  // AI provider check (key presence)
  checks.ai_provider = {
    status: process.env.OPENAI_API_KEY ? "ok" : "error",
  };

  // Derived overall status
  const criticalChecks = ["database", "ai_provider"];
  const allCriticalOk = criticalChecks.every(k => checks[k]?.status === "ok");
  const overallStatus = allCriticalOk ? "ok" : "degraded";

  return Response.json(
    {
      status: overallStatus,
      checks,
      totalLatencyMs: Date.now() - start,
      timestamp: new Date().toISOString(),
      version: process.env.VERCEL_GIT_COMMIT_SHA?.slice(0, 7) ?? "local",
    },
    { status: allCriticalOk ? 200 : 503 }
  );
}
```

### Pattern 3: Daily Cost Alert Cron
```typescript
// /app/api/cron/cost-check/route.ts
import { db } from "@/lib/db";
import { sendEmail } from "@/lib/email";

export async function GET(req: Request) {
  // Protect with cron secret
  const authHeader = req.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }

  const DAILY_BUDGET = parseFloat(process.env.DAILY_AI_BUDGET_USD ?? "50");
  const yesterday = new Date(Date.now() - 24 * 60 * 60 * 1000);

  // Query cost by feature for last 24 hours
  const costByFeature = await db.aiGeneration.groupBy({
    by: ["feature"],
    where: { createdAt: { gte: yesterday }, status: "success" },
    _sum: { estimatedCostUsd: true, totalTokens: true },
    _count: true,
    orderBy: { _sum: { estimatedCostUsd: "desc" } },
  });

  const totalCost = costByFeature.reduce(
    (sum, row) => sum + (row._sum.estimatedCostUsd ?? 0),
    0
  );

  if (totalCost > DAILY_BUDGET) {
    await sendEmail({
      to: process.env.ALERT_EMAIL!,
      subject: `⚠️ AI Cost Alert: $${totalCost.toFixed(2)} spent in last 24h`,
      html: `
        <h2>Daily AI Cost Exceeded Budget</h2>
        <p>Total: <strong>$${totalCost.toFixed(4)}</strong> (budget: $${DAILY_BUDGET})</p>
        <table>
          <tr><th>Feature</th><th>Calls</th><th>Cost (USD)</th></tr>
          ${costByFeature.map(r => `
            <tr>
              <td>${r.feature}</td>
              <td>${r._count}</td>
              <td>$${(r._sum.estimatedCostUsd ?? 0).toFixed(4)}</td>
            </tr>
          `).join("")}
        </table>
      `,
    });
  }

  return Response.json({ totalCostUsd: totalCost, features: costByFeature.length, alertSent: totalCost > DAILY_BUDGET });
}
```

### Pattern 4: Alert Runbooks — Every Alert Needs a "Now What?"

An alert without a runbook creates panic. Every alert should link to a short Markdown file that answers: "What does this alert mean? What should I check first? How do I fix it?"

```markdown
<!-- docs/runbooks/ai-cost-spike.md -->
# Alert: Daily AI Cost Spike

**Trigger:** Daily AI spend > $50 (or your threshold)

## What this means
AI generation costs exceeded the expected daily budget. This usually means:
- A single user is running many requests (check ai_generations by userId)
- A new feature is generating more tokens than expected
- A runaway background job

## Investigation Steps
1. Check Supabase: `SELECT user_id, SUM(cost_usd) FROM ai_generations WHERE DATE(created_at) = CURRENT_DATE GROUP BY 1 ORDER BY 2 DESC LIMIT 10`
2. Check for background jobs stuck in a loop (Inngest dashboard → Functions → check for high invocation counts)
3. Check Sentry for any new errors that might be causing retries

## Resolution
- If one user: check if they're on free tier and should be rate-limited
- If background job: cancel the job in Inngest dashboard, fix the bug, redeploy
- Immediate relief: lower the DAILY_AI_BUDGET_USD env var to trigger earlier alerts

## Escalation
If unable to resolve in 30 minutes, pause AI generation features via feature flag.
```

Store runbooks in `docs/runbooks/`. In your alerting tool (Better Uptime, PagerDuty, etc.), link each alert to its runbook URL.

### Pattern 5: Distributed Request Correlation — Trace Errors Across Services

When a user reports "I got an error", you need to find the exact log entry. Add a request ID to every request and propagate it through service calls.

```typescript
// middleware.ts — add X-Request-ID to every request
import { NextResponse } from "next/server"
import type { NextRequest } from "next/server"
import crypto from "crypto"

export function middleware(request: NextRequest) {
  const requestId = request.headers.get("x-request-id") ?? crypto.randomUUID()

  const response = NextResponse.next()
  response.headers.set("x-request-id", requestId)

  return response
}

// lib/logger.ts — include requestId in every log entry
export function createLogger(requestId: string) {
  return {
    info: (message: string, data?: object) =>
      console.log(JSON.stringify({ level: "info", requestId, message, ...data, timestamp: new Date().toISOString() })),
    error: (message: string, error?: unknown) =>
      console.error(JSON.stringify({ level: "error", requestId, message, error: String(error), timestamp: new Date().toISOString() })),
  }
}

// In route handler:
// const requestId = req.headers.get("x-request-id") ?? crypto.randomUUID()
// const log = createLogger(requestId)
// log.info("Starting AI generation", { userId, feature: "chat" })

// Return requestId in error responses so users can report it:
// return Response.json({ success: false, error: "Something went wrong", requestId }, { status: 500 })
```

When a user reports a problem with a `requestId`, you can grep your logs: `grep "requestId:\"abc-123\"" logs/` to find the full request trace.

### Pattern 6: Vercel Analytics Custom Events
```typescript
// /lib/analytics.ts — centralise all tracking calls
import { track } from "@vercel/analytics";

export const analytics = {
  reminderCreated: (userId: string) =>
    track("reminder_created", { userId }),

  lessonGenerated: (topic: string, level: string, durationMs: number) =>
    track("lesson_generated", { topic, level, durationMs }),

  chatMessageSent: (messageLength: number) =>
    track("chat_message_sent", { messageLength }),

  featureUsed: (featureName: string) =>
    track("feature_used", { feature: featureName }),
};

// Usage in components:
// analytics.reminderCreated(userId);
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Scrub PII from Error Reports
Before any error reaches Sentry or external logging, sensitive data must be removed:
```typescript
Sentry.init({
  beforeSend(event) {
    // Remove auth headers
    if (event.request?.headers) {
      delete event.request.headers["authorization"];
      delete event.request.headers["cookie"];
    }
    // Remove request bodies containing passwords or tokens
    if (event.request?.data) {
      const data = event.request.data as Record<string, unknown>;
      delete data.password;
      delete data.token;
      delete data.apiKey;
    }
    return event;
  },
});
```

### Rule 2: Protect Cron and Admin Endpoints
Any endpoint that accesses aggregate data or triggers system actions must be authenticated:
```typescript
// Cron endpoints: check CRON_SECRET header
// Admin pages: check user has admin role
// Health endpoint: public (intentionally — no sensitive data)
```

### Rule 3: Never Include User PII in Metrics
Analytics events and cost tracking should use user IDs, not emails or names:
```typescript
// ❌ Exposes PII to analytics platform
track("lesson_created", { userEmail: user.email });

// ✅ Anonymous identifier only
track("lesson_created", { userId: user.id });
```

### Rule 4: Set Data Retention Limits
The `ai_generations` table will grow indefinitely. Set up automatic cleanup:
```sql
-- Add to a weekly cron job
DELETE FROM ai_generations
WHERE created_at < NOW() - INTERVAL '90 days';
```

Retaining more than 90 days of individual AI call data is usually not valuable and creates GDPR/privacy obligations.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Setting Up Monitoring After the First Incident
By the time a production bug occurs without monitoring, you have no data to diagnose it. You are flying blind in a post-mortem.  
**Fix:** Sentry + uptime monitoring before the first user arrives. Non-negotiable.

### ❌ Alerting on Everything
Every alert that fires and gets ignored makes the next alert less likely to be acted on. If your Slack channel has 50 Sentry alerts per day, the one real critical one gets missed.  
**Fix:** Configure Sentry to alert only on new error types (not every occurrence). Set error volume thresholds.

### ❌ Not Tracking AI Costs Until the Bill Arrives
A lesson generation feature with a prompt that is too long quietly costs $0.08 per call. At 500 calls/day it is $40/day. You find out on the monthly invoice.  
**Fix:** Set up `ai_generations` tracking and cost alerts before enabling any feature with AI calls.

### ❌ Health Check That Always Returns 200
```typescript
// ❌ Health check that never reports problems
export async function GET() {
  return Response.json({ status: "ok" }); // Never actually checks anything
}
```
**Fix:** Health check must verify actual dependencies (DB connection, required env vars). See Pattern 2.

### ❌ Monitoring Only Errors, Not Slowness
An AI endpoint that takes 8 seconds is not an error — it just feels broken to users. Error tracking will not catch this.  
**Fix:** Track and alert on p95 response time, not just error rates. Vercel Analytics provides this automatically.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Monitoring

### Add Real User Monitoring (RUM)
Track what real users experience — not just server-side performance:
```
Vercel Speed Insights (already added in Step 5) tracks:
- Largest Contentful Paint (LCP): how fast does the main content load?
- First Input Delay (FID): how responsive is the page to clicks?
- Cumulative Layout Shift (CLS): does the layout jump around while loading?

Target thresholds (Google's "Good" rating):
- LCP < 2.5 seconds
- FID < 100ms
- CLS < 0.1
```

### Add Distributed Tracing for Multi-Service Apps
When your app spans multiple services (Next.js + Python AI service + multiple databases):
```
Tool: Sentry Performance or OpenTelemetry
Ask your AI agent: "Add OpenTelemetry tracing to my Next.js app and FastAPI service.
Trace the full request journey: Next.js route → API call to FastAPI → AI provider → DB.
Export traces to [Sentry / Jaeger / Honeycomb]."
```

### Add Anomaly Detection for Cost Spikes
```typescript
// Compare today's spend to the 7-day rolling average
// Alert if today > 2x the average
const rollingAvg = await db.aiGeneration.aggregate({
  _avg: { estimatedCostUsd: true },
  where: {
    createdAt: { gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) },
    status: "success",
  },
});
const todayCost = /* today's spend */;
if (todayCost > rollingAvg._avg.estimatedCostUsd! * 2) {
  // Send anomaly alert
}
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — How Monitoring Caught a Cost Anomaly
**Situation:** 3 weeks after launch, the daily AI cost alert fired: $24 spent in 24 hours. Budget was $5/day.

**Investigation using ai_generations table:**
```sql
SELECT user_id, COUNT(*), SUM(total_tokens), SUM(estimated_cost_usd)
FROM ai_generations
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY user_id
ORDER BY SUM(estimated_cost_usd) DESC
LIMIT 5;
```

**Finding:** One user account had made 847 API calls. Manual review showed the user was in an automated test loop — their testing script was hammering the chat endpoint.

**Fix applied:**
1. Rate limiting added: 50 AI requests per user per day
2. The test user was excluded from the rate limit via a flag

**Without monitoring:** This would have run for days before the monthly bill revealed it. Cost saved: ~$150 over the week before it was noticed.

---

### Case Study 2: EdTech App — Uptime Alert That Prevented Churn
**Situation:** At 2:47am on a Tuesday, the uptime monitor fired: health check failing.

**Alert received via Slack:** "LearnFlow is DOWN — /api/health returning 503"

**Investigation (5 minutes):**
- Health check showed: `{ database: "error", ai_provider: "ok" }`
- Supabase status page: "Incident: degraded performance affecting connection pooling"
- Supabase resolved: 3:12am (25 minutes of downtime)

**Without monitoring:** This would have been discovered by students at 7am when school started. Support tickets would have started arriving. With monitoring: issue logged, Supabase incident tracked, and a response prepared for any users who experienced issues — all before anyone noticed.

**Post-incident improvement added:** Retry logic on DB connections (3 attempts with 1 second backoff) to handle brief Supabase degradations gracefully without full downtime.

</real_world_examples>

</skill_document>
