---
title: "Complete Go Mastery Part 5: Production Architecture Patterns"
date: 2025-01-10 18:00:00 +0530
categories: [Programming, Go, Architecture]
tags: [golang, go, architecture, clean-architecture, ddd, hexagonal, microservices, api-design, patterns]
---

# Complete Go Mastery Part 5: Production Architecture Patterns

## Introduction

Welcome to Part 5 of the Complete Go Mastery series. In Parts 1-4, we covered Go fundamentals, concurrency, advanced techniques, and testing. Now we explore how to architect production-ready systems that scale, evolve, and remain maintainable.

Building production systems requires more than knowing the language—it requires understanding architectural patterns, design principles, and best practices that have been proven in real-world applications. Go's simplicity makes it easy to write code, but without proper architecture, systems become unmaintainable as they grow.

**Why is architecture critical?**

- **Scalability**: Handle growing traffic and data
- **Maintainability**: Easy to modify and extend
- **Testability**: Comprehensive testing at all levels
- **Team productivity**: Multiple developers working efficiently
- **Business agility**: Adapt to changing requirements

**What you'll learn in Part 5:**

- **Clean Architecture**: Organizing code by layers and dependencies
- **Domain-Driven Design**: Modeling complex business domains
- **Hexagonal Architecture**: Isolating core logic from infrastructure
- **API Design**: RESTful and RPC patterns
- **Database Patterns**: Repository, Unit of Work, migrations
- **Configuration Management**: Environment-based config
- **Dependency Injection**: Managing dependencies cleanly
- **Microservices Patterns**: Building distributed systems
- **Error Handling**: Production-grade error strategies
- **Logging and Observability**: Structured logging, metrics, tracing

By the end of Part 5, you'll be able to design and implement production-ready Go applications that follow industry best practices and scale with your business needs.

---

## 5.1 Clean Architecture in Go

Clean Architecture separates concerns into layers with clear dependency rules.

### Core Principles

**The Dependency Rule:**
> Source code dependencies must point only inward, toward higher-level policies.

```
┌─────────────────────────────────────────────┐
│          Frameworks & Drivers               │
│   (Web, DB, External APIs, UI)              │
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │     Interface Adapters                │ │
│  │  (Controllers, Presenters, Gateways)  │ │
│  │                                       │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │      Use Cases                  │ │ │
│  │  │  (Application Business Rules)   │ │ │
│  │  │                                 │ │ │
│  │  │  ┌───────────────────────────┐ │ │ │
│  │  │  │      Entities             │ │ │ │
│  │  │  │ (Enterprise Business)     │ │ │ │
│  │  │  │      Rules                │ │ │ │
│  │  │  └───────────────────────────┘ │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘

Dependencies flow inward →
```

### Project Structure

```
project/
├── cmd/
│   └── api/
│       └── main.go                 # Application entry point
├── internal/
│   ├── domain/                     # Enterprise Business Rules
│   │   ├── user.go                 # Entities
│   │   ├── order.go
│   │   └── errors.go
│   ├── usecase/                    # Application Business Rules
│   │   ├── user_usecase.go         # Use cases
│   │   ├── order_usecase.go
│   │   └── interfaces.go           # Repository interfaces
│   ├── delivery/                   # Interface Adapters
│   │   ├── http/
│   │   │   ├── handler.go          # HTTP handlers
│   │   │   ├── middleware.go
│   │   │   └── router.go
│   │   └── grpc/
│   │       └── server.go           # gRPC server
│   └── repository/                 # Frameworks & Drivers
│       ├── postgres/
│       │   └── user_repo.go        # PostgreSQL implementation
│       └── redis/
│           └── cache_repo.go       # Redis implementation
├── pkg/                            # Public packages
│   ├── logger/
│   └── config/
└── go.mod
```

### Layer 1: Entities (Domain)

Business objects with business rules:

```go
// internal/domain/user.go
package domain

import (
    "errors"
    "time"
)

// User is the core business entity
type User struct {
    ID        string
    Email     string
    Name      string
    Password  string // hashed
    CreatedAt time.Time
    UpdatedAt time.Time
}

// Validate checks business rules
func (u *User) Validate() error {
    if u.Email == "" {
        return errors.New("email is required")
    }
    
    if !isValidEmail(u.Email) {
        return errors.New("invalid email format")
    }
    
    if u.Name == "" {
        return errors.New("name is required")
    }
    
    return nil
}

// ChangeEmail updates email with validation
func (u *User) ChangeEmail(newEmail string) error {
    if !isValidEmail(newEmail) {
        return errors.New("invalid email format")
    }
    
    u.Email = newEmail
    u.UpdatedAt = time.Now()
    return nil
}

// Domain errors
var (
    ErrUserNotFound      = errors.New("user not found")
    ErrUserAlreadyExists = errors.New("user already exists")
    ErrInvalidCredentials = errors.New("invalid credentials")
)

func isValidEmail(email string) bool {
    // Email validation logic
    return len(email) > 3 && strings.Contains(email, "@")
}
```

### Layer 2: Use Cases

Application-specific business rules:

```go
// internal/usecase/interfaces.go
package usecase

import (
    "context"
    "github.com/myapp/internal/domain"
)

// UserRepository defines what we need from the data layer
type UserRepository interface {
    Create(ctx context.Context, user *domain.User) error
    GetByID(ctx context.Context, id string) (*domain.User, error)
    GetByEmail(ctx context.Context, email string) (*domain.User, error)
    Update(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id string) error
}

// PasswordHasher defines password hashing interface
type PasswordHasher interface {
    Hash(password string) (string, error)
    Compare(hashed, password string) error
}
```

```go
// internal/usecase/user_usecase.go
package usecase

import (
    "context"
    "github.com/myapp/internal/domain"
)

type UserUseCase struct {
    userRepo UserRepository
    hasher   PasswordHasher
}

func NewUserUseCase(repo UserRepository, hasher PasswordHasher) *UserUseCase {
    return &UserUseCase{
        userRepo: repo,
        hasher:   hasher,
    }
}

// RegisterUser handles user registration use case
func (uc *UserUseCase) RegisterUser(ctx context.Context, email, password, name string) (*domain.User, error) {
    // Check if user already exists
    existing, err := uc.userRepo.GetByEmail(ctx, email)
    if err == nil && existing != nil {
        return nil, domain.ErrUserAlreadyExists
    }
    
    // Hash password
    hashedPassword, err := uc.hasher.Hash(password)
    if err != nil {
        return nil, err
    }
    
    // Create user entity
    user := &domain.User{
        Email:    email,
        Password: hashedPassword,
        Name:     name,
    }
    
    // Validate business rules
    if err := user.Validate(); err != nil {
        return nil, err
    }
    
    // Persist user
    if err := uc.userRepo.Create(ctx, user); err != nil {
        return nil, err
    }
    
    return user, nil
}

// AuthenticateUser handles user login
func (uc *UserUseCase) AuthenticateUser(ctx context.Context, email, password string) (*domain.User, error) {
    user, err := uc.userRepo.GetByEmail(ctx, email)
    if err != nil {
        return nil, domain.ErrInvalidCredentials
    }
    
    if err := uc.hasher.Compare(user.Password, password); err != nil {
        return nil, domain.ErrInvalidCredentials
    }
    
    return user, nil
}

// GetUser retrieves a user by ID
func (uc *UserUseCase) GetUser(ctx context.Context, id string) (*domain.User, error) {
    user, err := uc.userRepo.GetByID(ctx, id)
    if err != nil {
        return nil, domain.ErrUserNotFound
    }
    
    return user, nil
}

// UpdateUserEmail updates user's email
func (uc *UserUseCase) UpdateUserEmail(ctx context.Context, userID, newEmail string) error {
    user, err := uc.userRepo.GetByID(ctx, userID)
    if err != nil {
        return domain.ErrUserNotFound
    }
    
    // Use domain method (business logic in entity)
    if err := user.ChangeEmail(newEmail); err != nil {
        return err
    }
    
    return uc.userRepo.Update(ctx, user)
}
```

### Layer 3: Interface Adapters

Convert data for use cases and external agencies:

