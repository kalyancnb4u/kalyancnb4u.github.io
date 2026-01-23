---
title: "Complete TypeScript Mastery Part 8: Production Deployment, Monitoring & Performance"
date: 2026-01-08 17:00:00 +0530
categories: [Programming, TypeScript, Web Development]
tags: [typescript, production, deployment, monitoring, performance, docker, kubernetes, ci-cd, observability]
---

# Complete TypeScript Mastery Part 8: Production Deployment, Monitoring & Performance

## Introduction

Welcome to Part 8, the final part of the Complete TypeScript Mastery series! In Parts 1-7, we covered TypeScript from fundamentals to building complete real-world applications. Now we'll explore production deployment, monitoring, performance optimization, and operational excellence.

This part focuses on production topics:
- Docker containerization
- Kubernetes orchestration
- CI/CD pipelines
- Monitoring and observability
- Logging strategies
- Performance optimization
- Error tracking and debugging
- Load balancing and scaling
- Database optimization
- Caching strategies
- Security hardening
- Disaster recovery

By the end of this part, you'll have complete mastery of deploying and operating TypeScript applications at scale in production environments.

---

## 8.1 Docker Containerization

### Basic Dockerfile

```dockerfile
# Dockerfile for Node.js/TypeScript application
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install dependencies
RUN npm ci

# Build the application
FROM base AS builder
WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy source code
COPY . .

# Build TypeScript
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

# Set production environment
ENV NODE_ENV=production

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nodejs

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./package.json

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start application
CMD ["node", "dist/index.js"]
```

### Multi-stage Build for Next.js

```dockerfile
# Dockerfile for Next.js application
FROM node:20-alpine AS base

# Install dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

# Build application
FROM base AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build Next.js
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy public assets
COPY --from=builder /app/public ./public

# Copy built application
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

### Docker Compose for Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
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

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### Development Dockerfile

```dockerfile
# Dockerfile.dev
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm ci

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start development server
CMD ["npm", "run", "dev"]
```

### Docker Optimization Tips

```typescript
// .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
dist
build
coverage
.next
.cache
*.md
!README.md
```

### Build Script

```typescript
// scripts/docker-build.ts
import { execSync } from 'child_process'
import * as fs from 'fs'

interface BuildConfig {
  image: string
  tag: string
  dockerfile: string
  platform?: string
  push?: boolean
}

function buildImage(config: BuildConfig): void {
  console.log(`Building ${config.image}:${config.tag}...`)

  const platformArg = config.platform ? `--platform ${config.platform}` : ''
  
  const buildCommand = `docker build ${platformArg} -t ${config.image}:${config.tag} -f ${config.dockerfile} .`

  try {
    execSync(buildCommand, { stdio: 'inherit' })
    console.log('✓ Build successful')

    if (config.push) {
      console.log('Pushing image...')
      execSync(`docker push ${config.image}:${config.tag}`, { stdio: 'inherit' })
      console.log('✓ Push successful')
    }
  } catch (error) {
    console.error('✗ Build failed:', error)
    process.exit(1)
  }
}

// Read version from package.json
const packageJson = JSON.parse(fs.readFileSync('package.json', 'utf-8'))
const version = packageJson.version

buildImage({
  image: 'myapp',
  tag: version,
  dockerfile: 'Dockerfile',
  platform: 'linux/amd64,linux/arm64',
  push: process.env.PUSH === 'true'
})
```

---

## 8.2 Kubernetes Deployment

### Deployment Configuration

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
    version: v1
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
        version: v1
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:1.0.0
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: NODE_ENV
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
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: config
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: myapp-config
```

### Service Configuration

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
  labels:
    app: myapp
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
    name: http
  selector:
    app: myapp
```

### Ingress Configuration

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  tls:
  - hosts:
    - myapp.com
    - www.myapp.com
    secretName: myapp-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
  - host: www.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

### ConfigMap and Secrets

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production
data:
  redis-url: "redis://redis-service:6379"
  log-level: "info"
  api-timeout: "5000"

---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: production
type: Opaque
stringData:
  database-url: "postgresql://user:password@postgres:5432/mydb"
  jwt-secret: "your-secret-key"
  stripe-key: "sk_live_..."
```

### Horizontal Pod Autoscaler

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
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
        value: 2
        periodSeconds: 15
      selectPolicy: Max
```

### Deployment Script

```typescript
// scripts/k8s-deploy.ts
import { execSync } from 'child_process'
import * as fs from 'fs'
import * as path from 'path'

interface DeployConfig {
  namespace: string
  environment: 'development' | 'staging' | 'production'
  version: string
  dryRun?: boolean
}

class KubernetesDeployer {
  private config: DeployConfig

  constructor(config: DeployConfig) {
    this.config = config
  }

  deploy(): void {
    console.log(`Deploying to ${this.config.environment}...`)

    // Apply configurations
    this.applyConfigMaps()
    this.applySecrets()
    this.applyDeployment()
    this.applyService()
    this.applyIngress()
    this.applyHPA()

    // Wait for rollout
    this.waitForRollout()

    console.log('✓ Deployment successful')
  }

  private applyConfigMaps(): void {
    console.log('Applying ConfigMaps...')
    this.kubectl('apply', 'k8s/configmap.yaml')
  }

  private applySecrets(): void {
    console.log('Applying Secrets...')
    this.kubectl('apply', 'k8s/secret.yaml')
  }

  private applyDeployment(): void {
    console.log('Applying Deployment...')
    
    // Update image tag
    const deployment = this.readYaml('k8s/deployment.yaml')
    deployment.spec.template.spec.containers[0].image = 
      `myregistry/myapp:${this.config.version}`
    
    this.writeYaml('k8s/deployment.yaml', deployment)
    this.kubectl('apply', 'k8s/deployment.yaml')
  }

  private applyService(): void {
    console.log('Applying Service...')
    this.kubectl('apply', 'k8s/service.yaml')
  }

  private applyIngress(): void {
    console.log('Applying Ingress...')
    this.kubectl('apply', 'k8s/ingress.yaml')
  }

  private applyHPA(): void {
    console.log('Applying HPA...')
    this.kubectl('apply', 'k8s/hpa.yaml')
  }

