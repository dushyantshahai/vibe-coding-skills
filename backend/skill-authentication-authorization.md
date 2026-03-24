# Skill: Authentication & Authorization

```json
{
  "skill_id": "authentication-authorization",
  "category": "Architecture & Backend",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Authentication & Authorization — Securing Your AI Product From Day One</title>

<overview>

## What This Skill Enables
- Add secure user login, session management, and sign-up flows to any AI product
- Control what each user can see, do, and access (authorization)
- Understand the difference between "who are you?" (authentication) and "what are you allowed to do?" (authorization)

## Why It Matters for Vibe Coders
Auth is the most skipped and most regretted part of vibe-coded apps. Skipping it means any user can access any other user's data. Adding it late means retrofitting every single route. This skill gives you the patterns to add auth correctly at the start — or safely add it now if you skipped it.

## When to Use This Skill
- At the very beginning of any app that has user accounts
- When adding a second user role (e.g. admin vs student vs teacher)
- When a route is accidentally accessible without login
- When setting up social login (Google, GitHub) or magic links

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js App Router / FastAPI / Express]",
    "auth_provider": "[REPLACE: e.g. Clerk / Supabase Auth / Auth0 / NextAuth / none yet]",
    "login_methods": "[REPLACE: e.g. email+password, Google OAuth, magic link]",
    "user_roles": "[REPLACE: e.g. free_user, pro_user / student, teacher, admin / single role]",
    "protected_routes": "[REPLACE: list routes that require login, e.g. /dashboard, /api/lessons]",
    "public_routes": "[REPLACE: list routes that are intentionally public, e.g. /, /pricing, /api/health]",
    "existing_auth_files": "[REPLACE: list auth-related files already created]",
    "auth_task_today": "[REPLACE: ONE task, e.g. add Google OAuth / protect all /api routes / add admin role check]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Auth

### Mental Model 1: The Two-Door Building
**Authentication** = the front door with an ID check. "Are you who you say you are?"  
**Authorization** = the internal doors with keycards. "Are you allowed into this specific room?"

Both are required. A building with only a front door but no internal locks is only half secure.

### Mental Model 2: The Token System
When a user logs in, the server gives them a **token** — like a wristband at a venue. Every time they make a request, they show the wristband. The server checks:
1. Is this wristband real (valid token)?
2. Has it expired?
3. Does this wristband level allow access to this area (role check)?

**JWT (JSON Web Token)** is the most common wristband format. Your auth provider manages creating and validating these automatically.

### Mental Model 3: Authentication Providers Save You Months
Building auth from scratch is a solved problem and an enormous security risk if done wrong. Use an auth provider:

| Provider | Best For | Pricing |
|---|---|---|
| **Clerk** | Next.js apps, best DX, excellent UI components | Free up to 10k MAU |
| **Supabase Auth** | Apps already using Supabase DB | Free tier available |
| **Auth0** | Enterprise, complex role requirements | Free up to 7.5k MAU |
| **NextAuth / Auth.js** | Full control, self-hosted, open source | Free |

**Recommendation for most vibe coders:** Clerk (Next.js) or Supabase Auth. Both handle OAuth, magic links, MFA, and session management out of the box.

</mental_models>

---

<system_design_breakdown>

## Auth Flow Architecture

### Login Flow
```
USER ENTERS CREDENTIALS
         │
         ▼
┌─────────────────────────┐
│    AUTH PROVIDER        │
│  (Clerk / Supabase)     │
│  Validates credentials  │
│  Creates session/token  │
└────────────┬────────────┘
             │ Returns JWT / Session Cookie
             ▼
┌─────────────────────────┐
│       CLIENT            │
│  Stores token securely  │
│  (HTTP-only cookie      │
│   or secure storage)    │
└────────────┬────────────┘
             │ Sends token with every request
             ▼
┌─────────────────────────┐
│    YOUR API ROUTES      │
│  Validate token first   │
│  Extract userId + role  │
│  Then run business logic│
└─────────────────────────┘
```

### Role-Based Access Control (RBAC)
```
USER REQUEST
     │
     ▼
Is user authenticated? ──No──→ 401 Unauthorized
     │
    Yes
     │
     ▼
Does user have required role? ──No──→ 403 Forbidden
     │
    Yes
     │
     ▼
