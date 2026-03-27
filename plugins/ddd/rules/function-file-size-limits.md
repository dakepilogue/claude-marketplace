---
title: Function and File Size Limits
impact: MEDIUM
paths:
  - "**/*"
  - "**/*"
---

# Function and File Size Limits

- Decompose methods longer than 80 lines into smaller, focused methods of 50 lines or fewer. When a method grows beyond 80 lines, it is almost certainly doing more than one thing and should be split.
- Keep files under 200 lines of code. Large functions accumulate multiple responsibilities, making them harder to test, review, and reuse.
- Extract cohesive blocks of logic into named methods that each serve a single purpose. If extracted methods are only used within the same context, keep them in the same class. However, when a file exceeds 200 lines even after decomposition, split related methods into separate classes grouped by responsibility.

## Incorrect

A single method handles validation, transformation, persistence, and notification. At over 80 lines it is difficult to test individual behaviors or reuse any part of the logic.

```java
public User processUserRegistration(Object input) {
    // Validate input (lines 1-20)
    if (input == null) throw new ValidationException("Invalid input");
    Map<String, Object> data = (Map<String, Object>) input;
    String email = (String) data.get("email");
    String name = (String) data.get("name");
    String password = (String) data.get("password");
    String role = (String) data.get("role");
    if (email == null || email.isEmpty()) throw new ValidationException("Email required");
    if (name == null || name.isEmpty()) throw new ValidationException("Name required");
    if (password == null || password.isEmpty()) throw new ValidationException("Password required");
    if (password.length() < 8) throw new ValidationException("Password too short");
    if (!password.matches(".*[A-Z].*")) throw new ValidationException("Password needs uppercase");
    if (!password.matches(".*[0-9].*")) throw new ValidationException("Password needs digit");

    // Normalize data (lines 21-35)
    String normalizedEmail = email.toLowerCase().trim();
    String normalizedName = name.trim().replaceAll("\\s+", " ");
    String hashedPassword = passwordEncoder.encode(password);
    String assignedRole = "admin".equals(role) ? "user" : role != null ? role : "user";
    LocalDateTime createdAt = LocalDateTime.now();
    LocalDateTime updatedAt = LocalDateTime.now();

    // Check duplicates and persist (lines 36-55)
    if (userRepository.existsByEmail(normalizedEmail))
        throw new ValidationException("Email already registered");
    User user = userRepository.save(User.builder()
        .email(normalizedEmail)
        .name(normalizedName)
        .password(hashedPassword)
        .role(assignedRole)
        .createdAt(createdAt)
        .updatedAt(updatedAt)
        .build());

    // Send notifications (lines 56-80+)
    String welcomeHtml = "<h1>Welcome " + normalizedName + "</h1><p>Your account is ready.</p>";
    emailService.send(EmailRequest.builder()
        .to(normalizedEmail)
        .subject("Welcome!")
        .html(welcomeHtml)
        .build());
    analyticsService.track("user_registered", Map.of(
        "userId", user.getId(),
        "role", assignedRole,
        "timestamp", createdAt.toString()
    ));
    auditLog.record("registration", Map.of("userId", user.getId(), "email", normalizedEmail));

    return user;
}
```

## Correct

Each responsibility is extracted into a focused method under 50 lines. Methods that are only used together stay in the same class.

```java
public UserInput validateRegistrationInput(Object input) {
    if (input == null) throw new ValidationException("Invalid input");
    Map<String, Object> data = (Map<String, Object>) input;
    String email = (String) data.get("email");
    String name = (String) data.get("name");
    String password = (String) data.get("password");
    String role = (String) data.get("role");
    if (email == null || email.isEmpty()) throw new ValidationException("Email required");
    if (name == null || name.isEmpty()) throw new ValidationException("Name required");
    if (password == null || password.isEmpty()) throw new ValidationException("Password required");
    if (password.length() < 8) throw new ValidationException("Password too short");
    if (!password.matches(".*[A-Z].*")) throw new ValidationException("Password needs uppercase");
    if (!password.matches(".*[0-9].*")) throw new ValidationException("Password needs digit");
    return new UserInput(email, name, password, role != null ? role : "user");
}

public NormalizedUser normalizeAndHash(UserInput input) {
    return NormalizedUser.builder()
        .email(input.email().toLowerCase().trim())
        .name(input.name().trim().replaceAll("\\s+", " "))
        .password(passwordEncoder.encode(input.password()))
        .role("admin".equals(input.role()) ? "user" : input.role())
        .build();
}

public User persistUser(NormalizedUser data) {
    if (userRepository.existsByEmail(data.getEmail()))
        throw new ValidationException("Email already registered");
    return userRepository.save(data.toUser(LocalDateTime.now()));
}

public void notifyRegistration(User user) {
    emailService.send(EmailRequest.builder()
        .to(user.getEmail())
        .subject("Welcome!")
        .html("<h1>Welcome " + user.getName() + "</h1>")
        .build());
    analyticsService.track("user_registered", Map.of("userId", user.getId(), "role", user.getRole()));
    auditLog.record("registration", Map.of("userId", user.getId(), "email", user.getEmail()));
}

public User processUserRegistration(Object input) {
    UserInput validated = validateRegistrationInput(input);
    NormalizedUser normalized = normalizeAndHash(validated);
    User user = persistUser(normalized);
    notifyRegistration(user);
    return user;
}
```
