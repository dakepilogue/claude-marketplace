---
title: Enforce Separation of Concerns Between Layers
paths:
  - "src/**/*"
impact: HIGH
---

# Enforce Separation of Concerns Between Layers

Do NOT mix business logic with REST controllers or place database queries directly in service classes. Each architectural layer must have a single responsibility: controllers handle HTTP concerns, services encapsulate business logic, and repositories manage data access. Violating these boundaries creates tightly coupled code that is difficult to test, refactor, and reason about. When business rules live inside controllers, they cannot be reused across different entry points (REST, gRPC, events) and changes to infrastructure leak into domain logic. Maintain clear boundaries between contexts by delegating work through well-defined interfaces rather than inlining cross-cutting concerns.

## Critical principles

- Do NOT mix business logic with UI components
- Keep database queries out of controllers
- Maintain clear boundaries between contexts
- Ensure proper separation of responsibilities

## Incorrect

The controller mixes HTTP handling, business logic, and database queries in a single method, making it impossible to reuse or test the business rules independently.

```java
// OrderController.java — everything in one place
@RestController
public class OrderController {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest req) {
        // Database query directly in controller
        Customer customer = jdbcTemplate.queryForObject(
            "SELECT * FROM customers WHERE id = ?",
            new Object[]{req.getCustomerId()},
            (rs, rowNum) -> mapCustomer(rs)
        );
        if (customer == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Customer not found");
        }

        // Business logic mixed into controller
        double total = 0;
        for (OrderItem item : req.getItems()) {
            Product product = jdbcTemplate.queryForObject(
                "SELECT * FROM products WHERE id = ?",
                new Object[]{item.getProductId()},
                (rs, rowNum) -> mapProduct(rs)
            );
            total += product.getPrice() * item.getQuantity();
        }
        if (total > 10000) {
            total = total * 0.9; // 10% discount for large orders
        }

        // More database queries inline
        Order order = jdbcTemplate.queryForObject(
            "INSERT INTO orders (customer_id, total) VALUES (?, ?) RETURNING *",
            new Object[]{req.getCustomerId(), total},
            (rs, rowNum) -> mapOrder(rs)
        );

        return order;
    }
}
```

## Correct

The controller delegates to a service for business logic and a repository for data access. Each layer has a single responsibility and can be tested and reused independently.

```java
// OrderController.java — handles HTTP only
@RestController
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest req) {
        return orderService.createOrder(req.getCustomerId(), req.getItems());
    }
}

// OrderService.java — business logic only
@Service
public class OrderService {
    private final CustomerRepository customerRepository;
    private final ProductRepository productRepository;
    private final OrderRepository orderRepository;

    public OrderService(CustomerRepository customerRepository,
                       ProductRepository productRepository,
                       OrderRepository orderRepository) {
        this.customerRepository = customerRepository;
        this.productRepository = productRepository;
        this.orderRepository = orderRepository;
    }

    public Order createOrder(String customerId, List<OrderItem> items) {
        Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new NotFoundException("Customer not found"));

        double total = calculateTotal(items);
        return orderRepository.create(new Order(customerId, total));
    }

    private double calculateTotal(List<OrderItem> items) {
        double total = 0;
        for (OrderItem item : items) {
            Product product = productRepository.findById(item.getProductId());
            total += product.getPrice() * item.getQuantity();
        }
        return total > 10000 ? total * 0.9 : total;
    }
}

// OrderRepository.java — data access only
@Repository
public class OrderRepository {
    private final JdbcTemplate jdbcTemplate;

    public OrderRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Order create(Order order) {
        return jdbcTemplate.queryForObject(
            "INSERT INTO orders (customer_id, total) VALUES (?, ?) RETURNING *",
            (rs, rowNum) -> mapOrder(rs),
            order.getCustomerId(),
            order.getTotal()
        );
    }
}
```

## Reference

- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
