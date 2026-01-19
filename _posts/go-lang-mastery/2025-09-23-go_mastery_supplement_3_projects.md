---
title: "Complete Go Mastery - 10 Practice Projects"
date: 2025-01-11 07:00:00 +0530
categories: [Programming, Go, Projects]
tags: [golang, go, projects, practice, portfolio]
---

# Complete Go Mastery - 10 Practice Projects

## Project Difficulty Levels
- 🟢 **Beginner** (Weeks 1-2)
- 🟡 **Intermediate** (Weeks 3-5)
- 🔴 **Advanced** (Weeks 6-8)

---

## Project 1: CLI Task Manager 🟢

### Overview
Build a command-line task management application with file persistence.

### Learning Objectives
- File I/O operations
- Struct and method design
- Command-line argument parsing
- Error handling
- JSON encoding/decoding

### Requirements

**Core Features**:
- Add tasks with title, description, priority
- List all tasks (with filtering)
- Mark tasks as complete
- Delete tasks
- Save/load from JSON file

**Technical Requirements**:
- Use `flag` or `cobra` package for CLI
- Struct-based task representation
- Persistence in JSON format
- Error handling for all operations
- Unit tests for core functions

### Example Usage
```bash
$ taskman add "Buy groceries" --priority high
Task added: #1

$ taskman list
#1 [HIGH] Buy groceries (pending)

$ taskman complete 1
Task #1 marked complete

$ taskman list --status pending
No pending tasks
```

### Implementation Hints
```go
type Task struct {
    ID          int       `json:"id"`
    Title       string    `json:"title"`
    Description string    `json:"description"`
    Priority    string    `json:"priority"`
    Status      string    `json:"status"`
    CreatedAt   time.Time `json:"created_at"`
}

type TaskManager struct {
    tasks []Task
    file  string
}

func (tm *TaskManager) Add(task Task) error
func (tm *TaskManager) List(filter string) []Task
func (tm *TaskManager) Complete(id int) error
func (tm *TaskManager) Delete(id int) error
func (tm *TaskManager) Save() error
func (tm *TaskManager) Load() error
```

### Extensions
- Due dates and reminders
- Task categories/tags
- Search functionality
- Statistics (completion rate)
- Export to CSV

### Success Criteria
- [ ] All CRUD operations work
- [ ] Data persists between runs
- [ ] 80%+ test coverage
- [ ] Clean, documented code
- [ ] Error handling for edge cases

---

## Project 2: Concurrent Web Scraper 🟡

### Overview
Build a web scraper that fetches multiple pages concurrently and extracts structured data.

### Learning Objectives
- Goroutines and channels
- HTTP client usage
- HTML parsing
- Worker pool pattern
- Rate limiting

### Requirements

**Core Features**:
- Scrape multiple URLs concurrently
- Extract specific data (links, titles, metadata)
- Rate limiting to respect servers
- Save results to JSON/CSV
- Progress tracking

**Technical Requirements**:
- Worker pool (limit concurrent requests)
- Context for cancellation
- Proper error handling
- Respect robots.txt
- User-agent header

### Example Usage
```bash
$ scraper --urls urls.txt --workers 5 --output results.json
Scraping 100 URLs with 5 workers...
Progress: [=========>         ] 45/100 (45%)
Completed in 23.5s
Results saved to results.json
```

### Implementation Hints
```go
type Scraper struct {
    client  *http.Client
    workers int
    limiter *rate.Limiter
}

type Result struct {
    URL       string
    Title     string
    Links     []string
    Error     string
    Timestamp time.Time
}

func (s *Scraper) Scrape(urls []string) []Result
func (s *Scraper) worker(jobs <-chan string, results chan<- Result)
func extractData(doc *html.Node) (*Result, error)
```

### Extensions
- Recursive crawling (follow links)
- Duplicate URL detection
- Image downloading
- JavaScript rendering (with headless browser)
- Sitemap generation

### Success Criteria
- [ ] Scrapes 100+ URLs successfully
- [ ] Handles network errors gracefully
- [ ] Respects rate limits
- [ ] No goroutine leaks
- [ ] Progress indication

---

## Project 3: RESTful Blog API 🟡

### Overview
Create a complete REST API for a blog with authentication and CRUD operations.

### Learning Objectives
- HTTP server and routing
- Database integration (PostgreSQL)
- Authentication (JWT)
- Middleware patterns
- API design

### Requirements

**Endpoints**:
```
POST   /auth/register
POST   /auth/login
GET    /posts
POST   /posts
GET    /posts/:id
PUT    /posts/:id
DELETE /posts/:id
POST   /posts/:id/comments
GET    /users/:id/posts
```

**Features**:
- User registration and login
- JWT-based authentication
- CRUD for blog posts
- Comments on posts
- User profiles
- Pagination
- Input validation

