---
title: "Java Mastery - Part 2: JVM Internals & Memory Management"
date: 2025-07-03 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, JVM, Performance, Garbage-collection, Memory-management, Performance, Class-loading, JIT-compilation]
---

## Introduction

Welcome to Part 2 of the Complete Java Mastery series. In Part 1, we covered Java fundamentals and object-oriented programming. Now we dive deep into the heart of Java: the **Java Virtual Machine (JVM)**.

Understanding JVM internals is what separates junior developers from senior engineers. When your application experiences memory leaks, performance degradation, or mysterious crashes in production, knowledge of JVM internals becomes critical. This isn't just theoretical knowledge—it's practical expertise that you'll use to:

* Diagnose and fix memory leaks
* Optimize application performance
* Tune garbage collection for your workload
* Understand class loading issues
* Debug production problems
* Make informed architecture decisions

### What You'll Learn in Part 2

* **JVM Architecture**: Components, execution engine, and how Java code actually runs
* **Class Loading**: The complete lifecycle of a class from .java to execution
* **Memory Management**: Heap structure, stack organization, and memory allocation
* **Garbage Collection**: Algorithms, tuning, and choosing the right GC for your application
* **Performance Tuning**: JVM flags, profiling tools, and optimization techniques

This knowledge is essential for:
* **Development**: Writing memory-efficient code
* **Testing**: Understanding performance characteristics
* **Deployment**: Configuring JVM for production
* **Maintenance**: Monitoring and troubleshooting production issues

---

## 2.1 JVM Architecture

The Java Virtual Machine is an abstract computing machine that provides a runtime environment for executing Java bytecode. Understanding its architecture is fundamental to understanding how Java works.

### Overview: From Source to Execution

```
Development Time:
┌──────────────┐    javac     ┌──────────────┐
│  .java file  │ ───────────> │  .class file │
│ (Source code)│              │  (Bytecode)  │
└──────────────┘              └──────────────┘

Runtime:
┌──────────────┐              ┌──────────────┐
│  .class file │ ───────────> │     JVM      │
│  (Bytecode)  │   Loading    │  Execution   │
└──────────────┘              └──────────────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │   Platform   │
                              │ (OS + Hardware)
                              └──────────────┘
```

The JVM acts as an intermediary between bytecode and the underlying platform, providing the "Write Once, Run Anywhere" capability.

### JVM Architecture Components

```
┌─────────────────────────────────────────────────────────┐
│                     JVM Architecture                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │         Class Loader Subsystem                  │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐     │    │
│  │  │ Loading  │→ │ Linking  │→ │Initialize│     │    │
│  │  └──────────┘  └──────────┘  └──────────┘     │    │
│  └────────────────────────────────────────────────┘    │
│                        ↓                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │         Runtime Data Areas                      │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐     │    │
│  │  │  Method  │  │   Heap   │  │  Stacks  │     │    │
│  │  │   Area   │  │          │  │  (per    │     │    │
│  │  │(Metaspace)  │          │  │  thread) │     │    │
│  │  └──────────┘  └──────────┘  └──────────┘     │    │
│  │  ┌──────────┐  ┌──────────┐                   │    │
│  │  │    PC    │  │  Native  │                   │    │
│  │  │ Register │  │  Method  │                   │    │
│  │  │          │  │  Stacks  │                   │    │
│  │  └──────────┘  └──────────┘                   │    │
│  └────────────────────────────────────────────────┘    │
│                        ↓                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │         Execution Engine                        │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐     │    │
│  │  │Interpreter  │  JIT     │  │ Garbage  │     │    │
│  │  │          │  │ Compiler │  │Collector │     │    │
│  │  └──────────┘  └──────────┘  └──────────┘     │    │
│  └────────────────────────────────────────────────┘    │
│                        ↓                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │    Native Method Interface (JNI)               │    │
│  └────────────────────────────────────────────────┘    │
│                        ↓                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │    Native Method Libraries                      │    │
│  └────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

Let's explore each component in detail.

### Class Loader Subsystem

The **Class Loader Subsystem** is responsible for loading class files into memory. This is the first stage of the JVM runtime.

#### Three Phases of Class Loading

**1. Loading**

Finds and imports the binary data for a class.

```java
// When you write:
MyClass obj = new MyClass();

// JVM performs:
// 1. Check if MyClass is already loaded
// 2. If not, locate MyClass.class file
// 3. Read bytecode from .class file
// 4. Create java.lang.Class object in memory
```

**2. Linking**

Combines the binary type data into the JVM runtime state.

**Linking has three sub-phases:**

a) **Verification**: Ensures class file is valid and doesn't violate security constraints
   ```
   - Checks bytecode validity
   - Verifies type safety
   - Ensures no illegal operations
   - Validates symbolic references
   ```

b) **Preparation**: Allocates memory for static variables and initializes to default values
   ```java
   public class Example {
       private static int count;      // Set to 0
       private static String name;    // Set to null
       private static final int MAX = 100;  // Not set yet
   }
   ```

c) **Resolution**: Converts symbolic references to direct references
   ```java
   // Before resolution (symbolic):
   MyClass.staticMethod();  // Reference by name
   
   // After resolution (direct):
   // Direct memory address of staticMethod()
   ```

**3. Initialization**

Executes static initializers and initializes static fields.

```java
public class Example {
    private static int count = 10;  // Now set to 10
    
    static {
        System.out.println("Static block executed");
        count = 20;  // Final value: 20
    }
}
```

**Initialization Order:**

```java
public class Parent {
    static {
        System.out.println("1. Parent static block");
    }
}

public class Child extends Parent {
    static {
        System.out.println("2. Child static block");
    }
    
    public static void main(String[] args) {
        System.out.println("3. Main method");
    }
}

// Output:
// 1. Parent static block
// 2. Child static block
// 3. Main method
```

#### ClassLoader Hierarchy

JVM uses a **delegation model** with three built-in class loaders:

```
┌─────────────────────────────┐
│  Bootstrap ClassLoader      │  (C/C++, not Java)
│  - Loads rt.jar             │  Loads: java.lang.*, java.util.*
│  - JRE/lib directory        │
└──────────────┬──────────────┘
               │ Parent
               ▼
┌─────────────────────────────┐
│  Platform/Extension         │  (Java)
│  ClassLoader (Java 9+)      │  Loads: java.sql.*, javax.*
│  - JRE/lib/ext              │
└──────────────┬──────────────┘
               │ Parent
               ▼
┌─────────────────────────────┐
│  Application/System         │  (Java)
│  ClassLoader                │  Loads: Application classes
│  - CLASSPATH               │  (your code)
└─────────────────────────────┘
```

**Delegation Model:**

When a class needs to be loaded:

```
1. Check if already loaded in current classloader
2. If not, delegate to parent classloader
3. If parent can't load, attempt to load in current classloader
4. If still can't load, throw ClassNotFoundException
```

**Example:**

```java
// Loading MyApp.class

Application ClassLoader:
  - "Is MyApp loaded? No"
  - "Delegate to parent (Platform ClassLoader)"
  
Platform ClassLoader:
  - "Is MyApp loaded? No"
  - "Delegate to parent (Bootstrap ClassLoader)"
  
Bootstrap ClassLoader:
  - "Is MyApp in rt.jar? No"
  - "Return: Cannot load"
  
Platform ClassLoader:
  - "Can I find MyApp in extensions? No"
  - "Return: Cannot load"
  
Application ClassLoader:
  - "Can I find MyApp in CLASSPATH? Yes!"
  - "Load MyApp.class"
  - "Return: MyApp class"
```

**Why This Design?**

1. **Security**: Core classes loaded by Bootstrap can't be replaced by malicious code
2. **Uniqueness**: Each class loaded only once by hierarchy
3. **Visibility**: Child can see parent classes, not vice versa

**Custom ClassLoader:**

```java
public class CustomClassLoader extends ClassLoader {
    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        // Load class from custom source (database, network, etc.)
        byte[] classData = loadClassData(name);
        return defineClass(name, classData, 0, classData.length);
    }
    
    private byte[] loadClassData(String className) {
        // Custom logic to load bytecode
        // e.g., from encrypted file, database, network
        return new byte[0];  // Placeholder
    }
}

// Usage
CustomClassLoader loader = new CustomClassLoader();
Class<?> clazz = loader.loadClass("com.example.MyClass");
Object instance = clazz.getDeclaredConstructor().newInstance();
```

**Use Cases for Custom ClassLoaders:**
* Loading classes from non-standard sources (databases, networks)
* Hot deployment/reloading (application servers)
* Class encryption/obfuscation
* Plugin architectures
* OSGi frameworks

#### Class Loading Methods

```java
// Method 1: Class.forName() - loads and initializes
Class<?> clazz = Class.forName("com.example.MyClass");
// Static blocks executed

// Method 2: ClassLoader.loadClass() - loads but doesn't initialize
ClassLoader loader = MyClass.class.getClassLoader();
Class<?> clazz2 = loader.loadClass("com.example.MyClass");
// Static blocks NOT executed until first use

// When is a class initialized?
// 1. Creating an instance: new MyClass()
// 2. Accessing static field: MyClass.staticField
// 3. Calling static method: MyClass.staticMethod()
// 4. Reflection with forceInit: Class.forName("MyClass", true, loader)
// 5. Initializing subclass triggers parent initialization
```

### Runtime Data Areas

JVM memory is divided into several runtime data areas. Understanding these is crucial for performance tuning and debugging.

#### 1. Method Area (Metaspace in Java 8+)

**What it stores:**
* Class metadata (structure, fields, methods)
* Static variables
* Constant pool (string literals, numeric constants)
* Method bytecode

**Before Java 8 (PermGen):**
```
┌────────────────────────┐
│   PermGen Space        │  Fixed size, part of heap
│  - Class metadata      │  Default: 64MB
│  - Static variables    │  Could cause OutOfMemoryError
│  - Interned strings    │
└────────────────────────┘
```

**Java 8+ (Metaspace):**
```
┌────────────────────────┐
│   Metaspace            │  Dynamic size, native memory
│  - Class metadata      │  Default: unlimited (constrained by OS)
│  - Static variables    │  Auto-grows as needed
└────────────────────────┘

