# Orroborous

An AI-powered chief-of-staff system for managers, built on [Obsidian](https://obsidian.md) and [Cursor](https://cursor.sh). It turns your daily journal into a living operational brain — tracking your team, stakeholders, projects, and commitments so you don't have to hold it all in your head.

Named after the ouroboros: the system feeds itself.

---

## What you can do

| Command | What happens |
|---|---|
| `"process my journal"` | Parses today's entry, updates dossiers, extracts action items, creates Monday tasks, schedules follow-ups |
| `"show my action items"` | Surfaces all open tasks across all journals, grouped by urgency |
| `"show team ratings"` | Displays 9-box performance/potential grid for your direct reports |
| `"show stakeholder health"` | Plots all stakeholders on a trust × advocacy grid |
| `"prep for the week"` | Generates a strategic week-ahead briefing: calendar, commitments, team attention needed, stakeholder maintenance, priorities, risks |
| `"staff meeting"` | Runs an interactive chief-of-staff briefing — reviews projects, team, action items, and relationship health section by section, updating files as you make decisions |
| `"weekly wins"` | Scans journals for team wins, drafts recognition messages |
| `"show candidates"` | Lists all interview candidates with status and recommendation scores |
| `"sync tasks"` | Bi-directional sync between journal action items and Monday.com |
| `"schedule follow-up with [person]"` | Creates a calendar reminder in Monday (syncs to Google Calendar) |
| `"performance review summary"` | Generates a full review cycle summary with 9-box, trends, and recommendations |

---

## How it's built

Orroborous has three layers:

```
┌─────────────────────────────────────────────┐
│  Obsidian Vault                             │
│  Markdown files, wiki-links, templates      │
│  Journal/, team-members/, Stakeholders/,    │
│  Projects/, Goals/, Candidates/             │
└────────────────────┬────────────────────────┘
                     │ reads / writes
┌────────────────────▼────────────────────────┐
│  Cursor AI IDE                              │
│  Reads your vault, executes skills,         │
│  calls external APIs                        │
└────────────────────┬────────────────────────┘
                     │ loads
┌────────────────────▼────────────────────────┐
│  Skill System                               │
│  skills.md  ←  master router               │
│  skills/*.md ← 12 modular skills           │
└─────────────────────────────────────────────┘
```

### Skill system

`skills.md` is a master orchestrator. On every invocation it:

1. **Discovers** all `skills/*.md` files by reading their YAML frontmatter
2. **Routes** your command to the matching skill(s) by fuzzy-matching triggers
3. **Resolves dependencies** via topological sort (e.g. `journal-processor` runs before `dossier-updater`)
4. **Executes** each skill in order, passing structured JSON between them
5. **Summarizes** what was done

Each skill is a self-contained markdown file with frontmatter metadata and plain-English instructions. New skills are auto-discovered — no changes to the router needed.

### The 12 skills

| Skill | What it does |
|---|---|
| `journal-processor` | Finds and parses daily journal entries, extracts entities and action items |
| `dossier-updater` | Updates team member, stakeholder, project, and goal dossiers from journal data |
| `action-tracker` | Tracks open action items across all journals with overdue detection |
| `monday-integration` | Creates Monday.com tasks from action items; bi-directional status sync |
| `calendar-follow-up` | Auto-schedules follow-up reminders in Google Calendar via Monday board |
| `performance-ratings` | Manages 9-box performance/potential ratings with evidence-based update proposals |
| `performance-review-summary` | Generates semi-annual review cycle summaries with team trends and recommendations |
| `relationship-tracker` | Tracks trust battery and advocacy status for stakeholders; visualizes on a grid |
| `week-ahead-prep` | Strategic week planning briefing across calendar, team, stakeholders, priorities, and risks |
| `staff-meeting` | Interactive executive briefing that reviews all dimensions and writes decisions back to files |
| `win-tracker` | Detects team wins in journals and drafts recognition messages |
| `interview-processor` | Processes interview notes into structured candidate scorecards |

### External integrations

- **Monday.com MCP** — task creation, status sync, and calendar events
- **Google Calendar** — via Monday's Google Calendar integration on a dedicated board
- **Cursor browser extension** — for reading your Google Calendar in week-ahead prep

---

## Setup

### Prerequisites

1. **[Obsidian](https://obsidian.md)** — free, available for macOS, Windows, Linux, iOS, Android
2. **[Cursor](https://cursor.sh)** — AI-powered IDE (free tier available)
3. A **Monday.com** account (for task sync and calendar features — optional but recommended)

### Steps

1. **Clone this repo**
   ```bash
   git clone https://github.com/OrrSegevPersonal/orroborous.git
   cd orroborous
   ```

2. **Open as an Obsidian vault**
   - Open Obsidian → "Open folder as vault" → select the `orroborous` folder
   - This gives you the file structure, wiki-linking, and template support

3. **Open in Cursor**
   - Open Cursor → File → Open Folder → select the same `orroborous` folder
   - Cursor reads the vault files directly

4. **Set up your environment**
   ```bash
   cp .env.example .env
   # Edit .env and add your Monday.com API key
   ```

5. **Configure Monday MCP** (optional — for task and calendar sync)

   Add to your Cursor `mcp.json`:
   ```json
   {
     "mcpServers": {
       "monday-mcp": {
         "url": "https://mcp.monday.com/mcp"
       }
     }
   }
   ```
   Then authenticate via Monday's OAuth flow when prompted.

6. **Create your first journal entry**

   In Obsidian, use the Daily Journal template to create `Journal/YYYY-MM-DD.md`. Write notes from your day in the standard format:
   ```markdown
   ## Meetings
   - 1:1 with [[Alex K]] — discussed Q1 goals, she's blocked on the API review

   ## Action Items
   - [ ] Follow up with [[Alex K]] on API review by Wednesday
   ```

7. **Start the AI**

   Open Cursor's chat panel (pointed at your vault) and type:
   ```
   process my journal
   ```

---

## Vault structure

```
orroborous/
├── skills.md              # Master skill orchestrator (start here)
├── skills/                # 12 modular skill definitions
├── Journal/               # Daily journal entries (gitignored)
├── team-members/          # Direct report dossiers (gitignored)
├── Stakeholders/          # Stakeholder dossiers (gitignored)
├── Projects/              # Project dossiers (gitignored)
├── Goals/                 # Personal OKRs (gitignored)
├── Candidates/            # Interview scorecards (gitignored)
├── Weekly Prep/           # Weekly planning reports (gitignored)
└── staff-meetings/        # Staff meeting reports (gitignored)
```

All personal data directories are gitignored. Only the skills (the "brain") live in version control.

---

## Customization

### Monday board IDs

The Monday-related skills reference placeholder values (`YOUR_BOARD_ID`, `YOUR_COLUMN_ID`, etc.). To use them:

1. Create a board in Monday.com for your action items
2. Create a second board for calendar events (with Google Calendar sync enabled)
3. Find the board IDs in the URL: `https://monday.com/boards/YOUR_BOARD_ID`
4. Find column IDs via Monday's API or developer tools
5. Replace the placeholders in `skills/monday-integration.md` and `skills/calendar-follow-up.md`

### Calendar URL

In `skills/week-ahead-prep.md` and `skills/staff-meeting.md`, replace `YOUR_CALENDAR_URL` with your Google Calendar embed URL.

### Adding new skills

1. Create `skills/my-new-skill.md`
2. Add frontmatter:
   ```yaml
   ---
   name: my-new-skill
   description: What this skill does and when to use it
   triggers:
     - "command that activates it"
   dependencies: []
   version: 1.0.0
   ---
   ```
3. Write the skill logic in the body
4. Done — the orchestrator discovers it automatically

---

## Privacy

Your journal entries, dossiers, and any personal data live in gitignored directories and never leave your machine. Only the skill definitions (which contain no personal data) are tracked in version control.

---

## License

MIT
