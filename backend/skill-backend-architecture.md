---
name: backend-architecture
description: Designs scalable backend architecture for AI products. Use when starting a new project, choosing a tech stack, structuring folders, designing database layer with serverless connection pooling, background job orchestration, event-driven patterns, or setting up service-to-service communication before writing any code.
---

# Skill: Backend Architecture

```json
{
  "skill_id": "backend-architecture",
  "category": "Architecture & Backend",
  "version": "2.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2026"
}
```

---

<skill_document>

<title>Backend Architecture — Designing a Scalable, Production-Ready Backend for AI Products</title>

<overview>

## What This Skill Enables
- Design a backend architecture that can handle real users, real data, and real AI workloads without painful rewrites at scale
- Make confident decisions about stack, database pooling, background jobs, and service splitting before writing a single line of code
- Understand the architectural patterns that separate hobby apps from production systems
- Identify when to evolve from monolith to distributed services, and do it at exactly the right time

## Why It Matters for Vibe Coders
Backend architecture is the set of structural decisions that determine how your app handles requests, processes data, runs background work, and scales. For AI products, these decisions are especially consequential: AI workloads are slow (seconds, not milliseconds), expensive (tokens cost money), and stateful (conversations, documents, embeddings). The right architecture at the start prevents painful rewrites at 1000 users. Most vibe-coded AI apps fail not because of bad ideas but because the backend was never properly designed. Without architecture decisions made upfront, you end up with spaghetti code that breaks when you add a second feature or hits your first traffic spike.

## When to Use This Skill
- At the very beginning of any new project, before writing any code
- When restarting or refactoring a project that has grown messy
- When adding a major new feature that touches multiple parts of the app
- When you notice background jobs are piling up or database connections are maxing out

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

> **INSTRUCTION FOR VIBE CODER:** Copy this block into your AI agent at the start of every new coding session. Fill in the `[REPLACE]` fields with your actual project details. This prevents the AI from hallucinating files or structures that do not exist.

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "app_type": "[REPLACE: e.g. EdTech web app / Conversational reminder mobile app]",
    "primary_language": "[REPLACE: e.g. TypeScript / Python]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI / Express]",
    "hosting_target": "[REPLACE: e.g. Vercel / Railway / Supabase / AWS]",
    "ai_provider": "[REPLACE: e.g. OpenAI / Anthropic / Gemini]",
    "database": "[REPLACE: e.g. PostgreSQL via Supabase / MongoDB Atlas]",
    "auth_provider": "[REPLACE: e.g. Clerk / Supabase Auth / Auth0]",
    "background_job_tool": "[REPLACE: e.g. Inngest / Trigger.dev / none yet]",
    "current_backend_files": "[REPLACE: list key files e.g. /api/chat.ts, /lib/db.ts]",
    "features_already_built": "[REPLACE: e.g. user login, basic chat UI]",
    "features_to_build_next": "[REPLACE: e.g. lesson generation endpoint, reminder scheduler]",
    "current_user_count": "[REPLACE: e.g. <50 / 100–500 / 1000+ for scaling decisions]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Backend Architecture

### Mental Model 1: The City Infrastructure Model
Your app is a city. API routes are roads (immediate, synchronous traffic). Background jobs are the postal service (async, reliable delivery). The database is utilities (shared infrastructure, connection limits matter). Events are city announcements (broadcast to whoever needs to know). Build the right infrastructure for your population size — a village doesn't need a subway system, and a city of 1 million can't survive with village roads.

### Mental Model 2: Synchronous vs. Asynchronous Split
The most important architectural decision for AI products. If a task takes > 3 seconds, it should be async (background job + webhook/polling). If it takes < 3 seconds, it can be synchronous. AI generation is almost always async at production scale.

| Task | Synchronous? | Why |
|---|---|---|
| Chat reply (streaming) | ✅ Yes | User is watching in real time |
| Document ingestion (RAG) | ❌ No — async | Takes 10–60 seconds |
| Batch AI processing | ❌ No — async | Could run for minutes |
| Sending emails | ❌ No — async | Retries needed |
| Nightly reports | ❌ No — cron job | Not user-initiated |
| Webhook processing | ❌ No — async | Must return 200 immediately |

### Mental Model 3: The Monolith-First Rule
Start with one Next.js app. Only split into separate services when you have a concrete reason: different scaling requirements, different deployment frequencies, or a team boundary. Premature microservices is the most common architectural mistake vibe coders make.

