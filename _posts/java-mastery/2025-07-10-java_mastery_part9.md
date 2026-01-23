---
title: "Complete Java Mastery Part 9: Performance & Optimization"
date: 2024-01-23 10:00:00 +0000
categories: [Java, Performance, Optimization]
tags: [java, performance, jvm-tuning, profiling, benchmarking, jmh, memory-optimization, garbage-collection, optimization]
---

## Introduction

Welcome to Part 9 of the Complete Java Mastery series. In Parts 1-8, we covered Java fundamentals through cloud-native deployment. Now we explore **Performance & Optimization** - making Java applications run faster, use less memory, and handle more load.

Performance optimization is both an art and science. It requires understanding the JVM, application behavior, and system resources. The key is **measure first, optimize second**—premature optimization wastes time, while data-driven optimization delivers results.

### What You'll Learn in Part 9

* **JVM Tuning**: Heap sizing, GC tuning, JVM flags
* **Profiling Tools**: JProfiler, VisualVM, async-profiler, Flight Recorder
* **Memory Optimization**: Leak detection, heap analysis, object allocation
* **Garbage Collection**: Understanding GC algorithms, tuning for latency/throughput
* **Benchmarking with JMH**: Writing accurate microbenchmarks
* **Database Optimization**: Query optimization, connection pooling, caching
* **Application Profiling**: Finding bottlenecks, CPU hotspots
* **Performance Monitoring**: Production monitoring, metrics, alerting

This knowledge enables you to:
* Tune JVM for optimal performance
* Find and fix performance bottlenecks
* Prevent memory leaks
* Write accurate benchmarks
* Optimize database queries
* Monitor production performance

---

## 9.1 JVM Tuning Fundamentals

### Memory Architecture Review

```
JVM Memory Structure:

┌─────────────────────────────────────────┐
│            Heap Memory                  │
│  ┌────────────┐    ┌─────────────────┐ │
│  │   Young    │    │      Old        │ │
│  │  (Eden +   │    │   Generation    │ │
│  │ Survivor)  │    │                 │ │
│  └────────────┘    └─────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│         Non-Heap Memory                 │
│  ┌────────────┐    ┌─────────────────┐ │
│  │ Metaspace  │    │  Code Cache     │ │
│  │ (Classes)  │    │  (JIT code)     │ │
│  └────────────┘    └─────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│       Thread Stacks (per thread)        │
│  Thread 1 │ Thread 2 │ ... │ Thread N  │
└─────────────────────────────────────────┘
```

### Heap Sizing

**Basic Heap Settings:**

```bash
# Set initial and max heap size (recommended: same value)
java -Xms2g -Xmx2g -jar app.jar

# Why same value?
# Xms != Xmx → JVM resizes heap → performance hit
# Xms == Xmx → Fixed heap → predictable performance
```

**Young Generation Sizing:**

```bash
# Set young generation size
java -Xms4g -Xmx4g -Xmn1g -jar app.jar

# Rule of thumb:
# Young = 25-40% of total heap
# For 4GB heap: 1-1.5GB young generation
```

**Calculating Heap Size:**

```bash
# For throughput-optimized app:
# Total memory: 8GB
# OS + other: 1GB
# Thread stacks: 1GB (500 threads × 2MB)
# Metaspace: 512MB
# Code cache: 256MB
# Direct memory: 512MB
# Remaining for heap: 4.7GB

java -Xms4g -Xmx4g \
     -Xmn1500m \
     -XX:MetaspaceSize=512m \
     -XX:MaxMetaspaceSize=512m \
     -XX:ReservedCodeCacheSize=256m \
     -XX:MaxDirectMemorySize=512m \
     -Xss2m \
     -jar app.jar
```

### Garbage Collection Basics

**GC Algorithms:**

```bash
# G1GC (default in Java 9+) - balanced
java -XX:+UseG1GC -jar app.jar

# ZGC - low latency (Java 15+)
java -XX:+UseZGC -jar app.jar

# Shenandoah - low latency
java -XX:+UseShenandoahGC -jar app.jar

# Parallel GC - high throughput
java -XX:+UseParallelGC -jar app.jar

# Serial GC - small apps
java -XX:+UseSerialGC -jar app.jar
```

**GC Algorithm Comparison:**

```
Algorithm     | Pause Time | Throughput | Heap Size  | Use Case
--------------|------------|------------|------------|------------------
G1GC          | Medium     | Good       | Any        | General purpose
ZGC           | Very Low   | Good       | Large      | Low-latency apps
Shenandoah    | Very Low   | Good       | Any        | Low-latency apps
Parallel GC   | High       | Excellent  | Any        | Batch processing
Serial GC     | Medium     | Poor       | Small      | Small apps/CLI
```

### G1GC Tuning (Most Common)

