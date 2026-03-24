---
name: model-selection-evaluation
description: Chooses the right AI model for each task and evaluates quality with a structured test protocol. Use when AI costs are too high, when quality is inconsistent, when comparing OpenAI vs Anthropic, or before upgrading to a new model version.
---
# Skill: Model Selection & Evaluation

```json
{
  "skill_id": "model-selection-evaluation",
  "category": "AI-Specific",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Model Selection & Evaluation — Choosing the Right AI Model for Every Task</title>

<overview>

## What This Skill Enables
- Choose the right AI model for each feature in your app — balancing quality, speed, and cost
- Evaluate model outputs systematically rather than guessing which is better
- Understand when to upgrade a model (user-facing quality) vs when to downgrade (background tasks)
- Make model migration decisions safely without breaking existing features

## Why It Matters for Vibe Coders
The default choice of "use GPT-4o for everything" is expensive and sometimes slower than necessary. For a product with 5 AI features, 3 of them probably work just as well on a model that costs 10–20x less. This skill gives you a decision framework for every feature and a testing protocol that lets you verify quality before committing to a model.

## When to Use This Skill
- When starting a new AI feature — choose the model before writing the prompt
- When your AI API costs are higher than your budget allows
- When a model is performing poorly on a specific feature and you want to compare alternatives
- When a new model is released and you want to evaluate whether it improves your product

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "ai_provider": "[REPLACE: e.g. OpenAI / Anthropic / Google]",
    "current_models_in_use": {
      "feature_1": "[REPLACE: e.g. lesson generation: gpt-4o]",
      "feature_2": "[REPLACE: e.g. intent parsing: gpt-4o-mini]"
    },
    "monthly_ai_spend_usd": "[REPLACE: e.g. $45 / unknown]",
    "evaluation_task": "[REPLACE: e.g. 'find a cheaper model for intent classification' / 'compare claude vs gpt for lesson generation']",
    "quality_bar": "[REPLACE: e.g. must pass 90% of test cases / good enough for MVP / production-critical]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Model Selection

### Mental Model 1: The Right Tool for the Job
Using GPT-4o for intent classification is like using a hammer to turn a screw. It works, but it wastes the tool's capability — and costs money for every use.

Three task categories with different model requirements:

```
CATEGORY 1: CLASSIFICATION / EXTRACTION
  What it is: classify intent, extract fields, route requests, summarise briefly
  Quality needed: medium — mostly gets it right matters more than being perfect
  Speed needed: fast
  Right model: fast/cheap model (GPT-4o-mini, Claude Haiku, Gemini Flash)
  Cost: low

CATEGORY 2: GENERATION / REASONING
  What it is: write content, generate lessons, explain concepts, complex reasoning
  Quality needed: high — output quality directly affects user experience
  Speed needed: medium (can stream to manage perception)
  Right model: powerful model (GPT-4o, Claude Sonnet, Gemini Pro)
  Cost: medium

CATEGORY 3: COMPLEX ANALYSIS
  What it is: evaluate code, multi-document reasoning, research synthesis
  Quality needed: very high — errors are costly
  Speed needed: low priority (can be background task)
  Right model: most capable model (GPT-4o, Claude Opus, Gemini Ultra)
  Cost: high
```

### Mental Model 2: The Cost-Quality Frontier
Every model occupies a position on the cost-quality frontier. The goal is not to always be at the top-right (best quality, highest cost) — it is to be at the *right* point for each feature.

```
Quality
  ▲
  │                              ● GPT-4o / Claude Sonnet
  │                         ●
  │                    ●
  │               ●  Claude Haiku / GPT-4o-mini
  │          ●
  └──────────────────────────────────────→ Cost per 1M tokens
