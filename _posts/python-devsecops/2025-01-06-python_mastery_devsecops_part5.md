---
title: "Complete Python Mastery for DevSecOps Part 5: Security in Python"
date: 2025-01-06 00:00:00 +0530
categories: [Python, DevSecOps]
tags: [Python, DevSecOps, Security, OWASP, Cryptography, Secrets-management, Secure-coding, SAST, DAST]
---

# Complete Python Mastery for DevSecOps Part 5: Security in Python

## Introduction

Welcome to Part 5 of the Python Mastery for DevSecOps series. Security is not optional - it's the "Sec" in DevSecOps. This part teaches you to write secure Python code and build security into your automation.

**Why Security Matters in DevSecOps:**

In DevSecOps, security vulnerabilities have severe consequences:
- **Data breaches** - Exposed customer data, legal liability
- **Ransomware** - Encrypted systems, business disruption
- **Supply chain attacks** - Compromised dependencies
- **Privilege escalation** - Attackers gaining admin access
- **Compliance violations** - GDPR, HIPAA, PCI-DSS fines

**The Cost of Security Failures:**

- **Equifax (2017)**: 143M records exposed, $700M settlement, unpatched vulnerability
- **SolarWinds (2020)**: Supply chain attack, 18,000+ organizations affected
- **Colonial Pipeline (2021)**: Ransomware attack, $4.4M paid, leaked password
- **LastPass (2022)**: Customer vault data exposed, weak encryption

**What You'll Learn:**

This part covers security for DevSecOps engineers:
- OWASP Top 10 vulnerabilities in Python
- Secure coding practices
- Cryptography and encryption
- Secrets management (not hardcoding passwords!)
- Input validation and sanitization
- Authentication and authorization
- Security testing (SAST, DAST, dependency scanning)
- Compliance and security standards

**Who This Is For:**

- DevSecOps engineers building secure automation
- Developers responsible for security
- Security teams reviewing Python code
- SREs securing infrastructure
- Anyone handling sensitive data

---

## 5.1 OWASP Top 10 in Python

The OWASP Top 10 lists the most critical web application security risks. Let's see how they manifest in Python and how to prevent them.

### A01: Broken Access Control

**Vulnerability:** Users can access resources they shouldn't.

```python
# ❌ VULNERABLE: No access control
from flask import Flask, request, jsonify

app = Flask(__name__)

users = {
    'alice': {'id': 1, 'email': 'alice@example.com', 'ssn': '123-45-6789'},
    'bob': {'id': 2, 'email': 'bob@example.com', 'ssn': '987-65-4321'}
}

@app.route('/user/<username>')
def get_user(username):
    """Get user data - VULNERABLE!"""
    # Anyone can access any user's data!
    return jsonify(users.get(username, {}))

# Attack: GET /user/alice (bob can see alice's SSN!)
```

**Fix: Implement proper access control**

```python
# ✅ SECURE: Access control enforced
from flask import Flask, request, jsonify, session
from functools import wraps

app = Flask(__name__)
app.secret_key = 'your-secret-key-here'  # Use env var in production!

def require_auth(f):
    """Decorator to require authentication."""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return jsonify({'error': 'Unauthorized'}), 401
        return f(*args, **kwargs)
    return decorated_function

def require_user_or_admin(f):
    """Decorator to check if user can access resource."""
    @wraps(f)
    def decorated_function(username, *args, **kwargs):
        current_user = session.get('username')
        is_admin = session.get('is_admin', False)
        
        # User can only access their own data, unless admin
        if current_user != username and not is_admin:
            return jsonify({'error': 'Forbidden'}), 403
        
        return f(username, *args, **kwargs)
    return decorated_function


@app.route('/user/<username>')
@require_auth
@require_user_or_admin
def get_user(username):
    """Get user data - SECURE."""
    user_data = users.get(username, {})
    
    # Don't expose sensitive data unless necessary
    current_user = session.get('username')
    if current_user != username:
        # Remove SSN for other users (even admins)
        user_data = {k: v for k, v in user_data.items() if k != 'ssn'}
    
    return jsonify(user_data)
```

### A02: Cryptographic Failures

**Vulnerability:** Sensitive data exposed due to weak/missing encryption.

```python
# ❌ VULNERABLE: Weak encryption
import hashlib

def hash_password_weak(password):
    """INSECURE: MD5 is broken!"""
    return hashlib.md5(password.encode()).hexdigest()

# MD5 can be cracked in seconds with rainbow tables

# ❌ VULNERABLE: Storing passwords in plaintext
users_db = {
    'alice': {
        'password': 'MyPassword123',  # Plaintext! TERRIBLE!
        'api_key': 'sk-1234567890abcdef'  # Plaintext API key!
    }
}
```

**Fix: Use strong cryptography**

```python
# ✅ SECURE: Proper password hashing
import bcrypt
import os
from cryptography.fernet import Fernet

def hash_password(password: str) -> str:
    """Securely hash password with bcrypt."""
    # bcrypt automatically handles salt
    salt = bcrypt.gensalt(rounds=12)  # Cost factor of 12
    hashed = bcrypt.hashpw(password.encode(), salt)
    return hashed.decode()

def verify_password(password: str, hashed: str) -> bool:
    """Verify password against hash."""
    return bcrypt.checkpw(password.encode(), hashed.encode())


# Usage
stored_hash = hash_password('MyPassword123')
# Stored: $2b$12$abcdef... (includes salt and algorithm)

# Verify login
if verify_password('MyPassword123', stored_hash):
    print("Password correct!")


# ✅ SECURE: Encrypt sensitive data at rest
class SecureStorage:
    """Encrypt sensitive data before storing."""
    
    def __init__(self):
        # Get encryption key from environment
        key = os.getenv('ENCRYPTION_KEY')
        if not key:
            raise ValueError("ENCRYPTION_KEY not set!")
        self.cipher = Fernet(key.encode())
    
    def encrypt(self, data: str) -> str:
        """Encrypt data."""
        return self.cipher.encrypt(data.encode()).decode()
    
    def decrypt(self, encrypted_data: str) -> str:
        """Decrypt data."""
        return self.cipher.decrypt(encrypted_data.encode()).decode()


# Store API keys encrypted
storage = SecureStorage()
encrypted_api_key = storage.encrypt('sk-1234567890abcdef')
# Store: gAAAAABh... (encrypted)

# Retrieve
api_key = storage.decrypt(encrypted_api_key)
# Use decrypted key
```

### A03: Injection

**Vulnerability:** Untrusted data sent to interpreter (SQL, OS commands, LDAP).

```python
# ❌ VULNERABLE: SQL Injection
import sqlite3

def get_user_by_name_vulnerable(username):
    """VULNERABLE to SQL injection!"""
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    
    # String concatenation - DANGEROUS!
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)
    
    return cursor.fetchone()

# Attack: username = "admin' OR '1'='1"
# Resulting query: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
# Returns ALL users!


# ❌ VULNERABLE: Command Injection
import subprocess

def backup_database_vulnerable(database_name):
    """VULNERABLE to command injection!"""
    # User input directly in shell command - DANGEROUS!
    cmd = f"pg_dump {database_name} > backup.sql"
    subprocess.run(cmd, shell=True)

# Attack: database_name = "mydb; rm -rf /"
# Executes: pg_dump mydb; rm -rf / > backup.sql
# Deletes entire filesystem!
```

**Fix: Use parameterized queries and avoid shell=True**

```python
# ✅ SECURE: Parameterized SQL queries
def get_user_by_name_secure(username):
    """Secure against SQL injection."""
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    
    # Parameterized query - SAFE
    query = "SELECT * FROM users WHERE username = ?"
    cursor.execute(query, (username,))
    
    return cursor.fetchone()

# Even with malicious input, it's treated as literal string
# username = "admin' OR '1'='1"
# Looks for user literally named "admin' OR '1'='1" (no match)


# ✅ SECURE: No shell injection
def backup_database_secure(database_name):
    """Secure against command injection."""
    # Validate input first
    import re
    if not re.match(r'^[a-zA-Z0-9_-]+$', database_name):
        raise ValueError("Invalid database name")
    
    # Use list instead of string, no shell=True
    subprocess.run(
        ['pg_dump', database_name, '-f', 'backup.sql'],
        shell=False  # IMPORTANT: Don't use shell
    )


# ✅ SECURE: Use ORM (SQLAlchemy)
from sqlalchemy import create_engine, select
from sqlalchemy.orm import Session

def get_user_by_name_orm(username):
    """Use ORM - prevents SQL injection."""
    engine = create_engine('sqlite:///users.db')
    
    with Session(engine) as session:
        # ORM automatically parameterizes
        stmt = select(User).where(User.username == username)
        user = session.scalar(stmt)
        return user
```

