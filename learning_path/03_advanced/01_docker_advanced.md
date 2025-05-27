# Docker Advanced: Production and Optimization

## 1. Multi-Stage Builds

### 1.1 Optimized Build Example
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 1.2 Benefits
- Smaller final image
- Separation of build and runtime
- Better security
- Optimized layers

## 2. Docker Security

### 2.1 Security Best Practices
```dockerfile
# Use specific versions
FROM node:16-alpine

# Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Minimize installed packages
RUN apk add --no-cache package-name

# Use COPY instead of ADD
COPY --chown=appuser:appgroup . .
```

### 2.2 Container Security
```bash
# Scan for vulnerabilities
docker scan myapp:1.0

# Run with limited capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE

# Enable read-only root filesystem
docker run --read-only
```

## 3. Performance Optimization

### 3.1 Resource Limits
```bash
# Memory limits
docker run --memory=512m --memory-swap=1g

# CPU limits
docker run --cpus=0.5 --cpu-shares=512

# Block I/O limits
docker run --device-write-bps=/dev/sda:1mb
```

### 3.2 Monitoring
```bash
# Container stats
docker stats

# Health checks
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

## 4. Production Deployment

### 4.1 Docker Registry
```bash
# Push to registry
docker tag myapp:1.0 registry.example.com/myapp:1.0
docker push registry.example.com/myapp:1.0

# Pull from private registry
docker login registry.example.com
docker pull registry.example.com/myapp:1.0
```

### 4.2 Load Balancing
```yaml
# Docker Compose with load balancing
version: '3.8'
services:
  web:
    image: myapp:1.0
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

## Advanced Projects:
1. Set up a secure multi-stage build pipeline
2. Implement container health monitoring
3. Deploy to production with rolling updates
4. Create a private Docker registry

## Next Steps:
- Learn container orchestration with Kubernetes
- Study advanced networking concepts
- Explore CI/CD integration
- Master Docker Swarm for clustering 