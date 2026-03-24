---
name: growth-distribution
description: Builds SEO, email sequences, push notifications, and referral mechanics. Use after core retention is solid, when building re-engagement systems, when setting up reminder notifications, or when adding organic acquisition.
---
# Skill: Growth & Distribution

```json
{
  "skill_id": "growth-distribution",
  "category": "Product & Non-Technical",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Growth & Distribution — Getting Users to Your AI Product and Bringing Them Back</title>

<overview>

## What This Skill Enables
- Implement the technical foundations of growth: SEO, push notifications, email sequences, and referral mechanics
- Build re-engagement systems that bring users back to your AI product without constant manual marketing effort
- Set up the notification infrastructure that makes AI reminders, learning nudges, and progress updates actually reach users
- Understand which growth levers matter most for AI products specifically

## Why It Matters for Vibe Coders
Building the product is only half the challenge. Getting users to find it, use it repeatedly, and tell others about it requires deliberate growth mechanics. For AI products — where the core value often depends on repeated use (a reminder app that you use once is useless; an AI tutor that you come back to daily becomes indispensable) — retention mechanics are product features, not marketing activities.

## When to Use This Skill
- After your core features are working and you are ready to acquire real users
- When users sign up but do not return after day 1
- When your notification system needs to be built or improved
- When you want to add SEO to a public-facing part of your app

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "app_type": "[REPLACE: e.g. web app / mobile / both]",
    "target_audience": "[REPLACE: e.g. adult professionals / students / busy parents]",
    "primary_acquisition_channel": "[REPLACE: e.g. organic search / word of mouth / social / unknown]",
    "current_day7_retention": "[REPLACE: e.g. 15% / unknown]",
    "notification_channels_in_use": "[REPLACE: e.g. email only / email + push / none yet]",
    "growth_task_today": "[REPLACE: e.g. set up reminder email notifications / add SEO metadata / build referral flow]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Growth for AI Products

### Mental Model 1: Retention Before Acquisition
The biggest growth mistake is spending on user acquisition before fixing retention. If 70% of your users leave after day 1, acquiring more users just runs them through a leaky bucket.

**Fix the bucket before filling it:**
```
Retention loop:
  User experiences value → comes back → experiences more value → tells others

If day-1 retention is below 20%: fix onboarding and first-value experience first.
If day-7 retention is below 10%: fix core product loop and notifications first.
Only then: invest in acquisition.
```

### Mental Model 2: AI-Specific Re-engagement Levers
AI products have unique re-engagement mechanics that traditional apps do not:

| Lever | How It Works | AI Product Example |
|---|---|---|
| **Scheduled output delivery** | AI generates content on a schedule and pushes it to users | "Your weekly learning summary is ready" |
| **Triggered notifications** | AI recognises a moment where a user would benefit from returning | "You have 3 reminders due tomorrow" |
| **Progress milestones** | AI tracks what users have done and celebrates it | "You completed 5 lessons this week!" |
| **Personalised content pull** | AI creates personalised content that draws users back | "We found 3 new lessons based on your interests" |

### Mental Model 3: The Notification Permission Triangle
For push notifications and email to work, three things must all be true:

```
1. USER GRANTS PERMISSION — they said yes when asked
2. CONTENT IS VALUABLE — the notification is actually useful, not spam
3. TIMING IS RIGHT — the notification arrives when it is actionable

If ANY of these fail: users disable notifications, defeating the entire system.
```

The most common mistake: getting permission (1) but failing at content (2) or timing (3). Result: users revoke permission and you lose the channel permanently.

</mental_models>

---

<system_design_breakdown>

## Growth Stack for AI Products

```
ACQUISITION (getting new users)
  SEO → /app/sitemap.ts + metadata + structured data
  Content → AI-generated public landing pages (e.g. /topics/fractions)
  Referral → invite link with attribution tracking

  ↓

ACTIVATION (first value experience)
  Onboarding → guided first action (see skill-user-experience-design.md)
  Welcome email → sent immediately after sign-up
  First AI generation → makes the product feel magical on day 1

  ↓

