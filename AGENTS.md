# Copilot Agent Instructions

This file is read at the start of every agent session.
It defines the spec-driven development workflow, instruction routing, and hard constraints for this repository.

---

## Instruction architecture

This repository uses a layered instruction system. Before doing any work, understand which layer applies:

| Layer | File(s) | Loaded when |
|-------|---------|-------------|
| Always | `copilot-instructions.md` | Every request |
| Always | This file (`AGENTS.md`) | Every agent session |
| Auto by file | `.github/instructions/testing/unit-test.instructions.md` | Writing `*Test.kt` files |
| Auto by file | `.github/instructions/testing/integration-test.instructions.md` | Writing `*IT.kt` files |
| Auto by file | `.github/instructions/testing/cucumber-bdd.instructions.md` | Writing `*.feature` or `**/steps/**/*.kt` files |
| Auto by file | `.github/instructions/backend/api-rules.instructions.md` | Writing `**/controller/**/*.kt` files |
| Auto by file | `.github/instructions/backend/jooq.instructions.md` | Writing `**/repository/**/*.kt` files |
| On demand | `.github/skills/unit-test/SKILL.md` | When writing unit tests — load for examples and templates |
| On demand | `.github/skills/integration-test/SKILL.md` | When writing integration tests — load for examples and templates |
| On demand | `.github/skills/cucumber-bdd/SKILL.md` | When writing BDD scenarios or step definitions — load for examples |
| On demand | `.github/skills/jooq/SKILL.md` | When writing jOOQ queries — load for DSL patterns and examples |
| On demand | `.github/skills/api-rules/SKILL.md` | When writing controllers or API contracts — load for examples |

**Rule:** Load skills when generating or reviewing code for that domain. Do not load all skills at once — only load what is relevant to the current file or task.

---

## Spec-driven development workflow

All feature development in this repository follows a strict four-phase workflow: **PLAN → REVIEW → REFINE → EXECUTE**. Do not skip phases. Do not write implementation code before a spec exists and has been approved.

---

### Phase 1 — PLAN

Triggered when the developer pastes a JIRA ticket or feature description into the chat.

**Step 1 — Understand the requirement**

Read the ticket carefully. Before doing anything else:
- Identify the functional goal, affected users, and acceptance criteria.
- If any of the following are unclear or missing, ask your questions in a single batch before proceeding. Do not ask one question at a time:
  - Scope boundaries (what is explicitly out of scope?)
  - Edge cases not covered by the description
  - Acceptance criteria that are ambiguous or missing
  - Which user roles or permissions are involved
  - Any external system dependencies

Do not guess. Do not assume. If something is ambiguous, ask.

**Step 2 — Analyse the codebase**

Once the requirement is clear, search the codebase to understand the impact:
- Identify every file that will need to change and why.
- Identify every file that could be affected indirectly (shared services, interfaces, constants).
- Map any external dependencies (libraries, APIs, configuration) that need to be added or changed.
- If you find anything in the codebase that conflicts with the requirement or is unclear, ask a second batch of questions before proceeding.

**Step 3 — Write the spec file**

Create a new file at `specs/JIRA-[ID].md` using the spec template below. Fill every section. Do not leave sections empty — if a section does not apply, write "N/A" and a one-line reason.

Do not write any implementation code during this phase. Code snippets in the spec are illustrative only.

**Spec file template:**

