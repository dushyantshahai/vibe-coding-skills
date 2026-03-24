---
name: ai-agent-design
description: Builds tool-calling AI agents that query databases, call APIs, and take actions. Use when building a conversational AI feature that needs to read or write data, when designing multi-agent orchestrators, or when adding agent memory.
---
# Skill: AI Agent Design

```json
{
  "skill_id": "ai-agent-design",
  "category": "AI-Specific",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>AI Agent Design — Building Agents That Use Tools, Take Actions, and Stay On Track</title>

<overview>

## What This Skill Enables
- Build AI agents that go beyond text generation — agents that query databases, call APIs, and take actions on behalf of users
- Design tool-calling agents that decide which tool to use based on context, without hardcoded logic for every case
- Build multi-agent orchestration systems using the patterns from `skill-single-vs-multi-agent.md`
- Implement memory so agents can remember context across multiple sessions or conversation turns

## Why It Matters for Vibe Coders
Most AI features start as "call the AI, show the response." Real AI products need agents that act: fetch the user's calendar, create a reminder, look up their lesson history, send a notification. This skill is how you give AI the ability to reach into your app's data and services — and do it reliably without going off-script.

## When to Use This Skill
- When your AI feature needs to read from or write to your database as part of generating a response
- When you need the AI to decide which action to take based on user input (not hardcoded if/else logic)
- After consulting `skill-single-vs-multi-agent.md` and deciding you need a tool-calling or multi-agent pattern
- When building a conversational feature that needs to remember context across turns

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "ai_provider": "[REPLACE: e.g. OpenAI / Anthropic]",
    "ai_model": "[REPLACE: e.g. gpt-4o / claude-3-5-sonnet]",
    "agent_type": "[REPLACE: single tool-calling agent / sequential multi-agent / parallel multi-agent]",
    "agent_feature": "[REPLACE: plain English, e.g. 'chat assistant that can create and list reminders']",
    "tools_needed": [
      "[REPLACE: e.g. createReminder — writes a reminder to DB]",
      "[REPLACE: e.g. listReminders — reads user's reminders from DB]",
      "[REPLACE: e.g. searchLessons — semantic search over lesson content]"
    ],
    "memory_requirement": "[REPLACE: none / conversation history only / long-term user preferences]",
    "existing_agent_files": "[REPLACE: any agent files already created]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About AI Agents

### Mental Model 1: The Smart Assistant With a Toolbelt
A tool-calling agent is like a capable assistant who has a set of tools on their belt (database query, API call, search function). When you give them a task, they:
1. Understand what you need
2. Decide which tool(s) to use
3. Use the tool(s) and get results
4. Use the results to form their response

You define the tools. The AI decides when and how to use them. This is fundamentally different from hardcoded logic where your code decides which function to call.

### Mental Model 2: The Conversation Loop
```
USER MESSAGE
      │
      ▼
AI READS MESSAGE + AVAILABLE TOOLS
      │
      ├─→ If AI decides to use a tool:
      │     AI calls the tool → your code runs the tool → result returned to AI
      │     AI reads result → decides next step (another tool? final answer?)
      │
      └─→ If AI has enough information:
            AI writes final response
            Loop ends
```

This loop can run multiple times per user message. The AI might call `searchLessons`, read the results, then call `getUserLevel`, read that result, then finally generate a personalised recommendation — all in response to one user message.

### Mental Model 3: Tool Design Principles
```
GOOD TOOLS have:
✅ A single, clear responsibility
✅ Typed input parameters with descriptions
✅ Predictable return shape
✅ Fast execution (<500ms where possible)
✅ Graceful error handling (return null, not throw)

BAD TOOLS have:
❌ Multiple responsibilities ("getAndUpdateUser")
❌ Vague parameter descriptions (the AI misuses them)
❌ Unpredictable return shapes (varies by data)
❌ Slow execution (blocks the response)
❌ Side effects without confirmation (deletes data silently)
```

</mental_models>

---

<system_design_breakdown>

## Agent Architecture Patterns

### Pattern A: Single Tool-Calling Agent
```
USER MESSAGE + CONVERSATION HISTORY
           │
           ▼
    LLM WITH TOOLS
    ├── Tool: createReminder(task, scheduledAt)
    ├── Tool: listReminders(userId, filter?)
    ├── Tool: deleteReminder(reminderId)
    └── Tool: searchKnowledge(query)
           │
    (may call 0, 1, or multiple tools per response)
           │
           ▼
    FINAL TEXT RESPONSE
