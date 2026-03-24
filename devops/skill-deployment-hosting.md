# Skill: Deployment & Hosting

```json
{
  "skill_id": "deployment-hosting",
  "category": "Deployment & DevOps",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Deployment & Hosting — Getting Your AI Product Live, Reliably</title>

<overview>

## What This Skill Enables
- Deploy your app to a live URL that real users can access — in under 30 minutes
- Set up preview deployments so every pull request gets its own testable URL before going live
- Understand the difference between hosting platforms and choose the right one for your AI product's needs

## Why It Matters for Vibe Coders
"It works on my laptop" is the most common vibe coder failure mode. Deployment is where local assumptions collide with production reality: missing environment variables, wrong Node versions, serverless function timeouts, and build errors that never appeared locally. This skill gives you a deployment checklist and platform decision guide so your first deploy succeeds — and every subsequent deploy is safe.

## When to Use This Skill
- The first time you deploy a project to production
- When switching hosting providers
- When a deploy broke something that worked locally
- When adding a new service (database, background jobs, AI provider) that requires production configuration

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI / Express]",
    "hosting_platform": "[REPLACE: e.g. Vercel / Railway / Render / not chosen yet]",
    "database_provider": "[REPLACE: e.g. Supabase / PlanetScale / Railway Postgres]",
    "background_jobs": "[REPLACE: e.g. Inngest / Trigger.dev / none]",
    "current_deploy_status": "[REPLACE: e.g. not yet deployed / deployed but broken / works in staging only]",
    "custom_domain": "[REPLACE: e.g. myapp.com / not yet / using platform subdomain]",
    "deploy_task_today": "[REPLACE: e.g. first production deploy / fix failing build / add staging environment]",
    "known_env_vars_needed": "[REPLACE: list env var names needed — e.g. DATABASE_URL, OPENAI_API_KEY, CLERK_SECRET_KEY]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Deployment

### Mental Model 1: Environments Are Separate Worlds
Every app should have at least two environments — ideally three:

```
LOCAL (your laptop)
  ↓ push to branch
PREVIEW (auto-generated per PR)
  ↓ merge to main
PRODUCTION (what users see)
```

Each environment has its own database, its own API keys, its own URL. They share the same codebase but nothing else. Treating them as separate worlds prevents the most common disaster: a developer accidentally running destructive tests against real user data.

### Mental Model 2: Build ≠ Deploy
Two separate steps happen when you push to production:

```
BUILD → Your source code is compiled/bundled into deployable artifacts
         - TypeScript is compiled
         - Next.js pages are rendered or prepared
         - Dependencies are installed
         - If this fails: no deploy happens, your last working version stays live

DEPLOY → The build artifact is placed on servers and made accessible
          - Old version swapped for new version
          - Environment variables injected
          - Health checks run
          - If this fails: rollback to previous version
```

A failed build is safe — nothing changes for users. A failed deploy after a bad build going live is dangerous — this is why health checks matter.

### Mental Model 3: Serverless vs Always-On
| | Serverless | Always-On Server |
|---|---|---|
| **What it means** | Code runs on demand, sleeps when idle | Server runs 24/7 |
| **Examples** | Vercel Functions, AWS Lambda | Railway, Render, Fly.io |
| **Best for** | API routes, most AI endpoints | WebSockets, long-running jobs, scheduled tasks |
| **Cost** | Pay per request (very cheap at low scale) | Pay per hour (predictable) |
| **Timeout limit** | 10–60s (platform dependent) | No limit |
| **Cold start** | Yes (first request after idle is slower) | No |

**For most AI products:** Use Vercel (serverless) for the web app and API routes. Use Railway or Render for any long-running background service or Python FastAPI backend.

</mental_models>

---

<system_design_breakdown>

## Platform Decision Guide

```
Are you building a Next.js app?
    │
    YES ──→ Use VERCEL
    │         - Zero config for Next.js
    │         - Preview deployments on every PR
    │         - Edge runtime for streaming AI
    │         - Free tier generous for early products
    │
    NO
    │
    ├─→ Python/FastAPI backend?
    │         Use RAILWAY
    │         - Easiest Python deploy
    │         - Built-in PostgreSQL
    │         - Scales to zero when idle
    │
    ├─→ Need Docker?
    │         Use RENDER or FLY.IO
    │         - Full Docker support
    │         - More control over runtime
    │
    └─→ Need background job workers?
              Use RAILWAY (dedicated worker service)
              or pair Vercel + Inngest (managed job queue)
