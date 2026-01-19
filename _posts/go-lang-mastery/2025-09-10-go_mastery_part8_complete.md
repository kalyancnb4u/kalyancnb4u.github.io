---
title: "Go Mastery - Part 8: Performance Optimization & Profiling"
date: 2025-09-10 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Performance, Optimization, Profiling, Benchmarking, Memory, CPU, Scalability]
---

# Complete Go Mastery Part 8: Performance Optimization & Profiling

## Introduction

Welcome to Part 8 of the Complete Go Mastery series. In Parts 1-7, we covered Go fundamentals through deployment. Now we focus on **performance optimization**—making your Go applications fast, efficient, and scalable.

Go is already fast. But "fast enough" depends on your requirements. A web API serving millions of requests per second has different needs than a batch processing system. Understanding performance optimization helps you make informed decisions about when to optimize, what to optimize, and how to measure success.

**Why performance optimization matters:**

- **User experience**: Fast applications delight users
- **Cost efficiency**: Less CPU/memory = lower cloud bills
- **Scalability**: Handle more load with same resources
- **Competitive advantage**: Performance can be a differentiator
- **Resource conservation**: Environmental and economic benefits

**What you'll learn in Part 8:**

- **Performance Fundamentals**: Big O, profiling, benchmarking
- **CPU Optimization**: Hot path optimization, algorithm selection
- **Memory Optimization**: Allocation reduction, GC tuning
- **Concurrency Optimization**: Goroutine pools, channel patterns
- **I/O Optimization**: Buffering, batching, caching
- **Database Optimization**: Query optimization, connection pooling
- **Network Optimization**: HTTP/2, gRPC, compression
- **Compiler Optimizations**: Inlining, escape analysis
- **Production Profiling**: Live profiling without disruption
- **Performance Patterns**: Common optimization strategies

**The Golden Rule of Optimization:**

> "Premature optimization is the root of all evil" - Donald Knuth

Always:
1. **Measure first** - Profile before optimizing
2. **Focus on bottlenecks** - 80/20 rule applies
3. **Verify improvements** - Benchmark before and after
4. **Maintain readability** - Don't sacrifice maintainability

By the end of Part 8, you'll know how to identify performance bottlenecks and optimize Go applications systematically.

---

## 8.1 Performance Fundamentals

### Understanding Performance Metrics

**Latency** - Time to complete an operation
```go
start := time.Now()
result := doWork()
latency := time.Since(start)
// Target: p99 < 100ms
```

**Throughput** - Operations per unit time
```go
operations := 0
start := time.Now()
for time.Since(start) < time.Minute {
    doWork()
    operations++
}
throughput := float64(operations) / 60.0 // ops/sec
// Target: 10,000 req/sec
```

**Resource Utilization** - CPU, memory, network, disk
```go
import "runtime"

var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("Alloc = %v MB", m.Alloc / 1024 / 1024)
fmt.Printf("TotalAlloc = %v MB", m.TotalAlloc / 1024 / 1024)
fmt.Printf("Sys = %v MB", m.Sys / 1024 / 1024)
fmt.Printf("NumGC = %v", m.NumGC)
```

### Algorithm Complexity

**Common complexities:**

```go
// O(1) - Constant
func getByIndex(arr []int, index int) int {
    return arr[index]
}

// O(log n) - Logarithmic
func binarySearch(arr []int, target int) int {
    left, right := 0, len(arr)-1
    for left <= right {
        mid := (left + right) / 2
        if arr[mid] == target {
            return mid
        }
        if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1
}

// O(n) - Linear
func sum(arr []int) int {
    total := 0
    for _, v := range arr {
        total += v
    }
    return total
}

// O(n log n) - Linearithmic
func quickSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    // Quick sort implementation
}

// O(n²) - Quadratic
func bubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n; i++ {
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
}
```

**Choosing the right data structure:**

```go
// ❌ BAD: O(n) lookup
type UserStore struct {
    users []User
}

func (s *UserStore) Find(id string) *User {
    for i := range s.users {
        if s.users[i].ID == id {
            return &s.users[i]
        }
    }
    return nil
}

// ✅ GOOD: O(1) lookup
type UserStore struct {
    users map[string]User
}

func (s *UserStore) Find(id string) (*User, bool) {
    user, ok := s.users[id]
    return &user, ok
}
```

### Profiling Workflow

```
1. Identify performance problem
   └─ User complaints, monitoring alerts, SLO violations

2. Reproduce in isolation
   └─ Benchmark or load test

3. Profile to find bottleneck
   └─ CPU, memory, blocking, or mutex profile

4. Form hypothesis
   └─ What's causing the bottleneck?

5. Optimize
   └─ Make targeted changes

6. Verify improvement
   └─ Re-benchmark and compare

7. Deploy and monitor
   └─ Verify production improvements
```

### Benchmarking Best Practices

