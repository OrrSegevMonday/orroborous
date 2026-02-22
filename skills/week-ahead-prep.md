---
name: week-ahead-prep
description: Strategic week planning with calendar, team, stakeholder, and priority insights
triggers:
  - "prep for the week"
  - "week ahead"
  - "plan my week"
  - "sunday prep"
dependencies: [action-tracker, dossier-updater, calendar-follow-up]
version: 1.1.0
---

# Week-Ahead Prep

Strategic intelligence for the upcoming week across all dimensions: calendar, commitments, team, stakeholders, priorities, and risks.

## Quick Start

When you say "prep for the week" or "sunday prep", this skill provides a comprehensive week-ahead briefing.

## Data Sources

This skill scans and synthesizes information from:

1. **Google Calendar** - Strategic meetings for the week
2. **Action Items** - Commitments due this week
3. **People Dossiers** - Team check-ins needed
4. **Stakeholder Dossiers** - Relationship maintenance
5. **Projects** - Active initiatives and blockers
6. **Goals** - Strategic priorities
7. **Recent Journals** - Pattern recognition and risks

## Processing Steps

### Step 1: Determine Week Range

Calculate the upcoming week:
- If run on Sunday: This week (Sunday-Saturday)
- If run on other days: Next full week (upcoming Sunday-Saturday)
- Date format: "Jan 27 - Feb 2"

### Step 2: Fetch Calendar Data (Strategic Meetings Only)

**Access Google Calendar:**
- Use cursor-browser-extension MCP to navigate to:
  `YOUR_CALENDAR_URL`
- Parse meetings for the upcoming week

**Strategic Filtering:**

Apply these filters to show ONLY important meetings:

| Meeting Type | Action |
|--------------|--------|
| Regular 1:1s (titles like "1:1 with", "1-1", "one-on-one") | EXCLUDE (unless related action item) |
| Recurring syncs (titles like "Team Sync", "Weekly Standup", "Daily Standup") | EXCLUDE (unless related action item) |
| Project meetings (contain project names from Projects/) | INCLUDE |
| Stakeholder meetings (contain stakeholder names) | INCLUDE |
| Workshops, reviews, strategic sessions | INCLUDE |
| Planning, decision meetings | INCLUDE |

**Exception Logic:**
- If a 1:1 or sync has a related action item due → INCLUDE with context
- Check action item text for person names or meeting topics
- If match found → add meeting with prep context

**Extract for each meeting:**
- Day of week and date
- Time (start-end)
- Meeting title
- Attendees (if visible on calendar)
- Link attendees to team-members/ or Stakeholders/ dossiers

### Step 3: Load Action Items

Use action-tracker skill dependency to get:
- All open action items ([ ], [~], [!])
- Parse due dates relative to the week range
- Group by urgency:
  - **Due Monday-Tuesday**: Due in next 2 days
  - **Due Later This Week**: Due Wed-Sat
  - **Overdue**: Due date < today
- Link action items to calendar meetings where relevant

### Step 4: Analyze Team Attention Needed

Scan all files in `team-members/` directory:

**1:1 Timing:**
- Look in "## Log" section for last interaction date
- Calculate days since last 1:1
- Flag if > 7 days (1:1 due)

**Recent Performance Signals:**
- Scan last week's journals for mentions of each person
- Look for:
  - Positive signals (wins, quality work, growth)
  - Concern signals (struggling, blocked, needs support)
- Surface for discussion in 1:1s

**Risk Signals:**
- Check recent journals for words like "blocked", "stuck", "struggling"
- Flag team members showing risk patterns

### Step 5: Analyze Stakeholder Maintenance

Scan all files in `Stakeholders/` directory:

**Relationship Debt:**
- Look in "## Interaction Log" for last interaction date
- Flag if > 21 days (3 weeks)
- Suggest quick touch-base

**At-Risk Relationships:**
- Check frontmatter `trust_battery` field
- Check frontmatter `advocacy_status` field
- Flag if trust_battery = "Low" OR advocacy_status = "Detractor"
- Pull recent context from journals

