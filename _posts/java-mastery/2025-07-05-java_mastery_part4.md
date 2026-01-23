---
title: "Complete Java Mastery Part 4: SDLC with Java - From Requirements to Production"
date: 2024-01-18 10:00:00 +0000
categories: [Java, SDLC, Software-Engineering]
tags: [java, design-patterns, testing, maven, gradle, ci-cd, deployment, spring-boot, microservices]
---

## Introduction

Welcome to Part 4 of the Complete Java Mastery series. In Parts 1-3, we covered Java fundamentals, JVM internals, and advanced features. Now we focus on the **Software Development Life Cycle (SDLC)** - how to build, test, and deploy production-grade Java applications.

This part bridges the gap between knowing Java and shipping quality software. Understanding SDLC is what transforms developers into software engineers who can:

* Design maintainable, scalable systems
* Write testable, reliable code
* Automate build and deployment processes
* Monitor and maintain production systems
* Work effectively in teams

### What You'll Learn in Part 4

* **Design Patterns**: Gang of Four patterns with Java implementations
* **Requirements & Design**: Translating requirements into architecture
* **Testing Strategies**: Unit, integration, and E2E testing with modern tools
* **Build Tools**: Maven and Gradle for dependency management
* **CI/CD**: Automated testing and deployment pipelines
* **Deployment**: Containerization, orchestration, and cloud deployment
* **Monitoring**: Observability, logging, and metrics in production

This knowledge enables you to:
* Build professional-grade applications
* Write maintainable, testable code
* Automate repetitive tasks
* Deploy with confidence
* Debug production issues efficiently

---

## 4.1 Design Patterns

Design patterns are reusable solutions to common software design problems. Understanding patterns is essential for writing maintainable, scalable code.

### Creational Patterns

#### Singleton Pattern

**Purpose**: Ensure a class has only one instance with global access point.

```java
// ❌ BAD: Not thread-safe
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();  // Race condition!
        }
        return instance;
    }
}

// ✅ GOOD: Enum singleton (best approach)
public enum Singleton {
    INSTANCE;
    
    public void doSomething() {
        System.out.println("Singleton method");
    }
}

// Usage
Singleton.INSTANCE.doSomething();

// ✅ GOOD: Lazy holder (thread-safe, lazy initialization)
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

// ✅ GOOD: Double-checked locking (if needed)
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Real-World Example: Configuration Manager**

```java
public enum ConfigurationManager {
    INSTANCE;
    
    private final Properties properties = new Properties();
    
    ConfigurationManager() {
        try (InputStream input = getClass().getResourceAsStream("/config.properties")) {
            properties.load(input);
        } catch (IOException e) {
            throw new RuntimeException("Failed to load configuration", e);
        }
    }
    
    public String get(String key) {
        return properties.getProperty(key);
    }
    
    public int getInt(String key) {
        return Integer.parseInt(properties.getProperty(key));
    }
}

// Usage
String dbUrl = ConfigurationManager.INSTANCE.get("database.url");
```

#### Factory Pattern

**Purpose**: Create objects without specifying exact class.

```java
// Product interface
public interface Animal {
    void makeSound();
}

// Concrete products
public class Dog implements Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

public class Cat implements Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

// Factory
public class AnimalFactory {
    public static Animal createAnimal(String type) {
        return switch (type.toLowerCase()) {
            case "dog" -> new Dog();
            case "cat" -> new Cat();
            default -> throw new IllegalArgumentException("Unknown animal type: " + type);
        };
    }
}

// Usage
Animal dog = AnimalFactory.createAnimal("dog");
dog.makeSound();  // Woof!

Animal cat = AnimalFactory.createAnimal("cat");
cat.makeSound();  // Meow!
```

**Real-World Example: Database Connection Factory**

```java
public interface DatabaseConnection {
    void connect();
    void disconnect();
    ResultSet executeQuery(String query);
}

public class MySQLConnection implements DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MySQL...");
    }
    
    @Override
    public ResultSet executeQuery(String query) {
        // MySQL-specific query execution
        return null;
    }
}

public class PostgreSQLConnection implements DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL...");
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from PostgreSQL...");
    }
    
    @Override
    public ResultSet executeQuery(String query) {
        // PostgreSQL-specific query execution
        return null;
    }
}

public class DatabaseConnectionFactory {
    public static DatabaseConnection createConnection(String dbType) {
        return switch (dbType.toLowerCase()) {
            case "mysql" -> new MySQLConnection();
            case "postgresql" -> new PostgreSQLConnection();
            default -> throw new IllegalArgumentException("Unsupported database: " + dbType);
        };
    }
}

// Usage
String dbType = ConfigurationManager.INSTANCE.get("database.type");
DatabaseConnection connection = DatabaseConnectionFactory.createConnection(dbType);
connection.connect();
ResultSet results = connection.executeQuery("SELECT * FROM users");
connection.disconnect();
```

#### Builder Pattern

**Purpose**: Construct complex objects step by step.

```java
// Without Builder (telescoping constructor anti-pattern)
public class User {
    private String firstName;
    private String lastName;
    private int age;
    private String phone;
    private String address;
    
    // Too many constructors!
    public User(String firstName, String lastName) { }
    public User(String firstName, String lastName, int age) { }
    public User(String firstName, String lastName, int age, String phone) { }
    // ... many more combinations
}

