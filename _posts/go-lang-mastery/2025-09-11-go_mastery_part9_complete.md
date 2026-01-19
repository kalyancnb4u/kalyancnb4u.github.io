---
title: "Complete Go Mastery Part 9: Security Best Practices"
date: 2025-01-11 02:00:00 +0530
categories: [Programming, Go, Security]
tags: [golang, go, security, cryptography, authentication, authorization, owasp, vulnerabilities, secure-coding]
---

# Complete Go Mastery Part 9: Security Best Practices

## Introduction

Welcome to Part 9 of the Complete Go Mastery series. In Parts 1-8, we covered Go fundamentals through performance optimization. Now we focus on **security**—protecting your applications, data, and users from threats.

Security isn't an afterthought—it's a fundamental requirement. A single vulnerability can compromise user data, damage reputation, and result in financial loss. With Go's growing adoption for cloud-native and security-critical applications, understanding security best practices is essential.

**Why security matters:**

- **Data protection**: Safeguard sensitive user information
- **Compliance**: Meet regulatory requirements (GDPR, HIPAA, PCI-DSS)
- **Trust**: Maintain user and customer confidence
- **Business continuity**: Prevent breaches and downtime
- **Legal liability**: Avoid lawsuits and penalties

**What you'll learn in Part 9:**

- **Input Validation**: Preventing injection attacks
- **Authentication**: Secure user identity verification
- **Authorization**: Access control and permissions
- **Cryptography**: Encryption, hashing, and key management
- **HTTPS & TLS**: Secure communication
- **SQL Injection Prevention**: Safe database queries
- **XSS & CSRF Protection**: Web security fundamentals
- **Secrets Management**: Handling sensitive credentials
- **Dependency Security**: Managing third-party risks
- **Security Headers**: HTTP security best practices
- **Rate Limiting**: Preventing abuse
- **Secure Coding Practices**: Common vulnerabilities

**Security Principles:**

> "Security is a process, not a product" - Bruce Schneier

Follow these principles:
1. **Defense in depth** - Multiple layers of security
2. **Least privilege** - Minimal necessary access
3. **Fail securely** - Default to secure state
4. **Never trust input** - Validate everything
5. **Keep it simple** - Complexity breeds vulnerabilities

By the end of Part 9, you'll know how to build secure Go applications that protect against common vulnerabilities and follow industry best practices.

---

## 9.1 Input Validation & Sanitization

### Understanding Input Validation

**Why validate input?**
- Prevent injection attacks (SQL, command, code)
- Ensure data integrity
- Prevent application crashes
- Enforce business rules

### String Validation

```go
import (
    "errors"
    "net/mail"
    "regexp"
    "unicode/utf8"
)

// Email validation
func validateEmail(email string) error {
    if email == "" {
        return errors.New("email is required")
    }
    
    if len(email) > 254 {
        return errors.New("email too long")
    }
    
    _, err := mail.ParseAddress(email)
    if err != nil {
        return errors.New("invalid email format")
    }
    
    return nil
}

// Username validation
func validateUsername(username string) error {
    if username == "" {
        return errors.New("username is required")
    }
    
    if len(username) < 3 || len(username) > 20 {
        return errors.New("username must be 3-20 characters")
    }
    
    // Only alphanumeric and underscore
    matched, _ := regexp.MatchString("^[a-zA-Z0-9_]+$", username)
    if !matched {
        return errors.New("username can only contain letters, numbers, and underscore")
    }
    
    return nil
}

// URL validation
func validateURL(urlStr string) error {
    u, err := url.Parse(urlStr)
    if err != nil {
        return errors.New("invalid URL format")
    }
    
    // Only allow HTTP/HTTPS
    if u.Scheme != "http" && u.Scheme != "https" {
        return errors.New("only HTTP/HTTPS URLs allowed")
    }
    
    // Prevent SSRF by blocking private IPs
    if isPrivateIP(u.Hostname()) {
        return errors.New("private IP addresses not allowed")
    }
    
    return nil
}

func isPrivateIP(host string) bool {
    ip := net.ParseIP(host)
    if ip == nil {
        return false
    }
    
    return ip.IsPrivate() || ip.IsLoopback() || ip.IsLinkLocalUnicast()
}
```

### Struct Validation with Tags

```go
import "github.com/go-playground/validator/v10"

type User struct {
    Email    string `validate:"required,email,max=254"`
    Username string `validate:"required,min=3,max=20,alphanum"`
    Age      int    `validate:"required,min=18,max=120"`
    Password string `validate:"required,min=8,max=72"`
}

func validateUser(user User) error {
    validate := validator.New()
    return validate.Struct(user)
}

// Custom validation
func validatePassword(fl validator.FieldLevel) bool {
    password := fl.Field().String()
    
    // At least one uppercase, one lowercase, one digit
    hasUpper := regexp.MustCompile(`[A-Z]`).MatchString(password)
    hasLower := regexp.MustCompile(`[a-z]`).MatchString(password)
    hasDigit := regexp.MustCompile(`[0-9]`).MatchString(password)
    
    return hasUpper && hasLower && hasDigit
}

// Register custom validator
validate := validator.New()
validate.RegisterValidation("password", validatePassword)
```

### Path Traversal Prevention

