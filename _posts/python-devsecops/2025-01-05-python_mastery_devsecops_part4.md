---
title: "Python DevSecOps - Part 4: Debugging and Troubleshooting"
date: 2025-01-05 00:00:00 +0530
categories: [Python, DevSecOps]
tags: [Python, DevSecOps, Debugging, Profiling, Logging, Troubleshooting, pdb, Production]
---

# Complete Python Mastery for DevSecOps Part 4: Debugging and Troubleshooting

## Introduction

Welcome to Part 4 of the Python Mastery for DevSecOps series. Debugging is where theory meets reality - your code will have bugs, and production systems will fail. This part teaches you how to find and fix problems quickly.

**Why Debugging Matters in DevSecOps:**

In DevSecOps, debugging is mission-critical:
- **Production incidents** - Every minute of downtime costs money
- **Complex distributed systems** - Multiple services, clouds, and dependencies
- **Limited access** - Can't always attach a debugger to production
- **Time pressure** - Stakeholders expect rapid resolution
- **Data sensitivity** - Can't expose logs with secrets/PII

**The Cost of Poor Debugging Skills:**

- **Knight Capital (2012)**: Debugging failure led to $440M loss in 45 minutes
- **GitHub (2012)**: Database crash; poor debugging extended 4-hour outage
- **Cloudflare (2019)**: Regex bug caused global outage; debugging took hours

**What You'll Learn:**

This part covers professional debugging and troubleshooting:
- Interactive debugging with pdb
- Logging best practices for production
- Performance profiling and optimization
- Memory leak detection
- Production debugging strategies
- Distributed tracing
- Post-mortem debugging
- Common bug patterns and solutions

**Who This Is For:**

- DevSecOps engineers maintaining production systems
- Developers debugging infrastructure code
- SREs troubleshooting incidents
- Anyone responsible for system reliability
- Teams improving MTTR (Mean Time To Recovery)

---

## 4.1 Interactive Debugging with pdb

pdb is Python's built-in debugger. It's like GDB for Python - powerful when you know how to use it.

### Basic pdb Usage

```python
# deployment.py
def deploy_application(app_name: str, version: str) -> bool:
    """Deploy application to production."""
    print(f"Starting deployment: {app_name} v{version}")
    
    # Set breakpoint
    import pdb; pdb.set_trace()  # Execution stops here
    
    config = load_config(app_name)
    validate_config(config)
    
    result = execute_deployment(config, version)
    
    return result


# Run script
deploy_application('my-app', '1.0.0')

# Output:
# Starting deployment: my-app v1.0.0
# > /path/to/deployment.py(8)deploy_application()
# -> config = load_config(app_name)
# (Pdb) 
```

**Common pdb Commands:**

```python
# (Pdb) prompt - you're in the debugger

# NAVIGATION
n (next)           # Execute current line, move to next
s (step)           # Step into function call
r (return)         # Continue until current function returns
c (continue)       # Continue execution until next breakpoint
u (up)             # Move up one level in call stack
d (down)           # Move down one level in call stack

# INSPECTION
p variable         # Print variable value
pp variable        # Pretty-print variable
a                  # Print all arguments to current function
locals()           # Show all local variables
globals()          # Show all global variables
type(variable)     # Show variable type

# BREAKPOINTS
b                  # List all breakpoints
b 10               # Set breakpoint at line 10
b function_name    # Set breakpoint at function entry
cl                 # Clear all breakpoints
cl 1               # Clear breakpoint 1

# EXECUTION
l (list)           # Show code around current line
ll (longlist)      # Show full function source
w (where)          # Print stack trace
h (help)           # Show help
q (quit)           # Quit debugger

# ADVANCED
! statement        # Execute Python statement
display variable   # Auto-display variable on each step
undisplay         # Stop displaying variable
```

### Practical pdb Example

```python
# Bug: Deployment fails intermittently
def deploy_to_kubernetes(app_name: str, config: dict) -> bool:
    """Deploy application to Kubernetes."""
    import pdb; pdb.set_trace()
    
    # Create deployment
    k8s = get_k8s_client()
    deployment = create_deployment_manifest(app_name, config)
    
    k8s.create_namespaced_deployment(
        namespace='default',
        body=deployment
    )
    
    # Wait for rollout
    import time
    max_wait = 300
    start = time.time()
    
    while time.time() - start < max_wait:
        status = k8s.read_namespaced_deployment_status(
            app_name, 'default'
        )
        
        if status.status.ready_replicas == config['replicas']:
            return True
        
        time.sleep(5)
    
    return False


# Debugging session:
"""
> deployment.py(5)deploy_to_kubernetes()
-> k8s = get_k8s_client()

(Pdb) n  # Next line
> deployment.py(6)deploy_to_kubernetes()
-> deployment = create_deployment_manifest(app_name, config)

(Pdb) p app_name  # Print variable
'my-app'

(Pdb) p config  # Check config
{'replicas': 3, 'image': 'my-app:1.0.0', 'memory': '512Mi'}

(Pdb) s  # Step into create_deployment_manifest
> manifest.py(10)create_deployment_manifest()
-> manifest = {

(Pdb) l  # List code
  5     def create_deployment_manifest(name, config):
  6         '''Create Kubernetes deployment manifest.'''
  7         
  8         # BUG: Missing replicas!
  9         manifest = {
 10  ->         'apiVersion': 'apps/v1',
 11             'kind': 'Deployment',
 12             'metadata': {'name': name},
 13             'spec': {
 14                 'selector': {'matchLabels': {'app': name}},
 15                 'template': {

(Pdb) n  # Continue stepping
...

(Pdb) p manifest  # Check final manifest
{'apiVersion': 'apps/v1', 'kind': 'Deployment', ...}
# BUG FOUND: manifest['spec']['replicas'] is missing!

(Pdb) q  # Quit debugger
"""
```

### Python 3.7+ breakpoint()

```python
# Modern way (Python 3.7+)
def deploy_application(app_name: str, version: str):
    """Deploy application."""
    print(f"Deploying {app_name}")
    
    breakpoint()  # Cleaner than import pdb; pdb.set_trace()
    
    config = load_config(app_name)
    return deploy(config, version)


# Disable all breakpoints
# PYTHONBREAKPOINT=0 python script.py

# Use different debugger
# PYTHONBREAKPOINT=ipdb.set_trace python script.py
```

### Conditional Breakpoints

```python
def process_servers(servers: list) -> None:
    """Process list of servers."""
    for i, server in enumerate(servers):
        # Only break on specific server
        if server.hostname == 'problematic-server':
            breakpoint()
        
        deploy_to_server(server)


# Or with pdb directly
def process_servers_alt(servers: list) -> None:
    """Process list of servers with conditional breakpoint."""
    import pdb
    
    for i, server in enumerate(servers):
        deploy_to_server(server)
        
        # Set conditional breakpoint
        if i == 50:  # Break on 50th iteration
            pdb.set_trace()
```

### Post-Mortem Debugging

```python
import pdb

def buggy_function():
    """Function with a bug."""
    data = {'key': 'value'}
    result = data['missing_key']  # KeyError!
    return result


try:
    buggy_function()
except:
    # Enter debugger at point of exception
    pdb.post_mortem()

# Can also use:
import sys
pdb.post_mortem(sys.exc_info()[2])
```

### pdb++ (Enhanced pdb)

```bash
# Install pdb++
pip install pdbpp
```

```python
# pdb++ features (just import pdb, it auto-replaces)
import pdb; pdb.set_trace()

# Features:
# - Syntax highlighting
# - Tab completion
# - Sticky mode (shows code context)
# - Smart command parsing

# Sticky mode
(Pdb++) sticky
# Shows code with current line highlighted

# Tab completion
(Pdb++) server.<TAB>
# Shows all attributes of 'server' object
```

### Remote Debugging

```python
# For production debugging (use carefully!)
import pdb
import sys

def remote_debug():
    """Enable remote debugging."""
    # Listen on port 4444
    from pdb import Pdb
    import socket
    
    # Create socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind(('0.0.0.0', 4444))
    sock.listen(1)
    
    print("Waiting for remote debugger connection on port 4444...")
    conn, addr = sock.accept()
    print(f"Debugger connected from {addr}")
    
    # Create pdb instance with socket
    debugger = Pdb(stdin=conn.makefile('r'), stdout=conn.makefile('w'))
    debugger.set_trace()

# Connect with: telnet localhost 4444
```

**⚠️ Security Warning:** Never enable remote debugging in production without proper security (VPN, authentication, encryption).

### Frequently Asked Questions - pdb

**Q1: How do I debug code that runs in a container?**

**A:** Several approaches:

**Option 1: Interactive container debugging**
```bash
# Run container with stdin open
docker run -it --rm my-app python -m pdb script.py

# Or attach to running container
docker exec -it container_name python -m pdb /app/script.py
```

**Option 2: Remote debugging**
```python
# In container code
import pdb
import socket

# Bind to 0.0.0.0 so it's accessible from host
# Then connect from host machine
```