### Mental Model 4: Database Connection Pooling in Serverless
Each serverless function invocation creates a new process with its own database connections. With 100 concurrent invocations, you could have 100 connections open — most databases only allow 100–200 total. The solution: connection pooling. PgBouncer (via Supabase) or connection limits in your database client prevent connection exhaustion.

</mental_models>

---

<system_design_breakdown>

## Core Components of an AI Product Backend

```
┌─────────────────────────────────────────────────────┐
│                    CLIENT (Browser / App)            │
└────────────────────────┬────────────────────────────┘
                         │ HTTP / WebSocket
┌────────────────────────▼────────────────────────────┐
│                    API LAYER                         │
│   /api/auth   /api/users   /api/ai   /api/data      │
└──────┬──────────────┬──────────────┬────────────────┘
       │              │              │
┌──────▼──────┐ ┌────▼─────┐ ┌─────▼──────────────┐
│  Auth       │ │  Business│ │  AI Orchestration  │
│  Service    │ │  Logic   │ │  (Prompt→LLM→Parse)│
└──────┬──────┘ └────┬─────┘ └─────┬───────────────┘
       │             │              │
┌──────▼─────────────▼──────────────▼───────────────┐
│                 DATABASE LAYER                     │
│    (PostgreSQL/MongoDB via PgBouncer pooling)      │
└────────────────────┬─────────────────────────────┘
       ┌────────────┬┘
       │            │
┌──────▼──────┐ ┌──▼─────────────────┐
│   EVENTS    │ │  BACKGROUND JOBS   │
│  (pub/sub)  │ │  (Inngest/Trigger) │
└─────────────┘ └────────────────────┘
```

### Stack Decision Tree

```
Building an AI web product?
├── Primary interface is web UI → Next.js App Router (full-stack)
│   ├── AI features use streaming → add Edge runtime to AI routes
│   ├── Heavy background processing → add Inngest (stays in Next.js)
│   └── Need Python ML libraries → add FastAPI sidecar service
└── Primary interface is API-only → FastAPI
    ├── Need async task queue → Celery + Redis
    └── Hosting → Railway or Fly.io
```

### Component Descriptions

| Component | What It Does | Recommended Tool |
|---|---|---|
| API Layer | Receives and routes all requests | Next.js App Router / FastAPI / Express |
| Auth Service | Handles login, sessions, tokens | Clerk / Supabase Auth / Auth0 |
| Business Logic | Core feature processing | Your own functions/services |
| AI Orchestration | Calls LLM, manages prompts, parses responses | LangChain / Vercel AI SDK / custom |
| Database Layer | Stores and retrieves all data | Supabase (PostgreSQL) / MongoDB Atlas |
| Connection Pooling | Manages DB connections in serverless | PgBouncer (Supabase port 6543) |
| Background Jobs | Async tasks: emails, reminders, AI batch | Inngest / Trigger.dev / Upstash QStash |
| Event System | Decouples services, enables event-driven flow | Inngest events / RabbitMQ / AWS SNS+SQS |

### Recommended Folder Structure (Next.js)

```
app/
  api/                    # API route handlers
    auth/                 # Clerk/Supabase auth webhooks
    [feature]/
      route.ts            # GET, POST handlers
  (app)/                  # App shell routes (authenticated)
    dashboard/
    [feature]/
  (marketing)/            # Public routes
    page.tsx
components/
  ui/                     # shadcn/ui primitives (button, input, etc.)
  features/               # Feature-specific composite components
lib/
  ai/                     # AI utilities (prompts, models, streaming)
  db/                     # Prisma client singleton
  auth/                   # Auth helpers (getCurrentUser, requireAuth)
  jobs/                   # Inngest job definitions
  api/                    # Typed fetch wrappers for client→server calls
  utils/                  # Shared utilities
types/                    # TypeScript types and Zod schemas
prisma/
  schema.prisma
  migrations/
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Complete and verify each step before moving to the next. -->

## Phase 1: Project Setup

```bash
# Create Next.js app with all dependencies
npx create-next-app@latest my-app --typescript --tailwind --app
cd my-app
npm install prisma @prisma/client zod @clerk/nextjs inngest
npm install -D @types/node
npx prisma init --datasource-provider postgresql
```

---

## Phase 2: Database Client (Singleton — Critical for Serverless)

This is non-negotiable. In serverless environments, each function invocation creates a new module instance. Without the global singleton pattern, you'll exhaust your database connection pool at scale.

```typescript
// lib/db/index.ts
import { PrismaClient } from "@prisma/client"

// CRITICAL: Singleton pattern prevents connection pool exhaustion
// Each serverless invocation reuses the same Prisma instance
const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }

export const db =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === "development" ? ["query", "error", "warn"] : ["error"],
    datasources: {
      db: {
        url: process.env.DATABASE_URL,
      },
    },
  })

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db
```

---

## Phase 3: Connection Pooling for Serverless (PgBouncer)

Prisma's default connection pool creates up to 10 connections per instance. With 100 concurrent Vercel function invocations, that's 1,000 connections — most databases allow 100–200 total.

**Solution:** Use PgBouncer connection pooling for app traffic, direct connection for migrations.

```
# .env — use PgBouncer URL for serverless
DATABASE_URL="postgres://[user]:[pass]@[host]:6543/[db]?pgbouncer=true&connection_limit=1"
DIRECT_URL="postgres://[user]:[pass]@[host]:5432/[db]"  # For migrations only
```

```prisma
// prisma/schema.prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")  // Used by prisma migrate — bypasses PgBouncer
}
```

**How to set this up with Supabase:**
1. In your Supabase dashboard → Settings → Database → Connection pooling
2. Enable connection pooling
3. Use port `6543` with `pgbouncer=true` in CONNECTION_URL
4. Keep the direct URL (port `5432`) for migrations only

Why `connection_limit=1`? PgBouncer handles connection pooling centrally — each serverless instance needs only one connection to PgBouncer, which maintains the pool for the whole application.

---

## Phase 4: Auth Helper (use everywhere)

```typescript
// lib/auth/index.ts
import { auth, currentUser } from "@clerk/nextjs/server"

export async function requireAuth() {
  const { userId } = await auth()
  if (!userId) throw new Error("Unauthorized")
  return userId
}

export async function getCurrentUser() {
  const user = await currentUser()
  if (!user) throw new Error("Unauthorized")
  return { id: user.id, email: user.emailAddresses[0]?.emailAddress }
}
```

---

## Phase 5: Background Jobs with Inngest

Most AI operations should be background jobs, not synchronous API responses. Inngest handles retries, concurrency controls, and webhooks automatically.

```typescript
// lib/jobs/index.ts
import { Inngest } from "inngest"

export const inngest = new Inngest({ id: "my-app" })
```

```typescript
// lib/jobs/document-ingest.ts
import { inngest } from "./index"
import { ingestDocument } from "@/lib/rag/ingest"

export const documentIngestJob = inngest.createFunction(
  {
    id: "document-ingest",
    retries: 3,  // Inngest handles retry logic with exponential backoff
  },
  { event: "document/uploaded" },
  async ({ event, step }) => {
    const { documentId, userId, content } = event.data

    // step.run() creates a checkpoint — if this crashes, Inngest retries from here
    await step.run("extract-text", async () => {
      // Extract text from PDF/DOCX if needed
      return { content }
    })

    await step.run("chunk-and-embed", async () => {
      await ingestDocument({ content, documentId, userId })
    })

    await step.run("notify-user", async () => {
      // Send email or push notification that document is ready
      await sendNotification(userId, "Your document is ready to search!")
    })

    return { success: true, documentId }
  }
)
```

```typescript
// app/api/inngest/route.ts — Inngest webhook endpoint
import { serve } from "inngest/next"
import { inngest } from "@/lib/jobs"
import { documentIngestJob } from "@/lib/jobs/document-ingest"

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [documentIngestJob],
})
```

Trigger the job from your API route (returns immediately — job runs in background):

```typescript
// app/api/documents/upload/route.ts
await inngest.send({
  name: "document/uploaded",
  data: { documentId, userId, content }
})

return NextResponse.json({ success: true, documentId, status: "processing" }, { status: 202 })
```

---

## Phase 6: Event-Driven Architecture Pattern

Start with direct function calls. Move to events when a single action triggers > 3 side effects or when side effects have different failure characteristics.

**Direct call approach (fine for < 3 side effects):**
```typescript
async function afterUserSignup(userId: string) {
  await createUserProfile(userId)
  await sendWelcomeEmail(userId)
  await trackSignupEvent(userId)
}
```

**Event-driven approach (better when you have > 3 side effects):**
```typescript
// Emit one event, multiple handlers react independently
await inngest.send({
  name: "user/signed-up",
  data: { userId }
})

// Separate handlers:
export const createProfileJob = inngest.createFunction(
  { id: "user-create-profile" },
  { event: "user/signed-up" },
  async ({ event }) => {
    await createUserProfile(event.data.userId)
  }
)

export const sendWelcomeEmailJob = inngest.createFunction(
  { id: "user-send-welcome-email" },
  { event: "user/signed-up" },
  async ({ event }) => {
    await sendWelcomeEmail(event.data.userId)
  }
)

