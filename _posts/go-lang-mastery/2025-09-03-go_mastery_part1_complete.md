---
title: "Complete Go Mastery Part 1: Go Fundamentals & Language Design"
date: 2025-01-10 10:00:00 +0530
categories: [Programming, Go, Backend]
tags: [golang, go, fundamentals, types, functions, interfaces, packages, modules]
---

# Complete Go Mastery Part 1: Go Fundamentals & Language Design

## Introduction

Welcome to the first part of the Complete Go Mastery series. This comprehensive guide will take you from absolute beginner to production-ready Go developer, covering every aspect of the language, its runtime, and the entire software development lifecycle.

Go (often referred to as Golang) is a statically typed, compiled programming language designed at Google by Robert Griesemer, Rob Pike, and Ken Thompson. Since its public release in 2009, Go has become the language of choice for cloud infrastructure, distributed systems, microservices, and DevOps tooling.

**What makes Go special?**

Go was designed to address real problems Google faced with existing languages: slow build times, unnecessary complexity, difficulty in writing concurrent programs, and poor dependency management. The result is a language that prioritizes simplicity, readability, and practical engineering over theoretical elegance.

**Who is this series for?**

This series targets developers at all levels who want to truly master Go—not just write code that works, but code that's idiomatic, performant, maintainable, and production-ready. Whether you're preparing for interviews, building distributed systems, or transitioning from another language, this guide provides the deep understanding you need.

**What you'll learn in Part 1:**

In this first part, we'll cover the foundational elements of Go: its design philosophy, basic syntax, type system, control flow, functions, interfaces, and package management. Each concept will be explained not just in terms of syntax, but with insights into why Go works this way, how it's implemented internally, and how to use it idiomatically.

By the end of Part 1, you'll have a solid foundation in Go fundamentals and understand the philosophy that guides all Go programming.

---

## 1.1 Introduction to Go

### What is Go and Why Was It Created?

Go emerged from frustration with existing programming languages at Google around 2007. The company was dealing with massive codebases written primarily in C++ and Java, and developers were encountering several pain points:

1. **Compilation times were unbearable** - Large C++ projects could take hours to compile
2. **Dependencies were a mess** - C++ header inclusion was exponential and uncontrolled
3. **Concurrency was hard** - Writing correct multithreaded code in C++ and Java was error-prone
4. **Too much complexity** - Modern C++ had become incredibly complex with features upon features
5. **Slow developer onboarding** - It took too long for new engineers to become productive

Rob Pike describes the creation of Go in a famous talk: while waiting for a large C++ compilation, three Google engineers started designing a new language that would eliminate these problems.

**The Core Design Goals:**

Go was designed with specific, practical objectives:

- **Fast compilation** - Programs should compile in seconds, not minutes or hours
- **Simplicity** - The language should be simple enough to fit in your head
- **Concurrency** - Concurrent programming should be a first-class citizen, not an afterthought
- **Safety** - Memory safety, type safety, but without sacrificing too much performance
- **Good tooling** - The language should come with batteries included
- **Fast execution** - Performance should be comparable to C/C++
- **Productive** - Developers should be productive quickly

### Go's Design Philosophy

Understanding Go's philosophy is crucial to writing idiomatic Go code. Go makes deliberate trade-offs that distinguish it from other languages:

#### **Simplicity Over Features**

Go intentionally omits features found in other modern languages:
- No classes or inheritance (composition over inheritance)
- No generics until Go 1.18 (and even then, implemented conservatively)
- No exceptions (explicit error handling)
- No operator overloading
- No default parameters
- No macros or template metaprogramming
- Limited type inference

This isn't an oversight—it's by design. The Go team believes that language features have a cost: they make the language harder to learn, harder to read, and harder to maintain. Every feature must justify its inclusion by solving real problems that can't be solved simply otherwise.

```go
// ❌ NOT ALLOWED IN GO: Method overloading
type Calculator struct{}

func (c Calculator) Add(a, b int) int {
    return a + b
}

// This won't compile - no method overloading
// func (c Calculator) Add(a, b, c int) int {
//     return a + b + c
// }

// ✅ THE GO WAY: Explicit naming
func (c Calculator) Add(a, b int) int {
    return a + b
}

func (c Calculator) AddThree(a, b, c int) int {
    return a + b + c
}
```

**Why this matters:** Code should be easy to read and understand. When you see a function call, you should know exactly which function is being called without consulting complex overload resolution rules.

#### **Explicit Over Implicit**

Go favors explicitness over magic:
- Errors are returned as values, not thrown
- No implicit type conversions (even int to int64 requires explicit casting)
- No implicit interface implementation
- Imports must be used or the code won't compile
- Variables must be used or the code won't compile

```go
// ❌ BAD: Implicit type conversion (won't compile in Go)
var i int = 42
var f float64 = i  // Compiler error!

// ✅ GOOD: Explicit conversion
var i int = 42
var f float64 = float64(i)  // Clear and explicit
```

#### **Composition Over Inheritance**

Go doesn't have classes or inheritance. Instead, it uses composition through struct embedding and interfaces:

```go
// Instead of inheritance, Go uses composition
type Animal struct {
    Name string
    Age  int
}

func (a Animal) Speak() string {
    return "Some sound"
}

// Dog "inherits" from Animal through embedding
type Dog struct {
    Animal  // Embedded struct - Dog has all Animal fields and methods
    Breed string
}

func (d Dog) Speak() string {
    return "Woof!"  // Override the embedded method
}

func main() {
    dog := Dog{
        Animal: Animal{Name: "Buddy", Age: 3},
        Breed:  "Golden Retriever",
    }
    
    // Can access embedded fields directly
    fmt.Println(dog.Name)    // "Buddy"
    fmt.Println(dog.Speak()) // "Woof!"
}
```

**Why this matters:** Composition is more flexible than inheritance. You can compose behaviors at runtime, and there's no fragile base class problem. Your code structure mirrors real-world relationships more naturally.

#### **Concurrency as a First-Class Citizen**

Go was designed for the multicore era. Concurrency isn't bolted on—it's fundamental:

```go
// ✅ Concurrency in Go is simple and built-in
func main() {
    // Start a concurrent function (goroutine)
    go fetchData()
    
    // Continue with other work
    processLocalData()
}

func fetchData() {
    // This runs concurrently
    resp, _ := http.Get("https://api.example.com/data")
    defer resp.Body.Close()
    // ... process response
}
```

This simplicity hides sophisticated runtime machinery that we'll explore in Part 2, but the key insight is: **in Go, starting concurrent operations should be as easy as calling a function**.

### Go vs Other Languages

Understanding where Go fits in the language ecosystem helps you know when to use it:

#### **Go vs C/C++**

| Aspect | C/C++ | Go |
|--------|-------|-----|
| **Memory Management** | Manual (malloc/free) | Garbage collected |
| **Compilation Speed** | Slow for large projects | Fast (seconds) |
| **Complexity** | Very high (especially C++) | Low to medium |
| **Performance** | Maximum possible | Very close, ~90-95% |
| **Concurrency** | Threads, manual | Goroutines, channels |
| **Safety** | Prone to memory errors | Memory safe |
| **Use Case** | Systems programming, embedded | Cloud services, tools |

**When to choose Go over C/C++:**
- When development speed matters more than last 5-10% performance
- When you need garbage collection and memory safety
- When building network services or tools
- When team size is large and maintainability matters

**When to choose C/C++ over Go:**
- When you need maximum performance (HFT, game engines)
- When you're targeting embedded systems
- When you need zero-cost abstractions
- When GC pauses are unacceptable

#### **Go vs Java**

| Aspect | Java | Go |
|--------|------|-----|
| **Startup Time** | Slow (JVM warmup) | Fast (native binary) |
| **Memory Usage** | Higher (JVM overhead) | Lower |
| **Compilation** | To bytecode | To native binary |
| **Deployment** | JAR/WAR + JVM | Single static binary |
| **Generics** | Full generics since 1.5 | Limited generics since 1.18 |
| **OOP** | Class-based inheritance | Composition-based |
| **Ecosystem** | Massive, mature | Growing rapidly |

**When to choose Go over Java:**
- When deployment simplicity matters (single binary)
- When startup time is critical (CLI tools, serverless)
- When you want simpler concurrency (goroutines vs threads)
- When you prefer composition over inheritance

**When to choose Java over Go:**
- When you need mature enterprise frameworks
- When you need complex generics and type system
- When your team is already Java-expert
- When you need specific Java ecosystem libraries

#### **Go vs Python**

| Aspect | Python | Go |
|--------|--------|-----|
| **Type System** | Dynamic | Static |
| **Performance** | Slow (interpreted) | Fast (compiled) |
| **Concurrency** | GIL limits parallelism | True parallelism |
| **Learning Curve** | Very gentle | Moderate |
| **Deployment** | Requires interpreter | Single binary |
| **Use Case** | Scripts, ML, data science | Services, tools, infrastructure |

**When to choose Go over Python:**
- When performance matters (Go is 10-100x faster typically)
- When you need type safety and compile-time checks
- When building production services
- When true parallelism is needed

**When to choose Python over Go:**
- For rapid prototyping and scripts
- For data science and machine learning (ecosystem)
- When development speed matters more than execution speed
- For simple automation tasks

#### **Go vs Rust**

| Aspect | Rust | Go |
|--------|------|-----|
| **Memory Management** | Ownership system, no GC | Garbage collected |
| **Learning Curve** | Steep | Moderate |
| **Compilation Speed** | Slow | Fast |
| **Performance** | Maximum (with zero-cost abstractions) | Very high |
| **Concurrency** | Fearless (ownership prevents data races) | Easy (runtime prevents most issues) |
| **Safety** | Compile-time guarantees | Runtime safety |

**When to choose Go over Rust:**
- When development speed is more important than maximum performance
- When you need faster compilation times
- When team velocity matters (easier onboarding)
- When GC pauses are acceptable (most services)

**When to choose Rust over Go:**
- When you need guaranteed memory safety without GC
- When maximum performance is critical
- When you can't tolerate GC pauses
- For systems programming, embedded systems

### The Go Ecosystem and Community

**Official Resources:**
- **golang.org** - Official website with documentation
- **go.dev** - Modern Go documentation and learning resources
- **pkg.go.dev** - Package documentation
- **Go Blog** - Official blog with in-depth articles
- **Go Playground** - Online Go environment

**Community:**
- Active Gophers Slack community
- r/golang on Reddit
- Golang Weekly newsletter
- GopherCon (annual conference)
- Many local Go meetups worldwide

**Who Uses Go in Production:**
- Google (Kubernetes, many services)
- Docker (containerization platform)
- Cloudflare (edge services)
- Uber (microservices)
- Twitch (chat, video infrastructure)
- Dropbox (migration from Python)
- Netflix (various tools)
- Twitter (parts of infrastructure)

### Setting Up Go Development Environment (Go 1.24)

#### **Installation**

**Linux/macOS:**
```bash
# Download and install Go 1.24
wget https://go.dev/dl/go1.24.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.24.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin

# Verify installation
go version  # Should show: go version go1.24 linux/amd64
```

**Windows:**
Download the MSI installer from golang.org and run it. The installer will set up PATH automatically.

**macOS (using Homebrew):**
```bash
brew install go@1.24
```

#### **Understanding GOPATH and GOROOT**

**GOROOT:**
- Location where Go is installed
- Contains the standard library and toolchain
- Usually `/usr/local/go` on Linux/Mac
- You rarely need to set this manually

```bash
go env GOROOT  # See your GOROOT
```

**GOPATH (Legacy, mostly deprecated with modules):**
- Historical workspace for Go code
- Defaults to `~/go`
- With Go modules (1.11+), GOPATH is less important
- Still used for caching downloaded modules (`$GOPATH/pkg/mod`)

```bash
go env GOPATH  # See your GOPATH
```

**Go Modules (Modern Approach - Go 1.11+):**
Go modules replaced GOPATH-based development. You can now write Go code anywhere:

```bash
# Create a new project anywhere
mkdir ~/projects/myapp
cd ~/projects/myapp

# Initialize a module
go mod init github.com/yourusername/myapp

# This creates go.mod file - your project is now a module!
```

#### **Essential Go Environment Variables**

```bash
# See all Go environment variables
go env

# Key variables:
GO111MODULE=on      # Use modules (default in Go 1.16+)
GOROOT=/usr/local/go
GOPATH=/home/user/go
GOCACHE=/home/user/.cache/go-build
GOMODCACHE=/home/user/go/pkg/mod
GOPROXY=https://proxy.golang.org,direct
GOSUMDB=sum.golang.org
```

#### **Setting Up Your Editor**

**VS Code (Recommended):**
1. Install VS Code
2. Install "Go" extension by Go Team at Google
3. Open a .go file - extension will prompt to install gopls (language server)
4. Install suggested tools (gopls, dlv, staticcheck, etc.)

**Vim/Neovim:**
- Use vim-go plugin
- Or use coc.nvim with coc-go

**GoLand (JetBrains):**
- Full-featured IDE specifically for Go
- Commercial, but excellent for large projects

**Essential Tools (will be installed by editor extensions):**
- `gopls` - Language server
- `dlv` - Debugger
- `staticcheck` - Linter
- `goimports` - Import management and formatting
- `gopls` - Go language server

### First Go Program: Hello World Explained Deeply

Let's write the traditional "Hello, World!" but understand every single line:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Save this as `hello.go` and run:
```bash
go run hello.go
# Output: Hello, World!
```

**Line-by-line breakdown:**

**1. `package main`**

Every Go file must belong to a package. Packages are Go's way of organizing code:

- `package main` is special - it defines an executable program
- Any other package name creates a library, not an executable
- The package name must match the directory name (except for `main`)
- A directory can only contain files from one package (except `_test` packages)

```go
// This file creates an executable
package main

// These files create libraries
package utils
package database
package handlers
```

**2. `import "fmt"`**

The import statement brings in other packages:

