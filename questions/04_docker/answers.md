# Docker - COMPREHENSIVE ANSWERS (All 80 Questions)

---

# PART 1: BASICS & FUNDAMENTALS (30 Qs)

---

## Core Concepts

**1. What is Docker? What problem does it solve?**
Platform for building, shipping, running apps in containers. Solves: "works on my machine" — packages app + dependencies + config into a portable unit that runs identically everywhere.

**2. Container vs VM?**
| Container | VM |
|---|---|
| Shares host OS kernel | Has own OS kernel |
| Lightweight (MBs) | Heavy (GBs) |
| Starts in seconds | Starts in minutes |
| Process-level isolation | Full hardware isolation |
| Uses cgroups/namespaces | Uses hypervisor |
| Docker, containerd | VMware, VirtualBox, Hyper-V |

**3. Docker image vs container?**
- **Image**: Read-only template with app + dependencies. Like a class/blueprint.
- **Container**: Running instance of an image. Like an object/instance. Writable layer on top.

**4. Docker Engine components?**
- **Docker daemon (dockerd)**: Background service managing images, containers, networks, volumes
- **Docker CLI**: Command-line tool (`docker build`, `docker run`)
- **containerd**: Container runtime
- **runc**: Low-level OCI runtime

**5. Docker daemon vs Docker CLI?**
- **daemon**: Server process that does the actual work (building, running containers)
- **CLI**: Client that sends commands to daemon via REST API
- Can run on different machines (remote Docker)

**6. Docker registry? Name 3.**
Storage for Docker images. Registries:
1. **Docker Hub**: Public default registry
2. **Azure Container Registry (ACR)**: Azure private registry
3. **Amazon ECR**: AWS private registry
4. **GitHub Container Registry (ghcr.io)**
5. **Harbor**: Self-hosted open-source

**7. Docker Hub? Private registry?**
Docker Hub: Default public registry. Free for public images. Yes, private registries: ACR, ECR, self-hosted Harbor, or Docker Hub paid plans.

**8. Container lifecycle?**
```bash
docker create nginx        # Create (not running)
docker start <id>          # Start
docker pause <id>          # Pause
docker unpause <id>        # Resume
docker stop <id>           # Graceful stop (SIGTERM → SIGKILL)
docker kill <id>           # Force stop (SIGKILL)
docker rm <id>             # Remove
```

**9. Docker layer? How layering works?**
Each Dockerfile instruction creates a layer. Layers are cached and shared between images. Only changed layers are rebuilt.
```
Layer 4: COPY app.py          ← Changes most often
Layer 3: RUN pip install      ← Cached if requirements unchanged
Layer 2: COPY requirements.txt
Layer 1: FROM python:3.11-slim ← Base layer
```

**10. Union filesystem?**
Combines multiple read-only layers into one unified view. Container adds writable layer on top. Technologies: OverlayFS (default), AUFS.

**11. List containers?**
```bash
docker ps                  # Running containers
docker ps -a               # All containers (including stopped)
docker ps -q               # Only IDs
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**12. Show container logs?**
```bash
docker logs <container>
docker logs -f <container>        # Follow (real-time)
docker logs --tail 100 <container> # Last 100 lines
docker logs --since 1h <container> # Last hour
```

**13. Execute command inside running container?**
```bash
docker exec -it <container> /bin/sh    # Interactive shell
docker exec -it <container> bash       # Bash shell
docker exec <container> ls /app        # One-off command
```

**14. `docker inspect`?**
Returns detailed JSON info about container/image: IP address, mounts, environment variables, network settings, health status.
```bash
docker inspect <container>
docker inspect --format '{{.NetworkSettings.IPAddress}}' <container>
```

**15. Remove stopped containers and unused images?**
```bash
docker container prune       # Remove stopped containers
docker image prune           # Remove dangling images
docker image prune -a        # Remove all unused images
docker system prune          # Remove everything unused
docker system prune -a --volumes  # Nuclear option
```

---

## Dockerfile Deep Dive

**16. Dockerfile? Build context?**
- **Dockerfile**: Text file with instructions to build an image
- **Build context**: Directory sent to Docker daemon. Everything in the directory (minus .dockerignore) is sent.
```bash
docker build -t myapp:v1 .    # '.' is build context
```

**17. Dockerfile instructions?**
```dockerfile
FROM python:3.11-slim          # Base image
WORKDIR /app                   # Set working directory
ENV APP_ENV=production         # Set environment variable
ARG BUILD_VERSION=1.0          # Build-time variable
LABEL maintainer="vaibhav"    # Metadata
COPY requirements.txt .        # Copy files from build context
ADD archive.tar.gz /app/       # Copy + auto-extract archives
RUN pip install -r requirements.txt  # Execute command during build
COPY . .                       # Copy rest of app
EXPOSE 8080                    # Document port (metadata only)
USER appuser                   # Run as non-root user
VOLUME ["/data"]               # Create mount point
HEALTHCHECK --interval=30s CMD curl -f http://localhost:8080/health
CMD ["python", "app.py"]       # Default command
ENTRYPOINT ["python"]          # Fixed executable
```

**18. COPY vs ADD?**
- **COPY**: Copies files/directories. Simple, predictable. **Always prefer COPY.**
- **ADD**: Same as COPY + auto-extracts tar archives + supports URLs.
- Only use ADD when you need tar extraction.

**19. CMD vs ENTRYPOINT?**
- **CMD**: Default command. Can be overridden at `docker run`.
- **ENTRYPOINT**: Fixed executable. Arguments from `docker run` are appended.
```dockerfile
# CMD: user can override entire command
CMD ["python", "app.py"]
docker run myapp python other.py    # Overrides CMD