Does user own the resource? ──No──→ 404 Not Found (not 403 — don't reveal it exists)
     │
    Yes
     │
     ▼
Proceed with request ✅
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Set up auth provider → protect one route → test → then expand. -->

## Step 1 — Set Up Your Auth Provider
**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Auth provider: [YOUR CHOICE]
Set up [AUTH PROVIDER] for my [FRAMEWORK] app.

Generate:
1. Installation command
2. Required environment variables (names only, not values)
3. The provider wrapper/middleware setup file
4. How to access the current user's ID in an API route
5. How to access the current user's ID in a page/component

Do not add login UI yet. Just the core setup.
```

**Verify:** In an API route, log `userId`. Call the route while unauthenticated — confirm it returns 401. Call it with a valid session — confirm it returns the userId.

---

## Step 2 — Protect Your First Route
Add auth to one route before touching any others.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Add authentication protection to this route: [ROUTE FILE PATH]
Auth provider: [PROVIDER]

Requirements:
- Check authentication at the very top of the handler, before any other logic
- Return 401 with { success: false, error: "UNAUTHORIZED" } if not authenticated
- Extract userId from the session and make it available to the rest of the handler
- Do not change any existing business logic in the route
```

**Verify:** Call the route without a token → should return 401. Call with valid token → should return the normal response.

---

## Step 3 — Add Login UI
Only after backend auth is working.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Add a login page to my app using [AUTH PROVIDER]'s pre-built components.
Requirements:
- Route: /login
- Support: [LIST LOGIN METHODS — e.g. email+password, Google]
- After successful login: redirect to /dashboard
- If already logged in: redirect away from /login to /dashboard
- Style: match my existing [TAILWIND / CSS] setup
```

**Verify:** Open `/login` in a browser. Complete the login flow. Confirm redirect to `/dashboard`. Confirm the session persists on page refresh.

---

## Step 4 — Protect All Routes with Middleware
After individual route protection is confirmed working.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Add a global auth middleware that:
- Protects ALL routes under /dashboard/* and /api/* (except /api/health and /api/webhooks/*)
- Redirects unauthenticated web requests to /login
- Returns 401 JSON for unauthenticated API requests
- Allows all routes under / (public pages) and /login
Auth provider: [PROVIDER]
Framework: [FRAMEWORK]
Return only the middleware file.
```

**Verify:** Without login, navigate to `/dashboard` — should redirect to `/login`. Call `/api/users` without token — should return 401.

---

## Step 5 — Add Roles (If Your App Has Multiple User Types)
Only if your app has more than one user role.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Add role-based authorization to my app.
Roles needed: [LIST ROLES — e.g. student, teacher, admin]

Generate:
1. How to store and retrieve the user's role (in [AUTH PROVIDER] metadata or DB)
2. A checkRole(userId, requiredRole) utility function
3. Apply role check to this specific route: [ROUTE] — only [ROLE] can access it
4. A roleGuard middleware for protecting entire route groups by role

Do not apply to all routes yet — just this one example route.
```

**Verify:** Log in as a user without the required role. Call the protected route — should return 403. Log in as correct role — should return normal response.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on auth for my app.
Project context: [PASTE context_anchor JSON]
Auth already set up: [YES/NO — if yes, describe what works so far]
Today I am implementing: [ONE AUTH TASK]
Before writing code, tell me which files you will create or modify.
```

### Clerk Setup Prompt (Next.js)
```
Set up Clerk authentication in my Next.js App Router project.
Project context: [PASTE context_anchor JSON]

Generate:
1. Installation: npm install @clerk/nextjs
2. Environment variables needed (names only)
3. /app/layout.tsx — wrap with ClerkProvider
4. /middleware.ts — protect /dashboard and /api routes, allow public routes
5. A helper: getCurrentUserId() for server components and API routes

Do not generate any UI yet.
```

### Supabase Auth Setup Prompt
```
Set up Supabase Auth in my [Next.js / other] project.
Project context: [PASTE context_anchor JSON]
Login methods: [email+password / magic link / Google OAuth]

Generate:
1. Supabase client setup for both server and client usage
2. Auth helper: getSession() for server-side use in API routes
3. Middleware to protect /dashboard/* routes
4. Environment variables needed (names only)

Do not generate UI yet.
```

### Add OAuth Provider Prompt
```
Add [Google / GitHub / Apple] OAuth login to my app.
Auth provider: [CLERK / SUPABASE AUTH / AUTH0]
Project context: [PASTE context_anchor JSON]

Steps I need:
1. What to enable/configure in the [AUTH PROVIDER] dashboard (describe steps)
2. Any code changes needed in my app
3. The callback URL I need to register with [Google / GitHub / Apple]
4. How to test it locally

Keep the existing email login working. Only add [PROVIDER] as an additional option.
```

### Route Protection Audit Prompt
```
Audit all routes in my app for missing authentication.
Scan these directories: [LIST DIRECTORIES — e.g. /app/api, /app/(dashboard)]
Project context: [PASTE context_anchor JSON]

For each route found:
1. Is it currently protected? (yes/no)
2. Should it be protected based on the app's purpose?
3. If unprotected and should be: show me the exact lines to add auth check

Do not modify any files. Return an audit table only.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Clerk vs Supabase Auth — which should I pick?"

**Choose Clerk if:**
- You are building a Next.js web app
- You want the best developer experience and pre-built UI components
- You need features like organisations, multi-tenancy, or complex user management
- You do not want to think about auth at all — just drop in components

**Choose Supabase Auth if:**
- You are already using Supabase as your database
- You want everything in one platform (DB + auth + storage)
- Your auth needs are straightforward (email/password + a couple of OAuth providers)
- You want to minimise the number of third-party services

**Honest take:** Both are excellent. For a new project using Supabase as the DB, use Supabase Auth. For a Next.js project where you want the fastest possible setup with the best UI, use Clerk.

---

### "What is the difference between authentication and authorization, and why does it matter?"

**Authentication** answers: "Are you who you say you are?"  
→ Handled by your auth provider (Clerk/Supabase)

**Authorization** answers: "Are you allowed to do this specific thing?"  
→ Handled by your code (role checks, ownership checks)

Most vibe coders set up authentication (login works!) but forget authorization (any logged-in user can access any other user's data). Both are required.

The simplest authorization rule — and one you must always have — is **ownership check**: when a user requests a resource, verify it belongs to them before returning it.

---

### "Do I need JWT or sessions? What is the difference?"

You do not need to choose — your auth provider handles this for you. But understanding the difference helps you debug auth issues:

**Sessions:** Server stores login state. Client gets a session cookie. Simple, secure, works everywhere.  
**JWTs:** Server issues a signed token. Client stores and sends it. Stateless, works across services.

Clerk uses JWTs. Supabase Auth uses both depending on setup. For most apps: you will never need to think about this if you use an auth provider.

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing Your Auth Implementation

### Auth Test Checklist (Run After Every Auth Change)

```
□ Unauthenticated user visits /dashboard → redirected to /login ✅
□ Unauthenticated request to /api/protected → returns 401 ✅
□ Authenticated user visits /login → redirected to /dashboard ✅
□ Authenticated request to /api/protected → returns data ✅
□ User A cannot access User B's data via /api/resource/[user-b-id] → returns 404 ✅
□ Logged-out user's old token is rejected → returns 401 ✅
□ Role check: student cannot access /api/admin → returns 403 ✅ (if roles exist)
```

### Manual Test Flow
```bash
# Test 1: No auth header
curl -X GET http://localhost:3000/api/reminders
# Expected: 401 {"success":false,"error":"UNAUTHORIZED"}

# Test 2: Invalid token
curl -X GET http://localhost:3000/api/reminders \
  -H "Authorization: Bearer fake-token-123"
# Expected: 401

# Test 3: Valid token (get this from your browser's network tab after logging in)
curl -X GET http://localhost:3000/api/reminders \
  -H "Authorization: Bearer [REAL_TOKEN_FROM_BROWSER]"
# Expected: 200 with your data

# Test 4: Cross-user access (replace ID with another user's resource ID)
curl -X GET http://localhost:3000/api/reminders/[OTHER_USERS_REMINDER_ID] \
  -H "Authorization: Bearer [YOUR_TOKEN]"
# Expected: 404 (not 403 — do not confirm the resource exists)
```

### Debugging Loop
```
Auth error received →
  1. Check if it is 401 (not authenticated) or 403 (authenticated but forbidden)
  2. For 401: Is the token being sent? Check the request headers in browser DevTools → Network tab
  3. For 401: Is the token expired? Check the token's exp claim at jwt.io
  4. For 403: Is the user's role correctly set? Log the role claim from the token
  5. Paste the error + your middleware/route code to AI agent:
     "Auth fails with [STATUS CODE]. Token is: [paste decoded JWT claims, NOT the raw token].
      Here is my auth check code: [PASTE].
      What is wrong with this check?"
```

### Common Auth Errors

| Error | Meaning | Fix |
|---|---|---|
| `Invalid token signature` | Token was tampered with or wrong secret key | Verify CLERK_SECRET_KEY or Supabase JWT secret in env vars |
| `Token expired` | Session is too old | Implement token refresh; check session expiry settings |
| `Missing authorization header` | Client not sending token | Check frontend — is the auth header being added to requests? |
| `User not found` | Token valid but user deleted from DB | Handle this case in your auth check |
| `CORS + 401` | Auth header blocked by CORS | Add `Authorization` to your CORS allowed headers |

</testing_and_qa>

---

<common_patterns>

## Reusable Auth Patterns

### Pattern 1: Auth Helper for Next.js App Router (Clerk)
```typescript
// /lib/auth.ts
import { auth, currentUser } from "@clerk/nextjs/server";
import { NextResponse } from "next/server";

export async function requireAuth() {
  const { userId } = await auth();
  if (!userId) {
    return {
      userId: null,
      error: NextResponse.json(
        { success: false, error: "UNAUTHORIZED" },
        { status: 401 }
      ),
    };
  }
  return { userId, error: null };
}

// Usage in any route:
// const { userId, error } = await requireAuth();
// if (error) return error;
```

### Pattern 2: Middleware (Next.js + Clerk)
```typescript
// /middleware.ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isPublicRoute = createRouteMatcher([
  "/",
  "/login(.*)",
  "/pricing",
  "/api/health",
  "/api/webhooks(.*)",
]);

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: ["/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)", "/(api|trpc)(.*)"],
};
```

### Pattern 3: Role Check Utility
```typescript
// /lib/roles.ts
type Role = "student" | "teacher" | "admin";