```go
// Good benchmark structure
func BenchmarkOperation(b *testing.B) {
    // Setup (not timed)
    data := setupTestData()
    
    // Reset timer to exclude setup
    b.ResetTimer()
    
    // Benchmark loop
    for i := 0; i < b.N; i++ {
        // Operation to benchmark
        _ = processData(data)
    }
}

// Prevent compiler optimizations
func BenchmarkOperation(b *testing.B) {
    var result int
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        result = expensiveOperation()
    }
    
    // Prevent dead code elimination
    _ = result
}

// Report allocations
func BenchmarkWithAllocs(b *testing.B) {
    b.ReportAllocs()
    
    for i := 0; i < b.N; i++ {
        _ = createData()
    }
}

// Sub-benchmarks for comparison
func BenchmarkApproaches(b *testing.B) {
    b.Run("approach-1", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            approach1()
        }
    })
    
    b.Run("approach-2", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            approach2()
        }
    })
}
```

---

## 8.2 CPU Optimization

### Profiling CPU Usage

```bash
# Generate CPU profile
go test -bench=. -cpuprofile=cpu.prof

# Analyze with pprof
go tool pprof cpu.prof

# Interactive commands
(pprof) top        # Show top functions
(pprof) list func  # Show source with time
(pprof) web        # Generate graph (requires graphviz)
(pprof) pdf        # Generate PDF report
```

**In production:**

```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    // Expose pprof endpoints
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // Your application
}
```

```bash
# Capture 30-second CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

### Reducing Function Call Overhead

**Inlining:**

```go
// Small functions are automatically inlined
func add(a, b int) int {
    return a + b
}

// Force inlining (if beneficial)
//go:inline
func fastPath(x int) int {
    return x * 2
}

// Prevent inlining (for benchmarking)
//go:noinline
func slowPath(x int) int {
    return x * 2
}
```

**Check if function is inlined:**

```bash
go build -gcflags="-m" main.go 2>&1 | grep inline
```

### Loop Optimizations

```go
// ❌ BAD: Inefficient loop
func sumBad(data []int) int {
    sum := 0
    for i := 0; i < len(data); i++ {  // len() called each iteration
        sum += data[i]
    }
    return sum
}

// ✅ GOOD: Cache length
func sumGood(data []int) int {
    sum := 0
    n := len(data)  // Compute once
    for i := 0; i < n; i++ {
        sum += data[i]
    }
    return sum
}

// ✅ BETTER: Use range (compiler optimized)
func sumBetter(data []int) int {
    sum := 0
    for _, v := range data {
        sum += v
    }
    return sum
}

// Loop unrolling for performance-critical code
func sumUnrolled(data []int) int {
    sum := 0
    n := len(data)
    
    // Process 4 elements at a time
    i := 0
    for ; i+3 < n; i += 4 {
        sum += data[i] + data[i+1] + data[i+2] + data[i+3]
    }
    
    // Handle remainder
    for ; i < n; i++ {
        sum += data[i]
    }
    
    return sum
}
```

### Avoiding Unnecessary Work

```go
// ❌ BAD: Recomputing in loop
func processItemsBad(items []Item) {
    for _, item := range items {
        if isValid(item) && isAllowed(item) && isReady(item) {
            process(item)
        }
    }
}

// ✅ GOOD: Short-circuit evaluation
func processItemsGood(items []Item) {
    for _, item := range items {
        // Check cheapest condition first
        if !isReady(item) {
            continue
        }
        if !isValid(item) {
            continue
        }
        if !isAllowed(item) {
            continue
        }
        process(item)
    }
}

// ❌ BAD: Expensive operation in condition
func filterItemsBad(items []Item) []Item {
    var result []Item
    for _, item := range items {
        if expensiveCheck(item) {  // Called for every item
            result = append(result, item)
        }
    }
    return result
}

// ✅ GOOD: Cache expensive result
func filterItemsGood(items []Item) []Item {
    var result []Item
    checked := make(map[string]bool)
    
    for _, item := range items {
        key := item.Key()
        if valid, ok := checked[key]; ok {
            if valid {
                result = append(result, item)
            }
            continue
        }
        
        valid := expensiveCheck(item)
        checked[key] = valid
        if valid {
            result = append(result, item)
        }
    }
    return result
}
```

### String Operations

```go
// ❌ BAD: String concatenation in loop
func buildStringBad(parts []string) string {
    result := ""
    for _, part := range parts {
        result += part  // Creates new string each iteration
    }
    return result
}

// ✅ GOOD: strings.Builder
func buildStringGood(parts []string) string {
    var sb strings.Builder
    sb.Grow(len(parts) * 10)  // Pre-allocate
    for _, part := range parts {
        sb.WriteString(part)
    }
    return sb.String()
}

// ✅ GOOD: strings.Join
func buildStringBest(parts []string) string {
    return strings.Join(parts, "")
}
```

**Benchmark comparison:**

```go
func BenchmarkStringConcat(b *testing.B) {
    parts := make([]string, 100)
    for i := range parts {
        parts[i] = "part"
    }
    
    b.Run("concatenation", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = buildStringBad(parts)
        }
    })
    
    b.Run("builder", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = buildStringGood(parts)
        }
    })
    
    b.Run("join", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = buildStringBest(parts)
        }
    })
}

// Results:
// concatenation    50000 ns/op    50000 B/op    99 allocs/op
// builder           1000 ns/op      512 B/op     1 allocs/op
// join              1000 ns/op      512 B/op     1 allocs/op
```

### Interface and Reflection Overhead

```go
// ❌ BAD: Interface boxing/unboxing in hot path
func processBad(data []interface{}) int {
    sum := 0
    for _, v := range data {
        sum += v.(int)  // Type assertion overhead
    }
    return sum
}

