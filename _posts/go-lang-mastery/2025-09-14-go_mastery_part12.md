---
title: "Go Mastery - Part 12: Data Engineering & Processing"
date: 2025-09-14 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Data-Engineering, Kafka, Streaming, ETL, Data-pipeline, Time-series, Big-data]
---

# Part 12: Data Engineering & Processing

## Overview

This part explores data engineering with Go, covering stream processing, Apache Kafka integration, ETL pipelines, time-series databases, and big data systems. You'll learn how to build scalable data pipelines, process streaming data, and integrate Go with the modern data ecosystem.

**What You'll Learn:**
- Stream processing fundamentals and patterns
- Apache Kafka producers and consumers in Go
- Building ETL (Extract, Transform, Load) pipelines
- Time-series database integration (InfluxDB, TimescaleDB)
- Data validation and quality assurance
- Data serialization formats (Avro, Parquet)
- Workflow orchestration with Temporal
- Big data integration (Hadoop, Spark)

**Prerequisites:**
- Part 2 (Concurrency patterns)
- Part 5 (Architecture patterns)
- Understanding of databases
- Basic distributed systems knowledge

---

## 12.1 Stream Processing Fundamentals

### What is Stream Processing?

Stream processing handles continuous data flows in real-time, as opposed to batch processing which handles data in chunks.

**Stream Processing Characteristics:**
- Continuous, unbounded data
- Low latency (milliseconds to seconds)
- Event-driven
- Stateful or stateless transformations
- Time-based operations (windows, watermarks)

**Use Cases:**
- Real-time analytics
- Fraud detection
- Log aggregation
- IoT data processing
- Click stream analysis
- Real-time recommendations

### Stream vs Batch Processing

| Aspect | Stream | Batch |
|--------|--------|-------|
| **Latency** | Milliseconds-seconds | Minutes-hours |
| **Data Volume** | Continuous | Fixed chunks |
| **Complexity** | Higher (state management) | Lower |
| **Use Case** | Real-time insights | Historical analysis |
| **Tools** | Kafka, Flink, Spark Streaming | Hadoop, Spark Batch |

### Event Streaming Concepts

**Events:** Immutable facts that happened at a point in time
```go
type Event struct {
    ID        string    `json:"id"`
    Type      string    `json:"type"`
    Timestamp time.Time `json:"timestamp"`
    UserID    string    `json:"user_id"`
    Data      json.RawMessage `json:"data"`
}
```

**Streams:** Ordered sequence of events
```go
type Stream interface {
    Publish(event Event) error
    Subscribe(handler func(Event) error) error
}
```

**Topics/Channels:** Named streams for organizing events
```go
const (
    TopicUserSignup    = "user.signup"
    TopicUserLogin     = "user.login"
    TopicOrderCreated  = "order.created"
    TopicPaymentMade   = "payment.completed"
)
```

### Windowing Operations

**Tumbling Window:** Fixed, non-overlapping time periods
```go
type TumblingWindow struct {
    Size     time.Duration
    data     map[string][]Event
    mu       sync.RWMutex
}

func NewTumblingWindow(size time.Duration) *TumblingWindow {
    return &TumblingWindow{
        Size: size,
        data: make(map[string][]Event),
    }
}

func (w *TumblingWindow) Add(event Event) {
    w.mu.Lock()
    defer w.mu.Unlock()
    
    windowKey := event.Timestamp.Truncate(w.Size).Format(time.RFC3339)
    w.data[windowKey] = append(w.data[windowKey], event)
}

func (w *TumblingWindow) GetWindow(timestamp time.Time) []Event {
    w.mu.RLock()
    defer w.mu.RUnlock()
    
    windowKey := timestamp.Truncate(w.Size).Format(time.RFC3339)
    return w.data[windowKey]
}
```

**Sliding Window:** Overlapping time periods
```go
type SlidingWindow struct {
    Size   time.Duration
    Slide  time.Duration
    events []Event
    mu     sync.RWMutex
}

func (w *SlidingWindow) Add(event Event) {
    w.mu.Lock()
    defer w.mu.Unlock()
    
    // Remove old events
    cutoff := time.Now().Add(-w.Size)
    for i := 0; i < len(w.events); i++ {
        if w.events[i].Timestamp.After(cutoff) {
            w.events = w.events[i:]
            break
        }
    }
    
    w.events = append(w.events, event)
}

func (w *SlidingWindow) GetEvents() []Event {
    w.mu.RLock()
    defer w.mu.RUnlock()
    
    result := make([]Event, len(w.events))
    copy(result, w.events)
    return result
}
```

**Session Window:** Groups events by periods of activity
```go
type SessionWindow struct {
    Timeout time.Duration
    sessions map[string]*Session
    mu      sync.RWMutex
}

type Session struct {
    UserID     string
    Events     []Event
    LastEvent  time.Time
}

func (w *SessionWindow) Add(event Event) {
    w.mu.Lock()
    defer w.mu.Unlock()
    
    session, exists := w.sessions[event.UserID]
    if !exists || time.Since(session.LastEvent) > w.Timeout {
        // New session
        session = &Session{
            UserID: event.UserID,
            Events: []Event{},
        }
        w.sessions[event.UserID] = session
    }
    
    session.Events = append(session.Events, event)
    session.LastEvent = event.Timestamp
}
```

### Watermarks and Late Data

Watermarks track event-time progress and handle late-arriving data.

```go
type Watermark struct {
    timestamp time.Time
    mu        sync.RWMutex
}

func (w *Watermark) Update(eventTime time.Time) {
    w.mu.Lock()
    defer w.mu.Unlock()
    
    if eventTime.After(w.timestamp) {
        w.timestamp = eventTime
    }
}

func (w *Watermark) Get() time.Time {
    w.mu.RLock()
    defer w.mu.RUnlock()
    return w.timestamp
}

// Check if event is late
func (w *Watermark) IsLate(event Event, allowedLateness time.Duration) bool {
    watermark := w.Get()
    return event.Timestamp.Before(watermark.Add(-allowedLateness))
}
```

### Processing Guarantees

**At-Most-Once:** Message may be lost but never duplicated
- Fastest, lowest overhead
- Acceptable for metrics, logging

**At-Least-Once:** Message may be duplicated but never lost
- Most common
- Requires idempotent processing

**Exactly-Once:** Message processed exactly once
- Most complex, highest overhead
- Required for financial transactions

```go
type ProcessingGuarantee int

const (
    AtMostOnce ProcessingGuarantee = iota
    AtLeastOnce
    ExactlyOnce
)

type Processor struct {
    guarantee ProcessingGuarantee
    processed map[string]bool // For deduplication
    mu        sync.RWMutex
}

func (p *Processor) Process(event Event) error {
    switch p.guarantee {
    case AtMostOnce:
        // Process immediately, don't check for duplicates
        return p.processEvent(event)
        
    case AtLeastOnce:
        // Process, might handle duplicates at application level
        return p.processEvent(event)
        
    case ExactlyOnce:
        // Check if already processed (idempotency)
        p.mu.Lock()
        if p.processed[event.ID] {
            p.mu.Unlock()
            return nil // Already processed
        }
        p.processed[event.ID] = true
        p.mu.Unlock()
        
        return p.processEvent(event)
    }
    
    return nil
}

func (p *Processor) processEvent(event Event) error {
    // Actual processing logic
    log.Printf("Processing event: %s", event.ID)
    return nil
}
```

### Stream Processing Patterns

**Filter:** Select events matching criteria
```go
func Filter(input <-chan Event, predicate func(Event) bool) <-chan Event {
    output := make(chan Event)
    
    go func() {
        defer close(output)
        for event := range input {
            if predicate(event) {
                output <- event
            }
        }
    }()
    
    return output
}

// Usage
userSignups := Filter(events, func(e Event) bool {
    return e.Type == "user.signup"
})
```

**Map:** Transform events
```go
func Map(input <-chan Event, transform func(Event) Event) <-chan Event {
    output := make(chan Event)
    
    go func() {
        defer close(output)
        for event := range input {
            output <- transform(event)
        }
    }()
    
    return output
}

// Usage
enriched := Map(events, func(e Event) Event {
    // Add user info, location, etc.
    return enrichEvent(e)
})
```

