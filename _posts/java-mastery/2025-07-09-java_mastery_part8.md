---
title: "Complete Java Mastery Part 8: Cloud-Native Java"
date: 2024-01-22 10:00:00 +0000
categories: [Java, Cloud-Native, Kubernetes]
tags: [java, docker, kubernetes, cloud-native, serverless, aws, gcp, azure, helm, service-mesh, graalvm]
---

## Introduction

Welcome to Part 8 of the Complete Java Mastery series. In Parts 1-7, we covered Java fundamentals through reactive programming. Now we explore **Cloud-Native Java** - building, deploying, and scaling Java applications in cloud environments.

Cloud-native represents a fundamental shift in how we build and run applications. It's not just about deploying to the cloud—it's about designing applications specifically for cloud infrastructure, embracing containers, orchestration, and distributed systems patterns.

### What You'll Learn in Part 8

* **Containerization with Docker**: Building optimized Java images, multi-stage builds
* **Kubernetes Fundamentals**: Pods, Deployments, Services, ConfigMaps, Secrets
* **Kubernetes Advanced**: StatefulSets, Ingress, Persistent Volumes, Operators
* **Helm**: Package management, charts, releases, templating
* **Cloud Platforms**: AWS, GCP, Azure deployment strategies
* **Serverless Java**: AWS Lambda, GCP Cloud Functions, performance optimization
* **Service Mesh**: Istio, traffic management, observability
* **GraalVM**: Native images, faster startup, lower memory footprint
* **Cloud-Native Patterns**: 12-Factor App, health checks, graceful shutdown

This knowledge enables you to:
* Build production-ready container images
* Deploy and scale applications on Kubernetes
* Leverage cloud platform services effectively
* Optimize Java for serverless environments
* Implement service mesh for microservices
* Create native executables with GraalVM

---

## 8.1 Containerization with Docker

Docker packages applications with all dependencies into portable containers.

### Basic Dockerfile for Java

```dockerfile
# ❌ SIMPLE BUT INEFFICIENT
FROM openjdk:21
COPY target/myapp.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]

# Problems:
# - Large image size (~400MB)
# - Full JDK when only JRE needed
# - No layer optimization
# - Runs as root (security risk)
```

### Optimized Multi-Stage Dockerfile

```dockerfile
# ✅ PRODUCTION-READY DOCKERFILE

# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app

# Copy only pom.xml first (layer caching)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Change ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Expose port
EXPOSE 8080

# JVM optimization for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

# Run application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Benefits:**
* **Smaller image**: ~150MB vs ~400MB
* **Layer caching**: Dependencies cached separately
* **Security**: Non-root user
* **Health checks**: Container health monitoring
* **JVM optimized**: Container-aware memory settings

### Spring Boot Layered JARs

**pom.xml:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <layers>
                    <enabled>true</enabled>
                </layers>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Layered Dockerfile:**

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

# Runtime stage with layers
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app

# Layer 1: Dependencies (changes rarely)
COPY --from=build /app/target/dependency/BOOT-INF/lib ./lib

# Layer 2: Spring Boot loader
COPY --from=build /app/target/dependency/org ./org

# Layer 3: Snapshot dependencies (if any)
COPY --from=build /app/target/dependency/BOOT-INF/lib-snapshot ./lib-snapshot

# Layer 4: Application classes (changes frequently)
COPY --from=build /app/target/dependency/BOOT-INF/classes ./classes
COPY --from=build /app/target/dependency/META-INF ./META-INF

RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 8080

ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -cp 'classes:lib/*' org.springframework.boot.loader.JarLauncher"]
```

**Why Layers Matter:**

```
Docker layer caching:
1. Dependencies layer ──────► Cached (rarely changes)
2. Spring Boot loader ─────► Cached (rarely changes)
3. Application classes ────► Rebuilt (changes frequently)

Result: Faster builds, smaller uploads to registry
```

### Docker Compose for Local Development

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/myapp
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret
      SPRING_REDIS_HOST: redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    volumes:
      - ./logs:/app/logs
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
  
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

**Usage:**

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f app

# Scale application
docker-compose up -d --scale app=3

# Stop all services
docker-compose down

# Remove volumes
docker-compose down -v
```

### Docker Best Practices

```dockerfile
# ✅ BEST PRACTICES DOCKERFILE

FROM eclipse-temurin:21-jre-alpine AS runtime

