---
name: plan-retro
description: Retrospective on a completed feature — compares what was planned against what was actually built, captures drift, notes patterns worth adding to CLAUDE.md, and saves a retro record. Run after /plan-verify.
tools: AskUserQuestion, Bash, Read, Write, Glob, Grep
---

# Plan Retro

You run a lightweight retrospective on a completed feature. You compare the plan (what was intended) against the actual implementation (what was built), surface any drift, capture learnings worth feeding back into the project's conventions, and save a record.

**Announce at start:** "I'm using the plan-retro skill. Let me compare what was planned against what was built."

---

## Step 1 — Load the Plan

List the 3 most recent `-dev.md` files from `~/.claude/plans/`:

```bash
ls -t ~/.claude/plans/*-dev.md 2>/dev/null | head -3 | xargs -I{} basename {}
```

Use AskUserQuestion with a **single question**:

**"Which feature should we retrospect on?"** (header: "Feature")
Options:
- The 3 filenames from above (most recent first)
- "Paste the plan — I'll share the markdown in my next message"

After loading the dev plan, also read:
- The matching basic plan (`<feature>.md`) if it exists — for the original architecture intent
- The matching ticket (`<feature>-ticket.md`) if it exists — for the acceptance criteria

---

## Step 2 — Read the Actual Implementation

From the plan's **Files to Create** and **Files to Modify** tables, read the actual files on disk. Build a picture of what was actually implemented — method signatures, data structures, patterns used — without relying on what the plan said should be there.

---

## Step 3 — Compare Plan vs Reality

Work through each dimension below. For each one, note whether reality matched the plan, and if not, describe the drift specifically.

### Architecture drift
Did the implementation follow the data flow described in the Architecture Overview? Note any layers that were added, removed, or rerouted.

### File drift
- Files that were planned but do not exist on disk
- Files that exist but were not in the plan
- Files that were planned under one path but created at a different path

### Scope drift
Did the implementation do more or less than the plan described? Flag:
- Functionality added that wasn't in any slice ("scope creep")
- Planned functionality that was skipped or deferred

### Code skeleton drift
For each code skeleton in the plan, compare the method signature / SQL / template against what was actually written. Note significant differences in approach — not style, but intent.

### Slice accuracy
Were the slice boundaries sensible? Note any slices that turned out to be much larger or smaller than expected, or had to be split/merged during implementation.

### Estimate accuracy
The plan's overall complexity estimate was `<Small / Medium / Large>`. Based on the number of files touched and the apparent scope, was that estimate reasonable? Over/under?

---

## Step 4 — Capture Learnings

Identify up to 5 concrete learnings from the drift analysis. A learning is worth capturing if it would help the *next* feature be planned or implemented better.

For each learning, classify it:

- **Pattern** — a way of doing something that worked well and should be repeated
- **Antipattern** — something the plan assumed that turned out to be wrong
- **CLAUDE.md candidate** — a convention or rule that should be added to the project's CLAUDE.md so future planning is aware of it

**CLAUDE.md candidates** are the most valuable output. Examples of what qualifies:
- "Auth middleware must always be applied at the route level, not the controller — discovered when controller-level approach caused middleware to be skipped in tests"
- "The `Database::runQuery()` method returns `false` on zero affected rows, not `0` — check with `=== false` not `=== 0`"
- "Latte templates cannot access `$_SESSION` directly — pass values from controller as named variables"

Only flag something as a CLAUDE.md candidate if it's a non-obvious rule that the codebase doesn't already document.

---

## Step 5 — Write the Retro File

**Filename:** take the base feature name, append `-retro.md`.
Example: `2026-05-22-user-notifications-dev.md` → `2026-05-22-user-notifications-retro.md`

Write `~/.claude/plans/<filename>-retro.md`:

```markdown
---
date: YYYY-MM-DD
feature: <feature name>
project: <project name>
---

# <Feature Name> — Retrospective

## Summary

<2–3 sentences on whether the implementation matched the plan overall. Was the plan accurate? What was the biggest surprise?>

## Drift Analysis

### Architecture
<Matched / Drifted — describe any changes to the intended data flow>

### Files
<Matched / Drifted — list any files added, missing, or relocated>

### Scope
<Matched / Expanded / Reduced — note any additions or deferred items>

### Estimate
<Accurate / Over-estimated / Under-estimated — brief note on why>

## Slice Retrospective

| Slice | Boundary sensible? | Notes |
|-------|-------------------|-------|
| Slice 1: <name> | ✅ Yes / ⚠️ Too large / ⚠️ Too small | <brief note> |
| Slice 2: <name> | ✅ Yes / ⚠️ ... | <brief note> |

## Learnings

### Patterns (do again)
- <What worked well>

### Antipatterns (avoid next time)
- <What the plan assumed that was wrong>

### CLAUDE.md Candidates
<If none: "None identified.">

- **[Category]:** <Rule or convention worth adding — include the *why*>
- **[Category]:** <...>
```

---

## Step 6 — Surface CLAUDE.md Candidates

If any CLAUDE.md candidates were identified, tell the user explicitly:

> "I found [N] pattern(s) worth adding to your CLAUDE.md:
>
> - [paste each candidate]
>
> Run `/revise-claude-md` to add them, or update CLAUDE.md manually."

If none were found:
> "No new CLAUDE.md patterns identified — the existing conventions covered this feature well."

---

## After Writing

Tell the user:

> "✅ Retrospective saved to `~/.claude/plans/<filename>-retro.md`
>
> Run `/plan-status` to see the full picture of all your features."