```bash
# G1GC with tuning
java -XX:+UseG1GC \
     -Xms4g -Xmx4g \
     -XX:MaxGCPauseMillis=200 \        # Target pause time
     -XX:G1HeapRegionSize=16m \        # Region size (default: auto)
     -XX:ParallelGCThreads=8 \         # Parallel GC threads
     -XX:ConcGCThreads=2 \             # Concurrent GC threads
     -XX:InitiatingHeapOccupancyPercent=45 \  # When to start marking
     -jar app.jar

# G1GC Regions:
# Heap divided into regions (default: 2048 regions)
# Region size: 1MB, 2MB, 4MB, 8MB, 16MB, 32MB
# Choose size: heap_size / 2048
```

### ZGC Tuning (Low Latency)

```bash
# ZGC - sub-millisecond pauses
java -XX:+UseZGC \
     -Xms8g -Xmx8g \
     -XX:ZCollectionInterval=5 \       # GC interval (seconds)
     -XX:ZAllocationSpikeTolerance=2 \ # Spike tolerance
     -XX:ConcGCThreads=4 \             # Concurrent threads
     -jar app.jar

# ZGC Benefits:
# - Pause times: <1ms (even for 100GB heaps!)
# - Scales to terabyte heaps
# - No generational collection

# ZGC Limitations:
# - Only on Linux/macOS (Windows in Java 16+)
# - Higher memory overhead
# - Requires Java 15+ for production
```

### Essential JVM Flags

```bash
# Production-ready JVM settings
java \
  # Memory
  -Xms4g -Xmx4g \
  -Xmn1500m \
  -XX:MetaspaceSize=256m \
  -XX:MaxMetaspaceSize=512m \
  
  # GC
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:ParallelGCThreads=8 \
  -XX:ConcGCThreads=2 \
  
  # GC Logging
  -Xlog:gc*:file=/var/log/gc.log:time,level,tags \
  -XX:+UseGCLogFileRotation \
  -XX:NumberOfGCLogFiles=10 \
  -XX:GCLogFileSize=50M \
  
  # Heap Dump on OutOfMemoryError
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/heapdump.hprof \
  -XX:OnOutOfMemoryError="kill -9 %p" \
  
  # Performance
  -XX:+UseStringDeduplication \     # Deduplicate strings
  -XX:+OptimizeStringConcat \       # Optimize string concatenation
  -XX:+UseCompressedOops \          # Compress object pointers
  
  # Diagnostics
  -XX:+UnlockDiagnosticVMOptions \
  -XX:NativeMemoryTracking=summary \
  
  # Container support
  -XX:+UseContainerSupport \
  -XX:MaxRAMPercentage=75.0 \
  
  -jar app.jar
```

### JVM Flag Categories

**Memory Management:**

```bash
# Heap
-Xms<size>              # Initial heap size
-Xmx<size>              # Maximum heap size
-Xmn<size>              # Young generation size
-Xss<size>              # Thread stack size

# Metaspace
-XX:MetaspaceSize=<size>     # Initial metaspace
-XX:MaxMetaspaceSize=<size>  # Max metaspace

# Direct Memory
-XX:MaxDirectMemorySize=<size>
```

**Garbage Collection:**

```bash
# Algorithm Selection
-XX:+UseG1GC
-XX:+UseZGC
-XX:+UseShenandoahGC
-XX:+UseParallelGC
-XX:+UseSerialGC

# G1GC Tuning
-XX:MaxGCPauseMillis=<ms>
-XX:G1HeapRegionSize=<size>
-XX:InitiatingHeapOccupancyPercent=<percent>

# GC Threads
-XX:ParallelGCThreads=<n>
-XX:ConcGCThreads=<n>
```

**Diagnostics:**

```bash
# Logging
-Xlog:gc*:file=gc.log:time,level,tags

# Heap Dump
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dump.hprof

# Native Memory Tracking
-XX:NativeMemoryTracking=summary

# Print flags
-XX:+PrintFlagsFinal
-XX:+PrintCommandLineFlags
```

---

## 9.2 Profiling Tools

### JProfiler

**Installation:**

```bash
# Download from https://www.ej-technologies.com/products/jprofiler/overview.html

# Attach to running JVM
java -agentpath:/path/to/jprofiler/bin/linux-x64/libjprofilerti.so=port=8849 \
     -jar app.jar

# Or use JProfiler GUI to attach
```

**Key Features:**

```
1. CPU Profiling
   - Hot spots (methods consuming most CPU)
   - Call tree
   - Method statistics

2. Memory Profiling
   - Live memory (current objects)
   - Heap walker
   - Allocation hot spots
   - GC activity

3. Thread Profiling
   - Thread states
   - Deadlock detection
   - Lock contention

4. Database Profiling
   - JDBC calls
   - Query performance
   - Connection pool monitoring
```

**Example - Finding CPU Hotspot:**

```java
// Profiler reveals this method consumes 60% CPU
public List<User> findUsers(String query) {
    return allUsers.stream()
        .filter(user -> user.getName().contains(query))  // Expensive!
        .collect(Collectors.toList());
}

// Optimization: Use index
public List<User> findUsersFast(String query) {
    return userIndex.get(query);  // Pre-built index
}
```

