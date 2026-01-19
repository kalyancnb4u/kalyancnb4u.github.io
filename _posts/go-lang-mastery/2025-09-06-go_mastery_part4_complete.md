---
title: "Go Mastery - Part 4: Testing, Benchmarking & Code Quality"
date: 2025-09-06 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Testing, Benchmarking, Profiling, Code-quality, TDD, Coverage, Fuzzing]
---

# Complete Go Mastery Part 4: Testing, Benchmarking & Code Quality

## Introduction

Welcome to Part 4 of the Complete Go Mastery series. In Parts 1-3, we covered Go fundamentals, concurrency, and advanced techniques. Now we focus on what separates good code from great code: **comprehensive testing and quality assurance**.

Go was designed with testing as a first-class citizen. The `testing` package is built into the standard library, and the `go test` command makes running tests trivial. But Go's testing capabilities go far beyond simple unit tests—they include benchmarking, profiling, fuzzing, and race detection.

**Why is testing critical?**

- **Confidence**: Ship code knowing it works correctly
- **Refactoring**: Change code without breaking functionality
- **Documentation**: Tests document expected behavior
- **Design**: Writing testable code improves design
- **Debugging**: Tests help isolate and fix bugs quickly

**What you'll learn in Part 4:**

- **Unit Testing**: Writing effective tests with table-driven patterns
- **Test Organization**: Structuring tests and using subtests
- **Mocking and Stubbing**: Testing with dependencies
- **Benchmarking**: Measuring and optimizing performance
- **Profiling**: CPU, memory, and blocking profiles
- **Fuzzing**: Finding edge cases automatically (Go 1.18+)
- **Code Coverage**: Measuring and improving coverage
- **Integration Testing**: Testing complete systems
- **Static Analysis**: Automated code quality checks
- **Best Practices**: Writing maintainable, reliable tests

By the end of Part 4, you'll be able to write comprehensive test suites that give you confidence in your code and help you deliver high-quality software.

---

## 4.1 Unit Testing Fundamentals

### Basic Test Structure

Go tests are regular Go functions with specific naming conventions:

```go
// math.go
package math

func Add(a, b int) int {
    return a + b
}

func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

```go
// math_test.go
package math

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    
    if result != expected {
        t.Errorf("Add(2, 3) = %d; want %d", result, expected)
    }
}

func TestDivide(t *testing.T) {
    result, err := Divide(10, 2)
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    
    if result != 5 {
        t.Errorf("Divide(10, 2) = %d; want 5", result)
    }
}

func TestDivideByZero(t *testing.T) {
    _, err := Divide(10, 0)
    if err == nil {
        t.Error("expected error for division by zero")
    }
}
```

**Running tests:**

```bash
go test                    # Run tests in current directory
go test ./...              # Run tests in all subdirectories
go test -v                 # Verbose output
go test -run TestAdd       # Run specific test
go test -run "TestAdd|TestDivide"  # Run multiple tests
```

### Test File Organization

**Naming conventions:**

```go
// Source file: calculator.go
// Test file: calculator_test.go

// Same package (white-box testing)
package calculator

// External package (black-box testing)
package calculator_test
```

**White-box vs Black-box testing:**

```go
// calculator.go
package calculator

func add(a, b int) int {  // unexported
    return a + b
}

func Calculate(a, b int) int {  // exported
    return add(a, b)
}

// calculator_test.go (white-box)
package calculator

func TestAdd(t *testing.T) {
    result := add(2, 3)  // ✅ Can test unexported functions
    if result != 5 {
        t.Errorf("add(2, 3) = %d; want 5", result)
    }
}

// calculator_external_test.go (black-box)
package calculator_test

import (
    "testing"
    "myproject/calculator"
)

