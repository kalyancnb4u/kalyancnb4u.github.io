---
title: "Go Mastery - Supplement 5: Interview Flashcards (200 Questions)"
date: 2025-09-25 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Interview, Flash-cards, Quick-review]
---

# Complete Go Mastery - Interview Flashcards

## How to Use
- Review 10-20 cards daily
- Cover answer, try to recall
- Mark cards you struggle with
- Focus on weak areas

---

## Fundamentals (40 cards)

**Q1**: What is the zero value for different types in Go?
**A**: int=0, float=0.0, bool=false, string="", pointer/slice/map/chan/func=nil

**Q2**: Difference between array and slice?
**A**: Array is fixed size, value type. Slice is dynamic, reference type pointing to underlying array.

**Q3**: How do you create a slice with capacity?
**A**: `s := make([]int, length, capacity)` or `s := make([]int, 0, capacity)`

**Q4**: What happens when you append to a full slice?
**A**: Go allocates new underlying array (usually 2x size), copies elements, returns new slice.

**Q5**: How are maps implemented internally?
**A**: Hash table with buckets. Average O(1) lookup, O(n) worst case with collisions.

**Q6**: Are maps safe for concurrent access?
**A**: No. Need mutex or sync.Map for concurrent access.

**Q7**: Difference between `make` and `new`?
**A**: `new(T)` returns `*T` with zero value. `make(T)` only for slice/map/chan, returns initialized T.

**Q8**: What is a rune in Go?
**A**: int32 alias representing a Unicode code point. Used for character operations.

**Q9**: How do you iterate over a string properly?
**A**: Use `for i, r := range str` to get runes, not bytes.

**Q10**: What is struct embedding?
**A**: Composition mechanism. Embedded struct's fields/methods promoted to outer struct.

**Q11**: Value receiver vs pointer receiver?
**A**: Value creates copy (immutable), pointer allows modification. Pointer more efficient for large structs.

**Q12**: Can you have methods on non-struct types?
**A**: Yes, on any type defined in same package: `type MyInt int; func (m MyInt) Double() int`

**Q13**: What is an interface in Go?
**A**: Set of method signatures. Types implicitly satisfy interfaces (duck typing).

**Q14**: What is the empty interface?
**A**: `interface{}` or `any`. Satisfied by all types. Similar to Object in Java.

**Q15**: How do you check interface type at runtime?
**A**: Type assertion `v, ok := i.(Type)` or type switch.

**Q16**: What is defer?
**A**: Delays function call until surrounding function returns. LIFO order. Used for cleanup.

**Q17**: What is panic and recover?
**A**: Panic stops normal execution. Recover (in deferred func) catches panic, allows graceful handling.

**Q18**: Should you use panic for error handling?
**A**: No. Reserve for unrecoverable errors. Return errors instead.

**Q19**: What is the error interface?
**A**: `type error interface { Error() string }`. Any type with Error() method satisfies it.

**Q20**: How do you create custom errors?
**A**: `errors.New("msg")`, `fmt.Errorf("msg: %w", err)`, or custom type implementing error interface.

**Q21**: What is error wrapping?
**A**: fmt.Errorf with %w preserves original error. Use errors.Is/As to check wrapped errors.

**Q22**: What are variadic functions?
**A**: Functions accepting variable number of arguments: `func sum(nums ...int) int`

**Q23**: What is a closure?
**A**: Anonymous function that captures variables from enclosing scope.

**Q24**: Are function variables allowed?
**A**: Yes. Functions are first-class: `var f func(int) int = func(x int) int { return x*2 }`

**Q25**: What is init() function?
**A**: Special function executed before main(). Each package can have multiple init functions.

**Q26**: What is a blank identifier?
**A**: `_` discards values. Used for unused variables, imports for side effects, or ignoring returns.

**Q27**: What is short variable declaration?
**A**: `:=` declares and initializes. Only in function scope. At least one new variable required.

**Q28**: Can you redeclare variables with :=?
**A**: Yes, in same scope if at least one new variable is declared.

**Q29**: What is a struct tag?
**A**: Metadata in struct fields: `type User struct { Name string \`json:"name"\` }`. Used by encoding packages.

**Q30**: How do you export identifiers?
**A**: Capital first letter = exported. Lowercase = unexported (package-private).

**Q31**: What is a package in Go?
**A**: Collection of source files in same directory with same package statement. Unit of compilation.

**Q32**: Difference between package and module?
**A**: Package is directory of Go files. Module is collection of packages with go.mod file.