- `fmt` is from the standard library (formatting/printing)
- Import paths are strings
- Unused imports cause compilation errors (Go's opinionated stance)
- Multiple imports use parentheses:

```go
import (
    "fmt"
    "os"
    "time"
)
```

**3. `func main()`**

- `func` keyword declares a function
- `main` is the entry point of the program (like C/C++/Java)
- Must be in `package main`
- Takes no arguments, returns nothing
- When the program runs, `main()` is called automatically

**4. `fmt.Println("Hello, World!")`**

- Calls the `Println` function from the `fmt` package
- Capital 'P' means this function is exported (public)
- Lowercase would be unexported (private to the package)
- `Println` prints and adds a newline

**Building vs Running:**

```bash
# Run without building (compiles to temp location)
go run hello.go

# Build an executable
go build hello.go
# Creates 'hello' binary (or hello.exe on Windows)
./hello

# Build with custom output name
go build -o myapp hello.go
./myapp

# Install to $GOPATH/bin
go install hello.go
```

### Understanding Go's Compilation Model

Go is a compiled language, but what does that mean?

**Compilation Process:**

```
Source Code (.go)
    ↓
Lexical Analysis (tokenization)
    ↓
Parsing (syntax tree)
    ↓
Type Checking
    ↓
SSA (Static Single Assignment) form
    ↓
Optimizations
    ↓
Machine Code
    ↓
Linking
    ↓
Executable Binary
```

**Key Characteristics:**

1. **Fast Compilation:**
   - Go's import system is designed for fast compilation
   - Each package is compiled once
   - No header files to parse repeatedly
   - Parallel compilation of packages

2. **Static Linking:**
   - By default, Go creates statically linked binaries
   - All dependencies are included in the executable
   - No runtime dependencies needed (except libc for cgo programs)
   - Great for deployment: just copy one file

3. **Cross-Compilation:**

Go makes it trivial to compile for different platforms:

```bash
# Build for Linux AMD64 from any platform
GOOS=linux GOARCH=amd64 go build

# Build for Windows
GOOS=windows GOARCH=amd64 go build

# Build for macOS ARM (M1/M2)
GOOS=darwin GOARCH=arm64 go build

# Build for Raspberry Pi
GOOS=linux GOARCH=arm GOARM=7 go build
```

**Supported platforms:**
```bash
go tool dist list  # Shows all supported OS/ARCH combinations
```

**Build tags and conditional compilation:**

```go
// +build linux darwin

package main

// This code only compiles on Linux and macOS
```

Or using the newer Go 1.17+ syntax:

```go
//go:build linux || darwin

package main

// Same as above, clearer syntax
```

### Key Takeaways

- Go was designed to solve real problems at Google: slow compilation, complex dependencies, difficult concurrency
- Go prioritizes simplicity over features - if something can be done simply, it won't be added
- Composition over inheritance: Go uses struct embedding and interfaces instead of classes
- Explicit over implicit: No magic, clear intent
- Go compiles to native binaries, fast compilation, easy cross-compilation
- Modern Go uses modules (go.mod) - no more GOPATH constraints
- `package main` with `func main()` creates an executable

---

## 1.2 Basic Syntax and Types

Understanding Go's type system is fundamental to writing correct, efficient Go code. Go's type system is deliberately simple compared to languages like C++ or Rust, but it's powerful enough for building complex systems.

### Variables and Constants

#### **Variable Declaration**

Go provides multiple ways to declare variables, each with specific use cases:

**Method 1: `var` keyword (traditional)**

```go
// Declare with type, zero value initialization
var name string
var age int
var isActive bool

fmt.Println(name)     // "" (empty string - zero value)
fmt.Println(age)      // 0 (zero value for int)
fmt.Println(isActive) // false (zero value for bool)

// Declare with initialization
var name string = "Alice"
var age int = 30
var isActive bool = true
```

**Method 2: Type inference with `var`**

```go
// Compiler infers the type
var name = "Alice"      // string
var age = 30            // int
var temperature = 98.6  // float64
```

**Method 3: Short declaration `:=` (most common in functions)**

```go
// Only works inside functions, not at package level
func example() {
    name := "Alice"
    age := 30
    temperature := 98.6
    
    // Multiple declarations
    x, y := 10, 20
    firstName, lastName := "John", "Doe"
}
```

**Method 4: Multiple declarations**

```go
// Group related variables
var (
    name    string
    age     int
    isActive bool
)

// With initialization
var (
    name     = "Alice"
    age      = 30
    isActive = true
)

// Mixed types
var (
    name     string
    age      int = 25
    height   = 5.9
)
```

**Common Pitfall: := vs =**

```go
var name string = "Alice"

func updateName() {
    // ❌ BAD: Creates a new local variable, shadowing the package-level one
    name := "Bob"
    fmt.Println(name) // "Bob"
}

func updateNameCorrectly() {
    // ✅ GOOD: Updates the existing variable
    name = "Bob"
    fmt.Println(name) // "Bob"
}
```

**When to use each method:**

- Use `var` for package-level variables
- Use `var` when you want to declare without initialization (relying on zero value)
- Use `:=` inside functions for local variables (most common)
- Use `var` with explicit type when clarity is needed over inference

#### **Understanding Zero Values**

One of Go's most important concepts: **every type has a zero value**. Variables declared without initialization automatically receive their type's zero value:

```go
var i int           // 0
var f float64       // 0.0
var b bool          // false
var s string        // "" (empty string)
var p *int          // nil
var slice []int     // nil
var m map[string]int // nil
var fn func()       // nil
var ch chan int     // nil
var iface interface{} // nil
```

**Why zero values matter:**

1. **No uninitialized variables** - Unlike C/C++, you can't have random memory
2. **Predictable behavior** - You know exactly what an uninitialized variable contains
3. **Useful defaults** - Zero values are often useful (empty string, 0, false, nil)

```go
// Example: Zero values are often useful
type Counter struct {
    count int  // Automatically 0, which is exactly what we want
}

c := Counter{}
fmt.Println(c.count) // 0 - ready to use!
```

#### **Constants and iota**

Constants are immutable values known at compile time:

```go
// Simple constants
const Pi = 3.14159
const AppName = "MyApp"
const MaxConnections = 100

// Typed constants
const MaxInt int = 100
const Timeout time.Duration = 30 * time.Second

// Multiple constants
const (
    StatusOK       = 200
    StatusNotFound = 404
    StatusError    = 500
)
```

**iota - The Enumeration Generator**

`iota` is a special constant generator that increments for each constant in a const block:

```go
const (
    Sunday = iota    // 0
    Monday           // 1
    Tuesday          // 2
    Wednesday        // 3
    Thursday         // 4
    Friday           // 5
    Saturday         // 6
)

// Skip values
const (
    _  = iota  // Skip 0
    KB = 1 << (10 * iota)  // 1 << 10 = 1024
    MB                      // 1 << 20 = 1048576
    GB                      // 1 << 30 = 1073741824
    TB                      // 1 << 40
)

// Multiple expressions
const (
    a, b = iota, iota + 10   // 0, 10
    c, d                      // 1, 11
    e, f                      // 2, 12
)
```

**Typed vs Untyped Constants**

Go has a unique concept of untyped constants that are flexible:

```go
const x = 42  // Untyped constant

var i int = x
var f float64 = x
var c complex128 = x

// All valid! x adapts to the target type

const y int = 42  // Typed constant
var f2 float64 = y  // ❌ Compiler error: cannot use y (type int) as float64
```

**Why this matters:** Untyped constants make numeric code more flexible without sacrificing type safety.

#### **Scope and Shadowing**

Go has lexical scoping with block-level scope:

```go
package main

var x = "package level"

func main() {
    fmt.Println(x) // "package level"
    
    x := "function level"  // Shadows package-level x
    fmt.Println(x) // "function level"
    
    {
        x := "block level"  // Shadows function-level x
        fmt.Println(x) // "block level"
    }
    
    fmt.Println(x) // "function level"
}
```

**Shadowing Detection:**
Modern linters (like `shadow` checker in `go vet`) can detect unintentional shadowing:

```bash
go vet -vettool=$(which shadow) ./...
```

**Common shadowing pitfall:**

```go
var config *Config

func loadConfig() error {
    config, err := ReadConfig("config.json")  // ❌ Shadows package-level config
    if err != nil {
        return err
    }
    // package-level config is still nil!
    return nil
}

// ✅ Correct version
func loadConfig() error {
    var err error
    config, err = ReadConfig("config.json")  // Updates package-level config
    if err != nil {
        return err
    }
    return nil
}
```

### Basic Types

#### **Numeric Types**

Go has a rich set of numeric types with explicit sizes:

**Integers (signed):**
```go
int8    // -128 to 127
int16   // -32768 to 32767
int32   // -2147483648 to 2147483647
int64   // -9223372036854775808 to 9223372036854775807
int     // Platform dependent: 32 or 64 bits
```

**Integers (unsigned):**
```go
uint8   // 0 to 255
uint16  // 0 to 65535
uint32  // 0 to 4294967295
uint64  // 0 to 18446744073709551615
uint    // Platform dependent: 32 or 64 bits
```

**Floating point:**
```go
float32 // IEEE-754 32-bit floating point
float64 // IEEE-754 64-bit floating point
```

**Complex numbers:**
```go
complex64  // float32 real and imaginary parts
complex128 // float64 real and imaginary parts

// Usage
c := 3 + 4i
fmt.Println(real(c))  // 3
fmt.Println(imag(c))  // 4
```

**Special numeric types:**
```go
byte    // Alias for uint8 (for byte data)
rune    // Alias for int32 (for Unicode code points)
uintptr // Unsigned integer large enough to hold a pointer value
```

**Which type to use?**

```go
// ✅ GOOD: Use specific types when size matters
type Pixel struct {
    R, G, B uint8  // Only need 0-255
}

// ✅ GOOD: Use int for general counting, indexing
for i := 0; i < len(slice); i++ {  // i is int
    // ...
}

// ✅ GOOD: Use int64 for timestamps, large numbers
timestamp := time.Now().Unix()  // int64

// ✅ GOOD: Use float64 for most floating point (more precision)
pi := 3.14159265359  // float64

// ❌ AVOID: Don't use platform-dependent int/uint in structs for serialization
type Message struct {
    ID int  // Bad: size varies by platform
}

// ✅ BETTER: Use explicit size
type Message struct {
    ID int64  // Good: consistent size everywhere
}
```

**Numeric operations and overflow:**

```go
var x uint8 = 255
x++  // Wraps around to 0 (no error!)

var y int8 = 127
y++  // Wraps around to -128 (no error!)

// ✅ Check for overflow manually if needed
func safeAdd(a, b uint32) (uint32, error) {
    if a > math.MaxUint32 - b {
        return 0, errors.New("overflow")
    }
    return a + b, nil
}
```

#### **Strings and Runes (UTF-8 Handling)**

Go strings are **immutable** sequences of **bytes** (not characters!). Go source code is UTF-8, and string literals are UTF-8 encoded.

```go
// String basics
s := "Hello, 世界"
fmt.Println(len(s))  // 13 (bytes, not characters!)

// Indexing gives bytes
fmt.Println(s[0])  // 72 (byte value of 'H')

// Iterate over bytes
for i := 0; i < len(s); i++ {
    fmt.Printf("%d ", s[i])  // Prints byte values
}

// Iterate over runes (characters)
for i, r := range s {
    fmt.Printf("Index: %d, Rune: %c\n", i, r)
}
// Output:
// Index: 0, Rune: H
// Index: 1, Rune: e
// Index: 2, Rune: l
// Index: 3, Rune: l
// Index: 4, Rune: o
// Index: 5, Rune: ,
// Index: 6, Rune:  
// Index: 7, Rune: 世
// Index: 10, Rune: 界
```

**Understanding the difference: bytes vs runes**

```go
s := "Hello, 世界"

// len() gives bytes
fmt.Println(len(s))  // 13 bytes

// To count runes (characters)
fmt.Println(utf8.RuneCountInString(s))  // 9 runes

// String to []byte
bytes := []byte(s)

// String to []rune
runes := []rune(s)
fmt.Println(len(runes))  // 9
```

**String immutability:**

```go
s := "hello"
// s[0] = 'H'  // ❌ Compiler error: cannot assign to s[0]

// ✅ To "modify" a string, create a new one
bytes := []byte(s)
bytes[0] = 'H'
s = string(bytes)  // "Hello"

// Or use strings.Builder for efficient concatenation
var builder strings.Builder
builder.WriteString("Hello")
builder.WriteString(", ")
builder.WriteString("World")
result := builder.String()  // "Hello, World"
```

**String operations:**

```go
// Concatenation
s := "Hello" + " " + "World"  // Creates new string

// Efficient concatenation in loops
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString("a")  // Much better than s += "a"
}
result := builder.String()

// Common operations (strings package)
strings.Contains("hello", "ell")      // true
strings.HasPrefix("hello", "he")      // true
strings.HasSuffix("hello", "lo")      // true
strings.Index("hello", "l")           // 2
strings.Split("a,b,c", ",")           // ["a", "b", "c"]
strings.Join([]string{"a","b"}, ",") // "a,b"
strings.ToUpper("hello")              // "HELLO"
strings.TrimSpace("  hello  ")        // "hello"
```

**Raw string literals:**

```go
// Regular string (interpreted)
s1 := "Hello\nWorld"  // Newline is interpreted

// Raw string literal (backticks)
s2 := `Hello\nWorld`  // Backslash-n as literal text
s3 := `Line 1
Line 2
Line 3`  // Preserves newlines

// Useful for regex, SQL, JSON templates
sqlQuery := `
    SELECT id, name, email
    FROM users
    WHERE age > ?
    ORDER BY created_at DESC
`
```

#### **Booleans**

Simple true/false values:

```go
var b bool  // false (zero value)
b = true

// Logical operators
result := true && false  // false (AND)
result = true || false   // true (OR)
result = !true           // false (NOT)

// Comparison operators return bool
x := 5
y := 10
fmt.Println(x == y)  // false
fmt.Println(x != y)  // true
fmt.Println(x < y)   // true
fmt.Println(x >= y)  // false
```

**No truthiness:**

Unlike many languages, Go has no concept of "truthy" or "falsy" values:

```go
// ❌ These don't compile
if 1 { }           // Error: non-bool condition
if "hello" { }     // Error: non-bool condition
if ptr { }         // Error: non-bool condition

// ✅ Explicit comparison required
if x != 0 { }
if s != "" { }
if ptr != nil { }
```

**Why this matters:** Makes code explicit and prevents subtle bugs from implicit conversions.

#### **Type Conversions vs Type Assertions**

Go requires **explicit type conversion** between different types, even between related numeric types:

```go
// ❌ Implicit conversion not allowed
var i int = 42
var f float64 = i  // Compiler error!

// ✅ Explicit conversion required
var i int = 42
var f float64 = float64(i)  // OK

// Between numeric types
var x int32 = 100
var y int64 = int64(x)
var z float64 = float64(x)

// String conversions
i := 65
s := string(i)        // "A" (rune to string)
s = strconv.Itoa(i)   // "65" (int to string representation)

s = "123"
i, err := strconv.Atoi(s)       // string to int
f, err := strconv.ParseFloat(s, 64)  // string to float64
```

**Type assertions (for interfaces):**

```go
var i interface{} = "hello"

// Type assertion
s := i.(string)  // s = "hello"

// Unsafe type assertion (panics if wrong type)
s := i.(string)

// Safe type assertion (returns ok boolean)
s, ok := i.(string)
if ok {
    fmt.Println(s)
} else {
    fmt.Println("not a string")
}

// Type switch
switch v := i.(type) {
case string:
    fmt.Println("string:", v)
case int:
    fmt.Println("int:", v)
default:
    fmt.Println("unknown type")
}
```

We'll cover interfaces and type assertions in depth in section 1.5.

### Composite Types

#### **Arrays (Fixed Size, Value Semantics)**

Arrays in Go have **fixed size** and are **value types** (copied when assigned):

```go
// Declaration
var arr [5]int  // Array of 5 ints, all initialized to 0

// With initialization
arr := [5]int{1, 2, 3, 4, 5}

// Let compiler count
arr := [...]int{1, 2, 3, 4, 5}  // Same as [5]int

// Partial initialization
arr := [5]int{1, 2}  // [1, 2, 0, 0, 0]

// Specific indices
arr := [5]int{1: 10, 3: 30}  // [0, 10, 0, 30, 0]

// Access elements
fmt.Println(arr[0])  // First element
arr[0] = 100         // Modify element

// Length
fmt.Println(len(arr))  // 5
```

**Arrays are value types:**

```go
arr1 := [3]int{1, 2, 3}
arr2 := arr1  // Copies all elements!

arr2[0] = 100
fmt.Println(arr1[0])  // 1 (unchanged)
fmt.Println(arr2[0])  // 100

// Passing to function also copies
func modifyArray(arr [3]int) {
    arr[0] = 999  // Modifies copy
}

arr := [3]int{1, 2, 3}
modifyArray(arr)
fmt.Println(arr[0])  // 1 (unchanged!)
```

**When to use arrays:**
- When size is known and fixed
- When you want value semantics
- For small fixed-size collections (like RGB values, coordinates)
- In practice, **slices are used much more often**

```go
// ✅ Good use of array: fixed-size data
type Point3D struct {
    coords [3]float64  // x, y, z
}

type RGB struct {
    values [3]uint8  // red, green, blue
}
```

#### **Slices (Dynamic Arrays, Reference Semantics)**

Slices are Go's dynamic arrays - they're what you'll use 99% of the time instead of arrays.

**Slice Internals:**

A slice is a descriptor containing three fields:
```
┌──────────┐
│ pointer  │──→ [underlying array]
├──────────┤
│ length   │
├──────────┤
│ capacity │
└──────────┘
```

```go
// Create slice
var s []int  // nil slice (pointer, length, capacity all zero)

s = []int{1, 2, 3, 4, 5}  // Slice literal

// make() for pre-allocation
s = make([]int, 5)       // length 5, capacity 5, [0,0,0,0,0]
s = make([]int, 5, 10)   // length 5, capacity 10

// Length and capacity
fmt.Println(len(s))  // Number of elements
fmt.Println(cap(s))  // Capacity of underlying array
```

**Slice operations:**

```go
s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

// Slicing (creates new slice descriptor, shares array)
s1 := s[2:5]   // [2, 3, 4] - elements from index 2 to 4
s2 := s[:5]    // [0, 1, 2, 3, 4] - from start to index 4
s3 := s[5:]    // [5, 6, 7, 8, 9] - from index 5 to end
s4 := s[:]     // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] - full slice

// Append
s = append(s, 10)         // Append single element
s = append(s, 11, 12, 13) // Append multiple
s = append(s, []int{14, 15, 16}...)  // Append another slice

// Copy
s1 := []int{1, 2, 3}
s2 := make([]int, len(s1))
copy(s2, s1)  // Copies elements from s1 to s2
```

**Critical Understanding: Append and Reallocation**

```go
s := make([]int, 0, 3)  // len=0, cap=3
fmt.Printf("addr: %p, len: %d, cap: %d\n", s, len(s), cap(s))

s = append(s, 1)  // len=1, cap=3
fmt.Printf("addr: %p, len: %d, cap: %d\n", s, len(s), cap(s))

s = append(s, 2)  // len=2, cap=3
fmt.Printf("addr: %p, len: %d, cap: %d\n", s, len(s), cap(s))

s = append(s, 3)  // len=3, cap=3
fmt.Printf("addr: %p, len: %d, cap: %d\n", s, len(s), cap(s))

s = append(s, 4)  // len=4, cap=6 (reallocated! address changes)
fmt.Printf("addr: %p, len: %d, cap: %d\n", s, len(s), cap(s))

// When capacity is exceeded:
// 1. New array is allocated (usually 2x capacity)
// 2. Elements are copied to new array
// 3. Slice descriptor points to new array
```

**Common Slice Pitfalls:**

**Pitfall 1: Shared underlying array**

```go
s1 := []int{1, 2, 3, 4, 5}
s2 := s1[1:3]  // [2, 3]

s2[0] = 999
fmt.Println(s1)  // [1, 999, 3, 4, 5] - s1 modified!
fmt.Println(s2)  // [999, 3]

// Why? s1 and s2 share the same underlying array
```

**Pitfall 2: Append can modify other slices**

```go
s1 := []int{1, 2, 3, 4, 5}
s2 := s1[:3]  // [1, 2, 3], but capacity is 5

s2 = append(s2, 999)
fmt.Println(s1)  // [1, 2, 3, 999, 5] - s1 modified!
fmt.Println(s2)  // [1, 2, 3, 999]

// Why? s2 had capacity, so append overwrote existing array
```

**Pitfall 3: Loop variable in goroutine**

```go
// ❌ WRONG: All goroutines print the same value
values := []int{1, 2, 3, 4, 5}
for _, v := range values {
    go func() {
        fmt.Println(v)  // All print 5!
    }()
}

// ✅ CORRECT: Pass value to goroutine
for _, v := range values {
    go func(val int) {
        fmt.Println(val)  // Prints different values
    }(v)
}

// ✅ ALSO CORRECT: Copy to new variable (Go 1.22+ fixes this)
for _, v := range values {
    v := v  // Create new variable
    go func() {
        fmt.Println(v)
    }()
}
```

**Pre-allocation strategies:**

```go
// ❌ BAD: Repeated reallocations
var s []int
for i := 0; i < 1000; i++ {
    s = append(s, i)  // Reallocates multiple times
}

// ✅ GOOD: Pre-allocate if size is known
s := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    s = append(s, i)  // No reallocations
}

// ✅ ALSO GOOD: Pre-allocate and index
s := make([]int, 1000)
for i := 0; i < 1000; i++ {
    s[i] = i  // Direct assignment, even faster
}
```

**nil slice vs empty slice:**

```go
var s1 []int        // nil slice
s2 := []int{}       // empty slice (not nil)
s3 := make([]int, 0) // empty slice (not nil)

fmt.Println(s1 == nil)  // true
fmt.Println(s2 == nil)  // false
fmt.Println(s3 == nil)  // false

// However, len() works the same
fmt.Println(len(s1))  // 0
fmt.Println(len(s2))  // 0
fmt.Println(len(s3))  // 0

// ✅ BEST PRACTICE: Check length, not nil
if len(s) > 0 {
    // has elements
}

// Not: if s != nil { ... }
```

#### **Maps (Hash Tables)**

Maps are Go's built-in hash table implementation providing O(1) average-case lookup, insert, and delete.

**Map Internals:**

A Go map is implemented as a hash table with the following characteristics:
- Uses separate chaining for collision resolution
- Dynamically resizes when load factor exceeds threshold
- Not thread-safe (requires external synchronization)
- Iteration order is **intentionally randomized**

**Creating maps:**

```go
// Declaration (nil map - can't be used yet)
var m map[string]int

// Initialize with make
m = make(map[string]int)

// Map literal
m = map[string]int{
    "alice": 25,
    "bob":   30,
    "carol": 35,
}

// make with size hint (optimization)
m = make(map[string]int, 100)  // Pre-allocate for ~100 elements
```

**Map operations:**

```go
m := make(map[string]int)

// Insert or update
m["alice"] = 25
m["bob"] = 30

// Retrieve
age := m["alice"]  // 25

// Check existence (important!)
age, ok := m["alice"]
if ok {
    fmt.Println("Alice's age:", age)
} else {
    fmt.Println("Alice not found")
}

// Delete
delete(m, "alice")

// Length
fmt.Println(len(m))

// Iterate (order is random!)
for key, value := range m {
    fmt.Printf("%s: %d\n", key, value)
}

// Iterate keys only
for key := range m {
    fmt.Println(key)
}
```

**Critical: Map is not thread-safe:**

```go
// ❌ DANGER: Concurrent map writes cause panic
m := make(map[int]int)

go func() {
    for i := 0; i < 1000; i++ {
        m[i] = i  // Writing
    }
}()

go func() {
    for i := 0; i < 1000; i++ {
        m[i] = i  // Concurrent write - PANIC!
    }
}()

// ✅ SOLUTION 1: Use sync.Mutex
var mu sync.Mutex
mu.Lock()
m[key] = value
mu.Unlock()

// ✅ SOLUTION 2: Use sync.Map (for specific use cases)
var sm sync.Map
sm.Store(key, value)
value, ok := sm.Load(key)

// ✅ SOLUTION 3: Use channels for synchronization
```

**Map key requirements:**

Only comparable types can be used as map keys:

```go
// ✅ Valid key types
map[string]int          // strings
map[int]string          // integers
map[float64]bool        // floats (careful with precision!)
map[struct{x,y int}]string  // structs (if all fields comparable)
map[[3]int]string       // arrays (if element type comparable)

// ❌ Invalid key types
map[[]int]string        // slices are not comparable
map[map[string]int]bool // maps are not comparable
map[func()]int          // functions are not comparable
```

**Common map patterns:**

```go
// Check and initialize pattern
if _, ok := m[key]; !ok {
    m[key] = defaultValue
}

// Or more concisely (relies on zero value)
m[key]++  // If key doesn't exist, zero value (0) + 1 = 1

// Set-like behavior (map[T]bool)
set := make(map[string]bool)
set["apple"] = true
set["banana"] = true

if set["apple"] {
    fmt.Println("apple is in set")
}

// Or use empty struct to save memory
set := make(map[string]struct{})
set["apple"] = struct{}{}
set["banana"] = struct{}{}

if _, ok := set["apple"]; ok {
    fmt.Println("apple is in set")
}
```

**Map pitfalls:**

```go
// Pitfall 1: Nil map
var m map[string]int  // nil map
// m["key"] = 1  // ❌ PANIC: assignment to entry in nil map

// ✅ Must initialize
m = make(map[string]int)
m["key"] = 1  // OK

// Pitfall 2: Getting from nil map is OK (returns zero value)
var m map[string]int
value := m["key"]  // 0 (no panic, but probably not what you want)

// Pitfall 3: Random iteration order
m := map[string]int{"a": 1, "b": 2, "c": 3}
for k, v := range m {
    fmt.Println(k, v)  // Order is random and changes between runs!
}

// If you need order, use a slice of keys
keys := make([]string, 0, len(m))
for k := range m {
    keys = append(keys, k)
}
sort.Strings(keys)
for _, k := range keys {
    fmt.Println(k, m[k])  // Now in sorted order
}
```

#### **Structs**

Structs are Go's way of creating composite data types:

```go
// Define a struct type
type Person struct {
    Name    string
    Age     int
    Email   string
    Address Address  // Nested struct
}

type Address struct {
    Street  string
    City    string
    ZipCode string
}

// Create struct instances
var p1 Person  // Zero value: all fields are zero values

p2 := Person{
    Name:  "Alice",
    Age:   30,
    Email: "alice@example.com",
}

p3 := Person{"Bob", 25, "bob@example.com", Address{}}  // Positional (discouraged)

// Access fields
fmt.Println(p2.Name)
p2.Age = 31

// Pointer to struct
ptr := &Person{Name: "Carol"}
ptr.Age = 35  // Auto-dereferencing (no need for (*ptr).Age)

// Anonymous structs
config := struct {
    Host string
    Port int
}{
    Host: "localhost",
    Port: 8080,
}
```

**Struct embedding (composition):**

Go doesn't have inheritance, but it has embedding which is more powerful:

```go
type Animal struct {
    Name string
    Age  int
}

func (a Animal) Speak() string {
    return "Some sound"
}

type Dog struct {
    Animal  // Embedded struct - promotes Animal's fields and methods
    Breed   string
}

func (d Dog) Speak() string {
    return "Woof!"  // Override embedded method
}

// Usage
dog := Dog{
    Animal: Animal{Name: "Buddy", Age: 3},
    Breed:  "Golden Retriever",
}

// Can access embedded fields directly
fmt.Println(dog.Name)  // "Buddy" (promoted from Animal)
fmt.Println(dog.Age)   // 3 (promoted from Animal)

// Can also access via embedded type
fmt.Println(dog.Animal.Name)  // "Buddy"

// Method calls
fmt.Println(dog.Speak())        // "Woof!" (Dog's method)
fmt.Println(dog.Animal.Speak()) // "Some sound" (Animal's method)
```

**Struct tags and reflection:**

Struct tags are metadata for fields, commonly used with JSON, XML, database ORMs:

```go
type User struct {
    ID        int    `json:"id" db:"user_id"`
    FirstName string `json:"first_name" db:"first_name"`
    LastName  string `json:"last_name" db:"last_name"`
    Email     string `json:"email,omitempty" validate:"required,email"`
    Password  string `json:"-" db:"password_hash"`  // "-" means ignore
}

// JSON marshaling uses these tags
user := User{
    ID:        1,
    FirstName: "John",
    LastName:  "Doe",
    Email:     "john@example.com",
}

jsonData, _ := json.Marshal(user)
fmt.Println(string(jsonData))
// {"id":1,"first_name":"John","last_name":"Doe","email":"john@example.com"}
```

**Memory layout and alignment:**

Go structs have memory layout considerations:

```go
// Poor layout (16 bytes + padding)
type BadLayout struct {
    a bool   // 1 byte
    // 7 bytes padding
    b int64  // 8 bytes
}

// Good layout (12 bytes, less padding)
type GoodLayout struct {
    b int64  // 8 bytes
    a bool   // 1 byte
    // 3 bytes padding
}

// Check struct size
fmt.Println(unsafe.Sizeof(BadLayout{}))   // 16
fmt.Println(unsafe.Sizeof(GoodLayout{}))  // 12

// General rule: Order fields from largest to smallest
type OptimalLayout struct {
    // 8-byte fields first
    ptr    *int
    slice  []int
    int64  int64
    // 4-byte fields
    int32  int32
    float32 float32
    // 2-byte fields
    int16  int16
    // 1-byte fields
    bool1  bool
    byte1  byte
}
```

**Struct comparison:**

Structs are comparable if all their fields are comparable:

```go
type Point struct {
    X, Y int
}

p1 := Point{1, 2}
p2 := Point{1, 2}
p3 := Point{3, 4}

fmt.Println(p1 == p2)  // true
fmt.Println(p1 == p3)  // false

// Structs with slices/maps are NOT comparable
type Container struct {
    Items []int  // Slice - not comparable
}

// c1 := Container{[]int{1,2,3}}
// c2 := Container{[]int{1,2,3}}
// fmt.Println(c1 == c2)  // ❌ Compile error!

// Use reflect.DeepEqual for deep comparison
fmt.Println(reflect.DeepEqual(c1, c2))  // Works but slow
```

#### **Pointers**

Pointers hold memory addresses. Go has pointers but no pointer arithmetic (for safety).

```go
// Declare pointer
var p *int  // nil pointer

// Get address of variable
x := 42
p = &x  // p points to x

// Dereference pointer
fmt.Println(*p)  // 42 (value at address)

// Modify through pointer
*p = 100
fmt.Println(x)  // 100 (x was modified)

// Nil pointer
var p *int
// *p = 10  // ❌ PANIC: nil pointer dereference

// Check for nil
if p != nil {
    *p = 10
}
```

**Pointer vs value semantics:**

```go
// Value semantics (copy)
func modifyValue(x int) {
    x = 100  // Modifies copy
}

x := 42
modifyValue(x)
fmt.Println(x)  // 42 (unchanged)

// Pointer semantics (reference)
func modifyPointer(p *int) {
    *p = 100  // Modifies original
}

x = 42
modifyPointer(&x)
fmt.Println(x)  // 100 (changed!)
```

**When to use pointers:**

```go
// ✅ Use pointers for:
// 1. Large structs (avoid copying)
type LargeStruct struct {
    data [1000]int
}

func ProcessLarge(ls *LargeStruct) {  // Pass pointer
    // Work with large struct without copying
}

// 2. When you need to modify the original
func (p *Person) SetAge(age int) {  // Pointer receiver
    p.Age = age  // Modifies original
}

// 3. When nil is a valid value
func FindUser(id int) *User {
    // ...
    if notFound {
        return nil  // Indicate "no user"
    }
    return &user
}

// ❌ Don't use pointers for:
// 1. Small structs (copying is faster)
type Point struct {
    X, Y int
}

func Distance(p Point) float64 {  // Value is fine
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}

// 2. Slices, maps, channels (already reference types)
func ProcessSlice(s []int) {  // Don't do *[]int!
    // s already references the underlying array
}
```

**new() vs make():**

```go
// new() allocates memory, returns pointer to zero value
p := new(int)  // *int pointing to 0
fmt.Println(*p)  // 0

s := new([]int)  // *[]int pointing to nil slice
// Can't use s directly, need to initialize:
*s = make([]int, 10)

// make() initializes slices, maps, channels (returns value, not pointer)
slice := make([]int, 10)     // []int ready to use
m := make(map[string]int)    // map[string]int ready to use
ch := make(chan int)         // chan int ready to use

// In practice:
// Use new() rarely (mostly just do &Type{})
// Use make() for slices, maps, channels
```

**Pointer arithmetic is not allowed:**

```go
p := &x
// p++  // ❌ Compile error: invalid operation

// Use unsafe package if you really need it (rarely)
import "unsafe"

p := &x
nextP := (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(p)) + unsafe.Sizeof(x)))
// Don't do this unless you really know what you're doing!
```

### Type Conversions and Type Aliases

**Type conversions:**

```go
// Between numeric types
var i int = 42
var i64 int64 = int64(i)
var f float64 = float64(i)

// String conversions
i := 65
s := string(i)  // "A" (converts to rune/character)
s = strconv.Itoa(i)  // "65" (converts to string representation)

f := 3.14
s = fmt.Sprintf("%f", f)  // "3.140000"
s = strconv.FormatFloat(f, 'f', 2, 64)  // "3.14"

// From string
s = "123"
i, err := strconv.Atoi(s)
f, err := strconv.ParseFloat(s, 64)
b, err := strconv.ParseBool("true")

// Byte slice to string and vice versa
bytes := []byte("hello")
str := string(bytes)
bytes = []byte(str)
```

**Custom types:**

```go
// Define custom type
type Age int
type Name string

var age Age = 25
var name Name = "Alice"

// Cannot mix with base types without conversion
// var x int = age  // ❌ Error: cannot use age (type Age) as int

// ✅ Explicit conversion needed
var x int = int(age)

// Custom types can have methods
func (a Age) IsAdult() bool {
    return a >= 18
}

age := Age(25)
fmt.Println(age.IsAdult())  // true
```

**Type aliases (Go 1.9+):**

```go
// Type alias - truly identical to original type
type MyInt = int  // Note the "="

var x MyInt = 10
var y int = x  // ✅ No conversion needed!

// Useful for:
// 1. Gradual code refactoring
// 2. Providing alternate names

// Example from standard library:
type byte = uint8  // byte is an alias for uint8
type rune = int32  // rune is an alias for int32
```

### Frequently Asked Questions

**Q1: What's the difference between nil and zero value?**

**A:** Zero value is the default value a variable gets when declared without initialization. `nil` is the zero value for pointers, slices, maps, channels, functions, and interfaces:

```go
var i int        // Zero value: 0
var s string     // Zero value: ""
var b bool       // Zero value: false

var ptr *int     // Zero value: nil
var slice []int  // Zero value: nil
var m map[string]int  // Zero value: nil
```

**Why This Matters:** Understanding zero values prevents bugs. For example, a nil slice can be safely used with `len()` and `append()`, but a nil map cannot be written to.

**Related Concepts:** Memory initialization, safe defaults, nil checking

---

**Q2: When should I use arrays vs slices?**

**A:** Use slices 99% of the time. Use arrays only when:
1. Size is truly fixed and known at compile time
2. You need value semantics (copy on assignment)
3. You're working with fixed-size data structures (RGB, coordinates, etc.)

```go
// ✅ Slice (most common)
func ProcessItems(items []int) {
    // Dynamic, flexible
}

// ✅ Array (rare, specific use case)
type RGB struct {
    colors [3]uint8  // Always exactly 3 colors
}
```

**Why This Matters:** Slices are more flexible and idiomatic in Go. Arrays are rarely needed in practice.

**Related Concepts:** Slice internals (pointer, length, capacity), pass by value vs reference

---

**Q3: Why can't I compare slices with ==?**

**A:** Slices are not comparable because they're reference types and comparison semantics would be ambiguous:

```go
s1 := []int{1, 2, 3}
s2 := []int{1, 2, 3}
// fmt.Println(s1 == s2)  // ❌ Compile error

// ✅ Use reflect.DeepEqual for deep comparison
fmt.Println(reflect.DeepEqual(s1, s2))  // true

// ✅ Or compare elements manually for performance
func slicesEqual(a, b []int) bool {
    if len(a) != len(b) {
        return false
    }
    for i := range a {
        if a[i] != b[i] {
            return false
        }
    }
    return true
}
```

**Why This Matters:** This design prevents confusion about whether == should compare slice descriptors or elements. Go forces you to be explicit.

**Related Concepts:** Comparable types, slice internals, reference semantics

---

**Q4: What's the difference between make() and new()?**

**A:** 
- `new(T)` allocates memory for type T, initializes to zero value, returns `*T`
- `make(T, args)` initializes slices/maps/channels (only), returns `T` (not pointer)

```go
// new() - any type, returns pointer
p := new(int)     // *int, points to 0
s := new([]int)   // *[]int, points to nil slice

// make() - slices, maps, channels only, returns value
slice := make([]int, 10)      // []int with 10 zeros
m := make(map[string]int)     // empty map, ready to use
ch := make(chan int, 5)       // buffered channel

// In practice:
// Use make() for slices, maps, channels
// Use &Type{} instead of new() for structs
p := &Person{}  // Better than new(Person)
```

**Why This Matters:** Using the right allocation function makes code clearer and avoids initialization bugs.

**Related Concepts:** Memory allocation, initialization, reference types

---

**Q5: How do I choose between value and pointer receivers for methods?**

**A:** Use pointer receivers when:
1. Method modifies the receiver
2. Receiver is large (avoid copying)
3. You need consistency (if some methods use pointers, all should)

```go
type Counter struct {
    count int
}

// ✅ Pointer receiver - modifies state
func (c *Counter) Increment() {
    c.count++
}

// ✅ Value receiver - doesn't modify, small struct
func (c Counter) Value() int {
    return c.count
}

// For consistency: if any method uses pointer receiver,
// all methods should use pointer receiver
```

**Why This Matters:** Incorrect receiver type can lead to subtle bugs where modifications don't persist, or unnecessary copying hurts performance.

**Related Concepts:** Method sets, interface satisfaction, value vs reference semantics

---

### Interview Questions

**Question 1: Explain what happens when you append to a slice that has reached its capacity.**

**Difficulty:** Mid-Level

**Answer:**

When appending to a slice at full capacity, Go performs these steps:

1. **Allocates a new array** with larger capacity (typically 2x for small slices, <1024 elements)
2. **Copies existing elements** from old array to new array
3. **Appends the new element** to the new array
4. **Returns a new slice descriptor** pointing to the new array

```go
s := make([]int, 3, 3)  // len=3, cap=3, [0,0,0]
fmt.Printf("Before: ptr=%p, len=%d, cap=%d\n", s, len(s), cap(s))

s = append(s, 4)  // Triggers reallocation
fmt.Printf("After:  ptr=%p, len=%d, cap=%d\n", s, len(s), cap(s))

// Output shows different pointer address and increased capacity
```

**What Interviewers Look For:**
- Understanding of slice internals (pointer, length, capacity)
- Knowledge of reallocation trigger (capacity exceeded)
- Awareness that append returns a new slice
- Performance implications of repeated reallocations

**Follow-up Questions:**
- How would you optimize appending in a loop? (Pre-allocate with make)
- What's the time complexity of append? (Amortized O(1))
- Can appending to one slice affect another slice? (Yes, if they share an array)

**Related Topics:**
- Slice internals, memory allocation, append performance optimization

---

**Question 2: What is the zero value for a map, and why is it different from an empty map?**

**Difficulty:** Junior

**Answer:**

The zero value for a map is `nil`. A nil map is different from an empty map:

```go
// Nil map (zero value)
var m1 map[string]int
fmt.Println(m1 == nil)  // true
fmt.Println(len(m1))    // 0
value := m1["key"]      // OK: returns 0 (zero value of int)
// m1["key"] = 1        // ❌ PANIC: assignment to entry in nil map

// Empty map (initialized but empty)
m2 := make(map[string]int)
fmt.Println(m2 == nil)  // false
fmt.Println(len(m2))    // 0
m2["key"] = 1           // ✅ OK: can write to empty map
```

Key differences:
1. **Nil map**: Cannot write to it (will panic)
2. **Empty map**: Can write to it (initialized)
3. **Both**: Can read from them (returns zero value for missing keys), both have length 0

**What Interviewers Look For:**
- Understanding that nil and empty are different
- Knowledge that you must initialize maps before writing
- Awareness that reading from nil map is safe (returns zero value)

**Follow-up Questions:**
- How do you check if a map is initialized? (Check `m != nil` or try to write)
- What's the difference between `var m map[string]int` and `m := make(map[string]int)`?
- Can you add to a nil slice? (Yes! append handles nil slices)

**Related Topics:**
- Zero values, map initialization, nil checking

---

**Question 3: Explain the difference between these two struct declarations and when you'd use each:**

```go
type Server struct {
    Host string
    Port int
}

type SecureServer struct {
    Server
    CertFile string
}
```

**Difficulty:** Mid-Level

**Answer:**

This demonstrates **struct embedding** (composition):

`SecureServer` **embeds** `Server`, which means:
1. SecureServer has all of Server's fields (Host, Port) **promoted** to the outer struct
2. SecureServer can access Server's methods
3. SecureServer can override Server's methods
4. This is Go's way of achieving composition without inheritance

```go
ss := SecureServer{
    Server: Server{
        Host: "localhost",
        Port: 443,
    },
    CertFile: "/path/to/cert",
}

// Can access Server's fields directly (promoted)
fmt.Println(ss.Host)  // "localhost"
fmt.Println(ss.Port)  // 443

// Or through the embedded field
fmt.Println(ss.Server.Host)  // "localhost"

// If Server had methods, SecureServer would inherit them
```

**When to use embedding:**
- When you want to extend or specialize a type
- When you want to reuse fields and methods from another struct
- When modeling "has-a" relationships (composition)
- As an alternative to inheritance in OOP

**What Interviewers Look For:**
- Understanding of composition over inheritance in Go
- Knowledge of field promotion
- Awareness that Go doesn't have classes or inheritance
- Ability to explain when embedding is appropriate

**Follow-up Questions:**
- What happens if SecureServer has a field with the same name as Server? (SecureServer's field shadows Server's)
- Can you embed interfaces? (Yes, and it's a common pattern)
- How is this different from inheritance in Java/C++? (No polymorphism in the same way; promotes composition)

**Related Topics:**
- Composition, interfaces, method sets, embedding

---

### Key Takeaways

✅ **Go's type system is simple but strict:**
- No implicit conversions (even between int and int64)
- Every type has a zero value (no uninitialized variables)
- Explicit conversions make intent clear

✅ **Value vs reference semantics:**
- Arrays, structs, basic types: value semantics (copied)
- Slices, maps, channels, pointers: reference semantics (shared)
- Understanding this prevents subtle bugs

✅ **Slices are central to Go:**
- Dynamic, flexible, most common collection type
- Understand internals: pointer, length, capacity
- Pre-allocate when size is known for performance
- Watch out for shared underlying arrays

✅ **Maps require initialization:**
- Nil map: cannot write, can read (returns zero value)
- Empty map: can read and write
- Not thread-safe (need sync.Mutex or sync.Map)

✅ **Structs use composition, not inheritance:**
- Embed structs to reuse fields and methods
- Field and method promotion makes embedding powerful
- More flexible than class-based inheritance

✅ **Pointers provide reference semantics:**
- Use for large structs, when you need to modify, or when nil is valid
- Don't use for slices/maps/channels (already references)
- No pointer arithmetic (safety)

Next in Part 1, we'll dive deep into control flow, functions, interfaces, and package management - the building blocks of any Go program.

## 1.3 Control Flow

Go's control flow statements are deliberately simple and consistent. Unlike many languages, Go has fewer constructs but uses them in powerful ways.

### Conditionals

#### **if/else Statements**

The basic `if` statement in Go:

```go
if condition {
    // code
}

if condition {
    // code
} else {
    // code
}

if condition1 {
    // code
} else if condition2 {
    // code
} else {
    // code
}
```

**Key differences from other languages:**
- No parentheses required around condition (in fact, they're discouraged)
- Braces `{}` are required (even for single statements)
- Condition must be a boolean expression (no truthiness)

```go
// ❌ These don't compile
if (x > 5) { }  // Unnecessary parentheses (works but not idiomatic)
if x { }        // Error: non-bool used as condition
if 1 { }        // Error: non-bool used as condition

// ✅ Idiomatic Go
if x > 5 { }
if x != 0 { }
if ptr != nil { }
```

#### **if with Initialization Statement**

One of Go's most useful patterns - declare and check in one line:

```go
// Traditional way
value, err := someFunction()
if err != nil {
    return err
}
// use value

// ✅ BETTER: if with initialization
if value, err := someFunction(); err != nil {
    return err
}
// use value (still in scope!)

// More examples
if age := calculateAge(birthdate); age >= 18 {
    fmt.Println("Adult")
} else {
    fmt.Println("Minor")
}

// Useful for limiting scope
if val, ok := myMap[key]; ok {
    // val only exists in this scope
    processValue(val)
}
// val is not accessible here
```

**Why this matters:** This pattern limits variable scope, making code cleaner and reducing the chance of variable misuse.

#### **switch Statements**

Go's `switch` is more powerful than in most languages:

**Expression switch:**

```go
day := "Monday"

switch day {
case "Monday":
    fmt.Println("Start of week")
case "Friday":
    fmt.Println("End of week")
case "Saturday", "Sunday":  // Multiple values
    fmt.Println("Weekend!")
default:
    fmt.Println("Midweek")
}
```

**No automatic fallthrough:**

Unlike C/Java, Go's switch does NOT fall through by default:

```go
x := 1

switch x {
case 1:
    fmt.Println("one")
    // Automatically breaks here!
case 2:
    fmt.Println("two")
}
// Output: "one" (and stops)

// If you want fallthrough (rare):
switch x {
case 1:
    fmt.Println("one")
    fallthrough  // Explicitly continue to next case
case 2:
    fmt.Println("two")
}
// Output: "one" then "two"
```

**switch with initialization:**

```go
switch x := computeValue(); x {
case 1:
    fmt.Println("one")
case 2:
    fmt.Println("two")
default:
    fmt.Println("other")
}
// x only exists within switch scope
```

**Tagless switch (replaces if-else chains):**

```go
age := 25
switch {
case age < 18:
    fmt.Println("Minor")
case age < 65:
    fmt.Println("Adult")
default:
    fmt.Println("Senior")
}

// This is idiomatic for replacing long if-else chains
// More readable than:
// if age < 18 { ... } else if age < 65 { ... } else { ... }
```

**Type switch:**

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("int: %d\n", v)
    case string:
        fmt.Printf("string: %s\n", v)
    case bool:
        fmt.Printf("bool: %t\n", v)
    case Person:
        fmt.Printf("Person: %+v\n", v)
    case *Person:  // Can match pointer types too
        fmt.Printf("*Person: %+v\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

describe(42)        // int: 42
describe("hello")   // string: hello
describe(true)      // bool: true
```

**Why no ternary operator?**

Go intentionally omits the ternary operator (`condition ? true : false`):

```go
// ❌ NOT IN GO: max = (a > b) ? a : b

// ✅ Use if-else
var max int
if a > b {
    max = a
} else {
    max = b
}

// Or extract to a function
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

result := max(a, b)
```

The Go team believes explicit if-else is clearer than ternary operators, which can become hard to read when nested.

### Loops

#### **for Loop (The Only Loop in Go)**

Go has only ONE loop keyword: `for`. But it can express all loop patterns:

**Traditional C-style for:**

```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// init, condition, post are all optional
for ; i < 10; {  // while-style
    i++
}

for i < 10 {  // More idiomatic while-style
    i++
}

for {  // Infinite loop
    // do something forever
    if condition {
        break
    }
}
```

**Range loops:**

Range iterates over slices, arrays, maps, strings, and channels:

```go
// Slice/Array: index and value
slice := []int{10, 20, 30}
for i, v := range slice {
    fmt.Printf("Index: %d, Value: %d\n", i, v)
}

// Index only
for i := range slice {
    fmt.Println(i)
}

// Value only (use _ to ignore index)
for _, v := range slice {
    fmt.Println(v)
}

// Map: key and value
m := map[string]int{"a": 1, "b": 2}
for k, v := range m {
    fmt.Printf("%s: %d\n", k, v)
}

// String: index and rune
s := "Hello, 世界"
for i, r := range s {
    fmt.Printf("%d: %c\n", i, r)
}

// Channel: value only
ch := make(chan int)
for v := range ch {  // Receives until channel is closed
    fmt.Println(v)
}
```

**Critical range gotcha:**

```go
// ❌ WRONG: All goroutines print the same value
values := []int{1, 2, 3, 4, 5}
for _, v := range values {
    go func() {
        fmt.Println(v)  // All print 5!
    }()
}

// ✅ CORRECT: Pass value to goroutine
for _, v := range values {
    go func(val int) {
        fmt.Println(val)  // Prints different values
    }(v)
}

// ✅ ALSO CORRECT: Copy to new variable (Go 1.22+ fixes this)
for _, v := range values {
    v := v  // Creates new v in each iteration
    go func() {
        fmt.Println(v)
    }()
}
```

**Why this happens:** The loop variable `v` is reused in each iteration. When the goroutine runs (later), it captures the variable, not the value, so all goroutines see the final value.

**Range makes a copy:**

```go
arr := [3]int{1, 2, 3}
for i, v := range arr {
    arr[i] = v + 10  // Modifying original array
    fmt.Println(v)   // But v is from a copy
}
// Prints: 1, 2, 3 (not 11, 12, 13)

// To modify during iteration, use indices:
for i := range arr {
    arr[i] = arr[i] + 10
}
```

#### **Break and Continue**

```go
// break exits the loop
for i := 0; i < 10; i++ {
    if i == 5 {
        break  // Exit loop when i is 5
    }
    fmt.Println(i)
}

// continue skips to next iteration
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue  // Skip even numbers
    }
    fmt.Println(i)  // Only prints odd numbers
}
```

#### **Labels**

Labels allow breaking out of nested loops:

```go
OuterLoop:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if i == 1 && j == 1 {
            break OuterLoop  // Breaks outer loop
        }
        fmt.Printf("(%d,%d) ", i, j)
    }
}