```markdown
# JIRA-[ID]: [Title]

**Status:** Draft
**Author:** Copilot + [Developer name]
**Created:** [Date]
**Last updated:** [Date]

---

## Summary

[2–3 sentence plain-English description of what this feature does and why.]

---

## Requirement source

[Paste the original JIRA description or a link to the ticket here.]

---

## Files to modify

| File path | Type of change | Reason |
|-----------|---------------|--------|
| | | |

---

## New files to create

| File path | Purpose |
|-----------|---------|
| | |

---

## Key code changes

[Illustrative snippets showing the shape of important changes. Not final implementation — for alignment only.]

---

## External dependencies

| Dependency | Version | Purpose | Added to |
|------------|---------|---------|---------|
| | | | |

---

## Test plan

### Unit tests
[List the unit test cases to be written. Include happy path, edge cases, and error cases.]

### Integration tests
[List the integration test scenarios.]

### BDD scenarios
[List the Gherkin scenario titles to be written.]

---

## Implementation checklist

Work through these in order. Tick each item as it is completed.

- [ ] [Step 1]
- [ ] [Step 2]
- [ ] [Step 3]
- [ ] Unit tests written and passing
- [ ] Integration tests written and passing
- [ ] BDD scenarios written and passing
- [ ] No new compiler warnings introduced
- [ ] PR description references this spec file
```

---

### Phase 2 — REVIEW

The developer reads the spec file. Do not proceed to execution until the developer explicitly says the spec is approved.

During review, the developer may:
- Ask questions about specific decisions in the spec
- Point out missing scenarios or files
- Challenge assumptions made during codebase analysis

Answer all questions clearly. If a question reveals a gap in the spec, update the spec file immediately and note the change.

---

### Phase 3 — REFINE

If the developer requests changes to the spec:
- Edit the spec file directly.
- Update the status field from `Draft` to `In Review`.
- Update the `Last updated` date.
- Never create a new spec file for the same ticket — always update the existing one.
- After changes, summarise what was changed and ask the developer to confirm before moving to execution.

Repeat this phase as many times as needed until the developer explicitly approves the spec.

---

### Phase 4 — EXECUTE

Triggered when the developer says the spec is approved and asks for implementation to begin.

**Before writing any code:**
- Re-read the approved spec file in full.
- Load all relevant skill files for the file types you will be creating.
- Confirm you understand the full checklist.

**During execution:**
- Work through the implementation checklist in order. Do not jump ahead.
- After completing each checklist item, tick it off in the spec file.
- After each logical unit of work (e.g. a service class, a repository method), run the relevant tests and fix any failures before moving on.
- If you encounter something not covered by the spec — a conflict, an unexpected dependency, a missing case — stop and ask the developer before continuing.
- Do not make architectural decisions not covered in the spec without asking.

**When execution is complete:**
- Update the spec status to `Implemented`.
- Summarise what was built, what tests were added, and any deviations from the original spec (with reasons).

---

## Hard constraints

These rules apply at all times regardless of what the developer asks:

- Never query the database directly. All database access goes through the repository layer using jOOQ.
- Never use Spring Data JPA. This project uses jOOQ exclusively.
- Never expose internal entity IDs in API responses.
- Never write implementation code before a spec file exists for that ticket.
- Never modify files outside the scope defined in the spec without asking first.
- Never combine multiple JIRA tickets into one spec file.
- Never skip tests. Every new feature must have unit tests, integration tests, and BDD scenarios unless the spec explicitly states otherwise with a reason.
- Never hardcode credentials, secrets, or environment-specific values. Use configuration properties.

---

## Commit conventions

Follow this format for all commits:

```
[JIRA-ID] <type>: <short description>

<optional body: what changed and why>

Spec: specs/JIRA-[ID].md
```

Types: `feat`, `fix`, `test`, `refactor`, `chore`, `docs`

Example:
```
[JIRA-123] feat: add user preference endpoint

Adds GET /users/{id}/preferences returning the user's stored
notification and display preferences.

Spec: specs/JIRA-123.md
```

---

## Asking questions

When you need to ask questions:
- Always batch questions together. Never ask one question, wait for an answer, then ask another.
- Number your questions so the developer can answer by number.
- Group related questions under a heading.
- After the developer answers, confirm your understanding before proceeding.

---

## Spec file location

All spec files live in `/specs/`. The naming convention is `JIRA-[ID].md` where `[ID]` is the full JIRA ticket identifier (e.g. `JIRA-1234.md`, `PAY-567.md`).

Never delete spec files. They are the permanent record of decisions made for each feature.