```go
import "path/filepath"

// ❌ DANGEROUS: Vulnerable to path traversal
func serveFileDangerous(filename string) ([]byte, error) {
    path := filepath.Join("/var/www/", filename)
    return os.ReadFile(path)
}

// Attacker could use: ../../etc/passwd

// ✅ SAFE: Validate and sanitize path
func serveFileSafe(filename string) ([]byte, error) {
    // Clean the path
    filename = filepath.Clean(filename)
    
    // Prevent directory traversal
    if strings.Contains(filename, "..") {
        return nil, errors.New("invalid filename")
    }
    
    // Ensure path is within allowed directory
    basePath := "/var/www/"
    fullPath := filepath.Join(basePath, filename)
    
    // Check if path is still within base directory
    if !strings.HasPrefix(fullPath, basePath) {
        return nil, errors.New("invalid path")
    }
    
    return os.ReadFile(fullPath)
}
```

### HTML Sanitization

```go
import (
    "html"
    "html/template"
)

// Escape HTML to prevent XSS
func sanitizeHTML(input string) string {
    return html.EscapeString(input)
}

// Using templates (automatic escaping)
func renderTemplate(data map[string]interface{}) (string, error) {
    tmpl := template.Must(template.New("page").Parse(`
        <h1>{{.Title}}</h1>
        <p>{{.Content}}</p>
    `))
    
    var buf bytes.Buffer
    err := tmpl.Execute(&buf, data)
    return buf.String(), err
}

// For rich text, use a sanitization library
import "github.com/microcosm-cc/bluemonday"

func sanitizeRichText(input string) string {
    p := bluemonday.StrictPolicy()
    return p.Sanitize(input)
}

// Allow specific tags
func sanitizeWithPolicy(input string) string {
    p := bluemonday.NewPolicy()
    p.AllowElements("p", "br", "strong", "em")
    p.AllowAttrs("href").OnElements("a")
    return p.Sanitize(input)
}
```

### Command Injection Prevention

```go
import (
    "os/exec"
    "strings"
)

// ❌ DANGEROUS: Vulnerable to command injection
func runCommandDangerous(userInput string) error {
    cmd := exec.Command("sh", "-c", "ls "+userInput)
    return cmd.Run()
}

// Attacker could use: "; rm -rf /"

// ✅ SAFE: Use proper command construction
func runCommandSafe(userInput string) error {
    // Validate input
    if strings.ContainsAny(userInput, ";|&$`") {
        return errors.New("invalid characters in input")
    }
    
    // Use separate arguments
    cmd := exec.Command("ls", userInput)
    return cmd.Run()
}

// ✅ BETTER: Use allowlist
func runCommandAllowlist(userInput string) error {
    allowed := map[string]bool{
        "docs":      true,
        "downloads": true,
        "projects":  true,
    }
    
    if !allowed[userInput] {
        return errors.New("directory not allowed")
    }
    
    cmd := exec.Command("ls", filepath.Join("/home/user", userInput))
    return cmd.Run()
}
```

---

## 9.2 Authentication

### Password Hashing with bcrypt

```go
import "golang.org/x/crypto/bcrypt"

// Hash password
func hashPassword(password string) (string, error) {
    // Use appropriate cost (10-12 for production)
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return "", err
    }
    return string(hash), nil
}

// Verify password
func verifyPassword(hashedPassword, password string) error {
    return bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(password))
}

// ❌ NEVER store plain text passwords
func badPasswordStorage(password string) string {
    return password  // DON'T DO THIS!
}

// ❌ NEVER use weak hashing
import "crypto/md5"  // DON'T USE MD5 for passwords!
import "crypto/sha1" // DON'T USE SHA1 for passwords!
```

### JWT Authentication

```go
import (
    "github.com/golang-jwt/jwt/v5"
    "time"
)

type Claims struct {
    UserID   string `json:"user_id"`
    Username string `json:"username"`
    jwt.RegisteredClaims
}

// Generate JWT token
func generateToken(userID, username string, secret []byte) (string, error) {
    claims := Claims{
        UserID:   userID,
        Username: username,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            NotBefore: jwt.NewNumericDate(time.Now()),
            Issuer:    "myapp",
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secret)
}

// Validate JWT token
func validateToken(tokenString string, secret []byte) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        // Validate signing method
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
        }
        return secret, nil
    })
    
    if err != nil {
        return nil, err
    }
    
    if claims, ok := token.Claims.(*Claims); ok && token.Valid {
        return claims, nil
    }
    
    return nil, errors.New("invalid token")
}

// Middleware for JWT authentication
func authMiddleware(secret []byte) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            authHeader := r.Header.Get("Authorization")
            if authHeader == "" {
                http.Error(w, "missing authorization header", http.StatusUnauthorized)
                return
            }
            
            // Extract token
            parts := strings.Split(authHeader, " ")
            if len(parts) != 2 || parts[0] != "Bearer" {
                http.Error(w, "invalid authorization header", http.StatusUnauthorized)
                return
            }
            
            // Validate token
            claims, err := validateToken(parts[1], secret)
            if err != nil {
                http.Error(w, "invalid token", http.StatusUnauthorized)
                return
            }
            
            // Add claims to context
            ctx := context.WithValue(r.Context(), "user_id", claims.UserID)
            ctx = context.WithValue(ctx, "username", claims.Username)
            
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}
```

### Session Management

```go
import (
    "crypto/rand"
    "encoding/base64"
    "sync"
    "time"
)

type Session struct {
    ID        string
    UserID    string
    CreatedAt time.Time
    ExpiresAt time.Time
}

type SessionStore struct {
    sessions map[string]*Session
    mu       sync.RWMutex
}

func NewSessionStore() *SessionStore {
    store := &SessionStore{
        sessions: make(map[string]*Session),
    }
    
    // Cleanup expired sessions
    go func() {
        ticker := time.NewTicker(time.Hour)
        defer ticker.Stop()
        for range ticker.C {
            store.cleanup()
        }
    }()
    
    return store
}

