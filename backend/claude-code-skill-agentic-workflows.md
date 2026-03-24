# Skill: Agentic Workflows

```json
{
  "skill_id": "agentic-workflows",
  "category": "Architecture & Backend",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Agentic Workflows — Building Multi-Step AI Pipelines That Actually Finish</title>

<overview>

## What This Skill Enables
- Build AI workflows where multiple steps happen in sequence or in parallel, each dependent on the last
- Handle the most common failure modes: AI hallucinations mid-pipeline, timeouts, and broken state
- Design agents that can use tools (search, database, external APIs) not just generate text

## Why It Matters for Vibe Coders
A single AI call (user asks → AI answers) is easy. But most real AI products need chains: understand intent → fetch relevant data → generate response → save result → trigger notification. Each step can fail. This skill teaches you how to build these chains reliably, so your product does not silently break at step 3 without you knowing.

## When to Use This Skill
- When your feature requires more than one AI call to complete
- When your AI needs to access external data (database, web, user history) before responding
- When you need AI to take actions (create records, send messages, schedule tasks) not just generate text
- When a single AI call regularly hits timeout limits

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "ai_provider": "[REPLACE: e.g. OpenAI / Anthropic / Google]",
    "ai_model": "[REPLACE: e.g. gpt-4o / claude-3-5-sonnet / gemini-1.5-pro]",
    "orchestration_library": "[REPLACE: e.g. Vercel AI SDK / LangChain / custom / none yet]",
    "database": "[REPLACE: e.g. Supabase PostgreSQL]",
    "background_job_runner": "[REPLACE: e.g. Inngest / Trigger.dev / none yet]",
    "workflow_to_build_today": "[REPLACE: describe the full pipeline in plain English, e.g. User sends message → extract intent → fetch user context from DB → generate personalised lesson → save lesson → notify user]",
    "existing_workflow_files": "[REPLACE: list any existing files, e.g. /lib/ai.ts, /lib/workflows/lesson.ts]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Agentic Workflows

### Mental Model 1: The Assembly Line
Think of a workflow as a factory assembly line:
- Each station does ONE specific job
- The output of station 1 becomes the input of station 2
- If station 2 breaks, the line stops — it does not try to guess what station 1 would have produced
- A supervisor (orchestrator) watches the whole line and handles breakdowns

Your job is to design the stations, define what flows between them, and handle what happens when any station breaks.

### Mental Model 2: Steps vs Agents
| Concept | What It Means | When to Use |
|---|---|---|
| **Step** | A single AI call with a defined input → output | When you know exactly what needs to happen |
| **Agent** | An AI that decides which tools to call and in what order | When the path through the workflow depends on the user's input |
| **Tool** | A function the AI can choose to call (search, DB query, API call) | When the AI needs external information to complete its task |

**For most vibe-coded products:** Start with Steps. Only upgrade to Agents when Steps are insufficient.

### Mental Model 3: State Is Your Enemy
The biggest problem in agentic workflows is lost state — the workflow forgets what happened at step 1 by step 3. Always pass state explicitly between steps. Never assume the AI remembers.

</mental_models>

---

<system_design_breakdown>

## Anatomy of an Agentic Workflow

```
USER INPUT
     │
     ▼
┌──────────────────────────────────────────────────┐
│  ORCHESTRATOR                                    │
│  - Receives input                                │
│  - Manages state across steps                   │
│  - Decides: next step? retry? fail gracefully?  │
└───────────────────────┬──────────────────────────┘
                        │
          ┌─────────────┼──────────────┐
          ▼             ▼              ▼
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │  STEP 1  │  │  STEP 2  │  │  STEP 3  │
    │ (Intent  │  │ (Fetch   │  │(Generate │
    │ Extract) │  │ Context) │  │ Content) │
    └──────────┘  └──────────┘  └──────────┘
          │             │              │
          └─────────────┼──────────────┘
                        │
                        ▼
              ┌─────────────────┐
              │  STATE OBJECT   │
              │  {              │
              │   intent: ...,  │
              │   context: ..., │
              │   output: ...,  │
              │   step: 3,      │
              │   status: "ok"  │
              │  }              │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  SAVE + RETURN  │
              │  (DB + Response)│
              └─────────────────┘
```

### Tool Calling Pattern
When your AI needs to retrieve information before responding:

```
AI receives message
     │
     ▼
Does AI need external data?
     │
  YES: AI calls a tool (DB query, API call, search)
     │
     ▼