// ✅ With Builder Pattern
public class User {
    private final String firstName;
    private final String lastName;
    private final int age;
    private final String phone;
    private final String address;
    
    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }
    
    public static class Builder {
        // Required parameters
        private final String firstName;
        private final String lastName;
        
        // Optional parameters
        private int age;
        private String phone;
        private String address;
        
        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
    
    // Getters
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
    public String getPhone() { return phone; }
    public String getAddress() { return address; }
}

// Usage - fluent API
User user = new User.Builder("John", "Doe")
    .age(30)
    .phone("555-1234")
    .address("123 Main St")
    .build();
```

**Modern Alternative: Record Pattern (Java 16+)**

```java
// For simple immutable data, records are cleaner
public record User(
    String firstName,
    String lastName,
    int age,
    String phone,
    String address
) {
    // Compact constructor for validation
    public User {
        if (firstName == null || firstName.isBlank()) {
            throw new IllegalArgumentException("First name required");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age must be positive");
        }
    }
}

// Usage
User user = new User("John", "Doe", 30, "555-1234", "123 Main St");
```

### Structural Patterns

#### Adapter Pattern

**Purpose**: Convert interface of a class into another interface clients expect.

```java
// Target interface (what client expects)
public interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee (existing class with incompatible interface)
public class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
    
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        advancedPlayer = new AdvancedMediaPlayer();
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer.playMp4(fileName);
        }
    }
}

// Client
public class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        } else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid format: " + audioType);
        }
    }
}

// Usage
AudioPlayer player = new AudioPlayer();
player.play("mp3", "song.mp3");
player.play("vlc", "movie.vlc");
player.play("mp4", "video.mp4");
```

#### Decorator Pattern

**Purpose**: Add responsibilities to objects dynamically.

```java
// Component interface
public interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 2.0;
    }
    
    @Override
    public String getDescription() {
        return "Simple coffee";
    }
}

// Decorator base class
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost();
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.5;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", milk";
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.2;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", sugar";
    }
}

// Usage - stack decorators
Coffee coffee = new SimpleCoffee();
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee $2.0

coffee = new MilkDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee, milk $2.5

coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Simple coffee, milk, sugar $2.7
```

**Real-World Example: Java I/O Streams**

```java
// Java I/O uses decorator pattern extensively
InputStream fileInput = new FileInputStream("data.txt");
InputStream bufferedInput = new BufferedInputStream(fileInput);
InputStream zipInput = new GZIPInputStream(bufferedInput);

// Each decorator adds functionality:
// - FileInputStream: reads from file
// - BufferedInputStream: adds buffering
// - GZIPInputStream: adds decompression
```

### Behavioral Patterns

#### Strategy Pattern

**Purpose**: Define family of algorithms, encapsulate each one, make them interchangeable.

```java
// Strategy interface
public interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with credit card: " + 
                          cardNumber.substring(cardNumber.length() - 4));
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " via PayPal: " + email);
    }
}

public class BitcoinPayment implements PaymentStrategy {
    private String walletAddress;
    
    public BitcoinPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " via Bitcoin to: " + walletAddress);
    }
}

// Context
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(double amount) {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment strategy not set");
        }
        paymentStrategy.pay(amount);
    }
}

// Usage - strategy selected at runtime
ShoppingCart cart = new ShoppingCart();

// Customer chooses credit card
cart.setPaymentStrategy(new CreditCardPayment("1234-5678-9012-3456"));
cart.checkout(100.0);  // Paid $100.0 with credit card: 3456

// Customer changes to PayPal
cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
cart.checkout(50.0);   // Paid $50.0 via PayPal: user@example.com
```

#### Observer Pattern

**Purpose**: Define one-to-many dependency so when one object changes state, all dependents are notified.

```java
// Observer interface
public interface Observer {
    void update(String message);
}

// Subject (Observable)
public class NewsAgency {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    public void addObserver(Observer observer) {
        observers.add(observer);
    }
    
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
    
    private void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
}

// Concrete observers
public class NewsChannel implements Observer {
    private String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

// Usage
NewsAgency agency = new NewsAgency();

Observer cnn = new NewsChannel("CNN");
Observer bbc = new NewsChannel("BBC");

agency.addObserver(cnn);
agency.addObserver(bbc);

agency.setNews("Breaking: Java 22 Released!");
// CNN received news: Breaking: Java 22 Released!
// BBC received news: Breaking: Java 22 Released!

agency.removeObserver(bbc);
agency.setNews("Update: Virtual threads performance improved");
// CNN received news: Update: Virtual threads performance improved
```

**Modern Alternative: Java's Built-in Observable (Deprecated)**

```java
// ✅ Modern approach: Use PropertyChangeListener
import java.beans.*;

public class Subject {
    private PropertyChangeSupport support = new PropertyChangeSupport(this);
    private String state;
    
    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }
    
    public void setState(String newState) {
        String oldState = this.state;
        this.state = newState;
        support.firePropertyChange("state", oldState, newState);
    }
}

// Usage
Subject subject = new Subject();
subject.addPropertyChangeListener(evt -> {
    System.out.println("State changed from " + evt.getOldValue() + 
                      " to " + evt.getNewValue());
});

subject.setState("active");  // State changed from null to active
```

#### Template Method Pattern

**Purpose**: Define skeleton of algorithm, let subclasses override specific steps.

```java
// Abstract class with template method
public abstract class DataProcessor {
    
