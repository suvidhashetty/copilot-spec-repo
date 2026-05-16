# Spec-driven development with GitHub Copilot — full design summary

This document captures the complete architecture, decisions, and file contents discussed for implementing a spec-driven development workflow using GitHub Copilot. It is intended to be used as a prompt for a Copilot or LLM model to generate the repository skeleton.

---

## Context and goal

The goal is to implement a structured **PLAN → REVIEW → REFINE → EXECUTE** development workflow inside GitHub Copilot, driven by JIRA tickets, that enforces consistent code generation across a Kotlin/Spring Boot/jOOQ/Maven codebase.

The workflow:
1. Developer pastes a JIRA ticket into Copilot
2. Copilot understands the requirement, asks clarifying questions if needed
3. Copilot analyses the codebase to find all affected files
4. Copilot produces a spec markdown file with a full implementation plan, file list, test plan, and checklist
5. Developer reviews and refines the spec with Copilot until it is approved
6. Developer triggers execution — Copilot implements everything following the spec checklist

---

## Workflow diagram (ASCII representation)

```
┌─────────────────────────────────────────────────────────────────┐
│                        JIRA TICKET (input)                      │
│              Developer pastes description into Copilot          │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌──────────────────────── PLAN ───────────────────────────────────┐
│  1. Understand requirement — ask clarifying questions (batched) │
│  2. Analyse codebase — find all affected files, dependencies    │
│  3. Write spec file → specs/JIRA-[ID].md                       │
│     (files to modify, new files, snippets, test plan, checklist)│
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────── REVIEW ──────────────────────────────────┐
│  Developer reads the spec file                                  │
│  Asks questions, flags gaps, challenges assumptions             │
└──────────────────┬───────────────────────┬──────────────────────┘
                   │                       │
            changes needed?          approved?
                   │                       │
                   ▼                       ▼
┌────────────── REFINE ──────┐   ┌──── EXECUTE ────────────────────┐
│ Edit spec file directly    │   │ Re-read approved spec in full   │
│ Update status: In Review   │   │ Load relevant skills per file   │
│ Summarise changes          │   │ Work checklist in order         │
│ Re-confirm with developer  │   │ Tick items off as completed     │
│         │                  │   │ Run tests after each unit       │
│         └── back to REVIEW │   │ Stop and ask if spec is silent  │
└────────────────────────────┘   │ Update status: Implemented      │
                                 └─────────────────────────────────┘
                                              │
                                              ▼
                                 ┌────────────────────────┐
                                 │  PR ready for review   │
                                 │  Code + tests +        │
                                 │  updated spec file     │
                                 └────────────────────────┘
```

---

## Architecture: the four-layer instruction system

Copilot instructions are split across four layers. Each layer has a specific responsibility. **Nothing lives in two layers — no duplication.**

```
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — Organisation (GitHub org settings UI)                        │
│  Security baselines, compliance rules, applies to every repo            │
│  → Configured in GitHub org settings, not a file                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │ always inherited
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 2 — Always loaded (repo-wide)                                    │
│                                                                         │
│  copilot-instructions.md   → stack, package structure, hard rules      │
│  AGENTS.md                 → workflow contract, commit rules, routing   │
│                                                                         │
│  Rule: lean. No examples. No templates. Rules only.                     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │ auto-loaded by file match
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 3 — Path-scoped instructions (*.instructions.md with applyTo)    │
│                                                                         │
│  unit-test.instructions.md         applyTo: **/*Test.kt                │
│  integration-test.instructions.md  applyTo: **/*IT.kt                  │
│  cucumber-bdd.instructions.md      applyTo: **/*.feature + steps       │
│  api-rules.instructions.md         applyTo: **/controller/**/*.kt      │
│  jooq.instructions.md              applyTo: **/repository/**/*.kt      │
│                                                                         │
│  Rule: rules only. No code examples. No templates.                      │
│  The applyTo glob is enforced by the IDE — automatic, not optional.     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │ on-demand / progressive
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 4 — Skills and prompts                                           │
│                                                                         │
│  Skills (.github/skills/*/SKILL.md)                                    │
│    unit-test/SKILL.md          → JUnit5 templates, real examples       │
│    integration-test/SKILL.md   → Testcontainers setup, examples        │
│    cucumber-bdd/SKILL.md       → Gherkin examples, step templates      │
│    jooq/SKILL.md               → DSL patterns, query examples          │
│    api-rules/SKILL.md          → Controller templates, response shapes  │
│                                                                         │
│  Prompts (.github/prompts/)                                             │
│    plan-jira.prompt.md         → invoke to start PLAN phase            │
│    refine-spec.prompt.md       → invoke during REVIEW/REFINE           │
│    execute.prompt.md           → invoke to start EXECUTE phase         │
│                                                                         │
│  Rule: all code examples and templates live here, not in instructions.  │
│  Skills are loaded progressively — only when relevant to the task.     │
└─────────────────────────────────────────────────────────────────────────┘
```