RETENTION (bringing users back)
  Scheduled emails → weekly digest / progress summary
  Push notifications → time-sensitive reminders, nudges
  In-app badges → streaks, milestones, achievements

  ↓

REFERRAL (users bringing other users)
  Share output → share a lesson / share a reminder
  Invite link → referral code + credit system
  Social proof → public achievements or content
```

## Notification Infrastructure

```
TRIGGER         CHANNEL       TIMING          CONTENT
──────────────────────────────────────────────────────
Reminder due    Push + Email  Scheduled time   "Time to [task]"
Lesson ready    Email         Weekly Monday    "Your lesson on X is ready"
Streak at risk  Push          Evening          "Keep your 5-day streak!"
Progress hit    Email         On milestone     "You completed 10 lessons!"
Inactivity      Email         Day 3 + Day 7    "Come back" with new content
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Fix retention before acquisition. Build one notification type at a time. -->

## Step 1 — Add SEO Metadata to Public Pages

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add SEO metadata to my Next.js app.
Public pages that need SEO: [LIST — e.g. landing page, pricing, public lesson previews]

For each page, generate:
1. Metadata using Next.js generateMetadata function:
   - title: [page-specific title]
   - description: [1-2 sentence description for search snippets]
   - openGraph: title, description, image, url
   - twitter: card, title, description, image
2. Canonical URL to prevent duplicate content
3. robots: index for public pages, noindex for app/dashboard pages

Also generate:
- /app/sitemap.ts — lists all public URLs with lastModified dates
- /app/robots.ts — allows search engines, disallows /app/ and /api/ paths

My domain: [YOUR DOMAIN]
```

**Verify:** Run `curl https://yourapp.com` and check the `<head>` section for meta tags. Check `https://yourapp.com/sitemap.xml` returns your page list.

---

## Step 2 — Build the Welcome Email Sequence

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Build a 3-email welcome sequence triggered after user_signed_up event.
Email service: Resend (from skill-api-design-integration.md)

Email 1: Welcome (send immediately after sign-up)
- Subject: "Welcome to [App Name] — here's what to do first"
- Content: what the app does in 2 sentences + one clear CTA to the core action

Email 2: Tips (send 24 hours later, only if user has NOT done the core action)
- Subject: "Quick tip for [App Name]"
- Content: one tip that helps them get value faster + CTA back to the app

Email 3: Check-in (send 3 days later, only if user is still not active)
- Subject: "Everything okay? Here's a shortcut"
- Content: acknowledge they might be busy + offer a simpler starting point

Implementation:
- Use Inngest to schedule the sequence (see skill-agentic-workflows.md)
- Check event data to skip emails for users who are already active
- Include unsubscribe link in every email (required by law)
- Track: email_opened, email_clicked events via Resend webhooks
```

**Verify:** Create a test account. Confirm Email 1 arrives immediately. Confirm Email 2 is not sent if you complete the core action before 24 hours.

---

## Step 3 — Build Reminder/Notification Delivery

The core growth mechanic for time-sensitive AI products.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Build the notification delivery system for [REMINDER TYPE].

When a reminder's scheduledAt time arrives:
1. Background job triggers (Inngest cron every 5 minutes)
2. Query: reminders where scheduledAt <= NOW() AND is_notified = false AND deleted_at IS NULL
3. For each pending reminder:
   a. Send email notification via Resend (always — email is the reliable baseline)
   b. Send push notification if user has web push enabled (optional enhancement)
   c. Update: is_notified = true, notified_at = NOW()
4. Handle: notification send failure (retry once, then mark as failed but do not block others)

Also build: GET /api/notifications/push-subscribe
- Accepts push subscription object from browser
- Stores in user_push_subscriptions table

Environment: the Inngest job runs on its own schedule — not tied to a user request.
```

**Verify:** Create a reminder scheduled for 2 minutes from now. Wait. Confirm the email arrives within 3 minutes of the scheduled time.

---

