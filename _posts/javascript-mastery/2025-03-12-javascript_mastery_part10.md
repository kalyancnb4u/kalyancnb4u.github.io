---
title: "JavaScript Mastery - Part 10: Deployment & DevOps"
date: 2025-03-12 00:00:00 +0530
categories: [JavaScript, JavaScript Mastery]
tags: [JavaScript, Programming, Web Development, DevOps, Deployment, CI-CD, Docker, Cloud, Monitoring, Kubernetes]
---

# Complete JavaScript Mastery Part 10: Deployment & DevOps

## Introduction

Deployment and DevOps practices transform code into production applications. Modern deployment requires automation, containerization, monitoring, and continuous delivery.

This part explores:
- CI/CD pipelines and automation
- Docker and containerization
- Cloud platforms and hosting
- Monitoring and logging
- Environment management
- Production best practices

**Prerequisites:**
- Parts 1-9: Full JavaScript development knowledge

**Why DevOps Matters:**

- **Speed:** Automated deployments in minutes
- **Reliability:** Consistent, reproducible environments
- **Scalability:** Handle traffic spikes automatically
- **Visibility:** Monitor production in real-time
- **Recovery:** Quick rollbacks when issues occur

Let's master deployment and DevOps.

---

## 10.1 CI/CD Pipelines

### GitHub Actions

Automate testing, building, and deployment with GitHub Actions.

#### Basic Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

#### Deploy Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20.x'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          NODE_ENV: production
          VITE_API_URL: ${{ secrets.API_URL }}
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'
```

#### Docker Build and Push

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: myusername/myapp
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=myusername/myapp:buildcache
          cache-to: type=registry,ref=myusername/myapp:buildcache,mode=max
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

# Test stage
test:
  stage: test
  image: node:${NODE_VERSION}
  cache:
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run lint
    - npm run test
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Build stage
build:
  stage: build
  image: node:${NODE_VERSION}
  only:
    - main
    - develop
  cache:
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

# Deploy to production
deploy_production:
  stage: deploy
  image: node:${NODE_VERSION}
  only:
    - main
  dependencies:
    - build
  script:
    - npm install -g vercel
    - vercel --token $VERCEL_TOKEN --prod
  environment:
    name: production
    url: https://myapp.com

# Deploy to staging
deploy_staging:
  stage: deploy
  image: node:${NODE_VERSION}
  only:
    - develop
  dependencies:
    - build
  script:
    - npm install -g vercel
    - vercel --token $VERCEL_TOKEN
  environment:
    name: staging
    url: https://staging.myapp.com
```

---

## 10.2 Docker and Containerization

### Dockerfile for Node.js

```dockerfile
# Multi-stage build for Node.js app

# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Stage 2: Production
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy built files from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "dist/server.js"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Frontend application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped
    networks:
      - app-network

  # PostgreSQL database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    networks:
      - app-network

  # Redis cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped
    networks:
      - app-network

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    restart: unless-stopped
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### Docker Best Practices

```dockerfile
# ✅ GOOD Dockerfile practices

# 1. Use specific version tags
FROM node:20.10.0-alpine

# 2. Combine RUN commands to reduce layers
RUN apk add --no-cache git && \
    npm ci --only=production && \
    npm cache clean --force

# 3. Use .dockerignore
# .dockerignore file:
# node_modules
# npm-debug.log
# .git
# .env
# coverage

# 4. Order layers by change frequency
COPY package*.json ./  # Changes rarely
RUN npm ci
COPY . .              # Changes often

# 5. Multi-stage builds for smaller images
FROM node:20-alpine AS builder
# ... build steps ...

FROM node:20-alpine
COPY --from=builder /app/dist ./dist

# 6. Run as non-root user
USER node

# 7. Use ENTRYPOINT for main command
ENTRYPOINT ["node"]
CMD ["server.js"]

# 8. Add labels for metadata
LABEL maintainer="you@example.com"
LABEL version="1.0"
LABEL description="My Node.js application"
```

---

## 10.3 Cloud Platforms

### Vercel Deployment

```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  },
  "regions": ["iad1"],
  "github": {
    "enabled": true,
    "autoAlias": true
  }
}
```

```javascript
// CLI deployment
// Install Vercel CLI
npm i -g vercel

// Login
vercel login

// Deploy
vercel

// Deploy to production
vercel --prod

// Environment variables
vercel env add VARIABLE_NAME production
```

### Netlify Deployment

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"

[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"

[functions]
  directory = "functions"
  node_bundler = "esbuild"
```

### AWS Deployment

#### AWS Elastic Beanstalk