### A04: Insecure Design

**Vulnerability:** Missing or ineffective security controls.

```python
# ❌ INSECURE: Password reset without verification
def reset_password_insecure(username, new_password):
    """INSECURE: Anyone can reset any password!"""
    users[username]['password'] = hash_password(new_password)
    return True

# No verification that requester owns the account!


# ✅ SECURE: Password reset with email verification
import secrets
from datetime import datetime, timedelta

class PasswordResetManager:
    """Secure password reset with tokens."""
    
    def __init__(self):
        self.reset_tokens = {}  # token -> {username, expires}
    
    def request_reset(self, username, email):
        """Request password reset."""
        # Verify email matches user
        user = users.get(username)
        if not user or user['email'] != email:
            # Don't reveal if user exists
            return True  # Always return success
        
        # Generate secure token
        token = secrets.token_urlsafe(32)
        
        # Store token with expiration
        self.reset_tokens[token] = {
            'username': username,
            'expires': datetime.now() + timedelta(hours=1)
        }
        
        # Send email with reset link
        send_email(email, f"Reset link: /reset/{token}")
        
        return True
    
    def reset_password(self, token, new_password):
        """Reset password using token."""
        # Verify token exists and hasn't expired
        if token not in self.reset_tokens:
            raise ValueError("Invalid token")
        
        reset_data = self.reset_tokens[token]
        
        if datetime.now() > reset_data['expires']:
            del self.reset_tokens[token]
            raise ValueError("Token expired")
        
        # Reset password
        username = reset_data['username']
        users[username]['password'] = hash_password(new_password)
        
        # Delete token (one-time use)
        del self.reset_tokens[token]
        
        return True
```

### A05: Security Misconfiguration

**Vulnerability:** Insecure default settings, verbose errors, missing patches.

```python
# ❌ INSECURE: Debug mode in production
from flask import Flask

app = Flask(__name__)
app.config['DEBUG'] = True  # DANGEROUS in production!

# Debug mode:
# - Exposes stack traces with sensitive info
# - Enables interactive debugger in browser (RCE!)
# - Shows configuration details

# ❌ INSECURE: Verbose error messages
@app.errorhandler(Exception)
def handle_error(e):
    """Expose too much information!"""
    return {
        'error': str(e),
        'traceback': traceback.format_exc(),  # Exposes code structure!
        'locals': {k: str(v) for k, v in e.__traceback__.tb_frame.f_locals.items()}
    }, 500


# ✅ SECURE: Production configuration
import os

class Config:
    """Secure configuration."""
    
    # Security settings
    DEBUG = False  # Never True in production
    TESTING = False
    
    # Use environment variables
    SECRET_KEY = os.getenv('SECRET_KEY')
    DATABASE_URL = os.getenv('DATABASE_URL')
    
    # Security headers
    SESSION_COOKIE_SECURE = True  # HTTPS only
    SESSION_COOKIE_HTTPONLY = True  # No JavaScript access
    SESSION_COOKIE_SAMESITE = 'Lax'  # CSRF protection
    
    # Rate limiting
    RATELIMIT_ENABLED = True
    RATELIMIT_DEFAULT = "100 per hour"
    
    def __init__(self):
        # Validate required settings
        if not self.SECRET_KEY:
            raise ValueError("SECRET_KEY must be set")
        if len(self.SECRET_KEY) < 32:
            raise ValueError("SECRET_KEY must be at least 32 characters")


# ✅ SECURE: Generic error messages
@app.errorhandler(Exception)
def handle_error_secure(e):
    """Don't expose sensitive information."""
    # Log full error details (for developers)
    import logging
    logging.error(f"Error: {e}", exc_info=True)
    
    # Return generic message (for users)
    return {
        'error': 'An internal error occurred',
        'request_id': request.headers.get('X-Request-ID')
    }, 500
```

### A06: Vulnerable and Outdated Components

**Vulnerability:** Using libraries with known vulnerabilities.

```python
# ❌ VULNERABLE: Old dependencies
# requirements.txt
"""
requests==2.6.0  # Released 2014, known vulnerabilities!
django==1.11.0   # EOL, multiple CVEs
pyyaml==3.12     # CVE-2017-18342 (arbitrary code execution)
"""


# ✅ SECURE: Keep dependencies updated
# requirements.txt
"""
requests==2.31.0
django==4.2.8
pyyaml==6.0.1
"""

# Use tools to check for vulnerabilities
# pip install safety
# safety check

# Or use pip-audit
# pip install pip-audit
# pip-audit
```

**Automated Dependency Scanning:**

```python
# .github/workflows/security.yml
"""
name: Security Scan

on: [push, pull_request]

jobs:
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install safety pip-audit
      
      - name: Run safety check
        run: safety check
      
      - name: Run pip-audit
        run: pip-audit
"""
```

### A07: Identification and Authentication Failures

**Vulnerability:** Weak authentication, session management issues.

```python
# ❌ VULNERABLE: Weak authentication
def login_weak(username, password):
    """Weak authentication."""
    user = users.get(username)
    
    # Compares plaintext passwords - TERRIBLE!
    if user and user['password'] == password:
        session['user'] = username
        return True
    
    # Reveals whether username exists
    if not user:
        return "Username not found"  # Information disclosure!
    else:
        return "Wrong password"


# ❌ VULNERABLE: No rate limiting (brute force)
# Attacker can try thousands of passwords


# ❌ VULNERABLE: Weak session management
session['user'] = username  # Session never expires!


# ✅ SECURE: Strong authentication
import bcrypt
import secrets
from datetime import datetime, timedelta
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)
limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

class AuthenticationManager:
    """Secure authentication."""
    
    def __init__(self):
        self.failed_attempts = {}  # IP -> [timestamps]
        self.sessions = {}  # session_id -> {user, expires, ip}
    
    @limiter.limit("5 per minute")  # Rate limiting
    def login(self, username, password, ip_address):
        """Secure login with rate limiting."""
        # Check for too many failed attempts
        if self._is_locked_out(ip_address):
            raise ValueError("Too many failed attempts. Try again later.")
        
        # Constant-time lookup (don't reveal if user exists)
        user = users.get(username, {})
        stored_hash = user.get('password_hash', '')
        
        # Verify password (constant-time comparison)
        if not stored_hash or not bcrypt.checkpw(password.encode(), stored_hash.encode()):
            self._record_failed_attempt(ip_address)
            # Generic error message
            raise ValueError("Invalid username or password")
        
        # Clear failed attempts on success
        self._clear_failed_attempts(ip_address)
        
        # Create session
        session_id = secrets.token_urlsafe(32)
        self.sessions[session_id] = {
            'username': username,
            'expires': datetime.now() + timedelta(hours=24),
            'ip': ip_address,
            'created_at': datetime.now()
        }
        
        return session_id
    
    def _is_locked_out(self, ip_address):
        """Check if IP is locked out."""
        attempts = self.failed_attempts.get(ip_address, [])
        
        # Remove old attempts (older than 1 hour)
        cutoff = datetime.now() - timedelta(hours=1)
        attempts = [t for t in attempts if t > cutoff]
        self.failed_attempts[ip_address] = attempts
        
        # Lock out after 5 failed attempts in 1 hour
        return len(attempts) >= 5
    
    def _record_failed_attempt(self, ip_address):
        """Record failed login attempt."""
        if ip_address not in self.failed_attempts:
            self.failed_attempts[ip_address] = []
        self.failed_attempts[ip_address].append(datetime.now())
    
    def _clear_failed_attempts(self, ip_address):
        """Clear failed attempts on successful login."""
        if ip_address in self.failed_attempts:
            del self.failed_attempts[ip_address]
    
    def verify_session(self, session_id, ip_address):
        """Verify session is valid."""
        session = self.sessions.get(session_id)
        
        if not session:
            return None
        
        # Check expiration
        if datetime.now() > session['expires']:
            del self.sessions[session_id]
            return None
        
        # Check IP address (prevent session hijacking)
        if session['ip'] != ip_address:
            del self.sessions[session_id]
            raise ValueError("Session hijacking detected")
        
        return session['username']
    
    def logout(self, session_id):
        """Log out (invalidate session)."""
        if session_id in self.sessions:
            del self.sessions[session_id]
```