## Step 4 — Build a Simple Referral System

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Build a lightweight referral system.

Feature: each user gets a unique referral link
URL pattern: https://[app]/invite/[REFERRAL_CODE]

When someone clicks a referral link:
1. Store the referral code in a cookie (expires in 30 days)
2. Show the landing page normally (do not redirect to sign-up — let them explore first)

When someone signs up:
1. Check for referral code cookie
2. If found: record in referrals table: { referrer_user_id, referred_user_id, created_at }
3. If referrer should receive credit: queue a reward (define what the reward is: [DESCRIBE])

Dashboard: add a simple "Invite friends" section showing:
- Their referral link (copyable)
- How many people they have referred
- What reward they will receive

No complex multi-level system — flat, one-hop referrals only.
```

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on growth and distribution for my app.
Project context: [PASTE context_anchor JSON]

Current retention: [DAY-1 / DAY-7 rates if known]
Notification channels: [WHAT IS ALREADY SET UP]
Today I am building: [ONE GROWTH MECHANIC]

Before writing code:
1. Confirm this is the right thing to work on (fix retention before acquisition)
2. List the files that will be created or modified
3. Confirm the notification channel is already set up (Resend, Inngest, etc.)
```

### SEO Metadata Prompt
```
Add production-ready SEO to my Next.js app.
Project context: [PASTE context_anchor JSON]

Public pages: [LIST — e.g. /, /pricing, /features]
App pages (no index): [LIST — e.g. /dashboard, /lessons]

Generate:
1. generateMetadata() for each public page
   Include: title, description, OG tags, Twitter card
2. Default metadata in layout.tsx (fallback for any page without specific metadata)
3. /app/sitemap.ts — dynamic sitemap including all public pages
4. /app/robots.ts — allow public pages, block /app/ and /api/

My domain: [DOMAIN]
Primary keyword for each page: [DESCRIBE]
```

### Email Re-engagement Prompt
```
Build a re-engagement email for users who have not returned in [X days].
App: [APP NAME], Email service: Resend

Target: users who signed up, used the app at least once, but have not been active for [X] days

Email spec:
- Subject line: [3 options to A/B test]
- Opening: acknowledge absence naturally, do not be passive-aggressive
- Body: show what is new or what they left behind
  (e.g. "You have [N] reminders waiting" or "3 new lesson topics added since you last visited")
- CTA: single, clear action (not multiple options)
- Tone: [DESCRIBE YOUR APP'S TONE]

Implementation:
- Trigger via Inngest: daily job finds users inactive for exactly [X] days
- Do not send to users who have already received a re-engagement email in the last 14 days
- Track: re_engagement_email_sent, re_engagement_clicked, re_engagement_converted
```

