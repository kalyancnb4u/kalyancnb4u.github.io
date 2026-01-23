---
title: "Complete Java Mastery Part 7: Advanced Spring & Reactive Programming"
date: 2024-01-21 10:00:00 +0000
categories: [Java, Spring, Reactive-Programming]
tags: [java, spring-webflux, project-reactor, reactive-streams, spring-batch, spring-integration, reactive-programming, non-blocking]
---

## Introduction

Welcome to Part 7 of the Complete Java Mastery series. In Parts 1-6, we covered Java fundamentals through microservices architecture. Now we explore **Advanced Spring Features** and **Reactive Programming** - the paradigm shift from blocking to non-blocking, asynchronous programming.

Reactive programming enables building highly scalable, resilient applications that efficiently use system resources. Spring WebFlux, built on Project Reactor, brings reactive programming to the Spring ecosystem, allowing you to handle massive concurrency with minimal threads.

### What You'll Learn in Part 7

* **Reactive Programming Fundamentals**: Reactive Streams, backpressure, operators
* **Project Reactor**: Mono, Flux, reactive operators, error handling
* **Spring WebFlux**: Reactive REST APIs, functional endpoints, WebClient
* **Reactive Data Access**: R2DBC, reactive repositories, reactive transactions
* **Spring Batch**: Batch processing, jobs, steps, chunk processing
* **Spring Integration**: Enterprise Integration Patterns, message flows
* **Caching Strategies**: Redis, Caffeine, distributed caching
* **Advanced Patterns**: Reactive event sourcing, CQRS with reactive

This knowledge enables you to:
* Build high-performance reactive applications
* Handle millions of concurrent connections
* Process large batches efficiently
* Implement enterprise integration patterns
* Design scalable caching strategies

---

## 7.1 Reactive Programming Fundamentals

### The Problem with Blocking I/O

**Traditional Blocking Approach:**

```java
// ❌ Blocking: One thread per request
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // Thread blocks waiting for database
        User user = userRepository.findById(id).orElseThrow();
        
        // Thread blocks waiting for external API
        Profile profile = externalApi.getProfile(user.getEmail());
        
        // Thread blocks waiting for cache
        Preferences prefs = cache.get(id);
        
        return user;  // Thread tied up for entire request
    }
}

// With 1000 concurrent requests:
// - Need 1000 threads
// - Each thread: ~1MB memory
// - Context switching overhead
// - Most threads just waiting!
```

**Reactive Non-Blocking Approach:**

```java
// ✅ Non-blocking: Few threads handle many requests
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userRepository.findById(id)  // Non-blocking
            .zipWith(externalApi.getProfile(id))  // Parallel non-blocking
            .zipWith(cache.get(id))  // Parallel non-blocking
            .map(tuple -> enrichUser(tuple));
        
        // Thread returns immediately
        // Callbacks execute when data arrives
        // Same thread can handle other requests
    }
}

// With 1000 concurrent requests:
// - Need ~10-20 threads (based on CPU cores)
// - Minimal memory footprint
// - No blocking, no waiting
```

### Reactive Streams Specification

**Core Interfaces:**

```java
// Publisher: Produces items
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> subscriber);
}

// Subscriber: Consumes items
public interface Subscriber<T> {
    void onSubscribe(Subscription subscription);
    void onNext(T item);
    void onError(Throwable throwable);
    void onComplete();
}

// Subscription: Controls flow
public interface Subscription {
    void request(long n);  // Request n items (backpressure)
    void cancel();         // Cancel subscription
}

// Processor: Both publisher and subscriber
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

**How It Works:**

```
Publisher                Subscriber
    │                        │
    │     subscribe()        │
    │◄───────────────────────┤
    │                        │
    │   onSubscribe(sub)     │
    ├───────────────────────►│
    │                        │
    │     request(10)        │
    │◄───────────────────────┤ Backpressure: Request 10 items
    │                        │
    │      onNext(1)         │
    ├───────────────────────►│
    │      onNext(2)         │
    ├───────────────────────►│
    │        ...             │
    │      onNext(10)        │
    ├───────────────────────►│
    │                        │
    │     request(10)        │
    │◄───────────────────────┤ Request more
    │                        │
    │    onComplete()        │
    ├───────────────────────►│
```

### Backpressure

**The Problem:**

```
Fast Publisher                  Slow Subscriber
Produces 1000 items/sec    ─►   Processes 10 items/sec
                                
Without backpressure:
- Subscriber overwhelmed
- Memory overflow
- System crash
```

**The Solution:**

```java
// Subscriber controls rate
subscription.request(10);  // "I can handle 10 items"

// Publisher sends exactly 10 items
// Subscriber processes them
// Then requests more

subscription.request(10);  // "Ready for 10 more"
```

---

## 7.2 Project Reactor

Project Reactor is Spring's reactive library implementing Reactive Streams.

### Mono and Flux

**Mono: 0 or 1 item**

```java
// Create Mono
Mono<String> mono1 = Mono.just("Hello");
Mono<String> mono2 = Mono.empty();
Mono<String> mono3 = Mono.error(new RuntimeException("Error"));

// From Callable
Mono<String> mono4 = Mono.fromCallable(() -> {
    return expensiveOperation();
});

// From Supplier
Mono<String> mono5 = Mono.fromSupplier(() -> "Lazy evaluation");

// Defer (lazy)
Mono<String> mono6 = Mono.defer(() -> {
    // Created when subscribed
    return Mono.just(getCurrentTime());
});
```

**Flux: 0 to N items**

```java
// Create Flux
Flux<String> flux1 = Flux.just("A", "B", "C");
Flux<Integer> flux2 = Flux.range(1, 10);  // 1 to 10
Flux<Long> flux3 = Flux.interval(Duration.ofSeconds(1));  // Infinite

// From Iterable
List<String> list = Arrays.asList("A", "B", "C");
Flux<String> flux4 = Flux.fromIterable(list);

// From Stream
Stream<String> stream = Stream.of("A", "B", "C");
Flux<String> flux5 = Flux.fromStream(stream);

// Generate
Flux<Integer> flux6 = Flux.generate(
    () -> 0,  // Initial state
    (state, sink) -> {
        sink.next(state);  // Emit item
        if (state == 10) {
            sink.complete();  // Stop
        }
        return state + 1;  // Next state
    }
);

// Create (async)
Flux<Integer> flux7 = Flux.create(sink -> {
    for (int i = 0; i < 10; i++) {
        sink.next(i);
    }
    sink.complete();
});
```

### Subscribing

```java
// Simple subscribe
Flux.just("A", "B", "C")
    .subscribe(
        item -> System.out.println("Received: " + item),  // onNext
        error -> System.err.println("Error: " + error),   // onError
        () -> System.out.println("Completed")             // onComplete
    );

// Subscribe with Subscription control
Flux.range(1, 100)
    .subscribe(new Subscriber<Integer>() {
        private Subscription subscription;
        
        @Override
        public void onSubscribe(Subscription s) {
            this.subscription = s;
            s.request(10);  // Request first 10
        }
        
        @Override
        public void onNext(Integer item) {
            System.out.println("Received: " + item);
            // Request more after processing
            subscription.request(1);
        }
        
        @Override
        public void onError(Throwable t) {
            System.err.println("Error: " + t);
        }
        
        @Override
        public void onComplete() {
            System.out.println("Done!");
        }
    });