### VisualVM (Free)

```bash
# Start VisualVM
jvisualvm

# Attach to process
# - Select process from list
# - Monitor, Sampler, Profiler tabs

# Or start with monitoring
java -Dcom.sun.management.jmxremote \
     -Dcom.sun.management.jmxremote.port=9010 \
     -Dcom.sun.management.jmxremote.authenticate=false \
     -Dcom.sun.management.jmxremote.ssl=false \
     -jar app.jar
```

**VisualVM Features:**

```
1. Monitor Tab
   - Heap size, used heap
   - Threads, classes loaded
   - CPU usage

2. Sampler Tab
   - CPU sampling
   - Memory sampling

3. Profiler Tab
   - CPU profiling
   - Memory profiling

4. Threads Tab
   - Thread timeline
   - Thread dump
```

### Async-Profiler (Production-Safe)

```bash
# Download
wget https://github.com/async-profiler/async-profiler/releases/download/v2.9/async-profiler-2.9-linux-x64.tar.gz
tar -xzf async-profiler-2.9-linux-x64.tar.gz

# Profile running JVM
./profiler.sh -d 60 -f flamegraph.html <pid>

# Result: Flame graph showing CPU hotspots
```

**Flame Graph Interpretation:**

```
Width = Time spent
Height = Call stack depth

Example:
┌─────────────────────────────────────────┐  main()
│                                         │
│  ┌────────────┐      ┌──────────────┐  │
│  │ findUsers()│      │ saveUser()   │  │  <- Wide = CPU intensive
│  │            │      │              │  │
│  │  ┌──────┐  │      │  ┌────┐     │  │
│  │  │filter│  │      │  │save│     │  │  <- Narrow = Fast
│  └──┴──────┴──┘      └──┴────┴─────┘  │
└─────────────────────────────────────────┘
```

### Java Flight Recorder (JFR)

```bash
# Start JFR recording
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr \
     -jar app.jar

# Or attach to running process
jcmd <pid> JFR.start duration=60s filename=recording.jfr

# Stop recording
jcmd <pid> JFR.stop

# Analyze with JDK Mission Control
jmc
```

**JFR Events:**

```
- CPU Usage
- Memory Allocation
- GC Events
- Thread Activity
- I/O Operations
- Lock Contention
- Exception Statistics
```

---

## 9.3 Memory Optimization

### Finding Memory Leaks

**Heap Dump Analysis:**

```bash
# Capture heap dump
jmap -dump:live,format=b,file=heap.hprof <pid>

# Or automatically on OOM
java -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/var/log/heap.hprof \
     -jar app.jar

# Analyze with Eclipse MAT
# Download: https://eclipse.dev/mat/
```

**Common Leak Patterns:**

```java
// ❌ LEAK 1: Static collections
public class CacheManager {
    private static Map<String, User> cache = new HashMap<>();
    
    public void cacheUser(User user) {
        cache.put(user.getId(), user);  // Never removed!
    }
}

// ✅ FIX: Use bounded cache
public class CacheManager {
    private static Cache<String, User> cache = Caffeine.newBuilder()
        .maximumSize(10000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build();
}

// ❌ LEAK 2: Listeners not removed
public class UserService {
    private List<UserListener> listeners = new ArrayList<>();
    
    public void addListener(UserListener listener) {
        listeners.add(listener);  // Never removed!
    }
}

// ✅ FIX: Remove listeners
public void removeListener(UserListener listener) {
    listeners.remove(listener);
}

// ❌ LEAK 3: ThreadLocal not cleaned
public class RequestContext {
    private static ThreadLocal<User> currentUser = new ThreadLocal<>();
    
    public static void setUser(User user) {
        currentUser.set(user);  // Not cleaned!
    }
}

// ✅ FIX: Clean ThreadLocal
public static void clear() {
    currentUser.remove();
}

// Use try-finally
try {
    RequestContext.setUser(user);
    // Process request
} finally {
    RequestContext.clear();
}
```

### Allocation Profiling

```java
// Find allocation hotspots
// Use JProfiler or async-profiler --event alloc

// Example finding:
public List<String> processData(List<Data> data) {
    List<String> result = new ArrayList<>();
    for (Data d : data) {
        result.add(d.toString());  // Allocates many strings!
    }
    return result;
}

// Optimization: Pre-size list
public List<String> processDataOptimized(List<Data> data) {
    List<String> result = new ArrayList<>(data.size());  // Pre-sized
    for (Data d : data) {
        result.add(d.toString());
    }
    return result;
}
```

### Object Size Analysis

```java
// Use JOL (Java Object Layout)
```

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.17</version>
</dependency>
```

```java
import org.openjdk.jol.info.ClassLayout;
import org.openjdk.jol.info.GraphLayout;

public class ObjectSizeAnalysis {
    
    public static void main(String[] args) {
        User user = new User("John", "Doe", "john@example.com");
        
        // Print object layout
        System.out.println(ClassLayout.parseInstance(user).toPrintable());
        
        // Total size including referenced objects
        System.out.println(GraphLayout.parseInstance(user).totalSize());
    }
}

