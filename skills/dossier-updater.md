---
name: dossier-updater
description: Update team-members/Stakeholders/Projects/Goals dossiers with journal information
triggers: []
dependencies: [journal-processor]
version: 1.0.0
---

# Dossier Updater

Update team-members/Stakeholders/Projects/Goals dossiers with information extracted from journal entries.

## Vault Structure

```
Orraborous/
├── Journal/           # Daily entries
├── team-members/      # ONLY direct reports (team members)
├── Projects/          # Project dossiers
├── Stakeholders/      # Everyone else: peers, managers, cross-functional, external
├── Goals/             # Personal OKRs
├── Candidates/        # Interview summaries and scorecards
└── Templates/         # Obsidian templates
```

## People vs Stakeholders

**team-members/** is reserved for **direct reports only** (current team members). These dossiers track performance, potential, and development.

**Stakeholders/** is for **everyone else**:
- Your manager
- Peers
- Cross-functional partners
- HR/HRBP contacts
- External contacts (consultants, vendors)
- Anyone you interact with who doesn't report to you

Stakeholder dossiers use a strategic "User Manual" format to help build influence and achieve goals.

## Processing Steps

### Step 1: Receive Parsed Journal Data

Accept structured data from journal-processor with:
- Items by type (meetings, feedback, decisions, blockers, notes)
- Matched entities (existing dossiers)
- Unmatched entities (need to be created)

### Step 2: Handle New Entities

**IMPORTANT**: If an entity has no matching dossier, ASK THE USER before creating:

> "I found a reference to 'Sarah' but no dossier exists.
> Is Sarah a direct report (→ team-members/) or someone else (→ Stakeholders/)?"

Wait for confirmation. Options:
- "direct report" / "on my team" → Create in team-members/ with performance tracking
- "stakeholder" / "peer" / "manager" / etc. → Create in Stakeholders/ with strategic user manual format
- "skip" → Don't create, continue processing

### Step 3: Update Dossiers

Append entries to the appropriate section in each dossier:

**For People (direct reports)** → Add to `## Log` section:
```markdown
- **2026-01-19**: 1:1 - discussed Q1 goals, struggling with API project
```

**For Stakeholders** → Add to `## Interaction Log` section:
```markdown
- **2026-01-19**: Met to discuss Q1 priorities. She's focused on hitting ARR targets. Seemed stressed about headcount freeze.
```

Also look for opportunities to fill in [DATA GAP] fields:
- Motivations, KPIs, communication preferences
- Rapport hooks (hobbies, interests mentioned)

**For Projects** → Add to relevant section (`## Decisions`, `## Blockers`, `## Notes`):
```markdown
- **2026-01-19**: Might slip due to resource constraints #blocker
```

**For Goals** → Add to `## Progress Log`:
```markdown
- **2026-01-19**: Manager feedback - need to delegate more
```

## Data Gap Filling

When updating Stakeholder dossiers, look for information that fills [DATA GAP] placeholders:

| Field | Look For |
|-------|----------|
| Motivations | What drives them, goals mentioned, concerns expressed |
| KPIs | Metrics they care about, success criteria mentioned |
| Communication Style | How they prefer to communicate (email, meetings, direct, diplomatic) |
| Rapport Hooks | Hobbies, interests, personal topics mentioned |
| Current Focus | Projects or initiatives they're working on |

If found, replace [DATA GAP] with the discovered information.

## Output Summary

After processing, report:

```
Updated dossiers:
- [[team-members/Sarah]] - added 1:1 notes (direct report)
- [[Stakeholders/Mike]] - added interaction log

Created:
- [[Stakeholders/Jane]] - new stakeholder (peer)

Data gaps filled:
- [[Stakeholders/Mike]] - added rapport hook (loves hiking)
```

## Templates

When creating new dossiers, use:
- `Templates/Person.md` for direct reports
- `Templates/Stakeholder.md` for everyone else
- `Templates/Project.md` for projects
- `Templates/Goal.md` for goals

## Coordination with Other Skills

This skill should run AFTER journal-processor and can trigger:
- performance-ratings (for People updates with performance signals)
- relationship-tracker (for Stakeholder updates with relationship signals)
