---
title: "Python DevSecOps - Part 1: Python Fundamentals for Production Code"
date: 2025-01-02 00:00:00 +0530
categories: [Python, DevSecOps]
tags: [Python, DevSecOps, Software Engineering, Security, Testing, Debugging, Automation, Infrastructure, Monitoring, CI-CD]
---

# Complete Python Mastery for DevSecOps Part 1: Python Fundamentals for Production Code

## Introduction

Welcome to the most comprehensive Python guide designed specifically for DevSecOps engineers. This series will transform you from a Python beginner to a production-ready DevSecOps engineer who writes secure, maintainable, and highly reliable code.

**What Makes This Guide Different:**

This isn't just another Python tutorial. This is a battle-tested guide written from the trenches of production systems, security incidents, and 3 AM debugging sessions. Every concept is tied directly to DevSecOps practices, security considerations, and operational excellence.

**Who This Series Is For:**

- DevSecOps engineers building automation and infrastructure
- Developers transitioning to DevOps/SRE roles
- Security engineers learning to code and automate
- Platform engineers building internal tooling
- Anyone who needs to write production-grade Python

**What You'll Learn:**

By the end of this series, you'll be able to:

- Write secure, maintainable Python code that passes production code reviews
- Build comprehensive test suites that catch bugs before production
- Debug complex issues in distributed systems
- Automate infrastructure and security operations
- Design scalable, resilient systems
- Pass technical interviews for senior DevSecOps roles

**Prerequisites:**

- Basic command-line knowledge
- Familiarity with Linux/Unix
- Understanding of basic programming concepts
- Git basics
- No prior Python experience required

---

## 1.1 Introduction to Python for DevSecOps

### Why Python for DevSecOps?

Python has become the de facto language for DevSecOps for compelling reasons that go beyond "it's easy to learn."

**1. Ecosystem Dominance**

Python's ecosystem provides production-ready libraries for virtually every DevSecOps task:

- **Infrastructure**: boto3 (AWS), azure-sdk, google-cloud
- **Configuration Management**: Ansible, SaltStack
- **Container Orchestration**: kubernetes-client
- **Monitoring**: prometheus-client, datadog
- **Security**: bandit, safety, cryptography
- **Testing**: pytest, hypothesis, locust

**2. Readability Equals Maintainability**

In DevSecOps, your code is read 10x more than it's written. Python's emphasis on readability means:

- Faster incident response when reading automation scripts at 3 AM
- Easier knowledge transfer when team members change
- Reduced cognitive load during code reviews
- Less time debugging cryptic syntax

**3. Rapid Prototyping to Production**

Python excels at quick scripts that evolve into production tools:

```python
# ❌ BAD: Shell script that grew unwieldy
# deploy.sh with 500 lines of bash

# ✅ GOOD: Python script with proper error handling
import boto3
import logging
from typing import List, Dict

def deploy_application(
    app_name: str,
    version: str,
    environment: str
) -> Dict[str, str]:
    """
    Deploy application to specified environment.
  
    Args:
        app_name: Application identifier
        version: Semantic version to deploy
        environment: Target environment (dev/staging/prod)
  
    Returns:
        Deployment details including URL and status
      
    Raises:
        DeploymentError: If deployment fails
    """
    logger.info(f"Deploying {app_name} v{version} to {environment}")
    # Implementation with proper error handling
    pass
```

**4. Strong Standard Library**

Python's "batteries included" philosophy means less dependency management:

```python
import json          # JSON parsing
import logging       # Production logging
import subprocess    # Safe process execution
import pathlib       # Modern path handling
import secrets       # Cryptographically secure random
import hashlib       # Cryptographic hashing
import urllib.parse  # URL handling
import argparse      # CLI parsing
```

### Python Philosophy and Zen of Python

Understanding Python's philosophy makes you a better Python developer:

```python
import this
```

**Output:**

```
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

**DevSecOps Translation:**

1. **"Explicit is better than implicit"** - Don't hide security checks in magic methods
2. **"Errors should never pass silently"** - Log everything, fail loudly in development
3. **"Readability counts"** - Your deployment script should be readable at 3 AM
4. **"There should be one obvious way"** - Consistent patterns across your codebase

### Python 2 vs Python 3: The Definitive Answer

**NEVER USE PYTHON 2. EVER.**

Python 2 reached End of Life on January 1, 2020. Using Python 2 in production is a security vulnerability:

- No security patches
- No bug fixes
- Incompatible with modern libraries
- Unicode handling is broken
- String formatting is inconsistent

**Security Impact:**

```python
# Python 2 - SECURITY VULNERABILITY
raw_input()  # No input validation, easily exploited

# Python 3 - Modern and secure
input()  # Returns string, type-safe
```

**If you encounter Python 2 code:**

1. Flag it as technical debt
2. Schedule migration immediately
3. Document the security risk
4. Never extend Python 2 code - rewrite in Python 3

### CPython and Alternative Implementations

**CPython** is the reference implementation written in C. It's what you get when you run `python3`.

**Alternative Implementations:**

| Implementation           | Use Case                  | Pros                             | Cons                      |
| ------------------------ | ------------------------- | -------------------------------- | ------------------------- |
| **PyPy**           | Performance-critical apps | 4-7x faster on long-running code | Not all C extensions work |
| **Jython**         | Java integration          | Access Java libraries            | Outdated, Python 2 only   |
| **IronPython**     | .NET integration          | Access .NET libraries            | Outdated, limited         |
| **MicroPython**    | IoT devices               | Embedded systems                 | Limited stdlib            |
| **GraalVM Python** | Polyglot apps             | Multiple languages               | Experimental              |

**DevSecOps Recommendation:**

Stick with **CPython 3.10+** unless you have a specific reason:

- It's the most compatible
- Best library support
- Easiest to find documentation
- What your team knows

### Python's Role in Modern Infrastructure

**1. Infrastructure Automation**

```python
# AWS infrastructure provisioning
import boto3

def create_secure_s3_bucket(bucket_name: str, region: str) -> dict:
    """Create S3 bucket with security best practices."""
    s3 = boto3.client('s3', region_name=region)
  
    # Create bucket
    s3.create_bucket(
        Bucket=bucket_name,
        CreateBucketConfiguration={'LocationConstraint': region}
    )
  
    # Enable versioning
    s3.put_bucket_versioning(
        Bucket=bucket_name,
        VersioningConfiguration={'Status': 'Enabled'}
    )
  
    # Enable encryption
    s3.put_bucket_encryption(
        Bucket=bucket_name,
        ServerSideEncryptionConfiguration={
            'Rules': [{
                'ApplyServerSideEncryptionByDefault': {
                    'SSEAlgorithm': 'AES256'
                }
            }]
        }
    )
  
    # Block public access
    s3.put_public_access_block(
        Bucket=bucket_name,
        PublicAccessBlockConfiguration={
            'BlockPublicAcls': True,
            'IgnorePublicAcls': True,
            'BlockPublicPolicy': True,
            'RestrictPublicBuckets': True
        }
    )
  
    return {'bucket': bucket_name, 'region': region, 'secure': True}
```

**2. Security Scanning**

```python
# Vulnerability scanning automation
import subprocess
import json
from typing import List, Dict

def scan_dependencies(requirements_file: str) -> List[Dict]:
    """Scan dependencies for vulnerabilities."""
    result = subprocess.run(
        ['pip-audit', '--format', 'json', '-r', requirements_file],
        capture_output=True,
        text=True
    )
  
    if result.returncode != 0:
        vulnerabilities = json.loads(result.stdout)
        return vulnerabilities
  
    return []
```

**3. Deployment Automation**

```python
# Blue-green deployment
def blue_green_deploy(
    service_name: str,
    new_version: str,
    health_check_url: str
) -> bool:
    """Execute blue-green deployment with automatic rollback."""
    try:
        # Deploy to green environment
        deploy_to_environment('green', service_name, new_version)
      
        # Health check
        if not wait_for_healthy('green', health_check_url):
            raise DeploymentError("Health check failed")
      
        # Switch traffic
        switch_traffic_to('green')
      
        # Monitor for errors
        if error_rate_acceptable():
            decommission_environment('blue')
            return True
        else:
            # Automatic rollback
            switch_traffic_to('blue')
            raise DeploymentError("Error rate too high, rolled back")
          
    except Exception as e:
        logger.error(f"Deployment failed: {e}")
        # Ensure we're on stable version
        switch_traffic_to('blue')
        return False
```

### Setting Up Production-Grade Python Environments

#### System-Level Python Installation

**Never modify system Python.** Use version management tools.

**Install pyenv (recommended):**

```bash
# Install pyenv
curl https://pyenv.run | bash

# Add to ~/.bashrc or ~/.zshrc
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Install Python versions
pyenv install 3.11.7
pyenv install 3.10.13

# Set global default
pyenv global 3.11.7

# Verify
python --version  # Python 3.11.7
```

**Why pyenv?**

- Install multiple Python versions
- No sudo required
- Project-specific Python versions
- No conflict with system Python

#### Virtual Environments: Essential for Isolation

**Virtual environments isolate project dependencies.** This prevents:

- Dependency conflicts between projects
- Accidental system Python modification
- "Works on my machine" problems

**Three Approaches:**

**1. venv (stdlib, always available):**

```bash
# Create virtual environment
python -m venv .venv

# Activate (Linux/Mac)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Deactivate
deactivate
```

**2. virtualenv (more features):**

```bash
# Install
pip install virtualenv

# Create with specific Python version
virtualenv -p python3.11 .venv

# Same activation as venv
source .venv/bin/activate
```

**3. poetry (modern, recommended for projects):**

```bash
# Install poetry
curl -sSL https://install.python-poetry.org | python3 -

# Initialize project
poetry init

# Add dependencies
poetry add requests
poetry add --group dev pytest black mypy

# Install dependencies
poetry install

# Run in virtual environment
poetry run python script.py

# Activate shell
poetry shell
```

**Poetry advantages:**

- Automatic dependency resolution
- Lock file for reproducible builds
- Dependency groups (dev, test, docs)
- Publishing to PyPI built-in
- Modern pyproject.toml configuration

#### Dependency Management Best Practices

**1. Pin All Dependencies**

```python
# ❌ BAD: Unpinned dependencies
# requirements.txt
requests
flask
boto3

# ✅ GOOD: Pinned dependencies
# requirements.txt
requests==2.31.0
flask==3.0.0
boto3==1.34.10
```

**2. Use Separate Requirement Files**

```
requirements/
├── base.txt          # Core dependencies
├── dev.txt           # Development tools
├── test.txt          # Testing dependencies
└── prod.txt          # Production only
```

```python
# requirements/base.txt
requests==2.31.0
pydantic==2.5.3

# requirements/dev.txt
-r base.txt
black==23.12.1
mypy==1.8.0
ruff==0.1.9

# requirements/test.txt
-r base.txt
pytest==7.4.3
pytest-cov==4.1.0
pytest-mock==3.12.0

# requirements/prod.txt
-r base.txt
gunicorn==21.2.0
```

**3. Lock Dependencies with pip-tools**

```bash
# Install pip-tools
pip install pip-tools

# Create requirements.in
echo "requests>=2.31.0" > requirements.in
echo "flask>=3.0.0" >> requirements.in

# Compile to requirements.txt with hashes
pip-compile --generate-hashes requirements.in

# Produces requirements.txt with all transitive dependencies pinned
```

**4. Scan for Vulnerabilities**

```bash
# Install security scanner
pip install pip-audit

# Scan dependencies
pip-audit

# In CI/CD pipeline
pip-audit --format json --output vulnerabilities.json
```

#### Python Version Management in CI/CD

**GitHub Actions Example:**

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
  
    steps:
    - uses: actions/checkout@v4
  
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v5
      with:
        python-version: ${{ matrix.python-version }}
  
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements/test.txt
  
    - name: Run security checks
      run: |
        pip-audit
        bandit -r src/
  
    - name: Run tests
      run: |
        pytest --cov=src --cov-report=xml
  
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

### Security Considerations

**1. Never Install Packages as Root**

```bash
# ❌ BAD: Installs system-wide, requires sudo
sudo pip install requests

# ✅ GOOD: User-level or virtual environment
pip install --user requests  # User-level
# OR
python -m venv .venv && source .venv/bin/activate && pip install requests
```

**2. Verify Package Integrity**

```bash
# Use hashes to verify packages
pip install --require-hashes -r requirements.txt
```

**3. Use Private PyPI for Internal Packages**

```python
# ~/.pip/pip.conf or pip.conf
[global]
index-url = https://pypi.org/simple
extra-index-url = https://pypi.internal.company.com/simple
trusted-host = pypi.internal.company.com
```

**4. Scan Dependencies Before Installation**

```bash
# Check package safety before installing
pip install safety
safety check
```

### Common Pitfalls

**1. Mixing System and Virtual Environment Packages**

```bash
# ❌ BAD: Installing without virtual environment active
pip install requests  # Where does this go?

# ✅ GOOD: Always verify virtual environment
which python  # Should show .venv path
pip install requests
```

**2. Committing Virtual Environment to Git**

```bash
# .gitignore (always include)
.venv/
venv/
env/
*.pyc
__pycache__/
.pytest_cache/
.mypy_cache/
.coverage
htmlcov/
dist/
build/
*.egg-info/
```

**3. Not Updating Dependencies**

```bash
# Check for outdated packages
pip list --outdated

# Update with pip-tools
pip-compile --upgrade requirements.in
```

### Frequently Asked Questions

**Q1: Should I use conda or pip for dependency management?**

**A:** For DevSecOps, prefer **pip + virtual environments** or **poetry**:

- Conda is excellent for data science with binary dependencies
- pip is standard, lighter weight, better CI/CD integration
- poetry provides modern dependency management

Use conda if:

- Heavy numerical computing (NumPy, SciPy, pandas)
- Need non-Python dependencies (R, system libraries)

Use pip/poetry for:

- Web services and APIs
- Infrastructure automation
- DevOps tooling
- Better reproducibility in containers

**Q2: How do I handle Python versions across different environments?**

**A:** Use Docker for complete environment isolation:

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Non-root user for security
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

CMD ["python", "app.py"]
```

**Q3: How many virtual environments should I create?**

**A:** One per project:

- Keeps dependencies isolated
- Prevents version conflicts
- Makes requirements.txt accurate
- Simplifies troubleshooting

**Q4: Should I commit requirements.txt or poetry.lock?**

**A:** **YES, always commit:**

- requirements.txt or pyproject.toml (source of truth)
- requirements.lock or poetry.lock (exact versions)
- These enable reproducible builds

**Q5: How do I audit Python dependencies for security vulnerabilities?**

**A:** Use multiple tools in your CI/CD pipeline:

```bash
# Install security tools
pip install pip-audit safety bandit

# Audit dependencies
pip-audit --format json

# Check against safety database
safety check

# Scan code for security issues
bandit -r src/
```

### Interview Questions

**Question 1: Explain the difference between pip and pip3. When would you use each?**

**Difficulty:** Junior

**Answer:**

`pip` and `pip3` are both package managers for Python, but they target different Python versions:

- **pip**: Could refer to Python 2's package manager (on systems with both Python 2 and 3)
- **pip3**: Explicitly refers to Python 3's package manager

**Best Practice:** Always use `python -m pip` instead:

```bash
# ❌ Ambiguous - which Python version?
pip install requests

# ✅ Explicit - uses current Python interpreter
python -m pip install requests
python3 -m pip install requests

# ✅ Even better - use virtual environment
python -m venv .venv
source .venv/bin/activate
pip install requests  # Now unambiguous
```

**Why `python -m pip`?**

- Guarantees you're using the pip associated with that Python version
- No ambiguity about which Python environment
- Works consistently across platforms

**In DevSecOps context:**

- Scripts should use `python -m pip` for clarity
- Document which Python version you require
- Never assume `pip` points to Python 3

**Follow-up Questions:**

- How do you ensure reproducible pip installations?
- What security considerations apply to pip?
- How do you handle pip in air-gapped environments?

---

**Question 2: You need to deploy a Python application across development, staging, and production environments. How do you manage dependencies to ensure consistency?**

**Difficulty:** Mid-Level

**Answer:**

**Multi-layered approach:**

**1. Pin exact versions with lock files:**

```bash
# Using pip-tools
pip-compile --generate-hashes requirements.in -o requirements.txt

# Using poetry
poetry lock
```

**2. Separate requirements by environment:**

```
requirements/
├── base.txt          # Core application dependencies
├── dev.txt           # Development tools (debuggers, linters)
├── test.txt          # Testing frameworks
└── prod.txt          # Production-only (monitoring, logging)
```

**3. Use Docker for environment parity:**

```dockerfile
FROM python:3.11-slim as base

# Install base dependencies
COPY requirements/base.txt .
RUN pip install --no-cache-dir -r base.txt

FROM base as development
COPY requirements/dev.txt .
RUN pip install --no-cache-dir -r dev.txt

FROM base as production
COPY requirements/prod.txt .
RUN pip install --no-cache-dir -r prod.txt
```

**4. Verify dependencies in CI/CD:**

```yaml
# .github/workflows/ci.yml
- name: Check dependency hashes
  run: pip install --require-hashes -r requirements.txt

- name: Audit dependencies
  run: |
    pip install pip-audit
    pip-audit --require-hashes -r requirements.txt
```

**5. Use identical Python versions:**

```python
# setup.py or pyproject.toml
python_requires='>=3.11,<3.12'
```

**Why This Matters in DevSecOps:**

- Prevents "works on my machine" issues
- Security: Same audited dependencies everywhere
- Compliance: Traceable dependency versions
- Incident response: Faster debugging with consistent environments

**Follow-up Questions:**

- How do you handle dependencies with C extensions across platforms?
- What's your strategy for updating dependencies?
- How do you manage secrets in different environments?

