---
title: "Java Mastery - Part 1: Java Fundamentals & Language Basics"
date: 2025-07-02 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, OpenJDK, JVM, Fundamentals, OOP, Collections, Generics, Functional-programming]
---

## Introduction

Welcome to the first installment of the Complete Java Mastery series. This comprehensive guide will take you from absolute fundamentals to expert-level proficiency in Java programming, with a focus on OpenJDK—the open-source, production-ready Java platform that powers millions of applications worldwide.

Java remains one of the most widely used programming languages in enterprise software development, powering everything from Android applications to massive distributed systems. Understanding Java deeply—from its syntax and semantics to the Java Virtual Machine (JVM) that executes your code—is essential for any serious software engineer.

This series is structured around the complete Software Development Life Cycle (SDLC), ensuring you understand not just how to write Java code, but how to design, test, deploy, and maintain production-quality Java applications. We'll focus exclusively on OpenJDK, which is free for commercial use and represents the reference implementation of Java.

### What You'll Learn in Part 1

In this foundational part, we'll cover:

* **Java and OpenJDK fundamentals**: Understanding the Java platform, its philosophy, and setting up your development environment
* **Language basics**: Syntax, data types, operators, and control flow
* **Object-Oriented Programming**: Classes, objects, inheritance, polymorphism, abstraction, and encapsulation
* **Advanced OOP concepts**: Nested classes, enums, records, and sealed classes
* **Exception handling**: Robust error handling and resource management
* **Collections Framework**: Lists, sets, maps, and the data structures that power Java applications
* **Generics**: Type-safe programming with parameterized types
* **Functional programming**: Lambda expressions, streams, and the modern Java approach
* **I/O fundamentals**: Reading and writing data

By the end of this part, you'll have a solid foundation in Java programming and be ready to dive into JVM internals, advanced features, and enterprise development in subsequent parts.

---

## 1.1 Introduction to Java and OpenJDK

### What is Java and Why Does It Exist?

Java was created by James Gosling at Sun Microsystems (now Oracle) in 1995 with a revolutionary goal: **Write Once, Run Anywhere (WORA)**. Before Java, developers had to write different versions of their software for each operating system and hardware platform. Java solved this problem through an ingenious architecture.

**The Core Java Philosophy:**

1. **Platform Independence**: Java code compiles to bytecode that runs on any system with a Java Virtual Machine (JVM)
2. **Object-Oriented**: Everything in Java is an object (with exceptions for primitives), promoting code reusability and maintainability
3. **Secure**: Built-in security features and a managed runtime environment
4. **Robust**: Strong type checking, exception handling, and automatic memory management
5. **Simple**: Clean syntax influenced by C/C++ but without their complexity (no pointers, no manual memory management)

**How Java Achieves Platform Independence:**

```
Source Code (.java) 
    ↓ 
[Java Compiler (javac)]
    ↓
Bytecode (.class)
    ↓
[JVM - Windows] → Runs on Windows
[JVM - Linux]   → Runs on Linux  
[JVM - macOS]   → Runs on macOS
```

The same `.class` file runs on any platform that has a compatible JVM. This is the essence of WORA.

### OpenJDK vs Oracle JDK: Understanding Your Options

**OpenJDK (Open Java Development Kit)** is the free, open-source reference implementation of Java. As of 2019, Oracle JDK and OpenJDK are functionally identical for most purposes.

| Aspect | OpenJDK | Oracle JDK |
|--------|---------|------------|
| **License** | GPL v2 with Classpath Exception (free for all use) | Proprietary (free for development/testing, paid for production before Java 17) |
| **Source** | Completely open source | Based on OpenJDK with proprietary additions |
| **Support** | Community support, commercial support available from vendors | Oracle commercial support |
| **Updates** | Community-driven | Oracle-controlled |
| **Features** | Standard Java features | Same as OpenJDK since Java 11 |
| **Commercial Use** | ✅ Free | ⚠️ License changes over time |

**Key Takeaway**: For this series, we focus exclusively on OpenJDK because:
* It's free for commercial use without licensing concerns
* It's the reference implementation
* Major vendors (Red Hat, Amazon, Azul, etc.) provide production-ready distributions
* It receives regular updates and security patches

### Java Platform Components

Understanding the Java platform architecture is crucial:

```
┌─────────────────────────────────────────┐
│         Java Development Kit (JDK)      │
│  ┌───────────────────────────────────┐  │
│  │   Java Runtime Environment (JRE)  │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │  Java Virtual Machine (JVM) │  │  │
│  │  └─────────────────────────────┘  │  │
│  │  Core Libraries & APIs             │  │
│  └───────────────────────────────────┘  │
│  Development Tools (javac, jar, etc.)  │
└─────────────────────────────────────────┘
```

**JVM (Java Virtual Machine)**: 
* The runtime engine that executes Java bytecode
* Provides memory management (garbage collection)
* Performs JIT (Just-In-Time) compilation for performance
* Platform-specific (different JVM for Windows, Linux, etc.)

**JRE (Java Runtime Environment)**:
* JVM + core libraries and APIs
* Everything needed to run Java applications
* **Note**: As of Java 11, Oracle no longer provides standalone JRE downloads, but the runtime is included in JDK

**JDK (Java Development Kit)**:
* JRE + development tools (compiler, debugger, etc.)
* Everything needed to develop and run Java applications
* What you install as a developer

### Java Editions

Java comes in different editions for different use cases:

**Java SE (Standard Edition)**:
* Core Java platform
* What we focus on in this series
* Includes fundamental APIs, collections, I/O, networking, concurrency, etc.

**Java EE (Enterprise Edition) / Jakarta EE**:
* Built on Java SE
* Adds enterprise features: servlets, JSP, EJB, JPA, etc.
* Now governed by the Eclipse Foundation as Jakarta EE

**Java ME (Micro Edition)**:
* For embedded and mobile devices
* Subset of Java SE
* Less common in modern development

### Java Release Cycle and LTS Versions

Understanding Java's release model is important for production planning:

**Modern Release Cycle (Since Java 9)**:
* **New feature release every 6 months** (March and September)
* **LTS (Long-Term Support) release every 3 years** (approximately)

**Current LTS Versions**:
* **Java 8** (March 2014) - Legacy, still widely used
* **Java 11** (September 2018) - First modern LTS
* **Java 17** (September 2021) - Current recommended LTS
* **Java 21** (September 2023) - Latest LTS

**Non-LTS Versions** (feature releases):
* Java 9, 10, 12, 13, 14, 15, 16, 18, 19, 20, 22, 23...
* Receive updates only until the next release (6 months)
* Used for trying new features

**Recommendation for Production**:
* Use the latest LTS version (currently Java 21)
* Plan migration path from older LTS versions
* Test new features in non-LTS releases, adopt in next LTS

### Setting Up OpenJDK Development Environment

Let's get you set up with a proper Java development environment.

#### Installing OpenJDK

**Option 1: Using SDKMAN (Recommended for Linux/macOS)**

```bash
# Install SDKMAN
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Install OpenJDK 21 (LTS)
sdk install java 21-open

# List available Java versions
sdk list java

# Switch between Java versions
sdk use java 21-open
```

**Option 2: Using Package Managers**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install openjdk-21-jdk

# macOS (Homebrew)
brew install openjdk@21

# Fedora/RHEL
sudo dnf install java-21-openjdk-devel
```

**Option 3: Direct Download**

Visit [https://jdk.java.net/](https://jdk.java.net/) and download the appropriate build for your platform.

#### Verifying Installation

```bash
# Check Java version
java -version

# Expected output (version numbers may vary):
# openjdk version "21" 2023-09-19
# OpenJDK Runtime Environment (build 21+35)
# OpenJDK 64-Bit Server VM (build 21+35, mixed mode, sharing)

# Check compiler version
javac -version

# Expected output:
# javac 21
```

#### Setting JAVA_HOME

```bash
# Linux/macOS (add to ~/.bashrc or ~/.zshrc)
export JAVA_HOME=$(/usr/libexec/java_home -v 21)  # macOS
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk     # Linux
export PATH=$JAVA_HOME/bin:$PATH

# Windows (System Environment Variables)
# JAVA_HOME: C:\Program Files\Java\jdk-21
# PATH: %JAVA_HOME%\bin
```

### IDE Setup

While you can write Java in any text editor, modern IDEs provide invaluable features for professional development.

#### IntelliJ IDEA (Recommended)

**Why IntelliJ IDEA**:
* Best-in-class code completion and refactoring
* Excellent debugging capabilities
* Strong Spring Framework support
* Free Community Edition available

**Setup**:
1. Download from [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)
2. Install Community Edition (free) or Ultimate (paid, more features)
3. Configure JDK: File → Project Structure → Platform Settings → SDKs
4. Create new Java project: File → New → Project

#### Eclipse

**Popular alternative**:
* Completely free and open source
* Extensive plugin ecosystem
* Good for Java EE development

**Setup**:
1. Download Eclipse IDE for Java Developers
2. Install and launch
3. Set JDK: Window → Preferences → Java → Installed JREs

#### Visual Studio Code

**Lightweight option**:
* Fast and minimal
* Requires Java Extension Pack
* Good for smaller projects

**Setup**:
1. Install VS Code
2. Install Extension Pack for Java
3. Configure java.home setting

### Your First Java Program

Let's write the traditional "Hello, World!" program to understand the anatomy of a Java application.

#### Creating the Program

```java
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Anatomy Breakdown**:

```java
public class HelloWorld {
    // 'public': access modifier - visible everywhere
    // 'class': keyword to define a class
    // 'HelloWorld': class name (must match filename)
    
    public static void main(String[] args) {
        // 'public': accessible from anywhere
        // 'static': belongs to class, not instance
        // 'void': returns nothing
        // 'main': special method name - entry point
        // 'String[] args': command-line arguments
        
        System.out.println("Hello, World!");
        // 'System': built-in class
        // 'out': static field of type PrintStream
        // 'println': method to print with newline
    }
}
```

#### Compilation and Execution Process

```bash
# Step 1: Compile Java source to bytecode
javac HelloWorld.java

# This creates HelloWorld.class (bytecode file)

# Step 2: Run the bytecode on the JVM
java HelloWorld

# Output: Hello, World!
```

**What Happens Behind the Scenes**:

1. **Compilation** (`javac HelloWorld.java`):
   * Java compiler reads `.java` source file
   * Performs syntax checking and type validation
   * Generates platform-independent bytecode (`.class` file)
   * Bytecode is intermediate representation, not machine code

2. **Execution** (`java HelloWorld`):
   * JVM class loader loads `HelloWorld.class`
   * JVM verifies bytecode for security and correctness
   * JVM finds and invokes the `main()` method
   * JIT compiler may optimize hot code paths during execution

**Understanding Bytecode**:

```bash
# View the bytecode (human-readable disassembly)
javap -c HelloWorld

# Output shows JVM instructions:
# 0: getstatic     #7    // Field java/lang/System.out
# 3: ldc           #13   // String Hello, World!
# 5: invokevirtual #15   // Method java/io/PrintStream.println
# 8: return
```

This bytecode is what the JVM actually executes. It's platform-independent and the same on Windows, Linux, or macOS.

---

## 1.2 Java Language Fundamentals

Now that you understand what Java is and have your environment set up, let's dive into the core language syntax and constructs.

### Basic Syntax Rules

Java has strict syntax rules that you must follow:

1. **Case Sensitive**: `myVariable` and `myvariable` are different
2. **Class Names**: Should start with uppercase letter (PascalCase)
3. **Method Names**: Should start with lowercase letter (camelCase)
4. **File Name**: Must match the public class name exactly
5. **main() Method**: Every Java application needs one as the entry point
6. **Semicolons**: Statements end with semicolons
7. **Braces**: Code blocks are enclosed in `{ }`

### Package Declaration and Imports

**Packages** organize Java classes into namespaces, preventing naming conflicts and providing access control.

```java
// Package declaration (must be first statement if present)
package com.company.project.module;

// Import statements (after package, before class)
import java.util.ArrayList;        // Import specific class
import java.util.*;                // Import all classes from package (avoid in production)
import java.time.LocalDate;
import static java.lang.Math.PI;   // Static import

public class MyClass {
    // Class definition
}
```

**Package Naming Conventions**:
* Use reverse domain name: `com.company.project`
* All lowercase
* Separate words with dots

**Import Best Practices**:
* ✅ Import specific classes for clarity
* ❌ Avoid wildcard imports (`import java.util.*`) in production code
* Use static imports sparingly (for constants and utility methods)

### Comments

Java supports three types of comments:

```java
// Single-line comment
// Used for brief explanations

/*
 * Multi-line comment
 * Used for longer explanations
 * or temporarily disabling code
 */

/**
 * Javadoc comment
 * Used to generate API documentation
 * 
 * @param args command line arguments
 * @return void
 * @author Your Name
 * @since 1.0
 */
public static void main(String[] args) {
    // Code here
}
```

**Comment Best Practices**:
* Write self-documenting code that needs minimal comments
* Use comments to explain *why*, not *what*
* Keep comments up-to-date with code changes
* Use Javadoc for public APIs

### Data Types

Java is **strongly typed**, meaning every variable must have a declared type.

#### Primitive Types

Java has 8 primitive types:

| Type | Size | Range | Default | Description |
|------|------|-------|---------|-------------|
| `byte` | 8 bits | -128 to 127 | 0 | Smallest integer type |
| `short` | 16 bits | -32,768 to 32,767 | 0 | Short integer |
| `int` | 32 bits | -2³¹ to 2³¹-1 | 0 | Most common integer type |
| `long` | 64 bits | -2⁶³ to 2⁶³-1 | 0L | Large integer values |
| `float` | 32 bits | ~±3.4 × 10³⁸ | 0.0f | Single-precision decimal |
| `double` | 64 bits | ~±1.7 × 10³⁰⁸ | 0.0d | Double-precision decimal (default for decimals) |
| `char` | 16 bits | 0 to 65,535 | '\u0000' | Unicode character |
| `boolean` | 1 bit* | true/false | false | Boolean value |

*JVM implementation-dependent, but logically 1 bit

**Primitive Type Examples**:

```java
// Integer types
byte age = 25;
short year = 2024;
int population = 8_000_000_000;  // Underscores for readability (Java 7+)
long distance = 9_460_730_472_580_800L;  // L suffix for long literals

// Floating-point types
float price = 19.99f;      // f suffix required for float literals
double pi = 3.14159265359; // Default for decimal literals

// Character type
char grade = 'A';          // Single quotes for char
char unicode = '\u0041';   // Unicode for 'A'

// Boolean type
boolean isActive = true;
boolean hasPermission = false;
```

**Memory Footprint Matters**:

```java
// ❌ BAD: Wasting memory
long x = 5L;  // Using 64 bits for a small value

// ✅ GOOD: Appropriate type for value
int x = 5;    // Using 32 bits appropriately

// When dealing with millions of objects, memory adds up!
```

#### Wrapper Classes

Every primitive type has a corresponding **wrapper class** that treats it as an object:

| Primitive | Wrapper Class |
|-----------|---------------|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