┌────────────────────────┐
│   Heap                 │  Interned strings moved here
│  - String pool         │
│  - Objects             │
└────────────────────────┘
```

**Why the Change?**

```java
// Problem with PermGen:
// Loading many classes (e.g., application server with many apps)
for (int i = 0; i < 100000; i++) {
    CustomClassLoader loader = new CustomClassLoader();
    loader.loadClass("MyClass" + i);
}
// OutOfMemoryError: PermGen space

// With Metaspace:
// Automatically expands (limited by native memory)
// Fewer OutOfMemoryError from class loading
```

**Configuring Metaspace:**

```bash
# Set initial metaspace size
-XX:MetaspaceSize=128m

# Set maximum metaspace size
-XX:MaxMetaspaceSize=512m

# If not set, MaxMetaspaceSize is unlimited (constrained by OS)
```

**Monitoring Metaspace:**

```java
// Using JMX
MemoryPoolMXBean metaspacePool = null;
for (MemoryPoolMXBean pool : ManagementFactory.getMemoryPoolMXBeans()) {
    if ("Metaspace".equals(pool.getName())) {
        metaspacePool = pool;
        break;
    }
}

if (metaspacePool != null) {
    MemoryUsage usage = metaspacePool.getUsage();
    System.out.println("Metaspace used: " + usage.getUsed() / 1024 / 1024 + " MB");
    System.out.println("Metaspace max: " + usage.getMax() / 1024 / 1024 + " MB");
}
```

#### 2. Heap

**The heap** is the runtime data area where objects are allocated.

**Structure (Generational Hypothesis):**

```
┌────────────────────────────────────────────────────────┐
│                     Java Heap                           │
├────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────┐  ┌─────────────────┐ │
│  │    Young Generation          │  │      Old        │ │
│  │                              │  │   Generation    │ │
│  │  ┌─────┐  ┌────┐  ┌────┐   │  │  (Tenured)      │ │
│  │  │Eden │  │ S0 │  │ S1 │   │  │                 │ │
│  │  │     │  │    │  │    │   │  │  Long-lived     │ │
│  │  │     │  │    │  │    │   │  │  objects        │ │
│  │  └─────┘  └────┘  └────┘   │  │                 │ │
│  │                              │  │                 │ │
│  │  New objects                 │  │                 │ │
│  │  Most objects die young      │  │                 │ │
│  └─────────────────────────────┘  └─────────────────┘ │
│                                                          │
└────────────────────────────────────────────────────────┘

Typical ratio: Young:Old = 1:2
Eden:Survivor = 8:1:1 (Eden:S0:S1)
```

**Object Lifecycle:**

```java
// 1. Object created in Eden
MyObject obj = new MyObject();  // Allocated in Eden space

// 2. Eden fills up → Minor GC occurs
//    - Live objects copied to S0 (Survivor 0)
//    - Dead objects collected
//    - Age counter incremented for survivors

// 3. Next Minor GC
//    - Eden + S0 copied to S1
//    - S0 cleared
//    - Age counter incremented

// 4. After several Minor GCs (default: 15)
//    - Object promoted to Old Generation (Tenured)

// 5. Old Generation fills → Major GC (Full GC)
//    - Collect Old Generation
//    - Usually slower, causes application pause
```

**Age-based Promotion:**

```java
// Object header contains age (4 bits = max 15)
// After each Minor GC, age++
// When age > MaxTenuringThreshold, promote to Old

// Configure threshold:
-XX:MaxTenuringThreshold=15  // Default
-XX:MaxTenuringThreshold=3   // More aggressive promotion
```

**Why Generational GC?**

**Generational Hypothesis:**
* Most objects die young
* Old objects rarely reference young objects

**Benefits:**
* Collect young generation frequently (most objects dead)
* Collect old generation rarely (most objects still alive)
* Minor GC much faster than Full GC

**Heap Sizing:**

```bash
# Set initial heap size
-Xms2g

# Set maximum heap size
-Xmx4g

# Set young generation size
-Xmn1g

# Or set ratio (Young:Total)
-XX:NewRatio=2  # Young = 1/3 of heap

# Survivor space ratio (Eden:Survivor)
-XX:SurvivorRatio=8  # Eden:S0:S1 = 8:1:1
```

**Example Configuration:**

```bash
# Application with 4GB heap
-Xms4g -Xmx4g        # Fixed size (avoids resize overhead)
-Xmn1g               # Young = 1GB, Old = 3GB
-XX:SurvivorRatio=8  # Eden=800MB, S0=100MB, S1=100MB
```

#### 3. Stack (Java Virtual Machine Stacks)

Each thread has its own **stack** containing stack frames.

**Stack Frame Structure:**

```
┌────────────────────────────────┐
│         Stack Frame            │
│  ┌──────────────────────────┐ │
│  │   Local Variable Array   │ │  Method parameters + local variables
│  │   [0] = this (instance)  │ │
│  │   [1] = param1           │ │
│  │   [2] = param2           │ │
│  │   [3] = localVar1        │ │
│  └──────────────────────────┘ │
│  ┌──────────────────────────┐ │
│  │    Operand Stack         │ │  Intermediate values during computation
│  │    [top] = value3        │ │
│  │           value2          │ │
│  │           value1          │ │
│  └──────────────────────────┘ │
│  ┌──────────────────────────┐ │
│  │    Frame Data            │ │  Constant pool reference, return address
│  │  - Constant pool ptr     │ │
│  │  - Return address        │ │
│  │  - Exception handling    │ │
│  └──────────────────────────┘ │
└────────────────────────────────┘
```

**Stack in Action:**

```java
public class StackExample {
    public static void main(String[] args) {     // Frame 1
        int x = 10;
        int result = calculate(x, 20);          // Frame 2 created
        System.out.println(result);
    }
    
    public static int calculate(int a, int b) { // Frame 2
        int sum = a + b;                         // Local variable
        return sum;                              // Frame 2 destroyed
    }
}

// Stack state during execution:
/*
main() called:
┌─────────────────┐
│ Frame: main()   │
│ - args          │
│ - x = 10        │
└─────────────────┘

calculate() called:
┌─────────────────┐
│ Frame: calc()   │  ← Current frame
│ - a = 10        │
│ - b = 20        │
│ - sum = 30      │
├─────────────────┤
│ Frame: main()   │
│ - args          │
│ - x = 10        │
│ - result = ?    │
└─────────────────┘

calculate() returns:
┌─────────────────┐
│ Frame: main()   │  ← Current frame
│ - args          │
│ - x = 10        │
│ - result = 30   │
└─────────────────┘
*/
```

**Stack Overflow:**

```java
// Recursive method without base case
public void recursiveMethod() {
    recursiveMethod();  // Infinite recursion
}
// Eventually: StackOverflowError

// Stack overflow occurs when:
// 1. Too deep recursion
// 2. Very large local variables/arrays
// 3. Stack size too small for application needs
```

**Stack Size Configuration:**

```bash
# Set thread stack size
-Xss1m    # 1 MB per thread (default varies by platform)
-Xss512k  # Reduce if many threads (saves memory)
-Xss2m    # Increase if deep recursion needed
```

**Stack vs Heap:**

| Aspect | Stack | Heap |
|--------|-------|------|
| Storage | Local variables, method calls | Objects |
| Scope | Thread-private | Shared across threads |
| Size | Small (typically 1MB) | Large (typically GBs) |
| Allocation | Fast (pointer bump) | Slower (find free space) |
| Deallocation | Automatic (method return) | Garbage collection |
| Structure | LIFO (Last-In-First-Out) | Unorganized |
| Fragmentation | No fragmentation | Can fragment |

#### 4. PC Register (Program Counter Register)

Each thread has a **PC register** that holds the address of the currently executing instruction.

```java
// Bytecode example
public int add(int a, int b) {
    return a + b;
}

// Compiled bytecode:
0: iload_1         // PC = 0: Load parameter a
1: iload_2         // PC = 1: Load parameter b
2: iadd            // PC = 2: Add values
3: ireturn         // PC = 3: Return result

// PC register points to current instruction
```

**Purpose:**
* Track current execution point
* Support multithreading (each thread has own PC)
* Enable thread context switching

#### 5. Native Method Stacks

Similar to Java stacks, but for **native methods** (C/C++ code called via JNI).

```java
// Native method declaration
public class NativeExample {
    // Native method (implemented in C/C++)
    public native void nativeMethod();
    
    static {
        // Load native library
        System.loadLibrary("nativelib");
    }
}

// When nativeMethod() called:
// - Java stack frame removed
// - Native method stack frame created
// - Native code executes
// - Native frame removed
// - Java stack frame restored
```

### Execution Engine

The **Execution Engine** executes the bytecode. It has three main components:

#### 1. Interpreter

**Reads and executes bytecode line by line.**

```java
// Java source
int sum = a + b;

// Bytecode
iload_1    // Load 'a'
iload_2    // Load 'b'
iadd       // Add
istore_3   // Store result in 'sum'

// Interpreter:
// Execute → iload_1
// Execute → iload_2
// Execute → iadd
// Execute → istore_3
```

**Characteristics:**
* ✅ Fast startup (no compilation delay)
* ❌ Slower execution (each instruction interpreted)
* ✅ Simple implementation
* ❌ No optimization

**When Used:**
* Application startup
* Rarely executed code
* Before JIT compilation kicks in

#### 2. JIT Compiler (Just-In-Time Compiler)

**Compiles hot bytecode to native machine code** for better performance.

**How JIT Works:**

```
┌──────────┐     Interpret     ┌──────────┐
│ Bytecode │ ─────────────────>│ Interpret│
└──────────┘                   └──────────┘
     │                              │
     │                              ▼
     │                         ┌──────────┐
     │                         │ Profile  │
     │                         │ Count    │
     │                         │ calls    │
     │                         └──────────┘
     │                              │
     │                              ▼
     │                         Hot spot?
     │                              │
     │                              ▼ Yes
     │                         ┌──────────┐
     └────────────────────────>│   JIT    │
                               │ Compile  │
                               └──────────┘
                                    │
                                    ▼
                          ┌──────────────────┐
                          │ Native Machine   │
                          │      Code        │
                          │  (very fast!)    │
                          └──────────────────┘
