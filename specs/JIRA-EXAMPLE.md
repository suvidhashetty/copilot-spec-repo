# JIRA-EXAMPLE: Example spec (replace with real ticket)

**Status:** Draft
**Author:** Copilot + [Developer name]
**Created:** [Date]
**Last updated:** [Date]

---

## Summary

[2–3 sentence plain-English description of what this feature does and why. Written for someone who hasn't read the ticket.]

---

## Requirement source

[Paste the original JIRA description here, or link to the ticket. Do not paraphrase — keep the original text so decisions can be traced back to it.]

---

## Files to modify

| File path | Type of change | Reason |
|-----------|---------------|--------|
| `src/main/kotlin/.../controller/ExampleController.kt` | Add new endpoint | [Why] |
| `src/main/kotlin/.../service/ExampleService.kt` | Add business logic | [Why] |
| `src/main/kotlin/.../repository/ExampleRepository.kt` | Add query | [Why] |

---

## New files to create

| File path | Purpose |
|-----------|---------|
| `src/main/kotlin/.../model/ExampleRequest.kt` | Request DTO for the new endpoint |
| `src/test/kotlin/.../ExampleServiceTest.kt` | Unit tests for ExampleService |
| `src/test/kotlin/.../ExampleControllerIT.kt` | Integration tests for the new endpoint |
| `src/test/resources/features/example.feature` | BDD scenarios |

---

## Key code changes

[Illustrative snippets only — not final implementation. These exist to align understanding before coding starts.]

```kotlin
// Example: shape of the new endpoint
@GetMapping("/example/{id}")
fun getExample(@PathVariable id: String): ResponseEntity<ApiResponse<ExampleResponse>> {
    // delegate to service
}
```

---

## External dependencies

| Dependency | Version | Purpose | Added to |
|------------|---------|---------|---------|
| N/A | — | No new dependencies required | — |

---

## Test plan

### Unit tests
- `ExampleService.getExample()` — returns result when entity exists
- `ExampleService.getExample()` — throws `ExampleNotFoundException` when entity does not exist
- [Add more cases from the requirement's edge cases and error paths]

### Integration tests
- `GET /api/v1/example/{id}` returns `200` with correct body when entity exists
- `GET /api/v1/example/{id}` returns `404` when entity does not exist
- `GET /api/v1/example/{id}` returns `401` when unauthenticated

### BDD scenarios
- `User retrieves an example by ID successfully`
- `User receives not found error for unknown example ID`
- [Add scenarios for any business rule flows described in the ticket]

---

## Implementation checklist

Work through in order. Tick each item as completed.

- [ ] Add `ExampleRepository.findById()` query
- [ ] Add `ExampleService.getExample()` with error handling
- [ ] Add `ExampleRequest` and `ExampleResponse` DTOs
- [ ] Add `ExampleController.getExample()` endpoint
- [ ] Write unit tests for `ExampleService` — all cases passing
- [ ] Write integration tests for `ExampleController` — all cases passing
- [ ] Write BDD feature file and step definitions — all scenarios passing
- [ ] No new compiler warnings introduced
- [ ] OpenAPI annotations complete on new endpoint
- [ ] PR description references this spec file
