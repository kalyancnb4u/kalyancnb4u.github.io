---
title: "Java Mastery - Part 3: Advanced Java Features & Modern Java"
date: 2025-07-04 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, Advanced-Features, Modern-Java, Concurrency, Virtual-threads, Streams, Lambda, Functional-programming, Java8-21, Pattern-matching]
---

## Introduction

Welcome to Part 3 of the Complete Java Mastery series. In Parts 1 and 2, we covered Java fundamentals and JVM internals. Now we dive into **advanced features** that make modern Java powerful, expressive, and efficient.

Java has evolved dramatically since Java 8 (2014), which introduced lambda expressions and streams. Subsequent releases through Java 21 have added game-changing features like records, pattern matching, sealed classes, and virtual threads. Understanding these features is essential for writing modern, production-grade Java code.

### What You'll Learn in Part 3

* **Concurrency & Multithreading**: Thread fundamentals, synchronization, concurrent collections, and thread safety patterns
* **Virtual Threads (Project Loom)**: Revolutionary concurrency model for high-throughput applications
* **Functional Programming**: Lambda expressions, method references, and functional interfaces
* **Streams API**: Declarative data processing with sequential and parallel streams
* **Modern Java Features**: Java 8 through Java 21 features with practical examples
* **Pattern Matching**: New ways to work with data in Java 17-21
* **Date/Time API**: Modern temporal handling with java.time

This knowledge enables you to:
* Write concurrent code that scales
* Process data efficiently with streams
* Leverage modern Java syntax for cleaner code
* Build high-performance applications with virtual threads
* Stay current with Java evolution

---

## 3.1 Concurrency and Multithreading

Concurrency is one of the most complex and critical aspects of Java programming. Understanding threading is essential for building scalable, responsive applications.

### Thread Fundamentals

#### Creating Threads

**Method 1: Extend Thread class**

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + getName());
    }
}

// Usage
MyThread thread = new MyThread();
thread.start();  // Starts new thread, calls run() in that thread
// thread.run();  // ❌ Don't call run() directly! Executes in current thread
```

**Method 2: Implement Runnable** (preferred)

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

// Usage
Thread thread = new Thread(new MyRunnable());
thread.start();

// Or with lambda (Java 8+)
Thread thread = new Thread(() -> {
    System.out.println("Thread running: " + Thread.currentThread().getName());
});
thread.start();
```

**Method 3: Callable (returns result)**

```java
import java.util.concurrent.*;

Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
Integer result = future.get();  // Blocks until result available
executor.shutdown();
```

**Why Runnable over Thread?**

```java
// ❌ Extending Thread: Limited (single inheritance)
public class MyTask extends Thread {
    // Can't extend another class
}

// ✅ Implementing Runnable: Flexible
public class MyTask extends SomeClass implements Runnable {
    // Can extend other classes
    // Separates task from thread mechanics
}
```

#### Thread Lifecycle

```
Thread States:
┌─────────┐
│   NEW   │  Created but not started
└────┬────┘
     │ start()
     ▼
┌─────────┐
│RUNNABLE │  Executing or ready to execute
└────┬────┘
     │
     ├──► BLOCKED (waiting for monitor lock)
     │
     ├──► WAITING (waiting indefinitely)
     │     - Object.wait()
     │     - Thread.join()
     │     - LockSupport.park()
     │
     ├──► TIMED_WAITING (waiting with timeout)
     │     - Thread.sleep()
     │     - Object.wait(timeout)
     │     - Thread.join(timeout)
     │
     ▼
┌─────────┐
│TERMINATED  Execution completed
└─────────┘
```

**Checking Thread State:**

```java
Thread thread = new Thread(() -> {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

System.out.println(thread.getState());  // NEW
thread.start();
System.out.println(thread.getState());  // RUNNABLE
Thread.sleep(100);
System.out.println(thread.getState());  // TIMED_WAITING
thread.join();
System.out.println(thread.getState());  // TERMINATED
```

#### Thread Methods

```java
// Basic thread operations
Thread thread = new Thread(() -> {
    // Thread work
});

thread.start();              // Start thread execution
thread.join();               // Wait for thread to complete
thread.join(1000);           // Wait max 1 second
thread.interrupt();          // Interrupt thread
boolean interrupted = thread.isInterrupted();  // Check if interrupted
thread.setName("Worker-1");  // Set thread name
String name = thread.getName();
thread.setPriority(Thread.MAX_PRIORITY);  // 1-10, default 5
int priority = thread.getPriority();

// Static methods (operate on current thread)
Thread.sleep(1000);          // Sleep current thread
Thread.yield();              // Hint to scheduler to yield
Thread current = Thread.currentThread();
boolean interrupted = Thread.interrupted();  // Check and clear interrupt
```

**Thread Priority:**

```java
// Priority range: 1 (MIN_PRIORITY) to 10 (MAX_PRIORITY)
// Default: 5 (NORM_PRIORITY)

Thread lowPriority = new Thread(() -> {});
lowPriority.setPriority(Thread.MIN_PRIORITY);  // 1

Thread highPriority = new Thread(() -> {});
highPriority.setPriority(Thread.MAX_PRIORITY); // 10

// ⚠️ Priority is just a hint to OS scheduler
// Don't rely on it for correctness, only optimization
```

**Daemon Threads:**

```java
Thread daemon = new Thread(() -> {
    while (true) {
        System.out.println("Background work");
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
    }
});

daemon.setDaemon(true);  // Must set before start()
daemon.start();

// Daemon threads don't prevent JVM from exiting
// When only daemon threads remain, JVM exits
// Use for: background tasks, monitoring, cleanup
```

#### Thread Interruption

```java
// Proper interruption handling
Thread thread = new Thread(() -> {
    try {
        while (!Thread.currentThread().isInterrupted()) {
            // Do work
            System.out.println("Working...");
            Thread.sleep(100);  // InterruptedException clears interrupt flag!
        }
    } catch (InterruptedException e) {
        // Thread was interrupted during sleep
        System.out.println("Interrupted during sleep");
        // Re-interrupt if needed
        Thread.currentThread().interrupt();
    }
    System.out.println("Thread stopping gracefully");
});

thread.start();
Thread.sleep(500);
thread.interrupt();  // Request thread to stop
thread.join();       // Wait for graceful shutdown
```

**Best Practice: Check Interruption**

```java
// ✅ GOOD: Responsive to interruption
public void processItems(List<Item> items) {
    for (Item item : items) {
        if (Thread.currentThread().isInterrupted()) {
            System.out.println("Processing interrupted");
            return;
        }
        process(item);
    }
}

// ❌ BAD: Ignores interruption
public void processItems(List<Item> items) {
    for (Item item : items) {
        process(item);  // Can't be stopped!
    }
}
```

### Synchronization

**The Problem: Race Conditions**

```java
// ❌ NOT THREAD-SAFE
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // Read-modify-write: NOT ATOMIC!
        // Actually: temp = count; temp++; count = temp;
    }
    
    public int getCount() {
        return count;
    }
}

// Two threads calling increment():
// Thread 1: Read count (0) → Increment (1) → Write (1)
// Thread 2: Read count (0) → Increment (1) → Write (1)
// Result: count = 1 (should be 2!)
```

