<div align="center">

# 🧠 Vibe Coding Skills

### 26 battle-tested skill documents for building production-grade AI products — without a CS degree.

<br/>

[![Skills](https://img.shields.io/badge/Skills-26%20Documents-6366f1?style=for-the-badge&logo=bookstack&logoColor=white)](.)
[![Compatible](https://img.shields.io/badge/Works%20With-Claude%20%7C%20Cursor%20%7C%20Codex-0ea5e9?style=for-the-badge&logo=anthropic&logoColor=white)](.)
[![License](https://img.shields.io/badge/Licenshe-MIT-22c55e?style=for-the-badge)](./LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-f59e0b?style=for-the-badge)](.)

<br/>

> **Paste any skill into Claude Code, Cursor, or Codex at the start of a session.**  
> Your AI agent instantly knows the patterns, guardrails, and architecture decisions for that domain.  
> Build production-ready AI products — one skill at a time.

<br/>

[**→ Use as a Claude Project (recommended)**](./CLAUDE_PROJECT_SETUP.md) · [**→ Browse all skills**](#-the-26-skills) · [**→ Quick start**](#-quick-start)

</div>

---

## What Is This?

Most vibe coders hit the same walls:

- The AI agent builds the wrong thing because there's no shared context
- Code works locally, breaks in production
- AI calls cost 10x more than they should
- Security issues slip through because nobody told the agent to check
- Every session starts from zero — no memory of what was decided before

**These 26 skill documents fix that.**

Each one is a structured reference document that you feed to your AI coding agent. It contains the mental models, architecture patterns, step-by-step build instructions, production-ready code snippets, security guardrails, and debugging loops for one specific domain of software development.

The documents were built for **AI product builders** — people creating EdTech platforms, conversational AI apps, productivity tools, and AI-powered SaaS — using natural language in their IDE rather than writing code from scratch.

---

## ⚡ Quick Start

**Option 1 — Claude Project (best experience)**  
Set up once. Every conversation is grounded in all 26 skills. → [Full setup guide](./CLAUDE_PROJECT_SETUP.md)

**Option 2 — Paste per session**  
Copy the relevant skill document into your Claude/Cursor/Codex context at the start of a session. Works with any AI coding tool.

**Option 3 — Add to `.cursorrules`**  
Drop skill content into your `.cursorrules` file for persistent context in Cursor.

```bash
# Clone this folder into your project
git clone https://github.com/dushyantshahai/dushyantshah-product-portfolio.git
cd dushyantshah-product-portfolio/vibe-coding-skills
```

**Then, start any AI session with:**
```
Read the skill document I'm pasting below. Apply its patterns, guardrails, 
and architecture decisions to everything you build in this session.

My project context:
- App: [your app name and type]
- Stack: [your stack]
- Already built: [what exists]
- Building today: [one specific thing]

[PASTE SKILL DOCUMENT HERE]
```

---

## 📂 Folder Structure

```
vibe-coding-skills/
│
├── README.md                          ← You are here
├── CLAUDE_PROJECT_SETUP.md            ← How to build the AI assistant
│
├── backend/
│   ├── skill-backend-architecture.md
│   ├── skill-api-routes.md
│   ├── skill-agentic-workflows.md
│   ├── skill-database-storage.md
│   ├── skill-authentication-authorization.md
│   └── skill-api-design-integration.md
│
├── frontend/
│   ├── skill-frontend-architecture.md
│   ├── skill-ui-components.md
│   ├── skill-ai-streaming-ui.md
│   └── skill-responsive-design.md
│
├── testing/
│   ├── skill-testing-strategy.md
│   ├── skill-debugging-error-handling.md
│   └── skill-automated-testing.md
│
├── devops/
│   ├── skill-deployment-hosting.md
│   ├── skill-environment-variables-secrets.md
│   └── skill-monitoring-observability.md
│
├── ai-specific/
│   ├── skill-single-vs-multi-agent.md
│   ├── skill-prompt-engineering.md
│   ├── skill-ai-agent-design.md
│   ├── skill-rag.md
│   ├── skill-model-selection-evaluation.md
│   └── skill-ai-safety-guardrails.md
│
└── product/
    ├── skill-product-documentation.md
    ├── skill-user-experience-design.md
    ├── skill-analytics-experimentation.md
    └── skill-growth-distribution.md
```

---

## 🗺️ The 26 Skills

### 🏗️ Backend (6 skills)
*The foundation everything else is built on. Start here.*

| Skill | What It Solves | Use When |
|---|---|---|
| [**Backend Architecture**](./backend/skill-backend-architecture.md) | System design, stack decisions, folder structure, AI orchestration layer | Starting a new project or restructuring a messy one |
| [**API Routes**](./backend/skill-api-routes.md) | Building REST endpoints with validation, error handling, and consistent response shapes | Adding any server-side feature |
| [**Agentic Workflows**](./backend/skill-agentic-workflows.md) | Multi-step AI pipelines with state management, retries, and graceful failure | Building features that require more than one AI call |
| [**Database & Storage**](./backend/skill-database-storage.md) | Schema design, SQL/NoSQL/Vector DB selection, Prisma, Row Level Security | Designing any data model |
| [**Auth & Authorization**](./backend/skill-authentication-authorization.md) | Login flows, session management, RBAC, Clerk/Supabase Auth setup | Adding users or protecting any route |
| [**API Design & Integration**](./backend/skill-api-design-integration.md) | Consuming third-party APIs, webhook handlers, idempotency, integration wrappers | Connecting to Stripe, OpenAI, Resend, or any external service |

---

### 🎨 Frontend (4 skills)
*Build UIs that feel intentional, not accidental.*

| Skill | What It Solves | Use When |
|---|---|---|
| [**Frontend Architecture**](./frontend/skill-frontend-architecture.md) | Component hierarchy, state management decisions, data flow from API to UI | Starting a new frontend or when pages grow unmanageable |
| [**UI Components**](./frontend/skill-ui-components.md) | Building a reusable component library with shadcn/ui, accessibility built in | Building any shared UI element |
| [**AI Streaming UI**](./frontend/skill-ai-streaming-ui.md) | Rendering AI responses word-by-word with Vercel AI SDK, useChat, Markdown rendering | Any feature where users wait for AI output |
| [**Responsive Design**](./frontend/skill-responsive-design.md) | Mobile-first Tailwind layouts, app shells, iOS safe areas, 100dvh fix | Any layout that needs to work on phones |

---

### 🧪 Testing & Reliability (3 skills)
*Deploy with confidence, not dread.*

| Skill | What It Solves | Use When |
|---|---|---|
| [**Testing Strategy**](./testing/skill-testing-strategy.md) | What to test, test priority matrix, the AI output testing problem | Before writing your first test |
| [**Debugging & Error Handling**](./testing/skill-debugging-error-handling.md) | Systematic debug process, structured logging, Sentry setup, error boundaries | When stuck on a bug or setting up production error tracking |
| [**Automated Testing**](./testing/skill-automated-testing.md) | Vitest setup, unit/integration/AI output tests, GitHub Actions CI pipeline | Setting up a test suite or adding CI/CD |

---

### 🚀 Deployment & DevOps (3 skills)
*Get it live. Keep it live.*

| Skill | What It Solves | Use When |
|---|---|---|
| [**Deployment & Hosting**](./devops/skill-deployment-hosting.md) | Vercel/Railway deploy, preview environments, function timeouts for AI routes | First deploy or switching platforms |
| [**Environment Variables & Secrets**](./devops/skill-environment-variables-secrets.md) | Managing API keys across dev/preview/prod, secret rotation, startup validation | Setting up a new project or rotating a compromised key |
| [**Monitoring & Observability**](./devops/skill-monitoring-observability.md) | Sentry, uptime monitoring, AI cost tracking, daily cost alerts | Before going live with real users |

---

### 🤖 AI-Specific (6 skills)
*The skills that make this library unique. The difference between an AI feature that works 60% of the time and one that works 94% of the time.*

| Skill | What It Solves | Use When |
|---|---|---|
| [**Single vs Multi-Agent**](./ai-specific/skill-single-vs-multi-agent.md) | The architecture decision: one agent or many? Decision tree + upgrade triggers | **Before designing any AI feature** — this is the entry point |
| [**Prompt Engineering**](./ai-specific/skill-prompt-engineering.md) | Writing prompts that produce consistent, parseable outputs every time | When AI outputs are inconsistent or keep failing format checks |
| [**AI Agent Design**](./ai-specific/skill-ai-agent-design.md) | Tool-calling agents, multi-agent orchestration, agent memory patterns | Building AI that takes actions, not just generates text |
| [**RAG**](./ai-specific/skill-rag.md) | Retrieval Augmented Generation: chunking, embeddings, pgvector, semantic search | When AI needs to answer from your own content or remember past conversations |
| [**Model Selection**](./ai-specific/skill-model-selection-evaluation.md) | Choosing the right model for each task, evaluation protocol, A/B testing models | When costs are too high or quality is inconsistent |
| [**AI Safety & Guardrails**](./ai-specific/skill-ai-safety-guardrails.md) | Rate limiting, prompt injection defence, output validation, content policy | Before exposing any AI feature to real users |

---

### 📦 Product & Non-Technical (4 skills)
*The layer that wraps everything. Often skipped. Never should be.*

| Skill | What It Solves | Use When |
|---|---|---|
| [**Product Documentation**](./product/skill-product-documentation.md) | PRDs, tech specs, AGENT_CONTEXT.md — context files that ground your AI agent | Before starting any significant feature |
| [**User Experience Design**](./product/skill-user-experience-design.md) | User flows, friction audits, empty states, UI copy, accessibility | Before building any new screen |
| [**Analytics & Experimentation**](./product/skill-analytics-experimentation.md) | Event tracking, funnels, A/B tests, AI quality metrics | Before your first real users arrive |
| [**Growth & Distribution**](./product/skill-growth-distribution.md) | SEO, email sequences, push notifications, referral mechanics | After retention is solid and you're ready to grow |

---

## 🔑 How Each Skill Document Is Structured

Every document follows the same 12-section format so your AI agent always knows where to look:

```
1. Overview          → What this skill solves and when to use it
2. Context Anchor    → JSON template to fill in for your specific project
3. Mental Models     → How to think about this domain (plain English)
4. System Design     → Architecture diagrams and component breakdown
5. Step-by-Step      → Incremental build instructions with verify steps
6. AI Agent Prompts  → Ready-to-paste prompts for each step
7. Vibe Coder Bridge → Plain-English trade-off explanations
8. Testing & QA      → How to verify this works + debugging loops
9. Common Patterns   → Reusable code templates
10. Security         → Non-negotiable guardrails for this domain
11. Mistakes to Avoid → The anti-patterns that cause the most pain
12. Real Examples    → Mini case studies from real AI products
```

The **Context Anchor** section is the most important piece. Fill it in at the start of every session:

```json
{
  "project_context": {
    "app_name": "your app name",
    "framework": "Next.js / FastAPI / etc.",
    "what_already_exists": "list what works",
    "building_today": "ONE specific thing"
  }
}
```

Paste it into your AI agent before the skill content. It prevents the agent from hallucinating files that don't exist or suggesting tools you've already set up differently.

---

## 📐 Design Principles

These documents were built with five constraints that make them different from generic tutorials:

**1. Incremental build rule** — Every execution section enforces: build one unit → test it → verify → proceed. No "generate the whole backend" prompts.

**2. Context-first** — Every session starts by grounding the AI in your specific project, not a hypothetical one.

**3. Security non-negotiable** — Every document has a `<security_guardrails>` section that applies to that domain. Auth checks, input validation, secret management — always present.

**4. AI-specific patterns** — The AI-Specific section covers problems that don't exist in standard dev tutorials: prompt injection, non-deterministic output validation, per-feature cost tracking, and the single-vs-multi-agent decision that most guides skip entirely.

**5. Plain-English bridges** — Every technical decision has a `vibe_coder_bridge` section that explains it in plain English, with trade-offs stated clearly.

---

## 🤝 Contributing

Found a pattern that should be in here? A security guardrail that's missing? A better code pattern for a specific framework?

1. Fork the repo
2. Add or update the relevant skill document
3. Follow the 12-section structure above
4. Submit a PR with a one-line description of what you changed and why

Bug reports and experience reports (what worked, what didn't) are equally welcome as Issues.

---

## 🙏 Credits

Built by **[Dushyant Shah](https://github.com/dushyantshahai)** — Senior PM building AI products and documenting the patterns along the way.

The meta-prompt framework these documents are built on was synthesised from three generations of vibe coding frameworks, refined through parallel AI product development across an EdTech app and a conversational reminder app.

---

<div align="center">

**If this saved you time, a ⭐ helps others find it.**

[Star this repo](https://github.com/dushyantshahai/dushyantshah-product-portfolio) · [Follow for updates](https://github.com/dushyantshahai) · [Share on X](https://twitter.com/intent/tweet?text=Just%20discovered%20this%20free%20library%20of%2026%20skill%20documents%20for%20building%20production-grade%20AI%20products%20%E2%80%94%20no%20CS%20degree%20needed.%20Feed%20any%20skill%20to%20Claude%20Code%2C%20Cursor%2C%20or%20Codex.&url=https%3A%2F%2Fgithub.com%2Fdushyantshahai%2Fvibe-coding-skills)

</div>