```

## Vercel Deployment Architecture (Most Common for AI Products)

```
GitHub Repo (main branch)
         │
         │ push
         ▼
    Vercel Build
    ├── next build
    ├── Type check
    └── Static generation
         │
         │ success
         ▼
   Vercel Deploy
   ├── Edge Network (global CDN)
   ├── Serverless Functions (/app/api/*)
   └── Static Assets (/_next/static/*)
         │
         ▼
   Production URL: myapp.com
   Preview URL: myapp-git-branch-org.vercel.app
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Get a basic deploy working first. Add complexity (custom domain, staging, etc.) one step at a time. -->

## Step 1 — Pre-Deploy Checklist
Run through this before your first deploy attempt.

```
□ npm run build runs locally without errors
□ All environment variables are listed in .env.example
□ .env and .env.local are in .gitignore (never committed)
□ No hardcoded localhost URLs in API calls (use relative paths: /api/... not http://localhost:3000/api/...)
□ No hardcoded secrets anywhere in the codebase (grep for API keys)
□ Database is a hosted service (Supabase, PlanetScale) — not localhost
□ package.json has a "build" script
□ If Next.js: next.config.ts has no local-only settings that break in production
```

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Run a pre-deploy audit on my codebase.
Check for:
1. Any hardcoded localhost URLs (should use relative paths or env vars)
2. Any hardcoded API keys or secrets outside of env vars
3. Any imports that reference files that do not exist
4. Any console.log statements that leak sensitive data
5. Confirm build script exists in package.json

Return a checklist of issues found. Do not fix anything yet.
```

---

## Step 2 — Deploy to Vercel (First Deploy)

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
I am deploying to Vercel for the first time.

Give me the exact steps to:
1. Connect my GitHub repo to Vercel (via vercel.com UI — describe the steps)
2. Configure the build settings for [FRAMEWORK]
3. Set these environment variables in Vercel dashboard: [LIST YOUR ENV VARS — names only]
4. Trigger the first deploy
5. Verify the deploy succeeded using Vercel's function logs

Also generate a vercel.json file if any of these are needed:
- Custom build command
- Function timeout overrides (for streaming AI routes — needs maxDuration: 60)
- Redirect rules
- Header configuration

Do not include any secret values in the vercel.json — only configuration.
```

**Verify:** Open the Vercel deployment URL. Core functionality works. Check Vercel dashboard → Functions for any runtime errors.

---

## Step 3 — Configure Function Timeouts for AI Routes
Default serverless timeout (10s) kills most AI generation routes.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create/update vercel.json to set extended timeouts for AI-heavy routes.
Routes that need extended timeouts: [LIST — e.g. /api/lessons/generate, /api/chat]

Set:
- Streaming routes (edge runtime): maxDuration: 60 (60 seconds)
- Non-streaming AI routes: maxDuration: 30
- All other routes: default (10 seconds)

Also confirm: all streaming routes have export const runtime = "edge" in the route file.
```

**Verify:** Test your AI endpoint with a prompt that takes 15+ seconds. Confirm it completes without timeout.

---

## Step 4 — Set Up Preview Deployments
Automatic — every PR gets its own live URL.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Configure Vercel preview deployments to:
1. Use a separate preview database (not the production Supabase project)
   - Create a new Supabase project named [APP_NAME]-preview
   - Add DATABASE_URL_PREVIEW to Vercel as an environment variable
   - In vercel.json: use DATABASE_URL_PREVIEW for preview environments
2. Use test/sandbox API keys for Stripe, not live keys
3. Show a "PREVIEW" banner in the UI when running in preview environment
   - Check: process.env.VERCEL_ENV === "preview"

Show me the Vercel environment variable configuration to make this work.
```

**Verify:** Open a PR. Wait for Vercel to comment with a preview URL. Open it. Confirm the "PREVIEW" banner shows.

---

## Step 5 — Add a Custom Domain

**Prompt your AI agent:**
```
I want to connect my domain [DOMAIN.COM] to my Vercel deployment.

Give me the step-by-step:
1. DNS records to add at my domain registrar (A record or CNAME — which one for Vercel?)
2. How to add the domain in Vercel dashboard
3. How to verify SSL certificate was issued (Vercel handles this automatically)
4. How to set up www redirect (www.domain.com → domain.com)
5. How to update any hardcoded URLs in my app that used the vercel.app subdomain

Expected DNS propagation time: up to 48 hours (usually under 1 hour).
```

**Verify:** After DNS propagation, open your custom domain. SSL lock icon shows in browser. Both www and non-www redirect correctly.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on deployment for my app.
Project context: [PASTE context_anchor JSON]

Current deploy status: [DESCRIBE — e.g. "first deploy attempt" / "build failing" / "deployed but route X is broken"]
Platform: [VERCEL / RAILWAY / RENDER]

Before doing anything:
1. Tell me the most likely cause of the current issue
2. Tell me exactly which file or config you will change
3. Confirm you will not modify any application logic — only deployment config
```

### Build Error Debug Prompt
```
My Vercel/Railway build is failing with this error:
[PASTE FULL BUILD ERROR LOG]

Project context: [PASTE context_anchor JSON]

Identify:
1. The exact line causing the build failure
2. Whether this is a: TypeScript error / missing dependency / missing env var / config error
3. The minimal fix

Important: this error does NOT appear when I run npm run build locally.
Check for differences between local and production environments that could cause this.
```

### Railway Deploy Prompt (Python/FastAPI)
```
Deploy my FastAPI app to Railway.
Project context: [PASTE context_anchor JSON]

Generate:
1. Procfile or railway.toml with the correct start command
2. requirements.txt or pyproject.toml with all dependencies
3. Health check endpoint: GET /health → { status: "ok" }
4. The environment variables I need to set in Railway dashboard
5. How to connect Railway's built-in PostgreSQL to my app

My app entry point: [e.g. main.py with app = FastAPI()]
```

### Rollback Prompt
```
My latest Vercel deploy broke production.
Users are reporting: [DESCRIBE THE ISSUE]
The deploy that broke it: [DESCRIBE WHAT CHANGED]

Tell me:
1. How to instantly rollback to the previous working deploy in Vercel dashboard
2. How to identify which change caused the break from the deploy diff
3. What to fix before re-deploying
4. How to test the fix in a preview environment before deploying to production again

Do not touch any code yet. Just give me the rollback and investigation plan.
```

### vercel.json Configuration Prompt
```
Generate a production-ready vercel.json for my Next.js app.
Project context: [PASTE context_anchor JSON]

Include:
1. Function timeout overrides: [LIST AI ROUTES] → 60s, all others → default
2. Security headers: X-Frame-Options, X-Content-Type-Options, Referrer-Policy
3. Redirect: www.myapp.com → myapp.com (if using custom domain)
4. Cache headers: static assets cached for 1 year, API routes not cached

Do not include any secrets or environment variable values.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Vercel vs Railway — which should I use?"

**Use Vercel if:**
- Your app is built with Next.js (Vercel made Next.js — zero-config deploy)
- You want preview deployments on every PR with zero setup
- You want global edge CDN with no configuration
- You are comfortable with serverless (no persistent server)

**Use Railway if:**
- Your backend is Python/FastAPI
- You need a persistent server (WebSockets, long-running processes)
- You want your database and app in one platform with internal networking
- You need Docker support

**Use Both (common pattern for AI products):**
- Vercel for the Next.js frontend and API routes
- Railway for a Python AI service or background worker that needs more than 60 seconds

---

### "What is a preview deployment and why do I need it?"

Every time you push a branch to GitHub, Vercel automatically builds and deploys it to a unique temporary URL (e.g. `myapp-git-feat-reminders-org.vercel.app`).

This means:
- You can share a working version of a new feature with someone before merging
- You can test on a real hosted environment (not your laptop) before going live
- You can run your integration tests against a real deployed URL
- You catch "works on my machine but not in production" issues before users do

This is enabled by default on Vercel — no extra setup. Connect your GitHub repo and it works immediately.

---

### "My build works locally but fails on Vercel. Why?"

Five most common causes in order of frequency:

1. **Missing environment variable** — you have it in `.env.local` but did not add it to Vercel's dashboard. Fix: add all env vars to Vercel → Settings → Environment Variables.

2. **Case-sensitive file imports** — macOS (your laptop) is case-insensitive. Linux (Vercel's servers) is case-sensitive. `import Component from './myComponent'` fails if the file is named `MyComponent.tsx`. Fix: match import case exactly.

3. **TypeScript error in a file not usually run locally** — a type error in a rarely-touched file that `tsc --noEmit` would catch. Fix: run `npm run build` locally — not just `npm run dev`.

4. **Node.js version mismatch** — your laptop runs Node 20, Vercel defaults to an older version. Fix: add `"engines": { "node": "20.x" }` to `package.json`.

5. **Dynamic imports with server-only modules in client components** — a module that only works on the server gets imported somewhere it runs on the client. Fix: add `"use server"` directive or move the import.

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Deployment

### Post-Deploy Verification Checklist
```
□ App loads at the production URL (no blank page, no 500 error)
□ User can log in (auth flow works)
□ Core feature works end-to-end (create/read the main entity)
□ AI endpoint responds without timeout
□ Database is connected (check Vercel function logs for DB errors)
□ No console errors in browser DevTools
□ SSL certificate is valid (padlock icon in browser)
□ www redirect works if configured
□ /api/health returns { status: "ok" }
```

### Smoke Test Script (Run After Every Deploy)
```bash
#!/bin/bash
# smoke-test.sh — run after every production deploy

BASE_URL="https://yourapp.com"

echo "Testing health endpoint..."
HEALTH=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL/api/health")
[ "$HEALTH" = "200" ] && echo "✅ Health: OK" || echo "❌ Health: FAILED ($HEALTH)"

echo "Testing homepage..."
HOME=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL")
[ "$HOME" = "200" ] && echo "✅ Homepage: OK" || echo "❌ Homepage: FAILED ($HOME)"

echo "Testing API (unauthenticated)..."
API=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL/api/reminders")
[ "$API" = "401" ] && echo "✅ API auth: OK (correctly returns 401)" || echo "❌ API auth: FAILED ($API)"
```

### Common Deployment Errors

| Error | Platform | Meaning | Fix |
|---|---|---|---|
| `Build failed: Cannot find module` | Vercel | Import path wrong or package not in dependencies | Check import; add to `dependencies` (not `devDependencies`) in package.json |
| `Function timeout` | Vercel | Route took >10s | Add `maxDuration: 60` in vercel.json for that route |
| `500 on all API routes` | Any | Missing env var in production | Check Vercel/Railway environment variables dashboard |
| `Application failed to respond` | Railway | App crashed on startup | Check Railway deploy logs → find the startup error |
| `ERR_SSL_PROTOCOL_ERROR` | Any | SSL certificate not issued yet | Wait 10–30 minutes; verify DNS records are correct |
| `CORS error in production` | Any | CORS not configured for production domain | Update CORS allowed origins to include your custom domain |

### Debugging a Production-Only Bug
```
Production bug that does not reproduce locally →
  1. Check Vercel/Railway function logs immediately (before they expire)
     Vercel: dashboard → your project → Functions → click the failing route
  2. Check: does the bug affect all users or specific users?
     All users: likely missing env var or build issue
     Specific users: likely data-dependent or auth-level issue
  3. Add structured logging to the suspect route (deploy the logging change first)
  4. Reproduce on a preview deployment using production-like data
  5. Fix on a branch → preview deploy → verify → merge → production deploy
```

</testing_and_qa>

---

<common_patterns>

## Reusable Deployment Patterns

### Pattern 1: vercel.json for AI Products
```json
{
  "functions": {
    "app/api/chat/route.ts": {
      "maxDuration": 60
    },
    "app/api/lessons/generate/route.ts": {
      "maxDuration": 60
    },
    "app/api/reminders/*/route.ts": {
      "maxDuration": 30
    }
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" }
      ]
    },
    {
      "source": "/_next/static/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/",
      "has": [{ "type": "host", "value": "www.myapp.com" }],
      "destination": "https://myapp.com",
      "permanent": true
    }
  ]
}
```

### Pattern 2: railway.toml for FastAPI
```toml
# railway.toml
[build]
builder = "NIXPACKS"
buildCommand = "pip install -r requirements.txt"