func TestCalculate(t *testing.T) {
    result := calculator.Calculate(2, 3)
    // Can't access calculator.add (unexported)
    if result != 5 {
        t.Errorf("Calculate(2, 3) = %d; want 5", result)
    }
}
```

**When to use each:**
- **White-box**: Testing internal logic, private functions
- **Black-box**: Testing public API, avoiding import cycles

### Table-Driven Tests

The most idiomatic way to write Go tests:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed signs", -2, 3, 1},
        {"zero", 0, 0, 0},
        {"large numbers", 1000000, 2000000, 3000000},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", 
                    tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

**Benefits of table-driven tests:**
- Easy to add new test cases
- Clear test structure
- Each case has a name
- Can run specific cases
- Reduces code duplication

**Advanced table-driven pattern:**

```go
func TestDivide(t *testing.T) {
    tests := []struct {
        name      string
        a, b      int
        want      int
        wantErr   bool
        errMsg    string
    }{
        {
            name:    "normal division",
            a:       10,
            b:       2,
            want:    5,
            wantErr: false,
        },
        {
            name:    "division by zero",
            a:       10,
            b:       0,
            want:    0,
            wantErr: true,
            errMsg:  "division by zero",
        },
        {
            name:    "negative result",
            a:       10,
            b:       -2,
            want:    -5,
            wantErr: false,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Divide(tt.a, tt.b)
            
            if tt.wantErr {
                if err == nil {
                    t.Errorf("Divide() error = nil; want error")
                    return
                }
                if err.Error() != tt.errMsg {
                    t.Errorf("Divide() error = %q; want %q", 
                        err.Error(), tt.errMsg)
                }
                return
            }
            
            if err != nil {
                t.Errorf("Divide() unexpected error: %v", err)
                return
            }
            
            if got != tt.want {
                t.Errorf("Divide(%d, %d) = %d; want %d", 
                    tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

### Subtests

Organize tests hierarchically:

```go
func TestCalculator(t *testing.T) {
    t.Run("Addition", func(t *testing.T) {
        t.Run("Positive", func(t *testing.T) {
            if Add(2, 3) != 5 {
                t.Error("2 + 3 should equal 5")
            }
        })
        
        t.Run("Negative", func(t *testing.T) {
            if Add(-2, -3) != -5 {
                t.Error("-2 + -3 should equal -5")
            }
        })
    })
    
    t.Run("Subtraction", func(t *testing.T) {
        t.Run("Positive", func(t *testing.T) {
            if Subtract(5, 3) != 2 {
                t.Error("5 - 3 should equal 2")
            }
        })
    })
}
```

**Running specific subtests:**

```bash
go test -run TestCalculator/Addition
go test -run TestCalculator/Addition/Positive
```

### Test Helpers

Functions that help reduce test boilerplate:

```go
// Helper for checking errors
func assertNoError(t *testing.T, err error) {
    t.Helper()  // Marks this as a helper function
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

// Helper for checking equality
func assertEqual(t *testing.T, got, want interface{}) {
    t.Helper()
    if got != want {
        t.Errorf("got %v; want %v", got, want)
    }
}

// Helper for checking error messages
func assertError(t *testing.T, err error, wantMsg string) {
    t.Helper()
    if err == nil {
        t.Fatal("expected error but got nil")
    }
    if err.Error() != wantMsg {
        t.Errorf("error message = %q; want %q", err.Error(), wantMsg)
    }
}

// Usage in tests
func TestUser(t *testing.T) {
    user, err := CreateUser("alice@example.com")
    assertNoError(t, err)
    assertEqual(t, user.Email, "alice@example.com")
    
    _, err = CreateUser("invalid-email")
    assertError(t, err, "invalid email format")
}
```

**Why t.Helper()?**
- Marks function as test helper
- Stack traces point to the calling test, not the helper
- Makes debugging easier

### Testing Package vs Testing Function

**t.Error vs t.Fatal:**

```go
func TestMultipleChecks(t *testing.T) {
    result1 := Calculate(1)
    if result1 != 1 {
        t.Error("first check failed")  // Continues to next check
    }
    
    result2 := Calculate(2)
    if result2 != 2 {
        t.Error("second check failed")  // Still runs
    }
}

func TestDependentChecks(t *testing.T) {
    user, err := CreateUser("test@example.com")
    if err != nil {
        t.Fatal("cannot create user")  // Stops here
    }
    
    // This won't run if user creation failed
    if user.Email != "test@example.com" {
        t.Error("wrong email")
    }
}
```

**When to use each:**
- `t.Error()`: Continue running the test
- `t.Fatal()`: Stop the test immediately
- `t.Errorf()`: Error with formatted message
- `t.Fatalf()`: Fatal with formatted message

### Setup and Teardown

**TestMain for package-level setup:**

```go
func TestMain(m *testing.M) {
    // Setup
    fmt.Println("Setting up tests")
    db := setupTestDatabase()
    
    // Run tests
    code := m.Run()
    
    // Teardown
    fmt.Println("Cleaning up tests")
    db.Close()
    
    // Exit with test result code
    os.Exit(code)
}
```

**Per-test setup:**

```go
func setupTest(t *testing.T) *Database {
    t.Helper()
    
    db, err := NewDatabase()
    if err != nil {
        t.Fatalf("setup failed: %v", err)
    }
    
    // Cleanup after test
    t.Cleanup(func() {
        db.Close()
    })
    
    return db
}

func TestUserCreation(t *testing.T) {
    db := setupTest(t)
    
    // Test using db
    user, err := db.CreateUser("alice")
    if err != nil {
        t.Errorf("failed to create user: %v", err)
    }
    
    // Cleanup happens automatically via t.Cleanup()
}
```

**t.Cleanup() vs defer:**

```go
// ✅ t.Cleanup: Runs after subtests complete
func TestWithCleanup(t *testing.T) {
    resource := acquireResource()
    t.Cleanup(func() {
        resource.Release()
    })
    
    t.Run("subtest1", func(t *testing.T) {
        // resource still available
    })
    
    t.Run("subtest2", func(t *testing.T) {
        // resource still available
    })
    
    // resource.Release() called here
}

// ❌ defer: Runs before subtests complete
func TestWithDefer(t *testing.T) {
    resource := acquireResource()
    defer resource.Release()
    
    t.Run("subtest1", func(t *testing.T) {
        // resource might be released!
    })
}
```

### Testing Parallel Execution

```go
func TestParallel1(t *testing.T) {
    t.Parallel()  // Mark as parallel
    
    // This test runs in parallel with other parallel tests
    time.Sleep(time.Second)
}

func TestParallel2(t *testing.T) {
    t.Parallel()
    
    time.Sleep(time.Second)
}

func TestSequential(t *testing.T) {
    // Runs sequentially
    time.Sleep(time.Second)
}
```

**Running tests in parallel:**

```bash
go test -parallel 4  # Run up to 4 tests in parallel
```

**Important with table-driven tests:**

```go
func TestParallelTable(t *testing.T) {
    tests := []struct {
        name string
        val  int
    }{
        {"test1", 1},
        {"test2", 2},
    }
    
    for _, tt := range tests {
        tt := tt  // Capture range variable!
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            // Use tt safely
        })
    }
}
```

### Example Tests

Document usage with runnable examples:

```go
func ExampleAdd() {
    result := Add(2, 3)
    fmt.Println(result)
    // Output: 5
}

func ExampleDivide() {
    result, err := Divide(10, 2)
    if err != nil {
        fmt.Println("error:", err)
        return
    }
    fmt.Println(result)
    // Output: 5
}

func ExampleUser_Email() {
    user := User{Name: "Alice", Email: "alice@example.com"}
    fmt.Println(user.Email)
    // Output: alice@example.com
}

// Unordered output
func ExamplePrintMap() {
    m := map[string]int{"a": 1, "b": 2}
    for k, v := range m {
        fmt.Println(k, v)
    }
    // Unordered output:
    // a 1
    // b 2
}
```

**Benefits of example tests:**
- Verified by `go test`
- Appear in godoc documentation
- Show actual usage
- Can't become outdated

### Testing Unexported Functions

**Option 1: Export for testing (export_test.go):**

```go
// calculator.go
package calculator

func add(a, b int) int {
    return a + b
}

// export_test.go
package calculator

var Add = add  // Export for testing

// calculator_test.go
package calculator

func TestAdd(t *testing.T) {
    result := Add(2, 3)  // Use exported version
    // ...
}
```

**Option 2: Test through public API:**

```go
// Better approach - test through public interface
func TestCalculate(t *testing.T) {
    result := Calculate(2, 3)  // Public function that uses add()
    // ...
}
```

### Frequently Asked Questions (Unit Testing)

**Q1: Should I test private/unexported functions?**

**A:**

**Generally no** - test through the public API:

```go
// ❌ AVOID: Testing implementation details
func TestInternalHelper(t *testing.T) {
    result := internalHelper(5)  // Private function
    // ...
}

// ✅ BETTER: Test through public API
func TestPublicFunction(t *testing.T) {
    result := PublicFunction(5)  // Uses internalHelper internally
    // ...
}
```

**Why?**
- Private functions are implementation details
- Testing them creates brittle tests
- Refactoring breaks tests unnecessarily
- Public API is what matters

**Exceptions:**
- Complex internal logic
- Performance-critical algorithms
- Temporary during development

---

**Q2: How many test cases are enough?**

**A:**

Test at least:
1. **Happy path** - normal expected usage
2. **Edge cases** - boundaries, empty values, zero
3. **Error cases** - invalid inputs, failure scenarios
4. **Concurrent usage** - if applicable

```go
func TestUserValidation(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        // Happy path
        {"valid email", "user@example.com", false},
        
        // Edge cases
        {"empty email", "", true},
        {"no @", "userexample.com", true},
        {"no domain", "user@", true},
        
        // Special cases
        {"multiple @", "user@@example.com", true},
        {"unicode", "用户@example.com", false},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if (err != nil) != tt.wantErr {
                t.Errorf("ValidateEmail(%q) error = %v; wantErr %v", 
                    tt.email, err, tt.wantErr)
            }
        })
    }
}
```

---

**Q3: Should I use assertion libraries like testify?**

**A:**

**Pros of standard library:**
- No dependencies
- Simple and explicit
- Official way

**Pros of testify:**
- Less boilerplate
- Better error messages
- Rich assertions

```go
// Standard library
func TestUser(t *testing.T) {
    user := CreateUser("alice")
    if user.Name != "alice" {
        t.Errorf("got %q; want %q", user.Name, "alice")
    }
}

// With testify
func TestUser(t *testing.T) {
    user := CreateUser("alice")
    assert.Equal(t, "alice", user.Name)
}
```

**Recommendation:** Start with standard library. Add testify if needed.

---

### Interview Questions (Unit Testing)

**Question 1: What's wrong with this test?**

```go
func TestCalculate(t *testing.T) {
    result := Calculate(5)
    if result > 0 {
        t.Log("Result is positive")
    }
}
```

**Difficulty:** Junior

**Answer:**

**Problems:**

1. **No assertion** - Test never fails
2. **Unclear expectations** - What should result be?
3. **t.Log in wrong place** - Only shows on failure

**Fixed version:**

```go
func TestCalculate(t *testing.T) {
    result := Calculate(5)
    want := 10
    
    if result != want {
        t.Errorf("Calculate(5) = %d; want %d", result, want)
    }
}
```

**What Interviewers Look For:**
- Understanding that tests must have assertions
- Knowledge of proper error reporting
- Clear test expectations

---

**Question 2: Implement a table-driven test for this function:**

```go
func IsPalindrome(s string) bool {
    // Implementation
}
```

**Difficulty:** Mid-Level

**Answer:**

```go
func TestIsPalindrome(t *testing.T) {
    tests := []struct {
        name  string
        input string
        want  bool
    }{
        {"empty string", "", true},
        {"single char", "a", true},
        {"two same chars", "aa", true},
        {"two different chars", "ab", false},
        {"odd length palindrome", "racecar", true},
        {"even length palindrome", "noon", true},
        {"not palindrome", "hello", false},
        {"with spaces", "race car", false},  // Depends on requirements
        {"uppercase", "Racecar", false},     // Case-sensitive
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := IsPalindrome(tt.input)
            if got != tt.want {
                t.Errorf("IsPalindrome(%q) = %v; want %v", 
                    tt.input, got, tt.want)
            }
        })
    }
}
```

**What Interviewers Look For:**
- Comprehensive test cases
- Edge cases (empty, single character)
- Proper table structure
- Good test names
- Subtests usage

---

### Key Takeaways (Unit Testing)

✅ **Testing is built into Go**
- `testing` package in standard library
- `go test` command runs tests
- Simple, consistent conventions

✅ **Table-driven tests are idiomatic**
- Easy to add test cases
- Clear structure
- Each case has a name
- Can run specific cases

✅ **Use subtests for organization**
- Hierarchical test structure
- Can run specific subtests
- Better test output

✅ **Test helpers reduce boilerplate**
- Mark with t.Helper()
- Make tests more readable
- Centralize common checks

✅ **Test the public API**
- Don't test implementation details
- Test behavior, not internals
- Makes refactoring easier

Continuing with Part 4...
## 4.2 Mocking and Test Doubles

Testing code with dependencies requires isolating units through mocks, stubs, and fakes.

### Dependency Injection

The foundation of testable code:

```go
// ❌ BAD: Hard-coded dependency
type UserService struct{}

func (s *UserService) CreateUser(name string) error {
    db := database.Connect()  // Hard to test!
    return db.Insert(name)
}

// ✅ GOOD: Dependency injection
type UserService struct {
    db Database
}

func NewUserService(db Database) *UserService {
    return &UserService{db: db}
}

func (s *UserService) CreateUser(name string) error {
    return s.db.Insert(name)  // Can inject mock!
}
```

### Interface-Based Mocking

Define interfaces for dependencies:

```go
// Define interface
type Database interface {
    Insert(name string) error
    Get(id int) (string, error)
}

// Production implementation
type PostgresDB struct {
    conn *sql.DB
}

func (db *PostgresDB) Insert(name string) error {
    _, err := db.conn.Exec("INSERT INTO users (name) VALUES ($1)", name)
    return err
}

func (db *PostgresDB) Get(id int) (string, error) {
    var name string
    err := db.conn.QueryRow("SELECT name FROM users WHERE id = $1", id).Scan(&name)
    return name, err
}

// Mock implementation for tests
type MockDatabase struct {
    InsertFunc func(name string) error
    GetFunc    func(id int) (string, error)
}

func (m *MockDatabase) Insert(name string) error {
    if m.InsertFunc != nil {
        return m.InsertFunc(name)
    }
    return nil
}

func (m *MockDatabase) Get(id int) (string, error) {
    if m.GetFunc != nil {
        return m.GetFunc(id)
    }
    return "", nil
}

// Test using mock
func TestUserService_CreateUser(t *testing.T) {
    mock := &MockDatabase{
        InsertFunc: func(name string) error {
            if name == "alice" {
                return nil
            }
            return errors.New("invalid name")
        },
    }
    
    service := NewUserService(mock)
    
    err := service.CreateUser("alice")
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    
    err = service.CreateUser("bob")
    if err == nil {
        t.Error("expected error for bob")
    }
}
```

### Table-Driven Mocks

```go
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name      string
        userName  string
        mockSetup func(*MockDatabase)
        wantErr   bool
    }{
        {
            name:     "successful insert",
            userName: "alice",
            mockSetup: func(m *MockDatabase) {
                m.InsertFunc = func(name string) error {
                    return nil
                }
            },
            wantErr: false,
        },
        {
            name:     "database error",
            userName: "bob",
            mockSetup: func(m *MockDatabase) {
                m.InsertFunc = func(name string) error {
                    return errors.New("db error")
                }
            },
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            mock := &MockDatabase{}
            tt.mockSetup(mock)
            
            service := NewUserService(mock)
            err := service.CreateUser(tt.userName)
            
            if (err != nil) != tt.wantErr {
                t.Errorf("CreateUser() error = %v; wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

### Verification Mocks

Track calls to verify behavior:

```go
type MockDatabase struct {
    InsertCalled bool
    InsertCount  int
    InsertArgs   []string
}

func (m *MockDatabase) Insert(name string) error {
    m.InsertCalled = true
    m.InsertCount++
    m.InsertArgs = append(m.InsertArgs, name)
    return nil
}

func TestUserService_CreateMultipleUsers(t *testing.T) {
    mock := &MockDatabase{}
    service := NewUserService(mock)
    
    service.CreateUser("alice")
    service.CreateUser("bob")
    service.CreateUser("carol")
    
    if !mock.InsertCalled {
        t.Error("Insert was not called")
    }
    
    if mock.InsertCount != 3 {
        t.Errorf("Insert called %d times; want 3", mock.InsertCount)
    }
    
    expected := []string{"alice", "bob", "carol"}
    if !reflect.DeepEqual(mock.InsertArgs, expected) {
        t.Errorf("Insert args = %v; want %v", mock.InsertArgs, expected)
    }
}
```

### Using testify/mock

Popular mocking library:

```go
import (
    "testing"
    "github.com/stretchr/testify/mock"
)

// Mock with testify
type MockDatabase struct {
    mock.Mock
}

func (m *MockDatabase) Insert(name string) error {
    args := m.Called(name)
    return args.Error(0)
}

func (m *MockDatabase) Get(id int) (string, error) {
    args := m.Called(id)
    return args.String(0), args.Error(1)
}

// Test using testify mock
func TestUserService_CreateUser(t *testing.T) {
    mock := new(MockDatabase)
    
    // Set expectations
    mock.On("Insert", "alice").Return(nil)
    mock.On("Insert", "bob").Return(errors.New("db error"))
    
    service := NewUserService(mock)
    
    // Test successful insert
    err := service.CreateUser("alice")
    assert.NoError(t, err)
    
    // Test failed insert
    err = service.CreateUser("bob")
    assert.Error(t, err)
    
    // Verify all expectations were met
    mock.AssertExpectations(t)
}
```

### Using mockgen

Generate mocks automatically:

```go
// database.go
//go:generate mockgen -source=database.go -destination=database_mock.go -package=main

type Database interface {
    Insert(name string) error
    Get(id int) (string, error)
}
```

```bash
# Generate mocks
go generate ./...
```

```go
// Test using generated mock
func TestUserService_CreateUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    mock := NewMockDatabase(ctrl)
    
    // Set expectations
    mock.EXPECT().Insert("alice").Return(nil)
    mock.EXPECT().Insert("bob").Return(errors.New("db error"))
    
    service := NewUserService(mock)
    
    err := service.CreateUser("alice")
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    
    err = service.CreateUser("bob")
    if err == nil {
        t.Error("expected error for bob")
    }
}
```

### Fakes

Simplified implementations for testing:

```go
// Fake in-memory database
type FakeDatabase struct {
    users map[int]string
    mu    sync.RWMutex
    nextID int
}

func NewFakeDatabase() *FakeDatabase {
    return &FakeDatabase{
        users: make(map[int]string),
        nextID: 1,
    }
}

func (db *FakeDatabase) Insert(name string) error {
    db.mu.Lock()
    defer db.mu.Unlock()
    
    db.users[db.nextID] = name
    db.nextID++
    return nil
}

func (db *FakeDatabase) Get(id int) (string, error) {
    db.mu.RLock()
    defer db.mu.RUnlock()
    
    name, ok := db.users[id]
    if !ok {
        return "", errors.New("not found")
    }
    return name, nil
}

// Test using fake
func TestUserService_Integration(t *testing.T) {
    db := NewFakeDatabase()
    service := NewUserService(db)
    
    // Create user
    err := service.CreateUser("alice")
    if err != nil {
        t.Fatalf("failed to create user: %v", err)
    }
    
    // Verify user exists
    name, err := db.Get(1)
    if err != nil {
        t.Fatalf("failed to get user: %v", err)
    }
    
    if name != "alice" {
        t.Errorf("got name %q; want %q", name, "alice")
    }
}
```

### Httptest for HTTP Testing

Testing HTTP handlers and clients:

```go
// Handler to test
func UserHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }
    
    id := r.URL.Query().Get("id")
    if id == "" {
        w.WriteHeader(http.StatusBadRequest)
        json.NewEncoder(w).Encode(map[string]string{
            "error": "id required",
        })
        return
    }
    
    user := User{ID: id, Name: "Alice"}
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(user)
}

// Test handler
func TestUserHandler(t *testing.T) {
    tests := []struct {
        name       string
        method     string
        query      string
        wantStatus int
        wantBody   string
    }{
        {
            name:       "valid request",
            method:     http.MethodGet,
            query:      "?id=1",
            wantStatus: http.StatusOK,
            wantBody:   `{"ID":"1","Name":"Alice"}`,
        },
        {
            name:       "missing id",
            method:     http.MethodGet,
            query:      "",
            wantStatus: http.StatusBadRequest,
        },
        {
            name:       "wrong method",
            method:     http.MethodPost,
            query:      "?id=1",
            wantStatus: http.StatusMethodNotAllowed,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            req := httptest.NewRequest(tt.method, "/user"+tt.query, nil)
            w := httptest.NewRecorder()
            
            UserHandler(w, req)
            
            if w.Code != tt.wantStatus {
                t.Errorf("status = %d; want %d", w.Code, tt.wantStatus)
            }
            
            if tt.wantBody != "" {
                got := strings.TrimSpace(w.Body.String())
                if got != tt.wantBody {
                    t.Errorf("body = %q; want %q", got, tt.wantBody)
                }
            }
        })
    }
}

// Test HTTP client
func TestHTTPClient(t *testing.T) {
    // Create mock server
    server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(User{ID: "1", Name: "Alice"})
    }))
    defer server.Close()
    
    // Test client using mock server
    resp, err := http.Get(server.URL + "/user?id=1")
    if err != nil {
        t.Fatalf("request failed: %v", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        t.Errorf("status = %d; want %d", resp.StatusCode, http.StatusOK)
    }
    
    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        t.Fatalf("failed to decode: %v", err)
    }
    
    if user.Name != "Alice" {
        t.Errorf("name = %q; want %q", user.Name, "Alice")
    }
}
```

### Testing Time-Dependent Code

```go
// ❌ BAD: Hard-coded time
func IsExpired(expiryDate time.Time) bool {
    return time.Now().After(expiryDate)
}