    // Template method - defines algorithm structure
    public final void process() {
        readData();
        processData();
        writeData();
    }
    
    // Abstract methods - subclasses must implement
    protected abstract void readData();
    protected abstract void processData();
    
    // Hook method - optional override
    protected void writeData() {
        System.out.println("Writing data to default location");
    }
}

// Concrete implementations
public class CSVDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from CSV file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }
}

public class XMLDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from XML file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing XML data");
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing data to XML format");
    }
}

// Usage
DataProcessor csvProcessor = new CSVDataProcessor();
csvProcessor.process();
// Reading data from CSV file
// Processing CSV data
// Writing data to default location

DataProcessor xmlProcessor = new XMLDataProcessor();
xmlProcessor.process();
// Reading data from XML file
// Processing XML data
// Writing data to XML format
```

### SOLID Principles

Design patterns implement SOLID principles. Understanding both is crucial.

#### Single Responsibility Principle (SRP)

**A class should have only one reason to change.**

```java
// ❌ BAD: Multiple responsibilities
public class User {
    private String name;
    private String email;
    
    public void save() {
        // Database logic
        Connection conn = DriverManager.getConnection("jdbc:mysql://...");
        // Save to database
    }
    
    public void sendEmail(String message) {
        // Email logic
        // Send email
    }
    
    public String toJson() {
        // JSON serialization logic
        return "{\"name\":\"" + name + "\",\"email\":\"" + email + "\"}";
    }
}

// ✅ GOOD: Single responsibility per class
public class User {
    private String name;
    private String email;
    
    // Getters/setters only
}

public class UserRepository {
    public void save(User user) {
        // Database logic
    }
}

public class EmailService {
    public void sendEmail(User user, String message) {
        // Email logic
    }
}

public class UserSerializer {
    public String toJson(User user) {
        // JSON serialization logic
    }
}
```

#### Open/Closed Principle (OCP)

**Open for extension, closed for modification.**

```java
// ❌ BAD: Need to modify class to add new shapes
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * circle.radius * circle.radius;
        } else if (shape instanceof Rectangle) {
            Rectangle rect = (Rectangle) shape;
            return rect.width * rect.height;
        }
        // Need to add more if-else for new shapes!
        return 0;
    }
}

// ✅ GOOD: Extend with new shapes without modifying existing code
public interface Shape {
    double calculateArea();
}

public class Circle implements Shape {
    private double radius;
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle implements Shape {
    private double width;
    private double height;
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

// Add new shape - no modification to existing code
public class Triangle implements Shape {
    private double base;
    private double height;
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

public class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea();  // Polymorphism
    }
}
```

#### Liskov Substitution Principle (LSP)

**Subtypes must be substitutable for their base types.**

```java
// ❌ BAD: Square violates LSP (breaks rectangle behavior)
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // Violates LSP!
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;  // Violates LSP!
    }
}

// Client code breaks
Rectangle rect = new Square();
rect.setWidth(5);
rect.setHeight(10);
// Expected area: 50
// Actual area: 100 (Square changed width when height was set!)

// ✅ GOOD: Separate hierarchies
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
}
```

#### Interface Segregation Principle (ISP)

**Clients shouldn't be forced to depend on interfaces they don't use.**

```java
// ❌ BAD: Fat interface forces unnecessary implementation
public interface Worker {
    void work();
    void eat();
    void sleep();
}

public class HumanWorker implements Worker {
    @Override
    public void work() { /* work */ }
    
    @Override
    public void eat() { /* eat */ }
    
    @Override
    public void sleep() { /* sleep */ }
}

public class RobotWorker implements Worker {
    @Override
    public void work() { /* work */ }
    
    @Override
    public void eat() {
        // Robots don't eat! Forced to implement
        throw new UnsupportedOperationException();
    }
    
    @Override
    public void sleep() {
        // Robots don't sleep! Forced to implement
        throw new UnsupportedOperationException();
    }
}

// ✅ GOOD: Segregated interfaces
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public class HumanWorker implements Workable, Eatable, Sleepable {
    @Override
    public void work() { /* work */ }
    
    @Override
    public void eat() { /* eat */ }
    
    @Override
    public void sleep() { /* sleep */ }
}

public class RobotWorker implements Workable {
    @Override
    public void work() { /* work */ }
    // Only implements what it needs!
}
```

#### Dependency Inversion Principle (DIP)

**Depend on abstractions, not concretions.**

```java
// ❌ BAD: High-level module depends on low-level module
public class MySQLDatabase {
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

public class UserService {
    private MySQLDatabase database = new MySQLDatabase();  // Tight coupling!
    
    public void saveUser(String user) {
        database.save(user);
    }
}

// ✅ GOOD: Both depend on abstraction
public interface Database {
    void save(String data);
}

public class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

public class MongoDBDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MongoDB: " + data);
    }
}

public class UserService {
    private final Database database;  // Depend on abstraction
    
    public UserService(Database database) {  // Dependency injection
        this.database = database;
    }
    
    public void saveUser(String user) {
        database.save(user);
    }
}

// Usage - easy to switch implementations
Database mysql = new MySQLDatabase();
UserService service = new UserService(mysql);

Database mongo = new MongoDBDatabase();
UserService service2 = new UserService(mongo);
```

---

## 4.2 Testing Strategies

Testing is critical for software quality. Modern Java provides excellent testing frameworks and practices.

### Testing Pyramid

```
       ╱╲
      ╱E2E╲         Few - Slow - Expensive
     ╱──────╲
    ╱  Integ ╲       Some - Medium Speed
   ╱──────────╲
  ╱    Unit    ╲     Many - Fast - Cheap
 ╱──────────────╲