export async function requireRole(userId: string, requiredRole: Role) {
  const user = await clerkClient.users.getUser(userId);
  const userRole = user.publicMetadata?.role as Role | undefined;

  const roleHierarchy: Record<Role, number> = {
    student: 1,
    teacher: 2,
    admin: 3,
  };

  const hasAccess =
    userRole &&
    roleHierarchy[userRole] >= roleHierarchy[requiredRole];

  if (!hasAccess) {
    return {
      authorized: false,
      error: NextResponse.json(
        { success: false, error: "FORBIDDEN" },
        { status: 403 }
      ),
    };
  }

  return { authorized: true, error: null, role: userRole };
}
```

### Pattern 4: Auth Setup for FastAPI (Python)
```python
# /auth/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    token = credentials.credentials
    try:
        payload = jwt.decode(
            token,
            options={"verify_signature": False}  # Use your provider's verification
        )
        user_id = payload.get("sub")
        if not user_id:
            raise HTTPException(status_code=401, detail="UNAUTHORIZED")
        return {"user_id": user_id}
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="TOKEN_EXPIRED")
    except Exception:
        raise HTTPException(status_code=401, detail="INVALID_TOKEN")

# Usage in any route:
# @router.get("/reminders")
# async def get_reminders(current_user = Depends(get_current_user)):
#     user_id = current_user["user_id"]
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE — every app with user accounts must follow these -->