**Strategic Touches:**
- High-value relationships (High Trust + Promoter)
- Suggest maintenance even if not overdue

### Step 6: Schedule Missing Follow-ups

After identifying 1:1s and stakeholder touchbases that are due, invoke the **calendar-follow-up** skill to automatically create calendar events:

**For 1:1s Due (from Step 4):**
- Pass list of people with last 1:1 > 7 days ago
- Calculate next 1:1 date (last_date + 7 days)
- Check if calendar event already exists
- Create Monday board event if missing
- Report: "✓ Scheduled: [Date], 09:00-09:15" or "(already in calendar: [Date], [Time])"

**For Stakeholder Maintenance (from Step 5):**
- Pass list of stakeholders with last interaction > 21 days ago
- Calculate next touchbase date (last_date + 21 days)
- Check if calendar event already exists
- Create Monday board event if missing
- Report: "✓ Scheduled touchbase: [Date], 09:00-09:15" or "(already in calendar: [Date], [Time])"

**Integration:**
Events created in Monday board (ID: YOUR_BOARD_ID) automatically sync to Google Calendar.

### Step 7: Identify Strategic Priorities

**From Goals/:**
- List all goal files
- Extract goal titles
- Link to related projects or action items

**From Projects/:**
- List active projects (check for recent updates in journals)
- Extract known blockers from project files
- Flag critical projects

**Pending Decisions:**
- Scan recent journals for "decided", "decision", "need to decide"
- Extract pending decision items

### Step 8: Extract Blockers & Risks

Scan journals from past week (last 7 days):

**Active Blockers:**
- Look for "blocked", "blocker", "waiting on"
- Extract context and what's blocked

**Dependencies:**
- Look for "waiting for", "depends on", "blocked by"
- Identify external dependencies

**Risks:**
- Look for "risk", "concern", "might slip", "at risk"
- Surface for awareness

## Output Format

**Save Location:**
- Always save the generated report to `Weekly Prep/YYYY-MM-DD_week_prep.md`
- Filename uses the Monday date of the week (e.g., `Weekly Prep/2026-01-27_week_prep.md` for week of Jan 27 - Feb 2)
- This creates a historical record of weekly planning for retrospective analysis

Generate comprehensive week-ahead briefing:

```
Week-Ahead Prep: [Date Range]

━━━ THIS WEEK'S CALENDAR (Strategic Meetings Only) ━━━

[DAY] ([Date]):
• [Meeting Title] ([Time])
  With: [[Dossier/Name1]], [[Dossier/Name2]]
  [Optional: Related action: ✓ Action item text]
  [Optional: Last interactions: Name1 (X days ago), Name2 (Y days ago)]
  Prep: [Context from recent journals or action items]

[Repeat for each day with meetings]

━━━ THIS WEEK'S COMMITMENTS ([N] total) ━━━

DUE MONDAY-TUESDAY ([N]):
- [ ] Action item text → [Optional: Related to meeting]
- [~] In progress item [IN PROGRESS]

DUE LATER THIS WEEK ([N]):
- [ ] Action item text (due: [Day])

OVERDUE ([N]):
- [ ] Overdue item (overdue by [N] days) ⚠️

━━━ TEAM ATTENTION NEEDED ━━━

1:1s DUE THIS WEEK:
- [[team-members/Name]] (last 1:1: [N] days ago on [Date])
  ✓ Scheduled: [Date], 09:00-09:15
  Topics: [Context from recent journals or performance signals]
- [[team-members/Name2]] (last 1:1: [N] days ago on [Date])
  (already in calendar: [Date], [Time])
  Topics: [Context]

WINS TO ACKNOWLEDGE:
- [Name]: [Win from recent journal]
- Run "weekly wins" for full recognition report

CONCERNS TO ADDRESS:
- [Name]: [Concern or risk signal from recent journal]

━━━ STAKEHOLDER MAINTENANCE ━━━

RELATIONSHIP DEBT ([N]+ weeks since interaction):
- [[Stakeholders/Name]] (last: [Date], [N] days ago)
  ✓ Scheduled touchbase: [Date], 09:00-09:15
  Status: [Trust], [Advocacy] - [Context]
- [[Stakeholders/Name2]] (last: [Date], [N] days ago)
  (already in calendar: [Date], [Time])
  Status: [Trust], [Advocacy] - [Context]

AT-RISK RELATIONSHIPS:
- [[Stakeholders/Name]] (last: [Date])
  Status: [Trust], [Advocacy] - needs [repair/attention]
  Context: [Recent journal mention explaining why]
  Suggestion: [Action to take]

━━━ STRATEGIC PRIORITIES ━━━

TOP GOALS THIS WEEK:
1. [[Goals/Goal Name]]
   - [Related action items or next steps]

2. [Goal 2]
   - [Related actions]

CRITICAL PROJECTS:
- [[Projects/Project Name]] - [Status or key milestone]

PENDING DECISIONS:
- [Decision item from journals]

━━━ KNOWN BLOCKERS & RISKS ━━━

ACTIVE BLOCKERS:
- [Blocker description from journal]
  [Optional: Mitigation: [Solution if mentioned]]

DEPENDENCIES WAITING:
- [Dependency description]

WATCHING:
- [Risk items from journals]
```

