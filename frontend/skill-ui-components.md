# Skill: UI Components

```json
{
  "skill_id": "ui-components",
  "category": "Frontend & UI/UX",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>UI Components — Building a Reusable, Accessible Component Library for AI Products</title>

<overview>

## What This Skill Enables
- Build a small, consistent set of UI components that power your entire app without duplication
- Use shadcn/ui and Radix UI primitives correctly so you get accessibility for free
- Design components that are easy for your AI agent to reuse reliably across every page

## Why It Matters for Vibe Coders
Without a component system, every page gets its own slightly different button, its own slightly different input, and its own slightly different modal. The AI agent generates slight variations every session because it has no shared reference. A component library — even a tiny one of 8–10 components — eliminates this drift and makes every future page faster to build.

## When to Use This Skill
- Before building any feature pages (build your UI primitives first)
- When you notice the same button or card being rebuilt slightly differently in multiple files
- When adding a new shared UI pattern (toast notifications, data tables, file upload)
- When your current components fail keyboard navigation or screen reader tests

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / React]",
    "styling": "[REPLACE: e.g. Tailwind CSS v3 / Tailwind CSS v4]",
    "component_library": "[REPLACE: e.g. shadcn/ui / Radix UI / Headless UI / none]",
    "design_tokens": {
      "primary_color": "[REPLACE: e.g. indigo / blue / violet]",
      "border_radius": "[REPLACE: e.g. rounded-md / rounded-lg / rounded-full]",
      "font": "[REPLACE: e.g. Inter / Geist / DM Sans]"
    },
    "components_already_built": [
      "[REPLACE: e.g. Button, Input, Card]"
    ],
    "component_to_build_today": "[REPLACE: ONE component only]",
    "where_this_component_is_used": "[REPLACE: e.g. reminder list, lesson cards, chat interface]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About UI Components

### Mental Model 1: The LEGO Brick System
Good UI components are like LEGO bricks:
- Each brick does one thing and does it well
- Bricks snap together predictably
- You do not redesign the 2x4 brick every time you build something new
- Complex structures are built from simple, reusable pieces

A Button is a brick. A Card is a brick. A Modal is a brick built from smaller bricks (overlay, panel, close button).

### Mental Model 2: Three Levels of Components

| Level | Responsibility | Examples |
|---|---|---|
| **Primitive** | Raw HTML with styling and accessibility | Button, Input, Label, Badge |
| **Composite** | Combines primitives into a pattern | FormField (Label + Input + Error), Card (header + body + footer) |
| **Feature** | Business-logic-aware | ReminderCard (Card + data), LessonBadge (Badge + lesson status) |

Only Primitive and Composite components live in `/components/ui`. Feature components live in `/components/features/`.

### Mental Model 3: Props = Configuration
A good component is configured entirely through props. No hardcoded colours, sizes, or text inside the component file:

```typescript
// ❌ Hardcoded — cannot reuse in a different context
<button className="bg-indigo-600 text-white px-4 py-2 rounded-md">
  Save Changes
</button>

// ✅ Configurable — reusable everywhere
<Button variant="primary" size="md">
  Save Changes
</Button>
```

</mental_models>

---

<system_design_breakdown>

## The Core Component Set for AI Products

Build these 10 components first. Everything else is built from or alongside them.

```
/components/ui/
  button.tsx           ← variants: primary, secondary, ghost, destructive; sizes: sm, md, lg
  input.tsx            ← text, email, password, search; with error state
  textarea.tsx         ← auto-resize; with character count; with error state
  card.tsx             ← header, body, footer slots; optional hover state
  modal.tsx            ← accessible overlay; trap focus; close on Escape
  spinner.tsx          ← loading indicator; sizes match button sizes
  empty-state.tsx      ← icon + title + description + optional CTA
  badge.tsx            ← variants: default, success, warning, error, info
  toast.tsx            ← success/error/info notifications; auto-dismiss
  skeleton.tsx         ← content placeholder shapes for loading states
```

## Component Anatomy

Every component should have this structure:

```typescript
// 1. Props interface — explicitly typed, no implicit any
interface ButtonProps {
  variant?: "primary" | "secondary" | "ghost" | "destructive";
  size?: "sm" | "md" | "lg";
  isLoading?: boolean;
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
  type?: "button" | "submit" | "reset";
  className?: string; // always allow className override
}

// 2. Variant map — separates styling logic from render logic
const variants = {
  primary: "bg-indigo-600 text-white hover:bg-indigo-700",
  secondary: "bg-white text-gray-900 border border-gray-200 hover:bg-gray-50",
  ghost: "text-gray-600 hover:bg-gray-100",
  destructive: "bg-red-600 text-white hover:bg-red-700",
};

