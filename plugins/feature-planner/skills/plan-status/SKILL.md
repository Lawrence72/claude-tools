---
name: plan-status
description: Dashboard view of all feature plans in ~/.claude/plans/ — shows which artefacts exist for each feature, task completion progress, and overall feature state at a glance.
tools: Bash, Read
---

# Plan Status

You produce a dashboard of all feature plans saved in `~/.claude/plans/`. For each feature, you show which planning artefacts exist, how far through implementation it is, and its overall state.

**Announce at start:** "I'm using the plan-status skill. Scanning your plans..."

---

## Step 1 — Discover All Features

List all `.md` files in `~/.claude/plans/`:

```bash
ls ~/.claude/plans/*.md 2>/dev/null | sort -r
```

Group files by base feature slug — strip the known suffixes (`-dev`, `-ticket`, `-retro`) and the date prefix to identify the feature name. Each unique base slug is one feature.

Example grouping:
```
2026-05-22-user-notifications.md
2026-05-22-user-notifications-dev.md
2026-05-22-user-notifications-ticket.md
2026-05-22-user-notifications-retro.md
```
→ Feature: `user-notifications` (2026-05-22)

---

## Step 2 — Assess Each Feature

For each feature, determine:

**Artefacts present:**
- Basic plan (`.md`) — ✅ exists / ❌ missing
- Dev plan (`-dev.md`) — ✅ exists / ❌ missing
- Ticket (`-ticket.md`) — ✅ exists / ❌ missing
- Retro (`-retro.md`) — ✅ exists / ❌ missing

**Implementation progress** (only if `-dev.md` exists):

```bash
grep -c '\- \[x\]' ~/.claude/plans/<feature>-dev.md 2>/dev/null || echo "0"
grep -c '\- \[ \]' ~/.claude/plans/<feature>-dev.md 2>/dev/null || echo "0"
```

Calculate: `done / (done + remaining)` tasks.

**Overall state** — derive from the artefacts and progress:

| State | Condition |
|-------|-----------|
| 📋 **Planned** | Basic plan exists, no dev plan |
| 🔪 **Sliced** | Dev plan exists, 0 tasks done |
| 🔄 **In progress** | Dev plan exists, some tasks done but not all |
| ✅ **Implemented** | Dev plan exists, all tasks done (`- [x]`), no retro |
| 🎉 **Complete** | Dev plan exists, all tasks done, retro exists |

---

## Step 3 — Display the Dashboard

Print the dashboard in a clean, scannable format:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Plan Status — ~/.claude/plans/
  <N> features  ·  <today's date>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎉 user-notifications  (2026-05-22)
   Plan ✅  Dev ✅  Ticket ✅  Retro ✅
   Tasks: 12/12 done

🔄 game-lobby-filter  (2026-05-10)
   Plan ✅  Dev ✅  Ticket ✅  Retro ❌
   Tasks: 5/8 done  ████████░░░░  63%

📋 password-reset  (2026-05-03)
   Plan ✅  Dev ❌  Ticket ❌  Retro ❌
   Not yet sliced — run /plan-slice to continue

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Legend: 📋 Planned  🔪 Sliced  🔄 In progress
          ✅ Implemented  🎉 Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `~/.claude/plans/` is empty or contains no recognisable plan files:
> "No plans found in `~/.claude/plans/`. Run `/feature-planner` to create your first plan."

---

## Step 4 — Surface Next Actions

After the dashboard, print a brief "what to do next" for any in-progress or stalled features:

```
Suggested next steps:
  game-lobby-filter  →  3 tasks remaining — run /plan-implement to continue
  password-reset     →  not sliced — run /plan-slice first
```

Only show this section if there are features that are not yet complete.
