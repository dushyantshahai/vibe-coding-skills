---
name: frontend-architecture
description: Structures frontend apps with clean component hierarchy, state management, and data flow. Use when starting a new frontend, when pages become unmanageable, or when deciding between TanStack Query, Zustand, and useState.
---
# Skill: Frontend Architecture

```json
{
  "skill_id": "frontend-architecture",
  "category": "Frontend & UI/UX",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Frontend Architecture — Structuring a Scalable UI for AI Products</title>

<overview>

## What This Skill Enables
- Design a frontend folder structure and component hierarchy that stays manageable as your app grows
- Make confident decisions about state management before writing a single component
- Understand how data flows from your backend API routes to what users actually see on screen

## Why It Matters for Vibe Coders
Most vibe-coded frontends start as one big file and become unmaintainable within a week. Components get copy-pasted, state gets duplicated, and the AI agent loses context about what already exists. A defined architecture means your AI agent always knows where to put new code — and where to find existing code — without hallucinating duplicate files.

## When to Use This Skill
- At the start of any new project, before building any UI
- When your current frontend has become a mess of copy-pasted components
- Before adding a major new feature that requires new pages or significant state
- When your app needs to share state between components that are far apart in the tree

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js App Router / React / Vue]",
    "styling": "[REPLACE: e.g. Tailwind CSS / CSS Modules / styled-components]",
    "component_library": "[REPLACE: e.g. shadcn/ui / Radix UI / none]",
    "state_management": "[REPLACE: e.g. Zustand / React Context / TanStack Query / none yet]",
    "key_pages": [
      "[REPLACE: e.g. /dashboard — main app view]",
      "[REPLACE: e.g. /lessons/[id] — lesson detail]",
      "[REPLACE: e.g. /reminders — reminder list]"
    ],
    "existing_components": "[REPLACE: list key components already built]",
    "page_or_component_to_build_today": "[REPLACE: ONE page or component only]",
    "api_routes_available": "[REPLACE: list backend routes the frontend can call]",
    "auth_provider": "[REPLACE: e.g. Clerk / Supabase Auth]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Frontend Architecture

### Mental Model 1: The City Planning Analogy
A frontend is like a city:
- **Pages** = districts (each has a clear purpose)
- **Layout components** = roads and infrastructure (shared across the city)
- **Feature components** = buildings (belong to one district)
- **UI components** = bricks and windows (reused everywhere)
- **State** = the city's utility system (water, electricity — shared resources flowing to every building)

A well-planned city has zoning laws. A poorly planned one has a power plant next to a school next to a restaurant — everything entangled with everything else.

### Mental Model 2: Component Hierarchy
Every UI can be broken into three levels:

| Level | What It Is | Examples |
|---|---|---|
| **Pages** | Full screens — one per route | `/dashboard`, `/lessons/[id]`, `/login` |
| **Feature Components** | Complex components tied to a specific feature | `ReminderList`, `LessonPlayer`, `ChatInterface` |
| **UI Components** | Pure, reusable building blocks — no business logic | `Button`, `Card`, `Input`, `Modal`, `Avatar` |

**Rule:** UI components never know about your app's business logic. Feature components never get reused in unrelated features. Pages wire everything together.

### Mental Model 3: Where State Lives
| State Type | What It Is | Where to Put It |
|---|---|---|
| **Server state** | Data from your API (lessons, reminders, user profile) | TanStack Query / SWR |
| **Global UI state** | Things many components need (theme, sidebar open, current user) | Zustand / React Context |
| **Local UI state** | Things only one component needs (dropdown open, input value) | `useState` inside the component |

**Most common mistake:** putting everything in global state. If only one component needs a piece of data, keep it local.

</mental_models>

---

<system_design_breakdown>

## Recommended Folder Structure (Next.js App Router)

```
/app                          ← Pages and layouts (Next.js routing)
  layout.tsx                  ← Root layout (fonts, providers, global nav)
  page.tsx                    ← Landing / home page
  /(auth)                     ← Route group: auth pages
    login/page.tsx
    signup/page.tsx
  /(app)                      ← Route group: authenticated app pages
    layout.tsx                ← App shell (sidebar, top nav)
    dashboard/page.tsx
    lessons/
      page.tsx                ← Lesson list
      [id]/page.tsx           ← Individual lesson
    reminders/page.tsx