```

### Pattern B: Orchestrator + Specialist Agents
```
USER INPUT
    │
    ▼
ORCHESTRATOR AGENT
(decides which specialist to invoke)
    │
    ├──→ SPECIALIST: LessonCreator
    │       Uses tools: fetchCurriculum, generateContent, saveLesson
    │
    ├──→ SPECIALIST: ProgressTracker
    │       Uses tools: fetchStudentRecord, updateProgress, calculateScore
    │
    └──→ SPECIALIST: Recommender
            Uses tools: fetchHistory, semanticSearch, rankResults
```

### Tool Definition Contract
Every tool needs three things the AI uses to decide whether and how to call it:

```typescript
{
  name: "createReminder",                    // Must be clear and verb-based
  description: "Create a new reminder for the user. Call this when the user asks to be reminded about something.",  // Critical — AI uses this to decide when to call
  parameters: {                              // Exact schema the AI must provide
    type: "object",
    properties: {
      task: { type: "string", description: "What the user wants to be reminded about" },
      scheduledAt: { type: "string", description: "ISO 8601 datetime for the reminder" },
    },
    required: ["task"],
  },
}
```

## Agent Memory: Three Levels

| Memory Type | Storage | Use Case | Implementation |
|---|---|---|---|
| In-session | JS array in route handler | Single conversation | `const messages = []` passed to streamText |
| Cross-session | Database (messages table) | Persistent chat history | Load last N messages from DB on each request |
| Long-term semantic | pgvector / Pinecone | "Remember what the user told me weeks ago" | Embed + store facts, retrieve by similarity |

### Sliding Window Context Trimming

When conversation history exceeds the model's context window, trim oldest messages:

```typescript
// lib/ai/memory.ts
const MAX_CONTEXT_TOKENS = 8000 // leave room for system prompt + response

export function trimConversationHistory(
  messages: Message[],
  maxTokens = MAX_CONTEXT_TOKENS
): Message[] {
  // rough estimate: 1 token ≈ 4 chars
  let totalChars = 0
  const trimmed: Message[] = []

  for (const msg of [...messages].reverse()) {
    const chars = JSON.stringify(msg).length
    if (totalChars + chars > maxTokens * 4) break
    trimmed.unshift(msg)
    totalChars += chars
  }

  return trimmed
}
```

Always keep the FIRST message (system context) and trim from the middle, not the start.

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Define tools → build one tool → test it → add it to agent → test agent → add next tool. -->

## Step 1 — Define Your Tool Catalogue

Before building anything, map every tool your agent needs.

**Tool design worksheet for each tool:**
```
Tool name: [verb_noun format — e.g. create_reminder, list_reminders]
When should the AI call this? [describe the trigger condition in one sentence]
Input parameters: [name, type, required/optional, description]
What it does: [one sentence — no ambiguity]
What it returns: [describe the return shape]
Side effects? [yes/no — does it write/delete data?]
```

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Design the tool catalogue for my [FEATURE] agent.
The agent needs to: [DESCRIBE WHAT THE AGENT DOES]

For each tool, provide:
1. Tool name (verb_noun format)
2. One-sentence description the AI will use to decide when to call it
3. Input parameters with types and descriptions
4. Return type and shape
5. Whether it has side effects (write/delete operations)

List only the tools that are truly necessary. Start minimal.
```

---

## Step 2 — Build and Test One Tool Function

Each tool is a regular TypeScript/Python function. Build and test it independently before connecting to the agent.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Build the tool function: [TOOL NAME]
File: /lib/tools/[tool-name].ts

This tool: [DESCRIBE WHAT IT DOES]
Input: [DESCRIBE PARAMETERS]
Returns: [DESCRIBE RETURN SHAPE]

Requirements:
- Returns null (not throws) if the operation finds no data
- Returns { success: false, error: string } if the operation fails
- Includes a JSDoc comment with the tool description (same text used in AI tool definition)
- Uses the existing DB client from /lib/db.ts

