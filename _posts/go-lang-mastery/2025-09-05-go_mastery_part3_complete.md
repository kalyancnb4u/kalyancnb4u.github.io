---
title: "Go Mastery - Part 3: Advanced Go Patterns & Techniques"
date: 2025-09-05 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Advanced, Reflection, Generics, Code-generation, Unsafe, Assembly, Patterns, Meta-programming]
---

# Complete Go Mastery Part 3: Advanced Go Patterns & Techniques

## Introduction

Welcome to Part 3 of the Complete Go Mastery series. In Parts 1 and 2, we covered Go fundamentals and concurrency. Now we dive into advanced techniques that distinguish expert Go programmers from intermediate ones.

Advanced Go programming involves understanding features that aren't needed for everyday coding but become essential when building sophisticated systems, libraries, or frameworks. These techniques include reflection for runtime type introspection, generics for type-safe code reuse, code generation for eliminating boilerplate, and unsafe operations for maximum performance.

**What makes these topics "advanced"?**

These features are more complex, have performance implications, and require deeper understanding of Go's type system and runtime. They're powerful tools that should be used judiciously—with great power comes great responsibility.

**What you'll learn in Part 3:**

- **Reflection**: Runtime type inspection and manipulation
- **Generics**: Type-safe generic programming (Go 1.18+)
- **Code Generation**: Automating boilerplate with go generate
- **Unsafe Operations**: Breaking type safety for performance
- **Assembly Integration**: Calling assembly from Go
- **Advanced Interface Patterns**: Type switches, assertion patterns
- **Functional Programming**: Higher-order functions, composition
- **Design Patterns**: Implementing classic patterns in Go

By the end of Part 3, you'll have mastery over Go's advanced features and know when and how to apply them appropriately.

---

## 3.1 Reflection and Metaprogramming

Reflection allows a program to examine its own structure at runtime. Go's `reflect` package provides this capability, enabling code that works with values of unknown types.

### What is Reflection?

Reflection is the ability to inspect type information and manipulate values at runtime:

```go
import "reflect"

func inspect(x interface{}) {
    t := reflect.TypeOf(x)
    v := reflect.ValueOf(x)
    
    fmt.Printf("Type: %v\n", t)
    fmt.Printf("Value: %v\n", v)
    fmt.Printf("Kind: %v\n", t.Kind())
}

inspect(42)           // Type: int, Kind: int
inspect("hello")      // Type: string, Kind: string
inspect([]int{1,2,3}) // Type: []int, Kind: slice
```

### Type and Value

The `reflect` package provides two fundamental types:

**Type**: Represents a Go type

```go
type Type interface {
    // Returns the kind of type (int, string, struct, etc.)
    Kind() Kind
    
    // Returns the type's name within its package
    Name() string
    
    // Returns the type's package path
    PkgPath() string
    
    // For structs: number of fields
    NumField() int
    
    // For structs: get field by index
    Field(i int) StructField
    
    // For functions: number of input parameters
    NumIn() int
    
    // Many more methods...
}
```

**Value**: Represents a Go value

```go
type Value struct {
    // internal fields
}

func (v Value) Kind() Kind
func (v Value) Type() Type
func (v Value) Interface() interface{}
func (v Value) Int() int64
func (v Value) String() string
func (v Value) Bool() bool
// ... many more methods
```

### Basic Reflection Operations

#### **Getting Type Information**

```go
type Person struct {
    Name string
    Age  int
}

func examineType(x interface{}) {
    t := reflect.TypeOf(x)
    
    fmt.Printf("Type: %v\n", t)
    fmt.Printf("Name: %v\n", t.Name())
    fmt.Printf("Kind: %v\n", t.Kind())
    fmt.Printf("Size: %v bytes\n", t.Size())
    
    // For struct types
    if t.Kind() == reflect.Struct {
        fmt.Printf("Number of fields: %v\n", t.NumField())
        
        for i := 0; i < t.NumField(); i++ {
            field := t.Field(i)
            fmt.Printf("  Field %d: %s %s\n", i, field.Name, field.Type)
        }
    }
}

examineType(Person{Name: "Alice", Age: 30})
// Output:
// Type: main.Person
// Name: Person
// Kind: struct
// Size: 24 bytes
// Number of fields: 2
//   Field 0: Name string
//   Field 1: Age int
```

#### **Getting and Setting Values**

```go
func modifyValue(x interface{}) {
    v := reflect.ValueOf(x)
    
    fmt.Printf("Value: %v\n", v)
    fmt.Printf("Type: %v\n", v.Type())
    fmt.Printf("Kind: %v\n", v.Kind())
    fmt.Printf("Can set? %v\n", v.CanSet())  // false! (not addressable)
}

func modifyPointer(x interface{}) {
    v := reflect.ValueOf(x)
    
    if v.Kind() != reflect.Ptr {
        fmt.Println("Not a pointer!")
        return
    }
    
    // Get the element the pointer points to
    elem := v.Elem()
    
    fmt.Printf("Can set? %v\n", elem.CanSet())  // true!
    
    if elem.Kind() == reflect.Int {
        elem.SetInt(100)
    } else if elem.Kind() == reflect.String {
        elem.SetString("modified")
    }
}

// Usage
x := 42
modifyValue(x)   // Can't modify (not addressable)

modifyPointer(&x)  // Can modify
fmt.Println(x)     // 100
```

**Key rule**: To modify a value via reflection, you need a pointer to it.

### Struct Field Inspection

```go
type User struct {
    ID       int    `json:"id" db:"user_id"`
    Username string `json:"username" db:"username"`
    Email    string `json:"email" db:"email" validate:"email"`
}

func inspectStruct(x interface{}) {
    t := reflect.TypeOf(x)
    v := reflect.ValueOf(x)
    
    if t.Kind() != reflect.Struct {
        fmt.Println("Not a struct")
        return
    }
    
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        
        fmt.Printf("Field: %s\n", field.Name)
        fmt.Printf("  Type: %s\n", field.Type)
        fmt.Printf("  Value: %v\n", value.Interface())
        fmt.Printf("  Tag 'json': %s\n", field.Tag.Get("json"))
        fmt.Printf("  Tag 'db': %s\n", field.Tag.Get("db"))
        fmt.Println()
    }
}

user := User{
    ID:       1,
    Username: "alice",
    Email:    "alice@example.com",
}

inspectStruct(user)
```

### Working with Struct Tags

```go
func getJSONFieldNames(x interface{}) []string {
    t := reflect.TypeOf(x)
    
    if t.Kind() == reflect.Ptr {
        t = t.Elem()
    }
    
    if t.Kind() != reflect.Struct {
        return nil
    }
    
    var fields []string
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        
        // Get JSON tag
        jsonTag := field.Tag.Get("json")
        if jsonTag == "" || jsonTag == "-" {
            continue
        }
        
        // Handle "fieldname,omitempty" format
        parts := strings.Split(jsonTag, ",")
        fields = append(fields, parts[0])
    }
    
    return fields
}

user := User{}
fields := getJSONFieldNames(user)
fmt.Println(fields)  // [id username email]
```

### Function Reflection

```go
func inspectFunction(fn interface{}) {
    t := reflect.TypeOf(fn)
    
    if t.Kind() != reflect.Func {
        fmt.Println("Not a function")
        return
    }
    
    fmt.Printf("Function type: %v\n", t)
    fmt.Printf("Number of inputs: %d\n", t.NumIn())
    fmt.Printf("Number of outputs: %d\n", t.NumOut())
    
    // Input types
    for i := 0; i < t.NumIn(); i++ {
        fmt.Printf("  Input %d: %v\n", i, t.In(i))
    }
    
    // Output types
    for i := 0; i < t.NumOut(); i++ {
        fmt.Printf("  Output %d: %v\n", i, t.Out(i))
    }
    
    // Variadic?
    if t.IsVariadic() {
        fmt.Println("Function is variadic")
    }
}

// Example function
func add(a, b int) int {
    return a + b
}

inspectFunction(add)
// Output:
// Function type: func(int, int) int
// Number of inputs: 2
// Number of outputs: 1
//   Input 0: int
//   Input 1: int
//   Output 0: int
```

### Calling Functions Dynamically

```go
func callFunction(fn interface{}, args ...interface{}) []interface{} {
    v := reflect.ValueOf(fn)
    
    if v.Kind() != reflect.Func {
        panic("Not a function")
    }
    
    // Convert arguments to reflect.Value
    in := make([]reflect.Value, len(args))
    for i, arg := range args {
        in[i] = reflect.ValueOf(arg)
    }
    
    // Call function
    out := v.Call(in)
    
    // Convert results back to interface{}
    result := make([]interface{}, len(out))
    for i, val := range out {
        result[i] = val.Interface()
    }
    
    return result
}

// Usage
add := func(a, b int) int { return a + b }
result := callFunction(add, 3, 4)
fmt.Println(result[0])  // 7

// With multiple returns
divide := func(a, b int) (int, int) { return a / b, a % b }
result = callFunction(divide, 10, 3)
fmt.Printf("Quotient: %v, Remainder: %v\n", result[0], result[1])
```

### Creating Values Dynamically

```go
func createInstance(typeName string) interface{} {
    var t reflect.Type
    
    switch typeName {
    case "int":
        t = reflect.TypeOf(0)
    case "string":
        t = reflect.TypeOf("")
    case "User":
        t = reflect.TypeOf(User{})
    default:
        return nil
    }
    
    // Create new value of type
    v := reflect.New(t)
    
    return v.Interface()
}

// Create a *User
userPtr := createInstance("User").(*User)
userPtr.Username = "bob"
fmt.Printf("%+v\n", userPtr)
```

### Practical Example: Generic Deep Copy

```go
func deepCopy(src interface{}) interface{} {
    srcValue := reflect.ValueOf(src)
    return deepCopyValue(srcValue).Interface()
}

func deepCopyValue(src reflect.Value) reflect.Value {
    switch src.Kind() {
    case reflect.Ptr:
        // Handle nil pointers
        if src.IsNil() {
            return reflect.Zero(src.Type())
        }
        
        // Create new pointer
        dst := reflect.New(src.Elem().Type())
        dst.Elem().Set(deepCopyValue(src.Elem()))
        return dst
        
    case reflect.Struct:
        dst := reflect.New(src.Type()).Elem()
        for i := 0; i < src.NumField(); i++ {
            if src.Type().Field(i).PkgPath != "" {
                // Unexported field, skip
                continue
            }
            dst.Field(i).Set(deepCopyValue(src.Field(i)))
        }
        return dst
        
    case reflect.Slice:
        if src.IsNil() {
            return reflect.Zero(src.Type())
        }
        
        dst := reflect.MakeSlice(src.Type(), src.Len(), src.Cap())
        for i := 0; i < src.Len(); i++ {
            dst.Index(i).Set(deepCopyValue(src.Index(i)))
        }
        return dst
        
    case reflect.Map:
        if src.IsNil() {
            return reflect.Zero(src.Type())
        }
        
        dst := reflect.MakeMap(src.Type())
        for _, key := range src.MapKeys() {
            dst.SetMapIndex(key, deepCopyValue(src.MapIndex(key)))
        }
        return dst
        
    default:
        // For basic types, just return the value
        return src
    }
}

// Usage
type Person struct {
    Name    string
    Age     int
    Friends []string
}

original := Person{
    Name:    "Alice",
    Age:     30,
    Friends: []string{"Bob", "Carol"},
}

copied := deepCopy(original).(Person)
copied.Name = "Alice Clone"
copied.Friends[0] = "Dave"

fmt.Printf("Original: %+v\n", original)
// Original: {Name:Alice Age:30 Friends:[Bob Carol]}

fmt.Printf("Copied: %+v\n", copied)
// Copied: {Name:Alice Clone Age:30 Friends:[Dave Carol]}
```

### Practical Example: Struct to Map Converter

```go
func structToMap(s interface{}) map[string]interface{} {
    result := make(map[string]interface{})
    
    v := reflect.ValueOf(s)
    t := reflect.TypeOf(s)
    
    // Handle pointer
    if t.Kind() == reflect.Ptr {
        v = v.Elem()
        t = t.Elem()
    }
    
    if t.Kind() != reflect.Struct {
        return nil
    }
    
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        
        // Skip unexported fields
        if field.PkgPath != "" {
            continue
        }
        
        // Use json tag if present, otherwise use field name
        name := field.Tag.Get("json")
        if name == "" || name == "-" {
            name = field.Name
        } else {
            // Remove options like ",omitempty"
            name = strings.Split(name, ",")[0]
        }
        
        result[name] = value.Interface()
    }
    
    return result
}

// Usage
user := User{
    ID:       1,
    Username: "alice",
    Email:    "alice@example.com",
}

m := structToMap(user)
fmt.Printf("%+v\n", m)
// map[email:alice@example.com id:1 username:alice]
```

### Reflection in Standard Library

Reflection is used extensively in Go's standard library:

**1. encoding/json**

```go
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

// JSON encoder uses reflection to:
// 1. Discover struct fields
// 2. Read struct tags
// 3. Get field values
// 4. Convert to JSON

data, _ := json.Marshal(user)
```

**2. encoding/xml, encoding/gob**

Similarly use reflection for serialization.

**3. database/sql**

```go
rows, _ := db.Query("SELECT id, name FROM users")

for rows.Next() {
    var user User
    // Scan uses reflection to populate struct fields
    rows.Scan(&user.ID, &user.Name)
}
```

**4. fmt package**

```go
// fmt.Printf uses reflection for %v, %+v, %#v
fmt.Printf("%+v\n", user)
```

**5. testing package**

```go
// testing.T uses reflection to discover test functions
// Functions named TestXxx are found via reflection
```

### Performance Implications

**Reflection is slow** compared to direct operations:

```go
func BenchmarkDirect(b *testing.B) {
    user := User{ID: 1, Name: "Alice"}
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        _ = user.ID
    }
}

func BenchmarkReflection(b *testing.B) {
    user := User{ID: 1, Name: "Alice"}
    v := reflect.ValueOf(user)
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        _ = v.Field(0).Interface()
    }
}

// Results (typical):
// BenchmarkDirect-8       1000000000    0.25 ns/op
// BenchmarkReflection-8     50000000   30.00 ns/op
//
// Reflection is ~100x slower!
```