## Meeting Prep Suggestions

Generate context-aware prep suggestions:

**Based on Related Action Items:**
```
Prep: Action item due - [item text]
```

**Based on Last Interaction:**
```
Last interaction: [X days ago] - [follow up on previous topic if in journal]
```

**Based on Relationship Status:**
```
Status: Low Trust - consider trust-building approach
Status: High Trust, Promoter - strong ally, maintain relationship
```

**Based on Recent Journal Mentions:**
```
Prep: From your [date] journal - [relevant context]
```

## Edge Cases

### No Calendar Access
If browser automation fails:
```
━━━ THIS WEEK'S CALENDAR ━━━

Calendar access unavailable (browser automation error).
Manual review: YOUR_CALENDAR_URL

Continue with other sections...
```

### No Action Items Due
```
━━━ THIS WEEK'S COMMITMENTS ━━━

No action items due this week. Clear slate! ✓
```

### No 1:1s Due
```
━━━ TEAM ATTENTION NEEDED ━━━

1:1s DUE THIS WEEK:
All team members have recent check-ins (within last 7 days) ✓
```

### Empty Sections
Skip sections that have no data rather than showing "None"

## Saving the Report

After generating the week-ahead prep:

1. **Create filename**: `Weekly Prep/[Monday-date]_week_prep.md` (e.g., `Weekly Prep/2026-01-27_week_prep.md`)
2. **Add frontmatter**:
   ```yaml
   ---
   date: [Week start date]
   week_range: [Date range string]
   type: week-prep
   tags: [planning, weekly]
   ---
   ```
3. **Write full report** to the file
4. **Confirm to user**: "Week-ahead prep saved to Weekly Prep/2026-01-27_week_prep.md"

This creates a historical archive of weekly plans for:
- Reviewing what was planned vs what happened
- Tracking progress on goals over time
- Reference during weekly reviews

## User Commands

| Command | Action |
|---------|--------|
| "prep for the week" | Full week-ahead analysis (saves to Weekly Prep/) |
| "week ahead" | Same as above |
| "plan my week" | Same as above |
| "sunday prep" | Same as above (typically run Sunday morning) |

## Integration with Other Skills

**Depends on:**
- `action-tracker` - for open action items and due dates
- `dossier-updater` - for understanding team-members/ and Stakeholders/ structure

**Data it reads:**
- `team-members/*.md` - team 1:1 timing and performance signals
- `Stakeholders/*.md` - relationship health tracking
- `Projects/*.md` - project status and blockers
- `Goals/*.md` - strategic priorities
- `Journal/*.md` - recent patterns and risks

**Tools it uses:**
- cursor-browser-extension MCP - for calendar access
- File system reads - for dossier scanning
- Date calculations - for week ranges and overdue logic
