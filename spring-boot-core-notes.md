# Spring Boot Core Notes

## How To Use These Notes

This file is arranged as:

- Chapter 1: Spring Boot annotations, validation, configuration, JSON mapping, scheduling, caching, transactions, and bean lifecycle.
- Chapter 2: Old notes area. I could not find an existing old-notes file in this workspace, so Chapter 2 is ready for those notes when they are available.

---

# Chapter 1: Spring Boot Core Annotations And Concepts

## 1. Big Picture

Spring Boot is built around three core ideas:

1. Create objects called beans.
2. Store those beans in the Spring container.
3. Inject and use those beans automatically where needed.

Most annotations in this chapter tell Spring one of these things:

- This class is a Spring-managed bean.
- This method creates a bean.
- This method handles an HTTP request.
- This field should receive configuration.
- This request body should be validated.
- This error should be handled globally.
- This method should run on a schedule, use cache, or run in a transaction.

Common package layout:

```text
com.example.demo
├── DemoApplication.java
├── controller
├── service
├── repository
├── dto
├── entity
├── config
└── exception
```

Keep the main application class in the root package, such as `com.example.demo`, so `@ComponentScan` can find controllers, services, repositories, configuration classes, and other components in subpackages.

## 2. Application Startup

### `@SpringBootApplication`

`@SpringBootApplication` marks the main entry point of a Spring Boot app.

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Internally, `@SpringBootApplication` combines:

- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

Meaning:

- `@SpringBootConfiguration`: this class can define Spring configuration.
- `@EnableAutoConfiguration`: Spring Boot guesses and configures beans based on dependencies and properties.
- `@ComponentScan`: Spring scans the current package and child packages for beans.

### `@ComponentScan`

`@ComponentScan` tells Spring where to search for annotated classes like `@Controller`, `@Service`, `@Repository`, and `@Component`.

Most of the time you do not need to write it manually because `@SpringBootApplication` already includes it.

Manual example:

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.example")
public class DemoApplication {
}
```

Use manual `@ComponentScan` only when your beans are outside the package tree of the main application class.

## 3. Configuration And Bean Creation

### `@Configuration`

`@Configuration` marks a class as a source of bean definitions.

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
}
```

Use it when you want to define setup logic, third-party objects, or custom beans.

### `@Bean`

`@Bean` marks a method whose return value should be stored in the Spring container.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public TaxCalculator taxCalculator() {
        return new TaxCalculator(18);
    }
}
```

Use `@Bean` when:

- The class comes from a third-party library.
- You need special construction logic.
- You do not control the class source code.

Do not use `@Bean` for your normal services if you can simply annotate the class with `@Service` or `@Component`.

## 4. Component Stereotype Annotations

These annotations all register a class as a Spring bean. The difference is semantic: they describe the role of the class.

### `@Component`

Generic Spring bean annotation.

```java
import org.springframework.stereotype.Component;

@Component
public class InvoiceNumberGenerator {
    public String next() {
        return "INV-1001";
    }
}
```

Use it for helper classes that are not clearly controller, service, or repository.

### `@Controller`

Used for MVC controllers that return views such as Thymeleaf pages.

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class PageController {

    @GetMapping("/home")
    public String homePage() {
        return "home";
    }
}
```

Use `@Controller` when the method returns a view name.

### `@RestController`

Used for REST APIs. It combines `@Controller` and `@ResponseBody`.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HealthController {

    @GetMapping("/health")
    public String health() {
        return "OK";
    }
}
```

Use `@RestController` when methods return JSON, text, or response objects directly.

### `@Service`

Marks business logic classes.

```java
import org.springframework.stereotype.Service;

@Service
public class UserService {
    public String findDisplayName(long id) {
        return "Piyush";
    }
}
```

Controller should call service. Service should contain the real business rules.

### `@Repository`

Correct spelling: `@Repository`.

Marks persistence classes that talk to a database or external storage.

```java
import org.springframework.stereotype.Repository;

@Repository
public class UserRepository {
    public String findNameById(long id) {
        return "Piyush";
    }
}
```

Special benefit: Spring can translate database exceptions into Spring's data-access exception hierarchy.

## 5. Startup Runners

Startup runners execute code after the application context starts.

### `CommandLineRunner`

Correct spelling: `CommandLineRunner`.

Receives raw command-line arguments as `String... args`.

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class StartupCommandRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("Application started");
    }
}
```