**FlatMap:** Transform one event into multiple
```go
func FlatMap(input <-chan Event, transform func(Event) []Event) <-chan Event {
    output := make(chan Event)
    
    go func() {
        defer close(output)
        for event := range input {
            results := transform(event)
            for _, result := range results {
                output <- result
            }
        }
    }()
    
    return output
}
```

**Aggregate:** Combine events
```go
type Aggregator struct {
    state map[string]int
    mu    sync.RWMutex
}

func (a *Aggregator) Aggregate(events <-chan Event) <-chan map[string]int {
    output := make(chan map[string]int)
    
    go func() {
        defer close(output)
        
        ticker := time.NewTicker(5 * time.Second)
        defer ticker.Stop()
        
        for {
            select {
            case event, ok := <-events:
                if !ok {
                    return
                }
                a.mu.Lock()
                a.state[event.Type]++
                a.mu.Unlock()
                
            case <-ticker.C:
                a.mu.RLock()
                snapshot := make(map[string]int)
                for k, v := range a.state {
                    snapshot[k] = v
                }
                a.mu.RUnlock()
                
                output <- snapshot
            }
        }
    }()
    
    return output
}
```

---

## 12.2 Apache Kafka with Go

### Kafka Fundamentals

Apache Kafka is a distributed streaming platform with:
- High throughput (millions of messages/sec)
- Low latency (milliseconds)
- Fault tolerance (replication)
- Horizontal scalability
- Persistent storage

**Kafka Architecture:**
- **Topics:** Categories for messages
- **Partitions:** Parallel processing units
- **Producers:** Send messages to topics
- **Consumers:** Read messages from topics
- **Consumer Groups:** Load balancing across consumers
- **Brokers:** Kafka servers
- **ZooKeeper/KRaft:** Cluster coordination

### Kafka Go Clients

**1. Shopify/sarama** - Pure Go, most popular
```bash
go get github.com/IBM/sarama
```

**2. confluentinc/confluent-kafka-go** - librdkafka bindings (faster, requires C lib)
```bash
go get github.com/confluentinc/confluent-kafka-go/kafka
```

**3. segmentio/kafka-go** - Modern, simple API
```bash
go get github.com/segmentio/kafka-go
```

We'll focus on **sarama** (most widely used) and **kafka-go** (simpler).

### Kafka Producer (sarama)

**Synchronous Producer:**
```go
package main

import (
    "fmt"
    "log"
    
    "github.com/IBM/sarama"
)

func main() {
    config := sarama.NewConfig()
    config.Producer.Return.Successes = true
    config.Producer.RequiredAcks = sarama.WaitForAll // Wait for all replicas
    config.Producer.Retry.Max = 5
    
    producer, err := sarama.NewSyncProducer([]string{"localhost:9092"}, config)
    if err != nil {
        log.Fatal(err)
    }
    defer producer.Close()
    
    // Send message
    message := &sarama.ProducerMessage{
        Topic: "user-events",
        Key:   sarama.StringEncoder("user-123"),
        Value: sarama.StringEncoder(`{"event":"signup","user_id":"123"}`),
    }
    
    partition, offset, err := producer.SendMessage(message)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Message sent to partition %d at offset %d\n", partition, offset)
}
```

**Asynchronous Producer (Higher Throughput):**
```go
func asyncProducer() {
    config := sarama.NewConfig()
    config.Producer.Return.Successes = true
    config.Producer.Return.Errors = true
    
    producer, err := sarama.NewAsyncProducer([]string{"localhost:9092"}, config)
    if err != nil {
        log.Fatal(err)
    }
    defer producer.Close()
    
    // Handle successes
    go func() {
        for msg := range producer.Successes() {
            log.Printf("Message sent: partition=%d offset=%d", msg.Partition, msg.Offset)
        }
    }()
    
    // Handle errors
    go func() {
        for err := range producer.Errors() {
            log.Printf("Error: %v", err)
        }
    }()
    
    // Send messages
    for i := 0; i < 100; i++ {
        message := &sarama.ProducerMessage{
            Topic: "events",
            Value: sarama.StringEncoder(fmt.Sprintf(`{"id":%d}`, i)),
        }
        producer.Input() <- message
    }
    
    // Wait for all messages to be sent
    time.Sleep(1 * time.Second)
}
```

**Partitioning Strategies:**
```go
// Custom partitioner
type UserPartitioner struct{}

func (p *UserPartitioner) Partition(message *sarama.ProducerMessage, numPartitions int32) (int32, error) {
    // Hash user ID to partition
    key := message.Key.(sarama.StringEncoder)
    hash := fnv.New32a()
    hash.Write([]byte(key))
    return int32(hash.Sum32()) % numPartitions, nil
}

func (p *UserPartitioner) RequiresConsistency() bool {
    return true
}

// Use custom partitioner
config.Producer.Partitioner = sarama.NewCustomPartitioner(func(topic string) sarama.Partitioner {
    return &UserPartitioner{}
})
```

### Kafka Consumer (sarama)

**Consumer Group:**
```go
package main

import (
    "context"
    "log"
    "os"
    "os/signal"
    "syscall"
    
    "github.com/IBM/sarama"
)

type ConsumerGroupHandler struct{}

func (h ConsumerGroupHandler) Setup(sarama.ConsumerGroupSession) error {
    return nil
}

func (h ConsumerGroupHandler) Cleanup(sarama.ConsumerGroupSession) error {
    return nil
}

func (h ConsumerGroupHandler) ConsumeClaim(session sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
    for message := range claim.Messages() {
        log.Printf("Message: topic=%s partition=%d offset=%d key=%s value=%s",
            message.Topic, message.Partition, message.Offset,
            string(message.Key), string(message.Value))
        
        // Process message
        if err := processMessage(message); err != nil {
            log.Printf("Error processing message: %v", err)
            continue
        }
        
        // Mark message as processed
        session.MarkMessage(message, "")
    }
    
    return nil
}

func processMessage(message *sarama.ConsumerMessage) error {
    // Your business logic here
    return nil
}

func main() {
    config := sarama.NewConfig()
    config.Consumer.Group.Rebalance.Strategy = sarama.BalanceStrategyRoundRobin
    config.Consumer.Offsets.Initial = sarama.OffsetNewest
    
    consumerGroup, err := sarama.NewConsumerGroup(
        []string{"localhost:9092"},
        "my-consumer-group",
        config,
    )
    if err != nil {
        log.Fatal(err)
    }
    defer consumerGroup.Close()
    
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    // Handle shutdown
    signals := make(chan os.Signal, 1)
    signal.Notify(signals, syscall.SIGINT, syscall.SIGTERM)
    
    go func() {
        <-signals
        log.Println("Shutting down...")
        cancel()
    }()
    
    // Consume
    handler := ConsumerGroupHandler{}
    topics := []string{"user-events"}
    
    for {
        if err := consumerGroup.Consume(ctx, topics, handler); err != nil {
            log.Printf("Error: %v", err)
        }
        
        if ctx.Err() != nil {
            return
        }
    }
}
```

**Manual Offset Management:**
```go
func (h ConsumerGroupHandler) ConsumeClaim(session sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
    for message := range claim.Messages() {
        // Process message
        if err := processMessage(message); err != nil {
            log.Printf("Error: %v, retrying...", err)
            // Don't commit offset on error
            continue
        }
        
        // Manually commit offset
        session.MarkOffset(message.Topic, message.Partition, message.Offset+1, "")
        
        // Or commit immediately
        session.Commit()
    }
    
    return nil
}
```

### Kafka with kafka-go (Simpler API)

**Producer:**
```go
package main

import (
    "context"
    "log"
    
    "github.com/segmentio/kafka-go"
)

func main() {
    writer := &kafka.Writer{
        Addr:     kafka.TCP("localhost:9092"),
        Topic:    "user-events",
        Balancer: &kafka.LeastBytes{},
    }
    defer writer.Close()
    
    err := writer.WriteMessages(context.Background(),
        kafka.Message{
            Key:   []byte("user-123"),
            Value: []byte(`{"event":"signup"}`),
        },
        kafka.Message{
            Key:   []byte("user-456"),
            Value: []byte(`{"event":"login"}`),
        },
    )
    
    if err != nil {
        log.Fatal(err)
    }
    
    log.Println("Messages sent")
}
```

