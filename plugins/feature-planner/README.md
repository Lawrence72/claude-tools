# feature-planner

A suite of Claude Code skills for planning features before you build them. Interviews you about what you want, explores your codebase, and produces structured planning artefacts — a developer plan, a vertical-slice implementation guide, and a PM ticket.

Works with any tech stack — PHP, TypeScript, Python, Go, Ruby, or anything else.

## Installation

Add the marketplace (if not already added):

```
/plugins add marketplace github:Lawrence72/claude-tools
```

Install the plugin:

```
/plugins install feature-planner
```

## Skills

This plugin provides three slash commands:

| Command | What it does |
|---------|-------------|
| `/feature-planner` | Full planning workflow — asks questions, explores the codebase, writes a structured plan, then optionally chains into the next two skills |
| `/plan-slice` | Restructures any existing plan into vertical slices with code skeletons |
| `/plan-ticket` | Generates a PM-ready ticket (user story + acceptance criteria) from any existing plan — Generic, Jira, or ADO format |
| `/plan-implement` | Works through a vertical-slice plan step by step — implements and verifies each slice, suggests commit messages but leaves committing to the developer |
| `/plan-verify` | Independently verifies the implementation against the plan — checks files, tests, functional verification, code alignment, and acceptance criteria |
| `/plan-retro` | Retrospective on a completed feature — compares plan vs reality, captures drift, and surfaces patterns worth adding to CLAUDE.md |
| `/plan-status` | Dashboard view of all plans — shows artefacts, task completion, and overall state for every feature at a glance |

---

## `/feature-planner`

The main entry point. Guides you through four phases:

1. **Discovery** — asks what you want to build, the scope, and any hard constraints
2. **Exploration** — scans the relevant parts of your codebase for existing patterns, similar features, and potential conflicts
3. **Design questions** — targeted follow-ups based on what it found (auth patterns, test strategy, data ownership)
4. **Plan generation** — writes a structured markdown plan to `~/.claude/plans/`

After the plan is saved, it asks if you want to continue to `/plan-slice`, `/plan-ticket`, or both — all in the same session.

**Output:** `~/.claude/plans/YYYY-MM-DD-feature-name.md`

---

## `/plan-slice`

Takes any existing feature plan and restructures it into vertical slices — ordered, end-to-end increments where each slice delivers something working and testable before the next begins.

Each slice includes:
- A "Delivers" statement describing what can be verified once complete
- Ordered sub-tasks with **code skeletons** drawn from your actual codebase patterns
- A verification step and commit point

Claude decides the slice boundaries based on the feature — no fixed formula.

**Input:** any plan file in `~/.claude/plans/`  
**Output:** `~/.claude/plans/YYYY-MM-DD-feature-name-dev.md`

---

## `/plan-ticket`

Takes any existing feature plan and generates a non-technical PM-ready ticket — written for a product manager, stakeholder, or QA engineer.

Includes:
- User story (As a / I want / So that)
- Background (why this is being built)
- Acceptance criteria (Given / When / Then, independently testable)
- Out of scope
- Dependencies
- Complexity estimate

Suitable for pasting directly into Jira, ADO, Linear, or any ticket system.

**Input:** any plan file in `~/.claude/plans/`  
**Output:** `~/.claude/plans/YYYY-MM-DD-feature-name-ticket.md`

---

## Example Output Files

Running `/feature-planner` on a "user notifications" feature, then choosing "Both", produces three files:

```
~/.claude/plans/
├── 2026-05-22-user-notifications.md        ← basic architecture plan
├── 2026-05-22-user-notifications-dev.md    ← vertical slices + code skeletons
└── 2026-05-22-user-notifications-ticket.md ← PM ticket for Jira/ADO
```

Use `/list-plans` to browse and load saved plans.

---

## `/plan-implement`

The final step in the chain. Takes a vertical-slice developer plan and works through it slice by slice — implementing each sub-task using the code skeletons from the plan, checking off tasks as they complete, verifying each slice before committing, and never moving forward on a broken slice.

**Two ways to provide the plan:**
- Select from the 3 most recent `-dev.md` files (the plan file gets updated in place with `- [x]` checkboxes as tasks complete)
- Paste the markdown content directly into the conversation

**Two execution modes:**
- **Interactive** — pauses after each slice for review before continuing
- **Autonomous** — works through all slices without stopping

**Output:** working code, one commit per slice, all checkboxes ticked in the plan file.

---

## Full Chain

```
/feature-planner          → feature-name.md           (architecture plan)
  └─ /plan-slice          → feature-name-dev.md        (vertical slices + code skeletons)
  └─ /plan-ticket         → feature-name-ticket.md     (PM ticket — Generic / Jira / ADO)

/plan-implement           reads feature-name-dev.md    (implements, suggests commits)

/plan-verify              reads feature-name-dev.md    (independent QA pass)
  + optionally reads      feature-name-ticket.md       (checks acceptance criteria too)

<developer commits & raises PR using their team's conventions>

/plan-retro               reads feature-name-dev.md    (plan vs reality, CLAUDE.md candidates)
  + optionally reads      feature-name-ticket.md       → feature-name-retro.md

/plan-status              scans ~/.claude/plans/        (dashboard of all features)
```

---

## Why plan first?

Jumping straight to "add a notifications feature" often produces code that misses existing conventions, duplicates patterns, or conflicts with the rest of the codebase. This plugin does the upfront investigation so implementation is fast, consistent, and targeted.

The saved plans are portable — open them in a fresh session and hand them to Claude (or another developer) with full context intact.