Tool runs and returns data
     │
     ▼
AI uses tool result to form final response
     │
  NO: AI responds directly
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Build one step of the pipeline. Test it. Then add the next step. -->

## Step 1 — Map Your Workflow on Paper First
Do not touch code. Write out every step your workflow needs.

**Template:**
```
Workflow Name: [e.g. Personalised Lesson Generation]

Input: [What does the user send? e.g. { topic: string, userId: string }]

Step 1: [Name] → [What happens] → [Output]
Step 2: [Name] → [What happens] → [Output]  
Step 3: [Name] → [What happens] → [Output]
...

Final Output: [What does the user receive? e.g. { lessonId, content, estimatedReadTime }]

Failure modes:
- If Step 1 fails: [what should happen?]
- If Step 2 fails: [what should happen?]
```

**Verify:** You can trace one example request all the way through. Each step's output is the next step's input.

---

## Step 2 — Build the State Object
Define the data structure that flows through your entire pipeline.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
I am building a workflow called: [WORKFLOW NAME]
Here are the steps: [PASTE YOUR STEP MAP]

Create a TypeScript type (or Python Pydantic model) for the workflow state object.
It should hold: input, the output of each step, current step number, status (pending/running/complete/failed), and any error message.
Do not write any workflow logic yet. Just the state type.
```

**Verify:** The state type has a field for every piece of data the workflow produces or needs.

---

## Step 3 — Build Step 1 as a Standalone Function
**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Workflow state type: [PASTE STATE TYPE]

Build ONLY Step 1 of the workflow: [STEP 1 NAME]
Input: [describe input]
Expected output: [describe output]
This must be a pure function that:
- Takes the current state as input
- Returns an updated state object
- Handles its own errors (catches + sets status to "failed" + sets error message)
- Does NOT call any other step

Show me how to test this step in isolation.
```

**Verify:** Call Step 1 with a test input. It returns the expected output or a clear error state.

---

## Step 4 — Add Step 2, Then Step 3
One at a time. Same pattern as Step 3. Each step is a function that takes state and returns updated state.

**Verify each step in isolation before connecting them.**

---

## Step 5 — Build the Orchestrator
Only after all steps work individually.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
I have these working step functions: [LIST STEP FUNCTION NAMES]
Each takes the workflow state and returns updated state.

Build the orchestrator function that:
1. Takes the initial user input
2. Creates the initial state object
3. Runs each step in sequence
4. If any step sets status to "failed", stops and returns the error state
5. Logs the current step number at each stage
6. Returns the final state object on success

Do not add retry logic yet. Just sequential execution with error stopping.
```

**Verify:** Run the full workflow with a test input. Check the logs show each step completing in order.

---

## Step 6 — Add Retry Logic and Timeouts
**Prompt your AI agent:**
```
The orchestrator is working. Now add:
1. Retry logic for AI calls only (max 2 retries with 1 second delay)
2. A 30-second timeout for the full workflow
3. If timeout is hit, set status to "failed" with message "Workflow timed out"

Do not retry database steps — only steps that call [AI_PROVIDER].
```

**Verify:** Simulate a slow AI response. Confirm retry fires correctly and timeout stops the workflow.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am building an agentic workflow for my app.
Project context: [PASTE context_anchor JSON]

Today I am working on: [SPECIFIC STEP OR THE ORCHESTRATOR]
Files already created: [LIST FILES]

Before writing any code, tell me:
1. What file you will create or modify
2. What the function signature will look like
3. Any dependencies you need that are not yet installed
```

### Workflow Mapper Prompt
```
I want to build this feature for my [APP TYPE] app:
[DESCRIBE FEATURE IN PLAIN ENGLISH]

Map this into a step-by-step workflow:
1. Each step as a function with clear input → output
2. Identify which steps require AI calls
3. Identify which steps require database calls
4. Identify which steps could fail and why
5. Suggest the correct order of execution

Do not write any code. Return a structured workflow map only.
```

### Tool Function Prompt
```
I need a tool function that [DESCRIBE WHAT IT DOES — e.g. fetches a user's last 5 lessons from the DB].
This tool will be called by an AI agent during a workflow.

Requirements:
- Input: [DESCRIBE INPUT]
- Output: [DESCRIBE OUTPUT]  
- Return null (not throw) if the data does not exist
- Include a JSDoc comment explaining when this tool should be called
- Database: [YOUR DATABASE]
Context: [PASTE context_anchor JSON]
```

