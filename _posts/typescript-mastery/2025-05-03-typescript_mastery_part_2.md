---
title: "TypeScript Mastery - Part 2: Advanced Type System & Generics"
date: 2025-05-03 00:00:00 +0530
categories: [TypeScript, TS Mastery]
tags: [TypeScript, JavaScript, Programming, Generics, Advanced-types, Conditional-types, Mapped-types, Type-system, Utility-types]
---

# Complete TypeScript Mastery Part 2: Advanced Type System & Generics

## Introduction

Welcome to Part 2 of the Complete TypeScript Mastery series! In Part 1, we built a solid foundation with TypeScript fundamentals, basic types, interfaces, and functions. Now we're ready to explore the truly powerful features that make TypeScript's type system one of the most sophisticated available.

In this part, we'll dive deep into:
- Generic programming with constraints and advanced patterns
- TypeScript's built-in utility types and how to create your own
- Conditional types for type-level programming
- Mapped types for transforming types
- Template literal types for string manipulation
- Advanced type guards and narrowing techniques
- Decorators and metadata reflection
- Type-level algorithms and patterns

These advanced features enable you to build highly type-safe, flexible, and maintainable applications. By the end of this part, you'll be able to create sophisticated type utilities and understand the type definitions in major libraries like React, Redux, and Express.

---

## 2.1 Generics Deep Dive

Generics are one of TypeScript's most powerful features, enabling you to write reusable, type-safe code that works with multiple types.

### What Are Generics and Why?

Generics allow you to write code that works with any type while maintaining type safety.

**Without Generics:**

```typescript
// Separate functions for each type
function identityString(value: string): string {
  return value;
}

function identityNumber(value: number): number {
  return value;
}

function identityBoolean(value: boolean): boolean {
  return value;
}

// Using any loses type safety
function identityAny(value: any): any {
  return value;
}

const result = identityAny("hello");
result.toFixed(2);  // No error, but crashes at runtime!
```

**With Generics:**

```typescript
// One function that works with all types
function identity<T>(value: T): T {
  return value;
}

const str = identity("hello");    // Type: string
const num = identity(42);         // Type: number
const bool = identity(true);      // Type: boolean

str.toUpperCase();  // ✅ OK: TypeScript knows it's a string
num.toFixed(2);     // ✅ OK: TypeScript knows it's a number
```

**Why Generics Matter:**

1. **Type Safety**: Maintain precise types throughout your code
2. **Reusability**: Write once, use with any type
3. **IntelliSense**: Better IDE autocomplete and suggestions
4. **Refactoring**: Safer code changes with compiler checks
5. **Self-Documentation**: Types document what the code does

### Generic Functions

#### Basic Generic Functions

```typescript
// Single type parameter
function first<T>(array: T[]): T | undefined {
  return array[0];
}

const firstNumber = first([1, 2, 3]);      // Type: number | undefined
const firstString = first(["a", "b", "c"]); // Type: string | undefined

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const p1 = pair("hello", 42);        // Type: [string, number]
const p2 = pair(true, "world");      // Type: [boolean, string]

// Generic with inference
function map<T, U>(array: T[], fn: (item: T) => U): U[] {
  return array.map(fn);
}

const numbers = [1, 2, 3, 4, 5];
const strings = map(numbers, n => n.toString());  // Type: string[]
const doubled = map(numbers, n => n * 2);         // Type: number[]
```

#### Type Parameter Naming Conventions

```typescript
// Common conventions:
// T = Type (generic type)
// K = Key (object key type)
// V = Value (value type)
// E = Element (array element type)
// R = Return type
// P = Props (component props)

function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]];
}

function groupBy<T, K extends string | number>(
  array: T[],
  getKey: (item: T) => K
): Record<K, T[]> {
  const result = {} as Record<K, T[]>;
  for (const item of array) {
    const key = getKey(item);
    if (!result[key]) {
      result[key] = [];
    }
    result[key].push(item);
  }
  return result;
}

interface User {
  id: number;
  name: string;
  role: string;
}

const users: User[] = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "admin" }
];

const grouped = groupBy(users, user => user.role);
// Type: Record<string, User[]>
// Result: { admin: [...], user: [...] }
```

### Generic Interfaces

```typescript
// Basic generic interface
interface Box<T> {
  value: T;
}

const stringBox: Box<string> = { value: "hello" };
const numberBox: Box<number> = { value: 42 };

// Multiple type parameters
interface KeyValuePair<K, V> {
  key: K;
  value: V;
}

const pair1: KeyValuePair<string, number> = {
  key: "age",
  value: 30
};

const pair2: KeyValuePair<number, User> = {
  key: 1,
  value: { id: 1, name: "Alice", role: "admin" }
};

// Generic interface with methods
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(item: T): Promise<T>;
  update(id: string, item: Partial<T>): Promise<T>;
  delete(id: string): Promise<boolean>;
}

// Implementing generic interface
class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> {
    // Implementation
    return null;
  }
  
  async findAll(): Promise<User[]> {
    // Implementation
    return [];
  }
  
  async create(item: User): Promise<User> {
    // Implementation
    return item;
  }
  
  async update(id: string, item: Partial<User>): Promise<User> {
    // Implementation
    return item as User;
  }
  
  async delete(id: string): Promise<boolean> {
    // Implementation
    return true;
  }
}
```

### Generic Classes

```typescript
// Basic generic class
class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
  
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  
  get size(): number {
    return this.items.length;
  }
  
  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.pop();  // Type: number | undefined

const stringStack = new Stack<string>();
stringStack.push("hello");
stringStack.push("world");

// Generic class with constraints
class DataStore<T extends { id: string }> {
  private data = new Map<string, T>();
  
  add(item: T): void {
    this.data.set(item.id, item);
  }
  
  get(id: string): T | undefined {
    return this.data.get(id);
  }
  
  getAll(): T[] {
    return Array.from(this.data.values());
  }
  
  remove(id: string): boolean {
    return this.data.delete(id);
  }
}

interface Product {
  id: string;
  name: string;
  price: number;
}

const productStore = new DataStore<Product>();
productStore.add({ id: "1", name: "Laptop", price: 999 });

// Generic class with static methods
class ArrayUtils<T> {
  static create<U>(length: number, value: U): U[] {
    return Array(length).fill(value);
  }
  
  static range(start: number, end: number): number[] {
    return Array.from({ length: end - start }, (_, i) => start + i);
  }
}

const numbers = ArrayUtils.create(5, 0);      // [0, 0, 0, 0, 0]
const strings = ArrayUtils.create(3, "hi");   // ["hi", "hi", "hi"]
const range = ArrayUtils.range(1, 10);        // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Generic Type Aliases

```typescript
// Basic generic type alias
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

function fetchUser(id: string): Result<User> {
  // Implementation
  return { success: true, data: { id: "1", name: "Alice", role: "admin" } };
}

const result = fetchUser("1");
if (result.success) {
  console.log(result.data.name);  // TypeScript knows data is User
} else {
  console.log(result.error);      // TypeScript knows error is string
}

// Generic type alias with multiple parameters
type Dictionary<K extends string | number, V> = {
  [key in K]: V;
};

type UserDictionary = Dictionary<string, User>;
type ScoreDictionary = Dictionary<number, number>;

// Generic type alias for function types
type Predicate<T> = (value: T) => boolean;
type Mapper<T, U> = (value: T) => U;
type Reducer<T, U> = (accumulator: U, current: T) => U;

const isEven: Predicate<number> = n => n % 2 === 0;
const toString: Mapper<number, string> = n => n.toString();
const sum: Reducer<number, number> = (acc, n) => acc + n;

// Recursive generic type
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};

const tree: TreeNode<string> = {
  value: "root",
  children: [
    {
      value: "child1",
      children: [
        { value: "grandchild1", children: [] },
        { value: "grandchild2", children: [] }
      ]
    },
    {
      value: "child2",
      children: []
    }
  ]
};
```

### Generic Constraints

Constraints limit what types can be used with generics using the `extends` keyword.

#### Basic Constraints

```typescript
// Constraint to specific type
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30, email: "alice@example.com" };

const name = getProperty(user, "name");    // Type: string
const age = getProperty(user, "age");      // Type: number
// const invalid = getProperty(user, "id"); // ❌ Error: "id" not in user

// Constraint to object with specific properties
function logLength<T extends { length: number }>(item: T): void {
  console.log(`Length: ${item.length}`);
}

logLength("hello");        // ✅ OK: string has length
logLength([1, 2, 3]);      // ✅ OK: array has length
logLength({ length: 10 }); // ✅ OK: object has length property
// logLength(42);          // ❌ Error: number doesn't have length

// Constraint to primitive types
function parseNumber<T extends string | number>(value: T): number {
  return typeof value === "string" ? parseFloat(value) : value;
}

parseNumber("42");   // ✅ OK
parseNumber(42);     // ✅ OK
// parseNumber(true); // ❌ Error: boolean not assignable
```

#### Constraining to Object Shapes

```typescript
// Constraint requiring specific properties
interface Identifiable {
  id: string;
}

function findById<T extends Identifiable>(
  items: T[],
  id: string
): T | undefined {
  return items.find(item => item.id === id);
}

interface User extends Identifiable {
  name: string;
  email: string;
}

interface Product extends Identifiable {
  name: string;
  price: number;
}

const users: User[] = [
  { id: "1", name: "Alice", email: "alice@example.com" },
  { id: "2", name: "Bob", email: "bob@example.com" }
];

const products: Product[] = [
  { id: "1", name: "Laptop", price: 999 },
  { id: "2", name: "Mouse", price: 29 }
];

const user = findById(users, "1");      // Type: User | undefined
const product = findById(products, "1"); // Type: Product | undefined
```

#### Using Type Parameters in Constraints

```typescript
// One type parameter constrains another
function assign<T extends object, U extends object>(
  target: T,
  source: U
): T & U {
  return { ...target, ...source };
}

const person = { name: "Alice" };
const details = { age: 30, city: "New York" };

const combined = assign(person, details);
// Type: { name: string } & { age: number; city: string }

// Constraint using another type parameter
function merge<T extends U, U>(target: T, source: U): T {
  return { ...target, ...source };
}

interface Base {
  id: string;
}

interface Extended extends Base {
  name: string;
  email: string;
}

const base: Base = { id: "1" };
const extended: Extended = { id: "1", name: "Alice", email: "alice@example.com" };

const result = merge(extended, base);  // ✅ OK: Extended extends Base
// const invalid = merge(base, extended); // ❌ Error: Base doesn't extend Extended
```

#### Constraint Inference

```typescript
// TypeScript infers constraints from usage
function firstElement<T extends any[]>(arr: T): T[0] {
  return arr[0];
}

const first = firstElement([1, 2, 3]);  // Type: number
const second = firstElement(["a", "b"]); // Type: string

// Constraint with conditional types
type Flatten<T> = T extends Array<infer U> ? U : T;

function flatten<T>(value: T): Flatten<T> {
  return Array.isArray(value) ? value[0] : value as Flatten<T>;
}

const num = flatten(42);        // Type: number
const str = flatten([1, 2, 3]); // Type: number
```

### Default Type Parameters

Provide default types when no type argument is specified.

```typescript
// Basic default type parameter
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

const strings = createArray(3, "hello");  // Type: string[]
const numbers = createArray<number>(3, 0); // Type: number[]
const defaults = createArray(3, "hi");     // Type: string[] (uses default)

// Default type parameter with constraint
interface Config<T extends object = { timeout: number }> {
  options: T;
}

const config1: Config = {
  options: { timeout: 5000 }  // Uses default type
};

const config2: Config<{ host: string; port: number }> = {
  options: { host: "localhost", port: 3000 }
};

// Multiple defaults
type AsyncResult<T = unknown, E = Error> = 
  | { status: "success"; data: T }
  | { status: "error"; error: E };

const result1: AsyncResult = {
  status: "success",
  data: "some data"  // Type: unknown
};

const result2: AsyncResult<User> = {
  status: "success",
  data: { id: "1", name: "Alice", role: "admin" }  // Type: User
};

const result3: AsyncResult<User, CustomError> = {
  status: "error",
  error: new CustomError("Failed")  // Type: CustomError
};
```

### Advanced Generic Patterns

#### Generic Type Inference

```typescript
// TypeScript infers generic types from arguments
function tuple<T extends any[]>(...args: T): T {
  return args;
}

const t1 = tuple(1, "hello", true);
// Type: [number, string, boolean]

const t2 = tuple(42);
// Type: [number]

// Inference with objects
function createPair<T>(value: T) {
  return { value, valueOf: () => value };
}

const pair = createPair("hello");
// Type: { value: string; valueOf: () => string }

// Complex inference
function pipe<A, B, C>(
  value: A,
  fn1: (value: A) => B,
  fn2: (value: B) => C
): C {
  return fn2(fn1(value));
}

const result = pipe(
  5,
  n => n * 2,      // number => number
  n => n.toString() // number => string
);
// Type: string
```

#### Variadic Tuple Types

Work with tuples of any length:

```typescript
// Concatenate tuples
type Concat<T extends any[], U extends any[]> = [...T, ...U];

type Result1 = Concat<[1, 2], [3, 4]>;  // [1, 2, 3, 4]
type Result2 = Concat<[string], [number, boolean]>;  // [string, number, boolean]

// Function with variadic tuples
function concat<T extends any[], U extends any[]>(
  arr1: [...T],
  arr2: [...U]
): [...T, ...U] {
  return [...arr1, ...arr2];
}

const concatenated = concat([1, 2], ["a", "b"]);
// Type: [number, number, string, string]

// Prepend to tuple
type Prepend<T, U extends any[]> = [T, ...U];

type WithId = Prepend<string, [number, boolean]>;
// Type: [string, number, boolean]

// Extract tail
type Tail<T extends any[]> = T extends [any, ...infer Rest] ? Rest : never;

type Numbers = [1, 2, 3, 4];
type AfterFirst = Tail<Numbers>;  // [2, 3, 4]
```

#### Higher-Kinded Types (Workaround)

TypeScript doesn't support higher-kinded types directly, but we can simulate them:

```typescript
// Simulating higher-kinded types
interface Mappable<T> {
  map<U>(fn: (value: T) => U): Mappable<U>;
}

class Box<T> implements Mappable<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): Box<U> {
    return new Box(fn(this.value));
  }
  
  getValue(): T {
    return this.value;
  }
}

const box = new Box(5);
const mapped = box.map(n => n * 2).map(n => n.toString());
// Type: Box<string>

// Generic Functor pattern
interface Functor<A> {
  map<B>(fn: (a: A) => B): Functor<B>;
}

class Maybe<T> implements Functor<T> {
  constructor(private value: T | null) {}
  
  map<U>(fn: (value: T) => U): Maybe<U> {
    if (this.value === null) {
      return new Maybe<U>(null);
    }
    return new Maybe(fn(this.value));
  }
  
  getOrElse(defaultValue: T): T {
    return this.value === null ? defaultValue : this.value;
  }
}

const maybe = new Maybe(5);
const result = maybe
  .map(n => n * 2)
  .map(n => n.toString())
  .getOrElse("0");
```

#### Recursive Generics

```typescript
// Deep readonly type
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};

interface User {
  name: string;
  address: {
    street: string;
    city: string;
    country: {
      name: string;
      code: string;
    };
  };
}

const user: DeepReadonly<User> = {
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "New York",
    country: {
      name: "USA",
      code: "US"
    }
  }
};