### `ApplicationRunner`

Receives arguments as an `ApplicationArguments` object.

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class StartupApplicationRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        System.out.println("Option names: " + args.getOptionNames());
    }
}
```

### Difference

Use `CommandLineRunner` for simple startup code.

Use `ApplicationRunner` when you need parsed command-line options.

## 6. REST API Mapping

### `@RequestMapping`

Correct spelling: `@RequestMapping`.

Can be used at class level or method level.

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/users")
public class UserController {
}
```

Class-level mapping sets the base path.

### `@GetMapping`

Handles HTTP GET requests.

```java
@GetMapping("/{id}")
public UserResponse getUser(@PathVariable long id) {
    return new UserResponse(id, "Piyush");
}
```

Use for reading data.

### `@PostMapping`

Correct spelling: `@PostMapping`.

Handles HTTP POST requests.

```java
@PostMapping
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserResponse response = new UserResponse(1L, request.name());
    return ResponseEntity.status(201).body(response);
}
```

Use for creating data.

### `@PutMapping`

Handles HTTP PUT requests.

```java
@PutMapping("/{id}")
public UserResponse replaceUser(@PathVariable long id, @RequestBody UpdateUserRequest request) {
    return new UserResponse(id, request.name());
}
```

Use for replacing or updating an existing resource.

### `@PatchMapping`

This was missing from your list.

Handles HTTP PATCH requests.

```java
@PatchMapping("/{id}")
public UserResponse updatePartially(@PathVariable long id, @RequestBody PatchUserRequest request) {
    return new UserResponse(id, request.name());
}
```

Use for partial updates.

### `@DeleteMapping`

Handles HTTP DELETE requests.

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteUser(@PathVariable long id) {
}
```

Use for deleting data.

## 7. Request Data Annotations

### `@RequestParam`

Reads query parameters.

Example URL:

```text
GET /api/users?page=0&size=20
```

Code:

```java
@GetMapping
public List<UserResponse> listUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    return List.of();
}
```

Use for filters, pagination, sorting, and optional values.

### `@RequestBody`

Reads JSON request body.

```java
@PostMapping
public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
    return new UserResponse(1L, request.name());
}
```

Use for POST, PUT, and PATCH payloads.

### `@PathVariable`

The usual Spring MVC annotation is `@PathVariable`, not `@PathParam`.

Example URL:

```text
GET /api/users/10
```

Code:

```java
@GetMapping("/{id}")
public UserResponse getUser(@PathVariable long id) {
    return new UserResponse(id, "Piyush");
}
```

Use path variables for resource identifiers.

### `@PathParam`

`@PathParam` is from Jakarta/JAX-RS, not the normal Spring MVC style.

In Spring Boot REST controllers, prefer `@PathVariable`.

### `@RequestHeader`

Reads request headers.

```java
@GetMapping("/profile")
public UserResponse profile(@RequestHeader("X-User-Id") long userId) {
    return new UserResponse(userId, "Piyush");
}
```

Use for correlation IDs, auth-related metadata, client information, and custom headers.

### `@CookieValue`

This was missing from your list.

Reads cookies.

```java
@GetMapping("/theme")
public String theme(@CookieValue(defaultValue = "light") String theme) {
    return theme;
}
```

## 8. Response Handling

### `@ResponseStatus`

Sets the HTTP status code for a controller method or exception class.

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public UserResponse createUser(@RequestBody CreateUserRequest request) {
    return new UserResponse(1L, request.name());
}
```

Useful when the response status is fixed.

### `ResponseEntity`

`ResponseEntity` gives full control over status, headers, and body.

```java
@GetMapping("/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable long id) {
    if (id <= 0) {
        return ResponseEntity.badRequest().build();
    }

    UserResponse response = new UserResponse(id, "Piyush");
    return ResponseEntity.ok(response);
}
```

Use `ResponseEntity` when status or headers are dynamic.

Common examples:

```java
return ResponseEntity.ok(body);
return ResponseEntity.status(HttpStatus.CREATED).body(body);
return ResponseEntity.noContent().build();
return ResponseEntity.badRequest().body(error);
```

## 9. Exception Handling

### `@RestControllerAdvice`

Global exception handling for REST APIs.

```java
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {
}
```

It applies to controllers and returns response bodies directly.

### `@ExceptionHandler`