**Q33**: What is go.mod file?
**A**: Defines module path and dependencies. Created with `go mod init`.

**Q34**: How do you add a dependency?
**A**: `go get package@version` or just import and `go mod tidy`.

**Q35**: What is vendor directory?
**A**: Local copy of dependencies. Created with `go mod vendor`. Used for reproducible builds.

**Q36**: What are build tags?
**A**: Conditional compilation: `//go:build linux` at top of file.

**Q37**: What is CGO?
**A**: Allows calling C code from Go. Enabled with `import "C"` and cgo comments.

**Q38**: Should you use CGO?
**A**: Avoid if possible. Breaks cross-compilation, impacts performance, complicates builds.

**Q39**: What is go generate?
**A**: Runs commands to generate Go source. Directive: `//go:generate command args`

**Q40**: What is a type alias?
**A**: `type NewType = ExistingType`. Creates alternate name, not new type.

---

## Concurrency (40 cards)

**Q41**: What is a goroutine?
**A**: Lightweight thread managed by Go runtime. Starts with 2KB stack, multiplexed on OS threads.

**Q42**: How many goroutines can you create?
**A**: Millions (limited by memory). Much cheaper than OS threads.

**Q43**: How do you start a goroutine?
**A**: `go functionName()` or `go func() { ... }()`

**Q44**: What is GOMAXPROCS?
**A**: Max number of OS threads executing Go code. Default: number of CPU cores.

**Q45**: What is the Go scheduler?
**A**: M:N scheduler mapping M goroutines onto N OS threads. Uses work-stealing algorithm.

**Q46**: What is GMP model?
**A**: G=goroutine, M=machine (OS thread), P=processor (context for executing goroutines).

**Q47**: What is a channel?
**A**: Typed conduit for communication between goroutines. Thread-safe by design.

**Q48**: Buffered vs unbuffered channel?
**A**: Unbuffered blocks until receiver ready. Buffered blocks only when full.

**Q49**: How do you create a channel?
**A**: `ch := make(chan Type)` (unbuffered) or `ch := make(chan Type, capacity)` (buffered)

**Q50**: How do you send to a channel?
**A**: `ch <- value`

**Q51**: How do you receive from a channel?
**A**: `value := <-ch` or `value, ok := <-ch` (ok false if closed)

**Q52**: How do you close a channel?
**A**: `close(ch)`. Only sender should close. Closing again panics.

**Q53**: Can you iterate over a channel?
**A**: Yes. `for value := range ch { ... }` receives until closed.

**Q54**: What is select statement?
**A**: Waits on multiple channel operations. Chooses random case if multiple ready.

**Q55**: What is default case in select?
**A**: Executes immediately if no channel ready. Enables non-blocking operations.

**Q56**: How do you implement timeout with select?
**A**: `case <-time.After(duration):` or use context with deadline.

**Q57**: What is sync.WaitGroup?
**A**: Counter for waiting on goroutines. Add(), Done(), Wait() methods.

**Q58**: What is sync.Mutex?
**A**: Mutual exclusion lock. Lock(), Unlock() methods. Protects critical sections.

**Q59**: What is sync.RWMutex?
**A**: Read-write mutex. Multiple readers OR one writer. RLock/RUnlock for readers.

**Q60**: When to use Mutex vs RWMutex?
**A**: RWMutex when reads >> writes. Mutex simpler, lower overhead for write-heavy.

**Q61**: What is sync.Once?
**A**: Ensures function executes exactly once. Thread-safe lazy initialization.

**Q62**: What is atomic package?
**A**: Low-level atomic operations on integers. Faster than mutex for simple counters.

**Q63**: What is sync.Map?
**A**: Concurrent map. Optimized for two cases: (1) entry written once, read many times (2) disjoint key sets.

**Q64**: What is a race condition?
**A**: Multiple goroutines access shared data, at least one writes, without synchronization.

**Q65**: How do you detect races?
**A**: Run tests with `-race` flag: `go test -race`

**Q66**: What is context package for?
**A**: Carries deadlines, cancellation signals, request-scoped values across API boundaries.

**Q67**: Types of context?
**A**: Background(), TODO(), WithCancel(), WithDeadline(), WithTimeout(), WithValue()

**Q68**: Should you store context in struct?
**A**: No. Pass as first parameter to functions. Exception: existing interfaces can't change.

**Q69**: What is context.WithValue for?
**A**: Request-scoped values (request ID, user). Don't use for function parameters.

