---
title: "Python Mastery - Quick Reference Cheat Sheet"
date: 2025-01-04 00:00:00 +0530
categories: [Python, Reference, Cheatsheet]
tags: [python, cheatsheet, quick-reference, summary]
pin: false
---

# Python Mastery - Quick Reference Cheat Sheet

**One-page summary of essential Python concepts**

---

## 🐍 Python Fundamentals

### Data Types
```python
# Numbers
int: 42, 0b1010, 0o52, 0x2A
float: 3.14, 1e-3, float('inf')
complex: 3+4j

# Sequences
str: "text", 'text', """multi-line"""
list: [1, 2, 3], list(range(10))
tuple: (1, 2, 3), (1,)  # comma for single
bytes: b'data', bytes([65, 66])

# Collections
dict: {'a': 1, 'b': 2}, dict(a=1, b=2)
set: {1, 2, 3}, set([1, 2, 2])  # unique
frozenset: frozenset([1, 2, 3])  # immutable

# Boolean
bool: True, False, bool(x)
```

### Operators
```python
# Arithmetic: +, -, *, /, //, %, **
# Comparison: ==, !=, <, >, <=, >=
# Logical: and, or, not
# Membership: in, not in
# Identity: is, is not
# Bitwise: &, |, ^, ~, <<, >>
# Walrus: x := value  # (3.8+)
```

### Control Flow
```python
# Conditional
if condition:
    pass
elif other_condition:
    pass
else:
    pass

# Ternary
value = x if condition else y

# Match (3.10+)
match value:
    case 1: ...
    case [x, y]: ...
    case _: ...  # default

# Loops
for item in iterable:
    if condition: continue
    if other: break
else:  # no break
    ...

while condition:
    ...

# Comprehensions
[x*2 for x in range(10) if x%2 == 0]
{x: x**2 for x in range(5)}
{x for x in data if x > 0}
(x*2 for x in range(1000))  # generator
```

### Functions
```python
def func(pos_only, /, std, *, kwd_only, **kwargs):
    """Docstring"""
    return value

# Lambda
f = lambda x, y: x + y

# Decorators
@decorator
def func(): ...

# Type hints
def add(x: int, y: int) -> int:
    return x + y
```

---

## 🎯 Object-Oriented Programming

### Classes
```python
class MyClass:
    class_var = "shared"  # class variable
    
    def __init__(self, value):
        self.instance_var = value  # instance variable
    
    def method(self):
        return self.instance_var
    
    @classmethod
    def class_method(cls):
        return cls.class_var
    
    @staticmethod
    def static_method():
        return "no self/cls"
    
    @property
    def prop(self):
        return self._value
    
    @prop.setter
    def prop(self, value):
        self._value = value

# Inheritance
class Child(Parent):
    def __init__(self):
        super().__init__()

# Multiple inheritance
class Multi(Parent1, Parent2):
    pass  # MRO: left-to-right, depth-first

# Dataclasses (3.7+)
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
    
    def distance(self):
        return (self.x**2 + self.y**2)**0.5
```

### Magic Methods
```python
__init__    # Constructor
__str__     # str(obj)
__repr__    # repr(obj)
__len__     # len(obj)
__getitem__ # obj[key]
__setitem__ # obj[key] = value
__iter__    # for x in obj
__next__    # next(obj)
__enter__   # with obj:
__exit__    # context exit
__call__    # obj()
__eq__      # obj == other
__lt__      # obj < other
__add__     # obj + other
```

---

## ⚡ Advanced Features

### Generators & Iterators
```python
# Generator function
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Generator expression
squares = (x**2 for x in range(1000000))

# Iterator protocol
class MyIterator:
    def __iter__(self):
        return self
    
    def __next__(self):
        if condition:
            return value
        raise StopIteration
```

### Context Managers
```python
# Protocol
class Manager:
    def __enter__(self):
        return resource
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        cleanup()
        return False  # propagate exceptions

# contextlib
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire()
    try:
        yield resource
    finally:
        release(resource)

# Usage
with managed_resource() as r:
    use(r)
```

### Decorators
```python
# Function decorator
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        # before
        result = func(*args, **kwargs)
        # after
        return result
    return wrapper

# Parameterized decorator
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("Hello")

# Class decorator
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance
```

