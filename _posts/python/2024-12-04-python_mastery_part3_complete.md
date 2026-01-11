---
title: "Python Mastery Part 3: SDLC - Requirements, Design & Architecture"
date: 2024-12-04 00:00:00 +0530
categories: [Python, Python Mastery]
tags: [Python, Software Engineering, SDLC, Architecture, Design Patterns, API-design, Requirements, System Design]
---
## Table of Contents

- [Introduction](#introduction)
- [3.1 Requirements &amp; Planning Phase](#31-requirements--planning-phase)
  - [Requirement Gathering](#requirement-gathering)
  - [Technical Design Documents](#technical-design-documents)
  - [Project Planning](#project-planning)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [3.2 Software Architecture &amp; Design Patterns](#32-software-architecture--design-patterns)
  - [Architectural Patterns](#architectural-patterns)
  - [Design Patterns - Creational](#design-patterns---creational)
  - [Design Patterns - Structural](#design-patterns---structural)
  - [Design Patterns - Behavioral](#design-patterns---behavioral)
  - [Python-Specific Patterns](#python-specific-patterns)
  - [SOLID Principles](#solid-principles)
  - [Domain-Driven Design](#domain-driven-design-ddd)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
  - [Interview Questions](#interview-questions-1)
- [3.3 API Design](#33-api-design)
  - [REST API Design](#rest-api-design)
  - [GraphQL API Design](#graphql-api-design)
  - [API Best Practices](#api-best-practices)
  - [Web Frameworks](#web-frameworks)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
  - [Interview Questions](#interview-questions-2)
- [3.4 Database Design &amp; ORM](#34-database-design--orm)
  - [Relational Database Design](#relational-database-design)
  - [SQLAlchemy](#sqlalchemy)
  - [Django ORM](#django-orm)
  - [Frequently Asked Questions](#frequently-asked-questions-3)
  - [Interview Questions](#interview-questions-3)

---

# Complete Python Mastery Part 3: SDLC - Requirements, Design & Architecture

## Introduction

Welcome to **Part 3** of the Complete Python Mastery series. Having mastered Python fundamentals (Part 1) and internals (Part 2), you're now ready to apply this knowledge to real-world software development.

**What You'll Learn in Part 3:**

This post covers the early phases of the Software Development Life Cycle (SDLC), focusing on:

- **Requirements gathering**: User stories, technical specifications, API contracts
- **Software architecture**: Architectural patterns, system design, scalability
- **Design patterns**: Gang of Four patterns + Python-specific patterns
- **SOLID principles**: Writing maintainable, extensible code
- **API design**: REST, GraphQL, gRPC best practices
- **Database design**: Schema design, ORMs, migrations, optimization

**Why This Matters:**

Most Python code failures happen in design, not implementation. This part teaches you to:

- Translate business requirements into technical specifications
- Choose appropriate architectural patterns
- Design scalable, maintainable systems
- Create robust APIs
- Structure databases efficiently
- Apply proven design patterns

**Prerequisites:**

- Part 1: Python Fundamentals
- Part 2: Python Internals (recommended)
- Basic understanding of software development
- Familiarity with databases (helpful)

Let's begin building production-grade systems! 🏗️

---

## 3.1 Requirements & Planning Phase

**SDLC Phase:** Requirements, Planning

**Relevant For:**

- [X] Requirements gathering
- [X] System design
- [ ] Development
- [ ] Testing
- [ ] Deployment
- [ ] Maintenance

Effective requirements gathering prevents costly rework and ensures alignment between business needs and technical implementation.

### Requirement Gathering

#### User Stories

```python
# User Story Format:
# As a [role], I want [feature] so that [benefit]

"""
EXAMPLE USER STORIES:

Story 1: User Authentication
As a user,
I want to log in with my email and password
So that I can access my personalized content

Acceptance Criteria:
- User can log in with valid credentials
- Invalid credentials show error message
- Session persists for 24 hours
- User can log out manually

Story 2: Product Search
As a customer,
I want to search for products by name or category
So that I can quickly find what I'm looking for

Acceptance Criteria:
- Search returns results within 1 second
- Results can be filtered by price range
- Results can be sorted by relevance, price, rating
- Search handles typos (fuzzy matching)

Story 3: Payment Processing
As a customer,
I want to pay with credit card securely
So that I can complete my purchase

Acceptance Criteria:
- Payment data is encrypted (PCI DSS compliant)
- User receives confirmation email
- Failed payments show clear error message
- Supports multiple currencies
"""

# ✅ GOOD: Convert user stories to technical tasks
class UserStoryTechnicalBreakdown:
    """
    User Story: User Authentication
  
    Technical Tasks:
    1. Design database schema (users table)
    2. Implement password hashing (bcrypt)
    3. Create authentication API endpoints
       - POST /api/auth/login
       - POST /api/auth/logout
       - GET /api/auth/me
    4. Implement JWT token generation
    5. Add session management
    6. Write unit tests (login, logout, token validation)
    7. Write integration tests (end-to-end auth flow)
    8. Add rate limiting (prevent brute force)
    9. Implement audit logging
    10. Document API endpoints (OpenAPI)
  
    Estimated Effort: 3-5 days
    Dependencies: Database setup, API framework
    Risks: Security vulnerabilities, performance bottlenecks
    """
```

#### Technical Requirements Documents

```python
"""
TECHNICAL REQUIREMENTS DOCUMENT (TRD)

Project: E-Commerce Platform API
Version: 1.0
Date: 2025-01-02

1. OVERVIEW
   - Purpose: Build REST API for e-commerce platform
   - Scope: User management, product catalog, orders, payments
   - Stakeholders: Engineering, Product, Business

2. FUNCTIONAL REQUIREMENTS

   2.1 User Management
       - User registration with email verification
       - Login with email/password or OAuth2 (Google, Facebook)
       - Password reset functionality
       - Profile management (update details, change password)
       - Role-based access control (customer, admin, vendor)
   
   2.2 Product Catalog
       - Product CRUD operations
       - Category hierarchy (nested categories)
       - Product search with filters (price, category, brand)
       - Product reviews and ratings
       - Inventory tracking
   
   2.3 Order Management
       - Shopping cart (add, update, remove items)
       - Checkout process
       - Order tracking
       - Order history
       - Invoice generation

3. NON-FUNCTIONAL REQUIREMENTS

   3.1 Performance
       - API response time: < 200ms (p95)
       - Support 1000 concurrent users
       - Database query time: < 50ms (p95)
       - Search results: < 500ms
   
   3.2 Scalability
       - Horizontal scaling (stateless API)
       - Database read replicas
       - Caching layer (Redis)
       - CDN for static assets
   
   3.3 Security
       - HTTPS only (TLS 1.3)
       - Password hashing (bcrypt, cost factor 12)
       - JWT tokens (RSA-256, 24-hour expiry)
       - Rate limiting (100 req/min per IP)
       - Input validation and sanitization
       - SQL injection prevention (parameterized queries)
       - XSS prevention
       - CSRF protection
   
   3.4 Reliability
       - Uptime: 99.9% (< 8.76 hours downtime/year)
       - Database backups: hourly incremental, daily full
       - Disaster recovery: < 4 hour RTO, < 1 hour RPO
       - Error handling and graceful degradation
   
   3.5 Monitoring
       - Application metrics (request rate, errors, latency)
       - Infrastructure metrics (CPU, memory, disk)
       - Business metrics (orders, revenue, users)
       - Alerting (PagerDuty integration)
       - Log aggregation (ELK stack)

4. TECHNOLOGY STACK

   4.1 Backend
       - Language: Python 3.11+
       - Framework: FastAPI 0.109+
       - Database: PostgreSQL 15+
       - Cache: Redis 7+
       - Queue: Celery + Redis
   
   4.2 Infrastructure
       - Cloud: AWS
       - Container: Docker
       - Orchestration: Kubernetes
       - CI/CD: GitHub Actions
       - Monitoring: Prometheus + Grafana

5. API SPECIFICATIONS

   5.1 Authentication
       POST /api/v1/auth/register
       POST /api/v1/auth/login
       POST /api/v1/auth/refresh
       POST /api/v1/auth/logout
   
   5.2 Users
       GET /api/v1/users/me
       PUT /api/v1/users/me
       DELETE /api/v1/users/me
   
   5.3 Products
       GET /api/v1/products
       GET /api/v1/products/{id}
       POST /api/v1/products (admin only)
       PUT /api/v1/products/{id} (admin only)
       DELETE /api/v1/products/{id} (admin only)

6. DATA MODELS

   6.1 User Model
       - id: UUID (primary key)
       - email: String (unique, indexed)
       - password_hash: String
       - full_name: String
       - role: Enum (customer, admin, vendor)
       - is_active: Boolean
       - created_at: Timestamp
       - updated_at: Timestamp
   
   6.2 Product Model
       - id: UUID (primary key)
       - name: String (indexed)
       - description: Text
       - price: Decimal (2 decimal places)
       - category_id: UUID (foreign key)
       - stock_quantity: Integer
       - is_active: Boolean
       - created_at: Timestamp
       - updated_at: Timestamp

7. CONSTRAINTS AND ASSUMPTIONS

   - Initial user base: 10,000 users
   - Expected growth: 50% per quarter
   - Peak traffic: 3x average (during sales events)
   - Product catalog: 100,000 products initially
   - Average order value: $50-$100
   - Payment processing: Integration with Stripe
   - Multi-region support: Not required initially

8. RISKS AND MITIGATIONS

   Risk: Payment gateway downtime
   Mitigation: Implement retry logic, queue failed payments
   
   Risk: Database bottleneck
   Mitigation: Read replicas, query optimization, caching
   
   Risk: Security breach
   Mitigation: Regular security audits, penetration testing
"""

# ✅ GOOD: Structured TRD as code
from dataclasses import dataclass
from typing import List, Dict
from enum import Enum

class NFRCategory(Enum):
    PERFORMANCE = "performance"
    SCALABILITY = "scalability"
    SECURITY = "security"
    RELIABILITY = "reliability"

@dataclass
class NonFunctionalRequirement:
    category: NFRCategory
    metric: str
    target: str
    measurement: str

@dataclass
class TechnicalRequirement:
    id: str
    title: str
    description: str
    priority: str  # High, Medium, Low
    acceptance_criteria: List[str]
    dependencies: List[str]
    estimated_effort: str

# Define requirements programmatically
nfr_performance = NonFunctionalRequirement(
    category=NFRCategory.PERFORMANCE,
    metric="API Response Time (p95)",
    target="< 200ms",
    measurement="Prometheus metrics"
)

req_authentication = TechnicalRequirement(
    id="REQ-001",
    title="User Authentication",
    description="Implement secure user authentication with JWT",
    priority="High",
    acceptance_criteria=[
        "Users can log in with email and password",
        "JWT tokens expire after 24 hours",
        "Refresh tokens supported",
        "Rate limiting: 5 login attempts per minute"
    ],
    dependencies=["Database schema", "API framework"],
    estimated_effort="3-5 days"
)
```

#### API Specifications

```python
"""
API SPECIFICATION DOCUMENT

Use OpenAPI (Swagger) for API contracts
"""

# FastAPI automatically generates OpenAPI specs
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, EmailStr, Field
from typing import Optional, List
from datetime import datetime

app = FastAPI(
    title="E-Commerce API",
    description="REST API for e-commerce platform",
    version="1.0.0",
    docs_url="/docs",  # Swagger UI
    redoc_url="/redoc"  # ReDoc
)

# Request/Response models (API contract)
class UserRegistration(BaseModel):
    """User registration request"""
    email: EmailStr = Field(..., description="User email address")
    password: str = Field(..., min_length=8, description="Password (min 8 chars)")
    full_name: str = Field(..., min_length=1, description="User full name")
  
    class Config:
        schema_extra = {
            "example": {
                "email": "user@example.com",
                "password": "SecurePass123!",
                "full_name": "John Doe"
            }
        }

class UserResponse(BaseModel):
    """User response"""
    id: str = Field(..., description="User ID (UUID)")
    email: EmailStr
    full_name: str
    role: str
    is_active: bool
    created_at: datetime
  
    class Config:
        schema_extra = {
            "example": {
                "id": "123e4567-e89b-12d3-a456-426614174000",
                "email": "user@example.com",
                "full_name": "John Doe",
                "role": "customer",
                "is_active": True,
                "created_at": "2025-01-02T10:00:00Z"
            }
        }

class ErrorResponse(BaseModel):
    """Standard error response"""
    error_code: str
    message: str
    details: Optional[Dict] = None

# API endpoints with OpenAPI documentation
@app.post(
    "/api/v1/auth/register",
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    responses={
        201: {"description": "User created successfully"},
        400: {"model": ErrorResponse, "description": "Invalid input"},
        409: {"model": ErrorResponse, "description": "Email already exists"}
    },
    tags=["Authentication"],
    summary="Register new user",
    description="""
    Register a new user with email and password.
  
    Password requirements:
    - Minimum 8 characters
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one number
    - At least one special character
  
    On success, returns user details (password excluded).
    """
)
async def register_user(user: UserRegistration):
    """Register new user endpoint"""
    # Implementation here
    pass

# ✅ GOOD: Version API specifications
"""
API Versioning Strategy:

1. URL Versioning (Recommended)
   - /api/v1/users
   - /api/v2/users
   
2. Header Versioning
   - Accept: application/vnd.api+json; version=1
   
3. Query Parameter
   - /api/users?version=1

Best Practice: Use URL versioning for major changes,
header versioning for minor changes.
"""

# Document breaking changes
"""
CHANGELOG - API v2.0.0

BREAKING CHANGES:
- Removed: /api/v1/products (legacy endpoint)
- Changed: UserResponse.role is now an enum (was string)
- Changed: Authentication now requires Bearer token (was API key)

NEW FEATURES:
- Added: /api/v2/products/search (advanced search)
- Added: Pagination on all list endpoints
- Added: Filtering and sorting support

DEPRECATIONS:
- /api/v1/users/list (use /api/v2/users instead)
  Deprecation date: 2025-01-02
  Removal date: 2025-07-02

MIGRATION GUIDE:
See docs/migration-v1-to-v2.md
"""
```

### Technical Design Documents

#### Design Doc Structure

```python
"""
TECHNICAL DESIGN DOCUMENT (TDD)

Title: Product Search Service
Author: Engineering Team
Date: 2025-01-02
Status: Review
Reviewers: @architect, @tech-lead, @security

1. OVERVIEW

   1.1 Background
       Current product search is slow (> 2 seconds) and doesn't
       support advanced features like fuzzy matching, faceted search,
       or relevance ranking.
   
   1.2 Goals
       - Reduce search latency to < 500ms (p95)
       - Support fuzzy search (typo tolerance)
       - Enable faceted search (filter by category, price, brand)
       - Relevance ranking based on user behavior
       - Support 10,000+ searches per second
   
   1.3 Non-Goals
       - Natural language search (future enhancement)
       - Image-based search
       - Multi-language support (MVP)

2. PROPOSED SOLUTION

   2.1 High-Level Architecture
   
       [User] → [API Gateway] → [Search Service] → [Elasticsearch]
                                         ↓
                                  [Product Database]
   
   2.2 Components
   
       a) Search Service (Python/FastAPI)
          - Handles search requests
          - Builds Elasticsearch queries
          - Aggregates and ranks results
          - Caches frequent searches (Redis)
   
       b) Elasticsearch Cluster
          - Stores product search indices
          - Performs full-text search
          - Supports faceted search
          - Analyzes search queries
   
       c) Data Sync Service
          - Syncs product data from PostgreSQL to Elasticsearch
          - Handles incremental updates
          - Runs as background job (Celery)

3. DETAILED DESIGN

   3.1 Data Model
   
   Elasticsearch Index: products_v1
   
   {
     "id": "uuid",
     "name": "Product Name",
     "description": "Full description",
     "category": "Electronics > Phones",
     "brand": "BrandName",
     "price": 299.99,
     "in_stock": true,
     "rating": 4.5,
     "review_count": 150,
     "created_at": "2025-01-01T00:00:00Z"
   }
   
   Index Settings:
   - Shards: 5
   - Replicas: 2
   - Analyzer: standard + custom synonym filter
   
   3.2 Search Algorithm
   
   Multi-field search with boosting:
   - name^3 (highest priority)
   - description^2
   - category^1.5
   - brand^1
   
   Fuzzy matching: Levenshtein distance = 2
   
   Ranking factors:
   - Text relevance (BM25 score)
   - Product popularity (review_count)
   - Product rating
   - Price competitiveness
   - Recency boost (newer products)
   
   3.3 Caching Strategy
   
   L1 Cache (Redis):
   - Cache frequent searches (TTL: 5 minutes)
   - Cache structure: search_query -> [product_ids]
   - Invalidate on product updates
   
   L2 Cache (Application):
   - In-memory cache for hot products
   - LRU eviction policy
   - Max size: 10,000 products

4. API DESIGN

   GET /api/v1/products/search
   
   Query Parameters:
   - q: Search query (required)
   - category: Filter by category (optional)
   - min_price: Minimum price (optional)
   - max_price: Maximum price (optional)
   - brand: Filter by brand (optional)
   - sort: Sort order (relevance|price_asc|price_desc|rating)
   - page: Page number (default: 1)
   - page_size: Results per page (default: 20, max: 100)
   
   Response:
   {
     "results": [
       {
         "id": "uuid",
         "name": "Product Name",
         "price": 299.99,
         "rating": 4.5,
         "in_stock": true
       }
     ],
     "total": 150,
     "page": 1,
     "page_size": 20,
     "facets": {
       "categories": [
         {"name": "Electronics", "count": 80},
         {"name": "Books", "count": 30}
       ],
       "brands": [
         {"name": "BrandA", "count": 50},
         {"name": "BrandB", "count": 40}
       ]
     }
   }

5. DATA FLOW

   5.1 Search Request Flow
   
   1. User sends search request
   2. API Gateway authenticates and rate limits
   3. Search Service checks Redis cache
   4. If cache hit: return cached results
   5. If cache miss:
      a. Build Elasticsearch query
      b. Execute query
      c. Aggregate and rank results
      d. Cache results in Redis
   6. Return results to user
   
   5.2 Product Update Flow
   
   1. Product updated in PostgreSQL
   2. Trigger database event (PostgreSQL NOTIFY)
   3. Sync Service receives event
   4. Update Elasticsearch index
   5. Invalidate Redis cache for affected queries

6. SCALABILITY

   - Horizontal scaling: Stateless search service
   - Elasticsearch: 5 shards, 2 replicas
   - Redis: Cluster mode (3 masters, 3 replicas)
   - Auto-scaling: Scale based on request rate
   - Load balancing: Round-robin with health checks

7. PERFORMANCE

   - Search latency: < 500ms (p95)
   - Cache hit rate: > 60%
   - Elasticsearch query time: < 200ms
   - Throughput: 10,000 searches/second
   - Database sync lag: < 1 second

8. MONITORING

   - Metrics: Search latency, cache hit rate, error rate
   - Logs: Search queries, Elasticsearch queries, errors
   - Alerts: Latency > 1s, error rate > 1%, cache hit rate < 40%
   - Dashboards: Grafana (search performance, cache metrics)

9. SECURITY

   - Rate limiting: 100 searches per minute per user
   - Input validation: Sanitize search queries
   - SQL injection: Not applicable (NoSQL)
   - Query complexity limits: Max 10 filters, max 100 results

10. TESTING STRATEGY

    - Unit tests: Search query builder, ranking algorithm
    - Integration tests: End-to-end search flow
    - Performance tests: Load testing (10,000 RPS)
    - Chaos tests: Elasticsearch node failure

11. ROLLOUT PLAN

    Phase 1 (Week 1): Deploy to staging
    Phase 2 (Week 2): Canary deployment (10% traffic)
    Phase 3 (Week 3): Gradual rollout (25% → 50% → 100%)
    Phase 4 (Week 4): Monitor and optimize

12. RISKS AND MITIGATIONS

    Risk: Elasticsearch cluster failure
    Mitigation: Fallback to PostgreSQL search (degraded mode)
  
    Risk: Cache stampede (cache expires during high traffic)
    Mitigation: Probabilistic early expiration, request coalescing
  
    Risk: Data inconsistency between PostgreSQL and Elasticsearch
    Mitigation: Periodic full reindex (daily), consistency checks

13. ALTERNATIVES CONSIDERED

    Alternative 1: PostgreSQL Full-Text Search
    Pros: No additional infrastructure
    Cons: Slower, limited features, doesn't scale
    Reason rejected: Doesn't meet latency requirements
  
    Alternative 2: Algolia (SaaS)
    Pros: Fully managed, excellent performance
    Cons: High cost, vendor lock-in, data privacy concerns
    Reason rejected: Cost-prohibitive at scale

14. OPEN QUESTIONS

    Q: Should we support saved searches?
    A: Not in MVP, but design should allow for future enhancement
  
    Q: How to handle search analytics?
    A: Log all searches to data warehouse for analysis

15. APPENDIX

    A. Elasticsearch Query Example
    B. Performance Benchmarks
    C. Cost Estimates
    D. Database Schema
"""

# ✅ GOOD: Design doc template as code
from dataclasses import dataclass
from typing import List, Dict, Optional
from enum import Enum

class DesignDocStatus(Enum):
    DRAFT = "draft"
    REVIEW = "review"
    APPROVED = "approved"
    IMPLEMENTED = "implemented"

@dataclass
class TechnicalDesignDoc:
    title: str
    author: str
    date: str
    status: DesignDocStatus
    reviewers: List[str]
  
    # Overview
    background: str
    goals: List[str]
    non_goals: List[str]
  
    # Solution
    architecture: str
    components: List[Dict]
    data_models: List[Dict]
    api_design: Dict
  
    # NFRs
    performance_targets: Dict
    scalability_plan: str
    security_considerations: List[str]
  
    # Implementation
    rollout_plan: List[str]
    testing_strategy: str
    monitoring_plan: Dict
  
    # Risk management
    risks: List[Dict]  # {risk, likelihood, impact, mitigation}
    alternatives: List[Dict]  # {alternative, pros, cons, reason_rejected}
  
    # Metadata
    open_questions: List[str]
    dependencies: List[str]
    estimated_effort: str
```

### Project Planning

#### Breaking Down Work

```python
"""
AGILE PROJECT BREAKDOWN

Epic: E-Commerce Platform MVP
Duration: 12 weeks
Team Size: 5 engineers

EPICS (Large features):

Epic 1: User Management
  - User registration
  - Authentication
  - Profile management
  - Role-based access

Epic 2: Product Catalog
  - Product CRUD
  - Categories
  - Search functionality
  - Inventory management

Epic 3: Order Management
  - Shopping cart
  - Checkout
  - Order tracking
  - Payment integration

USER STORIES (1-3 days each):

Epic 1 → Story 1.1: User Registration
  - Task 1.1.1: Design user database schema (4 hours)
  - Task 1.1.2: Implement registration API (8 hours)
  - Task 1.1.3: Add email verification (6 hours)
  - Task 1.1.4: Write unit tests (4 hours)
  - Task 1.1.5: Write integration tests (4 hours)
  Total: 26 hours (~3 days)

Epic 1 → Story 1.2: User Authentication
  - Task 1.2.1: Implement JWT token generation (4 hours)
  - Task 1.2.2: Create login API endpoint (6 hours)
  - Task 1.2.3: Add refresh token logic (4 hours)
  - Task 1.2.4: Implement logout (2 hours)
  - Task 1.2.5: Add rate limiting (4 hours)
  - Task 1.2.6: Write tests (6 hours)
  Total: 26 hours (~3 days)

TASKS (Sub-day granularity):
- Granular implementation steps
- Typically 2-8 hours each
- Assigned to individual engineers
"""

# ✅ GOOD: Estimation techniques
class EstimationTechnique:
    """
    1. STORY POINTS (Relative sizing)
       - 1 point: Few hours (trivial change)
       - 2 points: Half day
       - 3 points: 1 day
       - 5 points: 2-3 days
       - 8 points: 1 week
       - 13 points: Too large, split it!
   
    2. T-SHIRT SIZING
       - XS: < 1 day
       - S: 1-2 days
       - M: 3-5 days
       - L: 1-2 weeks
       - XL: Too large, split it!
  
    3. THREE-POINT ESTIMATION
       - Best case: 2 days
       - Most likely: 4 days
       - Worst case: 8 days
       - Expected = (Best + 4×Most Likely + Worst) / 6
       - Expected = (2 + 16 + 8) / 6 = 4.3 days
  
    4. VELOCITY-BASED PLANNING
       - Measure team velocity (points per sprint)
       - Typical team: 20-40 points per 2-week sprint
       - Use historical data to forecast completion
    """
  
    @staticmethod
    def three_point_estimate(best, most_likely, worst):
        """Calculate expected duration using PERT formula"""
        return (best + 4 * most_likely + worst) / 6
  
    @staticmethod
    def buffer_time(estimate, confidence=0.8):
        """Add buffer based on confidence level"""
        if confidence >= 0.9:
            return estimate * 1.1  # 10% buffer
        elif confidence >= 0.7:
            return estimate * 1.25  # 25% buffer
        else:
            return estimate * 1.5  # 50% buffer

# Sprint Planning Example
sprint_plan = {
    "sprint": 1,
    "duration": "2 weeks",
    "capacity": 80,  # hours (2 weeks × 40 hours)
    "team": ["Engineer A", "Engineer B", "Engineer C", "Engineer D"],
  
    "committed_stories": [
        {
            "id": "STORY-1",
            "title": "User Registration",
            "points": 5,
            "estimated_hours": 26,
            "assignee": "Engineer A"
        },
        {
            "id": "STORY-2",
            "title": "User Authentication",
            "points": 5,
            "estimated_hours": 26,
            "assignee": "Engineer B"
        },
        {
            "id": "STORY-3",
            "title": "Product CRUD API",
            "points": 8,
            "estimated_hours": 28,
            "assignee": "Engineer C"
        }
    ],
  
    "total_points": 18,
    "total_hours": 80,
    "buffer": 0,  # No buffer (risky!)
  
    "dependencies": [
        "Database setup must be complete",
        "API framework configured"
    ],
  
    "risks": [
        "Authentication complexity may be underestimated",
        "Engineer A on vacation last 2 days"
    ]
}

# ✅ GOOD: Track technical debt
technical_debt_register = [
    {
        "id": "TD-001",
        "title": "Remove hardcoded database credentials",
        "description": "Use environment variables or secrets manager",
        "priority": "High",
        "estimated_effort": "2 hours",
        "interest_rate": "Security risk",
        "created_date": "2025-01-01",
        "target_resolution": "Sprint 2"
    },
    {
        "id": "TD-002",
        "title": "Add database connection pooling",
        "description": "Currently creating new connection per request",
        "priority": "Medium",
        "estimated_effort": "4 hours",
        "interest_rate": "Performance degradation under load",
        "created_date": "2025-01-01",
        "target_resolution": "Sprint 3"
    },
    {
        "id": "TD-003",
        "title": "Refactor duplicate validation logic",
        "description": "Email validation duplicated in 3 places",
        "priority": "Low",
        "estimated_effort": "3 hours",
        "interest_rate": "Maintenance burden, inconsistency risk",
        "created_date": "2025-01-02",
        "target_resolution": "Sprint 4"
    }
]
```

### Frequently Asked Questions

**Q1: How detailed should requirements be before starting development?**

**A:**

```python
# Balance: Too little detail → rework; Too much → analysis paralysis

# ❌ TOO VAGUE:
"""
Build a user authentication system.
"""
# Missing: Security requirements, auth methods, session management, etc.

# ❌ TOO DETAILED:
"""
User Authentication System Requirements (50-page document):
- Button color: #3498db
- Button border radius: 4px
- Input field padding: 8px 12px
- Error message font: 14px Arial
- (Hundreds more UI specifications...)
"""
# Over-specifies implementation details

# ✅ JUST RIGHT:
"""
User Authentication Requirements:

Functional:
- Users log in with email and password
- Support OAuth2 (Google, GitHub)
- Session expires after 24 hours of inactivity
- Password reset via email
- Multi-factor authentication (optional)

Non-Functional:
- Login response time: < 500ms
- Password hashing: bcrypt (cost factor 12)
- JWT tokens with RS256 signing
- Rate limiting: 5 failed attempts per minute

Acceptance Criteria:
- User can log in with valid credentials
- Invalid login shows clear error
- Session persists across browser restarts
- MFA can be enabled/disabled

Out of Scope:
- Biometric authentication
- Social login beyond Google/GitHub
- Single sign-on (SSO)
"""

# Rule of thumb:
# - Start with high-level requirements
# - Add detail as you learn more
# - Write acceptance criteria, not implementation
# - Leave room for technical decisions
# - Iterate based on feedback

# Agile approach: "Just enough" documentation
# - User story: 1 paragraph
# - Acceptance criteria: 3-5 bullets
# - Technical spike: 1-2 days investigation
# - Detailed design: During sprint
```

**Why This Matters:** Right level of detail prevents both ambiguity and over-engineering.

---

**Q2: How do you estimate work when requirements are unclear?**

**A:**

```python
# Use progressive refinement and risk buffers

# Scenario: "Build a recommendation engine"

# Step 1: Identify unknowns
unknowns = [
    "Algorithm complexity (collaborative filtering? ML-based?)",
    "Data volume (100 products or 1 million?)",
    "Performance requirements (batch or real-time?)",
    "Integration points (existing systems?)"
]

# Step 2: Create rough estimate ranges
initial_estimate = {
    "best_case": "2 weeks",  # Simple rule-based system
    "most_likely": "6 weeks",  # Standard collab filtering
    "worst_case": "12 weeks",  # Full ML pipeline with training
    "confidence": "Low (30%)"
}

# Step 3: Time-box investigation (spike)
spike = {
    "duration": "2-3 days",
    "goals": [
        "Prototype 2-3 algorithm approaches",
        "Measure performance on sample data",
        "Identify technical constraints",
        "Estimate data pipeline complexity"
    ],
    "outcome": "Refined estimate with 70% confidence"
}

# Step 4: Break into phases
phased_approach = {
    "phase_1": {
        "scope": "MVP - Simple rule-based recommendations",
        "estimate": "2 weeks",
        "confidence": "High (80%)"
    },
    "phase_2": {
        "scope": "Collaborative filtering",
        "estimate": "3 weeks",
        "confidence": "Medium (60%)"
    },
    "phase_3": {
        "scope": "ML-based personalization",
        "estimate": "4-6 weeks",
        "confidence": "Low (40%)"
    }
}

# Step 5: Add buffers
def estimate_with_buffer(base_estimate, confidence):
    """Add buffer based on uncertainty"""
    if confidence > 0.8:
        return base_estimate * 1.1  # 10% buffer
    elif confidence > 0.6:
        return base_estimate * 1.3  # 30% buffer
    elif confidence > 0.4:
        return base_estimate * 1.5  # 50% buffer
    else:
        return base_estimate * 2.0  # 100% buffer (very uncertain)

# Result: "Phase 1 will take 2.2 weeks with high confidence"

# ✅ GOOD: Communicate uncertainty
estimate_communication = """
Recommendation Engine Estimate:

MVP (High Confidence):
- Simple rule-based recommendations
- 2-3 weeks
- Delivers basic functionality

Full Solution (Low Confidence):
- ML-based personalization
- 6-12 weeks
- Requires clarification on:
  * Algorithm requirements
  * Performance targets
  * Data availability
  
Recommendation:
- Start with MVP (2-3 weeks)
- Run technical spike (2 days)
- Re-estimate full solution after MVP
"""
```

**Why This Matters:** Acknowledging uncertainty prevents missed deadlines and manages stakeholder expectations.

---

### Interview Questions

**Question 1: How would you gather requirements for a new feature?**

**Difficulty:** Junior/Mid-Level

**SDLC Relevance:** Requirements Gathering

**Answer:**

```python
"""
REQUIREMENT GATHERING PROCESS:

1. UNDERSTAND THE PROBLEM
   - Talk to stakeholders (product, business, users)
   - Ask "Why is this needed?" (understand business value)
   - Identify success metrics (how to measure success)

2. DEFINE USE CASES
   - Who will use this feature? (user personas)
   - What are they trying to accomplish?
   - What's the happy path? Edge cases?

3. DOCUMENT REQUIREMENTS
   - Functional requirements (what it does)
   - Non-functional requirements (performance, security, etc.)
   - Acceptance criteria (definition of done)
   - Constraints (technical, business, timeline)

4. VALIDATE ASSUMPTIONS
   - Review with stakeholders
   - Create mockups/prototypes
   - Get feedback early

5. PRIORITIZE (MoSCoW method)
   - Must have: Core functionality
   - Should have: Important but not critical
   - Could have: Nice to have
   - Won't have: Explicitly out of scope

EXAMPLE:

Feature Request: "Add product recommendations"

Questions to Ask:
Q: Why do users need recommendations?
A: Increase discovery and sales

Q: Where should recommendations appear?
A: Product detail page, homepage

Q: What types of recommendations?
A: Similar products, frequently bought together

Q: How many recommendations to show?
A: 4-6 products

Q: Performance requirements?
A: Load within 200ms

Q: How to measure success?
A: Click-through rate, conversion rate

Requirements Document:

Functional:
- Show 4-6 recommended products on product detail page
- Recommendations based on collaborative filtering
- Update recommendations daily (batch process)

Non-Functional:
- Recommendation load time: < 200ms
- Fallback to popular products if no recommendations
- A/B testing capability

Acceptance Criteria:
- Recommendations appear within 200ms
- At least 4 recommendations shown (if available)
- CTR > 5% (baseline metric)
- Recommendations change as user behavior updates

Out of Scope (V1):
- Real-time personalization
- ML-based recommendations (future enhancement)
"""

# ✅ GOOD: Document as user story
user_story = """
As a customer,
I want to see product recommendations
So that I can discover related products I might like

Acceptance Criteria:
- Recommendations appear on product detail page
- Shows 4-6 similar or complementary products
- Loads within 200ms
- Falls back to popular products if no recommendations
- Tracked in analytics (impressions, clicks)

Technical Notes:
- Use collaborative filtering algorithm
- Cache recommendations (update daily)
- Serve from Redis for performance
- Implement as microservice

Estimated: 5 story points (1 week)
"""
```

**Key Points to Mention:**

- Talk to stakeholders to understand business value
- Define use cases and user personas
- Document functional and non-functional requirements
- Create acceptance criteria (definition of done)
- Validate assumptions early
- Prioritize features (MoSCoW)

**Why This Matters:** Good requirements prevent costly rework and ensure alignment between business and technical teams.

---

### Key Takeaways

**Requirements & Planning:**

- **User stories**: "As a [role], I want [feature] so that [benefit]"
- **Acceptance criteria**: Define "done" clearly and measurably
- **Technical requirements**: Document functional and non-functional requirements
- **NFRs**: Performance, scalability, security, reliability targets
- **API specifications**: Use OpenAPI/Swagger for contracts
- **Design documents**: Architecture, data flow, alternatives, risks
- **Estimation**: Use story points, three-point estimates, add buffers
- **Work breakdown**: Epic → Story → Task hierarchy
- **Technical debt**: Track and prioritize repayment

**Best Practices:**

- Start with user needs, not technical solutions
- Balance detail (just enough, not too much)
- Validate assumptions early
- Communicate uncertainty in estimates
- Document decisions and trade-offs
- Track dependencies and risks

---

(Continuing with Section 3.2 - Software Architecture & Design Patterns...)

## 3.2 Software Architecture & Design Patterns

**SDLC Phase:** System Design, Development

**Relevant For:**

- [ ] Requirements gathering
- [X] System design
- [X] Development
- [ ] Testing
- [ ] Deployment
- [X] Maintenance (refactoring)

Software architecture defines the high-level structure of your system, while design patterns provide proven solutions to common problems.

### Architectural Patterns

#### Monolithic Architecture

```python
"""
MONOLITHIC ARCHITECTURE

Single deployable unit containing all functionality.

Structure:
application/
├── api/
│   ├── routes.py
│   └── controllers.py
├── services/
│   ├── user_service.py
│   ├── product_service.py
│   └── order_service.py
├── models/
│   ├── user.py
│   ├── product.py
│   └── order.py
├── database/
│   └── connection.py
└── main.py
"""

# Example: Flask monolithic application
from flask import Flask, jsonify, request
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

app = Flask(__name__)
engine = create_engine('postgresql://localhost/shop')
Session = sessionmaker(bind=engine)

# User endpoints
@app.route('/api/users', methods=['POST'])
def create_user():
    session = Session()
    user = User(**request.json)
    session.add(user)
    session.commit()
    return jsonify(user.to_dict()), 201

# Product endpoints
@app.route('/api/products', methods=['GET'])
def list_products():
    session = Session()
    products = session.query(Product).all()
    return jsonify([p.to_dict() for p in products])

# Order endpoints
@app.route('/api/orders', methods=['POST'])
def create_order():
    session = Session()
    order = Order(**request.json)
    session.add(order)
    session.commit()
    return jsonify(order.to_dict()), 201

# Pros:
# ✅ Simple to develop and deploy
# ✅ Easy to test (everything in one place)
# ✅ No distributed system complexity
# ✅ Strong consistency (single database)

# Cons:
# ❌ Tight coupling (hard to modify)
# ❌ Scaling challenges (must scale entire app)
# ❌ Deployment risk (small change = full redeployment)
# ❌ Technology lock-in (single tech stack)

# When to use:
# - Small to medium applications
# - MVP or prototypes
# - Team < 10 developers
# - Simple business domain
```

#### Microservices Architecture

```python
"""
MICROSERVICES ARCHITECTURE

Multiple independent services, each owning its domain.

Structure:
services/
├── user-service/
│   ├── api/
│   ├── models/
│   ├── database/
│   └── main.py
├── product-service/
│   ├── api/
│   ├── models/
│   ├── database/
│   └── main.py
└── order-service/
    ├── api/
    ├── models/
    ├── database/
    └── main.py
"""

# User Service (FastAPI)
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import httpx

app = FastAPI()

class User(BaseModel):
    id: str
    name: str
    email: str

@app.post("/users")
async def create_user(user: User):
    # Store in user database
    await user_repository.save(user)
  
    # Publish event to message queue
    await event_bus.publish("user.created", user.dict())
  
    return user

# Order Service (FastAPI)
order_app = FastAPI()

class Order(BaseModel):
    id: str
    user_id: str
    product_ids: list[str]
    total: float

@order_app.post("/orders")
async def create_order(order: Order):
    # Verify user exists (call user service)
    async with httpx.AsyncClient() as client:
        response = await client.get(f"http://user-service/users/{order.user_id}")
        if response.status_code != 200:
            raise HTTPException(404, "User not found")
  
    # Verify products exist (call product service)
    async with httpx.AsyncClient() as client:
        for product_id in order.product_ids:
            response = await client.get(f"http://product-service/products/{product_id}")
            if response.status_code != 200:
                raise HTTPException(404, f"Product {product_id} not found")
  
    # Create order
    await order_repository.save(order)
  
    # Publish event
    await event_bus.publish("order.created", order.dict())
  
    return order

# Pros:
# ✅ Independent deployment
# ✅ Technology diversity (each service can use different tech)
# ✅ Team autonomy (separate teams per service)
# ✅ Scalability (scale services independently)
# ✅ Fault isolation (service failure doesn't crash entire system)

# Cons:
# ❌ Complex distributed system
# ❌ Network latency (inter-service calls)
# ❌ Data consistency challenges
# ❌ Harder to test (requires running multiple services)
# ❌ Operational overhead (monitoring, logging, tracing)

# When to use:
# - Large applications
# - Multiple teams (> 10 developers)
# - Need independent scaling
# - Complex business domains
```

#### Event-Driven Architecture

```python
"""
EVENT-DRIVEN ARCHITECTURE

Services communicate via events (asynchronous messaging).

Components:
- Event Producer: Publishes events
- Event Bus: Message broker (RabbitMQ, Kafka, AWS SNS/SQS)
- Event Consumer: Subscribes to events
"""

# Example with RabbitMQ
import pika
import json

# Event Bus abstraction
class EventBus:
    def __init__(self, host='localhost'):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host)
        )
        self.channel = self.connection.channel()
  
    def publish(self, event_type: str, data: dict):
        """Publish event to exchange"""
        self.channel.exchange_declare(
            exchange='events',
            exchange_type='topic',
            durable=True
        )
    
        self.channel.basic_publish(
            exchange='events',
            routing_key=event_type,
            body=json.dumps(data),
            properties=pika.BasicProperties(
                delivery_mode=2,  # Persistent
            )
        )
  
    def subscribe(self, event_type: str, callback):
        """Subscribe to event type"""
        self.channel.exchange_declare(
            exchange='events',
            exchange_type='topic',
            durable=True
        )
    
        # Create queue for this consumer
        result = self.channel.queue_declare(queue='', exclusive=True)
        queue_name = result.method.queue
    
        # Bind queue to event type
        self.channel.queue_bind(
            exchange='events',
            queue=queue_name,
            routing_key=event_type
        )
    
        # Set up consumer
        self.channel.basic_consume(
            queue=queue_name,
            on_message_callback=callback,
            auto_ack=True
        )

# Producer: User Service
class UserService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus
  
    async def create_user(self, user_data: dict):
        # Save user to database
        user = await self.repository.save(user_data)
    
        # Publish event
        self.event_bus.publish('user.created', {
            'user_id': user.id,
            'email': user.email,
            'timestamp': datetime.utcnow().isoformat()
        })
    
        return user

# Consumer: Email Service
class EmailService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus
        # Subscribe to user.created events
        self.event_bus.subscribe('user.created', self.handle_user_created)
  
    def handle_user_created(self, ch, method, properties, body):
        """Handle user.created event"""
        data = json.loads(body)
    
        # Send welcome email
        self.send_email(
            to=data['email'],
            subject='Welcome!',
            body=f'Welcome user {data["user_id"]}!'
        )
    
        print(f"Sent welcome email to {data['email']}")

# Consumer: Analytics Service
class AnalyticsService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus
        self.event_bus.subscribe('user.created', self.handle_user_created)
        self.event_bus.subscribe('order.created', self.handle_order_created)
  
    def handle_user_created(self, ch, method, properties, body):
        data = json.loads(body)
        # Track user registration
        self.track_event('user_registered', data)
  
    def handle_order_created(self, ch, method, properties, body):
        data = json.loads(body)
        # Track order
        self.track_event('order_placed', data)

# Pros:
# ✅ Loose coupling (services don't know about each other)
# ✅ Async processing (non-blocking)
# ✅ Easy to add new consumers (just subscribe)
# ✅ Event replay (reprocess events)
# ✅ Audit trail (all events logged)

# Cons:
# ❌ Eventual consistency (not immediate)
# ❌ Complex debugging (distributed traces needed)
# ❌ Message ordering challenges
# ❌ Duplicate message handling
# ❌ Event versioning complexity

# When to use:
# - Need loose coupling
# - Async workflows
# - Event sourcing
# - Real-time updates
```

#### CQRS (Command Query Responsibility Segregation)

```python
"""
CQRS: Separate read and write models

Commands: Change state (write)
Queries: Read state (read)
"""

from abc import ABC, abstractmethod
from typing import List
from dataclasses import dataclass
from datetime import datetime

# Commands (Write side)
@dataclass
class CreateUserCommand:
    email: str
    name: str

@dataclass
class UpdateUserCommand:
    user_id: str
    name: str

# Command Handler
class CommandHandler(ABC):
    @abstractmethod
    async def handle(self, command):
        pass

class CreateUserCommandHandler(CommandHandler):
    def __init__(self, repository, event_bus):
        self.repository = repository
        self.event_bus = event_bus
  
    async def handle(self, command: CreateUserCommand):
        # Validate
        if await self.repository.exists_by_email(command.email):
            raise ValueError("Email already exists")
    
        # Create user (write model)
        user = User(
            id=generate_uuid(),
            email=command.email,
            name=command.name,
            created_at=datetime.utcnow()
        )
    
        await self.repository.save(user)
    
        # Publish event
        await self.event_bus.publish('user.created', user.to_dict())
    
        return user.id

# Queries (Read side)
@dataclass
class GetUserQuery:
    user_id: str

@dataclass
class SearchUsersQuery:
    query: str
    page: int
    page_size: int

# Query Handler
class QueryHandler(ABC):
    @abstractmethod
    async def handle(self, query):
        pass

class GetUserQueryHandler(QueryHandler):
    def __init__(self, read_repository):
        self.read_repository = read_repository
  
    async def handle(self, query: GetUserQuery):
        # Read from optimized read model
        return await self.read_repository.get_by_id(query.user_id)

class SearchUsersQueryHandler(QueryHandler):
    def __init__(self, search_service):
        self.search_service = search_service
  
    async def handle(self, query: SearchUsersQuery):
        # Read from Elasticsearch (optimized for search)
        return await self.search_service.search(
            query=query.query,
            page=query.page,
            page_size=query.page_size
        )

# Command/Query Bus
class CommandBus:
    def __init__(self):
        self.handlers = {}
  
    def register(self, command_type, handler):
        self.handlers[command_type] = handler
  
    async def execute(self, command):
        command_type = type(command)
        handler = self.handlers.get(command_type)
        if not handler:
            raise ValueError(f"No handler for {command_type}")
        return await handler.handle(command)

class QueryBus:
    def __init__(self):
        self.handlers = {}
  
    def register(self, query_type, handler):
        self.handlers[query_type] = handler
  
    async def execute(self, query):
        query_type = type(query)
        handler = self.handlers.get(query_type)
        if not handler:
            raise ValueError(f"No handler for {query_type}")
        return await handler.handle(query)

# Usage in API
from fastapi import FastAPI

app = FastAPI()
command_bus = CommandBus()
query_bus = QueryBus()

# Register handlers
command_bus.register(CreateUserCommand, CreateUserCommandHandler(...))
query_bus.register(GetUserQuery, GetUserQueryHandler(...))

@app.post("/users")
async def create_user(email: str, name: str):
    command = CreateUserCommand(email=email, name=name)
    user_id = await command_bus.execute(command)
    return {"user_id": user_id}

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    query = GetUserQuery(user_id=user_id)
    user = await query_bus.execute(query)
    return user

# Pros:
# ✅ Optimized read and write models separately
# ✅ Scalability (scale reads and writes independently)
# ✅ Clear separation of concerns
# ✅ Better performance (optimized queries)

# Cons:
# ❌ Increased complexity
# ❌ Eventual consistency between read/write models
# ❌ More code to maintain
```

#### Hexagonal Architecture (Ports & Adapters)

```python
"""
HEXAGONAL ARCHITECTURE

Domain logic isolated from infrastructure.

Layers:
- Domain: Business logic (pure Python)
- Ports: Interfaces (abstract classes)
- Adapters: Implementations (database, API, etc.)
"""

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional, List

# Domain Layer (Core business logic)
@dataclass
class User:
    id: str
    email: str
    name: str
  
    def change_email(self, new_email: str):
        """Business rule: Email must be valid"""
        if '@' not in new_email:
            raise ValueError("Invalid email")
        self.email = new_email

class UserService:
    """Domain service (pure business logic)"""
  
    def __init__(self, user_repository: 'UserRepository'):
        self.user_repository = user_repository
  
    async def register_user(self, email: str, name: str) -> User:
        """Business logic for user registration"""
    
        # Business rule: Email must be unique
        existing = await self.user_repository.find_by_email(email)
        if existing:
            raise ValueError("Email already registered")
    
        # Create user
        user = User(
            id=self._generate_id(),
            email=email,
            name=name
        )
    
        # Save user
        await self.user_repository.save(user)
    
        return user
  
    def _generate_id(self) -> str:
        import uuid
        return str(uuid.uuid4())

# Ports (Interfaces)
class UserRepository(ABC):
    """Port: Interface for user persistence"""
  
    @abstractmethod
    async def save(self, user: User) -> None:
        pass
  
    @abstractmethod
    async def find_by_id(self, user_id: str) -> Optional[User]:
        pass
  
    @abstractmethod
    async def find_by_email(self, email: str) -> Optional[User]:
        pass

class EmailService(ABC):
    """Port: Interface for email sending"""
  
    @abstractmethod
    async def send_email(self, to: str, subject: str, body: str) -> None:
        pass

# Adapters (Implementations)

# Adapter 1: PostgreSQL Repository
from sqlalchemy import create_engine, Column, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class UserModel(Base):
    __tablename__ = 'users'
    id = Column(String, primary_key=True)
    email = Column(String, unique=True)
    name = Column(String)

class PostgreSQLUserRepository(UserRepository):
    """Adapter: PostgreSQL implementation"""
  
    def __init__(self, session):
        self.session = session
  
    async def save(self, user: User) -> None:
        model = UserModel(
            id=user.id,
            email=user.email,
            name=user.name
        )
        self.session.add(model)
        self.session.commit()
  
    async def find_by_id(self, user_id: str) -> Optional[User]:
        model = self.session.query(UserModel).filter_by(id=user_id).first()
        if not model:
            return None
        return User(id=model.id, email=model.email, name=model.name)
  
    async def find_by_email(self, email: str) -> Optional[User]:
        model = self.session.query(UserModel).filter_by(email=email).first()
        if not model:
            return None
        return User(id=model.id, email=model.email, name=model.name)

# Adapter 2: In-Memory Repository (for testing)
class InMemoryUserRepository(UserRepository):
    """Adapter: In-memory implementation for tests"""
  
    def __init__(self):
        self.users: dict[str, User] = {}
  
    async def save(self, user: User) -> None:
        self.users[user.id] = user
  
    async def find_by_id(self, user_id: str) -> Optional[User]:
        return self.users.get(user_id)
  
    async def find_by_email(self, email: str) -> Optional[User]:
        for user in self.users.values():
            if user.email == email:
                return user
        return None

# Adapter 3: SendGrid Email Service
class SendGridEmailService(EmailService):
    """Adapter: SendGrid implementation"""
  
    def __init__(self, api_key: str):
        self.api_key = api_key
  
    async def send_email(self, to: str, subject: str, body: str) -> None:
        # Send via SendGrid API
        pass

# Adapter 4: SMTP Email Service
class SMTPEmailService(EmailService):
    """Adapter: SMTP implementation"""
  
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
  
    async def send_email(self, to: str, subject: str, body: str) -> None:
        # Send via SMTP
        pass

# Dependency Injection (wiring)
def create_user_service(environment: str) -> UserService:
    """Factory: Create service with appropriate adapters"""
  
    if environment == 'production':
        # Production: Use PostgreSQL and SendGrid
        engine = create_engine('postgresql://...')
        Session = sessionmaker(bind=engine)
        session = Session()
        repository = PostgreSQLUserRepository(session)
    
    elif environment == 'test':
        # Test: Use in-memory repository
        repository = InMemoryUserRepository()
  
    return UserService(repository)

# API Layer (Adapter)
from fastapi import FastAPI, Depends

app = FastAPI()

def get_user_service() -> UserService:
    return create_user_service('production')

@app.post("/users")
async def register_user(
    email: str,
    name: str,
    service: UserService = Depends(get_user_service)
):
    user = await service.register_user(email, name)
    return {"user_id": user.id}

# Pros:
# ✅ Domain logic independent of infrastructure
# ✅ Easy to test (swap implementations)
# ✅ Easy to change adapters (e.g., PostgreSQL → MongoDB)
# ✅ Clear separation of concerns

# Cons:
# ❌ More abstraction layers
# ❌ More interfaces to maintain
# ❌ Can be overkill for simple apps
```

### Design Patterns - Creational

#### Singleton Pattern

```python
"""
SINGLETON: Ensure only one instance exists
"""

# ❌ BAD: Simple but not thread-safe
class Database:
    _instance = None
  
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.connection = "DB Connection"
        return cls._instance

# ✅ GOOD: Thread-safe singleton
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()
  
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                # Double-checked locking
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialize()
        return cls._instance
  
    def _initialize(self):
        self.connection = "DB Connection"

# ✅ BETTER: Using metaclass
class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()
  
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            with cls._lock:
                if cls not in cls._instances:
                    instance = super().__call__(*args, **kwargs)
                    cls._instances[cls] = instance
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = "DB Connection"

db1 = Database()
db2 = Database()
print(db1 is db2)  # True

# ✅ PYTHONIC: Module-level singleton
# database.py
class DatabaseConnection:
    def __init__(self):
        self.connection = "DB Connection"

# Single instance at module level
database = DatabaseConnection()

# Other modules import the instance
# from database import database
```

#### Factory Pattern

```python
"""
FACTORY: Create objects without specifying exact class
"""

from abc import ABC, abstractmethod

# Product interface
class Database(ABC):
    @abstractmethod
    def connect(self) -> str:
        pass
  
    @abstractmethod
    def query(self, sql: str):
        pass

# Concrete products
class PostgreSQL(Database):
    def connect(self) -> str:
        return "Connected to PostgreSQL"
  
    def query(self, sql: str):
        return f"PostgreSQL: {sql}"

class MySQL(Database):
    def connect(self) -> str:
        return "Connected to MySQL"
  
    def query(self, sql: str):
        return f"MySQL: {sql}"

class MongoDB(Database):
    def connect(self) -> str:
        return "Connected to MongoDB"
  
    def query(self, sql: str):
        return f"MongoDB: {sql}"

# Factory
class DatabaseFactory:
    @staticmethod
    def create(db_type: str) -> Database:
        if db_type == "postgresql":
            return PostgreSQL()
        elif db_type == "mysql":
            return MySQL()
        elif db_type == "mongodb":
            return MongoDB()
        else:
            raise ValueError(f"Unknown database type: {db_type}")

# Usage
db = DatabaseFactory.create("postgresql")
print(db.connect())  # Connected to PostgreSQL

# ✅ BETTER: Registration pattern
class DatabaseFactory:
    _creators = {}
  
    @classmethod
    def register(cls, db_type: str, creator):
        cls._creators[db_type] = creator
  
    @classmethod
    def create(cls, db_type: str) -> Database:
        creator = cls._creators.get(db_type)
        if not creator:
            raise ValueError(f"Unknown database type: {db_type}")
        return creator()

# Register databases
DatabaseFactory.register("postgresql", PostgreSQL)
DatabaseFactory.register("mysql", MySQL)
DatabaseFactory.register("mongodb", MongoDB)

# Usage
db = DatabaseFactory.create("postgresql")
```

#### Builder Pattern

```python
"""
BUILDER: Construct complex objects step by step
"""

# ❌ COMPLEX: Too many constructor parameters
class Pizza:
    def __init__(
        self,
        size,
        cheese=False,
        pepperoni=False,
        mushrooms=False,
        onions=False,
        bacon=False,
        extra_cheese=False
    ):
        self.size = size
        self.cheese = cheese
        self.pepperoni = pepperoni
        self.mushrooms = mushrooms
        self.onions = onions
        self.bacon = bacon
        self.extra_cheese = extra_cheese

# Hard to use
pizza = Pizza("large", True, True, False, True, True, False)

# ✅ GOOD: Builder pattern
class Pizza:
    def __init__(self, size):
        self.size = size
        self.toppings = []
  
    def add_topping(self, topping):
        self.toppings.append(topping)
        return self

class PizzaBuilder:
    def __init__(self, size):
        self.pizza = Pizza(size)
  
    def add_cheese(self):
        self.pizza.add_topping("cheese")
        return self
  
    def add_pepperoni(self):
        self.pizza.add_topping("pepperoni")
        return self
  
    def add_mushrooms(self):
        self.pizza.add_topping("mushrooms")
        return self
  
    def build(self) -> Pizza:
        return self.pizza

# Fluent interface
pizza = (PizzaBuilder("large")
    .add_cheese()
    .add_pepperoni()
    .add_mushrooms()
    .build())

# ✅ PYTHONIC: Using dataclass with builder
from dataclasses import dataclass, field
from typing import List

@dataclass
class Pizza:
    size: str
    toppings: List[str] = field(default_factory=list)
  
    class Builder:
        def __init__(self, size: str):
            self._size = size
            self._toppings = []
    
        def add_cheese(self):
            self._toppings.append("cheese")
            return self
    
        def add_pepperoni(self):
            self._toppings.append("pepperoni")
            return self
    
        def build(self) -> 'Pizza':
            return Pizza(self._size, self._toppings)

pizza = (Pizza.Builder("large")
    .add_cheese()
    .add_pepperoni()
    .build())
```

#### Dependency Injection

```python
"""
DEPENDENCY INJECTION: Provide dependencies from outside
"""

# ❌ BAD: Tight coupling
class UserService:
    def __init__(self):
        self.db = PostgreSQLDatabase()  # Hard-coded!
        self.email = SendGridEmailService()  # Hard-coded!
  
    def create_user(self, email, name):
        user = self.db.save(User(email, name))
        self.email.send_welcome(user)
        return user

# ✅ GOOD: Constructor injection
class UserService:
    def __init__(self, database: Database, email_service: EmailService):
        self.db = database
        self.email = email_service
  
    def create_user(self, email, name):
        user = self.db.save(User(email, name))
        self.email.send_welcome(user)
        return user

# Inject dependencies
db = PostgreSQLDatabase()
email = SendGridEmailService()
service = UserService(db, email)

# ✅ BETTER: Using dependency injection framework
from dependency_injector import containers, providers

class Container(containers.DeclarativeContainer):
    config = providers.Configuration()
  
    # Singletons
    database = providers.Singleton(
        PostgreSQLDatabase,
        connection_string=config.database.url
    )
  
    email_service = providers.Singleton(
        SendGridEmailService,
        api_key=config.sendgrid.api_key
    )
  
    # Services
    user_service = providers.Factory(
        UserService,
        database=database,
        email_service=email_service
    )

# Wire up dependencies
container = Container()
container.config.database.url.from_env("DATABASE_URL")
container.config.sendgrid.api_key.from_env("SENDGRID_API_KEY")

# Get service (dependencies auto-injected)
service = container.user_service()

# ✅ FASTAPI: Built-in dependency injection
from fastapi import Depends, FastAPI

app = FastAPI()

# Dependency
def get_database() -> Database:
    db = PostgreSQLDatabase()
    try:
        yield db
    finally:
        db.close()

def get_email_service() -> EmailService:
    return SendGridEmailService()

# Inject into endpoint
@app.post("/users")
async def create_user(
    email: str,
    name: str,
    db: Database = Depends(get_database),
    email_service: EmailService = Depends(get_email_service)
):
    user = db.save(User(email, name))
    email_service.send_welcome(user)
    return user
```

### Design Patterns - Structural

#### Adapter Pattern

```python
"""
ADAPTER: Make incompatible interfaces work together
"""

# Legacy payment gateway (incompatible interface)
class LegacyPaymentGateway:
    def process_payment(self, amount_cents: int, card_number: str):
        print(f"Processing ${amount_cents/100} via legacy gateway")
        return {"status": "success", "transaction_id": "TXN123"}

# New payment interface
class PaymentProcessor(ABC):
    @abstractmethod
    def charge(self, amount: float, payment_method: dict) -> dict:
        pass

# Adapter
class LegacyPaymentAdapter(PaymentProcessor):
    def __init__(self):
        self.legacy_gateway = LegacyPaymentGateway()
  
    def charge(self, amount: float, payment_method: dict) -> dict:
        # Convert interface
        amount_cents = int(amount * 100)
        card_number = payment_method['card_number']
    
        # Call legacy method
        result = self.legacy_gateway.process_payment(amount_cents, card_number)
    
        # Convert response
        return {
            'success': result['status'] == 'success',
            'transaction_id': result['transaction_id']
        }

# Usage
payment_processor = LegacyPaymentAdapter()
result = payment_processor.charge(99.99, {'card_number': '1234-5678'})
```

#### Decorator Pattern

```python
"""
DECORATOR: Add behavior to objects dynamically
"""

from abc import ABC, abstractmethod

# Component interface
class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float:
        pass
  
    @abstractmethod
    def description(self) -> str:
        pass

# Concrete component
class SimpleCoffee(Coffee):
    def cost(self) -> float:
        return 2.0
  
    def description(self) -> str:
        return "Simple coffee"

# Decorator base
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
  
    def cost(self) -> float:
        return self._coffee.cost()
  
    def description(self) -> str:
        return self._coffee.description()

# Concrete decorators
class Milk(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.5
  
    def description(self) -> str:
        return self._coffee.description() + ", milk"

class Sugar(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.2
  
    def description(self) -> str:
        return self._coffee.description() + ", sugar"

class Whip(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.7
  
    def description(self) -> str:
        return self._coffee.description() + ", whipped cream"

# Usage: Build coffee with decorators
coffee = SimpleCoffee()
print(f"{coffee.description()}: ${coffee.cost()}")
# Simple coffee: $2.0

coffee = Milk(coffee)
print(f"{coffee.description()}: ${coffee.cost()}")
# Simple coffee, milk: $2.5

coffee = Sugar(coffee)
coffee = Whip(coffee)
print(f"{coffee.description()}: ${coffee.cost()}")
# Simple coffee, milk, sugar, whipped cream: $3.4

# ✅ PYTHONIC: Function decorators
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed, retrying...")
        return wrapper
    return decorator

@timer
@retry(max_attempts=3)
def fetch_data():
    # Some operation
    pass
```

#### Facade Pattern

```python
"""
FACADE: Simplified interface to complex subsystem
"""

# Complex subsystem
class VideoFile:
    def __init__(self, filename):
        self.filename = filename

class CodecFactory:
    @staticmethod
    def extract(file):
        return "MPEG4"

class BitrateReader:
    @staticmethod
    def read(filename, codec):
        return "High bitrate"
  
    @staticmethod
    def convert(buffer, codec):
        return "Converted buffer"

class AudioMixer:
    def fix(self, result):
        return "Fixed audio"

# ❌ COMPLEX: Client must know all subsystems
def convert_video_complex(filename):
    file = VideoFile(filename)
    codec = CodecFactory.extract(file)
    buffer = BitrateReader.read(filename, codec)
    result = BitrateReader.convert(buffer, codec)
    audio = AudioMixer()
    result = audio.fix(result)
    return result

# ✅ FACADE: Simple interface
class VideoConverter:
    """Facade for video conversion subsystem"""
  
    def convert(self, filename: str, format: str) -> str:
        file = VideoFile(filename)
        codec = CodecFactory.extract(file)
    
        if format == "mp4":
            buffer = BitrateReader.read(filename, codec)
            result = BitrateReader.convert(buffer, codec)
            audio = AudioMixer()
            result = audio.fix(result)
            return result
        else:
            raise ValueError(f"Unsupported format: {format}")

# Simple usage
converter = VideoConverter()
result = converter.convert("video.avi", "mp4")

# Real-world example: Database facade
class DatabaseFacade:
    """Simplify database operations"""
  
    def __init__(self, connection_string: str):
        self.engine = create_engine(connection_string)
        self.Session = sessionmaker(bind=self.engine)
  
    def save(self, obj):
        """Simple save (hides session management)"""
        session = self.Session()
        try:
            session.add(obj)
            session.commit()
        except:
            session.rollback()
            raise
        finally:
            session.close()
  
    def find_by_id(self, model_class, id):
        """Simple query (hides session management)"""
        session = self.Session()
        try:
            return session.query(model_class).filter_by(id=id).first()
        finally:
            session.close()

# Simple usage
db = DatabaseFacade("postgresql://...")
db.save(user)
user = db.find_by_id(User, "123")
```

### Design Patterns - Behavioral

#### Strategy Pattern

```python
"""
STRATEGY: Define family of algorithms, make them interchangeable
"""

from abc import ABC, abstractmethod

# Strategy interface
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass

# Concrete strategies
class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number: str, cvv: str):
        self.card_number = card_number
        self.cvv = cvv
  
    def pay(self, amount: float) -> bool:
        print(f"Paying ${amount} with credit card {self.card_number}")
        # Process credit card payment
        return True

class PayPalPayment(PaymentStrategy):
    def __init__(self, email: str):
        self.email = email
  
    def pay(self, amount: float) -> bool:
        print(f"Paying ${amount} via PayPal ({self.email})")
        # Process PayPal payment
        return True

class CryptoPayment(PaymentStrategy):
    def __init__(self, wallet_address: str):
        self.wallet_address = wallet_address
  
    def pay(self, amount: float) -> bool:
        print(f"Paying ${amount} via crypto to {self.wallet_address}")
        # Process crypto payment
        return True

# Context
class ShoppingCart:
    def __init__(self):
        self.items = []
        self.payment_strategy = None
  
    def add_item(self, item, price):
        self.items.append((item, price))
  
    def set_payment_strategy(self, strategy: PaymentStrategy):
        self.payment_strategy = strategy
  
    def checkout(self) -> bool:
        total = sum(price for _, price in self.items)
        return self.payment_strategy.pay(total)

# Usage
cart = ShoppingCart()
cart.add_item("Book", 29.99)
cart.add_item("Laptop", 999.99)

# Choose payment strategy at runtime
cart.set_payment_strategy(CreditCardPayment("1234-5678", "123"))
cart.checkout()  # Pays with credit card

cart.set_payment_strategy(PayPalPayment("user@example.com"))
cart.checkout()  # Pays with PayPal

# ✅ PYTHONIC: Strategy with functions
def credit_card_payment(amount, card_number, cvv):
    print(f"Paying ${amount} with credit card")
    return True

def paypal_payment(amount, email):
    print(f"Paying ${amount} via PayPal")
    return True

class ShoppingCart:
    def __init__(self):
        self.items = []
  
    def checkout(self, payment_function, **kwargs):
        total = sum(price for _, price in self.items)
        return payment_function(total, **kwargs)

cart = ShoppingCart()
cart.checkout(credit_card_payment, card_number="1234", cvv="123")
cart.checkout(paypal_payment, email="user@example.com")
```

#### Observer Pattern

```python
"""
OBSERVER: Notify multiple objects of state changes
"""

from abc import ABC, abstractmethod
from typing import List

# Subject (Observable)
class Subject:
    def __init__(self):
        self._observers: List['Observer'] = []
        self._state = None
  
    def attach(self, observer: 'Observer'):
        self._observers.append(observer)
  
    def detach(self, observer: 'Observer'):
        self._observers.remove(observer)
  
    def notify(self):
        for observer in self._observers:
            observer.update(self)
  
    @property
    def state(self):
        return self._state
  
    @state.setter
    def state(self, value):
        self._state = value
        self.notify()  # Notify on state change

# Observer interface
class Observer(ABC):
    @abstractmethod
    def update(self, subject: Subject):
        pass

# Concrete observers
class EmailObserver(Observer):
    def update(self, subject: Subject):
        print(f"Email: State changed to {subject.state}")

class SMSObserver(Observer):
    def update(self, subject: Subject):
        print(f"SMS: State changed to {subject.state}")

class LogObserver(Observer):
    def update(self, subject: Subject):
        print(f"Log: State changed to {subject.state}")

# Usage
subject = Subject()

# Attach observers
subject.attach(EmailObserver())
subject.attach(SMSObserver())
subject.attach(LogObserver())

# Change state (all observers notified)
subject.state = "New Order"
# Email: State changed to New Order
# SMS: State changed to New Order
# Log: State changed to New Order

# ✅ PYTHONIC: Observer with events
from typing import Callable, Dict, List

class EventEmitter:
    def __init__(self):
        self._events: Dict[str, List[Callable]] = {}
  
    def on(self, event: str, callback: Callable):
        """Subscribe to event"""
        if event not in self._events:
            self._events[event] = []
        self._events[event].append(callback)
  
    def off(self, event: str, callback: Callable):
        """Unsubscribe from event"""
        if event in self._events:
            self._events[event].remove(callback)
  
    def emit(self, event: str, *args, **kwargs):
        """Emit event to all subscribers"""
        if event in self._events:
            for callback in self._events[event]:
                callback(*args, **kwargs)

# Usage
emitter = EventEmitter()

# Subscribe
def on_order_created(order_id):
    print(f"Email: New order {order_id}")

def on_order_created_sms(order_id):
    print(f"SMS: New order {order_id}")

emitter.on('order.created', on_order_created)
emitter.on('order.created', on_order_created_sms)

# Emit event
emitter.emit('order.created', order_id='12345')
# Email: New order 12345
# SMS: New order 12345
```

#### Command Pattern

```python
"""
COMMAND: Encapsulate request as object
"""

from abc import ABC, abstractmethod
from typing import List

# Command interface
class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
  
    @abstractmethod
    def undo(self):
        pass

# Receiver
class TextEditor:
    def __init__(self):
        self.text = ""
  
    def write(self, text: str):
        self.text += text
  
    def delete(self, count: int):
        self.text = self.text[:-count]

# Concrete commands
class WriteCommand(Command):
    def __init__(self, editor: TextEditor, text: str):
        self.editor = editor
        self.text = text
  
    def execute(self):
        self.editor.write(self.text)
  
    def undo(self):
        self.editor.delete(len(self.text))

class DeleteCommand(Command):
    def __init__(self, editor: TextEditor, count: int):
        self.editor = editor
        self.count = count
        self.deleted_text = None
  
    def execute(self):
        # Save for undo
        self.deleted_text = self.editor.text[-self.count:]
        self.editor.delete(self.count)
  
    def undo(self):
        self.editor.write(self.deleted_text)

# Invoker
class CommandHistory:
    def __init__(self):
        self.history: List[Command] = []
        self.position = -1
  
    def execute(self, command: Command):
        command.execute()
    
        # Remove commands after current position
        self.history = self.history[:self.position + 1]
    
        # Add new command
        self.history.append(command)
        self.position += 1
  
    def undo(self):
        if self.position >= 0:
            self.history[self.position].undo()
            self.position -= 1
  
    def redo(self):
        if self.position < len(self.history) - 1:
            self.position += 1
            self.history[self.position].execute()

# Usage
editor = TextEditor()
history = CommandHistory()

# Execute commands
history.execute(WriteCommand(editor, "Hello "))
history.execute(WriteCommand(editor, "World"))
print(editor.text)  # "Hello World"

# Undo
history.undo()
print(editor.text)  # "Hello "

# Redo
history.redo()
print(editor.text)  # "Hello World"

# More commands
history.execute(DeleteCommand(editor, 5))  # Delete "World"
print(editor.text)  # "Hello "

history.undo()  # Restore "World"
print(editor.text)  # "Hello World"
```

### Python-Specific Patterns

#### Context Manager Pattern

```python
"""
CONTEXT MANAGER: Resource management with with statement
"""

# ✅ GOOD: Using __enter__ and __exit__
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
  
    def __enter__(self):
        print("Opening database connection")
        self.connection = connect_to_database(self.connection_string)
        return self.connection
  
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing database connection")
        if self.connection:
            self.connection.close()
        # Return False to propagate exceptions
        return False

# Usage
with DatabaseConnection("postgresql://...") as conn:
    conn.execute("SELECT * FROM users")
# Connection automatically closed

# ✅ BETTER: Using contextlib
from contextlib import contextmanager

@contextmanager
def database_connection(connection_string):
    conn = connect_to_database(connection_string)
    try:
        yield conn
    finally:
        conn.close()

with database_connection("postgresql://...") as conn:
    conn.execute("SELECT * FROM users")

# ✅ REAL WORLD: Transaction management
@contextmanager
def transaction(session):
    """Database transaction context manager"""
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

with transaction(Session()) as session:
    user = User(name="Alice")
    session.add(user)
# Automatically commits on success, rolls back on error
```

#### Descriptor Pattern

```python
"""
DESCRIPTOR: Reusable property logic
"""

class Validated:
    """Descriptor for validated attributes"""
  
    def __init__(self, validator):
        self.validator = validator
        self.data = {}
  
    def __set_name__(self, owner, name):
        self.name = name
  
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.data.get(id(instance))
  
    def __set__(self, instance, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value for {self.name}: {value}")
        self.data[id(instance)] = value

class User:
    # Reusable validation
    email = Validated(lambda x: '@' in x)
    age = Validated(lambda x: 0 <= x <= 150)
  
    def __init__(self, email, age):
        self.email = email
        self.age = age

user = User("alice@example.com", 30)
# user.email = "invalid"  # ValueError!
# user.age = 200  # ValueError!
```

### SOLID Principles

```python
"""
SOLID PRINCIPLES

S - Single Responsibility Principle
O - Open/Closed Principle
L - Liskov Substitution Principle
I - Interface Segregation Principle
D - Dependency Inversion Principle
"""

# S - Single Responsibility Principle
# Each class should have one reason to change

# ❌ BAD: Multiple responsibilities
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
  
    def save(self):
        # Database logic (responsibility 1)
        pass
  
    def send_email(self):
        # Email logic (responsibility 2)
        pass
  
    def generate_report(self):
        # Reporting logic (responsibility 3)
        pass

# ✅ GOOD: Single responsibility
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user: User):
        # Database logic only
        pass

class EmailService:
    def send_email(self, user: User):
        # Email logic only
        pass

class ReportGenerator:
    def generate_report(self, user: User):
        # Reporting logic only
        pass

# O - Open/Closed Principle
# Open for extension, closed for modification

# ❌ BAD: Must modify class to add discount types
class DiscountCalculator:
    def calculate(self, order, discount_type):
        if discount_type == "percentage":
            return order.total * 0.1
        elif discount_type == "fixed":
            return 10
        elif discount_type == "seasonal":
            return order.total * 0.2
        # Must modify this method for new discount types!

# ✅ GOOD: Extend with new classes
class Discount(ABC):
    @abstractmethod
    def calculate(self, order) -> float:
        pass

class PercentageDiscount(Discount):
    def __init__(self, percentage):
        self.percentage = percentage
  
    def calculate(self, order) -> float:
        return order.total * self.percentage

class FixedDiscount(Discount):
    def __init__(self, amount):
        self.amount = amount
  
    def calculate(self, order) -> float:
        return self.amount

class SeasonalDiscount(Discount):
    def calculate(self, order) -> float:
        return order.total * 0.2

# Add new discounts without modifying existing code
class BirthdayDiscount(Discount):
    def calculate(self, order) -> float:
        return order.total * 0.15

# L - Liskov Substitution Principle
# Subtypes must be substitutable for base types

# ❌ BAD: Violates LSP
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
  
    def area(self):
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, size):
        super().__init__(size, size)
  
    def set_width(self, width):
        self.width = width
        self.height = width  # Breaks LSP!

# ✅ GOOD: Proper hierarchy
class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
  
    def area(self) -> float:
        return self.width * self.height

class Square(Shape):
    def __init__(self, size):
        self.size = size
  
    def area(self) -> float:
        return self.size * self.size

# I - Interface Segregation Principle
# Clients shouldn't depend on interfaces they don't use

# ❌ BAD: Fat interface
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass
  
    @abstractmethod
    def eat(self):
        pass

class Human(Worker):
    def work(self):
        print("Working")
  
    def eat(self):
        print("Eating")

class Robot(Worker):
    def work(self):
        print("Working")
  
    def eat(self):
        # Robots don't eat!
        raise NotImplementedError

# ✅ GOOD: Segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass

class Human(Workable, Eatable):
    def work(self):
        print("Working")
  
    def eat(self):
        print("Eating")

class Robot(Workable):
    def work(self):
        print("Working")

# D - Dependency Inversion Principle
# Depend on abstractions, not concretions

# ❌ BAD: High-level depends on low-level
class MySQLDatabase:
    def save(self, data):
        print("Saving to MySQL")

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # Depends on concrete class
  
    def create_user(self, user):
        self.db.save(user)

# ✅ GOOD: Both depend on abstraction
class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):
    def save(self, data):
        print("Saving to MySQL")

class PostgreSQLDatabase(Database):
    def save(self, data):
        print("Saving to PostgreSQL")

class UserService:
    def __init__(self, database: Database):
        self.db = database  # Depends on abstraction
  
    def create_user(self, user):
        self.db.save(user)

# Can swap implementations
service = UserService(MySQLDatabase())
service = UserService(PostgreSQLDatabase())
```

### Domain-Driven Design (DDD)

```python
"""
DOMAIN-DRIVEN DESIGN

Core concepts:
- Entities: Objects with identity
- Value Objects: Immutable objects defined by attributes
- Aggregates: Cluster of entities and value objects
- Repositories: Access aggregates
- Domain Events: Something happened in domain
"""

from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime
from uuid import uuid4

# Value Object (immutable)
@dataclass(frozen=True)
class Money:
    amount: float
    currency: str
  
    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("Amount cannot be negative")
  
    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)

@dataclass(frozen=True)
class Address:
    street: str
    city: str
    country: str
    postal_code: str

# Entity (has identity)
class Order:
    def __init__(self, customer_id: str):
        self.id = str(uuid4())  # Identity
        self.customer_id = customer_id
        self.items: List['OrderItem'] = []
        self.status = "pending"
        self.created_at = datetime.utcnow()
  
    def add_item(self, product_id: str, quantity: int, price: Money):
        """Business logic"""
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
    
        item = OrderItem(product_id, quantity, price)
        self.items.append(item)
  
    def total(self) -> Money:
        """Domain logic"""
        if not self.items:
            return Money(0, "USD")
    
        total = self.items[0].subtotal()
        for item in self.items[1:]:
            total = total.add(item.subtotal())
        return total
  
    def place(self):
        """Business rule"""
        if not self.items:
            raise ValueError("Cannot place empty order")
        if self.status != "pending":
            raise ValueError(f"Cannot place order in {self.status} status")
    
        self.status = "placed"
        # Raise domain event
        return OrderPlacedEvent(self.id, self.customer_id, self.total())

@dataclass
class OrderItem:
    product_id: str
    quantity: int
    unit_price: Money
  
    def subtotal(self) -> Money:
        return Money(
            self.unit_price.amount * self.quantity,
            self.unit_price.currency
        )

# Domain Event
@dataclass
class OrderPlacedEvent:
    order_id: str
    customer_id: str
    total: Money
    occurred_at: datetime = None
  
    def __post_init__(self):
        if self.occurred_at is None:
            self.occurred_at = datetime.utcnow()

# Aggregate Root
class Customer:
    """Aggregate root controlling orders"""
  
    def __init__(self, email: str, name: str):
        self.id = str(uuid4())
        self.email = email
        self.name = name
        self.orders: List[Order] = []
  
    def place_order(self, order: Order) -> OrderPlacedEvent:
        """Business logic at aggregate level"""
        if order.customer_id != self.id:
            raise ValueError("Order doesn't belong to this customer")
    
        event = order.place()
        self.orders.append(order)
        return event

# Repository (persistence abstraction)
class OrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> None:
        pass
  
    @abstractmethod
    def find_by_id(self, order_id: str) -> Optional[Order]:
        pass
  
    @abstractmethod
    def find_by_customer(self, customer_id: str) -> List[Order]:
        pass

# Concrete repository
class InMemoryOrderRepository(OrderRepository):
    def __init__(self):
        self._orders: dict[str, Order] = {}
  
    def save(self, order: Order) -> None:
        self._orders[order.id] = order
  
    def find_by_id(self, order_id: str) -> Optional[Order]:
        return self._orders.get(order_id)
  
    def find_by_customer(self, customer_id: str) -> List[Order]:
        return [
            order for order in self._orders.values()
            if order.customer_id == customer_id
        ]

# Domain Service (business logic that doesn't belong to entity)
class OrderService:
    def __init__(self, repository: OrderRepository):
        self.repository = repository
  
    def calculate_customer_lifetime_value(self, customer_id: str) -> Money:
        """Domain logic spanning multiple orders"""
        orders = self.repository.find_by_customer(customer_id)
    
        if not orders:
            return Money(0, "USD")
    
        total = orders[0].total()
        for order in orders[1:]:
            total = total.add(order.total())
    
        return total

# Usage
customer = Customer("alice@example.com", "Alice")
order = Order(customer.id)

# Add items
order.add_item("PROD-1", 2, Money(29.99, "USD"))
order.add_item("PROD-2", 1, Money(49.99, "USD"))

# Place order (domain logic)
event = customer.place_order(order)

# Save to repository
repository = InMemoryOrderRepository()
repository.save(order)

# Domain service
service = OrderService(repository)
ltv = service.calculate_customer_lifetime_value(customer.id)
print(f"Customer LTV: {ltv.amount} {ltv.currency}")
```

### Frequently Asked Questions

**Q1: When should I use microservices vs monolithic architecture?**

**A:**

```python
"""
DECISION MATRIX:

Use MONOLITH when:
- ✅ Small to medium application
- ✅ Team size < 10 developers
- ✅ MVP or prototype
- ✅ Simple business domain
- ✅ Need fast development
- ✅ Limited DevOps resources

Use MICROSERVICES when:
- ✅ Large application
- ✅ Multiple teams (10+ developers)
- ✅ Complex business domain with clear boundaries
- ✅ Need independent scaling
- ✅ Different tech stacks required
- ✅ Strong DevOps culture

MIGRATION PATH:
1. Start with monolith
2. Identify bounded contexts
3. Extract services incrementally
4. Use strangler fig pattern

Example: E-commerce evolution

Phase 1 (Month 1-6): Monolith
- Single application
- All features together
- Fast development

Phase 2 (Month 6-12): Modular monolith
- Clear module boundaries
- Separate databases per module
- Prepare for extraction

Phase 3 (Month 12-18): Hybrid
- Extract high-value services (payments, inventory)
- Keep rest as monolith
- Incremental migration

Phase 4 (Month 18-24): Microservices
- Most features as services
- Keep simple features in monolith
- Full microservices architecture
"""

# ✅ GOOD: Start simple, evolve as needed
# Don't build microservices from day 1 unless:
# - Team is experienced
# - Business justifies complexity
# - Infrastructure is ready
```

---

### Key Takeaways

**Software Architecture & Design Patterns:**

- **Architectural patterns**: Choose based on scale, team size, and complexity
- **Monolithic**: Simple, fast development, good for small/medium apps
- **Microservices**: Independent scaling, team autonomy, complex to operate
- **Event-driven**: Loose coupling, async processing, eventual consistency
- **CQRS**: Separate read/write models, optimized for each
- **Hexagonal**: Domain logic independent of infrastructure

**Design Patterns:**

- **Creational**: Control object creation (Singleton, Factory, Builder)
- **Structural**: Compose objects (Adapter, Decorator, Facade)
- **Behavioral**: Object communication (Strategy, Observer, Command)
- **Python-specific**: Context managers, descriptors

**SOLID Principles:**

- **Single Responsibility**: One reason to change
- **Open/Closed**: Open for extension, closed for modification
- **Liskov Substitution**: Subtypes must be substitutable
- **Interface Segregation**: Small, focused interfaces
- **Dependency Inversion**: Depend on abstractions

**Domain-Driven Design:**

- **Entities**: Objects with identity
- **Value Objects**: Immutable, defined by attributes
- **Aggregates**: Consistency boundaries
- **Repositories**: Persistence abstraction
- **Domain Events**: Capture business events

---

(Continuing with Section 3.3 - API Design in next file...)

## 3.3 API Design

**SDLC Phase:** System Design, Development

**Relevant For:**

- [ ] Requirements gathering
- [X] System design
- [X] Development
- [X] Testing (API testing)
- [X] Deployment (API versioning)
- [X] Maintenance (backward compatibility)

APIs are the contracts between systems. Good API design is crucial for maintainability, scalability, and developer experience.

### REST API Design

#### RESTful Principles

```python
"""
REST (Representational State Transfer)

Core Principles:
1. Resource-based (nouns, not verbs)
2. HTTP methods for operations
3. Stateless communication
4. Standard status codes
5. HATEOAS (optional)
"""

from fastapi import FastAPI, HTTPException, status, Query
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

app = FastAPI(title="E-Commerce API")

# Resource models
class Product(BaseModel):
    id: str
    name: str
    description: str
    price: float
    category: str
    in_stock: bool
    created_at: datetime

class ProductCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=200)
    description: str = Field(..., max_length=1000)
    price: float = Field(..., gt=0)
    category: str
    in_stock: bool = True

class ProductUpdate(BaseModel):
    name: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = None
    price: Optional[float] = Field(None, gt=0)
    category: Optional[str] = None
    in_stock: Optional[bool] = None

# ✅ GOOD: RESTful resource endpoints

# GET /products - List products (Collection)
@app.get("/products", response_model=List[Product])
async def list_products(
    category: Optional[str] = None,
    min_price: Optional[float] = None,
    max_price: Optional[float] = None,
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100)
):
    """
    List products with filtering and pagination
  
    Query Parameters:
    - category: Filter by category
    - min_price: Minimum price filter
    - max_price: Maximum price filter
    - page: Page number (default: 1)
    - page_size: Items per page (default: 20, max: 100)
    """
    # Filter and paginate
    products = await product_service.list_products(
        category=category,
        min_price=min_price,
        max_price=max_price,
        page=page,
        page_size=page_size
    )
    return products

# GET /products/{id} - Get single product (Item)
@app.get("/products/{product_id}", response_model=Product)
async def get_product(product_id: str):
    """Get product by ID"""
    product = await product_service.get_by_id(product_id)
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    return product

# POST /products - Create product (Collection)
@app.post(
    "/products",
    response_model=Product,
    status_code=status.HTTP_201_CREATED
)
async def create_product(product: ProductCreate):
    """Create new product"""
    created = await product_service.create(product)
    return created

# PUT /products/{id} - Replace product (Item)
@app.put("/products/{product_id}", response_model=Product)
async def replace_product(product_id: str, product: ProductCreate):
    """Replace entire product"""
    updated = await product_service.replace(product_id, product)
    if not updated:
        raise HTTPException(status_code=404, detail="Product not found")
    return updated

# PATCH /products/{id} - Partial update (Item)
@app.patch("/products/{product_id}", response_model=Product)
async def update_product(product_id: str, product: ProductUpdate):
    """Partially update product"""
    updated = await product_service.update(product_id, product)
    if not updated:
        raise HTTPException(status_code=404, detail="Product not found")
    return updated

# DELETE /products/{id} - Delete product (Item)
@app.delete("/products/{product_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_product(product_id: str):
    """Delete product"""
    deleted = await product_service.delete(product_id)
    if not deleted:
        raise HTTPException(status_code=404, detail="Product not found")
    return None

# ❌ BAD: Non-RESTful endpoints
# POST /getProducts - Wrong! Use GET /products
# POST /updateProduct/{id} - Wrong! Use PUT or PATCH
# GET /deleteProduct/{id} - Wrong! Use DELETE

# ✅ GOOD: Nested resources
@app.get("/products/{product_id}/reviews")
async def list_product_reviews(product_id: str):
    """List reviews for a product"""
    return await review_service.list_by_product(product_id)

@app.post("/products/{product_id}/reviews")
async def create_product_review(product_id: str, review: ReviewCreate):
    """Create review for a product"""
    return await review_service.create(product_id, review)

# ✅ GOOD: Actions on resources (non-CRUD operations)
@app.post("/orders/{order_id}/cancel")
async def cancel_order(order_id: str):
    """Cancel an order (custom action)"""
    return await order_service.cancel(order_id)

@app.post("/products/{product_id}/publish")
async def publish_product(product_id: str):
    """Publish a product (state transition)"""
    return await product_service.publish(product_id)
```

#### HTTP Status Codes

```python
"""
HTTP STATUS CODES

1xx - Informational
2xx - Success
3xx - Redirection
4xx - Client Error
5xx - Server Error
"""

from fastapi import HTTPException, status

# 2xx Success
# 200 OK - Successful GET, PUT, PATCH
# 201 Created - Successful POST (resource created)
# 204 No Content - Successful DELETE

@app.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate):
    """Returns 201 Created with Location header"""
    created = await user_service.create(user)
    return created

@app.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: str):
    """Returns 204 No Content (no body)"""
    await user_service.delete(user_id)
    return None

# 4xx Client Errors
# 400 Bad Request - Invalid input
# 401 Unauthorized - Missing/invalid authentication
# 403 Forbidden - Authenticated but not authorized
# 404 Not Found - Resource doesn't exist
# 409 Conflict - Resource conflict (e.g., duplicate email)
# 422 Unprocessable Entity - Validation error
# 429 Too Many Requests - Rate limit exceeded

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    user = await user_service.get_by_id(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    return user

@app.post("/users")
async def create_user(user: UserCreate):
    # Check if email already exists
    existing = await user_service.get_by_email(user.email)
    if existing:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Email already registered"
        )
  
    return await user_service.create(user)

# 5xx Server Errors
# 500 Internal Server Error - Unexpected error
# 502 Bad Gateway - Invalid response from upstream
# 503 Service Unavailable - Service temporarily down

@app.exception_handler(Exception)
async def global_exception_handler(request, exc):
    """Catch all unhandled exceptions"""
    logger.error(f"Unhandled exception: {exc}", exc_info=True)
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "error_code": "INTERNAL_ERROR",
            "message": "An unexpected error occurred"
        }
    )
```

#### API Versioning

```python
"""
API VERSIONING STRATEGIES

1. URL Versioning (Recommended)
2. Header Versioning
3. Query Parameter Versioning
"""

# Strategy 1: URL Versioning (Most common)
from fastapi import APIRouter

# Version 1 router
v1_router = APIRouter(prefix="/api/v1", tags=["v1"])

@v1_router.get("/users/{user_id}")
async def get_user_v1(user_id: str):
    """V1: Returns basic user info"""
    user = await user_service.get_by_id(user_id)
    return {
        "id": user.id,
        "name": user.name,
        "email": user.email
    }

# Version 2 router
v2_router = APIRouter(prefix="/api/v2", tags=["v2"])

@v2_router.get("/users/{user_id}")
async def get_user_v2(user_id: str):
    """V2: Returns enhanced user info with profile"""
    user = await user_service.get_by_id(user_id)
    return {
        "id": user.id,
        "name": user.name,
        "email": user.email,
        "profile": {
            "bio": user.bio,
            "avatar_url": user.avatar_url,
            "created_at": user.created_at
        }
    }

app.include_router(v1_router)
app.include_router(v2_router)

# Strategy 2: Header Versioning
from fastapi import Header

@app.get("/users/{user_id}")
async def get_user(
    user_id: str,
    api_version: str = Header(default="1", alias="X-API-Version")
):
    """Version specified in header"""
    if api_version == "1":
        return await get_user_v1_logic(user_id)
    elif api_version == "2":
        return await get_user_v2_logic(user_id)
    else:
        raise HTTPException(400, f"Unsupported API version: {api_version}")

# Strategy 3: Query Parameter (Not recommended)
@app.get("/users/{user_id}")
async def get_user(user_id: str, version: int = Query(1)):
    """Version in query parameter"""
    if version == 1:
        return await get_user_v1_logic(user_id)
    elif version == 2:
        return await get_user_v2_logic(user_id)

# ✅ GOOD: Deprecation strategy
@v1_router.get("/users/{user_id}")
async def get_user_v1(user_id: str, response: Response):
    """V1 endpoint (deprecated)"""
    # Add deprecation header
    response.headers["X-API-Deprecated"] = "true"
    response.headers["X-API-Sunset"] = "2025-12-31"
    response.headers["Link"] = '</api/v2/users/{user_id}>; rel="successor-version"'
  
    return await get_user_v1_logic(user_id)
```

#### Pagination, Filtering, Sorting

```python
"""
PAGINATION, FILTERING, SORTING PATTERNS
"""

from typing import Generic, TypeVar, List
from pydantic import BaseModel
from fastapi import Query

T = TypeVar('T')

# Pagination response model
class PaginatedResponse(BaseModel, Generic[T]):
    items: List[T]
    total: int
    page: int
    page_size: int
    total_pages: int

@app.get("/products", response_model=PaginatedResponse[Product])
async def list_products(
    # Filtering
    category: Optional[str] = Query(None, description="Filter by category"),
    min_price: Optional[float] = Query(None, ge=0),
    max_price: Optional[float] = Query(None, ge=0),
    in_stock: Optional[bool] = Query(None),
    search: Optional[str] = Query(None, description="Search in name/description"),
  
    # Sorting
    sort_by: str = Query("created_at", regex="^(name|price|created_at)$"),
    sort_order: str = Query("desc", regex="^(asc|desc)$"),
  
    # Pagination
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100)
):
    """
    List products with comprehensive filtering, sorting, and pagination
  
    Filtering:
    - category: Filter by category name
    - min_price/max_price: Price range filter
    - in_stock: Only in-stock products
    - search: Full-text search in name and description
  
    Sorting:
    - sort_by: Field to sort by (name, price, created_at)
    - sort_order: asc or desc
  
    Pagination:
    - page: Page number (1-indexed)
    - page_size: Items per page (max 100)
    """
  
    # Build filters
    filters = {}
    if category:
        filters['category'] = category
    if min_price is not None:
        filters['price__gte'] = min_price
    if max_price is not None:
        filters['price__lte'] = max_price
    if in_stock is not None:
        filters['in_stock'] = in_stock
    if search:
        filters['search'] = search
  
    # Query with filters, sorting, pagination
    result = await product_service.list(
        filters=filters,
        sort_by=sort_by,
        sort_order=sort_order,
        page=page,
        page_size=page_size
    )
  
    # Calculate total pages
    total_pages = (result.total + page_size - 1) // page_size
  
    return PaginatedResponse(
        items=result.items,
        total=result.total,
        page=page,
        page_size=page_size,
        total_pages=total_pages
    )

# ✅ GOOD: Cursor-based pagination (for large datasets)
@app.get("/products/feed")
async def products_feed(
    cursor: Optional[str] = None,
    limit: int = Query(20, ge=1, le=100)
):
    """
    Cursor-based pagination (efficient for large datasets)
  
    Benefits:
    - Consistent results (no duplicates/skips)
    - Efficient for real-time data
    - Works with infinite scroll
    """
    result = await product_service.list_with_cursor(
        cursor=cursor,
        limit=limit
    )
  
    return {
        "items": result.items,
        "next_cursor": result.next_cursor,
        "has_more": result.has_more
    }
```

### GraphQL API Design

```python
"""
GRAPHQL: Query language for APIs

Advantages:
- Client specifies exact data needed
- Single endpoint
- Strong typing
- No over-fetching/under-fetching

Disadvantages:
- More complex than REST
- Caching is harder
- Learning curve
"""

# Install: pip install strawberry-graphql
import strawberry
from typing import List, Optional
from datetime import datetime

# GraphQL Types
@strawberry.type
class User:
    id: str
    name: str
    email: str
    created_at: datetime

@strawberry.type
class Product:
    id: str
    name: str
    price: float
    category: str
  
    # Resolver for related data
    @strawberry.field
    async def reviews(self) -> List['Review']:
        """Fetch reviews for this product"""
        return await review_service.get_by_product(self.id)

@strawberry.type
class Review:
    id: str
    product_id: str
    user_id: str
    rating: int
    comment: str
  
    @strawberry.field
    async def user(self) -> User:
        """Fetch user who wrote review"""
        return await user_service.get_by_id(self.user_id)

# Queries
@strawberry.type
class Query:
    @strawberry.field
    async def user(self, id: str) -> Optional[User]:
        """Get user by ID"""
        return await user_service.get_by_id(id)
  
    @strawberry.field
    async def products(
        self,
        category: Optional[str] = None,
        limit: int = 20
    ) -> List[Product]:
        """List products with optional filtering"""
        return await product_service.list(
            category=category,
            limit=limit
        )
  
    @strawberry.field
    async def product(self, id: str) -> Optional[Product]:
        """Get product with reviews (resolved automatically)"""
        return await product_service.get_by_id(id)

# Mutations
@strawberry.type
class Mutation:
    @strawberry.mutation
    async def create_user(
        self,
        name: str,
        email: str
    ) -> User:
        """Create new user"""
        return await user_service.create(name, email)
  
    @strawberry.mutation
    async def create_review(
        self,
        product_id: str,
        user_id: str,
        rating: int,
        comment: str
    ) -> Review:
        """Create product review"""
        return await review_service.create(
            product_id, user_id, rating, comment
        )

# Create schema
schema = strawberry.Schema(query=Query, mutation=Mutation)

# GraphQL endpoint
from strawberry.fastapi import GraphQLRouter

graphql_app = GraphQLRouter(schema)
app.include_router(graphql_app, prefix="/graphql")

# Example GraphQL query:
"""
query GetProduct {
  product(id: "123") {
    id
    name
    price
    reviews {
      rating
      comment
      user {
        name
        email
      }
    }
  }
}
"""

# DataLoader pattern (solve N+1 problem)
from strawberry.dataloader import DataLoader

async def load_users(keys: List[str]) -> List[User]:
    """Batch load users (single database query)"""
    users = await user_service.get_by_ids(keys)
    # Return in same order as keys
    user_map = {user.id: user for user in users}
    return [user_map.get(key) for key in keys]

user_loader = DataLoader(load_fn=load_users)

@strawberry.type
class Review:
    id: str
    user_id: str
  
    @strawberry.field
    async def user(self, info) -> User:
        """Use DataLoader to batch user fetches"""
        return await info.context["user_loader"].load(self.user_id)
```

### API Best Practices

#### Authentication and Authorization

```python
"""
AUTHENTICATION & AUTHORIZATION
"""

from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from datetime import datetime, timedelta

# JWT Configuration
SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

security = HTTPBearer()

# Token models
class TokenData(BaseModel):
    user_id: str
    email: str
    role: str

class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int

# Create JWT token
def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
  
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# Verify JWT token
async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> TokenData:
    """Dependency: Extract and verify JWT token"""
    token = credentials.credentials
  
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("user_id")
        if user_id is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid authentication credentials"
            )
    
        return TokenData(
            user_id=user_id,
            email=payload.get("email"),
            role=payload.get("role")
        )
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )

# Role-based authorization
def require_role(required_role: str):
    """Dependency factory for role-based access"""
    async def check_role(
        current_user: TokenData = Depends(get_current_user)
    ) -> TokenData:
        if current_user.role != required_role:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Insufficient permissions"
            )
        return current_user
  
    return check_role

# Protected endpoints
@app.get("/users/me")
async def get_current_user_profile(
    current_user: TokenData = Depends(get_current_user)
):
    """Protected: Requires authentication"""
    return await user_service.get_by_id(current_user.user_id)

@app.post("/products")
async def create_product(
    product: ProductCreate,
    current_user: TokenData = Depends(require_role("admin"))
):
    """Protected: Requires admin role"""
    return await product_service.create(product)

# Login endpoint
@app.post("/auth/login", response_model=Token)
async def login(email: str, password: str):
    """Authenticate and return JWT token"""
    user = await user_service.authenticate(email, password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password"
        )
  
    # Create token
    access_token = create_access_token({
        "user_id": user.id,
        "email": user.email,
        "role": user.role
    })
  
    return Token(
        access_token=access_token,
        expires_in=ACCESS_TOKEN_EXPIRE_MINUTES * 60
    )
```

#### Rate Limiting

```python
"""
RATE LIMITING: Prevent API abuse
"""

from fastapi import Request
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

# Create limiter
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# Apply rate limits
@app.get("/products")
@limiter.limit("100/minute")  # 100 requests per minute per IP
async def list_products(request: Request):
    """Rate limited endpoint"""
    return await product_service.list()

@app.post("/auth/login")
@limiter.limit("5/minute")  # Stricter limit for login
async def login(request: Request, email: str, password: str):
    """Login with rate limiting (prevent brute force)"""
    return await authenticate(email, password)

# User-based rate limiting
async def get_user_id(request: Request) -> str:
    """Extract user ID from token"""
    token = request.headers.get("Authorization")
    # Extract user ID from token
    return user_id

@app.get("/api/search")
@limiter.limit("1000/hour", key_func=get_user_id)
async def search(request: Request, query: str):
    """Rate limit by authenticated user"""
    return await search_service.search(query)

# Redis-backed rate limiting (distributed systems)
from redis import Redis
from slowapi.util import get_remote_address

redis_client = Redis(host='localhost', port=6379)

limiter = Limiter(
    key_func=get_remote_address,
    storage_uri="redis://localhost:6379"
)
```

#### Caching Strategies

```python
"""
API CACHING STRATEGIES
"""

from fastapi import Response
from functools import wraps
import hashlib
import json

# 1. HTTP Caching (Cache-Control headers)
@app.get("/products/{product_id}")
async def get_product(product_id: str, response: Response):
    """Cache response in browser/CDN"""
    product = await product_service.get_by_id(product_id)
  
    # Cache for 5 minutes
    response.headers["Cache-Control"] = "public, max-age=300"
  
    # ETag for validation
    etag = hashlib.md5(json.dumps(product.dict()).encode()).hexdigest()
    response.headers["ETag"] = etag
  
    return product

# 2. Application-level caching
from functools import lru_cache

@lru_cache(maxsize=1000)
async def get_product_cached(product_id: str):
    """In-memory cache"""
    return await product_service.get_by_id(product_id)

# 3. Redis caching
import redis
import pickle

redis_client = redis.Redis(host='localhost', port=6379)

async def get_product_with_redis(product_id: str):
    """Redis cache"""
    cache_key = f"product:{product_id}"
  
    # Try cache first
    cached = redis_client.get(cache_key)
    if cached:
        return pickle.loads(cached)
  
    # Cache miss - fetch from database
    product = await product_service.get_by_id(product_id)
  
    # Store in cache (TTL: 5 minutes)
    redis_client.setex(
        cache_key,
        300,
        pickle.dumps(product)
    )
  
    return product

# 4. Cache invalidation
@app.put("/products/{product_id}")
async def update_product(product_id: str, product: ProductUpdate):
    """Invalidate cache on update"""
    updated = await product_service.update(product_id, product)
  
    # Invalidate cache
    cache_key = f"product:{product_id}"
    redis_client.delete(cache_key)
  
    return updated
```

### Web Frameworks

#### FastAPI (Modern, Async)

```python
"""
FASTAPI: Modern, fast, async-first framework
"""

from fastapi import FastAPI, Depends, HTTPException, BackgroundTasks
from pydantic import BaseModel, EmailStr
from typing import List

app = FastAPI(
    title="My API",
    description="API with FastAPI",
    version="1.0.0"
)

# Automatic request validation
class User(BaseModel):
    name: str
    email: EmailStr
    age: int

@app.post("/users")
async def create_user(user: User):
    """Automatic validation with Pydantic"""
    return {"message": "User created", "user": user}

# Dependency injection
async def get_db():
    db = Database()
    try:
        yield db
    finally:
        await db.close()

@app.get("/users/{user_id}")
async def get_user(user_id: str, db: Database = Depends(get_db)):
    """Dependency injection for database"""
    return await db.query(User).filter_by(id=user_id).first()

# Background tasks
@app.post("/send-email")
async def send_email(
    email: str,
    background_tasks: BackgroundTasks
):
    """Send email in background"""
    background_tasks.add_task(send_email_task, email)
    return {"message": "Email will be sent"}

async def send_email_task(email: str):
    # Send email asynchronously
    await email_service.send(email)

# WebSocket support
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Echo: {data}")

# Pros:
# ✅ Fast (async/await)
# ✅ Automatic documentation (OpenAPI)
# ✅ Type hints everywhere
# ✅ Dependency injection
# ✅ WebSocket support

# Cons:
# ❌ Younger ecosystem
# ❌ Less mature than Flask/Django
```

#### Flask (Micro-framework)

```python
"""
FLASK: Micro-framework (flexible, simple)
"""

from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

# Request validation (manual)
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
  
    # Manual validation
    if 'name' not in data or 'email' not in data:
        return jsonify({"error": "Missing required fields"}), 400
  
    # Create user
    user = user_service.create(data)
    return jsonify(user), 201

# Blueprints (modular apps)
from flask import Blueprint

api_v1 = Blueprint('api_v1', __name__, url_prefix='/api/v1')

@api_v1.route('/users/<user_id>')
def get_user(user_id):
    user = user_service.get_by_id(user_id)
    return jsonify(user)

app.register_blueprint(api_v1)

# Authentication decorator
def require_auth(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token or not verify_token(token):
            return jsonify({"error": "Unauthorized"}), 401
        return f(*args, **kwargs)
    return decorated_function

@app.route('/protected')
@require_auth
def protected_route():
    return jsonify({"message": "You are authenticated"})

# Pros:
# ✅ Simple, flexible
# ✅ Large ecosystem
# ✅ Great for small to medium APIs

# Cons:
# ❌ No async support (without extensions)
# ❌ Manual validation
# ❌ No automatic documentation
```

#### Django REST Framework

```python
"""
DJANGO REST FRAMEWORK: Full-featured, batteries-included
"""

from rest_framework import viewsets, serializers, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django.contrib.auth.models import User

# Serializers (validation + serialization)
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']
        read_only_fields = ['id']

class UserCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['username', 'email', 'password']
        extra_kwargs = {'password': {'write_only': True}}
  
    def create(self, validated_data):
        user = User.objects.create_user(**validated_data)
        return user

# ViewSets (CRUD operations)
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
  
    def get_serializer_class(self):
        if self.action == 'create':
            return UserCreateSerializer
        return UserSerializer
  
    # Custom action
    @action(detail=True, methods=['post'])
    def set_password(self, request, pk=None):
        user = self.get_object()
        password = request.data.get('password')
        user.set_password(password)
        user.save()
        return Response({'status': 'password set'})

# URL routing
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'users', UserViewSet)

urlpatterns = router.urls

# Pros:
# ✅ Full-featured (auth, admin, ORM)
# ✅ Batteries included
# ✅ Great for complex applications

# Cons:
# ❌ Synchronous (no async)
# ❌ Heavy (lots of boilerplate)
# ❌ Opinionated
```

### Frequently Asked Questions

**Q1: REST vs GraphQL - which should I choose?**

**A:**

```python
"""
REST vs GRAPHQL COMPARISON

Use REST when:
- ✅ Simple CRUD operations
- ✅ Public APIs (easier to cache)
- ✅ Team familiar with REST
- ✅ Need HTTP caching
- ✅ Multiple clients with similar needs

Use GraphQL when:
- ✅ Complex data requirements
- ✅ Multiple client types (web, mobile, etc.)
- ✅ Need flexible data fetching
- ✅ Avoiding over-fetching/under-fetching
- ✅ Rapid client-side iteration

Example Decision:

Scenario 1: Public blog API
→ REST (simple, cacheable, standard)

Scenario 2: Mobile app with complex data needs
→ GraphQL (flexible, single endpoint, mobile-optimized)

Scenario 3: Microservices internal communication
→ gRPC (fast, type-safe, binary)

Hybrid Approach:
- REST for public APIs
- GraphQL for mobile/web clients
- gRPC for internal services
"""

# Example: Combining REST and GraphQL
app = FastAPI()

# REST endpoints
@app.get("/api/products/{id}")
async def get_product(id: str):
    return await product_service.get(id)

# GraphQL endpoint
from strawberry.fastapi import GraphQLRouter
app.include_router(GraphQLRouter(schema), prefix="/graphql")
```

**Why This Matters:** Choosing the right API style affects development speed, performance, and client satisfaction.

---

**Q2: How do I version APIs without breaking existing clients?**

**A:**

```python
"""
API VERSIONING BEST PRACTICES

Strategy 1: URL Versioning (Recommended)
"""

# Version 1: Original API
@app.get("/api/v1/users/{id}")
async def get_user_v1(id: str):
    return {
        "id": id,
        "name": "John Doe",
        "email": "john@example.com"
    }

# Version 2: Enhanced API (backward incompatible change)
@app.get("/api/v2/users/{id}")
async def get_user_v2(id: str):
    return {
        "id": id,
        "full_name": "John Doe",  # Changed: name → full_name
        "email": "john@example.com",
        "profile": {  # New: nested profile
            "bio": "Software Engineer",
            "avatar_url": "https://..."
        }
    }

# Deprecation process:
"""
1. Announce deprecation (6-12 months notice)
   - Add X-API-Deprecated: true header
   - Add deprecation notice in docs
   - Email users

2. Run both versions in parallel
   - V1 maintenance mode (bug fixes only)
   - V2 active development

3. Monitor V1 usage
   - Track API calls
   - Identify users still on V1
   - Contact them directly

4. Sunset V1
   - Return 410 Gone status
   - Redirect to V2 docs
"""

@app.get("/api/v1/users/{id}")
async def get_user_v1_deprecated(id: str, response: Response):
    # Add deprecation headers
    response.headers["X-API-Deprecated"] = "true"
    response.headers["X-API-Sunset-Date"] = "2025-12-31"
    response.headers["Link"] = '</api/v2/users/{id}>; rel="successor-version"'
  
    # Still functional
    return get_user_v1_logic(id)

# After sunset date:
@app.get("/api/v1/users/{id}")
async def get_user_v1_sunsetted(id: str):
    raise HTTPException(
        status_code=410,  # Gone
        detail="API v1 has been sunset. Please use /api/v2/users/{id}"
    )

# ✅ GOOD: Backward compatible changes (no new version needed)
# - Adding optional fields
# - Adding new endpoints
# - Adding optional query parameters

# ❌ BREAKING: Requires new version
# - Removing fields
# - Renaming fields
# - Changing field types
# - Changing authentication
# - Changing response structure
```

**Why This Matters:** Poor versioning breaks client applications and damages developer trust.

---

### Key Takeaways

**API Design:**

- **REST**: Resource-based, HTTP methods, stateless, standard status codes
- **GraphQL**: Client-specified queries, single endpoint, strong typing
- **gRPC**: Binary protocol, type-safe, high performance
- **Versioning**: URL versioning (recommended), deprecation process
- **Pagination**: Offset/limit for simple, cursor for large datasets
- **Authentication**: JWT tokens, role-based authorization
- **Rate limiting**: Prevent abuse, per-user/per-IP limits
- **Caching**: HTTP headers, Redis, invalidation strategies

**Web Frameworks:**

- **FastAPI**: Modern, async, automatic docs, type hints
- **Flask**: Simple, flexible, large ecosystem
- **Django REST**: Full-featured, batteries-included, opinionated

**Best Practices:**

- Use proper HTTP status codes
- Validate input (Pydantic, marshmallow)
- Document with OpenAPI/Swagger
- Version APIs explicitly
- Implement rate limiting
- Cache responses appropriately
- Handle errors consistently
- Support filtering and pagination

---

## 3.4 Database Design & ORM

**SDLC Phase:** System Design, Development, Maintenance

**Relevant For:**

- [ ] Requirements gathering
- [X] System design (data modeling)
- [X] Development
- [X] Testing (data integrity)
- [ ] Deployment
- [X] Maintenance (migrations, optimization)

Database design is critical for data integrity, performance, and scalability.

### Relational Database Design

#### Schema Design

```python
"""
RELATIONAL DATABASE SCHEMA DESIGN

Key Concepts:
- Primary keys
- Foreign keys
- Relationships (1:1, 1:N, M:N)
- Normalization
- Indexes
"""

from sqlalchemy import (
    create_engine, Column, String, Integer, Float,
    DateTime, Boolean, ForeignKey, Table, Text,
    UniqueConstraint, Index, CheckConstraint
)
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker
from datetime import datetime
import uuid

Base = declarative_base()

# ✅ GOOD: Well-designed schema

# Users table
class User(Base):
    __tablename__ = 'users'
  
    # Primary key (UUID)
    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
  
    # Required fields
    email = Column(String(255), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    full_name = Column(String(255), nullable=False)
  
    # Optional fields
    phone = Column(String(20))
    bio = Column(Text)
  
    # Status fields
    is_active = Column(Boolean, default=True, nullable=False)
    is_verified = Column(Boolean, default=False, nullable=False)
  
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
  
    # Relationships
    orders = relationship("Order", back_populates="user")
    reviews = relationship("Review", back_populates="user")
  
    # Indexes
    __table_args__ = (
        Index('idx_email', 'email'),
        Index('idx_created_at', 'created_at'),
    )

# Products table
class Product(Base):
    __tablename__ = 'products'
  
    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
    name = Column(String(255), nullable=False, index=True)
    description = Column(Text)
    price = Column(Float, nullable=False)
  
    # Foreign key
    category_id = Column(String(36), ForeignKey('categories.id'), nullable=False)
  
    # Stock tracking
    stock_quantity = Column(Integer, default=0, nullable=False)
  
    # Status
    is_active = Column(Boolean, default=True, nullable=False)
  
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
  
    # Relationships
    category = relationship("Category", back_populates="products")
    reviews = relationship("Review", back_populates="product")
    order_items = relationship("OrderItem", back_populates="product")
  
    # Constraints
    __table_args__ = (
        CheckConstraint('price >= 0', name='check_positive_price'),
        CheckConstraint('stock_quantity >= 0', name='check_positive_stock'),
        Index('idx_category_id', 'category_id'),
        Index('idx_name', 'name'),
    )

# Categories table (hierarchical)
class Category(Base):
    __tablename__ = 'categories'
  
    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
    name = Column(String(100), nullable=False, unique=True)
    description = Column(Text)
  
    # Self-referential foreign key (hierarchical categories)
    parent_id = Column(String(36), ForeignKey('categories.id'))
  
    # Relationships
    parent = relationship("Category", remote_side=[id], backref="children")
    products = relationship("Product", back_populates="category")

# Orders table
class Order(Base):
    __tablename__ = 'orders'
  
    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
  
    # Foreign key
    user_id = Column(String(36), ForeignKey('users.id'), nullable=False)
  
    # Order details
    total_amount = Column(Float, nullable=False)
    status = Column(String(20), default='pending', nullable=False)
  
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
  
    # Relationships
    user = relationship("User", back_populates="orders")
    items = relationship("OrderItem", back_populates="order", cascade="all, delete-orphan")
  
    __table_args__ = (
        Index('idx_user_id', 'user_id'),
        Index('idx_status', 'status'),
        Index('idx_created_at', 'created_at'),
    )

# Order items (many-to-many junction table)
class OrderItem(Base):
    __tablename__ = 'order_items'
  
    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
  
    # Foreign keys
    order_id = Column(String(36), ForeignKey('orders.id'), nullable=False)
    product_id = Column(String(36), ForeignKey('products.id'), nullable=False)
  
    # Item details
    quantity = Column(Integer, nullable=False)
    unit_price = Column(Float, nullable=False)
  
    # Relationships
    order = relationship("Order", back_populates="items")
    product = relationship("Product", back_populates="order_items")
  
    __table_args__ = (
        Index('idx_order_id', 'order_id'),
        Index('idx_product_id', 'product_id'),
        CheckConstraint('quantity > 0', name='check_positive_quantity'),
        CheckConstraint('unit_price >= 0', name='check_positive_price'),
    )

# Reviews table
class Review(Base):
    __tablename__ = 'reviews'
  
    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
  
    # Foreign keys
    product_id = Column(String(36), ForeignKey('products.id'), nullable=False)
    user_id = Column(String(36), ForeignKey('users.id'), nullable=False)
  
    # Review data
    rating = Column(Integer, nullable=False)
    comment = Column(Text)
  
    # Timestamps
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
  
    # Relationships
    product = relationship("Product", back_populates="reviews")
    user = relationship("User", back_populates="reviews")
  
    __table_args__ = (
        # Unique constraint: user can review product once
        UniqueConstraint('product_id', 'user_id', name='unique_product_user_review'),
        CheckConstraint('rating >= 1 AND rating <= 5', name='check_rating_range'),
        Index('idx_product_id', 'product_id'),
        Index('idx_user_id', 'user_id'),
    )
```

#### Normalization

```python
"""
DATABASE NORMALIZATION

1NF (First Normal Form):
- Atomic values (no arrays/lists in cells)
- Each column has unique name
- Each row is unique

2NF (Second Normal Form):
- Must be in 1NF
- No partial dependencies (all non-key attributes depend on entire primary key)

3NF (Third Normal Form):
- Must be in 2NF
- No transitive dependencies (non-key attributes don't depend on other non-key attributes)
"""

# ❌ BAD: Denormalized (violates 1NF)
class OrderBad(Base):
    __tablename__ = 'orders_bad'
  
    id = Column(String(36), primary_key=True)
    user_id = Column(String(36))
  
    # Violates 1NF: non-atomic values
    product_ids = Column(String)  # "PROD-1,PROD-2,PROD-3"
    quantities = Column(String)   # "2,1,5"
    prices = Column(String)       # "29.99,49.99,19.99"

# ✅ GOOD: Normalized (3NF)
# Orders table (1 row = 1 order)
class Order(Base):
    __tablename__ = 'orders'
    id = Column(String(36), primary_key=True)
    user_id = Column(String(36), ForeignKey('users.id'))
    total_amount = Column(Float)

# Order items table (1 row = 1 product in order)
class OrderItem(Base):
    __tablename__ = 'order_items'
    id = Column(String(36), primary_key=True)
    order_id = Column(String(36), ForeignKey('orders.id'))
    product_id = Column(String(36), ForeignKey('products.id'))
    quantity = Column(Integer)
    unit_price = Column(Float)

# When to denormalize:
# - Read-heavy workloads (optimize reads)
# - Reporting/analytics queries
# - Caching frequently accessed data

# Example: Denormalized for performance
class OrderWithCache(Base):
    __tablename__ = 'orders_with_cache'
  
    id = Column(String(36), primary_key=True)
    user_id = Column(String(36))
  
    # Denormalized: cache user name (avoid JOIN)
    user_name = Column(String(255))  # Copied from users table
    user_email = Column(String(255))  # Copied from users table
  
    # Trade-off: Faster reads, but must update when user changes
```

### SQLAlchemy

#### Core vs ORM

```python
"""
SQLALCHEMY: Python SQL toolkit

SQLAlchemy Core: SQL expression language (low-level)
SQLAlchemy ORM: Object-relational mapping (high-level)
"""

from sqlalchemy import create_engine, select, insert, update, delete
from sqlalchemy.orm import Session

# Database connection
engine = create_engine('postgresql://user:password@localhost/dbname')

# --- SQLAlchemy Core (SQL expressions) ---

# SELECT (Core)
stmt = select(User).where(User.email == 'alice@example.com')
with engine.connect() as conn:
    result = conn.execute(stmt)
    user = result.fetchone()

# INSERT (Core)
stmt = insert(User).values(
    id=str(uuid.uuid4()),
    email='bob@example.com',
    full_name='Bob Smith'
)
with engine.connect() as conn:
    conn.execute(stmt)
    conn.commit()

# UPDATE (Core)
stmt = update(User).where(User.id == user_id).values(full_name='New Name')
with engine.connect() as conn:
    conn.execute(stmt)
    conn.commit()

# DELETE (Core)
stmt = delete(User).where(User.id == user_id)
with engine.connect() as conn:
    conn.execute(stmt)
    conn.commit()

# --- SQLAlchemy ORM (Object mapping) ---

# Create session
SessionLocal = sessionmaker(bind=engine)

# SELECT (ORM)
with SessionLocal() as session:
    user = session.query(User).filter_by(email='alice@example.com').first()

# INSERT (ORM)
with SessionLocal() as session:
    user = User(
        email='bob@example.com',
        full_name='Bob Smith'
    )
    session.add(user)
    session.commit()

# UPDATE (ORM)
with SessionLocal() as session:
    user = session.query(User).filter_by(id=user_id).first()
    user.full_name = 'New Name'
    session.commit()

# DELETE (ORM)
with SessionLocal() as session:
    user = session.query(User).filter_by(id=user_id).first()
    session.delete(user)
    session.commit()

# When to use each:
# Core: Complex queries, bulk operations, performance-critical
# ORM: CRUD operations, business logic, type safety
```

#### Relationships and Joins

```python
"""
SQLALCHEMY RELATIONSHIPS
"""

# One-to-Many
class User(Base):
    __tablename__ = 'users'
    id = Column(String(36), primary_key=True)
    email = Column(String(255))
  
    # One user has many orders
    orders = relationship("Order", back_populates="user")

class Order(Base):
    __tablename__ = 'orders'
    id = Column(String(36), primary_key=True)
    user_id = Column(String(36), ForeignKey('users.id'))
  
    # Many orders belong to one user
    user = relationship("User", back_populates="orders")

# Query with relationship
session = Session(engine)

# Get user and all orders (generates JOIN)
user = session.query(User).filter_by(id=user_id).first()
for order in user.orders:  # Loads orders automatically
    print(order.total_amount)

# Many-to-Many
# Association table
product_tags = Table(
    'product_tags',
    Base.metadata,
    Column('product_id', String(36), ForeignKey('products.id')),
    Column('tag_id', String(36), ForeignKey('tags.id'))
)

class Product(Base):
    __tablename__ = 'products'
    id = Column(String(36), primary_key=True)
    name = Column(String(255))
  
    # Many products have many tags
    tags = relationship("Tag", secondary=product_tags, back_populates="products")

class Tag(Base):
    __tablename__ = 'tags'
    id = Column(String(36), primary_key=True)
    name = Column(String(100))
  
    # Many tags have many products
    products = relationship("Product", secondary=product_tags, back_populates="tags")

# Query many-to-many
product = session.query(Product).filter_by(id=product_id).first()
for tag in product.tags:
    print(tag.name)

# Lazy vs Eager Loading

# ❌ BAD: N+1 problem (lazy loading)
users = session.query(User).all()
for user in users:  # 1 query
    for order in user.orders:  # N queries (1 per user)!
        print(order.total_amount)
# Total: 1 + N queries

# ✅ GOOD: Eager loading (joinedload)
from sqlalchemy.orm import joinedload

users = session.query(User).options(joinedload(User.orders)).all()
for user in users:  # 1 query with JOIN
    for order in user.orders:  # No additional queries!
        print(order.total_amount)
# Total: 1 query

# ✅ GOOD: Subquery loading (subqueryload)
from sqlalchemy.orm import subqueryload

users = session.query(User).options(subqueryload(User.orders)).all()
# Generates 2 queries: 1 for users, 1 for all orders

# ✅ GOOD: Select-in loading (selectinload) - Most efficient
from sqlalchemy.orm import selectinload

users = session.query(User).options(selectinload(User.orders)).all()
# Generates 2 queries with WHERE IN clause
```

#### Query Optimization

```python
"""
SQLALCHEMY QUERY OPTIMIZATION
"""

from sqlalchemy import func, distinct, and_, or_
from sqlalchemy.orm import Session

session = Session(engine)

# 1. Use indexes
# Define in model:
# Index('idx_email', 'email')
# Index('idx_user_id_created_at', 'user_id', 'created_at')

# 2. Select only needed columns
# ❌ BAD: Select all columns
users = session.query(User).all()

# ✅ GOOD: Select specific columns
users = session.query(User.id, User.email).all()

# 3. Pagination
# ✅ GOOD: Limit results
users = session.query(User).limit(20).offset(0).all()

# 4. Aggregation
# Count
user_count = session.query(func.count(User.id)).scalar()

# Group by
from sqlalchemy import func
order_totals = (
    session.query(User.id, func.sum(Order.total_amount))
    .join(Order)
    .group_by(User.id)
    .all()
)

# 5. Complex filters
# Multiple conditions
users = (
    session.query(User)
    .filter(
        and_(
            User.is_active == True,
            User.created_at > datetime(2024, 1, 1),
            or_(
                User.email.like('%@gmail.com'),
                User.email.like('%@yahoo.com')
            )
        )
    )
    .all()
)

# 6. Raw SQL (when needed)
result = session.execute(
    """
    SELECT u.*, COUNT(o.id) as order_count
    FROM users u
    LEFT JOIN orders o ON o.user_id = u.id
    GROUP BY u.id
    HAVING COUNT(o.id) > 5
    """
)

# 7. Bulk operations
# Bulk insert (fast)
session.bulk_insert_mappings(
    User,
    [
        {'email': 'user1@example.com', 'full_name': 'User 1'},
        {'email': 'user2@example.com', 'full_name': 'User 2'},
        # ... thousands more
    ]
)
session.commit()

# Bulk update (fast)
session.bulk_update_mappings(
    User,
    [
        {'id': '123', 'full_name': 'Updated Name 1'},
        {'id': '456', 'full_name': 'Updated Name 2'},
    ]
)
session.commit()
```

#### Database Migrations with Alembic

```python
"""
ALEMBIC: Database migration tool for SQLAlchemy
"""

# Install: pip install alembic

# Initialize Alembic
# $ alembic init migrations

# alembic.ini configuration:
"""
[alembic]
script_location = migrations
sqlalchemy.url = postgresql://user:password@localhost/dbname
"""

# Create migration
# $ alembic revision --autogenerate -m "Create users table"

# Generated migration file:
"""
def upgrade():
    op.create_table(
        'users',
        sa.Column('id', sa.String(36), primary_key=True),
        sa.Column('email', sa.String(255), nullable=False),
        sa.Column('full_name', sa.String(255), nullable=False),
        sa.Column('created_at', sa.DateTime(), nullable=False),
    )
    op.create_index('idx_email', 'users', ['email'])

def downgrade():
    op.drop_index('idx_email', 'users')
    op.drop_table('users')
"""

# Run migration
# $ alembic upgrade head

# Create another migration
# $ alembic revision -m "Add phone to users"

"""
def upgrade():
    op.add_column('users', sa.Column('phone', sa.String(20)))

def downgrade():
    op.drop_column('users', 'phone')
"""

# Rollback migration
# $ alembic downgrade -1  # Go back 1 version
# $ alembic downgrade base  # Rollback all

# Migration best practices:
# ✅ Test migrations in staging first
# ✅ Write both upgrade and downgrade
# ✅ Keep migrations small and focused
# ✅ Never edit applied migrations
# ✅ Backup database before migrating production
```

### Django ORM

```python
"""
DJANGO ORM: Object-relational mapper built into Django
"""

from django.db import models

# Define models
class User(models.Model):
    email = models.EmailField(unique=True, db_index=True)
    full_name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
  
    class Meta:
        db_table = 'users'
        indexes = [
            models.Index(fields=['email']),
            models.Index(fields=['created_at']),
        ]
  
    def __str__(self):
        return self.email

class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='orders')
    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    status = models.CharField(max_length=20, default='pending')
    created_at = models.DateTimeField(auto_now_add=True)
  
    class Meta:
        db_table = 'orders'
        ordering = ['-created_at']

# QuerySets (lazy evaluation)
# ❌ NOT executed yet
users = User.objects.filter(is_active=True)

# ✅ Executed when accessed
for user in users:  # Now executes SQL
    print(user.email)

# Common queries
# Get single object
user = User.objects.get(email='alice@example.com')

# Filter
users = User.objects.filter(is_active=True, created_at__gte=date(2024, 1, 1))

# Exclude
users = User.objects.exclude(email__endswith='@spam.com')

# Order
users = User.objects.order_by('-created_at')  # Descending

# Limit
users = User.objects.all()[:10]  # First 10

# Count
count = User.objects.filter(is_active=True).count()

# Aggregation
from django.db.models import Count, Sum, Avg

# Count orders per user
users = User.objects.annotate(order_count=Count('orders'))

# Sum order totals
total = Order.objects.aggregate(total=Sum('total_amount'))

# N+1 Problem Solution
# ❌ BAD: N+1 queries
users = User.objects.all()
for user in users:  # 1 query
    for order in user.orders.all():  # N queries
        print(order.total_amount)

# ✅ GOOD: select_related (1:1, ForeignKey)
orders = Order.objects.select_related('user').all()
for order in orders:  # 1 query with JOIN
    print(order.user.email)  # No additional query

# ✅ GOOD: prefetch_related (M:N, reverse FK)
users = User.objects.prefetch_related('orders').all()
for user in users:  # 2 queries total
    for order in user.orders.all():  # No additional queries
        print(order.total_amount)

# Migrations
# Create migration: python manage.py makemigrations
# Apply migration: python manage.py migrate
```

### Frequently Asked Questions

**Q1: How do I handle database migrations in production?**

**A:**

```python
"""
PRODUCTION MIGRATION STRATEGY

Best Practices:
1. Test in staging first
2. Backup database before migrating
3. Run migrations during low-traffic periods
4. Use backwards-compatible changes
5. Monitor during and after migration
"""

# Step-by-Step Process:

# 1. Create migration
# $ alembic revision --autogenerate -m "Add user_preferences"

# 2. Test in development
# $ alembic upgrade head
# Run tests to verify

# 3. Test in staging
# Deploy to staging
# Run migration
# $ alembic upgrade head
# Verify application works

# 4. Backup production database
# $ pg_dump -U user dbname > backup_$(date +%Y%m%d).sql

# 5. Run migration in production
# Option A: Zero-downtime migration (preferred)
"""
For breaking changes, use multi-phase migration:

Phase 1: Add new column (backward compatible)
- Deploy code that writes to both old and new columns
- Run migration to add new column
- Backfill data

Phase 2: Switch to new column
- Deploy code that reads from new column
- Verify everything works

Phase 3: Remove old column
- Deploy code that stops using old column
- Run migration to drop old column
"""

# Example: Rename column with zero downtime

# Phase 1 migration: Add new column
"""
def upgrade():
    op.add_column('users', sa.Column('full_name', sa.String(255)))
    # Backfill data
    op.execute("UPDATE users SET full_name = name")
"""

# Phase 1 code: Write to both columns
def create_user(name):
    user = User(name=name, full_name=name)  # Write both
    session.add(user)
    session.commit()

# Phase 2: Switch to new column
def get_user_name(user):
    return user.full_name  # Read from new column

# Phase 3 migration: Remove old column
"""
def upgrade():
    op.drop_column('users', 'name')
"""

# Option B: Maintenance window (simpler but requires downtime)
# 1. Put application in maintenance mode
# 2. Run migration: alembic upgrade head
# 3. Deploy new code
# 4. Remove maintenance mode

# 6. Monitor after migration
# - Check application logs for errors
# - Monitor database performance
# - Check query execution times
# - Verify data integrity

# 7. Rollback plan
# Keep previous deployment ready
# Test rollback migration in staging
# $ alembic downgrade -1
```

**Why This Matters:** Poor migration strategies cause downtime, data loss, and production incidents.

---

### Key Takeaways

**Database Design & ORM:**

- **Schema design**: Primary keys, foreign keys, relationships, constraints
- **Normalization**: 1NF → 2NF → 3NF (eliminate redundancy)
- **Denormalization**: Trade-off for read performance
- **Indexes**: Speed up queries, especially on foreign keys
- **SQLAlchemy Core**: SQL expressions (low-level)
- **SQLAlchemy ORM**: Object mapping (high-level)
- **N+1 problem**: Use eager loading (joinedload, selectinload)
- **Migrations**: Alembic for schema versioning
- **Query optimization**: Select specific columns, use indexes, paginate
- **Django ORM**: select_related (joins), prefetch_related (separate queries)

**Best Practices:**

- Use UUID primary keys for distributed systems
- Add indexes on foreign keys and frequently queried columns
- Use constraints (unique, check) for data integrity
- Eager load relationships to avoid N+1 queries
- Test migrations in staging before production
- Backup before running production migrations
- Use connection pooling for better performance

---

## Part 3 Complete!

Congratulations! You've completed **Part 3: SDLC - Requirements, Design & Architecture**. You now have comprehensive knowledge of:

✅ **Section 3.1**: Requirements & Planning Phase
✅ **Section 3.2**: Software Architecture & Design Patterns
✅ **Section 3.3**: API Design
✅ **Section 3.4**: Database Design & ORM

**What You've Learned:**

- Requirements gathering (user stories, TRDs, API specs)
- Technical design documents (15-section template)
- Architectural patterns (monolithic, microservices, event-driven, CQRS, hexagonal)
- Design patterns (creational, structural, behavioral, Python-specific)
- SOLID principles with real-world examples
- Domain-Driven Design (entities, value objects, aggregates)
- REST, GraphQL, and gRPC API design
- Authentication, authorization, rate limiting, caching
- Database schema design and normalization
- SQLAlchemy and Django ORM
- Database migrations and query optimization

**Next Steps:**

Continue with **Part 4: Testing & Quality Assurance** to learn:

- Testing strategies (unit, integration, functional, performance)
- Test-driven development (TDD)
- Pytest and testing frameworks
- Code coverage and quality metrics
- CI/CD pipelines
- Security testing

Keep building production-grade systems! 🚀
