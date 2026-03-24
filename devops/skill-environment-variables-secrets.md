# Skill: Environment Variables & Secrets Management

```json
{
  "skill_id": "environment-variables-secrets",
  "category": "Deployment & DevOps",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Environment Variables & Secrets — Managing Config Across Dev, Preview, and Production</title>

<overview>

## What This Skill Enables
- Manage API keys, database URLs, and service credentials safely across every environment your app runs in
- Never accidentally expose a secret in your code, logs, or GitHub repository
- Set up a single source of truth for your app's configuration that your AI agent can reference reliably

## Why It Matters for Vibe Coders
Environment variables are the #1 source of deploy failures and security incidents in vibe-coded apps. Developers hardcode API keys during prototyping, forget to add them to production, or accidentally commit them to GitHub. This skill gives you a system: one `.env.example` file as the source of truth, a clear naming convention, and a verification step that catches missing variables at startup rather than silently at runtime.

## When to Use This Skill
- When setting up a new project for the first time
- When adding a new third-party service that requires an API key
- When a deploy fails with "environment variable undefined" errors
- When rotating compromised or expired secrets
- When onboarding a new developer or AI agent to your project

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
    "environments": "[REPLACE: e.g. local + preview + production / local + production only]",
    "services_in_use": [
      "[REPLACE: e.g. Supabase — database + auth]",
      "[REPLACE: e.g. OpenAI — lesson generation]",
      "[REPLACE: e.g. Clerk — authentication]",
      "[REPLACE: e.g. Stripe — subscriptions]",
      "[REPLACE: e.g. Resend — email]"
    ],
    "env_task_today": "[REPLACE: e.g. set up initial .env.example / add new service / rotate leaked key]",
    "known_missing_variable": "[REPLACE: env var name that is causing a current issue, or 'none']"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Environment Variables

### Mental Model 1: The Four Categories of Config
Every value in your app's configuration falls into one of four categories:

```
CATEGORY 1: Public runtime config
  Safe to expose in browser. Prefix: NEXT_PUBLIC_ (Next.js)
  Examples: NEXT_PUBLIC_APP_URL, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
  Where: .env.local + Vercel Environment Variables (all environments)

CATEGORY 2: Private server-only secrets
  Never exposed to browser. No prefix.
  Examples: OPENAI_API_KEY, SUPABASE_SERVICE_ROLE_KEY, CLERK_SECRET_KEY, STRIPE_SECRET_KEY
  Where: .env.local (local only, never committed) + Vercel Environment Variables (server-side)

CATEGORY 3: Build-time config
  Used during build, not at runtime.
  Examples: ANALYZE (bundle analysis), NEXT_TELEMETRY_DISABLED
  Where: CI environment + Vercel Build Environment Variables

CATEGORY 4: Infrastructure secrets
  Used by deployment infra, not your app code.
  Examples: VERCEL_TOKEN, RAILWAY_TOKEN
  Where: GitHub Actions Secrets only — never in app code
```

### Mental Model 2: Environment Inheritance
```
.env                  ← Defaults for all environments (safe to commit if no secrets)
.env.local            ← Local overrides (NEVER committed — in .gitignore)
.env.development      ← Development-specific values (safe to commit if no secrets)
.env.production       ← Production-specific values (safe to commit if no secrets)
.env.test             ← Test-specific values (safe to commit if no secrets)

Loading order (Next.js, last wins):
.env → .env.[environment] → .env.local → .env.[environment].local
```

**Simple rule for most vibe coders:** Only use `.env.local` for all local secrets. Let the hosting platform (Vercel/Railway) inject everything else in production.

### Mental Model 3: The Two Failure Modes
```
FAILURE MODE 1: Secret is missing in production
  Symptom: App works locally, crashes in production
  Cause: Added to .env.local but forgot to add to Vercel/Railway dashboard
  Prevention: .env.example as source of truth + startup validation