### Rule 1: Check Auth Before Everything Else
The authentication check is always the first line of every protected route handler. No exceptions.

### Rule 2: Use 404 Instead of 403 for Resource Ownership Failures
When a user tries to access a resource that belongs to someone else, return 404 — not 403. Returning 403 confirms the resource exists, which leaks information.

### Rule 3: Never Store Tokens in localStorage
```typescript
// ❌ Vulnerable to XSS attacks
localStorage.setItem("token", jwtToken);

// ✅ Use HTTP-only cookies (your auth provider handles this)
// Clerk and Supabase Auth store tokens in HTTP-only cookies automatically
```

### Rule 4: Always Verify Webhook Signatures
If you have webhooks from your auth provider (e.g. user created, user deleted), always verify the signature before processing:
```typescript
// Clerk webhook verification example
import { Webhook } from "svix";

export async function POST(req: NextRequest) {
  const svix_id = req.headers.get("svix-id");
  const svix_signature = req.headers.get("svix-signature");
  const body = await req.text();

  const wh = new Webhook(process.env.CLERK_WEBHOOK_SECRET!);
  try {
    wh.verify(body, { "svix-id": svix_id!, "svix-signature": svix_signature! });
  } catch {
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }
  // Process webhook event...
}
```

### Rule 5: Implement Session Revocation
Users must be able to log out and have their session immediately invalidated. Never use tokens with expiry > 24 hours without a refresh mechanism.

