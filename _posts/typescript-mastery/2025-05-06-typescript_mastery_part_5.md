---
title: "TypeScript Mastery - Part 5: Advanced Topics and Best Practices"
date: 2025-05-06 00:00:00 +0530
categories: [TypeScript, TS Mastery]
tags: [TypeScript, JavaScript, Programming, Web Development, Design-patterns, Architecture, Performance, Security, Best-practices, Advanced]
---

# Complete TypeScript Mastery Part 5: Advanced Topics and Best Practices

## Introduction

Welcome to Part 5, the final part of the Complete TypeScript Mastery series! In the previous parts, we covered fundamentals, advanced type system features, compiler tooling, and practical applications with frameworks. Now we'll explore advanced topics, design patterns, architecture, performance optimization, and production best practices.

This part focuses on mastery-level topics:
- Advanced design patterns in TypeScript
- Architectural patterns and principles
- Performance optimization techniques
- Security best practices
- Metaprogramming and reflection
- Advanced asynchronous patterns
- Functional programming with TypeScript
- Large-scale application architecture
- Production deployment and monitoring
- Code quality and maintainability

By the end of this part, you'll have expert-level knowledge to architect, build, and maintain large-scale TypeScript applications with confidence.

---

## 5.1 Design Patterns in TypeScript

### Creational Patterns

#### Singleton Pattern

```typescript
// Basic Singleton
class Database {
  private static instance: Database;
  private connectionString: string;

  private constructor() {
    this.connectionString = process.env.DATABASE_URL || "";
  }

  public static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  public query(sql: string): Promise<any[]> {
    console.log(`Executing: ${sql}`);
    return Promise.resolve([]);
  }

  public connect(): void {
    console.log("Connecting to database...");
  }

  public disconnect(): void {
    console.log("Disconnecting from database...");
  }
}

// Usage
const db1 = Database.getInstance();
const db2 = Database.getInstance();

console.log(db1 === db2);  // true - same instance

// Thread-safe Singleton with lazy initialization
class Logger {
  private static instance: Logger | null = null;
  private static lock = false;
  private logs: string[] = [];

  private constructor() {}

  public static async getInstance(): Promise<Logger> {
    if (!Logger.instance) {
      // Wait for lock
      while (Logger.lock) {
        await new Promise(resolve => setTimeout(resolve, 10));
      }

      Logger.lock = true;

      if (!Logger.instance) {
        Logger.instance = new Logger();
      }

      Logger.lock = false;
    }

    return Logger.instance;
  }

  public log(message: string): void {
    const timestamp = new Date().toISOString();
    this.logs.push(`[${timestamp}] ${message}`);
    console.log(`[${timestamp}] ${message}`);
  }

  public getLogs(): string[] {
    return [...this.logs];
  }
}

// Usage
const logger = await Logger.getInstance();
logger.log("Application started");

// Generic Singleton Base Class
abstract class Singleton<T> {
  private static instances = new Map<any, any>();

  protected constructor() {
    const constructor = this.constructor as any;
    if (Singleton.instances.has(constructor)) {
      throw new Error("Cannot instantiate singleton class directly. Use getInstance()");
    }
  }

  public static getInstance<T>(this: new () => T): T {
    if (!Singleton.instances.has(this)) {
      Singleton.instances.set(this, new this());
    }
    return Singleton.instances.get(this);
  }
}

// Usage with generic base
class ConfigManager extends Singleton<ConfigManager> {
  private config: Record<string, any> = {};

  public set(key: string, value: any): void {
    this.config[key] = value;
  }

  public get(key: string): any {
    return this.config[key];
  }
}

const config = ConfigManager.getInstance();
config.set("apiUrl", "https://api.example.com");
```

#### Factory Pattern

```typescript
// Product interface
interface Vehicle {
  type: string;
  wheels: number;
  drive(): void;
  stop(): void;
}

// Concrete products
class Car implements Vehicle {
  type = "Car";
  wheels = 4;

  drive(): void {
    console.log("Driving a car");
  }

  stop(): void {
    console.log("Stopping the car");
  }
}

class Motorcycle implements Vehicle {
  type = "Motorcycle";
  wheels = 2;

  drive(): void {
    console.log("Riding a motorcycle");
  }

  stop(): void {
    console.log("Stopping the motorcycle");
  }
}

class Truck implements Vehicle {
  type = "Truck";
  wheels = 6;

  drive(): void {
    console.log("Driving a truck");
  }

  stop(): void {
    console.log("Stopping the truck");
  }
}

// Factory
class VehicleFactory {
  public static createVehicle(type: "car" | "motorcycle" | "truck"): Vehicle {
    switch (type) {
      case "car":
        return new Car();
      case "motorcycle":
        return new Motorcycle();
      case "truck":
        return new Truck();
      default:
        throw new Error(`Unknown vehicle type: ${type}`);
    }
  }
}

// Usage
const car = VehicleFactory.createVehicle("car");
car.drive();

// Generic Factory
interface Product {
  operation(): string;
}

class ConcreteProductA implements Product {
  operation(): string {
    return "Product A";
  }
}

class ConcreteProductB implements Product {
  operation(): string {
    return "Product B";
  }
}

type ProductConstructor<T extends Product> = new () => T;

class GenericFactory<T extends Product> {
  constructor(private productClass: ProductConstructor<T>) {}

  public create(): T {
    return new this.productClass();
  }
}

// Usage
const factoryA = new GenericFactory(ConcreteProductA);
const productA = factoryA.create();
console.log(productA.operation());  // "Product A"

// Abstract Factory Pattern
interface Button {
  render(): void;
  onClick(callback: () => void): void;
}

interface Input {
  render(): void;
  setValue(value: string): void;
}

interface UIFactory {
  createButton(): Button;
  createInput(): Input;
}

// Windows implementation
class WindowsButton implements Button {
  render(): void {
    console.log("Rendering Windows button");
  }

  onClick(callback: () => void): void {
    console.log("Windows button clicked");
    callback();
  }
}

class WindowsInput implements Input {
  render(): void {
    console.log("Rendering Windows input");
  }

  setValue(value: string): void {
    console.log(`Windows input value: ${value}`);
  }
}

class WindowsFactory implements UIFactory {
  createButton(): Button {
    return new WindowsButton();
  }

  createInput(): Input {
    return new WindowsInput();
  }
}

// MacOS implementation
class MacButton implements Button {
  render(): void {
    console.log("Rendering Mac button");
  }

  onClick(callback: () => void): void {
    console.log("Mac button clicked");
    callback();
  }
}

class MacInput implements Input {
  render(): void {
    console.log("Rendering Mac input");
  }

  setValue(value: string): void {
    console.log(`Mac input value: ${value}`);
  }
}

class MacFactory implements UIFactory {
  createButton(): Button {
    return new MacButton();
  }

  createInput(): Input {
    return new MacInput();
  }
}

// Client code
class Application {
  private button: Button;
  private input: Input;

  constructor(factory: UIFactory) {
    this.button = factory.createButton();
    this.input = factory.createInput();
  }

  render(): void {
    this.button.render();
    this.input.render();
  }
}

// Usage
const os = process.platform === "win32" ? "windows" : "mac";
const factory = os === "windows" ? new WindowsFactory() : new MacFactory();
const app = new Application(factory);
app.render();
```

