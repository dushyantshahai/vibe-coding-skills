# Skill: User Experience Design

```json
{
  "skill_id": "user-experience-design",
  "category": "Product & Non-Technical",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>User Experience Design — Flows, Friction Points, and Accessibility for AI Products</title>

<overview>

## What This Skill Enables
- Map user flows before writing any UI code, so you build the right screens in the right order
- Identify and eliminate friction points that cause users to abandon your core feature
- Apply accessibility fundamentals that make your app usable by everyone — without a UX background
- Brief your AI coding agent with enough UX context that it builds screens that feel intentional, not accidental

## Why It Matters for Vibe Coders
Vibe-coded UIs often work technically but feel awkward to use. Buttons in unexpected places, confusing navigation, multi-step flows that could be one step, text that does not tell users what to do next. These are not design failures — they are missing user thinking. This skill gives you a lightweight process for thinking like a user before building, so the first version feels right rather than requiring expensive UX rewrites.

## When to Use This Skill
- Before building any new screen or user-facing flow
- When users are dropping off or abandoning a feature mid-way
- When your app's navigation feels confusing to someone seeing it for the first time
- Before a user testing session — to define what you are testing and why

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "user_type": "[REPLACE: e.g. adult professionals / students aged 10-16 / busy parents]",
    "primary_device": "[REPLACE: e.g. mobile-first / desktop-first / both equally]",
    "core_user_journey": "[REPLACE: describe in plain English what a new user does in their first 5 minutes]",
    "flow_to_design": "[REPLACE: which specific flow or screen are you designing?]",
    "known_friction_points": "[REPLACE: any known places where users struggle or abandon]",
    "accessibility_requirements": "[REPLACE: e.g. standard WCAG AA / educational platform for all ages / none specified]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About UX for AI Products

### Mental Model 1: The Three-Question Test
Before building any screen, answer three questions from the user's perspective:

```
1. WHERE AM I?
   Does the screen clearly tell me what part of the app I am in?

2. WHAT CAN I DO HERE?
   Is the primary action obvious? Is it the first thing I see?

3. WHAT HAPPENS NEXT?
   After I take the action, do I know what the result will be?
```

If any answer is "unclear," the screen has a UX problem. Fix the answer before writing code.

### Mental Model 2: AI Product UX Specifics
AI features have unique UX challenges that standard web UX does not prepare you for:

| Challenge | Why It Happens | UX Solution |
|---|---|---|
| Users do not know what to type | Open input fields are intimidating | Add 3 example prompts as chips below the input |
| AI takes too long | Generation takes 3–15 seconds | Show a progress indicator with words appearing as they stream |
| AI output is wrong | Model errors or misunderstanding | Always show an easy "regenerate" or "edit" option |
| Users do not trust AI output | First-time users are skeptical | Show sources, confidence levels, or cite where information comes from |
| Conversation context is lost | Multi-turn chats feel disconnected | Show conversation history, make it easy to start fresh |

### Mental Model 3: The Friction Audit
Every step a user must take to complete a task is potential friction. Count the steps. Then ask: can any step be eliminated, automated, or defaulted?

```
BEFORE: User creates a reminder
Step 1: Open app
Step 2: Tap "New Reminder"
Step 3: Type reminder text
Step 4: Select date (calendar picker)
Step 5: Select time (time picker)
Step 6: Choose recurrence
Step 7: Tap "Save"
Total: 7 steps

AFTER: User creates a reminder with NLP
Step 1: Open app
Step 2: Type "call mum Sunday 3pm"
Step 3: Confirm (tap "Create")
Total: 3 steps

Friction reduced: 57%. This is the UX value of AI input.
```

</mental_models>

---

<system_design_breakdown>

## User Flow Mapping

### Core Flow Types for AI Products

**1. Onboarding Flow** — first 5 minutes of a new user's experience
```
Land on homepage
    │
    ▼
Sign up / Sign in
    │
    ▼
Setup / preference collection (optional but valuable)
    │
    ▼
First core action (create first reminder / view first lesson)
    │
    ▼
Success confirmation + next step suggestion
```

**2. Core Feature Flow** — the primary repeated action
```
Entry point (dashboard / home screen)
    │
    ▼
Trigger the feature (button, input, voice)
    │
    ▼
AI processing (streaming indicator)
    │
    ▼
AI output displayed
    │
    ▼
User action on output (save / edit / share / dismiss)
    │
    ▼