# 1. Use specific versions (not 'latest')
# FROM eclipse-temurin:21-jre-alpine   ✅
# FROM openjdk:latest                   ❌

# 2. Minimize layers (combine RUN commands)
RUN apk add --no-cache curl wget \
    && addgroup -S appgroup \
    && adduser -S appuser -G appgroup

# 3. Order layers by change frequency
WORKDIR /app
COPY --chown=appuser:appgroup lib/ ./lib/       # Rarely changes
COPY --chown=appuser:appgroup classes/ ./       # Frequently changes

# 4. Don't run as root
USER appuser

# 5. Use COPY instead of ADD (unless extracting archives)
COPY app.jar .

# 6. Use .dockerignore
# .dockerignore file:
# target/
# .git/
# .idea/
# *.md

# 7. Use health checks
HEALTHCHECK --interval=30s --timeout=3s CMD wget -q --spider http://localhost:8080/health || exit 1

# 8. Use ENTRYPOINT for executables
ENTRYPOINT ["java", "-jar", "app.jar"]

# 9. Use CMD for default arguments
CMD ["--spring.profiles.active=prod"]

# 10. Set memory limits
ENV JAVA_OPTS="-Xmx512m -Xms256m"
```

### Building and Running

```bash
# Build image
docker build -t myapp:1.0.0 .

# Tag for registry
docker tag myapp:1.0.0 myregistry.com/myapp:1.0.0

# Push to registry
docker push myregistry.com/myapp:1.0.0

# Run container
docker run -d \
  --name myapp \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e JAVA_OPTS="-Xmx1g" \
  --memory=1g \
  --cpus=2 \
  --restart=unless-stopped \
  myapp:1.0.0

# View logs
docker logs -f myapp

# Execute command in container
docker exec -it myapp sh

# Stop and remove
docker stop myapp
docker rm myapp

# Clean up
docker system prune -a
```

---

## 8.2 Kubernetes Fundamentals

Kubernetes orchestrates containerized applications at scale.

### Key Concepts

```
Kubernetes Architecture:

┌─────────────────────────────────────────────┐
│              Control Plane                   │
│  ┌──────────┐  ┌────────┐  ┌────────────┐  │
│  │API Server│  │Scheduler│  │ Controller │  │
│  └──────────┘  └────────┘  └────────────┘  │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
┌─────────────┐         ┌─────────────┐
│   Node 1    │         │   Node 2    │
│             │         │             │
│  ┌──────┐   │         │  ┌──────┐   │
│  │ Pod  │   │         │  │ Pod  │   │
│  └──────┘   │         │  └──────┘   │
│  ┌──────┐   │         │  ┌──────┐   │
│  │ Pod  │   │         │  │ Pod  │   │
│  └──────┘   │         │  └──────┘   │
└─────────────┘         └─────────────┘
```

### Pod - Basic Deployment Unit

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    version: v1
spec:
  containers:
  - name: myapp
    image: myregistry.com/myapp:1.0.0
    ports:
    - containerPort: 8080
      name: http
    env:
    - name: SPRING_PROFILES_ACTIVE
      value: "prod"
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
    livenessProbe:
      httpGet:
        path: /actuator/health/liveness
        port: 8080
      initialDelaySeconds: 60
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /actuator/health/readiness
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

### Deployment - Managed Replicas

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: myregistry.com/myapp:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: JAVA_OPTS
          value: "-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
```

### Service - Network Access

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  type: LoadBalancer  # or ClusterIP, NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    name: http
  sessionAffinity: ClientIP  # Sticky sessions
```

**Service Types:**

```yaml
# ClusterIP (default) - Internal only
apiVersion: v1
kind: Service
metadata:
  name: myapp-internal
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 8080
    targetPort: 8080

---
# NodePort - Exposed on each node
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080  # Accessible on node:30080

---
# LoadBalancer - Cloud load balancer
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### ConfigMap - Configuration

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  application.properties: |
    server.port=8080
    spring.datasource.url=jdbc:postgresql://postgres:5432/myapp
    logging.level.root=INFO
    logging.level.com.example=DEBUG
  
  database.host: "postgres.default.svc.cluster.local"
  cache.ttl: "3600"
```

**Using ConfigMap:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        # As environment variables
        env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: database.host
        - name: CACHE_TTL
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: cache.ttl
        # As volume
        volumeMounts:
        - name: config-volume
          mountPath: /config
      volumes:
      - name: config-volume
        configMap:
          name: myapp-config
```