```go
// internal/delivery/http/handler.go
package http

import (
    "encoding/json"
    "net/http"
    
    "github.com/myapp/internal/domain"
    "github.com/myapp/internal/usecase"
)

type UserHandler struct {
    userUseCase *usecase.UserUseCase
}

func NewUserHandler(uc *usecase.UserUseCase) *UserHandler {
    return &UserHandler{userUseCase: uc}
}

// RegisterRequest is the HTTP request DTO
type RegisterRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
    Name     string `json:"name"`
}

// UserResponse is the HTTP response DTO
type UserResponse struct {
    ID    string `json:"id"`
    Email string `json:"email"`
    Name  string `json:"name"`
}

func (h *UserHandler) Register(w http.ResponseWriter, r *http.Request) {
    var req RegisterRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, "invalid request")
        return
    }
    
    user, err := h.userUseCase.RegisterUser(r.Context(), req.Email, req.Password, req.Name)
    if err != nil {
        switch err {
        case domain.ErrUserAlreadyExists:
            respondError(w, http.StatusConflict, err.Error())
        default:
            respondError(w, http.StatusInternalServerError, "internal error")
        }
        return
    }
    
    // Convert domain entity to response DTO
    resp := UserResponse{
        ID:    user.ID,
        Email: user.Email,
        Name:  user.Name,
    }
    
    respondJSON(w, http.StatusCreated, resp)
}

func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")
    
    user, err := h.userUseCase.GetUser(r.Context(), userID)
    if err != nil {
        switch err {
        case domain.ErrUserNotFound:
            respondError(w, http.StatusNotFound, err.Error())
        default:
            respondError(w, http.StatusInternalServerError, "internal error")
        }
        return
    }
    
    resp := UserResponse{
        ID:    user.ID,
        Email: user.Email,
        Name:  user.Name,
    }
    
    respondJSON(w, http.StatusOK, resp)
}

func respondJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func respondError(w http.ResponseWriter, status int, message string) {
    respondJSON(w, status, map[string]string{"error": message})
}
```

### Layer 4: Frameworks & Drivers

External tools and frameworks:

```go
// internal/repository/postgres/user_repo.go
package postgres

import (
    "context"
    "database/sql"
    "time"
    
    "github.com/myapp/internal/domain"
)

type PostgresUserRepository struct {
    db *sql.DB
}

func NewPostgresUserRepository(db *sql.DB) *PostgresUserRepository {
    return &PostgresUserRepository{db: db}
}

func (r *PostgresUserRepository) Create(ctx context.Context, user *domain.User) error {
    query := `
        INSERT INTO users (id, email, password, name, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6)
    `
    
    user.CreatedAt = time.Now()
    user.UpdatedAt = time.Now()
    
    _, err := r.db.ExecContext(ctx, query,
        user.ID, user.Email, user.Password, user.Name,
        user.CreatedAt, user.UpdatedAt,
    )
    
    return err
}

func (r *PostgresUserRepository) GetByID(ctx context.Context, id string) (*domain.User, error) {
    query := `
        SELECT id, email, password, name, created_at, updated_at
        FROM users
        WHERE id = $1
    `
    
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID, &user.Email, &user.Password, &user.Name,
        &user.CreatedAt, &user.UpdatedAt,
    )
    
    if err == sql.ErrNoRows {
        return nil, domain.ErrUserNotFound
    }
    
    return user, err
}

func (r *PostgresUserRepository) GetByEmail(ctx context.Context, email string) (*domain.User, error) {
    query := `
        SELECT id, email, password, name, created_at, updated_at
        FROM users
        WHERE email = $1
    `
    
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &user.ID, &user.Email, &user.Password, &user.Name,
        &user.CreatedAt, &user.UpdatedAt,
    )
    
    if err == sql.ErrNoRows {
        return nil, domain.ErrUserNotFound
    }
    
    return user, err
}

func (r *PostgresUserRepository) Update(ctx context.Context, user *domain.User) error {
    query := `
        UPDATE users
        SET email = $2, password = $3, name = $4, updated_at = $5
        WHERE id = $1
    `
    
    user.UpdatedAt = time.Now()
    
    _, err := r.db.ExecContext(ctx, query,
        user.ID, user.Email, user.Password, user.Name, user.UpdatedAt,
    )
    
    return err
}

func (r *PostgresUserRepository) Delete(ctx context.Context, id string) error {
    query := `DELETE FROM users WHERE id = $1`
    _, err := r.db.ExecContext(ctx, query, id)
    return err
}
```

### Dependency Injection & Wiring

```go
// cmd/api/main.go
package main

import (
    "database/sql"
    "log"
    "net/http"
    
    _ "github.com/lib/pq"
    
    httpDelivery "github.com/myapp/internal/delivery/http"
    "github.com/myapp/internal/repository/postgres"
    "github.com/myapp/internal/usecase"
    "github.com/myapp/pkg/bcrypt"
)

func main() {
    // Initialize database
    db, err := sql.Open("postgres", "postgres://localhost/myapp?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Initialize repositories (Layer 4)
    userRepo := postgres.NewPostgresUserRepository(db)
    
    // Initialize dependencies
    hasher := bcrypt.NewBcryptHasher()
    
    // Initialize use cases (Layer 2)
    userUseCase := usecase.NewUserUseCase(userRepo, hasher)
    
    // Initialize handlers (Layer 3)
    userHandler := httpDelivery.NewUserHandler(userUseCase)
    
    // Setup router
    mux := http.NewServeMux()
    mux.HandleFunc("/users/register", userHandler.Register)
    mux.HandleFunc("/users", userHandler.GetUser)
    
    // Start server
    log.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

```go
// pkg/bcrypt/hasher.go
package bcrypt

import "golang.org/x/crypto/bcrypt"

type BcryptHasher struct{}

func NewBcryptHasher() *BcryptHasher {
    return &BcryptHasher{}
}

func (h *BcryptHasher) Hash(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(bytes), err
}

func (h *BcryptHasher) Compare(hashed, password string) error {
    return bcrypt.CompareHashAndPassword([]byte(hashed), []byte(password))
}
```

### Testing with Clean Architecture

Each layer can be tested independently:

```go
// internal/domain/user_test.go
package domain_test

import (
    "testing"
    "github.com/myapp/internal/domain"
)

