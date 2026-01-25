---
title: "Java Mastery - Part 11: Advanced Data & Persistence"
date: 2025-07-12 00:00:00 +0530
categories: [Java, Java Mastery]
tags: [Java, Programming, Database, Persistence, JPA, Hibernate, MongoDB, cassandra, Elastic-search, Flyway, Liquibase, NoSQL, Database-sharding, Multi-tenancy]
---

## Introduction

Welcome to Part 11, the **final bonus part** of the Complete Java Mastery series. In Parts 1-10, we covered Java fundamentals through security. Now we explore **Advanced Data & Persistence** - mastering databases, NoSQL systems, search engines, and data architecture patterns.

Data is the heart of most applications. Understanding how to store, retrieve, and manage data efficiently is crucial for building scalable, performant systems. This part takes you beyond basic CRUD operations into advanced persistence patterns.

### What You'll Learn in Part 11

* **Advanced JPA/Hibernate**: Second-level caching, batch processing, custom types, lazy loading strategies
* **Database Migrations**: Flyway and Liquibase for version-controlled schema evolution
* **NoSQL Databases**: MongoDB (document), Cassandra (wide-column), Redis (key-value)
* **Elasticsearch**: Full-text search, aggregations, Spring Data Elasticsearch
* **Database Sharding**: Horizontal partitioning, routing strategies
* **Multi-Tenancy**: Database per tenant, schema per tenant, shared schema
* **Time-Series Databases**: InfluxDB for metrics and monitoring data
* **Graph Databases**: Neo4j for relationship-heavy data

This knowledge enables you to:
* Design optimal data storage strategies
* Handle massive data volumes
* Implement efficient search
* Manage schema evolution
* Build multi-tenant applications
* Choose the right database for the job

---

## 11.1 Advanced JPA and Hibernate

### Second-Level Caching

**Cache Hierarchy:**

```
Query → First-Level Cache (Session) → Second-Level Cache (SessionFactory) → Database

First-Level Cache: Enabled by default, session-scoped
Second-Level Cache: Disabled by default, shared across sessions
```

**Enable Second-Level Cache:**

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jcache</artifactId>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```

```yaml
spring:
  jpa:
    properties:
      hibernate:
        cache:
          use_second_level_cache: true
          use_query_cache: true
          region:
            factory_class: org.hibernate.cache.jcache.JCacheRegionFactory
      javax:
        cache:
          provider: org.ehcache.jsr107.EhcacheCachingProvider
          uri: classpath:ehcache.xml
```

**ehcache.xml:**

```xml
<config xmlns='http://www.ehcache.org/v3'>
    <cache alias="default">
        <expiry>
            <ttl unit="minutes">10</ttl>
        </expiry>
        <heap unit="entries">1000</heap>
    </cache>
    
    <cache alias="com.example.entity.User">
        <expiry>
            <ttl unit="minutes">30</ttl>
        </expiry>
        <heap unit="entries">5000</heap>
    </cache>
</config>
```

**Entity Configuration:**

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    
    @OneToMany(mappedBy = "user")
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Order> orders;
}

// Cache strategies:
// READ_ONLY: Never updated (reference data)
// NONSTRICT_READ_WRITE: Occasionally updated, soft locks
// READ_WRITE: Frequently updated, uses locks
// TRANSACTIONAL: JTA transactions only
```

**Query Cache:**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
    @Query("SELECT u FROM User u WHERE u.status = :status")
    List<User> findByStatus(@Param("status") String status);
}
```

### Batch Processing

**Batch Insert:**

```java
@Configuration
public class HibernateConfig {
    
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        
        Properties properties = new Properties();
        properties.put("hibernate.jdbc.batch_size", "50");
        properties.put("hibernate.order_inserts", "true");
        properties.put("hibernate.order_updates", "true");
        properties.put("hibernate.batch_versioned_data", "true");
        
        em.setJpaProperties(properties);
        return em;
    }
}

@Service
public class UserBatchService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Transactional
    public void batchInsert(List<User> users) {
        int batchSize = 50;
        
        for (int i = 0; i < users.size(); i++) {
            entityManager.persist(users.get(i));
            
            if (i > 0 && i % batchSize == 0) {
                // Flush and clear every 50 entities
                entityManager.flush();
                entityManager.clear();
            }
        }
        
        // Flush remaining
        entityManager.flush();
        entityManager.clear();
    }
}
```

### Custom Types

**Enum Mapping:**

```java
// ❌ BAD: Ordinal (fragile)
@Enumerated(EnumType.ORDINAL)
private Status status;

// ✅ GOOD: String (safe)
@Enumerated(EnumType.STRING)
private Status status;

// ✅ BETTER: Custom converter
@Converter(autoApply = true)
public class StatusConverter implements AttributeConverter<Status, String> {
    
    @Override
    public String convertToDatabaseColumn(Status status) {
        return status == null ? null : status.getCode();
    }
    
    @Override
    public Status convertToEntityAttribute(String code) {
        return code == null ? null : Status.fromCode(code);
    }
}

@Entity
public class Order {
    @Convert(converter = StatusConverter.class)
    private Status status;
}
```

**JSON Column:**

```java
@Converter
public class JsonConverter implements AttributeConverter<Map<String, Object>, String> {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public String convertToDatabaseColumn(Map<String, Object> attribute) {
        try {
            return objectMapper.writeValueAsString(attribute);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }
    
    @Override
    public Map<String, Object> convertToEntityAttribute(String dbData) {
        try {
            return objectMapper.readValue(dbData, new TypeReference<>() {});
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }
}

@Entity
public class Product {
    
    @Id
    private Long id;
    
