---
name: tech-stack:add-springboot-best-practices
description: Setup Spring Boot best practices and code style rules in CLAUDE.md
argument-hint: Optional argument which practices to add or avoid
---

# Setup Spring Boot Best Practices

Create or update CLAUDE.md in with following content, <critical>write it strictly as it is</critical>, do not summarise or introduce and new additional information:

```markdown
## Code Style Rules

### General Principles

- **Java/Spring Boot**: All code must use Spring Boot conventions, leverage Spring's type safety features and dependency injection

### Code style rules

- Interfaces over concrete classes for dependency injection - use interfaces for service abstractions
- Use enums for constant values, prefer them over string literals
- Export all types by default in DTOs and domain objects
- Use custom exceptions instead of generic RuntimeException
- Prefer constructor injection over field injection

### Best Practices

#### Library-First Approach

- Common areas where libraries should be preferred:
  - Date/time manipulation → java.time (built-in), Joda-Time
  - Form validation → Jakarta Validation (Bean Validation), Hibernate Validator
  - HTTP requests → RestTemplate, WebClient, Spring's RestClient
  - State management → Spring's @Scope, @SessionScope, @ApplicationScope
  - Utility functions → Apache Commons Lang, Guava

#### Code Quality

- Use builder pattern for complex object construction:
  - Instead of `new User(name, email, age)` use builder
  - Instead of `new Order(itemId, quantity, price)` use builder with fluent API
- Use Optional for null-returning methods instead of returning null
- Use @Slf4j for logging instead of System.out.println
```