**Q70**: How do you implement worker pool?
**A**: Fixed number of goroutines reading from job channel, sending to results channel.

**Q71**: What is fan-out, fan-in?
**A**: Fan-out: distribute work to multiple goroutines. Fan-in: combine results from multiple sources.

**Q72**: What is pipeline pattern?
**A**: Chain of stages connected by channels. Each stage runs in goroutines.

**Q73**: What causes goroutine leak?
**A**: Goroutine blocked forever on channel or waiting on condition that never happens.

**Q74**: How do you prevent goroutine leaks?
**A**: Use context for cancellation, buffered channels, or ensure all send/receive operations can complete.

**Q75**: What is sync.Cond?
**A**: Condition variable for goroutines waiting for/announcing event. Rarely needed in modern Go.

**Q76**: What is sync.Pool?
**A**: Cache of temporary objects. Reduces allocation overhead. Items may be removed anytime by GC.

**Q77**: What is done channel pattern?
**A**: Signal multiple goroutines to stop: `close(done)`, goroutines select on `<-done`.

**Q78**: What is or-channel pattern?
**A**: Combine multiple done channels: return first one that closes.

**Q79**: What is the difference between concurrency and parallelism?
**A**: Concurrency: dealing with many things at once (structure). Parallelism: doing many things at once (execution).

**Q80**: Can goroutines run in parallel?
**A**: Yes, if GOMAXPROCS > 1 and multiple CPU cores available.

---

## Testing & Performance (30 cards)

**Q81**: How do you write a unit test?
**A**: Function named TestXxx(t *testing.T) in _test.go file.

**Q82**: How do you run tests?
**A**: `go test` (current package) or `go test ./...` (all packages)

**Q83**: What is table-driven testing?
**A**: Test cases in slice of structs, iterate with t.Run for each case.

**Q84**: What is t.Run for?
**A**: Subtests. Enables parallel execution, better error reporting, selective running.

**Q85**: How do you mark test as parallel?
**A**: Call `t.Parallel()` at beginning of test. Max parallel tests = GOMAXPROCS.

**Q86**: What is test coverage?
**A**: Percentage of code executed by tests. Run with `go test -cover`.

**Q87**: How do you generate coverage report?
**A**: `go test -coverprofile=cover.out`, then `go tool cover -html=cover.out`

**Q88**: What is a benchmark?
**A**: Function named BenchmarkXxx(b *testing.B) that runs code b.N times.

**Q89**: How do you run benchmarks?
**A**: `go test -bench=.` (all benchmarks) or `go test -bench=BenchmarkName`

**Q90**: What does b.N represent?
**A**: Number of iterations. Adjusted by testing framework to get reliable timing.

**Q91**: What is b.ReportAllocs?
**A**: Include allocation statistics in benchmark output.

**Q92**: What is b.ResetTimer for?
**A**: Reset timer before benchmark loop to exclude setup time.

**Q93**: What is benchstat?
**A**: Tool to compare benchmark results statistically: `benchstat old.txt new.txt`

**Q94**: What is pprof?
**A**: Profiler for Go programs. Analyzes CPU, memory, goroutines, etc.

**Q95**: How do you generate CPU profile?
**A**: `go test -cpuprofile=cpu.prof -bench=.` then `go tool pprof cpu.prof`

**Q96**: How do you generate memory profile?
**A**: `go test -memprofile=mem.prof -bench=.` then `go tool pprof mem.prof`

**Q97**: What is escape analysis?
**A**: Compiler determines if variable can live on stack or must "escape" to heap.

**Q98**: How do you check escape analysis?
**A**: `go build -gcflags="-m"` shows escape decisions.

**Q99**: What is inlining?
**A**: Compiler replaces function call with function body. Reduces call overhead.

**Q100**: How do you prevent inlining?
**A**: `//go:noinline` directive above function.

**Q101**: What is go vet?
**A**: Static analysis tool finding suspicious code. Automatically run by go test.

**Q102**: What is golangci-lint?
**A**: Meta-linter running many linters in parallel. Industry standard for Go.

**Q103**: What is a mock?
**A**: Test double implementing interface for testing. Created manually or with gomock.

**Q104**: What is testify?
**A**: Popular testing library with assertions, mocks, suites: github.com/stretchr/testify

**Q105**: What is httptest?
**A**: Standard library package for testing HTTP servers and clients.

**Q106**: How do you test HTTP handlers?
**A**: `httptest.NewRecorder()` for ResponseWriter, `httptest.NewRequest()` for Request.

