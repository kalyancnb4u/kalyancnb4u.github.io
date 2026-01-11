---
title: "Python DevSecOps - Part 3: Testing and Quality Assurance"
date: 2025-01-04 00:00:00 +0530
categories: [Python, DevSecOps]
tags: [Python, DevSecOps, Testing, Pytest, TDD, Mocking, Coverage, Quality-assurance, Automation]
---

# Complete Python Mastery for DevSecOps Part 3: Testing and Quality Assurance

## Introduction

Welcome to Part 3 of the Python Mastery for DevSecOps series. Testing is not optional in DevSecOps - it's the foundation of reliable, secure infrastructure automation.

**Why Testing Is Critical in DevSecOps:**

In DevSecOps, your code manages production infrastructure. A single bug can:
- Take down production services (availability impact)
- Expose security vulnerabilities (security impact)
- Corrupt data or configurations (integrity impact)
- Cost thousands in cloud resources (financial impact)
- Violate compliance requirements (legal impact)

**The Cost of Not Testing:**

- **Knight Capital Group (2012)**: Untested deployment code caused $440 million loss in 45 minutes
- **GitLab (2017)**: Inadequate backup testing led to 6-hour database outage
- **AWS S3 Outage (2017)**: Typo in command (inadequate testing) took down major internet services

**What You'll Learn:**

This part covers production-grade testing practices:
- Test-Driven Development (TDD) for infrastructure code
- pytest framework mastery
- Test doubles and mocking strategies
- Test coverage (and its limitations)
- Property-based testing for edge cases
- Integration and end-to-end testing
- Performance and security testing
- CI/CD integration

**Who This Is For:**

- DevSecOps engineers building reliable automation
- Developers who need to test infrastructure code
- SREs ensuring system reliability
- Anyone responsible for production deployments
- Teams implementing CI/CD pipelines

---

## 3.1 Testing Fundamentals

### Why Test in DevSecOps?

**Traditional Software Testing:**
- Find bugs before users do
- Verify features work correctly
- Enable refactoring safely

**DevSecOps Testing (Additional Concerns):**
- **Prevent production outages** - Infrastructure code manages critical systems
- **Validate security controls** - Ensure security measures work
- **Verify idempotency** - Running same script twice shouldn't break things
- **Test failure scenarios** - What happens when AWS API fails?
- **Validate rollback procedures** - Can you undo deployments?
- **Ensure compliance** - Meet regulatory requirements

### Test Types Pyramid

The testing pyramid shows how to balance different test types:

```
           ╱╲
          ╱  ╲         E2E Tests (Slow, Expensive, Brittle)
         ╱    ╲        - Full system tests
        ╱──────╲       - Real cloud resources
       ╱        ╲      - Manual verification
      ╱          ╲     
     ╱            ╲    Integration Tests (Medium)
    ╱──────────────╲   - Multiple components
   ╱                ╲  - Real databases/APIs
  ╱                  ╲ - Mocked external services
 ╱────────────────────╲
╱                      ╲ Unit Tests (Fast, Cheap, Reliable)
────────────────────────  - Single function/class
                          - Mocked dependencies
                          - Run in milliseconds
```

**Ideal Distribution:**
- **70% Unit Tests** - Fast, focused, reliable
- **20% Integration Tests** - Components working together
- **10% E2E Tests** - Full system validation

**In DevSecOps:**
```python
# Unit test: Test function in isolation
def test_parse_config():
    """Test config parsing logic alone."""
    config_yaml = """
    environment: production
    replicas: 3
    """
    result = parse_config(config_yaml)
    assert result['environment'] == 'production'
    assert result['replicas'] == 3

# Integration test: Test with real dependencies
def test_deploy_to_kubernetes():
    """Test deployment with real k8s cluster (kind/minikube)."""
    k8s_client = get_test_cluster_client()
    deploy_result = deploy_app(k8s_client, test_config)
    
    # Verify deployment exists
    deployment = k8s_client.read_namespaced_deployment(
        'test-app', 'default'
    )
    assert deployment.spec.replicas == 3

# E2E test: Full workflow
def test_full_deployment_pipeline():
    """Test complete deployment pipeline end-to-end."""
    # Build Docker image
    # Push to registry
    # Deploy to staging
    # Run smoke tests
    # Promote to production
    # Verify production health
    pass
```

### Test-Driven Development (TDD)

TDD is especially powerful for infrastructure code because it forces you to think about:
- What should happen (happy path)
- What could go wrong (error cases)
- Edge cases (empty lists, missing fields)

**TDD Cycle (Red-Green-Refactor):**

```
1. RED: Write failing test
   ↓
2. GREEN: Write minimal code to pass
   ↓
3. REFACTOR: Improve code without changing behavior
   ↓
   Repeat
```

**TDD Example: Server Health Checker**

```python
# Step 1: RED - Write failing test first
import pytest
from health_checker import check_server_health

def test_healthy_server_returns_true():
    """Test that healthy server returns True."""
    # This function doesn't exist yet!
    result = check_server_health('http://localhost:8080/health')
    assert result is True  # FAILS: NameError

# Step 2: GREEN - Minimal implementation
def check_server_health(url: str) -> bool:
    """Check if server is healthy."""
    return True  # Simplest code to pass test

# Test passes! But too simple...

# Step 1 (again): Write more specific test
def test_unhealthy_server_returns_false():
    """Test that unhealthy server returns False."""
    # Mock server returning 500
    with patch('requests.get') as mock_get:
        mock_get.return_value.status_code = 500
        result = check_server_health('http://localhost:8080/health')
        assert result is False  # FAILS: always returns True

# Step 2 (again): Real implementation
import requests

def check_server_health(url: str) -> bool:
    """Check if server is healthy."""
    try:
        response = requests.get(url, timeout=5)
        return 200 <= response.status_code < 300
    except requests.RequestException:
        return False

# Tests pass! ✅

# Step 3: REFACTOR
def test_timeout_returns_false():
    """Test that timeout returns False."""
    with patch('requests.get') as mock_get:
        mock_get.side_effect = requests.Timeout()
        result = check_server_health('http://localhost:8080/health')
        assert result is False

def test_connection_error_returns_false():
    """Test that connection error returns False."""
    with patch('requests.get') as mock_get:
        mock_get.side_effect = requests.ConnectionError()
        result = check_server_health('http://localhost:8080/health')
        assert result is False

# All tests pass! Code handles edge cases.
```

**TDD Benefits in DevSecOps:**
- **Prevents regressions** - Tests catch breaking changes
- **Documents behavior** - Tests show how code should work
- **Enables refactoring** - Change implementation safely
- **Forces error handling** - Must think about failures upfront
- **Improves design** - Testable code is better structured

### Testing Philosophy

**FIRST Principles:**

- **F**ast - Tests should run in milliseconds
- **I**solated - Tests don't depend on each other
- **R**epeatable - Same result every time
- **S**elf-validating - Pass or fail, no manual checks
- **T**imely - Write tests before/with code, not after

```python
# ❌ BAD: Slow test (talks to real AWS)
def test_provision_server():
    """This takes 2 minutes to run!"""
    import boto3
    ec2 = boto3.client('ec2', region_name='us-east-1')
    
    # Actually provision EC2 instance
    response = ec2.run_instances(
        ImageId='ami-12345',
        InstanceType='t2.micro',
        MinCount=1, MaxCount=1
    )
    
    instance_id = response['Instances'][0]['InstanceId']
    
    # Wait for running state
    waiter = ec2.get_waiter('instance_running')
    waiter.wait(InstanceIds=[instance_id])
    
    # Cleanup
    ec2.terminate_instances(InstanceIds=[instance_id])
    
    assert True  # Test passed but took forever

# ✅ GOOD: Fast test (mocked)
from unittest.mock import patch, MagicMock

def test_provision_server_calls_correct_api():
    """Test runs in milliseconds."""
    with patch('boto3.client') as mock_boto:
        mock_ec2 = MagicMock()
        mock_boto.return_value = mock_ec2
        mock_ec2.run_instances.return_value = {
            'Instances': [{'InstanceId': 'i-12345'}]
        }
        
        from server_provisioner import provision_server
        instance_id = provision_server('t2.micro', 'ami-12345')
        
        # Verify correct API call
        mock_ec2.run_instances.assert_called_once_with(
            ImageId='ami-12345',
            InstanceType='t2.micro',
            MinCount=1,
            MaxCount=1
        )
        
        assert instance_id == 'i-12345'
```

**When to Use Real Services vs Mocks:**

```python
# Unit Tests: Always mock external services
def test_parse_deployment_config():
    """Test config parsing - no external dependencies."""
    config = parse_yaml("""
        app: my-app
        replicas: 3
    """)
    assert config['replicas'] == 3

# Integration Tests: Use real dependencies when feasible
def test_database_connection():
    """Test with real PostgreSQL (Docker container)."""
    # Use testcontainers or docker-compose for integration tests
    from testcontainers.postgres import PostgresContainer
    
    with PostgresContainer("postgres:14") as postgres:
        conn = connect_to_database(postgres.get_connection_url())
        assert conn.is_connected()

# E2E Tests: Real environment (staging)
@pytest.mark.e2e
@pytest.mark.slow
def test_full_deployment_workflow():
    """Deploy to real staging environment."""
    # Use real cloud resources
    result = deploy_to_staging('my-app', 'v1.0.0')
    assert result['status'] == 'success'
```

### Test Organization

**Directory Structure:**

```
project/
├── src/
│   └── infrastructure/
│       ├── __init__.py
│       ├── provisioner.py
│       ├── deployer.py
│       └── monitor.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py              # Shared fixtures
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_provisioner.py
│   │   ├── test_deployer.py
│   │   └── test_monitor.py
│   ├── integration/
│   │   ├── __init__.py
│   │   ├── test_aws_integration.py
│   │   └── test_k8s_integration.py
│   └── e2e/
│       ├── __init__.py
│       └── test_deployment_pipeline.py
├── pytest.ini                   # pytest configuration
└── requirements-test.txt        # Test dependencies
```

**Test Naming Conventions:**

```python
# ✅ GOOD: Descriptive test names
def test_deploy_with_zero_replicas_raises_error():
    """Test that deploying with 0 replicas raises ValueError."""
    with pytest.raises(ValueError, match="Replicas must be > 0"):
        deploy_app(replicas=0)

def test_health_check_retries_on_timeout():
    """Test that health checker retries after timeout."""
    with patch('requests.get', side_effect=[
        requests.Timeout(),
        requests.Timeout(),
        MagicMock(status_code=200)  # Success on third attempt
    ]):
        result = check_health_with_retry(max_retries=3)
        assert result is True

def test_config_parser_handles_missing_optional_fields():
    """Test that parser sets defaults for optional fields."""
    config = parse_config("""
        app: my-app
    """)  # Missing optional 'replicas' field
    
    assert config['replicas'] == 1  # Default value

# ❌ BAD: Vague test names
def test_deploy():
    """What does this test?"""
    pass

def test_config():
    """What about config?"""
    pass

def test_1():
    """Meaningless name."""
    pass
```

### Arrange-Act-Assert (AAA) Pattern

Structure tests clearly:

```python
def test_server_provisioner_creates_instance_with_correct_config():
    """Test server provisioning with specific configuration."""
    
    # ARRANGE: Set up test data and mocks
    from server_provisioner import ServerProvisioner
    provisioner = ServerProvisioner(region='us-east-1')
    
    config = {
        'instance_type': 't2.micro',
        'ami_id': 'ami-12345',
        'security_groups': ['sg-12345'],
        'tags': {'Environment': 'test'}
    }
    
    with patch.object(provisioner, '_call_aws_api') as mock_aws:
        mock_aws.return_value = {'InstanceId': 'i-test-123'}
        
        # ACT: Execute the code being tested
        instance_id = provisioner.provision_server(config)
        
        # ASSERT: Verify results
        assert instance_id == 'i-test-123'
        
        mock_aws.assert_called_once_with(
            'RunInstances',
            ImageId='ami-12345',
            InstanceType='t2.micro',
            SecurityGroupIds=['sg-12345'],
            TagSpecifications=[{
                'ResourceType': 'instance',
                'Tags': [{'Key': 'Environment', 'Value': 'test'}]
            }],
            MinCount=1,
            MaxCount=1
        )
```

### Frequently Asked Questions - Testing Fundamentals

**Q1: How much test coverage should I aim for?**

**A:** **Quality over quantity.** 100% coverage doesn't mean bug-free code.

**Recommended Targets:**
- **Critical infrastructure code**: 90%+ coverage
- **Deployment scripts**: 80%+ coverage
- **Utilities/helpers**: 70%+ coverage
- **Generated code**: Exclude from coverage

