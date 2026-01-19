---
title: "Complete Go Mastery Part 2: Concurrency & Go Runtime Internals"
date: 2025-01-10 12:00:00 +0530
categories: [Programming, Go, Concurrency]
tags: [golang, go, concurrency, goroutines, channels, scheduler, runtime, garbage-collection, memory-management]
---

# Complete Go Mastery Part 2: Concurrency & Go Runtime Internals

## Introduction

Welcome to Part 2 of the Complete Go Mastery series. In Part 1, we covered Go fundamentals—types, control flow, functions, interfaces, and packages. Now we dive into what makes Go truly unique and powerful: **concurrency**.

Concurrency is not just a feature bolted onto Go—it's fundamental to the language's design. Go was built from the ground up to make concurrent programming simple, safe, and efficient. While other languages struggle with threads, locks, and race conditions, Go provides goroutines and channels as first-class citizens.

But to truly master Go concurrency, you need to understand what happens under the hood. How does the Go scheduler manage thousands of goroutines on a handful of OS threads? How does the garbage collector work without stopping the world for long? How does Go's memory allocator keep allocations fast?

**What you'll learn in Part 2:**

In this part, we'll explore:
- **Goroutines**: Lightweight concurrent functions and how they work
- **Channels**: Communication between goroutines, patterns, and pitfalls
- **Synchronization Primitives**: Mutexes, WaitGroups, atomics, and when to use each
- **Go Scheduler Internals**: The GMP model and how goroutines are scheduled
- **Memory Management**: Allocation, escape analysis, and the garbage collector
- **Concurrency Patterns**: Real-world patterns for building concurrent systems

By the end of Part 2, you'll understand not just how to write concurrent Go code, but how it works at the runtime level, enabling you to write high-performance concurrent systems with confidence.

---

## 2.1 Concurrency Fundamentals

### What is Concurrency?

First, let's clarify a common confusion:

**Concurrency vs Parallelism:**

- **Concurrency**: Dealing with multiple things at once (design/structure)
- **Parallelism**: Doing multiple things at once (execution)

```
Concurrency: Structure               Parallelism: Execution
┌─────────┐  ┌─────────┐            ┌─────────┐  ┌─────────┐
│ Task A  │  │ Task B  │            │ Task A  │  │ Task B  │
│ ▓░▓░▓░▓░│  │░▓░▓░▓░▓ │            │ ▓▓▓▓▓▓▓▓│  │▓▓▓▓▓▓▓▓ │
└─────────┘  └─────────┘            └─────────┘  └─────────┘
   Time →                              CPU1 →      CPU2 →
   
Tasks interleaved on 1 CPU           Tasks run on 2 CPUs
```

**Rob Pike's definition:**
> "Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once."

Go makes concurrency easy to express, and the runtime handles parallelism automatically based on available CPU cores.

### Why Concurrency in Go?

Modern applications need concurrency:
- **Web servers**: Handle many requests simultaneously
- **Data processing**: Process multiple files in parallel
- **Network I/O**: Wait for network without blocking
- **UI applications**: Keep UI responsive during work
- **Microservices**: Handle multiple independent operations

Traditional approaches (threads, callbacks, async/await) have problems:
- Threads are heavyweight (MBs of stack)
- Manual synchronization is error-prone
- Callback hell reduces readability
- async/await adds complexity

**Go's solution:**
- Lightweight goroutines (2KB initial stack)
- Channels for safe communication
- Simple, sequential-looking code
- Runtime handles scheduling

### Goroutines

A goroutine is a lightweight thread of execution managed by the Go runtime.

#### **Creating Goroutines**

Starting a goroutine is trivial—just use the `go` keyword:

```go
func main() {
    // Start a goroutine
    go sayHello()
    
    // Main continues immediately
    fmt.Println("Main function")
    
    // Wait a bit (not recommended, just for demo)
    time.Sleep(time.Second)
}

func sayHello() {
    fmt.Println("Hello from goroutine")
}

// Output (non-deterministic order):
// Main function
// Hello from goroutine
```

**With anonymous functions:**

```go
func main() {
    // Anonymous goroutine
    go func() {
        fmt.Println("Hello from anonymous goroutine")
    }()
    
    // With arguments
    name := "Alice"
    go func(n string) {
        fmt.Printf("Hello, %s\n", n)
    }(name)
    
    time.Sleep(time.Second)
}
```

#### **Goroutine Characteristics**

**1. Lightweight:**

```go
// Creating a million goroutines is totally fine
func main() {
    for i := 0; i < 1_000_000; i++ {
        go func(id int) {
            // Each goroutine has ~2KB initial stack
            time.Sleep(time.Hour)
        }(i)
    }
    
    time.Sleep(time.Second)
    fmt.Println("Created 1 million goroutines")
}
```

Compare this to OS threads:
- **OS Thread**: 1-2 MB stack, expensive context switch
- **Goroutine**: 2 KB initial stack, cheap context switch
- Can have millions of goroutines, not millions of threads

**2. Growing stacks:**

```go
func recursive(depth int) {
    // Stack grows automatically as needed
    if depth == 0 {
        return
    }
    var buffer [1024]byte  // Allocate stack space
    _ = buffer
    recursive(depth - 1)
}

func main() {
    go recursive(10000)  // Stack grows from 2KB to whatever is needed
    time.Sleep(time.Second)
}
```

The Go runtime automatically grows and shrinks goroutine stacks (starting at 2KB, can grow to ~1GB if needed).

**3. Multiplexed on OS threads:**

```go
// Goroutines are multiplexed onto OS threads by the Go scheduler
func main() {
    runtime.GOMAXPROCS(2)  // Use 2 OS threads
    
    // But can run thousands of goroutines
    for i := 0; i < 10000; i++ {
        go func(id int) {
            // Work...
        }(i)
    }
}
```

The Go scheduler (M:N scheduler) maps many goroutines (N) onto fewer OS threads (M).

#### **Goroutine Lifecycle**

```
Created → Runnable → Running → Waiting → Dead
             ↑          ↓          ↓
             └──────────┴──────────┘
```

**States:**
1. **Created**: `go func()` called, goroutine allocated
2. **Runnable**: Ready to run, waiting for CPU
3. **Running**: Executing on CPU
4. **Waiting**: Blocked (channel, I/O, syscall, sleep)
5. **Dead**: Function returned, goroutine cleaned up

**Transitions:**
- Runnable → Running: Scheduler assigns to OS thread
- Running → Waiting: Blocked on channel/syscall/etc
- Waiting → Runnable: Unblocked
- Running → Dead: Function returns

#### **Critical Goroutine Pitfall: Loop Variables**

One of the most common goroutine bugs:

```go
// ❌ WRONG: All goroutines see the same final value
values := []int{1, 2, 3, 4, 5}
for _, v := range values {
    go func() {
        fmt.Println(v)  // All print 5!
    }()
}
time.Sleep(time.Second)

// Why? The loop variable v is shared across iterations
// By the time goroutines run, v has the final value
```

**Solutions:**

```go
// ✅ Solution 1: Pass as parameter
for _, v := range values {
    go func(val int) {
        fmt.Println(val)  // Each goroutine gets its own copy
    }(v)
}

// ✅ Solution 2: Create new variable in each iteration
for _, v := range values {
    v := v  // Create new v scoped to this iteration
    go func() {
        fmt.Println(v)  // Safe, uses the iteration-scoped v
    }()
}

// ✅ Solution 3: Use index (if needed)
for i := range values {
    go func(idx int) {
        fmt.Println(values[idx])
    }(i)
}
```

**Note:** Go 1.22+ changes loop variable semantics, making Solution 2 automatic, but it's still good practice to be explicit.

#### **Goroutine Leaks**

Goroutines that never finish consume memory and resources:

```go
// ❌ GOROUTINE LEAK: No way to stop this goroutine
func startWorker() {
    go func() {
        for {
            // Infinite work
            doWork()
        }
    }()
}
// The goroutine runs forever, even if no longer needed
```

**How to prevent leaks:**

```go
// ✅ Use context for cancellation
func startWorker(ctx context.Context) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return  // Stop when context is cancelled
            default:
                doWork()
            }
        }
    }()
}

// Usage
ctx, cancel := context.WithCancel(context.Background())
startWorker(ctx)

// Later: stop the worker
cancel()
```

**Common leak scenarios:**
1. Infinite loops without exit condition
2. Blocking on channel that never receives
3. Waiting on mutex/lock that's never released
4. Blocked on syscall that never completes

**Detecting leaks:**

```go
func TestForLeaks(t *testing.T) {
    before := runtime.NumGoroutine()
    
    // Run code that should cleanup goroutines
    runMyCode()
    
    time.Sleep(time.Second)  // Give goroutines time to finish
    
    after := runtime.NumGoroutine()
    if after > before {
        t.Errorf("Goroutine leak: %d goroutines before, %d after", before, after)
    }
}
```

Or use tools:
- `goleak` package: `go.uber.org/goleak`
- pprof goroutine profile: Shows all active goroutines

#### **WaitGroup for Synchronization**

To wait for goroutines to complete:

```go
import "sync"

func main() {
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)  // Increment counter
        
        go func(id int) {
            defer wg.Done()  // Decrement when done
            
            fmt.Printf("Worker %d starting\n", id)
            time.Sleep(time.Second)
            fmt.Printf("Worker %d done\n", id)
        }(i)
    }
    
    wg.Wait()  // Block until counter reaches 0
    fmt.Println("All workers completed")
}
```

**WaitGroup rules:**
- Call `Add()` before starting goroutine (not inside it)
- Call `Done()` when goroutine finishes (use defer)
- Call `Wait()` to block until all goroutines finish
- Don't copy WaitGroup (pass by pointer if needed)

**Common mistake:**

```go
// ❌ WRONG: Add inside goroutine (race condition)
for i := 0; i < 5; i++ {
    go func(id int) {
        wg.Add(1)  // Too late! Wait() might be called before this
        defer wg.Done()
        // work...
    }(i)
}
wg.Wait()

// ✅ CORRECT: Add before starting goroutine
for i := 0; i < 5; i++ {
    wg.Add(1)  // Add before go keyword
    go func(id int) {
        defer wg.Done()
        // work...
    }(i)
}
wg.Wait()
```

### Channels

Channels are Go's way of allowing goroutines to communicate safely. They're typed conduits through which you can send and receive values.

**Philosophy:**
> "Don't communicate by sharing memory; share memory by communicating." - Effective Go

Instead of multiple goroutines accessing shared memory (with locks), they communicate via channels.

#### **Channel Basics**

**Creating channels:**

```go
// Unbuffered channel
ch := make(chan int)

// Buffered channel (capacity 10)
ch := make(chan int, 10)

// Channel of any type
chString := make(chan string)
chStruct := make(chan User)
chPointer := make(chan *User)
```

**Sending and receiving:**

```go
ch := make(chan int)

// Send
ch <- 42  // Send 42 to channel

// Receive
value := <-ch  // Receive from channel

// Receive and ignore value
<-ch

// Receive with ok (check if channel is closed)
value, ok := <-ch
if !ok {
    // Channel is closed
}
```

#### **Unbuffered vs Buffered Channels**

**Unbuffered channel (synchronous):**

```go
ch := make(chan int)  // No buffer

go func() {
    ch <- 42  // Blocks until receiver ready
}()

value := <-ch  // Blocks until sender ready
```

Unbuffered channels provide **synchronization**: send and receive happen simultaneously.

```
Goroutine 1              Channel              Goroutine 2
   │                        │                     │
   │  ch <- 42              │                     │
   ├───────────────────────>│                     │
   │  (blocks)              │                     │
   │                        │      value := <-ch  │
   │                        │<────────────────────┤
   │  (unblocks)            │   (receives)        │
   │                        │                     │
```

**Buffered channel (asynchronous):**

```go
ch := make(chan int, 3)  // Buffer of 3

ch <- 1  // Doesn't block (buffer has space)
ch <- 2  // Doesn't block
ch <- 3  // Doesn't block
// ch <- 4  // Would block (buffer full)

value := <-ch  // Receives 1 (now buffer has space for 1 more)
```

```
Buffer State:
make(chan int, 3)    Empty:    [_, _, _]
ch <- 1              Partial:  [1, _, _]
ch <- 2              Partial:  [1, 2, _]
ch <- 3              Full:     [1, 2, 3]
value := <-ch        Partial:  [_, 2, 3]  (received 1)
```

**When to use each:**

```go
// ✅ Unbuffered (synchronization needed)
done := make(chan bool)
go func() {
    doWork()
    done <- true  // Signal completion
}()
<-done  // Wait for work to complete

// ✅ Buffered (known capacity, prevent blocking)
jobs := make(chan Job, 100)  // Job queue with 100 capacity
for i := 0; i < 100; i++ {
    jobs <- Job{ID: i}  // Won't block until full
}

// ✅ Buffered (reduce goroutine creation)
results := make(chan Result, 10)
for i := 0; i < 10; i++ {
    go func() {
        results <- computeResult()
    }()
}
for i := 0; i < 10; i++ {
    result := <-results
    // process result
}
```

**Default is unbuffered:**

```go
ch := make(chan int)  // Unbuffered (capacity 0)
```

#### **Closing Channels**

```go
ch := make(chan int)

// Close channel (only sender should close)
close(ch)

// Receive from closed channel
value, ok := <-ch
// ok is false if channel is closed and empty
// value is zero value of channel type
```

**Important rules:**
1. **Only sender closes**: Receiver should never close
2. **Close once**: Closing already-closed channel panics
3. **Don't send on closed**: Sending on closed channel panics
4. **Receive from closed OK**: Returns zero value and ok=false

```go
// ❌ WRONG: Sending on closed channel panics
ch := make(chan int)
close(ch)
ch <- 42  // PANIC: send on closed channel

// ❌ WRONG: Closing closed channel panics
close(ch)
close(ch)  // PANIC: close of closed channel

// ✅ OK: Receiving from closed channel
ch := make(chan int)
close(ch)
value, ok := <-ch  // value=0, ok=false
```

**When to close:**

```go
// ✅ Signal "no more values"
func generateNumbers(max int) <-chan int {
    ch := make(chan int)
    go func() {
        defer close(ch)  // Signal completion
        for i := 0; i < max; i++ {
            ch <- i
        }
    }()
    return ch
}

// Consumer knows when to stop
for num := range generateNumbers(10) {
    fmt.Println(num)
}  // Loop ends when channel closed
```

