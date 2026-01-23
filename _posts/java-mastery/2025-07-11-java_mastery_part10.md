---
title: "Complete Java Mastery Part 10: Security Deep Dive"
date: 2024-01-24 10:00:00 +0000
categories: [Java, Security, Spring-Security]
tags: [java, security, oauth2, jwt, spring-security, owasp, encryption, authentication, authorization, secure-coding]
---

## Introduction

Welcome to Part 10, the **final part** of the Complete Java Mastery series. In Parts 1-9, we covered Java fundamentals through performance optimization. Now we explore **Security** - protecting applications, data, and users from threats.

Security is not optional—it's fundamental. A single vulnerability can compromise entire systems, expose user data, and destroy trust. This part equips you with the knowledge to build secure Java applications from the ground up.

### What You'll Learn in Part 10

* **Authentication & Authorization**: OAuth2, OpenID Connect, JWT, session management
* **Spring Security**: Configuration, filters, method security, CORS, CSRF
* **API Security**: Rate limiting, input validation, secure communication
* **OWASP Top 10**: Understanding and preventing common vulnerabilities
* **Cryptography**: Hashing, encryption, secure random generation
* **Secrets Management**: Vault, cloud providers, best practices
* **Secure Coding**: Input validation, output encoding, safe deserialization
* **Security Testing**: SAST, DAST, dependency scanning, penetration testing

This knowledge enables you to:
* Implement robust authentication and authorization
* Prevent common security vulnerabilities
* Secure APIs and microservices
* Manage secrets and credentials safely
* Write security-conscious code
* Test and validate security measures

---

## 10.1 Authentication & Authorization Fundamentals

### Authentication vs Authorization

```
Authentication: "Who are you?"
- Verify identity (username/password, token, certificate)
- Answers: Is this user legitimate?

Authorization: "What can you do?"
- Verify permissions (roles, scopes, policies)
- Answers: Is this user allowed to perform this action?

Example:
1. User logs in with username/password → Authentication
2. User tries to delete a record → Authorization check
   - Is user an admin? Yes → Allow
   - Is user a regular user? No → Deny
```

### Common Authentication Methods

```java
// 1. Basic Authentication
// Credentials: username:password (Base64 encoded)
// Header: Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

@Configuration
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
}

// 2. Form-Based Authentication
@Configuration
public class FormLoginConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/register").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/perform-login")
                .defaultSuccessUrl("/dashboard")
                .failureUrl("/login?error=true")
            );
        
        return http.build();
    }
}

// 3. Token-Based Authentication (JWT)
@RestController
@RequestMapping("/auth")
public class AuthController {
    
    private final AuthenticationManager authenticationManager;
    private final JwtTokenProvider jwtTokenProvider;
    
    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@RequestBody LoginRequest request) {
        // Authenticate user
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            )
        );
        
        // Generate JWT token
        String token = jwtTokenProvider.generateToken(authentication);
        
        return ResponseEntity.ok(new LoginResponse(token));
    }
}
```

### Role-Based Access Control (RBAC)

```java
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String password;
    
    @ManyToMany(fetch = FetchType.EAGER)
    private Set<Role> roles;
}

@Entity
public class Role {
    @Id
    private Long id;
    private String name;  // ROLE_USER, ROLE_ADMIN, ROLE_MODERATOR
    
    @ManyToMany
    private Set<Permission> permissions;
}

@Entity
public class Permission {
    @Id
    private Long id;
    private String name;  // READ_USERS, WRITE_USERS, DELETE_USERS
}

// Authorization
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping
    @PreAuthorize("hasRole('USER')")
    public List<User> getUsers() {
        return userService.findAll();
    }
    
    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public User createUser(@RequestBody User user) {
        return userService.create(user);
    }
    
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') and hasPermission(#id, 'DELETE_USERS')")
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

---

## 10.2 OAuth2 and OpenID Connect

### OAuth2 Overview

```
OAuth2 is an authorization framework (not authentication!)

Roles:
1. Resource Owner: User
2. Client: Your application
3. Authorization Server: Issues tokens (e.g., Keycloak, Auth0)
4. Resource Server: API protected by tokens

Flow:
┌──────────┐                                     ┌──────────────┐
│  Client  │                                     │Authorization │
│          │─────1. Authorization Request───────>│   Server     │
│          │                                     │              │
│          │<────2. Authorization Code──────────│              │
│          │                                     └──────────────┘
│          │
│          │─────3. Exchange Code for Token─────>┌──────────────┐
│          │                                     │Authorization │
│          │<────4. Access Token + Refresh──────│   Server     │
│          │                                     └──────────────┘
│          │
│          │─────5. Request Resource + Token────>┌──────────────┐
│          │                                     │  Resource    │
│          │<────6. Protected Resource──────────│   Server     │
└──────────┘                                     └──────────────┘
```

### OAuth2 Grant Types

**1. Authorization Code Flow (Most Secure)**

```java
@Configuration
@EnableWebSecurity
public class OAuth2SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
            );
        
        return http.build();
    }
}

// application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope:
              - email
              - profile
        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/auth
            token-uri: https://oauth2.googleapis.com/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
```

**2. Client Credentials Flow (Service-to-Service)**

```java
@Configuration
public class OAuth2ClientConfig {
    
    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository clientRegistrationRepository,
            OAuth2AuthorizedClientRepository authorizedClientRepository) {
        
        OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                .clientCredentials()
                .build();
        
        DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                clientRegistrationRepository,
                authorizedClientRepository
            );
        
        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);
        
        return authorizedClientManager;
    }
}

@Service
public class ApiClient {
    
    private final WebClient webClient;
    
    public ApiClient(OAuth2AuthorizedClientManager clientManager) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2 =
            new ServletOAuth2AuthorizedClientExchangeFilterFunction(clientManager);
        
