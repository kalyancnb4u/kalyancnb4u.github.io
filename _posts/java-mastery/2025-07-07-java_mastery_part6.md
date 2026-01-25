---
title: "Java Mastery - Part 6: Microservices Architecture"
date: 2025-07-07 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, Distributed-Systems, Micro-services, Spring-cloud, Service-discovery, API-gateway, Circuit-breaker, Event-driven]
---

## Introduction

Welcome to Part 6 of the Complete Java Mastery series. In Parts 1-5, we covered Java fundamentals through Spring Framework. Now we explore **Microservices Architecture** - the dominant pattern for building scalable, distributed systems.

Microservices represent a shift from monolithic applications to small, independently deployable services. This architecture enables:

* **Independent deployment** of services
* **Technology diversity** (polyglot architecture)
* **Team autonomy** (Conway's Law alignment)
* **Horizontal scalability** at service level
* **Fault isolation** (failures don't cascade)

### What You'll Learn in Part 6

* **Microservices Fundamentals**: Architecture patterns, design principles
* **Service Discovery**: Eureka, Consul for dynamic service registration
* **API Gateway**: Spring Cloud Gateway, routing, filtering, security
* **Load Balancing**: Client-side load balancing with Spring Cloud LoadBalancer
* **Circuit Breaker**: Resilience4j for fault tolerance
* **Distributed Tracing**: Observability across services
* **Configuration Management**: Spring Cloud Config for centralized configuration
* **Messaging**: Event-driven architecture with Kafka, RabbitMQ
* **Data Management**: Database per service, SAGA pattern

This knowledge enables you to:
* Design scalable microservices architectures
* Implement service-to-service communication
* Build resilient distributed systems
* Manage configuration across services
* Implement event-driven patterns

---

## 6.1 Microservices Fundamentals

### Monolith vs Microservices

**Monolithic Architecture:**

```
┌─────────────────────────────────────┐
│         Monolithic Application       │
│                                      │
│  ┌──────────┐  ┌──────────┐        │
│  │   User   │  │  Order   │        │
│  │ Service  │  │ Service  │        │
│  └──────────┘  └──────────┘        │
│                                      │
│  ┌──────────┐  ┌──────────┐        │
│  │ Product  │  │ Payment  │        │
│  │ Service  │  │ Service  │        │
│  └──────────┘  └──────────┘        │
│                                      │
│  ┌─────────────────────────┐       │
│  │   Single Database       │       │
│  └─────────────────────────┘       │
└─────────────────────────────────────┘
```

**Problems:**
* Single point of failure
* Hard to scale specific components
* Technology lock-in
* Long deployment cycles
* Team coordination overhead

**Microservices Architecture:**

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   User   │    │  Order   │    │ Product  │    │ Payment  │
│ Service  │    │ Service  │    │ Service  │    │ Service  │
│          │    │          │    │          │    │          │
│ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │
│ │  DB  │ │    │ │  DB  │ │    │ │  DB  │ │    │ │  DB  │ │
│ └──────┘ │    │ └──────┘ │    │ └──────┘ │    │ └──────┘ │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     │               │               │               │
     └───────────────┴───────────────┴───────────────┘
                          │
                   ┌──────────────┐
                   │  API Gateway │
                   └──────────────┘
```

**Benefits:**
* Independent deployment and scaling
* Technology diversity
* Fault isolation
* Team autonomy
* Easier to understand individual services

**Challenges:**
* Distributed system complexity
* Network latency and failures
* Data consistency across services
* Testing complexity
* Operational overhead

### Microservices Design Principles

**1. Single Responsibility**

```java
// ❌ BAD: Service doing too much
@RestController
@RequestMapping("/api")
public class MonolithController {
    
    @PostMapping("/orders")
    public Order createOrder() {
        // Create order
        // Update inventory
        // Process payment
        // Send email
        // Update analytics
        return order;
    }
}

// ✅ GOOD: Each service has single responsibility
// Order Service
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    private final InventoryClient inventoryClient;
    private final PaymentClient paymentClient;
    private final NotificationClient notificationClient;
    
    @PostMapping
    public Order createOrder(@RequestBody CreateOrderRequest request) {
        // Orchestrate order creation across services
        Order order = orderService.create(request);
        inventoryClient.reserve(order.getItems());
        paymentClient.process(order.getId(), order.getTotal());
        notificationClient.sendOrderConfirmation(order.getId());
        return order;
    }
}
```

**2. Database per Service**

```java
// Each microservice has its own database
// User Service - PostgreSQL
@Entity
@Table(name = "users")
public class User {
    @Id
    private Long id;
    private String email;
    private String name;
}

// Order Service - MongoDB
@Document(collection = "orders")
public class Order {
    @Id
    private String id;
    private Long userId;  // Reference to user (no foreign key!)
    private List<OrderItem> items;
    private OrderStatus status;
}

// Product Service - MySQL
@Entity
@Table(name = "products")
public class Product {
    @Id
    private Long id;
    private String name;
    private BigDecimal price;
}
```

**Why Database per Service?**
* **Loose coupling**: Services don't share database schema
* **Technology choice**: Each service can use optimal database
* **Independent scaling**: Scale database with service needs
* **Fault isolation**: Database failure affects only one service

**3. API First**

```java
// Define API contract first (OpenAPI/Swagger)
@RestController
@RequestMapping("/api/products")
@Tag(name = "Product API", description = "Product management endpoints")
public class ProductController {
    
    @Operation(summary = "Get product by ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Product found"),
        @ApiResponse(responseCode = "404", description = "Product not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProduct(
        @Parameter(description = "Product ID") @PathVariable Long id
    ) {
        return ResponseEntity.ok(productService.findById(id));
    }
}
```

**4. Smart Endpoints, Dumb Pipes**

```java
// ✅ GOOD: Business logic in services, not infrastructure
@Service
public class OrderService {
    
    private final RestTemplate restTemplate;
    
    public Order createOrder(CreateOrderRequest request) {
        // Validate order
        validateOrder(request);
        
        // Create order
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setItems(request.getItems());
        order.setStatus(OrderStatus.PENDING);
        
        Order savedOrder = orderRepository.save(order);
        
        // Publish event (dumb pipe - just transport)
        orderEventPublisher.publish(new OrderCreatedEvent(savedOrder.getId()));
        
        return savedOrder;
    }
    
    private void validateOrder(CreateOrderRequest request) {
        // Business logic in service
        if (request.getItems().isEmpty()) {
            throw new InvalidOrderException("Order must have items");
        }
    }
}
```

**5. Decentralized Governance**

```
Each team decides:
- Programming language (Java, Go, Python, etc.)
- Database technology (PostgreSQL, MongoDB, etc.)
- Deployment strategy
- Testing approach

Common standards:
- API contracts (REST, gRPC)
- Communication protocols
- Authentication/authorization
- Observability (logs, metrics, traces)
```

### Domain-Driven Design (DDD) for Microservices

**Bounded Contexts:**

```java
// Order Context
public class Order {
    private OrderId id;
    private CustomerId customerId;  // Reference to Customer
    private List<OrderLine> orderLines;
    private Money totalAmount;
    
    public void place() {
        // Order domain logic
    }
}

// Customer Context (separate microservice)
public class Customer {
    private CustomerId id;
    private CustomerName name;
    private EmailAddress email;
    private List<Address> addresses;
    
    public void updateProfile(CustomerProfile profile) {
        // Customer domain logic
    }
}

// Each bounded context = potential microservice
```

**Aggregates:**

```java
// Order Aggregate Root
@Entity
@Table(name = "orders")
public class Order {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "order_id")
    private List<OrderLine> orderLines = new ArrayList<>();
    
    private OrderStatus status;
    
    // Aggregate root controls all modifications
    public void addOrderLine(Product product, int quantity) {
        OrderLine orderLine = new OrderLine(product, quantity);
        orderLines.add(orderLine);
    }
    
    public void removeOrderLine(Long orderLineId) {
        orderLines.removeIf(line -> line.getId().equals(orderLineId));
    }
    
    // Always modify through aggregate root
    public void place() {
        if (orderLines.isEmpty()) {
            throw new InvalidOrderException("Cannot place empty order");
        }
        this.status = OrderStatus.PLACED;
    }
}