#### synchronized Keyword

**Synchronized Method:**

```java
// ✅ THREAD-SAFE
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;  // Only one thread at a time
    }
    
    public synchronized int getCount() {
        return count;  // Ensures visibility
    }
}

// Equivalent to:
public void increment() {
    synchronized(this) {
        count++;
    }
}
```

**Synchronized Block:**

```java
public class BankAccount {
    private double balance = 0;
    private final Object lock = new Object();  // Explicit lock object
    
    public void deposit(double amount) {
        synchronized(lock) {  // Only critical section synchronized
            balance += amount;
        }
        // Non-critical code here (not synchronized)
        sendNotification();
    }
    
    public void withdraw(double amount) {
        synchronized(lock) {
            if (balance >= amount) {
                balance -= amount;
            }
        }
    }
}
```

**Static Synchronization:**

```java
public class IdGenerator {
    private static int nextId = 0;
    
    // Synchronizes on class object (IdGenerator.class)
    public static synchronized int getNextId() {
        return nextId++;
    }
    
    // Equivalent to:
    public static int getNextId() {
        synchronized(IdGenerator.class) {
            return nextId++;
        }
    }
}
```

**How Synchronization Works:**

```
Every object has an intrinsic lock (monitor):

Thread 1                    Object Monitor
   │                        ┌───────────┐
   ├──► synchronized(obj) ──┤ Acquired  │
   │                        └───────────┘
   │    // Critical section
   │
   └──► exit synchronized    Lock released

Thread 2 (waiting)
   │
   ├──► synchronized(obj) ──┤ BLOCKED   │ (waiting for lock)
   │                        └───────────┘
   │
   └──► Acquires lock when Thread 1 releases
```

#### wait(), notify(), notifyAll()

**Producer-Consumer Pattern:**

```java
public class ProducerConsumer {
    private Queue<Integer> queue = new LinkedList<>();
    private final int CAPACITY = 10;
    private final Object lock = new Object();
    
    public void produce(int value) throws InterruptedException {
        synchronized(lock) {
            while (queue.size() == CAPACITY) {
                lock.wait();  // Release lock and wait
            }
            queue.offer(value);
            System.out.println("Produced: " + value);
            lock.notifyAll();  // Notify waiting consumers
        }
    }
    
    public int consume() throws InterruptedException {
        synchronized(lock) {
            while (queue.isEmpty()) {
                lock.wait();  // Release lock and wait
            }
            int value = queue.poll();
            System.out.println("Consumed: " + value);
            lock.notifyAll();  // Notify waiting producers
            return value;
        }
    }
}

// Producer thread
new Thread(() -> {
    for (int i = 0; i < 20; i++) {
        try {
            pc.produce(i);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).start();

// Consumer thread
new Thread(() -> {
    for (int i = 0; i < 20; i++) {
        try {
            pc.consume();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).start();
```

**Why while instead of if:**

```java
// ❌ BAD: Using if (spurious wakeup problem)
synchronized(lock) {
    if (queue.isEmpty()) {
        lock.wait();  // May wake up even if condition still false!
    }
    int value = queue.poll();  // NullPointerException if still empty!
}

// ✅ GOOD: Using while (handles spurious wakeups)
synchronized(lock) {
    while (queue.isEmpty()) {
        lock.wait();  // Re-check condition after waking up
    }
    int value = queue.poll();  // Safe
}
```

**notify() vs notifyAll():**

```java
// notify(): Wakes ONE waiting thread (arbitrary)
// - Use when: Only one thread can proceed
// - Faster but error-prone

// notifyAll(): Wakes ALL waiting threads
// - Use when: Multiple threads might be able to proceed
// - Safer, recommended default

synchronized(lock) {
    // Modify shared state
    lock.notifyAll();  // ✅ Safe default choice
}
```

#### volatile Keyword

**Purpose:** Ensures visibility across threads (no caching)

```java
// ❌ Without volatile: Thread may cache value
public class StopThread {
    private boolean stopRequested = false;
    
    public void run() {
        while (!stopRequested) {  // May never see update!
            // Do work
        }
    }
    
    public void stop() {
        stopRequested = true;  // Update may not be visible
    }
}

// ✅ With volatile: Guarantees visibility
public class StopThread {
    private volatile boolean stopRequested = false;
    
    public void run() {
        while (!stopRequested) {  // Always sees latest value
            // Do work
        }
    }
    
    public void stop() {
        stopRequested = true;  // Immediately visible to all threads
    }
}
```

**What volatile Does:**

```
Without volatile:
Thread 1: Write x = 1 → Cache → (eventually) → Main Memory
Thread 2: Read x ← Cache ← (may not see update)

With volatile:
Thread 1: Write x = 1 → Main Memory (immediately)
Thread 2: Read x ← Main Memory (always latest)
```

**volatile vs synchronized:**

| Aspect | volatile | synchronized |
|--------|----------|--------------|
| Atomicity | No (except for single read/write) | Yes |
| Visibility | Yes | Yes |
| Locking | No | Yes |
| Performance | Faster | Slower |
| Use case | Flags, single variables | Complex operations |

```java
// ✅ volatile: Good for flags
private volatile boolean running = true;

// ❌ volatile: NOT enough for compound operations
private volatile int count = 0;
public void increment() {
    count++;  // NOT ATOMIC! Use synchronized or AtomicInteger
}

// ✅ synchronized: For compound operations
private int count = 0;
public synchronized void increment() {
    count++;  // Safe
}
```

### Thread Safety Patterns

#### 1. Immutability

```java
// ✅ BEST: Immutable objects are inherently thread-safe
public final class ImmutablePoint {
    private final int x;
    private final int y;
    
    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int getX() { return x; }
    public int getY() { return y; }
    
    // No setters → Can't be modified → Thread-safe
}

// No synchronization needed!
ImmutablePoint point = new ImmutablePoint(10, 20);
// Share freely across threads
```

#### 2. Thread Confinement

```java
// Each thread has its own copy → No sharing → Thread-safe

// ThreadLocal
public class UserContext {
    private static ThreadLocal<User> currentUser = new ThreadLocal<>();
    
    public static void setUser(User user) {
        currentUser.set(user);
    }
    
    public static User getUser() {
        return currentUser.get();
    }
    
    public static void clear() {
        currentUser.remove();  // Important: prevent leaks
    }
}

// Each thread has its own User instance
// No synchronization needed
```

#### 3. Atomic Variables

```java
import java.util.concurrent.atomic.*;

// ✅ Lock-free thread safety
public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Atomic, lock-free
    }
    
    public int getCount() {
        return count.get();
    }
}

// Common atomic classes:
AtomicInteger atomicInt = new AtomicInteger(0);
AtomicLong atomicLong = new AtomicLong(0);
AtomicBoolean atomicBool = new AtomicBoolean(false);
AtomicReference<User> atomicRef = new AtomicReference<>(user);

// Atomic operations:
int previous = atomicInt.getAndIncrement();  // i++
int previous = atomicInt.getAndDecrement();  // i--
int previous = atomicInt.incrementAndGet();  // ++i
int previous = atomicInt.decrementAndGet();  // --i
atomicInt.addAndGet(5);                      // i += 5
boolean success = atomicInt.compareAndSet(0, 1);  // CAS
```