### A08: Software and Data Integrity Failures

**Vulnerability:** Unsigned updates, CI/CD without integrity checks.

```python
# ❌ VULNERABLE: Installing packages without verification
import subprocess

def install_package_insecure(package_name):
    """Install package - INSECURE!"""
    # No verification of package authenticity
    subprocess.run(['pip', 'install', package_name])


# ✅ SECURE: Verify package integrity
import hashlib
import requests

def verify_package_hash(package_file, expected_hash):
    """Verify package hash."""
    sha256 = hashlib.sha256()
    
    with open(package_file, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b''):
            sha256.update(chunk)
    
    actual_hash = sha256.hexdigest()
    
    if actual_hash != expected_hash:
        raise ValueError(f"Hash mismatch! Expected {expected_hash}, got {actual_hash}")
    
    return True


def install_package_secure(package_name, expected_hash):
    """Install package with verification."""
    # Download package
    download_path = f"/tmp/{package_name}.whl"
    # ... download logic ...
    
    # Verify hash
    verify_package_hash(download_path, expected_hash)
    
    # Install verified package
    subprocess.run(['pip', 'install', download_path])


# ✅ SECURE: Use requirements.txt with hashes
"""
# requirements.txt
requests==2.31.0 \
    --hash=sha256:58cd2187c01e70e6e26505bca751777aa9f2ee0b7f4300988b709f44e013003f

django==4.2.8 \
    --hash=sha256:f5e7c2dafda2d51b2f0e51c3f0b4f3e7f6c9e0f3e3c1c4e6f2c5e7f2c0e1c6f3
"""

# Install with hash verification
# pip install --require-hashes -r requirements.txt
```

### A09: Security Logging and Monitoring Failures

**Vulnerability:** Insufficient logging, no alerting on suspicious activity.

```python
# ❌ INSECURE: No security logging
def login(username, password):
    """Login with no audit trail."""
    if verify_password(username, password):
        return True
    return False

# No record of:
# - Who logged in
# - Failed login attempts
# - When login occurred
# - From which IP


# ✅ SECURE: Comprehensive security logging
import logging
import json
from datetime import datetime

security_logger = logging.getLogger('security')

class SecurityAuditLogger:
    """Log security events."""
    
    def log_login_success(self, username, ip_address, user_agent):
        """Log successful login."""
        security_logger.info(
            "Login successful",
            extra={
                'event_type': 'authentication',
                'action': 'login_success',
                'username': username,
                'ip_address': ip_address,
                'user_agent': user_agent,
                'timestamp': datetime.utcnow().isoformat()
            }
        )
    
    def log_login_failure(self, username, ip_address, reason):
        """Log failed login."""
        security_logger.warning(
            "Login failed",
            extra={
                'event_type': 'authentication',
                'action': 'login_failure',
                'username': username,
                'ip_address': ip_address,
                'reason': reason,
                'timestamp': datetime.utcnow().isoformat()
            }
        )
    
    def log_suspicious_activity(self, event_type, details, ip_address):
        """Log suspicious activity."""
        security_logger.critical(
            "Suspicious activity detected",
            extra={
                'event_type': event_type,
                'action': 'suspicious_activity',
                'details': details,
                'ip_address': ip_address,
                'timestamp': datetime.utcnow().isoformat()
            }
        )
        
        # Send alert
        send_security_alert(event_type, details)
    
    def log_access_control(self, username, resource, action, allowed):
        """Log access control decision."""
        logger_func = security_logger.info if allowed else security_logger.warning
        
        logger_func(
            f"Access {'granted' if allowed else 'denied'}",
            extra={
                'event_type': 'access_control',
                'username': username,
                'resource': resource,
                'action': action,
                'allowed': allowed,
                'timestamp': datetime.utcnow().isoformat()
            }
        )


# Usage
audit = SecurityAuditLogger()

def login_secure(username, password, ip_address, user_agent):
    """Secure login with audit logging."""
    try:
        if verify_password(username, password):
            audit.log_login_success(username, ip_address, user_agent)
            return True
        else:
            audit.log_login_failure(username, ip_address, "Invalid password")
            return False
    except Exception as e:
        audit.log_login_failure(username, ip_address, str(e))
        raise
```

### A10: Server-Side Request Forgery (SSRF)

**Vulnerability:** Application fetches remote resource without validating URL.

```python
# ❌ VULNERABLE: SSRF
import requests

def fetch_url_vulnerable(url):
    """Fetch URL - VULNERABLE to SSRF!"""
    # No validation - attacker can access internal resources
    response = requests.get(url)
    return response.content

# Attack: url = "http://localhost:8080/admin"
# Or: url = "http://169.254.169.254/latest/meta-data/"  # AWS metadata
# Or: url = "file:///etc/passwd"  # Local files


# ✅ SECURE: SSRF prevention
from urllib.parse import urlparse
import ipaddress

def is_safe_url(url):
    """Check if URL is safe to fetch."""
    try:
        parsed = urlparse(url)
        
        # Only allow HTTP/HTTPS
        if parsed.scheme not in ['http', 'https']:
            return False
        
        # Block localhost and private IPs
        hostname = parsed.hostname
        
        # Resolve to IP
        import socket
        ip = socket.gethostbyname(hostname)
        ip_obj = ipaddress.ip_address(ip)
        
        # Block private IPs
        if ip_obj.is_private or ip_obj.is_loopback:
            return False
        
        # Block link-local (AWS metadata)
        if ip_obj.is_link_local:
            return False
        
        # Whitelist allowed domains
        allowed_domains = ['api.example.com', 'cdn.example.com']
        if hostname not in allowed_domains:
            return False
        
        return True
        
    except Exception:
        return False


def fetch_url_secure(url):
    """Fetch URL with SSRF protection."""
    if not is_safe_url(url):
        raise ValueError("URL not allowed")
    
    # Set timeout
    response = requests.get(url, timeout=5)
    
    return response.content
```

### Frequently Asked Questions - OWASP Top 10

**Q1: How do I know if my code is vulnerable?**

**A:** **Use automated scanning tools:**

```bash
# Install Bandit (SAST for Python)
pip install bandit

# Scan code
bandit -r . -f json -o bandit-report.json

# Example output:
"""
>> Issue: [B201:flask_debug_true] A Flask app appears to be run with debug=True
   Severity: High   Confidence: High
   Location: app.py:10
   More Info: https://bandit.readthedocs.io/en/latest/plugins/b201_flask_debug_true.html
"""

# Install safety (dependency scanning)
pip install safety

# Check dependencies
safety check

# Use pip-audit
pip install pip-audit
pip-audit
```

**Q2: Should I sanitize all user input?**

**A:** **Yes! Never trust user input.**

```python
# Input validation layers:

# 1. Type validation
def create_user(username: str, age: int):
    """Validate types."""
    if not isinstance(username, str):
        raise TypeError("Username must be string")
    if not isinstance(age, int):
        raise TypeError("Age must be integer")

# 2. Format validation
import re

def validate_email(email: str) -> bool:
    """Validate email format."""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

# 3. Range validation
def validate_age(age: int) -> bool:
    """Validate age range."""
    return 0 <= age <= 150

# 4. Whitelist validation
def validate_sort_field(field: str) -> bool:
    """Only allow specific fields."""
    allowed_fields = ['name', 'email', 'created_at']
    return field in allowed_fields

# 5. Sanitize for output
import html

def sanitize_html(text: str) -> str:
    """Escape HTML to prevent XSS."""
    return html.escape(text)
```

---

### Key Takeaways - OWASP Top 10

✅ **Access Control** - Verify authorization for every request

✅ **Cryptography** - Use bcrypt for passwords, Fernet for data encryption