  private waitForRollout(): void {
    console.log('Waiting for rollout...')
    this.kubectl('rollout', 'status', 'deployment/myapp')
  }

  private kubectl(...args: string[]): void {
    const dryRun = this.config.dryRun ? '--dry-run=client' : ''
    const namespace = `-n ${this.config.namespace}`
    
    const command = `kubectl ${args.join(' ')} ${namespace} ${dryRun}`
    
    try {
      execSync(command, { stdio: 'inherit' })
    } catch (error) {
      console.error('kubectl command failed:', error)
      process.exit(1)
    }
  }

  private readYaml(filePath: string): any {
    const content = fs.readFileSync(filePath, 'utf-8')
    return require('js-yaml').load(content)
  }

  private writeYaml(filePath: string, data: any): void {
    const content = require('js-yaml').dump(data)
    fs.writeFileSync(filePath, content)
  }

  rollback(): void {
    console.log('Rolling back deployment...')
    this.kubectl('rollout', 'undo', 'deployment/myapp')
    console.log('✓ Rollback successful')
  }

  getStatus(): void {
    console.log('Getting deployment status...')
    this.kubectl('get', 'deployments')
    this.kubectl('get', 'pods')
    this.kubectl('get', 'services')
  }
}

// Usage
const deployer = new KubernetesDeployer({
  namespace: 'production',
  environment: 'production',
  version: '1.0.0',
  dryRun: false
})

deployer.deploy()
```

---

## 8.3 CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run type-check

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/

  docker:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
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
            type=semver,pattern={{version}}
            type=sha,prefix={{branch}}-

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    name: Deploy to Kubernetes
    runs-on: ubuntu-latest
    needs: docker
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBE_CONFIG }}

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/configmap.yaml
          kubectl apply -f k8s/secret.yaml
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
          kubectl apply -f k8s/ingress.yaml
          kubectl rollout status deployment/myapp -n production

      - name: Verify deployment
        run: |
          kubectl get deployments -n production
          kubectl get pods -n production
          kubectl get services -n production
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_REGISTRY: registry.gitlab.com
  DOCKER_IMAGE: $DOCKER_REGISTRY/$CI_PROJECT_PATH
  KUBERNETES_NAMESPACE: production

test:
  stage: test
  image: node:20
  cache:
    paths:
      - node_modules/
  before_script:
    - npm ci
  script:
    - npm run lint
    - npm run type-check
    - npm test -- --coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: node:20
  cache:
    paths:
      - node_modules/
  before_script:
    - npm ci
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker tag $DOCKER_IMAGE:$CI_COMMIT_SHA $DOCKER_IMAGE:latest
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHA
    - docker push $DOCKER_IMAGE:latest
  only:
    - main

deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  before_script:
    - kubectl config use-context $KUBE_CONTEXT
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE:$CI_COMMIT_SHA -n $KUBERNETES_NAMESPACE
    - kubectl rollout status deployment/myapp -n $KUBERNETES_NAMESPACE
  environment:
    name: production
    url: https://myapp.com
  only:
    - main
  when: manual
```

### TypeScript CI Configuration

```typescript
// scripts/ci.ts
import { execSync } from 'child_process'

interface CIConfig {
  skipTests?: boolean
  skipBuild?: boolean
  skipLint?: boolean
}

class CIRunner {
  constructor(private config: CIConfig = {}) {}

  async run(): Promise<void> {
    console.log('🚀 Starting CI pipeline...\n')

    try {
      await this.installDependencies()
      
      if (!this.config.skipLint) {
        await this.runLinter()
      }
      
      await this.typeCheck()
      
      if (!this.config.skipTests) {
        await this.runTests()
      }
      
      if (!this.config.skipBuild) {
        await this.build()
      }

      console.log('\n✓ CI pipeline completed successfully!')
      process.exit(0)
    } catch (error) {
      console.error('\n✗ CI pipeline failed:', error)
      process.exit(1)
    }
  }

  private async installDependencies(): Promise<void> {
    console.log('📦 Installing dependencies...')
    this.exec('npm ci')
  }

  private async runLinter(): Promise<void> {
    console.log('🔍 Running linter...')
    this.exec('npm run lint')
  }

  private async typeCheck(): Promise<void> {
    console.log('📝 Type checking...')
    this.exec('npm run type-check')
  }

  private async runTests(): Promise<void> {
    console.log('🧪 Running tests...')
    this.exec('npm test -- --coverage')
  }

  private async build(): Promise<void> {
    console.log('🏗️  Building application...')
    this.exec('npm run build')
  }

  private exec(command: string): void {
    execSync(command, { stdio: 'inherit' })
  }
}

// Run CI
const ci = new CIRunner({
  skipTests: process.env.SKIP_TESTS === 'true',
  skipBuild: process.env.SKIP_BUILD === 'true',
  skipLint: process.env.SKIP_LINT === 'true'
})

ci.run()
```

---

## 8.4 Logging and Monitoring

### Structured Logging