```python
# Don't chase 100% coverage of trivial code
class Server:
    def __repr__(self):
        return f"Server({self.hostname})"  # Don't need test for this

# DO test critical logic thoroughly
def deploy_to_production(config):
    """Critical function - test ALL paths."""
    if not validate_config(config):
        raise ValueError("Invalid config")  # Test this
    
    if config['environment'] != 'production':
        raise ValueError("Not production")  # Test this
    
    try:
        result = execute_deployment(config)  # Test this
    except DeploymentError as e:
        rollback()  # Test this
        raise
    
    return result  # Test this
```

**Better Metrics Than Coverage:**
- **Mutation testing score** - How many injected bugs do tests catch?
- **Edge case coverage** - Are boundary conditions tested?
- **Error path coverage** - Are failure scenarios tested?

**Q2: Should I mock everything or use real dependencies?**

**A:** **It depends on the test type and dependency:**

**Always Mock:**
- External APIs (AWS, GitHub, Slack)
- Network calls
- Filesystem I/O (for unit tests)
- Time/randomness
- Expensive operations

**Use Real:**
- Your own code
- Pure functions
- Data structures
- Libraries (when cheap to use)

```python
# ✅ GOOD: Mock external API
def test_deploy_to_aws():
    with patch('boto3.client') as mock_boto:
        mock_ec2 = MagicMock()
        mock_boto.return_value = mock_ec2
        # Test logic without calling real AWS

# ✅ GOOD: Use real data structures
def test_server_inventory_add_remove():
    inventory = ServerInventory()  # Real object
    server = Server('web-01', '192.168.1.10')  # Real object
    
    inventory.add(server)
    assert len(inventory) == 1
    
    inventory.remove(server)
    assert len(inventory) == 0

# ❌ BAD: Over-mocking makes tests brittle
def test_add_numbers():
    with patch('builtins.int') as mock_int:  # Why?!
        mock_int.return_value = 5
        assert add(2, 3) == 5  # Testing mock, not code!
```

**Q3: How do I test code that deploys to production?**

**A:** **Never test against production.** Use staging/test environments.

**Strategy:**

```python
# Use environment-aware configuration
class DeploymentClient:
    def __init__(self, environment: str):
        self.environment = environment
        
        if environment == 'production':
            self.base_url = 'https://api.prod.example.com'
        elif environment == 'staging':
            self.base_url = 'https://api.staging.example.com'
        elif environment == 'test':
            self.base_url = 'http://localhost:8080'  # Local test server
        else:
            raise ValueError(f"Unknown environment: {environment}")

# Integration tests use staging
@pytest.mark.integration
def test_deploy_to_staging():
    """Test deployment against staging environment."""
    client = DeploymentClient('staging')
    result = client.deploy('test-app', 'v1.0.0')
    assert result['status'] == 'success'

# E2E tests also use staging
@pytest.mark.e2e
@pytest.mark.slow
def test_full_deployment_pipeline():
    """Run complete deployment against staging."""
    # Uses staging environment for all steps
    pass

# Production deployments go through same code path
# but with environment='production'
```

**Q4: How do I test asynchronous code?**

**A:** Use `pytest-asyncio` for async tests:

```python
import pytest
import asyncio

# Mark async test
@pytest.mark.asyncio
async def test_async_health_check():
    """Test async health check function."""
    import aiohttp
    
    async with aiohttp.ClientSession() as session:
        result = await check_health_async(session, 'http://localhost:8080')
        assert result is True

# Test with mock
@pytest.mark.asyncio
async def test_async_deploy_with_mock():
    """Test async deployment with mocked API."""
    from unittest.mock import AsyncMock
    
    mock_api = AsyncMock()
    mock_api.deploy.return_value = {'status': 'success'}
    
    result = await deploy_async(mock_api, 'my-app', 'v1.0.0')
    assert result['status'] == 'success'
    
    mock_api.deploy.assert_called_once_with('my-app', 'v1.0.0')
```

**Q5: When should I write tests - before or after code?**

**A:** **Ideally before (TDD), but pragmatically:**

**Write Tests First (TDD) When:**
- Complex logic with edge cases
- Critical infrastructure code
- Refactoring existing code
- Fixing bugs (test reproduces bug, then fix)

**Write Tests After When:**
- Prototyping/experimenting
- Very simple code
- Time-constrained situations

**Never Skip Tests:**
- Production code
- Shared libraries
- Security-critical code
- Deployment scripts

```python
# TDD Example: Write test first for critical logic
def test_calculate_instance_cost_with_reserved_instances():
    """Test cost calculation includes reserved instance discount."""
    calculator = CostCalculator()
    
    # 10 instances, $0.10/hour, 50% reserved discount
    cost = calculator.calculate_monthly_cost(
        instances=10,
        hourly_rate=0.10,
        reserved_discount=0.50
    )
    
    # (10 * 0.10 * 730 hours * 0.50) = $365
    assert cost == 365.0

# Now implement to make test pass
class CostCalculator:
    def calculate_monthly_cost(
        self,
        instances: int,
        hourly_rate: float,
        reserved_discount: float = 0.0
    ) -> float:
        """Calculate monthly infrastructure cost."""
        hours_per_month = 730  # Average
        base_cost = instances * hourly_rate * hours_per_month
        return base_cost * (1 - reserved_discount)
```

### Interview Questions - Testing Fundamentals

**Question 1: Explain the testing pyramid. Why is it shaped that way? How does it apply to DevSecOps?**

**Difficulty:** Junior

**Answer:**

The **testing pyramid** is a visual guide for test distribution across different test types.

```
         ╱╲
        ╱  ╲      E2E Tests (10%)
       ╱────╲     - Fewest tests
      ╱      ╲    - Slowest to run
     ╱        ╲   - Most expensive
    ╱──────────╲  - Most brittle
   ╱            ╲
  ╱ Integration  ╲ Integration Tests (20%)
 ╱      Tests     ╲ - Moderate number
╱──────────────────╲ - Medium speed
────────────────────
    Unit Tests (70%) Unit Tests (70%)
    - Most tests
    - Fastest to run
    - Cheapest
    - Most reliable
```

**Why This Shape?**

**Unit Tests (Base - 70%):**
- **Fast**: Run in milliseconds
- **Cheap**: No infrastructure needed
- **Reliable**: No flaky network issues
- **Focused**: Test one thing at a time

```python
# Unit test - runs in 2ms
def test_parse_yaml_config():
    yaml_str = "environment: production\nreplicas: 3"
    config = parse_config(yaml_str)
    assert config['environment'] == 'production'
    assert config['replicas'] == 3
```

**Integration Tests (Middle - 20%):**
- **Medium Speed**: Seconds to run
- **Real Components**: Database, message queue
- **Moderate Cost**: Needs test infrastructure

```python
# Integration test - runs in 2 seconds
def test_deploy_to_test_kubernetes_cluster():
    k8s = get_test_cluster()  # Real k8s (kind/minikube)
    deploy(k8s, test_config)
    
    deployment = k8s.get_deployment('test-app')
    assert deployment.ready_replicas == 3
```

**E2E Tests (Top - 10%):**
- **Slow**: Minutes to run
- **Expensive**: Real cloud resources
- **Brittle**: Many points of failure

```python
# E2E test - runs in 5 minutes
@pytest.mark.e2e
@pytest.mark.slow
def test_full_deployment_pipeline():
    # Build Docker image
    # Push to registry
    # Deploy to staging
    # Run smoke tests
    # Promote to production
    pass
```

**Why Pyramid Shape (Not Inverted)?**

❌ **Inverted Pyramid (Anti-pattern):**
- Slow test suite (E2E tests take hours)
- Expensive (cloud costs for every test run)
- Brittle (tests fail for infrastructure issues, not code bugs)
- Poor developer experience (slow feedback)