---

**Question 3: Explain how Python's Global Interpreter Lock (GIL) affects your choice of concurrency strategy in a production DevSecOps tool.**

**Difficulty:** Senior

**Answer:**

The **Global Interpreter Lock (GIL)** is a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecode simultaneously.

**GIL Implications:**

**1. CPU-Bound Tasks:**

```python
# ❌ BAD: Threading doesn't help CPU-bound work
import threading
import time

def cpu_intensive_task():
    # Pure computation - GIL prevents true parallelism
    result = sum(i * i for i in range(10_000_000))
    return result

threads = [threading.Thread(target=cpu_intensive_task) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
# No speedup - threads take turns holding GIL
```

```python
# ✅ GOOD: Multiprocessing bypasses GIL
import multiprocessing as mp

def cpu_intensive_task():
    return sum(i * i for i in range(10_000_000))

with mp.Pool(4) as pool:
    results = pool.map(cpu_intensive_task, range(4))
# True parallelism - separate processes, separate GILs
```

**2. I/O-Bound Tasks:**

```python
# ✅ GOOD: Threading works for I/O-bound operations
import threading
import requests

def check_endpoint(url):
    # I/O operation - releases GIL while waiting
    response = requests.get(url)
    return response.status_code

urls = ['http://api1.com', 'http://api2.com', 'http://api3.com']
threads = [threading.Thread(target=check_endpoint, args=(url,)) for url in urls]
for t in threads: t.start()
for t in threads: t.join()
# Threads speed this up - GIL released during I/O
```

**DevSecOps Decision Matrix:**

| Task Type              | Best Approach                            | Why                            |
| ---------------------- | ---------------------------------------- | ------------------------------ |
| API calls, network I/O | **asyncio** or **threading** | GIL released during I/O        |
| Database queries       | **asyncio** with async drivers     | Best performance for I/O       |
| File processing        | **multiprocessing**                | CPU-intensive parsing          |
| Log analysis           | **multiprocessing**                | Text processing is CPU-bound   |
| Security scanning      | **multiprocessing**                | CPU-intensive analysis         |
| Monitoring checks      | **asyncio**                        | Many concurrent I/O operations |

**Real-World Example:**

```python
# Scanning multiple servers for vulnerabilities
import asyncio
import aiohttp
from concurrent.futures import ProcessPoolExecutor

async def scan_server_io(server_url: str) -> dict:
    """I/O-bound: Fetch server data - use async"""
    async with aiohttp.ClientSession() as session:
        async with session.get(f"{server_url}/health") as resp:
            return await resp.json()

def analyze_vulnerabilities_cpu(data: dict) -> list:
    """CPU-bound: Parse and analyze - use multiprocessing"""
    # Complex analysis of response data
    vulnerabilities = []
    # ... intensive computation ...
    return vulnerabilities

async def full_scan(servers: list[str]):
    """Hybrid approach for optimal performance"""
    # Phase 1: Async I/O to fetch all data
    fetch_tasks = [scan_server_io(url) for url in servers]
    all_data = await asyncio.gather(*fetch_tasks)
  
    # Phase 2: Multiprocessing for CPU-intensive analysis
    with ProcessPoolExecutor() as executor:
        loop = asyncio.get_event_loop()
        analysis_tasks = [
            loop.run_in_executor(executor, analyze_vulnerabilities_cpu, data)
            for data in all_data
        ]
        results = await asyncio.gather(*analysis_tasks)
  
    return results
```

**Why This Matters in DevSecOps:**

- **Wrong choice costs money**: Thread pool for CPU work wastes resources
- **Wrong choice misses SLOs**: Sequential processing when concurrent would work
- **Right choice enables scale**: Proper concurrency handles thousands of servers

**Follow-up Questions:**

- How would you profile to determine if a task is CPU or I/O bound?
- What are the trade-offs of using PyPy to avoid GIL?
- How do you handle shared state in multiprocessing?

---

### Key Takeaways

✅ **Always use Python 3.10+** - Python 2 is a security vulnerability

✅ **Use virtual environments religiously** - One per project, no exceptions

✅ **Pin dependencies with lock files** - Reproducible builds are mandatory

✅ **Scan dependencies for vulnerabilities** - pip-audit and safety in CI/CD

✅ **Use pyenv for version management** - Multiple Python versions without conflicts

✅ **Prefer poetry for new projects** - Modern dependency management

✅ **Never modify system Python** - Keep system Python pristine

✅ **Use `python -m pip`** - Explicit is better than implicit

✅ **Separate dev/test/prod requirements** - Clear dependency boundaries

✅ **Audit dependencies before deployment** - Security is not optional

---

## 1.2 Python Syntax and Core Concepts (Continued)

### Variables and Memory Management

Understanding how Python manages memory is crucial for writing efficient, bug-free code.

#### Variable Assignment and References

In Python, variables are **references to objects**, not boxes that hold values.

```python
# Variable assignment creates reference
x = 42  # x is a reference to integer object 42

# Multiple variables can reference same object
y = x  # y references same object as x

# Checking object identity
print(x is y)  # True - same object
print(id(x), id(y))  # Same memory address
```

**Immutable vs Mutable Reference Behavior:**

```python
# Immutable: reassignment creates new object
x = 42
y = x
x = 43  # x now references different object
print(x, y)  # 43, 42 - y unchanged

# Mutable: both variables see changes
list1 = [1, 2, 3]
list2 = list1  # Both reference same list
list1.append(4)
print(list2)  # [1, 2, 3, 4] - list2 sees change!

# ✅ GOOD: Explicit copy for mutable objects
list2 = list1.copy()  # or list1[:]
list1.append(5)
print(list2)  # [1, 2, 3, 4] - list2 unchanged
```

**DevSecOps Pitfall:**

```python
# ❌ DANGEROUS: Shared mutable default argument
def add_server(server: str, server_list=[]):
    """Add server to list."""
    server_list.append(server)
    return server_list

# First call
prod_servers = add_server('web-01')  # ['web-01']

# Second call - SURPRISE!
dev_servers = add_server('dev-01')  # ['web-01', 'dev-01']
# dev_servers contains prod server!

# ✅ GOOD: Use None and create new list
def add_server(server: str, server_list=None):
    """Add server to list."""
    if server_list is None:
        server_list = []
    server_list.append(server)
    return server_list
```

#### Object Identity vs Equality

```python
# == checks value equality
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True - same values

# is checks object identity
print(a is b)  # False - different objects

# Singleton objects (None, True, False) use 'is'
# ❌ BAD: Using == for None
if value == None:
    pass

# ✅ GOOD: Using 'is' for None
if value is None:
    pass

# Small integer caching (CPython optimization)
x = 256
y = 256
print(x is y)  # True - cached

x = 257
y = 257
print(x is y)  # False (in interactive mode)
# Note: behavior may vary based on context
```

#### Memory Management and Garbage Collection

**Reference Counting:**

Python uses reference counting as primary memory management:

```python
import sys

x = []  # Refcount = 1
sys.getrefcount(x)  # 2 (function parameter adds reference)

y = x  # Refcount = 2
z = x  # Refcount = 3

del y  # Refcount = 2
del z  # Refcount = 1
del x  # Refcount = 0, object deallocated
```

**Cyclic References:**

Reference counting alone can't handle cycles:

```python
# Cyclic reference
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create cycle
node1 = Node(1)
node2 = Node(2)
node1.next = node2
node2.next = node1  # Cycle!

# Deleting doesn't free memory immediately
del node1
del node2
# Cycle keeps objects alive until garbage collector runs
```

**Garbage Collection:**

Python's cyclic garbage collector breaks reference cycles:

```python
import gc

# Manual garbage collection
gc.collect()  # Force collection

# Disable automatic GC (for performance-critical sections)
gc.disable()
# ... performance-critical code ...
gc.enable()

# Check for uncollectable objects (usually indicates bugs)
uncollectable = gc.garbage
if uncollectable:
    logger.warning(f"Uncollectable objects: {uncollectable}")
```

**Memory Profiling:**

```python
import tracemalloc

# Start tracking
tracemalloc.start()

# Your code here
data = [i for i in range(1000000)]

# Get memory usage
current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 1024 / 1024:.2f} MB")
print(f"Peak: {peak / 1024 / 1024:.2f} MB")

# Stop tracking
tracemalloc.stop()
```

**DevSecOps Application:**

```python
# Memory-efficient log processing
def process_large_log_file(filename: str):
    """Process log file without loading entire file into memory."""
    # ❌ BAD: Loads entire file into memory
    # with open(filename) as f:
    #     lines = f.readlines()  # Entire file in memory!
    #     for line in lines:
    #         process_line(line)
  
    # ✅ GOOD: Process line by line
    with open(filename) as f:
        for line in f:  # Generator - one line at a time
            if 'ERROR' in line:
                process_error(line.strip())
```

#### Weak References

Use weak references to avoid keeping objects alive:

```python
import weakref

class ExpensiveResource:
    def __init__(self, name):
        self.name = name
        print(f"Created {name}")
  
    def __del__(self):
        print(f"Deleted {name}")

# Strong reference - keeps object alive
resource = ExpensiveResource("db_connection")

# Weak reference - doesn't keep object alive
weak_ref = weakref.ref(resource)

# Access through weak reference
if weak_ref() is not None:
    print(weak_ref().name)

# Delete strong reference
del resource  # "Deleted db_connection" printed
print(weak_ref())  # None - object was deallocated
```

**Cache with Weak References:**

```python
import weakref
from typing import Dict, Any

class ServerCache:
    """Cache that doesn't prevent garbage collection."""
  
    def __init__(self):
        self._cache: weakref.WeakValueDictionary = weakref.WeakValueDictionary()
  
    def add(self, server_id: str, server: Any):
        """Add server to cache."""
        self._cache[server_id] = server
  
    def get(self, server_id: str) -> Any:
        """Get server from cache."""
        return self._cache.get(server_id)

# Usage
cache = ServerCache()
server = create_server("web-01")
cache.add("web-01", server)

# Server accessible while referenced
print(cache.get("web-01"))  # Returns server

# When no strong references remain, server is garbage collected
del server
print(cache.get("web-01"))  # None - server was collected
```

### Operators and Expressions

#### Arithmetic Operators

```python
# Basic arithmetic
x = 10 + 5   # 15
x = 10 - 5   # 5
x = 10 * 5   # 50
x = 10 / 5   # 2.0 (always returns float)
x = 10 // 3  # 3 (floor division)
x = 10 % 3   # 1 (modulo)
x = 2 ** 10  # 1024 (power)

# Augmented assignment
x = 10
x += 5   # x = x + 5
x -= 3   # x = x - 3
x *= 2   # x = x * 2
x //= 4  # x = x // 4
```

**Division Gotchas:**

```python
# Python 3: / always returns float
10 / 5  # 2.0, not 2

# Floor division rounds toward negative infinity
-10 // 3  # -4 (not -3!)

# Modulo preserves sign of divisor
-10 % 3  # 2 (not -1!)
10 % -3  # -2 (not 1!)

# For truncation toward zero, use int division
int(-10 / 3)  # -3
```

#### Comparison Operators

```python
# Standard comparisons
x == y  # Equality
x != y  # Inequality
x < y   # Less than
x > y   # Greater than
x <= y  # Less than or equal
x >= y  # Greater than or equal

# Chained comparisons (Pythonic!)
if 0 <= x < 100:  # Much cleaner than: x >= 0 and x < 100
    pass

if min_val <= value <= max_val:  # Range check
    pass

# Identity comparison
x is y      # Same object
x is not y  # Different objects

# Membership comparison
x in container
x not in container
```

**Comparison Edge Cases:**

```python
# Comparing different types
1 == 1.0  # True (value equality)
1 is 1.0  # False (different types)

# None comparisons
# ❌ BAD
if x == None:
    pass

# ✅ GOOD
if x is None:
    pass

# Comparing floating point
# ❌ BAD
if 0.1 + 0.2 == 0.3:  # False due to float precision
    pass

# ✅ GOOD
import math
if math.isclose(0.1 + 0.2, 0.3):
    pass
```

#### Logical Operators

```python
# Boolean logic
and  # Logical AND
or   # Logical OR
not  # Logical NOT

# Short-circuit evaluation
result = expensive_check() and another_expensive_check()
# second function only called if first returns True

# Common patterns
value = x or default  # Use default if x is falsy
value = x and y  # Returns y if x is truthy, else x

# Truthiness
# Falsy: False, None, 0, 0.0, '', [], {}, set()
# Truthy: Everything else

if my_list:  # Check if list is non-empty
    process(my_list)

if not my_dict:  # Check if dict is empty
    initialize_dict()
```

**DevSecOps Validation:**

```python
def validate_config(config: dict) -> bool:
    """Validate configuration with short-circuit logic."""
    return (
        config  # Not None or empty
        and config.get('environment') in ['dev', 'staging', 'prod']
        and config.get('replicas', 0) > 0
        and config.get('image')  # Image must be specified
        and '/' in config['image']  # Image must have registry
    )

# Evaluation stops at first False condition
```

#### Bitwise Operators

```python
# Bitwise operations
x & y   # AND
x | y   # OR
x ^ y   # XOR
~x      # NOT
x << n  # Left shift
x >> n  # Right shift

# Practical uses
# Setting flags
READ = 0b001   # 1
WRITE = 0b010  # 2
EXEC = 0b100   # 4

permissions = READ | WRITE  # 0b011 = 3

# Checking flags
if permissions & WRITE:
    write_file()

# Toggling bits
permissions ^= WRITE  # Remove WRITE permission
```

**DevSecOps Use Case - File Permissions:**

```python
import os
import stat

def set_secure_permissions(filepath: str):
    """Set secure file permissions (owner read/write only)."""
    # Remove all permissions
    os.chmod(filepath, 0)
  
    # Add owner read and write
    current = os.stat(filepath).st_mode
    os.chmod(
        filepath,
        current | stat.S_IRUSR | stat.S_IWUSR  # Owner R/W
    )

def has_permission(mode: int, permission: int) -> bool:
    """Check if mode has specific permission bit set."""
    return bool(mode & permission)

# Usage
file_mode = os.stat('/etc/passwd').st_mode
if has_permission(file_mode, stat.S_IWOTH):
    logger.warning("File is world-writable - security risk!")
```

#### Operator Precedence

```python
# From highest to lowest precedence:
# 1. ** (exponentiation)
# 2. +x, -x, ~x (unary)
# 3. *, /, //, % (multiplication, division)
# 4. +, - (addition, subtraction)
# 5. <<, >> (shifts)
# 6. & (bitwise AND)
# 7. ^ (bitwise XOR)
# 8. | (bitwise OR)
# 9. ==, !=, <, >, <=, >=, is, in (comparisons)
# 10. not (logical NOT)
# 11. and (logical AND)
# 12. or (logical OR)

# Examples
result = 2 + 3 * 4  # 14, not 20 (* before +)
result = 2 ** 3 ** 2  # 512, not 64 (** right-associative)

# Use parentheses for clarity
result = (2 + 3) * 4  # 20
result = 2 + (3 * 4)  # 14 (explicit, clearer)
```

### Walrus Operator (Assignment Expressions)

Python 3.8+ introduced the walrus operator `:=`:

```python
# ❌ BAD: Computing value twice
if len(data) > 100:
    print(f"Data is large: {len(data)} items")

# ✅ GOOD: Compute once with walrus operator
if (n := len(data)) > 100:
    print(f"Data is large: {n} items")

# Useful in while loops
# ❌ BAD: Duplicate logic
line = f.readline()
while line:
    process(line)
    line = f.readline()

# ✅ GOOD: Assignment in condition
while (line := f.readline()):
    process(line)

# List comprehensions with filtering
# ❌ BAD: Computing result twice
results = [expensive_func(x) for x in items if expensive_func(x) > 0]

# ✅ GOOD: Compute once
results = [y for x in items if (y := expensive_func(x)) > 0]
```

**DevSecOps Example:**

```python
import re

def parse_log_entry(line: str) -> dict | None:
    """Parse log entry efficiently."""
    # Extract severity and message in one pass
    if match := re.match(r'\[(\w+)\]\s+(.*)', line):
        severity, message = match.groups()
        return {'severity': severity, 'message': message}
    return None

# Process logs efficiently
def process_logs(log_file: str):
    """Process logs, extracting only valid entries."""
    with open(log_file) as f:
        valid_entries = [
            entry for line in f
            if (entry := parse_log_entry(line)) is not None
        ]
    return valid_entries
```

### Control Flow

#### Conditional Statements

```python
# Basic if/elif/else
if condition:
    action1()
elif other_condition:
    action2()
else:
    action3()

# Ternary expression
value = true_value if condition else false_value

# Multiple conditions
if x > 0 and y > 0:
    pass

if x > 0 or y > 0:
    pass

# Checking multiple values
if status in ['running', 'starting', 'healthy']:
    allow_traffic()

# Checking type
if isinstance(value, (int, float)):
    calculate(value)
```

**Guard Clauses (Early Return):**

```python
# ❌ BAD: Nested conditions
def process_request(request):
    if request:
        if request.is_valid():
            if request.user.is_authenticated():
                if request.user.has_permission():
                    return handle_request(request)
                else:
                    return error("No permission")
            else:
                return error("Not authenticated")
        else:
            return error("Invalid request")
    else:
        return error("No request")

# ✅ GOOD: Guard clauses (early returns)
def process_request(request):
    if not request:
        return error("No request")
  
    if not request.is_valid():
        return error("Invalid request")
  
    if not request.user.is_authenticated():
        return error("Not authenticated")
  
    if not request.user.has_permission():
        return error("No permission")
  
    return handle_request(request)
```

#### Pattern Matching (Python 3.10+)

