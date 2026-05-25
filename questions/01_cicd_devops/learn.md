# CI/CD & DevOps — Deep-Dive Learning Guide

---

## 1. What Is DevOps?

DevOps is a **culture + set of practices** that unifies software development (Dev) and IT operations (Ops) to shorten the development lifecycle and deliver high-quality software continuously.

```
┌──────────────────────── DevOps Infinity Loop ──────────────────────┐
│                                                                     │
│         Plan → Code → Build → Test                                 │
│        ↑                          ↓                                 │
│     Monitor                     Release                            │
│        ↑                          ↓                                 │
│       Operate  ←  Deploy  ←  Approve                               │
│                                                                     │
│  Dev side: Plan, Code, Build, Test                                 │
│  Ops side: Release, Deploy, Operate, Monitor                       │
│  DevOps: BOTH sides collaborating, automated, with feedback loops  │
└─────────────────────────────────────────────────────────────────────┘
```

### DevOps vs Traditional

| Aspect | Traditional (Waterfall/Siloed) | DevOps |
|--------|-------------------------------|--------|
| Releases | Monthly/quarterly | Multiple per day |
| Dev & Ops | Separate teams, throw over wall | Same team or close collaboration |
| Feedback | Weeks/months | Minutes (monitoring, alerts) |
| Infrastructure | Manual, snowflake servers | IaC, immutable, disposable |
| Testing | Manual, at the end | Automated, continuous |
| Failures | Blame, postmortems | Blameless, learning culture |

---

## 2. CI vs CD vs CD — The Three Stages

```
┌─── Continuous Integration (CI) ───────────────────────────────────┐
│                                                                    │
│  Developer ──► git push ──► Auto Build ──► Auto Test ──► Artifact │
│                                                                    │
│  Goal: Merge code frequently, catch bugs early                    │
│  Frequency: Every commit or PR                                    │
│  Output: Build artifact (Docker image, JAR, binary)               │
└────────────────────────────────────────────────────────────────────┘
                              │
┌─── Continuous Delivery (CD) ──────────────────────────────────────┐
│                                                                    │
│  Artifact ──► Deploy to Staging ──► Integration Tests             │
│                                       │                            │
│                              ┌────────▼─────────┐                 │
│                              │ Manual Approval   │                 │
│                              │ Gate (human click)│                 │
│                              └────────┬─────────┘                 │
│                                       │                            │
│  Goal: Always deployable, human decides when to ship              │
└────────────────────────────────────────────────────────────────────┘
                              │
┌─── Continuous Deployment (CD) ────────────────────────────────────┐
│                                                                    │
│  Artifact ──► Auto Deploy to Staging ──► Auto Deploy to Prod      │
│                                                                    │
│  Goal: Every passing change goes to production automatically      │
│  No human gate — requires very high test confidence               │
└────────────────────────────────────────────────────────────────────┘
```

| Term | Human Approval? | Risk | Maturity |
|------|----------------|------|----------|
| CI | N/A | Low | Basic |
| Continuous Delivery | Yes (before prod) | Medium | Intermediate |
| Continuous Deployment | No (fully automated) | Needs high test coverage | Advanced |

---

## 3. Pipeline Architecture

