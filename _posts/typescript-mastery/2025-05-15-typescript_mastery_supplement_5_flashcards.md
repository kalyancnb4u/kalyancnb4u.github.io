---
title: "TypeScript Mastery - Supplement 5: 200 TypeScript Interview Flashcards"
date: 2025-05-15 00:00:00 +0530
categories: [TypeScript, TS Mastery]
tags: [TypeScript, Programming, Resources, Interview, Flashcards, Questions]
---

# 200 TypeScript Interview Flashcards

> Quick review cards for TypeScript interview preparation

---

## 📋 How to Use These Flashcards

### **Study Methods**

**1. Traditional Method**
- Cover the answer
- Try to answer the question
- Check your response
- Mark cards you got wrong

**2. Spaced Repetition**
- Review daily cards
- Move mastered cards to weekly review
- Focus on challenging cards
- Track your progress

**3. Mock Interview Practice**
- Have someone ask you questions
- Answer out loud
- Explain your reasoning
- Practice under pressure

### **Difficulty Levels**
- ⭐ **Easy** - Fundamental concepts
- ⭐⭐ **Medium** - Practical application
- ⭐⭐⭐ **Hard** - Advanced concepts

---

## 🎯 Section 1: TypeScript Basics (40 Cards)

### **Card 1** ⭐
**Q:** What is TypeScript?

**A:** TypeScript is a strongly-typed superset of JavaScript that compiles to plain JavaScript. It adds optional static typing, classes, interfaces, and other features to JavaScript.

---

### **Card 2** ⭐
**Q:** What are the main benefits of using TypeScript?

**A:** 
1. Static type checking catches errors at compile time
2. Better IDE support with IntelliSense
3. Improved code maintainability
4. Better refactoring capabilities
5. Self-documenting code through types
6. Advanced OOP features

---

### **Card 3** ⭐
**Q:** What is the difference between `any` and `unknown`?

**A:** 
- `any`: Disables type checking completely; can be assigned to anything
- `unknown`: Type-safe alternative; requires type checking before use
- `unknown` forces you to verify the type before operations

```typescript
let a: any = 5
a.foo() // No error, dangerous

let u: unknown = 5
u.foo() // Error: must check type first
if (typeof u === 'number') {
  u.toFixed(2) // OK
}
```

---

### **Card 4** ⭐
**Q:** What is type inference in TypeScript?

**A:** Type inference is TypeScript's ability to automatically determine types based on values and context without explicit type annotations.

```typescript
let x = 5 // inferred as number
let arr = [1, 2, 3] // inferred as number[]
```

---

### **Card 5** ⭐
**Q:** What are the basic primitive types in TypeScript?

**A:** 
- `string` - text values
- `number` - numeric values (int and float)
- `boolean` - true/false
- `null` - intentional absence of value
- `undefined` - uninitialized value
- `symbol` - unique identifiers
- `bigint` - large integers

---

### **Card 6** ⭐
**Q:** What is the difference between `interface` and `type`?

**A:** 
**Interface:**
- Can be extended/implemented
- Declaration merging supported
- Only describes object shapes
- Better for OOP patterns

**Type:**
- Cannot be reopened
- Can represent any type (unions, primitives, etc.)
- More flexible
- Better for functional patterns

```typescript
// Interface
interface User {
  name: string
}
interface User { // Declaration merging
  age: number
}

// Type
type ID = string | number // Union type
```

---

### **Card 7** ⭐
**Q:** What are union types?

**A:** Union types allow a value to be one of several types, using the `|` operator.

```typescript
type StringOrNumber = string | number
let value: StringOrNumber = "hello"
value = 42 // Also valid
```

---

### **Card 8** ⭐
**Q:** What are intersection types?

**A:** Intersection types combine multiple types into one using the `&` operator.

```typescript
type Name = { name: string }
type Age = { age: number }
type Person = Name & Age

const person: Person = {
  name: "John",
  age: 30
}
```

---

### **Card 9** ⭐
**Q:** What is a tuple in TypeScript?

**A:** A tuple is an array with a fixed number of elements where each element has a known type.

```typescript
let tuple: [string, number] = ["hello", 42]
// Order and types matter
```

---

### **Card 10** ⭐
**Q:** What are literal types?

**A:** Literal types are exact values that a variable can have.

```typescript
type Direction = "up" | "down" | "left" | "right"
let dir: Direction = "up" // Only these 4 values allowed

type Zero = 0
let z: Zero = 0 // Only 0 is allowed
```

---

### **Card 11** ⭐
**Q:** What is the `void` type?

**A:** `void` represents the absence of a return value, typically used for functions that don't return anything.

```typescript
function log(message: string): void {
  console.log(message)
  // No return statement
}
```

---

### **Card 12** ⭐
**Q:** What is the `never` type?

**A:** `never` represents values that never occur. Used for functions that never return (throw errors or infinite loops).

```typescript
function throwError(message: string): never {
  throw new Error(message)
}

function infiniteLoop(): never {
  while (true) {}
}
```

---

### **Card 13** ⭐⭐
**Q:** What is type assertion and when should you use it?

**A:** Type assertion tells TypeScript to treat a value as a specific type. Use sparingly when you know more than TypeScript.

```typescript
// as syntax
let value = someValue as string

// Angle bracket syntax (not in JSX)
let value = <string>someValue

// Use when:
// - Working with DOM elements
// - Type inference is too general
// - External libraries
```

---

### **Card 14** ⭐⭐
**Q:** What is the non-null assertion operator?

**A:** The `!` operator tells TypeScript a value is not null/undefined.

```typescript
function getValue(): string | null {
  return "hello"
}

let value = getValue()!.toUpperCase() // Asserts not null
// Use carefully - runtime error if actually null
```

---

### **Card 15** ⭐⭐
**Q:** What are optional properties?

**A:** Properties that may or may not exist, marked with `?`.

```typescript
interface User {
  name: string
  age?: number // Optional
}

const user: User = { name: "John" } // Valid
```

---

### **Card 16** ⭐⭐
**Q:** What is the readonly modifier?

**A:** `readonly` prevents reassignment of properties after initialization.