### Async/Await
```python
import asyncio

async def fetch_data(url):
    await asyncio.sleep(1)  # async operation
    return data

async def main():
    result = await fetch_data("url")
    
    # Concurrent execution
    results = await asyncio.gather(
        fetch_data("url1"),
        fetch_data("url2")
    )

# Run
asyncio.run(main())

# Async context manager
async with async_resource() as r:
    await r.operation()

# Async iterator
async for item in async_iterable:
    await process(item)
```

---

## 📦 Standard Library Essentials

### Collections
```python
from collections import (
    Counter,        # count hashable objects
    defaultdict,    # dict with default factory
    deque,         # double-ended queue
    namedtuple,    # tuple with named fields
    OrderedDict,   # dict that remembers order (< 3.7)
    ChainMap,      # combine multiple dicts
)

Counter('abracadabra')  # {'a': 5, 'b': 2, ...}
defaultdict(list)       # auto-create lists
deque([1, 2, 3])       # O(1) append/pop both ends
Point = namedtuple('Point', ['x', 'y'])
```

### Itertools
```python
from itertools import (
    count,          # infinite counting
    cycle,          # infinite cycling
    repeat,         # repeat value
    chain,          # concatenate iterables
    islice,         # slice iterator
    combinations,   # nCr
    permutations,   # nPr
    product,        # cartesian product
    groupby,        # group by key
)

count(10, 2)           # 10, 12, 14, ...
cycle([1, 2, 3])      # 1, 2, 3, 1, 2, 3, ...
chain([1, 2], [3, 4]) # 1, 2, 3, 4
combinations([1,2,3], 2) # (1,2), (1,3), (2,3)
```

### Functools
```python
from functools import (
    lru_cache,      # memoization
    wraps,          # preserve metadata
    partial,        # partial application
    reduce,         # reduce function
    total_ordering, # fill in comparison methods
)

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2: return n
    return fibonacci(n-1) + fibonacci(n-2)

add_five = partial(add, 5)  # fix first argument
```

### Pathlib
```python
from pathlib import Path

p = Path('file.txt')
p.read_text()          # read file
p.write_text('data')   # write file
p.exists()             # check existence
p.is_file()            # is file?
p.is_dir()             # is directory?
p.parent               # parent directory
p.stem                 # filename without extension
p.suffix               # extension
list(p.glob('*.py'))   # find files
```

---

## 🔧 Testing & Quality

### Pytest Basics
```python
# test_module.py
def test_function():
    assert 1 + 1 == 2

# Fixtures
@pytest.fixture
def data():
    return [1, 2, 3]

def test_with_fixture(data):
    assert len(data) == 3

# Parametrize
@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (2, 4),
])
def test_multiple(input, expected):
    assert input * 2 == expected

# Mock
from unittest.mock import Mock, patch

@patch('module.function')
def test_with_mock(mock_func):
    mock_func.return_value = 42
    assert module.function() == 42
```

### Type Hints
```python
from typing import (
    List, Dict, Set, Tuple,
    Optional, Union, Any,
    Callable, TypeVar, Generic,
)

def process(items: List[int]) -> Dict[str, int]:
    return {str(i): i for i in items}

# Optional (can be None)
def find(name: str) -> Optional[User]:
    ...

# Union (multiple types)
def parse(value: Union[int, str]) -> int:
    ...

# Callable
def apply(func: Callable[[int], str], x: int) -> str:
    return func(x)

# Generic
T = TypeVar('T')

def first(items: List[T]) -> T:
    return items[0]
```

---

## 🚀 Performance & Best Practices

### Timing
```python
import time
import timeit

# Simple timing
start = time.time()
operation()
elapsed = time.time() - start

# Accurate timing
timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)
```

### Profiling
```python
import cProfile
import pstats

# Profile code
cProfile.run('function()', 'output.prof')

# Analyze
stats = pstats.Stats('output.prof')
stats.sort_stats('cumulative')
stats.print_stats(10)

# Line profiler
@profile
def function():
    ...
```

### Memory
```python
import sys
import tracemalloc

# Object size
sys.getsizeof(object)

# Track allocations
tracemalloc.start()
# ... code ...
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
```

---

## 📝 Common Patterns

