# Cucumber BDD skill

This skill provides templates and examples for writing Gherkin feature files and Cucumber step definitions.
Load this skill when writing `.feature` files or step definition files.

---

## Feature file template

[REPLACE: paste a real feature file from your project as the reference example. Annotate important decisions.]

```gherkin
# REPLACE THIS with a real feature file from your codebase

Feature: User registration
  As a new visitor
  I want to register for an account
  So that I can access the platform

  Background:
    Given the system is available
    And no user exists with email "test@example.com"

  Scenario: Successful registration with valid details
    When a user registers with email "test@example.com" and a valid password
    Then the user account is created
    And a confirmation email is sent to "test@example.com"

  Scenario: Registration fails when email is already taken
    Given a user already exists with email "existing@example.com"
    When a user registers with email "existing@example.com" and a valid password
    Then the registration is rejected with error "Email already in use"

  Scenario Outline: Registration fails with invalid email formats
    When a user registers with email "<email>" and a valid password
    Then the registration is rejected with error "Invalid email format"

    Examples:
      | email          |
      | notanemail     |
      | @nodomain.com  |
      | missing@       |
```

---

## Step definition template

[REPLACE: paste a representative step definition class from your project.]

```kotlin
// REPLACE THIS with a real step definition class from your codebase

class UserRegistrationSteps(
    private val userService: UserService // REPLACE: inject the right dependency
) {

    @Given("no user exists with email {string}")
    fun noUserExistsWithEmail(email: String) {
        // REPLACE: implement
    }

    @When("a user registers with email {string} and a valid password")
    fun userRegistersWithEmail(email: String) {
        // REPLACE: implement
    }

    @Then("the user account is created")
    fun userAccountIsCreated() {
        // REPLACE: implement assertion
    }
}
```

---

## Shared step definitions

[REPLACE: list or link to shared steps that are already defined in the project and can be reused.]

Common shared steps in this project:
- [REPLACE: e.g. `Given the system is available` → `CommonSteps.kt`]
- [REPLACE: add more shared steps as they are written]

---

## Reference

[REPLACE: links to your internal BDD wiki, Cucumber docs, or any team decisions about BDD scope and coverage.]
