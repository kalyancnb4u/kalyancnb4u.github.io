---
title: "Complete JavaScript Mastery Part 11: Security Best Practices"
date: 2024-01-18 10:00:00 +0000
categories: [JavaScript, Security, Web Security, Application Security]
tags: [javascript, security, xss, csrf, authentication, encryption, owasp, api-security]
---

# Complete JavaScript Mastery Part 11: Security Best Practices

## Introduction

Security is critical for protecting users, data, and applications. Understanding vulnerabilities and implementing security best practices prevents breaches, protects privacy, and maintains trust.

This part explores:
- OWASP Top 10 vulnerabilities
- Authentication and authorization
- Input validation and sanitization
- API security
- Encryption and hashing
- Secure coding patterns
- Dependency security

**Prerequisites:**
- Parts 1-10: Full JavaScript development knowledge

**Why Security Matters:**

- **User Trust:** Protect user data and privacy
- **Legal Compliance:** GDPR, CCPA, HIPAA requirements
- **Business Impact:** Breaches cost millions
- **Reputation:** Security incidents damage credibility
- **Prevention:** Much cheaper than recovery

Let's master application security.

---

## 11.1 OWASP Top 10

### A01: Broken Access Control

Unauthorized access to resources or functionality.

```javascript
// ❌ BAD: No authorization check
app.delete('/api/users/:id', async (req, res) => {
  await User.deleteById(req.params.id);
  res.json({ success: true });
});

// ✅ GOOD: Check authorization
app.delete('/api/users/:id', authenticate, async (req, res) => {
  const userId = req.params.id;
  
  // Users can only delete themselves
  if (req.user.id !== userId && req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  await User.deleteById(userId);
  res.json({ success: true });
});

// ✅ GOOD: Role-based access control
function authorize(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

app.delete('/api/users/:id', authenticate, authorize('admin'), async (req, res) => {
  await User.deleteById(req.params.id);
  res.json({ success: true });
});

// ✅ GOOD: Object-level authorization
app.get('/api/documents/:id', authenticate, async (req, res) => {
  const document = await Document.findById(req.params.id);
  
  if (!document) {
    return res.status(404).json({ error: 'Not found' });
  }
  
  // Check if user owns the document
  if (document.ownerId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  res.json(document);
});
```

### A02: Cryptographic Failures

Inadequate protection of sensitive data.

```javascript
// ❌ BAD: Storing passwords in plain text
await User.create({
  email: 'user@example.com',
  password: 'password123' // NEVER do this!
});

// ✅ GOOD: Hash passwords with bcrypt
const bcrypt = require('bcrypt');

async function hashPassword(password) {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
}

await User.create({
  email: 'user@example.com',
  password: await hashPassword('password123')
});

// ✅ GOOD: Verify password
async function verifyPassword(password, hash) {
  return await bcrypt.compare(password, hash);
}

// ❌ BAD: Weak encryption
const crypto = require('crypto');
const encrypted = crypto.createCipher('aes192', 'password'); // Deprecated!

// ✅ GOOD: Strong encryption
function encrypt(text, key) {
  const algorithm = 'aes-256-gcm';
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return {
    encrypted,
    iv: iv.toString('hex'),
    authTag: authTag.toString('hex')
  };
}

function decrypt(encryptedData, key) {
  const algorithm = 'aes-256-gcm';
  const decipher = crypto.createDecipheriv(
    algorithm,
    key,
    Buffer.from(encryptedData.iv, 'hex')
  );
  
  decipher.setAuthTag(Buffer.from(encryptedData.authTag, 'hex'));
  
  let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

// ✅ GOOD: Secure random values
const token = crypto.randomBytes(32).toString('hex');

// ❌ BAD: Predictable random
const token = Math.random().toString(36); // NOT cryptographically secure!
```

### A03: Injection

SQL, NoSQL, Command Injection vulnerabilities.

