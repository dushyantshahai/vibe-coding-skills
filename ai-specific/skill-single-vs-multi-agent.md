# Skill: Single-Agent vs Multi-Agent Architecture

```json
{
  "skill_id": "single-vs-multi-agent",
  "category": "AI-Specific",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Single-Agent vs Multi-Agent — Making the Right Architecture Decision Before You Build</title>

<overview>

## What This Skill Enables
- Decide confidently whether your AI feature needs one agent or many — before writing a single line of code
- Understand the three architectural patterns (single agent, sequential multi-agent, parallel/dynamic multi-agent) and which problems each solves
- Avoid the two most expensive mistakes in AI product development: under-building a system that needs more and over-engineering one that does not

## Why It Matters for Vibe Coders
This is the highest-leverage decision you make when building an AI product. Choose wrong and you either hit hard ceilings immediately (single agent trying to do too much) or build something too complex to debug, too expensive per request, and too slow to iterate on (multi-agent overkill). This skill is the decision layer — you consult it before building any AI feature, then use `skill-agentic-workflows.md` or `skill-ai-agent-design.md` to execute.

## When to Use This Skill
- Before designing any new AI feature — read this first, then pick your build pattern
- When your single-agent system is producing inconsistent or low-quality outputs despite good prompting
- When a user-facing AI feature is becoming too slow because it is doing too much in one call
- When you are tempted to add a second AI call to an existing feature

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "feature_to_design": "[REPLACE: plain English description of what the AI feature needs to do]",
    "user_facing_latency_budget": "[REPLACE: how long can the user wait? e.g. <2s / <5s / background OK]",
    "input_sources": "[REPLACE: what data does this feature need? e.g. user message, DB records, uploaded file]",
    "expected_output": "[REPLACE: what should the feature produce? e.g. structured reminder, formatted lesson, chat reply]",
    "ai_provider": "[REPLACE: e.g. OpenAI / Anthropic / Google]",
    "existing_ai_features": "[REPLACE: list any AI features already built in the app]",
    "cost_sensitivity": "[REPLACE: e.g. low — B2C free tier / medium — paid users / high — every call counts]"
  }
}
```

</context_anchor>

---

<mental_models>

## The Three Patterns and When Each Earns Its Cost

### Pattern 1: Single Agent
```
USER INPUT ──→ [ONE LLM CALL] ──→ OUTPUT
```
One prompt. One model call. One response. The AI does everything in a single context window.

**Use when:**
- The task has a clearly defined input and output
- The output fits in one context window (~4,000 tokens or less)
- The task does not require checking or validating its own work
- Response time matters (every additional agent adds 1–5 seconds)
- You are building an MVP and want to validate the concept cheaply

**Real examples:**
- Parse a natural language reminder into structured fields
- Generate a short explanation of a concept
- Classify user intent from a message
- Summarise a text passage
- Draft a short email

---

### Pattern 2: Sequential Multi-Agent (Orchestrator + Task Agents)
```
USER INPUT
    │
    ▼
[ORCHESTRATOR] decides task breakdown
    │
    ├──→ [AGENT A: Task 1] ──→ result A
    │                               │
    └──→ [AGENT B: Task 2] ◄────────┘ (uses result A)
              │
              ▼
         [FINAL OUTPUT]
```
A coordinator agent breaks the problem down. Specialist sub-agents handle each part in sequence. Each agent's output becomes the next agent's input.

**Use when:**
- The task has distinct phases that require different expertise or instructions
- An earlier step's output meaningfully changes what the next step should do
- Quality improves by having one agent check another's work (critic pattern)
- The full task is too complex or too long for a single context window
- Different steps benefit from different models (fast/cheap for classification, powerful for generation)

**Real examples:**
- Research → Draft → Review → Polish (content pipeline)
- Extract intent → Fetch context → Generate personalised response
- Generate lesson → Evaluate difficulty → Adjust for student level
- Parse user query → Determine tool calls → Execute tools → Synthesise answer

---

### Pattern 3: Parallel / Dynamic Multi-Agent
```
USER INPUT
    │
    ▼
[ORCHESTRATOR] analyses task
    │
    ├──→ [AGENT A] ──→ result A ──┐
    ├──→ [AGENT B] ──→ result B ──┼──→ [SYNTHESIS AGENT] ──→ OUTPUT
    └──→ [AGENT C] ──→ result C ──┘
    (all run simultaneously)
```
Multiple agents run in parallel on independent sub-tasks. An orchestrator spawns agents dynamically based on what the task requires.