**Compare-And-Swap (CAS):**

```java
// Atomic CAS operation
public void incrementIfLessThan100() {
    int current;
    int newValue;
    do {
        current = atomicInt.get();
        if (current >= 100) return;
        newValue = current + 1;
    } while (!atomicInt.compareAndSet(current, newValue));
    // Retry if another thread modified value
}

// This is lock-free! No blocking, no deadlock possible
```

#### 4. Concurrent Collections

```java
import java.util.concurrent.*;

// ❌ BAD: Regular collections aren't thread-safe
Map<String, User> users = new HashMap<>();
// Multiple threads → ConcurrentModificationException or corruption

// ❌ BAD: Synchronized wrapper (low performance)
Map<String, User> users = Collections.synchronizedMap(new HashMap<>());
// Every operation locks entire map

// ✅ GOOD: ConcurrentHashMap (high performance)
Map<String, User> users = new ConcurrentHashMap<>();
// Fine-grained locking, high concurrency

// Other concurrent collections:
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
ConcurrentLinkedQueue<Task> queue = new ConcurrentLinkedQueue<>();
BlockingQueue<Task> blockingQueue = new LinkedBlockingQueue<>();
```

**ConcurrentHashMap Example:**

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic operations
map.putIfAbsent("key", 1);  // Add if not present
map.computeIfAbsent("key", k -> expensiveComputation(k));
map.computeIfPresent("key", (k, v) -> v + 1);
map.compute("key", (k, v) -> (v == null) ? 1 : v + 1);
map.merge("key", 1, Integer::sum);  // Atomic increment

// Example: Thread-safe counter
ConcurrentHashMap<String, AtomicInteger> counters = new ConcurrentHashMap<>();

public void increment(String key) {
    counters.computeIfAbsent(key, k -> new AtomicInteger(0))
            .incrementAndGet();
}
```

### Deadlock

**The Classic Deadlock:**

```java
// ❌ DEADLOCK SCENARIO
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {           // Thread 1 acquires lock1
            System.out.println("Method1: Acquired lock1");
            synchronized(lock2) {       // Waits for lock2
                System.out.println("Method1: Acquired lock2");
            }
        }
    }
    
    public void method2() {
        synchronized(lock2) {           // Thread 2 acquires lock2
            System.out.println("Method2: Acquired lock2");
            synchronized(lock1) {       // Waits for lock1
                System.out.println("Method2: Acquired lock1");
            }
        }
    }
}

// Thread 1 calls method1(), Thread 2 calls method2()
// Thread 1: Has lock1, waiting for lock2
// Thread 2: Has lock2, waiting for lock1
// DEADLOCK! Both threads wait forever
```

**Preventing Deadlock:**

```java
// ✅ SOLUTION 1: Lock ordering (always acquire in same order)
public class NoDeadlock {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {      // Always lock1 first
            synchronized(lock2) {  // Then lock2
                // Work
            }
        }
    }
    
    public void method2() {
        synchronized(lock1) {      // Same order: lock1 first
            synchronized(lock2) {  // Then lock2
                // Work
            }
        }
    }
}

// ✅ SOLUTION 2: Lock timeout (try-lock with timeout)
Lock lock1 = new ReentrantLock();
Lock lock2 = new ReentrantLock();

public void method() throws InterruptedException {
    while (true) {
        if (lock1.tryLock(1, TimeUnit.SECONDS)) {
            try {
                if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                    try {
                        // Work with both locks
                        return;
                    } finally {
                        lock2.unlock();
                    }
                }
            } finally {
                lock1.unlock();
            }
        }
        // Retry if couldn't acquire both locks
    }
}

// ✅ SOLUTION 3: Avoid nested locks
// Only hold one lock at a time when possible
```

**Detecting Deadlock:**

```bash
# Thread dump
jstack <pid> > threads.txt

# Look for:
# "Found one Java-level deadlock:"
# Shows which threads are waiting for which locks
```

### Executor Framework

**Why Use Executors?**

```java
// ❌ BAD: Manual thread management
for (int i = 0; i < 1000; i++) {
    new Thread(() -> {
        // Task
    }).start();
}
// Creates 1000 threads! High overhead, resource exhaustion

// ✅ GOOD: Thread pool
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 1000; i++) {
    executor.submit(() -> {
        // Task
    });
}
executor.shutdown();
// Reuses 10 threads for 1000 tasks
```

**Executor Types:**

```java
// 1. Fixed thread pool (most common)
ExecutorService executor = Executors.newFixedThreadPool(10);
// Fixed number of threads, bounded queue

// 2. Cached thread pool
ExecutorService executor = Executors.newCachedThreadPool();
// Creates threads as needed, reuses idle threads
// Good for: Many short-lived tasks

// 3. Single thread executor
ExecutorService executor = Executors.newSingleThreadExecutor();
// Single worker thread, tasks executed sequentially
// Good for: Sequential task processing

// 4. Scheduled executor
ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);
// Supports delayed and periodic task execution

// 5. Work-stealing pool (Java 8+)
ExecutorService executor = Executors.newWorkStealingPool();
// Uses ForkJoinPool, good for recursive tasks
```

**Using Executors:**

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

// Submit Runnable (no return value)
executor.submit(() -> {
    System.out.println("Task executed");
});

// Submit Callable (returns Future)
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});

try {
    Integer result = future.get();  // Blocks until result available
    System.out.println("Result: " + result);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}

// Shutdown (important!)
executor.shutdown();  // No new tasks accepted, waits for existing
// executor.shutdownNow();  // Interrupts running tasks

// Wait for termination
if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
    executor.shutdownNow();  // Force shutdown if timeout
}
```

**Custom ThreadPoolExecutor:**

```java
import java.util.concurrent.*;

ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10,                      // corePoolSize: min threads
    20,                      // maximumPoolSize: max threads
    60,                      // keepAliveTime
    TimeUnit.SECONDS,        // time unit
    new LinkedBlockingQueue<>(100),  // work queue (bounded!)
    new ThreadPoolExecutor.CallerRunsPolicy()  // rejection policy
);

// Rejection policies:
// - AbortPolicy: Throw RejectedExecutionException (default)
// - CallerRunsPolicy: Execute in calling thread
// - DiscardPolicy: Silently discard task
// - DiscardOldestPolicy: Discard oldest task in queue
```

**CompletableFuture (Java 8+):**