// OrderLine - not directly accessible
@Entity
class OrderLine {
    @Id
    @GeneratedValue
    private Long id;
    
    private Long productId;
    private int quantity;
    private BigDecimal price;
    
    // Package-private - only Order can create
    OrderLine(Product product, int quantity) {
        this.productId = product.getId();
        this.quantity = quantity;
        this.price = product.getPrice();
    }
}
```

---

## 6.2 Service Discovery

Service discovery enables microservices to find and communicate with each other dynamically.

### Netflix Eureka

**Eureka Server (Service Registry):**

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false  # Don't register itself
    fetch-registry: false
  server:
    enable-self-preservation: false  # Disable in dev
```

**Eureka Client (Microservice):**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableDiscoveryClient  // or @EnableEurekaClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

```yaml
# application.yml
spring:
  application:
    name: order-service  # Service name for discovery

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 30
    lease-expiration-duration-in-seconds: 90
```

### Service-to-Service Communication

**Using RestTemplate with Service Discovery:**

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced  // Enable client-side load balancing
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    
    private final RestTemplate restTemplate;
    
    public OrderService(@LoadBalanced RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    public Product getProduct(Long productId) {
        // Use service name instead of URL
        String url = "http://product-service/api/products/" + productId;
        return restTemplate.getForObject(url, Product.class);
    }
    
    public void reserveInventory(List<OrderItem> items) {
        String url = "http://inventory-service/api/inventory/reserve";
        restTemplate.postForObject(url, items, Void.class);
    }
}
```

**Using WebClient (Reactive - Preferred):**

```java
@Configuration
public class WebClientConfig {
    
    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}

@Service
public class OrderService {
    
    private final WebClient.Builder webClientBuilder;
    
    public Product getProduct(Long productId) {
        return webClientBuilder.build()
            .get()
            .uri("http://product-service/api/products/{id}", productId)
            .retrieve()
            .bodyToMono(Product.class)
            .block();  // For synchronous call
    }
    
    // Async version
    public Mono<Product> getProductAsync(Long productId) {
        return webClientBuilder.build()
            .get()
            .uri("http://product-service/api/products/{id}", productId)
            .retrieve()
            .bodyToMono(Product.class);
    }
}
```

**Using Feign Client (Declarative):**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {
    // ...
}

// Declarative REST client
@FeignClient(name = "product-service")
public interface ProductClient {
    
    @GetMapping("/api/products/{id}")
    Product getProduct(@PathVariable Long id);
    
    @GetMapping("/api/products")
    List<Product> getAllProducts();
    
    @PostMapping("/api/products")
    Product createProduct(@RequestBody CreateProductRequest request);
}

// Usage
@Service
public class OrderService {
    
    private final ProductClient productClient;
    
    public Order createOrder(CreateOrderRequest request) {
        // Simple method call - Feign handles REST communication
        Product product = productClient.getProduct(request.getProductId());
        
        // Create order with product details
        Order order = new Order();
        order.setProductId(product.getId());
        order.setPrice(product.getPrice());
        
        return orderRepository.save(order);
    }
}
```

