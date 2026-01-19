---
title: "Go Mastery - Part 6: Observability, Monitoring & Debugging"
date: 2025-09-08 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, Programming, Observability, Monitoring, Logging, Metrics, Tracing, Debugging, Profiling, Alerting, Observability]
---

# Complete Go Mastery Part 6: Observability, Monitoring & Debugging

## Introduction

Welcome to Part 6 of the Complete Go Mastery series. In Parts 1-5, we covered Go fundamentals, concurrency, advanced techniques, testing, and architecture patterns. Now we dive into **observability**—the practice of understanding what's happening inside your running systems.

Production systems fail. They experience bugs, performance issues, resource exhaustion, and unexpected behaviors. Without proper observability, debugging these issues is like searching for a needle in a haystack while blindfolded. With good observability, you can quickly identify, diagnose, and fix problems before they impact users.

**Why is observability critical?**

- **Rapid incident response**: Quickly identify and fix issues
- **Performance optimization**: Find bottlenecks and inefficiencies
- **Capacity planning**: Understand resource usage and growth
- **User experience**: Monitor and improve application performance
- **Cost optimization**: Identify waste and optimize resource usage

**What you'll learn in Part 6:**

- **Three Pillars of Observability**: Logs, Metrics, Traces
- **Structured Logging**: Best practices with popular libraries
- **Metrics Collection**: Prometheus, custom metrics, dashboards
- **Distributed Tracing**: OpenTelemetry, request flows
- **Application Monitoring**: Health checks, readiness, liveness
- **Error Tracking**: Sentry, error aggregation, alerting
- **Performance Monitoring**: APM, resource tracking
- **Debugging Techniques**: Tools, strategies, common issues
- **Production Debugging**: Live debugging, incident response
- **Alerting Strategies**: What to alert on, alert fatigue

By the end of Part 6, you'll be able to build observable Go applications that are easy to monitor, debug, and maintain in production.

---

## 6.1 The Three Pillars of Observability

### Understanding Observability vs Monitoring

**Monitoring** answers: "Is something wrong?"
**Observability** answers: "What is wrong and why?"

```
┌─────────────────────────────────────────┐
│        Three Pillars of               │
│         Observability                 │
│                                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  │  LOGS    │  │ METRICS  │  │  TRACES  │
│  │          │  │          │  │          │
│  │ Events & │  │ Numbers  │  │ Request  │
│  │ Context  │  │ over     │  │ Flows    │
│  │          │  │ Time     │  │          │
│  └──────────┘  └──────────┘  └──────────┘
│                                       │
│         ↓                             │
│                                       │
│  ┌─────────────────────────────────┐ │
│  │    Unified Observability        │ │
│  │    Platform                     │ │
│  └─────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### When to Use Each Pillar

**Logs** - Discrete events
- Error occurred
- User logged in
- Request processed
- Configuration changed

**Metrics** - Aggregated data
- Request rate
- Error rate
- CPU usage
- Memory consumption

**Traces** - Request journeys
- Request flow through services
- Performance bottlenecks
- Service dependencies
- Latency breakdown

### The Observability Stack

```go
// pkg/observability/observability.go
package observability

import (
    "context"
    "io"
    
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/metric"
    "go.opentelemetry.io/otel/trace"
    "go.uber.org/zap"
)

// Observer provides unified observability interface
type Observer struct {
    logger  *zap.Logger
    tracer  trace.Tracer
    meter   metric.Meter
}

func New(serviceName, environment string) (*Observer, error) {
    // Initialize logger
    logger, err := initLogger(environment)
    if err != nil {
        return nil, err
    }
    
    // Initialize tracer
    tracer, err := initTracer(serviceName)
    if err != nil {
        return nil, err
    }
    
    // Initialize meter
    meter, err := initMeter(serviceName)
    if err != nil {
        return nil, err
    }
    
    return &Observer{
        logger: logger,
        tracer: tracer,
        meter:  meter,
    }, nil
}

func (o *Observer) Logger() *zap.Logger {
    return o.logger
}

func (o *Observer) Tracer() trace.Tracer {
    return o.tracer
}

func (o *Observer) Meter() metric.Meter {
    return o.meter
}

// Close shuts down all observability components
func (o *Observer) Close() error {
    return o.logger.Sync()
}
```

---

## 6.2 Structured Logging

### Why Structured Logging?

```go
// ❌ BAD: Unstructured logging
log.Printf("User %s logged in from %s at %s", userID, ipAddress, time.Now())

// ✅ GOOD: Structured logging
logger.Info("user logged in",
    zap.String("user_id", userID),
    zap.String("ip_address", ipAddress),
    zap.Time("timestamp", time.Now()),
)
```

**Benefits:**
- **Searchable**: Query specific fields
- **Parsable**: Machine-readable format
- **Contextual**: Rich metadata
- **Aggregatable**: Group and analyze

### Using Zap for High-Performance Logging

```go
// pkg/logger/logger.go
package logger