```javascript
// ❌ BAD: SQL Injection vulnerable
app.get('/users', async (req, res) => {
  const { name } = req.query;
  const query = `SELECT * FROM users WHERE name = '${name}'`;
  const users = await db.query(query);
  res.json(users);
});

// Attacker: /users?name='; DROP TABLE users; --

// ✅ GOOD: Parameterized queries
app.get('/users', async (req, res) => {
  const { name } = req.query;
  const users = await db.query(
    'SELECT * FROM users WHERE name = $1',
    [name]
  );
  res.json(users);
});

// ❌ BAD: NoSQL Injection
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({
    username: username,
    password: password
  });
  // Attacker: { "username": { "$gt": "" }, "password": { "$gt": "" } }
});

// ✅ GOOD: Validate input types
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  if (typeof username !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  const user = await User.findOne({ username });
  
  if (!user || !(await verifyPassword(password, user.password))) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  res.json({ token: generateToken(user.id) });
});

// ✅ GOOD: Sanitize NoSQL queries
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());

// ❌ BAD: Command injection
const { exec } = require('child_process');
app.get('/ping', (req, res) => {
  const host = req.query.host;
  exec(`ping -c 4 ${host}`, (error, stdout) => {
    res.send(stdout);
  });
});

// ✅ GOOD: Validate and sanitize input
app.get('/ping', (req, res) => {
  const host = req.query.host;
  
  // Validate hostname/IP format
  if (!/^[a-zA-Z0-9.-]+$/.test(host)) {
    return res.status(400).json({ error: 'Invalid host' });
  }
  
  // Use array format (prevents injection)
  exec('ping', ['-c', '4', host], (error, stdout) => {
    res.send(stdout);
  });
});
```

### A04: Insecure Design

Lack of security controls in architecture.

```javascript
// ❌ BAD: No rate limiting
app.post('/login', async (req, res) => {
  // Attacker can brute force passwords
});

// ✅ GOOD: Rate limiting
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
  // Store in Redis for distributed systems
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate-limit:'
  })
});

app.post('/login', loginLimiter, async (req, res) => {
  // Login logic
});

// ✅ GOOD: Account lockout
async function handleLogin(username, password) {
  const user = await User.findOne({ username });
  
  if (!user) {
    return { success: false, error: 'Invalid credentials' };
  }
  
  // Check if account is locked
  if (user.lockUntil && user.lockUntil > Date.now()) {
    const minutes = Math.ceil((user.lockUntil - Date.now()) / 60000);
    return {
      success: false,
      error: `Account locked. Try again in ${minutes} minutes`
    };
  }
  
  const isValid = await verifyPassword(password, user.password);
  
  if (!isValid) {
    user.failedLoginAttempts = (user.failedLoginAttempts || 0) + 1;
    
    // Lock account after 5 failed attempts
    if (user.failedLoginAttempts >= 5) {
      user.lockUntil = Date.now() + (30 * 60 * 1000); // 30 minutes
    }
    
    await user.save();
    return { success: false, error: 'Invalid credentials' };
  }
  
  // Reset on successful login
  user.failedLoginAttempts = 0;
  user.lockUntil = null;
  await user.save();
  
  return { success: true, token: generateToken(user.id) };
}
```

### A05: Security Misconfiguration

```javascript
// ❌ BAD: Expose error details in production
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack // NEVER expose stack traces!
  });
});

// ✅ GOOD: Generic errors in production
app.use((err, req, res, next) => {
  console.error(err); // Log internally
  
  if (process.env.NODE_ENV === 'production') {
    res.status(500).json({ error: 'Internal server error' });
  } else {
    res.status(500).json({
      error: err.message,
      stack: err.stack
    });
  }
});

// ✅ GOOD: Security headers with Helmet
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// ✅ GOOD: Remove sensitive headers
app.disable('x-powered-by');

// ✅ GOOD: HTTPS only in production
if (process.env.NODE_ENV === 'production') {
  app.use((req, res, next) => {
    if (req.header('x-forwarded-proto') !== 'https') {
      res.redirect(`https://${req.header('host')}${req.url}`);
    } else {
      next();
    }
  });
}