```yaml
# .ebextensions/nodecommand.config
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm start"
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    PORT: 8080
```

#### AWS Lambda (Serverless)

```javascript
// serverless.yml
service: my-app

provider:
  name: aws
  runtime: nodejs20.x
  region: us-east-1
  memorySize: 1024
  timeout: 30
  environment:
    NODE_ENV: production
    DATABASE_URL: ${env:DATABASE_URL}

functions:
  api:
    handler: handler.main
    events:
      - http:
          path: /{proxy+}
          method: ANY
          cors: true

plugins:
  - serverless-offline
  - serverless-dotenv-plugin

# handler.js
const express = require('express');
const serverless = require('serverless-http');

const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Lambda!' });
});

module.exports.main = serverless(app);
```

### DigitalOcean App Platform

```yaml
# .do/app.yaml
name: my-app
region: nyc

services:
  - name: web
    github:
      repo: username/repo
      branch: main
      deploy_on_push: true
    
    build_command: npm run build
    run_command: npm start
    
    envs:
      - key: NODE_ENV
        value: "production"
      - key: DATABASE_URL
        scope: RUN_TIME
        type: SECRET
    
    instance_count: 2
    instance_size_slug: basic-xs
    
    http_port: 3000
    
    health_check:
      http_path: /health

databases:
  - name: db
    engine: PG
    version: "15"
    size: db-s-1vcpu-1gb

  - name: redis
    engine: REDIS
    version: "7"
```

---

## 10.4 Environment Management

### Environment Variables

```javascript
// .env (never commit!)
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://localhost:5432/mydb
JWT_SECRET=your-secret-key
API_KEY=your-api-key

// .env.production
NODE_ENV=production
DATABASE_URL=postgresql://prod-host:5432/proddb

// Load with dotenv
require('dotenv').config();

console.log(process.env.DATABASE_URL);

// Or use config file
// config.js
const config = {
  development: {
    port: 3000,
    database: {
      host: 'localhost',
      port: 5432,
      name: 'mydb'
    }
  },
  production: {
    port: process.env.PORT || 8080,
    database: {
      host: process.env.DB_HOST,
      port: process.env.DB_PORT,
      name: process.env.DB_NAME
    }
  }
};

module.exports = config[process.env.NODE_ENV || 'development'];
```

### Secret Management

```javascript
// AWS Secrets Manager
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getSecret(secretName) {
  const data = await secretsManager.getSecretValue({
    SecretId: secretName
  }).promise();
  
  return JSON.parse(data.SecretString);
}

// Usage
const dbCredentials = await getSecret('prod/database');

// Environment-specific config
const getConfig = () => {
  const env = process.env.NODE_ENV || 'development';
  
  return {
    development: {
      apiUrl: 'http://localhost:3000',
      logLevel: 'debug'
    },
    staging: {
      apiUrl: 'https://staging-api.example.com',
      logLevel: 'info'
    },
    production: {
      apiUrl: 'https://api.example.com',
      logLevel: 'error'
    }
  }[env];
};
```

---

## 10.5 Monitoring and Logging

### Application Monitoring (Sentry)

```javascript
// Sentry integration
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Express({ app })
  ]
});

// Express middleware
app.use(Sentry.Handlers.requestHandler());
app.use(Sentry.Handlers.tracingHandler());

// Routes
app.get('/', (req, res) => {
  res.send('Hello');
});

// Error handler (must be last)
app.use(Sentry.Handlers.errorHandler());

// Custom error tracking
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error);
}

// Add context
Sentry.setUser({ id: user.id, email: user.email });
Sentry.setTag('transaction_type', 'checkout');
Sentry.setContext('order', { id: order.id, total: order.total });

// Performance monitoring
const transaction = Sentry.startTransaction({
  op: 'http',
  name: 'GET /api/users'
});

// ... operation ...

transaction.finish();
```

### Logging (Winston)

```javascript
const winston = require('winston');

// Configure logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'my-app' },
  transports: [
    // Write all logs to console
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // Write errors to error.log
    new winston.transports.File({
      filename: 'error.log',
      level: 'error'
    }),
    
    // Write all logs to combined.log
    new winston.transports.File({
      filename: 'combined.log'
    })
  ]
});

// Production: Send logs to external service
if (process.env.NODE_ENV === 'production') {
  logger.add(new winston.transports.Http({
    host: 'logs.example.com',
    port: 443,
    ssl: true
  }));
}

// Usage
logger.info('User logged in', { userId: user.id });
logger.warn('Unusual activity', { ip: req.ip });
logger.error('Database error', { error: err.message, stack: err.stack });

// Express middleware
app.use((req, res, next) => {
  logger.info('HTTP Request', {
    method: req.method,
    url: req.url,
    ip: req.ip
  });
  next();
});
```