**When to use reflection:**
- ✅ Marshaling/unmarshaling (JSON, XML, etc.)
- ✅ Generic libraries and frameworks
- ✅ Configuration parsing
- ✅ Testing helpers
- ✅ ORMs and database libraries
- ❌ Hot paths or performance-critical code
- ❌ When interfaces can solve the problem
- ❌ Simple type assertions suffice

### Reflection Best Practices

```go
// ✅ GOOD: Cache type information
var userType = reflect.TypeOf(User{})

func processUser(u User) {
    t := userType  // Reuse cached type
    // ...
}

// ✅ GOOD: Check Kind before operations
func processValue(v reflect.Value) {
    if v.Kind() != reflect.Struct {
        return  // Guard against invalid operations
    }
    // ...
}

// ✅ GOOD: Use type assertion when type is known
func process(x interface{}) {
    // If you know it's a User, use type assertion
    if user, ok := x.(User); ok {
        fmt.Println(user.Name)  // Direct access, fast
        return
    }
    
    // Fall back to reflection only if needed
    v := reflect.ValueOf(x)
    // ...
}

// ❌ BAD: Reflection in hot loop
for i := 0; i < 1000000; i++ {
    v := reflect.ValueOf(data)  // Expensive!
    process(v)
}

// ✅ GOOD: Reflect once, reuse
v := reflect.ValueOf(data)
for i := 0; i < 1000000; i++ {
    process(v)
}
```

### Advanced: Reflect Function with Type Safety

```go
// Generic function caller with type safety
type FuncCaller struct {
    fn   reflect.Value
    fnType reflect.Type
}

func NewFuncCaller(fn interface{}) *FuncCaller {
    v := reflect.ValueOf(fn)
    if v.Kind() != reflect.Func {
        panic("not a function")
    }
    
    return &FuncCaller{
        fn:   v,
        fnType: v.Type(),
    }
}

func (fc *FuncCaller) Call(args ...interface{}) ([]interface{}, error) {
    // Validate argument count
    if len(args) != fc.fnType.NumIn() {
        return nil, fmt.Errorf("expected %d args, got %d", 
            fc.fnType.NumIn(), len(args))
    }
    
    // Validate argument types and convert
    in := make([]reflect.Value, len(args))
    for i, arg := range args {
        argValue := reflect.ValueOf(arg)
        expectedType := fc.fnType.In(i)
        
        if !argValue.Type().AssignableTo(expectedType) {
            return nil, fmt.Errorf("arg %d: expected %v, got %v",
                i, expectedType, argValue.Type())
        }
        
        in[i] = argValue
    }
    
    // Call function
    out := fc.fn.Call(in)
    
    // Convert results
    result := make([]interface{}, len(out))
    for i, val := range out {
        result[i] = val.Interface()
    }
    
    return result, nil
}

// Usage
add := func(a, b int) int { return a + b }
caller := NewFuncCaller(add)

result, err := caller.Call(3, 4)
if err != nil {
    log.Fatal(err)
}
fmt.Println(result[0])  // 7

// Type mismatch caught
_, err = caller.Call("hello", "world")
fmt.Println(err)  // arg 0: expected int, got string
```

### Frequently Asked Questions (Reflection)

**Q1: When should I use reflection vs interfaces?**

**A:**

**Use interfaces when:**
- Behavior is known at compile time
- Type safety is important
- Performance matters
- Types implement common methods

```go
// ✅ GOOD: Use interface
type Saver interface {
    Save() error
}

func saveAll(items []Saver) error {
    for _, item := range items {
        if err := item.Save(); err != nil {
            return err
        }
    }
    return nil
}
```

**Use reflection when:**
- Working with unknown types (generic libraries)
- Inspecting struct tags
- Marshaling/unmarshaling
- Code generation tools
- Testing frameworks

```go
// ✅ GOOD: Use reflection
func marshalJSON(v interface{}) ([]byte, error) {
    // Unknown type, need reflection
    return json.Marshal(v)
}
```

---

**Q2: Why is reflection slow?**

**A:**

Reflection is slow because:
1. **Type information lookup** at runtime
2. **No compiler optimizations** (unknown types)
3. **Boxing and unboxing** (interface{} conversions)
4. **Validation overhead** (type checks, bounds checks)

```go
// Direct: ~0.3 ns
x := user.Name

// Reflection: ~30 ns (100x slower)
v := reflect.ValueOf(user)
x := v.Field(0).Interface().(string)
```

**Why the overhead?**
- Direct access: Compiler knows exact memory offset
- Reflection: Must look up field by name/index at runtime

---

**Q3: Can I modify unexported struct fields with reflection?**

**A:**

**No, not safely.** Unexported fields are not settable:

```go
type User struct {
    name string  // unexported
}

v := reflect.ValueOf(&user).Elem()
field := v.Field(0)

fmt.Println(field.CanSet())  // false

// This will panic!
// field.SetString("Alice")  // panic: cannot set unexported field
```

**Workaround with unsafe (not recommended):**

```go
import "unsafe"

// ⚠️ DANGEROUS: Breaks encapsulation
field := v.Field(0)
reflect.NewAt(field.Type(), unsafe.Pointer(field.UnsafeAddr())).
    Elem().SetString("Alice")
```

**Better approach:** Make field exported or provide setter method.

---

### Interview Questions (Reflection)

**Question 1: Implement a function that recursively prints all fields of a struct, including nested structs.**

**Difficulty:** Mid-Level

**Answer:**

```go
func printStruct(v interface{}, indent string) {
    val := reflect.ValueOf(v)
    typ := reflect.TypeOf(v)
    
    // Handle pointers
    if typ.Kind() == reflect.Ptr {
        if val.IsNil() {
            fmt.Printf("%s<nil>\n", indent)
            return
        }
        val = val.Elem()
        typ = typ.Elem()
    }
    
    if typ.Kind() != reflect.Struct {
        fmt.Printf("%s%v (%v)\n", indent, val.Interface(), typ)
        return
    }
    
    fmt.Printf("%s%s {\n", indent, typ.Name())
    
    for i := 0; i < typ.NumField(); i++ {
        field := typ.Field(i)
        fieldVal := val.Field(i)
        
        // Skip unexported fields
        if field.PkgPath != "" {
            continue
        }
        
        fmt.Printf("%s  %s: ", indent, field.Name)
        
        if field.Type.Kind() == reflect.Struct {
            fmt.Println()
            printStruct(fieldVal.Interface(), indent+"    ")
        } else {
            fmt.Printf("%v (%v)\n", fieldVal.Interface(), field.Type)
        }
    }
    
    fmt.Printf("%s}\n", indent)
}

// Usage
type Address struct {
    City    string
    ZipCode string
}

type Person struct {
    Name    string
    Age     int
    Address Address
}

p := Person{
    Name: "Alice",
    Age:  30,
    Address: Address{
        City:    "NYC",
        ZipCode: "10001",
    },
}

printStruct(p, "")
// Output:
// Person {
//   Name: Alice (string)
//   Age: 30 (int)
//   Address:
//     Address {
//       City: NYC (string)
//       ZipCode: 10001 (string)
//     }
// }
```

**What Interviewers Look For:**
- Understanding of reflection basics (Type, Value, Kind)
- Handling nested structures recursively
- Proper handling of pointers
- Skipping unexported fields
- Code organization and readability

---

**Question 2: What's wrong with this code?**

```go
func setField(obj interface{}, fieldName string, value interface{}) {
    v := reflect.ValueOf(obj)
    field := v.FieldByName(fieldName)
    field.Set(reflect.ValueOf(value))
}

user := User{Name: "Alice"}
setField(user, "Name", "Bob")
```

**Difficulty:** Mid-Level

**Answer:**

**Problems:**

1. **Passing value instead of pointer** - Can't modify original
2. **Not checking if field exists** - FieldByName returns zero Value if not found
3. **Not checking if field is settable** - Will panic

**Fixed version:**

```go
func setField(obj interface{}, fieldName string, value interface{}) error {
    v := reflect.ValueOf(obj)
    
    // Must be a pointer
    if v.Kind() != reflect.Ptr {
        return errors.New("obj must be a pointer")
    }
    
    // Get the element the pointer points to
    v = v.Elem()
    
    if v.Kind() != reflect.Struct {
        return errors.New("obj must be a struct")
    }
    
    // Get field
    field := v.FieldByName(fieldName)
    if !field.IsValid() {
        return fmt.Errorf("field %s not found", fieldName)
    }
    
    // Check if settable
    if !field.CanSet() {
        return fmt.Errorf("field %s is not settable", fieldName)
    }
    
    // Check type compatibility
    valueReflect := reflect.ValueOf(value)
    if !valueReflect.Type().AssignableTo(field.Type()) {
        return fmt.Errorf("value type %v not assignable to field type %v",
            valueReflect.Type(), field.Type())
    }
    
    // Set the value
    field.Set(valueReflect)
    return nil
}

// Usage
user := User{Name: "Alice"}
err := setField(&user, "Name", "Bob")  // Pass pointer!
if err != nil {
    log.Fatal(err)
}
fmt.Println(user.Name)  // Bob
```

**What Interviewers Look For:**
- Understanding that reflection needs pointers to modify
- Proper error handling
- Type checking before operations
- Knowledge of CanSet() and IsValid()

---

### Key Takeaways (Reflection)

✅ **Reflection enables runtime type inspection**
- TypeOf() for type information
- ValueOf() for value manipulation
- Kind() for fundamental type

✅ **Use reflection judiciously**
- Powerful but slow (~100x slower than direct access)
- Appropriate for generic libraries, serialization
- Not for hot paths or performance-critical code

✅ **Struct tags drive behavior**
- JSON, XML, database libraries use tags
- Define custom tags for your libraries
- Access with field.Tag.Get("tagname")

✅ **Modifying values requires pointers**
- CanSet() checks if value is settable
- Must pass pointer to modify original
- Unexported fields cannot be set

✅ **Standard library uses reflection extensively**
- encoding/json, encoding/xml
- database/sql
- fmt package
- testing framework

Continuing with Part 3...
## 3.2 Generics (Go 1.18+)

Go 1.18 introduced generics (officially called "type parameters"), enabling type-safe generic programming without reflection overhead.

### Why Generics?

Before generics, you had three options for generic behavior:

```go
// Option 1: Code duplication
func MinInt(a, b int) int {
    if a < b { return a }
    return b
}

func MinFloat64(a, b float64) float64 {
    if a < b { return a }
    return b
}

// Option 2: interface{} + type assertions (slow, not type-safe)
func Min(a, b interface{}) interface{} {
    // ... ugly type assertions
}

// Option 3: Code generation (complex build process)
// //go:generate genmin -type=int
```

**With generics:**

```go
func Min[T constraints.Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

// Type-safe, no duplication, compile-time checked
x := Min(3, 5)        // int
y := Min(3.14, 2.71)  // float64
z := Min("abc", "xyz") // string
```

### Generic Functions

#### **Basic Generic Function**

```go
// Single type parameter
func Print[T any](s []T) {
    for _, v := range s {
        fmt.Println(v)
    }
}

Print([]int{1, 2, 3})
Print([]string{"a", "b", "c"})

// Multiple type parameters
func Pair[K comparable, V any](key K, value V) map[K]V {
    return map[K]V{key: value}
}

m1 := Pair("name", "Alice")     // map[string]string
m2 := Pair(1, "one")            // map[int]string
m3 := Pair("count", 42)         // map[string]int
```

#### **Type Parameter Inference**

Go can often infer type parameters:

```go
func Identity[T any](x T) T {
    return x
}

// Explicit type parameter
result := Identity[int](42)

// Inferred (usually preferred)
result := Identity(42)  // T inferred as int
```

### Type Constraints

Constraints specify what operations are allowed on type parameters:

#### **The `any` Constraint**

```go
// any is an alias for interface{}
// Allows any type but no operations except type assertion
func First[T any](s []T) T {
    return s[0]
}

// Can't do this:
// func Add[T any](a, b T) T {
//     return a + b  // Error: operator + not defined for T
// }
```

#### **The `comparable` Constraint**

```go
// comparable allows == and != operators
func Contains[T comparable](slice []T, target T) bool {
    for _, v := range slice {
        if v == target {  // OK: comparable allows ==
            return true
        }
    }
    return false
}

Contains([]int{1, 2, 3}, 2)        // true
Contains([]string{"a", "b"}, "c")  // false
```

#### **Standard Constraints Package**

```go
import "golang.org/x/exp/constraints"

// Ordered: types that support <, <=, >, >=
func Max[T constraints.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

// Signed: signed integer types
func Abs[T constraints.Signed](x T) T {
    if x < 0 {
        return -x
    }
    return x
}

// Integer: all integer types
func GCD[T constraints.Integer](a, b T) T {
    for b != 0 {
        a, b = b, a%b
    }
    return a
}

// Float: all float types
func Average[T constraints.Float](numbers []T) T {
    var sum T
    for _, n := range numbers {
        sum += n
    }
    return sum / T(len(numbers))
}
```

#### **Custom Constraints**

```go
// Define interface constraint
type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

func Sum[T Numeric](numbers []T) T {
    var sum T
    for _, n := range numbers {
        sum += n
    }
    return sum
}

// The ~ allows types with underlying type
type MyInt int  // Has underlying type int

nums := []MyInt{1, 2, 3}
total := Sum(nums)  // Works! ~ allows MyInt
```

#### **Method Constraints**

```go
// Constraint requiring specific methods
type Stringer interface {
    String() string
}

func Concatenate[T Stringer](items []T) string {
    var result string
    for _, item := range items {
        result += item.String()  // OK: Stringer has String()
    }
    return result
}

type Person struct {
    Name string
}

func (p Person) String() string {
    return p.Name
}

people := []Person{{Name: "Alice"}, {Name: "Bob"}}
names := Concatenate(people)  // "AliceBob"
```

