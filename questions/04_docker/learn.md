# Docker — Deep-Dive Learning Guide

---

## 1. What Is a Container?

A container is an **isolated process** (or group of processes) running on the host OS kernel, constrained by:

- **Namespaces** — what the process can **see** (its own PID tree, network stack, mount points, users)
- **Cgroups** — what the process can **use** (CPU, memory, disk I/O limits)
- **Union Filesystem** — layered read-only image + thin writable layer on top

```
┌──────────────────────────────────────────┐
│              Container                    │
│  ┌──────────────────────────────────┐    │
│  │  Application Process (PID 1)     │    │
│  │  Libraries, Bins, Config         │    │
│  └──────────────────────────────────┘    │
│  ┌──────────────────────────────────┐    │
│  │  Writable Layer (container layer)│    │ ← copy-on-write
│  ├──────────────────────────────────┤    │
│  │  Image Layer 3 (COPY src/)       │    │ ← read-only
│  ├──────────────────────────────────┤    │
│  │  Image Layer 2 (RUN pip install) │    │ ← read-only
│  ├──────────────────────────────────┤    │
│  │  Image Layer 1 (FROM python)     │    │ ← read-only
│  └──────────────────────────────────┘    │
│  Namespaces: pid, net, mnt, uts, ipc     │
│  Cgroups: cpu=500m, memory=512Mi         │
└──────────────────────────────────────────┘
         │
    ┌────┴────┐
    │  Host   │
    │  Linux  │  ← shared kernel (no guest OS!)
    │  Kernel │
    └─────────┘
```

### Container vs VM

```
    Virtual Machines                    Containers
┌──────┐  ┌──────┐              ┌──────┐  ┌──────┐
│ App1 │  │ App2 │              │ App1 │  │ App2 │
│ Bins │  │ Bins │              │ Bins │  │ Bins │
│GuestOS│ │GuestOS│             └──┬───┘  └──┬───┘
└──┬───┘  └──┬───┘                 │         │
   │         │                  ┌──┴─────────┴──┐
┌──┴─────────┴──┐               │Container Runtme│
│  Hypervisor   │               ├───────────────┤
├───────────────┤               │   Host OS     │
│   Host OS     │               ├───────────────┤
├───────────────┤               │   Hardware    │
│   Hardware    │               └───────────────┘
└───────────────┘
```

| Aspect | Container | VM |
|--------|-----------|-----|
| Size | MBs (10-200MB typical) | GBs (1-20GB) |
| Start time | Milliseconds to seconds | Minutes |
| Isolation | Process-level (namespaces) | Full hardware-level (hypervisor) |
| Kernel | Shares host kernel | Own kernel per VM |
| Overhead | ~1-5% | ~15-30% |
| Density | 100s per host | 10s per host |
| Portability | Very high (OCI standard) | Medium (VM format varies) |

---

## 2. OCI (Open Container Initiative)

OCI is a Linux Foundation project that defines **open standards** so containers are portable across all tools (Docker, Podman, Kubernetes, etc.).

```
┌─────────────────────────────────────────────────┐
│                OCI Standards                     │
│                                                  │
│  ┌─────────────────┐  ┌──────────────────────┐  │
│  │  OCI Image Spec │  │  OCI Runtime Spec    │  │
│  │  - Manifest     │  │  - config.json       │  │
│  │  - Layers (tar) │  │  - rootfs path       │  │
│  │  - Config JSON  │  │  - mounts, hooks     │  │
│  │  - Index        │  │  - namespaces list   │  │
│  └─────────────────┘  │  - cgroups config    │  │
│                       └──────────────────────┘  │
│  ┌──────────────────────────────────────────┐   │
│  │  OCI Distribution Spec                   │   │
│  │  - Push/Pull API for registries          │   │
│  │  - Content discovery, manifest retrieval │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

**Why it matters**: Without OCI, a Docker image wouldn't work in Kubernetes or Podman. OCI is the contract that makes them interchangeable.

---

## 3. Container Runtime — Two Layers

```
┌───────────────────────────────────────────────┐
│  High-Level Runtime (Container Manager)        │
│  containerd / CRI-O                            │
│  - Pulls images from registry                  │
│  - Manages container lifecycle                 │
│  - Provides API (gRPC) for orchestrators       │
│  - Creates containerd-shim per container       │
└───────────────────┬───────────────────────────┘
                    │ calls