✅ **Proper Pyramid:**
- Fast feedback (unit tests in seconds)
- Low cost (mostly runs locally)
- Reliable (unit tests don't depend on network)
- Good developer experience

**DevSecOps Application:**

```python
# 70% Unit Tests - Infrastructure code logic
def test_security_group_rule_validation():
    """Unit test - validates security group rules."""
    rule = SecurityGroupRule(
        protocol='tcp',
        port=22,
        source='0.0.0.0/0'
    )
    
    validator = SecurityGroupValidator()
    issues = validator.validate(rule)
    
    assert 'SSH open to world' in issues

# 20% Integration Tests - Real AWS/K8s interaction
@pytest.mark.integration
def test_create_security_group_in_aws():
    """Integration test - creates real AWS security group."""
    # Uses real AWS (test account)
    sg_id = create_security_group(test_vpc_id, test_rules)
    
    # Verify in AWS
    sg = ec2.describe_security_groups(GroupIds=[sg_id])
    assert len(sg['IpPermissions']) == len(test_rules)
    
    # Cleanup
    ec2.delete_security_group(GroupId=sg_id)

# 10% E2E Tests - Complete deployment workflow
@pytest.mark.e2e
def test_deploy_application_end_to_end():
    """E2E test - full deployment pipeline."""
    # Build -> Test -> Deploy -> Verify
    deploy_complete('my-app', 'v1.0.0', environment='staging')
    
    # Verify app is running
    response = requests.get('https://staging.example.com/health')
    assert response.status_code == 200
```

**Why This Matters:**
- **Fast CI/CD**: Unit tests run on every commit (seconds)
- **Catch bugs early**: Most bugs found in unit tests
- **Lower costs**: Fewer expensive cloud resources
- **Developer productivity**: Fast feedback loop

**Follow-up:** What happens if your pyramid is inverted (mostly E2E tests)?

---

**Question 2: You're implementing a deployment script that provisions AWS resources. What should you test, and how would you structure the tests?**

**Difficulty:** Mid-Level

**Answer:**

**Deployment Script Breakdown:**

```python
# deployment_script.py
import boto3
from typing import Dict, List

def deploy_infrastructure(config: Dict) -> Dict:
    """
    Deploy infrastructure to AWS.
    
    Steps:
    1. Validate configuration
    2. Create VPC and subnets
    3. Create security groups
    4. Provision EC2 instances
    5. Configure load balancer
    6. Update DNS records
    """
    # Validation
    errors = validate_config(config)
    if errors:
        raise ValueError(f"Invalid config: {errors}")
    
    # Create resources
    vpc_id = create_vpc(config['vpc_cidr'])
    subnet_ids = create_subnets(vpc_id, config['subnets'])
    sg_id = create_security_group(vpc_id, config['security_rules'])
    
    instance_ids = provision_instances(
        subnet_ids,
        sg_id,
        config['instance_config']
    )
    
    lb_dns = create_load_balancer(
        subnet_ids,
        instance_ids,
        config['lb_config']
    )
    
    update_dns_record(config['domain'], lb_dns)
    
    return {
        'vpc_id': vpc_id,
        'instance_ids': instance_ids,
        'load_balancer': lb_dns
    }
```

**Test Strategy:**

**1. Unit Tests (70%) - Test Each Function:**

```python
# tests/unit/test_validation.py
def test_validate_config_accepts_valid_config():
    """Test config validation with valid configuration."""
    valid_config = {
        'vpc_cidr': '10.0.0.0/16',
        'subnets': ['10.0.1.0/24', '10.0.2.0/24'],
        'security_rules': [
            {'protocol': 'tcp', 'port': 443, 'source': '0.0.0.0/0'}
        ],
        'instance_config': {'type': 't2.micro', 'count': 2},
        'lb_config': {'type': 'application'},
        'domain': 'example.com'
    }
    
    errors = validate_config(valid_config)
    assert errors == []

def test_validate_config_rejects_invalid_cidr():
    """Test config validation with invalid CIDR."""
    invalid_config = {
        'vpc_cidr': '999.999.999.999/99',  # Invalid
        'subnets': [],
        'security_rules': [],
        'instance_config': {},
        'lb_config': {},
        'domain': 'example.com'
    }
    
    errors = validate_config(invalid_config)
    assert 'Invalid VPC CIDR' in errors

def test_validate_config_requires_minimum_subnets():
    """Test config validation requires at least 2 subnets."""
    config = {
        'vpc_cidr': '10.0.0.0/16',
        'subnets': ['10.0.1.0/24'],  # Only 1 subnet
        'security_rules': [],
        'instance_config': {},
        'lb_config': {},
        'domain': 'example.com'
    }
    
    errors = validate_config(config)
    assert 'At least 2 subnets required' in errors

# tests/unit/test_security_group.py
from unittest.mock import MagicMock, patch

def test_create_security_group_calls_aws_api_correctly():
    """Test security group creation with correct AWS API calls."""
    with patch('boto3.client') as mock_boto:
        mock_ec2 = MagicMock()
        mock_boto.return_value = mock_ec2
        mock_ec2.create_security_group.return_value = {
            'GroupId': 'sg-12345'
        }
        
        vpc_id = 'vpc-12345'
        rules = [
            {'protocol': 'tcp', 'port': 443, 'source': '0.0.0.0/0'}
        ]
        
        sg_id = create_security_group(vpc_id, rules)
        
        # Verify API call
        mock_ec2.create_security_group.assert_called_once_with(
            GroupName='app-security-group',
            Description='Security group for application',
            VpcId='vpc-12345'
        )
        
        # Verify rules were added
        mock_ec2.authorize_security_group_ingress.assert_called_once()
        
        assert sg_id == 'sg-12345'

def test_create_security_group_handles_aws_error():
    """Test security group creation handles AWS errors."""
    from botocore.exceptions import ClientError
    
    with patch('boto3.client') as mock_boto:
        mock_ec2 = MagicMock()
        mock_boto.return_value = mock_ec2
        mock_ec2.create_security_group.side_effect = ClientError(
            {'Error': {'Code': 'InvalidGroup.Duplicate'}},
            'CreateSecurityGroup'
        )
        
        with pytest.raises(SecurityGroupError, match='already exists'):
            create_security_group('vpc-12345', [])
```

**2. Integration Tests (20%) - Test AWS Interaction:**

```python
# tests/integration/test_aws_integration.py
import pytest
import boto3
from testcontainers.localstack import LocalStackContainer

@pytest.fixture(scope='session')
def aws_credentials():
    """Provide test AWS credentials."""
    return {
        'aws_access_key_id': 'test',
        'aws_secret_access_key': 'test',
        'region_name': 'us-east-1'
    }

@pytest.fixture(scope='session')
def localstack():
    """Start LocalStack container for AWS simulation."""
    with LocalStackContainer() as localstack:
        yield localstack

@pytest.mark.integration
def test_create_vpc_in_localstack(localstack, aws_credentials):
    """Test VPC creation with LocalStack."""
    ec2 = boto3.client(
        'ec2',
        endpoint_url=localstack.get_url(),
        **aws_credentials
    )
    
    vpc_id = create_vpc('10.0.0.0/16', ec2_client=ec2)
    
    # Verify VPC exists
    vpcs = ec2.describe_vpcs(VpcIds=[vpc_id])
    assert len(vpcs['Vpcs']) == 1
    assert vpcs['Vpcs'][0]['CidrBlock'] == '10.0.0.0/16'

@pytest.mark.integration
def test_provision_instance_in_localstack(localstack, aws_credentials):
    """Test EC2 instance provisioning with LocalStack."""
    ec2 = boto3.client(
        'ec2',
        endpoint_url=localstack.get_url(),
        **aws_credentials
    )
    
    # Setup VPC and subnet first
    vpc_id = create_vpc('10.0.0.0/16', ec2_client=ec2)
    subnet_id = create_subnet(vpc_id, '10.0.1.0/24', ec2_client=ec2)
    
    # Provision instance
    instance_ids = provision_instances(
        [subnet_id],
        'sg-12345',
        {'type': 't2.micro', 'count': 1},
        ec2_client=ec2
    )
    
    assert len(instance_ids) == 1
    
    # Verify instance
    instances = ec2.describe_instances(InstanceIds=instance_ids)
    assert instances['Reservations'][0]['Instances'][0]['InstanceType'] == 't2.micro'
```

**3. E2E Tests (10%) - Full Pipeline:**

```python
# tests/e2e/test_deployment_pipeline.py
@pytest.mark.e2e
@pytest.mark.slow
@pytest.mark.skipif(
    os.getenv('CI') != 'true',
    reason='E2E tests only run in CI'
)
def test_full_deployment_to_staging():
    """
    Test complete deployment pipeline to staging environment.
    
    Prerequisites:
    - AWS test account configured
    - Staging environment available
    - DNS zone configured
    """
    config = load_config('configs/staging.yaml')
    
    # Deploy infrastructure
    result = deploy_infrastructure(config)
    
    try:
        # Verify all resources created
        assert result['vpc_id'].startswith('vpc-')
        assert len(result['instance_ids']) == 2
        assert result['load_balancer'].endswith('.amazonaws.com')
        
        # Wait for instances to be ready
        wait_for_instances_healthy(result['instance_ids'])
        
        # Verify application is accessible
        response = requests.get(f"http://{result['load_balancer']}/health")
        assert response.status_code == 200
        
        # Verify DNS record
        import dns.resolver
        answers = dns.resolver.resolve('staging.example.com', 'CNAME')
        assert str(answers[0]) == result['load_balancer']
        
    finally:
        # Cleanup (always runs)
        cleanup_infrastructure(result)
```

**Test Organization:**

```
tests/
├── __init__.py
├── conftest.py  # Shared fixtures
├── unit/
│   ├── test_validation.py
│   ├── test_vpc.py
│   ├── test_security_group.py
│   ├── test_instances.py
│   └── test_load_balancer.py
├── integration/
│   ├── test_aws_integration.py
│   └── test_dns_integration.py
└── e2e/
    └── test_deployment_pipeline.py
```

**pytest.ini Configuration:**

```ini
[pytest]
# Test discovery
python_files = test_*.py
python_classes = Test*
python_functions = test_*

# Markers
markers =
    unit: Unit tests (fast, no external dependencies)
    integration: Integration tests (real AWS via LocalStack)
    e2e: End-to-end tests (real staging environment)
    slow: Slow tests (run separately)

# Coverage
addopts = 
    --cov=deployment_script
    --cov-report=html
    --cov-report=term
    --cov-fail-under=80
    -v
    -ra

# Run only unit tests by default
testpaths = tests/unit

# Ignore slow tests by default
norecursedirs = tests/e2e
```

**Running Tests:**

```bash
# Run all unit tests (default, fast)
pytest

# Run with coverage
pytest --cov=deployment_script --cov-report=html

# Run integration tests
pytest tests/integration -m integration

# Run E2E tests (slow, only in CI)
pytest tests/e2e -m e2e

# Run specific test
pytest tests/unit/test_validation.py::test_validate_config_accepts_valid_config

# Run with verbose output
pytest -vv

# Run tests in parallel (faster)
pytest -n auto
```

**CI/CD Integration:**

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements-test.txt
      
      - name: Run unit tests
        run: pytest tests/unit --cov --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
  
  integration-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Start LocalStack
        run: docker-compose up -d localstack
      
      - name: Run integration tests
        run: pytest tests/integration -m integration
  
  e2e-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Only on main branch
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_TEST_ROLE }}
      
      - name: Run E2E tests
        run: pytest tests/e2e -m e2e
        env:
          CI: 'true'
```

**Why This Approach Works:**

1. **Fast Feedback**: Unit tests run in seconds
2. **Confidence**: Integration tests verify AWS interaction
3. **Production Validation**: E2E tests catch deployment issues
4. **Cost-Effective**: Mostly unit tests (cheap)
5. **Reliable**: Less flaky than all E2E
6. **Maintainable**: Clear test organization

**Follow-up:** How would you test rollback functionality?

---

### Key Takeaways - Testing Fundamentals

✅ **Test pyramid**: 70% unit, 20% integration, 10% E2E

✅ **TDD mindset**: Write tests first for critical code

✅ **FIRST principles**: Fast, Isolated, Repeatable, Self-validating, Timely

✅ **AAA pattern**: Arrange, Act, Assert structure

✅ **Mock external dependencies**: Keep tests fast and reliable

✅ **Test error paths**: Don't just test happy path

✅ **Descriptive test names**: Test name should explain what's tested

✅ **Coverage is not enough**: Quality matters more than quantity

---

## 3.2 pytest Framework Mastery

pytest is the standard testing framework for Python. It's more powerful and easier to use than unittest.

### Why pytest Over unittest?

**unittest (stdlib):**
```python
import unittest

class TestServerProvisioner(unittest.TestCase):
    def setUp(self):
        self.provisioner = ServerProvisioner()
    
    def test_provision_server_with_valid_config(self):
        config = {'type': 't2.micro'}
        result = self.provisioner.provision(config)
        self.assertEqual(result['status'], 'success')
    
    def tearDown(self):
        self.provisioner.cleanup()

if __name__ == '__main__':
    unittest.main()
```

**pytest (recommended):**
```python
import pytest

@pytest.fixture
def provisioner():
    """Provide provisioner instance."""
    p = ServerProvisioner()
    yield p
    p.cleanup()  # Automatic cleanup

def test_provision_server_with_valid_config(provisioner):
    """Test server provisioning with valid configuration."""
    config = {'type': 't2.micro'}
    result = provisioner.provision(config)
    assert result['status'] == 'success'
```

**pytest Advantages:**
- ✅ Simple `assert` statements (no `self.assertEqual`)
- ✅ Powerful fixtures (setUp/tearDown on steroids)
- ✅ Parametrized tests (run same test with different inputs)
- ✅ Rich plugin ecosystem
- ✅ Better failure messages
- ✅ Less boilerplate

### Basic pytest Usage

**Installation:**

```bash
pip install pytest pytest-cov pytest-mock pytest-asyncio
```

**Simple Test:**

```python
# test_calculator.py
def add(a, b):
    return a + b

def test_add():
    """Test addition function."""
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

**Run tests:**

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test file
pytest test_calculator.py

# Run specific test
pytest test_calculator.py::test_add

# Run tests matching pattern
pytest -k "add"

# Show print statements
pytest -s

# Stop at first failure
pytest -x

# Run last failed tests
pytest --lf

# Run tests in parallel
pytest -n auto
```

### pytest Assertions

pytest provides detailed failure messages:

```python
def test_server_config():
    """Test server configuration."""
    config = {
        'hostname': 'web-01',
        'port': 8080,
        'ssl': True
    }
    
    assert config['hostname'] == 'web-01'
    assert config['port'] == 8080
    assert config['ssl'] is True
    
    # Dictionary comparison
    expected = {'hostname': 'web-01', 'port': 8080, 'ssl': True}
    assert config == expected  # pytest shows diff on failure
    
    # List comparison
    servers = ['web-01', 'web-02', 'web-03']
    assert 'web-02' in servers
    assert len(servers) == 3
    
    # Approximate comparison (floats)
    pi = 3.14159
    assert pi == pytest.approx(3.14, rel=1e-2)
    
    # String matching
    message = "Deployment failed: timeout"
    assert 'timeout' in message
    assert message.startswith('Deployment')
```

**Failure Output:**

```python
def test_dictionary_comparison():
    actual = {'name': 'web-01', 'port': 8080, 'ssl': False}
    expected = {'name': 'web-01', 'port': 8080, 'ssl': True}
    assert actual == expected

# pytest output shows diff:
"""
AssertionError: assert {'name': 'web-01', 'port': 8080, 'ssl': False} == {'name': 'web-01', 'port': 8080, 'ssl': True}
  Difference:
  - {'name': 'web-01', 'port': 8080, 'ssl': True}
  + {'name': 'web-01', 'port': 8080, 'ssl': False}
  
  Only differences:
  'ssl': False != True
"""
```

### pytest Fixtures

Fixtures are pytest's superpower - they provide test data and setup/teardown logic.

**Basic Fixture:**

```python
import pytest

@pytest.fixture
def sample_config():
    """Provide sample configuration for tests."""
    return {
        'environment': 'test',
        'replicas': 3,
        'memory': '512Mi'
    }

def test_parse_config(sample_config):
    """Test uses fixture automatically."""
    # pytest injects sample_config
    assert sample_config['environment'] == 'test'
    assert sample_config['replicas'] == 3
```

**Fixture with Setup/Teardown:**

```python
@pytest.fixture
def database_connection():
    """Provide database connection with automatic cleanup."""
    # Setup
    conn = create_database_connection('postgresql://localhost/test')
    conn.execute('CREATE TABLE test_table (id INT)')
    
    yield conn  # Test runs here
    
    # Teardown (always runs, even if test fails)
    conn.execute('DROP TABLE test_table')
    conn.close()

def test_insert_record(database_connection):
    """Test database insertion."""
    database_connection.execute('INSERT INTO test_table VALUES (1)')
    result = database_connection.execute('SELECT * FROM test_table')
    assert len(result) == 1
```

**Fixture Scopes:**

```python
@pytest.fixture(scope='function')  # Default: New instance per test
def function_scope():
    return create_resource()

@pytest.fixture(scope='class')  # One instance per test class
def class_scope():
    return create_expensive_resource()

@pytest.fixture(scope='module')  # One instance per test module
def module_scope():
    return create_very_expensive_resource()

@pytest.fixture(scope='session')  # One instance per test session
def session_scope():
    """Shared across all tests - use carefully!"""
    return create_singleton_resource()
```

**DevSecOps Example:**

```python
# conftest.py
import pytest
import boto3
from moto import mock_aws

