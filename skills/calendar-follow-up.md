---
name: calendar-follow-up
description: Automatically schedule follow-up reminders in Monday calendar board. Auto-invoked during journal processing (after action items) and week-ahead-prep to ensure follow-ups are in calendar. Creates events with smart timing based on context (1:1s = 7 days, stakeholders = 21 days).
triggers:
  - "schedule follow-up"
  - "create calendar reminder"
  - "schedule touchbase"
  - "set reminder for"
dependencies: []
auto_invoke: true
version: 1.0.0
---

# Calendar Follow-up Scheduler

Automatically create calendar reminders for follow-ups in Monday "Push To Calendar" board with Google Calendar sync.

## Quick Start

This skill is **auto-invoked** as a dependency from:
- `journal-processor` → after action items are extracted
- `week-ahead-prep` → after analyzing 1:1s and stakeholder maintenance needs

It can also be called manually: "schedule follow-up with [person] on [date]"

## Board Configuration

| Setting | Value |
|---------|-------|
| Board ID | `YOUR_BOARD_ID` |
| Board Name | Push To Calendar |
| Default Group | `topics` (New Events) |
| Item Terminology | Event |

**Column IDs:**
- `name` - Event title
- `YOUR_PERSON_COLUMN_ID` - Guests (people column)
- `YOUR_START_DATE_COLUMN_ID` - Start Date (date with time)
- `YOUR_END_DATE_COLUMN_ID` - End Date (date with time)
- `YOUR_GCAL_COLUMN_ID` - Google Calendar event (auto-syncs)

**Default Timing:**
- Start: 07:00:00 (GMT — renders as 09:00 Israel time)
- End: 07:15:00 (GMT — renders as 09:15 Israel time)
- Note: Time format requires seconds (HH:MM:SS)
- Note: Monday API stores times in GMT. Israel (Asia/Jerusalem) is UTC+2, so subtract 2 hours from desired local time.

## Core Workflow

### Step 1: Detect Follow-up Needs

**From Action Items** (during journal processing):

Scan action items for these patterns:
- `[ ] Follow up with [[Name]]`
- `[ ] 1:1 with [[Name]]`
- `[ ] Touch base with [[Name]]`
- `[ ] Schedule meeting with [[Name]]`
- `[ ] Check in with [[Name]]`
- `[ ] Sync with [[Name]]`

Extract:
1. Person/stakeholder name from wiki-links
2. Suggested timing from text (e.g., "next week", "in 3 days", "Wednesday")
3. Context (1:1, touchbase, meeting)

**From Dossiers** (during week-ahead-prep):

Check for overdue interactions:
- `team-members/*.md` files: Check `## Log` section for last 1:1 date
  - Flag if > 7 days ago
  - Calculate next 1:1 date as: last_date + 7 days
- `Stakeholders/*.md` files: Check `## Interaction Log` for last interaction
  - Flag if > 21 days ago
  - Calculate next touchbase as: last_date + 21 days

### Step 2: Check Existing Calendar

Before creating any event, check Monday board for existing events:

1. **Query board** YOUR_BOARD_ID for upcoming items (next 30 days)
2. **Search event titles** for person/stakeholder name
3. **Skip creation** if matching event exists

This prevents duplicates when:
- User has already manually scheduled
- Event was created in previous run
- Meeting already exists in different time slot

### Step 3: Calculate Follow-up Date

**Smart Date Calculation:**

| Context | Timing Logic | Example |
|---------|--------------|---------|
| **1:1 Due** | Last interaction + 7 days | Last 1:1 on Jan 25 → Schedule Feb 1 |
| **Stakeholder** | Last interaction + 21 days | Last touch on Jan 10 → Schedule Jan 31 |
| **Action Item** | Parse from text or +7 days default | "next week" → 7 days from today |
| **Recognition** | Last recognition + 14 days | Last feedback Jan 20 → Schedule Feb 3 |
| **Custom** | User-specified date | "Feb 15" → Feb 15 |

**Date Validation:**
- Ensure date is in the future
- If calculated date is in the past, use today + offset
- Format: `YYYY-MM-DD`