**Q107**: What is golden file testing?
**A**: Compare output to expected output stored in file. Good for complex outputs.

**Q108**: What is fuzzing in Go?
**A**: Automated testing with random inputs. Native support in Go 1.18+.

**Q109**: What causes "all goroutines are asleep - deadlock"?
**A**: All goroutines blocked waiting on channels with no sender/receiver to unblock them.

**Q110**: What is the race detector?
**A**: Tool to detect data races. Run with `-race` flag.

---

## Web & APIs (30 cards)

**Q111**: What is net/http?
**A**: Standard library HTTP client and server. Production-ready without third-party frameworks.

**Q112**: What is http.Handler?
**A**: Interface with ServeHTTP(ResponseWriter, *Request) method. Core of Go HTTP.

**Q113**: What is http.HandlerFunc?
**A**: Function type implementing Handler. Allows functions to be handlers.

**Q114**: How do you create HTTP server?
**A**: `http.ListenAndServe(":8080", handler)` or configure http.Server{}.

**Q115**: What is http.ServeMux?
**A**: HTTP request router/multiplexer. Maps URLs to handlers.

**Q116**: What is middleware in Go?
**A**: Function taking handler, returning new handler: `func(http.Handler) http.Handler`

**Q117**: How do you chain middleware?
**A**: Wrap handlers: `logger(recovery(auth(handler)))` or use middleware library.

**Q118**: How do you read JSON request body?
**A**: `json.NewDecoder(r.Body).Decode(&data)` (streaming) or `json.Unmarshal(body, &data)`

**Q119**: How do you write JSON response?
**A**: `json.NewEncoder(w).Encode(data)` or `json.Marshal(data)` then `w.Write()`

**Q120**: What is context in HTTP requests?
**A**: `r.Context()` provides request-scoped context for cancellation and deadlines.

**Q121**: How do you extract URL parameters?
**A**: `r.URL.Query().Get("param")` for query params. Use router for path params.

**Q122**: What is the difference between GET and POST?
**A**: GET retrieves, idempotent, cacheable. POST creates/modifies, not idempotent.

**Q123**: What HTTP status codes should you know?
**A**: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error

**Q124**: What is CORS?
**A**: Cross-Origin Resource Sharing. Browser security preventing cross-origin requests. Set headers to allow.

**Q125**: What is CSRF?
**A**: Cross-Site Request Forgery. Attack using authenticated user's credentials. Prevent with tokens.

**Q126**: How do you implement graceful shutdown?
**A**: Catch signal, call server.Shutdown(ctx) with timeout, wait for in-flight requests.

**Q127**: What is gRPC?
**A**: High-performance RPC framework using Protocol Buffers. HTTP/2-based.

**Q128**: Protocol Buffers vs JSON?
**A**: Protobuf: binary, smaller, faster, strongly-typed. JSON: text, human-readable, flexible.

**Q129**: What is database/sql package?
**A**: Generic interface for SQL databases. Use with driver (lib/pq, go-sql-driver/mysql).

**Q130**: How do you prevent SQL injection?
**A**: Use parameterized queries: `db.Query("SELECT * FROM users WHERE id = $1", userID)`

**Q131**: What is a prepared statement?
**A**: Pre-compiled SQL statement. Efficient for repeated queries, prevents injection.

**Q132**: What is a transaction?
**A**: Group of database operations executed as unit. All succeed or all fail.

**Q133**: How do you use transactions?
**A**: `tx := db.Begin()`, execute operations, `tx.Commit()` or `tx.Rollback()`

**Q134**: What is connection pooling?
**A**: Reuse database connections. Configure with SetMaxOpenConns, SetMaxIdleConns.

**Q135**: What is an ORM?
**A**: Object-Relational Mapping. Maps structs to tables. Examples: GORM, ent.

**Q136**: Should you use ORM in Go?
**A**: Depends. Good for rapid development. Raw SQL better for complex queries, performance.

**Q137**: What is JWT?
**A**: JSON Web Token. Stateless auth token with header, payload, signature.

**Q138**: How do you implement JWT auth?
**A**: Sign token on login, verify signature and claims on protected endpoints.

**Q139**: Where to store JWT?
**A**: Client: localStorage (vulnerable to XSS) or httpOnly cookie (vulnerable to CSRF). Defend accordingly.

**Q140**: What is bcrypt for?
**A**: Hashing passwords. Slow by design (mitigates brute force). Use bcrypt.GenerateFromPassword().

