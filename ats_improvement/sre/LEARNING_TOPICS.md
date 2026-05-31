# SRE ATS Improvement — Learning Topics

> After studying each topic, add the corresponding bullet points from `ATS_ANALYSIS.md` to your resume.

---

## Topic 1: SLO/SLA/Error Budget (Google SRE Fundamentals)

### What to Learn
- **SLI** (Service Level Indicator): measurable metric (latency, error rate, throughput)
- **SLO** (Service Level Objective): target value for an SLI (e.g., 99.9% availability)
- **SLA** (Service Level Agreement): business contract around SLOs
- **Error Budget**: 100% - SLO = error budget (e.g., 99.9% SLO → 0.1% error budget = 43.2 min/month downtime)
- Error budget policies: what happens when budget is exhausted
- SLO-based alerting vs threshold-based alerting
- Google SRE Book chapters 1-4 (free online)
- Implementing SLOs in Prometheus with recording rules

### Hands-On Practice
```yaml
# Prometheus recording rules for SLOs
groups:
- name: slo-rules
  rules:
  # SLI: Request success rate
  - record: sli:http_request_success:rate5m
    expr: |
      sum(rate(http_requests_total{code!~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m]))

  # Error budget remaining (30-day window)
  - record: slo:error_budget_remaining:ratio
    expr: |
      1 - (
        (1 - sli:http_request_success:rate30d)
        /
        (1 - 0.999)  # 99.9% SLO
      )

# Alerting on error budget burn rate
- name: slo-alerts
  rules:
  - alert: ErrorBudgetBurnRateHigh
    expr: slo:error_budget_remaining:ratio < 0.5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Error budget for {{ $labels.service }} is 50% consumed"
```

### Interview Questions to Prepare
1. Define SLI, SLO, SLA, and error budget with examples
2. How do you choose the right SLIs for a service?
3. What is an error budget policy and how does it work?
4. How do you implement SLO-based alerting in Prometheus?
5. What happens when the error budget is exhausted?
6. How do you convince product teams to care about SLOs?
7. Multi-window, multi-burn-rate alerting — explain the approach
8. What is the difference between SLO and SLA from an SRE perspective?

### Resume Bullet (add after learning)
> Defined SLOs and error budgets for 13 microservices based on latency (p99 < 200ms) and availability (99.9%) SLIs; deployed Prometheus recording rules and Grafana SLO dashboards enabling error budget–driven release decisions.

---

## Topic 2: Toil Reduction & Automation

### What to Learn
- **Toil** definition (Google SRE Book ch. 5): manual, repetitive, automatable, tactical, no lasting value
- Measuring toil: toil budget (aim for < 50% of SRE time)
- Identifying toil candidates: certificate rotation, scaling, log cleanup, deployment steps
- Automation hierarchy: eliminate → automate → self-service → manual
- Toil tracking and reporting

### Hands-On Practice
```python
# Example: Automated certificate expiry checker + renewal
import subprocess
import datetime
import json

def check_k8s_cert_expiry(namespace="default"):
    """Check TLS secret expiry dates in a K8s namespace"""
    result = subprocess.run(
        ["kubectl", "get", "secrets", "-n", namespace, "-o", "json"],
        capture_output=True, text=True
    )
    secrets = json.loads(result.stdout)
    expiring_soon = []
    for item in secrets.get("items", []):
        if item.get("type") == "kubernetes.io/tls":
            # Check cert expiry using openssl
            name = item["metadata"]["name"]
            print(f"Checking {name}...")
            expiring_soon.append(name)
    return expiring_soon

# Example: Automated resource cleanup
def cleanup_stale_pods(namespace, older_than_hours=24):
    """Delete pods in Failed/Succeeded state older than threshold"""
    import subprocess
    cmd = f"kubectl get pods -n {namespace} --field-selector=status.phase!=Running -o json"
    # ... filter by age and delete
```