export const trackSignupJob = inngest.createFunction(
  { id: "user-track-signup" },
  { event: "user/signed-up" },
  async ({ event }) => {
    await analytics.track("user_signup", { userId: event.data.userId })
  }
)
```

Move to event-driven when: (1) a single action triggers > 3 side effects, (2) side effects have different retry/failure characteristics, or (3) you want to add/remove side effects without touching the core flow.

---

## Phase 7: Service-to-Service Authentication

When you have a separate FastAPI sidecar or any service-to-service HTTP call:

**Option A: Shared Secret (simplest — fine for internal services on the same VPC/platform)**

```typescript
// Next.js → Python service
export async function callPythonService(endpoint: string, body: unknown) {
  const res = await fetch(`${process.env.PYTHON_SERVICE_URL}${endpoint}`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Service-Secret": process.env.SERVICE_SECRET!,  // shared between services
    },
    body: JSON.stringify(body),
  })
  if (!res.ok) throw new Error(`Python service error: ${res.status}`)
  return res.json()
}
```

In FastAPI (receiving end):
```python
from fastapi import Header, HTTPException

async def verify_service(x_service_secret: str = Header(...)):
    if x_service_secret != os.getenv("SERVICE_SECRET"):
        raise HTTPException(status_code=401)
```

**Option B: JWT Tokens (better for production, cross-infrastructure)**

Each service signs a JWT. The receiving service verifies the signature. More complex but production-standard.

---

## Phase 8: Scaling and Growth Signals

These concrete signals indicate when to split your monolith:

**Split into separate services when you observe:**
- A single slow background job is blocking other jobs (different scaling requirements)
- You need Python ML libraries that can't run in Node.js (FastAPI sidecar)
- A different team owns a distinct domain (team boundary)
- A feature has 10x the traffic of everything else (isolated scaling)

**DO NOT split because it "feels cleaner"** — the operational overhead is real.

---

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt (use every time you open a new session)
```
I am continuing work on my app. Here is my current project context:
[PASTE context_anchor JSON]

Files I worked on last session: [LIST FILES]
What I completed: [DESCRIBE]
What I am building today: [ONE THING ONLY]

Before writing any code, confirm you understand the current state of the project
and tell me what file(s) you will touch today.
```

### Architecture Review Prompt
```
Review my current backend architecture based on this context:
[PASTE context_anchor JSON]

Identify:
1. Any missing components for an AI-powered app
2. Database connection pooling setup (are we using PgBouncer?)
3. Background job architecture (where do long-running tasks live?)
4. Any security risks in the current structure
5. The single most important thing to build next

Do not rewrite anything. Just give me the assessment.
```

### Folder Structure Prompt
```
Generate the production-ready folder structure for a [STACK] backend
that supports: [LIST YOUR KEY FEATURES].
Include a one-line comment in each file describing its purpose.
Do not write any logic.
```

### Single Route Build Prompt
```
Build ONLY the [ROUTE NAME] route for my [APP TYPE] app.
Context: [PASTE context_anchor JSON]
Requirements: [LIST 3-5 SPECIFIC REQUIREMENTS FOR THIS ROUTE]
After generating, tell me exactly how to test it.
```

### Background Job Build Prompt
```
Add an Inngest background job called [JOB_NAME] that:
[DESCRIBE WHAT THE JOB DOES]
Use step.run() to create checkpoints after each major step.
Trigger it from [WHICH API ROUTE OR EVENT].
Handle retries and failures gracefully.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Should I use Next.js or a separate backend like FastAPI?"

**Choose Next.js App Router if:**
- You are building a web app
- You want to deploy to Vercel in one click
- You have fewer than ~50 API routes
- Speed to launch matters more than maximum flexibility

**Choose FastAPI (Python) if:**
- Your app is AI-heavy and you are using Python AI libraries (LangChain, LlamaIndex)
- You need complex background processing
- You are comfortable with slightly more DevOps setup

**The honest answer for most AI product builders:** Start with Next.js. You can always extract a Python microservice for your AI layer later.

---

### "What is serverless and why does it matter for my architecture?"

Serverless = your code only runs when someone calls it. You don't manage a server that runs 24/7.

**Why it matters for you:** It's cheaper at low traffic, scales automatically, and works perfectly with Vercel and Supabase. Most vibe-coded AI apps should use serverless functions.

**The critical difference from traditional servers:** Each invocation is a new process. Your database connections, file handles, and global state don't persist. This is why you need the singleton pattern for Prisma and why you need PgBouncer for connection pooling.