// Output:
// com.example.User object internals:
//  OFFSET  SIZE     TYPE DESCRIPTION
//       0     4          (object header)
//       4     4          (object header)
//       8     4          (object header)
//      12     4   String User.firstName
//      16     4   String User.lastName
//      20     4   String User.email
//      24     0          (loss due to the next object alignment)
// Instance size: 24 bytes
```

**Memory-Efficient Data Structures:**

```java
// ❌ Wasteful
List<Integer> list = new ArrayList<>();  // Default capacity: 10
list.add(1);  // Only 1 element!

// ✅ Efficient
List<Integer> list = new ArrayList<>(1);  // Exact capacity

// ❌ Many small objects
class Point {
    Integer x;  // 16 bytes overhead per Integer!
    Integer y;
}

// ✅ Primitive types
class Point {
    int x;  // 4 bytes
    int y;  // 4 bytes
}

// ❌ String concatenation in loop
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // Creates 1000 intermediate strings!
}

// ✅ StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

---

## 9.4 Garbage Collection Deep Dive

### GC Logging Analysis

```bash
# Enable detailed GC logging
java -Xlog:gc*=debug:file=gc.log:time,uptime,level,tags \
     -jar app.jar

# GC log format
[2024-01-23T10:30:45.123+0000][0.456s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 24M->6M(256M) 2.345ms
```

**Key Metrics:**

```
[time][uptime][level][gc] Event Details Heap_Before->Heap_After(Total_Heap) Pause_Time

Example:
24M->6M(256M) 2.345ms
- Before GC: 24MB used
- After GC: 6MB used
- Total heap: 256MB
- Pause time: 2.345ms
```

### GC Tuning Goals

```
Two competing goals:

1. Throughput (maximize application time)
   - Minimize GC time
   - Use: Parallel GC or G1GC

2. Latency (minimize pause times)
   - Minimize pause duration
   - Use: ZGC or Shenandoah

Choose based on workload:
- Batch processing → Throughput
- Web services → Latency
```

### G1GC Tuning Example

**Problem: Long GC pauses**

```bash
# Before tuning
[gc] GC(10) Pause Young (Normal) 2048M->1024M(4096M) 500ms  # Too long!

# Diagnosis:
# - Young generation too large
# - MaxGCPauseMillis too aggressive
```

**Solution:**

```bash
java -XX:+UseG1GC \
     -Xms4g -Xmx4g \
     -XX:MaxGCPauseMillis=100 \        # Was 50ms (too aggressive)
     -XX:G1HeapRegionSize=8m \         # Smaller regions
     -XX:InitiatingHeapOccupancyPercent=40 \  # Start marking earlier
     -jar app.jar

# After tuning
[gc] GC(10) Pause Young (Normal) 2048M->1024M(4096M) 95ms  # Better!
```

### Analyzing GC Logs with GCViewer

```bash
# Download GCViewer
wget https://github.com/chewiebug/GCViewer/releases/download/1.37/gcviewer-1.37.jar

# Analyze logs
java -jar gcviewer-1.37.jar gc.log

# Metrics shown:
# - Pause time distribution
# - Throughput percentage
# - Heap usage over time
# - GC frequency
```

---

## 9.5 Benchmarking with JMH

JMH (Java Microbenchmark Harness) provides accurate performance measurements.

### Setup

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
    <scope>provided</scope>
</dependency>
```

### Basic Benchmark

```java
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Thread)
@Fork(value = 1, warmups = 1)
@Warmup(iterations = 3, time = 1)
@Measurement(iterations = 5, time = 1)
public class StringConcatenationBenchmark {
    
    @Param({"10", "100", "1000"})
    private int iterations;
    
    @Benchmark
    public String stringConcat() {
        String result = "";
        for (int i = 0; i < iterations; i++) {
            result += i;
        }
        return result;
    }
    
    @Benchmark
    public String stringBuilder() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < iterations; i++) {
            sb.append(i);
        }
        return sb.toString();
    }
    
    @Benchmark
    public String stringBuilderWithCapacity() {
        StringBuilder sb = new StringBuilder(iterations * 2);
        for (int i = 0; i < iterations; i++) {
            sb.append(i);
        }
        return sb.toString();
    }
}
```

**Run Benchmark:**

```bash
# Build
mvn clean package

# Run
java -jar target/benchmarks.jar

# Results:
Benchmark                                      (iterations)  Mode  Cnt     Score    Error  Units
StringConcatenationBenchmark.stringConcat              10  avgt    5   450.2 ±   12.3  ns/op
StringConcatenationBenchmark.stringConcat             100  avgt    5  8234.1 ±  234.5  ns/op
StringConcatenationBenchmark.stringConcat            1000  avgt    5 98765.4 ± 1234.6  ns/op