// ✅ GOOD: Concrete types
func processGood(data []int) int {
    sum := 0
    for _, v := range data {
        sum += v
    }
    return sum
}

// ❌ BAD: Reflection in performance-critical code
func sumReflection(slice interface{}) int {
    v := reflect.ValueOf(slice)
    sum := 0
    for i := 0; i < v.Len(); i++ {
        sum += int(v.Index(i).Int())
    }
    return sum
}

// ✅ GOOD: Type-specific implementations
func sumInt(slice []int) int {
    sum := 0
    for _, v := range slice {
        sum += v
    }
    return sum
}
```

### Bounds Check Elimination

```go
// Compiler can eliminate bounds checks in safe patterns

// ❌ Manual bounds checks (redundant)
func sumManual(data []int) int {
    sum := 0
    for i := 0; i < len(data); i++ {
        if i < len(data) {  // Redundant check
            sum += data[i]
        }
    }
    return sum
}

// ✅ Compiler eliminates bounds check
func sumOptimized(data []int) int {
    sum := 0
    for i := range data {  // Compiler knows i is in bounds
        sum += data[i]
    }
    return sum
}

// Check bounds check elimination
// go build -gcflags="-d=ssa/check_bce/debug=1" main.go
```

Continuing with Part 8...
## 8.3 Memory Optimization

### Understanding Memory Allocation

**Stack vs Heap:**

```go
// Stack allocation (fast, no GC)
func stackAlloc() int {
    x := 42  // Allocated on stack
    return x
}

// Heap allocation (slower, GC needed)
func heapAlloc() *int {
    x := 42
    return &x  // Escapes to heap
}

// Check escape analysis
// go build -gcflags="-m" main.go
```

**Escape analysis:**

```go
// Does NOT escape
func noEscape() {
    x := make([]int, 100)
    process(x)  // Passed by value, doesn't escape
}

// DOES escape
func escapes() []int {
    x := make([]int, 100)
    return x  // Returned, must escape
}

func escapesPointer() *User {
    u := User{Name: "Alice"}
    return &u  // Escapes to heap
}

// Can prevent escape
func noEscapePointer(u *User) {
    u.Name = "Bob"  // Doesn't escape if u is on stack
}
```

### Reducing Allocations

**Pre-allocate slices:**

```go
// ❌ BAD: Growing slice
func buildSliceBad() []int {
    var result []int
    for i := 0; i < 1000; i++ {
        result = append(result, i)  // Multiple reallocations
    }
    return result
}

// ✅ GOOD: Pre-allocate
func buildSliceGood() []int {
    result := make([]int, 0, 1000)  // Pre-allocate capacity
    for i := 0; i < 1000; i++ {
        result = append(result, i)
    }
    return result
}

// ✅ BEST: Pre-allocate exact size
func buildSliceBest() []int {
    result := make([]int, 1000)  // Pre-allocate length
    for i := 0; i < 1000; i++ {
        result[i] = i
    }
    return result
}
```

**Benchmark:**

```go
func BenchmarkSliceAlloc(b *testing.B) {
    b.Run("bad", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = buildSliceBad()
        }
    })
    
    b.Run("good", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = buildSliceGood()
        }
    })
    
    b.Run("best", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = buildSliceBest()
        }
    })
}

// Results:
// bad     20000 ns/op    50000 B/op    10 allocs/op  (multiple reallocs)
// good    10000 ns/op     8192 B/op     1 allocs/op  (single alloc)
// best    10000 ns/op     8192 B/op     1 allocs/op  (single alloc)
```

### Object Pooling with sync.Pool

```go
import "sync"

// Reuse expensive objects
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processDataWithPool(data []byte) []byte {
    // Get buffer from pool
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()  // Clean for reuse
        bufferPool.Put(buf)  // Return to pool
    }()
    
    // Use buffer
    buf.Write(data)
    buf.WriteString(" processed")
    
    // Make copy before returning buffer to pool
    result := make([]byte, buf.Len())
    copy(result, buf.Bytes())
    
    return result
}

// Without pool
func processDataNoPool(data []byte) []byte {
    buf := new(bytes.Buffer)  // Allocates every call
    buf.Write(data)
    buf.WriteString(" processed")
    return buf.Bytes()
}
```

**Benchmark:**

```go
func BenchmarkBufferPool(b *testing.B) {
    data := []byte("test data")
    
    b.Run("with-pool", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = processDataWithPool(data)
        }
    })
    
    b.Run("without-pool", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            _ = processDataNoPool(data)
        }
    })
}

// Results:
// with-pool       100 ns/op      32 B/op     1 allocs/op
// without-pool    200 ns/op      96 B/op     2 allocs/op
```

### Avoiding Unnecessary Copies

```go
// ❌ BAD: Copying large structs
type LargeStruct struct {
    Data [1000]int
    More [1000]string
}

func processBad(s LargeStruct) {  // Copies entire struct
    // Process
}

// ✅ GOOD: Use pointer
func processGood(s *LargeStruct) {  // Passes pointer
    // Process
}

