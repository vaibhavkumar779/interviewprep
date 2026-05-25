# Docker - LEARNING MATERIAL

---

## Docker Architecture

```mermaid
graph TD
    CLI[Docker CLI] -->|REST API| Daemon[Docker Daemon / dockerd]
    Daemon -->|Manages| Containers[Containers]
    Daemon -->|Manages| Images[Images]
    Daemon -->|Manages| Volumes[Volumes]
    Daemon -->|Manages| Networks[Networks]
    Daemon -->|Pulls from| Registry[Docker Registry<br/>Docker Hub / ACR / ECR]

    subgraph Container
        App[Application Process]
        Libs[Libraries]
        FS[Filesystem Layer]
    end

    subgraph Host
        Kernel[Linux Kernel<br/>Shared by all containers]
    end
    Container --> Kernel
```

## Container vs VM

```mermaid
graph TD
    subgraph VMs [Virtual Machines]
        VM1[App1 + Bins + Guest OS]
        VM2[App2 + Bins + Guest OS]
        HV[Hypervisor]
        HW1[Host Hardware]
        VM1 --> HV
        VM2 --> HV
        HV --> HW1
    end
    subgraph Containers
        C1[App1 + Bins]
        C2[App2 + Bins]
        DE[Container Runtime]
        OS[Host OS + Kernel]
        HW2[Host Hardware]
        C1 --> DE
        C2 --> DE
        DE --> OS --> HW2
    end
```

| Aspect | Container | VM |
|---|---|---|
| Size | MBs | GBs |
| Start time | Seconds | Minutes |
| Isolation | Process-level | Full OS |
| Overhead | Low | High |
| Portability | High | Medium |

---

## Dockerfile Instructions

```dockerfile
# COMPLETE REFERENCE DOCKERFILE
FROM python:3.11-slim AS builder    # Base image + stage name
LABEL maintainer="vaibhav@example.com"  # Metadata

ARG APP_VERSION=1.0                 # Build-time variable
ENV APP_ENV=production              # Runtime env variable
ENV PORT=8080

WORKDIR /app                        # Set working directory

# Dependencies first (layer caching!)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Source code (changes frequently → after deps)
COPY src/ ./src/

# Security: non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

EXPOSE ${PORT}                      # Document port

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:${PORT}/health || exit 1

ENTRYPOINT ["python"]              # Fixed executable
CMD ["src/main.py"]                # Default arguments
```

## Docker Layer Caching

```mermaid
graph TD
    L1[Layer 1: FROM python:3.11-slim] --> L2[Layer 2: COPY requirements.txt]
    L2 --> L3[Layer 3: RUN pip install]
    L3 --> L4[Layer 4: COPY src/]
    L4 --> L5[Layer 5: RUN groupadd...]

    L1 -.->|Cached ✓| L1
    L2 -.->|Cached if requirements.txt unchanged ✓| L2
    L3 -.->|Cached if L2 cached ✓| L3
    L4 -.->|INVALIDATED if src/ changed ✗| L4
    L5 -.->|Rebuilt because L4 changed ✗| L5
```

**Rule**: Order instructions from **least** to **most** frequently changing.

---

## ENTRYPOINT vs CMD

```mermaid
graph TD
    E[ENTRYPOINT] -->|Sets| Fixed[Fixed executable]
    C[CMD] -->|Sets| Default[Default arguments]

    subgraph Combined
        EC["ENTRYPOINT ['python']<br/>CMD ['app.py']"]
        R1["docker run image → python app.py"]
        R2["docker run image test.py → python test.py"]
        EC --> R1
        EC --> R2
    end
```

| Scenario | ENTRYPOINT | CMD | docker run result |
|---|---|---|---|
| CMD only | — | `["python", "app.py"]` | `python app.py` |
| ENTRYPOINT only | `["python"]` | — | `python` |
| Both | `["python"]` | `["app.py"]` | `python app.py` |
| Override CMD | `["python"]` | `["app.py"]` | `docker run img test.py` → `python test.py` |

---

## Docker Networking

```mermaid
graph TD
    subgraph Bridge [bridge network - default]
        C1[Container 1<br/>172.17.0.2]
        C2[Container 2<br/>172.17.0.3]
        BR[Docker Bridge<br/>docker0 172.17.0.1]
        C1 --> BR
        C2 --> BR
        BR --> HOST[Host Network Stack]
    end
    HOST --> Internet
```

| Network Mode | Description | Use Case |
|---|---|---|
| `bridge` | Default. Containers get private IPs. | Most applications |
| `host` | Container shares host's network stack | Performance-critical |
| `none` | No networking | Security isolation |
| `overlay` | Multi-host networking | Docker Swarm / K8s |
| `macvlan` | Container gets own MAC/IP on physical network | Legacy apps |

---

## Docker Security Best Practices

1. Run as non-root user (`USER 1001`)
2. Use specific image tags, not `latest`
3. Use slim/distroless/alpine base images
4. Scan images (Trivy, Snyk)
5. Don't store secrets in images
6. Use `.dockerignore`
7. Set read-only filesystem where possible
8. Drop capabilities (`--cap-drop ALL`)
9. Use multi-stage builds (no build tools in prod)
10. Set resource limits (`--memory`, `--cpus`)