Handles a specific exception type.

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<ApiError> handleUserNotFound(UserNotFoundException ex) {
    ApiError error = new ApiError("USER_NOT_FOUND", ex.getMessage());
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
}
```

### Handling `MethodArgumentNotValidException`

Correct class name: `MethodArgumentNotValidException`.

It happens when `@Valid @RequestBody` fails validation.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.List;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<FieldValidationError> fieldErrors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> new FieldValidationError(
                        error.getField(),
                        error.getDefaultMessage()))
                .toList();

        List<String> globalErrors = ex.getBindingResult()
                .getGlobalErrors()
                .stream()
                .map(error -> error.getObjectName() + ": " + error.getDefaultMessage())
                .toList();

        ValidationErrorResponse response = new ValidationErrorResponse(fieldErrors, globalErrors);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
}
```

DTO records:

```java
public record ValidationErrorResponse(
        List<FieldValidationError> fieldErrors,
        List<String> globalErrors) {
}

public record FieldValidationError(
        String field,
        String message) {
}
```

Important methods:

- `ex.getBindingResult()`
- `getFieldErrors()`
- `getGlobalErrors()`

## 10. Controller Ordering

### `@Order`

`@Order` controls ordering when multiple beans of the same type exist.

```java
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(1)
public class FirstStartupTask {
}
```

Lower value means higher priority.

Common uses:

- Filters
- Interceptors
- Multiple runners
- Multiple exception handlers
- Ordered strategy classes

## 11. Configuration Properties

### `@Value`

Reads one property value.

`application.properties`:

```properties
app.support-email=support@example.com
```

Code:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class SupportConfig {

    @Value("${app.support-email}")
    private String supportEmail;
}
```

Use `@Value` for one or two simple values.

### `@ConfigurationProperties`

Binds a group of related properties to a class or record.

`application.properties`:

```properties
payment.provider=stripe
payment.timeout-seconds=5
payment.retry-count=3
```

Code:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "payment")
public record PaymentProperties(
        String provider,
        int timeoutSeconds,
        int retryCount) {
}
```

Enable scanning:

```java
import org.springframework.boot.context.properties.ConfigurationPropertiesScan;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@ConfigurationPropertiesScan
public class DemoApplication {
}
```

Use `@ConfigurationProperties` when properties belong together.

### `@Profile`

This was missing from your list.

Creates beans only for specific environments.

```java
@Service
@Profile("dev")
public class DevEmailService implements EmailService {
}
```

Activate profile:

```properties
spring.profiles.active=dev
```

## 12. Validation

Validation usually needs the dependency:

```text
spring-boot-starter-validation
```

### `@Valid`

Triggers validation on a request body or nested object.

```java
@PostMapping
public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
    return new UserResponse(1L, request.name());
}
```

### `@NotBlank`

String must not be `null`, empty, or only whitespace.

```java
public record CreateUserRequest(
        @NotBlank String name) {
}
```

Best for user-entered text.

### `@NotEmpty`

Value must not be `null` or empty.

Works on strings, collections, maps, and arrays.

```java
public record CreateUserRequest(
        @NotEmpty List<String> roles) {
}
```

Whitespace string passes `@NotEmpty`, so use `@NotBlank` for text fields.

### `@NotNull`

Value must not be `null`.

```java
public record CreateUserRequest(
        @NotNull Integer age) {
}
```

Use for numbers, booleans, enums, and object references.

### `@Size`

Checks length or collection size.

```java
public record CreateUserRequest(
        @Size(min = 2, max = 50) String name) {
}
```

### `@Min` And `@Max`

Correct spelling: `@Min`, `@Max`.

Checks numeric range.

```java
public record CreateUserRequest(
        @Min(18) @Max(100) Integer age) {
}
```

### `@Pattern`

Checks a string against a regular expression.

```java
public record CreateUserRequest(
        @Pattern(regexp = "^[A-Za-z0-9_]{3,20}$") String username) {
}
```

### Other Useful Validation Annotations

These were missing from your list but are common:

- `@Email`: validates email format.
- `@Positive`: number must be greater than zero.
- `@PositiveOrZero`: number must be zero or greater.
- `@Past`: date must be in the past.
- `@Future`: date must be in the future.
- `@AssertTrue`: boolean must be true.

Example:

```java
public record RegisterRequest(
        @Email String email,
        @Positive Integer quantity) {
}
```

## 13. CORS

### `@CrossOrigin`

Allows requests from another origin.

