# create-design-md — DESIGN.md Generator Skill

A Claude Code skill that generates a `DESIGN.md` file in the project root following the [Google design.md specification](https://github.com/google-labs-code/design.md). Supports three source modes — codebase exploration, Figma URL, or an existing DESIGN.md URL — and validates the output with the design.md linter.

---

## What this is

`DESIGN.md` is a machine-readable design specification that lives alongside your code. It encodes design tokens (colors, typography, spacing, border radius) and component guidelines in a structured YAML frontmatter + Markdown body format, making them accessible to AI coding agents without requiring a Figma file open.

This skill automates the creation of that file. Point it at your codebase, a Figma design, or an existing `DESIGN.md` from another project, and it generates a spec-compliant file, then runs the linter to catch any issues automatically.

```
Codebase / Figma URL / DESIGN.md URL  →  create-design-md  →  DESIGN.md (lint-validated)
```

---

## The 3 source modes

| Mode | When to use |
|------|-------------|
| **Explore codebase** | The project already has CSS tokens (`@theme` block) and components you want to document |
| **Figma URL** | You have a Figma design file with Variables defined |
| **Existing DESIGN.md URL** | You want to adopt a published `DESIGN.md` from another project (e.g. a GitHub raw URL) |

---

## Repo structure

```
skills/
  create-design-md/
    SKILL.md          — skill definition loaded by Claude Code
```

---

## Getting started

**1. Clone this repo**

```bash
git clone https://github.com/gaspanik/create-design-md-skill
```

**2. Install the skill into Claude Code**

Copy the skill directory into your Claude Code skills folder:

```bash
cp -r skills/create-design-md ~/.claude/skills/
```

**3. Run the skill**

Invoke with a slash command or natural language — both work:

```
/create-design-md
```

```
/create-design-md https://www.figma.com/design/<fileKey>/...?node-id=1-2
```

```
/create-design-md https://raw.githubusercontent.com/<user>/<repo>/main/DESIGN.md
```

```
Generate a DESIGN.md from the current codebase.
```

If no argument is provided, the skill asks you to choose a source mode interactively.

The report is written in whatever language you are using in the conversation — no configuration needed.

---

## Tips

- **Figma mode works best with Variables defined.** Files with color, typography, and spacing Variables produce richer token output. The more structured the Figma file, the better the `DESIGN.md`.
- **After generating, Claude reads it automatically.** Add the following to your `CLAUDE.md` so Claude consults `DESIGN.md` before any UI implementation:

  ```markdown
  ## Design System

  If `DESIGN.md` exists in the project root, read it before implementing any UI changes.
  It contains the design tokens (colors, typography, spacing, border radius) and component
  guidelines for this project.
  ```

- **Lint runs automatically.** Any errors are fixed before the skill completes. Warnings are reviewed and resolved where possible.

---

## Agent settings

- The skill requires `pnpm` to run the linter (`pnpm dlx @google/design.md lint`). If your project uses a different package manager, run the linter manually: `npx @google/design.md lint DESIGN.md`.
- Figma mode requires the official [Figma MCP server](https://github.com/figma/mcp-server-guide) to be connected in Claude Code settings.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `broken-ref` lint error | A component references a token that doesn't exist. The skill auto-fixes this, but if it recurs, check the token names in the frontmatter. |
| `missing-primary` lint warning | `colors.primary` is not defined. Add it to the `colors` section. |
| Figma mode returns no variables | The Figma file has no Variables defined. Use codebase mode instead, or define Variables in Figma first. |
| Linter not found | Ensure `pnpm` is available in the project, or run `npx @google/design.md lint DESIGN.md` manually. |

---

Built by Masaaki Komori - [@cipher](https://x.com/cipher) · Skill for [Claude Code](https://claude.ai/code)
