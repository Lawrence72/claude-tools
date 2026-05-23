# ticket-estimator

A Claude Code skill for estimating ticket cost in AI-assisted development teams. In AI-era development, implementation time collapses — but planning, review, and verification costs remain. This plugin measures those costs before engineering picks up a ticket.

Works with any tech stack. Codebase-agnostic.

## The AI-Era Cost Model

| Old model | New model |
|-----------|-----------|
| Cost = how long it takes to write the code | Cost = planning + review + verification |
| Implementation time dominates | Planning complexity dominates |
| Senior dev writes it faster | Complexity is in understanding, not typing |

## Skills

| Command | What it does |
|---------|-------------|
| `/ticket-cost` | Estimates the AI-era cost of a PM-written ticket across five dimensions — produces a cost report with a split/ship/refine verdict |

---

## `/ticket-cost`

Analyses a PM-written ticket and produces a structured cost report. The report is calibrated for AI-assisted development, where implementation time is not the bottleneck.

### The Five Dimensions

| Dimension | What it measures |
|-----------|-----------------|
| **Planning Complexity** | How hard will the `/plan` session be? How many design decisions? |
| **Review Surface Area** | How many files and lines must reviewers read? |
| **Cognitive Load** | How much system context must a reviewer hold to spot AI mistakes? |
| **Test Complexity** | How much test writing and verification is required? |
| **Ambiguity** | How well-defined is the ticket? Will AI execute it correctly? |

Each dimension is scored Low / Medium / High using concrete, measurable criteria grounded in the actual codebase.

### Verdicts

| Verdict | Meaning |
|---------|---------|
| **Ship as-is** | Appropriately sized — no dimension is High, fewer than 3 are Medium |
| **Refine ticket first** | The ticket itself is the problem — AC rewrites and actor clarifications are provided |
| **Split required** | Too large for one sprint item — named candidate sub-tickets are provided |

### Three Ways to Provide the Ticket

1. Select from recent `-ticket.md` files in `~/.claude/plans/`
2. Type a path manually
3. Paste the ticket text directly

### Output

**Saved to:** `~/.claude/plans/<date>-<feature-slug>-cost.md`

The cost report includes:
- Per-dimension score table with one-line rationale citing specific files
- Review-pressure flags with concrete mitigations
- Named split suggestions (not just "make it smaller")
- Ticket quality improvements: AC rewrites, missing actors, implicit assumptions
- Codebase grounding: comparable feature, LOC estimate, layers touched

---

## Full Chain

```
/plan-ticket              → feature-name-ticket.md     (PM ticket from dev plan)

/ticket-cost              reads feature-name-ticket.md  (AI-era cost estimate)
  ├── Ship as-is          → /feature-planner            (begin planning)
  ├── Refine ticket first → update ticket, re-run       (fix ambiguity first)
  └── Split required      → split into sub-tickets      (run /ticket-cost on each)

/feature-planner          → feature-name.md             (architecture plan)
  └─ /plan-slice          → feature-name-dev.md         (vertical slices + skeletons)
  └─ /plan-implement                                    (implements the plan)
  └─ /plan-verify                                       (independent QA pass)
```

---

## Why estimate before planning?

Engineering time spent planning a ticket that is too large — or too ambiguous — is wasted. The planning session stalls on design decisions that should have been resolved at the ticket stage. `/ticket-cost` surfaces that risk before the session starts, so the PM can refine or split the ticket while the cost is low.

The estimate is grounded in the actual codebase (via an Explore subagent) rather than just the ticket text, so the score reflects what will actually be touched — not what the ticket says.
