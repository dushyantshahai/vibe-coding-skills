---
name: analytics-experimentation
description: Instruments apps with event tracking, builds funnels, and runs A/B tests. Use before the first real users arrive, when you cannot explain why users churn, or when making product decisions that need evidence instead of opinion.
---
# Skill: Analytics & Experimentation

```json
{
  "skill_id": "analytics-experimentation",
  "category": "Product & Non-Technical",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Analytics & Experimentation — Measuring What Matters in AI Products</title>

<overview>

## What This Skill Enables
- Instrument your app with the right events so you can answer "is this feature working?" with data, not guesses
- Build a simple funnel that shows you where users drop off in your core journey
- Run lightweight A/B tests to make product decisions with evidence instead of opinions
- Track AI-specific metrics that tell you whether your AI features are actually helping users

## Why It Matters for Vibe Coders
You cannot improve what you cannot measure. Without analytics, you are making product decisions based on your own usage (which is atypical) and the occasional user complaint (which is a biased sample). Analytics reveal what users actually do versus what you think they do. For AI products specifically, you need metrics that capture AI quality — not just engagement — because a feature that is used but produces bad outputs is not a success.

## When to Use This Skill
- Before launching to real users — add basic event tracking first
- When you have users but cannot explain why some stay and most leave
- When you want to test whether a UX change or new AI feature actually improves outcomes
- When reporting progress to a stakeholder or investor and you need real numbers

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "analytics_tool": "[REPLACE: e.g. Vercel Analytics / PostHog / Mixpanel / none yet]",
    "current_tracking": "[REPLACE: what events are already tracked, if any]",
    "core_user_journey": "[REPLACE: describe what a successful user session looks like]",
    "north_star_metric": "[REPLACE: the ONE number that best represents product health, e.g. 'weekly active reminders per user' or 'lessons completed per week']",
    "analytics_task_today": "[REPLACE: e.g. add event tracking / build funnel / set up A/B test]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Analytics

### Mental Model 1: The Metrics Hierarchy
Not all metrics are equal. Organise them into a hierarchy:

```
NORTH STAR METRIC (one number)
  The single best indicator of product health.
  Examples:
  - Reminders completed per active user per week
  - Lessons completed per student per week
  - Messages sent in chat per DAU

  ↓ driven by ↓

DRIVER METRICS (3-5 numbers)
  Leading indicators that predict the North Star.
  Examples:
  - Day-1 retention (do users come back the next day?)
  - Reminder creation rate (are users actually creating reminders?)
  - AI generation success rate (are AI calls producing valid output?)

  ↓ supported by ↓

DIAGNOSTIC METRICS (many)
  Explain why drivers are moving.
  Examples:
  - Funnel drop-off at step X
  - Error rate on feature Y
  - Average AI response time
```

### Mental Model 2: The Four Questions Analytics Must Answer
```
1. ACQUISITION: How are users finding my app?
   (referral source, sign-up conversion rate)

2. ACTIVATION: Are new users experiencing the core value?
   (did they complete the first meaningful action?)

3. RETENTION: Are users coming back?
   (day-1, day-7, day-30 retention)

4. CORE VALUE: Are they doing the thing the product is for?
   (reminders created and completed / lessons finished)
```

### Mental Model 3: AI-Specific Metrics You Need
Standard analytics tools track user behaviour. For AI products, you also need to track AI performance:

```
AI QUALITY METRICS:
  - Generation success rate (% of AI calls that produce valid output)
  - User regeneration rate (% of AI outputs the user discards and retries)
  - User edit rate (% of AI outputs the user edits before using)
  - Implicit rating (did the user complete the journey after the AI output?)

AI COST METRICS:
  - Cost per active user per day
  - Cost per feature per day
  - Token efficiency (tokens used vs content produced)
```

</mental_models>

---

<system_design_breakdown>

## Analytics Architecture

```
USER ACTION (in browser / app)
       │
       ▼
EVENT FIRED (client-side)
       │ track("event_name", { properties })
       ▼
ANALYTICS SERVICE
(Vercel Analytics / PostHog / Mixpanel)
       │
       ▼