// Without label, would only break inner loop
```

### Defer, Panic, Recover

#### **defer Statement**

`defer` schedules a function call to run just before the surrounding function returns:

```go
func example() {
    defer fmt.Println("world")
    fmt.Println("hello")
}
// Output: hello
//         world
```

**LIFO execution (Last In, First Out):**

```go
func example() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
}
// Output: 3
//         2
//         1
```

**Common use cases:**

```go
// 1. Closing files
func readFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()  // Ensures file is closed when function returns
    
    // Read file...
    // Even if we return early or panic, f.Close() will be called
    return nil
}

// 2. Unlocking mutexes
func deposit(amount int) {
    mu.Lock()
    defer mu.Unlock()  // Ensures unlock even if panic occurs
    
    balance += amount
}

// 3. Logging execution time
func slowFunction() {
    start := time.Now()
    defer func() {
        fmt.Printf("Took: %v\n", time.Since(start))
    }()
    
    // Do slow work...
    time.Sleep(2 * time.Second)
}

// 4. Recovering from panics
func safeFunction() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    
    // Code that might panic
}
```

**defer pitfall - loop variables:**

```go
// ❌ WRONG: All defers see final value of i
for i := 0; i < 5; i++ {
    defer fmt.Println(i)  // All print 4!
}

// ✅ CORRECT: Pass i as argument
for i := 0; i < 5; i++ {
    defer func(val int) {
        fmt.Println(val)
    }(i)
}