// user.name = "Bob";  // ❌ Error: readonly
// user.address.street = "456 Oak Ave";  // ❌ Error: readonly
// user.address.country.code = "CA";  // ❌ Error: readonly (deep)

// Deep partial type
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? DeepPartial<T[P]>
    : T[P];
};

const partialUser: DeepPartial<User> = {
  name: "Alice",
  address: {
    city: "New York"
    // street and country are optional
  }
};

// Path type for nested object access
type Path<T> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? T[K] extends object
          ? K | `${K}.${Path<T[K]>}`
          : K
        : never;
    }[keyof T]
  : never;

type UserPath = Path<User>;
// Type: "name" | "address" | "address.street" | "address.city" | 
//       "address.country" | "address.country.name" | "address.country.code"
```

### Generic Contextual Typing

TypeScript uses context to infer generic types:

```typescript
// Array methods use contextual typing
const numbers = [1, 2, 3, 4, 5];

// TypeScript infers the parameter type
const doubled = numbers.map(n => n * 2);  // n is inferred as number

// Event handlers
document.addEventListener("click", (event) => {
  // event is inferred as MouseEvent
  console.log(event.clientX, event.clientY);
});

// Promise chains
fetch("/api/data")
  .then(response => response.json())  // response is Response
  .then(data => console.log(data));    // data is any

// Better with explicit types
interface ApiResponse {
  id: number;
  name: string;
}

fetch("/api/data")
  .then(response => response.json())
  .then((data: ApiResponse) => {
    console.log(data.name);  // data is ApiResponse
  });

// Generic promise wrapper
function fetchJson<T>(url: string): Promise<T> {
  return fetch(url).then(response => response.json());
}

const userData = await fetchJson<User>("/api/user");
// userData is typed as User
```

### Frequently Asked Questions

**Q1: What's the difference between generics and any?**

**A:** Generics maintain type information while `any` loses all type safety:

```typescript
// Using any - loses type safety
function identityAny(value: any): any {
  return value;
}

const result1 = identityAny("hello");
result1.toFixed(2);  // No error at compile time, crashes at runtime!

// Using generics - maintains type safety
function identity<T>(value: T): T {
  return value;
}

const result2 = identity("hello");  // Type: string
result2.toFixed(2);  // ❌ Error: toFixed doesn't exist on string
result2.toUpperCase();  // ✅ OK: toUpperCase exists on string
```

**Key differences:**
- `any` disables type checking completely
- Generics preserve and propagate type information
- Generics enable IntelliSense and autocomplete
- Generics catch errors at compile time

---

**Q2: When should I use generic constraints?**

**A:** Use constraints when your generic function needs to access specific properties or methods:

```typescript
// ❌ Without constraint - can't access properties
function logValue<T>(obj: T): void {
  console.log(obj.value);  // Error: Property 'value' doesn't exist on T
}

// ✅ With constraint - can access value property
function logValue<T extends { value: any }>(obj: T): void {
  console.log(obj.value);  // OK: T is constrained to have value
}

// Common use case: keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];  // OK: K is guaranteed to be a key of T
}
```

**Use constraints when:**
- Accessing properties or methods
- Requiring specific type characteristics
- Enforcing relationships between type parameters
- Limiting valid types for better error messages

---

**Q3: What are default type parameters and when should I use them?**

**A:** Default type parameters provide fallback types when none are specified:

```typescript
// Without default
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

createArray(3, "hi");  // Must infer or specify T

// With default
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

createArray(3, "hi");        // Uses inferred type: string
createArray<number>(3, 0);   // Uses specified type: number

// Practical use: Options/Config types
interface ApiOptions<T = unknown> {
  url: string;
  method: "GET" | "POST";
  body?: T;
}

const options1: ApiOptions = {
  url: "/api/data",
  method: "GET"
  // body is unknown by default
};

const options2: ApiOptions<User> = {
  url: "/api/users",
  method: "POST",
  body: { id: "1", name: "Alice", role: "admin" }
};
```

**Use defaults when:**
- Most users will use the same type
- Reducing boilerplate in common cases
- Providing sensible fallbacks

---

**Q4: How do I infer types from generic functions?**

**A:** TypeScript automatically infers type parameters from arguments:

```typescript
// TypeScript infers T from the argument
function identity<T>(value: T): T {
  return value;
}

const str = identity("hello");  // T inferred as string
const num = identity(42);       // T inferred as number

// Inference with multiple parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const p = pair(1, "hello");  // [number, string]

// You can explicitly specify if needed
const p2 = pair<string, string>("1", "hello");

// Inference with constraints
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length > b.length ? a : b;
}

longest([1, 2], [1, 2, 3]);    // T inferred as number[]
longest("hi", "hello");         // T inferred as string
```

**Inference works when:**
- Type parameters are used in function parameters
- TypeScript can determine type from argument values
- Constraints are satisfied by arguments

---

**Q5: What are variadic tuple types and when would I use them?**

**A:** Variadic tuple types let you work with tuples of varying lengths:

```typescript
// Concatenate tuples
function concat<T extends any[], U extends any[]>(
  arr1: [...T],
  arr2: [...U]
): [...T, ...U] {
  return [...arr1, ...arr2];
}

const result = concat([1, 2], ["a", "b"]);
// Type: [number, number, string, string]

// Practical use: Type-safe function composition
function pipe<A extends any[], B, C>(
  fn1: (...args: A) => B,
  fn2: (arg: B) => C
): (...args: A) => C {
  return (...args) => fn2(fn1(...args));
}

const addNumbers = (a: number, b: number) => a + b;
const toString = (n: number) => n.toString();

const addAndStringify = pipe(addNumbers, toString);
const result = addAndStringify(5, 10);  // "15"

// Type-safe curry
function curry<A extends any[], B, C>(
  fn: (...args: [...A, B]) => C
): (...prefix: A) => (last: B) => C {
  return (...prefix) => (last) => fn(...prefix, last);
}
```

**Use variadic tuples for:**
- Type-safe function composition
- Curry/partial application
- Tuple manipulation utilities
- Preserving exact tuple types

---

### Interview Questions

**Question 1: Explain how TypeScript infers generic types and when you need to explicitly specify them.**

**Difficulty:** Mid-Level

**Answer:**

TypeScript uses **type inference** to automatically determine generic type parameters from function arguments, eliminating the need to specify them explicitly in most cases.

**How Type Inference Works:**

```typescript
// Basic inference
function identity<T>(value: T): T {
  return value;
}

// TypeScript infers T from the argument
const str = identity("hello");  // T = string
const num = identity(42);       // T = number
const arr = identity([1, 2, 3]); // T = number[]

// Inference with multiple type parameters
function map<T, U>(array: T[], fn: (item: T) => U): U[] {
  return array.map(fn);
}

const numbers = [1, 2, 3];
const strings = map(numbers, n => n.toString());
// T = number (inferred from numbers)
// U = string (inferred from return type of arrow function)

// Inference from multiple arguments
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const result = merge(
  { name: "Alice" },
  { age: 30 }
);
// T = { name: string }
// U = { age: number }
// Result type: { name: string; age: number }
```

**When Inference Works Automatically:**

```typescript
// 1. Function arguments provide type information
function first<T>(array: T[]): T | undefined {
  return array[0];
}

first([1, 2, 3]);  // T = number
first(["a", "b"]); // T = string

// 2. Object properties provide context
interface Box<T> {
  value: T;
}

function unbox<T>(box: Box<T>): T {
  return box.value;
}

const numberBox: Box<number> = { value: 42 };
unbox(numberBox);  // T = number (inferred from argument)

// 3. Return type constraints
function toArray<T>(value: T): T[] {
  return [value];
}

toArray(42);  // T = number, returns number[]
```

**When You Need Explicit Type Arguments:**

```typescript
// 1. No way to infer from arguments
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

// TypeScript infers T from value
createArray(3, "hello");  // T = string

// But what if we want different type?
createArray<string | number>(3, "hello");  // T = string | number

// 2. Empty data structures
const emptyArray = [];  // Type: any[]

function createEmptyArray<T>(): T[] {
  return [];
}

const numbers = createEmptyArray<number>();  // Must specify T
const strings = createEmptyArray<string>();  // Must specify T

// 3. Type parameter not in parameters
function cast<T>(value: unknown): T {
  return value as T;
}

// Must specify T - can't infer from unknown
const user = cast<User>(jsonData);

// 4. Multiple valid inferences
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

// TypeScript infers most specific types
pair(1, "hello");  // [number, string]

// But you might want broader types
pair<number | string, string | boolean>(1, "hello");
// [number | string, string | boolean]

// 5. Working with libraries
// Sometimes library types are too broad
const element = document.querySelector("input");
// Type: Element | null (too broad)

const input = document.querySelector<HTMLInputElement>("input");
// Type: HTMLInputElement | null (specific)
```

**Best Practices:**

```typescript
// ✅ Let TypeScript infer when possible
const doubled = map([1, 2, 3], n => n * 2);

// ✅ Specify when inference isn't sufficient
const emptyUsers = createEmptyArray<User>();

// ✅ Specify for clarity in complex scenarios
const result = merge<UserBase, UserDetails>(base, details);

// ❌ Don't specify when inference works fine
const first = identity<string>("hello");  // Redundant
// Better:
const first = identity("hello");  // TypeScript infers string
```

**Why This Matters:**
- Reduces boilerplate code
- Leverages TypeScript's powerful inference
- Knowing when to specify improves code clarity
- Essential for writing ergonomic generic functions

**Follow-up Questions:**
- How does TypeScript infer types in generic constraints?
- What happens when inference produces `unknown`?
- Can you force TypeScript to infer a wider type?

---

**Question 2: Design a type-safe event emitter using generics.**

**Difficulty:** Senior

**Answer:**

A type-safe event emitter ensures event names and handler signatures match, preventing runtime errors:

```typescript
// Event map defining event names and their payload types
type EventMap = {
  "user:login": { userId: string; timestamp: Date };
  "user:logout": { userId: string };
  "data:update": { id: string; data: unknown };
  "error": { message: string; code: number };
};

// Type-safe event emitter
class TypedEventEmitter<Events extends Record<string, any>> {
  private listeners = new Map<keyof Events, Set<Function>>();
  
  // Subscribe to an event
  on<K extends keyof Events>(
    event: K,
    handler: (payload: Events[K]) => void
  ): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    
    this.listeners.get(event)!.add(handler);
    
    // Return unsubscribe function
    return () => {
      this.listeners.get(event)?.delete(handler);
    };
  }
  
  // Subscribe once
  once<K extends keyof Events>(
    event: K,
    handler: (payload: Events[K]) => void
  ): void {
    const unsubscribe = this.on(event, (payload) => {
      handler(payload);
      unsubscribe();
    });
  }
  
  // Emit an event
  emit<K extends keyof Events>(event: K, payload: Events[K]): void {
    const handlers = this.listeners.get(event);
    if (handlers) {
      handlers.forEach(handler => handler(payload));
    }
  }
  
  // Remove all listeners for an event
  off<K extends keyof Events>(event: K): void {
    this.listeners.delete(event);
  }
  
  // Remove all listeners
  removeAllListeners(): void {
    this.listeners.clear();
  }
}

// Usage
const emitter = new TypedEventEmitter<EventMap>();

// ✅ Type-safe event subscription
emitter.on("user:login", (payload) => {
  // payload is typed as { userId: string; timestamp: Date }
  console.log(`User ${payload.userId} logged in at ${payload.timestamp}`);
});

// ✅ Type-safe event emission
emitter.emit("user:login", {
  userId: "123",
  timestamp: new Date()
});

// ❌ Compile errors for mistakes
// emitter.on("invalid-event", () => {});  // Error: invalid event name
// emitter.emit("user:login", { userId: 123 });  // Error: wrong payload type
// emitter.on("user:logout", (payload) => {
//   console.log(payload.timestamp);  // Error: timestamp doesn't exist
// });

// Advanced: Async event handlers
class AsyncEventEmitter<Events extends Record<string, any>> {
  private listeners = new Map<
    keyof Events,
    Set<(payload: any) => Promise<void>>
  >();
  
  on<K extends keyof Events>(
    event: K,
    handler: (payload: Events[K]) => Promise<void>
  ): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    
    this.listeners.get(event)!.add(handler);
    
    return () => {
      this.listeners.get(event)?.delete(handler);
    };
  }
  
  async emit<K extends keyof Events>(
    event: K,
    payload: Events[K]
  ): Promise<void> {
    const handlers = this.listeners.get(event);
    if (handlers) {
      await Promise.all(
        Array.from(handlers).map(handler => handler(payload))
      );
    }
  }
}

// Enhanced version with error handling
class SafeEventEmitter<Events extends Record<string, any>> {
  private listeners = new Map<keyof Events, Set<Function>>();
  private errorHandler?: (error: Error) => void;
  
  onError(handler: (error: Error) => void): void {
    this.errorHandler = handler;
  }
  
  on<K extends keyof Events>(
    event: K,
    handler: (payload: Events[K]) => void
  ): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    
    this.listeners.get(event)!.add(handler);
    
    return () => {
      this.listeners.get(event)?.delete(handler);
    };
  }
  
  emit<K extends keyof Events>(event: K, payload: Events[K]): void {
    const handlers = this.listeners.get(event);
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(payload);
        } catch (error) {
          if (this.errorHandler) {
            this.errorHandler(error as Error);
          } else {
            console.error(`Error in handler for event "${String(event)}":`, error);
          }
        }
      });
    }
  }
}
```

**Real-World Usage:**

```typescript
// Application events
type AppEvents = {
  "app:ready": { version: string };
  "route:change": { from: string; to: string };
  "api:request": { url: string; method: string };
  "api:response": { url: string; status: number; data: unknown };
  "api:error": { url: string; error: Error };
};

const app = new TypedEventEmitter<AppEvents>();

// Setup logging
app.on("api:request", ({ url, method }) => {
  console.log(`[API] ${method} ${url}`);
});

app.on("api:response", ({ url, status }) => {
  console.log(`[API] ${url} responded with ${status}`);
});

app.on("api:error", ({ url, error }) => {
  console.error(`[API] ${url} failed:`, error.message);
});

// Trigger events
app.emit("app:ready", { version: "1.0.0" });
app.emit("route:change", { from: "/home", to: "/profile" });
```

**Why This Matters:**
- Prevents event name typos at compile time
- Ensures payload types match expectations
- Provides IntelliSense for event names and payloads
- Essential pattern for large applications
- Used extensively in libraries (Node.js EventEmitter, React events, etc.)

**Follow-up Questions:**
- How would you add event namespaces?
- How would you implement wildcard event handlers?
- How would you handle event bubbling/capturing?

---

**Question 3: Implement a type-safe function pipeline with generics.**

**Difficulty:** Senior

**Answer:**

A type-safe pipeline chains functions where each function's output type must match the next function's input type:

```typescript
// Basic pipeline with 2 functions
function pipe<A, B, C>(
  fn1: (arg: A) => B,
  fn2: (arg: B) => C
): (arg: A) => C {
  return (arg: A) => fn2(fn1(arg));
}

// Usage
const addOne = (n: number) => n + 1;
const double = (n: number) => n * 2;
const toString = (n: number) => n.toString();

const pipeline = pipe(
  addOne,    // number => number
  double     // number => number
);

pipeline(5);  // 12

// Extended pipeline (3 functions)
function pipe3<A, B, C, D>(
  fn1: (arg: A) => B,
  fn2: (arg: B) => C,
  fn3: (arg: C) => D
): (arg: A) => D {
  return (arg: A) => fn3(fn2(fn1(arg)));
}