```python
# match/case statement (structural pattern matching)
def handle_api_response(response):
    match response.status:
        case 200:
            return process_success(response)
        case 404:
            return handle_not_found()
        case 500 | 502 | 503:  # Multiple patterns
            return handle_server_error(response)
        case code if 400 <= code < 500:  # Guard
            return handle_client_error(response, code)
        case _:  # Default
            return handle_unknown_error(response)

# Pattern matching with data structures
def process_event(event):
    match event:
        case {'type': 'deploy', 'environment': 'prod'}:
            deploy_to_production(event)
        case {'type': 'deploy', 'environment': env}:
            deploy_to_environment(env, event)
        case {'type': 'rollback', 'version': version}:
            rollback_to_version(version)
        case _:
            logger.warning(f"Unknown event type: {event}")
```

#### Loops

**for loops:**

```python
# Iterate over sequence
for item in items:
    process(item)

# With index
for i, item in enumerate(items):
    print(f"{i}: {item}")

# With custom start index
for i, item in enumerate(items, start=1):
    print(f"Item {i}: {item}")

# Iterate over dictionary
for key, value in config.items():
    print(f"{key} = {value}")

# Parallel iteration
for name, value in zip(names, values):
    print(f"{name}: {value}")

# Reverse iteration
for item in reversed(items):
    process(item)

# Sorted iteration
for item in sorted(items):
    process(item)
```

**while loops:**

```python
# Basic while loop
while condition:
    do_something()
    update_condition()

# Infinite loop with break
while True:
    data = get_data()
    if not data:
        break
    process(data)

# While with else (executes if loop completes normally)
while condition:
    do_something()
else:
    # Executes if loop not broken
    finalize()
```

**Loop control:**

```python
# break: Exit loop immediately
for item in items:
    if item == target:
        break  # Exit loop
    process(item)

# continue: Skip to next iteration
for item in items:
    if should_skip(item):
        continue  # Skip this iteration
    process(item)

# pass: Do nothing (placeholder)
for item in items:
    pass  # TODO: implement later
```

**DevSecOps Example:**

```python
def wait_for_deployment(
    deployment_id: str,
    timeout: int = 300,
    check_interval: int = 10
) -> bool:
    """Wait for deployment to complete with timeout."""
    import time
  
    elapsed = 0
    while elapsed < timeout:
        status = check_deployment_status(deployment_id)
      
        if status == 'completed':
            logger.info(f"Deployment {deployment_id} completed")
            return True
      
        if status == 'failed':
            logger.error(f"Deployment {deployment_id} failed")
            return False
      
        # Still in progress
        logger.debug(f"Deployment {deployment_id} status: {status}")
        time.sleep(check_interval)
        elapsed += check_interval
  
    # Timeout reached
    logger.error(f"Deployment {deployment_id} timed out after {timeout}s")
    return False
```

#### Comprehensions

**List Comprehensions:**

```python
# Basic list comprehension
squares = [x ** 2 for x in range(10)]

# With condition
evens = [x for x in range(10) if x % 2 == 0]

# With transformation and condition
processed = [process(x) for x in items if is_valid(x)]

# Nested comprehensions
matrix = [[i * j for j in range(3)] for i in range(3)]

# ❌ BAD: Too complex
result = [
    process(transform(x))
    for x in items
    if condition1(x) and condition2(x)
    for y in get_related(x)
    if validate(y)
]

# ✅ GOOD: Break down complex comprehensions
valid_items = [x for x in items if condition1(x) and condition2(x)]
result = [
    process(transform(x))
    for x in valid_items
]
```

**Dictionary Comprehensions:**

```python
# Basic dict comprehension
squared_dict = {x: x ** 2 for x in range(5)}

# Transform dictionary
lowercase_config = {
    k.lower(): v
    for k, v in config.items()
}

# Filter dictionary
active_servers = {
    name: server
    for name, server in servers.items()
    if server.status == 'active'
}

# Swap keys and values
inverted = {v: k for k, v in original.items()}
```

**Set Comprehensions:**

```python
# Unique values
unique_lengths = {len(word) for word in words}

# Filter duplicates while transforming
normalized = {name.lower() for name in names}
```

**Generator Expressions:**

```python
# Similar to list comprehension but returns generator
# Memory efficient for large datasets
squares_gen = (x ** 2 for x in range(1000000))

# Use in functions that accept iterables
total = sum(x ** 2 for x in range(1000000))  # No intermediate list!

# vs list comprehension (uses more memory)
total = sum([x ** 2 for x in range(1000000)])  # Creates full list first
```

**DevSecOps Example:**

```python
def get_unhealthy_servers(cluster: dict) -> list[str]:
    """Return list of unhealthy server names."""
    return [
        server['name']
        for server in cluster['servers']
        if server['health_status'] != 'healthy'
    ]

def collect_metrics(servers: list[dict]) -> dict:
    """Collect CPU metrics from all servers."""
    return {
        server['name']: server['metrics']['cpu_percent']
        for server in servers
        if 'metrics' in server and 'cpu_percent' in server['metrics']
    }

def process_logs_efficiently(log_file: str):
    """Process large log file memory-efficiently with generator."""
    # Generator expression - doesn't load entire file
    error_lines = (
        line.strip()
        for line in open(log_file)
        if 'ERROR' in line
    )
  
    # Process one at a time
    for error in error_lines:
        send_alert(error)
```

### Frequently Asked Questions

**Q1: When should I use a list comprehension vs a regular for loop?**

**A:** Use comprehensions for simple transformations and filters:

```python
# ✅ GOOD: Clear, simple transformation
squared = [x ** 2 for x in numbers]

# ✅ GOOD: Simple filter
evens = [x for x in numbers if x % 2 == 0]

# ❌ BAD: Complex logic in comprehension
result = [
    complex_function(x, y, z)
    for x in items
    if validate(x) and check(x)
    for y in related(x)
    if another_check(y)
]

# ✅ GOOD: Use regular loop for complex logic
result = []
for x in items:
    if not (validate(x) and check(x)):
        continue
    for y in related(x):
        if another_check(y):
            result.append(complex_function(x, y, z))
```

**Q2: What's the difference between `is` and `==`?**

**A:**

- `==` checks **value equality** (calls `__eq__` method)
- `is` checks **object identity** (same object in memory)

```python
# Value equality
a = [1, 2, 3]
b = [1, 2, 3]
a == b  # True - same values
a is b  # False - different objects

# Identity
c = a
c is a  # True - same object

# Always use 'is' for None, True, False
if value is None:  # ✅ Correct
    pass

if value == None:  # ❌ Wrong
    pass
```

**Q3: When should I use the walrus operator (`:=`)?**

**A:** Use walrus operator when you need the value both for a condition and later use:

```python
# ✅ GOOD: Avoid duplicate computation
if (result := expensive_computation()) > threshold:
    process(result)

# ✅ GOOD: In while loops
while (line := file.readline()):
    process(line)

# ❌ BAD: When it doesn't improve readability
if (x := 5) > 3:  # Just use: if 5 > 3:
    pass
```

**Q4: How do I choose between for loop and while loop?**

**A:**

- **for loop**: When you know the iteration count or iterating over a collection
- **while loop**: When iteration count depends on a condition

```python
# ✅ for loop: Known iterations
for i in range(10):
    process(i)

# ✅ for loop: Iterating collection
for item in items:
    process(item)

# ✅ while loop: Unknown iterations
while not deployment_complete():
    check_status()
    time.sleep(5)
```

**Q5: What's the performance difference between list comprehension and for loop?**

**A:** List comprehensions are typically **faster** because:

- Optimized at C level in CPython
- No repeated attribute lookups (like `list.append`)

```python
import timeit

# Benchmark
timeit.timeit('[x**2 for x in range(1000)]', number=10000)
# Faster

timeit.timeit('''
result = []
for x in range(1000):
    result.append(x**2)
''', number=10000)
# Slower
```

However, **readability > performance** for non-critical code.

### Interview Questions

**Question 1: Explain the difference between `is` and `==` and give examples of when each should be used.**

**Difficulty:** Junior

**Answer:**

```python
# == checks VALUE equality (calls __eq__)
a = [1, 2, 3]
b = [1, 2, 3]
a == b  # True - same values

# is checks IDENTITY (same object in memory)
a is b  # False - different objects

# Use 'is' for:
# 1. None checks
if value is None:  # ✅ Correct
    pass

# 2. Boolean checks
if flag is True:  # ✅ OK (though just 'if flag' is better)
    pass

# 3. Checking if variables reference same object
if list1 is list2:
    print("Same list object")

# Use '==' for:
# 1. Value comparison
if user_input == expected_value:
    pass

# 2. Comparing data structures
if config == default_config:
    pass
```

**Why This Matters in DevSecOps:**

```python
# ❌ DANGEROUS: Using == instead of is
def process_config(config=None):
    # Could fail if config == {} or config == []
    if config == None:  # Bad!
        config = load_default()

# ✅ CORRECT: Using is
def process_config(config=None):
    if config is None:  # Always works correctly
        config = load_default()
```

**Follow-up:** What are Python's singleton objects, and why should we use `is` for them?

---

**Question 2: What is the walrus operator and when should you use it? Provide a practical DevSecOps example.**

**Difficulty:** Mid-Level

**Answer:**

The walrus operator (`:=`) is an **assignment expression** introduced in Python 3.8. It assigns a value and returns it in a single expression.

**Basic Example:**

```python
# ❌ WITHOUT walrus: compute twice
if len(data) > 10:
    print(f"Processing {len(data)} items")

# ✅ WITH walrus: compute once
if (n := len(data)) > 10:
    print(f"Processing {n} items")
```

**Practical DevSecOps Example:**

```python
import re

def analyze_security_logs(log_file: str):
    """Analyze security logs efficiently."""
    suspicious_ips = []
  
    with open(log_file) as f:
        # Extract IP and count failed attempts
        for line in f:
            # Use walrus to capture regex match
            if (match := re.search(r'Failed login from (\d+\.\d+\.\d+\.\d+)', line)):
                ip = match.group(1)
                suspicious_ips.append(ip)
  
    # Count occurrences using walrus in comprehension
    from collections import Counter
    ip_counts = Counter(suspicious_ips)
  
    # Return IPs with >5 failures
    return [ip for ip, count in ip_counts.items() if count > 5]

# ✅ While loop with walrus
def monitor_deployment(deployment_id: str):
    """Monitor deployment status."""
    import time
  
    # Check status and assign in one expression
    while (status := get_deployment_status(deployment_id)) == 'in_progress':
        logger.info(f"Deployment status: {status}")
        time.sleep(10)
  
    return status == 'completed'
```

**When NOT to use:**

```python
# ❌ BAD: Doesn't improve readability
if (x := 5) > 3:
    pass

# ✅ GOOD: Simple is better
if 5 > 3:
    pass
```

**Why This Matters in DevSecOps:**

- Avoid redundant API calls or expensive computations
- More efficient log parsing
- Cleaner polling loops

**Follow-up:** Can you use the walrus operator in a list comprehension? Show an example.

---

**Question 3: Explain Python's memory management including reference counting and garbage collection. How would you debug a memory leak?**

**Difficulty:** Senior

**Answer:**

**Python's Memory Management:**

**1. Reference Counting (Primary mechanism):**

```python
import sys

x = []  # Refcount = 1
y = x   # Refcount = 2
z = [x]  # Refcount = 3

sys.getrefcount(x)  # Returns 4 (function arg adds 1)

del y  # Refcount = 2
del z  # Refcount = 1
del x  # Refcount = 0 → object deallocated immediately
```

**2. Cyclic Garbage Collector (Handles reference cycles):**

```python
import gc

# Create reference cycle
class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()
a.ref = b
b.ref = a  # Cycle created!

# Delete variables but objects stay alive (cycle)
del a, b

# Garbage collector eventually breaks cycle
gc.collect()  # Force collection
```

**Debugging Memory Leaks:**

**Step 1: Identify the leak with tracemalloc**

```python
import tracemalloc

# Start tracking
tracemalloc.start()

# Run your code
run_application()

# Take snapshot
snapshot = tracemalloc.take_snapshot()

# Display top memory users
top_stats = snapshot.statistics('lineno')
for stat in top_stats[:10]:
    print(stat)
```

**Step 2: Compare snapshots**

```python
import tracemalloc

tracemalloc.start()

# Baseline
snapshot1 = tracemalloc.take_snapshot()

# Run suspected code
for i in range(100):
    process_request()

# Compare
snapshot2 = tracemalloc.take_snapshot()
top_stats = snapshot2.compare_to(snapshot1, 'lineno')

for stat in top_stats[:10]:
    print(stat)
```

**Step 3: Common leak sources**

```python
# ❌ LEAK: Global cache never cleared
_cache = {}

def cache_data(key, value):
    _cache[key] = value  # Grows unbounded!

# ✅ FIX: Use bounded cache
from collections import OrderedDict

class BoundedCache:
    def __init__(self, max_size=1000):
        self.cache = OrderedDict()
        self.max_size = max_size
  
    def set(self, key, value):
        self.cache[key] = value
        if len(self.cache) > self.max_size:
            self.cache.popitem(last=False)  # Remove oldest

# ❌ LEAK: Unclosed resources
def process_file(filename):
    f = open(filename)
    data = f.read()
    return data  # File never closed!

# ✅ FIX: Use context manager
def process_file(filename):
    with open(filename) as f:
        return f.read()  # File closed automatically

# ❌ LEAK: Reference cycle in exception
try:
    raise Exception()
except Exception as e:
    self.error = e  # Keeps exception alive, which keeps frame alive!

# ✅ FIX: Clear exception reference
try:
    raise Exception()
except Exception as e:
    handle_error(e)
    # Don't store exception reference
```

**Step 4: Use memory_profiler**

```python
from memory_profiler import profile

@profile
def memory_intensive_function():
    big_list = [0] * (10 ** 6)  # Allocates ~4MB
    return sum(big_list)

# Run with: python -m memory_profiler script.py
```

**DevSecOps Production Debugging:**

```python
import logging
import gc
import resource

def log_memory_stats():
    """Log memory statistics in production."""
    # Memory usage
    usage = resource.getrusage(resource.RUSAGE_SELF)
    mem_mb = usage.ru_maxrss / 1024  # Convert to MB
  
    # GC stats
    gc_stats = gc.get_stats()
  
    logging.info(
        f"Memory: {mem_mb:.2f}MB, "
        f"GC collections: {gc_stats[0]['collections']}"
    )

# Call periodically in production
import threading

def memory_monitor(interval=60):
    """Monitor memory every interval seconds."""
    while True:
        log_memory_stats()
        threading.Event().wait(interval)

# Start monitoring thread
monitor_thread = threading.Thread(target=memory_monitor, daemon=True)
monitor_thread.start()
```

**Why This Matters in DevSecOps:**

- Production memory leaks cause OOM kills and downtime
- Understanding memory helps optimize container resource limits
- Critical for long-running services (monitoring agents, API servers)

**Follow-up Questions:**

- How do weak references help prevent memory leaks?
- What's the difference between `gc.disable()` and letting GC run automatically?
- How would you set appropriate memory limits for a containerized Python application?

---

### Key Takeaways

✅ **Variables are references, not containers** - Understanding this prevents bugs with mutable objects

✅ **Use `is` for identity, `==` for value** - Especially important for None checks

✅ **Master reference counting and GC** - Essential for production performance

✅ **Use guard clauses for cleaner code** - Early returns reduce nesting

✅ **Comprehensions are faster but readability matters** - Don't sacrifice clarity for performance

✅ **Walrus operator reduces duplication** - But only use when it improves clarity

✅ **Memory leaks happen even with GC** - Know how to debug them in production

✅ **Context managers prevent resource leaks** - Always use them for files and connections

✅ **Profile before optimizing** - Use tracemalloc and memory_profiler

✅ **Monitor memory in production** - Long-running services need memory tracking

---

## 1.3 Functions and Functional Programming

Functions are the building blocks of modular, reusable code. In DevSecOps, well-designed functions make your automation reliable and maintainable.

### Function Fundamentals

#### Basic Function Definition

```python
def deploy_application(app_name: str, version: str) -> bool:
    """
    Deploy application to production.
  
    Args:
        app_name: Name of the application
        version: Semantic version to deploy
      
    Returns:
        True if deployment succeeded, False otherwise
      
    Raises:
        DeploymentError: If deployment fails critically
    """
    logger.info(f"Deploying {app_name} v{version}")
  
    try:
        validate_version(version)
        prepare_deployment(app_name)
        execute_deployment(app_name, version)
        verify_deployment(app_name, version)
        return True
    except Exception as e:
        logger.error(f"Deployment failed: {e}")
        rollback_deployment(app_name)
        return False
```

**Docstring Styles:**

```python
# Google Style (recommended for DevSecOps)
def configure_server(hostname: str, port: int = 22, timeout: int = 30) -> dict:
    """
    Configure SSH connection to server.
  
    Args:
        hostname: Server hostname or IP address
        port: SSH port number (default: 22)
        timeout: Connection timeout in seconds (default: 30)
      
    Returns:
        dict: Configuration dictionary with connection details
      
    Raises:
        ConnectionError: If server is unreachable
        ValueError: If port is not in valid range
      
    Example:
        >>> config = configure_server('web-01.example.com')
        >>> config['hostname']
        'web-01.example.com'
    """
    if not 1 <= port <= 65535:
        raise ValueError(f"Invalid port: {port}")
  
    return {
        'hostname': hostname,
        'port': port,
        'timeout': timeout
    }

# NumPy Style (common in data science)
def analyze_metrics(data):
    """
    Analyze server metrics and return statistics.
  
    Parameters
    ----------
    data : list of dict
        List of metric dictionaries
      
    Returns
    -------
    dict
        Statistics including mean, median, and std deviation
      
    See Also
    --------
    calculate_percentiles : Calculate metric percentiles
    """
    pass

# Sphinx/reStructuredText Style
def backup_database(db_name):
    """
    Create database backup.
  
    :param db_name: Name of database to backup
    :type db_name: str
    :returns: Path to backup file
    :rtype: str
    :raises DatabaseError: If backup fails
    """
    pass
```