// Block (for testing - don't use in production!)
String result = Mono.just("Hello").block();  // Blocks until value arrives
List<String> results = Flux.just("A", "B", "C").collectList().block();
```

### Operators

**Transformation:**

```java
// map: Transform each item
Flux.just(1, 2, 3)
    .map(n -> n * 2)
    .subscribe(System.out::println);  // 2, 4, 6

// flatMap: Transform to Publisher, flatten
Flux.just("A", "B", "C")
    .flatMap(letter -> {
        return Mono.just(letter)
            .delayElement(Duration.ofMillis(100))
            .map(String::toLowerCase);
    })
    .subscribe(System.out::println);

// flatMapSequential: Maintains order
Flux.just(1, 2, 3)
    .flatMapSequential(n -> {
        return Mono.just(n)
            .delayElement(Duration.ofMillis(100));
    })
    .subscribe(System.out::println);  // Still 1, 2, 3

// concatMap: Sequential (waits for each to complete)
Flux.just(1, 2, 3)
    .concatMap(n -> {
        return Mono.just(n * 2)
            .delayElement(Duration.ofSeconds(1));
    })
    .subscribe(System.out::println);

// filter: Keep items matching predicate
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)
    .subscribe(System.out::println);  // 2, 4, 6, 8, 10

// take: Take first N items
Flux.range(1, 100)
    .take(5)
    .subscribe(System.out::println);  // 1, 2, 3, 4, 5

// skip: Skip first N items
Flux.range(1, 10)
    .skip(5)
    .subscribe(System.out::println);  // 6, 7, 8, 9, 10
```

**Combining:**

```java
// zip: Combine items from multiple publishers
Mono<String> mono1 = Mono.just("Hello");
Mono<String> mono2 = Mono.just("World");

Mono.zip(mono1, mono2)
    .map(tuple -> tuple.getT1() + " " + tuple.getT2())
    .subscribe(System.out::println);  // "Hello World"

// zipWith
Flux.just(1, 2, 3)
    .zipWith(Flux.just("A", "B", "C"))
    .subscribe(tuple -> {
        System.out.println(tuple.getT1() + " -> " + tuple.getT2());
    });
// Output: 1 -> A, 2 -> B, 3 -> C

// merge: Interleave items (order not guaranteed)
Flux<Integer> flux1 = Flux.just(1, 2, 3);
Flux<Integer> flux2 = Flux.just(4, 5, 6);

Flux.merge(flux1, flux2)
    .subscribe(System.out::println);  // 1, 4, 2, 5, 3, 6 (or any order)

// concat: Sequential (flux1 completes before flux2)
Flux.concat(flux1, flux2)
    .subscribe(System.out::println);  // 1, 2, 3, 4, 5, 6 (guaranteed order)
```

**Error Handling:**

```java
// onErrorReturn: Return default value on error
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)  // Division by zero!
    .onErrorReturn(-1)
    .subscribe(System.out::println);
// Output: 10, 5, -1

// onErrorResume: Switch to fallback publisher
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .onErrorResume(error -> {
        System.err.println("Error: " + error.getMessage());
        return Flux.just(-1, -2, -3);
    })
    .subscribe(System.out::println);

// retry: Retry on error
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .retry(3)  // Retry up to 3 times
    .subscribe(
        System.out::println,
        error -> System.err.println("Failed after retries: " + error)
    );

// retryWhen: Custom retry logic
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)))
    .subscribe();

// doOnError: Side effect on error (doesn't handle)
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .doOnError(error -> {
        System.err.println("Logging error: " + error.getMessage());
    })
    .onErrorReturn(-1)
    .subscribe(System.out::println);
```

**Side Effects (Peeking):**

```java
Flux.just(1, 2, 3)
    .doOnSubscribe(s -> System.out.println("Subscribed"))
    .doOnNext(n -> System.out.println("Processing: " + n))
    .map(n -> n * 2)
    .doOnNext(n -> System.out.println("After map: " + n))
    .doOnComplete(() -> System.out.println("Completed"))
    .doOnError(e -> System.err.println("Error: " + e))
    .doFinally(signal -> System.out.println("Finally: " + signal))
    .subscribe();
```

### Schedulers

Control which thread executes operations:

```java
// Schedulers.immediate() - Current thread (default)
Flux.just(1, 2, 3)
    .map(n -> {
        System.out.println("Map on: " + Thread.currentThread().getName());
        return n * 2;
    })
    .subscribe();

// Schedulers.single() - Single reusable thread
Flux.just(1, 2, 3)
    .publishOn(Schedulers.single())
    .map(n -> {
        System.out.println("Map on: " + Thread.currentThread().getName());
        return n * 2;
    })
    .subscribe();

// Schedulers.parallel() - Fixed pool of workers (CPU cores)
Flux.range(1, 100)
    .parallel()
    .runOn(Schedulers.parallel())
    .map(n -> {
        System.out.println("Map on: " + Thread.currentThread().getName());
        return n * 2;
    })
    .sequential()
    .subscribe();

// Schedulers.boundedElastic() - Elastic pool for blocking I/O
Flux.range(1, 10)
    .publishOn(Schedulers.boundedElastic())
    .map(n -> {
        // Blocking I/O operation
        Thread.sleep(100);
        return n;
    })
    .subscribe();

// subscribeOn: Changes thread for entire chain
Mono.fromCallable(() -> {
    System.out.println("Callable on: " + Thread.currentThread().getName());
    return "Hello";
})
.subscribeOn(Schedulers.boundedElastic())
.subscribe(result -> {
    System.out.println("Subscribe on: " + Thread.currentThread().getName());
});

// publishOn: Changes thread from that point onwards
Flux.range(1, 5)
    .map(n -> {
        System.out.println("Map1 on: " + Thread.currentThread().getName());
        return n;
    })
    .publishOn(Schedulers.parallel())
    .map(n -> {
        System.out.println("Map2 on: " + Thread.currentThread().getName());
        return n * 2;
    })
    .subscribe();
```

---

## 7.3 Spring WebFlux

Spring WebFlux is the reactive web framework built on Project Reactor.

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

**⚠️ Important:** WebFlux and Spring MVC are mutually exclusive. Choose one.

### Reactive REST Controller

**Annotation-Based (Similar to Spring MVC):**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    // GET - Return Mono
    @GetMapping("/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // GET - Return Flux
    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.findAll();
    }
    
    // GET with query params
    @GetMapping("/search")
    public Flux<User> searchUsers(
        @RequestParam String name,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size
    ) {
        return userService.searchByName(name, page, size);
    }
    
    // POST - Create user
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<User> createUser(@Valid @RequestBody Mono<User> userMono) {
        return userMono.flatMap(userService::create);
    }
    
    // PUT - Update user
    @PutMapping("/{id}")
    public Mono<User> updateUser(
        @PathVariable Long id,
        @RequestBody Mono<User> userMono
    ) {
        return userMono.flatMap(user -> userService.update(id, user));
    }
    
    // DELETE - Delete user
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deleteUser(@PathVariable Long id) {
        return userService.delete(id);
    }
    
    // Server-Sent Events (SSE)
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamUsers() {
        return userService.findAll()
            .delayElements(Duration.ofSeconds(1));
    }
}
```

**Functional Endpoints (Recommended for Reactive):**