### The single most important maintenance rule

> Nothing lives in two places. Rules in instructions. Examples in skills. No duplication between layers. When a new convention is introduced, update the relevant `.instructions.md` in the same PR. When a new pattern is established, add an example to the relevant `SKILL.md`.

---

## Key design decisions and rationale

### Why AGENTS.md alongside copilot-instructions.md?

`AGENTS.md` is officially supported by GitHub Copilot's coding agent (confirmed November 2025). It is read at the start of every agent session and treated as a hard constraint. `copilot-instructions.md` is the always-on IDE context. The two serve different purposes:

- `copilot-instructions.md` → what the codebase is (stack, structure, rules)
- `AGENTS.md` → how to work in it (workflow, phases, routing map, commit format)

### Why split instructions from skills?

Instructions files (`*.instructions.md`) are loaded automatically based on the file being written. They must be lean — only rules — because they consume context on every matching request.

Skills (`SKILL.md`) are loaded on demand and progressively. They are where the rich content lives: real code examples from your codebase, templates, pattern references. Loading a skill only happens when Copilot determines it is relevant, so the context cost is only paid when needed.

Mixing examples into instruction files causes two problems: bloated context on every request, and examples becoming stale as the codebase evolves without being updated.

### Why not use chatmodes?

Chat modes (`.github/chatmodes/*.chatmode.md`) are VS Code-only. They do not work in JetBrains or other IDEs. The workflow enforcement is already handled by `AGENTS.md` and the prompts, so chat modes would be redundant duplication. They were considered and explicitly rejected in favour of the prompts approach which works across all IDEs.

### Why separate unit test and integration test instructions?

The `applyTo` glob disambiguates them by filename suffix: `*Test.kt` for unit tests, `*IT.kt` for integration tests (Spring Boot convention). Without this naming discipline, the wrong instruction file loads. The naming convention is therefore a hard requirement, not a suggestion.

### Why .copilotignore?

jOOQ generates classes from the database schema. Without `.copilotignore`, Copilot uses those generated files as context for completions, producing code that matches generated accessor patterns rather than handwritten idiomatic patterns. Ignoring generated sources forces Copilot to learn from real code only.

---

## Complete repository structure to generate

```
repo-root/
├── AGENTS.md
├── README.md
├── .copilotignore
├── specs/
│   └── JIRA-EXAMPLE.md                        ← example spec showing the template
└── .github/
    ├── copilot-instructions.md
    ├── instructions/
    │   ├── testing/
    │   │   ├── unit-test.instructions.md
    │   │   ├── integration-test.instructions.md
    │   │   └── cucumber-bdd.instructions.md
    │   └── backend/
    │       ├── api-rules.instructions.md
    │       └── jooq.instructions.md
    ├── skills/
    │   ├── unit-test/
    │   │   └── SKILL.md
    │   ├── integration-test/
    │   │   └── SKILL.md
    │   ├── cucumber-bdd/
    │   │   └── SKILL.md
    │   ├── jooq/
    │   │   └── SKILL.md
    │   └── api-rules/
    │       └── SKILL.md
    └── prompts/
        ├── plan-jira.prompt.md
        ├── refine-spec.prompt.md
        └── execute.prompt.md
```

---

## File contents

### `AGENTS.md` (production-ready, no placeholders)

```markdown
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

~~~markdown
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
~~~

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

```
[JIRA-ID] <type>: <short description>

<optional body: what changed and why>

