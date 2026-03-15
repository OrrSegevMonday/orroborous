# Daily Journal Master Router - Functional Flowcharts

Visual reference for the modular journal processing system defined in `AGENTS.md`.

---

## 1. System Architecture

The master router (`AGENTS.md`) discovers and orchestrates 12 modular skills organized into four tiers. The journal-processor pulls meeting summaries from Monday Notetaker (via MCP) as its first step.

```mermaid
flowchart TD
    MasterRouter["AGENTS.md<br>(Skill Orchestrator)"]

    subgraph ParseSkills [Parse]
        NT["Monday Notetaker<br>(via MCP)"] -.->|meeting summaries| JP
        JP[journal-processor<br>Pull meetings + parse entries]
        AT[action-tracker<br>Track open items]
        IP[interview-processor<br>Candidate scorecards]
    end

    subgraph UpdateSkills [Update]
        DU[dossier-updater<br>Update People / Stakeholders<br>/ Projects / Goals]
        MI[monday-integration<br>Sync with Monday.com]
        CF[calendar-follow-up<br>Schedule follow-up reminders]
    end

    subgraph AnalyzeSkills [Analyze]
        PR[performance-ratings<br>9-box grid ratings]
        RT[relationship-tracker<br>Trust & advocacy status]
        WT[win-tracker<br>Detect wins & draft recognition]
    end

    subgraph SynthesizeSkills [Synthesize]
        PRS[performance-review-summary<br>Review cycle summaries]
        WAP[week-ahead-prep<br>Strategic week planning]
        SM[staff-meeting<br>Executive briefing]
    end

    MasterRouter --> ParseSkills
    MasterRouter --> UpdateSkills
    MasterRouter --> AnalyzeSkills
    MasterRouter --> SynthesizeSkills

    JP -.->|parsed data| DU
    JP -.->|action items| AT
    AT -.->|tasks| MI
    AT -.->|follow-ups| CF
    DU -.->|people signals| PR
    DU -.->|stakeholder signals| RT
    DU -.->|win signals| WT
    PR -.->|ratings| PRS
    RT -.->|relationship health| SM
    WT -.->|wins| SM
    AT -.->|open items| WAP
    CF -.->|upcoming follow-ups| WAP
```

---

## 2. Skill Discovery Process

On every invocation the router dynamically discovers available skills. No hardcoded skill list.

```mermaid
flowchart TD
    Start([User Command Received]) --> Scan["Scan skills/*.md<br>non-recursive only"]
    Scan --> Parse["Parse YAML Frontmatter<br>for each .md file"]

    Parse --> ExtractFields{Extract Fields}

    ExtractFields --> Name[name]
    ExtractFields --> Desc[description]
    ExtractFields --> Triggers[triggers]
    ExtractFields --> Deps[dependencies]
    ExtractFields --> Version[version]

    Name --> Registry["Build In-Memory<br>Skill Registry"]
    Desc --> Registry
    Triggers --> Registry
    Deps --> Registry
    Version --> Registry

    Registry --> Cache["Cache Registry<br>for Session"]
    Cache --> Ready([Ready to Route Commands])
```

---

## 3. Command Routing Logic

User commands are matched against skill triggers. Some commands invoke a single skill; others trigger multi-skill workflows.

