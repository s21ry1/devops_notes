# Docker Intermediate: Building and Managing Applications

## 1. Writing Dockerfiles

### 1.1 Basic Dockerfile Structure
```dockerfile
# Start from a base image
FROM ubuntu:20.04

# Set working directory
WORKDIR /app

# Copy files
COPY . .

# Run commands
RUN apt-get update && \
    apt-get install -y python3

# Set environment variables
ENV PORT=3000

# Expose ports
EXPOSE 3000

# Define startup command
CMD ["python3", "app.py"]
```

### 1.2 Best Practices
- Use specific base image versions
- Combine RUN commands to reduce layers
- Remove unnecessary files
- Use .dockerignore file
- Keep images small and focused

## 2. Docker Compose

### 2.1 Basic docker-compose.yml
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
  
  database:
    image: mongo:latest
    volumes:
      - db-data:/data/db

volumes:
  db-data:
```

### 2.2 Common Commands
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs
```

## 3. Working with Volumes

### 3.1 Volume Types
1. Named Volumes
```bash
# Create volume
docker volume create my-data

# Use volume
docker run -v my-data:/app/data nginx
```

2. Bind Mounts
```bash
# Mount local directory
docker run -v $(pwd):/app nginx
```

## 4. Container Networking

### 4.1 Network Types
```bash
# Create network
docker network create my-network

# Run container in network
docker run --network my-network nginx

# Inspect network
docker network inspect my-network
```

### 4.2 Container Communication
- Containers in same network can communicate
- Use service names as hostnames
- Port mapping for external access

## Practice Projects:
1. Create a web application with database using Docker Compose
2. Set up development environment with volume mounting
3. Build and optimize a custom application image
4. Create a multi-container network setup

## Next Steps:
- Learn about multi-stage builds
- Study container orchestration
- Explore Docker security best practices
- Understand Docker Registry usage 