const pipeline3 = pipe3(
  addOne,     // number => number
  double,     // number => number
  toString    // number => string
);

pipeline3(5);  // "12"

// Generic pipeline with any number of functions (advanced)
type PipeFunction<T, U> = (arg: T) => U;

// Overload signatures for different numbers of functions
function pipe<A, B>(fn1: PipeFunction<A, B>): PipeFunction<A, B>;
function pipe<A, B, C>(
  fn1: PipeFunction<A, B>,
  fn2: PipeFunction<B, C>
): PipeFunction<A, C>;
function pipe<A, B, C, D>(
  fn1: PipeFunction<A, B>,
  fn2: PipeFunction<B, C>,
  fn3: PipeFunction<C, D>
): PipeFunction<A, D>;
function pipe<A, B, C, D, E>(
  fn1: PipeFunction<A, B>,
  fn2: PipeFunction<B, C>,
  fn3: PipeFunction<C, D>,
  fn4: PipeFunction<D, E>
): PipeFunction<A, E>;
function pipe<A, B, C, D, E, F>(
  fn1: PipeFunction<A, B>,
  fn2: PipeFunction<B, C>,
  fn3: PipeFunction<C, D>,
  fn4: PipeFunction<D, E>,
  fn5: PipeFunction<E, F>
): PipeFunction<A, F>;

// Implementation
function pipe(...fns: Function[]): Function {
  return (arg: any) => fns.reduce((acc, fn) => fn(acc), arg);
}

// Usage with full type safety
const transform = pipe(
  (n: number) => n + 1,           // number => number
  (n: number) => n * 2,           // number => number
  (n: number) => n.toString(),    // number => string
  (s: string) => s.padStart(5, '0'), // string => string
  (s: string) => `Result: ${s}`   // string => string
);

const result = transform(5);  // "Result: 00012"

// Async pipeline
type AsyncPipeFunction<T, U> = (arg: T) => Promise<U>;

function asyncPipe<A, B, C>(
  fn1: AsyncPipeFunction<A, B>,
  fn2: AsyncPipeFunction<B, C>
): AsyncPipeFunction<A, C> {
  return async (arg: A) => {
    const b = await fn1(arg);
    return fn2(b);
  };
}

// Usage
const fetchUser = async (id: string): Promise<User> => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

const extractName = async (user: User): Promise<string> => {
  return user.name;
};

const toUpperCase = async (name: string): Promise<string> => {
  return name.toUpperCase();
};

const getUserName = asyncPipe(fetchUser, extractName);
await getUserName("123");

// Advanced: Pipeline with error handling
type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E };

type SafePipeFunction<T, U> = (arg: T) => Result<U>;

function safePipe<A, B, C>(
  fn1: SafePipeFunction<A, B>,
  fn2: SafePipeFunction<B, C>
): SafePipeFunction<A, C> {
  return (arg: A) => {
    const result1 = fn1(arg);
    if (!result1.success) {
      return result1;
    }
    
    const result2 = fn2(result1.value);
    return result2;
  };
}

// Object transformation pipeline
interface Pipeline<T> {
  value: T;
  pipe<U>(fn: (value: T) => U): Pipeline<U>;
  tap(fn: (value: T) => void): Pipeline<T>;
  unwrap(): T;
}

function pipeline<T>(value: T): Pipeline<T> {
  return {
    value,
    pipe<U>(fn: (value: T) => U): Pipeline<U> {
      return pipeline(fn(value));
    },
    tap(fn: (value: T) => void): Pipeline<T> {
      fn(value);
      return this;
    },
    unwrap(): T {
      return value;
    }
  };
}

// Usage
const result2 = pipeline(5)
  .pipe(n => n + 1)
  .tap(n => console.log(`After add: ${n}`))
  .pipe(n => n * 2)
  .tap(n => console.log(`After multiply: ${n}`))
  .pipe(n => n.toString())
  .unwrap();

console.log(result2);  // "12"
```

**Real-World Example: Data Transformation:**

```typescript
interface RawUser {
  id: string;
  first_name: string;
  last_name: string;
  email_address: string;
}

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserWithAvatar extends User {
  avatar: string;
}

// Transformation functions
const normalizeUser = (raw: RawUser): User => ({
  id: raw.id,
  name: `${raw.first_name} ${raw.last_name}`,
  email: raw.email_address
});

const addAvatar = (user: User): UserWithAvatar => ({
  ...user,
  avatar: `https://avatars.example.com/${user.id}`
});

const validateEmail = (user: UserWithAvatar): UserWithAvatar => {
  if (!user.email.includes('@')) {
    throw new Error('Invalid email');
  }
  return user;
};

// Create pipeline
const processUser = pipe(
  normalizeUser,
  addAvatar,
  validateEmail
);

const rawUser: RawUser = {
  id: "123",
  first_name: "Alice",
  last_name: "Smith",
  email_address: "alice@example.com"
};

const processedUser = processUser(rawUser);
// Type: UserWithAvatar
```

**Why This Matters:**
- Enables functional programming patterns
- Ensures type safety through transformations
- Common in data processing pipelines
- Used in libraries like RxJS, Ramda, fp-ts
- Essential for composable, testable code

**Follow-up Questions:**
- How would you add error handling to pipelines?
- How would you implement a pipeline that can branch?
- How would you make a pipeline debuggable?

---

### Key Takeaways

- Generics enable writing reusable, type-safe code that works with multiple types
- Type parameters can be inferred from arguments or explicitly specified
- Generic constraints limit what types can be used with `extends`
- Default type parameters provide fallback types when none specified
- Generic interfaces, classes, and type aliases enable flexible abstractions
- Variadic tuple types enable type-safe operations on tuples of any length
- Contextual typing automatically infers generic types from context
- Recursive generics enable complex type transformations
- Understanding generics is essential for using libraries and writing reusable code
- Generics are the foundation for advanced TypeScript patterns

---

## 2.2 Utility Types and Type Manipulation

TypeScript provides powerful built-in utility types and patterns for transforming and manipulating types.

### Built-in Utility Types

TypeScript includes many utility types in its standard library.

#### Partial<T>

Makes all properties optional:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// All properties become optional
type PartialUser = Partial<User>;
// Equivalent to:
// {
//   id?: string;
//   name?: string;
//   email?: string;
//   age?: number;
// }

// Usage: Updating objects
function updateUser(id: string, updates: Partial<User>): User {
  const existingUser = findUser(id);
  return { ...existingUser, ...updates };
}

updateUser("123", { name: "Alice" });  // ✅ OK: Only name
updateUser("123", { email: "alice@example.com", age: 30 });  // ✅ OK: Multiple fields

// Implementation
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};
```

#### Required<T>

Makes all properties required:

```typescript
interface Config {
  host?: string;
  port?: number;
  timeout?: number;
}

// All properties become required
type RequiredConfig = Required<Config>;
// Equivalent to:
// {
//   host: string;
//   port: number;
//   timeout: number;
// }

function createServer(config: RequiredConfig) {
  // Can safely access all properties without checking
  console.log(`Server starting on ${config.host}:${config.port}`);
}

// Implementation
type MyRequired<T> = {
  [P in keyof T]-?: T[P];
};
// The -? removes the optional modifier
```

#### Readonly<T>

Makes all properties readonly:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

type ReadonlyUser = Readonly<User>;
// Equivalent to:
// {
//   readonly id: string;
//   readonly name: string;
//   readonly email: string;
// }

const user: ReadonlyUser = {
  id: "123",
  name: "Alice",
  email: "alice@example.com"
};

// user.name = "Bob";  // ❌ Error: Cannot assign to readonly property

// Usage: Immutable data structures
function processUser(user: Readonly<User>): void {
  // Can read but not modify
  console.log(user.name);
  // user.name = "Bob";  // ❌ Error
}

// Implementation
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

#### Record<K, T>

Creates an object type with keys of type K and values of type T:

```typescript
// Basic usage
type UserRoles = Record<string, string>;

const roles: UserRoles = {
  admin: "Administrator",
  user: "Regular User",
  guest: "Guest User"
};

// With specific keys
type Page = "home" | "about" | "contact";
type PageInfo = Record<Page, { title: string; url: string }>;

const pages: PageInfo = {
  home: { title: "Home", url: "/" },
  about: { title: "About Us", url: "/about" },
  contact: { title: "Contact", url: "/contact" }
};

// With number keys
type StatusMessages = Record<number, string>;

const httpMessages: StatusMessages = {
  200: "OK",
  404: "Not Found",
  500: "Internal Server Error"
};

// Implementation
type MyRecord<K extends string | number | symbol, T> = {
  [P in K]: T;
};
```

#### Pick<T, K>

Creates a type by picking specific properties from T:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
  address: string;
  phone: string;
}

// Pick only needed properties
type UserPreview = Pick<User, "id" | "name" | "email">;
// Equivalent to:
// {
//   id: string;
//   name: string;
//   email: string;
// }

function displayUserPreview(user: UserPreview) {
  console.log(`${user.name} (${user.email})`);
  // Can't access user.age or other properties
}

// Implementation
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

#### Omit<T, K>

Creates a type by omitting specific properties from T:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Omit sensitive data
type PublicUser = Omit<User, "password">;
// Equivalent to:
// {
//   id: string;
//   name: string;
//   email: string;
//   createdAt: Date;
// }

function sendToClient(user: PublicUser) {
  // Can't accidentally send password
  return user;
}

// Omit multiple properties
type UserPreview = Omit<User, "password" | "email" | "createdAt">;

// Implementation
type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

#### Exclude<T, U>

Excludes types from a union:

```typescript
// Remove types from union
type AllTypes = string | number | boolean;
type StringAndNumber = Exclude<AllTypes, boolean>;
// Type: string | number

type Status = "pending" | "approved" | "rejected" | "cancelled";
type ActiveStatus = Exclude<Status, "cancelled">;
// Type: "pending" | "approved" | "rejected"

// Practical use: Filter object keys
type UserKeys = keyof User;  // "id" | "name" | "email" | "password"
type PublicUserKeys = Exclude<UserKeys, "password">;
// Type: "id" | "name" | "email"

// Implementation
type MyExclude<T, U> = T extends U ? never : T;
```

#### Extract<T, U>

Extracts types from a union that are assignable to U:

```typescript
// Extract specific types
type AllTypes = string | number | boolean;
type OnlyStrings = Extract<AllTypes, string>;
// Type: string

type Mixed = "a" | "b" | 1 | 2 | true | false;
type OnlyNumbers = Extract<Mixed, number>;
// Type: 1 | 2

// Extract matching types
type Events = "click" | "focus" | "keydown" | "keyup" | "mouseenter";
type KeyEvents = Extract<Events, `key${string}`>;
// Type: "keydown" | "keyup"

// Implementation
type MyExtract<T, U> = T extends U ? T : never;
```

#### NonNullable<T>

Removes null and undefined from a type:

```typescript
// Remove null and undefined
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// Type: string

type MaybeUser = User | null | undefined;
type DefiniteUser = NonNullable<MaybeUser>;
// Type: User

// Usage in functions
function processValue<T>(value: T): NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error("Value cannot be null or undefined");
  }
  return value as NonNullable<T>;
}

// Implementation
type MyNonNullable<T> = T extends null | undefined ? never : T;
```

#### ReturnType<T>

Extracts the return type of a function:

```typescript
function createUser(name: string, email: string) {
  return {
    id: Math.random().toString(),
    name,
    email,
    createdAt: new Date()
  };
}

// Extract return type
type User = ReturnType<typeof createUser>;
// Type: {
//   id: string;
//   name: string;
//   email: string;
//   createdAt: Date;
// }

// With generic functions
function identity<T>(value: T): T {
  return value;
}

type IdentityReturn = ReturnType<typeof identity>;
// Type: unknown (because T is unknown)

// With async functions
async function fetchUser(id: string): Promise<User> {
  // ...
  return {} as User;
}

type FetchUserReturn = ReturnType<typeof fetchUser>;
// Type: Promise<User>

// Unwrap Promise
type UserFromFetch = Awaited<ReturnType<typeof fetchUser>>;
// Type: User

// Implementation
type MyReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : any;
```

#### Parameters<T>

Extracts parameter types as a tuple:

```typescript
function createUser(name: string, age: number, email?: string) {
  // ...
}

// Extract parameters
type CreateUserParams = Parameters<typeof createUser>;
// Type: [name: string, age: number, email?: string]

// Use with spread
function loggedCreateUser(...args: CreateUserParams) {
  console.log("Creating user with:", args);
  return createUser(...args);
}

// Access specific parameters
type FirstParam = CreateUserParams[0];   // string
type SecondParam = CreateUserParams[1];  // number
type ThirdParam = CreateUserParams[2];   // string | undefined

// Implementation
type MyParameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any ? P : never;
```

#### ConstructorParameters<T>

Extracts constructor parameter types:

```typescript
class User {
  constructor(
    public name: string,
    public age: number,
    public email?: string
  ) {}
}

// Extract constructor parameters
type UserConstructorParams = ConstructorParameters<typeof User>;
// Type: [name: string, age: number, email?: string]

// Use in factory function
function createUser(...args: UserConstructorParams): User {
  return new User(...args);
}

// Implementation
type MyConstructorParameters<T extends new (...args: any) => any> =
  T extends new (...args: infer P) => any ? P : never;
```

#### InstanceType<T>

Extracts the instance type of a class:

```typescript
class User {
  constructor(public name: string, public age: number) {}
  
  greet() {
    return `Hello, I'm ${this.name}`;
  }
}

// Extract instance type
type UserInstance = InstanceType<typeof User>;
// Type: User

// Use in generic code
function createInstance<T extends new (...args: any) => any>(
  constructor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new constructor(...args);
}

const user = createInstance(User, "Alice", 30);
// Type: User

// Implementation
type MyInstanceType<T extends new (...args: any) => any> =
  T extends new (...args: any) => infer R ? R : any;
```

#### ThisParameterType<T>

Extracts the type of the `this` parameter:

```typescript
function greet(this: User) {
  return `Hello, I'm ${this.name}`;
}

type GreetThis = ThisParameterType<typeof greet>;
// Type: User

// Implementation
type MyThisParameterType<T> =
  T extends (this: infer U, ...args: any) => any ? U : unknown;
```

#### OmitThisParameter<T>

Removes the `this` parameter from a function type:

```typescript
function greet(this: User, message: string) {
  return `${message}, I'm ${this.name}`;
}

type GreetWithoutThis = OmitThisParameter<typeof greet>;
// Type: (message: string) => string

// Implementation
type MyOmitThisParameter<T> =
  T extends (this: any, ...args: infer A) => infer R
    ? (...args: A) => R
    : T;
```

#### String Manipulation Types

TypeScript provides intrinsic string manipulation types:

```typescript
// Uppercase
type Loud = Uppercase<"hello">;  // "HELLO"
type LoudGreeting = Uppercase<"hello world">;  // "HELLO WORLD"

// Lowercase
type Quiet = Lowercase<"HELLO">;  // "hello"
type QuietGreeting = Lowercase<"HELLO WORLD">;  // "hello world"

// Capitalize
type Cap = Capitalize<"hello">;  // "Hello"
type CapSentence = Capitalize<"hello world">;  // "Hello world"

// Uncapitalize
type Uncap = Uncapitalize<"Hello">;  // "hello"
type UncapSentence = Uncapitalize<"Hello World">;  // "hello World"

// Practical usage with template literals
type HTTPMethod = "get" | "post" | "put" | "delete";
type HTTPMethodUpper = Uppercase<HTTPMethod>;
// Type: "GET" | "POST" | "PUT" | "DELETE"

