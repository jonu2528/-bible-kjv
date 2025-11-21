# Part 8: Deployment & DevOps

## Table of Contents

1. [Overview](#1-overview)
2. [Infrastructure Architecture](#2-infrastructure-architecture)
3. [Docker Configuration](#3-docker-configuration)
4. [CI/CD Pipeline](#4-cicd-pipeline)
5. [Cloud Hosting Options](#5-cloud-hosting-options)
6. [Database Hosting](#6-database-hosting)
7. [CDN Configuration](#7-cdn-configuration)
8. [Environment Configuration](#8-environment-configuration)
9. [Monitoring & Logging](#9-monitoring--logging)
10. [Security Best Practices](#10-security-best-practices)
11. [Backup & Recovery](#11-backup--recovery)
12. [Scaling Strategy](#12-scaling-strategy)

---

## 1. Overview

### 1.1 Deployment Goals

- **Zero-downtime deployments** - Users experience no interruption during updates
- **Automated pipeline** - From code commit to production with minimal manual intervention
- **Environment parity** - Development, staging, and production environments are consistent
- **Scalability** - Architecture supports horizontal and vertical scaling
- **Security** - Industry-standard security practices at all levels
- **Observability** - Comprehensive monitoring, logging, and alerting

### 1.2 Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                      PRODUCTION STACK                           │
├─────────────────────────────────────────────────────────────────┤
│  CDN Layer:        CloudFlare / AWS CloudFront                  │
│  Frontend:         Vercel / Netlify / AWS S3 + CloudFront       │
│  Backend:          AWS ECS / Google Cloud Run / DigitalOcean    │
│  Database:         AWS RDS PostgreSQL / PlanetScale / Supabase  │
│  Cache:            Redis (AWS ElastiCache / Upstash)            │
│  File Storage:     AWS S3 / Cloudflare R2                       │
│  Push Service:     Firebase Cloud Messaging                     │
│  Monitoring:       DataDog / New Relic / Grafana Cloud          │
│  CI/CD:            GitHub Actions / GitLab CI                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Infrastructure Architecture

### 2.1 High-Level Architecture Diagram

```
                                    ┌─────────────────┐
                                    │   CloudFlare    │
                                    │      CDN        │
                                    └────────┬────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
                    ▼                        ▼                        ▼
           ┌───────────────┐        ┌───────────────┐        ┌───────────────┐
           │   Frontend    │        │   Frontend    │        │   Frontend    │
           │   (Vercel)    │        │   (Vercel)    │        │   (Vercel)    │
           │   Edge Node   │        │   Edge Node   │        │   Edge Node   │
           └───────────────┘        └───────────────┘        └───────────────┘
                    │                        │                        │
                    └────────────────────────┼────────────────────────┘
                                             │
                                    ┌────────▼────────┐
                                    │  Load Balancer  │
                                    │   (AWS ALB)     │
                                    └────────┬────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
                    ▼                        ▼                        ▼
           ┌───────────────┐        ┌───────────────┐        ┌───────────────┐
           │   Backend     │        │   Backend     │        │   Backend     │
           │   API (ECS)   │        │   API (ECS)   │        │   API (ECS)   │
           │   Instance 1  │        │   Instance 2  │        │   Instance 3  │
           └───────┬───────┘        └───────┬───────┘        └───────┬───────┘
                   │                        │                        │
                   └────────────────────────┼────────────────────────┘
                                            │
                         ┌──────────────────┼──────────────────┐
                         │                  │                  │
                         ▼                  ▼                  ▼
                ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
                │  PostgreSQL │    │    Redis    │    │     S3      │
                │  (RDS)      │    │ (ElastiCache│    │   Bucket    │
                │  Primary    │    │   Cluster)  │    │             │
                └──────┬──────┘    └─────────────┘    └─────────────┘
                       │
                       ▼
                ┌─────────────┐
                │  PostgreSQL │
                │  (RDS)      │
                │  Read Replica│
                └─────────────┘
```

### 2.2 Network Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                            VPC (10.0.0.0/16)                           │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                    Public Subnet (10.0.1.0/24)                   │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │  │
│  │  │ NAT Gateway  │  │ Load Balancer│  │   Bastion    │            │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘            │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                   Private Subnet A (10.0.10.0/24)                │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │  │
│  │  │  ECS Task 1  │  │  ECS Task 2  │  │  ECS Task 3  │            │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘            │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                   Private Subnet B (10.0.20.0/24)                │  │
│  │  ┌──────────────┐  ┌──────────────┐                              │  │
│  │  │ RDS Primary  │  │ ElastiCache  │                              │  │
│  │  └──────────────┘  └──────────────┘                              │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Docker Configuration

### 3.1 Backend Dockerfile

```dockerfile
# Dockerfile.backend
# Multi-stage build for optimized production image

# ============================================
# Stage 1: Dependencies
# ============================================
FROM node:20-alpine AS deps

WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install dependencies
RUN npm ci --only=production

# ============================================
# Stage 2: Builder
# ============================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build the application
RUN npm run build

# ============================================
# Stage 3: Production
# ============================================
FROM node:20-alpine AS production

# Add non-root user for security
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

# Set environment
ENV NODE_ENV=production
ENV PORT=3000

# Copy necessary files from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start the application
CMD ["node", "dist/main.js"]
```

### 3.2 Frontend Dockerfile

```dockerfile
# Dockerfile.frontend
# Multi-stage build for Next.js application

# ============================================
# Stage 1: Dependencies
# ============================================
FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

# ============================================
# Stage 2: Builder
# ============================================
FROM node:20-alpine AS builder

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build arguments for environment variables
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_GA_ID
ARG NEXT_PUBLIC_VAPID_PUBLIC_KEY

ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_GA_ID=$NEXT_PUBLIC_GA_ID
ENV NEXT_PUBLIC_VAPID_PUBLIC_KEY=$NEXT_PUBLIC_VAPID_PUBLIC_KEY

# Disable telemetry
ENV NEXT_TELEMETRY_DISABLED=1

RUN npm run build

# ============================================
# Stage 3: Production
# ============================================
FROM node:20-alpine AS production

WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Copy built files
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json

# Copy Next.js build output
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

### 3.3 Docker Compose for Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  # ==========================================
  # Frontend Application
  # ==========================================
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000/api
      - WATCHPACK_POLLING=true
    depends_on:
      - backend

  # ==========================================
  # Backend API
  # ==========================================
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://bible_user:bible_pass@postgres:5432/bible_db
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=dev_jwt_secret_key_change_in_production
      - JWT_REFRESH_SECRET=dev_jwt_refresh_secret_change_in_production
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  # ==========================================
  # PostgreSQL Database
  # ==========================================
  postgres:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=bible_user
      - POSTGRES_PASSWORD=bible_pass
      - POSTGRES_DB=bible_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U bible_user -d bible_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ==========================================
  # Redis Cache
  # ==========================================
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ==========================================
  # Adminer (Database Management)
  # ==========================================
  adminer:
    image: adminer:latest
    ports:
      - "8080:8080"
    depends_on:
      - postgres

  # ==========================================
  # Redis Commander (Redis Management)
  # ==========================================
  redis-commander:
    image: rediscommander/redis-commander:latest
    ports:
      - "8081:8081"
    environment:
      - REDIS_HOSTS=local:redis:6379
    depends_on:
      - redis

volumes:
  postgres_data:
  redis_data:
```

### 3.4 Docker Compose for Production

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  frontend:
    image: ${DOCKER_REGISTRY}/bible-frontend:${VERSION:-latest}
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        max_attempts: 3
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  backend:
    image: ${DOCKER_REGISTRY}/bible-backend:${VERSION:-latest}
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - JWT_SECRET=${JWT_SECRET}
      - JWT_REFRESH_SECRET=${JWT_REFRESH_SECRET}
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        max_attempts: 3
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:4000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

## 4. CI/CD Pipeline

### 4.1 GitHub Actions - Main Workflow

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '20'
  DOCKER_REGISTRY: ghcr.io/${{ github.repository_owner }}

jobs:
  # ==========================================
  # Code Quality & Testing
  # ==========================================
  quality:
    name: Code Quality
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

      - name: Run linting
        run: npm run lint

      - name: Run type checking
        run: npm run type-check

      - name: Run unit tests
        run: npm run test:unit -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info

  # ==========================================
  # Security Scanning
  # ==========================================
  security:
    name: Security Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'

      - name: Run npm audit
        run: npm audit --production

  # ==========================================
  # Build Docker Images
  # ==========================================
  build:
    name: Build Images
    runs-on: ubuntu-latest
    needs: [quality, security]
    if: github.event_name == 'push'

    strategy:
      matrix:
        include:
          - name: frontend
            context: ./frontend
            dockerfile: Dockerfile.frontend
          - name: backend
            context: ./backend
            dockerfile: Dockerfile.backend

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.DOCKER_REGISTRY }}/bible-${{ matrix.name }}
          tags: |
            type=ref,event=branch
            type=sha,prefix=
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: ${{ matrix.context }}
          file: ${{ matrix.context }}/${{ matrix.dockerfile }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            NEXT_PUBLIC_API_URL=${{ vars.NEXT_PUBLIC_API_URL }}
            NEXT_PUBLIC_GA_ID=${{ vars.NEXT_PUBLIC_GA_ID }}

  # ==========================================
  # Deploy to Staging
  # ==========================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster bible-staging \
            --service bible-api \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster bible-staging \
            --services bible-api

      - name: Run smoke tests
        run: |
          curl -f https://staging-api.biblereader.app/health || exit 1

  # ==========================================
  # Deploy to Production
  # ==========================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Deploy to ECS (Blue/Green)
        run: |
          aws deploy create-deployment \
            --application-name bible-app \
            --deployment-group-name bible-production \
            --revision revisionType=AppSpecContent,appSpecContent='{
              "version": 1,
              "Resources": [{
                "TargetService": {
                  "Type": "AWS::ECS::Service",
                  "Properties": {
                    "TaskDefinition": "arn:aws:ecs:${{ vars.AWS_REGION }}:${{ secrets.AWS_ACCOUNT_ID }}:task-definition/bible-api:${{ github.sha }}",
                    "LoadBalancerInfo": {
                      "ContainerName": "bible-api",
                      "ContainerPort": 4000
                    }
                  }
                }
              }]
            }'

      - name: Notify deployment
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Production deployment completed",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "✅ *Production Deployment Successful*\nCommit: ${{ github.sha }}\nBy: ${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 4.2 Database Migration Workflow

```yaml
# .github/workflows/migrations.yml
name: Database Migrations

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to run migrations'
        required: true
        type: choice
        options:
          - staging
          - production
      action:
        description: 'Migration action'
        required: true
        type: choice
        options:
          - migrate
          - rollback
          - status

jobs:
  migrate:
    name: Run Migrations
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci
        working-directory: ./backend

      - name: Run migration
        run: |
          case "${{ github.event.inputs.action }}" in
            migrate)
              npm run db:migrate
              ;;
            rollback)
              npm run db:rollback
              ;;
            status)
              npm run db:status
              ;;
          esac
        working-directory: ./backend
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

### 4.3 Vercel Configuration (Frontend)

```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "https://api.biblereader.app/api/$1",
      "headers": {
        "x-forwarded-host": "biblereader.app"
      }
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    },
    {
      "source": "/sw.js",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=0, must-revalidate"
        },
        {
          "key": "Service-Worker-Allowed",
          "value": "/"
        }
      ]
    }
  ],
  "env": {
    "NEXT_PUBLIC_API_URL": "@next_public_api_url",
    "NEXT_PUBLIC_GA_ID": "@next_public_ga_id",
    "NEXT_PUBLIC_VAPID_PUBLIC_KEY": "@next_public_vapid_public_key"
  }
}
```

---

## 5. Cloud Hosting Options

### 5.1 Option A: AWS (Recommended for Scale)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         AWS Architecture                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Route 53 (DNS)                                                         │
│       │                                                                 │
│       ▼                                                                 │
│  CloudFront (CDN) ─────────────────────┐                                │
│       │                                │                                │
│       ▼                                ▼                                │
│  S3 (Static Assets)              ALB (Load Balancer)                    │
│                                        │                                │
│                           ┌────────────┼────────────┐                   │
│                           │            │            │                   │
│                           ▼            ▼            ▼                   │
│                     ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│                     │ECS Fargate│ │ECS Fargate│ │ECS Fargate│            │
│                     │ Task 1   │ │ Task 2   │ │ Task 3   │              │
│                     └────┬─────┘ └────┬─────┘ └────┬─────┘              │
│                          └────────────┼────────────┘                    │
│                                       │                                 │
│                    ┌──────────────────┼──────────────────┐              │
│                    │                  │                  │              │
│                    ▼                  ▼                  ▼              │
│              ┌──────────┐      ┌──────────┐      ┌──────────┐           │
│              │   RDS    │      │ElastiCache│      │    S3    │          │
│              │PostgreSQL│      │  Redis   │      │ Storage  │           │
│              └──────────┘      └──────────┘      └──────────┘           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Monthly Cost Estimate (Medium Scale):
├── ECS Fargate (3 tasks, 0.5vCPU, 1GB): ~$50/month
├── RDS PostgreSQL (db.t3.medium): ~$60/month
├── ElastiCache Redis (cache.t3.micro): ~$15/month
├── ALB: ~$25/month
├── CloudFront: ~$10/month (depends on traffic)
├── S3: ~$5/month
├── Route 53: ~$1/month
└── Total: ~$166/month
```

### 5.2 Option B: Vercel + PlanetScale (Simple & Cost-Effective)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Vercel + PlanetScale Architecture                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Vercel Edge Network (CDN + Hosting)                                    │
│       │                                                                 │
│       ├──► Static Assets (Edge Cached)                                  │
│       │                                                                 │
│       ├──► Next.js App (Serverless Functions)                           │
│       │         │                                                       │
│       │         ▼                                                       │
│       │    API Routes (/api/*)                                          │
│       │         │                                                       │
│       │         ├──────────────────┬──────────────────┐                 │
│       │         │                  │                  │                 │
│       │         ▼                  ▼                  ▼                 │
│       │   ┌──────────┐      ┌──────────┐      ┌──────────┐              │
│       │   │PlanetScale│      │  Upstash │      │Cloudflare│             │
│       │   │  MySQL   │      │  Redis   │      │    R2    │              │
│       │   └──────────┘      └──────────┘      └──────────┘              │
│       │                                                                 │
│       └──► Edge Functions (for auth, redirects)                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Monthly Cost Estimate:
├── Vercel Pro: $20/month
├── PlanetScale Scaler: $29/month
├── Upstash Redis: $10/month
├── Cloudflare R2: ~$5/month
└── Total: ~$64/month
```

### 5.3 Option C: DigitalOcean (Budget-Friendly)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DigitalOcean Architecture                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  CloudFlare (CDN + DDoS Protection)                                     │
│       │                                                                 │
│       ▼                                                                 │
│  DigitalOcean Load Balancer                                             │
│       │                                                                 │
│       ├──────────────────┬──────────────────┐                           │
│       │                  │                  │                           │
│       ▼                  ▼                  ▼                           │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐                       │
│  │ Droplet  │      │ Droplet  │      │ Droplet  │                       │
│  │(App + API)│      │(App + API)│      │(App + API)│                     │
│  │ $24/mo   │      │ $24/mo   │      │ $24/mo   │                       │
│  └────┬─────┘      └────┬─────┘      └────┬─────┘                       │
│       └──────────────────┼──────────────────┘                           │
│                          │                                              │
│           ┌──────────────┼──────────────┐                               │
│           │              │              │                               │
│           ▼              ▼              ▼                               │
│     ┌──────────┐   ┌──────────┐   ┌──────────┐                          │
│     │ Managed  │   │ Managed  │   │  Spaces  │                          │
│     │PostgreSQL│   │  Redis   │   │ (S3-like)│                          │
│     │ $15/mo   │   │ $15/mo   │   │  $5/mo   │                          │
│     └──────────┘   └──────────┘   └──────────┘                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Monthly Cost Estimate:
├── 3x Droplets (Basic, 2vCPU, 4GB): ~$72/month
├── Load Balancer: $12/month
├── Managed PostgreSQL (Basic): $15/month
├── Managed Redis: $15/month
├── Spaces (250GB): $5/month
├── CloudFlare (Free tier): $0/month
└── Total: ~$119/month
```

---

## 6. Database Hosting

### 6.1 AWS RDS Configuration

```hcl
# terraform/rds.tf

resource "aws_db_instance" "bible_db" {
  identifier     = "bible-db-production"
  engine         = "postgres"
  engine_version = "16.1"
  instance_class = "db.t3.medium"

  allocated_storage     = 100
  max_allocated_storage = 500
  storage_type          = "gp3"
  storage_encrypted     = true

  db_name  = "bible_db"
  username = var.db_username
  password = var.db_password

  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.bible.name

  # High Availability
  multi_az = true

  # Backup Configuration
  backup_retention_period = 30
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"

  # Performance Insights
  performance_insights_enabled          = true
  performance_insights_retention_period = 7

  # Monitoring
  monitoring_interval = 60
  monitoring_role_arn = aws_iam_role.rds_monitoring.arn

  # Deletion Protection
  deletion_protection = true
  skip_final_snapshot = false
  final_snapshot_identifier = "bible-db-final-snapshot"

  # Parameter Group
  parameter_group_name = aws_db_parameter_group.bible.name

  tags = {
    Name        = "bible-db-production"
    Environment = "production"
  }
}

resource "aws_db_parameter_group" "bible" {
  family = "postgres16"
  name   = "bible-db-params"

  parameter {
    name  = "log_statement"
    value = "ddl"
  }

  parameter {
    name  = "log_min_duration_statement"
    value = "1000"  # Log queries > 1 second
  }

  parameter {
    name  = "shared_preload_libraries"
    value = "pg_stat_statements"
  }
}

# Read Replica for read-heavy operations
resource "aws_db_instance" "bible_db_replica" {
  identifier          = "bible-db-replica"
  replicate_source_db = aws_db_instance.bible_db.identifier
  instance_class      = "db.t3.medium"

  vpc_security_group_ids = [aws_security_group.rds.id]

  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn        = aws_iam_role.rds_monitoring.arn

  tags = {
    Name        = "bible-db-replica"
    Environment = "production"
  }
}
```

### 6.2 Database Connection Pool

```typescript
// backend/src/config/database.ts
import { Pool, PoolConfig } from 'pg';

const poolConfig: PoolConfig = {
  connectionString: process.env.DATABASE_URL,

  // Connection Pool Settings
  max: 20,                    // Maximum connections
  min: 5,                     // Minimum connections
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 10000, // Connection timeout

  // SSL Configuration
  ssl: process.env.NODE_ENV === 'production'
    ? { rejectUnauthorized: true }
    : false,
};

// Primary pool for write operations
export const primaryPool = new Pool({
  ...poolConfig,
  connectionString: process.env.DATABASE_URL,
});

// Replica pool for read operations
export const replicaPool = new Pool({
  ...poolConfig,
  connectionString: process.env.DATABASE_REPLICA_URL || process.env.DATABASE_URL,
});

// Health check
export async function checkDatabaseHealth(): Promise<boolean> {
  try {
    await primaryPool.query('SELECT 1');
    return true;
  } catch (error) {
    console.error('Database health check failed:', error);
    return false;
  }
}

// Graceful shutdown
export async function closePools(): Promise<void> {
  await primaryPool.end();
  await replicaPool.end();
}
```

---

## 7. CDN Configuration

### 7.1 CloudFlare Configuration

```javascript
// cloudflare/worker.js - Edge Worker for custom logic

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  const url = new URL(request.url);

  // Cache static Bible content aggressively
  if (url.pathname.startsWith('/api/bible/')) {
    const cache = caches.default;
    let response = await cache.match(request);

    if (!response) {
      response = await fetch(request);

      // Cache Bible content for 1 week
      const cacheResponse = new Response(response.body, response);
      cacheResponse.headers.set('Cache-Control', 'public, max-age=604800');
      event.waitUntil(cache.put(request, cacheResponse.clone()));

      return cacheResponse;
    }

    return response;
  }

  // Add security headers
  const response = await fetch(request);
  const newResponse = new Response(response.body, response);

  newResponse.headers.set('X-Content-Type-Options', 'nosniff');
  newResponse.headers.set('X-Frame-Options', 'DENY');
  newResponse.headers.set('X-XSS-Protection', '1; mode=block');
  newResponse.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');

  return newResponse;
}
```

### 7.2 CloudFlare Page Rules

```yaml
# cloudflare/page_rules.yml

rules:
  # Cache static assets aggressively
  - url: "*.biblereader.app/static/*"
    settings:
      cache_level: "cache_everything"
      edge_cache_ttl: 2592000  # 30 days
      browser_cache_ttl: 2592000

  # Cache Bible content
  - url: "*.biblereader.app/api/bible/*"
    settings:
      cache_level: "cache_everything"
      edge_cache_ttl: 604800  # 7 days

  # Don't cache user-specific data
  - url: "*.biblereader.app/api/user/*"
    settings:
      cache_level: "bypass"

  # Don't cache authentication
  - url: "*.biblereader.app/api/auth/*"
    settings:
      cache_level: "bypass"
      security_level: "high"

  # PWA manifest and service worker
  - url: "*.biblereader.app/manifest.json"
    settings:
      cache_level: "cache_everything"
      edge_cache_ttl: 86400  # 1 day

  - url: "*.biblereader.app/sw.js"
    settings:
      cache_level: "standard"
      edge_cache_ttl: 0  # Always revalidate
```

---

## 8. Environment Configuration

### 8.1 Environment Variables Structure

```bash
# .env.example

# ==========================================
# Application
# ==========================================
NODE_ENV=development
PORT=4000
API_VERSION=v1

# ==========================================
# Database
# ==========================================
DATABASE_URL=postgresql://user:password@localhost:5432/bible_db
DATABASE_REPLICA_URL=postgresql://user:password@localhost:5432/bible_db
DATABASE_POOL_SIZE=20

# ==========================================
# Redis
# ==========================================
REDIS_URL=redis://localhost:6379
REDIS_PREFIX=bible:

# ==========================================
# Authentication
# ==========================================
JWT_SECRET=your-super-secret-jwt-key-min-32-chars
JWT_REFRESH_SECRET=your-super-secret-refresh-key-min-32-chars
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# OAuth Providers
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
FACEBOOK_APP_ID=your-facebook-app-id
FACEBOOK_APP_SECRET=your-facebook-app-secret
APPLE_CLIENT_ID=your-apple-client-id
APPLE_TEAM_ID=your-apple-team-id
APPLE_KEY_ID=your-apple-key-id

# ==========================================
# Push Notifications
# ==========================================
VAPID_PUBLIC_KEY=your-vapid-public-key
VAPID_PRIVATE_KEY=your-vapid-private-key
VAPID_SUBJECT=mailto:admin@biblereader.app

FIREBASE_PROJECT_ID=your-firebase-project-id
FIREBASE_PRIVATE_KEY=your-firebase-private-key
FIREBASE_CLIENT_EMAIL=your-firebase-client-email

# ==========================================
# Storage
# ==========================================
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_REGION=us-east-1
S3_BUCKET=bible-app-storage

# ==========================================
# Monitoring
# ==========================================
SENTRY_DSN=https://xxx@sentry.io/xxx
DATADOG_API_KEY=your-datadog-api-key
LOG_LEVEL=info

# ==========================================
# Feature Flags
# ==========================================
ENABLE_TTS=true
ENABLE_PUSH_NOTIFICATIONS=true
ENABLE_SOCIAL_LOGIN=true

# ==========================================
# Rate Limiting
# ==========================================
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100

# ==========================================
# CORS
# ==========================================
CORS_ORIGINS=http://localhost:3000,https://biblereader.app
```

### 8.2 AWS Secrets Manager

```typescript
// backend/src/config/secrets.ts
import {
  SecretsManagerClient,
  GetSecretValueCommand,
} from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: process.env.AWS_REGION });

interface AppSecrets {
  DATABASE_URL: string;
  JWT_SECRET: string;
  JWT_REFRESH_SECRET: string;
  VAPID_PRIVATE_KEY: string;
  GOOGLE_CLIENT_SECRET: string;
  FACEBOOK_APP_SECRET: string;
}

let cachedSecrets: AppSecrets | null = null;

export async function getSecrets(): Promise<AppSecrets> {
  if (cachedSecrets) {
    return cachedSecrets;
  }

  const secretName = `bible-app/${process.env.NODE_ENV}/secrets`;

  try {
    const command = new GetSecretValueCommand({ SecretId: secretName });
    const response = await client.send(command);

    if (response.SecretString) {
      cachedSecrets = JSON.parse(response.SecretString);
      return cachedSecrets!;
    }

    throw new Error('Secret value is empty');
  } catch (error) {
    console.error('Failed to retrieve secrets:', error);
    throw error;
  }
}

// Initialize secrets at startup
export async function initializeSecrets(): Promise<void> {
  if (process.env.NODE_ENV === 'production') {
    const secrets = await getSecrets();

    // Override environment variables with secrets
    process.env.DATABASE_URL = secrets.DATABASE_URL;
    process.env.JWT_SECRET = secrets.JWT_SECRET;
    process.env.JWT_REFRESH_SECRET = secrets.JWT_REFRESH_SECRET;
    process.env.VAPID_PRIVATE_KEY = secrets.VAPID_PRIVATE_KEY;
    process.env.GOOGLE_CLIENT_SECRET = secrets.GOOGLE_CLIENT_SECRET;
    process.env.FACEBOOK_APP_SECRET = secrets.FACEBOOK_APP_SECRET;
  }
}
```

---

## 9. Monitoring & Logging

### 9.1 Structured Logging

```typescript
// backend/src/utils/logger.ts
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',

  formatters: {
    level: (label) => ({ level: label }),
  },

  base: {
    service: 'bible-api',
    version: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV,
  },

  timestamp: () => `,"timestamp":"${new Date().toISOString()}"`,

  // Production: JSON format for log aggregation
  // Development: Pretty print
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined,
});

// Request logger middleware
export function requestLogger(req: Request, res: Response, next: NextFunction) {
  const startTime = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - startTime;

    logger.info({
      type: 'request',
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration,
      userAgent: req.headers['user-agent'],
      ip: req.ip,
      userId: req.user?.id,
    });
  });

  next();
}

export default logger;
```

### 9.2 Application Performance Monitoring (APM)

```typescript
// backend/src/monitoring/apm.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

export function initializeAPM() {
  if (process.env.NODE_ENV === 'production' && process.env.SENTRY_DSN) {
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
      release: process.env.APP_VERSION,

      integrations: [
        new ProfilingIntegration(),
      ],

      // Performance Monitoring
      tracesSampleRate: 0.1, // 10% of transactions
      profilesSampleRate: 0.1,

      // Error Filtering
      beforeSend(event) {
        // Don't send 404 errors
        if (event.exception?.values?.[0]?.type === 'NotFoundError') {
          return null;
        }
        return event;
      },
    });
  }
}

// Error handler middleware
export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  Sentry.captureException(err, {
    user: req.user ? { id: req.user.id, email: req.user.email } : undefined,
    extra: {
      url: req.url,
      method: req.method,
      body: req.body,
    },
  });

  next(err);
}
```

### 9.3 Health Check Endpoints

```typescript
// backend/src/routes/health.ts
import { Router } from 'express';
import { primaryPool, replicaPool } from '../config/database';
import { redisClient } from '../config/redis';

const router = Router();

interface HealthStatus {
  status: 'healthy' | 'unhealthy' | 'degraded';
  timestamp: string;
  version: string;
  uptime: number;
  checks: {
    database: { status: string; latency?: number };
    databaseReplica: { status: string; latency?: number };
    redis: { status: string; latency?: number };
    memory: { status: string; used: number; total: number };
  };
}

// Basic health check (for load balancer)
router.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// Detailed health check
router.get('/health/detailed', async (req, res) => {
  const health: HealthStatus = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || '1.0.0',
    uptime: process.uptime(),
    checks: {
      database: { status: 'unknown' },
      databaseReplica: { status: 'unknown' },
      redis: { status: 'unknown' },
      memory: { status: 'unknown', used: 0, total: 0 },
    },
  };

  // Check primary database
  try {
    const start = Date.now();
    await primaryPool.query('SELECT 1');
    health.checks.database = {
      status: 'healthy',
      latency: Date.now() - start
    };
  } catch (error) {
    health.checks.database = { status: 'unhealthy' };
    health.status = 'unhealthy';
  }

  // Check replica database
  try {
    const start = Date.now();
    await replicaPool.query('SELECT 1');
    health.checks.databaseReplica = {
      status: 'healthy',
      latency: Date.now() - start
    };
  } catch (error) {
    health.checks.databaseReplica = { status: 'unhealthy' };
    if (health.status === 'healthy') health.status = 'degraded';
  }

  // Check Redis
  try {
    const start = Date.now();
    await redisClient.ping();
    health.checks.redis = {
      status: 'healthy',
      latency: Date.now() - start
    };
  } catch (error) {
    health.checks.redis = { status: 'unhealthy' };
    if (health.status === 'healthy') health.status = 'degraded';
  }

  // Check memory
  const memUsage = process.memoryUsage();
  const memUsedMB = Math.round(memUsage.heapUsed / 1024 / 1024);
  const memTotalMB = Math.round(memUsage.heapTotal / 1024 / 1024);
  const memPercent = (memUsedMB / memTotalMB) * 100;

  health.checks.memory = {
    status: memPercent < 90 ? 'healthy' : 'warning',
    used: memUsedMB,
    total: memTotalMB,
  };

  const statusCode = health.status === 'healthy' ? 200 :
                     health.status === 'degraded' ? 200 : 503;

  res.status(statusCode).json(health);
});

// Readiness check (for Kubernetes)
router.get('/ready', async (req, res) => {
  try {
    await primaryPool.query('SELECT 1');
    res.status(200).json({ ready: true });
  } catch (error) {
    res.status(503).json({ ready: false });
  }
});

// Liveness check (for Kubernetes)
router.get('/live', (req, res) => {
  res.status(200).json({ alive: true });
});

export default router;
```

### 9.4 Metrics Collection

```typescript
// backend/src/monitoring/metrics.ts
import client from 'prom-client';

// Create a Registry
const register = new client.Registry();

// Add default metrics
client.collectDefaultMetrics({ register });

// Custom metrics
export const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5],
});

export const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

export const activeConnections = new client.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

export const databaseQueryDuration = new client.Histogram({
  name: 'database_query_duration_seconds',
  help: 'Duration of database queries in seconds',
  labelNames: ['operation', 'table'],
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1],
});

export const cacheHitRate = new client.Gauge({
  name: 'cache_hit_rate',
  help: 'Cache hit rate percentage',
  labelNames: ['cache_type'],
});

// Register custom metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(databaseQueryDuration);
register.registerMetric(cacheHitRate);

// Metrics endpoint handler
export async function metricsHandler(req: Request, res: Response) {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
}
```

### 9.5 Alerting Rules (Prometheus/AlertManager)

```yaml
# monitoring/alerts.yml
groups:
  - name: bible-app-alerts
    rules:
      # High Error Rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is above 5% for the last 5 minutes"

      # High Latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
          ) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is above 2 seconds"

      # Database Connection Issues
      - alert: DatabaseConnectionError
        expr: pg_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Database connection failed"
          description: "Cannot connect to PostgreSQL database"

      # Redis Connection Issues
      - alert: RedisConnectionError
        expr: redis_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Redis connection failed"
          description: "Cannot connect to Redis"

      # High Memory Usage
      - alert: HighMemoryUsage
        expr: |
          (process_resident_memory_bytes / 1024 / 1024) > 1024
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is above 1GB"

      # Pod Restart
      - alert: PodRestarting
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 3
        labels:
          severity: warning
        annotations:
          summary: "Pod is restarting frequently"
          description: "Pod has restarted more than 3 times in the last hour"
```

---

## 10. Security Best Practices

### 10.1 Security Headers Configuration

```typescript
// backend/src/middleware/security.ts
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import cors from 'cors';

// Helmet configuration for security headers
export const securityHeaders = helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", "https://www.googletagmanager.com"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      imgSrc: ["'self'", "data:", "https:"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      connectSrc: ["'self'", "https://api.biblereader.app", "wss:"],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  crossOriginEmbedderPolicy: false,
  crossOriginOpenerPolicy: { policy: 'same-origin-allow-popups' },
  crossOriginResourcePolicy: { policy: 'cross-origin' },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
});

// CORS configuration
export const corsOptions = cors({
  origin: (origin, callback) => {
    const allowedOrigins = process.env.CORS_ORIGINS?.split(',') || [];

    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  exposedHeaders: ['X-Total-Count', 'X-Page', 'X-Per-Page'],
  maxAge: 86400, // 24 hours
});

// Rate limiting
export const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
  message: {
    error: 'Too many requests',
    message: 'Please try again later',
    retryAfter: 60,
  },
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => {
    return req.user?.id || req.ip;
  },
});

// Stricter rate limit for auth endpoints
export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // 10 attempts per 15 minutes
  message: {
    error: 'Too many authentication attempts',
    message: 'Please try again in 15 minutes',
    retryAfter: 900,
  },
});
```

### 10.2 Input Validation & Sanitization

```typescript
// backend/src/middleware/validation.ts
import { z } from 'zod';
import xss from 'xss';

// XSS sanitization options
const xssOptions = {
  whiteList: {},
  stripIgnoreTag: true,
  stripIgnoreTagBody: ['script'],
};

// Sanitize string input
export function sanitizeString(input: string): string {
  return xss(input.trim(), xssOptions);
}

// Common validation schemas
export const schemas = {
  // User registration
  register: z.object({
    email: z.string().email().max(255).transform(sanitizeString),
    password: z.string()
      .min(8, 'Password must be at least 8 characters')
      .max(128)
      .regex(/[A-Z]/, 'Password must contain uppercase letter')
      .regex(/[a-z]/, 'Password must contain lowercase letter')
      .regex(/[0-9]/, 'Password must contain a number'),
    displayName: z.string().min(1).max(100).transform(sanitizeString),
  }),

  // Login
  login: z.object({
    email: z.string().email().transform(sanitizeString),
    password: z.string().min(1),
  }),

  // Note creation
  createNote: z.object({
    verseId: z.string().uuid(),
    content: z.string().min(1).max(10000).transform(sanitizeString),
  }),

  // Highlight creation
  createHighlight: z.object({
    verseId: z.string().uuid(),
    color: z.enum(['yellow', 'green', 'blue', 'pink', 'orange']),
    startOffset: z.number().int().min(0).optional(),
    endOffset: z.number().int().min(0).optional(),
  }),

  // Search query
  search: z.object({
    query: z.string().min(2).max(200).transform(sanitizeString),
    versionId: z.string().uuid().optional(),
    testament: z.enum(['old', 'new', 'all']).optional(),
    limit: z.number().int().min(1).max(100).default(20),
    offset: z.number().int().min(0).default(0),
  }),

  // Pagination
  pagination: z.object({
    page: z.coerce.number().int().min(1).default(1),
    limit: z.coerce.number().int().min(1).max(100).default(20),
  }),
};

// Validation middleware factory
export function validate<T>(schema: z.ZodSchema<T>) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      const data = schema.parse({
        ...req.body,
        ...req.query,
        ...req.params,
      });
      req.validated = data;
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        res.status(400).json({
          error: 'Validation Error',
          details: error.errors.map(e => ({
            field: e.path.join('.'),
            message: e.message,
          })),
        });
      } else {
        next(error);
      }
    }
  };
}
```

### 10.3 SQL Injection Prevention

```typescript
// backend/src/utils/database.ts
import { Pool, QueryConfig, QueryResult } from 'pg';

// Always use parameterized queries
export async function query<T = any>(
  pool: Pool,
  text: string,
  params?: any[]
): Promise<QueryResult<T>> {
  // Validate that query uses parameters for user input
  if (params === undefined && text.includes('$')) {
    throw new Error('Query expects parameters but none provided');
  }

  const queryConfig: QueryConfig = {
    text,
    values: params,
  };

  return pool.query<T>(queryConfig);
}

// Example safe queries
export const queries = {
  // GOOD: Parameterized query
  getVerseById: `
    SELECT * FROM verses WHERE id = $1
  `,

  // GOOD: Parameterized search with LIKE
  searchVerses: `
    SELECT * FROM verses
    WHERE to_tsvector('english', text) @@ plainto_tsquery('english', $1)
    LIMIT $2 OFFSET $3
  `,

  // GOOD: Parameterized IN clause
  getVersesByIds: (count: number) => `
    SELECT * FROM verses WHERE id = ANY($1::uuid[])
  `,
};

// NEVER do this:
// const badQuery = `SELECT * FROM users WHERE email = '${email}'`; // SQL Injection vulnerability!
```

### 10.4 Secrets Management

```yaml
# kubernetes/secrets.yml
apiVersion: v1
kind: Secret
metadata:
  name: bible-app-secrets
  namespace: production
type: Opaque
stringData:
  # These should be encrypted and managed via:
  # - AWS Secrets Manager
  # - HashiCorp Vault
  # - Kubernetes External Secrets Operator

  # Database credentials managed by AWS Secrets Manager
  database-url: "ENC[AES256_GCM,data:...,type:str]"

  # JWT secrets (auto-rotated monthly)
  jwt-secret: "ENC[AES256_GCM,data:...,type:str]"
  jwt-refresh-secret: "ENC[AES256_GCM,data:...,type:str]"

  # OAuth secrets
  google-client-secret: "ENC[AES256_GCM,data:...,type:str]"
  facebook-app-secret: "ENC[AES256_GCM,data:...,type:str]"

  # Push notification keys
  vapid-private-key: "ENC[AES256_GCM,data:...,type:str]"
```

---

## 11. Backup & Recovery

### 11.1 Database Backup Strategy

```bash
#!/bin/bash
# scripts/backup-database.sh

set -e

# Configuration
BACKUP_BUCKET="s3://bible-app-backups"
RETENTION_DAYS=30
DB_HOST="${DATABASE_HOST}"
DB_NAME="${DATABASE_NAME}"
DB_USER="${DATABASE_USER}"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="bible_db_${TIMESTAMP}.sql.gz"

# Create backup
echo "Starting database backup..."
PGPASSWORD="${DATABASE_PASSWORD}" pg_dump \
  -h "$DB_HOST" \
  -U "$DB_USER" \
  -d "$DB_NAME" \
  --format=custom \
  --compress=9 \
  --file="/tmp/${BACKUP_FILE}"

# Upload to S3
echo "Uploading backup to S3..."
aws s3 cp "/tmp/${BACKUP_FILE}" "${BACKUP_BUCKET}/daily/${BACKUP_FILE}" \
  --storage-class STANDARD_IA \
  --sse AES256

# Cleanup local file
rm "/tmp/${BACKUP_FILE}"

# Delete old backups (older than retention period)
echo "Cleaning up old backups..."
aws s3 ls "${BACKUP_BUCKET}/daily/" | while read -r line; do
  file_date=$(echo "$line" | awk '{print $1}')
  file_name=$(echo "$line" | awk '{print $4}')

  if [[ $(date -d "$file_date" +%s) -lt $(date -d "-${RETENTION_DAYS} days" +%s) ]]; then
    aws s3 rm "${BACKUP_BUCKET}/daily/${file_name}"
    echo "Deleted old backup: ${file_name}"
  fi
done

echo "Backup completed successfully!"
```

### 11.2 Backup Schedule (CronJob)

```yaml
# kubernetes/backup-cronjob.yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
  namespace: production
spec:
  # Run daily at 2 AM UTC
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3

  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure

          containers:
            - name: backup
              image: postgres:16-alpine
              command: ["/scripts/backup-database.sh"]

              envFrom:
                - secretRef:
                    name: database-credentials

              env:
                - name: AWS_REGION
                  value: "us-east-1"

              volumeMounts:
                - name: scripts
                  mountPath: /scripts

          volumes:
            - name: scripts
              configMap:
                name: backup-scripts
                defaultMode: 0755
```

### 11.3 Recovery Procedures

```bash
#!/bin/bash
# scripts/restore-database.sh

set -e

# Configuration
BACKUP_BUCKET="s3://bible-app-backups"
DB_HOST="${DATABASE_HOST}"
DB_NAME="${DATABASE_NAME}"
DB_USER="${DATABASE_USER}"

# Usage
if [ -z "$1" ]; then
  echo "Usage: $0 <backup-file-name>"
  echo "Example: $0 bible_db_20250114_020000.sql.gz"
  exit 1
fi

BACKUP_FILE="$1"

echo "==================================="
echo "DATABASE RESTORE PROCEDURE"
echo "==================================="
echo "Backup file: ${BACKUP_FILE}"
echo "Target database: ${DB_NAME}"
echo ""
echo "WARNING: This will overwrite the current database!"
read -p "Are you sure? (yes/no): " confirm

if [ "$confirm" != "yes" ]; then
  echo "Restore cancelled."
  exit 0
fi

# Download backup from S3
echo "Downloading backup from S3..."
aws s3 cp "${BACKUP_BUCKET}/daily/${BACKUP_FILE}" "/tmp/${BACKUP_FILE}"

# Terminate existing connections
echo "Terminating existing database connections..."
PGPASSWORD="${DATABASE_PASSWORD}" psql \
  -h "$DB_HOST" \
  -U "$DB_USER" \
  -d postgres \
  -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '${DB_NAME}' AND pid <> pg_backend_pid();"

# Restore database
echo "Restoring database..."
PGPASSWORD="${DATABASE_PASSWORD}" pg_restore \
  -h "$DB_HOST" \
  -U "$DB_USER" \
  -d "$DB_NAME" \
  --clean \
  --if-exists \
  "/tmp/${BACKUP_FILE}"

# Cleanup
rm "/tmp/${BACKUP_FILE}"

echo "Database restore completed successfully!"
```

### 11.4 Disaster Recovery Plan

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DISASTER RECOVERY PLAN                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Recovery Objectives:                                                   │
│  ├── RPO (Recovery Point Objective): 1 hour                             │
│  └── RTO (Recovery Time Objective): 4 hours                             │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Backup Schedule:                                                       │
│  ├── Continuous: Transaction logs (Point-in-time recovery)              │
│  ├── Hourly: Database snapshots (RDS automated)                         │
│  ├── Daily: Full database dump to S3                                    │
│  ├── Weekly: Full backup with verification                              │
│  └── Monthly: Cross-region backup replication                           │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Recovery Procedures:                                                   │
│                                                                         │
│  Level 1 - Minor (Single service failure):                              │
│  ├── Auto-healing via ECS/Kubernetes                                    │
│  ├── Automatic failover to healthy instances                            │
│  └── ETA: < 5 minutes                                                   │
│                                                                         │
│  Level 2 - Moderate (Database failover):                                │
│  ├── Automatic failover to RDS replica                                  │
│  ├── DNS update via Route 53 health checks                              │
│  └── ETA: < 15 minutes                                                  │
│                                                                         │
│  Level 3 - Major (AZ failure):                                          │
│  ├── Traffic routed to healthy AZ                                       │
│  ├── New instances launched automatically                               │
│  └── ETA: < 30 minutes                                                  │
│                                                                         │
│  Level 4 - Critical (Region failure):                                   │
│  ├── Activate DR site in secondary region                               │
│  ├── Restore from cross-region backups                                  │
│  ├── Update DNS to DR site                                              │
│  └── ETA: < 4 hours                                                     │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Contact Escalation:                                                    │
│  ├── On-call Engineer: PagerDuty rotation                               │
│  ├── Engineering Lead: After 15 minutes                                 │
│  ├── CTO: After 30 minutes (Level 3+)                                   │
│  └── CEO: After 1 hour (Level 4)                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 12. Scaling Strategy

### 12.1 Horizontal Scaling (ECS Auto Scaling)

```hcl
# terraform/autoscaling.tf

resource "aws_appautoscaling_target" "ecs_target" {
  max_capacity       = 10
  min_capacity       = 2
  resource_id        = "service/${aws_ecs_cluster.bible.name}/${aws_ecs_service.api.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

# Scale based on CPU utilization
resource "aws_appautoscaling_policy" "cpu_scaling" {
  name               = "cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value       = 70.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# Scale based on request count
resource "aws_appautoscaling_policy" "request_scaling" {
  name               = "request-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ALBRequestCountPerTarget"
      resource_label         = "${aws_lb.bible.arn_suffix}/${aws_lb_target_group.api.arn_suffix}"
    }
    target_value       = 1000.0  # 1000 requests per target
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# Scheduled scaling for known traffic patterns
resource "aws_appautoscaling_scheduled_action" "morning_scale_up" {
  name               = "morning-scale-up"
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  schedule           = "cron(0 6 * * ? *)"  # 6 AM UTC daily

  scalable_target_action {
    min_capacity = 4
    max_capacity = 10
  }
}

resource "aws_appautoscaling_scheduled_action" "night_scale_down" {
  name               = "night-scale-down"
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  schedule           = "cron(0 22 * * ? *)"  # 10 PM UTC daily

  scalable_target_action {
    min_capacity = 2
    max_capacity = 6
  }
}
```

### 12.2 Database Scaling Strategy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DATABASE SCALING STRATEGY                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Phase 1: Vertical Scaling (0-10K DAU)                                  │
│  ├── Instance: db.t3.medium → db.r6g.large                              │
│  ├── Storage: 100GB → 500GB (auto-scaling enabled)                      │
│  └── Connection pool: 20 → 50 connections                               │
│                                                                         │
│  Phase 2: Read Replicas (10K-50K DAU)                                   │
│  ├── Add 1-2 read replicas                                              │
│  ├── Route read queries to replicas                                     │
│  ├── Keep writes on primary                                             │
│  └── Implement connection routing in application                        │
│                                                                         │
│  Phase 3: Caching Layer (50K-100K DAU)                                  │
│  ├── Implement Redis caching for:                                       │
│  │   ├── Bible content (cache-first strategy)                           │
│  │   ├── User sessions                                                  │
│  │   ├── Search results                                                 │
│  │   └── Quote of the day                                               │
│  ├── Cache hit ratio target: >90%                                       │
│  └── Reduce database load by 70%                                        │
│                                                                         │
│  Phase 4: Sharding (100K+ DAU)                                          │
│  ├── Shard user data by user_id                                         │
│  ├── Keep Bible content on dedicated cluster                            │
│  ├── Use consistent hashing for shard routing                           │
│  └── Consider CitusDB or Vitess for managed sharding                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 12.3 Cache Scaling

```typescript
// backend/src/cache/redis-cluster.ts
import { Cluster } from 'ioredis';

const redisCluster = new Cluster(
  [
    { host: 'redis-1.example.com', port: 6379 },
    { host: 'redis-2.example.com', port: 6379 },
    { host: 'redis-3.example.com', port: 6379 },
  ],
  {
    redisOptions: {
      password: process.env.REDIS_PASSWORD,
      tls: process.env.NODE_ENV === 'production' ? {} : undefined,
    },
    scaleReads: 'slave', // Read from replicas
    maxRedirections: 16,
    retryDelayOnFailover: 100,
    retryDelayOnClusterDown: 100,
  }
);

// Cache-aside pattern with automatic scaling
export class CacheService {
  private readonly defaultTTL = 3600; // 1 hour

  async get<T>(key: string): Promise<T | null> {
    const cached = await redisCluster.get(key);
    if (cached) {
      return JSON.parse(cached);
    }
    return null;
  }

  async set<T>(key: string, value: T, ttl?: number): Promise<void> {
    await redisCluster.set(
      key,
      JSON.stringify(value),
      'EX',
      ttl || this.defaultTTL
    );
  }

  async getOrSet<T>(
    key: string,
    factory: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached !== null) {
      return cached;
    }

    const value = await factory();
    await this.set(key, value, ttl);
    return value;
  }

  async invalidate(pattern: string): Promise<void> {
    const keys = await redisCluster.keys(pattern);
    if (keys.length > 0) {
      await redisCluster.del(...keys);
    }
  }
}
```

### 12.4 Load Testing Configuration

```yaml
# k6/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const searchDuration = new Trend('search_duration');

export const options = {
  stages: [
    // Warm up
    { duration: '2m', target: 50 },
    // Ramp up
    { duration: '5m', target: 200 },
    // Sustained load
    { duration: '10m', target: 200 },
    // Spike test
    { duration: '2m', target: 500 },
    // Recovery
    { duration: '5m', target: 200 },
    // Ramp down
    { duration: '2m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    errors: ['rate<0.01'], // Error rate < 1%
    search_duration: ['p(95)<1000'],
  },
};

const BASE_URL = __ENV.API_URL || 'https://api.biblereader.app';

// Bible books for random selection
const books = ['genesis', 'exodus', 'psalms', 'proverbs', 'matthew', 'john', 'romans', 'revelation'];

export default function () {
  // Mix of different request types
  const scenario = Math.random();

  if (scenario < 0.4) {
    // 40% - Read Bible content
    const book = books[Math.floor(Math.random() * books.length)];
    const chapter = Math.floor(Math.random() * 20) + 1;

    const res = http.get(`${BASE_URL}/api/v1/bible/kjv/${book}/${chapter}`);
    check(res, { 'chapter loaded': (r) => r.status === 200 });
    errorRate.add(res.status !== 200);

  } else if (scenario < 0.6) {
    // 20% - Search
    const queries = ['love', 'faith', 'hope', 'grace', 'peace', 'truth'];
    const query = queries[Math.floor(Math.random() * queries.length)];

    const start = Date.now();
    const res = http.get(`${BASE_URL}/api/v1/search?q=${query}&limit=20`);
    searchDuration.add(Date.now() - start);
    check(res, { 'search completed': (r) => r.status === 200 });
    errorRate.add(res.status !== 200);

  } else if (scenario < 0.8) {
    // 20% - Quote of the day
    const res = http.get(`${BASE_URL}/api/v1/quotes/today`);
    check(res, { 'quote loaded': (r) => r.status === 200 });
    errorRate.add(res.status !== 200);

  } else {
    // 20% - User operations (requires auth token)
    const authToken = __ENV.AUTH_TOKEN;
    if (authToken) {
      const headers = { Authorization: `Bearer ${authToken}` };
      const res = http.get(`${BASE_URL}/api/v1/user/highlights`, { headers });
      check(res, { 'highlights loaded': (r) => r.status === 200 });
      errorRate.add(res.status !== 200);
    }
  }

  // Simulate user think time
  sleep(Math.random() * 3 + 1);
}
```

---

## Summary

This deployment and DevOps specification provides a comprehensive guide for deploying the Online Bible Reader application at scale. Key highlights:

### Infrastructure
- Multi-tier architecture with CDN, load balancing, and auto-scaling
- Multiple hosting options (AWS, Vercel+PlanetScale, DigitalOcean)
- Containerized deployment with Docker

### CI/CD
- Automated testing, building, and deployment
- Blue/Green deployments for zero-downtime updates
- Environment-specific configurations

### Security
- Defense in depth with multiple security layers
- Secrets management and rotation
- Input validation and SQL injection prevention

### Monitoring
- Comprehensive logging and metrics
- APM integration with Sentry
- Alerting rules for proactive issue detection

### Reliability
- Automated backups with retention policies
- Disaster recovery procedures
- Multi-AZ deployment for high availability

### Scalability
- Horizontal auto-scaling based on metrics
- Database read replicas and caching
- Load testing configuration

---

**Document Version**: 1.0.0
**Last Updated**: 2025-01-14
**Author**: System Design Team
