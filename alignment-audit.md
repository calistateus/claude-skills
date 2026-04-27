---
name: alignment-audit
description: Cross-audits a component's .tsx, .stories.tsx, spec sheet, and ZH doc for prop, token, variant, story convention, accessibility, and documentation alignment. Run occasionally to catch drift across artifacts.
---

## Role

Act as a design system quality engineer. For each component in scope, read its four artifacts and verify they describe the same component. Produce a severity-ranked report of every misalignment found.

---

## Configuration

Uses the same variables as component-builder and zeroheight-writer. If not already set, ask before proceeding.

| Variable | Example | Purpose |
|---|---|---|
| `COMPONENT_NAMESPACE_PREFIX` | `Elegant` | Code prefix used in filenames and interfaces |
| `STORYBOOK_TITLE_PREFIX` | `Components/Forms` | Storybook title path preceding the component name |

---

## Step 0 — Opening prompt

> "I'll audit alignment across the component, spec, story, and ZH doc for each component you name.
>
> Which components should I audit? Name them, or say 'all' for the full library."

---

## Step 1 — Resolve artifact paths

For each component in scope, resolve all four artifact paths before reading anything.

| Artifact | Path |
|---|---|
| Component | `src/components/{PREFIX}{Name}.tsx` |
| Spec | `.claude/specs/{PREFIX}{Name}.md` |
| Story | `src/stories/{PREFIX}{Name}.stories.tsx` |
| ZH doc | `docs/{kebab-name}-zh.md` (e.g. `ElegantButtonGroup` → `button-group-zh.md`) |

Note missing artifacts before running any checks — a missing artifact is itself a finding.

Then read:
- All four artifacts for each component in scope
- `src/app/globals.css` — read once, reuse across all components to validate token existence

---

## Step 2 — Alignment checklist

For each component, evaluate every check below. Mark each:

- ✅ Pass
- ❌ Fail — record which artifact(s), what's wrong, line number if available
- ⚠️ Needs human review — record why it can't be resolved automatically
- ➖ N/A — artifact missing, record which

---

### A. Props alignment

| Check | How to verify |
|---|---|
| Every prop in the component interface is in the spec's Props table | Compare `interface {PREFIX}{Name}Props` fields to spec `## Props` rows |
| Every prop in the spec is present in the component interface | Reverse of above |
| Story `argTypes` covers all user-facing props | Props not in `argTypes` and not hidden via `table: { disable: true }` are a gap |
| ZH doc prop table matches component interface | Column types, defaults, and required flags match |
| No invented props | Fail if a story `arg` or ZH table row names a prop not in the component interface |

---

### B. Token alignment

Extract every CSS custom property referenced in the component (any `var(--foo)` usage).

| Check | How to verify |
|---|---|
| Every token in the component exists in `globals.css` | Fail if not found |
| Spec `## Tokens used` list matches tokens actually used in the component | Missing from spec = spec drift; extra in spec = stale spec |
| No primitive token used where a semantic equivalent exists | Flag any `var(--primitive-*)` used for color, spacing, or radius if a semantic token covers the same intent |
| ZH doc `## 9. Design tokens` section lists the same tokens as used in the component | Fail if missing, truncated, or references tokens not in the component |

**Primitive vs semantic rule:**

❌ `var(--primitive-gray-900)` for body text → should be `var(--color-text-body)`
❌ `var(--primitive-black)` for interactive background → should be `var(--color-interactive-primary-bg)`
✅ `var(--primitive-font-sans)` for font family — no semantic equivalent exists, acceptable
✅ `var(--primitive-white)` in a hover inversion state — flag as ⚠️ needs human review to confirm intent

---

### C. Variants alignment

| Check | How to verify |
|---|---|
| Every variant in the spec has a corresponding exported story | Fail if a spec variant has no story coverage |
| No stories export variants not in the spec | Flag as stale or accidental |
| ZH doc `## 4. Variants` documents each variant from the spec | Fail if a spec variant is absent |
| ZH doc does not document variants absent from the spec | Flag as drift |
| Story variant names match ZH doc variant names | Check content, not label format — PascalCase vs sentence case is acceptable |

---

### D. Story convention compliance

Audit `argTypes` against these rules from the story-builder skill:

| Check | Rule |
|---|---|
| Image props use file upload control | `control: { type: 'file', accept: '...' }` — never `control: 'text'` |
| Icon props (`LucideIcon` type) use select + mapping | `options: Object.keys(iconOptions), mapping: iconOptions, control: { type: 'select' }` |
| String display props use text control | `control: 'text'` required |
| Boolean props use boolean control | `control: 'boolean'` required |
| String union / enum props use select control | `control: { type: 'select' }` with union values as options |
| `string \| false` props split into text + boolean pair | Must use flat `Args` type + `render` function |
| No `gt`, `gte`, `lt`, `lte` in `if` conditions | Use `truthy`, `exists`, `eq`, `neq` only — others are silently ignored |
| No convenience stories | No `AllVariants`, `Playground`, `Overview` — spec variants only |
| Implementation props hidden | `onClick`, `className`, `ref` hidden via `table: { disable: true }` |
| Backgrounds disabled | `backgrounds: { disable: true }` in `parameters` on all stories |
| Storybook block paths match ZH doc placeholders | Format must be `{STORYBOOK_TITLE_PREFIX}/{ComponentName}/{StoryName}` |