type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// Type: "onClick" | "onFocus" | "onBlur"
```

#### Awaited<T>

Recursively unwraps Promise types:

```typescript
// Single Promise
type P1 = Awaited<Promise<string>>;  // string

// Nested Promises
type P2 = Awaited<Promise<Promise<string>>>;  // string
type P3 = Awaited<Promise<Promise<Promise<number>>>>;  // number

// Non-Promise
type N = Awaited<string>;  // string

// Usage with async functions
async function fetchUser(): Promise<User> {
  return {} as User;
}

type FetchResult = Awaited<ReturnType<typeof fetchUser>>;
// Type: User

// Implementation (simplified)
type MyAwaited<T> = T extends Promise<infer U> ? MyAwaited<U> : T;
```

### Creating Custom Utility Types

Build your own utility types for common patterns:

```typescript
// Deep Partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface NestedUser {
  name: string;
  address: {
    street: string;
    city: string;
    country: {
      name: string;
      code: string;
    };
  };
}

const partialUser: DeepPartial<NestedUser> = {
  name: "Alice",
  address: {
    city: "New York"
    // Other fields optional
  }
};

// Deep Readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};

const immutableUser: DeepReadonly<NestedUser> = {
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "New York",
    country: {
      name: "USA",
      code: "US"
    }
  }
};

// immutableUser.address.city = "Boston";  // ❌ Error: readonly

// Mutable (remove readonly)
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

type MutableUser = Mutable<ReadonlyUser>;
// All properties become mutable

// PickByType
type PickByType<T, U> = {
  [P in keyof T as T[P] extends U ? P : never]: T[P];
};

interface Mixed {
  id: number;
  name: string;
  age: number;
  email: string;
  active: boolean;
}

type StringProps = PickByType<Mixed, string>;
// Type: { name: string; email: string; }

type NumberProps = PickByType<Mixed, number>;
// Type: { id: number; age: number; }

// OmitByType
type OmitByType<T, U> = {
  [P in keyof T as T[P] extends U ? never : P]: T[P];
};

type NonStringProps = OmitByType<Mixed, string>;
// Type: { id: number; age: number; active: boolean; }

// Nullable
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type NullableUser = Nullable<User>;
// All properties can be null

// OptionalKeys
type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

interface PartialConfig {
  required: string;
  optional?: number;
  alsoOptional?: boolean;
}

type ConfigOptionalKeys = OptionalKeys<PartialConfig>;
// Type: "optional" | "alsoOptional"

// RequiredKeys
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type ConfigRequiredKeys = RequiredKeys<PartialConfig>;
// Type: "required"

// Flatten object type
type Flatten<T> = T extends object
  ? { [K in keyof T]: Flatten<T[K]> }
  : T;

// Diff (properties in T but not in U)
type Diff<T, U> = Omit<T, keyof U>;

interface A {
  a: string;
  b: number;
  c: boolean;
}

interface B {
  b: number;
  c: boolean;
}

type OnlyInA = Diff<A, B>;
// Type: { a: string; }

// Intersection (common properties)
type Intersection<T, U> = Pick<T, Extract<keyof T, keyof U>>;

type Common = Intersection<A, B>;
// Type: { b: number; c: boolean; }

// Overwrite
type Overwrite<T, U> = Omit<T, keyof U> & U;

type UpdatedUser = Overwrite<User, { name: number }>;
// Changes name from string to number

// PromiseType
type PromiseType<T> = T extends Promise<infer U> ? U : never;

type UserPromise = Promise<User>;
type ExtractedUser = PromiseType<UserPromise>;
// Type: User

// FunctionKeys (keys that are functions)
type FunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

interface UserWithMethods {
  id: string;
  name: string;
  greet(): string;
  save(): Promise<void>;
}

type UserMethods = FunctionKeys<UserWithMethods>;
// Type: "greet" | "save"

// NonFunctionKeys
type NonFunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

type UserProperties = NonFunctionKeys<UserWithMethods>;
// Type: "id" | "name"
```

### Frequently Asked Questions

**Q1: When should I use Partial vs making properties optional with `?`?**

**A:** Use `Partial` for dynamic scenarios, `?` for known optional properties:

```typescript
// Known optional properties - use ?
interface Config {
  host: string;         // Always required
  port?: number;        // Known optional
  timeout?: number;     // Known optional
}

// Dynamic optional - use Partial
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Update function where any field might be updated
function updateUser(id: string, updates: Partial<User>) {
  // Can update any combination of fields
}

updateUser("123", { name: "Alice" });
updateUser("123", { email: "alice@example.com", age: 30 });
```

**Use Partial when:**
- Updating objects with unknown fields
- Accepting configuration overrides
- Building objects incrementally

**Use ? when:**
- Property is conceptually optional
- API design (clear which fields are optional)
- Form validation schemas

---

**Q2: What's the difference between Omit and Exclude?**

**A:** `Omit` works on object properties, `Exclude` works on union types:

```typescript
// Omit: Remove properties from object type
interface User {
  id: string;
  name: string;
  password: string;
}

type PublicUser = Omit<User, "password">;
// Type: { id: string; name: string; }

// Exclude: Remove types from union
type Status = "pending" | "approved" | "rejected";
type ActiveStatus = Exclude<Status, "rejected">;
// Type: "pending" | "approved"

// Can't use Omit on unions
type Invalid = Omit<Status, "rejected">;  // Doesn't work as expected

// Can't use Exclude on object properties
type AlsoInvalid = Exclude<User, "password">;  // Doesn't work
```

---

**Q3: How do I make nested properties optional/readonly?**

**A:** Use recursive utility types:

```typescript
// DeepPartial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface User {
  name: string;
  address: {
    street: string;
    city: string;
  };
}

// Partial only makes top level optional
type ShallowPartial = Partial<User>;
// { name?: string; address?: {...}; }

// DeepPartial makes all levels optional
type DeepPartial = DeepPartial<User>;
// { name?: string; address?: { street?: string; city?: string; }; }

// DeepReadonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};
```

---

**Q4: Can I create a utility type that picks properties by their type?**

**A:** Yes, using mapped types with conditional types:

```typescript
type PickByType<T, U> = {
  [P in keyof T as T[P] extends U ? P : never]: T[P];
};

interface Mixed {
  id: number;
  name: string;
  age: number;
  email: string;
  active: boolean;
}

type StringProps = PickByType<Mixed, string>;
// { name: string; email: string; }

type NumberProps = PickByType<Mixed, number>;
// { id: number; age: number; }

// Also works with complex types
type FunctionProps = PickByType<Mixed, Function>;
type ObjectProps = PickByType<Mixed, object>;
```

---

**Q5: How do ReturnType and Parameters work with generic functions?**

**A:** They extract types but lose generic information:

```typescript
// Non-generic function
function add(a: number, b: number): number {
  return a + b;
}

type AddReturn = ReturnType<typeof add>;  // number
type AddParams = Parameters<typeof add>;  // [number, number]

// Generic function
function identity<T>(value: T): T {
  return value;
}

type IdentityReturn = ReturnType<typeof identity>;  // unknown
type IdentityParams = Parameters<typeof identity>;  // [unknown]

// Solution: Use with specific type
type StringIdentity = typeof identity<string>;
type StringReturn = ReturnType<StringIdentity>;  // string

// Or extract from usage
const result = identity("hello");
type ResultType = typeof result;  // string
```

---

### Interview Questions

**Question 1: Implement a DeepPartial utility type that makes all properties optional recursively.**

**Difficulty:** Senior

**Answer:**

```typescript
// DeepPartial implementation
type DeepPartial<T> = T extends Function
  ? T
  : T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

// Test with nested interface
interface Company {
  name: string;
  address: {
    street: string;
    city: string;
    country: {
      name: string;
      code: string;
    };
  };
  employees: {
    name: string;
    role: string;
    salary: number;
  }[];
}

// All levels become optional
const partial: DeepPartial<Company> = {
  name: "Acme Corp",
  address: {
    city: "New York"
    // street and country are optional
  }
  // employees is optional
};

// Why this implementation works:
// 1. Check if T is a function - preserve function types
// 2. Check if T is an object - recurse into properties
// 3. Make each property optional with ?
// 4. Recursively apply DeepPartial to each property type
// 5. Primitive types pass through unchanged

// Advanced version handling arrays
type DeepPartialAdvanced<T> = T extends Function
  ? T
  : T extends Array<infer U>
  ? Array<DeepPartialAdvanced<U>>
  : T extends object
  ? { [P in keyof T]?: DeepPartialAdvanced<T[P]> }
  : T;

// Now correctly handles arrays
const advancedPartial: DeepPartialAdvanced<Company> = {
  employees: [
    { name: "Alice" }  // role and salary optional
  ]
};

// Practical usage
function updateCompany(
  id: string,
  updates: DeepPartial<Company>
): Company {
  const existing = getCompany(id);
  return deepMerge(existing, updates);
}

// Can update any nested property
updateCompany("123", {
  address: {
    country: {
      code: "US"  // Only update country code
    }
  }
});
```

**Why This Matters:**
- Common pattern in update/patch operations
- Enables flexible partial updates
- Used in ORMs and state management
- Demonstrates advanced type recursion

**Follow-up Questions:**
- How would you handle circular references?
- How would you implement DeepRequired?
- What about Date and other built-in objects?

---

**Question 2: Create a type-safe Pick utility that only picks properties with specific value types.**

**Difficulty:** Senior

**Answer:**

```typescript
// PickByValueType implementation
type PickByValueType<T, ValueType> = {
  [K in keyof T as T[K] extends ValueType ? K : never]: T[K];
};

// Test interface
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  active: boolean;
  createdAt: Date;
  save(): Promise<void>;
}

// Pick only string properties
type StringProps = PickByValueType<User, string>;
// Result: { name: string; email: string; }

// Pick only number properties
type NumberProps = PickByValueType<User, number>;
// Result: { id: number; age: number; }

// Pick only functions
type MethodProps = PickByValueType<User, Function>;
// Result: { save: () => Promise<void>; }

// Opposite: OmitByValueType
type OmitByValueType<T, ValueType> = {
  [K in keyof T as T[K] extends ValueType ? never : K]: T[K];
};

type NonStringProps = OmitByValueType<User, string>;
// Result: { id: number; age: number; active: boolean; createdAt: Date; save: () => Promise<void>; }

// Advanced: Pick by multiple value types
type PickByValueTypes<T, ValueTypes> = {
  [K in keyof T as T[K] extends ValueTypes ? K : never]: T[K];
};

type StringOrNumberProps = PickByValueTypes<User, string | number>;
// Result: { id: number; name: string; email: string; age: number; }

// Practical use cases
// 1. Extracting serializable properties
type SerializableProps<T> = OmitByValueType<
  T,
  Function | Symbol | undefined
>;

type SerializableUser = SerializableProps<User>;
// Excludes methods, only data properties

// 2. Type-safe property validators
function validateStringProperties<T>(
  obj: T,
  validators: {
    [K in keyof PickByValueType<T, string>]?: (value: string) => boolean;
  }
): boolean {
  // Implementation
  return true;
}

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  age: 30,
  active: true,
  createdAt: new Date(),
  save: async () => {}
};

validateStringProperties(user, {
  name: (value) => value.length > 0,
  email: (value) => value.includes("@")
  // TypeScript ensures only string properties here
});

// 3. Filtering form fields by type
interface FormData {
  username: string;
  email: string;
  age: number;
  agreedToTerms: boolean;
  subscribe: boolean;
}

type TextFields = PickByValueType<FormData, string>;
type CheckboxFields = PickByValueType<FormData, boolean>;

// Type-safe form field renderers
function renderTextFields(fields: TextFields) {
  // Render text inputs
}

function renderCheckboxes(fields: CheckboxFields) {
  // Render checkboxes
}
```

**Key Concepts Demonstrated:**
1. Mapped types with key remapping
2. Conditional types for filtering
3. Generic constraints for flexibility
4. Practical applications in validation and serialization

**Why This Matters:**
- Enables type-safe property filtering
- Common in ORMs and serialization libraries
- Used in form libraries for field grouping
- Demonstrates advanced type manipulation

**Follow-up Questions:**
- How would you handle optional properties?
- Can you make this work with nested objects?
- How would you pick by property name pattern?

---

### Key Takeaways

- TypeScript provides comprehensive built-in utility types for common transformations
- `Partial`, `Required`, `Readonly` modify property modifiers
- `Pick`, `Omit` select or exclude properties
- `Record` creates object types with specific keys and values
- `Exclude`, `Extract`, `NonNullable` work on union types
- `ReturnType`, `Parameters` extract function type information
- String manipulation types (`Uppercase`, `Lowercase`, etc.) enable template literal type transformations
- `Awaited` recursively unwraps Promise types
- Custom utility types enable domain-specific type transformations
- Understanding utility types is essential for advanced TypeScript development
- Combining utility types creates powerful type manipulations

---

*[Part 2 continues with sections 2.3-2.8, maintaining the same comprehensive structure with conditional types, mapped types, template literal types, type guards, decorators, and advanced patterns. Each section includes detailed explanations, code examples, FAQs, interview questions, and real-world applications. Total length will be 25,000+ words.]*

## 2.3 Conditional Types

Conditional types allow you to create types that depend on conditions, enabling powerful type-level programming.

### Conditional Type Basics

The syntax is similar to JavaScript's ternary operator:

```typescript
// Basic syntax: T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;   // true
type Test2 = IsString<number>;   // false
type Test3 = IsString<"hello">;  // true (string literal extends string)

// Real-world example: TypeOf
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

type T1 = TypeName<string>;     // "string"
type T2 = TypeName<42>;         // "number"
type T3 = TypeName<true>;       // "boolean"
type T4 = TypeName<() => void>; // "function"
```

### Distributive Conditional Types

Conditional types distribute over union types:

```typescript
// Distributive behavior
type ToArray<T> = T extends any ? T[] : never;

type StringOrNumber = string | number;
type Result = ToArray<StringOrNumber>;
// Result: string[] | number[]
// Distributes to: ToArray<string> | ToArray<number>

// Without distribution (naked type parameter)
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type Result2 = ToArrayNonDist<string | number>;
// Result: (string | number)[]
// No distribution, treats union as a whole

// Practical example: Filter union types
type Diff<T, U> = T extends U ? never : T;

type T1 = Diff<"a" | "b" | "c", "a">;  // "b" | "c"
type T2 = Diff<string | number | boolean, string>;  // number | boolean

// Extract matching types
type Filter<T, U> = T extends U ? T : never;

type T3 = Filter<"a" | "b" | "c", "a">;  // "a"
type T4 = Filter<string | number | boolean, string>;  // string
```

### Type Inference with `infer`

The `infer` keyword allows you to extract types within conditional types:

```typescript
// Extract return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet(): string {
  return "Hello";
}

type GreetReturn = ReturnType<typeof greet>;  // string

// Extract array element type
type Flatten<T> = T extends Array<infer U> ? U : T;

type Str = Flatten<string[]>;     // string
type Num = Flatten<number[]>;     // number
type NotArray = Flatten<boolean>; // boolean

// Extract Promise type
type Unpromise<T> = T extends Promise<infer U> ? U : T;

type User = { id: string; name: string };
type UserPromise = Promise<User>;
type UnwrappedUser = Unpromise<UserPromise>;  // User

// Nested inference
type DeepFlatten<T> = T extends Array<infer U>
  ? U extends Array<any>
    ? DeepFlatten<U>
    : U
  : T;