DASHBOARDS + FUNNELS + COHORTS

PARALLEL STREAM (server-side):
AI CALL completes
       │
       ▼
ai_generations TABLE (your DB)
       │
       ▼
SQL QUERIES on your DB dashboard
(Supabase table editor or pg admin)
```

## Event Taxonomy

Every event follows this naming convention:
```
[OBJECT]_[ACTION]
Examples:
  reminder_created
  reminder_completed
  reminder_deleted
  lesson_generated
  lesson_started
  lesson_completed
  chat_message_sent
  user_signed_up
  user_onboarded (first value moment completed)
```

Always include these properties on every event:
```typescript
{
  userId: string,       // who
  timestamp: Date,      // when (auto-added by most tools)
  sessionId?: string,   // which session
  // Feature-specific properties:
  [relevant_context]: value
}
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Track the core journey first. Add detail only after you understand the basics. -->

## Step 1 — Define Your North Star and Core Events

Before any tracking code, decide what you are measuring.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Help me define my analytics foundation:

1. North Star Metric: suggest the single best number to track product health for [APP TYPE]
2. Core Journey Events: list the 5-7 events that represent a user's journey through the key feature
   Format each as: [object]_[action] with properties to track
3. Activation Event: which single event best represents "this user got value from the product"?
4. Retention Signal: what does a retained user do that a churned user does not?

Keep it minimal. I can add more events later.
```

---

## Step 2 — Set Up Analytics Tool

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Set up [Vercel Analytics / PostHog / Mixpanel] in my Next.js app.

Requirements:
1. Install the analytics package
2. Add the provider to /app/layout.tsx
3. Create /lib/analytics.ts — a typed wrapper with one function per core event:
   - trackReminderCreated(userId: string, properties: {...})
   - trackLessonCompleted(userId: string, properties: {...})
   [etc. for each core event]
4. Each function calls the underlying analytics tool
5. Each function is safe to call from both server and client components

Do NOT track: passwords, tokens, full email addresses, health data, or any PII beyond userId.
```

**Verify:** Trigger each event manually. Check the analytics tool's live event stream. Each event appears with correct properties.

---

## Step 3 — Instrument the Core Journey

Add tracking calls to each step of your core user journey.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Analytics wrapper is set up at /lib/analytics.ts

Add tracking to the core user journey.
For each step in this flow: [DESCRIBE YOUR CORE FLOW]

Add the appropriate track() call:
- After successful reminder creation → trackReminderCreated()
- After lesson completion → trackLessonCompleted()
- etc.

Rules:
- Track on SUCCESS only (after the DB write confirms, not on button click)
- Never track in the try block before we know it succeeded
- Never track if userId is not available
- Do not block the UI for tracking calls (fire-and-forget with .catch(console.error))
```

**Verify:** Complete the core user journey end-to-end. Check the analytics dashboard — all events appear in the correct order.

---

## Step 4 — Build a Funnel

A funnel shows you where users drop off between steps.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Events tracked: [LIST YOUR EVENTS]

Help me build a funnel for the core user journey using [ANALYTICS TOOL]:
Steps:
1. user_signed_up
2. user_onboarded (completed setup)
3. [first core action] — e.g. reminder_created
4. [return visit] — e.g. day_1_active

For each step, tell me:
1. How to configure this funnel in [ANALYTICS TOOL]
2. What a healthy drop-off percentage looks like at each step
3. The most common reason users drop off at each step

If using PostHog: write the SQL query for this funnel using the raw events table.
```

---

## Step 5 — Set Up a Simple A/B Test

For testing a UX change or new feature variant.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

I want to A/B test: [DESCRIBE WHAT YOU ARE TESTING]
Hypothesis: [If we X, then Y will increase because Z]
Metric to improve: [WHICH EVENT OR RATE]
Traffic split: 50/50

Set up this test using [PostHog feature flags / custom]:
1. Create a feature flag: [flag_name] — boolean
2. In the relevant component: check the flag value and render variant A or B
3. Track which variant the user saw: trackExperimentViewed(flagName, variant)
4. Success metric: compare [EVENT RATE] between variant A and variant B