import (
    "context"
    "os"
    
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

type Logger struct {
    *zap.Logger
}

// New creates a new logger
func New(environment string, opts ...Option) (*Logger, error) {
    var config zap.Config
    
    if environment == "production" {
        config = zap.NewProductionConfig()
        config.EncoderConfig.TimeKey = "timestamp"
        config.EncoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
    } else {
        config = zap.NewDevelopmentConfig()
        config.EncoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder
    }
    
    // Apply options
    for _, opt := range opts {
        opt(&config)
    }
    
    logger, err := config.Build(
        zap.AddCallerSkip(1), // Skip wrapper functions
        zap.AddStacktrace(zapcore.ErrorLevel),
    )
    if err != nil {
        return nil, err
    }
    
    return &Logger{Logger: logger}, nil
}

type Option func(*zap.Config)

// WithLevel sets the logging level
func WithLevel(level string) Option {
    return func(config *zap.Config) {
        var zapLevel zapcore.Level
        zapLevel.UnmarshalText([]byte(level))
        config.Level = zap.NewAtomicLevelAt(zapLevel)
    }
}

// WithOutputPaths sets output paths
func WithOutputPaths(paths []string) Option {
    return func(config *zap.Config) {
        config.OutputPaths = paths
    }
}

// Context-aware logging
type contextKey string

const (
    requestIDKey contextKey = "request_id"
    userIDKey    contextKey = "user_id"
)

// WithRequestID adds request ID to context
func WithRequestID(ctx context.Context, requestID string) context.Context {
    return context.WithValue(ctx, requestIDKey, requestID)
}

// WithUserID adds user ID to context
func WithUserID(ctx context.Context, userID string) context.Context {
    return context.WithValue(ctx, userIDKey, userID)
}

// FromContext creates logger with context values
func (l *Logger) FromContext(ctx context.Context) *Logger {
    fields := []zap.Field{}
    
    if requestID, ok := ctx.Value(requestIDKey).(string); ok {
        fields = append(fields, zap.String("request_id", requestID))
    }
    
    if userID, ok := ctx.Value(userIDKey).(string); ok {
        fields = append(fields, zap.String("user_id", userID))
    }
    
    return &Logger{Logger: l.With(fields...)}
}

// Convenience methods
func (l *Logger) WithError(err error) *Logger {
    return &Logger{Logger: l.With(zap.Error(err))}
}

func (l *Logger) WithField(key string, value interface{}) *Logger {
    return &Logger{Logger: l.With(zap.Any(key, value))}
}

func (l *Logger) WithFields(fields map[string]interface{}) *Logger {
    zapFields := make([]zap.Field, 0, len(fields))
    for k, v := range fields {
        zapFields = append(zapFields, zap.Any(k, v))
    }
    return &Logger{Logger: l.With(zapFields...)}
}
```

### Logging Best Practices

```go
// internal/usecase/user_usecase.go
package usecase

import (
    "context"
    "time"
    
    "github.com/myapp/pkg/logger"
    "go.uber.org/zap"
)

type UserUseCase struct {
    repo   UserRepository
    logger *logger.Logger
}

func (uc *UserUseCase) CreateUser(ctx context.Context, req CreateUserRequest) (*User, error) {
    log := uc.logger.FromContext(ctx)
    
    // Log at start with input parameters
    log.Info("creating user",
        zap.String("email", req.Email),
        zap.String("name", req.Name),
    )
    
    start := time.Now()
    defer func() {
        // Log duration
        duration := time.Since(start)
        log.Info("user creation completed",
            zap.Duration("duration", duration),
        )
    }()
    
    // Validate input
    if err := req.Validate(); err != nil {
        log.Warn("validation failed",
            zap.Error(err),
            zap.String("email", req.Email),
        )
        return nil, err
    }
    
    // Check if user exists
    existing, err := uc.repo.GetByEmail(ctx, req.Email)
    if err != nil {
        log.Error("failed to check existing user",
            zap.Error(err),
            zap.String("email", req.Email),
        )
        return nil, err
    }
    
    if existing != nil {
        log.Warn("user already exists",
            zap.String("email", req.Email),
            zap.String("existing_user_id", existing.ID),
        )
        return nil, ErrUserAlreadyExists
    }
    
    // Create user
    user := &User{
        Email:     req.Email,
        Name:      req.Name,
        CreatedAt: time.Now(),
    }
    
    if err := uc.repo.Create(ctx, user); err != nil {
        log.Error("failed to create user",
            zap.Error(err),
            zap.String("email", req.Email),
        )
        return nil, err
    }
    
    log.Info("user created successfully",
        zap.String("user_id", user.ID),
        zap.String("email", user.Email),
    )
    
    return user, nil
}
```

### Log Levels Strategy

```go
// When to use each level:

// DEBUG - Development/troubleshooting
log.Debug("cache lookup",
    zap.String("key", key),
    zap.Bool("found", found),
)

// INFO - Normal operations, state changes
log.Info("user registered",
    zap.String("user_id", user.ID),
)

// WARN - Unexpected but recoverable
log.Warn("rate limit exceeded",
    zap.String("client_ip", ip),
    zap.Int("attempts", attempts),
)

// ERROR - Error conditions
log.Error("database query failed",
    zap.Error(err),
    zap.String("query", query),
)

// FATAL - Unrecoverable, will exit
log.Fatal("failed to connect to database",
    zap.Error(err),
)
```

### Logging HTTP Requests

```go
// internal/delivery/http/middleware/logging.go
package middleware

import (
    "net/http"
    "time"
    
    "github.com/myapp/pkg/logger"
    "go.uber.org/zap"
)

type responseWriter struct {
    http.ResponseWriter
    status      int
    size        int
    wroteHeader bool
}

func (rw *responseWriter) WriteHeader(code int) {
    if !rw.wroteHeader {
        rw.status = code
        rw.ResponseWriter.WriteHeader(code)
        rw.wroteHeader = true
    }
}

func (rw *responseWriter) Write(b []byte) (int, error) {
    if !rw.wroteHeader {
        rw.WriteHeader(http.StatusOK)
    }
    size, err := rw.ResponseWriter.Write(b)
    rw.size += size
    return size, err
}

func Logging(log *logger.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Get request ID from context
            requestID := r.Context().Value("request_id")
            
            wrapped := &responseWriter{
                ResponseWriter: w,
                status:         http.StatusOK,
            }
            
            // Add request ID to context
            ctx := logger.WithRequestID(r.Context(), requestID.(string))
            
            // Process request
            next.ServeHTTP(wrapped, r.WithContext(ctx))
            
            // Log request
            duration := time.Since(start)
            
            log.FromContext(ctx).Info("http request",
                zap.String("method", r.Method),
                zap.String("path", r.URL.Path),
                zap.String("remote_addr", r.RemoteAddr),
                zap.Int("status", wrapped.status),
                zap.Int("size", wrapped.size),
                zap.Duration("duration", duration),
                zap.String("user_agent", r.UserAgent()),
            )
            
            // Log slow requests
            if duration > 1*time.Second {
                log.FromContext(ctx).Warn("slow request",
                    zap.String("method", r.Method),
                    zap.String("path", r.URL.Path),
                    zap.Duration("duration", duration),
                )
            }
        })
    }
}
```

### Sampling for High-Traffic Services

```go
// pkg/logger/sampler.go
package logger

import (
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

// NewSampledLogger creates logger with sampling
func NewSampledLogger(environment string) (*Logger, error) {
    config := zap.NewProductionConfig()
    
    // Sample logs: keep first 100/sec, then 10% of remainder
    config.Sampling = &zap.SamplingConfig{
        Initial:    100,
        Thereafter: 10,
    }
    
    logger, err := config.Build()
    if err != nil {
        return nil, err
    }
    
    return &Logger{Logger: logger}, nil
}
```

### Log Aggregation

```go
// Sending logs to external systems

// Fluentd/Fluent Bit - Forward to log aggregation
// Configure output to send JSON logs

// Elasticsearch - Direct indexing
// Use elasticsearch output plugin

// CloudWatch Logs - AWS integration
import (
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/service/cloudwatchlogs"
)

func NewCloudWatchLogger(logGroup, logStream string) (*Logger, error) {
    // Create CloudWatch client
    svc := cloudwatchlogs.New(session.Must(session.NewSession()))
    
    // Create custom writer
    writer := &cloudWatchWriter{
        logGroup:  logGroup,
        logStream: logStream,
        svc:       svc,
    }
    
    // Create logger with CloudWatch output
    config := zap.NewProductionConfig()
    config.OutputPaths = []string{"stdout"}
    
    logger, err := config.Build(zap.WrapCore(func(core zapcore.Core) zapcore.Core {
        return zapcore.NewTee(
            core,
            zapcore.NewCore(
                zapcore.NewJSONEncoder(config.EncoderConfig),
                zapcore.AddSync(writer),
                config.Level,
            ),
        )
    }))
    
    return &Logger{Logger: logger}, err
}
```

Continuing with Part 6...
---

## 6.3 Metrics with Prometheus

### Understanding Metric Types

**Counter** - Only increases
```go
// Request count, error count, bytes sent
httpRequestsTotal.Inc()
```

**Gauge** - Can go up or down
```go
// Active connections, memory usage, queue size
activeConnections.Set(42)
```

**Histogram** - Samples observations
```go
// Request duration, response size
requestDuration.Observe(0.5)
```

**Summary** - Similar to histogram with quantiles
```go
// Request duration with quantiles
requestDuration.Observe(0.5)
```

### Implementing Prometheus Metrics

```go
// pkg/metrics/prometheus.go
package metrics

import (
    "net/http"
    "time"
    
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

// Metrics holds all application metrics
type Metrics struct {
    // HTTP metrics
    httpRequestsTotal   *prometheus.CounterVec
    httpRequestDuration *prometheus.HistogramVec
    httpRequestSize     *prometheus.HistogramVec
    httpResponseSize    *prometheus.HistogramVec
    
    // Application metrics
    activeUsers         prometheus.Gauge
    cacheHitRate        prometheus.Gauge
    queueSize           prometheus.Gauge
    
    // Database metrics
    dbConnectionsActive prometheus.Gauge
    dbConnectionsIdle   prometheus.Gauge
    dbQueriesTotal      *prometheus.CounterVec
    dbQueryDuration     *prometheus.HistogramVec
    
    // Business metrics
    ordersTotal         *prometheus.CounterVec
    orderValue          *prometheus.HistogramVec
}

func New(namespace string) *Metrics {
    return &Metrics{
        httpRequestsTotal: promauto.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "http_requests_total",
                Help:      "Total number of HTTP requests",
            },
            []string{"method", "path", "status"},
        ),
        
        httpRequestDuration: promauto.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "http_request_duration_seconds",
                Help:      "HTTP request duration in seconds",
                Buckets:   []float64{.001, .005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10},
            },
            []string{"method", "path"},
        ),
        
        httpRequestSize: promauto.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "http_request_size_bytes",
                Help:      "HTTP request size in bytes",
                Buckets:   prometheus.ExponentialBuckets(100, 10, 8),
            },
            []string{"method", "path"},
        ),
        
        httpResponseSize: promauto.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "http_response_size_bytes",
                Help:      "HTTP response size in bytes",
                Buckets:   prometheus.ExponentialBuckets(100, 10, 8),
            },
            []string{"method", "path"},
        ),
        
        activeUsers: promauto.NewGauge(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "active_users",
                Help:      "Number of currently active users",
            },
        ),
        
        cacheHitRate: promauto.NewGauge(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "cache_hit_rate",
                Help:      "Cache hit rate (0-1)",
            },
        ),
        
        queueSize: promauto.NewGauge(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "queue_size",
                Help:      "Current size of the work queue",
            },
        ),
        
        dbConnectionsActive: promauto.NewGauge(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "db_connections_active",
                Help:      "Number of active database connections",
            },
        ),
        
        dbConnectionsIdle: promauto.NewGauge(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "db_connections_idle",
                Help:      "Number of idle database connections",
            },
        ),
        
        dbQueriesTotal: promauto.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "db_queries_total",
                Help:      "Total number of database queries",
            },
            []string{"operation", "table", "status"},
        ),
        
        dbQueryDuration: promauto.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "db_query_duration_seconds",
                Help:      "Database query duration in seconds",
                Buckets:   []float64{.001, .005, .01, .025, .05, .1, .25, .5, 1},
            },
            []string{"operation", "table"},
        ),
        
        ordersTotal: promauto.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "orders_total",
                Help:      "Total number of orders",
            },
            []string{"status"},
        ),
        
        orderValue: promauto.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "order_value_dollars",
                Help:      "Order value in dollars",
                Buckets:   []float64{10, 25, 50, 100, 250, 500, 1000, 2500, 5000},
            },
            []string{"status"},
        ),
    }
}

