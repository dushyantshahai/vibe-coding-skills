# Skill: Responsive Design

```json
{
  "skill_id": "responsive-design",
  "category": "Frontend & UI/UX",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Responsive Design — Building AI Product UIs That Work on Every Screen</title>

<overview>

## What This Skill Enables
- Build layouts that look correct and feel native on mobile (375px), tablet (768px), and desktop (1280px+) without rebuilding components for each screen size
- Apply Tailwind CSS's responsive system correctly every time
- Design app shells (sidebar + main content) that adapt gracefully between mobile and desktop

## Why It Matters for Vibe Coders
Most vibe-coded apps are built on a laptop and break on a phone. For AI products — especially reminder apps, tutoring apps, and chat interfaces — mobile is often the primary use case. A broken mobile layout is not a polish issue; it is a retention issue. This skill gives you a system for building responsive layouts once, correctly.

## When to Use This Skill
- When designing any new page or layout component
- When your app looks broken on mobile (375px viewport)
- When building a sidebar + content layout (the most common AI app shell)
- When adding a chat interface that needs to fill the full screen height on mobile

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js App Router]",
    "styling": "[REPLACE: e.g. Tailwind CSS v3 / v4]",
    "primary_use_case": "[REPLACE: e.g. mobile-first / primarily desktop / equal split]",
    "app_shell_type": "[REPLACE: e.g. sidebar + content / top nav + content / full-screen chat]",
    "layout_to_fix_or_build_today": "[REPLACE: ONE layout or component]",
    "existing_layout_files": "[REPLACE: e.g. /app/(app)/layout.tsx, /components/sidebar.tsx]",
    "breakpoints_that_matter": "[REPLACE: e.g. mobile 375px, tablet 768px, desktop 1280px]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Responsive Design

### Mental Model 1: Mobile First
Always design the mobile layout first, then enhance it for larger screens. This is not just a philosophy — Tailwind's responsive system is built this way.

```
Tailwind class with no prefix = applies on ALL screen sizes (mobile and up)
sm: prefix               = applies at 640px and up
md: prefix               = applies at 768px and up
lg: prefix               = applies at 1024px and up
xl: prefix               = applies at 1280px and up
```

**The key insight:** An unprefixed class is your mobile style. Prefixed classes override it on larger screens.

```html
<!-- Mobile: full width, stacked. Desktop: side by side, each half width -->
<div class="flex flex-col md:flex-row">
  <div class="w-full md:w-1/2">Left</div>
  <div class="w-full md:w-1/2">Right</div>
</div>
```

### Mental Model 2: The Three-Layout System
Almost every AI product uses one of three layouts. Choose yours before building:

| Layout | Mobile | Desktop | Best For |
|---|---|---|---|
| **Stack** | Single column, top nav | Single column, top nav | Simple apps, landing pages |
| **Sidebar** | Bottom tab bar or hamburger menu | Fixed sidebar + main content | Dashboard apps, lesson apps |
| **Full-screen chat** | Chat takes full viewport height | Chat + sidebar history | Chat-first apps |

### Mental Model 3: The Viewport Height Problem
On mobile browsers, `100vh` includes the browser's address bar, which hides when scrolling. Your "full-screen" layout jumps when the address bar hides.

**Fix:** Use `100dvh` (dynamic viewport height) instead of `100vh` on mobile layouts.
```html
<div class="h-[100dvh]">Full screen chat</div>
```

</mental_models>

---

<system_design_breakdown>

## Responsive App Shell Patterns

### Pattern A: Sidebar + Content (Dashboard Apps)
```
MOBILE (< 768px)              DESKTOP (≥ 1024px)
┌─────────────────┐           ┌──────┬──────────────────┐
│    Top Header   │           │      │    Top Header    │
├─────────────────┤           │  S   ├──────────────────┤
│                 │           │  I   │                  │
│   Main Content  │           │  D   │   Main Content   │
│   (full width)  │           │  E   │                  │
│                 │           │  B   │                  │
│                 │           │  A   │                  │
│                 │           │  R   │                  │
├─────────────────┤           └──────┴──────────────────┘
│  Bottom Tab Bar │
└─────────────────┘
```

### Pattern B: Full-Screen Chat
```
MOBILE                        DESKTOP
┌─────────────────┐           ┌──────────┬───────────────┐
│ Chat Header     │           │          │  Chat Header  │
├─────────────────┤           │ History  ├───────────────┤
│                 │           │ Sidebar  │               │
│ Messages        │           │          │ Messages      │
│ (scrollable)    │           │          │ (scrollable)  │
│                 │           │          │               │
├─────────────────┤           │          ├───────────────┤
│ Input Bar       │           │          │  Input Bar    │
└─────────────────┘           └──────────┴───────────────┘
```

## Tailwind Breakpoint Reference

```
Default (no prefix) → all screens, mobile first
sm  → 640px+   (large phones, small tablets)
md  → 768px+   (tablets)
lg  → 1024px+  (laptops)
xl  → 1280px+  (desktops)
2xl → 1536px+  (large monitors)
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Build mobile layout first, verify it, then add desktop breakpoints. -->