// ✅ GOOD: Inject time provider
type Clock interface {
    Now() time.Time
}

type RealClock struct{}

func (c RealClock) Now() time.Time {
    return time.Now()
}

type FakeClock struct {
    current time.Time
}

func (c *FakeClock) Now() time.Time {
    return c.current
}

func (c *FakeClock) Advance(d time.Duration) {
    c.current = c.current.Add(d)
}

// Use clock interface
func IsExpired(expiryDate time.Time, clock Clock) bool {
    return clock.Now().After(expiryDate)
}

// Test with fake clock
func TestIsExpired(t *testing.T) {
    fakeClock := &FakeClock{
        current: time.Date(2024, 1, 1, 0, 0, 0, 0, time.UTC),
    }
    
    expiryDate := time.Date(2024, 1, 2, 0, 0, 0, 0, time.UTC)
    
    // Not expired yet
    if IsExpired(expiryDate, fakeClock) {
        t.Error("should not be expired")
    }
    
    // Advance time
    fakeClock.Advance(48 * time.Hour)
    
    // Now expired
    if !IsExpired(expiryDate, fakeClock) {
        t.Error("should be expired")
    }
}
```

### Testing Random Behavior

```go
// ❌ BAD: Hard-coded randomness
func GenerateID() string {
    return fmt.Sprintf("%d", rand.Int())
}

// ✅ GOOD: Inject random source
type RandomSource interface {
    Int() int
}

type ProductionRandom struct{}

func (r ProductionRandom) Int() int {
    return rand.Int()
}

type FakeRandom struct {
    values []int
    index  int
}

func (r *FakeRandom) Int() int {
    val := r.values[r.index]
    r.index = (r.index + 1) % len(r.values)
    return val
}

func GenerateID(random RandomSource) string {
    return fmt.Sprintf("%d", random.Int())
}

// Test with predictable values
func TestGenerateID(t *testing.T) {
    fake := &FakeRandom{values: []int{12345, 67890}}
    
    id1 := GenerateID(fake)
    if id1 != "12345" {
        t.Errorf("got %q; want %q", id1, "12345")
    }
    
    id2 := GenerateID(fake)
    if id2 != "67890" {
        t.Errorf("got %q; want %q", id2, "67890")
    }
}
```

### Best Practices for Mocking

```go
// ✅ GOOD: Small, focused interfaces
type UserRepository interface {
    GetUser(id int) (*User, error)
    SaveUser(user *User) error
}

// ❌ BAD: Large, unfocused interface
type Repository interface {
    GetUser(id int) (*User, error)
    SaveUser(user *User) error
    DeleteUser(id int) error
    ListUsers() ([]*User, error)
    GetPost(id int) (*Post, error)
    SavePost(post *Post) error
    // ... many more methods
}

// ✅ GOOD: Define interfaces at point of use
package service

type UserGetter interface {
    GetUser(id int) (*User, error)
}

func ProcessUser(getter UserGetter, id int) error {
    user, err := getter.GetUser(id)
    // ...
}

// ❌ BAD: Define interface with implementation
package repository

type UserRepository interface {
    GetUser(id int) (*User, error)
}

type PostgresUserRepository struct {
    // ...
}

func (r *PostgresUserRepository) GetUser(id int) (*User, error) {
    // ...
}
```

### Frequently Asked Questions (Mocking)

**Q1: Should I use mocks or fakes?**

**A:**

**Mocks:**
- Verify interactions
- Check method calls, arguments
- Fast to set up
- Good for unit tests

**Fakes:**
- Simplified working implementation
- Closer to real behavior
- Better for integration tests
- More maintainable for complex logic

```go
// Mock: Verify behavior
mock.EXPECT().Insert("alice").Return(nil).Times(1)

// Fake: Functional implementation
fake := NewFakeDatabase()  // Works like real DB, in-memory
```

**Recommendation:** Use mocks for unit tests, fakes for integration tests.

---

**Q2: How do I test code that uses global variables?**

**A:**

**Best approach:** Refactor to use dependency injection

```go
// ❌ BAD: Global variable
var DB *sql.DB

func GetUser(id int) (*User, error) {
    // Uses global DB
}

// ✅ GOOD: Dependency injection
type UserService struct {
    db *sql.DB
}

func (s *UserService) GetUser(id int) (*User, error) {
    // Uses injected db
}
```

**If you can't refactor:** Use package-level setup/teardown

```go
var originalDB *sql.DB

func TestMain(m *testing.M) {
    originalDB = DB
    code := m.Run()
    DB = originalDB
    os.Exit(code)
}

func TestGetUser(t *testing.T) {
    DB = testDB  // Replace global
    defer func() { DB = originalDB }()
    
    // Test using testDB
}
```

---

### Key Takeaways (Mocking)

✅ **Dependency injection enables testing**
- Inject dependencies, not hard-code
- Use interfaces for flexibility
- Constructor injection is clearest

✅ **Small interfaces are easier to mock**
- Define at point of use
- Single responsibility
- Easy to implement

✅ **Multiple approaches available**
- Hand-written mocks: simple, explicit
- testify/mock: feature-rich
- mockgen: auto-generated
- Fakes: simplified implementations

✅ **httptest for HTTP testing**
- httptest.NewRecorder for handlers
- httptest.NewServer for clients
- No real network needed

✅ **Make time and randomness testable**
- Inject clock interface
- Inject random source
- Tests become deterministic


## 4.3 Benchmarking

Benchmarking measures code performance and helps identify optimization opportunities.

### Basic Benchmarks

```go
// fibonacci.go
func Fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return Fibonacci(n-1) + Fibonacci(n-2)
}

// fibonacci_test.go
func BenchmarkFibonacci(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Fibonacci(10)
    }
}
```

**Running benchmarks:**

```bash
go test -bench=.                    # Run all benchmarks
go test -bench=BenchmarkFibonacci   # Run specific benchmark
go test -bench=. -benchtime=5s      # Run for 5 seconds
go test -bench=. -count=5           # Run 5 times
go test -bench=. -benchmem          # Include memory stats
```

**Output:**

```
BenchmarkFibonacci-8    3000000    450 ns/op    0 B/op    0 allocs/op
                        │          │            │         │
                        │          │            │         └─ allocations per operation
                        │          │            └─ bytes allocated per operation
                        │          └─ nanoseconds per operation
                        └─ iterations run
```

### Table-Driven Benchmarks

```go
func BenchmarkFibonacci(b *testing.B) {
    benchmarks := []struct {
        name string
        n    int
    }{
        {"Fib10", 10},
        {"Fib20", 20},
        {"Fib30", 30},
    }
    
    for _, bm := range benchmarks {
        b.Run(bm.name, func(b *testing.B) {
            for i := 0; i < b.N; i++ {
                Fibonacci(bm.n)
            }
        })
    }
}
```

### Benchmark Setup and Teardown

```go
func BenchmarkDatabaseQuery(b *testing.B) {
    // Setup (not timed)
    db := setupTestDatabase()
    defer db.Close()
    
    // Reset timer to exclude setup time
    b.ResetTimer()
    
    // Benchmark loop
    for i := 0; i < b.N; i++ {
        db.Query("SELECT * FROM users WHERE id = 1")
    }
    
    // Cleanup (not timed)
    b.StopTimer()
    cleanupDatabase(db)
}
```

### Reporting Allocations

```go
func BenchmarkStringConcatenation(b *testing.B) {
    b.ReportAllocs()  // Report allocation stats
    
    for i := 0; i < b.N; i++ {
        result := ""
        for j := 0; j < 100; j++ {
            result += "a"
        }
    }
}

// Compare different approaches
func BenchmarkStringBuilder(b *testing.B) {
    b.ReportAllocs()
    
    for i := 0; i < b.N; i++ {
        var sb strings.Builder
        for j := 0; j < 100; j++ {
            sb.WriteString("a")
        }
        _ = sb.String()
    }
}
```

**Results:**

```
BenchmarkStringConcatenation-8    50000    30000 ns/op    50000 B/op    99 allocs/op
BenchmarkStringBuilder-8        1000000     1200 ns/op      512 B/op     1 allocs/op
```

### Benchmark Comparison

```bash
# Save baseline
go test -bench=. -benchmem > old.txt

# Make changes to code

# Compare
go test -bench=. -benchmem > new.txt
benchstat old.txt new.txt
```

**Install benchstat:**

```bash
go install golang.org/x/perf/cmd/benchstat@latest
```

**Output:**

```
name                old time/op    new time/op    delta
StringConcatenation   30.0µs ± 2%     1.2µs ± 1%  -96.00%  (p=0.000 n=10+10)

name                old alloc/op   new alloc/op   delta
StringConcatenation   50.0kB ± 0%     0.5kB ± 0%  -99.00%  (p=0.000 n=10+10)

name                old allocs/op  new allocs/op  delta
StringConcatenation    99.0 ± 0%       1.0 ± 0%  -98.99%  (p=0.000 n=10+10)
```

### Parallel Benchmarks

```go
func BenchmarkParallel(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            // Work to benchmark
            DoExpensiveWork()
        }
    })
}

// Compare serial vs parallel
func BenchmarkMapSerial(b *testing.B) {
    m := make(map[int]int)
    for i := 0; i < b.N; i++ {
        m[i] = i
    }
}

func BenchmarkMapParallel(b *testing.B) {
    var mu sync.Mutex
    m := make(map[int]int)
    
    b.RunParallel(func(pb *testing.PB) {
        i := 0
        for pb.Next() {
            mu.Lock()
            m[i] = i
            mu.Unlock()
            i++
        }
    })
}
```

### Custom Metrics

```go
func BenchmarkCustomMetrics(b *testing.B) {
    var totalBytes int64
    
    for i := 0; i < b.N; i++ {
        data := processData()
        totalBytes += int64(len(data))
    }
    
    b.ReportMetric(float64(totalBytes)/float64(b.N), "bytes/op")
    b.ReportMetric(float64(b.N)/b.Elapsed().Seconds(), "ops/sec")
}
```

### Benchmark Examples

**Comparing algorithms:**

```go
func BenchmarkSortAlgorithms(b *testing.B) {
    sizes := []int{10, 100, 1000, 10000}
    
    for _, size := range sizes {
        data := generateRandomSlice(size)
        
        b.Run(fmt.Sprintf("BubbleSort-%d", size), func(b *testing.B) {
            for i := 0; i < b.N; i++ {
                b.StopTimer()
                dataCopy := make([]int, len(data))
                copy(dataCopy, data)
                b.StartTimer()
                
                bubbleSort(dataCopy)
            }
        })
        
        b.Run(fmt.Sprintf("QuickSort-%d", size), func(b *testing.B) {
            for i := 0; i < b.N; i++ {
                b.StopTimer()
                dataCopy := make([]int, len(data))
                copy(dataCopy, data)
                b.StartTimer()
                
                quickSort(dataCopy)
            }
        })
    }
}
```

**Comparing data structures:**

```go
func BenchmarkMapVsSlice(b *testing.B) {
    sizes := []int{10, 100, 1000}
    
    for _, size := range sizes {
        // Map lookup
        b.Run(fmt.Sprintf("Map-%d", size), func(b *testing.B) {
            m := make(map[int]string, size)
            for i := 0; i < size; i++ {
                m[i] = fmt.Sprintf("value%d", i)
            }
            
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                _ = m[size/2]
            }
        })
        
        // Slice linear search
        b.Run(fmt.Sprintf("Slice-%d", size), func(b *testing.B) {
            type kv struct {
                key   int
                value string
            }
            s := make([]kv, size)
            for i := 0; i < size; i++ {
                s[i] = kv{i, fmt.Sprintf("value%d", i)}
            }
            
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                for _, item := range s {
                    if item.key == size/2 {
                        break
                    }
                }
            }
        })
    }
}
```

### Optimization Workflow

```go
// 1. Baseline benchmark
func BenchmarkOriginal(b *testing.B) {
    for i := 0; i < b.N; i++ {
        ProcessData(testData)
    }
}

// 2. Optimize and benchmark
func BenchmarkOptimized(b *testing.B) {
    for i := 0; i < b.N; i++ {
        ProcessDataOptimized(testData)
    }
}

// 3. Compare
// go test -bench=. -benchmem | tee results.txt
// benchstat results.txt