**Why Wrapper Classes**:
* Required for collections (can't use primitives in generics)
* Provide utility methods
* Can be `null` (primitives cannot)

**Autoboxing and Unboxing** (Java 5+):

```java
// Autoboxing: automatic conversion from primitive to wrapper
Integer num = 10;  // Compiler converts to: Integer num = Integer.valueOf(10);

// Unboxing: automatic conversion from wrapper to primitive
int value = num;   // Compiler converts to: int value = num.intValue();

// In collections
List<Integer> numbers = new ArrayList<>();
numbers.add(5);        // Autoboxing: 5 → Integer.valueOf(5)
int first = numbers.get(0);  // Unboxing: Integer → int

// ⚠️ WATCH OUT: NullPointerException with unboxing
Integer nullValue = null;
int x = nullValue;  // NullPointerException! Can't unbox null
```

**Performance Consideration**:

```java
// ❌ BAD: Unnecessary autoboxing in loop
Integer sum = 0;
for (int i = 0; i < 1000; i++) {
    sum += i;  // Creates new Integer object on each iteration!
}

// ✅ GOOD: Use primitive for performance
int sum = 0;
for (int i = 0; i < 1000; i++) {
    sum += i;  // Fast primitive addition
}
```

#### String: The Special Reference Type

`String` is not a primitive but is so fundamental it deserves special attention.

**String Immutability**:

```java
String greeting = "Hello";
greeting.concat(" World");  // Creates new String, doesn't modify original
System.out.println(greeting);  // Still prints "Hello"

// To actually change the string:
greeting = greeting.concat(" World");  // Reassign the reference
System.out.println(greeting);  // Prints "Hello World"
```

**Why Strings are Immutable**:
* **Security**: Strings used for sensitive data (passwords, file paths)
* **Thread Safety**: Can be shared between threads safely
* **Performance**: String pooling optimization
* **Hashing**: Hash code can be cached (important for HashMap keys)

**String Pool**:

Java maintains a pool of String literals to save memory:

```java
String s1 = "Hello";  // Created in string pool
String s2 = "Hello";  // Reuses same object from pool
String s3 = new String("Hello");  // Creates new object on heap

System.out.println(s1 == s2);  // true (same object)
System.out.println(s1 == s3);  // false (different objects)
System.out.println(s1.equals(s3));  // true (same content)
```

**String vs StringBuilder vs StringBuffer**:

| Class | Mutability | Thread-Safe | Performance |
|-------|------------|-------------|-------------|
| `String` | Immutable | Yes (immutable) | Fast for read, slow for concatenation |
| `StringBuilder` | Mutable | No | Fastest for concatenation |
| `StringBuffer` | Mutable | Yes (synchronized) | Slower than StringBuilder |

```java
// ❌ BAD: String concatenation in loop (creates many objects)
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // Creates 1000 intermediate String objects!
}

// ✅ GOOD: StringBuilder for efficient concatenation
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();

// ✅ ALSO GOOD: Java 8+ uses StringBuilder automatically for simple cases
String result = "Value: " + x + ", Status: " + status;
// Compiler optimizes to StringBuilder internally
```

**When to Use Each**:
* **String**: When value won't change (most cases)
* **StringBuilder**: When building strings in loops or multiple operations (not thread-safe)
* **StringBuffer**: When building strings in multi-threaded context (rare, usually use StringBuilder and external synchronization)

### Type Casting

Java requires explicit casting when converting between types that could lose information:

**Widening Conversion (Automatic)**:

```java
// Smaller → Larger (no data loss, automatic)
byte b = 10;
int i = b;     // byte → int (automatic)
long l = i;    // int → long (automatic)
float f = l;   // long → float (automatic, might lose precision)
double d = f;  // float → double (automatic)
```

**Narrowing Conversion (Explicit Cast Required)**:

```java
// Larger → Smaller (potential data loss, must be explicit)
double d = 100.99;
int i = (int) d;  // 100 (fractional part lost)

long l = 130L;
byte b = (byte) l;  // 130 fits in byte

long big = 300L;
byte small = (byte) big;  // -44 (overflow, only lower 8 bits kept)
```

**Casting Between Reference Types** (covered more in OOP section):

```java
Object obj = "Hello";  // Upcasting (automatic)
String str = (String) obj;  // Downcasting (explicit, checked at runtime)
```

### Variables and Constants

#### Variable Declaration and Initialization

```java
// Declaration only
int count;

// Declaration with initialization
int age = 25;

// Multiple declarations
int x, y, z;
int a = 1, b = 2, c = 3;

// Type inference with var (Java 10+)
var message = "Hello";  // Inferred as String
var count = 10;         // Inferred as int
var list = new ArrayList<String>();  // Inferred as ArrayList<String>

// ❌ var requires initialization
var x;  // Compilation error! Cannot infer type
```

**Variable Types by Scope**:

```java
public class Example {
    // 1. Instance variables (non-static fields)
    private int instanceVar = 10;
    
    // 2. Class variables (static fields)
    private static int classVar = 20;
    
    public void method(int paramVar) {  // 3. Parameters
        // 4. Local variables
        int localVar = 30;
        
        System.out.println(instanceVar);  // Accessible
        System.out.println(classVar);     // Accessible
        System.out.println(paramVar);     // Accessible
        System.out.println(localVar);     // Accessible
    }
    
    // localVar is NOT accessible here
}
```

| Variable Type | Scope | Lifetime | Default Value |
|---------------|-------|----------|---------------|
| Local | Method/block | During method execution | None (must initialize) |
| Instance | Object | Until object is garbage collected | Type-specific defaults |
| Class (static) | Class | Until program ends | Type-specific defaults |
| Parameter | Method | During method execution | Value passed by caller |

#### Constants

Use `final` keyword to declare constants:

```java
// Local constant
final int MAX_SIZE = 100;
MAX_SIZE = 200;  // Compilation error!

// Class constant (naming convention: UPPER_SNAKE_CASE)
public static final int MAX_CONNECTIONS = 50;
public static final String APP_NAME = "MyApp";
public static final double PI = 3.14159;

// final with objects (reference is constant, not content)
final List<String> names = new ArrayList<>();
names.add("Alice");  // ✅ OK: modifying content
names = new ArrayList<>();  // ❌ Error: reassigning reference
```

### Variable Naming Conventions

Java has strict naming rules and conventions:

**Rules** (enforced by compiler):
* Can contain letters, digits, underscores, and dollar signs
* Must start with letter, underscore, or dollar sign
* Cannot be a Java keyword
* Case-sensitive

**Conventions** (community standards):

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | PascalCase | `UserAccount`, `ArrayList` |
| Interfaces | PascalCase | `Comparable`, `Serializable` |
| Methods | camelCase | `getName()`, `calculateTotal()` |
| Variables | camelCase | `firstName`, `totalAmount` |
| Constants | UPPER_SNAKE_CASE | `MAX_SIZE`, `PI` |
| Packages | lowercase | `com.company.project` |

**Examples**:

```java
// ✅ GOOD
public class CustomerAccount {
    private static final int MAX_LOGIN_ATTEMPTS = 3;
    private String userName;
    
    public void resetPassword() { }
}

// ❌ BAD (compiles but violates conventions)
public class customer_account {
    private static final int max_login_attempts = 3;
    private String UserName;
    
    public void ResetPassword() { }
}
```

### Operators

Java provides various operators for computations and comparisons.

#### Arithmetic Operators

```java
int a = 10, b = 3;

int sum = a + b;        // 13 (addition)
int diff = a - b;       // 7 (subtraction)
int product = a * b;    // 30 (multiplication)
int quotient = a / b;   // 3 (integer division)
int remainder = a % b;  // 1 (modulus)

// Integer division truncates
System.out.println(10 / 3);    // 3
System.out.println(10.0 / 3);  // 3.3333...

// Unary operators
int x = 5;
int y = -x;      // -5 (negation)
int z = +x;      // 5 (unary plus, rarely used)

// Increment/Decrement
int count = 5;
count++;  // count = 6 (post-increment)
++count;  // count = 7 (pre-increment)
count--;  // count = 6 (post-decrement)
--count;  // count = 5 (pre-decrement)

// Pre vs Post increment
int a = 5;
int b = a++;  // b = 5, then a = 6 (post: use then increment)
int c = ++a;  // a = 7, then c = 7 (pre: increment then use)
```

#### Relational Operators

```java
int x = 5, y = 10;

boolean equal = (x == y);      // false
boolean notEqual = (x != y);   // true
boolean greater = (x > y);     // false
boolean less = (x < y);        // true
boolean greaterEq = (x >= y);  // false
boolean lessEq = (x <= y);     // true

// ⚠️ COMMON MISTAKE: = vs ==
int a = 5;    // Assignment
if (a = 5) { }  // Compilation error! (not boolean)
if (a == 5) { } // ✅ Comparison
```

#### Logical Operators

```java
boolean a = true, b = false;

boolean and = a && b;   // false (logical AND)
boolean or = a || b;    // true (logical OR)
boolean not = !a;       // false (logical NOT)

// Short-circuit evaluation
boolean result = (5 > 3) || (10 / 0 == 2);  // true (doesn't evaluate 10/0)
boolean result2 = (5 < 3) && (10 / 0 == 2); // false (doesn't evaluate 10/0)

// Bitwise operators (work on bits, don't short-circuit)
boolean bitwiseAnd = a & b;   // false
boolean bitwiseOr = a | b;    // true
```

#### Bitwise Operators

```java
int a = 5;   // Binary: 0101
int b = 3;   // Binary: 0011

int bitwiseAnd = a & b;      // 1 (0001)
int bitwiseOr = a | b;       // 7 (0111)
int bitwiseXor = a ^ b;      // 6 (0110)
int bitwiseNot = ~a;         // -6 (inverts all bits)

int leftShift = a << 1;      // 10 (1010, multiply by 2)
int rightShift = a >> 1;     // 2 (0010, divide by 2)
int unsignedRight = a >>> 1; // 2 (unsigned right shift)

// Practical use: checking even/odd
boolean isEven = (num & 1) == 0;  // Faster than num % 2 == 0

// Setting/clearing bits
int flags = 0;
flags |= (1 << 2);   // Set bit 2
flags &= ~(1 << 2);  // Clear bit 2
boolean isSet = (flags & (1 << 2)) != 0;  // Check bit 2
```

#### Assignment Operators

```java
int x = 10;

x += 5;  // x = x + 5 → x = 15
x -= 3;  // x = x - 3 → x = 12
x *= 2;  // x = x * 2 → x = 24
x /= 4;  // x = x / 4 → x = 6
x %= 4;  // x = x % 4 → x = 2

// Bitwise compound assignments
x &= 3;  // x = x & 3
x |= 1;  // x = x | 1
x ^= 7;  // x = x ^ 7
x <<= 2; // x = x << 2
x >>= 1; // x = x >> 1
```

#### Ternary Operator

```java
// condition ? valueIfTrue : valueIfFalse

int age = 20;
String status = (age >= 18) ? "Adult" : "Minor";

// Nested ternary (avoid for readability)
String category = (age < 13) ? "Child" :
                  (age < 20) ? "Teenager" :
                  (age < 60) ? "Adult" : "Senior";

// ✅ BETTER: Use if-else for complex logic
String category;
if (age < 13) {
    category = "Child";
} else if (age < 20) {
    category = "Teenager";
} else if (age < 60) {
    category = "Adult";
} else {
    category = "Senior";
}
```

#### instanceof Operator

```java
String str = "Hello";
Object obj = str;

boolean isString = obj instanceof String;  // true
boolean isInteger = obj instanceof Integer; // false

// Prevents ClassCastException
if (obj instanceof String) {
    String s = (String) obj;  // Safe cast
}

// Pattern matching for instanceof (Java 16+)
if (obj instanceof String s) {
    // 's' is automatically cast and available here
    System.out.println(s.toUpperCase());
}
```

#### Operator Precedence

Understanding operator precedence prevents bugs:

| Precedence | Operators | Description |
|------------|-----------|-------------|
| 1 (highest) | `()`, `[]`, `.` | Parentheses, array access, member access |
| 2 | `++`, `--`, `!`, `~` | Unary operators |
| 3 | `*`, `/`, `%` | Multiplicative |
| 4 | `+`, `-` | Additive |
| 5 | `<<`, `>>`, `>>>` | Shift |
| 6 | `<`, `<=`, `>`, `>=`, `instanceof` | Relational |
| 7 | `==`, `!=` | Equality |
| 8 | `&` | Bitwise AND |
| 9 | `^` | Bitwise XOR |
| 10 | `|` | Bitwise OR |
| 11 | `&&` | Logical AND |
| 12 | `||` | Logical OR |
| 13 | `?:` | Ternary |
| 14 (lowest) | `=`, `+=`, `-=`, etc. | Assignment |

```java
// Examples showing precedence
int result = 2 + 3 * 4;      // 14, not 20 (* before +)
boolean b = 5 > 3 && 2 < 4;  // true (> before &&)

// Use parentheses for clarity
int result = (2 + 3) * 4;    // 20 (explicit grouping)
boolean b = (5 > 3) && (2 < 4);  // Same result, clearer intent
```

### Control Flow Statements

Control flow structures determine the execution path of your program.

#### if-else Statements

```java
// Basic if
if (condition) {
    // Execute if condition is true
}

// if-else
if (condition) {
    // Execute if true
} else {
    // Execute if false
}

// if-else-if ladder
if (condition1) {
    // Execute if condition1 is true
} else if (condition2) {
    // Execute if condition2 is true
} else if (condition3) {
    // Execute if condition3 is true
} else {
    // Execute if all conditions are false
}

// Example: Grade calculator
int score = 85;
String grade;

if (score >= 90) {
    grade = "A";
} else if (score >= 80) {
    grade = "B";
} else if (score >= 70) {
    grade = "C";
} else if (score >= 60) {
    grade = "D";
} else {
    grade = "F";
}
```

**Best Practices**:

```java
// ❌ BAD: Missing braces (error-prone)
if (condition)
    statement1;
    statement2;  // NOT part of if! Always executes

// ✅ GOOD: Always use braces
if (condition) {
    statement1;
    statement2;
}

// ❌ BAD: Unnecessary nesting
if (condition1) {
    if (condition2) {
        if (condition3) {
            // Deep nesting
        }
    }
}

// ✅ GOOD: Early returns to reduce nesting
if (!condition1) return;
if (!condition2) return;
if (!condition3) return;
// Happy path code here
```

#### switch Statements and Expressions

**Traditional Switch (Before Java 12)**:

```java
int day = 3;
String dayName;

switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    case 4:
        dayName = "Thursday";
        break;
    case 5:
        dayName = "Friday";
        break;
    case 6:
    case 7:
        dayName = "Weekend";
        break;
    default:
        dayName = "Invalid day";
}

// ⚠️ Common bug: Forgetting break
switch (value) {
    case 1:
        System.out.println("One");
        // Missing break! Falls through to case 2
    case 2:
        System.out.println("Two");
        break;
}
```

**Enhanced Switch Expressions (Java 12+, Standard in Java 14)**:

```java
// Arrow syntax (no fall-through, no break needed)
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4 -> "Thursday";
    case 5 -> "Friday";
    case 6, 7 -> "Weekend";  // Multiple case labels
    default -> "Invalid day";
};

// Block syntax for complex logic
String result = switch (day) {
    case 1, 2, 3, 4, 5 -> {
        System.out.println("Weekday");
        yield "Work day";  // yield returns value from block
    }
    case 6, 7 -> {
        System.out.println("Weekend");
        yield "Rest day";
    }
    default -> "Invalid";
};

// Works with Strings, enums, primitives
String fruit = "apple";
String color = switch (fruit) {
    case "apple" -> "red";
    case "banana" -> "yellow";
    case "grape" -> "purple";
    default -> "unknown";
};
```

**Pattern Matching for Switch (Java 17+ Preview, Java 21 Standard)**:

```java
// Type patterns
Object obj = "Hello";
String formatted = switch (obj) {
    case Integer i -> String.format("int %d", i);
    case Long l -> String.format("long %d", l);
    case Double d -> String.format("double %f", d);
    case String s -> String.format("String %s", s);
    default -> obj.toString();
};

// Guarded patterns (Java 21+)
String classify = switch (obj) {
    case String s when s.length() > 5 -> "Long string";
    case String s -> "Short string";
    case Integer i when i > 0 -> "Positive number";
    case Integer i -> "Non-positive number";
    default -> "Other";
};
```

#### for Loops

**Traditional for Loop**:

```java
// Basic syntax
for (initialization; condition; update) {
    // Loop body
}

// Example
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// Multiple variables
for (int i = 0, j = 10; i < j; i++, j--) {
    System.out.println(i + " " + j);
}

// Infinite loop
for (;;) {
    // Runs forever (until break)
}

// All parts are optional
int i = 0;
for (; i < 10; ) {
    System.out.println(i);
    i++;
}
```

**Enhanced for Loop (for-each)** (Java 5+):

```java
// Syntax
for (Type variable : collection) {
    // Loop body
}

// Examples
int[] numbers = {1, 2, 3, 4, 5};
for (int num : numbers) {
    System.out.println(num);
}

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
for (String name : names) {
    System.out.println(name);
}

// ⚠️ Cannot modify collection while iterating (for-each)
List<String> items = new ArrayList<>(Arrays.asList("a", "b", "c"));
for (String item : items) {
    items.remove(item);  // ConcurrentModificationException!
}

// ✅ Use iterator for modification
Iterator<String> iterator = items.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    if (shouldRemove(item)) {
        iterator.remove();  // Safe removal
    }
}
```

#### while and do-while Loops

**while Loop**:

```java
// Syntax
while (condition) {
    // Loop body
}

// Example
int count = 0;
while (count < 5) {
    System.out.println(count);
    count++;
}

// May not execute at all
int x = 10;
while (x < 5) {
    System.out.println("Never prints");
}
```

**do-while Loop**:

```java
// Syntax
do {
    // Loop body
} while (condition);

// Example
int count = 0;
do {
    System.out.println(count);
    count++;
} while (count < 5);

// Executes at least once
int x = 10;
do {
    System.out.println("Prints once");
} while (x < 5);

// Practical use: input validation
Scanner scanner = new Scanner(System.in);
int number;
do {
    System.out.print("Enter a positive number: ");
    number = scanner.nextInt();
} while (number <= 0);
```

#### break and continue

**break Statement**:

```java
// Exit loop immediately
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;  // Exit loop when i is 5
    }
    System.out.println(i);  // Prints 0, 1, 2, 3, 4
}

// Breaking from nested loops with labels
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) {
            break outer;  // Breaks from outer loop
        }
        System.out.println(i + "," + j);
    }
}

// Breaking from switch (already covered)
```

**continue Statement**:

```java
// Skip current iteration, continue with next
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue;  // Skip even numbers
    }
    System.out.println(i);  // Prints 1, 3, 5, 7, 9
}

// With labels
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            continue outer;  // Continue outer loop
        }
        System.out.println(i + "," + j);
    }
}
```

### Arrays

Arrays are fixed-size, homogeneous collections of elements.

#### Array Declaration and Initialization

```java
// Declaration (creates reference, not array)
int[] numbers;        // Preferred style
int numbers2[];       // C-style (valid but not preferred)

// Initialization
numbers = new int[5];  // Array of 5 integers, initialized to 0

// Declaration + initialization
int[] nums = new int[5];

// Array literal (initialization at declaration)
int[] values = {1, 2, 3, 4, 5};

// Explicit initialization with new
int[] scores = new int[]{10, 20, 30, 40, 50};

// Getting array length
int length = values.length;  // 5 (field, not method!)

// Accessing elements (0-indexed)
int first = values[0];  // 1
values[2] = 100;        // Modify element

// ⚠️ ArrayIndexOutOfBoundsException
int[] arr = new int[3];
int x = arr[3];  // Runtime exception! Valid indices: 0, 1, 2
```

#### Multi-Dimensional Arrays

```java
// 2D array (rectangular)
int[][] matrix = new int[3][4];  // 3 rows, 4 columns

// Initialization with values
int[][] grid = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Accessing elements
int element = grid[1][2];  // 6 (row 1, column 2)

// Iterating 2D array
for (int i = 0; i < grid.length; i++) {
    for (int j = 0; j < grid[i].length; j++) {
        System.out.print(grid[i][j] + " ");
    }
    System.out.println();
}

// Enhanced for with 2D arrays
for (int[] row : grid) {
    for (int value : row) {
        System.out.print(value + " ");
    }
    System.out.println();
}

// Jagged arrays (rows of different lengths)
int[][] jagged = new int[3][];
jagged[0] = new int[2];
jagged[1] = new int[4];
jagged[2] = new int[3];

int[][] jaggedInit = {
    {1, 2},
    {3, 4, 5, 6},
    {7, 8, 9}
};
```

#### Array Operations and Common Pitfalls

```java
// Copying arrays

// ❌ BAD: Assigns reference, not a copy
int[] original = {1, 2, 3};
int[] copy = original;  // Both point to same array!
copy[0] = 100;
System.out.println(original[0]);  // 100 (modified!)

// ✅ GOOD: Actual copy methods

// Method 1: Arrays.copyOf
int[] copy1 = Arrays.copyOf(original, original.length);

// Method 2: System.arraycopy
int[] copy2 = new int[original.length];
System.arraycopy(original, 0, copy2, 0, original.length);

// Method 3: clone()
int[] copy3 = original.clone();

// Method 4: Streams (Java 8+)
int[] copy4 = Arrays.stream(original).toArray();

// Comparing arrays
int[] arr1 = {1, 2, 3};
int[] arr2 = {1, 2, 3};

System.out.println(arr1 == arr2);           // false (different objects)
System.out.println(Arrays.equals(arr1, arr2));  // true (same content)

// Sorting
int[] numbers = {5, 2, 8, 1, 9};
Arrays.sort(numbers);  // {1, 2, 5, 8, 9}

// Searching (array must be sorted)
int index = Arrays.binarySearch(numbers, 5);  // 2

// Filling
int[] filled = new int[5];
Arrays.fill(filled, 10);  // {10, 10, 10, 10, 10}

// Converting to String
System.out.println(Arrays.toString(numbers));  // [1, 2, 5, 8, 9]
```

#### Varargs (Variable Arguments)

```java
// Method accepting variable number of arguments
public static int sum(int... numbers) {
    // numbers is treated as an array
    int total = 0;
    for (int num : numbers) {
        total += num;
    }
    return total;
}

// Calling with different number of arguments
sum();              // 0
sum(5);             // 5
sum(1, 2, 3);       // 6
sum(1, 2, 3, 4, 5); // 15

// Can also pass an array
int[] values = {10, 20, 30};
sum(values);        // 60

// Varargs rules
// 1. Must be last parameter
public void method(String name, int... numbers) { }  // ✅ OK
public void method(int... numbers, String name) { }  // ❌ Error

// 2. Only one varargs per method
public void method(int... nums1, int... nums2) { }   // ❌ Error
```

### Key Takeaways

* **Java Platform**: Write Once, Run Anywhere through bytecode and JVM
* **OpenJDK**: Free, open-source reference implementation of Java
* **Primitive Types**: 8 types for efficiency (byte, short, int, long, float, double, char, boolean)
* **Wrapper Classes**: Object versions of primitives, with autoboxing/unboxing
* **String Immutability**: Strings can't be changed; use StringBuilder for concatenation
* **Type Safety**: Java is strongly typed; explicit casting required for narrowing
* **Control Flow**: if-else, switch (now expressions), for, while, do-while, break, continue
* **Arrays**: Fixed-size, homogeneous collections with 0-based indexing
* **Modern Features**: var (type inference), switch expressions, pattern matching

### Frequently Asked Questions

**Q1: What's the difference between JDK, JRE, and JVM?**

**A:** 
* **JVM (Java Virtual Machine)**: The runtime engine that executes bytecode. Platform-specific.
* **JRE (Java Runtime Environment)**: JVM + core libraries. Everything needed to run Java applications.
* **JDK (Java Development Kit)**: JRE + development tools (javac, debugger, etc.). Everything needed to develop Java applications.

As a developer, you install the JDK. Users only need the JRE (though since Java 11, Oracle doesn't provide standalone JRE downloads).

---

**Q2: Why are Strings immutable in Java?**

**A:** Strings are immutable for several critical reasons:

1. **Security**: Strings hold sensitive data (passwords, file paths, network connections). Immutability prevents malicious modification.
2. **Thread Safety**: Multiple threads can safely share String objects without synchronization.
3. **String Pool**: JVM can optimize memory by reusing String literals because they can't change.
4. **Hash Code Caching**: String's hash code is cached (important for HashMap keys), which wouldn't be safe if Strings were mutable.

```java
String s1 = "Hello";
String s2 = "Hello";  // Reuses s1's object from pool
// If strings were mutable, changing s1 would affect s2!
```

---

**Q3: When should I use StringBuilder vs StringBuffer?**

**A:** 

* **StringBuilder**: Use in single-threaded scenarios (99% of cases). Faster because it's not synchronized.
* **StringBuffer**: Use only in multi-threaded scenarios where multiple threads modify the same instance (rare).

```java
// Single-threaded (normal case)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);  // Fast
}

// Multi-threaded (rare)
StringBuffer buffer = new StringBuffer();
// Multiple threads safely append to buffer
```

In modern code, you'd typically use StringBuilder with external synchronization if thread safety is needed, rather than StringBuffer.

---

**Q4: What's the difference between == and equals() for Strings?**

**A:**

* **==**: Compares object references (memory addresses)
* **equals()**: Compares object content

```java
String s1 = "Hello";
String s2 = "Hello";  // String pool
String s3 = new String("Hello");  // Heap

System.out.println(s1 == s2);       // true (same object in pool)
System.out.println(s1 == s3);       // false (different objects)
System.out.println(s1.equals(s3));  // true (same content)
```

**Rule**: Always use `equals()` for String comparison, unless you specifically need to check if two references point to the same object.

---

**Q5: What happens if I forget 'break' in a switch statement?**

**A:** You get "fall-through" behavior, where execution continues into the next case:

```java
int day = 1;
switch (day) {
    case 1:
        System.out.println("Monday");
        // Forgot break!
    case 2:
        System.out.println("Tuesday");
        break;
}
// Prints both "Monday" and "Tuesday"
```

This is usually a bug, but can be intentional for multiple cases with same behavior:

```java
switch (day) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        System.out.println("Weekday");
        break;
    case 6:
    case 7:
        System.out.println("Weekend");
        break;
}
```

**Solution**: Use enhanced switch expressions (Java 14+) which don't have fall-through:

```java
String type = switch (day) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7 -> "Weekend";
    default -> "Invalid";
};
```

### Interview Questions

**Question 1: Explain the difference between pass-by-value and pass-by-reference. How does Java handle parameter passing?**

**Difficulty:** Mid-Level

**Topics:** Language Fundamentals, Memory Model

**Answer:**

Java is **strictly pass-by-value**. This confuses many developers because object references are passed, but the reference itself is passed by value.

**For Primitives:**
```java
public void modify(int x) {
    x = 100;  // Only modifies local copy
}

int num = 5;
modify(num);
System.out.println(num);  // Still 5
```

**For Objects:**
```java
public void modify(StringBuilder sb) {
    sb.append(" World");  // Modifies object (reference points to)
}

public void reassign(StringBuilder sb) {
    sb = new StringBuilder("New");  // Only changes local reference
}

StringBuilder str = new StringBuilder("Hello");
modify(str);
System.out.println(str);  // "Hello World" (object modified)

reassign(str);
System.out.println(str);  // Still "Hello World" (reference unchanged)
```

**Why This Matters:** Understanding this prevents bugs where you expect a method to change a reference variable but it doesn't. You can modify an object's internal state, but you cannot make a variable point to a different object by passing it to a method.

**Follow-up Questions:**
* How would you design a method to "return" multiple values in Java?
* What's the difference between Integer object and int primitive when passed to a method?
* Can you modify an array inside a method? Why or why not?

---

**Question 2: What is autoboxing and unboxing? What are the performance implications?**

**Difficulty:** Mid-Level

**Topics:** Primitives, Wrapper Classes, Performance

**Answer:**

**Autoboxing**: Automatic conversion from primitive to wrapper object
**Unboxing**: Automatic conversion from wrapper object to primitive

```java
// Autoboxing
Integer num = 5;  // Compiler: Integer.valueOf(5)

// Unboxing
int value = num;  // Compiler: num.intValue()

// In collections
List<Integer> numbers = new ArrayList<>();
numbers.add(5);  // Autoboxing: 5 → Integer.valueOf(5)
int first = numbers.get(0);  // Unboxing: Integer → int
```

**Performance Implications:**

1. **Object Creation Overhead:**
```java
// ❌ BAD: Creates 100,000 Integer objects
Integer sum = 0;
for (int i = 0; i < 100000; i++) {
    sum += i;  // Unbox → add → autobox (new object each time!)
}

// ✅ GOOD: Uses primitive
int sum = 0;
for (int i = 0; i < 100000; i++) {
    sum += i;  // Fast primitive operation
}
```

2. **NullPointerException Risk:**
```java
Integer count = null;
int value = count;  // NullPointerException during unboxing!
```

3. **Caching (Performance Optimization):**
```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);  // true (cached)

Integer c = 1000;
Integer d = 1000;
System.out.println(c == d);  // false (not cached)
```

Java caches Integer objects from -128 to 127 for performance.

**Why This Matters:** In high-performance code or loops processing millions of items, unnecessary boxing/unboxing can significantly impact performance. Always use primitives for performance-critical code.

**Follow-up Questions:**
* What's the range of cached Integer values and why?
* How would you optimize a method that processes millions of numbers?
* When is it appropriate to use Integer instead of int?

**Red Flags in Answers:**
* Claiming Java is pass-by-reference for objects
* Not understanding the performance impact of autoboxing in loops
* Confusion about == vs equals() with wrapper classes

---

**Question 3: Explain String pool. How does it work, and what are the implications?**

**Difficulty:** Mid-Level

**Topics:** String, Memory Management

**Answer:**

The **String pool** (or String intern pool) is a special memory region where Java stores String literals to optimize memory usage through string interning.

**How It Works:**

```java
String s1 = "Hello";  // Created in String pool
String s2 = "Hello";  // Reuses same object from pool
String s3 = new String("Hello");  // Creates new object on heap

System.out.println(s1 == s2);  // true (same reference)
System.out.println(s1 == s3);  // false (different objects)
System.out.println(s1.equals(s3));  // true (same content)

// Manually intern a String
String s4 = s3.intern();  // Returns reference from pool
System.out.println(s1 == s4);  // true (now same reference)
```

**Memory Diagram:**

```
String Pool (PermGen/Metaspace):
  ┌─────────┐
  │ "Hello" │ ← s1, s2, s4
  └─────────┘

Heap:
  ┌─────────┐
  │ "Hello" │ ← s3
  └─────────┘
```

**Implications:**

1. **Memory Efficiency:** Duplicate literals share same object
2. **Thread Safety:** String literals are safely shared between threads
3. **Performance:** String comparison with == works for literals
4. **Security:** Immutability combined with pooling prevents tampering

**Why This Matters:** Understanding the String pool explains seemingly odd behavior with == comparisons and guides best practices like using equals() for content comparison.

**Follow-up Questions:**
* Where is the String pool located in different Java versions?
* What happens to the String pool during garbage collection?
* When would you use intern() explicitly?

---

This completes the first major section. Due to the length requirements (15,000-25,000 words), I'll continue with the remaining sections in the next part of this file.

---

## 1.3 Object-Oriented Programming (OOP) Fundamentals

Object-Oriented Programming is the cornerstone of Java. Understanding OOP deeply is essential for writing maintainable, scalable Java applications.

### Classes and Objects

A **class** is a blueprint or template that defines the structure and behavior of objects. An **object** is an instance of a class—a concrete entity with state and behavior.

**Fundamental Concept:**
```
Class (Blueprint)          →    Object (Instance)
Car                        →    myCar
  - properties: color           - color: "red"
               model                     model: "Tesla"
               year                      year: 2024
  - behaviors: start()          - can: start()
               stop()                   stop()
               drive()                  drive()
```

#### Class Definition Anatomy

```java
// Complete class structure
package com.company.vehicle;  // Package declaration

import java.time.LocalDate;   // Imports

/**
 * Represents a car with basic properties and behaviors.
 * 
 * @author Your Name
 * @version 1.0
 * @since 2024-01-15
 */
public class Car {  // Class declaration
    
    // 1. Instance variables (state)
    private String model;
    private String color;
    private int year;
    private double mileage;
    
    // 2. Class variables (shared across all instances)
    private static int carCount = 0;
    
    // 3. Constants
    private static final int MAX_SPEED = 200;
    
    // 4. Instance initialization block (rarely used)
    {
        System.out.println("Instance initialization block");
    }
    
    // 5. Static initialization block
    static {
        System.out.println("Static initialization block");
    }
    
    // 6. Constructors
    public Car() {
        this("Unknown", "Unknown", 2024);
    }
    
    public Car(String model, String color, int year) {
        this.model = model;
        this.color = color;
        this.year = year;
        this.mileage = 0.0;
        carCount++;
    }
    
    // 7. Instance methods (behavior)
    public void start() {
        System.out.println(model + " is starting...");
    }
    
    public void drive(double miles) {
        mileage += miles;
        System.out.println("Drove " + miles + " miles");
    }
    
    // 8. Getters and setters
    public String getModel() {
        return model;
    }
    
    public void setColor(String color) {
        this.color = color;
    }
    
    // 9. Static methods
    public static int getCarCount() {
        return carCount;
    }
    
    // 10. Override Object methods
    @Override
    public String toString() {
        return String.format("Car{model='%s', color='%s', year=%d, mileage=%.1f}",
                             model, color, year, mileage);
    }
}
```

#### Object Creation and Memory Allocation

```java
// Creating objects
Car car1 = new Car();  // Default constructor
Car car2 = new Car("Tesla Model 3", "White", 2024);  // Parameterized constructor

// What happens in memory:
/*
Stack:                    Heap:
┌──────┐                 ┌────────────────────────┐
│ car1 │ ───────────────>│ Car Object             │
└──────┘                 │  model: "Unknown"      │
                          │  color: "Unknown"      │
┌──────┐                 │  year: 2024            │
│ car2 │ ───────────┐    │  mileage: 0.0          │
└──────┘            │    └────────────────────────┘
                     │                             
                     └────>┌────────────────────────┐
                           │ Car Object             │
                           │  model: "Tesla Model 3"│
                           │  color: "White"        │
                           │  year: 2024            │
                           │  mileage: 0.0          │
                           └────────────────────────┘
*/
```

**Object Lifecycle:**

1. **Creation**: 
   ```java
   Car car = new Car();
   // 1. Memory allocated on heap
   // 2. Instance variables initialized to defaults
   // 3. Constructor called
   // 4. Reference returned and assigned to 'car'
   ```

2. **Usage**:
   ```java
   car.start();
   car.drive(100.5);
   ```

3. **Garbage Collection**:
   ```java
   car = null;  // Object eligible for GC
   // OR
   // car goes out of scope
   // JVM's Garbage Collector will reclaim memory
   ```

#### Constructors

Constructors are special methods called when creating objects.

**Constructor Rules:**
* Same name as class
* No return type (not even void)
* Can be overloaded
* If no constructor is defined, Java provides default no-arg constructor
* If any constructor is defined, default constructor is NOT provided

```java
public class Person {
    private String name;
    private int age;
    
    // Default constructor
    public Person() {
        this.name = "Unknown";
        this.age = 0;
    }
    
    // Parameterized constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Copy constructor
    public Person(Person other) {
        this.name = other.name;
        this.age = other.age;
    }
}

// Usage
Person p1 = new Person();                    // Default
Person p2 = new Person("Alice", 30);        // Parameterized
Person p3 = new Person(p2);                 // Copy
```

**Constructor Chaining** with `this()`:

```java
public class Employee {
    private String name;
    private String department;
    private double salary;
    
    public Employee() {
        this("Unknown", "General", 0.0);  // Calls 3-param constructor
    }
    
    public Employee(String name) {
        this(name, "General", 0.0);  // Calls 3-param constructor
    }
    
    public Employee(String name, String department) {
        this(name, department, 0.0);  // Calls 3-param constructor
    }
    
    public Employee(String name, String department, double salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }
}

// ⚠️ Rules for this():
// 1. Must be first statement in constructor
// 2. Cannot use this() and super() in same constructor
// 3. Prevents cyclic constructor calls (compiler error)
```

#### Instance Variables and Methods

```java
public class BankAccount {
    // Instance variables (each object has its own copy)
    private String accountNumber;
    private double balance;
    private String owner;
    
    // Instance methods (operate on instance variables)
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: $" + amount);
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
            return true;
        }
        return false;
    }
    
    public double getBalance() {
        return balance;
    }
}

// Each object has its own state
BankAccount acc1 = new BankAccount("12345", "Alice");
BankAccount acc2 = new BankAccount("67890", "Bob");

acc1.deposit(1000);  // Only affects acc1
acc2.deposit(500);   // Only affects acc2
```

#### The `this` Keyword

`this` is a reference to the current object.

**Uses of `this`:**

```java
public class Student {
    private String name;
    private int id;
    
    // 1. Distinguish instance variables from parameters
    public Student(String name, int id) {
        this.name = name;  // this.name is instance variable
        this.id = id;      // name and id are parameters
    }
    
    // 2. Call another constructor
    public Student() {
        this("Unknown", 0);  // Calls parameterized constructor
    }
    
    // 3. Pass current object as parameter
    public void registerForCourse(Course course) {
        course.addStudent(this);  // Pass this Student object
    }
    
    // 4. Return current object (method chaining)
    public Student setName(String name) {
        this.name = name;
        return this;  // Enable chaining
    }
    
    public Student setId(int id) {
        this.id = id;
        return this;  // Enable chaining
    }
}

// Method chaining
Student student = new Student()
    .setName("Alice")
    .setId(12345);
```

### Encapsulation

**Encapsulation** is the bundling of data and methods that operate on that data within a single unit (class), and restricting access to internal details.

**Why Encapsulation:**
* **Data Hiding**: Protect internal state from external modification
* **Flexibility**: Change internal implementation without affecting users
* **Validation**: Control how data is accessed and modified
* **Maintainability**: Clear interface reduces coupling

#### Access Modifiers

Java provides four access levels:

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| default (no modifier) | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

```java
public class AccessExample {
    public int publicVar = 1;        // Accessible everywhere
    protected int protectedVar = 2;  // Accessible in package + subclasses
    int defaultVar = 3;              // Package-private (no modifier)
    private int privateVar = 4;      // Only within this class
    
    public void publicMethod() { }
    protected void protectedMethod() { }
    void defaultMethod() { }
    private void privateMethod() { }
}
```

**Best Practice**: Make everything private unless there's a good reason for it to be more accessible. This is the principle of **least privilege**.

#### Getters and Setters

```java
public class User {
    private String username;  // Private field
    private String email;
    private int age;
    
    // Getter (accessor)
    public String getUsername() {
        return username;
    }
    
    // Setter (mutator) with validation
    public void setUsername(String username) {
        if (username != null && !username.trim().isEmpty()) {
            this.username = username;
        } else {
            throw new IllegalArgumentException("Username cannot be empty");
        }
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        // Validation: simple email check
        if (email != null && email.contains("@")) {
            this.email = email;
        } else {
            throw new IllegalArgumentException("Invalid email");
        }
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        // Validation: age must be reasonable
        if (age >= 0 && age <= 150) {
            this.age = age;
        } else {
            throw new IllegalArgumentException("Invalid age");
        }
    }
}

// Usage
User user = new User();
user.setUsername("alice");
user.setEmail("alice@example.com");
user.setAge(25);

// Attempting invalid values
user.setAge(-5);  // IllegalArgumentException
user.setEmail("invalid");  // IllegalArgumentException
```

**Getter/Setter Best Practices:**

```java
// ❌ BAD: Exposing mutable internal state
public class BadExample {
    private List<String> items = new ArrayList<>();
    
    public List<String> getItems() {
        return items;  // Caller can modify the list!
    }
}

// Usage:
BadExample bad = new BadExample();
bad.getItems().clear();  // Oops! Modified internal state

// ✅ GOOD: Return defensive copy
public class GoodExample {
    private List<String> items = new ArrayList<>();
    
    public List<String> getItems() {
        return new ArrayList<>(items);  // Return copy
    }
    
    // Or return unmodifiable view
    public List<String> getItemsUnmodifiable() {
        return Collections.unmodifiableList(items);
    }
}
```

#### JavaBeans Conventions

JavaBeans are reusable software components that follow specific conventions:

```java
/**
 * JavaBean example
 */
public class PersonBean implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String name;
    private int age;
    
    // 1. Public no-argument constructor (required)
    public PersonBean() {
    }
    
    // 2. Getters and setters (properties)
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    // 3. Implements Serializable (for persistence)
}
```

**JavaBeans are used extensively in:**
* JSP and Servlets
* Java Enterprise Edition (EE)
* Spring Framework
* JavaFX properties

### Inheritance

**Inheritance** allows a class to acquire properties and methods from another class, promoting code reuse and establishing hierarchical relationships.

**Key Terminology:**
* **Superclass** (Parent/Base class): The class being inherited from
* **Subclass** (Child/Derived class): The class that inherits
* **IS-A relationship**: Inheritance represents an "is-a" relationship (Dog IS-A Animal)

#### Basic Inheritance Syntax

```java
// Superclass
public class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

// Subclass
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    // Override parent method
    @Override
    public void makeSound() {
        System.out.println(name + " barks: Woof! Woof!");
    }
    
    // Add new method specific to Dog
    public void fetch() {
        System.out.println(name + " is fetching the ball");
    }
}

// Usage
Dog dog = new Dog("Max", 3, "Labrador");
dog.eat();        // Inherited from Animal
dog.sleep();      // Inherited from Animal
dog.makeSound();  // Overridden in Dog
dog.fetch();      // Specific to Dog
```

**Memory Representation:**

```
Dog object in memory:
┌──────────────────────────┐
│ Animal part:             │
│   name: "Max"            │
│   age: 3                 │
│   eat()                  │
│   sleep()                │
│   makeSound() [overridden]
├──────────────────────────┤
│ Dog part:                │
│   breed: "Labrador"      │
│   fetch()                │
│   makeSound() [override] │
└──────────────────────────┘
```

#### The `super` Keyword

`super` refers to the parent class and is used for:

1. **Calling Parent Constructor:**

```java
public class Vehicle {
    protected String brand;
    
    public Vehicle(String brand) {
        this.brand = brand;
    }
}

public class Car extends Vehicle {
    private int doors;
    
    public Car(String brand, int doors) {
        super(brand);  // Must be first statement
        this.doors = doors;
    }
    
    // ❌ ERROR: super() not first
    public Car(String brand, int doors) {
        this.doors = doors;
        super(brand);  // Compilation error!
    }
}
```

2. **Accessing Parent Methods:**

```java
public class Parent {
    public void display() {
        System.out.println("Parent display");
    }
}

public class Child extends Parent {
    @Override
    public void display() {
        super.display();  // Call parent's display()
        System.out.println("Child display");
    }
}

// Output:
// Parent display
// Child display
```

3. **Accessing Parent Fields:**

```java
public class Parent {
    protected int value = 10;
}

public class Child extends Parent {
    private int value = 20;  // Hides parent's value
    
    public void printValues() {
        System.out.println("Child value: " + value);        // 20
        System.out.println("Parent value: " + super.value); // 10
    }
}
```

#### Method Overriding

**Overriding** is providing a specific implementation of a method that's already defined in the parent class.

**Rules for Overriding:**
1. Method signature must be identical (name, parameters, return type)
2. Access modifier cannot be more restrictive
3. Cannot override `final` methods
4. Cannot override `static` methods (hiding, not overriding)
5. Exception thrown cannot be broader than parent's

```java
public class Shape {
    // Method to be overridden
    public double calculateArea() {
        return 0.0;
    }
    
    // final method cannot be overridden
    public final String getType() {
        return "Shape";
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    // ✅ Valid override
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
    
    // ❌ ERROR: Cannot override final method
    @Override
    public String getType() {  // Compilation error!
        return "Circle";
    }
}
```

**Access Modifier Rules:**

```java
public class Parent {
    protected void method1() { }
    public void method2() { }
}

public class Child extends Parent {
    // ✅ OK: Same or more accessible
    @Override
    public void method1() { }
    
    // ❌ ERROR: More restrictive
    @Override
    protected void method2() { }  // Compilation error!
}
```

**Covariant Return Types** (Java 5+):

```java
public class Animal {
    public Animal reproduce() {
        return new Animal();
    }
}

public class Dog extends Animal {
    // ✅ OK: Return type is subclass of parent's return type
    @Override
    public Dog reproduce() {
        return new Dog();
    }
}
```

#### @Override Annotation

Always use `@Override` to catch mistakes at compile time:

```java
public class Parent {
    public void process(String data) {
        System.out.println("Processing: " + data);
    }
}

public class Child extends Parent {
    // ❌ Typo in method name - without @Override, no error!
    public void proces(String data) {  // Different method, doesn't override
        System.out.println("Child processing: " + data);
    }
    
    // ✅ With @Override, compiler catches the error
    @Override
    public void proces(String data) {  // Compilation error: method doesn't override
        System.out.println("Child processing: " + data);
    }
}
```

#### Object Class: The Root of All

Every class in Java implicitly extends `Object` class.

**Important Object Methods:**

```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // 1. toString() - String representation
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
    
    // 2. equals() - Logical equality
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && 
               Objects.equals(name, person.name);
    }
    
    // 3. hashCode() - Hash code for hash-based collections
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    // 4. clone() - Create a copy (requires Cloneable interface)
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();  // Shallow copy
    }
}

// Usage
Person p1 = new Person("Alice", 30);
Person p2 = new Person("Alice", 30);

System.out.println(p1.toString());     // Person{name='Alice', age=30}
System.out.println(p1.equals(p2));     // true (same content)
System.out.println(p1 == p2);          // false (different objects)
System.out.println(p1.hashCode());     // Consistent with equals()
```

**Critical Contract: equals() and hashCode():**

> If two objects are equal according to equals(), they MUST have the same hashCode().

```java
// ❌ BAD: Breaks HashMap, HashSet, etc.
@Override
public boolean equals(Object obj) {
    // Implementation
}
// Missing hashCode() override!

// ✅ GOOD: Both overridden consistently
@Override
public boolean equals(Object obj) {
    // Implementation
}

@Override
public int hashCode() {
    // Implementation consistent with equals()
}
```

#### final Keyword with Inheritance

**final class**: Cannot be extended

```java
public final class ImmutableValue {
    private final int value;
    
    public ImmutableValue(int value) {
        this.value = value;
    }
}

// ❌ ERROR: Cannot extend final class
public class ExtendedValue extends ImmutableValue { }  // Compilation error!
```

**final method**: Cannot be overridden

```java
public class Parent {
    public final void criticalOperation() {
        // This implementation must not be changed
    }
}

public class Child extends Parent {
    // ❌ ERROR: Cannot override final method
    @Override
    public void criticalOperation() { }  // Compilation error!
}
```

**Why use final?**
* **Security**: Prevent modification of critical behavior
* **Performance**: JVM can inline final methods (minor optimization)
* **Design**: Clearly communicate intent that class/method shouldn't be extended/overridden

**Examples in Java:**
* `String` is final (immutability guarantee)
* `Integer`, `Double`, etc. are final
* Many methods in `Object` are final

#### Single Inheritance Limitation

Java supports **single inheritance** only (one direct parent class).

```java
public class A { }
public class B { }

// ❌ ERROR: Cannot extend multiple classes
public class C extends A, B { }  // Compilation error!

// ✅ OK: Can implement multiple interfaces (covered next)
public class C extends A implements InterfaceX, InterfaceY { }
```

**Why Single Inheritance?**
* Avoids the **Diamond Problem**:

```
     A
    / \
   B   C
    \ /
     D
     
If D inherits from both B and C, and both override a method from A,
which implementation does D get? Ambiguous!
```

**Solution**: Use interfaces for multiple inheritance of type (not implementation).

### Polymorphism

**Polymorphism** (Greek: "many forms") means an object can take many forms. In Java, this manifests as:

1. **Compile-time Polymorphism** (Static): Method overloading
2. **Runtime Polymorphism** (Dynamic): Method overriding

#### Compile-Time Polymorphism (Method Overloading)

**Method Overloading**: Multiple methods with same name but different parameters.

```java
public class Calculator {
    // Same method name, different parameters
    
    // 1. Different number of parameters
    public int add(int a, int b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    // 2. Different parameter types
    public double add(double a, double b) {
        return a + b;
    }
    
    // 3. Different parameter order (usually avoided for clarity)
    public void display(String text, int number) {
        System.out.println(text + ": " + number);
    }
    
    public void display(int number, String text) {
        System.out.println(number + ": " + text);
    }
    
    // ❌ NOT VALID: Different return type alone doesn't work
    public int add(int a, int b) { return a + b; }
    public double add(int a, int b) { return a + b; }  // Error!
}

// Compiler chooses method based on arguments
Calculator calc = new Calculator();
calc.add(5, 3);         // Calls add(int, int)
calc.add(5, 3, 2);      // Calls add(int, int, int)
calc.add(5.5, 3.2);     // Calls add(double, double)
```

**Overloading with Type Promotion:**

```java
public class Example {
    public void method(int x) {
        System.out.println("int");
    }
    
    public void method(long x) {
        System.out.println("long");
    }
    
    public void method(Integer x) {
        System.out.println("Integer");
    }
}

Example ex = new Example();
ex.method(5);          // "int" - exact match
ex.method(5L);         // "long" - exact match
ex.method((byte) 5);   // "int" - promotion byte → int
ex.method(Integer.valueOf(5));  // "Integer" - wrapper
```

**Overloading Best Practices:**

```java
// ✅ GOOD: Clear semantic meaning
public void print(String message) { }
public void print(String message, int copies) { }
public void print(String message, int copies, boolean color) { }

// ❌ CONFUSING: Different types, unclear which to call
public void process(List<String> data) { }
public void process(Set<String> data) { }
public void process(Collection<String> data) { }
```

#### Runtime Polymorphism (Method Overriding)

**Dynamic Method Dispatch**: JVM determines which method to call at runtime based on actual object type.

```java
public class Animal {
    public void makeSound() {
        System.out.println("Generic animal sound");
    }
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

public class Cow extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Moo!");
    }
}

// Runtime polymorphism in action
Animal animal1 = new Dog();  // Upcasting
Animal animal2 = new Cat();
Animal animal3 = new Cow();

animal1.makeSound();  // "Woof!" - Dog's implementation
animal2.makeSound();  // "Meow!" - Cat's implementation
animal3.makeSound();  // "Moo!" - Cow's implementation

// Array of polymorphic objects
Animal[] animals = {new Dog(), new Cat(), new Cow()};
for (Animal animal : animals) {
    animal.makeSound();  // Calls appropriate method for each
}
```

**How Dynamic Dispatch Works:**

```
Compile time:
  Animal animal = new Dog();
  animal.makeSound();
  ↓
  Compiler: "animal is type Animal, Animal has makeSound(), ✅ OK"

Runtime:
  JVM: "What's the actual type of the object animal points to?"
  JVM: "It's a Dog object"
  JVM: "Call Dog's makeSound() method"
  ↓
  "Woof!"
```

#### Upcasting and Downcasting

**Upcasting** (Implicit): Treating a subclass object as its superclass type.

```java
Dog dog = new Dog("Max");
Animal animal = dog;  // Upcasting (automatic)

// Can only call methods declared in Animal
animal.makeSound();  // ✅ OK
animal.eat();        // ✅ OK (if defined in Animal)
// animal.fetch();   // ❌ Error: method not in Animal
```

**Downcasting** (Explicit): Treating a superclass reference as a subclass type.

```java
Animal animal = new Dog("Max");  // Upcasting

// Downcasting (explicit cast required)
Dog dog = (Dog) animal;
dog.fetch();  // ✅ Now can call Dog-specific methods

// ⚠️ Dangerous downcasting
Animal cat = new Cat();
Dog wrongCast = (Dog) cat;  // ☠️ ClassCastException at runtime!
```

**Safe Downcasting with instanceof:**

```java
public void handleAnimal(Animal animal) {
    // Check before casting
    if (animal instanceof Dog) {
        Dog dog = (Dog) animal;
        dog.fetch();
    } else if (animal instanceof Cat) {
        Cat cat = (Cat) animal;
        cat.climb();
    }
}

// Pattern matching for instanceof (Java 16+)
public void handleAnimalModern(Animal animal) {
    if (animal instanceof Dog dog) {
        // 'dog' is automatically cast and available
        dog.fetch();
    } else if (animal instanceof Cat cat) {
        cat.climb();
    }
}
```

#### Polymorphism in Practice

**Example: Payment Processing System**

```java
// Abstract payment method
public abstract class Payment {
    protected double amount;
    
    public Payment(double amount) {
        this.amount = amount;
    }
    
    // Template method
    public final void process() {
        if (validate()) {
            executePayment();
            sendConfirmation();
        }
    }
    
    protected abstract boolean validate();
    protected abstract void executePayment();
    
    protected void sendConfirmation() {
        System.out.println("Payment confirmation sent");
    }
}

// Concrete implementations
public class CreditCardPayment extends Payment {
    private String cardNumber;
    
    public CreditCardPayment(double amount, String cardNumber) {
        super(amount);
        this.cardNumber = cardNumber;
    }
    
    @Override
    protected boolean validate() {
        // Validate credit card
        return cardNumber != null && cardNumber.length() == 16;
    }
    
    @Override
    protected void executePayment() {
        System.out.println("Processing credit card payment: $" + amount);
    }
}

public class PayPalPayment extends Payment {
    private String email;
    
    public PayPalPayment(double amount, String email) {
        super(amount);
        this.email = email;
    }
    
    @Override
    protected boolean validate() {
        return email != null && email.contains("@");
    }
    
    @Override
    protected void executePayment() {
        System.out.println("Processing PayPal payment: $" + amount);
    }
}

// Usage - polymorphic behavior
public class PaymentProcessor {
    public void processPayments(List<Payment> payments) {
        for (Payment payment : payments) {
            payment.process();  // Calls appropriate implementation
        }
    }
}

// Client code
List<Payment> payments = Arrays.asList(
    new CreditCardPayment(100.0, "1234567890123456"),
    new PayPalPayment(50.0, "user@example.com"),
    new CreditCardPayment(200.0, "9876543210987654")
);

PaymentProcessor processor = new PaymentProcessor();
processor.processPayments(payments);  // Processes each according to its type
```

### Abstraction

**Abstraction** hides implementation details and shows only essential features.

**Two Ways to Achieve Abstraction in Java:**
1. Abstract classes (0-100% abstraction)
2. Interfaces (100% abstraction)

#### Abstract Classes

An **abstract class** cannot be instantiated and may contain abstract methods (methods without implementation).

```java
// Abstract class
public abstract class Shape {
    protected String color;
    
    // Constructor
    public Shape(String color) {
        this.color = color;
    }
    
    // Abstract method (no implementation)
    public abstract double calculateArea();
    public abstract double calculatePerimeter();
    
    // Concrete method
    public void displayColor() {
        System.out.println("Color: " + color);
    }
    
    // Concrete method using abstract method
    public void printDetails() {
        System.out.println("Area: " + calculateArea());
        System.out.println("Perimeter: " + calculatePerimeter());
        displayColor();
    }
}

// Concrete class - must implement all abstract methods
public class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double calculatePerimeter() {
        return 2 * Math.PI * radius;
    }
}

public class Rectangle extends Shape {
    private double width;
    private double height;
    
    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
    
    @Override
    public double calculatePerimeter() {
        return 2 * (width + height);
    }
}

// Usage
Shape circle = new Circle("Red", 5.0);
Shape rectangle = new Rectangle("Blue", 4.0, 6.0);

circle.printDetails();
rectangle.printDetails();

// ❌ Cannot instantiate abstract class
Shape shape = new Shape("Green");  // Compilation error!
```

**Abstract Class Rules:**
* Cannot be instantiated directly
* May contain abstract methods (must be implemented by subclasses)
* May contain concrete methods (with implementation)
* May have constructors (called via super() from subclass)
* May have instance variables and static members
* Can extend only one class (single inheritance)

#### Interfaces

An **interface** is a contract that defines what a class can do without specifying how.

**Before Java 8** (Pure abstraction):

```java
// Interface (100% abstract before Java 8)
public interface Drawable {
    // All methods implicitly public abstract
    void draw();
    void resize(int width, int height);
    
    // All fields implicitly public static final
    int MAX_SIZE = 1000;
}

// Implementation
public class Circle implements Drawable {
    private int radius;
    
    @Override
    public void draw() {
        System.out.println("Drawing circle with radius: " + radius);
    }
    
    @Override
    public void resize(int width, int height) {
        radius = Math.min(width, height) / 2;
    }
}
```

**Java 8+: Default and Static Methods**

```java
public interface Vehicle {
    // Abstract method
    void start();
    void stop();
    
    // Default method (Java 8+) - provides default implementation
    default void honk() {
        System.out.println("Beep beep!");
    }
    
    // Static method (Java 8+)
    static void checkLicense(String license) {
        System.out.println("Checking license: " + license);
    }
    
    // Private method (Java 9+) - helper for default methods
    private void log(String message) {
        System.out.println("LOG: " + message);
    }
    
    default void startWithLog() {
        log("Starting vehicle");
        start();
    }
}

// Implementation can use or override default methods
public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting...");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopping...");
    }
    
    // honk() inherited from interface (can optionally override)
    
    @Override
    public void honk() {
        System.out.println("Car honk: Beep!");  // Override default
    }
}
```

**Multiple Interface Implementation:**

```java
public interface Flyable {
    void fly();
}

public interface Swimmable {
    void swim();
}

// Class can implement multiple interfaces
public class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("Duck is flying");
    }
    
    @Override
    public void swim() {
        System.out.println("Duck is swimming");
    }
}

// Interface can extend multiple interfaces
public interface Amphibious extends Flyable, Swimmable {
    void walk();
}
```

**Diamond Problem with Default Methods:**

```java
public interface Interface1 {
    default void method() {
        System.out.println("Interface1");
    }
}

public interface Interface2 {
    default void method() {
        System.out.println("Interface2");
    }
}

// ❌ Ambiguity: Which method() to inherit?
public class MyClass implements Interface1, Interface2 {
    // ✅ Must explicitly override to resolve conflict
    @Override
    public void method() {
        // Option 1: Choose one
        Interface1.super.method();
        
        // Option 2: Custom implementation
        System.out.println("MyClass");
        
        // Option 3: Call both
        Interface1.super.method();
        Interface2.super.method();
    }
}
```

#### Marker Interfaces

**Marker interface**: Empty interface used to mark classes with special property.

```java
// Built-in marker interfaces
public interface Serializable { }
public interface Cloneable { }
public interface RandomAccess { }

// Custom marker interface
public interface Auditable { }

public class User implements Serializable, Auditable {
    // This class is now serializable and auditable
}

// Check at runtime
if (obj instanceof Serializable) {
    // Object can be serialized
}

if (obj instanceof Auditable) {
    // Log this operation
}
```

#### Functional Interfaces

**Functional interface**: Interface with exactly one abstract method (SAM - Single Abstract Method).

```java
@FunctionalInterface  // Optional but recommended annotation
public interface Calculator {
    int calculate(int a, int b);  // Single abstract method
    
    // Can have default and static methods
    default int square(int x) {
        return calculate(x, x);
    }
    
    static int sum(int... numbers) {
        int total = 0;
        for (int num : numbers) {
            total += num;
        }
        return total;
    }
}

// Can be implemented with lambda (Java 8+)
Calculator addition = (a, b) -> a + b;
Calculator multiplication = (a, b) -> a * b;

System.out.println(addition.calculate(5, 3));        // 8
System.out.println(multiplication.calculate(5, 3));  // 15
```

**Common Functional Interfaces** (covered in detail in Section 1.8):
* `Predicate<T>`: Takes T, returns boolean
* `Function<T, R>`: Takes T, returns R
* `Consumer<T>`: Takes T, returns void
* `Supplier<T>`: Takes nothing, returns T

#### Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Abstract and concrete | Abstract, default, static, private (Java 8+) |
| Variables | Any type of variables | Only public static final (constants) |
| Inheritance | Single (extends) | Multiple (implements) |
| Constructor | Yes | No |
| Access Modifiers | Any | Public (methods implicitly public) |
| When to Use | IS-A relationship, shared code | CAN-DO capability, contract |

**When to Use What:**

```java
// ✅ Abstract Class: IS-A relationship, shared implementation
public abstract class Employee {
    protected String name;
    protected double salary;
    
    // Common implementation
    public double calculateTax() {
        return salary * 0.2;  // Shared logic
    }
    
    // Must be implemented by subclasses
    public abstract double calculateBonus();
}

// ✅ Interface: CAN-DO capability
public interface Payable {
    void processPayment();
}

public interface Taxable {
    double calculateTax();
}

// Class can be Employee and also Payable
public class Manager extends Employee implements Payable {
    @Override
    public double calculateBonus() {
        return salary * 0.15;
    }
    
    @Override
    public void processPayment() {
        System.out.println("Processing manager payment");
    }
}
```

### Composition vs Inheritance

**Composition** (HAS-A relationship) is often preferred over inheritance.

**Problem with Inheritance:**

```java
// ❌ Inheritance can be rigid
public class ArrayList extends AbstractList {
    // Inherits ALL methods, even ones you don't want exposed
}

// If you want to restrict some operations, you're stuck
public class RestrictedList extends ArrayList {
    @Override
    public void clear() {
        throw new UnsupportedOperationException("Clear not allowed");
    }
    // But users can still call it and get exception at runtime!
}
```

**Solution with Composition:**

```java
// ✅ Composition: HAS-A relationship
public class Engine {
    private int horsepower;
    
    public void start() {
        System.out.println("Engine starting...");
    }
    
    public void stop() {
        System.out.println("Engine stopping...");
    }
}

public class Car {
    private Engine engine;  // HAS-A Engine
    private String model;
    
    public Car(String model) {
        this.model = model;
        this.engine = new Engine();  // Composition
    }
    
    public void start() {
        engine.start();  // Delegate to engine
        System.out.println(model + " is ready to drive");
    }
    
    public void stop() {
        System.out.println(model + " is stopping");
        engine.stop();  // Delegate to engine
    }
}

// Flexibility: Can change engine implementation easily
public class ElectricCar {
    private ElectricEngine engine;  // Different engine type
    
    // Rest of implementation...
}
```

**Favor Composition Over Inheritance:**

```java
// ❌ Inheritance: Fragile, tight coupling
public class Stack extends ArrayList {
    public void push(Object item) {
        add(item);
    }
    
    public Object pop() {
        return remove(size() - 1);
    }
}
// Problem: Exposes ALL ArrayList methods (get, set, etc.)
// Breaking stack contract

// ✅ Composition: Flexible, loose coupling
public class Stack {
    private List<Object> items = new ArrayList<>();  // HAS-A List
    
    public void push(Object item) {
        items.add(item);  // Delegate
    }
    
    public Object pop() {
        return items.remove(items.size() - 1);  // Delegate
    }
    
    public int size() {
        return items.size();  // Delegate only what you want
    }
    
    // Don't expose get(), set(), etc.
}
```

**When to Use Each:**

* **Inheritance**: True IS-A relationship, need polymorphism
  * Dog IS-A Animal
  * Manager IS-A Employee
  
* **Composition**: HAS-A relationship, need flexibility
  * Car HAS-A Engine
  * Computer HAS-A Processor
  * Person HAS-A Address

---

## 1.4 Advanced OOP Concepts

### Nested and Inner Classes

Java allows you to define classes within other classes for better organization and encapsulation.

#### Static Nested Classes

**Static nested class**: A static member class that doesn't have access to instance members of outer class.

```java
public class OuterClass {
    private static int staticOuter = 10;
    private int instanceOuter = 20;
    
    // Static nested class
    public static class StaticNestedClass {
        public void display() {
            System.out.println("Static outer: " + staticOuter);  // ✅ Can access
            // System.out.println(instanceOuter);  // ❌ Cannot access instance members
        }
    }
}

// Usage: Create without outer class instance
OuterClass.StaticNestedClass nested = new OuterClass.StaticNestedClass();
nested.display();
```

**Use Case:** Grouping helper classes with their outer class.

```java
public class LinkedList {
    // Node is only used by LinkedList
    private static class Node {
        int data;
        Node next;
        
        Node(int data) {
            this.data = data;
        }
    }
    
    private Node head;
    
    public void add(int data) {
        Node newNode = new Node(data);
        // Implementation...
    }
}
```

#### Non-Static Inner Classes

**Inner class**: Has access to all members of outer class, including private ones.

```java
public class OuterClass {
    private int outerValue = 100;
    
    // Inner class
    public class InnerClass {
        private int innerValue = 200;
        
        public void display() {
            System.out.println("Outer: " + outerValue);      // ✅ Access outer
            System.out.println("Inner: " + innerValue);
            System.out.println("Outer this: " + OuterClass.this.outerValue);
        }
    }
}

// Usage: Need outer instance to create inner instance
OuterClass outer = new OuterClass();
OuterClass.InnerClass inner = outer.new InnerClass();
inner.display();
```

**Use Case:** Event handlers, callbacks

```java
public class Button {
    private String label;
    
    // Inner class for click listener
    public class ClickListener {
        public void onClick() {
            System.out.println(label + " was clicked");  // Access outer's label
        }
    }
    
    public Button(String label) {
        this.label = label;
    }
    
    public ClickListener getClickListener() {
        return new ClickListener();
    }
}
```

#### Local Classes

**Local class**: Defined inside a method, has access to final/effectively final local variables.

```java
public class Example {
    public void method() {
        final int localVar = 10;  // Must be final or effectively final
        int effectivelyFinal = 20;  // Not modified, so effectively final
        
        // Local class
        class LocalClass {
            public void display() {
                System.out.println("Local: " + localVar);
                System.out.println("Effectively final: " + effectivelyFinal);
                // effectivelyFinal = 30;  // ❌ Would make it non-effectively-final
            }
        }
        
        LocalClass local = new LocalClass();
        local.display();
    }
}
```

#### Anonymous Classes

**Anonymous class**: Class without a name, defined and instantiated in one expression.

```java
// Interface to implement
interface Greeting {
    void greet(String name);
}

public class Example {
    public void demonstrateAnonymous() {
        // Anonymous class implementing interface
        Greeting greeting = new Greeting() {
            @Override
            public void greet(String name) {
                System.out.println("Hello, " + name + "!");
            }
        };
        
        greeting.greet("Alice");
        
        // Anonymous class extending class
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Running in thread");
            }
        });
        thread.start();
    }
}

// Modern alternative: Lambda expressions (Java 8+)
Greeting greeting = (name) -> System.out.println("Hello, " + name + "!");
Thread thread = new Thread(() -> System.out.println("Running in thread"));
```

**When to Use Anonymous Classes:**
* Quick implementation of interface/abstract class
* Only used once
* Before Java 8, for functional interfaces
* After Java 8, prefer lambdas for functional interfaces

### Enumerations (Enums)

**Enum**: A special class representing a fixed set of constants.

#### Basic Enum

```java
// Simple enum
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Usage
Day today = Day.MONDAY;

// Switch with enums
switch (today) {
    case MONDAY:
        System.out.println("Start of work week");
        break;
    case FRIDAY:
        System.out.println("Almost weekend!");
        break;
    case SATURDAY:
    case SUNDAY:
        System.out.println("Weekend!");
        break;
    default:
        System.out.println("Midweek");
}

// Iteration
for (Day day : Day.values()) {
    System.out.println(day);
}

// String conversion
Day day = Day.valueOf("MONDAY");  // String to enum
String name = Day.MONDAY.name();  // "MONDAY"
int ordinal = Day.MONDAY.ordinal();  // 0 (position)
```

#### Enum with Fields and Methods

```java
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6),
    MARS(6.421e+23, 3.3972e6);
    
    private final double mass;    // in kilograms
    private final double radius;  // in meters
    
    // Constructor (implicitly private)
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }
    
    public double getMass() {
        return mass;
    }
    
    public double getRadius() {
        return radius;
    }
    
    // Universal gravitational constant (m^3 kg^-1 s^-2)
    public static final double G = 6.67300E-11;
    
    public double surfaceGravity() {
        return G * mass / (radius * radius);
    }
    
    public double surfaceWeight(double otherMass) {
        return otherMass * surfaceGravity();
    }
}

// Usage
double earthWeight = 75.0;  // kg
double mass = earthWeight / Planet.EARTH.surfaceGravity();

for (Planet p : Planet.values()) {
    System.out.printf("Weight on %s: %.2f kg%n", 
                      p, p.surfaceWeight(mass));
}
```

#### Enum with Abstract Methods

```java
public enum Operation {
    PLUS {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };
    
    public abstract double apply(double x, double y);
}

// Usage
double result = Operation.PLUS.apply(5, 3);  // 8.0
```

#### EnumSet and EnumMap

```java
import java.util.EnumSet;
import java.util.EnumMap;

public enum Permission {
    READ, WRITE, EXECUTE, DELETE
}

// EnumSet: High-performance set for enums
EnumSet<Permission> permissions = EnumSet.of(Permission.READ, Permission.WRITE);
EnumSet<Permission> allPerms = EnumSet.allOf(Permission.class);
EnumSet<Permission> noPerms = EnumSet.noneOf(Permission.class);

// EnumMap: High-performance map with enum keys
EnumMap<Permission, String> descriptions = new EnumMap<>(Permission.class);
descriptions.put(Permission.READ, "Can read files");
descriptions.put(Permission.WRITE, "Can write files");
```

### Records (Java 16+)

**Records**: Immutable data carrier classes with concise syntax.

#### Basic Record

```java
// Traditional class (verbose)
public final class PointOld {
    private final int x;
    private final int y;
    
    public PointOld(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int getX() { return x; }
    public int getY() { return y; }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof PointOld)) return false;
        PointOld other = (PointOld) obj;
        return x == other.x && y == other.y;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
    
    @Override
    public String toString() {
        return "Point{x=" + x + ", y=" + y + "}";
    }
}

// Record (concise) - generates all of the above automatically!
public record Point(int x, int y) { }

// Usage - both classes behave identically
Point p1 = new Point(10, 20);
System.out.println(p1.x());      // Accessor method
System.out.println(p1.y());
System.out.println(p1);          // toString()
Point p2 = new Point(10, 20);
System.out.println(p1.equals(p2));  // true
```

**What Records Provide Automatically:**
* Private final fields for each component
* Public accessor methods (not getters! Just field name)
* `equals()` and `hashCode()`
* `toString()`
* Canonical constructor
* Final class (cannot be extended)

#### Record with Custom Methods

```java
public record Person(String name, int age) {
    // Compact constructor (validation)
    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
    }
    
    // Custom methods
    public boolean isAdult() {
        return age >= 18;
    }
    
    public Person withAge(int newAge) {
        return new Person(name, newAge);  // Immutable update pattern
    }
    
    // Static methods
    public static Person createChild(String name) {
        return new Person(name, 0);
    }
}

// Usage
Person person = new Person("Alice", 25);
System.out.println(person.isAdult());  // true

// Immutable "modification"
Person olderPerson = person.withAge(26);
System.out.println(person.age());       // 25 (unchanged)
System.out.println(olderPerson.age());  // 26
```

#### Records vs Classes

| Feature | Record | Class |
|---------|--------|-------|
| Mutability | Immutable | Mutable or immutable |
| Inheritance | Final (cannot extend or be extended) | Can extend and be extended |
| Purpose | Data carrier | Any purpose |
| Boilerplate | Minimal | More verbose |
| Flexibility | Limited | Full |
| Use Case | DTOs, value objects | Complex logic, state management |

**When to Use Records:**
* Data Transfer Objects (DTOs)
* Value objects (coordinates, money, etc.)
* Configuration data
* API responses
* Database query results

**When NOT to Use Records:**
* Need mutability
* Need inheritance
* Complex business logic
* Need to extend other classes

### Sealed Classes (Java 17+)

**Sealed classes**: Restrict which classes can extend/implement them.

#### Basic Sealed Class

```java
// Sealed class - explicitly lists permitted subclasses
public sealed class Shape
    permits Circle, Rectangle, Triangle {
    
    public abstract double area();
}

// Permitted subclasses must be one of: final, sealed, or non-sealed

// 1. final - cannot be extended further
public final class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// 2. sealed - extends the sealed hierarchy
public sealed class Rectangle extends Shape
    permits Square {
    
    protected final double width;
    protected final double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}

// 3. non-sealed - opens hierarchy again (anyone can extend)
public non-sealed class Triangle extends Shape {
    private final double base;
    private final double height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double area() {
        return 0.5 * base * height;
    }
}

// Square extends sealed Rectangle
public final class Square extends Rectangle {
    public Square(double side) {
        super(side, side);
    }
}

// ❌ ERROR: Not in permits list
public class Pentagon extends Shape { }  // Compilation error!

// ✅ OK: Triangle is non-sealed
public class RightTriangle extends Triangle {
    public RightTriangle(double base, double height) {
        super(base, height);
    }
}
```

#### Sealed Interfaces

```java
public sealed interface Payment
    permits CreditCardPayment, DebitCardPayment, PayPalPayment {
    
    void process();
    double getAmount();
}

public final class CreditCardPayment implements Payment {
    private final double amount;
    private final String cardNumber;
    
    public CreditCardPayment(double amount, String cardNumber) {
        this.amount = amount;
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void process() {
        System.out.println("Processing credit card payment");
    }
    
    @Override
    public double getAmount() {
        return amount;
    }
}

// Other implementations...
```

#### Pattern Matching with Sealed Classes (Java 17+)

```java
public class PaymentProcessor {
    public void processPayment(Payment payment) {
        // Exhaustive pattern matching (compiler ensures all cases covered)
        switch (payment) {
            case CreditCardPayment cc -> 
                System.out.println("Credit card: " + cc.getAmount());
            case DebitCardPayment dc -> 
                System.out.println("Debit card: " + dc.getAmount());
            case PayPalPayment pp -> 
                System.out.println("PayPal: " + pp.getAmount());
            // No default needed! Compiler knows all possible types
        }
    }
    
    // With Java 21 pattern matching enhancements
    public String describeShape(Shape shape) {
        return switch (shape) {
            case Circle c -> "Circle with radius " + c.getRadius();
            case Rectangle r when r instanceof Square -> 
                "Square with side " + r.getWidth();
            case Rectangle r -> 
                "Rectangle " + r.getWidth() + "x" + r.getHeight();
            case Triangle t -> 
                "Triangle with base " + t.getBase();
        };
    }
}
```

#### Benefits of Sealed Classes

1. **Controlled Inheritance**: Explicitly define class hierarchy
2. **Exhaustive Switching**: Compiler verifies all cases handled
3. **Better API Design**: Clear contracts and type safety
4. **Domain Modeling**: Represent closed sets (like algebraic data types)

**Use Cases:**
* Domain models with fixed set of types (payment methods, shapes)
* State machines with known states
* AST (Abstract Syntax Tree) nodes
* Error types in result patterns

---

## 1.5 Exception Handling

Exception handling is critical for building robust, production-ready applications. Java provides a comprehensive mechanism for dealing with runtime errors gracefully.

### Exception Hierarchy

All exceptions in Java derive from `Throwable`:

```
java.lang.Object
    └── java.lang.Throwable
            ├── java.lang.Error (Unchecked)
            │       ├── OutOfMemoryError
            │       ├── StackOverflowError
            │       └── VirtualMachineError
            └── java.lang.Exception
                    ├── IOException (Checked)
                    ├── SQLException (Checked)
                    ├── ClassNotFoundException (Checked)
                    └── RuntimeException (Unchecked)
                            ├── NullPointerException
                            ├── ArrayIndexOutOfBoundsException
                            ├── IllegalArgumentException
                            ├── ArithmeticException
                            └── NumberFormatException
```

**Three Categories:**

1. **Errors**: Serious problems external to the application (usually unrecoverable)
2. **Checked Exceptions**: Recoverable conditions that must be declared or caught
3. **Unchecked Exceptions** (RuntimeException): Programming bugs, optional to handle

### Checked vs Unchecked Exceptions

#### Checked Exceptions

Must be either caught or declared in method signature:

```java
import java.io.*;

// ❌ Doesn't compile without handling
public void readFile(String path) {
    FileReader reader = new FileReader(path);  // Compilation error!
}

// ✅ Option 1: Declare with throws
public void readFile(String path) throws IOException {
    FileReader reader = new FileReader(path);
    // Method caller must handle IOException
}

// ✅ Option 2: Catch and handle
public void readFile(String path) {
    try {
        FileReader reader = new FileReader(path);
        // Use reader...
    } catch (IOException e) {
        System.err.println("Error reading file: " + e.getMessage());
    }
}
```

**Common Checked Exceptions:**
* `IOException` - I/O operations
* `SQLException` - Database operations
* `ClassNotFoundException` - Class loading
* `InterruptedException` - Thread interruption
* `FileNotFoundException` - File not found

#### Unchecked Exceptions (RuntimeException)

Do not require declaration or catching:

```java
// Compiles fine even though it might throw NullPointerException
public void processString(String str) {
    System.out.println(str.length());  // NPE if str is null
}

// Can optionally catch
public void safeProcessString(String str) {
    try {
        System.out.println(str.length());
    } catch (NullPointerException e) {
        System.out.println("String is null");
    }
}

// Better: Prevent with validation
public void betterProcessString(String str) {
    if (str == null) {
        throw new IllegalArgumentException("String cannot be null");
    }
    System.out.println(str.length());
}
```

**Common Unchecked Exceptions:**
* `NullPointerException` - Null reference access
* `ArrayIndexOutOfBoundsException` - Invalid array index
* `IllegalArgumentException` - Invalid method argument
* `IllegalStateException` - Invalid object state
* `ArithmeticException` - Math errors (e.g., divide by zero)
* `ClassCastException` - Invalid type cast
* `NumberFormatException` - Invalid number format

### Exception Handling Mechanisms

#### try-catch Blocks

```java
public void demonstrateTryCatch() {
    try {
        // Code that might throw exception
        int result = 10 / 0;
        System.out.println("This won't execute");
    } catch (ArithmeticException e) {
        // Handle specific exception
        System.out.println("Cannot divide by zero");
    }
}

// Multiple catch blocks (specific to general)
public void readAndParse(String filename) {
    try {
        FileReader reader = new FileReader(filename);
        BufferedReader br = new BufferedReader(reader);
        String line = br.readLine();
        int number = Integer.parseInt(line);
        
    } catch (FileNotFoundException e) {
        System.out.println("File not found: " + filename);
    } catch (IOException e) {
        System.out.println("Error reading file: " + e.getMessage());
    } catch (NumberFormatException e) {
        System.out.println("Invalid number format");
    }
}

// ❌ BAD: General exception before specific
try {
    // code
} catch (Exception e) {
    // Catches everything
} catch (IOException e) {  // Unreachable code! Compilation error
    // Never reached
}

// ✅ GOOD: Specific before general
try {
    // code
} catch (IOException e) {
    // Handle IO exception
} catch (Exception e) {
    // Handle any other exception
}
```

#### Multi-Catch (Java 7+)

```java
// Before Java 7: Duplicate code
try {
    // code
} catch (IOException e) {
    e.printStackTrace();
    logError(e);
} catch (SQLException e) {
    e.printStackTrace();
    logError(e);
}

// Java 7+: Multi-catch
try {
    // code
} catch (IOException | SQLException e) {
    e.printStackTrace();
    logError(e);
}

// ⚠️ Exception types cannot be in inheritance relationship
try {
    // code
} catch (IOException | FileNotFoundException e) {  // Error!
    // FileNotFoundException extends IOException
}
```

#### finally Block

Guaranteed to execute whether exception occurs or not:

```java
FileReader reader = null;
try {
    reader = new FileReader("file.txt");
    // Process file
    
} catch (IOException e) {
    System.err.println("Error: " + e.getMessage());
    
} finally {
    // Always executes (cleanup code)
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            System.err.println("Error closing file");
        }
    }
}
```

**finally Execution Scenarios:**

```java
public int testFinally() {
    try {
        return 1;  // Finally still executes before return!
    } finally {
        System.out.println("Finally executes");
    }
}

public int finallyWithReturn() {
    try {
        return 1;
    } finally {
        return 2;  // ⚠️ Overrides try's return! (Anti-pattern)
    }
    // Returns 2
}

// Only scenario finally doesn't execute: JVM crash
public void demonstrateFinallySkip() {
    try {
        System.exit(0);  // JVM terminates, finally skipped
    } finally {
        System.out.println("Never prints");
    }
}
```

#### try-with-resources (Java 7+)

Automatically closes resources implementing `AutoCloseable`:

```java
// Before Java 7: Manual resource management
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    String line = reader.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// Java 7+: try-with-resources (much cleaner!)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line = reader.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
}
// reader.close() called automatically

// Multiple resources
try (FileInputStream fis = new FileInputStream("input.txt");
     FileOutputStream fos = new FileOutputStream("output.txt");
     BufferedReader br = new BufferedReader(new InputStreamReader(fis));
     BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(fos))) {
    
    String line;
    while ((line = br.readLine()) != null) {
        bw.write(line);
        bw.newLine();
    }
} catch (IOException e) {
    e.printStackTrace();
}
// All resources closed in reverse order of creation

// Java 9+: Effectively final variables in try-with-resources
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
try (reader) {  // Java 9+: can use existing variable
    String line = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
```

**AutoCloseable Interface:**

```java
// Custom resource
public class CustomResource implements AutoCloseable {
    public CustomResource() {
        System.out.println("Resource acquired");
    }
    
    public void doWork() {
        System.out.println("Doing work");
    }
    
    @Override
    public void close() {
        System.out.println("Resource released");
    }
}

// Usage
try (CustomResource resource = new CustomResource()) {
    resource.doWork();
}
// Output:
// Resource acquired
// Doing work
// Resource released
```

### Throwing Exceptions

#### throw Statement

```java
public void setAge(int age) {
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("Invalid age: " + age);
    }
    this.age = age;
}

public void withdraw(double amount) {
    if (amount > balance) {
        throw new IllegalStateException("Insufficient funds");
    }
    balance -= amount;
}

// Rethrowing exception
public void processFile(String filename) throws IOException {
    try {
        // File operations
    } catch (IOException e) {
        System.err.println("Error processing file");
        throw e;  // Rethrow to caller
    }
}

// Wrapping exception (exception chaining)
public void loadData() throws DataException {
    try {
        // Database operation
    } catch (SQLException e) {
        throw new DataException("Failed to load data", e);  // Wrap original
    }
}
```

#### throws Clause

```java
// Single exception
public void readFile(String path) throws IOException {
    // May throw IOException
}

// Multiple exceptions
public void process() throws IOException, SQLException {
    // May throw either exception
}

// Method overriding rules for throws
class Parent {
    public void method() throws IOException {
    }
}

class Child extends Parent {
    // ✅ OK: Can throw same exception
    @Override
    public void method() throws IOException {
    }
    
    // ✅ OK: Can throw subclass exception
    @Override
    public void method() throws FileNotFoundException {
    }
    
    // ✅ OK: Can throw no exception
    @Override
    public void method() {
    }
    
    // ❌ ERROR: Cannot throw broader exception
    @Override
    public void method() throws Exception {  // Compilation error!
    }
    
    // ❌ ERROR: Cannot throw unrelated checked exception
    @Override
    public void method() throws SQLException {  // Compilation error!
    }
}
```

### Creating Custom Exceptions

```java
// Custom checked exception
public class InsufficientFundsException extends Exception {
    private double amount;
    private double balance;
    
    public InsufficientFundsException(double amount, double balance) {
        super("Insufficient funds: tried to withdraw " + amount + 
              " but balance is " + balance);
        this.amount = amount;
        this.balance = balance;
    }
    
    public double getAmount() {
        return amount;
    }
    
    public double getBalance() {
        return balance;
    }
}

// Custom unchecked exception
public class InvalidAccountException extends RuntimeException {
    public InvalidAccountException(String accountNumber) {
        super("Invalid account number: " + accountNumber);
    }
    
    public InvalidAccountException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Usage
public class BankAccount {
    private double balance;
    
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount, balance);
        }
        balance -= amount;
    }
}

// Client code
try {
    account.withdraw(1000);
} catch (InsufficientFundsException e) {
    System.out.println("Cannot withdraw: " + e.getMessage());
    System.out.println("Current balance: " + e.getBalance());
    System.out.println("Requested amount: " + e.getAmount());
}
```

### Exception Chaining

```java
public class DataException extends Exception {
    public DataException(String message, Throwable cause) {
        super(message, cause);
    }
}

public void loadUserData(int userId) throws DataException {
    try {
        // Database query
        String query = "SELECT * FROM users WHERE id = " + userId;
        // Execute query...
        
    } catch (SQLException e) {
        // Chain original exception
        throw new DataException("Failed to load user data for id: " + userId, e);
    }
}

// Retrieving cause
try {
    loadUserData(123);
} catch (DataException e) {
    System.err.println("Error: " + e.getMessage());
    
    Throwable cause = e.getCause();
    if (cause instanceof SQLException) {
        SQLException sqlEx = (SQLException) cause;
        System.err.println("SQL State: " + sqlEx.getSQLState());
        System.err.println("Error Code: " + sqlEx.getErrorCode());
    }
}
```

### Suppressed Exceptions

Occur when exception is thrown in try-with-resources cleanup:

```java
public class Resource implements AutoCloseable {
    private String name;
    
    public Resource(String name) {
        this.name = name;
    }
    
    public void doWork() throws Exception {
        System.out.println(name + " working");
        throw new Exception(name + " work exception");
    }
    
    @Override
    public void close() throws Exception {
        System.out.println(name + " closing");
        throw new Exception(name + " close exception");
    }
}

// Try-with-resources handles suppressed exceptions
try (Resource r1 = new Resource("R1");
     Resource r2 = new Resource("R2")) {
    
    r1.doWork();
    r2.doWork();
    
} catch (Exception e) {
    System.out.println("Main exception: " + e.getMessage());
    
    // Get suppressed exceptions
    Throwable[] suppressed = e.getSuppressed();
    for (Throwable t : suppressed) {
        System.out.println("Suppressed: " + t.getMessage());
    }
}

// Output:
// R1 working
// Main exception: R1 work exception
// Suppressed: R2 close exception
// Suppressed: R1 close exception
```

### Best Practices

#### 1. Catch Specific Exceptions

```java
// ❌ BAD: Catching generic Exception
try {
    processData();
} catch (Exception e) {
    // Too broad, might hide bugs
}

// ✅ GOOD: Catch specific exceptions
try {
    processData();
} catch (IOException e) {
    // Handle IO error
} catch (SQLException e) {
    // Handle database error
}
```

#### 2. Don't Swallow Exceptions

```java
// ❌ BAD: Empty catch block
try {
    riskyOperation();
} catch (Exception e) {
    // Silent failure, impossible to debug
}

// ✅ GOOD: At minimum, log the exception
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("Failed to execute risky operation", e);
    // Or rethrow if cannot handle
    throw new ApplicationException("Operation failed", e);
}
```

#### 3. Use try-with-resources

```java
// ❌ BAD: Manual resource management
FileInputStream fis = null;
try {
    fis = new FileInputStream("file.txt");
    // Use stream
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fis != null) {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// ✅ GOOD: try-with-resources
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // Use stream
} catch (IOException e) {
    e.printStackTrace();
}
```

#### 4. Provide Context in Exception Messages

```java
// ❌ BAD: Generic message
throw new Exception("Error");

// ✅ GOOD: Descriptive message with context
throw new IllegalArgumentException(
    "Invalid email address: '" + email + "'. Email must contain '@' symbol");

// ✅ BETTER: Include relevant data
throw new InsufficientFundsException(
    String.format("Cannot withdraw $%.2f. Current balance: $%.2f", 
                  amount, balance),
    amount,
    balance);
```

#### 5. Close Resources in finally or Use try-with-resources

```java
// ✅ Option 1: finally
Connection conn = null;
try {
    conn = getConnection();
    // Use connection
} catch (SQLException e) {
    handleError(e);
} finally {
    if (conn != null) {
        try {
            conn.close();
        } catch (SQLException e) {
            log.error("Error closing connection", e);
        }
    }
}

// ✅ Option 2: try-with-resources (preferred)
try (Connection conn = getConnection()) {
    // Use connection
} catch (SQLException e) {
    handleError(e);
}
```

#### 6. Document Exceptions with @throws

```java
/**
 * Retrieves user by ID from database.
 *
 * @param userId the user ID to retrieve
 * @return User object
 * @throws SQLException if database error occurs
 * @throws IllegalArgumentException if userId is negative
 */
public User getUserById(int userId) throws SQLException {
    if (userId < 0) {
        throw new IllegalArgumentException("User ID must be positive");
    }
    // Database query
}
```

#### 7. Convert Checked to Unchecked When Appropriate

```java
// In some cases, checked exceptions don't add value
public interface Callback {
    void execute();  // Cleaner without throws IOException
}

// Wrapper method converts checked to unchecked
public void safeExecute(Callback callback) {
    try {
        callback.execute();
    } catch (IOException e) {
        throw new UncheckedIOException(e);  // Java 8+
    }
}
```

### Common Exception Anti-Patterns

```java
// ❌ ANTI-PATTERN 1: Exception for flow control
public Integer parseIntOrNull(String str) {
    try {
        return Integer.parseInt(str);
    } catch (NumberFormatException e) {
        return null;  // Using exception for normal flow
    }
}

// ✅ BETTER: Validate first
public Integer parseIntOrNull(String str) {
    if (str == null || !str.matches("-?\\d+")) {
        return null;
    }
    return Integer.parseInt(str);
}

// ❌ ANTI-PATTERN 2: Catching Throwable
try {
    code();
} catch (Throwable t) {  // Catches EVERYTHING including Errors
    // Might catch OutOfMemoryError, StackOverflowError, etc.
}

// ✅ BETTER: Catch Exception
try {
    code();
} catch (Exception e) {
    // Catches exceptions, not Errors
}

// ❌ ANTI-PATTERN 3: Throwing in finally
try {
    riskyOperation();
} finally {
    throw new Exception();  // Masks exception from try block!
}

// ✅ BETTER: Log but don't throw in finally
try {
    riskyOperation();
} finally {
    try {
        cleanup();
    } catch (Exception e) {
        logger.error("Cleanup failed", e);
        // Don't throw
    }
}
```

### Frequently Asked Questions

**Q1: When should I use checked vs unchecked exceptions?**

**A:**

**Use Checked Exceptions when:**
* The caller can reasonably recover from the exception
* It's a business logic condition that the caller needs to handle
* Examples: `FileNotFoundException`, `SQLException`

**Use Unchecked Exceptions when:**
* It's a programming bug (should be fixed, not handled)
* Recovery is not possible or practical
* Examples: `NullPointerException`, `IllegalArgumentException`

```java
// Checked: Caller can handle
public User loadUser(int id) throws UserNotFoundException {
    // Caller might try alternative data source
}

// Unchecked: Programming bug
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age must be positive");
        // This is a bug in calling code, not recoverable condition
    }
}
```

---

**Q2: What's the difference between `throw` and `throws`?**

**A:**

* **`throw`**: Statement to actually throw an exception (used in method body)
* **`throws`**: Clause in method signature to declare potential exceptions

```java
// throws: Declares exception
public void method() throws IOException {
    // throw: Actually throws exception
    throw new IOException("File error");
}
```

---

**Q3: Why should I use try-with-resources instead of finally?**

**A:**

try-with-resources is safer and more concise:

**Advantages:**
1. Automatic resource cleanup (can't forget)
2. Handles suppressed exceptions properly
3. More readable and less boilerplate
4. Resources closed in reverse order automatically

```java
// finally: Verbose, error-prone
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("file.txt"));
    // Use br
} finally {
    if (br != null) {
        try {
            br.close();  // Might also throw!
        } catch (IOException e) {
            // What to do here?
        }
    }
}

// try-with-resources: Clean, safe
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    // Use br
}  // Automatically closed, suppressed exceptions handled
```

---

**Q4: Can finally block prevent a method from returning?**

**A:**

Yes, if finally contains a return statement, it overrides the try/catch return:

```java
public int confusingMethod() {
    try {
        return 1;
    } finally {
        return 2;  // ⚠️ This is returned (anti-pattern!)
    }
    // Method returns 2, not 1
}

// ✅ DON'T do this! finally should only cleanup, not change flow
```

**Best Practice:** Never put return, throw, or break in finally block.

---

**Q5: What happens if an exception is thrown in catch block?**

**A:**

The original exception is lost unless you chain it:

```java
// ❌ BAD: Loses original exception
try {
    operation1();
} catch (Exception e) {
    throw new RuntimeException("Operation failed");  // Original lost!
}

// ✅ GOOD: Chain original exception
try {
    operation1();
} catch (Exception e) {
    throw new RuntimeException("Operation failed", e);  // Original preserved
}
```

### Interview Questions

**Question 1: Explain the exception hierarchy in Java. What's the difference between Error and Exception?**

**Difficulty:** Junior

**Topics:** Exception Fundamentals

**Answer:**

Java's exception hierarchy:

```
Throwable
├── Error (Unchecked)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── IOException (Checked)
    ├── SQLException (Checked)
    └── RuntimeException (Unchecked)
        ├── NullPointerException
        ├── IllegalArgumentException
        └── ArrayIndexOutOfBoundsException
```

**Key Differences:**

| Aspect | Error | Exception |
|--------|-------|-----------|
| Purpose | System/JVM problems | Application problems |
| Recovery | Usually unrecoverable | Often recoverable |
| Handling | Should not catch | Should handle |
| Examples | OutOfMemoryError, StackOverflowError | IOException, SQLException |

```java
// ❌ Don't catch Errors
try {
    // Code
} catch (OutOfMemoryError e) {
    // Can't recover from OOM
}

// ✅ Do catch Exceptions
try {
    // Code
} catch (IOException e) {
    // Can retry, use fallback, etc.
}
```

**Why This Matters:** Understanding when to catch and when to let fail is crucial for application stability and debugging.

**Follow-up Questions:**
* What's the difference between checked and unchecked exceptions?
* When would you catch an Error?
* Why does RuntimeException extend Exception but is unchecked?

---

**Question 2: Explain try-with-resources. How does it handle suppressed exceptions?**

**Difficulty:** Mid-Level

**Topics:** Resource Management, Exception Handling

**Answer:**

try-with-resources automatically closes resources implementing `AutoCloseable` or `Closeable`:

```java
// Automatic resource management (Java 7+)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    return reader.readLine();
}  // reader.close() called automatically
```

**Suppressed Exceptions:**

When exception occurs in try block AND during close(), try-with-resources preserves both:

```java
public class Resource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new Exception("Close failed");
    }
}

try (Resource r = new Resource()) {
    throw new Exception("Operation failed");
} catch (Exception e) {
    System.out.println("Main: " + e.getMessage());  // "Operation failed"
    
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println("Suppressed: " + suppressed.getMessage());  // "Close failed"
    }
}
```

**Benefits over manual finally:**
1. Can't forget to close
2. Suppressed exceptions preserved
3. Resources closed in reverse order
4. More readable

**Why This Matters:** Proper resource management prevents resource leaks and makes exception handling more robust.

**Follow-up Questions:**
* What interfaces can be used with try-with-resources?
* What happens if resource creation throws an exception?
* Can you use final variables in try-with-resources (Java 9+)?

---

**Question 3: What are the rules for overriding methods that throw exceptions?**

**Difficulty:** Mid-Level

**Topics:** Inheritance, Exception Handling

**Answer:**

When overriding a method that throws checked exceptions:

**Rules:**
1. Can throw **same** exception
2. Can throw **subclass** (narrower) exception
3. Can throw **no** exception
4. **Cannot** throw **broader** exception
5. **Cannot** throw **new** checked exception
6. **Can** throw any RuntimeException (unchecked)

```java
class Parent {
    public void method() throws IOException {
    }
}

class Child extends Parent {
    // ✅ Same exception
    @Override
    public void method() throws IOException {
    }
    
    // ✅ Subclass exception (narrower)
    @Override
    public void method() throws FileNotFoundException {
    }
    
    // ✅ No exception
    @Override
    public void method() {
    }
    
    // ❌ Broader exception
    @Override
    public void method() throws Exception {  // ERROR!
    }
    
    // ❌ New checked exception
    @Override
    public void method() throws SQLException {  // ERROR!
    }
    
    // ✅ Can throw unchecked (RuntimeException)
    @Override
    public void method() throws IllegalArgumentException {
    }
}
```

**Why:** Liskov Substitution Principle - subclass must be usable wherever parent is used. If parent throws IOException, caller is prepared for it. If child throws Exception (broader), caller isn't prepared.

```java
Parent p = new Child();
try {
    p.method();  // Caller expects IOException, not Exception
} catch (IOException e) {
    // Handle
}
```

**Why This Matters:** Ensures polymorphism works correctly with exception handling.

**Follow-up Questions:**
* What about method overloading and exceptions?
* Can constructors in subclass throw broader exceptions?
* How do lambda expressions handle checked exceptions?

**Red Flags in Answers:**
* Claiming RuntimeException must be declared
* Not understanding checked vs unchecked distinction
* Thinking finally always executes (System.exit case)

---

## 1.6 Collections Framework

The Java Collections Framework provides data structures and algorithms for storing and manipulating groups of objects. Mastering collections is essential for efficient Java programming.

### Collection Hierarchy Overview

```
Iterable<E>
    └── Collection<E>
            ├── List<E> (ordered, allows duplicates)
            │       ├── ArrayList
            │       ├── LinkedList
            │       ├── Vector (legacy)
            │       └── Stack (legacy)
            ├── Set<E> (no duplicates)
            │       ├── HashSet
            │       ├── LinkedHashSet
            │       └── SortedSet<E>
            │               └── NavigableSet<E>
            │                       └── TreeSet
            └── Queue<E> (FIFO)
                    ├── PriorityQueue
                    └── Deque<E>
                            ├── ArrayDeque
                            └── LinkedList

Map<K,V> (separate hierarchy - key-value pairs)
    ├── HashMap
    ├── LinkedHashMap
    ├── Hashtable (legacy)
    ├── Properties
    └── SortedMap<K,V>
            └── NavigableMap<K,V>
                    └── TreeMap
```

### List Interface

**List**: Ordered collection that allows duplicates.

#### ArrayList

**Implementation**: Dynamic array (resizable array)

```java
import java.util.ArrayList;
import java.util.List;

// Creation
List<String> list = new ArrayList<>();  // Preferred (program to interface)
ArrayList<String> arrayList = new ArrayList<>();  // Specific type

// Initial capacity (optimization)
List<String> optimized = new ArrayList<>(1000);  // Avoid resizing

// From existing collection
List<String> copy = new ArrayList<>(existingList);

// Common operations
list.add("Apple");              // Add to end: O(1) amortized
list.add(0, "Banana");          // Add at index: O(n)
list.get(0);                    // Access by index: O(1)
list.set(1, "Cherry");          // Update: O(1)
list.remove(0);                 // Remove by index: O(n)
list.remove("Apple");           // Remove by value: O(n)
list.contains("Apple");         // Check existence: O(n)
list.size();                    // Get size: O(1)
list.clear();                   // Remove all: O(n)

// Iteration
for (String item : list) {
    System.out.println(item);
}

// Index-based iteration
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

// Iterator
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String item = it.next();
    if (shouldRemove(item)) {
        it.remove();  // Safe removal during iteration
    }
}
```

**ArrayList Characteristics:**
* Fast random access (index-based): O(1)
* Slow insertion/deletion in middle: O(n)
* Dynamic resizing (capacity doubles when full)
* Not thread-safe
* Best for: Read-heavy operations, random access

**Internal Working:**

```java
// Simplified ArrayList internals
public class SimpleArrayList<E> {
    private Object[] elementData;  // Internal array
    private int size;              // Number of elements
    
    public SimpleArrayList() {
        elementData = new Object[10];  // Default initial capacity
    }
    
    public boolean add(E e) {
        ensureCapacity(size + 1);
        elementData[size++] = e;
        return true;
    }
    
    private void ensureCapacity(int minCapacity) {
        if (minCapacity > elementData.length) {
            int newCapacity = elementData.length * 2;  // Double size
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }
    
    @SuppressWarnings("unchecked")
    public E get(int index) {
        if (index >= size) throw new IndexOutOfBoundsException();
        return (E) elementData[index];
    }
}
```

#### LinkedList

**Implementation**: Doubly-linked list

```java
import java.util.LinkedList;

LinkedList<String> list = new LinkedList<>();

// List operations
list.add("First");               // Add to end: O(1)
list.addFirst("New First");      // Add to beginning: O(1)
list.addLast("Last");            // Add to end: O(1)
list.getFirst();                 // Get first: O(1)
list.getLast();                  // Get last: O(1)
list.removeFirst();              // Remove first: O(1)
list.removeLast();               // Remove last: O(1)

// Queue operations (FIFO)
list.offer("Element");           // Add to end
list.poll();                     // Remove and return first
list.peek();                     // Get first without removing

// Stack operations (LIFO)
list.push("Element");            // Add to beginning
list.pop();                      // Remove and return first

// ⚠️ Slow random access
String element = list.get(500);  // O(n) - must traverse list
```

**LinkedList Characteristics:**
* Fast insertion/deletion at ends: O(1)
* Fast insertion/deletion in middle: O(1) if you have position
* Slow random access: O(n)
* More memory overhead (node objects, prev/next pointers)
* Implements both List and Deque
* Best for: Insertion/deletion heavy operations, queue/stack operations

#### ArrayList vs LinkedList

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| Random access (get) | O(1) | O(n) |
| Add to end | O(1)* | O(1) |
| Add to beginning | O(n) | O(1) |
| Add in middle | O(n) | O(1)** |
| Remove from end | O(1) | O(1) |
| Remove from beginning | O(n) | O(1) |
| Memory | Less overhead | More overhead (nodes) |

*O(1) amortized, O(n) when resizing  
**O(1) if position known, O(n) to find position

**Choosing Between Them:**

```java
// ✅ Use ArrayList when:
// - Mostly reading/accessing elements
// - Need fast random access by index
// - Memory constrained
List<String> names = new ArrayList<>();
for (int i = 0; i < names.size(); i++) {
    System.out.println(names.get(i));  // Fast with ArrayList
}

// ✅ Use LinkedList when:
// - Frequent insertions/deletions at beginning/end
// - Implementing queue/deque
// - Rarely need random access
Queue<Task> taskQueue = new LinkedList<>();
taskQueue.offer(new Task());  // Efficient queue operations
taskQueue.poll();
```

### Set Interface

**Set**: Collection with no duplicate elements.

#### HashSet

**Implementation**: Hash table (HashMap internally)

```java
import java.util.HashSet;
import java.util.Set;

Set<String> set = new HashSet<>();

// Operations
set.add("Apple");         // Add: O(1) average
set.add("Banana");
set.add("Apple");         // Duplicate ignored
set.contains("Apple");    // Check: O(1) average
set.remove("Banana");     // Remove: O(1) average
set.size();               // 1 (duplicates not counted)

// No order guaranteed
set.add("A");
set.add("Z");
set.add("M");
for (String item : set) {
    System.out.println(item);  // Order: unpredictable
}

// Set operations
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 3, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(3, 4, 5, 6));

// Union
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);  // {1, 2, 3, 4, 5, 6}

// Intersection
Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);  // {3, 4}

// Difference
Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);  // {1, 2}
```

**HashSet Requirements:**
* Elements must properly implement `hashCode()` and `equals()`
* Good hash function prevents collisions

```java
// Custom class in HashSet
public class Person {
    private String name;
    private int age;
    
    // Must override both!
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return age == other.age && Objects.equals(name, other.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

Set<Person> people = new HashSet<>();
people.add(new Person("Alice", 30));
people.add(new Person("Alice", 30));  // Duplicate (same hashCode and equals)
System.out.println(people.size());    // 1
```

#### LinkedHashSet

**Implementation**: Hash table + linked list (maintains insertion order)

```java
Set<String> linkedSet = new LinkedHashSet<>();

linkedSet.add("Zebra");
linkedSet.add("Apple");
linkedSet.add("Mango");

// Iteration order = insertion order
for (String item : linkedSet) {
    System.out.println(item);  // Zebra, Apple, Mango
}
```

**Use Case:** When you need Set uniqueness with predictable iteration order.

#### TreeSet

**Implementation**: Red-Black tree (self-balancing BST)

```java
import java.util.TreeSet;
import java.util.NavigableSet;

NavigableSet<Integer> treeSet = new TreeSet<>();

treeSet.add(5);
treeSet.add(2);
treeSet.add(8);
treeSet.add(1);

// Sorted order (natural ordering)
for (Integer num : treeSet) {
    System.out.println(num);  // 1, 2, 5, 8
}

// NavigableSet operations
treeSet.first();            // 1 (smallest)
treeSet.last();             // 8 (largest)
treeSet.lower(5);           // 2 (largest element < 5)
treeSet.higher(5);          // 8 (smallest element > 5)
treeSet.floor(4);           // 2 (largest element <= 4)
treeSet.ceiling(6);         // 8 (smallest element >= 6)

// Range views
treeSet.headSet(5);         // Elements < 5: {1, 2}
treeSet.tailSet(5);         // Elements >= 5: {5, 8}
treeSet.subSet(2, 8);       // Elements >= 2 and < 8: {2, 5}

// Descending order
NavigableSet<Integer> descending = treeSet.descendingSet();
// 8, 5, 2, 1
```

**TreeSet Requirements:**
* Elements must be Comparable OR provide Comparator
* Sorted in natural order or by Comparator

```java
// Custom class with TreeSet
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    
    @Override
    public int compareTo(Person other) {
        // Sort by age, then name
        int ageCompare = Integer.compare(this.age, other.age);
        if (ageCompare != 0) return ageCompare;
        return this.name.compareTo(other.name);
    }
}

TreeSet<Person> people = new TreeSet<>();
people.add(new Person("Alice", 30));
people.add(new Person("Bob", 25));
// Automatically sorted by age

// Custom Comparator
TreeSet<Person> byName = new TreeSet<>(
    Comparator.comparing(Person::getName)
);
```

**Performance:**

| Operation | HashSet | LinkedHashSet | TreeSet |
|-----------|---------|---------------|---------|
| Add | O(1) | O(1) | O(log n) |
| Remove | O(1) | O(1) | O(log n) |
| Contains | O(1) | O(1) | O(log n) |
| Ordering | None | Insertion | Sorted |
| Memory | Less | More | Most |

### Map Interface

**Map**: Key-value pairs, keys are unique.

#### HashMap

**Implementation**: Hash table

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> map = new HashMap<>();

// Put entries
map.put("Alice", 30);     // Add: O(1) average
map.put("Bob", 25);
map.put("Alice", 35);     // Updates existing key

// Get values
Integer age = map.get("Alice");     // 35: O(1) average
Integer missing = map.get("Charlie");  // null (key doesn't exist)

// Check and retrieve safely
if (map.containsKey("Alice")) {
    age = map.get("Alice");
}

// getOrDefault (Java 8+)
age = map.getOrDefault("Charlie", 0);  // 0 (default value)

// putIfAbsent (Java 8+)
map.putIfAbsent("David", 40);  // Only adds if key doesn't exist

// Remove
map.remove("Bob");              // Remove key
map.remove("Alice", 30);        // Remove only if value matches (Java 8+)

// Size and checks
map.size();
map.isEmpty();
map.containsKey("Alice");
map.containsValue(30);

// Iteration
// 1. Entry set (most efficient)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// 2. Key set
for (String key : map.keySet()) {
    System.out.println(key + ": " + map.get(key));
}

// 3. Values
for (Integer value : map.values()) {
    System.out.println(value);
}

// Java 8+ forEach
map.forEach((key, value) -> 
    System.out.println(key + ": " + value)
);

// Compute operations (Java 8+)
map.compute("Alice", (key, oldValue) -> oldValue + 1);  // Increment
map.computeIfPresent("Bob", (key, oldValue) -> oldValue * 2);
map.computeIfAbsent("Charlie", key -> 20);  // Add if missing

// Merge (Java 8+)
map.merge("Alice", 5, (oldVal, newVal) -> oldVal + newVal);  // Add 5 to existing
```

**Common Pattern: Counting**

```java
// ❌ OLD WAY: Verbose
Map<String, Integer> wordCount = new HashMap<>();
for (String word : words) {
    if (wordCount.containsKey(word)) {
        wordCount.put(word, wordCount.get(word) + 1);
    } else {
        wordCount.put(word, 1);
    }
}

// ✅ NEW WAY: computeIfAbsent (Java 8+)
Map<String, Integer> wordCount = new HashMap<>();
for (String word : words) {
    wordCount.merge(word, 1, Integer::sum);
}

// ✅ OR: getOrDefault
for (String word : words) {
    wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
}
```

**HashMap Internals:**

```java
// Simplified HashMap
public class SimpleHashMap<K, V> {
    private Entry<K, V>[] table;  // Array of buckets
    private int size;
    
    static class Entry<K, V> {
        K key;
        V value;
        Entry<K, V> next;  // For collision handling (chaining)
        
        Entry(K key, V value, Entry<K, V> next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    
    public V put(K key, V value) {
        int hash = hash(key);
        int index = indexFor(hash, table.length);
        
        // Check if key exists
        for (Entry<K, V> e = table[index]; e != null; e = e.next) {
            if (e.key.equals(key)) {
                V oldValue = e.value;
                e.value = value;  // Update
                return oldValue;
            }
        }
        
        // Add new entry (collision: add to front of chain)
        table[index] = new Entry<>(key, value, table[index]);
        size++;
        return null;
    }
    
    public V get(K key) {
        int hash = hash(key);
        int index = indexFor(hash, table.length);
        
        for (Entry<K, V> e = table[index]; e != null; e = e.next) {
            if (e.key.equals(key)) {
                return e.value;
            }
        }
        return null;
    }
    
    private int hash(K key) {
        return key == null ? 0 : key.hashCode();
    }
    
    private int indexFor(int hash, int length) {
        return hash & (length - 1);  // Equivalent to hash % length
    }
}
```

#### LinkedHashMap

**Implementation**: Hash table + linked list (maintains insertion order or access order)

```java
// Insertion order (default)
Map<String, Integer> linkedMap = new LinkedHashMap<>();
linkedMap.put("C", 3);
linkedMap.put("A", 1);
linkedMap.put("B", 2);

for (String key : linkedMap.keySet()) {
    System.out.println(key);  // C, A, B (insertion order)
}

// Access order (useful for LRU cache)
Map<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
accessOrder.put("A", 1);
accessOrder.put("B", 2);
accessOrder.put("C", 3);

accessOrder.get("A");  // Access A

for (String key : accessOrder.keySet()) {
    System.out.println(key);  // B, C, A (A moved to end)
}

// LRU Cache implementation
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxSize;
    
    public LRUCache(int maxSize) {
        super(16, 0.75f, true);  // access-order
        this.maxSize = maxSize;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxSize;  // Remove oldest when size exceeded
    }
}

LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("1", "One");
cache.put("2", "Two");
cache.put("3", "Three");
cache.put("4", "Four");  // "1" is removed (least recently used)
```

#### TreeMap

**Implementation**: Red-Black tree (sorted by keys)

```java
import java.util.TreeMap;
import java.util.NavigableMap;

NavigableMap<String, Integer> treeMap = new TreeMap<>();

treeMap.put("Zebra", 1);
treeMap.put("Apple", 2);
treeMap.put("Mango", 3);

// Sorted by keys
for (String key : treeMap.keySet()) {
    System.out.println(key);  // Apple, Mango, Zebra
}

// NavigableMap operations
treeMap.firstKey();           // "Apple"
treeMap.lastKey();            // "Zebra"
treeMap.lowerKey("Mango");    // "Apple" (key < "Mango")
treeMap.higherKey("Mango");   // "Zebra" (key > "Mango")

// Range views
treeMap.headMap("Mango");     // Keys < "Mango"
treeMap.tailMap("Mango");     // Keys >= "Mango"
treeMap.subMap("Apple", "Zebra");  // Keys >= "Apple" and < "Zebra"

// Descending
NavigableMap<String, Integer> descending = treeMap.descendingMap();
```

### Queue and Deque

#### Queue Interface

**Queue**: FIFO (First-In-First-Out) collection

```java
import java.util.Queue;
import java.util.LinkedList;

Queue<String> queue = new LinkedList<>();

// Add elements
queue.offer("First");     // Preferred (returns false if full)
queue.add("Second");      // Throws exception if full

// Access head
String head = queue.peek();   // Returns null if empty
String head2 = queue.element();  // Throws exception if empty

// Remove head
String removed = queue.poll();   // Returns null if empty
String removed2 = queue.remove();  // Throws exception if empty

// Example: Task queue
Queue<Task> taskQueue = new LinkedList<>();
taskQueue.offer(new Task("Task 1"));
taskQueue.offer(new Task("Task 2"));

while (!taskQueue.isEmpty()) {
    Task task = taskQueue.poll();
    task.execute();
}
```

#### PriorityQueue

**Implementation**: Binary heap (elements ordered by priority)

```java
import java.util.PriorityQueue;

// Natural ordering (min-heap for integers)
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(2);
pq.offer(8);
pq.offer(1);

while (!pq.isEmpty()) {
    System.out.println(pq.poll());  // 1, 2, 5, 8 (ascending)
}

// Custom comparator (max-heap)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(2);
maxHeap.offer(8);

while (!maxHeap.isEmpty()) {
    System.out.println(maxHeap.poll());  // 8, 5, 2 (descending)
}

// Custom objects
class Task implements Comparable<Task> {
    String name;
    int priority;  // Lower number = higher priority
    
    @Override
    public int compareTo(Task other) {
        return Integer.compare(this.priority, other.priority);
    }
}

PriorityQueue<Task> taskQueue = new PriorityQueue<>();
taskQueue.offer(new Task("Low", 10));
taskQueue.offer(new Task("High", 1));
taskQueue.offer(new Task("Medium", 5));

// Processes in priority order
while (!taskQueue.isEmpty()) {
    Task task = taskQueue.poll();
    System.out.println(task.name);  // High, Medium, Low
}
```

#### Deque Interface

**Deque**: Double-ended queue (can add/remove from both ends)

```java
import java.util.Deque;
import java.util.ArrayDeque;

Deque<String> deque = new ArrayDeque<>();

// Add to front
deque.offerFirst("First");
deque.addFirst("New First");

// Add to back
deque.offerLast("Last");
deque.addLast("New Last");

// Remove from front
deque.pollFirst();
deque.removeFirst();

// Remove from back
deque.pollLast();
deque.removeLast();

// Peek
deque.peekFirst();
deque.peekLast();

// Stack operations (LIFO)
deque.push("Element");  // Add to front
deque.pop();            // Remove from front
deque.peek();           // Peek at front

// ✅ ArrayDeque as Stack (better than Stack class)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
System.out.println(stack.pop());  // 3 (LIFO)

// ✅ ArrayDeque as Queue (better than LinkedList for queues)
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.offer(2);
queue.offer(3);
System.out.println(queue.poll());  // 1 (FIFO)
```

**ArrayDeque vs LinkedList for Queue:**

| Aspect | ArrayDeque | LinkedList |
|--------|------------|------------|
| Performance | Faster | Slower |
| Memory | Less overhead | More overhead (nodes) |
| Null elements | Not allowed | Allowed |
| Recommendation | ✅ Preferred for queues/stacks | Use for List operations |

### Collections Utility Class

**Collections**: Static utility methods for collections

```java
import java.util.Collections;

List<Integer> list = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9));

// Sorting
Collections.sort(list);  // [1, 2, 5, 8, 9]
Collections.sort(list, Comparator.reverseOrder());  // [9, 8, 5, 2, 1]

// Searching (list must be sorted)
int index = Collections.binarySearch(list, 5);

// Shuffling
Collections.shuffle(list);  // Random order

// Reversing
Collections.reverse(list);

// Rotating
Collections.rotate(list, 2);  // Shift elements right by 2

// Min/Max
int min = Collections.min(list);
int max = Collections.max(list);

// Frequency
int count = Collections.frequency(list, 5);  // Count occurrences

// Fill
Collections.fill(list, 0);  // Replace all elements with 0

// Copy
List<Integer> dest = new ArrayList<>(Collections.nCopies(list.size(), 0));
Collections.copy(dest, list);

// Unmodifiable views
List<String> readOnly = Collections.unmodifiableList(list);
readOnly.add("New");  // UnsupportedOperationException

Set<String> readOnlySet = Collections.unmodifiableSet(set);
Map<String, Integer> readOnlyMap = Collections.unmodifiableMap(map);

// Synchronized wrappers (thread-safe)
List<String> syncList = Collections.synchronizedList(list);
Set<String> syncSet = Collections.synchronizedSet(set);
Map<String, Integer> syncMap = Collections.synchronizedMap(map);

// ⚠️ Must synchronize iteration manually
synchronized (syncList) {
    for (String item : syncList) {
        // Thread-safe iteration
    }
}

// Singleton collections
Set<String> singleton = Collections.singleton("Only");  // Immutable set with 1 element
List<String> singletonList = Collections.singletonList("Only");
Map<String, Integer> singletonMap = Collections.singletonMap("key", 1);

// Empty collections
List<String> emptyList = Collections.emptyList();
Set<String> emptySet = Collections.emptySet();
Map<String, Integer> emptyMap = Collections.emptyMap();
```

### Choosing the Right Collection

**Decision Tree:**

```
Need key-value pairs?
├─ No → Need duplicates?
│       ├─ Yes → Ordered?
│       │        ├─ Yes → ArrayList (default choice)
│       │        └─ No → HashSet
│       └─ No → Ordered?
│                ├─ Sorted → TreeSet
│                ├─ Insertion → LinkedHashSet
│                └─ No order → HashSet
└─ Yes → Need ordering?
          ├─ Sorted → TreeMap
          ├─ Insertion → LinkedHashMap
          └─ No order → HashMap (default choice)

Need queue/stack?
├─ Queue (FIFO) → ArrayDeque
├─ Stack (LIFO) → ArrayDeque (not Stack class!)
├─ Priority → PriorityQueue
└─ Both ends → ArrayDeque
```

**Performance Characteristics Summary:**

| Collection | Get | Add | Remove | Contains | Notes |
|------------|-----|-----|--------|----------|-------|
| ArrayList | O(1) | O(1)* | O(n) | O(n) | Fast random access |
| LinkedList | O(n) | O(1) | O(1)** | O(n) | Fast insert/delete at ends |
| HashSet | N/A | O(1) | O(1) | O(1) | No duplicates, no order |
| LinkedHashSet | N/A | O(1) | O(1) | O(1) | Insertion order |
| TreeSet | N/A | O(log n) | O(log n) | O(log n) | Sorted |
| HashMap | O(1) | O(1) | O(1) | O(1) | No order |
| LinkedHashMap | O(1) | O(1) | O(1) | O(1) | Insertion/access order |
| TreeMap | O(log n) | O(log n) | O(log n) | O(log n) | Sorted keys |

*O(1) amortized  
**O(1) if position known

### Frequently Asked Questions

**Q1: When should I use ArrayList vs LinkedList?**

**A:**

**Use ArrayList (99% of the time):**
* Default choice for List
* Fast random access: `get(index)` is O(1)
* Better cache locality (elements in contiguous memory)
* Less memory overhead

**Use LinkedList (rare cases):**
* Frequent insertions/deletions at beginning
* Implementing queue/deque
* Never need random access

```java
// ✅ ArrayList: Typical usage
List<String> names = new ArrayList<>();
// Read-heavy operations
for (int i = 0; i < names.size(); i++) {
    process(names.get(i));  // Fast!
}

// ✅ LinkedList: Queue operations
Queue<Task> taskQueue = new LinkedList<>();
taskQueue.offer(task);  // O(1)
taskQueue.poll();       // O(1)

// ❌ LinkedList: Slow random access
for (int i = 0; i < linkedList.size(); i++) {
    process(linkedList.get(i));  // O(n) for EACH get!
}
```

---

**Q2: Why should I program to interfaces (List, Set, Map) instead of concrete classes?**

**A:**

**Flexibility**: Can change implementation without changing code.

```java
// ❌ BAD: Tied to ArrayList
ArrayList<String> list = new ArrayList<>();
// If requirements change (need LinkedList), must change type everywhere

// ✅ GOOD: Program to interface
List<String> list = new ArrayList<>();
// Can easily change to LinkedList:
List<String> list = new LinkedList<>();
// No other code changes needed!

// Even better: Accept interface in methods
public void processList(List<String> list) {  // Not ArrayList!
    // Works with ArrayList, LinkedList, Vector, etc.
}
```

**Only use concrete type when you need specific methods:**

```java
// Need Deque-specific methods
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("First");  // Deque method
deque.addLast("Last");    // Deque method
```

---

**Q3: What's the difference between HashMap and Hashtable?**

**A:**

| Feature | HashMap | Hashtable |
|---------|---------|-----------|
| Synchronization | Not synchronized | Synchronized |
| Null keys/values | Allows one null key, any null values | No nulls |
| Performance | Faster | Slower |
| Iterator | Fail-fast | Fail-fast |
| Since | Java 1.2 | Java 1.0 (legacy) |
| Recommendation | ✅ Use HashMap | ❌ Obsolete |

```java
// ✅ Use HashMap (modern)
Map<String, Integer> map = new HashMap<>();
map.put(null, 1);  // OK
map.put("key", null);  // OK

// ❌ Avoid Hashtable (legacy)
Hashtable<String, Integer> table = new Hashtable<>();
table.put(null, 1);  // NullPointerException!

// If need thread-safety:
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
// Or use ConcurrentHashMap (better for concurrent access)
```

---

## Summary and Key Takeaways

Congratulations on completing Part 1 of the Complete Java Mastery series! You've built a solid foundation in Java fundamentals and object-oriented programming.

### What You've Learned

**Java Platform and OpenJDK:**
* Java's "Write Once, Run Anywhere" philosophy through bytecode and JVM
* OpenJDK as the free, open-source reference implementation
* JDK, JRE, and JVM architecture
* Java release cycle and LTS versions (11, 17, 21)

**Language Fundamentals:**
* Primitive types and wrapper classes
* String immutability and the String pool
* Type casting and autoboxing/unboxing
* Operators and operator precedence
* Control flow: if-else, switch expressions, loops
* Arrays and multi-dimensional arrays

**Object-Oriented Programming:**
* Classes, objects, and constructors
* Four pillars: Encapsulation, Inheritance, Polymorphism, Abstraction
* Access modifiers and the principle of least privilege
* Method overloading vs overriding
* Abstract classes vs interfaces
* Composition over inheritance

**Advanced OOP:**
* Nested and inner classes
* Enumerations with fields and methods
* Records for immutable data carriers (Java 16+)
* Sealed classes for controlled hierarchies (Java 17+)
* Modern Java features integration

**Exception Handling:**
* Exception hierarchy: Error, checked, and unchecked exceptions
* try-catch-finally blocks
* try-with-resources for automatic resource management
* Custom exceptions and exception chaining
* Best practices for robust error handling

**Collections Framework:**
* List implementations: ArrayList (fast random access) vs LinkedList (fast insertion/deletion)
* Set implementations: HashSet (fast), LinkedHashSet (ordered), TreeSet (sorted)
* Map implementations: HashMap (fast), LinkedHashMap (ordered), TreeMap (sorted)
* Queue and Deque: ArrayDeque for stacks and queues
* PriorityQueue for priority-based processing
* Collections utility class
* Choosing the right collection for your use case

### Critical Principles to Remember

1. **Program to Interfaces**: Use `List`, `Set`, `Map` instead of concrete classes
2. **Immutability**: Prefer immutable objects (String, records) for thread safety and reliability
3. **Fail Fast**: Validate inputs and throw exceptions early
4. **Resource Management**: Always use try-with-resources for closeable resources
5. **Encapsulation**: Keep fields private, expose through methods
6. **equals() and hashCode() Contract**: Override both together, maintain consistency
7. **Performance**: Understand time complexity of operations
8. **Modern Java**: Leverage new features (records, sealed classes, pattern matching)

### Common Pitfalls to Avoid

```java
// ❌ Comparing objects with ==
String s1 = new String("Hello");
String s2 = new String("Hello");
if (s1 == s2) // false!

// ✅ Use equals()
if (s1.equals(s2)) // true

// ❌ Modifying collection during iteration
for (String item : list) {
    list.remove(item); // ConcurrentModificationException!
}

// ✅ Use Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (condition) it.remove();
}

// ❌ Not overriding hashCode() when overriding equals()
@Override
public boolean equals(Object obj) { /* ... */ }
// HashMap won't work correctly!

// ✅ Override both
@Override
public boolean equals(Object obj) { /* ... */ }
@Override
public int hashCode() { return Objects.hash(fields); }
```

### SDLC Context

Throughout this part, we've integrated Software Development Life Cycle considerations:

**Requirements Phase:**
* Understanding when to use different data structures based on requirements
* Choosing between checked and unchecked exceptions based on error handling needs

**Design Phase:**
* Applying SOLID principles through OOP
* Using design patterns (although covered more in later parts)
* Choosing appropriate collection types for algorithms

**Development Phase:**
* Writing clean, maintainable code with proper encapsulation
* Following Java naming conventions
* Using modern Java features appropriately

**Testing Phase:**
* Writing testable code through interfaces and dependency injection
* Understanding exception handling for test scenarios
* Validating edge cases with collections

### Comprehensive Final FAQ

**Q1: What's the difference between JDK, JRE, and JVM?**

**A:** 
* **JVM**: Executes bytecode, manages memory, provides runtime environment
* **JRE**: JVM + core libraries (everything to run Java applications)
* **JDK**: JRE + development tools like javac, debugger (everything to develop Java applications)

As a developer, you install the JDK.

---

**Q2: Why is String immutable?**

**A:** Security, thread safety, String pool optimization, and hash code caching. Immutability makes Strings safe to use as HashMap keys and share between threads.

---

**Q3: What's the difference between == and equals()?**

**A:** 
* `==`: Compares object references (memory addresses)
* `equals()`: Compares object content (logical equality)

Always use `equals()` for String and object comparison.

---

**Q4: When should I use ArrayList vs LinkedList?**

**A:** 
* **ArrayList** (99% of cases): Fast random access, less memory overhead
* **LinkedList** (rare): Only when frequently adding/removing at beginning/end

---

**Q5: What's the difference between HashMap and Hashtable?**

**A:** HashMap is modern, faster, allows null, not synchronized. Hashtable is legacy, slower, no nulls, synchronized. **Always use HashMap** (or ConcurrentHashMap for thread-safety).

---

**Q6: Why must I override both equals() and hashCode()?**

**A:** Hash-based collections (HashMap, HashSet) use hashCode() to find bucket, then equals() to find object. If hashCode() isn't overridden, equal objects may have different hash codes, breaking these collections.

---

**Q7: What's the difference between abstract class and interface?**

**A:**
* **Abstract class**: Can have state (fields), concrete methods, single inheritance
* **Interface**: No state (only constants), all methods abstract (except default/static), multiple inheritance

Use abstract class for IS-A with shared implementation, interface for CAN-DO capability.

---

**Q8: What are checked vs unchecked exceptions?**

**A:**
* **Checked**: Must be declared or caught, recoverable conditions (IOException, SQLException)
* **Unchecked**: Optional to handle, programming bugs (NullPointerException, IllegalArgumentException)

---

**Q9: How does HashMap work internally?**

**A:** Uses array of buckets. Hash code determines bucket, equals() finds object within bucket. Collisions handled by chaining (linked list, tree for many collisions). O(1) average time for get/put.

---

**Q10: What's the difference between ArrayList and Vector?**

**A:** Vector is legacy, synchronized (thread-safe but slow). ArrayList is modern, not synchronized (faster). **Use ArrayList** and synchronize externally if needed, or use ConcurrentModifications.

---

### Final Interview Questions

**Question 1: Implement a simple LRU Cache using LinkedHashMap**

**Difficulty:** Mid-Level

**Topics:** Collections, Design

**Answer:**

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxSize;
    
    public LRUCache(int maxSize) {
        // 16 = initial capacity, 0.75f = load factor, true = access-order
        super(16, 0.75f, true);
        this.maxSize = maxSize;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxSize;
    }
}

// Usage
LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("1", "One");
cache.put("2", "Two");
cache.put("3", "Three");
cache.get("1");  // Access "1", moves it to end
cache.put("4", "Four");  // "2" is removed (least recently used)
```

**Why This Matters:** Demonstrates understanding of LinkedHashMap's access-order mode and practical caching implementation.

---

**Question 2: Why can't we override static methods?**

**Difficulty:** Junior

**Topics:** OOP, Polymorphism

**Answer:**

Static methods belong to the class, not instances, so they're resolved at compile-time (static binding), not runtime (dynamic binding). They can be hidden but not overridden.

```java
class Parent {
    public static void staticMethod() {
        System.out.println("Parent static");
    }
}

class Child extends Parent {
    public static void staticMethod() {  // Hiding, not overriding
        System.out.println("Child static");
    }
}

Parent p = new Child();
p.staticMethod();  // Prints "Parent static" (compile-time type)

Child c = new Child();
c.staticMethod();  // Prints "Child static"
```

**Why This Matters:** Understanding static binding vs dynamic binding is fundamental to polymorphism.

---

**Question 3: What happens if you don't close a resource in try-with-resources?**

**Difficulty:** Mid-Level

**Topics:** Exception Handling

**Answer:**

try-with-resources **automatically** closes resources that implement AutoCloseable, even if an exception occurs. The close() method is called in reverse order of resource creation, and suppressed exceptions are handled properly.

```java
try (FileReader fr = new FileReader("file.txt");
     BufferedReader br = new BufferedReader(fr)) {
    // Use resources
}  // Both br and fr are automatically closed (br first, then fr)
```

If close() throws an exception, it's added as a suppressed exception to the main exception.

**Why This Matters:** Prevents resource leaks and simplifies error handling.

---

### What's Next

In **Part 2: JVM Internals & Memory Management**, we'll explore:
* JVM architecture and components
* Class loading mechanism
* Memory model and heap organization
* Garbage collection algorithms (G1GC, ZGC, Shenandoah)
* JIT compilation and optimization
* Performance tuning and monitoring

In **Part 3: Advanced Java Features & Modern Java**, we'll cover:
* Concurrency and multithreading
* Virtual threads (Project Loom)
* Annotations and reflection
* Modern Java features (Java 9-21)
* Streams and functional programming
* Date/Time API

Continue your journey to Java mastery with the next parts of this series!

### References

* [OpenJDK Official Site](https://openjdk.org/)
* [Java Language Specification](https://docs.oracle.com/javase/specs/)
* [Java API Documentation](https://docs.oracle.com/en/java/javase/21/docs/api/)
* [Effective Java by Joshua Bloch](https://www.pearson.com/store/p/effective-java/P100000801549)
* [Java Concurrency in Practice by Brian Goetz](https://jcip.net/)

---

**End of Part 1: Java Fundamentals & Language Basics**

*Next: Part 2 - JVM Internals & Memory Management*