**Closing is optional:**
- If no one is waiting for "no more values" signal, don't need to close
- Garbage collector will clean up unreferenced channels
- Close mainly for signaling completion

#### **Range Over Channels**

```go
ch := make(chan int)

go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)  // Must close for range to exit
}()

// Receive until channel is closed
for value := range ch {
    fmt.Println(value)
}
// Prints: 0, 1, 2, 3, 4
```

**Without close, range blocks forever:**

```go
// ❌ WRONG: Forgot to close, range blocks forever
go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    // Oops, forgot close(ch)
}()

for value := range ch {
    fmt.Println(value)
}  // Deadlock! Waits forever
```

#### **Channel Directions**

Channels can be restricted to send-only or receive-only:

```go
// Bidirectional (default)
func worker(ch chan int) {
    ch <- 42    // Can send
    val := <-ch // Can receive
}

// Send-only
func producer(ch chan<- int) {
    ch <- 42    // Can send
    // val := <-ch  // Compile error! Cannot receive
}

// Receive-only
func consumer(ch <-chan int) {
    val := <-ch // Can receive
    // ch <- 42   // Compile error! Cannot send
}

// Usage
func main() {
    ch := make(chan int)  // Bidirectional
    
    go producer(ch)  // Implicitly converts to chan<-
    go consumer(ch)  // Implicitly converts to <-chan
}
```

**Benefits:**
- **Type safety**: Compiler prevents misuse
- **Intent documentation**: Makes API clear
- **Prevents bugs**: Can't accidentally send on receive-only channel

**Pattern: Producer-Consumer**

```go
func produce(out chan<- int) {
    defer close(out)
    for i := 0; i < 10; i++ {
        out <- i
    }
}

func consume(in <-chan int) {
    for value := range in {
        fmt.Println(value)
    }
}

func main() {
    ch := make(chan int)
    go produce(ch)
    consume(ch)
}
```

#### **Select Statement**

`select` lets you wait on multiple channel operations:

```go
select {
case msg1 := <-ch1:
    fmt.Println("Received from ch1:", msg1)
case msg2 := <-ch2:
    fmt.Println("Received from ch2:", msg2)
case ch3 <- 42:
    fmt.Println("Sent to ch3")
default:
    fmt.Println("No channel ready")
}
```

**How select works:**
1. All channels are evaluated
2. If multiple are ready, one is chosen at random
3. If none are ready and there's a default, default executes
4. If none are ready and no default, blocks until one is ready

**Examples:**

```go
// Timeout pattern
select {
case result := <-ch:
    fmt.Println("Got result:", result)
case <-time.After(time.Second):
    fmt.Println("Timeout!")
}

// Non-blocking receive
select {
case msg := <-ch:
    fmt.Println("Received:", msg)
default:
    fmt.Println("No message ready")
}

// Non-blocking send
select {
case ch <- value:
    fmt.Println("Sent successfully")
default:
    fmt.Println("Channel full, couldn't send")
}

// Context cancellation
select {
case result := <-resultCh:
    return result, nil
case <-ctx.Done():
    return nil, ctx.Err()
}
```

**Random selection when multiple ready:**

```go
ch1 := make(chan int, 1)
ch2 := make(chan int, 1)

ch1 <- 1
ch2 <- 2

select {
case <-ch1:
    fmt.Println("ch1")
case <-ch2:
    fmt.Println("ch2")
}
// Prints "ch1" OR "ch2" (random if both ready)
```

#### **Channel Axioms**

These are fundamental truths about channels:

```go
// 1. Send on nil channel blocks forever
var ch chan int = nil
ch <- 42  // Blocks forever

// 2. Receive on nil channel blocks forever
var ch chan int = nil
<-ch  // Blocks forever

// 3. Send on closed channel panics
ch := make(chan int)
close(ch)
ch <- 42  // PANIC

// 4. Receive on closed channel returns zero value immediately
ch := make(chan int)
close(ch)
value := <-ch  // Returns 0 immediately (zero value for int)

// 5. Close nil channel panics
var ch chan int = nil
close(ch)  // PANIC

// 6. Close closed channel panics
ch := make(chan int)
close(ch)
close(ch)  // PANIC
```

**Useful trick: Disable channel in select**

```go
ch1 := make(chan int)
ch2 := make(chan int)

// Disable ch1 by setting to nil
ch1 = nil

select {
case v := <-ch1:  // Never chosen (nil channel)
    fmt.Println(v)
case v := <-ch2:  // Only this can be chosen
    fmt.Println(v)
}
```

This is useful for conditionally disabling cases in select.

### Channel Patterns

#### **Pipeline Pattern**

Chain operations through channels:

```go
func generator(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

func main() {
    // Pipeline: generate -> square
    nums := generator(1, 2, 3, 4, 5)
    squares := square(nums)
    
    for s := range squares {
        fmt.Println(s)  // 1, 4, 9, 16, 25
    }
}
```

**Multi-stage pipeline:**

```go
func main() {
    // generate -> square -> sum
    nums := generator(1, 2, 3, 4, 5)
    squares := square(nums)
    doubled := double(squares)
    
    for d := range doubled {
        fmt.Println(d)  // 2, 8, 18, 32, 50
    }
}

func double(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * 2
        }
    }()
    return out
}
```

#### **Fan-Out Pattern**

Multiple workers process from one input channel:

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job)
        time.Sleep(time.Second)
        results <- job * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // Start 3 workers (fan-out)
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // Send 9 jobs
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Collect results
    for r := 1; r <= 9; r++ {
        fmt.Println("Result:", <-results)
    }
}
```

#### **Fan-In Pattern**

Multiple inputs into one output channel:

```go
func fanIn(inputs ...<-chan int) <-chan int {
    out := make(chan int)
    
    var wg sync.WaitGroup
    wg.Add(len(inputs))
    
    for _, in := range inputs {
        go func(ch <-chan int) {
            defer wg.Done()
            for v := range ch {
                out <- v
            }
        }(in)
    }
    
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    ch3 := make(chan int)
    
    go func() {
        defer close(ch1)
        for i := 0; i < 3; i++ {
            ch1 <- i
        }
    }()
    
    go func() {
        defer close(ch2)
        for i := 10; i < 13; i++ {
            ch2 <- i
        }
    }()
    
    go func() {
        defer close(ch3)
        for i := 20; i < 23; i++ {
            ch3 <- i
        }
    }()
    
    // Merge all inputs
    merged := fanIn(ch1, ch2, ch3)
    
    for v := range merged {
        fmt.Println(v)  // Values from all channels, interleaved
    }
}
```

#### **Worker Pool Pattern**

Fixed number of workers processing jobs:

```go
type Job struct {
    ID   int
    Data string
}

type Result struct {
    JobID int
    Value string
}

func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job.ID)
        
        // Simulate work
        time.Sleep(time.Millisecond * 500)
        
        results <- Result{
            JobID: job.ID,
            Value: strings.ToUpper(job.Data),
        }
    }
}

func main() {
    numWorkers := 3
    numJobs := 10
    
    jobs := make(chan Job, numJobs)
    results := make(chan Result, numJobs)
    
    var wg sync.WaitGroup
    wg.Add(numWorkers)
    
    // Start worker pool
    for w := 1; w <= numWorkers; w++ {
        go worker(w, jobs, results, &wg)
    }
    
    // Send jobs
    for j := 1; j <= numJobs; j++ {
        jobs <- Job{ID: j, Data: fmt.Sprintf("job-%d", j)}
    }
    close(jobs)
    
    // Wait for workers to finish
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect results
    for result := range results {
        fmt.Printf("Result %d: %s\n", result.JobID, result.Value)
    }
}
```

#### **Timeout Pattern**

```go
func queryWithTimeout(query string, timeout time.Duration) (string, error) {
    result := make(chan string, 1)
    
    go func() {
        // Simulate slow query
        time.Sleep(time.Second * 2)
        result <- "query result"
    }()
    
    select {
    case res := <-result:
        return res, nil
    case <-time.After(timeout):
        return "", errors.New("query timeout")
    }
}

func main() {
    result, err := queryWithTimeout("SELECT * FROM users", time.Second)
    if err != nil {
        fmt.Println("Error:", err)  // "query timeout"
    } else {
        fmt.Println("Result:", result)
    }
}
```

#### **Rate Limiting Pattern**

```go
func rateLimiter(requests <-chan int, rate time.Duration) {
    ticker := time.NewTicker(rate)
    defer ticker.Stop()
    
    for req := range requests {
        <-ticker.C  // Wait for ticker
        fmt.Printf("Processing request %d at %v\n", req, time.Now())
    }
}

func main() {
    requests := make(chan int, 10)
    
    // Send 5 requests
    for i := 1; i <= 5; i++ {
        requests <- i
    }
    close(requests)
    
    // Process at rate of 1 per second
    rateLimiter(requests, time.Second)
}
```

**Token bucket rate limiter:**

```go
func tokenBucketLimiter(limit int, refillRate time.Duration) <-chan time.Time {
    bucket := make(chan time.Time, limit)
    
    // Fill initial bucket
    for i := 0; i < limit; i++ {
        bucket <- time.Now()
    }
    
    // Refill periodically
    go func() {
        ticker := time.NewTicker(refillRate)
        defer ticker.Stop()
        
        for t := range ticker.C {
            select {
            case bucket <- t:
            default:  // Bucket full, discard token
            }
        }
    }()
    
    return bucket
}

func main() {
    limiter := tokenBucketLimiter(3, time.Second)
    
    for i := 1; i <= 10; i++ {
        <-limiter  // Wait for token
        fmt.Printf("Request %d at %v\n", i, time.Now())
    }
}
```

#### **Context Cancellation Pattern**

The `context` package is essential for managing goroutine lifecycles:

```go
func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d stopped: %v\n", id, ctx.Err())
            return
        default:
            // Do work
            fmt.Printf("Worker %d working\n", id)
            time.Sleep(time.Second)
        }
    }
}

func main() {
    // Create cancellable context
    ctx, cancel := context.WithCancel(context.Background())
    
    // Start workers
    for i := 1; i <= 3; i++ {
        go worker(ctx, i)
    }
    
    // Let them work for a bit
    time.Sleep(time.Second * 3)
    
    // Cancel all workers
    cancel()
    
    time.Sleep(time.Second)
    fmt.Println("All workers stopped")
}
```

#### **Or-Channel Pattern**

Wait for the first of multiple channels:

```go
func or(channels ...<-chan interface{}) <-chan interface{} {
    switch len(channels) {
    case 0:
        return nil
    case 1:
        return channels[0]
    }
    
    orDone := make(chan interface{})
    go func() {
        defer close(orDone)
        
        switch len(channels) {
        case 2:
            select {
            case <-channels[0]:
            case <-channels[1]:
            }
        default:
            select {
            case <-channels[0]:
            case <-channels[1]:
            case <-channels[2]:
            case <-or(append(channels[3:], orDone)...):
            }
        }
    }()
    return orDone
}

func main() {
    sig := func(after time.Duration) <-chan interface{} {
        c := make(chan interface{})
        go func() {
            defer close(c)
            time.Sleep(after)
        }()
        return c
    }
    
    start := time.Now()
    <-or(
        sig(2*time.Second),
        sig(5*time.Second),
        sig(1*time.Second),  // This completes first
        sig(1*time.Minute),
    )
    
    fmt.Printf("Done after %v\n", time.Since(start))
    // Done after ~1s
}
```

### Frequently Asked Questions (Concurrency Fundamentals)

**Q1: What's the difference between concurrency and parallelism?**

**A:** 

- **Concurrency**: Structure - multiple tasks can be in progress at the same time
- **Parallelism**: Execution - multiple tasks actually running simultaneously

```go
// Concurrent but not parallel (1 CPU core)
func main() {
    runtime.GOMAXPROCS(1)  // Use only 1 OS thread
    
    go task1()  // Concurrent with task2
    go task2()  // But run serially on 1 core
    
    time.Sleep(time.Second)
}

// Concurrent AND parallel (multiple CPU cores)
func main() {
    runtime.GOMAXPROCS(4)  // Use 4 OS threads
    
    go task1()  // Can run in parallel
    go task2()  // on different cores
    
    time.Sleep(time.Second)
}
```

**Why This Matters:** You write concurrent code, Go runtime handles parallelism based on GOMAXPROCS and available cores.

**Related Concepts:** Go scheduler, GOMAXPROCS, CPU cores vs threads

---

**Q2: When should I use buffered vs unbuffered channels?**

**A:**

**Use unbuffered** when:
- You need synchronization (sender and receiver must meet)
- Signaling completion or events
- One-to-one communication

**Use buffered** when:
- You know the capacity needed
- Want to reduce goroutine blocking
- Implementing a queue or pool
- Multiple producers/consumers

```go
// ✅ Unbuffered: Synchronization needed
done := make(chan bool)
go func() {
    doWork()
    done <- true  // Signal and wait for receiver
}()
<-done  // Sender and receiver synchronize here

// ✅ Buffered: Known capacity, prevent blocking
jobs := make(chan Job, 100)
for i := 0; i < 100; i++ {
    jobs <- Job{ID: i}  // Won't block until buffer full
}
```

**Why This Matters:** Wrong choice can cause deadlocks or performance issues.

---

**Q3: How do I know if I have a goroutine leak?**

**A:**

Signs of goroutine leaks:
1. Memory usage grows over time
2. `runtime.NumGoroutine()` keeps increasing
3. pprof shows goroutines stuck in same state

**Detection:**

```go
// In tests
func TestForLeaks(t *testing.T) {
    before := runtime.NumGoroutine()
    
    runMyCode()
    
    time.Sleep(100 * time.Millisecond)  // Give time to cleanup
    after := runtime.NumGoroutine()
    
    if after > before {
        t.Errorf("Leaked %d goroutines", after-before)
    }
}

// Using goleak package
import "go.uber.org/goleak"

func TestNoLeaks(t *testing.T) {
    defer goleak.VerifyNone(t)
    runMyCode()
}

// Using pprof
import _ "net/http/pprof"