```java
import java.util.concurrent.CompletableFuture;

// Async computation
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Runs in ForkJoinPool.commonPool()
    return "Hello";
});

// Chain operations
future.thenApply(s -> s + " World")
      .thenApply(String::toUpperCase)
      .thenAccept(System.out::println)  // HELLO WORLD
      .join();  // Wait for completion

// Combine futures
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);

CompletableFuture<Integer> combined = future1.thenCombine(future2, 
    (a, b) -> a + b);
System.out.println(combined.join());  // 30

// Error handling
CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) {
        throw new RuntimeException("Error!");
    }
    return "Success";
}).exceptionally(ex -> {
    System.out.println("Handled: " + ex.getMessage());
    return "Default";
}).thenAccept(System.out::println);

// All of / Any of
CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2);
all.join();  // Waits for all to complete

CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);
Object result = any.join();  // Returns first completed
```

---

## 3.2 Virtual Threads (Project Loom)

**Virtual threads** (introduced in Java 19 as preview, standard in Java 21) are lightweight threads that dramatically simplify concurrent programming and enable massive scalability.

### The Problem with Platform Threads

**Platform threads** (traditional Java threads) are expensive:

```java
// ❌ Traditional approach: Limited by OS threads
ExecutorService executor = Executors.newFixedThreadPool(200);
// Can't scale beyond a few thousand threads
// Each thread: ~1MB of memory + OS overhead

// Example: Handling 10,000 concurrent requests
// Platform threads: Would need 10,000 OS threads (impossible!)
// Workaround: Thread pools + async/callback hell
```

**Platform Thread Limitations:**
* **Memory**: ~1MB per thread (stack size)
* **OS Overhead**: Limited by OS thread count (typically few thousand)
* **Context Switching**: Expensive when many threads
* **Can't scale** to millions of concurrent operations

### Enter Virtual Threads

**Virtual threads** are lightweight, managed by the JVM, not the OS:

```java
// ✅ Virtual threads: Can create millions!
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) {  // 1 million tasks!
        executor.submit(() -> {
            // Each task gets its own virtual thread
            Thread.sleep(Duration.ofSeconds(1));
            return "Result";
        });
    }
}  // All complete successfully

// 1 million virtual threads: Only a few MB of memory!
// Mapped to small number of OS threads (carrier threads)
```

**Key Differences:**

| Aspect | Platform Threads | Virtual Threads |
|--------|-----------------|-----------------|
| Managed by | Operating System | JVM |
| Memory | ~1MB per thread | Few KB per thread |
| Max count | Few thousand | Millions |
| Cost | Expensive to create | Cheap to create |
| Blocking | Blocks OS thread | Only blocks virtual thread |
| Use case | Heavy computation | I/O-bound tasks |

### Creating Virtual Threads

**Method 1: Thread.ofVirtual()**

```java
// Create and start a virtual thread
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread: " + Thread.currentThread());
});
vThread.join();

// Named virtual thread
Thread vThread = Thread.ofVirtual()
    .name("worker-", 0)  // Names: worker-0, worker-1, ...
    .start(() -> {
        // Work
    });

// Factory for creating virtual threads
ThreadFactory factory = Thread.ofVirtual().factory();
Thread vt1 = factory.newThread(() -> { /* task 1 */ });
Thread vt2 = factory.newThread(() -> { /* task 2 */ });
```

**Method 2: Executors.newVirtualThreadPerTaskExecutor()**

```java
// Executor that creates a new virtual thread for each task
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        int taskId = i;
        executor.submit(() -> {
            System.out.println("Task " + taskId + " in " + Thread.currentThread());
            Thread.sleep(Duration.ofSeconds(1));
            return taskId;
        });
    }
}  // Auto-shutdown with try-with-resources
```

**Method 3: Thread.startVirtualThread()**

```java
// One-liner to start a virtual thread
Thread vThread = Thread.startVirtualThread(() -> {
    System.out.println("Quick virtual thread");
});
vThread.join();
```

### How Virtual Threads Work

**Carrier Threads:**

```
Virtual Threads (millions)    Carrier Threads (few)    OS Threads
┌────────────┐               ┌────────────┐
│ VThread 1  │───────┐       │  Carrier 1 │───────────► OS Thread 1
└────────────┘       │       └────────────┘
┌────────────┐       ├──────►┌────────────┐
│ VThread 2  │───────┤       │  Carrier 2 │───────────► OS Thread 2
└────────────┘       │       └────────────┘
┌────────────┐       │       ┌────────────┐
│ VThread 3  │───────┤       │  Carrier 3 │───────────► OS Thread 3
└────────────┘       │       └────────────┘
     ...            │             ...
┌────────────┐       │
│VThread 1M  │───────┘
└────────────┘

When virtual thread blocks (e.g., I/O):
- Unmounted from carrier thread
- Carrier thread runs another virtual thread
- When I/O completes, virtual thread remounted
```

**Mounting and Unmounting:**

```java
// Virtual thread execution
Thread.ofVirtual().start(() -> {
    // Running on carrier thread #1
    System.out.println("Before I/O");
    
    // Blocking I/O call
    var data = socket.read();  // Virtual thread unmounts
    // Carrier thread #1 now free to run other virtual threads
    
    // I/O completes, virtual thread remounts (maybe different carrier)
    // Now running on carrier thread #2 (potentially)
    System.out.println("After I/O");
});

// Key insight: Blocking doesn't block carrier thread!
```

### When to Use Virtual Threads

**✅ Perfect for:**

```java
// I/O-bound tasks
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<CompletableFuture<String>> futures = new ArrayList<>();
    
    for (String url : urls) {  // Thousands of URLs
        futures.add(CompletableFuture.supplyAsync(() -> {
            return httpClient.send(HttpRequest.newBuilder()
                .uri(URI.create(url))
                .build(), 
                HttpResponse.BodyHandlers.ofString())
                .body();
        }, executor));
    }
    
    // Wait for all
    CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
}

// High-concurrency servers
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    ServerSocket serverSocket = new ServerSocket(8080);
    while (true) {
        Socket client = serverSocket.accept();
        executor.submit(() -> handleClient(client));  // One thread per client!
    }
}
```

**❌ Not ideal for:**

```java
// CPU-bound tasks
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1000; i++) {
        executor.submit(() -> {
            // Heavy computation (no blocking)
            computePrimesUpTo(1_000_000);
            // Virtual threads provide no benefit here
            // Use regular thread pool with parallelism = CPU count
        });
    }
}

// Better for CPU-bound:
ExecutorService executor = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);
```

### Structured Concurrency

**Structured concurrency** (preview in Java 19-21) ensures that concurrent subtasks complete before the parent task completes.

```java
import java.util.concurrent.StructuredTaskScope;

// ✅ Structured concurrency
String result = StructuredTaskScope.ShutdownOnFailure scope = new StructuredTaskScope.ShutdownOnFailure();

try (scope) {
    // Fork subtasks
    Future<String> user = scope.fork(() -> fetchUser(userId));
    Future<List<Order>> orders = scope.fork(() -> fetchOrders(userId));
    Future<Profile> profile = scope.fork(() -> fetchProfile(userId));
    
    // Join (wait for all)
    scope.join();
    scope.throwIfFailed();  // Throw if any failed
    
    // All completed successfully
    return new UserData(user.resultNow(), orders.resultNow(), profile.resultNow());
} catch (ExecutionException | InterruptedException e) {
    // Handle error
}

// Benefits:
// 1. All subtasks guaranteed to complete before parent returns
// 2. If parent cancelled, all subtasks cancelled
// 3. No orphaned threads
// 4. Clear ownership and lifecycle
```

