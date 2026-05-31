# Chaos Engineering, Buildkite CI/CD & PagerDuty — Complete Guide

> Three topics you have ZERO experience with that REA specifically mentions.
> This guide takes you from zero to interview-ready on each.

---

## TABLE OF CONTENTS

### PART A: CHAOS ENGINEERING
1. [What is Chaos Engineering?](#a1-what)
2. [Principles of Chaos Engineering](#a2-principles)
3. [Chaos Engineering Process](#a3-process)
4. [Types of Chaos Experiments](#a4-types)
5. [Tools — Chaos Mesh, Litmus, Gremlin](#a5-tools)
6. [Chaos Mesh on Kubernetes (Hands-On)](#a6-chaosmesh)
7. [Steady-State Hypothesis](#a7-steadystate)
8. [Game Days](#a8-gamedays)

### PART B: BUILDKITE CI/CD
9. [Buildkite Architecture](#b1-architecture)
10. [Pipeline YAML Syntax](#b2-syntax)
11. [Steps, Plugins, and Hooks](#b3-steps)
12. [Advanced Patterns](#b4-advanced)
13. [Buildkite vs Jenkins/GitHub Actions](#b5-comparison)

### PART C: PAGERDUTY
14. [PagerDuty Concepts](#c1-concepts)
15. [Escalation Policies](#c2-escalation)
16. [Incident Management Workflow](#c3-workflow)
17. [Integration with Monitoring Tools](#c4-integration)
18. [On-Call Best Practices](#c5-oncall)

### PART D: INTERVIEW Q&A
19. [Interview Questions & Answers](#d1-qa)

---

# PART A: CHAOS ENGINEERING

## 1. WHAT IS CHAOS ENGINEERING? <a name="a1-what"></a>

**Chaos Engineering** = The discipline of experimenting on a system to build confidence in its ability to withstand turbulent conditions in production.

```
Traditional Testing:  "Does the system work?"
Chaos Engineering:    "Does the system work when things FAIL?"
```

### Why Do It?

- Microservices increase system complexity → failure modes multiply
- Testing in isolation doesn't catch distributed system failures
- Production incidents reveal gaps that unit/integration tests miss
- Build confidence that redundancy, failover, and alerts actually work

### Real-World Examples (Netflix, Google, REA)

| Failure | What Chaos Engineering Tests |
|---|---|
| Pod crash | Does K8s restart it? Does traffic reroute? |
| Node failure | Do pods reschedule? Does the app stay available? |
| Network partition | Does the circuit breaker activate? |
| DNS failure | Is there graceful degradation? |
| Latency spike | Does the timeout work? Does the downstream handle it? |
| Disk full | Does the alerting fire? Does the app handle it? |

---

## 2. PRINCIPLES OF CHAOS ENGINEERING <a name="a2-principles"></a>

### From the Chaos Engineering Manifesto

1. **Start with a Steady-State Hypothesis**
   - Define "normal" behavior with measurable metrics
   - Example: "Our search API returns 200 with p99 < 500ms"

2. **Vary Real-World Events**
   - Simulate real failures: server crash, network issues, disk full
   - Not random destruction — intentional, controlled experiments

3. **Run Experiments in Production**
   - Staging environments don't replicate production complexity
   - Start small, use blast radius controls

4. **Automate Experiments to Run Continuously**
   - One-time chaos tests become stale
   - Run experiments regularly as part of CI/CD

5. **Minimize Blast Radius**
   - Start with a single pod, not the entire cluster
   - Have a kill switch to abort experiments immediately
   - Have rollback plans ready

---

## 3. CHAOS ENGINEERING PROCESS <a name="a3-process"></a>

```
┌────────────────────────────────────────────────────────┐
│                  CHAOS ENGINEERING LOOP                  │
│                                                         │
│  1. OBSERVE         Define steady-state                 │
│     │               (metrics, SLIs, expected behavior)  │
│     ▼                                                   │
│  2. HYPOTHESIZE     "If pod crashes, K8s will restart   │
│     │                it within 30s and users won't      │
│     │                notice"                            │
│     ▼                                                   │
│  3. EXPERIMENT      Kill a pod, inject latency, etc.    │
│     │                                                   │
│     ▼                                                   │
│  4. VERIFY          Did the hypothesis hold?            │
│     │               Check metrics, SLIs, alerts         │
│     ▼                                                   │
│  5. LEARN           Document findings                   │
│     │               If hypothesis failed → FIX IT       │
│     │               If passed → try harder failures     │
│     ▼                                                   │
│  6. REPEAT          Automate, run regularly             │
└────────────────────────────────────────────────────────┘
```

---

## 4. TYPES OF CHAOS EXPERIMENTS <a name="a4-types"></a>

### Infrastructure Level

| Experiment | What It Tests | Tool |
|---|---|---|
| **Pod kill** | K8s self-healing, pod restart | Chaos Mesh, Litmus |
| **Node drain** | Pod rescheduling, PDB compliance | kubectl, Chaos Mesh |
| **Node failure** | HA, multi-AZ distribution | Chaos Mesh, AWS FIS |
| **Disk fill** | Alerting, graceful handling | Chaos Mesh, stress-ng |
| **CPU stress** | Auto-scaling, resource limits | Chaos Mesh |
| **Memory stress** | OOM handling, resource limits | Chaos Mesh |

### Network Level

| Experiment | What It Tests | Tool |
|---|---|---|
| **Network latency** | Timeouts, circuit breakers | Chaos Mesh, tc |
| **Packet loss** | Retry logic, idempotency | Chaos Mesh |
| **DNS failure** | Fallback, caching | Chaos Mesh |
| **Network partition** | Split-brain handling | Chaos Mesh |
| **Bandwidth throttle** | Degraded performance behavior | Chaos Mesh |

### Application Level

| Experiment | What It Tests | Tool |
|---|---|---|
| **HTTP error injection** | Error handling, retry | Istio fault injection |
| **Dependency failure** | Circuit breaker, fallback | Chaos Mesh, Istio |
| **Config change** | Dynamic config handling | Feature flags |
| **Clock skew** | Time-dependent logic | Chaos Mesh |

---

## 5. TOOLS <a name="a5-tools"></a>

### Comparison

| Feature | Chaos Mesh | Litmus | Gremlin | AWS FIS |
|---|---|---|---|---|
| Open Source | ✅ | ✅ | ❌ (commercial) | ❌ (AWS service) |
| K8s Native | ✅ | ✅ | ✅ | Partial |
| Dashboard | ✅ | ✅ | ✅ | AWS Console |
| Scheduling | ✅ | ✅ | ✅ | ✅ |
| RBAC | ✅ | ✅ | ✅ | IAM |
| Best For | K8s chaos | K8s + litmus hub | Enterprise | AWS infra |

### REA likely uses: Chaos Mesh or Litmus (K8s-native, open source)

---

## 6. CHAOS MESH ON KUBERNETES <a name="a6-chaosmesh"></a>

### Installation

```bash
# Install via Helm
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh \
    -n chaos-mesh --create-namespace \
    --set chaosDaemon.runtime=containerd \
    --set chaosDaemon.socketPath=/run/containerd/containerd.sock
```

### Experiment: Pod Kill

```yaml
# pod-kill-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: property-api-pod-kill
  namespace: chaos-mesh
spec:
  action: pod-kill
  mode: one          # Kill ONE random matching pod
  selector:
    namespaces:
      - production
    labelSelectors:
      app: property-api
  scheduler:
    cron: "@every 2h"   # Run every 2 hours
  duration: "30s"        # N/A for pod-kill, but required
```

### Experiment: Network Latency

```yaml
# network-latency-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: property-api-latency
  namespace: chaos-mesh
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - production
    labelSelectors:
      app: property-api
  delay:
    latency: "200ms"
    correlation: "25"      # 25% of packets affected
    jitter: "50ms"
  duration: "5m"
```

### Experiment: CPU Stress

```yaml
# cpu-stress-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: property-api-cpu-stress
  namespace: chaos-mesh
spec:
  mode: one
  selector:
    namespaces:
      - production
    labelSelectors:
      app: property-api
  stressors:
    cpu:
      workers: 2
      load: 80           # 80% CPU load
  duration: "5m"
```

### Experiment: DNS Failure

```yaml
# dns-failure-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: DNSChaos
metadata:
  name: dns-failure
  namespace: chaos-mesh
spec:
  action: error
  mode: all
  selector:
    namespaces:
      - production
    labelSelectors:
      app: property-api
  patterns:
    - "db.internal.*"     # Only affect DB DNS resolution
  duration: "2m"
```

---

## 7. STEADY-STATE HYPOTHESIS <a name="a7-steadystate"></a>

### How to Define Steady State

```yaml
# chaos-experiment-plan.yaml (NOT actual Chaos Mesh — this is a planning document)

experiment: Property API Pod Kill
hypothesis: |
  If we kill 1 of 3 property-api pods, the service should:
  1. Continue serving requests with < 1% error rate
  2. Kubernetes should restart the killed pod within 30 seconds
  3. No alerts should fire (SLO not breached)
  4. P99 latency should remain under 500ms

steady_state_metrics:
  - metric: availability
    source: prometheus
    query: "sli:availability:ratio_rate5m{service='property-api'}"
    expected: ">= 0.99"
  
  - metric: latency_p99
    source: prometheus
    query: "sli:latency_p99:5m{service='property-api'}"
    expected: "< 0.5"  # seconds
  
  - metric: pod_count
    source: kubernetes
    query: "kubectl get pods -l app=property-api -n production --no-headers | wc -l"
    expected: "== 3"

experiment_action:
  type: pod-kill
  target: 1 random pod with label app=property-api
  namespace: production

verification:
  wait: 60 seconds
  then_check:
    - pod_count == 3 (pod restarted)
    - availability >= 0.99
    - latency_p99 < 0.5
    - no PagerDuty alerts fired

abort_conditions:
  - availability < 0.95 → immediately stop experiment
  - more than 1 pod in CrashLoopBackOff → stop
```

---

## 8. GAME DAYS <a name="a8-gamedays"></a>

**Game Day** = A planned event where the team runs chaos experiments and practices incident response.

### Game Day Agenda

```
1. Pre-Game (30 min)
   - Review runbooks and escalation policies
   - Ensure all monitoring dashboards are accessible
   - Define experiments for today
   - Assign roles: experimenter, observer, scribe

2. Experiments (2-3 hours)
   - Run experiments one at a time
   - Observe system behavior vs hypothesis
   - Practice incident response if something unexpected happens
   - Document everything

3. Debrief (30 min)
   - What worked as expected?
   - What surprised us?
   - What action items do we have?
   - Update runbooks based on findings
```

---

# PART B: BUILDKITE CI/CD

## 9. BUILDKITE ARCHITECTURE <a name="b1-architecture"></a>

### How Buildkite Is Different

```
┌──── Traditional CI (Jenkins/GH Actions) ────┐
│ CI Server runs builds on its own machines    │
│ You manage the CI infrastructure             │
└──────────────────────────────────────────────┘

┌──── Buildkite (Hybrid Model) ───────────────┐
│ Buildkite.com (SaaS)                         │
│ ├── Manages pipeline definitions             │
│ ├── Provides web UI & API                    │
│ ├── Handles scheduling & routing             │
│ └── Does NOT run your code                   │
│                                              │
│ Your Infrastructure                          │
│ ├── Buildkite Agents (run on YOUR machines)  │
│ ├── Code runs inside your network            │
│ └── You control security & access            │
└──────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description | Jenkins Equivalent |
|---|---|---|
| **Pipeline** | A series of steps to run | Job/Pipeline |
| **Step** | A single unit of work | Stage/Step |
| **Agent** | Worker that runs steps | Node/Agent |
| **Queue** | Group of agents by capability | Labels |
| **Build** | One execution of a pipeline | Build |
| **Artifact** | Files produced by steps | Artifacts |
| **Plugin** | Reusable step extensions | Plugins |

---

## 10. PIPELINE YAML SYNTAX <a name="b2-syntax"></a>

### Basic Pipeline

```yaml
# .buildkite/pipeline.yml

steps:
  # Simple command step
  - label: "🔨 Build"
    command: "make build"

  # Wait for previous step
  - wait

  # Test step
  - label: "🧪 Test"
    command: "make test"

  # Wait with continue on failure
  - wait: ~
    continue_on_failure: true

  # Deploy step (requires manual approval)
  - block: "Deploy to Production?"
    prompt: "Are you sure you want to deploy to production?"

  - label: "🚀 Deploy"
    command: "make deploy"
    branches: "main"
```

### Parallel Steps

```yaml
steps:
  - label: "🔨 Build"
    command: "make build"

  - wait

  # These run in parallel:
  - label: "🧪 Unit Tests"
    command: "make test-unit"

  - label: "🧪 Integration Tests"
    command: "make test-integration"

  - label: "🔍 Lint"
    command: "make lint"

  # Wait for ALL parallel steps
  - wait

  - label: "🚀 Deploy"
    command: "make deploy"
```

### Environment Variables & Agents

```yaml
steps:
  - label: "Build & Push Docker Image"
    command: |
      docker build -t $ECR_REPO:$BUILDKITE_BUILD_NUMBER .
      docker push $ECR_REPO:$BUILDKITE_BUILD_NUMBER
    env:
      ECR_REPO: "123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/property-api"
    agents:
      queue: "docker"      # Run on agents with Docker capability

  - label: "Deploy to K8s"
    command: |
      kubectl set image deployment/property-api \
        api=$ECR_REPO:$BUILDKITE_BUILD_NUMBER \
        -n production
    agents:
      queue: "k8s-deploy"  # Run on agents with kubectl access
```

### Key Built-in Environment Variables

```
BUILDKITE_BUILD_NUMBER       → Build number (auto-incremented)
BUILDKITE_BRANCH             → Git branch
BUILDKITE_COMMIT             → Git commit SHA
BUILDKITE_MESSAGE            → Commit message
BUILDKITE_PIPELINE_SLUG      → Pipeline name (URL-safe)
BUILDKITE_PULL_REQUEST       → PR number (or "false")
BUILDKITE_TAG                → Git tag (if triggered by tag)
BUILDKITE_AGENT_NAME         → Name of the agent running this step
BUILDKITE_ARTIFACT_PATHS     → Glob pattern for artifact upload
```

---

## 11. STEPS, PLUGINS, AND HOOKS <a name="b3-steps"></a>

### Step Types

```yaml
steps:
  # 1. Command Step — runs a command
  - label: "Build"
    command: "make build"

  # 2. Wait Step — synchronization point
  - wait

  # 3. Block Step — manual gate
  - block: "Deploy?"
    fields:
      - text: "Deploy environment"
        key: "deploy-env"
        default: "staging"

  # 4. Input Step — collect input without blocking
  - input: "Configure deployment"
    fields:
      - select: "Region"
        key: "region"
        options:
          - label: "Sydney"
            value: "ap-southeast-2"
          - label: "Mumbai"
            value: "ap-south-1"

  # 5. Trigger Step — trigger another pipeline
  - trigger: "deploy-pipeline"
    label: "Trigger Deploy"
    build:
      branch: "main"
      env:
        VERSION: "${BUILDKITE_BUILD_NUMBER}"

  # 6. Group Step — organize steps visually
  - group: "🧪 Tests"
    steps:
      - label: "Unit"
        command: "make test-unit"
      - label: "Integration"
        command: "make test-integration"
```

### Plugins

```yaml
steps:
  # Docker plugin — run in a Docker container
  - label: "Test in Docker"
    plugins:
      - docker#v5.8.0:
          image: "python:3.12"
          command: ["pytest", "-v"]

  # Docker Compose plugin
  - label: "Integration Test"
    plugins:
      - docker-compose#v4.16.0:
          run: app
          command: ["make", "test-integration"]

  # ECR plugin — authenticate to ECR
  - label: "Push to ECR"
    plugins:
      - ecr#v2.7.0:
          login: true
          region: "ap-southeast-2"
    command: "docker push $IMAGE"

  # Kubernetes plugin — run step in K8s pod
  - label: "Deploy"
    plugins:
      - kubernetes:
          podSpec:
            containers:
              - image: bitnami/kubectl
                command: ["kubectl", "apply", "-f", "manifests/"]

  # Artifacts plugin
  - label: "Build"
    command: "make build"
    artifact_paths:
      - "build/**/*"
      - "coverage/report.html"
```

### Hooks

```bash
# .buildkite/hooks/pre-command
# Runs before every command step
#!/bin/bash
echo "--- Setting up environment"
export AWS_REGION=ap-southeast-2
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_URL

# .buildkite/hooks/post-command
# Runs after every command step
#!/bin/bash
echo "--- Cleanup"
docker system prune -f
```

---

## 12. ADVANCED PATTERNS <a name="b4-advanced"></a>

### Dynamic Pipelines

```yaml
# .buildkite/pipeline.yml
steps:
  - label: "Generate Pipeline"
    command: ".buildkite/generate-pipeline.sh | buildkite-agent pipeline upload"

# generate-pipeline.sh generates YAML dynamically based on what changed
```

```bash
#!/bin/bash
# .buildkite/generate-pipeline.sh
# Only test changed services

CHANGED_DIRS=$(git diff --name-only HEAD~1 | cut -d'/' -f1 | sort -u)

echo "steps:"
for dir in $CHANGED_DIRS; do
    if [ -d "$dir" ] && [ -f "$dir/Makefile" ]; then
        echo "  - label: '🧪 Test $dir'"
        echo "    command: 'cd $dir && make test'"
    fi
done
```

### Retry Logic

```yaml
steps:
  - label: "Flaky Test"
    command: "make test"
    retry:
      automatic:
        - exit_status: 1
          limit: 2        # Retry up to 2 times on exit code 1
        - exit_status: -1
          limit: 1        # Retry once on agent lost
      manual:
        allowed: true     # Allow manual retry from UI
```

### Conditional Steps

```yaml
steps:
  - label: "Deploy to Staging"
    command: "make deploy-staging"
    branches: "main develop"

  - label: "Deploy to Production"
    command: "make deploy-prod"
    branches: "main"
    if: "build.tag =~ /^v/"    # Only on version tags

  - label: "PR Check"
    command: "make lint"
    if: "build.pull_request.id != null"
```

### Full REA-Style Pipeline

```yaml
# .buildkite/pipeline.yml — REA Property API
steps:
  - group: "📋 Quality Checks"
    steps:
      - label: "🔍 Lint"
        command: "make lint"
        plugins:
          - docker#v5.8.0:
              image: "golangci/golangci-lint:latest"

      - label: "🔐 Security Scan"
        command: "trivy fs --exit-code 1 --severity HIGH,CRITICAL ."
        plugins:
          - docker#v5.8.0:
              image: "aquasec/trivy:latest"

  - wait

  - group: "🧪 Tests"
    steps:
      - label: "Unit Tests"
        command: "make test-unit"
        artifact_paths: "coverage.html"
        plugins:
          - docker#v5.8.0:
              image: "golang:1.22"

      - label: "Integration Tests"
        command: "make test-integration"
        plugins:
          - docker-compose#v4.16.0:
              run: test
        agents:
          queue: "integration"

  - wait

  - label: "🐳 Build & Push Image"
    command: |
      IMAGE="${ECR_REPO}:${BUILDKITE_BUILD_NUMBER}"
      docker build -t $IMAGE .
      docker push $IMAGE
    plugins:
      - ecr#v2.7.0:
          login: true
          region: "ap-southeast-2"
    env:
      ECR_REPO: "123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/property-api"

  - wait

  - label: "🚀 Deploy to Staging"
    command: ".buildkite/scripts/deploy.sh staging"
    branches: "main"
    agents:
      queue: "k8s-deploy"

  - wait

  - label: "🧪 Smoke Test Staging"
    command: ".buildkite/scripts/smoke-test.sh staging"

  - wait

  - block: "🚨 Deploy to Production?"
    branches: "main"
    prompt: "Staging smoke tests passed. Deploy to production?"

  - label: "🚀 Deploy to Production"
    command: ".buildkite/scripts/deploy.sh production"
    branches: "main"
    agents:
      queue: "k8s-deploy"
    concurrency: 1
    concurrency_group: "production-deploy"

  - wait

  - label: "🧪 Smoke Test Production"
    command: ".buildkite/scripts/smoke-test.sh production"
```

---

## 13. BUILDKITE vs JENKINS/GITHUB ACTIONS <a name="b5-comparison"></a>

| Feature | Buildkite | Jenkins | GitHub Actions |
|---|---|---|---|
| Hosting | Hybrid (SaaS + your agents) | Self-hosted | GitHub-hosted or self-hosted |
| Config | YAML (pipeline.yml) | Groovy (Jenkinsfile) | YAML (workflows/) |
| Agents | Your infrastructure | Your infrastructure | GitHub/self-hosted runners |
| UI | Clean, modern | Dated but functional | GitHub-integrated |
| Plugins | Plugin system | Extensive plugins | Marketplace actions |
| Docker | First-class support | Via plugins | Built-in |
| Parallelism | Built-in | Via plugins | matrix strategy |
| Security | Code stays on your infra | Code on your infra | Code on GitHub infra |
| Cost | Per-user pricing | Free (OSS) | Free tier + per-minute |

### Key Differentiator
"Buildkite's hybrid model means the CI/CD orchestration is SaaS (no infra to manage for the control plane), but your code and builds run on YOUR infrastructure — so sensitive code never leaves your network. This is why security-conscious companies like REA use it."

---

# PART C: PAGERDUTY

## 14. PAGERDUTY CONCEPTS <a name="c1-concepts"></a>

### What is PagerDuty?

PagerDuty is an incident management platform that:
- Receives alerts from monitoring tools (Prometheus, CloudWatch, Splunk, Datadog)
- Routes them to the right on-call person
- Escalates if not acknowledged
- Tracks incident lifecycle (triggered → acknowledged → resolved)

### Core Components

| Component | Purpose | Example |
|---|---|---|
| **Service** | Represents a business service | "Property Search API" |
| **Integration** | Connects monitoring to PagerDuty | Prometheus → PagerDuty |
| **Escalation Policy** | Defines who gets paged and when | L1 → L2 → Manager |
| **Schedule** | On-call rotation | Weekly rotation among 4 engineers |
| **Incident** | An active problem | "High error rate on property-api" |
| **Alert** | Raw signal from monitoring | Prometheus fired "HighErrorRate" |
| **Event** | API payload that creates alerts | JSON sent to PagerDuty Events API |

### Alert Flow

```
Monitoring (Prometheus/CloudWatch/Splunk)
         │
         │ Alert fires
         ▼
    PagerDuty Service
         │
         │ Creates incident
         ▼
    Escalation Policy
         │
         │ Notifies on-call person
         ▼
    On-Call Engineer
    ├── Phone call
    ├── SMS
    ├── Push notification
    ├── Slack
    └── Email
         │
         │ If not acknowledged within 5 min
         ▼
    ESCALATE to next level
```

---

## 15. ESCALATION POLICIES <a name="c2-escalation"></a>

### Example Escalation Policy

```
Property Search API — Escalation Policy
────────────────────────────────────────
Level 1 (0 min):
  → On-call engineer (from weekly rotation schedule)
  → Notify: push notification + SMS

Level 2 (5 min, if L1 doesn't acknowledge):
  → Secondary on-call engineer
  → Notify: phone call + SMS

Level 3 (15 min, if L2 doesn't acknowledge):
  → Engineering Manager + Team Lead
  → Notify: phone call

Level 4 (30 min, if L3 doesn't acknowledge):
  → VP Engineering + CTO
  → Notify: phone call
```

### On-Call Schedule

```
Week 1: Engineer A (primary), Engineer B (secondary)
Week 2: Engineer B (primary), Engineer C (secondary)
Week 3: Engineer C (primary), Engineer D (secondary)
Week 4: Engineer D (primary), Engineer A (secondary)
(repeat)
```

---

## 16. INCIDENT MANAGEMENT WORKFLOW <a name="c3-workflow"></a>

### Incident Lifecycle

```
┌──────────────────────────────────────────────────────┐
│                 INCIDENT LIFECYCLE                     │
│                                                       │
│  TRIGGERED ──► ACKNOWLEDGED ──► RESOLVED              │
│     │              │                │                  │
│     │              │                │                  │
│   Alert fires   Engineer sees    Issue fixed           │
│   Page sent     it, starts       Incident closed       │
│                 investigating                          │
│                                                        │
│  Auto-resolve: If monitoring says "OK" → auto-resolve  │
│  Auto-escalate: If not ack'd in X min → next level     │
└────────────────────────────────────────────────────────┘
```

### Severity Levels

| Severity | Description | Response Time | Example |
|---|---|---|---|
| **SEV-1 (P1)** | Critical — service completely down | Immediate (< 5 min) | Property search returns 100% errors |
| **SEV-2 (P2)** | Major — significant impact | < 15 min | Search works but latency > 5s |
| **SEV-3 (P3)** | Minor — degraded but functional | < 1 hour | One region's search slow |
| **SEV-4 (P4)** | Low — cosmetic or minor | Next business day | Dashboard widget broken |

### Incident Commander (IC) Role

For SEV-1/SEV-2 incidents:
```
Incident Commander responsibilities:
1. Declare the incident severity
2. Create a Slack war room
3. Assign roles (IC, Communication Lead, Subject Matter Experts)
4. Coordinate investigation (NOT doing hands-on debugging)
5. Provide regular updates to stakeholders (every 15-30 min)
6. Decide on mitigation actions (rollback? failover?)
7. Declare incident resolved
8. Schedule postmortem
```

---

## 17. INTEGRATION WITH MONITORING <a name="c4-integration"></a>

### Prometheus → PagerDuty (via Alertmanager)

```yaml
# alertmanager.yml
global:
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

route:
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      repeat_interval: 5m
    - match:
        severity: warning
      receiver: 'pagerduty-warning'
      repeat_interval: 30m

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
  
  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<PAGERDUTY_SERVICE_KEY>'
        severity: critical
        description: '{{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          dashboard: '{{ .CommonAnnotations.dashboard }}'
          runbook: '{{ .CommonAnnotations.runbook }}'
  
  - name: 'pagerduty-warning'
    pagerduty_configs:
      - service_key: '<PAGERDUTY_SERVICE_KEY>'
        severity: warning
```

### PagerDuty Events API v2

```python
import requests
import json

def trigger_pagerduty_incident(
    routing_key: str,
    summary: str,
    severity: str = "critical",
    source: str = "monitoring",
    component: str = "property-api",
    custom_details: dict = None,
):
    """Trigger a PagerDuty incident via Events API v2."""
    payload = {
        "routing_key": routing_key,
        "event_action": "trigger",
        "payload": {
            "summary": summary,
            "severity": severity,  # critical, error, warning, info
            "source": source,
            "component": component,
            "custom_details": custom_details or {},
        },
        "links": [{
            "href": "https://grafana.rea.com/d/property-api",
            "text": "Grafana Dashboard"
        }],
    }
    
    resp = requests.post(
        "https://events.pagerduty.com/v2/enqueue",
        json=payload,
        timeout=10,
    )
    return resp.json()

# Usage:
# trigger_pagerduty_incident(
#     routing_key="R0123456789ABCDEF",
#     summary="Property API error rate > 5% for 10 minutes",
#     severity="critical",
#     custom_details={"error_rate": "7.2%", "affected_pods": 3},
# )
```

### Auto-Resolution

```python
def resolve_pagerduty_incident(routing_key: str, dedup_key: str):
    """Resolve a PagerDuty incident."""
    payload = {
        "routing_key": routing_key,
        "event_action": "resolve",
        "dedup_key": dedup_key,  # Same key used when triggering
    }
    resp = requests.post(
        "https://events.pagerduty.com/v2/enqueue",
        json=payload,
        timeout=10,
    )
    return resp.json()
```

---

## 18. ON-CALL BEST PRACTICES <a name="c5-oncall"></a>

### For the Interview — What REA Wants to Hear

1. **Actionable alerts only**: Every page should require human action. If it can be auto-remediated, automate it.

2. **Runbooks for every alert**: When paged, the engineer should have a documented procedure to follow.

3. **On-call handoff**: At the end of each rotation, outgoing on-call briefs incoming on any ongoing issues.

4. **Postmortems are blameless**: Focus on systemic improvements, not individual blame.

5. **On-call load balancing**: Track page frequency. If one team gets paged 10x more, redistribute or fix the root causes.

6. **Compensation**: On-call work should be compensated (time off or pay).

7. **Alert fatigue prevention**:
   - Deduplicate alerts (PagerDuty does this)
   - Group related alerts into one incident
   - Suppress during maintenance windows
   - Regularly review and tune alert thresholds

---

# PART D: INTERVIEW Q&A

## 19. INTERVIEW QUESTIONS & ANSWERS <a name="d1-qa"></a>

### Q1: "What is chaos engineering and why is it important for a platform team?"

**Answer**: "Chaos engineering is the practice of deliberately injecting failures into a system to verify it can handle them. For a platform team, it's critical because we build the infrastructure other teams rely on.

We might test: What happens when a Kubernetes node dies? Does the pod rescheduling work? Does the autoscaler respond? Do alerts fire correctly?

The process is: define what 'normal' looks like (steady-state hypothesis), inject a failure, observe if the hypothesis holds, and fix any gaps. We start small — kill a single pod — and gradually increase severity.

At REA, I'd integrate chaos experiments into the CI/CD pipeline — running them regularly in staging, and eventually controlled experiments in production during game days."

### Q2: "How would you set up CI/CD with Buildkite?"

**Answer**: "Buildkite uses a hybrid model — the orchestration is SaaS but builds run on your own agents, so code never leaves your network. The pipeline is defined in `.buildkite/pipeline.yml`.

For a platform service, I'd set up:
1. Quality gates: linting, security scanning (Trivy), and unit tests — running in parallel
2. Docker build and push to ECR
3. Deploy to staging automatically on main branch
4. Smoke tests against staging
5. Manual approval gate (block step) before production
6. Canary deployment to production with automated rollback

I'd use Buildkite plugins for Docker, ECR authentication, and Kubernetes deployment. For a mono-repo, I'd use dynamic pipelines that only test changed services."

### Q3: "Describe your incident management process."

**Answer**: "I follow a structured incident management process:

**Detection**: Monitoring (Prometheus/Splunk) detects anomaly → PagerDuty alert triggers → on-call engineer is paged.

**Response**: Engineer acknowledges within 5 minutes. For SEV-1/2, an Incident Commander is assigned to coordinate — they DON'T debug, they coordinate.

**Communication**: Regular updates to stakeholders via Slack war room. Status page updated for external-facing incidents.

**Mitigation**: First priority is restoring service — rollback, failover, scale up — whatever is fastest. Root cause investigation comes after service is stable.

**Resolution**: Confirm service is healthy, close the incident in PagerDuty.

**Postmortem**: Blameless review within 48 hours. Document what happened, timeline, root cause, and action items to prevent recurrence. Action items get tracked to completion."

### Q4: "How do you prevent alert fatigue?"

**Answer**: "Alert fatigue is when engineers stop paying attention because they get too many non-actionable alerts. I prevent it by:

1. **Every alert must be actionable** — if you can't do anything about it, don't page for it. Convert to a dashboard metric instead.
2. **Use severity levels** — only `critical` pages the on-call. `warning` goes to Slack, `info` goes to dashboards.
3. **Deduplicate and group** — PagerDuty groups related alerts into one incident. If 50 pods all fail at once, that's one incident, not 50 pages.
4. **Regular alert review** — monthly review of alert frequency. If an alert fires 20 times without action, tune or remove it.
5. **Maintenance windows** — suppress alerts during planned maintenance.
6. **SLO-based alerting** — use burn rate alerts instead of threshold alerts. They're more meaningful and fire less often."

### Q5: "How would you run a Game Day?"

**Answer**: "A Game Day is a planned chaos engineering session where the team practices handling failures.

**Prep**: Choose 3-4 experiments (pod kill, network latency, node failure). Define steady-state hypotheses for each. Ensure monitoring dashboards are ready. Brief the team.

**Execution**: Run one experiment at a time. One person runs the experiment, others observe dashboards and respond as if it's a real incident. Practice the full incident workflow — detection, response, communication, mitigation.

**After each experiment**: Did the hypothesis hold? Did alerts fire? Did runbooks help? Was recovery fast enough?

**Debrief**: Document findings, create tickets for any gaps discovered (like a missing alert, a runbook that's outdated, or a service that didn't failover correctly).

I'd start with staging and run Game Days quarterly, eventually doing controlled production experiments for critical services."
