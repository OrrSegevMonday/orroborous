---
name: win-tracker
description: Detect team wins and draft recognition messages for celebration
triggers:
  - "show team wins"
  - "weekly wins"
  - "recognition report"
  - "celebrate wins"
dependencies: [journal-processor, dossier-updater]
version: 1.0.0
---

# Win Tracker (Recognition & Celebration)

Automatically detect team wins from journals, track them in People dossiers, and draft recognition messages weekly.

## Quick Start

When you say "weekly wins" or "recognition report", this skill scans your journals for team member achievements and drafts recognition messages.

## Win Detection Signals

Scan journals for these performance signals when team members (team-members/) are mentioned:

| Signal Type | Example Phrases | Recognition Level |
|-------------|-----------------|-------------------|
| **Early delivery** | "shipped ahead of schedule", "delivered 3 days early", "finished early" | Public (team-wide) |
| **Quality work** | "excellent", "high quality", "thorough", "impressive", "well done" | Public or Private |
| **Exceeded expectations** | "went above and beyond", "exceeded", "outstanding", "exceptional" | Public |
| **Ownership shown** | "took initiative", "proactively", "drove", "stepped up" | Private (1:1) |
| **Growth moment** | "learned quickly", "adapted fast", "picked up", "improved" | Private (1:1) |
| **Impact delivered** | "saved us time", "unblocked the team", "solved", "helped", "enabled" | Public |
| **Positive feedback** | "praised", "told them good job", "appreciated", "thanked" | Track (already given) |

## Processing Steps

### Step 1: Determine Time Range

Default: Scan journals from past week (last 7 days)

Calculate dates:
- End date: Today
- Start date: Today - 7 days
- Format: "Week of Jan 20-26"

### Step 2: Scan Journals for Win Signals

For each journal in date range:

1. **Identify People mentions**
   - Scan for `[[team-members/Name]]` wiki-links
   - Scan for name mentions in text (cross-reference with team-members/ directory)

2. **Extract context around each mention**
   - Get full sentence or bullet point containing the mention
   - Look for win signal phrases (see table above)
   - Capture 1-2 sentences of context

3. **Categorize signal type**
   - Match phrase to signal type (early delivery, quality work, etc.)
   - Determine recognition level (public vs private)
   - Skip if already recognized (see Step 4)

4. **Check if already recognized**
   - Look in person's "## Feedback Given" section
   - If similar win recognized in past 14 days → skip to avoid duplication
   - If first time or different win → include

### Step 3: Draft Recognition Messages

For each detected win, generate an appropriate recognition message:

**Message Structure:**

```
FOR [NAME] ([Recognition Level] - [Channel/Timing]):
"[Draft message]"

Style: [Message tone]
Why [public/private]: [Reasoning]
[Optional: Channel: #team-channel or all-hands]
Timing: [When to deliver]
```

**Message Drafting Guidelines:**

| Win Type | Message Tone | Public vs Private |
|----------|--------------|-------------------|
| Early delivery, Impact | Appreciative, specific | Public |
| Exceeded expectations | Celebratory, proud | Public |
| Quality work (junior/new) | Encouraging, developmental | Private |
| Quality work (senior) | Appreciative, reinforcing | Public or Private |
| Ownership shown | Appreciative, reinforcing | Private |
| Growth moment | Encouraging, supportive | Private |

**Personalization Factors:**
- **Tenure**: For new team members (< 3 months), emphasize growth and learning
- **Level**: For senior team members, emphasize impact and leadership
- **Context**: Reference specific details from the journal entry
- **Tone**: Match your typical communication style - professional but warm

**Example Message Templates:**

**Public (Early Delivery/Impact):**
```
"Shout-out to [Name] for [specific achievement]! [Impact statement]. 
[Optional: What this means for the team]. Thanks for [action]! 🙌"
```

**Private (Growth Moment):**
```
"[Name], I wanted to highlight something I noticed: [specific achievement]. 
[Why it's notable]. [Encouraging statement]. Keep up the great work!"
```

**Private (Ownership):**
```
"I appreciate how [action/behavior]. [Specific example]. 
This shows [value - ownership/initiative/proactivity]. Thanks for [action]."
```

### Step 4: Update People Dossiers

For each detected win:

1. **Append to Wins section**
   - Open `team-members/[Name].md`
   - Find `## Wins` section
   - Add new line: `- **[Date]**: [Win description]`
   - Keep format consistent with existing entries

2. **Check if section exists**
   - If no "## Wins" section exists, add it after "## Development Areas"
   - Use same format as other team-members files

### Step 5: Track Recognition Status

For each team member:

1. **Find last recognition given**
   - Check "## Feedback Given" section
   - Look for most recent positive feedback date
   - Calculate days since last recognition

2. **Flag recognition gaps**
   - If > 14 days since last recognition → flag with ⚠️
   - If never recognized → flag with ⚠️
   - Suggest timing for recognition

## Output Format

Generate comprehensive weekly wins report:

```
Team Wins Report: Week of [Date Range]

━━━ WINS DETECTED ━━━

[[team-members/Name1]] ([Role]):
✓ [Win 1 description] ([Date])
✓ [Win 2 description] ([Date])
  Win type: [Signal types]

[[team-members/Name2]] ([Role]):
✓ [Win description] ([Date])
  Win type: [Signal type]

[Repeat for each person with wins]

━━━ DRAFTED RECOGNITION MESSAGES ━━━

FOR [NAME1] ([Public/Private] - [Channel/Timing]):
"[Full drafted message]"

Style: [Tone description]
Why [public/private]: [Reasoning]
[If public] Channel: [Suggested channel]
Timing: [When to deliver]

---

FOR [NAME2] ([Public/Private] - [Channel/Timing]):
"[Full drafted message]"

Style: [Tone description]
Why [public/private]: [Reasoning]
Timing: [When to deliver]

---

[Repeat for each recognition message]

━━━ DOSSIER UPDATES ━━━

Updated Wins sections:
- [[team-members/Name1.md]] - added [N] win(s)
- [[team-members/Name2.md]] - added [N] win(s)

━━━ RECOGNITION TRACKING ━━━

Last recognition given:
- [Name1]: [Date] ([Context]) - [N] days ago
- [Name2]: [Date] ([Context]) - [N] days ago
- [Name3]: No recent recognition (last: [Date or "Never"]) ⚠️

⚠️ = Recognition due - these team members haven't been recognized in 2+ weeks

━━━ RECOGNITION SUGGESTIONS ━━━

IMMEDIATE:
- [Action for high-priority recognition]

THIS WEEK:
- [Action for recognition to deliver this week]

WATCH LIST:
- [Team members to continue tracking]
```

## Win Detection Examples

**Example 1: Early Delivery (Public)**

Journal entry:
> "1:1 with [[Sarah]] - she shipped the API integration 3 days ahead of schedule and the code quality was excellent. Other teams are asking her for advice."

Detected:
- Win type: Early delivery + Impact
- Recognition level: Public
- Message: "Shout-out to Sarah for shipping the API integration 3 days early! The code quality is excellent, and other teams are already asking for her guidance. This is exactly the kind of work that raises the bar. Thanks for the strong execution! 🙌"

**Example 2: Growth Moment (Private)**

Journal entry:
> "[[Alex K]]'s notebooks are very thorough with lots of sanity checks - excellent attention to detail for someone in their 3rd week."

Detected:
- Win type: Quality work + Growth moment
- Recognition level: Private (early tenure)
- Message: "Alex, I wanted to highlight something I noticed in your recent work: your notebooks are incredibly thorough with excellent sanity checks. For someone in their 3rd week, this attention to detail is really impressive. Keep up the great work on building solid analytical foundations."

**Example 3: Ownership (Private)**

Journal entry:
> "[[Mike]] proactively identified the bug before it hit production and fixed it immediately. Good ownership."

Detected:
- Win type: Ownership + Impact
- Recognition level: Private
- Message: "I appreciate how you proactively caught that bug before it hit production. This shows great ownership and attention to quality. Thanks for staying on top of it."

## False Positive Prevention

**DO NOT detect as win if:**

1. **Context is negative**
   - "Sarah's code quality was excellent, BUT it was late"
   - "Mike took initiative, HOWEVER he didn't coordinate with the team"

2. **Hypothetical or future-focused**
   - "Sarah SHOULD ship early"
   - "HOPING Mike will take more ownership"

3. **Question or concern**
   - "Is Alex's work thorough ENOUGH?"
   - "Wondering IF Sarah can maintain quality"

4. **Already recognized**
   - Check Feedback Given section for recent similar recognition

**Detection Rules:**
- Require positive signal phrases (see table)
- Look for past tense or present tense (not future/conditional)
- Require context to be clearly positive
- Skip if already recognized for same thing within 14 days

## Edge Cases

### No Wins Detected
```
━━━ WINS DETECTED ━━━

No significant wins detected in journals from [date range].

This could mean:
- Wins weren't explicitly called out in journals
- Team is in execution mode (wins will show up later)
- Consider manual review of team's work this week

Try running again next week or check specific journal entries for team mentions.
```

### Recognition Already Given
If all detected wins were already recognized:
```
━━━ WINS DETECTED ━━━

[List wins as usual]

━━━ RECOGNITION TRACKING ━━━

All detected wins have already been recognized! ✓

No new recognition messages drafted (all wins were previously acknowledged).
```

### Team Member Not in team-members/
If win is detected for someone not in team-members/ directory:
```
⚠️ Detected win for "[Name]" but no dossier found in team-members/
   
   This person might be:
   - A stakeholder (should be in Stakeholders/)
   - Not a direct report
   - Missing dossier file

   Win: [Description from journal]
```

## User Commands

| Command | Action |
|---------|--------|
| "show team wins" | Display wins from current week |
| "weekly wins" | Same as above (typically run Friday or Sunday) |
| "recognition report" | Full report with drafted messages |
| "celebrate wins" | Same as "recognition report" |

## Integration with Other Skills

**Depends on:**
- `journal-processor` - for structured journal data and entity extraction
- `dossier-updater` - for understanding team-members/ structure

**Data it reads:**
- `Journal/*.md` - scans past week for win signals
- `team-members/*.md` - reads Feedback Given section, updates Wins section

**Data it writes:**
- `team-members/*.md` - appends to "## Wins" section

**Coordinates with:**
- `performance-ratings` - wins are evidence for performance rating updates
- `week-ahead-prep` - surfaces recent wins for 1:1 discussion topics