#### **Union Constraints**

```go
// Allow specific types
type IntOrString interface {
    int | string
}

func Double[T IntOrString](x T) T {
    var zero T
    switch any(x).(type) {
    case int:
        return any(x.(int) * 2).(T)
    case string:
        return any(x.(string) + x.(string)).(T)
    }
    return zero
}

Double(5)      // 10
Double("hi")   // "hihi"
```

### Generic Types

#### **Generic Struct**

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

func (s *Stack[T]) Peek() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    return s.items[len(s.items)-1], true
}

func (s *Stack[T]) Len() int {
    return len(s.items)
}

// Usage
intStack := Stack[int]{}
intStack.Push(1)
intStack.Push(2)
intStack.Push(3)

val, ok := intStack.Pop()  // 3, true

stringStack := Stack[string]{}
stringStack.Push("hello")
stringStack.Push("world")
```

#### **Generic Slice Wrapper**

```go
type List[T any] []T

func (l List[T]) Map[U any](fn func(T) U) List[U] {
    result := make(List[U], len(l))
    for i, v := range l {
        result[i] = fn(v)
    }
    return result
}

func (l List[T]) Filter(fn func(T) bool) List[T] {
    result := make(List[T], 0)
    for _, v := range l {
        if fn(v) {
            result = append(result, v)
        }
    }
    return result
}

func (l List[T]) Reduce[U any](initial U, fn func(U, T) U) U {
    result := initial
    for _, v := range l {
        result = fn(result, v)
    }
    return result
}

// Usage
numbers := List[int]{1, 2, 3, 4, 5}

// Map: int -> string
strings := numbers.Map(func(n int) string {
    return fmt.Sprintf("%d", n)
})
fmt.Println(strings)  // [1 2 3 4 5]

// Filter: keep even numbers
evens := numbers.Filter(func(n int) bool {
    return n%2 == 0
})
fmt.Println(evens)  // [2 4]

// Reduce: sum
sum := numbers.Reduce(0, func(acc, n int) int {
    return acc + n
})
fmt.Println(sum)  // 15
```

#### **Generic Map**

```go
type SafeMap[K comparable, V any] struct {
    mu    sync.RWMutex
    items map[K]V
}

func NewSafeMap[K comparable, V any]() *SafeMap[K, V] {
    return &SafeMap[K, V]{
        items: make(map[K]V),
    }
}

func (m *SafeMap[K, V]) Set(key K, value V) {
    m.mu.Lock()
    defer m.mu.Unlock()
    m.items[key] = value
}

func (m *SafeMap[K, V]) Get(key K) (V, bool) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    val, ok := m.items[key]
    return val, ok
}

func (m *SafeMap[K, V]) Delete(key K) {
    m.mu.Lock()
    defer m.mu.Unlock()
    delete(m.items, key)
}

func (m *SafeMap[K, V]) Len() int {
    m.mu.RLock()
    defer m.mu.RUnlock()
    return len(m.items)
}

// Usage
cache := NewSafeMap[string, int]()
cache.Set("age", 30)
age, ok := cache.Get("age")  // 30, true

userCache := NewSafeMap[int, User]()
userCache.Set(1, User{Name: "Alice"})
```

### Generic Interfaces

```go
type Container[T any] interface {
    Add(item T)
    Get() (T, bool)
    Len() int
}

type Queue[T any] struct {
    items []T
}

func (q *Queue[T]) Add(item T) {
    q.items = append(q.items, item)
}

func (q *Queue[T]) Get() (T, bool) {
    if len(q.items) == 0 {
        var zero T
        return zero, false
    }
    item := q.items[0]
    q.items = q.items[1:]
    return item, true
}

func (q *Queue[T]) Len() int {
    return len(q.items)
}

// Queue satisfies Container[T]
func ProcessContainer[T any](c Container[T], item T) {
    c.Add(item)
    val, ok := c.Get()
    if ok {
        fmt.Println(val)
    }
}

q := &Queue[string]{}
ProcessContainer(q, "hello")
```

### Practical Generic Examples

#### **Generic Optional/Maybe Type**

```go
type Optional[T any] struct {
    value   T
    present bool
}

func Some[T any](value T) Optional[T] {
    return Optional[T]{value: value, present: true}
}

func None[T any]() Optional[T] {
    return Optional[T]{present: false}
}

func (o Optional[T]) IsPresent() bool {
    return o.present
}

func (o Optional[T]) Get() (T, bool) {
    return o.value, o.present
}

func (o Optional[T]) OrElse(defaultValue T) T {
    if o.present {
        return o.value
    }
    return defaultValue
}

func (o Optional[T]) Map[U any](fn func(T) U) Optional[U] {
    if !o.present {
        return None[U]()
    }
    return Some(fn(o.value))
}

// Usage
opt := Some(42)
if val, ok := opt.Get(); ok {
    fmt.Println(val)  // 42
}

empty := None[int]()
val := empty.OrElse(0)  // 0

// Map transformation
strOpt := opt.Map(func(n int) string {
    return fmt.Sprintf("%d", n)
})
fmt.Println(strOpt.Get())  // "42", true
```

#### **Generic Result Type**

```go
type Result[T any] struct {
    value T
    err   error
}

func OK[T any](value T) Result[T] {
    return Result[T]{value: value}
}

func Err[T any](err error) Result[T] {
    return Result[T]{err: err}
}

func (r Result[T]) IsOK() bool {
    return r.err == nil
}

func (r Result[T]) Unwrap() (T, error) {
    return r.value, r.err
}

func (r Result[T]) UnwrapOr(defaultValue T) T {
    if r.err != nil {
        return defaultValue
    }
    return r.value
}

func (r Result[T]) Map[U any](fn func(T) U) Result[U] {
    if r.err != nil {
        return Err[U](r.err)
    }
    return OK(fn(r.value))
}

// Usage
func divide(a, b float64) Result[float64] {
    if b == 0 {
        return Err[float64](errors.New("division by zero"))
    }
    return OK(a / b)
}

result := divide(10, 2)
if result.IsOK() {
    val, _ := result.Unwrap()
    fmt.Println(val)  // 5
}

result = divide(10, 0)
val := result.UnwrapOr(0.0)  // 0.0 (default)
```

#### **Generic Tree**

```go
type Tree[T any] struct {
    value T
    left  *Tree[T]
    right *Tree[T]
}

func (t *Tree[T]) Insert(value T, less func(T, T) bool) {
    if t == nil {
        return
    }
    
    if less(value, t.value) {
        if t.left == nil {
            t.left = &Tree[T]{value: value}
        } else {
            t.left.Insert(value, less)
        }
    } else {
        if t.right == nil {
            t.right = &Tree[T]{value: value}
        } else {
            t.right.Insert(value, less)
        }
    }
}

func (t *Tree[T]) InOrder(visit func(T)) {
    if t == nil {
        return
    }
    
    t.left.InOrder(visit)
    visit(t.value)
    t.right.InOrder(visit)
}

// Usage
tree := &Tree[int]{value: 5}
less := func(a, b int) bool { return a < b }

tree.Insert(3, less)
tree.Insert(7, less)
tree.Insert(1, less)
tree.Insert(9, less)

// In-order traversal
tree.InOrder(func(val int) {
    fmt.Printf("%d ", val)
})
// Output: 1 3 5 7 9
```

### Generic Algorithms

```go
// Find first element matching predicate
func Find[T any](slice []T, predicate func(T) bool) (T, bool) {
    for _, item := range slice {
        if predicate(item) {
            return item, true
        }
    }
    var zero T
    return zero, false
}