```typescript
interface User {
  readonly id: string
  name: string
}

const user: User = { id: "123", name: "John" }
user.id = "456" // Error: cannot assign to readonly
user.name = "Jane" // OK
```

---

### **Card 17** ⭐⭐
**Q:** What are index signatures?

**A:** Index signatures allow objects to have any number of properties of a certain type.

```typescript
interface StringMap {
  [key: string]: string
}

const map: StringMap = {
  firstName: "John",
  lastName: "Doe",
  // Any string key with string value
}
```

---

### **Card 18** ⭐⭐
**Q:** What is the difference between `Array<T>` and `T[]`?

**A:** They are equivalent syntaxes for array types.

```typescript
let arr1: number[] = [1, 2, 3]
let arr2: Array<number> = [1, 2, 3]

// Both are the same
// T[] is more common
// Array<T> is more explicit with generics
```

---

### **Card 19** ⭐⭐
**Q:** What are enums in TypeScript?

**A:** Enums are a way to define a set of named constants.

```typescript
// Numeric enum
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

// String enum
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE"
}
```

---

### **Card 20** ⭐⭐
**Q:** What are function overloads?

**A:** Function overloads allow multiple function signatures for the same function.

```typescript
function process(x: string): string
function process(x: number): number
function process(x: string | number): string | number {
  if (typeof x === "string") {
    return x.toUpperCase()
  }
  return x * 2
}
```

---

### **Card 21** ⭐⭐
**Q:** What is the `this` type in TypeScript?

**A:** `this` type refers to the type of the current object.

```typescript
class Calculator {
  constructor(public value: number = 0) {}
  
  add(n: number): this {
    this.value += n
    return this // Returns Calculator instance
  }
  
  multiply(n: number): this {
    this.value *= n
    return this
  }
}

// Enables method chaining
new Calculator().add(5).multiply(2)
```

---

### **Card 22** ⭐
**Q:** What are access modifiers in TypeScript?

**A:** Access modifiers control property/method visibility:
- `public` - accessible everywhere (default)
- `private` - only within the class
- `protected` - within class and subclasses

```typescript
class User {
  public name: string
  private password: string
  protected role: string
}
```

---

### **Card 23** ⭐⭐
**Q:** What are abstract classes?

**A:** Abstract classes are base classes that cannot be instantiated and may contain abstract methods.

```typescript
abstract class Animal {
  abstract makeSound(): string // Must be implemented
  
  move(): void {
    console.log("Moving...")
  }
}

class Dog extends Animal {
  makeSound(): string {
    return "Woof!"
  }
}
```

---

### **Card 24** ⭐
**Q:** What is type narrowing?

**A:** Type narrowing is refining a union type to a more specific type.

```typescript
function print(value: string | number) {
  if (typeof value === "string") {
    // TypeScript knows value is string here
    console.log(value.toUpperCase())
  } else {
    // TypeScript knows value is number here
    console.log(value.toFixed(2))
  }
}
```

---

### **Card 25** ⭐⭐
**Q:** What is the `typeof` type operator?

**A:** `typeof` extracts the type from a value.

```typescript
const person = {
  name: "John",
  age: 30
}

type Person = typeof person
// { name: string; age: number }

const config = {
  apiUrl: "https://api.com",
  timeout: 5000
} as const

type Config = typeof config
```

---

### **Card 26** ⭐⭐
**Q:** What is the `keyof` type operator?

**A:** `keyof` creates a union type of all property keys.

```typescript
interface User {
  name: string
  age: number
  email: string
}

type UserKeys = keyof User
// "name" | "age" | "email"

function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key]
}
```

---

### **Card 27** ⭐⭐
**Q:** What are rest parameters in TypeScript?

**A:** Rest parameters allow functions to accept unlimited arguments as an array.

```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0)
}

sum(1, 2, 3, 4, 5) // 15
```

---

### **Card 28** ⭐⭐
**Q:** What is destructuring with types?

**A:** Destructuring with type annotations.

```typescript
interface Point {
  x: number
  y: number
}

function drawPoint({ x, y }: Point) {
  console.log(`x: ${x}, y: ${y}`)
}

const [first, second]: [number, string] = [1, "hello"]
```

---

### **Card 29** ⭐
**Q:** What is the difference between `==` and `===`?

**A:** While this is JavaScript, TypeScript encourages `===`:
- `==` - loose equality (type coercion)
- `===` - strict equality (no coercion)

TypeScript's `strict` mode helps catch these issues.

---

### **Card 30** ⭐⭐
**Q:** What are optional chaining and nullish coalescing?

**A:** 
**Optional chaining (`?.`)** - Safely access nested properties
**Nullish coalescing (`??`)** - Default value for null/undefined

```typescript
const user = { name: "John" }

// Optional chaining
const email = user?.contact?.email // undefined, no error

// Nullish coalescing
const name = user.name ?? "Guest"
const age = user.age ?? 0
```

---

### **Card 31** ⭐⭐
**Q:** What is a discriminated union?

**A:** A pattern using a common property to narrow union types.

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number }

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2
    case "square":
      return shape.size ** 2
  }
}
```

---

### **Card 32** ⭐⭐
**Q:** What are type guards?

**A:** Functions that perform runtime checks to narrow types.

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string"
}

function process(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase()) // TypeScript knows it's a string
  }
}
```

---

### **Card 33** ⭐⭐
**Q:** What is the `in` operator narrowing?

**A:** The `in` operator checks if a property exists and narrows the type.

```typescript
type Fish = { swim: () => void }
type Bird = { fly: () => void }

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim() // TypeScript knows it's Fish
  } else {
    animal.fly() // TypeScript knows it's Bird
  }
}
```

---

### **Card 34** ⭐
**Q:** What is the `instanceof` operator?

**A:** `instanceof` checks if an object is an instance of a class.

```typescript
class Dog {
  bark() { console.log("Woof!") }
}

function handle(animal: any) {
  if (animal instanceof Dog) {
    animal.bark() // TypeScript knows it's Dog
  }
}
```

---

### **Card 35** ⭐⭐
**Q:** What are assertion functions?

**A:** Functions that assert a condition is true and narrow types.

```typescript
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new Error(msg)
  }
}

function process(value: string | null) {
  assert(value !== null, "Value is null")
  // TypeScript knows value is string here
  console.log(value.toUpperCase())
}
```