// 3. Component — clean, readable render
export function Button({ variant = "primary", size = "md", isLoading, ...props }: ButtonProps) {
  return (
    <button
      className={cn(baseStyles, variants[variant], sizes[size], props.className)}
      disabled={isLoading || props.disabled}
      {...props}
    >
      {isLoading ? <Spinner size="sm" /> : props.children}
    </button>
  );
}
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: One component at a time. Build it, use it in one real place, then move on. -->

## Step 1 — Set Up shadcn/ui (Recommended Starting Point)
Skip this step if already set up or using a different library.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Set up shadcn/ui in my Next.js project.
Run the init command and configure:
- Style: default
- Base colour: [YOUR PRIMARY COLOR e.g. slate / zinc / neutral]
- CSS variables: yes
- Tailwind config: yes

Then add these components: button, input, card, badge, dialog, toast, skeleton
Show me the install commands only — do not modify any existing files.
```

**Verify:** Run the install commands. Confirm `/components/ui/button.tsx` exists with shadcn's code.

---

## Step 2 — Customise the Button Component
First component to customise — used everywhere.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Base: shadcn/ui Button (already installed at /components/ui/button.tsx)

Extend the Button component to also support:
1. isLoading prop — shows a spinner, disables the button, keeps the same size
2. leftIcon / rightIcon props — renders an icon before/after the label
3. fullWidth prop — makes button stretch to container width

Do not replace shadcn's variant system — extend it.
Keep the component accessible (aria-disabled when loading).
```

**Verify:** Render the button in 4 states: normal, loading, with icon, full width. Confirm all look correct.

---

## Step 3 — Build the Empty State Component
This is a high-frequency component — every list page needs it.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create /components/ui/empty-state.tsx with:
Props:
- icon: React.ReactNode (optional)
- title: string (required)
- description: string (optional)
- action: { label: string, onClick: () => void } (optional)

Styling: centred, generous spacing, muted text colour, consistent with [YOUR DESIGN TOKENS]
The action renders as a primary Button component.
Export with named export: EmptyState
```

**Verify:** Render with and without the action prop. Confirm it looks correct in both cases.

---

## Step 4 — Build the Skeleton Loading Component
**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Create /components/ui/skeleton.tsx with:
1. Base Skeleton component: takes className prop, renders an animated pulse placeholder
2. Pre-built skeleton shapes:
   - SkeletonText: simulates a line of text (full width by default, customisable)
   - SkeletonCard: simulates a card with a title line + 2 body lines
   - SkeletonAvatar: simulates a circular avatar

Use Tailwind's animate-pulse. Match border-radius to [YOUR BORDER_RADIUS token].
```

**Verify:** Replace a loading state with `<SkeletonCard />`. Confirm it matches the rough shape of the actual card.

---

## Step 5 — Build Feature-Specific Components
Using your UI primitives as building blocks.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Available UI components: Button, Input, Card, Badge, Spinner, EmptyState, Skeleton

Build the [FEATURE_COMPONENT_NAME] component at /components/features/[feature]/[name].tsx
It should:
- [DESCRIBE WHAT IT RENDERS]
- Use existing UI components — do not re-implement Button, Card, etc. from scratch
- Accept these props: [LIST PROPS]
- Emit these events/callbacks: [LIST CALLBACKS]
- Handle its own loading state using the Skeleton component
- Be a Client Component only if it needs interactivity

Do not fetch data inside this component — receive data as props.
```

**Verify:** Render the component in a page with hardcoded test data. Confirm it looks correct before wiring up real data.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am building UI components for my app.
Project context: [PASTE context_anchor JSON]
Components already built: [LIST]
Component library: [shadcn/ui / other]

Today I am building: [ONE COMPONENT ONLY]
Before writing code:
1. Check if a similar component already exists in /components/ui
2. Tell me if shadcn/ui already has this — if so, I want to extend it, not replace it
3. List the props this component will accept
```

### shadcn Component Extension Prompt
```
I have shadcn/ui's [COMPONENT] installed at /components/ui/[component].tsx
Extend it (do not replace it) to add:
[LIST EXTENSIONS]

Constraints:
- Keep all existing shadcn variants and props working
- New props should have sensible defaults
- Maintain accessibility attributes
- Match the styling system: [Tailwind classes / CSS variables]
```

