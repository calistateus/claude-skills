---
name: zeroheight-writer
description: Generates complete zeroheight component documentation pages from source files. Use when a user shares component source files and wants a paste-ready ZH documentation page.
---

## Goal

Produce a complete, paste-ready zeroheight documentation page for a design system component. The finished output should require only two things from the person: dropping in Storybook embeds at the marked placeholders, and a final review pass. Claude writes everything else.

A good ZH page is:
- **Specific** — every piece of guidance traces back to the actual component, not generic heuristics
- **Direct** — second-person, present tense, plain language ("Use this when…" not "This component can be used when…")
- **Complete** — every section is filled or explicitly marked for human follow-up

---

## Configuration

The team using this skill should set these values before running. If they're not set, Claude asks before proceeding.

| Variable | Example | Purpose |
|---|---|---|
| `COMPONENT_NAMESPACE_PREFIX` | `Elegant` | Code prefix to strip from all public-facing names |
| `STORYBOOK_TITLE_PREFIX` | `Components/Forms` | Storybook title path that precedes component name |

**Prefix rule:** Strip `COMPONENT_NAMESPACE_PREFIX` from every component name in the output — headings, body copy, prop descriptions, related components, and Storybook block paths. The prefix is a code namespace convention, not the public design system name.

---

## Step 0 — File request

When the user asks for ZH documentation, ask for these files before writing anything:

> "Share the component's source files and I'll produce the full ZH page. The more of these you can provide, the more complete the output:
>
> - **Component file** (`.tsx`, `.jsx`, `.vue`) — required
> - **Stories file** (`*.stories.tsx`) — primary source for variants and states
> - **Types or prop interface file** — if separate from the component
> - **Style file** (`.css`, `.module.css`, `.scss`) — for token usage and state styles
> - **Test file(s)** — reveals edge cases and accessibility intent
> - **Existing README** — any prior documentation to preserve or supersede
>
> If the stories file is missing, I'll derive variants from the component's props and types — but flag where human confirmation is needed."

### Fallback when stories file is absent

If no stories file is provided:
- Derive variants and states from TypeScript props, union types, and conditional logic in the component
- Mark every inferred variant with `[INFERRED — NEEDS CONFIRMATION]`
- Add a Reviewer note flagging that variant coverage may be incomplete
- Do not invent states that cannot be confirmed from the source

---

## Step 1 — Pre-write: build the component model

Before writing any section, read all provided files and build an internal model:

- **Public name** — component name with prefix stripped
- **Variants** — every exported story or prop union value
- **States** — every conditional class, state variable, or story decorator
- **Props** — full interface with types, defaults, required flags
- **Tokens** — every CSS custom property consumed in the style file
- **ARIA attributes** — every accessibility attribute used and what it maps to
- **Composition** — what other components appear in story args or test setups

Then produce a **Reviewer notes** block at the top of the document (see format below) before any content sections. This block is always present, even if there's nothing to flag.

---

## Output format

The full document is structured as ZH page sections in markdown. Storybook embed placeholders use this format:

`[STORYBOOK BLOCK: {STORYBOOK_TITLE_PREFIX}/{ComponentName}/{StoryName}]`

Use the exact story name as exported in the stories file. Inaccurate paths require manual lookup — get them right.

---

## Reviewer notes (always first)

```
## Reviewer notes

Files read: [list every file]
Files missing: [list expected files not provided]
Inferred content: [anything derived without direct source confirmation]
Needs human review: [props with unclear purpose, variants with no story, incomplete a11y attributes]
Recommended follow-ups: [missing stories, deprecated patterns, gaps to close]
```

---

## Sections

### 1. Overview

One sentence: what this component is and what problem it solves. Stop there.

---

### 2. When to use / When not to use

A two-column table. Every row must name the specific UI pattern and, in the Don't column, name the alternative component and why.

| Use | Don't use |
|---|---|
| [specific case] | [alternative + reason] |

Generic guidance ("use when you need X") is not acceptable. Derive every row from the component's design intent as visible in stories, tests, or prop names.

---

### 3. Anatomy

A numbered list of every named part of the component (container, label, icon, trailing element, badge, helper text, etc.) with one sentence describing its role.

Write this section even when there's no visual — it informs Figma annotation layers and handoff notes.

`[STORYBOOK BLOCK: {prefix}/{ComponentName}/Default]`

---

### 4. Variants

For every variant in the stories file (or inferred from props if no stories):

**{Variant name}**
- What it communicates visually and semantically
- When to choose it over other variants
- Any constraints (background requirements, content length, pairing rules)

`[STORYBOOK BLOCK: {prefix}/{ComponentName}/{VariantName}]`

Do not group variants unless their usage guidance is genuinely identical. Do not repeat prop definitions here — cross-reference the prop table.

---

### 5. States

For every interactive or conditional state (default, hover, focus, active, disabled, loading, error, success, empty, read-only):

**{State name}**
- What triggers it
- What changes visually
- Any behavior differences (keyboard, pointer, screen reader)

