---
name: docker
description: Help with any Docker task — containers, images, Dockerfiles, volumes, networks, Docker Compose, Swarm, security, and production patterns. Trigger with "dockerize this", "write a Dockerfile", "docker-compose for", "container won't start", "optimize my image", "docker networking", "docker volumes", "deploy with swarm", or any Docker CLI question.
argument-hint: "<task description, Dockerfile, docker-compose.yml, error message, or question>"
---

# /docker

> **Agent = Model + Harness → Docker = App + Container**
>
> Docker simplifies how we package, distribute, and run software by using containers for process isolation and images as immutable, shippable units.

Write, debug, optimize, or explain any Docker configuration.

## Usage

```
/docker <what you want to build, fix, or understand>
```

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                         DOCKER SKILL                             │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 1 — UNDERSTAND                                            │
│  ✓ Identify the task: write / debug / optimize / explain         │
│  ✓ Ask for missing context (language, framework, port, etc.)     │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 2 — BUILD / FIX                                           │
│  ✓ Produce Dockerfile, Compose file, or CLI commands             │
│  ✓ Apply best practices (layer consolidation, non-root, etc.)    │
│  ✓ Call out security risks and common pitfalls                   │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 3 — VERIFY                                                │
│  ✓ Explain what each instruction does                            │
│  ✓ Suggest how to test the result                                │
│  ✓ Flag potential issues (image size, port conflicts, etc.)      │
└──────────────────────────────────────────────────────────────────┘
```

---

## Core Concepts

### Images vs. Containers

| Concept | Description |
|---------|-------------|
| **Image** | Immutable, layered snapshot of all files and metadata a program needs. The shippable unit. |
| **Container** | A running instance of an image. Provides an isolated process environment using Linux namespaces and cgroups. |
| **Layer** | Each Dockerfile instruction produces one layer; layers below the writable layer are immutable and shareable. |
| **Tag** | Human-readable alias for an image version. `latest` should always point to the latest **stable** build. |
| **Digest (CAIID)** | Cryptographic hash `image@sha256:...` — references a specific, unchanging layer. Use in Dockerfiles for reproducible builds. |

### Container Lifecycle States

```
created → running → paused → running
                 ↓              ↓
              exited ←──── restarting
                 ↓
              removing (→ gone)
```

| Command | Effect |
|---------|--------|
| `docker create` | Creates container, does not start it |
| `docker start / run` | Starts or creates-and-starts |
| `docker stop / kill` | Graceful or immediate stop |
| `docker pause / unpause` | Freeze/unfreeze process |
| `docker rm` | Remove stopped container |
| `docker rm -f` | Force-remove running container |

---

## Dockerfile Reference

### Instruction Quick Reference

| Instruction | Purpose | Best Practice |
|-------------|---------|---------------|
| `FROM` | Set base image — **must be first** | Pin with digest (`FROM debian@sha256:...`) for reproducible builds |
| `LABEL` | Add metadata (maintainer, version) | Always set `maintainer` for accountability |
| `RUN` | Execute commands, commit layer | **Consolidate with `&&`** to minimize layers |
| `COPY` | Copy files from build context into image | Prefer over `ADD` for local files |
| `ADD` | Like `COPY` but also fetches URLs and extracts archives | Use only for archive extraction; use `RUN curl` for URLs (allows cleanup) |
| `ENV` | Set environment variables | Use for runtime config, not secrets |
| `ARG` | Build-time variable (not available at runtime) | Use for build-time config |
| `EXPOSE` | Document which port the container listens on | Documentation only — doesn't publish the port |
| `VOLUME` | Declare a mount point | Decouples data from the container lifecycle |
| `USER` | Set the default run-as user | **Always set a non-root user** |
| `WORKDIR` | Set working directory | Use instead of `RUN cd ...` |
| `ENTRYPOINT` | Main executable — not overridden by `docker run` args | Use exec form `["cmd"]` not shell form |
| `CMD` | Default args to `ENTRYPOINT`, or default command | Overridden by `docker run <image> <cmd>` |
| `HEALTHCHECK` | Command Docker runs to check if app is healthy | Always define for production images |
| `ONBUILD` | Trigger instruction for downstream builds | Useful for base/template images |

### ENTRYPOINT vs CMD

```
ENTRYPOINT ["node"]   # fixed executable
CMD ["server.js"]     # default arg, overridable

