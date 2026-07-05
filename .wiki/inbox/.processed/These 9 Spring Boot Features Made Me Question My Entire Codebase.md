
# [These 9 Spring Boot Features Made Me Question My Entire Codebase](https://blog.stackademic.com/these-9-spring-boot-features-made-me-question-my-entire-codebase-946360b564b8)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*tzqjU-b9sFIMHEaIvtlBPw.png)

## Most Spring Boot developers use only 20% of what the framework can actually do.

A few months ago, I was reviewing a backend service that had grown into a monster.

The application had dozens of REST endpoints, hundreds of classes, custom utility methods everywhere, and configuration files that looked like ancient scrolls nobody wanted to touch.

The funny part?

The team considered themselves “experienced Spring Boot developers.”

And they were.

But while reviewing the code, I noticed something interesting.

They were using Spring Boot every day, yet they were ignoring several features that Spring Boot provides out of the box.

Instead of using built-in capabilities, they had written hundreds of lines of custom code.

That’s when it hit me.

Most developers learn enough Spring Boot to build applications and then stop exploring.

As a result, they keep solving problems manually that Spring Boot already solved years ago.

Here are 9 powerful Spring Boot features that many developers either don’t know about or rarely use — and why they can make your applications cleaner, faster, and easier to maintain.

## 1\. Application Events

Most developers tightly couple components together.

One service calls another.

That service calls another.

And suddenly a simple operation creates a chain of dependencies that nobody wants to touch.

For example:

```c
userService.createUser();
emailService.sendWelcomeEmail();
auditService.logUserCreation();
analyticsService.trackSignup();
```

This works.

Until requirements change.

Spring Boot provides a much cleaner approach through Application Events.

Create an event:

```c
public class UserCreatedEvent {
    private final User user;

public UserCreatedEvent(User user) {
        this.user = user;
    }
    public User getUser() {
        return user;
    }
}
```

Publish it:

```c
applicationEventPublisher.publishEvent(
    new UserCreatedEvent(user)
);
```

Listen for it:

```c
@EventListener
public void handleUserCreated(UserCreatedEvent event) {
    emailService.sendWelcomeEmail(event.getUser());
}
```

Another listener:

```c
@EventListener
public void audit(UserCreatedEvent event) {
    auditService.log(event.getUser());
}
```

Now your services don’t know about each other.

Adding new functionality becomes easy.

This is one of the cleanest ways to reduce coupling in large applications.

## 2\. ConfigurationProperties

I’ve seen developers inject configuration values like this:

```c
@Value("${app.name}")
private String appName;

@Value("${app.timeout}")
private int timeout;
@Value("${app.max-users}")
private int maxUsers;
```

At first it looks harmless.

Then six months later there are fifty of them scattered across the codebase.

Spring Boot offers a much cleaner solution.

```c
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {

private String name;
    private int timeout;
    private int maxUsers;
    // getters and setters
}
```

Configuration:

```c
app:
  name: MyApp
  timeout: 5000
  max-users: 1000
```

Usage:

```c
@Autowired
private AppProperties properties;
```

Benefits:

- Type safety
- Better organization
- Easier testing
- Cleaner code

Once you switch to this approach, you’ll never want dozens of `@Value` annotations again.

## 3\. Profiles Beyond Development and Production

Most projects have only:

```c
application-dev.yml
application-prod.yml
```

That’s it.

But profiles can be far more useful.

For example:

```c
application-local.yml
application-staging.yml
application-loadtest.yml
application-integration.yml
```

You can activate them:

```c
-Dspring.profiles.active=staging
```

Or:

```c
spring:
  profiles:
    active: local
```

This becomes incredibly useful when dealing with:

- Different databases
- Mock services
- Third-party integrations
- Load testing environments

A good profile strategy can save countless deployment headaches.

## 4\. Actuator Endpoints

Many developers install Spring Boot Actuator.

Very few actually use it properly.

Add dependency:

```c
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Now you instantly get useful endpoints:

```c
/actuator/health
/actuator/info
/actuator/metrics
/actuator/env
/actuator/beans
```

Need to know if your application is alive?

```c
/actuator/health
```

Need JVM metrics?

```c
/actuator/metrics
```

Need memory usage?

```c
/actuator/metrics/jvm.memory.used
```

The amount of production debugging time this saves is ridiculous.

## 5\. Conditional Bean Creation

A common anti-pattern looks like this:

```c
if(environment.equals("prod")) {
    return new RealPaymentService();
} else {
    return new MockPaymentService();
}
```

Spring Boot provides conditional beans.

```c
@Bean
@ConditionalOnProperty(
    name = "payment.mock",
    havingValue = "true"
)
public PaymentService mockPaymentService() {
    return new MockPaymentService();
}
```

And:

```c
@Bean
@ConditionalOnProperty(
    name = "payment.mock",
    havingValue = "false"
)
public PaymentService realPaymentService() {
    return new RealPaymentService();
}
```

Configuration decides which bean gets loaded.

No ugly if-else blocks.

No manual wiring.

Clean and scalable.

## 6\. CommandLineRunner

Many teams create startup classes manually.

Spring Boot already has a feature for startup logic.

```c
@Component
public class StartupRunner implements CommandLineRunner {

@Override
    public void run(String... args) {
        System.out.println("Application Started");
    }
}
```

Useful for:

- Initial data loading
- Cache warming
- Startup validation
- Health checks
- Database verification

A surprisingly simple feature that solves many startup requirements.

## 7\. Scheduling Without Extra Frameworks

I’ve seen developers introduce Quartz for tiny scheduling requirements.

Sometimes they only need one scheduled job.

Spring Boot already includes scheduling.

Enable:

```c
@EnableScheduling
```

Create a task:

```c
@Scheduled(fixedRate = 5000)
public void executeTask() {
    System.out.println("Running...");
}
```

Or use cron:

```c
@Scheduled(cron = "0 0 * * * *")
public void generateReport() {
    reportService.generate();
}
```

Done.

No additional framework required.

No complicated setup.

Just a few lines of code.

## 8\. Bean Validation

Validation logic often becomes messy.

Example:

```c
if(user.getEmail() == null) {
    throw new RuntimeException();
}

if(user.getAge() < 18) {
    throw new RuntimeException();
}
```

Now imagine this repeated everywhere.

Spring Boot integrates Bean Validation beautifully.

DTO:

```c
public class UserRequest {

@NotBlank
    private String email;
    @Min(18)
    private int age;
}
```

Controller:

```c
@PostMapping
public ResponseEntity<?> createUser(
        @Valid @RequestBody UserRequest request) {

return ResponseEntity.ok().build();
}
```

Spring automatically validates requests.

Less boilerplate.

Cleaner controllers.

More readable code.

## 9\. ProblemDetail for Better API Errors

Many APIs still return errors like this:

```c
{
  "message": "Something went wrong"
}
```

Which tells clients absolutely nothing.

Modern Spring Boot supports `ProblemDetail`.

```c
ProblemDetail problem =
    ProblemDetail.forStatus(HttpStatus.NOT_FOUND);

problem.setTitle("User Not Found");
problem.setDetail("User with ID 100 does not exist");
return ResponseEntity.of(problem).build();
```

Response:

```c
{
  "type": "about:blank",
  "title": "User Not Found",
  "status": 404,
  "detail": "User with ID 100 does not exist"
}
```

Cleaner.

Standardized.

More useful for frontend developers and API consumers.

## The Real Lesson

The biggest Spring Boot productivity boost doesn’t come from learning another framework.

It comes from learning Spring Boot itself.

Many teams add new libraries every month while ignoring features already sitting inside their existing dependencies.

The result is often:

- More code
- More bugs
- More maintenance
- More complexity

Meanwhile, Spring Boot quietly offers solutions that are battle-tested, production-ready, and maintained by the framework itself.

The next time you think, “I need to build a custom solution for this,” spend five minutes checking whether Spring Boot already has one.

There’s a good chance it does.

And it probably does it better than the code you’re about to write.