// 4. Profile if needed (see next section)
// go test -bench=BenchmarkOptimized -cpuprofile=cpu.prof
```

### Common Pitfalls

```go
// ❌ WRONG: Work optimized away
func BenchmarkWrong(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Calculate(10)  // Result not used, might be optimized away
    }
}

// ✅ CORRECT: Store result
func BenchmarkCorrect(b *testing.B) {
    var result int
    for i := 0; i < b.N; i++ {
        result = Calculate(10)
    }
    _ = result  // Prevent unused variable error
}

// ❌ WRONG: Setup in loop
func BenchmarkWrong2(b *testing.B) {
    for i := 0; i < b.N; i++ {
        data := make([]int, 1000)  // Allocated every iteration
        process(data)
    }
}

// ✅ CORRECT: Setup before loop
func BenchmarkCorrect2(b *testing.B) {
    data := make([]int, 1000)  // Allocated once
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        process(data)
    }
}
```

### Frequently Asked Questions (Benchmarking)

**Q1: Why does my benchmark show inconsistent results?**

**A:**

**Common causes:**
1. Other processes using CPU
2. Thermal throttling
3. Not enough iterations
4. GC interference

**Solutions:**

```bash
# Run multiple times
go test -bench=. -count=10

# Increase benchmark time
go test -bench=. -benchtime=10s

# Disable GC (if measuring non-GC performance)
go test -bench=. -gcflags='-N -l'

# Use benchstat for statistical analysis
go test -bench=. -count=10 | tee results.txt
benchstat results.txt
```

---

**Q2: How do I know if my optimization worked?**

**A:**

Use benchstat to compare:

```bash
# Before optimization
go test -bench=. -benchmem -count=10 > old.txt

# After optimization  
go test -bench=. -benchmem -count=10 > new.txt

# Compare
benchstat old.txt new.txt
```

**Significant if:**
- p-value < 0.05 (statistically significant)
- Improvement > 10% (worthwhile)
- No regression in other metrics

---

### Key Takeaways (Benchmarking)

✅ **Benchmarks measure performance**
- Use `testing.B` for benchmarks
- Run with `go test -bench=.`
- Report allocations with `-benchmem`

✅ **Proper benchmark structure**
- Setup outside loop
- Use `b.ResetTimer()` after setup
- Store results to prevent optimization

✅ **Compare with benchstat**
- Run multiple times for accuracy
- Statistical comparison
- Identify significant changes

✅ **Parallel benchmarks**
- Use `b.RunParallel()` for concurrent code
- Measure scalability
- Compare serial vs parallel

✅ **Profile for deeper analysis**
- Benchmarks show what, profiling shows why
- CPU, memory, blocking profiles
- Identify hotspots


## 4.4 Profiling and Performance Analysis

Profiling identifies performance bottlenecks by collecting runtime data.

### CPU Profiling

Identifies where CPU time is spent:

```bash
# Generate CPU profile
go test -bench=. -cpuprofile=cpu.prof

# Analyze with pprof
go tool pprof cpu.prof

# Interactive commands in pprof:
# top      - Show top functions by CPU time
# list     - Show source code with annotations
# web      - Generate graph (requires graphviz)
# pdf      - Generate PDF report
```

**In code:**

```go
import (
    "os"
    "runtime/pprof"
)

func main() {
    f, err := os.Create("cpu.prof")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()
    
    if err := pprof.StartCPUProfile(f); err != nil {
        log.Fatal(err)
    }
    defer pprof.StopCPUProfile()
    
    // Code to profile
    doWork()
}
```

**Analyzing profiles:**

```bash
# Top CPU consumers
$ go tool pprof cpu.prof
(pprof) top10

Showing nodes accounting for 2.5s, 95% of 2.63s total
Showing top 10 nodes out of 45
      flat  flat%   sum%        cum   cum%
     1.2s 45.6% 45.6%      1.5s 57.0%  main.processData
     0.5s 19.0% 64.6%      0.8s 30.4%  runtime.mallocgc
     0.3s 11.4% 76.0%      0.3s 11.4%  runtime.memmove
     ...

# List function source with time
(pprof) list processData

Total: 2.63s
ROUTINE ======================== main.processData
     1.2s      1.5s (flat, cum) 57.0% of Total
         .          .      func processData(data []int) {
     100ms      100ms          sort.Ints(data)
     800ms      1.2s           for _, v := range data {
     200ms      200ms              compute(v)
         .          .          }
```

### Memory Profiling

Identifies memory allocation hotspots:

```bash
# Generate memory profile
go test -bench=. -memprofile=mem.prof

# Analyze
go tool pprof mem.prof
```

**In code:**

```go
import (
    "os"
    "runtime"
    "runtime/pprof"
)

func main() {
    // Do work
    doWork()
    
    // Write memory profile
    f, err := os.Create("mem.prof")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()
    
    runtime.GC() // Get up-to-date statistics
    if err := pprof.WriteHeapProfile(f); err != nil {
        log.Fatal(err)
    }
}
```

**Memory profile views:**

```bash
# In-use memory
$ go tool pprof -alloc_space mem.prof
(pprof) top

# Allocated memory
$ go tool pprof -inuse_space mem.prof
(pprof) top

# Allocation count
$ go tool pprof -alloc_objects mem.prof
```

### Blocking Profile

Identifies goroutine blocking on synchronization:

```go
import "runtime"

func main() {
    // Enable blocking profile
    runtime.SetBlockProfileRate(1)
    
    // Run program
    doWork()
    
    // Profile saved via http/pprof or manually
}
```

```bash
# Via pprof HTTP
go tool pprof http://localhost:6060/debug/pprof/block
```

### Mutex Profile

Identifies mutex contention:

```go
import "runtime"

func main() {
    // Enable mutex profiling
    runtime.SetMutexProfileFraction(1)
    
    doWork()
}
```

```bash
go tool pprof http://localhost:6060/debug/pprof/mutex
```

### Live Profiling with net/http/pprof

```go
import _ "net/http/pprof"

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // Your application code
    runApp()
}
```

**Access profiles:**

```bash
# CPU profile (30 seconds)
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Heap profile
go tool pprof http://localhost:6060/debug/pprof/heap

# Goroutine profile
go tool pprof http://localhost:6060/debug/pprof/goroutine

# Block profile
go tool pprof http://localhost:6060/debug/pprof/block

# Mutex profile
go tool pprof http://localhost:6060/debug/pprof/mutex

# View in browser
http://localhost:6060/debug/pprof/
```

### Trace Analysis

Execution traces show program behavior over time:

```go
import (
    "os"
    "runtime/trace"
)

func main() {
    f, err := os.Create("trace.out")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()
    
    if err := trace.Start(f); err != nil {
        log.Fatal(err)
    }
    defer trace.Stop()
    
    doWork()
}
```

**Analyze trace:**

```bash
go tool trace trace.out
```

**Trace viewer shows:**
- Goroutine execution
- GC pauses
- Syscalls
- Network I/O
- Goroutine blocking

### Flame Graphs

Visualize profiling data:

```bash
# Generate flame graph
go tool pprof -http=:8080 cpu.prof

# Or use go-torch (deprecated, use pprof -http)
# go-torch cpu.prof
```

**Flame graph interpretation:**
- Width: Time spent in function
- Height: Call stack depth
- Color: Random (for differentiation)

### Practical Profiling Examples

**Finding allocation hotspots:**

```go
func BenchmarkProcessData(b *testing.B) {
    data := generateTestData()
    b.ReportAllocs()
    
    for i := 0; i < b.N; i++ {
        processData(data)
    }
}
```

```bash
# Profile and find allocations
go test -bench=BenchmarkProcessData -memprofile=mem.prof
go tool pprof mem.prof

(pprof) top
(pprof) list processData
```

**Finding CPU hotspots:**

```go
func TestWithCPUProfile(t *testing.T) {
    f, _ := os.Create("cpu.prof")
    defer f.Close()
    
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // Run expensive operation
    processLargeDataset()
}
```

```bash
go test -run=TestWithCPUProfile
go tool pprof cpu.prof
(pprof) top
(pprof) web  # Visual graph
```

### Optimization Workflow

```
1. Benchmark
   ├─ Identify slow code
   └─ Get baseline numbers

2. Profile
   ├─ CPU profile: Where is time spent?
   ├─ Memory profile: Where are allocations?
   └─ Trace: What's happening over time?

3. Analyze
   ├─ Find hotspots
   ├─ Understand bottlenecks
   └─ Form hypothesis

4. Optimize
   ├─ Make targeted changes
   └─ Keep changes small

5. Verify
   ├─ Re-benchmark
   ├─ Compare with baseline
   └─ Ensure no regressions
```

### Performance Best Practices

```go
// ✅ Pre-allocate slices
func process(n int) []int {
    result := make([]int, 0, n)  // Pre-allocate capacity
    for i := 0; i < n; i++ {
        result = append(result, i)
    }
    return result
}

// ✅ Use strings.Builder for concatenation
func buildString(parts []string) string {
    var sb strings.Builder
    for _, p := range parts {
        sb.WriteString(p)
    }
    return sb.String()
}

// ✅ Use sync.Pool for frequently allocated objects
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processData() {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufferPool.Put(buf)
    }()
    
    // Use buf
}

// ✅ Avoid unnecessary allocations in loops
func sum(data []int) int {
    total := 0
    for _, v := range data {
        total += v  // No allocation
    }
    return total
}

// ❌ Avoid allocations in hot paths
func badSum(data []int) int {
    total := 0
    for _, v := range data {
        temp := v * 2  // Fine, on stack
        result := fmt.Sprintf("%d", temp)  // Allocation!
        total += len(result)
    }
    return total
}
```

### Frequently Asked Questions (Profiling)

**Q1: What's the difference between flat and cum in pprof?**

**A:**

**flat**: Time spent in the function itself
**cum** (cumulative): Time spent in the function + functions it calls

```
      flat  flat%   sum%        cum   cum%
     1.2s 45.6% 45.6%      1.5s 57.0%  main.processData
     
flat = 1.2s: Time in processData directly
cum  = 1.5s: Time in processData + functions it calls
```

**Example:**

```go
func processData() {        // cum time
    sort.Slice()           // not flat time (called function)
    for i := 0; i < n; i++ {  // flat time (this function)
        compute(i)          // not flat time (called function)
    }
}
```

---

**Q2: How do I profile production systems safely?**

**A:**

**Options:**

1. **Continuous profiling (sampling)**

```go
import _ "net/http/pprof"

// Safe: Low overhead, always on
go func() {
    http.ListenAndServe("localhost:6060", nil)
}()
```

2. **On-demand profiling**

```go
// Endpoint to trigger profiling
http.HandleFunc("/admin/profile", func(w http.ResponseWriter, r *http.Request) {
    f, _ := os.Create("cpu.prof")
    defer f.Close()
    
    pprof.StartCPUProfile(f)
    time.Sleep(30 * time.Second)
    pprof.StopCPUProfile()
    
    w.Write([]byte("Profile saved"))
})
```

3. **Use sampling profilers**

```bash
# Google Cloud Profiler
# Datadog Continuous Profiler
# Pyroscope
```

**Best practices:**
- Use sampling (not tracing) in production
- Keep profile duration short (30s-1m)
- Monitor overhead
- Restrict access to profile endpoints

---

### Key Takeaways (Profiling)

✅ **Multiple profile types**
- CPU: Where time is spent
- Memory: Where allocations happen
- Block: Where goroutines block
- Mutex: Where lock contention occurs

✅ **pprof is the main tool**
- `go tool pprof` for analysis
- Interactive and web modes
- Flame graphs for visualization

✅ **Profile before optimizing**
- Don't guess, measure
- Find actual bottlenecks
- Avoid premature optimization

✅ **Live profiling in production**
- net/http/pprof for always-on profiling
- Low overhead sampling
- Secure profile endpoints

✅ **Combine with benchmarks**
- Benchmarks show what's slow
- Profiling shows why it's slow
- Together they guide optimization


## 4.5 Fuzz Testing (Go 1.18+)

Fuzz testing automatically generates inputs to find bugs.

### Basic Fuzz Test

```go
func FuzzReverse(f *testing.F) {
    // Seed corpus (initial inputs)
    f.Add("hello")
    f.Add("world")
    f.Add("")
    
    f.Fuzz(func(t *testing.T, s string) {
        // Test that reversing twice gives original
        rev1 := Reverse(s)
        rev2 := Reverse(rev1)
        
        if s != rev2 {
            t.Errorf("Reverse twice: got %q, want %q", rev2, s)
        }
    })
}

