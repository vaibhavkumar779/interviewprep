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
