# Skills Directory

This directory contains modular skills for the Daily Journal Processor system.

## Architecture

The system uses a master-router pattern where `../AGENTS.md` (in the parent directory) discovers and orchestrates these sub-skills dynamically.

## Available Skills

### Core Skills (No Dependencies)

1. **journal-processor.md** - Find, parse, and structure daily journal entries
   - Triggers: "process my journal", "process journal for [date]"
   - Extracts entities, converts to wiki-links, outputs structured data

2. **action-tracker.md** - Track and surface open action items across journals
   - Triggers: "show my action items", "show overdue items", "clear action item"
   - Scans all journals for `[ ]`, `[~]`, `[!]` items with due date detection

3. **interview-processor.md** - Process interview notes and create scorecards
   - Triggers: "show candidates", "show scorecard"
   - Creates/updates `Candidates/*.md` files with structured scorecards

### Integration Skills

4. **dossier-updater.md** - Update People/Stakeholders/Projects/Goals dossiers
   - Dependencies: journal-processor
   - Auto-invoked during journal processing
   - Handles entity disambiguation (People vs Stakeholders)

5. **calendar-follow-up.md** - Automatically schedule follow-up reminders in Monday calendar board
   - Dependencies: None (invoked by journal-processor and week-ahead-prep)
   - Triggers: "schedule follow-up", "create calendar reminder", "schedule touchbase"
   - Auto-invoked during journal processing and week-ahead-prep
   - Creates Google Calendar events via Monday board integration with smart timing

6. **monday-integration.md** - Create and sync Monday.com tasks
   - Dependencies: action-tracker
   - Triggers: "sync to monday", "sync from monday", "sync tasks"
   - Bi-directional status sync with HTML comment linking

### Advanced Features

7. **performance-ratings.md** - Manage 9-box performance/potential ratings
   - Dependencies: journal-processor, dossier-updater
   - Triggers: "show team ratings", "show [name]'s rating"
   - Detects rating signals and proposes evidence-based updates

8. **performance-review-summary.md** - Generate comprehensive review cycle summaries
   - Dependencies: performance-ratings, dossier-updater, journal-processor
   - Triggers: "performance review summary", "january review", "june review"
   - Creates cycle summaries with trends, insights, and recommendations

9. **relationship-tracker.md** - Track trust battery and advocacy status
   - Dependencies: journal-processor, dossier-updater
   - Triggers: "show stakeholder health", "show relationship"
   - Visualizes relationships on 3x3 grid

### Executive & Planning Skills

10. **week-ahead-prep.md** - Strategic week planning briefing
    - Dependencies: action-tracker, dossier-updater, calendar-follow-up
   - Triggers: "prep for the week", "week ahead", "plan my week", "sunday prep"
   - Comprehensive week planning with calendar, team, stakeholders, priorities, and risks

11. **win-tracker.md** - Detect team wins and draft recognition messages
    - Dependencies: journal-processor, dossier-updater
    - Triggers: "show team wins", "weekly wins", "recognition report", "celebrate wins"
    - Scans journals for team achievements and generates recognition messages

12. **staff-meeting.md** - Chief-of-staff executive briefing
    - Dependencies: action-tracker, dossier-updater, week-ahead-prep
    - Triggers: "staff meeting", "chief of staff briefing", "executive briefing", "staff sync"
    - Interactive comprehensive briefing across all organizational dimensions

## How It Works

### 1. Skill Discovery

When you invoke the master skill (`../AGENTS.md`), it:
1. Scans this directory for `*.md` files
2. Parses frontmatter from each file (name, triggers, dependencies)
3. Builds an in-memory skill registry
4. Routes your command to the appropriate skill(s)

### 2. Dependency Resolution

For complex commands like "process my journal", multiple skills execute in order:

```
journal-processor (parse entry)
  ↓
dossier-updater (update dossiers)
  ↓
action-tracker (extract action items)
  ↓
calendar-follow-up (create calendar events for follow-ups)
  ↓
performance-ratings (check for rating signals)
  ↓
relationship-tracker (check for relationship signals)
  ↓
interview-processor (if ## Interviews section exists)
  ↓
monday-integration (create tasks from action items)
```

### 3. Data Passing