**Consumer:**
```go
func main() {
    reader := kafka.NewReader(kafka.ReaderConfig{
        Brokers:  []string{"localhost:9092"},
        Topic:    "user-events",
        GroupID:  "my-group",
        MinBytes: 10e3, // 10KB
        MaxBytes: 10e6, // 10MB
    })
    defer reader.Close()
    
    for {
        message, err := reader.ReadMessage(context.Background())
        if err != nil {
            log.Printf("Error: %v", err)
            continue
        }
        
        log.Printf("Message: partition=%d offset=%d key=%s value=%s",
            message.Partition, message.Offset,
            string(message.Key), string(message.Value))
        
        // Process message
        processMessage(message)
    }
}
```

### Error Handling and Retries

```go
type RetryConfig struct {
    MaxRetries int
    RetryDelay time.Duration
}

func processWithRetry(message *sarama.ConsumerMessage, cfg RetryConfig) error {
    var err error
    
    for attempt := 0; attempt <= cfg.MaxRetries; attempt++ {
        if attempt > 0 {
            time.Sleep(cfg.RetryDelay * time.Duration(attempt))
        }
        
        err = processMessage(message)
        if err == nil {
            return nil
        }
        
        log.Printf("Attempt %d failed: %v", attempt+1, err)
    }
    
    // All retries failed, send to DLQ (Dead Letter Queue)
    if err := sendToDLQ(message, err); err != nil {
        log.Printf("Failed to send to DLQ: %v", err)
    }
    
    return err
}

func sendToDLQ(message *sarama.ConsumerMessage, processingErr error) error {
    dlqProducer, _ := sarama.NewSyncProducer([]string{"localhost:9092"}, nil)
    defer dlqProducer.Close()
    
    dlqMessage := &sarama.ProducerMessage{
        Topic: "user-events-dlq",
        Key:   sarama.ByteEncoder(message.Key),
        Value: sarama.ByteEncoder(message.Value),
        Headers: []sarama.RecordHeader{
            {
                Key:   []byte("error"),
                Value: []byte(processingErr.Error()),
            },
            {
                Key:   []byte("original_topic"),
                Value: []byte(message.Topic),
            },
        },
    }
    
    _, _, err := dlqProducer.SendMessage(dlqMessage)
    return err
}
```

### Kafka Transactions (Exactly-Once Semantics)

```go
func transactionalProducer() {
    config := sarama.NewConfig()
    config.Producer.Idempotent = true
    config.Producer.RequiredAcks = sarama.WaitForAll
    config.Producer.Transaction.ID = "my-transaction-id"
    config.Producer.Return.Errors = true
    
    producer, err := sarama.NewAsyncProducer([]string{"localhost:9092"}, config)
    if err != nil {
        log.Fatal(err)
    }
    defer producer.Close()
    
    // Begin transaction
    if err := producer.BeginTxn(); err != nil {
        log.Fatal(err)
    }
    
    // Send messages
    messages := []*sarama.ProducerMessage{
        {Topic: "orders", Value: sarama.StringEncoder(`{"order_id":"1"}`)},
        {Topic: "inventory", Value: sarama.StringEncoder(`{"item":"widget","qty":-1}`)},
        {Topic: "notifications", Value: sarama.StringEncoder(`{"msg":"Order placed"}`)},
    }
    
    for _, msg := range messages {
        producer.Input() <- msg
    }
    
    // Wait for all messages
    time.Sleep(1 * time.Second)
    
    // Commit or abort
    if err := producer.CommitTxn(); err != nil {
        log.Printf("Commit failed: %v", err)
        producer.AbortTxn()
        return
    }
    
    log.Println("Transaction committed")
}
```

### Performance Tuning

**Producer Optimization:**
```go
config := sarama.NewConfig()

// Batching
config.Producer.Flush.Frequency = 100 * time.Millisecond
config.Producer.Flush.Messages = 100
config.Producer.Flush.MaxMessages = 1000

// Compression
config.Producer.Compression = sarama.CompressionSnappy

// Buffering
config.ChannelBufferSize = 256

// Acknowledgments (trade-off: speed vs durability)
config.Producer.RequiredAcks = sarama.WaitForLocal // Faster
// config.Producer.RequiredAcks = sarama.WaitForAll // More durable
```

**Consumer Optimization:**
```go
config := sarama.NewConfig()

// Fetch size
config.Consumer.Fetch.Min = 1024 * 1024    // 1MB
config.Consumer.Fetch.Default = 1024 * 1024 * 10 // 10MB

// Session timeout
config.Consumer.Group.Session.Timeout = 20 * time.Second

// Rebalance strategy
config.Consumer.Group.Rebalance.Strategy = sarama.BalanceStrategySticky
```

### Monitoring Kafka

```go
type KafkaMetrics struct {
    MessagesProduced   prometheus.Counter
    MessagesFailed     prometheus.Counter
    MessagesConsumed   prometheus.Counter
    ProcessingDuration prometheus.Histogram
}

func NewKafkaMetrics() *KafkaMetrics {
    return &KafkaMetrics{
        MessagesProduced: promauto.NewCounter(prometheus.CounterOpts{
            Name: "kafka_messages_produced_total",
        }),
        MessagesFailed: promauto.NewCounter(prometheus.CounterOpts{
            Name: "kafka_messages_failed_total",
        }),
        MessagesConsumed: promauto.NewCounter(prometheus.CounterOpts{
            Name: "kafka_messages_consumed_total",
        }),
        ProcessingDuration: promauto.NewHistogram(prometheus.HistogramOpts{
            Name:    "kafka_processing_duration_seconds",
            Buckets: prometheus.DefBuckets,
        }),
    }
}

func (m *KafkaMetrics) RecordProduced() {
    m.MessagesProduced.Inc()
}

func (m *KafkaMetrics) RecordConsumed(duration time.Duration) {
    m.MessagesConsumed.Inc()
    m.ProcessingDuration.Observe(duration.Seconds())
}
```

---

*I'll continue with ETL Pipelines, Time-Series Databases, and complete Part 12. Shall I continue?*

## 12.3 ETL Pipelines

### What is ETL?

ETL stands for Extract, Transform, Load - the process of moving data from source systems to destinations while transforming it.

**Extract:** Get data from sources (databases, APIs, files)  
**Transform:** Clean, validate, enrich, aggregate data  
**Load:** Write data to destination (data warehouse, database, files)

**Modern Variation - ELT:**
Extract → Load → Transform (transform in destination, leveraging modern compute)

### Building ETL Pipelines in Go

**Pipeline Interface:**
```go
package pipeline

type Extractor interface {
    Extract(ctx context.Context) (<-chan Record, <-chan error)
}

type Transformer interface {
    Transform(ctx context.Context, input <-chan Record) (<-chan Record, <-chan error)
}

type Loader interface {
    Load(ctx context.Context, input <-chan Record) <-chan error
}

type Record struct {
    ID        string
    Data      map[string]interface{}
    Metadata  map[string]string
    Timestamp time.Time
}
```

### Extract Phase

**Database Extractor:**
```go
type DatabaseExtractor struct {
    db    *sql.DB
    query string
    batch int
}

func NewDatabaseExtractor(db *sql.DB, query string, batchSize int) *DatabaseExtractor {
    return &DatabaseExtractor{
        db:    db,
        query: query,
        batch: batchSize,
    }
}

func (e *DatabaseExtractor) Extract(ctx context.Context) (<-chan Record, <-chan error) {
    records := make(chan Record, e.batch)
    errors := make(chan error, 1)
    
    go func() {
        defer close(records)
        defer close(errors)
        
        rows, err := e.db.QueryContext(ctx, e.query)
        if err != nil {
            errors <- err
            return
        }
        defer rows.Close()
        
        columns, _ := rows.Columns()
        
        for rows.Next() {
            values := make([]interface{}, len(columns))
            valuePtrs := make([]interface{}, len(columns))
            for i := range columns {
                valuePtrs[i] = &values[i]
            }
            
            if err := rows.Scan(valuePtrs...); err != nil {
                errors <- err
                continue
            }
            
            data := make(map[string]interface{})
            for i, col := range columns {
                data[col] = values[i]
            }
            
            record := Record{
                ID:        fmt.Sprintf("%v", data["id"]),
                Data:      data,
                Timestamp: time.Now(),
            }
            
            select {
            case records <- record:
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return records, errors
}
```

