---
title: Use Domain-Specific Names Instead of Generic Module Names
paths:
  - "**/*"
  - "**/*"
impact: HIGH
---

# Use Domain-Specific Names Instead of Generic Module Names

Avoid generic module names like `utils`, `helpers`, `common`, and `shared`. These names attract unrelated functions, creating grab-bag files with no cohesion. Use domain-specific names that reflect the bounded context and the module's single responsibility -- names like `OrderCalculator`, `UserAuthenticator`, or `InvoiceGenerator` make purpose immediately clear and enforce cohesion by design.

Generic names signal missing domain analysis. When a developer reaches for `Utils.java`, it usually means the function belongs in a domain module that has not been identified yet. Naming modules after their domain concept prevents them from becoming dumping grounds and keeps each module focused on a single, clear purpose.

## Critical princeples

- Follow domain-driven design and ubiquitous language
- **AVOID** generic names: `utils`, `helpers`, `common`, `shared`
- **USE** domain-specific names: `OrderCalculator`, `UserAuthenticator`, `InvoiceGenerator`
- Follow bounded context naming patterns
- Each module should have a single, clear purpose

## Incorrect

Generic module names attract unrelated functions, making the file a dumping ground with no cohesion or clear ownership.

```java
// OrderUtils.java — grab-bag of unrelated functions
public class OrderUtils {
    public static double calculateOrderTotal(List<OrderItem> items) {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }

    public static String formatUserDisplayName(User user) {
        return user.getFirstName() + " " + user.getLastName();
    }

    public static String generateInvoiceNumber() {
        return "INV-" + System.currentTimeMillis();
    }
}
```

Generic Naming Anti-Patterns:
- `Utils.java` with 50 unrelated methods
- `Helpers.java` as a dumping ground
- `Common.java` with unclear purpose

## Correct

Each function lives in a module named after its bounded context, enforcing single responsibility and making purpose self-documenting.

```java
// OrderCalculator.java — all order pricing logic
public class OrderCalculator {
    public static double calculateOrderTotal(List<OrderItem> items) {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
}

// UserDisplayService.java — user presentation formatting
public class UserDisplayService {
    public String formatDisplayName(User user) {
        return user.getFirstName() + " " + user.getLastName();
    }
}

// InvoiceGenerator.java — invoice creation logic
public class InvoiceGenerator {
    public String generateInvoiceNumber() {
        return "INV-" + System.currentTimeMillis();
    }
}
```

## Reference

- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.domainlanguage.com/ddd/) — Eric Evans
- Source: `plugins/ddd/skills/software-architecture/SKILL.md` — Naming Conventions and Generic Naming Anti-Patterns