### Singleton
```python
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

### Factory
```python
class ShapeFactory:
    @staticmethod
    def create(shape_type):
        if shape_type == "circle":
            return Circle()
        elif shape_type == "square":
            return Square()
```

### Observer
```python
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def notify(self, data):
        for observer in self._observers:
            observer.update(data)
```

### Strategy
```python
class Context:
    def __init__(self, strategy):
        self._strategy = strategy
    
    def execute(self):
        return self._strategy.algorithm()
```

---

## 🎯 Quick Tips

### List Operations
```python
# Most common
list.append(x)           # add to end: O(1)
list.extend(iterable)    # add multiple: O(k)
list.insert(i, x)        # insert at i: O(n)
list.remove(x)           # remove first x: O(n)
list.pop(i)              # remove at i: O(n), O(1) for end
list.sort()              # in-place: O(n log n)
sorted(list)             # new list: O(n log n)

# Slicing
list[start:stop:step]
list[::-1]               # reverse
list[::2]                # every 2nd
```

### Dict Operations
```python
dict.get(key, default)   # safe access
dict.setdefault(key, default)
dict.pop(key, default)
dict.update(other)
dict.keys()
dict.values()
dict.items()

# Dict comprehension
{k: v for k, v in items if condition}
```

### String Operations
```python
s.strip()                # remove whitespace
s.split(sep)             # split by separator
s.join(iterable)         # join strings
s.replace(old, new)      # replace substring
s.startswith(prefix)
s.endswith(suffix)
s.lower(), s.upper()
f"formatted {value}"     # f-strings (3.6+)
```

### File I/O
```python
# Read
with open('file.txt') as f:
    content = f.read()           # all
    lines = f.readlines()        # list
    for line in f:               # iterate

# Write
with open('file.txt', 'w') as f:
    f.write(text)
    f.writelines(lines)

# JSON
import json
json.loads(string)       # parse JSON
json.dumps(obj)          # to JSON
json.load(file)          # from file
json.dump(obj, file)     # to file
```

---

## ⚡ Performance Quick Wins

```python
# Use sets for membership testing
if x in my_set:          # O(1) vs O(n) for list

# Use dict for lookups
value = lookup_dict[key] # O(1) vs O(n) search

# List comprehensions > loops
[x*2 for x in items]     # faster than loop

# Generators for large data
(x*2 for x in huge_list) # lazy evaluation

# Use join for strings
''.join(strings)         # vs concatenation

# Cache repeated calls
@lru_cache(maxsize=128)
def expensive_func(n):
    ...
```

---

## 🔍 Common Gotchas

```python
# Mutable default arguments
def func(lst=[]):        # DON'T
    lst.append(1)
    return lst

def func(lst=None):      # DO
    if lst is None:
        lst = []
    lst.append(1)
    return lst

# Late binding in loops
funcs = [lambda: i for i in range(3)]
# All return 2 (late binding)

funcs = [lambda i=i: i for i in range(3)]
# Correct (default argument)

# is vs ==
x == y  # value equality
x is y  # identity (same object)

# Copy vs reference
new_list = old_list      # reference
new_list = old_list[:]   # shallow copy
new_list = old_list.copy()
new_list = list(old_list)

import copy
new = copy.deepcopy(old) # deep copy
```

---

## 🎓 Interview Essentials

### Time Complexity
```
O(1)      - Constant    - dict access, list append
O(log n)  - Logarithmic - binary search
O(n)      - Linear      - list iteration
O(n log n)- Linearithmic- sorted(), merge sort
O(n²)     - Quadratic   - nested loops
O(2ⁿ)     - Exponential - recursive fibonacci
```

### Common Algorithms
```python
# Binary search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# Two pointers
def two_sum_sorted(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        current = arr[left] + arr[right]
        if current == target:
            return [left, right]
        elif current < target:
            left += 1
        else:
            right -= 1

# Sliding window
def max_sum_subarray(arr, k):
    max_sum = sum(arr[:k])
    window_sum = max_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i-k]
        max_sum = max(max_sum, window_sum)
    return max_sum
```

---

*Quick Reference Cheat Sheet v1.0*  
*Print this. Keep it handy. Master Python.*  
*Created: January 4, 2025*
