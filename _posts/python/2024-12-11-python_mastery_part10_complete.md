---
title: "Python Mastery - Part 10: Interview Preparation & Real-World Scenarios"
date: 2024-12-11 00:00:00 +0530
categories: [Python, Python Mastery]
tags: [Python, Interview, Career, Real-World, System Design, Questions, Scenarios, Preparation]
---
## Table of Contents

- [Introduction](#introduction)
- [10.1 Technical Interview Questions](#101-technical-interview-questions)
  - [Junior Level Questions](#junior-level-questions)
  - [Mid-Level Questions](#mid-level-questions)
  - [Senior Level Questions](#senior-level-questions)
- [10.2 System Design Questions](#102-system-design-questions)
  - [Design a URL Shortener](#design-a-url-shortener)
  - [Design a Rate Limiter](#design-a-rate-limiter)
  - [Design a Cache System](#design-a-cache-system)
- [10.3 Coding Challenges](#103-coding-challenges)
  - [Data Structures](#data-structures)
  - [Algorithms](#algorithms)
  - [Real-World Problems](#real-world-problems)
- [10.4 Behavioral &amp; Situational Questions](#104-behavioral--situational-questions)
- [10.5 Career Development](#105-career-development)

---

# Complete Python Mastery Part 10: Interview Preparation & Real-World Scenarios

## Introduction

Welcome to **Part 10** - the final part of the Complete Python Mastery series. This part focuses on interview preparation and real-world problem-solving.

**What You'll Learn in Part 10:**

This comprehensive final part covers:

- **Technical interviews**: Questions for junior, mid, and senior levels
- **System design**: Real-world system design problems
- **Coding challenges**: Data structures, algorithms, practical problems
- **Behavioral questions**: Teamwork, leadership, communication
- **Career development**: Progression paths and strategies

**Why This Matters:**

Interview preparation is crucial for:

- Landing your dream job
- Advancing your career
- Demonstrating expertise
- Understanding real-world problems
- Continuous learning

**Prerequisites:**

- Parts 1-9 of this series (100,000+ words of Python mastery!)
- Understanding of Python fundamentals through advanced topics

Let's master Python interviews! 🚀

---

## 10.1 Technical Interview Questions

### Junior Level Questions

**Question 1: What's the difference between a list and a tuple in Python?**

**Difficulty:** Junior

**Answer:**

```python
"""
LIST vs TUPLE

Key differences:
1. Mutability
2. Performance
3. Use cases
4. Memory
"""

# 1. Mutability
# Lists are mutable (can be changed)
my_list = [1, 2, 3]
my_list[0] = 10  # ✅ OK
my_list.append(4)  # ✅ OK
print(my_list)  # [10, 2, 3, 4]

# Tuples are immutable (cannot be changed)
my_tuple = (1, 2, 3)
# my_tuple[0] = 10  # ❌ TypeError
# my_tuple.append(4)  # ❌ AttributeError

# 2. Performance
# Tuples are faster (immutable = optimizable)
import timeit

list_time = timeit.timeit('x = [1, 2, 3, 4, 5]', number=10000000)
tuple_time = timeit.timeit('x = (1, 2, 3, 4, 5)', number=10000000)

print(f"List: {list_time:.4f}s")
print(f"Tuple: {tuple_time:.4f}s")
# Tuple is typically 2x faster

# 3. Use cases

# Use LIST when data might change
shopping_cart = ['apple', 'banana']
shopping_cart.append('orange')  # Cart can grow

# Use TUPLE for fixed data
coordinates = (40.7128, -74.0060)  # NYC coordinates (fixed)
rgb_color = (255, 0, 0)  # Red color (fixed)
database_record = (1, 'John', 'Doe', 25)  # Fixed structure

# 4. Memory
# Tuples use less memory
import sys

my_list = [1, 2, 3, 4, 5]
my_tuple = (1, 2, 3, 4, 5)

print(f"List size: {sys.getsizeof(my_list)} bytes")  # 104 bytes
print(f"Tuple size: {sys.getsizeof(my_tuple)} bytes")  # 80 bytes

# 5. Syntax
list_example = [1, 2, 3]  # Square brackets
tuple_example = (1, 2, 3)  # Parentheses
tuple_single = (1,)  # Comma required for single element

# 6. Methods
# Lists have many methods
list_methods = ['append', 'extend', 'insert', 'remove', 'pop', 
                'clear', 'sort', 'reverse']

# Tuples have only 2 methods
tuple_methods = ['count', 'index']

# 7. Dictionary keys
# Tuples can be dictionary keys (hashable)
locations = {
    (40.7128, -74.0060): 'New York',
    (51.5074, -0.1278): 'London'
}

# Lists cannot be keys (not hashable)
# locations = {[40.7128, -74.0060]: 'New York'}  # ❌ TypeError

# Summary
"""
Aspect          List            Tuple
----------------------------------------------
Mutability      Mutable         Immutable
Performance     Slower          Faster
Memory          More            Less
Methods         Many            Few (2)
Use case        Dynamic data    Fixed data
Dict key        No              Yes
"""
```

---

**Question 2: Explain list comprehensions and when to use them.**

**Difficulty:** Junior

**Answer:**

```python
"""
LIST COMPREHENSIONS: Concise way to create lists

Syntax: [expression for item in iterable if condition]
"""

# Traditional loop approach
numbers = [1, 2, 3, 4, 5]

# ❌ Verbose: Create squares
squares = []
for n in numbers:
    squares.append(n ** 2)

# ✅ Concise: List comprehension
squares = [n ** 2 for n in numbers]
print(squares)  # [1, 4, 9, 16, 25]

# With condition
# Get even squares only
even_squares = [n ** 2 for n in numbers if n % 2 == 0]
print(even_squares)  # [4, 16]

# With if-else
# Mark odd/even
labels = ['even' if n % 2 == 0 else 'odd' for n in numbers]
print(labels)  # ['odd', 'even', 'odd', 'even', 'odd']

# Nested comprehensions
# Create 3x3 matrix
matrix = [[i * 3 + j for j in range(3)] for i in range(3)]
print(matrix)
# [[0, 1, 2],
#  [3, 4, 5],
#  [6, 7, 8]]

# Flatten matrix
flat = [num for row in matrix for num in row]
print(flat)  # [0, 1, 2, 3, 4, 5, 6, 7, 8]

# When to use list comprehensions

# ✅ GOOD: Simple transformations
squares = [x ** 2 for x in range(10)]

# ✅ GOOD: Filtering
evens = [x for x in range(10) if x % 2 == 0]

# ✅ GOOD: String operations
words = ['hello', 'world', 'python']
uppercase = [word.upper() for word in words]

# ❌ BAD: Complex logic (use regular loop)
# Hard to read
results = [
    complex_function(x, y, z) if condition1(x) and condition2(y) 
    else another_function(x) if condition3(z)
    else default_value
    for x, y, z in zip(list1, list2, list3)
    if x > 0 and y < 100
]

# ✅ BETTER: Use regular loop for complex logic
results = []
for x, y, z in zip(list1, list2, list3):
    if x > 0 and y < 100:
        if condition1(x) and condition2(y):
            results.append(complex_function(x, y, z))
        elif condition3(z):
            results.append(another_function(x))
        else:
            results.append(default_value)

# Performance comparison
import timeit

# Regular loop
def regular_loop():
    result = []
    for i in range(1000):
        result.append(i ** 2)
    return result

# List comprehension
def list_comp():
    return [i ** 2 for i in range(1000)]

print(f"Loop: {timeit.timeit(regular_loop, number=10000):.4f}s")
print(f"Comp: {timeit.timeit(list_comp, number=10000):.4f}s")
# Comprehension is typically 30-50% faster

# Other comprehensions

# Set comprehension
unique_squares = {x ** 2 for x in [1, 2, 2, 3, 3, 4]}
print(unique_squares)  # {1, 4, 9, 16}

# Dict comprehension
square_dict = {x: x ** 2 for x in range(5)}
print(square_dict)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Generator expression (memory efficient)
squares_gen = (x ** 2 for x in range(1000000))
# Doesn't create list in memory, generates on-the-fly

# Summary
"""
Use list comprehensions when:
✅ Simple transformation or filtering
✅ One-liner is readable
✅ Need entire list at once

Don't use when:
❌ Logic is complex (hurts readability)
❌ Need to handle exceptions
❌ Processing large data (use generator)
❌ Multiple statements needed
"""
```

---

### Mid-Level Questions

**Question 3: Explain Python's GIL and its implications.**

**Difficulty:** Mid-Level

**Answer:**

```python
"""
GIL (Global Interpreter Lock)

Definition: Mutex that protects access to Python objects,
preventing multiple threads from executing Python bytecode at once.

Key points:
1. Only affects CPython (not Jython, IronPython)
2. Only one thread executes Python code at a time
3. Doesn't affect I/O-bound operations
4. Major impact on CPU-bound multi-threading
"""

# Demonstration: GIL impact on CPU-bound tasks

import threading
import time

def cpu_bound_task(n):
    """CPU-intensive task"""
    count = 0
    for i in range(n):
        count += i ** 2
    return count

# Single-threaded execution
start = time.time()
result1 = cpu_bound_task(10000000)
result2 = cpu_bound_task(10000000)
single_thread_time = time.time() - start
print(f"Single thread: {single_thread_time:.2f}s")

# Multi-threaded execution (with GIL)
start = time.time()
thread1 = threading.Thread(target=cpu_bound_task, args=(10000000,))
thread2 = threading.Thread(target=cpu_bound_task, args=(10000000,))

thread1.start()
thread2.start()
thread1.join()
thread2.join()

multi_thread_time = time.time() - start
print(f"Multi-thread: {multi_thread_time:.2f}s")

# Result: Multi-threading is SLOWER due to GIL overhead!
# Single thread: 2.5s
# Multi-thread: 3.0s (slower!)

# Why GIL exists
"""
1. Simplifies CPython implementation
2. Makes C extension integration easier
3. Protects memory management (reference counting)
4. Historical reasons (CPython is old)
"""

# Solutions to GIL limitations

# 1. Use multiprocessing (separate processes, no GIL)
from multiprocessing import Pool

def cpu_task_wrapper(n):
    return cpu_bound_task(n)

start = time.time()
with Pool(processes=2) as pool:
    results = pool.map(cpu_task_wrapper, [10000000, 10000000])
multi_process_time = time.time() - start
print(f"Multi-process: {multi_process_time:.2f}s")
# Result: 1.3s (almost 2x faster!)

# 2. Use Cython/C extensions (release GIL)
"""
# Cython code with nogil
from cython.parallel import prange

def parallel_sum(int n):
    cdef int i
    cdef long long total = 0
  
    with nogil:  # Release GIL!
        for i in prange(n, schedule='static'):
            total += i * i
  
    return total
"""

# 3. Use PyPy (no GIL in some operations)
"""
PyPy uses STM (Software Transactional Memory)
Allows better multi-threading for some workloads
"""

# 4. Use async for I/O-bound tasks (not affected by GIL)
import asyncio
import aiohttp

async def fetch_url(url):
    """I/O-bound: Not affected by GIL"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

# Multiple async requests run concurrently despite GIL

# When GIL matters

# ❌ CPU-bound multi-threading (GIL is problem)
"""
- Image processing
- Data analysis
- Mathematical computations
- Video encoding

Solution: Use multiprocessing or Cython
"""

# ✅ I/O-bound multi-threading (GIL not a problem)
"""
- Web scraping
- API calls
- File I/O
- Database queries

Threading works fine because threads wait for I/O (release GIL)
"""

# Real-world example
import requests
import concurrent.futures

def download_page(url):
    """I/O-bound: GIL released during network wait"""
    response = requests.get(url)
    return len(response.content)

urls = ['http://example.com'] * 10

# Threading works great for I/O
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(download_page, urls))
# Fast! GIL released during network I/O

# Summary
"""
GIL Impact:
CPU-bound + threading = Slow (use multiprocessing)
I/O-bound + threading = Fast (GIL not an issue)
CPU-bound + multiprocessing = Fast (no GIL)
I/O-bound + async = Fastest (event loop)

Decision tree:
CPU-bound? → Use multiprocessing
I/O-bound? → Use threading or async
Need shared memory? → Use threading + careful design
"""
```

---

**Question 4: How does Python's garbage collection work?**

**Difficulty:** Mid-Level

**Answer:**

```python
"""
PYTHON GARBAGE COLLECTION

Two mechanisms:
1. Reference counting (primary)
2. Generational garbage collection (backup)
"""

import sys
import gc

# 1. Reference Counting

# Every object has reference count
x = []  # refcount = 1
y = x   # refcount = 2
z = x   # refcount = 3

print(sys.getrefcount(x) - 1)  # 3 (-1 for getrefcount's own reference)

del y  # refcount = 2
del z  # refcount = 1
# When refcount reaches 0, object is deallocated immediately

# Demonstration
class MyClass:
    def __init__(self, name):
        self.name = name
        print(f"Created {name}")
  
    def __del__(self):
        print(f"Deleted {name}")

obj = MyClass("Object1")  # Created Object1
del obj  # Deleted Object1 (immediately)

# Problem: Circular references
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create circular reference
node1 = Node(1)
node2 = Node(2)
node1.next = node2
node2.next = node1  # Circular!

# Delete references
del node1
del node2
# Objects NOT deallocated! (circular reference)

# 2. Generational Garbage Collection (solves circular references)

# Python's generations:
"""
Generation 0: Youngest objects (checked most frequently)
Generation 1: Middle-aged objects
Generation 2: Oldest objects (checked least frequently)

When object survives GC, it's promoted to next generation
"""

# View GC stats
print(gc.get_count())  # (700, 10, 10) - thresholds
# Meaning: (gen0_count, gen1_count, gen2_count)

# Trigger manual GC
collected = gc.collect()
print(f"Collected {collected} objects")

# Disable/Enable GC
gc.disable()  # Turn off automatic GC
gc.enable()   # Turn back on

# Find circular references
gc.set_debug(gc.DEBUG_SAVEALL)
gc.collect()
print(f"Garbage: {len(gc.garbage)}")

# Memory management example
import weakref

class Cache:
    """Cache with weak references (doesn't prevent GC)"""
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()
  
    def __setitem__(self, key, value):
        self._cache[key] = value
  
    def __getitem__(self, key):
        return self._cache[key]

# Objects can be garbage collected even if in cache
cache = Cache()
obj = MyClass("Cached")
cache['obj'] = obj

print(len(cache._cache))  # 1
del obj  # Object deleted (weak reference doesn't prevent GC)
print(len(cache._cache))  # 0 (automatically removed from cache)

# Context managers ensure cleanup
class Resource:
    def __init__(self):
        print("Acquiring resource")
  
    def __enter__(self):
        return self
  
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        # Cleanup guaranteed even with exceptions

with Resource() as r:
    # Use resource
    pass
# Resource automatically cleaned up

# Best practices

# 1. Don't rely on __del__
# ❌ BAD
class FileHandler:
    def __init__(self, filename):
        self.file = open(filename)
  
    def __del__(self):
        self.file.close()  # Might not be called!

# ✅ GOOD: Use context manager
class FileHandler:
    def __init__(self, filename):
        self.file = open(filename)
  
    def __enter__(self):
        return self.file
  
    def __exit__(self, *args):
        self.file.close()  # Guaranteed to be called

# 2. Break circular references
# ❌ BAD
parent = {'children': []}
child = {'parent': parent}
parent['children'].append(child)  # Circular!

# ✅ GOOD: Use weak references
parent = {'children': []}
child = {'parent': weakref.ref(parent)}
parent['children'].append(child)

# 3. Manual cleanup for large objects
large_data = [0] * 10000000  # 80MB
# ... use data ...
del large_data  # Explicitly delete
gc.collect()    # Force collection

# Summary
"""
Reference Counting:
✅ Immediate cleanup
✅ Deterministic
❌ Can't handle circular references

Generational GC:
✅ Handles circular references
✅ Efficient (focuses on young objects)
❌ Non-deterministic timing

Best practices:
1. Use context managers (with statement)
2. Avoid circular references
3. Use weak references when appropriate
4. Don't rely on __del__
5. Explicitly delete large objects
"""
```

---

### Senior Level Questions

**Question 5: Design a production-ready decorator that handles retries with exponential backoff.**

**Difficulty:** Senior

**Answer:**

```python
"""
PRODUCTION-READY RETRY DECORATOR

Requirements:
- Exponential backoff
- Maximum retries
- Exception filtering
- Logging
- Timeout
- Metrics
"""

import functools
import time
import logging
from typing import Type, Tuple, Callable, Any
import random

# Setup logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def retry(
    max_attempts: int = 3,
    initial_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0,
    jitter: bool = True,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
    on_retry: Callable[[Exception, int], None] = None,
):
    """
    Decorator for retrying function with exponential backoff.
  
    Args:
        max_attempts: Maximum number of retry attempts
        initial_delay: Initial delay between retries (seconds)
        max_delay: Maximum delay between retries (seconds)
        exponential_base: Base for exponential backoff
        jitter: Add random jitter to prevent thundering herd
        exceptions: Tuple of exceptions to catch and retry
        on_retry: Callback function called on each retry
  
    Example:
        @retry(max_attempts=3, initial_delay=1.0)
        def unstable_api_call():
            response = requests.get('http://api.example.com')
            return response.json()
    """
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            attempt = 0
            delay = initial_delay
          
            while attempt < max_attempts:
                try:
                    # Attempt function call
                    result = func(*args, **kwargs)
                  
                    # Log success if retried
                    if attempt > 0:
                        logger.info(
                            f"{func.__name__} succeeded on attempt {attempt + 1}"
                        )
                  
                    return result
              
                except exceptions as e:
                    attempt += 1
                  
                    # Last attempt failed
                    if attempt >= max_attempts:
                        logger.error(
                            f"{func.__name__} failed after {max_attempts} attempts"
                        )
                        raise
                  
                    # Calculate delay with exponential backoff
                    current_delay = min(
                        delay * (exponential_base ** (attempt - 1)),
                        max_delay
                    )
                  
                    # Add jitter (random factor) to prevent thundering herd
                    if jitter:
                        current_delay = current_delay * (0.5 + random.random())
                  
                    # Log retry
                    logger.warning(
                        f"{func.__name__} failed on attempt {attempt}. "
                        f"Retrying in {current_delay:.2f}s. Error: {e}"
                    )
                  
                    # Call retry callback if provided
                    if on_retry:
                        on_retry(e, attempt)
                  
                    # Wait before retry
                    time.sleep(current_delay)
          
        return wrapper
    return decorator

# Example usage

@retry(
    max_attempts=5,
    initial_delay=1.0,
    max_delay=30.0,
    exceptions=(ConnectionError, TimeoutError)
)
def fetch_data_from_api(url: str) -> dict:
    """Fetch data from unreliable API"""
    import requests
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()

# With callback
retry_count = {'count': 0}

def on_retry_callback(exception: Exception, attempt: int):
    """Called on each retry"""
    retry_count['count'] += 1
    # Could send metrics to monitoring system
    # send_metric('api_retry', 1, tags=['attempt': attempt])

@retry(
    max_attempts=3,
    on_retry=on_retry_callback
)
def api_call():
    # Simulated API call
    if random.random() < 0.7:  # 70% failure rate
        raise ConnectionError("Network error")
    return {"status": "success"}

# Advanced: Async version
import asyncio

def async_retry(
    max_attempts: int = 3,
    initial_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0,
    jitter: bool = True,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
):
    """Async version of retry decorator"""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        async def wrapper(*args, **kwargs) -> Any:
            attempt = 0
            delay = initial_delay
          
            while attempt < max_attempts:
                try:
                    result = await func(*args, **kwargs)
                    if attempt > 0:
                        logger.info(
                            f"{func.__name__} succeeded on attempt {attempt + 1}"
                        )
                    return result
              
                except exceptions as e:
                    attempt += 1
                  
                    if attempt >= max_attempts:
                        logger.error(
                            f"{func.__name__} failed after {max_attempts} attempts"
                        )
                        raise
                  
                    current_delay = min(
                        delay * (exponential_base ** (attempt - 1)),
                        max_delay
                    )
                  
                    if jitter:
                        current_delay = current_delay * (0.5 + random.random())
                  
                    logger.warning(
                        f"{func.__name__} attempt {attempt} failed. "
                        f"Retrying in {current_delay:.2f}s"
                    )
                  
                    await asyncio.sleep(current_delay)
          
        return wrapper
    return decorator

@async_retry(max_attempts=3, initial_delay=0.5)
async def async_api_call(url: str):
    """Async API call with retry"""
    import aiohttp
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# Production considerations

class RetryMetrics:
    """Collect retry metrics for monitoring"""
    def __init__(self):
        self.total_attempts = 0
        self.successful_retries = 0
        self.failed_retries = 0
  
    def record_attempt(self):
        self.total_attempts += 1
  
    def record_success(self):
        self.successful_retries += 1
  
    def record_failure(self):
        self.failed_retries += 1

# Circuit breaker pattern (advanced)
class CircuitBreaker:
    """
    Prevent cascading failures by stopping requests
    when service is down
    """
    def __init__(self, failure_threshold: int = 5, timeout: float = 60.0):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failures = 0
        self.last_failure_time = None
        self.state = 'closed'  # closed, open, half-open
  
    def call(self, func, *args, **kwargs):
        if self.state == 'open':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'half-open'
            else:
                raise Exception("Circuit breaker is OPEN")
      
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
  
    def on_success(self):
        self.failures = 0
        self.state = 'closed'
  
    def on_failure(self):
        self.failures += 1
        self.last_failure_time = time.time()
        if self.failures >= self.failure_threshold:
            self.state = 'open'
            logger.error("Circuit breaker opened")

# Summary
"""
Production-ready retry decorator includes:
✅ Exponential backoff (prevents overwhelming server)
✅ Maximum delay cap (prevents infinite waits)
✅ Jitter (prevents thundering herd)
✅ Exception filtering (only retry specific errors)
✅ Logging (observability)
✅ Metrics collection (monitoring)
✅ Async support (modern APIs)
✅ Circuit breaker (prevent cascading failures)

Key considerations:
- Not all failures should retry (4xx vs 5xx HTTP errors)
- Monitor retry rates (high rate = underlying issue)
- Set reasonable timeouts
- Consider circuit breaker for failing services
- Add metrics for observability
"""
```

---

(Part 10 continues with sections 10.2-10.5...)

## 10.2 System Design Questions

**Question: Design a URL Shortener (like bit.ly)**

**Difficulty:** Senior

**Answer:**

```python
"""
URL SHORTENER SYSTEM DESIGN

Requirements:
1. Functional:
   - Shorten long URL to short code
   - Redirect short URL to original
   - Custom aliases (optional)
   - Expiration (optional)
   - Analytics (clicks, locations)

2. Non-functional:
   - High availability (99.9%)
   - Low latency (<100ms)
   - Scalable (millions of URLs)
   - Durable (no data loss)
"""

# Component 1: URL Encoding
import hashlib
import string
from typing import Optional

class URLShortener:
    """URL shortener with multiple encoding strategies"""
  
    ALPHABET = string.ascii_letters + string.digits
    BASE = len(ALPHABET)  # 62
  
    def __init__(self):
        self.url_map = {}  # short_code -> original_url
        self.reverse_map = {}  # original_url -> short_code
        self.counter = 100000  # Start from 100000 for 6-char codes
  
    # Strategy 1: Counter-based (simple, sequential)
    def encode_counter(self, url: str) -> str:
        """Encode using counter (base62)"""
        if url in self.reverse_map:
            return self.reverse_map[url]
      
        num = self.counter
        self.counter += 1
      
        # Convert to base62
        short_code = self._to_base62(num)
      
        self.url_map[short_code] = url
        self.reverse_map[url] = short_code
      
        return short_code
  
    def _to_base62(self, num: int) -> str:
        """Convert number to base62"""
        if num == 0:
            return self.ALPHABET[0]
      
        result = []
        while num:
            result.append(self.ALPHABET[num % self.BASE])
            num //= self.BASE
      
        return ''.join(reversed(result))
  
    def _from_base62(self, code: str) -> int:
        """Convert base62 to number"""
        num = 0
        for char in code:
            num = num * self.BASE + self.ALPHABET.index(char)
        return num
  
    # Strategy 2: Hash-based (distributed-friendly)
    def encode_hash(self, url: str) -> str:
        """Encode using MD5 hash"""
        if url in self.reverse_map:
            return self.reverse_map[url]
      
        # Generate hash
        hash_value = hashlib.md5(url.encode()).hexdigest()
      
        # Take first 6 characters
        short_code = hash_value[:6]
      
        # Handle collisions
        while short_code in self.url_map:
            # Append counter and rehash
            hash_value = hashlib.md5((url + str(self.counter)).encode()).hexdigest()
            short_code = hash_value[:6]
            self.counter += 1
      
        self.url_map[short_code] = url
        self.reverse_map[url] = short_code
      
        return short_code
  
    def decode(self, short_code: str) -> Optional[str]:
        """Decode short code to original URL"""
        return self.url_map.get(short_code)

# Component 2: Database Schema
"""
-- URLs table
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    user_id BIGINT,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    click_count INT DEFAULT 0,
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id)
);

-- Analytics table
CREATE TABLE analytics (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) NOT NULL,
    clicked_at TIMESTAMP DEFAULT NOW(),
    ip_address VARCHAR(45),
    user_agent TEXT,
    referrer TEXT,
    country VARCHAR(2),
    INDEX idx_short_code (short_code),
    INDEX idx_clicked_at (clicked_at)
);
"""

# Component 3: API Implementation
from fastapi import FastAPI, HTTPException, Request
from pydantic import BaseModel, HttpUrl
import redis
from datetime import datetime, timedelta

app = FastAPI()

# Redis for caching
cache = redis.Redis(host='localhost', port=6379, decode_responses=True)

class URLCreate(BaseModel):
    url: HttpUrl
    custom_alias: Optional[str] = None
    expires_in_days: Optional[int] = None

class URLResponse(BaseModel):
    short_url: str
    original_url: str
    created_at: datetime
    expires_at: Optional[datetime]

@app.post("/shorten", response_model=URLResponse)
async def shorten_url(data: URLCreate):
    """Create short URL"""
    url = str(data.url)
  
    # Check if URL already shortened
    cached = cache.get(f"url:{url}")
    if cached:
        return URLResponse(
            short_url=f"https://short.ly/{cached}",
            original_url=url,
            created_at=datetime.now(),
            expires_at=None
        )
  
    # Generate short code
    if data.custom_alias:
        short_code = data.custom_alias
        # Check if alias available
        if cache.exists(f"code:{short_code}"):
            raise HTTPException(400, "Alias already taken")
    else:
        shortener = URLShortener()
        short_code = shortener.encode_counter(url)
  
    # Calculate expiration
    expires_at = None
    if data.expires_in_days:
        expires_at = datetime.now() + timedelta(days=data.expires_in_days)
        ttl = data.expires_in_days * 86400
    else:
        ttl = None
  
    # Store in cache
    cache.set(f"code:{short_code}", url, ex=ttl)
    cache.set(f"url:{url}", short_code, ex=ttl)
  
    # Store in database (async)
    # await db.insert_url(short_code, url, expires_at)
  
    return URLResponse(
        short_url=f"https://short.ly/{short_code}",
        original_url=url,
        created_at=datetime.now(),
        expires_at=expires_at
    )

@app.get("/{short_code}")
async def redirect_url(short_code: str, request: Request):
    """Redirect to original URL"""
    # Check cache first
    original_url = cache.get(f"code:{short_code}")
  
    if not original_url:
        # Check database
        # original_url = await db.get_url(short_code)
      
        if not original_url:
            raise HTTPException(404, "URL not found")
      
        # Cache for future requests
        cache.set(f"code:{short_code}", original_url, ex=86400)
  
    # Record analytics (async, don't wait)
    # await record_click(short_code, request)
  
    # Increment counter
    cache.incr(f"count:{short_code}")
  
    # Redirect
    from fastapi.responses import RedirectResponse
    return RedirectResponse(url=original_url)

@app.get("/stats/{short_code}")
async def get_stats(short_code: str):
    """Get URL statistics"""
    clicks = cache.get(f"count:{short_code}") or 0
  
    return {
        "short_code": short_code,
        "total_clicks": int(clicks),
        # More stats from database
    }

# Component 4: Scalability Considerations

"""
1. Database Sharding:
   - Shard by short_code first character
   - Consistent hashing for distribution

2. Caching Strategy:
   - Redis for hot URLs (80/20 rule)
   - LRU eviction policy
   - Cache aside pattern

3. Rate Limiting:
   - Per user: 100 URLs/hour
   - Per IP: 1000 redirects/hour

4. Analytics:
   - Write to queue (Kafka/RabbitMQ)
   - Process async with workers
   - Aggregate in data warehouse

5. High Availability:
   - Multi-region deployment
   - Read replicas for database
   - CDN for redirects
   - Health checks and failover
"""

# Component 5: Load Estimation

"""
Traffic:
- 100M URLs created per month
- 10B redirects per month

Storage:
- URL size: ~100 bytes
- Total: 100M * 100 = 10GB/month
- 5 years: 600GB

Database:
- Write: 100M / (30 * 86400) = 38 writes/sec
- Read: 10B / (30 * 86400) = 3,800 reads/sec

Cache:
- Cache 20% hot URLs: 20M URLs = 2GB
- Redis can handle easily

Bandwidth:
- Writes: 38 writes/sec * 100 bytes = 3.8 KB/sec
- Reads: 3,800 reads/sec * 100 bytes = 380 KB/sec

Conclusion: System is read-heavy (100:1 ratio)
"""

# Summary
"""
Key Design Decisions:

1. Encoding:
   ✅ Base62 counter (simple, sequential)
   ✅ Hash-based (distributed)
   Decision: Use both (counter for simplicity, hash for distribution)

2. Database:
   ✅ PostgreSQL (reliability)
   ✅ Sharding (scalability)
   ✅ Read replicas (performance)

3. Caching:
   ✅ Redis (speed)
   ✅ LRU eviction
   ✅ 2GB cache (20% hot data)

4. Analytics:
   ✅ Async processing (queue)
   ✅ Separate database (don't slow redirects)
   ✅ Aggregate periodically

5. Availability:
   ✅ Multi-region
   ✅ Load balancer
   ✅ Health checks
   ✅ Circuit breakers
"""
```

---

## 10.3 Coding Challenges

**Challenge 1: Implement LRU Cache**

**Difficulty:** Mid-to-Senior

**Answer:**

```python
"""
LRU CACHE: Least Recently Used cache

Requirements:
- O(1) get
- O(1) put
- Fixed capacity
- Evict least recently used when full

Data structures:
- Dictionary for O(1) lookup
- Doubly linked list for O(1) reordering
"""

class Node:
    """Doubly linked list node"""
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    """
    LRU Cache implementation
  
    Example:
        cache = LRUCache(capacity=2)
        cache.put(1, 1)  # cache: {1=1}
        cache.put(2, 2)  # cache: {1=1, 2=2}
        cache.get(1)     # returns 1, cache: {2=2, 1=1}
        cache.put(3, 3)  # evicts 2, cache: {1=1, 3=3}
    """
  
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}  # key -> node
      
        # Dummy head and tail
        self.head = Node(0, 0)
        self.tail = Node(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head
  
    def get(self, key: int) -> int:
        """Get value (O(1))"""
        if key not in self.cache:
            return -1
      
        node = self.cache[key]
      
        # Move to front (most recently used)
        self._remove(node)
        self._add_to_front(node)
      
        return node.value
  
    def put(self, key: int, value: int) -> None:
        """Put key-value (O(1))"""
        if key in self.cache:
            # Update existing
            node = self.cache[key]
            node.value = value
          
            # Move to front
            self._remove(node)
            self._add_to_front(node)
        else:
            # Add new
            if len(self.cache) >= self.capacity:
                # Evict least recently used (tail)
                lru = self.tail.prev
                self._remove(lru)
                del self.cache[lru.key]
          
            # Create and add new node
            node = Node(key, value)
            self.cache[key] = node
            self._add_to_front(node)
  
    def _remove(self, node: Node) -> None:
        """Remove node from list"""
        prev = node.prev
        next = node.next
        prev.next = next
        next.prev = prev
  
    def _add_to_front(self, node: Node) -> None:
        """Add node to front (after head)"""
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node

# Test
cache = LRUCache(2)
cache.put(1, 1)
cache.put(2, 2)
print(cache.get(1))  # 1
cache.put(3, 3)      # Evicts key 2
print(cache.get(2))  # -1 (not found)
cache.put(4, 4)      # Evicts key 1
print(cache.get(1))  # -1 (not found)
print(cache.get(3))  # 3
print(cache.get(4))  # 4

# Using OrderedDict (simpler)
from collections import OrderedDict

class LRUCacheSimple:
    """LRU Cache using OrderedDict"""
  
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity
  
    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
      
        # Move to end (most recent)
        self.cache.move_to_end(key)
        return self.cache[key]
  
    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            # Update and move to end
            self.cache.move_to_end(key)
      
        self.cache[key] = value
      
        if len(self.cache) > self.capacity:
            # Remove first (least recent)
            self.cache.popitem(last=False)

# Using functools.lru_cache (built-in)
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    """Fibonacci with automatic LRU caching"""
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # Fast! Cached results
print(fibonacci.cache_info())  # CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)
```

---

**Challenge 2: Design a Task Scheduler**

**Difficulty:** Senior

**Answer:**

```python
"""
TASK SCHEDULER

Requirements:
- Schedule tasks with dependencies
- Execute in correct order
- Handle failures
- Retry logic
- Priority queue
"""

from dataclasses import dataclass, field
from typing import List, Set, Dict, Optional
from enum import Enum
import heapq
from datetime import datetime, timedelta

class TaskStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"

@dataclass(order=True)
class Task:
    """Task with priority and dependencies"""
    priority: int
    id: str = field(compare=False)
    func: callable = field(compare=False)
    dependencies: Set[str] = field(default_factory=set, compare=False)
    status: TaskStatus = field(default=TaskStatus.PENDING, compare=False)
    retry_count: int = field(default=0, compare=False)
    max_retries: int = field(default=3, compare=False)
    created_at: datetime = field(default_factory=datetime.now, compare=False)

class TaskScheduler:
    """Task scheduler with dependency management"""
  
    def __init__(self):
        self.tasks: Dict[str, Task] = {}
        self.ready_queue = []  # Min heap by priority
        self.running: Set[str] = set()
        self.completed: Set[str] = set()
        self.failed: Set[str] = set()
  
    def add_task(
        self,
        task_id: str,
        func: callable,
        priority: int = 0,
        dependencies: Set[str] = None
    ):
        """Add task to scheduler"""
        task = Task(
            id=task_id,
            func=func,
            priority=priority,
            dependencies=dependencies or set()
        )
      
        self.tasks[task_id] = task
      
        # If no dependencies, add to ready queue
        if not task.dependencies:
            heapq.heappush(self.ready_queue, task)
  
    def _check_dependencies(self, task: Task) -> bool:
        """Check if all dependencies completed"""
        return task.dependencies.issubset(self.completed)
  
    def _update_ready_queue(self):
        """Move tasks with satisfied dependencies to ready queue"""
        for task_id, task in self.tasks.items():
            if (task.status == TaskStatus.PENDING and
                task_id not in self.running and
                self._check_dependencies(task)):
                heapq.heappush(self.ready_queue, task)
  
    def execute_next(self) -> Optional[str]:
        """Execute next ready task"""
        if not self.ready_queue:
            self._update_ready_queue()
      
        if not self.ready_queue:
            return None
      
        task = heapq.heappop(self.ready_queue)
        task.status = TaskStatus.RUNNING
        self.running.add(task.id)
      
        try:
            # Execute task
            result = task.func()
          
            # Mark completed
            task.status = TaskStatus.COMPLETED
            self.running.remove(task.id)
            self.completed.add(task.id)
          
            print(f"✓ Task {task.id} completed")
          
            # Check dependent tasks
            self._update_ready_queue()
          
            return task.id
      
        except Exception as e:
            print(f"✗ Task {task.id} failed: {e}")
          
            # Retry logic
            task.retry_count += 1
            if task.retry_count <= task.max_retries:
                print(f"  Retrying {task.id} (attempt {task.retry_count})")
                task.status = TaskStatus.PENDING
                self.running.remove(task.id)
                heapq.heappush(self.ready_queue, task)
            else:
                print(f"  Task {task.id} failed after {task.max_retries} retries")
                task.status = TaskStatus.FAILED
                self.running.remove(task.id)
                self.failed.add(task.id)
          
            return None
  
    def run(self):
        """Run all tasks"""
        while self.ready_queue or self.running:
            self.execute_next()
      
        print(f"\nCompleted: {len(self.completed)}")
        print(f"Failed: {len(self.failed)}")

# Example usage
scheduler = TaskScheduler()

# Define tasks
def task_a():
    print("  Executing task A")
    return "A done"

def task_b():
    print("  Executing task B")
    return "B done"

def task_c():
    print("  Executing task C")
    return "C done"

def task_d():
    print("  Executing task D")
    return "D done"

# Add tasks with dependencies
# D depends on B and C
# B depends on A
# C depends on A
scheduler.add_task("A", task_a, priority=1)
scheduler.add_task("B", task_b, priority=2, dependencies={"A"})
scheduler.add_task("C", task_c, priority=2, dependencies={"A"})
scheduler.add_task("D", task_d, priority=3, dependencies={"B", "C"})

# Run scheduler
scheduler.run()

# Output:
# ✓ Task A completed
# ✓ Task B completed
# ✓ Task C completed
# ✓ Task D completed
```

---

## 10.4 Behavioral & Situational Questions

```python
"""
BEHAVIORAL INTERVIEW QUESTIONS

STAR Method:
- Situation: Context
- Task: Challenge/goal
- Action: What you did
- Result: Outcome

Common questions and frameworks:
"""

behavioral_questions = {
    "Tell me about a time you debugged a difficult production issue": """
    Situation: Production API latency spiked to 5+ seconds
  
    Task: Identify and fix the issue ASAP
  
    Action:
    1. Checked monitoring (Datadog) - saw database query time spike
    2. Enabled slow query log - found N+1 query problem
    3. Added eager loading (joinedload) to reduce queries from 1000 to 1
    4. Deployed fix in 30 minutes
    5. Added alerts for query time > 100ms
  
    Result:
    - Latency dropped from 5s to 50ms
    - Prevented future issues with monitoring
    - Documented for team
    """,
  
    "Describe a time you made a technical decision with tradeoffs": """
    Situation: Needed caching layer for high-traffic API
  
    Task: Choose between Redis, Memcached, or application-level cache
  
    Action:
    1. Evaluated requirements:
       - Need persistence: Redis ✓ Memcached ✗
       - Need data structures: Redis ✓ Memcached ✗
       - Need simplicity: Memcached ✓ Redis ~
    2. Chose Redis for persistence and data structures
    3. Documented tradeoffs (slightly more complex setup)
  
    Result:
    - 80% cache hit rate
    - Response time reduced from 200ms to 20ms
    - Redis features (sorted sets) enabled new features
    """,
  
    "Tell me about a time you disagreed with a team decision": """
    Situation: Team wanted to use microservices for new feature
  
    Task: Evaluate if microservices were appropriate
  
    Action:
    1. Analyzed requirements - feature was simple, low traffic
    2. Presented monolith vs microservices tradeoffs
    3. Proposed starting monolith, extract to microservice if needed
    4. Team agreed after discussion
  
    Result:
    - Shipped feature in 2 weeks (vs 4 weeks with microservices)
    - Feature handled traffic fine in monolith
    - Saved team complexity
    """,
  
    "How do you stay updated with Python developments": """
    Methods:
    1. Read PEPs (Python Enhancement Proposals)
    2. Follow Python core developers on Twitter/Mastodon
    3. Read Real Python, Python Weekly newsletter
    4. Attend PyCon (conference)
    5. Contribute to open source projects
    6. Experiment with new Python versions
    7. Read source code of popular libraries
  
    Recent learning:
    - Python 3.12: Per-interpreter GIL (PEP 684)
    - Type hints improvements (PEP 695)
    - Pattern matching enhancements
    """
}
```

---

## 10.5 Career Development

```python
"""
CAREER DEVELOPMENT: Junior → Senior → Staff → Principal

Progression path and expectations:
"""

career_levels = {
    "Junior Developer (0-2 years)": {
        "Technical Skills": [
            "Python fundamentals",
            "Basic algorithms and data structures",
            "Git basics",
            "Write tests",
            "Debug code",
            "Read documentation"
        ],
        "Soft Skills": [
            "Ask questions",
            "Communicate progress",
            "Learn from feedback",
            "Follow team practices"
        ],
        "Focus": "Learn and execute assigned tasks",
        "Salary Range": "$60K - $90K"
    },
  
    "Mid-Level Developer (2-5 years)": {
        "Technical Skills": [
            "Design patterns",
            "System design basics",
            "Performance optimization",
            "CI/CD pipelines",
            "Code review",
            "Mentor juniors",
            "API design"
        ],
        "Soft Skills": [
            "Work independently",
            "Break down problems",
            "Give constructive feedback",
            "Collaborate cross-team"
        ],
        "Focus": "Own features end-to-end",
        "Salary Range": "$90K - $130K"
    },
  
    "Senior Developer (5-10 years)": {
        "Technical Skills": [
            "Architecture design",
            "System scalability",
            "Technical leadership",
            "Production operations",
            "Security best practices",
            "Performance at scale",
            "Technology evaluation"
        ],
        "Soft Skills": [
            "Lead projects",
            "Influence decisions",
            "Mentor team members",
            "Stakeholder management",
            "Technical writing"
        ],
        "Focus": "Deliver complex projects, raise team quality",
        "Salary Range": "$130K - $200K"
    },
  
    "Staff Engineer (10+ years)": {
        "Technical Skills": [
            "Company-wide architecture",
            "Technical strategy",
            "Complex system design",
            "Cross-team standards",
            "Technology roadmap"
        ],
        "Soft Skills": [
            "Influence without authority",
            "Strategic thinking",
            "Lead initiatives",
            "Communicate to executives"
        ],
        "Focus": "Multiply team effectiveness, shape technical direction",
        "Salary Range": "$200K - $350K+"
    }
}

# Skills to prioritize

priority_skills = {
    "Always Valuable": [
        "Problem solving",
        "Communication",
        "Learning ability",
        "System thinking",
        "Debugging"
    ],
  
    "Python Specific": [
        "Async programming",
        "Performance optimization",
        "Package management",
        "Testing strategies",
        "Type hints"
    ],
  
    "Infrastructure": [
        "Docker/Kubernetes",
        "CI/CD",
        "Monitoring",
        "Cloud platforms (AWS/GCP/Azure)",
        "Database optimization"
    ],
  
    "Soft Skills": [
        "Code review",
        "Technical writing",
        "Mentoring",
        "Project planning",
        "Stakeholder management"
    ]
}

# Learning resources
resources = {
    "Books": [
        "Fluent Python by Luciano Ramalho",
        "Python Cookbook by David Beazley",
        "Designing Data-Intensive Applications by Martin Kleppmann",
        "The Pragmatic Programmer",
        "Clean Code by Robert Martin"
    ],
  
    "Online": [
        "Real Python (realpython.com)",
        "Python Weekly newsletter",
        "PyCon talks (YouTube)",
        "Python documentation",
        "PEPs (Python Enhancement Proposals)"
    ],
  
    "Practice": [
        "LeetCode (algorithms)",
        "HackerRank (Python track)",
        "Open source contributions",
        "Build side projects",
        "Code review others' PRs"
    ]
}
```

---

### Key Takeaways

**Interview Preparation:**

- **Technical Questions**: Know fundamentals through advanced topics
- **System Design**: Practice common systems (URL shortener, cache, rate limiter)
- **Coding**: Practice data structures and algorithms
- **Behavioral**: Use STAR method, prepare stories
- **Career**: Understand progression path and required skills

**Final Series Summary:**

**Parts 1-10 Complete:**

1. Python Fundamentals (29,000 words)
2. Internals & Advanced (20,000 words)
3. Architecture & Design (16,000 words)
4. Testing & Quality (9,200 words)
5. Deployment & DevOps (8,000 words)
6. Version Control (6,500 words)
7. Performance & Production (5,850 words)
8. Advanced Topics (5,300 words)
9. Package Management (4,650 words)
10. Interview Prep (6,000 words)

**Total: ~110,500 words = ~442 pages!**

---

## Series Complete! 🎉

You've completed **all 10 parts** of the Complete Python Mastery series!

**🏆 From Zero to Python Master in 110,000+ Words! 🏆**

This comprehensive series covered:
✅ Python fundamentals → Advanced features
✅ CPython internals → Performance optimization
✅ Design patterns → System architecture
✅ Testing strategies → Production deployment
✅ Git workflows → Team collaboration
✅ Data science → Web scraping
✅ Package creation → PyPI publishing
✅ Interview preparation → Career development

**You now have complete Python mastery!** 🚀🎓