```

### Mental Model 3: Evaluation Is a Test, Not an Opinion
"I think Model A is better than Model B" is not useful. "Model A scored 9.2/10 on my 20-question test set vs Model B's 7.8/10" is useful. This skill gives you the evaluation protocol to turn opinions into numbers.

</mental_models>

---

<system_design_breakdown>

## Model Reference (as of early 2025)

> **Note:** Model pricing and capabilities change frequently. Always verify at the provider's pricing page before making budget decisions. These figures are approximate and for decision-making guidance only.

### OpenAI Models
| Model | Relative Speed | Relative Cost | Best For |
|---|---|---|---|
| gpt-4o | Fast | $$ | Most generation and reasoning tasks |
| gpt-4o-mini | Fastest | $ | Classification, routing, extraction |
| o1 / o3 | Slow | $$$$ | Complex multi-step reasoning, math |
| text-embedding-3-small | Fast | Very low | Embeddings (1536 dims) |

### Anthropic Models
| Model | Relative Speed | Relative Cost | Best For |
|---|---|---|---|
| claude-3-5-sonnet | Fast | $$ | Complex generation, coding, reasoning |
| claude-3-haiku | Fastest | $ | Classification, simple tasks |
| claude-3-opus | Medium | $$$ | Most complex reasoning tasks |

### Google Models
| Model | Relative Speed | Relative Cost | Best For |
|---|---|---|---|
| gemini-1.5-pro | Medium | $$ | Long context (1M tokens), multimodal |
| gemini-1.5-flash | Fast | $ | Fast tasks, high volume |

### Decision Guide
```
For each AI feature, ask:
1. Does this feature directly affect what users see/read?
   YES → Use a quality model (GPT-4o / Claude Sonnet)
   NO  → Consider a fast/cheap model (GPT-4o-mini / Claude Haiku)

2. Is speed critical (user is waiting, streaming not possible)?
   YES → Use the fastest model that meets quality bar
   NO  → Quality matters more than speed

3. Is this called >100 times/day?
   YES → Cost optimisation matters — evaluate if a cheaper model qualifies
   NO  → Cost is not a significant factor yet
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Define test set → run model A → run model B → compare scores → decide. -->

## Step 1 — Define Your Evaluation Test Set

Before comparing any models, create your test set. This is the foundation of every model decision.

**Test set requirements:**
```
Minimum: 20 questions/prompts
Composition:
  - 50% typical cases (what most users will send)
  - 30% edge cases (unusual inputs, boundary conditions)
  - 20% adversarial cases (inputs designed to confuse the model)

For each test case, define:
  - Input: what you will send to the model
  - Expected output: what a correct response looks like
  - Evaluation criterion: how you will score the output (see scoring below)
```

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

I need to evaluate models for this feature: [FEATURE NAME]
Feature description: [DESCRIBE]
Current prompt: [PASTE YOUR PROMPT]

Generate a 20-question evaluation test set.
Include: 10 typical, 6 edge cases, 4 adversarial
For each: input + expected output + scoring criterion (pass/fail or 1-5 scale)
```

---

## Step 2 — Run the Evaluation

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Test set: [PASTE YOUR 20 TEST CASES]
Current prompt: [PASTE YOUR SYSTEM PROMPT]

Run this test set against these two models:
- Model A: [CURRENT MODEL]
- Model B: [CANDIDATE MODEL]

For each test case:
1. Send the same system prompt + test input to both models
2. Score each response against the expected output
3. Record: response text, score (pass/fail or 1-5), latency in ms

Return a comparison table:
Test # | Input | Model A Score | Model B Score | Winner
```

---

## Step 3 — Analyse the Results

Look for patterns, not just totals:

```
Total score comparison → is one model clearly better overall?
Failure pattern analysis → where does each model fail? Is there a category?
Latency comparison → how much faster is the faster model?
Cost calculation → at X calls/day, what is the monthly cost difference?
```

**Prompt your AI agent:**
```
Model evaluation results: [PASTE YOUR COMPARISON TABLE]

Analyse:
1. Overall winner by score
2. Categories where each model fails (pattern in the failures)
3. At [X calls/day], monthly cost for each model at current pricing
4. Recommendation: which model to use and why
5. If the cheaper model scores within 10% of the better model: recommend the cheaper one

Be direct. Give me a clear recommendation, not a "it depends" answer.
```

---

## Step 4 — Implement and Verify the Model Change

After choosing a model:

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Decision: switching [FEATURE] from [MODEL A] to [MODEL B]

Update /lib/prompts/[feature].ts or /lib/ai.ts to:
1. Change the model constant for this feature
2. Keep the model as a named constant (not hardcoded string) so it is easy to change again
3. Add a comment: "Changed from [MODEL A] on [DATE]. Reason: [REASON]. Test score: [X/20]"