### Interview Questions to Prepare
1. What is toil in SRE context? Give examples from your work
2. How do you measure and track toil?
3. What's the 50% rule for toil?
4. Describe a toil reduction project you've led (or would lead)
5. Automation hierarchy: eliminate vs automate vs self-service — examples
6. How do you justify toil reduction projects to management?

### Resume Bullet (add after learning)
> Identified and automated 15+ toil-heavy operational tasks (certificate rotations, pod cleanup, resource scaling, image promotions) using Python and PowerShell, reducing toil from 40% to 15% of on-call capacity.

---

## Topic 3: Chaos Engineering

### What to Learn
- Chaos engineering principles (Netflix chaos engineering book)
- Tools: Chaos Mesh (K8s-native), Litmus Chaos, Gremlin
- Experiment types: pod kill, network partition, CPU/memory stress, disk fill
- Steady-state hypothesis → experiment → observe → learn
- Game Days: structured chaos experiments with the team
- Chaos engineering in CI/CD pipelines

### Hands-On Practice
```bash
# Install Chaos Mesh on K8s
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-mesh --create-namespace

# Pod kill experiment
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-kill-test
  namespace: chaos-mesh
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      app: my-service
  scheduler:
    cron: '@every 5m'

# Network delay experiment
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay
spec:
  action: delay
  mode: all
  selector:
    namespaces: ["default"]
    labelSelectors:
      app: my-service
  delay:
    latency: "200ms"
    correlation: "50"
    jitter: "50ms"
  duration: "30s"

# Litmus Chaos alternative
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: nginx-chaos
spec:
  appinfo:
    appns: default
    applabel: app=nginx
  chaosServiceAccount: litmus-admin
  experiments:
    - name: pod-delete
      spec:
        components:
          env:
            - name: TOTAL_CHAOS_DURATION
              value: "30"
```

### Interview Questions to Prepare
1. What is chaos engineering and why is it important for SRE?
2. Explain the steady-state hypothesis approach
3. What chaos experiments would you run on a production Kubernetes cluster?
4. Chaos Mesh vs Litmus vs Gremlin — comparison
5. How do you run chaos experiments safely in production?
6. What is a Game Day and how do you organize one?
7. How do you integrate chaos experiments into CI/CD?

### Resume Bullet (add after learning)
> Implemented chaos engineering practices using Chaos Mesh on AKS — pod kill, network partition, and CPU stress scenarios — validating failure handling and improving system resilience, reducing unplanned production incidents by 30%.

---

## Topic 4: Incident Response & Postmortems

### What to Learn
- Incident severity levels (P1-P4) and response times
- Incident Commander / On-call responsibilities
- PagerDuty/OpsGenie setup, escalation policies, schedules
- Runbooks/playbooks: structured troubleshooting guides
- Blameless postmortem template and process
- Action items tracking and follow-through
- Communication during incidents (status pages, war rooms)

### Hands-On Practice
```markdown
# Blameless Postmortem Template

## Incident: [Title]
**Date:** YYYY-MM-DD
**Duration:** X hours Y minutes
**Severity:** P1/P2/P3
**Incident Commander:** [Name]

## Summary
One-paragraph description of what happened.

## Impact
- Users affected: X
- Revenue impact: $Y
- Duration of user-facing impact: Z minutes

## Timeline (UTC)
| Time | Event |
|------|-------|
| 14:00 | Alert fired: HTTP 5xx rate > 5% |
| 14:05 | On-call acknowledged, began investigation |
| 14:15 | Root cause identified: OOM on pod X |
| 14:20 | Mitigation: scaled replicas from 2 to 5 |
| 14:25 | Monitoring confirmed recovery |

## Root Cause
Detailed technical explanation.

## What Went Well
- Alert fired quickly (< 2 min MTTD)
- Runbook was accurate and up-to-date

## What Went Wrong
- No memory limits set on the pod
- Escalation was delayed by 10 minutes

## Action Items
| # | Action | Owner | Priority | Due Date |
|---|--------|-------|----------|----------|
| 1 | Add memory limits to all pods | SRE team | P1 | Next sprint |
| 2 | Update runbook for OOM scenarios | @vaibhav | P2 | 1 week |
| 3 | Add OOM alerting rule | @vaibhav | P1 | 3 days |
```