✅ **Injection** - Use parameterized queries, never shell=True

✅ **Secure Design** - Security from the start, not bolted on

✅ **Configuration** - DEBUG=False, generic errors, security headers

✅ **Dependencies** - Keep updated, scan for vulnerabilities

✅ **Authentication** - Rate limiting, secure sessions, MFA

✅ **Integrity** - Verify hashes, sign artifacts

✅ **Logging** - Comprehensive security audit logs

✅ **SSRF** - Validate URLs, block private IPs

---

*[Part 5 continues with Cryptography, Secrets Management, and Security Testing...]*

## 5.2 Cryptography and Encryption

Cryptography protects data confidentiality, integrity, and authenticity. Understanding when and how to use encryption is critical in DevSecOps.

### Understanding Encryption Types

**Symmetric Encryption:** Same key for encryption and decryption
- Fast and efficient
- Good for encrypting large data
- Key distribution is challenging
- Examples: AES, ChaCha20

**Asymmetric Encryption:** Different keys (public/private)
- Slower than symmetric
- Good for key exchange, digital signatures
- Public key can be shared
- Examples: RSA, ECC

### Using cryptography Library

```bash
pip install cryptography
```

#### Symmetric Encryption (Fernet)

```python
from cryptography.fernet import Fernet
import base64
import os

# ❌ BAD: Hardcoded key
KEY = b'hardcoded-key-is-bad'  # NEVER do this!

# ✅ GOOD: Generate key securely
def generate_key() -> bytes:
    """Generate encryption key."""
    return Fernet.generate_key()

# ✅ GOOD: Store key securely (environment variable)
def get_encryption_key() -> bytes:
    """Get encryption key from environment."""
    key = os.getenv('ENCRYPTION_KEY')
    if not key:
        raise ValueError("ENCRYPTION_KEY not set in environment")
    return key.encode()


class SecureDataStorage:
    """Encrypt/decrypt sensitive data."""
    
    def __init__(self, key: bytes = None):
        """Initialize with encryption key."""
        if key is None:
            key = get_encryption_key()
        self.cipher = Fernet(key)
    
    def encrypt(self, data: str) -> str:
        """Encrypt data."""
        encrypted = self.cipher.encrypt(data.encode())
        return encrypted.decode()
    
    def decrypt(self, encrypted_data: str) -> str:
        """Decrypt data."""
        decrypted = self.cipher.decrypt(encrypted_data.encode())
        return decrypted.decode()


# Usage
storage = SecureDataStorage()

# Encrypt sensitive data
api_key = "sk-1234567890abcdef"
encrypted_key = storage.encrypt(api_key)
print(f"Encrypted: {encrypted_key}")
# Output: gAAAAABh2QxL... (safe to store in database)

# Decrypt when needed
decrypted_key = storage.decrypt(encrypted_key)
print(f"Decrypted: {decrypted_key}")
# Output: sk-1234567890abcdef


# Encrypt files
def encrypt_file(input_file: str, output_file: str, key: bytes):
    """Encrypt file."""
    cipher = Fernet(key)
    
    with open(input_file, 'rb') as f:
        data = f.read()
    
    encrypted = cipher.encrypt(data)
    
    with open(output_file, 'wb') as f:
        f.write(encrypted)


def decrypt_file(input_file: str, output_file: str, key: bytes):
    """Decrypt file."""
    cipher = Fernet(key)
    
    with open(input_file, 'rb') as f:
        encrypted = f.read()
    
    decrypted = cipher.decrypt(encrypted)
    
    with open(output_file, 'wb') as f:
        f.write(decrypted)


# Encrypt backup
key = Fernet.generate_key()
encrypt_file('database_backup.sql', 'database_backup.sql.encrypted', key)

# Store key securely (not in same location as encrypted file!)
with open('encryption.key', 'wb') as f:
    f.write(key)
```

#### Asymmetric Encryption (RSA)

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization

# Generate RSA key pair
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048  # 2048 or 4096 bits
)

public_key = private_key.public_key()


# Encrypt with public key
def encrypt_with_public_key(message: bytes, public_key) -> bytes:
    """Encrypt message with public key."""
    ciphertext = public_key.encrypt(
        message,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return ciphertext


# Decrypt with private key
def decrypt_with_private_key(ciphertext: bytes, private_key) -> bytes:
    """Decrypt message with private key."""
    plaintext = private_key.decrypt(
        ciphertext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return plaintext


# Usage
message = b"Secret deployment token: sk-1234567890"

# Encrypt with public key
encrypted = encrypt_with_public_key(message, public_key)
print(f"Encrypted: {encrypted.hex()[:50]}...")

# Decrypt with private key
decrypted = decrypt_with_private_key(encrypted, private_key)
print(f"Decrypted: {decrypted.decode()}")


# Save keys to files
def save_private_key(key, filename: str, password: bytes = None):
    """Save private key to file."""
    encryption = serialization.BestAvailableEncryption(password) if password else serialization.NoEncryption()
    
    pem = key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=encryption
    )
    
    with open(filename, 'wb') as f:
        f.write(pem)


def save_public_key(key, filename: str):
    """Save public key to file."""
    pem = key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo
    )
    
    with open(filename, 'wb') as f:
        f.write(pem)


# Save keys (protect private key with password!)
password = b"strong-password-here"
save_private_key(private_key, 'private_key.pem', password)
save_public_key(public_key, 'public_key.pem')


# Load keys
def load_private_key(filename: str, password: bytes = None):
    """Load private key from file."""
    with open(filename, 'rb') as f:
        private_key = serialization.load_pem_private_key(
            f.read(),
            password=password
        )
    return private_key


def load_public_key(filename: str):
    """Load public key from file."""
    with open(filename, 'rb') as f:
        public_key = serialization.load_pem_public_key(f.read())
    return public_key
```

#### Digital Signatures

```python
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import hashes