// Map transformation
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Filter elements
func Filter[T any](slice []T, predicate func(T) bool) []T {
    result := make([]T, 0)
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Reduce to single value
func Reduce[T, U any](slice []T, initial U, fn func(U, T) U) U {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

// All elements match predicate
func All[T any](slice []T, predicate func(T) bool) bool {
    for _, v := range slice {
        if !predicate(v) {
            return false
        }
    }
    return true
}

// Any element matches predicate
func Any[T any](slice []T, predicate func(T) bool) bool {
    for _, v := range slice {
        if predicate(v) {
            return true
        }
    }
    return false
}

// Usage
numbers := []int{1, 2, 3, 4, 5}

// Find first even number
even, found := Find(numbers, func(n int) bool { return n%2 == 0 })
fmt.Println(even, found)  // 2, true

// Double all numbers
doubled := Map(numbers, func(n int) int { return n * 2 })
fmt.Println(doubled)  // [2 4 6 8 10]

// Keep only odd numbers
odds := Filter(numbers, func(n int) bool { return n%2 != 0 })
fmt.Println(odds)  // [1 3 5]

// Sum all numbers
sum := Reduce(numbers, 0, func(acc, n int) int { return acc + n })
fmt.Println(sum)  // 15

// All positive?
allPositive := All(numbers, func(n int) bool { return n > 0 })
fmt.Println(allPositive)  // true
```

### Limitations and Gotchas

#### **No Operator Methods**

```go
// ❌ Can't do this (yet)
// type Addable[T any] interface {
//     Add(T) T  // Want to use + operator
// }

// ✅ Current workaround: Use constraints
func Add[T constraints.Integer | constraints.Float](a, b T) T {
    return a + b
}
```

#### **No Type Parameter in Methods**

```go
type Container[T any] struct {
    items []T
}

// ❌ Can't add type parameter to method
// func (c *Container[T]) Convert[U any](fn func(T) U) *Container[U] {
//     // Not allowed
// }

// ✅ Must use function
func Convert[T, U any](c *Container[T], fn func(T) U) *Container[U] {
    result := &Container[U]{items: make([]U, len(c.items))}
    for i, item := range c.items {
        result.items[i] = fn(item)
    }
    return result
}
```

#### **No Generic Type Assertion**

```go
func GetValue[T any](m map[string]interface{}, key string) (T, bool) {
    value, ok := m[key]
    if !ok {
        var zero T
        return zero, false
    }
    
    // ❌ Can't do: value.(T)
    // ✅ Use any intermediate
    if typedValue, ok := value.(T); ok {
        return typedValue, true
    }
    
    var zero T
    return zero, false
}
```

### When to Use Generics

**✅ Use generics for:**
- Data structures (stacks, queues, trees, etc.)
- Container types (optional, result, etc.)
- Algorithms (sort, search, map, filter, etc.)
- Type-safe wrappers (sync primitives, etc.)

**❌ Avoid generics for:**
- Simple functions (type assertion might be clearer)
- Performance-critical code (may have small overhead)
- When interfaces are sufficient
- When behavior varies significantly by type

**Comparison:**

```go
// ✅ GOOD: Generic algorithm
func Reverse[T any](slice []T) []T {
    result := make([]T, len(slice))
    for i, v := range slice {
        result[len(slice)-1-i] = v
    }
    return result
}

// ❌ OVERKILL: Too simple for generics
func Print[T any](x T) {
    fmt.Println(x)
}
// Just use: fmt.Println(x)

// ✅ GOOD: Type-safe data structure
type Set[T comparable] struct {
    items map[T]struct{}
}

// ❌ AVOID: Interface is better
type Processor[T any] interface {
    Process(T) error
}
// Better: type Processor interface { Process(interface{}) error }
```


### Frequently Asked Questions (Generics)

**Q1: What's the performance impact of generics?**

**A:**

Generics in Go use **monomorphization** at compile time:
- Separate code generated for each concrete type used
- No runtime overhead like interface{} + type assertions
- Performance is identical to hand-written typed code

```go
// This generic function
func Max[T constraints.Ordered](a, b T) T {
    if a > b { return a }
    return b
}

// When you call Max(3, 5) and Max(3.14, 2.71)
// Compiler generates essentially:
func MaxInt(a, b int) int {
    if a > b { return a }
    return b
}

func MaxFloat64(a, b float64) float64 {
    if a > b { return a }
    return b
}
```

**Trade-off:** Code size increases (more compiled code), but execution speed is the same.

---

**Q2: When should I use generics vs interfaces?**

**A:**

**Use generics when:**
- Type safety is important
- You need to preserve concrete type
- Working with value types (no boxing overhead)
- Building data structures or algorithms

**Use interfaces when:**
- Behavior matters more than type
- Different types need different implementations
- Runtime polymorphism is needed
- Working with unknown types at compile time

```go
// ✅ Generics: Type-safe data structure
type Stack[T any] struct {
    items []T
}

// ✅ Interface: Different behaviors
type Writer interface {
    Write([]byte) (int, error)
}
// Different types (file, network, buffer) implement differently
```

---

**Q3: Can I use generics with methods?**

**A:**

**Yes, but with limitations:**

```go
// ✅ Generic type with methods
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

// ❌ Can't add type parameters to methods
// func (s *Stack[T]) Convert[U any](fn func(T) U) Stack[U] {
//     // Not allowed
// }

// ✅ Use a generic function instead
func Convert[T, U any](s Stack[T], fn func(T) U) Stack[U] {
    result := Stack[U]{}
    for _, item := range s.items {
        result.Push(fn(item))
    }
    return result
}
```

---

### Interview Questions (Generics)

**Question 1: Implement a generic LRU cache.**

**Difficulty:** Senior

**Answer:**

```go
import "container/list"

type LRU[K comparable, V any] struct {
    capacity int
    cache    map[K]*list.Element
    list     *list.List
}

type entry[K comparable, V any] struct {
    key   K
    value V
}

func NewLRU[K comparable, V any](capacity int) *LRU[K, V] {
    return &LRU[K, V]{
        capacity: capacity,
        cache:    make(map[K]*list.Element),
        list:     list.New(),
    }
}

func (l *LRU[K, V]) Get(key K) (V, bool) {
    if elem, ok := l.cache[key]; ok {
        // Move to front (most recently used)
        l.list.MoveToFront(elem)
        return elem.Value.(*entry[K, V]).value, true
    }
    var zero V
    return zero, false
}

func (l *LRU[K, V]) Put(key K, value V) {
    if elem, ok := l.cache[key]; ok {
        // Update existing
        l.list.MoveToFront(elem)
        elem.Value.(*entry[K, V]).value = value
        return
    }
    
    // Add new
    elem := l.list.PushFront(&entry[K, V]{key: key, value: value})
    l.cache[key] = elem
    
    // Evict if over capacity
    if l.list.Len() > l.capacity {
        oldest := l.list.Back()
        if oldest != nil {
            l.list.Remove(oldest)
            delete(l.cache, oldest.Value.(*entry[K, V]).key)
        }
    }
}

// Usage
cache := NewLRU[string, int](3)
cache.Put("a", 1)
cache.Put("b", 2)
cache.Put("c", 3)
cache.Put("d", 4)  // Evicts "a"

if val, ok := cache.Get("a"); ok {
    fmt.Println(val)  // Not found
}
```

**What Interviewers Look For:**
- Understanding of LRU cache algorithm
- Proper use of generics with multiple type parameters
- Efficient data structure choice (map + doubly-linked list)
- Handling capacity limits
- Clean API design

---

### Key Takeaways (Generics)

✅ **Generics enable type-safe code reuse**
- No reflection overhead
- Compile-time type checking
- Same performance as hand-written code

✅ **Type parameters and constraints**
- any: allows any type
- comparable: allows == and !=
- Custom constraints: define with interfaces

✅ **Generic types and functions**
- Functions: func Foo[T any](x T) T
- Types: type Stack[T any] struct
- Methods: func (s *Stack[T]) Push(item T)

✅ **Common patterns**
- Data structures (stack, queue, tree, etc.)
- Container types (optional, result, etc.)
- Generic algorithms (map, filter, reduce, etc.)

✅ **When to use**
- Type safety is important
- Building reusable data structures
- Avoiding code duplication
- Not for simple cases where interfaces suffice

---

## 3.3 Code Generation

Code generation automates the creation of boilerplate code, improving productivity and reducing errors.

### go generate

The `go generate` command scans for special comments and executes the commands they specify:

```go
//go:generate command arguments

// Example:
//go:generate stringer -type=Status
//go:generate mockgen -source=interface.go -destination=mock.go
```

**Running:**

```bash
go generate ./...      # Run in all packages
go generate file.go    # Run for specific file
```

### Common Use Cases

#### **1. Generating String Methods**

```go
//go:generate stringer -type=Status

type Status int

const (
    Pending Status = iota
    InProgress
    Completed
    Failed
)

// After go generate, stringer creates:
// func (s Status) String() string {
//     switch s {
//     case Pending: return "Pending"
//     case InProgress: return "InProgress"
//     case Completed: return "Completed"
//     case Failed: return "Failed"
//     default: return fmt.Sprintf("Status(%d)", s)
//     }
// }

func main() {
    status := InProgress
    fmt.Println(status)  // "InProgress"
}
```

#### **2. Generating Mocks**

```go
// database.go
//go:generate mockgen -source=database.go -destination=database_mock.go -package=main

type Database interface {
    GetUser(id int) (*User, error)
    SaveUser(user *User) error
}

// After go generate, creates mock implementation:
// type MockDatabase struct {
//     ctrl     *gomock.Controller
//     recorder *MockDatabaseMockRecorder
// }
//
// func (m *MockDatabase) GetUser(id int) (*User, error) { ... }
// func (m *MockDatabase) SaveUser(user *User) error { ... }

// In tests:
func TestUserService(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    mockDB := NewMockDatabase(ctrl)
    mockDB.EXPECT().GetUser(1).Return(&User{Name: "Alice"}, nil)
    
    // Test code using mockDB...
}
```

#### **3. Generating Protocol Buffers**

```go
//go:generate protoc --go_out=. --go_opt=paths=source_relative user.proto

// user.proto defines:
// message User {
//     int32 id = 1;
//     string name = 2;
// }

// Generates user.pb.go with:
// type User struct {
//     Id   int32  `protobuf:"varint,1,opt,name=id,proto3"`
//     Name string `protobuf:"bytes,2,opt,name=name,proto3"`
// }
```

#### **4. Generating SQL Code**

```go
//go:generate sqlc generate

// schema.sql defines database schema
// queries.sql defines SQL queries
// sqlc.yaml configures code generation

// Generates type-safe database code:
// func (q *Queries) GetUser(ctx context.Context, id int64) (User, error)
// func (q *Queries) CreateUser(ctx context.Context, arg CreateUserParams) error
```

### Custom Code Generators

#### **Simple Template-Based Generator**

```go
// gen_methods.go
package main

import (
    "os"
    "text/template"
)

type TypeInfo struct {
    Name   string
    Fields []Field
}

type Field struct {
    Name string
    Type string
}

const tmpl = `// Code generated by gen_methods.go DO NOT EDIT.

package {{.Package}}

type {{.Name}} struct {
{{range .Fields}}    {{.Name}} {{.Type}}
{{end}}}

func New{{.Name}}() *{{.Name}} {
    return &{{.Name}}{}
}

{{range .Fields}}
func (t *{{$.Name}}) Get{{.Name}}() {{.Type}} {
    return t.{{.Name}}
}

func (t *{{$.Name}}) Set{{.Name}}(val {{.Type}}) {
    t.{{.Name}} = val
}
{{end}}
`

func main() {
    info := TypeInfo{
        Name: "User",
        Fields: []Field{
            {Name: "ID", Type: "int"},
            {Name: "Name", Type: "string"},
            {Name: "Email", Type: "string"},
        },
    }
    
    t := template.Must(template.New("methods").Parse(tmpl))
    
    f, _ := os.Create("user_generated.go")
    defer f.Close()
    
    t.Execute(f, map[string]interface{}{
        "Package": "main",
        "Name":    info.Name,
        "Fields":  info.Fields,
    })
}

// Use in code:
//go:generate go run gen_methods.go
```

#### **Using go/parser and go/ast**

```go
// analyze.go - Analyzes Go source code
package main

import (
    "fmt"
    "go/ast"
    "go/parser"
    "go/token"
)

func main() {
    src := `
package example

type User struct {
    ID   int
    Name string
}

func (u *User) GetID() int {
    return u.ID
}
`
    
    fset := token.NewFileSet()
    f, err := parser.ParseFile(fset, "", src, 0)
    if err != nil {
        panic(err)
    }
    
    // Inspect AST
    ast.Inspect(f, func(n ast.Node) bool {
        switch x := n.(type) {
        case *ast.TypeSpec:
            fmt.Printf("Type: %s\n", x.Name.Name)
        case *ast.FuncDecl:
            if x.Recv != nil {
                fmt.Printf("Method: %s\n", x.Name.Name)
            } else {
                fmt.Printf("Function: %s\n", x.Name.Name)
            }
        }
        return true
    })
}

// Output:
// Type: User
// Method: GetID
```

#### **Generating Code from AST**

```go
// generator.go
package main

import (
    "go/ast"
    "go/parser"
    "go/token"
    "os"
    "text/template"
)

func generateGetters(filename string) error {
    fset := token.NewFileSet()
    f, err := parser.ParseFile(fset, filename, nil, 0)
    if err != nil {
        return err
    }
    
    var types []TypeInfo
    
    ast.Inspect(f, func(n ast.Node) bool {
        typeSpec, ok := n.(*ast.TypeSpec)
        if !ok {
            return true
        }
        
        structType, ok := typeSpec.Type.(*ast.StructType)
        if !ok {
            return true
        }
        
        info := TypeInfo{Name: typeSpec.Name.Name}
        
        for _, field := range structType.Fields.List {
            for _, name := range field.Names {
                info.Fields = append(info.Fields, Field{
                    Name: name.Name,
                    Type: exprToString(field.Type),
                })
            }
        }
        
        types = append(types, info)
        return true
    })
    
    // Generate code
    tmpl := template.Must(template.New("getters").Parse(getterTemplate))
    
    outFile, _ := os.Create("getters_generated.go")
    defer outFile.Close()
    
    return tmpl.Execute(outFile, map[string]interface{}{
        "Package": f.Name.Name,
        "Types":   types,
    })
}

const getterTemplate = `// Code generated - DO NOT EDIT.
package {{.Package}}

{{range .Types}}
{{range .Fields}}
func (t *{{$.Name}}) Get{{.Name}}() {{.Type}} {
    return t.{{.Name}}
}
{{end}}
{{end}}
`

func exprToString(expr ast.Expr) string {
    switch t := expr.(type) {
    case *ast.Ident:
        return t.Name
    case *ast.StarExpr:
        return "*" + exprToString(t.X)
    case *ast.ArrayType:
        return "[]" + exprToString(t.Elt)
    case *ast.MapType:
        return "map[" + exprToString(t.Key) + "]" + exprToString(t.Value)
    default:
        return "interface{}"
    }
}
```

### Build Tags

Control which files are included in build:

```go
// +build linux darwin

package platform

// This file only compiled on Linux and macOS

func GetPlatform() string {
    return "Unix-like"
}
```

```go
// +build windows

package platform

// This file only compiled on Windows

func GetPlatform() string {
    return "Windows"
}
```

**Modern syntax (Go 1.17+):**

```go
//go:build linux || darwin

package platform
```

**Using in code generation:**

```go
//go:generate go run gen.go -tags=production
```

### Code Generation Best Practices

```go
// ✅ GOOD: Add generated file marker
// Code generated by TOOL_NAME. DO NOT EDIT.

// ✅ GOOD: Commit generated files (usually)
// Allows building without running generators
// Exception: Very large generated files

// ✅ GOOD: Separate generated code
// user.go          - Hand-written
// user_generated.go - Generated

// ✅ GOOD: Make generators idempotent
// Running twice produces same output

// ✅ GOOD: Use build tags if needed
//go:build tools
package tools

import _ "golang.org/x/tools/cmd/stringer"
```

### Popular Code Generation Tools

**1. stringer** - Generate String() methods for enums

```bash
go install golang.org/x/tools/cmd/stringer@latest
```

**2. mockgen** - Generate mocks for interfaces

```bash
go install github.com/golang/mock/mockgen@latest
```

**3. protoc-gen-go** - Protocol buffer compiler

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

**4. sqlc** - Type-safe SQL code generator

```bash
go install github.com/kyleconroy/sqlc/cmd/sqlc@latest
```

**5. ent** - Entity framework with code generation

```bash
go install entgo.io/ent/cmd/ent@latest
```

**6. Wire** - Dependency injection code generator

```bash
go install github.com/google/wire/cmd/wire@latest
```

### Frequently Asked Questions (Code Generation)

**Q1: Should I commit generated code to version control?**

**A:**

**Generally yes:**
- Makes builds possible without generators
- Shows what changed in code reviews
- Ensures consistency across environments

**Sometimes no:**
- Very large generated files
- Frequently changing generated code
- Team has consistent tooling setup

**Best practice:** Document in README which files are generated and how to regenerate them.

---

**Q2: How do I handle generated code in CI/CD?**

**A:**

```yaml
# .github/workflows/test.yml
name: Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - uses: actions/setup-go@v2
        with:
          go-version: '1.24'
      
      # Option 1: Run generators
      - name: Generate code
        run: go generate ./...
      
      # Option 2: Verify generated code is up-to-date
      - name: Check generated code
        run: |
          go generate ./...
          git diff --exit-code
      
      - name: Run tests
        run: go test ./...
```

---

### Key Takeaways (Code Generation)

✅ **go generate automates boilerplate**
- //go:generate comments trigger code generation
- Run with: go generate ./...
- Useful for enums, mocks, protobuf, SQL

✅ **Common generators**
- stringer: String() methods
- mockgen: Mock implementations
- protoc: Protocol buffers
- sqlc: Type-safe SQL

✅ **Custom generators**
- Use text/template for simple cases
- Use go/parser and go/ast for complex cases
- Make generators idempotent

✅ **Best practices**
- Mark generated files clearly
- Usually commit generated code
- Separate generated from hand-written
- Document regeneration process


## 3.4 Unsafe Operations

The `unsafe` package allows operations that bypass Go's type safety. It should be used sparingly and only when absolutely necessary.

### What is unsafe?

```go
import "unsafe"

// unsafe.Pointer is a special pointer type that can:
// 1. Be converted to/from any pointer type
// 2. Be converted to/from uintptr
// 3. Bypass type safety
```

**Warning:** `unsafe` operations can:
- Crash your program
- Corrupt memory
- Create security vulnerabilities
- Break with Go version updates
- Make code non-portable

### Basic unsafe Operations

#### **Pointer Conversion**

```go
type MyInt int64

func main() {
    var i int64 = 42
    var p *int64 = &i
    
    // ❌ Not allowed: type mismatch
    // var mp *MyInt = p
    
    // ✅ With unsafe: convert pointer types
    var mp *MyInt = (*MyInt)(unsafe.Pointer(p))
    fmt.Println(*mp)  // 42
}
```

#### **Getting Variable Size**

```go
var i int
var s string
var slice []int

fmt.Println(unsafe.Sizeof(i))     // 8 (on 64-bit)
fmt.Println(unsafe.Sizeof(s))     // 16 (pointer + length)
fmt.Println(unsafe.Sizeof(slice)) // 24 (pointer + len + cap)
```

#### **Struct Field Offset**

```go
type Person struct {
    Name string
    Age  int
}

p := Person{Name: "Alice", Age: 30}

// Get offset of Age field
offset := unsafe.Offsetof(p.Age)
fmt.Println(offset)  // 16 (after Name field)

// Access field via pointer arithmetic
ptrToPerson := unsafe.Pointer(&p)
ptrToAge := (*int)(unsafe.Add(ptrToPerson, offset))
fmt.Println(*ptrToAge)  // 30
```

#### **Memory Layout**

```go
type Example struct {
    a bool   // 1 byte
    b int64  // 8 bytes
    c int16  // 2 bytes
}

e := Example{}

fmt.Printf("Total size: %d\n", unsafe.Sizeof(e))
fmt.Printf("Field a offset: %d\n", unsafe.Offsetof(e.a))
fmt.Printf("Field b offset: %d\n", unsafe.Offsetof(e.b))
fmt.Printf("Field c offset: %d\n", unsafe.Offsetof(e.c))

// Output:
// Total size: 24
// Field a offset: 0
// Field b offset: 8  (aligned)
// Field c offset: 16 (aligned)
```

### Common unsafe Patterns

#### **String to []byte Zero-Copy**

```go
// Normal way: copies data
s := "hello"
b := []byte(s)  // Allocates new slice and copies

// unsafe way: no copy (read-only!)
func stringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

// ⚠️ WARNING: Result is read-only!
// Modifying it is undefined behavior
```

**Go 1.20+ safer way:**

```go
import "unsafe"

func stringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

func bytesToString(b []byte) string {
    return unsafe.String(unsafe.SliceData(b), len(b))
}
```

#### **Accessing Private Fields (Reflection Alternative)**

```go
type locked struct {
    secret string
}

func getSecret(l *locked) string {
    // Get offset of secret field (requires knowing struct layout)
    offset := unsafe.Offsetof(l.secret)
    
    // Get pointer to struct
    ptr := unsafe.Pointer(l)
    
    // Calculate pointer to field
    fieldPtr := (*string)(unsafe.Add(ptr, offset))
    
    return *fieldPtr
}

l := locked{secret: "hidden"}
fmt.Println(getSecret(&l))  // "hidden"

// ⚠️ DANGEROUS: Breaks encapsulation
// Don't do this in production code!
```

#### **Fast Type Assertion**

```go
// Normal type assertion
func normalAssertion(i interface{}) int {
    if v, ok := i.(int); ok {
        return v
    }
    return 0
}

// unsafe version (faster but dangerous)
func unsafeAssertion(i interface{}) int {
    // Interface layout: (type, value)
    type iface struct {
        typ  unsafe.Pointer
        data unsafe.Pointer
    }
    
    ptr := (*iface)(unsafe.Pointer(&i))
    return *(*int)(ptr.data)
}

// ⚠️ WARNING: No type checking!
// Will crash or return garbage if wrong type
```

#### **Slice Header Manipulation**

```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}

// Create slice from pointer (dangerous!)
func pointerToSlice(ptr unsafe.Pointer, len, cap int) []byte {
    header := SliceHeader{
        Data: uintptr(ptr),
        Len:  len,
        Cap:  cap,
    }
    return *(*[]byte)(unsafe.Pointer(&header))
}

// ⚠️ Modern Go 1.17+: use unsafe.Slice instead
func pointerToSliceSafe(ptr *byte, len int) []byte {
    return unsafe.Slice(ptr, len)
}
```

### When unsafe is Justified

**✅ Valid use cases:**

1. **Interfacing with C code (cgo)**

```go
/*
#include <stdlib.h>
*/
import "C"
import "unsafe"

func allocateC() unsafe.Pointer {
    return C.malloc(C.size_t(1024))
}

func freeC(ptr unsafe.Pointer) {
    C.free(ptr)
}
```

2. **Performance-critical paths**

```go
// High-performance serialization
func writeInt64(b []byte, offset int, value int64) {
    *(*int64)(unsafe.Pointer(&b[offset])) = value
}
```

3. **Implementing sync primitives**

```go
// From sync/atomic package
func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) bool
```

4. **Memory-mapped I/O**

```go
func mmapFile(filename string) ([]byte, error) {
    // Map file into memory using syscalls
    // Returns slice backed by mmap'd region
}
```

**❌ Invalid use cases:**
- "Clever" hacks to avoid copying
- Working around type system
- Accessing private fields
- Anything that can be done safely

### unsafe Rules and Guidelines

**The Six Rules of unsafe.Pointer:**

1. **Any pointer can be converted to unsafe.Pointer**
```go
var i int = 42
p := unsafe.Pointer(&i)
```

2. **unsafe.Pointer can be converted to any pointer**
```go
var f *float64 = (*float64)(p)  // Dangerous!
```

3. **uintptr can be converted to unsafe.Pointer**
```go
addr := uintptr(p)
p2 := unsafe.Pointer(addr)
```

4. **unsafe.Pointer can be converted to uintptr**
```go
addr := uintptr(p)
```

5. **Arithmetic on uintptr**
```go
// Add offset
addr := uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.field)
p := unsafe.Pointer(addr)
```

6. **Reflect values and unsafe.Pointer**
```go
v := reflect.ValueOf(&x).Elem()
p := unsafe.Pointer(v.UnsafeAddr())
```

**Critical:** uintptr is just a number. GC doesn't track uintptr, so:

```go
// ❌ WRONG: GC might move object
addr := uintptr(unsafe.Pointer(&obj))
// ... time passes, GC runs
p := unsafe.Pointer(addr)  // Might point to wrong memory!

