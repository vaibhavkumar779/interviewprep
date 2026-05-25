# Docker — COMPREHENSIVE ANSWERS (All 80 Questions)

---

# PART 1: BASICS & FUNDAMENTALS (30 Qs)

---

## Core Concepts

**1. What is Docker? What problem does it solve?**

Docker is a platform for building, shipping, and running applications in **containers** — lightweight, isolated environments that package an app with ALL its dependencies.

```
Without Docker:                        With Docker:
┌──────────────────┐                  ┌──────────────────┐
│   Dev Machine    │                  │   Dev Machine    │
│  Python 3.8      │                  │  ┌────────────┐  │
│  Node 14         │                  │  │ Container  │  │
│  PostgreSQL 12   │                  │  │ App + Deps │  │
│  Works here! ✅  │                  │  └────────────┘  │
└──────────────────┘                  └──────────────────┘
        │                                     │
┌──────────────────┐                  ┌──────────────────┐
│   Production     │                  │   Production     │
│  Python 3.11     │                  │  ┌────────────┐  │
│  Node 18         │                  │  │ Container  │  │
│  PostgreSQL 15   │                  │  │ App + Deps │  │
│  BROKEN! ❌      │                  │  │ Same image!│  │
│  "works on my    │                  │  └────────────┘  │
│   machine" 🤦    │                  │  Works here! ✅  │
└──────────────────┘                  └──────────────────┘
```

**Problems solved:** dependency conflicts, environment inconsistency, "works on my machine", slow onboarding, inconsistent deployments.

---

**2. Container vs VM?**

```
┌─── Virtual Machine ──────────────┐    ┌─── Container ───────────────────┐
│  ┌────────┐ ┌────────┐          │    │  ┌────────┐ ┌────────┐         │
│  │ App A  │ │ App B  │          │    │  │ App A  │ │ App B  │         │
│  ├────────┤ ├────────┤          │    │  ├────────┤ ├────────┤         │
│  │ Bins/  │ │ Bins/  │          │    │  │ Bins/  │ │ Bins/  │         │
│  │ Libs   │ │ Libs   │          │    │  │ Libs   │ │ Libs   │         │
│  ├────────┤ ├────────┤          │    │  └───┬────┘ └───┬────┘         │
│  │Guest OS│ │Guest OS│ ← FULL  │    │      │          │               │
│  │(Ubuntu)│ │(CentOS)│   OS!   │    │  ┌───▼──────────▼───┐          │
│  └────────┘ └────────┘          │    │  │ Container Runtime│          │
│  ┌──────────────────────┐       │    │  │ (containerd)     │          │
│  │     Hypervisor       │       │    │  └──────────────────┘          │
│  │  (VMware/Hyper-V)    │       │    │  ┌──────────────────┐          │
│  └──────────────────────┘       │    │  │  Host OS Kernel  │ ← SHARED│
│  ┌──────────────────────┐       │    │  └──────────────────┘          │
│  │     Host OS          │       │    │  ┌──────────────────┐          │
│  └──────────────────────┘       │    │  │    Hardware      │          │
│  ┌──────────────────────┐       │    │  └──────────────────┘          │
│  │     Hardware         │       │    └──────────────────────────────────┘
│  └──────────────────────┘       │
└──────────────────────────────────┘
```

| Aspect | Container | VM |
|--------|-----------|-----|
| Isolation | Process-level (namespaces/cgroups) | Hardware-level (hypervisor) |
| Size | MBs (5-500MB) | GBs (1-20GB) |
| Boot time | Seconds | Minutes |
| OS | Shares host kernel | Own OS kernel |
| Overhead | Near-native performance | 5-20% overhead |
| Density | 100s per host | 10-20 per host |
| Security | Process isolation (weaker) | Hardware isolation (stronger) |
| Tools | Docker, Podman, containerd | VMware, VirtualBox, Hyper-V, KVM |

---

**3. Docker image vs container?**

```
┌─── Image (Blueprint) ───────────┐    ┌─── Container (Instance) ────────┐
│                                  │    │                                  │
│  Read-only template              │    │  Running instance of image       │
│  Layered filesystem              │    │  Has writable layer on top       │
│  Shareable, storable, versioned  │    │  Isolated process with state     │
│  Stored in registry              │    │  Ephemeral (data lost on delete) │
│                                  │    │                                  │
│  Like a CLASS in programming     │    │  Like an OBJECT/INSTANCE         │
│                                  │    │                                  │
│  ┌──────────────────┐           │    │  ┌──────────────────────┐       │
│  │ Layer: COPY app  │           │    │  │ Writable Layer (R/W) │ ← new│
│  ├──────────────────┤           │    │  ├──────────────────────┤       │
│  │ Layer: RUN pip   │           │    │  │ Layer: COPY app (RO) │       │
│  ├──────────────────┤           │    │  ├──────────────────────┤       │
│  │ Layer: FROM python│          │    │  │ Layer: RUN pip  (RO) │       │
│  └──────────────────┘           │    │  ├──────────────────────┤       │
│                                  │    │  │ Layer: FROM     (RO) │       │
│  One image → many containers     │    │  └──────────────────────┘       │
└──────────────────────────────────┘    └──────────────────────────────────┘
```

---

**4. Docker Engine components?**

```
┌─── Docker Engine Architecture ──────────────────────────────────┐
│                                                                  │
│  ┌──────────┐     REST API      ┌──────────────┐               │
│  │ Docker   │ ──────────────►   │ Docker       │               │
│  │ CLI      │  /var/run/docker  │ Daemon       │               │
│  │ (client) │  .sock            │ (dockerd)    │               │
│  └──────────┘                   └──────┬───────┘               │
│                                        │                        │
│                                 ┌──────▼───────┐               │
│                                 │ containerd   │ Container     │
│                                 │              │ lifecycle     │
│                                 └──────┬───────┘               │
│                                        │                        │
│                                 ┌──────▼───────┐               │
│                                 │ runc         │ OCI runtime   │
│                                 │              │ (starts       │
│                                 │              │  containers)  │
│                                 └──────────────┘               │
└──────────────────────────────────────────────────────────────────┘
```

- **Docker daemon (dockerd)**: Background service — manages images, containers, networks, volumes
- **Docker CLI**: Command-line tool sends commands to daemon via REST API
- **containerd**: High-level container runtime — manages container lifecycle
- **runc**: Low-level OCI runtime — actually creates the container process

---

**5. Docker daemon vs Docker CLI?**

| Component | Role | Where |
|-----------|------|-------|
| Docker CLI | Client — sends commands | User's terminal |
| Docker daemon | Server — does the work | Background process |