StringConcatenationBenchmark.stringBuilder             10  avgt    5    85.3 ±    3.2  ns/op
StringConcatenationBenchmark.stringBuilder            100  avgt    5   523.4 ±   15.6  ns/op
StringConcatenationBenchmark.stringBuilder           1000  avgt    5  3456.7 ±   89.1  ns/op

StringConcatenationBenchmark.stringBuilderWithCapacity 10  avgt    5    72.1 ±    2.8  ns/op
StringConcatenationBenchmark.stringBuilderWithCapacity100  avgt    5   412.3 ±   11.2  ns/op
StringConcatenationBenchmark.stringBuilderWithCapacity1000 avgt    5  2987.5 ±   67.3  ns/op
```

### Benchmark Modes

```java
// Throughput: operations per second
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
public void throughputBenchmark() {
    // ...
}

// Average time: average execution time
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public void averageTimeBenchmark() {
    // ...
}

// Sample time: percentile distribution
@BenchmarkMode(Mode.SampleTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public void sampleTimeBenchmark() {
    // ...
}

// Single shot: cold startup time
@BenchmarkMode(Mode.SingleShotTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public void singleShotBenchmark() {
    // ...
}

// All modes
@BenchmarkMode(Mode.All)
public void allModesBenchmark() {
    // ...
}
```

### State Management

```java
@State(Scope.Thread)  // One instance per thread
public class ThreadState {
    private List<String> list;
    
    @Setup
    public void setup() {
        list = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            list.add("Item" + i);
        }
    }
    
    @TearDown
    public void tearDown() {
        list.clear();
    }
}

@State(Scope.Benchmark)  // Shared across all threads
public class BenchmarkState {
    private static final Random random = new Random();
}

@State(Scope.Group)  // Shared within thread group
public class GroupState {
    // ...
}
```

### Avoiding JIT Optimizations

```java
@Benchmark
public void doNotOptimizeAway(Blackhole blackhole) {
    int result = compute();
    blackhole.consume(result);  // Prevent dead code elimination
}

@Benchmark
@CompilerControl(CompilerControl.Mode.DONT_INLINE)
public int preventInlining() {
    return compute();  // Force method call
}
```

### Collection Benchmark Example

```java
@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class CollectionBenchmark {
    
    @Param({"1000", "10000", "100000"})
    private int size;
    
    private List<Integer> arrayList;
    private List<Integer> linkedList;
    private Set<Integer> hashSet;
    private Set<Integer> treeSet;
    
    @Setup
    public void setup() {
        arrayList = new ArrayList<>();
        linkedList = new LinkedList<>();
        hashSet = new HashSet<>();
        treeSet = new TreeSet<>();
        
        for (int i = 0; i < size; i++) {
            arrayList.add(i);
            linkedList.add(i);
            hashSet.add(i);
            treeSet.add(i);
        }
    }
    
    @Benchmark
    public boolean arrayListContains() {
        return arrayList.contains(size / 2);
    }
    
    @Benchmark
    public boolean linkedListContains() {
        return linkedList.contains(size / 2);
    }
    
    @Benchmark
    public boolean hashSetContains() {
        return hashSet.contains(size / 2);
    }
    
    @Benchmark
    public boolean treeSetContains() {
        return treeSet.contains(size / 2);
    }
}

// Results show:
// HashSet: O(1) - constant time
// TreeSet: O(log n) - logarithmic time
// ArrayList: O(n) - linear time
// LinkedList: O(n) - linear time
```

---

## 9.6 Database Optimization

### Query Optimization

**N+1 Query Problem:**

```java
// ❌ N+1 queries: 1 + N (one per user)
@Service
public class OrderService {
    
    public List<OrderDTO> getOrders() {
        List<Order> orders = orderRepository.findAll();  // Query 1
        
        return orders.stream()
            .map(order -> {
                User user = userRepository.findById(order.getUserId()).get();  // Query 2, 3, 4...
                return new OrderDTO(order, user);
            })
            .collect(Collectors.toList());
    }
}

// ✅ Solution: JOIN FETCH
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @Query("SELECT o FROM Order o JOIN FETCH o.user")
    List<Order> findAllWithUser();
}

@Service
public class OrderService {
    
    public List<OrderDTO> getOrders() {
        List<Order> orders = orderRepository.findAllWithUser();  // Single query
        return orders.stream()
            .map(order -> new OrderDTO(order, order.getUser()))
            .collect(Collectors.toList());
    }
}
```

**Pagination:**

```java
// ❌ Load all records
public List<User> getAllUsers() {
    return userRepository.findAll();  // Loads millions of rows!
}

// ✅ Use pagination
public Page<User> getUsers(int page, int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    return userRepository.findAll(pageable);
}
```

**Projection:**

```java
// ❌ Load entire entity
@Query("SELECT u FROM User u")
List<User> findAllUsers();  // Loads all fields

// ✅ Load only needed fields
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

@Query("SELECT u.id as id, u.name as name, u.email as email FROM User u")
List<UserSummary> findAllUserSummaries();
```

### Indexing

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email"),
    @Index(name = "idx_status", columnList = "status"),
    @Index(name = "idx_created_at", columnList = "created_at"),
    @Index(name = "idx_composite", columnList = "status, created_at")
})
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String status;
    
    @Column(nullable = false)
    private LocalDateTime createdAt;
}
```

