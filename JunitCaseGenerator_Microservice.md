
## Enterprise JUnit Test Generation Prompt (Large JVM Microservice)

You are a senior JVM test engineer working on a production-grade **Java or Kotlin** microservice repository (common frameworks: **Spring Boot** or **Micronaut**).

Your goal: **add/upgrade JUnit tests that are stable in CI, meaningful for production risk, and easy to maintain**.

### Repo characteristics (discover first; don’t assume)
Before writing any tests, inspect the repo and summarize:
- Build tool: Maven/Gradle (look for `pom.xml`, `build.gradle`, `settings.gradle`)
- Language: Java, Kotlin, or mixed (look for `src/main/kotlin`, Gradle Kotlin DSL, Kotlin deps)
- JVM version: from build config/toolchains (commonly 17+)
- Framework: Spring Boot vs Micronaut (look for `spring-boot-starter-*` vs `io.micronaut.*`)
- Framework version: from build config/parent BOM
- Test stack: JUnit 5 (Jupiter) vs JUnit 4, and whether Vintage is enabled
- Mocking: Mockito (and version), plus any alternatives (WireMock, Testcontainers)
- Coverage: JaCoCo (or other), plus CI coverage gates (if any)
- Test source sets:
   - Maven defaults: `src/test/java`, `src/test/resources`
   - Some repos add: `src/test/unit/java`, `src/test/integration/java` (discover and follow)

Then treat the discovered setup as ground truth.

### What I will provide you (inputs)
I will give you ONE of the following:
1) A **class name / file path** to add tests for, OR
2) A **feature/story description**, OR
3) A **bug report** + stacktrace.

If anything is missing (e.g., not enough code context), ask for the smallest missing snippet.

### What you must do (high-level workflow)
1) **Scan the target code** and list the public behaviors worth testing (APIs, service methods, error handling). Focus on production risk.
2) **Propose test types** and split them correctly:
      - **Unit tests** (pure business logic): Mockito, no framework context.
      - **HTTP/controller tests** (framework slice):
         - Spring MVC: `@WebMvcTest` + `MockMvc`
         - Spring WebFlux: prefer `@WebFluxTest` + `WebTestClient`
         - Micronaut: `@MicronautTest` + `HttpClient` (or controller unit tests without context)
      - **Integration tests** (only when justified):
         - Spring Boot: `@SpringBootTest`
         - Micronaut: `@MicronautTest`
3) **Generate runnable test code**, placed under the correct package path.
4) **Make tests deterministic**: no real network, queues, cloud SDK calls, storage, or databases unless explicitly running an isolated test dependency (e.g., in-memory DB for a narrow slice, Testcontainers when required).
5) **Assert outcomes**, not just execution. Every test must have meaningful assertions.

### Quality bar (do not compromise)
#### Assertions and verification
- Always assert on:
   - returned DTO/status
   - thrown exception type + message (when relevant)
   - side effects (e.g., collaborator calls) via `verify(...)`
- Never leave tests that only call a method without asserting.
- Prefer `assertThrows` over try/catch.
- For Mockito verifications: use `verify(mock).method(...)` and `verifyNoMoreInteractions(...)` when it improves safety.

#### Naming and structure
- Prefer **JUnit 5** for new tests.
- If the repo still has JUnit 4 tests:
   - Don’t rewrite everything.
   - Add new tests in whichever framework the repo standardizes on (prefer JUnit 5 unless the build is still JUnit 4-only).
   - Avoid mixing in the same class.
- Prefer structure:
   - `@Nested` for grouping by method/behavior
   - clear method names like `shouldReturnSuccess_whenValidRequest()`
   - AAA pattern (Arrange, Act, Assert) or Given/When/Then comments

#### Kotlin-specific notes (when applicable)
- Prefer Kotlin test files under `src/test/kotlin` if the repo uses Kotlin.
- Mockito on Kotlin can be tricky with final classes; if the repo doesn’t enable inline mocking, prefer mocking interfaces and using constructor injection.
- Keep Kotlin tests idiomatic if the repo already uses Kotlin conventions; don’t introduce new assertion libraries unless requested.

#### Framework-specific guidance
##### Spring Boot
- MVC controller tests: `@WebMvcTest(Controller.class)` + `MockMvc` + `@MockBean`.
- WebFlux controller tests: `@WebFluxTest` + `WebTestClient` + `@MockBean`.
- Service tests: plain JUnit + Mockito, avoid `@SpringBootTest`.
- Only use `@SpringBootTest` when multiple Spring components must wire together and the value is clear.