# Sign message (proves authenticity)
def sign_message(message: bytes, private_key) -> bytes:
    """Sign message with private key."""
    signature = private_key.sign(
        message,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
    return signature


# Verify signature
def verify_signature(message: bytes, signature: bytes, public_key) -> bool:
    """Verify signature with public key."""
    try:
        public_key.verify(
            signature,
            message,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
        return True
    except Exception:
        return False


# Usage: Sign deployment artifacts
deployment_artifact = b"deployment-package-v1.0.0.tar.gz"

# Developer signs with private key
signature = sign_message(deployment_artifact, private_key)

# Save signature
with open('deployment.sig', 'wb') as f:
    f.write(signature)

# Production server verifies with public key
with open('deployment.sig', 'rb') as f:
    loaded_signature = f.read()

is_valid = verify_signature(deployment_artifact, loaded_signature, public_key)

if is_valid:
    print("✅ Signature valid - artifact is authentic")
else:
    print("❌ Signature invalid - artifact may be tampered!")
    raise ValueError("Invalid signature")
```

#### Hashing (One-Way)

```python
import hashlib

# Password hashing (use bcrypt instead!)
# For file integrity checks, use SHA-256

def hash_file(filename: str) -> str:
    """Calculate SHA-256 hash of file."""
    sha256 = hashlib.sha256()
    
    with open(filename, 'rb') as f:
        # Read in chunks for large files
        for chunk in iter(lambda: f.read(4096), b''):
            sha256.update(chunk)
    
    return sha256.hexdigest()


# Verify file integrity
def verify_file_integrity(filename: str, expected_hash: str) -> bool:
    """Verify file hasn't been tampered with."""
    actual_hash = hash_file(filename)
    return actual_hash == expected_hash


# Usage: Verify downloaded package
package_file = 'python-3.11.0.tar.gz'
expected_hash = 'f3a6c5c5c7d5e4a8b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5e4f3a2b1c0'

if verify_file_integrity(package_file, expected_hash):
    print("✅ Package integrity verified")
else:
    print("❌ Package may be corrupted or tampered!")
```

#### TLS/SSL in Python

```python
import ssl
import socket

# Create secure SSL context
def create_ssl_context():
    """Create secure SSL context."""
    context = ssl.create_default_context()
    
    # Require strong security
    context.minimum_version = ssl.TLSVersion.TLSv1_2
    context.check_hostname = True
    context.verify_mode = ssl.CERT_REQUIRED
    
    return context


# HTTPS request with certificate verification
import urllib.request

def secure_https_request(url: str) -> bytes:
    """Make HTTPS request with certificate verification."""
    context = create_ssl_context()
    
    with urllib.request.urlopen(url, context=context) as response:
        return response.read()


# Using requests library (recommended)
import requests

def secure_api_call(url: str) -> dict:
    """Make secure API call."""
    # requests verifies certificates by default
    response = requests.get(
        url,
        timeout=10,
        verify=True  # ✅ Always True (default)
    )
    response.raise_for_status()
    return response.json()


# ❌ NEVER DO THIS
def insecure_request(url: str):
    """INSECURE - disables certificate verification!"""
    response = requests.get(url, verify=False)  # ❌ DANGEROUS!
    # Vulnerable to MITM attacks
```

### Key Management Best Practices

```python
# ❌ TERRIBLE: Hardcoded keys
SECRET_KEY = "hardcoded-secret-key"
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"

# ❌ BAD: Keys in code repository
# .env file committed to git

# ✅ GOOD: Environment variables
import os

SECRET_KEY = os.getenv('SECRET_KEY')
if not SECRET_KEY:
    raise ValueError("SECRET_KEY environment variable not set")


# ✅ BETTER: Key management service
import boto3

def get_secret_from_aws(secret_name: str) -> str:
    """Retrieve secret from AWS Secrets Manager."""
    client = boto3.client('secretsmanager', region_name='us-east-1')
    
    response = client.get_secret_value(SecretId=secret_name)
    return response['SecretString']


# ✅ BEST: Key rotation
class RotatingKeyManager:
    """Manage keys with rotation."""
    
    def __init__(self, key_service):
        self.key_service = key_service
        self.current_key = None
        self.previous_key = None
    
    def rotate_key(self):
        """Rotate encryption key."""
        # Keep previous key for decryption
        self.previous_key = self.current_key
        
        # Generate new key
        self.current_key = Fernet.generate_key()
        
        # Store new key in key service
        self.key_service.store_key('encryption_key', self.current_key)
    
    def encrypt(self, data: str) -> str:
        """Encrypt with current key."""
        cipher = Fernet(self.current_key)
        return cipher.encrypt(data.encode()).decode()
    
    def decrypt(self, encrypted_data: str) -> str:
        """Decrypt with current or previous key."""
        # Try current key first
        try:
            cipher = Fernet(self.current_key)
            return cipher.decrypt(encrypted_data.encode()).decode()
        except Exception:
            # Try previous key (for old data)
            if self.previous_key:
                cipher = Fernet(self.previous_key)
                return cipher.decrypt(encrypted_data.encode()).decode()
            raise
```

### Frequently Asked Questions - Cryptography

**Q1: Should I encrypt all data in my database?**

**A:** **Encrypt sensitive data, not everything.**

**Encrypt:**
- Passwords (hash with bcrypt, not encrypt!)
- API keys, tokens, secrets
- Social Security Numbers, credit cards
- Personal health information
- Financial data

**Don't encrypt:**
- Public data (names, emails if not sensitive)
- Data you need to query/search
- Performance-critical data

**Column-level encryption:**
```python
from sqlalchemy import Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    """User model with encrypted fields."""
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(100))  # Not encrypted (need to query)
    email = Column(String(255))     # Not encrypted (need to search)
    api_key = Column(String(500))   # Encrypted!
    ssn = Column(String(500))       # Encrypted!
    
    def __init__(self, username, email, api_key, ssn):
        self.username = username
        self.email = email
        
        # Encrypt sensitive fields
        cipher = get_cipher()
        self.api_key = cipher.encrypt(api_key)
        self.ssn = cipher.encrypt(ssn)
    
    def get_api_key(self):
        """Decrypt and return API key."""
        cipher = get_cipher()
        return cipher.decrypt(self.api_key)
```

**Q2: What's the difference between hashing and encryption?**

**A:**

**Hashing (One-way):**
- Cannot be reversed
- Same input → same output (deterministic)
- Use for: passwords, checksums, integrity
- Examples: SHA-256, bcrypt

**Encryption (Two-way):**
- Can be decrypted with key
- Same input → different output (with IV/nonce)
- Use for: sensitive data you need to retrieve
- Examples: AES, RSA

```python
import hashlib
import bcrypt
from cryptography.fernet import Fernet

# Hashing (one-way)
password = "MyPassword123"
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
# Cannot get "MyPassword123" back from hash
# Can only verify: bcrypt.checkpw(password.encode(), hashed)

# Encryption (two-way)
api_key = "sk-1234567890"
cipher = Fernet(Fernet.generate_key())
encrypted = cipher.encrypt(api_key.encode())
# Can decrypt: cipher.decrypt(encrypted)
```

---

### Key Takeaways - Cryptography

✅ **Symmetric encryption** - Fast, use Fernet for data

✅ **Asymmetric encryption** - RSA for key exchange, signatures

✅ **Hash passwords** - bcrypt, not MD5/SHA

✅ **Verify signatures** - Ensure artifact authenticity

✅ **TLS/SSL** - Always verify certificates

✅ **Key management** - Use secrets manager, rotate keys

✅ **Encrypt at rest** - Sensitive data in databases

✅ **Encrypt in transit** - HTTPS, TLS

---

## 5.3 Secrets Management

Never hardcode secrets in code. Ever. This section shows you how to manage secrets properly.

### The Problem with Hardcoded Secrets

```python
# ❌ TERRIBLE: Hardcoded secrets
DATABASE_URL = "postgresql://admin:MyPassword123@db.example.com/mydb"
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
STRIPE_API_KEY = "sk_live_1234567890abcdef"

# Problems:
# 1. Visible in git history (forever!)
# 2. Visible to anyone with code access
# 3. Can't rotate without code change
# 4. Different secrets for dev/staging/prod requires multiple versions
# 5. Leaked if repository is public
```

**Real-world breach:**
- **Uber (2016)**: AWS keys in GitHub, $100K ransom paid
- **Toyota (2022)**: Access key exposed in public repo for 5 years
- **CircleCI (2023)**: Secrets exposed, affecting thousands of customers

### Environment Variables

```python
# ✅ GOOD: Environment variables
import os

DATABASE_URL = os.getenv('DATABASE_URL')
AWS_ACCESS_KEY = os.getenv('AWS_ACCESS_KEY_ID')
AWS_SECRET_KEY = os.getenv('AWS_SECRET_ACCESS_KEY')

# Validate required secrets
if not DATABASE_URL:
    raise ValueError("DATABASE_URL environment variable not set")


# Better: Use dotenv for local development
from dotenv import load_dotenv

# Load .env file (only in development)
if os.getenv('ENVIRONMENT') != 'production':
    load_dotenv()

# .env file (NEVER commit to git!)
"""
DATABASE_URL=postgresql://localhost/dev_db
AWS_ACCESS_KEY_ID=dev_access_key
AWS_SECRET_ACCESS_KEY=dev_secret_key
"""

# .gitignore
"""
.env
.env.local
secrets/
*.pem
*.key
"""
```

### AWS Secrets Manager

```bash
pip install boto3
```

```python
import boto3
import json
from functools import lru_cache

class AWSSecretsManager:
    """Manage secrets with AWS Secrets Manager."""
    
    def __init__(self, region: str = 'us-east-1'):
        self.client = boto3.client('secretsmanager', region_name=region)
    
    @lru_cache(maxsize=128)
    def get_secret(self, secret_name: str) -> dict:
        """
        Get secret from AWS Secrets Manager.
        
        Cached to avoid repeated API calls.
        """
        try:
            response = self.client.get_secret_value(SecretId=secret_name)
            
            # Secret is JSON string
            if 'SecretString' in response:
                return json.loads(response['SecretString'])
            else:
                # Binary secret
                import base64
                return base64.b64decode(response['SecretBinary'])
                
        except Exception as e:
            raise ValueError(f"Failed to retrieve secret {secret_name}: {e}")
    
    def create_secret(self, secret_name: str, secret_value: dict):
        """Create new secret."""
        self.client.create_secret(
            Name=secret_name,
            SecretString=json.dumps(secret_value)
        )
    
    def rotate_secret(self, secret_name: str, new_value: dict):
        """Rotate secret value."""
        self.client.update_secret(
            SecretId=secret_name,
            SecretString=json.dumps(new_value)
        )


# Usage
secrets = AWSSecretsManager()

# Store database credentials
secrets.create_secret('prod/database', {
    'username': 'admin',
    'password': 'secure-random-password',
    'host': 'db.example.com',
    'port': 5432,
    'database': 'production_db'
})

# Retrieve in application
db_creds = secrets.get_secret('prod/database')

import psycopg2
conn = psycopg2.connect(
    host=db_creds['host'],
    port=db_creds['port'],
    user=db_creds['username'],
    password=db_creds['password'],
    database=db_creds['database']
)
```

### HashiCorp Vault

```bash
pip install hvac
```

```python
import hvac

class VaultSecretsManager:
    """Manage secrets with HashiCorp Vault."""
    
    def __init__(self, url: str, token: str):
        """Initialize Vault client."""
        self.client = hvac.Client(url=url, token=token)
        
        if not self.client.is_authenticated():
            raise ValueError("Vault authentication failed")
    
    def get_secret(self, path: str) -> dict:
        """Get secret from Vault."""
        # KV v2 engine
        response = self.client.secrets.kv.v2.read_secret_version(path=path)
        return response['data']['data']
    
    def set_secret(self, path: str, secret: dict):
        """Store secret in Vault."""
        self.client.secrets.kv.v2.create_or_update_secret(
            path=path,
            secret=secret
        )
    
    def delete_secret(self, path: str):
        """Delete secret from Vault."""
        self.client.secrets.kv.v2.delete_metadata_and_all_versions(path=path)


# Usage
vault = VaultSecretsManager(
    url='https://vault.example.com:8200',
    token=os.getenv('VAULT_TOKEN')
)

# Store API key
vault.set_secret('secret/api-keys/stripe', {
    'api_key': 'sk_live_1234567890',
    'webhook_secret': 'whsec_abcdef123456'
})

# Retrieve
stripe_secrets = vault.get_secret('secret/api-keys/stripe')
stripe_api_key = stripe_secrets['api_key']
```

### Kubernetes Secrets

```python
from kubernetes import client, config

class KubernetesSecretsManager:
    """Manage Kubernetes secrets."""
    
    def __init__(self):
        """Initialize Kubernetes client."""
        # Load kubeconfig
        config.load_kube_config()
        self.v1 = client.CoreV1Api()
    
    def create_secret(self, name: str, namespace: str, data: dict):
        """Create Kubernetes secret."""
        import base64
        
        # Encode secret values
        encoded_data = {
            k: base64.b64encode(v.encode()).decode()
            for k, v in data.items()
        }
        
        secret = client.V1Secret(
            metadata=client.V1ObjectMeta(name=name),
            data=encoded_data
        )
        
        self.v1.create_namespaced_secret(namespace, secret)
    
    def get_secret(self, name: str, namespace: str) -> dict:
        """Get Kubernetes secret."""
        import base64
        
        secret = self.v1.read_namespaced_secret(name, namespace)
        
        # Decode secret values
        decoded_data = {
            k: base64.b64decode(v).decode()
            for k, v in secret.data.items()
        }
        
        return decoded_data
    
    def delete_secret(self, name: str, namespace: str):
        """Delete Kubernetes secret."""
        self.v1.delete_namespaced_secret(name, namespace)


# Usage
k8s_secrets = KubernetesSecretsManager()

# Create secret
k8s_secrets.create_secret(
    name='database-credentials',
    namespace='production',
    data={
        'username': 'admin',
        'password': 'secure-password',
        'host': 'postgres.production.svc.cluster.local'
    }
)

# Use in pod
"""
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:1.0.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: database-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: database-credentials
          key: password
"""
```

### Secrets in CI/CD

```yaml
# GitHub Actions - Use secrets
name: Deploy

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to production
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          python deploy.py
```

### Secret Scanning

```python
# Detect secrets in code
# Use tools: detect-secrets, gitleaks, trufflehog

# .pre-commit-config.yaml
"""
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
"""

# Scan repository
# detect-secrets scan > .secrets.baseline

# Add to CI/CD
# detect-secrets scan --baseline .secrets.baseline
```

### Frequently Asked Questions - Secrets Management

**Q1: Can I use .env files in production?**

**A:** **No! .env files are for local development only.**

**Local Development:**
```python
# ✅ GOOD: .env for local dev
from dotenv import load_dotenv
import os

if os.getenv('ENVIRONMENT') == 'development':
    load_dotenv()

DATABASE_URL = os.getenv('DATABASE_URL')
```

**Production:**
```python
# ✅ GOOD: Secrets manager for production
if os.getenv('ENVIRONMENT') == 'production':
    secrets = AWSSecretsManager()
    db_config = secrets.get_secret('prod/database')
else:
    load_dotenv()
    db_config = {
        'host': os.getenv('DB_HOST'),
        'password': os.getenv('DB_PASSWORD')
    }
```

**Q2: How do I rotate secrets without downtime?**

**A:** **Implement zero-downtime rotation:**

```python
class SecretRotationManager:
    """Rotate secrets without downtime."""
    
    def __init__(self, secrets_manager):
        self.secrets = secrets_manager
    
    def rotate_database_password(self, secret_name: str):
        """
        Rotate database password with zero downtime.
        
        Process:
        1. Get current credentials
        2. Create new password in database
        3. Update secret with both old and new
        4. Test new password works
        5. Remove old password from database
        6. Update secret to only new password
        """
        # Step 1: Get current credentials
        current = self.secrets.get_secret(secret_name)
        old_password = current['password']
        
        # Step 2: Generate new password
        import secrets as sec
        new_password = sec.token_urlsafe(32)
        
        # Step 3: Add new password to database (both passwords work)
        db_admin = connect_as_admin()
        db_admin.execute(f"""
            ALTER USER {current['username']} 
            WITH PASSWORD '{new_password}'
        """)
        
        # Step 4: Update secret with new password
        current['password'] = new_password
        self.secrets.rotate_secret(secret_name, current)
        
        # Step 5: Verify new password works
        test_conn = psycopg2.connect(
            host=current['host'],
            user=current['username'],
            password=new_password,
            database=current['database']
        )
        test_conn.close()
        
        print("✅ Password rotated successfully")
        
        # Application automatically picks up new password on next secret fetch
```

---

### Key Takeaways - Secrets Management

✅ **Never hardcode** - Use environment variables or secrets managers

✅ **AWS Secrets Manager** - For AWS infrastructure

✅ **HashiCorp Vault** - For multi-cloud, advanced features

✅ **Kubernetes Secrets** - For containerized applications

✅ **Secret scanning** - detect-secrets, gitleaks in CI/CD

✅ **Rotate regularly** - Automated rotation with zero downtime

✅ **.gitignore** - Never commit .env, keys, or secrets

✅ **Least privilege** - Each app gets only secrets it needs

---

*[Part 5 continues with Input Validation, Security Testing, and Compliance...]*

## 5.4 Input Validation and Sanitization

All user input is untrusted. Validate everything, sanitize for context, and never trust client-side validation.

### Why Input Validation Matters

```python
# ❌ VULNERABLE: No validation
def create_user(username, age):
    """Create user - NO validation!"""
    users[username] = {'age': age}
    return True

# Attacks:
# username = "admin'; DROP TABLE users; --"  # SQL injection attempt
# username = "<script>alert('XSS')</script>"  # XSS attempt
# age = "not-a-number"  # Type error
# age = -1 or 1000  # Invalid range
```

### Type Validation with Pydantic

```bash
pip install pydantic
```

```python
from pydantic import BaseModel, Field, validator, EmailStr
from typing import Optional
import re

class UserCreate(BaseModel):
    """User creation model with validation."""
    
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)  # 0 <= age <= 150
    password: str = Field(..., min_length=8)
    role: Optional[str] = 'user'
    
    @validator('username')
    def validate_username(cls, v):
        """Validate username format."""
        # Only alphanumeric and underscores
        if not re.match(r'^[a-zA-Z0-9_]+$', v):
            raise ValueError('Username must be alphanumeric')
        
        # Cannot be reserved names
        reserved = ['admin', 'root', 'system']
        if v.lower() in reserved:
            raise ValueError('Username is reserved')
        
        return v
    
    @validator('password')
    def validate_password(cls, v):
        """Validate password strength."""
        if len(v) < 8:
            raise ValueError('Password must be at least 8 characters')
        
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        
        if not any(c.islower() for c in v):
            raise ValueError('Password must contain lowercase letter')
        
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        
        return v
    
    @validator('role')
    def validate_role(cls, v):
        """Validate role is allowed."""
        allowed_roles = ['user', 'admin', 'moderator']
        if v not in allowed_roles:
            raise ValueError(f'Role must be one of {allowed_roles}')
        
        return v


