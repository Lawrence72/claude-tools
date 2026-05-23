---
name: plan-ticket
description: Generate a PM-ready ticket from an existing feature plan — choose Generic, Jira, or ADO format. Suitable for any ticket system. Run after /feature-planner, or standalone on any plan file.
tools: AskUserQuestion, Bash, Read, Write
---

# Plan Ticket

You generate a non-technical PM-ready ticket from an existing developer feature plan. The ticket is written for a product manager, stakeholder, or QA engineer — someone who needs to understand *what* is being built and *how to verify it*, without needing to understand the code.

**Announce at start:** "I'm using the plan-ticket skill. I'll generate a PM ticket from your feature plan."

---

## Step 1 — Select the Plan File

List available plan files (exclude `-dev.md` and `-ticket.md` files, as those are already outputs):

```bash
ls ~/.claude/plans/ 2>/dev/null | grep -v '\-dev\.md' | grep -v '\-ticket\.md' | sort -r
```

Use AskUserQuestion with **two questions in a single call**:

**Question 1 — "Which plan should I generate a ticket for?"** (header: "Plan file")
Options: the filenames listed above (most recent first). Include "Let me type the path manually" as an escape hatch.

**Question 2 — "What format should the ticket be in?"** (header: "Ticket format")
Options:
- Generic — user story + acceptance criteria, works in any system
- Jira — Summary, Issue Type, Story Points, Priority, Labels
- ADO (Azure DevOps) — Title, Work Item Type, Effort, Priority, Tags, Definition of Done

After selection, read the chosen plan file in full.

---

## Step 2 — Derive Ticket Content

Before writing, reason through each section. The underlying content is the same regardless of format — only the structure and field names differ.

**User Story:** Identify the end user of this feature (authenticated user, admin, anonymous visitor, background system, etc.) and the concrete action they take. Avoid vague benefits — "so that I can manage my data" is weak; "so that I can see which notifications I haven't read yet" is strong.

**Background / Description:** Explain *why* this is being built now. What gap, pain point, or business need does it address? Write this for someone who doesn't know the codebase — no class names, no file paths, no technical jargon.

**Acceptance Criteria:** Derive from the Verification section of the plan plus any goals stated. Each criterion must be independently verifiable by a tester without reading code. Use Given/When/Then format. Cover:
- The happy path (at least 2 criteria)
- Edge cases (e.g. empty states, permission boundaries)
- Error states (e.g. what happens when input is invalid)

Aim for 4–7 criteria — enough to be thorough, few enough to stay focused.

**Out of Scope / Non-goals:** Look at the plan's Goals section for explicit non-goals. Also infer natural scope boundaries.

**Dependencies:** Check the plan's Context and Architecture sections. If none, state "None".

**Estimate:** Infer from the number of slices (if a `-dev.md` exists), the number of files being created/modified, and the scope. Map to: Small (< 1 day) = 1–2 points, Medium (1–3 days) = 3–5 points, Large (3+ days) = 8+ points.

---

## Step 3 — Write the Ticket File

**Filename:** take the input filename, strip `.md`, append `-ticket.md`.
Example: `2026-05-22-user-notifications.md` → `2026-05-22-user-notifications-ticket.md`

Check for collisions:
```bash
ls ~/.claude/plans/ 2>/dev/null | grep "<derived-slug>-ticket" || echo "clear"
```
If collision, append `-2`, `-3`, etc. before `.md`.

Write `~/.claude/plans/<filename>-ticket.md` using the structure for the chosen format:

---

### Generic format

```markdown
# <Feature Name> — Ticket

## User Story

As a [specific user role], I want to [specific action] so that [concrete, measurable benefit].

## Background

<2–3 sentences. Why this is needed now, what problem it addresses. Non-technical.>

## Acceptance Criteria

- [ ] Given [starting state], when [user action], then [observable outcome]
- [ ] Given [starting state], when [user action], then [observable outcome]
- [ ] Given [edge case], when [user action], then [expected behaviour]
- [ ] Given [error condition], when [user action], then [error outcome the user sees]

## Out of Scope

- <Non-goal>
- <Non-goal>

## Dependencies

- <Dependency or "None">

## Complexity

<Small / Medium / Large> — estimated <N days>
```

---

### Jira format

```markdown
# <Feature Name>

**Issue Type:** Story
**Priority:** <High / Medium / Low>
**Story Points:** <1 / 2 / 3 / 5 / 8 / 13>
**Labels:** <comma-separated relevant labels, e.g. "backend, notifications, auth">
**Components:** <relevant component if known, e.g. "API", "UI", "Auth">

---

## User Story

As a [specific user role], I want to [specific action] so that [concrete, measurable benefit].

## Description

<2–3 sentences of background. Why this is needed now, what problem it addresses. Non-technical.>

## Acceptance Criteria

- [ ] Given [starting state], when [user action], then [observable outcome]
- [ ] Given [starting state], when [user action], then [observable outcome]
- [ ] Given [edge case], when [user action], then [expected behaviour]
- [ ] Given [error condition], when [user action], then [error outcome the user sees]

## Out of Scope

- <Non-goal>

## Dependencies

- <Linked issue or "None">
```

---

### ADO format

```markdown
# <Feature Name>

**Work Item Type:** User Story
**Priority:** <1 (Critical) / 2 (High) / 3 (Medium) / 4 (Low)>
**Effort:** <story point number>
**Tags:** <semicolon-separated tags, e.g. "backend; notifications; auth">
**Area Path:** <team or component area if known>

---

## Description

As a [specific user role], I want to [specific action] so that [concrete, measurable benefit].

<2–3 sentences of background. Why this is needed now, what problem it addresses. Non-technical.>

## Acceptance Criteria

- [ ] Given [starting state], when [user action], then [observable outcome]
- [ ] Given [starting state], when [user action], then [observable outcome]
- [ ] Given [edge case], when [user action], then [expected behaviour]
- [ ] Given [error condition], when [user action], then [error outcome the user sees]

## Definition of Done

- [ ] Code reviewed and approved
- [ ] Acceptance criteria verified by QA
- [ ] No regressions in related features
- [ ] <Any feature-specific done criterion, e.g. "Automated tests passing">

## Out of Scope

- <Non-goal>

## Dependencies

- <Linked work item or "None">
```

---

## After Writing

Tell the user:

> "✅ PM ticket (<format>) saved to `~/.claude/plans/<filename>-ticket.md`
>
> You can paste this directly into <Jira / ADO / your ticket system>.
> Run `/plan-slice` if you also want a developer plan with vertical slices."