┌───────────────────▼───────────────────────────┐
│  Low-Level Runtime (OCI Runtime)               │
│  runc / crun / kata-runtime                    │
│  - Reads OCI config.json                       │
│  - Calls Linux kernel: clone(), unshare()      │
│  - Creates namespaces, sets cgroups             │
│  - Mounts rootfs                                │
│  - Execs container process (PID 1)              │
│  - Exits after spawning (stateless!)            │
└───────────────────────────────────────────────┘
```

| Layer | What It Does | Examples |
|-------|-------------|----------|
| **High-level** | Image mgmt, API, networking, shim mgmt | containerd, CRI-O |
| **Low-level** | Talks to Linux kernel (cgroups, namespaces) to spawn container | runc, crun, kata |

**Key insight**: `runc` is **stateless** — it creates the container and exits. The `containerd-shim` stays as the parent, collecting exit codes and keeping STDIO open even if containerd restarts.

---

## 4. Docker Architecture — Client-Server Model

### Modern Docker Architecture (post-2017)

```
┌──────────┐                     ┌──────────────────────────────────────┐
│          │    REST API          │        Docker Daemon (dockerd)       │
│  docker  │────────────────────►│                                      │
│   CLI    │  /var/run/docker.sock│  Manages: images, volumes, networks │
│ (Client) │  or TCP :2375/2376  │  REST API server                     │
│          │                     │                                      │
└──────────┘                     │  ┌──────────────────────────────┐    │
                                 │  │       containerd             │    │
                                 │  │  (high-level runtime, CNCF)  │    │
                                 │  │  - Image pull/push           │    │
                                 │  │  - Container lifecycle       │    │
                                 │  └─────────┬────────────────────┘    │
                                 │            │ per container           │
                                 │  ┌─────────▼────────────────────┐    │
                                 │  │    containerd-shim           │    │
                                 │  │  - Stays alive as parent     │    │
                                 │  │  - Allows containerd restart │    │
                                 │  │  - Collects exit codes       │    │
                                 │  └─────────┬────────────────────┘    │
                                 │            │                         │
                                 │  ┌─────────▼────────────────────┐    │
                                 │  │        runc                  │    │
                                 │  │  (low-level OCI runtime)     │    │
                                 │  │  - Creates namespaces/cgroups│    │
                                 │  │  - Starts container PID 1    │    │
                                 │  │  - Exits immediately after   │    │
                                 │  └──────────────────────────────┘    │
                                 └──────────────────────────────────────┘
```

### Why the breakup happened (2016-2017)

Docker was monolithic. Kubernetes didn't need the CLI, build, or Swarm — just the runtime. Docker donated:
- **containerd** → to CNCF (so K8s could use it directly)
- **runc** → to OCI (so any tool could use it)

### Complete Flow: `docker run nginx`

```
Step 1:  docker CLI ──► POST /containers/create ──► dockerd
Step 2:  dockerd checks local image cache, pulls from registry if missing
Step 3:  dockerd ──► containerd (gRPC: create container)
Step 4:  containerd ──► spawns containerd-shim
Step 5:  containerd-shim ──► forks runc
Step 6:  runc ──► clone() + unshare() → namespaces + cgroups
Step 7:  runc ──► pivots root to image rootfs
Step 8:  runc ──► exec nginx (PID 1 inside container)
Step 9:  runc exits (job done!)
Step 10: containerd-shim stays as parent of nginx process
Step 11: dockerd returns container ID to CLI
```

---

## 5. Docker on Non-Linux Systems

Docker containers need a **Linux kernel**. On macOS/Windows:

```
┌──────────────────────────────────────────┐
│  macOS / Windows                          │
│  ┌────────────────────────────────────┐  │
│  │  Lightweight Linux VM              │  │
│  │  (HyperKit on Mac, WSL2 on Win)   │  │
│  │  ┌──────────────────────────────┐ │  │
│  │  │  dockerd + containerd + runc │ │  │
│  │  │  All containers run HERE     │ │  │
│  │  └──────────────────────────────┘ │  │
│  │  Linux Kernel                     │  │
│  └────────────────────────────────────┘  │
│  docker CLI talks to VM via socket       │
└──────────────────────────────────────────┘
```

---

## 6. Dockerfile — Complete Reference

```dockerfile
# ─── Stage 1: Build ───
FROM python:3.11-slim AS builder
LABEL maintainer="devops@example.com"

