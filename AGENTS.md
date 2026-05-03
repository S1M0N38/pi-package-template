# Pi Package Template — Agent Context

## Project Overview

This is a **pi package template** — a starter kit for building [pi](https://pi.dev) packages. Packages bundle extensions, skills, prompt templates, and themes, and are distributed via npm or git.

Users fork this template to create their own pi packages. The sample code demonstrates all four resource types and should be replaced with real functionality.

**Tech Stack:** TypeScript (no build step — pi loads `.ts` directly via jiti), typebox for schemas, biome for lint/format.

### Structure

```
extensions/index.ts    # Extension entry point (tools, commands, events)
skills/hello/SKILL.md  # On-demand skill instructions
prompts/hello.md       # Slash-command prompt template
themes/template.json   # Theme with all 51 color tokens
package.json           # Pi manifest, peer deps, npm publish config
biome.json             # Linter/formatter config
tsconfig.json          # Type checking only (noEmit)
```

### Key Constraints

- **No build step** — pi loads `.ts` via jiti. Never add a build/compile step.
- **Peer dependencies** — `@mariozechner/pi-ai`, `@mariozechner/pi-coding-agent`, `@mariozechner/pi-tui`, `@mariozechner/pi-agent-core`, `typebox` are provided by pi at runtime. List them as `peerDependencies` with `"*"` range. Do not bundle them.
- **Conventional commits** — This project uses `feat:`, `fix:`, `docs:`, `chore:`, `ci:`, `refactor:` prefixes. Releases are automated via release-please.
- **2-space indentation** — Enforced by biome.
- **Themes require all 51 color tokens** — See `themes/template.json` for the full list.

---

## Development Commands

```bash
npm run typecheck      # TypeScript type checking (tsc --noEmit)
npm run lint           # Check lint + formatting (biome check)
npm run lint:fix       # Auto-fix lint + formatting issues (biome check --write)
npm run format         # Format code only (biome format --write)
```

### Testing the Package with pi

```bash
# Ephemeral test — loads package for current session only
pi -e .

# Test a specific tool
pi -e . -p "Use the hello tool to greet Alice"

# Test a slash command (interactive mode)
pi -e .
# Then type: /hello Alice

# Install locally for persistent testing
pi install .

# Verify what ships in the npm tarball
npm pack --dry-run
```

---

## Agentic Development Loop

Follow this iterative loop when developing the package:

### Step 1: Understand the Requirement

Clarify what the user needs:
- A new tool? command? event handler? skill? prompt template? theme?
- What should it do?
- Are there runtime dependencies (npm packages)?

Read the relevant pi docs before implementing:
- Extensions: `~/.local/share/npm/lib/node_modules/@mariozechner/pi-coding-agent/docs/extensions.md`
- Skills: `~/.local/share/npm/lib/node_modules/@mariozechner/pi-coding-agent/docs/skills.md`
- Themes: `~/.local/share/npm/lib/node_modules/@mariozechner/pi-coding-agent/docs/themes.md`
- Packages: `~/.local/share/npm/lib/node_modules/@mariozechner/pi-coding-agent/docs/packages.md`

### Step 2: Implement

Write the code. Key patterns:

**Extension** (`extensions/`):
```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";
import { Type } from "typebox";

export default function (pi: ExtensionAPI) {
  pi.registerTool({ ... });
  pi.registerCommand("name", { ... });
  pi.on("event_name", async (event, ctx) => { ... });
}
```

**Skill** (`skills/name/SKILL.md`):
```markdown
# Skill Name
Use this skill when the user asks about X.
## Steps
1. Do this
2. Then that
```

**Prompt template** (`prompts/name.md`):
```markdown
---
description: What this template does
---
Template content here.
```

**Theme** (`themes/name.json`): Must include all 51 color tokens. Copy `themes/template.json` as a starting point.

### Step 3: Verify

Run all checks after making changes:

```bash
npm run typecheck && npm run lint
```

If either fails, fix the issues before proceeding. Common fixes:
- Type errors: add missing types, fix imports
- Lint errors: run `npm run lint:fix`

### Step 4: Test with pi

```bash
pi -e . -p "Test prompt that exercises your new feature"
```

Verify:
- The extension loads without errors
- Tools return correct results
- Commands respond as expected
- No unexpected notifications or errors

### Step 5: Commit

Use conventional commit format:
```
feat: add new tool for X
fix: handle edge case in Y
docs: update README with Z
chore: update dependencies
ci: update workflow
refactor: simplify X
```

### Step 6: Iterate

Repeat steps 1-5 for each feature or fix. Keep commits small and focused.

---

## Release Flow

Releases are fully automated via CI/CD:

1. Push conventional commits to `main`
2. release-please opens a Release PR with updated `CHANGELOG.md` + version bump
3. Merge the Release PR → GitHub Release + `npm publish` happen automatically

Users update with: `pi update`

---

## Common Pitfalls

- **Forgetting `typebox` schemas** — Tool parameters must use `Type.Object()` from `typebox`, not raw TypeScript types
- **Importing from wrong package** — Use `import type { ExtensionAPI } from "@mariozechner/pi-coding-agent"` (type import)
- **Missing `export default function`** — Extensions must export a default factory function
- **Adding runtime deps as devDependencies** — Runtime npm packages go in `dependencies`, not `devDependencies`
- **Incomplete themes** — Pi requires all 51 color tokens; partial themes cause errors
- **Pinning peer deps** — Peer dependencies must use `"*"` range, not `"^0.70.0"` etc.
