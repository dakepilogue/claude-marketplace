---
title: Explicit Data Flow
paths:
  - "src/**/*"
impact: HIGH
---

# Explicit Data Flow

If a method produces a result, return it. Never rely on mutation of an input parameter to communicate output. Data should flow explicitly through return values so the reader can trace where each value comes from. When a method mutates its input, the data flow is hidden -- you cannot tell from the call site what changed or whether other references were affected. With explicit returns and immutable references, the call site becomes self-documenting: assignments show data origin, and immutability guarantees no downstream code silently altered what you are reading. Prefer pure expressions that produce new values over procedures that modify existing ones.

## Incorrect

Mutation of the input hides where data ends up. The call site looks like a standalone command, but the caller depends on the side effect for its result.

```java
OrderResult result = new OrderResult();
if(featureEnabled)
  applyNewFeature(result);  // mutates result in-place
```

Using reassignment is still unclear because the reader cannot tell whether `applyNewFeature` also mutates the original in addition to returning a value.

```java
OrderResult result = new OrderResult();
if(featureEnabled)
  result = applyNewFeature(result);  // unclear whether applyNewFeature also mutates
```

## Correct

A pure expression with immutable reference makes data flow visible in one line. The reader sees exactly what `result` is bound to, with no hidden mutation.

```java
OrderResult result = featureEnabled ? applyNewFeature(baseData) : new OrderResult();
```

## Reference

- [Referential Transparency](https://en.wikipedia.org/wiki/Referential_transparency)
- [Immutability and Pure Functions](https://www.oracle.com/java/technologies/javase/immutability.html)