**When you need a real server instead:** If your app has long-running tasks (>30 seconds) like video processing, batch AI generation, or WebSocket connections that stay open. In that case, look at Railway or Render for always-on servers.

---

### "When should I move background jobs from my API to a proper queue?"

**Start with:** Nothing — just return 202 Accepted immediately, don't wait for long work to finish.

**Add Inngest when:**
- A single API route is taking > 5 seconds to respond
- You have more than 3 different background tasks
- You need retries and guaranteed delivery

Inngest costs next to nothing at small scale and saves you weeks of DevOps work.

---

### "How do I know when to split my monolith into microservices?"

Don't split until you have a concrete, measurable reason:
1. **Different scaling profiles** — your AI endpoint needs 10x more resources than your chat endpoint
2. **Team boundary** — two different teams maintaining different parts
3. **Technology requirement** — need Python ML libraries that can't run in Node.js
4. **Different deployment cadence** — one service deploys 5x more often than the other

**The most common mistake:** Splitting "for cleanness" before hitting any of these signals. Microservices add debugging complexity, deployment complexity, and network latency. Start monolith, split only when forced.

---

### "What is PgBouncer and why do I need it for serverless?"

PgBouncer is a connection pooler. Your serverless functions don't each maintain direct connections to your database — that would exhaust your connection limit immediately. Instead, each function connects to PgBouncer (using just 1 connection), and PgBouncer manages a shared pool of connections to your real database.

**In Supabase:** It's automatic. Use port 6543 instead of 5432, add `?pgbouncer=true&connection_limit=1` to your DATABASE_URL, and you're done.

---

### 🗂️ Update Your AGENT_CONTEXT.md

This file should be your primary foundation. After initial setup, populate these fields in your project's AGENT_CONTEXT.md:

```md
## Architecture
- Stack: Next.js App Router + Prisma + [Clerk | Supabase Auth]
- Database: PostgreSQL via Supabase — connection pooling via PgBouncer port 6543
- Background jobs: Inngest — job definitions in `lib/jobs/`
- Service architecture: [monolith | monolith + FastAPI sidecar at SERVICE_URL]
- Service-to-service auth: [shared secret | not applicable]
- Event pattern: [direct function calls | Inngest events for user/signed-up etc.]

## Key Files
- DB client: `lib/db/index.ts` (singleton)
- Auth helpers: `lib/auth/index.ts`
- Inngest client: `lib/jobs/index.ts`
- API routes: `app/api/[feature]/route.ts`
- Folder conventions: [link to your specific structure]

## Scaling Decisions
- Connection limit per serverless instance: 1 (PgBouncer handles pooling)
- Background job retries: 3 (Inngest default)
- Monolith split trigger: [not yet | FastAPI sidecar for ML — deployed at X]

## Database Connection
- PgBouncer URL: [database_url with port 6543]
- Direct URL: [direct_url with port 5432 — migrations only]
- Current user count: [for tracking when to optimize]
```

**Why this matters:** Architecture decisions are the hardest to change later. Capturing them in AGENT_CONTEXT.md means every coding session starts with the right structure — preventing your agent from suggesting patterns that conflict with your existing architecture.

</vibe_coder_bridge>

---

<testing_and_qa>

## How to Verify Your Backend Architecture is Working

### After Folder Structure Setup
```bash
# Run this in your terminal to confirm all files exist
ls -R ./app/api    # Next.js
# or
ls -R ./routers    # FastAPI
```
✅ Pass: All expected folders and files are present
❌ Fail: Re-run the folder structure prompt and specify exact paths

### After Database Setup
```bash
# Test connection with Prisma
npx prisma db push
npx prisma generate
```
✅ Pass: Migrations apply successfully
❌ Fail: Check DATABASE_URL and DIRECT_URL in .env

### Connection Pool Health Check
```bash
# Query Supabase to see active connections
SELECT count(*) FROM pg_stat_activity;

# Should be much lower than your connection_limit (typically 5-20 in production)
```

### After First Route
```bash
# Test with curl
curl -X POST http://localhost:3000/api/health \
  -H "Content-Type: application/json" \
  -d '{"test": true}'
```
✅ Pass: Returns `200 OK` with expected JSON
❌ Fail: Check the error message — see debugging loop below

### After Adding Inngest Job
```bash
# Check Inngest dashboard
# Navigate to https://app.inngest.com and verify:
# 1. Your function is registered
# 2. Recent runs show successful executions
# 3. No jobs are stuck in retry loops
```

