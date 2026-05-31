> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Deep-dive learning guide (this file) |
| [complete.md](complete.md) | Complete question bank |
| [answers.md](answers.md) | All answers |

---

# Monitoring & Observability — Deep-Dive Learning Guide

---

## 1. Monitoring vs Observability

```
┌─── Monitoring ──────────────────────────────────────────────┐
│  "Is my system healthy?"                                     │
│  Pre-defined dashboards, alerts on known failure modes      │
│  Answers KNOWN questions                                    │
└──────────────────────────────────────────────────────────────┘

┌─── Observability ───────────────────────────────────────────┐
│  "WHY is my system unhealthy?"                              │
│  Explore, correlate, drill down into unknown problems       │
│  Answers UNKNOWN questions using three pillars:             │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │  Metrics  │  │   Logs   │  │  Traces  │                  │
│  │          │  │          │  │          │                  │
│  │ Numbers  │  │ Events   │  │ Request  │                  │
│  │ over time│  │ (text)   │  │ journey  │                  │
│  │          │  │          │  │ across   │                  │
│  │ CPU, mem,│  │ Error    │  │ services │                  │
│  │ requests,│  │ messages,│  │          │                  │
│  │ latency  │  │ audit    │  │ Spans,   │                  │
│  │          │  │ trail    │  │ trace IDs│                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. The Three Pillars

### Metrics — Numbers Over Time

```
┌─── Types of Metrics ────────────────────────────────────────┐
│                                                              │
│  Counter:    Only goes up (total requests, errors, bytes)   │
│              Rate: requests/sec = delta(counter) / time     │
│                                                              │
│  Gauge:      Goes up and down (CPU%, memory, queue depth)   │
│              Current snapshot value                          │
│                                                              │
│  Histogram:  Distribution (request latency buckets)         │
│              p50=10ms, p95=50ms, p99=200ms                  │
│                                                              │
│  Summary:    Like histogram but pre-calculated quantiles    │
│              Computed on client side                         │
└──────────────────────────────────────────────────────────────┘
```

### Logs — Discrete Events

```
Unstructured:  "2026-05-25 10:30:15 ERROR Failed to connect to database"
                ↓ MUCH BETTER ↓
Structured (JSON):
{
  "timestamp": "2026-05-25T10:30:15Z",
  "level": "ERROR",
  "service": "api",
  "message": "Failed to connect to database",
  "error": "connection refused",
  "host": "db1.internal",
  "port": 5432,
  "trace_id": "abc123",
  "duration_ms": 5000
}

Why structured?
  - Machine-parseable → queryable
  - Correlatable via trace_id
  - Filterable by any field
  - Aggregatable (count errors by service)
```

### Traces — Request Journey

```
Request: GET /api/orders/123

Trace ID: abc-123-def
├── Span: API Gateway (2ms)
│   └── Span: Auth Service (5ms)
│       └── Span: Token Validation (3ms)
├── Span: Order Service (50ms)
│   ├── Span: Database Query (20ms)  ← SLOW!
│   └── Span: Cache Lookup (1ms)
└── Span: Response Serialization (2ms)

Total: 59ms (DB query is the bottleneck)

Each span contains:
  - Operation name
  - Start time + duration
  - Service name
  - Status (OK, ERROR)
  - Tags/attributes
  - Parent span ID
```

---

## 3. Prometheus + Grafana Stack

```
┌─── Prometheus Architecture ────────────────────────────────┐
│                                                             │
│  ┌─────────────┐     PULL (scrape)                         │
│  │ Prometheus  │◄───────────────── /metrics endpoints      │
│  │   Server    │                                            │
│  │             │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────────────┐│
│  │  - Scraper  │  │App 1│ │App 2│ │Node │ │kube-state-  ││
│  │  - TSDB     │  │:9090│ │:9090│ │Exptr│ │metrics      ││
│  │  - PromQL   │  │/met.│ │/met.│ │:9100│ │:8080        ││
│  │  - Alerting │  └─────┘ └─────┘ └─────┘ └─────────────┘│
│  └──────┬──────┘                                            │
│         │                                                   │
│  ┌──────▼──────┐   ┌──────────────┐                        │
│  │ Alertmanager│   │   Grafana    │                        │
│  │             │   │              │                        │
│  │  - Routes   │   │  - Dashboards│                        │
│  │  - Silence  │   │  - PromQL    │                        │
│  │  - Group    │   │  - Alerts    │                        │
│  │  → Slack,   │   │  - Explore   │                        │
│  │    PagerDuty│   │              │                        │
│  └─────────────┘   └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘

Key concept: Prometheus PULLS metrics (scrapes endpoints)
             NOT push-based (apps don't send metrics to Prometheus)
```

### PromQL — Query Language

```promql
# ─── Basic queries ───
http_requests_total                              # Raw counter
rate(http_requests_total[5m])                    # Requests/sec over 5 min
increase(http_requests_total[1h])                # Total increase in 1 hour

# ─── Filtering ───
http_requests_total{method="GET", status="200"}
http_requests_total{status=~"5.."}               # Regex: 5xx errors
http_requests_total{job!="test"}                 # Exclude test

# ─── Aggregation ───
sum(rate(http_requests_total[5m])) by (service)  # Rate per service
avg(container_memory_usage_bytes) by (pod)       # Avg memory per pod
topk(5, rate(http_requests_total[5m]))          # Top 5 busiest

# ─── Percentiles (from histograms) ───
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
# p95 latency

# ─── Alerting rules ───
# High error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
# More than 5% errors

# Pod restarting
increase(kube_pod_container_status_restarts_total[1h]) > 3
# More than 3 restarts in 1 hour
```

---

## 4. ELK/EFK Stack — Log Management

```
┌─── ELK Stack ──────────────────────────────────────────────┐
│                                                             │
│  ┌──────────┐  ┌──────────────┐  ┌───────────────────────┐│
│  │ Beats /  │  │ Logstash     │  │ Elasticsearch         ││
│  │ Filebeat │─►│ (optional)   │─►│                       ││
│  │          │  │              │  │ - Store & index logs   ││
│  │ Collects │  │ - Parse      │  │ - Full-text search    ││
│  │ logs from│  │ - Transform  │  │ - Aggregations        ││
│  │ files,   │  │ - Enrich     │  │ - Distributed cluster ││
│  │ containers│ │ - Filter     │  │                       ││
│  └──────────┘  └──────────────┘  └───────────┬───────────┘│
│                                               │             │
│                                    ┌──────────▼──────────┐ │
│                                    │      Kibana         │ │
│                                    │                     │ │
│                                    │ - Search & explore  │ │
│                                    │ - Dashboards        │ │
│                                    │ - Visualizations    │ │
│                                    │ - Alerts            │ │
│                                    └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

EFK variant:  Fluentd instead of Logstash (lighter, K8s-native)
```

---

## 5. Distributed Tracing — Jaeger/Zipkin

```
┌─── How Distributed Tracing Works ──────────────────────────┐
│                                                             │
│  1. First service generates Trace ID                       │
│  2. Trace ID propagated in HTTP headers to downstream      │
│  3. Each service creates Spans (start, end, metadata)      │
│  4. Spans sent to collector (Jaeger, Zipkin, Tempo)        │
│  5. UI shows full request timeline across services         │
│                                                             │
│  Headers propagated:                                        │
│    traceparent: 00-<trace-id>-<span-id>-01                │
│    (W3C Trace Context standard)                            │
│                                                             │
│  Tools:                                                     │
│    Jaeger     — CNCF, popular with K8s                     │
│    Zipkin     — Twitter origin                             │
│    Tempo      — Grafana Labs (pairs with Grafana)          │
│    AWS X-Ray  — AWS native                                  │
│    App Insights — Azure native                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. OpenTelemetry (OTel) — Unified Standard

```
┌─── OpenTelemetry ──────────────────────────────────────────┐
│                                                             │
│  Single SDK for ALL three pillars:                         │
│    Metrics + Logs + Traces                                  │
│                                                             │
│  ┌─────────┐    ┌──────────────────┐    ┌───────────────┐ │
│  │  App    │    │ OTel Collector   │    │  Backends     │ │
│  │  + SDK  │───►│                  │───►│               │ │
│  │         │    │  - Receive       │    │  Prometheus   │ │
│  │ Metrics │    │  - Process       │    │  Jaeger       │ │
│  │ Traces  │    │  - Export        │    │  Elasticsearch│ │
│  │ Logs    │    │                  │    │  Grafana Cloud│ │
│  └─────────┘    └──────────────────┘    └───────────────┘ │
│                                                             │
│  Vendor-neutral: instrument once, send to any backend      │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Kubernetes Monitoring

```
┌─── What to Monitor in K8s ─────────────────────────────────┐
│                                                             │
│  Cluster Level:                                             │
│    - Node CPU/memory/disk usage                            │
│    - Node count (scaling events)                           │
│    - API server latency                                     │
│    - etcd health                                            │
│                                                             │
│  Pod/Container Level:                                       │
│    - CPU/memory usage vs limits                            │
│    - Restart count (CrashLoopBackOff)                      │
│    - Pod status (Pending, Running, Failed)                  │
│    - OOMKilled events                                       │
│                                                             │
│  Application Level:                                         │
│    - Request rate, error rate, latency (RED method)         │
│    - Business metrics (orders/sec, signups)                │
│    - Queue depth                                            │
│    - Database connections                                   │
│                                                             │
│  Key tools:                                                 │
│    metrics-server  — basic CPU/memory (kubectl top)         │
│    kube-state-metrics — K8s object states                   │
│    node-exporter   — node-level OS metrics                  │
│    Prometheus      — scrapes all above + app metrics        │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Alerting Best Practices

```
┌─── Good Alerts ─────────────────────────────────────────────┐
│                                                              │
│  ✅ Actionable: someone can fix it RIGHT NOW                │
│  ✅ Symptom-based: "error rate > 5%" not "CPU > 80%"        │
│  ✅ Include context: what, where, severity, runbook link    │
│  ✅ Severity levels:                                        │
│       P1 (Critical): page on-call, revenue impact           │
│       P2 (Warning): Slack alert, investigate within hours   │
│       P3 (Info): dashboard, review during business hours    │
│                                                              │
│  ❌ Alert fatigue: too many noisy alerts → people ignore all│
│  ❌ "CPU > 80%": not actionable, might be normal            │
│  ❌ No runbook: alert fires, nobody knows what to do        │
└──────────────────────────────────────────────────────────────┘
```

### Alert Example (Prometheus AlertManager)

```yaml
groups:
  - name: app-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m                    # Must be true for 5 min (avoid flapping)
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.service }}"
          description: "Error rate is {{ $value | humanizePercentage }}"
          runbook: "https://wiki/runbooks/high-error-rate"

      - alert: PodCrashLooping
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.pod }} is crash looping"
```

---

## 9. RED & USE Methods

```
┌─── RED Method (services/APIs) ──────────────────────────────┐
│                                                              │
│  R — Rate:     requests per second                          │
│  E — Errors:   number of failed requests                    │
│  D — Duration: latency distribution (p50, p95, p99)         │
│                                                              │
│  Best for: microservices, APIs, web endpoints               │
└──────────────────────────────────────────────────────────────┘

┌─── USE Method (infrastructure) ─────────────────────────────┐
│                                                              │
│  U — Utilization: % resource busy (CPU 80%)                 │
│  S — Saturation:  work queued (disk I/O queue length)       │
│  E — Errors:      error count (network packet errors)       │
│                                                              │
│  Best for: hardware, VMs, nodes, disks, networks            │
│  Apply to each resource: CPU, memory, disk, network         │
└──────────────────────────────────────────────────────────────┘

┌─── Four Golden Signals (Google SRE) ───────────────────────┐
│                                                              │
│  Latency     — time to serve a request                      │
│  Traffic     — requests per second                          │
│  Errors      — rate of failed requests                      │
│  Saturation  — how "full" is the system                     │
└──────────────────────────────────────────────────────────────┘
```

---

## 10. SLI, SLO, SLA

```
SLI (Service Level Indicator):
  A measured metric: "99.2% of requests completed in < 200ms"

SLO (Service Level Objective):
  Internal target: "99.9% of requests should complete in < 200ms"

SLA (Service Level Agreement):
  External contract: "We guarantee 99.5% uptime or refund"

  SLA < SLO (always have buffer!)
  Example: SLO = 99.9%, SLA = 99.5%

Error Budget:
  If SLO = 99.9% uptime per month:
    30 days × 24 hours × 60 min = 43,200 min
    0.1% error budget = 43.2 min of downtime allowed
    Used up? → freeze deployments until next month
```

---

## 11. Prometheus Exporters

```
Exporter = adapter that exposes metrics in Prometheus format (/metrics)

┌─── Common Exporters ────────────────────────────────────────────┐
│                                                                  │
│  node_exporter        — Linux host metrics (CPU, mem, disk, net)│
│  kube-state-metrics   — K8s object state (pods, deploys, nodes) │
│  cAdvisor             — Container resource usage (built into    │
│                         kubelet)                                 │
│  blackbox_exporter    — Probe endpoints (HTTP, DNS, TCP, ICMP)  │
│  postgres_exporter    — PostgreSQL metrics                      │
│  redis_exporter       — Redis metrics                           │
│  nginx_exporter       — NGINX stub_status metrics               │
│  process-exporter     — Per-process metrics                     │
│  mysqld_exporter      — MySQL metrics                           │
│                                                                  │
│  In K8s: Deploy as DaemonSet (node_exporter)                    │
│          or Deployment (blackbox, postgres)                      │
└──────────────────────────────────────────────────────────────────┘
```

---

## 12. Prometheus Service Discovery in Kubernetes

```yaml
# prometheus.yml — kubernetes_sd_configs
scrape_configs:
  # Discover pods with prometheus annotations
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with annotation: prometheus.io/scrape: "true"
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Use annotation for custom port
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)
      # Use annotation for custom path
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)

  # Auto-discover services
  - job_name: 'kubernetes-services'
    kubernetes_sd_configs:
      - role: service
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

Pod annotation pattern to enable scraping:
```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

---

## 13. Recording Rules & Pushgateway

**Recording Rules** — pre-compute expensive queries:
```yaml
# rules/recording.yml
groups:
  - name: api_rules
    interval: 30s
    rules:
      # Pre-compute request rate (expensive PromQL → cheap lookup)
      - record: job:http_requests_total:rate5m
        expr: rate(http_requests_total[5m])

      # Pre-compute error percentage
      - record: job:http_error_rate:ratio
        expr: |
          rate(http_requests_total{status=~"5.."}[5m])
          /
          rate(http_requests_total[5m])

  # Why: Dashboard loading faster, alert rules referencing
  # pre-computed values instead of raw metrics
```

**Pushgateway** — for short-lived jobs:
```
Normal:  long-running service → Prometheus scrapes /metrics (pull)
Problem: batch job runs 30 sec → exits before Prometheus scrapes
Solution: batch job → pushes metrics to Pushgateway → Prometheus scrapes Pushgateway

  Batch Job ──push──► Pushgateway ◄──scrape── Prometheus
  (exits)              (persists)

  Use for: cron jobs, CI jobs, migration scripts, one-off tasks
  ❌ Do NOT use for long-running services (use pull model)
```

---

## 14. Application Instrumentation

```python
# Python — prometheus_client library
from prometheus_client import Counter, Histogram, start_http_server

# Define metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0]
)

# Instrument endpoint
@app.route('/api/users')
def get_users():
    with REQUEST_LATENCY.labels('GET', '/api/users').time():
        users = db.query_users()
        REQUEST_COUNT.labels('GET', '/api/users', '200').inc()
        return users

# Expose /metrics endpoint
start_http_server(8000)  # Prometheus scrapes this
```

Four types of Prometheus metrics:
```
Counter    — only goes up (total requests, errors)        .inc()
Gauge      — goes up and down (temperature, queue size)   .set(), .inc(), .dec()
Histogram  — distribution in buckets (latency)            .observe()
Summary    — quantiles on client side (p50, p99)          .observe()
```

---

## 15. Long-Term Storage Solutions

```
Problem: Prometheus stores ~15 days locally, needs lots of disk

┌─── Long-Term Storage Options ──────────────────────────────────┐
│                                                                 │
│  Thanos     — sidecar per Prometheus, uploads to object store  │
│               Global query across clusters                      │
│               Downsampling (5m, 1h) for old data               │
│               Most popular open-source option                   │
│                                                                 │
│  Cortex     — horizontally scalable, multi-tenant              │
│  (now Mimir)  CNCF project, used by Grafana Cloud              │
│                                                                 │
│  Mimir      — Grafana fork of Cortex, simpler ops              │
│               Built for massive scale                           │
│                                                                 │
│  VictoriaMetrics — fast, resource-efficient alternative        │
│                    Drop-in Prometheus compatible                │
│                    Good for smaller teams                        │
└─────────────────────────────────────────────────────────────────┘

Architecture (Thanos):
  ┌────────┐    ┌─────────┐    ┌──────────┐
  │Prometh.│───►│ Thanos  │───►│ Object   │
  │        │    │ Sidecar │    │ Storage  │  (S3, GCS, Azure Blob)
  └────────┘    └─────────┘    └──────┬───┘
                                      │
                               ┌──────▼───┐
                               │  Thanos  │  ← query all clusters
                               │  Query   │
                               └──────────┘
```

---

## 16. Grafana Loki vs ELK

```
┌─── ELK Stack ──────────────────────────────────────────────────┐
│  Elasticsearch → Logstash → Kibana                             │
│                                                                 │
│  ✅ Full-text indexing (search any word in logs)               │
│  ✅ Powerful query language (Lucene, KQL)                      │
│  ✅ Rich visualizations in Kibana                              │
│  ❌ Expensive (indexes every field → lots of storage)          │
│  ❌ Heavy to operate (JVM, cluster management)                 │
│  ❌ Needs tuning for scale                                     │
└─────────────────────────────────────────────────────────────────┘

┌─── Grafana Loki ───────────────────────────────────────────────┐
│  "Like Prometheus, but for logs"                               │
│                                                                 │
│  ✅ Only indexes labels (not log content → 10x cheaper)        │
│  ✅ Same label model as Prometheus → easy correlation          │
│  ✅ Lightweight, easy to operate                               │
│  ✅ Native Grafana integration                                 │
│  ❌ No full-text search (grep-like, label-based filtering)     │
│  ❌ Less mature than ELK                                       │
└─────────────────────────────────────────────────────────────────┘

LogQL example:  {app="api", env="prod"} |= "error" | json | status >= 500
PromQL example: rate(http_requests_total{app="api"}[5m])
                   ↑ same labels → correlate logs + metrics!
```

---

## 17. Alertmanager Configuration

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alerts@example.com'

route:
  receiver: 'default-slack'
  group_by: ['alertname', 'namespace']      # Group related alerts
  group_wait: 30s                            # Wait before first notification
  group_interval: 5m                         # Wait between group updates
  repeat_interval: 4h                        # Re-notify every 4h if unresolved
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
    - match:
        severity: warning
      receiver: 'slack-warnings'

receivers:
  - name: 'pagerduty-critical'
    pagerduty_configs:
      - routing_key: '<key>'
  - name: 'slack-warnings'
    slack_configs:
      - channel: '#alerts-warning'
        text: '{{ .CommonAnnotations.summary }}'
  - name: 'default-slack'
    slack_configs:
      - channel: '#alerts'

inhibit_rules:
  # If critical fires, suppress warning for same alert
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'namespace']
```

---

## 18. Grafana Dashboard Provisioning (Dashboards as Code)

```yaml
# grafana/provisioning/dashboards/dashboards.yml
apiVersion: 1
providers:
  - name: 'default'
    folder: 'DevOps'
    type: file
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true