### Secret - Sensitive Data

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  # Base64 encoded values
  database-password: cGFzc3dvcmQxMjM=
  api-key: YWJjZGVmZ2hpamtsbW5vcA==
stringData:
  # Plain text (automatically encoded)
  jwt-secret: my-secret-key-12345
```

**Using Secret:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        env:
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: database-password
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: api-key
        # As volume (files)
        volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: myapp-secret
```

### Applying Resources

```bash
# Apply single file
kubectl apply -f deployment.yaml

# Apply directory
kubectl apply -f kubernetes/

# Apply from URL
kubectl apply -f https://example.com/app.yaml

# View resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get configmaps
kubectl get secrets

# Describe resource
kubectl describe pod myapp-pod
kubectl describe deployment myapp-deployment

# View logs
kubectl logs myapp-pod
kubectl logs -f myapp-pod  # Follow
kubectl logs myapp-pod --previous  # Previous container

# Execute command
kubectl exec -it myapp-pod -- /bin/sh

# Port forward (local testing)
kubectl port-forward service/myapp-service 8080:80

# Delete resources
kubectl delete -f deployment.yaml
kubectl delete pod myapp-pod
kubectl delete deployment myapp-deployment

# Scale deployment
kubectl scale deployment myapp-deployment --replicas=5

# Update image
kubectl set image deployment/myapp-deployment myapp=myapp:2.0.0

# Rollout status
kubectl rollout status deployment/myapp-deployment

# Rollback
kubectl rollout undo deployment/myapp-deployment
```

---

## 8.3 Kubernetes Advanced

### Ingress - HTTP Routing

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
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
            name: myapp-admin-service
            port:
              number: 80
```

### Persistent Volume and Claims

```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast
  hostPath:
    path: /mnt/data/postgres

---
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast

---
# Using PVC in Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: postgres
        image: postgres:15
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

### StatefulSet - Stateful Applications

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
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
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
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
# Headless Service for StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None  # Headless
  selector:
    app: postgres
  ports:
  - port: 5432
    name: postgres
```

**StatefulSet Benefits:**
* Stable network identities (postgres-0, postgres-1, postgres-2)
* Ordered deployment and scaling
* Persistent storage per pod
* Ordered, graceful termination

### HorizontalPodAutoscaler

```yaml
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
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 2
        periodSeconds: 60
```

### Resource Quotas and Limits

```yaml
# namespace-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    persistentvolumeclaims: "10"
    services.loadbalancers: "2"

---
# limit-range.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: production
spec:
  limits:
  - max:
      cpu: "2"
      memory: "4Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
```

### Network Policies

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: myapp-network-policy
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
```

---

## 8.4 Helm - Kubernetes Package Manager

Helm simplifies Kubernetes application deployment and management.

### Helm Chart Structure

```
myapp/
├── Chart.yaml              # Chart metadata
├── values.yaml            # Default configuration
├── charts/                # Dependencies
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── _helpers.tpl       # Template helpers
│   └── NOTES.txt          # Post-install notes
└── README.md
```

### Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for MyApp
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - java
  - spring-boot
  - microservice
maintainers:
  - name: Developer Name
    email: dev@example.com
dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: "17.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

### values.yaml

```yaml
# Default values for myapp
replicaCount: 3

image:
  repository: myregistry.com/myapp
  pullPolicy: IfNotPresent
  tag: "1.0.0"

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
  prometheus.io/path: "/actuator/prometheus"

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: http
  initialDelaySeconds: 60
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: http
  initialDelaySeconds: 30
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

env:
  - name: SPRING_PROFILES_ACTIVE
    value: "prod"
  - name: JAVA_OPTS
    value: "-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

# PostgreSQL dependency
postgresql:
  enabled: true
  auth:
    username: myapp
    password: changeme
    database: myapp
  primary:
    persistence:
      size: 10Gi

# Redis dependency
redis:
  enabled: true
  auth:
    enabled: true
    password: changeme
  master:
    persistence:
      size: 1Gi
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "myapp.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
      - name: {{ .Chart.Name }}
        securityContext:
          {{- toYaml .Values.securityContext | nindent 12 }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        env:
        {{- toYaml .Values.env | nindent 8 }}
        livenessProbe:
          {{- toYaml .Values.livenessProbe | nindent 10 }}
        readinessProbe:
          {{- toYaml .Values.readinessProbe | nindent 10 }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

### Helm Commands

```bash
# Create new chart
helm create myapp