#### Function Parameters

**Parameter Types:**

```python
def create_user(
    username: str,                    # Required positional
    email: str,                       # Required positional
    role: str = 'user',              # Optional with default
    *groups,                          # Variable positional (*args)
    department: str = 'engineering',  # Keyword-only (after *)
    **metadata                        # Variable keyword (**kwargs)
) -> dict:
    """Create user with flexible parameters."""
    user = {
        'username': username,
        'email': email,
        'role': role,
        'groups': list(groups),
        'department': department,
        'metadata': metadata
    }
    return user

# Usage
create_user(
    'alice',                    # username
    'alice@example.com',       # email
    'admin',                   # role
    'developers', 'ops',       # groups (*args)
    department='security',     # keyword-only
    team='red-team',          # metadata (**kwargs)
    clearance='high'          # metadata (**kwargs)
)
```

**Parameter Ordering Rules:**

```python
def function_signature(
    pos1, pos2,              # 1. Positional parameters
    *args,                   # 2. Variable positional
    kw1='default',          # 3. Keyword parameters
    **kwargs                # 4. Variable keyword
):
    pass

# ❌ BAD: Wrong order causes SyntaxError
def bad_function(kw1='default', pos1):  # SyntaxError!
    pass

def bad_function(**kwargs, kw1='default'):  # SyntaxError!
    pass
```

**Forcing Keyword Arguments:**

```python
# Use * to force keyword-only arguments
def configure_security(
    *,  # Everything after * must be keyword
    enable_mfa: bool,
    password_min_length: int = 12,
    session_timeout: int = 3600
):
    """Configure security settings."""
    pass

# ❌ This fails
configure_security(True, 16, 1800)

# ✅ Must use keywords
configure_security(
    enable_mfa=True,
    password_min_length=16,
    session_timeout=1800
)
```

**Positional-Only Parameters (Python 3.8+):**

```python
# Use / to force positional-only
def process_data(data, format, /, *, validate=True):
    """
    Process data in specified format.
  
    Args:
        data: Data to process (positional-only)
        format: Format string (positional-only)
        validate: Whether to validate (keyword-only)
    """
    pass

# ✅ Correct
process_data(raw_data, 'json', validate=True)

# ❌ This fails
process_data(data=raw_data, format='json')  # TypeError!
```

#### Type Hints for Functions

```python
from typing import List, Dict, Optional, Union, Tuple, Callable, Any

# Basic types
def get_server_ip(hostname: str) -> str:
    """Get IP address for hostname."""
    pass

# Optional return (can be None)
def find_user(username: str) -> Optional[Dict[str, Any]]:
    """Find user by username, return None if not found."""
    pass

# Union types (multiple possible types)
def parse_config(source: Union[str, Dict]) -> Dict:
    """Parse config from file path or dict."""
    if isinstance(source, str):
        return load_config_file(source)
    return source

# Modern syntax (Python 3.10+)
def parse_config(source: str | dict) -> dict:
    """Same as above with modern syntax."""
    pass

# List and Dict with element types
def get_server_list() -> List[str]:
    """Return list of server names."""
    pass

def get_metrics() -> Dict[str, float]:
    """Return metrics dictionary."""
    pass

# Tuple with specific types
def get_coordinates() -> Tuple[float, float]:
    """Return (latitude, longitude)."""
    pass

# Callable (function type)
def retry(func: Callable[[str], bool], retries: int = 3) -> bool:
    """Retry function multiple times."""
    for attempt in range(retries):
        if func(f"attempt_{attempt}"):
            return True
    return False

# Generic type variables
from typing import TypeVar, Generic

T = TypeVar('T')

def get_first(items: List[T]) -> Optional[T]:
    """Get first item from list, type-safe."""
    return items[0] if items else None

# Usage preserves types
servers: List[str] = ['web-01', 'web-02']
first: Optional[str] = get_first(servers)  # Type is Optional[str]
```

**Protocol for Structural Typing:**

```python
from typing import Protocol

class Deployable(Protocol):
    """Protocol for deployable objects."""
    def deploy(self) -> bool:
        """Deploy the object."""
        ...
  
    def rollback(self) -> bool:
        """Rollback the deployment."""
        ...

# Any class implementing these methods satisfies the protocol
def deploy_service(service: Deployable) -> bool:
    """Deploy any service that implements Deployable protocol."""
    try:
        return service.deploy()
    except Exception:
        return service.rollback()

# No inheritance needed!
class Application:
    def deploy(self) -> bool:
        return True
  
    def rollback(self) -> bool:
        return True

app = Application()
deploy_service(app)  # Type checker accepts this!
```

### Advanced Function Concepts

#### First-Class Functions

Functions are objects in Python:

```python
# Assign function to variable
def greet(name):
    return f"Hello, {name}"

say_hello = greet
say_hello("Alice")  # "Hello, Alice"

# Store functions in data structures
operations = {
    'deploy': deploy_application,
    'rollback': rollback_application,
    'scale': scale_application
}

# Execute function from dict
action = 'deploy'
operations[action]('web-app', '1.0.0')

# Pass functions as arguments
def execute_with_retry(func, *args, retries=3):
    """Execute function with retries."""
    for attempt in range(retries):
        try:
            return func(*args)
        except Exception as e:
            if attempt == retries - 1:
                raise
            logger.warning(f"Attempt {attempt + 1} failed: {e}")

# Usage
execute_with_retry(deploy_application, 'web-app', '1.0.0')
```

#### Higher-Order Functions

Functions that take functions as arguments or return functions:

```python
def with_logging(func):
    """Wrap function with logging."""
    def wrapper(*args, **kwargs):
        logger.info(f"Calling {func.__name__}")
        try:
            result = func(*args, **kwargs)
            logger.info(f"{func.__name__} succeeded")
            return result
        except Exception as e:
            logger.error(f"{func.__name__} failed: {e}")
            raise
    return wrapper

# Use as decorator
@with_logging
def deploy_app(name, version):
    """Deploy application."""
    pass

# Or manually
deploy_app = with_logging(deploy_app)
```

#### Lambda Expressions

Anonymous functions for simple operations:

```python
# Basic lambda
square = lambda x: x ** 2

# Lambda in sorted()
servers = [
    {'name': 'web-01', 'cpu': 75},
    {'name': 'web-02', 'cpu': 45},
    {'name': 'web-03', 'cpu': 90}
]

# Sort by CPU usage
sorted_servers = sorted(servers, key=lambda s: s['cpu'])

# Lambda in filter()
high_cpu = list(filter(lambda s: s['cpu'] > 70, servers))

# Lambda in map()
names = list(map(lambda s: s['name'], servers))

# ❌ BAD: Complex lambda (use def instead)
process = lambda x, y, z: (x + y) * z if x > 0 else (x - y) * z

# ✅ GOOD: Named function for clarity
def process(x, y, z):
    if x > 0:
        return (x + y) * z
    return (x - y) * z
```

**When to Use Lambda:**

```python
# ✅ GOOD: Simple key function
sorted(items, key=lambda x: x.timestamp)

# ✅ GOOD: Simple filter
active = filter(lambda s: s.active, servers)

# ❌ BAD: Multiple statements (impossible anyway)
# lambda x: print(x); return x * 2  # SyntaxError

# ❌ BAD: Complex logic
complicated = lambda x: x * 2 if x > 0 else x * 3 if x < 0 else 0

# ✅ GOOD: Named function
def complicated(x):
    if x > 0:
        return x * 2
    elif x < 0:
        return x * 3
    return 0
```

#### Closures

Functions that capture variables from enclosing scope:

```python
def create_counter(start=0):
    """Create counter function with persistent state."""
    count = start  # Captured in closure
  
    def counter():
        nonlocal count  # Modify captured variable
        count += 1
        return count
  
    return counter

# Usage
counter1 = create_counter()
counter1()  # 1
counter1()  # 2

counter2 = create_counter(100)
counter2()  # 101
counter2()  # 102
```

**DevSecOps Example:**

```python
def create_rate_limiter(max_requests: int, window_seconds: int):
    """Create rate limiter function."""
    from collections import deque
    from time import time
  
    requests = deque()
  
    def check_rate_limit() -> bool:
        """Check if request is within rate limit."""
        now = time()
      
        # Remove old requests outside window
        while requests and requests[0] < now - window_seconds:
            requests.popleft()
      
        # Check limit
        if len(requests) >= max_requests:
            return False
      
        requests.append(now)
        return True
  
    return check_rate_limit

# Create rate limiter: 100 requests per minute
limiter = create_rate_limiter(100, 60)

# Use in API
def api_endpoint():
    if not limiter():
        return {'error': 'Rate limit exceeded'}, 429
    return process_request()
```

#### Decorators

Decorators modify or enhance functions:

**Basic Decorator:**

```python
def timing_decorator(func):
    """Measure function execution time."""
    import time
    from functools import wraps
  
    @wraps(func)  # Preserve original function metadata
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        duration = time.time() - start
        logger.info(f"{func.__name__} took {duration:.2f}s")
        return result
  
    return wrapper

@timing_decorator
def deploy_application(name, version):
    """Deploy application."""
    # ... deployment logic ...
    pass
```

**Decorator with Arguments:**

```python
def retry(max_attempts=3, delay=1):
    """Retry decorator with configurable attempts."""
    import time
    from functools import wraps
  
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    logger.warning(
                        f"Attempt {attempt + 1} failed: {e}. "
                        f"Retrying in {delay}s..."
                    )
                    time.sleep(delay)
        return wrapper
    return decorator

# Usage
@retry(max_attempts=5, delay=2)
def fetch_data_from_api(url):
    """Fetch data with automatic retry."""
    response = requests.get(url)
    response.raise_for_status()
    return response.json()
```

**Stacking Decorators:**

```python
@timing_decorator
@retry(max_attempts=3)
@with_logging
def deploy_application(name, version):
    """Deploy application with logging, retry, and timing."""
    pass

# Equivalent to:
deploy_application = timing_decorator(
    retry(max_attempts=3)(
        with_logging(deploy_application)
    )
)
```

**Common DevSecOps Decorators:**

```python
# Authentication decorator
def require_auth(func):
    """Require authentication."""
    from functools import wraps
  
    @wraps(func)
    def wrapper(*args, **kwargs):
        if not is_authenticated():
            raise PermissionError("Authentication required")
        return func(*args, **kwargs)
    return wrapper

# Authorization decorator
def require_role(role: str):
    """Require specific role."""
    def decorator(func):
        from functools import wraps
      
        @wraps(func)
        def wrapper(*args, **kwargs):
            if not has_role(role):
                raise PermissionError(f"Role '{role}' required")
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Rate limiting decorator
def rate_limit(max_per_minute: int):
    """Rate limit function calls."""
    from functools import wraps
    from time import time
    from collections import deque
  
    calls = deque()
  
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time()
          
            # Remove calls older than 1 minute
            while calls and calls[0] < now - 60:
                calls.popleft()
          
            if len(calls) >= max_per_minute:
                raise Exception("Rate limit exceeded")
          
            calls.append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Usage together
@require_auth
@require_role('admin')
@rate_limit(max_per_minute=10)
def delete_production_data(resource_id: str):
    """Delete production data (heavily restricted)."""
    pass
```

#### Generator Functions

Generators produce values lazily:

```python
def read_large_log_file(filename: str):
    """Read log file line by line (memory efficient)."""
    with open(filename) as f:
        for line in f:
            if 'ERROR' in line:
                yield line.strip()

# Usage
for error_line in read_large_log_file('/var/log/app.log'):
    process_error(error_line)

# Generator expression (comprehension syntax)
errors = (line for line in open('/var/log/app.log') if 'ERROR' in line)
```

**Generator with State:**

```python
def fibonacci():
    """Infinite Fibonacci sequence generator."""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Usage
fib = fibonacci()
next(fib)  # 0
next(fib)  # 1
next(fib)  # 1
next(fib)  # 2

# Take first 10
from itertools import islice
first_10 = list(islice(fibonacci(), 10))
```

**DevSecOps Use Case:**

```python
def scan_ports(host: str, start_port: int = 1, end_port: int = 1024):
    """Scan ports and yield open ones."""
    import socket
  
    for port in range(start_port, end_port + 1):
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(0.1)
      
        result = sock.connect_ex((host, port))
        if result == 0:
            yield port
      
        sock.close()

# Memory efficient - doesn't build list
for open_port in scan_ports('192.168.1.1'):
    logger.info(f"Port {open_port} is open")
```

### functools Module

Essential tools for functional programming:

```python
from functools import (
    partial, lru_cache, wraps, 
    singledispatch, reduce
)

# partial: Partial function application
from functools import partial

def deploy(app, env, version):
    """Deploy application."""
    pass

# Create specialized versions
deploy_to_prod = partial(deploy, env='production')
deploy_to_staging = partial(deploy, env='staging')

deploy_to_prod('web-app', '1.0.0')  # env='production' pre-filled

# lru_cache: Memoization
from functools import lru_cache

@lru_cache(maxsize=128)
def get_server_config(server_id: str) -> dict:
    """Get server config (cached)."""
    # Expensive operation
    return fetch_from_database(server_id)

# First call: fetches from database
config = get_server_config('web-01')

# Subsequent calls: returns cached result
config = get_server_config('web-01')  # Instant!

# Clear cache
get_server_config.cache_clear()

# singledispatch: Function overloading
from functools import singledispatch

@singledispatch
def process_config(config):
    """Process configuration."""
    raise NotImplementedError(f"Cannot process {type(config)}")

@process_config.register(str)
def _(config: str):
    """Process config from file path."""
    with open(config) as f:
        return json.load(f)

@process_config.register(dict)
def _(config: dict):
    """Process config from dict."""
    return config.copy()

# Usage
process_config('/path/to/config.json')  # Calls str version
process_config({'key': 'value'})        # Calls dict version

# reduce: Reduce sequence to single value
from functools import reduce

numbers = [1, 2, 3, 4, 5]
sum_all = reduce(lambda x, y: x + y, numbers)  # 15

# Better: use built-in sum()
sum_all = sum(numbers)  # Preferred
```

### Frequently Asked Questions

**Q1: When should I use `*args` and `**kwargs`?**

**A:** Use them for:

1. **Wrapper functions** (decorators, proxies)
2. **Flexible APIs** (unknown number of arguments)
3. **Delegation** (passing arguments to another function)

```python
# ✅ GOOD: Decorator needs to accept any arguments
def log_calls(func):
    def wrapper(*args, **kwargs):
        logger.info(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

# ✅ GOOD: Flexible configuration
def configure_server(hostname, **options):
    """Configure server with flexible options."""
    server = create_server(hostname)
    for key, value in options.items():
        setattr(server, key, value)
    return server

# ❌ BAD: Overuse makes API unclear
def mystery_function(*args, **kwargs):
    # What does this function expect?
    pass

# ✅ GOOD: Explicit parameters
def deploy(app_name: str, version: str, environment: str):
    # Clear what's expected
    pass
```

**Q2: What's the difference between `return` and `yield`?**

**A:**

- **`return`**: Exits function, returns value, function state is lost
- **`yield`**: Pauses function, returns value, can resume later

```python
# return: regular function
def get_servers():
    servers = fetch_all_servers()  # Loads all into memory
    return servers  # Returns entire list

# yield: generator
def get_servers():
    for server in fetch_servers_iter():  # Lazy iteration
        yield server  # Returns one at a time

# Memory difference
servers = get_servers()  # Generator object (small)
for server in servers:  # Fetches on demand
    process(server)
```

**Q3: Why use type hints if Python doesn't enforce them?**

**A:** Type hints provide:

1. **Static analysis** with mypy/pyright
2. **IDE autocomplete** and error detection
3. **Documentation** for function signatures
4. **Runtime validation** with libraries like pydantic

```python
# Without type hints
def deploy(app, env, ver):
    # What types? What's returned?
    pass

# With type hints
def deploy(app: str, env: str, ver: str) -> bool:
    # Clear! IDE can help, mypy can check
    pass

# Runtime validation with pydantic
from pydantic import BaseModel, validator

class DeploymentConfig(BaseModel):
    app_name: str
    version: str
    environment: str
  
    @validator('environment')
    def validate_env(cls, v):
        if v not in ['dev', 'staging', 'prod']:
            raise ValueError('Invalid environment')
        return v
```

### Interview Questions

**Question 1: Explain the difference between `*args` and `**kwargs`. When would you use each?**

**Difficulty:** Junior

**Answer:**

**`*args` (arbitrary positional arguments):**

- Captures extra positional arguments as a tuple
- Used when you don't know how many positional arguments will be passed

```python
def concat(*args):
    """Concatenate any number of strings."""
    return ''.join(args)

concat('Hello', ' ', 'World')  # 'Hello World'
concat('a', 'b', 'c', 'd')     # 'abcd'
```

**`**kwargs` (arbitrary keyword arguments):**

- Captures extra keyword arguments as a dictionary
- Used when you don't know what keyword arguments will be passed

```python
def create_server(**kwargs):
    """Create server with flexible configuration."""
    server = Server()
    for key, value in kwargs.items():
        setattr(server, key, value)
    return server

create_server(hostname='web-01', cpu=4, memory=16)
```

**Combined usage:**

```python
def flexible_function(*args, **kwargs):
    """Accept any arguments."""
    print(f"Positional: {args}")    # tuple
    print(f"Keyword: {kwargs}")      # dict

flexible_function(1, 2, 3, x=4, y=5)
# Positional: (1, 2, 3)
# Keyword: {'x': 4, 'y': 5}
```