## Step 1 — Define Your App Shell
Before any page content.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
App shell type: [SIDEBAR+CONTENT / FULL-SCREEN CHAT / STACK]

Build the authenticated app layout at /app/(app)/layout.tsx
Mobile behaviour: [DESCRIBE — e.g. hamburger menu, bottom tab bar, hidden sidebar]
Desktop behaviour: [DESCRIBE — e.g. fixed sidebar 240px wide, collapsible]

Requirements:
1. Mobile first — start with the mobile layout, then add md:/lg: overrides
2. Use h-[100dvh] not h-screen for full-height layouts
3. Sidebar must be dismissable on mobile with an overlay
4. Main content area must be independently scrollable
5. No horizontal overflow (overflow-x-hidden on the root)

Build the shell with placeholder content — no real feature components yet.
```

**Verify:** Open the layout at 375px (Chrome DevTools → responsive mode). Confirm no horizontal scroll and the mobile layout looks correct.

---

## Step 2 — Build the Sidebar Component
For sidebar-style apps only.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Build /components/layout/sidebar.tsx with:
Mobile:
- Hidden by default (translate-x-[-100%])
- Slides in from left when isOpen is true
- Dark overlay behind it when open
- Close on overlay click or Escape key
- Width: w-72 (288px)

Desktop (lg: breakpoint):
- Always visible (static, no overlay)
- Fixed position on the left
- Width: w-60 (240px)

Content: navigation links (receive as props), user avatar at bottom
Use Zustand UI store to manage isOpen state — both this component and 
the hamburger button need to control it.
```

**Verify:** On mobile, confirm sidebar is hidden by default and slides in correctly. On desktop (1024px+), confirm it is always visible with no overlay.

---

## Step 3 — Make Every Page Component Responsive
Apply this pattern to each new page.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Make the [PAGE/COMPONENT NAME] responsive.
Current layout issue: [DESCRIBE what breaks on mobile]

Apply these rules:
1. Replace any fixed pixel widths with max-w-* + w-full
2. Replace horizontal flex layouts with flex-col on mobile, flex-row on md:
3. Replace fixed font sizes with responsive sizes: text-sm md:text-base
4. Replace fixed padding with responsive: p-4 md:p-6 lg:p-8
5. Any grid with 3+ columns: grid-cols-1 sm:grid-cols-2 lg:grid-cols-3
6. Tables: wrap in overflow-x-auto on mobile

Do not change any business logic or data fetching.
```

**Verify at these widths:** 375px (iPhone SE), 390px (iPhone 14), 768px (iPad), 1280px (laptop). Check each in Chrome DevTools.

---

## Step 4 — Fix the Full-Screen Chat Layout for Mobile
The most common layout challenge in AI products.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Fix the chat interface layout so it works correctly on mobile.
Problem: [DESCRIBE — e.g. input is hidden behind keyboard, or messages don't scroll independently]

Requirements:
1. Chat container: h-[100dvh] flex flex-col (not h-screen — avoids browser bar jump)
2. Messages area: flex-1 overflow-y-auto (takes remaining space, independently scrollable)
3. Input bar: sticky at the bottom, does NOT scroll away
4. When virtual keyboard opens on mobile: input stays visible (use env(safe-area-inset-bottom))
5. Message list auto-scrolls to bottom when new messages arrive

Apply to: [CHAT COMPONENT FILE PATH]
Do not change the streaming logic.
```