func TestUser_Validate(t *testing.T) {
    tests := []struct {
        name    string
        user    domain.User
        wantErr bool
    }{
        {
            name: "valid user",
            user: domain.User{
                Email: "test@example.com",
                Name:  "Test User",
            },
            wantErr: false,
        },
        {
            name: "missing email",
            user: domain.User{
                Name: "Test User",
            },
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := tt.user.Validate()
            if (err != nil) != tt.wantErr {
                t.Errorf("Validate() error = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

```go
// internal/usecase/user_usecase_test.go
package usecase_test

import (
    "context"
    "testing"
    
    "github.com/myapp/internal/domain"
    "github.com/myapp/internal/usecase"
)

// Mock repository
type MockUserRepository struct {
    CreateFunc    func(context.Context, *domain.User) error
    GetByEmailFunc func(context.Context, string) (*domain.User, error)
}

func (m *MockUserRepository) Create(ctx context.Context, user *domain.User) error {
    return m.CreateFunc(ctx, user)
}

func (m *MockUserRepository) GetByEmail(ctx context.Context, email string) (*domain.User, error) {
    return m.GetByEmailFunc(ctx, email)
}

// Implement other methods...

// Mock hasher
type MockHasher struct{}

func (m *MockHasher) Hash(password string) (string, error) {
    return "hashed_" + password, nil
}

func (m *MockHasher) Compare(hashed, password string) error {
    if hashed == "hashed_"+password {
        return nil
    }
    return domain.ErrInvalidCredentials
}

func TestUserUseCase_RegisterUser(t *testing.T) {
    mockRepo := &MockUserRepository{
        GetByEmailFunc: func(ctx context.Context, email string) (*domain.User, error) {
            return nil, domain.ErrUserNotFound
        },
        CreateFunc: func(ctx context.Context, user *domain.User) error {
            user.ID = "123"
            return nil
        },
    }
    
    mockHasher := &MockHasher{}
    uc := usecase.NewUserUseCase(mockRepo, mockHasher)
    
    user, err := uc.RegisterUser(context.Background(), "test@example.com", "password", "Test")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    
    if user.Email != "test@example.com" {
        t.Errorf("email = %s; want test@example.com", user.Email)
    }
}
```

Continuing with Part 5...
### Benefits of Hexagonal Architecture

**✅ Testability:**
- Core logic tested without infrastructure
- Easy to mock adapters
- Swap implementations for testing

**✅ Flexibility:**
- Change database without touching business logic
- Add new delivery mechanisms (HTTP → gRPC → CLI)
- Switch external services easily

**✅ Maintainability:**
- Clear separation of concerns
- Dependencies point inward
- Easy to understand and modify

Continuing with Part 5...


## 5.4 API Design Patterns

### RESTful API Design

**Resource-based routing:**

```go
// internal/delivery/http/router.go
package http

import (
    "net/http"
    "github.com/gorilla/mux"
)

func NewRouter(userHandler *UserHandler, orderHandler *OrderHandler) *mux.Router {
    r := mux.NewRouter()
    
    // User routes
    r.HandleFunc("/api/v1/users", userHandler.Create).Methods("POST")
    r.HandleFunc("/api/v1/users/{id}", userHandler.Get).Methods("GET")
    r.HandleFunc("/api/v1/users/{id}", userHandler.Update).Methods("PUT")
    r.HandleFunc("/api/v1/users/{id}", userHandler.Delete).Methods("DELETE")
    r.HandleFunc("/api/v1/users", userHandler.List).Methods("GET")
    
    // Order routes
    r.HandleFunc("/api/v1/orders", orderHandler.Create).Methods("POST")
    r.HandleFunc("/api/v1/orders/{id}", orderHandler.Get).Methods("GET")
    r.HandleFunc("/api/v1/orders", orderHandler.List).Methods("GET")
    r.HandleFunc("/api/v1/orders/{id}/cancel", orderHandler.Cancel).Methods("POST")
    
    // Nested resources
    r.HandleFunc("/api/v1/users/{userId}/orders", orderHandler.ListByUser).Methods("GET")
    
    return r
}
```

**Request/Response DTOs:**

```go
// internal/delivery/http/dto.go
package http

import (
    "time"
    "github.com/myapp/internal/domain"
)

// CreateUserRequest represents the API request
type CreateUserRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Name     string `json:"name" validate:"required,min=2,max=100"`
    Password string `json:"password" validate:"required,min=8"`
}

// UserResponse represents the API response
type UserResponse struct {
    ID        string    `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    CreatedAt time.Time `json:"created_at"`
}

// ToUserResponse converts domain entity to response DTO
func ToUserResponse(user *domain.User) UserResponse {
    return UserResponse{
        ID:        user.ID,
        Email:     user.Email,
        Name:      user.Name,
        CreatedAt: user.CreatedAt,
    }
}

// PaginatedResponse wraps paginated results
type PaginatedResponse struct {
    Data       interface{} `json:"data"`
    Page       int         `json:"page"`
    PerPage    int         `json:"per_page"`
    Total      int64       `json:"total"`
    TotalPages int         `json:"total_pages"`
}

// ErrorResponse represents error response
type ErrorResponse struct {
    Error   string            `json:"error"`
    Message string            `json:"message"`
    Details map[string]string `json:"details,omitempty"`
}
```

**Handler with validation:**

```go
// internal/delivery/http/user_handler.go
package http

import (
    "encoding/json"
    "net/http"
    
    "github.com/go-playground/validator/v10"
    "github.com/gorilla/mux"
    
    "github.com/myapp/internal/domain"
    "github.com/myapp/internal/usecase"
)

type UserHandler struct {
    userUseCase *usecase.UserUseCase
    validator   *validator.Validate
}

func NewUserHandler(uc *usecase.UserUseCase) *UserHandler {
    return &UserHandler{
        userUseCase: uc,
        validator:   validator.New(),
    }
}

func (h *UserHandler) Create(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, "Invalid request body", nil)
        return
    }
    
    // Validate request
    if err := h.validator.Struct(req); err != nil {
        validationErrors := make(map[string]string)
        for _, err := range err.(validator.ValidationErrors) {
            validationErrors[err.Field()] = err.Tag()
        }
        respondError(w, http.StatusBadRequest, "Validation failed", validationErrors)
        return
    }
    
    // Call use case
    user, err := h.userUseCase.RegisterUser(
        r.Context(),
        req.Email,
        req.Password,
        req.Name,
    )
    
    if err != nil {
        switch err {
        case domain.ErrUserAlreadyExists:
            respondError(w, http.StatusConflict, err.Error(), nil)
        default:
            respondError(w, http.StatusInternalServerError, "Failed to create user", nil)
        }
        return
    }
    
    respondJSON(w, http.StatusCreated, ToUserResponse(user))
}

func (h *UserHandler) Get(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    id := vars["id"]
    
    user, err := h.userUseCase.GetUser(r.Context(), id)
    if err != nil {
        switch err {
        case domain.ErrUserNotFound:
            respondError(w, http.StatusNotFound, err.Error(), nil)
        default:
            respondError(w, http.StatusInternalServerError, "Failed to get user", nil)
        }
        return
    }
    
    respondJSON(w, http.StatusOK, ToUserResponse(user))
}

func (h *UserHandler) List(w http.ResponseWriter, r *http.Request) {
    // Parse query parameters
    page := parseIntQuery(r, "page", 1)
    perPage := parseIntQuery(r, "per_page", 20)
    
    if perPage > 100 {
        perPage = 100 // Max limit
    }
    
    users, total, err := h.userUseCase.ListUsers(r.Context(), page, perPage)
    if err != nil {
        respondError(w, http.StatusInternalServerError, "Failed to list users", nil)
        return
    }
    
    // Convert to response DTOs
    userResponses := make([]UserResponse, len(users))
    for i, user := range users {
        userResponses[i] = ToUserResponse(user)
    }
    
    // Create paginated response
    totalPages := int(total) / perPage
    if int(total)%perPage != 0 {
        totalPages++
    }
    
    response := PaginatedResponse{
        Data:       userResponses,
        Page:       page,
        PerPage:    perPage,
        Total:      total,
        TotalPages: totalPages,
    }
    
    respondJSON(w, http.StatusOK, response)
}

func respondJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func respondError(w http.ResponseWriter, status int, message string, details map[string]string) {
    response := ErrorResponse{
        Error:   http.StatusText(status),
        Message: message,
        Details: details,
    }
    respondJSON(w, status, response)
}

func parseIntQuery(r *http.Request, key string, defaultValue int) int {
    value := r.URL.Query().Get(key)
    if value == "" {
        return defaultValue
    }
    
    intValue, err := strconv.Atoi(value)
    if err != nil {
        return defaultValue
    }
    
    return intValue
}
```

### Middleware Patterns

```go
// internal/delivery/http/middleware/logging.go
package middleware

import (
    "log"
    "net/http"
    "time"
)

func Logging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Wrap response writer to capture status code
        wrapped := &responseWriter{ResponseWriter: w, statusCode: http.StatusOK}
        
        next.ServeHTTP(wrapped, r)
        
        log.Printf(
            "%s %s %d %v",
            r.Method,
            r.URL.Path,
            wrapped.statusCode,
            time.Since(start),
        )
    })
}

type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}
```

```go
// internal/delivery/http/middleware/auth.go
package middleware

import (
    "context"
    "net/http"
    "strings"
)

type contextKey string

const userContextKey contextKey = "user"

func Authentication(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            http.Error(w, "Missing authorization header", http.StatusUnauthorized)
            return
        }
        
        // Extract token
        parts := strings.Split(authHeader, " ")
        if len(parts) != 2 || parts[0] != "Bearer" {
            http.Error(w, "Invalid authorization header", http.StatusUnauthorized)
            return
        }
        
        token := parts[1]
        
        // Validate token and get user
        user, err := validateToken(token)
        if err != nil {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }
        
        // Add user to context
        ctx := context.WithValue(r.Context(), userContextKey, user)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// GetUserFromContext retrieves user from context
func GetUserFromContext(ctx context.Context) (*User, bool) {
    user, ok := ctx.Value(userContextKey).(*User)
    return user, ok
}

func validateToken(token string) (*User, error) {
    // TODO: Implement JWT validation
    return &User{ID: "123"}, nil
}
```

```go
// internal/delivery/http/middleware/recovery.go
package middleware

import (
    "log"
    "net/http"
    "runtime/debug"
)

func Recovery(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                log.Printf("Panic recovered: %v\n%s", err, debug.Stack())
                http.Error(w, "Internal server error", http.StatusInternalServerError)
            }
        }()
        
        next.ServeHTTP(w, r)
    })
}
```

```go
// internal/delivery/http/middleware/ratelimit.go
package middleware

import (
    "net/http"
    "sync"
    "time"
    
    "golang.org/x/time/rate"
)

type RateLimiter struct {
    limiters map[string]*rate.Limiter
    mu       sync.RWMutex
    rate     rate.Limit
    burst    int
}

func NewRateLimiter(r rate.Limit, b int) *RateLimiter {
    return &RateLimiter{
        limiters: make(map[string]*rate.Limiter),
        rate:     r,
        burst:    b,
    }
}

func (rl *RateLimiter) getLimiter(key string) *rate.Limiter {
    rl.mu.RLock()
    limiter, exists := rl.limiters[key]
    rl.mu.RUnlock()
    
    if !exists {
        rl.mu.Lock()
        limiter = rate.NewLimiter(rl.rate, rl.burst)
        rl.limiters[key] = limiter
        rl.mu.Unlock()
    }
    
    return limiter
}

func (rl *RateLimiter) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Use IP address as key
        key := r.RemoteAddr
        
        limiter := rl.getLimiter(key)
        
        if !limiter.Allow() {
            http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}