```java
@Configuration
public class UserRoutes {
    
    @Bean
    public RouterFunction<ServerResponse> userRouter(UserHandler handler) {
        return RouterFunctions
            .route(GET("/api/users/{id}"), handler::getUser)
            .andRoute(GET("/api/users"), handler::getAllUsers)
            .andRoute(POST("/api/users"), handler::createUser)
            .andRoute(PUT("/api/users/{id}"), handler::updateUser)
            .andRoute(DELETE("/api/users/{id}"), handler::deleteUser)
            .andRoute(GET("/api/users/stream"), handler::streamUsers);
    }
}

@Component
public class UserHandler {
    
    private final UserService userService;
    
    public UserHandler(UserService userService) {
        this.userService = userService;
    }
    
    public Mono<ServerResponse> getUser(ServerRequest request) {
        Long id = Long.valueOf(request.pathVariable("id"));
        
        return userService.findById(id)
            .flatMap(user -> ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(user))
            .switchIfEmpty(ServerResponse.notFound().build());
    }
    
    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        return ServerResponse.ok()
            .contentType(MediaType.APPLICATION_JSON)
            .body(userService.findAll(), User.class);
    }
    
    public Mono<ServerResponse> createUser(ServerRequest request) {
        Mono<User> userMono = request.bodyToMono(User.class);
        
        return userMono
            .flatMap(userService::create)
            .flatMap(user -> ServerResponse
                .created(URI.create("/api/users/" + user.getId()))
                .bodyValue(user));
    }
    
    public Mono<ServerResponse> updateUser(ServerRequest request) {
        Long id = Long.valueOf(request.pathVariable("id"));
        Mono<User> userMono = request.bodyToMono(User.class);
        
        return userMono
            .flatMap(user -> userService.update(id, user))
            .flatMap(user -> ServerResponse.ok().bodyValue(user))
            .switchIfEmpty(ServerResponse.notFound().build());
    }
    
    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        Long id = Long.valueOf(request.pathVariable("id"));
        
        return userService.delete(id)
            .then(ServerResponse.noContent().build());
    }
    
    public Mono<ServerResponse> streamUsers(ServerRequest request) {
        Flux<User> userStream = userService.findAll()
            .delayElements(Duration.ofSeconds(1));
        
        return ServerResponse.ok()
            .contentType(MediaType.TEXT_EVENT_STREAM)
            .body(userStream, User.class);
    }
}
```

### Reactive Service Layer

```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final WebClient webClient;
    
    public Mono<User> findById(Long id) {
        return userRepository.findById(id)
            .switchIfEmpty(Mono.error(new UserNotFoundException(id)));
    }
    
    public Flux<User> findAll() {
        return userRepository.findAll();
    }
    
    public Mono<User> create(User user) {
        return userRepository.save(user);
    }
    
    public Mono<User> update(Long id, User user) {
        return userRepository.findById(id)
            .flatMap(existingUser -> {
                existingUser.setName(user.getName());
                existingUser.setEmail(user.getEmail());
                return userRepository.save(existingUser);
            })
            .switchIfEmpty(Mono.error(new UserNotFoundException(id)));
    }
    
    public Mono<Void> delete(Long id) {
        return userRepository.deleteById(id);
    }
    
    // Complex composition
    public Mono<UserDetails> getUserDetails(Long id) {
        Mono<User> userMono = findById(id);
        Mono<Profile> profileMono = getProfile(id);
        Mono<List<Order>> ordersMono = getOrders(id).collectList();
        
        return Mono.zip(userMono, profileMono, ordersMono)
            .map(tuple -> new UserDetails(
                tuple.getT1(),  // User
                tuple.getT2(),  // Profile
                tuple.getT3()   // Orders
            ));
    }
    
    // Call external API
    private Mono<Profile> getProfile(Long userId) {
        return webClient.get()
            .uri("/profiles/{id}", userId)
            .retrieve()
            .bodyToMono(Profile.class)
            .timeout(Duration.ofSeconds(5))
            .onErrorResume(error -> {
                log.error("Failed to get profile", error);
                return Mono.just(Profile.defaultProfile());
            });
    }
    
    private Flux<Order> getOrders(Long userId) {
        return webClient.get()
            .uri("/orders?userId={userId}", userId)
            .retrieve()
            .bodyToFlux(Order.class);
    }
}
```

---

## 7.4 Reactive Data Access with R2DBC

R2DBC (Reactive Relational Database Connectivity) provides reactive database access.

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>
```

```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/mydb
    username: postgres
    password: secret
  data:
    r2dbc:
      repositories:
        enabled: true
```

### Entity

```java
@Table("users")
public class User {
    
    @Id
    private Long id;
    
    @Column("email")
    private String email;
    
    @Column("name")
    private String name;
    
    @Column("created_at")
    private LocalDateTime createdAt;
    
    @Column("updated_at")
    private LocalDateTime updatedAt;
    
    // Constructors, getters, setters
}
```

### Reactive Repository

```java
public interface UserRepository extends ReactiveCrudRepository<User, Long> {
    
    // Derived query methods
    Mono<User> findByEmail(String email);
    Flux<User> findByNameContaining(String name);
    Flux<User> findByCreatedAtAfter(LocalDateTime date);
    
    // Custom query
    @Query("SELECT * FROM users WHERE email = :email AND active = true")
    Mono<User> findActiveUserByEmail(String email);
    
    @Query("SELECT * FROM users WHERE created_at > :date ORDER BY created_at DESC")
    Flux<User> findRecentUsers(LocalDateTime date);
    
    // Named query with sorting
    Flux<User> findAllBy(Sort sort);
    
    // Exists check
    Mono<Boolean> existsByEmail(String email);
    
    // Count
    Mono<Long> countByCreatedAtAfter(LocalDateTime date);
    
    // Delete
    Mono<Void> deleteByEmail(String email);
}
```

### Using Repository

```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public Mono<User> createUser(User user) {
        user.setCreatedAt(LocalDateTime.now());
        user.setUpdatedAt(LocalDateTime.now());
        
        return userRepository.save(user);
    }
    
    public Mono<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    public Flux<User> findAll() {
        return userRepository.findAll();
    }
    
    public Flux<User> findAllSorted() {
        return userRepository.findAllBy(Sort.by("createdAt").descending());
    }
    
    public Mono<User> updateUser(Long id, User updates) {
        return userRepository.findById(id)
            .flatMap(existing -> {
                existing.setName(updates.getName());
                existing.setEmail(updates.getEmail());
                existing.setUpdatedAt(LocalDateTime.now());
                return userRepository.save(existing);
            });
    }
    
    public Mono<Void> deleteUser(Long id) {
        return userRepository.deleteById(id);
    }
    
    // Complex query
    public Flux<User> searchUsers(String searchTerm) {
        return userRepository.findByNameContaining(searchTerm)
            .take(100)  // Limit results
            .timeout(Duration.ofSeconds(5));  // Timeout protection
    }
}
```

### Reactive Transactions

```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final InventoryRepository inventoryRepository;
    private final TransactionalOperator transactionalOperator;
    
    public OrderService(OrderRepository orderRepository,
                       InventoryRepository inventoryRepository,
                       ReactiveTransactionManager transactionManager) {
        this.orderRepository = orderRepository;
        this.inventoryRepository = inventoryRepository;
        this.transactionalOperator = TransactionalOperator.create(transactionManager);
    }
    
    public Mono<Order> createOrder(Order order) {
        return orderRepository.save(order)
            .flatMap(savedOrder -> 
                inventoryRepository.reserve(savedOrder.getProductId(), savedOrder.getQuantity())
                    .thenReturn(savedOrder)
            )
            .as(transactionalOperator::transactional)  // Wrap in transaction
            .onErrorResume(error -> {
                log.error("Order creation failed", error);
                return Mono.error(new OrderCreationException(error));
            });
    }
    
    // Or use @Transactional
    @Transactional
    public Mono<Order> createOrderDeclarative(Order order) {
        return orderRepository.save(order)
            .flatMap(savedOrder -> 
                inventoryRepository.reserve(savedOrder.getProductId(), savedOrder.getQuantity())
                    .thenReturn(savedOrder)
            );
    }
}
```

### DatabaseClient (Advanced)

```java
@Repository
public class CustomUserRepository {
    
