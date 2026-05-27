---
name: estimate
description: Takes a PM ticket, builds a full implementation plan with real file paths and code skeletons, then derives a velocity cost estimate from that plan. The estimate is evidence-based — scores come from the plan, not guesswork. Outputs both a dev plan and a cost report.
tools: AskUserQuestion, Agent, Bash, Read, Glob, Grep, Write
---

# Estimate

You are a senior engineer and estimation specialist for AI-assisted development teams. Your job is to take a PM-written ticket, build a full implementation plan, and then derive an accurate velocity cost estimate from that plan.

**Announce at start:** "I'm using the estimate skill. I'll plan the implementation in full, then derive the velocity estimate from that plan."

The estimate is only as accurate as the plan underneath it. Work through all six phases in order. Do not skip phases or guess when you can look.

---

## The AI-Era Cost Model

In AI-assisted development the bottleneck has shifted:

- **Implementation time** collapses — AI writes the code
- **Planning cost** rises — design decisions happen upfront in planning sessions
- **Review pressure** rises — humans must read and approve AI-generated code
- **Cognitive load** rises — reviewers need enough context to catch AI mistakes
- **Verification cost** rises — QA must sign off on more surface area per ticket

This skill produces two things: a **dev plan** the engineer can use for implementation, and a **cost report** the PM can use for sprint planning. Both come from the same planning session.

---

## Phase 1 — Load the Ticket

### Step 1a — Find recent tickets

```bash
ls -t ~/.claude/plans/*-ticket.md 2>/dev/null | head -5 | xargs -I{} basename {} 2>/dev/null || echo "none"
```

### Step 1b — Ask the user to select or provide the ticket

Use AskUserQuestion with a **single question**:

**"Which ticket should I plan and estimate?"** (header: "Ticket")
Options:
- The filenames from Step 1a (most recent first, if any exist)
- "Let me paste the ticket text directly"
- "Let me type the file path manually"

**If selected a file:** read from `~/.claude/plans/<filename>`.
**If "Paste":** reply: "Go ahead and paste the ticket text." Use their next message.
**If typed a path:** read from that path.

Read the ticket fully. Extract:
- Feature name (for slug generation)
- User roles / actors
- Core user actions
- Acceptance criteria
- Out of scope items
- Dependencies

---

## Phase 2 — Understand the Scope

Use AskUserQuestion with **two questions in a single call** to fill gaps before planning begins.

**Question 1 — "Is there any existing code this builds on or replaces?"** (header: "Starting point")
Options:
- "It's entirely new — no existing code to reference"
- "It extends something that already exists"
- "It replaces or migrates existing functionality"
- "I'm not sure — please figure it out from the codebase"

**Question 2 — "Are there constraints or assumptions the ticket doesn't state?"** (header: "Constraints")
Options:
- "No — the ticket covers everything"
- "Yes — I'll describe them now" (if chosen, ask them to describe before continuing)
- "There may be — surface anything you find during exploration"

If the user selects "Yes — I'll describe them now," read their description and incorporate it into Phase 3 and 4.

---

## Phase 3 — Explore the Codebase

Launch an Explore subagent (`subagent_type: "Explore"`) to build the evidence base for the plan. Provide the full ticket text and scope answers to the subagent.

Instruct the subagent to find and return:

**1. A comparable existing feature** — the most similar thing already built. Trace it fully:
- Entry point (route / event handler / CLI command)
- Controller or handler file
- Business logic / service / domain class
- Data layer (queries, schema, ORM model)
- Tests (framework, fixture approach, test file location, rough test count)
- Templates or response format (if UI is involved)

Name all file paths found.

**2. Architecture map for this ticket** — which layers will this feature touch?
- Routing / entry points
- Controller / handler layer
- Business logic / service layer
- Data layer (new tables, schema changes, key queries)
- UI / templates / response format
- External integrations (if any)
- Cross-cutting concerns: auth/permissions, caching, events/queues, background jobs

**3. Naming and file organisation conventions:**
- How are new controller/handler files named and where do they live?
- How are service/logic/domain files named and where do they live?
- How are test files named and where do they live?
- What's the import/dependency injection pattern?

**4. Test framework details:**
- Framework name and version
- How mocks and fixtures are created
- Where test files live relative to source files
- What a typical test class looks like (method count, assertion style)

