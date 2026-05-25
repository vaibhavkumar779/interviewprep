# Scenarios & Behavioral — Deep-Dive Learning Guide

---

## 1. STAR Method Framework

Every behavioral answer should follow **STAR**:

```
┌────────────────────────────────────────────────────────────┐
│  S — Situation   │  Context: Where, when, what project?    │
│                  │  Keep it brief (2-3 sentences)           │
├──────────────────┼─────────────────────────────────────────┤
│  T — Task        │  Your specific responsibility            │
│                  │  What was expected of YOU?               │
├──────────────────┼─────────────────────────────────────────┤
│  A — Action      │  What YOU did (not the team)            │
│                  │  Be specific: tools, steps, decisions   │
│                  │  This is 60% of your answer             │
├──────────────────┼─────────────────────────────────────────┤
│  R — Result      │  Measurable outcome                     │
│                  │  Numbers, percentages, time saved       │
│                  │  What did you learn?                    │
└────────────────────────────────────────────────────────────┘
```

### Rules

```
✅ Use "I" not "we" — interviewers want YOUR contribution
✅ Quantify results (reduced by 40%, saved 2 hours/day)
✅ Keep answers 2-3 minutes
✅ Prepare 6-8 stories that cover multiple question types
✅ Be honest — interviewers detect fabrication
❌ Don't ramble or give unnecessary backstory
❌ Don't badmouth previous employers
❌ Don't say "we" for everything — show YOUR ownership
```

---

## 2. Common Behavioral Questions & Frameworks

### Tell Me About Yourself (2-minute pitch)

```
Structure:
  1. Current Role   — "I'm currently working as..."
  2. Key Experience  — "Over N years, I've..."
  3. Why This Role   — "I'm excited about Ciena because..."

Example:
  "I'm a DevOps engineer with experience building and maintaining
  CI/CD pipelines, container platforms, and cloud infrastructure.
  In my current role, I've automated deployment workflows using
  Jenkins and Azure DevOps, containerized applications with Docker
  and Kubernetes, and managed infrastructure using Terraform and
  Ansible. I'm drawn to Ciena because of the opportunity to work
  with networking technology at scale, especially the intersection
  of DevOps and embedded systems."
```

### Why Ciena?

```
Research points to mention:
  - Ciena is a leader in networking platforms and services
  - Adaptive Network vision — programmable infrastructure
  - Blue Planet (intelligent automation)
  - WaveLogic (optical technology)
  - Open source contributions
  - Packet + optical convergence

  "I'm excited about Ciena's focus on network automation and the
  opportunity to apply DevOps practices to networking infrastructure.
  The combination of hardware and software development is unique
  and aligns with my interest in embedded systems and CI/CD."
```

### Strengths & Weaknesses

```
Strengths (pick 2-3, back with examples):
  - Automation mindset — "I automated X, saving Y hours"
  - Problem solving — "Debugged a production issue where..."
  - Learning quickly — "Picked up Kubernetes in N weeks"
  - Cross-team collaboration — "Worked with dev + QA + ops"

Weakness (genuine but manageable):
  - "I sometimes spend too much time optimizing before shipping.
    I've learned to set time limits and ship iteratively."
  - "I tend to take on too much. I've started using task boards
    to prioritize and communicate capacity."
  (Show self-awareness + improvement action)
```

---

## 3. Scenario-Based DevOps Questions

### Production Outage / Incident Response

```
Q: "Describe a time you dealt with a production incident"

STAR Example:
  S: Our production K8s cluster had multiple pods crash-looping
     during peak traffic hours, affecting user-facing services.

  T: As the on-call DevOps engineer, I needed to restore service
     ASAP while identifying root cause.

  A: 1. Checked pod status (kubectl get pods) — saw OOMKilled
     2. Checked resource limits — memory limit was 256Mi
     3. Reviewed Grafana dashboards — memory usage spiked after
        latest deployment
     4. Rolled back to previous version (kubectl rollout undo)
     5. Service restored in 12 minutes
     6. Root cause: new dependency had a memory leak
     7. Worked with dev team to fix the leak
     8. Implemented memory alerts in Prometheus
     9. Updated resource limits with proper headroom
     10. Wrote incident postmortem and shared with team

  R: Reduced MTTR from 45min to 12min. The memory alert we added
     caught 3 similar issues before they became incidents.
     Postmortem led to mandatory load testing before deployments.
```