**Verify:** Open on a real phone or in Chrome DevTools with device emulation. Type a message — confirm the input remains visible when the keyboard appears.

---

## Step 5 — Add Safe Area Insets for iOS
For any app used on iPhones with home indicators.

**Prompt your AI agent:**
```
Add iOS safe area inset support to my app.
Affected components: the bottom tab bar, the chat input bar, and any fixed bottom elements.

In tailwind.config.ts, add:
plugins: [require('tailwindcss-safe-area')]

Then apply: pb-safe (or pb-[env(safe-area-inset-bottom)]) to bottom-fixed elements.
This prevents content from being hidden behind the iPhone home indicator.
```

**Verify:** Use Chrome DevTools → iPhone 14 Pro emulation. Check that bottom elements are not cut off by the home indicator.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on responsive layout for my app.
Project context: [PASTE context_anchor JSON]

Layout issues to fix today: [DESCRIBE — one layout at a time]
Test viewports I need to support: [375px / 768px / 1280px]

Before writing code:
1. Identify which files need to change
2. Confirm you will use mobile-first (no prefix = mobile, prefixes = larger screens)
3. List any Tailwind classes being replaced
```

### App Shell Layout Prompt
```
Build a responsive app shell for my [APP TYPE].
Framework: Next.js App Router
File: /app/(app)/layout.tsx + /components/layout/

Shell type: [SIDEBAR+CONTENT / TOP-NAV / FULL-SCREEN]
Mobile: [DESCRIBE MOBILE LAYOUT — e.g. bottom tab bar with 4 icons]
Desktop: [DESCRIBE DESKTOP LAYOUT — e.g. 240px fixed left sidebar]

Requirements:
- Mobile-first Tailwind classes
- h-[100dvh] for full-height containers
- Sidebar uses Zustand for open/close state (shared with hamburger button)
- Main content area has its own scroll, not the page scroll
- No content hidden behind fixed headers/footers
```

### Responsive Grid Prompt
```
Make this layout responsive: [DESCRIBE THE LAYOUT — e.g. lesson cards grid, reminder list]
Current code: [PASTE THE RELEVANT JSX/TSX]

Target behaviour:
- Mobile (< 640px): [DESCRIBE — e.g. single column, full width cards]
- Tablet (640–1024px): [DESCRIBE — e.g. 2 columns]
- Desktop (1024px+): [DESCRIBE — e.g. 3 columns]

Use Tailwind responsive prefixes only. Do not use CSS media queries directly.
Do not change any component logic or data.
```

### Mobile Viewport Debug Prompt
```
My [PAGE/COMPONENT] looks broken at 375px viewport width.
Here is the current code: [PASTE]
Here is what breaks: [DESCRIBE — e.g. horizontal scroll, elements overflow, text too large]

Fix the responsive layout using mobile-first Tailwind classes.
Check for:
1. Fixed pixel widths that should be w-full or max-w-*
2. flex-row that should be flex-col on mobile
3. Absolute positioned elements overflowing the viewport
4. Font sizes that are too large for small screens
5. Padding/margin values that waste space on mobile

Return only the changed JSX/TSX. No logic changes.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is the fastest way to check if my app looks good on mobile?"

Chrome DevTools → Cmd+Shift+I (Mac) or F12 (Windows) → Click the phone icon at the top left of DevTools → Select "iPhone SE" (375px, the toughest test).