---

### E. Accessibility alignment

Run static analysis from **a11y-skill.md** on the component file and cross-reference findings against the ZH doc's accessibility section.

| Check | How to verify |
|---|---|
| ARIA attributes used in component are documented in ZH doc | If component sets `aria-expanded`, ZH doc must explain when and why |
| ZH doc doesn't claim ARIA attributes the component doesn't implement | Fail if ZH doc says "uses `aria-describedby`" but component doesn't |
| Interactive elements have accessible names | `<button>`, `<input>`, `<a>` must have label, `aria-label`, or `aria-labelledby` |
| `aria-disabled` present alongside HTML `disabled` | Both must be set together |
| `prefers-reduced-motion` respected | `@media (prefers-reduced-motion: reduce)` or JS `matchMedia` check |
| Keyboard nav documented in ZH doc matches component implementation | Cross-reference `onKeyDown`/`onKeyUp` handlers with ZH keyboard nav section |
| Focus-visible style present or gap documented | If no `:focus-visible` style, ZH doc must note it as a known gap |

---

### F. ZH doc completeness

Verify all required sections exist with real, component-specific content — not placeholders.

| Section | Check |
|---|---|
| Reviewer notes | Present at top |
| 1. Overview | One sentence, specific to this component |
| 2. When to use / When not to use | Rows are specific to this component, not generic |
| 3. Anatomy | Every named part in the component is listed |
| 4. Variants | One entry per spec variant |
| 5. States | Covers every state implemented in the component |
| 6. Properties | Matches component interface exactly |
| 7. Content guidelines | Present, or explicitly skipped with reason |
| 8. Accessibility | Keyboard nav, screen reader, contrast, motion, touch sections present |
| 9. Design tokens | Every component token listed |
| 10. Responsive behavior | Present — "does not adapt" is acceptable if accurate |
| 11. Composition patterns | At least one pattern present |
| 12. Related components | Present with specific guidance |
| 13. Do's and don'ts | Rules are specific to this component |
| 14. Changelog | Entry or acknowledged-empty placeholder |
| Storybook block paths | Format matches `{STORYBOOK_TITLE_PREFIX}/{ComponentName}/{StoryName}` |
| No unresolved `[NEEDS CONFIRMATION]` | Warn for every unresolved placeholder |

---

### G. Spec completeness

| Check | How to verify |
|---|---|
| `## Summary` present | Required |
| `## Props` table present | Required; must match component interface |
| `## Variants` section present | Required if component has a variant prop |
| `## Tokens used` list present | Required |
| `## Behaviour` section present | Required if component has interaction or animation |
| Every token in `## Tokens used` is actually used in the component | Fail if stale |
| No tokens used in component are absent from `## Tokens used` | Fail if spec is behind component |

---

## Step 3 — Severity classification

| Severity | Definition |
|---|---|
| **Critical** | A consumer using the component as documented will get broken behavior or inaccessible output — e.g. a story arg references a prop that doesn't exist; a token referenced in the component isn't in `globals.css` |
| **Serious** | Documented behavior doesn't match implementation, or a required ZH doc section is missing — misleads readers and design |
| **Moderate** | Drift between artifacts that doesn't break usage but causes confusion — e.g. stale token in spec, extra story variant not in spec |
| **Minor** | Convention violation or cosmetic gap — e.g. wrong control type for an arg |

### Escalate to a human when:

- A primitive token is used in a hover or inversion state that may be intentional design intent — can't auto-determine
- An ARIA attribute's value is dynamic data that can't be statically verified
- A ZH doc accessibility claim depends on runtime behavior (e.g. focus trap)
- A component wraps a third-party library whose internals can't be inspected
- The component has changed but the spec has not — unclear which is correct

---

## Step 4 — Report format

```
## Alignment Audit — [Component or scope]
Date: [today's date]

### Summary
X components audited
X critical · X serious · X moderate · X minor
X require human review · X artifacts missing

---

### Per-component results

#### {ComponentName}
Artifacts: Component ✅ · Spec ✅ · Story ✅ · ZH doc ❌ missing

| Severity | Category | Artifact | Issue | Fix |
|---|---|---|---|---|
| 🛑 Critical | Props | Story | `imagePath` uses `control: 'text'` — must be file upload | Change to `control: { type: 'file', accept: '...' }` |
| ⚠️ Serious | Tokens | Component | `var(--primitive-gray-900)` used for text — `--color-text-body` exists | Replace in component, update spec and ZH doc |

Human review needed:
- [Specific finding] — [Why it can't be resolved automatically]

---

### Passes
- [ComponentName] — all checks green
```

---

## Step 5 — Fixing

After the report, ask: **"Would you like me to fix any of these now?"**

If yes, follow this order:

1. Fix **Critical** issues first, then **Serious**, one component at a time
2. **Token issue:** update component → update spec `## Tokens used` → update ZH doc tokens table
3. **Props drift:** update spec first (spec is source of truth) → align component → align story → align ZH doc
4. **Story convention violation:** update story file only
5. **ZH doc gap:** update ZH doc only
6. Confirm each component is done before moving to the next