---

## Architecture & Design (30 cards)

**Q141**: What is Clean Architecture?
**A**: Layered architecture with dependency rule: inner layers don't depend on outer.

**Q142**: What are the layers in Clean Architecture?
**A**: Entities (domain), Use Cases (app logic), Interface Adapters (controllers), Frameworks (DB, web).

**Q143**: What is dependency injection?
**A**: Providing dependencies from outside rather than creating internally. Enables testing, flexibility.

**Q144**: What is the repository pattern?
**A**: Abstraction over data access. Interface in domain, implementation in infrastructure.

**Q145**: What is Domain-Driven Design?
**A**: Modeling software based on business domain. Entities, value objects, aggregates, bounded contexts.

**Q146**: What is a microservice?
**A**: Independently deployable service owning its data. Communicates via APIs (REST, gRPC).

**Q147**: Microservices vs monolith?
**A**: Microservices: scalable, complex, operational overhead. Monolith: simple, easier to develop, less scalable.

**Q148**: What is service discovery?
**A**: Mechanism for services to find each other. Solutions: Consul, etcd, Kubernetes DNS.

**Q149**: What is a circuit breaker?
**A**: Pattern preventing cascading failures. Opens after N failures, rejects requests, half-opens to test recovery.

**Q150**: What is rate limiting?
**A**: Restricting number of requests. Algorithms: token bucket, leaky bucket, fixed/sliding window.

**Q151**: What is caching?
**A**: Storing results for reuse. Strategies: cache-aside, read-through, write-through, write-back.

**Q152**: What is cache invalidation?
**A**: Removing stale data from cache. Strategies: TTL, event-based, manual.

**Q153**: What is idempotency?
**A**: Operation produces same result regardless of repetition. Important for retries, distributed systems.

**Q154**: What is eventual consistency?
**A**: Distributed system model where updates propagate asynchronously. Eventually all nodes agree.

**Q155**: What is CAP theorem?
**A**: Can't have Consistency, Availability, Partition tolerance simultaneously. Choose 2.

**Q156**: What is message queue?
**A**: Asynchronous communication between services. Examples: RabbitMQ, Kafka, SQS.

**Q157**: What is pub/sub?
**A**: Pattern where publishers send to topics, subscribers receive. Decouples producers/consumers.

**Q158**: What is event sourcing?
**A**: Store state changes as events, not current state. Enables audit trail, time travel.

**Q159**: What is CQRS?
**A**: Command Query Responsibility Segregation. Separate models for read and write operations.

**Q160**: What is database sharding?
**A**: Horizontal partitioning of database across servers. Improves scalability.

**Q161**: What is load balancing?
**A**: Distributing requests across multiple servers. Algorithms: round-robin, least connections, hash-based.

**Q162**: What is horizontal vs vertical scaling?
**A**: Horizontal: add more machines. Vertical: add more resources to machine. Horizontal better for distributed.

**Q163**: What is blue-green deployment?
**A**: Two identical environments. Deploy to one, switch traffic after validation. Easy rollback.

**Q164**: What is canary deployment?
**A**: Deploy to small subset of users first. Gradually increase if metrics good.

**Q165**: What is observability?
**A**: Understanding system state from external outputs. Three pillars: logs, metrics, traces.

**Q166**: What is the difference between monitoring and observability?
**A**: Monitoring answers "is it broken?". Observability answers "why is it broken?".

**Q167**: What is distributed tracing?
**A**: Tracking requests across multiple services. Tools: Jaeger, Zipkin, OpenTelemetry.

**Q168**: What is a service mesh?
**A**: Infrastructure layer for service-to-service communication. Examples: Istio, Linkerd.

**Q169**: What is API gateway?
**A**: Entry point for APIs. Handles routing, auth, rate limiting, transformation.

**Q170**: What is idempotency key?
**A**: Client-generated unique key for request. Server uses to detect retries, ensure exactly-once processing.

---

## Security (15 cards)

**Q171**: What is input validation?
**A**: Checking user input meets expectations. Prevent injection attacks, ensure data integrity.

**Q172**: What is XSS?
**A**: Cross-Site Scripting. Injecting malicious scripts. Prevent with output encoding, CSP headers.

**Q173**: How do you prevent XSS in Go?
**A**: Use html/template for auto-escaping. Set Content-Security-Policy header.