**Use when:**
- Sub-tasks are genuinely independent (no data dependency between them)
- Latency is critical and parallelism provides a real speedup
- The number of sub-tasks varies based on input (dynamic spawning)
- You need multiple perspectives or approaches synthesised into one answer

**Real examples:**
- Research 5 topics simultaneously, synthesise into one report
- Generate 3 different lesson difficulty levels in parallel, return the best one
- Check a piece of code against multiple quality criteria simultaneously
- Evaluate a student answer from multiple pedagogical angles at once

</mental_models>

---

<system_design_breakdown>

## The Decision Tree

```
START: I need an AI feature
         │
         ▼
Can this be done in ONE clear step
with a well-defined input and output?
         │
    YES  │  NO
         │
┌────────┴─────────────────────────────────┐
│                                          │
▼                                          ▼
Single Agent                    Does each step depend on
(start here,                    the previous step's output?
upgrade only if                            │
it fails)                             YES  │  NO
                                           │
                                ┌──────────┴──────────┐
                                │                     │
                                ▼                     ▼
                          Sequential            Parallel
                          Multi-Agent           Multi-Agent
                          (most common          (only when steps
                          multi-agent           are truly independent
                          pattern)              AND latency matters)
```

## Complexity vs Value Matrix

| Architecture | Complexity | Cost per Request | Debuggability | When Value Justifies It |
|---|---|---|---|---|
| Single Agent | Low | $ | High | Almost always — start here |
| Sequential Multi | Medium | $$ | Medium | When quality clearly improves |
| Parallel/Dynamic | High | $$$ | Low | When latency is the bottleneck |

## The Upgrade Triggers

Start with a single agent. Upgrade only when you hit one of these specific walls:

```
Wall 1: Context length overflow
  Single agent prompt + context > model's context window
  → Split into sequential agents: fetch context agent + generation agent

Wall 2: Quality ceiling
  Single agent produces inconsistent or wrong outputs despite prompt engineering
  → Add a critic/checker agent after the generation agent

Wall 3: Conflicting instructions
  Prompt has too many responsibilities — "be creative" AND "be factually accurate"
  → Split into specialist agents: each with one clear role

Wall 4: Latency from sequential tasks
  Multiple independent sub-tasks running one-by-one causes unacceptable delay
  → Parallelize genuinely independent sub-tasks

Wall 5: Different quality/cost trade-offs per step
  Some steps need GPT-4o; others are fine with GPT-4o-mini
  → Multi-agent lets you assign the right model to each step
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Always prototype as a single agent first. Multi-agent is a deliberate upgrade, not the default. -->

## Step 1 — Map Your Feature's Requirements

Before any architecture decision, answer these questions precisely:

```
Feature mapping worksheet:

1. INPUT: What data does this feature receive?
   [Free text / structured data / file / combination]

2. OUTPUT: What must this feature produce?
   [Text / structured JSON / action / combination]

3. STEPS: List every distinct cognitive step needed
   (e.g. "understand intent → fetch relevant data → generate response → validate output")

4. DEPENDENCIES: Does Step N need Step N-1's output?
   [Draw arrows between steps that depend on each other]

5. LATENCY: How long can the user wait?
   [<1s = streaming only | 1-5s = single or sequential | >5s = background job]

6. QUALITY BAR: How good does the output need to be?
   [Good enough for MVP / high quality / mission critical]
```

---

## Step 2 — Apply the Decision Tree

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

I am designing this AI feature:
[DESCRIBE THE FEATURE]

Steps I have identified:
1. [STEP 1]
2. [STEP 2]
3. [STEP 3]

Help me decide: single agent, sequential multi-agent, or parallel multi-agent?

Evaluate against:
1. Can this be done in one LLM call with acceptable quality? (try single first)
2. Are there distinct phases that benefit from specialist handling?
3. Are any steps independent enough to run in parallel?
4. What is the cost difference between the options?
5. What is the complexity/debuggability difference?

Recommend the SIMPLEST architecture that meets the quality bar.
Do not recommend multi-agent unless single agent clearly cannot do the job.
```

---