func (s *SessionStore) Create(userID string) (*Session, error) {
    sessionID, err := generateSessionID()
    if err != nil {
        return nil, err
    }
    
    session := &Session{
        ID:        sessionID,
        UserID:    userID,
        CreatedAt: time.Now(),
        ExpiresAt: time.Now().Add(24 * time.Hour),
    }
    
    s.mu.Lock()
    s.sessions[sessionID] = session
    s.mu.Unlock()
    
    return session, nil
}

func (s *SessionStore) Get(sessionID string) (*Session, bool) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    session, ok := s.sessions[sessionID]
    if !ok {
        return nil, false
    }
    
    if time.Now().After(session.ExpiresAt) {
        return nil, false
    }
    
    return session, true
}

func (s *SessionStore) Delete(sessionID string) {
    s.mu.Lock()
    defer s.mu.Unlock()
    delete(s.sessions, sessionID)
}

func (s *SessionStore) cleanup() {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    now := time.Now()
    for id, session := range s.sessions {
        if now.After(session.ExpiresAt) {
            delete(s.sessions, id)
        }
    }
}

func generateSessionID() (string, error) {
    b := make([]byte, 32)
    if _, err := rand.Read(b); err != nil {
        return "", err
    }
    return base64.URLEncoding.EncodeToString(b), nil
}

// Session cookie
func setSessionCookie(w http.ResponseWriter, sessionID string) {
    cookie := &http.Cookie{
        Name:     "session_id",
        Value:    sessionID,
        Path:     "/",
        HttpOnly: true,  // Prevent JavaScript access
        Secure:   true,  // Only send over HTTPS
        SameSite: http.SameSiteStrictMode,
        MaxAge:   86400, // 24 hours
    }
    http.SetCookie(w, cookie)
}
```

### Two-Factor Authentication (TOTP)

```go
import "github.com/pquerna/otp/totp"

// Generate TOTP secret
func generateTOTPSecret(userEmail string) (string, error) {
    key, err := totp.Generate(totp.GenerateOpts{
        Issuer:      "MyApp",
        AccountName: userEmail,
    })
    if err != nil {
        return "", err
    }
    
    return key.Secret(), nil
}

// Verify TOTP code
func verifyTOTP(secret, code string) bool {
    return totp.Validate(code, secret)
}

// Generate QR code for secret
func generateQRCode(secret, userEmail string) (string, error) {
    key, err := totp.Generate(totp.GenerateOpts{
        Issuer:      "MyApp",
        AccountName: userEmail,
        Secret:      []byte(secret),
    })
    if err != nil {
        return "", err
    }
    
    return key.URL(), nil
}
```

Continuing with Part 9...
## 9.3 Authorization & Access Control

### Role-Based Access Control (RBAC)

```go
type Role string

const (
    RoleAdmin  Role = "admin"
    RoleEditor Role = "editor"
    RoleViewer Role = "viewer"
)

type Permission string

const (
    PermissionRead   Permission = "read"
    PermissionWrite  Permission = "write"
    PermissionDelete Permission = "delete"
)

// Role permissions map
var rolePermissions = map[Role][]Permission{
    RoleAdmin:  {PermissionRead, PermissionWrite, PermissionDelete},
    RoleEditor: {PermissionRead, PermissionWrite},
    RoleViewer: {PermissionRead},
}

type User struct {
    ID    string
    Name  string
    Roles []Role
}

func (u *User) HasPermission(perm Permission) bool {
    for _, role := range u.Roles {
        if perms, ok := rolePermissions[role]; ok {
            for _, p := range perms {
                if p == perm {
                    return true
                }
            }
        }
    }
    return false
}

// Middleware for permission checking
func requirePermission(perm Permission) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            user := getUserFromContext(r.Context())
            if user == nil {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }
            
            if !user.HasPermission(perm) {
                http.Error(w, "Forbidden", http.StatusForbidden)
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

// Usage
http.Handle("/api/users", requirePermission(PermissionRead)(usersHandler))
http.Handle("/api/users/create", requirePermission(PermissionWrite)(createUserHandler))
http.Handle("/api/users/delete", requirePermission(PermissionDelete)(deleteUserHandler))
```

### Attribute-Based Access Control (ABAC)

```go
type Resource struct {
    Type    string
    ID      string
    OwnerID string
}

type AccessPolicy struct {
    Subject  string // User ID or role
    Resource string // Resource type
    Action   string // read, write, delete
    Condition func(*User, *Resource) bool
}

type AccessControl struct {
    policies []AccessPolicy
}

func NewAccessControl() *AccessControl {
    return &AccessControl{
        policies: []AccessPolicy{
            // Owner can read/write/delete their resources
            {
                Subject:  "*",
                Resource: "*",
                Action:   "read",
                Condition: func(u *User, r *Resource) bool {
                    return u.ID == r.OwnerID
                },
            },
            {
                Subject:  "*",
                Resource: "*",
                Action:   "write",
                Condition: func(u *User, r *Resource) bool {
                    return u.ID == r.OwnerID
                },
            },
            // Admin can do everything
            {
                Subject:  "admin",
                Resource: "*",
                Action:   "*",
                Condition: func(u *User, r *Resource) bool {
                    for _, role := range u.Roles {
                        if role == RoleAdmin {
                            return true
                        }
                    }
                    return false
                },
            },
        },
    }
}

func (ac *AccessControl) CanAccess(user *User, resource *Resource, action string) bool {
    for _, policy := range ac.policies {
        if ac.matchesPolicy(user, resource, action, policy) {
            return true
        }
    }
    return false
}

func (ac *AccessControl) matchesPolicy(user *User, resource *Resource, action string, policy AccessPolicy) bool {
    // Match resource
    if policy.Resource != "*" && policy.Resource != resource.Type {
        return false
    }
    
    // Match action
    if policy.Action != "*" && policy.Action != action {
        return false
    }
    
    // Check condition
    if policy.Condition != nil && !policy.Condition(user, resource) {
        return false
    }
    
    return true
}
```

---

## 9.4 Cryptography

### Encryption with AES

```go
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/base64"
    "io"
)

