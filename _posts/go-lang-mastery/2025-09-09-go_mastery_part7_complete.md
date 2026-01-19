---
title: "Go Mastery - Part 7: Deployment, DevOps & Cloud Native"
date: 2025-09-09 00:00:00 +0530
categories: [Go-lang, Go Mastery]
tags: [Go-lang, DevOps, Deployment, DevOps, Docker, Kubernetes, CI-CD, Cloud-native, Containers]
---

# Complete Go Mastery Part 7: Deployment, DevOps & Cloud Native

## Introduction

Welcome to Part 7 of the Complete Go Mastery series. In Parts 1-6, we covered Go fundamentals, concurrency, advanced techniques, testing, architecture, and observability. Now we explore **deployment and DevOps**—getting your Go applications into production and keeping them running reliably.

Building great software is only half the battle. The other half is deploying it safely, efficiently, and reliably. Modern DevOps practices and cloud-native technologies have transformed how we ship and run applications. Go's excellent tooling and cloud-native design make it perfect for containerized, microservices-based deployments.

**Why DevOps matters for Go developers?**

- **Faster releases**: Ship features and fixes quickly
- **Reliability**: Deploy without fear through automation
- **Scalability**: Handle growing traffic seamlessly
- **Cost efficiency**: Optimize resource usage
- **Developer productivity**: Focus on code, not infrastructure

**What you'll learn in Part 7:**

- **Containerization**: Building optimal Docker images
- **Kubernetes Deployment**: Running Go apps on K8s
- **CI/CD Pipelines**: Automated testing and deployment
- **Cloud Platforms**: AWS, GCP, Azure deployment
- **Configuration Management**: Environment-specific configs
- **Service Mesh**: Istio, Linkerd integration
- **Blue-Green & Canary Deployments**: Safe rollouts
- **Scaling Strategies**: Horizontal and vertical scaling
- **Secrets Management**: Secure credential handling
- **Infrastructure as Code**: Terraform, Pulumi

By the end of Part 7, you'll be able to deploy production-ready Go applications to any cloud platform with confidence.

---

## 7.1 Containerizing Go Applications

### Why Docker for Go?

**Benefits:**
- **Consistent environments**: Same everywhere
- **Isolation**: Dependencies contained
- **Portability**: Run anywhere
- **Efficiency**: Small, fast images

### Multi-Stage Docker Builds

The optimal pattern for Go applications:

```dockerfile
# Dockerfile
# Stage 1: Build
FROM golang:1.24-alpine AS builder

# Install build dependencies
RUN apk add --no-cache git make

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a \
    -o /app/myapp \
    ./cmd/api

# Stage 2: Runtime
FROM scratch

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy binary
COPY --from=builder /app/myapp /myapp

# Copy any static files if needed
# COPY --from=builder /app/static /static

# Run as non-root user
USER 1000:1000

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/myapp", "healthcheck"] || exit 1

# Run application
ENTRYPOINT ["/myapp"]
```

**Why this pattern?**
- **Small images**: Final image ~10-20MB (vs 800MB+ with full Go image)
- **Security**: No build tools in runtime image
- **Fast deployments**: Smaller images deploy faster
- **Secure**: Scratch image has minimal attack surface

### Optimized Dockerfile Patterns

**With Alpine for runtime (if you need shell/debugging):**

```dockerfile
# Stage 1: Build
FROM golang:1.24-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 go build -ldflags='-w -s' -o myapp ./cmd/api

# Stage 2: Runtime
FROM alpine:3.19

# Install ca-certificates and timezone data
RUN apk --no-cache add ca-certificates tzdata

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /home/appuser

# Copy binary
COPY --from=builder /app/myapp .

# Change ownership
RUN chown -R appuser:appuser /home/appuser

USER appuser

EXPOSE 8080

CMD ["./myapp"]
```