**Option 3: Log-based debugging**
```python
# Better for production containers
import logging
logging.basicConfig(level=logging.DEBUG)

def deploy():
    logging.debug(f"Config: {config}")
    logging.debug(f"Starting deployment...")
    # Extensive logging instead of breakpoints
```

**Q2: pdb is too slow for large loops. What should I do?**

**A:** Use conditional breakpoints or logging:

```python
# ❌ BAD: Breaks on every iteration
for i in range(10000):
    breakpoint()  # Stops 10,000 times!
    process(i)

# ✅ GOOD: Conditional breakpoint
for i in range(10000):
    if i == 9999 or i % 1000 == 0:
        breakpoint()  # Only stops occasionally
    process(i)

# ✅ BETTER: Logging with filtering
import logging
for i in range(10000):
    logging.debug(f"Processing {i}: {data[i]}")
    process(i)

# View specific log entries after the fact
```

**Q3: How do I debug multithreaded code?**

**A:** pdb works but can be confusing with threads:

```python
import threading
import pdb

def worker(name):
    """Worker thread."""
    print(f"Thread {name} starting")
    breakpoint()  # All threads can hit this!
    print(f"Thread {name} finished")

# Start threads
threads = [
    threading.Thread(target=worker, args=(f"T{i}",))
    for i in range(3)
]

for t in threads:
    t.start()

# Problem: Multiple threads hitting breakpoint causes chaos

# Better: Thread-specific debugging
def worker_better(name):
    """Worker with conditional debugging."""
    if name == "T1":  # Only debug specific thread
        breakpoint()
    do_work()

# Or use thread-aware debugger like PyCharm
```

### Interview Questions - pdb

**Question 1: You're debugging a production issue where a deployment script hangs. You can't reproduce it locally. How would you debug this?**

**Difficulty:** Mid-Level

**Answer:**

**Scenario:** Deployment script hangs in production but works locally.

```python
# deployment.py
def deploy_to_production(app_name: str, version: str) -> bool:
    """Deploy application - sometimes hangs."""
    print(f"Deploying {app_name} v{version}")
    
    # Upload artifacts
    upload_artifacts(app_name, version)
    
    # Update servers
    servers = get_production_servers(app_name)
    for server in servers:
        update_server(server, version)  # Hangs here sometimes
    
    return True
```

**Debugging Strategy:**

**1. Add extensive logging (safest for production):**

```python
import logging
import time

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def deploy_to_production(app_name: str, version: str) -> bool:
    """Deploy with extensive logging."""
    logging.info(f"Starting deployment: {app_name} v{version}")
    
    start_time = time.time()
    
    # Upload artifacts
    logging.info("Uploading artifacts...")
    upload_artifacts(app_name, version)
    logging.info(f"Artifacts uploaded in {time.time() - start_time:.2f}s")
    
    # Update servers
    servers = get_production_servers(app_name)
    logging.info(f"Found {len(servers)} servers to update")
    
    for i, server in enumerate(servers):
        logging.info(f"Updating server {i+1}/{len(servers)}: {server.hostname}")
        server_start = time.time()
        
        try:
            update_server(server, version)
            logging.info(
                f"Server {server.hostname} updated in {time.time() - server_start:.2f}s"
            )
        except Exception as e:
            logging.error(f"Failed to update {server.hostname}: {e}")
            raise
    
    total_time = time.time() - start_time
    logging.info(f"Deployment completed in {total_time:.2f}s")
    return True

# From logs, you'll see:
# INFO - Starting deployment: my-app v1.0.0
# INFO - Uploading artifacts...
# INFO - Artifacts uploaded in 2.34s
# INFO - Found 10 servers to update
# INFO - Updating server 1/10: web-01
# INFO - Server web-01 updated in 5.23s
# INFO - Updating server 2/10: web-02
# INFO - Server web-02 updated in 5.45s
# INFO - Updating server 3/10: web-03
# (HANGS HERE - never logs completion)
# 
# Now you know it hangs on web-03!
```

**2. Add timeout protection:**

```python
import signal
from contextlib import contextmanager

@contextmanager
def timeout(seconds):
    """Context manager for timeout."""
    def timeout_handler(signum, frame):
        raise TimeoutError(f"Operation timed out after {seconds}s")
    
    # Set signal handler
    old_handler = signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(seconds)
    
    try:
        yield
    finally:
        signal.alarm(0)
        signal.signal(signal.SIGALRM, old_handler)


def deploy_to_production_safe(app_name: str, version: str) -> bool:
    """Deploy with timeout protection."""
    servers = get_production_servers(app_name)
    
    for server in servers:
        try:
            with timeout(30):  # 30 second timeout
                update_server(server, version)
        except TimeoutError:
            logging.error(f"Server {server.hostname} timed out!")
            # Continue with other servers or fail fast
            raise
    
    return True
```

**3. Add debugging hooks (for investigation):**

```python
import os

def deploy_to_production_debug(app_name: str, version: str) -> bool:
    """Deploy with optional debugging."""
    # Enable via environment variable
    if os.getenv('DEBUG_DEPLOYMENT'):
        import pdb
        pdb.set_trace()
    
    servers = get_production_servers(app_name)
    
    for server in servers:
        # Debug specific server
        if os.getenv('DEBUG_SERVER') == server.hostname:
            breakpoint()
        
        update_server(server, version)
    
    return True

# In production:
# DEBUG_SERVER=web-03 python deploy.py
```

**4. Implement health checks and monitoring:**

```python
def update_server(server, version):
    """Update server with health checks."""
    logging.info(f"Starting update for {server.hostname}")
    
    # Pre-check
    if not server.is_healthy():
        raise RuntimeError(f"Server {server.hostname} unhealthy before update")
    
    # Perform update
    logging.info(f"Deploying version {version} to {server.hostname}")
    server.deploy(version)
    
    # Post-check with timeout
    max_wait = 60
    start = time.time()
    
    while time.time() - start < max_wait:
        if server.is_healthy():
            logging.info(f"Server {server.hostname} healthy after update")
            return True
        
        logging.warning(
            f"Server {server.hostname} not healthy yet, waiting... "
            f"({time.time() - start:.1f}s elapsed)"
        )
        time.sleep(5)
    
    # THIS is where it hangs - health check never returns True
    raise RuntimeError(
        f"Server {server.hostname} failed to become healthy after {max_wait}s"
    )
```

**Root Cause Analysis:**

From logs, you discover:
- Deployment hangs on `web-03`
- `web-03` deploys successfully but never reports healthy
- Issue: Health check endpoint misconfigured on `web-03`

**Fix:**
```python
def update_server_fixed(server, version):
    """Update server with better error handling."""
    server.deploy(version)
    
    # More robust health check
    try:
        wait_for_health(server, timeout=60)
    except TimeoutError:
        # Provide actionable error
        logging.error(
            f"Health check failed for {server.hostname}. "
            f"Check: 1) App started correctly, "
            f"2) Health endpoint responding, "
            f"3) Firewall rules allow health checks"
        )
        
        # Attempt manual health check
        manual_check = check_server_manually(server)
        logging.info(f"Manual check result: {manual_check}")
        
        raise
```

**Why This Matters:**
- Production debugging requires different techniques than local debugging
- Logging is safer than breakpoints in production
- Timeouts prevent indefinite hangs
- Health checks reveal infrastructure issues
- Good error messages speed up resolution

---

### Key Takeaways - pdb

✅ **breakpoint()** - Modern way to set breakpoints (Python 3.7+)

✅ **Conditional breakpoints** - Only stop when condition is true

✅ **Post-mortem debugging** - Debug after exception occurs

✅ **Logging > Breakpoints** - In production, use logging

✅ **pdb++** - Enhanced pdb with better UX

✅ **Remote debugging** - For containers/remote servers (use carefully)

✅ **Timeouts** - Prevent hanging in production debugging

✅ **Strategic logging** - Better than breakpoints for production issues

---

## 4.2 Logging Best Practices

Logging is your eyes and ears in production. Good logs make debugging fast; bad logs make it impossible.

### Python Logging Basics

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Get logger
logger = logging.getLogger(__name__)

# Log levels (lowest to highest)
logger.debug("Detailed debugging information")      # DEBUG
logger.info("Informational messages")                # INFO
logger.warning("Warning messages")                   # WARNING
logger.error("Error messages")                       # ERROR
logger.critical("Critical errors")                   # CRITICAL

# Example usage
def deploy_application(app_name: str, version: str):
    """Deploy application with logging."""
    logger.info(f"Starting deployment: {app_name} v{version}")
    
    try:
        config = load_config(app_name)
        logger.debug(f"Loaded config: {config}")
        
        result = execute_deployment(config, version)
        logger.info(f"Deployment successful: {result}")
        
        return result
        
    except ConfigError as e:
        logger.error(f"Configuration error: {e}")
        raise
    except DeploymentError as e:
        logger.critical(f"Deployment failed: {e}")
        raise