### Debugging Loop
```
1. Read the full error message — do not skip it
2. Paste the EXACT error into your AI agent with this prompt:
   "I got this error: [PASTE ERROR].
    Here is the file it references: [PASTE FILE CONTENT].
    Fix ONLY this error. Do not change any other files."
3. Re-test the same endpoint
4. If still failing, ask: "What are the top 3 reasons this error occurs in [STACK]?"
5. Fix one reason at a time
```

### Common Errors and What They Mean

| Error | Plain-English Meaning | Fix |
|---|---|---|
| `Cannot find module` | A file your code imports does not exist | Check the import path — typo or missing file |
| `Connection timeout` | Cannot reach the database | Check DATABASE_URL, PgBouncer port, firewall |
| `too many connections` | Connection pool exhausted | Verify singleton pattern for Prisma, use PgBouncer |
| `404 Not Found` | The route URL does not match what you defined | Check your route file name and URL pattern |
| `500 Internal Server Error` | Your code crashed on the server | Check server logs for the real error |
| `CORS error` | Browser is blocking the request | Add CORS headers to your API — ask AI agent to add CORS middleware |
| `Environment variable undefined` | Your `.env` file is missing a key | Add the missing key to `.env.local` |

### Architecture Health Checks

Once you're live:

```
- Database connection count under load (alert if > 80% of pool)
- Background job queue depth (alert if > 100 pending jobs)
- Background job failure rate (alert if > 5% failure rate over 1 hour)
- API response times (alert if p95 > 3 seconds)
```

</testing_and_qa>

---

<common_patterns>

## Reusable Backend Patterns for AI Products

### Pattern 1: Standard API Route Structure (Next.js)
```typescript
// /app/api/[feature]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@clerk/nextjs/server";
import { z } from "zod";
import { db } from "@/lib/db";

const CreateSchema = z.object({
  title: z.string().min(1).max(500),
});

export async function POST(req: NextRequest) {
  try {
    // 1. Auth check first
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json({ success: false, error: "UNAUTHORIZED" }, { status: 401 });
    }

    // 2. Parse and validate input
    const body = await req.json();
    const parsed = CreateSchema.safeParse(body);
    if (!parsed.success) {
      return NextResponse.json({
        success: false,
        error: "VALIDATION_ERROR",
        message: parsed.error.errors[0].message,
      }, { status: 400 });
    }

    // 3. Business logic
    const result = await db.item.create({
      data: { userId, ...parsed.data },
    });

    // 4. Return success response
    return NextResponse.json({ success: true, data: result }, { status: 201 });

  } catch (error) {
    console.error("[POST /api/feature]", error);
    return NextResponse.json({ success: false, error: "INTERNAL_ERROR" }, { status: 500 });
  }
}
```

### Pattern 2: Event-Driven Handler Pattern
```typescript
// lib/jobs/user-created.ts
export const userCreatedJob = inngest.createFunction(
  { id: "user-created" },
  { event: "user/created" },
  async ({ event, step }) => {
    const { userId, email } = event.data

    // Each step is retryable independently
    await step.run("create-profile", async () => {
      await db.userProfile.create({
        data: { userId }
      })
    })

    await step.run("send-email", async () => {
      await sendWelcomeEmail(email)
    })

    return { success: true }
  }
)
```

### Pattern 3: Long-Running AI Task with Streaming
```typescript
// /app/api/generate/route.ts
export async function POST(req: NextRequest) {
  const { userId } = await auth();
  if (!userId) return NextResponse.json({ error: "UNAUTHORIZED" }, { status: 401 });

  const body = await req.json();

  // Return immediately with job ID, don't wait for AI
  const jobId = crypto.randomUUID();

  // Trigger background job
  await inngest.send({
    name: "content/generate",
    data: { jobId, userId, prompt: body.prompt }
  });

  return NextResponse.json({ success: true, jobId, status: "processing" }, { status: 202 });
}

// Background job does the real work
export const generateContentJob = inngest.createFunction(
  { id: "content-generate", retries: 2 },
  { event: "content/generate" },
  async ({ event, step }) => {
    const { jobId, userId, prompt } = event.data

    const content = await step.run("call-ai", async () => {
      return await callOpenAI(prompt)
    })

    await step.run("save-to-db", async () => {
      await db.content.create({
        data: { id: jobId, userId, content }
      })
    })
  }
)
```

