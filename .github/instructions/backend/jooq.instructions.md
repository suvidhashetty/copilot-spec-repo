---
applyTo: "**/repository/**/*.kt, **/db/**/*.kt"
---

# jOOQ conventions

These rules apply to all repository and database layer files.
For DSL patterns and query examples, load `.github/skills/jooq/SKILL.md`.

---

## DSL style

[REPLACE: define your preferred jOOQ DSL style. Example:]

- Prefer `selectFrom(TABLE)` for simple single-table queries
- Use `select(...fields).from(TABLE)` for projections
- Always alias long queries for readability
- Avoid inline magic strings — use generated DSL constants exclusively

---

## Generated classes

[REPLACE: define where generated classes live and how they are used. Example:]

- Generated jOOQ classes live in `[REPLACE: your generated sources path]`
- Always use generated table and field references — never reference table or column names as strings
- Generated classes are read-only — never edit them manually

---

## Pagination

[REPLACE: define your pagination pattern. Example:]

- All list queries must support pagination via `limit` and `offset`
- Return a `Page<T>` wrapper that includes total count — always execute a count query alongside the data query
- Default page size: [REPLACE]
- Maximum page size: [REPLACE]

---

## Transaction handling

[REPLACE: define your transaction rules. Example:]

- Transactions are managed at the service layer, not the repository layer
- Repository methods are transaction-agnostic — they participate in an existing transaction if one exists
- Never annotate repository methods with `@Transactional`

---

## Error handling

[REPLACE: define how database errors should be handled. Example:]

- Catch `DataAccessException` at the repository layer and translate to domain exceptions
- Never let jOOQ or JDBC exceptions propagate to the service or controller layer
- Define domain exception types in `domain/exception/`

---

## What NOT to do

[REPLACE: list anti-patterns. Example:]

- Do not write raw SQL strings — use the jOOQ DSL
- Do not call the `DSLContext` from outside the repository layer
- Do not perform N+1 queries — use joins or batch fetching
- Do not use `fetchOne()` when the result could be null — use `fetchOneInto()` or map the result explicitly