### Push Notification Setup Prompt
```
Add web push notifications to my Next.js app.
Project context: [PASTE context_anchor JSON]

Purpose: notify users when their reminders are due

Implementation:
1. Service worker: /public/sw.js — handles push events, shows notification
2. Push subscription: /app/api/push-subscribe/route.ts — saves subscription to DB
3. Notification sender: /lib/push.ts — sends push via Web Push API
4. UI: a small "Enable notifications" prompt that appears after user creates their first reminder

VAPID keys: generate a VAPID key pair
  Add: VAPID_PUBLIC_KEY (NEXT_PUBLIC_), VAPID_PRIVATE_KEY to .env.example

Permission request: ask AFTER first value moment (after reminder is created), not on first visit.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is SEO and does my AI app need it?"

SEO (Search Engine Optimisation) determines whether your app appears when people search for it on Google. For AI products, it matters differently depending on what you are building:

**SEO matters a lot if:**
- You have public content (e.g. publicly accessible lesson pages, topic pages)
- People search for the problem your app solves ("AI reminder app", "personalised math tutor")
- You want organic acquisition without paid advertising

**SEO matters less if:**
- Your app is invite-only or behind a login wall
- You are growing through word of mouth, communities, or direct sales
- You are in a niche where your target users are not searching Google for a solution

**The minimum:** Even if you are not optimising for SEO, add basic metadata (`<title>`, `<meta description>`, Open Graph tags) to your landing page. This affects how your app appears when shared on Slack, LinkedIn, or iMessage — which is how word-of-mouth growth spreads.

---

### "Email vs push notifications — which should I build first?"

**Email first, always.**

Email is:
- Universal (everyone has email, not everyone has enabled push)
- Reliable (emails arrive; push notifications get blocked or ignored)
- Acceptable delay (a 5-minute delivery delay is fine for email; not for time-sensitive push)
- Legal (required for certain notifications by law — email has an unsubscribe mechanism)

**Add push notifications when:**
- Your core feature is time-sensitive (reminders, real-time alerts)
- Your users are mobile-first
- Email engagement is plateauing

**The sequence:** Welcome email → transactional emails → engagement emails → then push notifications if metrics justify it.

---

### "My users sign up but 80% never come back after day 1. What is the most impactful fix?"

This is a retention problem, not a growth problem. The fix is almost always one of three things:

1. **The first session did not deliver value.** The user signed up but never experienced the "aha moment." Fix: make the core action faster and more guided. See `skill-user-experience-design.md`.

2. **No re-entry trigger.** Users need a reason to return. If nothing pulls them back (no reminder email, no notification, no scheduled output), they forget about the app. Fix: build a welcome email sequence with a Day-1 re-engagement email.

3. **The core loop is not compelling enough.** The product delivers value once but not repeatedly. Fix: add recurring value delivery (weekly digests, scheduled AI content, streaks).

Address these in order. Most day-1 churn problems are problem #1. Fix that before building elaborate notification systems.

---

### 🗂️ Update Your AGENT_CONTEXT.md

```md
## Growth & Distribution
- Referral system: nanoid codes in Referral table — lib/growth/referral.ts
- Referral attribution: tracked via cookie → webhook on user creation
- Email sequences: Inngest — 3-email welcome sequence in lib/jobs/welcome-sequence.ts
- Notification frequency caps: Upstash Redis — 3 emails/day, 2 push/day — lib/notifications/frequency-cap.ts
- SEO: generateMetadata in each page — OG images via /api/og (Vercel OG)
- Unsubscribe: one-click unsubscribe in every email — legal requirement
- Re-engagement: scheduled digest cron — /api/cron/digest
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing Your Growth Systems

### Notification Delivery Test
```bash
# Test reminder notification end-to-end
# 1. Create a reminder scheduled for 5 minutes from now
# 2. Wait for Inngest cron to run (every 5 minutes)
# 3. Check email inbox — should arrive within 5 minutes of scheduled time
# 4. Check DB: is_notified should be true, notified_at should be set

# Test failure handling:
# 1. Create a reminder with an invalid email address
# 2. Confirm: job continues (does not crash on one failure)
# 3. Confirm: the failure is logged
# 4. Confirm: other reminders in the same batch are still sent
```

### SEO Verification
```bash
# Check metadata renders correctly
curl -s https://yourapp.com | grep -E '<title>|<meta name="description"'
# Should show your title and description meta tags

# Check sitemap
curl https://yourapp.com/sitemap.xml
# Should return XML listing your public pages

# Check robots.txt
curl https://yourapp.com/robots.txt
# Should allow public pages, disallow /app/ and /api/

# Test Open Graph (paste URL into opengraph.xyz)
# Verifies how your page looks when shared on social media / Slack
```

### Email Sequence Testing
```
1. Create a test email address (use + aliases: you+test1@gmail.com)
2. Sign up with that address
3. Verify: welcome email arrives within 2 minutes

To test conditional sending (Email 2 only for inactive users):
4. Do NOT complete the core action
5. Wait 24+ hours (or manually trigger the Inngest function)
6. Verify: Email 2 arrives

7. Sign up with another test address
8. Complete the core action immediately
9. Wait 24+ hours
10. Verify: Email 2 does NOT arrive (user is active)
```

### Common Growth Issues