Spec: specs/JIRA-[ID].md
```

Types: `feat`, `fix`, `test`, `refactor`, `chore`, `docs`

---

## Asking questions

When you need to ask questions:
- Always batch questions together. Never ask one question, wait for an answer, then ask another.
- Number your questions so the developer can answer by number.
- Group related questions under a heading.
- After the developer answers, confirm your understanding before proceeding.

---

## Spec file location

All spec files live in `/specs/`. The naming convention is `JIRA-[ID].md` where `[ID]` is the full JIRA ticket identifier.

Never delete spec files. They are the permanent record of decisions made for each feature.
```

---

### `.github/copilot-instructions.md` (fill in REPLACE markers)

```markdown
# Project conventions

This file is loaded on every Copilot request. Keep it lean — only universal rules that apply to every file.
Task-specific rules live in `.github/instructions/`. Examples and templates live in `.github/skills/`.

## Stack

- Language: Kotlin [REPLACE: version]
- Build: Maven [REPLACE: version]
- Framework: Spring Boot [REPLACE: version]
- Database access: jOOQ only — no Spring Data JPA, no raw JDBC, no native queries
- Unit testing: JUnit 5 + MockK
- Integration testing: [REPLACE: e.g. Spring Boot Test + Testcontainers]
- BDD: Cucumber with Gherkin

## Package structure

[REPLACE: paste your actual package structure]

## Naming conventions

- Controllers: [Domain]Controller.kt
- Services: [Domain]Service.kt
- Repositories: [Domain]Repository.kt
- Unit tests: [ClassName]Test.kt       ← must end in Test for applyTo glob to work
- Integration tests: [ClassName]IT.kt  ← must end in IT for applyTo glob to work
- Feature files: [domain]-[scenario].feature

## Non-negotiable rules

- All database access through the repository layer using jOOQ. Never call DB directly from service or controller.
- Never return internal entity IDs in API responses.
- All configuration values in application.yml. No hardcoded values.
- No System.out.println — use SLF4J.
- [REPLACE: add your own rules]

## What NOT to put in this file

- Test conventions → .github/instructions/testing/
- API rules → .github/instructions/backend/api-rules.instructions.md
- jOOQ patterns → .github/instructions/backend/jooq.instructions.md
- Code examples → .github/skills/
- Workflow → AGENTS.md
```

---

### `.github/instructions/testing/unit-test.instructions.md`

```markdown
---
applyTo: "**/*Test.kt"
---

# Unit test conventions

These rules apply whenever a *Test.kt file is being written or reviewed.
For examples and templates, load `.github/skills/unit-test/SKILL.md`.

## Framework and imports

[REPLACE: list exact imports and annotations required in every unit test]
- Use JUnit 5 (org.junit.jupiter.api.*) — no JUnit 4 annotations
- Use MockK for mocking (io.mockk.*) — no Mockito
- Annotate the class with @ExtendWith(MockKExtension::class)

## Test naming

[REPLACE: define your naming convention]
- Method name format: should_[expectedBehaviour]_when_[condition]
- Use @DisplayName for human-readable descriptions

## Structure rules

[REPLACE: define structural rules]
- One assertion per test
- Arrange / Act / Assert sections separated by blank lines
- No logic in tests (no loops, no conditionals)
- Each test must be independent — no shared mutable state

## What to test

[REPLACE: define coverage expectations]
- Happy path
- All edge cases mentioned in the spec
- All error/exception paths
- Boundary values

## What NOT to do

[REPLACE: list anti-patterns]
- Do not test private methods directly
- Do not use Thread.sleep
- Do not mock value objects — construct them directly
```

---

### `.github/instructions/testing/integration-test.instructions.md`

```markdown
---
applyTo: "**/*IT.kt"
---

# Integration test conventions

These rules apply whenever a *IT.kt file is being written or reviewed.
For examples and templates, load `.github/skills/integration-test/SKILL.md`.

## Framework and setup

[REPLACE: define integration test setup]
- Use @SpringBootTest with appropriate slice annotation
- Use Testcontainers for database-dependent tests
- Database state must be reset between tests

## Test slice guidance

[REPLACE: define which slice to use when]
- Controller layer only → @WebMvcTest
- Full stack with database → @SpringBootTest + Testcontainers
- Repository layer only → @DataJdbcTest

## Naming and structure

[REPLACE: define naming rules]
- Class name format: [ClassName]IT
- Method name format: should_[behaviour]_when_[condition]

## Data setup

[REPLACE: define test data rules]
- Use test data builders — do not construct domain objects inline
- Test data builders live in src/test/kotlin/.../testdata/
- Do not use production seed data

## What NOT to do

[REPLACE: list anti-patterns]
- Do not use H2 — use Testcontainers with the real database engine
- Do not share test state between test methods
- Do not make real HTTP calls to external services — use WireMock
```

