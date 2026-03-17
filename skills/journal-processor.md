---
name: journal-processor
description: Pull Notetaker meetings and Slack mentions, find/enrich daily journal entries, parse and structure them into processable data
triggers:
  - "process my journal"
  - "process journal for"
  - "parse journal"
dependencies: []
version: 2.1.0
---

# Journal Processor

Core skill that pulls meeting summaries from Monday Notetaker, finds and enriches daily journal entries, and extracts structured information for downstream processing.

## Quick Start

When invoked:
0. Pull Notetaker meetings and merge into journal (supplement or fallback)
0.5. Pull Slack mentions and merge into journal `## Slack` section
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

## Step 0: Pull Notetaker Meetings

Enrich the journal with meeting summaries from Monday Notetaker before parsing. Operates in two modes:
- **Supplement**: If a journal entry already exists, merge meeting summaries into its `## Meetings` section
- **Fallback**: If no journal entry exists, create one from meeting summaries

### 0a. Fetch today's meetings (metadata only)

Call `get_notetaker_meetings` via MCP (`user-monday-mcp`) with:
- `access: "OWN"` (only meetings the user attended)
- `include_summary: false` (metadata only -- fast, no payload)

If user specified a date, there is no date filter on the API, so fetch recent meetings and filter client-side by `start_time`.

If no meetings found, skip to Step 1.

### 0b. Present meeting list to user

Show each meeting as a selectable option:

```
Meetings found for YYYY-MM-DD:
1. Weekly Team Sync (09:00-09:45, 8 participants)
2. 1:1 with Direct Report (10:00-10:30, 2 participants)
3. Product All Hands (14:45-16:00, 57 participants)

Which meetings should I include? (comma-separated numbers, or "skip")
```

### 0c. User selects meetings

If user skips, proceed to Step 1 with no meeting enrichment.

### 0d. Fetch summaries for selected meetings

Call `get_notetaker_meetings` with:
- `ids: [array of selected meeting IDs]`
- `include_summary: true`

### 0e. Translate Hebrew to English

Scan each summary for Hebrew content. If any Hebrew is detected, translate the entire summary to English while preserving:
- Markdown formatting
- Proper nouns and names (keep original spelling)
- Technical terms

### 0f. Format and merge into journal

For each selected meeting, format a `###` subsection:

```markdown
### Meeting Title
*YYYY-MM-DD HH:MM-HH:MM | N participants | [link](meeting_link)* #meeting

Summary narrative from Notetaker...

**Decisions:**
- Decision bullet parsed from summary...

**Action Items:**
- [ ] Action item description -- *Owner Name*

**People mentioned:** [[Person A]], [[Person B]], [[Person C]]
```

Formatting rules:
- Parse "Decisions & Results" from the summary into `**Decisions:**` bullets
- Parse "Action Items & Owners" from the summary into `- [ ]` checkboxes with owner
- Extract all named people from the summary, render as `[[Name]]` wikilinks
- Add `#meeting` tag to the metadata line

#### Merge strategy

Check if `Journal/YYYY-MM-DD.md` exists:

**If the journal entry exists (supplement mode):**

For each meeting, scan the existing `## Meetings` section for a matching reference. Match by any of:
- **Title**: a `###` header contains the meeting title or a close substring
- **Participants**: the subsection body mentions names from the meeting summary
- **Time**: the subsection references a time overlapping the meeting window

If a match is found, **append** the Notetaker content below the user's existing notes:

```markdown
### Weekly Team Sync
User's handwritten notes about this meeting go here...

---
*From Notetaker:*

Notetaker summary narrative...

**Decisions:**
- ...

**Action Items:**
- [ ] ...

**People mentioned:** [[Person A]], [[Person B]]
```

If no match is found, **add** a new `###` subsection at the end of `## Meetings`.

**If no journal entry exists (fallback mode):**

Create a new `Journal/YYYY-MM-DD.md` from `Templates/Daily Journal.md` and populate the `## Meetings` section with all formatted meeting subsections.

## Step 0.5: Pull Slack Mentions

Enrich the journal with Slack messages where the user was mentioned or tagged. Runs after Notetaker pull (Step 0) so the journal file exists before we write to it.

### 0.5a. Search for mentions

Call `slack_search_public_and_private` via MCP (`user-slack`) with:
- `query`: `"to:me after:YYYY-MM-DD before:YYYY-MM-DD+1"` (journal date boundaries)
- `sort`: `"timestamp"`
- `sort_dir`: `"asc"` (chronological order)
- `limit`: `20`

`to:me` captures both @-mentions in channels and DMs sent directly to the user.

If no results, skip to Step 1 — leave `## Slack` empty.

If results exceed 20, paginate using `cursor` until exhausted.

### 0.5b. Enrich with thread context

For each search result that is part of a thread:

1. Call `slack_read_thread` with the result's `channel_id` and parent `message_ts` to get the full conversation
2. Keep the thread context for summarization — the mention alone is often not enough to understand intent

For standalone messages (not in a thread), use the message as-is.

### 0.5c. Resolve users to names

Collect all unique user IDs from the results. For each, call `slack_search_users` or `slack_read_user_profile` to get display names.

Map resolved names to existing vault entities:
- Check `team-members/*.md` filenames for direct reports
- Check `Stakeholders/*.md` filenames for everyone else
- If a match is found, use `[[wiki-link]]` format; otherwise use plain display name

Cache user ID → name mappings within the session to avoid repeated lookups.

### 0.5d. Format and merge into journal

Group results by channel/DM, then format into the `## Slack` section:

```markdown
## Slack
* **#channel-name** — [[Sender Name]]: summary of the message or thread #slack/mention
  * Key context or decision from the thread
  * Action items surfaced: `- [ ] description — *[[Owner]]*`
* **DM from [[Person]]** — topic summary #slack/dm
  * Key points from the conversation
```

Formatting rules:
- Summarize each thread/message into 1-2 bullet points (not raw transcript)
- Preserve decisions and action items explicitly
- Tag channel mentions with `#slack/mention`, DMs with `#slack/dm`
- Use `[[wiki-links]]` for all resolved people
- If a thread is long (>10 messages), summarize the arc rather than listing every message
- Translate Hebrew content to English (same rule as Step 0e for Notetaker)

#### Merge strategy

If `## Slack` section exists in the journal and already has content, **append** new entries below existing ones (avoid duplicates by checking if the channel + timestamp combo is already present).

If the section is empty (template default), replace it with the formatted content.

## Step 1: Find the Journal Entry

Search for journal in this order:
1. `Journal/YYYY-MM-DD.md`
2. Root directory `YYYY-MM-DD.md`

If user specifies a date: "process journal for 2026-01-15"

### No Entry Found

If Step 0 already created the entry from meetings, it will be found here.

Otherwise:
> "I couldn't find a journal entry for today (YYYY-MM-DD) and no meetings were pulled.
> Would you like me to create one from the template?"

If yes, create from `Templates/Daily Journal.md`

## Step 2: Parse Entry into Items

Split the entry by section headers and bullet points. Identify:

| Type | Indicators |
|------|------------|
| Meeting | "met with", "1:1", "sync", "call with", under `## Meetings` |
| Slack Mention | Channel mention or thread under `## Slack`, tagged `#slack/mention` |
| Slack DM | Direct message under `## Slack`, tagged `#slack/dm` |
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
    "slack": true,
    "feedback": true,
    "action_items": true,
    "interviews": false
  }
}
```

## Tags to Preserve

When parsing, preserve and categorize these tags:
- `#meeting` `#1on1` `#team-sync`
- `#slack/mention` `#slack/dm`
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