```mermaid
flowchart TD
    UserCmd([User Command]) --> PatternMatch{"Pattern Match<br>Against Triggers"}

    PatternMatch -->|"process my journal"| MultiSkill["Multi-Skill Workflow<br>12 skills sequential"]
    PatternMatch -->|"show team ratings"| SinglePR["performance-ratings<br>Single Skill"]
    PatternMatch -->|"sync monday"| TaskSync["action-tracker<br>+ monday-integration"]
    PatternMatch -->|"show candidates"| SingleIP["interview-processor<br>Single Skill"]
    PatternMatch -->|"show stakeholder health"| SingleRT["relationship-tracker<br>Single Skill"]
    PatternMatch -->|"show action items"| SingleAT["action-tracker<br>Single Skill"]
    PatternMatch -->|"prep for the week"| SingleWAP["week-ahead-prep<br>Single Skill"]
    PatternMatch -->|"weekly wins"| SingleWT["win-tracker<br>Single Skill"]
    PatternMatch -->|"staff meeting"| SingleSM["staff-meeting<br>Single Skill"]
    PatternMatch -->|"schedule follow-up"| SingleCF["calendar-follow-up<br>Single Skill"]
    PatternMatch -->|"performance review summary"| SinglePRS["performance-review-summary<br>Single Skill"]
    PatternMatch -->|"show available skills"| ListSkills["List All Discovered<br>Skills from Registry"]
    PatternMatch -->|No Match| Fallback["Pass to Default<br>Agent Behavior"]

    MultiSkill --> DepResolve[Dependency Resolution]
    TaskSync --> DepResolve

    SinglePR --> Execute
    SingleIP --> Execute
    SingleRT --> Execute
    SingleAT --> Execute
    SingleWAP --> Execute
    SingleWT --> Execute
    SingleSM --> Execute
    SingleCF --> Execute
    SinglePRS --> Execute
    ListSkills --> Execute

    DepResolve --> Execute[Execute Skills]
    Execute --> Summary([Return Summary])
```

---

## 4. Dependency Resolution Flow

For multi-skill workflows, a topological sort determines execution order so data flows correctly between skills.

```mermaid
flowchart TD
    Matched(["Matched Skills<br>from Triggers"]) --> BuildGraph["Build Dependency Graph<br>from each skill's deps field"]

    BuildGraph --> TopoSort[Topological Sort]

    TopoSort --> CheckDeps{"All Dependencies<br>Available?"}

    CheckDeps -->|Yes| ExecOrder["Determine<br>Execution Order"]
    CheckDeps -->|No| WarnUser["Warn: Missing Dependency<br>Execute available skills"]

    ExecOrder --> RunFirst["Execute First Skill<br>no dependencies"]
    RunFirst --> PassData["Pass Output Data<br>to Next Skill"]
    PassData --> RunNext{More Skills?}
    RunNext -->|Yes| RunFirst
    RunNext -->|No| Done([All Skills Complete])

    WarnUser --> ExecOrder
```

---

## 5. Full Journal Processing Workflow

End-to-end flow when the user says **"process my journal"** -- the most complex multi-skill workflow.

```mermaid
flowchart TD
    Cmd(["process my journal"]) --> Step0

    Step0["0. journal-processor: Step 0<br>Pull Notetaker meetings<br>User picks + merge into journal"]
    Step0 -->|"enriched journal entry"| Step1

    Step1["1. journal-processor: Steps 1-5<br>Find, parse, extract entities<br>wiki-link, structure"]
    Step1 -->|"parsed entries<br>entities extracted"| Step2

    Step2["2. dossier-updater<br>Update People / Stakeholders<br>/ Projects / Goals dossiers"]
    Step2 -->|people signals| Step3
    Step2 -->|stakeholder signals| Step4
    Step2 -->|win signals| Step4b

    Step3["3. performance-ratings<br>Detect rating changes<br>Propose updates"]
    Step3 --> Step5

    Step4["4. relationship-tracker<br>Detect relationship changes<br>Propose updates"]
    Step4 --> Step5

    Step4b["5. win-tracker<br>Detect wins<br>Draft recognition"]
    Step4b --> Step5

    Step5{"Interviews section<br>present?"}
    Step5 -->|Yes| Step5a["6. interview-processor<br>Process interview notes<br>Create scorecards"]
    Step5 -->|No| Step6
    Step5a --> Step6

    Step6{"Action items<br>found?"}
    Step6 -->|Yes| Step6a["7. action-tracker<br>Extract and track<br>action items"]
    Step6 -->|No| Step8
    Step6a --> Step6b

    Step6b["8. monday-integration<br>Create Monday.com tasks"]
    Step6b --> Step6c

    Step6c["9. calendar-follow-up<br>Schedule follow-up reminders"]
    Step6c --> Step8

    Step8["10. performance-review-summary<br>Update review cycle data"]
    Step8 --> Step9

    Step9["11. week-ahead-prep<br>Refresh week planning context"]
    Step9 --> Step10

    Step10["12. staff-meeting<br>Update briefing context"]

    Step10 --> Output(["Output Summary<br>Dossier updates | Rating signals<br>Monday tasks | Follow-ups<br>Wins | Open items"])
```

