---
title: "Complete Python Mastery Part 4: Testing & Quality Assurance"
date: 2024-12-05 00:00:00 +0530
categories: [Python, Mastery]
tags: [Python, Testing, Quality Assurance, TDD, Pytest, Unittest, Code-quality, Coverage]
---
## Table of Contents

- [Introduction](#introduction)
- [4.1 Testing Fundamentals](#41-testing-fundamentals)
  - [Testing Pyramid](#testing-pyramid)
  - [Test-Driven Development (TDD)](#test-driven-development-tdd)
  - [Testing Principles](#testing-principles)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [4.2 Unit Testing](#42-unit-testing)
  - [unittest Framework](#unittest-framework)
  - [pytest Framework](#pytest-framework)
  - [Mocking](#mocking)
  - [Test Doubles](#test-doubles)
  - [Code Coverage](#code-coverage)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
  - [Interview Questions](#interview-questions-1)
- [4.3 Integration Testing](#43-integration-testing)
  - [Database Testing](#database-testing)
  - [API Testing](#api-testing)
  - [External Service Testing](#external-service-testing)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
  - [Interview Questions](#interview-questions-2)
- [4.4 Advanced Testing](#44-advanced-testing)
  - [Property-Based Testing](#property-based-testing)
  - [Performance Testing](#performance-testing)
  - [Security Testing](#security-testing)
  - [Snapshot Testing](#snapshot-testing)
  - [Frequently Asked Questions](#frequently-asked-questions-3)
  - [Interview Questions](#interview-questions-3)
- [4.5 Code Quality &amp; Standards](#45-code-quality--standards)
  - [Code Formatting](#code-formatting)
  - [Linting](#linting)
  - [Type Checking](#type-checking)
  - [Static Analysis](#static-analysis)
  - [Frequently Asked Questions](#frequently-asked-questions-4)

---

# Complete Python Mastery Part 4: Testing & Quality Assurance

## Introduction

Welcome to **Part 4** of the Complete Python Mastery series. Testing and quality assurance are critical for maintaining reliable, maintainable software.

**What You'll Learn in Part 4:**

This post covers comprehensive testing strategies and code quality practices:

- **Testing fundamentals**: Testing pyramid, TDD, testing principles
- **Unit testing**: unittest, pytest, mocking, test doubles
- **Integration testing**: Database, API, and external service testing
- **Advanced testing**: Property-based, performance, security testing
- **Code quality**: Formatting, linting, type checking, static analysis
- **Best practices**: Test organization, coverage, continuous testing

**Why This Matters:**

Quality testing prevents:

- Production bugs and downtime
- Costly hotfixes and rollbacks
- Technical debt accumulation
- Customer dissatisfaction
- Security vulnerabilities

**Prerequisites:**

- Part 1: Python Fundamentals
- Part 2: Python Internals
- Part 3: SDLC & Architecture (recommended)
- Understanding of software development basics

Let's build bulletproof Python applications! 🛡️

---

## 4.1 Testing Fundamentals

**SDLC Phase:** Testing, Development

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (TDD)
- [X] Testing
- [ ] Deployment
- [X] Maintenance (regression testing)

### Testing Pyramid

```python
"""
TESTING PYRAMID

Structure (from bottom to top):
1. Unit Tests (70%) - Fast, isolated, numerous
2. Integration Tests (20%) - Medium speed, moderate isolation
3. E2E/UI Tests (10%) - Slow, full system, few

Why Pyramid Shape?
- More unit tests = faster feedback
- Fewer E2E tests = maintainable test suite
- Balance speed, coverage, and confidence
"""

# Example: E-commerce testing strategy

# Layer 1: Unit Tests (70% of tests)
# Fast, isolated, test individual functions/classes

def calculate_discount(price: float, discount_percent: float) -> float:
    """Calculate discounted price"""
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be between 0 and 100")
    return price * (1 - discount_percent / 100)

# Unit test
def test_calculate_discount():
    assert calculate_discount(100, 10) == 90.0
    assert calculate_discount(100, 0) == 100.0
    assert calculate_discount(100, 100) == 0.0

def test_calculate_discount_invalid():
    with pytest.raises(ValueError):
        calculate_discount(100, -10)
    with pytest.raises(ValueError):
        calculate_discount(100, 150)

# Layer 2: Integration Tests (20% of tests)
# Test components working together

class OrderService:
    def __init__(self, db, payment_gateway):
        self.db = db
        self.payment_gateway = payment_gateway
  
    def create_order(self, user_id, items):
        # Create order in database
        order = self.db.create_order(user_id, items)
      
        # Process payment
        payment = self.payment_gateway.charge(order.total)
      
        return order

# Integration test (with real database, mocked payment)
@pytest.fixture
def db():
    """Test database fixture"""
    db = Database('test.db')
    db.create_tables()
    yield db
    db.drop_tables()

def test_create_order_integration(db):
    payment_gateway = Mock()
    payment_gateway.charge.return_value = {'status': 'success'}
  
    service = OrderService(db, payment_gateway)
    order = service.create_order('user123', [{'product': 'ABC', 'qty': 2}])
  
    # Verify order created in database
    assert db.get_order(order.id) is not None
    # Verify payment called
    payment_gateway.charge.assert_called_once()

# Layer 3: E2E Tests (10% of tests)
# Test entire system from user perspective

def test_checkout_flow_e2e():
    """Test complete checkout flow"""
    client = TestClient(app)
  
    # 1. Login
    response = client.post('/auth/login', json={
        'email': 'test@example.com',
        'password': 'password123'
    })
    token = response.json()['access_token']
  
    # 2. Add items to cart
    client.post('/cart', json={'product_id': 'ABC', 'quantity': 2},
                headers={'Authorization': f'Bearer {token}'})
  
    # 3. Checkout
    response = client.post('/checkout',
                          headers={'Authorization': f'Bearer {token}'})
  
    assert response.status_code == 200
    order_id = response.json()['order_id']
  
    # 4. Verify order
    response = client.get(f'/orders/{order_id}',
                         headers={'Authorization': f'Bearer {token}'})
    assert response.status_code == 200

# Anti-patterns:
# ❌ Inverted pyramid (too many E2E tests)
# ❌ Ice cream cone (many E2E, few unit tests)
# ❌ Hourglass (many unit + E2E, few integration)

# ✅ GOOD: Follow pyramid
"""
Unit Tests:       ████████████████████ (1000+ tests, <1 second)
Integration:      ████████ (100+ tests, <10 seconds)
E2E:              ██ (10+ tests, <1 minute)
"""
```

### Test-Driven Development (TDD)

```python
"""
TEST-DRIVEN DEVELOPMENT (TDD)

Red-Green-Refactor Cycle:
1. RED: Write failing test
2. GREEN: Write minimal code to pass
3. REFACTOR: Improve code without breaking tests
"""

# Example: Implementing a shopping cart with TDD

# STEP 1: RED - Write failing test
import pytest

def test_empty_cart_has_zero_items():
    cart = ShoppingCart()
    assert cart.item_count() == 0

# Run test → FAILS (ShoppingCart doesn't exist)

# STEP 2: GREEN - Minimal implementation
class ShoppingCart:
    def __init__(self):
        self.items = []
  
    def item_count(self):
        return 0  # Simplest implementation

# Run test → PASSES

# STEP 3: REFACTOR - Improve implementation
class ShoppingCart:
    def __init__(self):
        self.items = []
  
    def item_count(self):
        return len(self.items)  # Better implementation

# Run test → STILL PASSES

# Continue cycle: Add more features

# RED: Test add_item
def test_add_item_increases_count():
    cart = ShoppingCart()
    cart.add_item('product123', quantity=2)
    assert cart.item_count() == 1  # One unique item

# Run → FAILS (add_item doesn't exist)

# GREEN: Implement add_item
class ShoppingCart:
    def __init__(self):
        self.items = []
  
    def add_item(self, product_id, quantity=1):
        self.items.append({
            'product_id': product_id,
            'quantity': quantity
        })
  
    def item_count(self):
        return len(self.items)

# Run → PASSES

# RED: Test calculate_total
def test_calculate_total():
    cart = ShoppingCart()
    cart.add_item('prod1', quantity=2, price=10.0)
    cart.add_item('prod2', quantity=1, price=20.0)
    assert cart.calculate_total() == 40.0

# GREEN: Implement calculate_total
class ShoppingCart:
    def __init__(self):
        self.items = []
  
    def add_item(self, product_id, quantity=1, price=0.0):
        self.items.append({
            'product_id': product_id,
            'quantity': quantity,
            'price': price
        })
  
    def item_count(self):
        return len(self.items)
  
    def calculate_total(self):
        return sum(item['quantity'] * item['price'] for item in self.items)

# Run → PASSES

# Benefits of TDD:
# ✅ Tests drive design (forces modularity)
# ✅ High test coverage (test-first)
# ✅ Confidence in refactoring
# ✅ Documentation through tests
# ✅ Prevents over-engineering (YAGNI)

# When to use TDD:
# ✅ New features with clear requirements
# ✅ Bug fixes (write test first)
# ✅ Complex business logic
# ✅ Public APIs

# When NOT to use TDD:
# ❌ Prototyping/experimentation
# ❌ UI/UX exploration
# ❌ Unclear requirements
# ❌ Simple CRUD operations
```

### Testing Principles

```python
"""
TESTING PRINCIPLES

1. AAA Pattern: Arrange, Act, Assert
2. Test Isolation
3. Test Determinism
4. Fast Tests
5. Maintainable Tests
"""

# 1. AAA Pattern (Arrange, Act, Assert)

def test_user_registration():
    # ARRANGE: Set up test data and dependencies
    db = TestDatabase()
    email = 'test@example.com'
    password = 'SecurePass123!'
  
    # ACT: Execute the code under test
    user = register_user(db, email, password)
  
    # ASSERT: Verify the outcome
    assert user.email == email
    assert user.is_active == True
    assert db.user_exists(email) == True

# Alternative: Given-When-Then (BDD style)
def test_user_registration_bdd():
    # GIVEN: a user with email and password
    email = 'test@example.com'
    password = 'SecurePass123!'
  
    # WHEN: user registers
    user = register_user(email, password)
  
    # THEN: user is created and active
    assert user.email == email
    assert user.is_active == True

# 2. Test Isolation (tests don't depend on each other)

# ❌ BAD: Tests depend on execution order
class TestUserService:
    def test_create_user(self):
        self.user_id = user_service.create('alice@example.com')
        assert self.user_id is not None
  
    def test_get_user(self):
        # Depends on test_create_user running first!
        user = user_service.get(self.user_id)
        assert user.email == 'alice@example.com'

# ✅ GOOD: Each test is independent
class TestUserService:
    @pytest.fixture
    def created_user(self):
        """Fixture creates user for each test"""
        user_id = user_service.create('alice@example.com')
        yield user_id
        user_service.delete(user_id)  # Cleanup
  
    def test_create_user(self):
        user_id = user_service.create('alice@example.com')
        assert user_id is not None
        user_service.delete(user_id)
  
    def test_get_user(self, created_user):
        user = user_service.get(created_user)
        assert user.email == 'alice@example.com'

# 3. Test Determinism (same input = same output)

# ❌ BAD: Non-deterministic test (flaky)
def test_get_current_users():
    users = user_service.get_active_users()
    assert len(users) == 5  # Depends on database state!

def test_with_random():
    import random
    value = random.randint(1, 100)
    result = process(value)
    assert result > 0  # May fail randomly

# ✅ GOOD: Deterministic tests
def test_get_current_users():
    # Create known test data
    db.clear()
    for i in range(5):
        db.create_user(f'user{i}@example.com')
  
    users = user_service.get_active_users()
    assert len(users) == 5

def test_with_seed():
    import random
    random.seed(42)  # Fixed seed
    value = random.randint(1, 100)
    result = process(value)
    assert result == 73  # Predictable with seed

# 4. Fast Tests

# ❌ SLOW: Test takes seconds
def test_slow_operation():
    time.sleep(2)  # Don't do this!
    result = heavy_computation()
    assert result == expected

# ✅ FAST: Test is instant
@pytest.mark.slow  # Mark slow tests
def test_with_database():
    # Use in-memory database
    db = sqlite3.connect(':memory:')
    # ... test logic
    # Runs in milliseconds

def test_with_mock():
    # Mock slow external calls
    with patch('module.slow_api_call') as mock_api:
        mock_api.return_value = {'status': 'success'}
        result = function_under_test()
        assert result == expected

# 5. Maintainable Tests

# ❌ BAD: Hard to maintain
def test_complex_scenario():
    user = User('test@example.com', 'pass123', 'John', 'Doe', 
                '123 Main St', 'City', 'State', '12345', 'US',
                '+1234567890', True, False, datetime.now(), None, 
                [], {'pref1': 'value1'}, None)
    # ... 50 more lines ...
    assert user.something == expected

# ✅ GOOD: Maintainable with fixtures
@pytest.fixture
def default_user():
    """Reusable user fixture"""
    return User(
        email='test@example.com',
        password='pass123',
        first_name='John',
        last_name='Doe'
    )

def test_simple_scenario(default_user):
    result = process_user(default_user)
    assert result.status == 'active'

# Test helper functions
def create_test_user(**overrides):
    """Factory function for test users"""
    defaults = {
        'email': 'test@example.com',
        'password': 'pass123',
        'first_name': 'John',
        'last_name': 'Doe',
        'is_active': True
    }
    defaults.update(overrides)
    return User(**defaults)

def test_inactive_user():
    user = create_test_user(is_active=False)
    assert cannot_login(user)
```

### Frequently Asked Questions

**Q1: How much test coverage should I aim for?**

**A:**

```python
"""
TEST COVERAGE TARGET

Common targets:
- 80%+ coverage: Good baseline
- 90%+ coverage: Excellent
- 100% coverage: Unnecessary (diminishing returns)

Coverage ≠ Quality!
High coverage doesn't mean good tests.
"""

# ❌ BAD: 100% coverage with poor tests
def add(a, b):
    return a + b

def test_add():
    add(2, 3)  # No assertion! Still counts as "covered"

# ✅ GOOD: Lower coverage with meaningful tests
def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0

# Focus on:
# 1. Critical paths (authentication, payments, data loss)
# 2. Business logic (calculations, validations)
# 3. Edge cases and error handling
# 4. Public APIs

# Don't obsess over:
# - Simple getters/setters
# - Framework boilerplate
# - Generated code
# - Configuration files

# Coverage by component:
"""
Core Business Logic:    95%+ coverage (critical!)
API Endpoints:          90%+ coverage
Database Models:        80%+ coverage
Utilities:              70%+ coverage
Configuration:          50%+ coverage (less critical)
"""

# Measure what matters:
# - Branch coverage (not just line coverage)
# - Mutation score (test quality)
# - Regression detection
```

**Why This Matters:** Chasing 100% coverage wastes time. Focus on high-value tests.

---

**Q2: Should I write tests for legacy code without tests?**

**A:**

```python
"""
TESTING LEGACY CODE STRATEGY

Golden Rule: Don't write tests for code you're not changing.

Approach:
1. Write tests for new features only
2. When fixing bugs, add regression test
3. When refactoring, add safety net tests
4. Gradually improve coverage
"""

# Strategy 1: Characterization Tests
# Document current behavior (even if it's wrong)

def test_legacy_calculation():
    """Characterization test: documents current behavior"""
    # This documents what the code DOES (not what it SHOULD do)
    result = legacy_calculate(10, 20, 5)
    assert result == 42  # Whatever it currently returns
  
    # Now you can refactor safely!

# Strategy 2: Golden Master Testing
# Capture output, compare future runs

import json

def test_legacy_report_generation():
    """Golden master test"""
    report = generate_legacy_report(user_id='123')
  
    # First run: Save output as "golden master"
    # golden_master = report
    # save_json('golden_master.json', golden_master)
  
    # Future runs: Compare against golden master
    expected = load_json('golden_master.json')
    assert report == expected

# Strategy 3: Approval Testing
# Review output manually, then lock it in

def test_legacy_email_template():
    """Approval test"""
    email = generate_welcome_email(user_name='Alice')
  
    # First run: Review email_output.html manually
    # Approve: Copy to email_output.approved.html
  
    # Future runs: Compare against approved version
    approved = read_file('email_output.approved.html')
    assert email == approved

# Strategy 4: Seam Injection
# Create testing seams in legacy code

# Legacy code (can't test directly)
class LegacyService:
    def process(self):
        # Directly accesses database
        data = database.query("SELECT * FROM users")
        # Process...
        return result

# Refactored: Inject dependency
class RefactoredService:
    def __init__(self, database):
        self.database = database  # Injected!
  
    def process(self):
        data = self.database.query("SELECT * FROM users")
        return result

# Now testable!
def test_refactored_service():
    mock_db = Mock()
    mock_db.query.return_value = [{'id': 1, 'name': 'Alice'}]
  
    service = RefactoredService(mock_db)
    result = service.process()
  
    assert result is not None

# Prioritize tests for:
# ✅ Code you're actively changing
# ✅ Bug fixes (write regression test)
# ✅ Critical business logic
# ✅ High-risk areas

# Don't waste time on:
# ❌ Code that never changes
# ❌ Code scheduled for deletion
# ❌ Simple CRUD operations
```

**Why This Matters:** Testing every line of legacy code is impractical. Focus on value.

---

### Interview Questions

**Question 1: Explain the testing pyramid and why it's important.**

**Difficulty:** Mid-Level

**SDLC Relevance:** Testing Strategy

**Answer:**

```python
"""
TESTING PYRAMID

Structure (bottom to top):
┌─────────────┐
│   E2E (10%) │ ← Few, slow, expensive
├─────────────┤
│ Integration │ ← Moderate, medium speed
│    (20%)    │
├─────────────┤
│   Unit      │ ← Many, fast, cheap
│   (70%)     │
└─────────────┘

Why Pyramid Shape?

1. Speed: Unit tests run in milliseconds
   - Instant feedback during development
   - Fast CI/CD pipeline
   - Example: 1000 unit tests = 1 second
            100 integration tests = 10 seconds
            10 E2E tests = 1 minute
            Total: ~71 seconds

2. Isolation: Unit tests pinpoint failures
   - "test_calculate_discount failed" → exact problem
   - E2E fails → could be anywhere in system

3. Maintenance: Unit tests easy to maintain
   - Small, focused tests
   - Few dependencies
   - Less brittle

4. Cost: Unit tests cheapest to write/run
   - No infrastructure needed
   - No external dependencies
   - Can run locally

Anti-Pattern: Inverted Pyramid (Ice Cream Cone)
┌─────────────┐
│   Unit      │ ← Few unit tests
│   (10%)     │
├─────────────┤
│ Integration │
│    (20%)    │
├─────────────┤
│   E2E       │ ← Too many E2E tests
│   (70%)     │
└─────────────┘

Problems:
❌ Slow feedback (hours to run tests)
❌ Flaky tests (timing issues)
❌ Hard to debug (failures anywhere)
❌ Expensive to maintain
"""

# Example implementation:

# Layer 1: Unit Tests (70%)
def test_calculate_discount():
    """Fast, isolated, specific"""
    assert calculate_discount(100, 10) == 90.0

# Layer 2: Integration Tests (20%)
def test_order_service_with_db(test_db):
    """Tests components together"""
    service = OrderService(test_db)
    order = service.create_order(user_id='123', items=[...])
    assert test_db.get_order(order.id) is not None

# Layer 3: E2E Tests (10%)
def test_complete_checkout_flow(api_client):
    """Tests entire system"""
    # Login → Add to cart → Checkout → Verify order
    # Tests UI, API, database, payment, email
```

**Key Points to Mention:**

- Pyramid shape: many unit tests, fewer integration, fewest E2E
- Reasons: speed, isolation, maintenance, cost
- Anti-pattern: inverted pyramid (ice cream cone)
- Balance: confidence vs speed
- Each layer serves different purpose

---

## 4.2 Unit Testing

**SDLC Phase:** Development, Testing

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (TDD)
- [X] Testing
- [ ] Deployment
- [X] Maintenance (refactoring)

### unittest Framework

```python
"""
UNITTEST: Python's built-in testing framework

Based on JUnit, uses classes and methods
"""

import unittest
from datetime import datetime

# Code under test
class User:
    def __init__(self, email, password):
        self.email = email
        self.password = password
        self.is_active = True
        self.created_at = datetime.utcnow()
  
    def deactivate(self):
        self.is_active = False
  
    def update_email(self, new_email):
        if '@' not in new_email:
            raise ValueError("Invalid email")
        self.email = new_email

# Test case
class TestUser(unittest.TestCase):
    """Test User class"""
  
    def setUp(self):
        """Run before each test method"""
        self.user = User('test@example.com', 'password123')
  
    def tearDown(self):
        """Run after each test method"""
        # Cleanup if needed
        pass
  
    @classmethod
    def setUpClass(cls):
        """Run once before all tests in class"""
        print("Setting up test class")
  
    @classmethod
    def tearDownClass(cls):
        """Run once after all tests in class"""
        print("Tearing down test class")
  
    # Test methods
    def test_user_creation(self):
        """Test user is created with correct attributes"""
        self.assertEqual(self.user.email, 'test@example.com')
        self.assertTrue(self.user.is_active)
        self.assertIsNotNone(self.user.created_at)
  
    def test_deactivate_user(self):
        """Test user deactivation"""
        self.user.deactivate()
        self.assertFalse(self.user.is_active)
  
    def test_update_email_valid(self):
        """Test updating email with valid address"""
        new_email = 'newemail@example.com'
        self.user.update_email(new_email)
        self.assertEqual(self.user.email, new_email)
  
    def test_update_email_invalid(self):
        """Test updating email with invalid address raises error"""
        with self.assertRaises(ValueError):
            self.user.update_email('invalid-email')
  
    def test_update_email_invalid_with_message(self):
        """Test exception message"""
        with self.assertRaisesRegex(ValueError, "Invalid email"):
            self.user.update_email('invalid-email')

# Common assertions
class TestAssertions(unittest.TestCase):
    def test_equality(self):
        self.assertEqual(2 + 2, 4)
        self.assertNotEqual(2 + 2, 5)
  
    def test_truth(self):
        self.assertTrue(True)
        self.assertFalse(False)
  
    def test_none(self):
        self.assertIsNone(None)
        self.assertIsNotNone("value")
  
    def test_membership(self):
        self.assertIn(1, [1, 2, 3])
        self.assertNotIn(4, [1, 2, 3])
  
    def test_type(self):
        self.assertIsInstance("text", str)
        self.assertIsInstance(42, int)
  
    def test_comparison(self):
        self.assertGreater(5, 3)
        self.assertLess(3, 5)
        self.assertGreaterEqual(5, 5)
        self.assertLessEqual(3, 5)
  
    def test_float_comparison(self):
        # Use assertAlmostEqual for floats
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=7)
  
    def test_collections(self):
        self.assertListEqual([1, 2], [1, 2])
        self.assertDictEqual({'a': 1}, {'a': 1})
        self.assertSetEqual({1, 2}, {2, 1})

# Test suites
def suite():
    """Create test suite"""
    suite = unittest.TestSuite()
    suite.addTest(TestUser('test_user_creation'))
    suite.addTest(TestUser('test_deactivate_user'))
    return suite

# Run tests
if __name__ == '__main__':
    # Method 1: Run all tests
    unittest.main()
  
    # Method 2: Run specific suite
    # runner = unittest.TextTestRunner()
    # runner.run(suite())
  
    # Method 3: Discovery
    # unittest.main(module=None, argv=['', 'discover'])
```

### pytest Framework

```python
"""
PYTEST: Modern, Pythonic testing framework

Advantages over unittest:
- Simple assert statements (no assertEqual, assertTrue, etc.)
- Powerful fixtures
- Parametrized tests
- Better error messages
- Large plugin ecosystem
"""

import pytest

# Simple test (no class needed!)
def test_addition():
    """Test simple addition"""
    assert 2 + 2 == 4

def test_string_operations():
    """Test string operations"""
    text = "hello"
    assert text.upper() == "HELLO"
    assert len(text) == 5
    assert "ell" in text

# Fixtures
@pytest.fixture
def user():
    """Fixture: Create user for testing"""
    return User('test@example.com', 'password123')

def test_user_email(user):
    """Test using fixture"""
    assert user.email == 'test@example.com'

def test_user_is_active(user):
    """Another test using same fixture"""
    assert user.is_active == True

# Fixture scopes
@pytest.fixture(scope="function")  # Default: new instance per test
def function_scope():
    return expensive_setup()

@pytest.fixture(scope="class")  # One instance per test class
def class_scope():
    return expensive_setup()

@pytest.fixture(scope="module")  # One instance per module
def module_scope():
    return expensive_setup()

@pytest.fixture(scope="session")  # One instance per test session
def session_scope():
    return expensive_setup()

# Fixture with setup/teardown
@pytest.fixture
def database():
    """Database fixture with cleanup"""
    db = Database('test.db')
    db.connect()
    yield db  # This is returned to test
    db.disconnect()  # Cleanup runs after test

def test_database_query(database):
    result = database.query("SELECT * FROM users")
    assert len(result) > 0

# Parametrized tests
@pytest.mark.parametrize("input,expected", [
    (2, 4),
    (3, 9),
    (4, 16),
    (5, 25),
])
def test_square(input, expected):
    """Test square function with multiple inputs"""
    assert square(input) == expected

# Multiple parameters
@pytest.mark.parametrize("x,y,expected", [
    (1, 2, 3),
    (2, 3, 5),
    (10, 20, 30),
])
def test_add(x, y, expected):
    assert add(x, y) == expected

# Test exceptions
def test_division_by_zero():
    """Test exception is raised"""
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_exception_message():
    """Test exception message"""
    with pytest.raises(ValueError, match="Invalid email"):
        validate_email("invalid")

# Markers
@pytest.mark.slow
def test_slow_operation():
    """Mark as slow test"""
    time.sleep(2)
    assert expensive_computation() == expected

@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    """Skip this test"""
    pass

@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires Python 3.10+")
def test_python_310_feature():
    """Skip on older Python versions"""
    pass

@pytest.mark.xfail(reason="Known bug")
def test_known_bug():
    """Expected to fail"""
    assert buggy_function() == expected

# Run specific markers:
# pytest -m slow  # Run only slow tests
# pytest -m "not slow"  # Skip slow tests

# conftest.py (shared fixtures)
"""
conftest.py - Automatically discovered by pytest
"""
import pytest

@pytest.fixture
def api_client():
    """Shared fixture available to all tests"""
    client = TestClient(app)
    return client

# pytest.ini (configuration)
"""
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests
    unit: marks tests as unit tests
"""
```

### Mocking

```python
"""
MOCKING: Replace dependencies with test doubles

When to mock:
- External APIs
- Databases
- File system
- Time-dependent code
- Expensive operations
"""

from unittest.mock import Mock, MagicMock, patch, call
import pytest

# Code under test
class EmailService:
    def send_email(self, to, subject, body):
        # Actually sends email
        pass

class UserService:
    def __init__(self, db, email_service):
        self.db = db
        self.email_service = email_service
  
    def register_user(self, email, password):
        # Create user in database
        user = self.db.create_user(email, password)
      
        # Send welcome email
        self.email_service.send_email(
            to=email,
            subject="Welcome!",
            body="Thanks for registering"
        )
      
        return user

# Test with mocks
def test_user_registration():
    # Create mocks
    mock_db = Mock()
    mock_email = Mock()
  
    # Configure mock return value
    mock_db.create_user.return_value = User('test@example.com')
  
    # Create service with mocks
    service = UserService(mock_db, mock_email)
  
    # Call method
    user = service.register_user('test@example.com', 'password123')
  
    # Verify mock was called correctly
    mock_db.create_user.assert_called_once_with('test@example.com', 'password123')
    mock_email.send_email.assert_called_once_with(
        to='test@example.com',
        subject="Welcome!",
        body="Thanks for registering"
    )
  
    assert user.email == 'test@example.com'

# Mock return values
def test_mock_return_value():
    mock = Mock()
    mock.method.return_value = 42
    assert mock.method() == 42

# Mock side effects (exceptions, multiple returns)
def test_mock_side_effect_exception():
    mock = Mock()
    mock.method.side_effect = ValueError("Error!")
  
    with pytest.raises(ValueError):
        mock.method()

def test_mock_side_effect_sequence():
    mock = Mock()
    mock.method.side_effect = [1, 2, 3]
  
    assert mock.method() == 1
    assert mock.method() == 2
    assert mock.method() == 3

# Mock with custom function
def test_mock_side_effect_function():
    mock = Mock()
    mock.calculate.side_effect = lambda x: x * 2
  
    assert mock.calculate(5) == 10

# Verify call arguments
def test_mock_call_arguments():
    mock = Mock()
    mock.method(1, 2, key='value')
  
    # Check if called
    assert mock.method.called
    assert mock.method.call_count == 1
  
    # Check call arguments
    mock.method.assert_called_with(1, 2, key='value')
    mock.method.assert_called_once_with(1, 2, key='value')
  
    # Get call arguments
    args, kwargs = mock.method.call_args
    assert args == (1, 2)
    assert kwargs == {'key': 'value'}

# Verify multiple calls
def test_mock_multiple_calls():
    mock = Mock()
    mock.method(1)
    mock.method(2)
    mock.method(3)
  
    assert mock.method.call_count == 3
    mock.method.assert_has_calls([
        call(1),
        call(2),
        call(3)
    ])

# Patch decorator (replace at runtime)
@patch('module.EmailService')
def test_with_patch(mock_email_class):
    """Patch replaces EmailService with mock"""
    mock_email = mock_email_class.return_value
  
    service = UserService(db, mock_email)
    service.register_user('test@example.com', 'pass123')
  
    mock_email.send_email.assert_called_once()

# Patch context manager
def test_with_patch_context():
    """Patch using context manager"""
    with patch('module.EmailService') as mock_email_class:
        mock_email = mock_email_class.return_value
      
        service = UserService(db, mock_email)
        service.register_user('test@example.com', 'pass123')
      
        mock_email.send_email.assert_called_once()

# Patch multiple
@patch('module.EmailService')
@patch('module.Database')
def test_patch_multiple(mock_db_class, mock_email_class):
    """Multiple patches (bottom to top order!)"""
    pass

# Patch object attribute
def test_patch_object():
    service = UserService(db, email_service)
  
    with patch.object(service, 'email_service') as mock_email:
        service.register_user('test@example.com', 'pass123')
        mock_email.send_email.assert_called_once()

# MagicMock (supports magic methods)
def test_magic_mock():
    mock = MagicMock()
  
    # Supports iteration
    mock.__iter__.return_value = iter([1, 2, 3])
    assert list(mock) == [1, 2, 3]
  
    # Supports context manager
    with mock as m:
        pass
    mock.__enter__.assert_called_once()
    mock.__exit__.assert_called_once()

# When NOT to mock:
# ❌ Don't mock what you don't own (third-party libraries)
# ❌ Don't mock simple objects (use real ones)
# ❌ Don't mock everything (test real integration)
```

### Test Doubles

```python
"""
TEST DOUBLES: Different types of fake objects

1. Dummy: Objects passed but never used
2. Stub: Returns hard-coded responses
3. Spy: Records calls for verification
4. Mock: Verifies behavior (calls, arguments)
5. Fake: Working implementation (simplified)
"""

# 1. Dummy (passed but not used)
class DummyLogger:
    """Dummy: Never actually called"""
    def log(self, message):
        pass  # Does nothing

def test_with_dummy():
    dummy_logger = DummyLogger()
    service = Service(logger=dummy_logger)  # Passed but not used
    result = service.calculate(10, 20)
    assert result == 30

# 2. Stub (hard-coded responses)
class StubDatabase:
    """Stub: Returns pre-programmed data"""
    def get_user(self, user_id):
        # Always returns same user
        return User(id=user_id, name='Test User')

def test_with_stub():
    stub_db = StubDatabase()
    service = UserService(stub_db)
    user = service.get_user('123')
    assert user.name == 'Test User'

# 3. Spy (records interactions)
class SpyEmailService:
    """Spy: Records all method calls"""
    def __init__(self):
        self.calls = []
  
    def send_email(self, to, subject, body):
        self.calls.append({
            'to': to,
            'subject': subject,
            'body': body
        })

def test_with_spy():
    spy = SpyEmailService()
    service = UserService(db, spy)
  
    service.register_user('alice@example.com', 'pass123')
    service.register_user('bob@example.com', 'pass456')
  
    # Verify calls
    assert len(spy.calls) == 2
    assert spy.calls[0]['to'] == 'alice@example.com'
    assert spy.calls[1]['to'] == 'bob@example.com'

# 4. Mock (behavior verification)
def test_with_mock():
    mock_email = Mock()
    service = UserService(db, mock_email)
  
    service.register_user('test@example.com', 'pass123')
  
    # Verify behavior
    mock_email.send_email.assert_called_once_with(
        to='test@example.com',
        subject='Welcome!',
        body='Thanks for registering'
    )

# 5. Fake (working implementation)
class FakeDatabase:
    """Fake: In-memory database"""
    def __init__(self):
        self.users = {}
  
    def create_user(self, email, password):
        user_id = str(len(self.users) + 1)
        user = User(id=user_id, email=email)
        self.users[user_id] = user
        return user
  
    def get_user(self, user_id):
        return self.users.get(user_id)
  
    def delete_user(self, user_id):
        if user_id in self.users:
            del self.users[user_id]

def test_with_fake():
    fake_db = FakeDatabase()
    service = UserService(fake_db, email_service)
  
    # Create users
    user1 = service.register_user('alice@example.com', 'pass123')
    user2 = service.register_user('bob@example.com', 'pass456')
  
    # Verify in fake database
    assert fake_db.get_user(user1.id) is not None
    assert fake_db.get_user(user2.id) is not None
    assert len(fake_db.users) == 2

# When to use each:

# Dummy: When parameter is required but not used
# Stub: When you need hard-coded responses
# Spy: When you need to verify interactions
# Mock: When you need strict behavior verification
# Fake: When you need a working but simplified implementation
```

### Code Coverage

```python
"""
CODE COVERAGE: Measure which code is executed during tests

Install: pip install coverage pytest-cov
"""

# Run with coverage
# $ coverage run -m pytest
# $ coverage report
# $ coverage html  # Generate HTML report

# pytest-cov (simpler)
# $ pytest --cov=mymodule --cov-report=html

# Example output:
"""
Name                Stmts   Miss  Cover
---------------------------------------
mymodule/__init__.py    2      0   100%
mymodule/user.py       45      5    89%
mymodule/service.py    67     12    82%
---------------------------------------
TOTAL                 114     17    85%
"""

# Configuration (.coveragerc)
"""
[run]
source = mymodule
omit =
    */tests/*
    */migrations/*
    */__pycache__/*

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
    if __name__ == .__main__.:
    if TYPE_CHECKING:
"""

# Branch coverage (more thorough)
# $ pytest --cov=mymodule --cov-branch

# Example: Missing branch coverage
def discount_price(price, customer_type):
    if customer_type == 'premium':  # Branch 1
        return price * 0.8
    return price  # Branch 2

# Test (only covers branch 1)
def test_premium_discount():
    assert discount_price(100, 'premium') == 80

# Coverage report shows: 75% branch coverage (missing else branch)

# Fix: Test both branches
def test_premium_discount():
    assert discount_price(100, 'premium') == 80

def test_regular_price():
    assert discount_price(100, 'regular') == 100  # Now 100% branch coverage

# Ignore specific lines
def debug_function():
    print("Debug info")  # pragma: no cover

# Coverage in CI/CD
"""
# GitHub Actions
- name: Test with coverage
  run: |
    pytest --cov=mymodule --cov-report=xml
  
- name: Upload coverage
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage.xml
"""

# Coverage thresholds (fail if below)
# pytest.ini
"""
[pytest]
addopts = --cov=mymodule --cov-fail-under=80
"""

# ✅ GOOD: High-value coverage
# - Critical business logic
# - Error handling
# - Edge cases
# - Public APIs

# ❌ Don't obsess over:
# - Simple getters/setters
# - Framework code
# - Generated code
```

### Frequently Asked Questions

**Q1: pytest vs unittest - which should I use?**

**A:**

```python
"""
PYTEST VS UNITTEST

Use pytest when:
✅ Starting new project
✅ Want simple, Pythonic tests
✅ Need powerful fixtures
✅ Want parametrized tests
✅ Need plugin ecosystem

Use unittest when:
✅ Company/team standard
✅ Need standard library only (no dependencies)
✅ Migrating from other xUnit frameworks

Recommendation: Use pytest for new projects
"""

# Comparison:

# unittest: More boilerplate
import unittest

class TestCalculator(unittest.TestCase):
    def setUp(self):
        self.calc = Calculator()
  
    def test_add(self):
        self.assertEqual(self.calc.add(2, 3), 5)
  
    def test_divide_by_zero(self):
        with self.assertRaises(ZeroDivisionError):
            self.calc.divide(10, 0)

# pytest: More concise
import pytest

@pytest.fixture
def calc():
    return Calculator()

def test_add(calc):
    assert calc.add(2, 3) == 5

def test_divide_by_zero(calc):
    with pytest.raises(ZeroDivisionError):
        calc.divide(10, 0)

# pytest advantages:
# - Simple assert statements
# - Better error messages
# - Powerful fixtures
# - Parametrization
# - Plugin ecosystem
# - Can run unittest tests!

# Migration: pytest can run unittest tests
# $ pytest  # Runs both pytest and unittest tests
```

**Why This Matters:** Choosing the right framework affects productivity and test maintainability.

---

(Continuing with sections 4.3, 4.4, and 4.5 in next file...)

## 4.3 Integration Testing

**SDLC Phase:** Testing, System Integration

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development
- [X] Testing (integration)
- [ ] Deployment
- [X] Maintenance

Integration tests verify that components work together correctly.

### Database Testing

```python
"""
DATABASE INTEGRATION TESTING

Strategies:
1. Test database (separate from production)
2. In-memory database (SQLite :memory:)
3. Docker containers (real database)
4. Transactions with rollback
"""

# Strategy 1: Test Database with Fixtures

import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

@pytest.fixture(scope="session")
def test_engine():
    """Create test database engine (once per test session)"""
    engine = create_engine('postgresql://test:test@localhost/test_db')
  
    # Create all tables
    Base.metadata.create_all(engine)
  
    yield engine
  
    # Drop all tables after tests
    Base.metadata.drop_all(engine)

@pytest.fixture(scope="function")
def db_session(test_engine):
    """Create database session with transaction rollback"""
    connection = test_engine.connect()
    transaction = connection.begin()
  
    Session = sessionmaker(bind=connection)
    session = Session()
  
    yield session
  
    # Rollback transaction (test changes not committed)
    session.close()
    transaction.rollback()
    connection.close()

# Test with real database
def test_create_user(db_session):
    """Test user creation in database"""
    user = User(email='test@example.com', name='Test User')
    db_session.add(user)
    db_session.commit()
  
    # Verify user in database
    queried_user = db_session.query(User).filter_by(email='test@example.com').first()
    assert queried_user is not None
    assert queried_user.name == 'Test User'

def test_user_relationships(db_session):
    """Test user-order relationship"""
    user = User(email='test@example.com', name='Test User')
    db_session.add(user)
    db_session.commit()
  
    # Create orders
    order1 = Order(user_id=user.id, total=100.0)
    order2 = Order(user_id=user.id, total=200.0)
    db_session.add_all([order1, order2])
    db_session.commit()
  
    # Verify relationship
    assert len(user.orders) == 2
    assert sum(o.total for o in user.orders) == 300.0

# Strategy 2: In-Memory Database (SQLite)

@pytest.fixture
def memory_db():
    """In-memory SQLite database (fast!)"""
    engine = create_engine('sqlite:///:memory:')
    Base.metadata.create_all(engine)
  
    Session = sessionmaker(bind=engine)
    session = Session()
  
    yield session
  
    session.close()

def test_with_memory_db(memory_db):
    """Test with in-memory database (very fast)"""
    user = User(email='test@example.com', name='Test')
    memory_db.add(user)
    memory_db.commit()
  
    assert memory_db.query(User).count() == 1

# Strategy 3: Docker Container (Real Database)

import pytest
from testcontainers.postgres import PostgresContainer

@pytest.fixture(scope="session")
def postgres_container():
    """Start PostgreSQL in Docker container"""
    with PostgresContainer("postgres:15") as postgres:
        engine = create_engine(postgres.get_connection_url())
        Base.metadata.create_all(engine)
        yield engine

def test_with_docker_postgres(postgres_container):
    """Test with real PostgreSQL in Docker"""
    Session = sessionmaker(bind=postgres_container)
    session = Session()
  
    user = User(email='test@example.com', name='Test')
    session.add(user)
    session.commit()
  
    assert session.query(User).count() == 1

# Seed Data Fixtures

@pytest.fixture
def seed_data(db_session):
    """Populate database with test data"""
    users = [
        User(email=f'user{i}@example.com', name=f'User {i}')
        for i in range(10)
    ]
    db_session.add_all(users)
    db_session.commit()
  
    return users

def test_with_seed_data(db_session, seed_data):
    """Test with pre-populated data"""
    users = db_session.query(User).all()
    assert len(users) == 10

# Testing Migrations

def test_migration_creates_table():
    """Test Alembic migration"""
    from alembic.config import Config
    from alembic import command
  
    # Setup test database
    alembic_cfg = Config("alembic.ini")
    alembic_cfg.set_main_option("sqlalchemy.url", "postgresql://test:test@localhost/test_db")
  
    # Run migration
    command.upgrade(alembic_cfg, "head")
  
    # Verify table exists
    engine = create_engine("postgresql://test:test@localhost/test_db")
    assert engine.dialect.has_table(engine.connect(), "users")
  
    # Rollback
    command.downgrade(alembic_cfg, "base")

# ✅ GOOD: Fast, isolated database tests
# - Use transactions with rollback
# - Use in-memory database for speed
# - Use Docker for real database tests
# - Seed data with fixtures
```

### API Testing

```python
"""
API INTEGRATION TESTING

Test REST APIs end-to-end
"""

import pytest
from fastapi.testclient import TestClient
from httpx import AsyncClient

# FastAPI example
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: str):
    return {"id": user_id, "name": "Test User"}

@app.post("/users")
def create_user(email: str, name: str):
    return {"id": "123", "email": email, "name": name}

# Test client fixture
@pytest.fixture
def client():
    """Test client for API"""
    return TestClient(app)

# Test GET endpoint
def test_get_user(client):
    """Test GET /users/{user_id}"""
    response = client.get("/users/123")
  
    assert response.status_code == 200
    assert response.json() == {
        "id": "123",
        "name": "Test User"
    }

# Test POST endpoint
def test_create_user(client):
    """Test POST /users"""
    response = client.post("/users", params={
        "email": "test@example.com",
        "name": "Test User"
    })
  
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == "test@example.com"
    assert data["name"] == "Test User"
    assert "id" in data

# Test error cases
def test_user_not_found(client):
    """Test 404 error"""
    response = client.get("/users/nonexistent")
    assert response.status_code == 404

def test_invalid_input(client):
    """Test 422 validation error"""
    response = client.post("/users", params={
        "email": "invalid-email",  # Invalid format
        "name": ""
    })
    assert response.status_code == 422

# Test authentication
def test_protected_endpoint(client):
    """Test endpoint requires authentication"""
    # Without auth
    response = client.get("/protected")
    assert response.status_code == 401
  
    # With auth token
    token = "test-token"
    response = client.get(
        "/protected",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200

# Test async endpoints
@pytest.fixture
async def async_client():
    """Async test client"""
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.mark.asyncio
async def test_async_endpoint(async_client):
    """Test async endpoint"""
    response = await async_client.get("/users/123")
    assert response.status_code == 200

# JSON Schema validation
import json
from jsonschema import validate

USER_SCHEMA = {
    "type": "object",
    "properties": {
        "id": {"type": "string"},
        "email": {"type": "string"},
        "name": {"type": "string"}
    },
    "required": ["id", "email", "name"]
}

def test_response_schema(client):
    """Test response matches schema"""
    response = client.post("/users", params={
        "email": "test@example.com",
        "name": "Test"
    })
  
    data = response.json()
    validate(instance=data, schema=USER_SCHEMA)  # Validates schema

# Test pagination
def test_pagination(client):
    """Test paginated endpoint"""
    response = client.get("/users?page=1&page_size=10")
  
    assert response.status_code == 200
    data = response.json()
  
    assert "items" in data
    assert "total" in data
    assert "page" in data
    assert len(data["items"]) <= 10

# Test rate limiting
import time

def test_rate_limiting(client):
    """Test rate limiting works"""
    # Make multiple requests quickly
    for i in range(10):
        response = client.get("/api/endpoint")
        if i < 5:
            assert response.status_code == 200
        else:
            # Should be rate limited after 5 requests
            assert response.status_code == 429
  
    # Wait for rate limit to reset
    time.sleep(60)
    response = client.get("/api/endpoint")
    assert response.status_code == 200

# Complete integration test
def test_user_flow_integration(client):
    """Test complete user flow"""
    # 1. Register user
    response = client.post("/auth/register", json={
        "email": "test@example.com",
        "password": "SecurePass123!"
    })
    assert response.status_code == 201
  
    # 2. Login
    response = client.post("/auth/login", json={
        "email": "test@example.com",
        "password": "SecurePass123!"
    })
    assert response.status_code == 200
    token = response.json()["access_token"]
  
    # 3. Access protected resource
    response = client.get(
        "/users/me",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200
    assert response.json()["email"] == "test@example.com"
  
    # 4. Update profile
    response = client.put(
        "/users/me",
        headers={"Authorization": f"Bearer {token}"},
        json={"name": "Updated Name"}
    )
    assert response.status_code == 200
  
    # 5. Logout
    response = client.post(
        "/auth/logout",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200
```

### External Service Testing

```python
"""
EXTERNAL SERVICE TESTING

Strategies:
1. Mock external services
2. Record/replay HTTP interactions (VCR.py)
3. Use service test instances
4. Docker Compose for dependencies
"""

# Strategy 1: Mock External Services

import pytest
from unittest.mock import patch, Mock

# Code that calls external API
import requests

class PaymentService:
    def charge(self, amount, card_token):
        response = requests.post(
            'https://api.stripe.com/v1/charges',
            data={'amount': amount, 'source': card_token}
        )
        return response.json()

# Test with mock
@patch('requests.post')
def test_payment_success(mock_post):
    """Test successful payment"""
    # Mock external API response
    mock_response = Mock()
    mock_response.json.return_value = {
        'id': 'ch_123',
        'status': 'succeeded'
    }
    mock_post.return_value = mock_response
  
    service = PaymentService()
    result = service.charge(1000, 'tok_visa')
  
    assert result['status'] == 'succeeded'
    mock_post.assert_called_once()

# Strategy 2: VCR.py (Record/Replay)

import vcr

# First run: Records HTTP interactions to cassette
# Subsequent runs: Replays from cassette (no network calls)

@vcr.use_cassette('fixtures/vcr_cassettes/payment.yaml')
def test_payment_with_vcr():
    """Test payment with VCR (records/replays HTTP)"""
    service = PaymentService()
    result = service.charge(1000, 'tok_visa')
  
    # First run: Makes real API call, records to payment.yaml
    # Later runs: Replays from payment.yaml (fast, no network)
    assert result['status'] == 'succeeded'

# Cassette file (payment.yaml):
"""
interactions:
- request:
    method: POST
    uri: https://api.stripe.com/v1/charges
    body: amount=1000&source=tok_visa
  response:
    status: 200
    body: '{"id": "ch_123", "status": "succeeded"}'
"""

# Strategy 3: responses library (mock requests)

import responses

@responses.activate
def test_payment_with_responses():
    """Test with responses library"""
    # Mock external endpoint
    responses.add(
        responses.POST,
        'https://api.stripe.com/v1/charges',
        json={'id': 'ch_123', 'status': 'succeeded'},
        status=200
    )
  
    service = PaymentService()
    result = service.charge(1000, 'tok_visa')
  
    assert result['status'] == 'succeeded'
    assert len(responses.calls) == 1

@responses.activate
def test_payment_failure():
    """Test payment failure"""
    responses.add(
        responses.POST,
        'https://api.stripe.com/v1/charges',
        json={'error': 'card_declined'},
        status=402
    )
  
    service = PaymentService()
    with pytest.raises(requests.exceptions.HTTPError):
        result = service.charge(1000, 'tok_visa')

# Strategy 4: Testcontainers (real services in Docker)

from testcontainers.postgres import PostgresContainer
from testcontainers.redis import RedisContainer

@pytest.fixture(scope="session")
def redis_container():
    """Redis in Docker for testing"""
    with RedisContainer() as redis:
        yield redis

def test_with_redis(redis_container):
    """Test with real Redis"""
    import redis
  
    client = redis.Redis(
        host=redis_container.get_container_host_ip(),
        port=redis_container.get_exposed_port(6379)
    )
  
    client.set('key', 'value')
    assert client.get('key') == b'value'

# Docker Compose for complex setups
"""
# docker-compose.test.yml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: test
  redis:
    image: redis:7
  rabbitmq:
    image: rabbitmq:3
"""

# Test with docker-compose
import subprocess

@pytest.fixture(scope="session")
def docker_services():
    """Start services with docker-compose"""
    subprocess.run(['docker-compose', '-f', 'docker-compose.test.yml', 'up', '-d'])
    yield
    subprocess.run(['docker-compose', '-f', 'docker-compose.test.yml', 'down'])

def test_with_docker_compose(docker_services):
    """Test with docker-compose services"""
    # Services are running, test normally
    pass

# ✅ GOOD: External service testing
# - Mock for unit tests
# - VCR.py for reproducible integration tests
# - Docker for real service testing
# - Separate test environment
```

### Frequently Asked Questions

**Q1: How do I test database migrations safely?**

**A:**

```python
"""
DATABASE MIGRATION TESTING STRATEGY

1. Test migrations in CI/CD before production
2. Use separate test database
3. Test both upgrade and downgrade
4. Verify data integrity
5. Test with production-like data volume
"""

# Test migration script
import pytest
from alembic.config import Config
from alembic import command
from sqlalchemy import create_engine, inspect

@pytest.fixture
def test_db_url():
    """Test database URL"""
    return "postgresql://test:test@localhost/migration_test"

def test_migration_upgrade(test_db_url):
    """Test migration upgrade"""
    # Setup Alembic config
    alembic_cfg = Config("alembic.ini")
    alembic_cfg.set_main_option("sqlalchemy.url", test_db_url)
  
    # Start from base
    command.downgrade(alembic_cfg, "base")
  
    # Run migration
    command.upgrade(alembic_cfg, "head")
  
    # Verify schema
    engine = create_engine(test_db_url)
    inspector = inspect(engine)
  
    # Check tables exist
    tables = inspector.get_table_names()
    assert "users" in tables
    assert "orders" in tables
  
    # Check columns
    columns = {col['name'] for col in inspector.get_columns('users')}
    assert "id" in columns
    assert "email" in columns
    assert "created_at" in columns

def test_migration_downgrade(test_db_url):
    """Test migration downgrade (rollback)"""
    alembic_cfg = Config("alembic.ini")
    alembic_cfg.set_main_option("sqlalchemy.url", test_db_url)
  
    # Upgrade to head
    command.upgrade(alembic_cfg, "head")
  
    # Downgrade one version
    command.downgrade(alembic_cfg, "-1")
  
    # Verify schema changed back
    engine = create_engine(test_db_url)
    inspector = inspect(engine)
    # ... verify rollback worked

def test_migration_with_data(test_db_url):
    """Test migration preserves data"""
    engine = create_engine(test_db_url)
  
    # Create old schema and add data
    command.upgrade(alembic_cfg, "abc123")  # Old version
  
    with engine.connect() as conn:
        conn.execute("INSERT INTO users (email) VALUES ('test@example.com')")
        conn.commit()
  
    # Run migration
    command.upgrade(alembic_cfg, "head")
  
    # Verify data still exists
    with engine.connect() as conn:
        result = conn.execute("SELECT * FROM users WHERE email = 'test@example.com'")
        assert result.fetchone() is not None

# Production migration checklist:
"""
1. ✅ Test migration in staging
2. ✅ Backup production database
3. ✅ Run migration during low-traffic window
4. ✅ Monitor application after migration
5. ✅ Have rollback plan ready
6. ✅ Test rollback in staging first
"""
```

**Why This Matters:** Failed migrations can cause downtime and data loss in production.

---

### Interview Questions

**Question 1: How would you test a service that depends on an external API?**

**Difficulty:** Mid-Level

**SDLC Relevance:** Integration Testing

**Answer:**

```python
"""
TESTING EXTERNAL API DEPENDENCIES

Strategy: Use different approaches based on test type

1. UNIT TESTS: Mock the API
   - Fast, isolated
   - No network calls
   - Full control over responses

2. INTEGRATION TESTS: Record/replay (VCR.py)
   - Real API responses
   - Reproducible
   - Works offline

3. E2E TESTS: Use test API instance
   - Real integration
   - Requires test account
   - Slower
"""

# Example: Weather API service

import requests

class WeatherService:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.weather.com/v1"
  
    def get_forecast(self, city):
        response = requests.get(
            f"{self.base_url}/forecast",
            params={"city": city, "api_key": self.api_key}
        )
        response.raise_for_status()
        return response.json()

# Approach 1: Unit test with mock
from unittest.mock import patch, Mock

@patch('requests.get')
def test_get_forecast_mock(mock_get):
    """Unit test: Mock external API"""
    # Setup mock response
    mock_response = Mock()
    mock_response.json.return_value = {
        "city": "London",
        "temperature": 15,
        "conditions": "Cloudy"
    }
    mock_response.raise_for_status = Mock()
    mock_get.return_value = mock_response
  
    # Test
    service = WeatherService(api_key="test-key")
    forecast = service.get_forecast("London")
  
    # Verify
    assert forecast["temperature"] == 15
    assert forecast["conditions"] == "Cloudy"
    mock_get.assert_called_once()

# Approach 2: Integration test with VCR.py
import vcr

@vcr.use_cassette('fixtures/weather_london.yaml')
def test_get_forecast_vcr():
    """Integration test: Record/replay real API"""
    service = WeatherService(api_key="real-api-key")
    forecast = service.get_forecast("London")
  
    # First run: Makes real API call, records response
    # Later runs: Replays from cassette (fast!)
    assert "temperature" in forecast
    assert forecast["city"] == "London"

# Approach 3: E2E test with test API
def test_get_forecast_real_api():
    """E2E test: Real API call"""
    service = WeatherService(api_key=os.getenv("TEST_API_KEY"))
    forecast = service.get_forecast("London")
  
    # Real API call (slow, requires network)
    assert "temperature" in forecast
    assert isinstance(forecast["temperature"], (int, float))

# Approach 4: responses library
import responses

@responses.activate
def test_get_forecast_responses():
    """Unit test: responses library"""
    responses.add(
        responses.GET,
        "https://api.weather.com/v1/forecast",
        json={"city": "London", "temperature": 15},
        status=200
    )
  
    service = WeatherService(api_key="test-key")
    forecast = service.get_forecast("London")
  
    assert forecast["temperature"] == 15

# Test error cases
@responses.activate
def test_api_error_handling():
    """Test API error handling"""
    responses.add(
        responses.GET,
        "https://api.weather.com/v1/forecast",
        json={"error": "Invalid API key"},
        status=401
    )
  
    service = WeatherService(api_key="invalid-key")
  
    with pytest.raises(requests.exceptions.HTTPError):
        service.get_forecast("London")

# Best practices:
# ✅ Unit tests: Mock everything (fast CI)
# ✅ Integration: Use VCR.py (reproducible)
# ✅ E2E: Real API in staging (before production)
# ✅ Handle errors (timeouts, rate limits, auth)
# ✅ Test retry logic
```

**Key Points:**

- Use mocks for unit tests (fast, isolated)
- Use VCR.py for integration tests (reproducible)
- Use real API for E2E tests (staging only)
- Test error cases (timeout, auth, rate limit)
- Consider using test API instances

---

## 4.4 Advanced Testing

**SDLC Phase:** Testing, Quality Assurance

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development
- [X] Testing (advanced)
- [ ] Deployment
- [X] Maintenance (quality)

### Property-Based Testing

```python
"""
PROPERTY-BASED TESTING: Test with generated data

Instead of specific examples, define properties that should always hold.
Hypothesis generates test cases automatically.

Install: pip install hypothesis
"""

from hypothesis import given, strategies as st
import pytest

# Example: Testing list operations

# Traditional example-based test
def test_reverse_twice():
    """Test specific example"""
    lst = [1, 2, 3]
    assert list(reversed(list(reversed(lst)))) == lst

# Property-based test (tests hundreds of examples!)
@given(st.lists(st.integers()))
def test_reverse_twice_property(lst):
    """Property: reversing twice returns original"""
    assert list(reversed(list(reversed(lst)))) == lst

# Hypothesis generates:
# - Empty list: []
# - Single element: [0]
# - Multiple elements: [1, 2, 3]
# - Negative numbers: [-5, -2]
# - Large numbers: [999999]
# - Many elements: [1, 2, 3, ..., 100]

# More examples

@given(st.integers(), st.integers())
def test_addition_commutative(a, b):
    """Property: a + b == b + a"""
    assert a + b == b + a

@given(st.text())
def test_string_length(s):
    """Property: len(s) >= 0"""
    assert len(s) >= 0

@given(st.lists(st.integers()))
def test_sort_idempotent(lst):
    """Property: sorting twice == sorting once"""
    assert sorted(sorted(lst)) == sorted(lst)

# Custom strategies
@given(st.emails())
def test_email_validation(email):
    """Test email validator with generated emails"""
    assert validate_email(email) == True

@given(st.integers(min_value=0, max_value=100))
def test_discount_range(discount):
    """Test discount in valid range"""
    price = 100
    discounted = apply_discount(price, discount)
    assert 0 <= discounted <= price

# Composite strategies
User = namedtuple('User', ['name', 'email', 'age'])

users = st.builds(
    User,
    name=st.text(min_size=1, max_size=50),
    email=st.emails(),
    age=st.integers(min_value=18, max_value=120)
)

@given(users)
def test_user_creation(user):
    """Test user validation"""
    assert len(user.name) > 0
    assert '@' in user.email
    assert 18 <= user.age <= 120

# Stateful testing
from hypothesis.stateful import RuleBasedStateMachine, rule

class ShoppingCartMachine(RuleBasedStateMachine):
    """Test shopping cart with state"""
  
    def __init__(self):
        super().__init__()
        self.cart = ShoppingCart()
        self.items = {}
  
    @rule(product_id=st.text(), price=st.floats(min_value=0, max_value=1000))
    def add_item(self, product_id, price):
        """Add item to cart"""
        self.cart.add_item(product_id, price)
        self.items[product_id] = price
  
    @rule(product_id=st.text())
    def remove_item(self, product_id):
        """Remove item from cart"""
        if product_id in self.items:
            self.cart.remove_item(product_id)
            del self.items[product_id]
  
    @rule()
    def check_total(self):
        """Verify total is correct"""
        expected = sum(self.items.values())
        assert abs(self.cart.total() - expected) < 0.01

TestShoppingCart = ShoppingCartMachine.TestCase

# Benefits:
# ✅ Finds edge cases automatically
# ✅ Tests hundreds of examples
# ✅ Shrinks failures to minimal example
# ✅ Catches bugs example-based tests miss
```

### Performance Testing

```python
"""
PERFORMANCE TESTING

1. Microbenchmarks (pytest-benchmark)
2. Load testing (locust)
3. Performance regression testing
"""

# Install: pip install pytest-benchmark

# Microbenchmarks
def test_list_comprehension(benchmark):
    """Benchmark list comprehension"""
    result = benchmark(lambda: [i**2 for i in range(1000)])
    assert len(result) == 1000

def test_generator_expression(benchmark):
    """Benchmark generator expression"""
    result = benchmark(lambda: list(i**2 for i in range(1000)))
    assert len(result) == 1000

# Compare algorithms
def bubble_sort(lst):
    n = len(lst)
    for i in range(n):
        for j in range(0, n-i-1):
            if lst[j] > lst[j+1]:
                lst[j], lst[j+1] = lst[j+1], lst[j]
    return lst

def quick_sort(lst):
    if len(lst) <= 1:
        return lst
    pivot = lst[len(lst) // 2]
    left = [x for x in lst if x < pivot]
    middle = [x for x in lst if x == pivot]
    right = [x for x in lst if x > pivot]
    return quick_sort(left) + middle + quick_sort(right)

def test_bubble_sort_performance(benchmark):
    """Benchmark bubble sort"""
    data = list(range(100, 0, -1))
    result = benchmark(bubble_sort, data.copy())

def test_quick_sort_performance(benchmark):
    """Benchmark quick sort"""
    data = list(range(100, 0, -1))
    result = benchmark(quick_sort, data.copy())

# Run: pytest --benchmark-only
# Output:
"""
Name                          Min       Max      Mean
test_bubble_sort_performance  2.5ms     3.0ms    2.7ms
test_quick_sort_performance   0.1ms     0.2ms    0.15ms
"""

# Load Testing with Locust
"""
# locustfile.py

from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 5)  # Wait 1-5s between requests
  
    @task(3)  # Weight: 3x more likely than other tasks
    def view_products(self):
        self.client.get("/products")
  
    @task(1)
    def view_product(self):
        self.client.get("/products/123")
  
    @task(2)
    def search(self):
        self.client.get("/search?q=laptop")
  
    def on_start(self):
        # Login before starting
        self.client.post("/auth/login", json={
            "email": "test@example.com",
            "password": "password"
        })

# Run: locust -f locustfile.py
# Open: http://localhost:8089
# Configure: Users, spawn rate
"""

# Performance regression testing
def test_api_response_time(benchmark):
    """Ensure API responds within 100ms"""
    result = benchmark(lambda: requests.get("http://localhost:8000/api/users"))
  
    # Assert performance requirement
    assert benchmark.stats.mean < 0.1  # 100ms

# Memory profiling
import tracemalloc

def test_memory_usage():
    """Test memory usage"""
    tracemalloc.start()
  
    # Run code
    data = [i**2 for i in range(1000000)]
  
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
  
    # Assert memory usage
    assert peak < 50 * 1024 * 1024  # Less than 50MB
```

### Security Testing

```python
"""
SECURITY TESTING

1. Static analysis (Bandit)
2. Dependency scanning (Safety)
3. Security-specific tests
"""

# Install: pip install bandit safety

# Bandit: Find common security issues
# Run: bandit -r myproject/

# Example issues Bandit finds:
"""
# ❌ Hardcoded password
password = "admin123"

# ❌ SQL injection vulnerability
query = f"SELECT * FROM users WHERE id = {user_id}"

# ❌ Weak cryptographic algorithm
import md5
hash = md5.md5(password).hexdigest()

# ❌ Insecure random
import random
token = random.randint(1000, 9999)
"""

# ✅ Fixed versions:
"""
# ✅ Environment variable
password = os.getenv("DB_PASSWORD")

# ✅ Parameterized query
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))

# ✅ Strong hash
import hashlib
hash = hashlib.sha256(password.encode()).hexdigest()

# ✅ Cryptographically secure random
import secrets
token = secrets.token_urlsafe(32)
"""

# Safety: Check dependencies for vulnerabilities
# Run: safety check

# Test authentication
def test_login_requires_credentials(client):
    """Test login without credentials fails"""
    response = client.post("/auth/login", json={})
    assert response.status_code == 400

def test_weak_password_rejected(client):
    """Test weak passwords are rejected"""
    response = client.post("/auth/register", json={
        "email": "test@example.com",
        "password": "123"  # Too weak
    })
    assert response.status_code == 400
    assert "password" in response.json()["errors"]

def test_password_hashing(db_session):
    """Test passwords are hashed"""
    user = User(email="test@example.com", password="SecurePass123!")
    db_session.add(user)
    db_session.commit()
  
    # Password should be hashed, not plaintext
    assert user.password_hash != "SecurePass123!"
    assert len(user.password_hash) > 50  # Hashed length

# Test authorization
def test_user_cannot_access_others_data(client):
    """Test users can't access other users' data"""
    # Login as user1
    user1_token = login(client, "user1@example.com")
  
    # Try to access user2's data
    response = client.get(
        "/users/user2_id",
        headers={"Authorization": f"Bearer {user1_token}"}
    )
    assert response.status_code == 403  # Forbidden

# Test rate limiting
def test_brute_force_protection(client):
    """Test login is rate limited"""
    for i in range(10):
        response = client.post("/auth/login", json={
            "email": "test@example.com",
            "password": "wrong"
        })
  
    # Should be rate limited after multiple failures
    assert response.status_code == 429  # Too Many Requests

# Test input validation
def test_sql_injection_prevention():
    """Test SQL injection is prevented"""
    malicious_input = "1; DROP TABLE users; --"
  
    # Should be safely handled
    user = db_session.query(User).filter_by(id=malicious_input).first()
  
    # Table should still exist
    assert db_session.query(User).count() >= 0

def test_xss_prevention(client):
    """Test XSS is prevented"""
    malicious_script = "<script>alert('XSS')</script>"
  
    response = client.post("/comments", json={
        "text": malicious_script
    })
  
    # Script should be escaped in response
    comment = response.json()
    assert "<script>" not in comment["text"]
    assert "<script>" in comment["text"]  # Escaped
```

### Snapshot Testing

```python
"""
SNAPSHOT TESTING: Compare output against approved baseline

Useful for:
- Complex data structures
- HTML/JSON output
- Rendered templates
"""

# Install: pip install pytest-snapshot

def test_user_serialization(snapshot):
    """Test user serialization matches snapshot"""
    user = User(
        id="123",
        email="test@example.com",
        name="Test User",
        created_at=datetime(2025, 1, 1)
    )
  
    serialized = user.to_dict()
  
    # First run: Creates snapshot
    # Future runs: Compares against snapshot
    snapshot.assert_match(serialized, 'user_snapshot.json')

# Snapshot file (user_snapshot.json):
"""
{
  "id": "123",
  "email": "test@example.com",
  "name": "Test User",
  "created_at": "2025-01-01T00:00:00"
}
"""

# Update snapshots: pytest --snapshot-update

def test_email_template(snapshot):
    """Test email template rendering"""
    email = render_email_template('welcome.html', {
        'user_name': 'Alice',
        'verification_link': 'https://example.com/verify/abc123'
    })
  
    snapshot.assert_match(email, 'welcome_email.html')

# Benefits:
# ✅ Catch unexpected changes
# ✅ Easy to review changes (diff)
# ✅ Works with complex structures
# ✅ Good for refactoring
```

### Frequently Asked Questions

**Q1: When should I use property-based testing?**

**A:**

```python
"""
WHEN TO USE PROPERTY-BASED TESTING

Use property-based testing for:
✅ Pure functions (no side effects)
✅ Data transformations
✅ Algorithms with invariants
✅ Parsers and serializers
✅ Validation logic

Don't use for:
❌ UI interactions
❌ Complex external dependencies
❌ Time-dependent code
❌ Random behavior
"""

# ✅ GOOD: Testing sort function
@given(st.lists(st.integers()))
def test_sort_properties(lst):
    """Properties of sorting"""
    sorted_lst = sorted(lst)
  
    # Property 1: Result is sorted
    for i in range(len(sorted_lst) - 1):
        assert sorted_lst[i] <= sorted_lst[i + 1]
  
    # Property 2: Same elements
    assert sorted(sorted_lst) == sorted(lst)
  
    # Property 3: Same length
    assert len(sorted_lst) == len(lst)

# ✅ GOOD: Testing serialization
@given(st.dictionaries(st.text(), st.integers()))
def test_json_roundtrip(data):
    """JSON serialization round-trip"""
    serialized = json.dumps(data)
    deserialized = json.loads(serialized)
    assert deserialized == data

# ❌ BAD: Testing with external dependencies
@given(st.text())
def test_api_call(url):
    """Don't do this - external dependency!"""
    response = requests.get(url)  # Real API call!
    assert response.status_code == 200

# Hypothesis is great for finding edge cases:
# - Empty inputs
# - Very large inputs
# - Special characters
# - Boundary values
# - Combinations you didn't think of
```

**Why This Matters:** Property-based testing finds bugs that example-based tests miss.

---

## 4.5 Code Quality & Standards

**SDLC Phase:** Development, Code Review

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development
- [ ] Testing
- [ ] Deployment
- [X] Maintenance (code quality)

### Code Formatting

```python
"""
CODE FORMATTING: Consistent style across codebase

Tools:
- Black: Opinionated auto-formatter
- isort: Sort imports
- autopep8: PEP 8 compliance
"""

# Install: pip install black isort

# Black: The uncompromising formatter
# Run: black myproject/

# Before Black:
def very_long_function(argument1,argument2,argument3,argument4,argument5):
    return argument1+argument2+argument3+argument4+argument5

# After Black:
def very_long_function(
    argument1, argument2, argument3, argument4, argument5
):
    return (
        argument1
        + argument2
        + argument3
        + argument4
        + argument5
    )

# pyproject.toml configuration:
"""
[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
exclude = '''
/(
    \.git
  | \.venv
  | build
  | dist
)/
'''
"""

# isort: Sort imports
# Run: isort myproject/

# Before isort:
import os
from myproject import utils
import sys
from typing import List
import django

# After isort:
import os
import sys

import django
from typing import List

from myproject import utils

# .isort.cfg configuration:
"""
[settings]
profile = black
line_length = 88
multi_line_output = 3
include_trailing_comma = True
force_grid_wrap = 0
use_parentheses = True
"""

# Pre-commit hooks: Auto-format on commit
# Install: pip install pre-commit

# .pre-commit-config.yaml:
"""
repos:
  - repo: https://github.com/psf/black
    rev: 24.1.1
    hooks:
      - id: black
  
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort
  
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
"""

# Setup: pre-commit install
# Now runs automatically on git commit!

# Benefits:
# ✅ Consistent code style
# ✅ No style debates in code reviews
# ✅ Automatic formatting
# ✅ Reduces diffs (less noise)
```

### Linting

```python
"""
LINTING: Static code analysis for issues

Tools:
- Pylint: Comprehensive linter
- Flake8: Fast, modular
- Ruff: Extremely fast (Rust-based)
"""

# Install: pip install pylint flake8 ruff

# Flake8: Check code quality
# Run: flake8 myproject/

# Example issues:
"""
myproject/utils.py:10:1: E302 expected 2 blank lines, found 1
myproject/utils.py:15:80: E501 line too long (91 > 79 characters)
myproject/utils.py:20:1: F401 'os' imported but unused
"""

# .flake8 configuration:
"""
[flake8]
max-line-length = 88
exclude = .git,__pycache__,venv
ignore = E203, W503
per-file-ignores =
    __init__.py: F401
"""

# Pylint: More comprehensive
# Run: pylint myproject/

# Example output:
"""
************* Module myproject.utils
myproject/utils.py:1:0: C0114: Missing module docstring (missing-module-docstring)
myproject/utils.py:5:0: C0116: Missing function docstring (missing-function-docstring)
myproject/utils.py:10:4: W0612: Unused variable 'x' (unused-variable)

Your code has been rated at 7.50/10
"""

# .pylintrc configuration:
"""
[MASTER]
ignore=tests,migrations

[MESSAGES CONTROL]
disable=C0111,  # missing-docstring
        R0903   # too-few-public-methods

[FORMAT]
max-line-length=88

[DESIGN]
max-args=7
max-locals=15
"""

# Ruff: Fast linter (100x faster than Flake8!)
# Run: ruff check myproject/

# pyproject.toml:
"""
[tool.ruff]
line-length = 88
select = ["E", "F", "W", "I", "N"]
ignore = ["E501"]
"""

# Custom linting rules
# Example: Enforce logging best practices
"""
# ❌ BAD
print("Error occurred")  # Don't use print in production

# ✅ GOOD
logger.error("Error occurred")
"""

# Benefits:
# ✅ Catch bugs before runtime
# ✅ Enforce code quality standards
# ✅ Consistent code style
# ✅ Automatic in CI/CD
```

### Type Checking

```python
"""
TYPE CHECKING: Static type verification

Tools:
- mypy: Standard type checker
- pyright: Microsoft's type checker
- pyre: Facebook's type checker
"""

# Install: pip install mypy

# Add type hints
def greet(name: str) -> str:
    return f"Hello, {name}!"

def calculate_total(prices: list[float]) -> float:
    return sum(prices)

# Run: mypy myproject/

# Example errors:
"""
myproject/utils.py:10: error: Argument 1 to "greet" has incompatible type "int"; expected "str"
myproject/utils.py:15: error: Incompatible return value type (got "None", expected "str")
"""

# mypy.ini configuration:
"""
[mypy]
python_version = 3.11
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True

[mypy-tests.*]
disallow_untyped_defs = False

[mypy-third_party_lib.*]
ignore_missing_imports = True
"""

# Gradual typing adoption
# Start with critical modules:
"""
# mypy: strict  # Enable strict mode for this file
def important_function(data: dict[str, int]) -> int:
    return sum(data.values())
"""

# Type stubs for third-party libraries
# Create .pyi file for libraries without types
"""
# requests.pyi
def get(url: str, **kwargs) -> Response: ...

class Response:
    status_code: int
    text: str
    def json(self) -> dict: ...
"""

# Benefits:
# ✅ Catch type errors before runtime
# ✅ Better IDE support
# ✅ Self-documenting code
# ✅ Refactoring confidence
```

### Static Analysis

```python
"""
STATIC ANALYSIS: Find issues without running code

Tools:
- Bandit: Security issues
- Radon: Complexity metrics
- vulture: Dead code detection
"""

# Install: pip install bandit radon vulture

# Bandit: Security linting
# Run: bandit -r myproject/

# Example findings:
"""
>> Issue: [B105:hardcoded_password_string] Possible hardcoded password: 'admin123'
   Severity: Low   Confidence: Medium
   Location: myproject/config.py:5

>> Issue: [B608:hardcoded_sql_expressions] Possible SQL injection vector
   Severity: Medium   Confidence: Medium
   Location: myproject/database.py:15
"""

# Radon: Complexity metrics
# Run: radon cc myproject/ -a

# Output:
"""
myproject/utils.py
    F 10:0 calculate_discount - A (2)
    F 25:0 complex_function - C (15)  # Too complex!
  
Average complexity: B (7)
"""

# Cyclomatic complexity grades:
"""
A (1-5):   Simple, low risk
B (6-10):  Moderate, acceptable
C (11-20): Complex, consider refactoring
D (21-50): Very complex, high risk
F (50+):   Extremely complex, must refactor
"""

# Refactor complex code:
# ❌ BAD: High complexity
def complex_function(x, y, z):
    if x > 0:
        if y > 0:
            if z > 0:
                if x + y > 10:
                    if z < 5:
                        return "case1"
                    else:
                        return "case2"
                else:
                    return "case3"
            else:
                return "case4"
        else:
            return "case5"
    else:
        return "case6"

# ✅ GOOD: Lower complexity
def simple_function(x, y, z):
    if x <= 0:
        return "case6"
    if y <= 0:
        return "case5"
    if z <= 0:
        return "case4"
    if x + y <= 10:
        return "case3"
    if z < 5:
        return "case1"
    return "case2"

# Vulture: Find dead code
# Run: vulture myproject/

# Example output:
"""
myproject/utils.py:10: unused function 'old_function' (60% confidence)
myproject/config.py:5: unused variable 'DEBUG_MODE' (60% confidence)
"""

# Configure in pyproject.toml:
"""
[tool.vulture]
min_confidence = 80
paths = ["myproject"]
ignore_names = ["test_*", "setUp", "tearDown"]
"""
```

### Frequently Asked Questions

**Q1: Should I use Black even though it doesn't follow all PEP 8 rules?**

**A:**

```python
"""
BLACK vs PEP 8

Black deliberately differs from PEP 8 in some cases:
- Line length: 88 chars (PEP 8: 79)
- String quotes: prefers double quotes
- Some formatting choices

Recommendation: Use Black

Reasons:
✅ Saves time (no formatting debates)
✅ Consistent across team
✅ Auto-formatting
✅ Widely adopted
✅ Black's style is still readable

Most teams prefer consistency over strict PEP 8 compliance.
"""

# Configure to match team preferences:
"""
[tool.black]
line-length = 100  # If team prefers longer lines
skip-string-normalization = true  # Keep quote style
"""

# Use with Flake8:
"""
[flake8]
max-line-length = 88  # Match Black
extend-ignore = E203, W503  # Black-compatible
"""
```

**Why This Matters:** Formatting tools save time and reduce bikeshedding in code reviews.

---

### Key Takeaways

**Testing & Quality Assurance:**

- **Testing Pyramid**: 70% unit, 20% integration, 10% E2E
- **TDD**: Red-Green-Refactor cycle
- **unittest**: Python's built-in testing framework
- **pytest**: Modern, Pythonic testing with fixtures
- **Mocking**: Replace dependencies (Mock, Stub, Spy, Fake, Dummy)
- **Coverage**: Aim for 80%+, focus on high-value code
- **Integration**: Test databases, APIs, external services
- **Property-based**: Hypothesis for edge case discovery
- **Performance**: pytest-benchmark, locust for load testing
- **Security**: Bandit, Safety, authentication/authorization tests
- **Code Quality**: Black, isort, Flake8, mypy, pre-commit hooks

**Best Practices:**

- Write tests first (TDD) for clear requirements
- Keep tests fast and isolated
- Use fixtures for reusable test setup
- Mock external dependencies in unit tests
- Test both happy path and error cases
- Measure coverage but focus on quality
- Automate formatting with pre-commit hooks
- Run linters in CI/CD
- Use type hints and mypy for safety

**Tools Summary:**

- **Testing**: pytest, unittest, Hypothesis
- **Mocking**: unittest.mock, responses, VCR.py
- **Coverage**: coverage.py, pytest-cov
- **Formatting**: Black, isort
- **Linting**: Flake8, Pylint, Ruff
- **Type Checking**: mypy, pyright
- **Security**: Bandit, Safety
- **Performance**: pytest-benchmark, locust

---

## Part 4 Complete!

Congratulations! You've completed **Part 4: Testing & Quality Assurance**. You now have comprehensive knowledge of:

✅ **Section 4.1**: Testing Fundamentals
✅ **Section 4.2**: Unit Testing
✅ **Section 4.3**: Integration Testing
✅ **Section 4.4**: Advanced Testing
✅ **Section 4.5**: Code Quality & Standards

**What You've Learned:**

- Testing pyramid and strategy
- Test-Driven Development (TDD)
- unittest and pytest frameworks
- Mocking and test doubles
- Code coverage measurement
- Database and API integration testing
- Property-based testing with Hypothesis
- Performance and security testing
- Code formatting (Black, isort)
- Linting (Flake8, Pylint, Ruff)
- Type checking (mypy)
- Static analysis (Bandit, Radon)

**Next Steps:**

Continue with **Part 5: Deployment & DevOps** to learn:

- Containerization (Docker)
- Orchestration (Kubernetes)
- CI/CD pipelines (GitHub Actions, GitLab CI)
- Monitoring and logging
- Infrastructure as Code
- Cloud deployment (AWS, GCP, Azure)

Keep writing high-quality, well-tested Python code! 🧪