```

**Composing middleware:**

```go
// cmd/api/main.go
func main() {
    // ... setup dependencies
    
    router := http.NewRouter(userHandler, orderHandler)
    
    // Apply middleware
    var handler http.Handler = router
    
    // Apply in reverse order (innermost first)
    handler = middleware.Recovery(handler)
    handler = middleware.Logging(handler)
    
    rateLimiter := middleware.NewRateLimiter(10, 20) // 10 req/sec, burst 20
    handler = rateLimiter.Middleware(handler)
    
    // Protected routes only
    protected := mux.NewRouter()
    protected.HandleFunc("/api/v1/profile", profileHandler.Get).Methods("GET")
    protected.Use(middleware.Authentication)
    
    http.Handle("/", handler)
    http.Handle("/api/v1/profile", protected)
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### gRPC API Pattern

```protobuf
// api/proto/user.proto
syntax = "proto3";

package user;

option go_package = "github.com/myapp/api/gen/user";

service UserService {
    rpc CreateUser(CreateUserRequest) returns (UserResponse);
    rpc GetUser(GetUserRequest) returns (UserResponse);
    rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
}

message CreateUserRequest {
    string email = 1;
    string name = 2;
    string password = 3;
}

message GetUserRequest {
    string id = 1;
}

message ListUsersRequest {
    int32 page = 1;
    int32 per_page = 2;
}

message UserResponse {
    string id = 1;
    string email = 2;
    string name = 3;
    int64 created_at = 4;
}

message ListUsersResponse {
    repeated UserResponse users = 1;
    int64 total = 2;
}
```

```go
// internal/delivery/grpc/user_server.go
package grpc

import (
    "context"
    
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    
    pb "github.com/myapp/api/gen/user"
    "github.com/myapp/internal/domain"
    "github.com/myapp/internal/usecase"
)

type UserServer struct {
    pb.UnimplementedUserServiceServer
    userUseCase *usecase.UserUseCase
}

func NewUserServer(uc *usecase.UserUseCase) *UserServer {
    return &UserServer{userUseCase: uc}
}

func (s *UserServer) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.UserResponse, error) {
    user, err := s.userUseCase.RegisterUser(ctx, req.Email, req.Password, req.Name)
    if err != nil {
        switch err {
        case domain.ErrUserAlreadyExists:
            return nil, status.Error(codes.AlreadyExists, err.Error())
        default:
            return nil, status.Error(codes.Internal, "Failed to create user")
        }
    }
    
    return toUserProto(user), nil
}

func (s *UserServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.UserResponse, error) {
    user, err := s.userUseCase.GetUser(ctx, req.Id)
    if err != nil {
        switch err {
        case domain.ErrUserNotFound:
            return nil, status.Error(codes.NotFound, err.Error())
        default:
            return nil, status.Error(codes.Internal, "Failed to get user")
        }
    }
    
    return toUserProto(user), nil
}

func (s *UserServer) ListUsers(ctx context.Context, req *pb.ListUsersRequest) (*pb.ListUsersResponse, error) {
    page := int(req.Page)
    if page <= 0 {
        page = 1
    }
    
    perPage := int(req.PerPage)
    if perPage <= 0 {
        perPage = 20
    }
    
    users, total, err := s.userUseCase.ListUsers(ctx, page, perPage)
    if err != nil {
        return nil, status.Error(codes.Internal, "Failed to list users")
    }
    
    userProtos := make([]*pb.UserResponse, len(users))
    for i, user := range users {
        userProtos[i] = toUserProto(user)
    }
    
    return &pb.ListUsersResponse{
        Users: userProtos,
        Total: total,
    }, nil
}

func toUserProto(user *domain.User) *pb.UserResponse {
    return &pb.UserResponse{
        Id:        user.ID,
        Email:     user.Email,
        Name:      user.Name,
        CreatedAt: user.CreatedAt.Unix(),
    }
}
```

### API Versioning

```go
// Version in URL path
r.HandleFunc("/api/v1/users", handlerV1.List)
r.HandleFunc("/api/v2/users", handlerV2.List)

// Version in header
func VersionMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        version := r.Header.Get("API-Version")
        if version == "" {
            version = "v1" // Default
        }
        
        ctx := context.WithValue(r.Context(), "api-version", version)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

---

## 5.5 Database Patterns

### Repository Pattern

Already covered in previous sections, here's advanced usage:

```go
// internal/repository/repository.go
package repository

import (
    "context"
    "database/sql"
)

// Transaction represents a database transaction
type Transaction interface {
    Commit() error
    Rollback() error
}

// TransactionManager manages database transactions
type TransactionManager interface {
    BeginTx(ctx context.Context) (Transaction, error)
    WithTransaction(ctx context.Context, fn func(ctx context.Context) error) error
}

// postgresTransactionManager implements TransactionManager
type postgresTransactionManager struct {
    db *sql.DB
}

func NewTransactionManager(db *sql.DB) TransactionManager {
    return &postgresTransactionManager{db: db}
}

func (tm *postgresTransactionManager) BeginTx(ctx context.Context) (Transaction, error) {
    return tm.db.BeginTx(ctx, nil)
}

func (tm *postgresTransactionManager) WithTransaction(ctx context.Context, fn func(ctx context.Context) error) error {
    tx, err := tm.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    
    // Add transaction to context
    ctx = context.WithValue(ctx, "tx", tx)
    
    defer func() {
        if p := recover(); p != nil {
            tx.Rollback()
            panic(p)
        }
    }()
    
    if err := fn(ctx); err != nil {
        tx.Rollback()
        return err
    }
    
    return tx.Commit()
}

// GetTx retrieves transaction from context
func GetTx(ctx context.Context) (*sql.Tx, bool) {
    tx, ok := ctx.Value("tx").(*sql.Tx)
    return tx, ok
}
```

```go
// internal/repository/postgres/user_repo.go
package postgres

import (
    "context"
    "database/sql"
    
    "github.com/myapp/internal/domain"
    "github.com/myapp/internal/repository"
)

type userRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *userRepository {
    return &userRepository{db: db}
}

// getExecutor returns either transaction or database
func (r *userRepository) getExecutor(ctx context.Context) executor {
    if tx, ok := repository.GetTx(ctx); ok {
        return tx
    }
    return r.db
}

type executor interface {
    ExecContext(ctx context.Context, query string, args ...interface{}) (sql.Result, error)
    QueryContext(ctx context.Context, query string, args ...interface{}) (*sql.Rows, error)
    QueryRowContext(ctx context.Context, query string, args ...interface{}) *sql.Row
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    query := `
        INSERT INTO users (id, email, name, created_at)
        VALUES ($1, $2, $3, $4)
    `
    
    exec := r.getExecutor(ctx)
    _, err := exec.ExecContext(ctx, query, user.ID, user.Email, user.Name, user.CreatedAt)
    return err
}
```

**Using transactions:**

```go
// internal/usecase/order_usecase.go
func (uc *OrderUseCase) PlaceOrder(ctx context.Context, order *domain.Order) error {
    return uc.txManager.WithTransaction(ctx, func(ctx context.Context) error {
        // All operations in same transaction
        
        // 1. Save order
        if err := uc.orderRepo.Create(ctx, order); err != nil {
            return err
        }
        
        // 2. Update inventory
        for _, item := range order.Items {
            if err := uc.inventoryRepo.DecreaseStock(ctx, item.ProductID, item.Quantity); err != nil {
                return err // Will rollback
            }
        }
        
        // 3. Create payment record
        payment := &domain.Payment{
            OrderID: order.ID,
            Amount:  order.Total,
        }
        if err := uc.paymentRepo.Create(ctx, payment); err != nil {
            return err // Will rollback
        }
        
        return nil // Will commit
    })
}
```

### Query Builder Pattern

```go
// internal/repository/query_builder.go
package repository

import (
    "fmt"
    "strings"
)

type QueryBuilder struct {
    table      string
    columns    []string
    where      []string
    args       []interface{}
    orderBy    string
    limit      int
    offset     int
}

func NewQueryBuilder(table string) *QueryBuilder {
    return &QueryBuilder{
        table:   table,
        columns: []string{"*"},
    }
}

func (qb *QueryBuilder) Select(columns ...string) *QueryBuilder {
    qb.columns = columns
    return qb
}

func (qb *QueryBuilder) Where(condition string, args ...interface{}) *QueryBuilder {
    qb.where = append(qb.where, condition)
    qb.args = append(qb.args, args...)
    return qb
}

func (qb *QueryBuilder) OrderBy(column string, direction string) *QueryBuilder {
    qb.orderBy = fmt.Sprintf("%s %s", column, direction)
    return qb
}

func (qb *QueryBuilder) Limit(limit int) *QueryBuilder {
    qb.limit = limit
    return qb
}

func (qb *QueryBuilder) Offset(offset int) *QueryBuilder {
    qb.offset = offset
    return qb
}

func (qb *QueryBuilder) Build() (string, []interface{}) {
    query := fmt.Sprintf("SELECT %s FROM %s", 
        strings.Join(qb.columns, ", "), 
        qb.table,
    )
    
    if len(qb.where) > 0 {
        query += " WHERE " + strings.Join(qb.where, " AND ")
    }
    
    if qb.orderBy != "" {
        query += " ORDER BY " + qb.orderBy
    }
    
    if qb.limit > 0 {
        query += fmt.Sprintf(" LIMIT %d", qb.limit)
    }
    
    if qb.offset > 0 {
        query += fmt.Sprintf(" OFFSET %d", qb.offset)
    }
    
    return query, qb.args
}

// Usage
func (r *userRepository) FindWithFilters(ctx context.Context, filters UserFilters) ([]*domain.User, error) {
    qb := NewQueryBuilder("users").
        Select("id", "email", "name", "created_at")
    
    if filters.Email != "" {
        qb.Where("email = $1", filters.Email)
    }
    
    if filters.CreatedAfter != nil {
        qb.Where("created_at > $2", filters.CreatedAfter)
    }
    
    qb.OrderBy("created_at", "DESC").
        Limit(filters.Limit).
        Offset(filters.Offset)
    
    query, args := qb.Build()
    
    rows, err := r.db.QueryContext(ctx, query, args...)
    // ... scan results
}
```

Continuing with Part 5...


### Database Migrations

```go
// internal/repository/migrations/migrations.go
package migrations

import (
    "database/sql"
    "embed"
    
    "github.com/golang-migrate/migrate/v4"
    "github.com/golang-migrate/migrate/v4/database/postgres"
    "github.com/golang-migrate/migrate/v4/source/iofs"
)

//go:embed *.sql
var migrationsFS embed.FS

func RunMigrations(db *sql.DB) error {
    driver, err := postgres.WithInstance(db, &postgres.Config{})
    if err != nil {
        return err
    }
    
    source, err := iofs.New(migrationsFS, ".")
    if err != nil {
        return err
    }
    
    m, err := migrate.NewWithInstance("iofs", source, "postgres", driver)
    if err != nil {
        return err
    }
    
    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        return err
    }
    
    return nil
}
```

```sql
-- internal/repository/migrations/001_create_users_table.up.sql
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    active BOOLEAN DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