### New Primitive Component Prompt
```
Create a new primitive UI component: [COMPONENT NAME]
File: /components/ui/[name].tsx

Props: [LIST ALL PROPS WITH TYPES AND WHETHER REQUIRED/OPTIONAL]
Variants: [LIST VARIANTS IF ANY]
Sizes: [LIST SIZES IF ANY]
Accessibility requirements:
- [ARIA role if not standard HTML element]
- [Keyboard interaction if interactive]
- [Focus management if modal/dropdown]

Styling: Tailwind CSS, use cn() utility for class merging
Export: named export [ComponentName]
Do not import from other feature components — only from /components/ui/ or external libraries.
```

### Component Audit Prompt
```
Audit my /components directory for inconsistencies.
Project context: [PASTE context_anchor JSON]

Find:
1. Duplicate components doing the same thing in different feature folders
2. Components that hardcode colours/sizes instead of using Tailwind tokens
3. Interactive components missing keyboard navigation or aria attributes
4. Components importing from feature folders they should not know about

Return a table: component file, issue, recommended fix.
Do not change any files.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "Should I use shadcn/ui or build my own components?"

**Use shadcn/ui (strongly recommended for almost everyone):**
- Gives you 40+ well-designed, accessible components instantly
- Code lives in your project (you own it, not locked to a package)
- Built on Radix UI — keyboard navigation and screen reader support built in
- Tailwind-based — matches whatever styling you are already using
- AI agents know shadcn/ui well and generate correct usage reliably

**Build your own only if:**
- You have very specific design requirements that shadcn cannot accommodate
- You are building for a platform where shadcn is not available (e.g. React Native)

**The hybrid approach that works best:** Install shadcn/ui → extend their components with your app's specific needs → build the few custom ones that shadcn does not have (EmptyState, Skeleton, custom chat bubbles).

---

### "What is accessibility and why should I care if I am just building an MVP?"

Accessibility means your app works for people who use keyboards (no mouse), screen readers (visually impaired users), or have motor difficulties.

**The practical reason to care even at MVP stage:**
- Proper keyboard navigation (Tab, Enter, Escape) is used by power users who prefer keyboard shortcuts — not just people with disabilities
- Screen reader support catches broken UI states your eyes miss
- shadcn/ui gives you most of this for free — it costs you nothing to have it

**The one thing you must never do:** build a custom dropdown, modal, or date picker from scratch. These have deeply complex keyboard and ARIA requirements. Use shadcn/Radix — they have solved this.

---

### "How do I decide what deserves its own component vs just inline Tailwind?"

**Make a component when:**
- The same visual pattern appears in 2+ places
- The pattern has interactive behaviour (click handlers, state)
- The pattern has more than ~5 Tailwind classes

**Inline Tailwind is fine when:**
- It is a one-off layout detail (a specific margin, a grid layout)
- The element has no behaviour
- It only exists in one place and will not be reused

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing UI Components

### Visual Checklist for Every Component
```
□ Renders correctly with minimum required props
□ Renders correctly with all optional props provided
□ Loading state (if applicable) shows skeleton/spinner
□ Error state (if applicable) shows readable message
□ Empty/null data does not cause crash
□ Hover state is visually distinct
□ Focus state is visible (outline or ring — never removed)
□ Disabled state is visually distinct and non-interactive
□ Responsive: looks correct on mobile (375px) and desktop (1280px)
□ Dark mode: readable if your app supports it
```

### Accessibility Quick Check
```bash
# Install axe accessibility checker as a browser extension
# https://www.deque.com/axe/browser-extensions/

# Or add to your dev setup:
npm install --save-dev @axe-core/react

# In your _app or layout (development only):
if (process.env.NODE_ENV !== "production") {
  const ReactDOM = await import("react-dom");
  const axe = await import("@axe-core/react");
  axe.default(React, ReactDOM, 1000);
}
```

### Keyboard Navigation Test
For every interactive component (Button, Input, Modal, Dropdown):
1. Tab to the component — it should receive visible focus
2. Enter/Space on a button — it should activate
3. Escape on a modal/dropdown — it should close
4. Arrow keys on a select/menu — they should navigate options

### Common Component Errors

| Error | Meaning | Fix |
|---|---|---|
| `Warning: Each child in a list should have a unique "key" prop` | Missing key in `.map()` | Add `key={item.id}` to each list item |
| Component renders but has no style | Tailwind classes not applied | Check cn() utility is installed; check class names for typos |
| Modal doesn't close on Escape | Custom modal missing key handler | Use shadcn Dialog — Radix handles this automatically |
| Input loses focus when state updates | Component remounting on state change | Move component definition outside of render function |
| `className` prop not working | Component not accepting className | Add `className?: string` to props and apply with `cn()` |

</testing_and_qa>

---

<common_patterns>

## Reusable Component Patterns

### Pattern 1: The cn() Utility (Install Once, Use Everywhere)
```typescript
// /lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
// npm install clsx tailwind-merge
```

### Pattern 2: Full Button Component
```typescript
// /components/ui/button.tsx
import { cn } from "@/lib/utils";
import { Spinner } from "./spinner";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "ghost" | "destructive";
  size?: "sm" | "md" | "lg";
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  fullWidth?: boolean;
}