// ❌ BAD: Copying slices
func filterBad(items []Item) []Item {
    var result []Item
    for _, item := range items {  // Copies each item
        if item.Valid {
            result = append(result, item)
        }
    }
    return result
}

// ✅ GOOD: Use indices or pointers
func filterGood(items []Item) []Item {
    result := make([]Item, 0, len(items))
    for i := range items {  // Use index
        if items[i].Valid {
            result = append(result, items[i])
        }
    }
    return result
}
```

### String to Byte Conversion

```go
// ❌ BAD: Allocates new byte slice
func stringToBytesBad(s string) []byte {
    return []byte(s)  // Allocates and copies
}

// ✅ GOOD: Zero-copy conversion (read-only)
import "unsafe"

func stringToBytesGood(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

// WARNING: Result must be read-only!
// Modifying the slice will crash or corrupt memory

// Safe wrapper
func stringToBytesReadOnly(s string) []byte {
    if len(s) == 0 {
        return nil
    }
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

// Use case: Hash computation
func hashString(s string) uint64 {
    b := stringToBytesReadOnly(s)
    return hash(b)  // No allocation
}
```

### Memory Profiling

```bash
# Generate memory profile
go test -bench=. -memprofile=mem.prof

# Analyze allocations
go tool pprof -alloc_space mem.prof
(pprof) top

# Analyze in-use memory
go tool pprof -inuse_space mem.prof
(pprof) top

# In production
curl http://localhost:6060/debug/pprof/heap > heap.prof
go tool pprof heap.prof
```

### GC Tuning

```go
import "runtime/debug"

// Adjust GC target percentage (default 100)
debug.SetGCPercent(50)  // More aggressive GC
debug.SetGCPercent(200) // Less aggressive GC

// Force GC (rarely needed)
runtime.GC()

// Monitor GC stats
var stats debug.GCStats
debug.ReadGCStats(&stats)
fmt.Printf("NumGC: %d\n", stats.NumGC)
fmt.Printf("PauseTotal: %v\n", stats.PauseTotal)
```

**GOGC environment variable:**

```bash
# Default: GOGC=100 (GC when heap doubles)
GOGC=200 ./myapp  # Less frequent GC
GOGC=50 ./myapp   # More frequent GC
GOGC=off ./myapp  # Disable GC (not recommended)
```

---

## 8.4 Concurrency Optimization

### Goroutine Pool Pattern

```go
// Worker pool for bounded concurrency
type WorkerPool struct {
    workers   int
    tasks     chan func()
    wg        sync.WaitGroup
}

func NewWorkerPool(workers int) *WorkerPool {
    pool := &WorkerPool{
        workers: workers,
        tasks:   make(chan func(), workers*2),  // Buffered channel
    }
    
    // Start workers
    for i := 0; i < workers; i++ {
        pool.wg.Add(1)
        go pool.worker()
    }
    
    return pool
}

func (p *WorkerPool) worker() {
    defer p.wg.Done()
    for task := range p.tasks {
        task()
    }
}

func (p *WorkerPool) Submit(task func()) {
    p.tasks <- task
}

func (p *WorkerPool) Close() {
    close(p.tasks)
    p.wg.Wait()
}

// Usage
func processItemsConcurrent(items []Item) {
    pool := NewWorkerPool(runtime.NumCPU())
    defer pool.Close()
    
    for _, item := range items {
        item := item  // Capture
        pool.Submit(func() {
            process(item)
        })
    }
}
```

### Optimizing Channel Operations

```go
// ❌ BAD: Unbuffered channel (blocking)
func processBad(items []Item) {
    ch := make(chan Result)
    
    for _, item := range items {
        go func(item Item) {
            ch <- process(item)  // May block
        }(item)
    }
    
    // Collect results
    for range items {
        <-ch
    }
}

// ✅ GOOD: Buffered channel (non-blocking)
func processGood(items []Item) {
    ch := make(chan Result, len(items))  // Buffered
    
    for _, item := range items {
        go func(item Item) {
            ch <- process(item)  // Won't block
        }(item)
    }
    
    // Collect results
    for range items {
        <-ch
    }
}

// ✅ BETTER: Worker pool (bounded concurrency)
func processBetter(items []Item) {
    results := make(chan Result, len(items))
    pool := NewWorkerPool(runtime.NumCPU())
    defer pool.Close()
    
    for _, item := range items {
        item := item
        pool.Submit(func() {
            results <- process(item)
        })
    }
    
    // Collect results
    for range items {
        <-results
    }
}
```

### Avoiding Goroutine Leaks

```go
// ❌ BAD: Goroutine leak
func leaky() {
    ch := make(chan int)
    go func() {
        result := expensiveOperation()
        ch <- result  // Blocks forever if no receiver
    }()
    // Goroutine never exits
}

// ✅ GOOD: Use context for cancellation
func noLeak(ctx context.Context) {
    ch := make(chan int, 1)  // Buffered
    
    go func() {
        result := expensiveOperation()
        select {
        case ch <- result:
        case <-ctx.Done():
            return  // Exit on cancellation
        }
    }()
    
    select {
    case result := <-ch:
        process(result)
    case <-ctx.Done():
        return
    }
}
```

### Lock Contention

```go
// ❌ BAD: High lock contention
type Counter struct {
    mu    sync.Mutex
    value int64
}

func (c *Counter) Increment() {
    c.mu.Lock()
    c.value++
    c.mu.Unlock()
}

// ✅ GOOD: Atomic operations
type AtomicCounter struct {
    value atomic.Int64
}

func (c *AtomicCounter) Increment() {
    c.value.Add(1)
}

// ✅ GOOD: Sharded counters for high concurrency
type ShardedCounter struct {
    shards []paddedCounter
}

type paddedCounter struct {
    value atomic.Int64
    _     [7]int64  // Padding to prevent false sharing
}

func NewShardedCounter(shards int) *ShardedCounter {
    return &ShardedCounter{
        shards: make([]paddedCounter, shards),
    }
}

func (c *ShardedCounter) Increment() {
    shard := int(fastrand()) % len(c.shards)
    c.shards[shard].value.Add(1)
}

func (c *ShardedCounter) Value() int64 {
    var total int64
    for i := range c.shards {
        total += c.shards[i].value.Load()
    }
    return total
}
```

### False Sharing Prevention

```go
// ❌ BAD: False sharing
type BadCounter struct {
    counter1 int64
    counter2 int64  // Likely same cache line as counter1
}

// ✅ GOOD: Cache line padding
type GoodCounter struct {
    counter1 int64
    _        [7]int64  // Padding (64 bytes total)
    counter2 int64
    _        [7]int64  // Padding
}

// Or use atomic with padding
type PaddedAtomic struct {
    value atomic.Int64
    _     [7]int64
}
```

---

## 8.5 I/O Optimization

### Buffered I/O

```go
// ❌ BAD: Unbuffered I/O
func readFileBad(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    for {
        var line string
        _, err := fmt.Fscanln(file, &line)  // Unbuffered
        if err != nil {
            break
        }
        process(line)
    }
    return nil
}

// ✅ GOOD: Buffered I/O
func readFileGood(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    scanner := bufio.NewScanner(file)  // Buffered
    for scanner.Scan() {
        process(scanner.Text())
    }
    return scanner.Err()
}

// ✅ BETTER: Custom buffer size
func readFileBetter(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    reader := bufio.NewReaderSize(file, 64*1024)  // 64KB buffer
    scanner := bufio.NewScanner(reader)
    
    // Increase scanner buffer if needed
    buf := make([]byte, 64*1024)
    scanner.Buffer(buf, 1024*1024)
    
    for scanner.Scan() {
        process(scanner.Text())
    }
    return scanner.Err()
}
```

### Batch Processing

```go
// ❌ BAD: Process one at a time
func insertOneByOne(db *sql.DB, items []Item) error {
    for _, item := range items {
        _, err := db.Exec("INSERT INTO items VALUES ($1, $2)", item.ID, item.Name)
        if err != nil {
            return err
        }
    }
    return nil
}

// ✅ GOOD: Batch insert
func insertBatch(db *sql.DB, items []Item) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    stmt, err := tx.Prepare("INSERT INTO items VALUES ($1, $2)")
    if err != nil {
        return err
    }
    defer stmt.Close()
    
    for _, item := range items {
        if _, err := stmt.Exec(item.ID, item.Name); err != nil {
            return err
        }
    }
    
    return tx.Commit()
}

// ✅ BETTER: Batch with value list
func insertBatchValues(db *sql.DB, items []Item) error {
    if len(items) == 0 {
        return nil
    }
    
    valueStrings := make([]string, 0, len(items))
    valueArgs := make([]interface{}, 0, len(items)*2)
    
    for i, item := range items {
        valueStrings = append(valueStrings, fmt.Sprintf("($%d, $%d)", i*2+1, i*2+2))
        valueArgs = append(valueArgs, item.ID, item.Name)
    }
    
    query := fmt.Sprintf("INSERT INTO items VALUES %s", strings.Join(valueStrings, ","))
    _, err := db.Exec(query, valueArgs...)
    return err
}
```

### Connection Pooling

```go
// Configure database connection pool
func setupDB(dsn string) (*sql.DB, error) {
    db, err := sql.Open("postgres", dsn)
    if err != nil {
        return nil, err
    }
    
    // Maximum open connections
    db.SetMaxOpenConns(25)
    
    // Maximum idle connections
    db.SetMaxIdleConns(5)
    
    // Maximum lifetime of a connection
    db.SetConnMaxLifetime(5 * time.Minute)
    
    // Maximum idle time
    db.SetConnMaxIdleTime(5 * time.Minute)
    
    return db, nil
}

// Monitor connection pool stats
func monitorConnectionPool(db *sql.DB) {
    stats := db.Stats()
    log.Printf("Open connections: %d", stats.OpenConnections)
    log.Printf("In use: %d", stats.InUse)
    log.Printf("Idle: %d", stats.Idle)
    log.Printf("Wait count: %d", stats.WaitCount)
    log.Printf("Wait duration: %v", stats.WaitDuration)
}
```

### HTTP Client Optimization

```go
// ✅ Optimized HTTP client
var httpClient = &http.Client{
    Timeout: 10 * time.Second,
    Transport: &http.Transport{
        MaxIdleConns:        100,
        MaxIdleConnsPerHost: 10,
        IdleConnTimeout:     90 * time.Second,
        DisableCompression:  false,
        ForceAttemptHTTP2:   true,
    },
}

// Reuse client
func makeRequest(url string) (*http.Response, error) {
    return httpClient.Get(url)
}
```

Continuing with Part 8...


## 8.6 Caching Strategies

### In-Memory Caching

```go
// Simple cache with expiration
type Cache struct {
    data map[string]cacheEntry
    mu   sync.RWMutex
}

type cacheEntry struct {
    value      interface{}
    expiration time.Time
}

func NewCache() *Cache {
    c := &Cache{
        data: make(map[string]cacheEntry),
    }
    
    // Cleanup expired entries
    go func() {
        ticker := time.NewTicker(time.Minute)
        defer ticker.Stop()
        
        for range ticker.C {
            c.cleanup()
        }
    }()
    
    return c
}

func (c *Cache) Set(key string, value interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.data[key] = cacheEntry{
        value:      value,
        expiration: time.Now().Add(ttl),
    }
}

func (c *Cache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    entry, ok := c.data[key]
    if !ok {
        return nil, false
    }
    
    if time.Now().After(entry.expiration) {
        return nil, false
    }
    
    return entry.value, true
}

func (c *Cache) cleanup() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    now := time.Now()
    for key, entry := range c.data {
        if now.After(entry.expiration) {
            delete(c.data, key)
        }
    }
}
```

### LRU Cache Implementation

```go
import "container/list"

