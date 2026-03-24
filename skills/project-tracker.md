---
name: project-tracker
description: Surface status, blockers, decisions, and open action items for any project in Projects/. Runs automatically after dossier-updater in the journal processing pipeline, and can be invoked on-demand for a specific project.
triggers:
  - "track project"
  - "project status"
  - "update project"
  - "project update"
  - "show projects"
dependencies: [dossier-updater]
auto_invoke: false
version: 1.0.0
---

# Project Tracker

Generic project tracking skill. Works for any file in `Projects/`. Surfaces stale fields, open items, and prompts targeted updates — interactively or as part of the journal processing pipeline.

## Two Modes

### Mode A: Pipeline (auto, after dossier-updater)

Runs silently after dossier-updater on every journal processing run. Does not show all projects — only surfaces projects with something worth flagging. If nothing is stale or notable, outputs nothing and passes control to action-tracker.

### Mode B: On-demand (triggered by user)

User says "track project [Name]" or "show projects" — runs interactively for one or all projects.

---

## Processing Steps

### Step 1: Load All Active Projects

Glob `Projects/*.md`. For each file, read frontmatter:
- `status`: skip if `completed`
- `owner`
- `started`
- `event_date` (optional — for deadline-driven projects)

Only process projects where `status: active` (or status is absent/blank — treat as active).

### Step 2: Extract Project State

For each active project, extract:

| Field | Source | Stale if... |
|-------|--------|-------------|
| Owner | frontmatter `owner` | blank or unassigned |
| Last note | most recent dated entry in `## Notes` | > 14 days ago |
| Open blockers | any entry in `## Blockers` that isn't struck through or marked resolved | any present |
| Pending decisions | any entry in `## Decisions` ending in `?` or containing "TBD", "pending", "need to decide" | any present |
| Event/deadline | frontmatter `event_date` | within 30 days |
| Journal mentions | scan last 7 days of journals for project name | 0 mentions = no recent activity |

### Step 3: Score Each Project

Assign flags:

| Flag | Condition |
|------|-----------|
| NO_OWNER | owner blank or "unassigned" |
| STALE | no journal mentions in last 7 days AND last note > 14 days ago |
| BLOCKER | open blocker entries present |
| DECISION_NEEDED | pending decision entries present |
| DEADLINE_NEAR | event_date within 30 days |
| NEW | started within last 14 days and has minimal content |

### Step 4: Output

#### Pipeline mode (after dossier-updater)

Only show projects with at least one flag. Format:

```
Projects needing attention:

[[Projects/Project Alpha]] — DEADLINE_NEAR (May 6), NO_OWNER
  Blockers: [System Name] connection needed; analyst allocation TBD
  Decisions: Goals of the day still TBD

[[Projects/Project Beta]] — BLOCKER, STALE
  Blockers: Data pipeline dependency — flag with [Stakeholder Name]
  Last activity: 2026-02-15 (37 days ago)

Update any of these, or continue to action items?
```

If user responds with updates, execute them (see Step 5) before passing to action-tracker.
If user says "continue" or "skip", pass directly to action-tracker.

#### On-demand mode (single project)

Show full project state:

```
[[Projects/Project Name]]
Status: active
Owner: [name or UNASSIGNED]
Started: [date]
Event date: [date or none]

Last activity: [date] ([N] days ago)
Journal mentions (last 7 days): [N]

Blockers:
- [blocker text] (logged [date])

Pending decisions:
- [decision text]

Recent notes:
- [last 2-3 notes]

Flags: [list]

What would you like to update?
```

#### On-demand mode (all projects)

Show a summary table then offer to drill into any:

```
Active Projects:

| Project | Owner | Last Activity | Flags |
|---------|-------|--------------|-------|
| [[Project Alpha]] | [YOUR_NAME] | 2026-03-23 | DEADLINE_NEAR |
| [[Project Beta]] | unassigned | 2026-01-26 | NO_OWNER, STALE |
| [[Project Gamma]] | unassigned | 2026-03-22 | NO_OWNER |
| [[Project Delta]] | [Team Member] | 2026-02-15 | BLOCKER, STALE |
| [[Project Epsilon]] | unassigned | 2026-03-22 | NO_OWNER, NEW |

Drill into a project? (name or number, or "done")
```

### Step 5: Execute Updates

When user provides an update, apply it to the project file:

| Update type | Action |
|-------------|--------|
| Assign owner | Update frontmatter `owner:` field |
| Add blocker | Append to `## Blockers` with date: `- **YYYY-MM-DD**: [text]` |
| Resolve blocker | Append resolution note: `- **YYYY-MM-DD**: [blocker] — resolved` |
| Add decision | Append to `## Decisions` with date |
| Add note | Append to `## Notes` with date |
| Change status | Update frontmatter `status:` field; if `completed`, add `completed: YYYY-MM-DD` |

After each update:
- Confirm: `✓ Updated Projects/[name].md`
- Continue to next flagged project or return to pipeline

### Step 6: Pass to Action-Tracker

After all flagged projects are handled (or skipped), output:

```
Project review done. Passing to action items...
```

Then invoke action-tracker as normal.

---

## User Commands (On-Demand)

| Command | Action |
|---------|--------|
| "track project [name]" | Single project deep dive |
| "show projects" / "project status" | Summary table of all active projects |
| "update project [name]" | Jump directly to update mode for a project |

---

## Edge Cases

### No active projects
```
No active projects found in Projects/. All clear.
```
Pass to action-tracker immediately.

### All projects clean (pipeline mode)
No output. Pass to action-tracker silently.

### Project file missing referenced in journal
If a journal entry mentions a project that has no file in `Projects/`:
```
Journal mentions "[Project Name]" but no project file exists.
Create one? (yes/no)
```
If yes, create from `Templates/Project.md` with today's date and add first note from journal context.

### Completed projects
Skip entirely — do not surface in pipeline or summary table.