---

### `.github/instructions/testing/cucumber-bdd.instructions.md`

```markdown
---
applyTo: "**/*.feature, **/steps/**/*.kt, **/stepdefs/**/*.kt"
---

# Cucumber BDD conventions

These rules apply to .feature files and step definition Kotlin files.
For examples and templates, load `.github/skills/cucumber-bdd/SKILL.md`.

## Gherkin writing rules

[REPLACE: define your Gherkin standards]
- Given for preconditions, When for action, Then for outcome
- Use And to chain — never repeat Given/When/Then in sequence
- One scenario = one behaviour. More than ~6 steps → split it.

## Scenario naming

[REPLACE: define naming conventions]
- Readable as plain English sentences
- Format: [Actor] [action] [outcome]

## Feature file structure

[REPLACE: define file-level structure]
- One feature file per domain concept, not per endpoint
- Feature block must explain business value in 1–2 sentences
- Use Background: for preconditions shared across all scenarios

## Step definition rules

[REPLACE: define step definition standards]
- Step definitions live in src/test/kotlin/.../steps/
- One file per feature domain
- Steps call service or API layer — never access database directly from a step
- Reuse existing step definitions before writing new ones

## What NOT to do

[REPLACE: list anti-patterns]
- Do not write implementation details in Gherkin
- Do not use But — rephrase as Then or And
- Do not have a single scenario cover multiple user journeys
```

---

### `.github/instructions/backend/api-rules.instructions.md`

```markdown
---
applyTo: "**/controller/**/*.kt, **/api/**/*.kt"
---

# API design rules

These rules apply to all controller and API layer files.
For examples and templates, load `.github/skills/api-rules/SKILL.md`.

## Response envelope

[REPLACE: define your standard response wrapper]
All API responses must be wrapped in the standard envelope.
Never return a raw domain object or bare list directly from a controller.

## HTTP status codes

[REPLACE: define which status codes to use and when]
- 200 OK — successful GET, successful update
- 201 Created — successful resource creation (include Location header)
- 204 No Content — successful DELETE
- 400 Bad Request — validation failure (include field-level errors)
- 401 Unauthorized — missing or invalid auth token
- 403 Forbidden — authenticated but not authorised
- 404 Not Found — resource does not exist
- 409 Conflict — duplicate resource or state conflict
- 422 Unprocessable Entity — business rule violation
- 500 Internal Server Error — never expose stack traces

## Versioning

[REPLACE: define your versioning strategy]
- All endpoints versioned via URL path: /api/v1/...

## Mandatory annotations

[REPLACE: list required annotations on every endpoint]
- @Operation(summary = "...") — OpenAPI summary
- @ApiResponse annotations for all possible status codes
- @PreAuthorize(...) — explicit authorisation rule on every endpoint

## Validation

[REPLACE: define validation rules]
- Use @Valid on request bodies — never validate manually in the controller
- Validation annotations on request DTO, not domain model
- Controllers must not contain business logic — delegate to service layer

## What NOT to do

[REPLACE: list anti-patterns]
- Do not expose internal database IDs in responses
- Do not put business logic in controllers
- Do not catch generic Exception in controllers — use @ControllerAdvice
```

---

### `.github/instructions/backend/jooq.instructions.md`

```markdown
---
applyTo: "**/repository/**/*.kt, **/db/**/*.kt"
---

# jOOQ conventions

These rules apply to all repository and database layer files.
For DSL patterns and query examples, load `.github/skills/jooq/SKILL.md`.

## DSL style

[REPLACE: define your preferred jOOQ DSL style]
- Prefer selectFrom(TABLE) for simple single-table queries
- Use select(...fields).from(TABLE) for projections
- Avoid inline magic strings — use generated DSL constants exclusively

## Generated classes

[REPLACE: define where generated classes live and how they are used]
- Generated jOOQ classes live in [REPLACE: your generated sources path]
- Always use generated table and field references — never reference column names as strings
- Generated classes are read-only — never edit them manually

## Pagination

[REPLACE: define your pagination pattern]
- All list queries must support pagination via limit and offset
- Always execute a count query alongside the data query
- Default page size: [REPLACE]
- Maximum page size: [REPLACE]

## Transaction handling

[REPLACE: define transaction rules]
- Transactions managed at the service layer, not the repository layer
- Never annotate repository methods with @Transactional

## Error handling

[REPLACE: define how database errors should be handled]
- Catch DataAccessException at repository layer and translate to domain exceptions
- Never let jOOQ or JDBC exceptions propagate to service or controller layer

## What NOT to do

[REPLACE: list anti-patterns]
- Do not write raw SQL strings — use the jOOQ DSL
- Do not call DSLContext from outside the repository layer
- Do not perform N+1 queries — use joins or batch fetching
- Do not use fetchOne() when result could be null — use fetchOneInto() or map explicitly
```

