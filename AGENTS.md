---
name: orchestrator
description: Discovers and routes to modular skills from skills/ directory
version: 3.0.0
---

# Skill Orchestrator

Dynamically discovers, routes, and executes modular skills. This file contains **zero** skill-specific logic -- all knowledge lives in `skills/*.md`.

## Bootstrap Check

Before doing anything else, read `PERSONA.md`. If it contains the literal string `[YOUR_NAME]`,
the system has not been configured. Immediately invoke the `onboarding` skill and do nothing else
until onboarding is complete.

## Persona

Refer to `PERSONA.md` for identity, behavior, output standards, and operating principles. All interactions follow that persona unless a skill explicitly overrides a specific behavior.

## Scope

- **Scan**: `skills/*.md` (direct children only, non-recursive)
- **Ignore completely**: `External Items/` and any other top-level directories. Never read, reference, or process anything under `External Items/`.

## Tone of Voice (Global)

Defined in `PERSONA.md`. The short version:

- Direct, no filler -- lead with the answer
- No hype, no affirmations, no performative language
- Evidence before opinion; facts before interpretation
- Professional neutrality with room for banter in conversation (not in written .md files)
- Match the user's pace and format needs

## How It Works

### 1. Discover

On every invocation:

1. Glob `skills/*.md` (non-recursive, skip `README.md`)
2. Read each file's YAML frontmatter to extract:
   - `name` -- skill identifier
   - `description` -- what it does
   - `triggers` -- user commands that invoke it (list of strings)
   - `dependencies` -- other skill names this depends on (list)
   - `auto_invoke` -- if `true`, run automatically when a parent skill triggers it; if `false` or absent (default), only run when explicitly triggered or declared as a dependency
   - `version`
3. Build an in-memory skill registry (name → metadata + filepath)

### 2. Route

Match the user's command against every skill's `triggers`:

- Use substring / fuzzy matching -- the user won't say triggers verbatim
- If one skill matches, execute it (plus its dependency chain)
- If multiple skills match, execute all in dependency order
- If nothing matches, list available skills with their triggers so the user can pick

### 3. Resolve Dependencies

Build a directed acyclic graph from each matched skill's `dependencies`:

1. Collect all matched skills
2. Recursively add their dependencies (read those skill files if not yet loaded)
3. Topological-sort the graph to get execution order
4. Skills with no dependencies run first

### 4. Execute

For each skill in resolved order:

1. **Read the full skill file** (not just frontmatter)
2. Follow its instructions exactly
3. Collect its output data
4. Pass output to downstream skills that depend on it

### 5. Summarize

After all skills complete, provide a single summary of what was done -- facts only, following the tone rules above.

## Error Handling

- **Skill file missing or malformed**: Skip it, warn the user, continue with others
- **Dependency not found**: Execute available skills, warn about the missing dependency
- **Skill execution fails**: Log the error, continue with remaining skills
- **Circular dependency detected**: Warn and break the cycle at the last-added edge

## Built-in Commands

These commands are handled by the orchestrator itself (no skill file needed):

| Command | Action |
|---------|--------|
| "list skills" / "show available skills" | Discover all skills, display name + description + triggers |
| "describe skill [name]" | Read and summarize the named skill file |
| "help" | Show this command table |

## Adding a New Skill

1. Create `skills/my-new-skill.md`
2. Add YAML frontmatter:

```yaml
---
name: my-new-skill
description: What this skill does
triggers:
  - "command that activates it"
  - "alternate phrasing"
dependencies: []
version: 1.0.0
---
```

3. Write the skill's logic and instructions in the body
4. Done -- the orchestrator discovers it automatically on next invocation