FAILURE MODE 2: Secret is exposed publicly  
  Symptom: API key appears in browser network requests, GitHub, or build logs
  Cause: Used a NEXT_PUBLIC_ prefix on a secret, or hardcoded in source code
  Prevention: Never prefix secrets with NEXT_PUBLIC_; use server-side API routes
```

</mental_models>

---

<system_design_breakdown>

## Complete Environment Variable Architecture

```
/project-root
  .env.example        ← COMMITTED: all variable names, no values, with comments
  .env.local          ← NOT COMMITTED: all real local values
  .gitignore          ← Ensures .env.local is never committed

Vercel Dashboard
  ├── Production env vars   ← Live keys (Stripe live, production DB)
  ├── Preview env vars      ← Test keys (Stripe test, staging DB)  
  └── Development env vars  ← Not usually needed (use .env.local locally)

GitHub Actions Secrets
  └── CI/CD tokens          ← Deploy keys, service tokens for CI only
```

## Naming Convention

```
# Pattern: [SERVICE]_[KEY_TYPE]
# All caps, underscores, no hyphens

# Database
DATABASE_URL                         ← Primary DB connection string
DATABASE_URL_UNPOOLED                ← Direct connection (for migrations)

# AI Providers  
OPENAI_API_KEY
ANTHROPIC_API_KEY
GOOGLE_AI_API_KEY

# Auth
CLERK_SECRET_KEY
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY    ← Public — safe in browser
SUPABASE_SERVICE_ROLE_KEY            ← Private — server only
NEXT_PUBLIC_SUPABASE_URL             ← Public — safe in browser
NEXT_PUBLIC_SUPABASE_ANON_KEY        ← Public — safe in browser (RLS protects data)

# Payments
STRIPE_SECRET_KEY
STRIPE_WEBHOOK_SECRET
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY   ← Public — safe in browser

# Email
RESEND_API_KEY

# App Config (public)
NEXT_PUBLIC_APP_URL                  ← e.g. https://myapp.com
NEXT_PUBLIC_VERCEL_ENV               ← Automatically set by Vercel

# Internal tokens
CRON_SECRET                          ← Protects cron job endpoints
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Create .env.example first. Then add variables one service at a time. -->

## Step 1 — Create the .env.example File

This is your single source of truth. Create it at project start and keep it updated whenever you add a new service.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create a comprehensive .env.example file for my project.
Services in use: [LIST ALL SERVICES]

For each service, include:
- All required environment variable names (no values)
- A one-line comment explaining what each variable is and where to get it
- Grouped by service
- Distinction between NEXT_PUBLIC_ (client-safe) and private (server-only) variables

Also update .gitignore to include: .env, .env.local, .env*.local
```

**Verify:** `.env.example` exists and is committed to git. `.env.local` is in `.gitignore`. Run `git status` to confirm no `.env.local` appears as an untracked file.

---

## Step 2 — Add Startup Validation

Catch missing variables at app startup, not silently at runtime.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create /lib/env.ts — a startup environment validation module.

Requirements:
1. List all REQUIRED environment variables (without defaults)
2. List all OPTIONAL variables with their default values
3. On startup: check all required vars exist — if any are missing, throw a descriptive error:
   "Missing required environment variable: [NAME]. 
    See .env.example for setup instructions."
4. Export typed, validated env object for use throughout the app:
   export const env = { openaiApiKey: process.env.OPENAI_API_KEY!, ... }
5. Import this file in your app entry point so it runs on startup

Use zod for validation (recommended) or manual checks.
```

**Verify:** Temporarily remove a required env var from `.env.local`. Start the app. Confirm you get a clear error message pointing to the missing variable.

---

## Step 3 — Sync to Vercel (or Railway)

After `.env.example` is complete, add all variables to your hosting platform.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Here is my .env.example: [PASTE THE FILE CONTENTS — variable names only, no values]

Give me the Vercel CLI command to bulk-upload environment variables.
Also list which variables should be set as:
- Production only (live keys, production DB)
- Preview only (test keys, staging DB)  
- Both production and preview (shared config like NEXT_PUBLIC_APP_URL)
- Development only (rarely needed on Vercel — usually handled by .env.local)
```

**Prompt for Vercel CLI upload:**
```bash
# Install Vercel CLI if needed
npm i -g vercel