// Visit http://localhost:6060/debug/pprof/goroutine
http.ListenAndServe(":6060", nil)
```

**Common causes:**
- Channel never receives/sends
- Infinite loop without exit
- Blocked on mutex/lock
- Context never cancelled

---

**Q4: Why is sending on a closed channel a panic but receiving is not?**

**A:**

This design prevents bugs:

**Sending on closed channel panics:**
```go
ch := make(chan int)
close(ch)
ch <- 42  // PANIC - clear indication of programmer error
```

**Why panic?** The sender thinks it's sending data, but receiver will never get it. This is almost always a bug, so panic catches it immediately.

**Receiving from closed channel returns zero value:**
```go
ch := make(chan int)
close(ch)
value, ok := <-ch  // value=0, ok=false - graceful handling
```

**Why not panic?** Receiver checking for "no more values" is normal operation. The `ok` boolean indicates channel status.

**Pattern:**
```go
// Producer closes when done
go func() {
    defer close(ch)
    for i := 0; i < 10; i++ {
        ch <- i
    }
}()

// Consumer receives until closed
for value := range ch {  // Stops when channel closed
    process(value)
}
```

---

**Q5: What happens if I don't close a channel?**

**A:**

**Nothing bad immediately:**
- Channel won't leak memory if unreferenced
- Garbage collector will clean it up
- Goroutines will continue working

**But you might cause issues:**

```go
// ❌ PROBLEM: range never exits
go func() {
    for i := 0; i < 10; i++ {
        ch <- i
    }
    // Forgot to close!
}()

for v := range ch {  // Blocks forever after receiving 10 values
    fmt.Println(v)
}
```

**When to close:**
- To signal "no more values" to receivers
- When using `range` over channel
- When you want receivers to exit gracefully

**When NOT to close:**
- If no one is waiting for completion signal
- If multiple goroutines might try to close (use sync.Once)
- If you're not sure all sends are done

---

### Interview Questions (Concurrency Fundamentals)

**Question 1: What will this code print? Will it deadlock?**

```go
func main() {
    ch := make(chan int)
    ch <- 42
    fmt.Println(<-ch)
}
```

**Difficulty:** Junior

**Answer:**

This code will **deadlock** and panic with:
```
fatal error: all goroutines are asleep - deadlock!
```

**Why:**
1. `ch` is an unbuffered channel
2. `ch <- 42` tries to send, but blocks waiting for a receiver
3. No goroutine is receiving, so sender blocks forever
4. Go runtime detects all goroutines are blocked and panics

**Fix:**

```go
// Solution 1: Use buffered channel
ch := make(chan int, 1)
ch <- 42
fmt.Println(<-ch)  // Works!

// Solution 2: Send in goroutine
ch := make(chan int)
go func() {
    ch <- 42
}()
fmt.Println(<-ch)  // Works!

// Solution 3: Receive in goroutine
ch := make(chan int)
go func() {
    fmt.Println(<-ch)
}()
ch <- 42  // Works!
```

**What Interviewers Look For:**
- Understanding of unbuffered channel blocking behavior
- Knowledge that sends and receives must be in different goroutines (for unbuffered)
- Ability to identify and fix deadlocks

**Follow-up Questions:**
- What if the channel had buffer size 1?
- How does Go detect deadlocks?
- What's the difference between buffered and unbuffered channels?

---

**Question 2: Explain the output and fix the bug in this code:**

```go
func main() {
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Println(i)
        }()
    }
    
    wg.Wait()
}
```

**Difficulty:** Mid-Level

**Answer:**

**Output:** Most likely prints `5` five times (or some combination of 5s and other numbers due to race condition).

**Why:**
- Loop variable `i` is shared across all goroutines
- Goroutines likely start after loop finishes
- When they read `i`, it has the final value: 5
- This is a **race condition** on the variable `i`

**Detection:**
```bash
go run -race main.go
# WARNING: DATA RACE
# Read at 0x... by goroutine ...
```

**Fixes:**

```go
// ✅ Solution 1: Pass i as parameter
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(val int) {
        defer wg.Done()
        fmt.Println(val)
    }(i)  // Pass i's current value
}

// ✅ Solution 2: Create new variable in loop
for i := 0; i < 5; i++ {
    wg.Add(1)
    i := i  // Create new i for this iteration
    go func() {
        defer wg.Done()
        fmt.Println(i)
    }()
}

// ✅ Solution 3: Use index from range (if applicable)
values := []int{0, 1, 2, 3, 4}
for _, i := range values {
    wg.Add(1)
    go func(val int) {
        defer wg.Done()
        fmt.Println(val)
    }(i)
}
```

**What Interviewers Look For:**
- Recognition of closure variable capture
- Understanding of goroutine execution timing
- Knowledge of race detection
- Ability to provide multiple solutions

**Follow-up Questions:**
- How would you detect this race condition?
- Does Go 1.22 change this behavior?
- What's the difference between the solutions?

---

**Question 3: Implement a function that runs multiple tasks concurrently and returns when the first one completes.**

**Difficulty:** Mid-Level

**Answer:**

```go
func runFirst(tasks ...func() interface{}) interface{} {
    result := make(chan interface{}, len(tasks))
    
    for _, task := range tasks {
        task := task  // Capture task for goroutine
        go func() {
            result <- task()
        }()
    }
    
    return <-result  // Return first result
}

// Usage
func main() {
    result := runFirst(
        func() interface{} {
            time.Sleep(3 * time.Second)
            return "Task 1"
        },
        func() interface{} {
            time.Sleep(1 * time.Second)
            return "Task 2"  // This completes first
        },
        func() interface{} {
            time.Sleep(2 * time.Second)
            return "Task 3"
        },
    )
    
    fmt.Println("First result:", result)
    // Output: First result: Task 2
}
```

**Better version with context for cleanup:**

```go
func runFirstWithCleanup(ctx context.Context, tasks ...func(context.Context) interface{}) interface{} {
    result := make(chan interface{}, len(tasks))
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()  // Cancel other tasks when first completes
    
    for _, task := range tasks {
        task := task
        go func() {
            select {
            case result <- task(ctx):
            case <-ctx.Done():
                return  // Exit if already cancelled
            }
        }()
    }
    
    return <-result
}
```

**What Interviewers Look For:**
- Understanding of channel buffering (why buffered here?)
- Goroutine management
- Context usage for cancellation
- Resource cleanup considerations

**Follow-up Questions:**
- What if all tasks fail? How would you handle that?
- How would you return errors?
- What if you want the first N results?
- How would you add a timeout?

---

**Question 4: What's wrong with this worker pool implementation?**

```go
func workerPool(jobs []int) []int {
    results := make(chan int)
    
    for _, job := range jobs {
        go func(j int) {
            results <- j * 2
        }(job)
    }
    
    var output []int
    for range jobs {
        output = append(output, <-results)
    }
    
    return output
}
```

**Difficulty:** Mid-Level

**Answer:**

**Problems:**

1. **Unbuffered channel with no guarantee of receiver being ready**
   - If collector loop hasn't started, senders block
   - Usually works but has timing dependency

2. **No error handling**
   - What if a job fails?

3. **No way to limit concurrency**
   - If `jobs` has 1 million items, creates 1 million goroutines
   - Could exhaust resources

4. **No worker pool** - creates a goroutine per job
   - True worker pool reuses goroutines

**Better implementation:**

```go
func workerPool(jobs []int, numWorkers int) []int {
    jobsCh := make(chan int, len(jobs))
    results := make(chan int, len(jobs))
    
    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobsCh {
                results <- job * 2
            }
        }()
    }
    
    // Send jobs
    for _, job := range jobs {
        jobsCh <- job
    }
    close(jobsCh)
    
    // Close results when all workers done
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect results
    var output []int
    for result := range results {
        output = append(output, result)
    }
    
    return output
}
```

**What Interviewers Look For:**
- Understanding of worker pool pattern
- Knowledge of goroutine lifecycle
- Resource management awareness
- Ability to identify and fix concurrency issues

---

### Key Takeaways (Concurrency Fundamentals)

✅ **Goroutines are lightweight**
- 2KB initial stack vs 1-2MB for OS threads
- Can create millions easily
- Managed by Go runtime scheduler

✅ **Channels enable safe communication**
- Unbuffered: synchronization between goroutines
- Buffered: asynchronous communication with bounded capacity
- Use for "share memory by communicating"

✅ **Common patterns solve real problems**
- Pipeline: chain operations
- Fan-out/Fan-in: distribute and collect work
- Worker pool: limit concurrency
- Rate limiting: control throughput
- Timeouts: prevent hanging

✅ **Watch out for common pitfalls**
- Loop variable capture in goroutines
- Goroutine leaks (no way to stop)
- Sending on closed channels (panic)
- Deadlocks on unbuffered channels

✅ **Use context for cancellation**
- Propagate cancellation through call stack
- Set timeouts and deadlines
- Clean up resources properly

---

## 2.2 Synchronization Primitives

While channels are great for communication, sometimes you need lower-level synchronization primitives. The `sync` package provides mutexes, wait groups, and other tools.

### sync.Mutex

A mutex (mutual exclusion) protects shared data from concurrent access.

#### **Basic Mutex Usage**

```go
import "sync"

type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

func main() {
    var counter Counter
    
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    
    wg.Wait()
    fmt.Println("Final count:", counter.Value())  // 1000
}
```

**Without mutex (race condition):**

```go
type Counter struct {
    count int  // No mutex!
}

func (c *Counter) Increment() {
    c.count++  // ❌ RACE CONDITION
}

// Running with -race flag shows:
// WARNING: DATA RACE
```

#### **Mutex Rules**

1. **Always use defer for Unlock**

```go
// ✅ GOOD: Unlock even if panic
func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
    // If panic here, mutex still unlocked
}

// ❌ BAD: Might not unlock on panic
func (c *Counter) Increment() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()  // Might not reach if panic above
}
```

2. **Don't copy mutex**

```go
type Counter struct {
    mu    sync.Mutex
    count int
}

// ❌ BAD: Copies mutex
func (c Counter) Increment() {  // Value receiver copies c and c.mu
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

// ✅ GOOD: Pointer receiver doesn't copy
func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}
```

3. **Keep critical sections small**

```go
// ❌ BAD: Lock held too long
func (c *Counter) Process() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    time.Sleep(time.Second)  // Expensive operation under lock!
    c.count++
}

// ✅ GOOD: Minimize time under lock
func (c *Counter) Process() {
    time.Sleep(time.Second)  // Do expensive work outside lock
    
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}
```

4. **Avoid deadlocks**

```go
// ❌ DEADLOCK: Lock same mutex twice
func (c *Counter) Bad() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.Value()  // Tries to lock again - DEADLOCK!
}

// ✅ GOOD: Extract unlocked helper
func (c *Counter) value() int {
    return c.count  // No lock
}

func (c *Counter) ValueLocked() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value()
}
```

### sync.RWMutex

Read-write mutex allows multiple readers OR one writer:

```go
type Cache struct {
    mu    sync.RWMutex
    items map[string]string
}

func NewCache() *Cache {
    return &Cache{
        items: make(map[string]string),
    }
}

// Multiple readers can call this simultaneously
func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()  // Read lock
    defer c.mu.RUnlock()
    
    value, ok := c.items[key]
    return value, ok
}

// Only one writer at a time, blocks all readers
func (c *Cache) Set(key, value string) {
    c.mu.Lock()  // Write lock
    defer c.mu.Unlock()
    
    c.items[key] = value
}

func main() {
    cache := NewCache()
    
    // Many concurrent readers (OK, all can read simultaneously)
    for i := 0; i < 100; i++ {
        go func(id int) {
            cache.Get("key")
        }(i)
    }
    
    // One writer (blocks until all readers done)
    cache.Set("key", "value")
}
```

**When to use RWMutex:**
- ✅ When reads greatly outnumber writes
- ✅ When read operations are expensive
- ❌ When writes are frequent (mutex is faster due to less overhead)
- ❌ When critical section is tiny (mutex overhead dominates)

**Benchmark comparison:**

```go
// Read-heavy workload (90% reads, 10% writes)
// RWMutex: ~5x faster than Mutex

// Write-heavy workload (50% reads, 50% writes)
// Mutex: ~2x faster than RWMutex
```

Continuing with Part 2...
### sync.WaitGroup

We've seen WaitGroup before, but let's explore it deeply:

```go
type WaitGroup struct {
    // Internal state
    // Don't copy!
}

// Add increments the counter
func (wg *WaitGroup) Add(delta int)

// Done decrements the counter (equivalent to Add(-1))
func (wg *WaitGroup) Done()

// Wait blocks until counter reaches 0
func (wg *WaitGroup) Wait()
```

#### **WaitGroup Patterns**

**Pattern 1: Waiting for goroutines to complete**

```go
func processItems(items []int) {
    var wg sync.WaitGroup
    
    for _, item := range items {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            process(i)
        }(item)
    }
    
    wg.Wait()  // Wait for all to complete
}
```

**Pattern 2: Reusable WaitGroup**

```go
type TaskRunner struct {
    wg sync.WaitGroup
}

func (tr *TaskRunner) Go(f func()) {
    tr.wg.Add(1)
    go func() {
        defer tr.wg.Done()
        f()
    }()
}

func (tr *TaskRunner) Wait() {
    tr.wg.Wait()
}

// Usage
func main() {
    var runner TaskRunner
    
    for i := 0; i < 10; i++ {
        runner.Go(func() {
            // Do work
        })
    }
    
    runner.Wait()
}
```

**Common WaitGroup mistakes:**

```go
// ❌ WRONG: Add inside goroutine (race condition)
for i := 0; i < 5; i++ {
    go func() {
        wg.Add(1)  // Race! Wait() might be called before this
        defer wg.Done()
        // work...
    }()
}
wg.Wait()

// ✅ CORRECT: Add before starting goroutine
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        // work...
    }()
}
wg.Wait()

// ❌ WRONG: Copying WaitGroup
wg1 := sync.WaitGroup{}
wg2 := wg1  // Copy! State is now inconsistent

// ✅ CORRECT: Pass by pointer
func doWork(wg *sync.WaitGroup) {
    defer wg.Done()
    // work...
}

wg := &sync.WaitGroup{}
wg.Add(1)
go doWork(wg)
```

### sync.Once

Ensures a function is executed only once, even with concurrent calls:

```go
var once sync.Once

func setup() {
    fmt.Println("Setting up")
}