```java
@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "http://localhost:3000")
public class UserController {
}
```

Use it when a frontend app on another domain or port calls your API.

For production, avoid `origins = "*"` when credentials are involved.

## 14. Jackson JSON Annotations

Spring Boot commonly uses Jackson for JSON serialization and deserialization.

### `@JsonProperty`

Changes the JSON field name.

```java
import com.fasterxml.jackson.annotation.JsonProperty;

public record UserResponse(
        @JsonProperty("user_id") long userId,
        String name) {
}
```

JSON:

```json
{
  "user_id": 1,
  "name": "Piyush"
}
```

### `@JsonIgnore`

Excludes a field from JSON.

```java
public class UserResponse {
    private String name;

    @JsonIgnore
    private String password;
}
```

Use for sensitive or internal fields.

### `@JsonInclude`

Controls when fields are included.

```java
import com.fasterxml.jackson.annotation.JsonInclude;

@JsonInclude(JsonInclude.Include.NON_NULL)
public record UserResponse(
        Long id,
        String name,
        String optionalMessage) {
}
```

`null` fields are not included in the JSON output.

### `@JsonFormat`

Controls date, time, or number formatting.

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import java.time.LocalDate;

public record UserResponse(
        String name,
        @JsonFormat(pattern = "yyyy-MM-dd")
        LocalDate createdDate) {
}
```

### `@JsonIgnoreProperties`

Ignores unknown or selected properties.

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public record ExternalUserRequest(
        String name,
        String email) {
}
```

Useful when consuming JSON from another service that may send extra fields.

## 15. Scheduling

Scheduling runs methods automatically based on time.

### `@EnableScheduling`

Enable scheduling in the application.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### `@Scheduled`

Runs a method on a fixed schedule.

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ReportScheduler {

    @Scheduled(fixedRate = 2000)
    public void runEveryTwoSeconds() {
        System.out.println("Runs every 2 seconds");
    }

    @Scheduled(fixedDelay = 3000, initialDelay = 5000)
    public void runWithDelay() {
        System.out.println("Runs 3 seconds after previous execution finishes, first run after 5 seconds");
    }

    @Scheduled(cron = "0 * 19 * * ?")
    public void runEveryMinuteDuringSevenPm() {
        System.out.println("Runs every minute during the 19th hour");
    }
}
```

Important difference:

- `fixedRate`: starts based on a regular interval, measured from the previous start time.
- `fixedDelay`: waits after the previous execution finishes.
- `initialDelay`: waits before the first execution.
- `cron`: uses a cron expression.

Cron expression format used by Spring:

```text
second minute hour day-of-month month day-of-week
```

Example:

```java
@Scheduled(cron = "0 0 9 * * MON-FRI")
```

Meaning: run at 9:00 AM, Monday to Friday.

## 16. Caching

Caching stores method results so repeated calls can be faster.

### `@EnableCaching`

Enable caching in the application.

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class DemoApplication {
}
```

### `@Cacheable`

Checks cache first. If value exists, returns cached value. If not, runs method and stores result.

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public ProductResponse getProduct(long id) {
        return loadProductSlowly(id);
    }
}
```

Use for read operations.

### `@CachePut`

Always runs the method and updates the cache with the new result.

```java
@CachePut(value = "products", key = "#id")
public ProductResponse updateProduct(long id, UpdateProductRequest request) {
    return saveProduct(id, request);
}
```

Use when updating data and refreshing cache.

### `@CacheEvict`

Correct spelling: `@CacheEvict`.

Removes cache entries.

```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(long id) {
    deleteFromDatabase(id);
}
```

Evict all entries:

```java
@CacheEvict(value = "products", allEntries = true)
public void clearProductCache() {
}
```

### Common Cache Warning

Spring caching uses proxies. A cached method usually must be called from another bean. If a method inside the same class calls its own `@Cacheable` method, caching may not apply.

## 17. Transactions

### `@Transactional`

Runs a method inside a database transaction.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {

    @Transactional
    public void placeOrder(CreateOrderRequest request) {
        saveOrder(request);
        reduceStock(request);
        takePayment(request);
    }
}
```

If an unchecked exception happens, Spring rolls back the transaction.

Common attributes:

```java
@Transactional(readOnly = true)
public OrderResponse getOrder(long id) {
    return findOrder(id);
}

@Transactional(rollbackFor = Exception.class)
public void importOrders() throws Exception {
}
```