        this.webClient = WebClient.builder()
            .apply(oauth2.oauth2Configuration())
            .build();
    }
    
    public String callProtectedApi() {
        return webClient.get()
            .uri("https://api.example.com/data")
            .attributes(clientRegistrationId("my-client"))
            .retrieve()
            .bodyToMono(String.class)
            .block();
    }
}
```

**3. Resource Owner Password Credentials (Legacy - Avoid)**

```java
// ⚠️ Only use for trusted first-party clients
@PostMapping("/token")
public ResponseEntity<TokenResponse> getToken(@RequestBody PasswordRequest request) {
    // Validate credentials
    Authentication authentication = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(
            request.getUsername(),
            request.getPassword()
        )
    );
    
    // Generate tokens
    String accessToken = tokenProvider.generateAccessToken(authentication);
    String refreshToken = tokenProvider.generateRefreshToken(authentication);
    
    return ResponseEntity.ok(new TokenResponse(accessToken, refreshToken));
}
```

### OpenID Connect (OIDC)

```java
// OIDC = OAuth2 + Authentication
// Adds ID Token (JWT) with user information

@Configuration
public class OIDCConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .oauth2Login(oauth2 -> oauth2
                .userInfoEndpoint(userInfo -> userInfo
                    .oidcUserService(oidcUserService())
                )
            );
        
        return http.build();
    }
    
    @Bean
    public OidcUserService oidcUserService() {
        return new OidcUserService();
    }
}

// Get user information from ID Token
@RestController
public class UserController {
    
    @GetMapping("/user")
    public Map<String, Object> user(@AuthenticationPrincipal OidcUser oidcUser) {
        Map<String, Object> userInfo = new HashMap<>();
        userInfo.put("sub", oidcUser.getSubject());
        userInfo.put("email", oidcUser.getEmail());
        userInfo.put("name", oidcUser.getFullName());
        userInfo.put("picture", oidcUser.getPicture());
        
        return userInfo;
    }
}
```

---

## 10.3 JWT (JSON Web Tokens)

### JWT Structure

```
JWT = Header.Payload.Signature (Base64 encoded)

Example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Decoded:

Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "1234567890",      // Subject (user ID)
  "name": "John Doe",
  "iat": 1516239022,        // Issued at
  "exp": 1516242622,        // Expiration
  "roles": ["USER", "ADMIN"]
}

Signature:
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

### JWT Implementation

```java
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String secretKey;
    
    @Value("${jwt.expiration}")
    private long validityInMilliseconds;
    
    @PostConstruct
    protected void init() {
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
    }
    
    // Generate token
    public String generateToken(Authentication authentication) {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + validityInMilliseconds);
        
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("roles", userDetails.getAuthorities())
            .signWith(SignatureAlgorithm.HS512, secretKey)
            .compact();
    }
    
    // Validate token
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);
            return true;
        } catch (SignatureException ex) {
            log.error("Invalid JWT signature");
        } catch (MalformedJwtException ex) {
            log.error("Invalid JWT token");
        } catch (ExpiredJwtException ex) {
            log.error("Expired JWT token");
        } catch (UnsupportedJwtException ex) {
            log.error("Unsupported JWT token");
        } catch (IllegalArgumentException ex) {
            log.error("JWT claims string is empty");
        }
        return false;
    }
    
    // Extract username from token
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(token)
            .getBody();
        
        return claims.getSubject();
    }
    
    // Extract roles from token
    @SuppressWarnings("unchecked")
    public List<String> getRolesFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(token)
            .getBody();
        
        return (List<String>) claims.get("roles");
    }
}

// JWT Authentication Filter
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtTokenProvider tokenProvider;
    private final UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        try {
            String jwt = getJwtFromRequest(request);
            
            if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
                String username = tokenProvider.getUsernameFromToken(jwt);
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );
                
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception ex) {
            log.error("Could not set user authentication", ex);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

### JWT Best Practices

```java
// ✅ DO: Use strong secrets
@Value("${jwt.secret}")
private String secret;  // At least 256 bits for HS256

// ❌ DON'T: Hardcode secrets
private String secret = "mySecret123";  // Vulnerable!

// ✅ DO: Set expiration
.setExpiration(new Date(now.getTime() + 15 * 60 * 1000))  // 15 minutes

// ❌ DON'T: No expiration or very long
.setExpiration(new Date(now.getTime() + 365 * 24 * 60 * 60 * 1000))  // 1 year!

// ✅ DO: Use refresh tokens
public class TokenResponse {
    private String accessToken;   // Short-lived (15 min)
    private String refreshToken;  // Long-lived (7 days)
}

// ❌ DON'T: Store sensitive data in JWT
.claim("password", user.getPassword())  // Never!
.claim("creditCard", user.getCreditCard())  // Never!

// ✅ DO: Store minimal data
.claim("userId", user.getId())
.claim("roles", user.getRoles())

// ✅ DO: Validate signature
if (!tokenProvider.validateToken(token)) {
    throw new UnauthorizedException("Invalid token");
}

// ✅ DO: Use HTTPS only
// JWT transmitted over HTTP can be intercepted!
```

---

## 10.4 Spring Security Configuration

### Comprehensive Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final AuthenticationEntryPoint authenticationEntryPoint;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Disable defaults
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            
            // Configure CORS
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            
            // Configure authorization
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/", "/auth/**", "/public/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                
                // Admin endpoints
                .requestMatchers("/admin/**").hasRole("ADMIN")
                
                // API endpoints
                .requestMatchers(HttpMethod.GET, "/api/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.PUT, "/api/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")
                
                // All other requests
                .anyRequest().authenticated()
            )
            
            // Exception handling
            .exceptionHandling(exception -> exception
                .authenticationEntryPoint(authenticationEntryPoint)
            )
            
            // Add JWT filter
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://app.example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("Authorization", "Content-Type"));
        configuration.setExposedHeaders(Arrays.asList("Authorization"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        
        return source;
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // Strength 12
    }
}
```

### Method-Level Security