ARG APP_VERSION=1.0                 # Build-time only
ENV APP_ENV=production              # Available at runtime

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ ./src/

# ─── Stage 2: Production ───
FROM python:3.11-slim AS production
WORKDIR /app
COPY --from=builder /app /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages

RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["python"]
CMD ["src/main.py"]
```

### Instruction Reference

| Instruction | Purpose | Build or Runtime? |
|-------------|---------|-------------------|
| `FROM` | Base image (starts new stage) | Build |
| `ARG` | Build-time variable (not in final image) | Build only |
| `ENV` | Environment variable (persists in image) | Both |
| `WORKDIR` | Set working directory | Both |
| `COPY` | Copy files from context into image | Build |
| `ADD` | Like COPY + extract tars + fetch URLs | Build |
| `RUN` | Execute command, commit as new layer | Build |
| `EXPOSE` | Document port (doesn't publish) | Documentation |
| `USER` | Set user for subsequent instructions | Both |
| `HEALTHCHECK` | Define health check command | Runtime |
| `ENTRYPOINT` | Fixed executable | Runtime |
| `CMD` | Default arguments (easily overridden) | Runtime |

### ENTRYPOINT vs CMD

```
ENTRYPOINT = the "executable"     CMD = the "default arguments"

docker run image          → ENTRYPOINT + CMD
docker run image arg1     → ENTRYPOINT + arg1 (CMD replaced)
docker run --entrypoint sh image  → overrides ENTRYPOINT
```

| Scenario | ENTRYPOINT | CMD | `docker run img` | `docker run img test.py` |
|----------|-----------|-----|-------------------|--------------------------|
| CMD only | — | `["python","app.py"]` | `python app.py` | `test.py` |
| ENTRY only | `["python"]` | — | `python` | `python test.py` |
| Both | `["python"]` | `["app.py"]` | `python app.py` | `python test.py` |

**Shell vs Exec form**: Always use exec form `["python","app.py"]`. Shell form wraps in `/bin/sh -c` — PID 1 becomes shell, signals don't reach your app!

---

## 7. Layer Caching

```
FROM python:3.11-slim               Layer 1  ✅ cached
COPY requirements.txt .             Layer 2  ✅ cached if file unchanged
RUN pip install -r requirements.txt Layer 3  ✅ cached if Layer 2 cached
COPY src/ ./src/                    Layer 4  ❌ INVALIDATED (src changed)
RUN compileall src/                 Layer 5  ❌ rebuilt (Layer 4 changed)
```

**Rules**: Order least→most changing. Use `.dockerignore`. Combine RUN commands. Use `--no-cache-dir`.

### Multi-Stage Builds

```
Without multi-stage:  python:3.11 + gcc + pip → ~1.2GB (includes build tools)
With multi-stage:     python:3.11-slim + COPY --from=builder → ~150MB ✅
```

---

## 8. Docker Networking

```
┌────────── bridge (docker0) ──────────────┐
│  172.17.0.1                               │
│  ┌───────────┐    ┌───────────┐          │
│  │Container A│    │Container B│          │
│  │172.17.0.2 │    │172.17.0.3 │          │
│  └───────────┘    └───────────┘          │
└──────────────────┬───────────────────────┘
                   │ NAT (iptables)
            Host Network Stack → Internet
