# Docker - Complete Study Guide
> From Zero to Confident: Understand Docker deeply and explain it to others

---

## Table of Contents
1. [What is Docker? (The Big Picture)](#1-what-is-docker)
2. [Why Docker? (The Problem It Solves)](#2-why-docker)
3. [Core Concepts](#3-core-concepts)
4. [Installation](#4-installation)
5. [Your First Docker Commands](#5-your-first-docker-commands)
6. [Working with Images](#6-working-with-images)
7. [Working with Containers](#7-working-with-containers)
8. [Dockerfile - Building Your Own Images](#8-dockerfile)
9. [Volumes - Persisting Data](#9-volumes)
10. [Networking in Docker](#10-networking)
11. [Docker Compose - Multi-Container Apps](#11-docker-compose)
12. [Docker Registry & Docker Hub](#12-docker-registry)
13. [Real-World Project Example](#13-real-world-example)
14. [Best Practices](#14-best-practices)
15. [Quick Reference Cheat Sheet](#15-cheat-sheet)

---

## 1. What is Docker?

### Simple Explanation
Think of Docker like a **shipping container** for your software.

Before shipping containers existed, moving goods between ships, trucks, and trains was a nightmare — different sizes, different handling requirements, different weather protection needed.

**Shipping containers solved this** by creating a standard box that:
- Works the same on any ship, truck, or train
- Protects the goods inside regardless of weather outside
- Can be stacked, moved, and tracked easily

**Docker does the same for software:**
- Your app runs the same on your laptop, your colleague's machine, or a cloud server
- It carries everything the app needs (code, libraries, settings) inside itself
- It's isolated from other apps on the same machine

### Technical Definition
Docker is a platform that packages applications into **containers** — lightweight, standalone, executable units that include everything needed to run the application: code, runtime, libraries, and settings.

---

## 2. Why Docker?

### The Classic Problem: "It Works on My Machine"

Imagine this scenario:
```
Developer: "I finished the feature, here's the code."
Server:    [crashes immediately]
Developer: "But it works on my machine!"
```

**Why does this happen?**
- Your machine has Python 3.11, server has Python 3.8
- You installed a library your colleague never installed
- Your OS is Windows, production is Linux
- Environment variables are different

### How Docker Solves This

```
Without Docker:                    With Docker:
┌─────────────────────┐           ┌─────────────────────┐
│   Your Laptop       │           │   Your Laptop       │
│ ┌─────────────────┐ │           │ ┌─────────────────┐ │
│ │ App             │ │           │ │ Container       │ │
│ │ Python 3.11     │ │           │ │ ┌─────────────┐ │ │
│ │ Library v2.1    │ │           │ │ │ App         │ │ │
│ └─────────────────┘ │           │ │ │ Python 3.11 │ │ │
└─────────────────────┘           │ │ │ Lib v2.1    │ │ │
                                  │ └─────────────┘ │ │
┌─────────────────────┐           └─────────────────────┘
│   Server            │
│ ┌─────────────────┐ │           ┌─────────────────────┐
│ │ App             │ │           │   Server            │
│ │ Python 3.8      │ │           │ ┌─────────────────┐ │
│ │ Library v1.5    │ │           │ │ Container       │ │
│ └─────────────────┘ │           │ │ ┌─────────────┐ │ │
│ [APP CRASHES]       │           │ │ │ App         │ │ │
└─────────────────────┘           │ │ │ Python 3.11 │ │ │
                                  │ │ │ Lib v2.1    │ │ │
    ❌ Broken                      │ └─────────────┘ │ │
                                  └─────────────────────┘
                                      ✅ Works Perfectly
```

### Benefits Summary
| Benefit | Meaning |
|---|---|
| **Consistency** | Same behavior everywhere |
| **Isolation** | Apps don't interfere with each other |
| **Portability** | Run anywhere Docker is installed |
| **Speed** | Starts in seconds (vs. minutes for VMs) |
| **Efficiency** | Uses less RAM/disk than Virtual Machines |
| **Scalability** | Easy to run multiple copies |

---

## 3. Core Concepts

### 3.1 Docker vs Virtual Machine

This is a very common interview question. Understand it deeply.

```
Virtual Machine (VM):                Docker Container:
┌──────────────────────┐            ┌──────────────────────┐
│  Your Computer (Host)│            │  Your Computer (Host)│
│ ┌──────────────────┐ │            │ ┌──────────────────┐ │
│ │  Hypervisor      │ │            │ │  Docker Engine   │ │
│ │ ┌────┐  ┌────┐   │ │            │ │ ┌────┐  ┌────┐   │ │
│ │ │VM 1│  │VM 2│   │ │            │ │ │Con1│  │Con2│   │ │
│ │ │    │  │    │   │ │            │ │ │    │  │    │   │ │
│ │ │ OS │  │ OS │   │ │            │ │ │App │  │App │   │ │
│ │ │App │  │App │   │ │            │ │ │Lib │  │Lib │   │ │
│ │ └────┘  └────┘   │ │            │ │ └────┘  └────┘   │ │
│ └──────────────────┘ │            │ └──────────────────┘ │
│ ← Each VM has its    │            │ ← Containers SHARE   │
│   own full OS        │            │   the Host OS kernel  │
└──────────────────────┘            └──────────────────────┘

VM size: ~1-20 GB                   Container size: ~10-500 MB
Startup: 1-5 minutes                Startup: < 1 second
```

**Key difference:** VMs virtualize hardware. Containers virtualize the OS.

---

### 3.2 Image

An **Image** is like a **recipe** or a **snapshot**.

- Read-only template used to create containers
- Contains: OS base, dependencies, your app code, configuration
- Stored in layers (like a cake — each layer adds something)
- Can be shared via Docker Hub

**Real-world analogy:** An image is like a **cookie cutter**. You use it to create many identical cookies (containers). The cutter itself doesn't change.

```
Image (Read-only layers):
┌─────────────────────────┐
│  Layer 4: Your App Code │  ← What you added
├─────────────────────────┤
│  Layer 3: Python 3.11   │  ← Runtime
├─────────────────────────┤
│  Layer 2: Ubuntu 22.04  │  ← OS base
├─────────────────────────┤
│  Layer 1: Base Layer    │  ← Foundation
└─────────────────────────┘
```

---

### 3.3 Container

A **Container** is a **running instance** of an image.

- Created from an image
- Has its own filesystem, process space, and network
- Can be started, stopped, paused, deleted
- Multiple containers can run from the same image

**Real-world analogy:** A container is like a **house built from a blueprint** (image). Many houses can be built from the same blueprint. Each house is independent.

---

### 3.4 Dockerfile

A **Dockerfile** is a **text file with instructions** to build a Docker image.

Think of it as a **step-by-step recipe** that Docker follows to build your image.

---

### 3.5 Docker Hub

**Docker Hub** is like **GitHub but for Docker images**.

- A public registry where images are stored and shared
- Contains official images for: nginx, postgres, python, node, etc.
- You can push your own images and pull others

---

### 3.6 Volume

A **Volume** is Docker's way to **persist data**.

Containers are temporary — when deleted, their data is gone. Volumes solve this by storing data outside the container on the host machine.

**Analogy:** A volume is like a **USB drive** plugged into a container. Even if you throw away the container, the data on the USB stays safe.

---

### 3.7 Network

Docker containers by default are isolated. **Networks** let containers talk to each other.

**Analogy:** Containers on the same network are like computers in the same office LAN — they can see and communicate with each other.

---

## 4. Installation

### Windows
1. Download **Docker Desktop** from [docker.com](https://www.docker.com/products/docker-desktop/)
2. Run the installer
3. Restart your computer
4. Docker Desktop will start automatically

### macOS
```bash
# Option 1: Download Docker Desktop (recommended for beginners)
# Visit: https://www.docker.com/products/docker-desktop/

# Option 2: Homebrew
brew install --cask docker
```

### Linux (Ubuntu/Debian)
```bash
# Update package list
sudo apt-get update

# Install dependencies
sudo apt-get install ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Run without sudo (optional but recommended)
sudo usermod -aG docker $USER
newgrp docker
```

### Verify Installation
```bash
docker --version
# Docker version 24.x.x, build xxxxxxx

docker run hello-world
# Should print: "Hello from Docker!"
```

---

## 5. Your First Docker Commands

### The Golden Command to Understand Docker
```bash
docker run hello-world
```

What happens behind the scenes:
```
1. Docker looks for 'hello-world' image locally
2. Not found → Downloads from Docker Hub
3. Creates a container from the image
4. Starts the container
5. Container prints a message
6. Container exits (job done)
```

### Basic Command Structure
```bash
docker [command] [subcommand] [options] [arguments]
#  ↑        ↑          ↑          ↑
# tool   action   modifiers    target
```

---

## 6. Working with Images

### Pull an Image
```bash
# Pull the latest version
docker pull nginx

# Pull a specific version (tag)
docker pull nginx:1.25

# Pull specific OS flavor
docker pull python:3.11-slim
```

### List Images
```bash
docker images
# or
docker image ls

# Output:
# REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
# nginx        latest    a6bd71f48f68   2 weeks ago   187MB
# python       3.11      ...            ...           ...
```

### Remove an Image
```bash
# Remove by name
docker rmi nginx

# Remove by ID
docker rmi a6bd71f48f68

# Force remove (even if containers use it)
docker rmi -f nginx

# Remove all unused images
docker image prune
```

### Inspect an Image
```bash
docker inspect nginx
# Shows detailed JSON info: layers, environment vars, exposed ports, etc.
```

### Image Naming Convention
```
[registry/][username/]repository[:tag]
     ↑           ↑          ↑       ↑
  docker.io  your-name  image-name  version

Examples:
  nginx                    → docker.io/library/nginx:latest
  python:3.11-slim         → docker.io/library/python:3.11-slim
  myname/myapp:v1.0        → docker.io/myname/myapp:v1.0
```

---

## 7. Working with Containers

### Run a Container
```bash
# Basic run (foreground)
docker run nginx

# Run in detached mode (background)
docker run -d nginx

# Run with a name
docker run -d --name my-nginx nginx

# Run with port mapping  host:container
docker run -d -p 8080:80 nginx
#                ↑   ↑
#           your PC  container

# Run interactively (enter the container's shell)
docker run -it ubuntu bash
#           ↑↑
#    interactive + terminal
```

### List Containers
```bash
# Running containers only
docker ps

# All containers (running + stopped)
docker ps -a

# Output:
# CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
# abc123def456   nginx   ...       1 min     Up       ...     my-nginx
```

### Stop / Start / Restart
```bash
docker stop my-nginx      # Graceful stop (SIGTERM, then SIGKILL after 10s)
docker start my-nginx     # Start a stopped container
docker restart my-nginx   # Stop then start
docker kill my-nginx      # Immediate stop (SIGKILL)
```

### Remove a Container
```bash
docker rm my-nginx              # Remove stopped container
docker rm -f my-nginx           # Force remove (even if running)
docker container prune          # Remove all stopped containers
```

### Execute Commands Inside a Running Container
```bash
# Open a shell inside a running container
docker exec -it my-nginx bash

# Run a single command
docker exec my-nginx ls /etc/nginx

# Check nginx config from outside
docker exec my-nginx cat /etc/nginx/nginx.conf
```

### View Container Logs
```bash
docker logs my-nginx              # All logs
docker logs -f my-nginx           # Follow (live stream)
docker logs --tail 50 my-nginx    # Last 50 lines
docker logs --since 1h my-nginx   # Logs from last 1 hour
```

### Container Resource Usage
```bash
docker stats                   # Live resource usage of all containers
docker stats my-nginx          # For specific container
docker inspect my-nginx        # Detailed container info
```

### Copy Files To/From Container
```bash
# Copy from container to host
docker cp my-nginx:/etc/nginx/nginx.conf ./nginx.conf

# Copy from host to container
docker cp ./index.html my-nginx:/usr/share/nginx/html/
```

---

## 8. Dockerfile

### What is a Dockerfile?
A Dockerfile is a script that tells Docker how to build your image step by step.

### Dockerfile Instructions Reference

| Instruction | Purpose | Example |
|---|---|---|
| `FROM` | Base image to start from | `FROM python:3.11-slim` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files from host to image | `COPY . .` |
| `RUN` | Execute command during build | `RUN pip install -r requirements.txt` |
| `ENV` | Set environment variable | `ENV PORT=8000` |
| `EXPOSE` | Document which port app uses | `EXPOSE 8000` |
| `CMD` | Default command when container starts | `CMD ["python", "app.py"]` |
| `ENTRYPOINT` | Fixed executable for container | `ENTRYPOINT ["gunicorn"]` |
| `ARG` | Build-time variable | `ARG VERSION=1.0` |
| `VOLUME` | Declare a mount point | `VOLUME /data` |
| `USER` | Set user to run commands | `USER appuser` |
| `LABEL` | Add metadata | `LABEL version="1.0"` |

### Example 1: Simple Python App

**Project Structure:**
```
my-python-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

**app.py:**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**requirements.txt:**
```
flask==3.0.0
```

**Dockerfile:**
```dockerfile
# Step 1: Start from official Python image
FROM python:3.11-slim

# Step 2: Set the working directory inside container
WORKDIR /app

# Step 3: Copy requirements first (for layer caching optimization)
COPY requirements.txt .

# Step 4: Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Step 5: Copy the rest of your application code
COPY . .

# Step 6: Tell Docker which port the app uses
EXPOSE 5000

# Step 7: Command to run when container starts
CMD ["python", "app.py"]
```

**Build and Run:**
```bash
# Build the image (run from the folder containing Dockerfile)
docker build -t my-python-app .
#                ↑            ↑
#             name/tag     build context (current folder)

# Run a container from it
docker run -d -p 5000:5000 --name my-app my-python-app

# Visit http://localhost:5000
```

---

### Example 2: Node.js App

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Copy package files first (layer cache optimization)
COPY package*.json ./

RUN npm ci --only=production

COPY . .

EXPOSE 3000

# Use non-root user for security
USER node

CMD ["node", "server.js"]
```

---

### Understanding Layer Caching (Important!)

Docker builds images in layers. Each instruction = one layer. Layers are **cached**.

```dockerfile
# BAD - every code change rebuilds everything including npm install
FROM node:20
COPY . .              ← Any file change invalidates cache here
RUN npm install       ← Gets re-run every time!

# GOOD - only re-runs npm install when package.json changes
FROM node:20
COPY package*.json .  ← Only changes when dependencies change
RUN npm install       ← Cached unless package.json changed
COPY . .              ← Other files copied after install
```

**Rule:** Put things that change less frequently **earlier** in the Dockerfile.

---

### CMD vs ENTRYPOINT

| | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default arguments | Fixed executable |
| Overridable | Yes (easily) | Yes (with --entrypoint) |
| Use case | Flexible defaults | Container as a command |

```dockerfile
# CMD - can be overridden at runtime
CMD ["python", "app.py"]
# docker run myimage              → runs python app.py
# docker run myimage python other.py → runs python other.py

# ENTRYPOINT - container behaves like a command
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run myimage              → runs python app.py
# docker run myimage other.py     → runs python other.py
```

---

### .dockerignore File

Like `.gitignore` but for Docker. Prevents unnecessary files from being sent to the Docker build context.

```
# .dockerignore
node_modules/
.git/
.env
*.log
__pycache__/
.pytest_cache/
dist/
build/
README.md
```

---

## 9. Volumes

### The Problem
When a container is deleted, all data inside it is **gone forever**.

```bash
docker run -d --name mydb postgres
# ... create some data in the database ...
docker rm -f mydb
# All database data is GONE!
```

### Types of Storage in Docker

```
┌────────────────────────────────────────────┐
│              Host Machine                  │
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │  Named   │  │  Bind    │  │  tmpfs   │ │
│  │  Volume  │  │  Mount   │  │  Mount   │ │
│  │(managed  │  │(your own │  │(in RAM,  │ │
│  │by Docker)│  │folder)   │  │temp)     │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘ │
│       │              │              │       │
└───────┼──────────────┼──────────────┼───────┘
        │              │              │
        └──────────────┴──────────────┘
                       │
              ┌────────┴────────┐
              │    Container    │
              └─────────────────┘
```

### Named Volumes (Recommended for Production Data)
```bash
# Create a named volume
docker volume create mydata

# Use it with a container
docker run -d \
  --name mydb \
  -v mydata:/var/lib/postgresql/data \
  postgres

# List volumes
docker volume ls

# Inspect a volume (find where data is stored on host)
docker volume inspect mydata

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

### Bind Mounts (Best for Development)
```bash
# Mount your local folder into the container
docker run -d \
  -p 5000:5000 \
  -v /path/on/your/pc:/app \
  my-python-app

# On Windows
docker run -d \
  -p 5000:5000 \
  -v C:\Users\you\project:/app \
  my-python-app

# On Mac/Linux shortcut using $(pwd)
docker run -d \
  -p 5000:5000 \
  -v $(pwd):/app \
  my-python-app
```

**Use Case:** During development, changes to your code on the host are immediately reflected inside the container — no rebuild needed!

---

## 10. Networking

### Docker Network Types

| Type | Description | Use Case |
|---|---|---|
| **bridge** | Default. Containers on same bridge can talk to each other | Most apps |
| **host** | Container uses host's network directly | High-performance needs |
| **none** | No networking at all | Maximum isolation |
| **overlay** | Multi-host networking (Docker Swarm) | Distributed systems |

### Default Bridge Network
```bash
# All containers by default join the 'bridge' network
docker run -d --name app1 nginx
docker run -d --name app2 nginx

# They can communicate by IP but NOT by name on default bridge
docker inspect app1 | grep IPAddress
# "IPAddress": "172.17.0.2"

# app2 can reach app1 via: 172.17.0.2 (but not by name 'app1')
```

### Custom Bridge Network (Recommended)
```bash
# Create a custom network
docker network create my-network

# Run containers on the same network
docker run -d --name backend --network my-network my-api
docker run -d --name frontend --network my-network my-ui

# Now they can talk by NAME!
# frontend can reach backend at http://backend:8000
# backend can reach frontend at http://frontend:3000

# List networks
docker network ls

# Inspect a network
docker network inspect my-network

# Connect an existing container to a network
docker network connect my-network existing-container

# Disconnect
docker network disconnect my-network existing-container

# Remove network
docker network rm my-network
```

### Port Publishing (Host to Container)
```bash
# -p hostPort:containerPort
docker run -p 8080:80 nginx
#               ↑   ↑
#          localhost  container
# Visit http://localhost:8080 → hits port 80 inside the container

# Bind to specific host IP
docker run -p 127.0.0.1:8080:80 nginx

# Random host port
docker run -p 80 nginx
docker port container-name  # Find which port was assigned
```

---

## 11. Docker Compose

### What is Docker Compose?

Docker Compose lets you define and run **multi-container applications** using a single YAML file.

Instead of running many `docker run` commands, you write one `docker-compose.yml` file and run `docker compose up`.

**Real-world scenario:** A web app usually needs:
- Frontend (React/Angular)
- Backend (Python/Node)
- Database (PostgreSQL/MySQL)
- Cache (Redis)

Managing all of these with individual `docker run` commands is painful. Compose makes it easy.

### docker-compose.yml Structure

```yaml
version: "3.8"                    # Compose file format version

services:                         # Each service is a container
  service-name:
    image: image-name             # Use existing image
    # OR
    build: ./path/to/dockerfile   # Build from Dockerfile
    ports:
      - "host:container"
    volumes:
      - "host:container"
    environment:
      - KEY=VALUE
    depends_on:
      - other-service
    networks:
      - my-network

networks:                         # Define networks
  my-network:

volumes:                          # Define volumes
  my-volume:
```

### Full Example: Web App + PostgreSQL + Redis

**docker-compose.yml:**
```yaml
version: "3.8"

services:
  # Web Application
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://admin:secret@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    volumes:
      - .:/app          # Bind mount for development (live reload)
    networks:
      - app-network

  # PostgreSQL Database
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data   # Named volume for persistence
    networks:
      - app-network

  # Redis Cache
  cache:
    image: redis:7-alpine
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:          # Docker manages this volume
```

### Docker Compose Commands

```bash
# Start all services (in background)
docker compose up -d

# Start and rebuild images
docker compose up -d --build

# View running services
docker compose ps

# View logs for all services
docker compose logs

# View logs for specific service
docker compose logs web

# Follow logs live
docker compose logs -f web

# Stop all services (keeps containers and volumes)
docker compose stop

# Stop and remove containers (keeps volumes)
docker compose down

# Stop and remove containers + volumes (DELETES DATA!)
docker compose down -v

# Scale a service (run multiple instances)
docker compose up -d --scale web=3

# Execute command in a running service
docker compose exec web bash

# Run a one-off command in a new container
docker compose run web python manage.py migrate
```

### Development vs Production Compose

```bash
# Base file
docker-compose.yml

# Development overrides
docker-compose.override.yml    # Auto-loaded with docker compose up

# Production
docker-compose.prod.yml

# Use specific files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

**docker-compose.override.yml (development):**
```yaml
version: "3.8"
services:
  web:
    volumes:
      - .:/app          # Live code reload
    environment:
      - DEBUG=true
    command: python app.py --debug
```

**docker-compose.prod.yml:**
```yaml
version: "3.8"
services:
  web:
    restart: always     # Auto-restart on failure
    environment:
      - DEBUG=false
```

---

## 12. Docker Registry

### Docker Hub
```bash
# Login to Docker Hub
docker login

# Tag your image for Docker Hub
docker tag my-python-app yourusername/my-python-app:v1.0

# Push to Docker Hub
docker push yourusername/my-python-app:v1.0

# Pull from Docker Hub
docker pull yourusername/my-python-app:v1.0
```

### Run Your Own Private Registry
```bash
# Start a local registry
docker run -d -p 5001:5000 --name registry registry:2

# Tag image for local registry
docker tag my-app localhost:5001/my-app:latest

# Push to local registry
docker push localhost:5001/my-app:latest

# Pull from local registry
docker pull localhost:5001/my-app:latest
```

---

## 13. Real-World Example

### Full Stack App: Todo API with PostgreSQL

**Project Structure:**
```
todo-app/
├── app/
│   ├── main.py
│   └── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── .dockerignore
```

**app/main.py:**
```python
from flask import Flask, jsonify, request
import psycopg2
import os

app = Flask(__name__)

def get_db():
    return psycopg2.connect(os.getenv("DATABASE_URL"))

@app.route("/todos", methods=["GET"])
def get_todos():
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT id, task, done FROM todos")
    todos = [{"id": r[0], "task": r[1], "done": r[2]} for r in cur.fetchall()]
    conn.close()
    return jsonify(todos)

@app.route("/todos", methods=["POST"])
def add_todo():
    task = request.json.get("task")
    conn = get_db()
    cur = conn.cursor()
    cur.execute("INSERT INTO todos (task, done) VALUES (%s, false) RETURNING id", (task,))
    todo_id = cur.fetchone()[0]
    conn.commit()
    conn.close()
    return jsonify({"id": todo_id, "task": task, "done": False}), 201

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

EXPOSE 5000

CMD ["python", "main.py"]
```

**docker-compose.yml:**
```yaml
version: "3.8"

services:
  api:
    build: .
    ports:
      - "5000:5000"
    environment:
      DATABASE_URL: postgresql://postgres:password@db:5432/tododb
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-net

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: tododb
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Auto-runs on first start
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-net

networks:
  app-net:

volumes:
  db-data:
```

**Run it:**
```bash
docker compose up -d
# Visit http://localhost:5000/todos
```

---

## 14. Best Practices

### Image Best Practices

**1. Use specific tags, not `latest`**
```dockerfile
# Bad
FROM python:latest

# Good
FROM python:3.11.7-slim
```

**2. Use slim/alpine variants when possible**
```dockerfile
# Full image: ~900MB
FROM python:3.11

# Slim: ~130MB
FROM python:3.11-slim

# Alpine: ~50MB (but can have compatibility issues)
FROM python:3.11-alpine
```

**3. Run as non-root user**
```dockerfile
FROM python:3.11-slim

# Create a user
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app
COPY . .

# Switch to non-root user
USER appuser

CMD ["python", "app.py"]
```

**4. Use multi-stage builds to reduce image size**
```dockerfile
# Stage 1: Build (with all dev tools)
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build       # Produces /app/dist/

# Stage 2: Production (tiny image, only runtime)
FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
# Final image has NO node_modules, NO source code, NO dev tools!
```

**5. Order Dockerfile layers for cache efficiency**
```dockerfile
# Dependencies first (rarely change)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Source code last (changes often)
COPY . .
```

### Security Best Practices

```dockerfile
# Don't hardcode secrets
# Bad:
ENV DB_PASSWORD=supersecret123

# Good: Use environment variables at runtime or Docker secrets
# docker run -e DB_PASSWORD=secret myapp
# or use docker secrets for production
```

```bash
# Scan image for vulnerabilities
docker scout cves myimage:latest
# or
docker scan myimage:latest
```

### Container Best Practices

```yaml
# docker-compose.yml
services:
  web:
    # Always restart on failure in production
    restart: unless-stopped

    # Limit resources
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

    # Health checks
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### The 12-Factor App Principles for Docker

1. **Config via environment variables** — never hardcode
2. **One process per container** — keep containers focused
3. **Stateless containers** — store state in volumes/databases
4. **Logs to stdout** — let Docker handle log collection
5. **Fast startup and graceful shutdown**

---

## 15. Cheat Sheet

### Image Commands
```bash
docker pull image:tag              # Download image
docker build -t name:tag .         # Build from Dockerfile
docker images / docker image ls    # List images
docker rmi image                   # Remove image
docker image prune                 # Remove unused images
docker tag src:tag dest:tag        # Tag an image
docker push name:tag               # Push to registry
docker inspect image               # Image details
docker history image               # Show image layers
```

### Container Commands
```bash
docker run image                   # Create + start container
docker run -d image                # Detached (background)
docker run -it image bash          # Interactive + terminal
docker run --name NAME image       # Give it a name
docker run -p 8080:80 image        # Port mapping host:container
docker run -v /host:/container     # Bind mount
docker run -e KEY=VALUE image      # Set env variable
docker run --rm image              # Auto-remove when done

docker ps                          # Running containers
docker ps -a                       # All containers
docker start NAME                  # Start stopped container
docker stop NAME                   # Stop container (graceful)
docker kill NAME                   # Stop container (immediate)
docker restart NAME                # Restart container
docker rm NAME                     # Remove stopped container
docker rm -f NAME                  # Force remove

docker exec -it NAME bash          # Shell into running container
docker exec NAME command           # Run command in container
docker logs NAME                   # View logs
docker logs -f NAME                # Follow logs
docker cp NAME:/path ./local       # Copy from container
docker stats                       # Resource usage
docker top NAME                    # Processes in container
docker inspect NAME                # Container details
```

### Volume Commands
```bash
docker volume create NAME          # Create volume
docker volume ls                   # List volumes
docker volume inspect NAME         # Volume details
docker volume rm NAME              # Remove volume
docker volume prune                # Remove unused volumes
```

### Network Commands
```bash
docker network create NAME         # Create network
docker network ls                  # List networks
docker network inspect NAME        # Network details
docker network connect NET CONT    # Connect container to network
docker network disconnect NET CONT # Disconnect
docker network rm NAME             # Remove network
docker network prune               # Remove unused networks
```

### Docker Compose Commands
```bash
docker compose up                  # Start services
docker compose up -d               # Start in background
docker compose up --build          # Rebuild and start
docker compose down                # Stop and remove containers
docker compose down -v             # Also remove volumes
docker compose ps                  # List services
docker compose logs                # View all logs
docker compose logs -f SERVICE     # Follow service logs
docker compose exec SERVICE bash   # Shell into service
docker compose run SERVICE CMD     # One-off command
docker compose restart SERVICE     # Restart service
docker compose pull                # Pull latest images
docker compose build               # Build images only
docker compose stop                # Stop without removing
docker compose start               # Start stopped services
```

### System Commands
```bash
docker system df                   # Disk usage
docker system prune                # Remove all unused resources
docker system prune -a             # Remove everything not running
docker info                        # Docker system info
docker version                     # Docker version
```

---

## Common Interview Questions

**Q1: What is the difference between Docker Image and Docker Container?**
> Image is the template (blueprint), Container is the running instance (building). You can create many containers from one image.

**Q2: How is Docker different from a Virtual Machine?**
> Docker shares the host OS kernel — only the application layer is isolated. VMs virtualize the full OS including kernel. Docker is faster, lighter, and starts in seconds vs minutes for VMs.

**Q3: What is a Dockerfile?**
> A text file with step-by-step instructions to build a Docker image. Each instruction creates a new layer in the image.

**Q4: How do you persist data in Docker?**
> Using Volumes (named volumes for production, bind mounts for development). Container filesystems are ephemeral — data is lost when a container is removed without volumes.

**Q5: What is Docker Compose used for?**
> Managing multi-container applications. Instead of running multiple `docker run` commands, you define all services in a `docker-compose.yml` file and manage them together.

**Q6: What is the difference between CMD and ENTRYPOINT?**
> Both define what runs when a container starts. CMD sets default arguments that can be easily overridden. ENTRYPOINT sets the fixed executable; arguments are appended to it.

**Q7: What is a multi-stage build?**
> A technique to reduce final image size by using separate stages in one Dockerfile — a build stage with all tools, and a runtime stage that copies only the built artifacts. The final image doesn't have build tools.

**Q8: How do containers communicate with each other?**
> By connecting them to the same Docker network. On a custom bridge network, containers can reach each other by service name. On the default bridge network, they must use IP addresses.

---

*Created for learning Docker from scratch. Practice each section hands-on for best results.*