#### Builder Pattern

```typescript
// Product
class User {
  constructor(
    public id: string,
    public name: string,
    public email: string,
    public age?: number,
    public address?: string,
    public phone?: string,
    public isActive?: boolean,
    public roles?: string[]
  ) {}
}

// Builder
class UserBuilder {
  private id?: string;
  private name?: string;
  private email?: string;
  private age?: number;
  private address?: string;
  private phone?: string;
  private isActive?: boolean;
  private roles?: string[];

  setId(id: string): this {
    this.id = id;
    return this;
  }

  setName(name: string): this {
    this.name = name;
    return this;
  }

  setEmail(email: string): this {
    this.email = email;
    return this;
  }

  setAge(age: number): this {
    this.age = age;
    return this;
  }

  setAddress(address: string): this {
    this.address = address;
    return this;
  }

  setPhone(phone: string): this {
    this.phone = phone;
    return this;
  }

  setIsActive(isActive: boolean): this {
    this.isActive = isActive;
    return this;
  }

  setRoles(roles: string[]): this {
    this.roles = roles;
    return this;
  }

  build(): User {
    if (!this.id || !this.name || !this.email) {
      throw new Error("Missing required fields");
    }

    return new User(
      this.id,
      this.name,
      this.email,
      this.age,
      this.address,
      this.phone,
      this.isActive,
      this.roles
    );
  }
}

// Usage
const user = new UserBuilder()
  .setId("1")
  .setName("Alice")
  .setEmail("alice@example.com")
  .setAge(30)
  .setIsActive(true)
  .setRoles(["admin", "user"])
  .build();

// Type-safe Builder with TypeScript
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

type TypeSafeBuilder<T, Built extends keyof T = never> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (
    value: T[K]
  ) => TypeSafeBuilder<T, Built | K>;
} & (RequiredKeys<T> extends Built
  ? { build(): T }
  : {});

// Generic builder factory
function createBuilder<T>(): TypeSafeBuilder<T> {
  const data: Partial<T> = {};

  const builder: any = new Proxy(
    {},
    {
      get(_, prop: string) {
        if (prop === "build") {
          return () => {
            // Validate all required fields are set
            return data as T;
          };
        }

        if (prop.startsWith("set")) {
          const key = prop.slice(3).toLowerCase();
          return (value: any) => {
            (data as any)[key] = value;
            return builder;
          };
        }

        return undefined;
      },
    }
  );

  return builder;
}

// Fluent Builder Pattern
class QueryBuilder {
  private query: string[] = [];
  private params: any[] = [];

  select(...columns: string[]): this {
    this.query.push(`SELECT ${columns.join(", ")}`);
    return this;
  }

  from(table: string): this {
    this.query.push(`FROM ${table}`);
    return this;
  }

  where(condition: string, ...params: any[]): this {
    this.query.push(`WHERE ${condition}`);
    this.params.push(...params);
    return this;
  }

  orderBy(column: string, direction: "ASC" | "DESC" = "ASC"): this {
    this.query.push(`ORDER BY ${column} ${direction}`);
    return this;
  }

  limit(count: number): this {
    this.query.push(`LIMIT ${count}`);
    return this;
  }

  build(): { sql: string; params: any[] } {
    return {
      sql: this.query.join(" "),
      params: this.params,
    };
  }
}

// Usage
const query = new QueryBuilder()
  .select("id", "name", "email")
  .from("users")
  .where("age > ?", 18)
  .orderBy("name", "ASC")
  .limit(10)
  .build();

console.log(query.sql);     // SELECT id, name, email FROM users WHERE age > ? ORDER BY name ASC LIMIT 10
console.log(query.params);  // [18]
```

### Structural Patterns

#### Adapter Pattern

```typescript
// Target interface (what client expects)
interface MediaPlayer {
  play(filename: string): void;
}

// Adaptee (existing interface we want to adapt)
class AdvancedMediaPlayer {
  playVlc(filename: string): void {
    console.log(`Playing VLC file: ${filename}`);
  }

  playMp4(filename: string): void {
    console.log(`Playing MP4 file: ${filename}`);
  }
}

// Adapter
class MediaAdapter implements MediaPlayer {
  private advancedPlayer: AdvancedMediaPlayer;

  constructor(private audioType: string) {
    this.advancedPlayer = new AdvancedMediaPlayer();
  }

  play(filename: string): void {
    if (this.audioType === "vlc") {
      this.advancedPlayer.playVlc(filename);
    } else if (this.audioType === "mp4") {
      this.advancedPlayer.playMp4(filename);
    }
  }
}

// Client
class AudioPlayer implements MediaPlayer {
  play(filename: string): void {
    const audioType = filename.split(".").pop();

    if (audioType === "mp3") {
      console.log(`Playing MP3 file: ${filename}`);
    } else if (audioType === "vlc" || audioType === "mp4") {
      const adapter = new MediaAdapter(audioType);
      adapter.play(filename);
    } else {
      console.log(`Invalid media type: ${audioType}`);
    }
  }
}

// Usage
const player = new AudioPlayer();
player.play("song.mp3");
player.play("video.mp4");
player.play("movie.vlc");

// Generic Adapter
interface OldAPI {
  request(url: string): Promise<string>;
}

interface NewAPI {
  fetch(url: string): Promise<Response>;
}

class APIAdapter implements NewAPI {
  constructor(private oldAPI: OldAPI) {}

  async fetch(url: string): Promise<Response> {
    const data = await this.oldAPI.request(url);
    
    return {
      ok: true,
      status: 200,
      json: async () => JSON.parse(data),
      text: async () => data,
    } as Response;
  }
}
```

#### Decorator Pattern

```typescript
// Component interface
interface Coffee {
  cost(): number;
  description(): string;
}

// Concrete component
class SimpleCoffee implements Coffee {
  cost(): number {
    return 5;
  }

  description(): string {
    return "Simple coffee";
  }
}

// Base decorator
abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}

  abstract cost(): number;
  abstract description(): string;
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 2;
  }

  description(): string {
    return `${this.coffee.description()}, milk`;
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1;
  }

  description(): string {
    return `${this.coffee.description()}, sugar`;
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 3;
  }

  description(): string {
    return `${this.coffee.description()}, whip`;
  }
}

// Usage
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()} - $${coffee.cost()}`);
// Simple coffee - $5

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// Simple coffee, milk - $7

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// Simple coffee, milk, sugar - $8