Show me a test to verify this tool works before adding it to the agent.
```

**Verify:** Call the tool function directly with test inputs. It returns the expected shape.

---

## Step 3 — Build the Agent with One Tool

Start the agent with just one tool. Get it working end-to-end before adding more.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Tool function is working: [TOOL NAME] at /lib/tools/[tool-name].ts

Build an agent at /lib/agents/[agent-name].ts with:
1. System prompt that defines the agent's role and scope
2. One tool registered: [TOOL NAME]
3. The tool's execute function calls the real tool function from /lib/tools/
4. Returns the agent's final text response
5. Uses Vercel AI SDK (streamText with tools) OR raw OpenAI tool calling

Also generate a test function: testAgent(message: string) → logs full response
```

**Verify:** Call `testAgent("create a reminder to call mum tomorrow")`. Confirm:
- The AI calls the `createReminder` tool (check logs)
- The tool executes with correct parameters
- The AI produces a confirmation response

---

## Step 4 — Add Remaining Tools One at a Time

For each additional tool: build the function → test it → add to agent → test agent with that tool.

**Never add multiple tools at once.** When an agent behaves unexpectedly, you need to know which tool is causing the issue.

---

## Step 5 — Add Conversation Memory

If the agent needs to remember previous turns in the conversation.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Agent is working with [N] tools.

Add conversation history to the agent:
1. Accept messages: Message[] as input (full conversation history)
2. Append the new user message to history
3. Pass full history to the LLM call
4. Append the AI's response to history
5. Return: { response: string, updatedHistory: Message[] }

The caller is responsible for persisting and passing history — the agent function is stateless.
Show me how the caller (API route) should store and pass history between requests.
```

---

## Step 6 — Add Long-Term Memory (User Preferences / Profile)

For agents that should remember things across sessions (not just conversation turns).

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add long-term user memory to the agent.
What to remember: [DESCRIBE — e.g. user's preferred reminder time, learning level, completed topics]

Pattern:
1. At agent startup: fetch user's stored preferences from DB
2. Inject into system prompt: "User preferences: [preferences]"
3. After significant interactions: extract new preferences to store
4. Tool: updateUserPreference(key, value) — agent can update preferences as it learns

Do not use vector search for this — just a simple user_preferences JSON column.
```

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am building an AI agent for my app.
Project context: [PASTE context_anchor JSON]

Agent type: [single tool-calling / sequential multi-agent]
Tools already built and tested: [LIST]
Building today: [ONE TOOL OR THE AGENT ORCHESTRATOR]

Before writing code:
1. Tell me the file you will create
2. Tell me the function signature
3. Confirm the tool I am adding follows the single-responsibility principle
```

### Tool Catalogue Design Prompt
```
Design the complete tool catalogue for this AI agent:
Agent purpose: [DESCRIBE IN ONE SENTENCE]
User actions it must support: [LIST — e.g. "create reminder", "list reminders", "delete reminder"]
Data it can access: [LIST TABLES/COLLECTIONS — e.g. reminders table, lessons table]
Actions it can take: [LIST WRITE OPERATIONS — e.g. create, update, delete]

For each tool:
1. Name (verb_noun)
2. Trigger description (when should the AI call this?)
3. Input parameters with types
4. Return type
5. Side effect warning (if any)

Rules: minimum tools needed, one responsibility per tool, read tools separate from write tools.
```

### Tool-Calling Agent Setup Prompt (Vercel AI SDK)
```
Build a tool-calling agent using Vercel AI SDK.
Project context: [PASTE context_anchor JSON]

File: /lib/agents/[name]-agent.ts
Model: [MODEL]
System prompt: [PASTE YOUR SYSTEM PROMPT FROM skill-prompt-engineering.md]

Tools to register:
[PASTE YOUR TOOL CATALOGUE]

For each tool's execute function: import and call the real function from /lib/tools/[name].ts
Return: AsyncGenerator (for streaming) or string (for non-streaming)
Include: maxSteps: 5 (prevent infinite tool-calling loops)
```

### Multi-Agent Orchestrator Prompt
```
Build an orchestrator that coordinates these specialist agents:
[LIST AGENTS AND THEIR RESPONSIBILITIES]

File: /lib/agents/orchestrator.ts

The orchestrator should:
1. Receive the user's message and conversation history
2. Classify the intent (which specialist is needed?)
3. Route to the appropriate specialist agent
4. Pass the specialist's response back to the user
5. Handle specialist agent failures gracefully (return error, not crash)

Classification method: use a fast model (gpt-4o-mini) for routing, expensive model (gpt-4o) for specialists.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is tool calling and how does it differ from a regular AI call?"

**Regular AI call:** You send a prompt, the AI returns text. The AI has no ability to take action — it can only generate words.

