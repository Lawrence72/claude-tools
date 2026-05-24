# ticket-estimator

A Claude Code skill that takes a PM-written ticket, builds a full implementation plan with real file paths and code skeletons, then derives a velocity cost estimate from that plan. The estimate is evidence-based — scores come from the plan, not guesswork.

Works with any tech stack. Codebase-agnostic.

---

## Why plan first, then estimate?

Traditional estimation tools ask humans to guess complexity. This skill asks Claude to *plan* the implementation in full — then the estimate falls out naturally from what the plan reveals:

- **Review Surface Area** comes from the file manifest (exact count and LOC)
- **Test Complexity** comes from the test scenario list (exact count)
- **Planning Complexity** comes from the design decisions made during planning
- **Cognitive Load** comes from the cross-cutting concerns found during codebase exploration
- **Ambiguity** comes from the assumptions that had to be made when the ticket was unclear

The estimate is only as accurate as the plan underneath it. So the skill builds the plan first.

---

## Skills

| Command | What it does |
|---------|-------------|
| `/estimate` | Takes a PM ticket → builds a full dev plan with file paths + code skeletons → derives a velocity cost estimate from that plan → saves both files |

---

## `/estimate`

### What it produces

Two files in `~/.claude/plans/`:

**Dev plan** (`YYYY-MM-DD-feature-slug.md`):
- File manifest with paths and estimated LOC
- Code skeletons (class declarations, method signatures)
- Data layer changes (schema, key queries)
- Test approach and full scenario list
- Vertical slices with tasks

**Cost report** (`YYYY-MM-DD-feature-slug-cost.md`):
- Five-dimension score table with evidence citations
- Velocity points (calibrated to 1pt = 4 hours of human overhead)
- Verdict: Ship as-is / Refine ticket first / Split required
- Named sub-ticket breakdown (if split required)
- Review-pressure flags with concrete mitigations

### The Five Dimensions

| Dimension | Evidence source |
|-----------|----------------|
| **Planning Complexity** | Design decisions made during planning |
| **Review Surface Area** | Exact file count and LOC from the file manifest |
| **Cognitive Load** | Cross-cutting concerns and novel patterns found in codebase exploration |
| **Test Complexity** | Exact scenario count from the test plan |
| **Ambiguity** | Assumptions made during planning where the ticket was unclear |

### Velocity formula

```
Scoring: Low=1, Medium=3, High=6  (non-linear — High is disproportionately harder)

Base = Planning + Review + Cognitive + Test  (range 4–24)

Ambiguity multiplier:
  Low    → ×1.0   (no rework risk)
  Medium → ×1.25  (some iteration expected)
  High   → ×1.5   (rework likely; all overhead re-runs)

Weighted = Base × multiplier

Velocity: 4–9→1pt, 10–14→2pt, 15–19→3pt, 20–23→5pt, 24–28→8pt, 29+→13pt
          (1pt = 4 hours of human overhead)
```

### Split rule

- **Split required** if 2+ dimensions score High, OR weighted score ≥ 20
- **Refine ticket first** if Ambiguity is Medium/High with no split trigger
- **Ship as-is** otherwise

Split suggestions map directly to vertical slices in the dev plan.

---

## Full workflow chain

```
PM writes ticket

/estimate
  ├── reads ticket
  ├── explores codebase (Explore subagent)
  ├── builds dev plan (file manifest + code skeletons + test plan + slices)
  ├── scores 5 dimensions from the plan
  ├── calculates velocity
  └── saves dev plan + cost report

  ├── Ship as-is     → /plan-implement (uses the saved dev plan)
  ├── Refine ticket  → PM updates ticket → /estimate again
  └── Split required → split into sub-tickets → /estimate on each
                       → /plan-implement on each in dependency order
```

---

## What makes this different

Most estimation tools score the *ticket*. This tool scores the *plan*.

The difference is that a ticket describes intent. A plan reveals reality — how many files actually change, how many test scenarios are actually required, what design decisions actually have to be made, what a reviewer actually needs to understand.

Claude can't be pressured by sprint politics. It applies the rubric to what it finds in the code, not what the team wants to hear.