# Result: docker run myapp         → node server.js
#         docker run myapp test.js → node test.js
```

### Multi-Stage Build Pattern

Separates build tools from the final runtime image — dramatically smaller and more secure images.

```dockerfile
# Stage 1: build
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: runtime (only what's needed)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/app.jar .
USER nobody
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s CMD wget -q -O- http://localhost:8080/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### .dockerignore

Prevents large or sensitive files from being sent to the daemon (affects build speed and security).

```
.git
*.log
node_modules
.env
target/
*.md
```

---

## Essential CLI Commands

### Container Management

```bash
# Run a container
docker run -d --name myapp -p 8080:80 myimage:1.0

# Run interactively
docker run -it --rm ubuntu:22.04 bash

# List containers
docker ps          # running only
docker ps -a       # all

# Logs
docker logs myapp
docker logs -f myapp          # follow
docker logs --tail 50 myapp   # last 50 lines

# Exec into running container
docker exec -it myapp bash

# Inspect
docker inspect myapp

# Stop / remove
docker stop myapp && docker rm myapp

# Remove all stopped containers
docker container prune
```

### Image Management

```bash
# Build
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .

# List / remove
docker images
docker rmi myapp:1.0

# Tag
docker tag myapp:1.0 registry.example.com/myapp:1.0

# Push / pull
docker push registry.example.com/myapp:1.0
docker pull ubuntu:22.04

# Remove unused images
docker image prune -a

# Inspect layers
docker history myapp:1.0
```

### System Cleanup

```bash
docker system prune        # remove stopped containers, unused networks, dangling images
docker system prune -a     # also remove unused images
docker volume prune        # remove unused volumes
```

---

## Storage & Volumes

### Three Storage Types

| Type | How to mount | Use case |
|------|-------------|----------|
| **Docker Volume** | `--mount type=volume,src=mydata,dst=/data` | Production data — portable, managed by Docker |
| **Bind Mount** | `--mount type=bind,src=/host/path,dst=/app` | Dev workflow, config files, sharing host files |
| **tmpfs** | `--mount type=tmpfs,dst=/tmp,tmpfs-size=64m` | Sensitive in-memory data (secrets, caches) |

```bash
# Create and use a named volume
docker volume create mydata
docker run -d --mount type=volume,src=mydata,dst=/var/lib/data myimage

# Inspect volume
docker volume inspect mydata

# Share volume between containers
docker run -d --volumes-from container_a myimage
```

**Best Practice:** Use Docker-managed **named volumes** over bind mounts in production — keeps container definitions host-agnostic.

---

## Networking

### Default Networks

| Network | Driver | Behavior |
|---------|--------|---------|
| `bridge` | bridge | Default; containers can reach each other by IP, not name |
| `host` | host | No network namespace — container shares host network stack |
| `none` | null | Total isolation; no network access |

### User-Defined Bridge Network (Recommended)

```bash
# Create
docker network create mynet

# Run containers on it — they find each other by name (DNS)
docker run -d --name db --network mynet postgres:16
docker run -d --name api --network mynet myapi:1.0

# Inside api container: connect to "db:5432" by name
```

**Best Practice:** Always use **user-defined bridge networks** instead of the default `bridge` or legacy `--link`. User-defined networks have automatic DNS-based service discovery.

### Port Publishing

```bash
docker run -p 8080:80 myimage       # host:container
docker run -p 127.0.0.1:8080:80 myimage  # bind to localhost only
docker run -P myimage               # auto-publish all EXPOSE'd ports
```

---

## Docker Compose

Declarative multi-container environments using YAML. Models services, networks, and volumes as a single stack.

### Compose File Structure

```yaml
version: "3.9"

services:
  api:
    build: ./api
    image: myorg/api:latest
    ports:
      - "8080:8080"
    environment:
      DB_HOST: postgres
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - backend
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  postgres:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Compose Commands

```bash
docker compose up -d          # start all services (detached)
docker compose down           # stop and remove containers
docker compose down -v        # also remove volumes
docker compose logs -f api    # follow logs for one service
docker compose ps             # list services
docker compose build          # rebuild images

# Deploy a Swarm stack
docker stack deploy -c docker-compose.yml --prune mystack
docker stack rm mystack
docker stack services mystack
```

**Best Practice:** Keep a single Compose file representing the entire environment. Use `--prune` on `docker stack deploy` to auto-clean removed services.

---

## Security

### Principle of Least Privilege

```dockerfile
# Create a dedicated non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser
```

### Linux Capabilities

Docker drops most admin capabilities by default. Drop more where possible:

```bash
docker run --cap-drop net_raw --cap-drop sys_admin myimage
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myimage
```

### Remove SUID/SGID Bits

Add to Dockerfile to reduce privilege escalation risk:

```dockerfile
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true
```

### Secrets (Never Use ENV for Secrets)

```bash
# Create secret in Swarm
echo "mysecretpassword" | docker secret create db_password -

# Access in container at /run/secrets/db_password (in-memory tmpfs)
```

**Why not env vars?** They're visible to all child processes, appear in `docker inspect`, and can be logged. Docker Secrets mount to a read-only in-memory tmpfs — far safer.

### Immutable Base Images

```dockerfile
# Pin by digest for reproducible, tamper-proof builds
FROM debian@sha256:6aedee3ef827...
```

### Read-Only Container Filesystem

```bash
docker run --read-only --tmpfs /tmp myimage
```

---

## Resource Limits

```bash
# Memory: prevent one container from starving the host
docker run --memory 512m --memory-swap 512m myimage

# CPU: hard limit (0.75 = 75% of one core)
docker run --cpus 0.75 myimage

# CPU: relative weight (default 1024)
docker run --cpu-shares 512 myimage

# Device access (whitelist, not privileged)
docker run --device /dev/video0:/dev/video0 myimage
```

---

## Health Checks

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```bash
# At runtime
docker run --health-cmd="curl -f http://localhost/health" \
           --health-interval=30s \
           myimage
```

Swarm uses health checks to determine if a task is eligible to receive traffic and when to restart it.

---

## Docker Swarm (Production Orchestration)

Swarm continuously reconciles **desired state** with **current state** — automatically resurrects failed tasks.

```bash
# Initialize a swarm
docker swarm init

# Deploy a stack
docker stack deploy -c docker-compose.yml mystack

# Scale a service
docker service scale mystack_api=5

# Inspect services
docker service ls
docker service ps mystack_api

# Rolling update
docker service update --image myapi:2.0 mystack_api

# Service rollback
docker service rollback mystack_api
```

### Routing Mesh

Swarm listens on a published port on **every node** and forwards traffic to healthy containers — clients can connect to any node.

```
Client → any_node:8080 → routing mesh → healthy container (any node)
```

---

## Image Distribution

| Option | Best For |
|--------|----------|
| **Docker Hub (public)** | Open source, community images |
| **Docker Hub (private)** | Small teams, low-budget projects |
| **Private registry** | Proprietary software, compliance, full control over availability and longevity |

```bash
# Login and push
docker login registry.example.com
docker tag myapp:1.0 registry.example.com/myorg/myapp:1.0
docker push registry.example.com/myorg/myapp:1.0

# Run private registry locally
docker run -d -p 5000:5000 --name registry registry:2
```

---

## Best Practices Summary

| Area | Rule |
|------|------|
| **Layers** | Consolidate `RUN` commands with `&&` to minimize layer count and image size |
| **Base image** | Use minimal bases (`alpine`, `slim`, `distroless`); pin with digest for production |
| **Multi-stage** | Always separate build tools from runtime in production images |
| **User** | Always run as a non-root user (`USER` instruction) |
| **Secrets** | Use Docker Secrets or environment-injected files — never `ENV` for sensitive values |
| **Tags** | Tag stable builds with both `latest` and a specific version (e.g., `1.9.1`) |
| **Health checks** | Define `HEALTHCHECK` in every production image |
| **Networks** | Use user-defined bridge networks — never the default `bridge` or legacy `--link` |
| **Volumes** | Use named Docker volumes over bind mounts for production data |
| **Logs** | Don't rely on `docker logs` for long-lived containers — use a volume or logging driver |
| **.dockerignore** | Always define to keep build context small and prevent secrets leaking into images |
| **Resource limits** | Always set `--memory` and `--cpus` in production to prevent resource starvation |

---

## Common Patterns

### 12-Factor App with Docker

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM base AS app
COPY --chown=node:node . .
USER node
EXPOSE 3000
HEALTHCHECK CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "server.js"]
```

### Development with Hot Reload

```yaml
services:
  app:
    build:
      context: .
      target: base        # stop at base stage
    volumes:
      - .:/app            # bind mount source for live reload
      - /app/node_modules # keep container's node_modules
    command: npm run dev
    ports:
      - "3000:3000"
```

### Database + API + Reverse Proxy

```yaml
services:
  nginx:
    image: nginx:alpine
    ports: ["80:80"]
    volumes: ["./nginx.conf:/etc/nginx/nginx.conf:ro"]
    depends_on: [api]

  api:
    build: ./api
    expose: ["8080"]
    environment:
      DB_URL: postgresql://postgres:5432/mydb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s

volumes:
  pgdata:
```