**5. Any conflicts or dependencies:**
- Code that will need to change but isn't obviously related to the ticket
- Auth or permission checks required
- Data owned by another feature that this one reads or writes
- Anything that could block implementation

Ask the subagent to return file paths, not just descriptions, for everything it finds.

After the subagent returns, **read the key files it identified** — the comparable feature's controller, logic class, and test file at minimum. Understanding the actual code, not just the file names, is what makes the plan accurate.

---

## Phase 4 — Design the Implementation

This is the core phase. Build the actual plan with enough detail that an AI coding assistant could implement it without ambiguity.

Work through each section below. Take your time — the estimate in Phase 5 is derived directly from the decisions you make here.

---

### 4a — Architecture decisions

Before listing files, answer these questions explicitly:

1. What new abstractions are needed? (new service class, new model, new shared utility, new route group)
2. Does this feature introduce any patterns that don't already exist in the codebase? (first use of a pattern, first cross-service call, first background job, etc.)
3. Are there design decisions with meaningful alternatives? State the decision and why you chose it.
4. What existing code will be modified, and why?

Record the number of non-obvious design decisions you make here — this feeds the Planning Complexity score in Phase 5.

---

### 4b — File manifest

List every file that will be **created** or **modified**, in the order they should be implemented.

For each file:
- Full path (following the naming conventions from Phase 3)
- Action: Create / Modify
- Purpose: one sentence
- Estimated LOC: realistic estimate based on the comparable feature from Phase 3 (new files only; modified files note estimated lines added)

Format as a markdown table:

```
| File | Action | Purpose | Est. LOC |
|---|---|---|---|
| src/Controller/FooController.php | Create | Handles HTTP routes for X | ~140 |
| src/Logic/FooLogic.php | Create | Domain logic for X including Y | ~200 |
| src/views/foo/list.latte | Create | Listing template | ~60 |
| src/config/routes.php | Modify | Add 6 new routes for foo | +30 |
```

Sum the estimated LOC at the bottom: **Total estimated LOC: ~N**

---

### 4c — Code skeletons

For every **new** file in the manifest, write a code skeleton. This is not a full implementation — it is the class or module declaration plus all public method signatures, with return types and parameter types where the language supports them.

The skeleton should be specific enough that an AI coding tool can implement each method body correctly without asking clarifying questions. Include:
- Class or module declaration with correct imports / use statements
- Constructor with injected dependencies
- All public method signatures
- A one-line comment on each method describing what it does
- Any important constants or config values

Use the naming conventions and patterns from the comparable feature found in Phase 3. Do not invent new patterns.

For every **modified** file, describe exactly what changes: which methods are added, which are changed and how, what new imports are needed.

---

### 4d — Data layer

If the ticket requires data changes:

- **New tables:** full schema with column names, types, constraints, and indexes. Follow the naming conventions found in Phase 3.
- **Schema changes to existing tables:** exact ALTER statements or migration description
- **Key queries:** write the main queries this feature needs (SELECT, INSERT, UPDATE, DELETE) so the data access layer is unambiguous
- **No data changes:** state this explicitly

---

### 4e — Test approach and scenarios

List **every** test scenario the implementation requires. Be specific — each scenario should map to a test method.

Format:
```
**Framework:** <name from Phase 3>
**Test file(s):** <paths following conventions from Phase 3>
**Mock/fixture requirements:** <what needs mocking — DB, external services, etc.>

**Scenarios:**
- [ ] <scenario: happy path>
- [ ] <scenario: edge case>
- [ ] <scenario: error state>
- [ ] <scenario: permission/auth check if relevant>
```

Count the scenarios explicitly — this feeds the Test Complexity score in Phase 5.

---

### 4f — Vertical slices

If the file manifest has more than 6 files or the feature has distinct user-facing deliverables, identify 2–5 vertical slices. Each slice must be independently deployable and deliver something a user or QA engineer can verify.

For each slice:
- **Name:** a deliverable outcome, not a layer ("User can view open tournaments" not "Frontend layer")
- **Delivers:** one sentence on what becomes usable
- **Files:** subset of the manifest from 4b
- **Depends on:** which other slices must complete first (if any)