// ✅ CORRECT: Keep unsafe.Pointer
p := unsafe.Pointer(&obj)
// ... object won't move while pointer exists
```

### Performance Example

Comparing safe vs unsafe string conversion:

```go
func BenchmarkSafeConversion(b *testing.B) {
    s := strings.Repeat("hello", 1000)
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        bytes := []byte(s)  // Copies
        _ = bytes
    }
}

func BenchmarkUnsafeConversion(b *testing.B) {
    s := strings.Repeat("hello", 1000)
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        bytes := unsafe.Slice(unsafe.StringData(s), len(s))
        _ = bytes
    }
}

// Results:
// BenchmarkSafeConversion-8      1000000    1200 ns/op  5000 B/op  1 allocs/op
// BenchmarkUnsafeConversion-8   50000000      25 ns/op     0 B/op  0 allocs/op
//
// Unsafe is ~50x faster, but read-only!
```

### Testing with unsafe

```go
// Use build tags to enable/disable unsafe code
//go:build !nounsafe

package mypackage

import "unsafe"

func fastPath() {
    // unsafe implementation
}
```

```go
//go:build nounsafe

package mypackage

func fastPath() {
    // safe fallback implementation
}
```

**Test both versions:**
```bash
go test ./...              # With unsafe
go test -tags=nounsafe ./... # Without unsafe
```

### Frequently Asked Questions (unsafe)

**Q1: Is unsafe actually unsafe?**

**A:**

**Yes!** Using `unsafe` can:
- Crash your program (segfaults)
- Corrupt memory silently
- Create data races
- Break on different platforms
- Break with Go updates
- Create security vulnerabilities

**It's called "unsafe" for a reason.**

**When it's relatively safe:**
- Well-tested code
- Isolated in a library
- Has safe fallback
- Thoroughly documented
- Used by experts who understand memory layout

---

**Q2: Why does the standard library use unsafe?**

**A:**

Standard library uses `unsafe` for:
1. **Performance** - sync/atomic, runtime
2. **Interfacing with OS** - syscalls
3. **Implementing fundamentals** - reflect, runtime

**But even standard library:**
- Uses it sparingly
- Has extensive tests
- Written by Go experts
- Maintained carefully

**You probably don't have these same constraints.**

---

**Q3: Can unsafe code be made safe?**

**A:**

**Partially, with discipline:**

```go
// ✅ Encapsulate unsafe code
func SafeConvert(s string) []byte {
    // unsafe internally, but checked
    if len(s) == 0 {
        return nil
    }
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

// ✅ Provide safe alternative
func Convert(s string) []byte {
    if unsafeEnabled {
        return unsafeConvert(s)
    }
    return safeConvert(s)
}

// ✅ Document dangers
// WARNING: returned slice is read-only
// Writing to it causes undefined behavior

// ✅ Add runtime checks in debug mode
func UnsafeOperation() {
    if debugMode {
        // Perform safety checks
    }
    // unsafe code
}
```

---

### Key Takeaways (unsafe)

✅ **unsafe bypasses Go's safety guarantees**
- Can crash, corrupt memory, create security holes
- Use only when absolutely necessary
- Isolate and test thoroughly

✅ **Common uses**
- Interfacing with C code
- Performance-critical paths (after profiling!)
- Implementing low-level libraries
- Memory-mapped I/O

✅ **Rules to follow**
- Pointer conversions via unsafe.Pointer
- No arithmetic on unsafe.Pointer (use uintptr)
- Don't keep uintptr across GC points
- Test on all target platforms

✅ **Better alternatives**
- Reflection (slower but safe)
- Code generation
- Accept the copy overhead
- Redesign to avoid need

✅ **If you must use unsafe**
- Document thoroughly
- Provide safe fallback
- Test extensively
- Review carefully
- Expect to maintain

---

## 3.5 Assembly Integration

Go allows inline assembly for performance-critical code or platform-specific operations.

### When to Use Assembly

**✅ Valid reasons:**
- Implementing crypto primitives
- SIMD operations
- Atomic operations
- Specific CPU instructions
- Absolute maximum performance

**❌ Invalid reasons:**
- "It might be faster"
- Avoiding Go's abstractions
- Because you can

**Always profile first!**

### Go Assembly Basics

Go uses Plan 9 assembly syntax (different from x86 assembly):

```assembly
// add_amd64.s
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX    // Load first argument
    MOVQ b+8(FP), BX    // Load second argument
    ADDQ BX, AX         // Add them
    MOVQ AX, ret+16(FP) // Store result
    RET
```

```go
// add.go
func Add(a, b int64) int64
```

**How it works:**
1. Declare function in Go (no body)
2. Implement in assembly file
3. Linker connects them

### Simple Example

**sum_amd64.s:**

```assembly
#include "textflag.h"

// func Sum(nums []int64) int64
TEXT ·Sum(SB), NOSPLIT, $0-32
    MOVQ nums_base+0(FP), SI  // SI = pointer to array
    MOVQ nums_len+8(FP), CX   // CX = length
    XORQ AX, AX                // AX = 0 (accumulator)
    
    // If length is 0, return 0
    TESTQ CX, CX
    JZ done
    
loop:
    ADDQ (SI), AX    // Add current element
    ADDQ $8, SI      // Move to next element
    DECQ CX          // Decrement counter
    JNZ loop         // Jump if not zero
    
done:
    MOVQ AX, ret+24(FP)  // Store result
    RET
```

**sum.go:**

```go
//go:build amd64

package asm

func Sum(nums []int64) int64
```

**sum_generic.go:**

```go
//go:build !amd64

package asm

func Sum(nums []int64) int64 {
    var sum int64
    for _, n := range nums {
        sum += n
    }
    return sum
}
```

### SIMD Example

```assembly
// dot_product_amd64.s
#include "textflag.h"

// func DotProduct(a, b []float32) float32
TEXT ·DotProduct(SB), NOSPLIT, $0-52
    MOVQ a_base+0(FP), SI
    MOVQ b_base+24(FP), DI
    MOVQ a_len+8(FP), CX
    
    // Initialize XMM0 to zero
    XORPS X0, X0
    
    // Process 4 floats at a time with SSE
    SHRQ $2, CX  // CX /= 4
    JZ remainder
    
simd_loop:
    MOVUPS (SI), X1      // Load 4 floats from a
    MOVUPS (DI), X2      // Load 4 floats from b
    MULPS X2, X1         // Multiply
    ADDPS X1, X0         // Accumulate
    ADDQ $16, SI         // Advance pointers
    ADDQ $16, DI
    DECQ CX
    JNZ simd_loop
    
    // Horizontal sum of XMM0
    MOVHLPS X0, X1
    ADDPS X1, X0
    MOVSS X0, X1
    SHUFPS $1, X0, X0
    ADDSS X1, X0
    
remainder:
    // Handle remaining elements (< 4)
    // ... scalar code ...
    
    MOVSS X0, ret+48(FP)
    RET
```

### Calling Conventions

**Function prologue:**

```assembly
TEXT ·MyFunc(SB), NOSPLIT, $framesize-argsize
// framesize: local stack frame size
// argsize: arguments + return values size
```

**Accessing arguments:**

```assembly
// func Example(a int64, b int32) (int64, error)
TEXT ·Example(SB), NOSPLIT, $0-28
    MOVQ a+0(FP), AX      // First argument (8 bytes)
    MOVL b+8(FP), BX      // Second argument (4 bytes, but 8-byte aligned)
    MOVQ AX, ret1+16(FP)  // First return value
    MOVQ $0, ret2+24(FP)  // Second return value (interface is 16 bytes!)
    RET
```

**Important:** Go interfaces are 16 bytes (type pointer + data pointer)

### Build Tags for Assembly

```go
// sum_amd64.s - x86-64 implementation
//go:build amd64

// sum_arm64.s - ARM64 implementation  
//go:build arm64

// sum_generic.go - fallback implementation
//go:build !amd64 && !arm64
```

### Testing Assembly

```go
func TestSum(t *testing.T) {
    tests := []struct {
        name string
        nums []int64
        want int64
    }{
        {"empty", []int64{}, 0},
        {"single", []int64{42}, 42},
        {"multiple", []int64{1, 2, 3, 4, 5}, 15},
        {"negative", []int64{-1, -2, -3}, -6},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Sum(tt.nums)
            if got != tt.want {
                t.Errorf("Sum() = %v, want %v", got, tt.want)
            }
        })
    }
}

func BenchmarkSum(b *testing.B) {
    nums := make([]int64, 1000)
    for i := range nums {
        nums[i] = int64(i)
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        Sum(nums)
    }
}
```

### Real-World Examples

**From crypto/aes:**

```assembly
// aesenc_amd64.s
TEXT ·encryptBlockAsm(SB),NOSPLIT,$0
    MOVQ xk+0(FP), AX
    MOVQ dst+8(FP), DI
    MOVQ src+16(FP), SI
    
    MOVOU (SI), X0       // Load plaintext
    MOVOU (AX), X1       // Load round key
    PXOR X1, X0          // Initial XOR
    
    // ... AES rounds using AESENC instruction ...
    
    MOVOU X0, (DI)       // Store ciphertext
    RET
```

**From math:**

```assembly
// sqrt_amd64.s
TEXT ·Sqrt(SB),NOSPLIT,$0-16
    SQRTSD x+0(FP), X0
    MOVSD X0, ret+8(FP)
    RET
```

### Debugging Assembly

```bash
# Disassemble Go code
go build -gcflags='-S' > output.s

# See assembly in objdump
go build -o myapp
objdump -d myapp