// Encrypt data with AES-GCM
func encrypt(plaintext []byte, key []byte) (string, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return "", err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return "", err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return "", err
    }
    
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

// Decrypt data with AES-GCM
func decrypt(ciphertextBase64 string, key []byte) ([]byte, error) {
    ciphertext, err := base64.StdEncoding.DecodeString(ciphertextBase64)
    if err != nil {
        return nil, err
    }
    
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonceSize := gcm.NonceSize()
    if len(ciphertext) < nonceSize {
        return nil, errors.New("ciphertext too short")
    }
    
    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    return gcm.Open(nil, nonce, ciphertext, nil)
}

// Generate encryption key
func generateKey() ([]byte, error) {
    key := make([]byte, 32) // AES-256
    if _, err := rand.Read(key); err != nil {
        return nil, err
    }
    return key, nil
}
```

### Hashing

```go
import (
    "crypto/sha256"
    "crypto/subtle"
    "encoding/hex"
)

// Hash data with SHA-256
func hashSHA256(data []byte) string {
    hash := sha256.Sum256(data)
    return hex.EncodeToString(hash[:])
}

// Constant-time comparison (prevent timing attacks)
func compareHashes(hash1, hash2 string) bool {
    return subtle.ConstantTimeCompare([]byte(hash1), []byte(hash2)) == 1
}

// HMAC for message authentication
import "crypto/hmac"

func generateHMAC(message, key []byte) string {
    h := hmac.New(sha256.New, key)
    h.Write(message)
    return hex.EncodeToString(h.Sum(nil))
}

func verifyHMAC(message, key []byte, messageMAC string) bool {
    expectedMAC := generateHMAC(message, key)
    return subtle.ConstantTimeCompare([]byte(messageMAC), []byte(expectedMAC)) == 1
}
```

### Public Key Cryptography (RSA)

```go
import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "crypto/x509"
    "encoding/pem"
)

// Generate RSA key pair
func generateRSAKeyPair(bits int) (*rsa.PrivateKey, error) {
    return rsa.GenerateKey(rand.Reader, bits)
}

// Encrypt with RSA public key
func encryptRSA(plaintext []byte, publicKey *rsa.PublicKey) ([]byte, error) {
    return rsa.EncryptOAEP(sha256.New(), rand.Reader, publicKey, plaintext, nil)
}

// Decrypt with RSA private key
func decryptRSA(ciphertext []byte, privateKey *rsa.PrivateKey) ([]byte, error) {
    return rsa.DecryptOAEP(sha256.New(), rand.Reader, privateKey, ciphertext, nil)
}

// Sign data
func signRSA(data []byte, privateKey *rsa.PrivateKey) ([]byte, error) {
    hash := sha256.Sum256(data)
    return rsa.SignPKCS1v15(rand.Reader, privateKey, crypto.SHA256, hash[:])
}

// Verify signature
func verifyRSA(data, signature []byte, publicKey *rsa.PublicKey) error {
    hash := sha256.Sum256(data)
    return rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, hash[:], signature)
}

// Export private key to PEM
func exportPrivateKeyPEM(key *rsa.PrivateKey) []byte {
    keyBytes := x509.MarshalPKCS1PrivateKey(key)
    return pem.EncodeToMemory(&pem.Block{
        Type:  "RSA PRIVATE KEY",
        Bytes: keyBytes,
    })
}

// Import private key from PEM
func importPrivateKeyPEM(pemData []byte) (*rsa.PrivateKey, error) {
    block, _ := pem.Decode(pemData)
    if block == nil {
        return nil, errors.New("failed to decode PEM block")
    }
    
    return x509.ParsePKCS1PrivateKey(block.Bytes)
}
```

---

## 9.5 SQL Injection Prevention

### Parameterized Queries

```go
import "database/sql"

// ❌ DANGEROUS: String concatenation (SQL injection)
func getUserBad(db *sql.DB, username string) (*User, error) {
    query := "SELECT id, username FROM users WHERE username = '" + username + "'"
    // Attacker could use: ' OR '1'='1
    
    var user User
    err := db.QueryRow(query).Scan(&user.ID, &user.Username)
    return &user, err
}

// ✅ SAFE: Parameterized query
func getUserGood(db *sql.DB, username string) (*User, error) {
    query := "SELECT id, username FROM users WHERE username = $1"
    
    var user User
    err := db.QueryRow(query, username).Scan(&user.ID, &user.Username)
    return &user, err
}

// ✅ SAFE: Named parameters with sqlx
import "github.com/jmoiron/sqlx"

func getUserNamed(db *sqlx.DB, username string) (*User, error) {
    query := "SELECT id, username FROM users WHERE username = :username"
    
    var user User
    err := db.Get(&user, query, map[string]interface{}{
        "username": username,
    })
    return &user, err
}
```

### Safe Dynamic Queries

```go
// ❌ DANGEROUS: Dynamic column names
func searchBad(db *sql.DB, column, value string) error {
    query := fmt.Sprintf("SELECT * FROM users WHERE %s = ?", column)
    // Attacker could inject: "id = 1 OR 1=1 --"
    
    _, err := db.Query(query, value)
    return err
}