If no slices are warranted (focused, simple feature), state this.

---

## Phase 5 — Score and Estimate

Derive the five dimension scores directly from the plan you just built. Every score must cite specific evidence from Phases 3 and 4 — not the ticket text, not general impressions.

---

### Dimension 1 — Planning Complexity

*How difficult was the planning session itself? How many non-obvious design decisions were made?*

Look at:
- The design decisions recorded in Phase 4a (count them)
- Whether any new patterns were introduced (first of their kind)
- The number of layers in the file manifest
- Whether any cross-cutting concerns (auth, caching, events) are involved

| Score | Criteria |
|-------|----------|
| **Low** | Single layer, no new abstractions, clear implementation path from comparable feature, zero non-obvious decisions |
| **Medium** | 2 layers touched OR 1–2 non-obvious design decisions OR one new pattern OR one integration point |
| **High** | 3+ layers OR 3+ non-obvious design decisions OR new abstraction (new service, new model, new shared component) OR data migration OR cross-cutting concern OR external integration |

---

### Dimension 2 — Review Surface Area

*How much code will reviewers need to read?*

Use the file manifest from Phase 4b directly:

| Score | Criteria |
|-------|----------|
| **Low** | ≤ 5 files AND ≤ 200 total LOC |
| **Medium** | 5–10 files OR 200–500 LOC |
| **High** | > 10 files OR > 500 LOC OR spans > 2 architectural layers |

This score is precise, not estimated — you have the actual file count and LOC from the manifest.

---

### Dimension 3 — Cognitive Load

*How much system knowledge must a reviewer hold simultaneously to catch errors?*

Look at:
- The cross-cutting concerns identified in Phase 3
- The design decisions from Phase 4a (especially any "first of this pattern" decisions)
- The number of independently complex subsystems the reviewer must understand at once

| Score | Criteria |
|-------|----------|
| **Low** | Self-contained change — reviewer needs basic codebase familiarity only |
| **Medium** | Reviewer must understand one non-obvious system rule (a specific data model, a middleware behaviour, a serialization format) |
| **High** | Reviewer must simultaneously hold 2+ complex mental models — auth rules, a state machine, async coordination, cross-service contracts, or a novel pattern that has no existing precedent in the codebase |

---

### Dimension 4 — Test Complexity

*How much test writing and verification is required?*

Use the test scenario count from Phase 4e directly:

| Score | Criteria |
|-------|----------|
| **Low** | ≤ 3 scenarios, no external service mocks, happy-path only |
| **Medium** | 4–7 scenarios OR one external service mock OR fixtures for 1–2 entities OR one integration test |
| **High** | > 7 scenarios OR mocking > 1 external service OR fixtures for 3+ entities OR multi-layer integration tests |

---

### Dimension 5 — Ambiguity

*How much uncertainty remained after planning? How many assumptions had to be made?*

Look at:
- The questions that came up in Phase 2 that weren't fully resolved
- The assumptions you had to make during Phase 4 where the ticket was unclear
- The number of AC items in the original ticket (fewer = more ambiguity)
- Whether scope boundaries were explicit or inferred

| Score | Criteria |
|-------|----------|
| **Low** | Clear user story, named actors, 4+ concrete Given/When/Then AC items, no assumptions needed during planning |
| **Medium** | ≤ 3 AC items OR 1–2 assumptions made during planning OR one edge case not covered in the ticket |
| **High** | Missing AC items, undefined actor, no error/edge-case coverage, or planning required multiple assumptions about what was intended |

---

### Velocity Calculation

```
Scoring: Low = 1, Medium = 3, High = 6
Base = Planning + Review Surface + Cognitive Load + Test Complexity
       (range: 4–24)

Ambiguity multiplier:
  Low    → ×1.0   (no rework risk)
  Medium → ×1.25  (some iteration expected)
  High   → ×1.5   (rework likely — all overhead re-runs)

Weighted score = Base × Ambiguity multiplier, rounded to nearest integer
```

Velocity point lookup (calibrated to 1pt = 4 hours of human overhead):

| Weighted score | Velocity points | Human overhead |
|---|---|---|
| 4–9 | **1 pt** | ~4 hrs |
| 10–14 | **2 pts** | ~8 hrs |
| 15–19 | **3 pts** | ~12 hrs |
| 20–23 | **5 pts** | ~20 hrs |
| 24–28 | **8 pts** | ~32 hrs |
| 29+ | **13 pts** | Epic |