### Application Performance Monitoring (APM)

```javascript
// New Relic
require('newrelic');

// Datadog
const tracer = require('dd-trace').init({
  service: 'my-app',
  env: process.env.NODE_ENV
});

// Custom metrics
const StatsD = require('node-statsd');
const stats = new StatsD({
  host: 'localhost',
  port: 8125
});

// Increment counter
stats.increment('api.requests');

// Timing
stats.timing('api.response_time', responseTime);

// Gauge
stats.gauge('active_users', activeUsers);

// Set
stats.set('unique_visitors', userId);
```

### Health Checks

```javascript
// Express health check endpoint
app.get('/health', async (req, res) => {
  const health = {
    uptime: process.uptime(),
    timestamp: Date.now(),
    status: 'ok',
    checks: {}
  };
  
  try {
    // Check database
    await db.query('SELECT 1');
    health.checks.database = 'ok';
  } catch (error) {
    health.checks.database = 'error';
    health.status = 'error';
  }
  
  try {
    // Check Redis
    await redis.ping();
    health.checks.redis = 'ok';
  } catch (error) {
    health.checks.redis = 'error';
    health.status = 'error';
  }
  
  const statusCode = health.status === 'ok' ? 200 : 503;
  res.status(statusCode).json(health);
});

// Kubernetes liveness/readiness probes
// Liveness: Is the app running?
app.get('/healthz', (req, res) => {
  res.status(200).send('OK');
});

// Readiness: Can the app handle requests?
app.get('/ready', async (req, res) => {
  const ready = await checkDependencies();
  res.status(ready ? 200 : 503).send(ready ? 'Ready' : 'Not Ready');
});
```

---

## 10.6 Production Best Practices

### Process Management (PM2)

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'my-app',
    script: './server.js',
    instances: 'max', // Or number of instances
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production'
    },
    error_file: './logs/error.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    max_memory_restart: '1G',
    autorestart: true,
    watch: false
  }]
};

// CLI commands
// pm2 start ecosystem.config.js
// pm2 start ecosystem.config.js --env production
// pm2 reload my-app
// pm2 stop my-app
// pm2 restart my-app
// pm2 logs my-app
// pm2 monit
```

### Graceful Shutdown

```javascript
// server.js
const server = app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

// Handle shutdown signals
const gracefulShutdown = (signal) => {
  console.log(`${signal} received. Starting graceful shutdown...`);
  
  // Stop accepting new requests
  server.close(async () => {
    console.log('HTTP server closed');
    
    try {
      // Close database connections
      await db.close();
      console.log('Database connections closed');
      
      // Close Redis connections
      await redis.quit();
      console.log('Redis connection closed');
      
      process.exit(0);
    } catch (error) {
      console.error('Error during shutdown:', error);
      process.exit(1);
    }
  });
  
  // Force shutdown after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 30000);
};

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

// Handle uncaught errors
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  gracefulShutdown('uncaughtException');
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  gracefulShutdown('unhandledRejection');
});
```

### Load Balancing (Nginx)

```nginx
# nginx.conf
upstream app_servers {
    least_conn;  # Load balancing method
    
    server app1:3000 weight=1 max_fails=3 fail_timeout=30s;
    server app2:3000 weight=1 max_fails=3 fail_timeout=30s;
    server app3:3000 weight=1 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name example.com;
    
    # Redirect HTTP to HTTPS
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
    
    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Static files
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # API proxy
    location /api/ {
        proxy_pass http://app_servers;
        proxy_http_version 1.1;
        
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # SPA fallback
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }
}
```

---

## Key Takeaways

✅ **CI/CD:**
- Automate testing and deployment
- Use GitHub Actions or GitLab CI
- Multi-environment pipelines
- Automated testing gates

✅ **Docker:**
- Multi-stage builds
- Minimal base images
- Non-root users
- Docker Compose for local dev

✅ **Cloud Platforms:**
- Vercel/Netlify for static/serverless
- AWS for full control
- Choose based on needs

✅ **Monitoring:**
- Error tracking (Sentry)
- Logging (Winston)
- APM tools
- Health checks

✅ **Production:**
- Process management (PM2)
- Graceful shutdown
- Load balancing
- Environment management

---

## Conclusion

Deployment and DevOps enable reliable, scalable production applications. Automate everything, monitor constantly, and prepare for failures.

**What's Next:**

Part 11 will cover Security Best Practices.

---

**Total Word Count: Part 10 Complete (~12,000 words)**