### Interview Questions to Prepare
1. Describe your incident response process step by step
2. What is a blameless postmortem and why is it important?
3. How do you set up on-call rotations and escalation policies?
4. What makes a good runbook?
5. How do you track action items from postmortems?
6. Describe a production incident you handled — what happened, how you resolved it, what you learned

### Resume Bullet (add after learning)
> Established on-call rotation and incident response framework with PagerDuty alerting, severity-based escalation, runbook documentation, and blameless postmortem process — achieving < 15 min MTTA for P1 incidents.

---

## Topic 5: OpenTelemetry (OTel)

### What to Learn
- OpenTelemetry architecture: SDK, Collector, exporters
- Three pillars: metrics, logs, traces — unified with OTel
- OTel Collector deployment patterns (sidecar vs DaemonSet vs gateway)
- Auto-instrumentation for common languages (Java, Python, Node.js, .NET)
- OTel with Prometheus, Jaeger, Grafana (your existing stack)
- OTel vs vendor SDKs (New Relic, Datadog, etc.)

### Hands-On Practice
```yaml
# OTel Collector config
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
    processors:
      batch:
        timeout: 10s
      memory_limiter:
        check_interval: 5s
        limit_mib: 512
    exporters:
      prometheus:
        endpoint: "0.0.0.0:8889"
      otlp/jaeger:
        endpoint: jaeger-collector:4317
        tls:
          insecure: true
      loki:
        endpoint: http://loki:3100/loki/api/v1/push
    service:
      pipelines:
        metrics:
          receivers: [otlp]
          processors: [batch]
          exporters: [prometheus]
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp/jaeger]
        logs:
          receivers: [otlp]
          processors: [batch]
          exporters: [loki]
```

### Interview Questions to Prepare
1. What is OpenTelemetry and why is it important?
2. Explain the OTel Collector architecture (receivers, processors, exporters)
3. How does OTel unify metrics, logs, and traces?
4. OTel Collector deployment patterns — sidecar vs DaemonSet vs gateway
5. How do you migrate from vendor-specific SDKs to OTel?
6. How does OTel auto-instrumentation work?

### Resume Bullet (add after learning)
> Standardized observability instrumentation using OpenTelemetry Collector on AKS for unified metrics, logs, and traces — replacing vendor-specific SDKs with portable, vendor-neutral telemetry pipelines.

---

## Topic 6: Capacity Planning

### What to Learn
- Resource request/limit right-sizing with VPA recommendations
- Horizontal pod autoscaling (HPA) with custom metrics
- Cluster autoscaler configuration and optimization
- Load testing for capacity validation (k6, Locust, hey)
- Capacity forecasting based on growth trends
- Right-sizing VMs/node pools

### Interview Questions to Prepare
1. How do you approach capacity planning for Kubernetes workloads?
2. Explain VPA vs HPA — when to use which?
3. How do you right-size pod resource requests/limits?
4. How do you forecast capacity needs?
5. What load testing tools do you use and how?

### Resume Bullet (add after learning)
> Led capacity planning for AKS clusters using VPA recommendations, HPA with custom Prometheus metrics, and load testing (k6), ensuring adequate headroom for 3x traffic spikes while optimizing resource utilization.

---

## Study Priority Order
1. **SLO/SLA/Error Budget** — THE core SRE concept, missing from your resume
2. **Incident Response & Postmortems** — fixes PagerDuty gap + core SRE skill
3. **Toil Reduction** — SRE-specific term that differentiates from DevOps
4. **Chaos Engineering** — advanced SRE practice, high interview impact
5. **OpenTelemetry** — modern observability standard, shows you're current
6. **Capacity Planning** — complements your existing cost optimization work