---

### **Card 36** ⭐⭐
**Q:** What is type aliasing?

**A:** Creating a new name for a type.

```typescript
type ID = string | number
type User = {
  id: ID
  name: string
}

type UserList = User[]
```

---

### **Card 37** ⭐⭐
**Q:** What are getter and setter methods?

**A:** Special methods to control property access.

```typescript
class User {
  private _name: string = ""
  
  get name(): string {
    return this._name
  }
  
  set name(value: string) {
    if (value.length > 0) {
      this._name = value
    }
  }
}

const user = new User()
user.name = "John" // Calls setter
console.log(user.name) // Calls getter
```

---

### **Card 38** ⭐
**Q:** What is a class expression?

**A:** Defining a class as an expression.

```typescript
const MyClass = class {
  method() {
    return "Hello"
  }
}

const instance = new MyClass()
```

---

### **Card 39** ⭐⭐
**Q:** What are static members in classes?

**A:** Members that belong to the class itself, not instances.

```typescript
class MathUtil {
  static PI = 3.14159
  
  static square(x: number): number {
    return x * x
  }
}

console.log(MathUtil.PI)
console.log(MathUtil.square(5))
```

---

### **Card 40** ⭐
**Q:** What is method overriding?

**A:** Redefining a parent class method in a child class.

```typescript
class Animal {
  makeSound(): string {
    return "Some sound"
  }
}

class Dog extends Animal {
  makeSound(): string {
    return "Woof!"
  }
}
```

---

## 🚀 Section 2: Advanced Types (40 Cards)

### **Card 41** ⭐⭐⭐
**Q:** What are generics in TypeScript?

**A:** Generics allow creating reusable components that work with multiple types.

```typescript
function identity<T>(value: T): T {
  return value
}

identity<string>("hello")
identity<number>(42)

class Box<T> {
  constructor(public value: T) {}
}
```

---

### **Card 42** ⭐⭐⭐
**Q:** What are generic constraints?

**A:** Constraints limit which types can be used with generics.

```typescript
interface Lengthwise {
  length: number
}

function getLength<T extends Lengthwise>(item: T): number {
  return item.length
}

getLength("hello") // OK: string has length
getLength([1, 2, 3]) // OK: array has length
getLength(42) // Error: number doesn't have length
```

---

### **Card 43** ⭐⭐⭐
**Q:** What are conditional types?

**A:** Types that depend on a condition.

```typescript
type IsString<T> = T extends string ? true : false

type A = IsString<string> // true
type B = IsString<number> // false

// Extract return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never
```

---

### **Card 44** ⭐⭐⭐
**Q:** What is the `infer` keyword?

**A:** `infer` extracts types within conditional types.

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never

function getString(): string {
  return "hello"
}

type Result = ReturnType<typeof getString> // string

type UnpackArray<T> = T extends (infer U)[] ? U : T
type Str = UnpackArray<string[]> // string
```

---

### **Card 45** ⭐⭐⭐
**Q:** What are mapped types?

**A:** Types that transform properties of existing types.

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P]
}

type Optional<T> = {
  [P in keyof T]?: T[P]
}

interface User {
  name: string
  age: number
}

type ReadonlyUser = Readonly<User>
```

---

### **Card 46** ⭐⭐⭐
**Q:** What are template literal types?

**A:** Types created using template literal syntax.

```typescript
type EventName<T extends string> = `${T}Changed`

type UserEvents = EventName<"name" | "age">
// "nameChanged" | "ageChanged"

type HTTPMethod = "GET" | "POST"
type Endpoint = `/${string}`
type URL = `${HTTPMethod} ${Endpoint}`
// "GET /..." | "POST /..."
```

---

### **Card 47** ⭐⭐
**Q:** What is the `Partial<T>` utility type?

**A:** Makes all properties optional.

```typescript
interface User {
  name: string
  age: number
  email: string
}

type PartialUser = Partial<User>
// { name?: string; age?: number; email?: string }

function updateUser(user: User, updates: Partial<User>) {
  return { ...user, ...updates }
}
```

---

### **Card 48** ⭐⭐
**Q:** What is the `Required<T>` utility type?

**A:** Makes all properties required.

```typescript
interface User {
  name?: string
  age?: number
}

type RequiredUser = Required<User>
// { name: string; age: number }
```

---

### **Card 49** ⭐⭐
**Q:** What is the `Readonly<T>` utility type?

**A:** Makes all properties readonly.

```typescript
interface User {
  name: string
  age: number
}

type ReadonlyUser = Readonly<User>
// { readonly name: string; readonly age: number }
```

---

### **Card 50** ⭐⭐
**Q:** What is the `Pick<T, K>` utility type?

**A:** Selects specific properties from a type.

```typescript
interface User {
  id: string
  name: string
  age: number
  email: string
}

type UserPreview = Pick<User, "id" | "name">
// { id: string; name: string }
```

---

### **Card 51** ⭐⭐
**Q:** What is the `Omit<T, K>` utility type?

**A:** Excludes specific properties from a type.

```typescript
interface User {
  id: string
  name: string
  password: string
}

type PublicUser = Omit<User, "password">
// { id: string; name: string }
```

---

### **Card 52** ⭐⭐
**Q:** What is the `Record<K, T>` utility type?

**A:** Creates an object type with keys K and values T.

```typescript
type Roles = "admin" | "user" | "guest"

const permissions: Record<Roles, string[]> = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
}
```

---

### **Card 53** ⭐⭐
**Q:** What is the `Exclude<T, U>` utility type?

**A:** Removes types from a union.

```typescript
type AllTypes = string | number | boolean
type NoBoolean = Exclude<AllTypes, boolean>
// string | number
```

---

### **Card 54** ⭐⭐
**Q:** What is the `Extract<T, U>` utility type?

**A:** Extracts types from a union.

```typescript
type AllTypes = string | number | boolean
type OnlyString = Extract<AllTypes, string>
// string
```

---

### **Card 55** ⭐⭐
**Q:** What is the `NonNullable<T>` utility type?

**A:** Removes null and undefined from a type.

```typescript
type MaybeString = string | null | undefined
type DefinitelyString = NonNullable<MaybeString>
// string
```