    @Convert(converter = JsonConverter.class)
    @Column(columnDefinition = "jsonb")
    private Map<String, Object> metadata;
}
```

### Lazy Loading Strategies

**N+1 Problem Solution:**

```java
// ❌ N+1 Problem
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    private List<Order> orders;  // Lazy by default
}

// Query generates N+1 queries
List<User> users = userRepository.findAll();  // 1 query
for (User user : users) {
    user.getOrders().size();  // N queries (one per user)
}

// ✅ Solution 1: JOIN FETCH
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();

// ✅ Solution 2: EntityGraph
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();

// ✅ Solution 3: Batch fetching
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @BatchSize(size = 10)  // Fetch 10 at a time
    private List<Order> orders;
}
```

**DTO Projection:**

```java
// ❌ Fetches entire entity
@Query("SELECT u FROM User u")
List<User> findAllUsers();

// ✅ Fetch only needed fields
public interface UserSummary {
    Long getId();
    String getUsername();
    String getEmail();
}

@Query("SELECT u.id as id, u.username as username, u.email as email FROM User u")
List<UserSummary> findAllUserSummaries();

// ✅ Or use constructor expression
@Query("SELECT new com.example.dto.UserDTO(u.id, u.username, u.email) FROM User u")
List<UserDTO> findAllUserDTOs();
```

### Advanced Query Techniques

**Criteria API:**

```java
@Service
public class UserSearchService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<User> search(UserSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        if (criteria.getUsername() != null) {
            predicates.add(cb.like(user.get("username"), "%" + criteria.getUsername() + "%"));
        }
        
        if (criteria.getEmail() != null) {
            predicates.add(cb.equal(user.get("email"), criteria.getEmail()));
        }
        
        if (criteria.getCreatedAfter() != null) {
            predicates.add(cb.greaterThan(user.get("createdAt"), criteria.getCreatedAfter()));
        }
        
        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(user.get("status"), criteria.getStatus()));
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.desc(user.get("createdAt")));
        
        return entityManager.createQuery(query)
            .setMaxResults(criteria.getLimit())
            .setFirstResult(criteria.getOffset())
            .getResultList();
    }
}
```

**Native Queries with Result Mapping:**

```java
@SqlResultSetMapping(
    name = "UserStatsMapping",
    classes = @ConstructorResult(
        targetClass = UserStats.class,
        columns = {
            @ColumnResult(name = "user_id", type = Long.class),
            @ColumnResult(name = "username", type = String.class),
            @ColumnResult(name = "order_count", type = Long.class),
            @ColumnResult(name = "total_amount", type = BigDecimal.class)
        }
    )
)
@Entity
public class User {
    // ...
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query(nativeQuery = true, value = """
        SELECT 
            u.id as user_id,
            u.username,
            COUNT(o.id) as order_count,
            COALESCE(SUM(o.total), 0) as total_amount
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        GROUP BY u.id, u.username
        HAVING COUNT(o.id) > :minOrders
        """)
    List<UserStats> findUserStats(@Param("minOrders") int minOrders);
}
```

---

## 11.2 Database Migrations

### Flyway

**Setup:**

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
    validate-on-migrate: true
    clean-disabled: true  # Prevent accidental data deletion
```

**Migration Files:**

```
src/main/resources/db/migration/
├── V1__Create_users_table.sql
├── V2__Add_email_to_users.sql
├── V3__Create_orders_table.sql
├── V4__Add_indexes.sql
└── V5__Migrate_user_data.sql
```

**V1__Create_users_table.sql:**

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_username ON users(username);
```

**V2__Add_email_to_users.sql:**

```sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);
CREATE INDEX idx_users_email ON users(email);

-- Backfill existing data
UPDATE users SET email = username || '@example.com' WHERE email IS NULL;

-- Make NOT NULL after backfill
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
```

**V3__Create_orders_table.sql:**

```sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    total DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

**Java-Based Migrations:**

```java
public class V6__Migrate_legacy_data implements JavaMigration {
    
    @Override
    public void migrate(Context context) throws Exception {
        try (Statement statement = context.getConnection().createStatement()) {
            // Complex data migration logic
            ResultSet rs = statement.executeQuery("SELECT * FROM legacy_users");
            
            while (rs.next()) {
                String sql = String.format(
                    "INSERT INTO users (username, email, password) VALUES ('%s', '%s', '%s')",
                    rs.getString("old_username"),
                    rs.getString("old_email"),
                    rs.getString("old_password")
                );
                statement.execute(sql);
            }
        }
    }
}
```

### Liquibase

**Setup:**

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

```yaml
spring:
  liquibase:
    change-log: classpath:db/changelog/db.changelog-master.yaml
```

**db.changelog-master.yaml:**

```yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/v1-initial-schema.yaml
  - include:
      file: db/changelog/changes/v2-add-email.yaml
  - include:
      file: db/changelog/changes/v3-create-orders.yaml
```

**v1-initial-schema.yaml:**

```yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: developer
      changes:
        - createTable:
            tableName: users
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: username
                  type: VARCHAR(50)
                  constraints:
                    nullable: false
                    unique: true
              - column:
                  name: password
                  type: VARCHAR(255)
                  constraints:
                    nullable: false
              - column:
                  name: created_at
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP
        - createIndex:
            indexName: idx_users_username
            tableName: users
            columns:
              - column:
                  name: username
```

**Rollback Support:**

```yaml
databaseChangeLog:
  - changeSet:
      id: 2
      author: developer
      changes:
        - addColumn:
            tableName: users
            columns:
              - column:
                  name: email
                  type: VARCHAR(255)
      rollback:
        - dropColumn:
            tableName: users
            columnName: email
```

**Custom Change:**