### Health Checks and Instance Status

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        // Check dependencies
        boolean databaseUp = checkDatabase();
        boolean cacheUp = checkCache();
        
        if (databaseUp && cacheUp) {
            return Health.up()
                .withDetail("database", "available")
                .withDetail("cache", "available")
                .build();
        } else {
            return Health.down()
                .withDetail("database", databaseUp ? "available" : "unavailable")
                .withDetail("cache", cacheUp ? "available" : "unavailable")
                .build();
        }
    }
    
    private boolean checkDatabase() {
        // Check database connectivity
        return true;
    }
    
    private boolean checkCache() {
        // Check cache connectivity
        return true;
    }
}
```

---

## 6.3 API Gateway

API Gateway provides single entry point for clients, handling routing, security, and cross-cutting concerns.

### Spring Cloud Gateway

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

**Configuration-Based Routing:**

```yaml
spring:
  cloud:
    gateway:
      routes:
        # Order Service routes
        - id: order-service
          uri: lb://order-service  # Load balanced
          predicates:
            - Path=/api/orders/**
          filters:
            - RewritePath=/api/orders/(?<segment>.*), /${segment}
        
        # Product Service routes
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
          filters:
            - RewritePath=/api/products/(?<segment>.*), /${segment}
        
        # User Service routes with authentication
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - RewritePath=/api/users/(?<segment>.*), /${segment}
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

**Programmatic Route Configuration:**

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // Order service
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .rewritePath("/api/orders/(?<segment>.*)", "/${segment}")
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .circuitBreaker(config -> config
                        .setName("orderServiceCircuitBreaker")
                        .setFallbackUri("forward:/fallback/orders"))
                )
                .uri("lb://order-service")
            )
            // Product service with retry
            .route("product-service", r -> r
                .path("/api/products/**")
                .filters(f -> f
                    .rewritePath("/api/products/(?<segment>.*)", "/${segment}")
                    .retry(retryConfig -> retryConfig
                        .setRetries(3)
                        .setStatuses(HttpStatus.SERVICE_UNAVAILABLE))
                )
                .uri("lb://product-service")
            )
            .build();
    }
}
```

### Custom Filters

**Global Filter (applies to all routes):**

```java
@Component
public class LoggingGlobalFilter implements GlobalFilter, Ordered {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingGlobalFilter.class);
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        
        logger.info("Request: {} {}", 
            request.getMethod(), 
            request.getURI()
        );
        
        long startTime = System.currentTimeMillis();
        
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            long duration = System.currentTimeMillis() - startTime;
            ServerHttpResponse response = exchange.getResponse();
            
            logger.info("Response: {} - Status: {} - Duration: {}ms",
                request.getURI(),
                response.getStatusCode(),
                duration
            );
        }));
    }
    
    @Override
    public int getOrder() {
        return -1;  // High priority
    }
}
```

**Custom Gateway Filter Factory:**

```java
@Component
public class AuthenticationGatewayFilterFactory 
        extends AbstractGatewayFilterFactory<AuthenticationGatewayFilterFactory.Config> {
    
    private final JwtUtil jwtUtil;
    
    public AuthenticationGatewayFilterFactory(JwtUtil jwtUtil) {
        super(Config.class);
        this.jwtUtil = jwtUtil;
    }
    
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            
            // Extract token
            String token = extractToken(request);
            
            if (token == null || !jwtUtil.validateToken(token)) {
                exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                return exchange.getResponse().setComplete();
            }
            
            // Add user info to request headers
            String username = jwtUtil.extractUsername(token);
            ServerHttpRequest modifiedRequest = request.mutate()
                .header("X-User-Id", username)
                .build();
            
            return chain.filter(exchange.mutate().request(modifiedRequest).build());
        };
    }
    
    private String extractToken(ServerHttpRequest request) {
        List<String> headers = request.getHeaders().get("Authorization");
        if (headers != null && !headers.isEmpty()) {
            String authHeader = headers.get(0);
            if (authHeader.startsWith("Bearer ")) {
                return authHeader.substring(7);
            }
        }
        return null;
    }
    
    public static class Config {
        // Configuration properties
    }
}
```

### Rate Limiting

```java
@Configuration
public class RateLimiterConfig {
    
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> {
            // Rate limit per user
            String userId = exchange.getRequest().getHeaders().getFirst("X-User-Id");
            return Mono.just(userId != null ? userId : "anonymous");
        };
    }
    
    @Bean
    public KeyResolver apiKeyResolver() {
        return exchange -> {
            // Rate limit per API key
            String apiKey = exchange.getRequest().getHeaders().getFirst("X-API-Key");
            return Mono.just(apiKey != null ? apiKey : "no-api-key");
        };
    }
}
```

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: api-route
          uri: lb://api-service
          predicates:
            - Path=/api/**
          filters:
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@userKeyResolver}"
                redis-rate-limiter.replenishRate: 10  # 10 requests per second
                redis-rate-limiter.burstCapacity: 20  # Max 20 requests burst
```

### Fallback

```java
@RestController
@RequestMapping("/fallback")
public class FallbackController {
    
    @GetMapping("/orders")
    public ResponseEntity<ErrorResponse> orderServiceFallback() {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
            .body(new ErrorResponse(
                "ORDER_SERVICE_UNAVAILABLE",
                "Order service is currently unavailable. Please try again later."
            ));
    }
    
    @GetMapping("/products")
    public ResponseEntity<ErrorResponse> productServiceFallback() {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
            .body(new ErrorResponse(
                "PRODUCT_SERVICE_UNAVAILABLE",
                "Product service is currently unavailable. Please try again later."
            ));
    }
}
```

---

## 6.4 Circuit Breaker Pattern - Resilience4j

Circuit breakers prevent cascading failures by detecting failures and preventing calls to failing services.

### Circuit Breaker States

```
CLOSED → Normal operation
  │
  │ Failure threshold exceeded
  ▼
OPEN → Reject all calls immediately
  │
  │ Wait duration elapsed
  ▼
HALF_OPEN → Allow test calls
  │         │
  │ Success │ Failure
  ▼         ▼
CLOSED    OPEN
```

### Resilience4j Setup

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

**Configuration:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      productService:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        automatic-transition-from-open-to-half-open-enabled: true
        wait-duration-in-open-state: 10s
        failure-rate-threshold: 50
        slow-call-rate-threshold: 100
        slow-call-duration-threshold: 2s
        record-exceptions:
          - java.net.ConnectException
          - java.util.concurrent.TimeoutException
        ignore-exceptions:
          - com.example.exception.BusinessException
      
      orderService:
        sliding-window-size: 20
        minimum-number-of-calls: 10
        wait-duration-in-open-state: 30s
        failure-rate-threshold: 60
  
  retry:
    instances:
      productService:
        max-attempts: 3
        wait-duration: 1s
        enable-exponential-backoff: true
        exponential-backoff-multiplier: 2
        retry-exceptions:
          - java.net.ConnectException
  
  bulkhead:
    instances:
      productService:
        max-concurrent-calls: 10
        max-wait-duration: 500ms
  
  timelimiter:
    instances:
      productService:
        timeout-duration: 2s
```

### Using Circuit Breaker

**Declarative Approach:**

```java
@Service
public class ProductService {
    
    private final ProductClient productClient;
    
    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    @Retry(name = "productService")
    @Bulkhead(name = "productService")
    @TimeLimiter(name = "productService")
    public Product getProduct(Long id) {
        return productClient.getProduct(id);
    }
    
    // Fallback method - same signature + Throwable parameter
    private Product getProductFallback(Long id, Throwable throwable) {
        log.error("Product service call failed for id: {}. Error: {}", 
            id, throwable.getMessage());
        
        // Return cached data or default
        return productCacheService.getCachedProduct(id)
            .orElse(Product.builder()
                .id(id)
                .name("Product temporarily unavailable")
                .available(false)
                .build());
    }
    
    @CircuitBreaker(name = "orderService", fallbackMethod = "createOrderFallback")
    public Order createOrder(CreateOrderRequest request) {
        return orderClient.createOrder(request);
    }
    
    private Order createOrderFallback(CreateOrderRequest request, Throwable throwable) {
        // Queue order for later processing
        orderQueueService.queueOrder(request);
        
        return Order.builder()
            .status(OrderStatus.QUEUED)
            .message("Order queued for processing")
            .build();
    }
}
```

**Programmatic Approach:**

```java
@Service
public class OrderService {
    
    private final CircuitBreakerRegistry circuitBreakerRegistry;
    private final RetryRegistry retryRegistry;
    private final RestTemplate restTemplate;
    
    public Product getProductWithCircuitBreaker(Long id) {
        CircuitBreaker circuitBreaker = circuitBreakerRegistry
            .circuitBreaker("productService");
        
        Retry retry = retryRegistry.retry("productService");
        
        Supplier<Product> supplier = () -> 
            restTemplate.getForObject(
                "http://product-service/api/products/" + id, 
                Product.class
            );
        
        // Decorate supplier with circuit breaker and retry
        Supplier<Product> decoratedSupplier = Decorators
            .ofSupplier(supplier)
            .withCircuitBreaker(circuitBreaker)
            .withRetry(retry)
            .withFallback(Arrays.asList(Exception.class), 
                throwable -> getProductFallback(id))
            .decorate();
        
        return decoratedSupplier.get();
    }
    
    private Product getProductFallback(Long id) {
        return Product.builder()
            .id(id)
            .name("Fallback Product")
            .build();
    }
}
```

### Monitoring Circuit Breaker

```java
@Component
public class CircuitBreakerEventListener {
    
    private static final Logger logger = LoggerFactory.getLogger(CircuitBreakerEventListener.class);
    
    @PostConstruct
    public void registerEventListeners(CircuitBreakerRegistry registry) {
        registry.circuitBreaker("productService").getEventPublisher()
            .onSuccess(event -> logger.info("Circuit breaker success: {}", event))
            .onError(event -> logger.error("Circuit breaker error: {}", event))
            .onStateTransition(event -> logger.warn("Circuit breaker state transition: {} -> {}", 
                event.getStateTransition().getFromState(),
                event.getStateTransition().getToState()))
            .onFailureRateExceeded(event -> logger.error("Failure rate exceeded: {}%", 
                event.getFailureRate()))
            .onSlowCallRateExceeded(event -> logger.warn("Slow call rate exceeded: {}%",
                event.getSlowCallRate()));
    }
}
```

**Actuator Endpoint:**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,circuitbreakers,circuitbreakerevents
  
  health:
    circuitbreakers:
      enabled: true
```

```bash
# Check circuit breaker health
GET /actuator/health/circuitBreakers

# Get circuit breaker events
GET /actuator/circuitbreakerevents
GET /actuator/circuitbreakerevents/productService
```

---

## 6.5 Distributed Configuration Management

Centralized configuration management for all microservices.

### Spring Cloud Config Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

**Config Server:**

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/config-repo
          default-label: main
          search-paths: '{application}'
          clone-on-start: true
        # Or use native (filesystem)
        native:
          search-locations: classpath:/config,file:/config
  profiles:
    active: native  # or git
```

**Git Repository Structure:**

```
config-repo/
├── application.yml              # Common config for all services
├── application-dev.yml          # Common dev config
├── application-prod.yml         # Common prod config
├── order-service.yml            # Order service specific
├── order-service-dev.yml
├── order-service-prod.yml
├── product-service.yml
├── product-service-dev.yml
└── product-service-prod.yml
```

**Example Configuration Files:**

```yaml
# application.yml (common)
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

management:
  endpoints:
    web:
      exposure:
        include: "*"

logging:
  level:
    root: INFO
```

```yaml
# order-service.yml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/orders
    username: orderuser
    password: '{cipher}AQBxxxxxxxx'  # Encrypted

app:
  order:
    max-items: 100
    auto-cancel-minutes: 30
```

### Config Client

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

```yaml
# bootstrap.yml (loaded before application.yml)
spring:
  application:
    name: order-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
      retry:
        max-attempts: 5
        initial-interval: 1000
        multiplier: 1.5
  profiles:
    active: dev
```

### Refresh Configuration at Runtime

```java
@RestController
@RefreshScope  // Beans recreated when configuration changes
public class OrderController {
    
    @Value("${app.order.max-items}")
    private int maxItems;
    
    @GetMapping("/config")
    public Map<String, Object> getConfig() {
        return Map.of("maxItems", maxItems);
    }
}
```

**Trigger Refresh:**

```bash
# Refresh specific service
POST /actuator/refresh

# Or use Spring Cloud Bus to refresh all instances
POST /actuator/bus-refresh
```

### Encryption/Decryption

**Encrypt Sensitive Data:**

```bash
# Encrypt value
curl http://localhost:8888/encrypt -d "mysecretpassword"
# Returns: AQBxxxxxxxxxxxxxxx

# Decrypt value
curl http://localhost:8888/decrypt -d "AQBxxxxxxxxxxxxxxx"
# Returns: mysecretpassword
```

**Use in Configuration:**

```yaml
spring:
  datasource:
    password: '{cipher}AQBxxxxxxxxxxxxxxx'
```

---

## 6.6 Distributed Tracing

Distributed tracing tracks requests across multiple services for observability.

### Spring Cloud Sleuth + Zipkin

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

**Configuration:**

```yaml
spring:
  sleuth:
    sampler:
      probability: 1.0  # Sample 100% of requests (reduce in production)
  zipkin:
    base-url: http://localhost:9411
    sender:
      type: web  # or kafka, rabbit

logging:
  level:
    org.springframework.cloud.sleuth: DEBUG
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

**How It Works:**

```
Request arrives at API Gateway
  │
  ├─ Trace ID: abc123
  └─ Span ID: 001
      │
      ├─> Order Service (same Trace ID: abc123, new Span ID: 002)
      │   │
      │   ├─> Product Service (Trace ID: abc123, Span ID: 003)
      │   └─> Inventory Service (Trace ID: abc123, Span ID: 004)
      │
      └─> Payment Service (Trace ID: abc123, Span ID: 005)

All spans linked by same Trace ID for end-to-end visibility
```

**Automatic Instrumentation:**

```java
// Sleuth automatically adds trace/span IDs to:
// - HTTP headers
// - Log statements
// - Message queues
// - Database calls

@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(CreateOrderRequest request) {
        // Logs automatically include [traceId, spanId]
        logger.info("Creating order for user: {}", request.getUserId());
        
        // HTTP call automatically propagates trace context
        Product product = productClient.getProduct(request.getProductId());
        
        // New span automatically created
        Order order = saveOrder(request, product);
        
        logger.info("Order created: {}", order.getId());
        return order;
    }
}
```

**Custom Spans:**

```java
@Service
public class OrderService {
    
    private final Tracer tracer;
    
    public Order processOrder(Order order) {
        // Create custom span
        Span span = tracer.nextSpan().name("process-order").start();
        
        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            // Add custom tags
            span.tag("order.id", order.getId().toString());
            span.tag("order.total", order.getTotal().toString());
            
            // Add event
            span.event("order.validated");
            
            // Business logic
            validateOrder(order);
            calculateTotal(order);
            
            span.event("order.processed");
            
            return order;
        } catch (Exception e) {
            span.error(e);
            throw e;
        } finally {
            span.end();
        }
    }
}
```

### Running Zipkin

```bash
# Using Docker
docker run -d -p 9411:9411 openzipkin/zipkin

# Access UI
http://localhost:9411
```

**Zipkin UI Features:**
* Search traces by service, operation, tags
* View trace timeline (waterfall diagram)
* Identify slow services
* See error traces
* Analyze service dependencies

### Micrometer + Prometheus + Grafana

**Complete Observability Stack:**

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
management:
  metrics:
    export:
      prometheus:
        enabled: true
  endpoints:
    web:
      exposure:
        include: prometheus,health,metrics
```

**Custom Metrics:**

```java
@Service
public class OrderService {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;
    
    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        this.orderCounter = Counter.builder("orders.created")
            .description("Total orders created")
            .tag("service", "order-service")
            .register(meterRegistry);
        
        this.orderProcessingTimer = Timer.builder("order.processing.duration")
            .description("Order processing duration")
            .tag("service", "order-service")
            .register(meterRegistry);
    }
    
    public Order createOrder(CreateOrderRequest request) {
        return orderProcessingTimer.record(() -> {
            Order order = processOrder(request);
            orderCounter.increment();
            
            // Add dynamic tags
            meterRegistry.counter("orders.by.status", 
                "status", order.getStatus().toString()).increment();
            
            return order;
        });
    }
}
```

**Prometheus Configuration (prometheus.yml):**

```yaml
scrape_configs:
  - job_name: 'spring-boot'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8081', 'localhost:8082', 'localhost:8083']
```

**Grafana Dashboards:**
* JVM metrics (heap, threads, GC)
* HTTP request rates and latencies
* Custom business metrics
* Service health and availability

---

## 6.7 Event-Driven Architecture

Event-driven architecture enables loose coupling between services through asynchronous messaging.

### Apache Kafka

**Producer (Publishing Events):**

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
    consumer:
      group-id: order-service-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
      auto-offset-reset: earliest
```

**Domain Events:**

```java
// Event
public class OrderCreatedEvent {
    private Long orderId;
    private Long userId;
    private BigDecimal total;
    private LocalDateTime createdAt;
    private List<OrderItem> items;
    
    // Constructors, getters, setters
}

public class OrderCancelledEvent {
    private Long orderId;
    private String reason;
    private LocalDateTime cancelledAt;
}

public class PaymentProcessedEvent {
    private Long orderId;
    private Long paymentId;
    private BigDecimal amount;
    private PaymentStatus status;
}
```

**Event Publisher:**

```java
@Service
public class OrderService {
    
    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final OrderRepository orderRepository;
    
    public Order createOrder(CreateOrderRequest request) {
        // Create order
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setItems(request.getItems());
        order.setTotal(calculateTotal(request.getItems()));
        order.setStatus(OrderStatus.CREATED);
        
        order = orderRepository.save(order);
        
        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getTotal(),
            order.getCreatedAt(),
            order.getItems()
        );
        
        kafkaTemplate.send("order-events", order.getId().toString(), event);
        
        logger.info("Published OrderCreatedEvent for order: {}", order.getId());
        
        return order;
    }
    
    public void cancelOrder(Long orderId, String reason) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
        
        // Publish cancellation event
        OrderCancelledEvent event = new OrderCancelledEvent(
            orderId,
            reason,
            LocalDateTime.now()
        );
        
        kafkaTemplate.send("order-events", orderId.toString(), event);
    }
}
```

**Event Consumer:**

```java
@Service
public class InventoryEventListener {
    
    private final InventoryService inventoryService;
    
    @KafkaListener(topics = "order-events", groupId = "inventory-service-group")
    public void handleOrderEvent(ConsumerRecord<String, String> record) {
        String eventType = record.headers().lastHeader("eventType").value().toString();
        
        switch (eventType) {
            case "OrderCreatedEvent":
                OrderCreatedEvent createdEvent = parseEvent(record.value(), OrderCreatedEvent.class);
                handleOrderCreated(createdEvent);
                break;
            case "OrderCancelledEvent":
                OrderCancelledEvent cancelledEvent = parseEvent(record.value(), OrderCancelledEvent.class);
                handleOrderCancelled(cancelledEvent);
                break;
        }
    }
    
    private void handleOrderCreated(OrderCreatedEvent event) {
        logger.info("Handling OrderCreatedEvent: {}", event.getOrderId());
        
        try {
            // Reserve inventory
            inventoryService.reserve(event.getItems());
            logger.info("Inventory reserved for order: {}", event.getOrderId());
        } catch (InsufficientInventoryException e) {
            logger.error("Failed to reserve inventory for order: {}", event.getOrderId());
            // Publish failure event for compensation
            publishInventoryReservationFailed(event.getOrderId());
        }
    }
    
    private void handleOrderCancelled(OrderCancelledEvent event) {
        logger.info("Handling OrderCancelledEvent: {}", event.getOrderId());
        
        // Release reserved inventory
        inventoryService.release(event.getOrderId());
    }
}
```

**Typed Consumer with @Payload:**

```java
@Component
public class PaymentEventListener {
    
    @KafkaListener(topics = "order-events", 
                   groupId = "payment-service-group",
                   containerFactory = "orderEventListenerFactory")
    public void handleOrderCreated(@Payload OrderCreatedEvent event,
                                   @Header(KafkaHeaders.RECEIVED_KEY) String key,
                                   @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
                                   @Header(KafkaHeaders.OFFSET) long offset) {
        
        logger.info("Received OrderCreatedEvent: orderId={}, partition={}, offset={}", 
            event.getOrderId(), partition, offset);
        
        try {
            // Process payment
            Payment payment = paymentService.processPayment(
                event.getOrderId(),
                event.getTotal()
            );
            
            // Publish success event
            PaymentProcessedEvent paymentEvent = new PaymentProcessedEvent(
                event.getOrderId(),
                payment.getId(),
                payment.getAmount(),
                PaymentStatus.COMPLETED
            );
            
            kafkaTemplate.send("payment-events", key, paymentEvent);
            
        } catch (PaymentFailedException e) {
            logger.error("Payment failed for order: {}", event.getOrderId(), e);
            
            // Publish failure event
            PaymentProcessedEvent paymentEvent = new PaymentProcessedEvent(
                event.getOrderId(),
                null,
                event.getTotal(),
                PaymentStatus.FAILED
            );
            
            kafkaTemplate.send("payment-events", key, paymentEvent);
        }
    }
}
```

### RabbitMQ (Alternative to Kafka)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

**Configuration:**

```java
@Configuration
public class RabbitMQConfig {
    
    public static final String ORDER_EXCHANGE = "order-exchange";
    public static final String ORDER_QUEUE = "order-queue";
    public static final String ORDER_ROUTING_KEY = "order.created";
    
    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange(ORDER_EXCHANGE);
    }
    
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable(ORDER_QUEUE)
            .withArgument("x-dead-letter-exchange", "dlx-exchange")
            .build();
    }
    
    @Bean
    public Binding orderBinding(Queue orderQueue, TopicExchange orderExchange) {
        return BindingBuilder
            .bind(orderQueue)
            .to(orderExchange)
            .with(ORDER_ROUTING_KEY);
    }
    
    @Bean
    public Jackson2JsonMessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

**Publisher:**

```java
@Service
public class OrderEventPublisher {
    
    private final RabbitTemplate rabbitTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.ORDER_EXCHANGE,
            RabbitMQConfig.ORDER_ROUTING_KEY,
            event
        );
        
        logger.info("Published OrderCreatedEvent: {}", order.getId());
    }
}
```

**Consumer:**

```java
@Component
public class OrderEventListener {
    
    @RabbitListener(queues = RabbitMQConfig.ORDER_QUEUE)
    public void handleOrderCreated(OrderCreatedEvent event) {
        logger.info("Received OrderCreatedEvent: {}", event.getOrderId());
        
        // Process event
        inventoryService.reserve(event.getItems());
    }
}
```

### Event Sourcing

**Event Store:**

```java
@Entity
@Table(name = "order_events")
public class OrderEventEntity {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false)
    private Long aggregateId;  // Order ID
    
    @Column(nullable = false)
    private String eventType;
    
    @Column(nullable = false, columnDefinition = "TEXT")
    private String eventData;  // JSON
    
    @Column(nullable = false)
    private LocalDateTime occurredAt;
    
    @Column(nullable = false)
    private Long version;  // For optimistic locking
}

@Repository
public interface OrderEventRepository extends JpaRepository<OrderEventEntity, Long> {
    List<OrderEventEntity> findByAggregateIdOrderByVersionAsc(Long aggregateId);
}
```

**Event-Sourced Aggregate:**

```java
@Service
public class OrderEventStore {
    
    private final OrderEventRepository eventRepository;
    private final ObjectMapper objectMapper;
    
    public void saveEvent(Long orderId, Object event) {
        OrderEventEntity entity = new OrderEventEntity();
        entity.setAggregateId(orderId);
        entity.setEventType(event.getClass().getSimpleName());
        entity.setEventData(objectMapper.writeValueAsString(event));
        entity.setOccurredAt(LocalDateTime.now());
        
        // Get current version
        long currentVersion = eventRepository.findByAggregateIdOrderByVersionAsc(orderId)
            .stream()
            .mapToLong(OrderEventEntity::getVersion)
            .max()
            .orElse(0L);
        
        entity.setVersion(currentVersion + 1);
        
        eventRepository.save(entity);
    }
    
    public Order rebuildOrder(Long orderId) {
        List<OrderEventEntity> events = eventRepository
            .findByAggregateIdOrderByVersionAsc(orderId);
        
        Order order = new Order();
        
        for (OrderEventEntity eventEntity : events) {
            Object event = deserializeEvent(eventEntity);
            applyEvent(order, event);
        }
        
        return order;
    }
    
    private void applyEvent(Order order, Object event) {
        if (event instanceof OrderCreatedEvent e) {
            order.setId(e.getOrderId());
            order.setUserId(e.getUserId());
            order.setItems(e.getItems());
            order.setTotal(e.getTotal());
            order.setStatus(OrderStatus.CREATED);
        } else if (event instanceof OrderShippedEvent e) {
            order.setStatus(OrderStatus.SHIPPED);
            order.setShippedAt(e.getShippedAt());
        } else if (event instanceof OrderCancelledEvent e) {
            order.setStatus(OrderStatus.CANCELLED);
        }
    }
}
```

---

## 6.8 Data Management in Microservices

Managing data in distributed systems requires different patterns than monoliths.

### Database per Service Pattern

**Challenges:**
* No direct database joins across services
* Distributed transactions
* Data duplication
* Eventual consistency

**Solutions:**

**1. API Composition Pattern:**

```java
@Service
public class OrderDetailsService {
    
    private final OrderClient orderClient;
    private final ProductClient productClient;
    private final UserClient userClient;
    
    public OrderDetails getOrderDetails(Long orderId) {
        // Fetch from multiple services
        Order order = orderClient.getOrder(orderId);
        User user = userClient.getUser(order.getUserId());
        List<Product> products = order.getItems().stream()
            .map(item -> productClient.getProduct(item.getProductId()))
            .collect(Collectors.toList());
        
        // Compose response
        return OrderDetails.builder()
            .order(order)
            .user(user)
            .products(products)
            .build();
    }
}
```

**2. CQRS (Command Query Responsibility Segregation):**

```java
// Command side - Write model
@Service
public class OrderCommandService {
    
    private final OrderRepository orderRepository;
    private final KafkaTemplate kafkaTemplate;
    
    @Transactional
    public Order createOrder(CreateOrderCommand command) {
        Order order = new Order();
        order.setUserId(command.getUserId());
        order.setItems(command.getItems());
        
        order = orderRepository.save(order);
        
        // Publish event
        kafkaTemplate.send("order-events", new OrderCreatedEvent(order));
        
        return order;
    }
}

// Query side - Read model (denormalized)
@Document(collection = "order_views")
public class OrderView {
    @Id
    private String id;
    private Long orderId;
    private String userName;
    private String userEmail;
    private List<ProductView> products;
    private BigDecimal total;
    private OrderStatus status;
}

@Service
public class OrderViewService {
    
    private final OrderViewRepository viewRepository;
    
    @KafkaListener(topics = "order-events")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Build denormalized view
        OrderView view = new OrderView();
        view.setOrderId(event.getOrderId());
        
        // Fetch additional data
        User user = userClient.getUser(event.getUserId());
        view.setUserName(user.getName());
        view.setUserEmail(user.getEmail());
        
        List<ProductView> products = event.getItems().stream()
            .map(this::buildProductView)
            .collect(Collectors.toList());
        view.setProducts(products);
        
        viewRepository.save(view);
    }
    
    public OrderView getOrderView(Long orderId) {
        // Single query - fast read
        return viewRepository.findByOrderId(orderId);
    }
}
```

### SAGA Pattern

**Choreography-Based SAGA:**

```java
// Order Service
@Service
public class OrderSagaService {
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // Step 1: Create order (local transaction)
        Order order = new Order();
        order.setStatus(OrderStatus.PENDING);
        order = orderRepository.save(order);
        
        // Publish event to trigger next step
        kafkaTemplate.send("order-events", new OrderCreatedEvent(order));
        
        return order;
    }
    
    @KafkaListener(topics = "payment-events")
    public void handlePaymentEvent(PaymentProcessedEvent event) {
        if (event.getStatus() == PaymentStatus.COMPLETED) {
            // Payment succeeded - update order
            Order order = orderRepository.findById(event.getOrderId()).orElseThrow();
            order.setStatus(OrderStatus.PAID);
            orderRepository.save(order);
            
            // Trigger next step
            kafkaTemplate.send("order-events", new OrderPaidEvent(order));
        } else {
            // Payment failed - compensate
            cancelOrder(event.getOrderId(), "Payment failed");
        }
    }
    
    @KafkaListener(topics = "inventory-events")
    public void handleInventoryEvent(InventoryReservedEvent event) {
        if (!event.isSuccess()) {
            // Inventory reservation failed - compensate
            cancelOrder(event.getOrderId(), "Insufficient inventory");
            
            // Trigger payment refund
            kafkaTemplate.send("payment-events", new RefundPaymentEvent(event.getOrderId()));
        }
    }
}

// Inventory Service
@Service
public class InventorySagaService {
    
    @KafkaListener(topics = "order-events")
    public void handleOrderPaid(OrderPaidEvent event) {
        try {
            // Step 2: Reserve inventory (local transaction)
            inventoryService.reserve(event.getItems());
            
            // Publish success
            kafkaTemplate.send("inventory-events", 
                new InventoryReservedEvent(event.getOrderId(), true));
                
        } catch (InsufficientInventoryException e) {
            // Publish failure
            kafkaTemplate.send("inventory-events",
                new InventoryReservedEvent(event.getOrderId(), false));
        }
    }
    
    @KafkaListener(topics = "order-events")
    public void handleOrderCancelled(OrderCancelledEvent event) {
        // Compensating transaction
        inventoryService.release(event.getOrderId());
    }
}
```

**Orchestration-Based SAGA:**

```java
@Service
public class OrderSagaOrchestrator {
    
    private final OrderRepository orderRepository;
    private final PaymentClient paymentClient;
    private final InventoryClient inventoryClient;
    private final ShippingClient shippingClient;
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = null;
        Payment payment = null;
        InventoryReservation reservation = null;
        
        try {
            // Step 1: Create order
            order = orderService.create(request);
            
            // Step 2: Process payment
            payment = paymentClient.process(order.getId(), order.getTotal());
            
            // Step 3: Reserve inventory
            reservation = inventoryClient.reserve(order.getItems());
            
            // Step 4: Create shipment
            shippingClient.createShipment(order.getId());
            
            // All steps successful
            order.setStatus(OrderStatus.CONFIRMED);
            return orderRepository.save(order);
            
        } catch (Exception e) {
            // Compensate in reverse order
            logger.error("Order creation failed, compensating...", e);
            
            if (reservation != null) {
                inventoryClient.cancelReservation(reservation.getId());
            }
            
            if (payment != null) {
                paymentClient.refund(payment.getId());
            }
            
            if (order != null) {
                order.setStatus(OrderStatus.FAILED);
                orderRepository.save(order);
            }
            
            throw new OrderCreationFailedException("Failed to create order", e);
        }
    }
}
```

### Outbox Pattern

Ensures reliable event publishing:

```java
@Entity
@Table(name = "outbox_events")
public class OutboxEvent {
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false)
    private String aggregateType;  // "Order"
    
    @Column(nullable = false)
    private Long aggregateId;
    
    @Column(nullable = false)
    private String eventType;  // "OrderCreatedEvent"
    
    @Column(nullable = false, columnDefinition = "TEXT")
    private String payload;  // JSON
    
    @Column(nullable = false)
    private LocalDateTime createdAt;
    
    @Column(nullable = false)
    private Boolean published = false;
}

@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final OutboxRepository outboxRepository;
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // 1. Create order (database write)
        Order order = new Order();
        order = orderRepository.save(order);
        
        // 2. Save event to outbox (same transaction!)
        OutboxEvent outboxEvent = new OutboxEvent();
        outboxEvent.setAggregateType("Order");
        outboxEvent.setAggregateId(order.getId());
        outboxEvent.setEventType("OrderCreatedEvent");
        outboxEvent.setPayload(objectMapper.writeValueAsString(
            new OrderCreatedEvent(order)
        ));
        outboxEvent.setCreatedAt(LocalDateTime.now());
        
        outboxRepository.save(outboxEvent);
        
        // Both saved in single transaction - guaranteed consistency
        return order;
    }
}

// Separate process publishes events
@Component
public class OutboxEventPublisher {
    
    @Scheduled(fixedDelay = 1000)  // Every second
    public void publishPendingEvents() {
        List<OutboxEvent> pendingEvents = outboxRepository
            .findByPublishedFalseOrderByCreatedAtAsc(PageRequest.of(0, 100));
        
        for (OutboxEvent event : pendingEvents) {
            try {
                // Publish to Kafka
                kafkaTemplate.send(
                    event.getAggregateType().toLowerCase() + "-events",
                    event.getAggregateId().toString(),
                    event.getPayload()
                );
                
                // Mark as published
                event.setPublished(true);
                outboxRepository.save(event);
                
            } catch (Exception e) {
                logger.error("Failed to publish event: {}", event.getId(), e);
                // Will retry on next run
            }
        }
    }
}
```

---

## Summary and Key Takeaways

### Microservices Architecture Principles

**Core Concepts:**
* **Single Responsibility**: Each service owns one business capability
* **Database per Service**: Data independence and isolation
* **Decentralized Governance**: Teams choose their technology
* **API First**: Contract-driven development
* **Design for Failure**: Assume everything can fail

**Benefits:**
* Independent deployment and scaling
* Technology diversity
* Fault isolation
* Team autonomy
* Easier to understand individual services

**Challenges:**
* Distributed system complexity
* Network failures and latency
* Data consistency
* Testing complexity
* Operational overhead

### Service Communication

**Synchronous (Request-Response):**
* REST with Feign/WebClient
* Service discovery with Eureka
* Client-side load balancing
* Circuit breakers for resilience

**Asynchronous (Event-Driven):**
* Kafka/RabbitMQ for messaging
* Event sourcing for audit trail
* CQRS for read/write separation
* SAGA for distributed transactions

### Resilience Patterns

**Circuit Breaker:**
* Prevents cascading failures
* Fast-fail when service down
* Automatic recovery testing
* Fallback responses

**Retry:**
* Transient failure handling
* Exponential backoff
* Maximum retry limits

**Bulkhead:**
* Isolate resources
* Prevent thread pool exhaustion

**Rate Limiting:**
* Protect services from overload
* Per-user or per-API-key limits

### Observability

**Distributed Tracing:**
* Track requests across services
* Identify bottlenecks
* Debug distributed systems
* Sleuth + Zipkin integration

**Metrics:**
* Micrometer + Prometheus
* Business and technical metrics
* Custom metrics per service
* Grafana dashboards

**Logging:**
* Centralized log aggregation
* Correlation IDs
* Structured logging (JSON)

### Data Management

**Patterns:**
* **Database per Service**: Data independence
* **API Composition**: Join data at API layer
* **CQRS**: Separate read and write models
* **Event Sourcing**: Store events, not state
* **SAGA**: Distributed transactions
* **Outbox**: Reliable event publishing

**Trade-offs:**
* Consistency vs Availability
* Strong vs Eventual consistency
* Complexity vs Scalability

### Production Checklist

**Service Design:**
- [ ] Single responsibility per service
- [ ] Database per service
- [ ] API contracts defined (OpenAPI)
- [ ] Health check endpoints
- [ ] Graceful shutdown

**Communication:**
- [ ] Service discovery configured
- [ ] Circuit breakers implemented
- [ ] Timeouts configured
- [ ] Retry logic with backoff
- [ ] Fallbacks defined

**Observability:**
- [ ] Distributed tracing enabled
- [ ] Metrics exported (Prometheus)
- [ ] Centralized logging
- [ ] Health checks exposed
- [ ] Dashboards created (Grafana)

**Resilience:**
- [ ] Circuit breakers tested
- [ ] Chaos testing performed
- [ ] Failure scenarios documented
- [ ] Runbooks created

**Data:**
- [ ] Transaction boundaries clear
- [ ] Compensating transactions defined
- [ ] Event schema versioning
- [ ] Data migration strategy

### Frequently Asked Questions

**Q1: When should I use microservices vs monolith?**

**A:**

**Use Microservices when:**
* Large, complex application
* Multiple teams (>3-4 teams)
* Need independent deployment
* Different scalability requirements per component
* Long-term project (multi-year)

**Use Monolith when:**
* Small to medium application
* Small team (<10 developers)
* Unclear domain boundaries
* Startup/MVP (speed matters most)
* Simple deployment requirements

**Start with modular monolith**, extract microservices as needed.

---

**Q2: How do I handle distributed transactions across microservices?**

**A:**

Avoid distributed transactions (2PC/XA). Instead:

**1. SAGA Pattern** (recommended):
```java
// Choreography: Each service reacts to events
Order created → Payment processed → Inventory reserved → Shipment created

// Orchestration: Orchestrator coordinates steps
Orchestrator controls: Create order → Process payment → Reserve inventory
```

**2. Eventual Consistency:**
* Accept temporary inconsistency
* Design for eventual consistency
* Use compensating transactions

**3. Outbox Pattern:**
* Guarantee event publishing
* Single database transaction
* Background process publishes events

---

**Q3: How many microservices should I have?**

**A:**

**No magic number**, but guidelines:

* **Start small**: 3-5 services initially
* **Grow organically**: Split when needed
* **Team size**: 2-pizza team per 1-3 services
* **Domain boundaries**: Follow DDD bounded contexts
* **Independence**: Each service should be independently deployable

**Anti-patterns:**
* Too many micro-services (nano-services)
* Services that always deploy together
* Chatty services (many inter-service calls)

**Right size:**
* Can be understood by one developer
* Has clear business responsibility
* Can be deployed independently
* Has its own database/data

### Interview Questions

**Question 1: Explain the difference between choreography and orchestration in SAGA pattern.**

**Difficulty:** Senior

**Topics:** Microservices, Data Management

**Answer:**

**Choreography:**
* **Decentralized**: Each service knows what to do when event occurs
* **Event-driven**: Services react to events
* **No coordinator**: Services collaborate peer-to-peer
* **Loose coupling**: Services don't know about each other

```
Order Service → OrderCreated event → Kafka
    ↓
Payment Service listens → Process payment → PaymentCompleted event
    ↓
Inventory Service listens → Reserve inventory → InventoryReserved event
```

**Orchestration:**
* **Centralized**: Orchestrator controls workflow
* **Command-driven**: Orchestrator tells services what to do
* **Single coordinator**: Orchestrator knows all steps
* **Tighter coupling**: Services know orchestrator

```
Orchestrator → Create order (Order Service)
             → Process payment (Payment Service)
             → Reserve inventory (Inventory Service)
```

**When to use:**
* **Choreography**: Simple workflows, highly decoupled services
* **Orchestration**: Complex workflows, need visibility/control

---

**Question 2: How do you implement circuit breaker pattern and why is it important?**

**Difficulty:** Mid-Level

**Topics:** Resilience, Microservices

**Answer:**

Circuit breaker prevents cascading failures:

**States:**
1. **CLOSED**: Normal operation, track failures
2. **OPEN**: Threshold exceeded, fail fast (no calls)
3. **HALF_OPEN**: Test if service recovered

**Implementation with Resilience4j:**
```java
@CircuitBreaker(name = "productService", fallbackMethod = "getFallback")
public Product getProduct(Long id) {
    return productClient.getProduct(id);
}

private Product getFallback(Long id, Throwable ex) {
    return productCache.get(id);
}
```

**Why important:**
* **Prevents resource exhaustion**: Don't waste threads on failing calls
* **Fast failure**: Immediate response instead of timeout
* **Automatic recovery**: Tests service health periodically
* **Cascading failure prevention**: Stop failure propagation

**Configuration:**
* Failure threshold: 50% in 10 requests
* Open duration: 10 seconds
* Half-open test calls: 3

---

**End of Part 6: Microservices Architecture**

### Congratulations!

You've completed the **Complete Java Mastery series**:
* Part 1: Java Fundamentals ✅
* Part 2: JVM Internals ✅
* Part 3: Advanced Java Features ✅
* Part 4: SDLC with Java ✅
* Part 5: Spring Framework ✅
* Part 6: Microservices Architecture ✅

**Total: 6 comprehensive parts** covering everything from Java basics to distributed systems!

### References

* [Microservices Patterns by Chris Richardson](https://microservices.io/patterns/)
* [Building Microservices by Sam Newman](https://www.oreilly.com/library/view/building-microservices-2nd/9781492034018/)
* [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
* [Resilience4j Documentation](https://resilience4j.readme.io/)
* [Martin Fowler's Microservices Resource Guide](https://martinfowler.com/microservices/)
* [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)

---