```
┌──── Trigger ────────────────────────────────────────────────────┐
│  git push / PR / schedule / manual / webhook / tag              │
└────────┬────────────────────────────────────────────────────────┘
         │
┌────────▼──── Stage 1: BUILD ────────────────────────────────────┐
│  1. Checkout source code                                        │
│  2. Install dependencies                                        │
│  3. Compile / build                                             │
│  4. Create artifact (Docker image, binary, package)             │
│  5. Push artifact to registry/store                             │
└────────┬────────────────────────────────────────────────────────┘
         │
┌────────▼──── Stage 2: TEST ─────────────────────────────────────┐
│  1. Unit tests         (fast, isolated, mock deps)              │
│  2. Integration tests  (real DB, real API calls)                │
│  3. Static analysis    (SonarQube, pylint, ESLint)              │
│  4. Security scan      (Trivy, Snyk, OWASP ZAP)                │
│  5. Code coverage      (fail if < threshold)                    │
└────────┬────────────────────────────────────────────────────────┘
         │
┌────────▼──── Stage 3: DEPLOY TO STAGING ────────────────────────┐
│  1. Deploy artifact to staging environment                      │
│  2. Run smoke tests / E2E tests                                 │
│  3. Performance tests (optional)                                │
└────────┬────────────────────────────────────────────────────────┘
         │
┌────────▼──── Stage 4: APPROVAL GATE ────────────────────────────┐
│  Manual approval (Continuous Delivery)                          │
│  — OR —                                                         │
│  Automatic (Continuous Deployment)                              │
└────────┬────────────────────────────────────────────────────────┘
         │
┌────────▼──── Stage 5: DEPLOY TO PRODUCTION ─────────────────────┐
│  1. Rolling update / Blue-Green / Canary                        │
│  2. Health checks                                               │
│  3. Rollback if health check fails                              │
│  4. Notify team (Slack, Teams, email)                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Deployment Strategies

### Rolling Update

```
Before:   [v1] [v1] [v1] [v1]
Step 1:   [v1] [v1] [v1] [v2]  ← start 1 new, kill 1 old
Step 2:   [v1] [v1] [v2] [v2]
Step 3:   [v1] [v2] [v2] [v2]
Step 4:   [v2] [v2] [v2] [v2]  ← done, zero downtime
```

### Blue-Green

```
             Load Balancer
                  │
       ┌──────────┴──────────┐
       │                      │
  ┌────▼─────┐          ┌────▼─────┐
  │  BLUE    │          │  GREEN   │
  │  (v1)    │          │  (v2)    │
  │  LIVE ✓  │          │  IDLE    │
  └──────────┘          └──────────┘

  Step 1: Deploy v2 to GREEN (BLUE still serves traffic)
  Step 2: Test GREEN thoroughly
  Step 3: Switch LB to GREEN (instant cutover)
  Step 4: Keep BLUE as rollback (switch back instantly if issues)
```

### Canary

```
             Load Balancer
                  │
       ┌──────────┴──────────┐
       │ 90%                  │ 10%
  ┌────▼─────┐          ┌────▼─────┐
  │ Stable   │          │ Canary   │
  │ (v1)     │          │ (v2)     │
  │ 9 pods   │          │ 1 pod    │
  └──────────┘          └──────────┘

  Step 1: Deploy v2 to 1 pod (10% traffic)
  Step 2: Monitor error rate, latency, logs
  Step 3: If good → gradually increase to 100%
  Step 4: If bad → route 100% back to v1 (only 10% users affected)
```

| Strategy | Downtime? | Rollback Speed | Resource Cost | Best For |
|----------|-----------|----------------|---------------|----------|
| Rolling | No | Slow (re-roll) | 1x + surge | Default |
| Blue-Green | No | Instant (switch LB) | 2x (both up) | Critical apps |
| Canary | No | Fast (kill canary) | 1x + small | Risk-averse |
| Recreate | Yes | Slow | 1x | DB migrations |

---

## 5. Artifact Management

```
┌─── Build produces ───┐     ┌─── Stored in ───────────────────┐
│                       │     │                                  │
│  Docker Image         │────►│  Registry: ACR, ECR, Docker Hub │
│  JAR/WAR              │────►│  Artifactory, Nexus              │
│  npm package          │────►│  npm registry, GitHub Packages   │
│  Python wheel         │────►│  PyPI, Azure Artifacts           │
│  NuGet package        │────►│  NuGet Gallery, Azure Artifacts  │
│  Binary/executable    │────►│  S3, Azure Blob, GitHub Releases │
│                       │     │                                  │
└───────────────────────┘     └──────────────────────────────────┘
```

### Versioning strategies

```
Semantic Versioning:  MAJOR.MINOR.PATCH  (e.g., 2.3.1)
  MAJOR: breaking changes
  MINOR: new features, backward compatible
  PATCH: bug fixes

Git-based:  v1.2.3-<short-sha>  (e.g., v1.2.3-a1b2c3d)
Date-based: 2026.05.25.3        (year.month.day.buildnum)
```

---

## 6. Testing Pyramid

```
          ┌─────────┐
          │  E2E    │  Slow, expensive, few
          │  Tests  │  (Selenium, Playwright, Cypress)
          ├─────────┤
          │Integration│  Medium speed, medium count
          │  Tests   │  (real DB, real APIs)
          ├──────────┤
          │  Unit    │  Fast, cheap, many (80%+)
          │  Tests   │  (mocked deps, isolated)
          └──────────┘