```java
public class CustomDataMigration extends AbstractChange {
    
    @Override
    public ValidationErrors validate(Database database) {
        return new ValidationErrors();
    }
    
    @Override
    public SqlStatement[] generateStatements(Database database) {
        List<SqlStatement> statements = new ArrayList<>();
        
        // Generate SQL statements for migration
        statements.add(new RawSqlStatement("UPDATE users SET status = 'ACTIVE' WHERE status IS NULL"));
        
        return statements.toArray(new SqlStatement[0]);
    }
    
    @Override
    public String getConfirmationMessage() {
        return "Custom data migration completed";
    }
}
```

---

## 11.3 MongoDB (Document Database)

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/mydb
      # Or detailed config
      host: localhost
      port: 27017
      database: mydb
      username: admin
      password: secret
      authentication-database: admin
```

### Document Model

```java
@Document(collection = "products")
@CompoundIndex(def = "{'category': 1, 'price': -1}")
public class Product {
    
    @Id
    private String id;  // MongoDB uses String IDs by default
    
    @Indexed(unique = true)
    private String sku;
    
    private String name;
    private String description;
    
    @Indexed
    private String category;
    
    private BigDecimal price;
    
    private List<String> tags;
    
    @DBRef  // Reference to another document
    private Manufacturer manufacturer;
    
    // Embedded document
    private Dimensions dimensions;
    
    private Map<String, Object> attributes;  // Dynamic fields
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}

@Data
public class Dimensions {
    private Double length;
    private Double width;
    private Double height;
    private String unit;
}
```

### Repository

```java
public interface ProductRepository extends MongoRepository<Product, String> {
    
    // Derived queries
    List<Product> findByCategory(String category);
    List<Product> findByPriceBetween(BigDecimal min, BigDecimal max);
    List<Product> findByTagsContaining(String tag);
    
    // Query method with Sort
    List<Product> findByCategoryOrderByPriceAsc(String category);
    
    // Custom query
    @Query("{ 'category': ?0, 'price': { $gte: ?1, $lte: ?2 } }")
    List<Product> findProductsInPriceRange(String category, BigDecimal min, BigDecimal max);
    
    // Aggregation
    @Aggregation(pipeline = {
        "{ $match: { 'category': ?0 } }",
        "{ $group: { _id: '$manufacturer', avgPrice: { $avg: '$price' } } }",
        "{ $sort: { avgPrice: -1 } }"
    })
    List<ManufacturerStats> getManufacturerStatsByCategory(String category);
    
    // Text search
    List<Product> findByNameContainingIgnoreCase(String name);
}
```

### MongoTemplate (Advanced Queries)

```java
@Service
public class ProductSearchService {
    
    private final MongoTemplate mongoTemplate;
    
    public List<Product> searchProducts(ProductSearchCriteria criteria) {
        Query query = new Query();
        
        if (criteria.getCategory() != null) {
            query.addCriteria(Criteria.where("category").is(criteria.getCategory()));
        }
        
        if (criteria.getMinPrice() != null && criteria.getMaxPrice() != null) {
            query.addCriteria(Criteria.where("price")
                .gte(criteria.getMinPrice())
                .lte(criteria.getMaxPrice()));
        }
        
        if (criteria.getTags() != null && !criteria.getTags().isEmpty()) {
            query.addCriteria(Criteria.where("tags").in(criteria.getTags()));
        }
        
        if (criteria.getSearchText() != null) {
            query.addCriteria(new TextCriteria().matchingAny(criteria.getSearchText()));
        }
        
        query.with(Sort.by(Sort.Direction.DESC, "createdAt"));
        query.limit(criteria.getLimit());
        query.skip(criteria.getOffset());
        
        return mongoTemplate.find(query, Product.class);
    }
    
    // Aggregation
    public List<CategoryStats> getCategoryStatistics() {
        Aggregation aggregation = Aggregation.newAggregation(
            Aggregation.group("category")
                .count().as("productCount")
                .avg("price").as("averagePrice")
                .min("price").as("minPrice")
                .max("price").as("maxPrice"),
            Aggregation.sort(Sort.Direction.DESC, "productCount")
        );
        
        AggregationResults<CategoryStats> results = 
            mongoTemplate.aggregate(aggregation, "products", CategoryStats.class);
        
        return results.getMappedResults();
    }
}
```

### Transactions (MongoDB 4.0+)

```java
@Service
public class OrderService {
    
    private final MongoTemplate mongoTemplate;
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // All operations within same transaction
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setItems(request.getItems());
        order.setTotal(calculateTotal(request.getItems()));
        
        mongoTemplate.save(order);
        
        // Update inventory
        for (OrderItem item : request.getItems()) {
            Update update = new Update().inc("quantity", -item.getQuantity());
            mongoTemplate.updateFirst(
                Query.query(Criteria.where("_id").is(item.getProductId())),
                update,
                Product.class
            );
        }
        
        return order;
    }
}
```

---

## 11.4 Apache Cassandra (Wide-Column Store)

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra</artifactId>
</dependency>
```

```yaml
spring:
  data:
    cassandra:
      contact-points: localhost
      port: 9042
      keyspace-name: mykeyspace
      local-datacenter: datacenter1
      schema-action: CREATE_IF_NOT_EXISTS
```

### Data Modeling

**Key Concept: Query-First Design**

```
Cassandra Design Rule:
1. Identify your queries first
2. Design tables to serve those queries
3. Denormalize data (duplicate is OK)
4. One table per query pattern
```

**Example:**