type Deep = DeepFlatten<string[][][]>;  // string

// Extract first parameter
type FirstParameter<T> = T extends (first: infer F, ...args: any[]) => any
  ? F
  : never;

function process(name: string, age: number): void {}

type FirstParam = FirstParameter<typeof process>;  // string

// Extract all parameters
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

type ProcessParams = Parameters<typeof process>;  // [string, number]

// Extract constructor parameters
type ConstructorParameters<T> = T extends new (...args: infer P) => any
  ? P
  : never;

class User {
  constructor(public name: string, public age: number) {}
}

type UserParams = ConstructorParameters<typeof User>;  // [string, number]
```

### Nested Conditional Types

Chain multiple conditions for complex logic:

```typescript
// Multi-level type checking
type TypeCheck<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends null ? "null" :
  T extends Function ? "function" :
  T extends any[] ? "array" :
  "object";

type Check1 = TypeCheck<string>;      // "string"
type Check2 = TypeCheck<42>;          // "number"
type Check3 = TypeCheck<[1, 2, 3]>;   // "array"
type Check4 = TypeCheck<{}>;          // "object"

// Nested conditions with inference
type UnwrapAll<T> = 
  T extends Promise<infer U>
    ? UnwrapAll<U>
    : T extends Array<infer U>
      ? UnwrapAll<U>
      : T;

type Test1 = UnwrapAll<Promise<Promise<string>>>;  // string
type Test2 = UnwrapAll<string[][]>;                 // string
type Test3 = UnwrapAll<Promise<string[]>>;         // string

// Conditional with constraints
type ExtractStrings<T> = T extends { [K in keyof T]: infer U }
  ? U extends string
    ? U
    : never
  : never;

interface Mixed {
  a: string;
  b: number;
  c: string;
}

type Strings = ExtractStrings<Mixed>;  // string
```

### Mapped Types with Conditional Types

Combine mapped types with conditional types for powerful transformations:

```typescript
// Make properties optional based on type
type OptionalByType<T, K> = {
  [P in keyof T]: T[P] extends K ? T[P] | undefined : T[P];
};

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type UserWithOptionalStrings = OptionalByType<User, string>;
// {
//   id: number;
//   name: string | undefined;
//   email: string | undefined;
//   age: number;
// }

// Nullable based on type
type NullableByType<T, K> = {
  [P in keyof T]: T[P] extends K ? T[P] | null : T[P];
};

type UserWithNullableNumbers = NullableByType<User, number>;
// {
//   id: number | null;
//   name: string;
//   email: string;
//   age: number | null;
// }

// Convert types conditionally
type Stringify<T> = {
  [P in keyof T]: T[P] extends number ? string : T[P];
};

type StringifiedUser = Stringify<User>;
// {
//   id: string;      // number -> string
//   name: string;
//   email: string;
//   age: string;     // number -> string
// }

// Remove properties based on type
type RemoveByType<T, K> = {
  [P in keyof T as T[P] extends K ? never : P]: T[P];
};

type UserWithoutStrings = RemoveByType<User, string>;
// {
//   id: number;
//   age: number;
// }
```

### Advanced Conditional Patterns

#### Type-Level If-Else Chains

```typescript
// Complex type branching
type StringToNumber<T> = T extends `${infer N extends number}` ? N : never;

type Num1 = StringToNumber<"123">;   // 123
type Num2 = StringToNumber<"hello">; // never

// Type-level switch
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

type ResponseType<M extends HttpMethod> = 
  M extends "GET" ? { data: unknown } :
  M extends "POST" ? { created: true; id: string } :
  M extends "PUT" ? { updated: true } :
  M extends "DELETE" ? { deleted: true } :
  never;

type GetResponse = ResponseType<"GET">;     // { data: unknown }
type PostResponse = ResponseType<"POST">;   // { created: true; id: string }
```

#### Recursive Conditional Types

```typescript
// Deep flatten array
type DeepArray<T> = Array<T | DeepArray<T>>;

type FlattenDeep<T> = T extends Array<infer U>
  ? FlattenDeep<U>
  : T;

type Nested = DeepArray<number>;
type Flat = FlattenDeep<Nested>;  // number

// Deep property access
type GetProperty<T, Path extends string> = 
  Path extends `${infer First}.${infer Rest}`
    ? First extends keyof T
      ? GetProperty<T[First], Rest>
      : never
    : Path extends keyof T
      ? T[Path]
      : never;

interface Company {
  name: string;
  address: {
    street: string;
    city: {
      name: string;
      code: string;
    };
  };
}

type CityName = GetProperty<Company, "address.city.name">;  // string
type Street = GetProperty<Company, "address.street">;       // string

// Path validation
type ValidPaths<T, Prefix extends string = ""> = 
  T extends object
    ? {
        [K in keyof T]: K extends string
          ? T[K] extends object
            ? `${Prefix}${K}` | ValidPaths<T[K], `${Prefix}${K}.`>
            : `${Prefix}${K}`
          : never;
      }[keyof T]
    : never;

type CompanyPaths = ValidPaths<Company>;
// "name" | "address" | "address.street" | "address.city" | "address.city.name" | "address.city.code"
```

#### Type-Safe Function Overloading

```typescript
// Conditional return types based on parameters
function get<K extends "name">(key: K): string;
function get<K extends "age">(key: K): number;
function get<K extends "active">(key: K): boolean;
function get(key: string): any {
  // Implementation
  return null;
}

const name = get("name");     // Type: string
const age = get("age");       // Type: number
const active = get("active"); // Type: boolean

// Using conditional types
type GetReturnType<K> = 
  K extends "name" ? string :
  K extends "age" ? number :
  K extends "active" ? boolean :
  never;

function betterGet<K extends "name" | "age" | "active">(
  key: K
): GetReturnType<K> {
  // Implementation
  return null as any;
}
```

### Practical Conditional Type Patterns

#### API Response Typing

```typescript
// Different response shapes based on status
type APIResponse<Status extends number> = 
  Status extends 200 ? { success: true; data: unknown } :
  Status extends 201 ? { success: true; created: true; id: string } :
  Status extends 400 ? { success: false; error: string; code: "BAD_REQUEST" } :
  Status extends 401 ? { success: false; error: string; code: "UNAUTHORIZED" } :
  Status extends 404 ? { success: false; error: string; code: "NOT_FOUND" } :
  Status extends 500 ? { success: false; error: string; code: "SERVER_ERROR" } :
  never;

function handleResponse<S extends number>(
  status: S,
  response: APIResponse<S>
): void {
  if (status === 200) {
    // TypeScript knows response is { success: true; data: unknown }
  }
}
```

#### Form Field Types

```typescript
// Different input types based on field type
type FormField<T> = 
  T extends string ? { type: "text"; value: string } :
  T extends number ? { type: "number"; value: number } :
  T extends boolean ? { type: "checkbox"; checked: boolean } :
  T extends Date ? { type: "date"; value: string } :
  never;

interface FormData {
  name: FormField<string>;
  age: FormField<number>;
  subscribed: FormField<boolean>;
  birthDate: FormField<Date>;
}

const form: FormData = {
  name: { type: "text", value: "Alice" },
  age: { type: "number", value: 30 },
  subscribed: { type: "checkbox", checked: true },
  birthDate: { type: "date", value: "1990-01-01" }
};
```

#### State Machine Types

```typescript
// Type-safe state transitions
type State = "idle" | "loading" | "success" | "error";

type ValidTransitions<S extends State> = 
  S extends "idle" ? "loading" :
  S extends "loading" ? "success" | "error" :
  S extends "success" ? "idle" :
  S extends "error" ? "idle" | "loading" :
  never;

function transition<S extends State>(
  from: S,
  to: ValidTransitions<S>
): ValidTransitions<S> {
  return to;
}

// Valid transitions
transition("idle", "loading");        // ✅ OK
transition("loading", "success");     // ✅ OK
transition("loading", "error");       // ✅ OK
// transition("idle", "success");     // ❌ Error: Invalid transition
```

### Frequently Asked Questions

**Q1: What's the difference between conditional types and union types?**

**A:** Conditional types make decisions based on type relationships, while unions represent "one of" types:

```typescript
// Union: Value can be string OR number
type StringOrNumber = string | number;

let value: StringOrNumber = "hello";  // Can be string
value = 42;                           // Or number

// Conditional: Type is determined by condition
type IsString<T> = T extends string ? true : false;

type Result1 = IsString<string>;  // true
type Result2 = IsString<number>;  // false

// Conditional types are computed at type level
// Unions are a set of possible types
```

---

**Q2: How does type inference with `infer` work?**

**A:** `infer` declares a type variable within a conditional type that TypeScript infers from the matched type:

```typescript
// Extract return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
//                                                  ^^^^^
//                              TypeScript infers R from the return type

function greet(): string {
  return "Hello";
}

type Return = ReturnType<typeof greet>;  // string

// Multiple infer positions
type FirstAndLast<T> = T extends [infer First, ...any[], infer Last]
  ? [First, Last]
  : never;

type Result = FirstAndLast<[1, 2, 3, 4, 5]>;  // [1, 5]

// Infer in nested positions
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type User = { name: string };
type UserPromise = Promise<User>;
type Unwrapped = UnwrapPromise<UserPromise>;  // User
```

---

**Q3: What are distributive conditional types?**

**A:** Distributive conditional types automatically distribute over union types:

```typescript
// Distributive (naked type parameter)
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>;
// Distributes to: ToArray<string> | ToArray<number>
// Result: string[] | number[]

// Non-distributive (wrapped type parameter)
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type Result2 = ToArrayNonDist<string | number>;
// No distribution: (string | number)[]

// Practical: Filter union
type NonNullable<T> = T extends null | undefined ? never : T;

type Clean = NonNullable<string | null | number | undefined>;
// Result: string | number
```

**Key point:** Distribution happens when the type parameter appears "naked" (not wrapped) on the left side of `extends`.

---

**Q4: When should I use conditional types vs function overloads?**

**A:** Use conditional types for type-level logic, overloads for different implementations:

```typescript
// Conditional types: Same implementation, different return types
type Result<T> = T extends string ? number : string;

function process<T>(value: T): Result<T> {
  return (typeof value === "string" ? 42 : "hello") as Result<T>;
}

const r1 = process("hello");  // Type: number
const r2 = process(42);       // Type: string

// Function overloads: Different implementations
function format(value: string): string;
function format(value: number): string;
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else {
    return value.toFixed(2);
  }
}

// Use conditional types when:
// - Return type depends on input type
// - Type-level computation
// - Generic type transformations

// Use overloads when:
// - Different implementations needed
// - Complex parameter relationships
// - Better error messages desired
```

---

**Q5: How do I debug complex conditional types?**

**A:** Break them down into smaller parts and test incrementally:

```typescript
// Complex type (hard to debug)
type ComplexType<T> = T extends Array<infer U>
  ? U extends { id: string }
    ? { items: U[]; count: number }
    : never
  : never;

// Break into steps
type Step1<T> = T extends Array<infer U> ? U : never;
type Test1 = Step1<User[]>;  // User

type Step2<T> = T extends { id: string } ? T : never;
type Test2 = Step2<User>;    // User (if has id)

type Step3<T> = { items: T[]; count: number };
type Test3 = Step3<User>;    // { items: User[]; count: number }

// Combine steps
type Debugged<T> = T extends Array<infer U>
  ? Step2<U> extends never
    ? never
    : Step3<Step2<U>>
  : never;

// Testing strategy:
// 1. Test each condition separately
// 2. Use type assertions to verify
// 3. Add comments explaining logic
// 4. Use descriptive intermediate types

// Helper for debugging
type Debug<T> = { [K in keyof T]: T[K] } & {};

type DebugUser = Debug<ComplexType<User[]>>;
// Expands the type for easier inspection
```

---

### Interview Questions

**Question 1: Implement a type that extracts all possible paths from a nested object type.**

**Difficulty:** Senior

**Answer:**

```typescript
// PathOf implementation
type PathOf<T, Prefix extends string = ""> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? T[K] extends object
          ? 
              | `${Prefix}${K}`
              | PathOf<T[K], `${Prefix}${K}.`>
          : `${Prefix}${K}`
        : never;
    }[keyof T]
  : never;

// Test with nested interface
interface Company {
  name: string;
  address: {
    street: string;
    city: {
      name: string;
      code: string;
      country: {
        name: string;
        iso: string;
      };
    };
  };
  employees: {
    count: number;
    departments: string[];
  };
}

type CompanyPaths = PathOf<Company>;
// Result:
// | "name"
// | "address"
// | "address.street"
// | "address.city"
// | "address.city.name"
// | "address.city.code"
// | "address.city.country"
// | "address.city.country.name"
// | "address.city.country.iso"
// | "employees"
// | "employees.count"
// | "employees.departments"

// How it works:
// 1. Check if T is an object
// 2. Map over each key K
// 3. If value is object, recurse with updated prefix
// 4. If value is primitive, return current path
// 5. Union all paths together

// Enhanced version with type access
type PathValue<T, P extends string> = 
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? PathValue<T[K], Rest>
      : never
    : P extends keyof T
      ? T[P]
      : never;

type CityName = PathValue<Company, "address.city.name">;  // string
type EmployeeCount = PathValue<Company, "employees.count">;  // number

// Type-safe get function
function get<T, P extends PathOf<T>>(
  obj: T,
  path: P
): PathValue<T, P> {
  const keys = path.split(".");
  let result: any = obj;
  for (const key of keys) {
    result = result[key];
  }
  return result;
}

const company: Company = {
  name: "Acme Corp",
  address: {
    street: "123 Main St",
    city: {
      name: "New York",
      code: "NYC",
      country: {
        name: "USA",
        iso: "US"
      }
    }
  },
  employees: {
    count: 100,
    departments: ["Engineering", "Sales"]
  }
};

const cityName = get(company, "address.city.name");  // Type: string
const count = get(company, "employees.count");        // Type: number
// const invalid = get(company, "address.invalid");   // ❌ Error
```

**Why This Matters:**
- Type-safe property access in deeply nested objects
- Used in form libraries (React Hook Form, Formik)
- Essential for type-safe ORMs
- Common in configuration management

**Follow-up Questions:**
- How would you handle arrays in paths?
- How would you implement type-safe set()?
- What about optional properties?

---

**Question 2: Create a type-safe builder pattern using conditional types.**

**Difficulty:** Senior

**Answer:**

```typescript
// Builder pattern with type safety
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Track which properties have been set
type SetProperties<T, K extends keyof T> = {
  [P in keyof T]: P extends K ? T[P] : never;
};

// Builder type that tracks state
type UserBuilder<Set extends keyof User = never> = {
  setId(id: string): UserBuilder<Set | "id">;
  setName(name: string): UserBuilder<Set | "name">;
  setEmail(email: string): UserBuilder<Set | "email">;
  setAge(age: number): UserBuilder<Set | "age">;
} & (Set extends keyof User
  ? Required<User> extends Pick<User, Set>
    ? { build(): User }
    : {}
  : {});

// Implementation
class UserBuilderImpl<Set extends keyof User = never> {
  private data: Partial<User> = {};

  setId(id: string): UserBuilder<Set | "id"> {
    this.data.id = id;
    return this as any;
  }

  setName(name: string): UserBuilder<Set | "name"> {
    this.data.name = name;
    return this as any;
  }

  setEmail(email: string): UserBuilder<Set | "email"> {
    this.data.email = email;
    return this as any;
  }

