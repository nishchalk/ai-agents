# Java Coding Standards Reference

## Design principles (priority order)
1. **SOLID** — especially Single Responsibility and Dependency Inversion
2. **Fail fast** — validate inputs at entry points; throw immediately on invalid state
3. **Least surprise** — method names match exactly what they do
4. **Immutability by default** — mutate only when necessary and document why

## Exception handling
```java
// Good — wrap low-level exception in domain exception
try {
    repository.save(entity);
} catch (DataAccessException e) {
    throw new PaymentPersistenceException("Failed to save payment id=" + entity.getId(), e);
}

// Bad — swallowing or logging-and-rethrowing
catch (Exception e) {
    log.error("error", e); // then falls through — caller has no idea it failed
}
```

## Logging
```java
private static final Logger log = LoggerFactory.getLogger(PaymentService.class);

// Correct — guard expensive string construction
if (log.isDebugEnabled()) {
    log.debug("Processing payload: {}", payload.toJson());
}

// Use structured args, not concatenation
log.info("Payment processed id={} amount={}", id, amount); // ✅
log.info("Payment processed id=" + id);                    // ❌
```

## Collections
```java
// Prefer
List<String> names = List.of("Alice", "Bob");     // immutable
Map<String, Integer> scores = Map.of("a", 1);

// Avoid returning internal mutable collections
public List<Order> getOrders() {
    return Collections.unmodifiableList(orders); // or List.copyOf(orders)
}
```

## Optional
```java
// ✅ Return Optional from finders
Optional<User> findById(long id);

// ✅ Chain without null checks
service.findById(id)
       .map(User::getEmail)
       .orElseThrow(() -> new UserNotFoundException(id));

// ❌ Never use Optional as parameter
void process(Optional<String> name); // forces callers to wrap
```

## Concurrency
- Prefer `java.util.concurrent` types (`ConcurrentHashMap`, `AtomicLong`) over manual `synchronized`
- Document thread-safety guarantees with `@ThreadSafe` / `@NotThreadSafe` (jcip-annotations)
- Avoid `volatile` unless you understand the memory model implications

## Records (Java 16+)
```java
// Prefer records for immutable DTOs
public record PaymentRequest(String cardToken, BigDecimal amount, Currency currency) {}

// Add compact constructor for validation
public record PaymentRequest(String cardToken, BigDecimal amount, Currency currency) {
    public PaymentRequest {
        Objects.requireNonNull(cardToken, "cardToken must not be null");
        if (amount.compareTo(BigDecimal.ZERO) <= 0) throw new IllegalArgumentException("amount must be positive");
    }
}
```

## Maven conventions
- Dependency versions in `<dependencyManagement>`, not scattered in `<dependencies>`
- Plugin versions pinned — never rely on implicit defaults
- Use `${project.build.sourceEncoding}` = `UTF-8` explicitly
- Profiles for environment-specific config, not for changing source sets
