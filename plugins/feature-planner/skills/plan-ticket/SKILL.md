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

**This ticket is written for a PM, stakeholder, or QA engineer — not a developer.** Before writing each section, apply this filter: *"Would a PM who has never seen the codebase write this?"* If the answer is no, reframe or omit it.

### What must never appear in the ticket

The following are developer concerns that belong in the dev plan, not the PM ticket:

- **File paths** — `src/Logic/NotificationService.php`, `public/css/lobby.css` — never reference internal paths
- **Class or method names** — `Sanitizer::cleanText()`, `AuthMiddleware`, `LobbyLogic` — use plain English instead
- **Database table or column names** — `letterbug_startmenu`, `game_settings`, `ban_disallow` — describe the behaviour, not the storage
- **Internal data model details** — "ban type 2", "ban type 3" — describe what the user experiences, not how it's stored
- **Test commands or coverage targets** — `vendor/bin/phpunit`, `--coverage-text`, "100% class coverage" — these go in the DoD of the dev plan only
- **Framework or language names in tags** — `slim-framework`, `php` — tags should be business domain, not tech stack
- **CLAUDE.md or project conventions** — coding rules are for developers, not PMs
- **Internal URLs or environment details** — `http://someurl.com`, `Vagrant` — irrelevant to a PM

### How to translate technical detail into PM language

| Dev plan says | PM ticket says |
|---|---|
| `Sanitizer::cleanText()` applied to message field | User input is stored safely |
| `letterbug_startmenu` table populates dropdown | The game options form loads correctly for all game types |
| ban type 2 / ban type 3 logic | Certain player restrictions prevent joining |
| Auth middleware redirects unauthenticated users | Visitors who are not logged in are redirected to the login page |
| `game_settings` serialized format is preserved | No changes to how game configurations are stored |

### Content derivation rules

**User Story:** Identify the end user (logged-in player, admin, anonymous visitor, etc.) and the concrete action they take. Avoid vague benefits — "so that I can manage my data" is weak; "so that I can easily find opponents" is strong.

**Background / Description:** Explain *why* this is being built now — the gap, pain point, or business need. Two to three sentences maximum. No jargon, no file paths, no class names.

**Acceptance Criteria:** Describe what a tester can observe in the running application — what they see, what happens, what error messages appear. Use Given/When/Then format. Cover the happy path, edge cases, and error states. Aim for 4–7 criteria.

AC must describe user-observable outcomes only. If an AC item references a class name, file path, database table, or internal data field, rewrite it in plain English or drop it — those details are verified by the dev plan, not the ticket.

**Out of Scope:** Explicit non-goals stated in terms of features or user capabilities — not technical implementation decisions. "No changes to the game-start flow" is fine. "The existing serialized `game_settings` format is preserved" is not — that's a dev concern.

**Dependencies:** Other *features or tickets* this depends on — not file paths or code components. If auth must be live before this can be built, the dependency is "User authentication feature" not "`src/Middleware/AuthMiddleware.php`". If there are no feature-level dependencies, write "None".

**Tags / Labels:** Business domain tags only — the feature area, user-facing concept, or product area. Never the programming language, framework, or library.

**Estimate:** Infer from number of slices and scope. Map to: Small (< 1 day) = 1–2 points, Medium (1–3 days) = 3–5 points, Large (3+ days) = 8+ points.

**Definition of Done:** Process and outcome items only — things a PM or QA engineer would verify, not a developer.
✅ Allowed: "All acceptance criteria signed off by QA", "Feature live in staging", "No open bugs"
❌ Not allowed: Test commands, coverage targets, class names, CLAUDE.md conventions — those belong in the dev plan.

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
**Labels:** <comma-separated business domain labels, e.g. "lobby, matchmaking, gameplay" — no framework or language names>
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
**Tags:** <semicolon-separated business domain tags, e.g. "lobby; matchmaking; gameplay" — no framework or language names>
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
- [ ] All acceptance criteria signed off by QA
- [ ] Feature verified in staging environment
- [ ] No open bugs or regressions

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