Show me a quick smoke test to verify the new model works correctly in production.
```

**Verify:** Run 5 of your test cases against the production endpoint with the new model. All 5 should pass.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am evaluating or selecting AI models for my app.
Project context: [PASTE context_anchor JSON]

Task today: [DESCRIBE — e.g. "compare gpt-4o vs gpt-4o-mini for intent classification"]
Feature affected: [FEATURE NAME]
Current quality score (if known): [X/20 or "not yet measured"]

Before running any evaluation:
1. Confirm the test set has at least 20 diverse cases
2. Confirm both models will use the same system prompt
3. Confirm how we will score: pass/fail or 1-5 scale
```

### Cost Optimisation Audit Prompt
```
Audit my AI feature costs and identify optimisation opportunities.
Project context: [PASTE context_anchor JSON]
Current models in use: [LIST FEATURES AND THEIR MODELS]

For each feature:
1. Classify as: classification/extraction / generation / complex analysis
2. Current model appropriate for task type? (yes/no)
3. Estimated monthly calls at [X] daily active users
4. Current estimated monthly cost
5. Alternative cheaper model to evaluate (only if task type allows)

Return a prioritised list of cost optimisation opportunities,
ordered by potential monthly savings.
```

### Model Migration Safety Prompt
```
I want to migrate [FEATURE] from [MODEL A] to [MODEL B].
Current prompt: [PASTE]
Test results: Model B scored [X/20] vs Model A's [Y/20]

Before migrating, identify:
1. Any test cases where Model B failed that Model A passed (regressions)
2. Are these regressions in critical paths (user-facing) or edge cases?
3. Can prompt adjustments fix the regressions before migrating?
4. Recommended migration strategy: full switch / A/B test / gradual rollout

Only recommend the migration if it is clearly safe. Do not recommend migrating
if there are regressions in critical-path features without a fix.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "GPT-4o-mini vs GPT-4o — when does it actually matter?"

For most simple tasks, you will not notice the difference in quality. The situations where it matters:

**GPT-4o-mini is fine for:**
- Classifying user intent ("is this a reminder or a question?")
- Extracting structured fields from a message
- Simple summarisation (100 words or less)
- Routing decisions (which agent should handle this?)
- Translating between simple formats

**GPT-4o is worth the cost for:**
- Generating educational content (quality and nuance matter)
- Complex reasoning about user problems
- Evaluating or critiquing content
- Tasks requiring deep understanding of subtle context
- When your product's core value proposition is AI quality

**How to decide empirically:** Run your 20-question test on both. If GPT-4o-mini scores within 10% of GPT-4o, use GPT-4o-mini. If the gap is larger, the quality difference is real and the cost is justified.

---

### "When should I switch from OpenAI to Anthropic (or vice versa)?"

The models are close in capability for most tasks. Switch when you have a specific reason:

**Consider Anthropic Claude if:**
- Your feature involves following complex multi-step instructions precisely
- You are building coding or technical writing features (Claude tends to excel here)
- You need very long context windows (Claude handles 200k tokens)
- You are concerned about safety/alignment characteristics

**Stick with OpenAI if:**
- Your prompts are already tuned for GPT-4o and working well
- Your team is more familiar with the OpenAI API
- You rely heavily on function/tool calling (both are excellent, but OpenAI has more ecosystem support)

**Never switch models without running your evaluation test set on both.** Anecdotes and benchmarks are not your app's performance — your test set is.

---

### 🗂️ Update Your AGENT_CONTEXT.md

After completing model selection, capture these decisions:

```md
## Model Configuration
- Primary model: [pinned model ID e.g. gpt-4o-2024-08-06]
- Fast/cheap model: [pinned model ID e.g. gpt-4o-mini-2024-07-18]
- Fallback provider: [anthropic | openai]
- Model config location: `lib/ai/models.ts`
- Eval protocol: [link to your eval spreadsheet or test file]
- Latency SLO: p95 < [X]ms for [feature name]
- Next deprecation review: [date — set 90 days out]
```

</vibe_coder_bridge>

---

<testing_and_qa>

## Model Evaluation Protocol

### Scoring Rubric for Generation Tasks (1–5 Scale)
```
5 = Perfect: matches expected output in content, format, and tone
4 = Good: minor issues (slight wording difference, one missing detail) — acceptable for production
3 = Acceptable: noticeable issues but core content is correct — acceptable for MVP
2 = Poor: significant issues — wrong format, missing key content, misleading
1 = Fail: wrong answer, hallucination, format broken, or refused appropriate request
```

### Pass/Fail for Classification/Extraction Tasks
```
Pass: correct label or correctly extracted field
Fail: wrong label, missing required field, hallucinated field, wrong type
```

### Decision Thresholds
```
Score ≥ 90% → Production ready for this feature
Score 75-89% → Acceptable for MVP, revisit before scaling
Score < 75%  → Not production ready — iterate on prompt or model before shipping
```

### Model Comparison Template
```
Feature: [FEATURE NAME]
Date evaluated: [DATE]
Test set size: 20

