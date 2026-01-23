---
title: "TypeScript Quick Reference Cheat Sheet"
date: 2026-01-08 18:30:00 +0530
categories: [Programming, TypeScript, Reference]
tags: [typescript, cheat-sheet, quick-reference, syntax]
pin: true
---

# TypeScript Quick Reference Cheat Sheet

> One-page summary of essential TypeScript concepts - Print this for quick reference!

---

## 🎯 Basic Types

```typescript
// Primitives
let name: string = "John"
let age: number = 30
let active: boolean = true
let anything: any = "flexible"
let safe: unknown = "type-safe any"
let nothing: void = undefined
let never: never = throw new Error()
let undef: undefined = undefined
let nil: null = null

// Arrays
let numbers: number[] = [1, 2, 3]
let strings: Array<string> = ["a", "b"]

// Tuples
let tuple: [string, number] = ["age", 30]
let named: [name: string, age: number] = ["John", 30]

// Objects
let person: { name: string; age: number } = {
  name: "John",
  age: 30
}

// Enum
enum Color { Red, Green, Blue }
enum Status { Active = "ACTIVE", Inactive = "INACTIVE" }

// Literal Types
let direction: "up" | "down" = "up"
let code: 200 | 404 | 500 = 200

// Union & Intersection
type StringOrNumber = string | number
type Combined = TypeA & TypeB
```

---

## 🔧 Functions

```typescript
// Basic function
function add(a: number, b: number): number {
  return a + b
}

// Arrow function
const multiply = (a: number, b: number): number => a * b

// Optional parameters
function greet(name: string, greeting?: string): string {
  return `${greeting || "Hello"}, ${name}`
}

// Default parameters
function power(base: number, exponent: number = 2): number {
  return Math.pow(base, exponent)
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0)
}

// Function types
type MathOp = (a: number, b: number) => number
const divide: MathOp = (a, b) => a / b

// Overloads
function process(x: string): string
function process(x: number): number
function process(x: string | number): string | number {
  return typeof x === "string" ? x.toUpperCase() : x * 2
}

// Generic function
function identity<T>(value: T): T {
  return value
}

// Async function
async function fetchData(url: string): Promise<Response> {
  return await fetch(url)
}
```

---

## 📦 Interfaces & Types

```typescript
// Interface
interface User {
  id: string
  name: string
  email: string
  age?: number
  readonly created: Date
}

// Extending interface
interface Admin extends User {
  role: "admin"
  permissions: string[]
}

// Type alias
type ID = string | number
type Point = { x: number; y: number }

// Union types
type Status = "pending" | "approved" | "rejected"
type Result = Success | Error

// Intersection types
type Employee = Person & Worker
type Combined = { a: string } & { b: number }

// Interface vs Type
// Interface: extendable, declaration merging
// Type: unions, intersections, computed properties

// Index signature
interface Dictionary {
  [key: string]: string
}

// Call signature
interface SearchFunc {
  (source: string, search: string): boolean
}

// Construct signature
interface ClockConstructor {
  new (hour: number, minute: number): Clock
}
```

---

## 🧩 Generics

```typescript
// Generic function
function wrap<T>(value: T): T[] {
  return [value]
}

// Generic interface
interface Box<T> {
  value: T
}

// Generic class
class Container<T> {
  constructor(private value: T) {}
  getValue(): T {
    return this.value
  }
}

// Generic constraints
function getLength<T extends { length: number }>(item: T): number {
  return item.length
}

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second]
}

// Default type parameters
interface Response<T = any> {
  data: T
  status: number
}

// Generic constraints with keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```

---

## 🎨 Advanced Types

```typescript
// Conditional types
type IsString<T> = T extends string ? true : false

// Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P]
}

type Optional<T> = {
  [P in keyof T]?: T[P]
}

// Template literal types
type EventName<T extends string> = `${T}Changed`
type Greeting = `Hello ${string}`

// Utility types
type PartialUser = Partial<User>        // All optional
type RequiredUser = Required<User>      // All required
type PickedUser = Pick<User, "id" | "name">  // Select props
type OmittedUser = Omit<User, "email">  // Exclude props
type ReadonlyUser = Readonly<User>      // All readonly
type RecordType = Record<string, number>  // Key-value

// Type inference
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never
type Parameters<T> = T extends (...args: infer P) => any ? P : never

// Discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number }
  | { kind: "rectangle"; width: number; height: number }
```