| Issue | Symptom | Fix |
|---|---|---|
| Emails going to spam | Low open rates, users report not receiving emails | Set up SPF, DKIM, DMARC records at your domain DNS |
| Push notifications not delivered | No delivery, no errors in logs | Check VAPID keys are correct; verify service worker is registered |
| Referral code not tracked | Users say they used a referral link but it was not attributed | Check cookie is being set and read before sign-up completion |
| Re-engagement emails sent to active users | Users complain about irrelevant emails | Add active user check before queuing re-engagement email |
| Notifications at wrong time | Users in different timezones receive 3am notifications | Store user timezone at sign-up; schedule in user's local time |

</testing_and_qa>

---

<common_patterns>

## Reusable Growth Patterns

### Pattern 1: Next.js SEO Metadata
```typescript
// /app/layout.tsx — default metadata for all pages
export const metadata: Metadata = {
  metadataBase: new URL("https://yourapp.com"),
  title: {
    default: "RemindAI — Smart Reminders That Understand Plain English",
    template: "%s | RemindAI",
  },
  description: "Create reminders by typing naturally. RemindAI understands when you say 'call mum Sunday at 3pm'.",
  openGraph: {
    type: "website",
    locale: "en_US",
    url: "https://yourapp.com",
    siteName: "RemindAI",
  },
  twitter: { card: "summary_large_image" },
  robots: { index: true, follow: true },
};

// /app/(app)/dashboard/page.tsx — override for app pages
export const metadata: Metadata = {
  robots: { index: false, follow: false }, // Don't index app pages
};
```

### Pattern 2: Inngest Email Sequence
```typescript
// /lib/jobs/welcome-sequence.ts
import { inngest } from "@/lib/inngest";

// Trigger when user signs up
export const welcomeSequence = inngest.createFunction(
  { id: "welcome-email-sequence" },
  { event: "user/signed_up" },
  async ({ event, step }) => {
    const { userId, email, name } = event.data;

    // Email 1: Immediate welcome
    await step.run("send-welcome-email", async () => {
      await sendEmail({ to: email, template: "welcome", data: { name } });
    });

    // Wait 24 hours
    await step.sleep("wait-24h", "24h");

    // Email 2: Only if user has not completed core action
    await step.run("send-tips-if-inactive", async () => {
      const isActive = await checkUserActivated(userId);
      if (!isActive) {
        await sendEmail({ to: email, template: "tips", data: { name } });
      }
    });

    // Wait 3 more days (total: 4 days after sign-up)
    await step.sleep("wait-3d", "3d");

    // Email 3: Only if still inactive
    await step.run("send-checkin-if-still-inactive", async () => {
      const isActive = await checkUserActivated(userId);
      if (!isActive) {
        await sendEmail({ to: email, template: "checkin", data: { name } });
      }
    });
  }
);
```

### Pattern 3: Reminder Notification Cron
```typescript
// /lib/jobs/reminder-notifier.ts
export const reminderNotifier = inngest.createFunction(
  { id: "reminder-notifier", concurrency: { limit: 5 } },
  { cron: "*/5 * * * *" }, // Every 5 minutes
  async ({ step }) => {
    const pendingReminders = await step.run("fetch-pending", async () => {
      return db.reminder.findMany({
        where: {
          scheduledAt: { lte: new Date() },
          isNotified: false,
          deletedAt: null,
        },
        include: { user: { select: { email: true, name: true } } },
        take: 50, // Process max 50 per run
      });
    });

    await Promise.allSettled(
      pendingReminders.map((reminder) =>
        step.run(`notify-${reminder.id}`, async () => {
          try {
            await sendReminderEmail({
              to: reminder.user.email,
              name: reminder.user.name,
              task: reminder.task,
              scheduledAt: reminder.scheduledAt,
            });
            await db.reminder.update({
              where: { id: reminder.id },
              data: { isNotified: true, notifiedAt: new Date() },
            });
          } catch (error) {
            console.error(`Failed to notify reminder ${reminder.id}:`, error);
            // Log but do not rethrow — let other reminders process
          }
        })
      )
    );

    return { processed: pendingReminders.length };
  }
);
```