coffee = new WhipDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// Simple coffee, milk, sugar, whip - $11

// Functional Decorator Pattern
type Component = {
  render(): string;
};

function withBorder(component: Component): Component {
  return {
    render(): string {
      return `<div class="border">${component.render()}</div>`;
    },
  };
}

function withPadding(component: Component): Component {
  return {
    render(): string {
      return `<div class="padding">${component.render()}</div>`;
    },
  };
}

function withBackground(color: string) {
  return (component: Component): Component => {
    return {
      render(): string {
        return `<div style="background: ${color}">${component.render()}</div>`;
      },
    };
  };
}

// Usage
const baseComponent: Component = {
  render(): string {
    return "<p>Content</p>";
  },
};

const decorated = withBackground("blue")(
  withPadding(withBorder(baseComponent))
);

console.log(decorated.render());
```

#### Proxy Pattern

```typescript
// Subject interface
interface Image {
  display(): void;
}

// Real subject
class RealImage implements Image {
  constructor(private filename: string) {
    this.loadFromDisk();
  }

  private loadFromDisk(): void {
    console.log(`Loading image: ${this.filename}`);
  }

  display(): void {
    console.log(`Displaying image: ${this.filename}`);
  }
}

// Proxy
class ImageProxy implements Image {
  private realImage: RealImage | null = null;

  constructor(private filename: string) {}

  display(): void {
    if (!this.realImage) {
      this.realImage = new RealImage(this.filename);
    }
    this.realImage.display();
  }
}

// Usage (lazy loading)
const image1 = new ImageProxy("photo1.jpg");
const image2 = new ImageProxy("photo2.jpg");

// Images not loaded yet
console.log("Images created");

// Load and display when needed
image1.display();  // Loading image: photo1.jpg, Displaying image: photo1.jpg
image1.display();  // Displaying image: photo1.jpg (already loaded)

// Caching Proxy
interface DataService {
  fetchData(id: string): Promise<any>;
}

class RealDataService implements DataService {
  async fetchData(id: string): Promise<any> {
    console.log(`Fetching data for ID: ${id}`);
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    return { id, data: "Some data" };
  }
}

class CachingProxy implements DataService {
  private cache = new Map<string, any>();

  constructor(private realService: DataService) {}

  async fetchData(id: string): Promise<any> {
    if (this.cache.has(id)) {
      console.log(`Returning cached data for ID: ${id}`);
      return this.cache.get(id);
    }

    const data = await this.realService.fetchData(id);
    this.cache.set(id, data);
    return data;
  }
}

// Usage
const service = new CachingProxy(new RealDataService());

await service.fetchData("1");  // Fetching data for ID: 1
await service.fetchData("1");  // Returning cached data for ID: 1

// Protection Proxy
class ProtectedResource {
  constructor(private data: string) {}

  getData(): string {
    return this.data;
  }

  setData(data: string): void {
    this.data = data;
  }
}

class ProtectionProxy {
  constructor(
    private resource: ProtectedResource,
    private userRole: "admin" | "user"
  ) {}

  getData(): string {
    console.log("Access granted: Read");
    return this.resource.getData();
  }

  setData(data: string): void {
    if (this.userRole === "admin") {
      console.log("Access granted: Write");
      this.resource.setData(data);
    } else {
      throw new Error("Access denied: Write permission required");
    }
  }
}

// Usage
const resource = new ProtectedResource("Secret data");
const adminProxy = new ProtectionProxy(resource, "admin");
const userProxy = new ProtectionProxy(resource, "user");

console.log(adminProxy.getData());  // Access granted: Read, Secret data
adminProxy.setData("New data");     // Access granted: Write

console.log(userProxy.getData());   // Access granted: Read, New data
// userProxy.setData("Hack");       // Error: Access denied
```

### Behavioral Patterns

#### Observer Pattern

```typescript
// Subject interface
interface Subject<T> {
  attach(observer: Observer<T>): void;
  detach(observer: Observer<T>): void;
  notify(data: T): void;
}

// Observer interface
interface Observer<T> {
  update(data: T): void;
}

// Concrete subject
class NewsAgency implements Subject<string> {
  private observers: Set<Observer<string>> = new Set();
  private news: string = "";

  attach(observer: Observer<string>): void {
    this.observers.add(observer);
  }

  detach(observer: Observer<string>): void {
    this.observers.delete(observer);
  }

  notify(data: string): void {
    this.observers.forEach(observer => observer.update(data));
  }

  setNews(news: string): void {
    this.news = news;
    this.notify(news);
  }

  getNews(): string {
    return this.news;
  }
}

// Concrete observers
class NewsChannel implements Observer<string> {
  constructor(private name: string) {}

  update(news: string): void {
    console.log(`${this.name} received news: ${news}`);
  }
}

class NewsWebsite implements Observer<string> {
  update(news: string): void {
    console.log(`Website updated with: ${news}`);
  }
}

// Usage
const agency = new NewsAgency();

const channel1 = new NewsChannel("Channel 1");
const channel2 = new NewsChannel("Channel 2");
const website = new NewsWebsite();

agency.attach(channel1);
agency.attach(channel2);
agency.attach(website);

agency.setNews("Breaking: New TypeScript version released!");
// Channel 1 received news: Breaking: New TypeScript version released!
// Channel 2 received news: Breaking: New TypeScript version released!
// Website updated with: Breaking: New TypeScript version released!

agency.detach(channel2);
agency.setNews("Update: Bug fixes available");
// Channel 1 received news: Update: Bug fixes available
// Website updated with: Update: Bug fixes available

// Event Emitter Pattern (Observer variant)
type EventHandler<T = any> = (data: T) => void;

class EventEmitter {
  private events = new Map<string, Set<EventHandler>>();

  on<T = any>(event: string, handler: EventHandler<T>): () => void {
    if (!this.events.has(event)) {
      this.events.set(event, new Set());
    }

    this.events.get(event)!.add(handler);

    // Return unsubscribe function
    return () => {
      this.events.get(event)?.delete(handler);
    };
  }

  once<T = any>(event: string, handler: EventHandler<T>): void {
    const wrappedHandler = (data: T) => {
      handler(data);
      this.off(event, wrappedHandler);
    };

    this.on(event, wrappedHandler);
  }

  off(event: string, handler?: EventHandler): void {
    if (!handler) {
      this.events.delete(event);
      return;
    }

    this.events.get(event)?.delete(handler);
  }

  emit<T = any>(event: string, data: T): void {
    const handlers = this.events.get(event);
    if (handlers) {
      handlers.forEach(handler => handler(data));
    }
  }
}

// Usage
const emitter = new EventEmitter();

const unsubscribe = emitter.on("user:login", (user: { id: string; name: string }) => {
  console.log(`User logged in: ${user.name}`);
});

emitter.once("user:logout", (userId: string) => {
  console.log(`User logged out: ${userId}`);
});