```

| Mode | Description | Use Case |
|------|-------------|----------|
| `bridge` | Default, private subnet, NAT | Most apps |
| `host` | Shares host network stack | Max performance |
| `none` | No networking | Security isolation |
| `overlay` | Multi-host via VXLAN | Swarm/multi-host |
| `macvlan` | Own MAC on physical network | Legacy apps |

**User-defined bridge** gives DNS resolution by container name. Default bridge does NOT.

---

## 9. Docker Volumes & Storage

| Type | Managed by | Persists? | Use Case |
|------|-----------|-----------|----------|
| Named Volume | Docker | Yes | DB data, app state |
| Bind Mount | You | Yes | Dev: mount source code |
| tmpfs | Kernel | No (RAM) | Secrets, temp cache |

```bash
docker run -v mydata:/app/data       # Named volume
docker run -v /host/path:/app/data   # Bind mount
docker run --tmpfs /app/cache        # tmpfs
```

---

## 10. Docker Compose

```yaml
version: "3.9"
services:
  web:
    build: ./web
    ports: ["8080:80"]
    environment: [DB_HOST=db]
    depends_on:
      db: { condition: service_healthy }
    networks: [frontend, backend]

  db:
    image: postgres:15
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
    networks: [backend]

volumes:
  pgdata:
networks:
  frontend:
  backend:
```

```bash
docker compose up -d       # Start all
docker compose down        # Stop + remove
docker compose logs -f web # Follow logs
docker compose exec web sh # Shell into container
```

---

## 11. Docker Security Checklist

```
Image:   ✅ Specific tags (never :latest)  ✅ Slim/distroless base
         ✅ Multi-stage builds             ✅ Scan with Trivy/Snyk
         ✅ .dockerignore                  ✅ No secrets in image

Runtime: ✅ USER 1001 (non-root)           ✅ --read-only rootfs
         ✅ --cap-drop ALL                 ✅ --memory/--cpus limits
         ✅ No --privileged                ✅ DOCKER_CONTENT_TRUST=1
```

---

## 12. K8s Removed Docker — Why?

```
Before K8s 1.24:  kubelet → dockershim → dockerd → containerd → runc
After K8s 1.24:   kubelet → CRI → containerd → runc  (Docker removed!)
```

K8s only needed containerd. Docker added unnecessary layers (API, CLI, Swarm). Your Docker images still work — they're OCI-compliant.

---

## 13. Docker vs Podman

| Feature | Docker | Podman |
|---------|--------|--------|
| Daemon | Yes (dockerd) | Daemonless |
| Root | Requires root (or rootless) | Rootless by default |
| CLI | `docker` | `podman` (drop-in) |
| Pods | No | Yes (like K8s) |

---

## 14. Debugging Containers

```bash
docker logs <ctr>              # App logs
docker inspect <ctr>           # Config, state, exit code
docker exec -it <ctr> sh       # Shell in
docker stats <ctr>             # Live CPU/memory
docker top <ctr>               # Processes
docker diff <ctr>              # Changed files
docker history <image>         # Layer history
```

---

## 15. Container Lifecycle & Essential Commands

```
Container States:
  Created ──► Running ──► Paused ──► Running ──► Stopped ──► Removed
     │            │           │                      │
  docker       docker     docker                  docker
  create       start      pause/unpause           stop/kill

Full Lifecycle:
  docker create --name myapp nginx     # Create (not started)
  docker start myapp                   # Start
  docker pause myapp                   # Freeze (SIGSTOP via cgroups)
  docker unpause myapp                 # Resume
  docker stop myapp                    # Graceful (SIGTERM → 10s → SIGKILL)
  docker kill myapp                    # Immediate (SIGKILL)
  docker restart myapp                 # stop + start
  docker rm myapp                      # Remove stopped container
  docker rm -f myapp                   # Force remove running container
```

**`docker commit`** — Create image from running container's changes:
```bash
docker exec -it myapp bash
# make changes inside container...
apt-get update && apt-get install -y curl

docker commit myapp myapp-with-curl:v1
# Creates new image from container's current state
# ❌ Not recommended for production — use Dockerfiles instead
# ✅ Useful for debugging/experimenting
```

**`docker save` / `docker load`** — Transfer images as tarballs:
```bash
docker save myapp:v1 -o myapp-v1.tar     # Export image to file
docker load -i myapp-v1.tar               # Import image from file

# Use case: air-gapped environments, offline transfer
```

**`docker export` / `docker import`** — Container filesystem:
```bash
docker export mycontainer -o container-fs.tar  # Export container filesystem
docker import container-fs.tar myimage:v1      # Import as flat image (1 layer)