// ✅ ALSO CORRECT: Use return value
for i := 0; i < 5; i++ {
    i := i  // Create new i for each iteration
    defer fmt.Println(i)
}
```

**defer evaluates arguments immediately:**

```go
func example() {
    x := 10
    defer fmt.Println(x)  // Captures value 10 immediately
    
    x = 20
    fmt.Println(x)
}
// Output: 20
//         10 (not 20!)
```

#### **panic and recover**

**panic** stops normal execution and begins panicking:

```go
func doSomething() {
    panic("something went wrong!")
}

func main() {
    doSomething()
    fmt.Println("This never prints")
}
// Program crashes with panic message
```

**When to use panic:**

```go
// ✅ Use panic for programming errors (bugs)
if index < 0 || index >= len(slice) {
    panic("index out of bounds")  // Shouldn't happen if code is correct
}

// ✅ Use panic in init() or package initialization
func init() {
    if !databaseAvailable() {
        panic("database required but not available")
    }
}

// ❌ DON'T use panic for normal errors
func OpenFile(name string) {
    f, err := os.Open(name)
    if err != nil {
        panic(err)  // ❌ BAD: file not found is a normal error
    }
    // ...
}

// ✅ Return error instead
func OpenFile(name string) error {
    f, err := os.Open(name)
    if err != nil {
        return err  // ✅ GOOD: let caller handle
    }
    // ...
}
```

**recover** catches panics:

```go
func safeCall(fn func()) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
            debug.PrintStack()  // Print stack trace
        }
    }()
    
    fn()  // Might panic
}

