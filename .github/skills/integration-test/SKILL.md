# Integration test skill

This skill provides templates and examples for writing integration tests in this project.
Load this skill when writing or reviewing `*IT.kt` files.

---

## Standard integration test setup

[REPLACE: paste a representative integration test that shows your full test setup — Spring slice, Testcontainers config, data setup, assertion style.]

```kotlin
// REPLACE THIS with a real example from your codebase

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserControllerIT {

    companion object {
        @Container
        val postgres = PostgreSQLContainer("postgres:[REPLACE: version]")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test")
    }

    @Autowired
    private lateinit var testRestTemplate: TestRestTemplate

    @Test
    fun `should_return200_when_userExists`() {
        // REPLACE: show your full test including data setup, HTTP call, and assertion
    }
}
```

---

## Testcontainers setup

[REPLACE: show how Testcontainers is configured in your project — shared container, lifecycle, datasource override.]

```kotlin
// REPLACE: paste your actual Testcontainers base class or configuration
```

---

## Test data setup patterns

[REPLACE: show how integration tests create and clean up test data in your project.]

```kotlin
// REPLACE: show your data setup approach
// Examples: SQL scripts, test data builders that call the repository, @BeforeEach/@AfterEach cleanup
```

---

## WireMock / external service stubbing

[REPLACE: if your project uses WireMock or similar, show the pattern for stubbing external HTTP calls.]

```kotlin
// REPLACE: paste your WireMock setup pattern if applicable
```

---

## Reference

[REPLACE: add links to your internal integration testing standards doc and any relevant Testcontainers configuration docs.]