Confirmation + return to entry point
```

**3. Error/Recovery Flow** — what happens when things go wrong
```
User attempts action
    │
    ▼
Error occurs (AI fails / invalid input / network issue)
    │
    ▼
Clear error message (what went wrong, not a code)
    │
    ▼
Recovery path (retry / alternative action / contact support)
```

## Empty State Design

Every list and content area needs three states designed before first use:

```
EMPTY STATE (no data yet)
  Message: "You have no reminders yet."
  Subtext: "Type a reminder in the box below to get started."
  CTA: [Create your first reminder]

LOADING STATE (data is being fetched)
  Show skeleton cards matching the shape of real content
  Never show a full-page spinner for lists

POPULATED STATE (has data)
  Show the data
  Include a way to add more
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Map the flow before building any screen. Design empty/loading/populated states for every data view. -->

## Step 1 — Map the User Flow

Before any UI code, map the flow in plain text.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

I am designing the user flow for: [FEATURE OR SCREEN]

Map this as a numbered flow:
1. Where does the user start? (which screen, what they were doing before)
2. What triggers this flow? (button tap, navigation, notification)
3. Each screen the user sees, in order
4. Decision points where the flow branches (success/error, logged in/not)
5. Where the flow ends (what the user has achieved)

For each step, note:
- What the user SEES
- What the user DOES
- What could go wrong and what happens then

Keep it as plain text — no wireframes yet.
```

**Verify:** Walk through the flow yourself pretending to be a first-time user. Count the steps. Are there any steps that feel unnecessary? Can any be automated or defaulted?

---

## Step 2 — Identify and Eliminate Friction

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
User flow: [PASTE YOUR FLOW MAP]

Perform a friction audit:
1. Count total user steps to complete the core action
2. Identify steps that could be eliminated (unnecessary confirmation dialogs, redundant inputs)
3. Identify steps that could be automated with a sensible default (e.g. "today at 9am" as default reminder time)
4. Identify steps that block users if they do not have information (e.g. "what is your timezone?" — can this be detected?)
5. Identify steps where the AI could do the work instead of the user

Recommend the top 3 friction reductions.
Only recommend changes achievable in the current tech stack.
```

---

## Step 3 — Write UI Copy

Screen copy (labels, buttons, error messages, empty states) is UX. Bad copy causes confusion even on a well-designed screen.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Write all the UI copy for: [SCREEN OR FEATURE]

Include:
1. Page/section title
2. Subheading or description (if needed)
3. Input placeholder text (describe what to type, not "Enter text here")
4. Button labels (action verbs, specific: "Create Reminder" not "Submit")
5. Empty state: title + subtext + CTA button label
6. Loading state: brief text shown while AI processes
7. Success message (after action completes)
8. Error messages for each error type (readable, not codes)
9. Confirmation dialogs (if any destructive actions)