```

**Compilation Thresholds:**

```bash
# Method invocation threshold (default: 10,000 for server JVM)
-XX:CompileThreshold=10000

# After 10,000 invocations, method compiled to native code

# Example:
public void hotMethod() {
    // Method body
}

// Execution pattern:
// Calls 1-9,999: Interpreted
// Call 10,000: JIT compiles method
// Calls 10,001+: Execute native code (fast!)
```

**JIT Compiler Types:**

**C1 Compiler (Client):**
* Fast compilation
* Basic optimizations
* Lower compilation overhead
* Used for client applications

**C2 Compiler (Server):**
* Slower compilation
* Aggressive optimizations
* Higher compilation overhead
* Better peak performance
* Used for long-running server applications

**Tiered Compilation (Default since Java 8):**

```
Level 0: Interpreter
    ↓
Level 1: C1 with full optimization
    ↓
Level 2: C1 with invocation and backedge counters
    ↓
Level 3: C1 with full profiling
    ↓
Level 4: C2 with aggressive optimization
```

**Benefits:**
* Fast startup (C1 compiles quickly)
* Good peak performance (C2 optimizes aggressively)
* Best of both worlds

**Enabling/Disabling:**

```bash
# Disable JIT (interpret only) - for debugging
-Xint

# Disable interpreter (compile all) - not recommended
-Xcomp

# Tiered compilation (default)
-XX:+TieredCompilation

# Disable tiered compilation (C2 only)
-XX:-TieredCompilation
```

**JIT Optimizations:**

```java
// Example: Method inlining
public int compute(int x) {
    return add(x, 5) * 2;
}

public int add(int a, int b) {
    return a + b;
}

// Before inlining (interpreted):
compute(10):
  call add(10, 5) → 15
  multiply 15 * 2 → 30

// After inlining (JIT compiled):
compute(10):
  // add() method inlined
  temp = 10 + 5     // 15
  result = temp * 2 // 30
  // No method call overhead!
```

**Other JIT Optimizations:**
* **Dead code elimination**: Remove unused code
* **Loop unrolling**: Reduce loop overhead
* **Escape analysis**: Allocate objects on stack if don't escape method
* **Lock elision**: Remove unnecessary synchronization
* **Constant folding**: Evaluate constants at compile time
* **Null check elimination**: Remove redundant null checks

**Monitoring JIT:**

```bash
# Print compilation
-XX:+PrintCompilation

# Output:
#     25    1       java.lang.String::hashCode (55 bytes)
#     27    2       java.lang.String::charAt (33 bytes)
#    101    3  n    java.lang.System::arraycopy (native)
# │    │    │  │    │
# │    │    │  │    └─ Method name
# │    │    │  └─ Flags (n=native, s=synchronized, etc.)
# │    │    └─ Compilation level (1-4)
# │    └─ Compilation ID
# └─ Timestamp (ms)
```

#### 3. Garbage Collector

**Automatic memory management** that reclaims memory from unreachable objects. This is covered in detail in Section 2.3.

### Native Method Interface (JNI)

**JNI** allows Java code to call native (C/C++) code and vice versa.

**Use Cases:**
* Performance-critical operations
* Hardware interaction
* Legacy system integration
* Using platform-specific libraries

**Example:**

```java
public class NativeExample {
    // Declare native method
    public native void printHello();
    
    // Load native library
    static {
        System.loadLibrary("hello");  // Loads libhello.so or hello.dll
    }
    
    public static void main(String[] args) {
        new NativeExample().printHello();
    }
}
```

**C Implementation (hello.c):**

```c
#include <jni.h>
#include <stdio.h>
#include "NativeExample.h"

JNIEXPORT void JNICALL Java_NativeExample_printHello
  (JNIEnv *env, jobject obj) {
    printf("Hello from C!\n");
}
```

**Compilation:**

```bash
# Generate header
javac -h . NativeExample.java

# Compile C code
gcc -shared -fPIC -I${JAVA_HOME}/include \
    -I${JAVA_HOME}/include/linux \
    -o libhello.so hello.c

# Run
java -Djava.library.path=. NativeExample
```

---

## 2.2 Class Loading Deep Dive

Understanding class loading is crucial for debugging ClassNotFoundException, NoClassDefFoundError, and classloader-related issues.

### When Are Classes Loaded?

**Lazy Loading Principle:** Classes loaded on first use, not at application startup.

```java
public class LazyLoadingDemo {
    public static void main(String[] args) {
        System.out.println("Main started");
        // MyClass NOT loaded yet
        
        MyClass obj = new MyClass();  // MyClass loaded NOW
    }
}

class MyClass {
    static {
        System.out.println("MyClass static block");
    }
}

// Output:
// Main started
// MyClass static block
```

**Triggers for Class Loading:**

1. **Creating instance**: `new MyClass()`
2. **Accessing static member**: `MyClass.staticField` or `MyClass.staticMethod()`
3. **Reflection**: `Class.forName("MyClass")`
4. **Loading subclass**: Triggers parent class loading
5. **JVM startup**: Main class loading

**Not Triggers:**

```java
// These DON'T load the class:
MyClass ref;  // Just reference declaration
MyClass[] arr = new MyClass[10];  // Array creation (elements still null)
```

### Class Loading Phases in Detail

#### Phase 1: Loading

```java
// ClassLoader finds and reads .class file

// 1. Check if already loaded
if (alreadyLoaded(className)) {
    return cachedClass;
}

// 2. Delegate to parent
if (parent != null) {
    Class c = parent.loadClass(className);
    if (c != null) return c;
}

// 3. Find class file
byte[] classData = findClassFile(className);

// 4. Parse bytecode
Class<?> clazz = defineClass(className, classData);

// 5. Cache and return
cache(clazz);
return clazz;
```

#### Phase 2: Linking

**a) Verification**

Ensures class file is well-formed and safe:

```java
// Checks performed:
// 1. Magic number: 0xCAFEBABE
// 2. Version compatibility
// 3. Constant pool validity
// 4. Type safety
// 5. Flow analysis
// 6. Symbolic reference validation

// If verification fails:
// VerifyError or ClassFormatError
```

**b) Preparation**

Allocates memory and sets default values:

```java
public class PrepExample {
    private static int count;           // Set to 0
    private static String name;         // Set to null
    private static boolean flag;        // Set to false
    private static final int MAX = 100; // NOT set to 100 yet!
    
    // After preparation:
    // count = 0, name = null, flag = false, MAX = 0
}
```

**c) Resolution**

Converts symbolic references to direct memory addresses:

```java
// Before resolution (symbolic):
class UserService {
    public void process() {
        Database.query();  // Symbolic reference: "Database.query()"
    }
}

// After resolution (direct):
class UserService {
    public void process() {
        // Direct memory address: 0x1234ABCD
        invoke 0x1234ABCD
    }
}
```

#### Phase 3: Initialization

Executes static initializers in order:

```java
public class InitOrder {
    // 1. Static variables (in order)
    private static int a = 10;
    private static int b = 20;
    
    // 2. Static blocks (in order)
    static {
        System.out.println("First static block: a=" + a + ", b=" + b);
        a = 30;
    }
    
    static {
        System.out.println("Second static block: a=" + a + ", b=" + b);
        b = 40;
    }
    
    private static int c = compute();
    
    static int compute() {
        System.out.println("compute() called: a=" + a + ", b=" + b);
        return a + b;
    }
    
    // Final values: a=30, b=40, c=70
}

// Output:
// First static block: a=10, b=20
// Second static block: a=30, b=20
// compute() called: a=30, b=40
```

**Initialization with Inheritance:**

```java
class Parent {
    static {
        System.out.println("1. Parent static");
    }
    
    {
        System.out.println("3. Parent instance");
    }
    
    public Parent() {
        System.out.println("4. Parent constructor");
    }
}

class Child extends Parent {
    static {
        System.out.println("2. Child static");
    }
    
    {
        System.out.println("5. Child instance");
    }
    
    public Child() {
        System.out.println("6. Child constructor");
    }
}

// new Child();
// Output:
// 1. Parent static
// 2. Child static
// 3. Parent instance
// 4. Parent constructor
// 5. Child instance
// 6. Child constructor
```

### ClassLoader Isolation

Different classloaders create **separate namespaces**:

```java
// Same class loaded by different classloaders
ClassLoader loader1 = new CustomClassLoader();
ClassLoader loader2 = new CustomClassLoader();

Class<?> class1 = loader1.loadClass("com.example.MyClass");
Class<?> class2 = loader2.loadClass("com.example.MyClass");

// Different classes!
System.out.println(class1 == class2);  // false

Object obj1 = class1.getDeclaredConstructor().newInstance();
Object obj2 = class2.getDeclaredConstructor().newInstance();

// ClassCastException!
MyClass casted = (MyClass) obj2;  // If obj1's class used here
```

**Full Class Identity:**

```
ClassIdentity = FullyQualifiedClassName + ClassLoader
```

Two classes are the same only if:
1. Same fully qualified name, AND
2. Loaded by same classloader

**Use Cases:**
* **Application servers**: Isolate different applications
* **Plugin systems**: Load/unload plugins without restarting
* **OSGi**: Module isolation
* **Hot deployment**: Reload classes without restart

### Common ClassLoader Issues

#### ClassNotFoundException

**Cause**: Class not found in classpath

```java
try {
    Class.forName("com.example.NonExistent");
} catch (ClassNotFoundException e) {
    // Class file doesn't exist in classpath
}
```

**Solutions:**
1. Check classpath
2. Verify package name and class name
3. Ensure JAR/directory in classpath

#### NoClassDefFoundError

**Cause**: Class was found during compilation but not at runtime

```java
// Compile time: MyClass exists
MyClass obj = new MyClass();