// ✅ SAFE: Allowlist for column names
func searchGood(db *sql.DB, column, value string) error {
    allowedColumns := map[string]bool{
        "username": true,
        "email":    true,
        "status":   true,
    }
    
    if !allowedColumns[column] {
        return errors.New("invalid column")
    }
    
    // Safe to use column name now
    query := fmt.Sprintf("SELECT * FROM users WHERE %s = $1", column)
    _, err := db.Query(query, value)
    return err
}

// ✅ SAFE: Query builder with validation
type QueryBuilder struct {
    table   string
    columns []string
    where   []string
    args    []interface{}
}

func NewQueryBuilder(table string) *QueryBuilder {
    allowedTables := map[string]bool{
        "users":  true,
        "posts":  true,
        "orders": true,
    }
    
    if !allowedTables[table] {
        panic("invalid table")
    }
    
    return &QueryBuilder{table: table}
}

func (qb *QueryBuilder) Where(column, operator string, value interface{}) *QueryBuilder {
    allowedColumns := map[string]bool{
        "id": true, "username": true, "email": true, "status": true,
    }
    allowedOperators := map[string]bool{
        "=": true, ">": true, "<": true, "LIKE": true,
    }
    
    if !allowedColumns[column] || !allowedOperators[operator] {
        panic("invalid column or operator")
    }
    
    qb.where = append(qb.where, fmt.Sprintf("%s %s $%d", column, operator, len(qb.args)+1))
    qb.args = append(qb.args, value)
    return qb
}

func (qb *QueryBuilder) Build() (string, []interface{}) {
    query := fmt.Sprintf("SELECT * FROM %s", qb.table)
    if len(qb.where) > 0 {
        query += " WHERE " + strings.Join(qb.where, " AND ")
    }
    return query, qb.args
}
```

---

## 9.6 Web Security (XSS, CSRF, CORS)

### XSS Prevention

```go
import "html/template"

// ✅ Use html/template for automatic escaping
func renderTemplate(w http.ResponseWriter, data interface{}) error {
    tmpl := template.Must(template.New("page").Parse(`
        <!DOCTYPE html>
        <html>
        <head><title>{{.Title}}</title></head>
        <body>
            <h1>{{.Title}}</h1>
            <p>{{.Content}}</p>
        </body>
        </html>
    `))
    
    return tmpl.Execute(w, data)
}

// ❌ Don't use text/template for HTML
import "text/template"  // NO automatic escaping!

// Content Security Policy header
func setCSPHeaders(w http.ResponseWriter) {
    w.Header().Set("Content-Security-Policy",
        "default-src 'self'; "+
        "script-src 'self'; "+
        "style-src 'self' 'unsafe-inline'; "+
        "img-src 'self' data: https:; "+
        "font-src 'self'; "+
        "connect-src 'self'; "+
        "frame-ancestors 'none'",
    )
}
```

### CSRF Protection

```go
import (
    "crypto/rand"
    "encoding/base64"
    "time"
)

type CSRFToken struct {
    Token     string
    ExpiresAt time.Time
}

// Generate CSRF token
func generateCSRFToken() (string, error) {
    b := make([]byte, 32)
    if _, err := rand.Read(b); err != nil {
        return "", err
    }
    return base64.URLEncoding.EncodeToString(b), nil
}

// CSRF middleware
func csrfMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if r.Method == "GET" || r.Method == "HEAD" || r.Method == "OPTIONS" {
            next.ServeHTTP(w, r)
            return
        }
        
        // Get token from session
        sessionToken := getCSRFTokenFromSession(r)
        
        // Get token from request
        requestToken := r.Header.Get("X-CSRF-Token")
        if requestToken == "" {
            requestToken = r.FormValue("csrf_token")
        }
        
        // Verify token
        if requestToken != sessionToken {
            http.Error(w, "Invalid CSRF token", http.StatusForbidden)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// Using gorilla/csrf
import "github.com/gorilla/csrf"

func setupCSRF() http.Handler {
    CSRF := csrf.Protect(
        []byte("32-byte-long-auth-key"),
        csrf.Secure(true),      // Only HTTPS
        csrf.HttpOnly(true),    // No JavaScript access
        csrf.SameSite(csrf.SameSiteStrictMode),
    )
    
    mux := http.NewServeMux()
    // ... setup routes
    
    return CSRF(mux)
}
```

### CORS Configuration

```go
import "github.com/rs/cors"

func setupCORS() http.Handler {
    c := cors.New(cors.Options{
        AllowedOrigins: []string{
            "https://example.com",
            "https://app.example.com",
        },
        AllowedMethods: []string{
            http.MethodGet,
            http.MethodPost,
            http.MethodPut,
            http.MethodDelete,
        },
        AllowedHeaders: []string{
            "Accept",
            "Authorization",
            "Content-Type",
            "X-CSRF-Token",
        },
        ExposedHeaders: []string{
            "Link",
        },
        AllowCredentials: true,
        MaxAge:           300, // 5 minutes
    })
    
    mux := http.NewServeMux()
    // ... setup routes
    
    return c.Handler(mux)
}

// ❌ Don't use wildcard in production
func badCORS() http.Handler {
    c := cors.New(cors.Options{
        AllowedOrigins: []string{"*"},  // DON'T DO THIS!
        AllowCredentials: true,         // Incompatible with *
    })
    return c.Handler(http.NewServeMux())
}
```

Continuing with Part 9...


## 9.7 HTTPS & TLS

### TLS Server Configuration

```go
import (
    "crypto/tls"
    "net/http"
)

// ✅ Secure TLS configuration
func newTLSConfig() *tls.Config {
    return &tls.Config{
        MinVersion: tls.VersionTLS13,  // Require TLS 1.3
        CipherSuites: []uint16{
            tls.TLS_AES_128_GCM_SHA256,
            tls.TLS_AES_256_GCM_SHA384,
            tls.TLS_CHACHA20_POLY1305_SHA256,
        },
        PreferServerCipherSuites: true,
        CurvePreferences: []tls.CurveID{
            tls.X25519,
            tls.CurveP256,
        },
    }
}

// HTTPS server
func startHTTPSServer(handler http.Handler) error {
    server := &http.Server{
        Addr:      ":443",
        Handler:   handler,
        TLSConfig: newTLSConfig(),
        
        // Timeouts
        ReadTimeout:       10 * time.Second,
        WriteTimeout:      10 * time.Second,
        IdleTimeout:       60 * time.Second,
        ReadHeaderTimeout: 5 * time.Second,
    }
    
    return server.ListenAndServeTLS("cert.pem", "key.pem")
}

// Redirect HTTP to HTTPS
func redirectToHTTPS(w http.ResponseWriter, r *http.Request) {
    target := "https://" + r.Host + r.URL.Path
    if r.URL.RawQuery != "" {
        target += "?" + r.URL.RawQuery
    }
    http.Redirect(w, r, target, http.StatusMovedPermanently)
}

func startHTTPRedirect() error {
    return http.ListenAndServe(":80", http.HandlerFunc(redirectToHTTPS))
}
```

### TLS Client Configuration

```go
// ✅ Secure HTTP client
func newSecureClient() *http.Client {
    return &http.Client{
        Timeout: 30 * time.Second,
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{
                MinVersion: tls.VersionTLS13,
            },
            DisableCompression: false,
            ForceAttemptHTTP2:  true,
        },
    }
}