**ShutdownOnSuccess** (first success wins):

```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    // Try multiple servers
    scope.fork(() -> fetchFromServer1());
    scope.fork(() -> fetchFromServer2());
    scope.fork(() -> fetchFromServer3());
    
    scope.join();
    
    String result = scope.result();  // First successful result
    return result;
}
```

### Virtual Threads Best Practices

**DO:**

```java
// ✅ Use virtual threads for I/O-bound tasks
Thread.ofVirtual().start(() -> {
    var response = httpClient.send(request, handler);
    processResponse(response);
});

// ✅ Create many virtual threads (don't pool them)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (var task : tasks) {
        executor.submit(task);  // Create new virtual thread per task
    }
}

// ✅ Block freely in virtual threads
Thread.ofVirtual().start(() -> {
    Thread.sleep(Duration.ofSeconds(10));  // Fine! Doesn't block carrier
    var data = blockingIO();               // Fine!
});
```

**DON'T:**

```java
// ❌ Don't pool virtual threads
ExecutorService pool = Executors.newFixedThreadPool(100);  // Wrong!
// Virtual threads are cheap, create as needed

// ❌ Don't use for CPU-bound tasks
Thread.ofVirtual().start(() -> {
    intensiveComputation();  // No benefit, wastes resources
});

// ❌ Don't use synchronized with virtual threads extensively
synchronized(lock) {  // Pins virtual thread to carrier
    // Long operation
}
// Use ReentrantLock instead for long operations

// ✅ Better:
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // Long operation - virtual thread can unmount
} finally {
    lock.unlock();
}
```

**Pinning Issue:**

```java
// ⚠️ Pinning: Virtual thread stuck to carrier thread
synchronized(lock) {
    blockingOperation();  // Virtual thread CAN'T unmount
    // Carrier thread blocked until this completes
}

// ✅ Solution: Use ReentrantLock
private final ReentrantLock lock = new ReentrantLock();

lock.lock();
try {
    blockingOperation();  // Virtual thread CAN unmount
} finally {
    lock.unlock();
}
```

### Real-World Example: Web Server

**Before Virtual Threads:**

```java
// Traditional thread pool approach
ExecutorService threadPool = Executors.newFixedThreadPool(200);

ServerSocket serverSocket = new ServerSocket(8080);
while (true) {
    Socket client = serverSocket.accept();
    threadPool.submit(() -> {
        try {
            handleClient(client);
        } catch (IOException e) {
            e.printStackTrace();
        }
    });
}

// Limited to ~200 concurrent connections
// Each connection holds a thread even when idle
```

**With Virtual Threads:**

```java
// One virtual thread per connection!
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    ServerSocket serverSocket = new ServerSocket(8080);
    while (true) {
        Socket client = serverSocket.accept();
        executor.submit(() -> {
            try {
                handleClient(client);  // Can handle millions of connections!
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }
}

private void handleClient(Socket client) throws IOException {
    BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
    PrintWriter out = new PrintWriter(client.getOutputStream(), true);
    
    String request = in.readLine();  // Blocks, but doesn't block carrier thread
    String response = processRequest(request);
    out.println(response);
    
    client.close();
}
```

### Monitoring Virtual Threads

```java
// Enable virtual thread monitoring
-Djdk.tracePinnedThreads=full

// Thread dump includes virtual threads
jcmd <pid> Thread.dump_to_file -format=json vthreads.json

// JFR events for virtual threads
-XX:StartFlightRecording=filename=recording.jfr

// Count virtual threads
long vThreadCount = Thread.getAllStackTraces().keySet().stream()
    .filter(Thread::isVirtual)
    .count();
```

---

## 3.3 Functional Programming Features

Java 8 introduced lambda expressions and the Streams API, fundamentally changing how we write Java code. Understanding functional programming is essential for modern Java.

### Lambda Expressions

**Syntax:**

```java
// Traditional anonymous class
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda expression (concise!)
Runnable r2 = () -> System.out.println("Hello");

// Lambda with parameters
Comparator<String> c1 = (String s1, String s2) -> s1.compareTo(s2);

// Type inference (even shorter)
Comparator<String> c2 = (s1, s2) -> s1.compareTo(s2);

// Block body
Comparator<String> c3 = (s1, s2) -> {
    int result = s1.compareTo(s2);
    return result;
};

// Single parameter (parentheses optional)
Function<String, Integer> length = s -> s.length();

// No parameters
Supplier<Double> random = () -> Math.random();
```

**Functional Interface:**

```java
// Functional interface: Exactly ONE abstract method
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // Can have default methods
    default int square(int x) {
        return calculate(x, x);
    }
    
    // Can have static methods
    static int add(int a, int b) {
        return a + b;
    }
}

// Usage with lambda
Calculator addition = (a, b) -> a + b;
Calculator multiplication = (a, b) -> a * b;

System.out.println(addition.calculate(5, 3));        // 8
System.out.println(multiplication.calculate(5, 3));  // 15
```

**Variable Capture:**

```java
// Variables must be effectively final
int factor = 2;  // Effectively final (not modified)

Function<Integer, Integer> multiply = x -> x * factor;  // ✅ OK

factor = 3;  // ❌ Error! Now not effectively final
// Lambda captures value at time of creation, not reference
```

### Method References

**Four types of method references:**

```java
// 1. Static method reference
Function<String, Integer> parser1 = Integer::parseInt;
// Equivalent to: s -> Integer.parseInt(s)

// 2. Instance method reference (specific object)
String prefix = "Hello, ";
Function<String, String> greeter = prefix::concat;
// Equivalent to: s -> prefix.concat(s)

// 3. Instance method reference (arbitrary object)
Function<String, String> upper = String::toUpperCase;
// Equivalent to: s -> s.toUpperCase()

BiPredicate<String, String> contains = String::contains;
// Equivalent to: (s1, s2) -> s1.contains(s2)

// 4. Constructor reference
Supplier<List<String>> listFactory = ArrayList::new;
// Equivalent to: () -> new ArrayList<>()

Function<Integer, int[]> arrayFactory = int[]::new;
// Equivalent to: size -> new int[size]
```

**Usage Examples:**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Method reference
names.forEach(System.out::println);

// Equivalent lambda
names.forEach(name -> System.out.println(name));

// Chaining
names.stream()
    .map(String::toUpperCase)
    .map(String::trim)
    .forEach(System.out::println);
```

### Functional Interfaces in java.util.function

**Core Functional Interfaces:**

```java
// 1. Predicate<T>: T → boolean
Predicate<String> isEmpty = String::isEmpty;
Predicate<Integer> isEven = n -> n % 2 == 0;

System.out.println(isEmpty.test(""));       // true
System.out.println(isEven.test(4));         // true