// Runtime: MyClass.class deleted or not in classpath
// NoClassDefFoundError
```

**Difference:**

| ClassNotFoundException | NoClassDefFoundError |
|----------------------|---------------------|
| Checked exception | Error (unchecked) |
| Explicit loading (Class.forName) | Implicit loading (new, static access) |
| Class never found | Class was found but now missing |

#### LinkageError

**Cause**: Different versions of same class loaded

```java
// Version 1 of library.jar compiled against app
MyClass obj = new MyClass();
obj.method();  // method() exists in version 1

// Runtime: version 2 of library.jar (method() removed)
// LinkageError: NoSuchMethodError
```

**Prevention:**
* Dependency management (Maven, Gradle)
* Fat JAR with shaded dependencies
* ClassLoader isolation

---

## 2.3 Memory Management

Memory management is one of the most critical aspects of JVM understanding. Proper memory management prevents OutOfMemoryErrors, reduces GC pauses, and improves application performance.

### Heap Memory Deep Dive

#### Object Allocation

**Fast Path Allocation** (most common case):

```java
MyObject obj = new MyObject();

// JVM allocates memory:
// 1. Check if space available in Eden (Thread Local Allocation Buffer)
// 2. If yes: Bump the pointer (O(1) operation)
//    Eden pointer += object size
// 3. If no: Trigger Minor GC

// Thread Local Allocation Buffer (TLAB)
// Each thread has its own buffer in Eden
// No synchronization needed for allocation
```

**TLAB (Thread-Local Allocation Buffer):**

```
Eden Space:
┌────────────────────────────────────────┐
│ Thread 1 TLAB │ Thread 2 TLAB │ Free  │
│ [allocated]    │ [allocated]    │       │
└────────────────────────────────────────┘

Benefits:
- No contention between threads
- Fast allocation (pointer bump)
- Reduced synchronization overhead
```

**Configuring TLAB:**

```bash
# Enable TLAB (default: enabled)
-XX:+UseTLAB

# TLAB size (default: auto-sized)
-XX:TLABSize=1m

# Disable for debugging
-XX:-UseTLAB
```

**Large Object Allocation:**

```java
// Large objects (> half of Eden) allocated directly in Old Generation
byte[] hugeArray = new byte[100 * 1024 * 1024];  // 100MB

// Why?
// - Would immediately trigger Minor GC if in Eden
// - Better to allocate in Old directly
```

**G1GC Humongous Objects:**

```bash
# In G1GC, objects > 50% of region size are "humongous"
# Default region size: heap / 2048
# Humongous objects allocated in special humongous regions

-XX:G1HeapRegionSize=4m  // Explicit region size
```

#### Object Layout in Memory

**Object Structure:**

```
Object in Memory:
┌──────────────────────────────────────┐
│         Object Header                 │  8-16 bytes
│  ┌────────────────────────────────┐  │
│  │  Mark Word (8 bytes)           │  │  Hash code, GC age, lock info
│  ├────────────────────────────────┤  │
│  │  Class Pointer (4-8 bytes)     │  │  Pointer to class metadata
│  └────────────────────────────────┘  │
├──────────────────────────────────────┤
│         Instance Data                 │  Variable size
│  ┌────────────────────────────────┐  │
│  │  Field 1                       │  │
│  │  Field 2                       │  │
│  │  Field 3                       │  │
│  └────────────────────────────────┘  │
├──────────────────────────────────────┤
│         Padding                       │  0-7 bytes (8-byte alignment)
└──────────────────────────────────────┘
```

**Mark Word (64-bit JVM):**

```
Normal state:
┌──────────┬──────────┬──────┬──────┬──────────────┐
│ Unused   │ hashCode │  age │ lock │ Lock state   │
│ 25 bits  │ 31 bits  │4 bits│2 bits│              │
└──────────┴──────────┴──────┴──────┴──────────────┘

Locked state:
┌──────────────────────────────────┬──────┬──────┐
│ Pointer to lock record           │  00  │ Locked
│ 62 bits                           │2 bits│      │
└──────────────────────────────────┴──────┴──────┘
```

**Example Object Size:**

```java
public class Person {
    private String name;      // 4-8 bytes (compressed oops: 4 bytes)
    private int age;          // 4 bytes
    private boolean active;   // 1 byte
}

// Memory layout (compressed oops enabled):
// Object header: 12 bytes (8 + 4)
// name reference: 4 bytes
// age: 4 bytes
// active: 1 byte
// Padding: 3 bytes (align to 8 bytes)
// Total: 24 bytes
```

**Compressed OOPs (Ordinary Object Pointers):**

```bash
# Enabled by default for heaps < 32GB
# Reduces object pointers from 8 bytes to 4 bytes

-XX:+UseCompressedOops  # Enable (default for < 32GB heap)
-XX:-UseCompressedOops  # Disable

# With compressed oops: Can address up to 32GB with 4-byte pointers
# 2^32 * 8 bytes (object alignment) = 32GB
```

**Measuring Object Size:**

```java
import org.openjdk.jol.info.ClassLayout;

public class ObjectSizeExample {
    public static void main(String[] args) {
        Object obj = new Object();
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());
        
        Person person = new Person();
        System.out.println(ClassLayout.parseInstance(person).toPrintable());
    }
}

// Output shows exact memory layout
```

#### Memory Leak Detection

**What is a Memory Leak in Java?**

Objects that are no longer needed but still referenced, preventing GC.

**Common Causes:**

1. **Static Collections:**

```java
// ❌ BAD: Memory leak
public class Cache {
    private static Map<String, Object> cache = new HashMap<>();
    
    public void add(String key, Object value) {
        cache.put(key, value);  // Never removed!
    }
}

// ✅ GOOD: Bounded cache
public class BoundedCache {
    private static Map<String, Object> cache = 
        Collections.synchronizedMap(new LinkedHashMap<String, Object>() {
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > 1000;  // Max 1000 entries
            }
        });
}

// ✅ BETTER: Use WeakHashMap for cache
private static Map<String, Object> cache = new WeakHashMap<>();
```

2. **Unclosed Resources:**

```java
// ❌ BAD: Resource leak
public void readFile(String path) throws IOException {
    FileInputStream fis = new FileInputStream(path);
    // Process file
    // fis never closed if exception occurs!
}

// ✅ GOOD: try-with-resources
public void readFile(String path) throws IOException {
    try (FileInputStream fis = new FileInputStream(path)) {
        // Process file
    }  // Always closed
}
```

3. **Listeners and Callbacks:**

```java
// ❌ BAD: Listener not removed
public class EventSource {
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);  // Never removed!
    }
}

// ✅ GOOD: Remove listeners
public void cleanup() {
    eventSource.removeListener(this);
}

// ✅ BETTER: Use WeakReference
private List<WeakReference<EventListener>> listeners = new ArrayList<>();
```

4. **ThreadLocal:**

```java
// ❌ BAD: ThreadLocal not cleaned
public class UserContext {
    private static ThreadLocal<User> currentUser = new ThreadLocal<>();
    
    public static void setUser(User user) {
        currentUser.set(user);  // Never removed in thread pool!
    }
}

// ✅ GOOD: Always remove
public static void clearUser() {
    currentUser.remove();  // Call when done
}

// In thread pool, always clean up:
try {
    UserContext.setUser(user);
    // Process request
} finally {
    UserContext.clearUser();  // Prevent leak in reused thread
}
```

**Detecting Memory Leaks:**

```bash
# 1. Monitor heap usage
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:gc.log

# 2. Take heap dumps
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dumps

# 3. Manual heap dump
jmap -dump:format=b,file=heap.bin <pid>

# 4. Analyze with tools
# - Eclipse MAT (Memory Analyzer Tool)
# - VisualVM
# - YourKit
```

**Heap Dump Analysis:**

```java
// Using jmap
jmap -dump:live,format=b,file=heap.bin 12345

// Analyze with MAT:
// 1. Load heap.bin
// 2. Run "Leak Suspects Report"
// 3. Check "Dominator Tree" for large retentions
// 4. Look for:
//    - Unexpected large collections
//    - Objects with high retention
//    - Duplicate strings
```

### Stack Memory

#### Stack Frame Lifecycle

```java
public class StackDemo {
    public static void main(String[] args) {        // Frame 1
        int x = 10;
        methodA(x);                                 // Frame 2 created
        System.out.println("Back in main");
    }                                               // Frame 1 destroyed
    
    public static void methodA(int param) {         // Frame 2
        int local = param * 2;
        methodB(local);                             // Frame 3 created
        System.out.println("Back in methodA");
    }                                               // Frame 2 destroyed
    
    public static void methodB(int value) {         // Frame 3
        System.out.println("In methodB: " + value);
    }                                               // Frame 3 destroyed
}

/*
Stack evolution:
                                          
main() starts:                methodA() called:             methodB() called:
┌──────────────┐             ┌──────────────┐             ┌──────────────┐
│ Frame: main()│             │ Frame:       │             │ Frame:       │
│ - args       │             │ methodA()    │             │ methodB()    │
│ - x=10       │             │ - param=10   │             │ - value=20   │
└──────────────┘             │ - local=20   │             ├──────────────┤
                              ├──────────────┤             │ Frame:       │
                              │ Frame: main()│             │ methodA()    │
                              │ - args       │             │ - param=10   │
                              │ - x=10       │             │ - local=20   │
                              └──────────────┘             ├──────────────┤
                                                           │ Frame: main()│
                                                           │ - args       │
                                                           │ - x=10       │
                                                           └──────────────┘

methodB() returns:            methodA() returns:           End:
┌──────────────┐             ┌──────────────┐             ┌──────────────┐
│ Frame:       │             │ Frame: main()│             (Empty)
│ methodA()    │             │ - args       │
│ - param=10   │             │ - x=10       │
│ - local=20   │             └──────────────┘
├──────────────┤
│ Frame: main()│
│ - args       │
│ - x=10       │
└──────────────┘
*/
```

#### Stack vs Heap Allocation

```java
public class AllocationExample {
    public void stackAllocation() {
        int x = 10;              // Stack: primitive
        int y = 20;              // Stack: primitive
        int sum = x + y;         // Stack: primitive
        
        // When method returns, all deallocated instantly
    }
    