## Step 3 — Prototype as Single Agent First

Even if you expect to need multi-agent, build the single agent version first. It:
- Validates your prompt and the AI model's capability for this task
- Gives you a quality baseline to compare against
- Often reveals that multi-agent was unnecessary
- Takes 20 minutes instead of 2 hours

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Build a prototype single-agent version of [FEATURE].
Keep it minimal — one function, one AI call, hardcoded test input.

Function signature: async function [featureName](input: [TYPE]): Promise<[OUTPUT_TYPE]>
AI call: use [MODEL] with this task: [DESCRIBE IN ONE SENTENCE]
Output format: [DESCRIBE OR PASTE SCHEMA]

This is a prototype — no error handling, no DB, no auth.
I just want to see if a single call can produce acceptable output.
Show me the test to run after generating the code.
```

**Verify:** Run the prototype with 5 different real inputs. Does it produce acceptable output? If yes — you do not need multi-agent. If no — identify specifically what is failing, then apply an upgrade trigger from Step 2.

---

## Step 4 — Upgrade to Multi-Agent (Only If Prototype Fails)

Only proceed here if the single-agent prototype hit a specific wall from the upgrade triggers.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Single agent prototype result: [DESCRIBE WHAT FAILED AND WHY]
Upgrade trigger identified: [WHICH WALL DID YOU HIT?]

Design a [SEQUENTIAL / PARALLEL] multi-agent system to address this specific failure.

For each agent, specify:
- Name and responsibility
- Input (what it receives)
- Output (what it produces)  
- Model to use (and why this model for this step)
- Estimated latency

Do not write any code yet. Return the agent design as a structured plan.
```

**Verify:** The plan addresses the specific failure mode. Each agent has a single clear responsibility. The total expected latency is acceptable given the user's wait budget.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Architecture Decision Prompt
```
I need to design an AI feature for my app.
Project context: [PASTE context_anchor JSON]

Feature description: [DESCRIBE IN PLAIN ENGLISH]

Recommend an architecture (single agent / sequential multi-agent / parallel multi-agent).
For your recommendation:
1. Show me the simplest architecture that meets the requirements
2. Show me what the prompt structure would look like at a high level
3. Estimate: latency, cost per call, complexity to debug
4. Tell me exactly what would need to change to upgrade to the next complexity tier

Start with the simplest option. Only recommend complexity if the task clearly requires it.
```

### Single Agent Evaluation Prompt
```
I built a single-agent prototype for [FEATURE].
Here are 5 test outputs I received: [PASTE OUTPUTS]

Evaluate:
1. Does the output consistently meet the quality bar? (yes/no for each)
2. What specific failure modes do you see?
3. Are these failures fixable with better prompting, or do they indicate a structural limitation?
4. If structural: which upgrade trigger applies and what architecture would fix it?

Be direct. If single agent can work with better prompting, say so — do not recommend multi-agent unnecessarily.
```

### Multi-Agent Design Prompt
```
Design a multi-agent system for [FEATURE].
Pattern: [SEQUENTIAL / PARALLEL]

For each agent in the system:
1. Agent name and single responsibility
2. Input schema (what it receives from previous agent or user)
3. Output schema (what it passes to next agent or user)
4. System prompt (one paragraph — focused, single role)
5. Recommended model and why
6. Estimated latency

Also design:
- The orchestrator: how does it coordinate the agents?
- Error handling: what happens if one agent fails?
- State object: what data flows through the entire pipeline?

Return as a structured design document, not code.
```