// HTTP Metrics

func (m *Metrics) RecordHTTPRequest(method, path string, status int, duration time.Duration, requestSize, responseSize int) {
    m.httpRequestsTotal.WithLabelValues(method, path, http.StatusText(status)).Inc()
    m.httpRequestDuration.WithLabelValues(method, path).Observe(duration.Seconds())
    m.httpRequestSize.WithLabelValues(method, path).Observe(float64(requestSize))
    m.httpResponseSize.WithLabelValues(method, path).Observe(float64(responseSize))
}

// Application Metrics

func (m *Metrics) SetActiveUsers(count int) {
    m.activeUsers.Set(float64(count))
}

func (m *Metrics) SetCacheHitRate(rate float64) {
    m.cacheHitRate.Set(rate)
}

func (m *Metrics) SetQueueSize(size int) {
    m.queueSize.Set(float64(size))
}

// Database Metrics

func (m *Metrics) UpdateDBStats(stats DBStats) {
    m.dbConnectionsActive.Set(float64(stats.OpenConnections))
    m.dbConnectionsIdle.Set(float64(stats.Idle))
}

func (m *Metrics) RecordDBQuery(operation, table, status string, duration time.Duration) {
    m.dbQueriesTotal.WithLabelValues(operation, table, status).Inc()
    m.dbQueryDuration.WithLabelValues(operation, table).Observe(duration.Seconds())
}

// Business Metrics

func (m *Metrics) RecordOrder(status string, value float64) {
    m.ordersTotal.WithLabelValues(status).Inc()
    m.orderValue.WithLabelValues(status).Observe(value)
}

// Handler returns Prometheus HTTP handler
func Handler() http.Handler {
    return promhttp.Handler()
}
```

### Metrics Middleware

```go
// internal/delivery/http/middleware/metrics.go
package middleware

import (
    "net/http"
    "strconv"
    "time"
    
    "github.com/myapp/pkg/metrics"
)

type metricsResponseWriter struct {
    http.ResponseWriter
    statusCode int
    size       int
}

func (mrw *metricsResponseWriter) WriteHeader(code int) {
    mrw.statusCode = code
    mrw.ResponseWriter.WriteHeader(code)
}

func (mrw *metricsResponseWriter) Write(b []byte) (int, error) {
    size, err := mrw.ResponseWriter.Write(b)
    mrw.size += size
    return size, err
}

func Metrics(m *metrics.Metrics) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            wrapped := &metricsResponseWriter{
                ResponseWriter: w,
                statusCode:     http.StatusOK,
            }
            
            next.ServeHTTP(wrapped, r)
            
            duration := time.Since(start)
            
            m.RecordHTTPRequest(
                r.Method,
                r.URL.Path,
                wrapped.statusCode,
                duration,
                int(r.ContentLength),
                wrapped.size,
            )
        })
    }
}
```

### Database Metrics Collection

```go
// internal/repository/postgres/metrics.go
package postgres

import (
    "context"
    "database/sql"
    "time"
    
    "github.com/myapp/pkg/metrics"
)

type metricsRepository struct {
    db      *sql.DB
    metrics *metrics.Metrics
}

func NewMetricsRepository(db *sql.DB, m *metrics.Metrics) *metricsRepository {
    repo := &metricsRepository{
        db:      db,
        metrics: m,
    }
    
    // Collect DB stats periodically
    go repo.collectDBStats()
    
    return repo
}

func (r *metricsRepository) collectDBStats() {
    ticker := time.NewTicker(15 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        stats := r.db.Stats()
        r.metrics.UpdateDBStats(metrics.DBStats{
            OpenConnections: stats.OpenConnections,
            Idle:            stats.Idle,
        })
    }
}

func (r *metricsRepository) Query(ctx context.Context, query string, args ...interface{}) (*sql.Rows, error) {
    start := time.Now()
    
    rows, err := r.db.QueryContext(ctx, query, args...)
    
    duration := time.Since(start)
    status := "success"
    if err != nil {
        status = "error"
    }
    
    r.metrics.RecordDBQuery("SELECT", extractTable(query), status, duration)
    
    return rows, err
}
```

### Custom Business Metrics

```go
// internal/usecase/order_usecase.go
package usecase

import (
    "context"
    
    "github.com/myapp/internal/domain"
    "github.com/myapp/pkg/metrics"
)

type OrderUseCase struct {
    repo    OrderRepository
    metrics *metrics.Metrics
}

func (uc *OrderUseCase) CreateOrder(ctx context.Context, order *domain.Order) error {
    if err := uc.repo.Create(ctx, order); err != nil {
        return err
    }
    
    // Record business metric
    uc.metrics.RecordOrder("created", order.Total)
    
    return nil
}

func (uc *OrderUseCase) CompleteOrder(ctx context.Context, orderID string) error {
    order, err := uc.repo.GetByID(ctx, orderID)
    if err != nil {
        return err
    }
    
    order.Status = "completed"
    
    if err := uc.repo.Update(ctx, order); err != nil {
        return err
    }
    
    // Record business metric
    uc.metrics.RecordOrder("completed", order.Total)
    
    return nil
}
```

### Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'myapp'
    static_configs:
      - targets: ['localhost:8080']
    metrics_path: '/metrics'
```

### Essential Metrics Checklist

**RED Metrics (Requests, Errors, Duration):**
- ✅ Request rate
- ✅ Error rate
- ✅ Request duration (p50, p95, p99)

**USE Metrics (Utilization, Saturation, Errors):**
- ✅ CPU utilization
- ✅ Memory utilization
- ✅ Disk I/O
- ✅ Network throughput

**Golden Signals:**
- ✅ Latency
- ✅ Traffic
- ✅ Errors
- ✅ Saturation

**Business Metrics:**
- ✅ Orders created/completed
- ✅ Revenue
- ✅ User signups
- ✅ Feature usage

---

## 6.4 Distributed Tracing

### Understanding Distributed Tracing

```
Request Flow with Tracing:

┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│  Client  │─────▶│   API    │─────▶│  User    │─────▶│ Database │
│          │      │ Gateway  │      │ Service  │      │          │
└──────────┘      └──────────┘      └──────────┘      └──────────┘
                       │                  │                  │
                       │                  │                  │
                  ┌────▼──────────────────▼──────────────────▼────┐
                  │           Trace (Request Journey)            │
                  │                                               │
                  │  Span 1: API Gateway (10ms)                  │
                  │  Span 2: User Service (50ms)                 │
                  │    Span 3: Database Query (30ms)             │
                  │  Total: 60ms                                 │
                  └───────────────────────────────────────────────┘
```

### OpenTelemetry Setup