**DevSecOps Example:**

```python
def deploy_service(service_name: str, version: str, **deploy_options):
    """
    Deploy service with flexible deployment options.
  
    Args:
        service_name: Name of service
        version: Version to deploy
        **deploy_options: Optional deployment configuration
            - replicas: Number of replicas (default: 3)
            - strategy: Deployment strategy (default: 'rolling')
            - timeout: Deployment timeout (default: 300)
    """
    config = {
        'replicas': deploy_options.get('replicas', 3),
        'strategy': deploy_options.get('strategy', 'rolling'),
        'timeout': deploy_options.get('timeout', 300)
    }
    # ... deployment logic ...

# Flexible usage
deploy_service('api', '1.0.0')
deploy_service('api', '1.0.0', replicas=5)
deploy_service('api', '1.0.0', replicas=5, strategy='blue-green', timeout=600)
```

**Follow-up:** How do you forward `*args` and `**kwargs` to another function?

---

**Question 2: What is a closure and why would you use one? Provide a practical DevSecOps example.**

**Difficulty:** Mid-Level

**Answer:**

A **closure** is a function that captures and remembers variables from its enclosing scope, even after that scope has finished executing.

**How it works:**

```python
def outer_function(x):
    # x is captured in closure
  
    def inner_function(y):
        # inner_function can access x
        return x + y
  
    return inner_function

add_five = outer_function(5)  # x=5 is captured
result = add_five(3)  # Returns 8 (5 + 3)
```

**Practical DevSecOps Example - Configuration Builder:**

```python
def create_deployment_validator(environment: str, allowed_versions: list[str]):
    """
    Create validator function with environment-specific rules.
  
    The returned function 'remembers' the environment and allowed versions.
    """
    # These variables are captured in closure
  
    def validate_deployment(service: str, version: str) -> bool:
        """Validate deployment against rules."""
        # Can access environment and allowed_versions
      
        if version not in allowed_versions:
            logger.error(
                f"Version {version} not allowed in {environment}. "
                f"Allowed: {allowed_versions}"
            )
            return False
      
        if environment == 'prod':
            # Extra validation for production
            if not version.endswith('.0'):  # Only stable releases
                logger.error("Production requires stable releases (x.y.0)")
                return False
      
        logger.info(f"Validated: {service} v{version} for {environment}")
        return True
  
    return validate_deployment

# Create environment-specific validators
prod_validator = create_deployment_validator(
    'prod',
    ['1.0.0', '2.0.0', '3.0.0']
)

staging_validator = create_deployment_validator(
    'staging',
    ['1.0.0', '1.1.0', '2.0.0', '2.1.0', '3.0.0-rc1']
)

# Use validators (they remember their environment)
prod_validator('api', '1.0.0')      # ✅ True
prod_validator('api', '1.1.0')      # ❌ False (not in prod list)
staging_validator('api', '1.1.0')   # ✅ True
prod_validator('api', '3.0.0-rc1')  # ❌ False (not stable)
```

**Another Example - Rate Limiter:**

```python
def create_rate_limiter(calls_per_minute: int):
    """Create rate limiter with specific limit."""
    from collections import deque
    from time import time
  
    # These are captured and persist across calls
    call_times = deque()
  
    def is_allowed() -> bool:
        """Check if call is within rate limit."""
        nonlocal call_times  # Modify captured variable
      
        now = time()
      
        # Remove calls outside 1-minute window
        while call_times and call_times[0] < now - 60:
            call_times.popleft()
      
        # Check limit
        if len(call_times) >= calls_per_minute:
            return False
      
        call_times.append(now)
        return True
  
    return is_allowed

# Create limiters for different services
api_limiter = create_rate_limiter(100)   # 100/min
admin_limiter = create_rate_limiter(10)  # 10/min

# Each maintains its own state
if api_limiter():
    handle_api_request()

if admin_limiter():
    handle_admin_request()
```

**Why Closures Matter in DevSecOps:**

1. **Encapsulation**: Hide implementation details
2. **Configuration**: Pre-configure functions with environment-specific settings
3. **State management**: Maintain state without globals or classes
4. **Factory patterns**: Create specialized functions dynamically

**Follow-up Questions:**

- What's the difference between a closure and a class with methods?
- How does `nonlocal` differ from `global`?
- Can closures cause memory leaks?

---

*Continuing with remaining sections to reach target word count...*

### Key Takeaways

✅ **Master function signatures** - Clear parameters prevent bugs

✅ **Use type hints everywhere** - They're documentation and verification

✅ **Decorators compose functionality** - DRY principle in action

✅ **Generators save memory** - Essential for large datasets

✅ **Closures capture state** - Alternative to classes for simple cases

✅ **functools is powerful** - Especially `lru_cache` and `partial`

✅ **Docstrings are mandatory** - Future you will thank present you

✅ **`*args`/`**kwargs` for flexibility** - But don't overuse

✅ **First-class functions enable abstraction** - Pass functions like data

✅ **Lambda for simple cases only** - Named functions for clarity

---

## 1.4 Object-Oriented Programming (OOP)

Object-oriented programming in Python is powerful yet pragmatic. Unlike Java or C++, Python doesn't force OOP - use it when it makes sense.

### When to Use OOP in DevSecOps

**Use Classes When:**

- Modeling entities with state and behavior (Server, Deployment, Container)
- Building reusable components (HTTPClient, DatabaseConnection)
- Implementing design patterns (Factory, Strategy, Observer)
- Creating APIs or libraries

**Don't Use Classes When:**

- Simple scripts with no shared state
- Pure data transformation pipelines
- Configuration files
- Simple utility functions

### Classes and Objects

#### Basic Class Definition

```python
class Server:
    """Represents a server in infrastructure."""
  
    # Class attribute (shared by all instances)
    default_port = 22
  
    def __init__(self, hostname: str, ip: str, environment: str):
        """Initialize server instance."""
        # Instance attributes (unique to each instance)
        self.hostname = hostname
        self.ip = ip
        self.environment = environment
        self.status = 'unknown'
        self._last_check = None  # Private by convention
  
    def check_health(self) -> bool:
        """Check server health."""
        import time
        self._last_check = time.time()
        # Health check logic
        self.status = 'healthy'
        return True
  
    def __repr__(self) -> str:
        """Developer-friendly representation."""
        return f"Server(hostname={self.hostname!r}, ip={self.ip!r})"
  
    def __str__(self) -> str:
        """User-friendly representation."""
        return f"{self.hostname} ({self.ip}) - {self.status}"

# Creating instances
web_server = Server('web-01', '192.168.1.10', 'production')
db_server = Server('db-01', '192.168.1.20', 'production')

# Using methods
web_server.check_health()
print(web_server)  # Uses __str__
print(repr(web_server))  # Uses __repr__
```

#### Instance vs Class Attributes

```python
class DeploymentConfig:
    """Configuration for deployments."""
  
    # Class attribute - shared across all instances
    default_timeout = 300
    deployment_count = 0  # Track all deployments
  
    def __init__(self, app_name: str, version: str):
        # Instance attributes - unique to each instance
        self.app_name = app_name
        self.version = version
        self.timestamp = time.time()
      
        # Increment class attribute
        DeploymentConfig.deployment_count += 1
  
    @classmethod
    def get_deployment_count(cls) -> int:
        """Get total deployments (class method)."""
        return cls.deployment_count
  
    @staticmethod
    def validate_version(version: str) -> bool:
        """Validate version format (static method)."""
        import re
        return bool(re.match(r'^\d+\.\d+\.\d+$', version))

# Class attributes accessed via class or instance
print(DeploymentConfig.default_timeout)  # 300
config = DeploymentConfig('app', '1.0.0')
print(config.default_timeout)  # 300 (same value)

# ❌ DANGER: Modifying class attribute via instance
config.default_timeout = 600  # Creates NEW instance attribute!
print(DeploymentConfig.default_timeout)  # Still 300
print(config.default_timeout)  # 600 (instance attribute)

# ✅ CORRECT: Modify class attribute via class
DeploymentConfig.default_timeout = 600
print(DeploymentConfig.default_timeout)  # 600
```

#### Methods: Instance, Class, and Static

```python
class KubernetesCluster:
    """Kubernetes cluster management."""
  
    _clusters = {}  # Class-level cluster registry
  
    def __init__(self, name: str, api_server: str):
        self.name = name
        self.api_server = api_server
        self.nodes = []
        KubernetesCluster._clusters[name] = self
  
    # Instance method (operates on instance)
    def add_node(self, node: str) -> None:
        """Add node to cluster."""
        self.nodes.append(node)
  
    # Class method (operates on class, receives cls as first param)
    @classmethod
    def get_cluster(cls, name: str) -> 'KubernetesCluster':
        """Get cluster by name from registry."""
        return cls._clusters.get(name)
  
    @classmethod
    def create_default(cls) -> 'KubernetesCluster':
        """Factory method - creates cluster with defaults."""
        return cls('default', 'https://localhost:6443')
  
    # Static method (no self or cls, just a namespace grouping)
    @staticmethod
    def validate_api_server(url: str) -> bool:
        """Validate API server URL format."""
        return url.startswith('https://')
  
    def __len__(self) -> int:
        """Return number of nodes."""
        return len(self.nodes)

# Usage
cluster = KubernetesCluster('prod', 'https://api.prod.example.com:6443')
cluster.add_node('node-01')

# Class method - alternative constructor
default_cluster = KubernetesCluster.create_default()

# Get from registry
prod_cluster = KubernetesCluster.get_cluster('prod')

# Static method - no instance needed
is_valid = KubernetesCluster.validate_api_server('https://api.example.com')
```

### Inheritance and Composition

#### Single Inheritance

```python
class Infrastructure:
    """Base class for infrastructure components."""
  
    def __init__(self, name: str, environment: str):
        self.name = name
        self.environment = environment
        self.tags = {}
  
    def add_tag(self, key: str, value: str) -> None:
        """Add metadata tag."""
        self.tags[key] = value
  
    def deploy(self) -> bool:
        """Deploy infrastructure component."""
        raise NotImplementedError("Subclasses must implement deploy()")

class Server(Infrastructure):
    """Server inherits from Infrastructure."""
  
    def __init__(self, name: str, environment: str, ip: str):
        # Call parent constructor
        super().__init__(name, environment)
        self.ip = ip
  
    def deploy(self) -> bool:
        """Deploy server."""
        print(f"Deploying server {self.name} at {self.ip}")
        return True

class LoadBalancer(Infrastructure):
    """Load balancer inherits from Infrastructure."""
  
    def __init__(self, name: str, environment: str, backend_servers: list):
        super().__init__(name, environment)
        self.backend_servers = backend_servers
  
    def deploy(self) -> bool:
        """Deploy load balancer."""
        print(f"Deploying load balancer {self.name}")
        for server in self.backend_servers:
            print(f"  Adding backend: {server}")
        return True

# Usage - polymorphism
components: list[Infrastructure] = [
    Server('web-01', 'prod', '192.168.1.10'),
    LoadBalancer('lb-01', 'prod', ['web-01', 'web-02'])
]

for component in components:
    component.add_tag('team', 'platform')
    component.deploy()  # Calls appropriate deploy() method
```

#### Multiple Inheritance and MRO

Python supports multiple inheritance but use it carefully:

```python
# Method Resolution Order (MRO) determines method lookup

class Loggable:
    """Mixin for logging functionality."""
  
    def log(self, message: str) -> None:
        """Log message."""
        print(f"[{self.__class__.__name__}] {message}")

class Monitorable:
    """Mixin for monitoring functionality."""
  
    def send_metrics(self, metrics: dict) -> None:
        """Send metrics."""
        print(f"Metrics: {metrics}")

class Deployable:
    """Mixin for deployment functionality."""
  
    def deploy(self) -> bool:
        """Deploy resource."""
        raise NotImplementedError

class Application(Loggable, Monitorable, Deployable):
    """Application with multiple capabilities."""
  
    def __init__(self, name: str):
        self.name = name
  
    def deploy(self) -> bool:
        """Deploy application."""
        self.log(f"Deploying {self.name}")
        self.send_metrics({'deployment': 'started'})
        # Deployment logic
        self.log(f"{self.name} deployed successfully")
        return True

app = Application('api-service')
app.deploy()

# Check Method Resolution Order
print(Application.__mro__)
# (<class 'Application'>, <class 'Loggable'>, <class 'Monitorable'>, 
#  <class 'Deployable'>, <class 'object'>)
```

**Diamond Problem and super():**

```python
class Base:
    def __init__(self):
        print("Base.__init__")
        self.base_value = 1

class Left(Base):
    def __init__(self):
        print("Left.__init__")
        super().__init__()  # Calls next in MRO, not Base!
        self.left_value = 2

class Right(Base):
    def __init__(self):
        print("Right.__init__")
        super().__init__()  # Calls next in MRO
        self.right_value = 3

class Diamond(Left, Right):
    def __init__(self):
        print("Diamond.__init__")
        super().__init__()  # Calls Left.__init__
        self.diamond_value = 4

# Creating instance
d = Diamond()
# Output:
# Diamond.__init__
# Left.__init__
# Right.__init__
# Base.__init__

# Base.__init__ called ONCE (correct behavior)
# super() follows MRO, not parent class directly

print(Diamond.__mro__)
# (Diamond, Left, Right, Base, object)
```

#### Composition Over Inheritance

**Prefer composition when:**

- "Has-a" relationship (not "is-a")
- Behavior changes at runtime
- Avoiding deep inheritance hierarchies

```python
# ❌ BAD: Inheritance for code reuse
class ServerWithLogging(Server):
    """Server that logs everything."""
  
    def check_health(self):
        log("Checking health")
        return super().check_health()

# ✅ GOOD: Composition
class Logger:
    """Logging functionality."""
  
    def log(self, message: str) -> None:
        """Log message with timestamp."""
        import time
        timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
        print(f"[{timestamp}] {message}")

class Server:
    """Server with composable logging."""
  
    def __init__(self, hostname: str, logger: Logger = None):
        self.hostname = hostname
        self.logger = logger or Logger()
  
    def check_health(self) -> bool:
        """Check health with logging."""
        self.logger.log(f"Checking health of {self.hostname}")
        # Health check logic
        return True

# Easy to change logger implementation
class CloudWatchLogger(Logger):
    """Logger that sends to CloudWatch."""
  
    def log(self, message: str) -> None:
        send_to_cloudwatch(message)

# Use different logger without changing Server class
server = Server('web-01', CloudWatchLogger())
```

**DevSecOps Example - Deployment Pipeline:**

```python
from typing import Protocol

# Define protocols (interfaces) instead of inheritance
class DeploymentStrategy(Protocol):
    """Protocol for deployment strategies."""
    def deploy(self, version: str) -> bool: ...
    def rollback(self) -> bool: ...

class BlueGreenDeployment:
    """Blue-green deployment strategy."""
  
    def deploy(self, version: str) -> bool:
        print(f"Blue-green deploying {version}")
        return True
  
    def rollback(self) -> bool:
        print("Rolling back blue-green")
        return True

class CanaryDeployment:
    """Canary deployment strategy."""
  
    def deploy(self, version: str) -> bool:
        print(f"Canary deploying {version}")
        return True
  
    def rollback(self) -> bool:
        print("Rolling back canary")
        return True

class Application:
    """Application with pluggable deployment strategy."""
  
    def __init__(self, name: str, strategy: DeploymentStrategy):
        self.name = name
        self.strategy = strategy  # Composition!
  
    def deploy(self, version: str) -> bool:
        """Deploy using configured strategy."""
        try:
            return self.strategy.deploy(version)
        except Exception:
            return self.strategy.rollback()

# Use different strategies without changing Application
app = Application('api', BlueGreenDeployment())
app.deploy('1.0.0')

# Switch strategy at runtime
app.strategy = CanaryDeployment()
app.deploy('1.1.0')
```

### Abstract Base Classes (ABC)

Enforce interface contracts:

```python
from abc import ABC, abstractmethod

class InfrastructureProvider(ABC):
    """Abstract base class for cloud providers."""
  
    @abstractmethod
    def create_instance(self, config: dict) -> str:
        """Create compute instance."""
        pass
  
    @abstractmethod
    def delete_instance(self, instance_id: str) -> bool:
        """Delete compute instance."""
        pass
  
    @abstractmethod
    def get_instance_status(self, instance_id: str) -> str:
        """Get instance status."""
        pass
  
    # Concrete method (optional to override)
    def list_instances(self) -> list[str]:
        """List all instances."""
        return []

class AWSProvider(InfrastructureProvider):
    """AWS implementation."""
  
    def __init__(self, region: str):
        self.region = region
        self.ec2_client = boto3.client('ec2', region_name=region)
  
    def create_instance(self, config: dict) -> str:
        """Create EC2 instance."""
        response = self.ec2_client.run_instances(
            ImageId=config['ami_id'],
            InstanceType=config['instance_type'],
            MinCount=1,
            MaxCount=1
        )
        return response['Instances'][0]['InstanceId']
  
    def delete_instance(self, instance_id: str) -> bool:
        """Terminate EC2 instance."""
        self.ec2_client.terminate_instances(InstanceIds=[instance_id])
        return True
  
    def get_instance_status(self, instance_id: str) -> str:
        """Get EC2 instance status."""
        response = self.ec2_client.describe_instance_status(
            InstanceIds=[instance_id]
        )
        return response['InstanceStatuses'][0]['InstanceState']['Name']

# ❌ This would fail - missing abstract methods
# class IncompleteProvider(InfrastructureProvider):
#     def create_instance(self, config):
#         pass
#     # Missing delete_instance and get_instance_status!

# Can't instantiate abstract class
# provider = InfrastructureProvider()  # TypeError!

# Can instantiate concrete implementation
aws = AWSProvider('us-east-1')
```