func Reverse(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}
```

**Running fuzz tests:**

```bash
go test -fuzz=FuzzReverse                # Run until failure or Ctrl+C
go test -fuzz=FuzzReverse -fuzztime=30s  # Run for 30 seconds
go test -fuzz=. -fuzztime=1000x          # Run 1000 iterations
```

### Fuzz Test Patterns

**Testing for panics:**

```go
func FuzzParse(f *testing.F) {
    f.Add("123")
    f.Add("-456")
    
    f.Fuzz(func(t *testing.T, input string) {
        // Should not panic on any input
        _, err := ParseInt(input)
        // Check error is reasonable
        if err != nil && input != "" {
            t.Logf("Input %q produced error: %v", input, err)
        }
    })
}
```

**Testing properties:**

```go
func FuzzEncodeDecode(f *testing.F) {
    f.Add([]byte("test data"))
    
    f.Fuzz(func(t *testing.T, data []byte) {
        // Encode then decode should give original
        encoded := Encode(data)
        decoded, err := Decode(encoded)
        
        if err != nil {
            t.Fatalf("Decode failed: %v", err)
        }
        
        if !bytes.Equal(data, decoded) {
            t.Errorf("Round trip failed: got %v, want %v", decoded, data)
        }
    })
}
```

**Multiple parameters:**

```go
func FuzzDivide(f *testing.F) {
    f.Add(10, 2)
    f.Add(100, 3)
    
    f.Fuzz(func(t *testing.T, a, b int) {
        if b == 0 {
            t.Skip("division by zero")
        }
        
        result := a / b
        
        // Verify result is reasonable
        if b > 0 && result > a {
            t.Errorf("Division result %d > dividend %d", result, a)
        }
    })
}
```

### Corpus Management

```
testdata/fuzz/FuzzReverse/
├── seed_corpus/          # Hand-crafted inputs
│   ├── input1
│   └── input2
└── corpus/               # Automatically generated
    ├── 0a1b2c3d...
    └── 4e5f6g7h...
```

**Adding to corpus:**

```go
f.Add("edge case")  // Added to seed corpus
```

**Minimizing corpus:**

```bash
go test -fuzz=FuzzReverse -fuzzminimizetime=30s
```

## 4.6 Code Coverage

Measuring test coverage helps identify untested code.

### Basic Coverage

```bash
# Run tests with coverage
go test -cover

# Generate coverage profile
go test -coverprofile=coverage.out

# View coverage in browser
go tool cover -html=coverage.out

# Coverage by function
go tool cover -func=coverage.out
```

**Output:**

```
main.go:10:     Add             100.0%
main.go:14:     Divide          85.7%
main.go:22:     Multiply        100.0%
total:          (statements)    92.3%
```

### Coverage Modes

```bash
# Statement coverage (default)
go test -cover -covermode=set

# Count coverage (how many times executed)
go test -cover -covermode=count

# Atomic coverage (thread-safe count)
go test -cover -covermode=atomic
```

### Package Coverage

```bash
# All packages
go test ./... -cover

# With coverage profile
go test ./... -coverprofile=coverage.out
go tool cover -func=coverage.out

# By package
go test -coverpkg=./... ./...
```

### Coverage Requirements

```bash
# Fail if coverage below threshold
go test -cover | grep -E 'coverage: [0-9]+' | \
    awk '{if ($2 < 80) exit 1}'
```

**In CI/CD:**

```yaml
# .github/workflows/test.yml
- name: Test with coverage
  run: go test -coverprofile=coverage.out ./...

- name: Check coverage
  run: |
    coverage=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
    if (( $(echo "$coverage < 80" | bc -l) )); then
      echo "Coverage $coverage% is below 80%"
      exit 1
    fi
```

### What to Cover

```go
// ✅ Test happy path
func TestAdd(t *testing.T) {
    if Add(2, 3) != 5 {
        t.Error("2 + 3 should be 5")
    }
}

// ✅ Test error cases
func TestDivideByZero(t *testing.T) {
    _, err := Divide(10, 0)
    if err == nil {
        t.Error("expected error for division by zero")
    }
}

// ✅ Test edge cases
func TestEdgeCases(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {0, 0, 0},
        {-1, 1, 0},
        {math.MaxInt, 1, math.MaxInt},
    }
    
    for _, tt := range tests {
        if got := Add(tt.a, tt.b); got != tt.want {
            t.Errorf("Add(%d, %d) = %d; want %d", 
                tt.a, tt.b, got, tt.want)
        }
    }
}
```

### Coverage Limitations

```go
// ❌ 100% coverage doesn't mean bug-free
func Divide(a, b int) int {
    if b == 0 {
        return 0  // Wrong! Should error
    }
    return a / b
}

func TestDivide(t *testing.T) {
    // Covers all lines but doesn't catch bug
    result := Divide(10, 2)
    if result != 5 {
        t.Error("10 / 2 should be 5")
    }
    
    result = Divide(10, 0)  // Covers error path but doesn't verify behavior
}
```

**Coverage shows what's tested, not if it's tested correctly.**

## 4.7 Integration Testing

Testing complete system components together.

### Test Containers

```go
import (
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/wait"
)

func TestWithPostgres(t *testing.T) {
    ctx := context.Background()
    
    // Start PostgreSQL container
    req := testcontainers.ContainerRequest{
        Image:        "postgres:14",
        ExposedPorts: []string{"5432/tcp"},
        Env: map[string]string{
            "POSTGRES_PASSWORD": "password",
            "POSTGRES_DB":       "testdb",
        },
        WaitingFor: wait.ForLog("database system is ready to accept connections"),
    }
    
    container, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: req,
        Started:          true,
    })
    if err != nil {
        t.Fatal(err)
    }
    defer container.Terminate(ctx)
    
    // Get connection details
    host, _ := container.Host(ctx)
    port, _ := container.MappedPort(ctx, "5432")
    
    // Connect and test
    db, err := sql.Open("postgres", fmt.Sprintf(
        "host=%s port=%s user=postgres password=password dbname=testdb sslmode=disable",
        host, port.Port()))
    if err != nil {
        t.Fatal(err)
    }
    defer db.Close()
    
    // Run integration tests
    testRepository(t, db)
}
```

### HTTP Integration Tests

```go
func TestAPIIntegration(t *testing.T) {
    // Start test server
    srv := startTestServer()
    defer srv.Close()
    
    tests := []struct {
        name       string
        method     string
        path       string
        body       string
        wantStatus int
        wantBody   string
    }{
        {
            name:       "create user",
            method:     "POST",
            path:       "/users",
            body:       `{"name":"alice","email":"alice@example.com"}`,
            wantStatus: http.StatusCreated,
        },
        {
            name:       "get user",
            method:     "GET",
            path:       "/users/1",
            wantStatus: http.StatusOK,
            wantBody:   `{"id":1,"name":"alice"}`,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            var body io.Reader
            if tt.body != "" {
                body = strings.NewReader(tt.body)
            }
            
            req, _ := http.NewRequest(tt.method, srv.URL+tt.path, body)
            resp, err := http.DefaultClient.Do(req)
            if err != nil {
                t.Fatal(err)
            }
            defer resp.Body.Close()
            
            if resp.StatusCode != tt.wantStatus {
                t.Errorf("status = %d; want %d", resp.StatusCode, tt.wantStatus)
            }
            
            if tt.wantBody != "" {
                gotBody, _ := io.ReadAll(resp.Body)
                if !strings.Contains(string(gotBody), tt.wantBody) {
                    t.Errorf("body = %q; want to contain %q", gotBody, tt.wantBody)
                }
            }
        })
    }
}
```

### Database Integration Tests

```go
func TestUserRepository(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }
    
    db := setupTestDB(t)
    defer cleanupTestDB(t, db)
    
    repo := NewUserRepository(db)
    
    t.Run("Create and Get", func(t *testing.T) {
        user := &User{Name: "alice", Email: "alice@example.com"}
        
        err := repo.Create(user)
        if err != nil {
            t.Fatalf("failed to create user: %v", err)
        }
        
        retrieved, err := repo.Get(user.ID)
        if err != nil {
            t.Fatalf("failed to get user: %v", err)
        }
        
        if retrieved.Name != user.Name {
            t.Errorf("name = %q; want %q", retrieved.Name, user.Name)
        }
    })
}

func setupTestDB(t *testing.T) *sql.DB {
    db, err := sql.Open("postgres", "postgres://localhost/testdb")
    if err != nil {
        t.Fatalf("failed to connect: %v", err)
    }
    
    // Run migrations
    if err := runMigrations(db); err != nil {
        t.Fatalf("failed to migrate: %v", err)
    }
    
    return db
}

func cleanupTestDB(t *testing.T, db *sql.DB) {
    // Clean up test data
    _, err := db.Exec("TRUNCATE users CASCADE")
    if err != nil {
        t.Errorf("cleanup failed: %v", err)
    }
    db.Close()
}
```

## 4.8 Static Analysis and Code Quality

Automated tools for maintaining code quality.

### go vet

Built-in static analyzer:

```bash
go vet ./...
```

**Checks for:**
- Printf format errors
- Unreachable code
- Misuse of sync types
- Shadow variables
- Struct tag formatting

### staticcheck

Comprehensive static analysis:

```bash
# Install
go install honnef.co/go/tools/cmd/staticcheck@latest

# Run
staticcheck ./...
```

**Example checks:**
- Unused code
- Deprecated APIs
- Performance issues
- Potential bugs
- Style violations

### golangci-lint

Meta-linter running multiple linters:

```bash
# Install
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Run
golangci-lint run

# With config
golangci-lint run --config .golangci.yml
```

**Configuration (.golangci.yml):**

```yaml
linters:
  enable:
    - errcheck      # Check error handling
    - gosimple      # Simplify code
    - govet         # Go vet
    - ineffassign   # Detect ineffectual assignments
    - staticcheck   # Staticcheck
    - unused        # Detect unused code
    - gofmt         # Check formatting
    - goimports     # Check imports
    - misspell      # Check spelling

linters-settings:
  errcheck:
    check-blank: true
  gocyclo:
    min-complexity: 15

issues:
  exclude-use-default: false
```

### Code Formatting

```bash
# Format code
go fmt ./...
gofmt -w .

# Check formatting
gofmt -l .

# Organize imports
goimports -w .
```

### Documentation

```bash
# Generate and view docs
godoc -http=:6060

# Check documentation
go doc fmt.Println
go doc -all encoding/json
```

**Documentation comments:**

```go
// Package calculator provides basic arithmetic operations.
package calculator

// Add returns the sum of a and b.
func Add(a, b int) int {
    return a + b
}

// Divide divides a by b.
// It returns an error if b is zero.
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

## Part 4 Summary

### What You've Learned

**Unit Testing:**
- Table-driven tests
- Subtests and test organization
- Test helpers and setup/teardown
- Parallel testing
- Example tests

**Mocking:**
- Dependency injection
- Interface-based mocks
- Hand-written and generated mocks
- Fakes for integration tests
- HTTP testing with httptest

**Benchmarking:**
- Writing benchmarks
- Table-driven benchmarks
- Measuring allocations
- Comparing performance
- Optimization workflow

**Profiling:**
- CPU, memory, blocking, mutex profiles
- pprof analysis
- Live profiling with net/http/pprof
- Trace analysis
- Performance optimization

**Fuzz Testing:**
- Property-based testing
- Automatic input generation
- Corpus management
- Finding edge cases

**Code Coverage:**
- Measuring coverage
- Coverage profiles
- Setting coverage targets
- Understanding limitations

**Integration Testing:**
- Test containers
- HTTP integration tests
- Database testing
- End-to-end workflows

**Code Quality:**
- Static analysis (vet, staticcheck)
- Linting (golangci-lint)
- Code formatting
- Documentation

### Best Practices Recap

✅ **Test pyramid:** Many unit tests, some integration tests, few E2E tests
✅ **Test behavior, not implementation:** Focus on public API
✅ **Use table-driven tests:** Easy to add cases, clear structure
✅ **Mock at boundaries:** Database, HTTP, external services
✅ **Profile before optimizing:** Measure, don't guess
✅ **Automate quality checks:** CI/CD with tests, linters, coverage
✅ **Write testable code:** Dependency injection, small functions
✅ **Document public APIs:** Clear, concise godoc comments

### What's Next

In **Part 5: Production Architecture Patterns**, we'll explore:

- Clean architecture in Go
- Domain-driven design
- Hexagonal architecture
- CQRS and Event Sourcing
- Microservices patterns
- Database patterns
- Caching strategies
- Message queues
- API design
- Configuration management

### Practice Exercises

1. **Write comprehensive tests** for a REST API with >80% coverage
2. **Profile and optimize** a slow function using pprof
3. **Create integration tests** using test containers
4. **Set up CI/CD pipeline** with tests, linters, and coverage checks
5. **Write fuzz tests** for a parser or serializer

---

**Ready to build production-ready Go architectures?** Continue to Part 5 where we'll explore patterns for building scalable, maintainable systems!


## Advanced Testing Patterns

### Golden Files

Testing against known-good output:

```go
// testdata/golden/output.json.golden
func TestRenderOutput(t *testing.T) {
    tests := []struct {
        name       string
        input      Input
        goldenFile string
    }{
        {
            name:       "basic output",
            input:      Input{Name: "test"},
            goldenFile: "output.json.golden",
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := RenderOutput(tt.input)
            
            goldenPath := filepath.Join("testdata", "golden", tt.goldenFile)
            
            // Update golden files with -update flag
            if *update {
                err := os.WriteFile(goldenPath, got, 0644)
                if err != nil {
                    t.Fatal(err)
                }
            }
            
            want, err := os.ReadFile(goldenPath)
            if err != nil {
                t.Fatal(err)
            }
            
            if !bytes.Equal(got, want) {
                t.Errorf("Output mismatch:\ngot:\n%s\nwant:\n%s", got, want)
            }
        })
    }
}

var update = flag.Bool("update", false, "update golden files")
```

**Usage:**

```bash
# Run tests
go test

# Update golden files
go test -update
```

### Snapshot Testing

```go
type Snapshot struct {
    mu        sync.Mutex
    snapshots map[string]string
    updated   bool
}

func NewSnapshot(t *testing.T) *Snapshot {
    s := &Snapshot{
        snapshots: make(map[string]string),
    }
    
    // Load existing snapshots
    data, err := os.ReadFile("snapshots.json")
    if err == nil {
        json.Unmarshal(data, &s.snapshots)
    }
    
    // Save on test completion
    t.Cleanup(func() {
        if s.updated {
            data, _ := json.MarshalIndent(s.snapshots, "", "  ")
            os.WriteFile("snapshots.json", data, 0644)
        }
    })
    
    return s
}

func (s *Snapshot) Match(t *testing.T, name string, got interface{}) {
    t.Helper()
    
    gotJSON, _ := json.MarshalIndent(got, "", "  ")
    gotStr := string(gotJSON)
    
    s.mu.Lock()
    defer s.mu.Unlock()
    
    want, exists := s.snapshots[name]
    
    if !exists || *update {
        s.snapshots[name] = gotStr
        s.updated = true
        return
    }
    
    if gotStr != want {
        t.Errorf("Snapshot mismatch for %s:\ngot:\n%s\nwant:\n%s", 
            name, gotStr, want)
    }
}

// Usage
func TestUserSerialization(t *testing.T) {
    snap := NewSnapshot(t)
    
    user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}
    snap.Match(t, "user_basic", user)
}
```

### Test Fixtures

```go
// fixtures.go
package testutil

type Fixtures struct {
    DB    *sql.DB
    Users []*User
}

func NewFixtures(t *testing.T) *Fixtures {
    t.Helper()
    
    db := setupTestDB(t)
    
    fixtures := &Fixtures{
        DB: db,
    }
    
    // Create test users
    fixtures.Users = []*User{
        {ID: 1, Name: "Alice", Email: "alice@example.com"},
        {ID: 2, Name: "Bob", Email: "bob@example.com"},
    }
    
    for _, user := range fixtures.Users {
        _, err := db.Exec(
            "INSERT INTO users (id, name, email) VALUES ($1, $2, $3)",
            user.ID, user.Name, user.Email,
        )
        if err != nil {
            t.Fatal(err)
        }
    }
    
    t.Cleanup(func() {
        db.Exec("TRUNCATE users")
        db.Close()
    })
    
    return fixtures
}

// Usage
func TestUserRepository(t *testing.T) {
    fixtures := NewFixtures(t)
    repo := NewUserRepository(fixtures.DB)
    
    user, err := repo.Get(fixtures.Users[0].ID)
    if err != nil {
        t.Fatal(err)
    }
    
    if user.Name != fixtures.Users[0].Name {
        t.Errorf("got %q; want %q", user.Name, fixtures.Users[0].Name)
    }
}
```

### Test Builders

```go
// builder.go
type UserBuilder struct {
    user User
}

func NewUserBuilder() *UserBuilder {
    return &UserBuilder{
        user: User{
            ID:    1,
            Name:  "Test User",
            Email: "test@example.com",
            Age:   25,
        },
    }
}

func (b *UserBuilder) WithID(id int) *UserBuilder {
    b.user.ID = id
    return b
}

func (b *UserBuilder) WithName(name string) *UserBuilder {
    b.user.Name = name
    return b
}

func (b *UserBuilder) WithEmail(email string) *UserBuilder {
    b.user.Email = email
    return b
}

func (b *UserBuilder) Build() User {
    return b.user
}

// Usage in tests
func TestUserValidation(t *testing.T) {
    tests := []struct {
        name    string
        user    User
        wantErr bool
    }{
        {
            name: "valid user",
            user: NewUserBuilder().
                WithName("Alice").
                WithEmail("alice@example.com").
                Build(),
            wantErr: false,
        },
        {
            name: "invalid email",
            user: NewUserBuilder().
                WithEmail("invalid").
                Build(),
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateUser(tt.user)
            if (err != nil) != tt.wantErr {
                t.Errorf("ValidateUser() error = %v; wantErr %v", 
                    err, tt.wantErr)
            }
        })
    }
}
```

### Testing Concurrency

```go
func TestConcurrentAccess(t *testing.T) {
    cache := NewCache()
    
    var wg sync.WaitGroup
    goroutines := 100
    operations := 1000
    
    // Concurrent writers
    for i := 0; i < goroutines; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for j := 0; j < operations; j++ {
                key := fmt.Sprintf("key-%d-%d", id, j)
                cache.Set(key, j)
            }
        }(i)
    }
    
    // Concurrent readers
    for i := 0; i < goroutines; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for j := 0; j < operations; j++ {
                key := fmt.Sprintf("key-%d-%d", id, j)
                cache.Get(key)
            }
        }(i)
    }
    
    wg.Wait()
    
    // Verify state
    expectedKeys := goroutines * operations
    if cache.Len() != expectedKeys {
        t.Errorf("cache size = %d; want %d", cache.Len(), expectedKeys)
    }
}

// Test with race detector
// go test -race
```

### Deterministic Concurrency Testing

```go
func TestRaceCondition(t *testing.T) {
    for i := 0; i < 1000; i++ {  // Run many times
        t.Run(fmt.Sprintf("iteration-%d", i), func(t *testing.T) {
            t.Parallel()
            
            counter := 0
            var wg sync.WaitGroup
            
            for j := 0; j < 100; j++ {
                wg.Add(1)
                go func() {
                    defer wg.Done()
                    counter++  // Race!
                }()
            }
            
            wg.Wait()
            
            // This will fail sometimes due to race
            if counter != 100 {
                t.Errorf("counter = %d; want 100 (race detected)", counter)
            }
        })
    }
}

// Run with:
// go test -race -count=100
```

### Testing Timeouts

```go
func TestWithTimeout(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping test in short mode")
    }
    
    timeout := time.After(5 * time.Second)
    done := make(chan bool)
    
    go func() {
        result := LongRunningOperation()
        if result != "expected" {
            t.Error("unexpected result")
        }
        done <- true
    }()
    
    select {
    case <-done:
        // Success
    case <-timeout:
        t.Fatal("test timed out")
    }
}
```

### Custom Test Assertions

```go
package assert

import (
    "reflect"
    "testing"
)

func Equal(t *testing.T, got, want interface{}) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("\ngot:  %+v\nwant: %+v", got, want)
    }
}

func NoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

func Error(t *testing.T, err error, msg string) {
    t.Helper()
    if err == nil {
        t.Fatal("expected error but got nil")
    }
    if err.Error() != msg {
        t.Errorf("error message = %q; want %q", err.Error(), msg)
    }
}

func Contains(t *testing.T, haystack, needle string) {
    t.Helper()
    if !strings.Contains(haystack, needle) {
        t.Errorf("%q does not contain %q", haystack, needle)
    }
}

func Len(t *testing.T, slice interface{}, want int) {
    t.Helper()
    v := reflect.ValueOf(slice)
    if v.Len() != want {
        t.Errorf("length = %d; want %d", v.Len(), want)
    }
}

// Usage
func TestAssertions(t *testing.T) {
    user, err := GetUser(1)
    assert.NoError(t, err)
    assert.Equal(t, user.Name, "Alice")
    
    users := GetAllUsers()
    assert.Len(t, users, 10)
}
```

### Comprehensive Interview Questions

**Question 1: How would you test a function that uses time.Now()?**

**Difficulty:** Mid-Level

**Answer:**

**Problem:** `time.Now()` makes tests non-deterministic.

**Solution:** Inject a clock interface:

```go
// Define clock interface
type Clock interface {
    Now() time.Time
}

// Real clock
type RealClock struct{}

func (c RealClock) Now() time.Time {
    return time.Now()
}

// Fake clock for testing
type FakeClock struct {
    current time.Time
}

func (c *FakeClock) Now() time.Time {
    return c.current
}

func (c *FakeClock) Set(t time.Time) {
    c.current = t
}

// Function using clock
type Session struct {
    clock     Clock
    expiresAt time.Time
}

func NewSession(clock Clock, duration time.Duration) *Session {
    return &Session{
        clock:     clock,
        expiresAt: clock.Now().Add(duration),
    }
}

func (s *Session) IsExpired() bool {
    return s.clock.Now().After(s.expiresAt)
}

// Test
func TestSession(t *testing.T) {
    fakeClock := &FakeClock{
        current: time.Date(2024, 1, 1, 0, 0, 0, 0, time.UTC),
    }
    
    session := NewSession(fakeClock, time.Hour)
    
    // Not expired yet
    if session.IsExpired() {
        t.Error("session should not be expired")
    }
    
    // Advance time
    fakeClock.Set(time.Date(2024, 1, 1, 2, 0, 0, 0, time.UTC))
    
    // Now expired
    if !session.IsExpired() {
        t.Error("session should be expired")
    }
}
```

**What Interviewers Look For:**
- Understanding of dependency injection
- Knowledge of interface-based design
- Ability to make code testable
- Clean separation of concerns

---

**Question 2: How do you test code that makes HTTP requests?**

**Difficulty:** Mid-Level

**Answer:**

**Multiple approaches:**

**1. httptest.NewServer (for testing clients):**

```go
func TestHTTPClient(t *testing.T) {
    // Create mock server
    server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if r.URL.Path != "/api/users" {
            w.WriteHeader(http.StatusNotFound)
            return
        }
        
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(map[string]string{
            "name": "Alice",
        })
    }))
    defer server.Close()
    
    // Test client using mock server
    client := NewAPIClient(server.URL)
    user, err := client.GetUser(1)
    
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    
    if user.Name != "Alice" {
        t.Errorf("name = %q; want %q", user.Name, "Alice")
    }
}
```

**2. httptest.NewRecorder (for testing handlers):**

```go
func TestHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "/users/1", nil)
    w := httptest.NewRecorder()
    
    UserHandler(w, req)
    
    if w.Code != http.StatusOK {
        t.Errorf("status = %d; want %d", w.Code, http.StatusOK)
    }
    
    var user User
    if err := json.NewDecoder(w.Body).Decode(&user); err != nil {
        t.Fatalf("failed to decode: %v", err)
    }
}
```

**3. Inject HTTP client:**

```go
type HTTPClient interface {
    Do(*http.Request) (*http.Response, error)
}

type APIClient struct {
    client HTTPClient
    baseURL string
}

func (c *APIClient) GetUser(id int) (*User, error) {
    req, _ := http.NewRequest("GET", 
        fmt.Sprintf("%s/users/%d", c.baseURL, id), nil)
    
    resp, err := c.client.Do(req)
    // ... handle response
}

// Mock HTTP client
type MockHTTPClient struct {
    DoFunc func(*http.Request) (*http.Response, error)
}

func (m *MockHTTPClient) Do(req *http.Request) (*http.Response, error) {
    return m.DoFunc(req)
}

// Test
func TestAPIClient(t *testing.T) {
    mock := &MockHTTPClient{
        DoFunc: func(req *http.Request) (*http.Response, error) {
            body := `{"name":"Alice"}`
            return &http.Response{
                StatusCode: 200,
                Body:       io.NopCloser(strings.NewReader(body)),
            }, nil
        },
    }
    
    client := &APIClient{client: mock, baseURL: "http://api.example.com"}
    user, err := client.GetUser(1)
    
    if err != nil {
        t.Fatal(err)
    }
    if user.Name != "Alice" {
        t.Errorf("name = %q; want %q", user.Name, "Alice")
    }
}
```

---

**Question 3: How would you test a function with database dependencies?**

**Difficulty:** Senior

**Answer:**

**Multiple approaches:**

**1. Use interface abstraction:**