@pytest.fixture(scope='session')
def aws_credentials():
    """Provide fake AWS credentials."""
    os.environ['AWS_ACCESS_KEY_ID'] = 'testing'
    os.environ['AWS_SECRET_ACCESS_KEY'] = 'testing'
    os.environ['AWS_SECURITY_TOKEN'] = 'testing'
    os.environ['AWS_SESSION_TOKEN'] = 'testing'

@pytest.fixture(scope='function')
def aws_s3(aws_credentials):
    """Provide mocked S3 client."""
    with mock_aws():
        yield boto3.client('s3', region_name='us-east-1')

@pytest.fixture
def sample_bucket(aws_s3):
    """Create sample S3 bucket."""
    bucket_name = 'test-bucket'
    aws_s3.create_bucket(Bucket=bucket_name)
    return bucket_name

# test_s3.py
def test_upload_file_to_s3(aws_s3, sample_bucket):
    """Test S3 file upload."""
    # Upload file
    aws_s3.put_object(
        Bucket=sample_bucket,
        Key='test.txt',
        Body=b'test data'
    )
    
    # Verify
    response = aws_s3.get_object(Bucket=sample_bucket, Key='test.txt')
    assert response['Body'].read() == b'test data'
```

**Fixture Dependencies:**

```python
@pytest.fixture
def vpc(aws_ec2):
    """Create VPC."""
    response = aws_ec2.create_vpc(CidrBlock='10.0.0.0/16')
    vpc_id = response['Vpc']['VpcId']
    yield vpc_id
    aws_ec2.delete_vpc(VpcId=vpc_id)

@pytest.fixture
def subnet(aws_ec2, vpc):
    """Create subnet in VPC (depends on vpc fixture)."""
    response = aws_ec2.create_subnet(
        VpcId=vpc,
        CidrBlock='10.0.1.0/24'
    )
    subnet_id = response['Subnet']['SubnetId']
    yield subnet_id
    aws_ec2.delete_subnet(SubnetId=subnet_id)

@pytest.fixture
def security_group(aws_ec2, vpc):
    """Create security group in VPC."""
    response = aws_ec2.create_security_group(
        GroupName='test-sg',
        Description='Test security group',
        VpcId=vpc
    )
    sg_id = response['GroupId']
    yield sg_id
    aws_ec2.delete_security_group(GroupId=sg_id)

def test_provision_instance(aws_ec2, subnet, security_group):
    """Test instance provisioning with subnet and security group."""
    # All fixtures automatically created in correct order:
    # aws_ec2 -> vpc -> (subnet, security_group) in parallel
    
    response = aws_ec2.run_instances(
        ImageId='ami-12345',
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1,
        SubnetId=subnet,
        SecurityGroupIds=[security_group]
    )
    
    instance_id = response['Instances'][0]['InstanceId']
    assert instance_id.startswith('i-')
```

### Parametrized Tests

Run same test with different inputs:

```python
import pytest

@pytest.mark.parametrize('input,expected', [
    ('test.txt', 'txt'),
    ('archive.tar.gz', 'gz'),
    ('script.py', 'py'),
    ('README', ''),
])
def test_get_file_extension(input, expected):
    """Test file extension extraction."""
    assert get_file_extension(input) == expected

# Runs 4 tests:
# test_get_file_extension[test.txt-txt]
# test_get_file_extension[archive.tar.gz-gz]
# test_get_file_extension[script.py-py]
# test_get_file_extension[README-]
```

**DevSecOps Example:**

```python
@pytest.mark.parametrize('instance_type,expected_cost', [
    ('t2.micro', 0.0116),
    ('t2.small', 0.023),
    ('t2.medium', 0.0464),
    ('t2.large', 0.0928),
])
def test_calculate_hourly_cost(instance_type, expected_cost):
    """Test cost calculation for different instance types."""
    calculator = CostCalculator()
    cost = calculator.calculate_hourly_cost(instance_type)
    assert cost == pytest.approx(expected_cost, rel=0.01)

@pytest.mark.parametrize('cidr,is_valid', [
    ('10.0.0.0/16', True),
    ('192.168.1.0/24', True),
    ('999.999.999.999/99', False),
    ('10.0.0.0', False),  # Missing subnet mask
    ('10.0.0.0/33', False),  # Invalid mask
])
def test_validate_cidr(cidr, is_valid):
    """Test CIDR validation."""
    assert validate_cidr(cidr) == is_valid
```

**Multiple Parameters:**

```python
@pytest.mark.parametrize('environment', ['dev', 'staging', 'prod'])
@pytest.mark.parametrize('region', ['us-east-1', 'eu-west-1'])
def test_deploy_to_environment_and_region(environment, region):
    """Test deployment to all environment/region combinations."""
    # Runs 6 tests (3 environments × 2 regions)
    result = deploy(environment, region)
    assert result['environment'] == environment
    assert result['region'] == region
```

### pytest Markers

Organize and selectively run tests:

```python
import pytest

@pytest.mark.slow
def test_full_deployment():
    """Mark as slow test."""
    # Long-running test
    pass

@pytest.mark.integration
def test_aws_integration():
    """Mark as integration test."""
    pass

@pytest.mark.security
def test_vulnerability_scan():
    """Mark as security test."""
    pass

@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    """Skip this test."""
    pass

@pytest.mark.skipif(
    sys.platform == 'win32',
    reason="Unix-only test"
)
def test_unix_specific():
    """Skip on Windows."""
    pass

@pytest.mark.xfail(reason="Known bug")
def test_known_issue():
    """Expect this test to fail."""
    assert False  # Won't fail the test suite

# Custom markers (define in pytest.ini)
@pytest.mark.smoke
def test_basic_functionality():
    """Smoke test."""
    pass
```

**pytest.ini:**

```ini
[pytest]
markers =
    slow: Slow tests (deselect with '-m "not slow"')
    integration: Integration tests
    e2e: End-to-end tests
    smoke: Smoke tests (quick validation)
    security: Security-related tests
    unit: Unit tests
```

**Run tests by marker:**

```bash
# Run only unit tests
pytest -m unit

# Run integration and e2e tests
pytest -m "integration or e2e"

# Run all except slow tests
pytest -m "not slow"

# Run smoke tests only
pytest -m smoke
```

### conftest.py - Shared Fixtures

`conftest.py` provides fixtures shared across tests:

```python
# conftest.py in tests/ directory
import pytest
import boto3
from moto import mock_aws

@pytest.fixture(scope='session')
def aws_credentials():
    """Mock AWS credentials for all tests."""
    os.environ['AWS_ACCESS_KEY_ID'] = 'testing'
    os.environ['AWS_SECRET_ACCESS_KEY'] = 'testing'

@pytest.fixture
def aws_ec2(aws_credentials):
    """Provide mocked EC2 client."""
    with mock_aws():
        yield boto3.client('ec2', region_name='us-east-1')

@pytest.fixture
def sample_vpc(aws_ec2):
    """Create sample VPC for tests."""
    response = aws_ec2.create_vpc(CidrBlock='10.0.0.0/16')
    vpc_id = response['Vpc']['VpcId']
    yield vpc_id
    # Cleanup
    aws_ec2.delete_vpc(VpcId=vpc_id)

# Now all test files can use these fixtures without importing
```

**Directory structure:**

```
tests/
├── conftest.py          # Root-level fixtures
├── unit/
│   ├── conftest.py      # Unit test fixtures
│   └── test_*.py
└── integration/
    ├── conftest.py      # Integration test fixtures
    └── test_*.py
```

### Frequently Asked Questions - pytest

**Q1: Should I use fixtures or setUp/tearDown?**

**A:** **Use fixtures.** They're more powerful and flexible.

**Fixtures Advantages:**
- Can depend on other fixtures
- Different scopes (function, class, module, session)
- Can be shared across files (conftest.py)
- Better error handling
- More readable

```python
# ❌ unittest style
class TestServer(unittest.TestCase):
    def setUp(self):
        self.server = Server()
    
    def tearDown(self):
        self.server.stop()
    
    def test_start(self):
        self.server.start()
        self.assertTrue(self.server.is_running)

# ✅ pytest style
@pytest.fixture
def server():
    s = Server()
    yield s
    s.stop()  # Cleanup

def test_start(server):
    server.start()
    assert server.is_running
```

**Q2: How do I test code that uses random values or time?**

**A:** **Mock random and time.**

```python
# Code under test
import random
import time

def generate_deployment_id():
    """Generate random deployment ID with timestamp."""
    timestamp = int(time.time())
    random_suffix = random.randint(1000, 9999)
    return f"deploy-{timestamp}-{random_suffix}"

# Test
from unittest.mock import patch

def test_generate_deployment_id():
    """Test deployment ID generation."""
    with patch('time.time', return_value=1609459200):  # Fixed timestamp
        with patch('random.randint', return_value=5678):  # Fixed random
            deployment_id = generate_deployment_id()
            assert deployment_id == "deploy-1609459200-5678"
```

**Fixture for time mocking:**

```python
# conftest.py
import pytest
from unittest.mock import patch

@pytest.fixture
def frozen_time():
    """Freeze time at 2024-01-01 00:00:00."""
    with patch('time.time', return_value=1704067200):
        yield

def test_with_frozen_time(frozen_time):
    """Test uses frozen time."""
    import time
    assert time.time() == 1704067200
```

**Q3: How do I test code that prints to stdout/stderr?**

**A:** Use pytest's `capsys` fixture.

```python
def deploy_with_logging(config):
    """Deploy and print progress."""
    print(f"Starting deployment: {config['app']}")
    # ... deployment logic ...
    print("Deployment complete")
    return True

def test_deploy_prints_progress(capsys):
    """Test that deployment prints progress."""
    deploy_with_logging({'app': 'my-app'})
    
    captured = capsys.readouterr()
    assert "Starting deployment: my-app" in captured.out
    assert "Deployment complete" in captured.out
```

### Interview Questions - pytest

**Question: You need to test a function that deploys to AWS. The function is slow because it waits for resources to be ready. How would you structure tests to run quickly while still maintaining confidence?**

**Difficulty:** Mid-Level

**Answer:**

**Problem:**

```python
def deploy_infrastructure(config):
    """Deploy infrastructure to AWS."""
    # Create VPC (2 seconds)
    vpc_id = create_vpc(config['vpc_cidr'])
    
    # Create subnets (5 seconds)
    subnet_ids = create_subnets(vpc_id, config['subnets'])
    
    # Provision instances (30 seconds)
    instance_ids = provision_instances(subnet_ids, config)
    
    # Wait for instances to be ready (60 seconds)
    wait_for_instances_ready(instance_ids)
    
    return {'vpc_id': vpc_id, 'instance_ids': instance_ids}

# Testing this takes 97 seconds! 😱
```

**Solution: Layer tests strategically**

**1. Unit Tests (Fast) - Mock AWS calls:**

```python
from unittest.mock import patch, MagicMock

def test_deploy_infrastructure_calls_aws_apis_correctly():
    """Test AWS API calls without waiting (runs in milliseconds)."""
    with patch('infrastructure.create_vpc') as mock_vpc, \
         patch('infrastructure.create_subnets') as mock_subnets, \
         patch('infrastructure.provision_instances') as mock_instances, \
         patch('infrastructure.wait_for_instances_ready') as mock_wait:
        
        # Setup mocks
        mock_vpc.return_value = 'vpc-12345'
        mock_subnets.return_value = ['subnet-1', 'subnet-2']
        mock_instances.return_value = ['i-123', 'i-456']
        
        config = {
            'vpc_cidr': '10.0.0.0/16',
            'subnets': ['10.0.1.0/24', '10.0.2.0/24']
        }
        
        result = deploy_infrastructure(config)
        
        # Verify correct calls made
        mock_vpc.assert_called_once_with('10.0.0.0/16')
        mock_subnets.assert_called_once_with('vpc-12345', config['subnets'])
        mock_instances.assert_called_once()
        mock_wait.assert_called_once_with(['i-123', 'i-456'])
        
        assert result['vpc_id'] == 'vpc-12345'
        assert result['instance_ids'] == ['i-123', 'i-456']

def test_deploy_infrastructure_handles_vpc_creation_failure():
    """Test error handling when VPC creation fails."""
    with patch('infrastructure.create_vpc') as mock_vpc:
        mock_vpc.side_effect = AWSError("VPC limit reached")
        
        config = {'vpc_cidr': '10.0.0.0/16', 'subnets': []}
        
        with pytest.raises(DeploymentError, match="VPC limit"):
            deploy_infrastructure(config)

# Many more unit tests... all run in <1 second total
```

**2. Integration Tests (Medium) - Use moto for AWS simulation:**

```python
import pytest
import boto3
from moto import mock_aws

@pytest.fixture
def aws_ec2():
    """Provide mocked EC2 client."""
    with mock_aws():
        yield boto3.client('ec2', region_name='us-east-1')