const variants = {
  primary:     "bg-indigo-600 text-white hover:bg-indigo-700 focus-visible:ring-indigo-500",
  secondary:   "bg-white text-gray-900 border border-gray-200 hover:bg-gray-50 focus-visible:ring-gray-400",
  ghost:       "text-gray-600 hover:bg-gray-100 focus-visible:ring-gray-400",
  destructive: "bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-500",
};

const sizes = {
  sm: "h-8 px-3 text-sm gap-1.5",
  md: "h-10 px-4 text-sm gap-2",
  lg: "h-12 px-6 text-base gap-2",
};

export function Button({
  variant = "primary", size = "md", isLoading, leftIcon, fullWidth, children, className, ...props
}: ButtonProps) {
  return (
    <button
      className={cn(
        "inline-flex items-center justify-center rounded-md font-medium transition-colors",
        "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2",
        "disabled:pointer-events-none disabled:opacity-50",
        variants[variant], sizes[size],
        fullWidth && "w-full",
        className
      )}
      disabled={isLoading || props.disabled}
      aria-busy={isLoading}
      {...props}
    >
      {isLoading ? <Spinner size={size === "lg" ? "md" : "sm"} /> : (
        <>{leftIcon}{children}</>
      )}
    </button>
  );
}
```

### Pattern 3: Empty State Component
```typescript
// /components/ui/empty-state.tsx
import { Button } from "./button";
import { cn } from "@/lib/utils";

interface EmptyStateProps {
  icon?: React.ReactNode;
  title: string;
  description?: string;
  action?: { label: string; onClick: () => void };
  className?: string;
}

export function EmptyState({ icon, title, description, action, className }: EmptyStateProps) {
  return (
    <div className={cn("flex flex-col items-center justify-center py-16 px-4 text-center", className)}>
      {icon && <div className="mb-4 text-gray-400">{icon}</div>}
      <h3 className="text-lg font-medium text-gray-900">{title}</h3>
      {description && <p className="mt-2 text-sm text-gray-500 max-w-sm">{description}</p>}
      {action && (
        <Button variant="primary" size="md" className="mt-6" onClick={action.onClick}>
          {action.label}
        </Button>
      )}
    </div>
  );
}
```

### Pattern 4: Skeleton Components
```typescript
// /components/ui/skeleton.tsx
import { cn } from "@/lib/utils";

export function Skeleton({ className }: { className?: string }) {
  return <div className={cn("animate-pulse rounded-md bg-gray-200", className)} />;
}

export function SkeletonText({ className }: { className?: string }) {
  return <Skeleton className={cn("h-4 w-full", className)} />;
}

export function SkeletonCard() {
  return (
    <div className="rounded-lg border border-gray-200 p-4 space-y-3">
      <Skeleton className="h-5 w-2/3" />
      <SkeletonText />
      <SkeletonText className="w-4/5" />
    </div>
  );
}