type LRUCache struct {
    capacity int
    cache    map[string]*list.Element
    lru      *list.List
    mu       sync.Mutex
}

type entry struct {
    key   string
    value interface{}
}

func NewLRUCache(capacity int) *LRUCache {
    return &LRUCache{
        capacity: capacity,
        cache:    make(map[string]*list.Element),
        lru:      list.New(),
    }
}

func (c *LRUCache) Get(key string) (interface{}, bool) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    if elem, ok := c.cache[key]; ok {
        c.lru.MoveToFront(elem)
        return elem.Value.(*entry).value, true
    }
    
    return nil, false
}

func (c *LRUCache) Set(key string, value interface{}) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    if elem, ok := c.cache[key]; ok {
        c.lru.MoveToFront(elem)
        elem.Value.(*entry).value = value
        return
    }
    
    if c.lru.Len() >= c.capacity {
        oldest := c.lru.Back()
        if oldest != nil {
            c.lru.Remove(oldest)
            delete(c.cache, oldest.Value.(*entry).key)
        }
    }
    
    elem := c.lru.PushFront(&entry{key, value})
    c.cache[key] = elem
}
```

### Cache-Aside Pattern

```go
type UserService struct {
    cache *LRUCache
    db    *sql.DB
}

func (s *UserService) GetUser(ctx context.Context, id string) (*User, error) {
    // Try cache first
    if cached, ok := s.cache.Get(id); ok {
        return cached.(*User), nil
    }
    
    // Cache miss - fetch from database
    user, err := s.fetchUserFromDB(ctx, id)
    if err != nil {
        return nil, err
    }
    
    // Update cache
    s.cache.Set(id, user)
    
    return user, nil
}