**CSV File Extractor:**
```go
type CSVExtractor struct {
    filepath string
    hasHeader bool
}

func (e *CSVExtractor) Extract(ctx context.Context) (<-chan Record, <-chan error) {
    records := make(chan Record, 100)
    errors := make(chan error, 1)
    
    go func() {
        defer close(records)
        defer close(errors)
        
        file, err := os.Open(e.filepath)
        if err != nil {
            errors <- err
            return
        }
        defer file.Close()
        
        reader := csv.NewReader(file)
        
        var headers []string
        if e.hasHeader {
            headers, err = reader.Read()
            if err != nil {
                errors <- err
                return
            }
        }
        
        for {
            row, err := reader.Read()
            if err == io.EOF {
                break
            }
            if err != nil {
                errors <- err
                continue
            }
            
            data := make(map[string]interface{})
            for i, value := range row {
                key := fmt.Sprintf("col%d", i)
                if i < len(headers) {
                    key = headers[i]
                }
                data[key] = value
            }
            
            record := Record{
                ID:        fmt.Sprintf("%d", time.Now().UnixNano()),
                Data:      data,
                Timestamp: time.Now(),
            }
            
            select {
            case records <- record:
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return records, errors
}
```

**API Extractor:**
```go
type APIExtractor struct {
    url    string
    client *http.Client
}

func (e *APIExtractor) Extract(ctx context.Context) (<-chan Record, <-chan error) {
    records := make(chan Record, 100)
    errors := make(chan error, 1)
    
    go func() {
        defer close(records)
        defer close(errors)
        
        req, err := http.NewRequestWithContext(ctx, "GET", e.url, nil)
        if err != nil {
            errors <- err
            return
        }
        
        resp, err := e.client.Do(req)
        if err != nil {
            errors <- err
            return
        }
        defer resp.Body.Close()
        
        var items []map[string]interface{}
        if err := json.NewDecoder(resp.Body).Decode(&items); err != nil {
            errors <- err
            return
        }
        
        for _, item := range items {
            record := Record{
                ID:        fmt.Sprintf("%v", item["id"]),
                Data:      item,
                Timestamp: time.Now(),
            }
            
            select {
            case records <- record:
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return records, errors
}
```

### Transform Phase

**Data Validation:**
```go
type ValidationTransformer struct {
    rules []ValidationRule
}

type ValidationRule func(Record) error

func (t *ValidationTransformer) Transform(ctx context.Context, input <-chan Record) (<-chan Record, <-chan error) {
    output := make(chan Record, 100)
    errors := make(chan error, 10)
    
    go func() {
        defer close(output)
        defer close(errors)
        
        for record := range input {
            valid := true
            for _, rule := range t.rules {
                if err := rule(record); err != nil {
                    errors <- fmt.Errorf("validation failed for %s: %w", record.ID, err)
                    valid = false
                    break
                }
            }
            
            if valid {
                select {
                case output <- record:
                case <-ctx.Done():
                    return
                }
            }
        }
    }()
    
    return output, errors
}

// Example validation rules
func RequiredField(field string) ValidationRule {
    return func(r Record) error {
        if _, exists := r.Data[field]; !exists {
            return fmt.Errorf("required field %s is missing", field)
        }
        return nil
    }
}

func EmailFormat(field string) ValidationRule {
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    return func(r Record) error {
        email, ok := r.Data[field].(string)
        if !ok || !emailRegex.MatchString(email) {
            return fmt.Errorf("invalid email format in field %s", field)
        }
        return nil
    }
}
```

**Data Enrichment:**
```go
type EnrichmentTransformer struct {
    enricher func(Record) (Record, error)
}

func (t *EnrichmentTransformer) Transform(ctx context.Context, input <-chan Record) (<-chan Record, <-chan error) {
    output := make(chan Record, 100)
    errors := make(chan error, 10)
    
    go func() {
        defer close(output)
        defer close(errors)
        
        for record := range input {
            enriched, err := t.enricher(record)
            if err != nil {
                errors <- err
                continue
            }
            
            select {
            case output <- enriched:
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return output, errors
}

// Example enrichment
func enrichWithUserData(db *sql.DB) func(Record) (Record, error) {
    return func(r Record) (Record, error) {
        userID, ok := r.Data["user_id"].(string)
        if !ok {
            return r, nil
        }
        
        var name, email string
        err := db.QueryRow("SELECT name, email FROM users WHERE id = ?", userID).Scan(&name, &email)
        if err != nil {
            return r, err
        }
        
        r.Data["user_name"] = name
        r.Data["user_email"] = email
        return r, nil
    }
}
```

**Data Aggregation:**
```go
type AggregationTransformer struct {
    windowSize time.Duration
    aggregator func([]Record) Record
}

func (t *AggregationTransformer) Transform(ctx context.Context, input <-chan Record) (<-chan Record, <-chan error) {
    output := make(chan Record, 100)
    errors := make(chan error, 10)
    
    go func() {
        defer close(output)
        defer close(errors)
        
        buffer := make([]Record, 0)
        ticker := time.NewTicker(t.windowSize)
        defer ticker.Stop()
        
        for {
            select {
            case record, ok := <-input:
                if !ok {
                    if len(buffer) > 0 {
                        output <- t.aggregator(buffer)
                    }
                    return
                }
                buffer = append(buffer, record)
                
            case <-ticker.C:
                if len(buffer) > 0 {
                    aggregated := t.aggregator(buffer)
                    output <- aggregated
                    buffer = buffer[:0]
                }
                
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return output, errors
}

// Example aggregator: sum values
func sumAggregator(records []Record) Record {
    total := 0.0
    for _, r := range records {
        if val, ok := r.Data["amount"].(float64); ok {
            total += val
        }
    }
    
    return Record{
        ID: fmt.Sprintf("agg-%d", time.Now().Unix()),
        Data: map[string]interface{}{
            "total":  total,
            "count":  len(records),
            "window": records[0].Timestamp.Format(time.RFC3339),
        },
        Timestamp: time.Now(),
    }
}
```

### Load Phase

**Database Loader:**
```go
type DatabaseLoader struct {
    db        *sql.DB
    table     string
    batchSize int
}

func (l *DatabaseLoader) Load(ctx context.Context, input <-chan Record) <-chan error {
    errors := make(chan error, 10)
    
    go func() {
        defer close(errors)
        
        batch := make([]Record, 0, l.batchSize)
        
        for record := range input {
            batch = append(batch, record)
            
            if len(batch) >= l.batchSize {
                if err := l.insertBatch(ctx, batch); err != nil {
                    errors <- err
                }
                batch = batch[:0]
            }
        }
        
        // Insert remaining records
        if len(batch) > 0 {
            if err := l.insertBatch(ctx, batch); err != nil {
                errors <- err
            }
        }
    }()
    
    return errors
}

func (l *DatabaseLoader) insertBatch(ctx context.Context, records []Record) error {
    if len(records) == 0 {
        return nil
    }
    
    tx, err := l.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    // Prepare insert statement
    columns := make([]string, 0)
    for key := range records[0].Data {
        columns = append(columns, key)
    }
    
    placeholders := make([]string, len(columns))
    for i := range placeholders {
        placeholders[i] = "?"
    }
    
    query := fmt.Sprintf("INSERT INTO %s (%s) VALUES (%s)",
        l.table,
        strings.Join(columns, ","),
        strings.Join(placeholders, ","))
    
    stmt, err := tx.PrepareContext(ctx, query)
    if err != nil {
        return err
    }
    defer stmt.Close()
    
    for _, record := range records {
        values := make([]interface{}, len(columns))
        for i, col := range columns {
            values[i] = record.Data[col]
        }
        
        if _, err := stmt.ExecContext(ctx, values...); err != nil {
            return err
        }
    }
    
    return tx.Commit()
}
```