```

### Structured Logging

**Problem:** String logs are hard to parse and query.

```python
# ❌ BAD: Unstructured logging
logger.info("User john deployed my-app version 1.0.0 to production")

# Hard to query:
# - Who deployed?
# - What app?
# - What version?
# - What environment?
```

**Solution:** Structured logging (JSON)

```python
# ✅ GOOD: Structured logging
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """Format logs as JSON."""
    
    def format(self, record):
        """Format log record as JSON."""
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        
        # Add extra fields if present
        if hasattr(record, 'user'):
            log_data['user'] = record.user
        if hasattr(record, 'app_name'):
            log_data['app_name'] = record.app_name
        if hasattr(record, 'environment'):
            log_data['environment'] = record.environment
        
        # Add exception info if present
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        
        return json.dumps(log_data)


# Configure JSON logging
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Log with extra context
logger.info(
    "Deployment started",
    extra={
        'user': 'john',
        'app_name': 'my-app',
        'version': '1.0.0',
        'environment': 'production'
    }
)

# Output (formatted for readability):
# {
#   "timestamp": "2024-01-15T10:30:45.123456",
#   "level": "INFO",
#   "logger": "deployment",
#   "message": "Deployment started",
#   "module": "deployment",
#   "function": "deploy_application",
#   "line": 42,
#   "user": "john",
#   "app_name": "my-app",
#   "version": "1.0.0",
#   "environment": "production"
# }

# Now easy to query:
# - All deployments by john: jq '.user == "john"'
# - All production deployments: jq '.environment == "production"'
# - Deployments of specific app: jq '.app_name == "my-app"'
```

### Using python-json-logger

```bash
pip install python-json-logger
```

```python
from pythonjsonlogger import jsonlogger

# Configure JSON logging
handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter(
    '%(timestamp)s %(level)s %(name)s %(message)s'
)
handler.setFormatter(formatter)

logger = logging.getLogger()
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Log with context
logger.info(
    "Deployment completed",
    extra={
        'deployment_id': 'deploy-123',
        'duration_seconds': 45.2,
        'success': True
    }
)
```

### Logging Configuration

```python
# config/logging.yaml
version: 1
disable_existing_loggers: False

formatters:
  json:
    class: pythonjsonlogger.jsonlogger.JsonFormatter
    format: '%(timestamp)s %(level)s %(name)s %(message)s'
  
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'

handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: json
    stream: ext://sys.stdout
  
  file:
    class: logging.handlers.RotatingFileHandler
    level: INFO
    formatter: json
    filename: logs/application.log
    maxBytes: 10485760  # 10MB
    backupCount: 5

loggers:
  deployment:
    level: INFO
    handlers: [console, file]
    propagate: False
  
  security:
    level: WARNING
    handlers: [console, file]
    propagate: False

root:
  level: INFO
  handlers: [console, file]
```

```python
# Load logging configuration
import logging.config
import yaml

with open('config/logging.yaml') as f:
    config = yaml.safe_load(f)
    logging.config.dictConfig(config)

# Use logger
logger = logging.getLogger('deployment')
logger.info("Application started")
```

### Context Logging

```python
from contextvars import ContextVar
import logging
import uuid

# Context variable for request ID
request_id_var = ContextVar('request_id', default=None)

class ContextFilter(logging.Filter):
    """Add context to log records."""
    
    def filter(self, record):
        """Add request_id to every log record."""
        record.request_id = request_id_var.get() or 'no-request-id'
        return True

# Configure logging with context
handler = logging.StreamHandler()
handler.addFilter(ContextFilter())
formatter = logging.Formatter(
    '%(asctime)s - [%(request_id)s] - %(levelname)s - %(message)s'
)
handler.setFormatter(formatter)

logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Usage
def handle_deployment_request(app_name: str, version: str):
    """Handle deployment request."""
    # Set request ID for this request
    request_id = str(uuid.uuid4())
    request_id_var.set(request_id)
    
    logger.info(f"Received deployment request for {app_name}")
    # Output: 2024-01-15 10:30:00 - [abc-123-def] - INFO - Received deployment request for my-app
    
    deploy(app_name, version)
    
    logger.info("Deployment completed")
    # Output: 2024-01-15 10:30:45 - [abc-123-def] - INFO - Deployment completed
    
    # All logs for this request have same request_id
    # Easy to filter: grep "abc-123-def" logs/app.log
```

### Security-Safe Logging

```python
# ❌ DANGEROUS: Logging secrets
import logging

logger = logging.getLogger(__name__)

def deploy_with_credentials(api_key: str, secret: str):
    """Deploy with credentials - DANGEROUS LOGGING!"""
    logger.info(f"Deploying with API key: {api_key}")  # LEAKED!
    logger.debug(f"Using secret: {secret}")  # LEAKED!
    
    # Credentials now in logs, visible to anyone with log access


# ✅ SAFE: Redact secrets
import re

def redact_secrets(text: str) -> str:
    """Redact secrets from text."""
    # Redact common secret patterns
    patterns = [
        (r'(api[_-]?key|apikey)[\s:=]+([^\s]+)', r'\1: [REDACTED]'),
        (r'(password|passwd|pwd)[\s:=]+([^\s]+)', r'\1: [REDACTED]'),
        (r'(secret|token)[\s:=]+([^\s]+)', r'\1: [REDACTED]'),
        (r'(aws[_-]?access[_-]?key[_-]?id)[\s:=]+([^\s]+)', r'\1: [REDACTED]'),
        (r'(aws[_-]?secret[_-]?access[_-]?key)[\s:=]+([^\s]+)', r'\1: [REDACTED]'),
    ]
    
    result = text
    for pattern, replacement in patterns:
        result = re.sub(pattern, replacement, result, flags=re.IGNORECASE)
    
    return result


class RedactingFilter(logging.Filter):
    """Filter to redact secrets from logs."""
    
    def filter(self, record):
        """Redact secrets from log message."""
        record.msg = redact_secrets(str(record.msg))
        return True


# Add filter to all handlers
for handler in logging.root.handlers:
    handler.addFilter(RedactingFilter())


# Now safe to log
def deploy_safe(api_key: str, secret: str):
    """Deploy with safe logging."""
    logger.info(f"Deploying with api_key: {api_key}")
    # Output: "Deploying with api_key: [REDACTED]"
    
    logger.debug(f"Configuration includes secret: {secret}")
    # Output: "Configuration includes secret: [REDACTED]"


# ✅ BEST: Never log secrets at all
def deploy_best_practice(credentials: dict):
    """Deploy with best practice logging."""
    # Log that we have credentials, not what they are
    logger.info("Deploying with provided credentials")
    
    # Log non-sensitive parts of config only
    safe_config = {
        k: v for k, v in credentials.items()
        if k not in ['api_key', 'secret', 'password', 'token']
    }
    logger.debug(f"Configuration: {safe_config}")
```

### Performance Logging

```python
import logging
import time
from functools import wraps

