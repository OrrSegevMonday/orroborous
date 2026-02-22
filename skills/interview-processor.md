---
name: interview-processor
description: Process interview notes from journals and create structured candidate scorecards
triggers:
  - "show candidates"
  - "show scorecard"
  - "update recommendation"
dependencies: [journal-processor]
version: 1.0.0
---

# Interview Processor

Process interview notes from journals and create structured candidate summaries with scorecards.

## Input Format

In daily journal under `## Interviews`:

```markdown
## Interviews
### Sarah Chen - Senior Product Analyst - Management Interview
- Strong motivation, wants to grow into leadership
- Good communication, clear explanations
- Collaborative mindset, gave examples of cross-team work
- Concern: Needs more experience with stakeholder management

### David Kim - Senior Product Analyst - SQL Assessment
- Clean CTEs, good structure
- Understands query optimization
- Rating: Analytics-SQL 4
```

## Interview Types

| Type | Focus Areas | Scorecard Categories |
|------|-------------|---------------------|
| Management Interview | Soft skills, motivation, fit | Fit - Impact-driven, Ownership, Fit-additional |
| SQL Assessment | Technical SQL skills | Analytics - SQL |
| Case Study | Business thinking | Business - Common sense, Business orientation |
| Technical Deep Dive | All technical skills | All Analytics categories |
| Custom (specify in title) | As described | As mentioned |

**Important:** Only score categories explicitly assessed in the interview. Leave others as 0.

## Output Format (Candidates/{Name}.md)

```markdown
---
name: Sarah Chen
role: Senior Product Analyst
status: in-process
recommendation: 3
last_interview: 2026-01-20
---

# Sarah Chen

**Role**: Senior Product Analyst
**Recommendation**: 3 - Yes
**Status**: In Process

---

## Pros
- Strong motivation, wants to grow into leadership
- Good communication, clear explanations
- Collaborative mindset

---

## Concerns
- Needs more experience with stakeholder management

---

## Further Review
- 

---

## Scorecard

| Category | Score | Notes |
|----------|-------|-------|
| Analytics - SQL | 0 | |
| Analytics - Data expertise | 0 | |
| Business - Common sense | 0 | |
| Business orientation | 0 | |
| Fit - Impact-driven | 4 | Strong motivation |
| Fit - Ownership & independence | 3 | |
| Analytics - additional | 0 | |
| Business - additional | 0 | |
| Tool building & user focus | 0 | |
| Hypothesis evaluation | 0 | |
| Fit - additional | 4 | Collaborative |

---

## Interview Log
- **2026-01-20** (Management Interview): Strong motivation, collaborative mindset...
```

## Scorecard Categories (Senior Product Analyst)

| Category | What to Look For |
|----------|------------------|
| **Analytics - SQL** | Structured code, best practices, data modeling opinions |
| **Analytics - Data expertise** | Smart, identifies nuanced data dynamics |
| **Business - Common sense** | Rational, structured, practical, seeks validation |
| **Business orientation** | Clear communication, considers audience, data storytelling |
| **Fit - Impact-driven** | Owns full problem, results-oriented, shapes agenda |
| **Fit - Ownership & independence** | Plans complex projects, high commitment ("Hodor" levels) |
| **Analytics - additional** | Other tools, attention to detail, statistics |
| **Business - additional** | Domain knowledge, full picture thinking |
| **Tool building & user focus** | (Supporting) |
| **Hypothesis evaluation** | (Supporting) |
| **Fit - additional** | Quick delivery, collaborative, resilient |

**Rating Scale:** 1-5 (0 = not assessed)

**Recommendation Scale:** 1 = Strong No, 2 = No, 3 = Yes, 4 = Strong Yes

## Processing Logic

When processing journal with `## Interviews`:

1. **Parse entry**: `### {Name} - {Role} - {Interview Type}`
2. **Determine focus** based on interview type:
   - Management Interview → Fit categories only
   - SQL Assessment → Analytics-SQL only
   - Case Study → Business categories
   - Technical Deep Dive → All Analytics
3. **Check for existing file**: `Candidates/{Name}.md`
4. **If new candidate**:
   - Create from `Templates/Candidate.md`
   - Extract pros (positive statements)
   - Extract concerns (negative statements, "concern:", "needs")
   - Score only categories relevant to interview type
5. **If existing candidate**:
   - Append to Interview Log with type
   - Update scores for newly assessed categories
   - Keep existing scores unchanged
6. **Prompt for recommendation** if multiple interviews completed

**Key Rule:** Do NOT infer technical scores from management interviews. Only score what's explicitly assessed.

## Extracting Pros vs Concerns

| Indicators | Category |
|------------|----------|
| Positive adjectives, achievements, strengths | Pros |
| "Concern:", "needs", "struggled", "weak", negatives | Concerns |
| Questions, uncertainties, "check", "verify" | Further Review |

## Output Summary

After processing interviews:

```
Processed interviews from 2026-01-20:

Created:
- [[Candidates/Sarah Chen]] - Management Interview
  - Pros: 3 items added
  - Concerns: 1 item added
  - Scores: Fit-Impact 4, Fit-additional 4

Updated:
- [[Candidates/David Kim]] - SQL Assessment
  - New score: Analytics-SQL 4
  - Interview log updated
```

## Viewing Candidates

When user says "show candidates":

```
Candidates (3 total):

IN PROCESS:
- Sarah Chen (Senior Product Analyst) - Rec: 3 (Yes)
  Last: Management Interview on Jan 20
- David Kim (Senior Product Analyst) - Rec: 0 (Pending)
  Last: SQL Assessment on Jan 20

DECIDED:
- Alex Wong (Senior Product Analyst) - Rec: 4 (Strong Yes) ✓
```

## Manual Recommendation Update

When user says "update [name]'s recommendation to [1-4]":

1. Confirm the change
2. Update `recommendation` in frontmatter
3. Update display text in file
4. Confirm: "Updated [Name]'s recommendation to [N] ([Label])"

## User Commands

| Command | Action |
|---------|--------|
| "show candidates" | List all candidates with status and recommendation |
| "show [name]'s scorecard" | Display full scorecard for a candidate |
| "update [name]'s recommendation" | Set recommendation (1-4) for a candidate |