Duration: run for [X days] or until [X conversions per variant] — whichever comes first.
```

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am adding analytics to my app.
Project context: [PASTE context_anchor JSON]

Analytics tool: [TOOL OR "not set up yet"]
Events currently tracked: [LIST OR "none"]
Today I am adding: [ONE TASK]

Before writing code:
1. Confirm the event naming convention we are using ([object]_[action])
2. Confirm that tracking calls fire on success (not on button click)
3. Confirm no PII will be included in event properties
```

### North Star + Metric Design Prompt
```
Help me define my product metrics.
App type: [DESCRIBE — e.g. EdTech for students / productivity reminder app]
Core user action: [DESCRIBE — e.g. creating and completing reminders]

Design:
1. North Star Metric (one number that captures product health)
2. Three driver metrics (leading indicators that predict the North Star)
3. Five core events to track (the user journey)
4. One activation event (the moment a new user gets value)
5. AI quality metric specific to my app (not just standard engagement)

Be opinionated. Suggest specific numbers for healthy benchmarks
where industry data exists (e.g. "day-1 retention above 30% is healthy for productivity apps").
```

### PostHog Event Tracking Setup Prompt
```
Set up PostHog analytics for my Next.js app.
Project context: [PASTE context_anchor JSON]

1. Install: npm install posthog-js posthog-node
2. Configure PostHog provider in /app/layout.tsx
3. Create /lib/analytics.ts with typed event functions:
   [LIST YOUR CORE EVENTS AND THEIR PROPERTIES]
4. Server-side tracking: use posthog-node for events that occur in API routes
5. Client-side tracking: use posthog-js for UI interaction events

POSTHOG_KEY env var needed — add to .env.example.
```

### A/B Test Results Analysis Prompt
```
My A/B test has run for [X days].
Test: [DESCRIBE WHAT WAS TESTED]
Metric: [WHAT WAS MEASURED]

Results:
Control (A): [N] users, [X]% conversion
Variant (B): [N] users, [Y]% conversion

Analyse:
1. Is the difference statistically significant? (use a 95% confidence threshold)
2. What is the relative improvement (or decline)?
3. Should I ship variant B, revert to A, or run the test longer?
4. What does this result tell me about my hypothesis?

Show your statistical reasoning.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Which analytics tool should I use?"

| Tool | Best For | Cost |
|---|---|---|
| **Vercel Analytics** | Page views, web vitals — zero setup if on Vercel | Free on hobby plan |
| **PostHog** | Full-featured product analytics, A/B testing, session replay | Free up to 1M events/month |
| **Mixpanel** | Funnel analysis, cohort retention — strong out-of-box charts | Free up to 20M events/month |
| **Custom (DB queries)** | AI-specific metrics — query your ai_generations table | Free (uses your existing DB) |

**Recommendation for most vibe coders:** Vercel Analytics for page performance + PostHog for product events. Both have generous free tiers and together cover 90% of what you need.

---

### "What is a good day-1 retention rate?"

Industry benchmarks vary by product category, but for an AI productivity/EdTech app:
- Day-1 retention > 30%: healthy
- Day-7 retention > 15%: healthy
- Day-30 retention > 8%: healthy

If your day-1 retention is below 15%, the onboarding flow or first-value experience needs work before worrying about anything else. Nothing compounds positively if people do not come back after day 1.

---

### "What is statistical significance and do I need it for A/B tests?"

Statistical significance tells you whether the difference you see between A and B is likely real, or just random variation.

**Why it matters:** If variant B has 25% conversion and variant A has 20%, that sounds like a 25% improvement. But if you only tested 50 people in each group, the difference could easily be random chance. Statistical significance tells you how likely that is.

**Practical rule:** Run your A/B test until each variant has at least 100 conversions (or 1,000 users) before drawing conclusions. Use a free tool like [abtestguide.com](https://abtestguide.com/calc/) to check significance. Never ship a variant based on less than 100 conversions per group.

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## Analytics & Experimentation
- Analytics tool: PostHog — typed wrapper in lib/analytics/index.ts
- Consent: CookieBanner component — gates PostHog on user consent (GDPR compliant)
- Event catalog: lib/analytics/events.ts — typed constants, versioned
- AI quality metrics: regeneration_rate, edit_rate tracked as custom events
- North Star metric: [your metric e.g. "generations per active user per week"]
- A/B testing: PostHog feature flags — minimum 100 conversions per variant before deciding
- Cohort retention: PostHog Retention insight + SQL in docs/analytics-queries.sql
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Analytics Setup

### Event Tracking Verification
```typescript
// Manual verification — run this after adding any new tracking
// 1. Open your app in development
// 2. Open the analytics tool's live event stream (PostHog: Live Events, Vercel: Analytics)
// 3. Complete each tracked action
// 4. Verify:

const EXPECTED_EVENTS = [
  { name: "reminder_created", properties: ["userId", "task", "hasRecurrence"] },
  { name: "lesson_completed", properties: ["userId", "lessonId", "completionPercent"] },
  // Add all your core events
];

// Check each event in the live stream:
// ✅ Event appears within 5 seconds of the action
// ✅ All expected properties are present
// ✅ No PII (email, full name, phone) in properties
// ✅ userId is present and correct
// ✅ Event fires ONCE (not duplicated by React re-renders)
```

### Funnel Verification
```
After instrumenting the core journey:
1. Create a test account
2. Complete the full journey from sign-up to first core value
3. Open your analytics funnel
4. Verify: each step you completed appears in the funnel
5. Verify: the funnel shows 100% for steps you completed (or close to it)
   (Small discrepancy is normal due to event processing delay)
```

### Common Analytics Issues

| Issue | Symptom | Fix |
|---|---|---|
| Events fire multiple times | Funnel shows 200% users at first step | Track on success only, not on button click; check for React double-renders |
| Events missing userId | Events appear in analytics but cannot be attributed to a user | Wait for auth to resolve before tracking; pass userId explicitly |
| Events not appearing | No events in live stream | Check env var (POSTHOG_KEY); verify provider is wrapping the app in layout.tsx |
| PII in events | User emails visible in analytics | Audit event properties; replace email with userId |
| A/B test contamination | Users see both variants | Ensure feature flag assignment is consistent per user (cached by userId, not session) |

</testing_and_qa>

---

<common_patterns>

## Reusable Analytics Patterns

### Pattern 1: Typed Analytics Wrapper
```typescript
// /lib/analytics.ts
import posthog from "posthog-js";

// All events in one place — typed, consistent naming
export const analytics = {
  // User lifecycle
  userSignedUp: (userId: string, method: "email" | "google") =>
    posthog.capture("user_signed_up", { userId, method }),

  userOnboarded: (userId: string) =>
    posthog.capture("user_onboarded", { userId }),

  // Core feature — Reminders
  reminderCreated: (userId: string, props: {
    hasRecurrence: boolean;
    createdViaAI: boolean;
    daysUntilDue: number;
  }) => posthog.capture("reminder_created", { userId, ...props }),

  reminderCompleted: (userId: string, props: {
    reminderId: string;
    daysAfterDue: number; // negative = completed early
  }) => posthog.capture("reminder_completed", { userId, ...props }),

  // AI quality signals
  aiOutputRated: (userId: string, props: {
    feature: string;
    rating: "up" | "down";
    generationId: string;
  }) => posthog.capture("ai_output_rated", { userId, ...props }),

  aiOutputRegenerated: (userId: string, feature: string) =>
    posthog.capture("ai_output_regenerated", { userId, feature }),
};

// Usage: analytics.reminderCreated(userId, { hasRecurrence: true, createdViaAI: true, daysUntilDue: 3 });
```

### Pattern 2: AI Quality Metric Query
```sql
-- AI regeneration rate by feature (lower = better AI output quality)
-- Run weekly to track AI quality trends
SELECT
  feature,
  COUNT(*) as total_generations,
  SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) as successes,
  ROUND(SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) as success_rate
FROM ai_generations
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY feature
ORDER BY success_rate ASC;