// Combining predicates
Predicate<String> isLong = s -> s.length() > 5;
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> combined = isLong.and(startsWithA);

// 2. Function<T, R>: T → R
Function<String, Integer> length = String::length;
Function<Integer, String> toString = Object::toString;

System.out.println(length.apply("Hello"));  // 5

// Composing functions
Function<String, Integer> lengthSquared = length.andThen(x -> x * x);
System.out.println(lengthSquared.apply("Hi"));  // 4

// 3. Consumer<T>: T → void
Consumer<String> printer = System.out::println;
printer.accept("Hello");  // Prints "Hello"

// Chaining consumers
Consumer<String> logger = s -> log.info(s);
Consumer<String> combined = printer.andThen(logger);

// 4. Supplier<T>: () → T
Supplier<Double> random = Math::random;
Supplier<String> uuid = UUID.randomUUID()::toString;

System.out.println(random.get());  // Random number

// 5. BiFunction<T, U, R>: (T, U) → R
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(3, 5));  // 8

// 6. BiPredicate<T, U>: (T, U) → boolean
BiPredicate<String, String> startsWith = String::startsWith;
System.out.println(startsWith.test("Hello", "Hel"));  // true

// 7. BiConsumer<T, U>: (T, U) → void
BiConsumer<String, Integer> printWithIndex = (s, i) -> 
    System.out.println(i + ": " + s);

// 8. UnaryOperator<T>: T → T (special case of Function)
UnaryOperator<Integer> square = x -> x * x;
System.out.println(square.apply(5));  // 25

// 9. BinaryOperator<T>: (T, T) → T (special case of BiFunction)
BinaryOperator<Integer> max = Integer::max;
System.out.println(max.apply(5, 10));  // 10
```

**Primitive Specializations** (avoid boxing):

```java
// IntPredicate instead of Predicate<Integer>
IntPredicate isEven = n -> n % 2 == 0;
isEven.test(4);  // No boxing!

// IntFunction<R> instead of Function<Integer, R>
IntFunction<String> toString = i -> String.valueOf(i);

// ToIntFunction<T> instead of Function<T, Integer>
ToIntFunction<String> length = String::length;

// IntConsumer instead of Consumer<Integer>
IntConsumer printer = System.out::println;

// IntSupplier instead of Supplier<Integer>
IntSupplier random = () -> (int) (Math.random() * 100);

// IntUnaryOperator instead of UnaryOperator<Integer>
IntUnaryOperator square = x -> x * x;

// IntBinaryOperator instead of BinaryOperator<Integer>
IntBinaryOperator add = (a, b) -> a + b;

// Similarly: LongPredicate, DoublePredicate, etc.
```

---

## 3.4 Streams API

The **Streams API** (Java 8+) provides a declarative approach to processing collections of data.

### Stream Creation

```java
// From collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();
Stream<String> parallelStream = list.parallelStream();

// From array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// From values
Stream<String> stream3 = Stream.of("a", "b", "c");

// Empty stream
Stream<String> empty = Stream.empty();

// Infinite streams
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);  // 0, 1, 2, 3, ...
Stream<Double> random = Stream.generate(Math::random);

// Bounded infinite stream
Stream<Integer> bounded = Stream.iterate(0, n -> n < 10, n -> n + 1);  // Java 9+

// From file
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
}

// From string
IntStream chars = "Hello".chars();  // Stream of char codes

// Range
IntStream range = IntStream.range(1, 5);        // 1, 2, 3, 4
IntStream rangeClosed = IntStream.rangeClosed(1, 5);  // 1, 2, 3, 4, 5
```

### Intermediate Operations (Lazy)

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// filter: Keep elements matching predicate
names.stream()
    .filter(name -> name.length() > 3)
    .forEach(System.out::println);  // Bob, Charlie, David

// map: Transform elements
names.stream()
    .map(String::toUpperCase)
    .forEach(System.out::println);  // ALICE, BOB, CHARLIE, DAVID

List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());  // [5, 3, 7, 5]

// flatMap: Flatten nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);

List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // [1, 2, 3, 4, 5, 6]

// distinct: Remove duplicates
List<Integer> distinct = Stream.of(1, 2, 2, 3, 3, 3)
    .distinct()
    .collect(Collectors.toList());  // [1, 2, 3]

// sorted: Sort elements
names.stream()
    .sorted()
    .forEach(System.out::println);  // Alice, Bob, Charlie, David

names.stream()
    .sorted(Comparator.reverseOrder())
    .forEach(System.out::println);  // David, Charlie, Bob, Alice

// peek: Debug/side-effects (intermediate operation)
names.stream()
    .filter(name -> name.length() > 3)
    .peek(name -> System.out.println("Filtered: " + name))
    .map(String::toUpperCase)
    .peek(name -> System.out.println("Mapped: " + name))
    .collect(Collectors.toList());

// limit: Take first n elements
Stream.iterate(0, n -> n + 1)
    .limit(5)
    .forEach(System.out::println);  // 0, 1, 2, 3, 4

// skip: Skip first n elements
Stream.of(1, 2, 3, 4, 5)
    .skip(2)
    .forEach(System.out::println);  // 3, 4, 5

// takeWhile: Take while predicate true (Java 9+)
Stream.of(1, 2, 3, 4, 5, 1, 2)
    .takeWhile(n -> n < 4)
    .forEach(System.out::println);  // 1, 2, 3

// dropWhile: Drop while predicate true (Java 9+)
Stream.of(1, 2, 3, 4, 5, 1, 2)
    .dropWhile(n -> n < 4)
    .forEach(System.out::println);  // 4, 5, 1, 2
```

### Terminal Operations (Eager)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// forEach: Perform action on each element
numbers.stream().forEach(System.out::println);

// forEachOrdered: Ordered forEach (useful for parallel streams)
numbers.parallelStream().forEachOrdered(System.out::println);

// collect: Accumulate into collection
List<Integer> list = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // [2, 4]

Set<Integer> set = numbers.stream()
    .collect(Collectors.toSet());

// toArray: Convert to array
Integer[] array = numbers.stream().toArray(Integer[]::new);

// reduce: Combine elements
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);  // 15

int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 15 (with identity)

int sum = numbers.stream()
    .reduce(0, Integer::sum);  // Clearer

// count: Count elements
long count = numbers.stream()
    .filter(n -> n > 2)
    .count();  // 3

// anyMatch: Check if any element matches
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // true

// allMatch: Check if all elements match
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);  // true

// noneMatch: Check if no elements match
boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);  // true

// findFirst: Get first element
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 3)
    .findFirst();  // Optional[4]

// findAny: Get any element (nondeterministic in parallel)
Optional<Integer> any = numbers.parallelStream()
    .filter(n -> n > 3)
    .findAny();  // Optional[4] or Optional[5]