```java
@Service
public class UserService {
    
    // Check role
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        userRepository.deleteById(userId);
    }
    
    // Check multiple roles
    @PreAuthorize("hasAnyRole('ADMIN', 'MODERATOR')")
    public void banUser(Long userId) {
        userRepository.updateStatus(userId, UserStatus.BANNED);
    }
    
    // Check permission
    @PreAuthorize("hasAuthority('DELETE_USERS')")
    public void deleteUserByPermission(Long userId) {
        userRepository.deleteById(userId);
    }
    
    // Check ownership
    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public void updateUser(Long userId, User updates) {
        userRepository.save(updates);
    }
    
    // Custom SpEL expression
    @PreAuthorize("@securityService.canAccess(#userId, authentication)")
    public void accessUserData(Long userId) {
        // Access user data
    }
    
    // Post-authorization (filter results)
    @PostFilter("filterObject.ownerId == authentication.principal.id or hasRole('ADMIN')")
    public List<Document> getDocuments() {
        return documentRepository.findAll();
    }
}

@Component
public class SecurityService {
    
    public boolean canAccess(Long userId, Authentication authentication) {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        
        // Custom logic
        return userId.equals(userDetails.getId()) ||
               userDetails.getAuthorities().contains(new SimpleGrantedAuthority("ROLE_ADMIN"));
    }
}
```

---

## 10.5 OWASP Top 10 for Java

### 1. Injection Attacks

**SQL Injection:**

```java
// ❌ VULNERABLE: String concatenation
@GetMapping("/users")
public List<User> getUsers(@RequestParam String name) {
    String sql = "SELECT * FROM users WHERE name = '" + name + "'";
    return jdbcTemplate.query(sql, userRowMapper);
}
// Attack: ?name=' OR '1'='1
// Result: Returns all users!

// ✅ SAFE: Parameterized queries
@GetMapping("/users")
public List<User> getUsers(@RequestParam String name) {
    String sql = "SELECT * FROM users WHERE name = ?";
    return jdbcTemplate.query(sql, userRowMapper, name);
}

// ✅ SAFE: JPA with parameters
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.name = :name")
    List<User> findByName(@Param("name") String name);
}
```

**Command Injection:**

```java
// ❌ VULNERABLE: Direct command execution
public void backupDatabase(String filename) {
    Runtime.getRuntime().exec("mysqldump -u root -p password db > " + filename);
}
// Attack: filename = "backup.sql; rm -rf /"

// ✅ SAFE: Validate and sanitize input
public void backupDatabase(String filename) {
    // Validate filename
    if (!filename.matches("^[a-zA-Z0-9_\\-\\.]+$")) {
        throw new IllegalArgumentException("Invalid filename");
    }
    
    // Use ProcessBuilder with separate arguments
    ProcessBuilder pb = new ProcessBuilder(
        "mysqldump",
        "-u", "root",
        "-p", "password",
        "db"
    );
    pb.redirectOutput(new File(filename));
    pb.start();
}
```

### 2. Broken Authentication

```java
// ❌ VULNERABLE: Weak password requirements
public void registerUser(String username, String password) {
    if (password.length() < 6) {
        throw new WeakPasswordException();
    }
    // Save user...
}

// ✅ SAFE: Strong password requirements
public void registerUser(String username, String password) {
    if (!isPasswordStrong(password)) {
        throw new WeakPasswordException(
            "Password must be at least 12 characters, " +
            "include uppercase, lowercase, numbers, and special characters"
        );
    }
    
    String hashedPassword = passwordEncoder.encode(password);
    userRepository.save(new User(username, hashedPassword));
}

private boolean isPasswordStrong(String password) {
    return password.length() >= 12 &&
           password.matches(".*[A-Z].*") &&    // Uppercase
           password.matches(".*[a-z].*") &&    // Lowercase
           password.matches(".*[0-9].*") &&    // Digit
           password.matches(".*[!@#$%^&*].*"); // Special char
}

// ✅ SAFE: Account lockout after failed attempts
@Service
public class LoginService {
    
    private final ConcurrentHashMap<String, Integer> failedAttempts = new ConcurrentHashMap<>();
    private static final int MAX_ATTEMPTS = 5;
    
    public void recordFailedLogin(String username) {
        int attempts = failedAttempts.getOrDefault(username, 0) + 1;
        failedAttempts.put(username, attempts);
        
        if (attempts >= MAX_ATTEMPTS) {
            userRepository.lockAccount(username);
            log.warn("Account locked due to failed login attempts: {}", username);
        }
    }
    
    public void recordSuccessfulLogin(String username) {
        failedAttempts.remove(username);
    }
}
```

### 3. Sensitive Data Exposure

```java
// ❌ VULNERABLE: Logging sensitive data
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    log.info("Login attempt: username={}, password={}", 
        request.getUsername(), 
        request.getPassword());  // Never log passwords!
    
    // ...
}

// ✅ SAFE: Don't log sensitive data
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    log.info("Login attempt: username={}", request.getUsername());
    // ...
}

// ❌ VULNERABLE: Storing passwords in plain text
public void createUser(User user) {
    user.setPassword(user.getPassword());  // Plain text!
    userRepository.save(user);
}

// ✅ SAFE: Hash passwords
public void createUser(User user) {
    String hashedPassword = passwordEncoder.encode(user.getPassword());
    user.setPassword(hashedPassword);
    userRepository.save(user);
}

// ✅ SAFE: Encrypt sensitive data
@Entity
public class User {
    @Id
    private Long id;
    
    private String username;
    
    private String password;  // Hashed
    
    @Convert(converter = CreditCardEncryptor.class)
    private String creditCard;  // Encrypted at rest
}

@Converter
public class CreditCardEncryptor implements AttributeConverter<String, String> {
    
    @Override
    public String convertToDatabaseColumn(String attribute) {
        return encryptionService.encrypt(attribute);
    }
    
    @Override
    public String convertToEntityAttribute(String dbData) {
        return encryptionService.decrypt(dbData);
    }
}
```

### 4. XML External Entities (XXE)

```java
// ❌ VULNERABLE: Default XML parsing
public Document parseXml(String xmlContent) throws Exception {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    DocumentBuilder builder = factory.newDocumentBuilder();
    return builder.parse(new InputSource(new StringReader(xmlContent)));
}
// Attack: <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>

// ✅ SAFE: Disable external entities
public Document parseXmlSafe(String xmlContent) throws Exception {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    
    // Disable external entities
    factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
    factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
    factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
    factory.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
    factory.setXIncludeAware(false);
    factory.setExpandEntityReferences(false);
    
    DocumentBuilder builder = factory.newDocumentBuilder();
    return builder.parse(new InputSource(new StringReader(xmlContent)));
}
```

### 5. Broken Access Control

