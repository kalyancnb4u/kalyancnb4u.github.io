---
title: "Go Mastery - Supplement 1: Quick Reference Cheat Sheet"
date: 2025-09-21 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Reference, Cheat-sheet, Quick-reference, Summary]
---

# Complete Go Mastery - Quick Reference Cheat Sheet

## Go Fundamentals

### Basic Syntax
```go
// Variables
var name string = "Go"
age := 25                    // Short declaration
const PI = 3.14159

// Types
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
float32, float64
bool, string, byte, rune

// Arrays & Slices
arr := [3]int{1, 2, 3}       // Array (fixed)
slice := []int{1, 2, 3}      // Slice (dynamic)
slice = append(slice, 4)     // Add element

// Maps
m := make(map[string]int)
m["key"] = 42
delete(m, "key")

// Structs
type Person struct {
    Name string
    Age  int
}
p := Person{"Alice", 30}
```

### Control Flow
```go
// If-else
if x > 0 {
    // ...
} else if x < 0 {
    // ...
} else {
    // ...
}

// For loops
for i := 0; i < 10; i++ {  // Classic
    // ...
}
for _, v := range slice {   // Range
    // ...
}
for {                       // Infinite
    break
}

// Switch
switch val {
case 1:
    // ...
case 2, 3:
    // ...
default:
    // ...
}
```

### Functions
```go
// Basic function
func add(a, b int) int {
    return a + b
}

// Multiple returns
func swap(a, b string) (string, string) {
    return b, a
}

// Named returns
func divide(a, b float64) (result float64, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return
    }
    result = a / b
    return
}

// Variadic
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// Defer
defer file.Close()  // Executes when function returns
```

## Concurrency

### Goroutines
```go
// Start goroutine
go func() {
    // Runs concurrently
}()

// Wait for completion
var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    // Work
}()
wg.Wait()
```

### Channels
```go
// Unbuffered
ch := make(chan int)
ch <- 42        // Send (blocks)
val := <-ch     // Receive (blocks)

// Buffered
ch := make(chan int, 3)
ch <- 1         // Doesn't block until full

// Select
select {
case val := <-ch1:
    // Received from ch1
case ch2 <- val:
    // Sent to ch2
case <-time.After(time.Second):
    // Timeout
default:
    // Non-blocking
}

// Close
close(ch)
for val := range ch {  // Receive until closed
    // ...
}
```

### Sync Primitives
```go
// Mutex
var mu sync.Mutex
mu.Lock()
// Critical section
mu.Unlock()

// RWMutex
var rwmu sync.RWMutex
rwmu.RLock()   // Multiple readers
rwmu.RUnlock()
rwmu.Lock()    // Single writer
rwmu.Unlock()

// Once
var once sync.Once
once.Do(func() {
    // Runs only once
})

// Atomic
var counter atomic.Int64
counter.Add(1)
val := counter.Load()
```

### Context
```go
// With timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// With cancel
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

// With value
ctx := context.WithValue(ctx, "key", "value")
val := ctx.Value("key").(string)

// Check cancellation
select {
case <-ctx.Done():
    return ctx.Err()
default:
    // Continue
}
```

## Error Handling

### Basic Errors
```go
// Create error
err := errors.New("something went wrong")
err := fmt.Errorf("failed: %w", originalErr)

// Check error
if err != nil {
    return err
}

// Type assertion
if pathErr, ok := err.(*os.PathError); ok {
    fmt.Println(pathErr.Path)
}

// Unwrap
errors.Unwrap(err)
errors.Is(err, os.ErrNotExist)
errors.As(err, &target)
```

### Custom Errors
```go
type MyError struct {
    Code int
    Msg  string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("error %d: %s", e.Code, e.Msg)
}
```

## Interfaces

### Definition & Usage
```go
// Interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Implicit implementation
type File struct {}
func (f *File) Read(p []byte) (int, error) {
    // ...
}

// Empty interface
var v interface{} = 42
var v any = 42  // Go 1.18+

// Type assertion
s := v.(string)
s, ok := v.(string)

// Type switch
switch v := v.(type) {
case int:
    // v is int
case string:
    // v is string
}
```

## Testing

### Unit Tests
```go
// test_file.go
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}

// Table-driven
func TestAdd(t *testing.T) {
    tests := []struct{
        name string
        a, b int
        want int
    }{
        {"positive", 2, 3, 5},
        {"negative", -1, 1, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("got %d, want %d", got, tt.want)
            }
        })
    }
}
```

### Benchmarks
```go
func BenchmarkAdd(b *testing.B) {
    b.ReportAllocs()
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

// Run: go test -bench=. -benchmem
```

## HTTP Server

### Basic Server
```go
import "net/http"

func handler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello, World!"))
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

### Middleware
```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Println(r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}
```

### JSON
```go
// Encode
json.NewEncoder(w).Encode(data)

// Decode
var data MyStruct
json.NewDecoder(r.Body).Decode(&data)
```

## Database

### SQL
```go
import "database/sql"
import _ "github.com/lib/pq"

db, _ := sql.Open("postgres", "connection-string")