### Workflow Debug Prompt
```
My workflow is failing at Step [NUMBER]: [STEP NAME].
Here is the state object at the point of failure:
[PASTE STATE OBJECT]

Here is the step function code:
[PASTE CODE]

Here is the error:
[PASTE ERROR]

Identify the root cause. Fix only this step. Do not change the orchestrator or other steps.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Should I use LangChain or build my own workflow?"

**Use LangChain (or Vercel AI SDK) if:**
- You are just starting with agentic workflows
- You want built-in tool calling, memory, and streaming
- Your workflow is a common pattern (RAG, chat with tools, summarisation)

**Build your own if:**
- Your workflow is highly custom and LangChain is forcing you to work around it
- You need full control over retry logic, state, and error handling
- The LangChain abstraction is confusing your AI agent

**Honest recommendation for most vibe coders:** Vercel AI SDK for Next.js apps, LangChain for Python apps. Both have excellent documentation and are easy to prompt AI agents with.

---

### "What is the difference between a chain and an agent?"

**Chain:** Fixed sequence of steps. Step 1 always goes to Step 2. No decisions made by the AI about the path.  
*Use when:* You know exactly what needs to happen in what order.

**Agent:** The AI decides which tools to call based on the situation. The path through the workflow varies per request.  
*Use when:* Different user inputs require different actions (e.g. sometimes the AI needs to search the web, sometimes it queries the DB, sometimes neither).

**Start with chains.** Only add agent decision-making when you genuinely need it. Agents are harder to debug and more expensive per request.

---

### "How do I handle AI calls that take too long?"

Set a timeout. Never let your server wait forever for an AI response.

For user-facing features: if it takes more than 3 seconds, show a loading state. If it takes more than 30 seconds, fail gracefully and tell the user to try again.

For background jobs (not user-facing): you have more time. Use Inngest or Trigger.dev which handle long-running jobs natively.

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing Agentic Workflows

### Test Strategy: Bottom Up
Always test each step function before testing the full pipeline.

```
Test Step 1 alone ✅
Test Step 2 alone ✅
Test Step 3 alone ✅
Test Step 1 → Step 2 connected ✅
Test full pipeline ✅
```

### Unit Test Template for a Workflow Step
```typescript
// Test file: /tests/workflows/step-intent-extraction.test.ts
import { extractIntent } from "@/lib/workflows/steps/extract-intent";

describe("extractIntent step", () => {
  it("extracts reminder intent from natural language", async () => {
    const inputState = {
      input: { text: "remind me to call mum tomorrow at 3pm", userId: "test-user" },
      step: 1,
      status: "running",
    };

    const result = await extractIntent(inputState);

    expect(result.status).toBe("ok");
    expect(result.intent.action).toBe("create_reminder");
    expect(result.intent.scheduledAt).toBeDefined();
  });

  it("handles empty input gracefully", async () => {
    const inputState = {
      input: { text: "", userId: "test-user" },
      step: 1,
      status: "running",
    };

    const result = await extractIntent(inputState);
    expect(result.status).toBe("failed");
    expect(result.error).toBeDefined();
  });
});
```

### Debugging Loop for Workflows
```
Workflow fails →
  1. Check the returned state object: what is status? what is error? what is step?
  2. The step number tells you exactly where it broke
  3. Reproduce that step in isolation with the exact state it received
  4. Paste the isolated test + error to your AI agent:
     "Step [N] called [STEP_FUNCTION] with this state: [PASTE STATE]
      It returned this error: [PASTE ERROR]
      Fix only this step function. Return the corrected function."
  5. Re-test that step in isolation → then re-test full pipeline
```

### Common Agentic Workflow Errors

| Error | Meaning | Fix |
|---|---|---|
| `AI returned empty response` | The LLM returned nothing | Add fallback: if response is empty, retry once, then return error state |
| `Cannot parse AI output` | AI returned text when you expected JSON | Use structured output (JSON mode) or add a parsing step with fallback |
| `Workflow exceeded time limit` | Pipeline too slow end-to-end | Break into smaller steps, add streaming, or move to background job |
| `Step 3 received undefined context` | State was not passed correctly from Step 2 | Check that Step 2 explicitly includes all previous state fields in its return |
| `Tool not found` | AI tried to call a tool that does not exist | Verify tool name matches exactly — case-sensitive |

</testing_and_qa>

---

<common_patterns>

## Reusable Workflow Patterns

### Pattern 1: Sequential Workflow Orchestrator
```typescript
// /lib/workflows/orchestrator.ts
type WorkflowState = {
  input: Record<string, unknown>;
  step: number;
  status: "pending" | "running" | "complete" | "failed";
  error?: string;
  [key: string]: unknown; // Step outputs stored here
};