// Verify certificate
func verifyServerCertificate(rawCerts [][]byte, verifiedChains [][]*x509.Certificate) error {
    cert, err := x509.ParseCertificate(rawCerts[0])
    if err != nil {
        return err
    }
    
    // Additional certificate validation
    if time.Now().After(cert.NotAfter) {
        return errors.New("certificate has expired")
    }
    
    return nil
}
```

### Certificate Management with Let's Encrypt

```go
import (
    "golang.org/x/crypto/acme/autocert"
)

func setupAutoCert(domains []string) http.Handler {
    certManager := &autocert.Manager{
        Prompt:      autocert.AcceptTOS,
        HostPolicy:  autocert.HostWhitelist(domains...),
        Cache:       autocert.DirCache("/var/www/.cache"),
    }
    
    server := &http.Server{
        Addr: ":443",
        TLSConfig: &tls.Config{
            GetCertificate: certManager.GetCertificate,
            MinVersion:     tls.VersionTLS13,
        },
    }
    
    // Handle ACME challenges
    go http.ListenAndServe(":80", certManager.HTTPHandler(nil))
    
    return server.Handler
}
```

---

## 9.8 Secrets Management

### Environment Variables

```go
import "os"

// ❌ BAD: Hardcoded secrets
const apiKey = "sk_live_abc123"  // DON'T DO THIS!

// ✅ GOOD: Environment variables
func getAPIKey() string {
    key := os.Getenv("API_KEY")
    if key == "" {
        log.Fatal("API_KEY not set")
    }
    return key
}

// ✅ BETTER: Validation
func getSecret(name string) (string, error) {
    value := os.Getenv(name)
    if value == "" {
        return "", fmt.Errorf("environment variable %s not set", name)
    }
    return value, nil
}
```

### Vault Integration

```go
import (
    vault "github.com/hashicorp/vault/api"
)

type SecretManager struct {
    client *vault.Client
}

func NewSecretManager(address, token string) (*SecretManager, error) {
    config := vault.DefaultConfig()
    config.Address = address
    
    client, err := vault.NewClient(config)
    if err != nil {
        return nil, err
    }
    
    client.SetToken(token)
    
    return &SecretManager{client: client}, nil
}

func (sm *SecretManager) GetSecret(path string) (map[string]interface{}, error) {
    secret, err := sm.client.Logical().Read(path)
    if err != nil {
        return nil, err
    }
    
    if secret == nil {
        return nil, errors.New("secret not found")
    }
    
    return secret.Data, nil
}

func (sm *SecretManager) GetDatabaseCredentials() (string, string, error) {
    secret, err := sm.GetSecret("database/creds/myapp")
    if err != nil {
        return "", "", err
    }
    
    username := secret["username"].(string)
    password := secret["password"].(string)
    
    return username, password, nil
}
```

### AWS Secrets Manager

```go
import (
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/secretsmanager"
)

