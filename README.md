# claude-skills
Open source skills I have been experimenting with in Claude Code

# claude-skills

Skills for designers and design engineers building with Claude Code.

---

## What is a skill?

A skill is a markdown file that tells Claude Code how to handle a specific task — what files to read, what rules to follow, what to produce. Drop a skill file into your project and Claude picks it up automatically when the task matches.

→ [How skills work in Claude Code](https://docs.anthropic.com/en/docs/claude-code/memory)

---

## Skills in this repo

**[a11y-skill.md](./a11y-skill.md)**
Runs a structured WCAG 2.1 AA accessibility audit on components or flows. Covers static code analysis, guided runtime checks, and a severity-ranked report with fixes. Works from Figma specs, component code, or both.

**[component-builder.md](./component-builder.md)**
Builds React components that consume tokens from `globals.css` and nothing else. Enforces a three-layer token architecture (primitive → semantic → Tailwind), handles state tokens via `data-*` attributes, and delegates story authoring to a story-builder skill.

**[zh-docs.md](./zh-docs.md)**
Generates paste-ready zeroheight documentation pages from component source files. Reads the component, stories, types, styles, and tests — then produces all 14 ZH sections with Storybook embed placeholders marked.

**[alignment-audit.md](./alignment-audit.md)**
Cross-audits a component's four artifacts — component file, spec sheet, story file, and ZH doc — for prop, token, variant, story convention, accessibility, and documentation alignment. Run occasionally to catch drift.

---

## Stack assumptions

These skills assume:

- **Next.js** with the App Router
- **Tailwind CSS** with a custom `@theme` block exposing design tokens
- **CSS custom properties** in `src/app/globals.css` as the token source of truth
- **Storybook** for component stories
- **zeroheight** for design system documentation
- **TypeScript**

If your stack differs, the skills still work but you'll need to update file paths and token references.

---

## How to install

1. In your project root, create a `.claude/skills/` directory if it doesn't exist
2. Copy whichever skill files you need into that directory
3. That's it — Claude Code reads them automatically

```
your-project/
└── .claude/
    └── skills/
        ├── a11y-skill.md
        ├── component-builder.md
        ├── zh-docs.md
        └── alignment-audit.md
```

---

## How the skills connect

These skills are designed to work together. `component-builder` delegates story authoring to a `story-builder` skill (not included here — bring your own). `zh-docs` delegates accessibility checks to `a11y-skill`. `alignment-audit` references both `a11y-skill` and the output conventions from all four artifacts.

You don't need all four to get value — each skill works independently.

---

## Configuration

`zh-docs` and `alignment-audit` use two shared variables. Set these when you first run either skill, or add them to your project's Claude instructions:

| Variable | What it does |
|---|---|
| `COMPONENT_NAMESPACE_PREFIX` | Code prefix to strip from public-facing component names (e.g. `Elegant`) |
| `STORYBOOK_TITLE_PREFIX` | Storybook title path preceding the component name (e.g. `Components/Forms`) |

---

## Contributing

These are living skills — pull requests welcome. If you've adapted them for a different stack or found a gap, open an issue or PR.