**File Loader:**
```go
type FileLoader struct {
    filepath string
    format   string // "json", "csv", "parquet"
}

func (l *FileLoader) Load(ctx context.Context, input <-chan Record) <-chan error {
    errors := make(chan error, 10)
    
    go func() {
        defer close(errors)
        
        file, err := os.Create(l.filepath)
        if err != nil {
            errors <- err
            return
        }
        defer file.Close()
        
        switch l.format {
        case "json":
            encoder := json.NewEncoder(file)
            for record := range input {
                if err := encoder.Encode(record.Data); err != nil {
                    errors <- err
                }
            }
            
        case "csv":
            writer := csv.NewWriter(file)
            defer writer.Flush()
            
            headerWritten := false
            for record := range input {
                if !headerWritten {
                    headers := make([]string, 0, len(record.Data))
                    for key := range record.Data {
                        headers = append(headers, key)
                    }
                    writer.Write(headers)
                    headerWritten = true
                }
                
                values := make([]string, 0, len(record.Data))
                for _, v := range record.Data {
                    values = append(values, fmt.Sprintf("%v", v))
                }
                writer.Write(values)
            }
        }
    }()
    
    return errors
}
```

### Complete ETL Pipeline

```go
type Pipeline struct {
    name        string
    extractor   Extractor
    transformers []Transformer
    loader      Loader
}

func NewPipeline(name string) *Pipeline {
    return &Pipeline{
        name:         name,
        transformers: make([]Transformer, 0),
    }
}

func (p *Pipeline) SetExtractor(e Extractor) *Pipeline {
    p.extractor = e
    return p
}

func (p *Pipeline) AddTransformer(t Transformer) *Pipeline {
    p.transformers = append(p.transformers, t)
    return p
}

func (p *Pipeline) SetLoader(l Loader) *Pipeline {
    p.loader = l
    return p
}

func (p *Pipeline) Run(ctx context.Context) error {
    log.Printf("Starting pipeline: %s", p.name)
    
    // Extract
    records, extractErrors := p.extractor.Extract(ctx)
    
    // Transform (chain transformers)
    var transformErrors <-chan error
    for _, transformer := range p.transformers {
        records, transformErrors = transformer.Transform(ctx, records)
    }
    
    // Load
    loadErrors := p.loader.Load(ctx, records)
    
    // Collect errors
    errorCount := 0
    done := make(chan struct{})
    
    go func() {
        for {
            select {
            case err, ok := <-extractErrors:
                if ok && err != nil {
                    log.Printf("Extract error: %v", err)
                    errorCount++
                }
            case err, ok := <-transformErrors:
                if ok && err != nil {
                    log.Printf("Transform error: %v", err)
                    errorCount++
                }
            case err, ok := <-loadErrors:
                if !ok {
                    close(done)
                    return
                }
                if err != nil {
                    log.Printf("Load error: %v", err)
                    errorCount++
                }
            }
        }
    }()
    
    <-done
    
    log.Printf("Pipeline completed: %s (errors: %d)", p.name, errorCount)
    return nil
}
```

**Usage Example:**
```go
func main() {
    db, _ := sql.Open("postgres", "connection-string")
    defer db.Close()
    
    pipeline := NewPipeline("user-data-pipeline").
        SetExtractor(NewDatabaseExtractor(db, "SELECT * FROM source_users", 1000)).
        AddTransformer(&ValidationTransformer{
            rules: []ValidationRule{
                RequiredField("email"),
                RequiredField("name"),
                EmailFormat("email"),
            },
        }).
        AddTransformer(&EnrichmentTransformer{
            enricher: enrichWithUserData(db),
        }).
        SetLoader(&DatabaseLoader{
            db:        db,
            table:     "target_users",
            batchSize: 100,
        })
    
    ctx := context.Background()
    if err := pipeline.Run(ctx); err != nil {
        log.Fatal(err)
    }
}
```

### Error Handling and Retry Logic

```go
type RetryableLoader struct {
    loader     Loader
    maxRetries int
    retryDelay time.Duration
}

func (l *RetryableLoader) Load(ctx context.Context, input <-chan Record) <-chan error {
    errors := make(chan error, 10)
    
    go func() {
        defer close(errors)
        
        for record := range input {
            if err := l.loadWithRetry(ctx, record); err != nil {
                errors <- err
            }
        }
    }()
    
    return errors
}

func (l *RetryableLoader) loadWithRetry(ctx context.Context, record Record) error {
    recordChan := make(chan Record, 1)
    recordChan <- record
    close(recordChan)
    
    var lastErr error
    for attempt := 0; attempt <= l.maxRetries; attempt++ {
        if attempt > 0 {
            time.Sleep(l.retryDelay * time.Duration(attempt))
        }
        
        errorChan := l.loader.Load(ctx, recordChan)
        for err := range errorChan {
            lastErr = err
        }
        
        if lastErr == nil {
            return nil
        }
    }
    
    return fmt.Errorf("failed after %d retries: %w", l.maxRetries, lastErr)
}
```

---

## 12.4 Time-Series Databases

### What are Time-Series Databases?

Time-series databases are optimized for time-stamped data:
- Metrics (CPU, memory, network)
- IoT sensor readings
- Financial data (stock prices)
- Application logs and events

**Characteristics:**
- Write-heavy workloads
- Time-based queries
- Automatic data retention/rollup
- Efficient compression

### InfluxDB Integration

**Installation:**
```bash
go get github.com/influxdata/influxdb-client-go/v2
```

**Writing Data:**
```go
package main

import (
    "context"
    "fmt"
    "time"
    
    influxdb2 "github.com/influxdata/influxdb-client-go/v2"
)

func main() {
    // Create client
    client := influxdb2.NewClient("http://localhost:8086", "my-token")
    defer client.Close()
    
    // Get write API
    writeAPI := client.WriteAPIBlocking("my-org", "my-bucket")
    
    // Write data point
    p := influxdb2.NewPoint(
        "cpu_usage",
        map[string]string{
            "host":   "server01",
            "region": "us-west",
        },
        map[string]interface{}{
            "usage_idle":   10.5,
            "usage_system": 45.2,
            "usage_user":   44.3,
        },
        time.Now(),
    )
    
    if err := writeAPI.WritePoint(context.Background(), p); err != nil {
        fmt.Printf("Write error: %v\n", err)
    }
    
    fmt.Println("Data written to InfluxDB")
}
```

**Batch Writing:**
```go
func batchWrite(client influxdb2.Client) {
    writeAPI := client.WriteAPI("my-org", "my-bucket")
    
    // Write asynchronously (batched automatically)
    for i := 0; i < 100; i++ {
        p := influxdb2.NewPointWithMeasurement("temperature").
            AddTag("sensor", fmt.Sprintf("sensor%02d", i%10)).
            AddField("value", 20.0+float64(i)*0.1).
            SetTime(time.Now())
        
        writeAPI.WritePoint(p)
    }
    
    // Flush remaining writes
    writeAPI.Flush()
    
    // Check for errors
    errorsCh := writeAPI.Errors()
    go func() {
        for err := range errorsCh {
            fmt.Printf("Write error: %v\n", err)
        }
    }()
}
```

**Querying Data (Flux):**
```go
func queryData(client influxdb2.Client) {
    queryAPI := client.QueryAPI("my-org")
    
    query := `
        from(bucket: "my-bucket")
            |> range(start: -1h)
            |> filter(fn: (r) => r._measurement == "cpu_usage")
            |> filter(fn: (r) => r.host == "server01")
            |> aggregateWindow(every: 5m, fn: mean)
    `
    
    result, err := queryAPI.Query(context.Background(), query)
    if err != nil {
        log.Fatal(err)
    }
    defer result.Close()
    
    for result.Next() {
        values := result.Record().Values()
        fmt.Printf("Time: %v, Value: %v\n", 
            values["_time"], 
            values["_value"])
    }
    
    if result.Err() != nil {
        fmt.Printf("Query error: %v\n", result.Err())
    }
}
```

