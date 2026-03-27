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
  - HTTP requests → RestTemplate, WebClient
  - Utility functions → Apache Commons Lang, Guava

#### Code Quality

- Use builder pattern for complex object construction:
  - Instead of `new User(name, email, age)` use builder
  - Instead of `new Order(itemId, quantity, price)` use builder with fluent API
- Use Optional for null-returning methods instead of returning null
- Use @Slf4j for logging instead of System.out.println

## Technology Stack

**Core:**
- Java 17 (modern LTS with records, sealed classes, pattern matching)
- Spring Boot 3.5.5, Spring Cloud 2025.0.0
- Maven 3.x+

**Data Access:**
- MyBatis-Plus 3.5.7 (ORM with automatic CRUD)
- MySQL 8.0+ (primary storage)
- Redis 6.2+ (cache, locks, pub/sub)
- Redisson 3.32.0 (advanced Redis client)

**Messaging:**
- Kafka 3.3.9 (event streaming)
- Netty 4.2.8.Final (async I/O)

**Service Discovery:**
- Nacos 2023.3 (registry & config center)
- OpenFeign 3.1.2 (declarative HTTP)

**Utilities:**
- Guava (Google utilities)
- Caffeine (local cache)
- Lombok 1.18.30 (boilerplate reduction)

## Naming Conventions

| Pattern       | Example               | Purpose                |
|---------------|-----------------------|------------------------|
| `*BO`         | `ModelBO`, `ImageBO`  | Business objects / DTOs |
| `*Mapper`     | `ModelMapper`         | mapper                 |
| `*Repository` | `ModelRepository`     | mybatis plus service   |
| `*Service`    | `BiSyncTaskService`   | Business logic layer   |
| `*DTO`        | `SyncTaskReq`         | Request/response objects |
| `*Entity`     | `PlatformTaskPO`      | Database entities      |
| `*Config`     | `BiSyncHubProperties` | Configuration beans    |
| `*Enum`       | `MessageTypeEnum`     | Type-safe enumerations |

## Package Organization

cn.telecom.bac.starocean.xxx.xxx/
├── po/          # bussiness entity
├── dto/          # DTO entity
├── repository/  # bussiness repository using mybatis plus ServiceImpl
├── mapper/      # mybatis plus Mapper
└── service/        # Service interface
    └── impl/        # Service implementations

## 实体类规则

- 实体类中的枚举字段需要使用枚举类定义，序列化的值需要是枚举值，通过@EnumValue注解定义
- 所有DTO和VO的枚举字段也需要使用枚举类定义
- service层中实体类转VO类，封装为统一的批量buildVOs转换方法，重名字段通过BeanUtils.copyProperties拷贝。并且分页和详情查询接口共用同一个VO类，如果字段有区别公共部分抽象到基类VO中，转换方法共用，如果有特殊处理在单独定义一个方法，并复用基类的转换方法

## controller规则
所有controller以及全局异常处理的返回类型为cn.telecom.bac.starlink.common.core.util.R

## 异常处理

通过枚举ErrorCode来控制异常码，枚举类需要实现接口cn.telecom.bac.starocean.aip.common.core.enums.IErrorCode
并通过springboot的static/i18n/message_en.properties（英文）、message_zh_CN.properties（中文）来控制不同语言的提示消息
出现异常时抛出BusinessException异常，传入ErrorCode，并通过全局异常处理来捕获异常并返回提示消息

## 其他规则
- json的序列化和反序列化用jackson
```