Best practices:

- Put `@Transactional` on service methods, not controller methods.
- Keep transactions short.
- Do not perform slow external API calls inside a database transaction unless required.
- Use `readOnly = true` for read-only queries.

## 18. Bean Lifecycle

Spring bean lifecycle means the journey of a bean from creation to destruction.

Simple flow:

```text
1. Spring finds bean definition
2. Creates object
3. Injects dependencies
4. Applies aware callbacks
5. Runs BeanPostProcessor before initialization
6. Runs initialization callbacks
7. Runs BeanPostProcessor after initialization
8. Bean is ready to use
9. On shutdown, destruction callbacks run
```

### Constructor

Runs when the object is created.

```java
@Service
public class PaymentService {

    public PaymentService() {
        System.out.println("Constructor");
    }
}
```

### `@PostConstruct`

Runs after dependency injection.

```java
import jakarta.annotation.PostConstruct;
import org.springframework.stereotype.Service;

@Service
public class PaymentService {

    @PostConstruct
    public void init() {
        System.out.println("Bean initialized");
    }
}
```

### `@PreDestroy`

Runs before bean destruction.

```java
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Service;

@Service
public class PaymentService {

    @PreDestroy
    public void cleanup() {
        System.out.println("Bean destroyed");
    }
}
```

### `InitializingBean` And `DisposableBean`

Less common than annotations, but useful to recognize.

```java
@Component
public class LifecycleBean implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("After properties set");
    }

    @Override
    public void destroy() {
        System.out.println("Destroy");
    }
}
```

### `initMethod` And `destroyMethod`

Can be used with `@Bean`.

```java
@Bean(initMethod = "start", destroyMethod = "stop")
public ExternalClient externalClient() {
    return new ExternalClient();
}
```

## 19. Categories

### Dependency Injection

Prefer constructor injection.

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

Avoid field injection:

```java
@Autowired
private UserRepository userRepository;
```

Constructor injection is easier to test and makes required dependencies clear.

### `@Autowired`

Usually not needed on a single constructor in modern Spring.

```java
@Service
public class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

### `@Qualifier`

Use when multiple beans implement the same interface.

```java
@Service
public class NotificationService {
    private final MessageSender sender;

    public NotificationService(@Qualifier("emailSender") MessageSender sender) {
        this.sender = sender;
    }
}
```

### `@Primary`

Marks one bean as the default choice.

```java
@Component
@Primary
public class EmailSender implements MessageSender {
}
```

### Java Records And Usage

Records are immutable data-carrier classes. They are useful when a class only needs to hold data and expose that data clearly.

Normal class:

```java
public class UserResponse {
    private final long id;
    private final String name;
    private final String email;