**When to Index:**
- ✅ Foreign keys
- ✅ Columns in WHERE clauses
- ✅ Columns in ORDER BY
- ✅ Columns in JOIN conditions
- ❌ Small tables (<1000 rows)
- ❌ Frequently updated columns
- ❌ Columns with low cardinality (few distinct values)

### Connection Pooling

```yaml
# HikariCP configuration (default in Spring Boot)
spring:
  datasource:
    hikari:
      minimum-idle: 10              # Minimum connections
      maximum-pool-size: 50         # Maximum connections
      idle-timeout: 600000          # 10 minutes
      max-lifetime: 1800000         # 30 minutes
      connection-timeout: 30000     # 30 seconds
      pool-name: MyHikariPool
      
      # Performance
      auto-commit: false
      connection-test-query: SELECT 1
      
      # Leak detection
      leak-detection-threshold: 60000  # 60 seconds
```

**Pool Size Calculation:**

```
Formula: pool_size = Tn × (Cm - 1) + 1

Where:
- Tn = Number of threads
- Cm = Average concurrent database requests per thread

Example:
- 100 threads
- 2 concurrent DB requests per thread on average
- pool_size = 100 × (2 - 1) + 1 = 101

Rule of thumb: 2 × CPU cores to 10 × CPU cores
For 8 cores: 16-80 connections
```

### Query Caching

```java
@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
    
    @CachePut(value = "users", key = "#user.id")
    public User update(User user) {
        return userRepository.save(user);
    }
    
    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        userRepository.deleteById(id);
    }
    
    @Cacheable(value = "userList")
    public List<User> findAll() {
        return userRepository.findAll();
    }
}
```

### Batch Operations

```java
// ❌ Individual inserts
for (User user : users) {
    userRepository.save(user);  // One query per user
}

// ✅ Batch insert
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Modifying
    @Query(value = "INSERT INTO users (name, email) VALUES (:#{#users})", nativeQuery = true)
    void batchInsert(@Param("users") List<User> users);
}

// Or use JDBC batch
@Service
public class UserService {
    
    private final JdbcTemplate jdbcTemplate;
    
    public void batchInsert(List<User> users) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        
        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                User user = users.get(i);
                ps.setString(1, user.getName());
                ps.setString(2, user.getEmail());
            }
            
            @Override
            public int getBatchSize() {
                return users.size();
            }
        });
    }
}
```

---

## 9.7 Application Performance Best Practices

### CPU Optimization

**Avoid Unnecessary Object Creation:**

```java
// ❌ Creates many objects
public String format(List<User> users) {
    String result = "";
    for (User user : users) {
        result += user.getName() + ",";  // New string each iteration
    }
    return result;
}

// ✅ Reuse StringBuilder
public String format(List<User> users) {
    StringBuilder sb = new StringBuilder(users.size() * 20);
    for (User user : users) {
        sb.append(user.getName()).append(",");
    }
    return sb.toString();
}
```

**Lazy Initialization:**

```java
// ❌ Eager initialization
public class UserService {
    private final ExpensiveResource resource = new ExpensiveResource();  // Always created
}

// ✅ Lazy initialization
public class UserService {
    private ExpensiveResource resource;
    
    private ExpensiveResource getResource() {
        if (resource == null) {
            resource = new ExpensiveResource();  // Created when needed
        }
        return resource;
    }
}
```

**Parallel Processing:**

```java
// ❌ Sequential processing
public List<Result> processUsers(List<User> users) {
    return users.stream()
        .map(this::processUser)  // Sequential
        .collect(Collectors.toList());
}

// ✅ Parallel processing
public List<Result> processUsersParallel(List<User> users) {
    return users.parallelStream()
        .map(this::processUser)  // Parallel
        .collect(Collectors.toList());
}

// Note: Use parallel streams when:
// - Operation is CPU-intensive
// - List is large (>10,000 items)
// - No shared mutable state
```

### I/O Optimization

**Buffering:**

```java
// ❌ Unbuffered I/O
try (FileReader reader = new FileReader("file.txt")) {
    int c;
    while ((c = reader.read()) != -1) {  // One syscall per character!
        process(c);
    }
}

// ✅ Buffered I/O
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {  // Buffers internally
        process(line);
    }
}
```

**NIO for Large Files:**

```java
// Read large file efficiently
public List<String> readLargeFile(Path path) throws IOException {
    return Files.lines(path)
        .parallel()
        .filter(line -> !line.isEmpty())
        .collect(Collectors.toList());
}
```

### Async Processing