```go
// pkg/tracing/tracing.go
package tracing

import (
    "context"
    "fmt"
    
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/propagation"
    "go.opentelemetry.io/otel/sdk/resource"
    tracesdk "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.4.0"
    "go.opentelemetry.io/otel/trace"
)

// InitTracer initializes OpenTelemetry tracer
func InitTracer(serviceName, environment string) (func(context.Context) error, error) {
    // Create Jaeger exporter
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint())
    if err != nil {
        return nil, err
    }
    
    // Create resource with service information
    res, err := resource.New(context.Background(),
        resource.WithAttributes(
            semconv.ServiceNameKey.String(serviceName),
            semconv.DeploymentEnvironmentKey.String(environment),
        ),
    )
    if err != nil {
        return nil, err
    }
    
    // Create trace provider
    tp := tracesdk.NewTracerProvider(
        tracesdk.WithBatcher(exporter),
        tracesdk.WithResource(res),
        tracesdk.WithSampler(tracesdk.ParentBased(tracesdk.TraceIDRatioBased(0.5))), // 50% sampling
    )
    
    // Set global trace provider
    otel.SetTracerProvider(tp)
    
    // Set global propagator for trace context
    otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
        propagation.TraceContext{},
        propagation.Baggage{},
    ))
    
    // Return shutdown function
    return tp.Shutdown, nil
}

// Tracer returns a tracer for the given name
func Tracer(name string) trace.Tracer {
    return otel.Tracer(name)
}
```

### HTTP Tracing Middleware

```go
// internal/delivery/http/middleware/tracing.go
package middleware

import (
    "net/http"
    
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/codes"
    "go.opentelemetry.io/otel/propagation"
    "go.opentelemetry.io/otel/trace"
)

func Tracing(serviceName string) func(http.Handler) http.Handler {
    tracer := otel.Tracer(serviceName)
    propagator := otel.GetTextMapPropagator()
    
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Extract trace context from headers
            ctx := propagator.Extract(r.Context(), propagation.HeaderCarrier(r.Header))
            
            // Start span
            ctx, span := tracer.Start(ctx, r.Method+" "+r.URL.Path,
                trace.WithSpanKind(trace.SpanKindServer),
                trace.WithAttributes(
                    attribute.String("http.method", r.Method),
                    attribute.String("http.url", r.URL.String()),
                    attribute.String("http.target", r.URL.Path),
                    attribute.String("http.host", r.Host),
                    attribute.String("http.scheme", r.URL.Scheme),
                    attribute.String("http.user_agent", r.UserAgent()),
                ),
            )
            defer span.End()
            
            // Wrap response writer to capture status
            wrapped := &tracingResponseWriter{
                ResponseWriter: w,
                statusCode:     http.StatusOK,
            }
            
            // Process request
            next.ServeHTTP(wrapped, r.WithContext(ctx))
            
            // Record status code
            span.SetAttributes(attribute.Int("http.status_code", wrapped.statusCode))
            
            // Mark as error if 5xx
            if wrapped.statusCode >= 500 {
                span.SetStatus(codes.Error, http.StatusText(wrapped.statusCode))
            }
        })
    }
}

type tracingResponseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (trw *tracingResponseWriter) WriteHeader(code int) {
    trw.statusCode = code
    trw.ResponseWriter.WriteHeader(code)
}
```

### Service-to-Service Tracing

```go
// internal/client/user_client.go
package client

import (
    "context"
    "net/http"
    
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/propagation"
    "go.opentelemetry.io/otel/trace"
)

type UserClient struct {
    baseURL    string
    httpClient *http.Client
    tracer     trace.Tracer
}

func NewUserClient(baseURL string) *UserClient {
    return &UserClient{
        baseURL:    baseURL,
        httpClient: &http.Client{},
        tracer:     otel.Tracer("user-client"),
    }
}

func (c *UserClient) GetUser(ctx context.Context, userID string) (*User, error) {
    // Start span
    ctx, span := c.tracer.Start(ctx, "UserClient.GetUser",
        trace.WithSpanKind(trace.SpanKindClient),
        trace.WithAttributes(
            attribute.String("user.id", userID),
        ),
    )
    defer span.End()
    
    // Create request
    url := c.baseURL + "/users/" + userID
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }
    
    // Inject trace context into headers
    otel.GetTextMapPropagator().Inject(ctx, propagation.HeaderCarrier(req.Header))
    
    // Make request
    resp, err := c.httpClient.Do(req)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }
    defer resp.Body.Close()
    
    span.SetAttributes(attribute.Int("http.status_code", resp.StatusCode))
    
    // Decode response
    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        span.RecordError(err)
        return nil, err
    }
    
    return &user, nil
}
```

### Database Tracing

```go
// internal/repository/postgres/user_repo.go
package postgres

import (
    "context"
    "database/sql"
    
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/codes"
    "go.opentelemetry.io/otel/trace"
)

type userRepository struct {
    db     *sql.DB
    tracer trace.Tracer
}

func NewUserRepository(db *sql.DB) *userRepository {
    return &userRepository{
        db:     db,
        tracer: otel.Tracer("user-repository"),
    }
}

func (r *userRepository) GetByID(ctx context.Context, id string) (*domain.User, error) {
    ctx, span := r.tracer.Start(ctx, "UserRepository.GetByID",
        trace.WithSpanKind(trace.SpanKindClient),
        trace.WithAttributes(
            attribute.String("db.system", "postgresql"),
            attribute.String("db.operation", "SELECT"),
            attribute.String("db.table", "users"),
        ),
    )
    defer span.End()
    
    query := `SELECT id, email, name, created_at FROM users WHERE id = $1`
    span.SetAttributes(attribute.String("db.statement", query))
    
    var user domain.User
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Email,
        &user.Name,
        &user.CreatedAt,
    )
    
    if err != nil {
        if err == sql.ErrNoRows {
            span.SetStatus(codes.Error, "user not found")
        } else {
            span.RecordError(err)
            span.SetStatus(codes.Error, err.Error())
        }
        return nil, err
    }
    
    return &user, nil
}
```

### Custom Spans in Use Cases

```go
// internal/usecase/order_usecase.go
package usecase

import (
    "context"
    
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"
)

type OrderUseCase struct {
    orderRepo    OrderRepository
    inventoryRepo InventoryRepository
    tracer       trace.Tracer
}

func NewOrderUseCase(orderRepo OrderRepository, inventoryRepo InventoryRepository) *OrderUseCase {
    return &OrderUseCase{
        orderRepo:    orderRepo,
        inventoryRepo: inventoryRepo,
        tracer:       otel.Tracer("order-usecase"),
    }
}

func (uc *OrderUseCase) PlaceOrder(ctx context.Context, order *domain.Order) error {
    ctx, span := uc.tracer.Start(ctx, "OrderUseCase.PlaceOrder",
        trace.WithAttributes(
            attribute.String("order.id", order.ID),
            attribute.Int("order.items", len(order.Items)),
            attribute.Float64("order.total", order.Total),
        ),
    )
    defer span.End()
    
    // Validate order
    ctx, validateSpan := uc.tracer.Start(ctx, "validate_order")
    if err := order.Validate(); err != nil {
        validateSpan.RecordError(err)
        validateSpan.End()
        span.RecordError(err)
        return err
    }
    validateSpan.End()
    
    // Check inventory
    ctx, inventorySpan := uc.tracer.Start(ctx, "check_inventory")
    for _, item := range order.Items {
        available, err := uc.inventoryRepo.CheckStock(ctx, item.ProductID, item.Quantity)
        if err != nil {
            inventorySpan.RecordError(err)
            inventorySpan.End()
            span.RecordError(err)
            return err
        }
        
        if !available {
            err := ErrInsufficientStock
            inventorySpan.RecordError(err)
            inventorySpan.End()
            span.RecordError(err)
            return err
        }
    }
    inventorySpan.End()
    
    // Save order
    ctx, saveSpan := uc.tracer.Start(ctx, "save_order")
    if err := uc.orderRepo.Create(ctx, order); err != nil {
        saveSpan.RecordError(err)
        saveSpan.End()
        span.RecordError(err)
        return err
    }
    saveSpan.End()
    
    span.AddEvent("order_placed", trace.WithAttributes(
        attribute.String("order.id", order.ID),
    ))
    
    return nil
}
```

Continuing with Part 6...


---

## 6.5 Health Checks and Probes

### Types of Health Checks

**Liveness** - Is the application alive?
```go
// Returns 200 if app can process requests
func (h *HealthHandler) Liveness(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
}
```