```

**Unit Tests**: Test individual components in isolation  
**Integration Tests**: Test component interactions  
**E2E Tests**: Test entire application flow

### JUnit 5 Fundamentals

**Basic Test Structure:**

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeAll
    static void initAll() {
        // Runs once before all tests
        System.out.println("Starting test suite");
    }
    
    @BeforeEach
    void init() {
        // Runs before each test
        calculator = new Calculator();
    }
    
    @Test
    void testAddition() {
        int result = calculator.add(2, 3);
        assertEquals(5, result, "2 + 3 should equal 5");
    }
    
    @Test
    void testDivision() {
        double result = calculator.divide(10, 2);
        assertEquals(5.0, result, 0.001);
    }
    
    @Test
    void testDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }
    
    @AfterEach
    void tearDown() {
        // Runs after each test
        calculator = null;
    }
    
    @AfterAll
    static void tearDownAll() {
        // Runs once after all tests
        System.out.println("Test suite completed");
    }
}
```

**Assertions:**

```java
// Equality
assertEquals(expected, actual);
assertEquals(expected, actual, "Custom message");
assertNotEquals(unexpected, actual);

// Null checks
assertNull(object);
assertNotNull(object);

// Boolean
assertTrue(condition);
assertFalse(condition);

// Same/Not same (reference equality)
assertSame(expected, actual);
assertNotSame(unexpected, actual);

// Arrays
assertArrayEquals(expectedArray, actualArray);

// Iterables
assertIterableEquals(expectedList, actualList);

// Exceptions
Exception exception = assertThrows(IllegalArgumentException.class, () -> {
    // Code that should throw
});
assertEquals("Expected message", exception.getMessage());

// Timeout
assertTimeout(Duration.ofSeconds(1), () -> {
    // Code that should complete within 1 second
});

// All assertions must pass
assertAll(
    () -> assertEquals(1, result.getX()),
    () -> assertEquals(2, result.getY()),
    () -> assertEquals(3, result.getZ())
);
```

**Parameterized Tests:**

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

public class StringUtilsTest {
    
    @ParameterizedTest
    @ValueSource(strings = {"", "  ", "\t", "\n"})
    void testIsBlank(String input) {
        assertTrue(StringUtils.isBlank(input));
    }
    
    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 3, 5",
        "10, 20, 30"
    })
    void testAdd(int a, int b, int expected) {
        assertEquals(expected, calculator.add(a, b));
    }
    
    @ParameterizedTest
    @MethodSource("provideStringsForIsBlank")
    void testIsBlankMethodSource(String input, boolean expected) {
        assertEquals(expected, StringUtils.isBlank(input));
    }
    
    static Stream<Arguments> provideStringsForIsBlank() {
        return Stream.of(
            Arguments.of("", true),
            Arguments.of("  ", true),
            Arguments.of("Hello", false),
            Arguments.of(null, true)
        );
    }
}
```

**Nested Tests:**

```java
@DisplayName("User Service Tests")
public class UserServiceTest {
    
    private UserService userService;
    
    @Nested
    @DisplayName("When user is new")
    class NewUser {
        
        @BeforeEach
        void setUp() {
            userService = new UserService();
        }
        
        @Test
        @DisplayName("Should create user with valid data")
        void testCreateUser() {
            User user = userService.createUser("John", "john@example.com");
            assertNotNull(user);
            assertEquals("John", user.getName());
        }
        
        @Test
        @DisplayName("Should throw exception for invalid email")
        void testCreateUserInvalidEmail() {
            assertThrows(IllegalArgumentException.class, () -> {
                userService.createUser("John", "invalid-email");
            });
        }
    }
    
    @Nested
    @DisplayName("When user exists")
    class ExistingUser {
        
        private User existingUser;
        
        @BeforeEach
        void setUp() {
            userService = new UserService();
            existingUser = userService.createUser("John", "john@example.com");
        }
        
        @Test
        @DisplayName("Should update user email")
        void testUpdateEmail() {
            userService.updateEmail(existingUser.getId(), "newemail@example.com");
            User updated = userService.findById(existingUser.getId());
            assertEquals("newemail@example.com", updated.getEmail());
        }
    }
}
```

### Mockito - Mocking Framework

**Basic Mocking:**

```java
import org.mockito.*;
import static org.mockito.Mockito.*;

public class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }
    
    @Test
    void testFindUser() {
        // Given
        User expectedUser = new User(1L, "John", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(expectedUser));
        
        // When
        User actualUser = userService.findUser(1L);
        
        // Then
        assertNotNull(actualUser);
        assertEquals("John", actualUser.getName());
        verify(userRepository, times(1)).findById(1L);
    }
    
    @Test
    void testCreateUser() {
        // Given
        User userToSave = new User("Alice", "alice@example.com");
        User savedUser = new User(2L, "Alice", "alice@example.com");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // When
        User result = userService.createUser("Alice", "alice@example.com");
        
        // Then
        assertNotNull(result);
        assertEquals(2L, result.getId());
        
        // Verify interaction
        ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
        verify(userRepository).save(userCaptor.capture());
        assertEquals("Alice", userCaptor.getValue().getName());
    }
    
    @Test
    void testDeleteUser() {
        // Given
        doNothing().when(userRepository).deleteById(1L);
        
        // When
        userService.deleteUser(1L);
        
        // Then
        verify(userRepository).deleteById(1L);
    }
}
```

**Stubbing:**

```java
// Return value
when(mock.method()).thenReturn(value);