### Magic Methods (Dunder Methods)

Special methods that implement Python protocols:

#### String Representation

```python
class Deployment:
    """Represents a deployment."""
  
    def __init__(self, app: str, version: str, status: str):
        self.app = app
        self.version = version
        self.status = status
  
    def __repr__(self) -> str:
        """Unambiguous representation for debugging."""
        return f"Deployment(app={self.app!r}, version={self.version!r}, status={self.status!r})"
  
    def __str__(self) -> str:
        """Human-readable representation."""
        return f"{self.app} v{self.version} - {self.status}"

deploy = Deployment('api', '1.0.0', 'running')
print(deploy)  # api v1.0.0 - running (uses __str__)
print(repr(deploy))  # Deployment(app='api', version='1.0.0', status='running')

# In debugger or interactive shell, __repr__ is shown
[deploy]  # [Deployment(app='api', version='1.0.0', status='running')]
```

#### Comparison Methods

```python
class Version:
    """Semantic version comparison."""
  
    def __init__(self, version: str):
        self.version = version
        parts = version.split('.')
        self.major = int(parts[0])
        self.minor = int(parts[1]) if len(parts) > 1 else 0
        self.patch = int(parts[2]) if len(parts) > 2 else 0
  
    def __eq__(self, other) -> bool:
        """Equal comparison."""
        return (self.major, self.minor, self.patch) == \
               (other.major, other.minor, other.patch)
  
    def __lt__(self, other) -> bool:
        """Less than comparison."""
        return (self.major, self.minor, self.patch) < \
               (other.major, other.minor, other.patch)
  
    def __le__(self, other) -> bool:
        """Less than or equal."""
        return self < other or self == other
  
    def __gt__(self, other) -> bool:
        """Greater than."""
        return not self <= other
  
    def __ge__(self, other) -> bool:
        """Greater than or equal."""
        return not self < other
  
    def __repr__(self) -> str:
        return f"Version('{self.version}')"

# Usage
v1 = Version('1.0.0')
v2 = Version('1.1.0')
v3 = Version('2.0.0')

print(v1 < v2)  # True
print(v2 < v3)  # True
print(v1 == Version('1.0.0'))  # True

# Works with sorted()
versions = [v3, v1, v2]
sorted_versions = sorted(versions)
print(sorted_versions)  # [Version('1.0.0'), Version('1.1.0'), Version('2.0.0')]
```

#### Container Methods

```python
class ServerCluster:
    """Cluster of servers behaving like a collection."""
  
    def __init__(self):
        self.servers = {}
  
    def __getitem__(self, hostname: str) -> 'Server':
        """Get server by hostname - enables cluster[hostname]."""
        return self.servers[hostname]
  
    def __setitem__(self, hostname: str, server: 'Server') -> None:
        """Add server - enables cluster[hostname] = server."""
        self.servers[hostname] = server
  
    def __delitem__(self, hostname: str) -> None:
        """Delete server - enables del cluster[hostname]."""
        del self.servers[hostname]
  
    def __contains__(self, hostname: str) -> bool:
        """Check membership - enables hostname in cluster."""
        return hostname in self.servers
  
    def __len__(self) -> int:
        """Get count - enables len(cluster)."""
        return len(self.servers)
  
    def __iter__(self):
        """Make iterable - enables for server in cluster."""
        return iter(self.servers.values())

# Usage like a dict
cluster = ServerCluster()
cluster['web-01'] = Server('web-01', '192.168.1.10', 'prod')
cluster['web-02'] = Server('web-02', '192.168.1.11', 'prod')

# Check membership
if 'web-01' in cluster:
    print("web-01 is in cluster")

# Get length
print(f"Cluster has {len(cluster)} servers")

# Iterate
for server in cluster:
    server.check_health()

# Access by key
web_server = cluster['web-01']

# Delete
del cluster['web-02']
```

#### Context Manager Protocol

```python
class DatabaseConnection:
    """Database connection as context manager."""
  
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
        self.connection = None
  
    def __enter__(self):
        """Called when entering 'with' block."""
        print(f"Connecting to {self.host}:{self.port}")
        self.connection = create_connection(self.host, self.port)
        return self.connection
  
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Called when exiting 'with' block."""
        print("Closing connection")
        if self.connection:
            self.connection.close()
      
        # Return False to propagate exceptions
        # Return True to suppress exceptions
        return False

# Usage
with DatabaseConnection('localhost', 5432) as conn:
    conn.execute("SELECT * FROM users")
# Connection automatically closed

# Even if exception occurs, __exit__ is called
try:
    with DatabaseConnection('localhost', 5432) as conn:
        conn.execute("INVALID SQL")
except Exception:
    pass  # Connection still closed
```

**DevSecOps Context Manager:**

```python
import time
from contextlib import contextmanager

class DeploymentLock:
    """Prevent concurrent deployments with context manager."""
  
    def __init__(self, deployment_name: str):
        self.deployment_name = deployment_name
        self.lock_file = f"/var/lock/{deployment_name}.lock"
        self.locked = False
  
    def __enter__(self):
        """Acquire deployment lock."""
        if os.path.exists(self.lock_file):
            raise RuntimeError(
                f"Deployment {self.deployment_name} already in progress"
            )
      
        with open(self.lock_file, 'w') as f:
            f.write(f"Locked at {time.time()}")
      
        self.locked = True
        return self
  
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Release deployment lock."""
        if self.locked and os.path.exists(self.lock_file):
            os.remove(self.lock_file)
        return False

# Usage - prevents concurrent deployments
with DeploymentLock('api-service'):
    deploy_application('api-service', '1.0.0')
# Lock automatically released

# Alternative using @contextmanager decorator
@contextmanager
def deployment_lock(name: str):
    """Context manager using generator."""
    lock_file = f"/var/lock/{name}.lock"
  
    # Setup (before yield)
    if os.path.exists(lock_file):
        raise RuntimeError(f"Deployment {name} in progress")
  
    with open(lock_file, 'w') as f:
        f.write(str(time.time()))
  
    try:
        yield  # Control returns to with block
    finally:
        # Cleanup (after yield)
        if os.path.exists(lock_file):
            os.remove(lock_file)

# Usage is identical
with deployment_lock('api-service'):
    deploy_application('api-service', '1.0.0')
```

#### Callable Objects

```python
class RateLimiter:
    """Rate limiter as callable object."""
  
    def __init__(self, calls_per_minute: int):
        self.calls_per_minute = calls_per_minute
        self.calls = []
  
    def __call__(self) -> bool:
        """Make instance callable like a function."""
        import time
        now = time.time()
      
        # Remove calls outside 1-minute window
        self.calls = [t for t in self.calls if t > now - 60]
      
        # Check limit
        if len(self.calls) >= self.calls_per_minute:
            return False
      
        self.calls.append(now)
        return True

# Create rate limiter
limiter = RateLimiter(100)

# Call like a function
if limiter():  # Calls __call__
    process_request()
else:
    return_rate_limit_error()
```

### Properties and Descriptors

#### Properties

Properties provide getter/setter with validation:

```python
class Server:
    """Server with validated properties."""
  
    def __init__(self, hostname: str, port: int):
        self.hostname = hostname
        self._port = None
        self.port = port  # Uses property setter
  
    @property
    def port(self) -> int:
        """Get port."""
        return self._port
  
    @port.setter
    def port(self, value: int) -> None:
        """Set port with validation."""
        if not isinstance(value, int):
            raise TypeError("Port must be an integer")
        if not 1 <= value <= 65535:
            raise ValueError(f"Invalid port: {value}")
        self._port = value
  
    @port.deleter
    def port(self) -> None:
        """Delete port (reset to default)."""
        self._port = 22

# Usage
server = Server('web-01', 8080)
print(server.port)  # 8080 (uses getter)

server.port = 9000  # Uses setter with validation
# server.port = 99999  # ValueError!
# server.port = "80"  # TypeError!

del server.port  # Uses deleter (resets to 22)
```

**Read-only Properties:**

```python
class Deployment:
    """Deployment with computed properties."""
  
    def __init__(self, app: str, version: str):
        self.app = app
        self.version = version
        self._start_time = time.time()
  
    @property
    def uptime(self) -> float:
        """Read-only computed property."""
        return time.time() - self._start_time
  
    @property
    def age_minutes(self) -> int:
        """Another computed property."""
        return int(self.uptime / 60)

deploy = Deployment('api', '1.0.0')
time.sleep(2)
print(deploy.uptime)  # ~2.0
# deploy.uptime = 100  # AttributeError - no setter!
```

**Cached Properties:**

```python
from functools import cached_property

class ServerMetrics:
    """Server metrics with expensive computation."""
  
    def __init__(self, server_id: str):
        self.server_id = server_id
  
    @cached_property
    def cpu_stats(self) -> dict:
        """Compute CPU stats (expensive, cached)."""
        print("Computing CPU stats...")
        # Expensive computation
        time.sleep(1)
        return {'usage': 45.2, 'cores': 4}
  
    @cached_property
    def memory_stats(self) -> dict:
        """Compute memory stats (expensive, cached)."""
        print("Computing memory stats...")
        time.sleep(1)
        return {'used': 8192, 'total': 16384}

metrics = ServerMetrics('web-01')
print(metrics.cpu_stats)  # First call: computes and caches
print(metrics.cpu_stats)  # Second call: returns cached value
```

#### Descriptors

Descriptors provide fine-grained control over attribute access:

```python
class ValidatedString:
    """Descriptor for string validation."""
  
    def __init__(self, min_length: int = 0, max_length: int = 100):
        self.min_length = min_length
        self.max_length = max_length
  
    def __set_name__(self, owner, name):
        """Called when descriptor is assigned to class attribute."""
        self.name = name
        self.private_name = f"_{name}"
  
    def __get__(self, obj, objtype=None):
        """Get attribute value."""
        if obj is None:
            return self
        return getattr(obj, self.private_name, "")
  
    def __set__(self, obj, value):
        """Set attribute value with validation."""
        if not isinstance(value, str):
            raise TypeError(f"{self.name} must be a string")
      
        if len(value) < self.min_length:
            raise ValueError(
                f"{self.name} must be at least {self.min_length} characters"
            )
      
        if len(value) > self.max_length:
            raise ValueError(
                f"{self.name} must be at most {self.max_length} characters"
            )
      
        setattr(obj, self.private_name, value)

class User:
    """User with validated fields."""
  
    # Descriptors provide validation for multiple attributes
    username = ValidatedString(min_length=3, max_length=20)
    email = ValidatedString(min_length=5, max_length=100)
  
    def __init__(self, username: str, email: str):
        self.username = username  # Uses descriptor
        self.email = email        # Uses descriptor

# Usage
user = User('alice', 'alice@example.com')
user.username = 'bob'  # Validates
# user.username = 'ab'  # ValueError - too short!
# user.username = 123  # TypeError - not a string!
```

### Dataclasses (Modern Python)

Dataclasses reduce boilerplate for data-holding classes:

```python
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class Server:
    """Server configuration with automatic __init__, __repr__, __eq__."""
  
    hostname: str
    ip: str
    environment: str
    port: int = 22
    tags: dict = field(default_factory=dict)  # Mutable default
  
    def __post_init__(self):
        """Validation after initialization."""
        import ipaddress
        try:
            ipaddress.ip_address(self.ip)
        except ValueError:
            raise ValueError(f"Invalid IP address: {self.ip}")

# Automatic __init__
server = Server(
    hostname='web-01',
    ip='192.168.1.10',
    environment='prod'
)

# Automatic __repr__
print(server)
# Server(hostname='web-01', ip='192.168.1.10', environment='prod', port=22, tags={})

# Automatic __eq__
server2 = Server('web-01', '192.168.1.10', 'prod')
print(server == server2)  # True

# ❌ DANGER: Wrong default for mutable
@dataclass
class BadServer:
    tags: dict = {}  # SHARED across all instances!

# ✅ CORRECT: Use default_factory
@dataclass
class GoodServer:
    tags: dict = field(default_factory=dict)  # Each instance gets new dict
```

**Immutable Dataclasses:**

```python
@dataclass(frozen=True)
class Credentials:
    """Immutable credentials."""
    username: str
    password: str
    api_key: Optional[str] = None

creds = Credentials('admin', 'secret123')
# creds.password = 'new'  # FrozenInstanceError!
```

**Dataclass with Ordering:**

```python
@dataclass(order=True)
class Version:
    """Version with automatic ordering."""
    major: int
    minor: int
    patch: int

v1 = Version(1, 0, 0)
v2 = Version(1, 1, 0)
v3 = Version(2, 0, 0)

print(v1 < v2)  # True
print(sorted([v3, v1, v2]))  # [Version(1,0,0), Version(1,1,0), Version(2,0,0)]
```

### Frequently Asked Questions

**Q1: When should I use a class vs a function or dict?**

**A:** Use classes when you have:

1. **State + behavior together**: Server with health checks
2. **Multiple instances with shared behavior**: Many servers, one health check method
3. **Inheritance/polymorphism needs**: Different deployment strategies
4. **Complex initialization logic**: Validation, setup, connections

Use functions when:

1. **Stateless transformations**: `parse_log_line(line)`
2. **Simple utilities**: `validate_ip(address)`

Use dicts when:

1. **Pure data**: Configuration, JSON responses
2. **No behavior needed**: Just passing data around

```python
# ❌ BAD: Class for simple data
class ConfigData:
    def __init__(self, host, port):
        self.host = host
        self.port = port

# ✅ GOOD: Just use a dict or dataclass
config = {'host': 'localhost', 'port': 8080}

# OR for type safety:
@dataclass
class Config:
    host: str
    port: int

# ✅ GOOD: Class for state + behavior
class DatabaseConnection:
    def __init__(self, config):
        self.config = config
        self.connection = None
  
    def connect(self):
        """Establish connection."""
        pass
  
    def execute(self, query):
        """Execute query."""
        pass
```

**Q2: What's the difference between `@classmethod` and `@staticmethod`?**

**A:**

**`@classmethod`:**

- Receives class as first parameter (`cls`)
- Can access/modify class state
- Used for alternative constructors, factory methods

**`@staticmethod`:**

- No automatic first parameter
- Can't access class or instance state
- Just namespaced function

```python
class Config:
    _default_timeout = 30
  
    @classmethod
    def from_file(cls, filepath: str):
        """Alternative constructor - needs cls to create instance."""
        with open(filepath) as f:
            data = json.load(f)
        return cls(**data)  # Can create instance of cls
  
    @classmethod
    def set_default_timeout(cls, timeout: int):
        """Modify class state - needs cls."""
        cls._default_timeout = timeout
  
    @staticmethod
    def validate_timeout(timeout: int) -> bool:
        """Pure utility - doesn't need cls or self."""
        return 0 < timeout < 3600

# Usage
config = Config.from_file('config.json')  # classmethod
Config.set_default_timeout(60)  # classmethod
is_valid = Config.validate_timeout(30)  # staticmethod
```

**Q3: Should I use inheritance or composition?**

**A:** **Prefer composition** unless there's a clear "is-a" relationship:

**Use Inheritance When:**

- Clear "is-a" relationship: `Dog is-a Animal`
- Shared interface across subtypes
- Leveraging polymorphism

**Use Composition When:**

- "Has-a" relationship: `Server has-a Logger`
- Behavior changes at runtime
- Avoiding deep hierarchies
- Multiple behaviors to combine

```python
# ❌ BAD: Inheritance for code reuse
class LoggingServer(Server):
    """Server with logging."""
    pass

class MonitoringServer(Server):
    """Server with monitoring."""
    pass

class LoggingMonitoringServer(LoggingServer, MonitoringServer):
    """Server with both - getting messy!"""
    pass

# ✅ GOOD: Composition
class Server:
    def __init__(self, logger=None, monitor=None):
        self.logger = logger or Logger()
        self.monitor = monitor or Monitor()
  
    def check_health(self):
        self.logger.log("Checking health")
        result = perform_health_check()
        self.monitor.record_metric('health_check', result)
        return result

# Easy to mix and match
server1 = Server(CloudWatchLogger(), DatadogMonitor())
server2 = Server(FileLogger(), PrometheusMonitor())
```

**Q4: When should I use `__slots__`?**

**A:** Use `__slots__` when:

1. Creating **many instances** (thousands/millions)
2. Memory is constrained
3. You don't need dynamic attributes

**`__slots__` benefits:**

- Lower memory usage (~40% savings)
- Faster attribute access
- Prevents accidental attribute creation

```python
# Normal class
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# With __slots__
class OptimizedPoint:
    __slots__ = ['x', 'y']  # Only these attributes allowed
  
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Memory comparison
import sys
p1 = Point(1, 2)
p2 = OptimizedPoint(1, 2)

print(sys.getsizeof(p1.__dict__))  # ~240 bytes
print(sys.getsizeof(p2))  # ~64 bytes

# Prevents dynamic attributes
p1.z = 3  # Works
# p2.z = 3  # AttributeError!

# Use case: many instances
points = [OptimizedPoint(i, i) for i in range(1_000_000)]
# Saves ~176 MB vs normal class
```

**Q5: What's the difference between `__str__` and `__repr__`?**

**A:**

**`__str__`**: User-friendly, readable representation
**`__repr__`**: Developer-friendly, unambiguous representation (should be valid Python if possible)

```python
class Deployment:
    def __init__(self, app, version):
        self.app = app
        self.version = version
  
    def __str__(self):
        """For end users."""
        return f"{self.app} v{self.version}"
  
    def __repr__(self):
        """For developers/debugging."""
        return f"Deployment(app={self.app!r}, version={self.version!r})"

deploy = Deployment('api', '1.0.0')

print(str(deploy))   # api v1.0.0
print(repr(deploy))  # Deployment(app='api', version='1.0.0')

# In containers
[deploy]  # [Deployment(app='api', version='1.0.0')] - uses __repr__

# str() falls back to __repr__ if __str__ not defined
# Always implement at least __repr__!
```

