---
name: component-builder
description: Generates React components strictly using CSS variables defined in globals.css. Use when a user asks to build or update a design system component.
---

## Role

Build design system components that consume tokens from `globals.css` and nothing else. Token discipline is the primary constraint — correctness before cleverness.

---

## Required reading

Before writing any code, read these files in order:

1. `src/app/globals.css` — the single source of truth for all tokens. Derive every available token from this file. Do not rely on memory or prior context.
2. `.claude/specs/Elegant{ComponentName}Spec.md` — if it exists, read it before writing anything.

---

## Spec file rules

- Spec file wins over your own interpretation when there is a conflict.
- Spec file wins over the user's casual description when there is a conflict.
- If the spec contradicts `globals.css` tokens, stop and flag it — do not guess.
- If no spec exists, proceed from the user's description and `globals.css`.

---

## File location and naming

| File | Path |
|---|---|
| Component | `src/components/{ComponentName}.tsx` |
| Story | `src/stories/{component-name}.stories.tsx` |

Component names are PascalCase. Story filenames are kebab-case. Export is always named — `export function Button()` not `export default`.

---

## Token architecture

Tokens in `globals.css` are structured in three layers:

1. **Primitives** (`--primitive-*`) — raw values. Never reference in components.
2. **Semantic** (`--color-*`, `--size-*`, `--font-*`) — intent-named aliases. Always use these.
3. **Tailwind `@theme`** — exposes semantic tokens as utility classes (`text-accent`, `bg-primary`).

**Use Tailwind utilities by default.** Use inline styles only when no utility class exists for the token. Never mix both approaches in the same component without a clear reason.

**Undefined token = stop.** If a component needs a style that no semantic token covers, stop and ask what token name and value to add to `globals.css`. Do not use hardcoded values or primitive tokens as a workaround.

---

## Token usage rules

**Colors and sizing** — use semantic tokens only: `--color-*`, `--size-*`.

**Typography** — use semantic font tokens: `--font-heading`, `--font-body`, `--font-mono`, `--font-size-*`. These are aliases defined in `globals.css`. Do not reference `--primitive-font-*` directly in components.

**States** — all state styles (hover, focus, disabled, error) are defined as semantic tokens in `globals.css`. Use Tailwind pseudo-class variants for simple states:

```tsx
className="bg-interactive-primary-bg hover:bg-interactive-primary-bg-hover 
           disabled:bg-interactive-primary-bg-disabled disabled:text-disabled"
```

For complex states (error, loading, selected), use `data-*` attributes and their corresponding CSS in `globals.css`:

```tsx
<input data-state="error" />
```

```css
/* in globals.css */
[data-state="error"] {
  border-color: var(--color-border-error);
}
```

---

## `use client`

Add `'use client'` at the top of the file when the component uses React hooks (`useState`, `useEffect`, `useRef`, etc.) or browser event handlers (`onClick`, `onChange`, etc.). Omit it otherwise — server components are the default in Next.js.

---

## Example

```tsx
'use client';

export function Button({ label, onClick }: { label: string; onClick: () => void }) {
  return (
    <button
      className="bg-interactive-primary-bg text-interactive-primary-fg 
                 hover:bg-interactive-primary-bg-hover
                 disabled:bg-interactive-primary-bg-disabled disabled:text-disabled
                 rounded-btn px-btn-px py-btn-py font-body text-sm"
      onClick={onClick}
    >
      {label}
    </button>
  );
}
```

---

## Storybook

After every component build or fix, invoke the **story-builder** skill. It owns all story authoring conventions. Do not write story files without it.

Once the story file exists:
1. Run `npm run storybook` if not already running.
2. Report the story path and variant names that were registered.

---

## Motion

If motion is requested, invoke the **motion** skill before writing any animation code.