---

## 6. Data Flow Between Skills

Structured JSON objects are passed between skills. This diagram shows the data contracts.

```mermaid
flowchart LR
    NT["Monday Notetaker<br>(MCP)"] -->|"meeting summaries<br>translated to English"| JP

    JP[journal-processor] -->|"date, items,<br>entities: people[],<br>stakeholders[],<br>projects[], goals[]"| DU[dossier-updater]

    DU -->|"updated_people[]<br>name + signals[]"| PR[performance-ratings]

    DU -->|"updated_stakeholders[]<br>name + signals[]"| RT[relationship-tracker]

    DU -->|"win_signals[]"| WT[win-tracker]

    JP -->|"action items[]"| AT[action-tracker]

    AT -->|"tasks to sync"| MI[monday-integration]

    AT -->|"follow-up items[]"| CF[calendar-follow-up]

    JP -->|"interviews section"| IP[interview-processor]

    PR -->|"current ratings[]"| PRS[performance-review-summary]

    RT -->|"relationship health[]"| SM[staff-meeting]
    WT -->|"recent wins[]"| SM
    AT -->|"open items[]"| WAP[week-ahead-prep]
    CF -->|"upcoming follow-ups[]"| WAP
```

### Data Contracts Detail

**journal-processor output:**
```json
{
  "date": "2026-01-25",
  "items": ["...parsed entries..."],
  "entities": {
    "people": ["Sarah", "..."],
    "stakeholders": ["Mike", "..."],
    "projects": ["ProjectX", "..."],
    "goals": ["Q1 OKR", "..."]
  }
}
```

**dossier-updater to performance-ratings:**
```json
{
  "updated_people": [
    { "name": "Sarah", "signals": ["shipped ahead of schedule"] }
  ]
}
```

**dossier-updater to relationship-tracker:**
```json
{
  "updated_stakeholders": [
    { "name": "Mike", "signals": ["excluded from meeting"] }
  ]
}
```

**dossier-updater to win-tracker:**
```json
{
  "win_signals": [
    { "person": "Sarah", "signal": "shipped ahead of schedule", "date": "2026-01-25" }
  ]
}
```

**action-tracker to calendar-follow-up:**
```json
{
  "follow_up_items": [
    { "person": "Mike", "topic": "Alignment check", "suggested_date": "2026-01-28" }
  ]
}
```

---

## Error Handling Flow

How the system handles failures gracefully without halting the entire pipeline.

```mermaid
flowchart TD
    SkillExec([Execute Skill]) --> Check{"Skill File<br>Valid?"}

    Check -->|Missing or Malformed| Skip["Skip Skill<br>Log Warning"]
    Check -->|Valid| Run[Run Skill Logic]

    Run --> DepCheck{"Dependencies<br>Met?"}
    DepCheck -->|Yes| Execute[Execute Normally]
    DepCheck -->|No| Partial["Execute Available Skills<br>Warn User of Missing Deps"]

    Execute --> Result{"Execution<br>Succeeded?"}
    Result -->|Yes| PassData[Pass Data to Next Skill]
    Result -->|No| LogError["Log Error<br>Continue Pipeline"]

    Skip --> NextSkill{More Skills?}
    PassData --> NextSkill
    LogError --> NextSkill
    Partial --> NextSkill

    NextSkill -->|Yes| SkillExec
    NextSkill -->|No| FinalSummary(["Generate Summary<br>Include Warnings"])
```

---

## Adding New Skills

New skills are auto-discovered. No changes to the master router required.

```mermaid
flowchart LR
    Create["Create skills/new-skill.md<br>with frontmatter:<br>name, description,<br>triggers, dependencies"] --> Place["Place in skills/ directory<br>root level only"]
    Place --> NextRun[Next Invocation]
    NextRun --> Discovered["Master Router<br>Auto-Discovers Skill"]
    Discovered --> Available["Skill Available<br>for Routing"]
```