# Lint chart
helm lint myapp/

# Package chart
helm package myapp/

# Install chart
helm install myapp-release myapp/

# Install with custom values
helm install myapp-release myapp/ -f custom-values.yaml

# Install with inline values
helm install myapp-release myapp/ \
  --set image.tag=2.0.0 \
  --set replicaCount=5

# Upgrade release
helm upgrade myapp-release myapp/ \
  --set image.tag=2.0.0

# Rollback release
helm rollback myapp-release 1

# List releases
helm list

# Get release values
helm get values myapp-release

# Uninstall release
helm uninstall myapp-release

# Debug (dry-run)
helm install myapp-release myapp/ --dry-run --debug

# Template (render templates)
helm template myapp-release myapp/

# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Search charts
helm search repo postgresql

# Install from repository
helm install postgres bitnami/postgresql
```

---

## 8.5 Cloud Platforms

### AWS Deployment

**Elastic Kubernetes Service (EKS):**

```bash
# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster \
  --name myapp-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 5 \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name myapp-cluster

# Deploy application
kubectl apply -f kubernetes/

# Create load balancer
kubectl expose deployment myapp --type=LoadBalancer --port=80
```

**AWS Lambda (Serverless):**

```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-core</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-events</artifactId>
    <version>3.11.3</version>
</dependency>
```

```java
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;

public class MyLambdaHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    
    @Override
    public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent input, Context context) {
        context.getLogger().log("Processing request: " + input.getPath());
        
        APIGatewayProxyResponseEvent response = new APIGatewayProxyResponseEvent();
        response.setStatusCode(200);
        response.setBody("{\"message\": \"Hello from Lambda\"}");
        
        return response;
    }
}
```

**Deployment:**

```bash
# Package
mvn clean package

# Deploy with AWS SAM
sam deploy --guided

# Or with AWS CLI
aws lambda create-function \
  --function-name myapp-function \
  --runtime java21 \
  --handler com.example.MyLambdaHandler::handleRequest \
  --zip-file fileb://target/myapp-1.0.0.jar \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --memory-size 512 \
  --timeout 30
```

### Google Cloud Platform (GCP)

**Google Kubernetes Engine (GKE):**

```bash
# Install gcloud SDK
curl https://sdk.cloud.google.com | bash

# Initialize
gcloud init

# Create GKE cluster
gcloud container clusters create myapp-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-2 \
  --enable-autoscaling \
  --min-nodes 2 \
  --max-nodes 5

# Get credentials
gcloud container clusters get-credentials myapp-cluster --zone us-central1-a

# Deploy
kubectl apply -f kubernetes/
```

**Google Cloud Functions:**

```java
import com.google.cloud.functions.HttpFunction;
import com.google.cloud.functions.HttpRequest;
import com.google.cloud.functions.HttpResponse;
import java.io.BufferedWriter;

public class MyFunction implements HttpFunction {
    
    @Override
    public void service(HttpRequest request, HttpResponse response) throws Exception {
        BufferedWriter writer = response.getWriter();
        writer.write("{\"message\": \"Hello from Cloud Functions\"}");
    }
}
```

```bash
# Deploy
gcloud functions deploy myapp-function \
  --runtime java21 \
  --trigger-http \
  --allow-unauthenticated \
  --entry-point com.example.MyFunction \
  --memory 512MB \
  --timeout 60s
```

### Microsoft Azure

**Azure Kubernetes Service (AKS):**

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login

# Create resource group
az group create --name myapp-rg --location eastus

# Create AKS cluster
az aks create \
  --resource-group myapp-rg \
  --name myapp-cluster \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group myapp-rg --name myapp-cluster

# Deploy
kubectl apply -f kubernetes/
```

**Azure Functions:**

```java
import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

public class MyFunction {
    
    @FunctionName("myfunction")
    public HttpResponseMessage run(
        @HttpTrigger(
            name = "req",
            methods = {HttpMethod.GET, HttpMethod.POST},
            authLevel = AuthorizationLevel.ANONYMOUS
        ) HttpRequestMessage<Optional<String>> request,
        ExecutionContext context
    ) {
        context.getLogger().info("Processing request");
        
        return request.createResponseBuilder(HttpStatus.OK)
            .body("{\"message\": \"Hello from Azure Functions\"}")
            .build();
    }
}
```

