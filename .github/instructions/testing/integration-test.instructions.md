---
applyTo: "**/*IT.kt"
---

# Integration test conventions

These rules apply whenever a `*IT.kt` file is being written or reviewed.
For examples and templates, load `.github/skills/integration-test/SKILL.md`.

---

## Framework and setup

[REPLACE: define your integration test setup. Example:]

- Use `@SpringBootTest` with the appropriate slice annotation for the layer being tested
- Use Testcontainers for database-dependent tests
- Database state must be reset between tests — use `@Transactional` or explicit cleanup

---

## Test slice guidance

[REPLACE: define which slice to use when. Example:]

- Controller layer only → `@WebMvcTest`
- Full stack with database → `@SpringBootTest` + Testcontainers
- Repository layer only → `@DataJdbcTest` (or equivalent for jOOQ)

---

## Naming and structure

[REPLACE: define naming rules. Example:]

- Class name format: `[ClassName]IT`
- Method name format: `should_[behaviour]_when_[condition]`
- Separate setup, action, and assertion clearly

---

## Data setup

[REPLACE: define how test data should be created. Example:]

- Use test data builders — do not construct domain objects inline in every test
- Test data builders live in `src/test/kotlin/.../testdata/`
- Do not use production seed data in integration tests

---

## What NOT to do

[REPLACE: list anti-patterns. Example:]

- Do not use `H2` in-memory database — use Testcontainers with the real database engine
- Do not share test state between test methods
- Do not make real HTTP calls to external services — use WireMock or similar