```sql
-- internal/repository/migrations/001_create_users_table.down.sql
DROP TABLE IF EXISTS users;
```

---

## 5.6 Configuration Management

### Environment-Based Configuration

```go
// pkg/config/config.go
package config

import (
    "fmt"
    "os"
    "strconv"
    "time"
    
    "github.com/joho/godotenv"
)

type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    Redis    RedisConfig
    JWT      JWTConfig
    Email    EmailConfig
}

type ServerConfig struct {
    Port         int
    Environment  string
    ReadTimeout  time.Duration
    WriteTimeout time.Duration
}

type DatabaseConfig struct {
    Host     string
    Port     int
    User     string
    Password string
    Database string
    SSLMode  string
    MaxConns int
    MaxIdle  int
}

type RedisConfig struct {
    Host     string
    Port     int
    Password string
    DB       int
}

type JWTConfig struct {
    Secret     string
    Expiration time.Duration
}

type EmailConfig struct {
    Host     string
    Port     int
    Username string
    Password string
    From     string
}

func Load() (*Config, error) {
    // Load .env file if exists (development)
    _ = godotenv.Load()
    
    cfg := &Config{
        Server: ServerConfig{
            Port:         getEnvInt("SERVER_PORT", 8080),
            Environment:  getEnv("ENVIRONMENT", "development"),
            ReadTimeout:  getEnvDuration("SERVER_READ_TIMEOUT", 10*time.Second),
            WriteTimeout: getEnvDuration("SERVER_WRITE_TIMEOUT", 10*time.Second),
        },
        Database: DatabaseConfig{
            Host:     getEnv("DB_HOST", "localhost"),
            Port:     getEnvInt("DB_PORT", 5432),
            User:     getEnv("DB_USER", "postgres"),
            Password: getEnv("DB_PASSWORD", ""),
            Database: getEnv("DB_NAME", "myapp"),
            SSLMode:  getEnv("DB_SSLMODE", "disable"),
            MaxConns: getEnvInt("DB_MAX_CONNS", 25),
            MaxIdle:  getEnvInt("DB_MAX_IDLE", 5),
        },
        Redis: RedisConfig{
            Host:     getEnv("REDIS_HOST", "localhost"),
            Port:     getEnvInt("REDIS_PORT", 6379),
            Password: getEnv("REDIS_PASSWORD", ""),
            DB:       getEnvInt("REDIS_DB", 0),
        },
        JWT: JWTConfig{
            Secret:     getEnv("JWT_SECRET", ""),
            Expiration: getEnvDuration("JWT_EXPIRATION", 24*time.Hour),
        },
        Email: EmailConfig{
            Host:     getEnv("EMAIL_HOST", "smtp.gmail.com"),
            Port:     getEnvInt("EMAIL_PORT", 587),
            Username: getEnv("EMAIL_USERNAME", ""),
            Password: getEnv("EMAIL_PASSWORD", ""),
            From:     getEnv("EMAIL_FROM", "noreply@myapp.com"),
        },
    }
    
    if err := cfg.Validate(); err != nil {
        return nil, err
    }
    
    return cfg, nil
}

func (c *Config) Validate() error {
    if c.JWT.Secret == "" {
        return fmt.Errorf("JWT_SECRET is required")
    }
    
    if c.Database.Password == "" && c.Server.Environment == "production" {
        return fmt.Errorf("DB_PASSWORD is required in production")
    }
    
    return nil
}

func (c *Config) DatabaseDSN() string {
    return fmt.Sprintf(
        "host=%s port=%d user=%s password=%s dbname=%s sslmode=%s",
        c.Database.Host,
        c.Database.Port,
        c.Database.User,
        c.Database.Password,
        c.Database.Database,
        c.Database.SSLMode,
    )
}

func (c *Config) IsProduction() bool {
    return c.Server.Environment == "production"
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func getEnvInt(key string, defaultValue int) int {
    if value := os.Getenv(key); value != "" {
        if intVal, err := strconv.Atoi(value); err == nil {
            return intVal
        }
    }
    return defaultValue
}

func getEnvDuration(key string, defaultValue time.Duration) time.Duration {
    if value := os.Getenv(key); value != "" {
        if duration, err := time.ParseDuration(value); err == nil {
            return duration
        }
    }
    return defaultValue
}
```

**Usage:**

```go
// cmd/api/main.go
func main() {
    // Load configuration
    cfg, err := config.Load()
    if err != nil {
        log.Fatalf("Failed to load config: %v", err)
    }
    
    // Use configuration
    db, err := sql.Open("postgres", cfg.DatabaseDSN())
    if err != nil {
        log.Fatalf("Failed to connect to database: %v", err)
    }
    defer db.Close()
    
    db.SetMaxOpenConns(cfg.Database.MaxConns)
    db.SetMaxIdleConns(cfg.Database.MaxIdle)
    
    // ... rest of setup
}
```

**.env file for development:**

```env
# Server
SERVER_PORT=8080
ENVIRONMENT=development

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=myapp_dev
DB_SSLMODE=disable

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=your-secret-key-here
JWT_EXPIRATION=24h

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USERNAME=your-email@gmail.com
EMAIL_PASSWORD=your-app-password
EMAIL_FROM=noreply@myapp.com
```

---

## 5.7 Error Handling Strategies

### Custom Error Types

```go
// internal/apperrors/errors.go
package apperrors

import (
    "fmt"
    "net/http"
)

// AppError represents an application error
type AppError struct {
    Code    string // Error code for clients
    Message string // User-friendly message
    Err     error  // Underlying error
    Status  int    // HTTP status code
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// Predefined error constructors
func NotFound(message string, err error) *AppError {
    return &AppError{
        Code:    "NOT_FOUND",
        Message: message,
        Err:     err,
        Status:  http.StatusNotFound,
    }
}

func BadRequest(message string, err error) *AppError {
    return &AppError{
        Code:    "BAD_REQUEST",
        Message: message,
        Err:     err,
        Status:  http.StatusBadRequest,
    }
}

func Unauthorized(message string, err error) *AppError {
    return &AppError{
        Code:    "UNAUTHORIZED",
        Message: message,
        Err:     err,
        Status:  http.StatusUnauthorized,
    }
}

func Forbidden(message string, err error) *AppError {
    return &AppError{
        Code:    "FORBIDDEN",
        Message: message,
        Err:     err,
        Status:  http.StatusForbidden,
    }
}

func Conflict(message string, err error) *AppError {
    return &AppError{
        Code:    "CONFLICT",
        Message: message,
        Err:     err,
        Status:  http.StatusConflict,
    }
}

func Internal(message string, err error) *AppError {
    return &AppError{
        Code:    "INTERNAL_ERROR",
        Message: message,
        Err:     err,
        Status:  http.StatusInternalServerError,
    }
}
```