# View generated assembly
go tool compile -S file.go
```

### Performance Comparison

```go
func BenchmarkGoSum(b *testing.B) {
    nums := make([]int64, 1000)
    for i := range nums {
        nums[i] = int64(i)
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        var sum int64
        for _, n := range nums {
            sum += n
        }
    }
}

func BenchmarkAsmSum(b *testing.B) {
    nums := make([]int64, 1000)
    for i := range nums {
        nums[i] = int64(i)
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        Sum(nums)
    }
}

// Results (example):
// BenchmarkGoSum-8     5000000    250 ns/op
// BenchmarkAsmSum-8   10000000    120 ns/op
//
// Assembly is 2x faster (but Go compiler keeps improving!)
```

### When NOT to Use Assembly

**Modern Go compiler is excellent:**

```go
// This Go code compiles to efficient assembly
func Add(a, b int64) int64 {
    return a + b
}

// Compiler generates:
// MOVQ a, AX
// ADDQ b, AX
// RET

// No need for hand-written assembly!
```

**Compiler optimizations:**
- Inlining
- Bounds check elimination
- Dead code elimination
- Register allocation
- SIMD autovectorization (improving)

**Unless you have specific needs (crypto, SIMD), let the compiler do its job.**

### Key Takeaways (Assembly)

✅ **Assembly for very specific cases**
- Crypto primitives
- SIMD operations
- Platform-specific features
- After profiling shows need

✅ **Plan 9 assembly syntax**
- Different from x86/ARM assembly
- Function arguments via frame pointer
- Must handle calling conventions

✅ **Provide fallbacks**
- Generic Go implementation
- Build tags for platforms
- Test all versions

✅ **Maintenance burden**
- Hard to read and maintain
- Breaks with architecture changes
- Requires assembly expertise
- Compiler keeps improving

✅ **Profile first**
- Don't assume assembly is faster
- Compiler optimizations are good
- Measure actual performance
- Consider maintainability cost


## Part 3 Summary

Congratulations! You've completed Part 3 of the Complete Go Mastery series. You now have expertise in Go's advanced features and know when and how to apply them responsibly.

### What You've Learned

**Reflection:**
- Runtime type inspection with reflect.Type and reflect.Value
- Struct field manipulation and tag reading
- Dynamic function calling
- Building generic libraries (JSON, ORMs, etc.)
- Performance implications (~100x slower than direct access)

**Generics (Go 1.18+):**
- Type parameters and constraints
- Generic functions and types
- Standard constraints (any, comparable, Ordered, etc.)
- Custom constraints and unions
- Generic data structures (Stack, Queue, Map, etc.)
- When to use generics vs interfaces

**Code Generation:**
- go generate for automation
- Common tools (stringer, mockgen, protoc, sqlc)
- Custom generators with templates
- go/parser and go/ast for analyzing code
- Build tags and conditional compilation

**Unsafe Operations:**
- unsafe.Pointer for type conversions
- Performance optimizations (zero-copy)
- Pointer arithmetic and memory layout
- When unsafe is justified (rarely!)
- Rules and guidelines for safe unsafe usage

**Assembly Integration:**
- Plan 9 assembly syntax
- Calling conventions and frame pointer
- SIMD operations for performance
- Platform-specific implementations
- When assembly is necessary (almost never!)

### Critical Principles

✅ **Use the right tool for the job**
- Start with simple, safe Go code
- Profile before optimizing
- Consider maintainability

✅ **Reflection: powerful but slow**
- Appropriate for generic libraries
- Not for hot paths
- Cache Type information when possible

✅ **Generics: type-safe code reuse**
- Better than reflection for performance
- Better than code duplication for maintainability
- Use for data structures and algorithms

✅ **Code generation: automate boilerplate**
- Reduces human error
- Keeps code DRY
- Document regeneration process

✅ **Unsafe and assembly: last resort**
- Profile first, optimize if needed
- Provide safe fallbacks
- Extensive testing required
- High maintenance cost

### Design Philosophy

**Simplicity over cleverness:**
```go
// ❌ Clever but hard to maintain
func Process(x interface{}) {
    v := reflect.ValueOf(x)
    // Complex reflection logic...
}

// ✅ Simple and clear
func ProcessUser(u User) {
    // Direct, type-safe operations
}
```

**Type safety over performance tricks:**
```go
// ❌ Unsafe for minor speedup
func Convert(s string) []byte {
    return *(*[]byte)(unsafe.Pointer(&s))
}

// ✅ Safe and clear
func Convert(s string) []byte {
    return []byte(s)
}
```

**Maintainability over micro-optimizations:**
```go
// ❌ Assembly for everything
func Add(a, b int) int  // Implemented in assembly

// ✅ Let compiler optimize
func Add(a, b int) int {
    return a + b  // Compiles to optimal code
}
```

### What's Next

In **Part 4: Testing, Benchmarking & Code Quality**, we'll explore:

- Unit testing strategies
- Table-driven tests
- Mocking and stubbing
- Integration testing
- Fuzz testing (Go 1.18+)
- Benchmarking techniques
- CPU and memory profiling
- Code coverage
- Static analysis tools
- Documentation best practices

### Practice Exercises

Before moving to Part 4, reinforce your learning:

1. **Build a generic cache** with LRU eviction
2. **Create a code generator** for builder patterns
3. **Implement a type-safe ORM** using reflection
4. **Write performance benchmarks** comparing generics vs reflection
5. **Explore go/ast** by building a code analyzer

### Additional Resources

- [The Laws of Reflection](https://go.dev/blog/laws-of-reflection) - Rob Pike
- [How to Use Go Interfaces](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)
- [Unsafe Go](https://www.cs.unh.edu/~cs659/unsafe.html)
- [Go Assembly Guide](https://go.dev/doc/asm)
- [Type Parameters Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md)

### Interview Preparation

**Common interview topics from Part 3:**

1. **Reflection:**
   - Explain reflect.Type vs reflect.Value
   - When to use reflection vs interfaces
   - Performance implications

2. **Generics:**
   - Implement generic data structures
   - Explain constraints
   - Compare with other languages' generics

3. **Code Generation:**
   - Benefits and drawbacks
   - When to generate code vs write manually
   - Common generators used

4. **Performance:**
   - Profile-driven optimization
   - Understanding when optimizations matter
   - Trade-offs between readability and speed

**Sample questions:**

- "Implement a generic binary search tree"
- "Use reflection to create a deep copy function"
- "Write a code generator for SQL queries"
- "Explain when you'd use unsafe.Pointer"
- "How does Go's generics implementation differ from Java/C++?"

---

**Ready to master Go testing and quality practices?** Continue to Part 4 where we'll dive deep into testing strategies, benchmarking, profiling, and maintaining high code quality!

---

## Appendix: Quick Reference

### Reflection Cheat Sheet

```go
// Get type and value
t := reflect.TypeOf(x)
v := reflect.ValueOf(x)

// Type information
t.Kind()           // reflect.Struct, reflect.Int, etc.
t.Name()           // Type name
t.NumField()       // Number of struct fields
t.Field(i)         // i-th field info

// Value operations
v.Interface()      // Convert back to interface{}
v.Int()            // Get int value
v.String()         // Get string value
v.CanSet()         // Can modify value?
v.Set(newValue)    // Modify value

// Struct fields
v.Field(i)         // i-th field value
v.FieldByName("X") // Field by name

// Call functions
v.Call(args []reflect.Value) []reflect.Value
```

### Generics Cheat Sheet

```go
// Function with type parameter
func Foo[T any](x T) T { return x }

// Multiple type parameters
func Pair[K comparable, V any](k K, v V) map[K]V

// Type constraints
func Max[T constraints.Ordered](a, b T) T

// Generic type
type Stack[T any] struct { items []T }

// Generic method
func (s *Stack[T]) Push(item T) { ... }

// Custom constraint
type Number interface {
    ~int | ~int64 | ~float64
}
```

### Unsafe Cheat Sheet

```go
// Pointer conversion
p := unsafe.Pointer(&x)
intPtr := (*int)(p)

// Size and offset
unsafe.Sizeof(x)
unsafe.Offsetof(s.field)
unsafe.Alignof(x)

// Slice from pointer (Go 1.17+)
unsafe.Slice(ptr, length)

// String from bytes (Go 1.20+)
unsafe.String(bytePtr, length)

// Rules:
// 1. Any pointer → unsafe.Pointer
// 2. unsafe.Pointer → any pointer
// 3. uintptr → unsafe.Pointer
// 4. unsafe.Pointer → uintptr
```

### Assembly Cheat Sheet

```assembly
// Function declaration
TEXT ·FuncName(SB), NOSPLIT, $framesize-argsize

// Load arguments
MOVQ arg+0(FP), AX

// Store return value
MOVQ AX, ret+8(FP)

// Return
RET

// Common registers
AX, BX, CX, DX    // General purpose
SI, DI            // Source/destination index
SP                // Stack pointer
FP                // Frame pointer (pseudo-register)
```

---

**End of Part 3**


## Bonus: Advanced Patterns in Production

### Builder Pattern with Generics

```go
// Generic builder for any struct type
type Builder[T any] struct {
    value T
}

func NewBuilder[T any]() *Builder[T] {
    return &Builder[T]{}
}

func (b *Builder[T]) Set(fn func(*T)) *Builder[T] {
    fn(&b.value)
    return b
}

func (b *Builder[T]) Build() T {
    return b.value
}

// Usage with any struct
type User struct {
    Name  string
    Email string
    Age   int
}

user := NewBuilder[User]().
    Set(func(u *User) { u.Name = "Alice" }).
    Set(func(u *User) { u.Email = "alice@example.com" }).
    Set(func(u *User) { u.Age = 30 }).
    Build()

// Or with dedicated builder methods
type UserBuilder struct {
    *Builder[User]
}

func NewUserBuilder() *UserBuilder {
    return &UserBuilder{NewBuilder[User]()}
}

func (b *UserBuilder) WithName(name string) *UserBuilder {
    b.Set(func(u *User) { u.Name = name })
    return b
}

func (b *UserBuilder) WithEmail(email string) *UserBuilder {
    b.Set(func(u *User) { u.Email = email })
    return b
}

func (b *UserBuilder) WithAge(age int) *UserBuilder {
    b.Set(func(u *User) { u.Age = age })
    return b
}

user = NewUserBuilder().
    WithName("Bob").
    WithEmail("bob@example.com").
    WithAge(25).
    Build()
```

### Functional Options with Generics

```go
type Option[T any] func(*T)

type Config struct {
    Host    string
    Port    int
    Timeout time.Duration
    MaxConn int
}

func WithHost[T any](host string) Option[T] {
    return func(c *T) {
        if cfg, ok := any(c).(*Config); ok {
            cfg.Host = host
        }
    }
}

// More type-safe version with constraints
type Configurable interface {
    SetHost(string)
    SetPort(int)
}

func WithHost2[T Configurable](host string) Option[T] {
    return func(c *T) {
        (*c).SetHost(host)
    }
}

// Apply options
func NewConfig(opts ...Option[Config]) *Config {
    cfg := &Config{
        Host:    "localhost",
        Port:    8080,
        Timeout: 30 * time.Second,
        MaxConn: 100,
    }
    
    for _, opt := range opts {
        opt(cfg)
    }
    
    return cfg
}
```

### Type-Safe Event System

```go
type EventHandler[T any] func(T)

type EventBus[T any] struct {
    handlers []EventHandler[T]
    mu       sync.RWMutex
}

func NewEventBus[T any]() *EventBus[T] {
    return &EventBus[T]{}
}

func (eb *EventBus[T]) Subscribe(handler EventHandler[T]) {
    eb.mu.Lock()
    defer eb.mu.Unlock()
    eb.handlers = append(eb.handlers, handler)
}

func (eb *EventBus[T]) Publish(event T) {
    eb.mu.RLock()
    handlers := make([]EventHandler[T], len(eb.handlers))
    copy(handlers, eb.handlers)
    eb.mu.RUnlock()
    
    for _, handler := range handlers {
        handler(event)
    }
}

// Usage
type UserCreatedEvent struct {
    UserID int
    Name   string
    Email  string
}

bus := NewEventBus[UserCreatedEvent]()

// Subscribe to events
bus.Subscribe(func(event UserCreatedEvent) {
    fmt.Printf("User created: %s\n", event.Name)
})

bus.Subscribe(func(event UserCreatedEvent) {
    sendWelcomeEmail(event.Email)
})

// Publish event
bus.Publish(UserCreatedEvent{
    UserID: 1,
    Name:   "Alice",
    Email:  "alice@example.com",
})
```

### Generic Repository Pattern

```go
type Entity interface {
    GetID() string
}

type Repository[T Entity] struct {
    items map[string]T
    mu    sync.RWMutex
}

func NewRepository[T Entity]() *Repository[T] {
    return &Repository[T]{
        items: make(map[string]T),
    }
}

func (r *Repository[T]) Create(item T) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    id := item.GetID()
    if _, exists := r.items[id]; exists {
        return fmt.Errorf("item with ID %s already exists", id)
    }
    
    r.items[id] = item
    return nil
}

func (r *Repository[T]) Get(id string) (T, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    item, ok := r.items[id]
    if !ok {
        var zero T
        return zero, fmt.Errorf("item with ID %s not found", id)
    }
    
    return item, nil
}

func (r *Repository[T]) Update(item T) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    id := item.GetID()
    if _, exists := r.items[id]; !exists {
        return fmt.Errorf("item with ID %s not found", id)
    }
    
    r.items[id] = item
    return nil
}

func (r *Repository[T]) Delete(id string) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    if _, exists := r.items[id]; !exists {
        return fmt.Errorf("item with ID %s not found", id)
    }
    
    delete(r.items, id)
    return nil
}

func (r *Repository[T]) List() []T {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    items := make([]T, 0, len(r.items))
    for _, item := range r.items {
        items = append(items, item)
    }
    
    return items
}

func (r *Repository[T]) FindBy(predicate func(T) bool) []T {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    var results []T
    for _, item := range r.items {
        if predicate(item) {
            results = append(results, item)
        }
    }
    
    return results
}