---

## 8.6 Serverless Java Optimization

### Cold Start Problem

**Traditional Spring Boot Lambda:**

```
Cold Start: 8-12 seconds
Memory: 512MB
```

**Optimized Lambda:**

```xml
<!-- Use AWS Lambda Java runtime -->
<dependency>
    <groupId>com.amazonaws.serverless</groupId>
    <artifactId>aws-serverless-java-container-springboot3</artifactId>
    <version>2.0.0</version>
</dependency>
```

```java
import com.amazonaws.serverless.exceptions.ContainerInitializationException;
import com.amazonaws.serverless.proxy.model.AwsProxyRequest;
import com.amazonaws.serverless.proxy.model.AwsProxyResponse;
import com.amazonaws.serverless.proxy.spring.SpringBootLambdaContainerHandler;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestStreamHandler;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

public class StreamLambdaHandler implements RequestStreamHandler {
    
    private static SpringBootLambdaContainerHandler<AwsProxyRequest, AwsProxyResponse> handler;
    
    static {
        try {
            handler = SpringBootLambdaContainerHandler.getAwsProxyHandler(Application.class);
        } catch (ContainerInitializationException e) {
            e.printStackTrace();
            throw new RuntimeException("Could not initialize Spring Boot application", e);
        }
    }
    
    @Override
    public void handleRequest(InputStream inputStream, OutputStream outputStream, Context context)
            throws IOException {
        handler.proxyStream(inputStream, outputStream, context);
    }
}
```

**Optimization Techniques:**

```yaml
# AWS SAM template
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Runtime: java21
      Handler: com.example.StreamLambdaHandler::handleRequest
      MemorySize: 1024  # More memory = faster CPU
      Timeout: 30
      Environment:
        Variables:
          JAVA_TOOL_OPTIONS: -XX:+TieredCompilation -XX:TieredStopAtLevel=1  # Faster startup
      SnapStart:  # AWS SnapStart for faster cold starts
        ApplyOn: PublishedVersions
```

---

## 8.7 GraalVM Native Images

GraalVM Native Image compiles Java applications ahead-of-time into native executables.

### Benefits

**Traditional JVM:**
- Startup time: 3-10 seconds
- Memory: 100-200MB
- Peak performance: After warmup

**Native Image:**
- Startup time: **0.01-0.1 seconds** (100x faster!)
- Memory: **20-50MB** (4x less!)
- Peak performance: Immediately

### Spring Boot Native Setup

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.graalvm.buildtools</groupId>
            <artifactId>native-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**Build Native Image:**

```bash
# Install GraalVM
sdk install java 21.0.1-graal

# Build native image
./mvnw -Pnative native:compile

# Run native executable
./target/myapp

# Startup time: ~0.05 seconds!
```

### Reflection Configuration

```json
// reflect-config.json
[
  {
    "name": "com.example.User",
    "allDeclaredConstructors": true,
    "allPublicConstructors": true,
    "allDeclaredMethods": true,
    "allPublicMethods": true,
    "allDeclaredFields": true,
    "allPublicFields": true
  }
]
```

### Resource Configuration

```json
// resource-config.json
{
  "resources": {
    "includes": [
      {
        "pattern": "application.properties"
      },
      {
        "pattern": ".*\\.xml"
      }
    ]
  }
}
```

### Native Image Dockerfile

```dockerfile
# Stage 1: Build native image
FROM ghcr.io/graalvm/native-image:21 AS build
WORKDIR /app

# Copy source
COPY pom.xml .
COPY src ./src

# Build native image
RUN ./mvnw -Pnative native:compile

# Stage 2: Runtime
FROM gcr.io/distroless/base-debian12
WORKDIR /app

# Copy native executable
COPY --from=build /app/target/myapp ./myapp

# Run
ENTRYPOINT ["./myapp"]
```

**Result:**
- Image size: ~50MB (vs ~400MB traditional)
- Startup: ~0.05s (vs ~5s traditional)
- Memory: ~30MB (vs ~150MB traditional)

### Limitations

```java
// ❌ NOT SUPPORTED in Native Image
Class.forName("com.example.DynamicClass");  // Dynamic class loading
Method.invoke();  // Reflection (without configuration)
Proxy.newProxyInstance();  // Dynamic proxies

// ✅ SUPPORTED with configuration
// Add to reflect-config.json or use @RegisterReflectionForBinding
```