export function SkeletonAvatar({ size = "md" }: { size?: "sm" | "md" | "lg" }) {
  const sizes = { sm: "h-8 w-8", md: "h-10 w-10", lg: "h-12 w-12" };
  return <Skeleton className={cn("rounded-full", sizes[size])} />;
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Use dangerouslySetInnerHTML Without Sanitisation
```typescript
// ❌ XSS risk — if content contains <script>, it executes
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// ✅ Sanitise first
import DOMPurify from "isomorphic-dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent) }} />
```

### Rule 2: Never Remove Focus Outlines
```css
/* ❌ Removes keyboard navigation visibility for all users */
* { outline: none; }
button:focus { outline: none; }

/* ✅ Remove the default ugly outline but provide a custom visible one */
button:focus-visible { ring: 2px; ring-offset: 2px; ring-color: indigo; }
```

### Rule 3: Always Use Button Type Attribute
```typescript
// ❌ Default type="submit" — accidentally submits a form
<button onClick={handleAction}>Do Thing</button>

// ✅ Explicit type
<button type="button" onClick={handleAction}>Do Thing</button>
<button type="submit">Submit Form</button>
```

### Rule 4: Validate File Uploads Before Displaying
```typescript
// If displaying user-uploaded images or files:
const ALLOWED_TYPES = ["image/jpeg", "image/png", "image/webp"];
const MAX_SIZE_BYTES = 5 * 1024 * 1024; // 5MB

if (!ALLOWED_TYPES.includes(file.type)) throw new Error("File type not allowed");
if (file.size > MAX_SIZE_BYTES) throw new Error("File too large");
```

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Building Feature Logic Into UI Components
```typescript
// ❌ UI component fetches its own data — cannot be reused elsewhere
export function ReminderCard() {
  const { data } = useReminders(); // Business logic in UI layer
  return <Card>{data.title}</Card>;
}

// ✅ UI component receives data as props — reusable anywhere
export function ReminderCard({ reminder }: { reminder: Reminder }) {
  return <Card>{reminder.title}</Card>;
}
```

### ❌ Copy-Pasting Components Instead of Making Props Flexible
Every time a component is copied and slightly modified, you have two things to update when the design changes.  
**Fix:** Add a variant or prop instead of copying.

### ❌ Hardcoding Colours in Component Files
```typescript
// ❌ Cannot be themed or changed globally
className="bg-indigo-600"  // inside a component that should be generic

// ✅ Use semantic class names or CSS variables
className="bg-primary"     // defined in your Tailwind config
```

### ❌ Building a Custom Modal, Dropdown, or Date Picker
These three patterns have complex accessibility requirements (focus trap, ARIA roles, keyboard navigation). Every custom implementation gets them wrong in at least one way.  
**Fix:** Use shadcn Dialog, shadcn DropdownMenu, and a library like react-day-picker. The accessibility is solved.

### ❌ Not Handling the Loading State of Every Component That Fetches Data
Users see a blank or partially-rendered component while data loads. This feels broken.  
**Fix:** Build the skeleton variant of a component at the same time as the populated variant.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Component Library

### Add a Component Storybook
When your app has more than 15 components, Storybook lets you browse, test, and document them visually:
```
Ask your AI agent: "Set up Storybook for my Next.js project.
Create stories for: Button (all variants), Card, EmptyState, and the most complex feature component."
```

### Add Dark Mode Support
```typescript
// tailwind.config.ts
module.exports = {
  darkMode: "class", // Toggle by adding/removing 'dark' class on <html>
  // All components using text-gray-900 automatically become text-gray-100 in dark mode
  // when you add: dark:text-gray-100 to your component
};

// Toggle with:
document.documentElement.classList.toggle("dark");
```

### Add Component Composition with Slots
For highly configurable components (like a Card that can have many different header configurations), use the slot pattern:
```typescript
// Flexible Card composition
<Card>
  <Card.Header>
    <Card.Title>Lesson Title</Card.Title>
    <Card.Badge>Intermediate</Card.Badge>
  </Card.Header>
  <Card.Body>Content here</Card.Body>
  <Card.Footer>
    <Button>Start Lesson</Button>
  </Card.Footer>
</Card>
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: The Reminder App Component Audit
At week 3 of building, the reminder app had accumulated:
- 3 different button implementations (different padding, different colours, different hover states)
- 2 different spinner components (one from copy-paste, one from shadcn)
- No empty state component — every page showed a blank div when there was no data

**Fix applied in one session:**
```
1. Deleted all custom button implementations
2. Installed shadcn/ui Button, extended with isLoading prop
3. Used shadcn Spinner everywhere (deleted custom one)
4. Built EmptyState component — added to all 4 list pages
5. Added SkeletonCard — replaced all loading spinners on list pages

Result: UI became visually consistent. AI agent sessions became faster
because every prompt could reference the same component names with confidence.
```

### Case Study 2: EdTech App — Lesson Status Badge System
**Requirement:** Lessons needed to show statuses: Not Started, In Progress, Completed, Locked

```typescript
// Instead of 4 different styled spans across the codebase:
type LessonStatus = "not_started" | "in_progress" | "completed" | "locked";

const statusConfig: Record<LessonStatus, { label: string; variant: BadgeVariant; icon: string }> = {
  not_started: { label: "Not Started", variant: "default",  icon: "○" },
  in_progress: { label: "In Progress", variant: "warning",  icon: "◑" },
  completed:   { label: "Completed",   variant: "success",  icon: "●" },
  locked:      { label: "Locked",      variant: "secondary", icon: "🔒" },
};

export function LessonStatusBadge({ status }: { status: LessonStatus }) {
  const config = statusConfig[status];
  return (
    <Badge variant={config.variant}>
      {config.icon} {config.label}
    </Badge>
  );
}

// Used identically across: LessonCard, LessonList, LessonPlayer header, StudentDashboard
// When "Mastered" status was added later: one object entry change, not 4 file edits
```

</real_world_examples>

</skill_document>