Communication via REST API over Unix socket (`/var/run/docker.sock`) or TCP. CLI and daemon can run on different machines (remote Docker).

---

**6. Docker registry? Name 3.**

Storage and distribution service for Docker images. Like GitHub for container images.

```
docker push ──► Registry ──► docker pull
                  │
    ┌─────────────┼─────────────┐
    │             │             │
 Docker Hub   ACR (Azure)    ECR (AWS)
 (default)    Harbor (self-hosted)
              GHCR (GitHub)
```

| Registry | Type | Provider |
|----------|------|----------|
| Docker Hub | Public/Private | Docker Inc |
| Azure Container Registry (ACR) | Private | Microsoft |
| Amazon ECR | Private | AWS |
| GitHub Container Registry (ghcr.io) | Public/Private | GitHub |
| Harbor | Self-hosted | Open Source |

---

**7. Docker Hub? Private registry?**

Docker Hub is the default public registry. Free for public images. Private registries: ACR, ECR, Harbor, Docker Hub paid plans, or self-hosted with `docker run -d -p 5000:5000 registry:2`.

---

**8. Container lifecycle?**

```
         docker create           docker start
Created ──────────────► Stopped ──────────────► Running
                            ▲                      │ │
                            │      docker stop     │ │ docker pause
                            │ ◄────────────────────┘ │
                            │                        ▼
                            │      docker unpause  Paused
                            │ ◄──────────────────────┘
                            │
                      docker rm ──► Removed (deleted)
```

```bash
docker create nginx        # Created (not running)
docker start <id>          # Running
docker pause <id>          # Paused (frozen, still in memory)
docker unpause <id>        # Running again
docker stop <id>           # Graceful stop: SIGTERM → 10s → SIGKILL
docker kill <id>           # Force stop: SIGKILL immediately
docker rm <id>             # Remove container
docker run nginx           # create + start in one command
```

---

**8b. Create image from running container (`docker commit`):**

```
Running Container                    New Image
┌──────────────────────────┐        ┌──────────────────────────┐
│ nginx container          │        │ my-custom-nginx:v1       │
│                          │        │                          │
│ + installed vim          │ commit │ Base nginx layers        │
│ + edited nginx.conf      │──────►│ + vim layer              │
│ + added custom html      │        │ + config changes layer   │
│                          │        │ + html layer             │
└──────────────────────────┘        └──────────────────────────┘
```

```bash
# 1. Run and modify a container
docker run -it --name mycontainer nginx bash
# ... install packages, edit configs, etc.

# 2. Create image from modified container
docker commit mycontainer my-custom-nginx:v1

# With author and commit message
docker commit -a "Vaibhav" -m "Added custom config" mycontainer my-custom-nginx:v1

# With config changes (change CMD, expose port, etc.)
docker commit --change='CMD ["nginx", "-g", "daemon off;"]' mycontainer my-custom-nginx:v1

# ⚠️ NOT recommended for production — use Dockerfile instead!
# docker commit creates opaque layers (not reproducible)
# Use only for: quick debugging snapshots, one-off experiments
```

---

**8c. Save/Load images as tar (image backup & transfer):**

```
docker save vs docker export:

┌─── docker save (IMAGE → tar) ──────────┐   ┌─── docker export (CONTAINER → tar) ──┐
│                                          │   │                                       │
│ Saves FULL image with ALL layers,       │   │ Saves container filesystem as         │
│ tags, and metadata                       │   │ FLAT tar (single layer, no history)   │
│                                          │   │                                       │
│ docker save -o myapp.tar myapp:v1       │   │ docker export -o backup.tar container │
│ docker save myapp:v1 | gzip > myapp.tgz │   │ docker export container > backup.tar  │
│                                          │   │                                       │
│ Restore: docker load -i myapp.tar       │   │ Restore: docker import backup.tar     │
│          docker load < myapp.tgz        │   │          myapp:restored               │
│                                          │   │                                       │
│ Use for: offline transfer, air-gapped   │   │ Use for: filesystem backup,           │
│ environments, registry migration         │   │ creating minimal base images          │
└──────────────────────────────────────────┘   └───────────────────────────────────────┘
```

```bash
# ── docker save: Image → tar (preserves layers + metadata) ──
docker save -o myapp.tar myapp:v1                   # Save single image
docker save myapp:v1 myapp:v2 nginx:latest -o all.tar  # Save multiple images
docker save myapp:v1 | gzip > myapp.tar.gz          # Compressed

# Transfer to another machine (air-gapped, no registry)
scp myapp.tar.gz user@remote-host:/tmp/

# ── docker load: tar → Image (restore from save) ──
docker load -i myapp.tar                             # Load from file
docker load < myapp.tar.gz                           # Load from stdin
# Images appear with original names and tags

# ── docker export: Container filesystem → tar (flat, no layers) ──
docker export mycontainer -o container-fs.tar
docker export mycontainer > container-fs.tar

# ── docker import: tar → Image (from export, creates single layer) ──
docker import container-fs.tar myapp:imported
cat container-fs.tar | docker import - myapp:imported

# ── Quick Reference ──
# save/load  = IMAGE level (preserves layers, tags, history)
# export/import = CONTAINER level (flat filesystem, no history)
```

---

**9. Docker layer? How layering works?**

Each Dockerfile instruction creates a read-only layer. Layers are cached and shared between images.

```
Dockerfile:                    Resulting Layers:
                               ┌──────────────────────────┐
COPY . .              ──►      │ Layer 4: app code (5 MB) │ ← changes often
                               ├──────────────────────────┤
RUN pip install       ──►      │ Layer 3: deps (150 MB)   │ ← cached if req same
                               ├──────────────────────────┤
COPY requirements.txt ──►     │ Layer 2: req file (1 KB) │
                               ├──────────────────────────┤
FROM python:3.11-slim ──►     │ Layer 1: base (150 MB)   │ ← shared across images
                               └──────────────────────────┘

Key: Only changed layers rebuild. Layers below a change → cached ✅
     Layers above a change → must rebuild ❌
     Order matters for build speed!
```

---

**10. Union filesystem?**

Combines multiple read-only layers into a single unified filesystem view. Container adds a thin writable layer on top using **copy-on-write (CoW)**.

```
What container sees:              How it's actually stored:
┌──────────────────┐              ┌──────────────────┐ ← Writable (container)
│ /app/main.py     │              │ /app/main.py*    │   *modified copy
│ /app/config.yml  │              ├──────────────────┤ ← Read-only (image layer 3)
│ /usr/bin/python   │             │ /app/config.yml  │
│ /etc/passwd       │             ├──────────────────┤ ← Read-only (image layer 2)
│ (unified view)    │             │ /usr/bin/python   │
└──────────────────┘              ├──────────────────┤ ← Read-only (image layer 1)
                                  │ /etc/passwd       │
                                  └──────────────────┘
```

