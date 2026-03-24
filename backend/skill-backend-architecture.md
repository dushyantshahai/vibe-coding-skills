# Skill: Backend Architecture

```json
{
  "skill_id": "backend-architecture",
  "category": "Architecture & Backend",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Backend Architecture — Designing a Scalable, Production-Ready Backend for AI Products</title>

<overview>

## What This Skill Enables
- Design a backend architecture that can handle real users, real data, and real AI workloads
- Make confident decisions about server structure before writing a single line of code
- Avoid the most common trap of vibe coding: building features that cannot scale or connect to each other

## Why It Matters for Vibe Coders
Most AI-powered apps fail not because of bad ideas but because the backend was never properly designed. Without architecture decisions made upfront, you end up with spaghetti code that breaks when you add a second feature. This skill gives you a blueprint to hand your AI coding agent so it builds in the right direction from day one.

## When to Use This Skill
- At the very beginning of any new project, before writing any code
- When restarting or refactoring a project that has grown messy
- When adding a major new feature that touches multiple parts of the app

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
    "current_backend_files": "[REPLACE: list key files e.g. /api/chat.ts, /lib/db.ts]",
    "features_already_built": "[REPLACE: e.g. user login, basic chat UI]",
    "features_to_build_next": "[REPLACE: e.g. lesson generation endpoint, reminder scheduler]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Backend Architecture

### Mental Model 1: The Kitchen Analogy
Think of your backend like a restaurant kitchen:
- **The menu (API routes)** — what your app can do for users
- **The chefs (serverless functions / controllers)** — logic that processes each request
- **The pantry (database)** — where all data lives
- **The tickets system (queues / schedulers)** — background jobs like sending reminders or generating AI content
- **The health inspector rules (security + validation)** — rules every dish must follow

A well-designed kitchen has clear stations. A poorly designed kitchen has chefs bumping into each other.

### Mental Model 2: Monolith vs Modular
- **Monolith** = One big kitchen that does everything. Simple to start, hard to scale.
- **Modular/Serverless** = Many small specialist kitchens. Slightly more setup, but each part can scale independently.

**For AI products starting out:** begin modular-friendly monolith (e.g. Next.js App Router or FastAPI with clear route groupings). Do not build microservices on day one — that is premature complexity.

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
└──────┬─────────────┬──────────────┬─────────────────┘
       │             │              │
┌──────▼──────┐ ┌────▼─────┐ ┌─────▼──────────────────┐
│  Auth       │ │  Business│ │  AI Orchestration Layer  │
│  Service    │ │  Logic   │ │  (Prompt → LLM → Parse) │
└──────┬──────┘ └────┬─────┘ └─────┬──────────────────┘
       │             │              │
┌──────▼─────────────▼──────────────▼───────────────────┐
│                    DATABASE LAYER                       │
│         (PostgreSQL / MongoDB / Vector DB)              │
└────────────────────────────────────────────────────────┘
       │
┌──────▼───────────────────┐
│  BACKGROUND JOBS / QUEUE  │
│  (Reminders, Emails, AI   │
│   batch processing)       │
└───────────────────────────┘
```

### Component Descriptions

| Component | What It Does | Recommended Tool |
|---|---|---|
| API Layer | Receives and routes all requests | Next.js App Router / FastAPI / Express |
| Auth Service | Handles login, sessions, tokens | Clerk / Supabase Auth / Auth0 |
| Business Logic | Core feature processing | Your own functions/services |
| AI Orchestration | Calls LLM, manages prompts, parses responses | LangChain / Vercel AI SDK / custom |
| Database Layer | Stores and retrieves all data | Supabase (PostgreSQL) / MongoDB Atlas |
| Background Jobs | Async tasks: emails, reminders, AI batch | Inngest / Trigger.dev / Upstash QStash |

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Complete and verify each step before moving to the next. -->

## Step 1 — Define Your App's Core Actions
Before touching code, list every action your app needs to perform.

**Prompt your AI agent:**
```
I am building [APP NAME]. List every backend action this app needs to perform. 
Format as: ACTION_NAME → input → output → which component handles it.
Do not write any code yet. Just map the actions.
```

**Verify:** You should have a list of 5–15 actions (e.g. `create_user`, `generate_lesson`, `send_reminder`). If you have more than 20, you are over-scoping — cut to MVP.

---

## Step 2 — Choose Your Stack
Use the decision table below, then confirm your choice with your AI agent.