  setAge(age: number): UserBuilder<Set | "age"> {
    this.data.age = age;
    return this as any;
  }

  build(): User {
    if (
      this.data.id &&
      this.data.name &&
      this.data.email &&
      this.data.age !== undefined
    ) {
      return this.data as User;
    }
    throw new Error("Not all required fields are set");
  }
}

function createUserBuilder(): UserBuilder<never> {
  return new UserBuilderImpl() as any;
}

// Usage
const builder = createUserBuilder();

// Can't build yet - missing required fields
// builder.build();  // ❌ Error: build() doesn't exist

const builder2 = builder
  .setId("123")
  .setName("Alice");

// Still can't build
// builder2.build();  // ❌ Error: build() doesn't exist

const builder3 = builder2
  .setEmail("alice@example.com")
  .setAge(30);

// Now can build!
const user = builder3.build();  // ✅ OK: all fields set

// Alternative approach: Generic builder
type Builder<T, Set extends keyof T = never> = {
  set<K extends keyof T>(key: K, value: T[K]): Builder<T, Set | K>;
} & (Set extends keyof T
  ? Required<T> extends Pick<T, Set>
    ? { build(): T }
    : {}
  : {});

class GenericBuilder<T, Set extends keyof T = never> {
  private data: Partial<T> = {};

  set<K extends keyof T>(key: K, value: T[K]): Builder<T, Set | K> {
    this.data[key] = value;
    return this as any;
  }

  build(): T {
    return this.data as T;
  }
}

function builder<T>(): Builder<T, never> {
  return new GenericBuilder<T>() as any;
}

// Usage
const user2 = builder<User>()
  .set("id", "123")
  .set("name", "Alice")
  .set("email", "alice@example.com")
  .set("age", 30)
  .build();

// Advanced: Fluent API with method chaining
type FluentAPI<T, Methods extends string> = {
  [M in Methods]: (value: any) => FluentAPI<T, Exclude<Methods, M>>;
} & (Methods extends never ? { execute(): T } : {});

// Progressive disclosure pattern
type StepBuilder<T> = {
  step1: (value: string) => {
    step2: (value: number) => {
      step3: (value: boolean) => {
        build: () => T;
      };
    };
  };
};
```

**Why This Matters:**
- Compile-time validation of builder state
- Prevents calling build() before all fields are set
- Common in DSL implementations
- Used in test builders and fixtures

**Follow-up Questions:**
- How would you handle optional fields?
- How would you implement method order constraints?
- What about conditional fields (if A then B required)?

---

**Question 3: Implement a type-safe event emitter using conditional types for event payload validation.**

**Difficulty:** Senior

**Answer:**

```typescript
// Event map with strongly typed payloads
type EventMap = {
  "user:created": { userId: string; timestamp: Date };
  "user:updated": { userId: string; changes: string[] };
  "user:deleted": { userId: string };
  "data:sync": { recordCount: number; duration: number };
  "error": { message: string; code: number; stack?: string };
};

// Extract event names
type EventName = keyof EventMap;

// Get payload type for an event
type EventPayload<E extends EventName> = EventMap[E];

// Event handler type
type EventHandler<E extends EventName> = (
  payload: EventPayload<E>
) => void | Promise<void>;

// Type-safe event emitter
class TypedEventEmitter {
  private handlers = new Map<EventName, Set<Function>>();

  // Subscribe to event with type-safe payload
  on<E extends EventName>(
    event: E,
    handler: EventHandler<E>
  ): () => void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
    }

    this.handlers.get(event)!.add(handler);

    // Return unsubscribe function
    return () => {
      this.handlers.get(event)?.delete(handler);
    };
  }

  // Emit event with type-safe payload
  emit<E extends EventName>(
    event: E,
    payload: EventPayload<E>
  ): void {
    const handlers = this.handlers.get(event);
    if (handlers) {
      handlers.forEach((handler) => {
        try {
          handler(payload);
        } catch (error) {
          console.error(`Error in handler for ${event}:`, error);
        }
      });
    }
  }

  // Subscribe once
  once<E extends EventName>(
    event: E,
    handler: EventHandler<E>
  ): void {
    const unsubscribe = this.on(event, (payload) => {
      handler(payload);
      unsubscribe();
    });
  }

  // Wait for event
  wait<E extends EventName>(
    event: E,
    timeout?: number
  ): Promise<EventPayload<E>> {
    return new Promise((resolve, reject) => {
      const timer = timeout
        ? setTimeout(() => reject(new Error("Timeout")), timeout)
        : undefined;

      const unsubscribe = this.on(event, (payload) => {
        if (timer) clearTimeout(timer);
        unsubscribe();
        resolve(payload);
      });
    });
  }

  // Remove all listeners for event
  off<E extends EventName>(event: E): void {
    this.handlers.delete(event);
  }

  // Clear all listeners
  clear(): void {
    this.handlers.clear();
  }
}

// Usage
const emitter = new TypedEventEmitter();

// ✅ Type-safe subscription
emitter.on("user:created", (payload) => {
  // payload is typed as { userId: string; timestamp: Date }
  console.log(`User ${payload.userId} created at ${payload.timestamp}`);
});

// ✅ Type-safe emission
emitter.emit("user:created", {
  userId: "123",
  timestamp: new Date()
});

// ❌ Compile errors for mistakes
// emitter.emit("invalid:event", {});  // Error: unknown event
// emitter.emit("user:created", { userId: 123 });  // Error: wrong type
// emitter.on("user:updated", (payload) => {
//   console.log(payload.timestamp);  // Error: timestamp doesn't exist
// });

// Advanced: Conditional event handling based on payload
type ConditionalHandler<E extends EventName> = 
  EventPayload<E> extends { code: number }
    ? (payload: EventPayload<E>, retry: () => void) => void
    : (payload: EventPayload<E>) => void;

class AdvancedEmitter {
  private handlers = new Map<EventName, Set<Function>>();

  on<E extends EventName>(
    event: E,
    handler: ConditionalHandler<E>
  ): () => void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
    }

    this.handlers.get(event)!.add(handler);

    return () => {
      this.handlers.get(event)?.delete(handler);
    };
  }

  emit<E extends EventName>(
    event: E,
    payload: EventPayload<E>
  ): void {
    const handlers = this.handlers.get(event);
    if (handlers) {
      handlers.forEach((handler) => {
        if ("code" in payload) {
          // Payload has code, provide retry function
          handler(payload, () => this.emit(event, payload));
        } else {
          handler(payload);
        }
      });
    }
  }
}

// Wildcard event support
type WildcardEventMap = EventMap & {
  "*": { event: EventName; payload: EventMap[EventName] };
};

class WildcardEmitter {
  private handlers = new Map<keyof WildcardEventMap, Set<Function>>();

  on<E extends keyof WildcardEventMap>(
    event: E,
    handler: E extends "*"
      ? (data: WildcardEventMap["*"]) => void
      : EventHandler<E & EventName>
  ): () => void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
    }

    this.handlers.get(event)!.add(handler);

    return () => {
      this.handlers.get(event)?.delete(handler);
    };
  }

  emit<E extends EventName>(
    event: E,
    payload: EventPayload<E>
  ): void {
    // Emit to specific handlers
    const handlers = this.handlers.get(event);
    if (handlers) {
      handlers.forEach((handler) => handler(payload));
    }

    // Emit to wildcard handlers
    const wildcardHandlers = this.handlers.get("*");
    if (wildcardHandlers) {
      wildcardHandlers.forEach((handler) =>
        handler({ event, payload })
      );
    }
  }
}

const wildcardEmitter = new WildcardEmitter();

// Listen to all events
wildcardEmitter.on("*", ({ event, payload }) => {
  console.log(`Event ${event}:`, payload);
});

// Listen to specific event
wildcardEmitter.on("user:created", (payload) => {
  console.log("User created:", payload.userId);
});
```

**Real-World Applications:**
1. Frontend state management (Redux-like systems)
2. Backend microservices communication
3. WebSocket event handling
4. Plugin systems
5. Test event tracking

**Why This Matters:**
- Prevents event name typos at compile time
- Ensures payload structure matches expectations
- Provides excellent IntelliSense
- Type-safe async event handling
- Common pattern in large applications

**Follow-up Questions:**
- How would you add event priority?
- How would you implement event namespaces?
- What about async event handlers with error handling?

---

### Key Takeaways

- Conditional types enable type-level branching logic (T extends U ? X : Y)
- Distributive conditional types automatically distribute over union types
- The `infer` keyword extracts types within conditional type checks
- Conditional types can be nested for complex type transformations
- Combining mapped types with conditional types creates powerful utilities
- Recursive conditional types enable deep type manipulations
- Conditional types are essential for advanced type-safe patterns
- Understanding conditional types unlocks TypeScript's full power
- Used extensively in utility types and library type definitions
- Master conditional types to create sophisticated type utilities

---

## 2.4 Mapped Types

Mapped types create new types by transforming properties of existing types. They're one of TypeScript's most powerful features for type-level programming.

### Basic Mapped Types

```typescript
// Basic mapping syntax: { [P in K]: T }
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

interface User {
  name: string;
  age: number;
  email: string;
}

type ReadonlyUser = Readonly<User>;
// {
//   readonly name: string;
//   readonly age: number;
//   readonly email: string;
// }

const user: ReadonlyUser = {
  name: "Alice",
  age: 30,
  email: "alice@example.com"
};

// user.name = "Bob";  // ❌ Error: Cannot assign to readonly property

// Make all properties optional
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type PartialUser = Partial<User>;
// {
//   name?: string;
//   age?: number;
//   email?: string;
// }

// Make all properties required (remove optional modifier)
type Required<T> = {
  [P in keyof T]-?: T[P];  // -? removes the optional modifier
};

interface Config {
  host?: string;
  port?: number;
  timeout?: number;
}

type RequiredConfig = Required<Config>;
// {
//   host: string;
//   port: number;
//   timeout: number;
// }

// Make all properties mutable (remove readonly)
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];  // -readonly removes readonly modifier
};

type MutableUser = Mutable<ReadonlyUser>;
// {
//   name: string;
//   age: number;
//   email: string;
// }
```

### Mapping Modifiers

TypeScript provides `+` and `-` modifiers to add or remove property modifiers:

```typescript
// Add optional modifier
type AddOptional<T> = {
  [P in keyof T]+?: T[P];  // +? adds optional (default)
};

// Remove optional modifier
type RemoveOptional<T> = {
  [P in keyof T]-?: T[P];  // -? removes optional
};

// Add readonly modifier
type AddReadonly<T> = {
  +readonly [P in keyof T]: T[P];  // +readonly adds readonly (default)
};

// Remove readonly modifier
type RemoveReadonly<T> = {
  -readonly [P in keyof T]: T[P];  // -readonly removes readonly
};

// Practical example: Making some properties optional
type OptionalKeys<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface Person {
  id: string;
  name: string;
  email: string;
  phone: string;
}

type PersonWithOptionalContact = OptionalKeys<Person, "email" | "phone">;
// {
//   id: string;
//   name: string;
//   email?: string;
//   phone?: string;
// }
```

### Key Remapping with `as`

TypeScript 4.1+ allows remapping keys during mapping:

```typescript
// Basic key remapping
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
//   getEmail: () => string;
// }

// Setters
type Setters<T> = {
  [P in keyof T as `set${Capitalize<string & P>}`]: (value: T[P]) => void;
};

type PersonSetters = Setters<Person>;
// {
//   setName: (value: string) => void;
//   setAge: (value: number) => void;
//   setEmail: (value: string) => void;
// }

// Both getters and setters
type Accessors<T> = Getters<T> & Setters<T>;

// Filter keys based on conditions
type RemovePrivateFields<T> = {
  [P in keyof T as P extends `_${string}` ? never : P]: T[P];
};

interface Internal {
  _id: number;
  _secret: string;
  name: string;
  email: string;
  _timestamp: Date;
}

type Public = RemovePrivateFields<Internal>;
// {
//   name: string;
//   email: string;
// }

// Keep only function properties
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

interface Mixed {
  id: number;
  name: string;
  greet(): string;
  save(): Promise<void>;
  age: number;
}

type OnlyMethods = FunctionProperties<Mixed>;
// {
//   greet: () => string;
//   save: () => Promise<void>;
// }

// Remove function properties
type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

type OnlyData = NonFunctionProperties<Mixed>;
// {
//   id: number;
//   name: string;
//   age: number;
// }
```

### Conditional Mapping

Combine mapped types with conditional types for powerful transformations:

```typescript
// Make properties nullable based on type
type NullableStrings<T> = {
  [P in keyof T]: T[P] extends string ? T[P] | null : T[P];
};

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type UserWithNullableStrings = NullableStrings<User>;
// {
//   id: number;
//   name: string | null;
//   email: string | null;
//   age: number;
// }

// Convert types conditionally
type StringifyNumbers<T> = {
  [P in keyof T]: T[P] extends number ? string : T[P];
};

type StringifiedUser = StringifyNumbers<User>;
// {
//   id: string;
//   name: string;
//   email: string;
//   age: string;
// }

// Make properties optional based on type
type OptionalByType<T, U> = {
  [P in keyof T]: T[P] extends U ? T[P] | undefined : T[P];
};

type UserWithOptionalNumbers = OptionalByType<User, number>;
// {
//   id: number | undefined;
//   name: string;
//   email: string;
//   age: number | undefined;
// }

// Wrap types in arrays conditionally
type Arrayify<T, Condition> = {
  [P in keyof T]: T[P] extends Condition ? T[P][] : T[P];
};

type UserArrays = Arrayify<User, string>;
// {
//   id: number;
//   name: string[];
//   email: string[];
//   age: number;
// }
```

### Recursive Mapped Types

Create deep transformations by recursing into object structures:

```typescript
// Deep Partial
type DeepPartial<T> = T extends object
  ? {
      [P in keyof T]?: DeepPartial<T[P]>;
    }
  : T;

interface Company {
  name: string;
  address: {
    street: string;
    city: string;
    country: {
      name: string;
      code: string;
    };
  };
  employees: {
    name: string;
    role: string;
  }[];
}

type PartialCompany = DeepPartial<Company>;
// All nested properties become optional

const partialCompany: PartialCompany = {
  name: "Acme",
  address: {
    city: "NYC"
    // street and country optional
  }
  // employees optional
};

// Deep Readonly
type DeepReadonly<T> = T extends object
  ? {
      readonly [P in keyof T]: DeepReadonly<T[P]>;
    }
  : T;

type ImmutableCompany = DeepReadonly<Company>;
// All nested properties become readonly

// Deep Required
type DeepRequired<T> = T extends object
  ? {
      [P in keyof T]-?: DeepRequired<T[P]>;
    }
  : T;

// Deep Mutable
type DeepMutable<T> = T extends object
  ? {
      -readonly [P in keyof T]: DeepMutable<T[P]>;
    }
  : T;

// Deep Nullable
type DeepNullable<T> = T extends object
  ? {
      [P in keyof T]: DeepNullable<T[P]> | null;
    }
  : T | null;

type NullableCompany = DeepNullable<Company>;
// All properties (including nested) can be null
```

### Advanced Mapped Patterns

#### Property Value Transformations

```typescript
// Transform all property values
type Transform<T, From, To> = {
  [P in keyof T]: T[P] extends From ? To : T[P];
};

interface Data {
  id: number;
  name: string;
  count: number;
  description: string;
}

type StringData = Transform<Data, number, string>;
// {
//   id: string;
//   name: string;
//   count: string;
//   description: string;
// }