---

### `.github/skills/unit-test/SKILL.md` (structure — fill with real examples)

```markdown
# Unit test skill

Load this skill when writing or reviewing *Test.kt files.

## Anatomy of a unit test in this project

[REPLACE: paste a real, representative unit test from your codebase.
Annotate the important parts — MockK setup, naming, assertion style.]

## Test data construction

[REPLACE: show how test objects are constructed — test data builders, fixtures, factory methods.]

## Common MockK patterns

[REPLACE: add the MockK patterns your team uses most — coroutines, slots, verify ordering, relaxed mocks.]

## Reference

[REPLACE: link to your internal testing wiki or standards doc.]
```

---

### `.github/skills/integration-test/SKILL.md` (structure — fill with real examples)

```markdown
# Integration test skill

Load this skill when writing or reviewing *IT.kt files.

## Standard integration test setup

[REPLACE: paste a representative integration test showing full setup —
Spring slice, Testcontainers config, data setup, assertion style.]

## Testcontainers setup

[REPLACE: show how Testcontainers is configured — shared container, lifecycle, datasource override.]

## Test data setup patterns

[REPLACE: show how integration tests create and clean up test data.]

## WireMock / external service stubbing

[REPLACE: show the stubbing pattern for external HTTP calls if applicable.]

## Reference

[REPLACE: link to your internal integration testing standards doc.]
```

---

### `.github/skills/cucumber-bdd/SKILL.md` (structure — fill with real examples)

```markdown
# Cucumber BDD skill

Load this skill when writing .feature files or step definition files.

## Feature file template

[REPLACE: paste a real feature file from your project as the reference example.]

## Step definition template

[REPLACE: paste a representative step definition class from your project.]

## Shared step definitions

[REPLACE: list shared steps already defined in the project that can be reused.]

## Reference

[REPLACE: links to your internal BDD wiki and any team decisions about BDD scope.]
```

---

### `.github/skills/jooq/SKILL.md` (structure — fill with real examples)

```markdown
# jOOQ skill

Load this skill when writing or reviewing repository layer files.

## Standard repository structure

[REPLACE: paste a representative repository class — constructor injection of DSLContext,
method signatures, error handling, mapping approach.]

## Query patterns

[REPLACE: add the jOOQ query patterns used most frequently —
joins, aggregations, batch inserts, upserts, pagination with count.]

## Result mapping

[REPLACE: show how jOOQ results are mapped to domain objects —
custom record mapper, fetchInto, or manual mapping.]

## Generated DSL reference

[REPLACE: list the main generated tables and records used in this project.]

## Reference

[REPLACE: link to jOOQ version docs and internal wiki.]
```

---

### `.github/skills/api-rules/SKILL.md` (structure — fill with real examples)

```markdown
# API rules skill

Load this skill when writing or reviewing controller files.

## Standard controller template

[REPLACE: paste a representative controller showing full structure —
annotations, response wrapping, error handling delegation, OpenAPI annotations.]

## Request/response DTO pattern

[REPLACE: show how request and response DTOs are structured —
validation annotations, mapping from domain objects.]

## Standard error response

[REPLACE: show your @ControllerAdvice error handler or error response structure.]

## OpenAPI documentation conventions

[REPLACE: show project-specific OpenAPI annotation patterns — tags, security schemes.]

## Reference

[REPLACE: link to your internal API design guide and Swagger UI URL.]
```

---

### `.github/prompts/plan-jira.prompt.md`

