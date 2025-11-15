---
layout: post
title: "Docker Best Practices: Building Lean Production Images"
date: 2025-10-25 09:00:00 +0800
categories: devops docker containers
author: Sam Rodriguez
---

Our Docker images were bloated - 2GB+ base images causing slow deployments. Here's how we cut them down by 80%.

## The Problem

**Before**: Production image sizes averaging 2.1GB
- Node.js app: 1.8GB
- Python service: 2.3GB
- Go microservice: 1.2GB (Go! Should be tiny!)

**Deployment pain**:
- 15+ minutes to pull images
- Limited disk space on nodes
- Slow CI/CD pipelines

## Strategy 1: Multi-Stage Builds

**Bad approach**:
```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install  # Installs dev dependencies
RUN npm run build
CMD ["npm", "start"]
```

Result: 1.8GB (includes node_modules, build tools, source code)

**Good approach**:
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --production=false
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
CMD ["node", "dist/index.js"]
```

Result: 180MB (87% reduction)

## Strategy 2: Use Alpine Images

**Base image comparison**:
- `node:18` - 910MB
- `node:18-slim` - 240MB
- `node:18-alpine` - 120MB

Alpine is minimal Linux distro. Caveat: uses musl libc instead of glibc - can cause compatibility issues with native modules.

## Strategy 3: .dockerignore

Create `.dockerignore` like `.gitignore`:

```
node_modules
npm-debug.log
.git
.env
*.md
tests/
coverage/
.vscode/
```

Prevents copying unnecessary files into build context.

## Strategy 4: Layer Caching Optimization

**Order matters**:

```dockerfile
# Bad: Changes to source invalidate package install cache
COPY . .
RUN npm install

# Good: Package install cached unless package.json changes
COPY package*.json ./
RUN npm install
COPY . .
```

## Strategy 5: Distroless for Ultra-Lean Images

Google's distroless images contain only your app + runtime dependencies. No shell, no package manager.

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o app

# Production stage
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/app /app
CMD ["/app"]
```

Result for Go app: **2MB** (down from 1.2GB!)

## Strategy 6: Security Scanning

Use `trivy` to scan for vulnerabilities:

```bash
trivy image myapp:latest
```

We found:
- 127 vulnerabilities in `node:18` base image
- 12 vulnerabilities in `node:18-alpine`
- 0 vulnerabilities in distroless Go image

## Results

**Before vs After**:
- Node.js: 1.8GB → 180MB (90% reduction)
- Python: 2.3GB → 310MB (86% reduction)
- Go: 1.2GB → 2MB (99.8% reduction!)

**Impact**:
- Image pull time: 15min → 45sec
- CI/CD pipeline: 25min → 8min
- Disk usage on nodes: -67%

## Best Practices Checklist

- ✅ Use multi-stage builds
- ✅ Prefer Alpine or distroless base images
- ✅ Create comprehensive `.dockerignore`
- ✅ Optimize layer caching (copy package files first)
- ✅ Run security scans in CI
- ✅ Use specific version tags (not `latest`)
- ✅ Minimize layers (combine RUN commands)
- ✅ Remove build artifacts in same layer they're created

## Tools We Use

- **dive**: Analyze image layers
- **trivy**: Security scanning
- **hadolint**: Dockerfile linting
- **docker-slim**: Automated optimization (use with caution)

Lean images = faster deployments, better security, lower costs.

---

*Questions about Docker optimization? Ask away!*