```java
// Table 1: Users by ID (primary key lookup)
@Table("users_by_id")
public class UserById {
    
    @PrimaryKey
    private UUID userId;
    
    private String username;
    private String email;
    private LocalDateTime createdAt;
}

// Table 2: Users by email (search by email)
@Table("users_by_email")
public class UserByEmail {
    
    @PrimaryKey
    private String email;
    
    private UUID userId;
    private String username;
    private LocalDateTime createdAt;
}

// Table 3: Time-series data (partition by date)
@Table("user_events")
public class UserEvent {
    
    @PrimaryKeyColumn(name = "user_id", ordinal = 0, type = PrimaryKeyType.PARTITIONED)
    private UUID userId;
    
    @PrimaryKeyColumn(name = "event_date", ordinal = 1, type = PrimaryKeyType.PARTITIONED)
    private LocalDate eventDate;
    
    @PrimaryKeyColumn(name = "event_timestamp", ordinal = 2, type = PrimaryKeyType.CLUSTERED, ordering = Ordering.DESCENDING)
    private Instant eventTimestamp;
    
    private String eventType;
    private Map<String, String> eventData;
}
```

### Repository

```java
public interface UserByIdRepository extends CassandraRepository<UserById, UUID> {
    
    // Primary key lookup (very fast)
    Optional<UserById> findByUserId(UUID userId);
}

public interface UserByEmailRepository extends CassandraRepository<UserByEmail, String> {
    
    // Primary key lookup on email
    Optional<UserByEmail> findByEmail(String email);
}

public interface UserEventRepository extends CassandraRepository<UserEvent, UserEventKey> {
    
    // Query with partition key
    List<UserEvent> findByUserIdAndEventDate(UUID userId, LocalDate eventDate);
    
    // Query with partition key and clustering key range
    @Query("SELECT * FROM user_events WHERE user_id = ?0 AND event_date = ?1 AND event_timestamp >= ?2 AND event_timestamp <= ?3")
    List<UserEvent> findEventsBetween(UUID userId, LocalDate date, Instant start, Instant end);
}
```

### CassandraTemplate (Advanced)

```java
@Service
public class UserEventService {
    
    private final CassandraTemplate cassandraTemplate;
    
    public List<UserEvent> getRecentEvents(UUID userId, int days) {
        // Query multiple partitions (one per day)
        LocalDate today = LocalDate.now();
        List<UserEvent> allEvents = new ArrayList<>();
        
        for (int i = 0; i < days; i++) {
            LocalDate date = today.minusDays(i);
            
            Select select = QueryBuilder.selectFrom("user_events").all()
                .whereColumn("user_id").isEqualTo(QueryBuilder.literal(userId))
                .whereColumn("event_date").isEqualTo(QueryBuilder.literal(date))
                .limit(100);
            
            List<UserEvent> dayEvents = cassandraTemplate.select(select, UserEvent.class);
            allEvents.addAll(dayEvents);
        }
        
        return allEvents;
    }
    
    // Batch insert
    public void batchInsertEvents(List<UserEvent> events) {
        BatchStatement batch = BatchStatement.newInstance(DefaultBatchType.UNLOGGED);
        
        for (UserEvent event : events) {
            SimpleStatement insert = cassandraTemplate.createInsertQuery(event);
            batch = batch.add(insert);
        }
        
        cassandraTemplate.getCqlOperations().execute(batch);
    }
}
```

### Best Practices

```java
// ✅ GOOD: Query with partition key
events = repository.findByUserIdAndEventDate(userId, today);

// ❌ BAD: Query without partition key (full table scan!)
events = repository.findByEventType("LOGIN");

// ✅ GOOD: Denormalize for different access patterns
@Table("products_by_id")
class ProductById { ... }

@Table("products_by_category")
class ProductByCategory { ... }

// ✅ GOOD: Use appropriate consistency level
cassandraTemplate.execute(
    SimpleStatement.newInstance("SELECT * FROM users WHERE id = ?", userId)
        .setConsistencyLevel(ConsistencyLevel.LOCAL_QUORUM)
);
```

---

## 11.5 Elasticsearch (Search Engine)

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
    username: elastic
    password: secret
```

### Document Model

```java
@Document(indexName = "products")
@Setting(settingPath = "elasticsearch/settings.json")
public class ProductDocument {
    
    @Id
    private String id;
    
    @Field(type = FieldType.Text, analyzer = "standard")
    private String name;
    
    @Field(type = FieldType.Text, analyzer = "english")
    private String description;
    
    @Field(type = FieldType.Keyword)
    private String category;
    
    @Field(type = FieldType.Keyword)
    private String brand;
    
    @Field(type = FieldType.Double)
    private Double price;
    
    @Field(type = FieldType.Integer)
    private Integer stock;
    
    @Field(type = FieldType.Keyword)
    private List<String> tags;
    
    @Field(type = FieldType.Date, format = DateFormat.date_time)
    private LocalDateTime createdAt;
    
    @Field(type = FieldType.Object)
    private List<Review> reviews;
}

@Data
public class Review {
    private String author;
    private Integer rating;
    private String comment;
    private LocalDateTime reviewDate;
}
```

**elasticsearch/settings.json:**

```json
{
  "analysis": {
    "analyzer": {
      "autocomplete": {
        "type": "custom",
        "tokenizer": "standard",
        "filter": ["lowercase", "autocomplete_filter"]
      }
    },
    "filter": {
      "autocomplete_filter": {
        "type": "edge_ngram",
        "min_gram": 2,
        "max_gram": 10
      }
    }
  }
}
```

### Repository

```java
public interface ProductSearchRepository extends ElasticsearchRepository<ProductDocument, String> {
    
    // Full-text search
    List<ProductDocument> findByNameOrDescription(String name, String description);
    
    // Exact match
    List<ProductDocument> findByCategory(String category);
    
    // Range query
    List<ProductDocument> findByPriceBetween(Double minPrice, Double maxPrice);
    