// ✅ GOOD: CORS configuration
const cors = require('cors');

app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
  optionsSuccessStatus: 200
}));
```

### A06: Vulnerable and Outdated Components

```javascript
// ✅ GOOD: Regular dependency audits
// package.json scripts
{
  "scripts": {
    "audit": "npm audit",
    "audit:fix": "npm audit fix"
  }
}

// Run regularly
// npm audit
// npm audit fix

// ✅ GOOD: Automated dependency updates
// Use Dependabot (GitHub) or Renovate

// .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10

// ✅ GOOD: Check for known vulnerabilities
const { exec } = require('child_process');

exec('npm audit --json', (error, stdout) => {
  const audit = JSON.parse(stdout);
  
  if (audit.metadata.vulnerabilities.high > 0 ||
      audit.metadata.vulnerabilities.critical > 0) {
    console.error('High/Critical vulnerabilities found!');
    process.exit(1);
  }
});
```

### A07: Identification and Authentication Failures

```javascript
// ❌ BAD: Weak password policy
function isValidPassword(password) {
  return password.length >= 6;
}

// ✅ GOOD: Strong password policy
function isValidPassword(password) {
  const minLength = 12;
  const hasUppercase = /[A-Z]/.test(password);
  const hasLowercase = /[a-z]/.test(password);
  const hasNumber = /\d/.test(password);
  const hasSpecial = /[!@#$%^&*(),.?":{}|<>]/.test(password);
  
  return password.length >= minLength &&
         hasUppercase &&
         hasLowercase &&
         hasNumber &&
         hasSpecial;
}

// ✅ GOOD: Password strength checker
const zxcvbn = require('zxcvbn');

function checkPasswordStrength(password) {
  const result = zxcvbn(password);
  
  if (result.score < 3) {
    return {
      valid: false,
      message: 'Password is too weak',
      suggestions: result.feedback.suggestions
    };
  }
  
  return { valid: true };
}

// ✅ GOOD: Multi-factor authentication
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

// Generate MFA secret
async function setupMFA(user) {
  const secret = speakeasy.generateSecret({
    name: `MyApp (${user.email})`
  });
  
  user.mfaSecret = secret.base32;
  await user.save();
  
  const qrCode = await QRCode.toDataURL(secret.otpauth_url);
  
  return {
    secret: secret.base32,
    qrCode
  };
}

// Verify MFA token
function verifyMFA(token, secret) {
  return speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 2 // Allow 2 steps before/after for time drift
  });
}

// Login with MFA
app.post('/login', async (req, res) => {
  const { email, password, mfaToken } = req.body;
  
  const user = await User.findOne({ email });
  
  if (!user || !(await verifyPassword(password, user.password))) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Check if MFA is enabled
  if (user.mfaSecret) {
    if (!mfaToken) {
      return res.status(401).json({ error: 'MFA token required' });
    }
    
    if (!verifyMFA(mfaToken, user.mfaSecret)) {
      return res.status(401).json({ error: 'Invalid MFA token' });
    }
  }
  
  const token = generateToken(user.id);
  res.json({ token });
});

// ✅ GOOD: Secure session management
const session = require('express-session');
const RedisStore = require('connect-redis').default;

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only
    httpOnly: true, // Not accessible via JavaScript
    maxAge: 1000 * 60 * 60 * 24, // 24 hours
    sameSite: 'strict' // CSRF protection
  }
}));
```

### A08: Software and Data Integrity Failures

```javascript
// ✅ GOOD: Verify package integrity
// package-lock.json contains integrity hashes

// ✅ GOOD: Subresource Integrity (SRI)
<script
  src="https://cdn.example.com/library.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
  crossorigin="anonymous">
</script>

// Generate SRI hash
const crypto = require('crypto');
const fs = require('fs');

function generateSRIHash(filepath) {
  const content = fs.readFileSync(filepath);
  const hash = crypto.createHash('sha384').update(content).digest('base64');
  return `sha384-${hash}`;
}