// Usage
type User struct {
    ID    string
    Name  string
    Email string
}

func (u User) GetID() string {
    return u.ID
}

repo := NewRepository[User]()

repo.Create(User{ID: "1", Name: "Alice", Email: "alice@example.com"})
repo.Create(User{ID: "2", Name: "Bob", Email: "bob@example.com"})

user, _ := repo.Get("1")
fmt.Println(user.Name)  // Alice

users := repo.FindBy(func(u User) bool {
    return strings.Contains(u.Email, "@example.com")
})
fmt.Printf("Found %d users\n", len(users))
```

### Reflection-Based Validator

```go
type Validator struct {
    validators map[string]func(interface{}) error
}

func NewValidator() *Validator {
    return &Validator{
        validators: make(map[string]func(interface{}) error),
    }
}

func (v *Validator) RegisterTag(tag string, fn func(interface{}) error) {
    v.validators[tag] = fn
}

func (v *Validator) Validate(obj interface{}) error {
    val := reflect.ValueOf(obj)
    typ := reflect.TypeOf(obj)
    
    if typ.Kind() == reflect.Ptr {
        val = val.Elem()
        typ = typ.Elem()
    }
    
    if typ.Kind() != reflect.Struct {
        return fmt.Errorf("expected struct, got %v", typ.Kind())
    }
    
    for i := 0; i < typ.NumField(); i++ {
        field := typ.Field(i)
        fieldVal := val.Field(i)
        
        // Check validate tag
        tag := field.Tag.Get("validate")
        if tag == "" || tag == "-" {
            continue
        }
        
        // Split multiple validations
        rules := strings.Split(tag, ",")
        for _, rule := range rules {
            parts := strings.SplitN(rule, "=", 2)
            ruleName := parts[0]
            
            validator, ok := v.validators[ruleName]
            if !ok {
                continue
            }
            
            if err := validator(fieldVal.Interface()); err != nil {
                return fmt.Errorf("field %s: %w", field.Name, err)
            }
        }
    }
    
    return nil
}

// Usage
validator := NewValidator()

validator.RegisterTag("required", func(val interface{}) error {
    v := reflect.ValueOf(val)
    if v.IsZero() {
        return errors.New("field is required")
    }
    return nil
})

validator.RegisterTag("email", func(val interface{}) error {
    str, ok := val.(string)
    if !ok {
        return errors.New("email validation requires string")
    }
    if !strings.Contains(str, "@") {
        return errors.New("invalid email format")
    }
    return nil
})

validator.RegisterTag("min", func(val interface{}) error {
    num, ok := val.(int)
    if !ok {
        return errors.New("min validation requires int")
    }
    if num < 18 {
        return errors.New("must be at least 18")
    }
    return nil
})

type User struct {
    Name  string `validate:"required"`
    Email string `validate:"required,email"`
    Age   int    `validate:"min=18"`
}

user := User{
    Name:  "",
    Email: "invalid",
    Age:   16,
}

if err := validator.Validate(user); err != nil {
    fmt.Println(err)  // field Name: field is required
}
```

### Generic State Machine

```go
type State interface {
    comparable
}

type StateMachine[S State] struct {
    current     S
    transitions map[S][]S
    mu          sync.RWMutex
}

func NewStateMachine[S State](initial S) *StateMachine[S] {
    return &StateMachine[S]{
        current:     initial,
        transitions: make(map[S][]S),
    }
}

func (sm *StateMachine[S]) AddTransition(from, to S) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    sm.transitions[from] = append(sm.transitions[from], to)
}

func (sm *StateMachine[S]) CanTransition(to S) bool {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    
    allowed := sm.transitions[sm.current]
    for _, s := range allowed {
        if s == to {
            return true
        }
    }
    return false
}

func (sm *StateMachine[S]) Transition(to S) error {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    if !sm.canTransitionLocked(to) {
        return fmt.Errorf("cannot transition from %v to %v", sm.current, to)
    }
    
    sm.current = to
    return nil
}

func (sm *StateMachine[S]) canTransitionLocked(to S) bool {
    allowed := sm.transitions[sm.current]
    for _, s := range allowed {
        if s == to {
            return true
        }
    }
    return false
}

func (sm *StateMachine[S]) Current() S {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    return sm.current
}

// Usage - Order states
type OrderState string

const (
    OrderPending    OrderState = "pending"
    OrderProcessing OrderState = "processing"
    OrderShipped    OrderState = "shipped"
    OrderDelivered  OrderState = "delivered"
    OrderCancelled  OrderState = "cancelled"
)

func main() {
    sm := NewStateMachine(OrderPending)
    
    // Define valid transitions
    sm.AddTransition(OrderPending, OrderProcessing)
    sm.AddTransition(OrderPending, OrderCancelled)
    sm.AddTransition(OrderProcessing, OrderShipped)
    sm.AddTransition(OrderProcessing, OrderCancelled)
    sm.AddTransition(OrderShipped, OrderDelivered)
    
    // Try transitions
    fmt.Println(sm.Current())  // pending
    
    sm.Transition(OrderProcessing)
    fmt.Println(sm.Current())  // processing
    
    err := sm.Transition(OrderPending)
    fmt.Println(err)  // Error: cannot transition
    
    sm.Transition(OrderShipped)
    sm.Transition(OrderDelivered)
    fmt.Println(sm.Current())  // delivered
}
```

### Advanced Reflection: Struct Mapper

```go
// MapStruct maps fields from src to dst using reflection
func MapStruct(dst, src interface{}) error {
    dstVal := reflect.ValueOf(dst)
    srcVal := reflect.ValueOf(src)
    
    if dstVal.Kind() != reflect.Ptr {
        return errors.New("dst must be a pointer")
    }
    
    dstVal = dstVal.Elem()
    
    if dstVal.Kind() != reflect.Struct {
        return errors.New("dst must be a pointer to struct")
    }
    
    if srcVal.Kind() == reflect.Ptr {
        srcVal = srcVal.Elem()
    }
    
    if srcVal.Kind() != reflect.Struct {
        return errors.New("src must be a struct")
    }
    
    dstType := dstVal.Type()
    
    for i := 0; i < dstType.NumField(); i++ {
        dstField := dstType.Field(i)
        dstFieldVal := dstVal.Field(i)
        
        if !dstFieldVal.CanSet() {
            continue
        }
        
        // Try to find matching field in source
        srcFieldVal := srcVal.FieldByName(dstField.Name)
        if !srcFieldVal.IsValid() {
            continue
        }
        
        // Check type compatibility
        if !srcFieldVal.Type().AssignableTo(dstField.Type) {
            // Try conversion
            if srcFieldVal.Type().ConvertibleTo(dstField.Type) {
                srcFieldVal = srcFieldVal.Convert(dstField.Type)
            } else {
                continue
            }
        }
        
        dstFieldVal.Set(srcFieldVal)
    }
    
    return nil
}

// Usage
type Source struct {
    ID        int
    Name      string
    Email     string
    Extra     string
    CreatedAt time.Time
}

type Destination struct {
    ID    int
    Name  string
    Email string
}

src := Source{
    ID:        1,
    Name:      "Alice",
    Email:     "alice@example.com",
    Extra:     "ignored",
    CreatedAt: time.Now(),
}

var dst Destination
MapStruct(&dst, src)

fmt.Printf("%+v\n", dst)
// {ID:1 Name:Alice Email:alice@example.com}
```

### Compile-Time Assertions

```go
// Use blank imports and type constraints to enforce compile-time checks

type MustImplementInterface interface {
    RequiredMethod()
}

type MyType struct{}

func (MyType) RequiredMethod() {}

// Compile-time check that MyType implements MustImplementInterface
var _ MustImplementInterface = MyType{}

// Generic compile-time constraint
func AssertImplements[T any, I any]() {
    var _ I = *new(T)
}

// Usage - will fail at compile time if User doesn't implement Validator
type Validator interface {
    Validate() error
}

func init() {
    AssertImplements[User, Validator]()
}

// Size assertions
const (
    _ = uint(unsafe.Sizeof(MyStruct{}) - 32)  // Compile error if size != 32
)
```

### Performance Monitoring with Generics

```go
type Metrics[T any] struct {
    mu     sync.RWMutex
    values []T
}

func NewMetrics[T any]() *Metrics[T] {
    return &Metrics[T]{
        values: make([]T, 0, 1000),
    }
}

func (m *Metrics[T]) Record(value T) {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    m.values = append(m.values, value)
    
    // Keep only last 1000 values
    if len(m.values) > 1000 {
        m.values = m.values[1:]
    }
}

func (m *Metrics[T]) Average() T where T constraints.Float | constraints.Integer {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    if len(m.values) == 0 {
        var zero T
        return zero
    }
    
    var sum T
    for _, v := range m.values {
        sum += v
    }
    
    return sum / T(len(m.values))
}

func (m *Metrics[T]) Max() T where T constraints.Ordered {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    if len(m.values) == 0 {
        var zero T
        return zero
    }
    
    max := m.values[0]
    for _, v := range m.values[1:] {
        if v > max {
            max = v
        }
    }
    
    return max
}

// Usage
latency := NewMetrics[time.Duration]()

// Record metrics
latency.Record(10 * time.Millisecond)
latency.Record(15 * time.Millisecond)
latency.Record(8 * time.Millisecond)

fmt.Printf("Average latency: %v\n", latency.Average())
fmt.Printf("Max latency: %v\n", latency.Max())
```

### Comprehensive Error Handling with Generics

```go
type Result[T any, E error] struct {
    value T
    err   E
}

func Ok[T any, E error](value T) Result[T, E] {
    return Result[T, E]{value: value}
}

func Err[T any, E error](err E) Result[T, E] {
    return Result[T, E]{err: err}
}

func (r Result[T, E]) IsOk() bool {
    return r.err == nil
}

func (r Result[T, E]) IsErr() bool {
    return r.err != nil
}

func (r Result[T, E]) Unwrap() (T, E) {
    return r.value, r.err
}

func (r Result[T, E]) Expect(msg string) T {
    if r.err != nil {
        panic(msg + ": " + r.err.Error())
    }
    return r.value
}

func (r Result[T, E]) UnwrapOr(defaultValue T) T {
    if r.err != nil {
        return defaultValue
    }
    return r.value
}

func (r Result[T, E]) UnwrapOrElse(fn func() T) T {
    if r.err != nil {
        return fn()
    }
    return r.value
}

func (r Result[T, E]) Map[U any](fn func(T) U) Result[U, E] {
    if r.err != nil {
        return Err[U, E](r.err)
    }
    return Ok[U, E](fn(r.value))
}

func (r Result[T, E]) AndThen[U any](fn func(T) Result[U, E]) Result[U, E] {
    if r.err != nil {
        return Err[U, E](r.err)
    }
    return fn(r.value)
}

// Usage
func divide(a, b float64) Result[float64, error] {
    if b == 0 {
        return Err[float64, error](errors.New("division by zero"))
    }
    return Ok[float64, error](a / b)
}

func sqrt(x float64) Result[float64, error] {
    if x < 0 {
        return Err[float64, error](errors.New("negative number"))
    }
    return Ok[float64, error](math.Sqrt(x))
}

// Chain operations
result := divide(10, 2).
    AndThen(func(x float64) Result[float64, error] {
        return sqrt(x)
    }).
    Map(func(x float64) float64 {
        return x * 2
    })

if result.IsOk() {
    value, _ := result.Unwrap()
    fmt.Println(value)
}
```

### Final Thoughts on Advanced Go

**The Pragmatic Approach:**

1. **Start Simple**
   - Use basic Go features first
   - Profile before optimizing
   - Measure impact of "advanced" features

2. **Progressive Enhancement**
   - Add generics when code duplication hurts
   - Use reflection when type safety isn't enough
   - Generate code when boilerplate is excessive
   - Reach for unsafe only when profiling proves necessary

3. **Maintainability First**
   - Code is read more than written
   - Future you (or your team) will thank you
   - Document why advanced features are used

4. **Testing is Critical**
   - Advanced features need comprehensive tests
   - Test edge cases thoroughly
   - Benchmark performance claims

**Remember:** The best code is code that works correctly, performs adequately, and can be understood and maintained by your team. Advanced features are tools, not goals.


## Comprehensive Design Patterns in Go

### Creational Patterns

#### **Singleton with sync.Once**

```go
type Database struct {
    conn *sql.DB
}

var (
    instance *Database
    once     sync.Once
)

func GetInstance() *Database {
    once.Do(func() {
        conn, err := sql.Open("mysql", "dsn")
        if err != nil {
            panic(err)
        }
        instance = &Database{conn: conn}
    })
    return instance
}
```

#### **Factory Method**

```go
type Storage interface {
    Save(key, value string) error
    Load(key string) (string, error)
}

type storageType string

const (
    MemoryStorage storageType = "memory"
    FileStorage   storageType = "file"
    S3Storage     storageType = "s3"
)

func NewStorage(t storageType) (Storage, error) {
    switch t {
    case MemoryStorage:
        return &MemoryStore{data: make(map[string]string)}, nil
    case FileStorage:
        return &FileStore{basePath: "/tmp"}, nil
    case S3Storage:
        return &S3Store{bucket: "my-bucket"}, nil
    default:
        return nil, fmt.Errorf("unknown storage type: %s", t)
    }
}
```

#### **Abstract Factory**

```go
type UIFactory interface {
    CreateButton() Button
    CreateInput() Input
}

type WindowsFactory struct{}

func (w WindowsFactory) CreateButton() Button {
    return &WindowsButton{}
}

func (w WindowsFactory) CreateInput() Input {
    return &WindowsInput{}
}

type MacFactory struct{}

func (m MacFactory) CreateButton() Button {
    return &MacButton{}
}

func (m MacFactory) CreateInput() Input {
    return &MacInput{}
}