/components
  /ui                         ← Pure reusable UI components
    button.tsx
    card.tsx
    input.tsx
    modal.tsx
    spinner.tsx
  /features                   ← Feature-specific components
    /lessons
      lesson-card.tsx
      lesson-player.tsx
      lesson-generator.tsx
    /reminders
      reminder-list.tsx
      reminder-item.tsx
      reminder-form.tsx
    /chat
      chat-interface.tsx
      chat-message.tsx
      chat-input.tsx

/lib
  /api                        ← API call functions (fetch wrappers)
    lessons.ts
    reminders.ts
    users.ts
  /hooks                      ← Custom React hooks
    use-lessons.ts
    use-reminders.ts
    use-auth.ts
  /store                      ← Global state (Zustand)
    ui-store.ts
    user-store.ts
  /utils                      ← Pure helper functions
    format-date.ts
    truncate-text.ts

/types                        ← Shared TypeScript types
  lesson.ts
  reminder.ts
  user.ts
```

## Data Flow Architecture

```
BACKEND API ROUTE
      │
      │ HTTP fetch
      ▼
/lib/api/[feature].ts         ← Typed fetch wrapper
      │
      │ called by
      ▼
/lib/hooks/use-[feature].ts   ← TanStack Query hook
      │
      │ provides { data, isLoading, error }
      ▼
Feature Component              ← Renders data, handles interactions
      │
      │ uses
      ▼
UI Components                  ← Stateless, receives props
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Folder structure first. One page at a time. Test before adding the next. -->

## Step 1 — Generate Folder Structure
Before any component code.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Generate ONLY the folder structure for my frontend.
Include empty placeholder files with one-line comments.
Follow this hierarchy:
- /app for pages and routing
- /components/ui for reusable UI components
- /components/features/[feature-name] for feature-specific components
- /lib/api for typed fetch wrappers
- /lib/hooks for custom React hooks
- /types for shared TypeScript interfaces

Do not write any component logic. Placeholders only.
```

**Verify:** All folders exist. Open 3 files at random — each should have only a comment inside.

---

## Step 2 — Set Up Providers and Root Layout
One-time setup before building any pages.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Set up the root layout at /app/layout.tsx with:
1. [AUTH_PROVIDER] provider wrapper
2. TanStack Query provider (QueryClientProvider)
3. [ZUSTAND store initialisation if needed]
4. Global font setup using next/font
5. Metadata (title, description) for the app

Do not add any UI elements yet — just the providers.
Show me how to verify all providers are working.
```

**Verify:** Run the app. Open browser DevTools → React DevTools. Confirm providers appear in the component tree.

---

## Step 3 — Build One API Fetch Wrapper
Before building any component that needs data.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create /lib/api/[feature].ts — a typed fetch wrapper for the [FEATURE] API.
Include these functions: [LIST — e.g. fetchReminders, createReminder, deleteReminder]

Each function:
- Calls the corresponding backend route: [ROUTE PATH]
- Returns typed data matching the TypeScript interface in /types/[feature].ts
- Handles non-200 responses by throwing a typed error
- Does NOT handle loading/error state — that is the hook's job

Also create /types/[feature].ts with the TypeScript interface for this data.
```

**Verify:** Import one function in a test component. Call it. Confirm it returns typed data.

---

## Step 4 — Create One Custom Hook
Wraps the API call with loading, error, and caching logic.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
The API wrapper at /lib/api/[feature].ts is working.

Create /lib/hooks/use-[feature].ts using TanStack Query.
Include:
1. useGetAll[Feature](userId) — fetches list, caches for 30 seconds
2. useCreate[Feature]() — mutation that invalidates the list cache on success
3. useDelete[Feature]() — mutation with optimistic update (remove from list immediately, restore on error)

Return { data, isLoading, error } from all hooks — consistent shape.
```

**Verify:** Use the hook in a simple test component. Confirm data loads, loading state shows, and errors display.

---

## Step 5 — Build One Page End to End
Using the hook and components together.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
API wrapper ✅  Custom hook ✅

Now build the [PAGE NAME] page at [PAGE PATH].
It should:
1. Use use[Feature]() hook to fetch data
2. Show a loading skeleton while data loads
3. Show an empty state if data is empty
4. Show an error message if the hook returns an error
5. Render each item using [COMPONENT NAME] component
6. Include a button/form to create a new item using the create mutation

Build the page and its direct child components only.
Do not build UI components from scratch — use [shadcn/ui / your component library].
```

**Verify:** Page loads, shows data, shows loading state, and the create action works end to end.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am continuing frontend development for my app.
Project context: [PASTE context_anchor JSON]

Files I worked on last session: [LIST]
What works today: [DESCRIBE]
What I am building today: [ONE PAGE OR COMPONENT ONLY]

Before writing any code:
1. Tell me which files you will create or modify
2. Tell me which existing components or hooks you will reuse
3. Confirm no duplicate component will be created
```