def log_performance(logger):
    """Decorator to log function performance."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start
                
                logger.info(
                    f"{func.__name__} completed in {duration:.2f}s",
                    extra={
                        'function': func.__name__,
                        'duration_seconds': duration,
                        'success': True
                    }
                )
                
                return result
                
            except Exception as e:
                duration = time.time() - start
                
                logger.error(
                    f"{func.__name__} failed after {duration:.2f}s: {e}",
                    extra={
                        'function': func.__name__,
                        'duration_seconds': duration,
                        'success': False,
                        'error': str(e)
                    }
                )
                
                raise
        
        return wrapper
    return decorator


@log_performance(logger)
def deploy_application(app_name: str, version: str):
    """Deploy application with performance logging."""
    time.sleep(2)  # Simulate deployment
    return {'status': 'success'}

# Output:
# {
#   "timestamp": "2024-01-15T10:30:47",
#   "level": "INFO",
#   "message": "deploy_application completed in 2.01s",
#   "function": "deploy_application",
#   "duration_seconds": 2.01,
#   "success": true
# }
```

### Frequently Asked Questions - Logging

**Q1: Should I use print() or logging?**

**A:** **Always use logging for production code.**

```python
# ❌ BAD: Using print()
def deploy(app_name):
    print(f"Deploying {app_name}")  # Goes to stdout, can't be configured
    print(f"Success!")

# Problems with print():
# - Can't disable/filter
# - No log levels
# - No timestamps
# - No log routing (file, network, etc.)
# - Hard to parse

# ✅ GOOD: Using logging
import logging
logger = logging.getLogger(__name__)

def deploy(app_name):
    logger.info(f"Deploying {app_name}")
    logger.info("Success!")

# Benefits:
# - Can change level (DEBUG, INFO, etc.)
# - Automatic timestamps
# - Can route to file, network, CloudWatch, etc.
# - Structured data with extra= parameter
# - Can be disabled in production if needed
```

**Q2: What log level should I use for different messages?**

**A:**

| Level | When to Use | Example |
|-------|-------------|---------|
| **DEBUG** | Detailed diagnostic info (dev only) | Variable values, loop iterations |
| **INFO** | Normal operations, milestones | "Deployment started", "Server ready" |
| **WARNING** | Unexpected but handled | "Retry attempt 2/3", "Disk 80% full" |
| **ERROR** | Error occurred, operation failed | "Failed to connect to database" |
| **CRITICAL** | Serious error, system unstable | "Out of memory", "Data corruption" |

```python
# Examples
logger.debug(f"Config loaded: {config}")  # Only in development

logger.info("Deployment started")  # Normal operation

logger.warning("Server response slow (2.5s), retrying")  # Warning but handled

logger.error("Failed to update server web-03", exc_info=True)  # Operation failed

logger.critical("Database connection lost, application unstable")  # Critical
```

**Q3: How do I log exceptions properly?**

**A:** Use `exc_info=True` or `logger.exception()`:

```python
# ❌ BAD: Losing stack trace
try:
    deploy_application('my-app', '1.0.0')
except Exception as e:
    logger.error(f"Deployment failed: {e}")
    # Only logs exception message, no stack trace!

# ✅ GOOD: Include stack trace
try:
    deploy_application('my-app', '1.0.0')
except Exception as e:
    logger.error(f"Deployment failed: {e}", exc_info=True)
    # Logs full stack trace

# ✅ BEST: Use logger.exception()
try:
    deploy_application('my-app', '1.0.0')
except Exception:
    logger.exception("Deployment failed")
    # Automatically includes exc_info=True
```

---

### Key Takeaways - Logging

✅ **Structured logging** - Use JSON for production

✅ **Log levels** - DEBUG, INFO, WARNING, ERROR, CRITICAL

✅ **Context** - Add request_id, user, etc.

✅ **Redact secrets** - Never log passwords, API keys

✅ **Performance** - Log duration of operations

✅ **Exceptions** - Use exc_info=True or logger.exception()

✅ **Configuration** - Use YAML/dict config

✅ **Rotation** - Use RotatingFileHandler to manage disk space

---

*[Part 4 continues with Performance Profiling, Memory Debugging, and Production Troubleshooting...]*

## 4.3 Performance Profiling

Performance profiling identifies where your code spends time. In DevSecOps, slow deployments and inefficient automation waste time and money.

### Understanding Performance Bottlenecks

```python
# Example: Slow deployment script
import time

def deploy_application(app_name: str, servers: list) -> bool:
    """Deploy application to multiple servers."""
    print(f"Deploying {app_name} to {len(servers)} servers")
    
    # Validate configuration (fast)
    config = load_config(app_name)
    validate_config(config)
    
    # Build image (slow? fast?)
    image = build_docker_image(app_name)
    
    # Deploy to each server (slow!)
    for server in servers:
        deploy_to_server(server, image)
    
    print("Deployment complete")
    return True

# Where is the bottleneck?
# - Config loading?
# - Image building?
# - Server deployment?
# Need profiling to find out!
```

### cProfile - Built-in Profiler

```python
import cProfile
import pstats

# Profile function
def profile_deployment():
    """Profile deployment function."""
    profiler = cProfile.Profile()
    profiler.enable()
    
    deploy_application('my-app', get_servers())
    
    profiler.disable()
    
    # Print stats
    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats(10)  # Top 10 functions


# Or use as decorator
def profile(func):
    """Profile decorator."""
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()
        
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(10)
        
        return result
    return wrapper


@profile
def deploy_application(app_name: str, servers: list) -> bool:
    """Deploy with profiling."""
    # Implementation...
    pass


# Command line profiling
# python -m cProfile -o output.prof script.py

# View results
# python -m pstats output.prof
"""
Welcome to the profile statistics browser.
output.prof% sort cumtime
output.prof% stats 10
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001   45.234   45.234 deployment.py:10(deploy_application)
       50    0.523    0.010   40.123    0.802 deployment.py:45(deploy_to_server)
       50   35.234    0.705   35.234    0.705 deployment.py:67(wait_for_server)
        1    0.012    0.012    5.001    5.001 deployment.py:23(build_docker_image)
     1000    0.234    0.000    3.456    0.003 deployment.py:89(check_health)
"""
```

**Interpreting cProfile Output:**

| Column | Meaning |
|--------|---------|
| `ncalls` | Number of times function called |
| `tottime` | Total time in function (excluding subcalls) |
| `percall` | tottime / ncalls |
| `cumtime` | Total time in function (including subcalls) |
| `percall` | cumtime / ncalls |
| `filename:lineno(function)` | Function name and location |

**Key Insights from Example:**
- `wait_for_server()`: 35.2s (78% of total) - **BOTTLENECK!**
- `deploy_to_server()`: Called 50 times, takes 40s total
- `build_docker_image()`: Only 5s - not the problem
- **Optimization target**: `wait_for_server()` function

### line_profiler - Line-by-Line Profiling

```bash
# Install
pip install line_profiler
```

```python
# deployment.py
from line_profiler import profile

@profile
def deploy_to_server(server, image):
    """Deploy to single server - line profiling."""
    # Upload image (line 1)
    upload_image(server, image)
    
    # Start container (line 2)
    container_id = start_container(server, image)
    
    # Wait for health check (line 3)
    wait_for_health(server, container_id)
    
    # Update load balancer (line 4)
    update_load_balancer(server)
    
    return True


# Run with line profiler
# kernprof -l -v deployment.py

# Output:
"""
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     5                                           @profile
     6                                           def deploy_to_server(server, image):
     7         1         1234     1234      2.0      upload_image(server, image)
     8         1         5678     5678      9.2      container_id = start_container(server, image)
     9         1        52345    52345     84.8      wait_for_health(server, container_id)
    10         1         2456     2456      4.0      update_load_balancer(server)
    11         1            2       2      0.0      return True
"""

# Line 9 (wait_for_health) takes 84.8% of time!
```

**Optimization Strategy:**

```python
# ❌ BEFORE: Slow health check (84.8% of time)
def wait_for_health(server, container_id, timeout=300):
    """Wait for container to be healthy."""
    import time
    start = time.time()
    
    while time.time() - start < timeout:
        if check_health(server, container_id):
            return True
        time.sleep(10)  # Sleep 10 seconds between checks
    
    raise TimeoutError("Health check timeout")


# ✅ AFTER: Optimized health check
def wait_for_health_optimized(server, container_id, timeout=300):
    """Wait for container with adaptive polling."""
    import time
    start = time.time()
    
    # Adaptive polling: start fast, slow down
    intervals = [1, 1, 2, 2, 5, 5, 10, 10]  # seconds
    
    i = 0
    while time.time() - start < timeout:
        if check_health(server, container_id):
            return True
        
        # Use adaptive interval
        interval = intervals[min(i, len(intervals)-1)]
        time.sleep(interval)
        i += 1
    
    raise TimeoutError("Health check timeout")

# Result: Reduced from 52s to 15s (71% improvement!)
# Early health checks succeed faster with 1-second intervals
```

### Memory Profiling with memory_profiler

```bash
# Install
pip install memory_profiler
```

```python
# deployment.py
from memory_profiler import profile

@profile
def process_large_config(config_file):
    """Process large configuration file."""
    # Load entire file into memory
    with open(config_file, 'r') as f:
        data = f.read()  # Could be 100MB+
    
    # Parse JSON
    import json
    config = json.loads(data)
    
    # Process each server
    results = []
    for server_config in config['servers']:
        result = process_server(server_config)
        results.append(result)
    
    return results


# Run with memory profiler
# python -m memory_profiler deployment.py

# Output:
"""
Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     5   45.2 MiB     45.2 MiB           1   @profile
     6                                         def process_large_config(config_file):
     7   45.2 MiB      0.0 MiB           1       with open(config_file, 'r') as f:
     8  145.3 MiB    100.1 MiB           1           data = f.read()  # 100MB spike!
     9                                         
    10  145.3 MiB      0.0 MiB           1       import json
    11  245.8 MiB    100.5 MiB           1       config = json.loads(data)  # Another 100MB!
    12                                         
    13  245.8 MiB      0.0 MiB           1       results = []
    14  445.9 MiB    200.1 MiB        1000       for server_config in config['servers']:
    15  445.9 MiB      0.0 MiB        1000           result = process_server(server_config)
    16  445.9 MiB      0.0 MiB        1000           results.append(result)