---

### **Card 56** ⭐⭐⭐
**Q:** What is the `ReturnType<T>` utility type?

**A:** Extracts the return type of a function.

```typescript
function getUser() {
  return { name: "John", age: 30 }
}

type User = ReturnType<typeof getUser>
// { name: string; age: number }
```

---

### **Card 57** ⭐⭐⭐
**Q:** What is the `Parameters<T>` utility type?

**A:** Extracts parameter types as a tuple.

```typescript
function createUser(name: string, age: number) {
  return { name, age }
}

type Params = Parameters<typeof createUser>
// [string, number]
```

---

### **Card 58** ⭐⭐⭐
**Q:** What is the `InstanceType<T>` utility type?

**A:** Extracts the instance type of a constructor.

```typescript
class User {
  constructor(public name: string) {}
}

type UserInstance = InstanceType<typeof User>
// User
```

---

### **Card 59** ⭐⭐⭐
**Q:** What is the `Awaited<T>` utility type?

**A:** Unwraps Promise types recursively.

```typescript
type NestedPromise = Promise<Promise<string>>
type Result = Awaited<NestedPromise>
// string

async function getUser() {
  return { name: "John" }
}

type User = Awaited<ReturnType<typeof getUser>>
// { name: string }
```

---

### **Card 60** ⭐⭐⭐
**Q:** How do you create a custom mapped type?

**A:** Use the mapping syntax with `in keyof`.

```typescript
type Nullable<T> = {
  [P in keyof T]: T[P] | null
}

interface User {
  name: string
  age: number
}

type NullableUser = Nullable<User>
// { name: string | null; age: number | null }
```

---

### **Card 61** ⭐⭐⭐
**Q:** What are distributive conditional types?

**A:** Conditional types that distribute over union types.

```typescript
type ToArray<T> = T extends any ? T[] : never

type Result = ToArray<string | number>
// string[] | number[] (distributed)

// Non-distributive
type ToArray2<T> = [T] extends [any] ? T[] : never
type Result2 = ToArray2<string | number>
// (string | number)[]
```

---

### **Card 62** ⭐⭐⭐
**Q:** What is covariance and contravariance?

**A:** 
**Covariance**: Type can be more specific in output position
**Contravariance**: Type can be more general in input position

```typescript
// Covariance (return types)
type Producer<T> = () => T
let animalProducer: Producer<Animal>
let dogProducer: Producer<Dog>
animalProducer = dogProducer // OK (covariant)

// Contravariance (parameter types)
type Consumer<T> = (value: T) => void
let animalConsumer: Consumer<Animal>
let dogConsumer: Consumer<Dog>
dogConsumer = animalConsumer // OK (contravariant)
```

---

### **Card 63** ⭐⭐⭐
**Q:** What are recursive types?

**A:** Types that reference themselves.

```typescript
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue }

type LinkedList<T> = {
  value: T
  next: LinkedList<T> | null
}
```

---

### **Card 64** ⭐⭐⭐
**Q:** What is variance annotation in TypeScript?

**A:** TypeScript 4.7+ allows explicit variance with `in` and `out`.

```typescript
interface Producer<out T> {
  produce(): T
}

interface Consumer<in T> {
  consume(value: T): void
}

interface Both<in out T> {
  value: T
}
```

---

### **Card 65** ⭐⭐⭐
**Q:** How do you create a deep partial type?

**A:** Using recursive mapped types.

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

interface User {
  name: string
  address: {
    street: string
    city: string
  }
}

type PartialUser = DeepPartial<User>
// All properties and nested properties are optional
```

---

### **Card 66** ⭐⭐⭐
**Q:** What are template literal types with unions?

**A:** Combining template literals with union types.

```typescript
type Size = "small" | "medium" | "large"
type Color = "red" | "blue"

type Variant = `${Size}-${Color}`
// "small-red" | "small-blue" | "medium-red" | 
// "medium-blue" | "large-red" | "large-blue"
```

---

### **Card 67** ⭐⭐⭐
**Q:** What is the satisfies operator?

**A:** TypeScript 4.9+ operator for type checking without widening.

```typescript
const config = {
  apiUrl: "https://api.com",
  timeout: 5000
} satisfies Record<string, string | number>

// config keeps narrow types
config.apiUrl // type: "https://api.com" (not string)

// vs const assertion
const config2 = {
  apiUrl: "https://api.com" as const
} // Everything becomes readonly
```

---

### **Card 68** ⭐⭐⭐
**Q:** How do you create a type-safe event emitter?

**A:** Using mapped types and generics.

```typescript
type Events = {
  login: { userId: string }
  logout: { reason: string }
  message: { text: string; from: string }
}

class TypedEventEmitter<T extends Record<string, any>> {
  on<K extends keyof T>(event: K, handler: (data: T[K]) => void): void {
    // Implementation
  }
  
  emit<K extends keyof T>(event: K, data: T[K]): void {
    // Implementation
  }
}

const emitter = new TypedEventEmitter<Events>()
emitter.on("login", (data) => {
  console.log(data.userId) // Type-safe!
})
```

---

### **Card 69** ⭐⭐⭐
**Q:** What are branded types?

**A:** Types that add nominal typing to TypeScript.

```typescript
type Brand<K, T> = K & { __brand: T }

type USD = Brand<number, "USD">
type EUR = Brand<number, "EUR">

const usd = 100 as USD
const eur = 100 as EUR

function processUSD(amount: USD) {
  // Only accepts USD
}

processUSD(usd) // OK
processUSD(eur) // Error: type mismatch
```

---

### **Card 70** ⭐⭐⭐
**Q:** How do you create an exact type (no extra properties)?

**A:** Using a helper type.

```typescript
type Exact<T, Shape> = T extends Shape
  ? Exclude<keyof T, keyof Shape> extends never
    ? T
    : never
  : never

interface User {
  name: string
  age: number
}

function createUser<T extends User>(user: Exact<T, User>): T {
  return user
}

createUser({ name: "John", age: 30 }) // OK
createUser({ name: "John", age: 30, extra: true }) // Error
```

---

### **Card 71** ⭐⭐
**Q:** What is the `const` assertion?

**A:** Makes values deeply readonly and literal.

```typescript
const config = {
  apiUrl: "https://api.com",
  ports: [8080, 8081]
} as const