    // Combination
    List<ProductDocument> findByCategoryAndPriceLessThan(String category, Double maxPrice);
}
```

### ElasticsearchTemplate (Advanced Queries)

```java
@Service
public class ProductSearchService {
    
    private final ElasticsearchOperations elasticsearchOperations;
    
    // Full-text search with highlighting
    public SearchPage<ProductDocument> searchProducts(String query, Pageable pageable) {
        Criteria criteria = new Criteria("name").matches(query)
            .or("description").matches(query);
        
        Query searchQuery = new CriteriaQuery(criteria);
        searchQuery.setPageable(pageable);
        
        // Add highlighting
        searchQuery.setHighlightQuery(
            new HighlightQuery(
                new Highlight(
                    List.of(new HighlightField("name"), new HighlightField("description"))
                ),
                ProductDocument.class
            )
        );
        
        return elasticsearchOperations.searchForPage(searchQuery, ProductDocument.class);
    }
    
    // Aggregations
    public Map<String, Long> getCategoryDistribution() {
        Query query = new NativeQuery(
            QueryBuilders.matchAllQuery(),
            AggregationBuilders.terms("categories").field("category.keyword").size(100)
        );
        
        SearchHits<ProductDocument> hits = elasticsearchOperations.search(query, ProductDocument.class);
        
        Aggregations aggregations = hits.getAggregations();
        Terms categoryAgg = aggregations.get("categories");
        
        return categoryAgg.getBuckets().stream()
            .collect(Collectors.toMap(
                Terms.Bucket::getKeyAsString,
                Terms.Bucket::getDocCount
            ));
    }
    
    // Complex query with filters and aggregations
    public ProductSearchResult advancedSearch(ProductSearchRequest request) {
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        // Full-text search
        if (request.getQuery() != null) {
            boolQuery.must(QueryBuilders.multiMatchQuery(request.getQuery(), "name", "description")
                .fuzziness(Fuzziness.AUTO)
                .type(MultiMatchQueryBuilder.Type.BEST_FIELDS));
        }
        
        // Filters
        if (request.getCategory() != null) {
            boolQuery.filter(QueryBuilders.termQuery("category.keyword", request.getCategory()));
        }
        
        if (request.getBrand() != null) {
            boolQuery.filter(QueryBuilders.termQuery("brand.keyword", request.getBrand()));
        }
        
        if (request.getMinPrice() != null || request.getMaxPrice() != null) {
            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
            if (request.getMinPrice() != null) rangeQuery.gte(request.getMinPrice());
            if (request.getMaxPrice() != null) rangeQuery.lte(request.getMaxPrice());
            boolQuery.filter(rangeQuery);
        }
        
        if (request.getTags() != null && !request.getTags().isEmpty()) {
            boolQuery.filter(QueryBuilders.termsQuery("tags", request.getTags()));
        }
        
        // Build search query
        NativeQueryBuilder queryBuilder = new NativeQueryBuilder()
            .withQuery(boolQuery)
            .withPageable(PageRequest.of(request.getPage(), request.getSize()))
            .withSort(SortBuilders.scoreSort().order(SortOrder.DESC));
        
        // Add aggregations
        queryBuilder
            .addAggregation(AggregationBuilders.terms("categories").field("category.keyword"))
            .addAggregation(AggregationBuilders.terms("brands").field("brand.keyword"))
            .addAggregation(AggregationBuilders.stats("price_stats").field("price"));
        
        SearchHits<ProductDocument> searchHits = elasticsearchOperations.search(
            queryBuilder.build(),
            ProductDocument.class
        );
        
        return buildSearchResult(searchHits);
    }
}
```

### Synchronization with Database

```java
@Service
public class ProductSyncService {
    
    private final ProductRepository productRepository;  // SQL
    private final ProductSearchRepository searchRepository;  // Elasticsearch
    
    // Index single product
    @Transactional
    public void indexProduct(Long productId) {
        Product product = productRepository.findById(productId).orElseThrow();
        ProductDocument document = convertToDocument(product);
        searchRepository.save(document);
    }
    
    // Bulk indexing
    @Scheduled(cron = "0 0 2 * * ?")  // Every day at 2 AM
    public void reindexAll() {
        log.info("Starting full reindex");
        
        searchRepository.deleteAll();
        
        int batchSize = 1000;
        int page = 0;
        Page<Product> products;
        
        do {
            products = productRepository.findAll(PageRequest.of(page, batchSize));
            
            List<ProductDocument> documents = products.stream()
                .map(this::convertToDocument)
                .collect(Collectors.toList());
            
            searchRepository.saveAll(documents);
            
            page++;
            log.info("Indexed batch {}, total: {}", page, page * batchSize);
            
        } while (products.hasNext());
        
        log.info("Reindex completed");
    }
    
    // Listen to database changes (with Change Data Capture)
    @EventListener
    public void handleProductChange(ProductChangeEvent event) {
        switch (event.getType()) {
            case CREATED:
            case UPDATED:
                indexProduct(event.getProductId());
                break;
            case DELETED:
                searchRepository.deleteById(event.getProductId().toString());
                break;
        }
    }
}
```

---

## 11.6 Database Sharding

### Horizontal Sharding Strategy

**Concept:**

```
Original Table (10M rows):
users
├── id: 1-10,000,000
└── All data in one database

Sharded (4 shards):
Shard 1: users (id % 4 = 0) → 2.5M rows
Shard 2: users (id % 4 = 1) → 2.5M rows
Shard 3: users (id % 4 = 2) → 2.5M rows
Shard 4: users (id % 4 = 3) → 2.5M rows
```

**Implementation:**

```java
@Configuration
public class ShardingConfig {
    