    public void heapAllocation() {
        Person p = new Person(); // p: Stack (reference)
                                 // Person object: Heap
        
        // When method returns:
        // - p (reference) removed from stack
        // - Person object becomes eligible for GC
    }
    
    public void mixedAllocation() {
        int count = 5;                    // Stack
        String name = "Alice";            // Stack (reference)
                                          // Heap (String object in pool)
        List<String> items = new ArrayList<>();  // Stack (reference)
                                                 // Heap (ArrayList object)
        
        items.add("Item1");               // Heap ("Item1" string)
    }
}
```

**Escape Analysis Optimization:**

```java
// JIT can allocate on stack if object doesn't escape method
public void noEscape() {
    Point p = new Point(10, 20);  // May be stack-allocated!
    int x = p.getX();
    int y = p.getY();
    // p doesn't escape (not returned, not stored in field)
}

public Point escapes() {
    Point p = new Point(10, 20);  // Heap-allocated
    return p;  // Object escapes!
}

// Enable escape analysis (default: on)
-XX:+DoEscapeAnalysis
```

#### Stack Overflow Scenarios

**1. Deep Recursion:**

```java
public class RecursionOverflow {
    public static void recursiveMethod(int depth) {
        System.out.println("Depth: " + depth);
        recursiveMethod(depth + 1);  // No base case!
    }
    
    public static void main(String[] args) {
        recursiveMethod(1);  // StackOverflowError
    }
}

// Fix: Add base case
public static void recursiveMethod(int depth) {
    if (depth > 1000) return;  // Base case
    recursiveMethod(depth + 1);
}
```

**2. Large Local Arrays:**

```java
public void largeArray() {
    int[] huge = new int[10_000_000];  // May cause StackOverflowError
                                        // if stack size too small
}

// Fix: Allocate on heap
public void largeArrayHeap() {
    int[] huge = new int[10_000_000];  // Allocated on heap (fine)
}
```

**3. Circular Dependencies:**

```java
public class A {
    private B b = new B();  // Creates B
}

public class B {
    private A a = new A();  // Creates A → Creates B → ...
}
// StackOverflowError during class initialization
```

**Configuring Stack Size:**

```bash
# Default stack size (varies by platform)
# Windows 64-bit: 1MB
# Linux 64-bit: 1MB
# macOS: 1MB

# Set stack size
-Xss512k   # 512 KB (reduce for many threads)
-Xss2m     # 2 MB (increase for deep recursion)

# Calculate total memory for threads:
# Total = Number of threads × Stack size per thread
# Example: 1000 threads × 1MB = 1GB just for stacks!
```

### Metaspace (Method Area)

#### What's Stored in Metaspace

```java
public class Example {
    // ALL of this goes to Metaspace:
    
    // Class structure
    private int instanceField;
    private static int staticField;
    
    // Method bytecode
    public void method() {
        System.out.println("Hello");
    }
    
    // Constant pool
    private static final String CONSTANT = "Constant";
}
```

**Metaspace Contents:**
* Class metadata (structure, methods, fields)
* Method bytecode
* Static variables
* Constant pool (class-level constants)
* JIT compilation metadata

**NOT in Metaspace:**
* String literals (moved to heap in Java 7+)
* Object instances (always heap)

#### Metaspace vs PermGen

| Aspect | PermGen (Java 7) | Metaspace (Java 8+) |
|--------|------------------|---------------------|
| Location | Heap | Native memory |
| Size | Fixed | Dynamic (can grow) |
| Default Max | 64MB/82MB | Unlimited* |
| Resize | Manual | Automatic |
| OOM Errors | Common | Rare |
| GC | Full GC | Metaspace GC (rare) |

*Constrained by available native memory

**Why the Change?**

```java
// Problem with PermGen:
for (int i = 0; i < 100000; i++) {
    CustomClassLoader loader = new CustomClassLoader();
    Class<?> clazz = loader.loadClass("MyClass" + i);
    // PermGen fills up
}
// OutOfMemoryError: PermGen space

// With Metaspace:
// Automatically expands (limited by OS memory)
// Better for:
// - Dynamic class loading (application servers)
// - Reflection-heavy frameworks (Hibernate, Spring)
// - Groovy, JRuby, other JVM languages
```

#### Metaspace Configuration

```bash
# Initial metaspace size (triggers first GC)
-XX:MetaspaceSize=128m

# Maximum metaspace size (default: unlimited)
-XX:MaxMetaspaceSize=512m

# Minimum free space after GC (default: 40%)
-XX:MinMetaspaceFreeRatio=40

# Maximum free space after GC (default: 70%)
-XX:MaxMetaspaceFreeRatio=70

# Example configuration:
java -XX:MetaspaceSize=128m \
     -XX:MaxMetaspaceSize=512m \
     -jar myapp.jar
```

**When to Limit Metaspace:**

```bash
# Limit if:
# 1. Running in container with memory constraints
# 2. Want to fail-fast rather than consume all native memory
# 3. Debugging classloader leaks

# Don't limit if:
# 1. Application server with many apps
# 2. Dynamic class generation (Groovy, etc.)
# 3. Have abundant native memory
```

#### Metaspace Garbage Collection

```java
// Metaspace GC triggers when:
// 1. Metaspace reaches MetaspaceSize threshold
// 2. Class unloading needed (rare)

// Classes can be unloaded if:
// - ClassLoader is garbage collected
// - No instances of classes exist
// - No references to Class objects

// Example: Unloadable classes
ClassLoader loader = new URLClassLoader(urls);
Class<?> clazz = loader.loadClass("MyClass");
Object instance = clazz.getDeclaredConstructor().newInstance();

// For class to be unloaded:
instance = null;     // No instances
clazz = null;        // No class reference
loader = null;       // ClassLoader eligible for GC
System.gc();         // Request GC (may unload class)
```

**Monitoring Metaspace:**

```bash
# Using jstat
jstat -gc <pid> 1000

# Output includes:
# MC = Metaspace Capacity
# MU = Metaspace Used
# CCSC = Compressed Class Space Capacity
# CCSU = Compressed Class Space Used

# Example:
jstat -gc 12345 1000
#  MC       MU      CCSC     CCSU
#  51200.0  48500.0  6400.0   5800.0
```

```java
// Programmatic monitoring
MemoryPoolMXBean metaspaceBean = null;
for (MemoryPoolMXBean pool : ManagementFactory.getMemoryPoolMXBeans()) {
    if ("Metaspace".equals(pool.getName())) {
        metaspaceBean = pool;
        break;
    }
}

if (metaspaceBean != null) {
    MemoryUsage usage = metaspaceBean.getUsage();
    long used = usage.getUsed() / 1024 / 1024;  // MB
    long max = usage.getMax() / 1024 / 1024;    // MB
    System.out.printf("Metaspace: %d MB / %d MB%n", used, max);
}
```

### Direct Memory (Off-Heap)

**Direct ByteBuffers** allocate memory outside the JVM heap.

```java
import java.nio.ByteBuffer;

// Heap allocation
ByteBuffer heapBuffer = ByteBuffer.allocate(1024);
// Memory in Java heap

// Direct allocation
ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);
// Memory in native (off-heap) memory
```

**Characteristics:**

| Aspect | Heap ByteBuffer | Direct ByteBuffer |
|--------|----------------|-------------------|
| Location | Java heap | Native memory |
| GC | Subject to GC | Not in GC scope |
| Allocation | Fast | Slower |
| I/O | Copy required | Zero-copy I/O |
| Deallocation | Automatic | Cleaner thread |

**Use Cases:**

```java
// ✅ Good for: Large buffers, I/O operations
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024);  // 1MB
    while (channel.read(buffer) > 0) {
        buffer.flip();
        // Process buffer (zero-copy I/O)
        buffer.clear();
    }
}

// ❌ Bad for: Small, short-lived buffers
for (int i = 0; i < 1000; i++) {
    ByteBuffer buf = ByteBuffer.allocateDirect(100);  // Overhead!
}
```

**Configuration:**

```bash
# Maximum direct memory (default: -Xmx minus some overhead)
-XX:MaxDirectMemorySize=2g

# Monitor direct memory
jconsole  # BufferPool → direct
```

**Direct Memory Leaks:**

```java
// ❌ BAD: Direct buffer leak
for (int i = 0; i < 10000; i++) {
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024);
    // Never freed explicitly
}
// OutOfMemoryError: Direct buffer memory

// ✅ GOOD: Explicit cleanup
ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024);
try {
    // Use buffer
} finally {
    if (buffer.isDirect()) {
        ((DirectBuffer) buffer).cleaner().clean();
    }
}

// ✅ BETTER: Use try-with-resources pattern
```

### Memory Monitoring and Tuning

#### Key Metrics

```java
// Runtime memory info
Runtime runtime = Runtime.getRuntime();

long maxMemory = runtime.maxMemory();      // -Xmx
long totalMemory = runtime.totalMemory();  // Current heap size
long freeMemory = runtime.freeMemory();    // Free in current heap
long usedMemory = totalMemory - freeMemory;

System.out.printf("Max: %d MB%n", maxMemory / 1024 / 1024);
System.out.printf("Total: %d MB%n", totalMemory / 1024 / 1024);
System.out.printf("Used: %d MB%n", usedMemory / 1024 / 1024);
System.out.printf("Free: %d MB%n", freeMemory / 1024 / 1024);
```

```java
// Detailed memory info via JMX
MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();

MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
System.out.println("Heap: " + heapUsage);