# Pull current env vars from Vercel to local (after linking project)
vercel env pull .env.local

# Or add individually via CLI:
vercel env add OPENAI_API_KEY production
# (Will prompt for the value securely)
```

**Verify:** Deploy to Vercel. Check function logs — no "undefined environment variable" errors on startup.

---

## Step 4 — Rotate a Compromised Secret

Use this process whenever a secret is exposed.

**Prompt your AI agent:**
```
I need to rotate the [SECRET_NAME] secret.
It was exposed because: [DESCRIBE — e.g. accidentally committed / appeared in logs]

Give me the exact steps for [SERVICE NAME]:
1. Generate a new secret in [SERVICE] dashboard
2. Update in Vercel/Railway (without downtime — add new, remove old)
3. Update in .env.local
4. Verify the new secret works before removing the old one
5. Remove the old secret
6. If committed to git: commands to remove it from git history

Assume the old secret should be treated as fully compromised and must be invalidated.
```

---

## Step 5 — Validate All Environments Match

Run this check before every major release.

**Prompt your AI agent:**
```
Audit my environment variable setup for gaps.
My .env.example contents: [PASTE]
My Vercel environment variables (names only — copy from dashboard): [PASTE]

Identify:
1. Variables in .env.example not set in Vercel production
2. Variables in .env.example not set in Vercel preview
3. Variables in Vercel not documented in .env.example (undocumented secrets)
4. Any variable that should be environment-specific but is set identically across all environments
   (e.g. Stripe live key set in both preview and production — preview should use test key)

Return a table. Do not change anything.
```

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on environment variable management for my app.
Project context: [PASTE context_anchor JSON]

Current issue: [DESCRIBE — e.g. "missing var in production" / "setting up for first time" / "rotating a leaked key"]

Before doing anything:
1. List the environment variables currently in .env.example (if it exists)
2. Identify any gaps between .env.example and what my app actually needs
3. Tell me which changes should go to: .env.local / Vercel dashboard / both
Do not change any application code — only config files.
```

### env.ts Validation Prompt (Next.js + Zod)
```
Create /lib/env.ts — a fully typed environment validation module for my Next.js app.
Using: zod for validation

Required server-side variables (no defaults allowed):
[LIST: e.g. DATABASE_URL, OPENAI_API_KEY, CLERK_SECRET_KEY]

Required public variables (no defaults allowed):
[LIST: e.g. NEXT_PUBLIC_APP_URL, NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY]

Optional variables with defaults:
[LIST: e.g. LOG_LEVEL = "info", MAX_AI_TOKENS = "1000"]

Output:
- Zod schema validating all variables
- Typed export: export const env = { ... } with camelCase keys
- Error message format: "❌ Invalid environment variables:\n  DATABASE_URL: Required"
- Import in /app/layout.tsx to validate on startup
```

### Vercel Env Var Setup Prompt
```
I need to configure environment variables for my [APP NAME] on Vercel.
Here is my .env.example with all required variables: [PASTE]

For each variable, specify:
1. Should it be set in: Production / Preview / Development / All
2. Should it be a: Secret (hidden after saving) / Plain text
3. Any environment-specific values to note:
   e.g. STRIPE_SECRET_KEY = sk_test_... in Preview, sk_live_... in Production

Also generate: the Vercel CLI commands to add each variable without showing the values.
```