### CI/CD Pipeline Design

```
Q: "How would you design a CI/CD pipeline for a microservices app?"

Answer Structure:
  ┌─── Pipeline Flow ──────────────────────────────────────┐
  │                                                         │
  │  Developer pushes code                                  │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Pre-commit     │ Linting, formatting, secrets scan  │
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Build          │ Compile, Docker build               │
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Test           │ Unit tests, integration tests       │
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Security       │ SAST (SonarQube), image scan (Trivy)│
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Publish        │ Push image to registry              │
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Deploy Staging │ Helm deploy to staging K8s          │
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Smoke Tests    │ API health checks, E2E tests        │
  │  └─┬──────────────┘                                   │
  │    │                                                    │
  │  ┌─▼──────────────┐                                   │
  │  │ Deploy Prod    │ Canary/Blue-Green + approval gate   │
  │  └────────────────┘                                   │
  └─────────────────────────────────────────────────────────┘

Key points to mention:
  - Branch strategy (trunk-based or GitFlow)
  - Environment promotion (dev → staging → prod)
  - Rollback strategy (automated on failure)
  - Infrastructure as Code (Terraform for infra)
  - Observability (monitor deployments with Prometheus/Grafana)
  - Security at every stage (shift-left)
```

### Infrastructure Migration

```
Q: "How would you migrate an on-prem app to cloud/containers?"

Answer Framework:
  Phase 1: Assess
    - Inventory existing services, dependencies, data flows
    - Identify what can be containerized vs. needs VM
    - Risk assessment and compliance requirements

  Phase 2: Plan
    - Choose migration strategy per service:
      Lift & Shift    → minimal changes, VM to cloud VM
      Re-platform     → containerize, use managed services
      Re-architect    → redesign as microservices
    - Design target architecture (VNet, subnets, security)
    - Plan data migration (downtime window? sync replication?)

  Phase 3: Build Foundation
    - IaC for cloud infrastructure (Terraform)
    - CI/CD pipelines for deployment
    - Monitoring and logging stack
    - Security controls (NSGs, identity, secrets management)

  Phase 4: Migrate (iterative)
    - Start with non-critical services
    - Run parallel (old + new) with traffic splitting
    - Validate, then cut over
    - Repeat for critical services

  Phase 5: Optimize
    - Right-size resources
    - Implement auto-scaling
    - Cost optimization
    - Performance tuning
```

### Troubleshooting Scenarios

```
Q: "A deployment succeeded but the app isn't working. What do you do?"

Systematic approach:
  1. Verify deployment status
     kubectl get pods -n myapp          # Are pods running?
     kubectl describe pod <pod-name>    # Events, errors
     kubectl logs <pod-name>            # Application logs
     kubectl logs <pod-name> --previous # Previous container logs

  2. Check application health
     kubectl exec -it <pod> -- curl localhost:8080/health
     kubectl port-forward svc/myapp 8080:80  # Test locally

  3. Check configuration
     kubectl get configmap,secret -n myapp
     kubectl describe deployment myapp  # Image version, env vars
     # Is the right image tag deployed?
     # Are environment variables correct?
     # Are secrets mounted?

  4. Check networking
     kubectl get svc,ingress -n myapp   # Service exists?
     kubectl get endpoints myapp        # Does service have endpoints?
     # No endpoints = label selector doesn't match pods
     kubectl describe svc myapp

  5. Check resources
     kubectl top pods -n myapp          # CPU/memory usage
     kubectl describe node              # Node resource pressure?

  6. Check recent changes
     kubectl rollout history deployment myapp  # What changed?
     git log --oneline -5               # Recent commits

  7. If needed — rollback
     kubectl rollout undo deployment myapp
```

---

## 4. DevOps Philosophy Questions

### What is DevOps to you?