```typescript
// src/logging/Logger.ts
import winston from 'winston'
import DailyRotateFile from 'winston-daily-rotate-file'

export enum LogLevel {
  Error = 'error',
  Warn = 'warn',
  Info = 'info',
  Http = 'http',
  Debug = 'debug'
}

export interface LogMetadata {
  [key: string]: any
}

export class Logger {
  private logger: winston.Logger

  constructor(serviceName: string) {
    const format = winston.format.combine(
      winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
      winston.format.errors({ stack: true }),
      winston.format.splat(),
      winston.format.json(),
      winston.format.printf(({ timestamp, level, message, ...metadata }) => {
        let msg = `${timestamp} [${level.toUpperCase()}]: ${message}`
        
        if (Object.keys(metadata).length > 0) {
          msg += ` ${JSON.stringify(metadata)}`
        }
        
        return msg
      })
    )

    this.logger = winston.createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format,
      defaultMeta: { service: serviceName },
      transports: [
        // Console transport
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.colorize(),
            winston.format.simple()
          )
        }),

        // Error log file
        new DailyRotateFile({
          filename: 'logs/error-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          level: 'error',
          maxSize: '20m',
          maxFiles: '14d'
        }),

        // Combined log file
        new DailyRotateFile({
          filename: 'logs/combined-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          maxSize: '20m',
          maxFiles: '14d'
        })
      ]
    })

    // Handle unhandled exceptions and rejections
    this.logger.exceptions.handle(
      new DailyRotateFile({
        filename: 'logs/exceptions-%DATE%.log',
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '14d'
      })
    )

    this.logger.rejections.handle(
      new DailyRotateFile({
        filename: 'logs/rejections-%DATE%.log',
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '14d'
      })
    )
  }

  error(message: string, metadata?: LogMetadata): void {
    this.logger.error(message, metadata)
  }

  warn(message: string, metadata?: LogMetadata): void {
    this.logger.warn(message, metadata)
  }

  info(message: string, metadata?: LogMetadata): void {
    this.logger.info(message, metadata)
  }

  http(message: string, metadata?: LogMetadata): void {
    this.logger.http(message, metadata)
  }

  debug(message: string, metadata?: LogMetadata): void {
    this.logger.debug(message, metadata)
  }

  // Correlation ID support
  withCorrelationId(correlationId: string) {
    return {
      error: (message: string, metadata?: LogMetadata) =>
        this.error(message, { ...metadata, correlationId }),
      warn: (message: string, metadata?: LogMetadata) =>
        this.warn(message, { ...metadata, correlationId }),
      info: (message: string, metadata?: LogMetadata) =>
        this.info(message, { ...metadata, correlationId }),
      http: (message: string, metadata?: LogMetadata) =>
        this.http(message, { ...metadata, correlationId }),
      debug: (message: string, metadata?: LogMetadata) =>
        this.debug(message, { ...metadata, correlationId })
    }
  }
}

// Global logger instance
export const logger = new Logger('myapp')
```

### Request Logging Middleware

```typescript
// src/middleware/requestLogger.ts
import { Request, Response, NextFunction } from 'express'
import { v4 as uuidv4 } from 'uuid'
import { logger } from '../logging/Logger'

export interface RequestWithId extends Request {
  id: string
}

export function requestLogger(req: RequestWithId, res: Response, next: NextFunction): void {
  // Generate request ID
  req.id = req.headers['x-request-id'] as string || uuidv4()
  res.setHeader('X-Request-Id', req.id)

  const startTime = Date.now()

  // Log request
  logger.info('Incoming request', {
    requestId: req.id,
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.headers['user-agent']
  })

  // Log response
  res.on('finish', () => {
    const duration = Date.now() - startTime

    logger.http('Request completed', {
      requestId: req.id,
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: `${duration}ms`
    })
  })

  next()
}
```

### Application Performance Monitoring

```typescript
// src/monitoring/APM.ts
import * as Sentry from '@sentry/node'
import { ProfilingIntegration } from '@sentry/profiling-node'

export class APMService {
  static initialize(): void {
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
      integrations: [
        new ProfilingIntegration()
      ],
      tracesSampleRate: 1.0,
      profilesSampleRate: 1.0
    })
  }

  static captureException(error: Error, context?: Record<string, any>): void {
    Sentry.captureException(error, {
      contexts: context
    })
  }

  static captureMessage(message: string, level: Sentry.SeverityLevel = 'info'): void {
    Sentry.captureMessage(message, level)
  }

  static setUser(user: { id: string; email?: string; username?: string }): void {
    Sentry.setUser(user)
  }

  static addBreadcrumb(breadcrumb: Sentry.Breadcrumb): void {
    Sentry.addBreadcrumb(breadcrumb)
  }

  static startTransaction(name: string, op: string): Sentry.Transaction {
    return Sentry.startTransaction({ name, op })
  }
}
```

### Metrics Collection

```typescript
// src/monitoring/Metrics.ts
import { Counter, Histogram, Gauge, register } from 'prom-client'

export class MetricsService {
  private static httpRequestsTotal: Counter
  private static httpRequestDuration: Histogram
  private static activeConnections: Gauge
  private static databaseQueryDuration: Histogram

  static initialize(): void {
    // HTTP request counter
    this.httpRequestsTotal = new Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status']
    })

    // HTTP request duration
    this.httpRequestDuration = new Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route', 'status'],
      buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1, 5]
    })

    // Active connections
    this.activeConnections = new Gauge({
      name: 'active_connections',
      help: 'Number of active connections'
    })

    // Database query duration
    this.databaseQueryDuration = new Histogram({
      name: 'database_query_duration_seconds',
      help: 'Duration of database queries in seconds',
      labelNames: ['operation', 'table'],
      buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1]
    })
  }

  static recordHttpRequest(
    method: string,
    route: string,
    status: number,
    duration: number
  ): void {
    this.httpRequestsTotal.inc({ method, route, status })
    this.httpRequestDuration.observe({ method, route, status }, duration)
  }

  static incrementActiveConnections(): void {
    this.activeConnections.inc()
  }

  static decrementActiveConnections(): void {
    this.activeConnections.dec()
  }

  static recordDatabaseQuery(
    operation: string,
    table: string,
    duration: number
  ): void {
    this.databaseQueryDuration.observe({ operation, table }, duration)
  }

  static getMetrics(): Promise<string> {
    return register.metrics()
  }
}

// Middleware for metrics
import { Request, Response, NextFunction } from 'express'

export function metricsMiddleware(req: Request, res: Response, next: NextFunction): void {
  const startTime = Date.now()

  MetricsService.incrementActiveConnections()

  res.on('finish', () => {
    const duration = (Date.now() - startTime) / 1000

    MetricsService.recordHttpRequest(
      req.method,
      req.route?.path || req.path,
      res.statusCode,
      duration
    )

    MetricsService.decrementActiveConnections()
  })

  next()
}

// Metrics endpoint
import express from 'express'

const router = express.Router()

router.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType)
  res.end(await MetricsService.getMetrics())
})

export default router
```

export default router
```

---

## 8.5 Performance Optimization

### Code Splitting and Lazy Loading

```typescript
// React code splitting
import React, { lazy, Suspense } from 'react'
import { BrowserRouter, Routes, Route } from 'react-router-dom'

// Lazy load components
const Home = lazy(() => import('./pages/Home'))
const Dashboard = lazy(() => import('./pages/Dashboard'))
const Profile = lazy(() => import('./pages/Profile'))

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}

// Dynamic imports
async function loadModule() {
  const module = await import('./heavy-module')
  return module.default
}