**Tool calling:** You give the AI a list of functions it can call. When the AI decides it needs information or needs to take action, it tells your code to call a specific function with specific parameters. Your code runs the function, returns the result, and the AI uses that result to continue.

The key insight: the AI decides *whether* and *when* to call a tool. Your code decides *what the tool does*. This separation is what makes tool-calling agents feel intelligent — they are not following a hardcoded decision tree, they are making contextual decisions.

---

### "How do I prevent the agent from calling tools I did not intend?"

Tool description quality is the primary control. The AI uses your description to decide when to call a tool. A vague description leads to unexpected calls.

```typescript
// ❌ Vague — AI might call this unexpectedly
{
  name: "updateData",
  description: "Updates data in the system"
}

// ✅ Precise — AI knows exactly when to use this
{
  name: "updateReminderTime",
  description: "Updates the scheduled time of an existing reminder. Call ONLY when the user explicitly asks to change when a reminder fires. Do NOT call for new reminders — use createReminder instead."
}
```

Also use the `toolChoice` parameter to restrict which tools the AI can call in specific contexts:
```typescript
toolChoice: "required"  // Must use a tool
toolChoice: "none"      // Cannot use any tool (just respond)
toolChoice: { type: "tool", toolName: "searchKnowledge" }  // Can only use this specific tool
```

---

### "Should my agent have read tools and write tools, or just one type?"

Separate them — and be much more careful with write tools. A good pattern:

**Read tools** (safe, fast, no confirmation needed):
- `getReminders(userId)` — fetches data
- `searchLessons(query)` — searches content
- `getUserProfile(userId)` — fetches preferences

**Write tools** (consequential — consider confirmation):
- `createReminder(task, time)` — creates data
- `deleteReminder(id)` — destroys data
- `sendNotification(userId, message)` — external effect

For write tools that are hard to undo (delete, send email), consider asking the AI to confirm with the user before executing:
```
In your system prompt: "Before creating, deleting, or sending anything,
summarise what you are about to do and ask the user to confirm."
```

---

### 🗂️ Update Your AGENT_CONTEXT.md

After wiring up your agent, add this block to your `AGENT_CONTEXT.md` so future coding sessions don't re-debate architecture:

```md
## AI Agent
- Agent framework: Vercel AI SDK `streamText` with tools
- Tool definitions: `lib/ai/tools/` — one file per tool
- Memory strategy: [in-session array | DB messages table | pgvector long-term]
- Max steps: [your value] — set in streamText({ maxSteps: N })
- Cost tracking: ai_generations table, logged on stream completion
- Tracing: [Helicone proxy | LangSmith | console.log wrapper in lib/ai/trace.ts]
- User scoping: all tool calls receive userId from verified session (never from client)
```

**Why this matters:** Without this, your next coding session may suggest a different agent framework or forget the maxSteps limit you set — causing runaway loops in production.

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing AI Agents

### Agent Test Suite Structure
```typescript
// /tests/agents/[name]-agent.test.ts

describe("[AgentName] Agent", () => {
  // TOOL CALL TESTS — verify the AI calls tools when expected
  it("calls createReminder tool when user requests a new reminder", async () => {
    const { toolCalls } = await runAgentWithToolTracking(
      "remind me to call mum tomorrow at 3pm"
    );
    expect(toolCalls).toContainEqual(
      expect.objectContaining({ toolName: "createReminder" })
    );
  });

  it("does NOT call deleteReminder without explicit user request", async () => {
    const { toolCalls } = await runAgentWithToolTracking(
      "show me my reminders"
    );
    expect(toolCalls).not.toContainEqual(
      expect.objectContaining({ toolName: "deleteReminder" })
    );
  });

  // PARAMETER TESTS — verify tools are called with correct parameters
  it("passes correct task and time to createReminder", async () => {
    const { toolCalls } = await runAgentWithToolTracking(
      "remind me to take medication at 8am every day"
    );
    const createCall = toolCalls.find(t => t.toolName === "createReminder");
    expect(createCall?.parameters.task).toContain("medication");
    expect(createCall?.parameters.recurrence).toBe("daily");
  });

  // RESPONSE TESTS — verify final response quality
  it("confirms what it created in plain English", async () => {
    const { response } = await runAgent("remind me about the meeting at 2pm");
    expect(response.toLowerCase()).toContain("meeting");
    expect(response.toLowerCase()).toContain("2");
  });

  // BOUNDARY TESTS — verify agent stays in scope
  it("refuses to help with tasks outside its scope", async () => {
    const { response } = await runAgent("what is the capital of France?");
    expect(response.toLowerCase()).toMatch(/only|reminder|can't help/);
  });
});
```