### Pattern 4: Referral Link Generation
```typescript
// /lib/referral.ts
import { nanoid } from "nanoid";

export async function getOrCreateReferralCode(userId: string): Promise<string> {
  const existing = await db.user.findUnique({
    where: { id: userId },
    select: { referralCode: true },
  });

  if (existing?.referralCode) return existing.referralCode;

  const code = nanoid(8); // e.g. "k2Mn9pQr"
  await db.user.update({
    where: { id: userId },
    data: { referralCode: code },
  });
  return code;
}

export function getReferralUrl(code: string): string {
  return `${process.env.NEXT_PUBLIC_APP_URL}/invite/${code}`;
}
```

### Pattern 5: Referral Attribution Tracking — Complete the Referral Loop

Generating referral links is only half the pattern. You also need to track when a referred user converts and attribute the conversion to the referrer.

```typescript
// middleware.ts — capture referral codes on landing
export function middleware(request: NextRequest) {
  const { searchParams } = request.nextUrl
  const refCode = searchParams.get("ref")

  if (refCode) {
    const response = NextResponse.next()
    // Store in cookie for 30 days — survives navigation and signup flow
    response.cookies.set("referral_code", refCode, {
      maxAge: 30 * 24 * 60 * 60,
      httpOnly: true,
      sameSite: "lax",
    })
    return response
  }
}

// app/api/auth/webhook/route.ts — Clerk/Supabase webhook on user creation
export async function POST(req: Request) {
  const event = await req.json()

  if (event.type === "user.created") {
    const userId = event.data.id

    // Get referral code from user's session cookie
    // (Passed via webhook metadata or looked up from a pending_referrals table)
    const referralCode = await getPendingReferralCode(userId)

    if (referralCode) {
      // Find the referrer
      const referral = await db.referral.findFirst({
        where: { code: referralCode, convertedAt: null }
      })

      if (referral) {
        // Attribute the conversion
        await db.referral.update({
          where: { id: referral.id },
          data: {
            convertedUserId: userId,
            convertedAt: new Date()
          }
        })

        // Trigger reward for referrer (e.g., extend trial, add credits)
        await inngest.send({
          name: "referral/converted",
          data: { referrerId: referral.userId, convertedUserId: userId }
        })
      }
    }
  }
}
```

### Pattern 6: Open Graph and Twitter Card Tags — Rich Social Link Previews

Social sharing is one of the highest-leverage distribution channels for AI products. Without OG tags, links shared on Twitter/X, LinkedIn, Slack, and iMessage show as plain text. With them, they show a rich preview card.

```typescript
// app/layout.tsx — default metadata (inherited by all pages)
export const metadata: Metadata = {
  title: {
    default: "YourProduct — AI-powered [description]",
    template: "%s | YourProduct",  // page-specific titles
  },
  description: "One sentence that explains the value proposition.",
  openGraph: {
    type: "website",
    siteName: "YourProduct",
    title: "YourProduct — AI-powered [description]",
    description: "One sentence value proposition.",
    images: [{
      url: "https://yourproduct.com/og-image.png",  // 1200×630px
      width: 1200,
      height: 630,
      alt: "YourProduct",
    }],
  },
  twitter: {
    card: "summary_large_image",  // shows large image preview
    creator: "@yourhandle",
    images: ["https://yourproduct.com/og-image.png"],
  },
}

// app/(marketing)/[feature]/page.tsx — dynamic per-page OG image
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const feature = await db.feature.findUnique({ where: { slug: params.slug } })

  return {
    title: feature?.name,
    openGraph: {
      title: `${feature?.name} | YourProduct`,
      description: feature?.description,
      images: [{
        // Dynamic OG image via Vercel OG:
        url: `/api/og?title=${encodeURIComponent(feature?.name ?? "")}`,
        width: 1200,
        height: 630,
      }]
    }
  }
}
```

