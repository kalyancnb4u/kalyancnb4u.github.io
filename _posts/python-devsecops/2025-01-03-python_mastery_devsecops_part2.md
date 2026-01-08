---
title: "Python for DevSecOps Part 2: Software Engineering Principles"
date: 2025-01-03 00:00:00 +0530
categories: [Python, DevSecOps]
tags: [Python, DevSecOps, Software Engineering, SOLID, Design-patterns, Architecture, Refactoring, Code-quality, Testing]
---

# Complete Python Mastery for DevSecOps Part 2: Software Engineering Principles

## Introduction

Welcome to Part 2 of the Python Mastery for DevSecOps series. While Part 1 covered Python fundamentals, Part 2 focuses on **writing production-grade, maintainable code** using software engineering principles.

**Why Software Engineering Principles Matter in DevSecOps:**

In DevSecOps, you're not just writing scripts - you're building critical infrastructure automation, deployment pipelines, and security tooling that must be:
- **Reliable**: One bug can take down production
- **Maintainable**: Your code will be modified hundreds of times
- **Testable**: Must verify correctness before deploying
- **Extensible**: Requirements change constantly
- **Secure**: Automation code has privileged access

**What You'll Learn:**

This part covers the software engineering skills that separate junior from senior engineers:
- SOLID principles and when to apply them
- Design patterns for real-world DevSecOps problems
- Code architecture that scales with complexity
- Refactoring techniques to improve existing code
- Code quality metrics and how to use them

**Who This Is For:**

- DevSecOps engineers who want to write better code
- Developers moving from scripting to software engineering
- Anyone responsible for maintaining infrastructure code
- Teams building internal platforms and tools

---

## 2.1 SOLID Principles

SOLID is an acronym for five design principles that make software more maintainable and flexible. These aren't strict rules - they're guidelines based on decades of experience.

### Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change. Each class should have one well-defined responsibility.

**Why It Matters:**
- Easier to understand (focused purpose)
- Easier to test (fewer dependencies)
- Easier to modify (changes are localized)
- Reduces coupling between components

#### Identifying Violations

Ask yourself: "What does this class do?" If your answer includes "and," you likely have multiple responsibilities.

```python
# ❌ BAD: Multiple responsibilities
class ServerManager:
    """Manages servers - does EVERYTHING."""
    
    def __init__(self):
        self.servers = []
    
    def add_server(self, server):
        """Add server to inventory."""
        self.servers.append(server)
    
    def provision_server(self, server_config):
        """Provision new server (AWS API calls)."""
        # Talks to AWS API
        pass
    
    def check_server_health(self, server):
        """Check server health (HTTP calls)."""
        # Makes HTTP requests
        pass
    
    def send_alert(self, message):
        """Send alert (email/Slack)."""
        # Sends notifications
        pass
    
    def generate_report(self):
        """Generate HTML report."""
        # HTML generation
        pass
    
    def save_to_database(self):
        """Persist server data."""
        # Database operations
        pass

# This class has at least 6 responsibilities:
# 1. Server inventory management
# 2. Cloud provisioning
# 3. Health checking
# 4. Alerting
# 5. Report generation
# 6. Data persistence
```

**Problems with this design:**
- Can't test health checking without mocking AWS, email, and database
- Changes to reporting affect server provisioning
- Can't reuse health checking in other contexts
- Impossible to understand without reading all methods

#### Refactoring to SRP

```python
# ✅ GOOD: Single Responsibility Per Class

class ServerInventory:
    """Manages server collection - ONLY inventory operations."""
    
    def __init__(self):
        self.servers = []
    
    def add(self, server: 'Server') -> None:
        """Add server to inventory."""
        self.servers.append(server)
    
    def remove(self, server: 'Server') -> None:
        """Remove server from inventory."""
        self.servers.remove(server)
    
    def find_by_id(self, server_id: str) -> 'Server':
        """Find server by ID."""
        return next((s for s in self.servers if s.id == server_id), None)
    
    def get_all(self) -> list['Server']:
        """Get all servers."""
        return self.servers.copy()


class CloudProvisioner:
    """Handles cloud infrastructure provisioning - ONLY provisioning."""
    
    def __init__(self, cloud_client):
        self.client = cloud_client
    
    def provision_server(self, config: dict) -> 'Server':
        """Provision new server in cloud."""
        response = self.client.create_instance(
            image_id=config['image_id'],
            instance_type=config['instance_type']
        )
        return Server(
            id=response['InstanceId'],
            ip=response['PrivateIpAddress']
        )
    
    def terminate_server(self, server_id: str) -> bool:
        """Terminate cloud server."""
        self.client.terminate_instances(InstanceIds=[server_id])
        return True


class HealthChecker:
    """Checks server health - ONLY health monitoring."""
    
    def check_http_health(self, url: str, timeout: int = 5) -> bool:
        """Check if HTTP endpoint is healthy."""
        try:
            response = requests.get(url, timeout=timeout)
            return 200 <= response.status_code < 300
        except requests.RequestException:
            return False
    
    def check_ssh_health(self, host: str, port: int = 22) -> bool:
        """Check if SSH is accessible."""
        import socket
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(5)
            result = sock.connect_ex((host, port))
            sock.close()
            return result == 0
        except socket.error:
            return False


class AlertService:
    """Sends alerts - ONLY notification handling."""
    
    def __init__(self, slack_webhook: str, email_config: dict):
        self.slack_webhook = slack_webhook
        self.email_config = email_config
    
    def send_slack_alert(self, message: str) -> None:
        """Send Slack alert."""
        requests.post(self.slack_webhook, json={'text': message})
    
    def send_email_alert(self, subject: str, body: str) -> None:
        """Send email alert."""
        # Email sending logic
        pass


class ServerReportGenerator:
    """Generates server reports - ONLY report creation."""
    
    def generate_html_report(self, servers: list['Server']) -> str:
        """Generate HTML report of servers."""
        html = "<html><body><h1>Server Report</h1><table>"
        for server in servers:
            html += f"<tr><td>{server.id}</td><td>{server.ip}</td></tr>"
        html += "</table></body></html>"
        return html
    
    def generate_csv_report(self, servers: list['Server']) -> str:
        """Generate CSV report of servers."""
        import csv
        import io
        output = io.StringIO()
        writer = csv.writer(output)
        writer.writerow(['ID', 'IP', 'Status'])
        for server in servers:
            writer.writerow([server.id, server.ip, server.status])
        return output.getvalue()


class ServerRepository:
    """Persists server data - ONLY data storage."""
    
    def __init__(self, database):
        self.db = database
    
    def save(self, server: 'Server') -> None:
        """Save server to database."""
        self.db.execute(
            "INSERT INTO servers (id, ip, status) VALUES (?, ?, ?)",
            (server.id, server.ip, server.status)
        )
    
    def find_by_id(self, server_id: str) -> 'Server':
        """Load server from database."""
        row = self.db.query("SELECT * FROM servers WHERE id = ?", (server_id,))
        return Server(id=row['id'], ip=row['ip'], status=row['status'])
```

**Benefits of SRP Design:**
- Each class is easy to understand
- Easy to test in isolation
- Changes are localized (modify health checking without touching provisioning)
- Can reuse components (use HealthChecker in multiple contexts)
- Clear dependencies (explicit in constructors)

#### Composing Single-Responsibility Classes

```python
# Orchestrator that uses single-responsibility components
class ServerOrchestrator:
    """Coordinates server operations using specialized components."""
    
    def __init__(
        self,
        inventory: ServerInventory,
        provisioner: CloudProvisioner,
        health_checker: HealthChecker,
        alert_service: AlertService,
        repository: ServerRepository
    ):
        self.inventory = inventory
        self.provisioner = provisioner
        self.health_checker = health_checker
        self.alert_service = alert_service
        self.repository = repository
    
    def provision_and_monitor_server(self, config: dict) -> 'Server':
        """
        High-level workflow: provision, monitor, and persist server.
        
        This orchestrator has ONE responsibility: coordinating the workflow.
        Each step delegates to a specialized component.
        """
        # Provision
        server = self.provisioner.provision_server(config)
        
        # Add to inventory
        self.inventory.add(server)
        
        # Wait for server to be ready
        import time
        for attempt in range(10):
            if self.health_checker.check_ssh_health(server.ip):
                break
            time.sleep(10)
        else:
            self.alert_service.send_slack_alert(
                f"Server {server.id} failed health check"
            )
            raise HealthCheckTimeout(f"Server {server.id} not healthy")
        
        # Persist
        self.repository.save(server)
        
        # Success notification
        self.alert_service.send_slack_alert(
            f"Server {server.id} provisioned successfully"
        )
        
        return server
```

#### DevSecOps Example: Deployment Pipeline

```python
# ❌ BAD: God class doing everything
class DeploymentPipeline:
    def deploy(self, app_name, version):
        # Validate version format
        # Pull code from Git
        # Run tests
        # Build Docker image
        # Push to registry
        # Update Kubernetes deployment
        # Run health checks
        # Send notifications
        # Update database
        # Generate reports
        pass  # 500+ lines of coupled code

# ✅ GOOD: Each component has single responsibility

class VersionValidator:
    """Validates version strings."""
    
    def validate(self, version: str) -> bool:
        """Check if version follows semantic versioning."""
        import re
        pattern = r'^\d+\.\d+\.\d+$'
        return bool(re.match(pattern, version))


class GitClient:
    """Handles Git operations."""
    
    def clone_repository(self, repo_url: str, branch: str, target_dir: str):
        """Clone Git repository."""
        import subprocess
        subprocess.run(
            ['git', 'clone', '-b', branch, repo_url, target_dir],
            check=True
        )


class TestRunner:
    """Runs test suites."""
    
    def run_tests(self, test_dir: str) -> bool:
        """Execute pytest and return success/failure."""
        import subprocess
        result = subprocess.run(['pytest', test_dir], capture_output=True)
        return result.returncode == 0


class DockerBuilder:
    """Builds Docker images."""
    
    def build_image(self, dockerfile_path: str, tag: str) -> str:
        """Build Docker image and return image ID."""
        import docker
        client = docker.from_env()
        image, logs = client.images.build(
            path=dockerfile_path,
            tag=tag
        )
        return image.id


class ContainerRegistry:
    """Pushes images to registry."""
    
    def push_image(self, image_tag: str) -> str:
        """Push image to registry and return digest."""
        import docker
        client = docker.from_env()
        client.images.push(image_tag)
        return f"pushed: {image_tag}"


class KubernetesDeployer:
    """Deploys to Kubernetes."""
    
    def update_deployment(
        self,
        deployment_name: str,
        image: str,
        namespace: str = 'default'
    ) -> bool:
        """Update Kubernetes deployment with new image."""
        from kubernetes import client, config
        config.load_kube_config()
        apps_v1 = client.AppsV1Api()
        
        # Update deployment
        deployment = apps_v1.read_namespaced_deployment(
            deployment_name, namespace
        )
        deployment.spec.template.spec.containers[0].image = image
        
        apps_v1.patch_namespaced_deployment(
            deployment_name, namespace, deployment
        )
        return True


class DeploymentHealthChecker:
    """Checks deployment health."""
    
    def wait_for_rollout(
        self,
        deployment_name: str,
        namespace: str = 'default',
        timeout: int = 300
    ) -> bool:
        """Wait for Kubernetes rollout to complete."""
        from kubernetes import client, config
        import time
        
        config.load_kube_config()
        apps_v1 = client.AppsV1Api()
        
        start_time = time.time()
        while time.time() - start_time < timeout:
            deployment = apps_v1.read_namespaced_deployment_status(
                deployment_name, namespace
            )
            
            if deployment.status.updated_replicas == deployment.spec.replicas:
                if deployment.status.ready_replicas == deployment.spec.replicas:
                    return True
            
            time.sleep(5)
        
        return False


class DeploymentNotifier:
    """Sends deployment notifications."""
    
    def notify_success(self, app_name: str, version: str) -> None:
        """Send success notification."""
        message = f"✅ {app_name} v{version} deployed successfully"
        self._send_to_slack(message)
    
    def notify_failure(self, app_name: str, version: str, error: str) -> None:
        """Send failure notification."""
        message = f"❌ {app_name} v{version} deployment failed: {error}"
        self._send_to_slack(message)
    
    def _send_to_slack(self, message: str) -> None:
        """Send message to Slack."""
        import os
        import requests
        webhook = os.getenv('SLACK_WEBHOOK')
        requests.post(webhook, json={'text': message})


class DeploymentRecorder:
    """Records deployment history."""
    
    def record_deployment(
        self,
        app_name: str,
        version: str,
        status: str,
        metadata: dict
    ) -> None:
        """Record deployment in database."""
        import datetime
        deployment = {
            'app_name': app_name,
            'version': version,
            'status': status,
            'timestamp': datetime.datetime.utcnow().isoformat(),
            'metadata': metadata
        }
        # Save to database
        self._save_to_db(deployment)
    
    def _save_to_db(self, deployment: dict) -> None:
        """Save deployment record."""
        # Database persistence logic
        pass


# Orchestrator coordinates all components
class DeploymentOrchestrator:
    """
    Orchestrates deployment workflow.
    
    Single responsibility: coordinate the deployment process.
    Delegates all specific tasks to specialized components.
    """
    
    def __init__(
        self,
        validator: VersionValidator,
        git_client: GitClient,
        test_runner: TestRunner,
        docker_builder: DockerBuilder,
        registry: ContainerRegistry,
        k8s_deployer: KubernetesDeployer,
        health_checker: DeploymentHealthChecker,
        notifier: DeploymentNotifier,
        recorder: DeploymentRecorder
    ):
        self.validator = validator
        self.git = git_client
        self.tester = test_runner
        self.builder = docker_builder
        self.registry = registry
        self.deployer = k8s_deployer
        self.health_checker = health_checker
        self.notifier = notifier
        self.recorder = recorder
    
    def deploy(
        self,
        app_name: str,
        version: str,
        repo_url: str,
        deployment_name: str
    ) -> bool:
        """
        Execute complete deployment workflow.
        
        This method is readable because each step is a single,
        clear method call to a specialized component.
        """
        try:
            # Validate
            if not self.validator.validate(version):
                raise ValueError(f"Invalid version: {version}")
            
            # Clone code
            self.git.clone_repository(repo_url, f"v{version}", '/tmp/build')
            
            # Test
            if not self.tester.run_tests('/tmp/build'):
                raise RuntimeError("Tests failed")
            
            # Build
            image_tag = f"{app_name}:{version}"
            self.builder.build_image('/tmp/build', image_tag)
            
            # Push
            self.registry.push_image(image_tag)
            
            # Deploy
            self.deployer.update_deployment(deployment_name, image_tag)
            
            # Health check
            if not self.health_checker.wait_for_rollout(deployment_name):
                raise RuntimeError("Deployment health check failed")
            
            # Record and notify
            self.recorder.record_deployment(
                app_name, version, 'success', {}
            )
            self.notifier.notify_success(app_name, version)
            
            return True
            
        except Exception as e:
            # Record failure and notify
            self.recorder.record_deployment(
                app_name, version, 'failed', {'error': str(e)}
            )
            self.notifier.notify_failure(app_name, version, str(e))
            raise
```

**Why This Design Is Better:**

1. **Testable**: Each component can be tested independently
2. **Reusable**: Can use `DockerBuilder` in other workflows
3. **Maintainable**: Changes to Git operations don't affect Docker building
4. **Readable**: Orchestrator clearly shows the deployment flow
5. **Flexible**: Easy to swap implementations (different test runner, different registry)

#### When SRP Goes Too Far

```python
# ❌ TOO GRANULAR: Over-application of SRP
class ServerIdValidator:
    """Only validates server IDs."""
    def validate(self, server_id): pass

class ServerIpValidator:
    """Only validates IPs."""
    def validate(self, ip): pass

class ServerNameValidator:
    """Only validates names."""
    def validate(self, name): pass

# ✅ BALANCED: Cohesive validation
class ServerValidator:
    """Validates all server attributes (cohesive responsibility)."""
    
    def validate_id(self, server_id): pass
    def validate_ip(self, ip): pass
    def validate_name(self, name): pass
    def validate_all(self, server): pass
```

**Guidelines:**
- Group related methods that change for the same reason
- Don't split classes just for the sake of it
- Balance granularity with cohesion

### Frequently Asked Questions - SRP

**Q1: How do I know if a class has multiple responsibilities?**

**A:** Use these tests:

1. **"And" Test**: If describing the class uses "and," it likely has multiple responsibilities
   - ❌ "ServerManager provisions servers AND checks health AND sends alerts"
   - ✅ "ServerProvisioner provisions servers"

2. **Reason to Change Test**: List reasons the class might change
   - If more than one reason → multiple responsibilities

3. **Method Grouping Test**: Can methods be grouped into distinct categories?
   - If yes → each group might be a separate responsibility

```python
class ServerManager:
    # Group 1: CRUD operations
    def add_server(self): pass
    def remove_server(self): pass
    
    # Group 2: Cloud provisioning (DIFFERENT responsibility)
    def provision_aws_server(self): pass
    def terminate_aws_server(self): pass
    
    # Group 3: Monitoring (DIFFERENT responsibility)
    def check_health(self): pass
    def collect_metrics(self): pass
```

**Q2: Doesn't SRP create too many small classes?**

**A:** More classes is fine if each has a clear purpose. Benefits:
- **Easier to understand**: One focused class vs one complex class
- **Easier to test**: Test one thing at a time
- **Easier to reuse**: Can use components independently

Trade-off: More files to navigate, but modern IDEs handle this well.

**Rule of thumb**: If a class is hard to name concisely, it probably has multiple responsibilities.

**Q3: Should every class have exactly one public method?**

**A:** No! SRP is about **one reason to change**, not one method.

```python
# ✅ CORRECT: Multiple methods, single responsibility
class ServerInventory:
    """Single responsibility: manage server collection."""
    
    def add(self, server): pass
    def remove(self, server): pass
    def find_by_id(self, id): pass
    def get_all(self): pass
    
    # All methods relate to inventory management (one responsibility)
```

### Interview Questions - SRP

**Question 1: What is the Single Responsibility Principle? Give an example from your DevSecOps work where you applied it.**

**Difficulty:** Junior

**Answer:**

**Single Responsibility Principle**: A class should have only one reason to change, meaning it should have only one well-defined responsibility.

**Example from DevSecOps:**

```python
# Before SRP: Monitoring class doing too much
class MonitoringService:
    def collect_metrics(self):
        """Collect metrics from servers."""
        pass
    
    def store_metrics(self):
        """Store metrics in database."""
        pass
    
    def send_alerts(self):
        """Send alerts via email/Slack."""
        pass
    
    def generate_dashboard(self):
        """Generate HTML dashboard."""
        pass

# Problems:
# - Changes to database affect alerting code
# - Can't test metric collection without email configured
# - Can't reuse alerting in other contexts

# After SRP: Separated responsibilities
class MetricsCollector:
    """Single responsibility: collect metrics from infrastructure."""
    def collect_from_servers(self, servers): pass
    def collect_from_containers(self, containers): pass

class MetricsStore:
    """Single responsibility: persist metrics."""
    def save(self, metrics): pass
    def query(self, time_range): pass

class AlertManager:
    """Single responsibility: send alerts."""
    def send_slack_alert(self, message): pass
    def send_email_alert(self, subject, body): pass

class DashboardGenerator:
    """Single responsibility: generate visualizations."""
    def generate_html(self, metrics): pass
    def generate_json(self, metrics): pass

# Benefits:
# - Each class is easy to test in isolation
# - Can modify alerting without touching metric collection
# - Can reuse AlertManager in deployment pipeline
# - Clear dependencies (explicit in constructors)
```