### Architecture Review Prompt
```
Review this existing AI feature architecture:
[DESCRIBE OR PASTE CURRENT IMPLEMENTATION]

Evaluate:
1. Is it over-engineered? Could a simpler architecture produce the same quality?
2. Is it under-engineered? Is it hitting quality or reliability walls?
3. Is each agent doing exactly one thing, or are responsibilities mixed?
4. Is the model choice appropriate for each step?
5. Are there any agents that could be merged or eliminated?

Give me a concrete recommendation: simplify / keep / upgrade. With justification.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is an agent, exactly?"

An agent is a piece of code that uses an AI model to make decisions or take actions. The simplest agent is one AI call that reads input and produces output. A more complex agent can:
- Call tools (search the web, query your database, call an API)
- Make decisions about what to do next based on what it learned
- Call other agents

The word "agent" sounds intimidating but the concept is simple: it is a function that uses an AI model to do its work.

---

### "When is multi-agent worth the extra complexity?"

Multi-agent earns its cost in three specific situations:

**1. The task genuinely has distinct phases.** Generating a lesson plan is different from generating the actual lesson content. A specialist agent for each phase produces noticeably better results than one general agent doing both.

**2. You need a critic.** For high-stakes outputs (medical content, legal summaries, complex code), having a second agent review the first agent's work catches errors the first agent would miss about its own output.

**3. You need parallel speed.** If you genuinely need to do 5 independent things and parallelising them cuts latency from 15 seconds to 3 seconds — and that 3-second experience is meaningfully better for users — then parallel agents earn their complexity.

In every other situation, a well-prompted single agent is faster, cheaper, and easier to debug.

---

### "The AI agent design pattern I see everywhere — is it actually good?"

The "orchestrator + 10 sub-agents" pattern you see in demos and tutorials is often over-engineered for real products. It emerged from AI research contexts where the goal was capability, not product quality. For a product shipped to real users:

- More agents = more latency
- More agents = more cost per request
- More agents = more failure points to debug
- More agents = more surface area for hallucinations to compound

The best AI products tend to use the simplest architecture that meets the quality bar — and raise the bar incrementally as user feedback demands it.

</vibe_coder_bridge>

---

<testing_and_qa>

## Evaluating Your Architecture Choice

### Quality Evaluation Framework
Run every AI architecture candidate through this evaluation before committing to it:

```
Test set: prepare 10 representative inputs
  - 3 easy/typical cases
  - 3 edge cases (unusual input, missing data, ambiguous intent)
  - 2 adversarial cases (user trying to break the output format)
  - 2 failure cases (inputs the feature should refuse or handle gracefully)

For each architecture candidate, score each test output:
  3 = Perfect output, exactly what was needed
  2 = Good output with minor issues
  1 = Output is wrong or incomplete in a meaningful way
  0 = Output is broken, harmful, or completely wrong

Total score / 30 = quality rate
Target: >80% for production (>24/30)
```

### Latency Benchmarking
```typescript
// Measure actual latency before committing to architecture
async function benchmarkArchitecture(inputs: string[], runs = 5) {
  const results = [];
  for (const input of inputs.slice(0, 3)) { // Test 3 representative inputs
    const times = [];
    for (let i = 0; i < runs; i++) {
      const start = Date.now();
      await runYourFeature(input);
      times.push(Date.now() - start);
    }
    results.push({
      input: input.slice(0, 50),
      p50: times.sort()[Math.floor(runs / 2)],
      p95: times.sort()[Math.floor(runs * 0.95)],
    });
  }
  return results;
}
```

### Architecture Decision Log
Document your decision before building. This is valuable for future AI agent sessions:

```json
{
  "feature": "[FEATURE NAME]",
  "decision_date": "[DATE]",
  "architecture_chosen": "single_agent | sequential_multi | parallel_multi",
  "rationale": "[WHY THIS PATTERN]",
  "alternatives_considered": ["[OTHER PATTERN]", "reason rejected"],
  "upgrade_trigger": "[WHAT WOULD CAUSE US TO UPGRADE TO MORE COMPLEX ARCHITECTURE]",
  "estimated_cost_per_call": "$[X]",
  "estimated_latency_p95": "[Xms]"
}
```

</testing_and_qa>

---

<common_patterns>

## Reusable Architecture Patterns

### Pattern 1: The Critic Pattern (Most Valuable Multi-Agent Addition)
```typescript
// When to use: when single-agent output quality is inconsistent
// How it works: Agent A generates → Agent B critiques → Agent A revises

async function generateWithCritic(input: string): Promise<string> {
  // Step 1: Generate
  const draft = await generateText(input, "You are an expert [ROLE]. Generate [OUTPUT].");

  // Step 2: Critique
  const critique = await generateText(
    `Here is a draft [OUTPUT]: ${draft}\nIdentify any errors, gaps, or improvements needed. Be specific.`,
    "You are a critical reviewer. Find flaws. Be direct."
  );

  // Step 3: Only revise if critique found real issues
  if (critique.toLowerCase().includes("no issues") || critique.length < 100) {
    return draft; // Draft was good — skip revision
  }

  // Step 4: Revise based on critique
  return await generateText(
    `Original [OUTPUT]: ${draft}\nCritique: ${critique}\nRevise the [OUTPUT] addressing the critique.`,
    "You are an expert [ROLE]. Revise based on the critique provided."
  );
}
```

### Pattern 2: The Router Pattern (Dynamic Agent Selection)
```typescript
// When to use: different inputs need fundamentally different handling
// How it works: first call classifies input → routes to specialist agent

