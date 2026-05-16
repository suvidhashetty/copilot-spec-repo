# Project conventions

This file is loaded on every Copilot request. Keep it lean — only universal rules that apply to every file in the codebase.
Task-specific rules live in `.github/instructions/`. Examples and templates live in `.github/skills/`.

---

## Stack

- **Language:** Kotlin [REPLACE: specify version e.g. 1.9]
- **Build:** Maven [REPLACE: specify version]
- **Framework:** Spring Boot [REPLACE: specify version]
- **Database access:** jOOQ only — no Spring Data JPA, no raw JDBC, no native queries
- **Unit testing:** JUnit 5 + MockK
- **Integration testing:** [REPLACE: e.g. Spring Boot Test + Testcontainers]
- **BDD:** Cucumber with Gherkin

---

## Package structure

[REPLACE: paste your actual package structure here. Example:]

```
com.yourorg.yourapp
├── api
│   └── controller
├── domain
│   ├── model
│   └── service
├── infrastructure
│   └── repository
└── config
```

---

## Naming conventions

[REPLACE: define your naming rules. Example:]

- Controllers: `[Domain]Controller.kt`
- Services: `[Domain]Service.kt`
- Repositories: `[Domain]Repository.kt`
- Unit tests: `[ClassName]Test.kt`
- Integration tests: `[ClassName]IT.kt`
- Feature files: `[domain]-[scenario].feature`

---

## Non-negotiable rules

- All database access through the repository layer using jOOQ. Never call the database directly from a service or controller.
- Never return internal entity IDs in API responses. Use domain identifiers or UUIDs exposed explicitly.
- All configuration values go in `application.yml` or `application-[profile].yml`. No hardcoded values.
- No `System.out.println` or raw logging — use SLF4J via the `@Slf4j` annotation (or Kotlin logger equivalent).
- [REPLACE: add your own non-negotiable rules here]

---

## What NOT to put in this file

- Test-specific conventions → `.github/instructions/testing/`
- API design rules → `.github/instructions/backend/api-rules.instructions.md`
- jOOQ patterns → `.github/instructions/backend/jooq.instructions.md`
- Code examples or templates → `.github/skills/`
- Workflow instructions → `AGENTS.md`