Default: **OverlayFS** (overlay2). Previous: AUFS, devicemapper.

---

**11. List containers?**

```bash
docker ps                  # Running containers only
docker ps -a               # ALL containers (including stopped)
docker ps -q               # Only container IDs
docker ps -s               # Include size
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
docker ps -f "status=exited"   # Filter by status
```

---

**12. Show container logs?**

```bash
docker logs <container>               # All logs
docker logs -f <container>            # Follow (real-time, like tail -f)
docker logs --tail 100 <container>    # Last 100 lines
docker logs --since 1h <container>    # Last hour
docker logs --until 2h <container>    # Older than 2h
docker logs -t <container>            # Include timestamps
```

Logs come from container's **stdout** and **stderr** streams.

---

**13. Execute command inside running container?**

```bash
docker exec -it <container> /bin/sh    # Interactive shell (alpine/minimal)
docker exec -it <container> bash       # Bash shell (debian/ubuntu)
docker exec <container> ls /app        # One-off command
docker exec -u root <container> cmd    # Run as root
docker exec -w /app <container> cmd    # Set working directory
```

`-i` = interactive (keep stdin open), `-t` = allocate pseudo-TTY

---

**14. `docker inspect`?**

Returns detailed JSON info about any Docker object (container, image, network, volume).

```bash
docker inspect <container>
docker inspect --format '{{.NetworkSettings.IPAddress}}' <container>
docker inspect --format '{{.State.Health.Status}}' <container>
docker inspect --format '{{json .Config.Env}}' <container> | jq
docker inspect --format '{{range .Mounts}}{{.Source}} → {{.Destination}}{{end}}' <id>
```

Shows: IP address, mounts, env vars, network settings, health, image layers, config.

---

**15. Remove stopped containers and unused images?**

```bash
docker container prune             # Remove all stopped containers
docker image prune                 # Remove dangling images (untagged)
docker image prune -a              # Remove ALL unused images
docker volume prune                # Remove unused volumes
docker network prune               # Remove unused networks
docker system prune                # All of the above
docker system prune -a --volumes   # Nuclear: everything unused + volumes
docker system df                   # Check disk usage first
```

---

## Dockerfile Deep Dive

**16. Dockerfile? Build context?**

- **Dockerfile**: Text file with sequential instructions to build a Docker image
- **Build context**: Directory sent to Docker daemon — everything in it (minus `.dockerignore`) is available during build

```
Build context:                  Docker daemon:
┌────────────────┐   SEND ALL   ┌────────────────────┐
│ myproject/     │ ──────────►  │ Receives files     │
│ ├── Dockerfile │   via API    │ Executes each      │
│ ├── app.py     │              │ Dockerfile step    │
│ ├── .env       │ ← sent too! │ Creates layers     │
│ ├── node_modules│ ← huge!    │ Builds final image │
│ └── .git/      │ ← not needed│                    │
└────────────────┘              └────────────────────┘

Use .dockerignore to exclude!
```

```bash
docker build -t myapp:v1 .        # '.' is build context
docker build -t myapp:v1 -f Dockerfile.prod .
```

---

**17. Dockerfile instructions explained:**

```dockerfile
FROM python:3.11-slim          # Base image — REQUIRED, must be first
WORKDIR /app                   # Set working dir (mkdir + cd combined)
ENV APP_ENV=production         # Set env var — available at build + runtime
ARG BUILD_VERSION=1.0          # Build-time variable — NOT in final image
LABEL maintainer="vaibhav"    # Metadata key=value pair
COPY requirements.txt .        # Copy file from build context → image
ADD archive.tar.gz /app/       # Like COPY but auto-extracts tar & supports URLs
RUN pip install -r requirements.txt  # Execute command — creates a new layer
COPY . .                       # Copy rest of app (last for cache)
EXPOSE 8080                    # Documentation — does NOT publish port
USER appuser                   # Switch to non-root user
VOLUME ["/data"]               # Create mount point for external storage
HEALTHCHECK --interval=30s CMD curl -f http://localhost:8080/health
CMD ["python", "app.py"]       # Default command — overridable at docker run
ENTRYPOINT ["python"]          # Fixed executable — args appended at run
```

---

**18. COPY vs ADD?**

| Feature | COPY | ADD |
|---------|------|-----|
| Copy files | ✅ | ✅ |
| Auto-extract tar | ❌ | ✅ |
| Download URLs | ❌ | ✅ (avoid — no caching) |
| Predictable | ✅ | ❌ (magic behavior) |
| **Recommendation** | **Always use COPY** | Only for tar extraction |

---

**19. CMD vs ENTRYPOINT?**

```
┌─── CMD ──────────────────────────┬─── ENTRYPOINT ─────────────────────┐
│ Default command                   │ Fixed executable                   │
│ Fully overridden at docker run   │ docker run args APPENDED           │
│                                   │                                    │
│ CMD ["python", "app.py"]         │ ENTRYPOINT ["python"]              │
│ docker run myapp                  │ CMD ["app.py"]  ← default arg     │
│   → python app.py                │ docker run myapp                   │
│                                   │   → python app.py                 │
│ docker run myapp bash             │ docker run myapp test.py           │
│   → bash (CMD replaced!)        │   → python test.py (arg appended) │
└───────────────────────────────────┴────────────────────────────────────┘
```

**Best practice**: Use ENTRYPOINT for the executable, CMD for default arguments.

---

**20. Both CMD and ENTRYPOINT?**