// All properties are readonly
// Types are literal: "https://api.com", not string
// Array becomes readonly tuple
```

---

### **Card 72** ⭐⭐⭐
**Q:** How do you make a type writable (remove readonly)?

**A:** Create a mapped type that removes readonly.

```typescript
type Writable<T> = {
  -readonly [P in keyof T]: T[P]
}

interface ReadonlyUser {
  readonly name: string
  readonly age: number
}

type WritableUser = Writable<ReadonlyUser>
// { name: string; age: number }
```

---

### **Card 73** ⭐⭐⭐
**Q:** What are higher-order types?

**A:** Types that take type parameters and return types.

```typescript
type Box<T> = { value: T }
type BoxArray<T> = Box<T>[]

type Transform<T, F extends (x: any) => any> = {
  [K in keyof T]: F extends (x: T[K]) => infer R ? R : never
}

type User = { name: string; age: number }
type Nullable = Transform<User, (x: any) => x | null>
// { name: string | null; age: number | null }
```

---

### **Card 74** ⭐⭐⭐
**Q:** How do you create a union from object values?

**A:** Use indexed access types.

```typescript
const Colors = {
  Red: "red",
  Blue: "blue",
  Green: "green"
} as const

type Color = typeof Colors[keyof typeof Colors]
// "red" | "blue" | "green"
```

---

### **Card 75** ⭐⭐⭐
**Q:** What is the builder pattern with TypeScript?

**A:** Creating fluent APIs with method chaining and types.

```typescript
class QueryBuilder<T extends object> {
  private conditions: Partial<T> = {}
  
  where<K extends keyof T>(key: K, value: T[K]): this {
    this.conditions[key] = value
    return this
  }
  
  build(): Partial<T> {
    return this.conditions
  }
}

const query = new QueryBuilder<User>()
  .where("name", "John")
  .where("age", 30)
  .build()
```

---

### **Card 76** ⭐⭐⭐
**Q:** How do you create a type from array values?

**A:** Use `typeof` with indexed access.

```typescript
const fruits = ["apple", "banana", "orange"] as const
type Fruit = typeof fruits[number]
// "apple" | "banana" | "orange"

const users = [
  { name: "John", age: 30 },
  { name: "Jane", age: 25 }
] as const

type User = typeof users[number]
// { readonly name: "John"; readonly age: 30 } | 
// { readonly name: "Jane"; readonly age: 25 }
```

---

### **Card 77** ⭐⭐⭐
**Q:** What are assertion signatures?

**A:** Function signatures that assert type conditions.

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Not a string")
  }
}

function process(value: unknown) {
  assertIsString(value)
  // TypeScript knows value is string here
  console.log(value.toUpperCase())
}
```

---

### **Card 78** ⭐⭐⭐
**Q:** How do you create conditional mapped types?

**A:** Combine conditional types with mapped types.

```typescript
type NullableStrings<T> = {
  [K in keyof T]: T[K] extends string ? T[K] | null : T[K]
}

interface User {
  name: string
  age: number
  email: string
}

type NullableUser = NullableStrings<User>
// { name: string | null; age: number; email: string | null }
```

---

### **Card 79** ⭐⭐⭐
**Q:** What is the `is` keyword in type predicates?

**A:** Defines custom type guard functions.

```typescript
interface Dog {
  bark(): void
}

interface Cat {
  meow(): void
}

function isDog(animal: Dog | Cat): animal is Dog {
  return "bark" in animal
}

function handle(animal: Dog | Cat) {
  if (isDog(animal)) {
    animal.bark() // TypeScript knows it's Dog
  } else {
    animal.meow() // TypeScript knows it's Cat
  }
}
```

---

### **Card 80** ⭐⭐⭐
**Q:** How do you create a type-safe path string?

**A:** Using template literals and recursion.

```typescript
type PathsToStringProps<T> = T extends string
  ? []
  : {
      [K in Extract<keyof T, string>]: [K, ...PathsToStringProps<T[K]>]
    }[Extract<keyof T, string>]

type Join<T extends string[], D extends string = "."> = 
  T extends [] ? never :
  T extends [infer F] ? F :
  T extends [infer F, ...infer R] ?
    F extends string ?
      R extends string[] ?
        `${F}${D}${Join<R, D>}` : never
    : never : string

type User = {
  name: string
  address: {
    street: string
    city: string
  }
}

type UserPaths = Join<PathsToStringProps<User>>
// "name" | "address" | "address.street" | "address.city"
```

---

## ⚛️ Section 3: React & Frontend (30 Cards)

### **Card 81** ⭐⭐
**Q:** How do you type a React functional component?

**A:** Use `React.FC` or direct typing.

```typescript
// Method 1: FC (deprecated pattern)
const Component: React.FC<Props> = ({ name }) => {
  return <div>{name}</div>
}

// Method 2: Direct (recommended)
interface Props {
  name: string
  age?: number
}

function Component({ name, age }: Props) {
  return <div>{name}</div>
}
```

---

### **Card 82** ⭐⭐
**Q:** How do you type useState?

**A:** TypeScript usually infers, but you can be explicit.

```typescript
// Inferred
const [count, setCount] = useState(0) // number

// Explicit
const [user, setUser] = useState<User | null>(null)

// Array
const [items, setItems] = useState<string[]>([])
```

---

### **Card 83** ⭐⭐
**Q:** How do you type useRef?

**A:** Different types for DOM elements vs. values.

```typescript
// DOM element
const inputRef = useRef<HTMLInputElement>(null)
inputRef.current?.focus()

// Mutable value
const countRef = useRef<number>(0)
countRef.current = 5

// Read-only value
const constRef = useRef<number>(0)
```

---

### **Card 84** ⭐⭐
**Q:** How do you type event handlers in React?

**A:** Use React event types.

```typescript
function Component() {
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log(e.currentTarget)
  }
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
  }
  
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
  }
  
  return <button onClick={handleClick}>Click</button>
}
```

---

### **Card 85** ⭐⭐
**Q:** How do you type children prop?

**A:** Use `React.ReactNode`.