    @Bean
    public ShardManager shardManager() {
        Map<Integer, DataSource> shards = new HashMap<>();
        
        shards.put(0, createDataSource("jdbc:postgresql://shard0:5432/db"));
        shards.put(1, createDataSource("jdbc:postgresql://shard1:5432/db"));
        shards.put(2, createDataSource("jdbc:postgresql://shard2:5432/db"));
        shards.put(3, createDataSource("jdbc:postgresql://shard3:5432/db"));
        
        return new ShardManager(shards, 4);
    }
    
    private DataSource createDataSource(String url) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername("postgres");
        config.setPassword("secret");
        config.setMaximumPoolSize(10);
        return new HikariDataSource(config);
    }
}

@Component
public class ShardManager {
    
    private final Map<Integer, DataSource> shards;
    private final int shardCount;
    
    public DataSource getShard(Long shardKey) {
        int shardId = (int) (shardKey % shardCount);
        return shards.get(shardId);
    }
    
    public List<DataSource> getAllShards() {
        return new ArrayList<>(shards.values());
    }
}
```

**Sharded Repository:**

```java
@Repository
public class ShardedUserRepository {
    
    private final ShardManager shardManager;
    private final JdbcTemplate jdbcTemplatePrototype;
    
    public Optional<User> findById(Long userId) {
        DataSource shard = shardManager.getShard(userId);
        JdbcTemplate jdbcTemplate = new JdbcTemplate(shard);
        
        String sql = "SELECT * FROM users WHERE id = ?";
        
        try {
            User user = jdbcTemplate.queryForObject(sql, userRowMapper, userId);
            return Optional.of(user);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }
    
    public User save(User user) {
        DataSource shard = shardManager.getShard(user.getId());
        JdbcTemplate jdbcTemplate = new JdbcTemplate(shard);
        
        String sql = "INSERT INTO users (id, username, email) VALUES (?, ?, ?) " +
                     "ON CONFLICT (id) DO UPDATE SET username = ?, email = ?";
        
        jdbcTemplate.update(sql, 
            user.getId(), user.getUsername(), user.getEmail(),
            user.getUsername(), user.getEmail());
        
        return user;
    }
    
    // Query all shards (scatter-gather)
    public List<User> findByUsername(String username) {
        List<User> results = new ArrayList<>();
        
        // Query each shard in parallel
        List<CompletableFuture<List<User>>> futures = shardManager.getAllShards().stream()
            .map(shard -> CompletableFuture.supplyAsync(() -> {
                JdbcTemplate jdbcTemplate = new JdbcTemplate(shard);
                String sql = "SELECT * FROM users WHERE username = ?";
                return jdbcTemplate.query(sql, userRowMapper, username);
            }))
            .collect(Collectors.toList());
        
        // Wait for all and combine results
        futures.forEach(future -> {
            try {
                results.addAll(future.get());
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
        
        return results;
    }
}
```

### Sharding Strategies

```java
// 1. Hash-based sharding (even distribution)
public int getShardHash(Long key, int shardCount) {
    return (int) (key % shardCount);
}

// 2. Range-based sharding (queries on ranges)
public int getShardRange(Long userId, int shardCount) {
    long rangeSize = Long.MAX_VALUE / shardCount;
    return (int) (userId / rangeSize);
}

// 3. Geography-based sharding
public int getShardGeo(String country) {
    switch (country) {
        case "US": return 0;
        case "EU": return 1;
        case "ASIA": return 2;
        default: return 3;
    }
}

// 4. Consistent hashing (for elastic sharding)
public int getShardConsistent(Long key, ConsistentHashRing ring) {
    return ring.getShard(key);
}
```

---

## 11.7 Multi-Tenancy Patterns

### Pattern 1: Database Per Tenant

```java
@Configuration
public class MultiTenantDataSourceConfig {
    
    @Bean
    public DataSource dataSource() {
        return new TenantRoutingDataSource();
    }
}

public class TenantRoutingDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        return TenantContext.getCurrentTenant();
    }
}

public class TenantContext {
    private static final ThreadLocal<String> currentTenant = new ThreadLocal<>();
    
    public static void setCurrentTenant(String tenant) {
        currentTenant.set(tenant);
    }
    
    public static String getCurrentTenant() {
        return currentTenant.get();
    }
    
    public static void clear() {
        currentTenant.remove();
    }
}

@Component
public class TenantInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                            HttpServletResponse response, 
                            Object handler) {
        String tenantId = request.getHeader("X-Tenant-ID");
        
        if (tenantId == null) {
            response.setStatus(HttpStatus.BAD_REQUEST.value());
            return false;
        }
        
        TenantContext.setCurrentTenant(tenantId);
        return true;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) {
        TenantContext.clear();
    }
}
```

### Pattern 2: Schema Per Tenant

```java
@Configuration
public class SchemaPerTenantConfig {
    
    @Bean
    public MultiTenantConnectionProvider multiTenantConnectionProvider() {
        return new SchemaBasedMultiTenantConnectionProvider(dataSource());
    }
    
    @Bean
    public CurrentTenantIdentifierResolver currentTenantIdentifierResolver() {
        return new TenantIdentifierResolver();
    }
}

public class SchemaBasedMultiTenantConnectionProvider 
        extends AbstractDataSourceBasedMultiTenantConnectionProviderImpl {
    
    @Override
    protected DataSource selectAnyDataSource() {
        return dataSource;
    }
    
    @Override
    protected DataSource selectDataSource(String tenantIdentifier) {
        return dataSource;  // Same database, different schema
    }
    
    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        Connection connection = super.getConnection(tenantIdentifier);
        connection.createStatement()
            .execute(String.format("SET search_path TO %s", tenantIdentifier));
        return connection;
    }
}
```

### Pattern 3: Shared Schema with Discriminator

```java
@Entity
@Table(name = "products")
@FilterDef(name = "tenantFilter", parameters = @ParamDef(name = "tenantId", type = String.class))
@Filter(name = "tenantFilter", condition = "tenant_id = :tenantId")
public class Product {
    