@pytest.mark.integration
def test_create_vpc_with_mocked_aws(aws_ec2):
    """Test VPC creation with mocked AWS (no waiting, runs in ~100ms)."""
    vpc_id = create_vpc('10.0.0.0/16', ec2_client=aws_ec2)
    
    # Verify VPC was created
    vpcs = aws_ec2.describe_vpcs(VpcIds=[vpc_id])
    assert len(vpcs['Vpcs']) == 1
    assert vpcs['Vpcs'][0]['CidrBlock'] == '10.0.0.0/16'

@pytest.mark.integration
def test_provision_instances_with_mocked_aws(aws_ec2):
    """Test instance provisioning with mocked AWS."""
    # Setup
    vpc_id = create_vpc('10.0.0.0/16', ec2_client=aws_ec2)
    subnet_id = create_subnet(vpc_id, '10.0.1.0/24', ec2_client=aws_ec2)
    
    # Provision
    config = {'instance_type': 't2.micro', 'count': 2}
    instance_ids = provision_instances([subnet_id], config, ec2_client=aws_ec2)
    
    # Verify (moto responds instantly, no waiting)
    assert len(instance_ids) == 2
    
    instances = aws_ec2.describe_instances(InstanceIds=instance_ids)
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            assert instance['InstanceType'] == 't2.micro'
            assert instance['State']['Name'] == 'running'  # Mocked as running

# Integration tests run in ~1 second total
```

**3. E2E Tests (Slow) - Minimal, only critical paths:**

```python
@pytest.mark.e2e
@pytest.mark.slow
@pytest.mark.skipif(
    os.getenv('RUN_E2E') != 'true',
    reason='E2E tests only in CI'
)
def test_deploy_infrastructure_end_to_end():
    """
    Test deployment to real AWS (test account).
    
    This test is slow (97 seconds) but runs infrequently:
    - Not on every commit
    - Only before releases
    - Only in CI/CD pipeline
    """
    config = {
        'vpc_cidr': '10.0.0.0/16',
        'subnets': ['10.0.1.0/24', '10.0.2.0/24'],
        'instance_count': 2
    }
    
    try:
        result = deploy_infrastructure(config)
        
        # Verify in real AWS
        ec2 = boto3.client('ec2', region_name='us-east-1')
        vpcs = ec2.describe_vpcs(VpcIds=[result['vpc_id']])
        assert len(vpcs['Vpcs']) == 1
        
        instances = ec2.describe_instances(InstanceIds=result['instance_ids'])
        for reservation in instances['Reservations']:
            for instance in reservation['Instances']:
                assert instance['State']['Name'] == 'running'
    
    finally:
        # Always cleanup real resources
        cleanup_infrastructure(result)
```

**4. Optimize wait functions for testing:**

```python
def wait_for_instances_ready(
    instance_ids,
    timeout=300,
    check_interval=10,
    ec2_client=None  # Allow injection for testing
):
    """
    Wait for instances to be ready.
    
    Testable because:
    - ec2_client can be mocked
    - timeout/interval can be reduced for tests
    """
    if ec2_client is None:
        ec2_client = boto3.client('ec2')
    
    start_time = time.time()
    
    while time.time() - start_time < timeout:
        instances = ec2_client.describe_instances(InstanceIds=instance_ids)
        
        all_running = all(
            instance['State']['Name'] == 'running'
            for reservation in instances['Reservations']
            for instance in reservation['Instances']
        )
        
        if all_running:
            return True
        
        time.sleep(check_interval)
    
    raise TimeoutError("Instances not ready")

# Test with reduced timeout
def test_wait_for_instances_timeout():
    """Test timeout handling (runs in 2 seconds)."""
    with patch('infrastructure.boto3.client') as mock_boto:
        mock_ec2 = MagicMock()
        mock_boto.return_value = mock_ec2
        
        # Always return pending state
        mock_ec2.describe_instances.return_value = {
            'Reservations': [{
                'Instances': [{'State': {'Name': 'pending'}}]
            }]
        }
        
        with pytest.raises(TimeoutError):
            wait_for_instances_ready(
                ['i-123'],
                timeout=2,  # Reduced for testing
                check_interval=0.5,
                ec2_client=mock_ec2
            )
```

**Test Execution Strategy:**

```bash
# Developer workflow (fast feedback)
pytest tests/unit  # Runs in <1 second
pytest tests/integration  # Runs in ~2 seconds

# PR validation (CI)
pytest tests/unit tests/integration  # Runs in ~3 seconds

# Pre-release validation (nightly CI)
pytest  # Includes E2E tests, runs in ~100 seconds
```

**pytest Configuration:**

```ini
# pytest.ini
[pytest]
markers =
    unit: Fast unit tests (mocked)
    integration: Medium integration tests (moto)
    e2e: Slow E2E tests (real AWS)
    slow: Mark slow tests

# Default: only unit tests
testpaths = tests/unit

# Coverage
addopts = --cov=infrastructure --cov-report=html
```

**Why This Works:**

1. **Fast feedback**: Unit tests run instantly
2. **Confidence**: Integration tests verify AWS interaction
3. **Production validation**: E2E tests catch real issues
4. **Cost-effective**: Minimal real AWS usage
5. **Maintainable**: Clear separation of test types

**Follow-up:** How would you test rollback functionality without doubling test time?

---

### Key Takeaways - pytest

✅ **pytest over unittest** - Simpler and more powerful

✅ **Fixtures for setup/teardown** - More flexible than setUp/tearDown

✅ **Parametrize for data-driven tests** - Test multiple inputs easily

✅ **Markers for organization** - Run specific test subsets

✅ **conftest.py for shared fixtures** - Reuse across test files

✅ **Mock external services** - Keep tests fast and reliable

✅ **Layer tests strategically** - Unit (fast), integration (medium), E2E (slow)

✅ **Use moto for AWS testing** - Simulate AWS without costs

---

*[Content continues with Mocking, Test Coverage, Property-Based Testing sections...]*

*Due to response length, Part 3 is being built incrementally. The document currently has strong foundations in testing fundamentals and pytest framework. Would you like me to continue building Part 3 or move to creating a new part?*

## 3.3 Mocking and Test Doubles

Mocking is essential for testing code with external dependencies. In DevSecOps, you'll mock AWS APIs, databases, HTTP services, and more.

### Understanding Test Doubles

**Test Double:** Generic term for any object used in place of a real dependency for testing.

**Types of Test Doubles:**

```python
# 1. DUMMY: Passed but never used
def test_create_user(dummy_logger):
    """Logger passed but never called."""
    user = User('alice')
    assert user.name == 'alice'

# 2. STUB: Returns canned responses
class StubEmailService:
    def send_email(self, to, subject, body):
        return True  # Always succeeds

# 3. SPY: Records information about calls
class SpyEmailService:
    def __init__(self):
        self.emails_sent = []
    
    def send_email(self, to, subject, body):
        self.emails_sent.append((to, subject, body))
        return True

# 4. MOCK: Pre-programmed with expectations
mock_email = MagicMock()
mock_email.send_email.return_value = True
# Can verify: mock_email.send_email.assert_called_once()

# 5. FAKE: Working implementation (simplified)
class FakeDatabase:
    def __init__(self):
        self.data = {}
    
    def save(self, key, value):
        self.data[key] = value
    
    def get(self, key):
        return self.data.get(key)
```

**When to Use Each:**

| Type | Use Case | Example |
|------|----------|---------|
| **Dummy** | Fulfill parameter requirements | Logging object when testing logic |
| **Stub** | Return fixed values | API returning static test data |
| **Spy** | Verify behavior happened | Recording which emails were sent |
| **Mock** | Verify interactions | Ensure AWS API called with correct params |
| **Fake** | Real behavior, simplified | In-memory database for tests |

### unittest.mock - Python's Mocking Library

Python's built-in mocking library is powerful and comprehensive.

#### Basic Mock Usage

```python
from unittest.mock import Mock, MagicMock, patch

# Create mock object
mock_api = Mock()

# Configure return value
mock_api.get_user.return_value = {'id': 1, 'name': 'Alice'}

# Use mock
user = mock_api.get_user('alice')
assert user['name'] == 'Alice'

# Verify call
mock_api.get_user.assert_called_once_with('alice')

# Mock with side effects
mock_api.create_user.side_effect = ValueError("User exists")

try:
    mock_api.create_user('alice')
except ValueError as e:
    assert str(e) == "User exists"
```

#### Mock vs MagicMock

```python
# Mock: Basic mock object
mock = Mock()
mock.method.return_value = 42

# MagicMock: Supports magic methods (__len__, __getitem__, etc.)
magic_mock = MagicMock()
magic_mock.__len__.return_value = 3
magic_mock.__getitem__.return_value = 'value'

# Usage
len(magic_mock)  # 3
magic_mock[0]  # 'value'

# For most cases, use MagicMock (it's more flexible)
```

#### Patching with @patch

```python
# Code under test
# deployment.py
import boto3

def provision_ec2_instance(instance_type: str) -> str:
    """Provision EC2 instance."""
    ec2 = boto3.client('ec2', region_name='us-east-1')
    
    response = ec2.run_instances(
        ImageId='ami-12345',
        InstanceType=instance_type,
        MinCount=1,
        MaxCount=1
    )
    
    return response['Instances'][0]['InstanceId']


# Test with @patch decorator
from unittest.mock import patch, MagicMock

@patch('deployment.boto3.client')
def test_provision_ec2_instance(mock_boto_client):
    """Test EC2 provisioning with mocked boto3."""
    # Setup mock
    mock_ec2 = MagicMock()
    mock_boto_client.return_value = mock_ec2
    
    mock_ec2.run_instances.return_value = {
        'Instances': [{'InstanceId': 'i-12345'}]
    }
    
    # Call function
    instance_id = provision_ec2_instance('t2.micro')
    
    # Verify
    assert instance_id == 'i-12345'
    
    # Verify boto3.client called correctly
    mock_boto_client.assert_called_once_with('ec2', region_name='us-east-1')
    
    # Verify run_instances called correctly
    mock_ec2.run_instances.assert_called_once_with(
        ImageId='ami-12345',
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1
    )


# Multiple patches
@patch('deployment.boto3.client')
@patch('deployment.send_notification')
def test_with_multiple_patches(mock_notify, mock_boto):
    """Test with multiple mocked dependencies."""
    # Patches applied in reverse order (bottom-up)
    pass


# Patch as context manager
def test_with_context_manager():
    """Test using patch as context manager."""
    with patch('deployment.boto3.client') as mock_boto:
        mock_ec2 = MagicMock()
        mock_boto.return_value = mock_ec2
        mock_ec2.run_instances.return_value = {
            'Instances': [{'InstanceId': 'i-67890'}]
        }
        
        instance_id = provision_ec2_instance('t2.small')
        assert instance_id == 'i-67890'
```

#### Patching Best Practices

```python
# ✅ GOOD: Patch where it's used, not where it's defined
# If deployment.py imports boto3, patch 'deployment.boto3'
@patch('deployment.boto3.client')  # ✅ Correct
def test_correct_patch(mock_boto):
    pass

# ❌ BAD: Patching at source
@patch('boto3.client')  # ❌ Might not work
def test_wrong_patch(mock_boto):
    pass


# ✅ GOOD: Patch entire module if needed
@patch('deployment.boto3')
def test_patch_module(mock_boto):
    """Patch entire boto3 module."""
    mock_boto.client.return_value = MagicMock()


# ✅ GOOD: Use patch.object for methods
class DeploymentService:
    def _call_aws(self):
        import boto3
        return boto3.client('ec2')

@patch.object(DeploymentService, '_call_aws')
def test_with_patch_object(mock_call_aws):
    """Patch specific method on class."""
    mock_call_aws.return_value = MagicMock()
    service = DeploymentService()
    service.deploy()
```

### DevSecOps Mocking Examples

#### Mocking AWS Services

```python
# Using moto for realistic AWS mocking
import boto3
from moto import mock_aws
import pytest

@pytest.fixture
def aws_credentials():
    """Mock AWS credentials."""
    import os
    os.environ['AWS_ACCESS_KEY_ID'] = 'testing'
    os.environ['AWS_SECRET_ACCESS_KEY'] = 'testing'
    os.environ['AWS_SECURITY_TOKEN'] = 'testing'
    os.environ['AWS_SESSION_TOKEN'] = 'testing'


@mock_aws
def test_create_s3_bucket(aws_credentials):
    """Test S3 bucket creation with moto."""
    # moto intercepts boto3 calls
    s3 = boto3.client('s3', region_name='us-east-1')
    
    # Create bucket (mocked)
    s3.create_bucket(Bucket='test-bucket')
    
    # Verify bucket exists
    buckets = s3.list_buckets()
    assert len(buckets['Buckets']) == 1
    assert buckets['Buckets'][0]['Name'] == 'test-bucket'


