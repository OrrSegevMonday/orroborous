---
name: journal-processor
description: Find, parse, and structure daily journal entries into processable data
triggers:
  - "process my journal"
  - "process journal for"
  - "parse journal"
dependencies: [dossier-updater, action-tracker, calendar-follow-up, monday-integration]
version: 1.1.0
---

# Journal Processor

Core skill that finds and parses daily journal entries, extracting structured information for downstream processing.

## Quick Start

When invoked:
1. Find the journal entry (today or specified date)
2. Parse sections and bullet points
3. Extract entities (people, projects, stakeholders, goals)
4. Convert text to wiki-links
5. Return structured data for other skills

## Processing Pipeline

After journal-processor completes, dependent skills are invoked in order:
1. **dossier-updater** - Updates team-members/Stakeholders/Projects/Goals dossiers
2. **action-tracker** - Extracts and tracks action items
3. **calendar-follow-up** - Creates calendar events for follow-up action items
4. **monday-integration** - Creates Monday.com tasks from action items

## Step 1: Find the Journal Entry

Search for journal in this order:
1. `Journal/YYYY-MM-DD.md`
2. Root directory `YYYY-MM-DD.md`

If user specifies a date: "process journal for 2026-01-15"

### No Entry Found
> "I couldn't find a journal entry for today (2026-01-19).
> Would you like me to create one from the template?"

If yes, create from `Templates/Daily Journal.md`

## Step 2: Parse Entry into Items

Split the entry by section headers and bullet points. Identify:

| Type | Indicators |
|------|------------|
| Meeting | "met with", "1:1", "sync", "call with", under `## Meetings` |
| Feedback Given | "told X", "praised", "feedback to", under `## Feedback` > `### Gave` |
| Feedback Received | "X said", "got feedback", under `## Feedback` > `### Received` |
| Action Item | `- [ ]`, `- [~]`, `- [!]`, "need to", "should", "TODO", under `## Action Items` |
| Decision | "decided", "agreed", "will do X" |
| Blocker | "blocked", "waiting on", "stuck" |
| Note | Everything else |

## Step 3: Extract Entities

For each item, identify:

- **People**: Names (capitalized words that aren't common nouns) - potential direct reports
- **Projects**: Known project names or phrases like "Project X", "the X initiative"
- **Stakeholders**: Names of people who aren't direct reports (peers, managers, external contacts)
- **Goals**: References to personal objectives, OKRs

## Step 4: Match to Existing Dossiers

Search for matching files:
- `team-members/{name}.md` - **ONLY for direct reports**
- `Stakeholders/{name}.md` - **Everyone else** (peers, managers, cross-functional, external)
- `Projects/{project}.md`
- `Goals/{goal}.md`

Return both matched and unmatched entities for the dossier-updater skill to handle.

## Step 5: Convert to Wiki Links

Update the original journal entry to use `[[wiki-links]]`:
- `Sarah` → `[[Sarah]]`
- `Project Alpha` → `[[Project Alpha]]`
- `Acme Corp` → `[[Acme Corp]]`

Save the updated journal file.

## Output Data Structure

Return structured JSON for downstream skills:

```json
{
  "date": "2026-01-25",
  "filepath": "Journal/2026-01-25.md",
  "items": [
    {
      "type": "meeting",
      "text": "1:1 with [[Sarah]] - discussed Q1 goals",
      "entities": ["Sarah"],
      "section": "Meetings"
    },
    {
      "type": "action_item",
      "text": "Follow up with [[Mike]] by Wednesday",
      "entities": ["Mike"],
      "due_date": "2026-01-29",
      "status": "not_started",
      "monday_id": null
    }
  ],
  "entities": {
    "people": ["Sarah"],
    "stakeholders": ["Mike"],
    "projects": ["Project Alpha"],
    "goals": []
  },
  "matched_entities": {
    "people": ["Sarah"],
    "stakeholders": ["Mike"],
    "projects": ["Project Alpha"]
  },
  "unmatched_entities": {
    "people": [],
    "stakeholders": [],
    "projects": [],
    "goals": []
  },
  "sections": {
    "meetings": true,
    "feedback": true,
    "action_items": true,
    "interviews": false
  }
}
```

## Tags to Preserve

When parsing, preserve and categorize these tags:
- `#meeting` `#1on1` `#team-sync`
- `#feedback/given` `#feedback/received`
- `#decision` `#blocker`
- `#task`

## Edge Cases

### Empty sections
Skip sections that are empty or only contain template placeholders.

### Ambiguous entities
If unsure whether something is a person, project, or stakeholder, mark as ambiguous for user confirmation.

### Already linked
If text is already a `[[wiki-link]]`, don't double-link it.

### Interview sections
If `## Interviews` section exists, flag for interview-processor skill to handle.