### Secret Rotation Runbook Prompt
```
Create a runbook for rotating [SERVICE_NAME] API keys/secrets.
Project context: [PASTE context_anchor JSON]
Service: [e.g. OpenAI / Stripe / Supabase / Clerk]

The runbook should cover:
1. Where to generate a new key in [SERVICE] dashboard (describe UI steps)
2. How to add the new key to Vercel WITHOUT removing the old one yet
3. How to deploy with the new key and verify it works
4. How to remove the old key once confirmed working
5. How to update .env.local and .env.example
6. Estimated downtime: should be zero if done in this order

Format as a numbered checklist I can follow step by step.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is the difference between NEXT_PUBLIC_ and non-prefixed variables?"

**NEXT_PUBLIC_ prefix:** The variable value is embedded into the JavaScript bundle that gets sent to the user's browser. Anyone can see it in DevTools → Sources.

**No prefix:** The variable only exists on the server. It is never sent to the browser. Even if someone opens DevTools, they cannot find it.

**Rule:** Any variable that controls access to a paid service or database must NEVER have `NEXT_PUBLIC_` prefix. This includes: `OPENAI_API_KEY`, `STRIPE_SECRET_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `CLERK_SECRET_KEY`, `DATABASE_URL`.

Only use `NEXT_PUBLIC_` for things that are genuinely public: your app URL, a publishable key (designed to be public by the service), feature flags that users are allowed to know about.

---

### "I accidentally committed my .env file to GitHub. What do I do?"

**Do this immediately — within minutes:**

1. **Revoke all exposed secrets now** — do not wait. Go to each service dashboard and delete/regenerate the key.
   - OpenAI: platform.openai.com → API Keys → Delete the exposed key → Create new
   - Stripe: dashboard.stripe.com → Developers → API Keys → Roll key
   - Supabase: dashboard.supabase.com → Settings → API → Regenerate service role key

2. **Remove the file from git history:**
```bash
# Using BFG Repo Cleaner (easiest)
brew install bfg
bfg --delete-files .env
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

3. **Update your .gitignore** — add `.env`, `.env.local`, `.env.*` if not already there.

4. **Add new secrets to Vercel/Railway** — the old ones are now revoked.

The exposure window matters. If the repo was public, assume the key was harvested by automated scanners within minutes. GitHub's own secret scanning will have already notified you if it was a known API key format.

---

### "Why do I need different API keys for development, preview, and production?"

**Safety:** A bug in your development environment cannot accidentally charge real users' credit cards, send real emails to customers, or delete production data. Test keys are isolated sandboxes.

**Cost:** OpenAI, Anthropic, and other AI providers charge per token. Development and testing can use up significant quota. Separate keys let you monitor dev spending vs production spending independently.

**Auditability:** If a secret leaks, you need to know which environment was compromised. Separate keys give you that information.

**Most services make this easy:** Stripe has explicit test/live modes. OpenAI lets you create multiple keys with spending limits. Supabase gives you free projects perfect for staging.

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Environment Variable Setup

### Local Verification
```bash
# Check no secrets are committed to git
git log --all --full-history -- .env
git log --all --full-history -- .env.local
# Both should return nothing (no commit history)

# Check .gitignore is working
git check-ignore -v .env.local
# Should output: .gitignore:N:.env.local
# If no output: .env.local is NOT ignored — add it to .gitignore immediately

# Verify all variables in .env.example are set locally
while IFS= read -r line; do
  [[ "$line" =~ ^[A-Z] ]] && key="${line%%=*}" && \
  [[ -z "${!key}" ]] && echo "Missing: $key"
done < .env.example
```

### Production Verification
```bash
# After deploying, check startup validation runs correctly
# Vercel: Functions tab → check for "Missing required environment variable" errors

