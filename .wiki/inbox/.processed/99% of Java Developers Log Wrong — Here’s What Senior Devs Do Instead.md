
# [99% of Java Developers Log Wrong — Here’s What Senior Devs Do Instead](https://blog.stackademic.com/99-of-java-developers-log-wrong-heres-what-senior-devs-do-instead-ec8d21cffed6)

When your app crashes at 2 AM, useless logs turn a 5-minute fix into a 5-hour nightmare. Here’s what real engineers do differently.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*768-qADvNRb5bJxUL3_aBA.png)

If you are a non-member user, you can read it from [***HERE***](https://blog.stackademic.com/99-of-java-developers-log-wrong-heres-what-senior-devs-do-instead-ec8d21cffed6?sk=60c6de0392b10a0741bfaee96438757c).

When I started as a developer, logging meant adding `System.out.println()` everywhere. My first production issue changed that forever. The logs showed "Error: null" a thousand times. Useless.

My senior teammate fixed it with one line: “Added the order ID to the error log.” Suddenly, we could find the broken order.

Then he looked at my logging code and said, “Add the order ID to every log statement. If you can’t search for it, it didn’t happen.”

That one piece of advice changed everything. Production debugging went from “panic and guess” to “search and solve.” The difference wasn’t my code — it was my logging.

Today, I’ll show you the five patterns that separate developers who write logs from developers who debug with them.

## The Problem with Most Logging

Here’s what junior logging looks like:

```c
@Service
public class OrderService {
    
    public void processOrder(Order order) {
        System.out.println("Processing order");
        
        try {
            paymentService.charge(order);
            System.out.println("Success!");
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

This code has catastrophic logging problems:

- No context (which order failed?)
- No user identification (who was affected?)
- No searchable identifiers (how do I find this in logs?)
- Wrong log level (System.out instead of logger)
- Meaningless messages (“Success!” tells you nothing)

Let’s fix this with patterns that actually work in production.

## Pattern 1: Every Log Needs Context — Who and What

**The Problem:** Generic log messages are impossible to search and provide zero debugging value.

**The Solution:** Include identifiers that let you trace the entire flow.

```c
@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public void processOrder(Order order) {
        // Context-rich info log
        logger.info("Processing order", 
            "orderId", order.getId(),
            "userId", order.getUserId(),
            "itemCount", order.getItems().size(),
            "total", order.getTotal()
        );
        
        try {
            long startTime = System.currentTimeMillis();
            paymentService.charge(order);
            
            // Success with metrics
            logger.info("Order completed successfully",
                "orderId", order.getId(),
                "userId", order.getUserId(),
                "processingTimeMs", System.currentTimeMillis() - startTime
            );
            
        } catch (PaymentException e) {
            // Error with full context
            logger.error("Order payment failed",
                "orderId", order.getId(),
                "userId", order.getUserId(),
                "amount", order.getTotal(),
                "reason", e.getReason(),
                "errorCode", "PAY-101"
            );
            throw e;
        }
    }
}
```

**Why This Impresses:**

- Can search logs by `orderId` to see the full order flow
- Can search by `userId` to see all user activity
- Error codes enable filtering by error type
- Processing time reveals performance issues
- Every question “What happened?” has an answer in the logs

## Pattern 2: Use Log Levels Like They Actually Mean Something

**The Problem:** Most developers only use ERROR and INFO, making logs impossible to filter.

**The Solution:** Each log level has a specific purpose in production.

```c
@Service
public class PaymentService {
    
    private static final Logger logger = LoggerFactory.getLogger(PaymentService.class);
    