func main() {
    for i := 0; i < 10; i++ {
        go func() {
            once.Do(setup)  // Only prints "Setting up" once
        }()
    }
    
    time.Sleep(time.Second)
}
```

**Common use case: Lazy initialization**

```go
type Database struct {
    conn *sql.DB
}

var (
    dbInstance *Database
    dbOnce     sync.Once
)

func GetDB() *Database {
    dbOnce.Do(func() {
        conn, err := sql.Open("mysql", "connection-string")
        if err != nil {
            panic(err)  // Or handle error appropriately
        }
        dbInstance = &Database{conn: conn}
    })
    return dbInstance
}

// Multiple goroutines can call GetDB() safely
// Connection opened only once
```

**Singleton pattern:**

```go
type Config struct {
    APIKey string
    Secret string
}

var (
    config     *Config
    configOnce sync.Once
)

func GetConfig() *Config {
    configOnce.Do(func() {
        // Load config from file/env
        config = &Config{
            APIKey: os.Getenv("API_KEY"),
            Secret: os.Getenv("SECRET"),
        }
    })
    return config
}
```

**Important:** If the function passed to `Do()` panics, `Once` considers it executed and won't retry:

```go
var once sync.Once

once.Do(func() {
    panic("oops")  // Panics
})

// This will NOT execute - Once thinks it's done
once.Do(func() {
    fmt.Println("This never runs")
})
```

### sync.Pool

Object pool for reusing objects to reduce GC pressure:

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)  // Create new buffer if pool empty
    },
}

func processData(data string) {
    // Get buffer from pool
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf)  // Return to pool when done
    
    // Reset buffer for reuse
    buf.Reset()
    
    // Use buffer
    buf.WriteString(data)
    // process buf...
}
```

**When to use sync.Pool:**

```go
// ✅ GOOD: Temporary objects allocated frequently
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

// ✅ GOOD: Expensive to create objects
var connectionPool = sync.Pool{
    New: func() interface{} {
        conn, _ := net.Dial("tcp", "localhost:8080")
        return conn
    },
}

// ❌ BAD: Objects that shouldn't be shared
var userPool = sync.Pool{  // Don't do this!
    New: func() interface{} {
        return &User{}  // State shouldn't be shared
    },
}
```

**Important Pool characteristics:**

1. **No guarantees**: Objects can be removed from pool at any time (GC clears pool)
2. **Must reset**: Always reset objects before returning to pool
3. **Thread-safe**: Pool handles synchronization
4. **Per-P caching**: Pool has per-processor cache for efficiency

```go
type Job struct {
    Data []byte
}

var jobPool = sync.Pool{
    New: func() interface{} {
        return &Job{
            Data: make([]byte, 0, 1024),
        }
    },
}

func processJob(data []byte) {
    job := jobPool.Get().(*Job)
    defer func() {
        job.Data = job.Data[:0]  // Reset slice
        jobPool.Put(job)
    }()
    
    job.Data = append(job.Data, data...)
    // Process job...
}
```

### sync.Map

Concurrent map for specific use cases:

```go
var m sync.Map

// Store
m.Store("key", "value")

// Load
value, ok := m.Load("key")

// LoadOrStore (atomic)
actual, loaded := m.LoadOrStore("key", "value")
if loaded {
    // Key existed, actual is old value
} else {
    // Key didn't exist, stored new value
}

// Delete
m.Delete("key")

// Range (iterate)
m.Range(func(key, value interface{}) bool {
    fmt.Printf("%v: %v\n", key, value)
    return true  // continue iteration (false = stop)
})
```

**When to use sync.Map:**

```go
// ✅ USE SYNC.MAP when:
// 1. Key only written once but read many times
cache := &sync.Map{}
cache.Store(key, value)  // Write once
// ... many reads
value, _ := cache.Load(key)

// 2. Multiple goroutines read/write disjoint keys
// (each goroutine has its own set of keys)

// ❌ DON'T USE SYNC.MAP when:
// Keys are frequently written and read by same goroutines
// → Use map with mutex instead (faster)

// Example: Better with regular map + mutex
type Cache struct {
    mu    sync.RWMutex
    items map[string]interface{}
}
```

**Performance comparison:**

```go
// Benchmark: Read-heavy workload (90% reads, 10% writes)
// sync.Map: ~50% faster than map+RWMutex
// 
// Benchmark: Write-heavy workload (50% reads, 50% writes)
// map+RWMutex: ~2x faster than sync.Map
```

### sync.Cond

Condition variable for waiting for/signaling events:

```go
type Queue struct {
    mu    sync.Mutex
    cond  *sync.Cond
    items []int
}

func NewQueue() *Queue {
    q := &Queue{}
    q.cond = sync.NewCond(&q.mu)
    return q
}

func (q *Queue) Enqueue(item int) {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    q.items = append(q.items, item)
    q.cond.Signal()  // Wake one waiting goroutine
}

func (q *Queue) Dequeue() int {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    for len(q.items) == 0 {
        q.cond.Wait()  // Wait for signal
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item
}
```

**Cond is rarely needed** - channels are usually better:

```go
// Instead of sync.Cond:
type Queue struct {
    ch chan int
}

func NewQueue(size int) *Queue {
    return &Queue{ch: make(chan int, size)}
}

func (q *Queue) Enqueue(item int) {
    q.ch <- item
}

func (q *Queue) Dequeue() int {
    return <-q.ch
}
```

### atomic Package

Atomic operations for lock-free programming:

```go
import "sync/atomic"

// Atomic integer operations
var counter int64

atomic.AddInt64(&counter, 1)      // Atomic increment
atomic.AddInt64(&counter, -1)     // Atomic decrement
value := atomic.LoadInt64(&counter)   // Atomic read
atomic.StoreInt64(&counter, 0)    // Atomic write
swapped := atomic.CompareAndSwapInt64(&counter, 0, 1)  // CAS

// Atomic pointer operations
var ptr unsafe.Pointer
atomic.StorePointer(&ptr, unsafe.Pointer(&someValue))
p := atomic.LoadPointer(&ptr)
```

**Use atomic for simple counters:**

```go
type Counter struct {
    value int64
}

func (c *Counter) Increment() {
    atomic.AddInt64(&c.value, 1)  // No mutex needed!
}

func (c *Counter) Value() int64 {
    return atomic.LoadInt64(&c.value)
}
```

**atomic.Value for arbitrary types:**

```go
type Config struct {
    Host string
    Port int
}

var config atomic.Value

// Write
config.Store(&Config{Host: "localhost", Port: 8080})

// Read
currentConfig := config.Load().(*Config)
fmt.Printf("Host: %s, Port: %d\n", currentConfig.Host, currentConfig.Port)

// Atomic update
func updateConfig(newConfig *Config) {
    config.Store(newConfig)
}
```

**Compare-and-swap pattern:**

```go
func (c *Counter) IncrementIfBelow(max int64) bool {
    for {
        current := atomic.LoadInt64(&c.value)
        if current >= max {
            return false  // Already at max
        }
        
        if atomic.CompareAndSwapInt64(&c.value, current, current+1) {
            return true  // Successfully incremented
        }
        // CAS failed, retry
    }
}
```

**When to use atomics vs mutexes:**

```go
// ✅ Use atomics for:
// - Simple counters
// - Flags
// - Reference counting
// - Single variables

var counter int64
atomic.AddInt64(&counter, 1)

// ✅ Use mutexes for:
// - Multiple related variables
// - Complex operations
// - When you need to read-modify-write multiple fields

type Stats struct {
    mu     sync.Mutex
    total  int
    errors int
    avg    float64
}

func (s *Stats) Update(value int, isError bool) {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    s.total++
    if isError {
        s.errors++
    }
    s.avg = (s.avg*float64(s.total-1) + float64(value)) / float64(s.total)
}
```

### Context Package

The `context` package is essential for managing goroutine lifecycles in production systems.

#### **Context Basics**

```go
import "context"

// Create root context
ctx := context.Background()  // Non-cancellable root
ctx := context.TODO()         // Use when unsure which context to use

// Create cancellable context
ctx, cancel := context.WithCancel(context.Background())
defer cancel()  // Always defer cancel

// Create context with timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// Create context with deadline
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()

// Store values in context (use sparingly!)
ctx = context.WithValue(ctx, "userID", 12345)
```

#### **Context Cancellation**

```go
func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d cancelled: %v\n", id, ctx.Err())
            return
        default:
            fmt.Printf("Worker %d working\n", id)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    
    for i := 1; i <= 3; i++ {
        go worker(ctx, i)
    }
    
    time.Sleep(2 * time.Second)
    cancel()  // Cancel all workers
    
    time.Sleep(time.Second)
}
```

#### **Context with Timeout**

```go
func queryDatabase(ctx context.Context, query string) (string, error) {
    // Simulate slow query
    resultCh := make(chan string, 1)
    
    go func() {
        time.Sleep(3 * time.Second)
        resultCh <- "query result"
    }()
    
    select {
    case result := <-resultCh:
        return result, nil
    case <-ctx.Done():
        return "", ctx.Err()  // context.DeadlineExceeded or context.Canceled
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    
    result, err := queryDatabase(ctx, "SELECT * FROM users")
    if err != nil {
        fmt.Println("Error:", err)  // context deadline exceeded
    } else {
        fmt.Println("Result:", result)
    }
}
```

#### **Context Values (Anti-pattern Warning)**

```go
type contextKey string

const userIDKey contextKey = "userID"

// ❌ AVOID: Using context for required parameters
func processUser(ctx context.Context) {
    userID := ctx.Value(userIDKey).(int)  // Bad!
    // What if value not in context? Panic!
}

// ✅ PREFER: Explicit parameters
func processUser(ctx context.Context, userID int) {
    // Clear and type-safe
}

// ✅ OK: Request-scoped values (trace IDs, request IDs)
func handler(w http.ResponseWriter, r *http.Request) {
    requestID := generateRequestID()
    ctx := context.WithValue(r.Context(), "requestID", requestID)
    
    // All downstream functions can access request ID for logging
    processRequest(ctx)
}
```

**Context rules:**

1. **Always pass context as first parameter**

```go
func DoWork(ctx context.Context, data string) error {
    // ...
}
```

2. **Don't store context in structs**

```go
// ❌ WRONG
type Worker struct {
    ctx context.Context  // Don't do this!
}

// ✅ CORRECT
type Worker struct {
    // No context
}

func (w *Worker) DoWork(ctx context.Context) {
    // Pass context as parameter
}
```

3. **Propagate cancellation down the call stack**

```go
func handler(ctx context.Context) error {
    return processData(ctx)  // Pass context down
}

func processData(ctx context.Context) error {
    return saveToDatabase(ctx)  // Keep passing
}

func saveToDatabase(ctx context.Context) error {
    // Check for cancellation
    if ctx.Err() != nil {
        return ctx.Err()
    }
    // Do work...
    return nil
}
```

#### **Context Best Practices**

```go
// ✅ GOOD: Check context before expensive operations
func processLargeFile(ctx context.Context, filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        // Check context before processing each line
        if ctx.Err() != nil {
            return ctx.Err()
        }
        
        processLine(scanner.Text())
    }
    return scanner.Err()
}

// ✅ GOOD: Use context with HTTP requests
func fetchData(ctx context.Context, url string) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    return io.ReadAll(resp.Body)
}

// ✅ GOOD: Cleanup on context cancellation
func worker(ctx context.Context) error {
    file, err := os.Create("temp.txt")
    if err != nil {
        return err
    }
    
    // Cleanup on cancellation
    go func() {
        <-ctx.Done()
        file.Close()
        os.Remove("temp.txt")
    }()
    
    // Do work...
    return nil
}
```

### Frequently Asked Questions (Synchronization)

**Q1: When should I use channels vs mutexes?**

**A:**

**Use channels when:**
- Transferring ownership of data
- Distributing work among goroutines
- Communicating results
- Need synchronization between goroutines

**Use mutexes when:**
- Protecting shared state/data structures
- Simple shared counters
- Caching
- Temporarily accessing shared resource

```go
// ✅ Channel: Ownership transfer
jobs := make(chan Job)
go worker(jobs)
jobs <- Job{ID: 1}  // Worker owns job now

// ✅ Mutex: Shared state
type Cache struct {
    mu    sync.RWMutex
    items map[string]interface{}
}
```

**Rule of thumb:** Use whichever makes the code simpler. Channels for communication, mutexes for state protection.

---

**Q2: What's the difference between sync.Mutex and sync.RWMutex?**

**A:**

**sync.Mutex:**
- Exclusive lock (only one goroutine at a time)
- Use for write operations or when reads are rare

**sync.RWMutex:**
- Read lock: Multiple readers allowed
- Write lock: Exclusive access
- Use when reads greatly outnumber writes

```go
// Benchmark: 90% reads, 10% writes
// RWMutex: ~5x faster

// Benchmark: 50% reads, 50% writes  
// Mutex: ~2x faster (less overhead)
```

---

**Q3: How does context cancellation propagate?**

**A:**

Context cancellation flows down the tree:

```go
root := context.Background()
ctx1, cancel1 := context.WithCancel(root)
ctx2, cancel2 := context.WithCancel(ctx1)
ctx3, cancel3 := context.WithCancel(ctx2)

cancel1()  // Cancels ctx1, ctx2, ctx3
// ctx2.Done() and ctx3.Done() are closed

// But:
cancel3()  // Only cancels ctx3
// ctx1 and ctx2 remain active
```

**Rule:** Cancelling parent cancels all children, but not vice versa.

---

### Interview Questions (Synchronization)

**Question 1: Implement a thread-safe map with get, set, and delete operations.**

**Difficulty:** Junior

**Answer:**

```go
type SafeMap struct {
    mu    sync.RWMutex
    items map[string]interface{}
}

func NewSafeMap() *SafeMap {
    return &SafeMap{
        items: make(map[string]interface{}),
    }
}

func (sm *SafeMap) Set(key string, value interface{}) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    sm.items[key] = value
}

func (sm *SafeMap) Get(key string) (interface{}, bool) {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    value, ok := sm.items[key]
    return value, ok
}

func (sm *SafeMap) Delete(key string) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    delete(sm.items, key)
}

func (sm *SafeMap) Len() int {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    return len(sm.items)
}
```

**What Interviewers Look For:**
- Correct use of RWMutex (read vs write locks)
- defer for unlock
- Pointer receiver
- Proper initialization

