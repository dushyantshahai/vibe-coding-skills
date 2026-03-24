# Skill: Product Documentation

```json
{
  "skill_id": "product-documentation",
  "category": "Product & Non-Technical",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Product Documentation — PRDs, Tech Specs, and API Docs That Keep AI Agents Grounded</title>

<overview>

## What This Skill Enables
- Write a Product Requirements Document (PRD) that gives your AI coding agent the context it needs to build the right thing — not just something that runs
- Create lightweight technical specs that capture decisions before you build, preventing costly rewrites
- Generate API documentation that both human developers and AI agents can use reliably to understand your backend

## Why It Matters for Vibe Coders
Documentation is the memory of your product. Without it, every AI coding session starts from scratch — the agent has no idea what was decided last week, why a particular approach was chosen, or what the scope boundaries are. Good documentation is not bureaucracy; it is the context injection that makes your AI agent sessions dramatically more productive. A 30-minute PRD prevents 3 hours of misaligned code.

## When to Use This Skill
- Before starting any significant new feature — write a one-page spec first
- When your AI agent keeps building slightly the wrong thing despite good prompts
- When you need to hand off work to another developer (human or AI) and need them to understand the context
- When planning a new product and need to think through requirements systematically

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "app_type": "[REPLACE: e.g. EdTech web app / Conversational reminder mobile app]",
    "document_type": "[REPLACE: PRD / tech spec / API spec / architecture doc]",
    "feature_or_scope": "[REPLACE: what is this document about?]",
    "target_audience": "[REPLACE: e.g. AI coding agent / human developer / stakeholder review]",
    "existing_docs": "[REPLACE: list any existing PRDs or specs this relates to]",
    "tech_stack": "[REPLACE: e.g. Next.js, Supabase, OpenAI, Clerk]",
    "current_build_phase": "[REPLACE: e.g. MVP / v2 feature / architecture review]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Product Documentation

### Mental Model 1: Documentation as Context Injection
Your AI coding agent has no persistent memory. Every session, it starts from zero. A well-written PRD is a context injection tool — it answers the questions the agent would otherwise ask or assume wrong:

- What problem does this solve?
- Who is the user and what do they need?
- What are the constraints (timeline, tech, scope)?
- What does success look like?
- What is explicitly out of scope?

A PRD you paste at the start of a session transforms a generic AI into a product-aware collaborator.

### Mental Model 2: The Right Document for the Right Purpose

| Document | When to Write | Length | Audience |
|---|---|---|---|
| **PRD** (Product Requirements) | Before starting a feature | 1–2 pages | AI agent, yourself, stakeholders |
| **Tech Spec** | Before writing complex code | 1 page | AI coding agent |
| **API Spec** | After routes are designed | Varies | Frontend devs, AI agent |
| **Architecture Diagram** | When designing the system | 1 diagram | Everyone |
| **Decision Log** | When making a non-obvious choice | A few lines | Future you |

### Mental Model 3: The Minimum Viable PRD
You do not need a 20-page document. You need five questions answered precisely:

```
1. PROBLEM: What user problem are we solving? (One sentence)
2. WHO: Who is the user experiencing this problem?
3. WHAT: What will the feature do? (Bullet list of capabilities)
4. HOW WE KNOW IT WORKS: What does success look like? (Measurable if possible)
5. NOT IN SCOPE: What are we explicitly NOT building? (Prevents scope creep)
```

That is enough for most features. Add more detail only when the feature is complex enough to require it.

</mental_models>

---

<system_design_breakdown>

## Document Templates

### Template 1: Minimum Viable PRD
```markdown
# PRD: [Feature Name]
**Status:** Draft | In Review | Approved
**Owner:** [Your name]
**Last Updated:** [Date]

## Problem
[One sentence: What user pain does this solve? Be specific.]

## User
[Who experiences this problem? What context are they in when they need this?]

## Solution
What we are building:
- [Capability 1]
- [Capability 2]
- [Capability 3]

What we are NOT building (this sprint):
- [Explicitly excluded item 1]
- [Explicitly excluded item 2]

## Success Criteria
- [ ] [Measurable outcome 1 — e.g. "User can create a reminder in under 30 seconds"]
- [ ] [Measurable outcome 2]

## Technical Notes
- Stack: [Relevant stack decisions]
- Dependencies: [What must exist before this can be built]
- Risks: [Anything that could block or complicate this]

## Open Questions
- [Question that needs answering before or during build]
```

---

### Template 2: Tech Spec (for Complex Features)
```markdown
# Tech Spec: [Feature Name]
**PRD Reference:** [Link to PRD]
**Status:** Draft | Approved