```java
// ❌ VULNERABLE: No authorization check
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
}
// Any authenticated user can access any user's data!

// ✅ SAFE: Check ownership
@GetMapping("/users/{id}")
@PreAuthorize("#id == authentication.principal.id or hasRole('ADMIN')")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
}

// ✅ SAFE: Service-level check
@Service
public class UserService {
    
    public User getUser(Long id, Authentication auth) {
        User requestingUser = (User) auth.getPrincipal();
        
        // Check if user can access this resource
        if (!requestingUser.getId().equals(id) && !requestingUser.isAdmin()) {
            throw new AccessDeniedException("Cannot access other user's data");
        }
        
        return userRepository.findById(id).orElseThrow();
    }
}
```

### 6. Security Misconfiguration

```yaml
# ❌ VULNERABLE: Default credentials
spring:
  datasource:
    username: admin
    password: admin123

# ✅ SAFE: Environment variables
spring:
  datasource:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

```java
// ❌ VULNERABLE: Detailed error messages
@ExceptionHandler(Exception.class)
public ResponseEntity<?> handleException(Exception e) {
    return ResponseEntity.status(500).body(e.getMessage());
    // Exposes stack traces, database errors, etc.
}

// ✅ SAFE: Generic error messages
@ExceptionHandler(Exception.class)
public ResponseEntity<?> handleException(Exception e) {
    log.error("Error occurred", e);  // Log details internally
    return ResponseEntity.status(500).body("An error occurred");
}
```

### 7. Cross-Site Scripting (XSS)

```java
// ❌ VULNERABLE: Directly rendering user input
@GetMapping("/search")
public String search(@RequestParam String query, Model model) {
    model.addAttribute("query", query);
    return "search";  // search.html renders query directly
}
// search.html: <p>Results for: [[${query}]]</p>
// Attack: ?query=<script>alert('XSS')</script>

// ✅ SAFE: Thymeleaf auto-escapes
// search.html: <p>Results for: [[${query}]]</p>
// Thymeleaf automatically escapes HTML

// ✅ SAFE: OWASP Java Encoder
import org.owasp.encoder.Encode;

@GetMapping("/search")
public String search(@RequestParam String query, Model model) {
    model.addAttribute("query", Encode.forHtml(query));
    return "search";
}

// ✅ SAFE: Content Security Policy
@Configuration
public class SecurityHeadersConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers(headers -> headers
            .contentSecurityPolicy(csp -> csp
                .policyDirectives("default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'")
            )
        );
        return http.build();
    }
}
```

### 8. Insecure Deserialization

```java
// ❌ VULNERABLE: Unrestricted deserialization
public Object deserialize(byte[] data) throws Exception {
    ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(data));
    return ois.readObject();  // Can execute arbitrary code!
}

// ✅ SAFE: Use JSON instead
@PostMapping("/data")
public ResponseEntity<?> receiveData(@RequestBody DataDTO data) {
    // Jackson automatically deserializes from JSON
    return ResponseEntity.ok(processData(data));
}

// ✅ SAFE: If Java serialization required, validate classes
public Object deserializeSafe(byte[] data) throws Exception {
    ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(data)) {
        @Override
        protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
            if (!desc.getName().startsWith("com.example.safe.")) {
                throw new InvalidClassException("Unauthorized deserialization attempt", desc.getName());
            }
            return super.resolveClass(desc);
        }
    };
    return ois.readObject();
}
```

### 9. Using Components with Known Vulnerabilities

```xml
<!-- ❌ VULNERABLE: Old dependencies with known CVEs -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.0.0.RELEASE</version>  <!-- Old version! -->
</dependency>

<!-- ✅ SAFE: Keep dependencies updated -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.1</version>  <!-- Latest stable -->
</dependency>
```

**Dependency Scanning:**

```xml
<!-- OWASP Dependency-Check Maven Plugin -->
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>9.0.7</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
    </configuration>
</plugin>
```

### 10. Insufficient Logging & Monitoring

```java
// ❌ VULNERABLE: No security logging
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    Authentication auth = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
    );
    // No logging!
    return ResponseEntity.ok(generateToken(auth));
}

// ✅ SAFE: Comprehensive security logging
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request, HttpServletRequest httpRequest) {
    String ipAddress = httpRequest.getRemoteAddr();
    
    try {
        Authentication auth = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
        );
        
        // Log successful login
        log.info("Successful login: username={}, ip={}", request.getUsername(), ipAddress);
        
        // Log to security audit log
        securityAuditService.logSuccessfulLogin(request.getUsername(), ipAddress);
        
        return ResponseEntity.ok(generateToken(auth));
        
    } catch (AuthenticationException e) {
        // Log failed login
        log.warn("Failed login attempt: username={}, ip={}, reason={}", 
            request.getUsername(), ipAddress, e.getMessage());
        
        securityAuditService.logFailedLogin(request.getUsername(), ipAddress);
        
        throw e;
    }
}

@Service
public class SecurityAuditService {
    
    public void logSuccessfulLogin(String username, String ipAddress) {
        SecurityEvent event = new SecurityEvent();
        event.setEventType("LOGIN_SUCCESS");
        event.setUsername(username);
        event.setIpAddress(ipAddress);
        event.setTimestamp(LocalDateTime.now());
        
        securityEventRepository.save(event);
    }
    
    public void logFailedLogin(String username, String ipAddress) {
        SecurityEvent event = new SecurityEvent();
        event.setEventType("LOGIN_FAILURE");
        event.setUsername(username);
        event.setIpAddress(ipAddress);
        event.setTimestamp(LocalDateTime.now());
        
        securityEventRepository.save(event);
        
        // Alert on multiple failures
        long recentFailures = securityEventRepository.countRecentFailures(username, Duration.ofMinutes(15));
        if (recentFailures >= 5) {
            alertService.sendAlert("Multiple failed login attempts for user: " + username);
        }
    }
}
```

---

## 10.6 Cryptography and Secure Hashing

### Password Hashing

```java
// ❌ NEVER: Plain text or weak hashing
String password = user.getPassword();  // Plain text
String password = md5(user.getPassword());  // MD5 is broken
String password = sha1(user.getPassword());  // SHA-1 is broken

// ✅ USE: BCrypt
@Configuration
public class SecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // Cost factor 12
    }
}

@Service
public class UserService {
    