```java
@Service
public class NotificationService {
    
    @Async
    public CompletableFuture<Void> sendEmail(String to, String message) {
        // Long-running operation
        emailClient.send(to, message);
        return CompletableFuture.completedFuture(null);
    }
}

@Service
public class OrderService {
    
    private final NotificationService notificationService;
    
    public Order createOrder(CreateOrderRequest request) {
        Order order = orderRepository.save(new Order(request));
        
        // Send email asynchronously (don't wait)
        notificationService.sendEmail(
            request.getEmail(),
            "Order created: " + order.getId()
        );
        
        return order;  // Return immediately
    }
}
```

---

## Summary and Key Takeaways

### JVM Tuning Mastery

**Memory Configuration:**
* **Heap sizing**: Xms = Xmx for predictable performance
* **Young generation**: 25-40% of total heap
* **Metaspace**: 256MB-512MB for most apps

**GC Selection:**
* **G1GC**: General purpose (default)
* **ZGC**: Ultra-low latency (<1ms pauses)
* **Parallel GC**: Maximum throughput
* **Shenandoah**: Low latency alternative to ZGC

**Essential Flags:**
```bash
-Xms4g -Xmx4g              # Heap
-XX:+UseG1GC               # GC algorithm
-XX:MaxGCPauseMillis=200   # Target pause
-XX:+HeapDumpOnOutOfMemoryError  # OOM analysis
-Xlog:gc*:file=gc.log      # GC logging
```

### Profiling Expertise

**Tools:**
* **JProfiler**: Commercial, comprehensive
* **VisualVM**: Free, good for development
* **async-profiler**: Production-safe, flame graphs
* **Java Flight Recorder**: Low-overhead, comprehensive

**What to Profile:**
* **CPU**: Find hot methods
* **Memory**: Find allocations, leaks
* **Threads**: Find deadlocks, contention
* **I/O**: Find slow queries, file access

### Memory Optimization

**Common Leaks:**
* Static collections never cleared
* Listeners not removed
* ThreadLocal not cleaned
* Unclosed resources (streams, connections)

**Prevention:**
* Use try-with-resources
* Clear ThreadLocal in finally blocks
* Use weak references for caches
* Remove listeners when done

**Optimization:**
* Pre-size collections
* Use primitive types
* Avoid unnecessary object creation
* Pool expensive objects

### GC Tuning

**Throughput Optimization:**
```bash
-XX:+UseParallelGC
-XX:ParallelGCThreads=8
-XX:MaxGCPauseMillis=1000  # Allow longer pauses
```

**Latency Optimization:**
```bash
-XX:+UseZGC
-XX:MaxGCPauseMillis=10   # Target <10ms
```

**Monitoring:**
* GC log analysis with GCViewer
* Monitor pause times
* Track heap usage trends
* Alert on long pauses

### Benchmarking Best Practices

**JMH Rules:**
* Warm up JVM (3+ iterations)
* Multiple measurements (5+ iterations)
* Use @Blackhole to prevent optimizations
* Test realistic workloads
* Measure what matters (throughput vs latency)

**Avoid:**
* Microbenchmarks without warmup
* Single measurements
* Unrealistic test data
* Measuring irrelevant metrics

### Database Performance

**Query Optimization:**
* Eliminate N+1 queries (use JOIN FETCH)
* Use pagination for large result sets
* Use projections (load only needed fields)
* Add indexes on WHERE/ORDER BY columns

**Connection Pooling:**
* Pool size: 2× to 10× CPU cores
* Monitor connection usage
* Set leak detection threshold
* Configure timeouts

**Caching:**
* Cache frequent queries
* Set appropriate TTL
* Invalidate on updates
* Monitor hit rates

### Production Checklist

**JVM Configuration:**
- [ ] Heap size configured (Xms = Xmx)
- [ ] GC algorithm chosen
- [ ] GC logging enabled
- [ ] Heap dump on OOM
- [ ] JMX monitoring enabled

**Profiling:**
- [ ] CPU profiling in pre-prod
- [ ] Memory profiling in pre-prod
- [ ] Baseline metrics captured
- [ ] Profiling tools available

**Database:**
- [ ] Indexes on key columns
- [ ] N+1 queries eliminated
- [ ] Connection pool configured
- [ ] Slow query logging enabled
- [ ] Query caching configured

**Monitoring:**
- [ ] CPU usage monitored
- [ ] Memory usage monitored
- [ ] GC metrics tracked
- [ ] Response times tracked
- [ ] Error rates tracked

### Frequently Asked Questions

**Q1: How do I choose GC algorithm?**

**A:**

**Use G1GC when:**
* General-purpose application
* Heap: 4GB-100GB
* Pause time: <200ms acceptable
* **Default choice for most apps**

**Use ZGC when:**
* Ultra-low latency required
* Heap: >8GB
* Pause time: <10ms required
* Java 15+ available

**Use Parallel GC when:**
* Batch processing
* Throughput > latency
* Long pauses acceptable

**Use Shenandoah when:**
* Low latency required
* ZGC not available
* Any heap size

**Comparison:**

| Algorithm  | Pause Time | Throughput | Heap Size  |
|-----------|-----------|-----------|-----------|
| G1GC      | 10-200ms  | Good      | Any       |
| ZGC       | <10ms     | Good      | Large     |
| Parallel  | >200ms    | Excellent | Any       |
| Shenandoah| <10ms     | Good      | Any       |

