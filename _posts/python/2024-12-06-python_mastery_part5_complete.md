---
title: "Python Mastery Part 5: Deployment & DevOps"
date: 2024-12-06 00:00:00 +0530
categories: [Python, Language Mastery]
tags: [Python, DevOps, Deployment, Docker, Kubernetes, CI/CD, Monitoring]
---
## Table of Contents

- [Introduction](#introduction)
- [5.1 Containerization with Docker](#51-containerization-with-docker)
  - [Docker Fundamentals](#docker-fundamentals)
  - [Dockerfile Best Practices](#dockerfile-best-practices)
  - [Docker Compose](#docker-compose)
  - [Multi-stage Builds](#multi-stage-builds)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [5.2 CI/CD Pipelines](#52-cicd-pipelines)
  - [GitHub Actions](#github-actions)
  - [GitLab CI/CD](#gitlab-cicd)
  - [Pipeline Best Practices](#pipeline-best-practices)
  - [Automated Testing in CI](#automated-testing-in-ci)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
  - [Interview Questions](#interview-questions-1)
- [5.3 Container Orchestration](#53-container-orchestration)
- [5.4 Monitoring &amp; Logging](#54-monitoring--logging)
- [5.5 Cloud Deployment](#55-cloud-deployment)

---

# Complete Python Mastery Part 5: Deployment & DevOps

## Introduction

Welcome to **Part 5** of the Complete Python Mastery series. Deployment and DevOps practices are essential for running Python applications in production.

**What You'll Learn in Part 5:**

This post covers modern deployment and DevOps practices:

- **Containerization**: Docker fundamentals, best practices, multi-stage builds
- **CI/CD**: GitHub Actions, GitLab CI, automated testing and deployment
- **Orchestration**: Kubernetes basics, deployments, services, scaling
- **Monitoring**: Logging, metrics, tracing, alerting
- **Cloud deployment**: AWS, GCP, Azure deployment strategies

**Why This Matters:**

Modern deployment practices enable:

- Consistent environments (dev = staging = production)
- Automated testing and deployment
- Scalability and reliability
- Fast rollbacks on failures
- Infrastructure as code

**Prerequisites:**

- Part 1: Python Fundamentals
- Part 2: Python Internals
- Part 3: SDLC & Architecture
- Part 4: Testing & Quality Assurance
- Basic Linux/command line knowledge

Let's deploy production-ready applications! 🚀

---

## 5.1 Containerization with Docker

**SDLC Phase:** Deployment, Operations

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [ ] Development
- [ ] Testing
- [X] Deployment
- [X] Maintenance (operations)

### Docker Fundamentals

```dockerfile
"""
DOCKER: Containerization platform

Benefits:
- Consistent environments
- Isolation
- Portability
- Easy scaling
- Version control for infrastructure
"""

# Basic Dockerfile for Python application

FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy requirements first (better caching)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```python
# Build and run Docker image

# Build image
# $ docker build -t myapp:1.0 .

# Run container
# $ docker run -p 8000:8000 myapp:1.0

# Run in detached mode
# $ docker run -d -p 8000:8000 --name myapp myapp:1.0

# View running containers
# $ docker ps

# View logs
# $ docker logs myapp

# Stop container
# $ docker stop myapp

# Remove container
# $ docker rm myapp

# Remove image
# $ docker rmi myapp:1.0
```

### Dockerfile Best Practices

```dockerfile
"""
DOCKERFILE BEST PRACTICES

1. Use specific base image versions
2. Minimize layers
3. Use .dockerignore
4. Don't run as root
5. Use multi-stage builds
6. Leverage build cache
"""

# ❌ BAD: Generic, insecure Dockerfile
FROM python:latest

COPY . .
RUN pip install -r requirements.txt

CMD ["python", "app.py"]

# ✅ GOOD: Optimized, secure Dockerfile
FROM python:3.11-slim as base

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```dockerfile
# .dockerignore file
"""
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/

# Testing
.pytest_cache/
.coverage
htmlcov/

# Development
.git/
.gitignore
.env
.vscode/
.idea/

# Documentation
*.md
docs/

# CI/CD
.github/
.gitlab-ci.yml

# Large files
*.log
*.db
*.sqlite
"""
```

### Docker Compose

```yaml
# docker-compose.yml
"""
DOCKER COMPOSE: Multi-container applications

Benefits:
- Define multiple services
- Networking between containers
- Volume management
- Environment configuration
"""

version: '3.8'

services:
  # Web application
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    volumes:
      - ./app:/app
    networks:
      - app-network
    restart: unless-stopped

  # PostgreSQL database
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: unless-stopped

  # Redis cache
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: unless-stopped

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - web
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

```python
# Commands for Docker Compose

# Start all services
# $ docker-compose up

# Start in detached mode
# $ docker-compose up -d

# View logs
# $ docker-compose logs -f web

# Stop all services
# $ docker-compose down

# Rebuild images
# $ docker-compose up --build

# Scale service
# $ docker-compose up --scale web=3

# Execute command in container
# $ docker-compose exec web python manage.py migrate

# View running services
# $ docker-compose ps
```

### Multi-stage Builds

```dockerfile
"""
MULTI-STAGE BUILDS: Optimize image size

Benefits:
- Smaller final images
- Separate build and runtime dependencies
- Better security (no build tools in production)
"""

# Stage 1: Builder
FROM python:3.11-slim as builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    g++ \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python packages to /install
RUN pip install --prefix=/install --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim

WORKDIR /app

# Install runtime dependencies only
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy installed packages from builder
COPY --from=builder /install /usr/local

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Copy application code
COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# Result: Image size reduced from 1.5GB to 300MB!
```

```dockerfile
# Multi-stage with testing

FROM python:3.11-slim as base

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage: Testing
FROM base as testing

COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

COPY . .

# Run tests
RUN pytest tests/ --cov=app --cov-report=xml

# Stage: Production
FROM base as production

COPY --chown=appuser:appuser . .

USER appuser

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# Build for testing:
# $ docker build --target testing -t myapp:test .

# Build for production:
# $ docker build --target production -t myapp:prod .
```

### Advanced Docker Patterns

```dockerfile
# Development Dockerfile with hot reload
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt requirements-dev.txt ./
RUN pip install --no-cache-dir -r requirements.txt -r requirements-dev.txt

# Copy code (will be overridden by volume mount)
COPY . .

# Expose port
EXPOSE 8000

# Use uvicorn with reload for development
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

```yaml
# docker-compose.dev.yml for development
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      # Mount source code for hot reload
      - ./app:/app
      # Don't override installed packages
      - /app/__pycache__
    environment:
      - DEBUG=1
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp_dev
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp_dev
    ports:
      - "5432:5432"  # Expose for local access
    volumes:
      - postgres-dev-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres-dev-data:

# Use: docker-compose -f docker-compose.dev.yml up
```

```python
# Python script to manage Docker environments

import subprocess
import sys

def run_command(cmd):
    """Run shell command"""
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    if result.returncode != 0:
        print(f"Error: {result.stderr}")
        sys.exit(1)
    return result.stdout

def build_image(tag="latest", target="production"):
    """Build Docker image"""
    print(f"Building image: {tag}")
    run_command(f"docker build --target {target} -t myapp:{tag} .")
    print("Build complete!")

def run_tests():
    """Run tests in Docker"""
    print("Running tests...")
    run_command("docker build --target testing -t myapp:test .")
    print("Tests passed!")

def deploy(environment="staging"):
    """Deploy to environment"""
    print(f"Deploying to {environment}...")
  
    if environment == "staging":
        run_command("docker-compose -f docker-compose.staging.yml up -d")
    elif environment == "production":
        run_command("docker-compose -f docker-compose.prod.yml up -d")
  
    print(f"Deployed to {environment}!")

if __name__ == "__main__":
    import argparse
  
    parser = argparse.ArgumentParser()
    parser.add_argument("command", choices=["build", "test", "deploy"])
    parser.add_argument("--tag", default="latest")
    parser.add_argument("--env", default="staging")
  
    args = parser.parse_args()
  
    if args.command == "build":
        build_image(args.tag)
    elif args.command == "test":
        run_tests()
    elif args.command == "deploy":
        deploy(args.env)
```

### Frequently Asked Questions

**Q1: Should I use alpine or slim base images?**

**A:**

```dockerfile
"""
ALPINE vs SLIM BASE IMAGES

Alpine:
✅ Smallest size (~5MB base)
✅ Security-focused
❌ Uses musl libc (compatibility issues)
❌ Slower pip installs (compile from source)
❌ Some packages don't work

Slim:
✅ Debian-based (better compatibility)
✅ Faster pip installs (pre-compiled wheels)
✅ Most packages work
❌ Slightly larger (~40MB base)

Recommendation: Use slim for Python applications
"""

# Alpine (smaller but potential issues)
FROM python:3.11-alpine
# Size: ~150MB
# Issues: May need to compile C extensions

# Slim (recommended)
FROM python:3.11-slim
# Size: ~200MB
# Works with most Python packages

# Full (not recommended for production)
FROM python:3.11
# Size: ~900MB
# Includes unnecessary build tools

# Example: Installing packages on alpine requires build tools
FROM python:3.11-alpine
RUN apk add --no-cache gcc musl-dev postgresql-dev
RUN pip install psycopg2  # Compiles from source

# On slim, psycopg2-binary has pre-compiled wheels
FROM python:3.11-slim
RUN pip install psycopg2-binary  # Fast install

# Decision tree:
"""
Need smallest possible image? → Alpine (be ready to debug)
Want compatibility and speed? → Slim (recommended)
Need build tools in container? → Full (not production)
"""
```

**Why This Matters:** Wrong base image leads to build failures or slow deployments.

---

**Q2: How do I handle secrets in Docker?**

**A:**

```python
"""
DOCKER SECRETS MANAGEMENT

Never commit secrets to Dockerfile or version control!

Options:
1. Environment variables (development only)
2. Docker secrets (Swarm mode)
3. External secret managers (HashiCorp Vault, AWS Secrets Manager)
4. Build-time secrets (Docker BuildKit)
"""

# ❌ BAD: Hardcoded secrets
# Dockerfile
ENV DATABASE_PASSWORD=supersecret123  # Never do this!

# ❌ BAD: Secrets in image
COPY .env .  # Secrets end up in image layers!

# ✅ GOOD: Environment variables at runtime
# docker-compose.yml
services:
  web:
    environment:
      - DATABASE_PASSWORD=${DATABASE_PASSWORD}
    # Or from env file
    env_file:
      - .env.production

# .env.production (not in git!)
DATABASE_PASSWORD=supersecret123
REDIS_PASSWORD=anothersecret456

# ✅ GOOD: Docker secrets (Swarm)
# docker-compose.yml
version: '3.8'
services:
  web:
    secrets:
      - db_password
      - redis_password

secrets:
  db_password:
    external: true
  redis_password:
    external: true

# Create secrets:
# $ echo "supersecret123" | docker secret create db_password -

# Access in Python:
def get_secret(secret_name):
    """Read Docker secret"""
    secret_path = f"/run/secrets/{secret_name}"
    try:
        with open(secret_path) as f:
            return f.read().strip()
    except FileNotFoundError:
        # Fallback to environment variable
        return os.getenv(secret_name.upper())

DATABASE_PASSWORD = get_secret("db_password")

# ✅ GOOD: Build-time secrets (BuildKit)
# Dockerfile
# syntax=docker/dockerfile:1

FROM python:3.11-slim

# Mount secret during build (not in image!)
RUN --mount=type=secret,id=pip_config \
    pip install --no-cache-dir -r requirements.txt

# Build with secret:
# $ docker build --secret id=pip_config,src=$HOME/.pip/pip.conf .

# ✅ BEST: External secret manager
import boto3

def get_secret_from_aws(secret_name):
    """Get secret from AWS Secrets Manager"""
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return response['SecretString']

# Best practices:
"""
1. Never commit secrets to git
2. Use different secrets per environment
3. Rotate secrets regularly
4. Use secret managers for production
5. Limit secret access (IAM policies)
6. Audit secret usage
"""
```

**Why This Matters:** Exposed secrets lead to security breaches and data loss.

---

### Interview Questions

**Question 1: Explain multi-stage Docker builds and their benefits.**

**Difficulty:** Mid-Level

**SDLC Relevance:** Deployment

**Answer:**

```dockerfile
"""
MULTI-STAGE DOCKER BUILDS

Definition: Multiple FROM statements in single Dockerfile,
each stage can copy artifacts from previous stages.

Benefits:
1. Smaller final images (no build tools)
2. Faster deployments (less data to transfer)
3. Better security (fewer attack vectors)
4. Cleaner separation (build vs runtime)
"""

# Example: Web application with compiled assets

# Stage 1: Build frontend
FROM node:18 as frontend-builder

WORKDIR /frontend

COPY package*.json ./
RUN npm ci

COPY frontend/ .
RUN npm run build  # Creates /frontend/dist

# Stage 2: Build Python dependencies
FROM python:3.11-slim as python-builder

WORKDIR /app

# Install build tools
RUN apt-get update && apt-get install -y gcc g++

COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

# Stage 3: Runtime
FROM python:3.11-slim

WORKDIR /app

# Install runtime dependencies only (no gcc!)
RUN apt-get update && apt-get install -y libpq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy Python wheels from builder
COPY --from=python-builder /wheels /wheels
RUN pip install --no-cache /wheels/*

# Copy built frontend from first stage
COPY --from=frontend-builder /frontend/dist /app/static

# Copy application code
COPY app/ /app/

CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]

"""
Size comparison:
- Single stage: 1.5GB (includes node, gcc, g++, npm)
- Multi-stage: 300MB (only runtime dependencies)

Security improvement:
- No build tools in final image
- Smaller attack surface
- Fewer vulnerabilities
"""

# Real-world example: Machine learning model

# Stage 1: Train model
FROM python:3.11 as trainer

WORKDIR /training

COPY training_requirements.txt .
RUN pip install -r training_requirements.txt

COPY data/ data/
COPY train.py .

RUN python train.py  # Generates model.pkl

# Stage 2: Serve model
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy trained model only
COPY --from=trainer /training/model.pkl /app/

COPY serve.py .

CMD ["python", "serve.py"]

"""
Benefits demonstrated:
✅ Training data not in final image
✅ Training dependencies not in final image
✅ Smaller, faster deployments
✅ Cleaner separation of concerns
"""
```

**Key Points:**

- Multiple FROM statements = multiple stages
- Each stage can copy from previous stages
- Final image only includes last stage
- Reduces size by 70-80% typically
- Improves security (no build tools)
- Faster deployments

---

## 5.2 CI/CD Pipelines

**SDLC Phase:** Continuous Integration, Deployment

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [ ] Development
- [X] Testing (automated)
- [X] Deployment (automated)
- [X] Maintenance (continuous)

### GitHub Actions

```yaml
# .github/workflows/ci.yml
"""
GITHUB ACTIONS: CI/CD workflows

Triggers:
- Push to branches
- Pull requests
- Scheduled (cron)
- Manual dispatch
"""

name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  # Job 1: Lint and format check
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
    
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
    
      - name: Install dependencies
        run: |
          pip install black flake8 isort mypy
    
      - name: Check formatting (Black)
        run: black --check .
    
      - name: Lint (Flake8)
        run: flake8 .
    
      - name: Check imports (isort)
        run: isort --check-only .
    
      - name: Type check (mypy)
        run: mypy app/

  # Job 2: Run tests
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']
  
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
  
    steps:
      - uses: actions/checkout@v3
    
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
    
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
    
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
    
      - name: Run tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
          REDIS_URL: redis://localhost:6379/0
        run: |
          pytest tests/ -v --cov=app --cov-report=xml
    
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
          fail_ci_if_error: true

  # Job 3: Security scan
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
    
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
    
      - name: Install dependencies
        run: |
          pip install bandit safety
    
      - name: Security scan (Bandit)
        run: bandit -r app/
    
      - name: Dependency check (Safety)
        run: safety check

  # Job 4: Build Docker image
  build:
    needs: [lint, test, security]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
    
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
    
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
    
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            myorg/myapp:latest
            myorg/myapp:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Job 5: Deploy to staging
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - name: Deploy to staging
        run: |
          # Deploy to staging server
          echo "Deploying to staging..."

  # Job 6: Deploy to production
  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Deploy to production
        run: |
          # Deploy to production server
          echo "Deploying to production..."
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
"""
GITLAB CI/CD: Integrated CI/CD platform

Features:
- Built into GitLab
- Auto DevOps
- Container registry
- Environments
"""

stages:
  - lint
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

# Cache pip packages
cache:
  paths:
    - .cache/pip
    - venv/

# Lint stage
lint:black:
  stage: lint
  image: python:3.11-slim
  before_script:
    - pip install black
  script:
    - black --check .

lint:flake8:
  stage: lint
  image: python:3.11-slim
  before_script:
    - pip install flake8
  script:
    - flake8 .

lint:mypy:
  stage: lint
  image: python:3.11-slim
  before_script:
    - pip install mypy
  script:
    - mypy app/

# Test stage
test:unit:
  stage: test
  image: python:3.11-slim
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: test
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/test
    REDIS_URL: redis://redis:6379/0
  before_script:
    - pip install -r requirements.txt -r requirements-dev.txt
  script:
    - pytest tests/ -v --cov=app --cov-report=xml --cov-report=term
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

test:integration:
  stage: test
  image: python:3.11-slim
  services:
    - postgres:15
    - redis:7
  variables:
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/test
  script:
    - pip install -r requirements.txt -r requirements-dev.txt
    - pytest tests/integration/ -v

# Build stage
build:docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker tag $DOCKER_IMAGE $CI_REGISTRY_IMAGE:latest
    - docker push $DOCKER_IMAGE
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

# Deploy stages
deploy:staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  script:
    - ssh -o StrictHostKeyChecking=no user@staging.example.com "
        docker pull $DOCKER_IMAGE &&
        docker stop myapp || true &&
        docker rm myapp || true &&
        docker run -d --name myapp -p 8000:8000 $DOCKER_IMAGE
      "
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy:production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  script:
    - ssh -o StrictHostKeyChecking=no user@production.example.com "
        docker pull $DOCKER_IMAGE &&
        docker stop myapp || true &&
        docker rm myapp || true &&
        docker run -d --name myapp -p 8000:8000 $DOCKER_IMAGE
      "
  environment:
    name: production
    url: https://example.com
  when: manual  # Require manual approval
  only:
    - main
```

### Pipeline Best Practices

```yaml
# Advanced GitHub Actions patterns

name: Advanced CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    # Run nightly
    - cron: '0 2 * * *'

env:
  PYTHON_VERSION: '3.11'
  NODE_VERSION: '18'

jobs:
  # Matrix testing
  test-matrix:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.9', '3.10', '3.11']
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: |
          pip install -r requirements.txt
          pytest tests/

  # Parallel job execution
  test-parallel:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        test-group: [unit, integration, e2e]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pytest tests/${{ matrix.test-group }}/

  # Conditional deployment
  deploy:
    needs: [test-matrix, test-parallel]
    runs-on: ubuntu-latest
    if: |
      github.event_name == 'push' &&
      github.ref == 'refs/heads/main' &&
      !contains(github.event.head_commit.message, '[skip ci]')
    steps:
      - name: Deploy
        run: echo "Deploying..."

  # Artifact management
  build-and-archive:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
    
      - name: Build application
        run: |
          python setup.py sdist bdist_wheel
    
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-packages
          path: dist/
          retention-days: 30
    
      - name: Download artifacts (in another job)
        uses: actions/download-artifact@v3
        with:
          name: dist-packages

  # Notification on failure
  notify:
    needs: [test-matrix, test-parallel, deploy]
    runs-on: ubuntu-latest
    if: failure()
    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'CI pipeline failed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Automated Testing in CI

```python
# conftest.py for CI/CD testing
"""
CI/CD TESTING CONFIGURATION

Best practices:
1. Use fixtures for test data
2. Parallelize tests
3. Generate coverage reports
4. Fail fast on critical errors
"""

import pytest
import os

# Detect CI environment
def is_ci():
    return os.getenv('CI') == 'true'

@pytest.fixture(scope="session")
def db_url():
    """Database URL for CI vs local"""
    if is_ci():
        # CI environment (GitHub Actions, GitLab CI)
        return os.getenv('DATABASE_URL')
    else:
        # Local development
        return 'postgresql://localhost/test_db'

@pytest.fixture
def api_client(db_url):
    """Test client with CI-aware configuration"""
    from fastapi.testclient import TestClient
    from app.main import app
  
    # Configure for CI
    if is_ci():
        app.state.testing = True
        app.state.slow_tests = False  # Skip slow tests in CI
  
    client = TestClient(app)
    return client

# Markers for CI
def pytest_configure(config):
    config.addinivalue_line(
        "markers", "slow: marks tests as slow (deselect with '-m \"not slow\"')"
    )
    config.addinivalue_line(
        "markers", "integration: marks tests as integration tests"
    )

# Skip slow tests in CI (unless explicitly enabled)
def pytest_collection_modifyitems(config, items):
    if is_ci() and not config.getoption("--run-slow"):
        skip_slow = pytest.mark.skip(reason="Skipping slow tests in CI")
        for item in items:
            if "slow" in item.keywords:
                item.add_marker(skip_slow)
```

```yaml
# pytest.ini configuration for CI
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

# Markers
markers =
    slow: slow tests (skipped in CI by default)
    integration: integration tests
    e2e: end-to-end tests
    unit: unit tests

# Coverage
addopts =
    -v
    --strict-markers
    --cov=app
    --cov-report=term-missing
    --cov-report=xml
    --cov-report=html
    --cov-fail-under=80

# Parallel execution (install pytest-xdist)
# addopts = -n auto

# CI-specific settings
[pytest:ci]
addopts =
    -v
    --strict-markers
    --cov=app
    --cov-report=xml
    --tb=short
    --maxfail=5
    -m "not slow"
```

### Frequently Asked Questions

**Q1: How do I speed up CI/CD pipelines?**

**A:**

```yaml
"""
CI/CD PIPELINE OPTIMIZATION

Strategies:
1. Cache dependencies
2. Parallelize jobs
3. Fail fast
4. Use matrix builds wisely
5. Optimize Docker builds
"""

# 1. Cache dependencies
jobs:
  test:
    steps:
      - uses: actions/cache@v3
        with:
          path: |
            ~/.cache/pip
            ~/.npm
            node_modules
          key: ${{ runner.os }}-deps-${{ hashFiles('**/requirements.txt', '**/package-lock.json') }}
    
      # Dependencies install much faster with cache
      - run: pip install -r requirements.txt

# 2. Parallelize jobs
jobs:
  test-unit:
    # Runs in parallel with other jobs
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/unit/
  
  test-integration:
    # Runs simultaneously
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/integration/
  
  test-e2e:
    # Also runs in parallel
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/e2e/

# 3. Fail fast
strategy:
  fail-fast: true  # Stop all jobs if one fails
  matrix:
    python-version: ['3.9', '3.10', '3.11']

# 4. Skip unnecessary jobs
jobs:
  build:
    # Only run on main branch
    if: github.ref == 'refs/heads/main'
    steps:
      - run: docker build .

# 5. Optimize Docker builds
- name: Build with cache
  uses: docker/build-push-action@v4
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max

# Results:
"""
Before optimization: 15 minutes
After optimization:  3 minutes (5x faster!)

Breakdown:
- Dependency caching:    -5 min
- Parallel jobs:         -4 min
- Fail fast:             -2 min
- Docker build cache:    -1 min
"""
```

**Why This Matters:** Slow CI/CD wastes developer time and slows releases.

---

(Continuing with sections 5.3, 5.4, and 5.5 in next file...)

## 5.3 Container Orchestration with Kubernetes

**SDLC Phase:** Deployment, Operations, Scaling

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [ ] Development
- [ ] Testing
- [X] Deployment (orchestration)
- [X] Maintenance (scaling, updates)

### Kubernetes Fundamentals

```yaml
"""
KUBERNETES (K8s): Container orchestration platform

Core Concepts:
- Pods: Smallest deployable units
- Deployments: Manage replica sets
- Services: Expose pods to network
- ConfigMaps: Configuration data
- Secrets: Sensitive data
- Ingress: External access

Benefits:
✅ Automatic scaling
✅ Self-healing
✅ Rolling updates
✅ Load balancing
✅ Service discovery
"""

# Basic Pod definition
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: myorg/myapp:1.0
    ports:
    - containerPort: 8000
    env:
    - name: DATABASE_URL
      value: postgresql://postgres:5432/myapp
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"

# Apply: kubectl apply -f pod.yaml
# View: kubectl get pods
# Logs: kubectl logs myapp-pod
# Delete: kubectl delete pod myapp-pod
```

### Deployments

```yaml
"""
DEPLOYMENTS: Manage application lifecycle

Features:
- Replica management
- Rolling updates
- Rollback
- Self-healing
"""

# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  # Number of replicas
  replicas: 3
  
  # Selector for pods
  selector:
    matchLabels:
      app: myapp
  
  # Pod template
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myorg/myapp:1.0
        ports:
        - containerPort: 8000
      
        # Environment variables from ConfigMap
        envFrom:
        - configMapRef:
            name: myapp-config
      
        # Environment variables from Secret
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-password
      
        # Liveness probe (restart if unhealthy)
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
      
        # Readiness probe (remove from service if not ready)
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
      
        # Resource requests and limits
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
  
  # Rolling update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired count
      maxUnavailable: 0  # Max pods unavailable during update

# Apply: kubectl apply -f deployment.yaml
# Scale: kubectl scale deployment myapp-deployment --replicas=5
# Update image: kubectl set image deployment/myapp-deployment myapp=myorg/myapp:2.0
# Rollback: kubectl rollout undo deployment/myapp-deployment
# Status: kubectl rollout status deployment/myapp-deployment
```

### Services

```yaml
"""
SERVICES: Network access to pods

Types:
- ClusterIP: Internal only (default)
- NodePort: External access via node port
- LoadBalancer: External load balancer
"""

# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  # Service type
  type: LoadBalancer
  
  # Selector matches pod labels
  selector:
    app: myapp
  
  # Port mapping
  ports:
  - name: http
    protocol: TCP
    port: 80        # Service port
    targetPort: 8000  # Container port
  
  # Session affinity (sticky sessions)
  sessionAffinity: ClientIP

# Apply: kubectl apply -f service.yaml
# View: kubectl get services
# Describe: kubectl describe service myapp-service

---
# Internal service (ClusterIP) for database
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: ClusterIP  # Internal only
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### ConfigMaps and Secrets

```yaml
"""
CONFIGMAPS: Non-sensitive configuration
SECRETS: Sensitive data (base64 encoded)
"""

# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  # Key-value pairs
  LOG_LEVEL: "info"
  API_URL: "https://api.example.com"
  REDIS_URL: "redis://redis-service:6379/0"
  
  # File content
  app-config.yaml: |
    server:
      host: 0.0.0.0
      port: 8000
    database:
      pool_size: 10

# Apply: kubectl apply -f configmap.yaml

---
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
data:
  # Base64 encoded values
  db-password: cGFzc3dvcmQxMjM=  # "password123"
  api-key: c2VjcmV0a2V5NDU2      # "secretkey456"

# Create from literal: kubectl create secret generic myapp-secrets \
#   --from-literal=db-password=password123 \
#   --from-literal=api-key=secretkey456

# Create from file: kubectl create secret generic myapp-secrets \
#   --from-file=./secrets/db-password.txt

---
# Use in deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myorg/myapp:1.0
      
        # Environment from ConfigMap
        envFrom:
        - configMapRef:
            name: myapp-config
      
        # Individual secret as environment variable
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-password
      
        # Mount ConfigMap as volume
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
      
      volumes:
      - name: config-volume
        configMap:
          name: myapp-config
```

### Ingress

```yaml
"""
INGRESS: HTTP/HTTPS routing to services

Features:
- Path-based routing
- Host-based routing
- TLS/SSL termination
- Load balancing
"""

# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    # NGINX ingress controller
    kubernetes.io/ingress.class: "nginx"
  
    # SSL redirect
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
  
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
spec:
  # TLS configuration
  tls:
  - hosts:
    - example.com
    - www.example.com
    secretName: tls-secret
  
  # Routing rules
  rules:
  # Host-based routing
  - host: example.com
    http:
      paths:
      # Path-based routing
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
    
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
  
  # Another host
  - host: staging.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: staging-service
            port:
              number: 80

# Apply: kubectl apply -f ingress.yaml
# View: kubectl get ingress
# Describe: kubectl describe ingress myapp-ingress
```

### Horizontal Pod Autoscaling

```yaml
"""
HORIZONTAL POD AUTOSCALER (HPA)
Automatically scale pods based on metrics
"""

# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  
  # Min and max replicas
  minReplicas: 2
  maxReplicas: 10
  
  # Scaling metrics
  metrics:
  # CPU utilization
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale up at 70% CPU
  
  # Memory utilization
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # Scale up at 80% memory
  
  # Custom metric (requests per second)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  
  # Scaling behavior
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
      policies:
      - type: Percent
        value: 50  # Scale down by 50% at most
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0  # Scale up immediately
      policies:
      - type: Percent
        value: 100  # Can double pods
        periodSeconds: 15

# Apply: kubectl apply -f hpa.yaml
# View: kubectl get hpa
# Watch: kubectl get hpa myapp-hpa --watch
```

### StatefulSets for Databases

```yaml
"""
STATEFULSETS: For stateful applications

Features:
- Stable network identities
- Persistent storage
- Ordered deployment and scaling
- Ordered rolling updates
"""

# statefulset.yaml (PostgreSQL example)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-service
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  
  # Persistent volume claim template
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi

# Headless service for StatefulSet
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  clusterIP: None  # Headless service
  selector:
    app: postgres
  ports:
  - port: 5432

# Access pods: postgres-0.postgres-service, postgres-1.postgres-service, etc.
```

### Complete Application Stack

```yaml
# complete-app.yaml
"""
Complete Kubernetes deployment for Python application
"""

# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-production

---
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: myapp-production
data:
  LOG_LEVEL: "info"
  REDIS_URL: "redis://redis-service:6379/0"
  DATABASE_URL: "postgresql://postgres-service:5432/myapp"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: myapp-production
type: Opaque
data:
  db-password: cGFzc3dvcmQxMjM=
  secret-key: c2VjcmV0a2V5NDU2

---
# PostgreSQL StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: myapp-production
spec:
  serviceName: postgres-service
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: myapp
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-password
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi

---
# PostgreSQL Service
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: myapp-production
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - port: 5432

---
# Redis Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: myapp-production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379

---
# Redis Service
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: myapp-production
spec:
  selector:
    app: redis
  ports:
  - port: 6379

---
# Application Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp-production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myorg/myapp:1.0
        ports:
        - containerPort: 8000
        envFrom:
        - configMapRef:
            name: myapp-config
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db-password
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

---
# Application Service
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: myapp-production
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8000

---
# HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: myapp-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

# Deploy entire stack:
# kubectl apply -f complete-app.yaml
```

### Frequently Asked Questions

**Q1: When should I use Kubernetes vs just Docker?**

**A:**

```yaml
"""
KUBERNETES vs DOCKER DECISION

Use Docker (docker-compose) when:
✅ Simple application (1-5 services)
✅ Single server deployment
✅ Small team (<5 developers)
✅ No high availability requirements
✅ Development/testing environment

Use Kubernetes when:
✅ Complex application (10+ services)
✅ Multi-server deployment
✅ Need high availability (99.9%+)
✅ Need auto-scaling
✅ Large team (10+ developers)
✅ Production at scale

Example Decision Tree:

Scenario 1: Simple blog
→ Docker Compose ✅
   - Web server + PostgreSQL
   - Single server is enough
   - Easy to manage

Scenario 2: E-commerce platform
→ Kubernetes ✅
   - Multiple services (auth, cart, payment, inventory)
   - Need 99.9% uptime
   - Must scale on Black Friday
   - Load balancing required

Scenario 3: Startup MVP
→ Docker Compose ✅
   - Fast iteration needed
   - Simpler than K8s
   - Can migrate to K8s later

Scenario 4: Enterprise SaaS
→ Kubernetes ✅
   - Multi-tenant
   - Global deployment
   - Need auto-scaling
   - Compliance requirements
"""

# Migration path:
"""
Phase 1: Docker Compose (Months 0-6)
- Get product-market fit
- Simple deployment

Phase 2: Managed Kubernetes (Months 6-12)
- Use AWS EKS, GKE, or AKS
- Start with basic deployment
- Learn K8s gradually

Phase 3: Advanced K8s (Months 12+)
- Implement auto-scaling
- Add monitoring
- Optimize costs
"""
```

**Why This Matters:** Kubernetes adds complexity. Use when benefits outweigh costs.

---

## 5.4 Monitoring & Logging

**SDLC Phase:** Operations, Maintenance

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [ ] Development
- [ ] Testing
- [X] Deployment
- [X] Maintenance (observability)

### Structured Logging

```python
"""
STRUCTURED LOGGING: Machine-parseable logs

Benefits:
✅ Easy to search and filter
✅ Better analysis
✅ Integration with log aggregators
"""

# Install: pip install structlog

import structlog
from datetime import datetime

# Configure structlog
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.processors.JSONRenderer()  # Output as JSON
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

# Create logger
logger = structlog.get_logger()

# ❌ BAD: Unstructured logging
import logging
logging.info(f"User {user_id} placed order {order_id} for ${total}")
# Output: User 123 placed order 456 for $99.99
# Hard to parse!

# ✅ GOOD: Structured logging
logger.info(
    "order_placed",
    user_id="123",
    order_id="456",
    total=99.99,
    currency="USD"
)
# Output: {"event": "order_placed", "user_id": "123", "order_id": "456", 
#          "total": 99.99, "currency": "USD", "timestamp": "2025-01-02T10:30:00"}
# Easy to search: total > 50, user_id = "123"

# FastAPI integration
from fastapi import FastAPI, Request
import time
import uuid

app = FastAPI()

@app.middleware("http")
async def log_requests(request: Request, call_next):
    """Log all requests with structured data"""
    request_id = str(uuid.uuid4())
  
    # Add request_id to context
    structlog.contextvars.clear_contextvars()
    structlog.contextvars.bind_contextvars(request_id=request_id)
  
    logger.info(
        "request_started",
        method=request.method,
        path=request.url.path,
        client=request.client.host
    )
  
    start_time = time.time()
  
    try:
        response = await call_next(request)
        duration = time.time() - start_time
      
        logger.info(
            "request_completed",
            method=request.method,
            path=request.url.path,
            status_code=response.status_code,
            duration_seconds=duration
        )
      
        return response
  
    except Exception as e:
        duration = time.time() - start_time
      
        logger.error(
            "request_failed",
            method=request.method,
            path=request.url.path,
            error=str(e),
            duration_seconds=duration,
            exc_info=True
        )
        raise

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    logger.info("fetching_user", user_id=user_id)
  
    # ... fetch user ...
  
    logger.info("user_fetched", user_id=user_id, user_name="Alice")
    return {"id": user_id, "name": "Alice"}

# Log output (JSON):
"""
{"event": "request_started", "method": "GET", "path": "/users/123", 
 "client": "192.168.1.1", "request_id": "abc-123", "timestamp": "2025-01-02T10:30:00"}
{"event": "fetching_user", "user_id": "123", "request_id": "abc-123", 
 "timestamp": "2025-01-02T10:30:00"}
{"event": "user_fetched", "user_id": "123", "user_name": "Alice", 
 "request_id": "abc-123", "timestamp": "2025-01-02T10:30:01"}
{"event": "request_completed", "method": "GET", "path": "/users/123", 
 "status_code": 200, "duration_seconds": 0.05, "request_id": "abc-123", 
 "timestamp": "2025-01-02T10:30:01"}
"""

# Benefits:
# - Can filter by request_id to see all logs for one request
# - Can aggregate by status_code
# - Can calculate average duration
# - Can alert on errors
```

### Application Performance Monitoring (APM)

```python
"""
APM: Monitor application performance

Tools:
- Datadog APM
- New Relic
- Elastic APM
- Prometheus + Grafana
"""

# Install: pip install ddtrace (Datadog)

from ddtrace import tracer
from ddtrace.contrib.fastapi import patch

# Patch FastAPI for automatic tracing
patch()

app = FastAPI()

# Custom instrumentation
@tracer.wrap(service="myapp", resource="get_user")
def get_user_from_db(user_id: str):
    """Traced database query"""
    with tracer.trace("database.query", resource="SELECT users"):
        # Query database
        result = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    return result

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    # Automatically traced by patch()
  
    # Add custom tags
    tracer.current_span().set_tag("user_id", user_id)
  
    user = get_user_from_db(user_id)
  
    # Add custom metric
    tracer.current_span().set_metric("user_age", user.age)
  
    return user

# Prometheus metrics
from prometheus_client import Counter, Histogram, Gauge
from prometheus_client import make_asgi_app
from fastapi import FastAPI

app = FastAPI()

# Define metrics
request_count = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

active_users = Gauge(
    'active_users',
    'Number of active users'
)

@app.middleware("http")
async def track_metrics(request: Request, call_next):
    """Track request metrics"""
    with request_duration.labels(
        method=request.method,
        endpoint=request.url.path
    ).time():
        response = await call_next(request)
  
    request_count.labels(
        method=request.method,
        endpoint=request.url.path,
        status=response.status_code
    ).inc()
  
    return response

# Expose metrics endpoint
metrics_app = make_asgi_app()
app.mount("/metrics", metrics_app)

# Prometheus scrapes /metrics endpoint
# Grafana visualizes the metrics
```

### Distributed Tracing

```python
"""
DISTRIBUTED TRACING: Track requests across services

Tools:
- Jaeger
- Zipkin
- OpenTelemetry
"""

# Install: pip install opentelemetry-api opentelemetry-sdk \
#          opentelemetry-instrumentation-fastapi \
#          opentelemetry-exporter-jaeger

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

# Setup tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Jaeger exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)

trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

# Instrument FastAPI
app = FastAPI()
FastAPIInstrumentor.instrument_app(app)

# Manual instrumentation
@app.get("/users/{user_id}")
async def get_user(user_id: str):
    with tracer.start_as_current_span("get_user") as span:
        span.set_attribute("user_id", user_id)
      
        # Call to another service (automatically traced)
        async with httpx.AsyncClient() as client:
            response = await client.get(f"http://auth-service/verify/{user_id}")
      
        span.add_event("user_verified")
      
        # Call to database (trace this too)
        with tracer.start_as_current_span("database_query"):
            user = await get_user_from_db(user_id)
      
        return user

# Trace shows:
"""
Service: web-app
  └─ Span: GET /users/123 (100ms)
      ├─ Span: get_user (95ms)
      │   ├─ HTTP GET http://auth-service/verify/123 (30ms)
      │   │   └─ Service: auth-service
      │   │       └─ Span: verify_user (25ms)
      │   └─ database_query (60ms)
      │       └─ SELECT * FROM users WHERE id = 123
      └─ Span: response_serialization (5ms)
"""
```

### Health Checks

```python
"""
HEALTH CHECKS: Monitor service health

Types:
- Liveness: Is service running?
- Readiness: Can service handle traffic?
- Startup: Is service ready to start?
"""

from fastapi import FastAPI, status
from fastapi.responses import JSONResponse

app = FastAPI()

# Liveness probe (simple)
@app.get("/health")
async def health():
    """Kubernetes liveness probe"""
    return {"status": "healthy"}

# Readiness probe (check dependencies)
@app.get("/ready")
async def ready():
    """Kubernetes readiness probe"""
    checks = {}
  
    # Check database
    try:
        await db.execute("SELECT 1")
        checks["database"] = "healthy"
    except Exception as e:
        checks["database"] = "unhealthy"
        return JSONResponse(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            content={"status": "not ready", "checks": checks}
        )
  
    # Check Redis
    try:
        await redis.ping()
        checks["redis"] = "healthy"
    except Exception as e:
        checks["redis"] = "unhealthy"
        return JSONResponse(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            content={"status": "not ready", "checks": checks}
        )
  
    # Check external API
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get("https://api.example.com/health")
            if response.status_code == 200:
                checks["external_api"] = "healthy"
            else:
                checks["external_api"] = "degraded"
    except Exception as e:
        checks["external_api"] = "unhealthy"
        return JSONResponse(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            content={"status": "not ready", "checks": checks}
        )
  
    return {"status": "ready", "checks": checks}

# Startup probe (slow initialization)
@app.get("/startup")
async def startup():
    """Kubernetes startup probe"""
    # Check if initialization complete
    if not app.state.initialized:
        return JSONResponse(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            content={"status": "starting"}
        )
  
    return {"status": "started"}

# Kubernetes probes configuration:
"""
livenessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8000
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3

startupProbe:
  httpGet:
    path: /startup
    port: 8000
  failureThreshold: 30
  periodSeconds: 10
"""
```

### Alerting

```python
"""
ALERTING: Notify on issues

Tools:
- PagerDuty
- Opsgenie
- Slack
- Email
"""

# Alerting with Prometheus + Alertmanager
# prometheus_rules.yml
"""
groups:
- name: application_alerts
  rules:
  # High error rate
  - alert: HighErrorRate
    expr: |
      rate(http_requests_total{status=~"5.."}[5m]) > 0.05
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is {{ $value }} (threshold: 0.05)"
  
  # High latency
  - alert: HighLatency
    expr: |
      histogram_quantile(0.95, 
        rate(http_request_duration_seconds_bucket[5m])
      ) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High latency detected"
      description: "P95 latency is {{ $value }}s (threshold: 1s)"
  
  # Low memory
  - alert: LowMemory
    expr: |
      (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Low memory on node"
      description: "Only {{ $value }}% memory available"
"""

# Python alerting integration
import requests

def send_pagerduty_alert(message: str, severity: str = "error"):
    """Send alert to PagerDuty"""
    payload = {
        "routing_key": os.getenv("PAGERDUTY_KEY"),
        "event_action": "trigger",
        "payload": {
            "summary": message,
            "severity": severity,
            "source": "myapp",
            "timestamp": datetime.utcnow().isoformat()
        }
    }
  
    response = requests.post(
        "https://events.pagerduty.com/v2/enqueue",
        json=payload
    )
    return response.json()

def send_slack_alert(message: str, channel: str = "#alerts"):
    """Send alert to Slack"""
    payload = {
        "channel": channel,
        "text": message,
        "username": "Alert Bot",
        "icon_emoji": ":rotating_light:"
    }
  
    response = requests.post(
        os.getenv("SLACK_WEBHOOK_URL"),
        json=payload
    )
    return response.status_code

# Use in error handler
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    """Handle uncaught exceptions"""
  
    logger.error(
        "unhandled_exception",
        path=request.url.path,
        error=str(exc),
        exc_info=True
    )
  
    # Send alert for critical errors
    send_slack_alert(
        f"🚨 Unhandled exception in {request.url.path}: {exc}"
    )
  
    # For database errors, page on-call
    if isinstance(exc, DatabaseError):
        send_pagerduty_alert(
            f"Database error: {exc}",
            severity="critical"
        )
  
    return JSONResponse(
        status_code=500,
        content={"error": "Internal server error"}
    )
```

### Frequently Asked Questions

**Q1: How much logging is too much?**

**A:**

```python
"""
LOGGING BEST PRACTICES

Log levels:
- DEBUG: Detailed info for debugging (development only)
- INFO: General informational messages
- WARNING: Warning messages (potential issues)
- ERROR: Error messages (failures)
- CRITICAL: Critical errors (service down)

What to log:
✅ Request/response (with sampling)
✅ Business events (order placed, user registered)
✅ Errors and exceptions
✅ Performance metrics
✅ Security events (failed login, unauthorized access)

What NOT to log:
❌ Passwords or secrets
❌ Personal data (GDPR concern)
❌ Every database query (too much data)
❌ DEBUG logs in production
"""

# ❌ BAD: Too much logging
logger.debug(f"Entering function get_user")
logger.debug(f"user_id: {user_id}")
logger.debug(f"Querying database")
result = db.query(...)
logger.debug(f"Query result: {result}")
logger.debug(f"Processing result")
logger.debug(f"Exiting function")
# Generates thousands of logs per second!

# ✅ GOOD: Log important events
logger.info("user_fetched", user_id=user_id, duration_ms=50)

# ✅ GOOD: Log sampling (1% of requests)
import random

@app.middleware("http")
async def log_requests(request: Request, call_next):
    # Only log 1% of successful requests
    response = await call_next(request)
  
    if response.status_code >= 400 or random.random() < 0.01:
        logger.info(
            "request",
            method=request.method,
            path=request.url.path,
            status=response.status_code
        )
  
    return response

# Cost estimation:
"""
Without sampling:
- 1000 requests/sec
- 10 log lines per request
- 10,000 log lines/sec
- 864 million logs/day
- Storage: ~500 GB/month
- Cost: $500-1000/month

With 1% sampling + error logging:
- 10 error logs/sec (all errors)
- 100 sampled logs/sec (1% of success)
- 110 log lines/sec total
- Storage: ~5 GB/month
- Cost: $10-20/month
"""
```

**Why This Matters:** Excessive logging increases costs and makes finding issues harder.

---

## 5.5 Cloud Deployment

**SDLC Phase:** Deployment, Operations

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [ ] Development
- [ ] Testing
- [X] Deployment (cloud)
- [X] Maintenance (cloud operations)

### AWS Deployment

```python
"""
AWS: Amazon Web Services

Popular services:
- EC2: Virtual machines
- ECS: Container orchestration
- EKS: Kubernetes service
- Lambda: Serverless
- Elastic Beanstalk: PaaS
- RDS: Managed databases
"""

# AWS ECS (Elastic Container Service)
# task-definition.json
"""
{
  "family": "myapp",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "LOG_LEVEL",
          "value": "info"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/myapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
"""

# Deploy with AWS CLI
"""
# Create ECR repository
aws ecr create-repository --repository-name myapp

# Login to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Build and push image
docker build -t myapp .
docker tag myapp:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create service
aws ecs create-service \
  --cluster myapp-cluster \
  --service-name myapp-service \
  --task-definition myapp \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-123],securityGroups=[sg-123]}"
"""

# AWS Lambda (Serverless)
# handler.py
"""
AWS Lambda for serverless Python
"""

import json

def lambda_handler(event, context):
    """Lambda function handler"""
  
    # Parse request
    body = json.loads(event['body'])
    user_id = body.get('user_id')
  
    # Process request
    result = process_user(user_id)
  
    # Return response
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(result)
    }

# Deploy with Serverless Framework
# serverless.yml
"""
service: myapp

provider:
  name: aws
  runtime: python3.11
  region: us-east-1
  
  environment:
    LOG_LEVEL: info
  
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:GetItem
        - dynamodb:PutItem
      Resource: "arn:aws:dynamodb:us-east-1:*:table/users"

functions:
  api:
    handler: handler.lambda_handler
    events:
      - http:
          path: /users/{user_id}
          method: get
      - http:
          path: /users
          method: post

# Deploy: serverless deploy
"""
```

### GCP Deployment

```yaml
# Google Cloud Platform
"""
Popular services:
- Compute Engine: VMs
- Cloud Run: Serverless containers
- GKE: Kubernetes service
- App Engine: PaaS
- Cloud Functions: Serverless
"""

# Cloud Run (easiest for containers)
# Deploy with one command:
# gcloud run deploy myapp \
#   --image gcr.io/project/myapp:latest \
#   --platform managed \
#   --region us-central1 \
#   --allow-unauthenticated

# app.yaml for App Engine
runtime: python311

instance_class: F2

env_variables:
  LOG_LEVEL: "info"
  DATABASE_URL: "postgresql://..."

handlers:
- url: /.*
  script: auto

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.7

# Deploy: gcloud app deploy
```

### Infrastructure as Code

```python
# Terraform example (works with AWS, GCP, Azure)
"""
terraform/main.tf
"""

# AWS provider
provider "aws" {
  region = "us-east-1"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "myapp-vpc"
  }
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "myapp-cluster"
}

# Task definition
resource "aws_ecs_task_definition" "app" {
  family                   = "myapp"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 256
  memory                   = 512
  
  container_definitions = jsonencode([{
    name  = "myapp"
    image = "myorg/myapp:latest"
    portMappings = [{
      containerPort = 8000
    }]
  }])
}

# ECS Service
resource "aws_ecs_service" "main" {
  name            = "myapp-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 3
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets         = aws_subnet.private.*.id
    security_groups = [aws_security_group.app.id]
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "myapp"
    container_port   = 8000
  }
}

# Apply: terraform apply
```

### Frequently Asked Questions

**Q1: Which cloud provider should I choose?**

**A:**

```python
"""
CLOUD PROVIDER COMPARISON

AWS (Amazon Web Services):
✅ Most mature (2006)
✅ Largest service catalog
✅ Best for: Enterprise, complex architectures
❌ Steeper learning curve
❌ Complex pricing

GCP (Google Cloud Platform):
✅ Best for: Data/ML, Kubernetes
✅ Simpler than AWS
✅ Good pricing
❌ Smaller service catalog

Azure (Microsoft):
✅ Best for: Microsoft shops (.NET, Windows)
✅ Good enterprise integration
✅ Active Directory integration
❌ Less popular for startups

Decision factors:
1. Team expertise (use what you know)
2. Existing infrastructure
3. Specific service needs
4. Pricing
5. Geographic coverage

Recommendation for Python apps:
- Starting out? → GCP Cloud Run (simplest)
- Need Kubernetes? → Any (similar)
- Enterprise? → AWS (most mature)
- .NET stack? → Azure
"""
```

**Why This Matters:** Cloud choice affects costs, complexity, and team productivity.

---

### Key Takeaways

**Deployment & DevOps:**

- **Docker**: Containerize for consistency and portability
- **Multi-stage builds**: Reduce image size by 70-80%
- **docker-compose**: Multi-container local development
- **CI/CD**: Automate testing and deployment
- **GitHub Actions**: Matrix testing, caching, parallel jobs
- **Kubernetes**: Orchestration for production at scale
- **Deployments**: Manage replicas, rolling updates, rollbacks
- **Services**: Load balancing and service discovery
- **HPA**: Auto-scale based on CPU, memory, custom metrics
- **ConfigMaps/Secrets**: Configuration and sensitive data
- **Monitoring**: Structured logging, APM, distributed tracing
- **Health checks**: Liveness, readiness, startup probes
- **Cloud**: AWS ECS/EKS, GCP Cloud Run/GKE, Azure AKS

**Best Practices:**

- Use specific image versions (never :latest in production)
- Run containers as non-root users
- Implement health checks
- Use secrets managers (not environment variables)
- Structure logs as JSON
- Monitor application performance
- Set up alerts for critical issues
- Use Infrastructure as Code
- Test deployments in staging first
- Have rollback plans

**Tools Summary:**

- **Containers**: Docker, docker-compose
- **CI/CD**: GitHub Actions, GitLab CI, Jenkins
- **Orchestration**: Kubernetes, AWS ECS, GCP Cloud Run
- **Monitoring**: Prometheus, Grafana, Datadog, New Relic
- **Logging**: structlog, ELK stack, Loki
- **Tracing**: Jaeger, Zipkin, OpenTelemetry
- **IaC**: Terraform, CloudFormation, Pulumi

---

## Part 5 Complete!

Congratulations! You've completed **Part 5: Deployment & DevOps**. You now have comprehensive knowledge of:

✅ **Section 5.1**: Containerization with Docker
✅ **Section 5.2**: CI/CD Pipelines
✅ **Section 5.3**: Container Orchestration
✅ **Section 5.4**: Monitoring & Logging
✅ **Section 5.5**: Cloud Deployment

**What You've Learned:**

- Docker fundamentals and best practices
- Multi-stage builds for optimization
- docker-compose for local development
- CI/CD pipelines with GitHub Actions and GitLab CI
- Kubernetes deployments, services, and scaling
- Structured logging and monitoring
- Application performance monitoring
- Cloud deployment strategies (AWS, GCP, Azure)
- Infrastructure as Code with Terraform

**Next Steps:**

You've now completed a comprehensive Python mastery series covering:

- Part 1: Python Fundamentals
- Part 2: Python Internals
- Part 3: SDLC & Architecture
- Part 4: Testing & Quality
- Part 5: Deployment & DevOps

You're ready to build and deploy production-grade Python applications! 🚀