### Folder Structure Prompt
```
Generate the production-ready folder structure for a [FRAMEWORK] frontend.
App type: [DESCRIBE — e.g. EdTech app with lessons, quizzes, and student progress]
Key pages: [LIST YOUR PAGES]
Component library: [shadcn/ui / Radix / none]
State management: [Zustand / TanStack Query / Context]

Return: folder tree with one-line comment per file. No logic, no imports.
```

### API Layer Prompt
```
Create a typed API layer for the [FEATURE] feature.
Framework: [Next.js / React]
Backend route: [ROUTE PATH AND METHOD]
Response type: [DESCRIBE THE DATA SHAPE]

File: /lib/api/[feature].ts
Include functions: [LIST]
Each function uses fetch() with proper error handling and TypeScript return types.
Also generate: /types/[feature].ts with all interfaces for this feature.
```

### TanStack Query Hook Prompt
```
Create a TanStack Query hook for [FEATURE] data.
API functions available: [LIST FROM /lib/api/[feature].ts]
Cache requirements: [e.g. refetch every 60s / keep stale for 5min / always fresh]

Include:
- Query hook for reading data
- Mutation hook for creating
- Mutation hook for updating/deleting with optimistic UI
File: /lib/hooks/use-[feature].ts
```

### State Store Prompt (Zustand)
```
Create a Zustand store for [DESCRIBE WHAT STATE — e.g. UI state: sidebar open/closed, active tab, notification count].

File: /lib/store/[store-name]-store.ts
Include:
- State shape with TypeScript types
- Actions for each state change
- Persist to localStorage for: [LIST FIELDS TO PERSIST, or "none"]

Do not put server/API data in this store — only UI state.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Do I need TanStack Query or can I just use useState + useEffect for API calls?"

You can use `useState + useEffect` but you will quickly reinvent everything TanStack Query gives you for free:
- Automatic caching (no re-fetching the same data on every navigation)
- Loading and error states built in
- Background refetching when the user returns to the tab
- Optimistic updates (show the change instantly, revert if it fails)
- Request deduplication (two components asking for the same data → one request)

**Recommendation:** Use TanStack Query from day one. The setup is 5 minutes and saves hours of debugging later. For simple apps, you may never need Zustand — TanStack Query handles server state; `useState` handles local UI state.

---

### "When do I actually need Zustand (global state)?"

You need global state when:
- Multiple unrelated components need the same piece of data simultaneously (e.g. a notification count shown in the header AND the sidebar)
- You need to persist UI state across page navigations (e.g. whether the sidebar is collapsed)
- You have complex UI flows with many steps (e.g. a multi-step onboarding wizard)

You do **not** need global state for:
- Data from your API (use TanStack Query)
- State only used by one component (use `useState`)
- Form state (use `react-hook-form`)

**Practical rule:** Start with zero global state. Add Zustand only when you find yourself passing the same prop through 3+ levels of components.

---

### "Server Components vs Client Components in Next.js — when does it matter?"

| Component Type | Can It... | Use When |
|---|---|---|
| **Server Component** (default) | Fetch data directly, access DB, keep secrets | Displaying data, SEO-important pages |
| **Client Component** (`"use client"`) | Use useState, useEffect, handle clicks | Interactive UI, forms, real-time updates |

**Simple rule:** Start every component as a Server Component. Add `"use client"` only when you get an error saying you need it (usually when adding `useState`, event handlers, or browser APIs). Never add `"use client"` pre-emptively.

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## Frontend Architecture
- Framework: Next.js App Router — TypeScript
- Component hierarchy: pages (app/) → features (components/features/) → primitives (components/ui/)
- Server state: TanStack Query — `lib/hooks/use-[feature].ts`
- Global UI state: Zustand — `lib/store/` — only for cross-component UI state
- Local state: useState — for component-scoped UI state
- Typed fetch: `lib/api/` — typed wrappers around fetch
- Code splitting: dynamic() for rich text editors, charts, AI panels
- Error boundaries: route level + around AI features + around third-party widgets
- Performance: Web Vitals via next/web-vitals → PostHog custom events
```

</vibe_coder_bridge>

---

<testing_and_qa>

## How to Verify Your Frontend Architecture

