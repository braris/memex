---
title: "Junior Devs Use @Service. Senior Devs Use These 6 Spring Boot Layering Patterns Instead"
source: "https://blog.stackademic.com/junior-devs-use-service-senior-devs-use-these-6-spring-boot-layering-patterns-instead-e36074e1999b"
type: articles
ingested: 2026-07-05
tags: [spring-boot, java, software-architecture, layering, domain-driven-design, hexagonal-architecture]
summary: "Stackademic article explaining Spring Boot layering patterns such as domain vs application services, rich domain models, use cases, anti-corruption layers, DTO boundaries, and ports and adapters."
---

# [Junior Devs Use @Service. Senior Devs Use These 6 Spring Boot Layering Patterns Instead](https://blog.stackademic.com/junior-devs-use-service-senior-devs-use-these-6-spring-boot-layering-patterns-instead-e36074e1999b)

From 500-line god services to clean, testable architecture. The 6 patterns that transformed how I design Spring Boot apps.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*6y6wdYfv9z2ypnwsB4r-JQ.png)

Non-member user? Read the article from [***HERE***](https://blog.stackademic.com/junior-devs-use-service-senior-devs-use-these-6-spring-boot-layering-patterns-instead-e36074e1999b?sk=351a9e2bf34c1c77a44d3f2bf2b271f5).

I was three months into my first Spring Boot project when my tech lead rejected my pull request with a single comment:

â€œYour service layer is doing everything â€” data access, validation, business logic, external API calls, and email sending. This isnâ€™t a service, itâ€™s a junk drawer. Senior developers separate concerns. Junior developers collect them.â€

That comment stung. But it also opened my eyes to something Iâ€™d never learned in tutorials: architecture isnâ€™t about making code work, itâ€™s about making it maintainable.

Today, Iâ€™ll share the six layering patterns that transformed me from a @Service spammer into someone who actually designs systems.

## The Problem with @Service Everywhere

Most Spring Boot developers organize code like this:

```c
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private PaymentGateway paymentGateway;
    
    public Order createOrder(OrderRequest request) {
        // Validation
        if (request.getItems().isEmpty()) {
            throw new IllegalArgumentException("No items");
        }
        
        // Business logic
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setTotal(calculateTotal(request.getItems()));
        
        // Data access
        order = orderRepository.save(order);
        
        // External API
        paymentGateway.charge(order.getTotal());
        
        // Side effects
        emailService.sendConfirmation(order);
        
        return order;
    }
}
```

This â€œgod serviceâ€ has critical problems:

- Impossible to test without mocking everything
- Business logic mixed with infrastructure concerns
- Changes in one area break unrelated functionality
- No clear boundaries or responsibilities
- Violates the Single Responsibility Principle spectacularly

Letâ€™s fix this with patterns that actually scale.

## Pattern 1: Domain Service vs Application Service

**The Problem:** Mixing orchestration logic with pure business rules makes both harder to test and reuse.

**The Solution:** Separate domain logic from application workflows.

```c
// Domain Service - Pure business logic, no infrastructure
@Component
public class OrderPricingService {
    
    public BigDecimal calculateTotal(List<OrderItem> items) {
        BigDecimal subtotal = items.stream()
            .map(item -> item.getPrice().multiply(
                BigDecimal.valueOf(item.getQuantity())
            ))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        return applyDiscounts(subtotal, items);
    }
    
    public boolean isEligibleForFreeShipping(BigDecimal total) {
        return total.compareTo(new BigDecimal("50.00")) >= 0;
    }
}

// Application Service - Orchestrates workflow
@Service
public class OrderApplicationService {
    
    private final OrderRepository orderRepository;
    private final OrderPricingService pricingService;
    private final ApplicationEventPublisher eventPublisher;
    
    @Transactional
    public Order createOrder(CreateOrderCommand command) {
        // Use domain service for business logic
        BigDecimal total = pricingService.calculateTotal(
            command.getItems()
        );
        
        Order order = Order.create(
            command.getUserId(),
            command.getItems(),
            total
        );
        
        order = orderRepository.save(order);
        
        // Publish event for side effects
        eventPublisher.publishEvent(new OrderCreatedEvent(order));
        
        return order;
    }
}
```

**Why This Matters:**

- Domain logic is testable without Spring context
- Business rules are reusable across different workflows
- Clear separation between â€œwhatâ€ (domain) and â€œhowâ€ (application)
- Easy to move domain logic to different frameworks

## Pattern 2: Repository vs DAO Pattern

**The Problem:** Using repositories like data mappers, loading entities just to call setters.

```c
// âŒ Bad: Anemic domain, repository doing too much
@Service
public class UserService {

    public void updateEmail(Long userId, String newEmail) {
        User user = userRepository.findById(userId).orElseThrow();
        user.setEmail(newEmail);
        userRepository.save(user);
    }
}
```

**The Solution:** Rich domain models with repositories focused on aggregate retrieval.

```c
// Rich domain model with business logic
@Entity
public class User {
    
    @Id
    private Long id;
    
    private String email;
    private boolean emailVerified;
    private LocalDateTime emailChangedAt;
    
    public void changeEmail(String newEmail) {

        if (newEmail.equals(this.email)) {
            throw new IllegalArgumentException("Email unchanged");
        }
        
        this.email = newEmail;
        this.emailVerified = false;
        this.emailChangedAt = LocalDateTime.now();
    }
    
    public boolean canChangeEmail() {

        if (emailChangedAt == null) return true;

        return emailChangedAt.isBefore(
            LocalDateTime.now().minusDays(30)
        );
    }
}

// Repository focuses on aggregate access
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// Service orchestrates, domain enforces rules
@Service
public class UserService {
    
    @Transactional
    public void changeUserEmail(Long userId, String newEmail) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
        
        if (!user.canChangeEmail()) {
            throw new BusinessRuleException(
                "Email can only be changed once per 30 days"
            );
        }
        
        user.changeEmail(newEmail);
        // No need to call save() - managed entity
    }
}
```

**Why This Matters:**

- Business rules live in domain objects, not services
- Validation happens where data lives
- Impossible to bypass business logic
- Repositories stay focused on data access

## Pattern 3: Use Case / Interactor Pattern

**The Problem:** Service methods grow into 200-line monsters handling multiple scenarios.

**The Solution:** One use case class per business operation.

```c
// Each use case is a dedicated class
@Component
public class PlaceOrderUseCase {
    
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    
    @Transactional
    public OrderResult execute(PlaceOrderRequest request) {
        // 1. Validate inventory
        if (!inventoryService.areItemsAvailable(request.getItems())) {
            return OrderResult.failure("Items unavailable");
        }
        
        // 2. Reserve inventory
        inventoryService.reserve(request.getItems());
        
        // 3. Process payment
        PaymentResult payment = paymentService.charge(request);

        if (payment.failed()) {
            inventoryService.release(request.getItems());
            return OrderResult.failure("Payment failed");
        }
        
        // 4. Create order
        Order order = Order.from(request, payment.getTransactionId());
        order = orderRepository.save(order);
        
        return OrderResult.success(order);
    }
}

@Component
public class CancelOrderUseCase {
    
    private final OrderRepository orderRepository;
    private final RefundService refundService;
    
    @Transactional
    public void execute(Long orderId, String reason) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow();
        
        order.cancel(reason);
        refundService.process(order);
    }
}

// Controller delegates to use cases
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final PlaceOrderUseCase placeOrder;
    private final CancelOrderUseCase cancelOrder;
    
    @PostMapping
    public ResponseEntity<OrderResponse> create(
            @RequestBody PlaceOrderRequest request) {
        
        OrderResult result = placeOrder.execute(request);
        return result.isSuccess() 
            ? ResponseEntity.ok(OrderResponse.from(result.getOrder()))
            : ResponseEntity.badRequest().build();
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> cancel(
            @PathVariable Long id,
            @RequestBody CancelOrderRequest request) {
        
        cancelOrder.execute(id, request.getReason());
        return ResponseEntity.noContent().build();
    }
}
```

**Why This Matters:**

- Each business operation is isolated and testable
- Easy to understand what a use case does
- Adding features doesnâ€™t modify existing use cases
- Natural place for transaction boundaries

## Pattern 4: Anti-Corruption Layer

**The Problem:** External API changes break your entire application.

```c
// âŒ Bad: Direct dependency on external API

@Service
public class ProductService {
    
    @Autowired
    private StripeClient stripeClient; // Vendor lock-in
    
    public void createSubscription(User user) {
        stripeClient.createCustomer(user.getEmail());
        stripeClient.createSubscription(user.getId(), "premium");
    }
}
```

**The Solution:** Create an abstraction layer that translates between your domain and external systems.

```c
// Your domain interface
public interface PaymentProvider {
    PaymentResult charge(PaymentRequest request);
    RefundResult refund(String transactionId);
}

// Anti-corruption layer for Stripe
@Service
public class StripePaymentProvider implements PaymentProvider {
    
    private final StripeClient stripeClient;
    
    @Override
    public PaymentResult charge(PaymentRequest request) {
        // Translate domain model to Stripe API
        StripeChargeRequest stripeRequest = new StripeChargeRequest();

        stripeRequest.setAmount(request.getAmount().multiply(
            new BigDecimal("100")
        ).intValue()); // Stripe uses cents
        stripeRequest.setCurrency("usd");
        
        try {
            StripeCharge charge = stripeClient.charge(stripeRequest);
            return PaymentResult.success(charge.getId());
        } catch (StripeException e) {
            return PaymentResult.failure(e.getMessage());
        }
    }
}

// Your service depends on YOUR interface
@Service
public class OrderService {
    
    private final PaymentProvider paymentProvider; // Not StripeClient
    
    public Order processOrder(OrderRequest request) {
        PaymentResult result = paymentProvider.charge(
            PaymentRequest.from(request)
        );
        
        if (result.isSuccess()) {
            return createOrder(request, result.getTransactionId());
        }
        
        throw new PaymentFailedException();
    }
}
```

**Why This Matters:**

- Switch payment providers without touching business logic
- External API changes are contained in one layer
- Easy to mock for testing
- Your domain language stays pure

## Pattern 5: DTOs at Boundary, Domain Inside

**The Problem:** Passing request objects deep into your domain logic.

```c
// âŒ Bad: Controller DTO polluting domain
@Service
public class UserService {

    public User register(RegisterUserRequest request) {
        // Domain logic coupled to HTTP layer
        User user = new User();
        user.setEmail(request.getEmail());
        user.setPassword(request.getPassword());
        return userRepository.save(user);
    }
}
```

**The Solution:** Convert DTOs to domain objects at the boundary.

```c
// Controller DTO
public record RegisterUserRequest(
    @Email String email,
    @NotBlank String password,
    @NotBlank String firstName,
    @NotBlank String lastName
) {}

// Domain command (internal)
public record RegisterUserCommand(
    String email,
    String password,
    String firstName,
    String lastName
) {
    public static RegisterUserCommand from(RegisterUserRequest request) {
        return new RegisterUserCommand(
            request.email(),
            request.password(),
            request.firstName(),
            request.lastName()
        );
    }
}

// Controller converts at boundary
@RestController
public class UserController {
    
    @PostMapping("/register")
    public ResponseEntity<UserResponse> register(
            @Valid @RequestBody RegisterUserRequest request) {
        
        RegisterUserCommand command = RegisterUserCommand.from(request);
        User user = userService.register(command);
        
        return ResponseEntity.ok(UserResponse.from(user));
    }
}

// Service works with domain objects
@Service
public class UserService {
    
    public User register(RegisterUserCommand command) {
        // Pure domain logic, no web concerns
        User user = User.register(
            command.email(),
            passwordEncoder.encode(command.password()),
            command.firstName(),
            command.lastName()
        );
        
        return userRepository.save(user);
    }
}
```

**Why This Matters:**

- Domain logic is independent of the delivery mechanism
- Can add GraphQL and gRPC without changing services
- Validation stays at the boundary
- Domain models can evolve independently

## Pattern 6: Hexagonal Architecture (Ports & Adapters)

**The Problem:** Your business logic is tangled with Spring Framework and infrastructure.

**The Solution:** Define ports (interfaces) in your domain, implement adapters in infrastructure.

```c
// Domain layer - no Spring dependencies
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(Long id);
}

public interface NotificationService {
    void sendOrderConfirmation(Order order);
}

// Application service - uses ports
public class OrderApplicationService {
    
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;
    
    // Constructor injection of interfaces
    public OrderApplicationService(
            OrderRepository orderRepository,
            NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.notificationService = notificationService;
    }
    
    public Order placeOrder(OrderData data) {
        Order order = Order.create(data);
        order = orderRepository.save(order);
        notificationService.sendOrderConfirmation(order);
        return order;
    }
}

// Infrastructure adapters - implement ports
@Repository
public class JpaOrderRepository implements OrderRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public Order save(Order order) {
        entityManager.persist(order);
        return order;
    }
}

@Service
public class EmailNotificationService implements NotificationService {
    
    @Autowired
    private JavaMailSender mailSender;
    
    @Override
    public void sendOrderConfirmation(Order order) {
        // Email implementation
    }
}
```

**Why This Matters:**

- Business logic has zero framework dependencies
- Easy to test with simple implementations
- Can swap databases or notification providers
- Domain code is portable to any framework

## From Service Soup to Clean Architecture

The difference between junior and senior Spring Boot code isnâ€™t the number of @Service annotations â€” itâ€™s understanding that layers exist to isolate change and clarify responsibility.

**Before:**

```c
@Service
public class OrderService {
    // 500 lines of everything
}
```

**After:**

```c
@Component
public class OrderPricingService { /* domain logic */ }

@Service  
public class OrderApplicationService { /* orchestration */ }
public class PlaceOrderUseCase { /* single operation */ }
public class StripePaymentAdapter implements PaymentProvider { /* external */ }
```