| # | Input (preview) | Expected | Model A (score) | Model B (score) | Winner |
|---|---|---|---|---|---|
| 1 | "..." | JSON | 5 | 4 | A |
...

Model A: [MODEL NAME]
  Total: X/100 (X%)
  Avg latency: Xms
  Failed cases: #[N], #[N], #[N]

Model B: [MODEL NAME]
  Total: X/100 (X%)
  Avg latency: Xms
  Failed cases: #[N], #[N], #[N]

Decision: [MODEL CHOSEN AND BRIEF JUSTIFICATION]
Monthly cost estimate at [X] daily calls: $[X] vs $[X]
```

</testing_and_qa>

---

<common_patterns>

## Reusable Model Configuration Patterns

### Pattern 1: Latency SLOs — Defining Acceptable Performance

Before selecting a model, define your latency target. Different features have different tolerances:

| Feature | p50 target | p95 target | Model recommendation |
|---|---|---|---|
| Real-time chat | < 300ms TTFT | < 800ms TTFT | gpt-4o-mini, Claude Haiku |
| Document analysis | < 2s | < 5s | gpt-4o, Claude Sonnet |
| Background job | < 30s | < 60s | Any model |
| Code generation | < 1s TTFT | < 3s TTFT | gpt-4o, Claude Sonnet |

TTFT = Time to First Token (what users perceive as "response speed")

```typescript
// lib/ai/latency-tracker.ts
export async function measureLatency(
  modelCall: () => Promise<unknown>
): Promise<{ result: unknown; ttftMs: number; totalMs: number }> {
  const start = Date.now()
  let ttftMs = 0

  // For streaming, capture time to first chunk
  const result = await modelCall()
  const totalMs = Date.now() - start

  if (totalMs > 5000) {
    console.warn(`[latency:slow] ${totalMs}ms — consider switching to a faster model`)
  }

  return { result, ttftMs, totalMs }
}
```

Log p50/p95 latency weekly using your `ai_generations` table: `SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY latency_ms) FROM ai_generations WHERE feature = 'chat'`

### Pattern 2: Centralised Model Configuration
```typescript
// /lib/ai-config.ts
// Single place to manage all model choices — easy to update and review

export const AI_MODELS = {
  // Intent classification — fast, cheap (no quality difference from gpt-4o)
  intentClassification: "gpt-4o-mini",

  // Content generation — quality matters for user experience
  lessonGeneration: "gpt-4o",
  explanationGeneration: "gpt-4o",

  // Background tasks — cheap model fine, users don't wait
  reminderSummarisation: "gpt-4o-mini",
  progressAnalysis: "gpt-4o-mini",

  // Embeddings
  embedding: "text-embedding-3-small",
} as const;

// Model metadata for cost tracking
export const MODEL_PRICING = {
  "gpt-4o": { inputPer1M: 2.50, outputPer1M: 10.00 },
  "gpt-4o-mini": { inputPer1M: 0.15, outputPer1M: 0.60 },
  "text-embedding-3-small": { inputPer1M: 0.02, outputPer1M: 0 },
} as const;
```

### Pattern 2: A/B Testing Models in Production
```typescript
// /lib/ai/model-ab-test.ts
// Gradually roll out a new model to a percentage of users

