---
name: performance-ratings
description: Manage performance and potential ratings for direct reports with 9-box grid visualization
triggers:
  - "show team ratings"
  - "show rating"
  - "update rating"
  - "rating history"
dependencies: [journal-processor, dossier-updater]
version: 1.0.0
---

# Performance & Potential Rating System

Manage performance and potential ratings for direct reports (team-members/) with evidence-based updates and 9-box grid visualization.

## Rating System

Each person in `team-members/` has two scores (1-5):
- **Performance**: How well they deliver in their current role
- **Potential**: Their capacity for growth and taking on more

Default scores are **3/3** (Good performance / Consistent potential) when no evidence exists.

## Rating Scale Summary

| Score | Performance | Potential |
|-------|-------------|-----------|
| 1 | Underperforming | Limited potential |
| 2 | Inconsistent | Low potential |
| 3 | Good (default) | Consistent (default) |
| 4 | Strong | Strong potential |
| 5 | Exceptional | Exceptional potential |

## Updating Ratings from Journal Entries

When processing journal entries, look for rating-relevant signals:

### Performance Indicators

| Signal | Direction | Example |
|--------|-----------|---------|
| Missed deadline, needs re-work | ↓ | "Sarah's PR needed significant fixes" |
| Delivered on time, quality work | → or ↑ | "Sarah shipped the feature ahead of schedule" |
| Exceeded expectations, high impact | ↑ | "Sarah's solution saved us 2 weeks" |
| Required significant support | ↓ | "Had to step in to help Sarah finish" |
| Showed ownership, drove results | ↑ | "Sarah proactively fixed the bug before it hit prod" |

### Potential Indicators

| Signal | Direction | Example |
|--------|-----------|---------|
| Struggled with new challenge | ↓ | "Sarah resistant to learning the new framework" |
| Adapted quickly, learned fast | ↑ | "Sarah picked up Go in a week" |
| Took on stretch assignment | ↑ | "Sarah volunteered to lead the migration" |
| Needed hand-holding on complexity | ↓ | "Had to break down the task into smaller pieces for Sarah" |
| Influenced others, drove change | ↑ | "Sarah convinced the team to adopt better testing practices" |

## Rating Update Process

1. **Detect signals** in journal entry mentioning a person
2. **Assess impact** on performance and/or potential
3. **Propose adjustment** (if warranted):
   > "Based on today's entry, Sarah delivered ahead of schedule with high quality. 
   > This suggests strong performance. Update Performance from 3 → 4?"
4. **Wait for confirmation** before changing
5. **Log the change** in Rating History table with date and reason

## Rating Update Rules

- **Never change by more than 1 point** from a single entry
- **Require multiple signals** over time for score changes
- **Always ask before updating** - show the evidence and proposed change
- **Log every change** with date and reason in the Rating History table
- **Consider recency** - recent evidence weighs more than old

## Example Rating Update Flow

Journal entry contains:
> "1:1 with Sarah: She shipped the API integration 3 days early and the code quality was excellent. Other teams are asking her for advice on the pattern she used."

Processing:
```
Detected rating-relevant signals for Sarah:
- "shipped 3 days early" → Performance signal (exceeds expectations)
- "code quality was excellent" → Performance signal (quality delivery)
- "other teams asking for advice" → Potential signal (influence beyond scope)

Current rating: Performance 3, Potential 3

Suggested updates:
- Performance: 3 → 4 (Strong) - consistent delivery above expectations
- Potential: 3 → 4 (Strong) - showing influence beyond immediate scope

Update Sarah's ratings? [yes/no/just performance/just potential]
```

## Viewing Team Ratings

When user says "show team ratings", display a 9-box grid:

```
                        POTENTIAL
              Low (1-2)    Mid (3)    High (4-5)
           ┌───────────┬───────────┬───────────┐
High (4-5) │           │           │ ⭐ Sarah  │
           │           │           │           │
PERFORMANCE├───────────┼───────────┼───────────┤
Mid (3)    │           │ Mike      │           │
           │           │ Alex      │           │
           ├───────────┼───────────┼───────────┤
Low (1-2)  │           │           │           │
           │           │           │           │
           └───────────┴───────────┴───────────┘
```

## Viewing Individual Ratings

When user says "show [name]'s rating":

```
Sarah's Performance & Potential Rating

Performance: 4 - Strong ●●●●○
Potential: 4 - Strong ●●●●○

Rating History:
  | Date       | Change              | Reason                           |
  |------------|---------------------|----------------------------------|
  | 2026-01-20 | Performance 3 → 4   | Shipped 3 days early, excellent  |
  | 2026-01-20 | Potential 3 → 4     | Influencing beyond immediate scope|

Position in 9-box: High Performance, High Potential (Top Right)
```

## Manual Rating Override

User can manually set ratings:
> "Set Sarah's performance to 4"
> "Update Mike's potential to 2"

Always:
1. Confirm the change
2. Ask for a reason to log
3. Update the dossier frontmatter and Rating History table

## Dossier Integration

Ratings are stored in `team-members/{name}.md` frontmatter:

```yaml
---
name: Sarah Chen
performance: 4
potential: 4
---
```

And logged in the Rating History table within the file:

```markdown
## Rating History
| Date       | Change              | Reason                           |
|------------|---------------------|----------------------------------|
| 2026-01-20 | Performance 3 → 4   | Shipped ahead of schedule        |
| 2026-01-20 | Potential 3 → 4     | Influencing beyond scope         |
```

## User Commands

| Command | Action |
|---------|--------|
| "show team ratings" | Display 9-box grid of all direct reports |
| "show [name]'s rating" | Show detailed rating for a person |
| "update [name]'s rating" | Manually adjust performance/potential scores |
| "rating history for [name]" | Show rating changes over time |