// Query
rows, _ := db.Query("SELECT id, name FROM users WHERE age > $1", 18)
defer rows.Close()
for rows.Next() {
    var id int
    var name string
    rows.Scan(&id, &name)
}

// Execute
result, _ := db.Exec("INSERT INTO users (name, age) VALUES ($1, $2)", "Alice", 30)
```

## Common Patterns

### Worker Pool
```go
jobs := make(chan int, 100)
results := make(chan int, 100)

// Workers
for w := 0; w < 3; w++ {
    go func() {
        for job := range jobs {
            results <- process(job)
        }
    }()
}

// Submit jobs
for i := 0; i < 100; i++ {
    jobs <- i
}
close(jobs)
```

### Rate Limiting
```go
import "golang.org/x/time/rate"

limiter := rate.NewLimiter(10, 20)  // 10 req/sec, burst 20
if limiter.Allow() {
    // Process request
}
```

### Graceful Shutdown
```go
srv := &http.Server{Addr: ":8080"}

go srv.ListenAndServe()

quit := make(chan os.Signal, 1)
signal.Notify(quit, os.Interrupt)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
srv.Shutdown(ctx)
```

## Performance

### Profiling
```bash
# CPU profile
go test -cpuprofile=cpu.prof -bench=.
go tool pprof cpu.prof

# Memory profile
go test -memprofile=mem.prof -bench=.
go tool pprof mem.prof

# Production profiling
import _ "net/http/pprof"
go http.ListenAndServe("localhost:6060", nil)
```

### Optimization Tips
```go
// Pre-allocate slices
s := make([]int, 0, 100)

// Use sync.Pool for reusable objects
pool := &sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}
buf := pool.Get().(*bytes.Buffer)
defer pool.Put(buf)

// Use strings.Builder
var sb strings.Builder
sb.WriteString("hello")
result := sb.String()
```

## Security

### Input Validation
```go
// Parameterized queries (SQL injection prevention)
db.Query("SELECT * FROM users WHERE id = $1", userID)

// HTML escaping (XSS prevention)
import "html/template"
tmpl.Execute(w, data)  // Auto-escapes

// URL validation
u, err := url.Parse(userInput)
if err != nil || u.Scheme != "https" {
    return errors.New("invalid URL")
}
```

### Authentication
```go
// Password hashing
import "golang.org/x/crypto/bcrypt"

hash, _ := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
err := bcrypt.CompareHashAndPassword(hash, []byte(password))
```

### JWT
```go
import "github.com/golang-jwt/jwt/v5"

token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
tokenString, _ := token.SignedString(secretKey)

token, _ := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
    return secretKey, nil
})
```

## Go Commands

```bash
# Build
go build                # Build binary
go build -o app         # Named binary
go install              # Install to $GOPATH/bin

# Run
go run main.go          # Compile and run
go run .                # Run package

# Test
go test                 # Run tests
go test -v              # Verbose
go test -cover          # Coverage
go test -bench=.        # Benchmarks

# Modules
go mod init <module>    # Initialize module
go mod tidy             # Clean dependencies
go mod download         # Download dependencies
go get <package>        # Add dependency

# Format & Lint
gofmt -w .              # Format code
go vet                  # Static analysis
golangci-lint run       # Comprehensive linting
```

## Best Practices

### Code Organization
```
project/
├── cmd/                # Main applications
│   └── api/
│       └── main.go
├── internal/           # Private packages
│   ├── domain/
│   ├── usecase/
│   └── delivery/
├── pkg/                # Public packages
├── go.mod
└── go.sum
```

### Error Handling
```go
// ✅ Return errors, don't panic
func process() error {
    if err != nil {
        return fmt.Errorf("process failed: %w", err)
    }
    return nil
}

// ✅ Check all errors
if err := doSomething(); err != nil {
    return err
}
```

### Concurrency
```go
// ✅ Use channels to communicate
// ✅ Use sync primitives to protect state
// ✅ Always close channels (sender side)
// ✅ Use context for cancellation
```

## Quick Complexity Reference

| Operation | Time | Space |
|-----------|------|-------|
| Array access | O(1) | O(1) |
| Slice append | O(1)* | O(n)** |
| Map access | O(1)* | O(1) |
| Map insert | O(1)* | O(1) |
| Channel send/recv | O(1) | O(1) |

*Amortized, **When growing

## Interview Tips

### Coding Interview
1. Clarify requirements
2. Think out loud
3. Start with brute force
4. Optimize
5. Test with examples
6. Discuss complexity

### System Design
1. Clarify (5 min)
2. Estimate scale (5 min)
3. High-level design (10 min)
4. Deep dive (15 min)
5. Trade-offs (10 min)

### Behavioral
Use STAR method:
- **S**ituation
- **T**ask
- **A**ction
- **R**esult

## Resources

- **Official**: golang.org
- **Tour**: tour.golang.org
- **Playground**: play.golang.org
- **Packages**: pkg.go.dev
- **Blog**: blog.golang.org

---

**Keep this cheat sheet handy for quick reference during coding, interviews, and daily development!**