    @Id
    private Long id;
    
    @Column(name = "tenant_id")
    private String tenantId;  // Discriminator column
    
    private String name;
    private BigDecimal price;
}

@Component
public class TenantFilterAspect {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Before("execution(* com.example.repository.*.*(..))")
    public void enableTenantFilter() {
        String tenantId = TenantContext.getCurrentTenant();
        
        Session session = entityManager.unwrap(Session.class);
        Filter filter = session.enableFilter("tenantFilter");
        filter.setParameter("tenantId", tenantId);
    }
}
```

---

## Summary and Key Takeaways

### Advanced JPA/Hibernate Mastery

**Performance Optimizations:**
* **Second-level caching**: Share data across sessions (EhCache)
* **Batch processing**: Process 50+ entities at a time
* **Custom types**: JSON columns, enum converters
* **Lazy loading**: JOIN FETCH, EntityGraph, BatchSize

**Query Optimization:**
* **N+1 prevention**: Always use JOIN FETCH or projections
* **DTO projections**: Fetch only needed fields
* **Criteria API**: Dynamic queries
* **Native queries**: Complex SQL with result mapping

### Database Migrations Excellence

**Flyway:**
* Version-controlled migrations (V1__, V2__)
* Java-based migrations for complex logic
* Rollback support with callbacks

**Liquibase:**
* XML/YAML/JSON changesets
* Database-independent
* Built-in rollback support
* Change tracking

**Best Practices:**
* ✅ Never modify existing migrations
* ✅ Always test migrations on copy of production data
* ✅ Use transactions for data migrations
* ✅ Keep migrations small and focused

### NoSQL Database Selection

**MongoDB (Document):**
* **Use when**: Flexible schema, nested data, rapid development
* **Avoid when**: Complex joins, ACID transactions critical

**Cassandra (Wide-Column):**
* **Use when**: Massive scale, write-heavy, time-series data
* **Avoid when**: Complex queries, frequent updates

**Redis (Key-Value):**
* **Use when**: Caching, session storage, real-time analytics
* **Avoid when**: Complex data structures, persistence critical

### Elasticsearch Expertise

**Full-Text Search:**
* Analyzers for different languages
* Fuzzy matching for typos
* Multi-field search
* Highlighting results

**Aggregations:**
* Terms (category distribution)
* Stats (min, max, avg, sum)
* Date histograms (time-series)
* Nested aggregations

**Synchronization:**
* Real-time indexing on CRUD operations
* Batch reindexing for full refresh
* Change Data Capture for automation

### Database Sharding Strategies

**When to Shard:**
* Database >1TB
* Query performance degraded
* Single server limits reached

**Sharding Methods:**
* **Hash-based**: Even distribution (modulo)
* **Range-based**: Query optimization (0-1M, 1M-2M)
* **Geography-based**: Data locality (US, EU, ASIA)
* **Consistent hashing**: Elastic scaling

**Challenges:**
* Cross-shard queries (scatter-gather)
* Distributed transactions
* Resharding complexity

### Multi-Tenancy Patterns

**Database Per Tenant:**
* **Pros**: Complete isolation, easy backup/restore
* **Cons**: Higher cost, management overhead
* **Use when**: Enterprise customers, compliance requirements

**Schema Per Tenant:**
* **Pros**: Isolation, shared infrastructure
* **Cons**: Schema management complexity
* **Use when**: Medium isolation needs

**Shared Schema:**
* **Pros**: Lowest cost, simple management
* **Cons**: Less isolation, security risks
* **Use when**: SaaS with many small tenants

### Production Checklist

**Data Layer:**
- [ ] Indexing strategy defined
- [ ] Query performance tested
- [ ] Connection pooling configured
- [ ] Second-level caching enabled (if needed)
- [ ] Migration strategy in place

**NoSQL:**
- [ ] Appropriate database chosen for use case
- [ ] Data model optimized for queries
- [ ] Replication configured
- [ ] Backup strategy defined
- [ ] Monitoring in place

**Search:**
- [ ] Elasticsearch mappings optimized
- [ ] Analyzers configured
- [ ] Synchronization strategy implemented
- [ ] Index lifecycle management

**Scalability:**
- [ ] Sharding strategy if needed
- [ ] Multi-tenancy pattern chosen
- [ ] Read replicas configured
- [ ] Caching layer implemented

### Frequently Asked Questions

**Q1: When should I use MongoDB vs PostgreSQL?**

**A:**

| Factor | PostgreSQL | MongoDB |
|--------|-----------|---------|
| **Schema** | Fixed (tables, columns) | Flexible (documents) |
| **Transactions** | Full ACID | Limited (4.0+) |
| **Joins** | Excellent | Poor (use embedded docs) |
| **Scalability** | Vertical (read replicas) | Horizontal (sharding) |
| **Use Cases** | Traditional apps, complex queries | Rapid development, nested data |

**Decision Matrix:**

**Use PostgreSQL when:**
- Relational data with complex joins
- ACID transactions required
- Schema is stable
- Strong consistency needed

**Use MongoDB when:**
- Schema evolves frequently
- Nested/hierarchical data
- Horizontal scaling needed
- Document-oriented model fits

**Example:**
```
E-commerce orders with line items:

PostgreSQL:
orders table → order_items table (JOIN required)