MemoryUsage nonHeapUsage = memoryBean.getNonHeapMemoryUsage();
System.out.println("Non-Heap: " + nonHeapUsage);

// Memory pools
for (MemoryPoolMXBean pool : ManagementFactory.getMemoryPoolMXBeans()) {
    System.out.printf("%s (%s): %s%n",
        pool.getName(),
        pool.getType(),
        pool.getUsage()
    );
}
```

#### Command-Line Tools

```bash
# 1. jps - List Java processes
jps -v
# 12345 MyApplication -Xmx4g -Xms4g

# 2. jstat - Monitor GC and memory
jstat -gc 12345 1000  # Every 1 second
jstat -gcutil 12345   # Percentage utilization

# 3. jmap - Heap dump and histogram
jmap -heap 12345                    # Heap configuration and usage
jmap -histo 12345                   # Histogram of object counts
jmap -dump:live,format=b,file=heap.bin 12345  # Heap dump

# 4. jcmd - Multi-purpose
jcmd 12345 GC.heap_info            # Heap info
jcmd 12345 GC.class_histogram      # Class histogram
jcmd 12345 VM.native_memory summary  # Native memory tracking

# 5. jconsole - GUI monitoring
jconsole 12345

# 6. VisualVM - Advanced GUI
visualvm
```

#### Native Memory Tracking

```bash
# Enable native memory tracking
-XX:NativeMemoryTracking=summary  # or 'detail'

# Query native memory
jcmd <pid> VM.native_memory summary

# Output shows:
# - Java Heap
# - Class (Metaspace)
# - Thread stacks
# - Code (JIT compiled code)
# - GC (GC data structures)
# - Compiler
# - Symbols
# - Native Memory Tracking overhead
# - etc.

# Baseline
jcmd <pid> VM.native_memory baseline

# Compare to baseline
jcmd <pid> VM.native_memory summary.diff
```

---

## 2.4 Garbage Collection

Garbage Collection (GC) is automatic memory management that reclaims memory occupied by objects no longer in use. Understanding GC is essential for production applications.

### GC Fundamentals

#### Reachability

An object is **reachable** if it can be accessed from any **GC Root**.

**GC Roots include:**
* Local variables in active methods (stack)
* Static variables
* Active Java threads
* JNI references
* Synchronization monitors (locked objects)

```java
public class ReachabilityExample {
    private static Object staticRef;  // GC Root
    
    public void method() {
        Object localRef = new Object();  // GC Root (while method active)
        Object temp = new Object();      // GC Root
        
        temp = null;  // Object becomes unreachable → eligible for GC
        
        staticRef = new Object();  // Reachable via static reference
    }  // localRef out of scope → becomes unreachable
}
```

**Reachability Types:**

```java
import java.lang.ref.*;

// 1. Strong reference (default, prevents GC)
Object strong = new Object();  // Never GC'd while reference exists

// 2. Soft reference (GC'd when memory pressure)
SoftReference<Object> soft = new SoftReference<>(new Object());
Object obj = soft.get();  // May return null if GC'd

// 3. Weak reference (GC'd at next GC)
WeakReference<Object> weak = new WeakReference<>(new Object());
Object obj2 = weak.get();  // May return null after GC

// 4. Phantom reference (for cleanup, get() always returns null)
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(new Object(), queue);
```

**Use Cases:**

```java
// Soft references: Memory-sensitive caches
public class ImageCache {
    private Map<String, SoftReference<Image>> cache = new HashMap<>();
    
    public Image getImage(String path) {
        SoftReference<Image> ref = cache.get(path);
        Image image = (ref != null) ? ref.get() : null;
        
        if (image == null) {
            image = loadImage(path);
            cache.put(path, new SoftReference<>(image));
        }
        return image;
    }
}

// Weak references: Weak hash maps, listeners
WeakHashMap<Object, Metadata> metadata = new WeakHashMap<>();
// Entries automatically removed when key is GC'd

// Phantom references: Resource cleanup
PhantomReference<LargeResource> ref = new PhantomReference<>(resource, queue);
// Notified when resource is GC'd, can perform cleanup
```

#### GC Process Overview

```
1. Mark Phase
   - Start from GC roots
   - Traverse object graph
   - Mark all reachable objects

2. Sweep Phase (or Copy Phase)
   - Reclaim memory from unmarked objects
   - Compact memory (optional)

3. Compact Phase (optional)
   - Move objects to eliminate fragmentation
   - Update references
```

### Garbage Collection Algorithms

#### 1. Serial GC

**Single-threaded collector**, suitable for small applications.

```bash
# Enable Serial GC
-XX:+UseSerialGC
```

**Algorithm:**
* Young Generation: Copying algorithm (mark-copy)
* Old Generation: Mark-sweep-compact

```
Young GC (Minor GC):
┌────────────────────┐
│ Eden │ S0  │  S1   │  Before GC
│ ████ │ ██  │       │
└────────────────────┘
        ↓
┌────────────────────┐
│ Eden │ S0  │  S1   │  After GC
│      │     │  ███  │  Live objects copied to S1
└────────────────────┘

Old GC (Major GC):
- Stop-the-world pause
- Mark all live objects
- Sweep dead objects
- Compact remaining objects
```

**Characteristics:**
* ✅ Simple, predictable
* ✅ Low overhead
* ❌ Long pause times
* ❌ Doesn't use multiple CPUs

**Best for:**
* Single-CPU machines
* Small heap sizes (< 100MB)
* Client applications
* Batch processing (pauses acceptable)

#### 2. Parallel GC (Throughput GC)

**Multi-threaded collector** for maximum throughput.

```bash
# Enable Parallel GC (default in Java 8)
-XX:+UseParallelGC

# Configure GC threads
-XX:ParallelGCThreads=8  # Number of GC threads

# GC time ratio (app time vs GC time)
-XX:GCTimeRatio=99  # Spend 1/(1+99) = 1% time in GC
```

**Algorithm:**
* Young Generation: Parallel copying
* Old Generation: Parallel mark-compact

```
Parallel Young GC:
Thread 1: Scans Eden[0-25%]
Thread 2: Scans Eden[25-50%]
Thread 3: Scans Eden[50-75%]
Thread 4: Scans Eden[75-100%]
↓
All copy live objects to survivor space in parallel
```

**Characteristics:**
* ✅ High throughput
* ✅ Uses multiple CPUs
* ✅ Efficient for batch processing
* ❌ Long pause times (stop-the-world)
* ❌ Not suitable for latency-sensitive apps

**Tuning:**

```bash
# Maximize throughput
-XX:+UseParallelGC \
-XX:ParallelGCThreads=8 \
-XX:MaxGCPauseMillis=200 \  # Try to keep pauses under 200ms
-XX:GCTimeRatio=99           # 1% time in GC

# For large heaps
-Xms16g -Xmx16g \            # 16GB heap
-XX:+UseParallelGC \
-XX:ParallelGCThreads=16
```

**Best for:**
* Multi-CPU servers
* Batch processing
* Background tasks
* Applications where throughput > latency

#### 3. CMS (Concurrent Mark Sweep) - Deprecated

**Low-pause collector** (deprecated in Java 9, removed in Java 14).

```bash
# Enable CMS (don't use in new applications)
-XX:+UseConcMarkSweepGC
```

**Phases:**

```
1. Initial Mark (STW)       - Mark GC roots
2. Concurrent Mark          - Traverse object graph (concurrent)
3. Concurrent Preclean      - Handle objects modified during mark
4. Final Remark (STW)       - Finish marking
5. Concurrent Sweep         - Reclaim memory (concurrent)
6. Concurrent Reset         - Prepare for next GC
```

**Characteristics:**
* ✅ Low pause times
* ✅ Concurrent marking and sweeping
* ❌ CPU overhead during concurrent phases
* ❌ Fragmentation (no compaction)
* ❌ "Concurrent mode failure" if Old Gen fills during concurrent GC
* ❌ Deprecated and removed

**Replaced by:** G1GC and ZGC

#### 4. G1GC (Garbage First)

**Default GC since Java 9**, balances throughput and latency.

```bash
# Enable G1GC (default in Java 9+)
-XX:+UseG1GC

# Key tuning parameters
-XX:MaxGCPauseMillis=200           # Target pause time (default: 200ms)
-XX:G1HeapRegionSize=4m            # Region size (1MB-32MB)
-XX:InitiatingHeapOccupancyPercent=45  # Start concurrent marking at 45% occupancy
```

**Heap Structure:**

```
G1GC divides heap into equal-sized regions (typically 1-32MB):

┌────┬────┬────┬────┬────┬────┬────┬────┐
│ E  │ E  │ S  │ O  │ O  │ E  │ H  │ H  │  E = Eden
│    │    │    │    │    │    │    │    │  S = Survivor
└────┴────┴────┴────┴────┴────┴────┴────┘  O = Old
│ E  │ O  │ O  │ E  │ E  │ S  │ O  │ E  │  H = Humongous (large objects)
│    │    │    │    │    │    │    │    │
└────┴────┴────┴────┴────┴────┴────┴────┘

Regions can change type dynamically
```

**GC Phases:**

```
Young GC:
- Select all Eden and Survivor regions
- Copy live objects to new Survivor or Old regions
- Fully concurrent copying
- Pause time proportional to young gen size

Mixed GC:
- Young regions + some Old regions
- Selects Old regions with most garbage ("garbage first")
- Evacuates live objects
- Concurrent marking identifies garbage-rich regions

Concurrent Marking:
1. Initial Mark (STW, piggybacks on Young GC)
2. Root Region Scan (concurrent)
3. Concurrent Mark (concurrent)
4. Remark (STW, short)
5. Cleanup (STW, short)
```

**Characteristics:**
* ✅ Predictable pause times
* ✅ Good throughput
* ✅ Compacts memory (no fragmentation)
* ✅ Handles large heaps well (up to ~64GB)
* ✅ Default choice for most applications
* ❌ More CPU overhead than Parallel GC
* ❌ Can struggle with very large heaps (> 64GB)

**Tuning G1GC:**

```bash
# Basic configuration
java -Xms8g -Xmx8g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -jar myapp.jar