### Structural Checks
```bash
# Verify no component is doing too many things
# A component file should rarely exceed 150 lines
find ./components -name "*.tsx" -exec wc -l {} \; | sort -rn | head -10
# If any file is >200 lines, it probably needs to be split
```

### Data Flow Verification Checklist
```
□ API wrapper returns typed data (no `any` types)
□ Hook returns { data, isLoading, error } consistently
□ Loading state renders a skeleton (not blank screen)
□ Empty state renders a helpful message (not blank screen)
□ Error state renders a readable message (not a crash)
□ Create/update mutations invalidate the relevant query cache
□ No raw fetch() calls inside component files — only hook usage
```

### Common Errors and Fixes

| Error | Meaning | Fix |
|---|---|---|
| `Hydration mismatch` | Server-rendered HTML differs from client render | Wrap browser-only code in `useEffect` or use `dynamic()` with `ssr: false` |
| `Cannot read properties of undefined` | Data accessed before it loads | Add `if (!data) return null` or use optional chaining `data?.field` |
| `useQuery is not a function` | TanStack Query not installed or provider missing | Check `QueryClientProvider` wraps your app in layout.tsx |
| `Event handlers cannot be passed to Client Component props` | Passing function from Server to Client Component | Add `"use client"` to the component receiving the handler |
| `React Hook called conditionally` | Hook used inside an `if` statement | Move hook to top of component, outside any conditions |

### Debugging Loop
```
UI bug or data not showing →
  1. Open browser DevTools → Network tab
     - Is the API call being made? If not: check the hook is being called
     - Is the API returning data? If not: check the backend route (use skill-api-routes.md)
     - Is the response the right shape? If not: check your TypeScript types
  2. Open React DevTools → Components tab
     - Find your component in the tree
     - Check what props and state it has
  3. Add a temporary console.log in your hook: console.log('hook data:', data)
  4. Paste error + component code to AI agent:
     "Component [NAME] shows [PROBLEM].
      Hook returns: [PASTE console.log output]
      Component code: [PASTE]
      Fix only this component. Do not change the hook or API layer."
```

</testing_and_qa>

---

<common_patterns>

## Reusable Architecture Patterns

### Code Splitting — Lazy Load Heavy Components

Every component you import increases your initial bundle size and slows first-load for users. Use `dynamic()` to lazy load components that aren't needed immediately.

```typescript
// components/features/editor/rich-text-editor.tsx — heavy library (TipTap/Quill)
import dynamic from "next/dynamic"

// ✅ Only loads when the component is actually rendered
const RichTextEditor = dynamic(
  () => import("@/components/features/editor/rich-text-editor-impl"),
  {
    ssr: false,       // disable SSR for browser-only libraries
    loading: () => <div className="h-64 bg-gray-100 animate-pulse rounded-lg" />,
  }
)

// ✅ For AI components that are conditionally shown:
const AIGenerationPanel = dynamic(
  () => import("@/components/features/ai/generation-panel"),
  { ssr: false }
)

// ✅ For heavy charting libraries:
const AnalyticsChart = dynamic(
  () => import("@/components/features/analytics/chart"),
  { loading: () => <div className="h-48 bg-gray-100 animate-pulse" /> }
)
```

**When to use dynamic imports:**
- Rich text editors (TipTap, Quill, Slate) — large bundle sizes
- Chart libraries (Recharts, Chart.js) — only needed on analytics pages
- AI generation panels — not needed on landing page
- Any component using browser-only APIs (window, document, canvas)
- Any import > 50kb (check with `next build` and examine bundle analysis)

**Bundle analysis:** Add `ANALYZE=true npm run build` with `@next/bundle-analyzer` to see exactly what's in your bundle.

### Error Boundary Placement Strategy

Error boundaries catch component-level crashes and prevent one broken component from taking down the whole page. Place them strategically:

```tsx
// app/(app)/layout.tsx — catch auth/layout crashes
<ErrorBoundary fallback={<FullPageError />}>
  {children}
</ErrorBoundary>

// For AI generation components — crashes here shouldn't affect the rest of the page
<ErrorBoundary
  fallback={<div>AI generation failed. <button onClick={reset}>Try again</button></div>}
  onError={(err) => Sentry.captureException(err, { tags: { feature: "ai-generation" } })}
>
  <AIGenerationPanel />
</ErrorBoundary>

// For third-party widgets (Stripe Elements, auth components) — isolate their failures
<ErrorBoundary fallback={<PaymentFormError />}>
  <StripePaymentForm />
</ErrorBoundary>
```