// min/max: Find minimum/maximum
Optional<Integer> min = numbers.stream().min(Integer::compareTo);  // Optional[1]
Optional<Integer> max = numbers.stream().max(Integer::compareTo);  // Optional[5]
```

### Collectors

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 30, "Engineering"),
    new Person("Bob", 25, "Sales"),
    new Person("Charlie", 30, "Engineering"),
    new Person("David", 35, "Sales")
);

// toList, toSet, toCollection
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

Set<String> uniqueNames = people.stream()
    .map(Person::getName)
    .collect(Collectors.toSet());

TreeSet<String> sortedNames = people.stream()
    .map(Person::getName)
    .collect(Collectors.toCollection(TreeSet::new));

// joining: Concatenate strings
String allNames = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining());  // "AliceBobCharlieDavid"

String csv = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));  // "Alice, Bob, Charlie, David"

String formatted = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", ", "[", "]"));  // "[Alice, Bob, Charlie, David]"

// counting
long count = people.stream()
    .collect(Collectors.counting());  // 4

// summingInt, averagingInt, summarizingInt
int totalAge = people.stream()
    .collect(Collectors.summingInt(Person::getAge));  // 120

double avgAge = people.stream()
    .collect(Collectors.averagingInt(Person::getAge));  // 30.0

IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));
// count=4, sum=120, min=25, average=30.0, max=35

// groupingBy: Group by key
Map<String, List<Person>> byDept = people.stream()
    .collect(Collectors.groupingBy(Person::getDepartment));
// {Engineering=[Alice, Charlie], Sales=[Bob, David]}

Map<Integer, List<Person>> byAge = people.stream()
    .collect(Collectors.groupingBy(Person::getAge));
// {25=[Bob], 30=[Alice, Charlie], 35=[David]}

// groupingBy with downstream collector
Map<String, Long> countByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.counting()
    ));
// {Engineering=2, Sales=2}

Map<String, Double> avgAgeByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.averagingInt(Person::getAge)
    ));
// {Engineering=30.0, Sales=30.0}

// partitioningBy: Split into two groups (true/false)
Map<Boolean, List<Person>> partitioned = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));
// {false=[Bob], true=[Alice, Charlie, David]}

// toMap: Create map
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge
    ));

// Handle duplicate keys
Map<String, Person> byName = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        p -> p,
        (existing, replacement) -> existing  // Keep first
    ));

// maxBy, minBy
Optional<Person> oldest = people.stream()
    .collect(Collectors.maxBy(Comparator.comparing(Person::getAge)));

// collectingAndThen: Transform result
List<String> unmodifiableNames = people.stream()
    .map(Person::getName)
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));
```

### Parallel Streams

```java
// Create parallel stream
List<Integer> numbers = IntStream.rangeClosed(1, 1000)
    .boxed()
    .collect(Collectors.toList());

// Parallel processing
long sum = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();

// Convert to parallel
long count = numbers.stream()
    .parallel()
    .filter(n -> n % 2 == 0)
    .count();

// Convert back to sequential
long count2 = numbers.parallelStream()
    .sequential()
    .count();
```

**When to Use Parallel Streams:**

```java
// ✅ GOOD: Large dataset, CPU-intensive operations
List<Integer> large = IntStream.rangeClosed(1, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

long sum = large.parallelStream()
    .map(n -> expensiveComputation(n))
    .reduce(0, Integer::sum);

// ❌ BAD: Small dataset (overhead > benefit)
List<Integer> small = Arrays.asList(1, 2, 3, 4, 5);
small.parallelStream()  // Overkill
    .map(n -> n * 2)
    .collect(Collectors.toList());

// ❌ BAD: I/O operations (already bottlenecked)
files.parallelStream()
    .map(file -> readFile(file))  // I/O bound, not CPU bound
    .collect(Collectors.toList());

// ❌ BAD: Stateful operations
List<Integer> result = numbers.parallelStream()
    .map(n -> {
        counter++;  // Race condition!
        return n * 2;
    })
    .collect(Collectors.toList());
```

### Stream Performance Tips

```java
// ✅ GOOD: Filter early
people.stream()
    .filter(p -> p.getAge() > 25)  // Reduce dataset first
    .map(Person::getName)
    .collect(Collectors.toList());

// ❌ BAD: Filter late
people.stream()
    .map(Person::getName)
    .filter(name -> lookupAge(name) > 25)  // More work
    .collect(Collectors.toList());

// ✅ GOOD: Use primitive streams
int sum = numbers.stream()
    .mapToInt(Integer::intValue)  // Primitive stream
    .sum();

// ❌ BAD: Unnecessary boxing
int sum = numbers.stream()
    .reduce(0, Integer::sum);  // Boxing overhead

// ✅ GOOD: Short-circuit operations
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // Stops at first match

// ❌ BAD: Process all elements unnecessarily
boolean hasEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .count() > 0;  // Processes all elements
```

---

## 3.5 Modern Java Features Summary

### Java 8: Lambda & Streams Revolution

```java
// Lambda expressions
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println(name));

// Streams API
List<String> filtered = names.stream()
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());

// Optional
Optional<String> first = names.stream().findFirst();
first.ifPresent(System.out::println);

// Default methods in interfaces
interface Greeting {
    default void sayHello() {
        System.out.println("Hello");
    }
}

// Date/Time API
LocalDate today = LocalDate.now();
LocalDateTime now = LocalDateTime.now();
```

### Java 9: Modules & More

```java
// Module system
module com.myapp {
    requires java.sql;
    exports com.myapp.api;
}

// Collection factory methods
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);

// Stream enhancements
Stream.ofNullable(getValue());  // Handles null
Stream.iterate(0, n -> n < 10, n -> n + 1);  // Bounded iterate

// Private interface methods
interface MyInterface {
    private void helper() {
        // Helper for default methods
    }
}
```

### Java 10-11: var & HTTP Client

```java
// Local variable type inference
var list = new ArrayList<String>();  // Inferred as ArrayList<String>
var map = Map.of("key", "value");

// HTTP Client API (standard in 11)
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .build();
HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());
```

### Java 14-17: Records, Pattern Matching, Sealed Classes

```java
// Records (Java 16)
record Point(int x, int y) { }
Point p = new Point(10, 20);
System.out.println(p.x());  // Accessor

// Pattern matching for instanceof (Java 16)
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}

// Sealed classes (Java 17)
sealed class Shape permits Circle, Rectangle { }
final class Circle extends Shape { }
final class Rectangle extends Shape { }

// Text blocks (Java 15)
String json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;

// Switch expressions (Java 14)
String result = switch (day) {
    case MONDAY, FRIDAY -> "Good day";
    case TUESDAY -> "Okay day";
    default -> "Other day";
};
```

### Java 19-21: Virtual Threads, Pattern Matching

```java
// Virtual threads (Java 21)
Thread.startVirtualThread(() -> {
    System.out.println("Virtual thread");
});

// Pattern matching for switch (Java 21)
String formatted = switch (obj) {
    case Integer i -> String.format("int %d", i);
    case String s -> String.format("String %s", s);
    case null -> "null";
    default -> obj.toString();
};

// Record patterns (Java 21)
record Point(int x, int y) { }

if (obj instanceof Point(int x, int y)) {
    System.out.println("x: " + x + ", y: " + y);
}

// Sequenced collections (Java 21)
List<String> list = new ArrayList<>();
list.addFirst("first");
list.addLast("last");
String first = list.getFirst();
String last = list.getLast();
```

