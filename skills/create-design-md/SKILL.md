---
name: create-design-md
description: Generates a DESIGN.md file in the project root following Google's design.md specification. Supports three source modes — codebase exploration, Figma URL, or an existing DESIGN.md URL — and validates the output with the design.md linter.
argument-hint: "[figma-url | design-md-url]"
allowed-tools: Bash, Read, Write, Edit, Agent, AskUserQuestion, WebFetch, mcp__plugin_figma_figma__get_design_context, mcp__plugin_figma_figma__get_screenshot, mcp__plugin_figma_figma__get_metadata, mcp__plugin_figma_figma__get_variable_defs, mcp__plugin_figma_figma__search_design_system
---

# Generate DESIGN.md

Generate a `DESIGN.md` file in the project root following the [Google design.md specification](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md), then validate it with the linter.

**Output language:** Respond in the same language the user is using in this conversation.

## Step 1: Determine the source

First, check whether `DESIGN.md` already exists in the project root. If it does, ask the user with AskUserQuestion before proceeding:

- **Update it** — Regenerate `DESIGN.md` from a source of their choice (overwrites the existing file)
- **Keep it** — Abort and leave the existing file as-is

If the user chooses to keep it, stop here and report that the existing `DESIGN.md` was left unchanged.

If the user chooses to update (or if no `DESIGN.md` exists), evaluate `$ARGUMENTS` using the following rules:

- URL containing `figma.com` → **Figma mode**
- URL ending in `.md`, or containing `raw.githubusercontent.com` / `github.com` → **DESIGN.md URL mode**
- No argument → Ask the user with AskUserQuestion

When asking, present these options:

- **Explore codebase** — Extract design information from the current project's code and CSS
- **Figma URL** — Ask the user to provide a Figma design file URL
- **Existing DESIGN.md URL** — Fetch a publicly available DESIGN.md (e.g. a GitHub raw URL)

## Step 2: Collect design information

### DESIGN.md URL mode (Step 2A)

If a standard GitHub URL is provided, convert it to a raw URL:
- `github.com/<user>/<repo>/blob/<branch>/<path>` → `raw.githubusercontent.com/<user>/<repo>/<branch>/<path>`

Fetch the file with `WebFetch` and save it as `DESIGN.md` in the project root.  
Then skip directly to **Step 4 (lint validation)** — Step 3 is not needed.

### Codebase mode

Read the following files in order to gather design system information:

1. **Find the main CSS file** — run `grep -rl '@import.*tailwindcss' --include='*.css' .` to locate CSS files that import Tailwind. Read the first match (typically `src/index.css` or `src/app.css`). Extract color and spacing tokens from the `@theme` block.
2. `src/components/` — review component files to understand style patterns (buttons, cards, etc.)
3. `tailwind.config.*` or `vite.config.*` — check for additional configuration
4. `package.json` — get the project name

Collect:
- Project name and brand name
- Color palette (hex values)
- Typography (font family, size, weight)
- Spacing scale
- Border radius
- Component style patterns (buttons, etc.)
- Overall design mood and tone

### Figma mode

Extract `fileKey` and `nodeId` from the Figma URL (convert `node-id=1-2` → `1:2`).

Use the following tools to collect design information:
- `get_metadata` — file name and project overview
- `get_variable_defs` — Figma variables (color, typography, spacing tokens)
- `get_design_context` — detailed design context and component information

Collect the same information as codebase mode above.

## Step 3: Generate DESIGN.md

Generate `DESIGN.md` in the project root following the spec below.

### YAML frontmatter

```yaml
---
version: alpha
name: <project name>
description: <one-line project description>
colors:
  primary: "#XXXXXX"
  secondary: "#XXXXXX"
  # add tertiary, neutral, surface, etc. as needed
typography:
  h1:
    fontFamily: <font name>
    fontSize: <px>
    fontWeight: <number>
    lineHeight: <number>
    letterSpacing: <em>
  body-md:
    fontFamily: <font name>
    fontSize: <px>
    fontWeight: <number>
    lineHeight: <number>
  # add label-md, caption, etc. as needed
rounded:
  sm: <px>
  md: <px>
  lg: <px>
  full: 9999px
spacing:
  xs: <px>
  sm: <px>
  md: <px>
  lg: <px>
  xl: <px>
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "#FFFFFF"
    rounded: "{rounded.md}"
    padding: <px>
  button-primary-hover:
    backgroundColor: "{colors.secondary}"
  # add other components as needed
---
```

### Token rules

- **Color**: `#` + 6-digit hex (e.g. `"#1A1C1E"`) — always wrap in double quotes
- **Dimension**: number + unit (`px`, `em`, `rem`) — e.g. `48px`, `-0.02em`
- **Token Reference**: `{path.to.token}` — no quotes needed (e.g. `"{colors.primary}"`)
- **fontWeight**: number (e.g. `400`, `700`)
- **lineHeight**: number or Dimension (e.g. `1.6` or `24px`)
- `colors.primary` is required — omitting it triggers a `missing-primary` warning
- Ensure WCAG AA contrast ratio (4.5:1) for `backgroundColor` / `textColor` pairs in components

### Markdown body (section order is fixed)

```markdown
## Overview

[3–5 sentences on brand personality, target audience, and the impression the UI should convey]

## Colors

[Color palette description. Role and usage of each color as a bullet list]

- **Primary (#XXXXXX):** [description]
- **Secondary (#XXXXXX):** [description]

## Typography

[Typography strategy. Fonts used and the role of each level]

## Layout

[Layout and spacing strategy. Grid system and whitespace approach]

## Elevation & Depth

[How visual hierarchy is expressed. Shadows, tonal layers, etc.]

## Shapes

[Border radius and shape language]

## Components

[Style guidelines for key components]

## Do's and Don'ts

- Do [recommended practice]
- Don't [prohibited practice]
```

**Note:** Sections with insufficient information may be omitted, but any included sections must follow the order above.

## Step 4: Lint validation

After generating the file, run the linter. Replace `<pm>` with the package manager used in this project (`npx`, `pnpm dlx`, `yarn dlx`, etc.):

```bash
<pm> @google/design.md lint DESIGN.md
```

Review the results:

- **error** → fix `DESIGN.md` and re-run the linter (repeat until no errors remain)
- **warning** → review and fix where possible (contrast ratio issues, orphaned tokens, etc.)
- **info** → no action needed

### Common lint errors and fixes

| Error | Cause | Fix |
|---|---|---|
| `broken-ref` | Token reference `{colors.xxx}` does not exist | Check and correct the token name |
| `missing-primary` | `colors.primary` is not defined | Add `primary` to the `colors` section |
| `contrast-ratio` | Text/background contrast ratio is below 4.5:1 | Adjust to colors with greater contrast |
| `section-order` | Sections are not in the required order | Reorder to Overview → Colors → … |

## Step 5: Report completion

Report the following:
- Path to the generated `DESIGN.md`
- Lint result summary (number of errors / warnings / info)
- Overview of key design tokens (colors, typography)
