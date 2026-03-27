---
title: Use Existing Libraries Instead of Custom Code
impact: HIGH
paths:
  - "**/*"
  - "**/*"
---

# Use Existing Libraries Instead of Custom Code

Always search for existing libraries, services, or third-party APIs before writing custom code:
- Check Maven Central or Gradle Plugin Portal for existing libraries that solve the problem
- Evaluate existing services/SaaS solutions
- Consider third-party APIs for common functionality

Every line of custom code is a liability that requires maintenance, testing, and documentation. The Not Invented Here (NIH) syndrome leads to fragile, undertested reimplementations of solved problems.

Before writing any utility, helper, or infrastructure code, check for established libraries that solve the problem. For example, use `Resilience4j` instead of writing your own retry logic.

Custom code is only justified when:
- Specific business logic unique to the domain
- Performance-critical paths with special requirements
- When external dependencies would be overkill
- Security-sensitive code requiring full control
- When existing solutions don't meet requirements after thorough evaluation

## Incorrect

Custom retry logic is implemented from scratch instead of using an established library. This hand-rolled solution lacks features like exponential backoff, jitter, circuit breaking, and proper error classification that battle-tested libraries provide.

```java
// Custom retry utility - reinventing the wheel
public <T> T retry(Supplier<T> fn, int maxRetries, long delayMs) {
    Exception lastError = null;
    for (int attempt = 0; attempt < maxRetries; attempt++) {
        try {
            return fn.get();
        } catch (Exception error) {
            lastError = error;
            try {
                Thread.sleep(delayMs);
            } catch (InterruptedException ignored) {}
        }
    }
    throw lastError;
}

T data = retry(() -> fetchFromApi("/users"), 3, 1000);
```

Anti-Pattern to Avoid: NIH (Not Invented Here) Syndrome:
- Don't build custom auth when Spring Security exists
- Don't write custom state management instead of using Spring's scoped beans
- Don't create custom form validation instead of using Jakarta Validation

## Correct

An established library handles retry logic with proven patterns for backoff, jitter, and circuit breaking out of the box.

```java
import io.github.resilience4j.retry.Retry;
import io.github.resilience4j.retry.RetryConfig;

RetryConfig config = RetryConfig.custom()
    .maxAttempts(3)
    .waitDuration(Duration.ofSeconds(1))
    .retryExceptions(Exception.class)
    .build();

Retry retry = Retry.of("fetchApi", config);

T data = retry.executeSupplier(() -> fetchFromApi("/users"));
```

## Reference

- [Resilience4j Documentation](https://resilience4j.readme.io/)
- [Spring Boot Best Practices](https://spring.io/guides)