`[STORYBOOK BLOCK: {prefix}/{ComponentName}/{StateName}]`

Derive states from conditional classes in the style file and story decorators or args. Do not invent states.

---

### 6. Properties

| Prop | Type | Default | Required | Description |
|---|---|---|---|---|

For each prop:
- **Type**: actual TypeScript type; truncate long unions and note it (e.g. `'sm' | 'md' | 'lg' | …8 more`)
- **Description**: what it controls, not just what it is. "Controls visual weight and prominence" not "the variant."
- Flag deprecated, unstable, or props with non-obvious interactions with other props.

---

### 7. Content guidelines

**Skip this section if the component contains no human-authored copy** (icon-only, numeric, or purely structural components). If skipping, write one sentence saying why, then move on.

If the component does have copy, write only the rules that apply to this specific component. Derive character limits, casing conventions, and phrasing rules from the actual props, stories, and any existing README. Do not write generic writing advice.

Cover only what's relevant:
- **Label / primary text** — length, casing, tone, what to avoid
- **Supporting text** — relationship to label, when to include, length
- **Placeholder text** — what makes a good placeholder for this component specifically
- **Error messages** — format, sentence structure, ownership (content team vs. engineer fallback)
- **Truncation** — does the component handle it, or must copy be kept short by convention
- **Icon + text pairing** — when the icon is redundant, when it adds meaning, what's required for icon-only use

---

### 8. Accessibility

Run the checks from **a11y-skill.md** (static analysis only — no runtime checks at documentation time) and surface the findings here in documentation format. Do not reproduce the audit checklist. Write the results as documentation a designer or engineer would reference when building with this component.

Structure the findings under these headings. If a heading genuinely doesn't apply, say so explicitly — don't skip it silently.

**Keyboard navigation**
What a keyboard user can do with this component: tab behavior, activation keys (Enter, Space, Escape, Arrow keys, Home/End), focus trap behavior if applicable, focus restoration after dismiss.

**Screen reader behavior**
ARIA role and why it was chosen. Which props map to which ARIA attributes. What a screen reader announces on mount, on interaction, and on state change. Any `aria-live` regions and their politeness level.

**Color and contrast**
Whether all text and interactive states meet WCAG AA (4.5:1 text, 3:1 UI elements). Whether information is conveyed through color alone and what the non-color indicator is. Dark mode behavior if applicable.

**Motion**
Whether the component animates and whether it respects `prefers-reduced-motion`.

**Touch and pointer**
Minimum touch target size. Any pointer-specific behavior differences.

**Known gaps**
Be specific and honest. If something doesn't meet WCAG AA, document it and include the issue or ticket reference if one exists. A documented gap is better than a silent one.

---

### 9. Design tokens

| Token | Value | Where applied |
|---|---|---|
| `--token-name` | `#1A1A1A` | Label text |

Pull every CSS custom property from the style file. If the component consumes primitive tokens directly instead of semantic ones, flag each and suggest the correct semantic replacement.

---

### 10. Responsive behavior

How the component adapts across breakpoints. If it doesn't adapt, say so — that's useful to document.

- Props that control responsive behavior
- Breakpoint-specific layout changes
- Touch vs. pointer behavior differences (hover states, target sizes)

---

### 11. Composition and usage patterns

Common patterns where this component is combined with others. Derive these from story args and test file setups only — do not invent patterns.

For each pattern:
- Name it
- Describe when it's the right approach
- Call out gotchas: prop conflicts, z-index issues, spacing caveats, token collisions

`[STORYBOOK BLOCK: {prefix}/{ComponentName}/{CompositionStoryName}]`

If no composition patterns exist in the source files, write: `[No composition stories found — recommend adding examples for the most common usage patterns.]`

---

### 12. Related components

| Component | When to use it instead |
|---|---|
| [Name] | [Specific reason — not "similar functionality"] |

Honest guidance on when to reach for something else. If the component is genuinely unique with no near-alternatives, say so.

---

### 13. Do's and don'ts

Specific, visual rules — not generic advice. Each pair should describe something a designer could act on when building a screen.

**Do:** [specific action]
**Don't:** [specific failure mode and what to do instead]

Format each pair so the person building the ZH page can drop it directly into ZH's Do/Don't block type. Derive every rule from the component's actual constraints — don't write rules that apply to every component equally.

---

### 14. Changelog

| Version | Change | Breaking? |
|---|---|---|

Pull from git log or any existing CHANGELOG. If none exists: `[No changelog found — recommend tracking changes from this point forward.]`

---

## Quality rules

**Never invent behavior.** If something can't be confirmed from source files, write `[NEEDS CONFIRMATION]` inline and add it to Reviewer notes.

**Every section must be specific to this component.** If a sentence would be equally true for any component in any design system, rewrite it.

**Storybook block paths must be exact.** Use the `title` field from the stories file and the exported story name. No guessing.

**Accessibility findings come from a11y-skill.md.** Section 8 surfaces results — it is not a checklist to fill in manually.

**The Reviewer notes block is always present.** Even if there's nothing to flag, include it with "No issues found."