func (s *UserService) UpdateUser(ctx context.Context, user *User) error {
    // Update database
    if err := s.updateUserInDB(ctx, user); err != nil {
        return err
    }
    
    // Invalidate cache
    s.cache.Set(user.ID, user)
    
    return nil
}
```

### Redis Caching

```go
import "github.com/go-redis/redis/v8"

type RedisCache struct {
    client *redis.Client
}

func NewRedisCache(addr string) *RedisCache {
    return &RedisCache{
        client: redis.NewClient(&redis.Options{
            Addr:         addr,
            PoolSize:     10,
            MinIdleConns: 5,
        }),
    }
}

func (c *RedisCache) Get(ctx context.Context, key string) (string, error) {
    return c.client.Get(ctx, key).Result()
}

func (c *RedisCache) Set(ctx context.Context, key string, value interface{}, ttl time.Duration) error {
    return c.client.Set(ctx, key, value, ttl).Err()
}

// Cache with serialization
func (c *RedisCache) GetJSON(ctx context.Context, key string, dest interface{}) error {
    data, err := c.client.Get(ctx, key).Bytes()
    if err != nil {
        return err
    }
    return json.Unmarshal(data, dest)
}

func (c *RedisCache) SetJSON(ctx context.Context, key string, value interface{}, ttl time.Duration) error {
    data, err := json.Marshal(value)
    if err != nil {
        return err
    }
    return c.client.Set(ctx, key, data, ttl).Err()
}
```

---

## 8.7 Compiler Optimizations

### Understanding Compiler Flags

```bash
# View all optimizations
go build -gcflags="-m" main.go

# View inlining decisions
go build -gcflags="-m -m" main.go

# View escape analysis
go build -gcflags="-m" main.go 2>&1 | grep escape

# Disable optimizations (for debugging)
go build -gcflags="-N -l" main.go

# Generate assembly
go build -gcflags="-S" main.go > assembly.txt
```

### Inlining Budget

```go
// Small functions are automatically inlined
func add(a, b int) int {  // Will be inlined
    return a + b
}

