---
applyTo: "**/*Test.kt"
---

# Unit test conventions

These rules apply whenever a `*Test.kt` file is being written or reviewed.
For examples and templates, load `.github/skills/unit-test/SKILL.md`.

---

## Framework and imports

[REPLACE: list the exact imports and annotations required in every unit test. Example:]

- Use JUnit 5 (`org.junit.jupiter.api.*`) — no JUnit 4 annotations
- Use MockK for mocking (`io.mockk.*`) — no Mockito
- Annotate the class with `@ExtendWith(MockKExtension::class)`

---

## Test naming

[REPLACE: define your naming convention. Example:]

- Method name format: `should_[expectedBehaviour]_when_[condition]`
- Use `@DisplayName` for human-readable descriptions
- Example: `fun should_returnUserNotFound_when_userDoesNotExist()`

---

## Structure rules

[REPLACE: define your structural rules. Example:]

- One assertion per test (use `assertSoftly` if multiple assertions are needed for the same logical check)
- Arrange / Act / Assert sections separated by blank lines
- No logic in tests (no loops, no conditionals)
- Each test must be independent — no shared mutable state between tests

---

## What to test

[REPLACE: define what must be covered. Example:]

- Happy path
- All edge cases mentioned in the spec
- All error/exception paths
- Boundary values

---

## What NOT to do

[REPLACE: list anti-patterns to avoid. Example:]

- Do not test private methods directly
- Do not use `Thread.sleep` — use MockK's coroutine support or fake timers
- Do not mock value objects — construct them directly