    public UserResponse(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
```

Record version:

```java
public record UserResponse(
        long id,
        String name,
        String email) {
}
```

Java automatically creates:

- Constructor
- Accessor methods like `id()`, `name()`, and `email()`
- `equals()`
- `hashCode()`
- `toString()`

Records are excellent for request and response DTOs.

```java
public record CreateUserRequest(
        @NotBlank String name,
        @Email String email) {
}

public record UserResponse(
        long id,
        String name,
        String email) {
}
```

Controller usage:

```java
@PostMapping
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserResponse response = userService.createUser(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

Important: record fields are accessed by method name, not `getName()`.

```java
String name = request.name();
String email = request.email();
```

#### Records With Validation

Validation annotations can be placed directly on record components.

```java
public record RegisterUserRequest(
        @NotBlank String name,
        @Email @NotBlank String email,
        @Min(18) Integer age) {
}
```

Nested validation:

```java
public record CreateOrderRequest(
        @NotNull Long customerId,
        @Valid @NotEmpty List<OrderItemRequest> items) {
}

public record OrderItemRequest(
        @NotNull Long productId,
        @Positive int quantity) {
}
```

Use `@Valid` on the nested record or list so Spring validates inside it.

#### Records With Compact Constructors

A compact constructor lets you add simple rules while keeping the record short.

```java
public record Money(
        BigDecimal amount,
        String currency) {

    public Money {
        if (amount == null || amount.signum() < 0) {
            throw new IllegalArgumentException("Amount must be zero or positive");
        }
        if (currency == null || currency.isBlank()) {
            throw new IllegalArgumentException("Currency is required");
        }
        currency = currency.toUpperCase();
    }
}
```

Use compact constructors for small invariants. For request validation, prefer Bean Validation annotations like `@NotBlank`, `@Min`, and `@Valid`.

#### Records For Configuration Properties

Records work well with `@ConfigurationProperties`.

```java
@ConfigurationProperties(prefix = "payment")
public record PaymentProperties(
        String provider,
        int timeoutSeconds,
        int retryCount) {
}
```

`application.properties`:

```properties
payment.provider=stripe
payment.timeout-seconds=5
payment.retry-count=3
```

#### Records With Jackson JSON

Jackson can serialize and deserialize records in Spring Boot.

```java
public record UserJsonResponse(
        @JsonProperty("user_id") long userId,
        @JsonInclude(JsonInclude.Include.NON_NULL) String displayName,
        @JsonFormat(pattern = "yyyy-MM-dd") LocalDate createdDate) {
}
```

#### Records For API Error Responses

Records are good for consistent API error bodies.

```java
public record ApiErrorResponse(
        String code,
        String message,
        Instant timestamp,
        List<FieldValidationError> fieldErrors) {
}

public record FieldValidationError(
        String field,
        String message) {
}
```

#### When To Use Records

Use records for:

- Request DTOs
- Response DTOs
- Configuration properties
- API error response models
- Simple projection results
- Value objects with small validation rules

Avoid records for:

- JPA entities
- Classes that need mutable state
- Classes with complex behavior
- Classes requiring inheritance
- Objects that frameworks must modify through setters

JPA entities should usually remain normal classes because JPA needs identity, lifecycle, proxies, and often a no-argument constructor.

### Pagination, Page-Based, Cursor-Based, And N+1 Problem

Pagination means returning a list in small parts instead of returning all rows at once.

Use pagination for:

- Large database tables
- Search result pages
- Admin list screens
- Infinite scroll APIs
- Mobile APIs where response size matters

There are two common styles:

- Page-based pagination
- Cursor-based pagination

#### Page-Based Pagination

Page-based pagination uses `page`, `size`, and usually `sort`.

Example request:

```text
GET /api/users?page=0&size=20&sort=createdAt,desc
```

Important rule: Spring Data page numbers start from `0`, not `1`.

Controller example:

```java
@GetMapping
public Page<UserResponse> listUsers(
        @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
        Pageable pageable) {
    return userService.findAll(pageable);
}
```

Useful annotations:

- `@PageableDefault`
- `@SortDefault`

Imports:

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.data.domain.Sort;
```

Repository example:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(UserStatus status, Pageable pageable);
}
```

Service example:

```java
@Transactional(readOnly = true)
public Page<UserResponse> findUsers(UserStatus status, Pageable pageable) {
    return userRepository.findByStatus(status, pageable)
            .map(user -> new UserResponse(user.getId(), user.getName(), user.getEmail()));
}
```

If you do not want to expose Spring's `Page` JSON directly, create your own response:

```java
public record PageResponse<T>(
        List<T> content,
        int page,
        int size,
        long totalElements,
        int totalPages,
        boolean first,
        boolean last) {

    public static <T> PageResponse<T> from(Page<T> page) {
        return new PageResponse<>(
                page.getContent(),
                page.getNumber(),
                page.getSize(),
                page.getTotalElements(),
                page.getTotalPages(),
                page.isFirst(),
                page.isLast());
    }
}
```

Page-based pagination is easy for UI screens where users jump to page 1, 2, 3, and so on.

Drawbacks:

- Deep pages can become slow because the database must skip many rows.
- New inserts or deletes can shift results between pages.
- Counting total rows can be expensive for large tables.

#### `Page` Vs `Slice`

`Page<T>` includes total count information.

`Slice<T>` only knows whether there is a next slice.

Use `Page<T>` when the UI needs total pages.

Use `Slice<T>` when you only need "load more".

```java
Slice<User> findByStatus(UserStatus status, Pageable pageable);
```

#### Cursor-Based Pagination

Cursor-based pagination uses the last seen value as the starting point for the next request.

Example first request:

```text
GET /api/users?size=20
```

Example next request:

```text
GET /api/users?size=20&afterId=105
```

Repository example:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    List<User> findTop20ByIdGreaterThanOrderByIdAsc(Long afterId);
}
```

More flexible query:

```java
@Query("""
        select u
        from User u
        where (:afterId is null or u.id > :afterId)
        order by u.id asc
        """)
List<User> findNextUsers(@Param("afterId") Long afterId, Pageable pageable);
```

Controller example:

```java
@GetMapping("/cursor")
public CursorResponse<UserResponse> listUsersByCursor(
        @RequestParam(required = false) Long afterId,
        @RequestParam(defaultValue = "20") int size) {
    return userService.findUsersByCursor(afterId, size);
}
```

Service example:

```java
@Transactional(readOnly = true)
public CursorResponse<UserResponse> findUsersByCursor(Long afterId, int size) {
    Pageable limit = PageRequest.of(0, size + 1);
    List<User> users = userRepository.findNextUsers(afterId, limit);

    boolean hasNext = users.size() > size;
    List<User> visibleUsers = hasNext ? users.subList(0, size) : users;

    Long nextCursor = visibleUsers.isEmpty()
            ? null
            : visibleUsers.get(visibleUsers.size() - 1).getId();

    List<UserResponse> content = visibleUsers.stream()
            .map(user -> new UserResponse(user.getId(), user.getName(), user.getEmail()))
            .toList();

    return new CursorResponse<>(content, nextCursor, hasNext);
}
```

Cursor response DTO:

```java
public record CursorResponse<T>(
        List<T> content,
        Long nextCursor,
        boolean hasNext) {
}
```

Cursor-based pagination is best for:

- Infinite scroll
- Large tables
- Fast "next page" APIs
- Feeds sorted by stable values like `id` or `createdAt`

Cursor rules:

- Always sort by a stable column.
- Prefer indexed columns like `id`, `createdAt`, or `(createdAt, id)`.
- Do not use unstable ordering like random order.
- Return `hasNext` and `nextCursor`.

#### Page-Based Vs Cursor-Based

| Topic | Page-Based | Cursor-Based |
|---|---|---|
| Request | `page=2&size=20` | `afterId=105&size=20` |
| Best for | Admin tables | Infinite scroll |
| Jump to page | Easy | Hard |
| Deep page performance | Can be slow | Usually fast |
| Total count | Easy with `Page<T>` | Usually not included |
| Data changing during browsing | Can duplicate or skip rows | More stable |

#### N+1 Query Problem

The N+1 problem happens when one query loads parent records, then one extra query runs for each parent record.

Example:

```text
1 query loads 20 users
20 extra queries load orders for each user
Total = 21 queries
```

Entity example:

```java
@Entity
public class User {
    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}
```

Problem code:

```java
@Transactional(readOnly = true)
public List<UserOrderResponse> findUsersWithOrders() {
    List<User> users = userRepository.findAll();

    return users.stream()
            .map(user -> new UserOrderResponse(
                    user.getId(),
                    user.getName(),
                    user.getOrders().size()))
            .toList();
}
```

`user.getOrders().size()` can trigger one extra query per user.

Fix 1: Use `@EntityGraph`.

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(attributePaths = "orders")
    List<User> findByStatus(UserStatus status);
}
```

Fix 2: Use a JPQL fetch join.

```java
@Query("""
        select distinct u
        from User u
        left join fetch u.orders
        where u.status = :status
        """)
List<User> findUsersWithOrders(@Param("status") UserStatus status);
```

Fix 3: Use DTO projection when you only need selected fields.

```java
@Query("""
        select new com.example.demo.user.UserOrderCountResponse(
            u.id,
            u.name,
            count(o.id)
        )
        from User u
        left join u.orders o
        group by u.id, u.name
        """)
List<UserOrderCountResponse> findUserOrderCounts();
```

DTO:

```java
public record UserOrderCountResponse(
        Long userId,
        String name,
        long orderCount) {
}
```

N+1 prevention checklist:

- Do not return JPA entities directly from controllers.
- Map entities to DTOs inside the service layer.
- Watch SQL logs when adding list endpoints.
- Use `@EntityGraph`, fetch joins, or DTO projections.
- Be careful with pagination plus `join fetch` on collections; prefer DTO projections or a two-step query for large paged lists.

### OpenAPI / Swagger

OpenAPI documents REST APIs in a standard JSON format. Swagger UI reads that OpenAPI document and gives you a browser page where you can inspect and test endpoints.

Common Spring Boot dependency:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.17</version>
</dependency>
```

For a Spring WebFlux app, use:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webflux-ui</artifactId>
    <version>2.8.17</version>
</dependency>
```

Default URLs after starting the app:

```text
http://localhost:8080/swagger-ui.html
http://localhost:8080/v3/api-docs
```

Optional `application.properties` configuration:

```properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operations-sorter=method
springdoc.swagger-ui.tags-sorter=alpha
```

If you customize `springdoc.api-docs.path=/api-docs`, the JSON documentation moves to `http://localhost:8080/api-docs`.

Basic OpenAPI configuration class:

```java
package com.example.demo.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI applicationOpenApi() {
        return new OpenAPI()
                .info(new Info()
                        .title("User API")
                        .description("Spring Boot API documentation")
                        .version("v1")
                        .contact(new Contact()
                                .name("API Support")
                                .email("support@example.com"))
                        .license(new License()
                                .name("Apache 2.0")
                                .url("https://www.apache.org/licenses/LICENSE-2.0")));
    }
}
```

Controller documentation example:

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/users")
@Tag(name = "Users", description = "User management APIs")
public class UserController {

    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID", description = "Returns one user for the given ID")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "User found"),
            @ApiResponse(responseCode = "404", description = "User not found")
    })
    public ResponseEntity<UserResponse> getUser(@PathVariable long id) {
        return ResponseEntity.ok(new UserResponse(id, "Piyush", "piyush@example.com"));
    }
}
```

DTO schema example:

```java
import io.swagger.v3.oas.annotations.media.Schema;