### Pattern 4: Service-to-Service Call with Retry
```typescript
// lib/api/python-service.ts
export async function callMLService(endpoint: string, data: unknown) {
  const maxRetries = 3;
  let lastError: Error | null = null;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const res = await fetch(
        `${process.env.PYTHON_SERVICE_URL}${endpoint}`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "X-Service-Secret": process.env.SERVICE_SECRET!,
          },
          body: JSON.stringify(data),
          signal: AbortSignal.timeout(30000), // 30 second timeout
        }
      );

      if (!res.ok) {
        throw new Error(`Service error: ${res.status}`);
      }

      return await res.json();
    } catch (error) {
      lastError = error as Error;
      if (attempt < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 1000 * (attempt + 1))); // exponential backoff
      }
    }
  }

  throw new Error(`Failed after ${maxRetries} attempts: ${lastError?.message}`);
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE — Apply every single time, no exceptions -->

## Security Rules for Backend Architecture

### Rule 1: Never Hardcode Secrets
```typescript
// ❌ NEVER DO THIS
const apiKey = "sk-abc123realkey";

// ✅ ALWAYS DO THIS
const apiKey = process.env.OPENAI_API_KEY;
if (!apiKey) throw new Error("OPENAI_API_KEY is not set");
```

### Rule 2: Always Validate Inputs Before Processing
```typescript
// ❌ NEVER trust raw input
const { userId } = req.body;
await db.query(`SELECT * FROM users WHERE id = ${userId}`); // SQL injection risk

// ✅ ALWAYS validate and sanitize
import { z } from "zod";
const schema = z.object({ userId: z.string().uuid() });
const { userId } = schema.parse(req.body);
```

### Rule 3: Every Route Must Check Authentication (Unless Intentionally Public)
```typescript
// At the top of every protected route
const { userId } = await auth();
if (!userId) {
  return NextResponse.json({ success: false, error: "UNAUTHORIZED" }, { status: 401 });
}
```

### Rule 4: Rate Limit All AI Endpoints
AI calls are expensive. Without rate limiting, one user can bankrupt your API budget.
```
Ask your AI agent: "Add rate limiting to the [ROUTE] endpoint using Upstash Redis.
Limit to 10 requests per user per minute."
```

### Rule 5: Never Log Sensitive Data
```typescript
// ❌ Never log tokens, passwords, or PII
console.log("User data:", user); // could expose email, password hash

// ✅ Log only what you need for debugging
console.log("User action:", { userId: user.id, action: "login" });
```

### Rule 6: Database Credentials Must Use Least-Privilege Roles
Your app user should have no DROP/CREATE permissions. Only SELECT, INSERT, UPDATE, DELETE on your app's tables.

### Rule 7: Background Jobs Must Re-Verify Ownership Before Acting
A job may execute seconds or minutes after being triggered. Re-check that the user still owns the resource before operating on it.

```typescript
// ❌ Bad — assumes user still owns resource
await inngest.send({ name: "document/delete", data: { documentId, userId } });

// ✅ Good — re-verify ownership in the job
export const deleteDocumentJob = inngest.createFunction(
  { id: "document-delete" },
  { event: "document/delete" },
  async ({ event }) => {
    const { documentId, userId } = event.data;

    // Re-verify the user owns this document
    const doc = await db.document.findUnique({ where: { id: documentId } });
    if (!doc || doc.userId !== userId) {
      throw new Error("Access denied");
    }

    await db.document.delete({ where: { id: documentId } });
  }
);
```

### Environment Variables Checklist
Create a `.env.example` file (safe to commit) listing every key your app needs:
```
# AI
OPENAI_API_KEY=
ANTHROPIC_API_KEY=

# Database
DATABASE_URL=
DIRECT_URL=

# Auth
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=

# Background Jobs
INNGEST_EVENT_KEY=
INNGEST_SIGNING_KEY=

# Service-to-Service
PYTHON_SERVICE_URL=
SERVICE_SECRET=