// Usage
safeCall(func() {
    panic("oops!")
})
// Program continues running
```

**Panic/recover is NOT exception handling:**

```go
// ❌ ANTI-PATTERN: Using panic/recover like try/catch
func divide(a, b int) (result int) {
    defer func() {
        if r := recover(); r != nil {
            result = 0  // Don't do this!
        }
    }()
    
    if b == 0 {
        panic("division by zero")  // Don't do this!
    }
    return a / b
}

// ✅ IDIOMATIC: Return error
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

**Why panic/recover ≠ exceptions:**
- Panic/recover is for exceptional situations (bugs, invariant violations)
- Errors are for expected failure conditions
- Don't use panic for control flow
- Most Go code should never panic

### Frequently Asked Questions

**Q1: Why doesn't Go have a while loop?**

**A:** Go's `for` loop covers all cases that `while` and `do-while` cover in other languages:

```go
// while (condition)
for condition {
    // body
}

// do-while equivalent
for {
    // body
    if !condition {
        break
    }
}

// Traditional for
for i := 0; i < 10; i++ {
    // body
}
```

**Why This Matters:** Having one loop construct reduces cognitive load. You don't have to choose between `for`, `while`, and `do-while` - there's only one way to loop.

**Related Concepts:** Simplicity in design, orthogonality

---

**Q2: When should I use defer vs just calling a function at the end?**

**A:** Use `defer` when:
1. You want to guarantee cleanup even if function panics
2. You want to avoid forgetting cleanup in early returns
3. You want cleanup code near acquisition code

```go
// ✅ WITH DEFER: Cleanup guaranteed
func process() error {
    f, err := os.Open("file.txt")
    if err != nil {
        return err
    }
    defer f.Close()  // Cleanup right next to acquisition
    
    // Multiple return paths
    if someCondition {
        return earlyError
    }
    
    // All paths close the file
    return nil
}

// ❌ WITHOUT DEFER: Easy to forget cleanup
func process() error {
    f, err := os.Open("file.txt")
    if err != nil {
        return err
    }
    
    if someCondition {
        // Oops! Forgot to close file
        return earlyError
    }
    
    f.Close()
    return nil
}
```

**Why This Matters:** Defer prevents resource leaks and makes cleanup code more maintainable.

---

**Q3: What's the execution order of multiple defers?**

**A:** Defers execute in LIFO (Last In, First Out) order:

```go
func example() {
    defer fmt.Println("First defer")
    defer fmt.Println("Second defer")
    defer fmt.Println("Third defer")
}
// Output:
// Third defer
// Second defer
// First defer
```

This is useful for properly unwinding resources:

```go
func nested() {
    lockA.Lock()
    defer lockA.Unlock()  // Unlocks last
    
    lockB.Lock()
    defer lockB.Unlock()  // Unlocks first
    
    // Critical section with both locks
}
// Unlocks in reverse order: B then A
// Avoids deadlock if locks have strict ordering
```

---

### Interview Questions

**Question 1: What will this code print and why?**

```go
func main() {
    for i := 0; i < 3; i++ {
        defer fmt.Println(i)
    }
}
```

**Difficulty:** Junior

**Answer:**

Output:
```
2
1
0
```

Explanation:
- `defer` statements are evaluated when encountered but executed when function returns
- Multiple defers execute in LIFO (Last In, First Out) order
- The argument `i` is evaluated when `defer` is called, not when it executes
- Loop iterations create defers: defer print(0), defer print(1), defer print(2)
- At function return, they execute in reverse order: 2, 1, 0

**What Interviewers Look For:**
- Understanding defer execution timing
- Knowledge of LIFO ordering
- Awareness that defer arguments are evaluated immediately

**Follow-up Questions:**
- What if we used `defer func() { fmt.Println(i) }()` instead?
- How would you print 0, 1, 2 using defer?
- When is defer argument evaluation useful?

---

**Question 2: Explain the difference between panic and returning an error. When should you use each?**

**Difficulty:** Mid-Level

**Answer:**

**Panic:**
- For programming errors and bugs
- For unrecoverable situations
- For invariant violations
- Examples: index out of bounds, nil pointer dereference, impossible state

**Error:**
- For expected failure conditions
- For situations the caller can handle
- For normal operation failures
- Examples: file not found, network timeout, invalid input

```go
// ✅ Use error for expected failures
func OpenFile(name string) (*File, error) {
    f, err := os.Open(name)
    if err != nil {
        return nil, err  // File might not exist - normal
    }
    return f, nil
}

// ✅ Use panic for programmer errors
func GetElement(slice []int, index int) int {
    if index < 0 || index >= len(slice) {
        panic("index out of bounds")  // Bug in caller code
    }
    return slice[index]
}

// ❌ DON'T panic for normal errors
func Divide(a, b int) int {
    if b == 0 {
        panic("division by zero")  // WRONG!
    }
    return a / b
}

// ✅ Return error instead
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

**What Interviewers Look For:**
- Clear distinction between errors and panics
- Understanding of Go's error handling philosophy
- Ability to identify appropriate use cases
- Knowledge that panic/recover is NOT try/catch

**Follow-up Questions:**
- How do you recover from a panic?
- Should a library function ever panic?
- What's the performance difference between panic and error?

---

**Question 3: What's wrong with this code?**

```go
values := []int{1, 2, 3, 4, 5}
for _, v := range values {
    go func() {
        fmt.Println(v)
    }()
}
time.Sleep(time.Second)
```

**Difficulty:** Mid-Level

**Answer:**

The problem: All goroutines will likely print `5` (or possibly different values, but not 1-5 reliably).

**Why:**
- The loop variable `v` is reused in each iteration
- The goroutine's closure captures the variable `v`, not its value
- By the time goroutines execute, the loop has finished and `v` holds the last value (5)

**Solutions:**

```go
// ✅ Solution 1: Pass value as argument
for _, v := range values {
    go func(val int) {
        fmt.Println(val)
    }(v)
}

// ✅ Solution 2: Create new variable in loop
for _, v := range values {
    v := v  // Creates new v for this iteration
    go func() {
        fmt.Println(v)
    }()
}

// ✅ Solution 3: Use index
for i := range values {
    go func(idx int) {
        fmt.Println(values[idx])
    }(i)
}
```

**What Interviewers Look For:**
- Understanding of closures and variable capture
- Knowledge of goroutine execution timing
- Awareness of common concurrency pitfalls
- Ability to provide multiple solutions

**Follow-up Questions:**
- Does this problem exist without goroutines?
- How does Go 1.22+ address this?
- What if we used a WaitGroup instead of Sleep?

---

### Key Takeaways

✅ **Go has simple, consistent control flow:**
- No parentheses required in conditions
- Braces always required (even for one line)
- No truthiness - conditions must be boolean

✅ **Only one loop: for:**
- Can express while, do-while, and traditional for
- Range provides clean iteration over collections
- Watch out for loop variable capture in closures

✅ **defer is powerful for cleanup:**
- Guarantees execution before function returns
- Executes in LIFO order
- Arguments evaluated immediately
- Perfect for resource cleanup (files, locks, connections)

✅ **panic is not exception handling:**
- Use errors for expected failures
- Use panic for programming errors
- Don't use panic/recover for control flow
- Most production Go code should never panic

✅ **Type switches are idiomatic:**
- Clean way to handle interface{} values
- Better than type assertions chains
- Common in reflection and generic code

---

## 1.4 Functions

Functions are first-class citizens in Go. Understanding functions deeply—from basics through advanced patterns—is crucial for writing idiomatic Go.

### Function Basics

#### **Function Declaration**

```go
// Basic function
func add(a int, b int) int {
    return a + b
}

// Consecutive parameters of same type
func add(a, b int) int {
    return a + b
}

// Multiple return values
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Named return values
func divide(a, b int) (result int, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return  // Naked return uses named return values
    }
    result = a / b
    return
}
```

#### **Multiple Return Values**

Go's multiple return values enable clean error handling:

```go
func ReadFile(name string) ([]byte, error) {
    data, err := os.ReadFile(name)
    if err != nil {
        return nil, err
    }
    return data, nil
}

// Usage
data, err := ReadFile("config.json")
if err != nil {
    log.Fatal(err)
}
// use data
```

**Ignoring return values:**

```go
value, err := someFunction()
// Use both

value, _ := someFunction()
// Ignore error (discouraged unless you know error can't happen)

_, err := someFunction()
// Ignore value, only check error
```

#### **Named Return Values**

```go
// Named returns document intent
func calculate(x, y int) (sum, product int) {
    sum = x + y
    product = x * y
    return  // Naked return
}

// Useful for defer
func processFile() (err error) {
    f, err := os.Open("file.txt")
    if err != nil {
        return  // Returns the err
    }
    
    defer func() {
        if closeErr := f.Close(); closeErr != nil && err == nil {
            err = closeErr  // Can set return value in defer
        }
    }()
    
    // process file...
    return
}
```

**When to use named returns:**
- Short functions where names add clarity
- When using defer to modify return values
- When function has many returns of same type

**When to avoid:**
- Long functions (naked returns can be unclear)
- When names don't add information (`result int` isn't helpful)

#### **Variadic Functions**

Functions that accept variable number of arguments:

```go
func sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}