-- Cross-reference with your analytics tool's regeneration events
-- Features with low success_rate AND high regeneration_rate need prompt improvement
```

### Pattern 3: Core Funnel Query (PostHog SQL)
```sql
-- Day-1 retention funnel
-- Step 1: signed up
-- Step 2: created first reminder (activation)
-- Step 3: active next day (retention)
SELECT
  COUNT(DISTINCT person_id) as total_signups,
  COUNT(DISTINCT CASE WHEN activated THEN person_id END) as activated,
  COUNT(DISTINCT CASE WHEN retained_day1 THEN person_id END) as retained_day1,
  ROUND(COUNT(DISTINCT CASE WHEN activated THEN person_id END) * 100.0 /
        COUNT(DISTINCT person_id), 1) as activation_rate,
  ROUND(COUNT(DISTINCT CASE WHEN retained_day1 THEN person_id END) * 100.0 /
        COUNT(DISTINCT person_id), 1) as day1_retention_rate
FROM (
  SELECT
    person_id,
    MIN(timestamp) as signup_time,
    MAX(CASE WHEN event = 'reminder_created' THEN 1 END) = 1 as activated,
    MAX(CASE WHEN event = 'reminder_created'
             AND timestamp > MIN(timestamp) + INTERVAL '1 day'
             AND timestamp < MIN(timestamp) + INTERVAL '2 days'
             THEN 1 END) = 1 as retained_day1
  FROM events
  WHERE event IN ('user_signed_up', 'reminder_created')
    AND timestamp > NOW() - INTERVAL '30 days'
  GROUP BY person_id
) cohort;
```

### Pattern 4: Event Versioning — Prevent Analytics Schema Drift

When you rename an event or change its properties, historical data becomes inconsistent. Events queried today won't include data from before the rename.

```typescript
// lib/analytics/events.ts — typed event catalog with versions
// RULE: Never rename or remove an event. Add new ones instead.
// When changing behaviour: deprecate old event, add new event with v2 suffix

export const ANALYTICS_EVENTS = {
  // ✅ Stable events — never change property names once shipped
  "generation_started": "generation_started",           // v1 — keep forever
  "generation_completed": "generation_completed",        // v1

  // ✅ When you need to change structure: add v2, deprecate v1
  "generation_completed_v2": "generation_completed_v2", // v2 — added 2025-03 (added model field)

  // ✅ Document deprecation in comments
  /** @deprecated Use generation_completed_v2 — stop firing after 2025-06-01 */
  // "generation_completed": ...  ← already declared above, don't fire in new code
} as const

// lib/analytics/index.ts — add event version to every call for traceability
export const analytics = {
  track: (event: string, properties: Record<string, unknown>) => {
    posthog.capture(event, {
      ...properties,
      $event_schema_version: 1,  // increment when properties change
      $client_timestamp: new Date().toISOString(),
    })
  }
}
```

**Event registry pattern:** Keep a `docs/analytics-events.md` file that lists every event, its properties, when it was added, and when it was deprecated. This is your analytics schema changelog.

### Pattern 5: Cohort Analysis — Understanding Which Users Retain

Conversion rates in isolation are misleading. A 40% day-7 retention rate means nothing without knowing if it's improving or declining across cohorts.

```sql
-- Day-7 retention cohort query (run in PostHog SQL or your data warehouse)
-- Groups users by signup week and measures what % came back 7 days later

WITH cohorts AS (
  SELECT
    user_id,
    DATE_TRUNC('week', MIN(timestamp)) as cohort_week,
    MIN(timestamp) as first_seen
  FROM events
  WHERE event = 'user_signed_up'
  GROUP BY user_id
),
retention AS (
  SELECT
    c.user_id,
    c.cohort_week,
    DATE_TRUNC('week', e.timestamp) as activity_week,
    DATE_DIFF('week', c.cohort_week, DATE_TRUNC('week', e.timestamp)) as weeks_since_signup
  FROM cohorts c
  JOIN events e ON c.user_id = e.user_id
  WHERE e.event = 'generation_completed'  -- your activation/core value event
)
SELECT
  cohort_week,
  COUNT(DISTINCT CASE WHEN weeks_since_signup = 0 THEN user_id END) as week_0_users,
  COUNT(DISTINCT CASE WHEN weeks_since_signup = 1 THEN user_id END) as week_1_retained,
  ROUND(
    100.0 * COUNT(DISTINCT CASE WHEN weeks_since_signup = 1 THEN user_id END) /
    NULLIF(COUNT(DISTINCT CASE WHEN weeks_since_signup = 0 THEN user_id END), 0),
    1
  ) as week_1_retention_pct