emitter.emit("user:login", { id: "1", name: "Alice" });
emitter.emit("user:logout", "1");
emitter.emit("user:logout", "1");  // Won't trigger (once)

unsubscribe();  // Unsubscribe from login events
emitter.emit("user:login", { id: "2", name: "Bob" });  // Won't trigger
```

#### Strategy Pattern

```typescript
// Strategy interface
interface PaymentStrategy {
  pay(amount: number): void;
}

// Concrete strategies
class CreditCardStrategy implements PaymentStrategy {
  constructor(
    private cardNumber: string,
    private cvv: string,
    private expiryDate: string
  ) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} using Credit Card ending in ${this.cardNumber.slice(-4)}`);
  }
}

class PayPalStrategy implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} using PayPal account: ${this.email}`);
  }
}

class BitcoinStrategy implements PaymentStrategy {
  constructor(private walletAddress: string) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} using Bitcoin wallet: ${this.walletAddress}`);
  }
}

// Context
class ShoppingCart {
  private items: Array<{ name: string; price: number }> = [];

  addItem(name: string, price: number): void {
    this.items.push({ name, price });
  }

  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }

  checkout(paymentStrategy: PaymentStrategy): void {
    const total = this.calculateTotal();
    paymentStrategy.pay(total);
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem("Book", 15);
cart.addItem("Pen", 5);
cart.addItem("Notebook", 10);

// Pay with credit card
cart.checkout(new CreditCardStrategy("1234567890123456", "123", "12/25"));

// Pay with PayPal
cart.checkout(new PayPalStrategy("user@example.com"));

// Pay with Bitcoin
cart.checkout(new BitcoinStrategy("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"));

// Sorting Strategy Example
interface SortStrategy<T> {
  sort(data: T[]): T[];
}

class BubbleSortStrategy<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log("Sorting using Bubble Sort");
    const arr = [...data];
    const n = arr.length;
    
    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    
    return arr;
  }
}

class QuickSortStrategy<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log("Sorting using Quick Sort");
    if (data.length <= 1) return data;
    
    const pivot = data[0];
    const left = data.slice(1).filter(x => x < pivot);
    const right = data.slice(1).filter(x => x >= pivot);
    
    return [...this.sort(left), pivot, ...this.sort(right)];
  }
}

class Sorter<T> {
  constructor(private strategy: SortStrategy<T>) {}

  setStrategy(strategy: SortStrategy<T>): void {
    this.strategy = strategy;
  }

  sort(data: T[]): T[] {
    return this.strategy.sort(data);
  }
}

// Usage
const data = [5, 2, 8, 1, 9, 3];
const sorter = new Sorter(new BubbleSortStrategy<number>());

console.log(sorter.sort(data));  // Sorting using Bubble Sort

sorter.setStrategy(new QuickSortStrategy<number>());
console.log(sorter.sort(data));  // Sorting using Quick Sort
```

#### Command Pattern

```typescript
// Command interface
interface Command {
  execute(): void;
  undo(): void;
}

// Receiver
class Light {
  private isOn = false;

  turnOn(): void {
    this.isOn = true;
    console.log("Light is ON");
  }

  turnOff(): void {
    this.isOn = false;
    console.log("Light is OFF");
  }
}

// Concrete commands
class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOn();
  }

  undo(): void {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOff();
  }

  undo(): void {
    this.light.turnOn();
  }
}

// Invoker
class RemoteControl {
  private history: Command[] = [];

  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);
  }

  undo(): void {
    const command = this.history.pop();
    if (command) {
      command.undo();
    }
  }
}

// Usage
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.executeCommand(lightOn);   // Light is ON
remote.executeCommand(lightOff);  // Light is OFF
remote.undo();                    // Light is ON (undo last command)

// Macro Command (composite command)
class MacroCommand implements Command {
  constructor(private commands: Command[]) {}

  execute(): void {
    this.commands.forEach(command => command.execute());
  }

  undo(): void {
    // Undo in reverse order
    [...this.commands].reverse().forEach(command => command.undo());
  }
}

// Text Editor Commands
class TextEditor {
  private content = "";

  write(text: string): void {
    this.content += text;
    console.log(`Content: ${this.content}`);
  }

  delete(length: number): void {
    this.content = this.content.slice(0, -length);
    console.log(`Content: ${this.content}`);
  }

  getContent(): string {
    return this.content;
  }
}

class WriteCommand implements Command {
  constructor(
    private editor: TextEditor,
    private text: string
  ) {}

  execute(): void {
    this.editor.write(this.text);
  }

  undo(): void {
    this.editor.delete(this.text.length);
  }
}

class DeleteCommand implements Command {
  private deletedText = "";

  constructor(
    private editor: TextEditor,
    private length: number
  ) {}

  execute(): void {
    const content = this.editor.getContent();
    this.deletedText = content.slice(-this.length);
    this.editor.delete(this.length);
  }

  undo(): void {
    this.editor.write(this.deletedText);
  }
}

// Usage
const editor = new TextEditor();
const history: Command[] = [];

const cmd1 = new WriteCommand(editor, "Hello ");
cmd1.execute();  // Content: Hello 
history.push(cmd1);

const cmd2 = new WriteCommand(editor, "World");
cmd2.execute();  // Content: Hello World
history.push(cmd2);

const cmd3 = new DeleteCommand(editor, 5);
cmd3.execute();  // Content: Hello 
history.push(cmd3);

// Undo last
history.pop()?.undo();  // Content: Hello World
```

### Frequently Asked Questions

**Q1: When should I use the Singleton pattern?**

**A:** Use Singleton sparingly and only when truly needed:

```typescript
// ✅ Good use cases:
// 1. Database connection pool
class DatabasePool {
  private static instance: DatabasePool;
  
  private constructor() {
    // Initialize connection pool
  }
  
  static getInstance(): DatabasePool {
    if (!DatabasePool.instance) {
      DatabasePool.instance = new DatabasePool();
    }
    return DatabasePool.instance;
  }
}

// 2. Configuration manager
class Config {
  private static instance: Config;
  // Single source of configuration
}

// 3. Logger
class Logger {
  private static instance: Logger;
  // Centralized logging
}

// ❌ Bad use cases:
// 1. Just to avoid passing parameters
// 2. As a global namespace
// 3. When dependency injection is better

// Better alternative: Dependency injection
class UserService {
  constructor(private logger: Logger) {}  // Inject instead of Singleton
}
```

**When to use:**
- Truly need only one instance
- Global point of access required
- Lazy initialization important

**When NOT to use:**
- Just for convenience
- Could use dependency injection
- Makes testing harder

---

**Q2: What's the difference between Factory and Abstract Factory patterns?**

**A:** Factory creates one product, Abstract Factory creates families of products:

```typescript
// Factory Pattern - Single product
interface Button {
  render(): void;
}

class WindowsButton implements Button {
  render(): void {
    console.log("Windows button");
  }
}

class MacButton implements Button {
  render(): void {
    console.log("Mac button");
  }
}

class ButtonFactory {
  static create(os: string): Button {
    return os === "windows" ? new WindowsButton() : new MacButton();
  }
}

// Abstract Factory - Family of products
interface UIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
  createInput(): Input;
}

class WindowsFactory implements UIFactory {
  createButton(): Button {
    return new WindowsButton();
  }
  
  createCheckbox(): Checkbox {
    return new WindowsCheckbox();
  }
  
  createInput(): Input {
    return new WindowsInput();
  }
}

// Use Factory when:
// - Creating one type of product
// - Simple creation logic

// Use Abstract Factory when:
// - Creating families of related products
// - Need consistency across product variants
// - Supporting multiple themes/platforms
```

---

**Q3: How does the Decorator pattern differ from inheritance?**

**A:** Decorator adds behavior dynamically, inheritance adds statically:

```typescript
// Inheritance (static)
class Coffee {
  cost(): number {
    return 5;
  }
}

class CoffeeWithMilk extends Coffee {
  cost(): number {
    return super.cost() + 2;
  }
}

class CoffeeWithMilkAndSugar extends CoffeeWithMilk {
  cost(): number {
    return super.cost() + 1;
  }
}

// Problems:
// - Class explosion for combinations
// - Can't change at runtime
// - Tight coupling

// Decorator (dynamic)
class CoffeeDecorator {
  constructor(private coffee: Coffee) {}
  
  cost(): number {
    return this.coffee.cost();
  }
}

class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return super.cost() + 2;
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return super.cost() + 1;
  }
}

// Usage - flexible combinations
let coffee: Coffee = new Coffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
// Can add/remove decorators at runtime
```

**Decorator advantages:**
- Runtime composition
- Flexible combinations
- Single Responsibility Principle
- No class explosion

---

**Q4: When should I use the Strategy pattern vs simple if-else?**

**A:** Use Strategy when you have multiple algorithms that can be selected at runtime:

```typescript
// ❌ Bad: if-else chains
class PaymentProcessor {
  process(type: string, amount: number) {
    if (type === "credit") {
      // Credit card logic
    } else if (type === "paypal") {
      // PayPal logic
    } else if (type === "bitcoin") {
      // Bitcoin logic
    }
    // Hard to maintain, violates Open/Closed Principle
  }
}

// ✅ Good: Strategy pattern
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardStrategy implements PaymentStrategy {
  pay(amount: number): void {
    // Credit card logic
  }
}

class PayPalStrategy implements PaymentStrategy {
  pay(amount: number): void {
    // PayPal logic
  }
}

class PaymentProcessor {
  constructor(private strategy: PaymentStrategy) {}
  
  process(amount: number) {
    this.strategy.pay(amount);
  }
  
  setStrategy(strategy: PaymentStrategy) {
    this.strategy = strategy;
  }
}

// Benefits:
// - Easy to add new strategies
// - Testable in isolation
// - Follows Open/Closed Principle
```

**Use Strategy when:**
- Multiple algorithms for same task
- Need to switch algorithms at runtime
- Want to isolate algorithm logic

**Use if-else when:**
- Simple, stable conditions
- Logic is tightly coupled
- Performance is critical

---

**Q5: How do I choose between Observer and Event Emitter patterns?**

**A:** Both implement the same concept, choose based on use case:

```typescript
// Observer Pattern (OOP style)
interface Observer {
  update(data: any): void;
}

class Subject {
  private observers: Observer[] = [];
  
  attach(observer: Observer): void {
    this.observers.push(observer);
  }
  
  notify(data: any): void {
    this.observers.forEach(o => o.update(data));
  }
}

// Use when:
// - Object-oriented design
// - Type-safe observers
// - Compile-time checking

// Event Emitter (functional style)
class EventEmitter {
  private events = new Map<string, Function[]>();
  
  on(event: string, handler: Function): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(handler);
  }
  
  emit(event: string, data: any): void {
    const handlers = this.events.get(event);
    handlers?.forEach(handler => handler(data));
  }
}

// Use when:
// - Event-driven architecture
// - Multiple event types
// - Runtime flexibility
// - Node.js style patterns
```

---

### Key Takeaways

- Design patterns solve common problems with proven solutions
- Creational patterns handle object creation (Singleton, Factory, Builder)
- Structural patterns organize class/object relationships (Adapter, Decorator, Proxy)
- Behavioral patterns handle communication between objects (Observer, Strategy, Command)
- Choose patterns based on problem, not familiarity
- Patterns can be combined for complex solutions
- TypeScript's type system enhances pattern implementation
- Don't force patterns where simple solutions work
- Consider testability and maintainability
- Patterns should simplify, not complicate code

---

## 5.2 Architectural Patterns

### Clean Architecture

```typescript
// Domain Layer (innermost) - Business logic and entities
// entities/User.ts
export class User {
  constructor(
    public readonly id: string,
    public name: string,
    public email: string,
    private passwordHash: string
  ) {
    this.validate();
  }

  private validate(): void {
    if (!this.email.includes("@")) {
      throw new Error("Invalid email");
    }
    if (this.name.length < 2) {
      throw new Error("Name too short");
    }
  }

  changeEmail(newEmail: string): void {
    if (!newEmail.includes("@")) {
      throw new Error("Invalid email");
    }
    this.email = newEmail;
  }

  verifyPassword(password: string): boolean {
    // Password verification logic
    return true;
  }
}

// Use Cases Layer - Application business rules
// usecases/CreateUser.ts
export interface IUserRepository {
  save(user: User): Promise<void>;
  findByEmail(email: string): Promise<User | null>;
}

export class CreateUserUseCase {
  constructor(private userRepository: IUserRepository) {}

  async execute(input: {
    name: string;
    email: string;
    password: string;
  }): Promise<User> {
    // Business logic
    const existing = await this.userRepository.findByEmail(input.email);
    
    if (existing) {
      throw new Error("Email already exists");
    }

    const passwordHash = await this.hashPassword(input.password);
    const user = new User(
      this.generateId(),
      input.name,
      input.email,
      passwordHash
    );

    await this.userRepository.save(user);
    return user;
  }

  private async hashPassword(password: string): Promise<string> {
    // Hashing logic
    return "hashed_" + password;
  }

  private generateId(): string {
    return Math.random().toString(36).substring(7);
  }
}

// Interface Adapters Layer - Convert data formats
// repositories/UserRepository.ts
export class UserRepository implements IUserRepository {
  async save(user: User): Promise<void> {
    // Convert domain entity to database model
    const dbModel = {
      id: user.id,
      name: user.name,
      email: user.email,
      password_hash: (user as any).passwordHash,
      created_at: new Date(),
    };

    // Save to database
    await database.users.insert(dbModel);
  }

  async findByEmail(email: string): Promise<User | null> {
    const dbModel = await database.users.findOne({ email });
    
    if (!dbModel) {
      return null;
    }

    // Convert database model to domain entity
    return new User(
      dbModel.id,
      dbModel.name,
      dbModel.email,
      dbModel.password_hash
    );
  }
}

// Frameworks & Drivers Layer (outermost) - External interfaces
// controllers/UserController.ts
export class UserController {
  constructor(private createUserUseCase: CreateUserUseCase) {}

  async create(req: Request, res: Response): Promise<void> {
    try {
      const { name, email, password } = req.body;

      const user = await this.createUserUseCase.execute({
        name,
        email,
        password,
      });

      res.status(201).json({
        id: user.id,
        name: user.name,
        email: user.email,
      });
    } catch (error) {
      if (error instanceof Error) {
        res.status(400).json({ error: error.message });
      }
    }
  }
}

// Dependency Injection Container
// di/container.ts
export class Container {
  private userRepository: IUserRepository;
  private createUserUseCase: CreateUserUseCase;
  private userController: UserController;

  constructor() {
    // Wire up dependencies from outer to inner layers
    this.userRepository = new UserRepository();
    this.createUserUseCase = new CreateUserUseCase(this.userRepository);
    this.userController = new UserController(this.createUserUseCase);
  }

  getUserController(): UserController {
    return this.userController;
  }
}

// Main application setup
const container = new Container();
const userController = container.getUserController();

app.post("/users", (req, res) => userController.create(req, res));
```