func GetUIFactory(platform string) UIFactory {
    switch platform {
    case "windows":
        return WindowsFactory{}
    case "mac":
        return MacFactory{}
    default:
        return WindowsFactory{}
    }
}
```

#### **Builder Pattern**

```go
type Query struct {
    table   string
    fields  []string
    where   map[string]interface{}
    orderBy string
    limit   int
}

type QueryBuilder struct {
    query Query
}

func NewQueryBuilder(table string) *QueryBuilder {
    return &QueryBuilder{
        query: Query{
            table:  table,
            fields: []string{"*"},
            where:  make(map[string]interface{}),
        },
    }
}

func (b *QueryBuilder) Select(fields ...string) *QueryBuilder {
    b.query.fields = fields
    return b
}

func (b *QueryBuilder) Where(field string, value interface{}) *QueryBuilder {
    b.query.where[field] = value
    return b
}

func (b *QueryBuilder) OrderBy(field string) *QueryBuilder {
    b.query.orderBy = field
    return b
}

func (b *QueryBuilder) Limit(n int) *QueryBuilder {
    b.query.limit = n
    return b
}

func (b *QueryBuilder) Build() string {
    sql := fmt.Sprintf("SELECT %s FROM %s", 
        strings.Join(b.query.fields, ", "), 
        b.query.table)
    
    if len(b.query.where) > 0 {
        var conditions []string
        for k, v := range b.query.where {
            conditions = append(conditions, fmt.Sprintf("%s = %v", k, v))
        }
        sql += " WHERE " + strings.Join(conditions, " AND ")
    }
    
    if b.query.orderBy != "" {
        sql += " ORDER BY " + b.query.orderBy
    }
    
    if b.query.limit > 0 {
        sql += fmt.Sprintf(" LIMIT %d", b.query.limit)
    }
    
    return sql
}

// Usage
sql := NewQueryBuilder("users").
    Select("id", "name", "email").
    Where("age", 30).
    Where("active", true).
    OrderBy("created_at").
    Limit(10).
    Build()
```

#### **Object Pool**

```go
type Connection struct {
    ID     int
    InUse  bool
    client *http.Client
}

type ConnectionPool struct {
    mu          sync.Mutex
    connections []*Connection
    maxSize     int
    nextID      int
}

func NewConnectionPool(maxSize int) *ConnectionPool {
    return &ConnectionPool{
        connections: make([]*Connection, 0, maxSize),
        maxSize:     maxSize,
    }
}

func (p *ConnectionPool) Acquire() (*Connection, error) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    // Find available connection
    for _, conn := range p.connections {
        if !conn.InUse {
            conn.InUse = true
            return conn, nil
        }
    }
    
    // Create new if under limit
    if len(p.connections) < p.maxSize {
        conn := &Connection{
            ID:     p.nextID,
            InUse:  true,
            client: &http.Client{},
        }
        p.nextID++
        p.connections = append(p.connections, conn)
        return conn, nil
    }
    
    return nil, errors.New("connection pool exhausted")
}

func (p *ConnectionPool) Release(conn *Connection) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    conn.InUse = false
}
```

### Structural Patterns

#### **Adapter Pattern**

```go
// Legacy interface
type LegacyPrinter interface {
    PrintLegacy(s string)
}

type OldPrinter struct{}

func (p OldPrinter) PrintLegacy(s string) {
    fmt.Println("Legacy:", s)
}

// New interface
type ModernPrinter interface {
    Print(s string) error
}

// Adapter
type PrinterAdapter struct {
    legacy LegacyPrinter
}

func (a PrinterAdapter) Print(s string) error {
    a.legacy.PrintLegacy(s)
    return nil
}

// Usage
var modern ModernPrinter = PrinterAdapter{legacy: OldPrinter{}}
modern.Print("Hello")
```

#### **Decorator Pattern**

```go
type Component interface {
    Operation() string
}

type ConcreteComponent struct{}

func (c ConcreteComponent) Operation() string {
    return "Base"
}

type Decorator struct {
    component Component
}

func (d Decorator) Operation() string {
    return d.component.Operation()
}

type BoldDecorator struct {
    Decorator
}

func (b BoldDecorator) Operation() string {
    return "<b>" + b.component.Operation() + "</b>"
}

type ItalicDecorator struct {
    Decorator
}

func (i ItalicDecorator) Operation() string {
    return "<i>" + i.component.Operation() + "</i>"
}

// Usage
component := ConcreteComponent{}
bold := BoldDecorator{Decorator{component}}
italic := ItalicDecorator{Decorator{bold}}

fmt.Println(italic.Operation())
// Output: <i><b>Base</b></i>
```

#### **Facade Pattern**

```go
// Complex subsystems
type CPU struct{}
func (c CPU) Freeze() { fmt.Println("CPU freeze") }
func (c CPU) Jump(position int) { fmt.Println("CPU jump") }
func (c CPU) Execute() { fmt.Println("CPU execute") }

type Memory struct{}
func (m Memory) Load(position int, data []byte) { fmt.Println("Memory load") }

type HardDrive struct{}
func (h HardDrive) Read(lba int, size int) []byte {
    fmt.Println("HardDrive read")
    return []byte{}
}

// Facade
type ComputerFacade struct {
    cpu   CPU
    mem   Memory
    hd    HardDrive
}

func (c ComputerFacade) Start() {
    c.cpu.Freeze()
    c.mem.Load(0, c.hd.Read(0, 1024))
    c.cpu.Jump(0)
    c.cpu.Execute()
}

// Usage
computer := ComputerFacade{}
computer.Start()  // Simple interface to complex system
```

#### **Proxy Pattern**

```go
type Subject interface {
    Request() string
}

type RealSubject struct{}

func (r RealSubject) Request() string {
    return "Real Subject"
}

type Proxy struct {
    real   RealSubject
    cache  string
    cached bool
}

func (p *Proxy) Request() string {
    if p.cached {
        return "Cached: " + p.cache
    }
    
    p.cache = p.real.Request()
    p.cached = true
    return p.cache
}

// Usage
var subject Subject = &Proxy{}
fmt.Println(subject.Request())  // Real call
fmt.Println(subject.Request())  // Cached
```

### Behavioral Patterns

#### **Strategy Pattern**

```go
type SortStrategy interface {
    Sort([]int) []int
}

type BubbleSort struct{}

func (b BubbleSort) Sort(data []int) []int {
    // Bubble sort implementation
    result := make([]int, len(data))
    copy(result, data)
    // ... sorting logic
    return result
}

type QuickSort struct{}

func (q QuickSort) Sort(data []int) []int {
    // Quick sort implementation
    result := make([]int, len(data))
    copy(result, data)
    // ... sorting logic
    return result
}

type Sorter struct {
    strategy SortStrategy
}

func (s *Sorter) SetStrategy(strategy SortStrategy) {
    s.strategy = strategy
}

func (s *Sorter) Sort(data []int) []int {
    return s.strategy.Sort(data)
}

// Usage
sorter := Sorter{}
sorter.SetStrategy(BubbleSort{})
result := sorter.Sort([]int{5, 2, 8, 1})

sorter.SetStrategy(QuickSort{})
result = sorter.Sort([]int{5, 2, 8, 1})
```

#### **Observer Pattern**

```go
type Observer interface {
    Update(data interface{})
}

type Subject struct {
    observers []Observer
    state     interface{}
}

func (s *Subject) Attach(o Observer) {
    s.observers = append(s.observers, o)
}

func (s *Subject) Detach(o Observer) {
    for i, observer := range s.observers {
        if observer == o {
            s.observers = append(s.observers[:i], s.observers[i+1:]...)
            break
        }
    }
}

func (s *Subject) Notify() {
    for _, observer := range s.observers {
        observer.Update(s.state)
    }
}

func (s *Subject) SetState(state interface{}) {
    s.state = state
    s.Notify()
}

// Concrete observer
type ConcreteObserver struct {
    name string
}

func (o ConcreteObserver) Update(data interface{}) {
    fmt.Printf("%s received update: %v\n", o.name, data)
}

// Usage
subject := &Subject{}
observer1 := ConcreteObserver{name: "Observer1"}
observer2 := ConcreteObserver{name: "Observer2"}

subject.Attach(observer1)
subject.Attach(observer2)

subject.SetState("New State")
// Output:
// Observer1 received update: New State
// Observer2 received update: New State
```

#### **Command Pattern**

```go
type Command interface {
    Execute() error
    Undo() error
}

type Light struct {
    on bool
}

type LightOnCommand struct {
    light *Light
}

func (c LightOnCommand) Execute() error {
    c.light.on = true
    fmt.Println("Light is ON")
    return nil
}

func (c LightOnCommand) Undo() error {
    c.light.on = false
    fmt.Println("Light is OFF")
    return nil
}

type LightOffCommand struct {
    light *Light
}

func (c LightOffCommand) Execute() error {
    c.light.on = false
    fmt.Println("Light is OFF")
    return nil
}

func (c LightOffCommand) Undo() error {
    c.light.on = true
    fmt.Println("Light is ON")
    return nil
}

type RemoteControl struct {
    commands []Command
    history  []Command
}

func (r *RemoteControl) AddCommand(cmd Command) {
    r.commands = append(r.commands, cmd)
}

func (r *RemoteControl) ExecuteCommand(index int) error {
    if index >= len(r.commands) {
        return errors.New("invalid command index")
    }
    
    cmd := r.commands[index]
    err := cmd.Execute()
    if err == nil {
        r.history = append(r.history, cmd)
    }
    return err
}

func (r *RemoteControl) UndoLast() error {
    if len(r.history) == 0 {
        return errors.New("nothing to undo")
    }
    
    cmd := r.history[len(r.history)-1]
    r.history = r.history[:len(r.history)-1]
    
    return cmd.Undo()
}

// Usage
light := &Light{}
remote := &RemoteControl{}

remote.AddCommand(LightOnCommand{light: light})
remote.AddCommand(LightOffCommand{light: light})

remote.ExecuteCommand(0)  // Light ON
remote.UndoLast()         // Light OFF
```

#### **Chain of Responsibility**

```go
type Handler interface {
    SetNext(handler Handler)
    Handle(request string) string
}

type BaseHandler struct {
    next Handler
}

func (h *BaseHandler) SetNext(handler Handler) {
    h.next = handler
}

type AuthHandler struct {
    BaseHandler
}

func (h *AuthHandler) Handle(request string) string {
    if request == "unauthorized" {
        return "AuthHandler: Access denied"
    }
    
    if h.next != nil {
        return h.next.Handle(request)
    }
    
    return "AuthHandler: Passed"
}

type ValidationHandler struct {
    BaseHandler
}

func (h *ValidationHandler) Handle(request string) string {
    if request == "invalid" {
        return "ValidationHandler: Invalid request"
    }
    
    if h.next != nil {
        return h.next.Handle(request)
    }
    
    return "ValidationHandler: Passed"
}

type ProcessingHandler struct {
    BaseHandler
}

func (h *ProcessingHandler) Handle(request string) string {
    return "ProcessingHandler: Request processed"
}

// Usage
auth := &AuthHandler{}
validation := &ValidationHandler{}
processing := &ProcessingHandler{}

auth.SetNext(validation)
validation.SetNext(processing)

result := auth.Handle("valid")
fmt.Println(result)
// Output: ProcessingHandler: Request processed
```

### Concurrency Patterns

#### **Worker Pool Pattern**

```go
type Job func() error

type WorkerPool struct {
    workers   int
    jobs      chan Job
    results   chan error
    wg        sync.WaitGroup
}

func NewWorkerPool(workers int) *WorkerPool {
    return &WorkerPool{
        workers: workers,
        jobs:    make(chan Job, workers*2),
        results: make(chan error, workers*2),
    }
}

func (wp *WorkerPool) Start() {
    for i := 0; i < wp.workers; i++ {
        wp.wg.Add(1)
        go wp.worker()
    }
}

func (wp *WorkerPool) worker() {
    defer wp.wg.Done()
    
    for job := range wp.jobs {
        err := job()
        wp.results <- err
    }
}

func (wp *WorkerPool) Submit(job Job) {
    wp.jobs <- job
}

func (wp *WorkerPool) Stop() {
    close(wp.jobs)
    wp.wg.Wait()
    close(wp.results)
}

func (wp *WorkerPool) Results() <-chan error {
    return wp.results
}

// Usage
pool := NewWorkerPool(5)
pool.Start()

for i := 0; i < 10; i++ {
    i := i
    pool.Submit(func() error {
        fmt.Printf("Processing job %d\n", i)
        time.Sleep(time.Second)
        return nil
    })
}

pool.Stop()

for err := range pool.Results() {
    if err != nil {
        fmt.Println("Error:", err)
    }
}
```

#### **Pipeline Pattern**

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

func filter(in <-chan int, predicate func(int) bool) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if predicate(n) {
                out <- n
            }
        }
    }()
    return out
}

// Usage
nums := generator(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
squares := square(nums)
evens := filter(squares, func(n int) bool { return n%2 == 0 })

for n := range evens {
    fmt.Println(n)  // 4, 16, 36, 64, 100
}
```

### Best Practices Summary

**When to Use Each Pattern:**

| Pattern | Use Case |
|---------|----------|
| **Singleton** | Database connections, config |
| **Factory** | Object creation with logic |
| **Builder** | Complex object construction |
| **Adapter** | Interface compatibility |
| **Decorator** | Add functionality dynamically |
| **Facade** | Simplify complex subsystems |
| **Strategy** | Interchangeable algorithms |
| **Observer** | Event handling, pub-sub |
| **Command** | Action encapsulation, undo/redo |
| **Chain of Responsibility** | Request processing pipeline |

**Go-Specific Considerations:**

1. **Prefer Composition over Inheritance**
   - Use embedding instead of inheritance
   - Implement interfaces implicitly

2. **Use Interfaces Wisely**
   - Small, focused interfaces
   - Accept interfaces, return structs

3. **Leverage Concurrency**
   - Channels for communication
   - Goroutines for concurrent execution
   - sync package for shared state

4. **Keep It Simple**
   - Don't over-engineer
   - Use standard library patterns
   - Profile before optimizing

This completes our comprehensive tour of advanced Go patterns and techniques!

