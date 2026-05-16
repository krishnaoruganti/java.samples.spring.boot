# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

REST API for managing football players built with Java and Spring Boot. Implements CRUD operations with a layered architecture, Spring Data JPA + SQLite, Bean Validation, Spring Cache, and Swagger documentation. Part of a cross-language comparison study (.NET, Go, Python, Rust, TypeScript). This is a PoC/learning project — documentation is a first-class concern.

## Tech Stack

- **Language**: Java 25 (LTS, required)
- **Framework**: Spring Boot 4.0.0 (Spring MVC)
- **ORM**: Spring Data JPA + Hibernate
- **Database**: SQLite (file-based runtime, in-memory for tests)
- **Build**: Maven 3 — always use `./mvnw` wrapper
- **Validation**: Bean Validation (JSR-380)
- **Caching**: Spring `@Cacheable` (simple in-memory, no expiry)
- **Mapping**: ModelMapper
- **Logging**: SLF4J
- **Testing**: JUnit 5 + AssertJ + MockMvc + Mockito
- **Coverage**: JaCoCo (target: 85%; exclude `Application.java` and `models/`)
- **API Docs**: SpringDoc OpenAPI 3 (Swagger)
- **Boilerplate**: Lombok
- **Containerization**: Docker (multi-stage, Eclipse Temurin Alpine, non-root user, HEALTHCHECK via Actuator)

## Structure

```text
src/main/java/
├── controllers/        — HTTP handlers; delegate to services, no business logic        [HTTP layer]
├── services/           — Business logic + @Cacheable caching                           [business layer]
├── repositories/       — Spring Data JPA with derived queries                          [data layer]
├── models/             — Player entity + DTOs
└── converters/         — JPA AttributeConverter for ISO-8601 date handling
src/main/resources/     — application.properties, Logback config
src/test/java/          — test classes mirroring main structure
src/test/resources/     — test config, schema (ddl.sql), seed data (dml.sql)
storage/                — SQLite database file (runtime)
```

Base package: `ar.com.nanotaboada.java.samples.spring.boot`

**Layer rule**: `Controller → Service → Repository → JPA`. Controllers must not access repositories directly. Business logic must not live in controllers.

## Coding Guidelines

- **Naming**: camelCase (methods/variables), PascalCase (classes), UPPER_SNAKE_CASE (constants)
- **DI**: Constructor injection via Lombok `@RequiredArgsConstructor`; never field injection
- **Annotations**: `@RestController`, `@Service`, `@Repository`, `@Entity`, `@Data`/`@Builder`/`@AllArgsConstructor` (Lombok)
- **Transactions**: `@Transactional(readOnly = true)` on read service methods; `@Transactional` on writes
- **Errors**: `@ControllerAdvice` for global exception handling; HTTP status codes: 200, 201, 204, 400, 404, 409
- **Logging**: SLF4J only; never `System.out.println`
- **DTOs**: Never expose entities directly in controllers — always use DTOs; use `@Valid` on controller request bodies; validation annotations: `@NotBlank`, `@Past`, `@Positive`, `@URL`
- **Date handling**: Entity date fields use `IsoDateConverter` (ISO-8601 TEXT format for SQLite)
- **Avoid**: field injection, `new` for Spring beans, missing `@Transactional`, exposing entities in controllers, hardcoded configuration

## Testing

- **Naming**: BDD Given-When-Then (`givenX_whenY_thenZ`) with `@DisplayName` for readable descriptions
- **Assertions**: AssertJ BDD style (`then(result).isNotNull()`)
- **Mocking**: `@MockitoBean` (Spring Boot 4.0 — not `@MockBean`)
- **Caching**: use `@AutoConfigureCache` on slice tests that exercise cached operations
- **Test data**: use `PlayerFakes` / `PlayerDTOFakes` factories — do not inline ad-hoc test objects
- **Database**: in-memory SQLite auto-clears after each test

## Commands

```bash
./mvnw spring-boot:run                  # start on port 9000
./mvnw clean test                       # run all tests
./mvnw clean test jacoco:report         # tests + coverage report
open target/site/jacoco/index.html      # view coverage

# Run a single test class or method
./mvnw test -Dtest=PlayersControllerTests
./mvnw test -Dtest=PlayersServiceTests#givenX_whenY_thenZ

docker compose up
docker compose down -v
```

## Agent Mode

### Proceed freely

- Route handlers and controller endpoints
- Service layer business logic
- Repository custom queries
- Unit and integration tests
- Exception handling in `@ControllerAdvice`
- Documentation updates, bug fixes, and refactoring
- Utility classes and helpers

### Ask before changing

- Database schema (entity fields, relationships)
- Dependencies (`pom.xml`)
- CI/CD configuration (`.github/workflows/`)
- Docker setup
- Application properties
- API contracts (breaking DTO changes)
- Caching strategy or TTL values
- Package structure

### Never modify

- `.java-version` (JDK 25 required)
- Maven wrapper scripts (`mvnw`, `mvnw.cmd`)
- Port configuration (9000/9001)
- Test database configuration (in-memory SQLite)
- Production configurations or deployment secrets

## Creating Issues

This project uses Spec-Driven Development (SDD): discuss in Plan mode first, create a GitHub Issue as the spec artifact, then implement. Always offer to draft an issue before writing code.

**Feature request** (`enhancement` label):

- **Problem**: the pain point being solved
- **Proposed Solution**: expected behavior and functionality
- **Suggested Approach** *(optional)*: implementation plan if known
- **Acceptance Criteria**: at minimum — behaves as proposed, tests added/updated, no regressions
- **References**: related issues, docs, or examples

**Bug report** (`bug` label):

- **Description**: clear summary of the bug
- **Steps to Reproduce**: numbered, minimal steps
- **Expected / Actual Behavior**: one section each
- **Environment**: runtime versions + OS
- **Additional Context**: logs, screenshots, stack traces
- **Possible Solution** *(optional)*: suggested fix or workaround

## Key Workflows

**Add an endpoint**: Define DTO in `models/` with Bean Validation → add service method in `services/` with `@Transactional` → create controller endpoint with `@Operation` annotation + proper HTTP status codes → add tests using `PlayerDTOFakes` → run `./mvnw clean test jacoco:report`.

**Modify schema**: Update `@Entity` in `models/Player.java` → update DTOs if API changes → manually update `storage/players-sqlite3.db` (preserve 26 players) → update service, repository, and tests → run `./mvnw clean test`.

**After completing work**: Suggest a branch name (e.g. `feat/add-player-stats`) and a commit message following Conventional Commits including co-author line:

```text
feat(scope): description (#issue)

Co-authored-by: Claude Sonnet 4.6 <noreply@anthropic.com>
```

## Claude Code

- Run `/pre-commit` to execute the full pre-commit checklist for this project.