```typescript
interface Props {
  children: React.ReactNode
}

function Container({ children }: Props) {
  return <div>{children}</div>
}

// More specific
interface LayoutProps {
  header: React.ReactElement
  sidebar: React.ReactElement
  children: React.ReactNode
}
```

---

### **Card 86** ⭐⭐
**Q:** How do you type useContext?

**A:** Create context with explicit type.

```typescript
interface ThemeContextType {
  theme: "light" | "dark"
  toggleTheme: () => void
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error("useTheme must be used within ThemeProvider")
  }
  return context
}
```

---

### **Card 87** ⭐⭐
**Q:** How do you type useReducer?

**A:** Define state and action types.

```typescript
type State = {
  count: number
  error: string | null
}

type Action =
  | { type: "increment" }
  | { type: "decrement" }
  | { type: "set"; payload: number }
  | { type: "error"; payload: string }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 }
    case "set":
      return { ...state, count: action.payload }
    default:
      return state
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0, error: null })
```

---

### **Card 88** ⭐⭐
**Q:** How do you type custom hooks?

**A:** Define return type explicitly.

```typescript
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    const item = localStorage.getItem(key)
    return item ? JSON.parse(item) : initialValue
  })
  
  const setValue = (value: T) => {
    setStoredValue(value)
    localStorage.setItem(key, JSON.stringify(value))
  }
  
  return [storedValue, setValue]
}
```

---

### **Card 89** ⭐⭐
**Q:** How do you type component props with generics?

**A:** Create generic components.

```typescript
interface ListProps<T> {
  items: T[]
  renderItem: (item: T) => React.ReactNode
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map(renderItem)}</ul>
}

// Usage
<List<User>
  items={users}
  renderItem={(user) => <li>{user.name}</li>}
/>
```

---

### **Card 90** ⭐⭐⭐
**Q:** How do you type forwardRef?

**A:** Use generic types for ref and props.

```typescript
interface InputProps {
  label: string
  type?: string
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, type = "text" }, ref) => {
    return (
      <div>
        <label>{label}</label>
        <input ref={ref} type={type} />
      </div>
    )
  }
)
```

---

### **Card 91** ⭐⭐
**Q:** How do you type HOCs (Higher-Order Components)?

**A:** Define input and output props.

```typescript
interface InjectedProps {
  isAuthenticated: boolean
}

function withAuth<P extends InjectedProps>(
  Component: React.ComponentType<P>
) {
  return function WithAuthComponent(
    props: Omit<P, keyof InjectedProps>
  ) {
    const isAuthenticated = true // From context/hook
    
    return <Component {...props as P} isAuthenticated={isAuthenticated} />
  }
}

// Usage
interface Props extends InjectedProps {
  name: string
}

const MyComponent = ({ name, isAuthenticated }: Props) => {
  return <div>{name}</div>
}

const Enhanced = withAuth(MyComponent)
```

---

### **Card 92** ⭐⭐
**Q:** How do you type render props pattern?

**A:** Define the render function signature.

```typescript
interface MouseTrackerProps {
  render: (mouse: { x: number; y: number }) => React.ReactNode
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 })
  
  return <div onMouseMove={(e) => setPosition({ x: e.clientX, y: e.clientY })}>
    {render(position)}
  </div>
}

// Usage
<MouseTracker
  render={({ x, y }) => <p>Position: {x}, {y}</p>}
/>
```

---

### **Card 93** ⭐⭐
**Q:** How do you type React Router components?

**A:** Use types from react-router-dom.

```typescript
import { useParams, useNavigate } from 'react-router-dom'

interface RouteParams {
  id: string
}

function UserDetail() {
  const { id } = useParams<RouteParams>()
  const navigate = useNavigate()
  
  return <div>User {id}</div>
}
```

---

### **Card 94** ⭐⭐
**Q:** How do you type form handling?

**A:** Type form values and handlers.

```typescript
interface FormData {
  email: string
  password: string
  remember: boolean
}

function LoginForm() {
  const [formData, setFormData] = useState<FormData>({
    email: "",
    password: "",
    remember: false
  })
  
  const handleChange = <K extends keyof FormData>(
    field: K,
    value: FormData[K]
  ) => {
    setFormData(prev => ({ ...prev, [field]: value }))
  }
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    // Submit formData
  }
  
  return <form onSubmit={handleSubmit}>...</form>
}
```

---

### **Card 95** ⭐⭐
**Q:** How do you type CSS-in-JS with styled-components?

**A:** Use props for dynamic styling.

```typescript
import styled from 'styled-components'

interface ButtonProps {
  primary?: boolean
  size?: "small" | "medium" | "large"
}

const Button = styled.button<ButtonProps>`
  background: ${props => props.primary ? "blue" : "gray"};
  padding: ${props => {
    switch (props.size) {
      case "small": return "5px 10px"
      case "large": return "15px 30px"
      default: return "10px 20px"
    }
  }};
`
```

---

### **Card 96** ⭐⭐
**Q:** How do you type async data fetching?

**A:** Type loading states and data.

```typescript
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error }

function useAsync<T>(asyncFn: () => Promise<T>) {
  const [state, setState] = useState<AsyncState<T>>({ status: "idle" })
  
  const execute = async () => {
    setState({ status: "loading" })
    try {
      const data = await asyncFn()
      setState({ status: "success", data })
    } catch (error) {
      setState({ status: "error", error: error as Error })
    }
  }
  
  return { state, execute }
}
```

---

### **Card 97** ⭐⭐
**Q:** How do you type compound components?

**A:** Use dot notation with proper types.

```typescript
interface TabsProps {
  children: React.ReactNode
  defaultTab?: string
}

interface TabProps {
  label: string
  children: React.ReactNode
}

function Tabs({ children, defaultTab }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab)
  return <div>{children}</div>
}

function Tab({ children }: TabProps) {
  return <div>{children}</div>
}

Tabs.Tab = Tab

// Usage
<Tabs defaultTab="first">
  <Tabs.Tab label="First">Content 1</Tabs.Tab>
  <Tabs.Tab label="Second">Content 2</Tabs.Tab>
</Tabs>
```

---

### **Card 98** ⭐⭐⭐
**Q:** How do you type React Query hooks?

**A:** Use generics for data and error types.