    private final PasswordEncoder passwordEncoder;
    
    public void createUser(String username, String password) {
        String hashedPassword = passwordEncoder.encode(password);
        // hashedPassword = $2a$12$randomsalt...hashresult
        
        User user = new User(username, hashedPassword);
        userRepository.save(user);
    }
    
    public boolean verifyPassword(String rawPassword, String hashedPassword) {
        return passwordEncoder.matches(rawPassword, hashedPassword);
    }
}

// ✅ ALTERNATIVE: Argon2
@Bean
public PasswordEncoder passwordEncoder() {
    return new Argon2PasswordEncoder(16, 32, 1, 65536, 3);
}

// ✅ ALTERNATIVE: PBKDF2
@Bean
public PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder("mySecret", 185000, 256);
}
```

### Data Encryption

```java
@Service
public class EncryptionService {
    
    @Value("${encryption.secret}")
    private String encryptionKey;
    
    private SecretKey secretKey;
    private Cipher cipher;
    
    @PostConstruct
    public void init() throws Exception {
        // Derive key from secret
        SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
        KeySpec spec = new PBEKeySpec(
            encryptionKey.toCharArray(),
            "salt".getBytes(),  // Use random salt in production
            65536,
            256
        );
        SecretKey tmp = factory.generateSecret(spec);
        secretKey = new SecretKeySpec(tmp.getEncoded(), "AES");
        
        cipher = Cipher.getInstance("AES/GCM/NoPadding");
    }
    
    public String encrypt(String plaintext) throws Exception {
        byte[] iv = new byte[12];
        new SecureRandom().nextBytes(iv);
        GCMParameterSpec parameterSpec = new GCMParameterSpec(128, iv);
        
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, parameterSpec);
        byte[] cipherText = cipher.doFinal(plaintext.getBytes(StandardCharsets.UTF_8));
        
        // Concatenate IV and ciphertext
        byte[] encrypted = new byte[iv.length + cipherText.length];
        System.arraycopy(iv, 0, encrypted, 0, iv.length);
        System.arraycopy(cipherText, 0, encrypted, iv.length, cipherText.length);
        
        return Base64.getEncoder().encodeToString(encrypted);
    }
    
    public String decrypt(String ciphertext) throws Exception {
        byte[] decoded = Base64.getDecoder().decode(ciphertext);
        
        // Extract IV and ciphertext
        byte[] iv = new byte[12];
        System.arraycopy(decoded, 0, iv, 0, iv.length);
        byte[] cipherText = new byte[decoded.length - iv.length];
        System.arraycopy(decoded, iv.length, cipherText, 0, cipherText.length);
        
        GCMParameterSpec parameterSpec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, secretKey, parameterSpec);
        
        byte[] plaintext = cipher.doFinal(cipherText);
        return new String(plaintext, StandardCharsets.UTF_8);
    }
}
```

### Secure Random Generation

```java
// ❌ VULNERABLE: Predictable random
Random random = new Random();
String token = String.valueOf(random.nextLong());  // Predictable!

// ✅ SAFE: Cryptographically secure random
SecureRandom secureRandom = new SecureRandom();
byte[] tokenBytes = new byte[32];
secureRandom.nextBytes(tokenBytes);
String token = Base64.getUrlEncoder().withoutPadding().encodeToString(tokenBytes);

// ✅ SAFE: UUID for tokens
String token = UUID.randomUUID().toString();
```

---

## 10.7 Secrets Management

### HashiCorp Vault Integration

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
```

```yaml
# bootstrap.yml
spring:
  cloud:
    vault:
      uri: http://localhost:8200
      authentication: TOKEN
      token: ${VAULT_TOKEN}
      kv:
        enabled: true
        backend: secret
        default-context: myapp
```

```java
@Configuration
public class VaultConfig {
    
    @Value("${database.password}")
    private String databasePassword;  // From Vault
    
    @Value("${api.key}")
    private String apiKey;  // From Vault
    
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        config.setUsername("myuser");
        config.setPassword(databasePassword);  // From Vault
        
        return new HikariDataSource(config);
    }
}
```

### AWS Secrets Manager

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>secretsmanager</artifactId>
</dependency>
```

```java
@Configuration
public class SecretsConfig {
    
    @Bean
    public SecretsManagerClient secretsManagerClient() {
        return SecretsManagerClient.builder()
            .region(Region.US_EAST_1)
            .build();
    }
    
    @Bean
    public String databasePassword(SecretsManagerClient client) {
        GetSecretValueRequest request = GetSecretValueRequest.builder()
            .secretId("myapp/database/password")
            .build();
        
        GetSecretValueResponse response = client.getSecretValue(request);
        return response.secretString();
    }
}
```

### Environment Variables Best Practices

```java
// ✅ GOOD: Environment variables
@Value("${DATABASE_PASSWORD}")
private String databasePassword;

// ❌ BAD: Hardcoded secrets
private String databasePassword = "password123";

// ❌ BAD: Committed to Git
// application.properties:
// database.password=password123

// ✅ GOOD: .env file (gitignored)
// .env:
// DATABASE_PASSWORD=actual_password

// .gitignore:
// .env
// application-local.properties
```

---

## 10.8 Security Testing

### Dependency Scanning

```bash
# OWASP Dependency-Check
mvn org.owasp:dependency-check-maven:check

# Snyk
snyk test

# GitHub Dependabot (automatic)
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Static Application Security Testing (SAST)

```bash
# SpotBugs with Find Security Bugs
mvn spotbugs:check

# SonarQube
mvn sonar:sonar \
  -Dsonar.projectKey=myapp \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=token
```

### Dynamic Application Security Testing (DAST)

```bash
# OWASP ZAP
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t http://myapp.example.com

# Burp Suite
# Manual testing through proxy
```

---

## 10.9 API Security and Rate Limiting

### Input Validation

