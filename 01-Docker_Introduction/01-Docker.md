# Docker Deep Dive

Docker is a platform for packaging applications and their dependencies into lightweight, portable units called **containers**. A container runs an application in an isolated environment, but unlike a virtual machine, it shares the host OS kernel. That makes containers fast to start, efficient, and ideal for modern application delivery.

---

## Understanding Docker Architecture

### Overview

Docker works by combining a few core building blocks:

- Docker Client
- Docker Daemon
- Docker Engine
- Images
- Containers
- Registries
- Volumes
- Networks

The main idea is simple:

- You build an image
- You run that image as a container
- Docker manages isolation, networking, storage, and lifecycle

### High-level architecture

    +--------------------------+
    |      Docker Client       |
    |      (CLI / API)         |
    +------------+-------------+
                 |
                 v
    +--------------------------+
    |      Docker Daemon       |
    |        (dockerd)         |
    |                          |
    | - builds images          |
    | - runs containers        |
    | - manages networks       |
    | - manages volumes        |
    +------------+-------------+
                 |
          +------+------+
          |             |
          v             v
    +-----------+  +----------------+
    |  Images   |  |   Containers   |
    |  Layers   |  | Running        |
    |           |  | processes      |
    +-----+-----+  +----------------+
          |
          v
    +--------------------------+
    |      Docker Registry     |
    |                          |
    | Docker Hub / ECR         |
    | GHCR / Nexus etc.        |
    +--------------------------+

### Core components

#### Docker Client

The Docker client is what you interact with using commands like:

    docker build
    docker run
    docker ps
    docker images

It sends requests to the Docker daemon.

#### Docker Daemon

The daemon (`dockerd`) is the background service that does the real work:

- builds images
- starts/stops containers
- creates networks
- manages volumes
- talks to registries

#### Docker Engine

Docker Engine is the runtime environment that includes:

- daemon
- REST API
- CLI support

#### Docker Images

An image is a read-only template used to create containers.

Think of it like:

- image = blueprint
- container = running instance of that blueprint

#### Docker Containers

A container is a running process created from an image.

It includes:

- application code
- runtime
- libraries
- environment variables
- isolated filesystem

But it shares the host kernel.

#### Docker Registry

A registry stores images.

Examples:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Nexus Registry

---

## How Docker works under the hood

Docker uses Linux kernel features such as:

- Namespaces for isolation
- cgroups for resource limits
- Union file systems for layered images

### Namespaces

Namespaces isolate what a container can see.

Examples:

- PID namespace → separate process view
- NET namespace → separate network stack
- MNT namespace → separate filesystem mount view
- UTS namespace → separate hostname
- IPC namespace → isolated inter-process communication

### cgroups

Control groups limit and track resources:

- CPU
- memory
- I/O

This is why you can run:

    docker run --memory=512m --cpus=1 nginx

### Layered filesystem

Docker images are built in layers.

Example:

- base OS layer
- language runtime layer
- dependencies layer
- app code layer

This improves:

- build speed
- caching
- reusability
- pull/push efficiency

### Docker lifecycle at a glance

    Write Dockerfile
           |
           v
      docker build
           |
           v
      Docker Image
           |
           v
       docker run
           |
           v
        Container
           |
           +----> docker stop
           |
           +----> docker restart
           |
           +----> docker rm

---

## Creating Your First Docker Image

### Overview

A Docker image is created using a Dockerfile, which is a text file containing instructions to build the image.

### Simple example: Python app

#### App file

    # app.py
    print("Hello from Docker")

#### Dockerfile

    FROM python:3.12-slim
    WORKDIR /app

    COPY app.py .

    CMD ["python", "app.py"]

### Build image

    docker build -t hello-docker .

### Run container

    docker run --rm hello-docker

Output:

    Hello from Docker

---

## Important Dockerfile instructions

### FROM

Defines the base image.

    FROM ubuntu:24.04

### WORKDIR

Sets the working directory inside the container.

    WORKDIR /app

### COPY

Copies files from host to image.

    COPY . .

### ADD

Similar to COPY, but has extra features like extracting archives. Usually prefer COPY for clarity.

### RUN

Executes commands during image build.

    RUN apt-get update && apt-get install -y curl

### CMD

Default command when container starts.

    CMD ["nginx", "-g", "daemon off;"]

### ENTRYPOINT

Defines the main executable for the container.

    ENTRYPOINT ["python"]
    CMD ["app.py"]

### EXPOSE

Documents which port the app listens on.

    EXPOSE 8080

### ENV

Sets environment variables.

    ENV APP_ENV=production

### ARG

Used during build time.

    ARG VERSION=1.0

---

## Image layers and caching

Each Dockerfile instruction usually creates a layer.

Example:

    FROM node:20-alpine
    WORKDIR /app
    COPY package*.json .
    RUN npm install
    COPY . .
    CMD ["npm", "start"]

Why this order matters:

- COPY package*.json . and RUN npm install happen before copying the rest of the source
- if app code changes but dependencies do not, Docker reuses the cached dependency layer

This speeds up builds a lot.

---

## Image Creation and Management

### Overview

Once you understand how to build images, the next step is managing them properly.

This includes:

- tagging
- versioning
- listing
- inspecting
- pushing/pulling
- cleaning unused artifacts

### Common image commands

#### List images

    docker images

#### Inspect image

    docker image inspect nginx:latest

#### Remove image

    docker rmi image_name