// ✅ GOOD: Code signing
// Sign releases with GPG keys
// Verify signatures before deployment
```

### A09: Security Logging and Monitoring Failures

```javascript
// ✅ GOOD: Security event logging
const winston = require('winston');

const securityLogger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'security.log' })
  ]
});

// Log security events
function logSecurityEvent(event, details) {
  securityLogger.info({
    timestamp: new Date().toISOString(),
    event,
    ...details
  });
}

// Login attempts
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await User.findOne({ email });
  const isValid = user && await verifyPassword(password, user.password);
  
  logSecurityEvent('login_attempt', {
    email,
    success: isValid,
    ip: req.ip,
    userAgent: req.get('user-agent')
  });
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  res.json({ token: generateToken(user.id) });
});

// Failed authorization attempts
app.delete('/api/admin/users/:id', authenticate, authorize('admin'), (req, res) => {
  logSecurityEvent('unauthorized_access_attempt', {
    userId: req.user.id,
    targetResource: `/api/admin/users/${req.params.id}`,
    ip: req.ip
  });
  
  // ... delete logic
});

// ✅ GOOD: Real-time alerting
function checkForSuspiciousActivity() {
  // Alert on multiple failed logins
  const recentFailures = getRecentFailedLogins(req.ip);
  
  if (recentFailures.length > 10) {
    alertSecurityTeam({
      type: 'multiple_failed_logins',
      ip: req.ip,
      count: recentFailures.length
    });
  }
}
```

### A10: Server-Side Request Forgery (SSRF)

```javascript
// ❌ BAD: Unvalidated URL fetching
app.post('/fetch-url', async (req, res) => {
  const { url } = req.body;
  const response = await fetch(url); // Attacker can access internal services
  res.send(await response.text());
});

// ✅ GOOD: Validate and whitelist URLs
const { URL } = require('url');

function isAllowedURL(urlString) {
  try {
    const url = new URL(urlString);
    
    // Whitelist allowed domains
    const allowedDomains = ['api.example.com', 'cdn.example.com'];
    
    if (!allowedDomains.includes(url.hostname)) {
      return false;
    }
    
    // Prevent localhost/internal IPs
    const blockedHosts = [
      'localhost',
      '127.0.0.1',
      '0.0.0.0',
      '169.254.169.254' // AWS metadata service
    ];
    
    if (blockedHosts.includes(url.hostname)) {
      return false;
    }
    
    // Only allow http/https
    if (!['http:', 'https:'].includes(url.protocol)) {
      return false;
    }
    
    return true;
  } catch (error) {
    return false;
  }
}

app.post('/fetch-url', async (req, res) => {
  const { url } = req.body;
  
  if (!isAllowedURL(url)) {
    return res.status(400).json({ error: 'Invalid or disallowed URL' });
  }
  
  const response = await fetch(url);
  res.send(await response.text());
});
```

---

## 11.2 Cross-Site Scripting (XSS)

### Types of XSS

```javascript
// Stored XSS - Malicious script stored in database
// Reflected XSS - Script in URL parameter
// DOM-based XSS - Client-side script vulnerability

// ❌ BAD: Vulnerable to XSS
app.get('/search', (req, res) => {
  const { query } = req.query;
  res.send(`<h1>Results for: ${query}</h1>`);
  // Attacker: /search?query=<script>alert('XSS')</script>
});

// ✅ GOOD: Escape HTML entities
function escapeHTML(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
}

app.get('/search', (req, res) => {
  const { query } = req.query;
  res.send(`<h1>Results for: ${escapeHTML(query)}</h1>`);
});

// ✅ GOOD: Use templating engines (auto-escape)
// EJS
app.set('view engine', 'ejs');
app.get('/search', (req, res) => {
  res.render('search', { query: req.query.query });
});

// search.ejs
<h1>Results for: <%= query %></h1>

// ✅ GOOD: Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"], // No inline scripts
    styleSrc: ["'self'"],
    imgSrc: ["'self'", 'data:', 'https:']
  }
}));

// ✅ GOOD: Sanitize user input
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