**Why This Matters in DevSecOps:**
- Infrastructure code is long-lived and frequently modified
- Multiple team members need to understand and modify code
- Testing is critical (can't afford bugs in production automation)
- Components often need to be reused across different tools

**Follow-up:** How would you refactor a 1000-line deployment script to follow SRP?

---

**Question 2: You inherit a class that handles server provisioning, configuration, monitoring, and alerting. How would you refactor it to follow SRP? What challenges might you face?**

**Difficulty:** Mid-Level

**Answer:**

**Refactoring Strategy:**

**Step 1: Identify Responsibilities**
```python
# Original god class
class ServerManager:
    def provision_server(self):
        # AWS API calls
        pass
    
    def configure_server(self):
        # Ansible playbooks
        pass
    
    def monitor_server(self):
        # Prometheus queries
        pass
    
    def send_alerts(self):
        # Email/Slack
        pass
    
    def update_inventory(self):
        # Database operations
        pass

# Identified responsibilities:
# 1. Cloud provisioning (AWS)
# 2. Configuration management (Ansible)
# 3. Monitoring (Prometheus)
# 4. Alerting (notifications)
# 5. Inventory management (database)
```

**Step 2: Extract Each Responsibility**
```python
class CloudProvisioner:
    """Handles cloud infrastructure provisioning."""
    def __init__(self, aws_client):
        self.aws = aws_client
    
    def provision_ec2(self, config): pass
    def terminate_ec2(self, instance_id): pass

class ConfigurationManager:
    """Manages server configuration."""
    def __init__(self, ansible_runner):
        self.ansible = ansible_runner
    
    def apply_playbook(self, server, playbook): pass
    def verify_configuration(self, server): pass

class MonitoringCollector:
    """Collects monitoring data."""
    def __init__(self, prometheus_client):
        self.prometheus = prometheus_client
    
    def query_metrics(self, server): pass
    def check_health(self, server): pass

class NotificationService:
    """Sends notifications."""
    def __init__(self, slack_webhook, smtp_config):
        self.slack = slack_webhook
        self.smtp = smtp_config
    
    def send_slack(self, message): pass
    def send_email(self, subject, body): pass

class ServerInventory:
    """Manages server inventory."""
    def __init__(self, database):
        self.db = database
    
    def add_server(self, server): pass
    def update_server(self, server): pass
    def get_server(self, server_id): pass
```

**Step 3: Create Orchestrator**
```python
class ServerOrchestrator:
    """
    Coordinates server lifecycle operations.
    
    This class has ONE responsibility: orchestrating the workflow.
    It delegates all actual work to specialized components.
    """
    
    def __init__(
        self,
        provisioner: CloudProvisioner,
        config_manager: ConfigurationManager,
        monitoring: MonitoringCollector,
        notifications: NotificationService,
        inventory: ServerInventory
    ):
        self.provisioner = provisioner
        self.config_manager = config_manager
        self.monitoring = monitoring
        self.notifications = notifications
        self.inventory = inventory
    
    def create_and_configure_server(self, config: dict) -> Server:
        """High-level workflow for server creation."""
        try:
            # Provision
            server = self.provisioner.provision_ec2(config)
            
            # Configure
            self.config_manager.apply_playbook(server, config['playbook'])
            
            # Update inventory
            self.inventory.add_server(server)
            
            # Verify
            if not self.monitoring.check_health(server):
                raise HealthCheckFailed(f"Server {server.id} unhealthy")
            
            # Notify
            self.notifications.send_slack(
                f"Server {server.id} created successfully"
            )
            
            return server
            
        except Exception as e:
            self.notifications.send_email(
                "Server Creation Failed",
                f"Failed to create server: {e}"
            )
            raise
```

**Challenges You Might Face:**

**1. Circular Dependencies**
```python
# Problem: Classes depend on each other
class CloudProvisioner:
    def __init__(self, inventory):  # Needs inventory
        self.inventory = inventory

class ServerInventory:
    def __init__(self, provisioner):  # Needs provisioner
        self.provisioner = provisioner

# Solution: Use dependency injection and interfaces
class CloudProvisioner:
    def __init__(self, inventory: InventoryProtocol):
        self.inventory = inventory
    
    # Or better: Don't have provisioner depend on inventory
    # Let orchestrator handle coordination
```

**2. Shared State**
```python
# Problem: Multiple classes modify same data
class ConfigManager:
    def configure(self, server):
        server.status = 'configured'  # Modifying server

class Monitor:
    def check(self, server):
        server.status = 'healthy'  # Also modifying server

# Solution: Immutable data or clear ownership
# Option 1: Only ServerInventory modifies servers
# Option 2: Return new states instead of modifying
```

**3. Performance Overhead**
```python
# Concern: More objects = more overhead?
# Reality: Negligible in Python for most use cases
# Benefit: Clean design is worth minor overhead

# If performance critical, profile first, then optimize
```

**4. Existing Tests Break**
```python
# Problem: Tests coupled to old structure
def test_server_manager():
    manager = ServerManager()
    manager.provision_server()  # No longer exists!

# Solution: Refactor tests incrementally
# 1. Extract new classes
# 2. Keep old class as facade temporarily
# 3. Update tests gradually
# 4. Remove old class when all tests updated
```

**Refactoring Steps (Safe Process):**

1. **Add tests** for existing behavior (if missing)
2. **Extract one responsibility** at a time
3. **Keep old class** as facade initially
4. **Update tests** incrementally
5. **Remove facade** when migration complete

```python
# Facade pattern during migration
class ServerManager:
    """
    Legacy class - deprecated, use ServerOrchestrator.
    Kept temporarily for backward compatibility.
    """
    
    def __init__(self):
        # Internally uses new classes
        self.orchestrator = ServerOrchestrator(
            CloudProvisioner(...),
            ConfigurationManager(...),
            # ...
        )
    
    def provision_server(self, config):
        """Deprecated: Use ServerOrchestrator."""
        warnings.warn(
            "ServerManager.provision_server is deprecated",
            DeprecationWarning
        )
        return self.orchestrator.create_and_configure_server(config)
```

**Why This Matters in DevSecOps:**
- Infrastructure code evolves rapidly (new cloud services, tools)
- Multiple engineers collaborate on same codebase
- Need to add features without breaking existing functionality
- Must be able to test changes in isolation

**Follow-up:** How would you handle this refactoring in a production system with zero downtime?

---

### Key Takeaways - SRP

✅ **One reason to change** - Not one method, one responsibility

✅ **Easier to test** - Isolated components are simple to test

✅ **Easier to understand** - Clear, focused classes

✅ **Easier to modify** - Changes don't ripple through codebase

✅ **Easier to reuse** - Components work in different contexts

✅ **Balance granularity** - Don't create excessive tiny classes

✅ **Use orchestrators** - Coordinate single-responsibility components

✅ **Refactor incrementally** - Extract one responsibility at a time

---

## 2.2 Open/Closed Principle (OCP)

**Definition:** Software entities should be **open for extension** but **closed for modification**.

- **Open for extension**: Can add new functionality
- **Closed for modification**: Don't modify existing code

**Why It Matters:**
- Add features without breaking existing code
- Reduces regression risk (existing code untouched)
- Enables plugin architectures
- Supports multiple implementations

### Understanding OCP

The goal is to **add new behavior without modifying existing, working code**.

```python
# ❌ BAD: Violates OCP
class DeploymentService:
    def deploy(self, app, strategy):
        if strategy == 'blue-green':
            # Blue-green deployment logic
            print("Blue-green deployment")
            # ... 50 lines ...
        elif strategy == 'canary':
            # Canary deployment logic
            print("Canary deployment")
            # ... 50 lines ...
        elif strategy == 'rolling':
            # Rolling deployment logic
            print("Rolling deployment")
            # ... 50 lines ...
        # To add new strategy, must MODIFY this method!
        # Risk: might break existing strategies
```

**Problems:**
- Adding "A/B testing" strategy requires modifying existing code
- Risk of breaking blue-green or canary deployments
- Method grows indefinitely
- Hard to test strategies in isolation

### Applying OCP with Strategy Pattern

```python
# ✅ GOOD: Open for extension, closed for modification

from abc import ABC, abstractmethod
from typing import Protocol

# Define interface/protocol
class DeploymentStrategy(Protocol):
    """Protocol for deployment strategies."""
    
    def deploy(self, app: 'Application', version: str) -> bool:
        """Execute deployment."""
        ...
    
    def rollback(self, app: 'Application') -> bool:
        """Rollback deployment."""
        ...


# Concrete strategies (each in separate file)
class BlueGreenDeployment:
    """Blue-green deployment strategy."""
    
    def deploy(self, app: 'Application', version: str) -> bool:
        """Deploy using blue-green strategy."""
        print(f"Blue-green deploying {app.name} v{version}")
        # 1. Deploy to green environment
        # 2. Run health checks
        # 3. Switch traffic to green
        # 4. Decommission blue
        return True
    
    def rollback(self, app: 'Application') -> bool:
        """Rollback by switching traffic back."""
        print("Rolling back to blue environment")
        return True


class CanaryDeployment:
    """Canary deployment strategy."""
    
    def __init__(self, canary_percentage: int = 10):
        self.canary_percentage = canary_percentage
    
    def deploy(self, app: 'Application', version: str) -> bool:
        """Deploy using canary strategy."""
        print(f"Canary deploying {app.name} v{version}")
        # 1. Deploy to canary servers (10% traffic)
        # 2. Monitor metrics
        # 3. If healthy, promote to all servers
        # 4. If issues, rollback
        return True
    
    def rollback(self, app: 'Application') -> bool:
        """Rollback canary deployment."""
        print("Rolling back canary deployment")
        return True


class RollingDeployment:
    """Rolling update deployment strategy."""
    
    def __init__(self, batch_size: int = 1):
        self.batch_size = batch_size
    
    def deploy(self, app: 'Application', version: str) -> bool:
        """Deploy using rolling update."""
        print(f"Rolling deployment of {app.name} v{version}")
        # 1. Update servers in batches
        # 2. Wait for health checks between batches
        # 3. Continue until all servers updated
        return True
    
    def rollback(self, app: 'Application') -> bool:
        """Rollback rolling deployment."""
        print("Rolling back deployment")
        return True


# DeploymentService is CLOSED for modification
# To add new strategy, just create new class!
class DeploymentService:
    """
    Deployment service using strategy pattern.
    
    This class is CLOSED: adding new strategies doesn't require
    modifying this code. Just pass a new strategy implementation!
    """
    
    def __init__(self, strategy: DeploymentStrategy):
        self.strategy = strategy
    
    def deploy_application(self, app: 'Application', version: str) -> bool:
        """
        Deploy using configured strategy.
        
        No modification needed to add new strategies!
        """
        try:
            return self.strategy.deploy(app, version)
        except Exception as e:
            logger.error(f"Deployment failed: {e}")
            return self.strategy.rollback(app)


# Usage - different strategies without modifying DeploymentService
blue_green_service = DeploymentService(BlueGreenDeployment())
blue_green_service.deploy_application(app, '1.0.0')

canary_service = DeploymentService(CanaryDeployment(canary_percentage=20))
canary_service.deploy_application(app, '1.1.0')

rolling_service = DeploymentService(RollingDeployment(batch_size=2))
rolling_service.deploy_application(app, '1.2.0')

# NEW STRATEGY: A/B Testing (no changes to existing code!)
class ABTestingDeployment:
    """A/B testing deployment strategy."""
    
    def __init__(self, variant_a_percentage: int = 50):
        self.variant_a_percentage = variant_a_percentage
    
    def deploy(self, app: 'Application', version: str) -> bool:
        """Deploy using A/B testing."""
        print(f"A/B testing deployment: {self.variant_a_percentage}% variant A")
        # A/B testing logic
        return True
    
    def rollback(self, app: 'Application') -> bool:
        """Rollback A/B test."""
        return True

# Use new strategy without modifying DeploymentService!
ab_service = DeploymentService(ABTestingDeployment(variant_a_percentage=50))
ab_service.deploy_application(app, '2.0.0')
```

**Benefits:**
- Added A/B testing without touching existing code ✅
- Existing strategies unaffected (no regression risk) ✅
- Each strategy testable in isolation ✅
- Can add strategies in separate files/modules ✅

### OCP with Abstract Base Classes

```python
from abc import ABC, abstractmethod
from typing import Dict, Any

# Base class defines interface
class CloudProvider(ABC):
    """
    Abstract base class for cloud providers.
    
    New providers can be added without modifying existing code.
    """
    
    @abstractmethod
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision compute instance, return instance ID."""
        pass
    
    @abstractmethod
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate instance."""
        pass
    
    @abstractmethod
    def get_instance_status(self, instance_id: str) -> str:
        """Get instance status."""
        pass


# Concrete implementation 1
class AWSProvider(CloudProvider):
    """AWS cloud provider implementation."""
    
    def __init__(self, region: str):
        import boto3
        self.ec2 = boto3.client('ec2', region_name=region)
    
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision EC2 instance."""
        response = self.ec2.run_instances(
            ImageId=config['ami_id'],
            InstanceType=config['instance_type'],
            MinCount=1,
            MaxCount=1
        )
        return response['Instances'][0]['InstanceId']
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate EC2 instance."""
        self.ec2.terminate_instances(InstanceIds=[instance_id])
        return True
    
    def get_instance_status(self, instance_id: str) -> str:
        """Get EC2 instance status."""
        response = self.ec2.describe_instance_status(
            InstanceIds=[instance_id]
        )
        if response['InstanceStatuses']:
            return response['InstanceStatuses'][0]['InstanceState']['Name']
        return 'unknown'


# Concrete implementation 2
class GCPProvider(CloudProvider):
    """Google Cloud Platform provider implementation."""
    
    def __init__(self, project: str, zone: str):
        from google.cloud import compute_v1
        self.compute = compute_v1.InstancesClient()
        self.project = project
        self.zone = zone
    
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision GCE instance."""
        # GCP provisioning logic
        instance_name = config['name']
        # ... create instance ...
        return instance_name
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate GCE instance."""
        # GCP termination logic
        return True
    
    def get_instance_status(self, instance_id: str) -> str:
        """Get GCE instance status."""
        # GCP status check logic
        return 'running'


# Multi-cloud orchestrator - CLOSED for modification
class MultiCloudOrchestrator:
    """
    Orchestrates across multiple cloud providers.
    
    Adding Azure, DigitalOcean, etc. doesn't require modifying this class!
    """
    
    def __init__(self, providers: Dict[str, CloudProvider]):
        self.providers = providers
    
    def provision_in_cloud(
        self,
        cloud: str,
        config: Dict[str, Any]
    ) -> str:
        """Provision instance in specified cloud."""
        if cloud not in self.providers:
            raise ValueError(f"Unknown cloud provider: {cloud}")
        
        return self.providers[cloud].provision_instance(config)
    
    def provision_multi_cloud(
        self,
        config: Dict[str, Any]
    ) -> Dict[str, str]:
        """Provision identical instances across all clouds."""
        instance_ids = {}
        
        for cloud_name, provider in self.providers.items():
            instance_id = provider.provision_instance(config)
            instance_ids[cloud_name] = instance_id
        
        return instance_ids


# Usage
orchestrator = MultiCloudOrchestrator({
    'aws': AWSProvider('us-east-1'),
    'gcp': GCPProvider('my-project', 'us-central1-a')
})

# Provision in AWS
aws_instance = orchestrator.provision_in_cloud('aws', {
    'ami_id': 'ami-12345',
    'instance_type': 't2.micro'
})

# Provision across both clouds
instances = orchestrator.provision_multi_cloud({
    'ami_id': 'ami-12345',  # AWS config
    'instance_type': 't2.micro'
})

# NEW PROVIDER: Azure (no changes to orchestrator!)
class AzureProvider(CloudProvider):
    """Azure cloud provider implementation."""
    
    def __init__(self, subscription_id: str):
        from azure.mgmt.compute import ComputeManagementClient
        self.compute_client = ComputeManagementClient(...)
        self.subscription_id = subscription_id
    
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision Azure VM."""
        # Azure provisioning logic
        return 'vm-instance-id'
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate Azure VM."""
        return True
    
    def get_instance_status(self, instance_id: str) -> str:
        """Get Azure VM status."""
        return 'running'

# Add Azure without modifying MultiCloudOrchestrator!
orchestrator = MultiCloudOrchestrator({
    'aws': AWSProvider('us-east-1'),
    'gcp': GCPProvider('my-project', 'us-central1-a'),
    'azure': AzureProvider('subscription-123')  # Just add it!
})
```

### OCP with Plugin Architecture

```python
from typing import Dict, List, Callable
import importlib
import inspect

class MonitoringPlugin(ABC):
    """Base class for monitoring plugins."""
    
    @abstractmethod
    def collect_metrics(self) -> Dict[str, float]:
        """Collect metrics."""
        pass
    
    @abstractmethod
    def get_plugin_name(self) -> str:
        """Return plugin name."""
        pass


class CPUMonitorPlugin(MonitoringPlugin):
    """CPU monitoring plugin."""
    
    def collect_metrics(self) -> Dict[str, float]:
        """Collect CPU metrics."""
        import psutil
        return {
            'cpu_percent': psutil.cpu_percent(interval=1),
            'cpu_count': psutil.cpu_count()
        }
    
    def get_plugin_name(self) -> str:
        return 'cpu_monitor'


class MemoryMonitorPlugin(MonitoringPlugin):
    """Memory monitoring plugin."""
    
    def collect_metrics(self) -> Dict[str, float]:
        """Collect memory metrics."""
        import psutil
        mem = psutil.virtual_memory()
        return {
            'memory_percent': mem.percent,
            'memory_used_gb': mem.used / (1024 ** 3),
            'memory_total_gb': mem.total / (1024 ** 3)
        }
    
    def get_plugin_name(self) -> str:
        return 'memory_monitor'


class DiskMonitorPlugin(MonitoringPlugin):
    """Disk monitoring plugin."""
    
    def collect_metrics(self) -> Dict[str, float]:
        """Collect disk metrics."""
        import psutil
        disk = psutil.disk_usage('/')
        return {
            'disk_percent': disk.percent,
            'disk_used_gb': disk.used / (1024 ** 3),
            'disk_total_gb': disk.total / (1024 ** 3)
        }
    
    def get_plugin_name(self) -> str:
        return 'disk_monitor'


# Monitoring system is CLOSED for modification
class MonitoringSystem:
    """
    Plugin-based monitoring system.
    
    New plugins can be added dynamically without modifying this class!
    """
    
    def __init__(self):
        self.plugins: Dict[str, MonitoringPlugin] = {}
    
    def register_plugin(self, plugin: MonitoringPlugin) -> None:
        """Register a monitoring plugin."""
        plugin_name = plugin.get_plugin_name()
        self.plugins[plugin_name] = plugin
        logger.info(f"Registered plugin: {plugin_name}")
    
    def unregister_plugin(self, plugin_name: str) -> None:
        """Unregister a plugin."""
        if plugin_name in self.plugins:
            del self.plugins[plugin_name]
            logger.info(f"Unregistered plugin: {plugin_name}")
    
    def collect_all_metrics(self) -> Dict[str, Dict[str, float]]:
        """Collect metrics from all registered plugins."""
        all_metrics = {}
        
        for plugin_name, plugin in self.plugins.items():
            try:
                metrics = plugin.collect_metrics()
                all_metrics[plugin_name] = metrics
            except Exception as e:
                logger.error(f"Plugin {plugin_name} failed: {e}")
        
        return all_metrics
    
    def auto_discover_plugins(self, plugins_dir: str) -> None:
        """
        Automatically discover and load plugins from directory.
        
        This enables adding plugins without code changes!
        """
        import os
        import sys
        
        sys.path.insert(0, plugins_dir)
        
        for filename in os.listdir(plugins_dir):
            if filename.endswith('.py') and not filename.startswith('_'):
                module_name = filename[:-3]
                
                try:
                    module = importlib.import_module(module_name)
                    
                    # Find all MonitoringPlugin subclasses
                    for name, obj in inspect.getmembers(module):
                        if (inspect.isclass(obj) and
                            issubclass(obj, MonitoringPlugin) and
                            obj != MonitoringPlugin):
                            
                            # Instantiate and register
                            plugin = obj()
                            self.register_plugin(plugin)
                            
                except Exception as e:
                    logger.error(f"Failed to load plugin {module_name}: {e}")


# Usage
monitoring = MonitoringSystem()

# Register plugins
monitoring.register_plugin(CPUMonitorPlugin())
monitoring.register_plugin(MemoryMonitorPlugin())
monitoring.register_plugin(DiskMonitorPlugin())

# Collect metrics
metrics = monitoring.collect_all_metrics()
print(metrics)
# {
#     'cpu_monitor': {'cpu_percent': 45.2, 'cpu_count': 8},
#     'memory_monitor': {'memory_percent': 67.1, ...},
#     'disk_monitor': {'disk_percent': 34.5, ...}
# }

# NEW PLUGIN: Network monitoring (separate file: network_monitor.py)
class NetworkMonitorPlugin(MonitoringPlugin):
    """Network monitoring plugin."""
    
    def collect_metrics(self) -> Dict[str, float]:
        """Collect network metrics."""
        import psutil
        net = psutil.net_io_counters()
        return {
            'bytes_sent_mb': net.bytes_sent / (1024 ** 2),
            'bytes_recv_mb': net.bytes_recv / (1024 ** 2),
            'packets_sent': net.packets_sent,
            'packets_recv': net.packets_recv
        }
    
    def get_plugin_name(self) -> str:
        return 'network_monitor'

# Just drop plugin file in plugins directory!
# Or register manually:
monitoring.register_plugin(NetworkMonitorPlugin())
```

**Benefits of Plugin Architecture:**
- Add plugins without recompiling/restarting ✅
- Third parties can create plugins ✅
- Enable/disable features dynamically ✅
- Isolate plugin failures ✅

### When OCP Doesn't Apply

Not everything should be extensible:

```python
# ❌ Over-engineering: Making everything extensible
class String:
    """Don't need extensible string class."""
    def __init__(self, value):
        self._strategy = StringStorageStrategy()  # Overkill!
        self.value = value

# ✅ KISS: Use built-in str
text = "hello"
```

**Apply OCP when:**
- High likelihood of needing variations
- Plugin architecture makes sense
- Multiple implementations exist
- Behavior varies by deployment

**Don't apply OCP when:**
- Requirements are stable
- Only one implementation needed
- Adds unnecessary complexity

### Frequently Asked Questions - OCP

**Q1: Doesn't OCP mean I can never modify code?**

**A:** No! You can modify code, just avoid modifying existing working code when adding features.

**Modify existing code for:**
- Bug fixes
- Performance improvements
- Refactoring (improving design)

**Don't modify existing code for:**
- Adding new variants/strategies
- Supporting new platforms
- Adding optional features

```python
# ❌ Modifying existing code to add feature
class Logger:
    def log(self, message, destination):
        if destination == 'file':
            # Write to file
            pass
        elif destination == 'slack':  # Added this
            # Send to Slack - MODIFIED existing method
            pass

# ✅ OCP: Extend without modifying
class Logger:
    def __init__(self, handler: LogHandler):
        self.handler = handler
    
    def log(self, message):
        self.handler.write(message)  # No modification needed

# New handler added
class SlackLogHandler(LogHandler):
    def write(self, message):
        # Send to Slack
        pass
```

**Q2: How do I know what to make extensible?**

**A:** Look for variation points:

**Signs you need OCP:**
- If-else chains checking types/strategies
- Multiple implementations of same concept
- Requirements likely to add variations
- Plugin opportunities

```python
# Sign: If-else chain
if deployment_type == 'blue-green':
    # ...
elif deployment_type == 'canary':
    # ...
elif deployment_type == 'rolling':
    # ...
# Future: Will add more types!

# Refactor: Strategy pattern (OCP)
```

**Q3: Isn't OCP just the Strategy pattern?**

**A:** Strategy pattern is one way to achieve OCP, but not the only way.

**OCP techniques:**
1. **Strategy pattern**: Interchangeable algorithms
2. **Template method**: Subclass-defined behavior
3. **Plugin architecture**: Dynamically loaded extensions
4. **Event-driven**: Handlers registered for events
5. **Dependency injection**: Pass implementations at runtime

### Interview Questions - OCP

**Question 1: Explain the Open/Closed Principle and give an example where violating it caused problems in production.**

**Difficulty:** Junior

**Answer:**

**Open/Closed Principle**: Software should be **open for extension** (can add features) but **closed for modification** (don't change existing code).

**Real-World Problem:**

```python
# Production monitoring system (violated OCP)
class AlertSystem:
    def send_alert(self, message, channel):
        if channel == 'email':
            # Send email
            smtp_client.send(message)
        elif channel == 'slack':
            # Send Slack message
            slack_client.send(message)
        elif channel == 'pagerduty':
            # Send PagerDuty alert
            pagerduty_client.trigger(message)
        # ... 10 more elif blocks ...

# What went wrong:
# 1. Added SMS alerts → Modified this method
# 2. Introduced bug in Slack integration (typo)
# 3. Production alerts stopped working
# 4. Incident went unnoticed for 2 hours
# 5. Violated OCP: Modifications broke existing functionality
```

**How OCP Would Have Prevented This:**

```python
# Proper design following OCP
class AlertHandler(ABC):
    @abstractmethod
    def send(self, message): pass

class EmailAlertHandler(AlertHandler):
    def send(self, message):
        smtp_client.send(message)

class SlackAlertHandler(AlertHandler):
    def send(self, message):
        slack_client.send(message)

class AlertSystem:
    def __init__(self):
        self.handlers = []
    
    def register_handler(self, handler: AlertHandler):
        self.handlers.append(handler)
    
    def send_alert(self, message):
        for handler in self.handlers:
            try:
                handler.send(message)
            except Exception as e:
                logger.error(f"Handler failed: {e}")
                # Other handlers still work!

# Adding SMS alerts (NEW file, NO modifications to existing code)
class SMSAlertHandler(AlertHandler):
    def send(self, message):
        sms_client.send(message)

# Register without touching existing code
alert_system.register_handler(SMSAlertHandler())
```

**Why This Matters:**
- New SMS handler is in separate file ✅
- Bug in SMS won't affect Slack/Email ✅
- Can test SMS in isolation ✅
- Existing handlers proven working (unchanged) ✅
- Failed handler doesn't break others ✅

**Follow-up:** How would you migrate existing code to follow OCP without breaking production?

---

**Question 2: You need to add support for deploying to a new cloud provider (Azure) in an infrastructure automation tool. The current code uses if-else to handle AWS and GCP. How would you refactor this to follow OCP?**

**Difficulty:** Mid-Level

**Answer:**

**Current Code (Violates OCP):**

```python
class InfrastructureManager:
    def provision_server(self, provider, config):
        if provider == 'aws':
            # AWS-specific code (boto3)
            import boto3
            ec2 = boto3.client('ec2')
            response = ec2.run_instances(
                ImageId=config['image_id'],
                InstanceType=config['instance_type']
            )
            return response['Instances'][0]['InstanceId']
            
        elif provider == 'gcp':
            # GCP-specific code (google-cloud)
            from google.cloud import compute_v1
            client = compute_v1.InstancesClient()
            # ... GCP provisioning ...
            return instance_name
        
        else:
            raise ValueError(f"Unknown provider: {provider}")
    
    # Problem: Adding Azure requires modifying this method
    # Risk: Might break existing AWS/GCP provisioning
```

**Refactored to Follow OCP:**

**Step 1: Define Interface**
```python
from abc import ABC, abstractmethod
from typing import Dict, Any

class CloudProvider(ABC):
    """
    Abstract base class for cloud providers.
    
    New providers implement this interface without modifying existing code.
    """
    
    @abstractmethod
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """
        Provision compute instance.
        
        Args:
            config: Provider-specific configuration
            
        Returns:
            Instance identifier
        """
        pass
    
    @abstractmethod
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate instance."""
        pass
    
    @abstractmethod
    def get_instance_info(self, instance_id: str) -> Dict[str, Any]:
        """Get instance details."""
        pass
```

**Step 2: Implement Existing Providers**
```python
class AWSProvider(CloudProvider):
    """AWS cloud provider implementation."""
    
    def __init__(self, region: str):
        import boto3
        self.ec2 = boto3.client('ec2', region_name=region)
        self.region = region
    
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision EC2 instance."""
        response = self.ec2.run_instances(
            ImageId=config['image_id'],
            InstanceType=config.get('instance_type', 't2.micro'),
            MinCount=1,
            MaxCount=1,
            TagSpecifications=[{
                'ResourceType': 'instance',
                'Tags': [{'Key': 'Name', 'Value': config.get('name', 'unnamed')}]
            }]
        )
        return response['Instances'][0]['InstanceId']
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate EC2 instance."""
        self.ec2.terminate_instances(InstanceIds=[instance_id])
        return True
    
    def get_instance_info(self, instance_id: str) -> Dict[str, Any]:
        """Get EC2 instance details."""
        response = self.ec2.describe_instances(InstanceIds=[instance_id])
        instance = response['Reservations'][0]['Instances'][0]
        return {
            'id': instance['InstanceId'],
            'state': instance['State']['Name'],
            'type': instance['InstanceType'],
            'ip': instance.get('PublicIpAddress', 'N/A')
        }


class GCPProvider(CloudProvider):
    """GCP cloud provider implementation."""
    
    def __init__(self, project_id: str, zone: str):
        from google.cloud import compute_v1
        self.compute = compute_v1.InstancesClient()
        self.project_id = project_id
        self.zone = zone
    
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision GCE instance."""
        from google.cloud import compute_v1
        
        instance = compute_v1.Instance()
        instance.name = config['name']
        instance.machine_type = f"zones/{self.zone}/machineTypes/{config['instance_type']}"
        
        # Configure disk
        disk = compute_v1.AttachedDisk()
        disk.boot = True
        disk.auto_delete = True
        disk.initialize_params = compute_v1.AttachedDiskInitializeParams()
        disk.initialize_params.source_image = config['image_id']
        instance.disks = [disk]
        
        # Configure network
        network_interface = compute_v1.NetworkInterface()
        network_interface.name = "global/networks/default"
        instance.network_interfaces = [network_interface]
        
        operation = self.compute.insert(
            project=self.project_id,
            zone=self.zone,
            instance_resource=instance
        )
        
        return config['name']
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate GCE instance."""
        self.compute.delete(
            project=self.project_id,
            zone=self.zone,
            instance=instance_id
        )
        return True
    
    def get_instance_info(self, instance_id: str) -> Dict[str, Any]:
        """Get GCE instance details."""
        instance = self.compute.get(
            project=self.project_id,
            zone=self.zone,
            instance=instance_id
        )
        return {
            'id': instance.name,
            'state': instance.status,
            'type': instance.machine_type.split('/')[-1],
            'ip': instance.network_interfaces[0].network_i_p if instance.network_interfaces else 'N/A'
        }
```

**Step 3: NEW Provider (Azure) - No Modifications to Existing Code!**
```python
class AzureProvider(CloudProvider):
    """
    Azure cloud provider implementation.
    
    This is a NEW file, existing AWS/GCP code is UNTOUCHED!
    """
    
    def __init__(self, subscription_id: str, resource_group: str, location: str):
        from azure.identity import DefaultAzureCredential
        from azure.mgmt.compute import ComputeManagementClient
        
        credential = DefaultAzureCredential()
        self.compute_client = ComputeManagementClient(credential, subscription_id)
        self.resource_group = resource_group
        self.location = location
    
    def provision_instance(self, config: Dict[str, Any]) -> str:
        """Provision Azure VM."""
        from azure.mgmt.compute.models import (
            VirtualMachine, HardwareProfile, StorageProfile,
            OSProfile, NetworkProfile, ImageReference
        )
        
        vm_name = config['name']
        
        vm_parameters = VirtualMachine(
            location=self.location,
            hardware_profile=HardwareProfile(vm_size=config['instance_type']),
            storage_profile=StorageProfile(
                image_reference=ImageReference(
                    publisher='Canonical',
                    offer='UbuntuServer',
                    sku='18.04-LTS',
                    version='latest'
                )
            ),
            os_profile=OSProfile(
                computer_name=vm_name,
                admin_username=config.get('admin_username', 'azureuser'),
                admin_password=config.get('admin_password')
            ),
            network_profile=NetworkProfile(
                network_interfaces=[{
                    'id': config['network_interface_id']
                }]
            )
        )
        
        async_vm_creation = self.compute_client.virtual_machines.begin_create_or_update(
            self.resource_group,
            vm_name,
            vm_parameters
        )
        
        vm = async_vm_creation.result()
        return vm.name
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate Azure VM."""
        async_vm_delete = self.compute_client.virtual_machines.begin_delete(
            self.resource_group,
            instance_id
        )
        async_vm_delete.wait()
        return True
    
    def get_instance_info(self, instance_id: str) -> Dict[str, Any]:
        """Get Azure VM details."""
        vm = self.compute_client.virtual_machines.get(
            self.resource_group,
            instance_id
        )
        return {
            'id': vm.name,
            'state': vm.provisioning_state,
            'type': vm.hardware_profile.vm_size,
            'ip': 'N/A'  # Would need to query network interface
        }
```

**Step 4: Infrastructure Manager (CLOSED for modification)**
```python
class InfrastructureManager:
    """
    Multi-cloud infrastructure manager.
    
    This class is CLOSED: adding Azure didn't require any changes here!
    """
    
    def __init__(self):
        self.providers: Dict[str, CloudProvider] = {}
    
    def register_provider(self, name: str, provider: CloudProvider) -> None:
        """Register a cloud provider."""
        self.providers[name] = provider
        logger.info(f"Registered cloud provider: {name}")
    
    def provision_server(
        self,
        provider_name: str,
        config: Dict[str, Any]
    ) -> str:
        """
        Provision server on specified cloud provider.
        
        No modification needed to add new providers!
        """
        if provider_name not in self.providers:
            raise ValueError(f"Unknown provider: {provider_name}")
        
        provider = self.providers[provider_name]
        
        try:
            instance_id = provider.provision_instance(config)
            logger.info(
                f"Provisioned instance {instance_id} on {provider_name}"
            )
            return instance_id
        except Exception as e:
            logger.error(f"Provisioning failed on {provider_name}: {e}")
            raise
    
    def provision_multi_cloud(
        self,
        config: Dict[str, Any],
        provider_names: list[str]
    ) -> Dict[str, str]:
        """Provision identical instances across multiple clouds."""
        instances = {}
        
        for provider_name in provider_names:
            try:
                instance_id = self.provision_server(provider_name, config)
                instances[provider_name] = instance_id
            except Exception as e:
                logger.error(f"Failed to provision on {provider_name}: {e}")
        
        return instances
```

**Step 5: Usage**
```python
# Initialize manager
infra = InfrastructureManager()

# Register providers
infra.register_provider('aws', AWSProvider('us-east-1'))
infra.register_provider('gcp', GCPProvider('my-project', 'us-central1-a'))
infra.register_provider('azure', AzureProvider('subscription-id', 'resource-group', 'eastus'))

# Provision on AWS
aws_config = {
    'name': 'web-server-1',
    'image_id': 'ami-12345',
    'instance_type': 't2.micro'
}
aws_instance = infra.provision_server('aws', aws_config)

# Provision on Azure (no code changes to InfrastructureManager!)
azure_config = {
    'name': 'web-server-2',
    'instance_type': 'Standard_B1s',
    'admin_username': 'admin',
    'admin_password': 'SecurePass123!',
    'network_interface_id': '/subscriptions/...'
}
azure_instance = infra.provision_server('azure', azure_config)

# Provision across all clouds
multi_cloud_instances = infra.provision_multi_cloud(
    aws_config,
    ['aws', 'gcp', 'azure']
)
```

**Benefits of This Refactoring:**

1. **Azure added without touching existing code** ✅
   - AWS and GCP code unchanged (no regression risk)
   - Azure in separate file
   
2. **Easy to test in isolation** ✅
   - Each provider testable independently
   - Mock CloudProvider interface for InfrastructureManager tests
   
3. **Easy to add more providers** ✅
   - DigitalOcean, Linode, etc. just implement interface
   
4. **Configuration-driven** ✅
   - Can load providers from config file
   - Enable/disable providers at runtime
   
5. **Fail independently** ✅
   - Azure failing doesn't affect AWS/GCP

**Migration Strategy (Safe Production Rollout):**

```python
# Phase 1: Add new code alongside old (no breaking changes)
class InfrastructureManagerV2(InfrastructureManager):
    """New implementation with OCP."""
    pass

# Phase 2: Deprecate old method
class InfrastructureManager:
    def provision_server(self, provider, config):
        warnings.warn(
            "provision_server is deprecated, use InfrastructureManagerV2",
            DeprecationWarning
        )
        # Old implementation...

# Phase 3: Migrate callers incrementally

# Phase 4: Remove old implementation after migration complete
```

**Why This Matters in DevSecOps:**
- Infrastructure code changes frequently (new cloud features, providers)
- Must maintain multi-cloud support
- Cannot afford downtime from bad deployments
- Need to test each provider independently
- Enables gradual migration to new providers

**Follow-up:** How would you handle provider-specific features that don't fit the common interface?

---

### Key Takeaways - OCP

✅ **Extend without modifying** - Add features via new code, not changes

✅ **Strategy pattern** - Common way to achieve OCP

✅ **Abstract interfaces** - Define contracts for extensions

✅ **Plugin architecture** - Load extensions dynamically

✅ **Reduces regression risk** - Existing code unchanged, less testing needed

✅ **Don't over-engineer** - Apply OCP where variation is likely

✅ **Use for variation points** - If-else chains signal need for OCP

✅ **Separate files** - New strategies in separate modules

---

*[Continuing with Liskov Substitution Principle in next section...]*
## 2.3 Liskov Substitution Principle (LSP)

**Definition:** Objects of a superclass should be replaceable with objects of a subclass without breaking the application.

**In simpler terms:** Subtypes must be substitutable for their base types. If S is a subtype of T, then objects of type T can be replaced with objects of type S without altering program correctness.

**Why It Matters:**
- Enables polymorphism without surprises
- Maintains behavioral contracts
- Prevents subtle bugs from inheritance
- Ensures substitutability in frameworks

### Understanding LSP

LSP violations occur when a subclass changes the expected behavior of the parent class in a way that breaks client code.

```python
# ❌ BAD: LSP Violation
class Server:
    """Base server class."""
    
    def start(self) -> bool:
        """Start server, return True on success."""
        # Start logic
        return True
    
    def get_uptime(self) -> int:
        """Return uptime in seconds."""
        return self.uptime


class DatabaseServer(Server):
    """Database server - VIOLATES LSP!"""
    
    def start(self) -> bool:
        """
        Start database server.
        
        VIOLATION: Raises exception instead of returning bool!
        """
        if not self._check_disk_space():
            raise RuntimeError("Insufficient disk space")  # Unexpected!
        return True
    
    def get_uptime(self) -> int:
        """
        VIOLATION: Returns None instead of int when stopped!
        """
        if not self.is_running:
            return None  # Unexpected! Should return int
        return self.uptime


# Client code breaks
def restart_servers(servers: list[Server]):
    """Restart all servers - expects Server interface."""
    for server in servers:
        try:
            # Expects start() to return bool, not raise exception
            if server.start():
                print(f"Uptime: {server.get_uptime()}")  # Expects int, might get None!
        except Exception:
            # Shouldn't need to catch exceptions from start()
            print("Unexpected exception!")

servers = [Server(), DatabaseServer()]  # Mixed types
restart_servers(servers)  # BREAKS on DatabaseServer!
```

**Problems:**
- `DatabaseServer.start()` raises exception (parent returns bool)
- `DatabaseServer.get_uptime()` returns None (parent returns int)
- Client code written for `Server` breaks with `DatabaseServer`

### Fixing LSP Violations

```python
# ✅ GOOD: LSP Compliant
class Server:
    """Base server class with clear contract."""
    
    def start(self) -> bool:
        """
        Start server.
        
        Returns:
            True if started successfully, False otherwise.
            
        Note: Does NOT raise exceptions. Use get_last_error() for details.
        """
        try:
            self._do_start()
            return True
        except Exception as e:
            self.last_error = str(e)
            return False
    
    def get_uptime(self) -> int:
        """
        Return uptime in seconds.
        
        Returns:
            0 if server is stopped, actual uptime otherwise.
        """
        if not self.is_running:
            return 0
        return self.uptime
    
    def get_last_error(self) -> str:
        """Return last error message, empty string if none."""
        return getattr(self, 'last_error', '')
    
    def _do_start(self) -> None:
        """Internal start method, can raise exceptions."""
        # Implementation
        pass


class DatabaseServer(Server):
    """Database server - LSP compliant."""
    
    def _do_start(self) -> None:
        """
        Start database server.
        
        COMPLIANT: Raises exceptions here (internal method).
        Public start() method handles them properly.
        """
        if not self._check_disk_space():
            raise RuntimeError("Insufficient disk space")
        
        # Database-specific startup
        self._initialize_database()
        self._start_listeners()
    
    def get_uptime(self) -> int:
        """
        COMPLIANT: Always returns int, returns 0 when stopped.
        """
        if not self.is_running:
            return 0  # Consistent with parent
        return self.uptime


# Client code works perfectly
def restart_servers(servers: list[Server]):
    """Restart all servers - works with any Server subclass."""
    for server in servers:
        if server.start():
            print(f"Server started, uptime: {server.get_uptime()}s")
        else:
            print(f"Failed to start: {server.get_last_error()}")

# Works with mixed server types!
servers = [Server(), DatabaseServer(), WebServer(), CacheServer()]
restart_servers(servers)  # All servers work identically
```

**Key LSP Rules:**

1. **Preconditions cannot be strengthened**
2. **Postconditions cannot be weakened**
3. **Invariants must be preserved**
4. **Return types must be compatible**
5. **Exceptions must be compatible**

### LSP Violation: Rectangle-Square Problem

Classic example of LSP violation:

```python
# ❌ CLASSIC LSP VIOLATION
class Rectangle:
    """Rectangle with independent width and height."""
    
    def __init__(self, width: int, height: int):
        self._width = width
        self._height = height
    
    def set_width(self, width: int):
        """Set width."""
        self._width = width
    
    def set_height(self, height: int):
        """Set height."""
        self._height = height
    
    def get_area(self) -> int:
        """Calculate area."""
        return self._width * self._height


class Square(Rectangle):
    """
    Square inherits from Rectangle.
    VIOLATES LSP!
    """
    
    def __init__(self, side: int):
        super().__init__(side, side)
    
    def set_width(self, width: int):
        """
        VIOLATION: Changing width also changes height!
        Breaks Rectangle's invariant that width and height are independent.
        """
        self._width = width
        self._height = width  # Coupled!
    
    def set_height(self, height: int):
        """
        VIOLATION: Changing height also changes width!
        """
        self._width = height
        self._height = height  # Coupled!


# Client code breaks
def test_resize(rect: Rectangle):
    """Test rectangle resizing - expects Rectangle behavior."""
    rect.set_width(5)
    rect.set_height(4)
    
    # Expects area = 5 * 4 = 20
    assert rect.get_area() == 20, f"Expected 20, got {rect.get_area()}"

rect = Rectangle(0, 0)
test_resize(rect)  # ✅ Works: area = 20

square = Square(0)
test_resize(square)  # ❌ FAILS: area = 16 (4*4), not 20!
# Square changed behavior - LSP violated!
```

**Why This Violates LSP:**

Square changed the behavior of `set_width()` and `set_height()`. Code written for Rectangle breaks with Square.

**Solutions:**

```python
# ✅ SOLUTION 1: Composition, not inheritance
class Rectangle:
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height
    
    def get_area(self) -> int:
        return self.width * self.height


class Square:
    """Square doesn't inherit from Rectangle."""
    
    def __init__(self, side: int):
        self.side = side
    
    def get_area(self) -> int:
        return self.side * self.side


# ✅ SOLUTION 2: Common interface
class Shape(ABC):
    @abstractmethod
    def get_area(self) -> int:
        pass


class Rectangle(Shape):
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height
    
    def get_area(self) -> int:
        return self.width * self.height


class Square(Shape):
    def __init__(self, side: int):
        self.side = side
    
    def get_area(self) -> int:
        return self.side * self.side


# Client code uses Shape interface
def print_area(shape: Shape):
    """Works with any Shape."""
    print(f"Area: {shape.get_area()}")

print_area(Rectangle(5, 4))  # ✅ Works
print_area(Square(4))  # ✅ Works
```

### DevSecOps Example: Deployment Strategies

```python
# ❌ BAD: LSP Violation
class DeploymentStrategy(ABC):
    """Base deployment strategy."""
    
    @abstractmethod
    def deploy(self, app: str, version: str) -> bool:
        """
        Deploy application.
        
        Returns:
            True on success, False on failure.
        """
        pass
    
    @abstractmethod
    def rollback(self) -> bool:
        """Rollback deployment."""
        pass


class BlueGreenDeployment(DeploymentStrategy):
    """Blue-green deployment - LSP compliant."""
    
    def deploy(self, app: str, version: str) -> bool:
        """Deploy using blue-green strategy."""
        # Blue-green logic
        return True
    
    def rollback(self) -> bool:
        """Rollback to blue environment."""
        return True


class CanaryDeployment(DeploymentStrategy):
    """
    Canary deployment - VIOLATES LSP!
    """
    
    def deploy(self, app: str, version: str) -> bool:
        """
        VIOLATION: Requires manual approval, blocks indefinitely!
        Base class doesn't document blocking behavior.
        """
        # Deploy to canary servers
        print("Deployed to canary servers")
        
        # VIOLATION: Blocking call, waits for manual approval
        approval = wait_for_manual_approval()  # Blocks forever!
        
        if not approval:
            self.rollback()
            return False
        
        # Full deployment
        return True
    
    def rollback(self) -> bool:
        """Rollback canary deployment."""
        return True


# Client code breaks
def deploy_all_apps(apps: list[str], strategy: DeploymentStrategy):
    """Deploy all apps using given strategy."""
    for app in apps:
        # Expects deploy() to complete in reasonable time
        if strategy.deploy(app, 'v1.0.0'):
            print(f"{app} deployed")
        else:
            print(f"{app} failed")

# Works fine with BlueGreen
deploy_all_apps(['app1', 'app2'], BlueGreenDeployment())

# HANGS with Canary (blocks on manual approval)!
deploy_all_apps(['app1', 'app2'], CanaryDeployment())  # ❌ Never completes!
```

**Fixed Version:**

```python
# ✅ GOOD: LSP Compliant
class DeploymentStrategy(ABC):
    """
    Base deployment strategy.
    
    Contract:
    - deploy() completes automatically
    - rollback() can be called at any time
    """
    
    @abstractmethod
    def deploy(self, app: str, version: str) -> bool:
        """
        Deploy application automatically.
        
        Returns:
            True on success, False on failure.
            
        Note: Must complete without manual intervention.
        """
        pass
    
    @abstractmethod
    def rollback(self) -> bool:
        """Rollback deployment."""
        pass


class CanaryDeploymentWithAutoPromotion(DeploymentStrategy):
    """
    Canary deployment with automatic promotion.
    LSP COMPLIANT: No manual approval required.
    """
    
    def __init__(self, canary_duration_seconds: int = 300):
        self.canary_duration = canary_duration_seconds
    
    def deploy(self, app: str, version: str) -> bool:
        """
        Deploy using canary strategy with auto-promotion.
        
        COMPLIANT: Completes automatically after monitoring period.
        """
        # Deploy to canary servers
        deploy_to_canary(app, version)
        
        # Monitor for errors (automatic)
        import time
        time.sleep(self.canary_duration)
        
        # Check error rate
        error_rate = get_error_rate(app)
        if error_rate > 0.01:  # > 1% errors
            self.rollback()
            return False
        
        # Auto-promote to all servers
        promote_to_all_servers(app, version)
        return True
    
    def rollback(self) -> bool:
        """Rollback canary deployment."""
        rollback_canary()
        return True


# Client code works with all strategies
def deploy_all_apps(apps: list[str], strategy: DeploymentStrategy):
    """Deploy all apps using given strategy."""
    for app in apps:
        if strategy.deploy(app, 'v1.0.0'):
            print(f"{app} deployed")
        else:
            print(f"{app} failed")

# Both work identically!
deploy_all_apps(['app1', 'app2'], BlueGreenDeployment())
deploy_all_apps(['app1', 'app2'], CanaryDeploymentWithAutoPromotion())
```

**For Manual Approval, Use Separate Interface:**

```python
class ManualApprovalDeployment(ABC):
    """
    Deployment strategy requiring manual approval.
    
    Different interface than DeploymentStrategy!
    """
    
    @abstractmethod
    async def deploy_and_wait_approval(
        self,
        app: str,
        version: str,
        approval_callback: Callable[[str], bool]
    ) -> bool:
        """Deploy and wait for manual approval."""
        pass
```

### Invariants Must Be Preserved

```python
# ❌ BAD: Violates invariants
class BankAccount:
    """Bank account with balance invariant: balance >= 0."""
    
    def __init__(self, balance: float):
        if balance < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = balance
    
    def withdraw(self, amount: float) -> bool:
        """Withdraw money, maintain balance >= 0."""
        if amount > self._balance:
            return False
        self._balance -= amount
        return True
    
    def get_balance(self) -> float:
        """Get current balance."""
        return self._balance


class PremiumAccount(BankAccount):
    """
    Premium account with overdraft.
    VIOLATES LSP: Breaks invariant!
    """
    
    def __init__(self, balance: float, overdraft_limit: float):
        super().__init__(balance)
        self.overdraft_limit = overdraft_limit
    
    def withdraw(self, amount: float) -> bool:
        """
        VIOLATION: Allows negative balance (breaks invariant)!
        """
        if amount > self._balance + self.overdraft_limit:
            return False
        self._balance -= amount  # Can go negative!
        return True


# Client code breaks
def process_accounts(accounts: list[BankAccount]):
    """Process accounts - expects balance >= 0."""
    for account in accounts:
        account.withdraw(1000)
        
        # Expects balance >= 0 (invariant from BankAccount)
        assert account.get_balance() >= 0, "Balance went negative!"

accounts = [BankAccount(500), PremiumAccount(500, 1000)]
process_accounts(accounts)  # ❌ Assertion fails on PremiumAccount!
```

**Fixed Version:**

```python
# ✅ GOOD: Preserves invariant
class Account(ABC):
    """Base account - no invariant about positive balance."""
    
    @abstractmethod
    def withdraw(self, amount: float) -> bool:
        pass
    
    @abstractmethod
    def get_balance(self) -> float:
        pass


class StandardAccount(Account):
    """Standard account - balance must be >= 0."""
    
    def __init__(self, balance: float):
        if balance < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = balance
    
    def withdraw(self, amount: float) -> bool:
        """Withdraw, maintain balance >= 0."""
        if amount > self._balance:
            return False
        self._balance -= amount
        return True
    
    def get_balance(self) -> float:
        return self._balance


class PremiumAccount(Account):
    """Premium account - balance can go negative (overdraft)."""
    
    def __init__(self, balance: float, overdraft_limit: float):
        self._balance = balance
        self.overdraft_limit = overdraft_limit
    
    def withdraw(self, amount: float) -> bool:
        """Withdraw with overdraft protection."""
        if amount > self._balance + self.overdraft_limit:
            return False
        self._balance -= amount  # Can go negative
        return True
    
    def get_balance(self) -> float:
        return self._balance


# Client code that needs positive balance uses StandardAccount
def process_standard_accounts(accounts: list[StandardAccount]):
    """Process standard accounts - balance always >= 0."""
    for account in accounts:
        account.withdraw(1000)
        assert account.get_balance() >= 0  # ✅ Always true

# Generic code uses Account interface
def process_all_accounts(accounts: list[Account]):
    """Process any account type."""
    for account in accounts:
        account.withdraw(1000)
        print(f"Balance: {account.get_balance()}")  # May be negative
```

### Frequently Asked Questions - LSP

**Q1: How is LSP different from just "using inheritance correctly"?**

**A:** LSP is more specific - it's about **behavioral substitutability**, not just structural inheritance.

```python
# Structurally correct (inherits properly)
class Bird:
    def fly(self): pass

class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("Penguins can't fly!")

# ❌ But violates LSP behaviorally
def make_bird_fly(bird: Bird):
    bird.fly()  # Expects all Birds can fly

make_bird_fly(Bird())  # ✅ Works
make_bird_fly(Penguin())  # ❌ Throws exception!

# ✅ LSP compliant
class Bird(ABC):
    @abstractmethod
    def move(self): pass

class FlyingBird(Bird):
    def move(self):
        self.fly()
    
    def fly(self): pass

class Penguin(Bird):
    def move(self):
        self.swim()
    
    def swim(self): pass
```

**Q2: Can I ever change behavior in subclasses?**

**A:** Yes, but changes must be **compatible**:

✅ **Allowed:**
- Provide more specific implementation
- Return more specific type (covariant)
- Accept more general parameters (contravariant)
- Add optional features

❌ **Not Allowed:**
- Raise new exceptions
- Change return type incompatibly
- Require stronger preconditions
- Weaken postconditions

```python
# ✅ GOOD: More specific, still compatible
class Logger:
    def log(self, message: str) -> None:
        print(message)

class StructuredLogger(Logger):
    def log(self, message: str) -> None:
        """More specific implementation."""
        import json
        print(json.dumps({'message': message, 'timestamp': time.time()}))
        # Still logs the message - compatible behavior

# ❌ BAD: Incompatible change
class QuietLogger(Logger):
    def log(self, message: str) -> None:
        """VIOLATION: Doesn't log at all!"""
        pass  # Silent - breaks contract
```

### Interview Questions - LSP

**Question: What is the Liskov Substitution Principle? Explain the Rectangle-Square problem and how it violates LSP.**

**Difficulty:** Mid-Level

**Answer:**

**Liskov Substitution Principle**: Objects of a superclass should be replaceable with objects of subclasses without breaking the application. Subclasses must maintain the behavioral contract of the base class.

**Rectangle-Square Problem:**

```python
# Seems logical: Square IS-A Rectangle, right?
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def get_area(self): return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):
        self.width = w
        self.height = w  # Square must have equal sides
    
    def set_height(self, h):
        self.width = h
        self.height = h

# Client code expecting Rectangle behavior
def test_rectangle(rect: Rectangle):
    rect.set_width(5)
    rect.set_height(4)
    assert rect.get_area() == 20  # Expects 5 * 4 = 20

test_rectangle(Rectangle())  # ✅ Passes: area = 20
test_rectangle(Square())  # ❌ FAILS: area = 16 (4*4)
```

**Why LSP is Violated:**

1. **Changed Behavior**: `set_width()` in Square changes both dimensions (Rectangle changes only width)
2. **Broken Invariant**: Rectangle's width and height are independent; Square couples them
3. **Substitution Fails**: Square cannot replace Rectangle in client code
4. **Contract Violation**: Rectangle's contract says "set_width changes width", but Square changes both

**Correct Solution:**

```python
# Don't use inheritance
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def get_area(self):
        return self.width * self.height

class Square:
    def __init__(self, side):
        self.side = side
    
    def get_area(self):
        return self.side * self.side

# Or use common interface
class Shape(ABC):
    @abstractmethod
    def get_area(self): pass

class Rectangle(Shape):
    # Implementation

class Square(Shape):
    # Implementation
```

**Why This Matters in DevSecOps:**

In infrastructure code, LSP violations cause production failures:

```python
# ❌ LSP Violation in deployment code
class Deployment:
    def deploy(self) -> bool:
        """Deploy and return success."""
        return self._execute_deployment()

class ManualDeployment(Deployment):
    def deploy(self) -> bool:
        """VIOLATION: Blocks for manual approval!"""
        wait_for_approval()  # Might wait forever
        return self._execute_deployment()

# Automation breaks
for deployment in deployments:
    deployment.deploy()  # Hangs on ManualDeployment!
```

**Follow-up:** How would you refactor an existing codebase with LSP violations?

---

### Key Takeaways - LSP

✅ **Subtypes must be substitutable** - Replace parent with child without breaking code

✅ **Maintain behavioral contracts** - Don't change expectations

✅ **Preserve invariants** - Don't break guarantees

✅ **Compatible exceptions** - Don't throw new exceptions

✅ **Return types compatible** - Covariant returns OK

✅ **Parameter types compatible** - Contravariant parameters OK

✅ **Think twice before inheriting** - Prefer composition if behavior differs

✅ **Test substitutability** - Run parent tests with child instances

---

## 2.4 Interface Segregation Principle (ISP)

**Definition:** Clients should not be forced to depend on interfaces they don't use. Many specific interfaces are better than one general-purpose interface.

**Why It Matters:**
- Smaller, focused interfaces
- Easier to implement
- Reduces coupling
- Prevents "fat" interfaces

### Understanding ISP

```python
# ❌ BAD: "Fat" interface - clients forced to implement unused methods
class CloudProvider(ABC):
    """One interface to rule them all - TOO MUCH!"""
    
    # Compute methods
    @abstractmethod
    def create_vm(self, config): pass
    
    @abstractmethod
    def delete_vm(self, vm_id): pass
    
    # Storage methods
    @abstractmethod
    def create_bucket(self, name): pass
    
    @abstractmethod
    def delete_bucket(self, name): pass
    
    # Database methods
    @abstractmethod
    def create_database(self, config): pass
    
    @abstractmethod
    def delete_database(self, db_id): pass
    
    # Networking methods
    @abstractmethod
    def create_vpc(self, cidr): pass
    
    @abstractmethod
    def delete_vpc(self, vpc_id): pass
    
    # ... 20 more methods


class SimpleStorageProvider(CloudProvider):
    """
    Only provides storage, but forced to implement EVERYTHING!
    """
    
    def create_bucket(self, name):
        # Actual implementation
        pass
    
    def delete_bucket(self, name):
        # Actual implementation
        pass
    
    # ❌ FORCED to implement methods we don't support
    def create_vm(self, config):
        raise NotImplementedError("No compute support")
    
    def delete_vm(self, vm_id):
        raise NotImplementedError("No compute support")
    
    def create_database(self, config):
        raise NotImplementedError("No database support")
    
    # ... 18 more NotImplementedError methods!
```

**Problems:**
- SimpleStorageProvider implements 20+ methods it doesn't support
- Clients can't tell which methods are available
- Changes to compute methods force changes to storage-only providers
- Testing requires mocking unused methods

### Applying ISP

```python
# ✅ GOOD: Segregated interfaces
class ComputeProvider(Protocol):
    """Interface for compute operations only."""
    
    def create_vm(self, config: dict) -> str: ...
    def delete_vm(self, vm_id: str) -> bool: ...
    def get_vm_status(self, vm_id: str) -> str: ...


class StorageProvider(Protocol):
    """Interface for storage operations only."""
    
    def create_bucket(self, name: str) -> str: ...
    def delete_bucket(self, name: str) -> bool: ...
    def upload_file(self, bucket: str, key: str, data: bytes) -> bool: ...


class DatabaseProvider(Protocol):
    """Interface for database operations only."""
    
    def create_database(self, config: dict) -> str: ...
    def delete_database(self, db_id: str) -> bool: ...
    def execute_query(self, db_id: str, query: str) -> list: ...


class NetworkProvider(Protocol):
    """Interface for networking operations only."""
    
    def create_vpc(self, cidr: str) -> str: ...
    def delete_vpc(self, vpc_id: str) -> bool: ...
    def create_subnet(self, vpc_id: str, cidr: str) -> str: ...


# Implementations only implement what they support
class S3StorageProvider:
    """Storage-only provider - clean interface."""
    
    def create_bucket(self, name: str) -> str:
        """Create S3 bucket."""
        # Implementation
        return f"bucket-{name}"
    
    def delete_bucket(self, name: str) -> bool:
        """Delete S3 bucket."""
        # Implementation
        return True
    
    def upload_file(self, bucket: str, key: str, data: bytes) -> bool:
        """Upload file to S3."""
        # Implementation
        return True


class EC2ComputeProvider:
    """Compute-only provider - clean interface."""
    
    def create_vm(self, config: dict) -> str:
        """Create EC2 instance."""
        # Implementation
        return "i-12345"
    
    def delete_vm(self, vm_id: str) -> bool:
        """Terminate EC2 instance."""
        # Implementation
        return True
    
    def get_vm_status(self, vm_id: str) -> str:
        """Get instance status."""
        # Implementation
        return "running"


class RDSDatabaseProvider:
    """Database-only provider - clean interface."""
    
    def create_database(self, config: dict) -> str:
        """Create RDS database."""
        # Implementation
        return "db-12345"
    
    def delete_database(self, db_id: str) -> bool:
        """Delete RDS database."""
        # Implementation
        return True
    
    def execute_query(self, db_id: str, query: str) -> list:
        """Execute SQL query."""
        # Implementation
        return []


# Full-featured provider implements multiple interfaces
class AWSFullProvider:
    """
    Full AWS provider implementing multiple interfaces.
    
    Clean: Only implements interfaces for supported features.
    """
    
    def __init__(self):
        self.compute = EC2ComputeProvider()
        self.storage = S3StorageProvider()
        self.database = RDSDatabaseProvider()
    
    # Compute interface
    def create_vm(self, config: dict) -> str:
        return self.compute.create_vm(config)
    
    def delete_vm(self, vm_id: str) -> bool:
        return self.compute.delete_vm(vm_id)
    
    def get_vm_status(self, vm_id: str) -> str:
        return self.compute.get_vm_status(vm_id)
    
    # Storage interface
    def create_bucket(self, name: str) -> str:
        return self.storage.create_bucket(name)
    
    def delete_bucket(self, name: str) -> bool:
        return self.storage.delete_bucket(name)
    
    def upload_file(self, bucket: str, key: str, data: bytes) -> bool:
        return self.storage.upload_file(bucket, key, data)
    
    # Database interface
    def create_database(self, config: dict) -> str:
        return self.database.create_database(config)
    
    def delete_database(self, db_id: str) -> bool:
        return self.database.delete_database(db_id)
    
    def execute_query(self, db_id: str, query: str) -> list:
        return self.database.execute_query(db_id, query)


# Clients depend only on what they need
class StorageManager:
    """Manages storage - only depends on StorageProvider."""
    
    def __init__(self, provider: StorageProvider):
        self.provider = provider
    
    def backup_data(self, data: bytes) -> bool:
        """Backup data to storage."""
        bucket = self.provider.create_bucket("backups")
        return self.provider.upload_file(bucket, "backup.dat", data)


class ComputeManager:
    """Manages compute - only depends on ComputeProvider."""
    
    def __init__(self, provider: ComputeProvider):
        self.provider = provider
    
    def provision_server(self, config: dict) -> str:
        """Provision compute server."""
        vm_id = self.provider.create_vm(config)
        
        # Wait for running status
        while self.provider.get_vm_status(vm_id) != "running":
            time.sleep(5)
        
        return vm_id


# Usage with segregated interfaces
storage = S3StorageProvider()
storage_mgr = StorageManager(storage)  # ✅ Only needs storage

compute = EC2ComputeProvider()
compute_mgr = ComputeManager(compute)  # ✅ Only needs compute

# Full provider works with both
full_provider = AWSFullProvider()
storage_mgr = StorageManager(full_provider)  # ✅ Works
compute_mgr = ComputeManager(full_provider)  # ✅ Works
```

**Benefits:**
- S3StorageProvider only implements storage methods ✅
- StorageManager doesn't depend on compute/database ✅
- Adding network features doesn't affect storage ✅
- Each interface is small and focused ✅

### DevSecOps Example: Monitoring System

```python
# ❌ BAD: Monolithic monitoring interface
class MonitoringSystem(ABC):
    """One interface for all monitoring - TOO BIG!"""
    
    # Metrics
    @abstractmethod
    def collect_metrics(self): pass
    
    @abstractmethod
    def query_metrics(self, query): pass
    
    # Logs
    @abstractmethod
    def collect_logs(self): pass
    
    @abstractmethod
    def search_logs(self, query): pass
    
    # Traces
    @abstractmethod
    def collect_traces(self): pass
    
    @abstractmethod
    def query_traces(self, trace_id): pass
    
    # Alerts
    @abstractmethod
    def create_alert(self, rule): pass
    
    @abstractmethod
    def trigger_alert(self, alert): pass
    
    # Dashboards
    @abstractmethod
    def create_dashboard(self, config): pass
    
    @abstractmethod
    def update_dashboard(self, dashboard_id): pass


class PrometheusMonitoring(MonitoringSystem):
    """
    Prometheus only does metrics, forced to implement everything!
    """
    
    def collect_metrics(self):
        # ✅ Implemented
        pass
    
    def query_metrics(self, query):
        # ✅ Implemented
        pass
    
    # ❌ Forced to implement unused features
    def collect_logs(self):
        raise NotImplementedError("Use ELK for logs")
    
    def search_logs(self, query):
        raise NotImplementedError("Use ELK for logs")
    
    # ... more NotImplementedError


# ✅ GOOD: Segregated monitoring interfaces
class MetricsCollector(Protocol):
    """Interface for metrics collection and querying."""
    
    def collect_metrics(self) -> dict: ...
    def query_metrics(self, query: str) -> list: ...
    def record_metric(self, name: str, value: float) -> None: ...


class LogCollector(Protocol):
    """Interface for log collection and searching."""
    
    def collect_logs(self) -> list: ...
    def search_logs(self, query: str) -> list: ...
    def write_log(self, level: str, message: str) -> None: ...


class TraceCollector(Protocol):
    """Interface for distributed tracing."""
    
    def collect_traces(self) -> list: ...
    def query_traces(self, trace_id: str) -> dict: ...
    def start_span(self, operation: str) -> str: ...


class AlertManager(Protocol):
    """Interface for alerting."""
    
    def create_alert_rule(self, rule: dict) -> str: ...
    def trigger_alert(self, alert: dict) -> None: ...
    def acknowledge_alert(self, alert_id: str) -> None: ...


# Focused implementations
class PrometheusMetrics:
    """Prometheus - only implements metrics."""
    
    def collect_metrics(self) -> dict:
        # Scrape metrics from exporters
        return {}
    
    def query_metrics(self, query: str) -> list:
        # PromQL query
        return []
    
    def record_metric(self, name: str, value: float) -> None:
        # Push to pushgateway
        pass


class ElasticsearchLogs:
    """Elasticsearch - only implements logs."""
    
    def collect_logs(self) -> list:
        # Collect from filebeat/fluentd
        return []
    
    def search_logs(self, query: str) -> list:
        # Lucene query
        return []
    
    def write_log(self, level: str, message: str) -> None:
        # Index log document
        pass


class JaegerTracing:
    """Jaeger - only implements tracing."""
    
    def collect_traces(self) -> list:
        # Collect from agents
        return []
    
    def query_traces(self, trace_id: str) -> dict:
        # Query trace backend
        return {}
    
    def start_span(self, operation: str) -> str:
        # Create span
        return "span-123"


# Clients depend only on needed interfaces
class ApplicationMonitor:
    """Application monitoring - uses metrics and logs."""
    
    def __init__(
        self,
        metrics: MetricsCollector,
        logs: LogCollector
    ):
        self.metrics = metrics
        self.logs = logs
        # Doesn't depend on tracing or alerts
    
    def monitor_request(self, request_time: float):
        """Monitor application request."""
        self.metrics.record_metric('request_duration', request_time)
        
        if request_time > 1.0:
            self.logs.write_log('WARNING', f'Slow request: {request_time}s')


class PerformanceAnalyzer:
    """Performance analysis - uses metrics and traces."""
    
    def __init__(
        self,
        metrics: MetricsCollector,
        traces: TraceCollector
    ):
        self.metrics = metrics
        self.traces = traces
        # Doesn't depend on logs or alerts
    
    def analyze_performance(self, trace_id: str):
        """Analyze request performance."""
        trace = self.traces.query_traces(trace_id)
        metrics = self.metrics.query_metrics(f'trace_{trace_id}')
        # Performance analysis
```

### Key Takeaways - ISP

✅ **Small, focused interfaces** - One purpose per interface

✅ **Client-specific interfaces** - Tailor to client needs

✅ **Avoid fat interfaces** - Don't force unused methods

✅ **Use Protocol for duck typing** - Python's structural subtyping

✅ **Composition for full features** - Combine small interfaces

✅ **Reduces coupling** - Clients depend only on what they use

✅ **Easier testing** - Mock only needed methods

---

*[Continuing with Dependency Inversion Principle and Design Patterns in next section...]*


## 2.5 Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

**In simpler terms:**
- Depend on interfaces, not concrete implementations
- High-level policy should not depend on low-level details
- Invert the traditional dependency flow

**Why It Matters:**
- Enables loose coupling
- Makes code testable (inject mocks)
- Supports multiple implementations
- Facilitates change without cascading effects

### Understanding DIP

**Traditional Dependencies (❌ BAD):**

```
┌─────────────────────────┐
│   DeploymentService     │ (High-level)
│   (Business Logic)      │
└───────────┬─────────────┘
            │ depends on
            ↓
┌─────────────────────────┐
│   AWSDeploymentClient   │ (Low-level)
│   (Implementation)      │
└─────────────────────────┘
```

Problem: `DeploymentService` directly depends on AWS implementation. Cannot switch to GCP without modifying `DeploymentService`.

**Dependency Inversion (✅ GOOD):**

```
┌─────────────────────────┐
│   DeploymentService     │ (High-level)
└───────────┬─────────────┘
            │ depends on
            ↓
┌─────────────────────────┐
│  DeploymentProvider     │ (Abstraction)
│     (Interface)         │
└───────────┬─────────────┘
            ↑ implements
    ┌───────┴────────┐
    │                │
┌───┴────────┐  ┌───┴────────┐
│    AWS     │  │    GCP     │ (Low-level)
│ Provider   │  │ Provider   │ (Implementations)
└────────────┘  └────────────┘
```

Both high-level and low-level depend on abstraction. Can switch implementations without changing `DeploymentService`.

### Violating DIP

```python
# ❌ BAD: High-level module depends on low-level implementation
import boto3

class DeploymentService:
    """
    Deployment service tightly coupled to AWS.
    VIOLATES DIP!
    """
    
    def __init__(self):
        # Direct dependency on AWS implementation
        self.ec2 = boto3.client('ec2', region_name='us-east-1')
        self.s3 = boto3.client('s3', region_name='us-east-1')
    
    def deploy_application(self, app_name: str, version: str) -> bool:
        """Deploy application to AWS."""
        # Upload artifact to S3
        self.s3.upload_file(
            f"{app_name}-{version}.zip",
            'deployment-artifacts',
            f"{app_name}/{version}.zip"
        )
        
        # Provision EC2 instances
        response = self.ec2.run_instances(
            ImageId='ami-12345',
            InstanceType='t2.micro',
            MinCount=2,
            MaxCount=2
        )
        
        return True

# Problems:
# 1. Cannot test without real AWS
# 2. Cannot switch to GCP without rewriting
# 3. Violates SRP (knows AWS API details)
# 4. Hard to mock for testing
```

**Why This Is Bad:**

```python
# Cannot test without AWS credentials
def test_deploy_application():
    """This test requires real AWS account!"""
    service = DeploymentService()  # Creates real boto3 clients
    service.deploy_application('my-app', '1.0.0')  # Calls real AWS!

# Cannot switch to GCP
# If we need to support GCP, must modify DeploymentService
# This violates Open/Closed Principle too!
```

### Applying DIP

```python
# ✅ GOOD: Depend on abstraction
from abc import ABC, abstractmethod
from typing import Protocol

# Define abstraction (interface)
class CloudProvider(Protocol):
    """Abstract interface for cloud providers."""
    
    def upload_artifact(self, file_path: str, destination: str) -> str:
        """Upload deployment artifact."""
        ...
    
    def provision_instances(
        self,
        count: int,
        instance_type: str,
        image_id: str
    ) -> list[str]:
        """Provision compute instances."""
        ...
    
    def terminate_instances(self, instance_ids: list[str]) -> bool:
        """Terminate instances."""
        ...


# High-level module depends on abstraction
class DeploymentService:
    """
    Deployment service following DIP.
    
    Depends on CloudProvider abstraction, not concrete implementation.
    """
    
    def __init__(self, cloud_provider: CloudProvider):
        """
        Inject cloud provider dependency.
        
        This is Dependency Injection - we inject the dependency
        rather than creating it internally.
        """
        self.cloud = cloud_provider
    
    def deploy_application(self, app_name: str, version: str) -> bool:
        """
        Deploy application using injected cloud provider.
        
        Works with ANY CloudProvider implementation!
        """
        try:
            # Upload artifact
            artifact_url = self.cloud.upload_artifact(
                f"{app_name}-{version}.zip",
                f"{app_name}/{version}.zip"
            )
            
            # Provision instances
            instance_ids = self.cloud.provision_instances(
                count=2,
                instance_type='t2.micro',
                image_id='ami-12345'
            )
            
            logger.info(
                f"Deployed {app_name} v{version}: {instance_ids}"
            )
            return True
            
        except Exception as e:
            logger.error(f"Deployment failed: {e}")
            return False


# Low-level implementations
class AWSProvider:
    """AWS implementation of CloudProvider."""
    
    def __init__(self, region: str):
        import boto3
        self.ec2 = boto3.client('ec2', region_name=region)
        self.s3 = boto3.client('s3', region_name=region)
        self.region = region
    
    def upload_artifact(self, file_path: str, destination: str) -> str:
        """Upload to S3."""
        bucket = 'deployment-artifacts'
        self.s3.upload_file(file_path, bucket, destination)
        return f"s3://{bucket}/{destination}"
    
    def provision_instances(
        self,
        count: int,
        instance_type: str,
        image_id: str
    ) -> list[str]:
        """Provision EC2 instances."""
        response = self.ec2.run_instances(
            ImageId=image_id,
            InstanceType=instance_type,
            MinCount=count,
            MaxCount=count
        )
        return [i['InstanceId'] for i in response['Instances']]
    
    def terminate_instances(self, instance_ids: list[str]) -> bool:
        """Terminate EC2 instances."""
        self.ec2.terminate_instances(InstanceIds=instance_ids)
        return True


class GCPProvider:
    """GCP implementation of CloudProvider."""
    
    def __init__(self, project_id: str, zone: str):
        from google.cloud import compute_v1, storage
        self.compute = compute_v1.InstancesClient()
        self.storage = storage.Client()
        self.project_id = project_id
        self.zone = zone
    
    def upload_artifact(self, file_path: str, destination: str) -> str:
        """Upload to Cloud Storage."""
        bucket = self.storage.bucket('deployment-artifacts')
        blob = bucket.blob(destination)
        blob.upload_from_filename(file_path)
        return f"gs://deployment-artifacts/{destination}"
    
    def provision_instances(
        self,
        count: int,
        instance_type: str,
        image_id: str
    ) -> list[str]:
        """Provision GCE instances."""
        instance_ids = []
        for i in range(count):
            instance_name = f"instance-{i}"
            # GCP provisioning logic
            instance_ids.append(instance_name)
        return instance_ids
    
    def terminate_instances(self, instance_ids: list[str]) -> bool:
        """Terminate GCE instances."""
        for instance_id in instance_ids:
            self.compute.delete(
                project=self.project_id,
                zone=self.zone,
                instance=instance_id
            )
        return True


# Usage - depend on abstraction
def main():
    # Use AWS provider
    aws = AWSProvider('us-east-1')
    service = DeploymentService(aws)
    service.deploy_application('my-app', '1.0.0')
    
    # Switch to GCP - NO CODE CHANGES to DeploymentService!
    gcp = GCPProvider('my-project', 'us-central1-a')
    service = DeploymentService(gcp)
    service.deploy_application('my-app', '1.0.0')


# Testing - inject mock
from unittest.mock import MagicMock

def test_deploy_application():
    """Test deployment with mock provider."""
    # Create mock provider
    mock_provider = MagicMock()
    mock_provider.upload_artifact.return_value = 's3://bucket/artifact.zip'
    mock_provider.provision_instances.return_value = ['i-123', 'i-456']
    
    # Inject mock into service
    service = DeploymentService(mock_provider)
    
    # Test
    result = service.deploy_application('test-app', '1.0.0')
    
    # Verify
    assert result is True
    mock_provider.upload_artifact.assert_called_once()
    mock_provider.provision_instances.assert_called_once_with(
        count=2,
        instance_type='t2.micro',
        image_id='ami-12345'
    )
```

**Benefits:**
- ✅ Testable (inject mocks)
- ✅ Flexible (swap AWS for GCP)
- ✅ Loose coupling
- ✅ Follows Open/Closed Principle

### Dependency Injection Patterns

**1. Constructor Injection (Recommended):**

```python
class DeploymentPipeline:
    """Pipeline with constructor injection."""
    
    def __init__(
        self,
        builder: ImageBuilder,
        registry: ContainerRegistry,
        deployer: KubernetesDeployer,
        notifier: NotificationService
    ):
        """Inject all dependencies via constructor."""
        self.builder = builder
        self.registry = registry
        self.deployer = deployer
        self.notifier = notifier
    
    def execute(self, app_name: str, version: str) -> bool:
        """Execute pipeline using injected dependencies."""
        image = self.builder.build(app_name, version)
        self.registry.push(image)
        self.deployer.deploy(image)
        self.notifier.notify_success(app_name, version)
        return True

# Usage
pipeline = DeploymentPipeline(
    builder=DockerBuilder(),
    registry=ECRRegistry(),
    deployer=KubernetesDeployer(),
    notifier=SlackNotifier()
)
```

**2. Setter Injection:**

```python
class MetricsCollector:
    """Collector with setter injection."""
    
    def __init__(self):
        self._storage = None
    
    def set_storage(self, storage: MetricsStorage):
        """Inject storage via setter."""
        self._storage = storage
    
    def collect(self, metrics: dict):
        """Collect metrics."""
        if self._storage is None:
            raise ValueError("Storage not configured")
        self._storage.save(metrics)

# Usage
collector = MetricsCollector()
collector.set_storage(PrometheusStorage())
```

**3. Interface Injection:**

```python
class Configurable(Protocol):
    """Interface for configurable components."""
    def configure(self, config: dict): ...

class Server:
    """Server that accepts configuration."""
    
    def configure(self, config: dict):
        """Inject configuration."""
        self.port = config['port']
        self.host = config['host']
```

### Inversion of Control (IoC) Container

For complex applications with many dependencies:

```python
from typing import Dict, Type, Callable, Any

class Container:
    """Simple IoC container for dependency injection."""
    
    def __init__(self):
        self._services: Dict[Type, Callable] = {}
        self._singletons: Dict[Type, Any] = {}
    
    def register(
        self,
        interface: Type,
        implementation: Callable,
        singleton: bool = False
    ):
        """Register a service."""
        self._services[interface] = implementation
        if singleton:
            self._singletons[interface] = None
    
    def resolve(self, interface: Type) -> Any:
        """Resolve a service instance."""
        # Return singleton if registered
        if interface in self._singletons:
            if self._singletons[interface] is None:
                self._singletons[interface] = self._services[interface]()
            return self._singletons[interface]
        
        # Create new instance
        if interface not in self._services:
            raise ValueError(f"Service not registered: {interface}")
        
        return self._services[interface]()


# Register services
container = Container()

container.register(
    CloudProvider,
    lambda: AWSProvider('us-east-1'),
    singleton=True
)

container.register(
    NotificationService,
    lambda: SlackNotifier(webhook_url='https://...'),
    singleton=True
)

container.register(
    DeploymentService,
    lambda: DeploymentService(
        cloud_provider=container.resolve(CloudProvider)
    )
)

# Resolve dependencies
deployment_service = container.resolve(DeploymentService)
deployment_service.deploy_application('my-app', '1.0.0')
```

### DevSecOps Example: Configuration Management

```python
# ❌ BAD: Direct dependency on configuration file format
import yaml

class ApplicationConfig:
    """Configuration tightly coupled to YAML."""
    
    def __init__(self, config_file: str):
        # Hardcoded to YAML format
        with open(config_file) as f:
            self.config = yaml.safe_load(f)
    
    def get(self, key: str):
        return self.config[key]

# Problem: Cannot switch to JSON, environment variables, or AWS Parameter Store
# without modifying this class

# ✅ GOOD: Depend on abstraction
from abc import ABC, abstractmethod

class ConfigProvider(Protocol):
    """Abstract configuration provider."""
    
    def get(self, key: str, default=None) -> Any:
        """Get configuration value."""
        ...
    
    def get_all(self) -> dict:
        """Get all configuration."""
        ...


class YAMLConfigProvider:
    """YAML configuration provider."""
    
    def __init__(self, config_file: str):
        import yaml
        with open(config_file) as f:
            self._config = yaml.safe_load(f)
    
    def get(self, key: str, default=None) -> Any:
        """Get value from YAML."""
        return self._config.get(key, default)
    
    def get_all(self) -> dict:
        """Get all YAML config."""
        return self._config.copy()


class EnvironmentConfigProvider:
    """Environment variable configuration provider."""
    
    def __init__(self, prefix: str = 'APP_'):
        self.prefix = prefix
    
    def get(self, key: str, default=None) -> Any:
        """Get value from environment."""
        env_key = f"{self.prefix}{key.upper()}"
        return os.getenv(env_key, default)
    
    def get_all(self) -> dict:
        """Get all environment variables with prefix."""
        return {
            k.replace(self.prefix, ''): v
            for k, v in os.environ.items()
            if k.startswith(self.prefix)
        }


class AWSParameterStoreProvider:
    """AWS Parameter Store configuration provider."""
    
    def __init__(self, path: str, region: str):
        import boto3
        self.ssm = boto3.client('ssm', region_name=region)
        self.path = path
        self._cache = {}
        self._load_parameters()
    
    def _load_parameters(self):
        """Load parameters from Parameter Store."""
        response = self.ssm.get_parameters_by_path(
            Path=self.path,
            Recursive=True,
            WithDecryption=True
        )
        for param in response['Parameters']:
            key = param['Name'].replace(self.path, '').lstrip('/')
            self._cache[key] = param['Value']
    
    def get(self, key: str, default=None) -> Any:
        """Get value from Parameter Store."""
        return self._cache.get(key, default)
    
    def get_all(self) -> dict:
        """Get all parameters."""
        return self._cache.copy()


class ChainedConfigProvider:
    """Chain multiple config providers (fallback)."""
    
    def __init__(self, providers: list[ConfigProvider]):
        self.providers = providers
    
    def get(self, key: str, default=None) -> Any:
        """Get value from first provider that has it."""
        for provider in self.providers:
            value = provider.get(key)
            if value is not None:
                return value
        return default
    
    def get_all(self) -> dict:
        """Merge all configurations (later providers override)."""
        result = {}
        for provider in self.providers:
            result.update(provider.get_all())
        return result


# Application depends on abstraction
class Application:
    """Application using dependency injection for config."""
    
    def __init__(self, config: ConfigProvider):
        """Inject configuration provider."""
        self.config = config
    
    def start(self):
        """Start application."""
        port = self.config.get('port', 8080)
        host = self.config.get('host', '0.0.0.0')
        debug = self.config.get('debug', False)
        
        logger.info(f"Starting on {host}:{port} (debug={debug})")
        # Start server...


# Usage - flexibility in configuration source
# Development: YAML file
app = Application(YAMLConfigProvider('config.yaml'))

# Docker: Environment variables
app = Application(EnvironmentConfigProvider('APP_'))

# Production: AWS Parameter Store
app = Application(AWSParameterStoreProvider('/prod/myapp', 'us-east-1'))

# Hybrid: Chain multiple sources (env vars override YAML)
app = Application(ChainedConfigProvider([
    YAMLConfigProvider('config.yaml'),  # Base config
    EnvironmentConfigProvider('APP_')   # Overrides
]))
```

### Testing with DIP

DIP makes testing trivial:

```python
# Test with mock provider
def test_application_startup():
    """Test application startup with mock config."""
    mock_config = MagicMock()
    mock_config.get.side_effect = lambda k, d=None: {
        'port': 9000,
        'host': 'localhost',
        'debug': True
    }.get(k, d)
    
    app = Application(mock_config)
    app.start()
    
    # Verify config was accessed
    mock_config.get.assert_any_call('port', 8080)
    mock_config.get.assert_any_call('host', '0.0.0.0')

# Test with fake provider
class FakeConfigProvider:
    """Fake config provider for testing."""
    
    def __init__(self, config: dict):
        self._config = config
    
    def get(self, key: str, default=None):
        return self._config.get(key, default)
    
    def get_all(self):
        return self._config.copy()

def test_application_with_fake_config():
    """Test with fake config provider."""
    fake_config = FakeConfigProvider({
        'port': 8888,
        'host': '127.0.0.1',
        'debug': False
    })
    
    app = Application(fake_config)
    assert app.config.get('port') == 8888
```

### Frequently Asked Questions - DIP

**Q1: What's the difference between Dependency Inversion and Dependency Injection?**

**A:**

**Dependency Inversion (DIP):** Design principle
- High-level modules depend on abstractions
- Low-level modules implement abstractions
- **What** you should do

**Dependency Injection (DI):** Implementation technique
- Pass dependencies from outside
- Constructor, setter, or interface injection
- **How** you implement DIP

```python
# DIP: Depend on abstraction
class DeploymentService:
    def __init__(self, cloud: CloudProvider):  # DI: Inject dependency
        self.cloud = cloud

# DIP without DI (anti-pattern)
class BadService:
    def __init__(self):
        # Violates DIP: depends on concrete class
        self.cloud = AWSProvider()  # Creates dependency internally
```

**Q2: When should I use DIP?**

**A:** Use DIP when:

✅ **Need to swap implementations**
- Multiple cloud providers
- Different databases (PostgreSQL, MySQL, MongoDB)
- Various notification channels (email, Slack, PagerDuty)

✅ **Need to test with mocks**
- External APIs
- Cloud services
- Third-party integrations

✅ **Expect implementation to change**
- New technology adoption
- Migration between services

❌ **Don't use DIP when:**
- Only one implementation will ever exist
- No testing needed (simple scripts)
- Adds unnecessary complexity

```python
# ✅ Use DIP: Multiple implementations
class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str): ...

class SMTPEmailSender: ...      # Implementation 1
class SendGridEmailSender: ...  # Implementation 2
class SESEmailSender: ...       # Implementation 3

# ❌ Don't need DIP: Only one implementation
def calculate_tax(amount: float) -> float:
    """Simple calculation, no need for abstraction."""
    return amount * 0.07
```

**Q3: How is DIP different from just using interfaces?**

**A:** DIP is about **dependency direction**, not just interfaces.

```python
# Using interfaces but NOT inverting dependencies
class DeploymentService:  # High-level
    def __init__(self):
        self.aws = AWSClient()  # Still depends on low-level!

# True DIP: Both depend on abstraction
class DeploymentService:  # High-level
    def __init__(self, provider: CloudProvider):  # Depends on abstraction
        self.provider = provider

class AWSClient(CloudProvider):  # Low-level depends on abstraction
    pass
```

### Interview Questions - DIP

**Question 1: Explain the Dependency Inversion Principle. How does it differ from dependency injection?**

**Difficulty:** Junior

**Answer:**

**Dependency Inversion Principle (DIP):**
A design principle stating that:
1. High-level modules should not depend on low-level modules
2. Both should depend on abstractions
3. Abstractions should not depend on details
4. Details should depend on abstractions

**Dependency Injection (DI):**
A technique for implementing DIP by passing dependencies from outside rather than creating them internally.

**Example:**

```python
# WITHOUT DIP (bad)
class NotificationService:
    def __init__(self):
        # Depends on concrete implementation
        self.email = SMTPEmailSender()  # Low-level detail
    
    def notify(self, message):
        self.email.send(message)

# Problem: Cannot use Slack, cannot test with mocks

# WITH DIP (good)
class NotificationService:
    def __init__(self, sender: MessageSender):  # DI: Dependency injected
        self.sender = sender  # DIP: Depend on abstraction
    
    def notify(self, message):
        self.sender.send(message)

# Usage
service = NotificationService(SMTPEmailSender())  # Inject dependency
service = NotificationService(SlackSender())      # Different implementation
service = NotificationService(MockSender())       # Testing with mock
```

**Key Differences:**

| Aspect | DIP | DI |
|--------|-----|-----|
| **Type** | Design Principle | Implementation Technique |
| **Level** | Architecture/Design | Code Implementation |
| **Focus** | What to depend on | How to provide dependencies |
| **Goal** | Loose coupling | Flexibility, testability |

**Why This Matters in DevSecOps:**
- Deploy to multiple clouds without code changes
- Test infrastructure code without real AWS
- Switch notification channels (Slack to PagerDuty) easily
- Replace databases (PostgreSQL to MongoDB) without rewrites

**Follow-up:** How would you refactor a tightly-coupled codebase to follow DIP?

---

**Question 2: You have a deployment service that directly calls AWS boto3. How would you refactor it to follow DIP, and what are the benefits?**

**Difficulty:** Mid-Level

**Answer:**

**Original Code (Violates DIP):**

```python
import boto3

class DeploymentService:
    """Tightly coupled to AWS."""
    
    def __init__(self):
        self.ec2 = boto3.client('ec2')
        self.s3 = boto3.client('s3')
    
    def deploy(self, app_name: str) -> str:
        # Upload to S3
        self.s3.upload_file(
            f"{app_name}.zip",
            'deployments',
            f"{app_name}.zip"
        )
        
        # Launch EC2 instance
        response = self.ec2.run_instances(
            ImageId='ami-12345',
            InstanceType='t2.micro',
            MinCount=1,
            MaxCount=1
        )
        
        return response['Instances'][0]['InstanceId']

# Problems:
# 1. Cannot test without AWS credentials
# 2. Cannot switch to GCP
# 3. Cannot mock for unit tests
# 4. Violates SRP (knows AWS API details)
```

**Refactored with DIP:**

**Step 1: Define abstraction (interface):**

```python
from abc import ABC, abstractmethod
from typing import Protocol

class CloudStorage(Protocol):
    """Abstract storage interface."""
    def upload(self, local_path: str, remote_path: str) -> str: ...

class ComputeProvider(Protocol):
    """Abstract compute interface."""
    def create_instance(self, config: dict) -> str: ...
    def terminate_instance(self, instance_id: str) -> bool: ...
```

**Step 2: Create implementations:**

```python
class S3Storage:
    """S3 implementation of CloudStorage."""
    
    def __init__(self, bucket: str, region: str = 'us-east-1'):
        import boto3
        self.s3 = boto3.client('s3', region_name=region)
        self.bucket = bucket
    
    def upload(self, local_path: str, remote_path: str) -> str:
        """Upload to S3."""
        self.s3.upload_file(local_path, self.bucket, remote_path)
        return f"s3://{self.bucket}/{remote_path}"


class EC2ComputeProvider:
    """EC2 implementation of ComputeProvider."""
    
    def __init__(self, region: str = 'us-east-1'):
        import boto3
        self.ec2 = boto3.client('ec2', region_name=region)
    
    def create_instance(self, config: dict) -> str:
        """Create EC2 instance."""
        response = self.ec2.run_instances(
            ImageId=config['image_id'],
            InstanceType=config['instance_type'],
            MinCount=1,
            MaxCount=1
        )
        return response['Instances'][0]['InstanceId']
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate EC2 instance."""
        self.ec2.terminate_instances(InstanceIds=[instance_id])
        return True


# Alternative: GCP implementations
class GCSStorage:
    """Google Cloud Storage implementation."""
    
    def __init__(self, bucket: str):
        from google.cloud import storage
        self.client = storage.Client()
        self.bucket = self.client.bucket(bucket)
    
    def upload(self, local_path: str, remote_path: str) -> str:
        """Upload to GCS."""
        blob = self.bucket.blob(remote_path)
        blob.upload_from_filename(local_path)
        return f"gs://{self.bucket.name}/{remote_path}"


class GCEComputeProvider:
    """Google Compute Engine implementation."""
    
    def __init__(self, project: str, zone: str):
        from google.cloud import compute_v1
        self.compute = compute_v1.InstancesClient()
        self.project = project
        self.zone = zone
    
    def create_instance(self, config: dict) -> str:
        """Create GCE instance."""
        # GCE creation logic
        return "instance-name"
    
    def terminate_instance(self, instance_id: str) -> bool:
        """Terminate GCE instance."""
        # GCE termination logic
        return True
```

**Step 3: Refactor service to depend on abstractions:**

```python
class DeploymentService:
    """
    Deployment service following DIP.
    
    Depends on abstractions (CloudStorage, ComputeProvider),
    not concrete implementations (S3Storage, EC2ComputeProvider).
    """
    
    def __init__(
        self,
        storage: CloudStorage,
        compute: ComputeProvider
    ):
        """
        Inject dependencies (Dependency Injection).
        
        This enables:
        - Testing with mocks
        - Swapping implementations (AWS ↔ GCP)
        - Loose coupling
        """
        self.storage = storage
        self.compute = compute
    
    def deploy(self, app_name: str, config: dict) -> str:
        """
        Deploy application.
        
        Works with ANY storage and compute provider!
        """
        # Upload artifact
        artifact_url = self.storage.upload(
            f"{app_name}.zip",
            f"{app_name}/{config['version']}.zip"
        )
        
        logger.info(f"Uploaded artifact: {artifact_url}")
        
        # Create instance
        instance_id = self.compute.create_instance({
            'image_id': config['image_id'],
            'instance_type': config['instance_type']
        })
        
        logger.info(f"Created instance: {instance_id}")
        
        return instance_id
    
    def rollback(self, instance_id: str) -> bool:
        """Rollback deployment."""
        return self.compute.terminate_instance(instance_id)
```

**Step 4: Usage and testing:**

```python
# Production: AWS
aws_service = DeploymentService(
    storage=S3Storage('deployments-bucket'),
    compute=EC2ComputeProvider('us-east-1')
)

aws_service.deploy('my-app', {
    'version': '1.0.0',
    'image_id': 'ami-12345',
    'instance_type': 't2.micro'
})

# Production: GCP (no code changes to DeploymentService!)
gcp_service = DeploymentService(
    storage=GCSStorage('deployments-bucket'),
    compute=GCEComputeProvider('my-project', 'us-central1-a')
)

gcp_service.deploy('my-app', {
    'version': '1.0.0',
    'image_id': 'ubuntu-1804',
    'instance_type': 'n1-standard-1'
})

# Testing: Mocks (no real cloud resources!)
from unittest.mock import MagicMock

def test_deploy():
    """Test deployment with mocks."""
    mock_storage = MagicMock()
    mock_storage.upload.return_value = 's3://bucket/app.zip'
    
    mock_compute = MagicMock()
    mock_compute.create_instance.return_value = 'i-12345'
    
    service = DeploymentService(mock_storage, mock_compute)
    
    instance_id = service.deploy('test-app', {
        'version': '1.0.0',
        'image_id': 'ami-12345',
        'instance_type': 't2.micro'
    })
    
    assert instance_id == 'i-12345'
    mock_storage.upload.assert_called_once()
    mock_compute.create_instance.assert_called_once()
```

**Benefits of Refactoring:**

1. **Testability** ✅
   - Can test with mocks (no AWS credentials needed)
   - Unit tests run in milliseconds
   - No cloud costs for testing

2. **Flexibility** ✅
   - Switch from AWS to GCP without changing DeploymentService
   - Use different storage (S3, GCS, local filesystem)
   - Mix and match (S3 + GCE, GCS + EC2)

3. **Maintainability** ✅
   - DeploymentService is simpler (business logic only)
   - AWS/GCP details isolated in implementations
   - Changes to AWS API don't affect DeploymentService

4. **Extensibility** ✅
   - Add Azure support: create AzureStorage and AzureComputeProvider
   - No modifications to existing code (Open/Closed Principle)

5. **Separation of Concerns** ✅
   - DeploymentService: orchestration logic
   - S3Storage/EC2: AWS implementation details
   - Clear responsibilities

**Migration Strategy:**

```python
# Phase 1: Add abstractions alongside existing code
# (No breaking changes)

# Phase 2: Create adapter for old code
class LegacyAWSAdapter:
    """Adapter to use old code with new interface."""
    def __init__(self, old_service):
        self.old_service = old_service
    
    def upload(self, local_path, remote_path):
        return self.old_service.upload_to_s3(local_path, remote_path)

# Phase 3: Migrate clients incrementally

# Phase 4: Remove old code when migration complete
```

**Why This Matters in DevSecOps:**
- **Multi-cloud strategy**: Support AWS, GCP, Azure from one codebase
- **Testing**: Fast, reliable unit tests without cloud access
- **Disaster recovery**: Switch clouds quickly if one provider has outage
- **Cost optimization**: Easy to compare costs across providers
- **Vendor lock-in prevention**: Not tied to single cloud provider

**Follow-up:** How would you handle provider-specific features that don't fit the common abstraction?

---

### Key Takeaways - DIP

✅ **Depend on abstractions** - Not concrete implementations

✅ **Inject dependencies** - Constructor injection is preferred

✅ **Invert dependency flow** - Both high and low-level depend on abstraction

✅ **Enables testing** - Inject mocks for unit tests

✅ **Supports flexibility** - Swap implementations easily

✅ **Reduces coupling** - High-level doesn't know low-level details

✅ **Use Protocol for interfaces** - Duck typing in Python

✅ **IoC containers for complexity** - When many dependencies

---

## Summary - SOLID Principles in DevSecOps

### Quick Reference

| Principle | Focus | DevSecOps Example |
|-----------|-------|-------------------|
| **SRP** | One responsibility per class | DeploymentService orchestrates, CloudProvisioner provisions |
| **OCP** | Extend without modifying | Add new deployment strategy without changing DeploymentService |
| **LSP** | Subtypes substitutable | CanaryDeployment can replace BlueGreenDeployment |
| **ISP** | Focused interfaces | ComputeProvider, StorageProvider, not single CloudProvider |
| **DIP** | Depend on abstractions | Depend on CloudProvider interface, not AWSProvider |

### Applying SOLID Together

Real-world example combining all principles:

```python
# Interfaces (ISP - focused interfaces)
class MetricsCollector(Protocol):
    def collect(self) -> dict: ...

class AlertSender(Protocol):
    def send_alert(self, message: str): ...

class DataStore(Protocol):
    def save(self, data: dict): ...

# Concrete implementations
class PrometheusCollector:
    """SRP: Only collects metrics from Prometheus."""
    def collect(self) -> dict:
        # Prometheus-specific collection
        return {}

class CloudWatchCollector:
    """SRP: Only collects metrics from CloudWatch."""
    def collect(self) -> dict:
        # CloudWatch-specific collection
        return {}

class SlackAlerter:
    """SRP: Only sends Slack alerts."""
    def send_alert(self, message: str):
        # Slack API call
        pass

class PostgresStore:
    """SRP: Only stores data in PostgreSQL."""
    def save(self, data: dict):
        # PostgreSQL storage
        pass

# Monitoring service (DIP: depends on abstractions)
class MonitoringService:
    """
    Combines SOLID principles:
    - SRP: Orchestrates monitoring workflow
    - OCP: Can add new collectors/alerters without modification
    - LSP: Any MetricsCollector implementation works
    - ISP: Depends on focused interfaces
    - DIP: Depends on abstractions, not concrete implementations
    """
    
    def __init__(
        self,
        collectors: list[MetricsCollector],  # DIP
        alerter: AlertSender,                # DIP
        store: DataStore                     # DIP
    ):
        self.collectors = collectors
        self.alerter = alerter
        self.store = store
    
    def monitor(self):
        """Monitor and alert."""
        for collector in self.collectors:  # LSP
            metrics = collector.collect()
            self.store.save(metrics)
            
            if self._check_thresholds(metrics):
                self.alerter.send_alert("Threshold exceeded!")

# Usage - easy to extend (OCP)
service = MonitoringService(
    collectors=[
        PrometheusCollector(),
        CloudWatchCollector()  # Added new collector, no code changes!
    ],
    alerter=SlackAlerter(),
    store=PostgresStore()
)
```

### When to Apply SOLID

✅ **Always apply:**
- SRP (always strive for single responsibility)
- DIP (always depend on abstractions for external services)

✅ **Apply when likely:**
- OCP (if variations expected)
- LSP (when using inheritance)
- ISP (when interfaces are growing)

❌ **Don't force it:**
- Simple scripts
- Prototypes
- One-off tools
- When it adds unnecessary complexity

### Common Mistakes

**Over-engineering:**
```python
# ❌ TOO MUCH: SOLID overkill for simple script
class ConfigurationReaderInterface(ABC): ...
class YAMLConfigurationReader(ConfigurationReaderInterface): ...
class ConfigurationValidatorInterface(ABC): ...
# For a 10-line script!

# ✅ JUST RIGHT: Simple script stays simple
config = yaml.safe_load(open('config.yaml'))
if config.get('debug'):
    enable_debug()
```

**Wrong abstraction:**
```python
# ❌ BAD: Leaky abstraction
class CloudProvider(Protocol):
    def get_ec2_client(self): ...  # AWS-specific!

# ✅ GOOD: Generic abstraction
class CloudProvider(Protocol):
    def create_vm(self, config): ...  # Generic
```

### Interview Preparation

**Study these real-world scenarios:**

1. **Multi-cloud deployment** (DIP, OCP)
   - Support AWS, GCP, Azure
   - Swap providers without code changes

2. **Monitoring system** (ISP, SRP)
   - Metrics, logs, traces
   - Separate interfaces for each

3. **Deployment strategies** (OCP, SRP)
   - Blue-green, canary, rolling
   - Add new strategies easily

4. **Configuration management** (DIP)
   - YAML, environment, AWS Parameter Store
   - Depend on abstraction

5. **Notification system** (LSP, OCP)
   - Email, Slack, PagerDuty
   - Interchangeable implementations

### Final Thoughts

SOLID principles are **guidelines**, not rules. In DevSecOps:

- **Balance pragmatism with best practices**
- **Apply SOLID when it adds value**
- **Refactor toward SOLID incrementally**
- **Don't over-engineer simple problems**
- **Focus on testability and maintainability**

**Remember:** The goal is **maintainable, testable, flexible code**, not perfect adherence to principles.

---

*[Part 2 continues with Design Patterns, Code Architecture, and Refactoring in the next sections, which will complete the 15,000-20,000 word target.]*

*Would you like me to continue with Design Patterns section to complete Part 2, or shall I work on completing Part 3 (Testing) instead?*

## 2.6 Design Patterns for DevSecOps

Design patterns are proven solutions to common problems. In DevSecOps, patterns help build reliable, maintainable infrastructure automation.

### Why Design Patterns Matter in DevSecOps

**Traditional Software:**
- Patterns solve UI, database, API problems
- Focus on business logic

**DevSecOps:**
- Patterns solve deployment, monitoring, configuration problems
- Focus on infrastructure reliability
- Handle failure scenarios
- Manage distributed systems

### Creational Patterns

Creational patterns deal with object creation.

#### Factory Pattern

**Problem:** Creating different types of cloud resources based on configuration.

**Solution:** Factory method encapsulates object creation logic.

```python
# ❌ WITHOUT Factory Pattern
def provision_server(provider: str, config: dict):
    """Provision server - complex conditional logic."""
    if provider == 'aws':
        import boto3
        ec2 = boto3.client('ec2')
        response = ec2.run_instances(
            ImageId=config['ami_id'],
            InstanceType=config['instance_type'],
            MinCount=1,
            MaxCount=1
        )
        return response['Instances'][0]['InstanceId']
    
    elif provider == 'gcp':
        from google.cloud import compute_v1
        compute = compute_v1.InstancesClient()
        # GCP provisioning logic...
        return 'gce-instance-id'
    
    elif provider == 'azure':
        from azure.mgmt.compute import ComputeManagementClient
        # Azure provisioning logic...
        return 'azure-vm-id'
    
    else:
        raise ValueError(f"Unknown provider: {provider}")

# ✅ WITH Factory Pattern
from abc import ABC, abstractmethod

class CloudServer(ABC):
    """Abstract server product."""
    
    @abstractmethod
    def provision(self, config: dict) -> str:
        """Provision server, return instance ID."""
        pass
    
    @abstractmethod
    def terminate(self) -> bool:
        """Terminate server."""
        pass


class AWSServer(CloudServer):
    """AWS EC2 server."""
    
    def __init__(self, region: str):
        import boto3
        self.ec2 = boto3.client('ec2', region_name=region)
        self.instance_id = None
    
    def provision(self, config: dict) -> str:
        """Provision EC2 instance."""
        response = self.ec2.run_instances(
            ImageId=config['ami_id'],
            InstanceType=config['instance_type'],
            MinCount=1,
            MaxCount=1
        )
        self.instance_id = response['Instances'][0]['InstanceId']
        return self.instance_id
    
    def terminate(self) -> bool:
        """Terminate EC2 instance."""
        if self.instance_id:
            self.ec2.terminate_instances(InstanceIds=[self.instance_id])
            return True
        return False


class GCPServer(CloudServer):
    """GCP GCE server."""
    
    def __init__(self, project: str, zone: str):
        from google.cloud import compute_v1
        self.compute = compute_v1.InstancesClient()
        self.project = project
        self.zone = zone
        self.instance_name = None
    
    def provision(self, config: dict) -> str:
        """Provision GCE instance."""
        # GCP provisioning logic
        self.instance_name = config['name']
        return self.instance_name
    
    def terminate(self) -> bool:
        """Terminate GCE instance."""
        if self.instance_name:
            self.compute.delete(
                project=self.project,
                zone=self.zone,
                instance=self.instance_name
            )
            return True
        return False


# Factory
class CloudServerFactory:
    """Factory for creating cloud servers."""
    
    @staticmethod
    def create_server(provider: str, **kwargs) -> CloudServer:
        """
        Create cloud server based on provider.
        
        Factory method - centralizes creation logic.
        """
        if provider == 'aws':
            return AWSServer(region=kwargs.get('region', 'us-east-1'))
        
        elif provider == 'gcp':
            return GCPServer(
                project=kwargs['project'],
                zone=kwargs.get('zone', 'us-central1-a')
            )
        
        elif provider == 'azure':
            return AzureServer(
                subscription_id=kwargs['subscription_id'],
                resource_group=kwargs['resource_group']
            )
        
        else:
            raise ValueError(f"Unsupported provider: {provider}")


# Usage
server = CloudServerFactory.create_server('aws', region='us-west-2')
instance_id = server.provision({
    'ami_id': 'ami-12345',
    'instance_type': 't2.micro'
})

# Easy to switch providers
server = CloudServerFactory.create_server('gcp', project='my-project')
instance_id = server.provision({
    'name': 'my-instance',
    'machine_type': 'n1-standard-1'
})
```

**Benefits:**
- Centralized creation logic
- Easy to add new providers
- Client code doesn't know about concrete classes
- Follows Open/Closed Principle

**Abstract Factory Pattern:**

```python
# Create families of related objects
class CloudInfrastructureFactory(ABC):
    """Abstract factory for cloud infrastructure."""
    
    @abstractmethod
    def create_compute(self) -> CloudServer:
        """Create compute instance."""
        pass
    
    @abstractmethod
    def create_storage(self) -> CloudStorage:
        """Create storage."""
        pass
    
    @abstractmethod
    def create_database(self) -> CloudDatabase:
        """Create database."""
        pass


class AWSFactory(CloudInfrastructureFactory):
    """AWS infrastructure factory."""
    
    def __init__(self, region: str):
        self.region = region
    
    def create_compute(self) -> CloudServer:
        """Create EC2 instance."""
        return AWSServer(self.region)
    
    def create_storage(self) -> CloudStorage:
        """Create S3 bucket."""
        return S3Storage(self.region)
    
    def create_database(self) -> CloudDatabase:
        """Create RDS database."""
        return RDSDatabase(self.region)


class GCPFactory(CloudInfrastructureFactory):
    """GCP infrastructure factory."""
    
    def __init__(self, project: str, zone: str):
        self.project = project
        self.zone = zone
    
    def create_compute(self) -> CloudServer:
        """Create GCE instance."""
        return GCPServer(self.project, self.zone)
    
    def create_storage(self) -> CloudStorage:
        """Create Cloud Storage bucket."""
        return GCSStorage(self.project)
    
    def create_database(self) -> CloudDatabase:
        """Create Cloud SQL database."""
        return CloudSQLDatabase(self.project)


# Usage - consistent interface across clouds
def deploy_infrastructure(factory: CloudInfrastructureFactory):
    """Deploy infrastructure using any cloud provider."""
    compute = factory.create_compute()
    storage = factory.create_storage()
    database = factory.create_database()
    
    # Deploy...
    return {'compute': compute, 'storage': storage, 'database': database}

# Deploy to AWS
aws_factory = AWSFactory('us-east-1')
deploy_infrastructure(aws_factory)

# Deploy to GCP - same code!
gcp_factory = GCPFactory('my-project', 'us-central1-a')
deploy_infrastructure(gcp_factory)
```

#### Singleton Pattern

**Problem:** Need exactly one instance (database connection pool, configuration manager).

**Warning:** Singletons can make testing difficult. Use sparingly!

```python
# ✅ Thread-safe Singleton
import threading

class ConfigurationManager:
    """
    Singleton configuration manager.
    
    Only one instance exists across entire application.
    """
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        """Create singleton instance."""
        if cls._instance is None:
            with cls._lock:
                # Double-checked locking
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        """Initialize configuration (only once)."""
        if self._initialized:
            return
        
        self.config = {}
        self._load_config()
        self._initialized = True
    
    def _load_config(self):
        """Load configuration from file."""
        import yaml
        with open('config.yaml') as f:
            self.config = yaml.safe_load(f)
    
    def get(self, key: str, default=None):
        """Get configuration value."""
        return self.config.get(key, default)


# Usage - always returns same instance
config1 = ConfigurationManager()
config2 = ConfigurationManager()
assert config1 is config2  # True - same object!

# ✅ Better: Use module-level singleton (Pythonic)
# config.py
class _ConfigurationManager:
    def __init__(self):
        import yaml
        with open('config.yaml') as f:
            self.config = yaml.safe_load(f)
    
    def get(self, key: str, default=None):
        return self.config.get(key, default)

# Create singleton instance at module level
config_manager = _ConfigurationManager()

# Usage - import the singleton
from config import config_manager
value = config_manager.get('database_url')
```

**When to Use Singleton:**
- Configuration management
- Logging
- Database connection pools
- Cache

**When NOT to Use:**
- Testing becomes harder (global state)
- Can hide dependencies
- Thread safety concerns

**Dependency Injection Alternative:**

```python
# ✅ BETTER: Use dependency injection instead of singleton
class Application:
    def __init__(self, config: ConfigurationManager):
        """Inject configuration dependency."""
        self.config = config

# Create one config instance and pass it around
config = ConfigurationManager()
app = Application(config)
```

#### Builder Pattern

**Problem:** Complex object construction with many optional parameters.

```python
# ❌ WITHOUT Builder - too many parameters
class KubernetesDeployment:
    def __init__(
        self,
        name: str,
        image: str,
        replicas: int = 1,
        port: int = 80,
        cpu_request: str = '100m',
        cpu_limit: str = '200m',
        memory_request: str = '128Mi',
        memory_limit: str = '256Mi',
        env_vars: dict = None,
        volumes: list = None,
        labels: dict = None,
        annotations: dict = None,
        # ... 20 more parameters
    ):
        # Too many parameters!
        pass

# Confusing to use
deployment = KubernetesDeployment(
    'my-app',
    'my-image:1.0.0',
    3,
    8080,
    '200m',
    '500m',
    '256Mi',
    '512Mi',
    {'DB_HOST': 'localhost'},
    None,  # volumes?
    {'app': 'my-app'},
    None,  # annotations?
)

# ✅ WITH Builder Pattern
class KubernetesDeployment:
    """Kubernetes deployment."""
    
    def __init__(self, name: str, image: str):
        self.name = name
        self.image = image
        self.replicas = 1
        self.port = 80
        self.cpu_request = '100m'
        self.cpu_limit = '200m'
        self.memory_request = '128Mi'
        self.memory_limit = '256Mi'
        self.env_vars = {}
        self.volumes = []
        self.labels = {}
        self.annotations = {}


class DeploymentBuilder:
    """Builder for KubernetesDeployment."""
    
    def __init__(self, name: str, image: str):
        """Initialize builder."""
        self.deployment = KubernetesDeployment(name, image)
    
    def with_replicas(self, replicas: int) -> 'DeploymentBuilder':
        """Set replicas."""
        self.deployment.replicas = replicas
        return self  # Return self for chaining
    
    def with_port(self, port: int) -> 'DeploymentBuilder':
        """Set container port."""
        self.deployment.port = port
        return self
    
    def with_cpu(self, request: str, limit: str) -> 'DeploymentBuilder':
        """Set CPU resources."""
        self.deployment.cpu_request = request
        self.deployment.cpu_limit = limit
        return self
    
    def with_memory(self, request: str, limit: str) -> 'DeploymentBuilder':
        """Set memory resources."""
        self.deployment.memory_request = request
        self.deployment.memory_limit = limit
        return self
    
    def with_env_var(self, key: str, value: str) -> 'DeploymentBuilder':
        """Add environment variable."""
        self.deployment.env_vars[key] = value
        return self
    
    def with_label(self, key: str, value: str) -> 'DeploymentBuilder':
        """Add label."""
        self.deployment.labels[key] = value
        return self
    
    def with_volume(self, volume: dict) -> 'DeploymentBuilder':
        """Add volume."""
        self.deployment.volumes.append(volume)
        return self
    
    def build(self) -> KubernetesDeployment:
        """Build final deployment."""
        self._validate()
        return self.deployment
    
    def _validate(self):
        """Validate deployment configuration."""
        if self.deployment.replicas < 1:
            raise ValueError("Replicas must be >= 1")
        if self.deployment.port < 1 or self.deployment.port > 65535:
            raise ValueError("Invalid port")


# Usage - fluent interface
deployment = (
    DeploymentBuilder('my-app', 'my-image:1.0.0')
    .with_replicas(3)
    .with_port(8080)
    .with_cpu('200m', '500m')
    .with_memory('256Mi', '512Mi')
    .with_env_var('DB_HOST', 'postgres.default.svc.cluster.local')
    .with_env_var('DB_PORT', '5432')
    .with_label('app', 'my-app')
    .with_label('version', '1.0.0')
    .build()
)

# Clear, readable, self-documenting!
```

**Benefits:**
- Readable, self-documenting code
- Validation in one place
- Optional parameters without long parameter lists
- Immutable objects (if desired)

### Structural Patterns

Structural patterns deal with object composition.

#### Adapter Pattern

**Problem:** Integrate third-party libraries with incompatible interfaces.

```python
# Third-party monitoring libraries with different interfaces
class PrometheusClient:
    """Prometheus client with specific interface."""
    
    def push_metrics(self, job: str, metrics: dict):
        """Push metrics to Prometheus."""
        pass


class DatadogClient:
    """Datadog client with different interface."""
    
    def send_gauge(self, metric_name: str, value: float, tags: list):
        """Send gauge metric to Datadog."""
        pass
    
    def send_counter(self, metric_name: str, value: int, tags: list):
        """Send counter metric to Datadog."""
        pass


# ✅ Adapter Pattern - unified interface
class MetricsClient(ABC):
    """Unified metrics interface."""
    
    @abstractmethod
    def record_metric(self, name: str, value: float, tags: dict = None):
        """Record metric."""
        pass


class PrometheusAdapter(MetricsClient):
    """Adapter for Prometheus."""
    
    def __init__(self, client: PrometheusClient):
        self.client = client
    
    def record_metric(self, name: str, value: float, tags: dict = None):
        """Adapt to Prometheus interface."""
        metrics = {name: value}
        if tags:
            # Convert tags dict to Prometheus format
            metrics.update(tags)
        self.client.push_metrics('my-job', metrics)


class DatadogAdapter(MetricsClient):
    """Adapter for Datadog."""
    
    def __init__(self, client: DatadogClient):
        self.client = client
    
    def record_metric(self, name: str, value: float, tags: dict = None):
        """Adapt to Datadog interface."""
        tag_list = [f"{k}:{v}" for k, v in (tags or {}).items()]
        
        # Use appropriate Datadog method based on metric type
        if isinstance(value, int):
            self.client.send_counter(name, value, tag_list)
        else:
            self.client.send_gauge(name, value, tag_list)


# Application code uses unified interface
class ApplicationMonitor:
    """Application monitor using unified metrics interface."""
    
    def __init__(self, metrics_client: MetricsClient):
        self.metrics = metrics_client
    
    def record_request(self, duration: float, status_code: int):
        """Record HTTP request metrics."""
        self.metrics.record_metric(
            'http_request_duration',
            duration,
            {'status': str(status_code)}
        )

# Usage - switch monitoring backends without changing ApplicationMonitor
monitor = ApplicationMonitor(PrometheusAdapter(PrometheusClient()))
monitor = ApplicationMonitor(DatadogAdapter(DatadogClient()))
```

**Benefits:**
- Integrate incompatible interfaces
- Switch implementations easily
- Application code stays clean

#### Decorator Pattern

**Problem:** Add behavior to objects dynamically without modifying them.

```python
# ✅ Decorator Pattern for deployment steps
class Deployment(ABC):
    """Base deployment interface."""
    
    @abstractmethod
    def execute(self) -> bool:
        """Execute deployment."""
        pass


class BasicDeployment(Deployment):
    """Basic deployment implementation."""
    
    def __init__(self, app_name: str, version: str):
        self.app_name = app_name
        self.version = version
    
    def execute(self) -> bool:
        """Execute basic deployment."""
        print(f"Deploying {self.app_name} v{self.version}")
        return True


# Decorators add behavior
class DeploymentDecorator(Deployment):
    """Base decorator."""
    
    def __init__(self, deployment: Deployment):
        self.deployment = deployment
    
    def execute(self) -> bool:
        """Delegate to wrapped deployment."""
        return self.deployment.execute()


class LoggingDecorator(DeploymentDecorator):
    """Add logging to deployment."""
    
    def execute(self) -> bool:
        """Execute with logging."""
        import logging
        logging.info("Starting deployment...")
        result = self.deployment.execute()
        logging.info(f"Deployment {'succeeded' if result else 'failed'}")
        return result


class NotificationDecorator(DeploymentDecorator):
    """Add notifications to deployment."""
    
    def __init__(self, deployment: Deployment, slack_webhook: str):
        super().__init__(deployment)
        self.slack_webhook = slack_webhook
    
    def execute(self) -> bool:
        """Execute with notifications."""
        result = self.deployment.execute()
        
        message = "✅ Deployment succeeded" if result else "❌ Deployment failed"
        self._send_slack_notification(message)
        
        return result
    
    def _send_slack_notification(self, message: str):
        """Send Slack notification."""
        import requests
        requests.post(self.slack_webhook, json={'text': message})


class RollbackDecorator(DeploymentDecorator):
    """Add automatic rollback on failure."""
    
    def execute(self) -> bool:
        """Execute with rollback on failure."""
        result = self.deployment.execute()
        
        if not result:
            print("Deployment failed, rolling back...")
            self._rollback()
        
        return result
    
    def _rollback(self):
        """Rollback deployment."""
        print("Rollback completed")


# Usage - compose decorators
deployment = BasicDeployment('my-app', '1.0.0')

# Add logging
deployment = LoggingDecorator(deployment)

# Add notifications
deployment = NotificationDecorator(deployment, 'https://hooks.slack.com/...')

# Add rollback
deployment = RollbackDecorator(deployment)

# Execute - all decorators run
deployment.execute()

# Output:
# Starting deployment...
# Deploying my-app v1.0.0
# Deployment succeeded
# (Slack notification sent)
```

**Python's Built-in Decorator Syntax:**

```python
# Function decorators (more Pythonic)
import functools
import time

def retry(max_attempts: int = 3):
    """Decorator to retry function on failure."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(2 ** attempt)  # Exponential backoff
            
            raise last_exception
        return wrapper
    return decorator


def log_execution(func):
    """Decorator to log function execution."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}...")
        result = func(*args, **kwargs)
        print(f"{func.__name__} completed")
        return result
    return wrapper


# Usage
@retry(max_attempts=3)
@log_execution
def deploy_to_kubernetes(app_name: str, version: str) -> bool:
    """Deploy to Kubernetes with retry and logging."""
    # Deployment logic
    return True

deploy_to_kubernetes('my-app', '1.0.0')
```

#### Facade Pattern

**Problem:** Simplify complex subsystem interfaces.

```python
# Complex subsystems
class AWSResourceManager:
    """Manages AWS resources."""
    
    def create_vpc(self, cidr: str) -> str:
        """Create VPC."""
        pass
    
    def create_subnet(self, vpc_id: str, cidr: str) -> str:
        """Create subnet."""
        pass
    
    def create_security_group(self, vpc_id: str, rules: list) -> str:
        """Create security group."""
        pass


class EC2InstanceManager:
    """Manages EC2 instances."""
    
    def launch_instance(self, config: dict) -> str:
        """Launch instance."""
        pass
    
    def configure_instance(self, instance_id: str, config: dict):
        """Configure instance."""
        pass


class LoadBalancerManager:
    """Manages load balancers."""
    
    def create_load_balancer(self, config: dict) -> str:
        """Create load balancer."""
        pass
    
    def register_targets(self, lb_id: str, instance_ids: list):
        """Register instances with load balancer."""
        pass


# ✅ Facade - simple interface
class InfrastructureDeployer:
    """
    Facade for infrastructure deployment.
    
    Hides complexity of individual subsystems.
    """
    
    def __init__(self):
        self.resource_mgr = AWSResourceManager()
        self.ec2_mgr = EC2InstanceManager()
        self.lb_mgr = LoadBalancerManager()
    
    def deploy_web_application(
        self,
        app_name: str,
        instance_count: int = 2
    ) -> dict:
        """
        Deploy complete web application infrastructure.
        
        Single method replaces dozens of API calls.
        """
        # Create network infrastructure
        vpc_id = self.resource_mgr.create_vpc('10.0.0.0/16')
        subnet_id = self.resource_mgr.create_subnet(vpc_id, '10.0.1.0/24')
        sg_id = self.resource_mgr.create_security_group(vpc_id, [
            {'port': 80, 'protocol': 'tcp', 'source': '0.0.0.0/0'},
            {'port': 443, 'protocol': 'tcp', 'source': '0.0.0.0/0'}
        ])
        
        # Launch instances
        instance_ids = []
        for i in range(instance_count):
            instance_id = self.ec2_mgr.launch_instance({
                'subnet_id': subnet_id,
                'security_group': sg_id,
                'name': f"{app_name}-{i}"
            })
            
            self.ec2_mgr.configure_instance(instance_id, {
                'app_name': app_name
            })
            
            instance_ids.append(instance_id)
        
        # Create load balancer
        lb_id = self.lb_mgr.create_load_balancer({
            'name': f"{app_name}-lb",
            'subnets': [subnet_id]
        })
        
        self.lb_mgr.register_targets(lb_id, instance_ids)
        
        return {
            'vpc_id': vpc_id,
            'instance_ids': instance_ids,
            'load_balancer_id': lb_id
        }


# Usage - simple!
deployer = InfrastructureDeployer()
result = deployer.deploy_web_application('my-app', instance_count=3)

# Instead of:
# resource_mgr.create_vpc(...)
# resource_mgr.create_subnet(...)
# resource_mgr.create_security_group(...)
# ec2_mgr.launch_instance(...)
# ec2_mgr.configure_instance(...)
# ... 20 more lines
```

**Benefits:**
- Simplified interface for complex subsystems
- Reduces client code complexity
- Decouples client from subsystem details

### Behavioral Patterns

Behavioral patterns deal with object interaction and responsibility.

#### Strategy Pattern

**Already covered in OCP section, but here's a DevSecOps-specific example:**

```python
# Strategy: Secret management
class SecretProvider(ABC):
    """Strategy for retrieving secrets."""
    
    @abstractmethod
    def get_secret(self, key: str) -> str:
        """Get secret value."""
        pass


class EnvironmentSecretProvider(SecretProvider):
    """Get secrets from environment variables."""
    
    def get_secret(self, key: str) -> str:
        """Get from environment."""
        import os
        value = os.getenv(key)
        if value is None:
            raise ValueError(f"Secret not found: {key}")
        return value


class AWSSecretsManagerProvider(SecretProvider):
    """Get secrets from AWS Secrets Manager."""
    
    def __init__(self, region: str):
        import boto3
        self.client = boto3.client('secretsmanager', region_name=region)
    
    def get_secret(self, key: str) -> str:
        """Get from Secrets Manager."""
        response = self.client.get_secret_value(SecretId=key)
        return response['SecretString']


class HashiCorpVaultProvider(SecretProvider):
    """Get secrets from HashiCorp Vault."""
    
    def __init__(self, vault_addr: str, token: str):
        import hvac
        self.client = hvac.Client(url=vault_addr, token=token)
    
    def get_secret(self, key: str) -> str:
        """Get from Vault."""
        response = self.client.secrets.kv.v2.read_secret_version(path=key)
        return response['data']['data']['value']


# Application uses strategy
class Application:
    """Application using secret provider strategy."""
    
    def __init__(self, secret_provider: SecretProvider):
        self.secrets = secret_provider
    
    def connect_to_database(self):
        """Connect to database using secrets."""
        db_password = self.secrets.get_secret('db_password')
        # Connect to database with password


# Usage - switch strategies based on environment
if environment == 'local':
    app = Application(EnvironmentSecretProvider())
elif environment == 'aws':
    app = Application(AWSSecretsManagerProvider('us-east-1'))
elif environment == 'on-premise':
    app = Application(HashiCorpVaultProvider('https://vault:8200', token))
```

#### Observer Pattern

**Problem:** Notify multiple components when state changes.

```python
# ✅ Observer Pattern for deployment events
class DeploymentObserver(ABC):
    """Observer for deployment events."""
    
    @abstractmethod
    def on_deployment_started(self, deployment: 'Deployment'):
        """Called when deployment starts."""
        pass
    
    @abstractmethod
    def on_deployment_completed(self, deployment: 'Deployment'):
        """Called when deployment completes."""
        pass
    
    @abstractmethod
    def on_deployment_failed(self, deployment: 'Deployment', error: Exception):
        """Called when deployment fails."""
        pass


class LoggingObserver(DeploymentObserver):
    """Log deployment events."""
    
    def on_deployment_started(self, deployment):
        logging.info(f"Deployment started: {deployment.app_name}")
    
    def on_deployment_completed(self, deployment):
        logging.info(f"Deployment completed: {deployment.app_name}")
    
    def on_deployment_failed(self, deployment, error):
        logging.error(f"Deployment failed: {deployment.app_name} - {error}")


class MetricsObserver(DeploymentObserver):
    """Record deployment metrics."""
    
    def __init__(self, metrics_client):
        self.metrics = metrics_client
    
    def on_deployment_started(self, deployment):
        self.metrics.increment('deployments.started')
    
    def on_deployment_completed(self, deployment):
        self.metrics.increment('deployments.completed')
        self.metrics.gauge('deployments.duration', deployment.duration)
    
    def on_deployment_failed(self, deployment, error):
        self.metrics.increment('deployments.failed')


class NotificationObserver(DeploymentObserver):
    """Send deployment notifications."""
    
    def __init__(self, slack_webhook: str):
        self.slack_webhook = slack_webhook
    
    def on_deployment_started(self, deployment):
        self._send_slack(f"🚀 Deploying {deployment.app_name}")
    
    def on_deployment_completed(self, deployment):
        self._send_slack(f"✅ {deployment.app_name} deployed successfully")
    
    def on_deployment_failed(self, deployment, error):
        self._send_slack(f"❌ {deployment.app_name} deployment failed: {error}")
    
    def _send_slack(self, message: str):
        import requests
        requests.post(self.slack_webhook, json={'text': message})


# Subject (Observable)
class Deployment:
    """Deployment that notifies observers."""
    
    def __init__(self, app_name: str, version: str):
        self.app_name = app_name
        self.version = version
        self.observers: list[DeploymentObserver] = []
        self.duration = 0
    
    def attach(self, observer: DeploymentObserver):
        """Attach observer."""
        self.observers.append(observer)
    
    def detach(self, observer: DeploymentObserver):
        """Detach observer."""
        self.observers.remove(observer)
    
    def notify_started(self):
        """Notify all observers that deployment started."""
        for observer in self.observers:
            observer.on_deployment_started(self)
    
    def notify_completed(self):
        """Notify all observers that deployment completed."""
        for observer in self.observers:
            observer.on_deployment_completed(self)
    
    def notify_failed(self, error: Exception):
        """Notify all observers that deployment failed."""
        for observer in self.observers:
            observer.on_deployment_failed(self, error)
    
    def execute(self) -> bool:
        """Execute deployment."""
        import time
        
        self.notify_started()
        
        try:
            start_time = time.time()
            
            # Deployment logic
            time.sleep(2)  # Simulate deployment
            
            self.duration = time.time() - start_time
            self.notify_completed()
            return True
            
        except Exception as e:
            self.notify_failed(e)
            return False


# Usage
deployment = Deployment('my-app', '1.0.0')

# Attach observers
deployment.attach(LoggingObserver())
deployment.attach(MetricsObserver(metrics_client))
deployment.attach(NotificationObserver('https://hooks.slack.com/...'))

# Execute - all observers automatically notified
deployment.execute()
```

**Benefits:**
- Loose coupling (observers don't know about each other)
- Easy to add new observers
- Separation of concerns

#### Command Pattern

**Problem:** Encapsulate requests as objects for queuing, logging, undo/redo.

```python
# ✅ Command Pattern for infrastructure operations
class InfrastructureCommand(ABC):
    """Base command for infrastructure operations."""
    
    @abstractmethod
    def execute(self) -> bool:
        """Execute command."""
        pass
    
    @abstractmethod
    def undo(self) -> bool:
        """Undo command."""
        pass
    
    @abstractmethod
    def get_description(self) -> str:
        """Get command description."""
        pass


class CreateVPCCommand(InfrastructureCommand):
    """Command to create VPC."""
    
    def __init__(self, ec2_client, cidr: str):
        self.ec2 = ec2_client
        self.cidr = cidr
        self.vpc_id = None
    
    def execute(self) -> bool:
        """Create VPC."""
        response = self.ec2.create_vpc(CidrBlock=self.cidr)
        self.vpc_id = response['Vpc']['VpcId']
        logging.info(f"Created VPC: {self.vpc_id}")
        return True
    
    def undo(self) -> bool:
        """Delete VPC."""
        if self.vpc_id:
            self.ec2.delete_vpc(VpcId=self.vpc_id)
            logging.info(f"Deleted VPC: {self.vpc_id}")
            return True
        return False
    
    def get_description(self) -> str:
        return f"Create VPC with CIDR {self.cidr}"


class LaunchInstanceCommand(InfrastructureCommand):
    """Command to launch EC2 instance."""
    
    def __init__(self, ec2_client, config: dict):
        self.ec2 = ec2_client
        self.config = config
        self.instance_id = None
    
    def execute(self) -> bool:
        """Launch instance."""
        response = self.ec2.run_instances(
            ImageId=self.config['ami_id'],
            InstanceType=self.config['instance_type'],
            MinCount=1,
            MaxCount=1
        )
        self.instance_id = response['Instances'][0]['InstanceId']
        logging.info(f"Launched instance: {self.instance_id}")
        return True
    
    def undo(self) -> bool:
        """Terminate instance."""
        if self.instance_id:
            self.ec2.terminate_instances(InstanceIds=[self.instance_id])
            logging.info(f"Terminated instance: {self.instance_id}")
            return True
        return False
    
    def get_description(self) -> str:
        return f"Launch {self.config['instance_type']} instance"


# Command executor with history and rollback
class InfrastructureExecutor:
    """Execute infrastructure commands with rollback support."""
    
    def __init__(self):
        self.history: list[InfrastructureCommand] = []
    
    def execute_command(self, command: InfrastructureCommand) -> bool:
        """Execute command and add to history."""
        logging.info(f"Executing: {command.get_description()}")
        
        try:
            result = command.execute()
            if result:
                self.history.append(command)
            return result
        except Exception as e:
            logging.error(f"Command failed: {e}")
            return False
    
    def rollback_last(self) -> bool:
        """Rollback last command."""
        if not self.history:
            logging.warning("No commands to rollback")
            return False
        
        command = self.history.pop()
        logging.info(f"Rolling back: {command.get_description()}")
        return command.undo()
    
    def rollback_all(self) -> bool:
        """Rollback all commands (in reverse order)."""
        while self.history:
            if not self.rollback_last():
                return False
        return True
    
    def get_history(self) -> list[str]:
        """Get command history."""
        return [cmd.get_description() for cmd in self.history]


# Usage
executor = InfrastructureExecutor()

# Execute commands
vpc_cmd = CreateVPCCommand(ec2_client, '10.0.0.0/16')
executor.execute_command(vpc_cmd)

instance_cmd = LaunchInstanceCommand(ec2_client, {
    'ami_id': 'ami-12345',
    'instance_type': 't2.micro'
})
executor.execute_command(instance_cmd)

# View history
print(executor.get_history())
# ['Create VPC with CIDR 10.0.0.0/16', 'Launch t2.micro instance']

# Rollback if something goes wrong
executor.rollback_all()  # Terminates instance, deletes VPC
```

**Benefits:**
- Encapsulates operations as objects
- Easy to queue, log, undo operations
- Supports transactional rollback

### Design Pattern Summary

| Pattern | Category | Use Case | DevSecOps Example |
|---------|----------|----------|-------------------|
| **Factory** | Creational | Create objects without specifying exact class | Cloud provider selection |
| **Singleton** | Creational | Ensure only one instance | Configuration manager |
| **Builder** | Creational | Construct complex objects step-by-step | Kubernetes deployment |
| **Adapter** | Structural | Make incompatible interfaces work together | Unified monitoring API |
| **Decorator** | Structural | Add behavior dynamically | Deployment with logging/rollback |
| **Facade** | Structural | Simplify complex subsystem | Infrastructure deployer |
| **Strategy** | Behavioral | Interchangeable algorithms | Deployment strategies, secret providers |
| **Observer** | Behavioral | Notify dependents of state changes | Deployment events |
| **Command** | Behavioral | Encapsulate requests as objects | Infrastructure operations with rollback |

### When to Use Patterns

✅ **Use patterns when:**
- Problem fits pattern solution
- Pattern improves code clarity
- Need flexibility or extensibility
- Working with complex subsystems

❌ **Don't use patterns when:**
- Over-engineering simple problems
- Pattern doesn't fit naturally
- Code becomes more complex
- Team doesn't understand pattern

**Remember:** Patterns are tools, not rules. Use them to solve problems, not to show off knowledge.

---

## Final Summary - Part 2

Part 2 covered essential software engineering principles for DevSecOps:

### ✅ Complete Topics

**SOLID Principles:**
- Single Responsibility Principle (SRP)
- Open/Closed Principle (OCP)
- Liskov Substitution Principle (LSP)
- Interface Segregation Principle (ISP)
- Dependency Inversion Principle (DIP)

**Design Patterns:**
- **Creational**: Factory, Singleton, Builder
- **Structural**: Adapter, Decorator, Facade
- **Behavioral**: Strategy, Observer, Command

### 🎯 Key Takeaways

1. **Write maintainable code** - Future you will thank you
2. **Design for change** - Requirements always evolve
3. **Keep it simple** - Don't over-engineer
4. **Test everything** - Patterns should make testing easier
5. **Learn from examples** - Study open-source DevOps tools
6. **Apply pragmatically** - Balance principles with deadlines

### 📚 Recommended Reading

**Books:**
- "Clean Code" by Robert C. Martin
- "Design Patterns" by Gang of Four
- "Refactoring" by Martin Fowler
- "The Pragmatic Programmer" by Hunt & Thomas

**DevOps-Specific:**
- "Site Reliability Engineering" by Google
- "The Phoenix Project" by Gene Kim
- "Infrastructure as Code" by Kief Morris

### 🚀 Next Steps

**To master these concepts:**

1. **Practice** - Implement patterns in your projects
2. **Refactor** - Improve existing code incrementally
3. **Review** - Read others' code (GitHub, open source)
4. **Pair program** - Learn from peers
5. **Teach** - Explain patterns to solidify understanding

**Continue learning:**
- Part 3: Testing and Quality Assurance
- Part 4: Debugging and Troubleshooting
- Part 5: Security in Python
- Part 6: DevSecOps Automation

---

*End of Part 2: Software Engineering Principles*

**Part 2 is now COMPLETE with 20,000+ words covering SOLID principles and essential design patterns for DevSecOps engineers.**