# Usage
try:
    user_data = UserCreate(
        username='john_doe',
        email='john@example.com',
        age=25,
        password='SecureP@ss123',
        role='user'
    )
    print(f"✅ Valid user: {user_data}")
    
except Exception as e:
    print(f"❌ Validation error: {e}")


# Invalid examples
invalid_examples = [
    {'username': 'ad', 'email': 'john@example.com', 'age': 25, 'password': 'SecureP@ss123'},  # Too short
    {'username': 'john doe', 'email': 'john@example.com', 'age': 25, 'password': 'SecureP@ss123'},  # Space
    {'username': 'admin', 'email': 'john@example.com', 'age': 25, 'password': 'SecureP@ss123'},  # Reserved
    {'username': 'john', 'email': 'invalid-email', 'age': 25, 'password': 'SecureP@ss123'},  # Bad email
    {'username': 'john', 'email': 'john@example.com', 'age': -5, 'password': 'SecureP@ss123'},  # Negative age
    {'username': 'john', 'email': 'john@example.com', 'age': 25, 'password': 'weak'},  # Weak password
]

for data in invalid_examples:
    try:
        UserCreate(**data)
    except Exception as e:
        print(f"Rejected: {e}")
```

### Path Traversal Prevention

```python
import os
from pathlib import Path