**Placement rules:**
1. Route level — always (catches page-level crashes)
2. Around AI features — always (AI components have more runtime failure modes)
3. Around third-party widgets — always (you don't control their code)
4. Around individual cards/sections — optional, only if they're independent features
5. Around every single component — never (overhead isn't worth it)

### Pattern 1: Standard Page Structure (Next.js App Router)
```typescript
// /app/(app)/reminders/page.tsx
// Server Component — fetches initial data on the server
import { RemindersView } from "@/components/features/reminders/reminders-view";
import { auth } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";

export default async function RemindersPage() {
  const { userId } = await auth();
  if (!userId) redirect("/login");

  return (
    <main className="container mx-auto py-8">
      <h1 className="text-2xl font-semibold mb-6">Your Reminders</h1>
      <RemindersView userId={userId} />
    </main>
  );
}
```

### Pattern 2: Standard Feature Component (Client)
```typescript
// /components/features/reminders/reminders-view.tsx
"use client";
import { useReminders, useCreateReminder } from "@/lib/hooks/use-reminders";
import { ReminderItem } from "./reminder-item";
import { ReminderForm } from "./reminder-form";
import { Spinner } from "@/components/ui/spinner";
import { EmptyState } from "@/components/ui/empty-state";

export function RemindersView({ userId }: { userId: string }) {
  const { data: reminders, isLoading, error } = useReminders(userId);
  const { mutate: createReminder } = useCreateReminder();

  if (isLoading) return <Spinner />;
  if (error) return <p className="text-red-500">Failed to load reminders.</p>;
  if (!reminders?.length) return <EmptyState message="No reminders yet." />;

  return (
    <div className="space-y-4">
      <ReminderForm onSubmit={createReminder} />
      {reminders.map((reminder) => (
        <ReminderItem key={reminder.id} reminder={reminder} />
      ))}
    </div>
  );
}
```

### Pattern 3: Typed API Fetch Wrapper
```typescript
// /lib/api/reminders.ts
import type { Reminder } from "@/types/reminder";

const BASE_URL = "/api/reminders";

export async function fetchReminders(userId: string): Promise<Reminder[]> {
  const res = await fetch(`${BASE_URL}?userId=${userId}`);
  if (!res.ok) throw new Error(`Failed to fetch reminders: ${res.status}`);
  const json = await res.json();
  return json.data;
}

export async function createReminder(
  data: Pick<Reminder, "text" | "scheduledAt">
): Promise<Reminder> {
  const res = await fetch(`${BASE_URL}/create`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error(`Failed to create reminder: ${res.status}`);
  const json = await res.json();
  return json.data;
}
```

### Pattern 4: TanStack Query Hook
```typescript
// /lib/hooks/use-reminders.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { fetchReminders, createReminder } from "@/lib/api/reminders";

export function useReminders(userId: string) {
  return useQuery({
    queryKey: ["reminders", userId],
    queryFn: () => fetchReminders(userId),
    staleTime: 30_000, // 30 seconds
  });
}

export function useCreateReminder() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createReminder,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["reminders"] });
    },
  });
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Expose Secrets to the Client
Any environment variable that does not start with `NEXT_PUBLIC_` is server-only. Never import server-only files in client components.

```typescript
// ❌ Server secret in a client component
"use client";
const apiKey = process.env.OPENAI_API_KEY; // undefined in browser, exposes intent

// ✅ API calls go through your backend routes, never directly from the client
const response = await fetch("/api/ai/generate", { ... }); // your server handles the AI call
```

### Rule 2: Validate All User Input Before Sending to the Backend
Client-side validation is for UX. Server-side validation (in your API route) is for security. Always have both.

```typescript
// Client-side (UX feedback)
if (!text.trim()) { setError("Reminder text is required"); return; }

// Then also validate on the server (in the API route — see skill-api-routes.md)
```

### Rule 3: Never Trust URL Parameters for Access Control
```typescript
// ❌ Trusting a URL param to determine what data to show
const userId = searchParams.get("userId"); // anyone can change this URL
const data = await fetchUserData(userId);

// ✅ Get userId from the authenticated session
const { userId } = await auth(); // from Clerk/Supabase Auth
const data = await fetchUserData(userId);
```

### Rule 4: Sanitise Before Rendering User-Generated Content
If you display content created by users (or by AI, based on user input), sanitise it before rendering to prevent XSS.

```typescript
// ❌ Dangerous if content contains <script> tags
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// ✅ Use a sanitisation library
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent) }} />
```

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Putting Everything in One Page File
A single page file grows to 500 lines with inline components, API calls, and state all mixed together. The AI agent cannot navigate it reliably.  
**Fix:** Extract feature components the moment a page exceeds 100 lines.

### ❌ Calling the API Directly Inside Components
```typescript
// ❌ API call inside component — no caching, no error handling, duplicated everywhere
useEffect(() => {
  fetch("/api/reminders").then(r => r.json()).then(setData);
}, []);