| Situation | Recommended Stack |
|---|---|
| Building fast, deploying to Vercel | **Next.js App Router + Supabase** |
| Python-first, AI-heavy backend | **FastAPI + PostgreSQL + Railway** |
| Mobile app backend | **Supabase (BaaS) or Firebase** |
| Complex agent pipelines | **FastAPI + LangChain + Redis queue** |

**Prompt your AI agent:**
```
My project context: [PASTE context_anchor JSON]
Based on this context, confirm my stack choice is [YOUR STACK].
List any conflicts or missing pieces you can see before we start building.
Do not write any code yet.
```

**Verify:** AI confirms stack choice with no conflicts, or flags something you need to resolve first.

---

## Step 3 — Generate the Folder Structure
Only generate the skeleton — no logic yet.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Stack: [YOUR STACK]
Generate ONLY the folder structure for this backend. 
Include empty placeholder files with one-line comments describing what each file will do.
Do not write any business logic yet.
```

**Verify:** Run `ls -R` or open the folder in your IDE. Every folder and file should exist with only comments inside — no real code yet.

---

## Step 4 — Build One Route End-to-End
Pick the simplest, most critical route (usually health check or user creation). Build it completely before touching any other route.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Build ONLY the [ROUTE NAME] endpoint. 
Include: route handler, input validation, one database query, response format.
Do not touch any other files. Show me what to test after this is done.
```

**Verify:** Use a tool like Postman, Insomnia, or `curl` to hit the endpoint. It must return the expected response before you proceed.

---

## Step 5 — Add the AI Orchestration Layer
Only after Step 4 is working.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
The [ROUTE NAME] endpoint is working. 
Now add the AI call to this endpoint ONLY.
Use [AI PROVIDER] with this prompt pattern: [DESCRIBE WHAT THE AI SHOULD DO].
Handle: loading state, error from AI provider, empty response.
Do not add any new routes.
```

**Verify:** Test the endpoint with a real AI call. Check the response format is what you expect.

---

## Step 6 — Repeat for Each Remaining Route
One route at a time. Test each before building the next.

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
2. Any security risks in the current structure
3. The single most important thing to build next

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

### "What is a serverless function and do I need it?"

Serverless = your code only runs when someone calls it. You do not manage a server that runs 24/7.

**Why it matters for you:** It is cheaper at low traffic, scales automatically, and works perfectly with Vercel and Supabase. Most vibe-coded AI apps should use serverless functions.

**When you need a real server instead:** If your app has long-running tasks (>30 seconds) like video processing, batch AI generation, or WebSocket connections that stay open. In that case, look at Railway or Render for always-on servers.

---

### "How many API routes is too many?"

If your MVP has more than 15 routes, you are probably over-building. Cut to the routes that directly serve your core user journey. Everything else is a nice-to-have for v2.

</vibe_coder_bridge>

---

<testing_and_qa>

## How to Verify Your Backend Architecture is Working

### After Step 3 (Folder Structure)
```bash
# Run this in your terminal to confirm all files exist
ls -R ./app/api    # Next.js
# or
ls -R ./routers    # FastAPI
```
✅ Pass: All expected folders and files are present  
❌ Fail: Re-run the folder structure prompt and specify exact paths

### After Step 4 (First Route)
```bash
# Test with curl (replace URL and data)
curl -X POST http://localhost:3000/api/health \
  -H "Content-Type: application/json" \
  -d '{"test": true}'
```
✅ Pass: Returns `200 OK` with expected JSON  
❌ Fail: Check the error message — see debugging loop below

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
| `404 Not Found` | The route URL does not match what you defined | Check your route file name and URL pattern |
| `500 Internal Server Error` | Your code crashed on the server | Check server logs for the real error |
| `CORS error` | Browser is blocking the request | Add CORS headers to your API — ask AI agent to add CORS middleware |
| `Environment variable undefined` | Your `.env` file is missing a key | Add the missing key to `.env.local` |

</testing_and_qa>

---

<common_patterns>

## Reusable Backend Patterns for AI Products

### Pattern 1: Standard API Route Structure (Next.js)
```typescript
// /app/api/[feature]/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  try {
    // 1. Parse and validate input
    const body = await req.json();
    if (!body.requiredField) {
      return NextResponse.json({ error: "requiredField is missing" }, { status: 400 });
    }

    // 2. Business logic here

    // 3. Return success response
    return NextResponse.json({ success: true, data: result }, { status: 200 });

  } catch (error) {
    console.error("[FEATURE_NAME] error:", error);
    return NextResponse.json({ error: "Internal server error" }, { status: 500 });
  }
}
```

### Pattern 2: Standard API Route Structure (FastAPI)
```python
# /routers/feature.py
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter()

class FeatureRequest(BaseModel):
    required_field: str

