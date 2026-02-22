---
name: performance-review-summary
description: Generate comprehensive performance review cycle summaries for direct reports. Use when preparing for review cycles, writing performance summaries, or when the user asks for "review summary", "cycle summary", "performance review", or mentions January/June review periods.
triggers:
  - "performance review summary"
  - "review cycle summary"
  - "review summary"
  - "january review"
  - "june review"
  - "review period"
dependencies: [performance-ratings, dossier-updater, journal-processor]
version: 1.0.0
---

# Performance Review Summary

Generate comprehensive summaries of performance review cycles (January and June) for all direct reports in `team-members/`.

## Review Cycles

monday.com uses semi-annual review cycles:
- **January cycle**: Covers July 1 - December 31 (H2 of previous year)
- **June cycle**: Covers January 1 - June 30 (H1 of current year)

## When to Use

Run this skill when:
- Preparing for upcoming review calibration meetings
- Writing formal performance reviews
- Reflecting on team performance trends
- User says "review summary", "performance review", or mentions cycle months

## Summary Generation Process

### 1. Determine Current Cycle

Based on today's date, identify the relevant review period:

```
Current date: 2026-01-28

Logic:
- If date is in Jan-Mar: Most recent cycle is January (just completed)
- If date is in Apr-Jun: Upcoming cycle is June (in progress)
- If date is in Jul-Sep: Most recent cycle is June (just completed)
- If date is in Oct-Dec: Upcoming cycle is January (in progress)
```

### 2. Collect Team Data

For each person in `team-members/`:

1. **Read the file** and extract:
   - Current performance and potential ratings (from frontmatter)
   - Rating history entries (from Rating History table)
   - Recent log entries (from Log section)

2. **Filter for cycle period**:
   - Extract only rating changes within the cycle date range
   - Identify significant events from logs in that period

3. **Calculate metrics**:
   - Starting vs ending ratings
   - Number of rating changes
   - Trend direction (improving/declining/stable)

### 3. Generate Summary Report

Create a comprehensive report with these sections:

## Report Structure

### Executive Summary

```markdown
# Performance Review Summary - [Cycle Name] [Year]
Review Period: [Start Date] - [End Date]
Generated: [Today's Date]

## Team Overview
- Team size: [N] direct reports
- Average performance: [X.X]
- Average potential: [X.X]
- Rating changes: [N] updates during cycle
```

### Distribution Analysis

Show team distribution across the 9-box grid:

```
                        POTENTIAL
              Low (1-2)    Mid (3)    High (4-5)
           ┌───────────┬───────────┬───────────┐
High (4-5) │           │           │ ⭐ Name1  │ [N]
           │           │           │           │
PERFORMANCE├───────────┼───────────┼───────────┤
Mid (3)    │           │ Name2     │ Name3     │ [N]
           │           │           │           │
           ├───────────┼───────────┼───────────┤
Low (1-2)  │ Name4     │           │           │ [N]
           │           │           │           │
           └───────────┴───────────┴───────────┘
```

### Individual Summaries

For each person, provide:

```markdown
### [Person Name]
**Current Rating**: Performance [X], Potential [Y]
**Cycle Trend**: [Improving/Stable/Declining]

**Changes During Cycle**:
| Date | Change | Reason |
|------|--------|--------|
| [date] | Performance [old]→[new] | [reason] |
| [date] | Potential [old]→[new] | [reason] |

**Key Highlights**:
- [Extract 2-3 notable events from log entries during cycle]

**Summary**: 
[1-2 sentence performance summary based on rating and log data]
```

### Trends & Insights

Analyze patterns across the team:

```markdown
## Trends & Insights

**Performance Trends**:
- [N] reports improved performance
- [N] reports declined
- [N] remained stable

**Potential Trends**:
- [N] showing increased potential
- [N] growth concerns

**Notable Patterns**:
- [Identify any common themes from logs/ratings]
- [E.g., "Multiple reports struggling with new framework adoption"]
- [E.g., "Strong delivery on Project Alpha across team"]
```

### Recommendations

```markdown
## Recommendations

**High Performers (4-5 Performance)**:
- [Name]: [Specific recommendation, e.g., "Consider for tech lead role"]

**Development Focus (1-2 Performance)**:
- [Name]: [Specific action plan, e.g., "Needs closer supervision on quality"]

**Growth Opportunities**:
- [Identify team-wide development needs]
```

## User Commands

| Command | Action |
|---------|--------|
| "performance review summary" | Generate summary for most recent completed cycle |
| "january review summary" | Generate summary for January cycle |
| "june review summary" | Generate summary for June cycle |
| "review summary for [name]" | Generate individual summary for one person |

## Example Usage

**User**: "Generate the performance review summary"

**Assistant**:
1. Determines current date (Jan 28, 2026)
2. Identifies most recent cycle (January 2026 - covering Jul-Dec 2025)
3. Reads all `team-members/*.md` files
4. Filters rating changes between July 1 - December 31, 2025
5. Extracts log entries from that period
6. Generates comprehensive report with all sections

## Data Sources

### Primary: Rating History Table

From each `team-members/{name}.md` file:

```markdown
### Rating History
| Date | Performance | Potential | Reason |
|------|-------------|-----------|--------|
| 2025-11-15 | 3 → 4 | 3 | Shipped ahead of schedule |
| 2025-09-22 | - | 3 → 4 | Leading team initiatives |
```

### Secondary: Log Section

Extract relevant events:

```markdown
## Log
- **2025-11-15**: Delivered API integration 3 days early
- **2025-09-22**: Volunteered to lead migration project
```

### Frontmatter: Current State

```yaml
---
performance: 4
potential: 4
last_rating_update: 2025-11-15
---
```

## Cycle Date Ranges

Helper function to determine cycle dates:

```
function getCycleDates(cycleName, year):
  if cycleName == "January":
    start = "[year-1]-07-01"
    end = "[year-1]-12-31"
    review_month = "[year]-01"
  else if cycleName == "June":
    start = "[year]-01-01"
    end = "[year]-06-30"
    review_month = "[year]-06"
  
  return (start, end, review_month)
```

## Special Cases

### No Rating Changes in Cycle

If a person has no rating history entries during the cycle:

```markdown
### [Person Name]
**Current Rating**: Performance [X], Potential [Y]
**Cycle Trend**: Stable (no rating changes)

**Key Highlights**:
- [Extract notable log entries from cycle period]

**Summary**: 
Maintained consistent [performance level] performance throughout the cycle.
```

### New Hire During Cycle

If person's `started` date is within the cycle:

```markdown
### [Person Name] ⭐ NEW
**Current Rating**: Performance [X], Potential [Y]
**Joined**: [Start date]
**Time in role**: [N] months

**Initial Performance**:
[Focus on early indicators and onboarding success]
```

### Missing Data

If a person file has incomplete data:
- Default ratings: 3/3 (Good/Consistent)
- Note in summary: "*Rating data incomplete - using defaults*"

## Output Format Options

### Quick Summary (Default)

Concise report focusing on:
- Team overview
- 9-box grid
- Individual summaries (brief)
- Top 3 insights

### Detailed Summary

Full report with all sections including:
- Complete rating history tables
- All log entries from cycle
- Detailed recommendations

User can specify:
> "Generate detailed review summary"
> "Generate quick review summary"

## Integration with Other Skills

This skill builds on:

1. **performance-ratings.md**: Uses rating scales and history structure
2. **dossier-updater.md**: Reads dossier format from team-members/ files
3. **journal-processor.md**: References log entries format

## Calendar Integration Notes

When preparing summaries, consider:
- January reviews typically happen in late Jan/early Feb
- June reviews typically happen in late Jun/early Jul
- Plan to generate summaries 1-2 weeks before review meetings

## Example Output

```markdown
# Performance Review Summary - January 2026
Review Period: July 1, 2025 - December 31, 2025
Generated: January 28, 2026

## Team Overview
- Team size: 5 direct reports
- Average performance: 3.2
- Average potential: 3.4
- Rating changes: 8 updates during cycle

## Team Distribution

                        POTENTIAL
              Low (1-2)    Mid (3)    High (4-5)
           ┌───────────┬───────────┬───────────┐
High (4-5) │           │           │ ⭐ Casey  │ 1
           │           │           │           │
PERFORMANCE├───────────┼───────────┼───────────┤
Mid (3)    │           │ Sam       │ Alex      │ 4
           │           │ Dana      │ Jordan    │
           ├───────────┼───────────┼───────────┤
Low (1-2)  │           │           │           │ 0
           │           │           │           │
           └───────────┴───────────┴───────────┘

### Individual Summaries

#### Casey M ⭐
**Current Rating**: Performance 4, Potential 4
**Cycle Trend**: Improving

**Changes During Cycle**:
| Date | Change | Reason |
|------|--------|--------|
| 2025-11-15 | Performance 3→4 | Shipped ahead of schedule |
| 2025-09-22 | Potential 3→4 | Leading team initiatives |

**Key Highlights**:
- Delivered critical API integration 3 days early
- Volunteered to lead database migration
- Mentoring junior team members

**Summary**: 
Strong performance throughout H2 with consistent delivery above expectations and increasing leadership influence.

[... more individual summaries ...]

## Trends & Insights

**Performance Trends**:
- 2 reports improved performance during cycle
- 3 remained stable at good performance
- 0 declined

**Potential Trends**:
- 2 showing increased potential
- Team developing leadership capabilities

**Notable Patterns**:
- Strong delivery on MLS project across team
- Increased ownership and proactive problem-solving
- Need to continue developing technical depth

## Recommendations

**High Performers**:
- Casey M: Consider for senior role or tech lead responsibilities
- Alex K: Stretch project to develop strategic thinking

**Stable Performers**:
- Continue supporting growth through challenging assignments
- Regular 1:1s focused on career development

**Development Areas**:
- Team-wide: Deepen technical skills in new frameworks
- Focus on proactive communication with stakeholders
```

## Validation Checklist

Before finalizing summary:
- [ ] All team-members/*.md files read
- [ ] Rating changes filtered to correct date range
- [ ] Log entries reviewed for cycle period
- [ ] 9-box grid accurately reflects current ratings
- [ ] Individual summaries based on evidence
- [ ] Trends backed by data
- [ ] Recommendations are specific and actionable