**Technical Requirements**:
- PostgreSQL database
- JWT for auth
- Middleware (logging, auth, CORS)
- Request validation
- Error handling
- API documentation

### Database Schema
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    title VARCHAR(200) NOT NULL,
    content TEXT,
    published BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER REFERENCES posts(id),
    user_id INTEGER REFERENCES users(id),
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Implementation Hints
```go
type Server struct {
    router *chi.Mux
    db     *sql.DB
    jwt    *JWTService
}

func (s *Server) registerRoutes()
func (s *Server) handleLogin(w http.ResponseWriter, r *http.Request)
func (s *Server) handleCreatePost(w http.ResponseWriter, r *http.Request)
func authMiddleware(next http.Handler) http.Handler
```

### Extensions
- File uploads for images
- Rich text formatting
- Tags and categories
- Search functionality
- Social features (likes, shares)

### Success Criteria
- [ ] All endpoints working
- [ ] Secure authentication
- [ ] 90%+ test coverage
- [ ] Proper error responses
- [ ] API documentation

---

## Project 4: Real-Time Chat Server 🟡

### Overview
Build a WebSocket-based chat server supporting multiple rooms and private messages.

### Learning Objectives
- WebSocket protocol
- Concurrent client management
- Broadcast patterns
- Message queuing
- Redis pub/sub

### Requirements

**Core Features**:
- Multiple chat rooms
- Private messaging
- User presence (online/offline)
- Message history
- User authentication

**Technical Requirements**:
- WebSocket connections
- Gorilla WebSocket library
- Redis for pub/sub and history
- Concurrent client management
- Graceful shutdown

### Implementation Hints
```go
type ChatServer struct {
    clients   map[*Client]bool
    rooms     map[string]*Room
    broadcast chan *Message
    register  chan *Client
    unregister chan *Client
}

type Client struct {
    conn *websocket.Conn
    send chan []byte
    user *User
    room *Room
}

type Message struct {
    Type    string    `json:"type"`
    From    string    `json:"from"`
    To      string    `json:"to,omitempty"`
    Room    string    `json:"room,omitempty"`
    Content string    `json:"content"`
    Time    time.Time `json:"time"`
}

func (s *ChatServer) Run()
func (c *Client) readPump()
func (c *Client) writePump()
```

### Extensions
- File sharing
- Typing indicators
- Read receipts
- Message reactions
- User roles (admin, moderator)

### Success Criteria
- [ ] 100+ concurrent users supported
- [ ] No message loss
- [ ] Proper connection cleanup
- [ ] Redis integration
- [ ] Load tested

---

## Project 5: Distributed Task Queue 🔴

### Overview
Build a distributed task processing system with worker pools and retry logic.

### Learning Objectives
- Distributed systems concepts
- Message queues (RabbitMQ/Redis)
- Worker pattern at scale
- Failure handling
- Monitoring

### Requirements

**Components**:
- Producer API (submit jobs)
- Queue (RabbitMQ or Redis)
- Worker pool (process jobs)
- Results storage
- Admin dashboard

**Features**:
- Job submission via API
- Priority queues
- Retry with exponential backoff
- Dead letter queue
- Job status tracking
- Worker auto-scaling

### Architecture
```
[API] ---> [Queue] ---> [Workers] ---> [Results DB]
                            |
                            +-----> [Monitoring]
```

### Implementation Hints
```go
type Job struct {
    ID        string
    Type      string
    Payload   json.RawMessage
    Priority  int
    Retries   int
    Status    string
    CreatedAt time.Time
}

type Queue interface {
    Enqueue(job *Job) error
    Dequeue() (*Job, error)
    Ack(jobID string) error
    Nack(jobID string) error
}

type Worker struct {
    id       int
    queue    Queue
    handlers map[string]Handler
}

func (w *Worker) Start(ctx context.Context)
func (w *Worker) process(job *Job) error
```

### Extensions
- Scheduled jobs (cron-like)
- Job dependencies
- Parallel job execution
- Job cancellation
- Webhook notifications

### Success Criteria
- [ ] Processes 1000+ jobs/min
- [ ] Zero job loss
- [ ] Automatic retry working
- [ ] Monitoring dashboard
- [ ] Horizontally scalable

---

## Project 6: Metrics & Monitoring System 🔴

### Overview
Build a time-series metrics collection and visualization system.

### Learning Objectives
- Time-series data
- Prometheus integration
- Grafana dashboards
- Alert rules
- Data retention

### Requirements

**Components**:
- Metrics collector (Prometheus)
- Time-series database
- Query API
- Visualization dashboard
- Alerting system

**Features**:
- Custom metrics ingestion
- Historical data queries
- Real-time dashboards
- Alert rules and notifications
- Data aggregation