// Wrap values in promises
type Promisify<T> = {
  [P in keyof T]: Promise<T[P]>;
};

type AsyncUser = Promisify<User>;
// {
//   id: Promise<number>;
//   name: Promise<string>;
//   email: Promise<string>;
//   age: Promise<number>;
// }

// Unwrap promises
type Unpromisify<T> = {
  [P in keyof T]: T[P] extends Promise<infer U> ? U : T[P];
};

type SyncUser = Unpromisify<AsyncUser>;
// Back to original User type
```

#### Creating Proxy Types

```typescript
// Create a proxy that logs access
type Proxied<T> = {
  [P in keyof T]: {
    get(): T[P];
    set(value: T[P]): void;
  };
};

type ProxiedUser = Proxied<User>;
// {
//   id: { get(): number; set(value: number): void };
//   name: { get(): string; set(value: string): void };
//   email: { get(): string; set(value: string): void };
//   age: { get(): number; set(value: number): void };
// }

// Observable properties
type Observable<T> = {
  [P in keyof T]: {
    value: T[P];
    subscribe(callback: (value: T[P]) => void): () => void;
  };
};

// Memoized properties
type Memoized<T> = {
  [P in keyof T]: {
    compute(): T[P];
    cached: T[P] | undefined;
  };
};
```

#### Union to Intersection

```typescript
// Convert union to intersection
type UnionToIntersection<U> = (
  U extends any ? (x: U) => void : never
) extends (x: infer I) => void
  ? I
  : never;

type A = { a: string };
type B = { b: number };
type C = { c: boolean };

type Union = A | B | C;
type Intersection = UnionToIntersection<Union>;
// A & B & C

// Merge objects
type Merge<T> = UnionToIntersection<T[keyof T]>;

interface Objects {
  user: { name: string };
  account: { balance: number };
  profile: { bio: string };
}

type Merged = Merge<Objects>;
// { name: string; balance: number; bio: string; }
```

### Frequently Asked Questions

**Q1: What's the difference between mapped types and conditional types?**

**A:** Mapped types transform properties, conditional types make type-level decisions:

```typescript
// Mapped type: Transforms each property
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Conditional type: Makes type decision
type IsString<T> = T extends string ? true : false;

// Combined: Conditional transformation in mapping
type NullableStrings<T> = {
  [P in keyof T]: T[P] extends string ? T[P] | null : T[P];
};
```

**Use mapped types when:** Transforming object properties
**Use conditional types when:** Type-level branching logic
**Combine both for:** Conditional property transformations

---

**Q2: How do I filter properties in a mapped type?**

**A:** Use key remapping with `as` and conditional types:

```typescript
// Remove private properties
type PublicOnly<T> = {
  [P in keyof T as P extends `_${string}` ? never : P]: T[P];
};

// Keep only specific type
type OnlyStrings<T> = {
  [P in keyof T as T[P] extends string ? P : never]: T[P];
};

// Remove readonly properties
type MutableKeys<T> = {
  [P in keyof T as IfEquals<
    { [Q in P]: T[P] },
    { -readonly [Q in P]: T[P] },
    P,
    never
  >]: T[P];
};
```

---

**Q3: Can mapped types work with union types?**

**A:** Yes, but they behave differently than distributive conditional types:

```typescript
type Union = "a" | "b" | "c";

// Mapped type over union
type Mapped = {
  [K in Union]: { key: K; value: string };
};
// Result: {
//   a: { key: "a"; value: string };
//   b: { key: "b"; value: string };
//   c: { key: "c"; value: string };
// }

// Not distributed like conditional types
type Distributed = Union extends string ? Union[] : never;
// Result: "a"[] | "b"[] | "c"[]
```

---

**Q4: How do recursive mapped types work?**

**A:** They call themselves for nested object properties:

```typescript
type DeepPartial<T> = T extends object
  ? {
      [P in keyof T]?: T[P] extends object
        ? DeepPartial<T[P]>  // Recurse
        : T[P];
    }
  : T;

// Each level applies the transformation
interface Nested {
  a: {
    b: {
      c: string;
    };
  };
}

type Partial = DeepPartial<Nested>;
// All levels optional:
// {
//   a?: {
//     b?: {
//       c?: string;
//     };
//   };
// }
```

**Limitations:**
- TypeScript limits recursion depth (50 levels)
- Can cause slow compilation with complex types
- May need explicit type annotations

---

**Q5: What are the performance implications of complex mapped types?**

**A:** Complex mapped types can slow down the compiler:

```typescript
// Simple: Fast
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Complex: Slower
type DeepReadonly<T> = T extends object
  ? {
      readonly [P in keyof T]: T[P] extends object
        ? DeepReadonly<T[P]>
        : T[P] extends Array<infer U>
          ? ReadonlyArray<DeepReadonly<U>>
          : T[P];
    }
  : T;

// Very complex: Can be very slow
type UltraComplex<T> = {
  [P in keyof T as `${string & P}${"A" | "B" | "C"}`]: T[P];
};

// Performance tips:
// 1. Avoid unnecessary recursion
// 2. Use type caching when possible
// 3. Simplify complex string manipulations
// 4. Consider using type aliases for repeated patterns
```

---

### Interview Questions

**Question 1: Implement a mapped type that converts all methods to async versions.**

**Difficulty:** Senior

**Answer:**

```typescript
// Asyncify implementation
type Asyncify<T> = {
  [P in keyof T]: T[P] extends (...args: infer A) => infer R
    ? (...args: A) => Promise<R>
    : T[P];
};

interface UserService {
  getUser(id: string): User;
  createUser(data: CreateUserData): User;
  deleteUser(id: string): boolean;
  count: number;  // Non-function property preserved
}

type AsyncUserService = Asyncify<UserService>;
// {
//   getUser: (id: string) => Promise<User>;
//   createUser: (data: CreateUserData) => Promise<User>;
//   deleteUser: (id: string) => Promise<boolean>;
//   count: number;
// }

// Implementation
class AsyncUserServiceImpl implements AsyncUserService {
  count: number = 0;

  async getUser(id: string): Promise<User> {
    // Async implementation
    return {} as User;
  }

  async createUser(data: CreateUserData): Promise<User> {
    // Async implementation
    return {} as User;
  }

  async deleteUser(id: string): Promise<boolean> {
    // Async implementation
    return true;
  }
}

// Advanced: Handle already-async methods
type AsyncifyDeep<T> = {
  [P in keyof T]: T[P] extends (...args: infer A) => Promise<infer R>
    ? T[P]  // Already async, keep as is
    : T[P] extends (...args: infer A) => infer R
      ? (...args: A) => Promise<R>  // Make async
      : T[P];  // Non-function, preserve
};

// Even more advanced: Preserve this context
type AsyncifyWithThis<T> = {
  [P in keyof T]: T[P] extends (this: infer This, ...args: infer A) => infer R
    ? (this: This, ...args: A) => Promise<R>
    : T[P];
};
```

**Why This Matters:**
- Common when wrapping sync APIs to async
- Used in service layer transformations
- Enables gradual migration to async
- Pattern seen in database adapters

**Follow-up Questions:**
- How would you handle optional methods?
- What about methods that return promises?
- How do you preserve method overloads?

---

**Question 2: Create a type that converts an interface to a form validation schema.**

**Difficulty:** Senior

**Answer:**

```typescript
// Validation rule type
type ValidationRule<T> = {
  required?: boolean;
  min?: T extends number ? number : never;
  max?: T extends number ? number : never;
  minLength?: T extends string ? number : never;
  maxLength?: T extends string ? number : never;
  pattern?: T extends string ? RegExp : never;
  custom?: (value: T) => boolean | string;
};

// Convert to validation schema
type ValidationSchema<T> = {
  [P in keyof T]-?: ValidationRule<T[P]>;
};

interface User {
  name: string;
  email: string;
  age: number;
  password: string;
}

type UserValidation = ValidationSchema<User>;
// {
//   name: ValidationRule<string>;
//   email: ValidationRule<string>;
//   age: ValidationRule<number>;
//   password: ValidationRule<string>;
// }

// Implementation
const userValidation: UserValidation = {
  name: {
    required: true,
    minLength: 2,
    maxLength: 50
  },
  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    custom: (value) => value.includes("@") || "Invalid email"
  },
  age: {
    required: true,
    min: 18,
    max: 120
  },
  password: {
    required: true,
    minLength: 8,
    custom: (value) => /[A-Z]/.test(value) || "Must contain uppercase"
  }
};

// Validator function
function validate<T>(
  data: Partial<T>,
  schema: ValidationSchema<T>
): { valid: boolean; errors: Partial<Record<keyof T, string[]>> } {
  const errors: Partial<Record<keyof T, string[]>> = {};

  for (const key in schema) {
    const rules = schema[key];
    const value = data[key];
    const fieldErrors: string[] = [];

    if (rules.required && value === undefined) {
      fieldErrors.push("Field is required");
    }

    if (value !== undefined) {
      // Type-specific validations
      if (typeof value === "string") {
        if (rules.minLength && value.length < rules.minLength) {
          fieldErrors.push(`Minimum length is ${rules.minLength}`);
        }
        if (rules.maxLength && value.length > rules.maxLength) {
          fieldErrors.push(`Maximum length is ${rules.maxLength}`);
        }
        if (rules.pattern && !rules.pattern.test(value)) {
          fieldErrors.push("Pattern validation failed");
        }
      }

      if (typeof value === "number") {
        if (rules.min !== undefined && value < rules.min) {
          fieldErrors.push(`Minimum value is ${rules.min}`);
        }
        if (rules.max !== undefined && value > rules.max) {
          fieldErrors.push(`Maximum value is ${rules.max}`);
        }
      }

      // Custom validation
      if (rules.custom) {
        const result = rules.custom(value as any);
        if (typeof result === "string") {
          fieldErrors.push(result);
        } else if (!result) {
          fieldErrors.push("Custom validation failed");
        }
      }
    }

    if (fieldErrors.length > 0) {
      errors[key] = fieldErrors;
    }
  }

  return {
    valid: Object.keys(errors).length === 0,
    errors
  };
}

// Usage
const userData: Partial<User> = {
  name: "A",  // Too short
  email: "invalid",  // Invalid format
  age: 15,  // Too young
  password: "weak"  // No uppercase
};

const result = validate(userData, userValidation);
console.log(result);
// {
//   valid: false,
//   errors: {
//     name: ["Minimum length is 2"],
//     email: ["Pattern validation failed", "Invalid email"],
//     age: ["Minimum value is 18"],
//     password: ["Minimum length is 8", "Must contain uppercase"]
//   }
// }

// Advanced: Nested validation
type DeepValidationSchema<T> = {
  [P in keyof T]-?: T[P] extends object
    ? DeepValidationSchema<T[P]>
    : ValidationRule<T[P]>;
};

interface Profile {
  user: User;
  settings: {
    theme: string;
    notifications: boolean;
  };
}

type ProfileValidation = DeepValidationSchema<Profile>;
// Validates nested objects
```

**Why This Matters:**
- Common pattern in form libraries
- Type-safe validation schemas
- Ensures validation rules match data structure
- Used in API input validation

**Follow-up Questions:**
- How would you handle array properties?
- What about conditional validation (if X then Y required)?
- How would you implement async validators?

---

### Key Takeaways

- Mapped types transform all properties of a type systematically
- Modifiers (`readonly`, `?`, `-readonly`, `-?`) add or remove property modifiers
- Key remapping with `as` enables filtering and transforming property names
- Combine mapped types with conditional types for conditional transformations
- Recursive mapped types enable deep transformations of nested structures
- Mapped types are essential for creating flexible utility types
- Understanding mapped types unlocks advanced type-level programming
- Performance matters with complex mapped types in large codebases
- Mapped types power many built-in TypeScript utilities
- Master mapped types to create sophisticated type transformations

---

## 2.5 Template Literal Types

Template literal types enable string manipulation at the type level, creating powerful patterns for type-safe string operations.

### Basic Template Literal Types

```typescript
// Basic syntax
type Greeting = `Hello ${string}`;

let valid: Greeting = "Hello World";  // ✅ OK
let invalid: Greeting = "Hi World";   // ❌ Error

// Combining string literals
type Color = "red" | "green" | "blue";
type Shade = "light" | "dark";
type ColorVariant = `${Shade}-${Color}`;
// Result: "light-red" | "light-green" | "light-blue" | 
//         "dark-red" | "dark-green" | "dark-blue"

// Event handler names
type EventType = "click" | "focus" | "blur" | "change";
type EventHandler = `on${Capitalize<EventType>}`;
// Result: "onClick" | "onFocus" | "onBlur" | "onChange"

// CSS properties
type Side = "top" | "right" | "bottom" | "left";
type Property = "margin" | "padding" | "border";
type CSSProperty = `${Property}-${Side}`;
// Result: "margin-top" | "margin-right" | ... | "border-left"

// HTTP methods
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
type Route = `/api/${string}`;
type Endpoint = `${HTTPMethod} ${Route}`;
```

### Intrinsic String Manipulation Types

TypeScript provides built-in utilities for string manipulation:

```typescript
// Uppercase
type Loud = Uppercase<"hello world">;  // "HELLO WORLD"

// Lowercase
type Quiet = Lowercase<"HELLO WORLD">;  // "hello world"

// Capitalize (first letter only)
type Cap = Capitalize<"hello world">;  // "Hello world"

// Uncapitalize
type Uncap = Uncapitalize<"Hello World">;  // "hello World"

// Practical combinations
type HTTPMethodUpper = Uppercase<HTTPMethod>;
// "GET" | "POST" | "PUT" | "DELETE" | "PATCH"

type EntityName = "user" | "post" | "comment";
type EntityController = `${Capitalize<EntityName>}Controller`;
// "UserController" | "PostController" | "CommentController"

// Generate method names
type CRUDAction = "create" | "read" | "update" | "delete";
type CRUDMethod<Entity extends string> = 
  `${CRUDAction}${Capitalize<Entity>}`;

type UserMethods = CRUDMethod<"user">;
// "createUser" | "readUser" | "updateUser" | "deleteUser"
```

### Pattern Matching with Template Literals

Extract and infer parts of string patterns:

```typescript
// Extract ID from string
type ExtractId<T> = T extends `user-${infer ID}` ? ID : never;

type Id1 = ExtractId<"user-123">;  // "123"
type Id2 = ExtractId<"user-abc">;  // "abc"
type Id3 = ExtractId<"post-123">;  // never

// Parse URL paths
type ParsePath<T> = T extends `${infer First}/${infer Rest}`
  ? [First, ...ParsePath<Rest>]
  : [T];

type Path = ParsePath<"users/123/posts/456">;
// ["users", "123", "posts", "456"]

// Extract file extension
type GetExtension<T> = T extends `${string}.${infer Ext}`
  ? Ext
  : never;

type Ext1 = GetExtension<"file.txt">;     // "txt"
type Ext2 = GetExtension<"image.png">;    // "png"
type Ext3 = GetExtension<"doc.backup.pdf">;  // "pdf" (last extension)

// Extract domain from email
type GetDomain<T> = T extends `${string}@${infer Domain}`
  ? Domain
  : never;

type Domain = GetDomain<"user@example.com">;  // "example.com"

// Parse query parameters
type QueryParam = `?${string}=${string}`;
type ParseQuery<T extends QueryParam> = 
  T extends `?${infer Key}=${infer Value}`
    ? { [K in Key]: Value }
    : never;