## Approach
[One paragraph: how are we solving this technically? Why this approach over alternatives?]

## Data Model Changes
[New tables, modified columns, new indexes — reference skill-database-storage.md]

## API Changes
[New routes, modified routes — use the contract format from skill-api-routes.md]

## AI Integration
[If applicable: which model, what the prompt does, expected input/output]

## Incremental Build Order
1. [First thing to build and verify]
2. [Second thing — depends on #1]
3. [Third thing — depends on #2]

## Definition of Done
- [ ] All routes return expected responses
- [ ] Tests written (see skill-testing-strategy.md)
- [ ] Deployed to preview environment and verified
- [ ] No new console errors in production logs
```

---

### Template 3: API Documentation
```markdown
# API Reference: [Feature Area]
**Base Path:** /api/[feature]
**Authentication:** Required (Bearer token via [Auth Provider])

## Endpoints

### POST /api/[feature]/create
**Purpose:** [One sentence]

**Request Body:**
| Field | Type | Required | Description |
|---|---|---|---|
| field1 | string | Yes | [Description] |
| field2 | string | No | [Description, default value] |

**Success Response (201):**
```json
{
  "success": true,
  "data": { "[field]": "[description]" }
}
```

**Error Responses:**
| Status | Error Code | When |
|---|---|---|
| 400 | VALIDATION_ERROR | Required field missing or invalid |
| 401 | UNAUTHORIZED | No valid auth token |
| 500 | INTERNAL_ERROR | Server-side failure |
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Write the PRD before any code. Write the tech spec before complex code. Document APIs after they are built. -->

## Step 1 — Write the PRD Before Building

Use this as the first step of every new feature, before opening any code files.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Help me write a PRD for this feature: [DESCRIBE FEATURE IN PLAIN ENGLISH]

I need:
1. A precise one-sentence problem statement
2. User description (who has this problem, in what context)
3. Feature capabilities list (what it does — keep to MVP essentials)
4. Explicit out-of-scope list (prevent scope creep)
5. 2-3 measurable success criteria
6. Technical dependencies (what must already exist)
7. Risks or open questions

Use the Minimum Viable PRD template.
Ask me clarifying questions if anything is ambiguous before writing.
```

**Verify:** Read the PRD aloud in under 3 minutes. If you cannot, it is too long. Show it to someone unfamiliar with the project — can they understand what will be built?

---

## Step 2 — Write the Tech Spec for Complex Features

For features involving new database tables, multiple API routes, or AI integration — write a tech spec before touching code.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
PRD: [PASTE YOUR PRD]

Write a tech spec for this feature.
Include:
1. Technical approach (one paragraph — why this way vs alternatives)
2. Data model changes (new tables or columns needed)
3. New API routes (use route contract format from skill-api-routes.md)
4. AI integration details (if applicable)
5. Incremental build order (numbered steps, each verifiable before the next)
6. Definition of done checklist

Keep it to one page. This is not architecture documentation — it is a build guide.
```

**Verify:** Can you hand this spec to your AI coding agent at the start of a session and have it understand exactly what to build? If yes, the spec is ready.

---

## Step 3 — Generate API Documentation After Building

After routes are built and tested, generate documentation automatically.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Generate API documentation for these routes:
[LIST ROUTE FILES — e.g. /app/api/reminders/create/route.ts]

For each route, extract and document:
1. Method and path
2. Authentication requirement
3. Request body schema (field, type, required/optional, description)
4. Success response shape (status code + body)
5. Error responses (status codes + when they occur)

Format as a Markdown API reference.
File: /docs/api/[feature]-api.md
```

**Verify:** Give the API doc to someone who has not seen the code. Can they make a successful API call using only the documentation? If not, the doc is missing something.

---

## Step 4 — Build the Context File for Your AI Agent

A single file that your AI agent reads at the start of every session to understand the full project.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create /docs/AGENT_CONTEXT.md — a context file for AI coding agents.
Include:
1. App purpose (2-3 sentences)
2. Tech stack overview (framework, database, auth, AI provider)
3. Folder structure map (key directories and what they contain)
4. Naming conventions (file names, variable names, component names)
5. Key design decisions already made (with brief rationale)
6. What has been built so far (feature list)
7. What is actively being built (current sprint)
8. What is out of scope (do not build this)
9. Links to key docs: PRDs, tech specs, API reference

This file is read by AI agents at the start of every session.
Keep it under 500 words — dense, no padding.
```

**Verify:** Start a fresh AI agent session with only this file and the context_anchor. Does the agent understand the project well enough to build without asking basic questions?

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt (With Documentation)
```
I am continuing development on my app.

AGENT CONTEXT (read this first):
[PASTE YOUR AGENT_CONTEXT.md CONTENTS]

Current sprint PRD:
[PASTE RELEVANT PRD]

Today I am building: [ONE TASK ONLY]
```

### PRD Generation Prompt
```
Generate a PRD for this feature:
App: [APP NAME AND TYPE]
Feature: [DESCRIBE IN PLAIN ENGLISH]
Users affected: [WHO]
Current state: [WHAT EXISTS TODAY]
Desired state: [WHAT WE WANT AFTER BUILDING THIS]

Use the Minimum Viable PRD template.
Be direct and specific — no vague language like "improve user experience."
Include explicit out-of-scope items to prevent scope creep.
Flag any assumptions you are making so I can correct them.
```

### Tech Spec Generation Prompt
```
Generate a tech spec for this feature based on this PRD:
[PASTE PRD]

Tech stack: [STACK]

Structure:
1. Technical approach (why this way — one paragraph)
2. Data changes: tables/columns to add, modify, or delete
3. API changes: new routes using the contract format from skill-api-routes.md
4. AI integration (if applicable): model, prompt purpose, input/output
5. Build order: numbered steps, each independently testable
6. Definition of done: checkboxes I can verify

Keep to one page. No theory — only what is needed to build.
```

### Architecture Decision Record Prompt
```
I made this technical decision: [DESCRIBE THE DECISION]
Alternatives I considered: [LIST ALTERNATIVES]
Why I chose this approach: [REASONING]

Format this as an Architecture Decision Record (ADR):
File: /docs/decisions/[date]-[brief-title].md

Include:
- Status: Accepted
- Context: what problem this solves
- Decision: what was chosen
- Consequences: what this means going forward
- Alternatives rejected: why each was not chosen

Keep it under 200 words.
```

### AGENT_CONTEXT.md Update Prompt
```
Update /docs/AGENT_CONTEXT.md to reflect what was built in this session.

What was completed: [DESCRIBE]
New files created: [LIST]
New routes added: [LIST]
New decisions made: [DESCRIBE]

Update the "What has been built" and "Current sprint" sections.
Do not change anything else.
Show me a diff of the changes before applying.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Do I really need to write a PRD before building? It feels like it slows me down."

The PRD does not slow you down — it makes you faster. Here is the concrete tradeoff:

Without a PRD: Start coding immediately. Spend 3 hours building. Realise you built the wrong scope. Spend 2 more hours fixing.

With a PRD: Spend 30 minutes writing it. Spend 2 hours building the right thing exactly.

The PRD pays for itself the moment it prevents one misaligned build session. For a PM-as-builder, it also forces you to think through the product decisions before getting seduced by the technical details of building.

**The key insight:** A PRD is not documentation for documentation's sake. It is a tool for your AI coding agent. Without it, your agent assumes. With it, your agent knows.

---

### "What goes in a tech spec vs a PRD?"

**PRD** answers: *what* and *why.*
- What problem are we solving?
- What should the user be able to do?
- Why does this matter?

**Tech Spec** answers: *how.*
- How will the database change?
- How will the API be structured?
- In what order will we build things?

A small feature might only need a PRD. A complex feature with database migrations, multiple routes, and AI integration warrants both. When in doubt: if the feature touches more than 3 files, write a tech spec.

---

### "My AI agent keeps building things I did not ask for. What is going wrong?"

Three common causes:

1. **No explicit out-of-scope list.** The agent fills gaps in the requirements with its best guess. Add a "NOT building this sprint" section to your PRD and it will stop.

2. **Scope implied by analogies.** Saying "build this like Notion" tells the agent to build every feature Notion has. Be literal, not analogical.

3. **No context file.** Without `AGENT_CONTEXT.md`, the agent does not know what already exists and builds duplicate or conflicting code. Build the context file and paste it at every session start.

</vibe_coder_bridge>

---

<testing_and_qa>

## Verifying Your Documentation Is Working

### PRD Quality Checklist
```
□ Problem statement is one sentence — specific, not generic
□ Success criteria are measurable (numbers, not feelings)
□ Out-of-scope list explicitly names what is NOT being built
□ A developer who has never seen the project could understand it
□ Open questions are listed (not assumed away)
□ No jargon that requires insider knowledge to understand
```

### Tech Spec Quality Checklist
```
□ Build order is numbered and each step is independently verifiable
□ All new DB tables and columns are specified
□ All new API routes use the contract format (method, path, request, response)
□ Definition of done has checkboxes, not vague statements
□ One page or less (if longer: split into two specs)
```

### AGENT_CONTEXT.md Effectiveness Test
```
Test: Start a brand new AI agent session.
Paste ONLY AGENT_CONTEXT.md (no other context).
Ask: "What is this project and what has been built so far?"
Pass: Agent accurately describes the app, stack, and current state
Fail: Agent is confused, asks basic questions, or describes something wrong
```

### Common Documentation Failures

| Issue | Symptom | Fix |
|---|---|---|
| PRD too vague | Agent builds a different scope than intended | Replace qualitative descriptions with specific user stories |
| No out-of-scope list | Feature creep in every session | Add explicit "NOT building" list to every PRD |
| Stale AGENT_CONTEXT.md | Agent references files that do not exist | Update context file at the end of every significant session |
| Tech spec too detailed | Agent gets lost in the details, misses the goal | Reduce to build order + data changes + API contracts only |
| No decision log | Same decisions re-debated in every session | Create /docs/decisions/ folder and add one file per key decision |

</testing_and_qa>

---

<common_patterns>

## Reusable Documentation Patterns

### Pattern 1: AGENT_CONTEXT.md Template
```markdown
# [App Name] — Agent Context

## What This App Does
[2-3 sentences: what problem it solves, who uses it, what makes it distinct]

## Tech Stack
- **Framework:** Next.js 14 (App Router)
- **Database:** Supabase (PostgreSQL + pgvector)
- **Auth:** Clerk
- **AI:** OpenAI GPT-4o + text-embedding-3-small
- **Hosting:** Vercel
- **Email:** Resend
- **Background Jobs:** Inngest

## Key Directories
```
/app/api/          → Backend API routes (one folder per feature)
/app/(app)/        → Authenticated app pages
/components/ui/    → Reusable UI primitives (shadcn/ui based)
/components/features/ → Feature-specific components
/lib/              → Shared utilities, DB client, AI wrappers, prompts
/types/            → TypeScript interfaces
/tests/            → Unit, integration, and AI output tests
/docs/             → PRDs, tech specs, API references, decisions
```

## Naming Conventions
- Files: kebab-case (`reminder-form.tsx`)
- Components: PascalCase (`ReminderForm`)
- API routes: kebab-case folders (`/app/api/reminder-create/`)
- DB tables: snake_case (`ai_generations`)
- Environment variables: SCREAMING_SNAKE_CASE

## Key Design Decisions
- Auth always checked at top of every API route before any logic
- All AI calls wrapped in `/lib/openai.ts` — never called directly from routes
- Reminders use soft delete (deleted_at column) — never hard deleted
- Rate limiting: 20 AI calls/day free, 200/day paid (via Upstash)

## What Has Been Built
- [x] Auth (Clerk — email + Google OAuth)
- [x] User profile with preferences
- [x] Reminder CRUD (create, list, complete, soft-delete)
- [x] AI chat interface with reminder tool-calling agent
- [x] Email notifications via Resend

## Currently Building
- Recurring reminders (weekly/monthly repeat logic)

## Out of Scope (Do Not Build)
- Calendar integration (planned for v2)
- Mobile app (web-first for now)
- Team/shared reminders (single-user for MVP)

## Dependencies and Gotchas
- Supabase RLS is enabled on all tables — always use service role key in API routes
- AI streaming routes need `export const runtime = "edge"` — do not remove
- All AI calls tracked in `ai_generations` table for cost monitoring
```

### Pattern 2: Feature Decision Log Entry
```markdown
# Decision: Use Soft Deletes Instead of Hard Deletes
**Date:** 2025-01-15
**Status:** Accepted

**Context:** Need to implement delete functionality for reminders.

**Decision:** Use soft deletes (`deleted_at` timestamp column) instead of `DELETE` SQL.

**Reasons:**
1. Allows undo functionality without complex event sourcing
2. Preserves data for analytics (how many reminders were deleted?)
3. Prevents accidental data loss from bugs
4. Consistent with how most production apps handle deletion

**Consequences:**
- All queries must include `WHERE deleted_at IS NULL`
- Need periodic cleanup job (delete rows older than 90 days)
- Slightly more complex queries

**Rejected alternative:** Hard deletes with a separate `deleted_reminders` archive table.
Rejected because: two tables to manage, complex restore logic.
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Include Secrets in Documentation
PRDs, tech specs, and API docs are often shared with team members or stored in git. Never include actual API keys, connection strings with passwords, or user data in any documentation file.

```markdown
# ❌ Never in docs
DATABASE_URL=postgresql://user:actualpassword@host/db

# ✅ Reference only
DATABASE_URL — see .env.example for format, obtain from Supabase dashboard
```

### Rule 2: Treat AGENT_CONTEXT.md as a Security Surface
This file is pasted into every AI agent session. Do not include:
- Information about security vulnerabilities in your current code
- Details about rate limit thresholds that would help an attacker
- Any system prompt contents that users should not see

### Rule 3: Version Control All Documentation
Store all docs in your git repository (`/docs/` folder). This gives you:
- History of decisions and why they were made
- The ability to revert to a previous context if needed
- Shared visibility if you ever work with collaborators

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Writing Documentation After the Fact
Post-hoc docs are always incomplete and often wrong. Memory fades, decisions get rationalised differently, and the "why" gets lost.  
**Fix:** PRD before the first line of code. Tech spec before complex features. Decision log entries the moment a non-obvious decision is made.

### ❌ PRDs That Describe Ideal Futures Instead of Specific Builds
"The reminder feature will delight users with intelligent scheduling that learns their habits over time."  
This tells an AI agent nothing actionable.  
**Fix:** "Users can type a natural language reminder. The app parses it into structured fields (task, time, recurrence). The reminder is saved and displayed in the list."

### ❌ AGENT_CONTEXT.md That Goes Stale
You write AGENT_CONTEXT.md in week 1. By week 6, the stack has changed, new features are built, and several "out of scope" items are now in scope. The agent reads the stale file and wastes time.  
**Fix:** Update AGENT_CONTEXT.md at the end of every session where something significant changed. Treat it as a living document.

### ❌ Tech Specs That Describe Architecture Instead of Build Steps
A spec that explains database theory or API design philosophy instead of "Step 1: create this table with these columns" is useless for a coding agent.  
**Fix:** Every tech spec must have a numbered build order with verifiable steps.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Documentation Practice

### Add Automated API Documentation with OpenAPI
Generate API docs automatically from your route files:
```
Ask your AI agent: "Generate an OpenAPI 3.0 spec for all routes in /app/api/.
Export as /docs/api/openapi.json.
Include: all request/response schemas, authentication requirements, error codes."

Tools: swagger-jsdoc (JSDoc comments → OpenAPI), ts-to-openapi
```

### Add a CHANGELOG.md
Track every significant product change for users and stakeholders:
```markdown
# Changelog

## [Unreleased]
### Added
- Recurring reminder support (weekly and monthly)

## [1.2.0] — 2025-01-15
### Added
- Email notifications for upcoming reminders
### Fixed
- Reminder time parsing now handles timezone correctly
```

### Create a Feature Flag Documentation Pattern
When rolling out features incrementally:
```markdown
# Feature Flags

| Flag | Status | Description | Rollout |
|---|---|---|---|
| recurring_reminders | 10% | Weekly/monthly repeat | 10% of paid users |
| voice_input | 0% | Voice-to-reminder | Not started |
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: EdTech App — PRD That Prevented a Week of Rework

**Without PRD:** Developer (AI agent) built a lesson generator that let users pick from 15 subject areas, 4 difficulty levels, and 6 learning styles. It was technically impressive. The product owner had wanted a simple "enter a topic, get a lesson" interface for the MVP.

**With PRD (after this incident):**
```markdown
## What We Are NOT Building (this sprint):
- Subject area taxonomy or browsing
- Difficulty level selection (always calibrated to user's stored level)
- Learning style selection (always uses user's stored preference)
- Lesson library or history view (that is a separate feature)
```

**Result:** The next sprint built exactly what was needed in half the time. The PRD's out-of-scope list was the most valuable part.

---

### Case Study 2: Reminder App — AGENT_CONTEXT.md as a Session Multiplier

**Before AGENT_CONTEXT.md:** Each AI agent session began with 15–20 minutes of re-explaining the project, the stack, what had been built, and what conventions were in use. The agent would occasionally suggest adding Prisma when Supabase was already set up, or recommend building auth when Clerk was already integrated.

**After AGENT_CONTEXT.md:** Context injection at session start (paste the file + context_anchor JSON). Session immediately productive. Agent correctly references existing patterns, avoids duplicating existing code, and stays within established conventions.

**Time saved:** ~15 minutes per session. Across 40 sessions: 10 hours of recaptured productivity.

</real_world_examples>

</skill_document>
