---
name: monday-integration
description: Create Monday.com tasks from journal action items and sync status bi-directionally
triggers:
  - "sync to monday"
  - "sync from monday"
  - "sync tasks"
  - "full sync"
dependencies: [action-tracker]
version: 1.0.0
---

# Monday.com Integration

Create tasks in Monday.com from journal action items and sync status bi-directionally.

## Board Configuration

| Setting | Value |
|---------|-------|
| Board ID | `YOUR_BOARD_ID` |
| Board Name | YOUR_BOARD_NAME |
| Default Group | `topics` (Action Items) |
| API Endpoint | `https://api.monday.com/v2` |
| API Key | Stored in `.env` as `MONDAY_API_KEY` |

## Column Mapping

| Journal Data | Monday Column | Column ID | Notes |
|--------------|---------------|-----------|-------|
| Task text | Name | `name` | Item title |
| Parsed due date | Due Date | `YOUR_DUE_DATE_COLUMN_ID` | Format: `YYYY-MM-DD` |
| Journal date | Created Date | `YOUR_CREATED_DATE_COLUMN_ID` | Source entry date |
| Task context | Description | `YOUR_DESCRIPTION_COLUMN_ID` | Include source journal reference |
| Detected type | Action Type | `YOUR_ACTION_TYPE_COLUMN_ID` | Follow-up, Meeting, Email, Call |
| Initial state | Status | `YOUR_STATUS_COLUMN_ID` | Default: "Not Started" |
| Urgency signals | Task Priority | `YOUR_PRIORITY_COLUMN_ID` | Low, Medium, High, Critical |

## Action Type Detection

Detect action type from journal text:

| Pattern | Action Type | Dropdown ID |
|---------|-------------|-------------|
| "follow up", "check with", "ask about" | Follow-up | `1` |
| "meet", "1:1", "sync", "call with" | Meeting | `2` |
| "email", "send", "write to" | Email | `3` |
| "call", "phone" | Call | `4` |
| (default) | Follow-up | `1` |

## Priority Detection

Detect priority from journal text:

| Pattern | Priority | Label |
|---------|----------|-------|
| "urgent", "asap", "critical", "blocker" | Critical | `Critical ⚠️️` |
| "important", "high priority" | High | `High` |
| "when possible", "low priority" | Low | `Low` |
| (default) | Medium | `Medium` |

## GraphQL Mutation

To create a task, execute this mutation:

```graphql
mutation {
  create_item(
    board_id: YOUR_BOARD_ID,
    group_id: "topics",
    item_name: "Task title here",
    column_values: "{
      \"YOUR_DUE_DATE_COLUMN_ID\": {\"date\": \"2026-01-25\"},
      \"YOUR_CREATED_DATE_COLUMN_ID\": {\"date\": \"2026-01-20\"},
      \"YOUR_DESCRIPTION_COLUMN_ID\": \"From journal 2026-01-20\",
      \"YOUR_ACTION_TYPE_COLUMN_ID\": {\"ids\": [1]},
      \"YOUR_STATUS_COLUMN_ID\": {\"label\": \"Not Started\"},
      \"YOUR_PRIORITY_COLUMN_ID\": {\"label\": \"Medium\"}
    }"
  ) {
    id
    name
  }
}
```

## API Call Template

Use curl to create items:

```bash
curl -X POST https://api.monday.com/v2 \
  -H "Content-Type: application/json" \
  -H "Authorization: $MONDAY_API_KEY" \
  -d '{
    "query": "mutation { create_item(board_id: YOUR_BOARD_ID, group_id: \"topics\", item_name: \"TASK_NAME\", column_values: \"{\\\"YOUR_DUE_DATE_COLUMN_ID\\\":{\\\"date\\\":\\\"DUE_DATE\\\"},\\\"YOUR_DESCRIPTION_COLUMN_ID\\\":\\\"DESCRIPTION\\\",\\\"YOUR_STATUS_COLUMN_ID\\\":{\\\"label\\\":\\\"Not Started\\\"}}\" ) { id } }"
  }'
```

## Task Creation Process

For each `- [ ]` action item in the journal:

1. **Extract task name**: Remove checkbox, clean up text
2. **Parse due date**: Use action-tracker's due date detection rules
3. **Detect action type**: Match patterns to dropdown values
4. **Detect priority**: Match urgency patterns
5. **Build column_values JSON**: Include all mapped fields
6. **Execute mutation**: Call Monday API
7. **Link task to journal**: Add Monday item ID as HTML comment to the journal line
8. **Report result**: Show task name and Monday item ID

## Monday ID Linking

After creating a Monday task, append the item ID to the journal line as an HTML comment (invisible in Obsidian preview mode):

**Before:**
```markdown
- [ ] Follow up with [[Sarah]] about API review by Wednesday
```

**After:**
```markdown
- [ ] Follow up with [[Sarah]] about API review by Wednesday <!-- monday:11047155094 -->
```

This enables bi-directional sync between journals and Monday.

## Bi-directional Status Sync

Sync status changes between journals and Monday in both directions.

### Status Mapping

| Journal Marker | Monday Status |
|----------------|---------------|
| `- [ ]` | Not Started |
| `- [~]` | In Progress |
| `- [!]` | Stuck |
| `- [x]` | Done |

### Journal → Monday Sync

When processing a journal, check each action item for a Monday ID and sync status:

```
For each action item with <!-- monday:ID -->:
  1. Read current marker: [ ], [~], [!], or [x]
  2. Map to Monday status
  3. If status differs from Monday, update Monday:
     
     mutation {
       change_column_value(
         board_id: YOUR_BOARD_ID,
         item_id: ITEM_ID,
         column_id: "YOUR_STATUS_COLUMN_ID",
         value: "{\"label\":\"STATUS_LABEL\"}"
       ) { id }
     }
```

### Monday → Journal Sync

When user says "sync from monday":

```
1. Query all items from board YOUR_BOARD_ID
2. For each item:
   a. Get item ID and current status
   b. Search all journals for <!-- monday:ID -->
   c. If found, compare statuses
   d. If different, update journal marker:
      - "Done" → change to [x]
      - "In Progress" → change to [~]
      - "Stuck" → change to [!]
      - "Not Started" → change to [ ]
3. Report all changes made
```

## GraphQL for Status Updates

**Update to Done:**
```graphql
mutation {
  change_column_value(board_id: YOUR_BOARD_ID, item_id: ITEM_ID, 
    column_id: "YOUR_STATUS_COLUMN_ID", value: "{\"label\":\"Done\"}") { id }
}
```

**Update to In Progress:**
```graphql
mutation {
  change_column_value(board_id: YOUR_BOARD_ID, item_id: ITEM_ID, 
    column_id: "YOUR_STATUS_COLUMN_ID", value: "{\"label\":\"In Progress\"}") { id }
}
```

**Update to Stuck:**
```graphql
mutation {
  change_column_value(board_id: YOUR_BOARD_ID, item_id: ITEM_ID, 
    column_id: "YOUR_STATUS_COLUMN_ID", value: "{\"label\":\"Stuck\"}") { id }
}
```

**Update to Not Started:**
```graphql
mutation {
  change_column_value(board_id: YOUR_BOARD_ID, item_id: ITEM_ID, 
    column_id: "YOUR_STATUS_COLUMN_ID", value: "{\"label\":\"Not Started\"}") { id }
}
```

## Sync Output Examples

**After syncing to Monday:**
```
Synced to Monday:
- "Check with Jordan C..." → Done (was: Not Started)
- "Follow up on Q1 strategy decision..." → In Progress (was: Not Started)
```

**After syncing from Monday:**
```
Synced from Monday:
- Journal/2026-01-19.md: "Talk with Alex H..." → [~] In Progress
- Journal/2026-01-19.md: "Give feedback to Sam..." → [x] Done
```

## Error Handling

If Monday API fails:
1. Log the error
2. Continue processing other items
3. Report failed tasks at the end:

> "Monday API error for 'Follow up with Sarah': Rate limit exceeded
> Task not created - add manually or retry later."

If API key missing:
> "Monday integration unavailable (no API key in .env).
> Tasks to create manually:
> - Follow up with Sarah about API review (due: Wed Jan 22)"

## User Commands

| Command | Action |
|---------|--------|
| "sync to monday" | Push journal action items to Monday |
| "sync from monday" | Pull Monday status changes into journals |
| "sync tasks" / "full sync" | Bi-directional sync (both directions) |