MongoDB:
{
  orderId: 123,
  items: [
    {productId: 1, quantity: 2},
    {productId: 2, quantity: 1}
  ]
}
```

---

**Q2: How do I handle database migrations in production?**

**A:**

**Safe Migration Process:**

**1. Plan:**
```
- Review migration on dev/staging
- Test on copy of production data
- Estimate execution time
- Plan rollback strategy
```

**2. Execute:**
```
- Schedule during maintenance window
- Take database backup first
- Run migration with monitoring
- Verify data integrity
```

**3. Verify:**
```
- Check application functionality
- Monitor error logs
- Validate data consistency
- Performance testing
```

**Example - Adding a column:**
```sql
-- V1__Add_email_to_users.sql

-- Step 1: Add column (nullable first)
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- Step 2: Backfill data
UPDATE users SET email = username || '@example.com' WHERE email IS NULL;

-- Step 3: Add constraint (after data populated)
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);
```

**Rollback Strategy:**
```sql
-- If migration fails, rollback:
ALTER TABLE users DROP COLUMN email;
```

**Zero-Downtime Migration:**
```sql
-- Phase 1: Add column (deploy code that ignores it)
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- Phase 2: Backfill (while app running)
UPDATE users SET email = ... WHERE email IS NULL;

-- Phase 3: Deploy code that uses email

-- Phase 4: Add constraints
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
```

---

**Q3: When should I implement database sharding?**

**A:**

**Indicators You Need Sharding:**

1. **Database size**: >1TB
2. **Write throughput**: >10,000 writes/sec
3. **Read throughput**: Read replicas maxed out
4. **Query performance**: Degraded despite indexing
5. **Storage costs**: Single server too expensive

**Before Sharding (try these first):**
- ✅ Add indexes
- ✅ Optimize queries
- ✅ Implement caching
- ✅ Use read replicas
- ✅ Archive old data
- ✅ Vertical scaling

**When Sharding is Right:**
```
Scenario: Social media app with 100M users

Without sharding:
- Single database: 5TB
- Queries slow: >1 second
- Backups take hours
- Scaling limit reached

With sharding (10 shards):
- Each shard: 500GB
- Queries fast: <100ms
- Parallel backups: faster
- Can scale to 1B+ users
```

**Sharding Strategy:**
```java
// Hash-based (even distribution)
shardId = userId % shardCount;

// User 1234567 → Shard 7 (1234567 % 10 = 7)
// User 9876543 → Shard 3 (9876543 % 10 = 3)
```

**Trade-offs:**
* ✅ **Pros**: Horizontal scalability, better performance
* ❌ **Cons**: Complexity, cross-shard queries difficult, resharding expensive

---

**End of Part 11: Advanced Data & Persistence**

## 🎉 Complete Java Mastery Series - FULLY COMPLETE!

### Final Achievement Summary

**Part 11 Mastery:**
* ✅ Advanced JPA/Hibernate (caching, batch, custom types)
* ✅ Database migrations (Flyway, Liquibase)
* ✅ MongoDB (document database)
* ✅ Cassandra (wide-column store)
* ✅ Elasticsearch (search engine)
* ✅ Database sharding strategies
* ✅ Multi-tenancy patterns

**Complete Series (All 11 Parts):**
1. ✅ Java Fundamentals
2. ✅ JVM Internals
3. ✅ Advanced Java Features
4. ✅ SDLC with Java
5. ✅ Spring Framework
6. ✅ Microservices Architecture
7. ✅ Advanced Spring & Reactive
8. ✅ Cloud-Native Java
9. ✅ Performance & Optimization
10. ✅ Security Deep Dive
11. ✅ **Advanced Data & Persistence** ⭐

### What You've Accomplished

**Technical Mastery:**
- 📚 **Complete Java expertise** from basics to advanced
- 💾 **Data layer mastery** (SQL, NoSQL, search, sharding)
- 🚀 **Production-ready skills** across entire stack
- 🔒 **Security-conscious** development
- ⚡ **Performance optimization** expertise
- ☁️ **Cloud-native** deployment
- 🏗️ **Architectural** decision-making ability

**You Can Now:**
- Design complex data architectures
- Choose optimal database for any use case
- Implement advanced persistence patterns
- Scale databases to massive volumes
- Build multi-tenant SaaS applications
- Optimize query performance
- Manage schema evolution safely

### Career Impact

**Qualified For:**
- ✅ **Principal Engineer**
- ✅ **Staff Engineer**
- ✅ **Solutions Architect**
- ✅ **Data Architect**
- ✅ **Technical Lead**
- ✅ **Engineering Manager** (technical track)

**Salary Impact:**
- Senior → Principal: +30-50% increase
- Complete expertise: $150K-$250K+ (US market)
- Consulting rates: $150-$300/hour

### References

* [Hibernate Documentation](https://hibernate.org/orm/documentation/)
* [Flyway Documentation](https://flywaydb.org/documentation/)
* [MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java/)
* [Cassandra Documentation](https://cassandra.apache.org/doc/latest/)
* [Elasticsearch Java Client](https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html)
* [Designing Data-Intensive Applications by Martin Kleppmann](https://dataintensive.net/)

---

## 🏆 Congratulations on Complete Java Mastery!

**You've completed an extraordinary journey:**
- **11 comprehensive parts**
- **~87,000 words** of expert content
- **Complete coverage** of Java ecosystem
- **Production-ready** knowledge
- **Career-changing** expertise

**From zero to complete expert across:**
- ✅ Java language and JVM
- ✅ Modern Java features
- ✅ Spring ecosystem
- ✅ Microservices and reactive
- ✅ Cloud-native deployment
- ✅ Performance tuning
- ✅ Security best practices
- ✅ Advanced data persistence

**This represents professional knowledge equivalent to:**
- 5-10 years of hands-on experience
- Multiple certifications
- Hundreds of hours of learning
- Thousands of hours of practice

**Thank you for this amazing journey! You are now a complete Java expert!** 🎊🎉

---