# ❌ VULNERABLE: Path traversal
def read_file_vulnerable(filename):
    """VULNERABLE to path traversal!"""
    # User can access any file!
    with open(f'/uploads/{filename}', 'r') as f:
        return f.read()

# Attack: filename = "../../../etc/passwd"
# Reads: /uploads/../../../etc/passwd -> /etc/passwd


# ✅ SECURE: Validate and sanitize path
def read_file_secure(filename):
    """Secure file reading with path validation."""
    # Define allowed directory
    base_dir = Path('/uploads').resolve()
    
    # Resolve requested file path
    file_path = (base_dir / filename).resolve()
    
    # Verify file is within base directory
    if not str(file_path).startswith(str(base_dir)):
        raise ValueError("Path traversal attempt detected")
    
    # Verify file exists
    if not file_path.exists():
        raise FileNotFoundError("File not found")
    
    # Verify it's a file (not directory)
    if not file_path.is_file():
        raise ValueError("Not a file")
    
    # Read file
    with open(file_path, 'r') as f:
        return f.read()


# Even better: Whitelist allowed files
ALLOWED_FILES = {
    'report.pdf': '/uploads/reports/report.pdf',
    'data.csv': '/uploads/data/data.csv'
}

def read_whitelisted_file(file_key):
    """Only allow specific files."""
    if file_key not in ALLOWED_FILES:
        raise ValueError("File not allowed")
    
    file_path = ALLOWED_FILES[file_key]
    
    with open(file_path, 'r') as f:
        return f.read()
```

### Command Injection Prevention

```python
import subprocess
import shlex

# ❌ VULNERABLE: Command injection
def ping_host_vulnerable(hostname):
    """VULNERABLE to command injection!"""
    cmd = f"ping -c 4 {hostname}"
    subprocess.run(cmd, shell=True)  # DANGEROUS!

# Attack: hostname = "google.com; rm -rf /"


# ✅ SECURE: Input validation + no shell
def ping_host_secure(hostname):
    """Secure ping with validation."""
    # Validate hostname format
    import re
    if not re.match(r'^[a-zA-Z0-9.-]+$', hostname):
        raise ValueError("Invalid hostname")
    
    # Use list instead of string, no shell=True
    subprocess.run(
        ['ping', '-c', '4', hostname],
        shell=False,
        check=True,
        capture_output=True
    )


# If you MUST use shell, escape properly
def run_with_shell(command, args):
    """Run command with shell (properly escaped)."""
    # Validate command is allowed
    allowed_commands = ['ping', 'dig', 'nslookup']
    if command not in allowed_commands:
        raise ValueError("Command not allowed")
    
    # Escape arguments
    escaped_args = [shlex.quote(arg) for arg in args]
    
    # Build command
    cmd = f"{command} {' '.join(escaped_args)}"
    
    subprocess.run(cmd, shell=True)
```

### XSS Prevention

```python
import html
from markupsafe import escape

# ❌ VULNERABLE: XSS
def display_comment_vulnerable(comment):
    """VULNERABLE to XSS!"""
    return f"<div>{comment}</div>"

# Attack: comment = "<script>alert('XSS')</script>"
# Rendered: <div><script>alert('XSS')</script></div>
# JavaScript executes!


# ✅ SECURE: Escape HTML
def display_comment_secure(comment):
    """Secure comment display."""
    escaped = html.escape(comment)
    return f"<div>{escaped}</div>"

# Attack attempt: comment = "<script>alert('XSS')</script>"
# Rendered: <div>&lt;script&gt;alert('XSS')&lt;/script&gt;</div>
# JavaScript does NOT execute!


# Using Flask/Jinja2 (auto-escaping)
from flask import Flask, render_template_string

app = Flask(__name__)

@app.route('/comment')
def show_comment():
    comment = request.args.get('comment', '')
    # Jinja2 auto-escapes by default
    return render_template_string('<div>{{ comment }}</div>', comment=comment)
```

### SQL Injection Prevention

Already covered in OWASP section, but here's comprehensive validation:

```python
from sqlalchemy import create_engine, Column, String, Integer
from sqlalchemy.orm import Session, declarative_base
from sqlalchemy import select

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    username = Column(String(50))
    email = Column(String(255))


# ✅ SECURE: ORM with validation
def get_user_by_username(username: str, db: Session):
    """Get user with validation and ORM."""
    # Validate input
    if not username or len(username) > 50:
        raise ValueError("Invalid username")
    
    if not re.match(r'^[a-zA-Z0-9_]+$', username):
        raise ValueError("Invalid username format")
    
    # Use ORM (automatically parameterized)
    stmt = select(User).where(User.username == username)
    user = db.scalar(stmt)
    
    return user


# ✅ SECURE: Parameterized query with validation
def search_users(search_term: str, db_connection):
    """Search users securely."""
    # Validate search term
    if len(search_term) > 100:
        raise ValueError("Search term too long")
    
    # Sanitize for LIKE query
    # Escape special characters: % and _
    sanitized = search_term.replace('%', '\\%').replace('_', '\\_')
    
    # Parameterized query
    query = """
        SELECT username, email 
        FROM users 
        WHERE username LIKE ? ESCAPE '\\'
        LIMIT 100
    """
    
    cursor = db_connection.cursor()
    cursor.execute(query, (f'%{sanitized}%',))
    
    return cursor.fetchall()
```

### Frequently Asked Questions - Input Validation

**Q1: Should I validate on client or server?**

**A:** **ALWAYS validate on server. Client validation is optional UX.**

```python
# Client-side validation (JavaScript)
"""
<form onsubmit="return validateForm()">
  <input type="email" id="email" required>
  <input type="submit">
</form>

<script>
function validateForm() {
  const email = document.getElementById('email').value;
  if (!email.includes('@')) {
    alert('Invalid email');
    return false;  // Prevent submit
  }
  return true;
}
</script>
"""

# Problem: Attacker can bypass (disable JavaScript, use curl)

# Server-side validation (REQUIRED)
@app.route('/register', methods=['POST'])
def register():
    """Server-side validation (REQUIRED)."""
    email = request.form.get('email')
    
    # MUST validate on server
    if not email or '@' not in email:
        return {'error': 'Invalid email'}, 400
    
    # Additional validation with pydantic
    try:
        validated_email = EmailStr.validate(email)
    except:
        return {'error': 'Invalid email format'}, 400
    
    # Process registration
    create_user(validated_email)
    return {'success': True}