---

**Q2: When should I optimize performance?**

**A:**

**Optimize when:**
* Specific performance requirement not met
* User complaints about slowness
* Resource costs too high
* Scaling issues

**Don't optimize when:**
* No measurable problem
* Premature (before profiling)
* Minor gain for major complexity

**Process:**
1. **Measure**: Profile to find bottleneck
2. **Set goal**: Define target metric
3. **Optimize**: Fix the bottleneck
4. **Measure again**: Verify improvement
5. **Repeat**: Move to next bottleneck

**Example:**
```
1. Profile: findUsers() uses 60% CPU
2. Goal: Reduce to <10% CPU
3. Optimize: Add index on user.name
4. Measure: Now uses 5% CPU ✓
5. Next: Focus on next bottleneck
```

---

**Q3: How do I find memory leaks?**

**A:**

**Step-by-step:**

1. **Capture heap dump**:
```bash
jmap -dump:live,format=b,file=heap.hprof <pid>
```

2. **Analyze with Eclipse MAT**:
```
- Open heap.hprof in MAT
- Run "Leak Suspects" report
- Look for large objects
- Check retained size
```

3. **Common patterns**:
```java
// Static collections
private static Map<String, User> cache = new HashMap<>();

// ThreadLocal not cleared
private static ThreadLocal<User> context = new ThreadLocal<>();

// Listeners not removed
private List<Listener> listeners = new ArrayList<>();
```

4. **Fix and verify**:
```bash
# Monitor heap over time
jstat -gcutil <pid> 1000

# Heap should stabilize, not grow indefinitely
```

### Interview Questions

**Question 1: Explain the difference between throughput and latency GC tuning.**

**Difficulty:** Senior

**Topics:** JVM, GC, Performance

**Answer:**

**Throughput Tuning**: Maximize application execution time

**Goal**: Minimize % of time spent in GC
```
Example:
- Total time: 100 seconds
- GC time: 5 seconds
- Throughput: 95%
```

**Configuration**:
```bash
-XX:+UseParallelGC           # Maximize throughput
-XX:GCTimeRatio=99           # Target 1% GC time
-XX:MaxGCPauseMillis=1000    # Allow long pauses
```

**Trade-off**: Long pauses acceptable

---

**Latency Tuning**: Minimize GC pause times

**Goal**: Minimize maximum pause duration
```
Example:
- 99th percentile pause: <10ms
- Max pause: <50ms
```

**Configuration**:
```bash
-XX:+UseZGC                  # Ultra-low latency
-XX:MaxGCPauseMillis=10      # Target <10ms
```

**Trade-off**: Slightly lower throughput

---

**When to use:**
* **Throughput**: Batch processing, analytics
* **Latency**: Web services, real-time systems

---

**Question 2: How do you benchmark Java code accurately?**

**Difficulty:** Mid-Level

**Topics:** JMH, Benchmarking

**Answer:**

**Use JMH** (Java Microbenchmark Harness):

**Key principles:**

1. **Warm up JVM**:
```java
@Warmup(iterations = 3, time = 1)
```

2. **Multiple measurements**:
```java
@Measurement(iterations = 5, time = 1)
```

3. **Prevent optimizations**:
```java
@Benchmark
public void test(Blackhole blackhole) {
    int result = compute();
    blackhole.consume(result);  // Prevent DCE
}
```

4. **Realistic workload**:
```java
@State(Scope.Thread)
public class BenchmarkState {
    @Param({"100", "1000", "10000"})
    private int size;
    
    @Setup
    public void setup() {
        // Initialize realistic data
    }
}
```

**Common mistakes:**
* No warmup → Inaccurate results
* Single measurement → High variance
* Unrealistic data → Wrong conclusions
* Measuring wrong thing → Useless benchmark

---

**End of Part 9: Performance & Optimization**

### Congratulations!

You've mastered:
* ✅ JVM tuning (heap, GC, flags)
* ✅ Profiling tools (JProfiler, VisualVM, async-profiler)
* ✅ Memory optimization (leak detection, efficient structures)
* ✅ GC deep dive (algorithms, tuning, analysis)
* ✅ JMH benchmarking (accurate measurements)
* ✅ Database optimization (queries, indexing, pooling)
* ✅ Application performance (CPU, I/O, async)

**Complete Series: 9 parts complete! 🎉**

### References

* [JVM Tuning Guide](https://docs.oracle.com/en/java/javase/21/gctuning/)
* [JMH Samples](https://github.com/openjdk/jmh)
* [Eclipse MAT](https://eclipse.dev/mat/)
* [async-profiler](https://github.com/async-profiler/async-profiler)
* [GCViewer](https://github.com/chewiebug/GCViewer)
* [Java Performance: The Definitive Guide by Scott Oaks](https://www.oreilly.com/library/view/java-performance-the/9781449363512/)

---