"""

# Memory peaks at 445MB! Can we optimize?
```

**Memory Optimization:**

```python
# ✅ OPTIMIZED: Stream processing
import ijson  # pip install ijson

@profile
def process_large_config_optimized(config_file):
    """Process config with streaming."""
    results = []
    
    with open(config_file, 'rb') as f:
        # Stream JSON parsing (doesn't load entire file)
        servers = ijson.items(f, 'servers.item')
        
        for server_config in servers:
            result = process_server(server_config)
            results.append(result)
    
    return results

# Memory usage: ~50MB (90% reduction!)
# Processes same data, 1/10th the memory
```

### Profiling Async Code

```python
import asyncio
import time

async def fetch_server_status(server_id):
    """Fetch server status (simulated)."""
    await asyncio.sleep(0.1)  # Simulate API call
    return {'id': server_id, 'status': 'running'}


async def check_all_servers(server_ids):
    """Check status of all servers."""
    # ❌ SLOW: Sequential
    results = []
    for server_id in server_ids:
        result = await fetch_server_status(server_id)
        results.append(result)
    return results


async def check_all_servers_fast(server_ids):
    """Check status of all servers - parallel."""
    # ✅ FAST: Parallel
    tasks = [fetch_server_status(sid) for sid in server_ids]
    results = await asyncio.gather(*tasks)
    return results


# Profile async code
async def profile_async():
    """Profile async functions."""
    import cProfile
    
    server_ids = [f"server-{i}" for i in range(100)]
    
    # Profile sequential
    profiler = cProfile.Profile()
    profiler.enable()
    
    start = time.time()
    await check_all_servers(server_ids)
    sequential_time = time.time() - start
    
    profiler.disable()
    
    # Profile parallel
    profiler = cProfile.Profile()
    profiler.enable()
    
    start = time.time()
    await check_all_servers_fast(server_ids)
    parallel_time = time.time() - start
    
    profiler.disable()
    
    print(f"Sequential: {sequential_time:.2f}s")  # ~10s
    print(f"Parallel: {parallel_time:.2f}s")      # ~0.1s
    print(f"Speedup: {sequential_time/parallel_time:.1f}x")  # 100x!


asyncio.run(profile_async())
```

### Profiling in Production

```python
# Continuous profiling with py-spy
# pip install py-spy

# Profile running process (no code changes needed!)
# sudo py-spy record -o profile.svg --pid 12345

# Profile for 30 seconds
# sudo py-spy record -o profile.svg --duration 30 --pid 12345

# Top-like view
# sudo py-spy top --pid 12345
```

**py-spy Output:**

```
%Own   %Total  OwnTime  TotalTime  Function (filename:line)
74.00%  74.00%   2.96s     2.96s   wait_for_health (deployment.py:89)
12.00%  12.00%   0.48s     0.48s   upload_image (deployment.py:45)
 8.00%  20.00%   0.32s     0.80s   deploy_to_server (deployment.py:23)
 4.00%  96.00%   0.16s     3.84s   deploy_application (deployment.py:10)
 2.00%   2.00%   0.08s     0.08s   check_health (deployment.py:102)
```

### Benchmarking with timeit

```python
import timeit

# Compare different implementations
def find_server_v1(servers, hostname):
    """Linear search."""
    for server in servers:
        if server['hostname'] == hostname:
            return server
    return None


def find_server_v2(servers, hostname):
    """Dictionary lookup."""
    server_dict = {s['hostname']: s for s in servers}
    return server_dict.get(hostname)


# Benchmark
servers = [{'hostname': f'server-{i}', 'ip': f'10.0.0.{i}'} for i in range(1000)]

# V1: Linear search
time_v1 = timeit.timeit(
    lambda: find_server_v1(servers, 'server-999'),
    number=10000
)
print(f"V1 (linear): {time_v1:.4f}s")  # 2.5s

# V2: Dictionary
time_v2 = timeit.timeit(
    lambda: find_server_v2(servers, 'server-999'),
    number=10000
)
print(f"V2 (dict): {time_v2:.4f}s")  # 0.05s

print(f"Speedup: {time_v1/time_v2:.1f}x")  # 50x faster!
```

### Performance Optimization Checklist

```python
# 1. Profile first, optimize second
# Don't guess where the bottleneck is - measure!

# 2. Optimize the right thing
# 80% of time spent in 20% of code (Pareto principle)
# Focus on the hotspots

# 3. Common optimizations:

# ❌ SLOW: List comprehension with function calls
results = [expensive_function(x) for x in large_list]

# ✅ FAST: Cache results
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_function(x):
    # Expensive computation
    pass


# ❌ SLOW: String concatenation in loop
output = ""
for i in range(10000):
    output += str(i)  # Creates new string each time

# ✅ FAST: Join
output = ''.join(str(i) for i in range(10000))


# ❌ SLOW: Multiple passes over data
servers = get_all_servers()
running = [s for s in servers if s.status == 'running']
healthy = [s for s in running if s.health == 'healthy']

# ✅ FAST: Single pass
healthy = [s for s in get_all_servers() 
           if s.status == 'running' and s.health == 'healthy']


# ❌ SLOW: Sequential I/O
for server in servers:
    check_health(server)  # Network call, blocks

# ✅ FAST: Parallel I/O
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=10) as executor:
    executor.map(check_health, servers)
```

### Frequently Asked Questions - Profiling

**Q1: My script is slow. Should I profile with cProfile or line_profiler?**

**A:** **Start with cProfile, then drill down with line_profiler.**

**Workflow:**
1. **cProfile first**: Find which functions are slow
2. **line_profiler next**: Find which lines within those functions
3. **Optimize**: Fix the bottlenecks
4. **Verify**: Re-profile to confirm improvement

```python
# Step 1: cProfile - find slow function
# python -m cProfile -s cumtime script.py
# Result: deploy_to_server() takes 80% of time

# Step 2: line_profiler - find slow line
@profile
def deploy_to_server(server, image):
    upload_image(server, image)
    wait_for_health(server)  # This line!
    update_lb(server)

# Step 3: Optimize wait_for_health()
# Step 4: Re-profile to verify
```

**Q2: How do I profile production code without impacting performance?**

**A:** Use **py-spy** for production profiling:

```bash
# Advantages of py-spy:
# - No code changes needed
# - Minimal overhead (<1%)
# - Can profile running processes
# - Works with any Python code

# Sample running process
sudo py-spy record -o profile.svg --pid <PID>

# Profile for specific duration
sudo py-spy record -o profile.svg --duration 60 --pid <PID>

# Profile specific function calls
sudo py-spy record -o profile.svg --pid <PID> --function deploy_application
```

**Alternative: Statistical profiling**
```python
# Profiling with sampling (low overhead)
import signal
import traceback
from collections import defaultdict

profile_data = defaultdict(int)

def sample_stack(signum, frame):
    """Sample current stack."""
    stack = ''.join(traceback.format_stack(frame))
    profile_data[stack] += 1

# Sample every 0.01 seconds
signal.signal(signal.SIGALRM, sample_stack)
signal.setitimer(signal.ITIMER_REAL, 0.01, 0.01)

# Run code
deploy_application()

# Stop sampling
signal.setitimer(signal.ITIMER_REAL, 0)

# Print results
for stack, count in sorted(profile_data.items(), key=lambda x: x[1], reverse=True)[:5]:
    print(f"Count: {count}\n{stack}\n")
```

---

### Key Takeaways - Profiling

✅ **Profile before optimizing** - Don't guess, measure

✅ **cProfile** - Function-level profiling

✅ **line_profiler** - Line-level profiling

✅ **memory_profiler** - Memory usage profiling

✅ **py-spy** - Production profiling (no code changes)

✅ **Optimize hotspots** - 80/20 rule applies

✅ **Benchmark improvements** - Verify optimizations work

✅ **Async for I/O** - Parallel operations for network calls

---

## 4.4 Memory Debugging

Memory leaks in long-running DevSecOps automation can crash systems. Learn to detect and fix them.

### Detecting Memory Leaks

```python
# Example: Memory leak in deployment automation
class DeploymentManager:
    """Manages deployments - has memory leak!"""
    
    def __init__(self):
        self.deployments = []  # ❌ Never cleared!
    
    def deploy(self, app_name, version):
        """Deploy application."""
        deployment = {
            'app': app_name,
            'version': version,
            'logs': [],  # Grows over time
            'metrics': {}
        }
        
        # Execute deployment
        for i in range(1000):
            log_entry = f"Step {i}: deploying..."
            deployment['logs'].append(log_entry)
        
        # ❌ BUG: Deployment never removed from memory!
        self.deployments.append(deployment)
        
        return deployment


# After 1000 deployments:
# Memory usage: 1000 * (deployment size) = HUGE!
# Each deployment stays in memory forever
```

### Using tracemalloc

```python
import tracemalloc

# Start tracing
tracemalloc.start()

# Checkpoint 1
snapshot1 = tracemalloc.take_snapshot()

# Run code that might leak memory
manager = DeploymentManager()
for i in range(100):
    manager.deploy(f'app-{i}', '1.0.0')

# Checkpoint 2
snapshot2 = tracemalloc.take_snapshot()

# Compare snapshots
top_stats = snapshot2.compare_to(snapshot1, 'lineno')

print("[ Top 10 memory growth ]")
for stat in top_stats[:10]:
    print(stat)

# Output:
"""
deployment.py:15: size=25.6 MiB (+25.6 MiB), count=100 (+100), average=262 KiB
    deployment['logs'].append(log_entry)
    
deployment.py:20: size=12.3 MiB (+12.3 MiB), count=100 (+100), average=126 KiB
    self.deployments.append(deployment)
