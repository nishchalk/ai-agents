# Copilot Instructions — {PROJECT_NAME}

> Copy this file to `.github/copilot-instructions.md` in your repo root.
> Fill in all `{PLACEHOLDER}` values. Delete this header comment when done.

## Project overview

- **Name**: {PROJECT_NAME}
- **Purpose**: {ONE_SENTENCE_PURPOSE}
- **Java version**: {JAVA_VERSION}
- **Build tool**: Maven {MAVEN_VERSION}
- **Primary framework(s)**: {FRAMEWORKS}
- **Module layout**: {single-module | multi-module | microservices}
- **Deployment target**: {Docker/K8s | Lambda | bare metal | other}

## Repository layout

```
src/
  main/java/{BASE_PACKAGE}/   # production code
  main/resources/             # config, static assets
  test/java/{BASE_PACKAGE}/   # tests
pom.xml
```

## Coding standards

### Mandatory rules — always enforce these

- Follow {STYLE_GUIDE} enforced by {LINTER}. Run `mvn {STYLE_GOAL}` before committing.
- All public API must have Javadoc (`@param`, `@return`, `@throws`).
- Never use `System.out.println` — use SLF4J: `LoggerFactory.getLogger(getClass())`.
- Wrap checked exceptions in domain-specific unchecked exceptions; never swallow silently.
- Prefer immutability: `final` fields, records for DTOs (Java 16+), unmodifiable collections.
- Null safety: annotate with `@NonNull`/`@Nullable`; return `Optional` instead of `null` from public methods.
- Constructor injection only — no `@Autowired` on fields or setters.

### Naming conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | UpperCamelCase | `PaymentService` |
| Methods / fields | lowerCamelCase | `processPayment()` |
| Constants | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |
| Packages | lowercase, dot-separated | `com.example.payment` |
| Test classes | `{Subject}Test` (unit) / `{Subject}IT` (integration) | `PaymentServiceTest` |

### Package architecture

```
controller  →  service  →  repository
              ↓
            domain  (no inbound dependencies)
```

Never let `domain` import from `controller` or `service`.

## Build & run

```bash
mvn clean compile          # compile
mvn test                   # unit tests
mvn verify                 # unit + integration tests
mvn clean package          # build jar
mvn spring-boot:run        # run locally (Spring Boot)
```

## Testing guidelines

- Unit tests: JUnit 5 + Mockito. No Spring context loaded unless required.
- Integration tests (`*IT.java`): `@SpringBootTest` or Testcontainers.
- Target ≥ 80% line coverage on `service` and `domain` packages.
- Test method naming: `methodName_stateUnderTest_expectedBehavior`.
- No `Thread.sleep` or `new Date()` in tests — use `Clock` or mocks.

## CI pipeline

- Tool: {CI_TOOL}
- Pipeline runs `mvn verify` on every push and pull request.
- PRs require green build + at least one approval before merge.

## Off-limits — do not modify

{OFF_LIMITS_LIST}

## Agent operating rules

1. Always run `mvn test` after any source change before declaring done.
2. Do not modify files under `target/` or any generated source directory.
3. When adding a Maven dependency, check `<dependencyManagement>` for an existing version first.
4. Prefer editing existing classes over creating new ones for small changes.
5. When unsure about an architectural decision, ask before proceeding.
6. Do not add comments that restate what the code does — only explain non-obvious intent or trade-offs.