export function selectModel(
  featureName: keyof typeof AI_MODELS,
  userId: string,
  rolloutPercent = 0 // 0 = all users get current model, 50 = 50/50 split
): string {
  if (rolloutPercent === 0) return AI_MODELS[featureName];

  // Deterministic split based on userId (same user always gets same model)
  const userBucket = parseInt(userId.slice(-2), 16) % 100;
  const useNewModel = userBucket < rolloutPercent;

  const model = useNewModel ? NEW_MODELS[featureName] : AI_MODELS[featureName];

  // Track which model served this user (for quality comparison later)
  trackAIGeneration({ feature: `${featureName}_ab_test`, model, /* ... */ });

  return model;
}
```

### Pattern 3: Fallback Routing — Graceful Degradation During Outages

AI API providers have outages. Without fallback routing, your product goes down with them.

```typescript
// lib/ai/router.ts
import OpenAI from "openai"
import Anthropic from "@anthropic-ai/sdk"

const openai = new OpenAI()
const anthropic = new Anthropic()

type RouteConfig = {
  primary: "openai" | "anthropic"
  fallback: "openai" | "anthropic"
  primaryModel: string
  fallbackModel: string
}

export async function routedCompletion(
  prompt: string,
  config: RouteConfig
): Promise<string> {
  try {
    if (config.primary === "openai") {
      const res = await openai.chat.completions.create({
        model: config.primaryModel,
        messages: [{ role: "user", content: prompt }],
        max_tokens: 1000,
      })
      return res.choices[0].message.content ?? ""
    }
    // anthropic primary path...
  } catch (primaryErr) {
    console.error(`[model:fallback] Primary ${config.primaryModel} failed, trying ${config.fallbackModel}`)

    try {
      if (config.fallback === "anthropic") {
        const res = await anthropic.messages.create({
          model: config.fallbackModel,
          max_tokens: 1000,
          messages: [{ role: "user", content: prompt }],
        })
        return res.content[0].type === "text" ? res.content[0].text : ""
      }
      // openai fallback path...
    } catch (fallbackErr) {
      throw new Error("All AI providers unavailable. Please try again shortly.")
    }
  }
  return ""
}
```

Monitor fallback trigger rate in your observability dashboard — a spike means your primary provider is having issues.

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Hardcode Model Names in Application Logic
```typescript
// ❌ Model hardcoded — changing it requires finding all occurrences
const response = await openai.chat.completions.create({ model: "gpt-4o", ... });

// ✅ Model from centralised config — one place to update
import { AI_MODELS } from "@/lib/ai-config";
const response = await openai.chat.completions.create({ model: AI_MODELS.lessonGeneration, ... });
```

### Rule 2: Pin Model Versions for Production
```typescript
// ❌ Model alias — behaviour can change without notice when OpenAI updates it
model: "gpt-4o"

// ✅ Pinned version — predictable behaviour, intentional upgrades only
model: "gpt-4o-2024-11-20"
// Update the version intentionally after running your evaluation test set
```

### Rule 3: Test Before Migrating
Never switch a production model without running your evaluation test set first. A model that was better in a general benchmark may perform worse on your specific use case.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Using unpinned model identifiers in production

`gpt-4o` silently changed behaviour multiple times across 2024–2025. Using a floating identifier means your evals pass today but the model behaves differently next month.

✅ Always pin to dated snapshots in production:

```typescript
// lib/ai/models.ts — single source of truth for model config
export const MODELS = {
  // ✅ Pinned — predictable behaviour, survives model updates
  primary: "gpt-4o-2024-08-06",
  fast: "gpt-4o-mini-2024-07-18",
  reasoning: "claude-opus-4-6",  // Anthropic models are versioned by release

  // ❌ Never use these in production
  // primary: "gpt-4o",   // silently updated
  // fast: "gpt-4o-mini", // silently updated
} as const