// Webpack magic comments
const Chart = lazy(
  () => import(/* webpackChunkName: "chart" */ './components/Chart')
)
```

### Bundle Analysis and Optimization

```typescript
// webpack.config.ts
import webpack from 'webpack'
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer'
import TerserPlugin from 'terser-webpack-plugin'
import CompressionPlugin from 'compression-webpack-plugin'

const config: webpack.Configuration = {
  mode: 'production',
  
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true
          }
        }
      })
    ],
    
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          name: 'vendors'
        },
        common: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    },
    
    runtimeChunk: 'single'
  },
  
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false
    }),
    
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240,
      minRatio: 0.8
    })
  ]
}

export default config
```

### Memory Management

```typescript
// Memory leak prevention
class ResourceManager {
  private resources = new Map<string, any>()
  private timers = new Set<NodeJS.Timeout>()
  private listeners = new Map<EventEmitter, Array<[string, Function]>>()

  addResource(key: string, resource: any): void {
    this.resources.set(key, resource)
  }

  addTimer(timer: NodeJS.Timeout): void {
    this.timers.add(timer)
  }

  addEventListener(
    emitter: EventEmitter,
    event: string,
    listener: Function
  ): void {
    if (!this.listeners.has(emitter)) {
      this.listeners.set(emitter, [])
    }
    this.listeners.get(emitter)!.push([event, listener])
    emitter.on(event, listener as any)
  }

  cleanup(): void {
    // Clear resources
    this.resources.clear()

    // Clear timers
    this.timers.forEach(timer => clearTimeout(timer))
    this.timers.clear()

    // Remove event listeners
    this.listeners.forEach((listeners, emitter) => {
      listeners.forEach(([event, listener]) => {
        emitter.removeListener(event, listener as any)
      })
    })
    this.listeners.clear()
  }
}

// Usage in React
import { useEffect, useRef } from 'react'

function useResourceManager() {
  const managerRef = useRef<ResourceManager>()

  useEffect(() => {
    managerRef.current = new ResourceManager()

    return () => {
      managerRef.current?.cleanup()
    }
  }, [])

  return managerRef.current
}
```

### Database Query Optimization

```typescript
// Efficient query patterns
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// ❌ N+1 query problem
async function getUsersWithPostsBad() {
  const users = await prisma.user.findMany()
  
  for (const user of users) {
    user.posts = await prisma.post.findMany({
      where: { authorId: user.id }
    })
  }
  
  return users
}

// ✅ Use includes
async function getUsersWithPostsGood() {
  return prisma.user.findMany({
    include: {
      posts: true
    }
  })
}

// ✅ Select only needed fields
async function getUsersOptimized() {
  return prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      posts: {
        select: {
          id: true,
          title: true,
          createdAt: true
        }
      }
    }
  })
}

// Batch operations
async function createManyUsers(users: Array<{ name: string; email: string }>) {
  return prisma.user.createMany({
    data: users,
    skipDuplicates: true
  })
}

// Pagination with cursor
async function getPaginatedPosts(cursor?: string, limit: number = 10) {
  return prisma.post.findMany({
    take: limit,
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: {
      createdAt: 'desc'
    }
  })
}

// Connection pooling
const prismaWithPool = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  },
  log: ['query', 'error', 'warn'],
  // Connection pool settings
  connectionLimit: 10
})
```

### API Response Optimization

```typescript
// Response compression
import compression from 'compression'
import express from 'express'

const app = express()

app.use(compression({
  level: 6,
  threshold: 1024,
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false
    }
    return compression.filter(req, res)
  }
}))

// ETags for caching
app.use((req, res, next) => {
  res.set('Cache-Control', 'public, max-age=300')
  next()
})

// Response streaming for large data
import { Transform } from 'stream'

app.get('/api/large-dataset', async (req, res) => {
  res.setHeader('Content-Type', 'application/json')
  res.write('[')

  let first = true
  const transform = new Transform({
    objectMode: true,
    transform(chunk, encoding, callback) {
      const prefix = first ? '' : ','
      first = false
      callback(null, prefix + JSON.stringify(chunk))
    }
  })

  const dataStream = await getLargeDataset()
  dataStream.pipe(transform).pipe(res)

  dataStream.on('end', () => {
    res.write(']')
    res.end()
  })
})
```

### Image Optimization

```typescript
// Image processing service
import sharp from 'sharp'
import { S3 } from '@aws-sdk/client-s3'

interface ImageOptimizationOptions {
  width?: number
  height?: number
  quality?: number
  format?: 'jpeg' | 'png' | 'webp'
}

class ImageOptimizer {
  private s3: S3

  constructor() {
    this.s3 = new S3({ region: process.env.AWS_REGION })
  }

  async optimizeAndUpload(
    buffer: Buffer,
    key: string,
    options: ImageOptimizationOptions = {}
  ): Promise<string> {
    const {
      width = 1200,
      quality = 80,
      format = 'webp'
    } = options

    // Optimize image
    const optimized = await sharp(buffer)
      .resize(width, null, {
        withoutEnlargement: true,
        fit: 'inside'
      })
      .toFormat(format, { quality })
      .toBuffer()

    // Generate multiple sizes
    const sizes = [
      { width: 400, suffix: 'thumb' },
      { width: 800, suffix: 'medium' },
      { width: 1200, suffix: 'large' }
    ]

    const uploads = sizes.map(async ({ width, suffix }) => {
      const resized = await sharp(buffer)
        .resize(width, null, { withoutEnlargement: true })
        .toFormat(format, { quality })
        .toBuffer()

      const sizeKey = `${key}-${suffix}.${format}`

      await this.s3.putObject({
        Bucket: process.env.S3_BUCKET!,
        Key: sizeKey,
        Body: resized,
        ContentType: `image/${format}`,
        CacheControl: 'public, max-age=31536000'
      })

      return sizeKey
    })

    await Promise.all(uploads)

    // Upload original optimized
    await this.s3.putObject({
      Bucket: process.env.S3_BUCKET!,
      Key: `${key}.${format}`,
      Body: optimized,
      ContentType: `image/${format}`,
      CacheControl: 'public, max-age=31536000'
    })

    return `${key}.${format}`
  }
}
```

---

## 8.6 Caching Strategies

### Redis Caching

```typescript
// Redis cache service
import Redis from 'ioredis'