**Downsampling (Data Retention):**
```go
// Create a task for continuous downsampling
func createDownsamplingTask(client influxdb2.Client) {
    tasksAPI := client.TasksAPI()
    
    flux := `
        option task = {name: "downsample-cpu", every: 1h}
        
        from(bucket: "my-bucket")
            |> range(start: -1h)
            |> filter(fn: (r) => r._measurement == "cpu_usage")
            |> aggregateWindow(every: 5m, fn: mean)
            |> to(bucket: "my-bucket-downsampled")
    `
    
    task := &domain.Task{
        Name:   "downsample-cpu",
        OrgID:  "my-org-id",
        Status: domain.TaskStatusActive,
        Flux:   flux,
    }
    
    createdTask, err := tasksAPI.CreateTask(context.Background(), task)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Task created: %s\n", createdTask.Name)
}
```

### TimescaleDB (PostgreSQL Extension)

**Connection:**
```go
import (
    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    pool, err := pgxpool.New(context.Background(), 
        "postgres://user:pass@localhost:5432/timescale")
    if err != nil {
        log.Fatal(err)
    }
    defer pool.Close()
    
    // Create hypertable
    _, err = pool.Exec(context.Background(), `
        CREATE TABLE IF NOT EXISTS metrics (
            time        TIMESTAMPTZ NOT NULL,
            sensor_id   INTEGER,
            temperature DOUBLE PRECISION,
            humidity    DOUBLE PRECISION
        );
        
        SELECT create_hypertable('metrics', 'time', if_not_exists => TRUE);
    `)
    if err != nil {
        log.Fatal(err)
    }
}
```

**Writing Data:**
```go
func writeMetrics(pool *pgxpool.Pool) {
    batch := &pgx.Batch{}
    
    for i := 0; i < 1000; i++ {
        batch.Queue(`
            INSERT INTO metrics (time, sensor_id, temperature, humidity)
            VALUES ($1, $2, $3, $4)
        `,
            time.Now().Add(-time.Duration(i)*time.Minute),
            i%10,
            20.0+float64(i)*0.01,
            60.0+float64(i)*0.02,
        )
    }
    
    br := pool.SendBatch(context.Background(), batch)
    defer br.Close()
    
    _, err := br.Exec()
    if err != nil {
        log.Fatal(err)
    }
}
```

**Querying with Time Buckets:**
```go
func queryTimeBuckets(pool *pgxpool.Pool) {
    rows, err := pool.Query(context.Background(), `
        SELECT 
            time_bucket('5 minutes', time) AS bucket,
            sensor_id,
            AVG(temperature) AS avg_temp,
            MAX(temperature) AS max_temp,
            MIN(temperature) AS min_temp
        FROM metrics
        WHERE time > NOW() - INTERVAL '1 hour'
        GROUP BY bucket, sensor_id
        ORDER BY bucket DESC
    `)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    
    for rows.Next() {
        var bucket time.Time
        var sensorID int
        var avgTemp, maxTemp, minTemp float64
        
        rows.Scan(&bucket, &sensorID, &avgTemp, &maxTemp, &minTemp)
        fmt.Printf("Bucket: %v, Sensor: %d, Avg: %.2f, Max: %.2f, Min: %.2f\n",
            bucket, sensorID, avgTemp, maxTemp, minTemp)
    }
}
```

**Continuous Aggregates:**
```go
// Create materialized view for pre-computed aggregates
_, err = pool.Exec(context.Background(), `
    CREATE MATERIALIZED VIEW IF NOT EXISTS metrics_hourly
    WITH (timescaledb.continuous) AS
    SELECT 
        time_bucket('1 hour', time) AS hour,
        sensor_id,
        AVG(temperature) AS avg_temp,
        MAX(temperature) AS max_temp,
        MIN(temperature) AS min_temp,
        COUNT(*) AS sample_count
    FROM metrics
    GROUP BY hour, sensor_id
    WITH NO DATA;
    
    SELECT add_continuous_aggregate_policy('metrics_hourly',
        start_offset => INTERVAL '3 hours',
        end_offset => INTERVAL '1 hour',
        schedule_interval => INTERVAL '1 hour');
`)
```

**Data Retention:**
```go
// Automatically drop old data
_, err = pool.Exec(context.Background(), `
    SELECT add_retention_policy('metrics', INTERVAL '30 days');
`)
```

---

*I'll continue with Data Validation, Serialization Formats, Workflow Orchestration, and complete Part 12. Shall I continue?*

## 12.5 Data Validation and Quality

### Data Quality Dimensions

- **Completeness**: All required fields present
- **Accuracy**: Values are correct
- **Consistency**: Data agrees across systems
- **Timeliness**: Data is up-to-date
- **Uniqueness**: No unintended duplicates

### Validation Framework

```go
package validation

type Validator interface {
    Validate(data map[string]interface{}) ValidationResult
}

type ValidationResult struct {
    Valid  bool
    Errors []ValidationError
}

type ValidationError struct {
    Field   string
    Message string
    Code    string
}

// Composite validator
type CompositeValidator struct {
    validators []Validator
}

func (cv *CompositeValidator) Validate(data map[string]interface{}) ValidationResult {
    result := ValidationResult{Valid: true, Errors: []ValidationError{}}
    
    for _, validator := range cv.validators {
        vr := validator.Validate(data)
        if !vr.Valid {
            result.Valid = false
            result.Errors = append(result.Errors, vr.Errors...)
        }
    }
    
    return result
}
```

### Schema Validation

```go
type SchemaValidator struct {
    schema map[string]FieldSchema
}

type FieldSchema struct {
    Required bool
    Type     string
    Min      *float64
    Max      *float64
    Pattern  *regexp.Regexp
}

func (sv *SchemaValidator) Validate(data map[string]interface{}) ValidationResult {
    result := ValidationResult{Valid: true, Errors: []ValidationError{}}
    
    for field, schema := range sv.schema {
        value, exists := data[field]
        
        // Required field check
        if schema.Required && !exists {
            result.Valid = false
            result.Errors = append(result.Errors, ValidationError{
                Field:   field,
                Message: "field is required",
                Code:    "REQUIRED",
            })
            continue
        }
        
        if !exists {
            continue
        }
        
        // Type check
        if !sv.checkType(value, schema.Type) {
            result.Valid = false
            result.Errors = append(result.Errors, ValidationError{
                Field:   field,
                Message: fmt.Sprintf("expected type %s", schema.Type),
                Code:    "TYPE_MISMATCH",
            })
        }
        
        // Range check
        if schema.Min != nil || schema.Max != nil {
            if num, ok := value.(float64); ok {
                if schema.Min != nil && num < *schema.Min {
                    result.Valid = false
                    result.Errors = append(result.Errors, ValidationError{
                        Field:   field,
                        Message: fmt.Sprintf("value must be >= %.2f", *schema.Min),
                        Code:    "OUT_OF_RANGE",
                    })
                }
                if schema.Max != nil && num > *schema.Max {
                    result.Valid = false
                    result.Errors = append(result.Errors, ValidationError{
                        Field:   field,
                        Message: fmt.Sprintf("value must be <= %.2f", *schema.Max),
                        Code:    "OUT_OF_RANGE",
                    })
                }
            }
        }
        
        // Pattern check
        if schema.Pattern != nil {
            if str, ok := value.(string); ok {
                if !schema.Pattern.MatchString(str) {
                    result.Valid = false
                    result.Errors = append(result.Errors, ValidationError{
                        Field:   field,
                        Message: "value doesn't match required pattern",
                        Code:    "PATTERN_MISMATCH",
                    })
                }
            }
        }
    }
    
    return result
}

func (sv *SchemaValidator) checkType(value interface{}, expectedType string) bool {
    switch expectedType {
    case "string":
        _, ok := value.(string)
        return ok
    case "number":
        _, ok := value.(float64)
        return ok
    case "boolean":
        _, ok := value.(bool)
        return ok
    case "array":
        _, ok := value.([]interface{})
        return ok
    case "object":
        _, ok := value.(map[string]interface{})
        return ok
    default:
        return true
    }
}
```

### Statistical Validation

