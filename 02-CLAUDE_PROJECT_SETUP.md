<div align="center">

# 🤖 Set Up Your Vibe Coder Assistant

### Turn these 26 skill documents into an interactive AI coding partner — using a Claude Project.

<br/>

> **No code required. 15 minutes. Free on Claude.ai.**  
> Every conversation automatically draws from all 26 skills simultaneously.  
> Better than a CustomGPT — all documents are in full context, not retrieved in chunks.

</div>

---

## Why a Claude Project?

When you paste a skill document into a chat session, only that skill is available. A Claude Project loads all 26 documents into the model's context window for every conversation — meaning the assistant can cross-reference skills, apply the right pattern without being told which file to use, and maintain consistency across sessions.

| | Paste per session | Claude Project |
|---|---|---|
| Skills available | 1 at a time | All 26 simultaneously |
| Context between sessions | Starts fresh | Project instructions persist |
| Setup time | Every session | Once |
| Shareable | No | Yes — one link |
| Quality | Good | Significantly better |

---

## What You'll Have When You're Done

A private or public Claude Project called **Vibe Coder Assistant** that:

- Opens every conversation by asking for your project context
- Automatically identifies which skills apply to your question
- Enforces incremental building (one thing at a time, always)
- Applies security guardrails without being asked
- Gives plain-English explanations of every technical decision
- Ends every response with a concrete verification step

---

## Step-by-Step Setup

### Step 1 — Open Claude Projects