export type ModelKey = keyof typeof MODELS
```

**Upgrade process:**
1. A new model version releases (monitor OpenAI/Anthropic changelog)
2. Update `MODELS.primary` in a branch
3. Re-run your full 20-question eval suite against the new model
4. Compare scores — only merge if scores are equal or better
5. Deploy; monitor for 48 hours before removing old version

Set a calendar reminder every 90 days to review model deprecation notices. OpenAI provides minimum 30-day notice but typically 90+ days.

### ❌ Using the Same Model for Every Feature
If every feature uses GPT-4o, you are spending 10–20x more than necessary on features that would work equally well on GPT-4o-mini.  
**Fix:** Apply the task classification framework to every feature. Default to fast/cheap for classification; use quality model for generation.

### ❌ Making Model Decisions Based on Benchmarks Alone
"Model X scores highest on MMLU" does not mean it will perform best on your specific prompt and use case. Benchmarks measure general capability, not your feature's requirements.  
**Fix:** Always evaluate on your own test set.

### ❌ Not Re-Testing After Model Updates
OpenAI and Anthropic update their models regularly. A model alias like "gpt-4o" can behave differently after an update. Features that passed evaluation before the update may fail afterward.  
**Fix:** Pin specific model versions. Re-test before intentionally upgrading to a new version.

### ❌ Switching Models Without Checking for Regressions
You evaluate Model B vs Model A for your primary test cases. Model B scores better overall. You switch. You discover later that Model B fails on 3 edge cases that Model A handled well — and those edge cases cause user complaints.  
**Fix:** Look at failure patterns, not just totals. Identify every case where the new model is worse and decide consciously whether those regressions are acceptable.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Model Evaluation

### Automate Your Evaluation with PromptFoo
```bash
# Install PromptFoo for automated model comparison
npm install -g promptfoo

# Create promptfooconfig.yaml with your test cases
# Run: promptfoo eval
# View results: promptfoo view
```

### Add Continuous Model Monitoring
Track model quality over time in production using your `ai_generations` table:
```sql
-- Track quality proxy: user actions after AI responses
-- If user immediately regenerates after an AI response: implicit quality signal
SELECT
  model,
  DATE_TRUNC('week', created_at) as week,
  COUNT(*) as total_calls,
  AVG(latency_ms) as avg_latency,
  -- Add your own quality signal column if you track user feedback
FROM ai_generations
WHERE feature = 'lesson_generation'
GROUP BY model, week
ORDER BY week DESC;
```

### Implement LLM-as-Judge for Scale
When evaluating 100+ test cases, use a separate LLM call to score outputs automatically:
```typescript
async function evaluateOutput(expected: string, actual: string): Promise<number> {
  const score = await generateText(
    `Expected output: ${expected}\nActual output: ${actual}\n
     Score the actual output 1-5 based on how well it matches the expected.
     Return ONLY a number.`,
    "You are an objective evaluator. Return only a number 1-5."
  );
  return parseInt(score.data?.content ?? "1");
}
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Cost Reduction Without Quality Loss
```
Original: all 4 AI features using gpt-4o
Monthly cost at 500 DAU: $142

Evaluation run on 20 test cases per feature:
  Intent classification: gpt-4o 19/20 vs gpt-4o-mini 18/20 → within 5% → switch
  Reminder confirmation: gpt-4o 18/20 vs gpt-4o-mini 14/20 → 20% gap → keep gpt-4o
  Recurrence detection: gpt-4o 20/20 vs gpt-4o-mini 19/20 → within 5% → switch
  Response generation: gpt-4o 19/20 vs gpt-4o-mini 15/20 → 20% gap → keep gpt-4o

After optimisation:
  Intent classification: gpt-4o-mini ($)
  Reminder confirmation: gpt-4o ($$)
  Recurrence detection: gpt-4o-mini ($)
  Response generation: gpt-4o ($$)

New monthly cost: $71 (50% reduction)
Quality impact: intent classification accuracy 95% → 90% (acceptable tradeoff)
```

### Case Study 2: EdTech App — Anthropic vs OpenAI for Lesson Generation
```
Evaluation: gpt-4o vs claude-3-5-sonnet for lesson generation
Test set: 20 lesson generation tasks across 4 grade levels

gpt-4o: 16.8/20 average
  Strengths: consistent formatting, follows structure well
  Weaknesses: slightly generic examples, occasional repetition

claude-3-5-sonnet: 17.9/20 average
  Strengths: richer analogies, better calibration to student level
  Weaknesses: slightly longer than requested on 3 tests

Decision: switched to claude-3-5-sonnet for lesson generation
Cost impact: similar pricing → no change in monthly cost
Quality impact: student satisfaction (5-star ratings) increased from 3.8 to 4.1 over 4 weeks
```

</real_world_examples>

</skill_document>