### Rule 6: Rotate Secrets Regularly
Your `CLERK_SECRET_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, and JWT secrets must be rotated when:
- A team member leaves
- You suspect a leak
- Quarterly as a routine practice

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Adding Auth After Building All Features
Every route needs to be retrofitted. You will miss some. Your users' data will be exposed.  
**Fix:** Add auth setup in Step 1, before building any features.

### ❌ Checking Auth in Some Routes But Not Others
You protect `/api/reminders` but forget `/api/reminders/[id]`. An attacker guesses IDs.  
**Fix:** Use middleware to protect all routes by default, then explicitly list public exceptions.

### ❌ Storing the Service Role Key in Client Code
The Supabase service role key bypasses all RLS. If it reaches the browser, any user has admin access to your entire database.  
**Fix:** Service role key is only ever used in server-side API routes. Never in client components or `.env` files prefixed with `NEXT_PUBLIC_`.

### ❌ Not Syncing Auth Users to Your Database
Your auth provider knows about users. Your database does not. Your app tries to create a reminder for a user who does not exist in your DB yet.  
**Fix:** Use a webhook from your auth provider to create a matching user record in your DB whenever a new user signs up.

```typescript
// /api/webhooks/clerk/route.ts
// When Clerk fires "user.created", create the user in your database
case "user.created":
  await db.user.create({
    data: {
      id: event.data.id,  // Use Clerk's user ID as your DB user ID
      email: event.data.email_addresses[0].email_address,
      name: `${event.data.first_name} ${event.data.last_name}`.trim(),
    },
  });
```

### ❌ Returning Different Errors for Valid vs Invalid Users
```typescript
// ❌ Leaks whether a user account exists
if (!user) return 404("User not found");
if (!passwordMatch) return 401("Wrong password");

// ✅ Same generic error regardless
if (!user || !passwordMatch) return 401("Invalid credentials");
```

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Auth Layer

### Add Multi-Factor Authentication (MFA)
Both Clerk and Supabase Auth support MFA (TOTP authenticator apps, SMS) with minimal configuration. Enable it in the auth provider dashboard — no code changes required for basic MFA.

### Add Organisation / Workspace Support
For B2B AI products where multiple users share a workspace (e.g. a teacher's classroom, a company's reminder board):
```
Ask your AI agent: "Add Clerk Organizations to my app.
Allow users to create and join organisations.
Add organisation_id to all relevant database tables.
Update RLS policies to allow org members to see shared resources."
```

### Add Impersonation for Support / Admin
Allows admin users to view the app as any user — critical for debugging user-reported issues:
```
Clerk supports user impersonation natively in the dashboard.
For Supabase: implement an admin-only endpoint that issues a temporary session token for the target user.
```

### Audit Log for Sensitive Actions
```sql
CREATE TABLE audit_logs (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES users(id),
  action      TEXT NOT NULL,   -- e.g. 'reminder.deleted', 'user.role_changed'
  resource_id UUID,
  metadata    JSONB,
  ip_address  INET,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Adding Auth Mid-Build
**Situation:** App was 60% built with no auth. All API routes were public.

**Migration approach (one route at a time):**
```
Day 1: Set up Clerk, add middleware, test on /api/health only
Day 2: Protect /api/reminders/create — add userId from auth to every reminder
Day 3: Protect /api/reminders (GET) — filter by userId
Day 4: Protect /api/reminders/[id] — add ownership check
Day 5: Set up webhook to sync new Clerk users to DB users table
Day 6: Add login/signup UI using Clerk components
Total: 6 focused sessions, one route per session, zero data exposure
```

**Key lesson:** Migrating auth is painful but manageable if done one route at a time with the auth check added first before any other changes.

---

### Case Study 2: EdTech App — Multi-Role System
**Roles needed:** `student`, `teacher`, `admin`

```
Permissions matrix:
                    student   teacher   admin
View own lessons      ✅        ✅        ✅
Generate new lesson   ✅        ✅        ✅
View all students     ❌        ✅        ✅
Create curriculum     ❌        ✅        ✅
Delete any content    ❌        ❌        ✅
View usage analytics  ❌        ❌        ✅

Implementation:
- Role stored in Clerk's publicMetadata
- Set on user creation via webhook: default role = "student"
- Teachers promoted via admin dashboard (admin-only route)
- requireRole() utility applied per route
- Middleware handles the student/public split at the route group level
```

</real_world_examples>

</skill_document>
