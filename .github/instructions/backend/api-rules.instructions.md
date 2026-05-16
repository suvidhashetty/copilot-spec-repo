---
applyTo: "**/controller/**/*.kt, **/api/**/*.kt"
---

# API design rules

These rules apply to all controller and API layer files.
For examples and templates, load `.github/skills/api-rules/SKILL.md`.

---

## Response envelope

[REPLACE: define your standard response wrapper. Example:]

All API responses must be wrapped in the standard envelope:

```kotlin
data class ApiResponse<T>(
    val data: T?,
    val error: ApiError?,
    val meta: ResponseMeta?
)
```

Never return a raw domain object or a bare list directly from a controller.

---

## HTTP status codes

[REPLACE: define which status codes to use and when. Example:]

- `200 OK` — successful GET, successful update
- `201 Created` — successful resource creation (include `Location` header)
- `204 No Content` — successful DELETE
- `400 Bad Request` — validation failure (include field-level errors)
- `401 Unauthorized` — missing or invalid auth token
- `403 Forbidden` — authenticated but not authorised
- `404 Not Found` — resource does not exist
- `409 Conflict` — duplicate resource or state conflict
- `422 Unprocessable Entity` — business rule violation
- `500 Internal Server Error` — unexpected failure (never expose stack traces)

---

## Versioning

[REPLACE: define your versioning strategy. Example:]

- All endpoints are versioned via URL path: `/api/v1/...`
- New versions are only introduced when a breaking change is required
- Old versions must be supported for [REPLACE: define your deprecation period]

---

## Mandatory annotations

[REPLACE: list required annotations on every endpoint. Example:]

Every endpoint method must have:
- `@Operation(summary = "...")` — OpenAPI summary
- `@ApiResponse` annotations for all possible status codes
- `@PreAuthorize(...)` — explicit authorisation rule (even if public — use `permitAll`)

---

## Validation

[REPLACE: define validation rules. Example:]

- Use `@Valid` on request bodies — never validate manually in the controller
- Validation annotations live on the request DTO, not the domain model
- Controller methods must not contain business logic — delegate to the service layer

---

## What NOT to do

[REPLACE: list anti-patterns. Example:]

- Do not expose internal database IDs in responses
- Do not put business logic in controllers
- Do not return different response shapes for the same endpoint under different conditions
- Do not catch generic `Exception` in controllers — use a `@ControllerAdvice` for error handling