# Quick smoke test for env var presence (not value exposure)
curl https://yourapp.com/api/health
# Should return: { "status": "ok", "checks": { "database": "ok", "ai_provider": "ok" } }
# NOT: { "error": "process.env.OPENAI_API_KEY is undefined" }
```

### Environment Variable Audit Checklist
```
□ .env.example exists and is committed to git
□ .env.example has every variable the app uses, with comments
□ .env.local is in .gitignore
□ No hardcoded secrets in any source file (run: grep -r "sk-" --include="*.ts" .)
□ No NEXT_PUBLIC_ prefix on server-only secrets
□ Startup validation catches missing required variables
□ Vercel production has all variables from .env.example
□ Vercel preview uses test/sandbox keys, not live keys
□ Different Stripe keys for preview vs production
□ /api/health confirms all services are connected in production
```

### Common Env Var Errors

| Error | Meaning | Fix |
|---|---|---|
| `process.env.X is undefined` at runtime | Var not set in current environment | Add to Vercel dashboard or .env.local |
| `NEXT_PUBLIC_X is undefined` in browser | Var not set with correct prefix or not rebuilt | Add `NEXT_PUBLIC_` prefix; redeploy (not just restarted) |
| Build output contains actual secret value | Used template literal with secret in client code | Move to server-side route; never use secret in client component |
| Stripe charges real money in development | Using live key in dev environment | Switch to `sk_test_` key for local development |
| AI calls fail only in production | Different API key format or quota exhausted | Check production API key has sufficient credits/quota |

</testing_and_qa>

---

<common_patterns>

## Reusable Environment Variable Patterns

### Pattern 1: .env.example Template for AI Products
```bash
# =============================================================
# APP CONFIGURATION
# =============================================================
NEXT_PUBLIC_APP_URL=http://localhost:3000       # Your app's public URL

# =============================================================
# DATABASE (Supabase)
# Get from: supabase.com → Project → Settings → Database
# =============================================================
DATABASE_URL=                                  # Connection string (pooled, port 6543)
DATABASE_URL_UNPOOLED=                         # Direct connection (port 5432, for migrations)
NEXT_PUBLIC_SUPABASE_URL=                      # Project URL (safe to expose)
NEXT_PUBLIC_SUPABASE_ANON_KEY=                 # Anon key (safe to expose — RLS protects data)
SUPABASE_SERVICE_ROLE_KEY=                     # SERVICE ROLE — server only, never expose

# =============================================================
# AUTHENTICATION (Clerk)
# Get from: clerk.com → API Keys
# =============================================================
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=             # Safe to expose
CLERK_SECRET_KEY=                              # SERVER ONLY — never expose
CLERK_WEBHOOK_SECRET=                          # For verifying Clerk webhook events

# =============================================================
# AI PROVIDER (OpenAI)
# Get from: platform.openai.com → API Keys
# =============================================================
OPENAI_API_KEY=                                # SERVER ONLY — never expose

# =============================================================
# PAYMENTS (Stripe)
# Get from: dashboard.stripe.com → Developers → API Keys
# Use sk_test_ for development/preview, sk_live_ for production only
# =============================================================
STRIPE_SECRET_KEY=                             # SERVER ONLY
STRIPE_WEBHOOK_SECRET=                         # For verifying Stripe webhook events
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=            # Safe to expose

# =============================================================
# EMAIL (Resend)
# Get from: resend.com → API Keys
# =============================================================
RESEND_API_KEY=                                # SERVER ONLY

# =============================================================
# INTERNAL SECURITY
# =============================================================
CRON_SECRET=                                   # Random string to protect cron endpoints
                                               # Generate: openssl rand -base64 32
```

### Pattern 2: Zod Environment Validation (Next.js)
```typescript
// /lib/env.ts
import { z } from "zod";

const serverEnvSchema = z.object({
  // Database
  DATABASE_URL: z.string().url("DATABASE_URL must be a valid URL"),
  SUPABASE_SERVICE_ROLE_KEY: z.string().min(1),

  // Auth
  CLERK_SECRET_KEY: z.string().startsWith("sk_", "CLERK_SECRET_KEY must start with sk_"),
  CLERK_WEBHOOK_SECRET: z.string().min(1),

  // AI
  OPENAI_API_KEY: z.string().startsWith("sk-", "OPENAI_API_KEY must start with sk-"),

  // Optional with defaults
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
  MAX_AI_TOKENS: z.coerce.number().default(1000),
});

const clientEnvSchema = z.object({
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().startsWith("pk_"),
  NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
  NEXT_PUBLIC_SUPABASE_ANON_KEY: z.string().min(1),
});

