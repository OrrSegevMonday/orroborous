---
name: staff-meeting
description: Chief-of-staff style executive briefing with interactive review and decision documentation
triggers:
  - "staff meeting"
  - "chief of staff briefing"
  - "executive briefing"
  - "staff sync"
dependencies: [action-tracker, dossier-updater, week-ahead-prep]
version: 1.0.0
---

# Staff Meeting (Chief of Staff Briefing)

Comprehensive executive briefing that aggregates project status, team standing, upcoming meetings, action items, relationship health, and skill improvement suggestions, then interactively reviews each section to document decisions and update all relevant files.

## Quick Start

When you say "staff meeting" or "executive briefing":

1. **Generate comprehensive report** across all organizational dimensions
2. **Save to file** at `staff-meetings/YYYY-MM-DD_staff_meeting.md`
3. **Present summary** of all sections with key counts
4. **Review interactively** section by section, waiting for your input
5. **Document decisions** and update all relevant files
6. **Save final report** with decisions and follow-up actions

## Data Sources

This skill aggregates information from multiple systems:

| Source | What It Provides | Used For |
|--------|------------------|----------|
| `Projects/*.md` | Project status, blockers, decisions | Projects Status section |
| `team-members/*.md` | Performance ratings, 1:1 timing, wins | Employees Standing section |
| Google Calendar | Strategic meetings for next week | Upcoming Meetings section |
| `Journal/*.md` | Action items, risks, blockers, decisions | Multiple sections |
| `Stakeholders/*.md` | Relationship health, interaction timing | Important Matters section |
| Cursor chat history | Workflow patterns, feature requests | Skill Upgrade Suggestions |

## Report Structure

The generated report contains these sections:

```
## 📊 PROJECTS STATUS
Active projects with status, blockers, recent updates, decisions needed

## 👥 EMPLOYEES STANDING  
Team performance overview, 1:1 status, wins, concerns, development areas

## 📅 UPCOMING MEETINGS
Strategic meetings for the next week with prep context

## ✅ OPEN ACTION ITEMS
All open items categorized by urgency (overdue, due soon, no due date)

## ⚠️ IMPORTANT MATTERS
Risks, at-risk relationships, relationship debt, blockers, pending decisions

## 🔧 SKILL UPGRADE SUGGESTIONS
Recommendations based on recent conversation patterns

## 📝 DECISIONS MADE
Populated during interactive review

## 🎯 FOLLOW-UP ACTIONS
New action items created during the meeting
```

## Processing Steps

### Step 1: Generate Report Sections

#### 1.1 Projects Status

Scan all `Projects/*.md` files:

**Extract from each project:**
- Project name (from filename and H1)
- Status from frontmatter (`status: active/on-hold/completed`)
- Owner from frontmatter
- Recent updates: Scan last 7 days of journals for mentions
- Blockers: Extract from `## Blockers` section
- Decisions needed: Extract from `## Decisions` section or journal mentions

**Format:**
```
[[Projects/Project Name]] (Owner: [Name])
Status: [active/on-hold]
Recent: [Latest journal mention or "No recent activity"]
Blockers: [List blockers or "None"]
Decisions: [Pending decisions or "None"]
```

**Sorting:** Active projects first, then by most recent journal mention

#### 1.2 Employees Standing

Scan all `team-members/*.md` files:

**Extract from each person:**
- Name, role from frontmatter
- Performance/Potential ratings from frontmatter (`performance: N`, `potential: N`)
- Days since last 1:1: Parse `## Log` section, find most recent date
- Recent wins: Extract from `## Wins` section (last 2-3 entries)
- Development areas: Extract from `## Development Areas` section
- Recent journal mentions: Scan last 7 days for mentions, note sentiment (positive/concern)
- Recognition status: Days since last recognition (from `## Feedback Given`)

**Format:**
```
[[team-members/Name]] ([Role])
Rating: Performance [N], Potential [N]
Last 1:1: [Date] ([N] days ago) [🟢 Recent / 🟡 Due Soon / 🔴 Overdue]
Recent Wins: [Latest wins or "None captured"]
Development: [Key development areas]
Recognition: [Days since last] [⚠️ if > 14 days]
Recent Notes: [Positive signals or concerns from journals]
```