[deploy]
startCommand = "uvicorn main:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
healthcheckTimeout = 30
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

### Pattern 3: Environment-Aware Configuration
```typescript
// /lib/config.ts — centralise all environment-dependent config
export const config = {
  isProduction: process.env.NODE_ENV === "production",
  isPreview: process.env.VERCEL_ENV === "preview",
  isDevelopment: process.env.NODE_ENV === "development",
  appUrl: process.env.NEXT_PUBLIC_APP_URL ?? "http://localhost:3000",
  databaseUrl: process.env.DATABASE_URL!,
  openaiApiKey: process.env.OPENAI_API_KEY!,
};

// Validate all required env vars at startup
const required = ["DATABASE_URL", "OPENAI_API_KEY", "CLERK_SECRET_KEY"];
for (const key of required) {
  if (!process.env[key]) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
}
```

### Pattern 4: Preview Environment Banner
```typescript
// /components/layout/preview-banner.tsx
export function PreviewBanner() {
  if (process.env.NEXT_PUBLIC_VERCEL_ENV !== "preview") return null;
  return (
    <div className="bg-yellow-400 text-yellow-900 text-center text-sm py-1 font-medium">
      ⚠️ Preview environment — data will not be saved to production
    </div>
  );
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Commit .env Files
```bash
# .gitignore — verify these are present
.env
.env.local
.env.production
.env.*.local
```
If you ever accidentally commit a `.env` file: rotate all secrets immediately, then remove the file from git history using `git filter-branch` or BFG Repo Cleaner.

### Rule 2: Use Different Secrets Per Environment
```
Development:  STRIPE_SECRET_KEY=sk_test_...    (test mode — can't charge real cards)
Preview:      STRIPE_SECRET_KEY=sk_test_...    (test mode)
Production:   STRIPE_SECRET_KEY=sk_live_...    (live mode — real money)

Never use live keys in development or preview environments.
```

### Rule 3: Restrict Production Environment Variable Access
In Vercel: Settings → Environment Variables → set each production secret to "Production only." Team members can see preview/development values but not production secrets.

### Rule 4: Enable Vercel Deployment Protection
```
Vercel → Project Settings → Deployment Protection
- Enable: "Only GitHub users with access to this repository can trigger deployments"
- Enable: "Password protect preview deployments" (prevents external access to WIP features)
```

### Rule 5: Add Security Headers to Every Response
The `vercel.json` Pattern 1 above includes the critical security headers. Never deploy without these — they prevent clickjacking, MIME-type sniffing, and other browser-level attacks.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Deploying Without Running the Build Locally First
```bash
# Always run this before pushing to a deploy branch
npm run build
# If it fails locally: fix it locally (fast). Don't fix it via deploy-fail-iterate (slow).
```

### ❌ Using the Same Database for Development and Production
Every `db.drop()`, every test run, every bad migration you run locally hits production data.  
**Fix:** Three separate databases — local (Docker or Supabase CLI), preview (Supabase free project), production (Supabase paid project with backups).

### ❌ Not Setting Function Timeouts for AI Routes
Default 10s timeout kills streaming routes and complex AI workflows. This is a deployment config issue, not a code issue, and it only appears in production.  
**Fix:** Always add `vercel.json` with `maxDuration` before deploying AI routes.

### ❌ Forgetting to Add New Env Vars to All Environments
You add `RESEND_API_KEY` to your `.env.local` and the feature works locally. You deploy. Production is missing the variable. Email sends fail silently.  
**Fix:** Maintain a `.env.example` file. Add every new variable there. Before deploying: compare `.env.example` against your Vercel environment variable list.

### ❌ No Rollback Plan
A bad deploy goes live. You spend 30 minutes debugging instead of rolling back in 30 seconds.  
**Fix:** Know your rollback command before every production deploy:
- **Vercel:** Dashboard → Deployments → click any previous deploy → "Promote to Production"
- **Railway:** Dashboard → Deployments → click previous deploy → "Redeploy"

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Deployment

### Add Database Migrations to Deploy Pipeline
Never manually run migrations in production. Automate them:
```bash
# Add to package.json build script (runs before next build on Vercel):
"build": "prisma migrate deploy && next build"
# This ensures DB schema is updated before the new code that depends on it goes live
```

### Add Blue-Green Deployments
Zero-downtime deploys where the old version keeps serving traffic until the new version passes health checks. Vercel does this automatically — no extra config needed.

### Add Deploy Notifications to Slack
```
Vercel → Project Settings → Integrations → Slack
Configure: notify on production deploy success AND failure
This gives you instant awareness of every production change and any deploy failures
```

### Add a Staging Environment
Between preview and production:
```
Branch strategy:
  feature-branch → preview (auto, per PR)
  develop branch → staging (same spec as production, real data shape)
  main branch    → production

Staging uses:
  - Production-tier database (separate project, same schema)
  - Live API keys (Stripe test mode, real other services)
  - Same Vercel build settings as production
  - Password protected (not publicly accessible)
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — First Deploy Failures and Fixes
```
Attempt 1: Build failed
  Error: "Cannot find module '@/lib/ai'"
  Cause: File was named 'AI.ts' locally (macOS ignores case),
         Vercel (Linux) could not find it as 'ai.ts'
  Fix: Renamed file to lowercase 'ai.ts', updated import

Attempt 2: Build succeeded, app crashed on load
  Error: "Missing required environment variable: CLERK_SECRET_KEY"
  Cause: Added Clerk after initial Vercel connection — forgot to add to dashboard
  Fix: Added all missing env vars to Vercel → Environment Variables

Attempt 3: App loaded, AI endpoint timed out
  Error: Function timeout after 10000ms
  Cause: No vercel.json, default 10s timeout, AI route takes 15s
  Fix: Added vercel.json with maxDuration: 60 for /api/chat

Attempt 4: Everything worked ✅
Total time: 45 minutes including all failures and fixes
```

### Case Study 2: EdTech App — Staging Environment That Caught a Data Migration Bug
```
Situation: Adding a new 'difficulty' column to the lessons table.
Migration: ALTER TABLE lessons ADD COLUMN difficulty TEXT DEFAULT 'beginner'

Without staging: would have run migration directly on production.
With staging:
  1. Ran migration on staging database
  2. Deployed new code to staging
  3. Discovered: 3 existing API routes that queried lessons broke because
     they were now returning the new 'difficulty' field but the
     TypeScript types did not include it, causing a runtime error
  4. Fixed the types in a follow-up commit
  5. Re-tested on staging — all clear
  6. Deployed to production with confidence

The bug would have caused a 500 error on all lesson-related routes for
~20 minutes in production while the fix was deployed.
Staging caught it in 5 minutes with zero user impact.
```

</real_world_examples>

</skill_document>