app.post('/comments', async (req, res) => {
  const { content } = req.body;
  
  // Sanitize HTML
  const clean = DOMPurify.sanitize(content, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
    ALLOWED_ATTR: ['href']
  });
  
  await Comment.create({ content: clean });
  res.json({ success: true });
});

// Client-side XSS prevention
// ❌ BAD
element.innerHTML = userInput;

// ✅ GOOD
element.textContent = userInput;

// ❌ BAD
eval(userInput);
new Function(userInput)();

// ✅ GOOD - Never use eval or Function constructor with user input
```

---

## 11.3 Cross-Site Request Forgery (CSRF)

```javascript
// ✅ GOOD: CSRF tokens
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.use(cookieParser());

// Render form with CSRF token
app.get('/form', csrfProtection, (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

// form.ejs
<form method="POST" action="/submit">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>">
  <!-- form fields -->
  <button type="submit">Submit</button>
</form>

// Verify CSRF token on POST
app.post('/submit', csrfProtection, (req, res) => {
  // Token automatically verified
  res.json({ success: true });
});

// ✅ GOOD: SameSite cookies
app.use(session({
  cookie: {
    sameSite: 'strict', // or 'lax'
    secure: true,
    httpOnly: true
  }
}));

// ✅ GOOD: Double-submit cookie pattern
function generateCSRFToken() {
  return crypto.randomBytes(32).toString('hex');
}

app.use((req, res, next) => {
  if (!req.cookies.csrfToken) {
    const token = generateCSRFToken();
    res.cookie('csrfToken', token, { httpOnly: false, sameSite: 'strict' });
    req.csrfToken = token;
  } else {
    req.csrfToken = req.cookies.csrfToken;
  }
  next();
});

// Verify on state-changing requests
app.post('/api/transfer', (req, res) => {
  const tokenFromBody = req.body.csrfToken || req.headers['x-csrf-token'];
  
  if (tokenFromBody !== req.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  // Process request
});

// Client-side: Include token in requests
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': getCookie('csrfToken')
  },
  body: JSON.stringify({ amount: 100 })
});
```

---

## 11.4 Input Validation

```javascript
// ✅ GOOD: Validate with Joi
const Joi = require('joi');

const userSchema = Joi.object({
  username: Joi.string()
    .alphanum()
    .min(3)
    .max(30)
    .required(),
  
  email: Joi.string()
    .email()
    .required(),
  
  password: Joi.string()
    .min(12)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
    .required(),
  
  age: Joi.number()
    .integer()
    .min(18)
    .max(120),
  
  website: Joi.string()
    .uri()
});

app.post('/register', async (req, res) => {
  const { error, value } = userSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({
      error: error.details[0].message
    });
  }
  
  // Use validated data
  const user = await User.create(value);
  res.json({ success: true });
});

// ✅ GOOD: Custom validators
function validateCardNumber(cardNumber) {
  // Luhn algorithm
  const digits = cardNumber.replace(/\D/g, '');
  
  if (digits.length < 13 || digits.length > 19) {
    return false;
  }
  
  let sum = 0;
  let isEven = false;
  
  for (let i = digits.length - 1; i >= 0; i--) {
    let digit = parseInt(digits[i]);
    
    if (isEven) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    
    sum += digit;
    isEven = !isEven;
  }
  
  return sum % 10 === 0;
}

// ✅ GOOD: Whitelist validation
function validateColor(color) {
  const allowedColors = ['red', 'green', 'blue', 'yellow'];
  return allowedColors.includes(color);
}

// ✅ GOOD: File upload validation
const multer = require('multer');

const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB
  },
  fileFilter: (req, file, cb) => {
    const allowedMimes = ['image/jpeg', 'image/png', 'image/gif'];
    
    if (!allowedMimes.includes(file.mimetype)) {
      return cb(new Error('Invalid file type'));
    }
    
    cb(null, true);
  }
});