```

| Level | Speed | Scope | Count | Tools |
|-------|-------|-------|-------|-------|
| Unit | ms | Single function/class | 100s-1000s | pytest, JUnit, Jest |
| Integration | seconds | Multiple components | 10s-100s | Testcontainers, Postman |
| E2E | minutes | Full user workflow | 5-20 | Selenium, Cypress, Playwright |

---

## 7. GitOps

```
┌─── Traditional Push Model ──────────────────────────────┐
│  CI Pipeline ──► builds ──► pushes ──► deploys to K8s  │
│  Pipeline has cluster credentials (security risk!)      │
└─────────────────────────────────────────────────────────┘

┌─── GitOps Pull Model ──────────────────────────────────┐
│                                                         │
│  Developer ──► git push manifest changes               │
│                     │                                   │
│              ┌──────▼───────┐                          │
│              │  Git Repo    │  ← single source of truth │
│              │  (manifests) │                          │
│              └──────┬───────┘                          │
│                     │ watches                           │
│              ┌──────▼───────┐                          │
│              │  ArgoCD /    │  ← runs INSIDE cluster   │
│              │  FluxCD      │  ← no external creds!    │
│              │  (operator)  │                          │
│              └──────┬───────┘                          │
│                     │ applies                           │
│              ┌──────▼───────┐                          │
│              │  Kubernetes  │                          │
│              │  Cluster     │                          │
│              └──────────────┘                          │
└─────────────────────────────────────────────────────────┘
```

**Principles**:
1. Git is the single source of truth for declarative infrastructure
2. All changes via Git (PR → review → merge → auto-deploy)
3. Agent in cluster pulls changes (no push, no external credentials)
4. Self-healing: if someone manually changes cluster state, agent reverts it

---

## 8. Key DevOps Metrics (DORA)

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| **Deployment Frequency** | On-demand (multiple/day) | Weekly-monthly | Monthly-6mo | <1/6mo |
| **Lead Time for Changes** | <1 hour | 1 day - 1 week | 1-6 months | >6 months |
| **Change Failure Rate** | 0-15% | 16-30% | 16-30% | >30% |
| **Time to Restore Service** | <1 hour | <1 day | 1 day-1 week | >6 months |

---

## 9. Pipeline Security (Shift-Left)

```
Traditional:          Security check ──────────────────────► at the end
Shift-Left:  Security ──► at every stage ──► continuous

┌─── Pipeline with Shift-Left Security ──────────────────────────┐
│                                                                 │
│  Code    → pre-commit hooks (secrets detection, linting)       │
│  Build   → dependency scan (npm audit, pip-audit)              │
│  Test    → SAST (SonarQube), unit test coverage                │
│  Image   → container scan (Trivy), Dockerfile lint             │
│  Deploy  → DAST (OWASP ZAP), infrastructure scan              │
│  Runtime → runtime protection, WAF, monitoring                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Infrastructure as Code in CI/CD

```
┌─── IaC Pipeline ────────────────────────────────────────────────┐
│                                                                  │
│  git push terraform/ ──► terraform fmt ──► terraform validate   │
│                              │                                   │
│                     ┌────────▼──────────┐                       │
│                     │ terraform plan    │  ← shows what changes │
│                     │ (plan as artifact)│                        │
│                     └────────┬──────────┘                       │
│                              │                                   │
│                     ┌────────▼──────────┐                       │
│                     │ Manual Approval   │  ← review plan output │
│                     └────────┬──────────┘                       │
│                              │                                   │
│                     ┌────────▼──────────┐                       │
│                     │ terraform apply   │  ← apply saved plan   │
│                     └───────────────────┘                       │
└──────────────────────────────────────────────────────────────────┘
```

---

## 11. Key Terminology Quick Reference

| Term | Definition |
|------|-----------|
| **Pipeline** | Automated workflow: trigger → build → test → deploy |
| **Stage** | Logical grouping of jobs (Build, Test, Deploy) |
| **Job** | Unit of work that runs on an agent (e.g., "run unit tests") |
| **Step/Task** | Individual command within a job |
| **Agent/Runner** | Machine that executes jobs (self-hosted or cloud) |
| **Artifact** | Output of build (Docker image, binary, package) |
| **Trigger** | Event starting pipeline (push, PR, schedule, webhook, manual) |
| **Gate** | Approval or condition before proceeding |
| **Environment** | Target (dev, staging, prod) with protection rules |
| **Variable Group** | Shared variables across pipelines |
| **Secret** | Encrypted variable (API keys, passwords) |
| **Cache** | Saved dependencies between runs (node_modules, pip cache) |