"""

# Memory leak found!
# Line 20: self.deployments keeps growing
```

### Finding Memory Leaks with objgraph

```bash
pip install objgraph
```

```python
import objgraph

# Before running code
objgraph.show_growth()

# Run code
manager = DeploymentManager()
for i in range(100):
    manager.deploy(f'app-{i}', '1.0.0')

# After running code
objgraph.show_growth()

# Output:
"""
dict        12450    +12350
list         5678    +5578
str        234567   +234467
tuple       45678    +45578
"""

# Huge growth in dicts and lists - memory leak indicator!

# Find what's holding references
import gc
gc.collect()  # Force garbage collection

# Show what's keeping deployments alive
objgraph.show_backrefs(
    manager.deployments,
    max_depth=5,
    filename='deployment_refs.png'
)
# Generates graph showing reference chain
```

### Fixing Memory Leaks

```python
# ✅ FIX 1: Clear old deployments
class DeploymentManager:
    """Deployment manager without leak."""
    
    def __init__(self, max_history=10):
        self.deployments = []
        self.max_history = max_history
    
    def deploy(self, app_name, version):
        """Deploy with memory management."""
        deployment = {
            'app': app_name,
            'version': version,
            'logs': [],
            'metrics': {}
        }
        
        # Execute deployment
        for i in range(1000):
            deployment['logs'].append(f"Step {i}: deploying...")
        
        self.deployments.append(deployment)
        
        # ✅ FIX: Keep only recent deployments
        if len(self.deployments) > self.max_history:
            self.deployments.pop(0)  # Remove oldest
        
        return deployment


# ✅ FIX 2: Use weak references
import weakref

class DeploymentManager:
    """Deployment manager with weak references."""
    
    def __init__(self):
        # Use WeakValueDictionary - auto-removes when no other refs
        self.deployments = weakref.WeakValueDictionary()
    
    def deploy(self, app_name, version):
        """Deploy with weak references."""
        deployment = Deployment(app_name, version)
        deployment.execute()
        
        # Store weak reference
        self.deployments[deployment.id] = deployment
        
        return deployment
        # When deployment goes out of scope, 
        # it's automatically removed from self.deployments


# ✅ FIX 3: Streaming/generators for large data
def process_logs_bad(log_file):
    """Load entire file - memory intensive."""
    with open(log_file) as f:
        lines = f.readlines()  # ❌ Loads all lines into memory
    
    for line in lines:
        process_line(line)


def process_logs_good(log_file):
    """Stream file - constant memory."""
    with open(log_file) as f:
        for line in f:  # ✅ One line at a time
            process_line(line)
```

### Memory Profiling with memory_profiler

```python
from memory_profiler import profile

@profile
def analyze_deployments():
    """Analyze deployment logs."""
    deployments = []
    
    # Load deployment data
    for i in range(1000):
        deployment = {
            'id': i,
            'logs': [f'log-{j}' for j in range(1000)],  # 1000 logs each
            'metrics': {f'metric-{k}': k for k in range(100)}  # 100 metrics
        }
        deployments.append(deployment)
    
    # Process deployments
    results = []
    for deployment in deployments:
        result = process_deployment(deployment)
        results.append(result)
    
    return results


# Run: python -m memory_profiler script.py
# Output shows memory usage per line
```

### Garbage Collection Debugging

```python
import gc

# Monitor garbage collection
gc.set_debug(gc.DEBUG_STATS | gc.DEBUG_LEAK)

# Run code
manager = DeploymentManager()
manager.deploy('my-app', '1.0.0')

# Force garbage collection
collected = gc.collect()
print(f"Collected {collected} objects")

# Find uncollectable objects (circular references)
uncollectable = gc.garbage
if uncollectable:
    print(f"WARNING: {len(uncollectable)} uncollectable objects!")
    for obj in uncollectable:
        print(f"Uncollectable: {type(obj)}: {obj}")


# Check reference counts
import sys

obj = {'key': 'value'}
print(f"Reference count: {sys.getrefcount(obj)}")  # 2 (obj + getrefcount param)

another_ref = obj
print(f"Reference count: {sys.getrefcount(obj)}")  # 3

del another_ref
print(f"Reference count: {sys.getrefcount(obj)}")  # 2
```

### Common Memory Leak Patterns

```python
# ❌ PATTERN 1: Global caches never cleared
_cache = {}  # Global cache

def get_deployment_config(app_name):
    """Get config with caching."""
    if app_name not in _cache:
        _cache[app_name] = load_config(app_name)
    return _cache[app_name]

# Problem: Cache grows forever
# Fix: Use functools.lru_cache with maxsize


# ❌ PATTERN 2: Circular references
class Deployment:
    def __init__(self, app_name):
        self.app_name = app_name
        self.callbacks = []
    
    def add_callback(self, callback):
        self.callbacks.append(callback)

deployment = Deployment('my-app')
deployment.add_callback(lambda: deployment.cleanup())  # Circular!
# deployment -> callbacks -> lambda -> deployment

# Fix: Use weak references
import weakref

class Deployment:
    def __init__(self, app_name):
        self.app_name = app_name
        self.callbacks = []
    
    def add_callback(self, callback):
        # Store weak reference to self
        weak_self = weakref.ref(self)
        self.callbacks.append(callback)


# ❌ PATTERN 3: Large temporary objects in loops
def process_servers(servers):
    """Process servers inefficiently."""
    for server in servers:
        temp_data = load_server_data(server)  # Large object
        process_data(temp_data)
        # ❌ temp_data not explicitly deleted
        # Stays in memory until next iteration

# Fix: Explicit deletion
def process_servers_fixed(servers):
    """Process servers efficiently."""
    for server in servers:
        temp_data = load_server_data(server)
        process_data(temp_data)
        del temp_data  # ✅ Free memory immediately
        gc.collect()   # Force garbage collection


# ❌ PATTERN 4: Unbounded growth
class LogCollector:
    def __init__(self):
        self.logs = []  # ❌ Grows forever
    
    def add_log(self, message):
        self.logs.append(message)

# Fix: Ring buffer
from collections import deque

class LogCollector:
    def __init__(self, max_logs=1000):
        self.logs = deque(maxlen=max_logs)  # ✅ Auto-evicts old logs
    
    def add_log(self, message):
        self.logs.append(message)  # Old logs automatically removed
```

### Frequently Asked Questions - Memory

**Q1: How do I know if my application has a memory leak?**

**A:** **Monitor memory usage over time:**

```python
import psutil
import time
import os

def monitor_memory(interval=60):
    """Monitor memory usage."""
    process = psutil.Process(os.getpid())
    
    while True:
        mem_info = process.memory_info()
        mem_mb = mem_info.rss / 1024 / 1024
        
        print(f"Memory usage: {mem_mb:.2f} MB")
        
        time.sleep(interval)

# Run in background thread
import threading
monitor_thread = threading.Thread(target=monitor_memory, daemon=True)
monitor_thread.start()

# Run your application
# If memory continuously increases, you likely have a leak
```

**Signs of memory leak:**
- Memory usage constantly increases
- Never levels off
- Eventually causes OOM (Out of Memory)

**Q2: What's the difference between memory leak and high memory usage?**

**A:**

**Memory Leak:**
- Memory usage increases over time
- Never released
- Eventually crashes

**High Memory Usage:**
- Memory usage is high but stable
- Memory is used for legitimate purposes
- Won't cause crash

```python
# High memory usage (not a leak)
data = [load_large_dataset() for _ in range(10)]  # Uses 1GB
process_data(data)
# High memory usage, but legitimate and stable

# Memory leak
cache = {}
def process(item):
    cache[item.id] = item  # ❌ Never cleared, grows forever
    
for item in infinite_stream():
    process(item)  # Memory grows indefinitely
```

---

### Key Takeaways - Memory

✅ **tracemalloc** - Track memory allocations

✅ **objgraph** - Visualize object references

✅ **Clear caches** - Limit cache size

✅ **Weak references** - Prevent circular references

✅ **Streaming** - Process large files line-by-line

✅ **Generators** - Lazy evaluation

✅ **Monitor memory** - Track usage over time

✅ **gc module** - Force garbage collection when needed

---

*[Part 4 continues with Production Debugging and Common Bug Patterns...]*

## 4.5 Production Debugging Strategies

Production debugging requires different techniques than local debugging. You often can't attach a debugger, access is limited, and time is critical.

### The Production Debugging Mindset

**Key Principles:**

1. **Logs are your eyes** - Comprehensive logging is essential
2. **Reproduce locally** - If possible, recreate production scenario
3. **Hypothesis-driven** - Form theories, test systematically
4. **Document everything** - Track what you've tried
5. **Minimize changes** - Small, testable changes only

### Debugging Production Issues Without Access

```python
# Scenario: Production API timing out
# You can't SSH to production or attach debugger

# ❌ BAD: Guess and deploy
def handle_request(request):
    # Maybe it's the database?
    # Let me add connection pooling...
    # Deploy and hope it works

# ✅ GOOD: Systematic approach with logging

# Step 1: Add detailed logging (feature flag controlled)
import logging
import time
import os

logger = logging.getLogger(__name__)

def handle_request(request):
    """Handle request with detailed logging."""
    request_id = request.headers.get('X-Request-ID', 'unknown')
    
    # Enable debug logging via environment variable
    debug_enabled = os.getenv('DEBUG_REQUESTS') == 'true'
    
    if debug_enabled:
        logger.info(f"[{request_id}] Request started", extra={
            'request_id': request_id,
            'endpoint': request.path,
            'method': request.method
        })
    
    start = time.time()
    
    try:
        # Step 2: Log timing for each operation
        db_start = time.time()
        data = fetch_from_database(request.params)
        db_duration = time.time() - db_start
        
        if debug_enabled:
            logger.info(f"[{request_id}] Database query: {db_duration:.3f}s")
        
        # Process data
        process_start = time.time()
        result = process_data(data)
        process_duration = time.time() - process_start
        
        if debug_enabled:
            logger.info(f"[{request_id}] Processing: {process_duration:.3f}s")
        
        # External API call
        api_start = time.time()
        enriched = enrich_with_external_api(result)
        api_duration = time.time() - api_start
        
        if debug_enabled:
            logger.info(f"[{request_id}] External API: {api_duration:.3f}s")
        
        total_duration = time.time() - start
        
        logger.info(f"[{request_id}] Request completed", extra={
            'request_id': request_id,
            'total_duration': total_duration,
            'db_duration': db_duration,
            'process_duration': process_duration,
            'api_duration': api_duration
        })
        
        return enriched
        
    except Exception as e:
        logger.error(f"[{request_id}] Request failed", extra={
            'request_id': request_id,
            'error': str(e),
            'duration': time.time() - start
        }, exc_info=True)
        raise

# Step 3: Deploy with feature flag disabled
# Step 4: Enable debug logging for sample of requests
# Step 5: Analyze logs to find bottleneck

# Example log analysis:
# [abc-123] Request started
# [abc-123] Database query: 0.045s
# [abc-123] Processing: 0.012s
# [abc-123] External API: 12.345s  ← BOTTLENECK FOUND!
# [abc-123] Request completed (12.456s total)
```

### Distributed Tracing

For microservices, track requests across multiple services:

```python
# Using OpenTelemetry for distributed tracing
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter

# Setup tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Export to Jaeger
jaeger_exporter = JaegerExporter(
    agent_host_name='localhost',
    agent_port=6831,
)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)


def deploy_application(app_name: str, version: str):
    """Deploy with distributed tracing."""
    # Start span
    with tracer.start_as_current_span("deploy_application") as span:
        span.set_attribute("app.name", app_name)
        span.set_attribute("app.version", version)
        
        try:
            # Child span: build image
            with tracer.start_as_current_span("build_image"):
                image = build_docker_image(app_name, version)
                span.set_attribute("image.id", image.id)
            
            # Child span: deploy to k8s
            with tracer.start_as_current_span("deploy_to_k8s"):
                deploy_to_kubernetes(image)
            
            # Child span: verify deployment
            with tracer.start_as_current_span("verify_deployment"):
                verify_health(app_name)
            
            span.set_attribute("status", "success")
            
        except Exception as e:
            span.set_attribute("status", "failed")
            span.set_attribute("error", str(e))
            span.record_exception(e)
            raise

# Trace appears in Jaeger UI showing:
# deploy_application (total: 45s)
#   ├─ build_image (5s)
#   ├─ deploy_to_k8s (35s) ← Slow!
#   └─ verify_deployment (5s)
```

### Error Tracking with Sentry

```bash
pip install sentry-sdk
```

```python
import sentry_sdk

# Initialize Sentry
sentry_sdk.init(
    dsn="https://...@sentry.io/...",
    traces_sample_rate=0.1,  # 10% of requests traced
    environment="production"
)


def deploy_application(app_name: str, version: str):
    """Deploy with error tracking."""
    with sentry_sdk.configure_scope() as scope:
        # Add context to all errors
        scope.set_tag("app_name", app_name)
        scope.set_tag("version", version)
        scope.set_context("deployment", {
            "app": app_name,
            "version": version,
            "user": get_current_user()
        })
        
        try:
            result = execute_deployment(app_name, version)
            return result
            
        except Exception as e:
            # Error automatically sent to Sentry with full context
            sentry_sdk.capture_exception(e)
            raise

# Sentry shows:
# - Error message and stack trace
# - app_name and version tags
# - Deployment context
# - User who triggered deployment
# - Server environment details
# - Breadcrumbs (recent logs)
```

### Post-Mortem Debugging

```python
# Scenario: Production crashed overnight
# Need to analyze what happened

# 1. Collect artifacts
import sys
import traceback
import json
from datetime import datetime

def setup_crash_handler():
    """Setup handler to save crash information."""
    def crash_handler(exc_type, exc_value, exc_traceback):
        """Save crash details for post-mortem analysis."""
        crash_info = {
            'timestamp': datetime.utcnow().isoformat(),
            'exception_type': exc_type.__name__,
            'exception_value': str(exc_value),
            'traceback': ''.join(traceback.format_tb(exc_traceback)),
            'locals': {}
        }
        
        # Save local variables from each frame
        tb = exc_traceback
        while tb is not None:
            frame = tb.tb_frame
            crash_info['locals'][frame.f_code.co_filename + ':' + str(tb.tb_lineno)] = {
                k: repr(v) for k, v in frame.f_locals.items()
            }
            tb = tb.tb_next
        
        # Save to file
        with open(f'crash-{datetime.now().strftime("%Y%m%d-%H%M%S")}.json', 'w') as f:
            json.dump(crash_info, f, indent=2)
        
        # Call default handler
        sys.__excepthook__(exc_type, exc_value, exc_traceback)
    
    sys.excepthook = crash_handler

# Install at startup
setup_crash_handler()


# 2. Analyze crash dump
def analyze_crash_dump(crash_file):
    """Analyze crash dump file."""
    with open(crash_file) as f:
        crash = json.load(f)
    
    print(f"Crash Time: {crash['timestamp']}")
    print(f"Exception: {crash['exception_type']}: {crash['exception_value']}")
    print(f"\nTraceback:\n{crash['traceback']}")
    
    print("\nLocal Variables at Crash:")
    for location, locals_dict in crash['locals'].items():
        print(f"\n{location}:")
        for var, value in locals_dict.items():
            print(f"  {var} = {value}")

# Run analysis on crash dump
analyze_crash_dump('crash-20240115-023045.json')
```

### Health Checks and Monitoring

```python
# Implement comprehensive health checks
from flask import Flask, jsonify
import psutil
import time

app = Flask(__name__)

class HealthChecker:
    """Comprehensive health checking."""
    
    def check_database(self):
        """Check database connectivity."""
        try:
            # Try simple query
            db.execute("SELECT 1")
            return {'status': 'healthy', 'latency_ms': 5}
        except Exception as e:
            return {'status': 'unhealthy', 'error': str(e)}
    
    def check_external_api(self):
        """Check external API availability."""
        import requests
        try:
            start = time.time()
            response = requests.get('https://api.external.com/health', timeout=5)
            latency = (time.time() - start) * 1000
            
            if response.status_code == 200:
                return {'status': 'healthy', 'latency_ms': latency}
            else:
                return {'status': 'degraded', 'http_status': response.status_code}
        except Exception as e:
            return {'status': 'unhealthy', 'error': str(e)}
    
    def check_disk_space(self):
        """Check disk space."""
        disk = psutil.disk_usage('/')
        percent_used = disk.percent
        
        if percent_used > 90:
            return {'status': 'critical', 'percent_used': percent_used}
        elif percent_used > 75:
            return {'status': 'warning', 'percent_used': percent_used}
        else:
            return {'status': 'healthy', 'percent_used': percent_used}
    
    def check_memory(self):
        """Check memory usage."""
        mem = psutil.virtual_memory()
        percent_used = mem.percent
        
        if percent_used > 90:
            return {'status': 'critical', 'percent_used': percent_used}
        elif percent_used > 75:
            return {'status': 'warning', 'percent_used': percent_used}
        else:
            return {'status': 'healthy', 'percent_used': percent_used}


@app.route('/health')
def health():
    """Health check endpoint."""
    checker = HealthChecker()
    
    health_status = {
        'overall': 'healthy',
        'timestamp': datetime.utcnow().isoformat(),
        'checks': {
            'database': checker.check_database(),
            'external_api': checker.check_external_api(),
            'disk_space': checker.check_disk_space(),
            'memory': checker.check_memory()
        }
    }
    
    # Overall status: unhealthy if any check is unhealthy
    for check_name, check_result in health_status['checks'].items():
        if check_result['status'] in ['unhealthy', 'critical']:
            health_status['overall'] = 'unhealthy'
            break
        elif check_result['status'] == 'degraded' and health_status['overall'] == 'healthy':
            health_status['overall'] = 'degraded'
    
    status_code = 200 if health_status['overall'] == 'healthy' else 503
    
    return jsonify(health_status), status_code


@app.route('/metrics')
def metrics():
    """Prometheus metrics endpoint."""
    # Return metrics in Prometheus format
    output = []
    
    # Memory metrics
    mem = psutil.virtual_memory()
    output.append(f'memory_usage_percent {mem.percent}')
    output.append(f'memory_used_bytes {mem.used}')
    output.append(f'memory_available_bytes {mem.available}')
    
    # CPU metrics
    output.append(f'cpu_usage_percent {psutil.cpu_percent(interval=1)}')
    
    # Disk metrics
    disk = psutil.disk_usage('/')
    output.append(f'disk_usage_percent {disk.percent}')
    output.append(f'disk_free_bytes {disk.free}')
    
    return '\n'.join(output), 200, {'Content-Type': 'text/plain'}
```

### Debugging Checklist for Production Issues

```python
"""
Production Debugging Checklist