@router.post("/feature")
async def handle_feature(request: FeatureRequest):
    try:
        # Business logic here
        return {"success": True, "data": result}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Pattern 3: Standard AI Call Wrapper
```typescript
// /lib/ai.ts — reuse this across all AI calls
export async function callAI(prompt: string, systemPrompt: string) {
  const response = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "gpt-4o",
      messages: [
        { role: "system", content: systemPrompt },
        { role: "user", content: prompt }
      ],
      max_tokens: 1000,
    }),
  });
  
  if (!response.ok) throw new Error(`AI call failed: ${response.statusText}`);
  
  const data = await response.json();
  return data.choices[0].message.content;
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
const session = await getSession(req);
if (!session) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
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

### Environment Variables Checklist
Create a `.env.example` file (safe to commit) listing every key your app needs:
```
# AI
OPENAI_API_KEY=
ANTHROPIC_API_KEY=

# Database
DATABASE_URL=

# Auth
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=

# App
NEXT_PUBLIC_APP_URL=
```

</security_guardrails>

---

<mistakes_to_avoid>

## The Mistakes That Break Vibe-Coded Backends

### ❌ Mistake 1: Building All Routes at Once
You ask the AI to "build all the API routes" in one prompt. The AI generates 15 files with interconnected logic, half of which reference each other incorrectly, and now nothing works and you do not know where the problem is.

**Fix:** One route per session. Test before continuing.

### ❌ Mistake 2: No Folder Structure Before Logic
You start building features without defining where files live. The AI puts files in random locations across sessions because it has no memory of where it put things last time.

**Fix:** Always do Step 3 (folder structure) first. Always paste your context_anchor at session start.

### ❌ Mistake 3: Skipping Error Handling
You ask the AI to build a feature and it works in the happy path (everything goes right). But when the AI API is slow, or the database is empty, or the user sends bad data — the app crashes with no explanation.

**Fix:** Always include "handle all error states" in your build prompts.

### ❌ Mistake 4: Using Production Database During Development
You connect your real database during testing. You accidentally delete real user data or corrupt your schema.

**Fix:** Always use a separate development database or a branch. Supabase and PlanetScale both support this.

### ❌ Mistake 5: Not Pinning Your AI Model Version
```typescript
// ❌ Risky — model behaviour can change
model: "gpt-4"

// ✅ Safe — predictable behaviour
model: "gpt-4o-2024-08-06"
```

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Backend Architecture

### When You Hit 1,000 Users
- Add a caching layer (Redis / Upstash) in front of expensive AI calls and database reads
- Move background jobs to a proper queue (Inngest / Trigger.dev) instead of inline async functions
- Add database connection pooling (PgBouncer via Supabase)

### When You Hit 10,000 Users
- Separate your AI orchestration into its own service (a dedicated FastAPI app)
- Add observability: structured logging (Pino), error tracking (Sentry), performance monitoring (Datadog or Axiom)
- Consider read replicas for your database

### Production-Grade Additions
```
Ask your AI agent to add these one at a time, after core features are working:
1. "Add request logging middleware that captures: timestamp, route, status code, duration"
2. "Add Sentry error tracking to all API routes"
3. "Add a /api/health endpoint that checks DB connection and returns status"
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: EdTech App — Lesson Generation Backend
**App:** AI generates personalised lessons based on student level  
**Backend design decision:** Next.js App Router with Supabase

```
Routes built in order:
1. POST /api/auth/callback         → Supabase auth redirect handler
2. GET  /api/user/profile          → Fetch user learning level
3. POST /api/lessons/generate      → Call AI with level-specific prompt, save to DB
4. GET  /api/lessons/[id]          → Retrieve a saved lesson
5. POST /api/lessons/[id]/progress → Mark lesson section as complete
```

**Key architecture decision:** Lesson generation is async (can take 5–10s). Used Vercel streaming response so the UI shows content as it generates, rather than waiting for the full response.

---

### Case Study 2: Conversational Reminder App
**App:** Users set reminders via natural language chat  
**Backend design decision:** Next.js + Supabase + Inngest for scheduled reminders

```
Routes built in order:
1. POST /api/chat                  → Parse user message, extract intent
2. POST /api/reminders/create      → Save reminder to DB, schedule Inngest job
3. GET  /api/reminders             → List all reminders for user
4. POST /api/reminders/[id]/snooze → Reschedule an existing reminder
```

**Key architecture decision:** The chat endpoint does two things — understands the intent AND creates the reminder in one call. This was simpler than two separate calls at MVP stage, and was later split into two when complexity grew.

</real_world_examples>

</skill_document>
