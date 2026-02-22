---
name: action-tracker
description: Track and surface open action items across all journal entries
triggers:
  - "show my action items"
  - "open tasks"
  - "show overdue items"
  - "clear action item"
  - "mark as"
dependencies: []
version: 1.0.0
---

# Action Item Tracker

Track open action items across all journal entries and surface them with overdue detection.

## Scanning for Action Items

Scan all `Journal/*.md` files for action items:

| Pattern | Status | Description |
|---------|--------|-------------|
| `- [ ]` | Not Started | Open task, not yet begun |
| `- [~]` | In Progress | Task is being worked on |
| `- [!]` | Stuck | Task is blocked |
| `- [x]` | Done | Completed task |

**Open items** are those with `[ ]`, `[~]`, or `[!]` markers.
**Closed items** are those with `[x]` markers.

Preserve wiki-links in items for easy navigation.

When an item has a Monday ID (`<!-- monday:ID -->`), flag for monday-integration to sync status.

## Due Date Detection

Parse due dates from item text or context. Use the **journal entry date** as the baseline for relative references.

| Pattern | Interpretation |
|---------|----------------|
| "in a few days" | journal date + 3 days |
| "tomorrow" | journal date + 1 day |
| "by [weekday]" (e.g., "by Wednesday") | next occurrence of that weekday from journal date |
| "this week" | end of week containing journal date (Sunday) |
| "next week" | end of following week |
| "end of month" | last day of journal entry's month |
| Explicit date (e.g., "Jan 22", "2026-01-22") | that specific date |
| No time reference | no due date (but still tracked) |

## Overdue Logic

Compare parsed due date to **today's date**:
- **Overdue**: due date < today
- **Due soon**: due date is today or within next 3 days
- **No due date**: no time reference found

## Output Format

When showing action items (on-demand or after processing):

```
Open Action Items (6 total):

OVERDUE (2):
- [ ] Check with [[Jordan C]] about [[Sam S]]'s fit (from Jan 19, due ~Jan 22)
- [~] Follow up on Q1 strategy decision (from Jan 19, due ~Jan 22-23) [IN PROGRESS]

STUCK (1):
- [!] Talk with [[Alex H]] from consulting firm (from Jan 19) [STUCK]

DUE SOON (1):
- [ ] Schedule team offsite (from Jan 20, due Jan 22)

NO DUE DATE (3):
- [ ] Find out budget and confirm [[Casey S]] is onboard (from Jan 19)
- [~] Decide on pursuing Feature X (from Jan 19) [IN PROGRESS]
```

Status indicators: `[ ]` = Not Started, `[~]` = In Progress, `[!]` = Stuck

## Clearing Action Items

When user says "clear action item [text]":
1. Find the matching item in the source journal
2. Change `- [ ]`, `- [~]`, or `- [!]` to `- [x]`
3. If item has Monday ID, flag for monday-integration to update status
4. Confirm: "Marked as done in Journal/2026-01-19.md"

## Updating Action Item Status

User can also change status without completing:
- "mark [text] as in progress" → Change to `- [~]`
- "mark [text] as stuck" → Change to `- [!]`
- "mark [text] as not started" → Change to `- [ ]`

All status changes should be synced to Monday if the item has a Monday ID (coordinate with monday-integration skill).

## User Commands

| Command | Action |
|---------|--------|
| "show my action items" / "open tasks" | List all open action items with overdue flags |
| "show overdue items" | List only overdue action items |
| "clear action item [text]" | Mark an item as done in its source journal |
| "mark [text] as in progress" | Update status to in progress |
| "mark [text] as stuck" | Update status to stuck |
| "mark [text] as not started" | Update status to not started |

## Auto-Display After Journal Processing

When journal-processor completes, automatically show open action items to keep user accountable.