type RouteDecision = "generate_reminder" | "list_reminders" | "delete_reminder" | "general_chat";

async function routedAgent(userMessage: string): Promise<string> {
  // Step 1: Classify (fast, cheap model)
  const route = await classifyIntent(userMessage); // Returns RouteDecision

  // Step 2: Route to specialist
  switch (route) {
    case "generate_reminder":
      return await reminderCreationAgent(userMessage);
    case "list_reminders":
      return await reminderListAgent(userMessage);
    case "delete_reminder":
      return await reminderDeleteAgent(userMessage);
    default:
      return await generalChatAgent(userMessage);
  }
}
```

### Pattern 3: The Map-Reduce Pattern (Parallel Agents)
```typescript
// When to use: same operation needs to run on N independent items
// How it works: spawn N agents in parallel → aggregate results

async function mapReduceAgents<T, R>(
  items: T[],
  agentFn: (item: T) => Promise<R>,
  reduceFn: (results: R[]) => Promise<string>
): Promise<string> {
  // MAP: run all agents in parallel
  const results = await Promise.all(items.map(agentFn));

  // REDUCE: synthesise all results into final output
  return await reduceFn(results);
}

// Example usage: generate lesson summaries for 5 topics simultaneously
const summaries = await mapReduceAgents(
  topics,
  (topic) => generateTopicSummary(topic),
  (summaries) => synthesiseSummaries(summaries)
);
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Each Agent Must Validate Its Input
Do not trust that the previous agent produced valid output. Every agent in a chain must validate what it receives before processing it.
```typescript
// ❌ Trusts previous agent blindly
async function agentB(inputFromAgentA: unknown) {
  const result = await callAI(`Process: ${inputFromAgentA}`);
}

// ✅ Validates before using
async function agentB(inputFromAgentA: unknown) {
  const validated = AgentAOutputSchema.safeParse(inputFromAgentA);
  if (!validated.success) {
    return { status: "failed", error: "Agent A produced invalid output" };
  }
  const result = await callAI(`Process: ${validated.data}`);
}
```

### Rule 2: Set Maximum Agent Iteration Limits
Agentic loops can run indefinitely if not bounded. Always set a hard stop.
```typescript
const MAX_ITERATIONS = 5;
let iterations = 0;

while (!taskComplete && iterations < MAX_ITERATIONS) {
  await runAgentStep();
  iterations++;
}

if (iterations >= MAX_ITERATIONS) {
  return { status: "failed", error: "Max iterations reached without completing task" };
}
```

### Rule 3: Never Give Agents Write Access Beyond Their Scope
An agent responsible for fetching data should not have permission to write data. Scope each agent's tool access to the minimum required for its responsibility.

### Rule 4: Log Every Agent-to-Agent Communication
When debugging a multi-agent failure, you need to know exactly what each agent received and produced. Log the full input and output of each agent step.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Starting with Multi-Agent Architecture
You decide a feature needs 5 agents before building any of them. You spend 3 days building an orchestrator system. The quality is not better than a single well-prompted agent would have been.  
**Fix:** Single agent first, always. Upgrade only when you hit a specific wall.

### ❌ Giving Agents Multiple Responsibilities
```
# ❌ Agent with split personality
System prompt: "You are an expert researcher AND a clear writer AND a critical reviewer.
Research the topic, then write a lesson, then review it for accuracy."

# ✅ Three focused agents
Agent 1: "You are a researcher. Find key facts about [topic]. Return JSON."
Agent 2: "You are a lesson writer. Write a lesson using these facts: [facts]."
Agent 3: "You are a reviewer. Check this lesson for accuracy: [lesson]."
```

### ❌ Using the Same Powerful (Expensive) Model for Every Agent
Classification and routing tasks do not need GPT-4o. Use GPT-4o-mini or Claude Haiku for simple steps. Reserve the powerful model for the generation step.  
**Fix:** Match model capability to task complexity. See `skill-model-selection-evaluation.md`.