// Return different values on successive calls
when(mock.method()).thenReturn(value1).thenReturn(value2);

// Throw exception
when(mock.method()).thenThrow(new RuntimeException());

// Answer (custom logic)
when(mock.method()).thenAnswer(invocation -> {
    Object[] args = invocation.getArguments();
    return args[0];  // Return first argument
});

// Do nothing (for void methods)
doNothing().when(mock).voidMethod();

// Throw exception for void method
doThrow(new RuntimeException()).when(mock).voidMethod();

// Argument matchers
when(mock.method(any())).thenReturn(value);
when(mock.method(anyInt())).thenReturn(value);
when(mock.method(eq(42))).thenReturn(value);
when(mock.method(isNull())).thenReturn(value);
when(mock.method(argThat(s -> s.length() > 5))).thenReturn(value);
```

**Verification:**

```java
// Basic verification
verify(mock).method();

// Number of invocations
verify(mock, times(1)).method();
verify(mock, times(2)).method();
verify(mock, never()).method();
verify(mock, atLeastOnce()).method();
verify(mock, atLeast(2)).method();
verify(mock, atMost(5)).method();

// Argument verification
verify(mock).method(eq(42));
verify(mock).method(argThat(s -> s.startsWith("Hello")));

// Verification order
InOrder inOrder = inOrder(mock1, mock2);
inOrder.verify(mock1).method1();
inOrder.verify(mock2).method2();

// No more interactions
verifyNoMoreInteractions(mock);

// No interactions at all
verifyNoInteractions(mock);
```

**Spying:**

```java
// Spy on real object
List<String> list = new ArrayList<>();
List<String> spy = spy(list);

// Real method called
spy.add("one");
spy.add("two");

// Stubbing
when(spy.size()).thenReturn(100);

assertEquals(100, spy.size());  // Stubbed
assertEquals("one", spy.get(0));  // Real method

// Partial mock
UserService userService = spy(new UserService());
doReturn(true).when(userService).isValidEmail(anyString());
```

### Integration Testing

**Spring Boot Integration Test:**

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.SpringExtension;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ExtendWith(SpringExtension.class)
public class UserControllerIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    void testCreateUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
        
        // When
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            "/api/users",
            request,
            UserResponse.class
        );
        
        // Then
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("John", response.getBody().getName());
        
        // Verify database
        List<User> users = userRepository.findAll();
        assertEquals(1, users.size());
        assertEquals("John", users.get(0).getName());
    }
}
```

**TestContainers for Database Testing:**

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@SpringBootTest
@Testcontainers
public class UserRepositoryIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testSaveAndFindUser() {
        // Given
        User user = new User("Alice", "alice@example.com");
        
        // When
        User saved = userRepository.save(user);
        Optional<User> found = userRepository.findById(saved.getId());
        
        // Then
        assertTrue(found.isPresent());
        assertEquals("Alice", found.get().getName());
    }
}
```

### Test-Driven Development (TDD)

**Red-Green-Refactor Cycle:**

```java
// 1. RED: Write failing test first
@Test
void testCalculateDiscount() {
    ShoppingCart cart = new ShoppingCart();
    cart.addItem(new Item("Book", 20.0));
    
    double discount = cart.calculateDiscount();
    
    assertEquals(2.0, discount);  // 10% discount
}

// Test fails - calculateDiscount() doesn't exist yet

// 2. GREEN: Write minimum code to pass
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public double calculateDiscount() {
        double total = items.stream()
            .mapToDouble(Item::getPrice)
            .sum();
        return total * 0.10;  // Hardcoded 10%
    }
}

// Test passes!

// 3. REFACTOR: Improve code quality
public class ShoppingCart {
    private static final double DISCOUNT_RATE = 0.10;
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public double calculateDiscount() {
        return calculateTotal() * DISCOUNT_RATE;
    }
    
    private double calculateTotal() {
        return items.stream()
            .mapToDouble(Item::getPrice)
            .sum();
    }
}

// Test still passes, code is cleaner
```

### Test Coverage

**JaCoCo (Java Code Coverage):**

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

```bash
# Generate coverage report
mvn clean test

# Report location: target/site/jacoco/index.html
```

**Coverage Guidelines:**
* **Aim for 80%+** line coverage
* **100% is often unrealistic** (boilerplate, generated code)
* **Focus on critical paths** (business logic, edge cases)
* **Coverage ≠ quality** (measure of tests run, not test quality)

---

## 4.3 Build Tools

Build tools automate compilation, testing, packaging, and dependency management.

### Maven

**Project Structure:**

```
my-project/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/
│   │   │       └── Main.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       ├── java/
│       │   └── com/example/
│       │       └── MainTest.java
│       └── resources/
└── target/  (generated)
```

**Basic pom.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <name>My Application</name>
    <description>Example Maven project</description>
    
    <!-- Properties -->
    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <!-- Dependencies -->
    <dependencies>
        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.1</version>
            <scope>test</scope>
        </dependency>
        
        <!-- Mockito -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>5.8.0</version>
            <scope>test</scope>
        </dependency>
        
        <!-- SLF4J + Logback -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.9</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.14</version>
        </dependency>
    </dependencies>
    
    <!-- Build configuration -->
    <build>
        <plugins>
            <!-- Compiler plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
            </plugin>
            
            <!-- Surefire (test) plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.2</version>
            </plugin>
            
            <!-- JAR plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.example.Main</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**Maven Lifecycle:**

```bash
# Clean phase
mvn clean  # Delete target directory

