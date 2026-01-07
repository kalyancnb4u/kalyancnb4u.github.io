---
title: "Python Mastery Part 9: Package Management, Distribution & Ecosystem"
date: 2024-12-10 00:00:00 +0530
categories: [Python, Language Mastery]
tags: [Python, Packages, Distribution, PyPI, Poetry, Dependencies, Setup-tools, Eco-system]
---
## Table of Contents

- [Introduction](#introduction)
- [9.1 Package Structure &amp; Creation](#91-package-structure--creation)
  - [Package Organization](#package-organization)
  - [Setup Configuration](#setup-configuration)
  - [Modern Packaging](#modern-packaging)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [9.2 Dependency Management](#92-dependency-management)
  - [Virtual Environments](#virtual-environments)
  - [Requirements Files](#requirements-files)
  - [Poetry for Dependency Management](#poetry-for-dependency-management)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
- [9.3 Publishing to PyPI](#93-publishing-to-pypi)
  - [Building Distributions](#building-distributions)
  - [Publishing Packages](#publishing-packages)
  - [Versioning Strategy](#versioning-strategy)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
- [9.4 Python Ecosystem](#94-python-ecosystem)
- [9.5 Best Practices](#95-best-practices)

---

# Complete Python Mastery Part 9: Package Management, Distribution & Ecosystem

## Introduction

Welcome to **Part 9** of the Complete Python Mastery series. This part covers Python package creation, distribution, and the broader Python ecosystem.

**What You'll Learn in Part 9:**

This post covers package management and distribution:

- **Package structure**: Organizing Python packages
- **Modern packaging**: pyproject.toml, Poetry, setuptools
- **Dependency management**: Virtual environments, lock files
- **Publishing**: Building and uploading to PyPI
- **Versioning**: Semantic versioning strategies
- **Python ecosystem**: Key libraries and frameworks

**Why This Matters:**

Package management enables:

- Code reusability across projects
- Easy sharing with team/community
- Version control of dependencies
- Reproducible environments
- Professional code distribution

**Prerequisites:**

- Parts 1-8 of this series
- Python fundamentals
- Basic Git knowledge

Let's master Python packaging! 🚀

---

## 9.1 Package Structure & Creation

**SDLC Phase:** Development, Distribution

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (code organization)
- [X] Development (package creation)
- [ ] Testing
- [X] Deployment (distribution)
- [X] Maintenance (versioning)

### Package Organization

```python
"""
PACKAGE STRUCTURE: Organizing Python code

Basic structure:
mypackage/
├── mypackage/           # Package directory (same name)
│   ├── __init__.py      # Makes it a package
│   ├── module1.py       # Module
│   ├── module2.py       # Module
│   └── subpackage/      # Sub-package
│       ├── __init__.py
│       └── submodule.py
├── tests/               # Test directory
│   ├── __init__.py
│   ├── test_module1.py
│   └── test_module2.py
├── docs/                # Documentation
│   └── index.md
├── README.md           # Project description
├── LICENSE             # License file
├── setup.py            # Package metadata (legacy)
├── pyproject.toml      # Modern package metadata
└── requirements.txt    # Dependencies
"""

# Example package structure
"""
mymath/
├── mymath/
│   ├── __init__.py
│   ├── basic.py
│   ├── advanced.py
│   └── utils/
│       ├── __init__.py
│       └── helpers.py
├── tests/
│   ├── test_basic.py
│   └── test_advanced.py
├── README.md
├── LICENSE
└── pyproject.toml
"""

# mymath/__init__.py
"""
Package initialization file

Purpose:
1. Mark directory as package
2. Define package-level API
3. Import commonly used items
"""

# Import from submodules
from .basic import add, subtract
from .advanced import power, factorial

# Package metadata
__version__ = '1.0.0'
__author__ = 'Your Name'

# Define public API (used by *)
__all__ = ['add', 'subtract', 'power', 'factorial']

# mymath/basic.py
"""Basic math operations"""

def add(a, b):
    """Add two numbers"""
    return a + b

def subtract(a, b):
    """Subtract two numbers"""
    return a - b

def multiply(a, b):
    """Multiply two numbers"""
    return a * b

def divide(a, b):
    """Divide two numbers"""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# mymath/advanced.py
"""Advanced math operations"""

def power(base, exponent):
    """Calculate power"""
    return base ** exponent

def factorial(n):
    """Calculate factorial"""
    if n < 0:
        raise ValueError("Factorial not defined for negative numbers")
    if n == 0 or n == 1:
        return 1
    return n * factorial(n - 1)

# mymath/utils/__init__.py
"""Utility functions"""

from .helpers import is_even, is_prime

__all__ = ['is_even', 'is_prime']

# mymath/utils/helpers.py
"""Helper functions"""

def is_even(n):
    """Check if number is even"""
    return n % 2 == 0

def is_prime(n):
    """Check if number is prime"""
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

# Using the package
"""
# Installation
pip install mymath

# Import and use
import mymath
result = mymath.add(5, 3)  # 8

# Import specific functions
from mymath import add, power
result = add(5, 3)  # 8

# Import everything (not recommended)
from mymath import *
# Only imports what's in __all__

# Access version
print(mymath.__version__)  # 1.0.0
"""
```

### Setup Configuration

```python
"""
SETUP.PY: Legacy package configuration

Still widely used, but pyproject.toml is preferred
"""

# setup.py
from setuptools import setup, find_packages

# Read long description from README
with open("README.md", "r", encoding="utf-8") as fh:
    long_description = fh.read()

# Read requirements
with open("requirements.txt", "r", encoding="utf-8") as fh:
    requirements = [line.strip() for line in fh if line.strip() and not line.startswith("#")]

setup(
    # Package metadata
    name="mymath",
    version="1.0.0",
    author="Your Name",
    author_email="your.email@example.com",
    description="A simple math library",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/yourusername/mymath",
  
    # Package discovery
    packages=find_packages(exclude=["tests", "docs"]),
  
    # Dependencies
    install_requires=requirements,
  
    # Optional dependencies
    extras_require={
        'dev': ['pytest>=7.0', 'black>=23.0', 'flake8>=6.0'],
        'docs': ['sphinx>=5.0', 'sphinx-rtd-theme>=1.0'],
    },
  
    # Python version requirement
    python_requires='>=3.8',
  
    # Classifiers (for PyPI)
    classifiers=[
        "Development Status :: 4 - Beta",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.8",
        "Programming Language :: Python :: 3.9",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
    ],
  
    # Entry points (for CLI tools)
    entry_points={
        'console_scripts': [
            'mymath=mymath.cli:main',
        ],
    },
  
    # Include additional files
    include_package_data=True,
  
    # License
    license="MIT",
  
    # Keywords for PyPI search
    keywords="math mathematics calculator",
)

# setup.cfg (alternative configuration)
"""
[metadata]
name = mymath
version = 1.0.0
author = Your Name
author_email = your.email@example.com
description = A simple math library
long_description = file: README.md
long_description_content_type = text/markdown
url = https://github.com/yourusername/mymath
license = MIT
classifiers =
    Development Status :: 4 - Beta
    License :: OSI Approved :: MIT License
    Programming Language :: Python :: 3

[options]
packages = find:
python_requires = >=3.8
install_requires =
    numpy>=1.20.0
    requests>=2.28.0

[options.packages.find]
exclude =
    tests
    docs

[options.extras_require]
dev =
    pytest>=7.0
    black>=23.0
docs =
    sphinx>=5.0
"""

# MANIFEST.in (include non-Python files)
"""
include README.md
include LICENSE
include requirements.txt
recursive-include mymath/data *
recursive-include docs *.rst
global-exclude __pycache__
global-exclude *.py[co]
"""
```

### Modern Packaging

```toml
# pyproject.toml - Modern packaging standard (PEP 517/518)
"""
PYPROJECT.TOML: Modern Python packaging

Benefits:
✅ Single source of truth
✅ Tool-independent
✅ Declarative configuration
✅ Poetry/Flit/Hatchling support
"""

[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mymath"
version = "1.0.0"
description = "A simple math library"
readme = "README.md"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
license = {text = "MIT"}
requires-python = ">=3.8"
classifiers = [
    "Development Status :: 4 - Beta",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
]
keywords = ["math", "mathematics", "calculator"]

dependencies = [
    "numpy>=1.20.0",
    "requests>=2.28.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "black>=23.0",
    "flake8>=6.0",
    "mypy>=1.0",
]
docs = [
    "sphinx>=5.0",
    "sphinx-rtd-theme>=1.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/mymath"
Documentation = "https://mymath.readthedocs.io"
Repository = "https://github.com/yourusername/mymath"
"Bug Tracker" = "https://github.com/yourusername/mymath/issues"

[project.scripts]
mymath = "mymath.cli:main"

[tool.setuptools.packages.find]
where = ["."]
include = ["mymath*"]
exclude = ["tests*", "docs*"]

# Tool configurations in same file

[tool.black]
line-length = 88
target-version = ['py38', 'py39', 'py310', 'py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | build
  | dist
)/
'''

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--cov=mymath",
    "--cov-report=term-missing",
    "--cov-report=html",
]

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false

[tool.isort]
profile = "black"
line_length = 88
```

```python
# Dynamic versioning (single source of truth)

# Option 1: Version in __init__.py
# mymath/__init__.py
__version__ = '1.0.0'

# setup.py reads from __init__.py
import re
from pathlib import Path

def get_version():
    init_file = Path(__file__).parent / 'mymath' / '__init__.py'
    version_match = re.search(
        r"^__version__ = ['\"]([^'\"]*)['\"]",
        init_file.read_text(),
        re.M
    )
    if version_match:
        return version_match.group(1)
    raise RuntimeError("Unable to find version string")

setup(
    name="mymath",
    version=get_version(),
    # ...
)

# Option 2: Use setuptools_scm (Git-based versioning)
# pyproject.toml
"""
[build-system]
requires = ["setuptools>=61.0", "setuptools_scm>=8.0"]
build-backend = "setuptools.build_meta"

[tool.setuptools_scm]
write_to = "mymath/_version.py"
"""

# Version is automatically determined from Git tags
# Tag: v1.0.0 → Version: 1.0.0
```

### Frequently Asked Questions

**Q1: What's the difference between a module and a package?**

**A:**

```python
"""
MODULE vs PACKAGE

Module: Single Python file
Package: Directory with __init__.py

Examples:
"""

# MODULE: Single file
# math_utils.py
def add(a, b):
    return a + b

# Usage
import math_utils
result = math_utils.add(5, 3)

# PACKAGE: Directory structure
"""
mypackage/
├── __init__.py
├── module1.py
└── module2.py
"""

# Usage
import mypackage.module1
# Or
from mypackage import module1

# Key differences:

"""
Aspect          Module              Package
----------------------------------------------
Structure       Single file         Directory with __init__.py
Imports         import module       import package.module
Organization    Simple              Hierarchical
Use case        Small utilities     Large libraries

When to use:

Module (single file):
✅ Simple utility functions
✅ Small, focused functionality
✅ Standalone script

Package (directory):
✅ Large codebase
✅ Multiple related modules
✅ Need hierarchical organization
✅ Professional library
"""

# Example: requests library (package)
"""
requests/
├── __init__.py
├── api.py          # High-level API
├── models.py       # Response/Request models
├── sessions.py     # Session management
├── adapters.py     # HTTP adapters
└── auth.py         # Authentication
"""

# Example: json library (module - built-in)
# Single file: json.py
import json  # Imports the module

# Namespace packages (advanced)
# Multiple packages can contribute to same namespace
"""
project1/
└── mycompany/
    └── web/
        └── __init__.py

project2/
└── mycompany/
    └── data/
        └── __init__.py

# Both contribute to 'mycompany' namespace
from mycompany.web import app
from mycompany.data import database
"""
```

**Why This Matters:** Understanding structure helps organize code professionally.

---

### Interview Questions

**Question 1: How would you design a Python package for a team of 10 developers working on a large web application?**

**Difficulty:** Senior-Level

**SDLC Relevance:** System Design, Development

**Answer:**

```python
"""
LARGE-SCALE PACKAGE DESIGN

Requirements:
- 10 developers
- Large web application
- Maintainability
- Scalability
- Clear boundaries

Recommended structure:
"""

# Project structure
"""
webapp/
├── pyproject.toml              # Modern packaging config
├── README.md                   # Project documentation
├── .github/
│   └── workflows/
│       ├── ci.yml              # CI pipeline
│       └── release.yml         # Release automation
├── docs/                       # Sphinx documentation
│   ├── conf.py
│   ├── index.rst
│   └── api/
├── src/                        # Source code (src layout)
│   └── webapp/
│       ├── __init__.py
│       ├── api/                # API layer (routes)
│       │   ├── __init__.py
│       │   ├── v1/
│       │   │   ├── __init__.py
│       │   │   ├── users.py
│       │   │   ├── products.py
│       │   │   └── orders.py
│       │   └── v2/
│       ├── core/               # Business logic
│       │   ├── __init__.py
│       │   ├── auth/
│       │   │   ├── __init__.py
│       │   │   ├── service.py
│       │   │   └── models.py
│       │   ├── users/
│       │   ├── products/
│       │   └── orders/
│       ├── models/             # Data models
│       │   ├── __init__.py
│       │   ├── user.py
│       │   ├── product.py
│       │   └── order.py
│       ├── services/           # External services
│       │   ├── __init__.py
│       │   ├── email.py
│       │   ├── payment.py
│       │   └── storage.py
│       ├── utils/              # Shared utilities
│       │   ├── __init__.py
│       │   ├── validators.py
│       │   ├── formatters.py
│       │   └── helpers.py
│       ├── config.py           # Configuration
│       ├── exceptions.py       # Custom exceptions
│       └── constants.py        # Constants
├── tests/                      # Mirror src structure
│   ├── unit/
│   │   ├── core/
│   │   ├── api/
│   │   └── services/
│   ├── integration/
│   │   ├── test_api_integration.py
│   │   └── test_database_integration.py
│   └── e2e/
│       └── test_user_flow.py
├── scripts/                    # Dev/ops scripts
│   ├── migrate_db.py
│   ├── seed_data.py
│   └── deploy.py
└── requirements/               # Separated dependencies
    ├── base.txt                # Production dependencies
    ├── dev.txt                 # Development dependencies
    └── test.txt                # Testing dependencies
"""

# Design principles:

# 1. Src layout (prevents accidental imports)
"""
Benefits:
✅ Tests import from installed package (not source)
✅ Catches packaging issues early
✅ Cleaner imports
"""

# 2. Clear layer separation
"""
api/        → Routes and request handling
core/       → Business logic (domain layer)
models/     → Data models
services/   → External integrations
utils/      → Shared utilities

Dependencies flow:
api → core → models
api → services
services → core
"""

# 3. Feature-based organization (for core)
"""
core/
├── auth/           # Authentication feature
│   ├── service.py  # Business logic
│   ├── models.py   # Auth-specific models
│   └── utils.py    # Auth utilities
├── users/          # User feature
└── orders/         # Orders feature

Benefits:
✅ Easy to find related code
✅ Clear ownership (team can own feature)
✅ Easier to extract to microservice later
"""

# 4. Configuration management
# webapp/config.py
from pydantic import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    """Application settings"""
    # Database
    DATABASE_URL: str
    DATABASE_POOL_SIZE: int = 10
  
    # Redis
    REDIS_URL: str
  
    # Authentication
    SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
  
    # External services
    STRIPE_API_KEY: str
    SENDGRID_API_KEY: str
  
    # Environment
    ENVIRONMENT: str = "development"
    DEBUG: bool = False
  
    class Config:
        env_file = ".env"
        case_sensitive = False

@lru_cache()
def get_settings() -> Settings:
    """Get cached settings"""
    return Settings()

# Usage
from webapp.config import get_settings
settings = get_settings()

# 5. Dependency injection
# webapp/core/users/service.py
from typing import Protocol

class UserRepository(Protocol):
    """User repository interface"""
    def get_by_id(self, user_id: int): ...
    def create(self, user_data: dict): ...

class EmailService(Protocol):
    """Email service interface"""
    def send_welcome_email(self, email: str): ...

class UserService:
    """User service with dependency injection"""
  
    def __init__(
        self,
        repository: UserRepository,
        email_service: EmailService
    ):
        self.repository = repository
        self.email_service = email_service
  
    def create_user(self, user_data: dict):
        """Create user and send welcome email"""
        user = self.repository.create(user_data)
        self.email_service.send_welcome_email(user.email)
        return user

# Benefits:
"""
✅ Easy to test (inject mocks)
✅ Loose coupling
✅ Flexible (swap implementations)
✅ Clear dependencies
"""

# 6. Team workflows

# Code ownership (CODEOWNERS file)
"""
# Global reviewers
*                       @team-leads

# Feature ownership
/src/webapp/core/auth/  @auth-team
/src/webapp/core/users/ @users-team
/src/webapp/core/orders/@orders-team

# API ownership
/src/webapp/api/        @api-team

# Infrastructure
/scripts/               @devops-team
/.github/               @devops-team
"""

# Branch strategy: Feature branches
"""
main         → Production (protected)
develop      → Integration (protected)
feature/*    → Feature branches
bugfix/*     → Bug fixes
release/*    → Release preparation
"""

# 7. Documentation
"""
docs/
├── architecture.md      # System architecture
├── setup.md            # Development setup
├── contributing.md     # Contribution guide
├── api/               # API documentation
│   ├── authentication.md
│   ├── users.md
│   └── orders.md
└── deployment.md      # Deployment guide
"""

# Key decisions explained:

"""
1. Src layout: Catches packaging issues early
2. Feature-based: Easy navigation, clear ownership
3. Layer separation: Maintainable, testable
4. Config management: Single source of config
5. DI: Testable, flexible
6. Clear docs: Onboarding, maintenance
7. Code ownership: Accountability, expertise
"""
```

**Key Points:**

- Use src/ layout for larger projects
- Feature-based organization in core/
- Clear layer separation
- Dependency injection for testability
- Configuration management
- Documentation and code ownership
- Scalable for 10+ developers

---

(Part 9 continues with sections 9.2-9.5...)

## 9.2 Dependency Management

**SDLC Phase:** Development, Deployment

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (dependencies)
- [X] Testing
- [X] Deployment (reproducibility)
- [X] Maintenance (updates)

### Virtual Environments

```bash
# VIRTUAL ENVIRONMENTS: Isolated Python environments

# Why use virtual environments?
"""
✅ Isolate project dependencies
✅ Avoid version conflicts
✅ Reproducible environments
✅ Multiple Python versions
✅ Clean system Python
"""

# venv (built-in, Python 3.3+)

# Create virtual environment
python -m venv myenv

# Activate (Linux/Mac)
source myenv/bin/activate

# Activate (Windows)
myenv\Scripts\activate

# Deactivate
deactivate

# virtualenv (third-party, more features)

# Install
pip install virtualenv

# Create with specific Python version
virtualenv -p python3.11 myenv

# Create with system packages
virtualenv --system-site-packages myenv

# conda (Anaconda/Miniconda)

# Create environment
conda create -n myenv python=3.11

# Activate
conda activate myenv

# Deactivate
conda deactivate

# Install packages
conda install numpy pandas

# Export environment
conda env export > environment.yml

# Create from file
conda env create -f environment.yml

# Poetry (modern, includes dependency management)

# Create new project with venv
poetry new myproject

# Install dependencies (creates venv automatically)
poetry install

# Activate venv
poetry shell

# Run command in venv
poetry run python script.py
```

### Requirements Files

```python
"""
REQUIREMENTS.TXT: Dependency specification

Types:
- requirements.txt: Production dependencies
- requirements-dev.txt: Development dependencies
- requirements-test.txt: Testing dependencies
"""

# requirements.txt (production)
"""
# Core dependencies
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0

# Database
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
alembic==1.12.1

# Caching
redis==5.0.1

# Authentication
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4

# External services
requests==2.31.0
boto3==1.29.7

# Utilities
python-dotenv==1.0.0
"""

# requirements-dev.txt (development)
"""
# Include production requirements
-r requirements.txt

# Testing
pytest==7.4.3
pytest-cov==4.1.0
pytest-asyncio==0.21.1
pytest-mock==3.12.0

# Code quality
black==23.11.0
flake8==6.1.0
isort==5.12.0
mypy==1.7.1

# Development tools
ipython==8.17.2
jupyter==1.0.0

# Documentation
sphinx==7.2.6
sphinx-rtd-theme==2.0.0
"""

# Version pinning strategies

# 1. Exact versions (most reproducible)
"""
fastapi==0.104.1
# Pros: Reproducible, no surprises
# Cons: No security updates
"""

# 2. Compatible versions (recommended)
"""
fastapi~=0.104.0  # >=0.104.0, <0.105.0
# Pros: Gets patch updates
# Cons: Might break rarely
"""

# 3. Minimum versions (flexible)
"""
fastapi>=0.104.0
# Pros: Gets all updates
# Cons: Might break
"""

# 4. Range versions
"""
fastapi>=0.104.0,<0.106.0
# Pros: Controlled updates
# Cons: More complex
"""

# Lock files (exact versions)

# Generate from requirements.txt
"""
# Install pip-tools
pip install pip-tools

# Generate lock file
pip-compile requirements.in -o requirements.txt

# Upgrade
pip-compile --upgrade requirements.in
"""

# requirements.in (loose)
"""
fastapi
uvicorn[standard]
sqlalchemy
"""

# requirements.txt (generated, pinned)
"""
fastapi==0.104.1
  via -r requirements.in
uvicorn[standard]==0.24.0
  via -r requirements.in
starlette==0.27.0
  via fastapi
# ... all transitive dependencies pinned
"""

# Best practices

# Separate requirements by environment
"""
requirements/
├── base.txt        # Shared dependencies
├── dev.txt         # Development
├── test.txt        # Testing
└── prod.txt        # Production-specific
"""

# base.txt
"""
fastapi~=0.104.0
sqlalchemy~=2.0.0
"""

# dev.txt
"""
-r base.txt

black~=23.0.0
pytest~=7.4.0
"""

# prod.txt
"""
-r base.txt

gunicorn~=21.2.0
"""

# Install for dev
# $ pip install -r requirements/dev.txt
```

### Poetry for Dependency Management

```toml
# pyproject.toml with Poetry
"""
POETRY: Modern dependency management

Features:
✅ Dependency resolution
✅ Lock files (poetry.lock)
✅ Virtual environment management
✅ Build and publish
✅ Version bumping
"""

[tool.poetry]
name = "myproject"
version = "1.0.0"
description = "My awesome project"
authors = ["Your Name <email@example.com>"]
readme = "README.md"
homepage = "https://github.com/user/myproject"
repository = "https://github.com/user/myproject"
keywords = ["web", "api"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.8"  # Python version requirement
fastapi = "^0.104.0"  # Caret (^): >=0.104.0, <1.0.0
uvicorn = {extras = ["standard"], version = "^0.24.0"}
sqlalchemy = "~2.0.0"  # Tilde (~): >=2.0.0, <2.1.0

# Optional dependencies
redis = {version = "^5.0.0", optional = true}

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
black = "^23.0.0"
mypy = "^1.7.0"

[tool.poetry.group.test.dependencies]
pytest-cov = "^4.1.0"
pytest-asyncio = "^0.21.0"

[tool.poetry.group.docs.dependencies]
sphinx = "^7.2.0"

[tool.poetry.extras]
redis = ["redis"]  # Install with: poetry install -E redis

[tool.poetry.scripts]
myapp = "myproject.cli:main"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

```bash
# Poetry commands

# Install Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Create new project
poetry new myproject

# Initialize existing project
poetry init

# Add dependency
poetry add fastapi

# Add dev dependency
poetry add --group dev pytest

# Add with version constraint
poetry add "fastapi>=0.104.0,<0.106.0"

# Add from Git
poetry add git+https://github.com/user/repo.git

# Remove dependency
poetry remove fastapi

# Update dependencies
poetry update

# Update specific package
poetry update fastapi

# Install dependencies
poetry install

# Install without dev dependencies
poetry install --without dev

# Install with extras
poetry install -E redis

# Show dependencies
poetry show

# Show outdated
poetry show --outdated

# Export requirements.txt
poetry export -f requirements.txt -o requirements.txt

# Build package
poetry build

# Publish to PyPI
poetry publish

# Version bumping
poetry version patch  # 1.0.0 → 1.0.1
poetry version minor  # 1.0.0 → 1.1.0
poetry version major  # 1.0.0 → 2.0.0
```

### Frequently Asked Questions

**Q1: How do I handle conflicting dependencies?**

**A:**

```python
"""
DEPENDENCY CONFLICTS: Resolution strategies

Scenario: Package A needs library X>=1.0,<2.0
          Package B needs library X>=2.0,<3.0
          Conflict!

Solutions:
"""

# 1. Update packages (best solution)
"""
# Update Package A to support X>=2.0
# Or downgrade Package B to support X<2.0
"""

# 2. Use Poetry (automatic resolution)
"""
Poetry resolves conflicts automatically
Finds compatible versions for all dependencies

$ poetry add package-a package-b
# Poetry finds compatible X version or reports conflict
"""

# 3. Create constraints file
# constraints.txt
"""
library-x==1.5.0  # Specific version that works for both
"""

# Install with constraints
"""
$ pip install -r requirements.txt -c constraints.txt
"""

# 4. Separate environments (last resort)
"""
# Environment 1: Package A
venv1/
└── package-a (uses library-x 1.x)

# Environment 2: Package B
venv2/
└── package-b (uses library-x 2.x)
"""

# 5. Fork and patch
"""
If no compatible versions exist:
1. Fork Package A
2. Update to support library-x 2.x
3. Use your fork: pip install git+https://github.com/you/package-a.git
"""

# Prevention:

# Use loose version constraints
"""
# ❌ BAD: Too restrictive
package-x==1.2.3

# ✅ GOOD: Allows updates
package-x~=1.2  # >=1.2.0, <2.0.0
package-x^=1.2  # >=1.2.0, <2.0.0 (Poetry)
"""

# Regular dependency audits
"""
# Check for conflicts
$ pip check

# Update regularly
$ poetry update

# Review poetry.lock changes in PRs
"""

# Example: Real conflict resolution
"""
Problem:
- celery==5.2.0 requires kombu>=5.2.0,<6.0
- celery-redbeat==2.0.0 requires kombu>=5.0.0,<5.3.0
- Conflict on kombu!

Solution:
1. Check celery-redbeat releases
2. Found celery-redbeat==2.1.0 supports kombu<6.0
3. Update: poetry add celery-redbeat@^2.1.0
4. Resolved! kombu==5.2.7 works for both
"""
```

**Why This Matters:** Unresolved conflicts break builds and deployments.

---

## 9.3 Publishing to PyPI

**SDLC Phase:** Distribution, Release

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [ ] Development
- [ ] Testing
- [X] Deployment (publishing)
- [X] Maintenance (versioning)

### Building Distributions

```bash
# BUILD DISTRIBUTIONS: Creating installable packages

# Types:
# - Source distribution (sdist): .tar.gz
# - Wheel distribution (bdist_wheel): .whl

# Using setuptools

# Build both
python setup.py sdist bdist_wheel

# Using build (modern, recommended)
pip install build

# Build
python -m build

# Output:
"""
dist/
├── mypackage-1.0.0.tar.gz  # Source distribution
└── mypackage-1.0.0-py3-none-any.whl  # Wheel
"""

# Using Poetry
poetry build

# Test installation locally
pip install dist/mypackage-1.0.0-py3-none-any.whl

# Wheel naming convention
"""
mypackage-1.0.0-py3-none-any.whl
│         │     │   │    │
│         │     │   │    └── Platform (any, linux, win)
│         │     │   └── ABI (none, cp38, cp39)
│         │     └── Python version (py3, py38, py39)
│         └── Version
└── Package name
"""

# Platform-specific wheels
"""
mypackage-1.0.0-cp311-cp311-linux_x86_64.whl  # Linux, Python 3.11
mypackage-1.0.0-cp311-cp311-win_amd64.whl     # Windows, Python 3.11
mypackage-1.0.0-py3-none-any.whl              # Universal (pure Python)
"""
```

### Publishing Packages

```bash
# PUBLISHING TO PYPI

# 1. Create PyPI account
# https://pypi.org/account/register/

# 2. Create API token
# https://pypi.org/manage/account/token/

# 3. Configure credentials
# ~/.pypirc
"""
[pypi]
username = __token__
password = pypi-AgEIcHl...  # Your API token
"""

# 4. Install twine
pip install twine

# 5. Upload to TestPyPI (test first!)
twine upload --repository testpypi dist/*

# 6. Test installation
pip install --index-url https://test.pypi.org/simple/ mypackage

# 7. Upload to PyPI
twine upload dist/*

# With Poetry
# Configure token
poetry config pypi-token.pypi pypi-AgEIcHl...

# Publish
poetry publish --build

# Automated publishing with GitHub Actions
```

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
    
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
    
      - name: Install dependencies
        run: |
          pip install build twine
    
      - name: Build package
        run: python -m build
    
      - name: Publish to PyPI
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
        run: twine upload dist/*

# Usage:
# 1. Add PYPI_API_TOKEN to GitHub secrets
# 2. Create GitHub release
# 3. Package automatically published to PyPI
```

### Versioning Strategy

```python
"""
SEMANTIC VERSIONING (SemVer)

Format: MAJOR.MINOR.PATCH

MAJOR: Breaking changes (1.0.0 → 2.0.0)
MINOR: New features, backward compatible (1.0.0 → 1.1.0)
PATCH: Bug fixes, backward compatible (1.0.0 → 1.0.1)

Examples:
"""

# Version progression
"""
0.1.0  → Initial development
0.2.0  → Add features
0.3.0  → More features
1.0.0  → First stable release
1.0.1  → Bug fix
1.1.0  → New feature
1.1.1  → Bug fix
2.0.0  → Breaking change
"""

# Pre-release versions
"""
1.0.0-alpha     → Alpha version
1.0.0-alpha.1   → Alpha version 1
1.0.0-beta      → Beta version
1.0.0-beta.2    → Beta version 2
1.0.0-rc.1      → Release candidate 1
1.0.0           → Stable release
"""

# Version bumping with bump2version
"""
# Install
pip install bump2version

# Configuration: .bumpversion.cfg
[bumpversion]
current_version = 1.0.0
commit = True
tag = True

[bumpversion:file:mypackage/__init__.py]
[bumpversion:file:pyproject.toml]

# Bump version
bump2version patch  # 1.0.0 → 1.0.1
bump2version minor  # 1.0.0 → 1.1.0
bump2version major  # 1.0.0 → 2.0.0
"""

# Version in code
# mypackage/__init__.py
"""
__version__ = '1.0.0'
"""

# Check version at runtime
"""
import mypackage
print(mypackage.__version__)  # '1.0.0'
"""

# CHANGELOG.md
"""
# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added
- New feature X

## [1.1.0] - 2024-01-15

### Added
- Feature A
- Feature B

### Changed
- Improved performance of Y

### Fixed
- Bug fix Z

## [1.0.1] - 2024-01-10

### Fixed
- Critical bug in authentication

## [1.0.0] - 2024-01-01

### Added
- Initial stable release
"""
```

### Frequently Asked Questions

**Q1: How do I update my package on PyPI?**

**A:**

```bash
# UPDATING PYPI PACKAGES

# Step 1: Update version
# Bump version in:
# - __init__.py: __version__ = '1.0.1'
# - pyproject.toml: version = "1.0.1"

# Step 2: Update CHANGELOG.md
"""
## [1.0.1] - 2024-01-20

### Fixed
- Bug in authentication
"""

# Step 3: Commit changes
git add .
git commit -m "Release v1.0.1"
git tag -a v1.0.1 -m "Release version 1.0.1"
git push origin main --tags

# Step 4: Build new distribution
python -m build

# Step 5: Upload to PyPI
twine upload dist/*

# With Poetry (automatic version management)
poetry version patch  # Bumps version
poetry build
poetry publish

# Automated with GitHub Actions
"""
1. Update version
2. Create GitHub release
3. GitHub Actions automatically publishes
"""

# What NOT to do:
"""
❌ Don't delete old versions from PyPI
❌ Don't reuse version numbers
❌ Don't publish untested code
❌ Don't forget to update CHANGELOG
"""

# Yanking releases (if you made a mistake)
"""
# Mark version as "yanked" on PyPI
# (Still installable if explicitly requested, but not by default)

$ pip install mypackage  # Won't install yanked version
$ pip install mypackage==1.0.1  # Can still install if needed
"""
```

**Why This Matters:** Proper versioning ensures users get correct updates.

---

## 9.4 Python Ecosystem

```python
"""
PYTHON ECOSYSTEM: Key libraries and frameworks

Categories:
- Web frameworks
- Data science
- Testing
- DevOps
- Utilities
"""

# Web Frameworks
web_frameworks = {
    "FastAPI": "Modern, fast async framework",
    "Django": "Batteries-included web framework",
    "Flask": "Lightweight, flexible framework",
    "Tornado": "Async networking library",
    "Pyramid": "Flexible web framework",
    "Bottle": "Micro framework",
    "Quart": "Async Flask-compatible",
    "Starlette": "ASGI framework (FastAPI uses it)",
}

# Data Science & ML
data_science = {
    "NumPy": "Numerical computing",
    "Pandas": "Data analysis and manipulation",
    "Matplotlib": "Plotting and visualization",
    "Seaborn": "Statistical visualization",
    "Plotly": "Interactive visualizations",
    "Scikit-learn": "Machine learning",
    "TensorFlow": "Deep learning framework",
    "PyTorch": "Deep learning framework",
    "Keras": "High-level neural networks API",
    "XGBoost": "Gradient boosting",
    "LightGBM": "Gradient boosting",
    "NLTK": "Natural language processing",
    "spaCy": "Industrial NLP",
}

# Async & Concurrency
async_tools = {
    "asyncio": "Built-in async framework",
    "aiohttp": "Async HTTP client/server",
    "httpx": "Async HTTP client",
    "trio": "Friendly async framework",
    "anyio": "Async compatibility layer",
}

# Database & ORM
databases = {
    "SQLAlchemy": "SQL toolkit and ORM",
    "Django ORM": "Django's built-in ORM",
    "Peewee": "Lightweight ORM",
    "Tortoise ORM": "Async ORM",
    "psycopg2": "PostgreSQL adapter",
    "pymongo": "MongoDB driver",
    "redis-py": "Redis client",
}

# Testing
testing = {
    "pytest": "Modern testing framework",
    "unittest": "Built-in testing framework",
    "hypothesis": "Property-based testing",
    "tox": "Test automation",
    "coverage.py": "Code coverage",
    "locust": "Load testing",
}

# CLI & UI
cli_ui = {
    "click": "CLI framework",
    "typer": "Modern CLI framework",
    "rich": "Beautiful terminal output",
    "textual": "Terminal user interfaces",
    "prompt_toolkit": "Interactive CLI",
}

# Web Scraping
scraping = {
    "requests": "HTTP library",
    "beautifulsoup4": "HTML parsing",
    "scrapy": "Web scraping framework",
    "selenium": "Browser automation",
    "playwright": "Modern browser automation",
}

# DevOps & Deployment
devops = {
    "docker": "Containerization",
    "kubernetes": "Container orchestration",
    "ansible": "Automation",
    "fabric": "Remote execution",
    "invoke": "Task execution",
}

# Code Quality
quality = {
    "black": "Code formatter",
    "flake8": "Linter",
    "pylint": "Linter",
    "mypy": "Static type checker",
    "bandit": "Security linter",
    "safety": "Dependency vulnerability scanner",
}

# Utilities
utilities = {
    "requests": "HTTP requests",
    "python-dotenv": "Environment variables",
    "pydantic": "Data validation",
    "marshmallow": "Serialization",
    "celery": "Distributed task queue",
    "APScheduler": "Task scheduling",
    "loguru": "Logging",
}
```

## 9.5 Best Practices

```python
"""
PYTHON PACKAGE BEST PRACTICES

Checklist for professional packages:
"""

best_practices = {
    "Documentation": [
        "✅ README.md with examples",
        "✅ API documentation (Sphinx)",
        "✅ CHANGELOG.md",
        "✅ Contributing guide",
        "✅ Code of conduct",
    ],
  
    "Code Quality": [
        "✅ Type hints",
        "✅ Docstrings (Google/NumPy style)",
        "✅ Code formatting (Black)",
        "✅ Linting (Flake8/Pylint)",
        "✅ 80%+ test coverage",
    ],
  
    "Testing": [
        "✅ Unit tests",
        "✅ Integration tests",
        "✅ CI/CD pipeline",
        "✅ Test multiple Python versions",
        "✅ Test on multiple OS",
    ],
  
    "Security": [
        "✅ No hardcoded secrets",
        "✅ Dependency vulnerability scanning",
        "✅ Security policy (SECURITY.md)",
        "✅ Regular security updates",
    ],
  
    "Versioning": [
        "✅ Semantic versioning",
        "✅ Git tags for releases",
        "✅ CHANGELOG maintained",
        "✅ Automated version bumping",
    ],
  
    "Distribution": [
        "✅ Published on PyPI",
        "✅ Source and wheel distributions",
        "✅ Clear installation instructions",
        "✅ Minimal dependencies",
    ],
  
    "Maintenance": [
        "✅ Active development",
        "✅ Responsive to issues",
        "✅ Regular releases",
        "✅ Deprecation warnings",
    ],
}

# Example: Professional package structure
"""
awesome-package/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── release.yml
│   │   └── publish.yml
│   ├── ISSUE_TEMPLATE/
│   └── PULL_REQUEST_TEMPLATE.md
├── docs/
│   ├── conf.py
│   ├── index.rst
│   └── api/
├── src/
│   └── awesome_package/
│       ├── __init__.py
│       ├── module1.py
│       └── module2.py
├── tests/
│   ├── test_module1.py
│   └── test_module2.py
├── .gitignore
├── .pre-commit-config.yaml
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
├── pyproject.toml
└── tox.ini
"""
```

### Key Takeaways

**Package Management:**

- **Structure**: Use src/ layout for large projects
- **Configuration**: pyproject.toml (modern) or setup.py (legacy)
- **Dependencies**: Poetry for management, requirements.txt for simple projects
- **Virtual environments**: Always use them, venv or Poetry
- **Publishing**: Build → Test on TestPyPI → Publish to PyPI
- **Versioning**: Semantic versioning (MAJOR.MINOR.PATCH)
- **Best practices**: Documentation, testing, CI/CD, security

---

## Part 9 Complete!

You've completed **Part 9: Package Management, Distribution & Ecosystem**!

✅ **Part 1**: Python Fundamentals (~29,000 words)
✅ **Part 2**: Python Internals (~20,000 words)
✅ **Part 3**: SDLC & Architecture (~16,000 words)
✅ **Part 4**: Testing & Quality (~9,200 words)
✅ **Part 5**: Deployment & DevOps (~8,000 words)
✅ **Part 6**: Version Control (~6,500 words)
✅ **Part 7**: Performance & Production (~5,850 words)
✅ **Part 8**: Advanced Topics (~5,300 words)
✅ **Part 9**: Package Management (~4,500 words)

**Total: ~104,350 words = ~417 pages!** 🎊

🎉 **OVER 100,000 WORDS!** Complete Python mastery achieved! 🚀