### Agent Debugging Checklist
```
Agent not calling a tool →
  1. Is the tool description clear enough? (most common cause)
  2. Is the tool registered in the agent's tool list?
  3. Is the user's message clearly matching the tool's trigger description?
  4. Check: is toolChoice accidentally set to "none"?

Agent calling the wrong tool →
  1. Add disambiguation to the tool description: "Call ONLY when..."
  2. Add a constraint to the system prompt about when NOT to use each tool
  3. Check: are two tool descriptions similar enough to confuse the AI?

Agent in infinite tool-calling loop →
  1. Add maxSteps: 5 to the agent configuration
  2. Check: does a tool's return value ever trigger the same tool again?

Agent not using tool results correctly →
  1. Check tool return format — is it clear and parseable?
  2. Add instruction in system prompt: "After receiving tool results, use them to..."
```

</testing_and_qa>

---

<common_patterns>

## Reusable Agent Patterns

### Pattern 1: Complete Tool-Calling Agent (Vercel AI SDK)
```typescript
// /lib/agents/reminder-agent.ts
import { streamText, tool } from "ai";
import { openai } from "@ai-sdk/openai";
import { z } from "zod";
import { createReminder, listReminders, deleteReminder } from "@/lib/tools/reminders";
import { REMINDER_AGENT_SYSTEM_PROMPT } from "@/lib/prompts/reminder-agent";

export async function runReminderAgent(
  messages: { role: "user" | "assistant"; content: string }[],
  userId: string
) {
  const result = await streamText({
    model: openai("gpt-4o"),
    system: REMINDER_AGENT_SYSTEM_PROMPT,
    messages,
    maxSteps: 5, // prevent infinite loops

    tools: {
      createReminder: tool({
        description: "Create a new reminder for the user. Call when the user asks to be reminded about something.",
        parameters: z.object({
          task: z.string().describe("What to be reminded about"),
          scheduledAt: z.string().datetime().optional().describe("When to send the reminder (ISO 8601)"),
          recurrence: z.enum(["daily", "weekly", "monthly"]).optional(),
        }),
        execute: async ({ task, scheduledAt, recurrence }) => {
          const result = await createReminder({ userId, task, scheduledAt, recurrence });
          return result ?? { error: "Failed to create reminder" };
        },
      }),

      listReminders: tool({
        description: "Fetch the user's existing reminders. Call when the user asks to see, check, or review their reminders.",
        parameters: z.object({
          filter: z.enum(["all", "upcoming", "completed"]).default("upcoming"),
        }),
        execute: async ({ filter }) => {
          return await listReminders({ userId, filter });
        },
      }),

      deleteReminder: tool({
        description: "Delete a specific reminder. Call ONLY when the user explicitly asks to delete or remove a reminder by name or ID.",
        parameters: z.object({
          reminderId: z.string().describe("The ID of the reminder to delete"),
        }),
        execute: async ({ reminderId }) => {
          const deleted = await deleteReminder({ reminderId, userId });
          return { success: deleted };
        },
      }),
    },
  });

  return result;
}
```

### Pattern 2: Tool Function with Proper Return Shape
```typescript
// /lib/tools/reminders.ts
import { db } from "@/lib/db";

/**
 * Create a new reminder for a user.
 * Returns the created reminder or null on failure.
 * Called by the reminder agent when user requests a new reminder.
 */
export async function createReminder(params: {
  userId: string;
  task: string;
  scheduledAt?: string;
  recurrence?: "daily" | "weekly" | "monthly";
}) {
  try {
    return await db.reminder.create({
      data: {
        userId: params.userId,
        task: params.task,
        scheduledAt: params.scheduledAt ? new Date(params.scheduledAt) : null,
        recurrence: params.recurrence ?? null,
      },
      select: { id: true, task: true, scheduledAt: true, recurrence: true },
    });
  } catch (error) {
    console.error("[createReminder tool]", error);
    return null;
  }
}
```