**Time Parsing:**
- Default: 07:00:00-07:15:00 (GMT = 09:00-09:15 Israel time)
- If text contains time reference, convert to GMT: "at 2pm Israel" → 12:00:00-12:15:00
- Always use 15-minute slots
- Format: HH:MM:SS (seconds required by Monday API)
- IMPORTANT: All times must be in GMT. Subtract 2 hours from Israel local time.

### Step 4: Create Calendar Event

Use Monday MCP to create the item:

```javascript
CallMcpTool(
  server: "user-monday-mcp",
  toolName: "create_item",
  arguments: {
    boardId: YOUR_BOARD_ID,
    name: "[Event Title]",
    groupId: "topics",
    columnValues: JSON.stringify({
      "YOUR_START_DATE_COLUMN_ID": {
        "date": "YYYY-MM-DD",
        "time": "07:00:00"
      },
      "YOUR_END_DATE_COLUMN_ID": {
        "date": "YYYY-MM-DD",
        "time": "07:15:00"
      }
    })
  }
)
```

**Important:** Time format must include seconds (HH:MM:SS) for Monday API to accept it.

**Event Title Format:**

| Context | Title Format |
|---------|--------------|
| 1:1 from action item | "1:1 Follow-up: [Person Name]" |
| 1:1 from dossier | "1:1 Follow-up: [Person Name]" |
| Stakeholder touchbase | "Touch-base: [Stakeholder Name]" |
| Recognition reminder | "Recognition Reminder: [Person Name]" |
| Generic follow-up | "Follow-up: [Name]" |
| Custom | User-specified title |

### Step 5: Report Created Events

Return structured output for the calling skill to display:

```json
{
  "events_created": [
    {
      "person": "Alex K",
      "type": "1:1",
      "date": "2026-02-05",
      "time": "09:00-09:15",
      "title": "1:1 Follow-up: Alex K",
      "monday_item_id": "123456789"
    }
  ],
  "events_skipped": [
    {
      "person": "Jordan R",
      "reason": "Already in calendar: Feb 3, 10:00"
    }
  ],
  "errors": []
}
```

## Integration with Other Skills

### journal-processor.md Integration

Called after `action-tracker` completes:

```markdown
## Processing Steps

1. Parse journal entry
2. Extract entities (via dossier-updater)
3. Extract action items (via action-tracker)
4. **Detect follow-up needs** (via calendar-follow-up) ← NEW
5. Create Monday tasks (via monday-integration)
6. Show action item summary
```

**What it does:**
- Receives list of action items from action-tracker
- Scans for follow-up patterns
- Creates calendar events automatically
- Returns created events for reporting

**Example Output Addition:**
```
Action items extracted:
- [ ] Follow up with [[team-members/Alex K]] on API review

✓ Calendar follow-up created:
  - Title: "1:1 Follow-up: Alex K"
  - Date: Feb 8, 2026 (7 days from now)
  - Time: 09:00-09:15
  - Monday board: Push To Calendar
```

### week-ahead-prep.md Integration

Called after analyzing team and stakeholder needs:

```markdown
## Processing Steps

### Step 4: Analyze Team Attention Needed
- Scan team-members/*.md for 1:1 timing
- Calculate days since last 1:1
- Flag if > 7 days

### Step 5: Analyze Stakeholder Maintenance
- Scan Stakeholders/*.md for interaction timing
- Flag if > 21 days

### Step 6: Schedule Missing Follow-ups ← NEW
- Invoke calendar-follow-up skill
- Pass list of people/stakeholders needing follow-ups
- Check calendar for existing events
- Create events for those without scheduled follow-ups
- Report created events
```

**Example Enhanced Output:**
```
━━━ TEAM ATTENTION NEEDED ━━━

1:1s DUE THIS WEEK:
- [[team-members/Alex K]] (last 1:1: 8 days ago on Jan 26)
  ✓ Scheduled: Feb 5, 09:00-09:15
- [[team-members/Jordan R]] (last 1:1: 6 days ago on Jan 27)
  (already in calendar: Feb 3, 10:00)

STAKEHOLDER MAINTENANCE:
- [[Stakeholders/Sam G]] (last: Jan 10, 22 days ago)
  ✓ Scheduled touchbase: Feb 8, 09:00-09:15
```