**Status indicators:**
- 🟢 Recent: < 7 days since last 1:1
- 🟡 Due Soon: 7-10 days since last 1:1
- 🔴 Overdue: > 10 days since last 1:1

**Sorting:** By 1:1 status (overdue first), then by name

#### 1.3 Upcoming Meetings

**Determine time range:**
- Calculate next 7 days from today
- Format: "Jan 27 - Feb 2"

**Access Google Calendar:**
Use cursor-browser-extension MCP to navigate to:
```
YOUR_CALENDAR_URL
```

**Strategic Filtering Rules:**

Apply these filters to show ONLY important meetings:

| Meeting Type | Action |
|--------------|--------|
| Regular 1:1s (titles: "1:1 with", "1-1", "one-on-one") | EXCLUDE (unless action-item-related) |
| Recurring syncs (titles: "Team Sync", "Weekly Standup", "Daily") | EXCLUDE (unless action-item-related) |
| Project meetings (contain project names from Projects/) | INCLUDE |
| Stakeholder meetings (contain stakeholder names) | INCLUDE |
| Workshops, reviews, strategic sessions | INCLUDE |
| Planning, decision meetings | INCLUDE |

**Exception Logic:**
- If a 1:1 or sync has related action item due → INCLUDE with context
- Check action items for person names or meeting topics
- If match found → add meeting with prep note

**Extract for each meeting:**
- Day of week and date
- Time (start-end)
- Meeting title
- Attendees (if visible)
- Link attendees to team-members/ or Stakeholders/ dossiers
- Prep context from related action items or recent journals

**Format:**
```
[DAY] ([Date]):
• [Meeting Title] ([Time])
  With: [[Dossier/Name1]], [[Dossier/Name2]]
  [If action-item-related: Related: ✓ Action item text]
  [If dossier context: Last interaction: Name1 (X days ago)]
  Prep: [Context from journals/action items or "None"]
```

**Error handling:**
If calendar access fails:
```
━━━ UPCOMING MEETINGS ━━━

Calendar access unavailable (browser automation error).
Manual review: [calendar URL]

Continue with other sections...
```

#### 1.4 Open Action Items

Use `action-tracker` skill dependency to get all open items.

**Categorization:**
- **Overdue**: Due date < today
- **Due Soon**: Due today or within next 3 days
- **No Due Date**: No time reference found

**Format (same as action-tracker):**
```
OVERDUE ([N]):
- [ ] Item text (from [Journal date], due [date], [N] days overdue) ⚠️

DUE SOON ([N]):
- [ ] Item text (from [Journal date], due [date])

NO DUE DATE ([N]):
- [ ] Item text (from [Journal date])
```

Include status markers: `[ ]` = Not Started, `[~]` = In Progress, `[!]` = Stuck

#### 1.5 Important Matters

**At-Risk Relationships:**

Scan all `Stakeholders/*.md` files:
- Check frontmatter: `trust_battery: Low` OR `advocacy_status: Detractor`
- Extract recent context from journals (last 14 days)
- Format:
  ```
  [[Stakeholders/Name]] ([Role])
  Status: [Trust], [Advocacy]
  Last interaction: [Date] ([N] days ago)
  Context: [Recent journal mentions explaining situation]
  Suggested action: [Recommendation]
  ```

**Relationship Debt:**

Scan all `Stakeholders/*.md` files:
- Look in `## Interaction Log` for last interaction date
- Flag if > 21 days (3 weeks)
- Format:
  ```
  [[Stakeholders/Name]] ([Role])
  Last interaction: [Date] ([N] days ago)
  Status: [Trust], [Advocacy]
  Suggested action: Quick check-in or touchbase
  ```

**Recent Blockers:**

Scan journals from past 7 days:
- Look for: "blocked", "blocker", "waiting on", "stuck"
- Extract context (full bullet or sentence)
- Format:
  ```
  - [Blocker description] (from [Journal date])
  ```

**Pending Decisions:**

Scan journals from past 14 days:
- Look for: "need to decide", "decision pending", "decide on"
- Extract decision item
- Format:
  ```
  - [Decision description] (from [Journal date])
  ```

**Overall format:**
```
AT-RISK RELATIONSHIPS ([N]):
[Details]

RELATIONSHIP DEBT (3+ weeks):
[Details]

RECENT BLOCKERS:
[Details or "None detected"]

PENDING DECISIONS:
[Details or "None detected"]
```