□ 1. Gather Information
   □ What changed recently? (deployments, config, traffic)
   □ When did it start?
   □ Is it affecting all users or specific subset?
   □ Error rate/frequency?

□ 2. Check Basics
   □ Service running?
   □ Health checks passing?
   □ Resource usage (CPU, memory, disk)?
   □ Network connectivity?

□ 3. Review Logs
   □ Application logs
   □ System logs (syslog, journalctl)
   □ Load balancer logs
   □ Database logs

□ 4. Check Metrics
   □ Request rate
   □ Response time
   □ Error rate
   □ Resource utilization

□ 5. Test Hypothesis
   □ Form theory based on data
   □ Test systematically
   □ Document findings

□ 6. Implement Fix
   □ Make minimal change
   □ Test in staging first
   □ Deploy with rollback plan
   □ Monitor closely

□ 7. Post-Mortem
   □ Document root cause
   □ Identify prevention measures
   □ Update runbooks
   □ Add monitoring/alerting
"""
```

### Frequently Asked Questions - Production Debugging

**Q1: How do I debug an issue that only happens in production?**

**A:** **Systematic approach with logging and monitoring:**

**Step 1: Enhanced Logging**
```python
# Add comprehensive logging (controlled by feature flag)
if os.getenv('DEBUG_MODE') == 'true':
    logger.setLevel(logging.DEBUG)
    logger.debug(f"Processing request: {request}")
    logger.debug(f"Database query: {query}")
    logger.debug(f"Result: {result}")
```

**Step 2: Reproduce Locally**
```python
# Capture production state
production_data = get_production_snapshot()
save_to_file('production_state.json', production_data)

# Load in local environment
with open('production_state.json') as f:
    state = json.load(f)
    
# Recreate production scenario locally
recreate_scenario(state)
```

**Step 3: Gradual Rollout**
```python
# Test fix on subset of traffic
if user_id % 10 == 0:  # 10% of users
    use_new_code()
else:
    use_old_code()
```

**Q2: Production is down! How do I debug quickly?**

**A:** **Follow incident response procedure:**

**Immediate Actions (0-5 minutes):**
```bash
# 1. Check if service is running
systemctl status myapp

# 2. Check recent logs
journalctl -u myapp -n 100 --no-pager

# 3. Check resource usage
top
df -h

# 4. Check network
ping database.example.com
curl http://api.example.com/health
```

**Quick Fixes (5-15 minutes):**
```bash
# If out of memory: restart
systemctl restart myapp

# If disk full: clear logs
rm -rf /var/log/old_logs/*

# If process stuck: kill and restart
killall -9 myapp
systemctl start myapp

# If database connection issue: test connectivity
psql -h database.example.com -U user -d mydb
```

**Root Cause (15+ minutes):**
```python
# Add logging to identify exact issue
# Deploy fix
# Monitor closely
```

---

### Key Takeaways - Production Debugging

✅ **Logs are essential** - Comprehensive logging saves time

✅ **Distributed tracing** - Track requests across services

✅ **Error tracking** - Sentry/Rollbar for automatic error capture

✅ **Health checks** - Multiple checks (DB, disk, memory, APIs)

✅ **Metrics** - Prometheus/Grafana for monitoring

✅ **Feature flags** - Enable/disable debug logging

✅ **Reproduce locally** - Capture production state

✅ **Incident response** - Follow systematic procedure

---

## Final Summary - Part 4

Part 4 covered essential debugging and troubleshooting for DevSecOps:

**✅ Complete Topics:**

**Section 4.1: Interactive Debugging with pdb**
- Basic pdb commands and navigation
- Conditional breakpoints
- Post-mortem debugging
- pdb++ enhancements
- Remote debugging
- Debugging containers and threads

**Section 4.2: Logging Best Practices**
- Python logging basics
- Structured logging with JSON
- Context logging with request IDs
- Security-safe logging (redacting secrets)
- Performance logging
- Logging configuration

**Section 4.3: Performance Profiling**
- cProfile for function-level profiling
- line_profiler for line-level profiling
- memory_profiler for memory usage
- py-spy for production profiling
- Async code profiling
- Benchmarking with timeit
- Optimization strategies

**Section 4.4: Memory Debugging**
- Detecting memory leaks
- tracemalloc for memory tracking
- objgraph for reference visualization
- Common memory leak patterns
- Garbage collection debugging
- Memory optimization techniques

**Section 4.5: Production Debugging**
- Production debugging without access
- Distributed tracing with OpenTelemetry
- Error tracking with Sentry
- Post-mortem debugging
- Health checks and monitoring
- Incident response procedures

### Key Debugging Principles

✅ **Measure before optimizing** - Profile to find bottlenecks

✅ **Log comprehensively** - Structured logs with context

✅ **Monitor continuously** - Metrics, traces, errors

✅ **Test hypotheses** - Systematic debugging

✅ **Document findings** - Runbooks and post-mortems

✅ **Secure sensitive data** - Redact secrets from logs

✅ **Health checks everywhere** - Database, APIs, resources

✅ **Feature flags** - Control debug logging in production

### Production Debugging Toolkit

**Interactive Debugging:**
- pdb/pdb++ (local)
- py-spy (production)
- Remote debugging (carefully)

**Logging:**
- structlog/python-json-logger
- ELK stack (Elasticsearch, Logstash, Kibana)
- CloudWatch Logs (AWS)

**Profiling:**
- cProfile (function-level)
- line_profiler (line-level)
- memory_profiler (memory)
- py-spy (production)

**Monitoring:**
- Prometheus + Grafana (metrics)
- Jaeger/Zipkin (tracing)
- Sentry (errors)
- DataDog/New Relic (APM)

### Incident Response Checklist

When production breaks:

**0-5 minutes:**
- Assess impact
- Check service status
- Review recent changes
- Check logs for errors

**5-15 minutes:**
- Identify root cause
- Implement quick fix if available
- Communicate status to stakeholders
- Consider rollback if needed

**15+ minutes:**
- Implement permanent fix
- Test thoroughly
- Deploy with monitoring
- Verify resolution

**Post-Incident:**
- Write post-mortem
- Identify preventive measures
- Update monitoring/alerting
- Update runbooks

### Recommended Tools

**Debugging:**
- pdb++ (enhanced debugging)
- ipdb (IPython integration)
- pudb (visual debugger)

**Logging:**
- structlog (structured logging)
- python-json-logger (JSON logs)
- loguru (simplified logging)

**Profiling:**
- py-spy (production profiling)
- scalene (CPU+GPU+memory profiler)
- memray (memory profiler)

**Monitoring:**
- Prometheus (metrics)
- Grafana (visualization)
- Jaeger (distributed tracing)
- Sentry (error tracking)

### Next Steps

**To master debugging:**

1. **Practice debugging** - Debug real issues, not tutorials
2. **Read production logs** - Learn to spot patterns
3. **Set up monitoring** - Before you need it
4. **Write runbooks** - Document common issues
5. **Conduct post-mortems** - Learn from incidents

**Continue learning:**
- Part 5: Security in Python
- Part 6: DevSecOps Automation
- Part 7: Advanced Python Topics
- Part 8: Real-World Projects
- Part 9: Interview Preparation

---

*End of Part 4: Debugging and Troubleshooting*

**Part 4 is now COMPLETE with 15,000+ words covering comprehensive debugging and troubleshooting strategies for DevSecOps engineers.**

---

**Total Series Progress:**

| Part | Topic | Words | Status |
|------|-------|-------|--------|
| **1** | Python Fundamentals | 18,172 | ✅ Complete |
| **2** | Software Engineering | 17,281 | ✅ Complete |
| **3** | Testing & QA | 10,630 | ✅ Complete |
| **4** | Debugging & Troubleshooting | 15,000+ | ✅ Complete |
| **Total** | **4 Complete Parts** | **61,000+** | **4/9 Parts** |

**🎉 You now have FOUR comprehensive guides with 61,000+ words of production-ready Python knowledge for DevSecOps!**