// Large functions won't be inlined
func complex(a, b int) int {  // Won't be inlined (too large)
    // Many lines of code
    // ...
    return result
}

// Force inline (use sparingly)
//go:inline
func mustInline(x int) int {
    return x * 2
}

// Prevent inline (for debugging)
//go:noinline
func noInline(x int) int {
    return x * 2
}
```

### Dead Code Elimination

```go
// Compiler removes unreachable code
func example() {
    if false {
        // This code is eliminated
        expensiveOperation()
    }
    
    // This runs
    normalOperation()
}

// Constant folding
func constants() int {
    const a = 10
    const b = 20
    return a + b  // Computed at compile time: return 30
}
```

### Bounds Check Elimination

```go
// Compiler can eliminate bounds checks
func sum(data []int) int {
    total := 0
    for i := 0; i < len(data); i++ {
        total += data[i]  // Bounds check eliminated
    }
    return total
}

// Help compiler eliminate bounds checks
func sumOptimized(data []int) int {
    total := 0
    _ = data[len(data)-1]  // Hint to compiler
    for i := range data {
        total += data[i]  // Bounds check eliminated
    }
    return total
}
```

---

## 8.8 Production Profiling

### Continuous Profiling

```go
// Enable pprof endpoints
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    // Separate debug server
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // Main application
    runApp()
}
```

**Profile endpoints:**

```bash
# CPU profile (30 seconds)
curl http://localhost:6060/debug/pprof/profile?seconds=30 > cpu.prof

# Heap profile
curl http://localhost:6060/debug/pprof/heap > heap.prof

# Goroutine profile
curl http://localhost:6060/debug/pprof/goroutine > goroutine.prof

# All goroutines (detailed)
curl http://localhost:6060/debug/pprof/goroutine?debug=2

# Block profile
curl http://localhost:6060/debug/pprof/block > block.prof

# Mutex profile
curl http://localhost:6060/debug/pprof/mutex > mutex.prof
```

### Profiling Without Disruption

```go
// Sample CPU profile periodically
func continuousProfiler(interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()
    
    for range ticker.C {
        f, err := os.Create(fmt.Sprintf("cpu-%d.prof", time.Now().Unix()))
        if err != nil {
            log.Printf("failed to create profile: %v", err)
            continue
        }
        
        if err := pprof.StartCPUProfile(f); err != nil {
            log.Printf("failed to start CPU profile: %v", err)
            f.Close()
            continue
        }
        
        time.Sleep(30 * time.Second)
        
        pprof.StopCPUProfile()
        f.Close()
    }
}
```

### Automated Performance Regression Detection

```go
// benchmark_test.go
func BenchmarkCriticalPath(b *testing.B) {
    b.ReportAllocs()
    
    for i := 0; i < b.N; i++ {
        criticalOperation()
    }
}
```

```bash
# Baseline
go test -bench=. -benchmem > old.txt

# After changes
go test -bench=. -benchmem > new.txt

# Compare
benchstat old.txt new.txt
```

**CI/CD integration:**

```yaml
# .github/workflows/benchmark.yml
name: Benchmark

on:
  pull_request:
    branches: [main]

jobs:
  benchmark:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-go@v5
        with:
          go-version: '1.24'
      
      - name: Run benchmarks (baseline)
        run: |
          git checkout ${{ github.base_ref }}
          go test -bench=. -benchmem > baseline.txt
      
      - name: Run benchmarks (PR)
        run: |
          git checkout ${{ github.head_ref }}
          go test -bench=. -benchmem > pr.txt
      
      - name: Compare
        run: |
          go install golang.org/x/perf/cmd/benchstat@latest
          benchstat baseline.txt pr.txt
```

---

## 8.9 Performance Patterns

### Lazy Initialization

```go
// Initialize expensive resource only when needed
type Service struct {
    client     *http.Client
    clientOnce sync.Once
}

func (s *Service) getClient() *http.Client {
    s.clientOnce.Do(func() {
        s.client = &http.Client{
            Timeout: 10 * time.Second,
            Transport: &http.Transport{
                MaxIdleConns: 100,
            },
        }
    })
    return s.client
}
```

### Fast Path Optimization

```go
// Optimize common case
func processRequest(req Request) Response {
    // Fast path: common case
    if req.Type == "simple" {
        return simpleResponse(req)
    }
    
    // Slow path: complex cases
    return complexResponse(req)
}
```

### Copy-on-Write

```go
// Share read-only data, copy only when modifying
type ConfigStore struct {
    config atomic.Value  // *Config
}

func (s *ConfigStore) Get() *Config {
    return s.config.Load().(*Config)
}

func (s *ConfigStore) Update(newConfig *Config) {
    // All readers continue using old config
    // until this atomic store completes
    s.config.Store(newConfig)
}
```

### Zero-Copy Techniques

```go
// Use io.Copy instead of buffering
func proxy(dst io.Writer, src io.Reader) error {
    _, err := io.Copy(dst, src)  // Zero-copy when possible
    return err
}

// Use sendfile for file transfers (on Linux)
import "net"