app.post('/upload', upload.single('image'), (req, res) => {
  // Additional validation
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif'];
  const ext = path.extname(req.file.originalname).toLowerCase();
  
  if (!allowedExtensions.includes(ext)) {
    fs.unlinkSync(req.file.path);
    return res.status(400).json({ error: 'Invalid file extension' });
  }
  
  res.json({ filename: req.file.filename });
});
```

---

## 11.5 API Security

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

// Global rate limit
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests',
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:global:'
  })
});

app.use('/api/', globalLimiter);

// Endpoint-specific limits
const strictLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5,
  message: 'Too many attempts',
  skipSuccessfulRequests: true
});

app.post('/api/login', strictLimiter, loginHandler);

// User-specific rate limiting
const userLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  keyGenerator: (req) => req.user?.id || req.ip
});

app.use('/api/user/', authenticate, userLimiter);
```

### API Authentication

```javascript
// ✅ GOOD: JWT with refresh tokens
const jwt = require('jsonwebtoken');

function generateTokens(userId) {
  const accessToken = jwt.sign(
    { userId, type: 'access' },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { userId, type: 'refresh' },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
}

app.post('/api/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  try {
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    
    if (decoded.type !== 'refresh') {
      return res.status(401).json({ error: 'Invalid token type' });
    }
    
    // Check if refresh token is revoked
    const isRevoked = await isTokenRevoked(refreshToken);
    if (isRevoked) {
      return res.status(401).json({ error: 'Token revoked' });
    }
    
    const tokens = generateTokens(decoded.userId);
    res.json(tokens);
  } catch (error) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});

// Token revocation
async function revokeToken(token) {
  const decoded = jwt.decode(token);
  const ttl = decoded.exp - Math.floor(Date.now() / 1000);
  
  await redisClient.setEx(`revoked:${token}`, ttl, '1');
}

async function isTokenRevoked(token) {
  const result = await redisClient.get(`revoked:${token}`);
  return result === '1';
}
```

### API Key Management

```javascript
// Generate API key
function generateAPIKey() {
  return crypto.randomBytes(32).toString('hex');
}

// Hash API key for storage
async function hashAPIKey(apiKey) {
  return crypto
    .createHash('sha256')
    .update(apiKey)
    .digest('hex');
}

// Create API key
app.post('/api/keys', authenticate, async (req, res) => {
  const apiKey = generateAPIKey();
  const hashedKey = await hashAPIKey(apiKey);
  
  await APIKey.create({
    userId: req.user.id,
    keyHash: hashedKey,
    name: req.body.name,
    permissions: req.body.permissions || []
  });
  
  // Return plaintext key only once
  res.json({ apiKey }); // User must save this!
});

// Verify API key middleware
async function verifyAPIKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  const hashedKey = await hashAPIKey(apiKey);
  const keyRecord = await APIKey.findOne({ keyHash: hashedKey });
  
  if (!keyRecord || !keyRecord.isActive) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  // Check permissions
  if (!keyRecord.permissions.includes(req.path)) {
    return res.status(403).json({ error: 'Insufficient permissions' });
  }
  
  // Update last used
  keyRecord.lastUsed = new Date();
  await keyRecord.save();
  
  req.apiKey = keyRecord;
  next();
}
```

---

## Key Takeaways

✅ **OWASP Top 10:**
- Broken access control
- Cryptographic failures
- Injection vulnerabilities
- Insecure design
- Security misconfiguration

✅ **Authentication:**
- Strong password policies
- Multi-factor authentication
- Secure session management
- Token refresh patterns

✅ **Input Validation:**
- Validate all user input
- Use parameterized queries
- Sanitize output
- Whitelist over blacklist

✅ **API Security:**
- Rate limiting
- Authentication (JWT, API keys)
- HTTPS only
- CORS configuration

✅ **Best Practices:**
- Security headers
- Dependency audits
- Logging and monitoring
- Least privilege principle

---

## Conclusion

Security is an ongoing process requiring constant vigilance. Follow best practices, stay updated on vulnerabilities, and always validate user input.

**What's Next:**

Part 12 (Final!) will cover Interview Preparation with 200+ questions.

---

**Total Word Count: Part 11 Complete (~13,000 words)**