### Implementation Hints
```go
type MetricsCollector struct {
    registry *prometheus.Registry
    server   *http.Server
}

type Metric struct {
    Name      string
    Type      string // counter, gauge, histogram
    Value     float64
    Labels    map[string]string
    Timestamp time.Time
}

func (mc *MetricsCollector) RecordMetric(m Metric)
func (mc *MetricsCollector) Query(query string) ([]Sample, error)
func (mc *MetricsCollector) SetupAlerts(rules []AlertRule)
```

### Extensions
- Multi-tenant support
- Custom visualization
- Anomaly detection
- Forecasting
- Integration with third-party systems

### Success Criteria
- [ ] Handles 10k+ metrics/sec
- [ ] Sub-second query response
- [ ] 30-day data retention
- [ ] Real-time dashboards
- [ ] Alert delivery <1min

---

## Project 7: File Storage Service 🔴

### Overview
Build an S3-compatible object storage service with chunking and deduplication.

### Learning Objectives
- Object storage concepts
- Large file handling
- Content-addressable storage
- Multipart uploads
- CDN integration

### Requirements

**Core Features**:
- Upload/download files
- Chunked uploads for large files
- Content deduplication
- Access control (presigned URLs)
- Metadata storage

**API**:
```
PUT    /buckets/:bucket/objects/:key
GET    /buckets/:bucket/objects/:key
DELETE /buckets/:bucket/objects/:key
POST   /buckets/:bucket/objects/:key/multipart
HEAD   /buckets/:bucket/objects/:key
```

### Implementation Hints
```go
type StorageService struct {
    storage   Storage
    metadata  MetadataStore
    chunker   *Chunker
}

type Object struct {
    Key         string
    Bucket      string
    Size        int64
    ContentType string
    Chunks      []string
    Metadata    map[string]string
    CreatedAt   time.Time
}

func (s *StorageService) PutObject(bucket, key string, data io.Reader) error
func (s *StorageService) GetObject(bucket, key string) (io.ReadCloser, error)
func (s *StorageService) chunkFile(data io.Reader) ([]Chunk, error)
```

### Extensions
- Versioning
- Encryption at rest
- Lifecycle policies
- Replication
- Event notifications

### Success Criteria
- [ ] Handles files up to 5GB
- [ ] Deduplication working
- [ ] <100ms for small files
- [ ] Concurrent uploads
- [ ] 99.99% availability

---

## Project 8: API Gateway 🔴

### Overview
Build a production-grade API gateway with routing, rate limiting, and authentication.

### Learning Objectives
- Reverse proxy
- Load balancing
- Circuit breaker
- Service discovery
- API composition

### Requirements

**Features**:
- Dynamic routing
- Rate limiting per client
- JWT authentication
- Request/response transformation
- Circuit breaker
- Caching
- Metrics and logging

**Configuration**:
```yaml
routes:
  - path: /users/*
    target: http://user-service
    methods: [GET, POST]
    rateLimit: 100/minute
    
  - path: /orders/*
    target: http://order-service
    auth: required
    timeout: 30s
```

### Implementation Hints
```go
type Gateway struct {
    routes       []Route
    rateLimiter  *RateLimiter
    circuitBreaker *CircuitBreaker
    cache        Cache
}

type Route struct {
    Path        string
    Target      string
    Methods     []string
    RateLimit   int
    Auth        bool
    Timeout     time.Duration
}

func (g *Gateway) ServeHTTP(w http.ResponseWriter, r *http.Request)
func (g *Gateway) route(r *http.Request) (*Route, error)
func (g *Gateway) proxy(route *Route, r *http.Request) (*http.Response, error)
```

### Extensions
- GraphQL support
- WebSocket proxying
- Request validation
- Response caching
- Health checks

### Success Criteria
- [ ] Handles 10k req/sec
- [ ] <10ms overhead
- [ ] Dynamic configuration
- [ ] Zero downtime updates
- [ ] Comprehensive metrics

---

## Project 9: Microservices E-Commerce 🔴

### Overview
Build a complete microservices-based e-commerce system.

### Learning Objectives
- Microservices architecture
- Inter-service communication
- Event-driven architecture
- Saga pattern
- Service mesh

### Services

**User Service**:
- Registration and authentication
- Profile management
- JWT token generation

**Product Service**:
- Product catalog
- Inventory management
- Search and filtering

**Order Service**:
- Order creation
- Order status tracking
- Order history

**Payment Service**:
- Payment processing
- Refunds
- Payment history

**Notification Service**:
- Email notifications
- SMS notifications
- Push notifications