type StepFunction = (state: WorkflowState) => Promise<WorkflowState>;

export async function runWorkflow(
  initialInput: Record<string, unknown>,
  steps: StepFunction[]
): Promise<WorkflowState> {
  let state: WorkflowState = {
    input: initialInput,
    step: 0,
    status: "pending",
  };

  for (let i = 0; i < steps.length; i++) {
    state = { ...state, step: i + 1, status: "running" };
    console.log(`[Workflow] Running step ${i + 1}/${steps.length}`);

    state = await steps[i](state);

    if (state.status === "failed") {
      console.error(`[Workflow] Failed at step ${i + 1}:`, state.error);
      return state;
    }
  }

  return { ...state, status: "complete" };
}
```

### Pattern 2: AI Step with Structured Output
```typescript
// /lib/workflows/steps/extract-intent.ts
import { WorkflowState } from "../orchestrator";
import { z } from "zod";
import { callAIStructured } from "@/lib/ai";

const IntentSchema = z.object({
  action: z.enum(["create_reminder", "list_reminders", "delete_reminder", "unclear"]),
  task: z.string().optional(),
  scheduledAt: z.string().datetime().optional(),
});

export async function extractIntent(state: WorkflowState): Promise<WorkflowState> {
  try {
    const result = await callAIStructured(
      `Extract the intent from: "${state.input.text}"`,
      IntentSchema
    );

    return { ...state, intent: result, status: "running" };
  } catch (error) {
    return { ...state, status: "failed", error: `Intent extraction failed: ${error}` };
  }
}
```

### Pattern 3: Tool-Calling Pattern
```typescript
// /lib/tools/index.ts — Define tools the AI can call
export const tools = {
  getUserContext: {
    description: "Fetch the user's learning history and preferences",
    parameters: z.object({ userId: z.string() }),
    execute: async ({ userId }: { userId: string }) => {
      return await db.user.findUnique({
        where: { id: userId },
        select: { level: true, completedLessons: true, preferences: true },
      });
    },
  },
  saveLesson: {
    description: "Save a generated lesson to the database",
    parameters: z.object({ userId: z.string(), content: z.string(), topic: z.string() }),
    execute: async (params: { userId: string; content: string; topic: string }) => {
      return await db.lesson.create({ data: params });
    },
  },
};
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Validate AI Output Before Using It
Never use AI-generated data directly in database operations or downstream calls without validation.
```typescript
// ❌ Dangerous — AI output used directly
await db.reminder.create({ data: aiOutput });

// ✅ Safe — validate before using
const validated = ReminderSchema.safeParse(aiOutput);
if (!validated.success) {
  return { ...state, status: "failed", error: "AI output failed validation" };
}
await db.reminder.create({ data: validated.data });
```

### Rule 2: Prevent Prompt Injection in Tool-Calling Agents
When user input flows into an AI prompt that also has tool access, an attacker can craft input that tricks the AI into calling tools with malicious parameters.
```typescript
// Always sanitise user input before including in agent prompts
const safeInput = userInput
  .slice(0, 1000) // Limit length
  .replace(/[<>]/g, "") // Remove potential HTML/XML injection
  // Add your own sanitisation as needed
```

### Rule 3: Limit Tool Permissions
Each tool should only be able to do what it needs to do. A "read user context" tool should never have write access.

### Rule 4: Log Every Workflow Run
Every workflow execution should produce a structured log entry including: userId, workflowName, stepReached, finalStatus, durationMs, error (if any). This is your audit trail.

### Rule 5: Set Maximum Step Counts
Prevent infinite loops in agent-style workflows:
```typescript
const MAX_STEPS = 10;
if (stepCount > MAX_STEPS) {
  return { ...state, status: "failed", error: "Max step limit exceeded" };
}
```

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Building the Full Pipeline Before Testing Steps
You build all 5 steps and the orchestrator in one go. Something fails and you have no idea which step is broken.  
**Fix:** Test every step function in isolation before wiring them together.

### ❌ Letting State Mutate In Place
```typescript
// ❌ Mutates state — previous step's data can be lost or overwritten
state.intent = result;
return state;

// ✅ Spreads state — all previous fields preserved
return { ...state, intent: result };
```