# Store dashboard JSON in Git → mount into Grafana container
# Change dashboard → commit → CI deploys → Grafana picks up
```

---

## 19. Azure Monitor & Application Insights

```
┌─── Azure Monitor Stack ───────────────────────────────────────┐
│                                                                │
│  Azure Monitor         — platform metrics for Azure resources │
│  Log Analytics         — centralized log storage (KQL queries)│
│  Application Insights  — APM for applications                 │
│  Azure Alerts          — metric/log-based alert rules         │
│  Workbooks             — interactive dashboards               │
│                                                                │
│  KQL example:                                                  │
│  requests                                                      │
│  | where timestamp > ago(1h)                                   │
│  | where resultCode >= 500                                     │
│  | summarize count() by bin(timestamp, 5m), cloud_RoleName    │
│  | render timechart                                            │
│                                                                │
│  App Insights auto-collects: requests, dependencies,          │
│  exceptions, traces, page views, performance counters         │
└────────────────────────────────────────────────────────────────┘
```

---

## 20. On-Call Best Practices

```
┌─── On-Call Checklist ──────────────────────────────────────────┐
│                                                                 │
│  1. Runbooks for every alert (what to check, how to fix)       │
│  2. Escalation path: primary → secondary → manager             │
│  3. Alert fatigue mitigation:                                   │
│     - Only alert on actionable conditions                       │
│     - Use severity levels (critical=page, warning=slack)        │
│     - Group related alerts (Alertmanager group_by)              │
│  4. Post-incident review within 48 hours                        │
│  5. Fair rotation schedule (follow the sun for global teams)   │
│  6. Compensatory time off for after-hours pages                │
└─────────────────────────────────────────────────────────────────┘
```