```typescript
// app/api/og/route.tsx — dynamic OG image generator (Vercel OG)
import { ImageResponse } from "next/og"

export const runtime = "edge"

export async function GET(req: Request) {
  const { searchParams } = new URL(req.url)
  const title = searchParams.get("title") ?? "YourProduct"

  return new ImageResponse(
    <div style={{ display: "flex", width: "100%", height: "100%", background: "#0f172a", alignItems: "center", justifyContent: "center", padding: 60 }}>
      <h1 style={{ color: "white", fontSize: 60, fontWeight: "bold" }}>{title}</h1>
    </div>,
    { width: 1200, height: 630 }
  )
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Every Email Must Have an Unsubscribe Link
CAN-SPAM (US), GDPR (EU), and CASL (Canada) all require transactional and marketing emails to include a one-click unsubscribe mechanism. Resend includes this automatically in its email components — do not bypass it.

### Rule 2: Honour Unsubscribes Immediately
```typescript
// When a user unsubscribes:
await db.user.update({
  where: { id: userId },
  data: { emailOptOut: true, emailOptOutAt: new Date() },
});

// Before sending any email, check this flag:
if (user.emailOptOut) {
  console.log("Skipping email — user has opted out");
  return;
}
```

### Rule 3: Never Purchase Email Lists or Third-Party User Data
Only email users who explicitly signed up for your product. Purchased lists violate anti-spam laws, damage your email sender reputation, and often result in your email domain being blacklisted.

### Rule 4: Verify Push Notification Permissions Before Sending
```typescript
// Always check permission status before attempting to send push
if (Notification.permission === "granted") {
  // Safe to send
} else if (Notification.permission === "denied") {
  // User explicitly blocked — do not ask again
  // Fall back to email only
}
```

### Notification Fatigue Prevention

Sending too many notifications causes unsubscribes and churn. Enforce frequency caps at the application layer:

```typescript
// lib/notifications/frequency-cap.ts
import { Redis } from "@upstash/redis"

const redis = Redis.fromEnv()

type NotificationChannel = "email" | "push" | "sms"
type NotificationFrequency = "daily" | "weekly"

// Check if a user has received too many notifications recently
export async function isNotificationCapped(
  userId: string,
  channel: NotificationChannel,
  limit: number,
  window: NotificationFrequency
): Promise<boolean> {
  const windowSeconds = window === "daily" ? 86400 : 604800
  const key = `notif:${channel}:${userId}:${window}`

  const count = await redis.incr(key)
  if (count === 1) {
    await redis.expire(key, windowSeconds)
  }

  return count > limit
}