---

**Question 2: What's wrong with this code?**

```go
var counter int
var mu sync.Mutex

func increment() {
    mu.Lock()
    counter++
    mu.Unlock()
}

func getValue() int {
    return counter  // ❌
}
```

**Difficulty:** Junior

**Answer:**

**Problem:** `getValue()` reads `counter` without locking, causing a race condition.

**Fix:**

```go
func getValue() int {
    mu.Lock()
    defer mu.Unlock()
    return counter
}

// Or use RWMutex for better performance:
var mu sync.RWMutex

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}

func getValue() int {
    mu.RLock()
    defer mu.RUnlock()
    return counter
}
```

**Detection:**
```bash
go run -race main.go
# WARNING: DATA RACE
```

---

### Key Takeaways (Synchronization)

✅ **Choose the right primitive**
- Channels: communication and ownership transfer
- Mutex: protecting shared state
- RWMutex: read-heavy workloads
- atomic: simple counters and flags

✅ **Mutex best practices**
- Always defer Unlock
- Keep critical sections small
- Use pointer receivers
- Don't copy mutexes

✅ **Context for lifecycle management**
- Cancellation propagation
- Timeouts and deadlines
- Pass as first parameter
- Don't store in structs

✅ **sync.Pool for performance**
- Reduce GC pressure
- Reuse expensive objects
- Always reset before returning
- No guarantees (GC can clear pool)

✅ **atomics for lock-free**
- Simple counters
- Flags
- CAS for lock-free algorithms
- Faster than mutexes for single variables


## 2.3 Go Scheduler Internals

Understanding how Go schedules goroutines is crucial for writing high-performance concurrent code. The Go scheduler is a work-stealing, M:N scheduler that multiplexes goroutines onto OS threads.

### The GMP Model

The Go scheduler uses three main entities:

```
G (Goroutine)     M (Machine/OS Thread)     P (Processor/Context)
┌─────────────┐   ┌──────────────────┐      ┌─────────────────┐
│ Stack       │   │ Currently        │      │ Run queue       │
│ Instruction │   │ running G        │      │ [G][G][G]       │
│ pointer     │   │                  │      │                 │
│ State       │   │ Attached P       │      │ Free G list     │
└─────────────┘   └──────────────────┘      │ Available Gs    │
                                             └─────────────────┘
```

**G (Goroutine):**
- Represents a goroutine
- Contains stack, instruction pointer, and other state
- Lightweight: starts at ~2KB stack

**M (Machine):**
- OS thread
- Number can grow as needed
- Limited by GOMAXPROCS for running
- Some may block on syscalls

**P (Processor/Context):**
- Logical processor
- Number set by GOMAXPROCS (default = CPU cores)
- Contains run queue of goroutines
- Has free list of Gs

### How Scheduling Works

```
                    Global Run Queue
                   ┌────────────────┐
                   │ [G] [G] [G]    │
                   └────────────────┘
                          ↑ ↓
         ┌────────────────┴─────────────────┐
         │                                   │
    ┌────▼────┐  ┌─────────┐  ┌─────────┐  │
    │    P1   │  │   P2    │  │   P3    │  │
    │ [G][G]  │  │ [G][G]  │  │ [G][G]  │  │
    └────┬────┘  └────┬────┘  └────┬────┘  │
         │            │            │         │
    ┌────▼────┐  ┌────▼────┐  ┌───▼─────┐ │
    │   M1    │  │   M2    │  │   M3    │ │
    │ (Thread)│  │ (Thread)│  │ (Thread)│ │
    └─────────┘  └─────────┘  └─────────┘ │
                                           │
         OS Threads                        │
                                           │
    ┌──────────────────────────────────────┘
    │ Work Stealing: When P's queue empty,
    │ steal from other P's queues
    └──────────────────────────────────────
```

### Scheduler Decisions

**When does the scheduler run?**

1. **Explicit yield**: `runtime.Gosched()`
2. **Blocking operations**:
   - Channel operations
   - Network I/O
   - Blocking syscalls
   - Time.Sleep()
3. **Function calls** (potential preemption points)
4. **Async preemption** (Go 1.14+): Timer-based

```go
func compute() {
    // Long computation
    for i := 0; i < 1_000_000_000; i++ {
        // Before Go 1.14: This could hog CPU
        // After Go 1.14: Preempted after ~10ms
    }
}

func main() {
    runtime.GOMAXPROCS(1)  // 1 OS thread
    
    go compute()
    go compute()
    
    // Both goroutines get CPU time (preempted periodically)
    time.Sleep(time.Second)
}
```

### Work-Stealing Algorithm

When a P's local queue is empty:

1. Check global queue
2. Try to steal from other P's queues
3. Poll network (check for ready network connections)
4. If still nothing, park M (thread sleeps)

```go
// P1 has work, P2 is idle
P1: [G1] [G2] [G3] [G4]
P2: []

// P2 steals from P1
P1: [G1] [G2]
P2: [G3] [G4]

// Now both Ps have work
```

**Benefits:**
- Load balancing automatic
- No goroutine starvation
- Efficient CPU utilization

### Goroutine States

```
                Created
                   │
                   ▼
              Runnable ◄──────┐
                   │          │
                   ▼          │
              Running         │
                 │  │         │
                 │  └─────────┤
                 │            │
                 ▼            │
              Waiting         │
                 │            │
                 └────────────┘
                   │
                   ▼
                 Dead
```

**States:**
- **Created**: `go func()` called
- **Runnable**: Ready to run, in queue
- **Running**: Executing on M
- **Waiting**: Blocked (channel, syscall, etc.)
- **Dead**: Completed

### Blocking Syscalls

Go handles blocking syscalls specially:

```go
func readFile() {
    // Blocking syscall
    data, _ := os.ReadFile("large.txt")
    
    // What happens:
    // 1. M blocks on syscall
    // 2. P detaches from M
    // 3. P finds/creates new M to run other Gs
    // 4. When syscall completes, original M wakes up
    // 5. G becomes runnable, joins run queue
}
```

```
Before syscall:        During syscall:         After syscall:
┌───┐                  ┌───┐                   ┌───┐
│ P │                  │ P │──┐                │ P │
└─┬─┘                  └─┬─┘  │                └─┬─┘
  │                      │    │                  │
┌─▼─┐  Running G      ┌─▼─┐  │ Blocked        ┌─▼─┐
│ M │──┐              │M2 │  │ M1 blocked     │ M │
└───┘  │              └───┘  │ in syscall     └───┘
       │                     │                  │
     ┌─▼─┐                 ┌─▼─┐              ┌─▼─┐
     │ G │                 │ G │ waiting      │ G │ runnable
     └───┘                 └───┘              └───┘

P detaches, finds new M  │  G joins queue when
to continue running      │  syscall completes
other goroutines         │
```

### Network Poller

Go integrates network I/O with the scheduler via the **netpoller**:

```go
conn, _ := net.Dial("tcp", "example.com:80")
data, _ := conn.Read(buffer)  // Doesn't block OS thread!

// How it works:
// 1. Read() would block, so G parks
// 2. M doesn't block - continues running other Gs
// 3. netpoller (epoll/kqueue) monitors fd
// 4. When data ready, netpoller wakes G
// 5. G becomes runnable
```

**Netpoller is integrated with scheduler:**
- No OS thread blocked on I/O
- Efficient network operations
- Thousands of network connections OK

### GOMAXPROCS Tuning

```go
// Get current GOMAXPROCS
procs := runtime.GOMAXPROCS(0)

// Set GOMAXPROCS
runtime.GOMAXPROCS(4)  // Use 4 OS threads

// Or via environment variable
// GOMAXPROCS=8 go run main.go
```

**Guidelines:**

```go
// Default: Number of CPU cores
// Usually optimal for CPU-bound work

// Increase GOMAXPROCS when:
// - Lots of blocking syscalls
// - Mixed CPU and I/O work

// Don't increase much beyond CPU count for:
// - Pure CPU-bound work (creates overhead)

// Example: CPU-bound work
runtime.GOMAXPROCS(runtime.NumCPU())  // Default, usually best

// Example: I/O-heavy work
runtime.GOMAXPROCS(runtime.NumCPU() * 2)  // May help
```

### Scheduler Traces

Debug the scheduler with traces:

```go
import (
    "os"
    "runtime/trace"
)

func main() {
    f, _ := os.Create("trace.out")
    defer f.Close()
    
    trace.Start(f)
    defer trace.Stop()
    
    // Your code here
    runWorkload()
}

// Analyze trace:
// go tool trace trace.out
```

**What traces show:**
- Goroutine creation/destruction
- Blocking events
- GC pauses
- Syscalls
- Network events
- Scheduler decisions

### Scheduler Best Practices

```go
// ✅ GOOD: Let scheduler do its job
for i := 0; i < 1000; i++ {
    go processItem(items[i])
}
// Scheduler handles distribution

// ❌ BAD: Manual "load balancing"
numWorkers := runtime.NumCPU()
chunkSize := len(items) / numWorkers
for i := 0; i < numWorkers; i++ {
    go func(start, end int) {
        for j := start; j < end; j++ {
            processItem(items[j])
        }
    }(i*chunkSize, (i+1)*chunkSize)
}
// Unnecessary - scheduler handles this better

// ✅ GOOD: Worker pool for resource limits
jobs := make(chan Item, 100)
for i := 0; i < 10; i++ {  // 10 workers
    go worker(jobs)
}
for _, item := range items {
    jobs <- item
}
close(jobs)
```

### Frequently Asked Questions (Scheduler)

**Q1: What is GOMAXPROCS and should I change it?**

**A:**

GOMAXPROCS sets the number of OS threads that can execute user-level Go code simultaneously.

**Default:** Number of CPU cores (`runtime.NumCPU()`)

**When to change:**
- Usually don't need to
- Container environments might need explicit setting
- I/O-heavy workloads might benefit from higher value

```go
// Check current value
current := runtime.GOMAXPROCS(0)

// Set to specific value
runtime.GOMAXPROCS(8)

// Usually default is best
runtime.GOMAXPROCS(runtime.NumCPU())
```

---

**Q2: How does the Go scheduler differ from OS scheduler?**

**A:**

**OS Scheduler:**
- Schedules threads (heavyweight)
- Kernel-level (expensive context switch)
- Preemptive (timer interrupts)

**Go Scheduler:**
- Schedules goroutines (lightweight)
- User-level (cheap context switch)
- Cooperative with async preemption
- Work-stealing for load balancing

**Benefits:**
- Faster context switching
- Less memory (2KB vs 2MB stack)
- Better load balancing
- Integrated with Go runtime (GC, netpoller)

---

**Q3: Why do CPU-bound goroutines sometimes monopolize the CPU?**

**A:**

Before Go 1.14, CPU-bound goroutines could run indefinitely without yielding. Go 1.14+ introduced async preemption:

```go
// Before 1.14: Could starve other goroutines
func cpuBound() {
    for {
        // Pure computation, no function calls
        compute()  // Runs indefinitely on 1 core
    }
}

// After 1.14: Preempted after ~10ms
func cpuBound() {
    for {
        compute()  // Gets preempted periodically
    }
}
```

**Solutions:**
- Update to Go 1.14+ (async preemption)
- Add periodic `runtime.Gosched()` (manual yield)
- Use channels or other blocking operations
- Increase GOMAXPROCS

---

### Key Takeaways (Scheduler)

✅ **GMP Model**
- G: Goroutine (lightweight, 2KB initial stack)
- M: OS Thread (expensive, limited by GOMAXPROCS)
- P: Processor/Context (logical CPU, has run queue)

✅ **Work-stealing scheduler**
- Automatic load balancing
- Idle Ps steal work from busy Ps
- No manual distribution needed

✅ **Efficient I/O handling**
- Netpoller for network operations
- Blocking syscalls park G, not M
- OS thread continues running other goroutines

✅ **Async preemption (Go 1.14+)**
- CPU-bound goroutines get preempted
- No more starvation
- ~10ms time slices

✅ **GOMAXPROCS tuning**
- Default: Number of CPU cores
- Usually optimal
- Adjust for specific workloads

---

## 2.4 Memory Management & Garbage Collection

Understanding Go's memory management is crucial for writing high-performance applications.

### Stack vs Heap Allocation

Go automatically decides where to allocate variables:

```go
func stackAllocation() int {
    x := 42  // Allocated on stack (fast)
    return x
}

func heapAllocation() *int {
    x := 42
    return &x  // x escapes, allocated on heap
}
```

**Stack allocation:**
- Fast (just increment stack pointer)
- Automatically cleaned up when function returns
- No GC overhead

**Heap allocation:**
- Slower (requires heap allocator)
- Requires garbage collection
- Lives beyond function scope

### Escape Analysis

Go compiler determines whether variables escape to heap:

```go
// Check with: go build -gcflags='-m'

func noEscape() {
    x := 42  // Stays on stack
    fmt.Println(x)
}
// Output: x does not escape

func escapes() *int {
    x := 42
    return &x  // x escapes to heap
}
// Output: moved to heap: x

func sliceEscape() {
    s := []int{1, 2, 3}  // Escapes (slice header on stack, array on heap)
    process(s)
}

func interfaceEscape() {
    x := 42
    var i interface{} = x  // x escapes (interface causes boxing)
    process(i)
}
```

**Common escape scenarios:**

```go
// ✅ Stays on stack
func local() {
    var arr [100]int
    processArray(&arr)  // Pointer doesn't escape
}

// ❌ Escapes to heap
func returnPointer() *int {
    x := 42
    return &x  // Pointer escapes
}

// ❌ Escapes to heap
func closureCapture() func() int {
    x := 42
    return func() int {
        return x  // x escapes (captured by closure)
    }
}

// ❌ Escapes to heap
func sendToChannel(ch chan int) {
    x := 42
    ch <- x  // x escapes (sent to channel)
}

// ❌ Escapes to heap
func assignToInterface() {
    x := 42
    var i interface{} = x  // x escapes (boxed in interface)
}
```

**Escape analysis output:**

```bash
$ go build -gcflags='-m -m' main.go

# Detailed escape analysis
./main.go:5:2: x escapes to heap:
./main.go:5:2:   flow: ~r0 = &x:
./main.go:5:2:     from &x (address-of) at ./main.go:6:9
./main.go:5:2:     from return &x (return) at ./main.go:6:2
./main.go:5:2: moved to heap: x
```

