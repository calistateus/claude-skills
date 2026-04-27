---
name: accessibility-audit
description: Run a structured WCAG 2.1 AA accessibility audit on UI components or flows. Use when a user shares component code, describes a UI, or asks for an a11y review. Works across design-engineering contexts — from Figma component specs to production React code.
---

## How this audit works

Claude runs four phases and produces a single severity-ranked report.

1. **Scope** — Claude identifies what to audit and maps component dependencies.
2. **Static analysis** — Claude reads code and flags structural issues.
3. **Runtime checks** — Claude guides the user through live tests where human eyes are needed.
4. **Report** — Findings ranked by severity with suggested fixes.

---

## Step 0 — Opening prompt

When the user asks for an accessibility audit, respond with:

> "To get started, share whichever of these you have:
> - Component files (`.tsx`, `.jsx`, `.css`, token files)
> - A description of the component or flow
> - A Figma link or screenshot
>
> Also tell me: is this a design review (pre-build) or a code review (built and live)?"

Use the answer to determine which phases apply:

| Context | Phases that apply |
|---|---|
| Design spec / Figma only | Scope → Static (structure + labelling + color) → Report |
| Code, not yet live | Scope → Static (all checks) → Report |
| Code, live in browser | Scope → Static → Runtime → Report |

Calibrate language to the user. Design engineers who share code get technical specifics (file, line number, ARIA attribute). Designers who share Figma or descriptions get plain-language findings and design-system recommendations.

---

## Step 1 — Scope and dependency mapping

From the files or description provided, identify:

- Which components are in scope
- Which shared primitives they depend on (tokens, error messages, layout helpers)
- Which flows they appear in (checkout, auth, form submission, etc.)

Produce a brief dependency summary:

```
High blast radius (used in 5+ places):
  Button → Modal, Form, Nav, CheckoutSummary
  Input  → LoginForm, SearchBar, CheckoutForm

Critical flows containing these components:
  CheckoutForm → Button, Input, Select, ErrorMessage
  LoginModal   → Button, Input, PasswordInput
```

**Blast radius rule:** Any failure in a high-blast-radius component that appears in a critical flow is automatically escalated to **Critical** before the audit begins. Flag this immediately.

---

## Step 2 — Static analysis

Evaluate each component against the checks below. For each failure, record:
- Component name
- File and line number (if code was shared)
- Issue description
- WCAG criterion
- Suggested fix

Mark each item: ✅ Pass · ❌ Fail · ⚠️ Needs human review

---

### A. Keyboard navigation

| Check | What to look for | WCAG |
|---|---|---|
| All interactive elements keyboard-reachable | No `div`/`span` with `onClick` missing `tabIndex` or `role` | 2.1.1 |
| Logical tab order | No non-sequential `tabIndex` values | 2.4.3 |
| Keyboard activation | Buttons respond to Space/Enter; links to Enter | 2.1.1 |
| No accidental focus traps | Modals trap intentionally; nothing else does | 2.1.2 |
| Arrow key nav in composites | Listboxes, tabs, radio groups support ArrowUp/Down | 2.1.1 |
| Escape closes overlays | Modals, dropdowns, drawers close on Escape | 2.1.1 |

### B. Labels and accessible names

| Check | What to look for | WCAG |
|---|---|---|
| Every input has a label | `<label htmlFor>`, `aria-label`, or `aria-labelledby` | 1.3.1, 4.1.2 |
| Icon-only buttons | `aria-label` on the button; `aria-hidden="true"` on the icon | 4.1.2 |
| Icon beside visible text | `aria-hidden="true"` on icon only; visible text is the name | 1.3.1 |
| Decorative icons/badges | `aria-hidden="true"` on the element | 1.1.1 |
| Groups labelled | `<fieldset>` + `<legend>` or `role="group"` + `aria-labelledby` | 1.3.1 |
| Toggle/switch named | `aria-label` or label in scope of `role="switch"` | 4.1.2 |
| Error linked to input | `aria-describedby` points to error message `id` | 3.3.1 |
| Visible label matches accessible name | `aria-label` must contain the visible label text | 2.5.3 |

### C. ARIA roles and states