#### 1.6 Skill Upgrade Suggestions

**Analyze Cursor conversation history:**

Look for patterns in recent conversations:

1. **Repeated Manual Workflows**
   - Same type of request multiple times
   - Example: "Update all stakeholders with X" → Suggest: bulk-updater skill

2. **New Data Patterns**
   - Tracking new types of information not in current skills
   - Example: Multiple questions about specific metrics → Suggest: metric-tracker

3. **Gaps in Functionality**
   - User asks "can you track X?" but no skill exists
   - Feature requests discussed but not implemented

4. **Workflow Improvements**
   - Inefficient multi-step processes that could be automated
   - Manual steps that could be integrated

**Format each suggestion:**
```
[N]. [Skill Name]
   What: [Brief description of what it would do]
   Why: [What problem it solves or inefficiency it removes]
   Based on: [Pattern observed in conversations]
```

**If no chat history accessible:**
```
🔧 SKILL UPGRADE SUGGESTIONS

Chat history analysis unavailable in this session.
Note: Run this skill in a session with accessible conversation history to get suggestions.
```

### Step 2: Save Initial Report

**Filename:** `staff-meetings/YYYY-MM-DD_staff_meeting.md` (using today's date)

**Frontmatter:**
```yaml
---
date: YYYY-MM-DD
type: staff-meeting
generated: YYYY-MM-DDTHH:MM:SS
tags: [staff-meeting, executive-briefing]
---
```

**Content:** All sections as generated above, with empty "Decisions Made" and "Follow-up Actions" sections

**Confirm to user:**
```
Generated staff meeting report for [Date]
Saved to: staff-meetings/YYYY-MM-DD_staff_meeting.md
```

### Step 3: Present Summary

Show high-level overview:

```
Summary:
- [N] active projects
- [N] team members ([N] 1:1s overdue)
- [N] upcoming strategic meetings
- [N] open action items ([N] overdue)
- [N] at-risk relationships
- [N] relationship debts
- [N] skill upgrade suggestions

Let's review each section. Ready to proceed?
```

Wait for user acknowledgment before proceeding.

### Step 4: Interactive Section-by-Section Review

For each section, follow this pattern:

```
━━━ [SECTION NAME] ━━━

[Display section content]

Any decisions, updates, or actions on this section?
```

**Wait for user input** before proceeding to next section.

**Section order:**
1. Projects Status
2. Employees Standing
3. Upcoming Meetings
4. Open Action Items
5. Important Matters
6. Skill Upgrade Suggestions

**For each user response:**
1. Document in "Decisions Made" section with timestamp
2. Execute any file updates based on decision
3. Add any new action items to "Follow-up Actions" section
4. Confirm updates made
5. Move to next section

### Step 5: Document Decisions

**Format for "Decisions Made" section:**

```
## 📝 DECISIONS MADE

### [Section Name]
**Decision:** [User's decision or update]
**Action taken:** [What files were updated or what was done]
**Timestamp:** [HH:MM]

[Repeat for each decision across all sections]
```

### Step 6: File Updates Based on Decisions

Execute these updates based on user input:

| Decision Type | Action | Implementation |
|--------------|--------|----------------|
| **Project status change** | Update `Projects/[name].md` | Update `status:` in frontmatter, add note in `## Notes` with date |
| **Project blocker resolved** | Update `Projects/[name].md` | Remove from `## Blockers`, add resolution to `## Notes` |
| **Project blocker added** | Update `Projects/[name].md` | Add to `## Blockers` section |
| **Performance rating change** | Update `team-members/[name].md` | Update `performance:` or `potential:` in frontmatter, add entry to rating history table |
| **Schedule 1:1** | Document only | Add to "Follow-up Actions" (don't auto-calendar) |
| **Give recognition** | Update `team-members/[name].md` | Add to `## Feedback Given` section with date |
| **New action item** | Update `Journal/[today].md` | Add to `## Action Items` section |
| **Clear action item** | Update original journal | Find item in source journal, change `[ ]`/`[~]`/`[!]` to `[x]` |
| **Stakeholder follow-up** | Update `Stakeholders/[name].md` | Add to `## Strategic Leverage` section or schedule note |
| **Update trust/advocacy** | Update `Stakeholders/[name].md` | Update frontmatter and add to history tables |

**After each update:**
- Confirm: "✓ Updated [filename]"
- Document in "Decisions Made" section

### Step 7: Finalize Report

After all sections reviewed:

1. **Update report file** with all decisions and follow-up actions
2. **Show summary:**
   ```
   Staff meeting complete!
   
   Decisions made: [N]
   Files updated: [N]
   Follow-up actions: [N]
   
   Full report saved to: staff-meetings/YYYY-MM-DD_staff_meeting.md
   ```

## User Commands

| Command | Action |
|---------|--------|
| "staff meeting" | Full executive briefing with interactive review |
| "chief of staff briefing" | Same as above |
| "executive briefing" | Same as above |
| "staff sync" | Same as above |

## Integration with Other Skills

**Depends on:**
- `action-tracker` - for open action items list
- `dossier-updater` - for understanding dossier structure and update patterns
- `week-ahead-prep` - for calendar access pattern

**Complements:**
- `win-tracker` - can inform "Employees Standing" section
- `performance-ratings` - rating changes may be decided during staff meeting
- `relationship-tracker` - relationship issues surfaced in "Important Matters"

**Data it reads:**
- `Projects/*.md` - all project files
- `team-members/*.md` - all team member dossiers
- `Stakeholders/*.md` - all stakeholder dossiers
- `Journal/*.md` - recent journals for context
- Google Calendar - upcoming meetings
- Cursor conversation history - for skill suggestions

**Data it writes:**
- `staff-meetings/*.md` - meeting reports
- `Projects/*.md` - status and blocker updates
- `team-members/*.md` - rating, feedback, or note updates
- `Stakeholders/*.md` - relationship or strategic updates
- `Journal/[today].md` - new action items

## Edge Cases & Error Handling

### Calendar Access Fails
Show manual link, continue with other sections:
```
━━━ UPCOMING MEETINGS ━━━

Calendar access unavailable (browser automation error).
Manual review: https://calendar.google.com/calendar/...

Continuing with other sections...
```

### No Active Projects
```
📊 PROJECTS STATUS

No active projects found in Projects/ directory.
```

### No Team Members
```
👥 EMPLOYEES STANDING

No team member files found in team-members/ directory.
```

### Missing Dossier File
During interactive review, if user mentions updating a non-existent file:
```
⚠️ [Name].md doesn't exist in [team-members/Stakeholders/Projects].
Would you like me to create it? (yes/no)
```

### No Chat History
```
🔧 SKILL UPGRADE SUGGESTIONS

Chat history analysis unavailable in this session.
Run this skill in a session with accessible conversation history for suggestions.
```

### Action Tracker Dependency Fails
Fall back to inline scanning:
```
⚠️ Action tracker dependency unavailable, scanning journals directly...
[Perform inline action item scan]
```

## Example Full Interaction

```
User: "staff meeting"

Agent:
Generating staff meeting report...

[Scans Projects/, team-members/, Stakeholders/, Journals, Calendar]

Generated staff meeting report for 2026-01-27
Saved to: staff-meetings/2026-01-27_staff_meeting.md

Summary:
- 2 active projects
- 5 team members (2 1:1s overdue)
- 8 upcoming strategic meetings
- 13 open action items (4 overdue)
- 2 at-risk relationships
- 1 relationship debt
- 3 skill upgrade suggestions

Let's review each section. Ready to proceed?

User: "yes"

Agent:
━━━ PROJECTS STATUS ━━━

[[Projects/Q3 Infrastructure Project]] (Owner: unassigned)
Status: active
Recent: Shared gantt with Jamie and Alex (Jan 26)
Blockers: None
Decisions: Need better tracking - follow up with team

[[Projects/Analytics Platform Workshop]] (Owner: unassigned)
Status: active
Recent: Taylor enthusiastic after explanation (Jan 22)
Blockers: Waiting on Taylor's mock-UI
Decisions: None

Any decisions, updates, or actions on this section?

User: "Assign Q3 Infrastructure Project to Casey D, and create action item to follow up on tracking"

Agent:
✓ Updated Projects/Q3 Infrastructure Project.md (owner set to Casey D)
✓ Updated Journal/2026-01-27.md (added action item)

━━━ EMPLOYEES STANDING ━━━

[[team-members/Alex K]] (Product Analyst)
Rating: Performance 3, Potential 3
Last 1:1: Jan 26 (1 day ago) 🟢 Recent
Recent Wins: Thorough notebooks, CWM departments analysis
Development: Focus on hypotheses first, be more confident
Recognition: 1 day ago ✓
Recent Notes: Great work, still growing confidence

[[team-members/Jordan R]] (Team Member)
Rating: Performance 3, Potential 3
Last 1:1: Jan 21 (6 days ago) 🟡 Due Soon
Recent Wins: Sidekick dashboard, Deskless discovery ownership
Development: None captured
Recognition: 47+ days ago ⚠️
Recent Notes: Strong recent work, needs recognition

[... continues for all team members ...]

Any decisions on team members (ratings, feedback, actions)?

User: "Recognize Jordan for the dashboard work, and schedule 1:1s for those overdue"

Agent:
✓ Updated team-members/Jordan R.md (added recognition to Feedback Given)
✓ Added to Follow-up Actions: Schedule 1:1s with Sam B and Jordan R

[... continues through all sections ...]

━━━ SKILL UPGRADE SUGGESTIONS ━━━

1. gantt-project-tracker
   What: Automated tracking and updates for project gantt charts
   Why: User manually tracking Q3 Infrastructure Project, mentioned need for better tracking
   Based on: Jan 26 journal entry and repeated project tracking patterns

2. bulk-dossier-updater
   What: Update multiple dossier files at once with common changes
   Why: Streamline updates when patterns apply to multiple people/stakeholders
   Based on: Pattern of similar updates across multiple dossier files

3. meeting-prep-generator
   What: Generate prep materials for specific upcoming meetings
   Why: User needs context for strategic meetings beyond calendar entry
   Based on: Manual prep work visible in journal entries

Any decisions on these suggestions?

User: "Let's prioritize the gantt tracker and meeting prep. Not sure about bulk updater yet."

Agent:
✓ Documented in Follow-up Actions

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Staff meeting complete!

Decisions made: 5
Files updated: 4
Follow-up actions: 4

Full report saved to: staff-meetings/2026-01-27_staff_meeting.md
```

## Implementation Notes for AI Agent

### When Executing This Skill

1. **Load dependencies first:**
   - Read `action-tracker.md` to understand action item format
   - Read `dossier-updater.md` to understand dossier structure
   - Read `week-ahead-prep.md` for calendar access pattern

2. **Data gathering phase:**
   - Scan all relevant directories (Projects/, team-members/, Stakeholders/)
   - Use Glob tool to find all .md files
   - Read each file and extract required data
   - Scan journals for recent mentions and context
   - Access calendar if possible (handle failure gracefully)

3. **Report generation:**
   - Build each section systematically
   - Save complete report to staff-meetings/ directory
   - Show summary to user

4. **Interactive review:**
   - Present one section at a time
   - Wait for user response after each section
   - Parse user's response for actionable items
   - Execute file updates as needed
   - Document all decisions
   - Move to next section only after user responds

5. **Decision parsing patterns:**
   - "Assign X to Y" → Update project owner
   - "Schedule 1:1" → Add to follow-up actions
   - "Recognize X for Y" → Update team-members dossier
   - "Clear action item" → Update journal
   - "Change rating" → Update team-members dossier with history
   - "Follow up with X" → Add action item
   - "Update status to X" → Update project status

6. **File update execution:**
   - Use StrReplace for precise updates
   - Preserve existing structure and formatting
   - Add timestamps to new entries
   - Confirm each update to user

7. **Final save:**
   - Update staff-meetings/[date]_staff_meeting.md with all decisions
   - Show completion summary
   - List all files modified

### Chat History Analysis

When analyzing chat history for skill suggestions:

1. **Access conversation history:** Read from Cursor's conversation logs if available
2. **Pattern detection:** Look for:
   - Multiple similar requests ("update X", "update Y", "update Z")
   - User frustration with manual steps
   - Feature requests ("can you make it so...")
   - Workflow inefficiencies ("this takes too long")
3. **Generate suggestions:** Format as actionable skill proposals
4. **Fallback:** If no history accessible, show note and skip section

### Error Recovery

- If any section fails, continue with others
- Document failures in report
- Offer manual alternatives where possible
- Don't block interactive review on data gathering issues