    private final DatabaseClient databaseClient;
    
    public Mono<User> findByEmailCustom(String email) {
        return databaseClient.sql("SELECT * FROM users WHERE email = :email")
            .bind("email", email)
            .map((row, metadata) -> {
                User user = new User();
                user.setId(row.get("id", Long.class));
                user.setEmail(row.get("email", String.class));
                user.setName(row.get("name", String.class));
                return user;
            })
            .one();
    }
    
    public Flux<User> searchUsers(String query, int limit) {
        return databaseClient.sql("""
                SELECT * FROM users 
                WHERE name ILIKE :query 
                   OR email ILIKE :query 
                ORDER BY created_at DESC 
                LIMIT :limit
                """)
            .bind("query", "%" + query + "%")
            .bind("limit", limit)
            .fetch()
            .all()
            .map(this::mapToUser);
    }
    
    public Mono<Integer> updateUserStatus(Long userId, String status) {
        return databaseClient.sql("""
                UPDATE users 
                SET status = :status, updated_at = :updatedAt 
                WHERE id = :id
                """)
            .bind("id", userId)
            .bind("status", status)
            .bind("updatedAt", LocalDateTime.now())
            .fetch()
            .rowsUpdated();
    }
    
    private User mapToUser(Map<String, Object> row) {
        User user = new User();
        user.setId((Long) row.get("id"));
        user.setEmail((String) row.get("email"));
        user.setName((String) row.get("name"));
        return user;
    }
}
```

---

## 7.5 WebClient - Reactive HTTP Client

WebClient is the reactive replacement for RestTemplate.

### Configuration

```java
@Configuration
public class WebClientConfig {
    
    @Bean
    public WebClient webClient() {
        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .defaultHeader(HttpHeaders.USER_AGENT, "MyApp/1.0")
            .codecs(configurer -> configurer
                .defaultCodecs()
                .maxInMemorySize(16 * 1024 * 1024))  // 16MB buffer
            .build();
    }
    
    @Bean
    public WebClient authWebClient() {
        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .filter(ExchangeFilterFunctions.basicAuthentication("user", "password"))
            .build();
    }
    
    @Bean
    public WebClient timeoutWebClient() {
        HttpClient httpClient = HttpClient.create()
            .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
            .responseTimeout(Duration.ofSeconds(5))
            .doOnConnected(conn -> 
                conn.addHandlerLast(new ReadTimeoutHandler(5))
                    .addHandlerLast(new WriteTimeoutHandler(5))
            );
        
        return WebClient.builder()
            .clientConnector(new ReactorClientHttpConnector(httpClient))
            .build();
    }
}
```

### Basic Requests

```java
@Service
public class ApiClient {
    
    private final WebClient webClient;
    
    // GET request
    public Mono<User> getUser(Long id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class);
    }
    
    // GET with query parameters
    public Flux<User> searchUsers(String query, int page, int size) {
        return webClient.get()
            .uri(uriBuilder -> uriBuilder
                .path("/users/search")
                .queryParam("q", query)
                .queryParam("page", page)
                .queryParam("size", size)
                .build())
            .retrieve()
            .bodyToFlux(User.class);
    }
    
    // POST request
    public Mono<User> createUser(User user) {
        return webClient.post()
            .uri("/users")
            .bodyValue(user)
            .retrieve()
            .bodyToMono(User.class);
    }
    
    // PUT request
    public Mono<User> updateUser(Long id, User user) {
        return webClient.put()
            .uri("/users/{id}", id)
            .bodyValue(user)
            .retrieve()
            .bodyToMono(User.class);
    }
    
    // DELETE request
    public Mono<Void> deleteUser(Long id) {
        return webClient.delete()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(Void.class);
    }
    
    // Headers
    public Mono<User> getUserWithHeaders(Long id, String token) {
        return webClient.get()
            .uri("/users/{id}", id)
            .header("Authorization", "Bearer " + token)
            .header("X-Request-ID", UUID.randomUUID().toString())
            .retrieve()
            .bodyToMono(User.class);
    }
}
```

### Error Handling

```java
@Service
public class ApiClient {
    
    public Mono<User> getUserWithErrorHandling(Long id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .onStatus(HttpStatus::is4xxClientError, response -> {
                return response.bodyToMono(ErrorResponse.class)
                    .flatMap(error -> Mono.error(
                        new ClientException("Client error: " + error.getMessage())
                    ));
            })
            .onStatus(HttpStatus::is5xxServerError, response -> {
                return Mono.error(new ServerException("Server error occurred"));
            })
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5))
            .retry(3)
            .onErrorResume(TimeoutException.class, e -> {
                log.error("Request timeout for user: {}", id);
                return Mono.error(new ApiTimeoutException(e));
            });
    }
    
    // Using exchangeToMono for full control
    public Mono<User> getUserAdvanced(Long id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .exchangeToMono(response -> {
                if (response.statusCode().equals(HttpStatus.OK)) {
                    return response.bodyToMono(User.class);
                } else if (response.statusCode().equals(HttpStatus.NOT_FOUND)) {
                    return Mono.error(new UserNotFoundException(id));
                } else {
                    return response.createException()
                        .flatMap(Mono::error);
                }
            });
    }
}
```

### Advanced Features

```java
@Service
public class AdvancedApiClient {
    
    // Parallel requests
    public Mono<UserProfile> getUserProfile(Long userId) {
        Mono<User> userMono = webClient.get()
            .uri("/users/{id}", userId)
            .retrieve()
            .bodyToMono(User.class);
        
        Mono<List<Order>> ordersMono = webClient.get()
            .uri("/orders?userId={userId}", userId)
            .retrieve()
            .bodyToFlux(Order.class)
            .collectList();
        
        Mono<Preferences> prefsMono = webClient.get()
            .uri("/preferences/{userId}", userId)
            .retrieve()
            .bodyToMono(Preferences.class);
        
        return Mono.zip(userMono, ordersMono, prefsMono)
            .map(tuple -> new UserProfile(
                tuple.getT1(),  // User
                tuple.getT2(),  // Orders
                tuple.getT3()   // Preferences
            ));
    }
    
    // Streaming response
    public Flux<Event> streamEvents() {
        return webClient.get()
            .uri("/events/stream")
            .accept(MediaType.TEXT_EVENT_STREAM)
            .retrieve()
            .bodyToFlux(Event.class)
            .take(Duration.ofMinutes(5));  // Stream for 5 minutes
    }
    
    // File upload
    public Mono<UploadResponse> uploadFile(byte[] fileData, String filename) {
        MultiValueMap<String, HttpEntity<?>> body = new LinkedMultiValueMap<>();
        body.add("file", new ByteArrayResource(fileData) {
            @Override
            public String getFilename() {
                return filename;
            }
        });
        
        return webClient.post()
            .uri("/upload")
            .contentType(MediaType.MULTIPART_FORM_DATA)
            .bodyValue(body)
            .retrieve()
            .bodyToMono(UploadResponse.class);
    }
    