func getAWSSecret(secretName string) (string, error) {
    sess := session.Must(session.NewSession())
    svc := secretsmanager.New(sess)
    
    input := &secretsmanager.GetSecretValueInput{
        SecretId: aws.String(secretName),
    }
    
    result, err := svc.GetSecretValue(input)
    if err != nil {
        return "", err
    }
    
    return *result.SecretString, nil
}
```

---

## 9.9 Rate Limiting

### Token Bucket Algorithm

```go
import (
    "golang.org/x/time/rate"
    "sync"
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

func (rl *RateLimiter) Allow(key string) bool {
    return rl.getLimiter(key).Allow()
}

// Middleware
func rateLimitMiddleware(rl *RateLimiter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Use IP address as key
            key := r.RemoteAddr
            
            if !rl.Allow(key) {
                w.Header().Set("Retry-After", "60")
                http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}
```

### Redis-Based Rate Limiting

```go
import (
    "github.com/go-redis/redis/v8"
)

type RedisRateLimiter struct {
    client *redis.Client
}

func NewRedisRateLimiter(addr string) *RedisRateLimiter {
    return &RedisRateLimiter{
        client: redis.NewClient(&redis.Options{
            Addr: addr,
        }),
    }
}

func (rl *RedisRateLimiter) Allow(ctx context.Context, key string, limit int, window time.Duration) (bool, error) {
    now := time.Now().Unix()
    
    pipe := rl.client.Pipeline()
    
    // Remove old entries
    pipe.ZRemRangeByScore(ctx, key, "0", fmt.Sprintf("%d", now-int64(window.Seconds())))
    
    // Count current entries
    pipe.ZCard(ctx, key)
    
    // Add current request
    pipe.ZAdd(ctx, key, &redis.Z{
        Score:  float64(now),
        Member: now,
    })
    
    // Set expiration
    pipe.Expire(ctx, key, window)
    
    cmds, err := pipe.Exec(ctx)
    if err != nil {
        return false, err
    }
    
    count := cmds[1].(*redis.IntCmd).Val()
    return count < int64(limit), nil
}
```

---

## 9.10 Security Headers

### Comprehensive Security Headers

```go
func securityHeadersMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Prevent clickjacking
        w.Header().Set("X-Frame-Options", "DENY")
        
        // Prevent MIME sniffing
        w.Header().Set("X-Content-Type-Options", "nosniff")
        
        // Enable XSS protection
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        
        // Strict Transport Security (HSTS)
        w.Header().Set("Strict-Transport-Security", "max-age=63072000; includeSubDomains; preload")
        
        // Content Security Policy
        w.Header().Set("Content-Security-Policy",
            "default-src 'self'; "+
            "script-src 'self'; "+
            "style-src 'self' 'unsafe-inline'; "+
            "img-src 'self' data: https:; "+
            "font-src 'self'; "+
            "connect-src 'self'; "+
            "frame-ancestors 'none'; "+
            "base-uri 'self'; "+
            "form-action 'self'",
        )
        
        // Referrer Policy
        w.Header().Set("Referrer-Policy", "strict-origin-when-cross-origin")
        
        // Permissions Policy
        w.Header().Set("Permissions-Policy",
            "geolocation=(), "+
            "microphone=(), "+
            "camera=()",
        )
        
        next.ServeHTTP(w, r)
    })
}
```

---

## 9.11 Dependency Security

### Vulnerability Scanning

```bash
# Install govulncheck
go install golang.org/x/vuln/cmd/govulncheck@latest

# Scan for vulnerabilities
govulncheck ./...

# Check specific package
govulncheck -json ./cmd/api
```

### Dependency Management

```bash
# Update dependencies
go get -u ./...

# Check for outdated dependencies
go list -u -m all

# Verify dependencies
go mod verify

# Tidy dependencies
go mod tidy
```

### Private Module Authentication

```bash
# Configure Git for private repos
git config --global url."https://username:token@github.com/".insteadOf "https://github.com/"