### Interview Questions

**Question 1: Explain the difference between class attributes and instance attributes. What's a common pitfall with mutable class attributes?**

**Difficulty:** Junior

**Answer:**

**Class attributes**: Shared across all instances, defined directly in class body
**Instance attributes**: Unique to each instance, defined in `__init__`

```python
class Server:
    # Class attribute - shared by ALL instances
    default_port = 22
    connection_count = 0  # Shared counter
  
    def __init__(self, hostname):
        # Instance attributes - unique to each instance
        self.hostname = hostname
        Server.connection_count += 1

server1 = Server('web-01')
server2 = Server('web-02')

# Class attribute accessed via class or instance
print(Server.default_port)  # 22
print(server1.default_port)  # 22
print(server2.default_port)  # 22

# Modifying via class affects all instances
Server.default_port = 443
print(server1.default_port)  # 443

# BUT modifying via instance creates NEW instance attribute!
server1.default_port = 8080  # Creates instance attribute
print(server1.default_port)  # 8080
print(server2.default_port)  # 443 (still has class attribute)
print(Server.default_port)  # 443 (class attribute unchanged)
```

**Critical Pitfall - Mutable Class Attributes:**

```python
# ❌ DANGER: Mutable class attribute
class BadServer:
    tags = []  # SHARED list across ALL instances!
  
    def __init__(self, hostname):
        self.hostname = hostname
  
    def add_tag(self, tag):
        self.tags.append(tag)  # Modifies SHARED list!

server1 = BadServer('web-01')
server1.add_tag('production')

server2 = BadServer('web-02')
server2.add_tag('staging')

print(server1.tags)  # ['production', 'staging'] - UNEXPECTED!
print(server2.tags)  # ['production', 'staging'] - Same list!

# ✅ CORRECT: Instance attribute
class GoodServer:
    def __init__(self, hostname):
        self.hostname = hostname
        self.tags = []  # Each instance gets own list
  
    def add_tag(self, tag):
        self.tags.append(tag)

server1 = GoodServer('web-01')
server1.add_tag('production')

server2 = GoodServer('web-02')
server2.add_tag('staging')

print(server1.tags)  # ['production'] - Correct!
print(server2.tags)  # ['staging'] - Separate list!
```

**DevSecOps Impact:**
This bug causes data leakage between instances - a security risk if tags contain sensitive information.

**Follow-up:** How would you intentionally share state across all instances (e.g., a connection pool)?

---

**Question 2: What is the Method Resolution Order (MRO) and why does it matter? Explain with an example involving multiple inheritance.**

**Difficulty:** Mid-Level

**Answer:**

**Method Resolution Order (MRO)** determines the order in which Python searches for methods in a class hierarchy. It uses the **C3 Linearization** algorithm.

**Why it matters:**

- Prevents ambiguity in multiple inheritance
- Determines which method gets called
- Critical for understanding `super()`

**Example:**

```python
class Base:
    def greet(self):
        return "Base"

class Left(Base):
    def greet(self):
        return "Left -> " + super().greet()

class Right(Base):
    def greet(self):
        return "Right -> " + super().greet()

class Diamond(Left, Right):
    def greet(self):
        return "Diamond -> " + super().greet()

# Check MRO
print(Diamond.__mro__)
# (<class '__main__.Diamond'>, <class '__main__.Left'>, 
#  <class '__main__.Right'>, <class '__main__.Base'>, <class 'object'>)

d = Diamond()
print(d.greet())
# Diamond -> Left -> Right -> Base

# How it works:
# 1. Diamond.greet() calls super().greet()
#    → Next in MRO is Left
# 2. Left.greet() calls super().greet()
#    → Next in MRO is Right (NOT Base!)
# 3. Right.greet() calls super().greet()
#    → Next in MRO is Base
# 4. Base.greet() returns "Base"
```

**DevSecOps Example - Deployment with Multiple Capabilities:**

```python
class Deployable:
    def deploy(self):
        print("Deployable: Starting deployment")
        return True

class Loggable:
    def deploy(self):
        print("Loggable: Logging deployment")
        super().deploy()  # Continue MRO chain
        print("Loggable: Logged successfully")
        return True

class Monitorable:
    def deploy(self):
        print("Monitorable: Recording metrics")
        result = super().deploy()  # Continue MRO chain
        print("Monitorable: Metrics recorded")
        return result

class Application(Loggable, Monitorable, Deployable):
    def deploy(self):
        print("Application: Preparing deployment")
        result = super().deploy()  # Starts MRO chain
        print("Application: Deployment complete")
        return result

# Check MRO
print(Application.__mro__)
# (Application, Loggable, Monitorable, Deployable, object)

app = Application()
app.deploy()

# Output shows MRO in action:
# Application: Preparing deployment
# Loggable: Logging deployment
# Monitorable: Recording metrics
# Deployable: Starting deployment
# Monitorable: Metrics recorded
# Loggable: Logged successfully
# Application: Deployment complete
```

**Key Rules:**

1. Child classes come before parents
2. Parent order matches inheritance order: `class C(A, B)` → A before B
3. Parents come before grandparents
4. Diamond problem solved: base class appears only once at end

**Without `super()` (wrong):**

```python
class Wrong(Left, Right):
    def greet(self):
        return Left.greet(self) + Right.greet(self)
        # Calls Base.greet() twice! Not following MRO
```

**Why This Matters in DevSecOps:**

- Ensures all mixins execute in correct order
- Logging, monitoring, security checks happen in predictable sequence
- Prevents duplicate operations (e.g., acquiring same lock twice)

**Follow-up:** What happens if you create a class with inconsistent MRO (e.g., circular dependencies)?

---

**Question 3: Explain descriptors in Python. Implement a descriptor that validates and logs all attribute access for security auditing.**

**Difficulty:** Senior

**Answer:**

**Descriptors** are objects that implement the descriptor protocol (`__get__`, `__set__`, `__delete__`) and control attribute access on other classes.

**Descriptor Protocol:**

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        """Called when attribute is accessed."""
        pass
  
    def __set__(self, obj, value):
        """Called when attribute is set."""
        pass
  
    def __delete__(self, obj):
        """Called when attribute is deleted."""
        pass
```

**Security Auditing Descriptor:**

```python
import logging
from datetime import datetime
from typing import Any, Optional

logger = logging.getLogger(__name__)

class AuditedAttribute:
    """Descriptor that logs all attribute access for security auditing."""
  
    def __init__(self, attr_type: type, sensitive: bool = False):
        self.attr_type = attr_type
        self.sensitive = sensitive
  
    def __set_name__(self, owner, name):
        """Called when descriptor is assigned to class attribute."""
        self.name = name
        self.private_name = f"_{name}"
  
    def __get__(self, obj, objtype=None):
        """Log and return attribute value."""
        if obj is None:
            return self
      
        value = getattr(obj, self.private_name, None)
      
        # Audit log for access
        if self.sensitive:
            logger.warning(
                f"AUDIT: User {get_current_user()} accessed sensitive "
                f"attribute '{self.name}' on {obj.__class__.__name__} "
                f"at {datetime.now()}"
            )
        else:
            logger.info(
                f"AUDIT: Attribute '{self.name}' accessed on "
                f"{obj.__class__.__name__}"
            )
      
        return value
  
    def __set__(self, obj, value):
        """Validate, log, and set attribute value."""
        # Type validation
        if not isinstance(value, self.attr_type):
            raise TypeError(
                f"{self.name} must be {self.attr_type.__name__}, "
                f"got {type(value).__name__}"
            )
      
        # Get old value for audit trail
        old_value = getattr(obj, self.private_name, None)
      
        # Audit log for modification
        if self.sensitive:
            logger.warning(
                f"AUDIT: User {get_current_user()} modified sensitive "
                f"attribute '{self.name}' on {obj.__class__.__name__} "
                f"from {mask_value(old_value)} to {mask_value(value)} "
                f"at {datetime.now()}"
            )
        else:
            logger.info(
                f"AUDIT: Attribute '{self.name}' modified from "
                f"{old_value!r} to {value!r}"
            )
      
        # Set the value
        setattr(obj, self.private_name, value)
  
    def __delete__(self, obj):
        """Log and delete attribute."""
        logger.warning(
            f"AUDIT: User {get_current_user()} deleted "
            f"attribute '{self.name}' on {obj.__class__.__name__} "
            f"at {datetime.now()}"
        )
        delattr(obj, self.private_name)

def mask_value(value: Any) -> str:
    """Mask sensitive values in logs."""
    if value is None:
        return "None"
    str_val = str(value)
    if len(str_val) <= 4:
        return "***"
    return str_val[:2] + "***" + str_val[-2:]

def get_current_user() -> str:
    """Get current user for audit trail."""
    import getpass
    return getpass.getuser()

# Usage
class UserAccount:
    """User account with audited attributes."""
  
    username = AuditedAttribute(str, sensitive=False)
    email = AuditedAttribute(str, sensitive=True)
    api_key = AuditedAttribute(str, sensitive=True)
  
    def __init__(self, username: str, email: str, api_key: str):
        self.username = username
        self.email = email
        self.api_key = api_key

# Create user
user = UserAccount('alice', 'alice@example.com', 'sk_live_abc123xyz')
# AUDIT: Attribute 'username' modified from None to 'alice'
# AUDIT: User alice accessed sensitive attribute 'email' ...
# AUDIT: User alice modified sensitive attribute 'email' from None to al***om

# Access attributes (logged)
print(user.username)  # AUDIT: Attribute 'username' accessed
print(user.email)     # AUDIT: User accessed sensitive attribute 'email'

# Modify attributes (logged)
user.api_key = 'sk_live_new_key'
# AUDIT: User modified sensitive attribute 'api_key' from sk***23 to sk***ey

# Type validation
try:
    user.username = 123  # TypeError: username must be str, got int
except TypeError as e:
    print(e)

# Delete (logged)
del user.api_key
# AUDIT: User deleted attribute 'api_key'
```

**How Descriptors Work:**

1. **Descriptor lookup**: When you access `obj.attr`, Python checks if `attr` is a descriptor
2. **Priority order**:
   - Data descriptors (have `__set__` or `__delete__`) override instance dict
   - Instance dict (if no data descriptor)
   - Non-data descriptors (only `__get__`)
   - Class dict

**Another Example - Typed Attribute:**

```python
class TypedAttribute:
    """Descriptor with type checking."""
  
    def __init__(self, name: str, expected_type: type):
        self.name = name
        self.expected_type = expected_type
  
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)
  
    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name} must be {self.expected_type}, "
                f"got {type(value)}"
            )
        obj.__dict__[self.name] = value

class Server:
    hostname = TypedAttribute('hostname', str)
    port = TypedAttribute('port', int)
  
    def __init__(self, hostname: str, port: int):
        self.hostname = hostname
        self.port = port

server = Server('web-01', 8080)
# server.port = "80"  # TypeError!
```

**Why Descriptors Matter in DevSecOps:**

- **Security**: Audit trail for sensitive attribute access
- **Validation**: Type checking, range validation
- **Computed properties**: Lazy evaluation, caching
- **ORM implementations**: SQLAlchemy, Django ORM use descriptors

**Built-in Descriptors:**

- `property()`: Implements descriptor protocol
- `classmethod()`: Descriptor
- `staticmethod()`: Descriptor
- `__slots__`: Uses descriptors

**Follow-up Questions:**

- What's the difference between data and non-data descriptors?
- How would you implement a cached property using descriptors?
- Explain how `property()` is implemented using descriptors.

---

## 1.5 Error Handling and Exceptions

Proper error handling is critical in DevSecOps. Poor error handling causes production outages, security vulnerabilities, and data loss.

### Exception Hierarchy

Python's exception hierarchy:

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── FloatingPointError
    │   ├── OverflowError
    │   └── ZeroDivisionError
    ├── AssertionError
    ├── AttributeError
    ├── EOFError
    ├── ImportError
    │   └── ModuleNotFoundError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── MemoryError
    ├── NameError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   └── TimeoutError
    ├── RuntimeError
    │   └── RecursionError
    ├── TypeError
    ├── ValueError
    └── ...
```

**Key Rules:**

1. **Catch `Exception`, not `BaseException`** - Don't catch `KeyboardInterrupt`, `SystemExit`
2. **Specific exceptions before general** - Order matters
3. **Never use bare `except:`** - Catches everything, including `KeyboardInterrupt`

### Basic Exception Handling

```python
# ❌ BAD: Bare except catches everything
try:
    deploy_application()
except:  # Catches KeyboardInterrupt, SystemExit, etc.!
    log_error()

# ✅ GOOD: Catch specific exceptions
try:
    deploy_application()
except DeploymentError as e:
    log_error(e)
    send_alert(e)
except TimeoutError as e:
    log_timeout(e)
    retry_deployment()
except Exception as e:
    log_unexpected_error(e)
    raise  # Re-raise unexpected errors

# Multiple exceptions in one handler
try:
    connect_to_server()
except (ConnectionError, TimeoutError) as e:
    handle_network_error(e)

# try/except/else/finally
try:
    result = risky_operation()
except OperationError as e:
    handle_error(e)
else:
    # Executes ONLY if no exception
    log_success()
    process_result(result)
finally:
    # ALWAYS executes (cleanup)
    close_connections()
    release_locks()
```

### Custom Exceptions

Create domain-specific exceptions for clarity:

```python
class DeploymentError(Exception):
    """Base exception for deployment errors."""
    pass

class HealthCheckFailed(DeploymentError):
    """Raised when health check fails."""
    pass

class RollbackFailed(DeploymentError):
    """Raised when rollback fails (critical!)."""
    pass

class InsufficientResources(DeploymentError):
    """Raised when resources unavailable."""
  
    def __init__(self, resource: str, required: int, available: int):
        self.resource = resource
        self.required = required
        self.available = available
        super().__init__(
            f"Insufficient {resource}: need {required}, have {available}"
        )

# Usage
def deploy_application(app: str, replicas: int):
    """Deploy application with resource checking."""
    available_nodes = count_available_nodes()
  
    if available_nodes < replicas:
        raise InsufficientResources('nodes', replicas, available_nodes)
  
    # Deploy...

# Handling
try:
    deploy_application('api', replicas=10)
except InsufficientResources as e:
    logger.error(f"Cannot deploy: {e}")
    logger.error(f"Need {e.required} {e.resource}, have {e.available}")
    notify_ops_team(e)
```

### Exception Chaining

Preserve exception context:

```python
# ❌ BAD: Loses original exception
try:
    response = api.get('/users')
except RequestException:
    raise DeploymentError("API call failed")  # Original error lost!

# ✅ GOOD: Chain exceptions
try:
    response = api.get('/users')
except RequestException as e:
    raise DeploymentError("API call failed") from e
    # Original exception preserved in __cause__

# ✅ ALSO GOOD: Implicit chaining
try:
    response = api.get('/users')
except RequestException:
    # Processing that raises another exception
    raise DeploymentError("API call failed")
    # Python auto-sets __context__ to RequestException
```

**Accessing chained exceptions:**

```python
try:
    deploy()
except DeploymentError as e:
    logger.error(f"Deployment failed: {e}")
  
    # Access original exception
    if e.__cause__:
        logger.error(f"Caused by: {e.__cause__}")
  
    # Full traceback includes chain
    import traceback
    traceback.print_exc()
```

### EAFP vs LBYL

**EAFP (Easier to Ask for Forgiveness than Permission)** - Python philosophy
**LBYL (Look Before You Leap)** - Check before action

```python
# LBYL - Check first
if 'key' in dictionary:
    value = dictionary['key']
else:
    value = default

# EAFP - Try and handle
try:
    value = dictionary['key']
except KeyError:
    value = default

# Or better:
value = dictionary.get('key', default)

# ❌ LBYL: Race condition
if os.path.exists(file_path):
    with open(file_path) as f:  # File might be deleted here!
        data = f.read()

# ✅ EAFP: Atomic, no race condition
try:
    with open(file_path) as f:
        data = f.read()
except FileNotFoundError:
    data = None
```

**When to use LBYL:**

- Input validation before expensive operations
- User-facing validation messages
- Configuration validation

**When to use EAFP:**

- File operations (race conditions)
- Dictionary/list access
- Attribute access
- Most production code

### Context Managers for Resource Management

Context managers ensure cleanup:

```python
# Manual resource management (error-prone)
connection = create_connection()
try:
    connection.execute(query)
finally:
    connection.close()  # Might forget this!

# ✅ Context manager guarantees cleanup
with create_connection() as connection:
    connection.execute(query)
# connection.close() called automatically

# Multiple context managers
with open('input.txt') as infile, open('output.txt', 'w') as outfile:
    outfile.write(infile.read())

# Reusable context manager with contextlib
from contextlib import contextmanager

@contextmanager
def deployment_lock(service_name: str):
    """Ensure only one deployment at a time."""
    lock_file = f"/var/lock/{service_name}.lock"
  
    # Setup
    if os.path.exists(lock_file):
        raise RuntimeError(f"Deployment of {service_name} in progress")
  
    with open(lock_file, 'w') as f:
        f.write(str(os.getpid()))
  
    try:
        yield  # Control returns to with block
    finally:
        # Cleanup - even if exception raised
        if os.path.exists(lock_file):
            os.remove(lock_file)

# Usage
with deployment_lock('api-service'):
    deploy_application('api-service')
# Lock released even if deployment fails
```

### Error Handling Patterns

#### Retry with Exponential Backoff