### Error Handler Middleware

```go
// internal/delivery/http/middleware/error_handler.go
package middleware

import (
    "encoding/json"
    "errors"
    "log"
    "net/http"
    
    "github.com/myapp/internal/apperrors"
)

type ErrorResponse struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}

func ErrorHandler(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Create custom response writer to capture errors
        rw := &errorResponseWriter{ResponseWriter: w}
        
        next.ServeHTTP(rw, r)
        
        if rw.err != nil {
            handleError(w, r, rw.err)
        }
    })
}

type errorResponseWriter struct {
    http.ResponseWriter
    err error
}

func (rw *errorResponseWriter) WriteError(err error) {
    rw.err = err
}

func handleError(w http.ResponseWriter, r *http.Request, err error) {
    var appErr *apperrors.AppError
    
    if errors.As(err, &appErr) {
        // Known application error
        response := ErrorResponse{
            Code:    appErr.Code,
            Message: appErr.Message,
        }
        
        // Log internal errors
        if appErr.Status >= 500 {
            log.Printf("Internal error: %v", appErr.Err)
        }
        
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(appErr.Status)
        json.NewEncoder(w).Encode(response)
        return
    }
    
    // Unknown error - log and return generic error
    log.Printf("Unexpected error: %v", err)
    
    response := ErrorResponse{
        Code:    "INTERNAL_ERROR",
        Message: "An unexpected error occurred",
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusInternalServerError)
    json.NewEncoder(w).Encode(response)
}
```

### Using Errors in Handlers

```go
// internal/delivery/http/user_handler.go
func (h *UserHandler) Get(w http.ResponseWriter, r *http.Request) {
    id := mux.Vars(r)["id"]
    
    user, err := h.userUseCase.GetUser(r.Context(), id)
    if err != nil {
        var appErr *apperrors.AppError
        if errors.As(err, &appErr) {
            respondError(w, appErr.Status, appErr)
            return
        }
        
        // Wrap unknown error
        respondError(w, http.StatusInternalServerError, 
            apperrors.Internal("Failed to get user", err))
        return
    }
    
    respondJSON(w, http.StatusOK, ToUserResponse(user))
}
```

### Error Wrapping in Use Cases

```go
// internal/usecase/user_usecase.go
func (uc *UserUseCase) GetUser(ctx context.Context, id string) (*domain.User, error) {
    user, err := uc.userRepo.GetByID(ctx, id)
    if err != nil {
        if errors.Is(err, domain.ErrUserNotFound) {
            return nil, apperrors.NotFound("User not found", err)
        }
        return nil, apperrors.Internal("Failed to retrieve user", err)
    }
    
    return user, nil
}

func (uc *UserUseCase) RegisterUser(ctx context.Context, email, password, name string) (*domain.User, error) {
    // Check if user exists
    existing, err := uc.userRepo.GetByEmail(ctx, email)
    if err != nil && !errors.Is(err, domain.ErrUserNotFound) {
        return nil, apperrors.Internal("Failed to check existing user", err)
    }
    
    if existing != nil {
        return nil, apperrors.Conflict("User with this email already exists", nil)
    }
    
    // Hash password
    hashedPassword, err := uc.hasher.Hash(password)
    if err != nil {
        return nil, apperrors.Internal("Failed to hash password", err)
    }
    
    // Create user
    user := &domain.User{
        Email:    email,
        Password: hashedPassword,
        Name:     name,
    }
    
    if err := user.Validate(); err != nil {
        return nil, apperrors.BadRequest("Invalid user data", err)
    }
    
    if err := uc.userRepo.Create(ctx, user); err != nil {
        return nil, apperrors.Internal("Failed to create user", err)
    }
    
    return user, nil
}
```

---

## 5.8 Dependency Injection Patterns

### Manual Dependency Injection

```go
// cmd/api/main.go
package main

import (
    "database/sql"
    "log"
    "net/http"
    
    _ "github.com/lib/pq"
    
    "github.com/myapp/internal/delivery/http/handler"
    "github.com/myapp/internal/delivery/http/middleware"
    "github.com/myapp/internal/repository/postgres"
    "github.com/myapp/internal/usecase"
    "github.com/myapp/pkg/bcrypt"
    "github.com/myapp/pkg/config"
)

type Application struct {
    Config *config.Config
    DB     *sql.DB
    Server *http.Server
}

func NewApplication() (*Application, error) {
    // Load configuration
    cfg, err := config.Load()
    if err != nil {
        return nil, err
    }
    
    // Initialize database
    db, err := sql.Open("postgres", cfg.DatabaseDSN())
    if err != nil {
        return nil, err
    }
    
    db.SetMaxOpenConns(cfg.Database.MaxConns)
    db.SetMaxIdleConns(cfg.Database.MaxIdle)
    
    // Run migrations
    if err := migrations.RunMigrations(db); err != nil {
        return nil, err
    }
    
    // Initialize repositories
    userRepo := postgres.NewUserRepository(db)
    orderRepo := postgres.NewOrderRepository(db)
    
    // Initialize services
    hasher := bcrypt.NewBcryptHasher()
    
    // Initialize use cases
    userUseCase := usecase.NewUserUseCase(userRepo, hasher)
    orderUseCase := usecase.NewOrderUseCase(orderRepo, userRepo)
    
    // Initialize handlers
    userHandler := handler.NewUserHandler(userUseCase)
    orderHandler := handler.NewOrderHandler(orderUseCase)
    
    // Setup router
    router := setupRouter(userHandler, orderHandler)
    
    // Create server
    server := &http.Server{
        Addr:         fmt.Sprintf(":%d", cfg.Server.Port),
        Handler:      router,
        ReadTimeout:  cfg.Server.ReadTimeout,
        WriteTimeout: cfg.Server.WriteTimeout,
    }
    
    return &Application{
        Config: cfg,
        DB:     db,
        Server: server,
    }, nil
}

func (app *Application) Run() error {
    log.Printf("Server starting on %s", app.Server.Addr)
    return app.Server.ListenAndServe()
}

func (app *Application) Shutdown(ctx context.Context) error {
    log.Println("Shutting down server...")
    
    if err := app.Server.Shutdown(ctx); err != nil {
        return err
    }
    
    if err := app.DB.Close(); err != nil {
        return err
    }
    
    return nil
}

func setupRouter(userHandler *handler.UserHandler, orderHandler *handler.OrderHandler) http.Handler {
    r := mux.NewRouter()
    
    // API routes
    api := r.PathPrefix("/api/v1").Subrouter()
    
    // User routes
    api.HandleFunc("/users", userHandler.Create).Methods("POST")
    api.HandleFunc("/users/{id}", userHandler.Get).Methods("GET")
    api.HandleFunc("/users", userHandler.List).Methods("GET")
    
    // Order routes
    api.HandleFunc("/orders", orderHandler.Create).Methods("POST")
    api.HandleFunc("/orders/{id}", orderHandler.Get).Methods("GET")
    
    // Apply middleware
    var handler http.Handler = r
    handler = middleware.Recovery(handler)
    handler = middleware.Logging(handler)
    handler = middleware.CORS(handler)
    
    return handler
}

func main() {
    app, err := NewApplication()
    if err != nil {
        log.Fatalf("Failed to create application: %v", err)
    }
    
    // Graceful shutdown
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    go func() {
        sigCh := make(chan os.Signal, 1)
        signal.Notify(sigCh, os.Interrupt, syscall.SIGTERM)
        <-sigCh
        
        shutdownCtx, shutdownCancel := context.WithTimeout(ctx, 10*time.Second)
        defer shutdownCancel()
        
        if err := app.Shutdown(shutdownCtx); err != nil {
            log.Printf("Shutdown error: %v", err)
        }
        
        cancel()
    }()
    
    if err := app.Run(); err != nil && err != http.ErrServerClosed {
        log.Fatalf("Server error: %v", err)
    }
    
    log.Println("Server stopped")
}
```

### Using Wire for Dependency Injection

```go
// cmd/api/wire.go
//go:build wireinject
// +build wireinject

package main

import (
    "github.com/google/wire"
    
    "github.com/myapp/internal/delivery/http/handler"
    "github.com/myapp/internal/repository/postgres"
    "github.com/myapp/internal/usecase"
    "github.com/myapp/pkg/bcrypt"
    "github.com/myapp/pkg/config"
)

func InitializeApplication() (*Application, error) {
    wire.Build(
        // Config
        config.Load,
        
        // Database
        provideDatabase,
        
        // Repositories
        postgres.NewUserRepository,
        postgres.NewOrderRepository,
        
        // Services
        bcrypt.NewBcryptHasher,
        
        // Use cases
        usecase.NewUserUseCase,
        usecase.NewOrderUseCase,
        
        // Handlers
        handler.NewUserHandler,
        handler.NewOrderHandler,
        
        // Application
        NewApplication,
    )
    
    return &Application{}, nil
}

func provideDatabase(cfg *config.Config) (*sql.DB, error) {
    db, err := sql.Open("postgres", cfg.DatabaseDSN())
    if err != nil {
        return nil, err
    }
    
    db.SetMaxOpenConns(cfg.Database.MaxConns)
    db.SetMaxIdleConns(cfg.Database.MaxIdle)
    
    return db, nil
}
```