---

## 8.8 Service Mesh with Istio

Service mesh provides observability, traffic management, and security for microservices.

### Istio Installation

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -

# Install Istio
istioctl install --set profile=demo -y

# Enable automatic sidecar injection
kubectl label namespace default istio-injection=enabled

# Verify installation
kubectl get pods -n istio-system
```

### Deploy Application with Istio

```yaml
# deployment.yaml (same as before)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
```

**Apply deployment - Istio automatically injects sidecar:**

```bash
kubectl apply -f deployment.yaml

# Verify sidecar injected
kubectl get pods
# myapp-xxx-xxx   2/2   Running  # 2 containers: app + istio-proxy
```

### Virtual Service - Traffic Routing

```yaml
# virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.example.com
  http:
  # Canary deployment: 90% v1, 10% v2
  - match:
    - headers:
        user-agent:
          regex: ".*Mobile.*"
    route:
    - destination:
        host: myapp
        subset: v2
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10
```

### Destination Rule - Traffic Policy

```yaml
# destination-rule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
    loadBalancer:
      simple: LEAST_REQUEST
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

### Gateway - Ingress

```yaml
# gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: myapp-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - myapp.example.com
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: myapp-tls
    hosts:
    - myapp.example.com
```

### Observability

**Distributed Tracing:**

```bash
# Install Jaeger
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/jaeger.yaml

# Access Jaeger UI
istioctl dashboard jaeger
```

**Metrics:**

```bash
# Install Prometheus & Grafana
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/prometheus.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/grafana.yaml

# Access Grafana
istioctl dashboard grafana
```

**Service Graph:**

```bash
# Install Kiali
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml

# Access Kiali
istioctl dashboard kiali
```

### Mutual TLS

```yaml
# peer-authentication.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT  # Enforce mTLS for all services
```

---

## 8.9 Cloud-Native Patterns

### 12-Factor App Principles

**1. Codebase:** One codebase in version control

```bash
# Single Git repository
git clone https://github.com/myorg/myapp.git
```

**2. Dependencies:** Explicitly declare dependencies

```xml
<!-- pom.xml - all dependencies declared -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.2.1</version>
    </dependency>
</dependencies>
```

**3. Config:** Store config in environment

```java
@Configuration
public class AppConfig {
    
    @Value("${DATABASE_URL}")
    private String databaseUrl;
    
    @Value("${API_KEY}")
    private String apiKey;
}
```

**4. Backing Services:** Treat as attached resources

```yaml
# Database as attached resource
spring:
  datasource:
    url: ${DATABASE_URL}
```

**5. Build, Release, Run:** Strictly separate stages

```bash
# Build
mvn clean package

# Release (tag)
docker build -t myapp:1.0.0 .
docker push myapp:1.0.0

# Run (deploy)
kubectl apply -f deployment.yaml
```

**6. Processes:** Stateless processes

```java
// ✅ Stateless
@RestController
public class UserController {
    public User getUser(Long id) {
        return userService.findById(id);  // Fetch from DB
    }
}

// ❌ Stateful (don't do this)
public class UserController {
    private Map<Long, User> cache = new HashMap<>();  // In-memory state
}
```

**7. Port Binding:** Export via port binding

```yaml
# application.yml
server:
  port: ${PORT:8080}  # Read from environment
```

**8. Concurrency:** Scale via process model

```bash
# Scale horizontally
kubectl scale deployment myapp --replicas=10
```

**9. Disposability:** Fast startup and graceful shutdown

```java
@Component
public class GracefulShutdown {
    
    @PreDestroy
    public void onShutdown() {
        // Finish processing current requests
        // Close connections
        // Release resources
    }
}
```

```yaml
# Kubernetes graceful shutdown
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: myapp
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 15"]
```

**10. Dev/Prod Parity:** Keep environments similar

```yaml
# Same PostgreSQL version in all environments
services:
  db:
    image: postgres:15  # Exact version
```

**11. Logs:** Treat logs as event streams

```java
// Write to stdout
logger.info("Processing order: {}", orderId);

// Don't write to files
// FileAppender - NO
```

**12. Admin Processes:** Run as one-off processes

```bash
# Database migration
kubectl run db-migrate --image=myapp:1.0.0 --command -- npm run migrate
```