    public Payment charge(Order order) {
        // DEBUG - Development only, turned off in production
        logger.debug("Entering payment processing",
            "orderId", order.getId(),
            "gatewayConfig", getGatewayConfig()
        );
        
        // INFO - Normal business operations
        logger.info("Charging payment",
            "orderId", order.getId(),
            "amount", order.getTotal(),
            "gateway", "Stripe"
        );
        
        Payment payment = stripeGateway.charge(order);
        
        // WARN - Something unusual but handled
        if (payment.getProcessingTime() > 2000) {
            logger.warn("Payment processing slow",
                "orderId", order.getId(),
                "expectedMs", 500,
                "actualMs", payment.getProcessingTime(),
                "gateway", "Stripe"
            );
        }
        
        // ERROR - Actual failure that needs investigation
        if (payment.isFailed()) {
            logger.error("Payment charge failed",
                "orderId", order.getId(),
                "errorCode", "PAY-101",
                "reason", payment.getFailureReason()
            );
            throw new PaymentException(payment.getFailureReason());
        }
        
        return payment;
    }
}
```

**Log Level Guide:**

- **DEBUG**: Internal state for development (turn off in production)
- **INFO**: Normal business events (order created, payment succeeded)
- **WARN**: Unexpected but handled situations (slow response, retries)
- **ERROR**: Actual failures requiring attention (payment declined, database down)

**Why This Impresses:**

- Production logs stay clean (only INFO, WARN, ERROR)
- Can filter ERROR logs to see only real problems
- WARN logs catch performance degradation before it becomes critical
- On-call engineers know ERROR = page immediately

## Pattern 3: Don’t Kill Performance with Expensive Logging

**The Problem:** String concatenation and expensive operations run even when logging is disabled.

**The Solution:** Use conditional logging and parameterized messages.

```c
@Service
public class InventoryService {
    
    private static final Logger logger = LoggerFactory.getLogger(InventoryService.class);
    
    public void checkInventory(List<OrderItem> items) {
        // ❌ BAD - Builds string even when DEBUG is off
        logger.debug("Checking inventory for items: " + 
            items.stream()
                .map(OrderItem::toString)
                .collect(Collectors.joining(", ")));
        // This toString() and joining happens ALWAYS!
        
        // ✅ GOOD - Only evaluates when DEBUG is enabled
        if (logger.isDebugEnabled()) {
            logger.debug("Checking inventory for items: {}",
                items.stream()
                    .map(OrderItem::toString)
                    .collect(Collectors.joining(", ")));
        }
        
        // ✅ EVEN BETTER - Lazy evaluation with lambda
        logger.debug("Checking inventory for items", 
            "itemDetails", () -> items.stream()
                .map(OrderItem::toString)
                .collect(Collectors.joining(", ")));
        // Lambda only executes if DEBUG is on
    }
}
```

**Performance Impact:**

```c
// Bad logging on 1M requests/day
- String concatenation: 30ms × 1M = 8.3 hours CPU time wasted
- Method calls that don't need to run: 20ms × 1M = 5.6 hours wasted

// Good logging on 1M requests/day  
- Conditional check: 2ms × 1M = 33 minutes CPU time
- Savings: 13.6 hours of CPU time per day
```

**Why This Impresses:**

- Shows understanding of Java performance
- Prevents production slowdowns from debug logging
- Demonstrates awareness of hidden costs
- Uses modern Java features (lambdas for lazy evaluation

## Pattern 4: Make Errors Searchable with Error Codes

**The Problem:** Generic error messages make it impossible to filter logs or set up alerts.

**The Solution:** Use structured error codes and consistent fields.

```c
@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public void processOrder(Order order) {
        try {
            validateOrder(order);
            reserveInventory(order);
            chargePayment(order);
            confirmOrder(order);
            
        } catch (InsufficientInventoryException e) {
            logger.error("Order failed - insufficient inventory",
                "errorCode", "INV-101",
                "orderId", order.getId(),
                "userId", order.getUserId(),
                "productId", e.getProductId(),
                "requested", e.getRequestedQuantity(),
                "available", e.getAvailableQuantity()
            );
            throw e;
            
        } catch (PaymentDeclinedException e) {
            logger.error("Order failed - payment declined",
                "errorCode", "PAY-101",
                "orderId", order.getId(),
                "userId", order.getUserId(),
                "amount", order.getTotal(),
                "reason", e.getDeclineReason(),
                "retryable", e.isRetryable()
            );
            throw e;
            
        } catch (Exception e) {
            logger.error("Order failed - unexpected error",
                "errorCode", "ORD-999",
                "orderId", order.getId(),
                "userId", order.getUserId(),
                "errorType", e.getClass().getSimpleName(),
                "message", e.getMessage()
            );
            throw e;
        }
    }
}
```

**Error Code Convention:**

- `AUTH-001` = Invalid credentials
- `PAY-101` = Payment declined
- `INV-101` = Insufficient inventory
- `DB-201` = Database connection failed
- `API-301` = External API timeout
- `ORD-999` = Unexpected order error

**Why This Impresses:**

- Can search `errorCode:"PAY-101"` to find all payment failures
- Can set up alerts: “Page me if `errorCode:PAY-101` > 100/hour."
- Error codes enable analytics and dashboards
- Consistent structure makes log parsing reliable

## Pattern 5: Connect Related Logs with Correlation IDs

**The Problem:** When a user reports “My order failed,” you can’t find all related logs across different services.

**The Solution:** Use MDC (Mapped Diagnostic Context) to add correlation IDs to every log.

```c
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderController.class);
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        // Generate unique request ID
        String requestId = UUID.randomUUID().toString();
        
        try {
            // Add to MDC - automatically included in ALL logs
            MDC.put("requestId", requestId);
            MDC.put("userId", request.getUserId());
            
            logger.info("Creating order",
                "itemCount", request.getItems().size()
                // Automatically includes: requestId, userId
            );
            
            Order order = orderService.createOrder(request);
            
            logger.info("Order created successfully",
                "orderId", order.getId()
                // Still includes: requestId, userId
            );
            
            return ResponseEntity.ok(OrderResponse.from(order));
            
        } finally {
            // CRITICAL: Clean up MDC
            MDC.clear();
        }
    }
}