# For large heaps
java -Xms32g -Xmx32g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=100 \
     -XX:G1HeapRegionSize=16m \
     -jar myapp.jar

# Aggressive (low latency)
java -Xms16g -Xmx16g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=50 \
     -XX:G1ReservePercent=20 \
     -jar myapp.jar
```

**G1GC Monitoring:**

```bash
# GC logging
-Xlog:gc*:file=gc.log:time,uptime,level,tags

# Key metrics to watch:
# - Pause times (should be < MaxGCPauseMillis)
# - To-space exhausted (bad, means GC can't keep up)
# - Concurrent mark cycles (frequency indicates Old Gen pressure)
```

**Best for:**
* General-purpose applications
* Heaps 4GB - 64GB
* Balance of throughput and latency
* Production web applications

#### 5. ZGC (Z Garbage Collector)

**Ultra-low latency collector** (production-ready in Java 15+).

```bash
# Enable ZGC
-XX:+UseZGC

# Key parameters
-XX:ZCollectionInterval=5      # Max interval between GCs (seconds)
-XX:ZAllocationSpikeTolerance=2  # Tolerance for allocation spikes
-Xlog:gc*:file=gc.log          # GC logging
```

**Characteristics:**

```
Pause Times:
- Sub-millisecond pauses (< 1ms typical)
- Pause time independent of heap size
- Pause time independent of live set size

Scalability:
- Handles heaps from 8MB to 16TB
- Concurrent compaction
- Colored pointers for concurrent operations
```

**How ZGC Works:**

```
1. Concurrent Marking
   - Mark live objects while app runs
   - Uses colored pointers (42-bit addressing)
   
2. Concurrent Relocation
   - Relocate objects while app runs
   - Load barriers handle concurrent access
   
3. Brief Pauses
   - Root set scanning (< 1ms)
   - Relocation set selection (< 1ms)

Colored Pointers:
┌──────────────────┬──────────┬──────────────────┐
│ Object Address   │ Metadata │ Reserved         │
│ 42 bits          │ 4 bits   │ 18 bits          │
└──────────────────┴──────────┴──────────────────┘
                      ↑
                Marked, Remapped, Finalized bits
```

**Characteristics:**
* ✅ Extremely low pause times (< 1ms)
* ✅ Scales to massive heaps (16TB)
* ✅ Concurrent compaction
* ✅ Pause time doesn't grow with heap size
* ❌ Higher CPU overhead (~15% more than G1)
* ❌ Higher memory overhead (load barriers, metadata)
* ❌ Linux-only initially (now Windows/macOS in Java 16+)

**Configuration:**

```bash
# Basic ZGC
java -Xms16g -Xmx16g \
     -XX:+UseZGC \
     -jar myapp.jar

# Large heap
java -Xms128g -Xmx128g \
     -XX:+UseZGC \
     -XX:ConcGCThreads=8 \
     -jar myapp.jar

# Enable generational ZGC (Java 21+)
java -Xms16g -Xmx16g \
     -XX:+UseZGC \
     -XX:+ZGenerational \
     -jar myapp.jar
```

**Best for:**
* Latency-critical applications (trading systems, gaming)
* Large heaps (> 64GB)
* Applications requiring consistent response times
* Services with strict SLA requirements

#### 6. Shenandoah GC

**Low-pause GC** developed by Red Hat.

```bash
# Enable Shenandoah
-XX:+UseShenandoahGC

# Modes
-XX:ShenandoahGCMode=satb      # Snapshot-at-the-beginning (default)
-XX:ShenandoahGCMode=iu        # Incremental update
```

**Characteristics:**
* ✅ Low pause times (similar to ZGC)
* ✅ Concurrent compaction
* ✅ Pause times independent of heap size
* ❌ Slightly higher CPU overhead
* ❌ Brooks pointers (indirection overhead)
* ✅ Production-ready since Java 12

**Shenandoah vs ZGC:**

| Aspect | Shenandoah | ZGC |
|--------|------------|-----|
| Pause Times | 1-10ms | < 1ms |
| CPU Overhead | ~10% | ~15% |
| Memory Overhead | Lower (Brooks pointers) | Higher (colored pointers) |
| Max Heap | ~512GB | 16TB |
| Algorithm | Brooks forwarding | Colored pointers |

**Configuration:**

```bash
java -Xms32g -Xmx32g \
     -XX:+UseShenandoahGC \
     -XX:ShenandoahGCHeuristics=adaptive \
     -jar myapp.jar
```

**Best for:**
* Applications needing low latency
* Alternative to ZGC
* Red Hat ecosystem

### Choosing the Right GC

```
Decision Tree:

Heap Size?
├─ < 2GB → Serial GC or G1GC
├─ 2GB - 64GB
│   ├─ Batch/Throughput priority → Parallel GC
│   └─ Latency matters → G1GC (default choice)
└─ > 64GB
    ├─ Ultra-low latency (< 10ms) → ZGC or Shenandoah
    └─ Good latency (< 200ms) → G1GC

Special Cases:
- Trading systems, gaming → ZGC
- Real-time apps → ZGC or Shenandoah
- Batch processing → Parallel GC
- Embedded systems → Serial GC
```

**Recommendation Matrix:**

| Application Type | Heap Size | Recommended GC | Alternative |
|-----------------|-----------|----------------|-------------|
| Web Application | 4-16GB | G1GC | ZGC |
| Microservice | 1-4GB | G1GC | Parallel GC |
| Batch Processing | Any | Parallel GC | G1GC |
| Trading System | 16-128GB | ZGC | Shenandoah |
| Mobile/Embedded | < 512MB | Serial GC | - |
| Big Data | 32-512GB | ZGC | G1GC |

### GC Tuning

#### Common GC Flags

```bash
# Heap sizing
-Xms4g              # Initial heap size
-Xmx4g              # Maximum heap size (set equal to -Xms)
-Xmn1g              # Young generation size
-XX:NewRatio=2      # Young:Old ratio (1:2)

# GC selection
-XX:+UseG1GC
-XX:+UseZGC
-XX:+UseParallelGC
-XX:+UseShenandoahGC

# G1GC tuning
-XX:MaxGCPauseMillis=200        # Target pause time
-XX:G1HeapRegionSize=16m        # Region size
-XX:G1ReservePercent=10         # Reserve for to-space
-XX:InitiatingHeapOccupancyPercent=45  # Concurrent marking threshold

# Parallel GC tuning
-XX:ParallelGCThreads=8         # GC thread count
-XX:MaxGCPauseMillis=200        # Pause goal
-XX:GCTimeRatio=99              # Throughput goal

# ZGC tuning
-XX:ZCollectionInterval=5       # Max interval (seconds)
-XX:ConcGCThreads=4             # Concurrent threads

# GC logging
-Xlog:gc*:file=gc.log:time,uptime,level,tags
-XX:+PrintGCDetails             # Detailed GC info (Java 8)
-XX:+PrintGCDateStamps          # Timestamps (Java 8)
```

#### GC Tuning Strategy

**1. Baseline Measurement:**

```bash
# Enable detailed logging
java -Xms4g -Xmx4g \
     -XX:+UseG1GC \
     -Xlog:gc*:file=gc.log:time \
     -jar myapp.jar

# Run load test
# Analyze gc.log for:
# - Pause times
# - GC frequency
# - Throughput
# - Memory usage patterns
```

**2. Identify Issues:**

```
Signs of GC Problems:
✗ Long pause times (> target)
✗ Frequent Full GCs
✗ High GC CPU usage (> 10%)
✗ OutOfMemoryError
✗ To-space exhausted (G1GC)

Symptoms → Likely Cause:
Long pauses → Heap too large or wrong GC
Frequent GCs → Heap too small
Full GCs → Old Gen full or fragmentation
OOM → Heap too small or memory leak
```

**3. Tuning Steps:**

```bash
# Step 1: Right-size heap
# Monitor: Heap usage after Full GC
# Set: Xmx = Live Set * 3 to 4

# Example: Live set = 4GB
-Xms12g -Xmx12g  # 4GB * 3

# Step 2: Tune young generation
# Young too large → longer pauses
# Young too small → frequent Minor GCs

# Default G1GC: ~5-10% of heap
# Can adjust with NewRatio or Xmn

# Step 3: Set pause time goal
-XX:MaxGCPauseMillis=100  # G1GC target

# Step 4: Monitor and iterate
# - Check if goals met
# - Adjust based on metrics
# - Load test with real workload
```

**4. Example Tuning Scenarios:**

```bash
# Scenario 1: Long pause times (> 500ms)
# Problem: Heap too large for single-pause GC
# Solution: Switch to G1GC or ZGC

# Before:
java -Xms32g -Xmx32g -XX:+UseParallelGC -jar app.jar
# Pauses: 800ms

# After:
java -Xms32g -Xmx32g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -jar app.jar
# Pauses: 150ms

# Scenario 2: Frequent Full GCs
# Problem: Old Gen filling up
# Solution: Increase heap or fix memory leak

# Before:
java -Xms2g -Xmx2g -XX:+UseG1GC -jar app.jar
# Full GC every 10 minutes

# After (if not leak):
java -Xms4g -Xmx4g -XX:+UseG1GC -jar app.jar
# Full GC never

# Scenario 3: High GC CPU (> 20%)
# Problem: Too much GC activity
# Solution: Increase heap or reduce allocation rate

# Before:
java -Xms1g -Xmx1g -XX:+UseG1GC -jar app.jar
# GC CPU: 25%

# After:
java -Xms2g -Xmx2g -XX:+UseG1GC -jar app.jar
# GC CPU: 8%
```

### GC Analysis Tools

#### 1. GC Log Analysis

```bash
# Generate GC logs
java -Xlog:gc*:file=gc.log:time,uptime \
     -jar myapp.jar