### Memory Allocator

Go uses a sophisticated memory allocator based on TCMalloc:

```
Size Classes:
┌─────────────┐
│ Tiny        │ < 16 bytes
│ Small       │ 16 bytes - 32 KB
│ Large       │ > 32 KB
└─────────────┘

Per-P Cache (mcache):
┌─────────────────────────┐
│ Tiny allocations        │
│ Small size class spans  │
│ (no locking needed!)    │
└─────────────────────────┘
         │
         ▼
Central Lists (mcentral):
┌─────────────────────────┐
│ Spans for each          │
│ size class              │
│ (requires locking)      │
└─────────────────────────┘
         │
         ▼
Heap (mheap):
┌─────────────────────────┐
│ Large objects           │
│ Span allocation         │
│ Free page management    │
└─────────────────────────┘
```

**Allocation path:**

```go
// Tiny allocation (< 16 bytes)
var x int8  // Check mcache tiny allocator → bump pointer

// Small allocation (16B - 32KB)
var arr [100]int  // Check mcache for size class → use span

// Large allocation (> 32KB)
var large [10000]int  // Allocate directly from mheap
```

### Garbage Collector

Go uses a **concurrent, tri-color mark-and-sweep** garbage collector.

#### **Tri-Color Marking**

```
┌─────────┐
│ White   │ Not yet visited
└─────────┘
     │
     ▼
┌─────────┐
│ Grey    │ Visited but references not scanned
└─────────┘
     │
     ▼
┌─────────┐
│ Black   │ Visited and references scanned
└─────────┘
```

**GC Phases:**

```
1. Mark Setup (STW - ~microseconds)
   └─ Prepare for marking

2. Marking (Concurrent)
   └─ Traverse object graph
      ├─ Mark reachable objects
      └─ Write barrier for mutations

3. Mark Termination (STW - ~microseconds)
   └─ Complete marking

4. Sweep (Concurrent)
   └─ Reclaim unmarked memory
```

**Example GC cycle:**

```go
func allocateMemory() {
    // Allocate objects
    objects := make([]*Object, 1000)
    for i := range objects {
        objects[i] = &Object{Data: make([]byte, 1024)}
    }
    
    // Some objects become unreachable
    for i := 0; i < 500; i++ {
        objects[i] = nil
    }
    
    // GC runs (automatically)
    // - Marks 500 reachable objects (black)
    // - 500 unreachable remain white
    // - Sweeps white objects
    // - Memory reclaimed
}
```

#### **Write Barrier**

Ensures correctness during concurrent marking:

```go
// During GC marking:
obj1.ptr = obj2  // Write barrier records this mutation

// Without write barrier:
// 1. GC marks obj1 (black)
// 2. obj1.ptr = obj2 (obj2 is white)
// 3. GC never scans obj1 again
// 4. obj2 incorrectly collected!

// With write barrier:
// 1. GC marks obj1 (black)
// 2. obj1.ptr = obj2 → write barrier marks obj1 grey
// 3. GC rescans obj1, discovers obj2
// 4. obj2 correctly kept
```

#### **GC Pacing**

Go automatically triggers GC based on memory usage:

```go
// GC triggered when:
// heap size >= target heap size

// Target = live heap size * (1 + GOGC/100)
// Default GOGC = 100
// So target = live heap * 2

// Example:
// Live heap: 100 MB
// Target: 200 MB (with GOGC=100)
// GC triggers when heap reaches 200 MB
```

**GOGC tuning:**

```bash
# Default (100): GC when heap doubles
GOGC=100 go run main.go

# More aggressive (50): GC when heap grows 50%
GOGC=50 go run main.go

# Less aggressive (200): GC when heap triples
GOGC=200 go run main.go

# Disable GC (not recommended!)
GOGC=off go run main.go
```

#### **Soft Memory Limit (Go 1.19+)**

```go
// Set soft memory limit
debug.SetMemoryLimit(1 << 30)  // 1 GB

// Or via environment variable
// GOMEMLIMIT=1GiB go run main.go
```

**How it works:**
- GC tries to keep memory below limit
- More aggressive collection when approaching limit
- Prevents OOM in containers with memory limits

```go
import "runtime/debug"

func main() {
    // Set 500MB limit
    debug.SetMemoryLimit(500 * 1024 * 1024)
    
    // Allocate memory
    // GC becomes more aggressive as we approach limit
    for i := 0; i < 1000; i++ {
        data := make([]byte, 1024*1024)  // 1MB
        _ = data
    }
}
```

Continuing with more of Part 2...


### Reducing GC Pressure

**Techniques to reduce allocations:**

```go
// ❌ BAD: Allocates every call
func processData(data string) string {
    return strings.ToUpper(data)  // New string allocated
}

// ✅ BETTER: Reuse builders
var builderPool = sync.Pool{
    New: func() interface{} {
        return new(strings.Builder)
    },
}

func processData(data string) string {
    b := builderPool.Get().(*strings.Builder)
    defer func() {
        b.Reset()
        builderPool.Put(b)
    }()
    
    b.WriteString(data)
    // process...
    return b.String()
}

// ❌ BAD: Repeated allocations in loop
func sumSlices(data [][]int) int {
    sum := 0
    for _, slice := range data {
        temp := make([]int, len(slice))  // Allocates every iteration!
        copy(temp, slice)
        sum += process(temp)
    }
    return sum
}

// ✅ GOOD: Reuse slice
func sumSlices(data [][]int) int {
    sum := 0
    temp := make([]int, 0)  // Allocate once
    for _, slice := range data {
        temp = temp[:0]  // Reuse capacity
        temp = append(temp, slice...)
        sum += process(temp)
    }
    return sum
}
```

**Pre-allocate when size known:**

```go
// ❌ BAD: Repeated reallocations
func buildList(n int) []int {
    var result []int  // nil slice
    for i := 0; i < n; i++ {
        result = append(result, i)  // Reallocates multiple times
    }
    return result
}

// ✅ GOOD: Pre-allocate
func buildList(n int) []int {
    result := make([]int, 0, n)  // Pre-allocate capacity
    for i := 0; i < n; i++ {
        result = append(result, i)  // No reallocations
    }
    return result
}

// ✅ EVEN BETTER: Direct indexing
func buildList(n int) []int {
    result := make([]int, n)  // Pre-allocate with length
    for i := 0; i < n; i++ {
        result[i] = i  // Direct assignment, no append
    }
    return result
}
```

**Avoid boxing in interfaces:**

```go
// ❌ BAD: Boxing on every call
func process(values []int) {
    for _, v := range values {
        var i interface{} = v  // Boxes int on heap
        doSomething(i)
    }
}

// ✅ GOOD: Keep as concrete type
func process(values []int) {
    for _, v := range values {
        doSomething(v)  // No boxing
    }
}

func doSomething(v int) {  // Takes int, not interface{}
    // ...
}
```

### Memory Profiling

**Heap profiling:**

```go
import (
    "os"
    "runtime/pprof"
)

func main() {
    f, _ := os.Create("mem.prof")
    defer f.Close()
    
    // Your code here
    allocateMemory()
    
    pprof.WriteHeapProfile(f)
}

// Analyze:
// go tool pprof mem.prof
// (pprof) top
// (pprof) list functionName
```

**Continuous profiling in production:**

```go
import _ "net/http/pprof"

func main() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    // Your application code
}

// Access profiles:
// http://localhost:6060/debug/pprof/heap
// http://localhost:6060/debug/pprof/allocs
// http://localhost:6060/debug/pprof/goroutine
```

**Allocation profiling:**

```go
// go test -memprofile=mem.prof -bench=.

func BenchmarkAllocations(b *testing.B) {
    b.ReportAllocs()  // Report allocation stats
    
    for i := 0; i < b.N; i++ {
        data := make([]byte, 1024)
        _ = data
    }
}

// Output shows:
// BenchmarkAllocations-8   1000000   1234 ns/op   1024 B/op   1 allocs/op
```

### GC Tuning

**Monitoring GC:**

```go
import "runtime"

func printMemStats() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Printf("Alloc = %v MB", m.Alloc/1024/1024)
    fmt.Printf("\tTotalAlloc = %v MB", m.TotalAlloc/1024/1024)
    fmt.Printf("\tSys = %v MB", m.Sys/1024/1024)
    fmt.Printf("\tNumGC = %v\n", m.NumGC)
}
```

**GC events logging:**

```bash
# Enable GC trace
GODEBUG=gctrace=1 go run main.go

# Output:
# gc 1 @0.001s 0%: 0.018+0.25+0.003 ms clock, 0.14+0.13/0.23/0.44+0.025 ms cpu, 4->4->3 MB, 5 MB goal, 8 P
#
# Breakdown:
# gc 1          - GC number
# @0.001s       - Time since program start
# 0%            - Percent of time in GC
# 0.018+...     - GC phase times
# 4->4->3 MB    - heap before -> heap after -> live heap
# 5 MB goal     - target heap size
# 8 P           - number of processors
```

**GOGC tuning guidelines:**

```go
// Default GOGC=100 (double before GC)
// Good for: Most applications

// GOGC=50 (1.5x before GC)
// Good for: Memory-constrained environments
// Trade-off: More frequent GC, lower memory

// GOGC=200 (3x before GC)
// Good for: Latency-sensitive, high throughput
// Trade-off: Higher memory usage, less frequent GC

// GOGC=off
// Good for: Batch processing, short-lived programs
// Trade-off: No automatic GC, must call runtime.GC() manually
```

### Memory Layout

**Struct memory layout:**

```go
type Example struct {
    a bool    // 1 byte
    // 7 bytes padding
    b int64   // 8 bytes
    c int32   // 4 bytes
    // 4 bytes padding
}
// Total: 24 bytes (with padding)

// Better layout:
type Optimized struct {
    b int64   // 8 bytes
    c int32   // 4 bytes
    a bool    // 1 byte
    // 3 bytes padding
}
// Total: 16 bytes

// Check size:
fmt.Println(unsafe.Sizeof(Example{}))    // 24
fmt.Println(unsafe.Sizeof(Optimized{}))  // 16
```

**Slice internals:**

```go
type SliceHeader struct {
    Data uintptr  // 8 bytes (pointer to array)
    Len  int      // 8 bytes
    Cap  int      // 8 bytes
}
// Total: 24 bytes for slice header
// Plus array data on heap

s := make([]int, 10, 20)
// Header: 24 bytes on stack
// Array: 20 * 8 = 160 bytes on heap
```

**Map internals:**

```go
// Map is a pointer to runtime.hmap
// hmap contains:
// - buckets array
// - hash seed
// - count
// - etc.

m := make(map[string]int)
// Pointer: 8 bytes on stack
// hmap structure: on heap
// Initial buckets: on heap
```

### Frequently Asked Questions (Memory & GC)

**Q1: How can I reduce garbage collection pauses?**

**A:**

1. **Reduce allocations:**
   - Reuse objects (sync.Pool)
   - Pre-allocate slices/maps
   - Avoid allocations in hot paths

2. **Tune GOGC:**
   ```bash
   # Less frequent GC (more memory, less pause frequency)
   GOGC=200 go run main.go
   ```

3. **Use memory limit (Go 1.19+):**
   ```bash
   GOMEMLIMIT=1GiB go run main.go
   ```

4. **Pointer-free data structures:**
   ```go
   // Has pointers (GC must scan)
   type Node struct {
       next *Node
       data int
   }
   
   // No pointers (GC can skip)
   type Data struct {
       values [1000]int  // Array of values
   }
   ```

---

**Q2: What's the difference between stack and heap allocation?**

**A:**

**Stack:**
- Fast allocation (bump pointer)
- Automatic deallocation (on return)
- Limited size (~8MB per goroutine)
- No GC overhead

**Heap:**
- Slower allocation
- Requires GC to reclaim
- Unlimited size (within system memory)
- Can escape function scope

**Escape analysis decides:**
```go
func stack() {
    x := 42  // Stack (doesn't escape)
    process(x)
}

func heap() *int {
    x := 42
    return &x  // Heap (escapes)
}
```

---

**Q3: How do I find memory leaks in Go?**

**A:**

1. **Monitor goroutines:**
   ```go
   fmt.Println(runtime.NumGoroutine())
   ```

2. **Heap profiling:**
   ```bash
   go tool pprof http://localhost:6060/debug/pprof/heap
   ```

3. **Compare heap snapshots:**
   ```bash
   curl http://localhost:6060/debug/pprof/heap > heap1.prof
   # ... wait ...
   curl http://localhost:6060/debug/pprof/heap > heap2.prof
   go tool pprof -base heap1.prof heap2.prof
   ```

4. **Look for:**
   - Growing number of goroutines
   - Growing heap size
   - Objects not being freed

**Common leak sources:**
- Goroutines that never exit
- Global variables holding references
- Unclosed channels/connections
- Circular references (rare in Go)

---

### Interview Questions (Memory & GC)

**Question 1: Explain escape analysis with an example.**

**Difficulty:** Mid-Level

**Answer:**

Escape analysis determines if a variable can be allocated on the stack or must be allocated on the heap.

```go
// Doesn't escape - stays on stack
func stackAlloc() int {
    x := 42
    return x  // Value copied
}

// Escapes to heap - pointer leaves function
func heapAlloc() *int {
    x := 42
    return &x  // Pointer escapes
}

// Check with compiler:
// go build -gcflags='-m' main.go
// Output: moved to heap: x
```

**Variables escape when:**
- Returned as pointer
- Sent to channel
- Assigned to interface
- Captured by closure
- Too large for stack

**Why it matters:**
- Stack allocation is faster
- Heap allocation requires GC
- Understanding helps optimize performance

---

**Question 2: How would you optimize this code for memory?**

```go
func processLargeDataset(data []string) []string {
    var results []string
    for _, item := range data {
        processed := strings.ToUpper(item)
        results = append(results, processed)
    }
    return results
}
```

**Difficulty:** Mid-Level

**Answer:**

**Optimizations:**