# Or use SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# Set GOPRIVATE
export GOPRIVATE=github.com/myorg/*
```

---

## 9.12 Secure Logging

### Sanitizing Logs

```go
import (
    "go.uber.org/zap"
    "strings"
)

// ❌ BAD: Logging sensitive data
func badLogging(password, creditCard string) {
    log.Printf("Password: %s", password)  // DON'T!
    log.Printf("Credit card: %s", creditCard)  // DON'T!
}

// ✅ GOOD: Sanitize sensitive data
func goodLogging(password, creditCard string) {
    logger.Info("user authenticated")  // Don't log password at all
    logger.Info("payment processed",
        zap.String("card_last_4", maskCreditCard(creditCard)),
    )
}

func maskCreditCard(card string) string {
    if len(card) < 4 {
        return "****"
    }
    return "****" + card[len(card)-4:]
}

func maskEmail(email string) string {
    parts := strings.Split(email, "@")
    if len(parts) != 2 {
        return email
    }
    
    if len(parts[0]) <= 2 {
        return "**@" + parts[1]
    }
    
    return parts[0][:2] + "***@" + parts[1]
}

// Custom log sanitizer
type SanitizingLogger struct {
    logger *zap.Logger
}

func (sl *SanitizingLogger) Info(msg string, fields ...zap.Field) {
    sanitized := make([]zap.Field, len(fields))
    for i, field := range fields {
        sanitized[i] = sl.sanitizeField(field)
    }
    sl.logger.Info(msg, sanitized...)
}

func (sl *SanitizingLogger) sanitizeField(field zap.Field) zap.Field {
    sensitive := map[string]bool{
        "password":    true,
        "credit_card": true,
        "ssn":         true,
        "api_key":     true,
    }
    
    if sensitive[field.Key] {
        return zap.String(field.Key, "***REDACTED***")
    }
    
    return field
}
```

---

## Part 9 Summary and Key Takeaways

### Security Fundamentals

**✅ Input Validation:**
- Validate all user input
- Use allowlists, not blocklists
- Sanitize for context (HTML, SQL, OS commands)
- Prevent path traversal
- Validate email, URL, username formats

**✅ Authentication:**
- Use bcrypt for password hashing (never plain text, MD5, or SHA1)
- Implement JWT with proper validation
- Session management with secure cookies
- Two-factor authentication (TOTP)
- Secure password policies

**✅ Authorization:**
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)
- Principle of least privilege
- Check permissions on every request
- Never trust client-side checks

### Cryptography

**✅ Encryption:**
- AES-GCM for symmetric encryption
- RSA for asymmetric encryption
- Generate strong random keys
- Never roll your own crypto

**✅ Hashing:**
- SHA-256 for general hashing
- HMAC for message authentication
- Constant-time comparison for timing attack prevention
- bcrypt/scrypt/argon2 for passwords

### Web Security

**✅ SQL Injection:**
- Always use parameterized queries
- Validate column names with allowlist
- Never concatenate user input into queries
- Use query builders safely

**✅ XSS Prevention:**
- Use html/template with automatic escaping
- Set Content-Security-Policy headers
- Sanitize rich text with bluemonday
- Validate and encode all output

**✅ CSRF Prevention:**
- Generate and validate CSRF tokens
- Use SameSite cookies
- Implement double-submit cookie pattern
- Require custom headers for state-changing operations

**✅ CORS:**
- Whitelist specific origins (never use *)
- Configure allowed methods and headers
- Handle preflight requests
- Don't allow credentials with wildcard

### TLS & HTTPS

**✅ TLS Configuration:**
- Require TLS 1.3 minimum
- Use strong cipher suites
- Implement HSTS headers
- Redirect HTTP to HTTPS
- Use Let's Encrypt for certificates

### Secrets Management

**✅ Best Practices:**
- Never hardcode secrets
- Use environment variables
- Integrate with Vault or cloud secret managers
- Rotate secrets regularly
- Encrypt secrets at rest

### Rate Limiting

**✅ Strategies:**
- Token bucket algorithm
- Sliding window
- Redis-based distributed rate limiting
- Per-IP or per-user limits
- Return 429 with Retry-After header

### Security Checklist

**Before Deployment:**
- [ ] All inputs validated
- [ ] Parameterized SQL queries
- [ ] HTTPS enabled with strong TLS
- [ ] Security headers configured
- [ ] CSRF protection enabled
- [ ] Rate limiting implemented
- [ ] Secrets in secure storage
- [ ] Dependencies scanned for vulnerabilities
- [ ] Logging sanitized
- [ ] Authentication tested
- [ ] Authorization enforced
- [ ] Error messages don't leak info

**Security Headers:**
- [ ] X-Frame-Options
- [ ] X-Content-Type-Options
- [ ] X-XSS-Protection
- [ ] Strict-Transport-Security
- [ ] Content-Security-Policy
- [ ] Referrer-Policy
- [ ] Permissions-Policy

### Common Vulnerabilities (OWASP Top 10)

**1. Broken Access Control**
- Implement proper authorization
- Check permissions on every request
- Use RBAC or ABAC

**2. Cryptographic Failures**
- Use strong encryption
- Proper key management
- TLS for data in transit

**3. Injection**
- Parameterized queries
- Input validation
- Output encoding

**4. Insecure Design**
- Security by design
- Threat modeling
- Defense in depth

**5. Security Misconfiguration**
- Secure defaults
- Security headers
- Disable unnecessary features

**6. Vulnerable Components**
- Keep dependencies updated
- Scan for vulnerabilities
- Remove unused dependencies

**7. Authentication Failures**
- Strong password policies
- MFA implementation
- Session management

**8. Software and Data Integrity**
- Verify signatures
- Secure CI/CD pipeline
- Code signing

**9. Security Logging Failures**
- Log security events
- Sanitize sensitive data
- Monitor logs

**10. Server-Side Request Forgery**
- Validate URLs
- Block private IP ranges
- Use allowlists

### Tools & Libraries

**Security:**
- bcrypt (password hashing)
- jwt-go (JWT tokens)
- bluemonday (HTML sanitization)
- validator (input validation)

**Crypto:**
- crypto/aes
- crypto/rsa
- crypto/sha256
- golang.org/x/crypto

**Web Security:**
- gorilla/csrf (CSRF protection)
- rs/cors (CORS)
- autocert (Let's Encrypt)

**Scanning:**
- govulncheck (vulnerability scanning)
- gosec (security linter)
- nancy (dependency scanning)

### Interview Questions

**Question 1: How do you prevent SQL injection in Go?**

**Answer:**

Always use parameterized queries:

```go
// ✅ SAFE
query := "SELECT * FROM users WHERE username = $1"
db.QueryRow(query, username)

// ❌ UNSAFE
query := "SELECT * FROM users WHERE username = '" + username + "'"
```

**Key points:**
- Never concatenate user input into SQL
- Use placeholders ($1, $2, ?)
- Validate column names with allowlist for dynamic queries
- Use ORM or query builder with proper escaping

---

**Question 2: How do you securely store passwords?**

**Answer:**

Use bcrypt (or argon2, scrypt):

```go
// Hash password
hash, _ := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)

// Verify password
err := bcrypt.CompareHashAndPassword(hash, []byte(password))
```

**Never:**
- Store plain text passwords
- Use MD5, SHA1, or SHA256 for passwords
- Implement your own hashing algorithm

**Additional security:**
- Enforce strong password policies
- Implement rate limiting on login
- Use MFA
- Monitor for credential stuffing

---

This completes Part 9 of the Complete Go Mastery series on Security Best Practices!

### What's Next

You now know how to:
- Validate and sanitize input
- Implement secure authentication
- Enforce authorization
- Use cryptography correctly
- Prevent SQL injection, XSS, CSRF
- Configure TLS and HTTPS
- Manage secrets securely
- Implement rate limiting
- Set security headers
- Scan for vulnerabilities

**Continue to Part 10 for Interview Preparation & Career Mastery!**

