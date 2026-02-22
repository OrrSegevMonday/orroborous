---
name: relationship-tracker
description: Track Trust Battery and Advocacy Status changes for stakeholders with relationship health visualization
triggers:
  - "show relationship"
  - "show stakeholder health"
  - "show relationship chart"
  - "set trust"
  - "set advocacy"
  - "show at-risk relationships"
dependencies: [journal-processor, dossier-updater]
version: 1.0.0
---

# Relationship Health Tracking

Track Trust Battery and Advocacy Status changes for stakeholders over time with health visualization.

## Scales

### Trust Battery (3 levels)

| Level | Description |
|-------|-------------|
| Low | Damaged trust, skepticism, friction in interactions |
| Neutral | Standard professional relationship, no strong signal |
| High | Strong trust, gives benefit of the doubt, actively helpful |

### Advocacy Status (3 levels)

| Level | Description |
|-------|-------------|
| Detractor | Works against your interests, blocks initiatives, negative |
| Passive | Neutral, doesn't actively help or hurt |
| Promoter | Actively champions you, recommends you, opens doors |

## Signal Detection

When processing journals, look for relationship-relevant signals:

### Trust Battery Signals

| Signal | Direction | Example |
|--------|-----------|---------|
| Excluded from meeting/decision | ↓ | "was left out of the discussion" |
| Blamed, thrown under bus | ↓ | "she blamed the delay on my team" |
| Shared sensitive info, confided | ↑ | "told me confidentially that..." |
| Defended you to others | ↑ | "she backed me up in the meeting" |
| Gave heads-up/warning | ↑ | "gave me early warning about the reorg" |
| Dismissed your input | ↓ | "ignored my suggestion" |
| Sought your opinion | ↑ | "asked for my advice on..." |

### Advocacy Status Signals

| Signal | Direction | Example |
|--------|-----------|---------|
| Recommended you for opportunity | ↑ | "suggested I lead the initiative" |
| Praised you to leadership | ↑ | "mentioned my work to the VP" |
| Blocked your initiative | ↓ | "pushed back on my proposal" |
| Credited you publicly | ↑ | "gave me credit in the all-hands" |
| Took credit for your work | ↓ | "presented my analysis as her own" |
| Invited you to key meetings | ↑ | "included me in the strategy session" |
| Excluded you from visibility | ↓ | "didn't invite me to the exec review" |

## Update Process

1. **Detect signals** in journal entry mentioning a stakeholder
2. **Assess impact** on trust and/or advocacy
3. **Propose change** (if warranted):
   > "Based on today's entry, Sam H excluded you from the AI week discussion.
   > This suggests reduced trust. Update Trust Battery from Neutral → Low?"
4. **Wait for confirmation** before changing
5. **Log the change** in both:
   - History table with date and reason
   - Frontmatter field (trust_battery / advocacy_status)

## Update Rules

- **Always ask before updating** - show the evidence and proposed change
- **Log every change** with date and reason in the History table
- **Update frontmatter** to reflect current status
- **One direction at a time** - trust and advocacy are separate; one can change without the other

## Viewing Relationship Health

When user says "show [name]'s relationship" or "show relationship with [name]":

```
Relationship Health: Sam H

Trust Battery: Low ●○○
Advocacy Status: Passive ○●○

Trust Battery History:
  Neutral ████████████████░░░░ → Low (Jan 19)
  
  | Date       | Change        | Reason                              |
  |------------|---------------|-------------------------------------|
  | 2026-01-19 | Neutral → Low | Hijacked AI week meeting, excluded  |

Advocacy Status History:
  (No changes recorded)

Recent Interactions (last 30 days): 2
```

## Relationship Health Chart

When user says "show stakeholder health" or "show relationship chart":

Display an ASCII chart showing all stakeholders plotted on Trust vs Advocacy:

```
                        ADVOCACY STATUS
              Detractor    Passive     Promoter
           ┌────────────┬────────────┬────────────┐
     High  │            │ Taylor S   │ Jordan R   │
           │            │            │            │
TRUST      ├────────────┼────────────┼────────────┤
BATTERY    │            │ Alex A     │            │
  Neutral  │            │ Casey S    │            │
           │            │ Sam G      │            │
           ├────────────┼────────────┼────────────┤
     Low   │            │ Sam H      │            │
           │            │            │            │
           └────────────┴────────────┴────────────┘

Legend: 
- Top-right quadrant (High Trust + Promoter) = Strong allies
- Bottom-left quadrant (Low Trust + Detractor) = Risk relationships
- Middle = Neutral, opportunity to develop
```

## Relationship Trend

When user says "show [name]'s relationship trend" or "relationship history for [name]":

Display timeline visualization:

```
Relationship Trend: Sam H

Trust Battery:
  High    │                              
  Neutral │ ████████████████░            
  Low     │                 ████████████ ← current
          └──────────────────────────────
            Dec        Jan 19      Today

Advocacy Status:
  Promoter  │                              
  Passive   │ ████████████████████████████ ← current (no change)
  Detractor │                              
            └──────────────────────────────
              Dec        Jan 19      Today

Key Events:
- 2026-01-19: Trust ↓ (Hijacked AI week meeting, excluded from discussion)
```

## At-Risk Relationships

When user says "show at-risk relationships":

```
At-Risk Stakeholder Relationships (2):

HIGH RISK (Low Trust + Detractor):
- (none)

MEDIUM RISK (Low Trust OR Detractor):
- Sam H: Low Trust, Passive
  Last interaction: Jan 19 (Hijacked AI week meeting)

WATCH LIST (Neutral + Detractor):
- (none)
```

## Manual Override

User can manually set values:
> "Set Sam H's trust battery to Low"
> "Update Sarah's advocacy to Promoter"

Always:
1. Confirm the change
2. Ask for a reason to log
3. Update both frontmatter and History table

## Dossier Integration

Relationship health is stored in `Stakeholders/{name}.md` frontmatter:

```yaml
---
name: Sam H
trust_battery: Low
advocacy_status: Passive
---
```

And logged in History tables within the file:

```markdown
## Trust Battery History
| Date       | Change        | Reason                              |
|------------|---------------|-------------------------------------|
| 2026-01-19 | Neutral → Low | Hijacked AI week meeting, excluded  |

## Advocacy Status History
| Date       | Change              | Reason                              |
|------------|---------------------|-------------------------------------|
| (No changes recorded) |               |                                     |
```

## User Commands

| Command | Action |
|---------|--------|
| "show [name]'s relationship" | Display current trust/advocacy with history |
| "show stakeholder health" | Display 3x3 grid of all stakeholders |
| "show relationship chart" | Same as above |
| "show [name]'s relationship trend" | Timeline visualization for one stakeholder |
| "set [name]'s trust to [level]" | Manual trust update |
| "set [name]'s advocacy to [level]" | Manual advocacy update |
| "show at-risk relationships" | List stakeholders with Low trust or Detractor status |