## Action Item Pattern Detection

### Pattern Matching

Use regex patterns to detect follow-up needs:

```
Follow-up indicators:
- "follow up" (case insensitive)
- "1:1" or "one-on-one" or "1-1"
- "touch base" or "touchbase"
- "schedule meeting"
- "check in" or "check with"
- "sync with"

Person/Stakeholder extraction:
- Extract text between [[ and ]]
- Determine if team-members/ or Stakeholders/ prefix
- Fall back to name only if no prefix

Timing extraction:
- "next week" → +7 days
- "in X days" → +X days
- "on Monday/Tuesday/etc" → next occurrence of that day
- "Wednesday" → next Wednesday
- No timing → default +7 days
```

### Examples

**Input:** `[ ] Follow up with [[team-members/Alex K]] next week`
- Person: Alex K (from team-members)
- Type: 1:1
- Timing: +7 days
- Title: "1:1 Follow-up: Alex K"

**Input:** `[ ] Touch base with [[Stakeholders/Sam G]]`
- Person: Sam G (from Stakeholders)
- Type: Touchbase
- Timing: +21 days (default for stakeholders)
- Title: "Touch-base: Sam G"

**Input:** `[ ] 1:1 with [[Jordan R]] on Wednesday`
- Person: Jordan R (no prefix → check both team-members and Stakeholders)
- Type: 1:1
- Timing: Next Wednesday
- Title: "1:1 Follow-up: Jordan R"

**Input:** `[ ] Schedule meeting with [[Taylor S]] in 3 days at 2pm`
- Person: Taylor S
- Type: Meeting
- Timing: +3 days
- Time: 14:00-14:15
- Title: "Follow-up: Taylor S"

## Duplicate Prevention

**Query Existing Events:**

```javascript
// Use get_board_items_page to fetch upcoming events
CallMcpTool(
  server: "user-monday-mcp",
  toolName: "get_board_items_page",
  arguments: {
    boardId: YOUR_BOARD_ID,
    limit: 100
  }
)
```

**Check Logic:**

For each follow-up to create:
1. Get all items from Monday board
2. Filter items where Start Date >= today
3. Search item names for person/stakeholder name (case insensitive)
4. If match found within timing window (±3 days), skip creation
5. Report as "already in calendar"

**Timing Window:**
- For 1:1s: Skip if event exists within ±3 days of calculated date
- For stakeholders: Skip if event exists within ±7 days
- Reasoning: Allows flexibility if user manually scheduled slightly different date

## Manual Invocation

Users can manually call the skill:

**Commands:**
- "schedule follow-up with [person] on [date]"
- "create calendar reminder for [person] [date]"
- "schedule touchbase with [stakeholder] next week"
- "set reminder for 1:1 with [person]"

**Processing:**
1. Extract person/stakeholder name
2. Extract or calculate date
3. Determine context (1:1 vs touchbase)
4. Create event
5. Confirm creation

**Example:**
```
User: "schedule follow-up with Alex K on Feb 15"

Agent:
✓ Created calendar event:
  - Title: "1:1 Follow-up: Alex K"
  - Date: Feb 15, 2026
   - Time: 09:00-09:15 Israel time (stored as 07:00-07:15 GMT)
   - Monday board: Push To Calendar (#YOUR_BOARD_ID)
   - Google Calendar: Auto-synced
```

## Configuration

Default configuration (can be customized):

```yaml
calendar_follow_up_config:
  auto_schedule: true          # Auto-create from skills
  default_time: "07:00:00"     # Default start time in GMT (= 09:00 Israel time)
  default_duration: 15         # Minutes (00:15:00)
  timezone: "Asia/Jerusalem"   # Target timezone (UTC+2; all API times sent as GMT)
  board_id: YOUR_BOARD_ID      # Monday board
  group_id: "topics"           # Default group
  
  timing:
    one_on_one_days: 7         # Days for 1:1 follow-ups
    stakeholder_days: 21       # Days for stakeholder touchbases
    recognition_days: 14       # Days for recognition reminders
    default_days: 7            # Default for unspecified
    
  duplicate_window:
    one_on_one: 3              # Days ± for duplicate check
    stakeholder: 7             # Days ± for duplicate check
```

