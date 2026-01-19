---
title: "Go Mastery - Supplement 8: Study Notes Template"
date: 2025-09-28 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Study, Notes, Template, Study-guide]
---

# Study Notes Template

## How to Use

Copy this template for each topic or section you study. Consistent note-taking helps retention and review.

---

## Topic: _______________________

**Date**: __________ | **Part/Section**: __________ | **Duration**: __________ min

---

### Key Concepts

**Main Idea**:


**Important Terms**:
- **Term 1**: Definition
- **Term 2**: Definition
- **Term 3**: Definition

---

### Core Code Examples

**Example 1**: Basic Usage
```go
// What it demonstrates:


// Code:

```

**Example 2**: Advanced Usage
```go
// What it demonstrates:


// Code:

```

**Example 3**: Common Pattern
```go
// What it demonstrates:


// Code:

```

---

### When to Use

**Use this when**:
- 
- 
- 

**Don't use this when**:
- 
- 
- 

**Alternatives**:
- 
- 

---

### Common Mistakes

**Mistake 1**:
```go
// ❌ BAD:


// Why it's bad:


// ✅ GOOD:

```

**Mistake 2**:
```go
// ❌ BAD:


// Why it's bad:


// ✅ GOOD:

```

---

### Interview Questions

**Q1**: 


**A1**: 


**Q2**: 


**A2**: 


**Q3**: 


**A3**: 


---

### Practice Problems

**Problem 1**:
- Description:
- Difficulty:
- Link:
- Status: [ ] Todo [ ] In Progress [ ] Completed

**Problem 2**:
- Description:
- Difficulty:
- Link:
- Status: [ ] Todo [ ] In Progress [ ] Completed

---

### Related Topics

**Prerequisites** (study before this):
- 
- 

**Next Topics** (study after this):
- 
- 

**Related Concepts**:
- 
- 

---

### Resources

**Documentation**:
- 

**Articles**:
- 
- 

**Videos**:
- 

**Code Examples**:
- 

---

### Personal Insights

**What I learned**:


**What confused me**:


**Questions to explore**:
- 
- 
- 

**Real-world applications**:


---

### Quick Reference Card

```
┌─────────────────────────────────────┐
│ [Topic Name]                        │
├─────────────────────────────────────┤
│ Key Syntax:                         │
│   - Syntax 1                        │
│   - Syntax 2                        │
│                                     │
│ Common Pattern:                     │
│   [code snippet]                    │
│                                     │
│ Gotchas:                            │
│   - Gotcha 1                        │
│   - Gotcha 2                        │
└─────────────────────────────────────┘
```

---

### Review Schedule

- [ ] Review after 1 day (Date: __________)
- [ ] Review after 1 week (Date: __________)
- [ ] Review after 1 month (Date: __________)
- [ ] Review before interview

---

## Example: Filled Template for Channels

### Topic: Go Channels

**Date**: Jan 11, 2025 | **Part/Section**: Part 2.3 | **Duration**: 60 min

---

### Key Concepts

**Main Idea**:
Channels are typed conduits for goroutine communication. They provide thread-safe data transfer and synchronization.

**Important Terms**:
- **Unbuffered Channel**: Synchronous communication, sender blocks until receiver ready
- **Buffered Channel**: Asynchronous up to capacity, sender only blocks when full
- **Channel Direction**: Can specify send-only or receive-only channels

---

### Core Code Examples

**Example 1**: Basic Usage
```go
// What it demonstrates: Create, send, receive

// Code:
ch := make(chan int)
go func() {
    ch <- 42  // Send
}()
value := <-ch // Receive
```

**Example 2**: Buffered Channel
```go
// What it demonstrates: Non-blocking send up to capacity

// Code:
ch := make(chan int, 3)
ch <- 1  // Doesn't block
ch <- 2
ch <- 3
// ch <- 4 would block here
```

**Example 3**: Range and Close
```go
// What it demonstrates: Iterate until closed

// Code:
ch := make(chan int, 10)
go func() {
    for i := 0; i < 10; i++ {
        ch <- i
    }
    close(ch)
}()

for value := range ch {
    fmt.Println(value)
}
```

---

### When to Use