    // Retry with backoff
    public Mono<Data> getDataWithRetry() {
        return webClient.get()
            .uri("/data")
            .retrieve()
            .bodyToMono(Data.class)
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
                .maxBackoff(Duration.ofSeconds(10))
                .filter(throwable -> throwable instanceof TimeoutException)
                .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> {
                    throw new ApiException("Max retries exceeded");
                }));
    }
}
```

---

## 7.6 Spring Batch

Spring Batch is a framework for batch processing large volumes of data.

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableBatchProcessing
public class BatchApplication {
    public static void main(String[] args) {
        SpringApplication.run(BatchApplication.class, args);
    }
}
```

### Basic Job Configuration

```java
@Configuration
public class BatchConfig {
    
    @Bean
    public Job importUserJob(JobRepository jobRepository,
                            Step step1,
                            JobCompletionNotificationListener listener) {
        return new JobBuilder("importUserJob", jobRepository)
            .listener(listener)
            .start(step1)
            .build();
    }
    
    @Bean
    public Step step1(JobRepository jobRepository,
                     PlatformTransactionManager transactionManager,
                     ItemReader<User> reader,
                     ItemProcessor<User, User> processor,
                     ItemWriter<User> writer) {
        return new StepBuilder("step1", jobRepository)
            .<User, User>chunk(100, transactionManager)  // Process 100 items at a time
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
    }
}
```

### ItemReader

```java
// Read from CSV
@Bean
public FlatFileItemReader<User> reader() {
    return new FlatFileItemReaderBuilder<User>()
        .name("userItemReader")
        .resource(new ClassPathResource("users.csv"))
        .delimited()
        .names("firstName", "lastName", "email")
        .targetType(User.class)
        .build();
}

// Read from database
@Bean
public JdbcCursorItemReader<User> databaseReader(DataSource dataSource) {
    return new JdbcCursorItemReaderBuilder<User>()
        .name("databaseReader")
        .dataSource(dataSource)
        .sql("SELECT id, first_name, last_name, email FROM users WHERE active = true")
        .rowMapper((rs, rowNum) -> {
            User user = new User();
            user.setId(rs.getLong("id"));
            user.setFirstName(rs.getString("first_name"));
            user.setLastName(rs.getString("last_name"));
            user.setEmail(rs.getString("email"));
            return user;
        })
        .build();
}

// Read from JPA
@Bean
public JpaPagingItemReader<User> jpaReader(EntityManagerFactory entityManagerFactory) {
    return new JpaPagingItemReaderBuilder<User>()
        .name("jpaReader")
        .entityManagerFactory(entityManagerFactory)
        .queryString("SELECT u FROM User u WHERE u.active = true")
        .pageSize(100)
        .build();
}
```

### ItemProcessor

```java
@Component
public class UserItemProcessor implements ItemProcessor<User, User> {
    
    private static final Logger logger = LoggerFactory.getLogger(UserItemProcessor.class);
    
    @Override
    public User process(User user) throws Exception {
        // Transform data
        String firstName = user.getFirstName().toUpperCase();
        String lastName = user.getLastName().toUpperCase();
        String email = user.getEmail().toLowerCase();
        
        User transformed = new User();
        transformed.setFirstName(firstName);
        transformed.setLastName(lastName);
        transformed.setEmail(email);
        
        logger.info("Converting ({}) into ({})", user, transformed);
        
        // Return null to filter out item
        if (email.contains("spam")) {
            logger.warn("Filtering spam user: {}", email);
            return null;
        }
        
        return transformed;
    }
}

// Composite processor (chain multiple processors)
@Bean
public CompositeItemProcessor<User, User> compositeProcessor() {
    CompositeItemProcessor<User, User> processor = new CompositeItemProcessor<>();
    processor.setDelegates(Arrays.asList(
        new ValidationProcessor(),
        new TransformationProcessor(),
        new EnrichmentProcessor()
    ));
    return processor;
}
```

### ItemWriter

```java
// Write to database
@Bean
public JdbcBatchItemWriter<User> writer(DataSource dataSource) {
    return new JdbcBatchItemWriterBuilder<User>()
        .dataSource(dataSource)
        .sql("""
            INSERT INTO users (first_name, last_name, email) 
            VALUES (:firstName, :lastName, :email)
            """)
        .beanMapped()
        .build();
}

// Write to JPA
@Bean
public JpaItemWriter<User> jpaWriter(EntityManagerFactory entityManagerFactory) {
    JpaItemWriter<User> writer = new JpaItemWriter<>();
    writer.setEntityManagerFactory(entityManagerFactory);
    return writer;
}

// Write to file
@Bean
public FlatFileItemWriter<User> fileWriter() {
    return new FlatFileItemWriterBuilder<User>()
        .name("userItemWriter")
        .resource(new FileSystemResource("output/users.csv"))
        .delimited()
        .delimiter(",")
        .names("firstName", "lastName", "email")
        .build();
}

// Custom writer
@Component
public class CustomItemWriter implements ItemWriter<User> {
    
    private final UserService userService;
    
    @Override
    public void write(Chunk<? extends User> chunk) throws Exception {
        for (User user : chunk) {
            userService.save(user);
        }
    }
}
```

### Multi-Step Job

```java
@Configuration
public class MultiStepJobConfig {
    
    @Bean
    public Job multiStepJob(JobRepository jobRepository) {
        return new JobBuilder("multiStepJob", jobRepository)
            .start(step1())
            .next(step2())
            .next(step3())
            .build();
    }
    
    @Bean
    public Step step1() {
        return new StepBuilder("step1", jobRepository)
            .<InputData, ProcessedData>chunk(100, transactionManager)
            .reader(inputReader())
            .processor(processor1())
            .writer(intermediateWriter())
            .build();
    }
    
    @Bean
    public Step step2() {
        return new StepBuilder("step2", jobRepository)
            .<ProcessedData, EnrichedData>chunk(100, transactionManager)
            .reader(intermediateReader())
            .processor(processor2())
            .writer(enrichedWriter())
            .build();
    }
    
    @Bean
    public Step step3() {
        return new StepBuilder("step3", jobRepository)
            .tasklet((contribution, chunkContext) -> {
                // Cleanup or notification step
                System.out.println("Job completed successfully!");
                return RepeatStatus.FINISHED;
            }, transactionManager)
            .build();
    }
}
```

### Conditional Flow

```java
@Bean
public Job conditionalJob(JobRepository jobRepository) {
    return new JobBuilder("conditionalJob", jobRepository)
        .start(step1())
            .on("COMPLETED").to(step2())
            .on("FAILED").to(errorStep())
        .from(step2())
            .on("*").to(step3())
        .end()
        .build();
}
```

### Job Parameters

```java
@Component
public class JobParametersReader implements ItemReader<User> {
    
    private final String inputFile;
    
    public JobParametersReader(@Value("#{jobParameters['inputFile']}") String inputFile) {
        this.inputFile = inputFile;
    }
    
    @Override
    public User read() {
        // Use inputFile parameter
        return null;
    }
}

// Launch job with parameters
JobParameters jobParameters = new JobParametersBuilder()
    .addString("inputFile", "users.csv")
    .addDate("date", new Date())
    .addLong("userId", 12345L)
    .toJobParameters();

jobLauncher.run(job, jobParameters);
```