export class CacheService {
  private redis: Redis

  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
      password: process.env.REDIS_PASSWORD,
      retryStrategy: (times) => {
        const delay = Math.min(times * 50, 2000)
        return delay
      }
    })

    this.redis.on('error', (error) => {
      console.error('Redis error:', error)
    })

    this.redis.on('connect', () => {
      console.log('Redis connected')
    })
  }

  async get<T>(key: string): Promise<T | null> {
    const value = await this.redis.get(key)
    return value ? JSON.parse(value) : null
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value)
    
    if (ttl) {
      await this.redis.setex(key, ttl, serialized)
    } else {
      await this.redis.set(key, serialized)
    }
  }

  async delete(key: string): Promise<void> {
    await this.redis.del(key)
  }

  async deletePattern(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern)
    
    if (keys.length > 0) {
      await this.redis.del(...keys)
    }
  }

  async exists(key: string): Promise<boolean> {
    const result = await this.redis.exists(key)
    return result === 1
  }

  async increment(key: string): Promise<number> {
    return this.redis.incr(key)
  }

  async decrement(key: string): Promise<number> {
    return this.redis.decr(key)
  }

  async expire(key: string, seconds: number): Promise<void> {
    await this.redis.expire(key, seconds)
  }

  async getOrSet<T>(
    key: string,
    factory: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    const cached = await this.get<T>(key)
    
    if (cached !== null) {
      return cached
    }

    const value = await factory()
    await this.set(key, value, ttl)
    
    return value
  }

  async disconnect(): Promise<void> {
    await this.redis.quit()
  }
}

// Cache decorator
function Cacheable(ttl: number = 300) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value

    descriptor.value = async function (...args: any[]) {
      const cacheKey = `${target.constructor.name}:${propertyKey}:${JSON.stringify(args)}`
      const cache = new CacheService()

      const cached = await cache.get(cacheKey)
      
      if (cached !== null) {
        return cached
      }

      const result = await originalMethod.apply(this, args)
      await cache.set(cacheKey, result, ttl)

      return result
    }

    return descriptor
  }
}

// Usage
class UserService {
  @Cacheable(600) // Cache for 10 minutes
  async getUserById(id: string): Promise<User> {
    // Expensive database operation
    return prisma.user.findUnique({ where: { id } })
  }
}
```

### Cache-Aside Pattern

```typescript
// Cache-aside implementation
class DataService {
  constructor(
    private cache: CacheService,
    private database: PrismaClient
  ) {}

  async getUser(id: string): Promise<User | null> {
    // Try cache first
    const cacheKey = `user:${id}`
    const cached = await this.cache.get<User>(cacheKey)

    if (cached) {
      return cached
    }

    // Cache miss - fetch from database
    const user = await this.database.user.findUnique({
      where: { id }
    })

    if (user) {
      // Store in cache
      await this.cache.set(cacheKey, user, 3600)
    }

    return user
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    // Update database
    const user = await this.database.user.update({
      where: { id },
      data
    })

    // Invalidate cache
    await this.cache.delete(`user:${id}`)

    return user
  }

  async deleteUser(id: string): Promise<void> {
    // Delete from database
    await this.database.user.delete({ where: { id } })

    // Invalidate cache
    await this.cache.delete(`user:${id}`)
  }
}
```

### Write-Through Cache

```typescript
// Write-through cache implementation
class WriteThroughCache {
  constructor(
    private cache: CacheService,
    private database: PrismaClient
  ) {}

  async set(key: string, value: any): Promise<void> {
    // Write to database first
    await this.database.cacheEntry.upsert({
      where: { key },
      update: { value: JSON.stringify(value) },
      create: { key, value: JSON.stringify(value) }
    })

    // Then update cache
    await this.cache.set(key, value)
  }

  async get(key: string): Promise<any | null> {
    // Try cache first
    const cached = await this.cache.get(key)

    if (cached !== null) {
      return cached
    }

    // Cache miss - fetch from database
    const entry = await this.database.cacheEntry.findUnique({
      where: { key }
    })

    if (entry) {
      const value = JSON.parse(entry.value)
      // Populate cache
      await this.cache.set(key, value)
      return value
    }

    return null
  }
}
```

### HTTP Caching Headers

```typescript
// HTTP caching middleware
import { Request, Response, NextFunction } from 'express'

export function cacheControl(options: {
  maxAge?: number
  sMaxAge?: number
  public?: boolean
  private?: boolean
  noCache?: boolean
  noStore?: boolean
  mustRevalidate?: boolean
}) {
  return (req: Request, res: Response, next: NextFunction) => {
    const directives: string[] = []

    if (options.public) directives.push('public')
    if (options.private) directives.push('private')
    if (options.noCache) directives.push('no-cache')
    if (options.noStore) directives.push('no-store')
    if (options.mustRevalidate) directives.push('must-revalidate')
    if (options.maxAge !== undefined) directives.push(`max-age=${options.maxAge}`)
    if (options.sMaxAge !== undefined) directives.push(`s-maxage=${options.sMaxAge}`)

    res.setHeader('Cache-Control', directives.join(', '))
    next()
  }
}

// Usage
app.get(
  '/api/products',
  cacheControl({ public: true, maxAge: 300 }),
  async (req, res) => {
    const products = await getProducts()
    res.json(products)
  }
)

// ETag support
import etag from 'etag'

app.get('/api/user/:id', async (req, res) => {
  const user = await getUserById(req.params.id)
  
  const userEtag = etag(JSON.stringify(user))
  
  if (req.headers['if-none-match'] === userEtag) {
    res.status(304).end()
    return
  }

  res.setHeader('ETag', userEtag)
  res.setHeader('Cache-Control', 'private, max-age=300')
  res.json(user)
})
```

---

## 8.7 Database Optimization

### Connection Pooling

```typescript
// PostgreSQL connection pool
import { Pool } from 'pg'

const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20, // Maximum pool size
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
})

// Query with connection pool
async function queryWithPool<T>(
  text: string,
  params?: any[]
): Promise<T[]> {
  const client = await pool.connect()
  
  try {
    const result = await client.query(text, params)
    return result.rows
  } finally {
    client.release()
  }
}