### ❌ Not Handling Partial Pipeline Failures
Agent 1 succeeds, Agent 2 fails. The orchestrator does not handle this and either crashes or silently returns an empty result.  
**Fix:** Every multi-agent orchestrator must handle mid-pipeline failures: log which step failed, return a meaningful error state, and allow the pipeline to be resumed or retried from the failing step.

### ❌ Building Multi-Agent Before Validating the Single-Agent Quality
You assume a single agent cannot do the job without testing it. You over-engineer before you know what you need.  
**Fix:** Spend 20 minutes building and testing a single-agent prototype. Let the evidence guide the architecture decision.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Multi-Agent Systems

### Add Agent Memory Across Sessions
When agents need to remember context across multiple user sessions (not just within one pipeline run):
```
Short-term memory: pass conversation history in the messages array
Long-term memory: store summaries in the database, retrieve with semantic search
User-specific memory: store preferences and patterns per user in DB
```
See `skill-rag.md` for retrieval patterns that power long-term agent memory.

### Add Human-in-the-Loop Steps
For high-stakes agent actions (sending an email, making a purchase, deleting data), pause the pipeline and require human confirmation before the irreversible step executes. This is the single most important safety mechanism for agentic systems operating in the real world.

### Monitor Per-Agent Quality Over Time
```sql
-- Track which agent in your pipeline fails most often
SELECT
  feature,
  COUNT(*) FILTER (WHERE status = 'failed') as failures,
  COUNT(*) as total,
  ROUND(COUNT(*) FILTER (WHERE status = 'failed') * 100.0 / COUNT(*), 1) as failure_rate
FROM ai_generations
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY feature
ORDER BY failure_rate DESC;
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Single Agent Was Enough

**Feature:** Parse "remind me to call mum every Sunday at 3pm" into structured `{ task, recurrence, time }`.

**Single agent prototype result (5 test inputs):**
```
Input: "remind me to call mum every Sunday at 3pm"
Output: { task: "call mum", recurrence: "weekly", dayOfWeek: "sunday", time: "15:00" } ✅

Input: "every morning brush teeth"
Output: { task: "brush teeth", recurrence: "daily", time: "08:00" } ✅ (inferred reasonable time)

Input: "meeting with Sarah next Tuesday"
Output: { task: "meeting with Sarah", scheduledAt: "2025-01-21", recurrence: null } ✅

Input: "banana"
Output: { error: "unable to parse as reminder" } ✅

Input: "call dad on his birthday march 15"
Output: { task: "call dad", scheduledAt: "2025-03-15", recurrence: "yearly" } ✅
```

**Decision:** Single agent, 5/5 acceptable outputs. Multi-agent would have added latency and cost for zero quality benefit.

---

### Case Study 2: EdTech App — Sequential Multi-Agent Was Needed

**Feature:** Generate a personalised lesson on any topic, adapted to the student's level.

**Single agent prototype result:**
- Good at generating lesson content
- Consistently misjudged "intermediate" level — either too hard or too easy
- Could not simultaneously research the topic AND calibrate to the student's specific gaps

**Upgrade trigger hit:** Conflicting instructions ("be thorough" vs "be appropriate for this specific student").

**Sequential multi-agent design chosen:**
```
Agent 1: PROFILER
  Input: student's completed lessons, quiz scores, stated level
  Output: { actualLevel: 6/10, gapAreas: ["fractions", "decimals"], preferredStyle: "visual" }
  Model: gpt-4o-mini (classification task — cheap and fast)

Agent 2: CURRICULUM PLANNER
  Input: topic requested + student profile from Agent 1
  Output: { learningObjectives: [...], prerequisitesNeeded: [...], approachStrategy: "..." }
  Model: gpt-4o (reasoning task — needs quality)

Agent 3: LESSON WRITER
  Input: curriculum plan from Agent 2 + student profile from Agent 1
  Output: full lesson content (markdown)
  Model: gpt-4o (generation task — needs quality)

Total added latency: ~4 seconds
Quality improvement: student-appropriate calibration jumped from 60% to 91% on test set
Decision: sequential multi-agent justified by measurable quality improvement
```

</real_world_examples>

</skill_document>
