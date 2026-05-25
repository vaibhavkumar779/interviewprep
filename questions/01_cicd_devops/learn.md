# CI/CD & DevOps Fundamentals - LEARNING MATERIAL

---

## DevOps Lifecycle

```mermaid
graph LR
    A[Plan] --> B[Code]
    B --> C[Build]
    C --> D[Test]
    D --> E[Release]
    E --> F[Deploy]
    F --> G[Operate]
    G --> H[Monitor]
    H --> A
    style A fill:#4CAF50,color:#fff
    style B fill:#2196F3,color:#fff
    style C fill:#FF9800,color:#fff
    style D fill:#9C27B0,color:#fff
    style E fill:#F44336,color:#fff
    style F fill:#00BCD4,color:#fff
    style G fill:#795548,color:#fff
    style H fill:#607D8B,color:#fff
```

## CI vs CD vs CD

```mermaid
graph LR
    subgraph CI [Continuous Integration]
        A[Developer Commits] --> B[Automated Build]
        B --> C[Automated Tests]
        C --> D[Build Artifact]
    end
    subgraph CDel [Continuous Delivery]
        D --> E[Deploy to Staging]
        E --> F[Manual Approval Gate]
    end
    subgraph CDep [Continuous Deployment]
        F --> G[Auto Deploy to Prod]
    end
```

### Key Concepts

| Term | Definition |
|---|---|
| **CI** | Merge code frequently → auto build + test → catch bugs early |
| **CD (Delivery)** | Artifact always deployable, manual approval to prod |
| **CD (Deployment)** | Every passing change auto-deploys to prod |
| **Pipeline** | Automated workflow: build → test → deploy |
| **Artifact** | Output of build (Docker image, JAR, binary, package) |
| **Agent/Runner** | Machine that executes pipeline jobs |
| **Trigger** | Event that starts a pipeline (push, PR, schedule, manual) |

---

## Pipeline Architecture

```mermaid
graph TD
    subgraph Triggers
        T1[Git Push]
        T2[Pull Request]
        T3[Schedule/Cron]
        T4[Manual]
    end
    subgraph Pipeline
        S1[Stage: Build]
        S2[Stage: Test]
        S3[Stage: Security Scan]
        S4[Stage: Build Image]
        S5[Stage: Deploy Staging]
        S6[Stage: Approval Gate]
        S7[Stage: Deploy Prod]
    end
    T1 --> S1
    T2 --> S1
    T3 --> S1
    T4 --> S1
    S1 --> S2 --> S3 --> S4 --> S5 --> S6 --> S7
```

---

## Deployment Strategies

```mermaid
graph TD
    subgraph BlueGreen [Blue-Green]
        BG1[Blue v1 - LIVE] --> BG2[Green v2 - Staged]
        BG2 --> BG3[Switch Traffic]
        BG3 --> BG4[Green v2 - LIVE]
    end
```

```mermaid
graph TD
    subgraph Canary [Canary Deployment]
        C1[v1 serving 100%] --> C2[v2 gets 5%]
        C2 --> C3[v2 gets 25%]
        C3 --> C4[v2 gets 100%]
    end
```

```mermaid
graph TD
    subgraph Rolling [Rolling Update]
        R1[Pod1:v1 Pod2:v1 Pod3:v1] --> R2[Pod1:v2 Pod2:v1 Pod3:v1]
        R2 --> R3[Pod1:v2 Pod2:v2 Pod3:v1]
        R3 --> R4[Pod1:v2 Pod2:v2 Pod3:v2]
    end
```

### Strategy Comparison

| Strategy | Downtime | Rollback Speed | Resource Cost | Risk |
|---|---|---|---|---|
| **Blue-Green** | Zero | Instant (switch back) | 2x infrastructure | Low |
| **Canary** | Zero | Fast (route back) | 1x + small % | Very Low |
| **Rolling** | Zero | Slow (rollback pods) | 1x + surge | Medium |
| **Recreate** | Yes | Slow (redeploy) | 1x | High |

---

## DORA Metrics

| Metric | Elite | High | Medium | Low |
|---|---|---|---|---|
| **Deployment Frequency** | Multiple/day | Weekly-Monthly | Monthly-6months | >6months |
| **Lead Time for Changes** | <1 hour | 1day-1week | 1week-1month | >1month |
| **Change Failure Rate** | 0-15% | 16-30% | 16-30% | >30% |
| **MTTR** | <1 hour | <1 day | <1 week | >1 week |

---

## Infrastructure as Code (IaC)

```mermaid
graph LR
    subgraph Provisioning [Infrastructure Provisioning]
        T[Terraform]
        P[Pulumi]
        CF[CloudFormation]
    end
    subgraph ConfigMgmt [Configuration Management]
        A[Ansible]
        Ch[Chef]
        Pu[Puppet]
    end
    Provisioning -->|Creates Resources| Cloud[Cloud Infrastructure]
    ConfigMgmt -->|Configures Resources| Cloud
```

### Declarative vs Imperative

| Approach | Description | Example |
|---|---|---|
| **Declarative** | "I want 3 servers" - system figures out how | Terraform, K8s YAML |
| **Imperative** | "Create server1, then server2, then server3" | Shell scripts, some Ansible |

---

## GitOps

```mermaid
graph LR
    Dev[Developer] -->|Push Code| Git[Git Repo]
    Git -->|Webhook| CI[CI Pipeline]
    CI -->|Build + Test| Artifact[Container Registry]
    CI -->|Update Manifest| GitOps[GitOps Repo]
    GitOps -->|Watched by| ArgoCD[ArgoCD / Flux]
    ArgoCD -->|Sync| K8s[Kubernetes Cluster]
    K8s -->|Status| ArgoCD
```

### Push vs Pull Deployment

| Type | How | Tools | Pros |
|---|---|---|---|
| **Push** | CI pipeline pushes to cluster | Jenkins, Azure Pipelines | Simple, familiar |
| **Pull** | Agent in cluster pulls from Git | ArgoCD, Flux | More secure, self-healing, audit trail |