// ✅ Use a hook — caching, deduplication, and error handling for free
const { data } = useReminders(userId);
```

### ❌ Using `any` Type in the API Layer
The moment you use `any`, TypeScript cannot warn you when the API response shape changes or you access a field that does not exist.  
**Fix:** Always define TypeScript interfaces in `/types/` and use them in every API wrapper.

### ❌ Adding `"use client"` to Every Component Pre-emptively
This disables React Server Components and moves more work to the browser. It also prevents Server Components from fetching data directly, which removes a significant performance advantage.  
**Fix:** Only add `"use client"` when you need it (when you get an error, or when you knowingly need browser-only APIs).

### ❌ No Loading or Empty States
The page renders nothing (or crashes) while data is loading or when there is no data yet. This looks broken to users.  
**Fix:** Every component that fetches data must handle three states: loading, empty, and populated. Build these before styling the happy path.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Frontend

### Add Error Boundaries
Catch React rendering errors gracefully instead of showing a blank screen:
```
Ask your AI agent: "Add a React Error Boundary wrapper around each major feature section.
Show a 'Something went wrong' message with a retry button instead of crashing."
```

### Add Skeleton Loading Components
Replace spinners with content-shaped skeletons for a better perceived performance:
```
Ask your AI agent: "Create a [FeatureName]Skeleton component that matches the shape of 
the loaded [FeatureName] component using Tailwind's animate-pulse class.
Use it in the isLoading state of [ComponentName]."
```

### Add Infinite Scroll or Pagination
When lists grow beyond 20 items:
```
Ask your AI agent: "Add infinite scroll to [ComponentName] using TanStack Query's 
useInfiniteQuery. Load 10 items at a time. Trigger next page when user scrolls 
to within 200px of the bottom."
```

### Add Optimistic UI for Instant Feedback
Make mutations feel instant even before the server confirms:
```typescript
// TanStack Query optimistic update pattern
useMutation({
  mutationFn: deleteReminder,
  onMutate: async (deletedId) => {
    await queryClient.cancelQueries({ queryKey: ["reminders"] });
    const previous = queryClient.getQueryData(["reminders"]);
    queryClient.setQueryData(["reminders"], (old: Reminder[]) =>
      old.filter((r) => r.id !== deletedId)
    );
    return { previous }; // rollback context
  },
  onError: (err, variables, context) => {
    queryClient.setQueryData(["reminders"], context?.previous); // restore on error
  },
});
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Frontend Architecture
```
Pages:          / (landing) → /login → /dashboard → /reminders → /settings
Feature areas:  Reminders, Chat (NLP input), Notification preferences
State split:
  - TanStack Query: reminders list, user profile (server state)
  - Zustand: sidebar open/closed, active view (mobile), unread reminder count
  - useState: form inputs, modal open/closed (local UI state)

Component count at launch: 
  - 6 UI components (Button, Input, Card, Modal, Spinner, EmptyState)
  - 8 feature components (ReminderList, ReminderItem, ReminderForm, ChatInterface,
                          ChatMessage, ChatInput, NotificationSettings, UserMenu)
  - 4 pages

Build order that worked:
  1. UI components (no data dependency)
  2. API wrappers + hooks (no UI dependency)  
  3. Feature components (use hooks)
  4. Pages (wire feature components together)
```

### Case Study 2: EdTech App — Handling Complex State
```
Challenge: Lesson player needed to track:
  - Current section index (local — only the player cares)
  - Reading progress percentage (local — synced to DB every 30s)
  - Completion status (server state — affects other components like the lesson list)
  - Sidebar open/closed (global UI — affects layout outside the player)

Solution:
  - Section index: useState inside LessonPlayer
  - Reading progress: useState + useEffect that debounces DB sync via mutation
  - Completion status: TanStack Query mutation that invalidates lesson list query
  - Sidebar: Zustand UI store

Lesson: not every piece of state belongs in the same place.
Mapping state to the right layer prevented 3 separate bugs that would have
occurred from over-centralising everything in a global store.
```

</real_world_examples>

</skill_document>