# Difference from save/load:
#   save/load  → preserves layers, metadata, tags
#   export/import → flattens to single layer, loses history
```

---

## 16. docker inspect — Querying Container/Image Metadata

```bash
# Full JSON output
docker inspect mycontainer

# Specific fields with Go template:
docker inspect --format '{{.State.Status}}' mycontainer          # running
docker inspect --format '{{.NetworkSettings.IPAddress}}' myapp   # 172.17.0.2
docker inspect --format '{{.State.ExitCode}}' myapp              # 0
docker inspect --format '{{.Config.Env}}' myapp                  # [PATH=... DB_HOST=db]
docker inspect --format '{{.HostConfig.Memory}}' myapp           # 536870912 (bytes)
docker inspect --format '{{json .Mounts}}' myapp | jq .          # Volume mounts
docker inspect --format '{{.State.StartedAt}}' myapp             # Timestamp

# Check if container is healthy
docker inspect --format '{{.State.Health.Status}}' myapp         # healthy

# Get image layers
docker inspect --format '{{json .RootFS.Layers}}' nginx:latest | jq .
```

---

## 17. BuildKit — Modern Docker Build Engine

```
BuildKit = next-gen build backend (default since Docker 23.0)

  DOCKER_BUILDKIT=1 docker build .     # Explicitly enable (older Docker)

  Key Benefits:
  ┌──────────────────────────────────────────────────┐
  │ ✅ Parallel stage builds (multi-stage faster)    │
  │ ✅ Better caching (mount cache for pip/npm)      │
  │ ✅ Build secrets (--secret, not baked into image)│
  │ ✅ SSH forwarding for private repo access        │
  │ ✅ Smaller build context transfer                │
  │ ✅ Colored progress output                       │
  └──────────────────────────────────────────────────┘
```

**Cache mounts** — Persist package manager cache across builds:
```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
COPY . .
```

**Build secrets** — Use secrets without leaking into layers:
```dockerfile
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci --production
```
```bash
docker build --secret id=npmrc,src=.npmrc .
```

---

## 18. Advanced Docker Compose

```yaml
# docker-compose.yml with advanced features
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        BUILD_ENV: production
      cache_from:
        - myregistry/api:cache
    image: myregistry/api:${TAG:-latest}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      replicas: 3
      restart_policy:
        condition: on-failure
        max_attempts: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    profiles: ["production"]          # Only start with --profile production

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes

volumes:
  redis-data:
    driver: local
```

**Compose profiles** — Group services:
```bash
docker compose --profile production up    # Start services with "production" profile
docker compose --profile debug up         # Start services with "debug" profile
```

**Override files** — Environment-specific config:
```bash
# docker-compose.yml          → base config
# docker-compose.override.yml → auto-merged (local dev)
# docker-compose.prod.yml     → explicit merge for prod

docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## 19. Docker CLI Cheat Sheet

```
┌─── Images ───────────────────────────────────────────────┐
│ docker build -t name:tag .       Build from Dockerfile   │
│ docker images                    List images             │
│ docker rmi image                 Remove image            │
│ docker image prune -a            Remove unused images    │
│ docker tag src:v1 dest:v1        Tag/rename image        │
│ docker push registry/name:tag    Push to registry        │
│ docker pull registry/name:tag    Pull from registry      │
└──────────────────────────────────────────────────────────┘

┌─── Containers ───────────────────────────────────────────┐
│ docker run -d --name n img       Run detached            │
│ docker run -it img sh            Interactive shell       │
│ docker ps                        List running            │
│ docker ps -a                     List all (inc stopped)  │
│ docker stop/start/restart ctr    Manage state            │
│ docker rm ctr                    Remove stopped          │
│ docker container prune           Remove all stopped      │
└──────────────────────────────────────────────────────────┘

┌─── System ───────────────────────────────────────────────┐
│ docker system df                 Disk usage              │
│ docker system prune -a           Remove everything unused│
│ docker info                      System-wide info        │
│ docker version                   Client/server version   │
└──────────────────────────────────────────────────────────┘
```