```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    // ✅ Bean Validation
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}

public class CreateUserRequest {
    
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    @Pattern(regexp = "^[a-zA-Z0-9_]+$", message = "Username can only contain letters, numbers, and underscores")
    private String username;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 12, message = "Password must be at least 12 characters")
    private String password;
    
    @Min(value = 18, message = "Must be at least 18 years old")
    @Max(value = 120, message = "Invalid age")
    private Integer age;
}

// ✅ Custom Validation
@Constraint(validatedBy = StrongPasswordValidator.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface StrongPassword {
    String message() default "Password must contain uppercase, lowercase, digit, and special character";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class StrongPasswordValidator implements ConstraintValidator<StrongPassword, String> {
    
    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (password == null) return false;
        
        return password.length() >= 12 &&
               password.matches(".*[A-Z].*") &&
               password.matches(".*[a-z].*") &&
               password.matches(".*[0-9].*") &&
               password.matches(".*[!@#$%^&*].*");
    }
}
```

### Rate Limiting

**Bucket4j (Token Bucket Algorithm):**

```xml
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>8.7.0</version>
</dependency>
```

```java
@Component
public class RateLimitingFilter extends OncePerRequestFilter {
    
    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        
        String key = getClientKey(request);
        Bucket bucket = cache.computeIfAbsent(key, k -> createBucket());
        
        if (bucket.tryConsume(1)) {
            filterChain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded");
        }
    }
    
    private Bucket createBucket() {
        Bandwidth limit = Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1)));
        return Bucket.builder()
            .addLimit(limit)
            .build();
    }
    
    private String getClientKey(HttpServletRequest request) {
        // Rate limit per API key or IP
        String apiKey = request.getHeader("X-API-Key");
        if (apiKey != null) {
            return "api_key:" + apiKey;
        }
        return "ip:" + request.getRemoteAddr();
    }
}
```

**Redis-Based Rate Limiting:**

```java
@Component
public class RedisRateLimiter {
    
    private final StringRedisTemplate redisTemplate;
    
    public boolean isAllowed(String key, int maxRequests, Duration window) {
        String redisKey = "rate_limit:" + key;
        Long currentTime = System.currentTimeMillis();
        Long windowStart = currentTime - window.toMillis();
        
        // Remove old entries
        redisTemplate.opsForZSet().removeRangeByScore(redisKey, 0, windowStart);
        
        // Count requests in window
        Long requestCount = redisTemplate.opsForZSet().zCard(redisKey);
        
        if (requestCount < maxRequests) {
            // Add current request
            redisTemplate.opsForZSet().add(redisKey, currentTime.toString(), currentTime);
            redisTemplate.expire(redisKey, window);
            return true;
        }
        
        return false;
    }
}

@Component
public class RateLimitInterceptor implements HandlerInterceptor {
    
    private final RedisRateLimiter rateLimiter;
    
    @Override
    public boolean preHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler) throws Exception {
        
        String apiKey = request.getHeader("X-API-Key");
        if (apiKey == null) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            return false;
        }
        
        boolean allowed = rateLimiter.isAllowed(
            apiKey,
            100,  // 100 requests
            Duration.ofMinutes(1)  // per minute
        );
        
        if (!allowed) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            return false;
        }
        
        return true;
    }
}
```

### API Key Management

```java
@Service
public class ApiKeyService {
    
    private final ApiKeyRepository apiKeyRepository;
    
    public String generateApiKey(Long userId) {
        // Generate cryptographically secure key
        SecureRandom random = new SecureRandom();
        byte[] bytes = new byte[32];
        random.nextBytes(bytes);
        String apiKey = Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
        
        // Hash before storing
        String hashedKey = hashApiKey(apiKey);
        
        ApiKey key = new ApiKey();
        key.setUserId(userId);
        key.setKeyHash(hashedKey);
        key.setCreatedAt(LocalDateTime.now());
        key.setExpiresAt(LocalDateTime.now().plusYears(1));
        
        apiKeyRepository.save(key);
        
        // Return raw key only once
        return apiKey;
    }
    
    public boolean validateApiKey(String apiKey) {
        String hashedKey = hashApiKey(apiKey);
        
        Optional<ApiKey> key = apiKeyRepository.findByKeyHash(hashedKey);
        
        return key.isPresent() && 
               key.get().getExpiresAt().isAfter(LocalDateTime.now()) &&
               key.get().isActive();
    }
    
    private String hashApiKey(String apiKey) {
        return DigestUtils.sha256Hex(apiKey);
    }
}
```

### HTTPS Enforcement

```java
@Configuration
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure()  // Force HTTPS
            )
            .headers(headers -> headers
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)  // 1 year
                )
            );
        
        return http.build();
    }
}
```

### Request Signing

```java
@Service
public class RequestSigningService {
    
    @Value("${api.secret}")
    private String apiSecret;
    
    public String generateSignature(String method, String uri, String body, String timestamp) {
        String message = method + "\n" + uri + "\n" + body + "\n" + timestamp;
        
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec secretKey = new SecretKeySpec(apiSecret.getBytes(), "HmacSHA256");
            mac.init(secretKey);
            
            byte[] hash = mac.doFinal(message.getBytes());
            return Base64.getEncoder().encodeToString(hash);
        } catch (Exception e) {
            throw new RuntimeException("Failed to generate signature", e);
        }
    }
    
    public boolean verifySignature(String method, String uri, String body, 
                                   String timestamp, String signature) {
        String expected = generateSignature(method, uri, body, timestamp);
        return MessageDigest.isEqual(expected.getBytes(), signature.getBytes());
    }
}

@Component
public class RequestSigningFilter extends OncePerRequestFilter {
    
    private final RequestSigningService signingService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        
        String signature = request.getHeader("X-Signature");
        String timestamp = request.getHeader("X-Timestamp");
        
        if (signature == null || timestamp == null) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            return;
        }
        
        // Check timestamp (prevent replay attacks)
        long requestTime = Long.parseLong(timestamp);
        long currentTime = System.currentTimeMillis();
        if (Math.abs(currentTime - requestTime) > 300000) {  // 5 minutes
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.getWriter().write("Request expired");
            return;
        }
        
        // Verify signature
        CachedBodyHttpServletRequest cachedRequest = new CachedBodyHttpServletRequest(request);
        String body = new String(cachedRequest.getCachedBody());
        
        boolean valid = signingService.verifySignature(
            request.getMethod(),
            request.getRequestURI(),
            body,
            timestamp,
            signature
        );
        
        if (!valid) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.getWriter().write("Invalid signature");
            return;
        }
        
        filterChain.doFilter(cachedRequest, response);
    }
}
```