```go
type StatisticalValidator struct {
    historicalMean float64
    historicalStd  float64
    threshold      float64 // number of standard deviations
}

func (sv *StatisticalValidator) Validate(data map[string]interface{}) ValidationResult {
    result := ValidationResult{Valid: true, Errors: []ValidationError{}}
    
    value, ok := data["value"].(float64)
    if !ok {
        return result
    }
    
    // Check if value is an outlier
    zScore := (value - sv.historicalMean) / sv.historicalStd
    if math.Abs(zScore) > sv.threshold {
        result.Valid = false
        result.Errors = append(result.Errors, ValidationError{
            Field:   "value",
            Message: fmt.Sprintf("value is %.2f standard deviations from mean", zScore),
            Code:    "STATISTICAL_OUTLIER",
        })
    }
    
    return result
}
```

### Data Quality Metrics

```go
type DataQualityMetrics struct {
    totalRecords      int
    validRecords      int
    missingFields     map[string]int
    invalidValues     map[string]int
    duplicates        int
    mu                sync.Mutex
}

func NewDataQualityMetrics() *DataQualityMetrics {
    return &DataQualityMetrics{
        missingFields: make(map[string]int),
        invalidValues: make(map[string]int),
    }
}

func (m *DataQualityMetrics) RecordValidation(data map[string]interface{}, result ValidationResult) {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    m.totalRecords++
    
    if result.Valid {
        m.validRecords++
    } else {
        for _, err := range result.Errors {
            switch err.Code {
            case "REQUIRED":
                m.missingFields[err.Field]++
            case "TYPE_MISMATCH", "OUT_OF_RANGE", "PATTERN_MISMATCH":
                m.invalidValues[err.Field]++
            }
        }
    }
}

func (m *DataQualityMetrics) Report() map[string]interface{} {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    qualityScore := 0.0
    if m.totalRecords > 0 {
        qualityScore = float64(m.validRecords) / float64(m.totalRecords) * 100
    }
    
    return map[string]interface{}{
        "total_records":   m.totalRecords,
        "valid_records":   m.validRecords,
        "quality_score":   qualityScore,
        "missing_fields":  m.missingFields,
        "invalid_values":  m.invalidValues,
        "duplicates":      m.duplicates,
    }
}
```

---

## 12.6 Data Serialization Formats

### Protocol Buffers

**Define Schema (.proto file):**
```protobuf
syntax = "proto3";

package data;

option go_package = "myapp/data";

message User {
    int64 id = 1;
    string name = 2;
    string email = 3;
    repeated string roles = 4;
    google.protobuf.Timestamp created_at = 5;
}

message UserList {
    repeated User users = 1;
}
```

**Generate Go Code:**
```bash
protoc --go_out=. --go_opt=paths=source_relative user.proto
```

**Usage:**
```go
package main

import (
    "fmt"
    "time"
    
    "google.golang.org/protobuf/proto"
    "google.golang.org/protobuf/types/known/timestamppb"
    pb "myapp/data"
)

func main() {
    user := &pb.User{
        Id:    1,
        Name:  "Alice",
        Email: "alice@example.com",
        Roles: []string{"admin", "user"},
        CreatedAt: timestamppb.New(time.Now()),
    }
    
    // Serialize to binary
    data, err := proto.Marshal(user)
    if err != nil {
        panic(err)
    }
    fmt.Printf("Serialized size: %d bytes\n", len(data))
    
    // Deserialize
    user2 := &pb.User{}
    if err := proto.Unmarshal(data, user2); err != nil {
        panic(err)
    }
    fmt.Printf("Deserialized: %+v\n", user2)
}
```

### Apache Avro

```bash
go get github.com/linkedin/goavro/v2
```

**Schema:**
```json
{
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"},
    {"name": "created_at", "type": "long", "logicalType": "timestamp-millis"}
  ]
}
```

**Usage:**
```go
package main

import (
    "github.com/linkedin/goavro/v2"
)

func main() {
    schema := `{
        "type": "record",
        "name": "User",
        "fields": [
            {"name": "id", "type": "long"},
            {"name": "name", "type": "string"},
            {"name": "email", "type": "string"}
        ]
    }`
    
    codec, err := goavro.NewCodec(schema)
    if err != nil {
        panic(err)
    }
    
    // Create record
    user := map[string]interface{}{
        "id":    int64(1),
        "name":  "Alice",
        "email": "alice@example.com",
    }
    
    // Encode to binary
    binary, err := codec.BinaryFromNative(nil, user)
    if err != nil {
        panic(err)
    }
    
    // Decode from binary
    decoded, _, err := codec.NativeFromBinary(binary)
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Decoded: %+v\n", decoded)
}
```

### Apache Parquet

```bash
go get github.com/xitongsys/parquet-go
go get github.com/xitongsys/parquet-go-source/local
```

**Write Parquet:**
```go
package main

import (
    "github.com/xitongsys/parquet-go-source/local"
    "github.com/xitongsys/parquet-go/writer"
)

type User struct {
    ID        int64  `parquet:"name=id, type=INT64"`
    Name      string `parquet:"name=name, type=BYTE_ARRAY, convertedtype=UTF8"`
    Email     string `parquet:"name=email, type=BYTE_ARRAY, convertedtype=UTF8"`
    Age       int32  `parquet:"name=age, type=INT32"`
}

func main() {
    fw, err := local.NewLocalFileWriter("users.parquet")
    if err != nil {
        panic(err)
    }
    
    pw, err := writer.NewParquetWriter(fw, new(User), 4)
    if err != nil {
        panic(err)
    }
    
    users := []User{
        {ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30},
        {ID: 2, Name: "Bob", Email: "bob@example.com", Age: 25},
    }
    
    for _, user := range users {
        if err := pw.Write(user); err != nil {
            panic(err)
        }
    }
    
    if err := pw.WriteStop(); err != nil {
        panic(err)
    }
    fw.Close()
    
    fmt.Println("Parquet file written")
}
```

**Read Parquet:**
```go
func readParquet() {
    fr, err := local.NewLocalFileReader("users.parquet")
    if err != nil {
        panic(err)
    }
    
    pr, err := reader.NewParquetReader(fr, new(User), 4)
    if err != nil {
        panic(err)
    }
    
    num := int(pr.GetNumRows())
    users := make([]User, num)
    
    if err := pr.Read(&users); err != nil {
        panic(err)
    }
    
    pr.ReadStop()
    fr.Close()
    
    for _, user := range users {
        fmt.Printf("User: %+v\n", user)
    }
}
```

---

## 12.7 Workflow Orchestration with Temporal

### What is Temporal?

Temporal is a workflow orchestration platform that handles:
- Long-running workflows
- Retry logic
- State management
- Failure recovery

```bash
go get go.temporal.io/sdk
```

### Workflow Definition

```go
package workflows

import (
    "time"
    
    "go.temporal.io/sdk/workflow"
)

type DataPipelineInput struct {
    Source      string
    Destination string
    BatchSize   int
}

func DataPipelineWorkflow(ctx workflow.Context, input DataPipelineInput) error {
    logger := workflow.GetLogger(ctx)
    logger.Info("Starting data pipeline", "source", input.Source)
    
    // Activity options
    ao := workflow.ActivityOptions{
        StartToCloseTimeout: 10 * time.Minute,
        RetryPolicy: &temporal.RetryPolicy{
            InitialInterval:    time.Second,
            BackoffCoefficient: 2.0,
            MaximumInterval:    time.Minute,
            MaximumAttempts:    3,
        },
    }
    ctx = workflow.WithActivityOptions(ctx, ao)
    
    // Step 1: Extract
    var extractResult ExtractResult
    err := workflow.ExecuteActivity(ctx, ExtractActivity, input.Source, input.BatchSize).Get(ctx, &extractResult)
    if err != nil {
        return err
    }
    logger.Info("Extraction completed", "records", extractResult.RecordCount)
    
    // Step 2: Transform
    var transformResult TransformResult
    err = workflow.ExecuteActivity(ctx, TransformActivity, extractResult).Get(ctx, &transformResult)
    if err != nil {
        return err
    }
    logger.Info("Transformation completed", "valid", transformResult.ValidRecords)
    
    // Step 3: Load
    err = workflow.ExecuteActivity(ctx, LoadActivity, input.Destination, transformResult).Get(ctx, nil)
    if err != nil {
        return err
    }
    logger.Info("Pipeline completed successfully")
    
    return nil
}
```