FROM retention
GROUP BY cohort_week
ORDER BY cohort_week DESC;
```

In PostHog, use the **Retention** insight type and group by signup date. A healthy retention curve "flattens" (rather than dropping to zero) — the flat portion represents your retained power users.

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Track PII in Analytics Events
```typescript
// ❌ Stores email in analytics — GDPR/privacy risk
analytics.userSignedUp(user.email);

// ✅ Track only non-identifying info
analytics.userSignedUp(user.id, "google");
```

### Rule 2: Respect User Privacy Preferences
```typescript
// Check for Do Not Track header and analytics consent
export function initAnalytics() {
  if (
    typeof window !== "undefined" &&
    !window.navigator.doNotTrack &&
    process.env.NEXT_PUBLIC_ENABLE_ANALYTICS === "true"
  ) {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      opt_out_capturing_by_default: false,
      // PostHog automatically handles GDPR compliance in EU
    });
  }
}
```

### Rule 3: Disclose Analytics in Your Privacy Policy
Any user data collected for analytics must be disclosed in your privacy policy. At minimum, your policy should state that you collect usage events, what they contain, and how long you retain them.

### GDPR Consent — Gate Analytics on User Consent

In EU, UK, and California, you must obtain consent before firing analytics events. Initialising PostHog before consent is a GDPR violation that can result in fines.

```typescript
// lib/analytics/consent.ts
export function initAnalyticsWithConsent(userId?: string) {
  // Check if consent was previously given (stored in localStorage)
  const hasConsent = localStorage.getItem("analytics_consent") === "granted"

  if (hasConsent) {
    posthog.opt_in_capturing()
    if (userId) posthog.identify(userId)
  } else {
    posthog.opt_out_capturing()  // ensure not capturing until consent
  }
}

export function grantAnalyticsConsent(userId?: string) {
  localStorage.setItem("analytics_consent", "granted")
  posthog.opt_in_capturing()
  if (userId) posthog.identify(userId)
}

export function revokeAnalyticsConsent() {
  localStorage.setItem("analytics_consent", "denied")
  posthog.opt_out_capturing()
  posthog.reset()  // clear any stored user identity
}
```

```tsx
// components/features/consent/cookie-banner.tsx
// Show on first visit — gate on "analytics_consent" not being set
export function CookieBanner() {
  const [show, setShow] = useState(
    typeof window !== "undefined" && !localStorage.getItem("analytics_consent")
  )

  if (!show) return null

  return (
    <div role="dialog" aria-label="Cookie consent" className="fixed bottom-4 left-4 right-4 md:left-auto md:max-w-sm bg-white dark:bg-gray-800 border rounded-lg p-4 shadow-lg z-50">
      <p className="text-sm text-gray-700 dark:text-gray-300 mb-3">
        We use analytics to improve your experience. No personal data is sold.{" "}
        <a href="/privacy" className="underline">Privacy policy</a>
      </p>
      <div className="flex gap-2">
        <button onClick={() => { grantAnalyticsConsent(); setShow(false) }} className="btn-primary text-sm">
          Accept
        </button>
        <button onClick={() => { revokeAnalyticsConsent(); setShow(false) }} className="btn-secondary text-sm">
          Decline
        </button>
      </div>
    </div>
  )
}
```

**Minimum requirements by region:**
| Region | Requirement |
|---|---|
| EU/EEA | Explicit opt-in before any analytics (GDPR) |
| UK | Same as GDPR (UK GDPR) |
| California | Opt-out mechanism must be visible (CCPA) |
| Rest of world | No legal requirement, but best practice |

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Tracking Everything Before Knowing What Matters
Adding 50 events on day one creates data noise that is harder to interpret than useful signal. You end up with a dashboard full of numbers and no clarity.  
**Fix:** Track the 5–7 core journey events first. Add more only when you have a specific question those events cannot answer.

### ❌ Firing Events on Button Click Instead of on Success
```typescript
// ❌ Fires even if the API call fails
const handleCreate = () => {
  analytics.reminderCreated(userId, props); // fires immediately
  createReminder(data); // might fail
};