# App
NEXT_PUBLIC_APP_URL=
```

</security_guardrails>

---

<mistakes_to_avoid>

## The Mistakes That Break Vibe-Coded Backends

### ❌ Mistake 1: Prisma Without Singleton Pattern
You create a new Prisma client on every serverless invocation. At 100 concurrent users, that's 100 independent Prisma instances each with up to 10 connections — 1,000 connections when your database only allows 100.

**Fix:** Always use the global singleton pattern. Copy the code from Phase 2 above.

### ❌ Mistake 2: Long-Running Work in API Routes
You're trying to process a 50MB document in a synchronous API route. Vercel serverless functions timeout at 10 seconds.

**Fix:** Return `202 Accepted` immediately, trigger an Inngest background job, let the client poll or use webhooks.

### ❌ Mistake 3: Using Direct Database URL Instead of PgBouncer for App Traffic
You're using port 5432 (direct) for your application. Each serverless invocation opens a new connection. Connection pool exhausted.

**Fix:** Use port 6543 with `pgbouncer=true&connection_limit=1` for app traffic. Reserve port 5432 (DIRECT_URL) for migrations only.

### ❌ Mistake 4: Premature Microservices
You have 2 features and you've split them into 5 services. Now you're debugging distributed tracing problems instead of building features.

**Fix:** Start monolith (Next.js). Split only when you have a concrete reason: different scaling, different team, different tech requirement.

### ❌ Mistake 5: Not Using step.run() in Inngest
Your background job doesn't have checkpoints. It crashes halfway through. When it retries, it starts from scratch and corrupts your data.

**Fix:** Wrap each major section of your job in `step.run()`. Inngest will resume from the last successful step.

### ❌ Mistake 6: Building All Routes at Once
You ask the AI to "build all the API routes" in one prompt. The AI generates 15 files with interconnected logic, half reference each other incorrectly, and now nothing works.

**Fix:** One route per session. Test before continuing.

### ❌ Mistake 7: No Folder Structure Before Logic
You start building features without defining where files live. The AI puts files in random locations across sessions.

**Fix:** Always do folder structure first. Always paste your context_anchor at session start.

### ❌ Mistake 8: Blocking API Response on Background Job Completion
You start a background job but then wait for it to finish before returning to the client. If the job takes 8 seconds, serverless timeouts.

**Fix:** Trigger the job, return `202 Accepted` immediately with the job ID, let the client poll or subscribe to webhooks.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Backend Architecture

### When You Hit 1,000 Users
- Add a caching layer (Redis / Upstash) in front of expensive AI calls and database reads
- Move background jobs to a proper queue (Inngest / Trigger.dev) instead of inline async functions
- Add database connection pooling (PgBouncer via Supabase) — critical at this scale
- Add request logging middleware to track performance

### When You Hit 10,000 Users
- Separate your AI orchestration into its own service (a dedicated FastAPI app)
- Add observability: structured logging (Pino), error tracking (Sentry), performance monitoring (Datadog or Axiom)
- Consider read replicas for your database — OLAP queries separate from OLTP
- Add monitoring alerts for connection pool usage, background job queue depth, AI provider errors

### When You Hit 100,000 Users
- Implement feature flags (LaunchDarkly) for safer deployments
- Add CDN for static assets and API responses (Vercel Edge, Cloudflare)
- Consider database sharding if a single table gets >100GB
- Move chat/streaming to a dedicated service with WebSocket support

### Production-Grade Additions (Add One at a Time)
```
1. "Add request logging middleware that captures: timestamp, route, status code, duration"
2. "Add Sentry error tracking to all API routes"
3. "Add a /api/health endpoint that checks DB connection and returns status"
4. "Add database query performance monitoring to identify slow queries"
5. "Add background job queue monitoring and alerts"
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: EdTech App — Lesson Generation Backend
**App:** AI generates personalised lessons based on student level
**Backend design decision:** Next.js App Router with Supabase + Inngest

```
Initial architecture:
- API Layer: 5 routes (auth, user profile, generate, retrieve, progress)
- Database: PostgreSQL via Supabase with PgBouncer
- Background Jobs: Lesson generation async (takes 5-15 seconds)
- Connection Pooling: PgBouncer port 6543 with connection_limit=1

Scaling event at 500 users:
- Noticed connection pool alerts from Supabase
- Verified singleton Prisma pattern was in place
- Problem found: not all API routes were using the singleton
- Fixed: created lib/db middleware, all routes import from there

Scaling event at 5,000 users:
- Lesson generation was causing AI API rate limits
- Split into two services:
  1. Next.js for user-facing routes
  2. FastAPI with model caching for batch lesson generation
```

### Case Study 2: Conversational Reminder App
**App:** Users set reminders via natural language chat
**Backend design decision:** Next.js App Router + Supabase + Inngest for scheduled reminders

```
Initial routes (MVP):
1. POST /api/chat                  → Parse user message, extract intent
2. POST /api/reminders/create      → Save reminder to DB, schedule Inngest job
3. GET  /api/reminders             → List all reminders for user
4. POST /api/reminders/[id]/snooze → Reschedule an existing reminder

Architecture decision at launch:
- Chat endpoint does TWO things: parses intent AND creates reminder in one call
- This was simpler than two separate calls at MVP stage
- Later split into two when complexity grew

Background job for reminder delivery:
- Inngest cron job runs every minute: "reminder/check-due"
- Job finds reminders scheduled for right now
- Sends push notification + email via separate Inngest jobs
- Retry logic: 3 retries with exponential backoff for failed notifications
```

</real_world_examples>

</skill_document>
