---
title: Typed Error Handling with Logging
paths:
  - "src/**/*"
impact: HIGH
---

# Typed Error Handling with Logging

Never silently swallow exceptions. Every catch block must use typed error handling and log the error before rethrowing or returning a failure result. Generic `catch (Exception)` blocks hide the root cause of failures, making production debugging nearly impossible. Use typed catch blocks that distinguish between expected domain errors and unexpected system failures. When error handling logic grows complex, extract it into smaller, reusable handler methods rather than duplicating catch logic across the codebase. Log the error with sufficient context (operation name, relevant IDs) before rethrowing so that the failure is traceable in logs even if the caller also catches it.

## Incorrect

Errors are caught with an untyped generic block, silently swallowed or rethrown without logging. The caller has no trace of what failed or why.

```java
public PaymentResult processPayment(String orderId, BigDecimal amount) {
    try {
        PaymentResult result = paymentGateway.charge(orderId, amount);
        orderRepository.updateStatus(orderId, "paid");
        return result;
    } catch (Exception e) {
        // Silently swallowed — no logging, no typed handling
        return null;
    }
}
```

## Correct

Errors are caught with typed checks, logged with context before rethrowing, and complex handling is extracted into a reusable method.

```java
public PaymentResult processPayment(String orderId, BigDecimal amount) {
    try {
        PaymentResult result = paymentGateway.charge(orderId, amount);
        orderRepository.updateStatus(orderId, "paid");
        return result;
    } catch (PaymentDeclinedException e) {
        logger.warn("Payment declined", () -> Map.of(
            "orderId", orderId,
            "amount", amount,
            "reason", e.getReason()
        ));
        throw new DomainException("Payment declined for order " + orderId);
    } catch (NetworkException e) {
        logger.error("Payment gateway unreachable", () -> Map.of(
            "orderId", orderId,
            "amount", amount,
            "cause", e
        ));
        throw new InfrastructureException("Payment service unavailable", e);
    } catch (Exception e) {
        logger.error("Unexpected payment failure", () -> Map.of(
            "orderId", orderId,
            "amount", amount,
            "error", e
        ));
        throw e;
    }
}
```

## Reference

- [Java Exception Handling Best Practices](https://www.oracle.com/java/technologies/javase/exceptions.html)
- [Spring Error Handling Guide](https://spring.io/guides/gs/spring-boot/)