# ENTRYPOINT: user can only add arguments
ENTRYPOINT ["python"]
CMD ["app.py"]                       # Default argument
docker run myapp other.py            # Runs: python other.py
```

**20. Both CMD and ENTRYPOINT?**
ENTRYPOINT is the executable, CMD provides default arguments:
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run myapp           → python app.py
# docker run myapp test.py   → python test.py
```

**21. Shell form vs exec form?**
```dockerfile
# Shell form (runs via /bin/sh -c)
RUN apt-get update && apt-get install -y curl
CMD python app.py

# Exec form (direct execution, preferred)
RUN ["apt-get", "update"]
CMD ["python", "app.py"]
```
Exec form: no shell processing, proper signal handling (SIGTERM). **Prefer exec form for CMD/ENTRYPOINT.**

**22. .dockerignore?**
Excludes files from build context (faster builds, smaller context, security):
```
.git
node_modules
*.md
.env
__pycache__
*.pyc
.vscode
Dockerfile
docker-compose.yml
```

**23. ARG vs ENV?**
- **ARG**: Available only during build. Not in running container. `--build-arg VAR=value`
- **ENV**: Available during build AND in running container.
```dockerfile
ARG BUILD_VERSION=1.0              # Build-only
ENV APP_VERSION=${BUILD_VERSION}   # Persists to runtime
```