```bash
# Generate wire code
wire ./cmd/api
```

Continuing with Part 5...


## 5.9 Microservices Communication Patterns

### Service Discovery

```go
// pkg/discovery/consul.go
package discovery

import (
    "fmt"
    
    consul "github.com/hashicorp/consul/api"
)

type ServiceRegistry interface {
    Register(service *ServiceInfo) error
    Deregister(serviceID string) error
    Discover(serviceName string) ([]*ServiceInfo, error)
}

type ServiceInfo struct {
    ID      string
    Name    string
    Address string
    Port    int
    Tags    []string
}

type ConsulRegistry struct {
    client *consul.Client
}

func NewConsulRegistry(address string) (*ConsulRegistry, error) {
    config := consul.DefaultConfig()
    config.Address = address
    
    client, err := consul.NewClient(config)
    if err != nil {
        return nil, err
    }
    
    return &ConsulRegistry{client: client}, nil
}

func (r *ConsulRegistry) Register(service *ServiceInfo) error {
    registration := &consul.AgentServiceRegistration{
        ID:      service.ID,
        Name:    service.Name,
        Address: service.Address,
        Port:    service.Port,
        Tags:    service.Tags,
        Check: &consul.AgentServiceCheck{
            HTTP:     fmt.Sprintf("http://%s:%d/health", service.Address, service.Port),
            Interval: "10s",
            Timeout:  "3s",
        },
    }
    
    return r.client.Agent().ServiceRegister(registration)
}

func (r *ConsulRegistry) Deregister(serviceID string) error {
    return r.client.Agent().ServiceDeregister(serviceID)
}

func (r *ConsulRegistry) Discover(serviceName string) ([]*ServiceInfo, error) {
    services, _, err := r.client.Health().Service(serviceName, "", true, nil)
    if err != nil {
        return nil, err
    }
    
    var result []*ServiceInfo
    for _, service := range services {
        result = append(result, &ServiceInfo{
            ID:      service.Service.ID,
            Name:    service.Service.Service,
            Address: service.Service.Address,
            Port:    service.Service.Port,
            Tags:    service.Service.Tags,
        })
    }
    
    return result, nil
}
```

### Circuit Breaker Pattern

```go
// pkg/circuitbreaker/breaker.go
package circuitbreaker

import (
    "errors"
    "sync"
    "time"
)

var (
    ErrCircuitOpen = errors.New("circuit breaker is open")
)

type State int

const (
    StateClosed State = iota
    StateOpen
    StateHalfOpen
)

type CircuitBreaker struct {
    maxFailures  int
    timeout      time.Duration
    state        State
    failures     int
    lastFailTime time.Time
    mu           sync.RWMutex
}

func New(maxFailures int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures: maxFailures,
        timeout:     timeout,
        state:       StateClosed,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    
    // Check if we can transition from Open to HalfOpen
    if cb.state == StateOpen && time.Since(cb.lastFailTime) > cb.timeout {
        cb.state = StateHalfOpen
    }
    
    // Reject if circuit is open
    if cb.state == StateOpen {
        cb.mu.Unlock()
        return ErrCircuitOpen
    }
    
    cb.mu.Unlock()
    
    // Execute function
    err := fn()
    
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if err != nil {
        cb.failures++
        cb.lastFailTime = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = StateOpen
        }
        
        return err
    }
    
    // Success - reset failures and close circuit
    if cb.state == StateHalfOpen {
        cb.state = StateClosed
    }
    cb.failures = 0
    
    return nil
}

func (cb *CircuitBreaker) State() State {
    cb.mu.RLock()
    defer cb.mu.RUnlock()
    return cb.state
}
```

**Usage:**

```go
// internal/client/user_client.go
package client

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    
    "github.com/myapp/pkg/circuitbreaker"
)

type UserClient struct {
    baseURL string
    client  *http.Client
    breaker *circuitbreaker.CircuitBreaker
}

func NewUserClient(baseURL string) *UserClient {
    return &UserClient{
        baseURL: baseURL,
        client:  &http.Client{},
        breaker: circuitbreaker.New(5, 30*time.Second),
    }
}

func (c *UserClient) GetUser(ctx context.Context, id string) (*User, error) {
    var user User
    
    err := c.breaker.Call(func() error {
        url := fmt.Sprintf("%s/users/%s", c.baseURL, id)
        
        req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
        if err != nil {
            return err
        }
        
        resp, err := c.client.Do(req)
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        
        if resp.StatusCode != http.StatusOK {
            return fmt.Errorf("unexpected status: %d", resp.StatusCode)
        }
        
        return json.NewDecoder(resp.Body).Decode(&user)
    })
    
    if err != nil {
        return nil, err
    }
    
    return &user, nil
}
```

### Message Queue Integration

```go
// pkg/messaging/rabbitmq.go
package messaging

import (
    "context"
    "encoding/json"
    
    amqp "github.com/rabbitmq/amqp091-go"
)

type MessagePublisher interface {
    Publish(ctx context.Context, topic string, message interface{}) error
}

type MessageConsumer interface {
    Consume(topic string, handler func([]byte) error) error
}

type RabbitMQClient struct {
    conn    *amqp.Connection
    channel *amqp.Channel
}

func NewRabbitMQClient(url string) (*RabbitMQClient, error) {
    conn, err := amqp.Dial(url)
    if err != nil {
        return nil, err
    }
    
    channel, err := conn.Channel()
    if err != nil {
        return nil, err
    }
    
    return &RabbitMQClient{
        conn:    conn,
        channel: channel,
    }, nil
}

func (c *RabbitMQClient) Publish(ctx context.Context, topic string, message interface{}) error {
    // Declare queue
    _, err := c.channel.QueueDeclare(
        topic, // name
        true,  // durable
        false, // delete when unused
        false, // exclusive
        false, // no-wait
        nil,   // arguments
    )
    if err != nil {
        return err
    }
    
    // Marshal message
    body, err := json.Marshal(message)
    if err != nil {
        return err
    }
    
    // Publish message
    return c.channel.PublishWithContext(
        ctx,
        "",    // exchange
        topic, // routing key
        false, // mandatory
        false, // immediate
        amqp.Publishing{
            ContentType: "application/json",
            Body:        body,
        },
    )
}

func (c *RabbitMQClient) Consume(topic string, handler func([]byte) error) error {
    // Declare queue
    _, err := c.channel.QueueDeclare(
        topic,
        true,
        false,
        false,
        false,
        nil,
    )
    if err != nil {
        return err
    }
    
    // Start consuming
    msgs, err := c.channel.Consume(
        topic, // queue
        "",    // consumer
        false, // auto-ack
        false, // exclusive
        false, // no-local
        false, // no-wait
        nil,   // args
    )
    if err != nil {
        return err
    }
    
    // Process messages
    go func() {
        for msg := range msgs {
            if err := handler(msg.Body); err != nil {
                msg.Nack(false, true) // Requeue on error
            } else {
                msg.Ack(false)
            }
        }
    }()
    
    return nil
}

func (c *RabbitMQClient) Close() error {
    if err := c.channel.Close(); err != nil {
        return err
    }
    return c.conn.Close()
}
```

**Event-driven communication:**

```go
// internal/events/user_events.go
package events

type UserCreated struct {
    UserID string `json:"user_id"`
    Email  string `json:"email"`
    Name   string `json:"name"`
}

// Publisher
func (uc *UserUseCase) RegisterUser(ctx context.Context, email, password, name string) (*domain.User, error) {
    // ... create user logic
    
    // Publish event
    event := events.UserCreated{
        UserID: user.ID,
        Email:  user.Email,
        Name:   user.Name,
    }
    
    if err := uc.messagePublisher.Publish(ctx, "user.created", event); err != nil {
        log.Printf("Failed to publish user created event: %v", err)
        // Don't fail the operation
    }
    
    return user, nil
}

// Consumer (in email service)
func StartEmailConsumer(consumer messaging.MessageConsumer, emailService EmailService) error {
    return consumer.Consume("user.created", func(data []byte) error {
        var event events.UserCreated
        if err := json.Unmarshal(data, &event); err != nil {
            return err
        }
        
        return emailService.SendWelcomeEmail(event.Email, event.Name)
    })
}
```

---