**Q174**: What is CSRF?
**A**: Cross-Site Request Forgery. Unauthorized commands from authenticated user. Prevent with tokens, SameSite cookies.

**Q175**: What is SQL injection?
**A**: Injecting malicious SQL. Prevent with parameterized queries, never concatenate user input.

**Q176**: What is command injection?
**A**: Executing arbitrary commands. Prevent by validating input, using library functions over shell commands.

**Q177**: What is authentication vs authorization?
**A**: Authentication: verifying identity. Authorization: verifying permissions.

**Q178**: What is HTTPS/TLS?
**A**: Encrypted HTTP over SSL/TLS. Prevents eavesdropping, tampering. Always use in production.

**Q179**: What are security headers?
**A**: HTTP headers preventing attacks: X-Frame-Options, X-Content-Type-Options, Strict-Transport-Security, Content-Security-Policy.

**Q180**: What is OWASP Top 10?
**A**: List of most critical web security risks. Study: injection, broken auth, sensitive data exposure, XSS, broken access control, etc.

**Q181**: How do you store passwords?
**A**: Never plain text. Hash with bcrypt, scrypt, or argon2. Salt automatically included in bcrypt.

**Q182**: What is salting passwords?
**A**: Adding random data before hashing. Prevents rainbow table attacks. bcrypt does this automatically.

**Q183**: What is encryption at rest vs in transit?
**A**: At rest: encrypting stored data. In transit: encrypting data being transmitted (HTTPS).

**Q184**: What is the principle of least privilege?
**A**: Grant minimum necessary permissions. Applies to users, services, database access, etc.

**Q185**: How do you handle secrets in Go?
**A**: Environment variables, secret management systems (Vault, AWS Secrets Manager). Never hardcode.

---

## System Design (15 cards)

**Q186**: How do you design a URL shortener?
**A**: Hash function for IDs, database (id, long_url), cache for reads, handle collisions.

**Q187**: How do you design a rate limiter?
**A**: Token bucket or sliding window algorithm, Redis for distributed counting, per-user/IP limits.

**Q188**: How do you design a chat system?
**A**: WebSockets for real-time, message queue for reliability, database for history, presence service.

**Q189**: How do you design a file upload service?
**A**: Chunked uploads for large files, object storage (S3), content-addressable deduplication, metadata DB.

**Q190**: How do you design a notification system?
**A**: Priority queues, multiple channels (email, SMS, push), retry logic, template engine.

**Q191**: How do you design a search system?
**A**: Elasticsearch for full-text search, index updates via queue, caching for popular queries.

**Q192**: How do you scale a database?
**A**: Read replicas, write sharding, caching layer, connection pooling, denormalization for reads.

**Q193**: How do you handle high traffic?
**A**: Load balancing, caching (CDN, Redis), async processing, database optimization, horizontal scaling.

**Q194**: How do you ensure data consistency?
**A**: Transactions, distributed locks, saga pattern, event sourcing, eventual consistency where acceptable.

**Q195**: How do you design for high availability?
**A**: Redundancy (multi-AZ, multi-region), health checks, auto-recovery, circuit breakers, graceful degradation.

**Q196**: What is the trade-off between consistency and availability?
**A**: CAP theorem. Strong consistency requires coordination (impacts availability). Eventual consistency allows higher availability.

**Q197**: How do you handle failures in distributed systems?
**A**: Retries with exponential backoff, circuit breakers, timeout, fallback, idempotency.

**Q198**: How do you monitor distributed systems?
**A**: Distributed tracing, centralized logging, metrics aggregation, alerting, SLOs/SLIs.

**Q199**: What is the difference between latency and throughput?
**A**: Latency: time per request. Throughput: requests per unit time. Sometimes inversely related.

**Q200**: How do you estimate system scale?
**A**: DAU/MAU, peak QPS, storage (reads + writes), bandwidth. Always over-provision by 50-100%.

---

## Daily Practice Schedule

**Week 1**: Cards 1-40 (Fundamentals)
**Week 2**: Cards 41-80 (Concurrency)
**Week 3**: Cards 81-110 (Testing & Performance)
**Week 4**: Cards 111-140 (Web & APIs)
**Week 5**: Cards 141-170 (Architecture)
**Week 6**: Cards 171-200 (Security & System Design)
**Week 7-8**: Review all 200, focus on missed cards

**Daily Routine**:
- Morning: Review 20 cards
- Evening: Practice coding based on concepts
- Mark difficult cards for extra review

---

**Print these cards or create physical flashcards. Repetition is key to retention!**
