# API rules skill

This skill provides templates and examples for writing controllers and API layer code.
Load this skill when writing or reviewing controller files.

---

## Standard controller template

[REPLACE: paste a representative controller from your project showing the full structure — annotations, response wrapping, error handling delegation, OpenAPI annotations.]

```kotlin
// REPLACE THIS with a real controller example from your codebase

@RestController
@RequestMapping("/api/v1/users")
class UserController(private val userService: UserService) {

    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID")
    @ApiResponse(responseCode = "200", description = "User found")
    @ApiResponse(responseCode = "404", description = "User not found")
    @PreAuthorize("hasRole('USER')")
    fun getUser(@PathVariable id: String): ResponseEntity<ApiResponse<UserResponse>> {
        // REPLACE: show your actual implementation pattern
    }
}
```

---

## Request/response DTO pattern

[REPLACE: show how request and response DTOs are structured in your project.]

```kotlin
// REPLACE: show your request DTO pattern with validation annotations
// REPLACE: show your response DTO pattern
// REPLACE: show how domain objects are mapped to response DTOs
```

---

## Standard error response

[REPLACE: show what your error response body looks like and how it is produced.]

```kotlin
// REPLACE: paste your @ControllerAdvice error handler or error response structure
```

---

## OpenAPI documentation conventions

[REPLACE: show any project-specific OpenAPI annotation patterns — tags, security schemes, etc.]

---

## Reference

[REPLACE: link to your internal API design guide, Swagger UI URL for your dev environment, or any API governance docs.]
