---
title: Separate Domain Logic from Infrastructure
paths:
  - "src/**/*"
impact: HIGH
---

# Separate Domain Logic from Infrastructure

Keep business logic in pure domain and use case layers, free of framework or infrastructure dependencies. When domain logic is coupled to controllers, ORMs, or HTTP libraries, it becomes untestable in isolation, impossible to reuse across delivery mechanisms, and fragile to infrastructure changes. Define domain entities that model business rules with no imports from framework or database packages. Implement use cases as classes that depend on abstract repository interfaces, not concrete database clients. Let the infrastructure layer implement those interfaces and inject them at composition time. This dependency inversion ensures the domain drives the architecture rather than the framework dictating how business rules are organized.

## Critical Clean Architecture & DDD Principles

- Separate domain entities from infrastructure concerns
- Keep business logic independent of frameworks
- Define use cases clearly and keep them isolated
- Avoid code duplication through creation of reusable functions and modules

## Incorrect

Business logic is embedded directly in the REST controller, coupled to Spring MVC and database client. Testing requires spinning up the full application context and database.

```java
@RestController
public class OrderController {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest request) {
        // Business rule mixed into the controller
        double total = request.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQty())
            .sum();
        double discount = total > 100 ? total * 0.1 : 0;

        jdbcTemplate.update(
            "INSERT INTO orders (customer_id, total) VALUES (?, ?)",
            request.getCustomerId(), total - discount
        );

        return new Order(request.getCustomerId(), total - discount);
    }
}
```

Poor Architectural Choices:
- Mixing business logic with UI components
- Database queries directly in controllers
- Lack of clear separation of concerns

## Correct

Domain logic lives in a framework-free use case that depends on an abstract repository. The controller is a thin adapter that delegates to the use case.

```java
// domain/OrderDomain.java — pure business logic, no framework imports
public class OrderDomain {
    public static double calculateOrderTotal(List<OrderItem> items) {
        double subtotal = items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQty())
            .sum();
        double discount = subtotal > 100 ? subtotal * 0.1 : 0;
        return subtotal - discount;
    }
}

// application/CreateOrderUseCase.java — use case depends on abstraction
public class CreateOrderUseCase {
    private final OrderRepository orderRepository;

    public CreateOrderUseCase(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order execute(String customerId, List<OrderItem> items) {
        double total = OrderDomain.calculateOrderTotal(items);
        return orderRepository.save(new Order(customerId, total, items));
    }
}

// infrastructure/OrderController.java — thin adapter
@RestController
public class OrderController {
    private final CreateOrderUseCase createOrderUseCase;

    public OrderController(CreateOrderUseCase createOrderUseCase) {
        this.createOrderUseCase = createOrderUseCase;
    }

    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest request) {
        return createOrderUseCase.execute(request.getCustomerId(), request.getItems());
    }
}
```

## Reference

- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)