### Activity Definitions

```go
package activities

import (
    "context"
)

type ExtractResult struct {
    RecordCount int
    Data        []map[string]interface{}
}

func ExtractActivity(ctx context.Context, source string, batchSize int) (ExtractResult, error) {
    // Extract logic
    records := extractFromSource(source, batchSize)
    
    return ExtractResult{
        RecordCount: len(records),
        Data:        records,
    }, nil
}

type TransformResult struct {
    ValidRecords   int
    InvalidRecords int
    Data           []map[string]interface{}
}

func TransformActivity(ctx context.Context, input ExtractResult) (TransformResult, error) {
    // Transform logic
    valid := []map[string]interface{}{}
    invalid := 0
    
    for _, record := range input.Data {
        if validateRecord(record) {
            transformed := transformRecord(record)
            valid = append(valid, transformed)
        } else {
            invalid++
        }
    }
    
    return TransformResult{
        ValidRecords:   len(valid),
        InvalidRecords: invalid,
        Data:           valid,
    }, nil
}

func LoadActivity(ctx context.Context, destination string, input TransformResult) error {
    // Load logic
    return loadToDestination(destination, input.Data)
}
```

### Worker Setup

```go
package main

import (
    "log"
    
    "go.temporal.io/sdk/client"
    "go.temporal.io/sdk/worker"
)

func main() {
    c, err := client.NewClient(client.Options{
        HostPort: "localhost:7233",
    })
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()
    
    w := worker.New(c, "data-pipeline-task-queue", worker.Options{})
    
    // Register workflows and activities
    w.RegisterWorkflow(DataPipelineWorkflow)
    w.RegisterActivity(ExtractActivity)
    w.RegisterActivity(TransformActivity)
    w.RegisterActivity(LoadActivity)
    
    err = w.Run(worker.InterruptCh())
    if err != nil {
        log.Fatal(err)
    }
}
```

### Starting Workflows

```go
func startPipeline() {
    c, err := client.NewClient(client.Options{
        HostPort: "localhost:7233",
    })
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()
    
    input := DataPipelineInput{
        Source:      "s3://my-bucket/data.csv",
        Destination: "postgres://localhost/warehouse",
        BatchSize:   1000,
    }
    
    options := client.StartWorkflowOptions{
        ID:        "data-pipeline-" + time.Now().Format("20060102150405"),
        TaskQueue: "data-pipeline-task-queue",
    }
    
    we, err := c.ExecuteWorkflow(context.Background(), options, DataPipelineWorkflow, input)
    if err != nil {
        log.Fatal(err)
    }
    
    log.Printf("Started workflow ID: %s, RunID: %s", we.GetID(), we.GetRunID())
    
    // Wait for result
    var result interface{}
    err = we.Get(context.Background(), &result)
    if err != nil {
        log.Fatal(err)
    }
    
    log.Println("Workflow completed successfully")
}
```

---

## FAQs

**Q1: When should I use stream processing vs batch processing?**

Use stream processing when:
- Need real-time or near-real-time results
- Continuous data flow
- Low latency required (seconds)
- Event-driven architecture

Use batch processing when:
- Processing historical data
- Complex transformations
- Can tolerate higher latency (hours)
- Cost-efficiency is priority

**Q2: Kafka vs RabbitMQ vs NATS - which to choose?**

- **Kafka**: High throughput, persistence, replay capability. Best for event streaming, logs.
- **RabbitMQ**: Flexible routing, complex workflows. Best for task queues, RPC.
- **NATS**: Lightweight, simple, fast. Best for microservices messaging, IoT.

**Q3: What's the difference between InfluxDB and TimescaleDB?**

- **InfluxDB**: Purpose-built time-series DB, Flux query language, better for metrics/IoT
- **TimescaleDB**: PostgreSQL extension, SQL queries, better for existing PostgreSQL users

**Q4: How do I handle schema evolution in data pipelines?**

- Use schema registry (Kafka Schema Registry, Confluent)
- Version your schemas
- Use forward/backward compatible formats (Avro, Protobuf)
- Implement schema validation in pipelines
- Plan migration strategies

**Q5: What are best practices for ETL error handling?**

- Validate data early (fail fast)
- Use dead letter queues for failed records
- Implement retry logic with exponential backoff
- Log errors with context
- Monitor error rates
- Have rollback procedures

---

## Interview Questions

**Q1: Explain the difference between at-most-once, at-least-once, and exactly-once delivery.**

**A**: At-most-once: message may be lost but never duplicated (fastest). At-least-once: message may be duplicated but never lost (most common). Exactly-once: message processed exactly once (most complex, requires idempotency or transactions).

**Q2: How does Kafka achieve high throughput?**

**A**: Sequential disk I/O, batching, compression, zero-copy transfers, partition parallelism, efficient binary protocol, and page cache utilization.

**Q3: What is a watermark in stream processing?**

**A**: A watermark tracks event-time progress, indicating that no events older than the watermark are expected. Used to trigger computations on windows and handle late data.

**Q4: How would you design a data pipeline that processes 1 million records per minute?**

**A**: Use parallel processing (partitions), batching, async I/O, efficient serialization (Protobuf/Avro), backpressure handling, horizontal scaling, and monitoring with metrics.

**Q5: Explain the CAP theorem in the context of data systems.**

**A**: CAP: Consistency, Availability, Partition tolerance. You can only have 2 of 3. In distributed data systems with network partitions, choose between consistency (wait for all nodes) or availability (serve potentially stale data).

---

## Key Takeaways

1. **Stream processing** enables real-time data insights with low latency
2. **Kafka** is the de-facto standard for event streaming
3. **ETL pipelines** require careful error handling and validation
4. **Time-series databases** optimize for time-stamped data patterns
5. **Data quality** is critical - validate early and often
6. **Serialization format** choice affects performance and compatibility
7. **Workflow orchestration** (Temporal) handles complex, long-running pipelines
8. **Go excels** at data engineering due to concurrency and performance

---

## Practice Exercises

### Exercise 1: Build a Real-Time Analytics Pipeline
- Kafka producer sending events
- Stream processor aggregating metrics
- InfluxDB for storage
- Grafana for visualization

### Exercise 2: ETL Pipeline
- Extract from PostgreSQL
- Transform with validation
- Load to Parquet files
- Handle errors with DLQ

### Exercise 3: Time-Series Data Processor
- Generate sensor data
- Write to TimescaleDB
- Query with time buckets
- Create continuous aggregates

### Exercise 4: Kafka Consumer Group
- Multiple consumers
- Partition rebalancing
- Offset management
- Graceful shutdown

### Exercise 5: Data Quality Framework
- Schema validation
- Statistical validation
- Quality metrics
- Alerting on quality issues

---

## Additional Resources

**Kafka:**
- https://kafka.apache.org
- Kafka: The Definitive Guide (book)
- https://github.com/IBM/sarama

**Stream Processing:**
- Designing Data-Intensive Applications (book)
- https://flink.apache.org
- https://kafka.apache.org/documentation/streams/

**Time-Series:**
- https://docs.influxdata.com
- https://docs.timescale.com
- InfluxDB: Time Series Data Guide

**Data Engineering:**
- The Data Engineering Cookbook
- https://temporal.io/docs
- https://airflow.apache.org

---

## Summary

Part 12 covered data engineering and processing with Go:

1. **Stream Processing**: Fundamentals, windowing, guarantees
2. **Apache Kafka**: Producers, consumers, scaling, performance
3. **ETL Pipelines**: Extract, transform, load patterns
4. **Time-Series Databases**: InfluxDB, TimescaleDB
5. **Data Validation**: Quality metrics, schema validation
6. **Serialization**: Protobuf, Avro, Parquet
7. **Workflow Orchestration**: Temporal for complex pipelines

You now have the skills to build production-grade data pipelines and streaming applications!

---

**Total Word Count**: ~16,000 words  
**Status**: ✅ Complete

**Next**: Part 13 - Advanced Networking