type Query = ParseQuery<"?id=123">;  // { id: "123" }
```

### Practical Template Literal Patterns

#### Type-Safe Routes

```typescript
// Define route structure
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type ResourceName = "users" | "posts" | "comments";
type Route = `/${ResourceName}` | `/${ResourceName}/:id`;

// Full endpoint definition
type Endpoint = `${HTTPMethod} ${Route}`;

// Example endpoints
type ValidEndpoints = 
  | "GET /users"
  | "GET /users/:id"
  | "POST /users"
  | "PUT /users/:id"
  | "DELETE /users/:id"
  | "GET /posts"
  | "GET /posts/:id"
  | "POST /posts"
  | "PUT /posts/:id"
  | "DELETE /posts/:id";

// Type-safe route handler
function handle<E extends Endpoint>(
  endpoint: E,
  handler: (req: Request) => Response
): void {
  // Register handler
}

handle("GET /users", (req) => {
  // Handle GET /users
  return new Response();
});

// handle("INVALID /users", handler);  // ❌ Error: Invalid endpoint
```

#### API Endpoint Generator

```typescript
// Generate CRUD endpoints
type CRUDEndpoints<Resource extends string> = 
  | `GET /${Resource}`
  | `GET /${Resource}/:id`
  | `POST /${Resource}`
  | `PUT /${Resource}/:id`
  | `PATCH /${Resource}/:id`
  | `DELETE /${Resource}/:id`;

type UserEndpoints = CRUDEndpoints<"users">;

// With nested resources
type NestedResource<Parent extends string, Child extends string> = 
  | `GET /${Parent}/:${Parent}Id/${Child}`
  | `GET /${Parent}/:${Parent}Id/${Child}/:id`
  | `POST /${Parent}/:${Parent}Id/${Child}`
  | `PUT /${Parent}/:${Parent}Id/${Child}/:id`
  | `DELETE /${Parent}/:${Parent}Id/${Child}/:id`;

type UserPostEndpoints = NestedResource<"users", "posts">;
// "GET /users/:usersId/posts" | "GET /users/:usersId/posts/:id" | ...
```

#### CSS-in-JS Type Safety

```typescript
// CSS property names
type CSSProperty = "width" | "height" | "margin" | "padding";
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSValue = `${number}${CSSUnit}`;

type CSSRule = `${CSSProperty}: ${CSSValue}`;

// Valid rules
const rule1: CSSRule = "width: 100px";
const rule2: CSSRule = "margin: 2em";
const rule3: CSSRule = "padding: 5%";

// Generate CSS classes
type Breakpoint = "sm" | "md" | "lg" | "xl";
type UtilityClass = "text" | "bg" | "border";
type Color = "red" | "blue" | "green";

type ResponsiveUtility = `${Breakpoint}:${UtilityClass}-${Color}`;
// "sm:text-red" | "sm:text-blue" | ... | "xl:border-green"
```

#### State Machine Types

```typescript
// Define states and transitions
type State = "idle" | "loading" | "success" | "error";
type Transition = `${State}→${State}`;

type ValidTransitions = 
  | "idle→loading"
  | "loading→success"
  | "loading→error"
  | "success→idle"
  | "error→idle"
  | "error→loading";

// Type-safe transition function
function transition(from: State, to: State): Transition {
  const trans = `${from}→${to}` as Transition;
  // Validate transition
  return trans;
}
```

### Frequently Asked Questions

**Q1: How do template literal types differ from string literals?**

**A:** Template literals can include type parameters and wildcards, string literals are exact:

```typescript
// String literal: Exact match
type Exact = "hello";
let a: Exact = "hello";  // ✅ OK
let b: Exact = "Hello";  // ❌ Error

// Template literal: Pattern match
type Pattern = `hello ${string}`;
let c: Pattern = "hello world";  // ✅ OK
let d: Pattern = "hello there";  // ✅ OK
let e: Pattern = "hi world";     // ❌ Error

// With union: Generates combinations
type Color = "red" | "blue";
type Size = "sm" | "lg";
type Class = `${Color}-${Size}`;
// "red-sm" | "red-lg" | "blue-sm" | "blue-lg"
```

---

**Q2: Can I use template literals to transform property names?**

**A:** Yes, combine them with mapped types:

```typescript
interface User {
  firstName: string;
  lastName: string;
  age: number;
}

// Add "get" prefix
type Getters = {
  [K in keyof User as `get${Capitalize<K & string>}`]: () => User[K];
};
// {
//   getFirstName: () => string;
//   getLastName: () => string;
//   getAge: () => number;
// }

// Transform kebab-case to camelCase (conceptually)
type KebabToCamel<S extends string> = 
  S extends `${infer First}-${infer Rest}`
    ? `${First}${Capitalize<KebabToCamel<Rest>>}`
    : S;

type Camel = KebabToCamel<"user-first-name">;  // "userFirstName"
```

---

**Q3: How do I extract multiple parts from a template literal?**

**A:** Use multiple `infer` keywords:

```typescript
// Extract parts
type ParseURL<T> = T extends `https://${infer Domain}/${infer Path}`
  ? { domain: Domain; path: Path }
  : never;

type URL = ParseURL<"https://example.com/users/123">;
// { domain: "example.com"; path: "users/123" }

// Extract query params
type ParseQuery<T> = T extends `${infer Path}?${infer Query}`
  ? { path: Path; query: Query }
  : { path: T; query: never };

type Q = ParseQuery<"/users?id=123">;
// { path: "/users"; query: "id=123" }
```

---

### Key Takeaways

- Template literal types enable string pattern matching at type level
- Intrinsic utilities (Uppercase, Lowercase, Capitalize, Uncapitalize) transform strings
- Combine with unions to generate all possible combinations
- Extract parts using `infer` in pattern matching
- Essential for type-safe string APIs (routes, CSS, events)
- Use with mapped types for property name transformations
- Common in modern TypeScript libraries and frameworks
- Enables compile-time string validation

---

## 2.6 Type Guards and Type Predicates

Type guards enable runtime type checking that TypeScript understands, narrowing types based on runtime conditions.

### Built-in Type Guards

```typescript
// typeof
function processValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());  // value is string
  } else {
    console.log(value.toFixed(2));     // value is number
  }
}

// instanceof
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();  // animal is Dog
  } else {
    animal.meow();  // animal is Cat
  }
}

// in operator
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();  // animal is Fish
  } else {
    animal.fly();   // animal is Bird
  }
}

// Truthiness narrowing
function printName(name: string | null | undefined) {
  if (name) {
    console.log(name.toUpperCase());  // name is string
  } else {
    console.log("No name");  // name is null | undefined
  }
}

// Equality narrowing
function compare(x: string | number, y: string | boolean) {
  if (x === y) {
    // x and y are both string (only common type)
    console.log(x.toUpperCase());
    console.log(y.toUpperCase());
  }
}
```

### User-Defined Type Guards

Type predicates (`value is Type`) create custom type guards:

```typescript
// Basic type predicate
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase());  // value is string
  }
}

// Object type guard
interface User {
  name: string;
  email: string;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    typeof (value as User).name === "string" &&
    "email" in value &&
    typeof (value as User).email === "string"
  );
}

function processUser(data: unknown) {
  if (isUser(data)) {
    console.log(data.name, data.email);  // data is User
  }
}

// Generic type guard
function isArrayOf<T>(
  value: unknown,
  check: (item: unknown) => item is T
): value is T[] {
  return Array.isArray(value) && value.every(check);
}

// Usage
if (isArrayOf(data, isString)) {
  data.forEach(str => console.log(str.toUpperCase()));
}

// Discriminated union guard
type Success = { status: "success"; data: string };
type Failure = { status: "failure"; error: string };
type Result = Success | Failure;

function isSuccess(result: Result): result is Success {
  return result.status === "success";
}

function handleResult(result: Result) {
  if (isSuccess(result)) {
    console.log(result.data);   // result is Success
  } else {
    console.log(result.error);  // result is Failure
  }
}
```

### Assertion Functions

Assert functions throw if condition fails and narrow types:

```typescript
// Basic assertion
function assert(condition: unknown, message: string): asserts condition {
  if (!condition) {
    throw new Error(message);
  }
}

function processValue(value: unknown) {
  assert(typeof value === "string", "Value must be string");
  // After assertion, value is string
  console.log(value.toUpperCase());
}

// Type assertion function
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Not a string");
  }
}

function handle(data: unknown) {
  assertIsString(data);
  // data is string after assertion
  console.log(data.length);
}

// Assert with type predicate
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error("Invalid user object");
  }
}

function processUserData(data: unknown) {
  assertIsUser(data);
  // data is User after assertion
  console.log(data.name);
}

// Assert non-null
function assertNonNull<T>(
  value: T | null | undefined,
  message?: string
): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(message || "Value is null or undefined");
  }
}

function getValue(): string | null {
  return Math.random() > 0.5 ? "value" : null;
}

const value = getValue();
assertNonNull(value, "Value must not be null");
// value is string after assertion
console.log(value.toUpperCase());
```

### Advanced Type Guard Patterns

```typescript
// Composite type guards
function isValidEmail(value: unknown): value is string {
  return (
    typeof value === "string" &&
    value.includes("@") &&
    value.length > 3
  );
}

function isValidUser(value: unknown): value is User {
  return (
    isUser(value) &&
    isValidEmail(value.email)
  );
}

// Narrowing with multiple guards
function processData(data: unknown) {
  if (isString(data)) {
    console.log("String:", data);
  } else if (isArrayOf(data, isString)) {
    console.log("String array:", data);
  } else if (isUser(data)) {
    console.log("User:", data.name);
  } else {
    console.log("Unknown type");
  }
}

// Type guard with generics
function hasProperty<K extends string>(
  obj: unknown,
  key: K
): obj is Record<K, unknown> {
  return typeof obj === "object" && obj !== null && key in obj;
}

function getProperty<K extends string>(obj: unknown, key: K): unknown {
  if (hasProperty(obj, key)) {
    return obj[key];  // TypeScript knows obj has key
  }
  return undefined;
}

// Recursive type guard
type NestedArray<T> = T | NestedArray<T>[];

function flatten<T>(
  arr: NestedArray<T>[],
  isType: (value: unknown) => value is T
): T[] {
  const result: T[] = [];
  
  for (const item of arr) {
    if (isType(item)) {
      result.push(item);
    } else if (Array.isArray(item)) {
      result.push(...flatten(item, isType));
    }
  }
  
  return result;
}

const nested: NestedArray<number>[] = [1, [2, [3, [4]]], 5];
const flat = flatten(nested, (v): v is number => typeof v === "number");
// [1, 2, 3, 4, 5]
```

---

## 2.7 Decorators

Decorators provide a way to add annotations and meta-programming syntax for class declarations and members.

### Enabling Decorators

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### Class Decorators

```typescript
// Simple class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
}

// Decorator factory
function Component(config: { selector: string; template: string }) {
  return function (constructor: Function) {
    constructor.prototype.config = config;
  };
}

@Component({
  selector: "app-user",
  template: "<div>User Component</div>"
})
class UserComponent {
  // ...
}

// Class replacement
function Injectable<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    injected = true;
  };
}

@Injectable
class Service {
  // ...
}

const service = new Service();
console.log((service as any).injected);  // true
```

### Method Decorators

```typescript
// Log method calls
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

// Measure execution time
function measure(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  
  descriptor.value = async function (...args: any[]) {
    const start = performance.now();
    const result = await originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };
  
  return descriptor;
}

class DataService {
  @measure
  async fetchData(): Promise<any[]> {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    return [];
  }
}

// Memoization
function memoize(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  const cache = new Map<string, any>();
  
  descriptor.value = function (...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

class MathService {
  @memoize
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}
```

### Property Decorators

```typescript
// Readonly property
function readonly(target: any, propertyKey: string) {
  Object.defineProperty(target, propertyKey, {
    writable: false,
    configurable: false
  });
}

class Person {
  @readonly
  id: string = "123";
}

// Validation
function validate(validationFn: (value: any) => boolean) {
  return function (target: any, propertyKey: string) {
    let value: any;
    
    Object.defineProperty(target, propertyKey, {
      get() {
        return value;
      },
      set(newValue: any) {
        if (!validationFn(newValue)) {
          throw new Error(`Validation failed for ${propertyKey}`);
        }
        value = newValue;
      },
      enumerable: true,
      configurable: true
    });
  };
}

class User {
  @validate(v => typeof v === "string" && v.length > 0)
  name!: string;
  
  @validate(v => typeof v === "number" && v >= 18)
  age!: number;
}
```

---

## 2.8 Advanced Type Patterns

### Option/Maybe Pattern

```typescript
// Option type for null safety
type Option<T> = Some<T> | None;

interface Some<T> {
  readonly _tag: "Some";
  readonly value: T;
}

interface None {
  readonly _tag: "None";
}

function some<T>(value: T): Option<T> {
  return { _tag: "Some", value };
}

function none<T = never>(): Option<T> {
  return { _tag: "None" };
}

function map<T, U>(option: Option<T>, fn: (value: T) => U): Option<U> {
  return option._tag === "Some" ? some(fn(option.value)) : none();
}

function getOrElse<T>(option: Option<T>, defaultValue: T): T {
  return option._tag === "Some" ? option.value : defaultValue;
}

// Usage
const value = some(5);
const doubled = map(value, n => n * 2);
const result = getOrElse(doubled, 0);  // 10
```

### Builder Pattern

```typescript
class RequestBuilder {
  private url?: string;
  private method?: string;
  private headers?: Record<string, string>;
  private body?: any;

  setUrl(url: string): this {
    this.url = url;
    return this;
  }

  setMethod(method: string): this {
    this.method = method;
    return this;
  }

  setHeaders(headers: Record<string, string>): this {
    this.headers = headers;
    return this;
  }

  setBody(body: any): this {
    this.body = body;
    return this;
  }

  build(): Request {
    if (!this.url || !this.method) {
      throw new Error("URL and method are required");
    }
    return new Request(this.url, {
      method: this.method,
      headers: this.headers,
      body: this.body ? JSON.stringify(this.body) : undefined
    });
  }
}

const request = new RequestBuilder()
  .setUrl("https://api.example.com/users")
  .setMethod("POST")
  .setHeaders({ "Content-Type": "application/json" })
  .setBody({ name: "Alice" })
  .build();
```

### Phantom Types

```typescript
// Phantom type for compile-time state tracking
type Unvalidated = { readonly __brand: unique symbol };
type Validated = { readonly __brand: unique symbol };

interface Email<State> {
  readonly value: string;
  readonly state: State;
}

function createEmail(value: string): Email<Unvalidated> {
  return { value, state: null as any };
}

function validateEmail(email: Email<Unvalidated>): Email<Validated> | null {
  if (email.value.includes("@")) {
    return { value: email.value, state: null as any };
  }
  return null;
}

function sendEmail(email: Email<Validated>): void {
  console.log(`Sending email to ${email.value}`);
}

// Usage
const email = createEmail("user@example.com");
const validated = validateEmail(email);

if (validated) {
  sendEmail(validated);  // ✅ OK: validated email
}
// sendEmail(email);  // ❌ Error: unvalidated email
```

---

## Conclusion of Part 2

This completes Part 2 of the TypeScript Mastery series covering advanced type system features.

**Topics Covered:**
✅ Generics with constraints and patterns
✅ Utility types (built-in and custom)
✅ Conditional types with infer
✅ Mapped types and transformations
✅ Template literal types
✅ Type guards and assertions
✅ Decorators
✅ Advanced patterns

**Continue to Part 3: TypeScript Compiler & Tooling!**

---

**Total Word Count:** ~25,000 words
