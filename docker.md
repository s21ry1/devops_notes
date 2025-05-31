# Docker Complete Guide

## 1. Core Concepts

### 1.1 Container Fundamentals
- Lightweight, standalone packages
- Contains everything needed to run an application:
  * Code
  * Runtime
  * System tools
  * System libraries
  * Settings
- Shares the host OS kernel
- Isolated from other containers

### 1.2 Docker Architecture
```
Client (docker CLI) → Docker Daemon → Container Runtime
                   ↓
            Docker Registry
                   ↓
         Images & Containers
```

### 1.3 Key Components
1. Docker Daemon (dockerd)
   - Manages Docker objects
   - Handles container lifecycle
   - API requests processing

2. Docker Client (docker)
   - Command-line interface
   - Communicates with Docker daemon
   - Handles user commands

3. Docker Registry
   - Stores Docker images
   - Public (Docker Hub)
   - Private registries

## 2. Essential Commands

### 2.1 Container Lifecycle
```bash
# Run a container
docker run -d -p 8080:80 --name webserver nginx

# List containers
docker ps                  # Running containers
docker ps -a              # All containers

# Container operations
docker start container_id
docker stop container_id
docker restart container_id
docker rm container_id
docker logs -f container_id
docker exec -it container_id /bin/bash

# Resource limits
docker run -d --memory=512m --cpus=0.5 nginx
```

### 2.2 Image Management
```bash
# Image operations
docker pull nginx:latest
docker images
docker rmi image_id
docker build -t myapp:1.0 .
docker tag myapp:1.0 registry.example.com/myapp:1.0
docker push registry.example.com/myapp:1.0

# Save and load images
docker save myapp:1.0 > myapp.tar
docker load < myapp.tar
```

## 3. Dockerfile Best Practices

### 3.1 Basic Structure
```dockerfile
# Use specific version
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package files first (better caching)
COPY package*.json ./
RUN npm install

# Copy application code
COPY . .

# Build application
RUN npm run build

# Set environment variables
ENV NODE_ENV=production

# Expose port
EXPOSE 3000

# Start command
CMD ["npm", "start"]
```

### 3.2 Multi-stage Builds
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

## 4. Docker Compose

### 4.1 Basic Structure
```yaml
version: '3.8'
services:
  web:
    build: ./web
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    depends_on:
      - api
      - db

  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    volumes:
      - api-logs:/var/log
    depends_on:
      - db

  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password

volumes:
  api-logs:
  db-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## 5. Networking

### 5.1 Network Types
```bash
# Create network
docker network create --driver bridge my-network

# List networks
docker network ls

# Connect container to network
docker network connect my-network container_id

# Inspect network
docker network inspect my-network
```

### 5.2 Network Drivers
1. Bridge (default)
   - Isolated network on host
   - Containers can communicate
   - Port mapping for external access

2. Host
   - Uses host network directly
   - Better performance
   - Less isolation

3. Overlay
   - Multi-host networking
   - Swarm mode communication
   - Container-to-container across nodes

4. Macvlan
   - Assign MAC address to container
   - Direct network access
   - Physical network integration

## 6. Real-World Use Cases

### 6.1 Microservices Application
```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    environment:
      - API_URL=http://api:3000
    depends_on:
      - api

  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
    depends_on:
      - db
      - redis
    secrets:
      - jwt_secret

  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password

  redis:
    image: redis:alpine
    volumes:
      - redis-data:/data

  monitoring:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana

volumes:
  db-data:
  redis-data:
  grafana-data:

secrets:
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  db_password:
    file: ./secrets/db_password.txt
```

### 6.2 CI/CD Pipeline Integration
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Test') {
            steps {
                sh "docker-compose -f docker-compose.test.yml up --exit-code-from tests"
            }
        }
        
        stage('Push') {
            steps {
                sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
        
        stage('Deploy') {
            steps {
                sh "docker-compose -f docker-compose.prod.yml up -d"
            }
        }
    }
}
```

### 6.3 Development Environment
```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run dev

  db:
    image: postgres:13
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=devdb
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=devpass
    volumes:
      - dev-db-data:/var/lib/postgresql/data

volumes:
  dev-db-data:
```

## 7. Security Best Practices

### 7.1 Image Security
```dockerfile
# Use specific versions
FROM node:16-alpine

# Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Scan for vulnerabilities
# Use: docker scan myapp:1.0
```

### 7.2 Runtime Security
```yaml
# docker-compose.yml with security options
services:
  app:
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

### 7.3 Secret Management
```yaml
services:
  app:
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
      - ssl_cert

secrets:
  db_password:
    external: true
  ssl_cert:
    file: ./secrets/ssl_cert.pem
```

## 8. Monitoring and Logging

### 8.1 Container Monitoring
```yaml
# docker-compose.monitoring.yml
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
```

### 8.2 Logging Configuration
```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## 9. Performance Optimization

### 9.1 Image Optimization
```dockerfile
# Use multi-stage builds
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:16-alpine
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/main.js"]
```

### 9.2 Resource Management
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

## 10. Troubleshooting Guide

### 10.1 Common Issues
```bash
# Check container logs
docker logs container_id

# Inspect container
docker inspect container_id

# Check resource usage
docker stats

# Debug container
docker exec -it container_id /bin/sh
```

### 10.2 Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
``` 