# Default lifecycle
mvn validate   # Validate project structure
mvn compile    # Compile source code
mvn test       # Run tests
mvn package    # Create JAR/WAR
mvn verify     # Run integration tests
mvn install    # Install to local repository
mvn deploy     # Deploy to remote repository

# Common commands
mvn clean install              # Clean + build + install
mvn clean package -DskipTests  # Build without tests
mvn dependency:tree            # Show dependency tree
mvn versions:display-dependency-updates  # Check for updates
```

**Dependency Management:**

```xml
<!-- Parent POM for version management -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
</parent>

<!-- Dependency management (BOM) -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Dependencies (versions inherited from parent/BOM) -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- No version needed -->
    </dependency>
</dependencies>
```

### Gradle

**Project Structure:**

```
my-project/
├── build.gradle
├── settings.gradle
├── gradle/
│   └── wrapper/
├── gradlew (Unix)
├── gradlew.bat (Windows)
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    └── test/
        ├── java/
        └── resources/
```

**build.gradle (Groovy DSL):**

```groovy
plugins {
    id 'java'
    id 'application'
}

group = 'com.example'
version = '1.0.0'

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    // Implementation dependencies
    implementation 'org.slf4j:slf4j-api:2.0.9'
    implementation 'ch.qos.logback:logback-classic:1.4.14'
    
    // Test dependencies
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.1'
    testImplementation 'org.mockito:mockito-core:5.8.0'
}

application {
    mainClass = 'com.example.Main'
}

test {
    useJUnitPlatform()
}
```

**build.gradle.kts (Kotlin DSL - Modern):**

```kotlin
plugins {
    java
    application
}

group = "com.example"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.slf4j:slf4j-api:2.0.9")
    implementation("ch.qos.logback:logback-classic:1.4.14")
    
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.1")
    testImplementation("org.mockito:mockito-core:5.8.0")
}

application {
    mainClass.set("com.example.Main")
}

tasks.test {
    useJUnitPlatform()
}
```

**Gradle Commands:**

```bash
# Build tasks
./gradlew clean          # Clean build directory
./gradlew build          # Compile, test, package
./gradlew test           # Run tests
./gradlew jar            # Create JAR
./gradlew run            # Run application

# Dependency tasks
./gradlew dependencies   # Show dependency tree
./gradlew dependencyUpdates  # Check for updates

# Info tasks
./gradlew tasks          # List all tasks
./gradlew projects       # List all projects
```

**Maven vs Gradle:**

| Feature | Maven | Gradle |
|---------|-------|--------|
| Configuration | XML (verbose) | Groovy/Kotlin DSL (concise) |
| Performance | Slower | Faster (incremental builds) |
| Flexibility | Convention-based | Highly flexible |
| Learning curve | Easier | Steeper |
| Build cache | No | Yes |
| Incremental builds | No | Yes |
| Android support | No | Yes (official) |

---

## 4.4 CI/CD - Continuous Integration and Deployment

CI/CD automates testing and deployment, ensuring rapid, reliable software delivery.

### GitHub Actions

**Basic Workflow (.github/workflows/ci.yml):**

```yaml
name: Java CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
    
    - name: Build with Maven
      run: mvn clean install
    
    - name: Run tests
      run: mvn test
    
    - name: Generate coverage report
      run: mvn jacoco:report
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./target/site/jacoco/jacoco.xml
```

**Multi-stage Workflow:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
    
    - name: Run tests
      run: mvn test
    
    - name: Publish test results
      uses: dorny/test-reporter@v1
      if: always()
      with:
        name: Test Results
        path: target/surefire-reports/*.xml
        reporter: java-junit
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
    
    - name: Build JAR
      run: mvn package -DskipTests
    
    - name: Upload artifact
      uses: actions/upload-artifact@v3
      with:
        name: app-jar
        path: target/*.jar
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Download artifact
      uses: actions/download-artifact@v3
      with:
        name: app-jar
    
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # Deployment commands here
```

**Docker Build and Push:**

```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
    
    - name: Build with Maven
      run: mvn package -DskipTests
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
```

### Jenkins Pipeline