// Validate server env (only runs server-side)
const _serverEnv = serverEnvSchema.safeParse(process.env);
if (!_serverEnv.success && typeof window === "undefined") {
  console.error("❌ Invalid environment variables:");
  _serverEnv.error.issues.forEach((issue) => {
    console.error(`  ${issue.path.join(".")}: ${issue.message}`);
  });
  throw new Error("Invalid environment variables — see above for details");
}

// Validate client env (runs on both client and server)
const _clientEnv = clientEnvSchema.safeParse({
  NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: process.env.NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY,
  NEXT_PUBLIC_SUPABASE_URL: process.env.NEXT_PUBLIC_SUPABASE_URL,
  NEXT_PUBLIC_SUPABASE_ANON_KEY: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
});
if (!_clientEnv.success) {
  throw new Error(`Missing public environment variables: ${_clientEnv.error.message}`);
}

export const env = {
  ..._serverEnv.data,
  ..._clientEnv.data,
} as z.infer<typeof serverEnvSchema> & z.infer<typeof clientEnvSchema>;
```

### Pattern 3: Environment-Specific Service Clients
```typescript
// /lib/stripe.ts — uses correct key per environment automatically
import Stripe from "stripe";
import { env } from "@/lib/env";

// env.STRIPE_SECRET_KEY is automatically the right key
// because Vercel injects different values per environment
export const stripe = new Stripe(env.STRIPE_SECRET_KEY, {
  apiVersion: "2024-06-20",
});

// Helper to check if we are in test mode (safety check)
export const isStripeTestMode = env.STRIPE_SECRET_KEY.startsWith("sk_test_");
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: The NEXT_PUBLIC_ Rule Is Absolute
Any variable prefixed `NEXT_PUBLIC_` is visible to every user of your app. Treat it as fully public. Never put an API key, database password, or any secret with `NEXT_PUBLIC_` prefix.

### Rule 2: Validate Secrets Are Not in Source Files
Run this before every commit:
```bash
# Check for common secret patterns in source files
grep -r "sk-" --include="*.ts" --include="*.tsx" --include="*.js" . | grep -v "node_modules" | grep -v ".env"
grep -r "sk_live_" --include="*.ts" . | grep -v "node_modules"
# Any output here (except in .env.example as placeholder) is a security issue
```

### Rule 3: Use Short-Lived Tokens for CI/CD
GitHub Actions secrets used for deployment (Vercel tokens, Railway tokens) should be scoped to minimum permissions and rotated quarterly. A leaked CI token gives an attacker the ability to deploy malicious code to your production environment.

### Rule 4: Never Log Environment Variable Values
```typescript
// ❌ Logs your actual API key
console.log("Config:", process.env);
console.log("OpenAI key:", process.env.OPENAI_API_KEY);

// ✅ Log presence only
console.log("OpenAI configured:", !!process.env.OPENAI_API_KEY);
```

### Rule 5: Separate Service Accounts Per Environment
Do not share a single Stripe account, OpenAI account, or Supabase project across all environments. Create separate projects/accounts for production. This ensures:
- A development bug cannot charge real customers
- Production billing is isolated and auditable
- A compromised development key cannot access production data

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Checking In .env Files
The most common and most damaging mistake. One commit, and your secrets are in git history forever (even if you delete the file later).  
**Fix:** Run `git check-ignore -v .env.local` right now. If it prints nothing, add `.env.local` to `.gitignore` immediately.

### ❌ Using the Same Key for Test and Production
```bash
# ❌ All environments use the same key
STRIPE_SECRET_KEY=sk_live_realkey123  # In .env.local — can charge real cards in dev

# ✅ Environment-appropriate keys
STRIPE_SECRET_KEY=sk_test_testkey123  # In .env.local — sandbox only
STRIPE_SECRET_KEY=sk_live_realkey123  # In Vercel → Production environment only
```