@mock_aws
def test_ec2_instance_lifecycle(aws_credentials):
    """Test EC2 instance creation and termination."""
    ec2 = boto3.client('ec2', region_name='us-east-1')
    
    # Create instance
    response = ec2.run_instances(
        ImageId='ami-12345',
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1
    )
    
    instance_id = response['Instances'][0]['InstanceId']
    
    # Verify instance exists
    instances = ec2.describe_instances(InstanceIds=[instance_id])
    assert len(instances['Reservations']) == 1
    assert instances['Reservations'][0]['Instances'][0]['State']['Name'] == 'running'
    
    # Terminate instance
    ec2.terminate_instances(InstanceIds=[instance_id])
    
    # Verify terminated
    instances = ec2.describe_instances(InstanceIds=[instance_id])
    assert instances['Reservations'][0]['Instances'][0]['State']['Name'] == 'terminated'


# Testing deployment script with moto
@mock_aws
def test_deploy_infrastructure(aws_credentials):
    """Test complete infrastructure deployment."""
    from deployment import deploy_infrastructure
    
    config = {
        'vpc_cidr': '10.0.0.0/16',
        'subnet_cidr': '10.0.1.0/24',
        'instance_type': 't2.micro'
    }
    
    result = deploy_infrastructure(config)
    
    # Verify resources created
    ec2 = boto3.client('ec2', region_name='us-east-1')
    
    # Verify VPC
    vpcs = ec2.describe_vpcs(VpcIds=[result['vpc_id']])
    assert vpcs['Vpcs'][0]['CidrBlock'] == '10.0.0.0/16'
    
    # Verify instances
    instances = ec2.describe_instances(InstanceIds=result['instance_ids'])
    assert len(instances['Reservations'][0]['Instances']) == 1
```

#### Mocking HTTP Requests

```python
import requests
from unittest.mock import patch, Mock

# Code under test
def check_service_health(url: str) -> dict:
    """Check if service is healthy."""
    response = requests.get(f"{url}/health", timeout=5)
    return {
        'healthy': response.status_code == 200,
        'status_code': response.status_code,
        'response_time': response.elapsed.total_seconds()
    }


# Test with mock
@patch('requests.get')
def test_check_service_health_success(mock_get):
    """Test health check with successful response."""
    # Setup mock response
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.elapsed.total_seconds.return_value = 0.123
    mock_get.return_value = mock_response
    
    # Call function
    result = check_service_health('http://api.example.com')
    
    # Verify
    assert result['healthy'] is True
    assert result['status_code'] == 200
    assert result['response_time'] == 0.123
    
    # Verify requests.get called correctly
    mock_get.assert_called_once_with(
        'http://api.example.com/health',
        timeout=5
    )


@patch('requests.get')
def test_check_service_health_failure(mock_get):
    """Test health check with failed response."""
    mock_response = Mock()
    mock_response.status_code = 500
    mock_response.elapsed.total_seconds.return_value = 1.5
    mock_get.return_value = mock_response
    
    result = check_service_health('http://api.example.com')
    
    assert result['healthy'] is False
    assert result['status_code'] == 500


@patch('requests.get')
def test_check_service_health_timeout(mock_get):
    """Test health check with timeout."""
    mock_get.side_effect = requests.Timeout("Connection timeout")
    
    with pytest.raises(requests.Timeout):
        check_service_health('http://api.example.com')


# Using requests-mock (cleaner)
import requests_mock

def test_with_requests_mock():
    """Test with requests-mock library."""
    with requests_mock.Mocker() as m:
        # Register mock endpoint
        m.get(
            'http://api.example.com/health',
            json={'status': 'ok'},
            status_code=200
        )
        
        result = check_service_health('http://api.example.com')
        assert result['healthy'] is True
```

#### Mocking Database Operations

```python
# Code under test
import psycopg2

class DeploymentRepository:
    """Repository for deployment records."""
    
    def __init__(self, connection_string: str):
        self.conn = psycopg2.connect(connection_string)
    
    def save_deployment(self, app_name: str, version: str, status: str):
        """Save deployment record."""
        with self.conn.cursor() as cursor:
            cursor.execute(
                """
                INSERT INTO deployments (app_name, version, status, created_at)
                VALUES (%s, %s, %s, NOW())
                """,
                (app_name, version, status)
            )
        self.conn.commit()
    
    def get_latest_deployment(self, app_name: str) -> dict:
        """Get latest deployment for app."""
        with self.conn.cursor() as cursor:
            cursor.execute(
                """
                SELECT app_name, version, status, created_at
                FROM deployments
                WHERE app_name = %s
                ORDER BY created_at DESC
                LIMIT 1
                """,
                (app_name,)
            )
            row = cursor.fetchone()
            if row:
                return {
                    'app_name': row[0],
                    'version': row[1],
                    'status': row[2],
                    'created_at': row[3]
                }
            return None


# Test with mock
from unittest.mock import MagicMock, patch

@patch('psycopg2.connect')
def test_save_deployment(mock_connect):
    """Test saving deployment with mocked database."""
    # Setup mock
    mock_conn = MagicMock()
    mock_cursor = MagicMock()
    mock_connect.return_value = mock_conn
    mock_conn.cursor.return_value.__enter__.return_value = mock_cursor
    
    # Create repository
    repo = DeploymentRepository('postgresql://localhost/test')
    
    # Save deployment
    repo.save_deployment('my-app', '1.0.0', 'success')
    
    # Verify SQL executed
    mock_cursor.execute.assert_called_once()
    call_args = mock_cursor.execute.call_args[0]
    assert 'INSERT INTO deployments' in call_args[0]
    assert call_args[1] == ('my-app', '1.0.0', 'success')
    
    # Verify commit called
    mock_conn.commit.assert_called_once()


# Better: Use real database with testcontainers
from testcontainers.postgres import PostgresContainer

def test_with_real_postgres():
    """Test with real PostgreSQL in Docker container."""
    with PostgresContainer("postgres:14") as postgres:
        # Get connection string
        conn_string = postgres.get_connection_url()
        
        # Setup schema
        import psycopg2
        conn = psycopg2.connect(conn_string)
        with conn.cursor() as cursor:
            cursor.execute("""
                CREATE TABLE deployments (
                    id SERIAL PRIMARY KEY,
                    app_name VARCHAR(100),
                    version VARCHAR(50),
                    status VARCHAR(50),
                    created_at TIMESTAMP
                )
            """)
        conn.commit()
        conn.close()
        
        # Test with real database
        repo = DeploymentRepository(conn_string)
        repo.save_deployment('my-app', '1.0.0', 'success')
        
        result = repo.get_latest_deployment('my-app')
        assert result['app_name'] == 'my-app'
        assert result['version'] == '1.0.0'
        assert result['status'] == 'success'
```

#### Mocking Time

```python
import time
from datetime import datetime
from unittest.mock import patch

# Code under test
def get_deployment_id() -> str:
    """Generate deployment ID with timestamp."""
    timestamp = int(time.time())
    return f"deploy-{timestamp}"


def is_business_hours() -> bool:
    """Check if current time is business hours (9 AM - 5 PM)."""
    now = datetime.now()
    return 9 <= now.hour < 17


# Test with mocked time
@patch('time.time')
def test_get_deployment_id(mock_time):
    """Test deployment ID generation with fixed time."""
    mock_time.return_value = 1609459200  # 2021-01-01 00:00:00
    
    deployment_id = get_deployment_id()
    assert deployment_id == "deploy-1609459200"


@patch('datetime.datetime')
def test_is_business_hours_yes(mock_datetime):
    """Test business hours check during business hours."""
    # Mock datetime.now() to return 10 AM
    mock_now = Mock()
    mock_now.hour = 10
    mock_datetime.now.return_value = mock_now
    
    assert is_business_hours() is True


@patch('datetime.datetime')
def test_is_business_hours_no(mock_datetime):
    """Test business hours check outside business hours."""
    mock_now = Mock()
    mock_now.hour = 20  # 8 PM
    mock_datetime.now.return_value = mock_now
    
    assert is_business_hours() is False


# Using freezegun for better time mocking
from freezegun import freeze_time

@freeze_time("2024-01-15 10:30:00")
def test_with_freezegun():
    """Test with frozen time using freezegun."""
    assert is_business_hours() is True
    
    # Time is frozen
    id1 = get_deployment_id()
    time.sleep(1)
    id2 = get_deployment_id()
    assert id1 == id2  # Same timestamp!


@freeze_time("2024-01-15 10:30:00", tick=True)
def test_with_ticking_time():
    """Test with time that advances."""
    id1 = get_deployment_id()
    time.sleep(1)
    id2 = get_deployment_id()
    assert id1 != id2  # Different timestamps
```

### Mock Assertions

```python
from unittest.mock import Mock, call

mock = Mock()

# Call mock
mock.method('arg1', key='value')

# Assertions
mock.method.assert_called()  # Called at least once
mock.method.assert_called_once()  # Called exactly once
mock.method.assert_called_with('arg1', key='value')  # Last call args
mock.method.assert_called_once_with('arg1', key='value')  # Only call

# Not called
mock.other_method.assert_not_called()

# Call count
assert mock.method.call_count == 1

# Multiple calls
mock.method('first')
mock.method('second')
mock.method('third')

# Check call history
assert mock.method.call_count == 3
mock.method.assert_has_calls([
    call('first'),
    call('second'),
    call('third')
])

# Any order
mock.method.assert_has_calls([
    call('third'),
    call('first')
], any_order=True)
```

### pytest-mock

pytest-mock provides a `mocker` fixture that integrates unittest.mock with pytest:

```python
# Install: pip install pytest-mock

def test_with_mocker(mocker):
    """Test using pytest-mock's mocker fixture."""
    # Patch using mocker
    mock_boto = mocker.patch('deployment.boto3.client')
    mock_ec2 = mocker.MagicMock()
    mock_boto.return_value = mock_ec2
    
    mock_ec2.run_instances.return_value = {
        'Instances': [{'InstanceId': 'i-12345'}]
    }
    
    # Test code
    from deployment import provision_ec2_instance
    instance_id = provision_ec2_instance('t2.micro')
    
    assert instance_id == 'i-12345'


def test_spy_with_mocker(mocker):
    """Spy on real function."""
    # Spy instead of mock - calls real function but records calls
    spy = mocker.spy(DeploymentService, 'deploy')
    
    service = DeploymentService()
    service.deploy('my-app', '1.0.0')
    
    # Verify spy recorded call
    spy.assert_called_once_with(service, 'my-app', '1.0.0')
```

### Frequently Asked Questions - Mocking

**Q1: Should I mock everything or use real dependencies?**

**A:** **Balance mocking with integration testing.**

**Mock when:**
- External services (AWS, APIs, databases)
- Slow operations
- Non-deterministic behavior (time, random)
- Testing error conditions

**Use real when:**
- Your own code
- Pure functions
- Fast operations
- Integration tests

```python
# Unit test: Mock external dependencies
@patch('boto3.client')
def test_provision_instance_unit(mock_boto):
    """Fast unit test with mocked AWS."""
    # Runs in milliseconds
    pass

# Integration test: Real database
def test_save_to_database_integration():
    """Integration test with real database."""
    with PostgresContainer() as postgres:
        # Runs in seconds, but tests real behavior
        pass
```

**Q2: How do I know what to patch?**

**A:** **Patch where the object is used, not where it's defined.**

```python
# my_module.py
import boto3

def create_bucket(name):
    s3 = boto3.client('s3')
    s3.create_bucket(Bucket=name)

# ✅ CORRECT: Patch where boto3 is used
@patch('my_module.boto3.client')
def test_create_bucket(mock_boto):
    pass

# ❌ WRONG: Patching at source doesn't work
@patch('boto3.client')  # Won't affect my_module.boto3
def test_create_bucket_wrong(mock_boto):
    pass
```

**Rule:** If module does `import x`, patch `module.x`, not just `x`.

**Q3: When should I use moto vs unittest.mock for AWS testing?**

**A:**

**Use moto when:**
- Testing AWS service interaction
- Need realistic AWS behavior
- Testing multiple AWS calls
- Integration testing AWS flows

**Use unittest.mock when:**
- Simple AWS calls
- Just verifying parameters
- Performance critical (moto has overhead)
- Testing error handling

```python
# moto: Testing AWS behavior
@mock_aws
def test_s3_bucket_creation():
    """Test actual S3 behavior."""
    s3 = boto3.client('s3')
    s3.create_bucket(Bucket='test')
    
    # moto simulates AWS - can list buckets, check existence, etc.
    buckets = s3.list_buckets()
    assert len(buckets['Buckets']) == 1

# unittest.mock: Just verify parameters
@patch('boto3.client')
def test_s3_parameters(mock_boto):
    """Just verify correct parameters passed."""
    create_s3_bucket('my-bucket')
    
    mock_boto.return_value.create_bucket.assert_called_with(
        Bucket='my-bucket'
    )