func sendFile(conn net.Conn, file *os.File) error {
    // Uses sendfile syscall (zero-copy)
    _, err := io.Copy(conn, file)
    return err
}
```

---

## Part 8 Summary and Key Takeaways

### Performance Optimization Principles

**✅ Measure First:**
- Profile before optimizing
- Identify actual bottlenecks
- Set performance targets
- Benchmark improvements

**✅ Focus on Impact:**
- Optimize hot paths (80/20 rule)
- Algorithm > micro-optimizations
- Balance readability and performance
- Document performance decisions

### CPU Optimization

**✅ Key Techniques:**
- Choose efficient algorithms (O(n) vs O(n²))
- Reduce function call overhead
- Eliminate unnecessary work
- Use strings.Builder for concatenation
- Avoid reflection in hot paths
- Help compiler with bounds check elimination

### Memory Optimization

**✅ Key Techniques:**
- Pre-allocate slices with capacity
- Use sync.Pool for object reuse
- Avoid unnecessary heap allocations
- Understand escape analysis
- Tune GC with GOGC
- Profile memory usage

### Concurrency Optimization

**✅ Key Techniques:**
- Use worker pools for bounded concurrency
- Buffer channels appropriately
- Avoid goroutine leaks
- Use atomic operations vs mutexes
- Prevent false sharing
- Monitor goroutine count

### I/O Optimization

**✅ Key Techniques:**
- Use buffered I/O
- Batch operations
- Configure connection pools
- Reuse HTTP clients
- Enable HTTP/2
- Implement caching

### Caching Strategies

**✅ Patterns:**
- In-memory caching (fast, limited capacity)
- LRU cache (bounded memory)
- Cache-aside pattern
- Redis for distributed caching
- Set appropriate TTLs

### Performance Checklist

**Before Optimization:**
- [ ] Profile to find bottleneck
- [ ] Establish baseline benchmark
- [ ] Set performance target
- [ ] Understand root cause

**During Optimization:**
- [ ] Change one thing at a time
- [ ] Keep code readable
- [ ] Document assumptions
- [ ] Test for correctness

**After Optimization:**
- [ ] Benchmark improvement
- [ ] Verify production gains
- [ ] Monitor for regressions
- [ ] Update documentation

### Common Pitfalls

**❌ Premature Optimization:**
- Optimizing before profiling
- Sacrificing readability
- Optimizing cold paths
- Micro-optimizing without impact

**❌ Measurement Errors:**
- Not using b.ResetTimer()
- Dead code elimination
- Not running enough iterations
- Inconsistent environment

**❌ Memory Mistakes:**
- Growing slices without pre-allocation
- Not reusing expensive objects
- Creating goroutine leaks
- Ignoring GC pressure

### Optimization Workflow

```
1. Identify Problem
   └─ Monitoring alerts, user reports, SLO violations

2. Reproduce
   └─ Create benchmark or load test

3. Profile
   └─ CPU, memory, goroutine, or mutex profile

4. Analyze
   └─ Find hot path or bottleneck

5. Hypothesize
   └─ Form theory about cause

6. Optimize
   └─ Make focused changes

7. Benchmark
   └─ Verify improvement

8. Deploy
   └─ Monitor production impact

9. Iterate
   └─ Continue if needed
```

### Tools Ecosystem

**Profiling:**
- pprof (built-in)
- go tool trace
- benchstat
- Continuous profiling (Pyroscope, Parca)

**Benchmarking:**
- testing.B (built-in)
- benchstat
- benchcmp

**Analysis:**
- go-torch (flame graphs)
- pprof web UI
- Grafana dashboards

### Interview Questions

**Question 1: How do you identify a memory leak in a Go application?**

**Answer:**

1. **Monitor memory growth:**
```bash
# Check if memory grows unbounded
watch -n 1 'ps aux | grep myapp'
```

2. **Collect heap profiles:**
```bash
# Before
curl http://localhost:6060/debug/pprof/heap > heap1.prof

# Wait for leak to manifest
sleep 300

# After
curl http://localhost:6060/debug/pprof/heap > heap2.prof

# Compare
go tool pprof -base heap1.prof heap2.prof
(pprof) top
(pprof) list functionName
```

3. **Common causes:**
- Goroutine leaks (check goroutine profile)
- Unbounded caches
- Forgotten references in slices/maps
- Not closing resources

---

**Question 2: How would you optimize a slow HTTP endpoint?**

**Answer:**

**Step 1: Profile**
```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

**Step 2: Identify bottleneck**
```
(pprof) top
# Find slow function
```

**Step 3: Common optimizations**
- Add caching (in-memory or Redis)
- Database query optimization (indexes, N+1 queries)
- Batch operations
- Reduce allocations
- Connection pooling
- Concurrent processing

**Step 4: Verify**
```bash
# Benchmark before/after
ab -n 1000 -c 10 http://localhost:8080/endpoint
```

---

This completes Part 8 of the Complete Go Mastery series on Performance Optimization & Profiling!

### What's Next

You now know how to:
- Profile Go applications
- Optimize CPU-intensive code
- Reduce memory allocations
- Scale with concurrency
- Implement effective caching
- Use compiler optimizations
- Profile production systems
- Measure and verify improvements

**Continue to Part 9 for Security Best Practices, or apply these techniques to your projects!**

