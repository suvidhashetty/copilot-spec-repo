---
applyTo: "**/*.feature, **/steps/**/*.kt, **/stepdefs/**/*.kt"
---

# Cucumber BDD conventions

These rules apply to `.feature` files and step definition Kotlin files.
For examples and templates, load `.github/skills/cucumber-bdd/SKILL.md`.

---

## Gherkin writing rules

[REPLACE: define your Gherkin standards. Example:]

- Use `Given` for preconditions (system state before the action)
- Use `When` for the action the user or system takes
- Use `Then` for the expected outcome
- Use `And` to chain steps of the same type — never repeat `Given/When/Then` in sequence
- One scenario = one behaviour. If a scenario needs more than ~6 steps, split it.

---

## Scenario naming

[REPLACE: define naming conventions. Example:]

- Scenario titles must be readable as plain English sentences
- Format: `[Actor] [action] [outcome]`
- Example: `User receives an error when submitting an invalid email`

---

## Feature file structure

[REPLACE: define file-level structure. Example:]

- One feature file per domain concept (not per endpoint)
- Feature description (the `Feature:` block) must explain the business value in 1–2 sentences
- Use `Background:` for preconditions shared across all scenarios in a file
- Use `Scenario Outline:` with `Examples:` for data-driven scenarios

---

## Step definition rules

[REPLACE: define step definition standards. Example:]

- Step definitions live in `src/test/kotlin/.../steps/`
- One file per feature domain (e.g. `UserSteps.kt`, `PaymentSteps.kt`)
- Steps call service or API layer — never access the database directly from a step
- Reuse existing step definitions before writing new ones

---

## What NOT to do

[REPLACE: list anti-patterns. Example:]

- Do not write implementation details in Gherkin (no SQL, no class names, no method names)
- Do not use `But` — rephrase as a `Then` or `And`
- Do not have a single scenario cover multiple user journeys