### Pattern 3: Agent with Conversation History (API Route)
```typescript
// /app/api/agent/chat/route.ts
export async function POST(req: Request) {
  const { userId } = await auth();
  if (!userId) return new Response("Unauthorized", { status: 401 });

  const { messages } = await req.json(); // Full conversation history from client
  // messages format: [{ role: "user"|"assistant", content: string }]

  const result = await runReminderAgent(messages, userId);
  return result.toDataStreamResponse();
}

// Client-side: use Vercel AI SDK useChat hook
// const { messages, input, handleSubmit } = useChat({ api: "/api/agent/chat" });
// useChat automatically manages conversation history across turns
```

### Pattern 4: Agent Tracing Wrapper — Structured Logging for Every Tool Call
```typescript
// lib/ai/trace.ts — wrap every tool call with structured logging
import { Sentry } from "@sentry/nextjs"

interface ToolTrace {
  toolName: string
  input: unknown
  output: unknown
  durationMs: number
  error?: string
  userId: string
  stepIndex: number
}

export function createTracedTools<T extends Record<string, Function>>(
  tools: T,
  context: { userId: string }
): T {
  return Object.fromEntries(
    Object.entries(tools).map(([name, fn]) => [
      name,
      async (...args: unknown[]) => {
        const start = Date.now()
        const stepIndex = Math.random() // replace with actual step counter
        try {
          const output = await fn(...args)
          const trace: ToolTrace = {
            toolName: name,
            input: args[0],
            output,
            durationMs: Date.now() - start,
            userId: context.userId,
            stepIndex,
          }
          console.log("[agent:tool]", JSON.stringify(trace))
          return output
        } catch (err) {
          Sentry.captureException(err, { extra: { toolName: name, userId: context.userId } })
          throw err
        }
      },
    ])
  ) as T
}
```

For production-grade tracing, connect to **Helicone** (`HELICONE_API_KEY`) as a proxy in front of OpenAI, or use **LangSmith** for full agent run visualisation. Both require only a base URL change in your OpenAI client config.

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Always Scope Tool Operations to the Authenticated User
```typescript
// ❌ Dangerous — agent could list any user's data if prompted cleverly
execute: async ({ userId }) => listReminders({ userId })

// ✅ Safe — always use the authenticated userId from the session, not from the AI
execute: async () => listReminders({ userId: authenticatedUserId })
// authenticatedUserId comes from auth(), not from the AI's tool call parameters
```

### Rule 2: Never Allow Destructive Tools Without Ownership Verification
```typescript
execute: async ({ reminderId }) => {
  // Always verify the resource belongs to the authenticated user
  const reminder = await db.reminder.findUnique({
    where: { id: reminderId, userId: authenticatedUserId }, // userId check!
  });
  if (!reminder) return { error: "Reminder not found" };
  return await deleteReminder({ reminderId });
}
```

### Rule 3: Limit maxSteps to Prevent Runaway Agents
```typescript
const result = await streamText({
  maxSteps: 5, // Never allow more than 5 tool calls per user message
  // Without this, a malicious prompt could cause thousands of DB writes
});
```

### Rule 4: Sanitise Before Passing User Input to Tool Parameters
When user text flows into a tool (e.g. "task" field from their message), it may contain injection attempts. Validate and sanitise before writing to the database.

### Rule 5: Cost Per Run Tracking

Multi-step agents can consume 10–50x more tokens than a single completion. Track cost per agent run:

```typescript
// In your route handler, after streamText completes
const result = await streamText({ ... })

// Access usage after stream completes
result.usage.then(usage => {
  const costUsd =
    (usage.promptTokens / 1_000_000) * 2.50 +   // gpt-4o input price
    (usage.completionTokens / 1_000_000) * 10.00  // gpt-4o output price

  console.log(`[agent:cost] userId=${userId} steps=${steps} cost=$${costUsd.toFixed(4)}`)

  // Log to your ai_generations audit table
  await db.aiGeneration.create({
    data: { userId, promptTokens: usage.promptTokens, completionTokens: usage.completionTokens, costUsd, feature: "agent" }
  })
})
```

Set a hard per-user daily budget in your rate limiter — reject requests once a user exceeds e.g. $1.00/day in agent costs.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Using the Authenticated UserId from Tool Parameters
The most dangerous agent mistake. If the AI can set userId in a tool call, an adversarial prompt can access any user's data.  
**Fix:** userId always comes from `auth()` in your route handler — never from the agent's tool parameters.