public record UserResponse(
        @Schema(example = "1") long id,
        @Schema(example = "Piyush") String name,
        @Schema(example = "piyush@example.com") String email) {
}
```

Useful OpenAPI annotations:

- `@Operation`
- `@ApiResponse`
- `@ApiResponses`
- `@Tag`
- `@Schema`

### Logging And Correlation IDs

For real APIs, use request IDs to trace logs.

Common header:

```text
X-Correlation-Id
```

Controller example:

```java
@GetMapping("/{id}")
public UserResponse getUser(
        @PathVariable long id,
        @RequestHeader(value = "X-Correlation-Id", required = false) String correlationId) {
    return userService.findUser(id);
}
```

## 21. One Complete Mini Example

```java
package com.example.demo.user;

import jakarta.validation.Valid;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<UserResponse> create(@Valid @RequestBody CreateUserRequest request) {
        UserResponse response = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getById(@PathVariable long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable long id) {
        userService.delete(id);
    }
}

record CreateUserRequest(
        @NotBlank String name,
        @Email String email) {
}

record UserResponse(
        long id,
        String name,
        String email) {
}
```

Service:

```java
package com.example.demo.user;

import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    @Transactional
    public UserResponse create(CreateUserRequest request) {
        return new UserResponse(1L, request.name(), request.email());
    }

    @Transactional(readOnly = true)
    @Cacheable(value = "users", key = "#id")
    public UserResponse findById(long id) {
        return new UserResponse(id, "Piyush", "piyush@example.com");
    }

    @Transactional
    @CacheEvict(value = "users", key = "#id")
    public void delete(long id) {
    }
}
```

Global exception handler:

```java
package com.example.demo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.List;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<FieldValidationError> fieldErrors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> new FieldValidationError(error.getField(), error.getDefaultMessage()))
                .toList();

        List<String> globalErrors = ex.getBindingResult()
                .getGlobalErrors()
                .stream()
                .map(error -> error.getObjectName() + ": " + error.getDefaultMessage())
                .toList();

        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(new ValidationErrorResponse(fieldErrors, globalErrors));
    }
}

record ValidationErrorResponse(
        List<FieldValidationError> fieldErrors,
        List<String> globalErrors) {
}

record FieldValidationError(
        String field,
        String message) {
}
```

## 22. Memory Tips

- `@SpringBootApplication` starts the app and scans components.
- `@Configuration` plus `@Bean` manually creates beans.
- `@Controller` returns views; `@RestController` returns data.
- `@Service` is business logic.
- `@Repository` is database/persistence logic.
- `@RequestParam` means query string.
- `@PathVariable` means path value.
- `@RequestBody` means JSON body.
- `@RequestHeader` means header value.
- `@Valid` activates validation.
- `@RestControllerAdvice` plus `@ExceptionHandler` handles errors globally.
- `@ConfigurationProperties` is better than many `@Value` fields.
- `@Scheduled` runs timed jobs.
- `@Cacheable` reads from cache; `@CachePut` refreshes cache; `@CacheEvict` removes cache.
- `@Transactional` keeps database operations atomic.