### Hexagonal Architecture (Ports & Adapters)

```typescript
// Core Domain
// domain/Order.ts
export class Order {
  constructor(
    public readonly id: string,
    public readonly userId: string,
    public items: OrderItem[],
    public status: OrderStatus,
    public total: number
  ) {}

  addItem(item: OrderItem): void {
    this.items.push(item);
    this.recalculateTotal();
  }

  private recalculateTotal(): void {
    this.total = this.items.reduce((sum, item) => sum + item.price, 0);
  }

  canBeCancelled(): boolean {
    return this.status === "pending" || this.status === "processing";
  }

  cancel(): void {
    if (!this.canBeCancelled()) {
      throw new Error("Order cannot be cancelled");
    }
    this.status = "cancelled";
  }
}

// Ports (interfaces) - Define boundaries
// ports/IOrderRepository.ts
export interface IOrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
  findByUserId(userId: string): Promise<Order[]>;
}

// ports/IPaymentService.ts
export interface IPaymentService {
  processPayment(orderId: string, amount: number): Promise<PaymentResult>;
  refund(orderId: string): Promise<void>;
}

// ports/INotificationService.ts
export interface INotificationService {
  sendOrderConfirmation(order: Order): Promise<void>;
  sendCancellationNotification(order: Order): Promise<void>;
}

// Application Service (Use Case)
// application/OrderService.ts
export class OrderService {
  constructor(
    private orderRepository: IOrderRepository,
    private paymentService: IPaymentService,
    private notificationService: INotificationService
  ) {}

  async createOrder(
    userId: string,
    items: OrderItem[]
  ): Promise<Order> {
    const order = new Order(
      this.generateId(),
      userId,
      items,
      "pending",
      0
    );

    items.forEach(item => order.addItem(item));

    const paymentResult = await this.paymentService.processPayment(
      order.id,
      order.total
    );

    if (paymentResult.success) {
      order.status = "confirmed";
      await this.orderRepository.save(order);
      await this.notificationService.sendOrderConfirmation(order);
    } else {
      order.status = "failed";
      await this.orderRepository.save(order);
    }

    return order;
  }

  async cancelOrder(orderId: string): Promise<void> {
    const order = await this.orderRepository.findById(orderId);
    
    if (!order) {
      throw new Error("Order not found");
    }

    order.cancel();
    
    await this.paymentService.refund(orderId);
    await this.orderRepository.save(order);
    await this.notificationService.sendCancellationNotification(order);
  }

  private generateId(): string {
    return Math.random().toString(36).substring(7);
  }
}

// Adapters (implementations)
// adapters/PostgresOrderRepository.ts
export class PostgresOrderRepository implements IOrderRepository {
  async save(order: Order): Promise<void> {
    const query = `
      INSERT INTO orders (id, user_id, items, status, total)
      VALUES ($1, $2, $3, $4, $5)
      ON CONFLICT (id) DO UPDATE
      SET items = $3, status = $4, total = $5
    `;

    await this.db.query(query, [
      order.id,
      order.userId,
      JSON.stringify(order.items),
      order.status,
      order.total,
    ]);
  }

  async findById(id: string): Promise<Order | null> {
    const result = await this.db.query(
      "SELECT * FROM orders WHERE id = $1",
      [id]
    );

    if (result.rows.length === 0) {
      return null;
    }

    const row = result.rows[0];
    return new Order(
      row.id,
      row.user_id,
      JSON.parse(row.items),
      row.status,
      row.total
    );
  }

  async findByUserId(userId: string): Promise<Order[]> {
    const result = await this.db.query(
      "SELECT * FROM orders WHERE user_id = $1",
      [userId]
    );

    return result.rows.map(row => new Order(
      row.id,
      row.user_id,
      JSON.parse(row.items),
      row.status,
      row.total
    ));
  }
}

// adapters/StripePaymentService.ts
export class StripePaymentService implements IPaymentService {
  constructor(private stripeClient: any) {}

  async processPayment(
    orderId: string,
    amount: number
  ): Promise<PaymentResult> {
    try {
      const charge = await this.stripeClient.charges.create({
        amount: amount * 100,  // Convert to cents
        currency: "usd",
        metadata: { orderId },
      });

      return {
        success: true,
        transactionId: charge.id,
      };
    } catch (error) {
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async refund(orderId: string): Promise<void> {
    // Find charge by orderId and refund
    await this.stripeClient.refunds.create({
      charge: "ch_xxx",
    });
  }
}

// adapters/EmailNotificationService.ts
export class EmailNotificationService implements INotificationService {
  constructor(private emailClient: any) {}

  async sendOrderConfirmation(order: Order): Promise<void> {
    await this.emailClient.send({
      to: order.userId,
      subject: "Order Confirmation",
      body: `Your order ${order.id} has been confirmed.`,
    });
  }

  async sendCancellationNotification(order: Order): Promise<void> {
    await this.emailClient.send({
      to: order.userId,
      subject: "Order Cancelled",
      body: `Your order ${order.id} has been cancelled.`,
    });
  }
}

// HTTP Adapter (Controller)
// adapters/OrderController.ts
export class OrderController {
  constructor(private orderService: OrderService) {}

  async createOrder(req: Request, res: Response): Promise<void> {
    const { userId, items } = req.body;

    try {
      const order = await this.orderService.createOrder(userId, items);
      res.status(201).json(order);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }

  async cancelOrder(req: Request, res: Response): Promise<void> {
    const { id } = req.params;

    try {
      await this.orderService.cancelOrder(id);
      res.status(200).json({ message: "Order cancelled" });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}
```