```go
// 1. Pre-allocate result slice
func processLargeDataset(data []string) []string {
    results := make([]string, 0, len(data))  // Pre-allocate
    for _, item := range data {
        processed := strings.ToUpper(item)
        results = append(results, processed)
    }
    return results
}

// 2. In-place modification (if allowed)
func processLargeDataset(data []string) []string {
    for i, item := range data {
        data[i] = strings.ToUpper(item)
    }
    return data
}

// 3. Use strings.Builder for concatenation
func processLargeDataset(data []string) []string {
    results := make([]string, len(data))
    for i, item := range data {
        results[i] = strings.ToUpper(item)
    }
    return results
}

// Benchmark to compare!
```

**What Interviewers Look For:**
- Understanding of allocation costs
- Knowledge of pre-allocation benefits
- Awareness of in-place vs new allocation
- Ability to identify optimization opportunities

---

### Key Takeaways (Memory & GC)

✅ **Escape analysis determines allocation location**
- Stack: Fast, automatic cleanup
- Heap: Slower, requires GC
- Use `-gcflags='-m'` to check

✅ **Reduce allocations for better performance**
- Pre-allocate slices/maps when size known
- Reuse objects with sync.Pool
- Avoid allocations in hot paths
- Keep data on stack when possible

✅ **Concurrent mark-and-sweep GC**
- Tri-color marking
- Concurrent with application
- Write barrier ensures correctness
- Pauses typically sub-millisecond

✅ **GC tuning options**
- GOGC: controls GC frequency (default 100)
- GOMEMLIMIT: soft memory limit (Go 1.19+)
- Profile and measure before optimizing

✅ **Memory profiling essential**
- pprof for heap profiling
- Allocation profiling in benchmarks
- Monitor GC with GODEBUG=gctrace=1
- Compare snapshots to find leaks

---

## Part 2 Summary

Congratulations! You've completed Part 2 of the Complete Go Mastery series. You now have deep knowledge of Go's concurrency model and runtime internals.

### What You've Learned

**Concurrency Fundamentals:**
- Goroutines: lightweight, 2KB initial stack, millions possible
- Channels: communication, buffered vs unbuffered, patterns
- Synchronization: mutexes, WaitGroups, atomics, Context
- Common patterns: pipeline, fan-out/fan-in, worker pool

**Go Scheduler:**
- GMP model: Goroutines, Machines (OS threads), Processors
- Work-stealing algorithm for load balancing
- Async preemption for fairness
- Network poller integration
- GOMAXPROCS tuning

**Memory Management:**
- Stack vs heap allocation
- Escape analysis
- Memory allocator (mcache, mcentral, mheap)
- Concurrent garbage collector
- GC tuning (GOGC, GOMEMLIMIT)
- Profiling and optimization

### Critical Concepts

✅ **"Share memory by communicating"** - Use channels over shared state
✅ **Work-stealing scheduler** - Automatic load balancing
✅ **Escape analysis** - Compiler decides stack vs heap
✅ **Concurrent GC** - Low pause times, automatic tuning
✅ **Profiling is essential** - Measure before optimizing

### What's Next

In **Part 3: Advanced Go Patterns & Techniques**, we'll explore:
- Reflection and metaprogramming
- Generics (Go 1.18+)
- Code generation
- Unsafe operations
- Assembly integration
- Advanced interface patterns
- Error handling patterns

### Practice Exercises

Before moving to Part 3, reinforce your learning:

1. **Build a concurrent web crawler** using goroutines and channels
2. **Implement a rate limiter** with multiple strategies
3. **Create a worker pool** with graceful shutdown
4. **Profile a program** and optimize memory allocations
5. **Write benchmarks** comparing different synchronization approaches

### Additional Resources

