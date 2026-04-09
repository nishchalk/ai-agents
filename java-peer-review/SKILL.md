---
name: java-peer-review
description: >-
  Performs a structured peer review of Java/Maven code changes following
  balanced review standards: critical issues block merge, suggestions are
  advisory. Use when the user asks for a Java code review, PR review, pull
  request review, or wants to review Java changes before merging.
---

# Java Peer Review Agent

Performs a structured, balanced peer review of Java code. Critical findings
must be resolved before merge; all others are advisory.

## Onboarding — ask these questions first

Before starting the review, clarify scope with the `AskQuestion` tool (or
conversationally if unavailable):

1. **What is being reviewed?** — specific files, a PR diff, or an entire feature branch?
2. **Is there a `PROJECT-AGENT.md`?** — load it for project-specific context
3. **Any areas to focus on?** — performance, security, test coverage, API design?
4. **Any known trade-offs?** — intentional shortcuts, legacy constraints to be aware of?

## Review severity levels

| Level | Label | Meaning |
|-------|-------|---------|
| 🔴 | **CRITICAL** | Must fix before merge — correctness bug, security flaw, data loss risk |
| 🟠 | **MAJOR** | Should fix — violates project standards, likely to cause bugs at scale |
| 🟡 | **MINOR** | Advisory — style, readability, minor best-practice deviation |
| 🔵 | **NIT** | Optional — purely cosmetic, formatting preference |
| 🟢 | **PRAISE** | Acknowledge good patterns worth keeping |

A PR is merge-ready only when there are **zero CRITICAL and zero MAJOR** items open.

## Review checklist

Work through these areas in order. Skip areas not applicable to the diff.

### 1. Correctness
- [ ] Logic matches the stated intent / ticket description
- [ ] Null inputs handled; `@NonNull` / `@Nullable` contracts respected
- [ ] Integer overflow, off-by-one, and boundary conditions covered
- [ ] Concurrency: no race conditions on shared mutable state
- [ ] Transactions cover the right scope; no partial-commit risk

### 2. Security
- [ ] No hard-coded credentials, tokens, or PII in source
- [ ] SQL / JPQL uses parameterised queries (no string concatenation)
- [ ] User-controlled input validated before use (not just at the UI layer)
- [ ] Sensitive data not logged (passwords, card numbers, SSNs)
- [ ] File paths sanitised against path-traversal

### 3. Exceptions & error handling
- [ ] Exceptions not swallowed silently
- [ ] Checked exceptions wrapped in domain unchecked exceptions with cause
- [ ] `finally` / try-with-resources used for all closeable resources
- [ ] HTTP / RPC callers handle error responses; no unchecked propagation to end users

### 4. Code quality
- [ ] Methods ≤ 30 lines; classes ≤ 300 lines (flag if exceeded, not automatic CRITICAL)
- [ ] No raw types, unchecked casts, or `@SuppressWarnings("unchecked")` without comment
- [ ] No `System.out.println` — only SLF4J
- [ ] No mutable `static` fields
- [ ] DRY: no copy-paste blocks > 5 lines that should be extracted
- [ ] `equals` + `hashCode` consistent; `Comparable` consistent with `equals`

### 5. API & design
- [ ] Public method signatures are minimal — no leaking implementation types
- [ ] Constructor injection used (not field or setter injection)
- [ ] `Optional` not used as method parameter
- [ ] New public APIs have Javadoc with `@param`, `@return`, `@throws`
- [ ] No breaking changes to existing public contracts without versioning

### 6. Tests
- [ ] New production code has corresponding unit tests
- [ ] Tests follow `methodName_stateUnderTest_expectedBehavior` naming
- [ ] No `Thread.sleep` or wall-clock `new Date()` in tests — use `Clock` / mocks
- [ ] Mocks verify interactions; no over-mocking of simple value objects
- [ ] Integration tests (`*IT.java`) use Testcontainers or `@SpringBootTest` correctly

### 7. Maven / build
- [ ] New dependency added to `<dependencyManagement>` if project uses it
- [ ] No snapshot dependencies in a release branch
- [ ] No new `<scope>compile</scope>` for test-only libraries
- [ ] `pom.xml` changes don't break multi-module build order

### 8. Performance (flag only if change is on a hot path)
- [ ] No N+1 queries — collections fetched eagerly or with join fetch where needed
- [ ] Streams not materialised unnecessarily into large intermediate lists
- [ ] No synchronous I/O on a thread pool that shouldn't block

## Output format

Produce the review as structured markdown:

```markdown
## Peer Review — {filename or PR title}

### Summary
{1–3 sentence overview of the change and overall impression}

### Findings

#### 🔴 CRITICAL
- **[ClassName.java:42]** `userInput` concatenated directly into SQL query — SQL injection risk.
  Fix: use `JdbcTemplate` with `?` parameters.

#### 🟠 MAJOR
- **[PaymentService.java:87]** `DataAccessException` caught and swallowed; caller receives `null`.
  Fix: rethrow as `PaymentPersistenceException(message, cause)`.

#### 🟡 MINOR
- **[OrderController.java:15]** Field injection (`@Autowired` on field) — prefer constructor injection for testability.

#### 🔵 NIT
- **[UserMapper.java:33]** Trailing whitespace on line 33.

#### 🟢 PRAISE
- Clean use of `Record` for `PaymentRequest` — immutable and self-validating.

### Verdict
- [ ] ✅ Approve — no blocking issues
- [x] 🔄 Request changes — resolve CRITICAL / MAJOR items above, then re-request review
```

## Review workflow

```
1. Load PROJECT-AGENT.md (if present) for project context
2. Ask onboarding questions
3. Read changed files / diff
4. Work through checklist — note findings with file + line reference
5. Write structured review output
6. State verdict clearly
```

## Additional resources

- For project-specific conventions, see [java-project-agent skill](../java-project-agent/SKILL.md)