1. Go to **[claude.ai](https://claude.ai)**
2. Sign in (free account works; Pro unlocks larger context windows)
3. In the left sidebar, click **"Projects"**
4. Click **"+ New Project"**

> 💡 Projects are available on Claude.ai free and paid plans. The Pro plan gives you a significantly larger context window, which means all 26 files are available more reliably.

---

### Step 2 — Name Your Project

Give it a clear, descriptive name:

```
Vibe Coder Assistant
```

Optionally add a description:
```
AI coding partner for building production-grade AI products. 
Powered by 26 skill documents covering backend, frontend, AI agents, testing, deployment, and growth.
```

---

### Step 3 — Add the Project Instructions

In the **Project Instructions** field (also called the system prompt), paste the full contents of the system prompt below.

**[→ Copy the full system prompt](./vibe-coder-assistant-system-prompt.md)**

> ⚠️ **Important:** Paste the complete file contents. Do not truncate. The routing logic, conversation modes, and security rules all depend on the full document being present.

---

### Step 4 — Upload the Skill Documents

In the **Project Knowledge** section, upload all 26 skill files. You can find them in the folders of this repository.

**Upload in this order** (dependencies first):

**Upload Group 1 — Backend (6 files)**
```
backend/skill-backend-architecture.md
backend/skill-api-routes.md
backend/skill-agentic-workflows.md
backend/skill-database-storage.md
backend/skill-authentication-authorization.md
backend/skill-api-design-integration.md
```

**Upload Group 2 — Frontend (4 files)**
```
frontend/skill-frontend-architecture.md
frontend/skill-ui-components.md
frontend/skill-ai-streaming-ui.md
frontend/skill-responsive-design.md
```

**Upload Group 3 — Testing (3 files)**
```
testing/skill-testing-strategy.md
testing/skill-debugging-error-handling.md
testing/skill-automated-testing.md
```

**Upload Group 4 — DevOps (3 files)**
```
devops/skill-deployment-hosting.md
devops/skill-environment-variables-secrets.md
devops/skill-monitoring-observability.md
```

**Upload Group 5 — AI-Specific (6 files)**
```
ai-specific/skill-single-vs-multi-agent.md
ai-specific/skill-prompt-engineering.md
ai-specific/skill-ai-agent-design.md
ai-specific/skill-rag.md
ai-specific/skill-model-selection-evaluation.md
ai-specific/skill-ai-safety-guardrails.md
```

**Upload Group 6 — Product (4 files)**
```
product/skill-product-documentation.md
product/skill-user-experience-design.md
product/skill-analytics-experimentation.md
product/skill-growth-distribution.md
```

> 💡 **Tip:** Claude.ai accepts multiple file uploads at once. Select all files in a group and drag them in together.

---

### Step 5 — Test Your Setup

Start a new conversation in the project. You should see the opening message from the assistant:

```
👋 Welcome to Vibe Coder Assistant.

I'm your AI coding partner for building production-grade AI products...
```

**Run these three test prompts to verify everything is working:**

**Test 1 — Architecture decision**
```
I want to build a conversational reminder app. Users type reminders 
in plain English and the app schedules them. I haven't chosen a stack yet.
Where do I start?
```
Expected: The assistant asks for context, then walks through stack selection and the initial architecture decision.

**Test 2 — Skill routing**
```
Should I use a single agent or multiple agents for parsing reminder intent?
```
Expected: The assistant draws from `skill-single-vs-multi-agent` and gives a decision framework with the upgrade triggers.

**Test 3 — Security guardrail**
```
Build me an API route that creates a reminder. Here's my context:
App: RemindAI, Stack: Next.js + Supabase, Auth: Clerk
```
Expected: The assistant generates a route with auth check, Zod validation, error handling, and a verify step — without being asked for any of these.

If all three work correctly, your assistant is fully operational.

---

### Step 6 — (Optional) Share It Publicly

If you want to share your Vibe Coder Assistant with others:

1. In your project settings, find **"Share project"**
2. Toggle to **"Anyone with the link"**
3. Copy the link
4. Share on X, LinkedIn, Reddit (r/ClaudeAI, r/SideProject, r/ChatGPT), IndieHackers

> 🔒 **Privacy note:** Shared projects expose your project instructions and uploaded files to anyone with the link. The 26 skill documents are open-source so sharing is fine. If you've added any private project context (your own API keys, business logic, internal docs), create a separate clean project for sharing.

---

## Using the Assistant Effectively

### The Context Anchor — Do This Every Session

The single habit that makes the most difference. At the start of every conversation, paste this JSON filled in with your project details:

```json
{
  "project_context": {
    "app_name": "your app name",
    "app_type": "EdTech web app / reminder assistant / etc.",
    "framework": "Next.js / FastAPI / etc.",
    "database": "Supabase / PlanetScale / etc.",
    "auth_provider": "Clerk / Supabase Auth / etc.",
    "ai_provider": "OpenAI / Anthropic / etc.",
    "already_built": ["user auth", "reminder CRUD", "etc."],
    "building_today": "ONE specific thing"
  }
}
```

This takes 60 seconds and prevents the assistant from assuming the wrong stack, hallucinating files that don't exist, or suggesting things you've already built differently.

---

### The One-Thing Rule

The assistant enforces this, but it helps to think this way yourself:

> Every session has one goal. One route. One component. One feature. One bug.

When you're tempted to say "build the whole dashboard," say instead:
"Build the reminder list component. It should fetch from `/api/reminders` and show empty state when empty."

Smaller scope = faster build = easier to verify = less chance of breaking something.

---

### The Verify Step

Every response ends with a verification instruction. **Don't skip it.** It takes 30 seconds and is the difference between "it seems to work" and "it works."

---

### When to Use Which Mode

The assistant has three modes — it switches automatically, but knowing them helps you phrase requests better:

| Mode | When | How to trigger |
|---|---|---|
| **Architect** | Planning something new | "I want to build X" / "Should I use Y or Z?" |
| **Builder** | Ready to write code | "Build me X" / "Write the code for Y" |
| **Debugger** | Something is broken | "This is broken" / "I'm getting this error: [paste error]" |

---

## Frequently Asked Questions

**Q: Do I need Claude Pro?**  
A: Free works. Pro gives you a larger context window, which means all 26 files are more reliably accessible in longer conversations. For the best experience, Pro is recommended.

**Q: Can I use this with Cursor or Codex instead?**  
A: Yes. The skill documents work with any AI coding tool. For Cursor, add the relevant skill content to your `.cursorrules` file. For Codex, paste into the system prompt. The Claude Project is just the most convenient way to use all 26 simultaneously.

**Q: Can I add my own skills?**  
A: Absolutely. Fork this repo, add your skill document following the 12-section structure from the README, and upload it to your project. The assistant will use it automatically.

**Q: What if the assistant ignores a skill and gives a generic answer?**  
A: Try explicitly naming the skill: "Use the AI agent design skill to help me..." This forces the assistant to draw from the right document.

**Q: How do I keep the skills updated?**  
A: When you pull the latest version of this repo, re-upload the changed files in your Claude Project. You'll need to delete the old version and upload the new one.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Assistant gives generic answers, not skill-based ones | Check that all 26 files were successfully uploaded (project knowledge section) |
| Opening message doesn't appear | Verify the full system prompt was pasted into Project Instructions |
| Assistant builds multiple things at once | It's honouring a request that's too broad — rephrase to one specific thing |
| Wrong skill applied | Explicitly name the skill in your prompt: "Using the RAG skill..." |
| Context lost mid-conversation | Paste the context anchor JSON again — long conversations can lose early context |

---

<div align="center">

**Built from 26 hours of structured skill development.**  
**Designed to give every vibe coder a senior engineer as a pair programmer.**

[← Back to Skills README](./README.md)

</div>