@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(OrderRequest request) {
        // This log automatically includes requestId and userId from MDC!
        logger.info("Processing order request",
            "itemCount", request.getItems().size()
        );
        
        Order order = buildOrder(request);
        orderRepository.save(order);
        
        // This too!
        logger.info("Order persisted",
            "orderId", order.getId()
        );
        
        return order;
    }
}
```

**Logs now look like this:**

```c
{"time":"10:00:01","level":"INFO","requestId":"abc-123","userId":"789","msg":"Creating order","itemCount":3}
{"time":"10:00:02","level":"INFO","requestId":"abc-123","userId":"789","msg":"Processing order request","itemCount":3}
{"time":"10:00:03","level":"INFO","requestId":"abc-123","userId":"789","msg":"Order persisted","orderId":"456"}
{"time":"10:00:04","level":"INFO","requestId":"abc-123","userId":"789","msg":"Order created successfully","orderId":"456"}
```

**To debug, just search:**

```c
grep 'requestId":"abc-123"' app.log
```

Get the entire request flow across all classes and methods.

**Why This Impresses:**

- Trace the complete user journey across services
- Debug distributed systems effectively
- No manual ID passing between methods
- Industry-standard pattern used at scale

## From Noise to Signal

The difference between junior and senior logging isn’t the number of log statements — it’s what those statements contain.

**Before (Junior logging):**

```c
try {
    processOrder(order);
    System.out.println("Success!");
} catch (Exception e) {
    System.out.println("Error: " + e.getMessage());
}
```

**After (Senior logging):**

```c
logger.info("Processing order",
    "orderId", order.getId(),
    "userId", order.getUserId()
);

try {
    processOrder(order);
    
    logger.info("Order processed successfully",
        "orderId", order.getId(),
        "processingTimeMs", processingTime
    );
    
} catch (PaymentException e) {
    logger.error("Order processing failed",
        "errorCode", "PAY-101",
        "orderId", order.getId(),
        "userId", order.getUserId(),
        "reason", e.getReason()
    );
    throw e;
}
```

These patterns aren’t about writing perfect logs — they’re about writing debuggable systems. Because bugs will happen, production will break. Your job is to make finding the problem take five minutes instead of five hours.

The senior developer who taught me to “add the order ID” wasn’t being pedantic. He was teaching me that logs are your best friend during an outage, but only if you can actually search them.

Start with one pattern this week. Add IDs to your error logs. Fix one log level. Add one correlation ID. Small changes compound into debuggable systems.

**Which pattern will you implement first? Let me know in the comments what logging problem you’re facing right now.**