### Health Checks

**Spring Boot Actuator:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

**Endpoints:**
- `/actuator/health/liveness` - Is app alive?
- `/actuator/health/readiness` - Is app ready to serve traffic?

**Kubernetes Integration:**

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 60
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 5
```

### Graceful Shutdown

```java
@Configuration
public class GracefulShutdownConfig {
    
    @Bean
    public GracefulShutdown gracefulShutdown() {
        return new GracefulShutdown();
    }
}

@Component
class GracefulShutdown {
    
    private static final Logger logger = LoggerFactory.getLogger(GracefulShutdown.class);
    
    @EventListener(ContextClosedEvent.class)
    public void onShutdown() {
        logger.info("Shutdown signal received");
        
        // 1. Stop accepting new requests
        // 2. Finish processing current requests
        // 3. Close database connections
        // 4. Flush logs
        
        logger.info("Graceful shutdown complete");
    }
}
```

---

## Summary and Key Takeaways

### Container Mastery

**Docker Best Practices:**
* **Multi-stage builds**: Smaller images, faster builds
* **Layer optimization**: Dependencies cached separately
* **Non-root user**: Security best practice
* **Health checks**: Container health monitoring
* **JVM optimization**: Container-aware settings

**Image Sizes:**
* Traditional: ~400MB
* Optimized: ~150MB
* Native (GraalVM): ~50MB

### Kubernetes Expertise

**Core Concepts:**
* **Pods**: Basic deployment unit
* **Deployments**: Managed replicas with rolling updates
* **Services**: Network access (ClusterIP, NodePort, LoadBalancer)
* **ConfigMaps/Secrets**: Configuration management
* **Ingress**: HTTP routing

**Advanced Features:**
* **StatefulSets**: For stateful applications (databases)
* **HPA**: Auto-scaling based on CPU/memory
* **Network Policies**: Traffic control
* **Resource Quotas**: Namespace limits

### Helm Power

**Benefits:**
* **Templating**: Reusable configurations
* **Versioning**: Track releases
* **Dependencies**: Manage sub-charts
* **Values**: Environment-specific config

**Structure:**
* Chart.yaml: Metadata
* values.yaml: Default config
* templates/: Kubernetes manifests

### Cloud Platform Strategies

**AWS:**
* EKS for Kubernetes
* Lambda for serverless
* ECR for container registry

**GCP:**
* GKE for Kubernetes
* Cloud Functions for serverless
* GCR for container registry

**Azure:**
* AKS for Kubernetes
* Functions for serverless
* ACR for container registry

### GraalVM Revolution

**Performance:**
* Startup: **100x faster** (5s → 0.05s)
* Memory: **4x less** (150MB → 30MB)
* Image size: **8x smaller** (400MB → 50MB)

**Perfect for:**
* Serverless (cold start critical)
* CLI tools (instant startup)
* Microservices (resource efficiency)

**Trade-offs:**
* Build time: Longer
* Reflection: Requires configuration
* Dynamic features: Limited

### Service Mesh Benefits

**Istio Capabilities:**
* **Traffic management**: Canary, A/B testing
* **Security**: Mutual TLS
* **Observability**: Distributed tracing
* **Resilience**: Circuit breaking, retries

**When to Use:**
* Many microservices (>10)
* Need advanced traffic control
* Security requirements (mTLS)
* Observability critical

### Production Checklist

**Containerization:**
- [ ] Multi-stage Dockerfile
- [ ] Non-root user
- [ ] Health checks
- [ ] Resource limits
- [ ] Security scanning

**Kubernetes:**
- [ ] Liveness/readiness probes
- [ ] Resource requests/limits
- [ ] HPA configured
- [ ] ConfigMaps/Secrets for config
- [ ] Network policies

**Observability:**
- [ ] Distributed tracing (Jaeger)
- [ ] Metrics (Prometheus)
- [ ] Logging (ELK/Loki)
- [ ] Dashboards (Grafana)
- [ ] Alerts configured

**Security:**
- [ ] Images scanned for vulnerabilities
- [ ] Secrets encrypted
- [ ] RBAC configured
- [ ] Network policies
- [ ] mTLS enabled (if using service mesh)

### Frequently Asked Questions

**Q1: When should I use Kubernetes vs simple Docker Compose?**

**A:**

**Use Kubernetes when:**
* Production environment
* Need high availability
* Auto-scaling required
* Multiple services (microservices)
* Team size: 5+ developers

**Use Docker Compose when:**
* Local development
* Small applications (<5 services)
* Team size: 1-5 developers
* Simple deployment needs

**Migration path:**
1. Start with Docker Compose (local dev)
2. Move to Kubernetes (production)
3. Use Helm for Kubernetes management

---

**Q2: Should I use GraalVM Native Image?**

**A:**

**Yes, when:**
* Serverless applications (Lambda)
* CLI tools
* Startup time critical
* Memory constrained

**No, when:**
* Heavy use of reflection
* Dynamic class loading required
* Peak throughput > startup time
* Build time is critical

**Comparison:**

| Metric | JVM | Native Image |
|--------|-----|--------------|
| Startup | 5s | 0.05s |
| Memory | 150MB | 30MB |
| Peak Performance | Faster (after warmup) | Slower |
| Build Time | Fast | Slow |
| Reflection | Full support | Limited |

---

**Q3: How do I choose between AWS, GCP, and Azure?**

**A:**

**AWS:**
* Largest market share
* Most services
* Best for: General purpose, enterprise

**GCP:**
* Best Kubernetes (GKE)
* Best AI/ML tools
* Best for: Data-heavy, Kubernetes-first

**Azure:**
* Best Microsoft integration
* Best for: .NET + Java hybrid, enterprise Microsoft shops

**Multi-cloud strategy:**
* Use managed Kubernetes (EKS/GKE/AKS)
* Deploy same Helm charts everywhere
* Abstract cloud-specific services

### Interview Questions

**Question 1: Explain the difference between liveness and readiness probes.**

**Difficulty:** Mid-Level

**Topics:** Kubernetes, Cloud-Native

**Answer:**

**Liveness Probe**: "Is the application alive?"
* Determines if container should be restarted
* Fails → Kubernetes restarts container
* Example: Deadlock detection

**Readiness Probe**: "Is the application ready to serve traffic?"
* Determines if traffic should be sent
* Fails → Removed from service endpoints (no restart)
* Example: Waiting for database connection

**Configuration:**
```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 60  # Wait 60s before first check
  periodSeconds: 10        # Check every 10s
  failureThreshold: 3      # Restart after 3 failures

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 5
  failureThreshold: 3      # Remove from service after 3 failures