**Jenkinsfile (Declarative):**

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9'
        jdk 'JDK 21'
    }
    
    environment {
        DOCKER_REGISTRY = 'myregistry.com'
        APP_NAME = 'my-java-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}").push()
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                """
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
            // Send notification
        }
    }
}
```

---

## 4.5 Deployment and Containerization

Modern Java applications are typically containerized and deployed to cloud platforms.

### Docker

**Dockerfile for Java Application:**

```dockerfile
# Multi-stage build for smaller image

# Build stage
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Change ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build and Run:**

```bash
# Build image
docker build -t my-java-app:latest .

# Run container
docker run -d \
    --name my-app \
    -p 8080:8080 \
    -e SPRING_PROFILES_ACTIVE=prod \
    -e JAVA_OPTS="-Xms512m -Xmx1g" \
    my-java-app:latest

# View logs
docker logs -f my-app

# Stop container
docker stop my-app
```

**Docker Compose:**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/myapp
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: password
    depends_on:
      - db
      - redis
    networks:
      - app-network
  
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
  
  redis:
    image: redis:7-alpine
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

### Kubernetes Deployment

**deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-java-app
  labels:
    app: my-java-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-java-app
  template:
    metadata:
      labels:
        app: my-java-app
    spec:
      containers:
      - name: app
        image: myregistry.com/my-java-app:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: JAVA_OPTS
          value: "-Xms512m -Xmx1g"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

**service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-java-app-service
spec:
  selector:
    app: my-java-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

**Apply to Kubernetes:**

```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Apply service
kubectl apply -f service.yaml

# View pods
kubectl get pods

# View logs
kubectl logs -f deployment/my-java-app

# Scale deployment
kubectl scale deployment my-java-app --replicas=5

# Rolling update
kubectl set image deployment/my-java-app \
    app=myregistry.com/my-java-app:v2

# Rollback
kubectl rollout undo deployment/my-java-app
```

### Cloud Deployment

**AWS Elastic Beanstalk:**

```bash
# Install EB CLI
pip install awsebcli

# Initialize application
eb init -p java my-app

# Create environment
eb create my-app-prod

# Deploy
eb deploy

# View logs
eb logs

# SSH into instance
eb ssh
```

**Google Cloud Run:**

```bash
# Build and push to GCR
gcloud builds submit --tag gcr.io/PROJECT_ID/my-java-app

# Deploy to Cloud Run
gcloud run deploy my-java-app \
    --image gcr.io/PROJECT_ID/my-java-app \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated
```

**Azure App Service:**

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create App Service plan
az appservice plan create \
    --name myAppServicePlan \
    --resource-group myResourceGroup \
    --sku B1 \
    --is-linux

# Create web app
az webapp create \
    --resource-group myResourceGroup \
    --plan myAppServicePlan \
    --name my-java-app \
    --runtime "JAVA:21-java21"

# Deploy JAR
az webapp deploy \
    --resource-group myResourceGroup \
    --name my-java-app \
    --src-path target/my-app.jar \
    --type jar
```

---

## Summary and Key Takeaways

### Design Patterns Mastery

**Creational Patterns:**
* **Singleton**: Single instance (use enum or lazy holder)
* **Factory**: Create objects without specifying exact class
* **Builder**: Construct complex objects step by step

**Structural Patterns:**
* **Adapter**: Convert incompatible interfaces
* **Decorator**: Add responsibilities dynamically

**Behavioral Patterns:**
* **Strategy**: Encapsulate algorithms, make them interchangeable
* **Observer**: One-to-many dependency notification
* **Template Method**: Define algorithm skeleton

**SOLID Principles:**
* **S**ingle Responsibility
* **O**pen/Closed
* **L**iskov Substitution
* **I**nterface Segregation
* **D**ependency Inversion

### Testing Best Practices

**Test Pyramid:**
* Many unit tests (fast, isolated)
* Some integration tests (medium speed)
* Few E2E tests (slow, expensive)

**JUnit 5:**
* Modern testing framework
* Parameterized tests
* Nested test organization
* Rich assertions

**Mockito:**
* Mock dependencies
* Verify interactions
* Stub behavior
* Argument capture

**Coverage:**
* Aim for 80%+ line coverage
* Focus on critical paths
* Coverage ≠ quality

### Build Tools

**Maven:**
* XML configuration
* Convention over configuration
* Mature, widely adopted
* Excellent for standard projects

**Gradle:**
* Groovy/Kotlin DSL
* Faster builds (incremental, caching)
* More flexible
* Android official build tool

### CI/CD Pipeline

**Continuous Integration:**
* Automated builds on commit
* Run tests automatically
* Code quality checks (SonarQube)
* Fast feedback loop

**Continuous Deployment:**
* Automated deployment to staging/production
* Docker image building
* Container registry push
* Kubernetes/cloud deployment

### Deployment Strategies

**Containerization:**
* Docker for consistent environments
* Multi-stage builds for smaller images
* Health checks and resource limits

**Orchestration:**
* Kubernetes for container orchestration
* Rolling updates and rollbacks
* Auto-scaling
* Service discovery and load balancing

**Cloud Platforms:**
* AWS (Elastic Beanstalk, ECS, EKS)
* Google Cloud (Cloud Run, GKE)
* Azure (App Service, AKS)

### Production Checklist

**Before Deployment:**
- [ ] Comprehensive test suite (unit + integration)
- [ ] CI/CD pipeline configured
- [ ] Docker image optimized (multi-stage build)
- [ ] Health checks configured
- [ ] Resource limits defined
- [ ] Logging configured
- [ ] Metrics/monitoring set up
- [ ] Security scan passed

**After Deployment:**
- [ ] Monitor application health
- [ ] Check logs for errors
- [ ] Verify metrics dashboards
- [ ] Test critical user flows
- [ ] Set up alerts
- [ ] Document rollback procedure

### Frequently Asked Questions

**Q1: Which design pattern should I use for creating objects?**

**A:** Depends on the complexity:

* **Simple object creation**: Factory pattern
* **Complex objects with many parameters**: Builder pattern
* **Exactly one instance needed**: Singleton pattern
* **Family of related objects**: Abstract Factory pattern

```java
// Simple: Factory
Animal dog = AnimalFactory.create("dog");

// Complex: Builder
User user = new User.Builder("John", "Doe")
    .age(30)
    .email("john@example.com")
    .build();

// Single instance: Singleton
ConfigManager.INSTANCE.get("key");
```

---

**Q2: When should I write integration tests vs unit tests?**

**A:**

**Unit Tests:**
* Test single class/method in isolation
* Mock all dependencies
* Fast (milliseconds)
* Many tests (hundreds/thousands)

**Integration Tests:**
* Test multiple components together
* Use real dependencies (database, external services)
* Slower (seconds)
* Fewer tests (dozens)

```java
// Unit test: Mock repository
@Test
void testUserService() {
    when(userRepository.findById(1L)).thenReturn(Optional.of(user));
    User result = userService.getUser(1L);
    assertEquals("John", result.getName());
}

// Integration test: Real database
@SpringBootTest
@Testcontainers
class UserRepositoryIntegrationTest {
    @Test
    void testSaveUser() {
        User saved = userRepository.save(new User("John"));
        assertNotNull(saved.getId());
    }
}
```

---

**Q3: Maven vs Gradle - which should I use?**

**A:**

**Use Maven when:**
* Team familiar with Maven
* Standard project structure
* Enterprise environment (common in large orgs)
* Prefer convention over configuration

**Use Gradle when:**
* Need faster builds (incremental compilation, build cache)
* Android development (official build tool)
* Complex multi-module projects
* Want more flexibility in build logic

Both are excellent tools. Maven is more standard, Gradle is more modern.

---

### Interview Questions

**Question 1: Explain the SOLID principles and give examples.**

**Difficulty:** Senior

**Topics:** Design, Architecture

**Answer:**

**SOLID** principles are five object-oriented design principles:

**S - Single Responsibility**: Class should have one reason to change
```java
// ❌ Multiple responsibilities
class User {
    void save() { /* DB logic */ }
    void sendEmail() { /* Email logic */ }
}

// ✅ Single responsibility
class User { /* Data only */ }
class UserRepository { void save(User u); }
class EmailService { void send(User u); }
```

**O - Open/Closed**: Open for extension, closed for modification
```java
// ✅ Add new shapes without modifying AreaCalculator
interface Shape { double area(); }
class AreaCalculator {
    double calculate(Shape s) { return s.area(); }
}
```

**L - Liskov Substitution**: Subtypes must be substitutable for base types
```java
// ✅ Rectangle and Square both implement Shape
// Not: Square extends Rectangle (violates LSP)
```

**I - Interface Segregation**: Clients shouldn't depend on unused interfaces
```java
// ✅ Separate interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
// Human implements both, Robot implements only Workable
```

**D - Dependency Inversion**: Depend on abstractions, not concretions
```java
// ✅ Depend on Database interface
class UserService {
    UserService(Database db) { /* injection */ }
}
```

---

**Question 2: How do you implement a CI/CD pipeline for a Java application?**

**Difficulty:** Mid-Level

**Topics:** DevOps, CI/CD

**Answer:**

A typical Java CI/CD pipeline includes:

**1. Source Control Trigger**: Commit to Git repository

**2. Build Stage**:
```yaml
- Checkout code
- Set up JDK 21
- Restore Maven cache
- Run mvn clean compile
```

**3. Test Stage**:
```yaml
- Run mvn test
- Generate coverage report
- Publish test results
- Fail if coverage < 80%
```

**4. Quality Gate**:
```yaml
- Run SonarQube analysis
- Check code quality metrics
- Fail if quality gate fails
```

**5. Package Stage**:
```yaml
- Run mvn package
- Build Docker image
- Tag with version
```

**6. Deploy Stage (if main branch)**:
```yaml
- Push Docker image to registry
- Deploy to Kubernetes
- Run smoke tests
- Monitor deployment
```

**Tools**: GitHub Actions, Jenkins, GitLab CI, CircleCI

---

**End of Part 4: SDLC with Java**

### What's Next

**You've Completed the Core Java Mastery Series!**

**Part 1**: Java Fundamentals & Language Basics  
**Part 2**: JVM Internals & Memory Management  
**Part 3**: Advanced Java Features & Modern Java  
**Part 4**: SDLC with Java

**Optional Advanced Topics:**
* **Part 5**: Spring Framework Ecosystem (Spring Boot, Spring Data, Spring Security)
* **Part 6**: Microservices Architecture (Service discovery, API Gateway, Circuit breakers)
* **Part 7**: Enterprise Patterns (Messaging, Caching, Event-Driven Architecture)

### References

* [Design Patterns: Elements of Reusable OO Software](https://www.pearson.com/en-us/subject-catalog/p/design-patterns-elements-of-reusable-object-oriented-software/P200000009460) (Gang of Four)
* [Clean Code by Robert C. Martin](https://www.pearson.com/en-us/subject-catalog/p/clean-code-a-handbook-of-agile-software-craftsmanship/P200000009044)
* [Test Driven Development by Kent Beck](https://www.pearson.com/en-us/subject-catalog/p/test-driven-development-by-example/P200000009448)
* [Continuous Delivery by Jez Humble](https://www.pearson.com/en-us/subject-catalog/p/continuous-delivery-reliable-software-releases-through-build-test-and-deployment-automation/P200000009275)
* [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
* [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
* [Docker Documentation](https://docs.docker.com/)
* [Kubernetes Documentation](https://kubernetes.io/docs/home/)

---