Tone: [DESCRIBE YOUR APP'S TONE — e.g. friendly and conversational / professional and efficient]
User: [DESCRIBE WHO READS THIS]
```

**Verify:** Read every piece of copy aloud. Does it sound like something a helpful person would say? Replace anything that sounds robotic or vague.

---

## Step 4 — Design for Accessibility

Basic accessibility additions that take 30 minutes but dramatically expand who can use your app.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Add accessibility to this component/page: [PASTE CODE OR DESCRIBE THE SCREEN]

Ensure:
1. All interactive elements have aria-label when label is not visible text
2. Color is not the only indicator of state (add text or icon alongside color)
3. Focus order follows visual reading order (Tab key navigates logically)
4. All images have alt text (empty alt="" for decorative images)
5. Error messages are associated with their input fields (aria-describedby)
6. Loading states announce themselves to screen readers (aria-live="polite")
7. Minimum touch target size: 44x44px for all tappable elements

Do not change any visual design — only add accessibility attributes.
```

**Verify:** Tab through the page using only a keyboard. Can you reach and activate every interactive element? Does the tab order make sense?

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on UX design for my app.
Project context: [PASTE context_anchor JSON]

Flow or screen: [DESCRIBE]
User type: [DESCRIBE THE USER]
Primary device: [MOBILE / DESKTOP / BOTH]

Before building any UI:
1. Map the user flow in plain text (steps, decision points)
2. Count the total steps to complete the core action
3. Identify the top friction point
4. Suggest: what should the empty, loading, and populated state show?
```

### First-Time User Flow Prompt
```
Design the onboarding flow for a new user of my app.
Project context: [PASTE context_anchor JSON]

The user has just:
- Clicked a sign-up link from [WHERE — e.g. landing page, app store, referral]
- Completed sign-up with [AUTH METHOD]

Design:
1. First screen after sign-up (welcome + what to do first)
2. Whether to collect preferences immediately or later (and why)
3. First core action experience (make it feel good and fast)
4. What happens after first success (celebration + what's next)

Principle: reach first value (the "aha moment") in under 2 minutes.
```

### Empty State Design Prompt
```
Design empty, loading, and error states for: [COMPONENT/PAGE]
User context: [PASTE context_anchor JSON]

For each state:
1. Headline (what is the situation in plain words)
2. Subtext (what to do about it)
3. CTA (if applicable — label and what it does)
4. Illustration/icon suggestion (describe — do not generate an image)

Tone: [DESCRIBE — e.g. friendly / professional / playful]
Empty state should motivate action, not just inform.
Loading state should feel fast (show skeleton shapes, not spinners).
Error state should give a clear recovery path.
```

### UX Copy Audit Prompt
```
Audit the UX copy in this screen:
[PASTE JSX/HTML OR DESCRIBE THE SCREEN]

For each piece of text, evaluate:
1. Is it in active voice? (e.g. "Save reminder" not "Reminder will be saved")
2. Is it specific? (e.g. "Add your first reminder" not "Get started")
3. Is it free of jargon? (no technical terms a non-technical user would not know)
4. Does every button label describe what happens? (action + object: "Create Lesson" not "Submit")
5. Do error messages tell users what to do, not just what went wrong?

Return a table: current copy | issue | suggested replacement
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is WCAG and do I need to think about it?"

WCAG stands for Web Content Accessibility Guidelines. At a practical level, it defines what it means for a website to be usable by people with visual, motor, auditory, or cognitive disabilities.

**You do not need to be a WCAG expert.** You need to follow a few rules that cover the most impactful cases:

1. Text has enough contrast (light grey text on white is nearly invisible to many users)
2. Users can navigate with only a keyboard (no mouse required)
3. Images have text descriptions (`alt` attribute)
4. Forms tell users what each field is for (`label` elements or `aria-label`)

**Why it matters for AI products specifically:** Many AI products are used in accessibility-critical contexts — educational apps are used by students with learning disabilities, productivity apps are used by people with motor difficulties who rely on keyboard navigation. Building accessible means building for more users.

---

### "My app has too many screens and feels overwhelming. How do I simplify?"

Progressive disclosure is the answer. The principle: show users only what they need right now. Hide everything else until it is relevant.

**Applied to your app:**
- Dashboard: show only the most recent/urgent items, not everything
- Forms: show only required fields; put optional fields behind "Advanced options"
- Settings: show basic settings first; put rarely-changed settings in a secondary section
- Navigation: show only the top 3–4 sections; put less-used areas in a menu

**The question to ask about every UI element:** Does the user need this right now, or only sometimes? If "only sometimes" — hide it until needed.

---

### "Should I collect user preferences during onboarding?"

The rule of thumb: only collect what you will immediately use to personalise the experience. If you ask for it but do not use it for at least 5 minutes, do not ask.

**Ask immediately if:** You need it to give the first experience (e.g. student's grade level for lesson calibration).
**Ask later if:** You can infer it or it improves later sessions (e.g. preferred reminder time — learn from first few reminders).
**Never ask if:** You just want the data but do not use it to change the experience.

Fewer onboarding questions = more users completing onboarding = more users reaching first value.

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing Your UX

### The First-Time User Test
The most valuable UX test you can run:
```
Find someone who has never seen your app.
Give them one task: "[Your core action — e.g. create a reminder for tomorrow at 9am]"
Say nothing. Watch.
Note: where do they hesitate? What do they click that does not do what they expect?
Ask after: "Was anything confusing?"

You do not need 10 people for this. 3 people will surface 80% of your UX problems.
```

### Keyboard Navigation Test
```
Close your mouse/trackpad.
Open your app.
Use only Tab, Enter, Space, and arrow keys.
Try to: log in → navigate to core feature → create one item → log out.

Pass: you can complete all steps without a mouse.
Fail: any step requires a mouse — this is an accessibility failure.
```

### Mobile UX Test
```
Open your app on your phone (or Chrome DevTools → 390px).
Try to complete the core user journey.
Check:
□ All text is readable without zooming
□ All buttons can be tapped without accidentally hitting adjacent elements
□ The keyboard does not obscure the input when typing
□ Scroll works naturally without horizontal overflow
□ Back navigation works (browser back button or in-app back)
```

### Common UX Issues and Fixes

| Issue | Symptom | Fix |
|---|---|---|
| No empty state | Blank white screen when list is empty | Design and build empty state before building list items |
| Spinner instead of skeleton | Page looks broken while loading | Replace full-page spinners with content-shaped skeleton screens |
| Vague error messages | Users do not know what to do when something fails | Rewrite errors as: "[What happened]. [What to do]." |
| No loading feedback on AI calls | Users tap a button multiple times thinking nothing happened | Disable button + show loading spinner immediately on tap |
| Confirmation dialogs for everything | Users are interrupted constantly | Only use confirmation dialogs for irreversible, high-consequence actions |

</testing_and_qa>

---

<common_patterns>

## Reusable UX Patterns for AI Products

### Pattern 1: AI Input with Example Prompts
```typescript
// /components/features/chat/ai-input.tsx
const EXAMPLE_PROMPTS = [
  "Remind me to call mum Sunday at 3pm",
  "Gym every weekday at 7am",
  "Take medication at 8am and 8pm daily",
];

export function AIInput({ onSubmit }: { onSubmit: (text: string) => void }) {
  const [value, setValue] = useState("");

  return (
    <div className="space-y-3">
      <div className="flex gap-2">
        <textarea
          value={value}
          onChange={(e) => setValue(e.target.value)}
          placeholder="Type a reminder in plain English..."
          aria-label="Reminder text"
          className="flex-1 resize-none rounded-lg border p-3"
          rows={2}
        />
        <Button
          onClick={() => { onSubmit(value); setValue(""); }}
          disabled={!value.trim()}
          aria-label="Create reminder"
        >
          Create
        </Button>
      </div>

      {/* Example prompts — shown when input is empty */}
      {!value && (
        <div className="flex flex-wrap gap-2">
          <span className="text-xs text-gray-500">Try:</span>
          {EXAMPLE_PROMPTS.map((prompt) => (
            <button
              key={prompt}
              onClick={() => setValue(prompt)}
              className="rounded-full bg-gray-100 px-3 py-1 text-xs text-gray-600 hover:bg-gray-200"
            >
              {prompt}
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Pattern 2: AI Generation Status UI
```typescript
// /components/ui/ai-generation-status.tsx
type Status = "idle" | "generating" | "complete" | "error";

export function AIGenerationStatus({
  status,
  onRetry,
}: {
  status: Status;
  onRetry?: () => void;
}) {
  if (status === "idle") return null;

  return (
    <div
      role="status"
      aria-live="polite"
      aria-label={
        status === "generating" ? "Generating..." :
        status === "complete" ? "Complete" :
        "Generation failed"
      }
    >
      {status === "generating" && (
        <div className="flex items-center gap-2 text-sm text-gray-500">
          <Spinner size="sm" />
          <span>Generating...</span>
        </div>
      )}
      {status === "error" && (
        <div className="flex items-center gap-2 text-sm text-red-600">
          <span>Something went wrong.</span>
          {onRetry && (
            <button onClick={onRetry} className="underline">
              Try again
            </button>
          )}
        </div>
      )}
    </div>
  );
}
```

### Pattern 3: Destructive Action Confirmation
```typescript
// Only use for truly irreversible, high-consequence actions
export function DeleteConfirmationDialog({
  itemName,
  onConfirm,
  onCancel,
}: {
  itemName: string;
  onConfirm: () => void;
  onCancel: () => void;
}) {
  return (
    <Dialog open onOpenChange={onCancel}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Delete reminder?</DialogTitle>
          <DialogDescription>
            "{itemName}" will be permanently deleted. This cannot be undone.
          </DialogDescription>
        </DialogHeader>
        <DialogFooter>
          <Button variant="ghost" onClick={onCancel}>Cancel</Button>
          <Button variant="destructive" onClick={onConfirm}>Delete</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Expose System State in User-Facing Errors
```typescript
// ❌ Exposes internal architecture to users
toast.error("PostgreSQL error: duplicate key value violates unique constraint");

// ✅ User-friendly without revealing internals
toast.error("A reminder with this name already exists. Please use a different name.");
```

### Rule 2: Confirm Before Destructive Actions
Any action that permanently removes data (delete, cancel subscription, revoke access) must have a confirmation step. No exceptions. One accidental tap should not lose user data.

### Rule 3: Never Auto-Submit Forms
Forms should only submit when the user explicitly clicks a submit button. Never submit on blur, on a timer, or without clear user intent. This is both a UX and a trust issue.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Building Screens Without Designing the Empty and Error States
Every list you build will sometimes be empty. Every AI call will sometimes fail. If you do not design these states before building, users see blank screens and unexplained failures.  
**Fix:** Before writing any component that loads data or calls AI, define all three states: empty, loading, populated (and error for AI calls).

### ❌ Using Placeholder Text as Labels
```html
<!-- ❌ When user starts typing, the label disappears -->
<input placeholder="Email address" />

<!-- ✅ Label always visible -->
<label>Email address</label>
<input placeholder="you@example.com" />
```

### ❌ Confirmation Dialogs for Every Action
Asking "Are you sure?" for marking a reminder complete trains users to dismiss dialogs without reading them. When the actual destructive action (delete) requires confirmation, they dismiss it too.  
**Fix:** Confirmation dialogs only for actions that are irreversible AND high-consequence. Completing a reminder is neither — no dialog needed.

### ❌ AI Input With No Guidance
An empty text box with no placeholder, no examples, and no hint of what to type is intimidating. Users freeze.  
**Fix:** Every AI input needs: a descriptive placeholder, 2–3 example prompts, and a clear CTA button.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your UX Practice

### Run a 5-Second Test
Show someone your key screen for 5 seconds, then hide it. Ask:
- "What is this for?"
- "What can you do here?"
- "Who is it for?"

If they cannot answer accurately in 5 seconds, the screen's hierarchy and copy need work.

### Add User Feedback Loops
```typescript
// Thumb up/down on AI-generated content
// This data feeds into your model evaluation and prompt improvement
export function AIFeedback({ generationId }: { generationId: string }) {
  const [rating, setRating] = useState<"up" | "down" | null>(null);

  const handleFeedback = async (value: "up" | "down") => {
    setRating(value);
    await fetch("/api/feedback", {
      method: "POST",
      body: JSON.stringify({ generationId, rating: value }),
    });
  };

  return (
    <div className="flex gap-2 mt-2">
      <button onClick={() => handleFeedback("up")} aria-label="This was helpful">
        👍
      </button>
      <button onClick={() => handleFeedback("down")} aria-label="This was not helpful">
        👎
      </button>
    </div>
  );
}
```

### Implement Progressive Onboarding
Instead of a long onboarding form, use contextual prompts that appear when the user first encounters a feature:
```
First time the user opens the lesson generator:
"Welcome! Start by typing any topic — like 'fractions' or 'World War 2' — and we will create a personalised lesson."

First time the user sees an empty lesson list:
"Your lessons will appear here after you create them. Ready to start?"
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Reducing the Core Flow from 7 to 3 Steps
```
Original flow:
1. Tap "New Reminder"
2. Type reminder text
3. Open date picker → select date
4. Open time picker → select time
5. Toggle recurrence on
6. Select recurrence type
7. Tap "Save"
Total: 7 steps, 3 separate pickers

Friction audit identified:
- Steps 3-6 can be replaced with NLP parsing (AI does the work)
- Step 7 can be a single "Create" tap

Redesigned flow:
1. Type reminder in natural language ("call mum Sunday 3pm weekly")
2. AI parses → shows parsed fields for confirmation
3. Tap "Create"
Total: 3 steps

Result: Reminder creation time (median): 45 seconds → 12 seconds
User retention at day 7 improved from 31% to 48%
The UX change was larger than the AI capability change.
```

### Case Study 2: EdTech App — Empty State That Increased Lesson Creation by 40%

**Before:** Empty lesson list showed: "No lessons yet." with no further guidance.

**After:**
```
Title: "Your first lesson is one topic away"
Subtext: "Type any topic — 'photosynthesis', 'the French Revolution', 'algebra basics' — and we will create a personalised lesson in seconds."
Example chips: "Photosynthesis" | "World War II" | "Fractions"
CTA button: "Create my first lesson"
```

**Result:** 40% more users created a lesson within the first 5 minutes of signing up. The empty state was the highest-leverage UX change made in the entire product's history.

</real_world_examples>

</skill_document>