**Readiness** - Is the application ready to serve traffic?
```go
// Returns 200 only if all dependencies are available
func (h *HealthHandler) Readiness(w http.ResponseWriter, r *http.Request) {
    if !h.checkDatabase() || !h.checkCache() {
        w.WriteHeader(http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
}
```

**Startup** - Has the application finished starting?
```go
// Returns 200 when initialization complete
func (h *HealthHandler) Startup(w http.ResponseWriter, r *http.Request) {
    if !h.initialized {
        w.WriteHeader(http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
}
```

### Comprehensive Health Check Implementation

```go
// pkg/health/health.go
package health

import (
    "context"
    "database/sql"
    "encoding/json"
    "net/http"
    "sync"
    "time"
)

type Status string

const (
    StatusHealthy   Status = "healthy"
    StatusDegraded  Status = "degraded"
    StatusUnhealthy Status = "unhealthy"
)

type Check struct {
    Name     string        `json:"name"`
    Status   Status        `json:"status"`
    Message  string        `json:"message,omitempty"`
    Duration time.Duration `json:"duration"`
}

type HealthResponse struct {
    Status    Status            `json:"status"`
    Checks    []Check           `json:"checks"`
    Timestamp time.Time         `json:"timestamp"`
    Version   string            `json:"version"`
    Metadata  map[string]string `json:"metadata,omitempty"`
}

type Checker interface {
    Check(ctx context.Context) Check
}

type Health struct {
    checkers map[string]Checker
    version  string
    metadata map[string]string
    mu       sync.RWMutex
}

func New(version string) *Health {
    return &Health{
        checkers: make(map[string]Checker),
        version:  version,
        metadata: make(map[string]string),
    }
}

func (h *Health) RegisterChecker(name string, checker Checker) {
    h.mu.Lock()
    defer h.mu.Unlock()
    h.checkers[name] = checker
}

func (h *Health) AddMetadata(key, value string) {
    h.mu.Lock()
    defer h.mu.Unlock()
    h.metadata[key] = value
}

func (h *Health) Check(ctx context.Context) HealthResponse {
    h.mu.RLock()
    checkers := make(map[string]Checker, len(h.checkers))
    for name, checker := range h.checkers {
        checkers[name] = checker
    }
    h.mu.RUnlock()
    
    // Run checks concurrently
    checks := make([]Check, 0, len(checkers))
    checkChan := make(chan Check, len(checkers))
    
    var wg sync.WaitGroup
    for name, checker := range checkers {
        wg.Add(1)
        go func(name string, checker Checker) {
            defer wg.Done()
            checkChan <- checker.Check(ctx)
        }(name, checker)
    }
    
    go func() {
        wg.Wait()
        close(checkChan)
    }()
    
    for check := range checkChan {
        checks = append(checks, check)
    }
    
    // Determine overall status
    overallStatus := StatusHealthy
    for _, check := range checks {
        if check.Status == StatusUnhealthy {
            overallStatus = StatusUnhealthy
            break
        }
        if check.Status == StatusDegraded && overallStatus == StatusHealthy {
            overallStatus = StatusDegraded
        }
    }
    
    return HealthResponse{
        Status:    overallStatus,
        Checks:    checks,
        Timestamp: time.Now(),
        Version:   h.version,
        Metadata:  h.metadata,
    }
}

// HTTP Handler
func (h *Health) Handler() http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
        defer cancel()
        
        result := h.Check(ctx)
        
        statusCode := http.StatusOK
        if result.Status == StatusUnhealthy {
            statusCode = http.StatusServiceUnavailable
        } else if result.Status == StatusDegraded {
            statusCode = http.StatusOK // Still serving traffic
        }
        
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(statusCode)
        json.NewEncoder(w).Encode(result)
    }
}
```

### Database Health Checker

```go
// pkg/health/database.go
package health

import (
    "context"
    "database/sql"
    "time"
)

type DatabaseChecker struct {
    db      *sql.DB
    timeout time.Duration
}

func NewDatabaseChecker(db *sql.DB, timeout time.Duration) *DatabaseChecker {
    return &DatabaseChecker{
        db:      db,
        timeout: timeout,
    }
}

func (c *DatabaseChecker) Check(ctx context.Context) Check {
    start := time.Now()
    
    ctx, cancel := context.WithTimeout(ctx, c.timeout)
    defer cancel()
    
    if err := c.db.PingContext(ctx); err != nil {
        return Check{
            Name:     "database",
            Status:   StatusUnhealthy,
            Message:  err.Error(),
            Duration: time.Since(start),
        }
    }
    
    // Check connection pool stats
    stats := c.db.Stats()
    
    // Warn if no idle connections
    status := StatusHealthy
    message := "connected"
    
    if stats.Idle == 0 && stats.OpenConnections == stats.MaxOpenConnections {
        status = StatusDegraded
        message = "connection pool exhausted"
    }
    
    return Check{
        Name:     "database",
        Status:   status,
        Message:  message,
        Duration: time.Since(start),
    }
}
```

### Redis Health Checker

```go
// pkg/health/redis.go
package health

import (
    "context"
    "time"
    
    "github.com/go-redis/redis/v8"
)

type RedisChecker struct {
    client  *redis.Client
    timeout time.Duration
}

func NewRedisChecker(client *redis.Client, timeout time.Duration) *RedisChecker {
    return &RedisChecker{
        client:  client,
        timeout: timeout,
    }
}

func (c *RedisChecker) Check(ctx context.Context) Check {
    start := time.Now()
    
    ctx, cancel := context.WithTimeout(ctx, c.timeout)
    defer cancel()
    
    if err := c.client.Ping(ctx).Err(); err != nil {
        return Check{
            Name:     "redis",
            Status:   StatusUnhealthy,
            Message:  err.Error(),
            Duration: time.Since(start),
        }
    }
    
    return Check{
        Name:     "redis",
        Status:   StatusHealthy,
        Message:  "connected",
        Duration: time.Since(start),
    }
}
```

### External Service Health Checker

```go
// pkg/health/http_service.go
package health

import (
    "context"
    "net/http"
    "time"
)

type HTTPServiceChecker struct {
    name    string
    url     string
    client  *http.Client
    timeout time.Duration
}

func NewHTTPServiceChecker(name, url string, timeout time.Duration) *HTTPServiceChecker {
    return &HTTPServiceChecker{
        name: name,
        url:  url,
        client: &http.Client{
            Timeout: timeout,
        },
        timeout: timeout,
    }
}

func (c *HTTPServiceChecker) Check(ctx context.Context) Check {
    start := time.Now()
    
    req, err := http.NewRequestWithContext(ctx, "GET", c.url, nil)
    if err != nil {
        return Check{
            Name:     c.name,
            Status:   StatusUnhealthy,
            Message:  err.Error(),
            Duration: time.Since(start),
        }
    }
    
    resp, err := c.client.Do(req)
    if err != nil {
        return Check{
            Name:     c.name,
            Status:   StatusUnhealthy,
            Message:  err.Error(),
            Duration: time.Since(start),
        }
    }
    defer resp.Body.Close()
    
    if resp.StatusCode >= 500 {
        return Check{
            Name:     c.name,
            Status:   StatusUnhealthy,
            Message:  "service unavailable",
            Duration: time.Since(start),
        }
    }
    
    if resp.StatusCode >= 400 {
        return Check{
            Name:     c.name,
            Status:   StatusDegraded,
            Message:  "service degraded",
            Duration: time.Since(start),
        }
    }
    
    return Check{
        Name:     c.name,
        Status:   StatusHealthy,
        Message:  "available",
        Duration: time.Since(start),
    }
}
```

### Usage Example

```go
// cmd/api/main.go
func setupHealthChecks(app *Application) *health.Health {
    h := health.New(app.Config.Version)
    
    // Add metadata
    h.AddMetadata("environment", app.Config.Environment)
    h.AddMetadata("instance_id", os.Getenv("INSTANCE_ID"))
    
    // Register checkers
    h.RegisterChecker("database", health.NewDatabaseChecker(app.DB, 2*time.Second))
    h.RegisterChecker("redis", health.NewRedisChecker(app.Redis, 2*time.Second))
    h.RegisterChecker("user-service", 
        health.NewHTTPServiceChecker("user-service", "http://user-service/health", 2*time.Second))
    
    return h
}

func main() {
    app := NewApplication()
    health := setupHealthChecks(app)
    
    // Health endpoints
    http.HandleFunc("/health", health.Handler())
    http.HandleFunc("/health/live", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
    })
    http.HandleFunc("/health/ready", health.Handler())
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## 6.6 Error Tracking and Alerting

### Error Tracking with Sentry

```go
// pkg/errors/sentry.go
package errors

