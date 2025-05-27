# Docker Basics: Getting Started

## 1. What is Docker?
Docker is a platform that helps you package and run applications in containers. Think of containers as lightweight, portable boxes that contain everything your application needs to run.

### Key Benefits:
- Easy to share and move applications
- Works the same everywhere
- Uses less resources than virtual machines
- Quick to start and stop

## 2. First Steps with Docker

### 2.1 Basic Commands
```bash
# Check Docker version
docker --version

# Download your first image
docker pull hello-world

# Run your first container
docker run hello-world

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a
```

### 2.2 Working with Images
```bash
# List downloaded images
docker images

# Remove an image
docker rmi image_name

# Search for images
docker search ubuntu
```

## 3. Understanding Containers

### 3.1 Container Lifecycle
```bash
# Start a container
docker start container_name

# Stop a container
docker stop container_name

# Remove a container
docker rm container_name
```

### 3.2 Interactive Mode
```bash
# Run container with interactive terminal
docker run -it ubuntu bash

# Exit container
exit
```

## 4. Basic Docker Run Options
```bash
# Run container in background (detached)
docker run -d nginx

# Map container port to host
docker run -p 8080:80 nginx

# Give container a name
docker run --name my-web-server nginx
```

## Practice Exercises:
1. Install Docker on your system
2. Pull and run the hello-world image
3. Start an Ubuntu container and explore basic Linux commands
4. Run a web server (nginx) and access it through your browser
5. Learn to start, stop, and remove containers

## Next Steps:
- Learn about Dockerfile basics
- Understand container networking
- Explore volume mounting
- Study Docker Compose fundamentals 