| Check | What to look for | WCAG |
|---|---|---|
| Custom controls have correct role | Custom checkbox → `role="checkbox"` or native `<input>` | 4.1.2 |
| State kept in sync | `aria-checked`, `aria-selected`, `aria-expanded` match UI state | 4.1.2 |
| Tab panel wired | `role="tabpanel"` + `aria-labelledby` pointing to active tab `id` | 4.1.2 |
| Table headers scoped | `<th scope="col">` or `<th scope="row">` on all header cells | 1.3.1 |
| Sortable columns | `aria-sort` on `<th>`; updated on click; only one column active at a time | 1.3.1 |
| Disabled elements | Both HTML `disabled` and `aria-disabled="true"` set together | 4.1.2 |

### D. Live regions and dynamic content

| Check | What to look for | WCAG |
|---|---|---|
| Error messages announced | `role="alert"` or `aria-live` on error containers | 4.1.3 |
| Urgency correct | Info/success → `aria-live="polite"`; errors → `aria-live="assertive"` | 4.1.3 |
| Status updates announced | Loading states, toasts, no-results messages covered | 4.1.3 |
| Focus managed after actions | After modal closes, focus returns to the trigger | 2.4.3 |
| Live region in DOM before use | `aria-live` containers must exist before content is injected — not added at the same time | 4.1.3 |

### E. Color and visual

| Check | What to look for | WCAG |
|---|---|---|
| Text contrast ≥ 4.5:1 | Body text | 1.4.3 |
| Large text contrast ≥ 3:1 | 18px+ regular or 14px+ bold | 1.4.3 |
| Non-text contrast ≥ 3:1 | Focus indicators, UI icons, input borders | 1.4.11 |
| Focus indicator visible | `:focus-visible` style defined for all interactive elements | 2.4.7 |
| Information not color-only | Error states use icon + text, not red color alone | 1.4.1 |
| Decorative images/icons hidden | `alt=""` on decorative images; `aria-hidden="true"` on decorative SVGs | 1.1.1 |

### F. Motion

| Check | What to look for | WCAG |
|---|---|---|
| `prefers-reduced-motion` respected | `@media (prefers-reduced-motion: reduce)` in CSS | 2.3.3 |
| JS animations check preference | `window.matchMedia('(prefers-reduced-motion: reduce)')` before animating | 2.3.3 |

### G. Language

| Check | What to look for | WCAG |
|---|---|---|
| `lang` on `<html>` | `<html lang="en">` or appropriate locale | 3.1.1 |
| Foreign-language content | Inline text in another language has `lang` on its wrapper | 3.1.2 |

### H. Touch and pointer

| Check | What to look for | WCAG |
|---|---|---|
| Touch targets ≥ 44×44px | Use `min-height`/`min-width` or padding to reach 44px | 2.5.5 |
| Path-based gestures have alternatives | Swipe/drag must have a button or menu fallback | 2.5.1 |
| Destructive actions fire on pointer up | Use `onClick`, not `onMouseDown`, for irreversible actions | 2.5.2 |
| Hover/focus content dismissible | Tooltips: dismissible without moving pointer; persists until dismissed | 1.4.13 |

### I. Semantic structure

| Check | What to look for | WCAG |
|---|---|---|
| Skip link present | `<a href="#main-content">Skip to content</a>` visible on focus | 2.4.1 |
| Heading hierarchy correct | h1→h2→h3 in order; no skipped levels | 1.3.1, 2.4.6 |
| `<nav>` for navigation | Not a `<div>` | 1.3.1 |
| `<main>` with correct `id` | `id="main-content"` as skip link target | 2.4.1 |
| Lists use semantic elements | `<ul>` or `<ol>`, not divs | 1.3.1 |

---

## Step 3 — Component-type reference

Use this when evaluating specific component patterns.

**Buttons and links** — Native `<button>` for actions; `<a href>` for navigation. Icon-only: `aria-label` on button, `aria-hidden` on icon. Toggle: `aria-pressed`. Disclosure: `aria-expanded`. Always pair `disabled` + `aria-disabled="true"`.

**Form inputs** — `<label htmlFor>` or wrapping `<label>`. `aria-describedby` for descriptions and errors. `aria-invalid="true"` in error state.

**Checkbox** — Native `<input type="checkbox">` preferred. Custom: `role="checkbox"` + `aria-checked` (use `"mixed"` for indeterminate) + Space/Enter handlers.