import (
    "context"
    "time"
    
    "github.com/getsentry/sentry-go"
)

type ErrorTracker struct {
    client *sentry.Client
}

func NewErrorTracker(dsn, environment, release string) (*ErrorTracker, error) {
    err := sentry.Init(sentry.ClientOptions{
        Dsn:              dsn,
        Environment:      environment,
        Release:          release,
        AttachStacktrace: true,
        TracesSampleRate: 0.5, // 50% of transactions
    })
    
    if err != nil {
        return nil, err
    }
    
    return &ErrorTracker{
        client: sentry.CurrentHub().Client(),
    }, nil
}

func (et *ErrorTracker) CaptureError(err error, ctx context.Context) {
    hub := sentry.CurrentHub().Clone()
    
    // Add context
    if requestID, ok := ctx.Value("request_id").(string); ok {
        hub.Scope().SetTag("request_id", requestID)
    }
    
    if userID, ok := ctx.Value("user_id").(string); ok {
        hub.Scope().SetUser(sentry.User{ID: userID})
    }
    
    hub.CaptureException(err)
}

func (et *ErrorTracker) CaptureMessage(message string, level sentry.Level) {
    sentry.CaptureMessage(message)
}

func (et *ErrorTracker) Flush(timeout time.Duration) {
    sentry.Flush(timeout)
}
```

### Error Handling Middleware

```go
// internal/delivery/http/middleware/error_tracking.go
package middleware

import (
    "net/http"
    
    "github.com/getsentry/sentry-go"
    sentryhttp "github.com/getsentry/sentry-go/http"
)

func ErrorTracking() func(http.Handler) http.Handler {
    sentryHandler := sentryhttp.New(sentryhttp.Options{
        Repanic:         false,
        WaitForDelivery: false,
    })
    
    return sentryHandler.Handle
}

// Capture panic
func RecoverWithSentry(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                hub := sentry.CurrentHub().Clone()
                hub.Scope().SetRequest(r)
                hub.Recover(err)
                
                http.Error(w, "Internal Server Error", http.StatusInternalServerError)
            }
        }()
        
        next.ServeHTTP(w, r)
    })
}
```

### Alerting with Alert Manager

```go
// pkg/alerting/alertmanager.go
package alerting

import (
    "bytes"
    "context"
    "encoding/json"
    "net/http"
    "time"
)

type Alert struct {
    Labels      map[string]string `json:"labels"`
    Annotations map[string]string `json:"annotations"`
    StartsAt    time.Time         `json:"startsAt"`
    EndsAt      time.Time         `json:"endsAt,omitempty"`
}

type AlertManager struct {
    url    string
    client *http.Client
}

func NewAlertManager(url string) *AlertManager {
    return &AlertManager{
        url: url,
        client: &http.Client{
            Timeout: 5 * time.Second,
        },
    }
}

func (am *AlertManager) SendAlert(ctx context.Context, alert Alert) error {
    alerts := []Alert{alert}
    
    body, err := json.Marshal(alerts)
    if err != nil {
        return err
    }
    
    req, err := http.NewRequestWithContext(ctx, "POST", am.url+"/api/v1/alerts", bytes.NewBuffer(body))
    if err != nil {
        return err
    }
    
    req.Header.Set("Content-Type", "application/json")
    
    resp, err := am.client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    return nil
}

// Convenience methods
func (am *AlertManager) HighErrorRate(service string, rate float64) error {
    alert := Alert{
        Labels: map[string]string{
            "alertname": "HighErrorRate",
            "service":   service,
            "severity":  "warning",
        },
        Annotations: map[string]string{
            "summary":     "High error rate detected",
            "description": fmt.Sprintf("Error rate is %.2f%%", rate),
        },
        StartsAt: time.Now(),
    }
    
    return am.SendAlert(context.Background(), alert)
}

func (am *AlertManager) ServiceDown(service string) error {
    alert := Alert{
        Labels: map[string]string{
            "alertname": "ServiceDown",
            "service":   service,
            "severity":  "critical",
        },
        Annotations: map[string]string{
            "summary":     "Service is down",
            "description": fmt.Sprintf("%s is not responding", service),
        },
        StartsAt: time.Now(),
    }
    
    return am.SendAlert(context.Background(), alert)
}
```

---

## 6.7 Application Performance Monitoring (APM)

### Custom APM Implementation

```go
// pkg/apm/apm.go
package apm

import (
    "context"
    "sync"
    "time"
)

type Transaction struct {
    Name      string
    Type      string
    StartTime time.Time
    Duration  time.Duration
    Spans     []*Span
    Error     error
    Metadata  map[string]interface{}
    mu        sync.Mutex
}

type Span struct {
    Name      string
    Type      string
    StartTime time.Time
    Duration  time.Duration
    Error     error
    Metadata  map[string]interface{}
}

type APM struct {
    transactions chan *Transaction
    mu           sync.Mutex
}

func New() *APM {
    apm := &APM{
        transactions: make(chan *Transaction, 1000),
    }
    
    go apm.processTransactions()
    
    return apm
}

func (a *APM) processTransactions() {
    for tx := range a.transactions {
        // Send to APM backend (e.g., Elastic APM, Datadog)
        a.sendToBackend(tx)
    }
}

func (a *APM) sendToBackend(tx *Transaction) {
    // Implementation depends on APM provider
}

func (a *APM) StartTransaction(name, txType string) *Transaction {
    return &Transaction{
        Name:      name,
        Type:      txType,
        StartTime: time.Now(),
        Metadata:  make(map[string]interface{}),
        Spans:     make([]*Span, 0),
    }
}

func (tx *Transaction) End() {
    tx.Duration = time.Since(tx.StartTime)
    // Send to APM
}

func (tx *Transaction) StartSpan(name, spanType string) *Span {
    span := &Span{
        Name:      name,
        Type:      spanType,
        StartTime: time.Now(),
        Metadata:  make(map[string]interface{}),
    }
    
    tx.mu.Lock()
    tx.Spans = append(tx.Spans, span)
    tx.mu.Unlock()
    
    return span
}

func (s *Span) End() {
    s.Duration = time.Since(s.StartTime)
}

func (tx *Transaction) SetMetadata(key string, value interface{}) {
    tx.mu.Lock()
    tx.Metadata[key] = value
    tx.mu.Unlock()
}
```

### Usage in HTTP Handlers

```go
// internal/delivery/http/middleware/apm.go
package middleware

import (
    "net/http"
    
    "github.com/myapp/pkg/apm"
)

func APM(apmClient *apm.APM) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            tx := apmClient.StartTransaction(r.Method+" "+r.URL.Path, "request")
            defer tx.End()
            
            tx.SetMetadata("http.method", r.Method)
            tx.SetMetadata("http.url", r.URL.String())
            tx.SetMetadata("http.user_agent", r.UserAgent())
            
            wrapped := &apmResponseWriter{
                ResponseWriter: w,
                transaction:    tx,
            }
            
            next.ServeHTTP(wrapped, r)
            
            tx.SetMetadata("http.status_code", wrapped.statusCode)
        })
    }
}
```

Continuing with Part 6...


## 6.8 Debugging Production Systems

### Live Debugging Tools

**pprof endpoints for production:**

```go
// cmd/api/main.go
import _ "net/http/pprof"

func main() {
    // Debug server (separate port for security)
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // Main application server
    log.Fatal(http.ListenAndServe(":8080", handler))
}
```

**Accessing profiles:**

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
```

### Delve for Remote Debugging

```go
// Build with debug symbols
go build -gcflags="all=-N -l" -o myapp

// Start with delve
dlv exec ./myapp --headless --listen=:2345 --api-version=2
```

```bash
# Connect remotely
dlv connect localhost:2345
```

### Debug Logging Toggle