// Transaction support
async function executeTransaction<T>(
  callback: (client: PoolClient) => Promise<T>
): Promise<T> {
  const client = await pool.connect()
  
  try {
    await client.query('BEGIN')
    const result = await callback(client)
    await client.query('COMMIT')
    return result
  } catch (error) {
    await client.query('ROLLBACK')
    throw error
  } finally {
    client.release()
  }
}
```

### Query Optimization

```typescript
// Index creation
const createIndexes = `
  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email 
  ON users(email);

  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_posts_author_id 
  ON posts(author_id);

  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_posts_created_at 
  ON posts(created_at DESC);

  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_posts_author_created 
  ON posts(author_id, created_at DESC);
`

// Composite indexes for common queries
const createCompositeIndexes = `
  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_user_status
  ON orders(user_id, status)
  WHERE status IN ('pending', 'processing');
`

// Partial indexes
const createPartialIndexes = `
  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_active_users
  ON users(id, email)
  WHERE is_active = true;
`

// Query analysis
async function analyzeQuery(query: string): Promise<void> {
  const result = await pool.query(`EXPLAIN ANALYZE ${query}`)
  console.log('Query Plan:')
  result.rows.forEach(row => console.log(row['QUERY PLAN']))
}
```

### Database Migrations

```typescript
// Migration runner
import * as fs from 'fs/promises'
import * as path from 'path'
import { Pool } from 'pg'

interface Migration {
  version: number
  name: string
  up: string
  down: string
}

class MigrationRunner {
  constructor(private pool: Pool) {}

  async initialize(): Promise<void> {
    await this.pool.query(`
      CREATE TABLE IF NOT EXISTS migrations (
        version INTEGER PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `)
  }

  async getCurrentVersion(): Promise<number> {
    const result = await this.pool.query(
      'SELECT MAX(version) as version FROM migrations'
    )
    return result.rows[0].version || 0
  }

  async loadMigrations(directory: string): Promise<Migration[]> {
    const files = await fs.readdir(directory)
    const migrations: Migration[] = []

    for (const file of files.sort()) {
      if (file.endsWith('.sql')) {
        const content = await fs.readFile(
          path.join(directory, file),
          'utf-8'
        )
        
        const [up, down] = content.split('-- DOWN')
        const match = file.match(/^(\d+)_(.+)\.sql$/)
        
        if (match) {
          migrations.push({
            version: parseInt(match[1]),
            name: match[2],
            up: up.replace('-- UP', '').trim(),
            down: down ? down.trim() : ''
          })
        }
      }
    }

    return migrations
  }

  async migrate(directory: string): Promise<void> {
    await this.initialize()
    
    const currentVersion = await this.getCurrentVersion()
    const migrations = await this.loadMigrations(directory)
    
    const pending = migrations.filter(m => m.version > currentVersion)

    if (pending.length === 0) {
      console.log('No pending migrations')
      return
    }

    console.log(`Running ${pending.length} migrations...`)

    for (const migration of pending) {
      console.log(`Migrating ${migration.version}_${migration.name}...`)
      
      const client = await this.pool.connect()
      
      try {
        await client.query('BEGIN')
        await client.query(migration.up)
        await client.query(
          'INSERT INTO migrations (version, name) VALUES ($1, $2)',
          [migration.version, migration.name]
        )
        await client.query('COMMIT')
        
        console.log(`✓ Migrated ${migration.version}_${migration.name}`)
      } catch (error) {
        await client.query('ROLLBACK')
        console.error(`✗ Migration failed:`, error)
        throw error
      } finally {
        client.release()
      }
    }
  }

  async rollback(): Promise<void> {
    const currentVersion = await this.getCurrentVersion()
    
    if (currentVersion === 0) {
      console.log('No migrations to rollback')
      return
    }

    const result = await this.pool.query(
      'SELECT * FROM migrations WHERE version = $1',
      [currentVersion]
    )

    const migration = result.rows[0]
    const migrations = await this.loadMigrations('./migrations')
    const migrationDef = migrations.find(m => m.version === currentVersion)

    if (!migrationDef) {
      throw new Error(`Migration ${currentVersion} not found`)
    }

    console.log(`Rolling back ${migration.name}...`)

    const client = await this.pool.connect()
    
    try {
      await client.query('BEGIN')
      await client.query(migrationDef.down)
      await client.query(
        'DELETE FROM migrations WHERE version = $1',
        [currentVersion]
      )
      await client.query('COMMIT')
      
      console.log(`✓ Rolled back ${migration.name}`)
    } catch (error) {
      await client.query('ROLLBACK')
      console.error(`✗ Rollback failed:`, error)
      throw error
    } finally {
      client.release()
    }
  }
}
```

---

## 8.8 Security Hardening

### Rate Limiting

```typescript
// Rate limiting middleware
import rateLimit from 'express-rate-limit'
import RedisStore from 'rate-limit-redis'
import Redis from 'ioredis'

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379')
})

// General API rate limit
export const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:api:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per window
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false
})

// Strict rate limit for authentication
export const authLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:auth:'
  }),
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: 'Too many login attempts'
})

// Custom rate limiter
class CustomRateLimiter {
  constructor(
    private redis: Redis,
    private options: {
      points: number
      duration: number
      blockDuration: number
    }
  ) {}

  async consume(key: string): Promise<boolean> {
    const redisKey = `rl:${key}`
    const current = await this.redis.incr(redisKey)

    if (current === 1) {
      await this.redis.expire(redisKey, this.options.duration)
    }

    if (current > this.options.points) {
      // Block the key
      await this.redis.setex(
        `${redisKey}:blocked`,
        this.options.blockDuration,
        '1'
      )
      return false
    }

    return true
  }

  async isBlocked(key: string): Promise<boolean> {
    const blocked = await this.redis.get(`rl:${key}:blocked`)
    return blocked === '1'
  }

  async reset(key: string): Promise<void> {
    await this.redis.del(`rl:${key}`)
    await this.redis.del(`rl:${key}:blocked`)
  }
}

// Usage
app.post('/api/login', authLimiter, async (req, res) => {
  // Login logic
})