// Usage before sending any notification:
export async function sendNotificationIfAllowed(
  userId: string,
  notification: { subject: string; body: string; channel: "email" }
) {
  const isCapped = await isNotificationCapped(userId, "email", 3, "daily")  // max 3 emails/day

  if (isCapped) {
    console.log(`[notification:capped] userId=${userId} — skipping to prevent fatigue`)
    return { sent: false, reason: "frequency_cap" }
  }

  // Send notification...
  return { sent: true }
}
```

**Recommended frequency caps:**
| Channel | Max per day | Max per week | Notes |
|---|---|---|---|
| Email (transactional) | 3 | — | Order confirmations, password resets |
| Email (marketing) | 1 | 3 | Welcome sequences, digests |
| Push notification | 2 | 7 | Never push between 10pm–8am |
| In-app notification | Uncapped | — | User controls visibility |

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Building Growth Systems Before Fixing Retention
You build a beautiful referral system. Users share their referral link. New users arrive. 80% leave after day 1. The referral system has no effect on growth because the product loop is broken.  
**Fix:** Day-1 retention above 25% before investing in acquisition. Day-7 retention above 10% before investing in referral mechanics.

### ❌ Asking for Push Notification Permission on First Visit
The browser shows a permission dialog immediately when a new visitor arrives. They have no context for why the app would need this. They click "Block." You permanently lose the push channel for this user.  
**Fix:** Ask for push permission only after the user has experienced value (after their first reminder is created, after their first lesson is completed). The right moment is when they can see why notifications would help them.

### ❌ Sending Notifications That Are Not Genuinely Useful
```
❌ "We miss you! Come back to RemindAI" (no specific value, just guilt)
✅ "Your reminder to call mum is in 30 minutes" (immediate, specific value)
✅ "You completed 5 lessons this week — your streak is on 🔥" (positive milestone)
```

Every notification should either: remind of something time-sensitive, inform of a meaningful milestone, or surface new personalised content. If it does not do one of these three things, do not send it.

### ❌ No Timezone Handling for Scheduled Notifications
You schedule a reminder for "9am." You send it at 9am UTC. The user in Mumbai receives a notification at 2:30pm. The user in New York receives it at 4am.  
**Fix:** Store user timezone during sign-up (or detect from browser). Convert all scheduled times to UTC using the user's timezone before storing. Display and send in local time.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Growth Systems

### Add AI-Powered Re-engagement
Instead of generic "come back" emails, use AI to generate personalised re-engagement:
```typescript
// When composing a re-engagement email:
const personalised = await generateText(
  `User: ${userName}. Last active: ${daysInactive} days ago.
   Last action: created a reminder titled "${lastReminderTitle}".
   Write one sentence of personalised re-engagement copy. Be warm, not pushy.`,
  "You write brief, genuine re-engagement copy for a reminder app."
);
// Include this in the email template
```

### Add Email Performance Tracking
Use Resend webhooks to track open rates, click rates, and conversions per email type:
```typescript
// /app/api/webhooks/resend/route.ts
// Events: email.delivered, email.opened, email.clicked, email.bounced
// Track these in your analytics tool (PostHog) or custom events table
// Use click rates to identify which email subjects perform best
```

### Build a Content Loop for SEO
For EdTech apps, generate public landing pages for popular topics:
```
/topics/[slug] → server-rendered page with:
  - SEO metadata for the topic
  - A free preview of an AI-generated lesson
  - CTA to sign up for the full experience

These pages:
  - Get indexed by Google for topic-specific searches
  - Convert curious searchers into sign-ups
  - Can be generated automatically for your entire curriculum
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Email Sequence That Doubled Day-7 Retention
```
Before email sequence: day-7 retention = 11%
After 3-email welcome sequence: day-7 retention = 23%

The sequence:
  Email 1 (immediate): "Your first reminder is waiting — tap to get started"
    → 61% open rate (immediate, contextual)

  Email 2 (24h, inactive only): "Quick tip: try saying 'every Monday at 8am'"
    → 34% open rate, 18% click-through
    → 42% of clickers created their first reminder

  Email 3 (day 4, still inactive): "3 popular reminders to steal"
    → 22% open rate
    → showed real examples from other users

The highest leverage insight: Email 2 sent only to inactive users.
Sending it to everyone would have annoyed active users and reduced open rates.
Conditional sending was the key technical decision.
```

### Case Study 2: EdTech App — SEO Pages That Generated 34% of New Sign-Ups
```
Strategy: create public-facing pages for every topic in the curriculum
Format: /topics/[topic-slug] → free lesson preview + sign-up CTA

Implementation:
  1. Generated 127 topic pages from the curriculum database
  2. Each page: SEO metadata, 200-word free lesson preview, "Get the full lesson" CTA
  3. Pages indexed by Google over 6 weeks

Results after 3 months:
  Organic search traffic: 0 → 2,400 monthly visitors
  Sign-ups from SEO pages: 34% of total new sign-ups
  Cost: $0 (no paid acquisition)
  Time to implement: one sprint (the page template + sitemap update)

The highest-value topic pages:
  /topics/fractions → 340 monthly visitors (parents searching "teach fractions")
  /topics/algebra-basics → 290 visitors
  /topics/photosynthesis → 210 visitors

These were not randomly chosen — they were the topics with the highest
lesson completion rates in the app, which correlated with the highest
search volume for educational content.
```

</real_world_examples>

</skill_document>