```typescript
import { useQuery, useMutation } from '@tanstack/react-query'

interface User {
  id: string
  name: string
}

function useUser(id: string) {
  return useQuery<User, Error>({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id)
  })
}

function useCreateUser() {
  return useMutation<User, Error, { name: string }>({
    mutationFn: (data) => createUser(data)
  })
}
```

---

### **Card 99** ⭐⭐
**Q:** How do you type portal components?

**A:** Use ReactDOM.createPortal with types.

```typescript
import { createPortal } from 'react-dom'

interface ModalProps {
  isOpen: boolean
  onClose: () => void
  children: React.ReactNode
}

function Modal({ isOpen, onClose, children }: ModalProps) {
  if (!isOpen) return null
  
  return createPortal(
    <div onClick={onClose}>
      <div onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body
  )
}
```

---

### **Card 100** ⭐⭐
**Q:** How do you type lazy-loaded components?

**A:** Use React.lazy with dynamic imports.

```typescript
import { lazy, Suspense } from 'react'

const LazyComponent = lazy(() => import('./HeavyComponent'))

// With props
interface Props {
  userId: string
}

const LazyUserProfile = lazy<React.ComponentType<Props>>(
  () => import('./UserProfile')
)

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyUserProfile userId="123" />
    </Suspense>
  )
}
```

---

### **Card 101** ⭐⭐⭐
**Q:** How do you create a type-safe form library?

**A:** Use path strings and generics.

```typescript
type PathValue<T, P extends string> = 
  P extends keyof T ? T[P] :
  P extends `${infer K}.${infer R}` ?
    K extends keyof T ? PathValue<T[K], R> : never
  : never

interface FormProps<T> {
  initialValues: T
  onSubmit: (values: T) => void
}

function useForm<T extends Record<string, any>>({ initialValues }: FormProps<T>) {
  const [values, setValues] = useState<T>(initialValues)
  
  const register = <P extends string>(name: P) => ({
    value: values[name],
    onChange: (e: React.ChangeEvent<HTMLInputElement>) => {
      setValues(prev => ({ ...prev, [name]: e.target.value }))
    }
  })
  
  return { register, values }
}
```

---

### **Card 102** ⭐⭐
**Q:** How do you type ErrorBoundary?

**A:** Use class component with error state.

```typescript
interface Props {
  fallback: React.ReactNode
  children: React.ReactNode
}

interface State {
  hasError: boolean
  error: Error | null
}

class ErrorBoundary extends React.Component<Props, State> {
  state: State = {
    hasError: false,
    error: null
  }
  
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("Error caught:", error, errorInfo)
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback
    }
    
    return this.props.children
  }
}
```

---

### **Card 103** ⭐⭐
**Q:** How do you type memo components?

**A:** Use React.memo with comparison function.

```typescript
interface Props {
  name: string
  age: number
}

const MemoizedComponent = React.memo<Props>(
  ({ name, age }) => {
    return <div>{name} - {age}</div>
  },
  (prevProps, nextProps) => {
    return prevProps.name === nextProps.name &&
           prevProps.age === nextProps.age
  }
)
```

---

### **Card 104** ⭐⭐⭐
**Q:** How do you type a polymorphic component (as prop)?

**A:** Use generic component with element type.

```typescript
type AsProp<C extends React.ElementType> = {
  as?: C
}

type PropsToOmit<C extends React.ElementType, P> = keyof (AsProp<C> & P)

type PolymorphicComponentProp<
  C extends React.ElementType,
  Props = {}
> = React.PropsWithChildren<Props & AsProp<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>

interface TextProps {
  color?: string
}

function Text<C extends React.ElementType = "span">({
  as,
  color,
  children,
  ...restProps
}: PolymorphicComponentProp<C, TextProps>) {
  const Component = as || "span"
  return <Component style={{ color }} {...restProps}>{children}</Component>
}

// Usage
<Text as="h1" color="red">Title</Text>
<Text as="a" href="/link">Link</Text>
```

---

### **Card 105** ⭐⭐
**Q:** How do you type Zustand stores?

**A:** Define store interface and actions.

```typescript
import create from 'zustand'

interface User {
  id: string
  name: string
}

interface UserStore {
  user: User | null
  setUser: (user: User) => void
  clearUser: () => void
}

const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null })
}))
```

---

### **Card 106** ⭐⭐
**Q:** How do you type Redux with TypeScript?

**A:** Use typed hooks and actions.

```typescript
import { configureStore } from '@reduxjs/toolkit'
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux'

const store = configureStore({
  reducer: {
    user: userReducer
  }
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch

export const useAppDispatch = () => useDispatch<AppDispatch>()
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector
```

---

### **Card 107** ⭐⭐
**Q:** How do you type Next.js pages?

**A:** Use Next.js types for props and context.

```typescript
import { GetServerSideProps, GetStaticProps } from 'next'

interface Props {
  user: User
}

export const getServerSideProps: GetServerSideProps<Props> = async (context) => {
  const user = await fetchUser(context.params?.id as string)
  
  return {
    props: { user }
  }
}

export default function UserPage({ user }: Props) {
  return <div>{user.name}</div>
}
```

---

### **Card 108** ⭐⭐
**Q:** How do you type API routes in Next.js?

**A:** Use NextApiRequest and NextApiResponse.

```typescript
import type { NextApiRequest, NextApiResponse } from 'next'

type Data = {
  name: string
}

type Error = {
  error: string
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<Data | Error>
) {
  if (req.method === 'POST') {
    res.status(200).json({ name: 'John' })
  } else {
    res.status(405).json({ error: 'Method not allowed' })
  }
}
```

---

### **Card 109** ⭐⭐
**Q:** How do you type Vue 3 components with TypeScript?

**A:** Use defineComponent with setup function.

```typescript
import { defineComponent, ref, PropType } from 'vue'

interface User {
  name: string
  age: number
}

export default defineComponent({
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    }
  },
  setup(props) {
    const count = ref(0)
    
    const increment = () => {
      count.value++
    }
    
    return { count, increment }
  }
})
```

---

### **Card 110** ⭐⭐
**Q:** How do you type Angular components?

**A:** Use decorators with TypeScript classes.

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core'