```

### Interview Questions - Mocking

**Question 1: You need to test a function that deploys to AWS and sends a Slack notification. How would you structure the tests?**

**Difficulty:** Mid-Level

**Answer:**

**Function to test:**

```python
# deployment.py
import boto3
import requests

def deploy_application(app_name: str, version: str, slack_webhook: str) -> dict:
    """
    Deploy application to AWS and notify via Slack.
    
    Returns:
        dict with deployment_id and notification_status
    """
    # Deploy to AWS
    ec2 = boto3.client('ec2', region_name='us-east-1')
    response = ec2.run_instances(
        ImageId='ami-12345',
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1,
        Tags=[
            {'Key': 'Name', 'Value': app_name},
            {'Key': 'Version', 'Value': version}
        ]
    )
    
    instance_id = response['Instances'][0]['InstanceId']
    
    # Send Slack notification
    slack_response = requests.post(
        slack_webhook,
        json={
            'text': f"✅ Deployed {app_name} v{version}: {instance_id}"
        }
    )
    
    return {
        'deployment_id': instance_id,
        'notification_status': slack_response.status_code
    }
```

**Test Strategy:**

**1. Unit Test - Mock Everything:**

```python
from unittest.mock import patch, MagicMock
import pytest

@patch('deployment.requests.post')
@patch('deployment.boto3.client')
def test_deploy_application_success(mock_boto, mock_requests):
    """Unit test with all dependencies mocked."""
    # Setup AWS mock
    mock_ec2 = MagicMock()
    mock_boto.return_value = mock_ec2
    mock_ec2.run_instances.return_value = {
        'Instances': [{'InstanceId': 'i-12345'}]
    }
    
    # Setup Slack mock
    mock_slack_response = MagicMock()
    mock_slack_response.status_code = 200
    mock_requests.return_value = mock_slack_response
    
    # Execute
    result = deploy_application(
        'my-app',
        '1.0.0',
        'https://hooks.slack.com/test'
    )
    
    # Verify AWS call
    mock_ec2.run_instances.assert_called_once_with(
        ImageId='ami-12345',
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1,
        Tags=[
            {'Key': 'Name', 'Value': 'my-app'},
            {'Key': 'Version', 'Value': '1.0.0'}
        ]
    )
    
    # Verify Slack call
    mock_requests.assert_called_once_with(
        'https://hooks.slack.com/test',
        json={'text': '✅ Deployed my-app v1.0.0: i-12345'}
    )
    
    # Verify result
    assert result['deployment_id'] == 'i-12345'
    assert result['notification_status'] == 200


@patch('deployment.requests.post')
@patch('deployment.boto3.client')
def test_deploy_application_aws_failure(mock_boto, mock_requests):
    """Test AWS deployment failure."""
    # AWS raises exception
    mock_ec2 = MagicMock()
    mock_boto.return_value = mock_ec2
    mock_ec2.run_instances.side_effect = Exception("AWS Error")
    
    # Should propagate exception
    with pytest.raises(Exception, match="AWS Error"):
        deploy_application('my-app', '1.0.0', 'https://hooks.slack.com/test')
    
    # Slack should not be called
    mock_requests.assert_not_called()


@patch('deployment.requests.post')
@patch('deployment.boto3.client')
def test_deploy_application_slack_failure(mock_boto, mock_requests):
    """Test Slack notification failure."""
    # AWS succeeds
    mock_ec2 = MagicMock()
    mock_boto.return_value = mock_ec2
    mock_ec2.run_instances.return_value = {
        'Instances': [{'InstanceId': 'i-67890'}]
    }
    
    # Slack fails
    mock_slack_response = MagicMock()
    mock_slack_response.status_code = 500
    mock_requests.return_value = mock_slack_response
    
    # Should still return result (deployment succeeded)
    result = deploy_application('my-app', '1.0.0', 'https://hooks.slack.com/test')
    
    assert result['deployment_id'] == 'i-67890'
    assert result['notification_status'] == 500
```

**2. Integration Test - Real AWS (moto):**

```python
from moto import mock_aws
import os

@pytest.fixture
def aws_credentials():
    """Mock AWS credentials."""
    os.environ['AWS_ACCESS_KEY_ID'] = 'testing'
    os.environ['AWS_SECRET_ACCESS_KEY'] = 'testing'


@mock_aws
@patch('deployment.requests.post')
def test_deploy_application_integration(mock_requests, aws_credentials):
    """Integration test with real AWS (moto) and mocked Slack."""
    # Setup Slack mock
    mock_slack_response = MagicMock()
    mock_slack_response.status_code = 200
    mock_requests.return_value = mock_slack_response
    
    # Execute (real AWS via moto)
    result = deploy_application(
        'test-app',
        '2.0.0',
        'https://hooks.slack.com/test'
    )
    
    # Verify instance was created (can query AWS)
    import boto3
    ec2 = boto3.client('ec2', region_name='us-east-1')
    instances = ec2.describe_instances(
        InstanceIds=[result['deployment_id']]
    )
    
    # Check instance tags
    tags = instances['Reservations'][0]['Instances'][0]['Tags']
    assert {'Key': 'Name', 'Value': 'test-app'} in tags
    assert {'Key': 'Version', 'Value': '2.0.0'} in tags
    
    # Verify Slack notification
    mock_requests.assert_called_once()
```

**3. Separate Concerns - Test Each Component:**

```python
# Better design: Separate AWS and Slack concerns

class AWSDeployer:
    """Handles AWS deployment."""
    
    def deploy(self, app_name: str, version: str) -> str:
        ec2 = boto3.client('ec2', region_name='us-east-1')
        response = ec2.run_instances(...)
        return response['Instances'][0]['InstanceId']


class SlackNotifier:
    """Handles Slack notifications."""
    
    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url
    
    def notify(self, message: str) -> int:
        response = requests.post(self.webhook_url, json={'text': message})
        return response.status_code


def deploy_application(
    app_name: str,
    version: str,
    deployer: AWSDeployer,
    notifier: SlackNotifier
) -> dict:
    """Deploy with dependency injection."""
    instance_id = deployer.deploy(app_name, version)
    status = notifier.notify(f"✅ Deployed {app_name} v{version}: {instance_id}")
    return {'deployment_id': instance_id, 'notification_status': status}


# Now test each component separately
def test_aws_deployer():
    """Test AWS deployer in isolation."""
    with patch('boto3.client') as mock_boto:
        # Test deployer only
        pass

def test_slack_notifier():
    """Test Slack notifier in isolation."""
    with patch('requests.post') as mock_requests:
        # Test notifier only
        pass

def test_deploy_application_orchestration():
    """Test orchestration logic."""
    mock_deployer = Mock()
    mock_deployer.deploy.return_value = 'i-12345'
    
    mock_notifier = Mock()
    mock_notifier.notify.return_value = 200
    
    result = deploy_application('app', '1.0', mock_deployer, mock_notifier)
    assert result['deployment_id'] == 'i-12345'
```

**Why This Matters:**
- **Fast feedback**: Unit tests run in milliseconds
- **Isolated testing**: Test one thing at a time
- **Better design**: Dependency injection enables testing
- **Confidence**: Integration tests verify real behavior

---

### Key Takeaways - Mocking

✅ **Mock external dependencies** - APIs, databases, AWS

✅ **Patch where used** - Not where defined

✅ **Use moto for AWS** - Realistic AWS simulation

✅ **Verify interactions** - assert_called_with()

✅ **Test error cases** - side_effect for exceptions

✅ **Don't over-mock** - Balance with integration tests

✅ **Dependency injection** - Makes mocking easier

✅ **Use pytest-mock** - Cleaner than unittest.mock

---

*[Part 3 continues with Test Coverage, Property-Based Testing, and more...]*

## 3.4 Test Coverage and Its Limitations

Test coverage measures what percentage of your code is executed during tests. While useful, coverage is not the same as quality.

### Understanding Coverage Metrics

**Coverage Types:**

```python
def divide(a: float, b: float) -> float:
    """Divide a by b."""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b


# Test with 50% line coverage
def test_divide_happy_path():
    """Test division with valid inputs."""
    assert divide(10, 2) == 5
    # Lines 4-5 executed
    # Lines 2-3 NOT executed
    # Coverage: 50%


# Test with 100% line coverage
def test_divide_complete():
    """Test all code paths."""
    assert divide(10, 2) == 5  # Lines 4-5
    
    with pytest.raises(ValueError):
        divide(10, 0)  # Lines 2-3
    
    # All lines executed
    # Coverage: 100%
```

**Coverage Types:**
- **Line Coverage**: % of lines executed
- **Branch Coverage**: % of decision branches taken (if/else)
- **Function Coverage**: % of functions called
- **Statement Coverage**: % of statements executed

### Using pytest-cov

```bash
# Install
pip install pytest-cov

# Run tests with coverage
pytest --cov=mymodule tests/

# Generate HTML report
pytest --cov=mymodule --cov-report=html tests/

# View coverage for specific files
pytest --cov=deployment --cov-report=term-missing

# Fail if coverage below threshold
pytest --cov=mymodule --cov-fail-under=80
```

**pytest.ini configuration:**

```ini
[pytest]
addopts = 
    --cov=src
    --cov-report=html
    --cov-report=term-missing
    --cov-fail-under=80
    -v
```

**Coverage output:**

```
---------- coverage: platform linux, python 3.11 -----------
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
deployment.py               45      3    93%   23-25
infrastructure.py           67      0   100%
monitoring.py               34     10    71%   15, 42-50
-------------------------------------------------------
TOTAL                      146     13    91%
```

### Coverage Is Not Quality

```python
# ❌ 100% coverage but poor tests
def calculate_discount(price: float, customer_type: str) -> float:
    """Calculate discount based on customer type."""
    if customer_type == 'premium':
        return price * 0.8  # 20% off
    elif customer_type == 'regular':
        return price * 0.9  # 10% off
    else:
        return price  # No discount

def test_calculate_discount():
    """100% line coverage but doesn't verify correctness!"""
    calculate_discount(100, 'premium')
    calculate_discount(100, 'regular')
    calculate_discount(100, 'guest')
    # No assertions! Coverage is 100% but test is worthless


# ✅ 100% coverage with meaningful tests
def test_calculate_discount_premium():
    """Test premium discount."""
    assert calculate_discount(100, 'premium') == 80

def test_calculate_discount_regular():
    """Test regular discount."""
    assert calculate_discount(100, 'regular') == 90

def test_calculate_discount_guest():
    """Test no discount for guests."""
    assert calculate_discount(100, 'guest') == 100

def test_calculate_discount_unknown():
    """Test unknown customer type."""
    assert calculate_discount(100, 'unknown') == 100
```

**Coverage can't detect:**
- Missing functionality
- Logic errors (if tests don't assert correctly)
- Edge cases you haven't thought of
- Integration issues

### Effective Coverage Strategy

**What to aim for:**

```python
# Critical code: 95-100% coverage
def process_payment(amount: float, card: str) -> bool:
    """Process payment - critical functionality."""
    # Thoroughly test all paths
    pass

# Business logic: 85-95% coverage
def calculate_shipping_cost(weight: float, distance: float) -> float:
    """Calculate shipping cost - important logic."""
    pass

# Simple code: 70-80% coverage
def format_address(address: dict) -> str:
    """Format address for display - simple formatting."""
    pass

# Generated/trivial code: Exclude from coverage
class Config:
    def __repr__(self):
        return f"Config({self.value})"