```

**Best Practices:**
* Different endpoints for liveness vs readiness
* Conservative liveness (avoid restart loops)
* Aggressive readiness (remove slow pods quickly)

---

**Question 2: How does GraalVM Native Image achieve fast startup?**

**Difficulty:** Senior

**Topics:** GraalVM, Performance

**Answer:**

**Traditional JVM Startup:**
1. Load JVM
2. Parse bytecode
3. JIT compile hot methods
4. Initialize classes
5. **Total: 5-10 seconds**

**Native Image:**
1. Ahead-of-time (AOT) compilation
2. Closed-world analysis
3. Initialize at build time
4. Directly executable machine code
5. **Total: 0.05 seconds**

**Key Optimizations:**
```
Build Time:
- Analyze entire application
- Compile to native code
- Initialize static fields
- Resolve classes
- Remove unused code

Runtime:
- No JVM startup
- No class loading
- No JIT compilation
- Direct execution
```

**Trade-offs:**
* Faster startup, slower peak performance
* Smaller binary, longer build
* Less dynamic, more predictable

---

**End of Part 8: Cloud-Native Java**

### Congratulations!

You've mastered:
* ✅ Docker containerization (optimized images)
* ✅ Kubernetes orchestration (deployments, services, scaling)
* ✅ Helm package management
* ✅ Cloud platform deployment (AWS, GCP, Azure)
* ✅ Serverless Java (Lambda, Cloud Functions)
* ✅ GraalVM native images
* ✅ Service mesh (Istio)
* ✅ Cloud-native patterns (12-factor app)

**Complete Series: 8 parts complete! 🎉**

### References

* [Kubernetes Documentation](https://kubernetes.io/docs/)
* [Docker Documentation](https://docs.docker.com/)
* [Helm Documentation](https://helm.sh/docs/)
* [GraalVM Documentation](https://www.graalvm.org/latest/docs/)
* [Istio Documentation](https://istio.io/latest/docs/)
* [12-Factor App](https://12factor.net/)
* [Spring Boot Native Image](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html)

---