ENTRYPOINT is the fixed executable. CMD provides default arguments that can be overridden:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run myapp           → python app.py
# docker run myapp test.py   → python test.py (CMD overridden)
# docker run --entrypoint sh myapp  → sh (ENTRYPOINT overridden)
```

---

**21. Shell form vs exec form?**

```
┌─── Shell Form ─────────────────┬─── Exec Form ─────────────────────┐
│ RUN apt-get update             │ RUN ["apt-get", "update"]          │
│ CMD python app.py              │ CMD ["python", "app.py"]           │
│                                 │                                    │
│ Executed as: /bin/sh -c "..."  │ Direct execution (no shell)        │
│ Shell features: $VAR, pipes,   │ No shell processing                │
│   &&, ||, wildcards            │ Proper signal handling (SIGTERM)   │
│                                 │ PID 1 = your app (not sh)         │
│ PID 1 = sh (bad for signals!) │                                    │
│ Variable substitution works    │ No variable substitution           │
└─────────────────────────────────┴────────────────────────────────────┘
```

**Rule**: Use exec form for CMD and ENTRYPOINT (proper signal handling). Use shell form for RUN (need shell features like `&&`).

---

**22. .dockerignore?**

Excludes files from build context → faster builds, smaller context, better security.

```
.git
.gitignore
node_modules
__pycache__
*.pyc
*.md
.env
.vscode
Dockerfile
docker-compose*.yml
*.log
tests/
docs/
```

Without `.dockerignore`, a project with 500MB `node_modules` sends 500MB to daemon every build!

---

**23. ARG vs ENV?**

```
┌─── Build Time ───────────────────┬─── Runtime ──────────────┐
│                                   │                          │
│  ARG: ✅ Available                │  ARG: ❌ NOT available   │
│  ENV: ✅ Available                │  ENV: ✅ Available       │
│                                   │                          │
│  docker build                     │  docker run              │
│    --build-arg VER=1.0            │    -e APP_ENV=prod       │
└───────────────────────────────────┴──────────────────────────┘
```

```dockerfile
ARG BUILD_VERSION=1.0              # Build-only — NOT in running container
ENV APP_VERSION=${BUILD_VERSION}   # Captures ARG value → persists to runtime
```

**Never put secrets in ARG** — they're visible in image history (`docker history`).

---

**24. HEALTHCHECK?**

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```
Container Status Transitions:
  starting ──(start-period)──► healthy ──(check fails 3x)──► unhealthy
                                  ▲                              │
                                  │    (check passes again)      │
                                  └──────────────────────────────┘
```

Docker runs the command periodically. Used by orchestrators to restart unhealthy containers.

---

**25. USER instruction? Why not root?**

```dockerfile
RUN addgroup --system app && adduser --system --ingroup app appuser
USER appuser
```

```
Running as root:                    Running as non-root:
┌──────────────────────┐            ┌──────────────────────┐
│ Container            │            │ Container            │
│ Process runs as ROOT │            │ Process runs as      │
│                      │            │ appuser (UID 1001)   │
│ If exploited:        │            │                      │
│ - Can modify any file│            │ If exploited:        │
│ - Can install tools  │            │ - Limited access     │
│ - Container escape   │            │ - Can't install      │
│   risk!              │            │ - Much harder to     │
│                      │            │   escalate ✅        │
└──────────────────────┘            └──────────────────────┘
```

---

## Interview-Style (Basics)

**26. Walk through a Dockerfile?**

```dockerfile
FROM python:3.11-slim                    # Start with minimal Python
WORKDIR /app                             # Set working directory
COPY requirements.txt .                  # Copy deps FIRST (layer caching!)
RUN pip install --no-cache-dir \
    -r requirements.txt                  # Install deps (cached if req unchanged)