```go
type UserRepository interface {
    Create(user *User) error
    Get(id int) (*User, error)
}

type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) Create(user *User) error {
    _, err := r.db.Exec(
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        user.Name, user.Email,
    )
    return err
}

// Service depends on interface
type UserService struct {
    repo UserRepository
}

func (s *UserService) RegisterUser(name, email string) error {
    user := &User{Name: name, Email: email}
    return s.repo.Create(user)
}

// Mock for testing
type MockUserRepository struct {
    CreateFunc func(*User) error
}

func (m *MockUserRepository) Create(user *User) error {
    return m.CreateFunc(user)
}

func (m *MockUserRepository) Get(id int) (*User, error) {
    return nil, nil
}

// Test
func TestUserService(t *testing.T) {
    mock := &MockUserRepository{
        CreateFunc: func(user *User) error {
            if user.Email == "" {
                return errors.New("email required")
            }
            return nil
        },
    }
    
    service := &UserService{repo: mock}
    
    err := service.RegisterUser("Alice", "alice@example.com")
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    
    err = service.RegisterUser("Bob", "")
    if err == nil {
        t.Error("expected error for empty email")
    }
}
```

**2. Use test database:**

```go
func TestUserRepository_Integration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }
    
    db := setupTestDB(t)
    defer teardownTestDB(t, db)
    
    repo := &PostgresUserRepository{db: db}
    
    user := &User{Name: "Alice", Email: "alice@example.com"}
    err := repo.Create(user)
    if err != nil {
        t.Fatalf("failed to create user: %v", err)
    }
    
    retrieved, err := repo.Get(user.ID)
    if err != nil {
        t.Fatalf("failed to get user: %v", err)
    }
    
    if retrieved.Name != user.Name {
        t.Errorf("name = %q; want %q", retrieved.Name, user.Name)
    }
}

func setupTestDB(t *testing.T) *sql.DB {
    db, err := sql.Open("postgres", "postgres://localhost/testdb")
    if err != nil {
        t.Fatalf("failed to connect: %v", err)
    }
    
    // Run migrations
    runMigrations(db)
    
    return db
}

func teardownTestDB(t *testing.T, db *sql.DB) {
    db.Exec("DROP TABLE IF EXISTS users")
    db.Close()
}
```

**3. Use sqlmock for unit tests:**

```go
import "github.com/DATA-DOG/go-sqlmock"

func TestUserRepository_Create(t *testing.T) {
    db, mock, err := sqlmock.New()
    if err != nil {
        t.Fatal(err)
    }
    defer db.Close()
    
    repo := &PostgresUserRepository{db: db}
    
    // Set expectations
    mock.ExpectExec("INSERT INTO users").
        WithArgs("Alice", "alice@example.com").
        WillReturnResult(sqlmock.NewResult(1, 1))
    
    // Test
    user := &User{Name: "Alice", Email: "alice@example.com"}
    err = repo.Create(user)
    
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    
    // Verify expectations
    if err := mock.ExpectationsWereMet(); err != nil {
        t.Errorf("unfulfilled expectations: %v", err)
    }
}
```

**What Interviewers Look For:**
- Multiple testing strategies
- Understanding trade-offs (unit vs integration)
- Clean architecture (dependency injection)
- Proper use of mocks vs real databases

---

### Testing Anti-Patterns to Avoid

```go
// ❌ BAD: Testing implementation details
func TestUserInternal(t *testing.T) {
    u := &User{}
    u.internalField = "value"  // Testing private state
    // Breaks when refactoring
}

// ✅ GOOD: Test public behavior
func TestUserBehavior(t *testing.T) {
    u := NewUser("Alice")
    if u.Name() != "Alice" {
        t.Error("name should be Alice")
    }
}

// ❌ BAD: Tests depending on each other
var globalUser *User

func TestCreateUser(t *testing.T) {
    globalUser = CreateUser("Alice")
}

func TestGetUser(t *testing.T) {
    // Depends on TestCreateUser running first!
    if globalUser.Name != "Alice" {
        t.Error("wrong name")
    }
}

// ✅ GOOD: Independent tests
func TestCreateUser(t *testing.T) {
    user := CreateUser("Alice")
    if user == nil {
        t.Error("user should not be nil")
    }
}

func TestGetUser(t *testing.T) {
    // Setup own state
    user := CreateUser("Bob")
    retrieved := GetUser(user.ID)
    if retrieved.Name != "Bob" {
        t.Error("wrong name")
    }
}

// ❌ BAD: Fragile tests
func TestUserJSON(t *testing.T) {
    user := User{Name: "Alice", Age: 30}
    json, _ := json.Marshal(user)
    
    // Fragile: depends on exact JSON format
    if string(json) != `{"name":"Alice","age":30}` {
        t.Error("JSON doesn't match")
    }
}

// ✅ GOOD: Test behavior, not format
func TestUserJSON(t *testing.T) {
    user := User{Name: "Alice", Age: 30}
    json, _ := json.Marshal(user)
    
    var decoded User
    json.Unmarshal(json, &decoded)
    
    if decoded.Name != user.Name {
        t.Error("name not preserved")
    }
}
```


## Real-World Testing Strategies

### Microservices Testing Strategy

```go
// 1. Unit tests for business logic
func TestOrderService_CalculateTotal(t *testing.T) {
    service := NewOrderService(nil, nil) // No dependencies needed
    
    order := &Order{
        Items: []Item{
            {Price: 10.00, Quantity: 2},
            {Price: 5.00, Quantity: 3},
        },
    }
    
    total := service.CalculateTotal(order)
    if total != 35.00 {
        t.Errorf("total = %.2f; want 35.00", total)
    }
}

// 2. Integration tests for external dependencies
func TestOrderService_CreateOrder_Integration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }
    
    // Start dependencies (database, message queue)
    db := startTestDatabase(t)
    queue := startTestQueue(t)
    
    service := NewOrderService(db, queue)
    
    order, err := service.CreateOrder(testOrder)
    if err != nil {
        t.Fatal(err)
    }
    
    // Verify order in database
    retrieved, _ := db.GetOrder(order.ID)
    if retrieved.Total != order.Total {
        t.Errorf("total = %.2f; want %.2f", retrieved.Total, order.Total)
    }
    
    // Verify message published
    msg := <-queue.Messages()
    if msg.OrderID != order.ID {
        t.Error("message not published")
    }
}

// 3. Contract tests for API
func TestOrderAPI_Contract(t *testing.T) {
    server := startTestServer()
    defer server.Close()
    
    // Test API contract
    resp, err := http.Post(
        server.URL+"/orders",
        "application/json",
        strings.NewReader(`{"items":[{"price":10,"quantity":2}]}`),
    )
    if err != nil {
        t.Fatal(err)
    }
    defer resp.Body.Close()
    
    // Verify response structure
    var order Order
    if err := json.NewDecoder(resp.Body).Decode(&order); err != nil {
        t.Fatal("invalid response format")
    }
    
    if order.ID == 0 {
        t.Error("order ID should be set")
    }
}

// 4. End-to-end tests
func TestOrderFlow_E2E(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping E2E test")
    }
    
    // Start entire system
    system := startTestSystem(t)
    defer system.Shutdown()
    
    // Create order via API
    order := createOrderViaAPI(t, system)
    
    // Wait for processing
    waitForOrderProcessed(t, system, order.ID, 5*time.Second)
    
    // Verify order completed
    status := getOrderStatus(t, system, order.ID)
    if status != "completed" {
        t.Errorf("status = %s; want completed", status)
    }
}
```

### CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v2
      
      - uses: actions/setup-go@v2
        with:
          go-version: '1.24'
      
      - name: Install dependencies
        run: go mod download
      
      - name: Run unit tests
        run: go test -short -race -coverprofile=coverage.txt ./...
      
      - name: Run integration tests
        run: go test -race ./...
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/testdb
      
      - name: Upload coverage
        uses: codecov/codecov-action@v2
        with:
          files: ./coverage.txt
      
      - name: Run linters
        run: |
          go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
          golangci-lint run
      
      - name: Check formatting
        run: |
          gofmt -l .
          [ -z "$(gofmt -l .)" ]
```

### Test Organization Best Practices

```
project/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── domain/
│   │   ├── user.go
│   │   └── user_test.go          # Unit tests next to code
│   ├── repository/
│   │   ├── postgres.go
│   │   ├── postgres_test.go       # Unit tests
│   │   └── postgres_integration_test.go  # Integration tests
│   └── service/
│       ├── user_service.go
│       └── user_service_test.go
├── test/
│   ├── integration/               # Separate integration tests
│   │   └── api_test.go
│   ├── e2e/                       # End-to-end tests
│   │   └── user_flow_test.go
│   └── testutil/                  # Shared test utilities
│       ├── fixtures.go
│       ├── assert.go
│       └── mock.go
├── testdata/                      # Test data
│   ├── golden/
│   │   └── response.json.golden
│   └── fixtures/
│       └── users.json
└── go.mod
```

### Performance Testing

```go
func BenchmarkEndpoint(b *testing.B) {
    server := startTestServer()
    defer server.Close()
    
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        resp, err := http.Get(server.URL + "/api/users/1")
        if err != nil {
            b.Fatal(err)
        }
        resp.Body.Close()
        
        if resp.StatusCode != 200 {
            b.Errorf("status = %d; want 200", resp.StatusCode)
        }
    }
}

func BenchmarkWithLoadProfile(b *testing.B) {
    server := startTestServer()
    defer server.Close()
    
    // Simulate realistic load pattern
    b.Run("low-load", func(b *testing.B) {
        b.SetParallelism(10)  // 10 concurrent requests
        b.RunParallel(func(pb *testing.PB) {
            for pb.Next() {
                makeRequest(server.URL)
            }
        })
    })
    
    b.Run("high-load", func(b *testing.B) {
        b.SetParallelism(100)  // 100 concurrent requests
        b.RunParallel(func(pb *testing.PB) {
            for pb.Next() {
                makeRequest(server.URL)
            }
        })
    })
}
```

### Chaos Testing

```go
// Test resilience to failures
func TestCircuitBreaker(t *testing.T) {
    failureRate := 0.5
    attempts := 0
    
    // Flaky service
    server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        attempts++
        if rand.Float64() < failureRate {
            w.WriteHeader(http.StatusInternalServerError)
            return
        }
        w.WriteHeader(http.StatusOK)
    }))
    defer server.Close()
    
    // Client with circuit breaker
    client := NewResilientClient(server.URL)
    
    // Make many requests
    successes := 0
    failures := 0
    
    for i := 0; i < 100; i++ {
        err := client.Get("/api/data")
        if err == nil {
            successes++
        } else {
            failures++
        }
    }
    
    t.Logf("Successes: %d, Failures: %d, Attempts: %d", 
        successes, failures, attempts)
    
    // Circuit breaker should reduce attempts
    if attempts > 80 {
        t.Error("circuit breaker not working, too many attempts")
    }
}
```

### Testing Checklist

**Before Committing:**
- [ ] All tests pass (`go test ./...`)
- [ ] Tests pass with race detector (`go test -race ./...`)
- [ ] Code coverage is adequate (`go test -cover ./...`)
- [ ] Linters pass (`golangci-lint run`)
- [ ] Code is formatted (`gofmt -w .`)
- [ ] No unnecessary test skips
- [ ] Tests are independent (can run in any order)
- [ ] Meaningful test names
- [ ] Edge cases covered

**For New Features:**
- [ ] Unit tests for business logic
- [ ] Integration tests for external dependencies
- [ ] Examples for public APIs
- [ ] Benchmarks for performance-critical code
- [ ] Error cases tested
- [ ] Concurrent usage tested (if applicable)

**For Bug Fixes:**
- [ ] Test that reproduces the bug
- [ ] Test passes after fix
- [ ] Related edge cases covered

## Final Thoughts on Testing

### The Testing Mindset

**Think like an attacker:**
- What inputs will break this?
- What edge cases exist?
- What happens under load?
- What if dependencies fail?

**Write tests that:**
- Document expected behavior
- Prevent regressions
- Enable refactoring
- Guide design

**Remember:**
- Tests are code too - keep them clean
- 100% coverage ≠ bug-free
- Fast feedback > comprehensive tests
- Test behavior, not implementation

### Testing Philosophy

> "Write tests. Not too many. Mostly integration." - Guillermo Rauch

**Balance:**
- **Unit tests**: Many, fast, focused
- **Integration tests**: Some, slower, realistic
- **E2E tests**: Few, slowest, critical paths

**Priorities:**
1. **Correctness**: Does it work?
2. **Maintainability**: Can we change it?
3. **Performance**: Is it fast enough?
4. **Coverage**: What's untested?

**Quality Metrics:**
- Test coverage (aim for 70-80%)
- Code maintainability
- CI/CD pipeline speed
- Production error rate

---

## Part 4 Complete Summary

You've mastered Go testing, from unit tests to production-grade quality assurance:

**Core Skills:**
✅ Unit testing with table-driven tests
✅ Mocking and dependency injection
✅ Benchmarking and profiling
✅ Fuzz testing for edge cases
✅ Code coverage analysis
✅ Integration and E2E testing
✅ Static analysis and linters
✅ CI/CD integration

**Production Practices:**
✅ Test organization and structure
✅ Golden files and snapshots
✅ Test fixtures and builders
✅ Concurrency testing
✅ Performance testing
✅ Chaos testing

**Key Principles:**
✅ Test behavior, not implementation
✅ Keep tests independent
✅ Fast feedback loops
✅ Profile before optimizing
✅ Automate quality checks

### Total Series Progress

- ✅ **Part 1:** Go Fundamentals (~19,500 words)
- ✅ **Part 2:** Concurrency & Runtime (~15,760 words)
- ✅ **Part 3:** Advanced Patterns (~14,550 words)
- ✅ **Part 4:** Testing & Quality (~12,000 words)
- ⏳ **Part 5:** Production Architecture
- ⏳ **Part 6-10:** Coming soon

**Total:** ~61,810 words of comprehensive Go mastery!

Ready to build production systems? Continue to **Part 5: Production Architecture Patterns** where we'll explore clean architecture, DDD, microservices, and scalable system design in Go!


## Bonus: Advanced Testing Techniques

### Property-Based Testing

Beyond fuzz testing, manual property-based tests:

```go
func TestStringReverse_Properties(t *testing.T) {
    // Property: reversing twice gives original
    for i := 0; i < 1000; i++ {
        s := randomString(50)
        if Reverse(Reverse(s)) != s {
            t.Errorf("Reverse(Reverse(%q)) != %q", s, s)
        }
    }
    
    // Property: length unchanged
    for i := 0; i < 1000; i++ {
        s := randomString(50)
        if len(Reverse(s)) != len(s) {
            t.Errorf("length changed")
        }
    }
    
    // Property: first becomes last
    for i := 0; i < 1000; i++ {
        s := randomString(50)
        if s == "" {
            continue
        }
        rev := Reverse(s)
        first := []rune(s)[0]
        last := []rune(rev)[len(rev)-1]
        if first != last {
            t.Errorf("first char %c != last char %c", first, last)
        }
    }
}