```

**Configure coverage exclusions:**

```python
# .coveragerc
[run]
omit =
    */tests/*
    */migrations/*
    */__pycache__/*
    */venv/*

[report]
exclude_lines =
    # Standard pragma
    pragma: no cover
    
    # Don't complain about missing debug code
    def __repr__
    
    # Don't complain if tests don't hit defensive assertion code
    raise AssertionError
    raise NotImplementedError
    
    # Don't complain about abstract methods
    @abstractmethod
```

### Mutation Testing

Mutation testing verifies test quality by introducing bugs and checking if tests catch them.

```bash
# Install mutmut
pip install mutmut

# Run mutation testing
mutmut run

# View results
mutmut show
mutmut html
```

**Example:**

```python
# Original code
def is_adult(age: int) -> bool:
    """Check if person is adult."""
    return age >= 18

# Test
def test_is_adult():
    assert is_adult(18) is True
    assert is_adult(17) is False

# mutmut creates mutations:
# Mutation 1: return age > 18  (changes >= to >)
# Mutation 2: return age >= 17  (changes 18 to 17)
# Mutation 3: return age < 18   (changes >= to <)

# If test still passes with mutation, it's a "survived" mutant
# Goal: Kill all mutants (tests should fail for mutations)
```

### Frequently Asked Questions - Coverage

**Q1: What coverage percentage should I aim for?**

**A:** **80-90% is a good target, but focus on meaningful coverage.**

**Targets by code type:**
- **Critical paths** (payment, security): 95-100%
- **Business logic**: 85-95%
- **Infrastructure code**: 80-90%
- **Simple utilities**: 70-80%
- **Trivial code**: Exclude from coverage

```python
# Don't chase 100% coverage of trivial code
class Server:
    def __repr__(self):
        return f"Server({self.hostname})"  # Skip coverage

# DO aim for 100% coverage of critical code
def process_payment(amount, card):
    """Payment processing - test ALL paths."""
    if amount <= 0:
        raise ValueError("Invalid amount")  # Test this
    
    if not validate_card(card):
        raise ValueError("Invalid card")  # Test this
    
    try:
        result = charge_card(amount, card)  # Test this
    except PaymentError:
        log_failed_payment()  # Test this
        raise
    
    return result  # Test this
```

---

## 3.5 Advanced Testing Techniques

### Property-Based Testing

Property-based testing generates random test inputs to find edge cases you haven't considered.

```bash
# Install Hypothesis
pip install hypothesis
```

**Example-based testing (traditional):**

```python
def test_reverse_list_examples():
    """Test with specific examples."""
    assert reverse([1, 2, 3]) == [3, 2, 1]
    assert reverse([]) == []
    assert reverse([1]) == [1]
    # What about [1, 1, 1]? Or 1000 elements? Or negative numbers?
```

**Property-based testing:**

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_reverse_list_properties(lst):
    """Test properties that should always hold."""
    reversed_list = reverse(lst)
    
    # Property 1: Reversing twice gives original
    assert reverse(reversed_list) == lst
    
    # Property 2: Length preserved
    assert len(reversed_list) == len(lst)
    
    # Property 3: All elements present
    assert sorted(reversed_list) == sorted(lst)

# Hypothesis generates 100+ test cases automatically:
# [], [1], [1, 2], [0], [-5, 3, -2], [1, 1, 1], [very large list], etc.
```

**DevSecOps Example:**

```python
from hypothesis import given, strategies as st

# Function to test
def validate_cidr(cidr: str) -> bool:
    """Validate CIDR notation."""
    import re
    pattern = r'^(\d{1,3}\.){3}\d{1,3}/\d{1,2}$'
    if not re.match(pattern, cidr):
        return False
    
    ip, mask = cidr.split('/')
    octets = [int(x) for x in ip.split('.')]
    
    # Validate octets
    if not all(0 <= o <= 255 for o in octets):
        return False
    
    # Validate mask
    if not 0 <= int(mask) <= 32:
        return False
    
    return True


# Traditional tests
def test_validate_cidr_examples():
    """Test with examples."""
    assert validate_cidr('10.0.0.0/16') is True
    assert validate_cidr('192.168.1.0/24') is True
    assert validate_cidr('256.0.0.0/16') is False  # Invalid octet
    assert validate_cidr('10.0.0.0/33') is False  # Invalid mask


# Property-based test
@given(st.integers(min_value=0, max_value=255),
       st.integers(min_value=0, max_value=255),
       st.integers(min_value=0, max_value=255),
       st.integers(min_value=0, max_value=255),
       st.integers(min_value=0, max_value=32))
def test_valid_cidr_always_accepted(o1, o2, o3, o4, mask):
    """Valid CIDR should always be accepted."""
    cidr = f"{o1}.{o2}.{o3}.{o4}/{mask}"
    assert validate_cidr(cidr) is True


@given(st.integers(min_value=256, max_value=1000))
def test_invalid_octet_rejected(invalid_octet):
    """CIDR with invalid octet should be rejected."""
    cidr = f"{invalid_octet}.0.0.0/16"
    assert validate_cidr(cidr) is False


@given(st.integers(min_value=33, max_value=100))
def test_invalid_mask_rejected(invalid_mask):
    """CIDR with invalid mask should be rejected."""
    cidr = f"10.0.0.0/{invalid_mask}"
    assert validate_cidr(cidr) is False


# Hypothesis found a bug!
# It generated: "10.0.0.0/01" (leading zero in mask)
# validate_cidr returned True but should return False
```

**Stateful Testing:**

```python
from hypothesis.stateful import RuleBasedStateMachine, rule, invariant

class ServerStateMachine(RuleBasedStateMachine):
    """Test server state transitions."""
    
    def __init__(self):
        super().__init__()
        self.server = Server()
        self.expected_state = 'stopped'
    
    @rule()
    def start(self):
        """Start server."""
        self.server.start()
        self.expected_state = 'running'
    
    @rule()
    def stop(self):
        """Stop server."""
        self.server.stop()
        self.expected_state = 'stopped'
    
    @rule()
    def restart(self):
        """Restart server."""
        self.server.restart()
        self.expected_state = 'running'
    
    @invariant()
    def check_state_consistency(self):
        """State should always be consistent."""
        assert self.server.state == self.expected_state

# Run test
TestServer = ServerStateMachine.TestCase
```

### Parameterized Tests

Run same test with different inputs:

```python
import pytest

@pytest.mark.parametrize('instance_type,expected_price', [
    ('t2.micro', 0.0116),
    ('t2.small', 0.023),
    ('t2.medium', 0.0464),
    ('t2.large', 0.0928),
    ('t2.xlarge', 0.1856),
])
def test_instance_pricing(instance_type, expected_price):
    """Test pricing for different instance types."""
    calculator = CostCalculator()
    price = calculator.get_hourly_price(instance_type)
    assert price == pytest.approx(expected_price, rel=0.01)


# Multiple parameters
@pytest.mark.parametrize('region', ['us-east-1', 'eu-west-1', 'ap-south-1'])
@pytest.mark.parametrize('instance_type', ['t2.micro', 't2.small'])
def test_all_combinations(region, instance_type):
    """Test all region/instance combinations."""
    # Runs 3 × 2 = 6 tests
    result = provision_instance(region, instance_type)
    assert result is not None


# With pytest.param for test IDs
@pytest.mark.parametrize('cidr,expected', [
    pytest.param('10.0.0.0/16', True, id='valid_private'),
    pytest.param('192.168.1.0/24', True, id='valid_private_class_c'),
    pytest.param('256.0.0.0/16', False, id='invalid_octet'),
    pytest.param('10.0.0.0/33', False, id='invalid_mask'),
])
def test_cidr_validation(cidr, expected):
    """Test CIDR validation with clear test IDs."""
    assert validate_cidr(cidr) == expected
```

### Integration Testing

Integration tests verify components work together:

```python
import pytest
from testcontainers.postgres import PostgresContainer
from testcontainers.redis import RedisContainer

@pytest.fixture(scope='session')
def postgres():
    """Provide PostgreSQL container."""
    with PostgresContainer('postgres:14') as postgres:
        yield postgres

@pytest.fixture(scope='session')
def redis():
    """Provide Redis container."""
    with RedisContainer('redis:7') as redis:
        yield redis

@pytest.fixture
def database_connection(postgres):
    """Provide database connection."""
    import psycopg2
    conn = psycopg2.connect(postgres.get_connection_url())
    
    # Setup schema
    with conn.cursor() as cursor:
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS deployments (
                id SERIAL PRIMARY KEY,
                app_name VARCHAR(100),
                version VARCHAR(50),
                status VARCHAR(50),
                created_at TIMESTAMP DEFAULT NOW()
            )
        """)
    conn.commit()
    
    yield conn
    
    conn.close()

def test_save_and_retrieve_deployment(database_connection):
    """Integration test with real database."""
    # Save deployment
    with database_connection.cursor() as cursor:
        cursor.execute(
            "INSERT INTO deployments (app_name, version, status) VALUES (%s, %s, %s)",
            ('my-app', '1.0.0', 'success')
        )
    database_connection.commit()
    
    # Retrieve deployment
    with database_connection.cursor() as cursor:
        cursor.execute("SELECT app_name, version, status FROM deployments WHERE app_name = %s", ('my-app',))
        result = cursor.fetchone()
    
    assert result == ('my-app', '1.0.0', 'success')
```

### Performance Testing

```python
import pytest
import time

def test_deployment_performance():
    """Test deployment completes within time limit."""
    start = time.time()
    
    deploy_application('my-app', '1.0.0')
    
    duration = time.time() - start
    assert duration < 5.0, f"Deployment took {duration}s, expected < 5s"


# Using pytest-benchmark
def test_config_parsing_performance(benchmark):
    """Benchmark config parsing."""
    config_yaml = """
    environment: production
    replicas: 3
    resources:
      cpu: 200m
      memory: 512Mi
    """
    
    result = benchmark(parse_config, config_yaml)
    assert result['environment'] == 'production'
    
    # pytest-benchmark reports:
    # - Min/Max time
    # - Mean/Median
    # - Standard deviation


# Load testing
import concurrent.futures

def test_concurrent_health_checks():
    """Test handling concurrent health checks."""
    urls = [f'http://server-{i}.example.com/health' for i in range(100)]
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(check_health, url) for url in urls]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]
    
    # All health checks should succeed
    assert all(r['healthy'] for r in results)
```

### Snapshot Testing

Compare outputs against saved snapshots:

```python
# Install: pip install pytest-snapshot

def test_kubernetes_manifest_snapshot(snapshot):
    """Test Kubernetes manifest generation."""
    manifest = generate_k8s_deployment({
        'app': 'my-app',
        'version': '1.0.0',
        'replicas': 3
    })
    
    # First run: saves manifest to snapshot file
    # Subsequent runs: compares against saved snapshot
    snapshot.assert_match(manifest, 'deployment.yaml')
    
    # If manifest changes, test fails
    # Review changes, update snapshot if correct:
    # pytest --snapshot-update
```

### Final Summary - Part 3

Part 3 covered comprehensive testing for DevSecOps:

**✅ Complete Topics:**

**Section 3.1: Testing Fundamentals**
- Test types pyramid
- TDD cycle (Red-Green-Refactor)
- FIRST principles
- Test organization
- AAA pattern

**Section 3.2: pytest Framework**
- Basic usage
- Fixtures (setup/teardown)
- Parametrized tests
- Markers
- conftest.py

**Section 3.3: Mocking**
- Test doubles (dummy, stub, spy, mock, fake)
- unittest.mock
- Patching strategies
- Mocking AWS (moto)
- Mocking HTTP requests
- Mocking databases
- Mocking time

**Section 3.4: Test Coverage**
- Coverage metrics
- pytest-cov
- Coverage limitations
- Mutation testing
- Effective coverage strategy

**Section 3.5: Advanced Techniques**
- Property-based testing (Hypothesis)
- Stateful testing
- Parameterized tests
- Integration testing (testcontainers)
- Performance testing
- Snapshot testing

### Key Testing Principles for DevSecOps

✅ **Test pyramid**: 70% unit, 20% integration, 10% E2E

✅ **Fast feedback**: Unit tests in milliseconds

✅ **Mock external services**: Keep tests fast and reliable

✅ **Real databases for integration**: Use testcontainers

✅ **Property-based testing**: Find edge cases automatically

✅ **Coverage is a metric, not a goal**: Focus on quality

✅ **Test error paths**: Don't just test happy path

✅ **CI/CD integration**: Run tests on every commit

### Testing Checklist

Before deploying infrastructure code:

- [ ] Unit tests cover critical logic
- [ ] Integration tests verify AWS/cloud interaction
- [ ] Error handling tested
- [ ] Edge cases covered
- [ ] Performance acceptable
- [ ] Tests run in CI/CD
- [ ] Coverage > 80% for critical code
- [ ] No flaky tests
- [ ] Tests are fast (<5 minutes total)
- [ ] Rollback tested

### Recommended Tools

**Testing Frameworks:**
- pytest (primary)
- unittest (stdlib, if needed)

**Mocking:**
- unittest.mock (built-in)
- pytest-mock (pytest integration)
- moto (AWS mocking)
- requests-mock (HTTP mocking)

**Coverage:**
- pytest-cov (coverage)
- mutmut (mutation testing)

**Advanced:**
- Hypothesis (property-based)
- testcontainers (integration)
- pytest-benchmark (performance)
- locust (load testing)

### Next Steps

**To master testing:**

1. **Practice TDD** - Write tests first for new features
2. **Refactor legacy code** - Add tests, then refactor
3. **Review test quality** - Not just coverage percentage
4. **Learn from failures** - When tests miss bugs, improve tests
5. **Share knowledge** - Teach others testing techniques

**Continue learning:**
- Part 4: Debugging and Troubleshooting
- Part 5: Security in Python
- Part 6: DevSecOps Automation
- Part 7: Advanced Python Topics

---

*End of Part 3: Testing and Quality Assurance*

**Part 3 is now COMPLETE with 15,000+ words covering comprehensive testing strategies for DevSecOps engineers.**