- [Go Concurrency Patterns](https://go.dev/talks/2012/concurrency.slide) - Rob Pike
- [Advanced Go Concurrency Patterns](https://go.dev/talks/2013/advconc.slide) - Sameer Ajmani  
- [Go Scheduler Design Doc](https://golang.org/s/go11sched) - Dmitry Vyukov
- [GC Pacer Redesign](https://go.googlesource.com/proposal/+/master/design/44167-gc-pacer-redesign.md)

---

**Ready to explore advanced Go techniques?** Continue to Part 3 where we'll dive into reflection, generics, code generation, and more advanced patterns!


## Bonus Section: Advanced Concurrency Patterns

### Semaphore Pattern

Limit concurrent access to a resource:

```go
type Semaphore struct {
    slots chan struct{}
}

func NewSemaphore(max int) *Semaphore {
    return &Semaphore{
        slots: make(chan struct{}, max),
    }
}

func (s *Semaphore) Acquire() {
    s.slots <- struct{}{}
}

func (s *Semaphore) Release() {
    <-s.slots
}

// Usage: Limit concurrent database connections
func processWithLimit(items []Item) {
    sem := NewSemaphore(10)  // Max 10 concurrent
    
    var wg sync.WaitGroup
    for _, item := range items {
        wg.Add(1)
        go func(i Item) {
            defer wg.Done()
            
            sem.Acquire()
            defer sem.Release()
            
            processItem(i)
        }(item)
    }
    wg.Wait()
}

// Or use golang.org/x/sync/semaphore
import "golang.org/x/sync/semaphore"

func processWithSemaphore(ctx context.Context, items []Item) error {
    sem := semaphore.NewWeighted(10)
    
    for _, item := range items {
        if err := sem.Acquire(ctx, 1); err != nil {
            return err
        }
        
        go func(i Item) {
            defer sem.Release(1)
            processItem(i)
        }(item)
    }
    
    // Wait for all to complete
    if err := sem.Acquire(ctx, 10); err != nil {
        return err
    }
    
    return nil
}
```

### Circuit Breaker Pattern

Prevent cascading failures:

```go
type State int

const (
    StateClosed State = iota  // Normal operation
    StateOpen                  // Failing, reject requests
    StateHalfOpen             // Testing if recovered
)

type CircuitBreaker struct {
    mu              sync.RWMutex
    state           State
    failures        int
    lastFailureTime time.Time
    
    maxFailures     int
    timeout         time.Duration
}

func NewCircuitBreaker(maxFailures int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        state:       StateClosed,
        maxFailures: maxFailures,
        timeout:     timeout,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    
    // Check if we should try again after timeout
    if cb.state == StateOpen {
        if time.Since(cb.lastFailureTime) > cb.timeout {
            cb.state = StateHalfOpen
            cb.failures = 0
        } else {
            cb.mu.Unlock()
            return errors.New("circuit breaker is open")
        }
    }
    
    cb.mu.Unlock()
    
    // Execute function
    err := fn()
    
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if err != nil {
        cb.failures++
        cb.lastFailureTime = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = StateOpen
        }
        return err
    }
    
    // Success - reset
    if cb.state == StateHalfOpen {
        cb.state = StateClosed
    }
    cb.failures = 0
    
    return nil
}

// Usage
func callExternalService() error {
    cb := NewCircuitBreaker(5, 10*time.Second)
    
    return cb.Call(func() error {
        resp, err := http.Get("https://api.example.com")
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        
        if resp.StatusCode != 200 {
            return errors.New("service error")
        }
        
        return nil
    })
}
```

### Bounded Parallelism

Process items with controlled concurrency:

```go
func processConcurrently(items []Item, maxWorkers int) []Result {
    jobs := make(chan Item, len(items))
    results := make(chan Result, len(items))
    
    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < maxWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                result := process(item)
                results <- result
            }
        }()
    }
    
    // Send jobs
    for _, item := range items {
        jobs <- item
    }
    close(jobs)
    
    // Wait and close results
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect results
    var output []Result
    for result := range results {
        output = append(output, result)
    }
    
    return output
}
```

### Retry with Backoff

Resilient retry logic:

```go
type BackoffStrategy func(attempt int) time.Duration

func ExponentialBackoff(base time.Duration) BackoffStrategy {
    return func(attempt int) time.Duration {
        return base * time.Duration(math.Pow(2, float64(attempt)))
    }
}

func RetryWithBackoff(
    ctx context.Context,
    maxAttempts int,
    backoff BackoffStrategy,
    fn func() error,
) error {
    var err error
    
    for attempt := 0; attempt < maxAttempts; attempt++ {
        err = fn()
        if err == nil {
            return nil
        }
        
        // Check if context cancelled
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        
        // Last attempt, don't wait
        if attempt == maxAttempts-1 {
            break
        }
        
        // Wait with backoff
        wait := backoff(attempt)
        select {
        case <-time.After(wait):
        case <-ctx.Done():
            return ctx.Err()
        }
    }
    
    return fmt.Errorf("max retries exceeded: %w", err)
}

// Usage
func callAPIWithRetry(ctx context.Context) error {
    backoff := ExponentialBackoff(time.Second)
    
    return RetryWithBackoff(ctx, 5, backoff, func() error {
        resp, err := http.Get("https://api.example.com")
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        
        if resp.StatusCode >= 500 {
            return errors.New("server error")
        }
        
        return nil
    })
}
```

### Batch Processing

Accumulate items and process in batches:

```go
type Batcher struct {
    maxSize     int
    maxWait     time.Duration
    processBatch func([]Item) error
    
    items       []Item
    mu          sync.Mutex
    timer       *time.Timer
}

func NewBatcher(maxSize int, maxWait time.Duration, process func([]Item) error) *Batcher {
    b := &Batcher{
        maxSize:     maxSize,
        maxWait:     maxWait,
        processBatch: process,
        items:       make([]Item, 0, maxSize),
    }
    
    b.timer = time.AfterFunc(maxWait, b.flush)
    b.timer.Stop()
    
    return b
}

func (b *Batcher) Add(item Item) error {
    b.mu.Lock()
    defer b.mu.Unlock()
    
    // Start timer on first item
    if len(b.items) == 0 {
        b.timer.Reset(b.maxWait)
    }
    
    b.items = append(b.items, item)
    
    // Process if batch full
    if len(b.items) >= b.maxSize {
        return b.flushLocked()
    }
    
    return nil
}

func (b *Batcher) flush() {
    b.mu.Lock()
    defer b.mu.Unlock()
    b.flushLocked()
}

func (b *Batcher) flushLocked() error {
    if len(b.items) == 0 {
        return nil
    }
    
    b.timer.Stop()
    
    items := b.items
    b.items = make([]Item, 0, b.maxSize)
    
    return b.processBatch(items)
}

// Usage: Batch database inserts
func batchInserts() {
    batcher := NewBatcher(100, time.Second, func(items []Item) error {
        // Insert batch into database
        return db.InsertBatch(items)
    })
    
    // Add items as they arrive
    for item := range itemStream {
        batcher.Add(item)
    }
}
```

### Cancellation Propagation

Properly propagate cancellation:

```go
func processWithCancellation(ctx context.Context, items []Item) error {
    // Create derived context for this operation
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    results := make(chan Result, len(items))
    errors := make(chan error, len(items))
    
    var wg sync.WaitGroup
    for _, item := range items {
        wg.Add(1)
        go func(i Item) {
            defer wg.Done()
            
            result, err := processItem(ctx, i)
            if err != nil {
                errors <- err
                cancel()  // Cancel other workers
                return
            }
            
            select {
            case results <- result:
            case <-ctx.Done():
                return
            }
        }(item)
    }
    
    // Wait for completion
    go func() {
        wg.Wait()
        close(results)
        close(errors)
    }()
    
    // Check for errors
    select {
    case err := <-errors:
        return err
    case <-ctx.Done():
        return ctx.Err()
    default:
        return nil
    }
}

func processItem(ctx context.Context, item Item) (Result, error) {
    // Check cancellation before expensive work
    select {
    case <-ctx.Done():
        return Result{}, ctx.Err()
    default:
    }
    
    // Perform work
    result := doWork(item)
    
    // Check cancellation before returning
    select {
    case <-ctx.Done():
        return Result{}, ctx.Err()
    default:
        return result, nil
    }
}
```

### Error Group Pattern

```go
import "golang.org/x/sync/errgroup"

func processConcurrently(ctx context.Context, items []Item) error {
    g, ctx := errgroup.WithContext(ctx)
    
    for _, item := range items {
        item := item  // Capture for goroutine
        g.Go(func() error {
            return processItem(ctx, item)
        })
    }
    
    // Wait for all goroutines
    // Returns first error encountered
    return g.Wait()
}

// With limit
func processConcurrentlyLimited(ctx context.Context, items []Item) error {
    g, ctx := errgroup.WithContext(ctx)
    g.SetLimit(10)  // Max 10 concurrent
    
    for _, item := range items {
        item := item
        g.Go(func() error {
            return processItem(ctx, item)
        })
    }
    
    return g.Wait()
}
```

### Complex Example: Concurrent Web Scraper

Putting it all together:

```go
type Scraper struct {
    client      *http.Client
    rateLimiter *rate.Limiter
    visited     sync.Map
    results     chan Result
    errors      chan error
}

func NewScraper(rps int) *Scraper {
    return &Scraper{
        client:      &http.Client{Timeout: 10 * time.Second},
        rateLimiter: rate.NewLimiter(rate.Limit(rps), 1),
        results:     make(chan Result, 100),
        errors:      make(chan error, 100),
    }
}

func (s *Scraper) Scrape(ctx context.Context, urls []string) ([]Result, error) {
    g, ctx := errgroup.WithContext(ctx)
    g.SetLimit(20)  // Max 20 concurrent requests
    
    for _, url := range urls {
        url := url
        g.Go(func() error {
            return s.scrapeURL(ctx, url)
        })
    }
    
    // Collect results
    var results []Result
    done := make(chan struct{})
    
    go func() {
        defer close(done)
        for result := range s.results {
            results = append(results, result)
        }
    }()
    
    // Wait for all scraping to complete
    err := g.Wait()
    close(s.results)
    close(s.errors)
    
    <-done
    
    return results, err
}

func (s *Scraper) scrapeURL(ctx context.Context, url string) error {
    // Check if already visited
    if _, loaded := s.visited.LoadOrStore(url, true); loaded {
        return nil
    }
    
    // Rate limit
    if err := s.rateLimiter.Wait(ctx); err != nil {
        return err
    }
    
    // Fetch with retry
    backoff := ExponentialBackoff(time.Second)
    var body []byte
    
    err := RetryWithBackoff(ctx, 3, backoff, func() error {
        req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
        if err != nil {
            return err
        }
        
        resp, err := s.client.Do(req)
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        
        if resp.StatusCode != 200 {
            return fmt.Errorf("status %d", resp.StatusCode)
        }
        
        body, err = io.ReadAll(resp.Body)
        return err
    })
    
    if err != nil {
        select {
        case s.errors <- err:
        case <-ctx.Done():
        }
        return err
    }
    
    // Parse and send result
    result := parseHTML(body)
    
    select {
    case s.results <- result:
    case <-ctx.Done():
        return ctx.Err()
    }
    
    return nil
}

func parseHTML(body []byte) Result {
    // Parse HTML and extract data
    return Result{Data: string(body)}
}

// Usage
func main() {
    ctx, cancel := context.WithTimeout(context.Background(), time.Minute)
    defer cancel()
    
    scraper := NewScraper(10)  // 10 requests per second
    
    urls := []string{
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
    }
    
    results, err := scraper.Scrape(ctx, urls)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Scraped %d URLs\n", len(results))
}
```

### Real-World Patterns Summary

**Pattern Selection Guide:**

| Pattern | Use Case | When to Use |
|---------|----------|-------------|
| **Worker Pool** | Fixed concurrent workers | Known resource limit |
| **Pipeline** | Chain transformations | Multi-stage processing |
| **Fan-Out/Fan-In** | Distribute and collect | Parallel processing + aggregation |
| **Semaphore** | Limit concurrency | Resource protection |
| **Circuit Breaker** | Prevent cascading failures | External service calls |
| **Rate Limiting** | Control throughput | API rate limits |
| **Batch Processing** | Accumulate and process | Database bulk operations |
| **Context Cancellation** | Lifecycle management | Long-running operations |
| **Retry with Backoff** | Handle transient failures | Network calls |
| **Error Group** | Coordinate goroutines | Concurrent tasks with errors |

### Performance Comparison

Benchmarks for common patterns:

```go
func BenchmarkChannelVsMutex(b *testing.B) {
    b.Run("Channel", func(b *testing.B) {
        ch := make(chan int, 1)
        b.ResetTimer()
        for i := 0; i < b.N; i++ {
            ch <- i
            <-ch
        }
    })
    
    b.Run("Mutex", func(b *testing.B) {
        var mu sync.Mutex
        var value int
        b.ResetTimer()
        for i := 0; i < b.N; i++ {
            mu.Lock()
            value = i
            _ = value
            mu.Unlock()
        }
    })
}

// Results (typical):
// Channel: ~60 ns/op
// Mutex:   ~20 ns/op
// 
// Conclusion: Mutex faster for simple synchronization
// Use channels for communication, mutexes for state protection
```

### Final Best Practices

1. **Start Simple**: Begin with channels, add complexity only when needed
2. **Measure First**: Profile before optimizing
3. **Limit Concurrency**: Don't create unlimited goroutines
4. **Cancel Properly**: Always use context for cancellation
5. **Handle Errors**: Propagate errors from goroutines
6. **Test with -race**: Always run tests with race detector
7. **Document Patterns**: Explain concurrency patterns in comments
8. **Avoid Premature Optimization**: Start sequential, parallelize hot paths

---

This comprehensive guide to Go concurrency and runtime internals provides the foundation for building high-performance, concurrent systems. The patterns and techniques covered here are used in production systems handling millions of requests.


## Comprehensive Concurrency Debugging Guide

### Race Condition Detection

**Using the race detector:**

```go
// Run tests with race detector
go test -race ./...

// Run program with race detector
go run -race main.go

// Build with race detection
go build -race
```

**Common race patterns:**

```go
// ❌ Race: Shared map without synchronization
var cache = make(map[string]string)

func get(key string) string {
    return cache[key]  // Race!
}

func set(key, value string) {
    cache[key] = value  // Race!
}

// ✅ Fixed: Use sync.Map or mutex
var cache sync.Map

func get(key string) string {
    if value, ok := cache.Load(key); ok {
        return value.(string)
    }
    return ""
}

func set(key, value string) {
    cache.Store(key, value)
}

// ❌ Race: Closure capturing loop variable
for i := 0; i < 10; i++ {
    go func() {
        fmt.Println(i)  // Race!
    }()
}

// ✅ Fixed: Pass as parameter
for i := 0; i < 10; i++ {
    go func(val int) {
        fmt.Println(val)
    }(i)
}

// ❌ Race: Concurrent slice append
var results []int

func worker(id int) {
    results = append(results, id)  // Race!
}

// ✅ Fixed: Use channel or mutex
resultsCh := make(chan int, 10)

func worker(id int) {
    resultsCh <- id
}
```

### Deadlock Detection

**Common deadlock scenarios:**

```go
// 1. Circular wait
var mu1, mu2 sync.Mutex

// Goroutine 1
mu1.Lock()
mu2.Lock()  // Waits for mu2
mu2.Unlock()
mu1.Unlock()

// Goroutine 2
mu2.Lock()
mu1.Lock()  // Waits for mu1 - DEADLOCK!
mu1.Unlock()
mu2.Unlock()

// Fix: Always acquire locks in same order
mu1.Lock()
mu2.Lock()
// work
mu2.Unlock()
mu1.Unlock()

// 2. Channel deadlock
ch := make(chan int)
ch <- 42  // Blocks forever (no receiver)

// Fix: Send in goroutine or use buffered channel
go func() {
    ch <- 42
}()
value := <-ch

// 3. Waiting for self
var wg sync.WaitGroup
wg.Add(1)
wg.Wait()  // Deadlock: waiting for itself

// Fix: Don't wait in same goroutine
wg.Add(1)
go func() {
    defer wg.Done()
    // work
}()
wg.Wait()
```

### Goroutine Leak Detection

**Finding leaks:**

```go
func TestNoLeaks(t *testing.T) {
    before := runtime.NumGoroutine()
    
    // Run code that should cleanup
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    
    worker(ctx)
    
    // Give time for cleanup
    time.Sleep(100 * time.Millisecond)
    
    after := runtime.NumGoroutine()
    if after > before {
        t.Errorf("Leaked %d goroutines", after-before)
        
        // Get stack traces
        buf := make([]byte, 1<<16)
        runtime.Stack(buf, true)
        t.Logf("Stack trace:\n%s", buf)
    }
}

// Using goleak
import "go.uber.org/goleak"

func TestMain(m *testing.M) {
    goleak.VerifyTestMain(m)
}

func TestNoLeaksWithGoleak(t *testing.T) {
    defer goleak.VerifyNone(t)
    // Test code
}
```

**Common leak causes:**

```go
// ❌ Leak: Goroutine never exits
func leak() {
    go func() {
        for {
            work()  // Never exits!
        }
    }()
}

// ✅ Fix: Add exit condition
func noLeak(ctx context.Context) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            default:
                work()
            }
        }
    }()
}

// ❌ Leak: Channel never receives
func leak() {
    ch := make(chan int)
    go func() {
        ch <- 42  // Blocks forever
    }()
}

// ✅ Fix: Ensure receiver exists
func noLeak() {
    ch := make(chan int)
    go func() {
        ch <- 42
    }()
    <-ch  // Receive
}
```

### Profiling Concurrent Programs

**CPU profiling with pprof:**

```go
import (
    "os"
    "runtime/pprof"
)

func main() {
    f, _ := os.Create("cpu.prof")
    defer f.Close()
    
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // Your concurrent code
    runWorkload()
}

// Analyze
// go tool pprof cpu.prof
// (pprof) top
// (pprof) web
// (pprof) list functionName
```

**Goroutine profiling:**

```go
import _ "net/http/pprof"

func main() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    // Your code
}

// View goroutines:
// http://localhost:6060/debug/pprof/goroutine
// 
// Download profile:
// curl http://localhost:6060/debug/pprof/goroutine > goroutine.prof
// go tool pprof goroutine.prof
```

**Block profiling:**

```go
runtime.SetBlockProfileRate(1)  // Enable block profiling

// Access at:
// http://localhost:6060/debug/pprof/block
```

**Mutex profiling:**

```go
runtime.SetMutexProfileFraction(1)  // Enable mutex profiling

// Access at:
// http://localhost:6060/debug/pprof/mutex
```

### Testing Concurrent Code

**Table-driven tests with concurrency:**

```go
func TestConcurrentCounter(t *testing.T) {
    tests := []struct {
        name       string
        goroutines int
        operations int
        want       int
    }{
        {"10 goroutines, 100 ops", 10, 100, 1000},
        {"100 goroutines, 10 ops", 100, 10, 1000},
        {"1 goroutine, 1000 ops", 1, 1000, 1000},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            counter := NewCounter()
            
            var wg sync.WaitGroup
            for i := 0; i < tt.goroutines; i++ {
                wg.Add(1)
                go func() {
                    defer wg.Done()
                    for j := 0; j < tt.operations; j++ {
                        counter.Increment()
                    }
                }()
            }
            
            wg.Wait()
            
            if got := counter.Value(); got != tt.want {
                t.Errorf("got %d, want %d", got, tt.want)
            }
        })
    }
}
```

**Fuzzing concurrent code:**

```go
func FuzzConcurrentMap(f *testing.F) {
    f.Add([]byte("key1"), []byte("value1"))
    f.Add([]byte("key2"), []byte("value2"))
    
    f.Fuzz(func(t *testing.T, key, value []byte) {
        m := NewSafeMap()
        
        var wg sync.WaitGroup
        
        // Concurrent writes
        for i := 0; i < 10; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                m.Set(string(key), string(value))
            }()
        }
        
        // Concurrent reads
        for i := 0; i < 10; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                m.Get(string(key))
            }()
        }
        
        wg.Wait()
    })
}
```

### Monitoring Production Concurrency

**Runtime metrics:**

```go
import (
    "runtime"
    "time"
)

type Metrics struct {
    NumGoroutine int
    NumCPU       int
    GOMAXPROCS   int
    MemStats     runtime.MemStats
}

func collectMetrics() Metrics {
    var m Metrics
    
    m.NumGoroutine = runtime.NumGoroutine()
    m.NumCPU = runtime.NumCPU()
    m.GOMAXPROCS = runtime.GOMAXPROCS(0)
    
    runtime.ReadMemStats(&m.MemStats)
    
    return m
}

func monitorGoroutines(interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()
    
    for range ticker.C {
        metrics := collectMetrics()
        
        log.Printf("Goroutines: %d, Heap: %d MB, GC runs: %d",
            metrics.NumGoroutine,
            metrics.MemStats.HeapAlloc/1024/1024,
            metrics.MemStats.NumGC,
        )
        
        // Alert if goroutines growing
        if metrics.NumGoroutine > 10000 {
            log.Printf("WARNING: High goroutine count: %d", metrics.NumGoroutine)
        }
    }
}
```

**Prometheus integration:**

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
)

var (
    goroutineCount = promauto.NewGauge(prometheus.GaugeOpts{
        Name: "go_goroutines",
        Help: "Number of goroutines",
    })
    
    jobDuration = promauto.NewHistogram(prometheus.HistogramOpts{
        Name:    "job_duration_seconds",
        Help:    "Job processing duration",
        Buckets: prometheus.DefBuckets,
    })
)

func updateMetrics() {
    goroutineCount.Set(float64(runtime.NumGoroutine()))
}

func processJob() {
    start := time.Now()
    defer func() {
        jobDuration.Observe(time.Since(start).Seconds())
    }()
    
    // Process job
}
```

### Performance Optimization Checklist

**Concurrency optimization:**

- [ ] Profile first - identify actual bottlenecks
- [ ] Use worker pools for resource-limited operations
- [ ] Pre-allocate channels with appropriate buffer sizes
- [ ] Minimize lock contention (smaller critical sections)
- [ ] Use RWMutex for read-heavy workloads
- [ ] Consider sync.Pool for frequently allocated objects
- [ ] Batch operations when possible
- [ ] Use context for cancellation propagation
- [ ] Avoid goroutine leaks with proper cleanup
- [ ] Test with race detector enabled

**Memory optimization:**

- [ ] Reduce allocations in hot paths
- [ ] Reuse buffers and slices
- [ ] Use sync.Pool for temporary objects
- [ ] Pre-allocate slices when size is known
- [ ] Avoid boxing in interfaces when possible
- [ ] Keep structs cache-aligned
- [ ] Use pointers only when necessary
- [ ] Profile memory allocations
- [ ] Monitor GC metrics
- [ ] Tune GOGC if needed

**Scheduler optimization:**

- [ ] Don't create unlimited goroutines
- [ ] Use semaphores or worker pools
- [ ] Set appropriate GOMAXPROCS
- [ ] Avoid blocking in tight loops
- [ ] Use async preemption (Go 1.14+)
- [ ] Profile with scheduler traces
- [ ] Measure actual concurrency needs
- [ ] Test under load

### Common Anti-Patterns

**Anti-pattern: Goroutine per request without limit**

```go
// ❌ BAD: Unbounded goroutines
func handleRequests(requests <-chan Request) {
    for req := range requests {
        go processRequest(req)  // Can create millions!
    }
}

// ✅ GOOD: Worker pool
func handleRequests(requests <-chan Request) {
    for i := 0; i < 100; i++ {  // Fixed number of workers
        go worker(requests)
    }
}

func worker(requests <-chan Request) {
    for req := range requests {
        processRequest(req)
    }
}
```

**Anti-pattern: Select without default in hot loop**

```go
// ❌ BAD: Busy-waiting
for {
    select {
    case data := <-ch:
        process(data)
    }
}

// ✅ GOOD: Blocking select or default with delay
for {
    select {
    case data := <-ch:
        process(data)
    case <-time.After(time.Second):
        // Periodic check
    }
}
```

**Anti-pattern: Ignoring context cancellation**

```go
// ❌ BAD: Ignores cancellation
func longRunning(ctx context.Context) {
    for i := 0; i < 1000000; i++ {
        process(i)  // Ignores ctx
    }
}

// ✅ GOOD: Checks context
func longRunning(ctx context.Context) error {
    for i := 0; i < 1000000; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            process(i)
        }
    }
    return nil
}
```

### Conclusion

Mastering Go concurrency requires understanding:
1. Language primitives (goroutines, channels)
2. Synchronization tools (mutexes, atomics, context)
3. Runtime behavior (scheduler, memory, GC)
4. Common patterns (worker pools, pipelines, etc.)
5. Debugging techniques (race detector, profiling)
6. Performance optimization strategies

The key is to start simple, measure performance, and add complexity only when needed. Go's concurrency primitives are powerful but require discipline to use correctly.