```python
import time
import random
from typing import Callable, TypeVar, Any

T = TypeVar('T')

def retry_with_backoff(
    func: Callable[..., T],
    max_attempts: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0,
    jitter: bool = True,
    exceptions: tuple = (Exception,)
) -> T:
    """
    Retry function with exponential backoff.
  
    Args:
        func: Function to retry
        max_attempts: Maximum retry attempts
        base_delay: Initial delay in seconds
        max_delay: Maximum delay in seconds
        exponential_base: Base for exponential backoff
        jitter: Add random jitter to prevent thundering herd
        exceptions: Tuple of exceptions to catch and retry
  
    Returns:
        Function result
  
    Raises:
        Last exception if all attempts fail
    """
    last_exception = None
  
    for attempt in range(max_attempts):
        try:
            return func()
        except exceptions as e:
            last_exception = e
          
            if attempt == max_attempts - 1:
                # Last attempt, don't sleep
                break
          
            # Calculate delay with exponential backoff
            delay = min(base_delay * (exponential_base ** attempt), max_delay)
          
            # Add jitter: random value between 0 and delay
            if jitter:
                delay = random.uniform(0, delay)
          
            logger.warning(
                f"Attempt {attempt + 1}/{max_attempts} failed: {e}. "
                f"Retrying in {delay:.2f}s..."
            )
          
            time.sleep(delay)
  
    # All attempts failed
    raise last_exception

# Usage
def unstable_api_call():
    """API call that might fail temporarily."""
    response = requests.get('https://api.example.com/data')
    response.raise_for_status()
    return response.json()

try:
    data = retry_with_backoff(
        unstable_api_call,
        max_attempts=5,
        base_delay=1.0,
        exceptions=(requests.RequestException,)
    )
except requests.RequestException as e:
    logger.error(f"API call failed after retries: {e}")
    raise
```

#### Circuit Breaker Pattern

```python
from enum import Enum
import time
from typing import Callable

class CircuitState(Enum):
    """Circuit breaker states."""
    CLOSED = 'closed'  # Normal operation
    OPEN = 'open'      # Failing, reject requests
    HALF_OPEN = 'half_open'  # Testing if service recovered

class CircuitBreaker:
    """
    Circuit breaker prevents cascading failures.
  
    States:
    - CLOSED: Normal operation, requests pass through
    - OPEN: Too many failures, requests fail immediately
    - HALF_OPEN: Testing recovery, limited requests allowed
    """
  
    def __init__(
        self,
        failure_threshold: int = 5,
        success_threshold: int = 2,
        timeout: float = 60.0
    ):
        self.failure_threshold = failure_threshold
        self.success_threshold = success_threshold
        self.timeout = timeout
      
        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
  
    def call(self, func: Callable, *args, **kwargs):
        """Execute function through circuit breaker."""
      
        if self.state == CircuitState.OPEN:
            # Check if timeout expired
            if time.time() - self.last_failure_time >= self.timeout:
                logger.info("Circuit breaker entering HALF_OPEN state")
                self.state = CircuitState.HALF_OPEN
                self.success_count = 0
            else:
                # Still open, fail fast
                raise RuntimeError("Circuit breaker is OPEN")
      
        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise
  
    def _on_success(self):
        """Handle successful call."""
        self.failure_count = 0
      
        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1
            if self.success_count >= self.success_threshold:
                logger.info("Circuit breaker entering CLOSED state")
                self.state = CircuitState.CLOSED
                self.success_count = 0
  
    def _on_failure(self):
        """Handle failed call."""
        self.failure_count += 1
        self.last_failure_time = time.time()
      
        if self.failure_count >= self.failure_threshold:
            logger.warning(
                f"Circuit breaker opening after {self.failure_count} failures"
            )
            self.state = CircuitState.OPEN

# Usage
api_circuit_breaker = CircuitBreaker(
    failure_threshold=5,
    success_threshold=2,
    timeout=60.0
)

def call_external_api():
    """Call external API through circuit breaker."""
    try:
        return api_circuit_breaker.call(
            requests.get,
            'https://api.example.com/data',
            timeout=5
        )
    except RuntimeError as e:
        # Circuit breaker open
        logger.error(f"Circuit breaker prevented call: {e}")
        return get_cached_response()
    except Exception as e:
        logger.error(f"API call failed: {e}")
        return get_cached_response()
```

### Production Error Handling

#### Structured Logging

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """Format logs as JSON for structured logging."""
  
    def format(self, record: logging.LogRecord) -> str:
        """Format log record as JSON."""
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno,
        }
      
        # Add exception info if present
        if record.exc_info:
            log_data['exception'] = {
                'type': record.exc_info[0].__name__,
                'message': str(record.exc_info[1]),
                'traceback': self.formatException(record.exc_info)
            }
      
        # Add extra fields
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        if hasattr(record, 'request_id'):
            log_data['request_id'] = record.request_id
      
        return json.dumps(log_data)

# Configure logging
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger = logging.getLogger('app')
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Usage
try:
    deploy_application('api', '1.0.0')
except DeploymentError as e:
    logger.error(
        "Deployment failed",
        exc_info=True,
        extra={'user_id': 'admin', 'request_id': 'req-123'}
    )
```

#### Never Expose Sensitive Data in Errors

```python
# ❌ CRITICAL SECURITY FLAW: Exposing credentials
try:
    connect_to_database(host, username, password)
except ConnectionError as e:
    logger.error(f"Failed to connect with {username}:{password}")  # EXPOSED!
    raise Exception(f"Database connection failed: {e}")  # Stack trace might expose more!

# ✅ GOOD: Sanitized error messages
try:
    connect_to_database(host, username, password)
except ConnectionError as e:
    logger.error(
        f"Failed to connect to database at {host}",
        extra={'username': username}  # Safe to log username
    )
    # User-facing message doesn't expose details
    raise ConnectionError("Database connection failed")

# ✅ BETTER: Error codes instead of details
class ErrorCode(Enum):
    DB_CONNECTION_FAILED = 1001
    DB_AUTHENTICATION_FAILED = 1002
    DB_TIMEOUT = 1003

try:
    connect_to_database(host, username, password)
except ConnectionError:
    logger.error(f"Error code: {ErrorCode.DB_CONNECTION_FAILED}")
    raise ApplicationError(error_code=ErrorCode.DB_CONNECTION_FAILED)
```

### Exception Groups (Python 3.11+)

Handle multiple exceptions simultaneously:

```python
# Multiple related exceptions
try:
    deploy_to_multiple_regions(['us-east-1', 'eu-west-1', 'ap-south-1'])
except* DeploymentError as eg:
    # Handle all deployment errors
    for error in eg.exceptions:
        log_deployment_failure(error)
except* TimeoutError as eg:
    # Handle all timeout errors
    for error in eg.exceptions:
        retry_deployment(error)

# Raising exception groups
def deploy_to_multiple_regions(regions: list[str]):
    """Deploy to multiple regions, collect all failures."""
    errors = []
  
    for region in regions:
        try:
            deploy_to_region(region)
        except DeploymentError as e:
            errors.append(e)
  
    if errors:
        raise ExceptionGroup("Multiple deployments failed", errors)
```

### Frequently Asked Questions

**Q1: Should I catch all exceptions or let them propagate?**

**A:** **It depends on the layer:**

**Let propagate when:**

- Library code (caller should handle)
- Early in call stack
- You can't handle meaningfully

**Catch when:**

- Top-level (main, API endpoints)
- Can handle gracefully (retry, fallback)
- Need cleanup (context managers)

```python
# ❌ BAD: Swallowing exceptions
def process_data(data):
    try:
        result = expensive_computation(data)
        return result
    except Exception:
        return None  # Silent failure!

# ✅ GOOD: Let propagate or log and re-raise
def process_data(data):
    result = expensive_computation(data)  # Let propagate
    return result

# ✅ GOOD: Catch at boundary
def api_endpoint():
    """Top-level handler."""
    try:
        result = process_data(request.data)
        return jsonify(result)
    except ValidationError as e:
        logger.warning(f"Validation error: {e}")
        return jsonify({'error': str(e)}), 400
    except Exception as e:
        logger.error("Unexpected error", exc_info=True)
        return jsonify({'error': 'Internal error'}), 500
```

**Q2: When should I create custom exceptions?**

**A:** Create custom exceptions when:

1. **Domain-specific errors**: `DeploymentError`, `HealthCheckFailed`
2. **Different handling needed**: Each exception type needs different recovery
3. **Additional context**: Attach metadata to exceptions
4. **Clear error hierarchy**: Related exceptions inherit from base

```python
# ✅ GOOD: Clear hierarchy
class InfrastructureError(Exception):
    """Base for infrastructure errors."""
    pass

class ProvisioningError(InfrastructureError):
    """Failed to provision resources."""
    pass

class DeploymentError(InfrastructureError):
    """Failed to deploy application."""
    pass

# Handle hierarchically
try:
    provision_and_deploy()
except ProvisioningError:
    cleanup_partial_resources()
except DeploymentError:
    rollback_deployment()
except InfrastructureError:
    alert_ops_team()
```

**Q3: How do I handle exceptions in async code?**

**A:** Same principles, but use `async`/`await`:

```python
import asyncio

async def deploy_async(service: str):
    """Async deployment with error handling."""
    try:
        await provision_resources(service)
        await deploy_application(service)
    except asyncio.TimeoutError:
        logger.error("Deployment timed out")
        raise
    except Exception as e:
        logger.error(f"Deployment failed: {e}", exc_info=True)
        await rollback_async(service)
        raise
    finally:
        await cleanup_async(service)

# Exception groups with asyncio.gather
async def deploy_multiple_services(services: list[str]):
    """Deploy multiple services, collect errors."""
    tasks = [deploy_async(svc) for svc in services]
    results = await asyncio.gather(*tasks, return_exceptions=True)
  
    errors = [r for r in results if isinstance(r, Exception)]
    if errors:
        for error in errors:
            logger.error(f"Service deployment failed: {error}")
        raise ExceptionGroup("Multiple deployments failed", errors)
```

### Interview Questions

**Question 1: Explain the difference between `except Exception` and bare `except`. Why should you never use bare `except`?**

**Difficulty:** Junior

**Answer:**

**`except Exception`**: Catches all exceptions inheriting from `Exception`
**Bare `except:`**: Catches **everything**, including `BaseException` subclasses

**Problem with bare except:**

```python
# ❌ DANGEROUS: Catches KeyboardInterrupt, SystemExit, etc.
try:
    long_running_task()
except:  # Catches EVERYTHING
    log_error()
    # User presses Ctrl+C but can't exit!

# ✅ CORRECT: Only catch Exception
try:
    long_running_task()
except Exception as e:
    log_error(e)
    # KeyboardInterrupt propagates normally
```

**What bare except catches that you DON'T want:**

- `KeyboardInterrupt` - User trying to kill program
- `SystemExit` - Program trying to exit
- `GeneratorExit` - Generator cleanup

**DevSecOps Impact:**

- Can't gracefully shutdown services
- Can't cancel deployments with Ctrl+C
- Masks critical system signals

**Only exception to the rule:** Very top-level handlers that must catch literally everything, but should re-raise these special exceptions:

```python
try:
    main()
except (KeyboardInterrupt, SystemExit):
    raise  # Let these propagate
except Exception as e:
    log_critical_error(e)
```

**Follow-up:** What's the difference between `Exception` and `BaseException`?

---

**Question 2: Implement a decorator that retries a function with exponential backoff and logs all attempts. The decorator should be configurable for max attempts, base delay, and which exceptions to catch.**

**Difficulty:** Mid-Level

**Answer:**

```python
import time
import logging
import functools
from typing import Callable, Type, Tuple

logger = logging.getLogger(__name__)

def retry_with_exponential_backoff(
    max_attempts: int = 3,
    base_delay: float = 1.0,
    exponential_base: float = 2.0,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
    on_retry: Callable[[Exception, int], None] = None
):
    """
    Decorator that retries function with exponential backoff.
  
    Args:
        max_attempts: Maximum number of attempts
        base_delay: Initial delay in seconds
        exponential_base: Base for exponential calculation
        exceptions: Tuple of exceptions to catch and retry
        on_retry: Optional callback called on each retry
  
    Example:
        @retry_with_exponential_backoff(
            max_attempts=5,
            base_delay=1.0,
            exceptions=(requests.RequestException,)
        )
        def call_api():
            return requests.get('https://api.example.com/data')
    """
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
          
            for attempt in range(max_attempts):
                try:
                    logger.info(
                        f"Attempt {attempt + 1}/{max_attempts} for "
                        f"{func.__name__}"
                    )
                    result = func(*args, **kwargs)
                  
                    if attempt > 0:
                        logger.info(
                            f"{func.__name__} succeeded on attempt "
                            f"{attempt + 1}"
                        )
                  
                    return result
                  
                except exceptions as e:
                    last_exception = e
                  
                    if attempt == max_attempts - 1:
                        # Last attempt, don't sleep or log retry
                        logger.error(
                            f"{func.__name__} failed after "
                            f"{max_attempts} attempts: {e}"
                        )
                        break
                  
                    # Calculate delay
                    delay = base_delay * (exponential_base ** attempt)
                  
                    logger.warning(
                        f"Attempt {attempt + 1} failed: {e}. "
                        f"Retrying in {delay:.2f}s..."
                    )
                  
                    # Call optional callback
                    if on_retry:
                        on_retry(e, attempt + 1)
                  
                    time.sleep(delay)
          
            # All attempts failed
            raise last_exception
      
        return wrapper
    return decorator

# Usage example
@retry_with_exponential_backoff(
    max_attempts=5,
    base_delay=1.0,
    exponential_base=2.0,
    exceptions=(requests.RequestException, TimeoutError),
    on_retry=lambda e, attempt: send_monitoring_metric('retry', attempt)
)
def deploy_to_production(service_name: str, version: str) -> dict:
    """Deploy service to production with automatic retries."""
    response = requests.post(
        'https://deploy.example.com/api/deploy',
        json={'service': service_name, 'version': version},
        timeout=30
    )
    response.raise_for_status()
    return response.json()

# Test the decorator
try:
    result = deploy_to_production('api-service', '1.0.0')
    print(f"Deployment succeeded: {result}")
except requests.RequestException as e:
    print(f"Deployment failed after all retries: {e}")
    notify_ops_team(e)
```

**Why This Matters in DevSecOps:**

- **Resilience**: Handle transient failures (network blips, rate limits)
- **Observability**: Logs show retry patterns
- **Configurable**: Different strategies for different operations
- **Non-invasive**: Decorator doesn't change function signature

**Follow-up:** How would you add jitter to prevent thundering herd?

---

### Key Takeaways

✅ **Catch specific exceptions** - Never use bare `except:`

✅ **Exception chaining preserves context** - Use `raise ... from e`

✅ **EAFP over LBYL** - Try/except instead of checking first

✅ **Context managers ensure cleanup** - Always use for resources

✅ **Retry with exponential backoff** - Handle transient failures

✅ **Circuit breaker prevents cascades** - Fail fast when service is down

✅ **Never expose secrets in errors** - Sanitize all error messages

✅ **Structured logging for production** - JSON logs are searchable

✅ **Custom exceptions for domain errors** - Clear error hierarchy

✅ **Let exceptions propagate** - Unless you can handle them

---

## 1.6 Modules, Packages, and Imports (Summary)

### Key Concepts

**Modules** are Python files containing code. **Packages** are directories containing modules with an `__init__.py` file.

```python
# mypackage/
# ├── __init__.py
# ├── server.py
# └── deployment.py

# Import styles
import mypackage.server  # Full path
from mypackage import server  # Import module
from mypackage.server import Server  # Import class
from mypackage.server import *  # ❌ Never use!

# Relative imports (within package)
from . import deployment  # Same directory
from .. import utils  # Parent directory
```

**Best Practices:**

- One module per file, clear naming
- Avoid circular imports (redesign if needed)
- Use `if __name__ == '__main__':` for scripts
- Organize imports: stdlib, third-party, local

---

## 1.7 File I/O and Serialization (Summary)

### File Operations

```python
# ✅ Always use context managers
with open('config.yaml') as f:
    data = f.read()

# Write safely
with open('output.txt', 'w') as f:
    f.write(data)

# JSON
import json
with open('config.json') as f:
    config = json.load(f)

# YAML (for configs)
import yaml
with open('config.yaml') as f:
    config = yaml.safe_load(f)
```

**Security:**

- Validate paths (prevent path traversal)
- Check file permissions
- Never trust file content
- Use `safe_load()` for YAML

---

## Final Takeaways - Part 1

You now have a solid foundation in Python for DevSecOps:

✅ **Environment Management** - Virtual environments, dependency management, security scanning

✅ **Python Fundamentals** - Data types, memory management, operators, control flow

✅ **Functions** - Type hints, decorators, generators, closures

✅ **OOP** - Classes, inheritance, composition, magic methods, dataclasses

✅ **Error Handling** - Try/except, custom exceptions, retry patterns, circuit breakers

✅ **Core Concepts** - Modules, file I/O, serialization

**Next in Part 2:** Software Engineering Principles - SOLID, design patterns, code architecture, refactoring techniques.

**Continue practicing:** The best way to master Python is to write code daily. Start automating your DevOps tasks!

---

## References and Resources

**Official Documentation:**

- Python Docs: https://docs.python.org/3/
- Python Package Index (PyPI): https://pypi.org/

**Security:**

- OWASP Python Security: https://owasp.org/www-project-python-security/
- Bandit Documentation: https://bandit.readthedocs.io/

**Style Guides:**

- PEP 8: https://pep8.org/
- Google Python Style Guide
- Real Python Tutorials: https://realpython.com/

**DevSecOps Resources:**

- AWS boto3 Documentation
- Kubernetes Python Client
- Ansible Documentation
- Docker SDK for Python

---

*End of Part 1*