app.use('/api', apiLimiter)
```

### Input Validation and Sanitization

```typescript
// Input validation with Zod
import { z } from 'zod'
import { Request, Response, NextFunction } from 'express'

// Schema definitions
const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string()
    .min(8)
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[a-z]/, 'Must contain lowercase')
    .regex(/[0-9]/, 'Must contain number')
    .regex(/[^A-Za-z0-9]/, 'Must contain special character'),
  age: z.number().min(18).max(120).optional()
})

const updateUserSchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional(),
  age: z.number().min(18).max(120).optional()
})

// Validation middleware
function validate<T>(schema: z.ZodSchema<T>) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      req.body = schema.parse(req.body)
      next()
    } catch (error) {
      if (error instanceof z.ZodError) {
        res.status(400).json({
          error: 'Validation failed',
          details: error.errors
        })
      } else {
        res.status(500).json({ error: 'Internal server error' })
      }
    }
  }
}

// Usage
app.post(
  '/api/users',
  validate(createUserSchema),
  async (req, res) => {
    // req.body is now typed and validated
    const user = await createUser(req.body)
    res.json(user)
  }
)
```

### SQL Injection Prevention

```typescript
// Always use parameterized queries
// ❌ NEVER do this
async function getUserBad(email: string) {
  const query = `SELECT * FROM users WHERE email = '${email}'`
  return pool.query(query)
}

// ✅ ALWAYS use parameters
async function getUserGood(email: string) {
  const query = 'SELECT * FROM users WHERE email = $1'
  return pool.query(query, [email])
}

// With Prisma (safe by default)
async function getUserPrisma(email: string) {
  return prisma.user.findUnique({
    where: { email }
  })
}
```

### XSS Prevention

```typescript
// Content Security Policy
import helmet from 'helmet'

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  })
)

// HTML sanitization
import DOMPurify from 'isomorphic-dompurify'

function sanitizeHTML(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href']
  })
}

// Output encoding
function escapeHTML(str: string): string {
  const map: Record<string, string> = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  }
  return str.replace(/[&<>"'/]/g, (char) => map[char])
}
```

### HTTPS and TLS Configuration

```typescript
// HTTPS server configuration
import * as https from 'https'
import * as fs from 'fs'
import express from 'express'

const app = express()

const httpsOptions = {
  key: fs.readFileSync('./certs/private-key.pem'),
  cert: fs.readFileSync('./certs/certificate.pem'),
  ca: fs.readFileSync('./certs/ca-bundle.pem'),
  
  // TLS settings
  minVersion: 'TLSv1.2' as const,
  ciphers: [
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384'
  ].join(':'),
  honorCipherOrder: true
}

const server = https.createServer(httpsOptions, app)

server.listen(443, () => {
  console.log('HTTPS server running on port 443')
})

// Redirect HTTP to HTTPS
const httpApp = express()
httpApp.use((req, res) => {
  res.redirect(301, `https://${req.headers.host}${req.url}`)
})
httpApp.listen(80)
```

### Security Headers

```typescript
// Comprehensive security headers
import helmet from 'helmet'

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  noSniff: true,
  xssFilter: true,
  hidePoweredBy: true
}))

// Custom security headers
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff')
  res.setHeader('X-Frame-Options', 'DENY')
  res.setHeader('X-XSS-Protection', '1; mode=block')
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()')
  next()
})
```

---

## 8.9 Load Balancing and Scaling

### Horizontal Scaling

```typescript
// Cluster mode for Node.js
import cluster from 'cluster'
import * as os from 'os'
import { createServer } from './server'

const numCPUs = os.cpus().length

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`)

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork()
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`)
    console.log('Starting a new worker')
    cluster.fork()
  })
} else {
  const server = createServer()
  
  server.listen(3000, () => {
    console.log(`Worker ${process.pid} started`)
  })
}
```

### Load Balancer Configuration (Nginx)

```nginx
# nginx.conf
upstream backend {
    least_conn;  # Load balancing method
    
    server app1:3000 weight=3;
    server app2:3000 weight=2;
    server app3:3000 weight=1;
    
    # Health check
    keepalive 32;
}