**24. HEALTHCHECK?**
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```
Docker periodically runs this command. Container marked unhealthy if it fails.

**25. USER instruction? Why not root?**
```dockerfile
RUN addgroup --system app && adduser --system --ingroup app appuser
USER appuser
```
Running as root is a security risk: if container is compromised, attacker has root access to container (and potentially host via escape).

---

## Interview-Style (Basics)

**26. Walk through a Dockerfile?**
```dockerfile
FROM python:3.11-slim                    # Minimal Python base image
WORKDIR /app                             # All commands run from /app
COPY requirements.txt .                  # Copy deps first (cache optimization)
RUN pip install --no-cache-dir -r requirements.txt  # Install deps (cached if req unchanged)
COPY . .                                 # Copy application code
RUN adduser --system appuser             # Create non-root user
USER appuser                             # Switch to non-root
EXPOSE 8080                              # Document the port
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]  # Start app
```

**27. Image is 2GB — reduce size?**
1. Use smaller base: `python:3.11-slim` (150MB) instead of `python:3.11` (900MB)
2. Multi-stage build: build in one stage, copy only artifacts to final
3. Combine RUN commands: fewer layers, clean up in same layer
4. `.dockerignore`: exclude .git, node_modules, tests
5. `--no-cache-dir` for pip
6. Remove package manager cache: `apt-get clean && rm -rf /var/lib/apt/lists/*`
7. Use Alpine or distroless for final image

**28. `docker run` vs `docker exec`?**
- `docker run`: Creates and starts a **new** container from an image
- `docker exec`: Runs a command in an **existing** running container

**29. Debug container that exits immediately?**
```bash
# 1. Check exit code
docker ps -a                       # STATUS shows exit code

# 2. Check logs
docker logs <container>

# 3. Run interactively
docker run -it <image> /bin/sh     # Override CMD with shell

# 4. Check ENTRYPOINT/CMD
docker inspect <image> | grep -A5 "Cmd\|Entrypoint"

# 5. Common causes: missing env vars, wrong CMD, missing files, permission errors
```

**30. Dockerfile for Python Flask app?**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN adduser --system --no-create-home appuser
USER appuser
EXPOSE 5000
HEALTHCHECK --interval=30s CMD curl -f http://localhost:5000/health || exit 1
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

---

# PART 2: ADVANCED (50 Qs)

---

## Multi-Stage Builds

**1. Multi-stage build? Why?**
Multiple FROM statements in one Dockerfile. Build in one stage (with all tools), copy only final artifacts to clean final image. Result: much smaller images.

**2. Multi-stage for Go app?**
```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# Runtime stage
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/myapp /myapp
USER nonroot
ENTRYPOINT ["/myapp"]
# Result: ~10MB instead of 800MB
```

**3. Multi-stage for Node.js?**
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Runtime stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json .
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**4. Copy files between stages?**
```dockerfile
COPY --from=builder /app/output /app/
COPY --from=0 /app/binary /usr/local/bin/   # By stage index
```

**5. Name build stages?**
```dockerfile
FROM golang:1.21 AS builder     # Named "builder"
FROM node:20 AS frontend        # Named "frontend"
COPY --from=builder /app/api /
COPY --from=frontend /app/dist /static/
```

**6. How many stages? Limit?**
No practical limit. Use as many as needed. Common: 2-3 stages (build, test, runtime).

---

## Networking

**7. Docker networking modes?**
- **bridge**: Default. Containers on same bridge can communicate. Isolated from host.
- **host**: Container uses host's network directly. No isolation. Best performance.
- **none**: No networking. Completely isolated.
- **overlay**: Multi-host networking (Docker Swarm/K8s).
- **macvlan**: Container gets its own MAC address. Appears as physical device.

**8. Default network?**
`bridge` (docker0). All containers connect to it unless specified otherwise.

**9. Containers on same bridge network communicate?**
By container name (DNS). Custom bridge networks provide automatic DNS resolution.
```bash
docker network create mynet
docker run --network mynet --name web nginx
docker run --network mynet --name app myapp
# app can reach web at http://web:80
```

**10. Containers on different networks?**
They can't communicate by default. Solutions:
1. Connect container to both networks: `docker network connect net2 container1`
2. Use one shared network

**11. `docker network create`?**
```bash
docker network create mynet                    # Default bridge
docker network create --driver overlay mynet   # Overlay for swarm
docker network ls                              # List networks
docker network inspect mynet                   # Details
```

**12. EXPOSE vs `-p`?**
- `EXPOSE 8080`: Documentation only. Does NOT publish the port.
- `-p 8080:80`: Actually maps host port 8080 to container port 80.

**13. Port mapping?**
```bash
docker run -p 8080:80 nginx
# -p <host_port>:<container_port>
# Host port 8080 → Container port 80
docker run -p 127.0.0.1:8080:80 nginx   # Only localhost
docker run -P nginx                       # Map all EXPOSE ports to random host ports
```

**14. Container linking?**
```bash
# Legacy (deprecated)
docker run --link db:database myapp

# Modern: use custom networks (preferred)
docker network create mynet
docker run --network mynet --name db postgres
docker run --network mynet --name app myapp
```

---

## Storage

**15. Docker volume? Why not container filesystem?**
Container filesystem is ephemeral — destroyed with container. Volumes persist data beyond container lifecycle.

**16. Volume vs bind mount vs tmpfs?**
| Type | Location | Persistence | Use Case |
|---|---|---|---|
| Volume | Docker-managed (/var/lib/docker/volumes/) | Persists | Database data, shared data |
| Bind mount | Host path you specify | Persists | Dev: mount source code |
| tmpfs | RAM only | No | Sensitive temp data |

**17. Create and manage volumes?**
```bash
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata
docker run -v mydata:/app/data myapp      # Named volume
docker run -v /host/path:/container/path myapp  # Bind mount
```

**18. Share data between containers?**
```bash
# Shared volume
docker volume create shared
docker run -v shared:/data --name writer myapp
docker run -v shared:/data --name reader myapp2
```

**19. Named vs anonymous volume?**
```bash
docker run -v mydata:/data myapp         # Named: "mydata", easy to reference
docker run -v /data myapp               # Anonymous: random hash name, hard to manage
```

**20. Backup a Docker volume?**
```bash
docker run --rm -v mydata:/source -v $(pwd):/backup busybox \
    tar czf /backup/backup.tar.gz -C /source .
```

---

## Docker Compose

**21. Docker Compose? When use?**
Tool for defining multi-container apps in YAML. Use when: app needs multiple services (web + DB + cache), local development, testing.

**22. Compose: web app + PostgreSQL + Redis?**
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - app-net

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-net

  cache:
    image: redis:7-alpine
    networks:
      - app-net

volumes:
  pgdata:

networks:
  app-net:
```

**23. `docker-compose up` vs `docker-compose up -d`?**
- `docker-compose up`: Foreground, logs visible. Ctrl+C stops all.
- `docker-compose up -d`: Detached (background). Use `docker-compose logs` to see output.

**24. Scale a service?**
```bash
docker-compose up -d --scale web=3
# Or in compose file:
services:
  web:
    deploy:
      replicas: 3
```

**25. `depends_on`? Wait for ready?**
```yaml
depends_on:
  - db
```
**No, it does NOT wait for the service to be ready.** It only waits for the container to start. Use healthchecks:
```yaml
depends_on:
  db:
    condition: service_healthy
```

**26. Environment variables in Compose?**
```yaml
# 1. Inline
environment:
  - DB_HOST=postgres

# 2. .env file (auto-loaded)
# .env
DB_HOST=postgres

# 3. env_file
env_file:
  - ./config/.env.prod
```

**27. Compose v1 vs v2?**
- **v1**: `docker-compose` (separate Python binary). Legacy.
- **v2**: `docker compose` (built into Docker CLI). Faster, better features. **Use v2.**

---

## Security & Optimization

**28. 10 Docker security best practices?**
1. Use official/verified base images
2. Run as non-root user (`USER`)
3. Use multi-stage builds (minimize attack surface)
4. Scan images for vulnerabilities (Trivy, Snyk)
5. Don't store secrets in images (use runtime secrets)
6. Use `.dockerignore` to exclude sensitive files
7. Pin image versions (`python:3.11.7-slim` not `python:latest`)
8. Use read-only filesystem: `--read-only`
9. Limit capabilities: `--cap-drop ALL`
10. Use distroless/minimal base images

**29. Distroless image?**
Google's images containing ONLY the app runtime (no shell, no package manager, no tools). Minimal attack surface.
```dockerfile
FROM gcr.io/distroless/python3
COPY app.py /app.py
CMD ["app.py"]
```

**30. Alpine vs slim vs full?**
| Image | Size | Tools | Use |
|---|---|---|---|
| Full (`python:3.11`) | ~900MB | Everything | Dev/debugging |
| Slim (`python:3.11-slim`) | ~150MB | Minimal | Production (default choice) |
| Alpine (`python:3.11-alpine`) | ~50MB | musl libc, busybox | Size-critical (compatibility issues possible) |

**31. Scan images for vulnerabilities?**
```bash
# Trivy (free, comprehensive)
trivy image myapp:latest

# Snyk
snyk container test myapp:latest

# Docker Scout (built into Docker)
docker scout cves myapp:latest
```

**32. Layer caching? Optimize?**
Docker caches each layer. If a layer's instruction hasn't changed, it uses cache. **Put frequently changing instructions last:**
```dockerfile
FROM python:3.11-slim              # Rarely changes (cached)
COPY requirements.txt .            # Changes sometimes
RUN pip install -r requirements.txt # Cached if requirements unchanged
COPY . .                           # Changes every build (last!)
```

**33. Order instructions least→most frequently changing?**
Because Docker invalidates cache from the first changed layer onward. If you COPY source code before installing dependencies, dependencies reinstall every time code changes.

**34. BuildKit?**
Next-gen Docker build engine. Features: parallel builds, better caching, build secrets (`--mount=type=secret`), SSH forwarding, build output customization.
```bash
DOCKER_BUILDKIT=1 docker build .
# Or in daemon.json: { "features": { "buildkit": true } }
```

**35. `docker system prune`?**
```bash
docker system prune         # Remove stopped containers, unused networks, dangling images, build cache
docker system prune -a      # Also remove all unused images
docker system prune -a --volumes  # Also remove unused volumes
docker system df             # Check disk usage
```

**36. Memory and CPU limits?**
```bash
docker run --memory=512m --cpus=1.5 myapp
docker run -m 256m --memory-swap=512m myapp   # Memory + swap
# In Compose:
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
```

**37. Docker Content Trust?**
Ensures image integrity and publisher authentication. Uses digital signatures.
```bash
export DOCKER_CONTENT_TRUST=1
docker pull myimage:latest    # Only pulls signed images
```

**38. Prevent privilege escalation?**
```bash
docker run --security-opt=no-new-privileges --cap-drop ALL --read-only myapp
```
In Dockerfile: `USER nonroot`. In K8s: `allowPrivilegeEscalation: false`.

---

## Troubleshooting

**39. Container running but app not reachable?**
1. Check port mapping: `docker ps` → PORTS column
2. Check if app is binding to 0.0.0.0 (not 127.0.0.1)
3. Check container logs: `docker logs <container>`
4. Test from inside: `docker exec -it <container> curl localhost:8080`
5. Check firewall/security group rules
6. Check network: `docker network inspect <network>`

**40. Build fails at RUN step?**
1. Read the error message carefully
2. Run intermediate image: `docker run -it <last-successful-layer> /bin/sh`
3. Run the failing command manually inside container
4. Check network (can it download packages?)
5. Check file paths and permissions

**41. Images eating all disk space?**
```bash
docker system df                    # See what's using space
docker image prune -a               # Remove unused images
docker builder prune                # Clear build cache
docker volume prune                 # Remove unused volumes
```

**42. `docker logs` empty but container running?**
- App writes to file instead of stdout/stderr → `docker exec cat /app/logs/app.log`
- App uses logging framework that doesn't write to stdout → configure to log to stdout
- Logs are buffered → add `-u` flag for Python (`python -u`)

**43. Container using 100% CPU?**
```bash
docker stats                           # Live resource usage
docker top <container>                 # Processes inside container
docker exec -it <container> top        # top inside container
# Fix: set CPU limits, investigate application code
```

---

## Interview-Style (Advanced)

**44. Docker images in CI/CD?**
1. Build image in CI pipeline
2. Tag with build number/git SHA: `myapp:${BUILD_ID}`
3. Push to private registry (ACR/ECR)
4. Scan for vulnerabilities (Trivy)
5. Deploy by referencing specific tag
6. Keep `latest` tag for convenience, never use in production

**45. Image tagging strategy?**
```bash
myapp:1.2.3                    # Semantic version
myapp:20240115-abc1234          # Date + git SHA
myapp:${BUILD_BUILDID}          # CI build number
myapp:latest                    # Latest build (mutable, don't use in prod)
myapp:main-abc1234              # Branch + SHA
```
Best: **immutable tags** (semver or SHA). Never deploy `latest` to production.

**46. Containerized an existing app?**
"Analyzed the app's dependencies and runtime requirements. Created a Dockerfile starting with appropriate base image. Added dependency installation, copied application code, configured environment variables. Used multi-stage build to keep image small. Added health check. Tested locally, then integrated into CI/CD pipeline. Migrated from VM deployment to container orchestration on K8s."

**47. Docker vs Podman?**
| Docker | Podman |
|---|---|
| Daemon-based (dockerd) | Daemonless |
| Root by default | Rootless by default |
| docker CLI | podman CLI (compatible) |
| Docker Compose | podman-compose or pods |
| Industry standard | RHEL/Fedora default |

**48. OCI? OCI-compliant images?**
Open Container Initiative — standard specifications for container formats and runtimes. OCI-compliant images work across Docker, Podman, containerd, CRI-O. All modern tools produce OCI images.

**49. Secrets in Docker?**
```dockerfile
# Build-time (BuildKit)
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
docker build --secret id=mysecret,src=./secret.txt .

# Runtime
docker run -e SECRET_KEY=value myapp              # Env var (visible in inspect!)
docker run -v /secrets:/run/secrets:ro myapp       # Mount file (better)
# In K8s: use Secrets + volume mounts
```
**Never**: hardcode in Dockerfile, bake into image layers, use ARG for secrets.

**50. Docker Compose for 3 microservices?**
```yaml
version: '3.8'
services:
  api-gateway:
    build: ./api-gateway
    ports: ["8080:8080"]
    environment:
      USER_SERVICE_URL: http://user-service:3000
      ORDER_SERVICE_URL: http://order-service:4000
    depends_on: [user-service, order-service]
    networks: [backend]

  user-service:
    build: ./user-service
    environment:
      DB_HOST: user-db
    depends_on: [user-db]
    networks: [backend]

  order-service:
    build: ./order-service
    environment:
      DB_HOST: order-db
    depends_on: [order-db]
    networks: [backend]

  user-db:
    image: postgres:15
    environment:
      POSTGRES_DB: users
      POSTGRES_PASSWORD: pass
    volumes: [user-data:/var/lib/postgresql/data]
    networks: [backend]

  order-db:
    image: postgres:15
    environment:
      POSTGRES_DB: orders
      POSTGRES_PASSWORD: pass
    volumes: [order-data:/var/lib/postgresql/data]
    networks: [backend]

volumes:
  user-data:
  order-data:

networks:
  backend:
```