```go
// pkg/debug/debug.go
package debug

import (
    "net/http"
    "sync/atomic"
)

var debugEnabled atomic.Value

func init() {
    debugEnabled.Store(false)
}

func IsEnabled() bool {
    return debugEnabled.Load().(bool)
}

func Enable() {
    debugEnabled.Store(true)
}

func Disable() {
    debugEnabled.Store(false)
}

// HTTP handler to toggle debug mode
func ToggleHandler() http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        if r.Method != "POST" {
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
            return
        }
        
        action := r.URL.Query().Get("action")
        
        switch action {
        case "enable":
            Enable()
            w.Write([]byte("Debug mode enabled"))
        case "disable":
            Disable()
            w.Write([]byte("Debug mode disabled"))
        default:
            http.Error(w, "Invalid action", http.StatusBadRequest)
        }
    }
}

// Usage in code
func processRequest(ctx context.Context, req Request) {
    if debug.IsEnabled() {
        log.Debug("Processing request",
            zap.Any("request", req),
        )
    }
    
    // Process request
}
```

### Request Dumping

```go
// pkg/debug/httpdump.go
package debug

import (
    "bytes"
    "io"
    "net/http"
    "net/http/httputil"
)

func DumpRequest(r *http.Request) string {
    if !IsEnabled() {
        return ""
    }
    
    dump, err := httputil.DumpRequest(r, true)
    if err != nil {
        return err.Error()
    }
    
    return string(dump)
}

func DumpResponse(resp *http.Response) string {
    if !IsEnabled() {
        return ""
    }
    
    dump, err := httputil.DumpResponse(resp, true)
    if err != nil {
        return err.Error()
    }
    
    return string(dump)
}

// Middleware to log requests when debug enabled
func RequestDumper(log *logger.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            if IsEnabled() {
                dump := DumpRequest(r)
                log.Debug("HTTP Request", zap.String("dump", dump))
            }
            
            next.ServeHTTP(w, r)
        })
    }
}
```

---

## 6.9 Production Incident Response

### Incident Response Checklist

```markdown
# Production Incident Response

## 1. Detection (0-5 minutes)
- [ ] Alert received
- [ ] Severity assessed (P0/P1/P2/P3)
- [ ] Incident channel created
- [ ] On-call engineer paged

## 2. Initial Response (5-15 minutes)
- [ ] Acknowledge incident
- [ ] Check dashboards (CPU, memory, errors, latency)
- [ ] Review recent deployments
- [ ] Check logs for errors
- [ ] Check traces for slow requests

## 3. Mitigation (15-60 minutes)
- [ ] Identify root cause
- [ ] Apply immediate fix or rollback
- [ ] Verify fix resolves issue
- [ ] Monitor for regression

## 4. Communication
- [ ] Update status page
- [ ] Notify stakeholders
- [ ] Provide regular updates
- [ ] Declare resolved

## 5. Post-Incident (24-48 hours)
- [ ] Write postmortem
- [ ] Identify root cause
- [ ] Create action items
- [ ] Update runbooks
```

### Incident Response Tools

```go
// pkg/incident/incident.go
package incident

import (
    "context"
    "time"
)

type Severity string

const (
    SeverityP0 Severity = "P0" // Critical - Complete outage
    SeverityP1 Severity = "P1" // High - Major feature unavailable
    SeverityP2 Severity = "P2" // Medium - Minor feature issue
    SeverityP3 Severity = "P3" // Low - Cosmetic issue
)

type Incident struct {
    ID          string
    Title       string
    Severity    Severity
    Status      string // investigating, identified, monitoring, resolved
    StartTime   time.Time
    ResolvedAt  *time.Time
    Description string
    Updates     []Update
    RootCause   string
}

type Update struct {
    Timestamp time.Time
    Message   string
    Author    string
}

type IncidentManager struct {
    incidents map[string]*Incident
}

func NewIncidentManager() *IncidentManager {
    return &IncidentManager{
        incidents: make(map[string]*Incident),
    }
}

func (im *IncidentManager) Create(title string, severity Severity) *Incident {
    incident := &Incident{
        ID:        generateID(),
        Title:     title,
        Severity:  severity,
        Status:    "investigating",
        StartTime: time.Now(),
        Updates:   make([]Update, 0),
    }
    
    im.incidents[incident.ID] = incident
    
    // Notify stakeholders
    im.notify(incident)
    
    return incident
}

func (im *IncidentManager) AddUpdate(incidentID, message, author string) {
    incident := im.incidents[incidentID]
    
    update := Update{
        Timestamp: time.Now(),
        Message:   message,
        Author:    author,
    }
    
    incident.Updates = append(incident.Updates, update)
    
    // Notify stakeholders
    im.notify(incident)
}

func (im *IncidentManager) Resolve(incidentID, rootCause string) {
    incident := im.incidents[incidentID]
    
    now := time.Now()
    incident.Status = "resolved"
    incident.ResolvedAt = &now
    incident.RootCause = rootCause
    
    // Notify stakeholders
    im.notify(incident)
}

func (im *IncidentManager) notify(incident *Incident) {
    // Send to Slack, PagerDuty, email, etc.
}
```

### Runbook Automation

```go
// scripts/runbooks/high_cpu.go
package runbooks

import (
    "context"
    "fmt"
    "os/exec"
)

type HighCPURunbook struct {
    serviceName string
}

func (r *HighCPURunbook) Execute(ctx context.Context) error {
    fmt.Println("=== High CPU Runbook ===")
    
    // Step 1: Collect CPU profile
    fmt.Println("Step 1: Collecting CPU profile...")
    if err := r.collectCPUProfile(ctx); err != nil {
        return fmt.Errorf("failed to collect CPU profile: %w", err)
    }
    
    // Step 2: Check for goroutine leaks
    fmt.Println("Step 2: Checking for goroutine leaks...")
    if err := r.checkGoroutines(ctx); err != nil {
        return fmt.Errorf("failed to check goroutines: %w", err)
    }
    
    // Step 3: Review recent deployments
    fmt.Println("Step 3: Reviewing recent deployments...")
    r.reviewDeployments(ctx)
    
    // Step 4: Check for infinite loops in logs
    fmt.Println("Step 4: Checking logs for patterns...")
    r.checkLogs(ctx)
    
    return nil
}

func (r *HighCPURunbook) collectCPUProfile(ctx context.Context) error {
    cmd := exec.CommandContext(ctx,
        "curl",
        "-o", "cpu.prof",
        "http://localhost:6060/debug/pprof/profile?seconds=30",
    )
    
    return cmd.Run()
}

func (r *HighCPURunbook) checkGoroutines(ctx context.Context) error {
    cmd := exec.CommandContext(ctx,
        "curl",
        "http://localhost:6060/debug/pprof/goroutine?debug=1",
    )
    
    output, err := cmd.Output()
    if err != nil {
        return err
    }
    
    fmt.Printf("Goroutine count: %s\n", output)
    return nil
}
```

---

## 6.10 Observability Best Practices

### The Four Golden Signals

```go
// pkg/metrics/golden_signals.go
package metrics

import (
    "time"
    
    "github.com/prometheus/client_golang/prometheus"
)

type GoldenSignals struct {
    // Latency - How long it takes to service a request
    latency *prometheus.HistogramVec
    
    // Traffic - How much demand is being placed on your system
    traffic *prometheus.CounterVec
    
    // Errors - The rate of requests that fail
    errors *prometheus.CounterVec
    
    // Saturation - How "full" your service is
    saturation *prometheus.GaugeVec
}

func NewGoldenSignals(namespace string) *GoldenSignals {
    return &GoldenSignals{
        latency: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "request_duration_seconds",
                Help:      "Request latency in seconds",
            },
            []string{"method", "endpoint"},
        ),
        
        traffic: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "requests_total",
                Help:      "Total number of requests",
            },
            []string{"method", "endpoint"},
        ),
        
        errors: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "errors_total",
                Help:      "Total number of errors",
            },
            []string{"method", "endpoint", "status"},
        ),
        
        saturation: prometheus.NewGaugeVec(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "saturation_percent",
                Help:      "Service saturation percentage",
            },
            []string{"resource"},
        ),
    }
}

func (gs *GoldenSignals) RecordRequest(method, endpoint string, duration time.Duration, statusCode int) {
    // Latency
    gs.latency.WithLabelValues(method, endpoint).Observe(duration.Seconds())
    
    // Traffic
    gs.traffic.WithLabelValues(method, endpoint).Inc()
    
    // Errors
    if statusCode >= 500 {
        gs.errors.WithLabelValues(method, endpoint, "5xx").Inc()
    } else if statusCode >= 400 {
        gs.errors.WithLabelValues(method, endpoint, "4xx").Inc()
    }
}

func (gs *GoldenSignals) UpdateSaturation(resource string, percent float64) {
    gs.saturation.WithLabelValues(resource).Set(percent)
}
```