### ❌ Building the Full Agent Before Testing Individual Tools
All 5 tools built simultaneously. The agent calls one tool incorrectly and you have no idea which one.  
**Fix:** Build, test, and verify one tool before adding the next.

### ❌ No maxSteps Limit
A prompt injection attack causes the agent to call `createReminder` in an infinite loop. Thousands of DB rows are created.  
**Fix:** Always set `maxSteps: 5` or equivalent.

### ❌ Vague Tool Descriptions
The AI calls `listReminders` when the user says "how many reminders do I have?" because it was the only description that sounded close.  
**Fix:** Every tool description should state explicitly what it does AND when NOT to call it if there is ambiguity with another tool.

### ❌ Tools That Can Both Read and Write
A single `manageReminder` tool that can create, update, or delete depending on an `action` parameter. The AI misunderstands the action parameter and deletes instead of updates.
**Fix:** Separate read and write operations into distinct tools.

### ❌ Not handling tool call failures

When a tool throws, streamText surfaces it as a stream error — the client gets a broken response with no explanation.

✅ Wrap each tool execution with graceful degradation:

```typescript
// In your tool definition, catch and return structured errors
const tools = {
  searchDatabase: tool({
    description: "Search user's documents",
    parameters: z.object({ query: z.string() }),
    execute: async ({ query }) => {
      try {
        const results = await db.search(query)
        return { success: true, results }
      } catch (err) {
        // Return error as data — don't throw
        // The agent can then decide to retry or inform the user
        return {
          success: false,
          error: "Database search failed. Try a different query.",
          results: []
        }
      }
    }
  })
}
```

The agent will see the error in the tool output and can respond gracefully: "I had trouble searching your documents. Can you rephrase your question?"

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Agent System

### Add Agent Observability with LangSmith
Track every agent run, every tool call, and every LLM response in a searchable dashboard:
```
Ask your AI agent: "Add LangSmith tracing to my agent.
Capture: input message, tool calls with parameters, tool results, final response, total tokens, latency."
```

### Add Human Approval for Irreversible Actions
```typescript
// Instead of executing immediately, queue for approval
execute: async ({ reminderId }) => {
  // Store pending action instead of executing
  await db.pendingAction.create({
    data: { type: "delete_reminder", resourceId: reminderId, userId, status: "pending" }
  });
  return { status: "pending_approval", message: "Deletion queued for your approval" };
  // User approves via a separate UI action that calls the real delete
}
```

### Add Semantic Memory with Vector Search
When agents need to remember user history beyond recent conversation turns:
```
Ask your AI agent: "Add semantic memory to the [AGENT NAME].
Before each response, search for relevant past interactions using the query:
[user's current message].
Inject top 3 results into the system prompt as 'relevant context from past conversations'."
See skill-rag.md for the retrieval implementation.
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Conversational Agent with 4 Tools
```
Tools built (in order, one per session):
1. listReminders(filter) — read
   Test: "show me my reminders" → agent calls listReminders({filter:"upcoming"}) ✅

2. createReminder(task, scheduledAt, recurrence) — write
   Test: "remind me to call mum Sunday 3pm" → calls createReminder correctly ✅

3. deleteReminder(reminderId) — write, destructive
   Test: "delete the mum reminder" → agent asks for confirmation first ✅
   (system prompt includes: "confirm before deleting")

4. snoozeReminder(reminderId, newTime) — write
   Test: "push the mum reminder to 5pm" → finds reminder then snoozes ✅

Total build time: 4 sessions × 30 minutes = 2 hours
Key decision: userId was never exposed as a tool parameter — always from auth()
```

### Case Study 2: EdTech App — Multi-Agent with Router
```
Feature: student asks a question, gets personalised help

Router agent (gpt-4o-mini — cheap, fast):
  Classifies: is this a content question? progress question? or navigation?

Specialist: ContentExpert agent (gpt-4o — quality matters)
  Tools: searchLessons(query), getRelatedTopics(topic)
  Answers curriculum questions with relevant lesson references

Specialist: ProgressAdvisor agent (gpt-4o-mini — simpler task)
  Tools: getStudentProgress(userId), getRecommendedLesson(level)
  Advises on what to study next based on progress

Result:
  - Content questions answered with lesson-specific accuracy
  - Progress questions answered quickly (cheap model)
  - 30% lower cost than routing everything to gpt-4o
  - Response quality equal to a single gpt-4o agent for content
    (better, because the content agent had more focused context)
```

</real_world_examples>

</skill_document>