---

## 🛡️ Type Guards & Narrowing

```typescript
// typeof guard
function print(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase())
  } else {
    console.log(value.toFixed(2))
  }
}

// instanceof guard
if (error instanceof Error) {
  console.log(error.message)
}

// in operator
if ("swim" in animal) {
  animal.swim()
}

// Type predicate
function isString(value: unknown): value is string {
  return typeof value === "string"
}

// Assertion functions
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) throw new Error(msg)
}

// Discriminated unions
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2
    case "square":
      return shape.size ** 2
    case "rectangle":
      return shape.width * shape.height
  }
}

// Nullish coalescing & optional chaining
const name = user?.name ?? "Guest"
const email = user?.contact?.email
```

---

## 🏗️ Classes

```typescript
// Basic class
class Person {
  name: string
  private age: number
  protected role: string
  readonly id: string

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
    this.id = Math.random().toString()
  }

  greet(): string {
    return `Hello, I'm ${this.name}`
  }
}

// Inheritance
class Employee extends Person {
  constructor(name: string, age: number, private salary: number) {
    super(name, age)
  }

  getSalary(): number {
    return this.salary
  }
}

// Abstract class
abstract class Animal {
  abstract makeSound(): string
  
  move(): void {
    console.log("Moving...")
  }
}

// Static members
class MathUtils {
  static PI = 3.14159
  
  static square(x: number): number {
    return x * x
  }
}

// Getters & Setters
class User {
  private _name: string
  
  get name(): string {
    return this._name
  }
  
  set name(value: string) {
    this._name = value.trim()
  }
}

// Implements interface
interface Drawable {
  draw(): void
}

class Circle implements Drawable {
  draw(): void {
    console.log("Drawing circle")
  }
}
```

---

## 📦 Modules

```typescript
// Named exports
export const PI = 3.14
export function add(a: number, b: number) {
  return a + b
}
export class Calculator {}

// Default export
export default class MyClass {}

// Import
import MyClass from "./MyClass"
import { PI, add } from "./math"
import * as math from "./math"
import { add as sum } from "./math"

// Re-export
export { PI } from "./constants"
export * from "./utils"

// Type-only imports/exports
import type { User } from "./types"
export type { User } from "./types"

// Dynamic import
const module = await import("./module")
```

---

## 🎯 Decorators

```typescript
// Class decorator
@sealed
class Greeter {
  greeting: string
}

function sealed(constructor: Function) {
  Object.seal(constructor)
  Object.seal(constructor.prototype)
}

// Method decorator
class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b
  }
}

function log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${key} with`, args)
    return original.apply(this, args)
  }
}

// Property decorator
class User {
  @readonly
  name: string
}

// Parameter decorator
class Service {
  method(@required param: string) {}
}
```

---

## ⚙️ tsconfig.json Essentials

```json
{
  "compilerOptions": {
    // Target & Module
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    
    // Strict Type Checking
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    
    // Module Resolution
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "resolveJsonModule": true,
    "esModuleInterop": true,
    
    // Emit
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true,
    "removeComments": true,
    
    // Advanced
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 🔍 Common Patterns

```typescript
// Singleton
class Singleton {
  private static instance: Singleton
  private constructor() {}
  
  static getInstance(): Singleton {
    if (!this.instance) {
      this.instance = new Singleton()
    }
    return this.instance
  }
}

// Factory
interface Product {}
class ConcreteProductA implements Product {}
class ConcreteProductB implements Product {}

class Factory {
  createProduct(type: string): Product {
    switch (type) {
      case "A": return new ConcreteProductA()
      case "B": return new ConcreteProductB()
      default: throw new Error("Invalid type")
    }
  }
}

// Observer
interface Observer {
  update(data: any): void
}

class Subject {
  private observers: Observer[] = []
  
  attach(observer: Observer): void {
    this.observers.push(observer)
  }
  
  notify(data: any): void {
    this.observers.forEach(o => o.update(data))
  }
}