## Error Handling

### Monday API Errors

If Monday API call fails:
```
✗ Failed to create calendar event for [Name]:
  Error: [error message]
  
  Please create manually:
  - Title: "1:1 Follow-up: [Name]"
  - Date: [date] 09:00-09:15
```

Continue processing other events and report all failures at end.

### Invalid Date

If calculated date is invalid or in the past:
```
⚠️ Invalid date calculated for [Name]: [date]
  Using default: [today + 7 days]
```

### Missing Person/Stakeholder

If person cannot be resolved:
```
⚠️ Cannot resolve person: [Name]
  Creating event without guest linking
```

### Duplicate Event Detected

If event already exists (not an error):
```
(already in calendar: [date], [time])
```

## Output Format

### Success

```
✓ Calendar follow-up created:
  - Title: "[Event Title]"
  - Date: [Date] ([relative time])
  - Time: 09:00-09:15
  - Monday board: Push To Calendar
  - Google Calendar: Auto-synced
```

### Skipped

```
(already in calendar: [date], [time])
```

### Summary (for multiple events)

```
📅 Calendar Follow-ups Scheduled (3):
- ✓ Alex K (Feb 5, 09:00-09:15)
- ✓ Sam G (Feb 8, 09:00-09:15)
- (skipped) Jordan R (already scheduled: Feb 3, 10:00)
```

## Edge Cases

### Past Dates

If calculated date is in the past:
- Use today + default offset instead
- Log warning about adjustment

### Weekend Dates

If calculated date falls on weekend:
- Use the date as calculated (user's calendar might have weekend meetings)
- Don't auto-adjust to weekday

### No Wiki-link in Action Item

If action item has pattern but no wiki-link:
```
[ ] Follow up with Sarah on project
```
- Try to match "Sarah" against team-members/*.md and Stakeholders/*.md filenames
- If multiple matches, skip (ambiguous)
- If single match, use it
- If no match, skip

### Multiple People in One Action Item

```
[ ] 1:1 with [[Alex K]] and [[Jordan R]]
```
- Create separate events for each person
- Use same timing for both

### Already Scheduled Different Time

If event exists but at different time:
- Report as "already in calendar" with actual time
- Don't create duplicate
- User can manually adjust if needed

## Testing

### Unit Tests

**Test 1: Pattern Detection**
- Input: Various action item formats
- Verify: Correct person, type, timing extracted

**Test 2: Date Calculation**
- Input: Different timing patterns
- Verify: Correct dates calculated

**Test 3: Duplicate Detection**
- Input: Existing events in board
- Verify: Duplicates correctly identified

### Integration Tests

**Test 4: Journal Processing**
- Process journal with follow-up action items
- Verify: Events created in Monday board

**Test 5: Week-ahead-prep**
- Run with overdue 1:1s/stakeholders
- Verify: Events created only for missing ones

**Test 6: Manual Invocation**
- Call skill manually
- Verify: Event created correctly

### Edge Case Tests

**Test 7: Past Dates**
- Calculate date that would be in past
- Verify: Adjusted to future date

**Test 8: API Errors**
- Simulate Monday API failure
- Verify: Graceful error handling, continues processing

**Test 9: Ambiguous Names**
- Action item with name matching multiple people
- Verify: Handled appropriately

## User Commands

| Command | Action |
|---------|--------|
| "schedule follow-up with [person] on [date]" | Create calendar event for specific person/date |
| "create calendar reminder for [person]" | Create event with smart date calculation |
| "schedule touchbase with [stakeholder]" | Create stakeholder touchbase event |
| "set reminder for 1:1 with [person] next week" | Create 1:1 event for next week |

Auto-invoked commands (no explicit user call needed):
- During "process my journal" - automatically scans action items
- During "prep for the week" - automatically checks for missing follow-ups

## Version History

- **v1.0.0** (2026-02-01): Initial implementation
  - Auto-invoke during journal processing and week-ahead-prep
  - Smart timing based on context
  - Duplicate prevention
  - Monday board integration with Google Calendar sync