# Analyze with GCViewer
# Download: https://github.com/chewiebug/GCViewer
java -jar gcviewer.jar gc.log

# Or use GCEasy (online)
# Upload gc.log to https://gceasy.io
```

**Key Metrics from GC Logs:**

```
Metrics to Monitor:
- Average pause time
- Maximum pause time
- GC frequency
- Throughput (app time / total time)
- Memory usage (before/after GC)
- Full GC frequency

Good Targets:
- Pause time: < 200ms (or your SLA)
- Throughput: > 95%
- Full GC: Rare (ideally never)
- GC CPU: < 10%
```

#### 2. jstat

```bash
# Real-time GC monitoring
jstat -gc <pid> 1000  # Every 1 second

# Output columns:
# S0C, S1C: Survivor capacities
# S0U, S1U: Survivor utilization
# EC, EU: Eden capacity/utilization
# OC, OU: Old capacity/utilization
# MC, MU: Metaspace capacity/utilization
# YGC: Young GC count
# YGCT: Young GC time
# FGC: Full GC count
# FGCT: Full GC time
# GCT: Total GC time

# Percentage utilization
jstat -gcutil <pid> 1000
```

#### 3. jcmd

```bash
# Heap info
jcmd <pid> GC.heap_info

# Run GC
jcmd <pid> GC.run

# Class histogram
jcmd <pid> GC.class_histogram

# Heap dump
jcmd <pid> GC.heap_dump filename=heap.bin
```

### Summary and Key Takeaways

**Memory Areas:**
* **Heap**: Object storage, managed by GC, split into Young/Old generations
* **Stack**: Method frames, thread-private, LIFO structure
* **Metaspace**: Class metadata, native memory, auto-sized
* **Direct Memory**: Off-heap ByteBuffers, not GC'd

**GC Algorithms:**
* **Serial GC**: Single-threaded, small heaps
* **Parallel GC**: Multi-threaded, throughput-focused
* **G1GC**: Balanced, default choice
* **ZGC**: Ultra-low latency, large heaps
* **Shenandoah**: Low latency alternative

**Choosing GC:**
* < 2GB: Serial or G1
* 2-64GB: G1GC (default)
* > 64GB: ZGC or Shenandoah
* Latency-critical: ZGC
* Batch: Parallel GC

**Tuning Principles:**
1. Set -Xms equal to -Xmx (avoid resizing)
2. Heap = Live Set × 3-4
3. Monitor before tuning
4. Change one thing at a time
5. Load test with production workload

### Frequently Asked Questions

**Q1: How much heap should I allocate?**

**A:** Heap size should be 3-4× your live set size (memory used after Full GC).

```bash
# Measure live set:
# 1. Run application
# 2. Force Full GC: jcmd <pid> GC.run
# 3. Check heap usage: jstat -gc <pid>

# If live set = 2GB:
-Xms6g -Xmx8g  # 3-4× live set
```

Too small: Frequent GC, poor throughput  
Too large: Long pause times

---

**Q2: When should I use ZGC vs G1GC?**

**A:**

**Use G1GC when:**
* Heap < 64GB
* Pause times < 200ms acceptable
* Want good throughput
* General-purpose application

**Use ZGC when:**
* Need ultra-low latency (< 10ms)
* Large heap (> 64GB)
* Consistent response times critical
* Can afford ~15% CPU overhead

---

**Q3: What causes OutOfMemoryError?**

**A:**

```java
// 1. Heap space exhausted
OutOfMemoryError: Java heap space
// Fix: Increase heap (-Xmx) or fix memory leak

// 2. Metaspace exhausted
OutOfMemoryError: Metaspace
// Fix: Increase metaspace (-XX:MaxMetaspaceSize) or fix classloader leak

// 3. Direct memory exhausted
OutOfMemoryError: Direct buffer memory
// Fix: Increase direct memory (-XX:MaxDirectMemorySize)

// 4. Unable to create native thread
OutOfMemoryError: unable to create new native thread
// Fix: Reduce thread count or increase OS limits
```

---

**Q4: How do I detect memory leaks?**

**A:**

```bash
# 1. Monitor heap usage over time
jstat -gc <pid> 1000

# 2. If heap grows continuously → leak
# Take heap dumps at intervals
jmap -dump:live,format=b,file=heap1.bin <pid>
# Wait 1 hour
jmap -dump:live,format=b,file=heap2.bin <pid>

# 3. Analyze with MAT
# Compare heap1 vs heap2
# Look for:
# - Growing collections
# - Objects not GC'd
# - Unexpected retentions
```

**Common leak sources:**
* Static collections
* Unclosed resources
* Listeners not removed
* ThreadLocal not cleaned
* Classloader leaks

---

**Q5: What's the difference between Minor GC and Full GC?**

**A:**

| Aspect | Minor GC | Full GC |
|--------|----------|---------|
| Scope | Young Generation only | Entire heap |
| Frequency | Very frequent | Rare |
| Duration | Milliseconds | Seconds |
| Impact | Low | High (STW pause) |
| Triggers | Eden full | Old Gen full, System.gc() |

```java
// Typical pattern:
// Minor GCs: Every few seconds
// Full GCs: Every few hours (or never!)

// Good: Frequent Minor GCs, rare Full GCs
// Bad: Frequent Full GCs (heap too small or leak)
```

### Interview Questions

**Question 1: Explain the generational hypothesis and why Java GC uses generational collection.**

**Difficulty:** Mid-Level

**Topics:** GC Fundamentals

**Answer:**

**Generational Hypothesis:** Most objects die young; few survive to old age.

**Implications:**
* ~90% of objects die shortly after creation
* Collecting young objects is very effective
* Old objects rarely die

**Why Generational GC:**

```
1. Frequent collection of Young Generation
   - Most objects dead → high reclaim rate
   - Small space → fast collection
   - Minor GC: milliseconds

2. Infrequent collection of Old Generation
   - Most objects alive → low reclaim rate
   - Large space → slow collection
   - Full GC: seconds

Benefits:
- Better performance (focus on young gen)
- Lower pause times (usually Minor GC)
- Higher throughput (less Full GC)
```

**Why This Matters:** Understanding this principle explains why GC works efficiently and guides tuning decisions.

**Follow-up Questions:**
* What happens if too many objects survive to Old Gen?
* How does this relate to object pooling?
* Why might functional programming styles affect GC?

---

**Question 2: Compare G1GC and ZGC. When would you choose each?**

**Difficulty:** Senior

**Topics:** GC Algorithms, Performance Tuning

**Answer:**

| Aspect | G1GC | ZGC |
|--------|------|-----|
| Pause Times | 10-200ms | < 1ms (sub-millisecond) |
| Heap Scaling | Up to ~64GB | Up to 16TB |
| CPU Overhead | ~5% | ~15% |
| Memory Overhead | Moderate | Higher |
| Compaction | Concurrent (partial) | Fully concurrent |
| Maturity | Production since Java 9 | Production since Java 15 |

**Choose G1GC:**
```bash
# General web applications
java -Xms8g -Xmx8g -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -jar webapp.jar

# Good for:
# - Standard applications
# - 200ms pause acceptable
# - Want good throughput
# - Heap < 64GB
```

**Choose ZGC:**
```bash
# Latency-critical trading system
java -Xms64g -Xmx64g -XX:+UseZGC \
     -jar trading-app.jar

# Good for:
# - Need consistent low latency
# - Large heaps (> 64GB)
# - Strict SLAs (p99 < 10ms)
# - Can afford CPU overhead
```

**Why This Matters:** Choosing the right GC is critical for meeting application performance requirements.

---

**Question 3: What is metaspace and how does it differ from PermGen?**

**Difficulty:** Mid-Level

**Topics:** Memory Management, JVM Architecture

**Answer:**

**Metaspace** (Java 8+) stores class metadata in native memory.

**Key Differences:**

| Feature | PermGen (≤ Java 7) | Metaspace (≥ Java 8) |
|---------|-------------------|----------------------|
| Location | Heap | Native memory |
| Size | Fixed (64/82MB default) | Dynamic (unlimited default) |
| Resize | No | Yes (auto-grows) |
| OOM Errors | Common | Rare |

**Why the Change:**

```java
// Problem with PermGen:
public class ClassLoaderLeak {
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 100000; i++) {
            CustomClassLoader loader = new CustomClassLoader();
            Class<?> clazz = loader.loadClass("MyClass" + i);
            // PermGen fills up → OutOfMemoryError
        }
    }
}

// With Metaspace:
// - Auto-expands (limited by OS memory)
// - Better for application servers
// - Better for dynamic languages (Groovy, JRuby)
```

**Configuration:**

```bash
# PermGen (Java 7)
-XX:PermSize=128m
-XX:MaxPermSize=256m

# Metaspace (Java 8+)
-XX:MetaspaceSize=128m
-XX:MaxMetaspaceSize=512m
```

**Why This Matters:** Understanding metaspace prevents OutOfMemoryError in applications with dynamic class loading.

---

**End of Part 2: JVM Internals & Memory Management**

### What's Next

In **Part 3: Advanced Java Features & Modern Java**, we'll explore:
* Concurrency and multithreading
* Virtual threads (Project Loom)
* Streams and functional programming
* Modern Java features (Java 9-21)
* Reactive programming

In **Part 4: SDLC with Java**, we'll cover:
* Design patterns and architecture
* Testing strategies
* Build tools (Maven, Gradle)
* CI/CD and deployment
* Monitoring and operations

Continue your journey to Java mastery with the next parts of this series!

### References

* [JVM Specification](https://docs.oracle.com/javase/specs/jvms/se21/html/)
* [HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/en/java/javase/21/gctuning/)
* [Java Performance by Scott Oaks](https://www.oreilly.com/library/view/java-performance-2nd/9781492056102/)
* [Optimizing Java by Benjamin J Evans](https://www.oreilly.com/library/view/optimizing-java/9781492025986/)
* [GC Handbook](https://gchandbook.org/)

---