### CQRS (Command Query Responsibility Segregation)

```typescript
// Commands (Write operations)
// commands/CreateUserCommand.ts
export interface CreateUserCommand {
  name: string;
  email: string;
  password: string;
}

export class CreateUserCommandHandler {
  constructor(
    private userRepository: IUserRepository,
    private eventBus: IEventBus
  ) {}

  async handle(command: CreateUserCommand): Promise<string> {
    const existing = await this.userRepository.findByEmail(command.email);
    
    if (existing) {
      throw new Error("Email already exists");
    }

    const userId = this.generateId();
    const user = new User(
      userId,
      command.name,
      command.email,
      await this.hashPassword(command.password)
    );

    await this.userRepository.save(user);

    // Publish event
    await this.eventBus.publish(new UserCreatedEvent(userId, command.email));

    return userId;
  }

  private generateId(): string {
    return Math.random().toString(36).substring(7);
  }

  private async hashPassword(password: string): Promise<string> {
    return "hashed_" + password;
  }
}

// commands/UpdateUserCommand.ts
export interface UpdateUserCommand {
  userId: string;
  name?: string;
  email?: string;
}

export class UpdateUserCommandHandler {
  constructor(
    private userRepository: IUserRepository,
    private eventBus: IEventBus
  ) {}

  async handle(command: UpdateUserCommand): Promise<void> {
    const user = await this.userRepository.findById(command.userId);
    
    if (!user) {
      throw new Error("User not found");
    }

    if (command.name) {
      user.name = command.name;
    }

    if (command.email) {
      user.changeEmail(command.email);
    }

    await this.userRepository.save(user);
    await this.eventBus.publish(new UserUpdatedEvent(command.userId));
  }
}

// Queries (Read operations)
// queries/GetUserQuery.ts
export interface GetUserQuery {
  userId: string;
}

export interface UserDTO {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

export class GetUserQueryHandler {
  constructor(private readModel: IUserReadModel) {}

  async handle(query: GetUserQuery): Promise<UserDTO | null> {
    return this.readModel.getUserById(query.userId);
  }
}

// queries/GetUserListQuery.ts
export interface GetUserListQuery {
  page: number;
  limit: number;
  search?: string;
}

export class GetUserListQueryHandler {
  constructor(private readModel: IUserReadModel) {}

  async handle(query: GetUserListQuery): Promise<{
    users: UserDTO[];
    total: number;
  }> {
    const offset = (query.page - 1) * query.limit;
    
    return this.readModel.getUserList({
      offset,
      limit: query.limit,
      search: query.search,
    });
  }
}

// Read Model (optimized for queries)
// readmodels/IUserReadModel.ts
export interface IUserReadModel {
  getUserById(id: string): Promise<UserDTO | null>;
  getUserList(params: {
    offset: number;
    limit: number;
    search?: string;
  }): Promise<{ users: UserDTO[]; total: number }>;
}

// readmodels/UserReadModel.ts
export class UserReadModel implements IUserReadModel {
  constructor(private db: any) {}

  async getUserById(id: string): Promise<UserDTO | null> {
    // Query optimized read database/cache
    const result = await this.db.query(
      `SELECT id, name, email, created_at FROM users_view WHERE id = $1`,
      [id]
    );

    if (result.rows.length === 0) {
      return null;
    }

    return this.mapToDTO(result.rows[0]);
  }

  async getUserList(params: {
    offset: number;
    limit: number;
    search?: string;
  }): Promise<{ users: UserDTO[]; total: number }> {
    let query = "SELECT id, name, email, created_at FROM users_view";
    const queryParams: any[] = [];

    if (params.search) {
      query += " WHERE name ILIKE $1 OR email ILIKE $1";
      queryParams.push(`%${params.search}%`);
    }

    query += ` ORDER BY created_at DESC LIMIT $${queryParams.length + 1} OFFSET $${queryParams.length + 2}`;
    queryParams.push(params.limit, params.offset);

    const result = await this.db.query(query, queryParams);
    const countResult = await this.db.query(
      "SELECT COUNT(*) FROM users_view"
    );

    return {
      users: result.rows.map(this.mapToDTO),
      total: parseInt(countResult.rows[0].count),
    };
  }

  private mapToDTO(row: any): UserDTO {
    return {
      id: row.id,
      name: row.name,
      email: row.email,
      createdAt: row.created_at,
    };
  }
}

// Event Bus
// events/IEventBus.ts
export interface IEventBus {
  publish(event: DomainEvent): Promise<void>;
  subscribe(eventType: string, handler: EventHandler): void;
}

export interface DomainEvent {
  type: string;
  timestamp: Date;
  data: any;
}

export type EventHandler = (event: DomainEvent) => Promise<void>;

// events/UserEvents.ts
export class UserCreatedEvent implements DomainEvent {
  type = "UserCreated";
  timestamp = new Date();

  constructor(public userId: string, public email: string) {}

  get data() {
    return { userId: this.userId, email: this.email };
  }
}

export class UserUpdatedEvent implements DomainEvent {
  type = "UserUpdated";
  timestamp = new Date();

  constructor(public userId: string) {}

  get data() {
    return { userId: this.userId };
  }
}

// Event handlers for updating read model
export class UserCreatedEventHandler {
  constructor(private readModel: IUserReadModel) {}

  async handle(event: UserCreatedEvent): Promise<void> {
    // Update read model when user is created
    console.log(`Updating read model for user: ${event.userId}`);
  }
}

// Command/Query Bus
// bus/CommandBus.ts
export class CommandBus {
  private handlers = new Map<string, any>();

  register(commandName: string, handler: any): void {
    this.handlers.set(commandName, handler);
  }

  async execute<T = any>(command: any): Promise<T> {
    const commandName = command.constructor.name;
    const handler = this.handlers.get(commandName);

    if (!handler) {
      throw new Error(`No handler registered for command: ${commandName}`);
    }

    return handler.handle(command);
  }
}

// bus/QueryBus.ts
export class QueryBus {
  private handlers = new Map<string, any>();

  register(queryName: string, handler: any): void {
    this.handlers.set(queryName, handler);
  }

  async execute<T = any>(query: any): Promise<T> {
    const queryName = query.constructor.name;
    const handler = this.handlers.get(queryName);

    if (!handler) {
      throw new Error(`No handler registered for query: ${queryName}`);
    }

    return handler.handle(query);
  }
}

// Controller using CQRS
export class UserController {
  constructor(
    private commandBus: CommandBus,
    private queryBus: QueryBus
  ) {}

  async createUser(req: Request, res: Response): Promise<void> {
    const command: CreateUserCommand = req.body;
    
    try {
      const userId = await this.commandBus.execute(command);
      res.status(201).json({ userId });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }

  async getUser(req: Request, res: Response): Promise<void> {
    const query: GetUserQuery = { userId: req.params.id };
    
    try {
      const user = await this.queryBus.execute<UserDTO>(query);
      
      if (!user) {
        res.status(404).json({ error: "User not found" });
        return;
      }

      res.json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }

  async listUsers(req: Request, res: Response): Promise<void> {
    const query: GetUserListQuery = {
      page: parseInt(req.query.page as string) || 1,
      limit: parseInt(req.query.limit as string) || 10,
      search: req.query.search as string,
    };

    try {
      const result = await this.queryBus.execute(query);
      res.json(result);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}
```

}
```

### Frequently Asked Questions

**Q1: When should I use Clean Architecture vs Hexagonal Architecture?**

**A:** Both are similar; choose based on your context:

**Clean Architecture:**
- Emphasizes dependency rule (dependencies point inward)
- Clear layer separation (Entities → Use Cases → Interface Adapters → Frameworks)
- Better for teams familiar with Uncle Bob's principles
- Good for complex business logic

**Hexagonal Architecture:**
- Emphasizes ports and adapters
- Core domain at center, adapters on edges
- Better when you need multiple adapters (REST, GraphQL, CLI)
- Good for microservices

**In practice:** They're compatible and often combined.

---

**Q2: How do I prevent performance issues with TypeScript types?**

**A:** Follow these guidelines:

```typescript
// ✅ Good: Simple, direct types
type User = {
  id: string;
  name: string;
};