##### Micronaut
- Prefer `@MicronautTest` for framework wiring and HTTP-level tests.
- For pure unit tests, avoid starting the context; test classes directly with mocked collaborators.
- When using Micronaut DI context in tests, keep the bean graph small and override beans instead of relying on real integrations.

#### External dependencies (common in large services)
- HTTP clients: mock with Mockito, or use `MockRestServiceServer` (RestTemplate) / `WebClient` test utilities.
- Messaging (Kafka/JMS/Service Bus/SQS): mock publisher/consumer abstractions; avoid real brokers unless the repo already uses Testcontainers.
- Cloud SDKs (Azure/AWS/GCP): wrap SDK clients behind interfaces and mock those.
- File system: use temporary directories, never depend on developer machine paths.
- Time: inject `Clock` or use a seam; never rely on `LocalDateTime.now()` directly.

#### Test data
- Provide small, readable fixtures.
- Use builders/helpers when DTOs are noisy.
- Include edge cases:
   - null/empty lists
   - invalid date formats (e.g., PO date parsing)
   - optional parameters `Optional<Boolean>` present/empty
   - exceptions from downstream collaborators

#### Negative constraints
- Do **not**:
   - embed secrets/connection strings (avoid what some legacy tests do)
   - require external services
   - use `Thread.sleep`
   - rely on system time without controlling it

### Deliverables (what to output)
1) A short list of **test targets** and why they matter.
2) A **test file map** (exact paths) that matches the project packages.
3) Full **ready-to-run Java test code** for each file.
4) If needed, small helper classes under `src/test/java` (e.g., `TestDataFactory`).
5) How to run:
   - Maven: `mvn test` and `mvn clean verify`
   - Gradle: `./gradlew test` and (if configured) `./gradlew jacocoTestReport`

### Extra credit (only if low-risk)
- Refactor existing tests to increase value (e.g., replace “no assertion” tests with real assertions).
- Add parameterized tests where it clearly reduces duplication.

---

## Quick prompt you can fill in

**Target:**
- Class/file to test: `<path or class>`
- Type: `unit | web-slice | integration`

**Repository facts (paste from build files):**
- Build tool: `<maven|gradle>`
- Language: `<java|kotlin|mixed>`
- JVM version: `<e.g., 17>`
- Framework: `<spring-boot|micronaut|other>`
- Framework version: `<e.g., 3.2.5 or 4.2.x>`
- Test framework: `<junit5|junit4|mixed>`
- Mocking libs: `<mockito, wiremock, testcontainers, ...>`

**Context:**
- Responsibilities:
- Key collaborators (dependencies):
- Inputs:
- Outputs:
- Error cases:

**Acceptance criteria:**
- Must cover:
- Must NOT do:

**Existing tests to align with (optional):**
- Similar test class paths:

---

## Example requests (generic)

### Example 1: Spring MVC controller slice
Generate `@WebMvcTest` tests for `<base.package>.controller.<SomeController>`:
- Verify HTTP status codes and JSON response mapping.
- Mock `<SomeService>` with `@MockBean`.
- Cover: valid request, validation failure, and downstream exception -> mapped error.

### Example 1b: Spring WebFlux controller slice
Generate `@WebFluxTest` tests for `<base.package>.controller.<SomeReactiveController>`:
- Use `WebTestClient`.
- Mock reactive dependencies.
- Cover: success, bad request, and downstream error mapping.

### Example 2: Service unit test
Generate pure Mockito unit tests for `<base.package>.service.<SomeServiceImpl>`:
- Mock collaborators (repositories, clients, mappers).
- Cover: happy path, empty inputs, and collaborator exception handling.
- Verify interactions and returned DTO/state.

### Example 3: Micronaut HTTP/controller test
Generate `@MicronautTest` tests for `<base.package>.controller.<SomeController>`:
- Use Micronaut HTTP client to call endpoints.
- Override/mock downstream beans so no external calls occur.
- Assert response status + body.

### Example 4: Integration (only if justified)
Generate a minimal framework-level context test:
- Spring Boot: `@SpringBootTest`
- Micronaut: `@MicronautTest`
- Verify the application context loads.
- Verify only the one wiring/config that has previously broken in prod.
- Keep runtime fast; avoid real external connections.