```markdown
# Plan — JIRA ticket

Use this prompt to kick off the PLAN phase. Invoke with /plan-jira then paste the ticket.

---

You are a senior engineer on this codebase. I am going to paste a JIRA ticket.
Follow the PLAN phase defined in AGENTS.md exactly:

1. Read the ticket and identify anything ambiguous. Ask all questions at once before doing anything else.
2. Once the requirement is clear, analyse the codebase to map every file that will need to change.
3. Write the spec file to specs/[JIRA-ID].md using the template in AGENTS.md.

Do not write any implementation code. The only output of this phase is the spec file.

---

[PASTE JIRA TICKET HERE]
```

---

### `.github/prompts/refine-spec.prompt.md`

```markdown
# Refine spec

Use this prompt during REVIEW and REFINE phases. Invoke with /refine-spec.

---

We are in the REFINE phase for specs/[JIRA-ID].md.

I will describe the changes needed. You will:
1. Update the spec file directly — never create a new file.
2. Update the Status field to In Review and the Last updated date.
3. After making changes, summarise exactly what was changed and why.
4. Ask me to confirm before moving to execution.

Do not write any implementation code during this phase.

---

[DESCRIBE WHAT NEEDS TO CHANGE IN THE SPEC]
```

---

### `.github/prompts/execute.prompt.md`

```markdown
# Execute

Use this prompt to begin EXECUTE phase once the spec is approved. Invoke with /execute.

---

The spec at specs/[JIRA-ID].md is approved. Begin implementation.

Follow the EXECUTE phase defined in AGENTS.md exactly:
1. Re-read the spec file in full before writing any code.
2. Load all relevant skill files for the file types you will be creating.
3. Work through the implementation checklist in order — do not skip steps.
4. Tick off each checklist item in the spec file as you complete it.
5. Run tests after each logical unit of work and fix failures before continuing.
6. If you encounter anything not covered by the spec, stop and ask before continuing.

When complete, update spec status to Implemented and summarise what was built.
```

---

### `.copilotignore`

```
# Exclude jOOQ generated classes — prevents generated code from polluting suggestions
# REPLACE: update this path to match your actual generated sources output directory
target/generated-sources/jooq/

# Build output
target/
build/

# REPLACE: add any other generated file paths specific to your project
```

---

## What to fill in after generating the skeleton

Every `[REPLACE: ...]` marker in the skeleton is something only the team knows. Work through them in this order for maximum immediate value:

| Priority | File | What to add | Time estimate |
|----------|------|-------------|---------------|
| 1 | `copilot-instructions.md` | Actual stack versions, package structure, your non-negotiable rules | 30 min |
| 2 | `unit-test.instructions.md` | Your JUnit5/MockK conventions, naming rules, anti-patterns | 30 min |
| 3 | `unit-test/SKILL.md` | Paste 2–3 real unit tests from the codebase, annotated | 45 min |
| 4 | `jooq.instructions.md` | Your DSL style decisions, pagination pattern, transaction rules | 30 min |
| 5 | `jooq/SKILL.md` | Paste a real repository class showing your patterns | 45 min |
| 6 | `api-rules.instructions.md` | Your response envelope, status code rules, mandatory annotations | 30 min |
| 7 | `api-rules/SKILL.md` | Paste a real controller as the reference template | 30 min |
| 8 | Remaining instructions + skills | Integration test, BDD conventions and examples | 2–3 hrs |

**Ongoing maintenance rule:** When a new convention is introduced or a PR corrects a repeated mistake, the instruction or skill file update is included in the same PR. Instructions that lag behind the codebase become noise.

---

## What is NOT included and why

| Excluded | Reason |
|----------|--------|
| Chat modes (`.github/chatmodes/`) | VS Code only — does not work in JetBrains or other IDEs. The prompts and AGENTS.md provide the same workflow enforcement without IDE lock-in. |
| JIRA integration | Deliberate — the first version uses paste-in. Direct JIRA connection via Copilot Extensions (MCP) is the next planned phase and requires a separate implementation. |
| Org-level instructions | Configured in GitHub org settings UI, not a file. Add security baselines and compliance rules there once the repo-level setup is working. |

---

## Future: JIRA integration

The next evolution of this workflow is connecting Copilot directly to JIRA via a Copilot Extension or MCP server, turning step 1 of the PLAN phase from:

> "Paste the JIRA ticket description"

into:

> `@jira PAY-1234 create spec`

The file architecture, AGENTS.md workflow, and spec format are all designed to support this — the PLAN phase trigger is the only thing that changes.