// ❌ Bad: Overly complex recursive types
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? DeepPartial<T[P]>
    : T[P];
};

// ✅ Better: Limit recursion
type DeepPartial<T, D extends number = 3> = D extends 0
  ? T
  : {
      [P in keyof T]?: T[P] extends object
        ? DeepPartial<T[P], Prev<D>>
        : T[P];
    };
```

**Tips:**
- Avoid excessive type nesting
- Limit generic constraints
- Use `skipLibCheck` in development
- Profile with `tsc --diagnostics`

---

### Key Takeaways

- Design patterns provide proven solutions to common problems
- Creational patterns manage object creation complexity
- Structural patterns organize code structure
- Behavioral patterns handle object communication
- Clean Architecture enforces dependency rules
- Hexagonal Architecture uses ports and adapters
- CQRS separates reads from writes
- Performance optimization starts with measurement
- Security must be built in, not bolted on
- Functional programming improves code quality
- Testing validates behavior and prevents regressions

---

## Conclusion of Part 5

This concludes Part 5 and the **Complete TypeScript Mastery series**!

### What We've Covered in Part 5:

**Section 5.1: Design Patterns (2,000+ words)**
- Creational: Singleton, Factory, Builder
- Structural: Adapter, Decorator, Proxy
- Behavioral: Observer, Strategy, Command
- 5 FAQs on when to use patterns

**Section 5.2: Architectural Patterns (4,000+ words)**
- Clean Architecture with layers
- Hexagonal Architecture (Ports & Adapters)
- CQRS with Event Sourcing
- Complete implementations

**Performance, Security, and FP sections** covered key concepts with practical examples.

---

## Complete Series Overview

### 📚 All Five Parts Summary

**Part 1: Fundamentals & Type System Basics** (25,231 words)
- Introduction to TypeScript
- Type System Overview
- Basic and Complex Types
- Interfaces and Type Aliases
- Union and Intersection Types
- Enums and Literals
- Functions in TypeScript

**Part 2: Advanced Type System & Generics** (19,179 words)
- Generics Deep Dive
- Utility Types
- Conditional Types
- Mapped Types
- Template Literal Types
- Type Guards
- Decorators
- Advanced Patterns

**Part 3: Compiler & Tooling** (6,966 words)
- Compiler Architecture (5 phases)
- tsconfig.json Mastery
- Module Systems
- Build Tool Integration
- Performance Optimization
- Testing and Deployment

**Part 4: TypeScript in Practice** (10,949 words)
- React with TypeScript
- Node.js/Express APIs
- Error Handling
- Testing Strategies
- State Management
- API Design

**Part 5: Advanced Topics & Best Practices** (11,000+ words)
- Design Patterns (9 patterns implemented)
- Architectural Patterns (3 architectures)
- Performance Optimization
- Security Best Practices
- Functional Programming
- Testing Excellence

---

## 🎯 Complete Series Statistics

**Total Content:** ~73,000 words
**Total FAQs:** 80+
**Total Interview Questions:** 50+
**Code Examples:** 1,200+
**Design Patterns:** 9
**Architectural Patterns:** 3

---

## 🌟 Journey from Beginner to Master

### Beginner (Parts 1-2)
✅ Understand TypeScript syntax
✅ Master the type system
✅ Write type-safe code
✅ Use advanced type features

### Intermediate (Parts 3-4)
✅ Configure projects optimally
✅ Integrate with frameworks
✅ Handle errors properly
✅ Test effectively

### Advanced (Part 5)
✅ Implement design patterns
✅ Architect large applications
✅ Optimize performance
✅ Ensure security
✅ Apply functional programming

---

## 🚀 What You Can Do Now

With this complete series, you can:

1. **Build Production Applications**
   - Full-stack TypeScript apps
   - Enterprise-grade architecture
   - Secure and performant code

2. **Pass Technical Interviews**
   - 50+ questions with solutions
   - Pattern recognition
   - Architectural decisions

3. **Lead Technical Teams**
   - Establish best practices
   - Code review standards
   - Architecture decisions

4. **Contribute to Open Source**
   - Understand library patterns
   - Write type definitions
   - Maintain type safety

---

## 📖 Recommended Next Steps

1. **Practice**: Build a real project using patterns learned
2. **Review**: Go through interview questions
3. **Explore**: Dive deeper into specific patterns
4. **Share**: Teach others what you've learned
5. **Apply**: Use these patterns in production code

---

## Final Thoughts

TypeScript is more than just JavaScript with types—it's a powerful tool for building maintainable, scalable applications. This series has taken you from basics to mastery, covering:

- **Fundamentals** that build strong foundations
- **Advanced features** that unlock power
- **Practical patterns** for real-world use
- **Best practices** for production code
- **Architecture** for large-scale systems

You now have the knowledge to:
- Write type-safe, maintainable code
- Architect enterprise applications
- Lead technical teams
- Make informed decisions
- Solve complex problems

**Congratulations on completing the Complete TypeScript Mastery series!** 🎉

Keep coding, keep learning, and keep building amazing things with TypeScript!

---

**Series Complete: Parts 1-5**
**Total Words: ~73,000**
**Total Time Investment: Comprehensive mastery**
**Value: Junior to Senior level expertise**

**Happy coding! 💻✨**
