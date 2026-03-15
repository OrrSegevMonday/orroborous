---
name: onboarding
description: Interactive first-time setup -- identity, connectors, team, projects
triggers:
  - "setup"
  - "onboard me"
  - "first time"
  - "get started"
  - "initialize"
dependencies: []
auto_invoke: false
version: 1.0.0
---

# Onboarding

Interactive first-time setup for new Orroborous users. Walks through identity, connectors, team roster, and projects in a phased conversation.

This skill self-destructs after completion -- it deletes itself and removes the Bootstrap Check from AGENTS.md.

## Execution Model

Run one phase at a time. Wait for the user input before advancing. Never skip ahead without explicit approval.

---

## Phase 1 -- Identity

Ask the user:

1. **What is your first name?**
2. **What is your role?** (e.g. "Engineering Manager at Acme Corp")

Then update PERSONA.md:

- Replace every occurrence of `[YOUR_NAME]` with the name they gave (there are 2)
- In the Role and Identity section, update the first paragraph to reflect their role and context

Confirm:

> "Done. PERSONA.md now knows you as [name]. Moving to connectors."

---

## Phase 2 -- Connectors

Walk through each step sequentially. For each: show the exact snippet, ask "done?", wait for confirmation.

### Step 1 -- Monday.com MCP

Show:

    Add this to your Cursor mcp.json (Settings > MCP > Add):

    {
      "mcpServers": {
        "monday-mcp": {
          "url": "https://mcp.monday.com/mcp"
        }
      }
    }

    Cursor will prompt you to authenticate via Monday OAuth flow.

Ask: "Added and authenticated? (or skip)"

### Step 2 -- Monday API key

Show:

    1. Go to monday.com > your avatar > Developers > My Access Tokens
    2. Copy your API token
    3. Open .env in the vault root and set:

    MONDAY_API_KEY=your_token_here

Ask: "API key saved to .env? (or skip)"

### Step 3 -- Action Items board

This step is required for task sync to work.

Show:

    1. Go to monday.com and create a new board
    2. Name it "Action Items" (or whatever you prefer)
    3. Copy the board ID from the URL: monday.com/boards/YOUR_BOARD_ID
    4. Come back and paste the board ID here

When the user provides the board ID:

- Open `skills/monday-integration.md`
- Find the line containing `YOUR_BOARD_ID` in the Board Configuration table
- Replace `YOUR_BOARD_ID` with the provided ID

Confirm: "Board ID written to monday-integration.md."

Ask: "Want to set up the column IDs now too, or configure them later?" If now, walk through each column in the mapping table. Otherwise move on.

### Step 4 -- Calendar board (optional)

Show:

    For automatic follow-up reminders synced to Google Calendar:

    1. Create another Monday board (e.g. "Calendar Events")
    2. Enable the Google Calendar integration on it
    3. Paste the board ID here (or type "skip")

If provided:

- Open `skills/calendar-follow-up.md`
- Replace `YOUR_BOARD_ID` with the provided ID

### Step 5 -- Calendar URL (optional)

Show:

    For week-ahead prep and staff meetings, Orroborous can read your calendar.

    1. Go to Google Calendar > Settings > your calendar > "Integrate calendar"
    2. Copy the embed URL
    3. Paste it here (or type "skip")

If provided:

- Open `skills/week-ahead-prep.md` and `skills/staff-meeting.md`
- Replace `YOUR_CALENDAR_URL` with the provided URL

Confirm: "Connectors configured. Moving to team setup."

---

## Phase 3 -- Team roster (skippable)

Ask:

> "Want to add your team members now, or skip? They will be created automatically the first time they appear in a journal entry."

If **skip**: move to Phase 4.

If **now**, loop:

1. Ask for: name, role, start date, reports-to
2. Ask: "Initial performance/potential rating? (default: 3/3, or give two numbers like 4/3)"
3. Create `team-members/[Name].md` using the `Templates/Person.md` template:
   - Fill in the frontmatter fields: `role`, `reports_to`, `started`, `performance`, `potential`
   - Fill in the Overview section
   - Set `last_rating_update` to today
4. Confirm: "Created team-members/[Name].md"
5. Ask: "Add another, or done?"

When done, confirm count: "Added [N] team members."

---

## Phase 4 -- Projects (skippable)

Ask:

> "Want to add your active projects now, or skip? Like team members, they will be picked up automatically from journal entries."

If **skip**: move to Phase 5.

If **now**, loop:

1. Ask for: project name, status (active/on-hold), owner, start date
2. Create `Projects/[Name].md` using the `Templates/Project.md` template:
   - Fill in the frontmatter fields: `status`, `owner`, `started`
   - Fill in the Overview section
3. Confirm: "Created Projects/[Name].md"
4. Ask: "Add another, or done?"

When done, confirm count: "Added [N] projects."

---

## Phase 5 -- First journal entry

Ask:

> "Want to create today's journal entry? I will set up the file from the template."

If **yes**:

- Create `Journal/YYYY-MM-DD.md` using `Templates/Daily Journal.md`
- Fill in the `date` field in frontmatter and the heading with today's date
- Tell the user: "Created Journal/[date].md. Fill it in when you are ready, then say 'process my journal' to kick off the full pipeline."

If **skip**: move to Phase 6.

---

## Phase 6 -- Summary

Print a setup report:

    Setup complete. Here is what was configured:

    [x] Identity -- [name], [role]
    [x] Monday MCP -- connected / skipped
    [x] Monday API key -- saved / skipped
    [x] Action Items board -- [board_id] / skipped
    [ ] Calendar board -- skipped
    [ ] Calendar URL -- skipped
    [x] Team members -- [N] added / skipped
    [ ] Projects -- skipped
    [x] First journal -- created / skipped

    Try these first:
      "process my journal"  -- parse today's entry and update everything
      "show team ratings"   -- see your team's 9-box grid
      "prep for the week"   -- get your week-ahead briefing

Use [x] for completed steps and [ ] for skipped ones.

---

## Phase 7 -- Self-destruct

After the user acknowledges the summary:

1. **Delete this file**: remove `skills/onboarding.md`
2. **Remove the Bootstrap Check from AGENTS.md**: delete the entire `## Bootstrap Check` section (from the heading through to the next `##` heading, exclusive)

Do both silently. Do not ask for confirmation -- onboarding is done.