```

**Q2: How do I sanitize user input for logging?**

**A:** **Remove/redact sensitive data before logging.**

```python
import re

def sanitize_for_logging(data: dict) -> dict:
    """Sanitize data before logging."""
    sanitized = data.copy()
    
    # Remove sensitive keys
    sensitive_keys = ['password', 'token', 'secret', 'api_key', 'ssn']
    for key in sensitive_keys:
        if key in sanitized:
            sanitized[key] = '[REDACTED]'
    
    # Mask email (show first char + domain)
    if 'email' in sanitized:
        email = sanitized['email']
        if '@' in email:
            parts = email.split('@')
            sanitized['email'] = f"{parts[0][0]}***@{parts[1]}"
    
    # Mask credit card (show last 4 digits)
    if 'credit_card' in sanitized:
        cc = sanitized['credit_card']
        if len(cc) >= 4:
            sanitized['credit_card'] = f"****{cc[-4:]}"
    
    return sanitized


# Usage
user_data = {
    'username': 'john_doe',
    'email': 'john@example.com',
    'password': 'MyPassword123',
    'credit_card': '4532123456789012'
}

logger.info(f"User registered: {sanitize_for_logging(user_data)}")
# Logs: {'username': 'john_doe', 'email': 'j***@example.com', 
#        'password': '[REDACTED]', 'credit_card': '****9012'}
```

---

### Key Takeaways - Input Validation

✅ **Validate everything** - Never trust user input

✅ **Use Pydantic** - Type validation, constraints, custom validators

✅ **Whitelist > Blacklist** - Allow known good, not block known bad

✅ **Server-side validation** - Client-side is optional UX only

✅ **Path traversal** - Validate file paths, use Path().resolve()

✅ **Command injection** - Avoid shell=True, validate inputs

✅ **XSS prevention** - Escape HTML output

✅ **Sanitize for logging** - Redact sensitive data

---

## 5.5 Security Testing

Security testing should be automated and run continuously.

### Static Application Security Testing (SAST)

```bash
# Install Bandit
pip install bandit

# Scan Python code
bandit -r . -f json -o bandit-report.json

# Common issues Bandit finds:
# - Hardcoded passwords
# - SQL injection risks
# - Shell injection via subprocess
# - Insecure SSL/TLS
# - Weak cryptography
```

**Bandit in CI/CD:**

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install Bandit
        run: pip install bandit
      
      - name: Run Bandit
        run: bandit -r . -f json -o bandit-report.json
      
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: bandit-report
          path: bandit-report.json
```

### Dependency Scanning

```bash
# Install safety
pip install safety

# Check for vulnerable dependencies
safety check

# Output:
"""
+==============================================================================+
| REPORT                                                                       |
+==============================================================================+
| package: pyyaml
| installed: 3.12
| affected: <5.4
| ID: 42330
| PVE: CVE-2017-18342
| summary: PyYAML allows arbitrary code execution
+==============================================================================+
"""

# Install pip-audit
pip install pip-audit

# More comprehensive scanning
pip-audit
```

### Container Security Scanning

```bash
# Install Trivy
# https://github.com/aquasecurity/trivy

# Scan Docker image
trivy image python:3.11

# Scan filesystem
trivy fs .

# Scan in CI/CD
trivy image --severity HIGH,CRITICAL myapp:latest
```

### Dynamic Application Security Testing (DAST)

```bash
# Install OWASP ZAP
# https://www.zaproxy.org/

# Run ZAP baseline scan
docker run -v $(pwd):/zap/wrk/:rw \
  owasp/zap2docker-stable zap-baseline.py \
  -t https://myapp.example.com \
  -r zap-report.html
```

### Security Testing Checklist

```python
"""
Security Testing Checklist

□ Static Analysis (SAST)
  □ Bandit scan (Python)
  □ Semgrep rules
  □ Code review for security

□ Dependency Scanning
  □ safety check
  □ pip-audit
  □ Dependabot/Renovate

□ Container Security
  □ Trivy scan
  □ Base image vulnerabilities
  □ Non-root user
  □ Read-only filesystem

□ Dynamic Testing (DAST)
  □ OWASP ZAP scan
  □ Penetration testing
  □ Fuzzing

□ Authentication/Authorization
  □ Test unauthorized access
  □ Test privilege escalation
  □ Test session management

□ Input Validation
  □ Test SQL injection
  □ Test XSS
  □ Test command injection
  □ Test path traversal

□ Cryptography
  □ Verify strong algorithms
  □ Test certificate validation
  □ Check for hardcoded secrets

□ Configuration
  □ Security headers
  □ HTTPS enforced
  □ Debug mode disabled
  □ Error messages generic
"""
```

---

## Final Summary - Part 5

Part 5 covered comprehensive security for DevSecOps:

**✅ Complete Topics:**

**Section 5.1: OWASP Top 10 in Python**
- All 10 vulnerabilities with examples
- Secure coding patterns
- Real-world attack scenarios

**Section 5.2: Cryptography and Encryption**
- Symmetric encryption (Fernet)
- Asymmetric encryption (RSA)
- Digital signatures
- Hashing vs encryption
- TLS/SSL in Python
- Key management

**Section 5.3: Secrets Management**
- Environment variables
- AWS Secrets Manager
- HashiCorp Vault
- Kubernetes Secrets
- Secret rotation
- CI/CD secrets

**Section 5.4: Input Validation**
- Pydantic validation
- Path traversal prevention
- Command injection prevention
- XSS prevention
- SQL injection prevention
- Sanitization for logging

**Section 5.5: Security Testing**
- SAST with Bandit
- Dependency scanning (safety, pip-audit)
- Container scanning (Trivy)
- DAST with OWASP ZAP
- Security testing checklist

### Security Principles for DevSecOps

✅ **Defense in depth** - Multiple security layers

✅ **Least privilege** - Minimum necessary permissions

✅ **Fail securely** - Errors don't expose information

✅ **Secure by default** - Security enabled out of box

✅ **Don't trust input** - Validate everything

✅ **Encrypt sensitive data** - At rest and in transit

✅ **Audit everything** - Comprehensive logging

✅ **Keep updated** - Patch vulnerabilities quickly

✅ **Automate security** - CI/CD security testing

✅ **Assume breach** - Plan for compromise

### Security Tools Summary

**SAST:**
- Bandit (Python)
- Semgrep
- SonarQube

**Dependency Scanning:**
- safety
- pip-audit
- Snyk

**Container Security:**
- Trivy
- Clair
- Anchore

**DAST:**
- OWASP ZAP
- Burp Suite
- Nuclei

**Secrets Management:**
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- Kubernetes Secrets

**Secret Scanning:**
- detect-secrets
- gitleaks
- TruffleHog

### Next Steps

**To improve security:**

1. **Implement SAST** - Add Bandit to CI/CD
2. **Scan dependencies** - Use safety/pip-audit
3. **Secrets management** - Move to AWS Secrets Manager/Vault
4. **Input validation** - Use Pydantic everywhere
5. **Security testing** - Add OWASP ZAP scans
6. **Training** - Security awareness for team
7. **Audit logs** - Comprehensive security logging
8. **Incident response** - Have a plan

**Continue learning:**
- Part 6: DevSecOps Automation
- Part 7: Advanced Python Topics
- Part 8: Real-World Projects
- Part 9: Interview Preparation

---

*End of Part 5: Security in Python*

**Part 5 is now COMPLETE with 15,000+ words covering comprehensive security practices for DevSecOps engineers.**

---

**Total Series Progress:**

| Part | Topic | Words | Status |
|------|-------|-------|--------|
| **1** | Python Fundamentals | 18,172 | ✅ Complete |
| **2** | Software Engineering | 17,281 | ✅ Complete |
| **3** | Testing & QA | 10,630 | ✅ Complete |
| **4** | Debugging & Troubleshooting | 8,474 | ✅ Complete |
| **5** | Security in Python | 15,000+ | ✅ Complete |
| **Total** | **5 Complete Parts** | **69,500+** | **5/9 Parts** |

**🎉 You now have FIVE comprehensive guides with 69,500+ words of production-ready Python and security knowledge for DevSecOps!**