Skills communicate via structured JSON objects:

```json
{
  "date": "2026-01-25",
  "items": [...],
  "entities": {
    "people": ["Sarah"],
    "stakeholders": ["Mike"],
    "projects": ["Project Alpha"]
  }
}
```

## Adding New Skills

To add a new skill:

1. Create `new-skill-name.md` in this directory
2. Add frontmatter:
   ```yaml
   ---
   name: new-skill
   description: What this skill does
   triggers:
     - "command pattern 1"
     - "command pattern 2"
   dependencies: [other-skill-names]
   version: 1.0.0
   ---
   ```
3. Write your skill logic and documentation
4. **No changes needed to master router** - discovery is automatic!

## Testing Individual Skills

You can load and test skills independently:

```
"Load the journal-processor skill and parse my journal"
"Load the performance-ratings skill and show team ratings"
"Load the relationship-tracker skill and show stakeholder health"
```

## Migration from v1.0

The original monolithic `SKILL.md` (1214 lines) has been refactored into:
- 11 focused skills (ranging from 150-700 lines each)
- 1 master router with dynamic discovery
- Total: ~3500+ lines, but highly maintainable and extensible

### Benefits

- **Maintainability**: Each skill is self-contained and focused
- **Testability**: Skills can be tested independently
- **Extensibility**: Add new skills without modifying existing code
- **Clarity**: Dependencies and data flows are explicit
- **Reusability**: Skills can be composed into different workflows
- **Discoverability**: Dynamic scanning means no hardcoded lists

### Preserved Functionality

All original functionality is preserved:
- ✅ Journal processing with entity extraction
- ✅ Dossier updates (People/Stakeholders/Projects/Goals)
- ✅ Performance & potential ratings with 9-box grid
- ✅ Performance review cycle summaries (January/June)
- ✅ Relationship health tracking (trust/advocacy)
- ✅ Monday.com task creation and bi-directional sync
- ✅ Action item tracking with overdue detection
- ✅ Interview processing with scorecards
- ✅ All user commands and workflows

## Architecture Diagram

```
User Command: "process my journal"
        ↓
../AGENTS.md (Master Router)
        ↓
   [Skill Discovery]
   Scan skills/*.md
        ↓
   [Command Routing]
   Match "process my journal" to triggers
        ↓
   [Dependency Resolution]
   Build execution order
        ↓
   [Execute Pipeline]
        ↓
   ┌─────────────────────┐
   │ journal-processor   │ Parse journal entry
   └──────────┬──────────┘
              ↓ (structured data)
   ┌─────────────────────┐
   │ dossier-updater     │ Update dossiers
   └──────────┬──────────┘
              ↓ (entity updates)
   ┌─────────────────────┐
   │ action-tracker      │ Extract action items
   └──────────┬──────────┘
              ↓ (action items list)
   ┌─────────────────────┐
   │ calendar-follow-up  │ Create calendar events for follow-ups
   └──────────┬──────────┘
              ↓ (calendar events created)
   ┌─────────────────────┐
   │ performance-ratings │ Check for rating signals
   └──────────┬──────────┘
              ↓
   ┌─────────────────────┐
   │ relationship-tracker│ Check for relationship signals
   └──────────┬──────────┘
              ↓
   ┌─────────────────────┐
   │ interview-processor │ Process interviews (if present)
   └──────────┬──────────┘
              ↓
   ┌─────────────────────┐
   │ monday-integration  │ Create Monday tasks
   └─────────────────────┘
              ↓
        [Summary Report]
```

## Version History

- **v2.3.0** (2026-02-01): Added calendar-follow-up skill for automatic calendar event creation
  - Auto-invoked during journal processing and week-ahead-prep
  - Creates Google Calendar events via Monday board integration
  - Smart timing based on context (1:1s = 7 days, stakeholders = 21 days)
  - Duplicate prevention to avoid over-scheduling
- **v2.2.0** (2026-01-28): Added performance-review-summary skill for cycle summaries
- **v2.1.0** (2026-01-27): Added staff-meeting skill for executive briefings
- **v2.0.0** (2026-01-25): Refactored from monolithic to modular architecture
- **v1.0.0** (2025-12-09): Original monolithic implementation