**Radio group** — Native `<input type="radio">` with shared `name` (free arrow key nav). Wrap in `<fieldset>` + `<legend>`.

**Toggle/switch** — `role="switch"` + `aria-checked` + `aria-label`. Space activates.

**Dropdown/select** — Trigger: `aria-haspopup="listbox"` + `aria-expanded`. Listbox: `role="listbox"` + `aria-labelledby`. Options: `role="option"` + `aria-selected`. `aria-activedescendant` tracks keyboard cursor. Arrow keys navigate; Escape closes.

**Autocomplete** — Input: `role="combobox"` + `aria-autocomplete="list"` + `aria-expanded` + `aria-controls`. No-results: `aria-live="polite"` on a non-option element.

**Tabs** — `role="tablist"` + `aria-label` on strip. Each tab: `role="tab"` + `aria-selected` + `aria-controls`. Panel: `role="tabpanel"` + `aria-labelledby`. Arrow keys between tabs; Tab enters panel.

**Accordion** — Trigger: `<button>` + `aria-expanded` + `aria-controls`. Panel: `role="region"` + `aria-labelledby`.

**Modal** — `role="dialog"` + `aria-modal="true"` + `aria-labelledby`. Focus trap on open. Focus restored to trigger on close. Escape closes.

**Table** — `<th scope="col/row">`. Sortable columns: `aria-sort` on `<th>`, updated on click, one active column at a time. Complex headers: `id` + `headers` instead of `scope`.

**Alert/toast** — Immediate: `role="alert"` + `aria-live="assertive"`. Non-urgent: `aria-live="polite"`. Live region container must be in DOM *before* content is injected.

**Badge/indicator** — Text badges need no ARIA. Icon-only badges: `aria-label` on wrapper, `aria-hidden` on icon. Purely decorative: `aria-hidden="true"` on the whole element. White badge tokens only valid on dark backgrounds — flag for human review in all background contexts.

**Progress** — `role="progressbar"` + `aria-valuenow` + `aria-valuemin` + `aria-valuemax`. Use `aria-valuetext` for step-based or non-numeric values.

---

## Step 4 — Runtime checks

Present these one at a time only when the component is live in a browser. Wait for a response before moving to the next.

---

**R1 — Tab reachability**

> Click away from the component (try the browser address bar). Press **Tab** repeatedly.
>
> Can you reach every button, link, and input? Is there a visible highlight on each one as you land on it? Does the order feel logical?
>
> Tell me: can you reach all interactive elements, and is focus always visible?

---

**R2 — Keyboard activation**

> Tab to a button and press **Space** or **Enter**. Tab to a link and press **Enter**. If there's a dropdown, open it with Space/Enter, then close it with **Escape**.
>
> In a list of options, do arrow keys move between choices?
>
> Tell me: did everything respond correctly?

---

**R3 — Focus indicator**

> Tab through all interactive elements slowly.
>
> Every element should show a clearly visible outline, highlight, or underline when focused. If you ever Tab somewhere and can't see where focus is, that's a failure.
>
> Tell me: did every element show a clear focus state?

---

**R4 — Contrast**

> Right-click any text in the component → Inspect. In the Styles panel, click the color swatch next to the `color` property. A popup shows a contrast ratio (e.g. "4.8:1").
>
> Normal text needs **4.5:1**. Large text (18px+ or bold 14px+) needs **3:1**. Input borders and UI icons need **3:1**.
>
> Tell me: what ratios did you see, and what's the background color?

---

**R5 — 200% zoom**