#### Tag image

    docker tag myapp:latest myrepo/myapp:v1

#### Push image

    docker push myrepo/myapp:v1

#### Pull image

    docker pull nginx:latest

### Tagging strategy

Avoid relying only on latest.

Prefer tags like:

- v1.0.0
- 1.4.2
- prod-20260307
- git-commit-sha

Example:

    docker build -t myapp:1.0.0 .
    docker tag myapp:1.0.0 registry.example.com/myapp:1.0.0
    docker push registry.example.com/myapp:1.0.0

### Why not only latest

Because latest is ambiguous:

- not always newest in a meaningful sense
- hard to roll back
- can cause deployment confusion

### Image inspection

Useful for debugging and audit:

    docker history myapp:1.0.0
    docker inspect myapp:1.0.0

You can inspect:

- layers
- entrypoint
- environment variables
- working directory
- exposed ports

### Keeping images small

Good practices:

- use slim or minimal base images
- use multi-stage builds
- remove package manager caches
- avoid unnecessary tools
- copy only required files
- use `.dockerignore`

### Example `.dockerignore`

    .git
    node_modules
    __pycache__
    *.log
    .env
    dist
    target

This prevents unwanted files from being sent to the Docker build context.

---

## 4. Running and Managing Containers

### Overview

Containers are the live, running instances of images.

You manage their lifecycle using Docker commands.

### Common container commands

#### Run a container

    docker run nginx

#### Run in detached mode

    docker run -d nginx

#### Name a container

    docker run -d --name web nginx

#### Map ports

    docker run -d -p 8080:80 nginx

#### Set environment variable

    docker run -e APP_ENV=prod myapp

#### Mount volume

    docker run -v mydata:/data mysql

#### View running containers

    docker ps

#### View all containers

    docker ps -a

#### Stop container

    docker stop web

#### Remove container

    docker rm web

#### View logs

    docker logs web

#### Exec into container

    docker exec -it web sh

### Container states

A container can be:

- created
- running
- paused
- stopped
- exited
- removed

---

## 1️⃣ Remove Stopped Containers

    docker container prune

Removes all stopped containers.

### Example prompt

    WARNING! This will remove all stopped containers.
    Are you sure you want to continue? [y/N]

### Force without prompt

    docker container prune -f

---

## 2️⃣ Remove Unused Images

    docker image prune

Removes dangling images (untagged images).

### Remove all unused images

    docker image prune -a

### Force

    docker image prune -a -f

---

## 3️⃣ Remove Unused Volumes

    docker volume prune

Removes volumes not used by any container.

### Force

    docker volume prune -f

---

## 4️⃣ Remove Unused Networks

    docker network prune

Removes networks not used by any container.

### Force

    docker network prune -f

---

## 5️⃣ Remove Build Cache

    docker builder prune

Removes build cache created by docker build.

### Remove all cache

    docker builder prune -a

---

## 6️⃣ Clean Everything (Most Powerful)

    docker system prune

Removes:

- stopped containers
- unused networks
- dangling images
- build cache

### Remove everything including unused images

    docker system prune -a

### Remove everything including volumes

    docker system prune -a --volumes

### Force

    docker system prune -a --volumes -f

---

## 7️⃣ Check Disk Usage Before Pruning

Very useful before running prune.

    docker system df

### Example output

    TYPE         TOTAL ACTIVE SIZE
    Images        12   3    2.4GB
    Containers     5   1    500MB
    Volumes        7   2    1.2GB
    Build Cache   20   0    800MB

### 🚨 Important DevOps Tip

Never blindly run:

    docker system prune -a --volumes

on production servers because it can delete:

- important volumes (database data)
- cached images used by pipelines.

---

## ⭐ Most Common Command DevOps Engineers Use

For CI/CD runners:

    docker system prune -af

or

    docker builder prune -af

to clean build cache.

---

## Real-world example

Run a simple web server:

    docker run -d --name mynginx -p 8080:80 nginx

Then open:

    http://localhost:8080

You now have a web server running inside a container.

---

## Dockerfile — Maven Application

    FROM maven:3.9.9-eclipse-temurin-17
    WORKDIR /app

    # Copy pom.xml first (for Docker cache)
    COPY pom.xml .

    # Download dependencies
    RUN mvn -B -q -DskipTests dependency:go-offline

    # Copy source code
    COPY src ./src

    # Build the jar
    RUN mvn clean package -DskipTests

    # Expose application port
    EXPOSE 8080

    # Run the application
    CMD ["java", "-jar", "target/app.jar"]

---

## Multi-stage Dockerfile — Maven Application

    # ===== Build Stage =====
    FROM maven:3.9.9-eclipse-temurin-17 AS build
    WORKDIR /app
    # rrr

    # Copy pom.xml first to leverage Docker layer caching
    COPY pom.xml .

    # Pre-download dependencies (faster rebuilds)
    RUN mvn -B -q -DskipTests dependency:go-offline

    # Copy project source
    COPY src ./src

    # Build the JAR (skip tests for faster build)
    RUN mvn clean package -DskipTests

    # ===== Runtime Stage =====
    FROM eclipse-temurin:17-jre-alpine
    WORKDIR /app

    # Copy built jar from build stage
    COPY --from=build /app/target/*.jar app.jar

    # Expose app port
    EXPOSE 8080

    # Run the app
    ENTRYPOINT ["java", "-jar", "app.jar"]