**With distroless (Google's minimal images):**

```dockerfile
# Stage 1: Build
FROM golang:1.24 AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 go build -ldflags='-w -s' -o myapp ./cmd/api

# Stage 2: Runtime
FROM gcr.io/distroless/static-debian12:nonroot

COPY --from=builder /app/myapp /myapp

EXPOSE 8080

USER nonroot:nonroot

ENTRYPOINT ["/myapp"]
```

### Docker Compose for Local Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/myapp?sslmode=disable
      - REDIS_URL=redis:6379
      - ENVIRONMENT=development
    volumes:
      - .:/app
    depends_on:
      - db
      - redis
    command: ["go", "run", "./cmd/api"]
    networks:
      - app-network

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

  migrate:
    image: migrate/migrate
    volumes:
      - ./migrations:/migrations
    command:
      [
        "-path",
        "/migrations",
        "-database",
        "postgres://postgres:postgres@db:5432/myapp?sslmode=disable",
        "up"
      ]
    depends_on:
      - db
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

**Development Dockerfile:**

```dockerfile
# Dockerfile.dev
FROM golang:1.24-alpine

RUN apk add --no-cache git

# Install hot reload tool
RUN go install github.com/cosmtrek/air@latest

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

CMD ["air", "-c", ".air.toml"]
```

**Air configuration for hot reload:**

```toml
# .air.toml
root = "."
testdata_dir = "testdata"
tmp_dir = "tmp"

[build]
  args_bin = []
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ./cmd/api"
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  include_file = []
  kill_delay = "0s"
  log = "build-errors.log"
  poll = false
  poll_interval = 0
  rerun = false
  rerun_delay = 500
  send_interrupt = false
  stop_on_error = false

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  main_only = false
  time = false

[misc]
  clean_on_exit = false

[screen]
  clear_on_rebuild = false
  keep_scroll = true
```

### Build Arguments and Secrets

```dockerfile
# Dockerfile with build args
FROM golang:1.24-alpine AS builder

# Build arguments
ARG VERSION=dev
ARG BUILD_DATE
ARG COMMIT_SHA

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Inject version information
RUN CGO_ENABLED=0 go build \
    -ldflags="-w -s \
    -X main.Version=${VERSION} \
    -X main.BuildDate=${BUILD_DATE} \
    -X main.CommitSHA=${COMMIT_SHA}" \
    -o myapp ./cmd/api

FROM scratch
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
```

**Building with arguments:**

```bash
docker build \
  --build-arg VERSION=v1.2.3 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg COMMIT_SHA=$(git rev-parse HEAD) \
  -t myapp:v1.2.3 \
  .
```

**Using build secrets (Docker BuildKit):**

```dockerfile
# syntax=docker/dockerfile:1

FROM golang:1.24-alpine AS builder

WORKDIR /app

# Mount secret for private repos
RUN --mount=type=secret,id=github_token \
    git config --global url."https://$(cat /run/secrets/github_token)@github.com/".insteadOf "https://github.com/"

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 go build -o myapp ./cmd/api

FROM scratch
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
```

```bash
# Build with secret
DOCKER_BUILDKIT=1 docker build \
  --secret id=github_token,src=$HOME/.github-token \
  -t myapp:latest \
  .
```

### Image Size Optimization

**Comparison:**

```
golang:1.24              ~800MB
golang:1.24-alpine       ~300MB
alpine + binary          ~15MB
distroless + binary      ~12MB
scratch + binary         ~10MB
```

**Best practices:**

```dockerfile
# 1. Use specific versions
FROM golang:1.24-alpine AS builder  # ✅ Good
# FROM golang:latest AS builder     # ❌ Bad

# 2. Minimize layers
RUN apk add --no-cache git make     # ✅ Good
# RUN apk add git                   # ❌ Bad
# RUN apk add make                  # ❌ Bad

# 3. Order layers by change frequency
COPY go.mod go.sum ./              # ✅ Changes rarely
RUN go mod download
COPY . .                           # ✅ Changes often

# 4. Use .dockerignore
# See .dockerignore example below

# 5. Remove build artifacts
RUN go build -o myapp && \
    rm -rf /root/.cache/go-build   # ✅ Clean up
```

**.dockerignore:**

```
# .dockerignore
.git
.gitignore
.env
.env.*
*.md
LICENSE

# Build artifacts
tmp/
dist/
bin/
*.log

# Test files
*_test.go
testdata/
coverage.out

# Docker files
Dockerfile*
docker-compose*.yml
.dockerignore

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# Documentation
docs/
```

### Security Scanning

```bash
# Scan image for vulnerabilities
docker scan myapp:latest

# Using Trivy
trivy image myapp:latest

# Using Grype
grype myapp:latest

# Using Snyk
snyk container test myapp:latest
```

---

## 7.2 Kubernetes Deployment

### Basic Kubernetes Resources

**Deployment:**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
    version: v1.0.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1.0.0
    spec:
      serviceAccountName: myapp
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
      - name: myapp
        image: myregistry.io/myapp:v1.0.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        - name: metrics
          containerPort: 9090
          protocol: TCP
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: redis-url
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health/live
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /health/startup
            port: http
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 30
        volumeMounts:
        - name: config
          mountPath: /etc/myapp
          readOnly: true
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: config
        configMap:
          name: myapp-config
      - name: tmp
        emptyDir: {}
```

**Service:**

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: metrics
    protocol: TCP
```

**Ingress:**

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.example.com
    secretName: myapp-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              name: http
```

**ConfigMap:**

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production
data:
  redis-url: "redis://redis-master:6379"
  log-level: "info"
  app.yaml: |
    server:
      port: 8080
      timeout: 30s
    cache:
      ttl: 300
```

**Secret:**

```yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: production
type: Opaque
stringData:
  database-url: "postgres://user:pass@postgres:5432/myapp"
  jwt-secret: "your-secret-key-here"
```

**ServiceAccount:**

```yaml
# k8s/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp
  namespace: production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: myapp
  namespace: production
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: myapp
  namespace: production
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: myapp
subjects:
- kind: ServiceAccount
  name: myapp
  namespace: production
```

Continuing with Part 7...
### HorizontalPodAutoscaler

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
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
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

### PodDisruptionBudget

```yaml
# k8s/pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp
  namespace: production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

### Kustomize for Multi-Environment

```yaml
# k8s/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml
- configmap.yaml
- serviceaccount.yaml

commonLabels:
  app: myapp
  managed-by: kustomize
```

```yaml
# k8s/overlays/staging/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

namespace: staging

replicas:
- name: myapp
  count: 2

images:
- name: myapp
  newTag: staging-latest

configMapGenerator:
- name: myapp-config
  behavior: merge
  literals:
  - log-level=debug
```

```yaml
# k8s/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

namespace: production

replicas:
- name: myapp
  count: 3

images:
- name: myapp
  newTag: v1.0.0

resources:
- hpa.yaml
- pdb.yaml
- ingress.yaml

configMapGenerator:
- name: myapp-config
  behavior: merge
  literals:
  - log-level=info
```

**Deploy with Kustomize:**

```bash
# Staging
kubectl apply -k k8s/overlays/staging

# Production
kubectl apply -k k8s/overlays/production
```

---

## 7.3 CI/CD Pipelines

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  GO_VERSION: '1.24'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up Go
      uses: actions/setup-go@v5
      with:
        go-version: ${{ env.GO_VERSION }}
        cache: true
    
    - name: Install dependencies
      run: go mod download
    
    - name: Run tests
      run: go test -v -race -coverprofile=coverage.out ./...
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage.out
    
    - name: Run linters
      uses: golangci/golangci-lint-action@v3
      with:
        version: latest
        args: --timeout=5m

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Run Gosec Security Scanner
      uses: securego/gosec@master
      with:
        args: '-no-fail -fmt sarif -out results.sarif ./...'
    
    - name: Upload SARIF file
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: results.sarif

  build:
    name: Build and Push
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    
    permissions:
      contents: read
      packages: write
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha,prefix={{branch}}-
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        build-args: |
          VERSION=${{ steps.meta.outputs.version }}
          BUILD_DATE=${{ steps.meta.outputs.created }}
          COMMIT_SHA=${{ github.sha }}

  deploy-staging:
    name: Deploy to Staging
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > kubeconfig
        echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV
    
    - name: Deploy to staging
      run: |
        kubectl apply -k k8s/overlays/staging
        kubectl rollout status deployment/myapp -n staging

  deploy-production:
    name: Deploy to Production
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBE_CONFIG_PRODUCTION }}" | base64 -d > kubeconfig
        echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV
    
    - name: Deploy to production
      run: |
        kubectl apply -k k8s/overlays/production
        kubectl rollout status deployment/myapp -n production
    
    - name: Verify deployment
      run: |
        kubectl get pods -n production -l app=myapp
        kubectl get ingress -n production myapp
```

### GitLab CI/CD Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  GO_VERSION: "1.24"
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

.go-cache:
  cache:
    paths:
      - .go/pkg/mod/

test:
  stage: test
  image: golang:${GO_VERSION}
  extends: .go-cache
  script:
    - go mod download
    - go test -v -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.out

lint:
  stage: test
  image: golangci/golangci-lint:latest
  extends: .go-cache
  script:
    - golangci-lint run -v

security-scan:
  stage: test
  image: securego/gosec:latest
  script:
    - gosec -fmt json -out gosec-report.json ./...
  artifacts:
    reports:
      sast: gosec-report.json

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - |
      docker build \
        --build-arg VERSION=$CI_COMMIT_TAG \
        --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
        --build-arg COMMIT_SHA=$CI_COMMIT_SHA \
        -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA \
        -t $CI_REGISTRY_IMAGE:latest \
        .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/myapp -n staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context production
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/myapp -n production
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### CircleCI Configuration

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  go: circleci/go@1.9
  docker: circleci/docker@2.4
  kubernetes: circleci/kubernetes@1.3

executors:
  go-executor:
    docker:
      - image: cimg/go:1.24

jobs:
  test:
    executor: go-executor
    steps:
      - checkout
      - go/load-cache
      - go/mod-download
      - go/save-cache
      - run:
          name: Run tests
          command: |
            go test -v -race -coverprofile=coverage.out ./...
            go tool cover -html=coverage.out -o coverage.html
      - store_artifacts:
          path: coverage.html
      - store_test_results:
          path: /tmp/test-results

  lint:
    executor: go-executor
    steps:
      - checkout
      - run:
          name: Install golangci-lint
          command: |
            curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin
      - run:
          name: Run linters
          command: golangci-lint run

  build-and-push:
    executor: docker/docker
    steps:
      - checkout
      - setup_remote_docker
      - docker/check
      - docker/build:
          image: myapp
          tag: $CIRCLE_SHA1,latest
      - docker/push:
          image: myapp
          tag: $CIRCLE_SHA1,latest

  deploy-staging:
    executor: kubernetes/default
    steps:
      - checkout
      - kubernetes/install-kubectl
      - kubernetes/install-kubeconfig:
          kubeconfig: KUBECONFIG_STAGING
      - run:
          name: Deploy to staging
          command: |
            kubectl apply -k k8s/overlays/staging
            kubectl rollout status deployment/myapp -n staging

workflows:
  version: 2
  build-test-deploy:
    jobs:
      - test
      - lint
      - build-and-push:
          requires:
            - test
            - lint
          filters:
            branches:
              only:
                - main
                - develop
      - deploy-staging:
          requires:
            - build-and-push
          filters:
            branches:
              only: develop
```

---

## 7.4 Cloud Platform Deployment

### AWS Deployment

**ECS (Elastic Container Service):**

```json
// ecs-task-definition.json
{
  "family": "myapp",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789:role/myappTaskRole",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest",
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:myapp/database-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/myapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

**ECS Service:**

```json
// ecs-service.json
{
  "serviceName": "myapp",
  "taskDefinition": "myapp:1",
  "desiredCount": 3,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-12345678",
        "subnet-87654321"
      ],
      "securityGroups": [
        "sg-12345678"
      ],
      "assignPublicIp": "DISABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:123456789:targetgroup/myapp/1234567890",
      "containerName": "myapp",
      "containerPort": 8080
    }
  ],
  "healthCheckGracePeriodSeconds": 60,
  "deploymentConfiguration": {
    "maximumPercent": 200,
    "minimumHealthyPercent": 100
  }
}
```

**Deploy to ECS:**

```bash
# Build and push to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

docker build -t myapp:latest .
docker tag myapp:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Register task definition
aws ecs register-task-definition --cli-input-json file://ecs-task-definition.json

# Update service
aws ecs update-service \
  --cluster production \
  --service myapp \
  --task-definition myapp:2 \
  --force-new-deployment
```

**EKS (Elastic Kubernetes Service):**

```bash
# Create EKS cluster
eksctl create cluster \
  --name production \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 5 \
  --managed

# Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name production

# Deploy application
kubectl apply -k k8s/overlays/production
```

Continuing with Part 7...


### GCP Deployment

**Cloud Run (serverless containers):**

```yaml
# cloudrun.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "10"
    spec:
      containerConcurrency: 80
      containers:
      - image: gcr.io/my-project/myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: ENVIRONMENT
          value: "production"
        resources:
          limits:
            cpu: "1000m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
          initialDelaySeconds: 5
          periodSeconds: 10
```

```bash
# Deploy to Cloud Run
gcloud run deploy myapp \
  --image gcr.io/my-project/myapp:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --min-instances 1 \
  --max-instances 10 \
  --cpu 1 \
  --memory 512Mi \
  --timeout 300 \
  --concurrency 80
```

**GKE (Google Kubernetes Engine):**

```bash
# Create GKE cluster
gcloud container clusters create production \
  --region us-central1 \
  --num-nodes 3 \
  --machine-type n1-standard-2 \
  --enable-autoscaling \
  --min-nodes 2 \
  --max-nodes 10 \
  --enable-autorepair \
  --enable-autoupgrade

# Get credentials
gcloud container clusters get-credentials production --region us-central1

# Deploy
kubectl apply -k k8s/overlays/production
```

### Azure Deployment

**Azure Container Instances:**

```bash
# Deploy to ACI
az container create \
  --resource-group myapp-rg \
  --name myapp \
  --image myregistry.azurecr.io/myapp:latest \
  --cpu 1 \
  --memory 1 \
  --port 8080 \
  --environment-variables \
    ENVIRONMENT=production \
  --secure-environment-variables \
    DATABASE_URL=$DATABASE_URL \
  --restart-policy Always
```

**AKS (Azure Kubernetes Service):**

```bash
# Create AKS cluster
az aks create \
  --resource-group myapp-rg \
  --name production \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 10 \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group myapp-rg --name production

# Deploy
kubectl apply -k k8s/overlays/production
```

---

## 7.5 Advanced Deployment Strategies

### Blue-Green Deployment

```yaml
# k8s/blue-green/deployment-blue.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0.0
        ports:
        - containerPort: 8080
```

```yaml
# k8s/blue-green/deployment-green.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        ports:
        - containerPort: 8080
```

```yaml
# k8s/blue-green/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: blue  # Switch to green when ready
  ports:
  - port: 80
    targetPort: 8080
```

**Blue-Green deployment script:**

```bash
#!/bin/bash
# blue-green-deploy.sh

# Deploy green version
kubectl apply -f k8s/blue-green/deployment-green.yaml

# Wait for green to be ready
kubectl rollout status deployment/myapp-green

# Run smoke tests
./smoke-tests.sh myapp-green

# Switch traffic to green
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Monitor for issues
sleep 300

# If all good, scale down blue
kubectl scale deployment/myapp-blue --replicas=0

# Keep blue around for rollback if needed
```

### Canary Deployment

```yaml
# k8s/canary/deployment-stable.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9  # 90% of traffic
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
        version: v1.0.0
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0.0
```

```yaml
# k8s/canary/deployment-canary.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1  # 10% of traffic
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
        version: v2.0.0
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
```

**Progressive canary with Flagger:**

```yaml
# k8s/canary/flagger-canary.yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  service:
    port: 8080
  analysis:
    interval: 1m
    threshold: 10
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 1m
    webhooks:
    - name: smoke-test
      url: http://flagger-loadtester/
      timeout: 5s
      metadata:
        type: bash
        cmd: "curl -s http://myapp-canary:8080/health | grep OK"
```

### Rolling Deployment with Health Checks

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3          # 3 extra pods during update
      maxUnavailable: 1    # At most 1 pod down
  minReadySeconds: 10      # Wait 10s after pod ready
  progressDeadlineSeconds: 600  # Fail after 10 minutes
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 2  # Must pass twice
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
```

---

## 7.6 Secrets Management

### Kubernetes Secrets

```yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
stringData:
  database-password: "my-secret-password"
  api-key: "my-api-key"
```

**Using Sealed Secrets:**

```bash
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Encrypt secret
echo -n "my-secret-password" | kubectl create secret generic myapp-secrets \
  --dry-run=client \
  --from-file=database-password=/dev/stdin \
  -o yaml | \
  kubeseal -o yaml > myapp-sealed-secret.yaml

# Apply sealed secret (safe to commit)
kubectl apply -f myapp-sealed-secret.yaml
```

### External Secrets Operator

```yaml
# k8s/external-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secretsmanager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: myapp-secrets
    creationPolicy: Owner
  data:
  - secretKey: database-password
    remoteRef:
      key: myapp/database
      property: password
  - secretKey: api-key
    remoteRef:
      key: myapp/api
      property: key
```

### HashiCorp Vault Integration

```go
// pkg/vault/vault.go
package vault

import (
    "context"
    "fmt"
    
    vault "github.com/hashicorp/vault/api"
)

type Client struct {
    client *vault.Client
}

func NewClient(address, token string) (*Client, error) {
    config := vault.DefaultConfig()
    config.Address = address
    
    client, err := vault.NewClient(config)
    if err != nil {
        return nil, err
    }
    
    client.SetToken(token)
    
    return &Client{client: client}, nil
}

func (c *Client) GetSecret(ctx context.Context, path string) (map[string]interface{}, error) {
    secret, err := c.client.Logical().ReadWithContext(ctx, path)
    if err != nil {
        return nil, err
    }
    
    if secret == nil {
        return nil, fmt.Errorf("secret not found at %s", path)
    }
    
    return secret.Data, nil
}

func (c *Client) GetDatabaseCredentials(ctx context.Context) (string, string, error) {
    secret, err := c.GetSecret(ctx, "database/creds/myapp")
    if err != nil {
        return "", "", err
    }
    
    username := secret["username"].(string)
    password := secret["password"].(string)
    
    return username, password, nil
}
```

**Usage in application:**

```go
// cmd/api/main.go
func main() {
    vaultClient, err := vault.NewClient(
        os.Getenv("VAULT_ADDR"),
        os.Getenv("VAULT_TOKEN"),
    )
    if err != nil {
        log.Fatal(err)
    }
    
    username, password, err := vaultClient.GetDatabaseCredentials(context.Background())
    if err != nil {
        log.Fatal(err)
    }
    
    dsn := fmt.Sprintf("postgres://%s:%s@localhost/myapp", username, password)
    db, err := sql.Open("postgres", dsn)
    // ...
}
```

---

## 7.7 Infrastructure as Code

### Terraform for Kubernetes

```hcl
# terraform/main.tf
terraform {
  required_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
  }
}

provider "kubernetes" {
  config_path = "~/.kube/config"
}

resource "kubernetes_namespace" "production" {
  metadata {
    name = "production"
    labels = {
      environment = "production"
      managed-by  = "terraform"
    }
  }
}

resource "kubernetes_deployment" "myapp" {
  metadata {
    name      = "myapp"
    namespace = kubernetes_namespace.production.metadata[0].name
    labels = {
      app = "myapp"
    }
  }

  spec {
    replicas = var.replicas

    selector {
      match_labels = {
        app = "myapp"
      }
    }

    template {
      metadata {
        labels = {
          app     = "myapp"
          version = var.version
        }
      }

      spec {
        container {
          name  = "myapp"
          image = "${var.image_registry}/myapp:${var.version}"

          port {
            container_port = 8080
            name          = "http"
          }

          resources {
            requests = {
              cpu    = "100m"
              memory = "128Mi"
            }
            limits = {
              cpu    = "500m"
              memory = "512Mi"
            }
          }

          liveness_probe {
            http_get {
              path = "/health/live"
              port = "http"
            }
            initial_delay_seconds = 10
            period_seconds       = 10
          }

          readiness_probe {
            http_get {
              path = "/health/ready"
              port = "http"
            }
            initial_delay_seconds = 5
            period_seconds       = 5
          }

          env {
            name  = "ENVIRONMENT"
            value = "production"
          }

          env {
            name = "DATABASE_URL"
            value_from {
              secret_key_ref {
                name = kubernetes_secret.myapp.metadata[0].name
                key  = "database-url"
              }
            }
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "myapp" {
  metadata {
    name      = "myapp"
    namespace = kubernetes_namespace.production.metadata[0].name
  }

  spec {
    selector = {
      app = "myapp"
    }

    port {
      port        = 80
      target_port = "http"
    }

    type = "ClusterIP"
  }
}

resource "kubernetes_secret" "myapp" {
  metadata {
    name      = "myapp-secrets"
    namespace = kubernetes_namespace.production.metadata[0].name
  }

  data = {
    database-url = var.database_url
  }
}

resource "kubernetes_horizontal_pod_autoscaler" "myapp" {
  metadata {
    name      = "myapp"
    namespace = kubernetes_namespace.production.metadata[0].name
  }

  spec {
    max_replicas = 10
    min_replicas = 3

    scale_target_ref {
      api_version = "apps/v1"
      kind        = "Deployment"
      name        = kubernetes_deployment.myapp.metadata[0].name
    }

    metric {
      type = "Resource"
      resource {
        name = "cpu"
        target {
          type                = "Utilization"
          average_utilization = 70
        }
      }
    }
  }
}
```

```hcl
# terraform/variables.tf
variable "replicas" {
  description = "Number of replicas"
  type        = number
  default     = 3
}

variable "version" {
  description = "Application version"
  type        = string
}

variable "image_registry" {
  description = "Container registry"
  type        = string
}

variable "database_url" {
  description = "Database connection string"
  type        = string
  sensitive   = true
}
```

```hcl
# terraform/outputs.tf
output "service_endpoint" {
  value = kubernetes_service.myapp.status[0].load_balancer[0].ingress[0].ip
}

output "namespace" {
  value = kubernetes_namespace.production.metadata[0].name
}
```

**Deploy with Terraform:**

```bash
# Initialize
terraform init

# Plan
terraform plan -out=tfplan

# Apply
terraform apply tfplan

# Destroy
terraform destroy
```

### Pulumi for Infrastructure

```go
// main.go
package main

import (
    "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes"
    appsv1 "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes/apps/v1"
    corev1 "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes/core/v1"
    metav1 "github.com/pulumi/pulumi-kubernetes/sdk/v4/go/kubernetes/meta/v1"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        // Create namespace
        namespace, err := corev1.NewNamespace(ctx, "production", &corev1.NamespaceArgs{
            Metadata: &metav1.ObjectMetaArgs{
                Name: pulumi.String("production"),
            },
        })
        if err != nil {
            return err
        }

        // Create deployment
        deployment, err := appsv1.NewDeployment(ctx, "myapp", &appsv1.DeploymentArgs{
            Metadata: &metav1.ObjectMetaArgs{
                Namespace: namespace.Metadata.Name(),
                Name:      pulumi.String("myapp"),
            },
            Spec: &appsv1.DeploymentSpecArgs{
                Replicas: pulumi.Int(3),
                Selector: &metav1.LabelSelectorArgs{
                    MatchLabels: pulumi.StringMap{
                        "app": pulumi.String("myapp"),
                    },
                },
                Template: &corev1.PodTemplateSpecArgs{
                    Metadata: &metav1.ObjectMetaArgs{
                        Labels: pulumi.StringMap{
                            "app": pulumi.String("myapp"),
                        },
                    },
                    Spec: &corev1.PodSpecArgs{
                        Containers: corev1.ContainerArray{
                            &corev1.ContainerArgs{
                                Name:  pulumi.String("myapp"),
                                Image: pulumi.String("myapp:latest"),
                                Ports: corev1.ContainerPortArray{
                                    &corev1.ContainerPortArgs{
                                        ContainerPort: pulumi.Int(8080),
                                    },
                                },
                            },
                        },
                    },
                },
            },
        })
        if err != nil {
            return err
        }

        // Export deployment name
        ctx.Export("deploymentName", deployment.Metadata.Name())

        return nil
    })
}
```

**Deploy with Pulumi:**

```bash
# Login
pulumi login

# Create new stack
pulumi stack init production

# Deploy
pulumi up

# Destroy
pulumi destroy
```

---

## Part 7 Summary and Key Takeaways

### Deployment Fundamentals

**✅ Containerization:**
- Multi-stage Docker builds for minimal images
- Security best practices (non-root user, scratch/distroless)
- Image optimization (<15MB final images)
- Development with Docker Compose

**✅ Kubernetes Deployment:**
- Deployments, Services, Ingress
- ConfigMaps and Secrets
- HPA for auto-scaling
- PodDisruptionBudgets for availability
- Kustomize for multi-environment

**✅ CI/CD Pipelines:**
- GitHub Actions workflows
- GitLab CI/CD pipelines
- CircleCI configuration
- Automated testing and deployment
- Multi-environment promotion

### Cloud Platforms

**✅ AWS:**
- ECS/Fargate for containers
- EKS for Kubernetes
- ECR for container registry
- Secrets Manager integration

**✅ GCP:**
- Cloud Run for serverless
- GKE for Kubernetes
- GCR for container registry
- Secret Manager integration

**✅ Azure:**
- ACI for containers
- AKS for Kubernetes
- ACR for container registry
- Key Vault integration

### Advanced Strategies

**✅ Deployment Patterns:**
- Blue-Green deployments
- Canary releases with Flagger
- Rolling updates with health checks
- Progressive delivery

**✅ Secrets Management:**
- Kubernetes Secrets
- Sealed Secrets
- External Secrets Operator
- HashiCorp Vault
- Cloud provider secret managers

**✅ Infrastructure as Code:**
- Terraform for declarative infrastructure
- Pulumi for programmatic infrastructure
- Version-controlled infrastructure
- Repeatable deployments

### Best Practices Checklist

**Docker:**
- [ ] Use multi-stage builds
- [ ] Run as non-root user
- [ ] Use specific image tags
- [ ] Minimize image layers
- [ ] Implement health checks
- [ ] Scan for vulnerabilities

**Kubernetes:**
- [ ] Define resource limits
- [ ] Implement health probes
- [ ] Use HPA for scaling
- [ ] Set PodDisruptionBudget
- [ ] Use namespaces
- [ ] Implement RBAC

**CI/CD:**
- [ ] Automated testing
- [ ] Security scanning
- [ ] Multi-stage pipelines
- [ ] Environment promotion
- [ ] Rollback capability
- [ ] Deployment notifications

**Production:**
- [ ] Blue-green or canary deployments
- [ ] Secrets in secure storage
- [ ] Infrastructure as code
- [ ] Monitoring and alerting
- [ ] Disaster recovery plan
- [ ] Documentation

### Common Pitfalls to Avoid

**❌ Docker:**
- Running as root
- Large image sizes
- Using latest tag in production
- Secrets in images
- No health checks

**❌ Kubernetes:**
- No resource limits
- Missing health probes
- Single replica in production
- Secrets in ConfigMaps
- No PodDisruptionBudget

**❌ CI/CD:**
- No automated tests
- Direct production deploys
- No rollback strategy
- Secrets in code
- No deployment verification

### Production Deployment Workflow

```
1. Development
   ├─ Write code
   ├─ Run locally with Docker Compose
   └─ Create tests

2. CI Pipeline
   ├─ Run tests
   ├─ Run linters
   ├─ Security scan
   └─ Build Docker image

3. CD Pipeline
   ├─ Push to container registry
   ├─ Deploy to staging
   ├─ Run integration tests
   └─ Deploy to production (manual approval)

4. Production
   ├─ Canary or blue-green deployment
   ├─ Monitor metrics
   ├─ Health checks passing
   └─ Rollback if issues
```

### What's Next

With comprehensive deployment knowledge, you can:

1. **Deploy confidently** - Automated, tested pipelines
2. **Scale effectively** - Kubernetes auto-scaling
3. **Manage secrets** - Secure credential handling
4. **Release safely** - Canary and blue-green strategies
5. **Maintain infrastructure** - Infrastructure as code

**You're now equipped to deploy Go applications to any cloud platform!**

---

This completes Part 7 of the Complete Go Mastery series covering deployment, DevOps, and cloud-native practices!