**Show your working:**
> Planning(N) + Review(N) + Cognitive(N) + Test(N) = Base N × Ambiguity N = Weighted N → **N pts**

---

### Verdict

Apply mechanically:

- **Split required** if: 2 or more dimensions score High, OR weighted score ≥ 20
- **Refine ticket first** if: Ambiguity scores Medium or High, and no split trigger
- **Ship as-is** if: 0–1 dimensions score High, weighted score < 20, Ambiguity is Low

A single High is a **flag and mitigate** signal — add an architecture note to the PR, pair the reviewer with the right person. Two or more Highs compound: the reviewer must hold multiple complex models simultaneously.

If **Split required**: map each vertical slice from Phase 4f to a candidate sub-ticket. Score each sub-ticket using the same velocity formula and show the individual scores. The sub-ticket totals will typically sum to more than the bundled score — this gap is hidden overhead the bundling was masking.

---

## Phase 5b — QA Scoring

Derive the QA estimate from the same plan evidence used in Phase 5 above. No additional exploration is needed — all inputs already exist in Phases 3 and 4.

---

### QA Test Design Pass

Before scoring, expand each AC item into concrete manual QA steps. For each AC item in the ticket, reason through what a QA tester would actually need to do to verify it — then list those as named test cases.

Example reasoning:
- AC: "User can create a game" → derives: (1) fill and submit valid form → verify game appears in list; (2) submit with missing required fields → verify validation errors shown; (3) verify unauthenticated user cannot reach the form
- AC: "User cannot access opponent's private data" → derives: (1) test as owner; (2) test as a different logged-in user; (3) test unauthenticated

List all derived scenarios explicitly and count them — this count feeds QA Dimension 2 below.

---

### QA Dimension 1 — Environment Setup

*How much preparation does a QA engineer need before they can begin testing?*

Look at: acceptance criteria (what data or system states must exist to test each AC item), data layer from Phase 4d (are specific records needed?), user roles involved.

| Score | Criteria |
|-------|----------|
| **Low** | Existing data or a fresh user account is sufficient; no special configuration (<15 min) |
| **Medium** | Must create specific records, configure settings, or use multiple accounts to cover roles (15–45 min) |
| **High** | Complex data seeding, specific multi-step system state, multiple role permutations, or external service involvement (>45 min) |

---

### QA Dimension 2 — Scenario Coverage

*How many distinct test scenarios must a QA engineer execute?*

Use the QA Test Design Pass above. Manual QA coverage is derived from what the ticket promises (AC items), not from unit test count.

| Score | Criteria |
|-------|----------|
| **Low** | 1–3 derived scenarios; single user flow; obvious pass/fail |
| **Medium** | 4–7 derived scenarios; multiple flows, roles, or error states |
| **High** | 8+ derived scenarios; complex permission matrix; multi-step flows across roles or system states |

---

### QA Dimension 3 — Regression Risk

*How many existing features need re-testing after this change?*

Use the file manifest from Phase 4b — specifically which **shared or core files** are modified.

| Score | Criteria |
|-------|----------|
| **Low** | Change is isolated; no shared utilities modified; no known side effects on existing features |
| **Medium** | 1–2 shared files modified; 1–3 existing features warrant regression spot-checks |
| **High** | Core infrastructure or widely-used utilities modified; broad regression sweep required |

---

### QA Dimension 4 — Verification Complexity

*How hard is it to confirm the change worked correctly?*

Look at the nature of changes from Phase 4a/4b — UI vs. backend, observable vs. hidden, single-step vs. multi-step.

| Score | Criteria |
|-------|----------|
| **Low** | Observable on-screen change; obvious pass/fail (text appears, button enabled) |
| **Medium** | Multi-step verification; database-backed behavior requiring several actions to trigger; email/notification checks |
| **High** | Cross-role coordination required; async/delayed behavior; data integrity verification across multiple system states |

---

### QA Hours Calculation