---

## Summary and Key Takeaways

### Authentication & Authorization Mastery

**Authentication Methods:**
* **Basic Auth**: Simple but requires HTTPS
* **Form-Based**: Traditional web applications
* **Token-Based (JWT)**: Modern, stateless, scalable
* **OAuth2**: Industry standard for authorization
* **OIDC**: OAuth2 + authentication layer

**Authorization Patterns:**
* **RBAC**: Role-based (simple, widely used)
* **ABAC**: Attribute-based (flexible, complex)
* **PBAC**: Policy-based (enterprise, fine-grained)

### OAuth2 & JWT Expertise

**OAuth2 Grant Types:**
1. **Authorization Code**: Most secure, for web apps
2. **Client Credentials**: Service-to-service
3. **Refresh Token**: Long-lived access
4. **Password Credentials**: Legacy, avoid if possible

**JWT Best Practices:**
* ✅ Use strong secrets (256+ bits)
* ✅ Set short expiration (15 minutes)
* ✅ Use refresh tokens
* ✅ Validate signature always
* ✅ Don't store sensitive data
* ✅ Use HTTPS only

### OWASP Top 10 Prevention

**Critical Vulnerabilities:**
1. **Injection**: Use parameterized queries
2. **Broken Auth**: Strong passwords, account lockout
3. **Sensitive Data**: Encrypt at rest, hash passwords
4. **XXE**: Disable external entities
5. **Broken Access**: Check authorization
6. **Misconfiguration**: Secure defaults
7. **XSS**: Auto-escape output, CSP
8. **Insecure Deserialization**: Use JSON
9. **Known Vulnerabilities**: Update dependencies
10. **Logging**: Comprehensive security logging

### Cryptography Fundamentals

**Password Hashing:**
* ✅ BCrypt (cost 12+)
* ✅ Argon2 (winner of password hashing competition)
* ✅ PBKDF2 (NIST approved)
* ❌ Never: MD5, SHA-1, plain text

**Data Encryption:**
* ✅ AES-256-GCM (authenticated encryption)
* ✅ Secure random IVs
* ✅ Proper key management
* ❌ Never: DES, RC4, ECB mode

### Secrets Management

**Best Practices:**
* ✅ HashiCorp Vault for production
* ✅ Cloud provider secrets (AWS, Azure, GCP)
* ✅ Environment variables
* ✅ Never commit secrets to Git
* ✅ Rotate secrets regularly
* ✅ Use different secrets per environment

### API Security Patterns

**Essential Protections:**
* ✅ Rate limiting (prevent abuse)
* ✅ Input validation (prevent injection)
* ✅ API key management
* ✅ HTTPS enforcement
* ✅ Request signing (integrity)
* ✅ CORS configuration
* ✅ Security headers

### Production Security Checklist

**Authentication & Authorization:**
- [ ] Strong password requirements (12+ chars)
- [ ] Account lockout after failures
- [ ] MFA for sensitive operations
- [ ] JWT with short expiration
- [ ] Refresh token rotation
- [ ] Authorization checks on all endpoints

**Data Protection:**
- [ ] Passwords hashed with BCrypt/Argon2
- [ ] Sensitive data encrypted at rest
- [ ] HTTPS enforced everywhere
- [ ] Security headers configured
- [ ] No secrets in code/Git

**Input Validation:**
- [ ] All inputs validated
- [ ] Parameterized queries
- [ ] Output encoding
- [ ] File upload restrictions
- [ ] Request size limits

**Monitoring & Logging:**
- [ ] Security events logged
- [ ] Failed login attempts tracked
- [ ] Suspicious activity alerts
- [ ] Audit trail for sensitive operations
- [ ] Log retention policy

**Dependency Management:**
- [ ] Dependencies up to date
- [ ] Vulnerability scanning (OWASP, Snyk)
- [ ] Automated security updates
- [ ] License compliance

### Frequently Asked Questions

**Q1: Should I use OAuth2 or JWT?**

**A:**

**They're not mutually exclusive!** OAuth2 is an authorization framework; JWT is a token format.

**Use OAuth2 + JWT:**
```
OAuth2: Authorization framework
JWT: Token format for access tokens

Example:
1. User authenticates via OAuth2
2. Authorization server issues JWT access token
3. Client uses JWT to access resources
```

**When to use each:**

| Scenario | Use |
|----------|-----|
| Third-party API access | OAuth2 (authorization code flow) |
| Microservices auth | JWT (issued by auth service) |
| Mobile app backend | OAuth2 + JWT |
| Service-to-service | OAuth2 client credentials + JWT |
| Simple API | API keys or JWT |

---

**Q2: How do I securely store passwords?**

**A:**

**Never store plain text!** Always hash.

**✅ Use BCrypt:**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);  // Cost factor 12
}

// Register user
String hashed = passwordEncoder.encode(rawPassword);
user.setPassword(hashed);