server {
    listen 80;
    server_name example.com;

    # SSL redirect
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    # SSL configuration
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req zone=api burst=20 nodelay;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        
        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffering
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # Static files caching
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### Auto-scaling with Kubernetes

```yaml
# k8s/autoscaling.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 20
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
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 1
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 15
      selectPolicy: Max
```

---

## 8.10 Disaster Recovery and Best Practices

### Backup Strategy

```typescript
// Database backup service
import { exec } from 'child_process'
import { promisify } from 'util'
import * as fs from 'fs/promises'
import * as path from 'path'
import { S3 } from '@aws-sdk/client-s3'

const execAsync = promisify(exec)

class BackupService {
  private s3: S3
  private backupDir = './backups'

  constructor() {
    this.s3 = new S3({ region: process.env.AWS_REGION })
  }

  async createDatabaseBackup(): Promise<string> {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-')
    const filename = `backup-${timestamp}.sql`
    const filepath = path.join(this.backupDir, filename)

    await fs.mkdir(this.backupDir, { recursive: true })

    // Create backup using pg_dump
    const command = `pg_dump ${process.env.DATABASE_URL} > ${filepath}`
    await execAsync(command)

    console.log(`Backup created: ${filename}`)

    // Upload to S3
    await this.uploadToS3(filepath, filename)

    // Cleanup old local backups (keep last 7 days)
    await this.cleanupOldBackups(7)

    return filename
  }

  private async uploadToS3(filepath: string, filename: string): Promise<void> {
    const fileContent = await fs.readFile(filepath)

    await this.s3.putObject({
      Bucket: process.env.BACKUP_BUCKET!,
      Key: `database/${filename}`,
      Body: fileContent,
      ServerSideEncryption: 'AES256'
    })

    console.log(`Backup uploaded to S3: ${filename}`)
  }

  async restoreBackup(filename: string): Promise<void> {
    const filepath = path.join(this.backupDir, filename)

    // Download from S3 if not local
    if (!(await this.fileExists(filepath))) {
      await this.downloadFromS3(filename, filepath)
    }

    // Restore database
    const command = `psql ${process.env.DATABASE_URL} < ${filepath}`
    await execAsync(command)

    console.log(`Database restored from: ${filename}`)
  }

  private async downloadFromS3(filename: string, filepath: string): Promise<void> {
    const response = await this.s3.getObject({
      Bucket: process.env.BACKUP_BUCKET!,
      Key: `database/${filename}`
    })

    if (response.Body) {
      const content = await response.Body.transformToByteArray()
      await fs.writeFile(filepath, content)
      console.log(`Backup downloaded from S3: ${filename}`)
    }
  }

  private async cleanupOldBackups(days: number): Promise<void> {
    const files = await fs.readdir(this.backupDir)
    const cutoffDate = new Date()
    cutoffDate.setDate(cutoffDate.getDate() - days)

    for (const file of files) {
      const filepath = path.join(this.backupDir, file)
      const stats = await fs.stat(filepath)

      if (stats.mtime < cutoffDate) {
        await fs.unlink(filepath)
        console.log(`Deleted old backup: ${file}`)
      }
    }
  }

  private async fileExists(filepath: string): Promise<boolean> {
    try {
      await fs.access(filepath)
      return true
    } catch {
      return false
    }
  }

  async scheduleBackups(): Promise<void> {
    // Run backup every day at 2 AM
    const schedule = require('node-schedule')
    
    schedule.scheduleJob('0 2 * * *', async () => {
      try {
        await this.createDatabaseBackup()
      } catch (error) {
        console.error('Backup failed:', error)
      }
    })

    console.log('Backup schedule initialized')
  }
}

// Usage
const backupService = new BackupService()
backupService.scheduleBackups()
```

### Health Checks

```typescript
// Comprehensive health check system
interface HealthCheck {
  name: string
  check: () => Promise<boolean>
}

class HealthCheckService {
  private checks: HealthCheck[] = []

  addCheck(name: string, check: () => Promise<boolean>): void {
    this.checks.push({ name, check })
  }

  async runChecks(): Promise<{
    status: 'healthy' | 'unhealthy'
    checks: Array<{ name: string; status: boolean; error?: string }>
    timestamp: string
  }> {
    const results = await Promise.all(
      this.checks.map(async ({ name, check }) => {
        try {
          const status = await check()
          return { name, status }
        } catch (error) {
          return {
            name,
            status: false,
            error: error instanceof Error ? error.message : 'Unknown error'
          }
        }
      })
    )

    const allHealthy = results.every(r => r.status)

    return {
      status: allHealthy ? 'healthy' : 'unhealthy',
      checks: results,
      timestamp: new Date().toISOString()
    }
  }
}

// Setup health checks
const healthCheck = new HealthCheckService()

// Database check
healthCheck.addCheck('database', async () => {
  try {
    await prisma.$queryRaw`SELECT 1`
    return true
  } catch {
    return false
  }
})

// Redis check
healthCheck.addCheck('redis', async () => {
  try {
    await redis.ping()
    return true
  } catch {
    return false
  }
})

// External API check
healthCheck.addCheck('payment-api', async () => {
  try {
    const response = await fetch('https://api.stripe.com/v1/health')
    return response.ok
  } catch {
    return false
  }
})

// Health check endpoint
app.get('/health', async (req, res) => {
  const result = await healthCheck.runChecks()
  const statusCode = result.status === 'healthy' ? 200 : 503
  res.status(statusCode).json(result)
})
```

### Production Best Practices Checklist

```typescript
// Production readiness checklist
const productionChecklist = {
  security: [
    '✓ HTTPS/TLS configured',
    '✓ Security headers implemented',
    '✓ Rate limiting enabled',
    '✓ Input validation on all endpoints',
    '✓ SQL injection prevention',
    '✓ XSS prevention',
    '✓ CSRF protection',
    '✓ Authentication implemented',
    '✓ Authorization checks',
    '✓ Secrets management',
    '✓ Regular security audits'
  ],
  
  performance: [
    '✓ Caching implemented',
    '✓ Database queries optimized',
    '✓ Indexes created',
    '✓ Connection pooling configured',
    '✓ Code splitting',
    '✓ Image optimization',
    '✓ Gzip compression',
    '✓ CDN for static assets',
    '✓ Load testing completed'
  ],
  
  reliability: [
    '✓ Error handling comprehensive',
    '✓ Logging configured',
    '✓ Monitoring setup',
    '✓ Health checks implemented',
    '✓ Graceful shutdown',
    '✓ Circuit breakers',
    '✓ Retry logic',
    '✓ Backup strategy',
    '✓ Disaster recovery plan'
  ],
  
  operations: [
    '✓ CI/CD pipeline',
    '✓ Automated tests',
    '✓ Docker images',
    '✓ Kubernetes configs',
    '✓ Environment variables',
    '✓ Documentation',
    '✓ Runbooks',
    '✓ On-call rotation'
  ]
}
```

---

## Conclusion of Part 8 and Complete Series

This concludes Part 8 and the **Complete TypeScript Mastery series**!

### What We've Covered in Part 8:

1. **Docker Containerization** - Multi-stage builds, optimization
2. **Kubernetes Deployment** - Pods, services, scaling
3. **CI/CD Pipelines** - GitHub Actions, GitLab CI
4. **Logging & Monitoring** - Winston, Sentry, Prometheus
5. **Performance Optimization** - Code splitting, caching
6. **Caching Strategies** - Redis, cache patterns
7. **Database Optimization** - Indexing, pooling, migrations
8. **Security Hardening** - Rate limiting, validation, HTTPS
9. **Load Balancing** - Nginx, horizontal scaling
10. **Disaster Recovery** - Backups, health checks

---

## 🎓 Complete TypeScript Mastery Series Summary

### All Eight Parts:

| Part | Focus | Words |
|------|-------|-------|
| Part 1 | TypeScript Fundamentals | 25,231 |
| Part 2 | Advanced Type System | 19,179 |
| Part 3 | Compiler & Tooling | 6,966 |
| Part 4 | React & Node.js | 10,949 |
| Part 5 | Design Patterns | 6,968 |
| Part 6 | Framework Ecosystem | 7,950 |
| Part 7 | Real-World Applications | 8,969 |
| Part 8 | Production & Operations | 10,000+ |
| **Total** | **Complete Mastery** | **~96,000** |

**You now have the most comprehensive TypeScript course ever created!** 🎉🚀