// ✅ Fires only after confirmed success
const handleCreate = async () => {
  const result = await createReminder(data);
  if (result.success) {
    analytics.reminderCreated(userId, props); // fires only on success
  }
};
```

### ❌ Interpreting Early Data Before Statistical Significance
Day 3 of an A/B test: variant B has 40% conversion vs A's 20%. You ship variant B. A week later, with more data, both variants are at 22%.  
**Fix:** Never make decisions on less than 100 conversions per variant. Use statistical significance calculators before interpreting results.

### ❌ Tracking the AI Call, Not the User Outcome
You know your AI generation success rate. You do not know whether users found the AI output useful.  
**Fix:** Add an implicit quality signal: track whether users completed the journey after receiving the AI output. A user who creates a reminder after the NLP parsing is an implicit upvote. A user who immediately retries is an implicit downvote.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Analytics

### Add Session Replay
See exactly what users do — clicks, scrolls, confusion moments:
```
PostHog Session Replay: enable in PostHog dashboard
Useful for: diagnosing UX problems, watching confused users, validating A/B test impact
Privacy: configure to mask all input fields automatically
```

### Add Cohort Analysis
Group users by when they signed up or a key behaviour, and compare retention over time:
```
PostHog Cohort: "Users who created a reminder in their first session"
vs "Users who did NOT create a reminder in their first session"
Compare day-7 retention between cohorts.
This isolates the value of activation — the single most actionable insight for growth.
```

### Build a Weekly Product Health Dashboard
```sql
-- Combine your ai_generations DB with PostHog events
-- Run every Monday morning, send to Slack

SELECT
  DATE_TRUNC('week', created_at) as week,
  COUNT(DISTINCT user_id) as weekly_active_users,
  COUNT(*) as total_reminders_created,
  ROUND(COUNT(*) * 1.0 / COUNT(DISTINCT user_id), 1) as reminders_per_user
FROM reminder_events  -- join your events however you store them
WHERE created_at > NOW() - INTERVAL '8 weeks'
GROUP BY 1
ORDER BY 1 DESC;
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Finding the Drop-Off That Explained 40% Churn
```
Funnel built:
  user_signed_up: 100%
  onboarding_completed: 74% (expected — some churn here is normal)
  reminder_created: 41% (PROBLEM — should be closer to 65%)
  reminder_completed: 31%

Investigation: where did the 33% drop between onboarding and first reminder creation go?
Session replay (PostHog) showed: users reaching the dashboard,
looking confused, then leaving. The empty state said "No reminders yet."
No guidance. No examples. No CTA.

Fix: added example prompts and "Create your first reminder" CTA to empty state.

Post-fix funnel:
  user_signed_up: 100%
  onboarding_completed: 73%
  reminder_created: 61% (+20 percentage points)
  reminder_completed: 47%

This single change — discovered through the funnel — drove a 40% improvement
in north star metric (reminders per active user) within 2 weeks.
```

### Case Study 2: EdTech App — A/B Test That Overturned a Strong Opinion
```
Hypothesis: showing lesson difficulty level prominently would
increase lesson completion (users would choose appropriate levels)

Test: Control (no difficulty shown) vs Variant (difficulty shown as "Beginner/Intermediate/Advanced")
Metric: lesson_completed rate after lesson_started

Results after 2 weeks (N=400 per variant):
  Control: 68% completion rate
  Variant: 61% completion rate (statistically significant decline)

Post-hoc analysis (session replay):
  Variant users were selecting "Beginner" even when their profile showed Intermediate level.
  The difficulty label was causing users to self-downgrade and then disengage from "too easy" content.

Decision: Reverted to Control. Removed manual difficulty selection entirely.
Let the AI calibrate to the student's stored level automatically.

The test overturned a confident opinion held by the entire team.
Without measurement, the variant would have been shipped.
```

</real_world_examples>

</skill_document>