### ❌ Not Handling Empty AI Responses
The AI provider returns an empty string or `null`. Your next step tries to parse it and crashes.  
**Fix:** Every AI call step must check the response is non-empty before returning success state.

### ❌ Making Irreversible Tool Calls Without Confirmation
Your agent sends an email, deletes a record, or charges a payment card as part of a workflow — and it was a test run.  
**Fix:** Separate read tools (always safe) from write tools (require confirmation). Never run write tools in development without a `isDryRun` flag.

### ❌ Using Agent-Style Workflows for Simple Tasks
You use a full LangChain agent for something that is just "format this text into JSON". Agents are slow, expensive, and unpredictable.  
**Fix:** Use an agent only when the AI genuinely needs to make decisions about which tools to call. For everything else, use a simple sequential workflow.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Agentic Workflows

### Move Long Workflows to Background Jobs
Workflows that take more than 10 seconds should run in the background, not in an HTTP request.
```
Tools to evaluate:
- Inngest (recommended for Next.js — event-driven, excellent DX)
- Trigger.dev (excellent for complex multi-step jobs)
- BullMQ (if you are self-hosting with Redis)
```

Ask your AI agent: "Convert the [WORKFLOW NAME] workflow to run as an Inngest function triggered by the event [EVENT_NAME]."

### Add Workflow Observability
```
Ask your AI agent: "Add LangSmith tracing to all AI calls in the workflow.
Log: input, output, latency, model, tokens used for each step."
```

### Human-in-the-Loop Approval Steps
For high-stakes actions (sending emails, processing payments, deleting data), add a pause step that waits for human confirmation before proceeding.
```
Pattern: Workflow runs to Step N → pauses → stores state → sends approval notification → 
user approves → workflow resumes from Step N+1
Inngest and Trigger.dev both support this natively.
```

### Parallel Step Execution
When steps are independent (do not need each other's output), run them in parallel:
```typescript
const [contextResult, historyResult] = await Promise.all([
  fetchUserContext(state),
  fetchUserHistory(state),
]);
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Conversational Reminder App — Intent-to-Reminder Pipeline
**User input:** "Hey, remind me to review my team's PRs every Monday at 10am"

```
Step 1: extractIntent
  Input: raw text
  AI task: identify action=create_reminder, extract task, extract recurrence pattern
  Output: { action, task, recurrence: "weekly", dayOfWeek: "monday", time: "10:00" }

Step 2: resolveDateTime  
  Input: { recurrence, dayOfWeek, time }
  Logic: calculate next occurrence of Monday 10am from now
  Output: { scheduledAt: ISO datetime, nextOccurrences: [...] }

Step 3: checkConflicts
  Input: { userId, scheduledAt }
  DB query: does user already have a reminder at this time?
  Output: { hasConflict: bool, conflictingReminder?: {...} }

Step 4: saveReminder
  Input: full resolved reminder data
  DB write: create reminder + create recurrence schedule
  Output: { reminderId, confirmationText }

Step 5: generateConfirmation
  Input: { task, scheduledAt, recurrence }
  AI task: generate a natural language confirmation message
  Output: "Got it! I'll remind you to review your team's PRs every Monday at 10am,
           starting this Monday."
```

**Total workflow time:** ~3.5 seconds. Steps 2 and 3 run in parallel (no dependency on each other).

---

### Case Study 2: EdTech App — Adaptive Lesson Generation
```
Step 1: loadStudentProfile
  DB query: get student level, completed topics, learning style preference
  Output: { level: "intermediate", completedTopics: [...], preferredStyle: "visual" }

Step 2: selectTopicStrategy  
  AI task: given profile + requested topic, decide: introduce new concept? reinforce existing? challenge with advanced material?
  Output: { strategy: "reinforce", conceptFocus: "...", difficulty: 6/10 }

Step 3: generateLesson (streamed)
  AI task: generate lesson content matching strategy + style
  Output: streaming markdown lesson content

Step 4: saveLesson
  DB write: save lesson with metadata
  Output: { lessonId }

Step 5: updateStudentProgress
  DB write: record lesson started, update topic exposure
  Output: { updated: true }
```

**Key pattern:** Step 3 streams directly to the user while Steps 4 and 5 run after streaming completes — so the user sees content immediately without waiting for the database writes.

</real_world_examples>

</skill_document>