func randomString(n int) string {
    runes := make([]rune, n)
    for i := range runes {
        runes[i] = rune(rand.Intn(128))
    }
    return string(runes)
}
```

### Mutation Testing

Verify test quality by mutating code:

```go
// Original function
func Max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

// Mutants to try:
// 1. if a >= b { return a }     // Should be caught
// 2. if a < b { return a }      // Should be caught
// 3. return b                   // Should be caught

func TestMax_Comprehensive(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {5, 3, 5},   // Catches mutant 2, 3
        {3, 5, 5},   // Catches mutant 2
        {5, 5, 5},   // Catches mutant 1
    }
    
    for _, tt := range tests {
        if got := Max(tt.a, tt.b); got != tt.want {
            t.Errorf("Max(%d, %d) = %d; want %d", 
                tt.a, tt.b, got, tt.want)
        }
    }
}
```

### Data-Driven Testing

```go
// Load test cases from files
func TestCalculator_FromCSV(t *testing.T) {
    file, err := os.Open("testdata/calculator_tests.csv")
    if err != nil {
        t.Fatal(err)
    }
    defer file.Close()
    
    reader := csv.NewReader(file)
    records, err := reader.ReadAll()
    if err != nil {
        t.Fatal(err)
    }
    
    // Skip header
    for _, record := range records[1:] {
        operation := record[0]
        a, _ := strconv.Atoi(record[1])
        b, _ := strconv.Atoi(record[2])
        expected, _ := strconv.Atoi(record[3])
        
        t.Run(fmt.Sprintf("%s(%d,%d)", operation, a, b), func(t *testing.T) {
            var got int
            
            switch operation {
            case "add":
                got = Add(a, b)
            case "subtract":
                got = Subtract(a, b)
            case "multiply":
                got = Multiply(a, b)
            }
            
            if got != expected {
                t.Errorf("got %d; want %d", got, expected)
            }
        })
    }
}

// testdata/calculator_tests.csv:
// operation,a,b,expected
// add,2,3,5
// add,-1,1,0
// subtract,5,3,2
// multiply,4,5,20
```

### Parameterized Tests with Generics

```go
func TestSort[T constraints.Ordered](t *testing.T, sort func([]T)) {
    tests := []struct {
        name  string
        input []T
        want  []T
    }{
        {"empty", []T{}, []T{}},
        {"single", []T{1}, []T{1}},
        {"sorted", []T{1, 2, 3}, []T{1, 2, 3}},
        {"reverse", []T{3, 2, 1}, []T{1, 2, 3}},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := make([]T, len(tt.input))
            copy(got, tt.input)
            sort(got)
            
            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("got %v; want %v", got, tt.want)
            }
        })
    }
}

// Use for different types
func TestBubbleSort(t *testing.T) {
    t.Run("int", func(t *testing.T) {
        TestSort(t, func(s []int) { BubbleSort(s) })
    })
    
    t.Run("string", func(t *testing.T) {
        TestSort(t, func(s []string) { BubbleSort(s) })
    })
}
```

### State Machine Testing

```go
type OrderState int

const (
    OrderPending OrderState = iota
    OrderConfirmed
    OrderShipped
    OrderDelivered
    OrderCancelled
)

func TestOrderStateMachine(t *testing.T) {
    validTransitions := map[OrderState][]OrderState{
        OrderPending:    {OrderConfirmed, OrderCancelled},
        OrderConfirmed:  {OrderShipped, OrderCancelled},
        OrderShipped:    {OrderDelivered},
        OrderDelivered:  {},
        OrderCancelled:  {},
    }
    
    for fromState, toStates := range validTransitions {
        for toState := OrderPending; toState <= OrderCancelled; toState++ {
            order := &Order{State: fromState}
            err := order.TransitionTo(toState)
            
            valid := contains(toStates, toState)
            
            if valid && err != nil {
                t.Errorf("transition %v -> %v should be valid, got error: %v",
                    fromState, toState, err)
            }
            
            if !valid && err == nil {
                t.Errorf("transition %v -> %v should be invalid",
                    fromState, toState)
            }
        }
    }
}

func contains(slice []OrderState, item OrderState) bool {
    for _, s := range slice {
        if s == item {
            return true
        }
    }
    return false
}
```

### Contract Testing for APIs

```go
// Define API contract
type UserAPIContract struct {
    CreateUser struct {
        Request  CreateUserRequest
        Response User
        Status   int
    }
    GetUser struct {
        Request  int // User ID
        Response User
        Status   int
    }
}

func TestUserAPI_Contract(t *testing.T) {
    server := startTestServer()
    defer server.Close()
    
    t.Run("CreateUser", func(t *testing.T) {
        req := CreateUserRequest{
            Name:  "Alice",
            Email: "alice@example.com",
        }
        
        body, _ := json.Marshal(req)
        resp, err := http.Post(
            server.URL+"/users",
            "application/json",
            bytes.NewReader(body),
        )
        if err != nil {
            t.Fatal(err)
        }
        defer resp.Body.Close()
        
        // Verify status code
        if resp.StatusCode != http.StatusCreated {
            t.Errorf("status = %d; want %d", 
                resp.StatusCode, http.StatusCreated)
        }
        
        // Verify response structure
        var user User
        if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
            t.Fatal("invalid response structure")
        }
        
        // Verify required fields
        if user.ID == 0 {
            t.Error("ID should be set")
        }
        if user.Name != req.Name {
            t.Errorf("name = %q; want %q", user.Name, req.Name)
        }
        
        // Verify response headers
        if ct := resp.Header.Get("Content-Type"); ct != "application/json" {
            t.Errorf("Content-Type = %q; want application/json", ct)
        }
    })
}
```

### Load Testing

```go
func TestAPIUnderLoad(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping load test in short mode")
    }
    
    server := startTestServer()
    defer server.Close()
    
    // Test configuration
    duration := 30 * time.Second
    concurrency := 50
    
    // Metrics
    var (
        requests   int64
        errors     int64
        totalTime  int64
        mu         sync.Mutex
        latencies  []time.Duration
    )
    
    // Start time
    start := time.Now()
    deadline := start.Add(duration)
    
    // Launch workers
    var wg sync.WaitGroup
    for i := 0; i < concurrency; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            
            for time.Now().Before(deadline) {
                reqStart := time.Now()
                
                resp, err := http.Get(server.URL + "/api/users")
                
                latency := time.Since(reqStart)
                
                atomic.AddInt64(&requests, 1)
                atomic.AddInt64(&totalTime, int64(latency))
                
                if err != nil || resp.StatusCode != 200 {
                    atomic.AddInt64(&errors, 1)
                }
                
                if resp != nil {
                    resp.Body.Close()
                }
                
                mu.Lock()
                latencies = append(latencies, latency)
                mu.Unlock()
            }
        }()
    }
    
    wg.Wait()
    
    // Calculate metrics
    avgLatency := time.Duration(totalTime / requests)
    rps := float64(requests) / duration.Seconds()
    errorRate := float64(errors) / float64(requests) * 100
    
    // Sort latencies for percentiles
    sort.Slice(latencies, func(i, j int) bool {
        return latencies[i] < latencies[j]
    })
    
    p50 := latencies[len(latencies)*50/100]
    p95 := latencies[len(latencies)*95/100]
    p99 := latencies[len(latencies)*99/100]
    
    // Report
    t.Logf("Load Test Results:")
    t.Logf("  Duration: %v", duration)
    t.Logf("  Concurrency: %d", concurrency)
    t.Logf("  Total Requests: %d", requests)
    t.Logf("  Requests/sec: %.2f", rps)
    t.Logf("  Error Rate: %.2f%%", errorRate)
    t.Logf("  Avg Latency: %v", avgLatency)
    t.Logf("  P50 Latency: %v", p50)
    t.Logf("  P95 Latency: %v", p95)
    t.Logf("  P99 Latency: %v", p99)
    
    // Assertions
    if errorRate > 1.0 {
        t.Errorf("error rate %.2f%% exceeds 1%%", errorRate)
    }
    
    if p95 > 100*time.Millisecond {
        t.Errorf("P95 latency %v exceeds 100ms", p95)
    }
}
```

### Comparative Testing

```go
func TestAlgorithmEquivalence(t *testing.T) {
    implementations := map[string]func([]int) []int{
        "BubbleSort":    BubbleSort,
        "QuickSort":     QuickSort,
        "MergeSort":     MergeSort,
        "HeapSort":      HeapSort,
    }
    
    testCases := [][]int{
        {},
        {1},
        {1, 2, 3, 4, 5},
        {5, 4, 3, 2, 1},
        {3, 1, 4, 1, 5, 9, 2, 6},
    }
    
    for name, impl := range implementations {
        t.Run(name, func(t *testing.T) {
            for _, tc := range testCases {
                input := make([]int, len(tc))
                copy(input, tc)
                
                result := impl(input)
                
                // Verify sorted
                if !isSorted(result) {
                    t.Errorf("%s did not sort correctly: %v", name, result)
                }
                
                // Verify same elements
                if !sameElements(tc, result) {
                    t.Errorf("%s changed elements: got %v, want %v", 
                        name, result, tc)
                }
            }
        })
    }
}

func isSorted(arr []int) bool {
    for i := 1; i < len(arr); i++ {
        if arr[i] < arr[i-1] {
            return false
        }
    }
    return true
}

func sameElements(a, b []int) bool {
    if len(a) != len(b) {
        return false
    }
    
    freq := make(map[int]int)
    for _, v := range a {
        freq[v]++
    }
    for _, v := range b {
        freq[v]--
        if freq[v] < 0 {
            return false
        }
    }
    return true
}
```

### Testing Best Practices Summary

**1. Test Organization:**
```
✅ Group related tests with subtests
✅ Use descriptive test names
✅ Keep tests close to code (internal/)
✅ Separate integration tests (test/)
✅ Use testdata/ for fixtures
```

**2. Test Quality:**
```
✅ Test behavior, not implementation
✅ One assertion per test (when possible)
✅ Independent tests (no shared state)
✅ Fast feedback (parallelize when safe)
✅ Clear failure messages
```

**3. Coverage Strategy:**
```
✅ 70-80% code coverage target
✅ 100% coverage of critical paths
✅ Focus on edge cases
✅ Property-based tests for algorithms
✅ Fuzz tests for parsers/validators
```

**4. Continuous Integration:**
```
✅ Run tests on every commit
✅ Fast unit tests (<1min)
✅ Slower integration tests (gated)
✅ Nightly full test suite
✅ Coverage tracking over time
```

**5. Test Maintenance:**
```
✅ Refactor tests with production code
✅ Remove duplicate tests
✅ Update golden files when needed
✅ Review test failures promptly
✅ Delete obsolete tests
```

This comprehensive guide provides everything needed to write production-grade tests in Go, from basic unit tests to advanced load testing and chaos engineering!