// Usage
sum(1, 2, 3)
sum(1, 2, 3, 4, 5)
sum()  // OK: numbers is empty slice

// Pass slice with ...
values := []int{1, 2, 3, 4, 5}
sum(values...)  // Expands slice to arguments
```

**Variadic parameter must be last:**

```go
// ✅ Valid
func printf(format string, args ...interface{}) {
    // ...
}

// ❌ Invalid
func example(args ...int, format string) {  // Error!
    // ...
}
```

**fmt.Printf uses variadic:**

```go
fmt.Printf("Name: %s, Age: %d\n", name, age)
// Signature: func Printf(format string, a ...interface{}) (n int, err error)
```

#### **Function Types**

Functions are types and can be assigned to variables:

```go
// Function type
type BinaryOp func(int, int) int

// Functions matching the signature
func add(a, b int) int { return a + b }
func multiply(a, b int) int { return a * b }

// Assign to variable
var op BinaryOp
op = add
fmt.Println(op(3, 4))  // 7

op = multiply
fmt.Println(op(3, 4))  // 12

// Pass as argument
func apply(a, b int, op BinaryOp) int {
    return op(a, b)
}

result := apply(3, 4, add)  // 7
```

#### **Anonymous Functions and Closures**

```go
// Anonymous function
func() {
    fmt.Println("Hello from anonymous function")
}()  // Immediately invoked

// Assign to variable
greet := func(name string) {
    fmt.Printf("Hello, %s!\n", name)
}
greet("Alice")

// Closures capture variables from outer scope
func makeCounter() func() int {
    count := 0
    return func() int {
        count++  // Captures count from outer scope
        return count
    }
}

counter := makeCounter()
fmt.Println(counter())  // 1
fmt.Println(counter())  // 2
fmt.Println(counter())  // 3
```

**Closure example - iterator pattern:**

```go
func fibonacci() func() int {
    a, b := 0, 1
    return func() int {
        result := a
        a, b = b, a+b
        return result
    }
}

fib := fibonacci()
for i := 0; i < 10; i++ {
    fmt.Println(fib())
}
// Output: 0 1 1 2 3 5 8 13 21 34
```

### Advanced Function Patterns

#### **Higher-Order Functions**

Functions that take or return functions:

```go
// Map function
func mapInts(slice []int, fn func(int) int) []int {
    result := make([]int, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Usage
nums := []int{1, 2, 3, 4, 5}
doubled := mapInts(nums, func(x int) int {
    return x * 2
})
// doubled: [2, 4, 6, 8, 10]

// Filter function
func filterInts(slice []int, fn func(int) bool) []int {
    result := []int{}
    for _, v := range slice {
        if fn(v) {
            result = append(result, v)
        }
    }
    return result
}

// Usage
evens := filterInts(nums, func(x int) bool {
    return x%2 == 0
})
// evens: [2, 4]
```

**Reduce pattern:**

```go
func reduce(slice []int, initial int, fn func(int, int) int) int {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

nums := []int{1, 2, 3, 4, 5}
sum := reduce(nums, 0, func(acc, x int) int {
    return acc + x
})
// sum: 15
```

#### **Function Composition**

```go
// Compose two functions
func compose(f, g func(int) int) func(int) int {
    return func(x int) int {
        return f(g(x))
    }
}

double := func(x int) int { return x * 2 }
addOne := func(x int) int { return x + 1 }

// doubleAndAddOne applies addOne then double
doubleAndAddOne := compose(double, addOne)
fmt.Println(doubleAndAddOne(5))  // (5+1)*2 = 12
```

#### **Currying and Partial Application**

```go
// Currying: Transform multi-argument function into sequence of single-argument functions
func add(a int) func(int) int {
    return func(b int) int {
        return a + b
    }
}

add5 := add(5)
fmt.Println(add5(3))  // 8
fmt.Println(add5(10)) // 15

// Partial application
func makeAdder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}

add10 := makeAdder(10)
fmt.Println(add10(5))  // 15
```

#### **Decorator Pattern**

```go
// Logging decorator
func withLogging(fn func(int) int) func(int) int {
    return func(x int) int {
        fmt.Printf("Calling with %d\n", x)
        result := fn(x)
        fmt.Printf("Result: %d\n", result)
        return result
    }
}

square := func(x int) int { return x * x }
loggedSquare := withLogging(square)

loggedSquare(5)
// Output:
// Calling with 5
// Result: 25

// Timing decorator
func withTiming(fn func()) func() {
    return func() {
        start := time.Now()
        fn()
        fmt.Printf("Took: %v\n", time.Since(start))
    }
}
```

#### **Callback Patterns**

```go
type CompletionHandler func(result string, err error)

func fetchData(url string, callback CompletionHandler) {
    go func() {
        // Simulate network request
        time.Sleep(time.Second)
        
        resp, err := http.Get(url)
        if err != nil {
            callback("", err)
            return
        }
        defer resp.Body.Close()
        
        body, err := io.ReadAll(resp.Body)
        callback(string(body), err)
    }()
}

// Usage
fetchData("https://api.example.com", func(result string, err error) {
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Success:", result)
})
```

#### **Options Pattern for Configuration**

A popular Go idiom for configurable constructors:

```go
type Server struct {
    host    string
    port    int
    timeout time.Duration
    maxConn int
}

// Option is a function that modifies Server
type Option func(*Server)

// Option constructors
func WithHost(host string) Option {
    return func(s *Server) {
        s.host = host
    }
}

func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.timeout = timeout
    }
}

// Constructor with options
func NewServer(opts ...Option) *Server {
    // Default values
    s := &Server{
        host:    "localhost",
        port:    8080,
        timeout: 30 * time.Second,
        maxConn: 100,
    }
    
    // Apply options
    for _, opt := range opts {
        opt(s)
    }
    
    return s
}

// Usage
server := NewServer(
    WithHost("0.0.0.0"),
    WithPort(9000),
    WithTimeout(60 * time.Second),
)
```

**Benefits of options pattern:**
- Backward compatible (adding new options doesn't break existing code)
- Clear and readable
- Provides sensible defaults
- No need for multiple constructors

### Methods

Methods are functions with a receiver - they're associated with a type.

#### **Method Declaration**

```go
type Rectangle struct {
    Width  float64
    Height float64
}

// Method with value receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Method with pointer receiver
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// Usage
rect := Rectangle{Width: 10, Height: 5}
fmt.Println(rect.Area())  // 50

rect.Scale(2)
fmt.Println(rect.Area())  // 200 (modified)
```

#### **Value Receivers vs Pointer Receivers**

```go
type Counter struct {
    count int
}

// Value receiver - doesn't modify original
func (c Counter) IncrementValue() {
    c.count++  // Modifies copy
}

// Pointer receiver - modifies original
func (c *Counter) IncrementPointer() {
    c.count++  // Modifies original
}

// Demo
c := Counter{count: 0}
c.IncrementValue()
fmt.Println(c.count)  // 0 (unchanged)

c.IncrementPointer()
fmt.Println(c.count)  // 1 (changed)
```

**When to use pointer vs value receiver:**

```go
// ✅ Use POINTER receiver when:
// 1. Method modifies the receiver
type Account struct {
    balance int
}

func (a *Account) Deposit(amount int) {
    a.balance += amount  // Modifies state
}

// 2. Receiver is large (avoid copying)
type LargeStruct struct {
    data [10000]int
}

func (ls *LargeStruct) Process() {
    // Avoid copying 10000 integers
}

// 3. For consistency (if any method has pointer receiver, all should)
type Person struct {
    name string
    age  int
}

func (p *Person) Birthday() {
    p.age++
}

func (p *Person) GetAge() int {  // Use pointer for consistency
    return p.age
}

// ✅ Use VALUE receiver when:
// 1. Receiver is small and won't be modified
type Point struct {
    X, Y int
}

func (p Point) Distance() float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}

// 2. Receiver is a map, func, or chan (already reference types)
type Handler func(http.ResponseWriter, *http.Request)

func (h Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    h(w, r)
}
```

**Automatic dereferencing:**

Go automatically dereferences pointers when calling methods:

```go
type Dog struct {
    name string
}

func (d *Dog) Bark() {
    fmt.Printf("%s says Woof!\n", d.name)
}

dog := Dog{name: "Buddy"}
dog.Bark()  // Go automatically takes address: (&dog).Bark()

ptr := &Dog{name: "Max"}
ptr.Bark()  // Works directly
```

#### **Methods on Any Type**

You can define methods on any type you define (not just structs):

```go
type Celsius float64
type Fahrenheit float64

func (c Celsius) ToFahrenheit() Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}

func (f Fahrenheit) ToCelsius() Celsius {
    return Celsius((f - 32) * 5 / 9)
}

temp := Celsius(100)
fmt.Println(temp.ToFahrenheit())  // 212
```

**Cannot define methods on types from other packages:**

```go
// ❌ Cannot define method on built-in type
func (s string) IsPalindrome() bool {  // Error!
    // ...
}

// ✅ Create custom type first
type MyString string

func (s MyString) IsPalindrome() bool {
    // ...
}
```

#### **Method Expressions and Method Values**

```go
type Counter struct {
    count int
}

func (c *Counter) Increment() {
    c.count++
}

// Method expression: function that takes receiver as first argument
increment := (*Counter).Increment
c := &Counter{}
increment(c)  // Calls c.Increment()

// Method value: function bound to specific receiver
c2 := &Counter{}
inc := c2.Increment  // Bound to c2
inc()  // Calls c2.Increment()
fmt.Println(c2.count)  // 1
```

### Frequently Asked Questions (Functions)

**Q1: What's the difference between a function and a method?**

**A:** A method is a function with a receiver (associated with a type):

```go
// Function - standalone
func Add(a, b int) int {
    return a + b
}

// Method - associated with a type
type Calculator struct{}

func (c Calculator) Add(a, b int) int {
    return a + b
}

// Usage
result1 := Add(3, 4)  // Function call
calc := Calculator{}
result2 := calc.Add(3, 4)  // Method call
```

**Why This Matters:** Methods enable organizing code around types and are necessary for implementing interfaces.

**Related Concepts:** Receivers, interfaces, object-oriented patterns

---

**Q2: Should I use named return values or explicit returns?**

**A:** Use named returns when:
- They document the return values clearly
- You're using defer to modify returns
- The function is short and simple

Avoid in long functions where naked returns can be confusing.

```go
// ✅ Good use of named returns
func divide(a, b int) (quotient, remainder int, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return
    }
    quotient = a / b
    remainder = a % b
    return
}

// ✅ Also good: explicit returns for clarity
func divide(a, b int) (int, int, error) {
    if b == 0 {
        return 0, 0, errors.New("division by zero")
    }
    return a / b, a % b, nil
}
```

---

### Interview Questions (Functions)

**Question 1: Explain what a closure is and provide a practical example.**

**Difficulty:** Mid-Level

**Answer:**

A closure is a function that captures variables from its outer scope. The captured variables persist even after the outer function returns.

```go
func makeMultiplier(factor int) func(int) int {
    // factor is captured by the returned function
    return func(x int) int {
        return x * factor
    }
}

double := makeMultiplier(2)
triple := makeMultiplier(3)

fmt.Println(double(5))  // 10
fmt.Println(triple(5))  // 15
```

Practical example - rate limiter:

```go
func newRateLimiter(maxRequests int, window time.Duration) func() bool {
    requests := 0
    windowStart := time.Now()
    
    return func() bool {
        now := time.Now()
        
        // Reset window if expired
        if now.Sub(windowStart) > window {
            requests = 0
            windowStart = now
        }
        
        // Check limit
        if requests >= maxRequests {
            return false
        }
        
        requests++
        return true
    }
}

limiter := newRateLimiter(100, time.Minute)
if limiter() {
    // Process request
}
```

**What Interviewers Look For:**
- Clear explanation of variable capture
- Understanding of scope
- Practical use cases
- Awareness of memory implications (captured variables stay in memory)

---

**Question 2: When should you use a pointer receiver vs a value receiver for a method?**

**Difficulty:** Mid-Level

**Answer:**

Use **pointer receiver** when:
1. Method modifies the receiver
2. Receiver is a large struct (avoid copying)
3. For consistency across methods

Use **value receiver** when:
1. Receiver is small and won't be modified
2. Receiver is a primitive or small struct
3. Want to enforce immutability

```go
type Point struct {
    X, Y int
}

// ✅ Value receiver - small, immutable
func (p Point) Distance() float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}

type Database struct {
    conn *sql.DB
    // ... many fields
}

// ✅ Pointer receiver - large, mutable
func (db *Database) Query(sql string) (*Rows, error) {
    return db.conn.Query(sql)
}
```

**Rule of thumb:** If any method needs a pointer receiver, use pointer receivers for all methods on that type for consistency.

---

### Key Takeaways (Functions)

✅ **Multiple return values enable clean error handling**
- Return (value, error) is idiomatic
- Check errors immediately
- Don't ignore errors with `_` unless you're certain

✅ **Closures capture outer scope variables**
- Powerful for callbacks and configuration
- Be aware of captured variables in loops
- Can lead to memory retention

✅ **Defer is essential for cleanup**
- Use for closing files, unlocking mutexes
- Executes in LIFO order
- Arguments evaluated immediately

✅ **Choose receiver type carefully**
- Pointer for modification or large structs
- Value for small, immutable types
- Be consistent within a type

✅ **Higher-order functions are powerful**
- Functions as arguments enable flexibility
- Options pattern for configuration
- Decorator pattern for cross-cutting concerns

---

## 1.5 Interfaces

Interfaces are one of Go's most powerful features. They enable polymorphism, abstraction, and extremely flexible code design. Unlike many languages, Go interfaces are **implicit** - types satisfy interfaces automatically without declaring intent.

### Interface Fundamentals

#### **What is an Interface?**

An interface is a type that specifies a set of method signatures. Any type that has those methods automatically satisfies the interface:

```go
// Define an interface
type Writer interface {
    Write([]byte) (int, error)
}

// Any type with this method satisfies Writer
type FileWriter struct {
    file *os.File
}

func (fw *FileWriter) Write(p []byte) (int, error) {
    return fw.file.Write(p)
}

// FileWriter automatically satisfies Writer interface
var w Writer = &FileWriter{file: someFile}
```

**Key characteristics:**
- **Implicit satisfaction** - no "implements" keyword needed
- **Duck typing** at compile time - "if it walks like a duck..."
- **Zero value is nil** - uninitialized interface is nil

#### **Interface Declaration**

```go
// Simple interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Interface with multiple methods
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}

// Empty interface (accepts any type)
type Any interface{}  // or: interface{}

// Go 1.18+: use `any` as builtin
var x any = "hello"
var y any = 42
```

#### **Implementing Interfaces**

```go
type Animal interface {
    Speak() string
    Move() string
}

type Dog struct {
    Name string
}

func (d Dog) Speak() string {
    return "Woof!"
}

func (d Dog) Move() string {
    return "Running"
}

// Dog implicitly satisfies Animal
var animal Animal = Dog{Name: "Buddy"}
fmt.Println(animal.Speak())  // "Woof!"

type Cat struct {
    Name string
}

func (c Cat) Speak() string {
    return "Meow!"
}

func (c Cat) Move() string {
    return "Stalking"
}

// Cat also satisfies Animal
animal = Cat{Name: "Whiskers"}
fmt.Println(animal.Speak())  // "Meow!"
```

#### **Empty Interface**

The empty interface `interface{}` (or `any` in Go 1.18+) can hold values of any type:

```go
func describe(i interface{}) {
    fmt.Printf("Type: %T, Value: %v\n", i, i)
}

