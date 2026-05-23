---
name: plan-slice
description: Restructure an existing feature plan into vertical slices with code skeletons — each slice delivers working, testable functionality end-to-end. Run after /feature-planner, or standalone on any plan file.
tools: AskUserQuestion, Glob, Grep, Read, Bash, Write
---

# Plan Slice

You restructure an existing feature plan into vertical slices — ordered increments where each slice delivers something that runs and can be verified end-to-end before moving to the next slice.

**Announce at start:** "I'm using the plan-slice skill. I'll restructure your plan into vertical slices with code skeletons."

---

## Step 1 — Select the Plan File

List available basic plan files (exclude `-dev.md` and `-ticket.md` files that are already outputs):

```bash
ls ~/.claude/plans/ 2>/dev/null | grep -v '\-dev\.md' | grep -v '\-ticket\.md' | sort -r
```

Use AskUserQuestion with a single question to let the user pick which plan to slice. Show the filenames as options (most recent first). Include a "Let me type the path manually" escape hatch.

After selection, read the plan file in full.

---

## Step 2 — Read Supporting Context

Extract the file paths listed under "Files to Create" and "Files to Modify" in the plan. Read those files from the codebase (if they exist) to understand:
- Naming conventions and code style
- Method signatures and patterns already in use
- DB schema conventions
- Test patterns

This context is essential for generating realistic code skeletons.

---

## Step 3 — Determine Slice Boundaries

Analyse the feature end-to-end and identify **3–6 vertical slice boundaries**. Each slice must:

- Deliver a **working, independently testable increment** — not "all DB work" or "all API work"
- Touch all layers required for that increment (DB + service + API + UI as appropriate)
- Be named by what it **delivers**, not which layer it is

Good slice names: "Core data model", "Create flow end-to-end", "List view with filters"
Bad slice names: "Database layer", "Backend", "Frontend"

Consider natural dependencies — a "List view" slice can't come before the "Core data model" slice. Order slices so each one builds on the last.

---

## Step 4 — Write the Sliced Plan

**Filename:** take the input filename, strip `.md`, append `-dev.md`.
Example: `2026-05-22-user-notifications.md` → `2026-05-22-user-notifications-dev.md`

Check for collisions:
```bash
ls ~/.claude/plans/ 2>/dev/null | grep "<derived-slug>-dev" || echo "clear"
```
If collision, append `-2`, `-3`, etc. before `.md`.

Write `~/.claude/plans/<filename>-dev.md` using this structure:

```markdown
---
date: YYYY-MM-DD
project: <project name from original plan>
feature: <feature name from original plan>
type: vertical-slice-plan
---

# <Feature Name> — Developer Plan (Vertical Slices)

> Implement slices in order. Each slice should pass its own verification before moving to the next.

**Slices:** <N total> | **Estimated:** <X days>

---

## Slice 1: <Name — describes what is delivered>

**Delivers:** <One sentence: what a tester/developer can verify once this slice is complete>
**Layers:** <e.g. DB + Service, or API + UI + Tests>

- [ ] <Sub-task — be specific enough to execute without looking elsewhere>

  ```<lang>
  <Key method signature, SQL statement, or template snippet>
  <Show the critical logic structure — not boilerplate, just the decision points>
  ```

- [ ] <Next sub-task with skeleton if it involves meaningful choices>
- [ ] <Test sub-task — what to write and what to verify>
- [ ] Verify: <How to confirm this slice works in isolation before starting the next>
- [ ] Commit: `feat: <short description>`

---

## Slice 2: <Name>

**Delivers:** <...>
**Layers:** <...>

- [ ] <Sub-tasks with skeletons>
- [ ] Verify: <...>
- [ ] Commit: `feat: <...>`

<Repeat for all slices. Aim for 3–6.>

---

## Full Verification

<Copy the Verification section from the original plan — this is the end-to-end acceptance check once all slices are complete.>
```

---

## After Writing

Tell the user:

> "✅ Developer plan saved to `~/.claude/plans/<filename>-dev.md`
>
> The original plan is unchanged at `~/.claude/plans/<original-filename>.md`.
> Run `/plan-ticket` if you also want a PM ticket for this feature."