interface User {
  name: string
  age: number
}

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html'
})
export class UserComponent {
  @Input() user!: User
  @Output() userUpdated = new EventEmitter<User>()
  
  updateUser(): void {
    this.userUpdated.emit(this.user)
  }
}
```

---

## 🔧 Section 4: Node.js & Backend (30 Cards)

### **Card 111** ⭐⭐
**Q:** How do you type Express middleware?

**A:** Use Express types for request, response, and next.

```typescript
import { Request, Response, NextFunction } from 'express'

function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization
  
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' })
  }
  
  next()
}
```

---

### **Card 112** ⭐⭐
**Q:** How do you extend Express Request type?

**A:** Use declaration merging.

```typescript
declare global {
  namespace Express {
    interface Request {
      user?: {
        id: string
        email: string
      }
    }
  }
}

// Now available in all middleware
app.use((req, res, next) => {
  req.user = { id: '123', email: 'user@example.com' }
  next()
})
```

---

### **Card 113** ⭐⭐
**Q:** How do you type API responses?

**A:** Create response type helpers.

```typescript
type ApiResponse<T> = {
  success: true
  data: T
} | {
  success: false
  error: string
}

async function getUser(id: string): Promise<ApiResponse<User>> {
  try {
    const user = await fetchUser(id)
    return { success: true, data: user }
  } catch (error) {
    return { success: false, error: error.message }
  }
}
```

---

### **Card 114** ⭐⭐
**Q:** How do you type database models with Prisma?

**A:** Prisma generates types automatically.

```typescript
import { PrismaClient, User, Post } from '@prisma/client'

const prisma = new PrismaClient()

async function getUser(id: string): Promise<User | null> {
  return prisma.user.findUnique({
    where: { id }
  })
}

// With relations
type UserWithPosts = User & {
  posts: Post[]
}

async function getUserWithPosts(id: string): Promise<UserWithPosts | null> {
  return prisma.user.findUnique({
    where: { id },
    include: { posts: true }
  })
}
```

---

### **Card 115** ⭐⭐
**Q:** How do you type TypeORM entities?

**A:** Use decorators on classes.

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm'

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id!: string
  
  @Column()
  name!: string
  
  @Column({ unique: true })
  email!: string
  
  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt!: Date
}
```

---

### **Card 116** ⭐⭐
**Q:** How do you type validation schemas with Zod?

**A:** Define schemas and infer types.

```typescript
import { z } from 'zod'

const userSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  age: z.number().min(18).optional()
})

type User = z.infer<typeof userSchema>

function validateUser(data: unknown): User {
  return userSchema.parse(data) // Throws if invalid
}
```

---

### **Card 117** ⭐⭐
**Q:** How do you type NestJS controllers?

**A:** Use decorators with DTOs.

```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common'

class CreateUserDto {
  name!: string
  email!: string
}

@Controller('users')
export class UsersController {
  @Get(':id')
  async getUser(@Param('id') id: string): Promise<User> {
    return this.usersService.findOne(id)
  }
  
  @Post()
  async createUser(@Body() createUserDto: CreateUserDto): Promise<User> {
    return this.usersService.create(createUserDto)
  }
}
```

---

### **Card 118** ⭐⭐
**Q:** How do you type GraphQL resolvers?

**A:** Use GraphQL types and resolvers interface.

```typescript
import { GraphQLResolveInfo } from 'graphql'

interface Context {
  user?: User
}

interface Resolvers {
  Query: {
    user: (
      parent: any,
      args: { id: string },
      context: Context,
      info: GraphQLResolveInfo
    ) => Promise<User>
  }
  Mutation: {
    createUser: (
      parent: any,
      args: { input: CreateUserInput },
      context: Context,
      info: GraphQLResolveInfo
    ) => Promise<User>
  }
}
```

---

### **Card 119** ⭐⭐
**Q:** How do you type Socket.io events?

**A:** Define event maps for type safety.

```typescript
interface ServerToClientEvents {
  message: (data: { user: string; text: string }) => void
  userJoined: (user: string) => void
}

interface ClientToServerEvents {
  sendMessage: (text: string) => void
  joinRoom: (room: string) => void
}

const io: Server<ClientToServerEvents, ServerToClientEvents> = 
  new Server(httpServer)

io.on('connection', (socket) => {
  socket.on('sendMessage', (text) => {
    socket.emit('message', { user: 'John', text })
  })
})
```

---

### **Card 120** ⭐⭐
**Q:** How do you type environment variables?

**A:** Create typed env config.

```typescript
interface Env {
  NODE_ENV: 'development' | 'production' | 'test'
  PORT: number
  DATABASE_URL: string
  JWT_SECRET: string
}

function getEnv(): Env {
  return {
    NODE_ENV: process.env.NODE_ENV as Env['NODE_ENV'],
    PORT: parseInt(process.env.PORT || '3000'),
    DATABASE_URL: process.env.DATABASE_URL!,
    JWT_SECRET: process.env.JWT_SECRET!
  }
}

export const env = getEnv()
```

---

*Due to length constraints, I'll provide the remaining card categories in summary form. Would you like me to continue with the remaining 80 cards (Cards 121-200) covering:*

- **Section 5: Testing (20 cards)** - Jest, testing patterns, mocks
- **Section 6: Build Tools & Configuration (20 cards)** - tsconfig, webpack, etc.
- **Section 7: Best Practices & Patterns (20 cards)** - DRY, SOLID, etc.
- **Section 8: Interview Scenarios (20 cards)** - Problem solving questions

---

## 🎯 Study Tips

### **Daily Practice**
- Review 10-20 cards per day
- Mix difficulty levels
- Focus on weak areas
- Practice coding examples

### **Weekly Review**
- Go through all cards once
- Mark difficult ones
- Create flashcard app
- Test with friends

### **Before Interview**
- Review all 200 cards
- Focus on starred (hard) ones
- Practice explaining out loud
- Time yourself

---

**Master these flashcards and ace your TypeScript interview!** 🚀

*Interview Flashcards v1.0 - Complete TypeScript Mastery Series*

---

**Note:** This document contains the first 120 cards. The remaining 80 cards cover Testing, Build Tools, Best Practices, and Interview Scenarios. Would you like me to continue with those sections?