### Observability Maturity Model

```
Level 0: No Observability
- No logging
- No metrics
- No tracing
- Manual debugging

Level 1: Basic Observability
- Unstructured logs
- Basic metrics (request count)
- No tracing
- Limited debugging

Level 2: Structured Observability
- Structured logs
- RED/USE metrics
- Basic tracing
- Some correlation

Level 3: Comprehensive Observability
- Centralized logging
- Custom business metrics
- Distributed tracing
- Full correlation
- Dashboards

Level 4: Advanced Observability
- Real-time alerts
- Anomaly detection
- Predictive analytics
- SLO-based monitoring
- Auto-remediation
```

### SLO-Based Monitoring

```go
// pkg/slo/slo.go
package slo

import (
    "time"
    
    "github.com/prometheus/client_golang/prometheus"
)

type SLO struct {
    name              string
    target            float64 // e.g., 99.9%
    window            time.Duration
    successCounter    prometheus.Counter
    totalCounter      prometheus.Counter
}

func NewSLO(name string, target float64, window time.Duration) *SLO {
    return &SLO{
        name:   name,
        target: target,
        window: window,
        
        successCounter: prometheus.NewCounter(
            prometheus.CounterOpts{
                Name: name + "_success_total",
                Help: "Total successful requests",
            },
        ),
        
        totalCounter: prometheus.NewCounter(
            prometheus.CounterOpts{
                Name: name + "_requests_total",
                Help: "Total requests",
            },
        ),
    }
}

func (s *SLO) RecordSuccess() {
    s.successCounter.Inc()
    s.totalCounter.Inc()
}

func (s *SLO) RecordFailure() {
    s.totalCounter.Inc()
}

// Error budget: how many failures we can tolerate
func (s *SLO) ErrorBudget() float64 {
    return 1.0 - s.target
}
```

---

## Part 6 Summary and Key Takeaways

### Observability Pillars

**✅ Logs:**
- Structured logging with Zap
- Context-aware logging
- Log levels strategy
- Sampling for high traffic
- Centralized aggregation

**✅ Metrics:**
- Prometheus integration
- Four metric types (Counter, Gauge, Histogram, Summary)
- RED metrics (Rate, Errors, Duration)
- USE metrics (Utilization, Saturation, Errors)
- Business metrics

**✅ Traces:**
- Distributed tracing with OpenTelemetry
- Service-to-service correlation
- Database tracing
- Custom spans
- Performance bottleneck identification

### Production Readiness

**✅ Health Checks:**
- Liveness, readiness, startup probes
- Dependency health checking
- Comprehensive health responses

**✅ Error Tracking:**
- Sentry integration
- Error aggregation
- Alerting on errors
- Stack traces and context

**✅ Performance Monitoring:**
- APM implementation
- Transaction tracking
- Span analysis
- Resource monitoring

**✅ Debugging:**
- Live profiling with pprof
- Remote debugging with Delve
- Debug mode toggling
- Request/response dumping

**✅ Incident Response:**
- Incident management
- Runbook automation
- Post-mortem process
- Communication strategy

### Best Practices Checklist

**Logging:**
- [ ] Use structured logging
- [ ] Include request IDs
- [ ] Log at appropriate levels
- [ ] Sample high-volume logs
- [ ] Centralize log aggregation

**Metrics:**
- [ ] Implement RED metrics
- [ ] Add business metrics
- [ ] Create dashboards
- [ ] Set up alerts
- [ ] Monitor golden signals

**Tracing:**
- [ ] Enable distributed tracing
- [ ] Trace critical paths
- [ ] Include database calls
- [ ] Sample appropriately
- [ ] Correlate with logs

**Health:**
- [ ] Implement health endpoints
- [ ] Check all dependencies
- [ ] Include version info
- [ ] Monitor continuously

**Alerting:**
- [ ] Alert on symptoms, not causes
- [ ] Avoid alert fatigue
- [ ] Use SLOs for alerting
- [ ] Include runbooks
- [ ] Test alert routing

### Common Anti-Patterns to Avoid

**❌ Logging:**
- Logging sensitive data
- Too many debug logs in production
- No structured logging
- Ignoring log rotation
- Not correlating logs with traces

**❌ Metrics:**
- Too many unique labels
- Not using histograms for latency
- Tracking everything without purpose
- Ignoring cardinality issues

**❌ Tracing:**
- Tracing everything (100% sampling)
- Not propagating context
- Missing critical operations
- Overhead in hot paths

**❌ Alerting:**
- Alerting on everything
- No clear ownership
- Missing runbooks
- Alert without action

### Tools and Ecosystem

**Logging:**
- Zap (high-performance)
- Logrus (popular)
- Zerolog (zero-allocation)

**Metrics:**
- Prometheus (standard)
- Grafana (visualization)
- VictoriaMetrics (scalable alternative)

**Tracing:**
- Jaeger (distributed tracing)
- Zipkin (alternative)
- OpenTelemetry (standard)

**APM:**
- Elastic APM
- Datadog
- New Relic
- Dynatrace

**Error Tracking:**
- Sentry
- Bugsnag
- Rollbar

### What's Next

With comprehensive observability in place, you can:

1. **Detect issues quickly** - Before users report them
2. **Debug efficiently** - With full context and correlation
3. **Optimize performance** - Based on real data
4. **Plan capacity** - Using actual metrics
5. **Improve reliability** - Through SLOs and error budgets

**Continue your observability journey:**
- Implement SLO-based monitoring
- Build comprehensive dashboards
- Create automated runbooks
- Establish on-call rotation
- Conduct game days

**You're now equipped to build fully observable Go applications!**

---

## Interview Questions (Observability)

**Question 1: How do you debug a memory leak in production?**

**Difficulty:** Senior

**Answer:**

**Steps:**

1. **Detect the leak:**
```bash
# Monitor memory over time
watch -n 1 'ps aux | grep myapp'

# Check Prometheus metrics
query: go_memstats_heap_alloc_bytes
```

2. **Collect heap profile:**
```bash
# Get heap profile
curl http://localhost:6060/debug/pprof/heap > heap.prof

# Analyze
go tool pprof heap.prof
(pprof) top
(pprof) list functionName
```

3. **Compare snapshots:**
```bash
# Before
curl http://localhost:6060/debug/pprof/heap > heap1.prof

# Wait and collect again
curl http://localhost:6060/debug/pprof/heap > heap2.prof

# Compare
go tool pprof -base heap1.prof heap2.prof
```

4. **Common causes:**
- Goroutine leaks (check goroutine profile)
- Unbounded caches
- Forgotten pointers in slices
- Global variables accumulating data

**What Interviewers Look For:**
- Systematic approach
- Knowledge of profiling tools
- Understanding of memory management
- Production debugging experience

---

**Question 2: What metrics would you track for an HTTP API?**

**Difficulty:** Mid-Level

**Answer:**

**Essential metrics:**

```go
// RED Metrics
1. Request Rate
   httpRequestsTotal.Inc()

2. Error Rate
   httpErrorsTotal.Inc()

3. Duration (latency)
   httpDuration.Observe(duration)

// Additional metrics
4. Request size
   httpRequestSize.Observe(size)

5. Response size
   httpResponseSize.Observe(size)

6. Active connections
   activeConnections.Set(count)

7. Queue depth
   requestQueueDepth.Set(depth)
```

**What Interviewers Look For:**
- Understanding of RED metrics
- Knowledge of key indicators
- Practical experience
- Monitoring best practices

---

This completes Part 6 of the Complete Go Mastery series! You now have comprehensive knowledge of observability, monitoring, and debugging production Go applications.