// Verify login
boolean valid = passwordEncoder.matches(inputPassword, user.getPassword());
```

**Why BCrypt?**
* Adaptive (can increase cost as hardware improves)
* Built-in salt (automatically generated)
* Slow by design (prevents brute force)

**Comparison:**

| Algorithm | Security | Speed | Recommendation |
|-----------|----------|-------|----------------|
| BCrypt | Excellent | Slow | ✅ Recommended |
| Argon2 | Excellent | Slow | ✅ Best (newest) |
| PBKDF2 | Good | Medium | ✅ NIST approved |
| SHA-256 | Poor | Fast | ❌ Never for passwords |
| MD5 | Broken | Fast | ❌ Never |

---

**Q3: How do I prevent SQL injection?**

**A:**

**Use parameterized queries!**

**❌ Vulnerable:**
```java
String sql = "SELECT * FROM users WHERE username = '" + username + "'";
// Attack: username = "admin' OR '1'='1"
```

**✅ Safe - JdbcTemplate:**
```java
String sql = "SELECT * FROM users WHERE username = ?";
jdbcTemplate.query(sql, userRowMapper, username);
```

**✅ Safe - JPA:**
```java
@Query("SELECT u FROM User u WHERE u.username = :username")
User findByUsername(@Param("username") String username);
```

**✅ Safe - Named Parameters:**
```java
String sql = "SELECT * FROM users WHERE username = :username";
MapSqlParameterSource params = new MapSqlParameterSource();
params.addValue("username", username);
namedParameterJdbcTemplate.query(sql, params, userRowMapper);
```

**Why it works:**
* Parameters are properly escaped
* SQL structure is fixed
* Attacker input is treated as data, not code

---

### Interview Questions

**Question 1: Explain the OAuth2 authorization code flow.**

**Difficulty:** Mid-Level

**Topics:** OAuth2, Security

**Answer:**

**OAuth2 Authorization Code Flow** (most secure):

**Steps:**
1. **User clicks "Login with Google"**
2. **Client redirects to Authorization Server**
   ```
   https://accounts.google.com/o/oauth2/auth?
     client_id=abc123&
     redirect_uri=https://myapp.com/callback&
     response_type=code&
     scope=email profile
   ```

3. **User authenticates and grants permission**

4. **Authorization Server redirects with code**
   ```
   https://myapp.com/callback?code=xyz789
   ```

5. **Client exchanges code for tokens** (backend)
   ```
   POST https://oauth2.googleapis.com/token
   {
     "code": "xyz789",
     "client_id": "abc123",
     "client_secret": "secret",
     "redirect_uri": "https://myapp.com/callback",
     "grant_type": "authorization_code"
   }
   ```

6. **Authorization Server returns tokens**
   ```json
   {
     "access_token": "ya29.a0...",
     "refresh_token": "1//0...",
     "expires_in": 3600
   }
   ```

7. **Client uses access token to access resources**
   ```
   GET https://www.googleapis.com/oauth2/v1/userinfo
   Authorization: Bearer ya29.a0...
   ```

**Why secure?**
* Authorization code exposed to user agent (browser)
* Access token only in backend (never exposed to browser)
* Client secret never exposed
* Authorization code single-use

---

**Question 2: How do you prevent XSS attacks?**

**Difficulty:** Mid-Level

**Topics:** Security, OWASP

**Answer:**

**XSS (Cross-Site Scripting)** allows attackers to inject malicious scripts.

**Types:**
1. **Reflected XSS**: Input reflected in response
2. **Stored XSS**: Input stored in database
3. **DOM-based XSS**: Client-side script manipulation

**Prevention:**

**1. Output Encoding:**
```java
// ✅ Thymeleaf auto-escapes
<p th:text="${userInput}"></p>
// Input: <script>alert('XSS')</script>
// Output: &lt;script&gt;alert('XSS')&lt;/script&gt;

// ✅ OWASP Java Encoder
import org.owasp.encoder.Encode;
String safe = Encode.forHtml(userInput);
```

**2. Content Security Policy:**
```java
http.headers(headers -> headers
    .contentSecurityPolicy(csp -> csp
        .policyDirectives(
            "default-src 'self'; " +
            "script-src 'self' https://trusted.cdn.com; " +
            "style-src 'self' 'unsafe-inline'"
        )
    )
);
```

**3. Input Validation:**
```java
@Pattern(regexp = "^[a-zA-Z0-9\\s]+$")
private String username;  // Only alphanumeric
```

**4. HTTP-only Cookies:**
```java
Cookie cookie = new Cookie("session", sessionId);
cookie.setHttpOnly(true);  // Prevents JavaScript access
cookie.setSecure(true);    // HTTPS only
```

**Best Practices:**
* ✅ Auto-escape all output (use templating engines)
* ✅ Validate inputs
* ✅ Use CSP headers
* ✅ Set HTTP-only flag on cookies
* ✅ Sanitize HTML if needed (use libraries like OWASP HTML Sanitizer)

---

**End of Part 10: Security Deep Dive**

## 🎉 Series Complete! Congratulations!

You've now completed all **10 parts** of the Complete Java Mastery series!

### Final Achievement Summary

**Part 10 Mastery:**
* ✅ Authentication & Authorization (OAuth2, JWT, OIDC)
* ✅ Spring Security (configuration, filters, method security)
* ✅ OWASP Top 10 (all vulnerabilities with prevention)
* ✅ Cryptography (hashing, encryption, secure random)
* ✅ Secrets Management (Vault, cloud providers)
* ✅ API Security (rate limiting, input validation, signing)
* ✅ Security Testing (SAST, DAST, dependency scanning)

**Complete Series Mastery (Parts 1-10):**
* ✅ Java Fundamentals
* ✅ JVM Internals
* ✅ Advanced Java Features
* ✅ SDLC with Java
* ✅ Spring Framework
* ✅ Microservices Architecture
* ✅ Advanced Spring & Reactive
* ✅ Cloud-Native Java
* ✅ Performance & Optimization
* ✅ Security Deep Dive

### What You've Accomplished

**Technical Expertise:**
- 📚 **~78,000 words** of expert content
- 💻 **600+ code examples**
- 🎯 **10 comprehensive topics**
- 🚀 **Complete production-ready knowledge**
- 💡 **Interview questions throughout**
- 📊 **Real-world patterns and solutions**

**Career Readiness:**
- ✅ Senior Java Developer
- ✅ Java Architect
- ✅ Security Engineer
- ✅ Cloud Engineer
- ✅ Performance Engineer
- ✅ Principal/Staff Engineer
- ✅ Technical Lead

**You can now:**
- Build secure, scalable applications
- Deploy to production with confidence
- Tune performance for any workload
- Implement security best practices
- Lead technical teams
- Make architectural decisions
- Pass senior-level interviews

### References

* [OWASP Top 10](https://owasp.org/www-project-top-ten/)
* [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
* [OAuth 2.0 RFC](https://datatracker.ietf.org/doc/html/rfc6749)
* [JWT.io](https://jwt.io/)
* [NIST Password Guidelines](https://pages.nist.gov/800-63-3/)
* [Spring Security in Action by Laurentiu Spilca](https://www.manning.com/books/spring-security-in-action)
* [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)

---

## 🏆 You Are Now a Complete Java Expert!

From fundamentals to security, from local development to cloud deployment, from basic concepts to advanced optimization—you've mastered it all.

**This knowledge took years to compile and represents professional-level expertise across the entire Java ecosystem.**

**Congratulations on completing the Complete Java Mastery series!** 🎉🎊

---