---

## Summary and Key Takeaways

### Concurrency Mastery

**Platform Threads:**
* Use for CPU-bound tasks
* Thread pools for resource management
* Synchronization for thread safety
* Atomic classes for lock-free operations

**Virtual Threads:**
* Revolutionary for I/O-bound tasks
* Cheap to create (millions possible)
* Simplifies concurrent programming
* Perfect for high-throughput servers

**Best Practices:**
* Prefer immutability
* Use concurrent collections
* Avoid deadlocks (lock ordering)
* Minimize synchronized blocks

### Functional Programming

**Lambda Expressions:**
* Concise syntax for functional interfaces
* Method references for brevity
* Effectively final variable capture

**Functional Interfaces:**
* Predicate, Function, Consumer, Supplier
* BiFunction, BiPredicate, BiConsumer
* Primitive specializations (avoid boxing)

**Benefits:**
* More declarative code
* Better composability
* Easier parallelization

### Streams API

**Power of Streams:**
* Declarative data processing
* Lazy evaluation (intermediate operations)
* Easy parallelization
* Rich set of collectors

**When to Use:**
* Data transformations
* Filtering and mapping
* Aggregations
* Collection conversions

**When Not to Use:**
* Simple loops (readability)
* Modifying external state
* Very small datasets

### Modern Java Evolution

**Key Features to Master:**
* **Java 8**: Lambdas, Streams, Optional
* **Java 9**: Modules, Factory methods
* **Java 11**: var, HTTP Client
* **Java 14-16**: Records, Text blocks
* **Java 17**: Sealed classes, Pattern matching
* **Java 21**: Virtual threads, Enhanced pattern matching

**Adoption Strategy:**
* Stay on LTS versions (11, 17, 21)
* Test preview features
* Migrate gradually
* Leverage new syntax for cleaner code

### Production Checklist

**Concurrency:**
- [ ] Use appropriate thread model (platform vs virtual)
- [ ] Implement proper synchronization
- [ ] Avoid deadlocks
- [ ] Monitor thread pools
- [ ] Use concurrent collections

**Performance:**
- [ ] Profile before optimizing
- [ ] Use primitive streams when appropriate
- [ ] Consider parallel streams for large datasets
- [ ] Avoid unnecessary object creation
- [ ] Benchmark with JMH

**Code Quality:**
- [ ] Leverage modern Java features
- [ ] Use functional style where appropriate
- [ ] Keep code readable and maintainable
- [ ] Write comprehensive tests
- [ ] Document complex concurrency

### Frequently Asked Questions

**Q1: When should I use virtual threads vs platform threads?**

**A:** 
* **Virtual threads**: I/O-bound tasks (network calls, database queries, file I/O)
* **Platform threads**: CPU-bound tasks (heavy computation, data processing)

```java
// ✅ Virtual threads: I/O operations
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> httpClient.send(request, handler));
}

// ✅ Platform threads: CPU operations
ExecutorService executor = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);
executor.submit(() -> computeIntensiveTask());
```

---

**Q2: What's the difference between synchronized and ReentrantLock?**

**A:**

| Feature | synchronized | ReentrantLock |
|---------|-------------|---------------|
| Syntax | Keyword | Explicit lock/unlock |
| Fairness | No | Optional |
| Try lock | No | Yes (tryLock) |
| Interruptible | No | Yes |
| Condition variables | wait/notify | Condition objects |
| Performance | Slightly faster (simple cases) | More features |

```java
// synchronized
synchronized(lock) {
    // Critical section
}

// ReentrantLock
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();  // Must unlock in finally!
}
```

---

**Q3: When should I use parallel streams?**

**A:** Use parallel streams when:
* Large dataset (> 10,000 elements)
* CPU-intensive operations
* Independent operations (no shared state)
* Have multiple CPU cores

Don't use when:
* Small dataset (overhead > benefit)
* I/O operations (already bottlenecked)
* Stateful operations (race conditions)
* Order matters and can't be parallel

```java
// ✅ Good: Large dataset, CPU-intensive
List<Integer> results = largeList.parallelStream()
    .map(n -> expensiveComputation(n))
    .collect(Collectors.toList());

// ❌ Bad: Small dataset
List<Integer> results = Arrays.asList(1, 2, 3).parallelStream()
    .map(n -> n * 2)  // Overkill!
    .collect(Collectors.toList());
```

### Interview Questions

**Question 1: Explain how virtual threads work and when you would use them.**

**Difficulty:** Senior

**Topics:** Concurrency, Modern Java

**Answer:**

Virtual threads are lightweight threads managed by the JVM (not the OS). They work through:

1. **Mounting/Unmounting**: Virtual threads run on "carrier threads" (platform threads)
2. **Cooperative Scheduling**: When a virtual thread blocks (I/O), it unmounts from the carrier
3. **Scalability**: Millions of virtual threads can run on few carrier threads

```java
// Example: Server handling 1M concurrent connections
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    while (true) {
        Socket client = serverSocket.accept();
        executor.submit(() -> handleClient(client));
        // Can handle millions of concurrent connections!
    }
}
```

**When to use:**
* High I/O concurrency (web servers, microservices)
* Blocking API calls
* Request-per-thread model

**When NOT to use:**
* CPU-bound tasks (use platform threads)
* Need fine-grained control (use platform threads)

---

**Question 2: What are the key differences between map and flatMap?**

**Difficulty:** Mid-Level

**Topics:** Streams API

**Answer:**

* **map**: Transforms each element (1-to-1 mapping)
* **flatMap**: Transforms and flattens (1-to-many mapping)

```java
// map: List<T> → Stream<R>
List<String> words = Arrays.asList("Hello", "World");
words.stream()
    .map(String::length)  // Stream<Integer>
    .collect(Collectors.toList());  // [5, 5]

// flatMap: List<T> → Stream<Stream<R>> → Stream<R>
List<String> sentences = Arrays.asList("Hello World", "Java Stream");
sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")))  // Flattens
    .collect(Collectors.toList());  // [Hello, World, Java, Stream]

// Use case: Nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // [1, 2, 3, 4]
```

---

**End of Part 3: Advanced Java Features & Modern Java**

### What's Next

In **Part 4: SDLC with Java**, we'll cover:
* Design patterns (Creational, Structural, Behavioral)
* Testing strategies (JUnit, Mockito, TestContainers)
* Build tools (Maven, Gradle)
* CI/CD pipelines
* Deployment strategies
* Spring Framework ecosystem
* Microservices architecture

### References

* [Java Language Specification](https://docs.oracle.com/javase/specs/)
* [Java Concurrency in Practice](https://jcip.net/)
* [Effective Java by Joshua Bloch](https://www.pearson.com/en-us/subject-catalog/p/effective-java/P200000000138)
* [Modern Java in Action](https://www.manning.com/books/modern-java-in-action)
* [Project Loom](https://openjdk.org/projects/loom/)
* [Virtual Threads Documentation](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)

---
