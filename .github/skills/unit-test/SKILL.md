# Unit test skill

This skill provides templates and examples for writing unit tests in this project.
Load this skill when writing or reviewing `*Test.kt` files.

---

## Anatomy of a unit test in this project

[REPLACE: paste a real, representative unit test from your codebase here — ideally one that shows your MockK setup, naming, and assertion style. Annotate the important parts with comments.]

```kotlin
// REPLACE THIS with a real example from your codebase

@ExtendWith(MockKExtension::class)
@DisplayName("UserService")
class UserServiceTest {

    @MockK
    private lateinit var userRepository: UserRepository

    @InjectMockKs
    private lateinit var userService: UserService

    @Test
    fun `should_returnUser_when_userExists`() {
        // Arrange
        val userId = UserId("usr-123")
        val expectedUser = // REPLACE: construct a test user

        every { userRepository.findById(userId) } returns expectedUser

        // Act
        val result = userService.getUser(userId)

        // Assert
        assertThat(result).isEqualTo(expectedUser)
        verify(exactly = 1) { userRepository.findById(userId) }
    }

    @Test
    fun `should_throwUserNotFoundException_when_userDoesNotExist`() {
        // Arrange
        val userId = UserId("usr-999")
        every { userRepository.findById(userId) } returns null

        // Act & Assert
        assertThrows<UserNotFoundException> {
            userService.getUser(userId)
        }
    }
}
```

---

## Test data construction

[REPLACE: show how test objects are constructed in your project. If you use test data builders or fixtures, show the pattern here.]

```kotlin
// REPLACE: show your test data builder or factory pattern
// Example:
// val testUser = UserFixture.aUser(id = "usr-123", email = "test@example.com")
```

---

## Common MockK patterns

[REPLACE: add the MockK patterns your team uses most frequently — coroutines, slots, verify ordering, etc.]

```kotlin
// REPLACE: add your most-used MockK patterns
// Examples:
// Capturing arguments:     val slot = slot<UserId>(); every { repo.findById(capture(slot)) } returns user
// Verifying no calls:      verify { repo.delete(any()) wasNot Called }
// Relaxed mocks:           @MockK(relaxed = true)
```

---

## Reference

[REPLACE: add links to your internal testing wiki, confluence page, or any relevant documentation.]

- [REPLACE: link to your internal unit testing standards doc]
- [REPLACE: link to MockK documentation if relevant]
