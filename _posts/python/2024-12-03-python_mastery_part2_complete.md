---
title: "Python Mastery - Part 2: Python Internals & Advanced Language Features"
date: 2024-12-03 00:00:00 +0530
categories: [Python, Python Mastery]
tags: [Python, Software Engineering, Internals, Cpython, Async, Concurrency, Meta-programming, Type-hints, Performance]
---

## Table of Contents

- [Introduction](#introduction)
- [2.1 CPython Architecture](#21-cpython-architecture)
  - [CPython Overview](#cpython-overview)
  - [Compilation Pipeline](#compilation-pipeline)
  - [Bytecode Execution](#bytecode-execution)
  - [Memory Management](#memory-management)
  - [Garbage Collection](#garbage-collection)
  - [Global Interpreter Lock (GIL)](#global-interpreter-lock-gil)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [2.2 Type System &amp; Type Hints](#22-type-system--type-hints)
  - [Dynamic Typing](#dynamic-typing)
  - [Type Hints](#type-hints-pep-484)
  - [Static Type Checking](#static-type-checking)
  - [Runtime Type Checking](#runtime-type-checking)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
  - [Interview Questions](#interview-questions-1)
- [2.3 Iterators and Generators](#23-iterators-and-generators)
  - [Iterator Protocol](#iterator-protocol)
  - [Generators](#generators)
  - [Itertools Module](#itertools-module)
  - [Advanced Iteration](#advanced-iteration)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
  - [Interview Questions](#interview-questions-2)
- [2.4 Functional Programming](#24-functional-programming)
  - [Functional Paradigms](#functional-paradigms)
  - [Functional Tools](#functional-tools)
  - [Functional Patterns](#functional-patterns)
  - [Frequently Asked Questions](#frequently-asked-questions-3)
  - [Interview Questions](#interview-questions-3)
- [2.5 Asynchronous Programming](#25-asynchronous-programming)
  - [Async Fundamentals](#async-fundamentals)
  - [Async Patterns](#async-patterns)
  - [Event Loop](#event-loop)
  - [Async Libraries](#async-libraries)
  - [Async Best Practices](#async-best-practices)
  - [Frequently Asked Questions](#frequently-asked-questions-4)
  - [Interview Questions](#interview-questions-4)
- [2.6 Concurrency &amp; Parallelism](#26-concurrency--parallelism)
  - [Threading](#threading)
  - [Multiprocessing](#multiprocessing)
  - [Concurrent.futures](#concurrentfutures)
  - [Performance Comparison](#performance-comparison)
  - [Frequently Asked Questions](#frequently-asked-questions-5)
  - [Interview Questions](#interview-questions-5)
- [2.7 Metaprogramming](#27-metaprogramming)
  - [Introspection](#introspection)
  - [Dynamic Code Execution](#dynamic-code-execution)
  - [Metaclasses](#metaclasses)
  - [Descriptors](#descriptors)
  - [Decorators Deep Dive](#decorators-deep-dive)
  - [Abstract Syntax Trees](#abstract-syntax-trees-ast)
  - [Frequently Asked Questions](#frequently-asked-questions-6)
  - [Interview Questions](#interview-questions-6)

---

# Complete Python Mastery Part 2: Python Internals & Advanced Language Features

## Introduction

Welcome to **Part 2** of the Complete Python Mastery series. Having mastered Python's fundamentals in Part 1, you're now ready to explore what happens under the hood—how Python actually works at the implementation level, and how to leverage advanced language features for building high-performance, production-grade systems.

**What You'll Learn in Part 2:**

This post dives deep into Python's internal architecture and advanced features that separate junior developers from senior engineers. You'll understand:

- **How CPython executes your code**: From source to bytecode to machine execution
- **Memory management**: Reference counting, garbage collection, and optimization
- **The GIL**: What it is, why it exists, and how to work around it
- **Type systems**: Static type checking, runtime validation, and gradual typing
- **Advanced iteration**: Generators, iterators, and lazy evaluation patterns
- **Functional programming**: Leveraging Python's functional capabilities
- **Asynchronous programming**: Building high-concurrency applications with asyncio
- **Concurrency models**: Threading, multiprocessing, and when to use each
- **Metaprogramming**: Metaclasses, descriptors, decorators, and AST manipulation

**Why This Matters:**

Understanding Python internals is crucial for:

- Writing performant code (knowing what's expensive)
- Debugging complex issues (understanding stack traces, memory leaks)
- Making architectural decisions (async vs threading vs multiprocessing)
- Interview preparation (senior-level questions focus on internals)
- Optimizing production systems (profiling, tuning, scaling)

**Prerequisites:**

Before diving into Part 2, you should be comfortable with Part 1 topics:

- Python syntax and data structures
- Functions, closures, and decorators (basics)
- Object-oriented programming
- Modules and packages
- Exception handling

Let's begin our journey into Python's internals! 🚀

---

## 2.1 CPython Architecture

**SDLC Phase:** Development, Performance Optimization

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (understanding performance)
- [X] Testing (debugging, profiling)
- [ ] Deployment
- [X] Maintenance (optimization, troubleshooting)

CPython is the reference implementation of Python, written in C. Understanding how CPython works internally helps you write more efficient code and debug issues effectively.

### CPython Overview

#### Python Implementations

```python
# Multiple Python implementations exist:

# 1. CPython (Reference Implementation)
# - Written in C
# - Most widely used
# - Official Python distribution
# - Has the GIL

# 2. PyPy
# - JIT-compiled Python
# - Often faster than CPython for long-running processes
# - Uses RPython (subset of Python)
# - Has its own GIL implementation

# 3. Jython
# - Python for the JVM
# - Can use Java libraries
# - No GIL (uses JVM threading)

# 4. IronPython
# - Python for .NET
# - Can use .NET libraries
# - No GIL (uses CLR threading)

# 5. MicroPython
# - Python for microcontrollers
# - Subset of Python 3
# - Memory-optimized

# Check implementation
import sys
print(sys.implementation.name)  # 'cpython'
print(sys.version)  # CPython version info
```

#### Python Execution Model

```python
# Python execution follows this pipeline:
# 1. Source Code (.py)
# 2. → Lexical Analysis (Tokenization)
# 3. → Parsing (Parse Tree)
# 4. → Abstract Syntax Tree (AST)
# 5. → Bytecode (.pyc)
# 6. → Python Virtual Machine (PVM)
# 7. → Execution

# Example: What happens when you run 'python script.py'?

# Step 1: Source code
source = """
def add(a, b):
    return a + b

result = add(10, 20)
"""

# Step 2: Tokenization
import tokenize
import io

tokens = list(tokenize.generate_tokens(io.StringIO(source).readline))
# Tokens: NAME('def'), NAME('add'), OP('('), NAME('a'), etc.

# Step 3: Parsing & AST
import ast

tree = ast.parse(source)
print(ast.dump(tree, indent=2))
# Module(
#   body=[
#     FunctionDef(name='add', ...),
#     Assign(...),
#   ]
# )

# Step 4: Compilation to bytecode
code = compile(source, '<string>', 'exec')
print(code.co_code)  # Bytecode bytes

# Step 5: Execution
exec(code)  # Runs in PVM
```

#### C API Fundamentals

```python
# Python objects are C structures at the lowest level

# Every Python object is a PyObject*
# C structure (simplified):
"""
typedef struct _object {
    Py_ssize_t ob_refcnt;  # Reference count
    PyTypeObject *ob_type;  # Type object
} PyObject;
"""

# Integer object structure:
"""
typedef struct {
    PyObject_HEAD
    long ob_ival;  # Integer value
} PyIntObject;
"""

# Check reference count
import sys

x = [1, 2, 3]
print(sys.getrefcount(x))  # 2 (x + getrefcount parameter)

y = x  # New reference
print(sys.getrefcount(x))  # 3

del y
print(sys.getrefcount(x))  # 2

# Object size in memory
print(sys.getsizeof(x))  # 88 bytes (on 64-bit system)
print(sys.getsizeof(10))  # 28 bytes (integer)
print(sys.getsizeof("hello"))  # 54 bytes (string)
```

### Compilation Pipeline

#### Source to AST

```python
import ast

# Source code
source = """
x = 10
if x > 5:
    print("x is large")
"""

# Parse to AST
tree = ast.parse(source)

# Examine AST
print(ast.dump(tree, indent=2))
"""
Module(
  body=[
    Assign(
      targets=[Name(id='x', ctx=Store())],
      value=Constant(value=10)
    ),
    If(
      test=Compare(
        left=Name(id='x', ctx=Load()),
        ops=[Gt()],
        comparators=[Constant(value=5)]
      ),
      body=[
        Expr(
          value=Call(
            func=Name(id='print', ctx=Load()),
            args=[Constant(value='x is large')],
            keywords=[]
          )
        )
      ],
      orelse=[]
    )
  ]
)
"""

# Modify AST
class ConstantMultiplier(ast.NodeTransformer):
    def visit_Constant(self, node):
        if isinstance(node.value, int):
            node.value *= 2
        return node

# Transform AST
transformer = ConstantMultiplier()
new_tree = transformer.visit(tree)

# Compile modified AST
code = compile(new_tree, '<ast>', 'exec')
exec(code)  # x = 20, prints "x is large"
```

#### Bytecode Structure

```python
import dis

def add(a, b):
    return a + b

# Disassemble function
dis.dis(add)
"""
  2           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 RETURN_VALUE
"""

# Bytecode instructions explained:
# LOAD_FAST 0    : Load local variable 'a' onto stack
# LOAD_FAST 1    : Load local variable 'b' onto stack  
# BINARY_ADD     : Pop top 2 values, add them, push result
# RETURN_VALUE   : Return top of stack

# Access bytecode directly
print(add.__code__.co_code)  # b'd\x00d\x01\x17\x00S\x00'

# Code object attributes
code = add.__code__
print(code.co_argcount)    # 2 (number of arguments)
print(code.co_varnames)    # ('a', 'b') (local variable names)
print(code.co_consts)      # (None,) (constants)
print(code.co_names)       # () (global names)

# More complex example
def complex_function(x):
    if x > 10:
        result = x * 2
    else:
        result = x + 5
    return result

dis.dis(complex_function)
"""
  2           0 LOAD_FAST                0 (x)
              2 LOAD_CONST               1 (10)
              4 COMPARE_OP               4 (>)
              6 POP_JUMP_IF_FALSE       16

  3           8 LOAD_FAST                0 (x)
             10 LOAD_CONST               2 (2)
             12 BINARY_MULTIPLY
             14 STORE_FAST               1 (result)

  5     >>   16 LOAD_FAST                1 (result)
             18 RETURN_VALUE

  4          20 LOAD_FAST                0 (x)
             22 LOAD_CONST               3 (5)
             24 BINARY_ADD
             26 STORE_FAST               1 (result)
             28 JUMP_ABSOLUTE           16
"""
```

#### Compiler Optimizations

```python
# Constant Folding
def constant_folding():
    x = 24 * 60 * 60  # Computed at compile time
    return x

dis.dis(constant_folding)
"""
  2           0 LOAD_CONST               1 (86400)  # Already computed!
              2 STORE_FAST               0 (x)
              4 LOAD_FAST                0 (x)
              6 RETURN_VALUE
"""

# vs runtime computation
def no_folding(h, m, s):
    return h * m * s  # Computed at runtime

dis.dis(no_folding)
"""
  2           0 LOAD_FAST                0 (h)
              2 LOAD_FAST                1 (m)
              4 BINARY_MULTIPLY
              6 LOAD_FAST                2 (s)
              8 BINARY_MULTIPLY
             10 RETURN_VALUE
"""

# Peephole Optimization
# Example: Eliminating dead code
def dead_code():
    x = 10
    if False:  # Dead branch
        print("Never executed")
    return x

dis.dis(dead_code)
# 'if False' block is optimized away

# Example: Constant expressions
def constant_expr():
    x = [1, 2, 3]  # BUILD_LIST
    y = (1, 2, 3)  # LOAD_CONST (tuple is immutable)
    z = {1, 2, 3}  # BUILD_SET (set is mutable)
    return x, y, z

dis.dis(constant_expr)
"""
Notice: Tuple is loaded as constant, list/set are built at runtime
"""
```

### Bytecode Execution

#### Python Virtual Machine

```python
# PVM is a stack-based interpreter

# Example: a + b execution
# 1. LOAD_FAST a     → Stack: [a]
# 2. LOAD_FAST b     → Stack: [a, b]
# 3. BINARY_ADD      → Stack: [a+b]
# 4. RETURN_VALUE    → Returns a+b

# Simulating PVM (simplified)
class SimplePVM:
    def __init__(self):
        self.stack = []
        self.locals = {}
  
    def LOAD_CONST(self, value):
        self.stack.append(value)
  
    def LOAD_FAST(self, name):
        self.stack.append(self.locals[name])
  
    def STORE_FAST(self, name):
        self.locals[name] = self.stack.pop()
  
    def BINARY_ADD(self):
        b = self.stack.pop()
        a = self.stack.pop()
        self.stack.append(a + b)
  
    def RETURN_VALUE(self):
        return self.stack.pop()

# Execute: x = 10 + 20
pvm = SimplePVM()
pvm.LOAD_CONST(10)      # Stack: [10]
pvm.LOAD_CONST(20)      # Stack: [10, 20]
pvm.BINARY_ADD()        # Stack: [30]
pvm.STORE_FAST('x')     # locals={'x': 30}, Stack: []
```

#### Evaluation Loop

```python
# CPython's main evaluation loop (simplified C pseudocode):
"""
PyObject* PyEval_EvalFrameEx(PyFrameObject *f) {
    PyObject **stack_pointer;
    unsigned char *next_instr;
  
    for (;;) {
        opcode = *next_instr++;
      
        switch (opcode) {
            case LOAD_FAST:
                x = GETLOCAL(oparg);
                PUSH(x);
                break;
          
            case BINARY_ADD:
                b = POP();
                a = POP();
                x = PyNumber_Add(a, b);
                PUSH(x);
                break;
          
            case RETURN_VALUE:
                retval = POP();
                goto exit_eval_frame;
        }
    }
  
exit_eval_frame:
    return retval;
}
"""

# Frame objects
import sys

def outer():
    def inner():
        frame = sys._getframe()  # Current frame
        print(f"Function: {frame.f_code.co_name}")
        print(f"Line number: {frame.f_lineno}")
        print(f"Locals: {frame.f_locals}")
      
        # Parent frame
        parent = frame.f_back
        print(f"Caller: {parent.f_code.co_name}")
    inner()

outer()
"""
Function: inner
Line number: 123
Locals: {'frame': <frame object>}
Caller: outer
"""
```

### Memory Management

#### Object Representation

```python
# Every Python object has:
# 1. Reference count (ob_refcnt)
# 2. Type pointer (ob_type)
# 3. Object-specific data

import sys

# Reference count
x = [1, 2, 3]
print(sys.getrefcount(x))  # 2 (x + parameter to getrefcount)

y = x  # Increment refcount
print(sys.getrefcount(x))  # 3

del y  # Decrement refcount
print(sys.getrefcount(x))  # 2

# Object id (memory address)
print(id(x))  # 140234567890 (memory address)
print(hex(id(x)))  # 0x7f8b2c3d4e50

# Object type
print(type(x))  # <class 'list'>
print(type(x).__name__)  # 'list'

# Object size
print(sys.getsizeof([]))  # 56 bytes (empty list)
print(sys.getsizeof([1]))  # 64 bytes
print(sys.getsizeof([1, 2]))  # 72 bytes
# List grows by 8 bytes per element (pointer size on 64-bit)
```

#### Reference Counting

```python
import sys

# Create object
x = [1, 2, 3]  # refcount = 1

# New reference
y = x  # refcount = 2

# Reference in container
container = [x]  # refcount = 3

# Reference as parameter
def process(obj):
    print(sys.getrefcount(obj))  # refcount + 1 (parameter)

process(x)  # 4 (x, y, container, parameter)

# Remove references
del y  # refcount = 3
container.clear()  # refcount = 2
# When refcount reaches 0, object is deallocated

# ✅ GOOD: Let Python manage memory
def create_large_data():
    data = [i for i in range(1000000)]
    # Process data
    return result
# 'data' is deallocated when function returns

# ❌ BAD: Keeping unnecessary references
global_cache = []  # Keeps objects alive forever

def process_data(data):
    global_cache.append(data)  # Memory leak!
    return do_something(data)

# ✅ BETTER: Use weak references for caching
import weakref

class Cache:
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()
  
    def store(self, key, value):
        self._cache[key] = value
  
    def get(self, key):
        return self._cache.get(key)

# Objects can be garbage collected even if cached
```

#### Memory Allocators

```python
# Python uses multiple memory allocators:

# 1. PyMem_* - Raw memory allocator
# 2. PyObject_* - Object allocator
# 3. PyMem_Malloc/Free - General purpose

# Small object optimization
# Objects < 512 bytes use arena allocation
# Objects >= 512 bytes use system malloc

import sys

# Small integers are cached (-5 to 256)
a = 10
b = 10
print(a is b)  # True (same object)
print(id(a) == id(b))  # True

# Large integers are not cached
x = 1000
y = 1000
print(x is y)  # False (different objects)
print(x == y)  # True (same value)

# String interning
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True (interned)

# Longer strings may not be interned
long1 = "hello" * 100
long2 = "hello" * 100
print(long1 is long2)  # May be False

# Force interning
s3 = sys.intern("very long string that wouldn't normally be interned")
s4 = sys.intern("very long string that wouldn't normally be interned")
print(s3 is s4)  # True
```

#### Memory Pools

```python
# CPython uses memory pools for objects < 512 bytes

# Arena structure:
# Arena (256 KB)
#   ├─ Pool (4 KB)
#   │   └─ Blocks (8, 16, 24, ... 512 bytes)
#   ├─ Pool
#   └─ ...

# Size classes (8-byte aligned)
# 8, 16, 24, 32, 40, 48, 56, 64, ..., 504, 512 bytes

import sys

# Small objects use pools
class SmallObj:
    __slots__ = ('x', 'y')  # Reduce memory
  
    def __init__(self, x, y):
        self.x = x
        self.y = y

obj = SmallObj(1, 2)
print(sys.getsizeof(obj))  # 40 bytes (uses pool)

# Without __slots__
class LargeObj:
    def __init__(self, x, y):
        self.x = x
        self.y = y

obj2 = LargeObj(1, 2)
print(sys.getsizeof(obj2))  # 48 bytes (+ __dict__)
print(sys.getsizeof(obj2.__dict__))  # 112 bytes

# ✅ GOOD: Use __slots__ for memory efficiency
class Point:
    __slots__ = ('x', 'y')
  
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Create many points
points = [Point(i, i*2) for i in range(1000000)]
# Uses much less memory than without __slots__
```

### Garbage Collection

#### Reference Counting Limitations

```python
# Reference counting can't handle cycles

# Cycle example
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create cycle
node1 = Node(1)
node2 = Node(2)
node1.next = node2  # node2 refcount = 2
node2.next = node1  # node1 refcount = 2

# Delete references
del node1  # refcount = 1 (cycle keeps it alive!)
del node2  # refcount = 1 (cycle keeps it alive!)

# Both objects have refcount = 1 but are unreachable
# This is a memory leak with pure reference counting

# Python's solution: Cyclic garbage collector
import gc

# GC detects and collects cycles
gc.collect()  # Manual collection
```

#### Cyclic Garbage Collector

```python
import gc

# GC tracks container objects (list, dict, set, class instances)

# Check if object is tracked
x = [1, 2, 3]
print(gc.is_tracked(x))  # True

y = 42
print(gc.is_tracked(y))  # False (integers not tracked)

# GC statistics
print(gc.get_count())  # (threshold0, threshold1, threshold2)
print(gc.get_stats())  # Detailed GC stats

# Disable/enable GC
gc.disable()  # Turn off GC
# ... performance-critical code ...
gc.enable()  # Turn back on

# Manual collection
collected = gc.collect()  # Returns number of objects collected
print(f"Collected {collected} objects")

# Get all tracked objects
all_objects = gc.get_objects()
print(f"Tracking {len(all_objects)} objects")

# Find references to an object
obj = [1, 2, 3]
referrers = gc.get_referrers(obj)
print(referrers)
```

#### Generational GC

```python
import gc

# Python uses generational GC with 3 generations
# - Generation 0: Young objects
# - Generation 1: Survived one GC
# - Generation 2: Long-lived objects

# GC thresholds
thresholds = gc.get_threshold()
print(thresholds)  # (700, 10, 10)
# Gen 0: Collect after 700 allocations
# Gen 1: Collect after 10 gen-0 collections
# Gen 2: Collect after 10 gen-1 collections

# Tune GC (for high-throughput applications)
gc.set_threshold(1000, 15, 15)  # Less frequent collections

# Get collection counts
counts = gc.get_count()
print(counts)  # (allocations, gen1_count, gen2_count)

# Force collection of specific generation
gc.collect(0)  # Collect generation 0
gc.collect(1)  # Collect generations 0 and 1
gc.collect(2)  # Collect all generations

# GC callbacks
def gc_callback(phase, info):
    print(f"GC {phase}: {info}")

gc.callbacks.append(gc_callback)
gc.collect()  # Triggers callbacks
```

#### Memory Profiling

```python
# tracemalloc - Track memory allocations
import tracemalloc

tracemalloc.start()

# Code to profile
data = [i for i in range(1000000)]

# Get snapshot
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

# Display top 10 allocations
for stat in top_stats[:10]:
    print(stat)
# Output:
# file.py:123: size=8.00 MiB, count=1000000, average=8 B

tracemalloc.stop()

# memory_profiler - Line-by-line memory usage
# (Requires: pip install memory-profiler)
"""
from memory_profiler import profile

@profile
def memory_heavy_function():
    data = [i**2 for i in range(1000000)]
    return sum(data)

# Run with: python -m memory_profiler script.py
"""

# ✅ GOOD: Use generators for large datasets
def process_large_file(filename):
    with open(filename) as f:
        for line in f:  # Generator - memory efficient
            yield process(line)

# ❌ BAD: Load entire file
def bad_process(filename):
    with open(filename) as f:
        data = f.read()  # Loads entire file into memory!
    return process(data)
```

#### Weak References

```python
import weakref

# Weak reference doesn't increase refcount
obj = [1, 2, 3]
weak_ref = weakref.ref(obj)

print(weak_ref())  # [1, 2, 3]
print(weak_ref() is obj)  # True

# Delete strong reference
del obj
print(weak_ref())  # None (object was collected)

# WeakValueDictionary - Cache that doesn't prevent GC
class Cache:
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()
  
    def store(self, key, value):
        self._cache[key] = value
  
    def get(self, key):
        return self._cache.get(key)

cache = Cache()
data = [1, 2, 3]
cache.store('key1', data)

print(cache.get('key1'))  # [1, 2, 3]

del data  # Object can be garbage collected
print(cache.get('key1'))  # None

# ✅ GOOD: Use for observer pattern
class Subject:
    def __init__(self):
        self._observers = []
  
    def attach(self, observer):
        # Weak reference prevents memory leaks
        self._observers.append(weakref.ref(observer))
  
    def notify(self):
        # Clean up dead references
        self._observers = [obs for obs in self._observers if obs() is not None]
      
        for obs_ref in self._observers:
            obs = obs_ref()
            if obs is not None:
                obs.update()
```

### Global Interpreter Lock (GIL)

#### What is the GIL?

```python
# GIL = Global Interpreter Lock
# Mutex that protects access to Python objects
# Only one thread can execute Python bytecode at a time

# Why GIL exists:
# 1. Memory management (reference counting)
# 2. C extensions weren't thread-safe
# 3. Simpler interpreter implementation

import threading
import time

# CPU-bound task with GIL
def cpu_bound_task():
    total = 0
    for i in range(10**7):
        total += i
    return total

# Single thread
start = time.time()
result1 = cpu_bound_task()
print(f"Single thread: {time.time() - start:.2f}s")
# ~0.5 seconds

# Multi-threaded (doesn't help due to GIL!)
start = time.time()
threads = []
for _ in range(2):
    t = threading.Thread(target=cpu_bound_task)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"Two threads: {time.time() - start:.2f}s")
# ~0.5 seconds (same time! GIL prevents parallelism)

# I/O-bound task (GIL released during I/O)
def io_bound_task():
    time.sleep(0.1)  # Simulates I/O

# Multi-threaded I/O (works well!)
start = time.time()
threads = []
for _ in range(10):
    t = threading.Thread(target=io_bound_task)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"10 I/O threads: {time.time() - start:.2f}s")
# ~0.1 seconds (10x speedup!)
```

#### GIL Implementation

```python
# GIL acquisition and release (simplified C pseudocode):
"""
// Acquire GIL
void PyEval_AcquireThread(PyThreadState *tstate) {
    PyThread_acquire_lock(interpreter_lock, 1);  // Block until available
    // ... setup thread state ...
}

// Release GIL
void PyEval_ReleaseThread(PyThreadState *tstate) {
    // ... cleanup thread state ...
    PyThread_release_lock(interpreter_lock);
}

// Evaluation loop releases GIL periodically
for (;;) {
    if (--gil_drop_request <= 0) {
        // Release GIL
        PyThread_release_lock(interpreter_lock);
        // Immediately reacquire
        PyThread_acquire_lock(interpreter_lock, 1);
    }
  
    // Execute bytecode
    opcode = *next_instr++;
    // ...
}
"""

# Check if GIL is held
import sys

def holding_gil():
    # This function holds the GIL while executing
    total = sum(range(1000000))
    return total

# GIL is automatically released during:
# - I/O operations (file, network)
# - time.sleep()
# - Some C extensions (if they release it)
```

#### Impact on Multi-threading

```python
import threading
import time
import multiprocessing

# Benchmark: CPU-bound task
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Single-threaded
def single_thread():
    start = time.time()
    result = fibonacci(35)
    return time.time() - start

# Multi-threaded (doesn't help!)
def multi_threaded():
    start = time.time()
    threads = []
    for _ in range(2):
        t = threading.Thread(target=fibonacci, args=(35,))
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    return time.time() - start

# Multi-processing (true parallelism!)
def multi_process():
    start = time.time()
    with multiprocessing.Pool(2) as pool:
        results = pool.map(fibonacci, [35, 35])
    return time.time() - start

print(f"Single thread: {single_thread():.2f}s")  # ~3.5s
print(f"Multi-threaded: {multi_threaded():.2f}s")  # ~3.5s (no improvement!)
print(f"Multi-process: {multi_process():.2f}s")  # ~1.8s (2x speedup!)

# ✅ GOOD: Use threading for I/O-bound tasks
import requests

def fetch_url(url):
    return requests.get(url).status_code

urls = ['http://example.com'] * 10

# Threading works great for I/O
start = time.time()
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = list(executor.map(fetch_url, urls))
print(f"Threading: {time.time() - start:.2f}s")

# ✅ GOOD: Use multiprocessing for CPU-bound tasks
def heavy_computation(n):
    return sum(i**2 for i in range(n))

numbers = [10**6] * 4

start = time.time()
with multiprocessing.Pool() as pool:
    results = pool.map(heavy_computation, numbers)
print(f"Multiprocessing: {time.time() - start:.2f}s")
```

#### GIL-free Alternatives

```python
# 1. Multiprocessing - Bypass GIL
from multiprocessing import Process, Queue

def worker(queue, n):
    result = sum(i**2 for i in range(n))
    queue.put(result)

queue = Queue()
processes = []
for i in range(4):
    p = Process(target=worker, args=(queue, 10**6))
    processes.append(p)
    p.start()

for p in processes:
    p.join()

results = [queue.get() for _ in processes]

# 2. Async I/O - No GIL needed
import asyncio
import aiohttp

async def fetch_async(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    urls = ['http://example.com'] * 10
    tasks = [fetch_async(url) for url in urls]
    results = await asyncio.gather(*tasks)

asyncio.run(main())

# 3. Cython with nogil
"""
# example.pyx
from cython.parallel import prange

def parallel_sum(int n):
    cdef int i
    cdef long total = 0
  
    with nogil:  # Release GIL
        for i in prange(n, schedule='static'):
            total += i
  
    return total
"""

# 4. C extensions (release GIL manually)
"""
// In C extension
PyObject* heavy_computation(PyObject* self, PyObject* args) {
    // ... parse args ...
  
    Py_BEGIN_ALLOW_THREADS  // Release GIL
    // CPU-intensive work here
    result = do_heavy_computation();
    Py_END_ALLOW_THREADS  // Reacquire GIL
  
    return Py_BuildValue("i", result);
}
"""
```

#### Future of the GIL

```python
# PEP 703: Making the GIL Optional (Python 3.13+)

# Experimental nogil builds available
# Build CPython with --disable-gil flag

# Status as of 2025:
# - Experimental in Python 3.13
# - May become default in future versions
# - Performance overhead for single-threaded code
# - Significant speedup for multi-threaded CPU-bound code

# Check if GIL is disabled
import sys

try:
    print(sys._is_gil_enabled())  # Python 3.13+
except AttributeError:
    print("GIL status check not available")

# Code written for GIL-enabled Python will work with nogil
# But may need optimization for best performance

# ✅ FUTURE: True multi-threaded Python
"""
from concurrent.futures import ThreadPoolExecutor

def cpu_task(n):
    return sum(i**2 for i in range(n))

# With nogil, this will use all CPU cores
with ThreadPoolExecutor(max_workers=8) as executor:
    results = list(executor.map(cpu_task, [10**7] * 8))
"""
```

### Frequently Asked Questions

**Q1: Why is CPython slower than compiled languages?**

**A:**

```python
# Multiple factors make CPython slower:

# 1. Interpretation overhead
def add(a, b):
    return a + b

# In C/C++ (compiled):
# - Compiles to machine code once
# - Direct CPU execution
# - ~1 CPU instruction

# In CPython (interpreted):
# - Parses to bytecode
# - Interprets bytecode in VM
# - Type checks at runtime
# - Reference counting overhead
# - ~100s of CPU instructions per operation

import dis
dis.dis(add)
"""
  2           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD              # Calls PyNumber_Add (100+ instructions)
              6 RETURN_VALUE
"""

# 2. Dynamic typing overhead
x = 10  # Could be int, float, string, etc.
y = 20
z = x + y  # Runtime type check required

# C equivalent (static typing):
# int x = 10;
# int y = 20;
# int z = x + y;  // Compile-time type checking

# 3. Memory management overhead
# - Reference counting on every operation
# - Garbage collection pauses
# - Heap allocation for most objects

# 4. GIL prevents true parallelism

# When is Python fast enough?
# ✅ I/O-bound applications (network, database, file I/O)
# ✅ Applications with C extensions (NumPy, pandas)
# ✅ Rapid prototyping and development
# ✅ Glue code between systems
# ✅ Scripts and automation

# When Python is too slow:
# ❌ High-frequency trading systems
# ❌ Game engines
# ❌ Operating systems
# ❌ Embedded systems (use MicroPython)
# ❌ Real-time systems

# Solutions for performance:
# 1. Use C extensions (NumPy, pandas)
# 2. PyPy (JIT compilation)
# 3. Cython (compile to C)
# 4. Numba (JIT for numerical code)
# 5. Optimize algorithms first!
```

**Why This Matters:** Understanding performance characteristics helps you choose the right tool for the job and know when optimization is needed.

**Related Concepts:** Profiling (Section 4.7), Performance optimization (Section 8.4)

---

**Q2: How does Python manage memory automatically?**

**A:**

```python
# Python uses two mechanisms:

# 1. Reference Counting (primary)
x = [1, 2, 3]  # refcount = 1
y = x          # refcount = 2
z = x          # refcount = 3
del y          # refcount = 2
del z          # refcount = 1
del x          # refcount = 0 → object deallocated immediately

# Pros:
# - Immediate deallocation (deterministic)
# - Simple to implement
# - Works well for most cases

# Cons:
# - Can't handle cycles
# - Overhead on every assignment

# 2. Cyclic Garbage Collector (secondary)
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create cycle
a = Node(1)
b = Node(2)
a.next = b
b.next = a  # Cycle!

del a, b  # Both refcount = 1 (cycle keeps them alive)
# Cyclic GC will eventually collect them

import gc
gc.collect()  # Forces collection

# How GC finds cycles:
# 1. Track all container objects (list, dict, instances)
# 2. Subtract internal references
# 3. Objects with refcount > 0 are reachable
# 4. Objects with refcount = 0 are garbage

# Performance characteristics:
import time

# Reference counting: O(1) per operation
start = time.time()
for i in range(1000000):
    x = [i]  # Allocated
    del x    # Deallocated immediately
print(f"Refcount: {time.time() - start:.2f}s")

# GC collection: O(n) where n = tracked objects
gc.disable()
objects = []
for i in range(100000):
    objects.append([i])

start = time.time()
gc.collect()
print(f"GC collect: {time.time() - start:.2f}s")

# ✅ GOOD: Let Python manage memory
def process_data():
    large_data = load_large_dataset()
    result = analyze(large_data)
    # large_data automatically deallocated here
    return result

# ❌ BAD: Manual memory management
global_cache = {}  # Never cleared!

def process_with_cache(key):
    if key not in global_cache:
        global_cache[key] = expensive_computation(key)
    return global_cache[key]

# ✅ BETTER: Use weak references or LRU cache
from functools import lru_cache

@lru_cache(maxsize=128)
def process_cached(key):
    return expensive_computation(key)
```

**Why This Matters:** Understanding memory management prevents memory leaks and helps debug performance issues.

---

**Q3: What is bytecode and why should I care?**

**A:**

```python
# Bytecode is the intermediate representation
# Source Code → Bytecode → Machine Code (via PVM)

def example(x):
    return x * 2

# Compile to bytecode
import dis
dis.dis(example)
"""
  2           0 LOAD_FAST                0 (x)
              2 LOAD_CONST               1 (2)
              4 BINARY_MULTIPLY
              6 RETURN_VALUE
"""

# Why bytecode matters:

# 1. Debugging - Understand what Python is actually doing
def mystery_bug(items):
    return [x for x in items if x]

dis.dis(mystery_bug)
# Shows comprehension creates new function scope

# 2. Performance - Some patterns are faster
# Faster:
def fast_lookup(d, key):
    return d[key]  # BINARY_SUBSCR

# Slower:
def slow_lookup(d, key):
    return d.get(key)  # LOAD_METHOD, CALL_METHOD

# 3. Understanding Python internals
# List comprehension:
result = [x * 2 for x in range(10)]

# Is faster than:
result = []
for x in range(10):
    result.append(x * 2)

# Why? Comprehension uses optimized CALL_FUNCTION opcode

# 4. Security - Detect code injection
import ast

code = "import os; os.system('rm -rf /')"
tree = ast.parse(code)
# Inspect AST before execution to detect dangerous operations

# 5. Optimization - Understand what compiler optimizes
# Constant folding:
x = 24 * 60 * 60  # Computed at compile time
# vs
x = hours * minutes * seconds  # Computed at runtime

# When to look at bytecode:
# - Performance debugging
# - Understanding unexpected behavior
# - Learning Python internals
# - Security analysis
# - Verifying optimizations

# Tools for bytecode analysis:
# - dis module: Disassemble bytecode
# - py_compile: Compile to .pyc files
# - importlib: Inspect compiled modules
```

**Why This Matters:** Bytecode understanding helps debug issues, optimize performance, and understand Python's execution model.

**Related Concepts:** CPython architecture (Section 2.1), Performance profiling (Section 4.7)

---

### Interview Questions

**Question 1: Explain the Global Interpreter Lock (GIL) and its impact on performance.**

**Difficulty:** Mid-Level

**SDLC Relevance:** System Design, Performance Optimization

**Answer:**

```python
# The GIL is a mutex that protects access to Python objects
# Only one thread can execute Python bytecode at a time

# Why GIL exists:
# 1. Simplifies CPython implementation
# 2. Makes reference counting thread-safe
# 3. Protects C extensions that aren't thread-safe
# 4. Easier to integrate with C libraries

# Impact on multi-threading:

# ❌ BAD: CPU-bound tasks with threading (GIL limits)
import threading
import time

def cpu_intensive(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

# Two threads (no speedup due to GIL)
start = time.time()
t1 = threading.Thread(target=cpu_intensive, args=(10**7,))
t2 = threading.Thread(target=cpu_intensive, args=(10**7,))
t1.start()
t2.start()
t1.join()
t2.join()
print(f"Threading: {time.time() - start:.2f}s")  # ~2.0s

# ✅ GOOD: Multi-processing for CPU-bound
from multiprocessing import Pool

start = time.time()
with Pool(2) as pool:
    results = pool.map(cpu_intensive, [10**7, 10**7])
print(f"Multiprocessing: {time.time() - start:.2f}s")  # ~1.0s (2x faster!)

# ✅ GOOD: Threading for I/O-bound (GIL released during I/O)
import concurrent.futures
import requests

def fetch_url(url):
    return requests.get(url).status_code

urls = ['http://example.com'] * 20

# Threading works great (GIL released during network I/O)
start = time.time()
with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(fetch_url, urls))
print(f"Threading I/O: {time.time() - start:.2f}s")  # Fast!

# Workarounds for GIL:
# 1. Use multiprocessing.Pool for CPU-bound tasks
# 2. Use async/await for I/O-bound tasks
# 3. Use C extensions that release GIL (NumPy, etc.)
# 4. Use PyPy or other Python implementations
# 5. Future: nogil Python (PEP 703)

# When GIL doesn't matter:
# - I/O-bound applications (web servers, API clients)
# - Single-threaded programs
# - Using C extensions for heavy computation
# - Async I/O patterns
```

**Key Points to Mention:**

- GIL allows only one thread to execute Python bytecode at a time
- Prevents true parallelism for CPU-bound multi-threaded code
- Doesn't affect I/O-bound operations (GIL is released)
- Use multiprocessing for CPU-bound parallel tasks
- Use threading/async for I/O-bound concurrent tasks

**Why This Matters:** Understanding the GIL is critical for designing scalable Python applications and making correct architecture decisions.

**Follow-up Questions:**

- How would you parallelize a CPU-intensive task? (Use multiprocessing)
- When is threading still useful despite the GIL? (I/O-bound operations)
- What's the future of the GIL? (PEP 703 - optional GIL in Python 3.13+)

---

**Question 2: How does Python's garbage collector work, and when would you manually trigger it?**

**Difficulty:** Senior

**SDLC Relevance:** Performance Optimization, Memory Management

**Answer:**

```python
import gc

# Python uses two garbage collection mechanisms:

# 1. Reference Counting (primary, immediate)
x = [1, 2, 3]  # refcount = 1
y = x          # refcount = 2
del y          # refcount = 1
del x          # refcount = 0 → deallocated immediately

# 2. Cyclic GC (secondary, periodic)
# Handles reference cycles that refcounting can't

# Example: Cycle that needs GC
class Node:
    def __init__(self, value):
        self.value = value
        self.ref = None

a = Node(1)
b = Node(2)
a.ref = b
b.ref = a  # Cycle created

del a, b  # Refcount = 1 each (cycle keeps alive)
# Only cyclic GC can collect these

# How cyclic GC works:
# 1. Tracks all container objects (lists, dicts, instances)
# 2. Uses generational collection (3 generations)
# 3. Generation 0: Young objects (frequent collection)
# 4. Generation 1: Survived one GC
# 5. Generation 2: Long-lived objects (infrequent collection)

# GC thresholds
print(gc.get_threshold())  # (700, 10, 10)
# Gen 0: Collect after 700 allocations
# Gen 1: After 10 gen-0 collections
# Gen 2: After 10 gen-1 collections

# Manual GC triggering scenarios:

# ✅ GOOD: After large object deallocation
def process_large_dataset():
    data = load_huge_file()  # 10 GB
    result = analyze(data)
    del data  # Hint to GC
    gc.collect()  # Free memory immediately
    return result

# ✅ GOOD: Before memory-intensive operation
def memory_sensitive_task():
    gc.collect()  # Clean up first
    large_array = allocate_huge_array()
    process(large_array)

# ✅ GOOD: In long-running services
import time

def server_loop():
    iteration = 0
    while True:
        handle_requests()
      
        iteration += 1
        if iteration % 1000 == 0:
            # Periodic manual GC
            gc.collect()

# ✅ GOOD: Testing for memory leaks
def test_memory_leak():
    gc.collect()  # Clean slate
    before = len(gc.get_objects())
  
    # Run code under test
    for _ in range(1000):
        create_and_destroy_objects()
  
    gc.collect()
    after = len(gc.get_objects())
  
    assert after - before < 100, "Memory leak detected!"

# ❌ BAD: Calling gc.collect() frequently
for item in items:
    process(item)
    gc.collect()  # Unnecessary overhead!

# GC tuning for performance:
# Disable GC during performance-critical section
gc.disable()
# ... CPU-intensive work ...
gc.enable()
gc.collect()

# Adjust thresholds for fewer collections
gc.set_threshold(1000, 15, 15)  # Less frequent

# Monitor GC performance
gc.set_debug(gc.DEBUG_STATS)
gc.collect()  # Prints collection statistics
```

**Key Points to Mention:**

- Python uses reference counting + cyclic GC
- Cyclic GC handles reference cycles
- Generational approach (3 generations)
- Manual collection useful after large deallocations
- Can disable GC during performance-critical code
- Monitor GC stats for performance tuning

**Why This Matters:** Understanding GC helps optimize memory usage and performance in production systems.

---

### Key Takeaways

**CPython Architecture:**

- **CPython vs others**: Reference implementation, most widely used, has GIL
- **Execution pipeline**: Source → AST → Bytecode → PVM execution
- **Bytecode**: Intermediate representation, stack-based VM
- **Memory management**: Reference counting + cyclic GC
- **Object representation**: PyObject structure, reference count, type pointer
- **Memory allocators**: Small object pools, arena allocation
- **Garbage collection**: Generational GC (3 generations), handles cycles
- **GIL**: Limits multi-threading for CPU-bound tasks, released during I/O
- **Performance**: Interpretation overhead, dynamic typing costs
- **Optimization**: Constant folding, peephole optimization

**Performance Implications:**

- CPU-bound: Use multiprocessing, not threading
- I/O-bound: Threading or async/await work well
- Memory: Understand reference counting, use weak references for caches
- Profiling: Use dis, tracemalloc, memory_profiler

---

## 2.2 Type System & Type Hints

**SDLC Phase:** Development, Code Quality

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (API contracts)
- [X] Development
- [X] Testing (type checking)
- [ ] Deployment
- [X] Maintenance (refactoring safety)

Python is dynamically typed, but type hints (PEP 484) enable optional static type checking for better code quality and IDE support.

### Dynamic Typing

#### Duck Typing

```python
# "If it walks like a duck and quacks like a duck, it's a duck"

# Duck typing: Type determined by behavior, not inheritance
class Duck:
    def quack(self):
        return "Quack!"
  
    def fly(self):
        return "Flying!"

class Person:
    def quack(self):
        return "I'm quacking like a duck!"
  
    def fly(self):
        return "I'm flapping my arms!"

# Function accepts any object with quack() and fly()
def make_it_quack_and_fly(thing):
    print(thing.quack())
    print(thing.fly())

duck = Duck()
person = Person()

make_it_quack_and_fly(duck)    # Works
make_it_quack_and_fly(person)  # Also works!

# ✅ GOOD: Duck typing for flexibility
def process_file(file_obj):
    # Works with real files, StringIO, BytesIO, etc.
    data = file_obj.read()
    return data

# Works with any file-like object
import io
process_file(open('file.txt'))
process_file(io.StringIO("data"))

# ❌ BAD: Checking type explicitly
def bad_process(file_obj):
    if not isinstance(file_obj, io.IOBase):
        raise TypeError("Must be a file")
    return file_obj.read()
# Rejects valid file-like objects!
```

#### Runtime Type Checking

```python
# Check types at runtime
def add(a, b):
    # Manual type checking
    if not isinstance(a, (int, float)):
        raise TypeError(f"a must be numeric, got {type(a)}")
    if not isinstance(b, (int, float)):
        raise TypeError(f"b must be numeric, got {type(b)}")
    return a + b

# isinstance() - check if object is instance of class
x = [1, 2, 3]
print(isinstance(x, list))  # True
print(isinstance(x, (list, tuple)))  # True (multiple types)
print(isinstance(x, str))  # False

# issubclass() - check inheritance
print(issubclass(bool, int))  # True (bool inherits from int)
print(issubclass(list, object))  # True (all classes inherit from object)

# type() - get exact type
print(type(x))  # <class 'list'>
print(type(x) == list)  # True
print(type(x) is list)  # True

# ⚠️ CAUTION: type() vs isinstance()
class Animal:
    pass

class Dog(Animal):
    pass

dog = Dog()
print(type(dog) == Animal)  # False (exact type check)
print(isinstance(dog, Animal))  # True (inheritance aware)

# ✅ GOOD: Use isinstance() for type checking
def process_items(items):
    if not isinstance(items, (list, tuple)):
        raise TypeError("items must be list or tuple")
    return [item * 2 for item in items]

# Type introspection
import inspect

def func(x: int, y: str) -> bool:
    return len(y) > x

# Get type hints
hints = func.__annotations__
print(hints)  # {'x': <class 'int'>, 'y': <class 'str'>, 'return': <class 'bool'>}

# Get signature
sig = inspect.signature(func)
print(sig.parameters['x'].annotation)  # <class 'int'>
```

### Type Hints (PEP 484+)

#### Basic Type Annotations

```python
# Function annotations (PEP 484)
def greet(name: str) -> str:
    return f"Hello, {name}!"

# Variable annotations (PEP 526)
age: int = 30
name: str = "Alice"
scores: list = [85, 90, 78]

# Type hints don't enforce types at runtime!
def add(a: int, b: int) -> int:
    return a + b

result = add("hello", "world")  # No error! Returns "helloworld"
print(result)  # "helloworld"

# Use mypy for static checking:
# $ mypy script.py
# error: Argument 1 to "add" has incompatible type "str"; expected "int"

# ✅ GOOD: Type hints for documentation and tooling
def calculate_discount(price: float, discount_percent: float) -> float:
    """Calculate discounted price."""
    return price * (1 - discount_percent / 100)

# IDE shows parameter types and return type
result = calculate_discount(100.0, 10.0)  # IDE: result is float
```

#### Generic Types

```python
from typing import List, Dict, Set, Tuple, Optional, Union, Any

# List with element type
def process_numbers(numbers: List[int]) -> int:
    return sum(numbers)

# Dict with key and value types
def count_words(text: str) -> Dict[str, int]:
    words = text.split()
    return {word: words.count(word) for word in set(words)}

# Set with element type
def unique_items(items: List[str]) -> Set[str]:
    return set(items)

# Tuple with fixed types
def get_user() -> Tuple[str, int, str]:
    return ("Alice", 30, "alice@example.com")

# Optional (value or None)
def find_user(user_id: int) -> Optional[Dict[str, str]]:
    # Returns dict or None
    user = database.get(user_id)
    return user if user else None

# Union (one of multiple types)
def process_id(id: Union[int, str]) -> str:
    return str(id)

# Any (any type)
def flexible_function(value: Any) -> Any:
    return value

# ✅ GOOD: Specific types
def get_config() -> Dict[str, Union[str, int, bool]]:
    return {
        "host": "localhost",
        "port": 5432,
        "debug": True,
    }

# Python 3.9+: Built-in generics
def modern_types(items: list[int]) -> dict[str, int]:
    return {str(i): i for i in items}
```

#### Advanced Type Hints

```python
from typing import (
    Callable, TypeVar, Generic, Protocol, 
    Literal, Final, TypedDict, Annotated
)

# Callable - function types
def apply_operation(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

def add(x: int, y: int) -> int:
    return x + y

result = apply_operation(add, 10, 20)  # Type safe!

# TypeVar - generic type variables
T = TypeVar('T')

def first(items: List[T]) -> T:
    return items[0]

# Works with any type
numbers: List[int] = [1, 2, 3]
first_num: int = first(numbers)  # Type checker knows result is int

strings: List[str] = ["a", "b", "c"]
first_str: str = first(strings)  # Result is str

# Generic classes
class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: List[T] = []
  
    def push(self, item: T) -> None:
        self._items.append(item)
  
    def pop(self) -> T:
        return self._items.pop()

# Type-safe stack
int_stack: Stack[int] = Stack()
int_stack.push(10)
value: int = int_stack.pop()

# Protocol - structural subtyping (duck typing with types)
class Drawable(Protocol):
    def draw(self) -> None:
        ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(shape: Drawable) -> None:
    shape.draw()

# Both work without inheriting from Drawable
render(Circle())
render(Square())

# Literal - specific values
from typing import Literal

def set_mode(mode: Literal["read", "write", "append"]) -> None:
    print(f"Mode: {mode}")

set_mode("read")  # OK
# set_mode("delete")  # Type error!

# Final - constants
MAX_SIZE: Final[int] = 100
# MAX_SIZE = 200  # Type error!

# TypedDict - typed dictionaries
class User(TypedDict):
    name: str
    age: int
    email: str

def create_user(user: User) -> None:
    print(f"Creating user: {user['name']}")

user: User = {
    "name": "Alice",
    "age": 30,
    "email": "alice@example.com"
}
create_user(user)

# Annotated - metadata
from typing import Annotated

def process(value: Annotated[int, "Must be positive"]) -> int:
    return value * 2
```

#### Type Aliases

```python
from typing import Union, List, Dict

# Simple type alias
UserId = int
Username = str

def get_user(user_id: UserId) -> Username:
    return database.query(user_id)

# Complex type alias
JsonDict = Dict[str, Union[str, int, float, bool, None, List, Dict]]

def parse_json(data: str) -> JsonDict:
    import json
    return json.loads(data)

# Generic type alias
from typing import TypeVar, List

T = TypeVar('T')
Vector = List[T]

def process_vector(vec: Vector[int]) -> int:
    return sum(vec)

# NewType - distinct types
from typing import NewType

UserId = NewType('UserId', int)
OrderId = NewType('OrderId', int)

def get_user(user_id: UserId) -> str:
    return f"User {user_id}"

def get_order(order_id: OrderId) -> str:
    return f"Order {order_id}"

user_id = UserId(123)
order_id = OrderId(456)

get_user(user_id)  # OK
# get_user(order_id)  # Type error! Different types
# get_user(123)  # Type error! Must use UserId()
```

### Static Type Checking

#### mypy Fundamentals

```python
# Install mypy: pip install mypy
# Run: mypy script.py

# Example with type errors:
def add(a: int, b: int) -> int:
    return a + b

result = add(10, "20")  # Type error!
# $ mypy script.py
# error: Argument 2 to "add" has incompatible type "str"; expected "int"

# Type inference
def inferred_types():
    x = 10  # mypy infers: int
    y = "hello"  # mypy infers: str
    z = [1, 2, 3]  # mypy infers: List[int]
    return x, y, z

# Gradual typing - mix typed and untyped code
def untyped_function(x, y):  # No annotations
    return x + y

def typed_function(x: int, y: int) -> int:
    return untyped_function(x, y)  # mypy allows this

# Type narrowing
from typing import Union

def process_value(value: Union[int, str]) -> str:
    if isinstance(value, int):
        # mypy knows value is int here
        return str(value * 2)
    else:
        # mypy knows value is str here
        return value.upper()

# Optional type narrowing
from typing import Optional

def greet(name: Optional[str]) -> str:
    if name is None:
        return "Hello, stranger!"
    # mypy knows name is str here (not None)
    return f"Hello, {name}!"

# Type guards
def is_string_list(items: List[Union[int, str]]) -> bool:
    return all(isinstance(item, str) for item in items)

def process_items(items: List[Union[int, str]]) -> None:
    if is_string_list(items):
        # mypy still sees items as List[Union[int, str]]
        # Type guards needed for narrowing
        pass
```

#### mypy Configuration

```python
# mypy.ini configuration file
"""
[mypy]
# Global options
python_version = 3.11
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True

# Per-module options
[mypy-third_party_library.*]
ignore_missing_imports = True

[mypy-tests.*]
disallow_untyped_defs = False
"""

# Inline configuration
# type: ignore - Suppress error on line
result = add(10, "20")  # type: ignore

# type: ignore[error-code] - Suppress specific error
result = add(10, "20")  # type: ignore[arg-type]

# Ignore entire file
# mypy: ignore-errors

# Enable strict mode in file
# mypy: strict

# ✅ GOOD: Enable strict mode for new projects
# mypy.ini:
"""
[mypy]
strict = True
"""

# ✅ GOOD: Gradual adoption
# Start with: warn_unused_ignores = True
# Then: check_untyped_defs = True
# Finally: disallow_untyped_defs = True
```

#### Variance

```python
from typing import TypeVar, Generic, List

# Covariance: T can be substituted with subtype
# List[Dog] is-a List[Animal]? NO (List is invariant)

class Animal:
    pass

class Dog(Animal):
    pass

class Cat(Animal):
    pass

# Invariant (default for mutable containers)
def process_animals(animals: List[Animal]) -> None:
    animals.append(Cat())  # Might break if List[Dog] passed!

dogs: List[Dog] = [Dog(), Dog()]
# process_animals(dogs)  # Type error! List is invariant

# Covariant (read-only)
from typing import Sequence  # Immutable sequence

def process_readonly(animals: Sequence[Animal]) -> None:
    for animal in animals:
        print(animal)

dogs_seq: Sequence[Dog] = [Dog(), Dog()]
process_readonly(dogs_seq)  # OK! Sequence is covariant

# Contravariant (write-only)
T_contra = TypeVar('T_contra', contravariant=True)

class Printer(Generic[T_contra]):
    def print(self, item: T_contra) -> None:
        print(item)

animal_printer: Printer[Animal] = Printer()
dog_printer: Printer[Dog] = animal_printer  # OK! Contravariant

# Summary:
# Covariant: Producer[T] - can read T
# Contravariant: Consumer[T] - can write T
# Invariant: Storage[T] - can read and write T
```

### Runtime Type Checking

#### Pydantic Models

```python
# Install: pip install pydantic
from pydantic import BaseModel, Field, validator, root_validator
from typing import Optional, List
from datetime import datetime

# Basic model
class User(BaseModel):
    id: int
    name: str
    email: str
    age: int
    created_at: datetime = Field(default_factory=datetime.now)

# Automatic validation
try:
    user = User(
        id="not-an-int",  # Type error!
        name="Alice",
        email="invalid-email",
        age=30
    )
except ValueError as e:
    print(e)
    # validation error for User
    # id: value is not a valid integer

# Valid user
user = User(
    id=1,
    name="Alice",
    email="alice@example.com",
    age=30
)

# Access attributes
print(user.name)  # "Alice"
print(user.dict())  # Convert to dict
print(user.json())  # Convert to JSON

# Custom validators
class StrictUser(BaseModel):
    name: str
    age: int
    email: str
  
    @validator('age')
    def age_must_be_positive(cls, v):
        if v < 0:
            raise ValueError('age must be positive')
        return v
  
    @validator('email')
    def email_must_be_valid(cls, v):
        if '@' not in v:
            raise ValueError('invalid email')
        return v

# Root validator (validate entire model)
class OrderModel(BaseModel):
    items: List[str]
    total: float
    discount: float = 0.0
  
    @root_validator
    def check_discount(cls, values):
        total = values.get('total')
        discount = values.get('discount')
        if discount > total:
            raise ValueError('discount cannot exceed total')
        return values

# Config class
class ConfigurableModel(BaseModel):
    name: str
    value: int
  
    class Config:
        # Allow extra fields
        extra = 'allow'
        # Validate on assignment
        validate_assignment = True
        # Use enum values
        use_enum_values = True

# ✅ GOOD: API request/response models
class CreateUserRequest(BaseModel):
    name: str
    email: str
    age: int

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime

def create_user(request: CreateUserRequest) -> UserResponse:
    # Automatic validation!
    user_id = save_to_database(request.dict())
    return UserResponse(
        id=user_id,
        name=request.name,
        email=request.email,
        created_at=datetime.now()
    )
```

#### Dataclasses with Validation

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Product:
    name: str
    price: float
    quantity: int = 0
    tags: List[str] = field(default_factory=list)
  
    def __post_init__(self):
        # Validation after initialization
        if self.price < 0:
            raise ValueError("price must be non-negative")
        if self.quantity < 0:
            raise ValueError("quantity must be non-negative")

# Create product
product = Product(name="Widget", price=19.99, quantity=100)

try:
    bad_product = Product(name="Bad", price=-10)
except ValueError as e:
    print(e)  # "price must be non-negative"

# ✅ GOOD: Combine dataclass with property for validation
@dataclass
class ValidatedProduct:
    name: str
    _price: float = field(repr=False)
  
    @property
    def price(self) -> float:
        return self._price
  
    @price.setter
    def price(self, value: float) -> None:
        if value < 0:
            raise ValueError("price must be non-negative")
        self._price = value
  
    def __post_init__(self):
        # Validate initial price
        self.price = self._price
```

#### attrs Library

```python
# Install: pip install attrs
import attr
from attr import validators

@attr.s(auto_attribs=True)
class User:
    name: str = attr.ib(validator=validators.instance_of(str))
    age: int = attr.ib(validator=[
        validators.instance_of(int),
        validators.ge(0),  # >= 0
        validators.le(150)  # <= 150
    ])
    email: str = attr.ib()
  
    @email.validator
    def check_email(self, attribute, value):
        if '@' not in value:
            raise ValueError("Invalid email")

# Create user with validation
user = User(name="Alice", age=30, email="alice@example.com")

try:
    bad_user = User(name="Bob", age=-5, email="invalid")
except ValueError as e:
    print(e)

# ✅ GOOD: attrs for data classes with validation
@attr.s(auto_attribs=True, frozen=True)  # Immutable
class Point:
    x: float
    y: float
  
    @classmethod
    def origin(cls):
        return cls(0.0, 0.0)
```

### Frequently Asked Questions

**Q1: Should I use type hints in all my Python code?**

**A:**

```python
# Type hints are optional but recommended for:

# ✅ GOOD: Library/framework code
def calculate_tax(amount: float, rate: float) -> float:
    """Calculate tax amount."""
    return amount * rate

# ✅ GOOD: API boundaries
from pydantic import BaseModel

class UserRequest(BaseModel):
    name: str
    email: str

def create_user(request: UserRequest) -> dict:
    return {"id": 123, "name": request.name}

# ✅ GOOD: Complex business logic
from typing import List, Dict, Optional

def analyze_sales(
    sales: List[Dict[str, float]],
    threshold: float
) -> Optional[Dict[str, float]]:
    # Clear what inputs/outputs are
    pass

# ⚠️ SKIP: Simple scripts
# script.py
import sys
filename = sys.argv[1]
with open(filename) as f:
    print(len(f.read()))

# ⚠️ SKIP: Exploratory code/prototypes
def quick_test():
    data = fetch_data()
    result = process(data)
    return result

# Guidelines:
# 1. Start with public APIs
# 2. Add types to complex functions
# 3. Use gradual typing (add types incrementally)
# 4. Configure mypy for your needs

# Benefits:
# - Better IDE support (autocomplete, refactoring)
# - Catch bugs before runtime
# - Self-documenting code
# - Easier refactoring

# Costs:
# - More verbose code
# - Learning curve
# - Slower development initially
```

**Why This Matters:** Type hints improve code quality and maintainability without sacrificing Python's flexibility.

---

**Q2: What's the difference between runtime and static type checking?**

**A:**

```python
# Static type checking (mypy) - before code runs
# Runtime type checking (pydantic) - during execution

# Static type checking (mypy):
def add(a: int, b: int) -> int:
    return a + b

# Check with: mypy script.py
result = add(10, "20")  # mypy error: str is not int
# Code never runs if mypy fails

# Pros:
# - Catches errors before deployment
# - No runtime overhead
# - Works with existing code

# Cons:
# - Optional (code runs anyway without mypy)
# - May miss dynamic errors
# - Requires tool setup

# Runtime type checking (pydantic):
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

try:
    user = User(name="Alice", age="thirty")  # Raises error!
except ValueError as e:
    print("Validation failed!")

# Pros:
# - Guaranteed validation
# - Catches runtime issues
# - Works with external data

# Cons:
# - Runtime overhead
# - More verbose
# - Errors happen late (at runtime)

# ✅ GOOD: Use both together
# Static checking for development
def process_user(user: User) -> dict:  # mypy checks this
    return {"name": user.name, "age": user.age}

# Runtime checking for external data
class APIRequest(BaseModel):  # Validates JSON
    user_id: int
    action: str

def handle_request(data: dict) -> dict:
    request = APIRequest(**data)  # Runtime validation
    return process_request(request)  # Static type checking

# Decision guide:
# - Internal code: Static typing (mypy)
# - External data: Runtime validation (pydantic)
# - Critical logic: Both!
```

**Why This Matters:** Choosing the right validation strategy prevents bugs and improves reliability.

---

**Q3: How do I type hint a function that accepts any callable?**

**A:**

```python
from typing import Callable, Any, TypeVar

# Basic callable
def execute(func: Callable) -> Any:
    return func()

# Callable with specific signature
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

def add(x: int, y: int) -> int:
    return x + y

result = apply(add, 10, 20)  # Type safe!

# Callable with variable arguments
def retry(func: Callable[..., Any], *args, **kwargs) -> Any:
    for attempt in range(3):
        try:
            return func(*args, **kwargs)
        except Exception:
            continue

# Generic callable with TypeVar
T = TypeVar('T')

def call_twice(func: Callable[[T], T], value: T) -> T:
    return func(func(value))

def double(x: int) -> int:
    return x * 2

result = call_twice(double, 5)  # Returns 20, type is int

# ✅ GOOD: Protocol for complex callables
from typing import Protocol

class Transformer(Protocol):
    def __call__(self, data: str) -> str:
        ...

def process(transformer: Transformer, data: str) -> str:
    return transformer(data)

# Any callable that matches the signature works
def uppercase(s: str) -> str:
    return s.upper()

process(uppercase, "hello")  # OK!

# Callable class
class Multiplier:
    def __init__(self, factor: int):
        self.factor = factor
  
    def __call__(self, value: int) -> int:
        return value * self.factor

multiply_by_2: Callable[[int], int] = Multiplier(2)
```

**Why This Matters:** Properly typing callables enables type-safe higher-order functions and callbacks.

---

### Interview Questions

**Question 1: Explain the difference between List[int] and list in type hints.**

**Difficulty:** Junior

**SDLC Relevance:** Development, Code Quality

**Answer:**

```python
# list - generic list type (no element type specified)
def process_items(items: list) -> int:
    return len(items)

# Can pass any list
process_items([1, 2, 3])  # OK
process_items(["a", "b"])  # OK
process_items([1, "a", None])  # OK

# List[int] - list of integers specifically
from typing import List

def sum_numbers(numbers: List[int]) -> int:
    return sum(numbers)

# Type checker validates element types
sum_numbers([1, 2, 3])  # OK
# sum_numbers(["a", "b"])  # mypy error!

# Python 3.9+: Use built-in list
def modern_sum(numbers: list[int]) -> int:
    return sum(numbers)

# Why it matters:

# ✅ GOOD: Specific types catch bugs
def calculate_average(numbers: List[float]) -> float:
    return sum(numbers) / len(numbers)

# mypy catches this error:
# calculate_average(["1", "2", "3"])  # Error!

# ❌ BAD: Generic list allows anything
def bad_average(numbers: list) -> float:
    return sum(numbers) / len(numbers)

# mypy doesn't catch this:
bad_average(["1", "2", "3"])  # Runtime TypeError!

# Guidelines:
# - Use List[T] for APIs and libraries
# - Use list for simple internal code
# - Python 3.9+: Use list[int] instead of List[int]
```

**Key Points:**

- `list` accepts any list, no element type checking
- `List[int]` ensures all elements are integers
- Python 3.9+ supports `list[int]` syntax (preferred)
- Specific types catch more bugs at static analysis time

**Why This Matters:** Proper generic types prevent runtime errors and improve code documentation.

---

**Question 2: How do you handle functions that return different types based on input?**

**Difficulty:** Mid-Level

**SDLC Relevance:** Development, API Design

**Answer:**

```python
from typing import overload, Union, Literal

# Problem: Return type depends on parameter
def parse_value(value: str, as_int: bool) -> ???:
    if as_int:
        return int(value)
    else:
        return float(value)

# Solution 1: Union (less precise)
def parse_union(value: str, as_int: bool) -> Union[int, float]:
    if as_int:
        return int(value)
    return float(value)

result = parse_union("42", True)  # Type: Union[int, float]
# Caller must handle both types

# Solution 2: @overload (precise)
@overload
def parse_overload(value: str, as_int: Literal[True]) -> int:
    ...

@overload
def parse_overload(value: str, as_int: Literal[False]) -> float:
    ...

def parse_overload(value: str, as_int: bool) -> Union[int, float]:
    if as_int:
        return int(value)
    return float(value)

result1 = parse_overload("42", True)  # Type: int
result2 = parse_overload("3.14", False)  # Type: float

# Solution 3: Generics with TypeVar
T = TypeVar('T', int, float)

def parse_generic(value: str, return_type: type[T]) -> T:
    return return_type(value)

result_int = parse_generic("42", int)  # Type: int
result_float = parse_generic("3.14", float)  # Type: float

# Real-world example: open()
@overload
def open(file: str, mode: Literal['r']) -> TextIO:
    ...

@overload
def open(file: str, mode: Literal['rb']) -> BinaryIO:
    ...

def open(file: str, mode: str) -> Union[TextIO, BinaryIO]:
    # Implementation
    pass

# ✅ GOOD: Use overload for complex return types
from typing import overload, Literal

@overload
def fetch_data(endpoint: str, parse: Literal[True]) -> dict:
    ...

@overload  
def fetch_data(endpoint: str, parse: Literal[False]) -> str:
    ...

def fetch_data(endpoint: str, parse: bool = True) -> Union[dict, str]:
    response = make_request(endpoint)
    if parse:
        return json.loads(response)
    return response
```

**Key Points:**

- Use `Union` for simple cases
- Use `@overload` for precise type information
- `Literal` types for specific values
- Overloads help IDEs provide better autocomplete

**Why This Matters:** Accurate return types improve type safety and developer experience.

---

### Key Takeaways

**Type System & Type Hints:**

- **Dynamic typing**: Types determined at runtime, duck typing enabled
- **Type hints**: Optional annotations for static checking (PEP 484)
- **Generic types**: List[T], Dict[K,V], Optional[T], Union[A,B]
- **Advanced types**: TypeVar, Generic, Protocol, Literal, Final
- **Static checking**: mypy validates types before runtime
- **Runtime checking**: pydantic, attrs validate at execution
- **Variance**: Covariant (read), contravariant (write), invariant (both)
- **@overload**: Multiple function signatures for different inputs
- **Gradual typing**: Mix typed and untyped code incrementally

**Best Practices:**

- Type hint public APIs and complex functions
- Use specific types (List[int] not list) for better checking
- Combine static (mypy) and runtime (pydantic) validation
- Start with strict=False in mypy, increase gradually
- Use Protocol for duck typing with type safety

---

(Continuing with Section 2.3 - Iterators and Generators...)

## 2.3 Iterators and Generators

**SDLC Phase:** Development, Performance Optimization

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (memory efficiency)
- [ ] Testing
- [ ] Deployment
- [X] Maintenance (performance tuning)

Iterators and generators enable memory-efficient data processing through lazy evaluation.

### Iterator Protocol

#### Understanding Iterators

```python
# Iterator protocol: __iter__() and __next__()

# Iterable: Object that can return an iterator
# Iterator: Object that produces values one at a time

# Built-in iterables
numbers = [1, 2, 3]  # List is iterable
iterator = iter(numbers)  # Get iterator

# Manual iteration
print(next(iterator))  # 1
print(next(iterator))  # 2
print(next(iterator))  # 3
# next(iterator)  # StopIteration!

# for loop uses iterators internally
for num in numbers:
    print(num)

# Equivalent to:
iterator = iter(numbers)
while True:
    try:
        num = next(iterator)
        print(num)
    except StopIteration:
        break

# Check if iterable
from collections.abc import Iterable, Iterator

print(isinstance([1, 2, 3], Iterable))  # True
print(isinstance(iter([1, 2, 3]), Iterator))  # True
print(isinstance([1, 2, 3], Iterator))  # False (list is iterable, not iterator)
```

#### Custom Iterators

```python
# Implement iterator protocol
class Countdown:
    def __init__(self, start):
        self.current = start
  
    def __iter__(self):
        return self  # Return self (iterator object)
  
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# Use custom iterator
for num in Countdown(5):
    print(num)  # 5, 4, 3, 2, 1

# Separating iterable and iterator
class Range:
    def __init__(self, start, end):
        self.start = start
        self.end = end
  
    def __iter__(self):
        return RangeIterator(self.start, self.end)

class RangeIterator:
    def __init__(self, start, end):
        self.current = start
        self.end = end
  
    def __iter__(self):
        return self
  
    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# Can create multiple independent iterators
r = Range(0, 5)
it1 = iter(r)
it2 = iter(r)

print(next(it1))  # 0
print(next(it1))  # 1
print(next(it2))  # 0 (independent iterator!)

# ✅ GOOD: File iterator
class FileLineIterator:
    def __init__(self, filename):
        self.filename = filename
        self.file = None
  
    def __iter__(self):
        self.file = open(self.filename)
        return self
  
    def __next__(self):
        line = self.file.readline()
        if not line:
            self.file.close()
            raise StopIteration
        return line.strip()

# Memory efficient file reading
for line in FileLineIterator('large_file.txt'):
    process(line)  # Doesn't load entire file
```

#### Iterator Exhaustion

```python
# Iterators can only be used once
numbers = [1, 2, 3, 4, 5]
iterator = iter(numbers)

# First consumption
result1 = list(iterator)  # [1, 2, 3, 4, 5]

# Second consumption
result2 = list(iterator)  # [] - Iterator exhausted!

# ✅ GOOD: Create new iterator for each use
def process_data(data):
    # First pass
    for item in data:
        validate(item)
  
    # Second pass - works if data is iterable
    for item in data:
        process(item)

process_data([1, 2, 3])  # Works (list is iterable)

# ❌ BAD: Passing iterator instead of iterable
iterator = iter([1, 2, 3])
process_data(iterator)  # Second pass gets nothing!

# ✅ GOOD: Convert to list if multiple passes needed
def safe_process(iterator):
    data = list(iterator)  # Materialize once
    # Now can iterate multiple times
    for item in data:
        validate(item)
    for item in data:
        process(item)
```

### Generators

#### Generator Functions

```python
# Generator function uses yield
def countdown(n):
    while n > 0:
        yield n
        n -= 1

# Returns generator object (iterator)
gen = countdown(5)
print(type(gen))  # <class 'generator'>

# Use like any iterator
for num in gen:
    print(num)  # 5, 4, 3, 2, 1

# Generator vs function
def regular_function():
    results = []
    for i in range(1000000):
        results.append(i ** 2)
    return results  # Returns entire list

def generator_function():
    for i in range(1000000):
        yield i ** 2  # Yields one value at a time

# Memory comparison
import sys
regular = regular_function()  # ~37 MB
print(sys.getsizeof(regular))

generator = generator_function()  # ~200 bytes!
print(sys.getsizeof(generator))

# ✅ GOOD: Generator for large datasets
def read_large_file(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()  # Lazy evaluation

# Process without loading entire file
for line in read_large_file('huge_file.txt'):
    if 'ERROR' in line:
        print(line)

# Multiple yields
def fibonacci(n):
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

print(list(fibonacci(10)))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Generator state preservation
def counter():
    count = 0
    while True:
        count += 1
        yield count

c = counter()
print(next(c))  # 1
print(next(c))  # 2
print(next(c))  # 3
# State persists between calls!
```

#### Generator Expressions

```python
# Generator expression (like list comprehension with ())
squares_list = [x**2 for x in range(1000000)]  # List: ~37 MB
squares_gen = (x**2 for x in range(1000000))   # Generator: ~200 bytes

# Use generator expression
print(next(squares_gen))  # 0
print(next(squares_gen))  # 1

# ✅ GOOD: Generator in sum/max/min
total = sum(x**2 for x in range(1000000))  # Memory efficient

# ✅ GOOD: Generator in any/all
has_even = any(x % 2 == 0 for x in range(1000000))

# ✅ GOOD: Pipeline with generators
def process_log_file(filename):
    # Pipeline: read → filter → transform → consume
    lines = (line.strip() for line in open(filename))
    errors = (line for line in lines if 'ERROR' in line)
    timestamps = (line.split()[0] for line in errors)
    return list(timestamps)

# Each step is lazy, memory efficient

# ❌ BAD: Forcing evaluation
all_squares = list(x**2 for x in range(1000000))  # Defeats purpose!

# ✅ GOOD: Keep as generator
for square in (x**2 for x in range(1000000)):
    if square > 1000:
        break  # Can stop early without computing rest
```

#### Generator Methods

```python
# send() - Send value to generator
def echo():
    while True:
        value = yield
        print(f"Received: {value}")

gen = echo()
next(gen)  # Prime the generator
gen.send("Hello")  # Received: Hello
gen.send("World")  # Received: World

# throw() - Throw exception into generator
def resilient_generator():
    try:
        while True:
            value = yield
            print(f"Processing: {value}")
    except ValueError:
        print("Handling error")

gen = resilient_generator()
next(gen)
gen.send(10)  # Processing: 10
gen.throw(ValueError, "Bad value")  # Handling error

# close() - Stop generator
def endless():
    while True:
        yield "value"

gen = endless()
print(next(gen))  # "value"
gen.close()
# next(gen)  # StopIteration

# Coroutines (pre-async/await)
def average_coroutine():
    total = 0
    count = 0
    average = None
  
    while True:
        value = yield average
        total += value
        count += 1
        average = total / count

avg = average_coroutine()
next(avg)  # Prime
print(avg.send(10))  # 10.0
print(avg.send(20))  # 15.0
print(avg.send(30))  # 20.0
```

#### yield from

```python
# yield from - delegate to sub-generator

# Without yield from
def chain(*iterables):
    for iterable in iterables:
        for item in iterable:
            yield item

# With yield from (cleaner)
def chain_from(*iterables):
    for iterable in iterables:
        yield from iterable

# Usage
result = list(chain_from([1, 2], [3, 4], [5, 6]))
print(result)  # [1, 2, 3, 4, 5, 6]

# Recursive generators
def flatten(nested_list):
    for item in nested_list:
        if isinstance(item, list):
            yield from flatten(item)  # Recursive!
        else:
            yield item

nested = [1, [2, 3, [4, 5]], 6, [7, [8, 9]]]
flat = list(flatten(nested))
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# yield from with send()
def delegator():
    yield from another_generator()

def another_generator():
    value = yield "Ready"
    yield f"Got {value}"

gen = delegator()
print(next(gen))  # "Ready"
print(gen.send(42))  # "Got 42"
```

#### Generator Pipeline Patterns

```python
# Build processing pipelines with generators

# Step 1: Data source
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()

# Step 2: Filter
def filter_errors(lines):
    for line in lines:
        if 'ERROR' in line:
            yield line

# Step 3: Transform
def extract_timestamp(lines):
    for line in lines:
        parts = line.split()
        if parts:
            yield parts[0]

# Step 4: Consume
def unique_timestamps(timestamps):
    seen = set()
    for ts in timestamps:
        if ts not in seen:
            seen.add(ts)
            yield ts

# Build pipeline
pipeline = unique_timestamps(
    extract_timestamp(
        filter_errors(
            read_lines('server.log')
        )
    )
)

# Execute pipeline
for timestamp in pipeline:
    print(timestamp)

# ✅ GOOD: Reusable pipeline components
class Pipeline:
    def __init__(self, source):
        self.source = source
  
    def filter(self, predicate):
        return Pipeline(item for item in self.source if predicate(item))
  
    def map(self, func):
        return Pipeline(func(item) for item in self.source)
  
    def take(self, n):
        def _take():
            for i, item in enumerate(self.source):
                if i >= n:
                    break
                yield item
        return Pipeline(_take())
  
    def collect(self):
        return list(self.source)

# Usage
result = (Pipeline(range(100))
    .filter(lambda x: x % 2 == 0)
    .map(lambda x: x ** 2)
    .take(5)
    .collect())

print(result)  # [0, 4, 16, 36, 64]
```

### Itertools Module

#### Infinite Iterators

```python
import itertools

# count() - infinite counter
counter = itertools.count(start=10, step=2)
print(next(counter))  # 10
print(next(counter))  # 12
print(next(counter))  # 14

# Take first n elements
first_five = list(itertools.islice(counter, 5))

# cycle() - repeat sequence infinitely
colors = itertools.cycle(['red', 'green', 'blue'])
print(next(colors))  # 'red'
print(next(colors))  # 'green'
print(next(colors))  # 'blue'
print(next(colors))  # 'red' (cycles back)

# repeat() - repeat value
repeater = itertools.repeat('A', times=3)
print(list(repeater))  # ['A', 'A', 'A']

# ✅ GOOD: Generate test data
def generate_test_users(n):
    user_ids = itertools.count(1)
    names = itertools.cycle(['Alice', 'Bob', 'Charlie'])
  
    for _ in range(n):
        yield {
            'id': next(user_ids),
            'name': next(names)
        }

users = list(generate_test_users(5))
# [{'id': 1, 'name': 'Alice'}, {'id': 2, 'name': 'Bob'}, ...]
```

#### Terminating Iterators

```python
import itertools

# chain() - concatenate iterables
combined = itertools.chain([1, 2], [3, 4], [5, 6])
print(list(combined))  # [1, 2, 3, 4, 5, 6]

# chain.from_iterable() - flatten one level
nested = [[1, 2], [3, 4], [5, 6]]
flat = itertools.chain.from_iterable(nested)
print(list(flat))  # [1, 2, 3, 4, 5, 6]

# compress() - filter by selector
data = ['A', 'B', 'C', 'D']
selectors = [1, 0, 1, 0]  # Truthy values
result = itertools.compress(data, selectors)
print(list(result))  # ['A', 'C']

# dropwhile() - drop until condition fails
numbers = [1, 3, 5, 8, 10, 12]
result = itertools.dropwhile(lambda x: x < 5, numbers)
print(list(result))  # [5, 8, 10, 12]

# takewhile() - take until condition fails
result = itertools.takewhile(lambda x: x < 10, numbers)
print(list(result))  # [1, 3, 5, 8]

# filterfalse() - opposite of filter
numbers = range(10)
odds = itertools.filterfalse(lambda x: x % 2 == 0, numbers)
print(list(odds))  # [1, 3, 5, 7, 9]

# islice() - slice iterator
numbers = range(100)
subset = itertools.islice(numbers, 5, 10)  # [5:10]
print(list(subset))  # [5, 6, 7, 8, 9]

# groupby() - group consecutive items
data = [1, 1, 2, 2, 2, 3, 1, 1]
for key, group in itertools.groupby(data):
    print(f"{key}: {list(group)}")
# 1: [1, 1]
# 2: [2, 2, 2]
# 3: [3]
# 1: [1, 1]

# ✅ GOOD: Group by key function
words = ['apple', 'apricot', 'banana', 'blueberry', 'cherry']
words.sort()  # Must be sorted first!
for letter, group in itertools.groupby(words, key=lambda x: x[0]):
    print(f"{letter}: {list(group)}")
# a: ['apple', 'apricot']
# b: ['banana', 'blueberry']
# c: ['cherry']
```

#### Combinatoric Iterators

```python
import itertools

# product() - Cartesian product
colors = ['red', 'blue']
sizes = ['S', 'M', 'L']
products = itertools.product(colors, sizes)
print(list(products))
# [('red', 'S'), ('red', 'M'), ('red', 'L'),
#  ('blue', 'S'), ('blue', 'M'), ('blue', 'L')]

# product with repeat
dice = itertools.product(range(1, 7), repeat=2)
print(len(list(dice)))  # 36 (6 * 6)

# permutations() - ordered arrangements
items = ['A', 'B', 'C']
perms = itertools.permutations(items, 2)
print(list(perms))
# [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]

# combinations() - unordered selections (no replacement)
combs = itertools.combinations(items, 2)
print(list(combs))
# [('A', 'B'), ('A', 'C'), ('B', 'C')]

# combinations_with_replacement()
combs_rep = itertools.combinations_with_replacement(items, 2)
print(list(combs_rep))
# [('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]

# ✅ GOOD: Generate test cases
def generate_test_combinations(params):
    """Generate all combinations of parameters for testing."""
    keys = params.keys()
    values = params.values()
    for combination in itertools.product(*values):
        yield dict(zip(keys, combination))

test_params = {
    'browser': ['chrome', 'firefox'],
    'os': ['windows', 'mac'],
    'resolution': ['1080p', '4k']
}

for test_case in generate_test_combinations(test_params):
    print(test_case)
# {'browser': 'chrome', 'os': 'windows', 'resolution': '1080p'}
# {'browser': 'chrome', 'os': 'windows', 'resolution': '4k'}
# ...
```

### Advanced Iteration

#### Lazy Evaluation

```python
# Lazy evaluation: Compute values only when needed

# ❌ EAGER: Compute everything upfront
def eager_process(data):
    filtered = [x for x in data if x > 0]  # Processes all
    squared = [x**2 for x in filtered]     # Processes all
    return squared[:10]  # Only need first 10!

# ✅ LAZY: Compute only what's needed
def lazy_process(data):
    filtered = (x for x in data if x > 0)  # Generator
    squared = (x**2 for x in filtered)     # Generator
    return list(itertools.islice(squared, 10))  # Take first 10

# Benchmark
import time

large_data = range(-1000000, 1000000)

start = time.time()
result1 = eager_process(large_data)
print(f"Eager: {time.time() - start:.3f}s")  # ~0.5s

start = time.time()
result2 = lazy_process(large_data)
print(f"Lazy: {time.time() - start:.3f}s")  # ~0.001s (500x faster!)

# Lazy evaluation enables infinite sequences
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Can work with infinite sequence
first_20_fibs = list(itertools.islice(fibonacci(), 20))

# ✅ GOOD: Lazy database queries
def query_database_lazy(query):
    cursor = execute_query(query)
    for row in cursor:  # Fetches one row at a time
        yield process_row(row)

# vs eager loading (memory issues with large results)
def query_database_eager(query):
    return [process_row(row) for row in execute_query(query).fetchall()]
```

#### Iterator Chaining

```python
# Chain multiple iterators efficiently

# ✅ GOOD: itertools.chain
import itertools

def read_multiple_files(filenames):
    iterators = (read_file(f) for f in filenames)
    return itertools.chain.from_iterable(iterators)

# Process all files as one stream
for line in read_multiple_files(['file1.txt', 'file2.txt', 'file3.txt']):
    process(line)

# ✅ GOOD: Custom chaining
def chain_iterators(*iterators):
    for iterator in iterators:
        yield from iterator

combined = chain_iterators(range(3), range(10, 13), range(20, 23))
print(list(combined))  # [0, 1, 2, 10, 11, 12, 20, 21, 22]
```

#### Performance Considerations

```python
import time
import sys

# Memory comparison
list_data = [x for x in range(1000000)]  # List
gen_data = (x for x in range(1000000))   # Generator

print(f"List: {sys.getsizeof(list_data):,} bytes")  # ~8 MB
print(f"Generator: {sys.getsizeof(gen_data):,} bytes")  # ~200 bytes

# Speed comparison: sum
start = time.time()
total1 = sum([x**2 for x in range(1000000)])  # List
print(f"List comprehension: {time.time() - start:.3f}s")

start = time.time()
total2 = sum(x**2 for x in range(1000000))  # Generator
print(f"Generator expression: {time.time() - start:.3f}s")  # Faster!

# When to use each:
# ✅ Use generators when:
# - Processing large datasets
# - Single pass through data
# - Memory is constrained
# - Building pipelines

# ✅ Use lists when:
# - Need multiple passes
# - Need random access
# - Need len()
# - Small datasets
```

### Frequently Asked Questions

**Q1: When should I use a generator instead of a list?**

**A:**

```python
# Use generators when:

# 1. Processing large datasets
def read_large_file(filename):
    # ✅ Generator: Memory efficient
    with open(filename) as f:
        for line in f:
            yield line.strip()

# vs
def read_large_file_list(filename):
    # ❌ List: Loads entire file into memory!
    with open(filename) as f:
        return [line.strip() for line in f]

# 2. Infinite sequences
def fibonacci():
    a, b = 0, 1
    while True:  # Infinite!
        yield a
        a, b = b, a + b

# Take first 20
first_20 = list(itertools.islice(fibonacci(), 20))

# 3. Pipeline processing
def process_pipeline(data):
    # Each step is lazy
    filtered = (x for x in data if x > 0)
    squared = (x**2 for x in filtered)
    limited = itertools.islice(squared, 100)
    return list(limited)  # Only computes first 100

# 4. Early termination
def find_first_match(items, condition):
    # ✅ Generator: Stops as soon as found
    for item in items:
        if condition(item):
            return item  # Can stop early!

# Use lists when:

# 1. Need multiple passes
data = [1, 2, 3, 4, 5]
print(sum(data))  # First pass
print(max(data))  # Second pass - works!

# Generator would be exhausted
gen = (x for x in range(5))
print(sum(gen))  # Works
print(max(gen))  # Returns -inf (exhausted!)

# 2. Need random access
numbers = [10, 20, 30, 40, 50]
print(numbers[2])  # 30
# Generators don't support indexing

# 3. Need length
data = [1, 2, 3, 4, 5]
print(len(data))  # 5
# len(generator) doesn't work

# 4. Small datasets
small_data = [x**2 for x in range(10)]  # Fine for small data
```

**Why This Matters:** Choosing the right data structure affects memory usage and performance significantly.

---

**Q2: How do I pass a generator to a function multiple times?**

**A:**

```python
# Problem: Generators are exhausted after first use

# ❌ DOESN'T WORK:
def process_twice(data):
    print(list(data))  # First use
    print(list(data))  # Second use - empty!

gen = (x for x in range(5))
process_twice(gen)  # Prints [0,1,2,3,4] then []

# Solution 1: Convert to list (if memory allows)
def process_with_list(data):
    data_list = list(data)  # Materialize once
    print(data_list)  # First use
    print(data_list)  # Second use - works!

# Solution 2: Pass generator factory (function)
def process_with_factory(data_factory):
    print(list(data_factory()))  # First generator
    print(list(data_factory()))  # New generator

process_with_factory(lambda: (x for x in range(5)))

# Solution 3: Use itertools.tee
import itertools

def process_with_tee(data):
    iter1, iter2 = itertools.tee(data, 2)
    print(list(iter1))  # First iterator
    print(list(iter2))  # Second iterator (independent)

gen = (x for x in range(5))
process_with_tee(gen)

# ⚠️ WARNING: tee() stores values internally
# Not memory-efficient for large datasets

# ✅ BEST: Design to use generator once
def proper_design(data):
    # Process in single pass
    results = []
    for item in data:
        results.append(process(item))
    return results
```

**Why This Matters:** Understanding generator lifetime prevents bugs and memory issues.

---

### Interview Questions

**Question 1: Explain the difference between an iterable and an iterator.**

**Difficulty:** Junior

**SDLC Relevance:** Development

**Answer:**

```python
# Iterable: Object that can return an iterator
# Iterator: Object that produces values one at a time

# Iterable example: list
numbers = [1, 2, 3]  # Iterable (has __iter__)

# Get iterator from iterable
iterator = iter(numbers)  # Iterator (has __iter__ and __next__)

# Iterate manually
print(next(iterator))  # 1
print(next(iterator))  # 2
print(next(iterator))  # 3
# next(iterator)  # StopIteration

# Key differences:

# 1. Iterable can create multiple iterators
numbers = [1, 2, 3]
iter1 = iter(numbers)
iter2 = iter(numbers)

print(next(iter1))  # 1
print(next(iter2))  # 1 (independent!)

# 2. Iterator can only be used once
iterator = iter([1, 2, 3])
list(iterator)  # [1, 2, 3]
list(iterator)  # [] (exhausted!)

# 3. Iterator is also iterable
iterator = iter([1, 2, 3])
print(isinstance(iterator, Iterator))  # True
print(isinstance(iterator, Iterable))  # True
# But not all iterables are iterators!

# 4. Check with collections.abc
from collections.abc import Iterable, Iterator

numbers = [1, 2, 3]
print(isinstance(numbers, Iterable))  # True
print(isinstance(numbers, Iterator))  # False

iterator = iter(numbers)
print(isinstance(iterator, Iterator))  # True

# Implementation:
class MyIterable:
    def __iter__(self):
        return MyIterator()

class MyIterator:
    def __init__(self):
        self.count = 0
  
    def __iter__(self):
        return self
  
    def __next__(self):
        if self.count >= 3:
            raise StopIteration
        self.count += 1
        return self.count

# Use
for num in MyIterable():
    print(num)  # 1, 2, 3
```

**Key Points:**

- Iterable: has `__iter__()`, returns iterator
- Iterator: has `__iter__()` and `__next__()`
- Iterators are consumable (single use)
- Iterables can create multiple independent iterators

---

### Key Takeaways

**Iterators and Generators:**

- **Iterator protocol**: `__iter__()` returns iterator, `__next__()` produces values
- **Iterables vs iterators**: Iterables produce iterators, iterators produce values
- **Generator functions**: Use `yield` for lazy value production
- **Generator expressions**: Memory-efficient alternative to list comprehensions
- **Generator methods**: send(), throw(), close() for advanced control
- **yield from**: Delegate to sub-generators
- **itertools**: Powerful toolkit for iterator operations
- **Lazy evaluation**: Compute only what's needed, when needed
- **Pipelines**: Chain generators for efficient data processing

**Performance:**

- Generators use minimal memory (~200 bytes vs megabytes for lists)
- Enable processing datasets larger than memory
- Support early termination (don't compute unused values)
- Ideal for I/O-bound operations and streaming data

---

## 2.4 Functional Programming

**SDLC Phase:** Development

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (architectural patterns)
- [X] Development
- [ ] Testing
- [ ] Deployment
- [X] Maintenance (code clarity)

Functional programming emphasizes immutability, pure functions, and composition for predictable, testable code.

### Functional Paradigms

#### Pure Functions

```python
# Pure function: Same input → same output, no side effects

# ✅ PURE: Deterministic, no side effects
def add(a, b):
    return a + b

# Always returns same result
assert add(2, 3) == 5
assert add(2, 3) == 5  # Predictable!

# ❌ IMPURE: Side effects
counter = 0

def impure_increment():
    global counter
    counter += 1  # Side effect!
    return counter

# Results change each call
print(impure_increment())  # 1
print(impure_increment())  # 2 (unpredictable!)

# ❌ IMPURE: Depends on external state
import random

def impure_random():
    return random.randint(1, 10)  # Non-deterministic!

# ✅ PURE: Pass state explicitly
def pure_increment(counter):
    return counter + 1  # No side effects

result1 = pure_increment(0)  # 1
result2 = pure_increment(0)  # 1 (predictable!)

# Benefits of pure functions:
# 1. Easy to test
# 2. Easy to reason about
# 3. Can be cached (memoized)
# 4. Thread-safe
# 5. Composable

# ✅ PURE: Transform data
def double_items(items):
    return [x * 2 for x in items]

original = [1, 2, 3]
doubled = double_items(original)
print(original)  # [1, 2, 3] (unchanged!)
print(doubled)   # [2, 4, 6]

# ❌ IMPURE: Mutate input
def bad_double_items(items):
    for i in range(len(items)):
        items[i] *= 2  # Mutates input!
    return items

original = [1, 2, 3]
doubled = bad_double_items(original)
print(original)  # [2, 4, 6] (modified! Unexpected!)
```

#### Immutability

```python
# Immutable data structures prevent accidental modifications

# Built-in immutables
x = (1, 2, 3)  # Tuple
# x[0] = 10  # TypeError!

y = frozenset([1, 2, 3])
# y.add(4)  # AttributeError!

z = "hello"
# z[0] = 'H'  # TypeError!

# ✅ GOOD: Use immutable defaults
from typing import Tuple

def process_data(items: Tuple[int, ...]) -> Tuple[int, ...]:
    # Caller can't accidentally modify
    return tuple(x * 2 for x in items)

# ✅ GOOD: dataclass with frozen=True
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
# p.x = 3.0  # FrozenInstanceError!

# Create new instance instead
p2 = Point(3.0, p.y)

# ✅ GOOD: Use copy for "mutation"
def add_item(items, new_item):
    # Don't modify original
    return items + [new_item]

original = [1, 2, 3]
updated = add_item(original, 4)
print(original)  # [1, 2, 3] (unchanged)
print(updated)   # [1, 2, 3, 4]

# Immutable update patterns
# Dictionary
def update_user(user, **updates):
    return {**user, **updates}  # Merge creates new dict

user = {'name': 'Alice', 'age': 30}
updated = update_user(user, age=31)
print(user)     # {'name': 'Alice', 'age': 30} (unchanged)
print(updated)  # {'name': 'Alice', 'age': 31}
```

### Functional Tools

#### map, filter, reduce

```python
# map() - apply function to all items
numbers = [1, 2, 3, 4, 5]

# With map
squared = map(lambda x: x**2, numbers)
print(list(squared))  # [1, 4, 9, 16, 25]

# Equivalent comprehension (more Pythonic)
squared = [x**2 for x in numbers]

# filter() - keep items matching condition
evens = filter(lambda x: x % 2 == 0, numbers)
print(list(evens))  # [2, 4]

# Equivalent comprehension
evens = [x for x in numbers if x % 2 == 0]

# reduce() - cumulative operation
from functools import reduce

total = reduce(lambda acc, x: acc + x, numbers, 0)
print(total)  # 15

# Equivalent loop
total = 0
for x in numbers:
    total += x

# When to use functional tools:

# ✅ GOOD: With existing functions
def square(x):
    return x ** 2

numbers = [1, 2, 3, 4, 5]
squared = list(map(square, numbers))

# ✅ GOOD: Multiple iterables
nums1 = [1, 2, 3]
nums2 = [10, 20, 30]
sums = list(map(lambda x, y: x + y, nums1, nums2))
print(sums)  # [11, 22, 33]

# ✅ BETTER: Use comprehensions for simple cases
squared = [square(x) for x in numbers]
evens = [x for x in numbers if x % 2 == 0]

# ✅ GOOD: reduce for complex accumulation
from functools import reduce

def merge_dicts(dicts):
    return reduce(
        lambda acc, d: {**acc, **d},
        dicts,
        {}
    )

result = merge_dicts([{'a': 1}, {'b': 2}, {'c': 3}])
print(result)  # {'a': 1, 'b': 2, 'c': 3}
```

#### Partial Application

```python
from functools import partial

# partial() - fix some arguments
def power(base, exponent):
    return base ** exponent

# Create specialized functions
square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125

# ✅ GOOD: Configure functions
import logging

# Create specialized loggers
debug = partial(logging.log, logging.DEBUG)
info = partial(logging.log, logging.INFO)
error = partial(logging.log, logging.ERROR)

debug("Debug message")
error("Error occurred!")

# ✅ GOOD: Callback configuration
from functools import partial

def process_data(data, multiplier, offset):
    return [x * multiplier + offset for x in data]

# Create configured processors
double_plus_ten = partial(process_data, multiplier=2, offset=10)
triple_plus_five = partial(process_data, multiplier=3, offset=5)

data = [1, 2, 3]
print(double_plus_ten(data))   # [12, 14, 16]
print(triple_plus_five(data))  # [8, 11, 14]
```

#### Function Composition

```python
# Compose functions together

# Manual composition
def add_one(x):
    return x + 1

def double(x):
    return x * 2

def square(x):
    return x ** 2

# Nested calls (hard to read)
result = square(double(add_one(5)))  # ((5+1)*2)^2 = 144

# ✅ GOOD: Composition helper
def compose(*functions):
    def inner(x):
        for func in reversed(functions):
            x = func(x)
        return x
    return inner

# Create pipeline
pipeline = compose(square, double, add_one)
result = pipeline(5)  # 144

# ✅ BETTER: Use operator chaining
class Pipeline:
    def __init__(self, value):
        self.value = value
  
    def map(self, func):
        self.value = func(self.value)
        return self
  
    def get(self):
        return self.value

result = (Pipeline(5)
    .map(add_one)
    .map(double)
    .map(square)
    .get())

print(result)  # 144

# ✅ GOOD: Pipe operator (Python 3.10+ pattern match)
def pipe(value, *functions):
    for func in functions:
        value = func(value)
    return value

result = pipe(5, add_one, double, square)
print(result)  # 144
```

### Functional Patterns

#### Higher-Order Functions

```python
# Functions that take or return functions

# Function as argument
def apply_twice(func, value):
    return func(func(value))

def increment(x):
    return x + 1

result = apply_twice(increment, 5)  # (5+1)+1 = 7

# Function as return value
def make_multiplier(factor):
    def multiplier(x):
        return x * factor
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15

# ✅ GOOD: Decorator pattern
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Time: {time.time() - start:.3f}s")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)

slow_function()  # Prints: Time: 1.000s

# ✅ GOOD: Strategy pattern
def sort_by(key_func):
    def sorter(items):
        return sorted(items, key=key_func)
    return sorter

sort_by_length = sort_by(len)
sort_by_first_char = sort_by(lambda x: x[0])

words = ['apple', 'pie', 'a', 'cherry']
print(sort_by_length(words))  # ['a', 'pie', 'apple', 'cherry']
print(sort_by_first_char(words))  # ['apple', 'a', 'cherry', 'pie']
```

#### Closures

```python
# Closure: Function that captures enclosing scope

def make_counter():
    count = 0  # Captured by closure
  
    def increment():
        nonlocal count
        count += 1
        return count
  
    return increment

counter = make_counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3

# ✅ GOOD: Encapsulation with closures
def create_account(initial_balance):
    balance = initial_balance  # Private variable
  
    def deposit(amount):
        nonlocal balance
        balance += amount
        return balance
  
    def withdraw(amount):
        nonlocal balance
        if amount > balance:
            raise ValueError("Insufficient funds")
        balance -= amount
        return balance
  
    def get_balance():
        return balance
  
    return {
        'deposit': deposit,
        'withdraw': withdraw,
        'balance': get_balance
    }

account = create_account(1000)
account['deposit'](500)   # 1500
account['withdraw'](200)  # 1300
print(account['balance']())  # 1300
# balance variable is private!

# ✅ GOOD: Memoization with closures
def memoize(func):
    cache = {}  # Captured by closure
  
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
  
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # Fast! (cached)
```

### Frequently Asked Questions

**Q1: When should I use functional programming in Python?**

**A:**

```python
# Use functional programming when:

# 1. Data transformations
# ✅ GOOD: Pipeline of transformations
def process_sales(sales_data):
    return (
        Pipeline(sales_data)
        .map(lambda x: x['amount'])  # Extract amounts
        .filter(lambda x: x > 100)   # Filter high values
        .map(lambda x: x * 1.1)      # Apply tax
        .collect()
    )

# 2. Concurrent/parallel processing
from multiprocessing import Pool

def square(x):
    return x ** 2

with Pool() as pool:
    results = pool.map(square, range(1000000))

# Pure functions are thread-safe!

# 3. Testing
# ✅ Easy to test pure functions
def calculate_discount(price, percent):
    return price * (1 - percent / 100)

assert calculate_discount(100, 10) == 90  # Predictable!

# 4. Caching/memoization
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_calculation(n):
    # Pure function can be cached
    return sum(i**2 for i in range(n))

# Don't use functional programming when:

# ❌ Performance-critical low-level code
# Functional style may have overhead

# ❌ Stateful systems (UI, games)
# State is inherent to these domains

# ❌ I/O-heavy code
# Side effects are unavoidable

# Mixed approach (pragmatic Python):
class DataProcessor:
    def __init__(self, config):
        self.config = config  # State
  
    def process(self, data):
        # Use functional style for data transformation
        return [
            self._transform(item)  # Pure method
            for item in data
            if self._is_valid(item)  # Pure method
        ]
  
    def _transform(self, item):
        # Pure function
        return item * self.config['multiplier']
  
    def _is_valid(self, item):
        # Pure function
        return item > 0
```

**Why This Matters:** Functional programming improves code testability and predictability when used appropriately.

---

### Key Takeaways

**Functional Programming:**

- **Pure functions**: Same input → same output, no side effects
- **Immutability**: Prevent accidental modifications, create new data instead
- **map/filter/reduce**: Transform collections functionally
- **Partial application**: Fix some arguments, create specialized functions
- **Composition**: Combine simple functions into complex operations
- **Higher-order functions**: Functions that take/return functions
- **Closures**: Capture enclosing scope for encapsulation
- **Lazy evaluation**: Generators enable functional pipelines

**Benefits:**

- Easier to test (predictable behavior)
- Easier to reason about (no hidden state)
- Thread-safe (no shared mutable state)
- Composable (build complex from simple)

**Python approach:**

- Pragmatic: Mix functional and OOP styles
- Use comprehensions over map/filter
- Use pure functions for data transformations
- Use classes for stateful components

---

(Continuing with Section 2.5 - Asynchronous Programming in next file...)

## 2.5 Asynchronous Programming

**SDLC Phase:** Development, System Design

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (scalability)
- [X] Development
- [X] Testing (async testing)
- [X] Deployment (performance)
- [X] Maintenance (debugging async code)

Asynchronous programming enables high-concurrency applications by overlapping I/O operations without the overhead of threading.

### Async Fundamentals

#### Synchronous vs Asynchronous

```python
# Synchronous: Wait for each operation to complete
import time

def fetch_data_sync(url):
    time.sleep(1)  # Simulates network I/O
    return f"Data from {url}"

def sync_example():
    start = time.time()
  
    # Sequential execution
    result1 = fetch_data_sync("url1")
    result2 = fetch_data_sync("url2")
    result3 = fetch_data_sync("url3")
  
    print(f"Sync time: {time.time() - start:.2f}s")  # ~3 seconds

sync_example()

# Asynchronous: Overlap I/O operations
import asyncio

async def fetch_data_async(url):
    await asyncio.sleep(1)  # Simulates async I/O
    return f"Data from {url}"

async def async_example():
    start = time.time()
  
    # Concurrent execution
    results = await asyncio.gather(
        fetch_data_async("url1"),
        fetch_data_async("url2"),
        fetch_data_async("url3")
    )
  
    print(f"Async time: {time.time() - start:.2f}s")  # ~1 second!

asyncio.run(async_example())

# When to use async:
# ✅ I/O-bound operations (network, database, file I/O)
# ✅ High-concurrency applications (web servers, API clients)
# ✅ Real-time systems (WebSockets, streaming)

# When NOT to use async:
# ❌ CPU-bound operations (use multiprocessing)
# ❌ Simple scripts (unnecessary complexity)
# ❌ Libraries without async support
```

#### Coroutines (async/await)

```python
import asyncio

# Coroutine function (async def)
async def hello():
    print("Hello")
    await asyncio.sleep(1)  # Suspends coroutine
    print("World")

# Calling returns coroutine object
coro = hello()
print(type(coro))  # <class 'coroutine'>

# Run coroutine
asyncio.run(coro)

# await - suspend execution until coroutine completes
async def fetch_user(user_id):
    await asyncio.sleep(0.1)  # Async I/O
    return {"id": user_id, "name": f"User {user_id}"}

async def main():
    # Sequential (3 seconds)
    user1 = await fetch_user(1)
    user2 = await fetch_user(2)
    user3 = await fetch_user(3)
  
    # Concurrent (1 second)
    users = await asyncio.gather(
        fetch_user(1),
        fetch_user(2),
        fetch_user(3)
    )
    print(users)

asyncio.run(main())

# ✅ GOOD: Use await for I/O operations
async def process_request():
    # I/O operations are awaited
    data = await fetch_from_database()
    result = await call_external_api(data)
    await save_to_cache(result)
    return result

# ❌ BAD: Blocking I/O in async function
async def bad_request():
    data = blocking_database_call()  # Blocks event loop!
    return data

# ✅ GOOD: Use asyncio.to_thread for blocking calls
async def good_request():
    data = await asyncio.to_thread(blocking_database_call)
    return data
```

#### Event Loop

```python
import asyncio

# Event loop - heart of async Python
# Manages and schedules coroutines

# Get current event loop
async def main():
    loop = asyncio.get_event_loop()
    print(f"Loop: {loop}")

asyncio.run(main())

# Create and run tasks
async def task1():
    print("Task 1 start")
    await asyncio.sleep(1)
    print("Task 1 end")

async def task2():
    print("Task 2 start")
    await asyncio.sleep(0.5)
    print("Task 2 end")

async def main():
    # Create tasks
    t1 = asyncio.create_task(task1())
    t2 = asyncio.create_task(task2())
  
    # Tasks run concurrently
    await t1
    await t2

asyncio.run(main())
# Output:
# Task 1 start
# Task 2 start
# Task 2 end
# Task 1 end

# Schedule callbacks
def callback(arg):
    print(f"Callback with {arg}")

async def schedule_example():
    loop = asyncio.get_event_loop()
  
    # Schedule callback
    loop.call_soon(callback, "immediate")
    loop.call_later(1, callback, "after 1s")
  
    await asyncio.sleep(2)

asyncio.run(schedule_example())
```

#### Tasks and Futures

```python
import asyncio

# Task - wrapper for coroutine
async def my_coroutine():
    await asyncio.sleep(1)
    return "Done"

async def main():
    # Create task
    task = asyncio.create_task(my_coroutine())
  
    # Task runs in background
    print("Task created")
  
    # Wait for task
    result = await task
    print(result)

asyncio.run(main())

# Future - placeholder for future result
async def future_example():
    loop = asyncio.get_event_loop()
    future = loop.create_future()
  
    # Set result later
    loop.call_later(1, future.set_result, "Result!")
  
    # Wait for future
    result = await future
    print(result)

asyncio.run(future_example())

# Task vs Future:
# - Task: Wraps coroutine, manages execution
# - Future: Lower-level, manually set result
```

### Async Patterns

#### Async Context Managers

```python
import asyncio

# Async context manager
class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource")
        await asyncio.sleep(0.1)  # Async setup
        return self
  
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        await asyncio.sleep(0.1)  # Async cleanup
        return False

async def main():
    async with AsyncResource() as resource:
        print("Using resource")
        await asyncio.sleep(0.5)

asyncio.run(main())

# ✅ GOOD: Async database connection
class AsyncDatabase:
    async def __aenter__(self):
        self.conn = await connect_to_database()
        return self.conn
  
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

async def query_database():
    async with AsyncDatabase() as db:
        result = await db.query("SELECT * FROM users")
        return result

# Use @asynccontextmanager
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_timer():
    import time
    start = time.time()
    yield
    print(f"Time: {time.time() - start:.2f}s")

async def timed_operation():
    async with async_timer():
        await asyncio.sleep(1)

asyncio.run(timed_operation())
```

#### Async Generators

```python
import asyncio

# Async generator (async def + yield)
async def async_range(n):
    for i in range(n):
        await asyncio.sleep(0.1)  # Async operation
        yield i

async def main():
    # Async iteration
    async for value in async_range(5):
        print(value)

asyncio.run(main())

# ✅ GOOD: Stream processing
async def read_log_stream(filename):
    async with aiofiles.open(filename) as f:
        async for line in f:
            await asyncio.sleep(0.01)  # Rate limiting
            yield line.strip()

async def process_logs():
    async for line in read_log_stream('server.log'):
        if 'ERROR' in line:
            print(line)

# Async comprehensions
async def async_comprehension():
    # Async list comprehension
    squares = [i**2 async for i in async_range(5)]
    print(squares)
  
    # Async dict comprehension
    data = {i: i**2 async for i in async_range(5)}
    print(data)

asyncio.run(async_comprehension())
```

#### Concurrent Execution

```python
import asyncio

# gather() - run multiple coroutines concurrently
async def task(name, delay):
    await asyncio.sleep(delay)
    return f"{name} done"

async def gather_example():
    # Run concurrently, wait for all
    results = await asyncio.gather(
        task("Task1", 1),
        task("Task2", 0.5),
        task("Task3", 1.5)
    )
    print(results)  # All results in order

asyncio.run(gather_example())

# wait() - more control over completion
async def wait_example():
    tasks = [
        asyncio.create_task(task("Task1", 1)),
        asyncio.create_task(task("Task2", 2)),
        asyncio.create_task(task("Task3", 3))
    ]
  
    # Wait for first completion
    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED
    )
  
    print(f"Done: {len(done)}, Pending: {len(pending)}")
  
    # Cancel pending
    for t in pending:
        t.cancel()

asyncio.run(wait_example())

# as_completed() - process results as they complete
async def as_completed_example():
    tasks = [
        task("Task1", 2),
        task("Task2", 1),
        task("Task3", 3)
    ]
  
    # Process in completion order (not submission order)
    for coro in asyncio.as_completed(tasks):
        result = await coro
        print(result)  # Task2, Task1, Task3

asyncio.run(as_completed_example())

# ✅ GOOD: Fetch multiple URLs concurrently
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        return await asyncio.gather(*tasks)

urls = ['http://example.com'] * 10
# results = asyncio.run(fetch_all(urls))
```

### Event Loop

#### Event Loop Implementation

```python
import asyncio

# Event loop manages coroutines
async def main():
    # Get current loop
    loop = asyncio.get_running_loop()
  
    # Run in executor (thread pool)
    import time
    def blocking_io():
        time.sleep(1)
        return "Done"
  
    result = await loop.run_in_executor(None, blocking_io)
    print(result)

asyncio.run(main())

# Low-level event loop control
async def low_level():
    loop = asyncio.get_event_loop()
  
    # Schedule coroutine
    task = loop.create_task(asyncio.sleep(1))
  
    # Schedule callback
    loop.call_soon(lambda: print("Callback"))
  
    # Schedule delayed callback
    loop.call_later(0.5, lambda: print("Delayed"))
  
    await task

asyncio.run(low_level())

# ✅ GOOD: Custom event loop policy
class CustomEventLoopPolicy(asyncio.DefaultEventLoopPolicy):
    def new_event_loop(self):
        loop = super().new_event_loop()
        # Customize loop
        return loop

asyncio.set_event_loop_policy(CustomEventLoopPolicy())
```

#### Running the Loop

```python
import asyncio

# asyncio.run() - high-level API (recommended)
async def main():
    await asyncio.sleep(1)
    return "Done"

result = asyncio.run(main())  # Runs and cleans up

# Low-level loop management
async def coroutine():
    await asyncio.sleep(1)

# Create loop
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

try:
    # Run coroutine
    result = loop.run_until_complete(coroutine())
  
    # Run multiple tasks
    loop.run_until_complete(asyncio.gather(
        coroutine(),
        coroutine()
    ))
finally:
    loop.close()

# ✅ GOOD: Long-running event loop (servers)
async def server_loop():
    # Setup
    server = await start_server()
  
    try:
        # Run forever
        await server.serve_forever()
    except KeyboardInterrupt:
        print("Shutting down...")
    finally:
        await server.shutdown()
```

### Async Libraries

#### aiohttp - Async HTTP

```python
import asyncio
import aiohttp

# Async HTTP client
async def fetch_json(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# Concurrent requests
async def fetch_multiple(urls):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in urls:
            task = asyncio.create_task(fetch_json(url))
            tasks.append(task)
      
        return await asyncio.gather(*tasks)

# urls = ['http://api.example.com/1', 'http://api.example.com/2']
# results = asyncio.run(fetch_multiple(urls))

# Async HTTP server
from aiohttp import web

async def handle_request(request):
    name = request.match_info.get('name', 'World')
    return web.Response(text=f"Hello, {name}!")

app = web.Application()
app.router.add_get('/{name}', handle_request)

# web.run_app(app, port=8080)

# ✅ GOOD: Connection pooling
async def efficient_requests():
    # Reuse session for connection pooling
    async with aiohttp.ClientSession() as session:
        for i in range(100):
            async with session.get(f'http://api.example.com/{i}') as response:
                data = await response.text()
```

#### aiofiles - Async File I/O

```python
import asyncio
import aiofiles

# Async file reading
async def read_file(filename):
    async with aiofiles.open(filename, 'r') as f:
        contents = await f.read()
        return contents

# Async file writing
async def write_file(filename, data):
    async with aiofiles.open(filename, 'w') as f:
        await f.write(data)

# Process large file line by line
async def process_large_file(filename):
    async with aiofiles.open(filename, 'r') as f:
        async for line in f:
            await process_line(line)  # Async processing

# asyncio.run(process_large_file('large_file.txt'))
```

#### Database Async Drivers

```python
import asyncio

# asyncpg - Async PostgreSQL
import asyncpg

async def fetch_users():
    # Connect to database
    conn = await asyncpg.connect(
        host='localhost',
        database='mydb',
        user='user',
        password='password'
    )
  
    try:
        # Query
        rows = await conn.fetch('SELECT * FROM users')
        return rows
    finally:
        await conn.close()

# Connection pooling
async def with_pool():
    pool = await asyncpg.create_pool(
        host='localhost',
        database='mydb'
    )
  
    try:
        async with pool.acquire() as conn:
            result = await conn.fetch('SELECT * FROM users')
    finally:
        await pool.close()

# aiomysql - Async MySQL
import aiomysql

async def query_mysql():
    pool = await aiomysql.create_pool(
        host='localhost',
        user='user',
        password='password',
        db='mydb'
    )
  
    async with pool.acquire() as conn:
        async with conn.cursor() as cursor:
            await cursor.execute('SELECT * FROM users')
            result = await cursor.fetchall()
  
    pool.close()
    await pool.wait_closed()
```

### Async Best Practices

#### When to Use Async

```python
# ✅ GOOD: I/O-bound operations
async def io_bound_example():
    # Network requests
    async with aiohttp.ClientSession() as session:
        response = await session.get('http://api.example.com')
  
    # Database queries
    result = await db.query('SELECT * FROM users')
  
    # File I/O
    async with aiofiles.open('file.txt') as f:
        data = await f.read()

# ❌ BAD: CPU-bound operations
async def cpu_bound_example():
    # Don't use async for CPU-intensive work!
    result = sum(i**2 for i in range(10**7))  # Blocks event loop
    return result

# ✅ GOOD: CPU-bound with executor
async def cpu_bound_with_executor():
    loop = asyncio.get_event_loop()
  
    def cpu_task():
        return sum(i**2 for i in range(10**7))
  
    # Run in thread pool
    result = await loop.run_in_executor(None, cpu_task)
    return result

# Decision tree:
# - I/O-bound + high concurrency? → async/await
# - CPU-bound? → multiprocessing
# - I/O-bound + low concurrency? → threads or sync
# - Simple script? → synchronous
```

#### Mixing Sync and Async Code

```python
import asyncio

# ❌ BAD: Blocking call in async function
async def bad_mixing():
    import time
    time.sleep(1)  # Blocks entire event loop!
    return "Done"

# ✅ GOOD: Use asyncio.to_thread
async def good_mixing():
    import time
  
    def blocking_operation():
        time.sleep(1)
        return "Done"
  
    # Run in thread pool
    result = await asyncio.to_thread(blocking_operation)
    return result

# ✅ GOOD: Use loop.run_in_executor
async def with_executor():
    loop = asyncio.get_event_loop()
  
    def blocking_io():
        with open('file.txt') as f:
            return f.read()
  
    result = await loop.run_in_executor(None, blocking_io)
    return result

# Call async from sync code
def sync_function():
    async def async_operation():
        await asyncio.sleep(1)
        return "Done"
  
    # Run async code from sync context
    result = asyncio.run(async_operation())
    return result
```

#### Error Handling

```python
import asyncio

# Try/except in async functions
async def with_error_handling():
    try:
        result = await risky_async_operation()
    except ValueError as e:
        print(f"Error: {e}")
        return None
    except asyncio.TimeoutError:
        print("Operation timed out")
        return None
    finally:
        await cleanup()

# Timeout handling
async def with_timeout():
    try:
        result = await asyncio.wait_for(
            slow_operation(),
            timeout=5.0  # 5 seconds
        )
    except asyncio.TimeoutError:
        print("Operation timed out")
        return None

# Handle errors in gather
async def gather_with_errors():
    results = await asyncio.gather(
        task1(),
        task2(),
        task3(),
        return_exceptions=True  # Don't fail on first error
    )
  
    # Check results
    for result in results:
        if isinstance(result, Exception):
            print(f"Error: {result}")
        else:
            print(f"Success: {result}")

# ✅ GOOD: Graceful degradation
async def resilient_fetch(urls):
    async def safe_fetch(url):
        try:
            return await fetch_url(url)
        except Exception as e:
            print(f"Failed to fetch {url}: {e}")
            return None
  
    results = await asyncio.gather(*[safe_fetch(url) for url in urls])
    return [r for r in results if r is not None]
```

#### Cancellation and Timeouts

```python
import asyncio

# Task cancellation
async def cancellable_task():
    try:
        while True:
            await asyncio.sleep(1)
            print("Working...")
    except asyncio.CancelledError:
        print("Task was cancelled")
        # Cleanup
        raise  # Re-raise to propagate

async def cancel_example():
    task = asyncio.create_task(cancellable_task())
  
    # Let it run for a bit
    await asyncio.sleep(3)
  
    # Cancel task
    task.cancel()
  
    try:
        await task
    except asyncio.CancelledError:
        print("Task cancelled successfully")

asyncio.run(cancel_example())

# Timeout with context manager
async def with_timeout_cm():
    try:
        async with asyncio.timeout(5):  # Python 3.11+
            await slow_operation()
    except asyncio.TimeoutError:
        print("Timed out")

# Shield from cancellation
async def protected_operation():
    # This operation won't be cancelled
    result = await asyncio.shield(critical_task())
    return result
```

### Frequently Asked Questions

**Q1: What's the difference between async/await and threading?**

**A:**

```python
import asyncio
import threading
import time

# Threading: OS-level concurrency
def blocking_task(name):
    print(f"{name} start")
    time.sleep(1)
    print(f"{name} end")

def threading_example():
    start = time.time()
    threads = []
  
    for i in range(10):
        t = threading.Thread(target=blocking_task, args=(f"Thread{i}",))
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    print(f"Threading: {time.time() - start:.2f}s")  # ~1s

threading_example()

# Async: Cooperative concurrency
async def async_task(name):
    print(f"{name} start")
    await asyncio.sleep(1)  # Yields control
    print(f"{name} end")

async def async_example():
    start = time.time()
  
    tasks = [async_task(f"Task{i}") for i in range(10)]
    await asyncio.gather(*tasks)
  
    print(f"Async: {time.time() - start:.2f}s")  # ~1s

asyncio.run(async_example())

# Key differences:

# Threading:
# ✅ Works with blocking I/O
# ✅ No code changes needed
# ❌ GIL limits CPU parallelism
# ❌ Higher memory overhead (~8MB per thread)
# ❌ Thread switching overhead
# ❌ Race conditions possible

# Async/await:
# ✅ Very low memory overhead (~KB per task)
# ✅ No race conditions (single-threaded)
# ✅ Better performance for high concurrency
# ❌ Requires async libraries
# ❌ Doesn't work with blocking code
# ❌ More complex error handling

# When to use each:

# Use async/await when:
# - High concurrency (1000+ concurrent operations)
# - I/O-bound with async library support
# - Building async frameworks (web servers, etc.)

# Use threading when:
# - Working with blocking libraries
# - Low to medium concurrency (< 100 threads)
# - Need to integrate with sync code
```

**Why This Matters:** Choosing the right concurrency model affects performance, resource usage, and code complexity.

---

**Q2: How do I debug async code?**

**A:**

```python
import asyncio

# 1. Enable asyncio debug mode
asyncio.run(main(), debug=True)

# Or set environment variable:
# PYTHONASYNCIODEBUG=1 python script.py

# 2. Add logging
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

async def logged_operation():
    logger.debug("Starting operation")
    result = await async_call()
    logger.debug(f"Result: {result}")
    return result

# 3. Use task names
async def main():
    task = asyncio.create_task(
        slow_operation(),
        name="slow_operation_task"  # Named task
    )
  
    # List running tasks
    for task in asyncio.all_tasks():
        print(f"Task: {task.get_name()}")

# 4. Detect blocking code
import asyncio

async def detect_blocking():
    loop = asyncio.get_event_loop()
    loop.set_debug(True)  # Warns about slow callbacks
  
    # This will generate warning
    import time
    time.sleep(0.1)  # > 100ms threshold

# 5. Use asyncio REPL
# $ python -m asyncio
# >>> await asyncio.sleep(1)

# 6. Traceback for cancelled tasks
async def with_traceback():
    task = asyncio.create_task(long_running())
  
    try:
        task.cancel()
        await task
    except asyncio.CancelledError:
        import traceback
        traceback.print_stack()

# 7. Monitor task completion
async def monitor_tasks():
    tasks = [
        asyncio.create_task(task1(), name="task1"),
        asyncio.create_task(task2(), name="task2")
    ]
  
    done, pending = await asyncio.wait(tasks)
  
    for task in done:
        print(f"{task.get_name()} completed")
  
    for task in pending:
        print(f"{task.get_name()} still running")
```

**Why This Matters:** Async bugs can be subtle (race conditions, deadlocks). Proper debugging techniques save hours of troubleshooting.

---

### Interview Questions

**Question 1: Explain how the event loop works in asyncio.**

**Difficulty:** Mid-Level

**SDLC Relevance:** System Design, Performance

**Answer:**

```python
# Event loop is single-threaded cooperative scheduler

# How it works:
# 1. Maintains queue of tasks/callbacks
# 2. Runs one task until it awaits
# 3. Suspends task, runs next task
# 4. Resumes tasks when I/O completes

import asyncio

async def task1():
    print("Task 1: Start")
    await asyncio.sleep(1)  # Yields to event loop
    print("Task 1: End")

async def task2():
    print("Task 2: Start")
    await asyncio.sleep(0.5)
    print("Task 2: End")

async def main():
    # Create tasks
    t1 = asyncio.create_task(task1())
    t2 = asyncio.create_task(task2())
  
    # Event loop executes:
    # 1. task1 runs until await → suspends
    # 2. task2 runs until await → suspends
    # 3. After 0.5s: task2 resumes, completes
    # 4. After 1s: task1 resumes, completes
  
    await t1
    await t2

asyncio.run(main())
# Output:
# Task 1: Start
# Task 2: Start
# Task 2: End
# Task 1: End

# Event loop internals (simplified):
"""
class EventLoop:
    def __init__(self):
        self.ready_queue = deque()  # Ready tasks
        self.io_waiting = {}         # Tasks waiting for I/O
  
    def create_task(self, coro):
        task = Task(coro)
        self.ready_queue.append(task)
        return task
  
    def run(self):
        while self.ready_queue or self.io_waiting:
            # Run ready tasks
            while self.ready_queue:
                task = self.ready_queue.popleft()
                task.step()  # Run until next await
          
            # Wait for I/O (epoll/select)
            ready = self.poll_io()
            for task in ready:
                self.ready_queue.append(task)
"""

# Key points:
# - Single-threaded (no thread overhead)
# - Cooperative (tasks must yield)
# - Non-blocking I/O (epoll/select)
# - Callbacks scheduled efficiently

# Performance characteristics:
# - Can handle 10,000+ concurrent connections
# - Low memory per task (~2KB vs 8MB for threads)
# - No GIL issues (single thread)
```

**Key Points to Mention:**

- Event loop is single-threaded scheduler
- Tasks cooperatively yield control (await)
- Uses non-blocking I/O (epoll/select/IOCP)
- Tasks suspended at await, resumed when ready
- Very efficient for I/O-bound concurrency

---

**Question 2: How would you handle rate limiting in async code?**

**Difficulty:** Senior

**SDLC Relevance:** System Design, Production

**Answer:**

```python
import asyncio
import time
from collections import deque

# Solution 1: Semaphore (limit concurrent requests)
class RateLimiter:
    def __init__(self, max_concurrent):
        self.semaphore = asyncio.Semaphore(max_concurrent)
  
    async def __aenter__(self):
        await self.semaphore.acquire()
        return self
  
    async def __aexit__(self, *args):
        self.semaphore.release()

async def fetch_with_limit(url, limiter):
    async with limiter:
        return await fetch_url(url)

async def main():
    limiter = RateLimiter(max_concurrent=10)
  
    tasks = [
        fetch_with_limit(url, limiter)
        for url in urls
    ]
  
    results = await asyncio.gather(*tasks)

# Solution 2: Token bucket (requests per time window)
class TokenBucket:
    def __init__(self, rate, per):
        self.rate = rate  # Requests
        self.per = per    # Per time window (seconds)
        self.tokens = rate
        self.updated_at = time.monotonic()
        self.lock = asyncio.Lock()
  
    async def acquire(self):
        async with self.lock:
            # Refill tokens
            now = time.monotonic()
            elapsed = now - self.updated_at
            self.tokens = min(
                self.rate,
                self.tokens + elapsed * (self.rate / self.per)
            )
            self.updated_at = now
          
            # Wait for token
            if self.tokens < 1:
                wait_time = (1 - self.tokens) * (self.per / self.rate)
                await asyncio.sleep(wait_time)
                self.tokens = 0
            else:
                self.tokens -= 1

async def rate_limited_fetch(url, bucket):
    await bucket.acquire()
    return await fetch_url(url)

async def main():
    # 100 requests per minute
    bucket = TokenBucket(rate=100, per=60)
  
    tasks = [
        rate_limited_fetch(url, bucket)
        for url in urls
    ]
  
    results = await asyncio.gather(*tasks)

# Solution 3: Sliding window (precise rate limiting)
class SlidingWindowLimiter:
    def __init__(self, max_requests, window_size):
        self.max_requests = max_requests
        self.window_size = window_size
        self.requests = deque()
        self.lock = asyncio.Lock()
  
    async def acquire(self):
        async with self.lock:
            now = time.monotonic()
          
            # Remove old requests
            while self.requests and self.requests[0] < now - self.window_size:
                self.requests.popleft()
          
            # Wait if limit exceeded
            if len(self.requests) >= self.max_requests:
                wait_time = self.requests[0] + self.window_size - now
                await asyncio.sleep(wait_time)
                await self.acquire()  # Retry
            else:
                self.requests.append(now)

# Solution 4: aiolimiter library
from aiolimiter import AsyncLimiter

async def with_aiolimiter():
    # 10 requests per second
    limiter = AsyncLimiter(10, 1)
  
    async with limiter:
        result = await fetch_url(url)

# Production considerations:
# - Use Redis for distributed rate limiting
# - Implement exponential backoff for retries
# - Monitor rate limit metrics
# - Handle 429 (Too Many Requests) responses
```

**Key Points:**

- Semaphore: Limit concurrent requests
- Token bucket: Smooth rate over time
- Sliding window: Precise request counting
- Consider distributed systems (Redis)
- Handle backpressure gracefully

---

### Key Takeaways

**Asynchronous Programming:**

- **async/await**: Coroutines for non-blocking I/O
- **Event loop**: Single-threaded cooperative scheduler
- **Tasks**: Managed coroutine execution
- **asyncio.gather**: Run multiple coroutines concurrently
- **Async context managers**: Async resource management
- **Async generators**: Async iteration with yield
- **Error handling**: Try/except, timeouts, cancellation
- **Libraries**: aiohttp, aiofiles, asyncpg, aiomysql

**Best Practices:**

- Use async for I/O-bound, high-concurrency workloads
- Avoid blocking calls in async functions
- Use asyncio.to_thread for blocking operations
- Handle errors and timeouts properly
- Enable debug mode during development

**Performance:**

- 10,000+ concurrent connections possible
- Low memory overhead (~2KB per task)
- Single-threaded (no GIL issues)
- Requires async-compatible libraries

---

## 2.6 Concurrency & Parallelism

**SDLC Phase:** System Design, Performance Optimization

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (scalability)
- [X] Development
- [X] Testing (load testing)
- [X] Deployment (resource planning)
- [X] Maintenance (performance tuning)

Python offers multiple concurrency models: threading, multiprocessing, and async/await. Understanding when to use each is critical for performance.

### Threading

#### Thread Creation

```python
import threading
import time

# Create thread
def worker(name):
    print(f"{name}: Starting")
    time.sleep(2)
    print(f"{name}: Finished")

# Method 1: Thread class
thread = threading.Thread(target=worker, args=("Thread-1",))
thread.start()
thread.join()  # Wait for completion

# Method 2: Subclass Thread
class WorkerThread(threading.Thread):
    def __init__(self, name):
        super().__init__()
        self.name = name
  
    def run(self):
        print(f"{self.name}: Starting")
        time.sleep(2)
        print(f"{self.name}: Finished")

thread = WorkerThread("Thread-2")
thread.start()
thread.join()

# Multiple threads
threads = []
for i in range(5):
    t = threading.Thread(target=worker, args=(f"Worker-{i}",))
    threads.append(t)
    t.start()

# Wait for all
for t in threads:
    t.join()

# Daemon threads (background threads)
daemon = threading.Thread(target=worker, args=("Daemon",), daemon=True)
daemon.start()
# Program exits even if daemon thread is running
```

#### Thread Synchronization

```python
import threading

# Lock - mutual exclusion
lock = threading.Lock()
counter = 0

def increment():
    global counter
    for _ in range(100000):
        with lock:  # Thread-safe
            counter += 1

# Without lock - race condition!
counter = 0
threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)  # May not be 1,000,000!

# RLock - reentrant lock (can be acquired multiple times by same thread)
rlock = threading.RLock()

def recursive_function(n):
    with rlock:
        if n > 0:
            recursive_function(n - 1)  # Can reacquire

# Semaphore - limit concurrent access
semaphore = threading.Semaphore(3)  # Max 3 threads

def limited_access():
    with semaphore:
        print(f"{threading.current_thread().name}: Accessing resource")
        time.sleep(1)

threads = [threading.Thread(target=limited_access) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
# Only 3 threads access at once

# Event - signal between threads
event = threading.Event()

def waiter():
    print("Waiting for event...")
    event.wait()  # Block until set
    print("Event received!")

def setter():
    time.sleep(2)
    print("Setting event")
    event.set()

t1 = threading.Thread(target=waiter)
t2 = threading.Thread(target=setter)
t1.start()
t2.start()
t1.join()
t2.join()

# Condition - complex synchronization
condition = threading.Condition()
items = []

def producer():
    for i in range(5):
        with condition:
            items.append(i)
            print(f"Produced: {i}")
            condition.notify()  # Wake up consumer
        time.sleep(0.5)

def consumer():
    for _ in range(5):
        with condition:
            while not items:
                condition.wait()  # Wait for item
            item = items.pop(0)
            print(f"Consumed: {item}")

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)
t1.start()
t2.start()
t1.join()
t2.join()
```

#### Thread Communication

```python
import threading
import queue
import time

# Queue - thread-safe communication
q = queue.Queue()

def producer():
    for i in range(10):
        q.put(i)
        print(f"Produced: {i}")
        time.sleep(0.1)
    q.put(None)  # Sentinel

def consumer():
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Consumed: {item}")
        q.task_done()

threads = [
    threading.Thread(target=producer),
    threading.Thread(target=consumer)
]

for t in threads: t.start()
for t in threads: t.join()

# LifoQueue (stack)
lifo = queue.LifoQueue()
lifo.put(1)
lifo.put(2)
print(lifo.get())  # 2 (LIFO)

# PriorityQueue
pq = queue.PriorityQueue()
pq.put((2, "Low priority"))
pq.put((1, "High priority"))
print(pq.get())  # (1, "High priority")

# Thread-local data (separate data per thread)
thread_local = threading.local()

def worker():
    thread_local.value = threading.current_thread().name
    print(f"Thread value: {thread_local.value}")

threads = [threading.Thread(target=worker) for _ in range(3)]
for t in threads: t.start()
for t in threads: t.join()
```

#### Thread Pools

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    print(f"Processing {n}")
    time.sleep(1)
    return n * 2

# Thread pool
with ThreadPoolExecutor(max_workers=5) as executor:
    # Submit tasks
    futures = [executor.submit(task, i) for i in range(10)]
  
    # Get results
    for future in futures:
        result = future.result()
        print(f"Result: {result}")

# map() - simpler interface
with ThreadPoolExecutor(max_workers=5) as executor:
    results = executor.map(task, range(10))
    for result in results:
        print(result)

# ✅ GOOD: Thread pool for I/O-bound tasks
import requests

def fetch_url(url):
    response = requests.get(url)
    return len(response.content)

urls = ['http://example.com'] * 20

with ThreadPoolExecutor(max_workers=10) as executor:
    results = executor.map(fetch_url, urls)
    total = sum(results)
    print(f"Total bytes: {total}")
```

### Multiprocessing

#### Process Creation

```python
import multiprocessing
import time
import os

# Create process
def worker(name):
    print(f"{name} (PID: {os.getpid()})")
    time.sleep(2)
    print(f"{name} done")

# Method 1: Process class
process = multiprocessing.Process(target=worker, args=("Process-1",))
process.start()
process.join()

# Method 2: Subclass Process
class WorkerProcess(multiprocessing.Process):
    def __init__(self, name):
        super().__init__()
        self.name = name
  
    def run(self):
        print(f"{self.name} (PID: {os.getpid()})")
        time.sleep(2)

process = WorkerProcess("Process-2")
process.start()
process.join()

# Multiple processes
processes = []
for i in range(4):
    p = multiprocessing.Process(target=worker, args=(f"Worker-{i}",))
    processes.append(p)
    p.start()

for p in processes:
    p.join()

# Check process status
print(f"Alive: {process.is_alive()}")
print(f"Exit code: {process.exitcode}")
```

#### Process Communication

```python
import multiprocessing

# Queue - process-safe communication
def producer(q):
    for i in range(5):
        q.put(i)
    q.put(None)

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Consumed: {item}")

q = multiprocessing.Queue()
p1 = multiprocessing.Process(target=producer, args=(q,))
p2 = multiprocessing.Process(target=consumer, args=(q,))
p1.start()
p2.start()
p1.join()
p2.join()

# Pipe - two-way communication
def sender(conn):
    conn.send("Hello from sender")
    response = conn.recv()
    print(f"Sender received: {response}")
    conn.close()

def receiver(conn):
    msg = conn.recv()
    print(f"Receiver received: {msg}")
    conn.send("Hello from receiver")
    conn.close()

parent_conn, child_conn = multiprocessing.Pipe()
p1 = multiprocessing.Process(target=sender, args=(parent_conn,))
p2 = multiprocessing.Process(target=receiver, args=(child_conn,))
p1.start()
p2.start()
p1.join()
p2.join()

# Shared memory - Value and Array
def increment(shared_value):
    for _ in range(1000):
        with shared_value.get_lock():  # Thread-safe
            shared_value.value += 1

shared_val = multiprocessing.Value('i', 0)  # 'i' = integer
processes = [
    multiprocessing.Process(target=increment, args=(shared_val,))
    for _ in range(4)
]

for p in processes: p.start()
for p in processes: p.join()
print(f"Final value: {shared_val.value}")  # 4000

# Shared array
shared_array = multiprocessing.Array('i', [0, 0, 0, 0])

def fill_array(arr, index):
    arr[index] = index * 10

processes = [
    multiprocessing.Process(target=fill_array, args=(shared_array, i))
    for i in range(4)
]

for p in processes: p.start()
for p in processes: p.join()
print(list(shared_array))  # [0, 10, 20, 30]

# Manager - share complex objects
def update_dict(d, key, value):
    d[key] = value

manager = multiprocessing.Manager()
shared_dict = manager.dict()

processes = [
    multiprocessing.Process(target=update_dict, args=(shared_dict, i, i*10))
    for i in range(4)
]

for p in processes: p.start()
for p in processes: p.join()
print(dict(shared_dict))  # {0: 0, 1: 10, 2: 20, 3: 30}
```

#### Process Pools

```python
from multiprocessing import Pool
import time

def cpu_task(n):
    """CPU-intensive task"""
    total = 0
    for i in range(n):
        total += i ** 2
    return total

# Process pool
with Pool(processes=4) as pool:
    # map() - simple parallel execution
    results = pool.map(cpu_task, [10**6] * 8)
    print(f"Results: {results}")

# apply_async() - asynchronous execution
with Pool() as pool:
    async_results = [pool.apply_async(cpu_task, (10**6,)) for _ in range(8)]
    results = [r.get() for r in async_results]

# ✅ GOOD: CPU-bound parallelism
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Sequential
start = time.time()
results = [fibonacci(30) for _ in range(8)]
print(f"Sequential: {time.time() - start:.2f}s")  # ~8s

# Parallel
start = time.time()
with Pool() as pool:
    results = pool.map(fibonacci, [30] * 8)
print(f"Parallel: {time.time() - start:.2f}s")  # ~2s (4x speedup!)
```

### Concurrent.futures

#### Executor Interface

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import time

def task(n):
    time.sleep(1)
    return n * 2

# ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=5) as executor:
    # submit() - single task
    future = executor.submit(task, 10)
    result = future.result()  # Block until done
    print(result)
  
    # map() - multiple tasks
    results = executor.map(task, range(10))
    print(list(results))

# ProcessPoolExecutor (same interface!)
with ProcessPoolExecutor(max_workers=4) as executor:
    results = executor.map(task, range(10))
    print(list(results))

# Future methods
future = executor.submit(task, 5)

# Check status
print(future.done())  # False (running)
print(future.running())  # True

# Wait for result
result = future.result(timeout=5)  # Blocks up to 5 seconds

# Add callback
def callback(future):
    print(f"Task completed: {future.result()}")

future.add_done_callback(callback)

# as_completed() - process results as they finish
from concurrent.futures import as_completed

with ThreadPoolExecutor() as executor:
    futures = [executor.submit(task, i) for i in range(10)]
  
    for future in as_completed(futures):
        result = future.result()
        print(f"Completed: {result}")

# wait() - wait for multiple futures
from concurrent.futures import wait, FIRST_COMPLETED, ALL_COMPLETED

futures = [executor.submit(task, i) for i in range(10)]
done, pending = wait(futures, return_when=FIRST_COMPLETED)
print(f"Done: {len(done)}, Pending: {len(pending)}")
```

### Performance Comparison

#### Threading vs Multiprocessing vs Async

```python
import time
import threading
import multiprocessing
import asyncio
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# I/O-bound task
def io_bound(n):
    time.sleep(0.1)
    return n * 2

# CPU-bound task
def cpu_bound(n):
    return sum(i**2 for i in range(n))

# Benchmark I/O-bound (Threading wins)
def benchmark_io():
    data = list(range(100))
  
    # Sequential
    start = time.time()
    results = [io_bound(i) for i in data]
    print(f"Sequential I/O: {time.time() - start:.2f}s")  # ~10s
  
    # Threading
    start = time.time()
    with ThreadPoolExecutor(max_workers=10) as executor:
        results = list(executor.map(io_bound, data))
    print(f"Threading I/O: {time.time() - start:.2f}s")  # ~1s
  
    # Multiprocessing (overhead!)
    start = time.time()
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(io_bound, data))
    print(f"Multiprocessing I/O: {time.time() - start:.2f}s")  # ~3s

# Benchmark CPU-bound (Multiprocessing wins)
def benchmark_cpu():
    data = [10**6] * 8
  
    # Sequential
    start = time.time()
    results = [cpu_bound(i) for i in data]
    print(f"Sequential CPU: {time.time() - start:.2f}s")  # ~2s
  
    # Threading (GIL limits!)
    start = time.time()
    with ThreadPoolExecutor(max_workers=8) as executor:
        results = list(executor.map(cpu_bound, data))
    print(f"Threading CPU: {time.time() - start:.2f}s")  # ~2s (no speedup!)
  
    # Multiprocessing (true parallelism!)
    start = time.time()
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(cpu_bound, data))
    print(f"Multiprocessing CPU: {time.time() - start:.2f}s")  # ~0.5s (4x speedup!)

benchmark_io()
benchmark_cpu()

# Decision matrix:
"""
Task Type        | Best Choice      | Why
-----------------+------------------+-----------------------------------
I/O-bound        | Async/Await      | Lowest overhead, highest concurrency
I/O-bound (sync) | Threading        | Works with blocking libraries
CPU-bound        | Multiprocessing  | True parallelism (no GIL)
Mixed            | Async + Process  | Async for I/O, processes for CPU
"""
```

### Frequently Asked Questions

**Q1: When should I use threading vs multiprocessing?**

**A:**

```python
# Use threading for:
# - I/O-bound tasks (network, file, database)
# - Low to medium concurrency
# - Sharing data between threads

# Use multiprocessing for:
# - CPU-bound tasks (computation, data processing)
# - True parallelism needed
# - Independent tasks

# Example: Web scraper (I/O-bound)
import requests
from concurrent.futures import ThreadPoolExecutor

def fetch_url(url):
    return requests.get(url).text

urls = ['http://example.com'] * 100

# ✅ Threading: Perfect for I/O
with ThreadPoolExecutor(max_workers=20) as executor:
    results = executor.map(fetch_url, urls)

# ❌ Multiprocessing: Overkill, more overhead
with ProcessPoolExecutor() as executor:
    results = executor.map(fetch_url, urls)  # Slower!

# Example: Data processing (CPU-bound)
import pandas as pd

def process_data(data):
    # Heavy computation
    return data.apply(lambda x: x ** 2).sum()

datasets = [pd.DataFrame(range(10**6)) for _ in range(8)]

# ❌ Threading: GIL prevents speedup
with ThreadPoolExecutor() as executor:
    results = list(executor.map(process_data, datasets))  # No speedup!

# ✅ Multiprocessing: True parallelism
with ProcessPoolExecutor() as executor:
    results = list(executor.map(process_data, datasets))  # 4x faster!

# Guidelines:
# 1. Profile your code first
# 2. I/O-bound → Try async first, then threading
# 3. CPU-bound → Multiprocessing
# 4. Simple tasks → Keep it synchronous
```

**Why This Matters:** Choosing wrong concurrency model wastes resources or provides no speedup.

---

### Key Takeaways

**Concurrency & Parallelism:**

- **Threading**: OS-level concurrency, shared memory, GIL-limited
- **Multiprocessing**: True parallelism, separate memory, no GIL
- **Async/await**: Cooperative concurrency, single-threaded, efficient
- **concurrent.futures**: Unified interface for threads and processes
- **GIL**: Prevents CPU parallelism in threads, doesn't affect processes

**Use Cases:**

- I/O-bound + async libraries → async/await
- I/O-bound + blocking libraries → threading
- CPU-bound → multiprocessing
- High concurrency (10,000+) → async/await

**Performance:**

- Threading: Low overhead, shared memory, GIL-limited
- Multiprocessing: High overhead, isolated memory, true parallelism
- Async: Lowest overhead, highest concurrency, requires async libraries

---

## 2.7 Metaprogramming

**SDLC Phase:** Development, Frameworks

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (frameworks, DSLs)
- [X] Development
- [ ] Testing
- [ ] Deployment
- [X] Maintenance (dynamic behavior)

Metaprogramming enables writing code that manipulates code, creating flexible frameworks and domain-specific languages.

### Introspection

#### Type Inspection

```python
# type() - get object type
x = [1, 2, 3]
print(type(x))  # <class 'list'>

# isinstance() - check type
print(isinstance(x, list))  # True
print(isinstance(x, (list, tuple)))  # True

# issubclass() - check inheritance
print(issubclass(bool, int))  # True

# dir() - list attributes
print(dir(x))  # ['append', 'clear', 'copy', ...]

# vars() - get __dict__
class User:
    def __init__(self, name):
        self.name = name

user = User("Alice")
print(vars(user))  # {'name': 'Alice'}

# getattr/setattr/hasattr
print(getattr(user, 'name'))  # "Alice"
setattr(user, 'age', 30)
print(hasattr(user, 'age'))  # True

# ✅ GOOD: Dynamic attribute access
class Config:
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)

config = Config(host='localhost', port=5432)
print(config.host)  # 'localhost'
```

#### inspect Module

```python
import inspect

def example_function(a, b, c=10):
    """Example function"""
    return a + b + c

# Get signature
sig = inspect.signature(example_function)
print(sig)  # (a, b, c=10)

# Get parameters
for name, param in sig.parameters.items():
    print(f"{name}: default={param.default}")

# Get source code
print(inspect.getsource(example_function))

# Get docstring
print(inspect.getdoc(example_function))

# Check if callable
print(inspect.isfunction(example_function))  # True
print(inspect.ismethod(example_function))  # False

# Get call stack
def outer():
    def inner():
        frame = inspect.currentframe()
        print(inspect.getframeinfo(frame))
    inner()

outer()

# ✅ GOOD: Validate function arguments
def validate_signature(func):
    sig = inspect.signature(func)
  
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        # Validate bound arguments
        for name, value in bound.arguments.items():
            param = sig.parameters[name]
            if param.annotation != inspect.Parameter.empty:
                if not isinstance(value, param.annotation):
                    raise TypeError(f"{name} must be {param.annotation}")
      
        return func(*args, **kwargs)
  
    return wrapper

@validate_signature
def add(a: int, b: int) -> int:
    return a + b

print(add(10, 20))  # 30
# add(10, "20")  # TypeError!
```

### Metaclasses

#### type() as Metaclass

```python
# type() creates classes dynamically

# Normal class definition
class User:
    def __init__(self, name):
        self.name = name

# Equivalent using type()
User = type('User', (), {'__init__': lambda self, name: setattr(self, 'name', name)})

user = User("Alice")
print(user.name)  # "Alice"

# type() signature: type(name, bases, dict)
def __init__(self, name):
    self.name = name

def greet(self):
    return f"Hello, {self.name}"

# Create class with methods
User = type('User', (), {
    '__init__': __init__,
    'greet': greet
})

user = User("Bob")
print(user.greet())  # "Hello, Bob"

# Inheritance with type()
class Animal:
    def speak(self):
        return "Sound"

Dog = type('Dog', (Animal,), {
    'speak': lambda self: "Woof"
})

dog = Dog()
print(dog.speak())  # "Woof"
```

#### Custom Metaclasses

```python
# Metaclass controls class creation

class Meta(type):
    def __new__(mcs, name, bases, dct):
        print(f"Creating class: {name}")
      
        # Modify class before creation
        dct['added_by_meta'] = True
      
        # Create class
        cls = super().__new__(mcs, name, bases, dct)
        return cls
  
    def __init__(cls, name, bases, dct):
        print(f"Initializing class: {name}")
        super().__init__(name, bases, dct)

# Use metaclass
class MyClass(metaclass=Meta):
    pass

# Output:
# Creating class: MyClass
# Initializing class: MyClass

print(MyClass.added_by_meta)  # True

# ✅ GOOD: Singleton metaclass
class Singleton(type):
    _instances = {}
  
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    def __init__(self):
        self.connection = "Connected"

db1 = Database()
db2 = Database()
print(db1 is db2)  # True (same instance!)

# ✅ GOOD: Auto-register classes
class Registry(type):
    registry = {}
  
    def __new__(mcs, name, bases, dct):
        cls = super().__new__(mcs, name, bases, dct)
        if name != 'Base':  # Skip base class
            mcs.registry[name] = cls
        return cls

class Base(metaclass=Registry):
    pass

class Plugin1(Base):
    pass

class Plugin2(Base):
    pass

print(Registry.registry)  # {'Plugin1': <class 'Plugin1'>, 'Plugin2': <class 'Plugin2'>}

# ✅ GOOD: Enforce method implementation
class InterfaceMeta(type):
    def __new__(mcs, name, bases, dct):
        # Check required methods
        required = ['process', 'validate']
        for method in required:
            if method not in dct:
                raise TypeError(f"Must implement {method}")
      
        return super().__new__(mcs, name, bases, dct)

# class BadProcessor(metaclass=InterfaceMeta):  # TypeError!
#     pass

class GoodProcessor(metaclass=InterfaceMeta):
    def process(self):
        pass
  
    def validate(self):
        pass
```

### Descriptors

#### Descriptor Protocol

```python
# Descriptor: Object with __get__, __set__, or __delete__

class Descriptor:
    def __get__(self, instance, owner):
        print(f"__get__ called")
        return instance._value if instance else self
  
    def __set__(self, instance, value):
        print(f"__set__ called with {value}")
        instance._value = value
  
    def __delete__(self, instance):
        print("__delete__ called")
        del instance._value

class MyClass:
    attr = Descriptor()

obj = MyClass()
obj.attr = 10  # Calls __set__
print(obj.attr)  # Calls __get__
del obj.attr  # Calls __delete__

# Data descriptor (has __set__ or __delete__)
class DataDescriptor:
    def __set_name__(self, owner, name):
        self.name = name
  
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
  
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value

# Non-data descriptor (only __get__)
class NonDataDescriptor:
    def __get__(self, instance, owner):
        return 42

# Priority: instance dict > data descriptor > non-data descriptor

# ✅ GOOD: Validated attribute
class Validated:
    def __init__(self, validator):
        self.validator = validator
  
    def __set_name__(self, owner, name):
        self.name = f"_{name}"
  
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.name)
  
    def __set__(self, instance, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value: {value}")
        setattr(instance, self.name, value)

class User:
    age = Validated(lambda x: isinstance(x, int) and 0 <= x <= 150)
  
    def __init__(self, age):
        self.age = age

user = User(30)
# user.age = -10  # ValueError!
# user.age = "thirty"  # ValueError!
```

#### Property Implementation

```python
# property() is a descriptor

class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
  
    # Using property descriptor
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
  
    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

temp = Temperature(25)
print(temp.fahrenheit)  # 77.0
temp.fahrenheit = 32
print(temp._celsius)  # 0.0

# Equivalent descriptor implementation
class PropertyDescriptor:
    def __init__(self, fget=None, fset=None):
        self.fget = fget
        self.fset = fset
  
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.fget(instance)
  
    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("Can't set attribute")
        self.fset(instance, value)
  
    def setter(self, fset):
        return PropertyDescriptor(self.fget, fset)

class Temperature2:
    def __init__(self, celsius):
        self._celsius = celsius
  
    @PropertyDescriptor
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
  
    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9
```

### Decorators Deep Dive

#### Advanced Decorators

```python
from functools import wraps
import time

# Decorator with arguments
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def unstable_operation():
    import random
    if random.random() < 0.7:
        raise ValueError("Failed")
    return "Success"

# Class decorator
def singleton(cls):
    instances = {}
  
    @wraps(cls)
    def wrapper(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
  
    return wrapper

@singleton
class Database:
    pass

db1 = Database()
db2 = Database()
print(db1 is db2)  # True

# Decorator class
class Memoize:
    def __init__(self, func):
        self.func = func
        self.cache = {}
  
    def __call__(self, *args):
        if args not in self.cache:
            self.cache[args] = self.func(*args)
        return self.cache[args]

@Memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # Fast!

# ✅ GOOD: Stacked decorators
@timer
@retry(max_attempts=3)
@cache
def expensive_operation():
    pass

# Executed as: timer(retry(cache(expensive_operation)))
```

### Abstract Syntax Trees (AST)

#### Parsing Python Code

```python
import ast

# Parse Python code
code = """
def add(a, b):
    return a + b

result = add(10, 20)
"""

tree = ast.parse(code)
print(ast.dump(tree, indent=2))

# Visit nodes
class Visitor(ast.NodeVisitor):
    def visit_FunctionDef(self, node):
        print(f"Function: {node.name}")
        self.generic_visit(node)
  
    def visit_Call(self, node):
        if isinstance(node.func, ast.Name):
            print(f"Function call: {node.func.id}")
        self.generic_visit(node)

visitor = Visitor()
visitor.visit(tree)

# Transform AST
class Transformer(ast.NodeTransformer):
    def visit_Constant(self, node):
        # Double all integer constants
        if isinstance(node.value, int):
            node.value *= 2
        return node

transformer = Transformer()
new_tree = transformer.visit(tree)

# Compile and execute
code = compile(new_tree, '<ast>', 'exec')
exec(code)  # add() now adds doubled values!

# ✅ GOOD: Static analysis
def find_function_calls(code):
    tree = ast.parse(code)
    calls = []
  
    for node in ast.walk(tree):
        if isinstance(node, ast.Call):
            if isinstance(node.func, ast.Name):
                calls.append(node.func.id)
  
    return calls

code = """
print("Hello")
len([1, 2, 3])
custom_function()
"""

print(find_function_calls(code))  # ['print', 'len', 'custom_function']
```

### Key Takeaways

**Metaprogramming:**

- **Introspection**: Inspect objects at runtime (type, dir, inspect)
- **Metaclasses**: Control class creation (type, custom metaclasses)
- **Descriptors**: Control attribute access (__get__, __set__, __delete__)
- **Decorators**: Modify function/class behavior
- **AST**: Parse and transform Python code

**Use Cases:**

- Frameworks and ORMs (SQLAlchemy, Django)
- Validation and serialization (Pydantic, marshmallow)
- Testing frameworks (pytest fixtures)
- Domain-specific languages

**Cautions:**

- Increases complexity
- Harder to debug
- Can obscure code behavior
- Use only when benefits outweigh costs

---

## Part 2 Complete!

Congratulations! You've completed **Part 2: Python Internals & Advanced Language Features**. You now have deep knowledge of:

✅ **Section 2.1**: CPython Architecture
✅ **Section 2.2**: Type System & Type Hints
✅ **Section 2.3**: Iterators and Generators
✅ **Section 2.4**: Functional Programming
✅ **Section 2.5**: Asynchronous Programming
✅ **Section 2.6**: Concurrency & Parallelism
✅ **Section 2.7**: Metaprogramming

**Next Steps:**

Continue with **Part 3: SDLC - Requirements, Design & Architecture** to learn how to apply Python skills in real-world software development lifecycle.

**Topics in Part 3:**

- Requirements gathering and technical specifications
- Software architecture and design patterns
- API design and system scalability
- Database design and ORM patterns
- Microservices vs monolithic architecture

Keep practicing and building! 🚀