### Job Scheduling

```java
@Configuration
@EnableScheduling
public class ScheduledJobConfig {
    
    private final JobLauncher jobLauncher;
    private final Job importUserJob;
    
    @Scheduled(cron = "0 0 2 * * ?")  // Every day at 2 AM
    public void performDailyImport() throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
            .addDate("date", new Date())
            .toJobParameters();
        
        jobLauncher.run(importUserJob, jobParameters);
    }
}
```

---

## 7.7 Spring Integration

Spring Integration implements Enterprise Integration Patterns.

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
```

### Message Channels

```java
@Configuration
public class IntegrationConfig {
    
    @Bean
    public MessageChannel inputChannel() {
        return new DirectChannel();
    }
    
    @Bean
    public MessageChannel outputChannel() {
        return new DirectChannel();
    }
    
    @Bean
    public MessageChannel errorChannel() {
        return new PublishSubscribeChannel();
    }
    
    @Bean
    public MessageChannel queueChannel() {
        return new QueueChannel(100);  // Buffered
    }
}
```

### Service Activator

```java
@Component
public class MessageProcessor {
    
    @ServiceActivator(inputChannel = "inputChannel", outputChannel = "outputChannel")
    public String processMessage(String message) {
        return message.toUpperCase();
    }
    
    @ServiceActivator(inputChannel = "orderChannel")
    public void handleOrder(Order order) {
        // Process order
        System.out.println("Processing order: " + order.getId());
    }
}
```

### Message Gateway

```java
@MessagingGateway
public interface OrderGateway {
    
    @Gateway(requestChannel = "orderInputChannel")
    void submitOrder(Order order);
    
    @Gateway(requestChannel = "orderProcessChannel", replyChannel = "orderReplyChannel")
    OrderResponse processOrder(Order order);
    
    @Gateway(requestChannel = "asyncOrderChannel")
    Future<OrderResponse> submitOrderAsync(Order order);
}

// Usage
@Service
public class OrderService {
    
    private final OrderGateway orderGateway;
    
    public void createOrder(Order order) {
        orderGateway.submitOrder(order);  // Fire and forget
    }
    
    public OrderResponse processOrder(Order order) {
        return orderGateway.processOrder(order);  // Request-reply
    }
}
```

### Integration Flow (DSL)

```java
@Configuration
public class IntegrationFlowConfig {
    
    @Bean
    public IntegrationFlow fileProcessingFlow() {
        return IntegrationFlows
            .from(Files.inboundAdapter(new File("input"))
                .patternFilter("*.csv")
                .autoCreateDirectory(true),
                e -> e.poller(Pollers.fixedDelay(5000)))
            .transform(File.class, file -> {
                // Read file content
                try {
                    return Files.readString(file.toPath());
                } catch (IOException ex) {
                    throw new RuntimeException(ex);
                }
            })
            .split(s -> s.delimiters("\n"))
            .transform(String.class, line -> parseLine(line))
            .filter(User.class, user -> user.isValid())
            .handle(userService, "saveUser")
            .get();
    }
    
    @Bean
    public IntegrationFlow orderProcessingFlow() {
        return IntegrationFlows
            .from("orderInputChannel")
            .enrich(e -> e
                .requestChannel("enrichmentChannel")
                .propertyExpression("customerInfo", "payload.customerId"))
            .transform(order -> validateOrder(order))
            .route(Order.class, order -> order.getType(),
                mapping -> mapping
                    .subFlowMapping("STANDARD", sf -> sf.handle("standardOrderHandler"))
                    .subFlowMapping("EXPRESS", sf -> sf.handle("expressOrderHandler"))
                    .subFlowMapping("BULK", sf -> sf.handle("bulkOrderHandler")))
            .aggregate(a -> a
                .correlationStrategy(m -> m.getHeaders().get("orderId"))
                .releaseStrategy(g -> g.size() == 10))
            .handle("orderOutputChannel")
            .get();
    }
}
```

---

## 7.8 Caching Strategies

Caching improves performance by storing frequently accessed data in memory.

### Spring Cache Abstraction

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
            new ConcurrentMapCache("users"),
            new ConcurrentMapCache("products")
        ));
        return cacheManager;
    }
}
```

### Basic Caching Annotations

```java
@Service
public class UserService {
    
    // Cache result
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        // Expensive database call
        System.out.println("Fetching from database: " + id);
        return userRepository.findById(id).orElseThrow();
    }
    
    // Cache with condition
    @Cacheable(value = "users", key = "#id", condition = "#id > 0")
    public User findByIdConditional(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
    
    // Cache unless result is null
    @Cacheable(value = "users", key = "#email", unless = "#result == null")
    public User findByEmail(String email) {
        return userRepository.findByEmail(email).orElse(null);
    }
    
    // Update cache
    @CachePut(value = "users", key = "#user.id")
    public User update(User user) {
        return userRepository.save(user);
    }
    
    // Evict cache entry
    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        userRepository.deleteById(id);
    }
    
    // Evict all entries
    @CacheEvict(value = "users", allEntries = true)
    public void deleteAll() {
        userRepository.deleteAll();
    }
    
    // Multiple cache operations
    @Caching(
        cacheable = @Cacheable(value = "users", key = "#id"),
        evict = @CacheEvict(value = "userStats", key = "#id")
    )
    public User findAndUpdateStats(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}
```

### Redis Cache

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: secret
    timeout: 2000ms
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
```

**Redis Configuration:**

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))  // Default TTL
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();
        
        // Per-cache configurations
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        
        // Users cache: 2 hour TTL
        cacheConfigurations.put("users", 
            config.entryTtl(Duration.ofHours(2)));
        
        // Products cache: 30 minute TTL
        cacheConfigurations.put("products",
            config.entryTtl(Duration.ofMinutes(30)));
        
        // Session cache: 15 minute TTL
        cacheConfigurations.put("sessions",
            config.entryTtl(Duration.ofMinutes(15)));
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // Serializers
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        
        return template;
    }
}
```

**Using RedisTemplate:**

```java
@Service
public class CacheService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // Simple operations
    public void set(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }
    
    public void setWithExpiry(String key, Object value, long timeout, TimeUnit unit) {
        redisTemplate.opsForValue().set(key, value, timeout, unit);
    }
    
    public Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }
    
    public void delete(String key) {
        redisTemplate.delete(key);
    }
    
    public Boolean exists(String key) {
        return redisTemplate.hasKey(key);
    }
    
    // Hash operations
    public void hashSet(String key, String field, Object value) {
        redisTemplate.opsForHash().put(key, field, value);
    }
    
    public Object hashGet(String key, String field) {
        return redisTemplate.opsForHash().get(key, field);
    }
    
    // List operations
    public void listPush(String key, Object value) {
        redisTemplate.opsForList().rightPush(key, value);
    }
    
    public Object listPop(String key) {
        return redisTemplate.opsForList().leftPop(key);
    }
    
    // Set operations
    public void setAdd(String key, Object... values) {
        redisTemplate.opsForSet().add(key, values);
    }
    
    public Set<Object> setMembers(String key) {
        return redisTemplate.opsForSet().members(key);
    }
    
    // Sorted set operations
    public void zAdd(String key, Object value, double score) {
        redisTemplate.opsForZSet().add(key, value, score);
    }
    
    public Set<Object> zRange(String key, long start, long end) {
        return redisTemplate.opsForZSet().range(key, start, end);
    }
}
```