**Use this when**:
- Goroutines need to communicate
- Need synchronization between goroutines
- Implementing producer-consumer pattern

**Don't use this when**:
- Simple mutex would suffice
- No communication needed, just mutual exclusion

**Alternatives**:
- sync.Mutex for shared state
- atomic operations for simple counters
- WaitGroup for just synchronization

---

### Common Mistakes

**Mistake 1**:
```go
// ❌ BAD: Sending on unbuffered channel without receiver
ch := make(chan int)
ch <- 42  // Deadlock!

// Why it's bad: Sender blocks forever

// ✅ GOOD:
ch := make(chan int, 1)  // Buffered
ch <- 42
// or
go func() { ch <- 42 }()  // Send in goroutine
```

**Mistake 2**:
```go
// ❌ BAD: Closing channel from receiver
ch := make(chan int)
go func() {
    value := <-ch
    close(ch)  // Receiver closing!
}()

// Why it's bad: Only sender should close

// ✅ GOOD:
ch := make(chan int)
go func() {
    ch <- 42
    close(ch)  // Sender closing
}()
```

---

### Interview Questions

**Q1**: What's the difference between buffered and unbuffered channels?

**A1**: Unbuffered (ch := make(chan int)) synchronizes sender and receiver - sender blocks until receiver ready. Buffered (ch := make(chan int, 5)) allows asynchronous communication up to capacity - sender only blocks when buffer full.

**Q2**: Can you receive from a closed channel?

**A2**: Yes. Receiving from closed channel returns zero value immediately. Use two-value receive to check: value, ok := <-ch (ok is false if closed).

**Q3**: What happens if you send to a closed channel?

**A3**: Panic! Only sender should close channels, and must ensure no more sends after close.

---

### Practice Problems

**Problem 1**:
- Description: Implement worker pool with 5 workers processing 100 jobs
- Difficulty: Medium
- Link: [Project or LeetCode]
- Status: [x] Completed

**Problem 2**:
- Description: Build pipeline: generator -> squarer -> printer
- Difficulty: Medium
- Link: [Your implementation]
- Status: [x] Completed

---

### Related Topics

**Prerequisites** (study before this):
- Goroutines
- Basic concurrency concepts

**Next Topics** (study after this):
- Select statement
- Context package
- Advanced channel patterns

**Related Concepts**:
- sync.WaitGroup
- sync.Mutex
- Concurrency patterns

---

### Resources

**Documentation**:
- https://go.dev/ref/spec#Channel_types

**Articles**:
- Go by Example: Channels
- Effective Go: Channels section

**Videos**:
- GopherCon: Channels and goroutines

**Code Examples**:
- https://gobyexample.com/channels

---

### Personal Insights

**What I learned**:
Channels make concurrent programming much easier than traditional locks. The "share memory by communicating" philosophy is powerful.

**What confused me**:
Initially confused about when to use buffered vs unbuffered. Realized unbuffered is for synchronization, buffered is for decoupling.

**Questions to explore**:
- How are channels implemented internally?
- Performance comparison: channels vs mutexes
- Best practices for channel capacity

**Real-world applications**:
- Worker pools for parallel processing
- Pipeline for data processing stages
- Rate limiting with time.Tick channel

---

### Quick Reference Card

```
┌─────────────────────────────────────┐
│ Channels                            │
├─────────────────────────────────────┤
│ Create:                             │
│   ch := make(chan Type)             │
│   ch := make(chan Type, capacity)   │
│                                     │
│ Operations:                         │
│   ch <- value     // Send           │
│   v := <-ch       // Receive        │
│   close(ch)       // Close          │
│                                     │
│ Range:                              │
│   for v := range ch { ... }         │
│                                     │
│ Gotchas:                            │
│   - Send on closed channel panics   │
│   - Only sender should close        │
│   - Unbuffered blocks until ready   │
└─────────────────────────────────────┘
```

---

### Review Schedule

- [x] Review after 1 day (Date: Jan 12, 2025)
- [x] Review after 1 week (Date: Jan 18, 2025)
- [ ] Review after 1 month (Date: Feb 11, 2025)
- [ ] Review before interview

---

**Use this template for every topic you study. Consistent note-taking = better retention!**