// Repository
interface Repository<T> {
  findById(id: string): Promise<T | null>
  findAll(): Promise<T[]>
  create(data: T): Promise<T>
  update(id: string, data: Partial<T>): Promise<T>
  delete(id: string): Promise<void>
}
```

---

## ⚛️ React Patterns

```typescript
// Functional component
interface Props {
  name: string
  age?: number
  onClick: () => void
}

const MyComponent: React.FC<Props> = ({ name, age, onClick }) => {
  return <div onClick={onClick}>{name}</div>
}

// Hooks
const [count, setCount] = useState<number>(0)
const [user, setUser] = useState<User | null>(null)

useEffect(() => {
  // Effect logic
  return () => {
    // Cleanup
  }
}, [dependency])

const ref = useRef<HTMLDivElement>(null)
const inputRef = useRef<HTMLInputElement>(null)

// Custom hook
function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key)
    return stored ? JSON.parse(stored) : initial
  })
  
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value))
  }, [key, value])
  
  return [value, setValue] as const
}

// Context
interface ThemeContextType {
  theme: "light" | "dark"
  toggleTheme: () => void
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) throw new Error("useTheme must be used within ThemeProvider")
  return context
}
```

---

## 🎯 Best Practices

### ✅ Do's
```typescript
// Use const assertions
const config = {
  api: "https://api.example.com",
  timeout: 5000
} as const

// Use readonly for immutable data
interface Config {
  readonly apiKey: string
  readonly endpoints: readonly string[]
}

// Use discriminated unions
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string }

// Use type guards
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value
}

// Use utility types
type UserUpdate = Partial<Pick<User, "name" | "email">>
```

### ❌ Don'ts
```typescript
// Avoid any
let data: any // ❌ Bad

// Avoid non-null assertion unless sure
const value = maybeNull! // ❌ Risky

// Avoid type assertions
const user = data as User // ❌ Unsafe

// Avoid ignoring errors
// @ts-ignore // ❌ Bad practice

// Avoid overly complex types
type Complex = (A & B) | (C & D) | (E & F) // ❌ Hard to maintain
```

---

## 🚀 Quick Commands

```bash
# Install TypeScript
npm install -g typescript

# Initialize tsconfig
tsc --init

# Compile
tsc
tsc file.ts
tsc --watch

# Run with ts-node
npx ts-node file.ts

# Check types only
tsc --noEmit

# Generate declarations
tsc --declaration

# Show config
tsc --showConfig
```

---

## 📚 Utility Types Reference

```typescript
Partial<T>          // All properties optional
Required<T>         // All properties required
Readonly<T>         // All properties readonly
Pick<T, K>          // Select specific properties
Omit<T, K>          // Exclude specific properties
Record<K, T>        // Object with keys K and values T
Exclude<T, U>       // Remove types from union
Extract<T, U>       // Extract types from union
NonNullable<T>      // Remove null and undefined
Parameters<T>       // Function parameter types as tuple
ReturnType<T>       // Function return type
InstanceType<T>     // Instance type of constructor
Awaited<T>          // Unwrap Promise type
```

---

## 🎨 Type Manipulation

```typescript
// keyof - Get all keys
type UserKeys = keyof User // "id" | "name" | "email"

// typeof - Get type of value
const config = { api: "url", timeout: 5000 }
type Config = typeof config

// in - Iterate over keys
type Flags = { [K in "read" | "write"]: boolean }

// extends - Conditional
type IsNumber<T> = T extends number ? true : false

// infer - Extract type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never

// satisfies - Type checking
const config = {
  api: "url",
  timeout: 5000
} satisfies Config
```

---

## 💡 Pro Tips

1. **Enable strict mode** - Catches more errors
2. **Use unknown over any** - Type-safe alternative
3. **Leverage type inference** - Let TS infer when possible
4. **Use const assertions** - More specific types
5. **Create utility types** - Reusable type logic
6. **Use discriminated unions** - Safe type narrowing
7. **Avoid type assertions** - Prefer type guards
8. **Use branded types** - Nominal typing simulation
9. **Leverage template literals** - String manipulation
10. **Read compiler errors** - They're very helpful!

---

**Print this cheat sheet for quick reference while coding!** 🚀

*TypeScript Cheat Sheet v1.0 - Complete TypeScript Mastery Series*