describe(42)           // Type: int, Value: 42
describe("hello")      // Type: string, Value: hello
describe(true)         // Type: bool, Value: true
describe([]int{1,2,3}) // Type: []int, Value: [1 2 3]

// Container for any type
type Container struct {
    value interface{}
}

c := Container{value: "can hold anything"}
```

**Common uses:**
- fmt.Println (accepts ...interface{})
- JSON unmarshaling (map[string]interface{})
- Generic containers (pre-generics)

#### **Type Assertions**

Extract the concrete value from an interface:

```go
var i interface{} = "hello"

// Type assertion (unsafe - panics if wrong type)
s := i.(string)
fmt.Println(s)  // "hello"

// Safe type assertion with ok pattern
s, ok := i.(string)
if ok {
    fmt.Println("It's a string:", s)
} else {
    fmt.Println("Not a string")
}

// Wrong type assertion (without ok) panics
// n := i.(int)  // ❌ PANIC: interface conversion: interface {} is string, not int
```

#### **Type Switches**

Perform different actions based on concrete type:

```go
func process(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    case Person:
        fmt.Printf("Person: %+v\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

process(42)        // Integer: 42
process("hello")   // String: hello
process(true)      // Boolean: true
```

### Interface Values

An interface value holds two things:
1. **Type** - the concrete type of the value
2. **Value** - the actual value

```
┌─────────────┐
│  type: *Dog │
├─────────────┤
│ value: &Dog{│
│  Name:"Max" │
│}            │
└─────────────┘
```

**nil Interface vs nil Concrete Value:**

```go
var w io.Writer  // nil interface
fmt.Println(w == nil)  // true

var buf *bytes.Buffer = nil
w = buf  // w holds (*bytes.Buffer, nil)
fmt.Println(w == nil)  // FALSE! Interface is not nil

// Check for nil concrete value
if w == nil || reflect.ValueOf(w).IsNil() {
    // Handle nil
}
```

This is a common gotcha:

```go
func returnsError() error {
    var err *MyError = nil
    // ...
    return err  // ❌ Returns non-nil interface with nil concrete value!
}

if err := returnsError(); err != nil {
    // This branch is taken even though conceptual error is nil!
}

// ✅ CORRECT: Return explicit nil
func returnsError() error {
    var err *MyError = nil
    if err == nil {
        return nil  // Return typed nil
    }
    return err
}
```

### Small Interfaces (The Go Way)

**Go philosophy: Small interfaces are better**

```go
// ✅ GOOD: Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// ✅ GOOD: Compose small interfaces
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// ❌ AVOID: Large interfaces
type HugeInterface interface {
    Method1()
    Method2()
    Method3()
    Method4()
    Method5()
    Method6()
    // ... many more
}
```

**Benefits of small interfaces:**
- Easier to implement
- More reusable
- More flexible
- Better testability

**Famous Go proverb:**
> "The bigger the interface, the weaker the abstraction." - Rob Pike

### Common Standard Library Interfaces

#### **io.Reader and io.Writer**

The most important interfaces in Go:

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Examples that implement Reader:
// - *os.File
// - *bytes.Buffer
// - *strings.Reader
// - *bufio.Reader
// - http.Response.Body

// Examples that implement Writer:
// - *os.File
// - *bytes.Buffer
// - *bufio.Writer
// - http.ResponseWriter
```

**Using io.Reader:**

```go
func processData(r io.Reader) error {
    data, err := io.ReadAll(r)
    if err != nil {
        return err
    }
    // process data...
    return nil
}

// Works with files
file, _ := os.Open("data.txt")
processData(file)

// Works with strings
processData(strings.NewReader("some data"))

// Works with HTTP responses
resp, _ := http.Get("https://example.com")
processData(resp.Body)
```

#### **Stringer Interface**

```go
type Stringer interface {
    String() string
}

// Implemented by types that have string representation
type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s (%d years old)", p.Name, p.Age)
}

p := Person{"Alice", 30}
fmt.Println(p)  // Uses String() method: "Alice (30 years old)"
```

#### **error Interface**

```go
type error interface {
    Error() string
}

// Custom error type
type ValidationError struct {
    Field string
    Issue string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Issue)
}

// Usage
func validateAge(age int) error {
    if age < 0 {
        return ValidationError{"age", "cannot be negative"}
    }
    return nil
}
```

### Composing Interfaces

Interfaces can embed other interfaces:

```go
// Basic interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// Composed interfaces
type ReadWriter interface {
    Reader  // Embedding Reader
    Writer  // Embedding Writer
}

type ReadCloser interface {
    Reader
    Closer
}

type WriteCloser interface {
    Writer
    Closer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// Any type implementing all methods satisfies composed interface
type File struct {
    *os.File
}

// File satisfies ReadWriteCloser if it has Read, Write, and Close methods
```

### Interface Design Patterns

#### **Accept Interfaces, Return Structs**

Go idiom: Functions should accept interfaces but return concrete types:

```go
// ✅ GOOD: Accept interface
func SaveUser(w io.Writer, user User) error {
    data, err := json.Marshal(user)
    if err != nil {
        return err
    }
    _, err = w.Write(data)
    return err
}

// ✅ GOOD: Return concrete type
func LoadUser(r io.Reader) (User, error) {
    var user User
    decoder := json.NewDecoder(r)
    err := decoder.Decode(&user)
    return user, err  // Returns User, not interface
}
```

**Why?**
- Accepting interfaces: Maximum flexibility for callers
- Returning concrete types: Clear what you're getting, access to all methods

#### **Interface Segregation**

Define interfaces at point of use, not with the type:

```go
// ❌ BAD: Define interface with the type
package user

type User struct {
    ID   int
    Name string
}

type UserRepository interface {  // Don't define here
    Save(User) error
    Find(int) (User, error)
}

// ✅ GOOD: Define interface where it's used
package handler

// Define only what this package needs
type UserSaver interface {
    Save(user.User) error
}

func CreateUserHandler(saver UserSaver) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Use saver...
    }
}
```

**Benefits:**
- Package doesn't depend on unnecessary methods
- Easier to test (mock only what's needed)
- More flexible (any type with Save method works)

#### **Strategy Pattern**

Use interfaces to swap algorithms:

```go
type Sorter interface {
    Sort([]int) []int
}

type QuickSort struct{}
func (q QuickSort) Sort(data []int) []int {
    // Quick sort implementation
    return data
}

type MergeSort struct{}
func (m MergeSort) Sort(data []int) []int {
    // Merge sort implementation
    return data
}

func ProcessData(data []int, sorter Sorter) []int {
    return sorter.Sort(data)
}

// Usage
ProcessData(nums, QuickSort{})
ProcessData(nums, MergeSort{})
```

### Interface Internals

**eface (empty interface):**

```
┌────────────┐
│   _type    │──→ type information
├────────────┤
│    data    │──→ actual data
└────────────┘
```

**iface (non-empty interface):**

```
┌────────────┐
│    tab     │──→ type + method table
├────────────┤
│    data    │──→ actual data
└────────────┘
```

**Method lookup:**
- At compile time: Go ensures all methods exist
- At runtime: Method calls use the method table for dynamic dispatch
- Cost: One pointer indirection

**Performance implications:**
- Interface calls are slightly slower than direct calls (virtual dispatch)
- Small interfaces are generally cheap
- Escape analysis: Values often escape to heap when assigned to interface

```go
// Value on stack
x := 42

// Escapes to heap when assigned to interface
var i interface{} = x  // x copied to heap

// Check with escape analysis
// go build -gcflags='-m'
```

### Frequently Asked Questions (Interfaces)

**Q1: What's the difference between nil interface and nil concrete value in interface?**

**A:** 

```go
// Nil interface: both type and value are nil
var w io.Writer = nil
fmt.Println(w == nil)  // true

// Nil concrete value: type is set, but value is nil
var buf *bytes.Buffer = nil  // buf is nil
var w2 io.Writer = buf       // w2 is NOT nil
fmt.Println(w2 == nil)       // FALSE!

// w2 holds (*bytes.Buffer, nil)
// The interface itself is not nil
```

**Why This Matters:** This causes bugs when returning errors:

```go
func returnsError() error {
    var err *MyError = nil
    return err  // ❌ Returns non-nil interface!
}

// ✅ CORRECT:
func returnsError() error {
    var err *MyError = nil
    if err == nil {
        return nil  // Return typed nil
    }
    return err
}
```

---

**Q2: Why does Go use implicit interface satisfaction instead of explicit declaration?**

**A:** Implicit interfaces provide several benefits:

1. **Decoupling:** Implementations don't depend on interface packages
2. **Post-hoc abstraction:** Can define interfaces after types exist
3. **Gradual refinement:** Can extract interfaces from existing code
4. **Testing:** Easy to mock without changing production code

```go
// Production code doesn't know about test interface
type Database struct { /*...*/ }
func (db *Database) Query(sql string) (*Rows, error) { /*...*/ }

// Test code defines interface it needs
type Querier interface {
    Query(string) (*Rows, error)
}

// Database satisfies Querier without knowing about it
// Easy to create mock that also satisfies Querier
```

---

**Q3: When should I use empty interface (interface{}) vs generics (Go 1.18+)?**

**A:** 

Use **empty interface** (`any`) for:
- Truly unknown types (e.g., JSON unmarshaling)
- Small number of type switches
- Backward compatibility (pre-Go 1.18)

Use **generics** for:
- Type-safe containers and algorithms
- When you need type safety at compile time
- When performance matters (avoid boxing)

```go
// ❌ Empty interface loses type safety
func Max(a, b interface{}) interface{} {
    // Need runtime type assertions
}

// ✅ Generics provide type safety
func Max[T constraints.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

---

### Interview Questions (Interfaces)

**Question 1: Explain what it means that interfaces are satisfied implicitly in Go. What are the advantages?**

**Difficulty:** Mid-Level

**Answer:**

Implicit interface satisfaction means a type doesn't need to declare that it implements an interface. If a type has all the methods an interface requires, it automatically satisfies that interface.

```go
type Writer interface {
    Write([]byte) (int, error)
}

type Logger struct{}

// Logger satisfies Writer without declaring it
func (l Logger) Write(p []byte) (int, error) {
    fmt.Println(string(p))
    return len(p), nil
}

// Can use Logger anywhere Writer is expected
var w Writer = Logger{}
```

**Advantages:**
1. **Decoupling:** Implementation doesn't depend on interface definition
2. **Post-hoc interfaces:** Can define interfaces for existing types
3. **Multiple interfaces:** Type can satisfy many interfaces without modification
4. **Testing:** Easy to create mocks without changing production code
5. **Evolution:** Can refine interfaces without updating all implementations

**What Interviewers Look For:**
- Understanding of structural typing vs nominal typing
- Awareness of decoupling benefits
- Knowledge of testing implications
- Ability to contrast with explicit interfaces (Java, C#)

**Follow-up Questions:**
- How does this help with testing?
- What are downsides of implicit interfaces?
- How do you know what interfaces a type satisfies?

---

**Question 2: What will this code print and why?**

```go
func main() {
    var i interface{}
    describe(i)
    
    i = 42
    describe(i)
    
    i = "hello"
    describe(i)
}

func describe(i interface{}) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

**Difficulty:** Junior

**Answer:**

Output:
```
(<nil>, <nil>)
(42, int)
(hello, string)
```

Explanation:
- First call: `i` is nil interface (both type and value are nil)
- Second call: `i` holds int value 42
- Third call: `i` holds string value "hello"

Empty interface can hold any type. The `%T` verb prints the dynamic type.

**What Interviewers Look For:**
- Understanding of empty interface
- Knowledge that interface can change type at runtime
- Awareness of nil interface vs nil concrete value

---

**Question 3: Why does this code not compile? How would you fix it?**

```go
type Animal interface {
    Speak() string
}

type Dog struct {
    Name string
}

func (d *Dog) Speak() string {
    return "Woof!"
}

func main() {
    animals := []Animal{
        Dog{Name: "Buddy"},  // Error!
    }
}
```

**Difficulty:** Mid-Level

**Answer:**

The code doesn't compile because `Dog` (value type) doesn't satisfy `Animal` interface - only `*Dog` (pointer type) does.

The `Speak` method has a pointer receiver `(d *Dog)`, so only `*Dog` satisfies the interface.

**Fixes:**

```go
// Fix 1: Use pointer in slice
animals := []Animal{
    &Dog{Name: "Buddy"},  // ✅
}

// Fix 2: Change to value receiver
func (d Dog) Speak() string {  // Value receiver
    return "Woof!"
}

animals := []Animal{
    Dog{Name: "Buddy"},  // ✅ Now works
}
```

**Method sets:**
- Type `T`: methods with receiver `T`
- Type `*T`: methods with receiver `T` OR `*T`

**What Interviewers Look For:**
- Understanding of method sets
- Knowledge of pointer vs value receivers
- Ability to debug interface satisfaction
- Awareness that receiver type affects interface satisfaction

---

### Key Takeaways (Interfaces)

✅ **Interfaces enable polymorphism in Go**
- Implicit satisfaction (duck typing at compile time)
- Accept interfaces, return structs
- Define interfaces at point of use

✅ **Small interfaces are better**
- Single-method interfaces are common (io.Reader, io.Writer)
- Compose small interfaces for larger ones
- Easier to implement and test

✅ **Empty interface accepts any type**
- Use sparingly (loses type safety)
- Prefer generics for type-safe code (Go 1.18+)
- Common in fmt, JSON, reflection

✅ **Interface values have type and value**
- nil interface: both are nil
- nil concrete value: type is set, value is nil
- This distinction causes bugs - be careful with error returns

✅ **Method sets matter**
- Value receiver: both T and *T satisfy interface
- Pointer receiver: only *T satisfies interface
- Choose receiver type carefully

---

## 1.6 Error Handling

Error handling in Go is explicit and based on values, not exceptions. This section covers the error interface, patterns, and best practices.

### The error Interface

```go
type error interface {
    Error() string
}
```

Any type with an `Error() string` method satisfies the error interface.

#### **Creating Errors**

```go
// 1. errors.New
err := errors.New("something went wrong")

// 2. fmt.Errorf (with formatting)
err = fmt.Errorf("failed to process user %s", username)

// 3. Custom error type
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

err = ValidationError{Field: "email", Message: "invalid format"}
```

#### **Checking Errors**

```go
// Standard pattern
result, err := someFunction()
if err != nil {
    // Handle error
    return err  // Or handle appropriately
}
// Use result

// ❌ Don't ignore errors
result, _ := someFunction()  // Dangerous!
```

### Error Wrapping (Go 1.13+)

```go
// Wrap errors with fmt.Errorf and %w
func processFile(filename string) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return fmt.Errorf("processFile %s: %w", filename, err)
    }
    // process data...
    return nil
}

// Check wrapped errors with errors.Is
err := processFile("config.json")
if errors.Is(err, os.ErrNotExist) {
    // File doesn't exist
}

// Extract wrapped errors with errors.As
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("Failed path:", pathErr.Path)
}
```

**Error chains:**

```go
err1 := errors.New("base error")
err2 := fmt.Errorf("middle: %w", err1)
err3 := fmt.Errorf("top: %w", err2)

// errors.Is checks entire chain
errors.Is(err3, err1)  // true

// errors.Unwrap gets immediate wrapping error
errors.Unwrap(err3)  // err2
errors.Unwrap(err2)  // err1
errors.Unwrap(err1)  // nil
```

### Error Patterns

#### **Sentinel Errors**

```go
var (
    ErrNotFound    = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrInvalidInput = errors.New("invalid input")
)

func FindUser(id int) (*User, error) {
    if user doesn't exist {
        return nil, ErrNotFound
    }
    return user, nil
}

// Check with errors.Is
user, err := FindUser(123)
if errors.Is(err, ErrNotFound) {
    // Handle not found
}
```

#### **Custom Error Types**

```go
type APIError struct {
    StatusCode int
    Message    string
    Err        error  // Wrapped error
}

func (e *APIError) Error() string {
    return fmt.Sprintf("API error %d: %s", e.StatusCode, e.Message)
}

func (e *APIError) Unwrap() error {
    return e.Err
}

// Usage
func callAPI() error {
    resp, err := http.Get(url)
    if err != nil {
        return &APIError{
            StatusCode: 0,
            Message:    "failed to connect",
            Err:        err,
        }
    }
    
    if resp.StatusCode != 200 {
        return &APIError{
            StatusCode: resp.StatusCode,
            Message:    "unexpected status",
        }
    }
    
    return nil
}

// Check
err := callAPI()
var apiErr *APIError
if errors.As(err, &apiErr) {
    fmt.Printf("API returned %d\n", apiErr.StatusCode)
}
```

### Error Handling Strategies

```go
// 1. Return error immediately
func process() error {
    if err := step1(); err != nil {
        return err
    }
    if err := step2(); err != nil {
        return err
    }
    return nil
}

// 2. Add context while returning
func process() error {
    if err := step1(); err != nil {
        return fmt.Errorf("step1 failed: %w", err)
    }
    return nil
}

// 3. Handle and continue
func process() {
    if err := nonCritical(); err != nil {
        log.Printf("warning: %v", err)
        // Continue execution
    }
}

// 4. Retry on specific errors
func processWithRetry() error {
    for i := 0; i < 3; i++ {
        err := attemptOperation()
        if err == nil {
            return nil
        }
        if !isRetryable(err) {
            return err
        }
        time.Sleep(time.Second * time.Duration(i+1))
    }
    return errors.New("max retries exceeded")
}
```

Continuing with final Part 1 sections...
## 1.7 Packages and Modules

Packages are Go's way of organizing code. Understanding the package system is essential for writing maintainable Go applications.

### Package Basics

#### **Package Declaration**

Every Go file starts with a package declaration:

```go
package main  // Executable program

package utils  // Library package

package database  // Library package
```

**Rules:**
- Package name must match directory name (except `main` and `_test`)
- All files in a directory must be in the same package
- Package name should be short, lowercase, single word

#### **Package Naming Conventions**

```go
// ✅ GOOD: Short, descriptive
package http
package json
package strings
package user

// ❌ AVOID: Long, complex names
package httputils
package jsonparser
package stringhelper
```

#### **Import Statements**

```go
import "fmt"
import "os"
import "net/http"

// Better: Group imports
import (
    "fmt"
    "os"
    
    "net/http"
    "github.com/gorilla/mux"
)

// Import with alias
import (
    "database/sql"
    mysql "github.com/go-sql-driver/mysql"
)

// Blank import (for side effects)
import _ "github.com/go-sql-driver/mysql"  // Registers driver

// Dot import (discouraged)
import . "fmt"  // Now can use Println without fmt.
```

#### **Exported vs Unexported**

Identifiers starting with uppercase are exported (public):

```go
package user

// Exported (public)
type User struct {
    ID   int     // Exported field
    Name string  // Exported field
}

func NewUser(id int, name string) *User {  // Exported function
    return &User{ID: id, Name: name}
}

// Unexported (private)
type session struct {  // Unexported type
    token string
}

func validateUser(u *User) bool {  // Unexported function
    return u.ID > 0
}
```

### Go Modules

Go modules (introduced in Go 1.11) are the modern dependency management system.

#### **Creating a Module**

```bash
# Initialize a new module
go mod init github.com/username/myproject

# This creates go.mod file
```

**go.mod file:**

```go
module github.com/username/myproject

go 1.24

require (
    github.com/gorilla/mux v1.8.0
    github.com/lib/pq v1.10.9
)

require (
    github.com/davecgh/go-spew v1.1.1 // indirect
)

exclude github.com/bad/package v1.0.0

replace github.com/old/package => github.com/new/package v2.0.0
```

#### **go.sum file**

Contains cryptographic checksums of dependencies:

```
github.com/gorilla/mux v1.8.0 h1:i40aqfkR1h2SlN9hojwV5ZA91wcXFOvkdNIeFDP5koI=
github.com/gorilla/mux v1.8.0/go.mod h1:DVbg23sWSpFRCP0SfiEN6jmj59UnW/n46BH5rLB71So=
```

**Purpose:**
- Ensures reproducible builds
- Prevents tampering
- Verifies dependencies haven't changed

#### **Adding Dependencies**

```bash
# Add dependency (automatically updates go.mod)
go get github.com/gorilla/mux@v1.8.0

# Add latest version
go get github.com/gorilla/mux@latest

# Add specific version
go get github.com/gorilla/mux@v1.7.0

# Add branch or commit
go get github.com/gorilla/mux@master
go get github.com/gorilla/mux@abcd1234

# Update all dependencies
go get -u ./...

# Update to patch versions only
go get -u=patch ./...
```

#### **Module Commands**

```bash
# Download dependencies
go mod download

# Remove unused dependencies
go mod tidy

# Copy dependencies to vendor/
go mod vendor

# Verify dependencies haven't changed
go mod verify

# Show dependency graph
go mod graph

# Explain why dependency is needed
go mod why github.com/some/package

# List all modules
go list -m all
```

### Project Structure

#### **Standard Go Project Layout**

```
myproject/
├── go.mod
├── go.sum
├── README.md
├── LICENSE
├── .gitignore
├── cmd/                    # Command-line applications
│   ├── server/
│   │   └── main.go        # Server executable
│   └── client/
│       └── main.go        # Client executable
├── internal/              # Private application code
│   ├── auth/
│   │   ├── auth.go
│   │   └── auth_test.go
│   └── database/
│       ├── database.go
│       └── database_test.go
├── pkg/                   # Public library code
│   └── models/
│       └── user.go
├── api/                   # API definitions (OpenAPI, protobuf)
│   └── openapi.yaml
├── web/                   # Web assets (templates, static files)
│   ├── templates/
│   └── static/
├── configs/              # Configuration files
│   └── config.yaml
├── scripts/              # Build and deployment scripts
│   └── build.sh
├── test/                 # Integration tests
│   └── integration_test.go
└── docs/                 # Documentation
    └── architecture.md
```

**Key directories:**

**cmd/**: Main applications
- Each subdirectory has a main package
- Thin, imports code from internal/ and pkg/

**internal/**: Private code
- Cannot be imported by other projects
- Enforced by Go compiler
- Use for code that shouldn't be public API

**pkg/**: Public library code
- Can be imported by other projects
- Use sparingly - most code should be internal/

**Example cmd/server/main.go:**

```go
package main

import (
    "log"
    "net/http"
    
    "github.com/username/myproject/internal/auth"
    "github.com/username/myproject/internal/database"
)

func main() {
    db, err := database.Connect()
    if err != nil {
        log.Fatal(err)
    }
    
    authHandler := auth.NewHandler(db)
    
    http.HandleFunc("/login", authHandler.Login)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

#### **Init Functions**

```go
package database

import "database/sql"

var db *sql.DB

// init runs automatically before main
func init() {
    var err error
    db, err = sql.Open("mysql", "connection-string")
    if err != nil {
        panic(err)  // OK to panic in init
    }
}

// Multiple init functions run in order declared
func init() {
    // Runs second
}
```

**init() characteristics:**
- Runs automatically before main()
- Runs once per package
- Can't be called explicitly
- Multiple init functions execute in declaration order
- Used for package initialization

#### **Internal Packages**

The `internal/` directory is special:

```
myproject/
├── internal/
│   └── auth/
│       └── auth.go
└── pkg/
    └── models/
        └── user.go

otherproject/
└── main.go
```

```go
// ✅ OK: Same project can import internal
// myproject/cmd/server/main.go
import "github.com/username/myproject/internal/auth"

// ❌ ERROR: Other projects cannot import internal
// otherproject/main.go
import "github.com/username/myproject/internal/auth"  // Compile error!
```

### Package-Level Variables and Constants

```go
package config

// Package-level constant
const MaxRetries = 3

// Package-level variable
var GlobalConfig *Config

// Private package variable
var defaultTimeout = 30 * time.Second

func init() {
    GlobalConfig = &Config{
        Timeout: defaultTimeout,
        Retries: MaxRetries,
    }
}
```

### Circular Dependencies

Go **forbids** circular dependencies between packages:

```go
// ❌ This won't compile
package a
import "myproject/b"

package b
import "myproject/a"  // Circular dependency!
```

**Solutions:**

1. **Extract common code to third package:**

```go
package common  // New package
// Common types/functions

package a
import "myproject/common"

package b
import "myproject/common"
```

2. **Use interfaces:**

```go
package a

type BService interface {
    DoSomething()
}

func UseB(b BService) {
    b.DoSomething()
}

package b
import "myproject/a"

type Service struct{}

func (s Service) DoSomething() {
    // Implementation
}

// No circular dependency!
```

### Private Modules

```bash
# Configure for private repositories
go env -w GOPRIVATE=github.com/yourcompany/*

# For git credentials
git config --global url."ssh://git@github.com/".insteadOf "https://github.com/"

# Or use .netrc for HTTPS
# ~/.netrc
machine github.com
login your-username
password your-token
```

### Frequently Asked Questions (Packages)

**Q1: What's the difference between internal/ and pkg/ directories?**

**A:**

**internal/**: Private code, cannot be imported by external projects
- Enforced by Go compiler
- Use for code that's implementation detail

**pkg/**: Public library code, can be imported by anyone
- Use sparingly
- Only for code you want others to import

**Example:**

```
myproject/
├── internal/
│   └── auth/       # Only myproject can import
│       └── auth.go
└── pkg/
    └── models/     # Anyone can import
        └── user.go
```

**Best practice:** Most code should be in internal/. Only move to pkg/ when you explicitly want to provide a public library.

---

**Q2: How do replace directives work in go.mod?**

**A:**

Replace directives substitute one module for another:

```go
// Replace with local version (for development)
replace github.com/someone/lib => ../local/lib

// Replace with different version
replace github.com/old/lib => github.com/old/lib v2.0.0

// Replace with fork
replace github.com/original/lib => github.com/yourfork/lib v1.0.0
```

Common uses:
- Local development (use local copy of dependency)
- Use fork instead of original
- Work around broken dependency
- Test changes before publishing

**Remove before publishing your module!**

---

### Interview Questions (Packages)

**Question 1: Why does Go not allow circular dependencies between packages?**

**Difficulty:** Mid-Level

**Answer:**

Circular dependencies cause several problems:
1. **Compilation order**: Can't determine which package to compile first
2. **Initialization**: init() functions can't have deterministic order
3. **Complexity**: Makes code harder to understand and maintain
4. **Testing**: Difficult to test packages in isolation

**How to avoid:**
1. Extract common code to a third package
2. Use interfaces to break dependency
3. Restructure packages by responsibility

```go
// ❌ PROBLEM
package a
import "b"
func UseB() { b.Something() }

package b
import "a"
func UseA() { a.Something() }

// ✅ SOLUTION: Extract interface
package common
type Doer interface { Do() }

package a
import "common"
func Use(d common.Doer) { d.Do() }

package b
import "common"
type Service struct{}
func (s Service) Do() {}
```

**What Interviewers Look For:**
- Understanding of compilation and initialization order
- Knowledge of dependency management
- Ability to refactor to avoid cycles
- Interface-based decoupling

---

**Question 2: Explain the difference between go get, go install, and go build.**

**Difficulty:** Junior

**Answer:**

**go build**: Compiles packages and dependencies
- Creates executable in current directory
- Doesn't install anything
- Use for testing compilation

```bash
go build              # Build current package
go build ./cmd/app    # Build specific package
go build -o myapp     # Custom output name
```

**go install**: Builds and installs executables
- Places binary in $GOPATH/bin or $GOBIN
- Updates go.mod if needed
- Use for installing tools

```bash
go install                          # Install current package
go install github.com/user/tool@latest  # Install from remote
```

**go get**: Downloads and updates dependencies
- Modifies go.mod
- Downloads source to module cache
- Use for adding dependencies

```bash
go get github.com/user/lib@v1.0.0   # Add/update dependency
go get -u ./...                      # Update all dependencies
```

**Summary:**
- `go build`: Compile only
- `go install`: Compile + install to $GOPATH/bin
- `go get`: Add/update dependencies

---

### Key Takeaways (Packages)

✅ **Packages organize Go code**
- One package per directory
- Package name matches directory (except main)
- Short, descriptive names

✅ **Exported vs unexported controls visibility**
- Uppercase = exported (public)
- Lowercase = unexported (private)
- Package is the unit of encapsulation

✅ **Go modules are the dependency system**
- go.mod declares dependencies
- go.sum ensures integrity
- go mod tidy keeps it clean

✅ **Project structure matters**
- cmd/ for executables
- internal/ for private code (enforced by compiler)
- pkg/ for public libraries (use sparingly)

✅ **No circular dependencies allowed**
- Enforced by compiler
- Use interfaces or extract common code
- Indicates design issues

---

## Part 1 Summary

Congratulations! You've completed Part 1 of the Complete Go Mastery series. You now have a solid foundation in Go fundamentals.

### What You've Learned

**Go's Philosophy:**
- Simplicity over features
- Explicit over implicit
- Composition over inheritance
- Fast compilation, easy deployment

**Language Fundamentals:**
- Types: basic types, composite types, zero values
- Variables: var, :=, constants, iota
- Control flow: if, switch, for, defer, panic/recover
- Functions: multiple returns, variadic, closures, methods
- Interfaces: implicit satisfaction, composition, common patterns
- Error handling: error interface, wrapping, patterns
- Packages: organization, modules, dependencies

**Key Principles:**
- Errors are values, not exceptions
- Interfaces are satisfied implicitly
- Small interfaces are better
- Accept interfaces, return structs
- Define interfaces at point of use

### What's Next

In **Part 2: Concurrency & Go Runtime Internals**, we'll dive into:

- Goroutines and the Go scheduler
- Channels and synchronization primitives
- Concurrency patterns
- Memory management and garbage collection
- Understanding the Go runtime
- Profiling and optimization

### Practice Exercises

Before moving to Part 2, solidify your understanding:

1. **Build a CLI tool** that processes files using io.Reader/Writer
2. **Create a package** with proper tests and documentation
3. **Implement common interfaces** (Stringer, error, io.Reader)
4. **Practice error handling** with wrapping and custom error types
5. **Design a small API** using interfaces and composition

### Additional Resources

- [Effective Go](https://golang.org/doc/effective_go) - Official style guide
- [Go Blog](https://blog.golang.org/) - Deep dives by Go team
- [Go by Example](https://gobyexample.com/) - Practical examples
- [Go Playground](https://play.golang.org/) - Try code online

---

**Ready to master Go concurrency?** Continue to Part 2 where we'll explore goroutines, channels, and the Go runtime in depth!