### Caffeine Cache (In-Memory)

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**Configuration:**

```java
@Configuration
@EnableCaching
public class CaffeineCacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager("users", "products");
        cacheManager.setCaffeine(caffeineCacheBuilder());
        return cacheManager;
    }
    
    Caffeine<Object, Object> caffeineCacheBuilder() {
        return Caffeine.newBuilder()
            .initialCapacity(100)
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .recordStats();  // Enable statistics
    }
    
    // Custom cache with different configurations
    @Bean
    public Cache productCache() {
        return Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterAccess(5, TimeUnit.MINUTES)
            .refreshAfterWrite(1, TimeUnit.MINUTES)
            .build();
    }
}
```

**Manual Cache Usage:**

```java
@Service
public class ProductService {
    
    private final Cache<Long, Product> productCache;
    
    public ProductService() {
        this.productCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
    }
    
    public Product getProduct(Long id) {
        return productCache.get(id, key -> {
            // Compute if absent
            return productRepository.findById(key).orElse(null);
        });
    }
    
    public void updateProduct(Product product) {
        productRepository.save(product);
        productCache.put(product.getId(), product);
    }
    
    public void invalidate(Long id) {
        productCache.invalidate(id);
    }
}
```

### Reactive Caching

```java
@Service
public class ReactiveUserService {
    
    private final ReactiveRedisTemplate<String, User> redisTemplate;
    private final UserRepository userRepository;
    
    public Mono<User> findById(Long id) {
        String key = "user:" + id;
        
        return redisTemplate.opsForValue().get(key)
            .switchIfEmpty(
                userRepository.findById(id)
                    .flatMap(user -> 
                        redisTemplate.opsForValue()
                            .set(key, user, Duration.ofHours(1))
                            .thenReturn(user)
                    )
            );
    }
    
    public Mono<User> update(User user) {
        String key = "user:" + user.getId();
        
        return userRepository.save(user)
            .flatMap(savedUser ->
                redisTemplate.opsForValue()
                    .set(key, savedUser, Duration.ofHours(1))
                    .thenReturn(savedUser)
            );
    }
    
    public Mono<Void> delete(Long id) {
        String key = "user:" + id;
        
        return userRepository.deleteById(id)
            .then(redisTemplate.delete(key))
            .then();
    }
}
```

### Cache Patterns

**Cache-Aside (Lazy Loading):**

```java
public User getUserCacheAside(Long id) {
    // 1. Check cache first
    User user = cache.get(id);
    
    if (user == null) {
        // 2. Cache miss - load from database
        user = database.findById(id);
        
        if (user != null) {
            // 3. Populate cache
            cache.put(id, user);
        }
    }
    
    return user;
}
```

**Read-Through:**

```java
// Cache automatically loads from database on miss
@Cacheable(value = "users", key = "#id")
public User getUserReadThrough(Long id) {
    return database.findById(id);  // Called only on cache miss
}
```

**Write-Through:**

```java
@CachePut(value = "users", key = "#user.id")
public User updateUserWriteThrough(User user) {
    return database.save(user);  // Cache updated automatically
}
```

**Write-Behind (Write-Back):**

```java
public void updateUserWriteBehind(User user) {
    // 1. Update cache immediately
    cache.put(user.getId(), user);
    
    // 2. Async write to database
    CompletableFuture.runAsync(() -> {
        database.save(user);
    });
}
```

### Cache Monitoring

```java
@Component
public class CacheMetrics {
    
    private final CacheManager cacheManager;
    private final MeterRegistry meterRegistry;
    
    @PostConstruct
    public void bindCacheMetrics() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);
            
            if (cache instanceof CaffeineCache) {
                com.github.benmanes.caffeine.cache.Cache<Object, Object> nativeCache = 
                    ((CaffeineCache) cache).getNativeCache();
                
                // Register metrics
                Gauge.builder("cache.size", nativeCache, c -> c.estimatedSize())
                    .tag("cache", cacheName)
                    .register(meterRegistry);
                
                Gauge.builder("cache.hit.ratio", nativeCache, c -> c.stats().hitRate())
                    .tag("cache", cacheName)
                    .register(meterRegistry);
                
                Counter.builder("cache.evictions")
                    .tag("cache", cacheName)
                    .register(meterRegistry);
            }
        });
    }
    
    @Scheduled(fixedRate = 60000)  // Every minute
    public void logCacheStats() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);
            
            if (cache instanceof CaffeineCache) {
                com.github.benmanes.caffeine.cache.Cache<Object, Object> nativeCache = 
                    ((CaffeineCache) cache).getNativeCache();
                
                CacheStats stats = nativeCache.stats();
                
                log.info("Cache: {}, Size: {}, Hit Rate: {:.2f}%, Evictions: {}", 
                    cacheName,
                    nativeCache.estimatedSize(),
                    stats.hitRate() * 100,
                    stats.evictionCount()
                );
            }
        });
    }
}
```

---

## Summary and Key Takeaways

### Reactive Programming Revolution

**From Blocking to Non-Blocking:**
* **Traditional**: One thread per request, blocked during I/O
* **Reactive**: Few threads handle many requests, never block
* **Result**: 10-100x better resource utilization

**Core Concepts:**
* **Reactive Streams**: Publisher, Subscriber, Subscription
* **Backpressure**: Consumer controls rate
* **Mono**: 0 or 1 item
* **Flux**: 0 to N items

**When to Use Reactive:**
* High concurrency requirements (10,000+ concurrent users)
* I/O-intensive applications
* Event-driven systems
* Real-time data streaming
* Microservices with many external calls

**When NOT to Use:**
* Simple CRUD applications
* CPU-bound workloads
* Small user base
* Team unfamiliar with reactive

### Project Reactor Mastery

**Key Operators:**
* **Transformation**: map, flatMap, concatMap
* **Filtering**: filter, take, skip
* **Combining**: zip, merge, concat
* **Error Handling**: onErrorReturn, onErrorResume, retry
* **Side Effects**: doOnNext, doOnError, doFinally

**Threading:**
* **subscribeOn**: Upstream execution thread
* **publishOn**: Downstream execution thread
* **Schedulers**: immediate, single, parallel, boundedElastic

### Spring WebFlux Benefits

**Reactive REST APIs:**
* Annotation-based (familiar Spring MVC style)
* Functional endpoints (recommended for reactive)
* Server-Sent Events (SSE) support
* WebSocket support

**WebClient:**
* Reactive HTTP client
* Non-blocking I/O
* Composable API
* Better error handling

### R2DBC - Reactive Database Access

**Features:**
* Fully reactive database access
* No blocking database calls
* Reactive repositories
* Reactive transactions

**Limitations:**
* Fewer database drivers than JDBC
* Less mature than JPA
* Learning curve

### Spring Batch Excellence

**Batch Processing:**
* **Chunk-oriented**: Read → Process → Write
* **Tasklet**: Custom logic
* **Job flows**: Sequential, conditional, parallel

**Components:**
* **ItemReader**: Read from source
* **ItemProcessor**: Transform data
* **ItemWriter**: Write to destination

**Use Cases:**
* Data migration
* ETL processes
* Report generation
* Bulk operations

### Spring Integration Power

**Enterprise Integration Patterns:**
* **Channels**: Direct, queue, publish-subscribe
* **Gateways**: Hide messaging complexity
* **Service Activators**: Process messages
* **Integration Flows**: DSL for complex flows