## 5.10 Logging and Observability

### Structured Logging

```go
// pkg/logger/logger.go
package logger

import (
    "context"
    "os"
    
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

type Logger interface {
    Debug(msg string, fields ...Field)
    Info(msg string, fields ...Field)
    Warn(msg string, fields ...Field)
    Error(msg string, fields ...Field)
    With(fields ...Field) Logger
    WithContext(ctx context.Context) Logger
}

type Field = zapcore.Field

var (
    String  = zap.String
    Int     = zap.Int
    Error   = zap.Error
    Any     = zap.Any
    Duration = zap.Duration
)

type zapLogger struct {
    logger *zap.Logger
}

func New(environment string) (Logger, error) {
    var config zap.Config
    
    if environment == "production" {
        config = zap.NewProductionConfig()
    } else {
        config = zap.NewDevelopmentConfig()
        config.EncoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder
    }
    
    logger, err := config.Build()
    if err != nil {
        return nil, err
    }
    
    return &zapLogger{logger: logger}, nil
}

func (l *zapLogger) Debug(msg string, fields ...Field) {
    l.logger.Debug(msg, fields...)
}

func (l *zapLogger) Info(msg string, fields ...Field) {
    l.logger.Info(msg, fields...)
}

func (l *zapLogger) Warn(msg string, fields ...Field) {
    l.logger.Warn(msg, fields...)
}

func (l *zapLogger) Error(msg string, fields ...Field) {
    l.logger.Error(msg, fields...)
}

func (l *zapLogger) With(fields ...Field) Logger {
    return &zapLogger{logger: l.logger.With(fields...)}
}

func (l *zapLogger) WithContext(ctx context.Context) Logger {
    // Extract request ID from context
    if requestID, ok := ctx.Value("request_id").(string); ok {
        return l.With(String("request_id", requestID))
    }
    return l
}
```

**Usage:**

```go
// internal/usecase/user_usecase.go
type UserUseCase struct {
    userRepo UserRepository
    hasher   PasswordHasher
    logger   logger.Logger
}

func (uc *UserUseCase) RegisterUser(ctx context.Context, email, password, name string) (*domain.User, error) {
    log := uc.logger.WithContext(ctx)
    
    log.Info("Registering new user",
        logger.String("email", email),
        logger.String("name", name),
    )
    
    user, err := uc.userRepo.Create(ctx, &domain.User{
        Email: email,
        Name:  name,
    })
    
    if err != nil {
        log.Error("Failed to create user",
            logger.String("email", email),
            logger.Error(err),
        )
        return nil, err
    }
    
    log.Info("User registered successfully",
        logger.String("user_id", user.ID),
    )
    
    return user, nil
}
```

### Request Tracing

```go
// internal/delivery/http/middleware/tracing.go
package middleware

import (
    "context"
    "net/http"
    
    "github.com/google/uuid"
)

func RequestID(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestID := r.Header.Get("X-Request-ID")
        if requestID == "" {
            requestID = uuid.New().String()
        }
        
        // Add to context
        ctx := context.WithValue(r.Context(), "request_id", requestID)
        
        // Add to response header
        w.Header().Set("X-Request-ID", requestID)
        
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

### Metrics with Prometheus

```go
// pkg/metrics/prometheus.go
package metrics

import (
    "net/http"
    
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "path", "status"},
    )
    
    httpRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path"},
    )
    
    dbQueriesTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "db_queries_total",
            Help: "Total number of database queries",
        },
        []string{"operation", "table"},
    )
)

func MetricsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        timer := prometheus.NewTimer(httpRequestDuration.WithLabelValues(r.Method, r.URL.Path))
        defer timer.ObserveDuration()
        
        wrapped := &responseWriter{ResponseWriter: w, statusCode: http.StatusOK}
        next.ServeHTTP(wrapped, r)
        
        httpRequestsTotal.WithLabelValues(
            r.Method,
            r.URL.Path,
            http.StatusText(wrapped.statusCode),
        ).Inc()
    })
}

func Handler() http.Handler {
    return promhttp.Handler()
}

// Record database query
func RecordDBQuery(operation, table string) {
    dbQueriesTotal.WithLabelValues(operation, table).Inc()
}
```

---

## Interview Questions (Architecture)

**Question 1: How do you structure a Go project for maintainability?**

**Difficulty:** Mid-Level

**Answer:**

Use layered architecture with clear separation:

```
project/
├── cmd/              # Entry points
│   └── api/
│       └── main.go
├── internal/         # Private application code
│   ├── domain/       # Business entities
│   ├── usecase/      # Business logic
│   ├── delivery/     # API layer (HTTP, gRPC)
│   └── repository/   # Data access
├── pkg/              # Public libraries
└── api/              # API definitions (proto, openapi)
```

**Key principles:**
- Domain logic independent of infrastructure
- Dependencies point inward (Clean Architecture)
- Interfaces defined at point of use
- Repository pattern for data access
- Dependency injection for testability

**What Interviewers Look For:**
- Understanding of layered architecture
- Separation of concerns
- Testability
- Scalability considerations

---

**Question 2: How do you handle configuration in a Go application?**

**Difficulty:** Mid-Level

**Answer:**

Use environment variables with defaults:

```go
type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
}

func Load() (*Config, error) {
    cfg := &Config{
        Server: ServerConfig{
            Port: getEnvInt("PORT", 8080),
        },
        Database: DatabaseConfig{
            Host: getEnv("DB_HOST", "localhost"),
        },
    }
    
    return cfg, cfg.Validate()
}
```

**Benefits:**
- Environment-specific configuration
- No secrets in code
- Easy to test (override in tests)
- 12-factor app compliance

**What Interviewers Look For:**
- Security awareness (no hardcoded secrets)
- Environment management
- Validation
- Best practices (12-factor)

---

**Question 3: How do you implement graceful shutdown?**

**Difficulty:** Senior

**Answer:**

```go
func main() {
    server := &http.Server{Addr: ":8080"}
    
    // Start server
    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()
    
    // Wait for interrupt
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, os.Interrupt, syscall.SIGTERM)
    <-quit
    
    // Graceful shutdown with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
    
    log.Println("Server exited")
}
```

**What to handle:**
- In-flight requests complete
- Database connections close
- Background workers stop
- Resources cleaned up

**What Interviewers Look For:**
- Production readiness
- Resource management
- Error handling
- Timeout handling

---

## Part 5 Summary and Key Takeaways

### Architectural Patterns Covered

**✅ Clean Architecture:**
- Layered separation (Domain, Use Case, Interface, Infrastructure)
- Dependency rule (dependencies point inward)
- Testable and maintainable

**✅ Domain-Driven Design:**
- Entities, Value Objects, Aggregates
- Bounded Contexts
- Ubiquitous Language
- Domain Events

**✅ Hexagonal Architecture:**
- Ports (interfaces) and Adapters (implementations)
- Core business logic isolation
- Pluggable infrastructure

**✅ API Design:**
- RESTful patterns
- gRPC integration
- Middleware composition
- Request/Response DTOs
- Validation and error handling

**✅ Database Patterns:**
- Repository pattern
- Transaction management
- Query builders
- Migrations

**✅ Configuration:**
- Environment-based config
- Validation
- Type safety
- Secrets management

**✅ Error Handling:**
- Custom error types
- Error wrapping
- HTTP error mapping
- Logging

**✅ Dependency Injection:**
- Manual injection
- Wire for automation
- Application composition

**✅ Microservices:**
- Service discovery
- Circuit breaker
- Message queues
- Event-driven architecture

**✅ Observability:**
- Structured logging
- Request tracing
- Metrics (Prometheus)
- Health checks

### Best Practices

1. **Separation of Concerns**: Keep layers independent
2. **Dependency Inversion**: Depend on abstractions, not concretions
3. **Interface Segregation**: Small, focused interfaces
4. **Single Responsibility**: Each component has one reason to change
5. **DRY (Don't Repeat Yourself)**: But don't over-abstract
6. **YAGNI (You Aren't Gonna Need It)**: Don't add complexity prematurely
7. **Test at All Levels**: Unit, integration, end-to-end
8. **Monitor Everything**: Logs, metrics, traces
9. **Fail Fast**: Validate early, fail with clear errors
10. **Document Decisions**: ADRs (Architecture Decision Records)

### What's Next

With Parts 1-5 complete (~85,000 words!), you have mastered:
- Go fundamentals and language design
- Concurrency and runtime internals
- Advanced patterns and techniques
- Comprehensive testing strategies
- Production architecture patterns

**Continue your journey:**
- Build real projects using these patterns
- Contribute to open-source Go projects
- Deep dive into specific domains (networking, distributed systems)
- Explore advanced topics (Kubernetes operators, service mesh)
- Stay updated with Go evolution

**You're now equipped to build production-grade Go applications!**

