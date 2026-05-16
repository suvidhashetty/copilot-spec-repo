# jOOQ skill

This skill provides DSL patterns and query examples for working with jOOQ in this project.
Load this skill when writing or reviewing repository layer files.

---

## Standard repository structure

[REPLACE: paste a representative repository class showing your standard structure — constructor injection of DSLContext, method signatures, error handling.]

```kotlin
// REPLACE THIS with a real repository example from your codebase

@Repository
class UserRepository(private val dsl: DSLContext) {

    fun findById(id: UserId): User? {
        return dsl
            .selectFrom(USERS)
            .where(USERS.ID.eq(id.value))
            .fetchOneInto(User::class.java)
        // REPLACE: show your actual mapping approach
    }

    fun save(user: User): User {
        // REPLACE: show your insert/upsert pattern
    }

    fun findAll(pageable: Pageable): Page<User> {
        // REPLACE: show your pagination pattern
    }
}
```

---

## Query patterns

[REPLACE: add the jOOQ query patterns most commonly used in your project — joins, aggregations, batch inserts, etc.]

```kotlin
// REPLACE: join pattern
// REPLACE: pagination pattern with total count
// REPLACE: batch insert pattern
// REPLACE: upsert pattern
```

---

## Result mapping

[REPLACE: show how jOOQ results are mapped to domain objects in your project. Custom record mapper, fetchInto, or manual mapping?]

```kotlin
// REPLACE: show your preferred mapping approach
```

---

## Generated DSL reference

[REPLACE: list the main generated tables/records used in this project so Copilot knows what to reference.]

Key generated tables in this project:
- `USERS` → [REPLACE: describe the table]
- [REPLACE: add more]

---

## Reference

[REPLACE: link to your jOOQ version docs, any internal jOOQ wiki, or code generation config.]