> Press **Ctrl +** (Windows) or **Cmd +** (Mac) until the browser shows 200%.
>
> Is all text visible? Are buttons and inputs still usable? Does a horizontal scrollbar appear? (It shouldn't, except in data tables.)
>
> Tell me: does the component still work at 200%?

---

**R6 — Touch target size**

> In DevTools Elements panel, click an interactive element. In the Computed tab, check height and width.
>
> Every tappable area — including padding around the visible label — must be at least **44 × 44px**.
>
> Tell me: what are the pixel dimensions of your smallest interactive element?

---

**R7 — Reduced motion**

> Open DevTools → More tools → Rendering → "Emulate CSS media feature" → set `prefers-reduced-motion: reduce`.
>
> Do transitions and animations stop or become instant? Loading spinners may stay — decorative motion (slides, fades, bounces) should stop.
>
> Tell me: do animations stop when reduced motion is enabled?

---

**R8 — Screen reader announcement** *(optional but valuable)*

> In DevTools Elements panel, click an interactive element and open the **Accessibility** tab on the right. It shows the computed Name, Role, and State without needing a screen reader.
>
> Does the name make sense? ("Close dialog" not "X"). Does it show the correct role? For toggles and checkboxes, does it show "checked" or "unchecked"?
>
> Tell me: what name, role, and state does each interactive element show?

---

## Step 5 — Flow-level checks

Run this after the component audit when the user has identified a complete user journey (login, checkout, form submission, account settings).

### Error prevention

| Check | What to look for | WCAG |
|---|---|---|
| Reversible actions | Is there undo or cancel after destructive actions (delete, deactivate)? | 3.3.4 |
| Confirmation step | Do high-stakes submissions show a review screen before final action? | 3.3.4 |
| Data preserved on error | If submission fails, is the user's input retained? | 3.3.4 |
| Error recovery | Can users correct only the invalid field without losing the rest? | 3.3.1, 3.3.4 |

### Session and timing

| Check | What to look for | WCAG |
|---|---|---|
| Timeout warning | Warned with at least 20 seconds before session expires? | 2.2.1 |
| Extend option | Can users extend or disable the timeout? | 2.2.1 |
| Data preserved on timeout | Form data retained after re-login? | 2.2.1 |

### Focus management across views

| Check | What to look for | WCAG |
|---|---|---|
| Modal open | Focus moves into dialog — first interactive element or dialog container | 2.4.3 |
| Modal close | Focus returns to the element that opened it | 2.4.3 |
| Route change (SPA) | Focus moves to page heading or skip link after navigation | 2.4.3 |
| Toast/notification | Focus does *not* jump to toast; announced via live region only | 4.1.3 |

---

## Step 6 — Severity classification

### Impact levels

| Severity | Definition | Action |
|---|---|---|
| **Critical** | Blocks access entirely for a whole class of users | 🛑 Stop. Blocking issue. Human sign-off before merge. |
| **Serious** | Significantly degrades experience; workaround exists | ⚠️ Fix before next release. Flag to design + engineering lead. |
| **Moderate** | Noticeable gap; most users unaffected | 📋 Add to backlog with WCAG reference. |
| **Minor** | Best-practice improvement | 💡 Log as enhancement. |

### Blast radius modifier

| Pattern | Effect |
|---|---|
| Component in checkout, auth, or form submission | **Upgrade one severity level** |
| Component used in 5+ places | Note "high blast radius" in report |
| Component in a single low-traffic context | Note "low blast radius" |

### Escalate to a human reviewer when:

- A **Critical** issue is found in checkout, authentication, or form submission
- An accessible name depends on dynamic data (data-driven icon labels — every real-world value must be verified by a human)
- Color contrast depends on a background set by a parent component — flag for human review in all background contexts
- A custom ARIA pattern deviates from WAI-ARIA Authoring Practices in a non-obvious way
- The component wraps a third-party library whose internals cannot be inspected

---

## Step 7 — Report format

```
## Accessibility Audit — [Component or scope]
Audited: [date] · WCAG target: 2.1 AA

### Scope
- Components audited: [list]
- High blast radius: [list]
- Critical flows: [list]
- Runtime checks completed: [yes / no / partial — note which were skipped]

### Summary
X critical · X serious · X moderate · X minor · X require human review

---

### 🛑 Critical
| Component | File | Line | Issue | WCAG | Blast radius | Fix |

### ⚠️ Serious
| Component | File | Line | Issue | WCAG | Fix |

### 📋 Moderate
| Component | File | Line | Issue | WCAG | Fix |

### 💡 Minor
| Component | File | Line | Issue | WCAG | Fix |

---

### Flow-level findings
| Flow | Issue | WCAG | Severity |

---

### Human review required
| Component | Reason | What to check |

---

### Passes
[Components that fully passed, with a one-line note on what was checked]
```

After delivering the report:

> "Would you like me to fix any of these now? I'll start with Critical items, one component at a time, and confirm each change with you before moving on."