### Architecture
```
[API Gateway]
     |
     +---> [User Service] ---> [PostgreSQL]
     +---> [Product Service] ---> [PostgreSQL]
     +---> [Order Service] ---> [PostgreSQL]
     +---> [Payment Service] ---> [PostgreSQL]
     +---> [Notification Service] ---> [Redis Queue]
     
[RabbitMQ] <--- All Services (Events)
```

### Implementation Hints
```go
// Event-driven communication
type EventBus interface {
    Publish(event Event) error
    Subscribe(eventType string, handler EventHandler) error
}

type Event struct {
    Type      string
    Aggregate string
    Data      json.RawMessage
    Timestamp time.Time
}

// Saga pattern for distributed transactions
type Saga struct {
    steps []SagaStep
}

func (s *Saga) Execute(ctx context.Context) error
func (s *Saga) Compensate(ctx context.Context) error
```

### Extensions
- Shopping cart service
- Recommendation engine
- Review and rating system
- Analytics service
- Admin dashboard

### Success Criteria
- [ ] All services deployed
- [ ] Event-driven communication
- [ ] Distributed tracing
- [ ] Service mesh (Istio/Linkerd)
- [ ] Full CI/CD pipeline
- [ ] Kubernetes deployment

---

## Project 10: Performance Monitoring Platform 🔴

### Overview
Build a comprehensive APM (Application Performance Monitoring) platform.

### Learning Objectives
- Distributed tracing
- Performance profiling
- Real-time analytics
- Alerting and notifications
- Data visualization

### Requirements

**Core Features**:
- Trace collection and storage
- Performance metrics
- Error tracking
- Custom instrumentation
- Real-time dashboards
- Alert management

**Components**:
- Collector (receives traces and metrics)
- Storage (time-series + traces)
- Query API
- Dashboard UI
- Alert manager

### Implementation Hints
```go
type Trace struct {
    TraceID   string
    Spans     []Span
    Duration  time.Duration
    StartTime time.Time
}

type Span struct {
    SpanID    string
    ParentID  string
    Operation string
    Service   string
    Duration  time.Duration
    Tags      map[string]string
    Logs      []Log
}

type Collector struct {
    storage TraceStorage
    metrics MetricsStore
}

func (c *Collector) CollectTrace(trace Trace) error
func (c *Collector) CollectMetric(metric Metric) error
func (c *Collector) Query(filter TraceFilter) ([]Trace, error)
```

### Extensions
- Machine learning for anomaly detection
- Automatic performance insights
- Cost optimization recommendations
- Capacity planning
- Custom SLIs/SLOs

### Success Criteria
- [ ] Ingests 100k traces/sec
- [ ] Sub-second query latency
- [ ] 90-day data retention
- [ ] Real-time dashboards
- [ ] Integrates with popular frameworks

---

## General Project Guidelines

### Code Quality Standards
- **Tests**: Minimum 80% coverage
- **Documentation**: README with setup instructions
- **Code Style**: gofmt and golangci-lint passing
- **Git**: Clean commit history with meaningful messages

### Performance Requirements
- **Response Time**: p99 < 100ms for APIs
- **Throughput**: Handle expected load + 50%
- **Resource Usage**: Monitor CPU and memory
- **Scalability**: Horizontal scaling capability

### Security Requirements
- **Input Validation**: All user inputs validated
- **Authentication**: Secure auth where needed
- **Secrets**: No hardcoded credentials
- **Dependencies**: Regular security scans

### Deployment Requirements
- **Docker**: Containerized application
- **CI/CD**: Automated testing and deployment
- **Monitoring**: Metrics and logging
- **Documentation**: Deployment guide

---

## Project Submission Checklist

For each project:
- [ ] Code pushed to GitHub
- [ ] README with clear instructions
- [ ] Tests passing (with coverage report)
- [ ] Docker/docker-compose setup
- [ ] Environment variable configuration
- [ ] API documentation (if applicable)
- [ ] Architecture diagram
- [ ] Demo video or screenshots
- [ ] Issues/limitations documented

---

## Portfolio Presentation

### GitHub Profile
- Pin your best 5-6 projects
- Include comprehensive READMEs
- Add topics/tags for discoverability
- Maintain green contribution graph

### Project README Template
```markdown
# Project Name

Brief description (1-2 sentences)

## Features
- Feature 1
- Feature 2
- Feature 3

## Tech Stack
- Go 1.24
- PostgreSQL
- Redis
- Docker

## Quick Start
```bash
docker-compose up
```

## Architecture
[Diagram or description]

## API Documentation
[Link to docs]

## Tests
```bash
go test -v ./...
```

## Performance
- Handles X requests/second
- p99 latency: Xms
- Test methodology

## Future Improvements
- Improvement 1
- Improvement 2

## License
MIT
```

---

**Start with Project 1 and progress through the list. Each project builds on skills from previous ones. Good luck! 🚀**