Check in this order:
1. No horizontal scroll (the page should not be wider than the screen)
2. Text is readable (not tiny, not overflowing)
3. Buttons are tappable (at least 44px tall — Apple's guideline)
4. Fixed elements (nav, input bars) are not cut off

Test on a real phone once before launching — DevTools is good but not perfect.

---

### "When should I use a bottom tab bar vs a hamburger menu on mobile?"

**Bottom tab bar** (icons at the bottom of the screen):
- When your app has 3–5 main sections users switch between frequently
- When the app is used one-handed (bottom is easier to reach with thumb)
- Best for: chat apps, reminder apps, daily-use mobile apps

**Hamburger menu** (top-left icon that reveals a sidebar):
- When your app has more than 5 sections or a deep navigation hierarchy
- When the app is primarily used on desktop with mobile as secondary
- Best for: content-heavy apps, dashboards

**Rule of thumb for AI products:** If users will open the app on their phone daily, use a bottom tab bar. If mobile is secondary to desktop, use a hamburger menu.

---

### "My layout looks fine on my laptop but breaks on my phone. Why?"

Three most common causes:

1. **Fixed pixel widths:** `w-[400px]` looks fine on a 1280px screen; overflows on a 375px screen. Fix: `w-full max-w-sm`.

2. **`100vh` vs `100dvh`:** On mobile Safari, `100vh` includes the hidden browser bar, causing layouts to be too tall and scrollable. Fix: use `100dvh`.

3. **Missing `overflow-x-hidden`:** One element slightly overflows the viewport, creating a horizontal scroll on the entire page. Fix: add `overflow-x-hidden` to the root layout.

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing Responsive Layouts

### Viewport Test Checklist
Test every new page or layout at these exact widths in Chrome DevTools:

| Width | Device Simulated | What to Check |
|---|---|---|
| 375px | iPhone SE / older phones | Tightest test — nothing should overflow or be too small |
| 390px | iPhone 14 | Most common iPhone size today |
| 430px | iPhone 14 Pro Max | Large phone — extra horizontal space |
| 768px | iPad portrait | Tablet breakpoint — ensure layout shifts correctly |
| 1024px | iPad landscape / small laptop | Desktop breakpoint — sidebar should appear |
| 1280px | Standard laptop | Full desktop layout |

### Quick Automated Check
```bash
# Install and run Lighthouse CI to catch responsive issues automatically
npm install -g @lhci/cli
lhci autorun --collect.url=http://localhost:3000

# Check the Lighthouse report for:
# - "Tap targets are not sized appropriately" (buttons too small)
# - "Content is not sized correctly for the viewport" (overflow issues)
# - "Text is not legible" (font sizes)
```

### Common Responsive Errors and Fixes

| Issue | Cause | Fix |
|---|---|---|
| Horizontal scroll on mobile | Fixed-width element wider than viewport | Replace `w-[Npx]` with `w-full max-w-[Npx]` |
| Content hidden behind keyboard on iOS | Input not sticky, layout uses `100vh` | Use `100dvh`; add `pb-[env(safe-area-inset-bottom)]` |
| Sidebar shows on mobile unexpectedly | Missing `hidden lg:flex` or wrong breakpoint | Check breakpoint: `hidden lg:block` for desktop-only sidebar |
| Bottom tab bar hidden behind home indicator | Missing safe area inset | Add `pb-safe` using tailwindcss-safe-area plugin |
| Text too small on mobile | Fixed `text-xs` | Use `text-sm md:text-base` |
| Buttons too small to tap on mobile | Fixed `h-8` | Use minimum `h-11` (44px) for any tappable element on mobile |

### Debugging Loop
```
Mobile layout issue →
  1. Open Chrome DevTools → device emulation → set to 375px
  2. Right-click the broken element → Inspect
  3. In the Styles panel, look for:
     - Fixed pixel widths (width: 400px)
     - Overflow: visible causing content to escape
     - Position: fixed elements that overlap content
  4. Try adding these classes one at a time to the broken element:
     - w-full (if it's overflowing)
     - overflow-hidden (if children are escaping)
     - flex-col (if a row layout is breaking)
  5. Once you know the fix, paste the element's code to AI agent:
     "This element breaks at 375px: [PASTE CODE]
      The issue is: [DESCRIBE]
      Fix using mobile-first Tailwind. No logic changes."
```

</testing_and_qa>

---

<common_patterns>

## Reusable Responsive Patterns

### Pattern 1: Responsive Sidebar App Shell
```typescript
// /app/(app)/layout.tsx
import { Sidebar } from "@/components/layout/sidebar";
import { MobileHeader } from "@/components/layout/mobile-header";

export default function AppLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-[100dvh] overflow-hidden bg-gray-50">
      {/* Sidebar: hidden on mobile, visible on desktop */}
      <Sidebar />

      {/* Main content area */}
      <div className="flex flex-1 flex-col overflow-hidden">
        {/* Mobile header with hamburger */}
        <MobileHeader className="lg:hidden" />

        {/* Scrollable content */}
        <main className="flex-1 overflow-y-auto px-4 py-6 md:px-6 lg:px-8">
          {children}
        </main>
      </div>
    </div>
  );
}
```

### Pattern 2: Responsive Sidebar Component
```typescript
// /components/layout/sidebar.tsx
"use client";
import { useUIStore } from "@/lib/store/ui-store";
import { cn } from "@/lib/utils";

export function Sidebar() {
  const { sidebarOpen, setSidebarOpen } = useUIStore();

  return (
    <>
      {/* Mobile overlay */}
      {sidebarOpen && (
        <div
          className="fixed inset-0 z-20 bg-black/50 lg:hidden"
          onClick={() => setSidebarOpen(false)}
        />
      )}

      {/* Sidebar panel */}
      <aside className={cn(
        // Base styles (mobile): fixed, off-screen, slides in
        "fixed inset-y-0 left-0 z-30 w-72 bg-white border-r flex flex-col",
        "transition-transform duration-200 ease-in-out",
        sidebarOpen ? "translate-x-0" : "-translate-x-full",
        // Desktop: always visible, static positioning
        "lg:relative lg:translate-x-0 lg:w-60 lg:z-auto"
      )}>
        <nav className="flex-1 overflow-y-auto p-4 space-y-1">
          {/* Navigation items */}
        </nav>
        <div className="p-4 border-t">
          {/* User profile */}
        </div>
      </aside>
    </>
  );
}
```

### Pattern 3: Responsive Card Grid
```typescript
// Works for lessons, reminders, any card-based list
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  {items.map(item => <ItemCard key={item.id} item={item} />)}
</div>
```

### Pattern 4: Full-Screen Chat Layout (Mobile-Safe)
```typescript
// /components/features/chat/chat-layout.tsx
export function ChatLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-[100dvh] flex-col">
      {/* Chat header */}
      <header className="flex-none border-b px-4 py-3">
        Chat Header
      </header>

      {/* Messages — independently scrollable */}
      <div className="flex-1 overflow-y-auto">
        {children}
      </div>

      {/* Input — sticks to bottom, above iOS home indicator */}
      <footer className="flex-none border-t p-3 pb-[env(safe-area-inset-bottom,12px)]">
        Input area here
      </footer>
    </div>
  );
}
```

### Pattern 5: Responsive Typography Scale
```typescript
// Use these classes as defaults across your app
// Heading: large on desktop, appropriately smaller on mobile
className="text-2xl font-bold md:text-3xl lg:text-4xl"

// Body: readable at all sizes
className="text-sm md:text-base leading-relaxed"

// Caption/helper text
className="text-xs md:text-sm text-gray-500"

// Button text (never smaller than text-sm on mobile)
className="text-sm font-medium"
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Never Rely on `display: none` Alone to Hide Sensitive Content
CSS `hidden` (display:none) can be overridden by browser extensions or DevTools. Never use it to hide sensitive data — hide it at the data level (do not fetch it, or check permissions in the API).

### Rule 2: Ensure Touch Target Sizes Meet Minimum Standards
Buttons smaller than 44×44px are both a UX failure and an accessibility issue (WCAG 2.5.5). For AI products used on mobile, this is especially important:
```typescript
// ✅ Minimum tappable size for mobile
className="h-11 min-w-[44px] px-4" // 44px = h-11 in Tailwind
```

### Rule 3: Test with `prefers-reduced-motion`
Some users have motion sensitivities. Always provide a no-animation fallback:
```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```
Or use Tailwind's `motion-safe:` and `motion-reduce:` prefixes.

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Using `100vh` for Full-Screen Mobile Layouts
On mobile browsers, `100vh` is taller than the visible area (it includes the browser bar). Elements get hidden behind it.  
**Fix:** Use `h-[100dvh]` (dynamic viewport height — supported by all modern browsers).

### ❌ Building Desktop Layout First, Then "Making It Responsive"
Retrofitting responsive behaviour is 3× harder than building mobile-first. The AI agent generates desktop-first code by default — you must prompt it explicitly to use mobile-first Tailwind.  
**Fix:** Always start your layout prompt with "build mobile-first using Tailwind responsive prefixes."

### ❌ Fixed Widths on Container Elements
```typescript
// ❌ Overflows on small screens
<div className="w-[600px]">

// ✅ Fluid with a maximum
<div className="w-full max-w-2xl">
```

### ❌ Forgetting the Keyboard on Mobile
When the virtual keyboard appears, it reduces the visible viewport by 30–50%. Layouts that use `100vh` or fixed heights will push the input off screen.  
**Fix:** Use `flex flex-col h-[100dvh]` with `flex-1 overflow-y-auto` on the scrollable area and `flex-none` on the input bar.

### ❌ Not Testing on a Real Device Before Launch
Emulators are approximations. Chrome DevTools does not replicate iOS Safari's rubber-band scrolling, safe area behaviour, or font rendering. Test on at least one real iPhone before any public launch.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Responsive Design

### Add Responsive Design System Tokens
Define your spacing, typography, and breakpoint decisions once in `tailwind.config.ts`:
```typescript
// tailwind.config.ts
module.exports = {
  theme: {
    extend: {
      screens: {
        xs: "375px", // iPhone SE — useful for smallest phone targeting
      },
      spacing: {
        "safe-bottom": "env(safe-area-inset-bottom)",
      },
    },
  },
};
```

### Add Container Queries (Beyond Breakpoints)
When a component needs to respond to its container width (not the screen width):
```typescript
// A card that changes layout based on how wide its parent is — not the page
// npm install @tailwindcss/container-queries
<div className="@container">
  <div className="flex flex-col @md:flex-row">
    Responds to container width, not viewport
  </div>
</div>
```
This is powerful for dashboard widgets that can appear in a narrow sidebar or a wide main area.

### PWA Support for Mobile Installation
Make your AI app installable as a home screen icon on iPhone and Android:
```
Ask your AI agent: "Add PWA support to my Next.js app.
Generate: manifest.json, icons at 192px and 512px, service worker for offline fallback.
Meta tags needed: theme-color, apple-mobile-web-app-capable, viewport."
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App — Mobile-First Chat Interface
**Problem:** The conversational reminder input was built on desktop. On iPhone, the input disappeared behind the keyboard.

```
Root cause: Layout used h-screen (100vh) + bottom padding.
When keyboard appeared, it reduced viewport but h-screen didn't adapt.

Fix applied:
1. Changed h-screen to h-[100dvh]
2. Changed flex layout: flex flex-col overflow-hidden on the container
3. Messages area: flex-1 overflow-y-auto
4. Input bar: flex-none + pb-[env(safe-area-inset-bottom,8px)]

Result: Input remained visible and above the keyboard on all iPhone models.
Testing showed this alone increased message completion rate on mobile by ~30%
in the first week after the fix.
```

### Case Study 2: EdTech App — Dashboard Grid Layout
**Requirement:** Lesson cards needed to display as a grid — as many as fit per row.

```
Solution: Responsive grid with 4 breakpoints
grid-cols-1       → mobile: single column (cards are wide, easy to read)
sm:grid-cols-2    → large phones/tablets: 2 columns (comfortable)
lg:grid-cols-3    → laptop: 3 columns (good density)
xl:grid-cols-4    → large desktop: 4 columns (maximum browsing)

Each card used:
- w-full (fills its grid cell)
- aspect-[4/3] for the thumbnail (consistent height regardless of column count)
- text-sm md:text-base for the title

The grid required zero JavaScript — pure CSS/Tailwind.
Layout worked correctly on first attempt across all viewports
because it was built mobile-first with explicit breakpoints.
```

</real_world_examples>

</skill_document>