### ❌ Silently Ignoring Missing Variables
```typescript
// ❌ undefined becomes the string "undefined" in some contexts — silent failures
const apiKey = process.env.OPENAI_API_KEY;
// app continues with broken functionality

// ✅ Fail loudly at startup
const apiKey = process.env.OPENAI_API_KEY;
if (!apiKey) throw new Error("OPENAI_API_KEY is required — see .env.example");
```

### ❌ .env.example With Fake Values That Look Real
```bash
# ❌ Looks like a real key — developers might try to use it
OPENAI_API_KEY=sk-abc123fakekeyhere

# ✅ Obviously a placeholder
OPENAI_API_KEY=                    # Get from: platform.openai.com → API Keys
# or
OPENAI_API_KEY=your_openai_api_key_here
```

### ❌ Accessing env Vars Directly Throughout the Codebase
When `process.env.OPENAI_API_KEY` is scattered across 15 files, renaming it or adding validation means touching 15 files.  
**Fix:** Import from `/lib/env.ts` everywhere. One source of truth.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Secrets Management

### Use a Secrets Manager for Team Environments
When multiple developers need access to the same secrets:
```
Tools:
- Doppler (recommended — syncs secrets to Vercel/Railway automatically)
- HashiCorp Vault (enterprise)
- AWS Secrets Manager (if already on AWS)

With Doppler:
- One place to manage all secrets
- Developers pull secrets via CLI: doppler run -- npm run dev
- Secrets auto-sync to Vercel/Railway on change
- Access control: developers see staging keys, not production keys
```

### Add Secret Scanning to CI
Prevent secrets from ever reaching git:
```yaml
# .github/workflows/secret-scan.yml
name: Secret Scan
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          extra_args: --only-verified
```

### Automate Secret Rotation
For high-value secrets (database passwords, signing keys), set up quarterly automated rotation:
```
Pattern:
1. GitHub Action triggers on schedule (quarterly)
2. Action calls service API to generate new key
3. New key stored in secrets manager
4. Service restarted/redeployed with new key
5. Old key revoked after verification
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — The Missing Env Var That Took 2 Hours to Find

**Bug:** After deploying to production, all reminder creation attempts returned 500 errors. The app worked perfectly in the preview environment.

**Investigation:**
```
1. Checked Vercel function logs: "TypeError: Cannot read properties of undefined (reading 'from')"
2. Traced to /lib/email.ts: resend.emails.send({ from: env.RESEND_FROM_EMAIL, ... })
3. env.RESEND_FROM_EMAIL was undefined
4. Checked Vercel dashboard: RESEND_FROM_EMAIL was set in Preview but not Production
5. The variable had been added to Preview during testing but never added to Production
```

**Why it took 2 hours:** No startup validation. The app started successfully (Resend key existed), but the missing `RESEND_FROM_EMAIL` only surfaced when the code path ran for the first time.

**Prevention added:**
```typescript
// Added to env.ts
RESEND_FROM_EMAIL: z.string().email("RESEND_FROM_EMAIL must be a valid email address"),
```
Now the app refuses to start in any environment where this variable is missing.

---

### Case Study 2: EdTech App — Rotating a Leaked API Key in 12 Minutes

**Incident:** Developer accidentally ran `console.log(process.env)` in a production API route while debugging. The full env dump appeared in Vercel function logs, which were shared with the whole team.

**Response (timed):**
```
T+0:00  Noticed OPENAI_API_KEY in logs
T+0:02  Opened platform.openai.com → Revoked the exposed key
T+0:04  Generated new API key in OpenAI dashboard
T+0:05  Added new key to Vercel → Production environment variable
T+0:06  Triggered Vercel redeploy with new key
T+0:09  Verified new key works (checked AI generation endpoint)
T+0:10  Removed console.log from codebase
T+0:11  Updated .env.local with new key
T+0:12  Confirmed no usage on the old (revoked) key in OpenAI usage dashboard

Total downtime: ~3 minutes (during redeploy)
Total cost of incident: 0 (old key revoked before any external use detected)
```

**Prevention added:** Added secret scanning to CI (Trufflehog) to block commits containing API key patterns.

</real_world_examples>

</skill_document>