**Benefits:**
* Decoupled systems
* Async processing
* Message transformation
* Error handling

### Caching Strategies

**Cache Types:**
* **Local**: Caffeine (in-memory, fast)
* **Distributed**: Redis (shared, scalable)
* **Hybrid**: Local + distributed (best of both)

**Patterns:**
* **Cache-Aside**: Manual cache management
* **Read-Through**: Automatic cache loading
* **Write-Through**: Synchronous cache update
* **Write-Behind**: Asynchronous cache update

**Best Practices:**
* Set appropriate TTL
* Monitor cache hit rates
* Handle cache failures gracefully
* Warm up cache on startup

### Production Checklist

**Reactive Applications:**
- [ ] Use reactive stack consistently (no blocking calls)
- [ ] Configure appropriate schedulers
- [ ] Implement backpressure handling
- [ ] Set timeouts on all external calls
- [ ] Monitor reactive metrics

**Batch Jobs:**
- [ ] Configure appropriate chunk size
- [ ] Implement error handling
- [ ] Schedule jobs appropriately
- [ ] Monitor job execution
- [ ] Set up job restart capability

**Caching:**
- [ ] Choose appropriate cache type
- [ ] Set TTL values
- [ ] Implement cache eviction strategy
- [ ] Monitor hit/miss rates
- [ ] Handle cache failures

**Integration:**
- [ ] Define message channels
- [ ] Implement error channels
- [ ] Configure retries
- [ ] Monitor message flows

### Frequently Asked Questions

**Q1: When should I use reactive programming vs traditional blocking?**

**A:**

**Use Reactive when:**
* High concurrency (10,000+ concurrent requests)
* I/O-intensive operations (many external API calls)
* Real-time data streaming
* Event-driven architecture
* Microservices with cascading calls

**Use Traditional when:**
* Simple CRUD applications
* Low concurrency requirements (<1,000 concurrent requests)
* CPU-bound operations
* Team unfamiliar with reactive
* Existing blocking libraries required

**Cost-benefit:**
* Reactive: Higher development complexity, better resource utilization
* Traditional: Simpler code, easier debugging, more memory/threads

---

**Q2: How do I migrate from Spring MVC to Spring WebFlux?**

**A:**

**Step-by-step migration:**

1. **Start with data layer**:
```java
// Before: JPA
UserRepository extends JpaRepository<User, Long>

// After: R2DBC
UserRepository extends ReactiveCrudRepository<User, Long>
```

2. **Update service layer**:
```java
// Before
public User findById(Long id) {
    return repository.findById(id).orElseThrow();
}

// After
public Mono<User> findById(Long id) {
    return repository.findById(id);
}
```

3. **Update controllers**:
```java
// Before
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}

// After
@GetMapping("/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```

4. **Replace RestTemplate with WebClient**:
```java
// Before
User user = restTemplate.getForObject(url, User.class);

// After
Mono<User> user = webClient.get().uri(url).retrieve().bodyToMono(User.class);
```

**⚠️ Critical:** Can't mix blocking and reactive - must convert entire stack.

---

**Q3: What caching strategy should I use?**

**A:**

| Use Case | Strategy | Technology |
|----------|----------|------------|
| Simple lookup | Cache-Aside | Caffeine |
| High read, low write | Read-Through | Redis |
| Write-heavy | Write-Behind | Redis |
| Distributed system | Distributed cache | Redis Cluster |
| Session storage | TTL-based | Redis |
| Query results | Query cache | Caffeine + @Cacheable |

**Example decision:**
```java
// Simple app: Caffeine (local)
@Cacheable("users")
public User findById(Long id)

// Microservices: Redis (distributed)
redisTemplate.opsForValue().set(key, user, Duration.ofHours(1))

// Hybrid: Local (Caffeine) + Distributed (Redis)
// - Check local first (fast)
// - Fall back to Redis (shared)
// - Populate local from Redis
```

### Interview Questions

**Question 1: Explain backpressure in reactive streams.**

**Difficulty:** Mid-Level

**Topics:** Reactive Programming

**Answer:**

**Backpressure** is a mechanism where the subscriber controls the rate of data flow from the publisher.

**Without backpressure:**
```
Publisher (fast) → → → → → → → Subscriber (slow)
                                  ↓
                              Overwhelmed
                              OutOfMemory
```

**With backpressure:**
```
Publisher                 Subscriber
   │                          │
   │  request(10)             │
   │◄─────────────────────────┤ "I can handle 10"
   │                          │
   │  onNext(1-10)            │
   ├─────────────────────────►│ Send exactly 10
   │                          │
   │  request(10)             │
   │◄─────────────────────────┤ "Ready for more"
```

**Implementation:**
```java
Flux.range(1, 1000)
    .onBackpressureBuffer(100)  // Buffer up to 100
    .subscribe(new Subscriber<>() {
        public void onSubscribe(Subscription s) {
            s.request(10);  // Request 10 items
        }
        
        public void onNext(Integer item) {
            process(item);
            subscription.request(1);  // Request next
        }
    });
```

**Strategies:**
* **Buffer**: Store items temporarily
* **Drop**: Drop items when overwhelmed
* **Latest**: Keep only latest item
* **Error**: Fail fast

---

**Question 2: Compare Mono and Flux.**

**Difficulty:** Entry-Level

**Topics:** Reactive Programming

**Answer:**

**Mono<T>**: 0 or 1 item

```java
// Single item
Mono<User> user = userRepository.findById(id);

// Empty
Mono<User> empty = Mono.empty();

// Error
Mono<User> error = Mono.error(new Exception());

// Use cases:
// - Database single row query
// - HTTP request expecting single response
// - Method returning optional result
```

**Flux<T>**: 0 to N items

```java
// Multiple items
Flux<User> users = userRepository.findAll();

// Range
Flux<Integer> numbers = Flux.range(1, 10);

// Infinite
Flux<Long> ticks = Flux.interval(Duration.ofSeconds(1));

// Use cases:
// - Database query returning multiple rows
// - Streaming data
// - Event streams
// - Collection processing
```

**Relationship:**
```java
// Flux to Mono (collect)
Mono<List<User>> listMono = flux.collectList();

// Mono to Flux (expand)
Flux<User> flux = mono.flux();
```

---

**End of Part 7: Advanced Spring & Reactive Programming**

### Congratulations on Completing Part 7!

You've mastered:
* ✅ Reactive programming fundamentals
* ✅ Project Reactor (Mono, Flux, operators)
* ✅ Spring WebFlux (reactive REST APIs)
* ✅ R2DBC (reactive database access)
* ✅ WebClient (reactive HTTP client)
* ✅ Spring Batch (batch processing)
* ✅ Spring Integration (enterprise patterns)
* ✅ Caching strategies (Redis, Caffeine)

**Complete Series Progress: 7 parts complete!**

### References

* [Project Reactor Documentation](https://projectreactor.io/docs)
* [Spring WebFlux Reference](https://docs.spring.io/spring-framework/reference/web/webflux.html)
* [R2DBC Documentation](https://r2dbc.io/)
* [Spring Batch Reference](https://docs.spring.io/spring-batch/docs/current/reference/html/)
* [Spring Integration Reference](https://docs.spring.io/spring-integration/reference/)
* [Reactive Programming with RxJava by Tomasz Nurkiewicz](https://www.oreilly.com/library/view/reactive-programming-with/9781491931646/)

---