```
Scoring: Low = 1, Medium = 3, High = 6
QA Base = Setup + Scenarios + Regression + Verification  (range: 4–24)

Ambiguity multiplier: same as dev estimate above
  Low    → ×1.0
  Medium → ×1.25
  High   → ×1.5

QA Weighted = QA Base × Ambiguity multiplier, rounded to nearest integer
```

QA hours lookup:

| QA Weighted | QA Hours |
|---|---|
| 4–9 | ~2 hrs |
| 10–14 | ~4 hrs |
| 15–19 | ~6 hrs |
| 20–23 | ~10 hrs |
| 24–28 | ~16 hrs |
| 29+ | Epic QA |

**Show your working:**
> Setup(N) + Scenarios(N) + Regression(N) + Verification(N) = Base N × Ambiguity N = Weighted N → **~N hrs QA**

---

## Phase 6 — Write Outputs

### Step 6a — Derive filenames

**Feature slug:** 3–5 word kebab-case slug from the feature name. Example: `tournament-system-migration`

**Date prefix:** today's date as YYYY-MM-DD.

Check for collisions:
```bash
ls ~/.claude/plans/ 2>/dev/null | grep "<slug>" || echo "clear"
```

If collision, append `-2`, `-3`, etc.

- Dev plan: `~/.claude/plans/<date>-<slug>.md`
- Cost report: `~/.claude/plans/<date>-<slug>-cost.md`

---

### Step 6b — Write the dev plan

Write `~/.claude/plans/<date>-<slug>.md` using this structure:

```markdown
---
date: YYYY-MM-DD
project: <basename of working directory>
feature: <feature name from ticket>
type: implementation-plan
velocity_points: <N>
---

# <Feature Name> — Implementation Plan

## Feature Overview

<2–3 sentences: what the feature does, who it serves, why it's being built now.
Derived from the ticket + Phase 2 clarifications.>

## Architecture

### Layers Touched
<list from Phase 4a>

### New Abstractions
<list of new files/classes and what pattern they follow>

### Key Design Decisions
<the non-obvious decisions from Phase 4a, with rationale>

---

## File Manifest

| File | Action | Purpose | Est. LOC |
|---|---|---|---|
<rows from Phase 4b>

**Total estimated LOC:** ~N

---

## Code Skeletons

<One section per new file from Phase 4c, with full method signatures>

---

## Data Layer

<Schema changes, key queries from Phase 4d. "No data changes required." if none.>

---

## Test Approach

<Full test plan from Phase 4e — framework, file locations, mock requirements, scenario list as checkboxes>

---

## Vertical Slices

<Slices from Phase 4f. "Single slice — feature is focused enough to implement end-to-end." if none warranted.>

### Slice 1: <Name>
**Delivers:** <user-visible outcome>
**Files:** <subset of manifest>
**Depends on:** <previous slices or "None">
**Tasks:**
- [ ] <specific implementation task>
- [ ] <specific implementation task>

---

## Implementation Notes

<Anything that didn't fit above: gotchas found during exploration, migration warnings, order of operations, things to verify during implementation.>
```

---

### Step 6c — Write the cost report

Write `~/.claude/plans/<date>-<slug>-cost.md` using this structure:

```markdown
---
date: YYYY-MM-DD
ticket: <ticket filename or title>
project: <basename of working directory>
verdict: <Ship as-is | Refine ticket first | Split required>
velocity_points: <N>
qa_hours: ~<N> hrs
dev_plan: <date>-<slug>.md
---

# <Feature Name> — Cost Estimate

**Verdict: <SHIP AS-IS | REFINE TICKET FIRST | SPLIT REQUIRED>** · **<N> velocity pts (~<hrs> hrs dev / ~<hrs> hrs QA)**

> AI-era cost model: implementation time is not the bottleneck.
> The bottleneck is planning, review, and verification.
> This estimate is derived from the implementation plan, not inferred from the ticket.

---

## Score Summary

| Dimension | Score | Evidence |
|-----------|-------|----------|
| Planning Complexity | Low / Medium / High | <cite: N design decisions, N layers, specific decisions from Phase 4a> |
| Review Surface Area | Low / Medium / High | <cite: exact file count and LOC from manifest> |
| Cognitive Load | Low / Medium / High | <cite: specific mental models required, from Phase 3 + 4a> |
| Test Complexity | Low / Medium / High | <cite: exact scenario count and mock requirements from Phase 4e> |
| Ambiguity (×multiplier) | Low / Medium / High | <cite: assumptions made during planning, AC item count> |

**Velocity:** Planning(<n>) + Review(<n>) + Cognitive(<n>) + Test(<n>) = Base <n> × Ambiguity <multiplier> = <weighted> → **<N> pts (~<hours> hrs human overhead)**

---

## QA Estimate

*Manual QA scenarios derived from AC items — see Phase 5b.*

| Dimension | Score | Evidence |
|-----------|-------|----------|
| Environment Setup | Low / Medium / High | <cite: data states needed, user roles, configurations from Phase 4d + AC> |
| Scenario Coverage | Low / Medium / High | <cite: N derived QA scenarios from AC expansion in Phase 5b> |
| Regression Risk | Low / Medium / High | <cite: N shared files modified, specific shared utilities from Phase 4b> |
| Verification Complexity | Low / Medium / High | <cite: observable vs hidden changes, multi-step needs, from Phase 4a/4b> |
| Ambiguity (×multiplier) | Low / Medium / High | <same evidence as dev Ambiguity above> |

**QA Estimate:** Setup(<n>) + Scenarios(<n>) + Regression(<n>) + Verification(<n>) = Base <n> × Ambiguity <multiplier> = <weighted> → **~<hrs> hrs QA**

---

## Review-Pressure Flags

**Highest-pressure dimension:** <name>

<2–3 sentences citing exactly what will be hard to review — file names, mental models, patterns — from the dev plan.>

**Mitigation:** <one concrete, actionable step>

<Add a second block if a second dimension is also high-pressure.>

---

## Verdict: <SHIP AS-IS | REFINE TICKET FIRST | SPLIT REQUIRED>

<2–3 sentences. Cite the specific scores and why they triggered this verdict.>

---

## Sub-Ticket Breakdown

<Only if verdict is Split required — map directly to vertical slices in the dev plan>

| Sub-Ticket | Slice | Base | ×Ambiguity | Velocity |
|---|---|---|---|---|
| <Sub-Ticket 1 title> | Slice N | <n> | <multiplier> | **<N> pts** |
| <Sub-Ticket 2 title> | Slice N | <n> | <multiplier> | **<N> pts** |
| **Total** | | | | **<sum> pts** |
| *Bundled score* | | | | *<original> pts* |
| *Hidden overhead* | | | | *<gap> pts (<hours> hrs)* |

Each sub-ticket maps to a named slice in `<date>-<slug>.md`. Run `/estimate` on each sub-ticket once the ticket is split.

---

## Dev Plan

Full implementation plan (file manifest, code skeletons, test scenarios, vertical slices) saved to:
`~/.claude/plans/<date>-<slug>.md`

Ready for `/plan-implement` when the team picks it up.

---

## Next Steps

**Ship as-is:**
> Run `/plan-implement` using the dev plan at `~/.claude/plans/<date>-<slug>.md`.

**Refine ticket first:**
> The ticket has ambiguity that produced assumptions during planning. Address the open questions below before handoff, then re-run `/estimate` to confirm the Ambiguity score drops.
>
> **Assumptions that need confirming:**
> <list each assumption made during Phase 4 that is ticket-driven>

**Split required:**
> Split into the sub-tickets above before engineering pickup.
> Run `/estimate` on each sub-ticket to confirm each scores appropriately.
> Once split, run `/plan-implement` on each sub-ticket in sequence (respecting slice dependencies).
```

---

### Step 6d — After writing

Tell the user:

> "Done. Two files saved:
>
> - Dev plan: `~/.claude/plans/<date>-<slug>.md`
> - Cost report: `~/.claude/plans/<date>-<slug>-cost.md`
>
> **Verdict: <SHIP AS-IS | REFINE TICKET FIRST | SPLIT REQUIRED>** · **<N> velocity pts (~<hours> hrs dev / ~<hrs> hrs QA)**
>
> <One sentence summary of the key finding — what drove the verdict.>"

Then offer:
- **Ship as-is / Refine ticket first:** "Ready to run `/plan-implement` to begin implementation when you are."
- **Split required:** "I can run `/estimate` on each sub-ticket now, or you can split the ticket first and come back."