```
Key points:
  - Culture + practices + tools that enable rapid, reliable delivery
  - Breaking silos between Dev and Ops
  - Automation everywhere (build, test, deploy, monitor)
  - Feedback loops (monitoring → incident → fix → prevent)
  - Continuous improvement (blameless postmortems, DORA metrics)

  "DevOps is about enabling teams to deliver software faster and
  more reliably by automating everything from build to deployment
  to monitoring, while fostering collaboration between development
  and operations teams."
```

### How do you handle a disagreement with a team member?

```
Framework:
  1. Listen — understand their perspective first
  2. Find common ground — "We both want..."
  3. Data-driven — "Let's look at the metrics/evidence"
  4. Propose experiment — "Let's try both and measure"
  5. Escalate if needed — bring in a senior/lead for tiebreaker

  "In my experience, most disagreements come from different
  assumptions. I start by understanding why they think differently,
  then we look at data together. Often the right answer is a
  combination of both approaches."
```

### How do you stay current with technology?

```
Genuine answers:
  - Follow DevOps/SRE blogs (Google SRE, Netflix Tech Blog)
  - Hands-on labs (homelab, personal projects, Kubernetes the Hard Way)
  - Community (local meetups, Reddit r/devops, Hacker News)
  - Certifications (CKA, AWS/Azure certs)
  - Conferences (KubeCon, DevOps Days)
  - Read documentation and RFCs
  - Contribute to open source
```

---

## 5. Questions to Ask the Interviewer

```
Technical:
  - "What does the deployment pipeline look like today?"
  - "What's the biggest DevOps challenge the team faces?"
  - "How do you handle infrastructure for embedded systems?"
  - "What monitoring/observability tools do you use?"
  - "How does the team handle on-call rotations?"

Team & Culture:
  - "How is the team structured? How many DevOps engineers?"
  - "What does a typical day look like?"
  - "How do you handle blameless postmortems?"
  - "What opportunities are there for learning and growth?"

Product-specific (Ciena):
  - "How does CI/CD work for embedded/networking products?"
  - "What's the build system for the networking platform?"
  - "How do you handle firmware updates in the field?"
```

---

## 6. Scenario: Designing from Scratch

```
Q: "Design a monitoring solution for a microservices architecture"

Answer:
  ┌─── Three Pillars ──────────────────────────────────────┐
  │                                                         │
  │  Metrics (Prometheus + Grafana)                        │
  │    - Application: request rate, error rate, latency    │
  │    - Infrastructure: CPU, memory, disk, network        │
  │    - Business: orders/min, signups/hour                │
  │    - Scrape interval: 15-30 seconds                    │
  │    - Retention: 15-30 days local, long-term in Thanos  │
  │                                                         │
  │  Logs (ELK or Loki)                                    │
  │    - Structured JSON logs (timestamp, level, traceID)  │
  │    - Centralized collection (Fluent Bit → Elasticsearch)│
  │    - Retention: 30-90 days                             │
  │    - Correlation via traceID                           │
  │                                                         │
  │  Traces (Jaeger or Tempo)                              │
  │    - Distributed tracing across microservices          │
  │    - OpenTelemetry SDK in applications                 │
  │    - Sample rate: 1-10% in production                  │
  │                                                         │
  │  Alerting:                                              │
  │    - Alert on symptoms, not causes                     │
  │    - Use SLO-based alerts (error budget burn rate)     │
  │    - PagerDuty/OpsGenie for on-call routing            │
  │    - Runbooks linked to every alert                    │
  └─────────────────────────────────────────────────────────┘
```

---

## 7. Key Stories to Prepare

```
Prepare 6-8 stories that cover these themes:

  1. Technical Challenge    — debugging a hard problem
  2. Automation Win         — saved time through automation
  3. Incident Response      — handled a production issue
  4. Conflict Resolution    — disagreed with teammate
  5. Learning Quickly       — picked up new technology fast
  6. Leadership/Initiative  — led a project or improvement
  7. Failure/Mistake        — what went wrong, what you learned
  8. Collaboration          — worked across teams

Each story should have:
  ✅ Specific details (dates, tools, numbers)
  ✅ Your individual contribution
  ✅ Measurable result
  ✅ Lesson learned
  ✅ 2-3 minute delivery time
```