COPY . .                                 # Copy app code LAST (changes most)
RUN adduser --system appuser             # Create non-root user
USER appuser                             # Don't run as root
EXPOSE 8080                              # Document the port
HEALTHCHECK --interval=30s \
    CMD curl -f http://localhost:8080/health || exit 1
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]
```

**Key optimization**: COPY requirements.txt before COPY . . Because if only app code changes, pip install layer stays cached.

---

**27. Image is 2GB — reduce size?**

```
Strategy                         Impact
────────────────────────────────────────────────
1. Smaller base image            python:3.11 (900MB) → slim (150MB) → alpine (50MB)
2. Multi-stage build             Build in fat image, copy artifact to slim
3. Combine RUN commands          Fewer layers + clean in same layer
4. .dockerignore                 Exclude .git, node_modules, tests
5. --no-cache-dir (pip)          Don't cache downloaded packages
6. Clean apt cache               rm -rf /var/lib/apt/lists/* in same RUN
7. Distroless final image        Only app runtime, nothing else

Before: FROM python:3.11    → 900MB
After:  Multi-stage + slim  → 80MB   (90% reduction!)
```

---

**28. `docker run` vs `docker exec`?**

```
docker run:                          docker exec:
┌──────────────────────┐            ┌──────────────────────┐
│ Creates NEW container│            │ Runs in EXISTING     │
│ from an image        │            │ running container    │
│                      │            │                      │
│ docker run nginx     │            │ docker exec -it      │
│ → new container      │            │   <id> bash          │
│                      │            │ → shell in existing  │
│ Starts a new process │            │ Runs additional      │
│ tree (PID 1)         │            │ process alongside    │
└──────────────────────┘            └──────────────────────┘
```

---

**29. Debug container that exits immediately?**

```
Debugging Flowchart:
┌─────────────────────────────┐
│ docker ps -a                │ ← Check exit code in STATUS
└─────────┬───────────────────┘
          │
┌─────────▼───────────────────┐
│ docker logs <container>     │ ← Read error messages
└─────────┬───────────────────┘
          │
┌─────────▼───────────────────┐
│ docker run -it <image> sh   │ ← Override CMD with shell
│ (manually run the command)  │   to explore filesystem
└─────────┬───────────────────┘
          │
┌─────────▼───────────────────┐
│ docker inspect <image>      │ ← Check ENTRYPOINT/CMD config
│ grep Cmd/Entrypoint         │
└─────────────────────────────┘

Common causes:
  - Missing environment variables
  - Wrong CMD / file not found
  - Permission errors (running as non-root)
  - Missing config file / dependency
  - App crashes on startup (check logs!)
```

---

**30. Dockerfile for Python Flask app?**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Dependencies first (cache optimization)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Application code
COPY . .

# Security: non-root user
RUN adduser --system --no-create-home appuser
USER appuser

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=5s \
    CMD curl -f http://localhost:5000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

---

# PART 2: ADVANCED (50 Qs)

---

## Multi-Stage Builds

**1. Multi-stage build? Why?**

Multiple `FROM` statements in one Dockerfile. Build with all tools in one stage, copy only the final artifact to a clean minimal stage.

```
┌─── Stage 1: Builder ─────────────┐    ┌─── Stage 2: Runtime ────────────┐
│  FROM golang:1.21 AS builder     │    │  FROM gcr.io/distroless/static  │
│                                   │    │                                  │
│  Contains:                       │    │  Contains:                       │
│  - Go compiler (500MB)           │    │  - Your binary (10MB)            │
│  - Build tools                   │    │  - Nothing else!                 │
│  - Source code                   │    │                                  │
│  - All dependencies              │    │  COPY --from=builder /app/bin .  │
│                                   │    │                                  │
│  Total: ~800MB                   │    │  Total: ~15MB                    │
└───────────────────────────────────┘    └──────────────────────────────────┘
```

**Why?** 95%+ size reduction. No build tools in production = smaller attack surface.

---

**2. Multi-stage for Go app?**

```dockerfile
# ─── Build Stage ───
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download                    # Cache deps
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# ─── Runtime Stage ───
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/myapp /myapp
USER nonroot
ENTRYPOINT ["/myapp"]
# Result: ~10MB instead of ~800MB!
```

---

**3. Multi-stage for Node.js?**

```dockerfile
# ─── Build Stage ───
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# ─── Runtime Stage ───
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json .
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

**4. Copy files between stages?**

```dockerfile
COPY --from=builder /app/output /app/       # By stage name
COPY --from=0 /app/binary /usr/local/bin/   # By stage index (0-based)
COPY --from=nginx:latest /etc/nginx/nginx.conf /etc/nginx/  # From external image!
```

---

**5. Name build stages?**

```dockerfile
FROM golang:1.21 AS builder       # Named "builder"
FROM node:20 AS frontend           # Named "frontend"

FROM alpine:3.18
COPY --from=builder /app/api /api
COPY --from=frontend /app/dist /static/

# Build only specific stage:
# docker build --target builder -t myapp-builder .
```

---

**6. How many stages? Limit?**

No practical limit. Common patterns: 2-3 stages (build, test, runtime). Each stage starts fresh — only `COPY --from` connects them.

---

## Networking

**7. Docker networking modes?**

```
┌─── bridge (default) ────────────────────────────────────────────┐
│  Container A ──┐                                                │
│  Container B ──┤── docker0 bridge ──── NAT ──── Host Network   │
│  Container C ──┘                                                │
│  Isolated from host, containers communicate via bridge          │
└──────────────────────────────────────────────────────────────────┘

┌─── host ────────────────────────────────────────────────────────┐
│  Container uses host's network stack directly                   │
│  No port mapping needed — container IS the host network         │
│  Best performance, no isolation                                 │
│  docker run --network host nginx → port 80 on host directly    │
└──────────────────────────────────────────────────────────────────┘

┌─── none ────────────────────────────────────────────────────────┐
│  No networking at all. Only loopback. Complete isolation.       │
│  Use case: batch jobs, security-sensitive processing            │
└──────────────────────────────────────────────────────────────────┘

┌─── overlay ─────────────────────────────────────────────────────┐
│  Multi-host networking (Docker Swarm / K8s)                     │
│  VXLAN tunneling between hosts                                  │
│  Containers on different hosts communicate as if same network   │
└──────────────────────────────────────────────────────────────────┘

┌─── macvlan ─────────────────────────────────────────────────────┐
│  Container gets its own MAC address on physical network         │
│  Appears as a physical device to the network                    │
│  Use case: legacy apps that need to be on the LAN              │
└──────────────────────────────────────────────────────────────────┘
```

---

**8. Default network?**

`bridge` — the `docker0` bridge. All containers connect to it unless `--network` is specified.

---

**9. Containers on same bridge communicate?**

```
Default bridge (docker0):          Custom bridge (user-defined):
┌──────────────────────┐           ┌──────────────────────┐
│ Container A          │           │ Container A          │
│  172.17.0.2          │           │  "web"               │
│ Container B          │           │ Container B          │
│  172.17.0.3          │           │  "api"               │
│                      │           │                      │
│ Communicate by IP    │           │ Communicate by NAME  │
│ only! No DNS! ❌     │           │ DNS resolution! ✅   │
│                      │           │ curl http://api:8080 │
└──────────────────────┘           └──────────────────────┘
```

```bash
docker network create mynet
docker run --network mynet --name web nginx
docker run --network mynet --name api myapp
# api can reach web at http://web:80 — automatic DNS!
```

**Always use custom bridge networks** — never the default bridge.

---

**10. Containers on different networks?**

They **cannot** communicate by default (network isolation).

```bash
# Solution 1: Connect container to both networks
docker network connect net2 container1

# Solution 2: Use one shared network
docker run --network shared container1
docker run --network shared container2
```

---

**11. `docker network create`?**

```bash
docker network create mynet                         # Default bridge driver
docker network create --driver overlay mynet        # Overlay for Swarm
docker network create --subnet 10.0.0.0/24 mynet   # Custom subnet
docker network ls                                    # List all networks
docker network inspect mynet                        # Details + connected containers
docker network rm mynet                              # Delete
docker network prune                                 # Remove unused
```

---

**12. EXPOSE vs `-p`?**

```
EXPOSE 8080 (in Dockerfile):          -p 8080:80 (at docker run):
┌─────────────────────────────┐      ┌─────────────────────────────┐
│ Documentation ONLY          │      │ Actually PUBLISHES port     │
│ Port is NOT accessible      │      │                             │
│ from host                   │      │ Host:8080 ──► Container:80  │
│                              │      │                             │
│ Just metadata for humans    │      │ Creates iptables rule       │
│ and tools                   │      │ Traffic flows!              │
└─────────────────────────────┘      └─────────────────────────────┘
```

---

**13. Port mapping?**

```bash
docker run -p 8080:80 nginx              # host:8080 → container:80
docker run -p 127.0.0.1:8080:80 nginx    # Only localhost (not external)
docker run -p 8080:80/udp nginx          # UDP port
docker run -P nginx                       # Map ALL EXPOSEd ports to random host ports
docker run -p 3000-3005:3000-3005 nginx  # Port range
```

---

**14. Container linking?**

```bash
# ❌ LEGACY (deprecated) — don't use
docker run --link db:database myapp

# ✅ MODERN — use custom networks
docker network create mynet
docker run --network mynet --name db postgres
docker run --network mynet --name app myapp
# app can resolve "db" by name
```

---

## Storage

**15. Docker volume? Why not container filesystem?**

```
Container filesystem:                Volume:
┌──────────────────────┐            ┌──────────────────────┐
│ Data written here    │            │ Data written here    │
│ is EPHEMERAL         │            │ PERSISTS beyond      │
│                      │            │ container lifecycle  │
│ docker rm = data     │            │                      │
│ gone forever!        │            │ docker rm = data     │
│                      │            │ still there! ✅      │
│ Uses CoW (slow)      │            │ Direct I/O (fast)    │
│ Tied to container    │            │ Shareable between    │
│ Not shareable        │            │ containers           │
└──────────────────────┘            └──────────────────────┘
```

---

**16. Volume vs bind mount vs tmpfs?**

```
┌─── Volume (Docker-managed) ──────────────────────────────────┐
│  Location: /var/lib/docker/volumes/<name>/_data              │
│  Managed by Docker — best for production data                │
│  docker run -v mydata:/app/data myapp                        │
└──────────────────────────────────────────────────────────────┘

┌─── Bind Mount (Host path) ──────────────────────────────────┐
│  Location: Any path on host (you choose)                     │
│  Best for development — mount source code                    │
│  docker run -v /home/user/code:/app myapp                    │
│  Security risk: container accesses host filesystem           │
└──────────────────────────────────────────────────────────────┘

┌─── tmpfs (RAM only) ────────────────────────────────────────┐
│  Location: Host memory (RAM)                                 │
│  NOT persisted — gone when container stops                   │
│  Best for sensitive temp data (secrets processing)           │
│  docker run --tmpfs /run/secrets myapp                       │
└──────────────────────────────────────────────────────────────┘
```

| Type | Location | Persists? | Shareable? | Best For |
|------|----------|-----------|------------|----------|
| Volume | Docker-managed | ✅ Yes | ✅ Yes | DB data, prod |
| Bind mount | Host path | ✅ Yes | ✅ Yes | Dev, source code |
| tmpfs | RAM | ❌ No | ❌ No | Temp secrets |

---

**17. Create and manage volumes?**

```bash
docker volume create mydata                    # Create
docker volume ls                                # List
docker volume inspect mydata                   # Details
docker volume rm mydata                         # Remove
docker volume prune                             # Remove all unused

docker run -v mydata:/app/data myapp           # Named volume
docker run -v /host/path:/container/path myapp # Bind mount
docker run -v mydata:/app/data:ro myapp        # Read-only volume
```

---

**18. Share data between containers?**

```bash
docker volume create shared
docker run -v shared:/data --name writer myapp
docker run -v shared:/data --name reader myapp2
# Both containers read/write to the same volume!
```

---

**19. Named vs anonymous volume?**

```bash
docker run -v mydata:/data myapp       # Named: "mydata" — easy to find, manage
docker run -v /data myapp              # Anonymous: random hash (8f3a2b...) — hard to track

# Named volumes survive docker-compose down
# Anonymous volumes are harder to back up / reference
```

---

**20. Backup a Docker volume?**

```bash
# Backup
docker run --rm \
  -v mydata:/source:ro \
  -v $(pwd):/backup \
  busybox tar czf /backup/backup.tar.gz -C /source .

# Restore
docker run --rm \
  -v mydata:/target \
  -v $(pwd):/backup \
  busybox tar xzf /backup/backup.tar.gz -C /target
```

---

## Docker Compose

**21. Docker Compose? When use?**

Tool for defining and running **multi-container applications** using a YAML file.

```
Without Compose:                    With Compose:
$ docker network create mynet      $ docker compose up -d
$ docker run --network mynet       (one command!)
    --name db -v data:/var/lib
    -e POSTGRES_PASSWORD=pass
    postgres:15
$ docker run --network mynet
    --name web -p 8080:8080
    -e DB_HOST=db myapp            docker-compose.yml defines
$ ...repeat for every service...   everything declaratively
```

**Use when**: multiple services (web + DB + cache), local dev environments, integration testing, demo setups.

---

**22. Compose: web app + PostgreSQL + Redis?**

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/mydb
      REDIS_URL: redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
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

---

**23. `docker-compose up` vs `docker-compose up -d`?**

| Command | Mode | Logs | Stop |
|---------|------|------|------|
| `docker-compose up` | Foreground | Visible in terminal | Ctrl+C |
| `docker-compose up -d` | Detached (background) | Use `docker-compose logs` | `docker-compose down` |

---

**24. Scale a service?**

```bash
docker-compose up -d --scale web=3    # 3 instances of web

# Or in compose file:
services:
  web:
    deploy:
      replicas: 3
```

Note: Can't use fixed host port with scaling (port conflict). Use a load balancer or omit host port.

---

**25. `depends_on`? Does it wait for ready?**

```yaml
# ❌ Basic depends_on — only waits for container START (not ready!)
depends_on:
  - db

# ✅ With health check — waits for service HEALTHY
depends_on:
  db:
    condition: service_healthy
```

**No**, basic `depends_on` does NOT wait for the service to be ready. It only ensures container start order. Use `condition: service_healthy` for actual readiness.

---

**26. Environment variables in Compose?**

```yaml
# Method 1: Inline in compose file
environment:
  DB_HOST: postgres
  DB_PORT: "5432"

# Method 2: .env file (auto-loaded from same directory)
# .env
DB_HOST=postgres

# Reference in compose:
environment:
  DB_HOST: ${DB_HOST}

# Method 3: External env file
env_file:
  - ./config/.env.prod
```

---

**27. Compose v1 vs v2?**

| Aspect | v1 | v2 |
|--------|-----|-----|
| Command | `docker-compose` | `docker compose` (no hyphen) |
| Implementation | Separate Python binary | Built into Docker CLI (Go) |
| Speed | Slower | Faster |
| Status | **Deprecated** | **Current — use this** |
| Compose file | Same YAML format | Same YAML format |

---

## Security & Optimization

**28. 10 Docker security best practices?**

```
┌─── Docker Security Checklist ────────────────────────────────┐
│                                                               │
│  Image:                                                       │
│  ✅ 1. Use official/verified base images                     │
│  ✅ 2. Pin image versions (python:3.11.7-slim not :latest)   │
│  ✅ 3. Use multi-stage builds (minimal final image)          │
│  ✅ 4. Scan images (Trivy, Snyk, Docker Scout)               │
│  ✅ 5. Use distroless or slim base images                    │
│                                                               │
│  Runtime:                                                     │
│  ✅ 6. Run as non-root (USER instruction)                    │
│  ✅ 7. Read-only filesystem (--read-only)                    │
│  ✅ 8. Drop capabilities (--cap-drop ALL)                    │
│  ✅ 9. No secrets in images (use runtime mount/env)          │
│  ✅ 10. Use .dockerignore (exclude .env, .git, keys)         │
│                                                               │
│  Advanced:                                                    │
│  - No --privileged unless absolutely needed                  │
│  - Use Docker Content Trust (signed images)                  │
│  - Limit resources (--memory, --cpus)                        │
│  - Network segmentation (custom networks)                    │
│  - Audit docker.sock access                                  │
└───────────────────────────────────────────────────────────────┘
```

---

**29. Distroless image?**

Google's container images containing ONLY the application runtime — no shell, no package manager, no debugging tools.

```
Standard image:                     Distroless:
┌──────────────────────┐           ┌──────────────────────┐
│ Your app             │           │ Your app             │
│ bash, sh             │           │ (no shell!)          │
│ apt, curl, wget      │           │ (no package mgr!)   │
│ find, grep, sed      │           │ (no tools!)          │
│ man pages            │           │ Runtime only         │
│ ~150MB               │           │ ~5-20MB              │
│ Attack surface: HIGH │           │ Attack surface: LOW  │
└──────────────────────┘           └──────────────────────┘
```

```dockerfile
FROM gcr.io/distroless/python3
COPY app.py /app.py
CMD ["app.py"]
```

---

**30. Alpine vs slim vs full?**

| Image | Size | C Library | Tools | Production Use |
|-------|------|-----------|-------|----------------|
| Full (`python:3.11`) | ~900MB | glibc | Everything | Dev/debugging only |
| Slim (`python:3.11-slim`) | ~150MB | glibc | Minimal | **Default choice** ✅ |
| Alpine (`python:3.11-alpine`) | ~50MB | musl | busybox | Size-critical (compat issues) |
| Distroless | ~5-20MB | glibc | None | Highest security |

**Warning**: Alpine uses `musl` libc instead of `glibc` — can cause compatibility issues with some Python packages (numpy, pandas).

---

**31. Scan images for vulnerabilities?**

```bash
# Trivy (free, comprehensive) ← recommended
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# Snyk
snyk container test myapp:latest

# Docker Scout (built into Docker Desktop)
docker scout cves myapp:latest

# In CI/CD pipeline:
trivy image --exit-code 1 --severity CRITICAL myapp:latest
# Exit code 1 = fail the build if critical vulns found
```

---

**32. Layer caching? Optimize?**

```
Rule: Docker invalidates cache from first changed layer DOWN

  ┌──────────────────────────┐
  │ FROM python:3.11-slim    │ ← Rarely changes     → CACHED ✅
  ├──────────────────────────┤
  │ COPY requirements.txt . │ ← Sometimes changes   → CACHED ✅
  ├──────────────────────────┤
  │ RUN pip install          │ ← Cached if req same  → CACHED ✅
  ├──────────────────────────┤
  │ COPY . .                 │ ← Changes every build → REBUILT
  └──────────────────────────┘

  Bad order:
  COPY . .                    ← Every code change invalidates ALL below
  RUN pip install             ← Reinstalls EVERY build!
```

**Order instructions: least-frequently changing → most-frequently changing.**

---

**33. Why order least→most frequently changing?**

Because Docker invalidates cache from the **first changed layer onward**. Everything below a change is cached; everything above must rebuild. If you COPY source code before installing dependencies, dependencies reinstall every time code changes = slow builds.

---

**34. BuildKit?**

Next-gen Docker build engine (default since Docker 23.0).

```
BuildKit advantages over legacy builder:
  ✅ Parallel execution of independent stages
  ✅ Better cache management
  ✅ Build secrets (--mount=type=secret) — not in layers!
  ✅ SSH forwarding (--mount=type=ssh)
  ✅ Build output customization
  ✅ Faster builds
```

```bash
DOCKER_BUILDKIT=1 docker build .

# Build secret (NOT stored in any layer):
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm install
docker build --secret id=npmrc,src=.npmrc .
```

---

**35. `docker system prune`?**

```bash
docker system df               # Check disk usage FIRST

docker system prune            # Remove: stopped containers, unused networks,
                               #         dangling images, build cache

docker system prune -a         # Same + ALL unused images (not just dangling)

docker system prune -a --volumes  # Same + unused volumes (data loss risk!)
```

---

**36. Memory and CPU limits?**

```bash
# CLI
docker run --memory=512m --cpus=1.5 myapp
docker run -m 256m --memory-swap=512m myapp    # Memory + swap limit
docker run --cpus=2 --cpu-shares=512 myapp     # CPU limit + relative weight

# Docker Compose
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

# Monitor usage
docker stats                   # Live CPU/memory/network per container
```

---

**37. Docker Content Trust?**

Ensures image integrity and publisher authentication via digital signatures.

```bash
export DOCKER_CONTENT_TRUST=1
docker pull myimage:latest     # Only pulls signed images
docker push myimage:latest     # Signs before pushing

# Unsigned image → pull fails with DCT enabled
```

---

**38. Prevent privilege escalation?**

```bash
# Runtime flags
docker run \
  --security-opt=no-new-privileges \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  --read-only \
  --user 1000:1000 \
  myapp
```

```dockerfile
# In Dockerfile
RUN adduser --system --no-create-home appuser
USER appuser
```

```yaml
# In Kubernetes
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
```

---

## Troubleshooting

**39. Container running but app not reachable?**

```
Troubleshooting Flowchart:
┌──────────────────────────────────────┐
│ 1. docker ps → check PORTS column   │ ← Port published?
│    0.0.0.0:8080->80/tcp ✅          │
│    (nothing) ❌ → add -p flag       │
└─────────┬────────────────────────────┘
          │
┌─────────▼────────────────────────────┐
│ 2. docker logs <container>           │ ← App error? Binding error?
└─────────┬────────────────────────────┘
          │
┌─────────▼────────────────────────────┐
│ 3. App binding to 0.0.0.0?          │ ← NOT 127.0.0.1!
│    127.0.0.1 = only inside container│
│    0.0.0.0   = accessible from host │
└─────────┬────────────────────────────┘
          │
┌─────────▼────────────────────────────┐
│ 4. docker exec -it <c> curl         │ ← Test from inside container
│    localhost:8080                     │
└─────────┬────────────────────────────┘
          │
┌─────────▼────────────────────────────┐
│ 5. Check firewall / security group  │
│    Check docker network inspect      │
└──────────────────────────────────────┘
```

---

**40. Build fails at RUN step?**

```
1. Read error message carefully — usually tells you exactly what's wrong
2. Run intermediate image:
   docker run -it <last-successful-layer-sha> /bin/sh
   → manually run the failing command to debug
3. Check network — can container download packages?
4. Check file paths — does the file exist in the build context?
5. Check permissions — is USER set before the RUN?
6. Use --no-cache to rebuild from scratch:
   docker build --no-cache -t myapp .
```

---

**41. Images eating all disk space?**

```bash
# 1. Check what's using space
docker system df
# TYPE          TOTAL   ACTIVE  SIZE    RECLAIMABLE
# Images        45      3       12.3GB  11.2GB (91%)
# Containers    5       2       234MB   134MB
# Build Cache   0       0       3.4GB   3.4GB

# 2. Clean up
docker image prune -a              # Remove unused images
docker builder prune               # Clear build cache
docker volume prune                # Remove unused volumes
docker system prune -a --volumes   # Nuclear option
```

---

**42. `docker logs` empty but container running?**

| Cause | Solution |
|-------|----------|
| App writes to file, not stdout | `docker exec cat /app/logs/app.log` |
| Logging framework redirects | Configure to log to stdout |
| Output buffered | Python: `python -u` or `PYTHONUNBUFFERED=1` |
| Wrong log driver | Check: `docker inspect --format '{{.HostConfig.LogConfig}}'` |

---

**43. Container using 100% CPU?**

```bash
docker stats                           # Live resource usage per container
docker top <container>                 # Processes inside container
docker exec -it <container> top        # top inside container

# Fix:
docker update --cpus=1.0 <container>   # Limit CPU dynamically
# Or set limits in docker run / compose
```

---

## Interview-Style (Advanced)

**44. Docker images in CI/CD?**

```
CI/CD Pipeline with Docker:

  Code Push
      │
  ┌───▼─────────────┐
  │ Build Image      │  docker build -t myapp:${GIT_SHA} .
  └───┬──────────────┘
      │
  ┌───▼─────────────┐
  │ Run Tests        │  docker run myapp:${GIT_SHA} pytest
  └───┬──────────────┘
      │
  ┌───▼─────────────┐
  │ Scan Image       │  trivy image myapp:${GIT_SHA}
  └───┬──────────────┘
      │
  ┌───▼─────────────┐
  │ Push to Registry │  docker push acr.io/myapp:${GIT_SHA}
  └───┬──────────────┘
      │
  ┌───▼─────────────┐
  │ Deploy           │  kubectl set image deploy/myapp=acr.io/myapp:${GIT_SHA}
  └──────────────────┘
```

---

**45. Image tagging strategy?**

```bash
myapp:1.2.3                    # Semantic version ← best for releases
myapp:20240115-abc1234          # Date + git SHA ← unique, traceable
myapp:${BUILD_BUILDID}          # CI build number
myapp:main-abc1234              # Branch + SHA
myapp:latest                    # Mutable! ← NEVER use in production

# Best practice: immutable tags
# Tag with git SHA → always know exactly what code is deployed
# Also tag with semver for human readability
docker tag myapp:abc1234 myapp:1.2.3
docker push myapp:abc1234
docker push myapp:1.2.3
```

---

**46. Experience containerizing an existing app?**

```
Migration Steps:
  1. Analyze  → app dependencies, runtime, config, ports, storage
  2. Create   → Dockerfile (multi-stage, non-root, healthcheck)
  3. .dockerignore → exclude .git, node_modules, secrets
  4. Test     → build locally, run, verify all features
  5. Compose  → add DB, cache, other services
  6. CI/CD    → build + scan + push in pipeline
  7. Deploy   → K8s or ECS with proper resource limits
  8. Monitor  → logs to stdout, metrics endpoint, healthchecks

Challenges: stateful data, environment-specific config,
           legacy dependencies, team adoption
```

---

**47. Docker vs Podman?**

```
┌─── Docker ──────────────────────┬─── Podman ──────────────────────┐
│ Daemon-based (dockerd)          │ Daemonless (no central process) │
│ Root by default                 │ Rootless by default             │
│ docker CLI                      │ podman CLI (drop-in compatible) │
│ Docker Compose                  │ podman-compose / pods           │
│ Docker Swarm                    │ No built-in orchestration       │
│ Industry standard               │ RHEL/Fedora default             │
│ Larger attack surface (daemon)  │ Smaller attack surface          │
│                                  │ Pod concept (like K8s pods)     │
└──────────────────────────────────┴──────────────────────────────────┘
```

`alias docker=podman` works for most commands — Podman is CLI-compatible.

---

**48. OCI? OCI-compliant images?**

**Open Container Initiative** — industry standards for containers:

```
OCI Specifications:
  1. Image Spec     → how container images are built and stored
  2. Runtime Spec   → how containers are created and run
  3. Distribution Spec → how images are distributed (registries)

All modern tools produce OCI-compliant images:
  Docker → OCI image
  Podman → OCI image
  Buildah → OCI image
  containerd → runs OCI images
  CRI-O → runs OCI images

OCI = portability: build with Docker, run with Podman/containerd/CRI-O
```

---

**49. Secrets in Docker?**

```
┌─── ❌ BAD: Secrets in Image ────────────────────────────────┐
│  ENV API_KEY=mysecretkey      ← visible in docker inspect!  │
│  ARG PASSWORD=secret          ← visible in docker history!  │
│  COPY .env /app/.env          ← baked into image layer!     │
└──────────────────────────────────────────────────────────────┘

┌─── ✅ GOOD: Runtime Secrets ────────────────────────────────┐
│  # BuildKit secret (not stored in any layer)                │
│  RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \     │
│      npm install                                             │
│  docker build --secret id=npmrc,src=.npmrc .                │
│                                                              │
│  # Runtime: mount as file                                    │
│  docker run -v /secrets/api_key:/run/secrets/api_key:ro app │
│                                                              │
│  # Kubernetes: Secret + volume mount                         │
│  # Vault / Azure Key Vault / AWS Secrets Manager            │
└──────────────────────────────────────────────────────────────┘
```

---

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
    depends_on:
      user-service:
        condition: service_healthy
      order-service:
        condition: service_healthy
    networks: [backend]

  user-service:
    build: ./user-service
    environment:
      DB_HOST: user-db
    depends_on:
      user-db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
    networks: [backend]

  order-service:
    build: ./order-service
    environment:
      DB_HOST: order-db
    depends_on:
      order-db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 10s
    networks: [backend]

  user-db:
    image: postgres:15
    environment:
      POSTGRES_DB: users
      POSTGRES_PASSWORD: pass
    volumes: [user-data:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s
    networks: [backend]

  order-db:
    image: postgres:15
    environment:
      POSTGRES_DB: orders
      POSTGRES_PASSWORD: pass
    volumes: [order-data:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s
    networks: [backend]

volumes:
  user-data:
  order-data:

networks:
  backend:
```

```
Architecture:
                    ┌──────────────┐
  Client ──────────►│ API Gateway  │ :8080
                    └──────┬───────┘
                     ┌─────┴─────┐
              ┌──────▼──┐   ┌────▼────────┐
              │ User    │   │ Order       │
              │ Service │   │ Service     │
              │ :3000   │   │ :4000       │
              └────┬────┘   └──────┬──────┘
              ┌────▼────┐   ┌──────▼──────┐
              │ User DB │   │ Order DB    │
              │(postgres)│  │ (postgres)  │
              └─────────┘   └─────────────┘
```
