---
title: Command-Query Separation (CQS)
paths:
  - "src/**/*"
impact: HIGH
---

# Command-Query Separation (CQS)

A method must either return a value (query) or cause a side effect (command), never both. Mixing the two makes call sites deceptive: a mutation disguised as a query hides state changes, and a query that secretly throws hides control flow. Separate queries from commands so that assignments signal pure data retrieval and standalone calls signal state changes. When you need both a result and a side effect, split the operation into two explicit steps.

## Incorrect Mutation

`applyNewFeature(result)` mutates its input but the caller uses the mutated object as if it were a return value. The mutation is invisible at the call site.

```java
OrderResult result = new OrderResult();
if (featureEnabled)
  applyNewFeature(result);  // mutates result — looks like command but used as query
```

Reassignment does not fix it when the method both mutates AND returns. The caller cannot tell whether the original was changed.

```java
OrderResult result = new OrderResult();
if (featureEnabled)
  result = applyNewFeature(result);  // unclear: does it mutate AND return?
```

## Correct Pure Function

Pure expression that returns a new value without mutating input. The call site clearly shows this is a query.

```java
OrderResult result = featureEnabled ? applyNewFeature(baseData) : new OrderResult();
```

## Incorrect Hidden Command

`validateResult` looks like a query but secretly throws, making it a hidden command. The call site hides a control flow branch.

```java
OrderResult result = performProcess(param);
validateResult(result); // -> throws ProcessingException(...) — looks like query but is a command
```

## Correct Explicit Control Flow

Explicit control flow at the call site. The caller decides what to do with an invalid result instead of a hidden throw.

```java
OrderResult result = performProcess(param);
if (!isValid(result))
  throw new SomeError(result);
```

## Reference

- [CommandQuerySeparation by Martin Fowler](https://martinfowler.com/bliki/CommandQuerySeparation.html)
- [Command-Query Separation by Bertrand Meyer](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)
