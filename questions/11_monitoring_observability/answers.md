# Monitoring & Observability - COMPREHENSIVE ANSWERS (All 65 Questions)

---

## Fundamentals

**1. Monitoring vs observability?**
- **Monitoring**: Watching known failure modes. Alerts when metrics cross thresholds. "Is the system healthy?"
- **Observability**: Understanding system behavior from external outputs. Debug unknown unknowns. "Why is it unhealthy?"

**2. Three pillars of observability?**

```
┌───────────────────────────────────────────────────────────┐
│                Three Pillars of Observability             │
├───────────────────┬───────────────────┬───────────────────┤
│    METRICS        │    LOGS           │    TRACES          │
├───────────────────┼───────────────────┼───────────────────┤
│ Numeric values    │ Text events       │ Request flow      │
│ over time         │ with context      │ across services   │
│                   │                   │                   │
│ CPU: 85%          │ "Error: DB conn   │ Frontend 200ms   │
│ Latency: 200ms    │  timeout at       │  └─Auth 50ms     │
│ Errors/sec: 5     │  2024-01-15"      │    └─DB 150ms   │
│                   │                   │                   │
│ Best for:         │ Best for:         │ Best for:         │
│  Alerting         │  Debugging        │  Cross-service    │
│  Dashboards       │  Root cause       │  Latency analysis │
│  Trending         │  Audit trails     │  Dependency map   │
├───────────────────┼───────────────────┼───────────────────┤
│ Prometheus        │ ELK/Loki         │ Jaeger/Tempo      │
│ Datadog           │ Splunk           │ Zipkin            │
└───────────────────┴───────────────────┴───────────────────┘
```

1. **Metrics**: Numerical measurements over time (CPU usage, request rate, error count)
2. **Logs**: Discrete events with context (error messages, request details)
3. **Traces**: Request flow across services (distributed tracing)

**3. Metrics vs logs?**
- **Metrics**: Aggregated numbers, low cardinality, efficient storage, good for alerting. "99th percentile latency is 500ms"
- **Logs**: Individual events, high detail, expensive to store, good for debugging. "Request abc123 failed with error xyz"

**4. Distributed tracing? When needed?**
Tracks a single request as it flows through multiple microservices. Each service adds a "span." Needed when: microservices architecture, debugging cross-service latency, understanding request flow.

**5. Black-box vs white-box monitoring?**
- **Black-box**: External probes testing system as user sees it (HTTP check, ping). "Is it working?"
- **White-box**: Internal metrics from the system itself (CPU, memory, custom metrics). "How is it working?"

**6. SLIs, SLOs, SLAs?**
- **SLI (Service Level Indicator)**: Metric measuring service quality. e.g., "99.5% of requests complete in <200ms"
- **SLO (Service Level Objective)**: Target for SLI. e.g., "99.9% availability per month"
- **SLA (Service Level Agreement)**: Contract with consequences. e.g., "99.9% uptime or customer gets credit"

Example for web API:
- SLI: Percentage of requests returning 2xx within 200ms
- SLO: 99.9% of requests meet SLI over 30-day window
- SLA: 99.5% uptime guaranteed, 10% credit if breached

**7. Error budget?**
`Error budget = 1 - SLO`. If SLO is 99.9%, error budget is 0.1% (43.8 minutes/month).
If error budget is consumed: freeze new features, focus on reliability.
If error budget remaining: safe to deploy risky changes.

**8. RED method?**
For request-driven services:
- **R**ate: Requests per second
- **E**rrors: Failed requests per second
- **D**uration: Request latency distribution

**9. USE method?**
For infrastructure resources:
- **U**tilization: % of resource busy (CPU 80%)
- **S**aturation: Queue length / pending work
- **E**rrors: Error count for the resource

**10. Four golden signals?**
Google SRE's key metrics:
1. **Latency**: Time to serve requests (differentiate success vs error latency)
2. **Traffic**: Demand on system (requests/sec)
3. **Errors**: Rate of failed requests
4. **Saturation**: How "full" the service is (CPU, memory, queue depth)

---

## Prometheus

**11. Prometheus? Pull vs push?**
Open-source metrics monitoring system. Uses **pull model**: Prometheus scrapes HTTP endpoints (/metrics) periodically.
- **Pull**: Prometheus fetches metrics from targets. Simpler, more reliable.
- **Push**: Applications push to Pushgateway (for short-lived jobs).

**12. Prometheus architecture?**

```
Prometheus Architecture:
┌────────────────────────────────────────────────────────┐
│                  Prometheus Server                      │
│  ┌────────────┐  ┌───────────┐  ┌────────────────┐  │
│  │ Retrieval   │  │ TSDB       │  │ HTTP Server     │  │
│  │ (scraping)  │─▶│ (storage)  │─▶│ (PromQL API)    │  │
│  └────────────┘  └───────────┘  └────────────────┘  │
│       │ scrape                          │                │
└───────┼───────────────────────────────┼────────────────┘
        │                               │
  ┌─────┴────────┐                  ┌──┴───────────┐
  │ Targets        │                  │ Grafana        │
  │ ┌───────────┐ │                  │ (dashboards)   │
  │ │node_export│ │                  └───────────────┘
  │ │/metrics   │ │
  │ └───────────┘ │   ┌───────────────┐
  │ ┌───────────┐ │   │ Alertmanager  │──▶ PagerDuty/Slack/Email
  │ │app /metrcs│ │   │ (routing,     │
  │ └───────────┘ │   │  grouping,    │
  │ ┌───────────┐ │   │  silencing)   │
  │ │Pushgateway│─┘   └───────────────┘
  │ │(push jobs)│
  └─┴───────────┘
```

- **Prometheus Server**: Scrapes, stores, queries metrics
- **Exporters**: Expose metrics from third-party systems (node_exporter, postgres_exporter)
- **Alertmanager**: Handles alerts (routing, grouping, silencing)
- **Pushgateway**: For short-lived jobs to push metrics
- **Grafana**: Visualization (separate tool)

**13. Exporter? Name 5.**
Translates metrics from a system into Prometheus format:
1. **node_exporter**: Linux host metrics (CPU, memory, disk)
2. **blackbox_exporter**: Probes (HTTP, TCP, ICMP)
3. **postgres_exporter**: PostgreSQL metrics
4. **redis_exporter**: Redis metrics
5. **kube-state-metrics**: K8s object metrics (pods, deployments)
6. **cAdvisor**: Container resource metrics

**14. PromQL? 5 examples.**
```promql
# 1. Current CPU usage
node_cpu_seconds_total{mode="idle"}

# 2. HTTP request rate (per second over 5 min)
rate(http_requests_total[5m])

# 3. Error rate percentage
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100

# 4. Memory usage percentage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# 5. Top 5 pods by CPU
topk(5, rate(container_cpu_usage_seconds_total[5m]))
```

**15. Request rate per second?**
```promql
rate(http_requests_total[5m])
# rate() calculates per-second average over 5-minute window
# For specific endpoint:
rate(http_requests_total{handler="/api/users"}[5m])
```

**16. 95th percentile latency?**
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
# Requires histogram metric type
```

**17. Error rate percentage?**
```promql
# Method 1: status code based
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# Method 2: separate error counter
rate(http_errors_total[5m]) / rate(http_requests_total[5m]) * 100
```

**18. Metric types?**
- **Counter**: Only goes up (total requests, total errors). Use `rate()` to get per-second.
- **Gauge**: Goes up and down (temperature, current connections, memory usage).
- **Histogram**: Samples observations in buckets (request duration distribution). Enables quantiles.
- **Summary**: Like histogram but calculates quantiles on client side.

**19. Counter vs Gauge vs Histogram?**
- Use **Counter** for: total requests, total errors, total bytes processed
- Use **Gauge** for: current temperature, queue size, active connections, memory usage
- Use **Histogram** for: request latency, response size (when you need percentiles)

**20. Recording rule?**
Pre-computed query stored as new metric. Speeds up dashboards and alerts that use expensive queries:
```yaml
groups:
- name: example
  rules:
  - record: job:http_requests:rate5m
    expr: sum(rate(http_requests_total[5m])) by (job)
```

**21. Pushgateway?**
For short-lived jobs (cron, batch) that may not live long enough to be scraped. Job pushes metrics to gateway, Prometheus scrapes gateway.
```bash
echo "batch_job_duration_seconds 42" | curl --data-binary @- http://pushgateway:9091/metrics/job/batch
```

**22. Service discovery in K8s?**
Prometheus auto-discovers targets via K8s API:
```yaml
scrape_configs:
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: true
```
Pods with annotation `prometheus.io/scrape: "true"` are automatically scraped.

**23. Instrument Python/Go app?**
```python
# Python
from prometheus_client import Counter, Histogram, start_http_server

REQUEST_COUNT = Counter('http_requests_total', 'Total requests', ['method', 'endpoint'])
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'Request latency')

@REQUEST_DURATION.time()
def handle_request():
    REQUEST_COUNT.labels(method='GET', endpoint='/api').inc()
    # process request

start_http_server(8000)  # /metrics endpoint
```

**24. Retention period?**
Default: 15 days. Configure with `--storage.tsdb.retention.time=30d` or `--storage.tsdb.retention.size=50GB`.

**25. Limitations for long-term storage?**
Single-node (no native clustering), limited retention, no built-in replication. Solutions: Thanos, Cortex, Mimir, VictoriaMetrics for long-term storage + global querying.

---

## Grafana

**26. Grafana? Integration with Prometheus?**
Open-source visualization platform. Add Prometheus as data source → create dashboards with PromQL queries.

**27. Panel/visualization types?**
Time series (line/area), gauge, stat, bar chart, table, heatmap, pie chart, logs panel, node graph, candlestick.

**28. Create a dashboard?**
1. New Dashboard → Add Panel
2. Select data source (Prometheus)
3. Write PromQL query
4. Choose visualization type
5. Set thresholds, colors, legends
6. Save dashboard

**29. Grafana data source? Name 5.**
1. Prometheus
2. Elasticsearch
3. Loki
4. PostgreSQL/MySQL
5. CloudWatch
6. Azure Monitor
7. InfluxDB

**30. Set up alerts in Grafana?**
Dashboard → Panel → Alert tab → Create alert rule:
- Define query and threshold
- Set evaluation interval
- Configure notification channel (Slack, PagerDuty, email)

**31. Templatize dashboards with variables?**
```
Variables: Settings → Variables
$namespace = label_values(kube_pod_info, namespace)
$pod = label_values(kube_pod_info{namespace="$namespace"}, pod)

Query: rate(container_cpu_usage_seconds_total{namespace="$namespace", pod="$pod"}[5m])
```
Dropdown filters at top of dashboard.

**32. Dashboard provisioning? Dashboards as code?**
```yaml
# /etc/grafana/provisioning/dashboards/default.yaml
apiVersion: 1
providers:
- name: 'default'
  orgId: 1
  folder: ''
  type: file
  options:
    path: /var/lib/grafana/dashboards
```
Store JSON dashboard files in Git. Auto-loaded on Grafana start.

**33. Grafana Loki? vs ELK?**
- **Loki**: Log aggregation by Grafana Labs. Indexes only labels (not full text). Cheaper storage. Pairs with Grafana.
- **ELK**: Full-text indexing. More powerful search. More expensive. Better for complex queries.
- Loki is "like Prometheus, but for logs."

---

## Alerting

**34. Alertmanager?**
Handles alerts from Prometheus: deduplication, grouping, routing, silencing, inhibition.

```
Alerting Flow:

  Prometheus                Alertmanager              Receivers
  ┌─────────────┐        ┌───────────────┐     ┌─────────────┐
  │ Alert rules  │──────▶│ Deduplicate   │     │ PagerDuty   │
  │             │        │       │       │     │ (critical)  │
  │ expr: ...   │        │   Group by    │     └─────────────┘
  │ for: 5m     │        │   labels      │     ┌─────────────┐
  │             │        │       │       │───▶│ Slack       │
  └─────────────┘        │   Route by    │     │ (warning)   │
                         │   severity    │     └─────────────┘
                         │       │       │     ┌─────────────┐
                         │   Silence?    │───▶│ Email       │
                         │   Inhibit?    │     │ (info)      │
                         └───────────────┘     └─────────────┘
```

Sends notifications to: Slack, PagerDuty, email, webhooks.

**35. Alerting rule: CPU > 80% for 5 min?**
```yaml
groups:
- name: node-alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU on {{ $labels.instance }}"
      description: "CPU usage is {{ $value }}%"
```

**36. Alert routing, grouping, silencing, inhibition?**
- **Routing**: Direct alerts to different receivers based on labels (critical→PagerDuty, warning→Slack)
- **Grouping**: Batch similar alerts into one notification (100 pod alerts → 1 grouped alert)
- **Silencing**: Temporarily suppress alerts (during maintenance)
- **Inhibition**: Suppress alerts when related alert is firing (suppress pod alerts when node is down)

**37. Send alerts to Slack, PagerDuty?**
```yaml
# alertmanager.yml
route:
  receiver: 'slack-notifications'
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty'
receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    api_url: 'https://hooks.slack.com/services/xxx'
- name: 'pagerduty'
  pagerduty_configs:
  - service_key: 'xxx'
```

**38. Alert fatigue? Prevent?**
Too many alerts → team ignores them → miss critical issues.
Prevention:
1. Only alert on actionable issues
2. Proper thresholds (not too sensitive)
3. Group related alerts
4. Use warning vs critical severity
5. Regular alert review/cleanup
6. Dashboards for low-priority issues (don't alert)

**39. Warning vs critical alerts?**
- **Warning**: Needs attention soon but not immediately. Slack notification. Can wait until business hours.
- **Critical**: Immediate action required. User-facing impact. PagerDuty/phone call. 24/7 response.

**40. Write good alerts?**
1. **Actionable**: Alert only if human action needed
2. **Relevant**: Alert fires for real problems
3. **Timely**: Not too late, not too noisy
4. **Documented**: Runbook link in annotation
5. **Meaningful summary**: "API latency >500ms for 5 min" not "Check thing"

---

## Logging

**41. Centralized logging?**
Collect logs from all services/servers into one searchable system. Needed because: containers are ephemeral, microservices generate logs everywhere, debugging requires correlation.

**42. ELK stack?**

```
ELK Stack Architecture:

  App Servers / Containers / K8s Pods
  ┌────────┐ ┌────────┐ ┌────────┐
  │ App A  │ │ App B  │ │ App C  │   ← stdout/stderr
  └────┬───┘ └────┬───┘ └────┬───┘
       │            │            │
       └────────────┼────────────┘
                    ▼
  ┌───────────────────────────────────┐
  │  Logstash / Fluentd / Fluent Bit    │   ← Collect, parse,
  │  (log collector + processor)         │     transform, enrich
  └─────────────────┬─────────────────┘
                    ▼
  ┌───────────────────────────────────┐
  │  Elasticsearch                       │   ← Store, index,
  │  (search + analytics engine)          │     full-text search
  └─────────────────┬─────────────────┘
                    ▼
  ┌───────────────────────────────────┐
  │  Kibana                              │   ← Visualize, search,
  │  (dashboard + visualization)          │     dashboards, alerts
  └───────────────────────────────────┘

K8s variant: Fluent Bit DaemonSet (one per node) → Elasticsearch → Kibana
```

- **Elasticsearch**: Search and analytics engine (stores logs)
- **Logstash**: Log collection and processing pipeline
- **Kibana**: Visualization and dashboard UI

**43. EFK stack?**
- **Elasticsearch**: Storage
- **Fluentd**: Log collector (replaces Logstash, lighter, K8s-native)
- **Kibana**: Visualization

**44. Loki vs Elasticsearch?**
| Loki | Elasticsearch |
|---|---|
| Index labels only | Full-text indexing |
| Cheaper storage | More expensive |
| Simpler to operate | Complex cluster management |
| LogQL query language | Lucene/KQL |
| Pairs with Grafana | Pairs with Kibana |
| Better for K8s/cloud | Better for complex search |

**45. Structured logging?**
JSON format instead of plain text:
```json
{"timestamp": "2024-01-15T10:30:00Z", "level": "ERROR", "service": "api", "method": "POST", "path": "/users", "status": 500, "error": "db connection timeout", "request_id": "abc123"}
```
Benefits: parseable, searchable, filterable. Much better than: `ERROR 2024-01-15 db connection timeout`.

**46. Log levels?**
| Level | Use |
|---|---|
| DEBUG | Detailed diagnostic info (development only) |
| INFO | Normal operations (request served, job completed) |
| WARN | Something unexpected but not failure |
| ERROR | Failure that needs attention |
| FATAL/CRITICAL | System cannot continue |

**47. Aggregate logs from K8s pods?**
- DaemonSet running Fluent Bit/Fluentd on every node
- Reads from `/var/log/containers/*.log`
- Forwards to Loki/Elasticsearch
- Alternative: sidecar container per pod

**48. Search and filter logs effectively?**
```
# Kibana KQL
status: 500 AND service: "api-gateway" AND @timestamp > now-1h

# Loki LogQL
{namespace="production", app="api"} |= "ERROR" | json | status >= 500
```

**49. Log rotation?**
Prevents logs from filling disk:
- `logrotate` on Linux (daily, weekly, size-based)
- Docker: `--log-opt max-size=10m --log-opt max-file=3`
- Application-level: logging framework rotation
- K8s: kubelet rotates container logs

**50. How much logging?**
Log: errors, warnings, request summaries, security events, state changes.
Don't log: every successful request (metrics instead), sensitive data (passwords, PII), debug in production.
Rule: log what helps you debug at 3 AM.

---

## Tracing

**51. Jaeger? Zipkin?**
Distributed tracing backends:
- **Jaeger**: By Uber. CNCF project. Popular with K8s.
- **Zipkin**: By Twitter. Older. Simpler.
Both: collect traces, visualize request flow, identify bottlenecks.

**52. OpenTelemetry?**
Vendor-neutral observability framework. Unified SDK for metrics, logs, traces. Replaces OpenTracing + OpenCensus. Supports: export to Jaeger, Zipkin, Prometheus, etc.

**53. Span? Trace?**
- **Trace**: Complete journey of a request through the system (unique trace ID)
- **Span**: Single operation within a trace (service call, DB query). Has: name, start time, duration, parent span, tags.

**54. Instrument app for tracing?**
```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider

provider = TracerProvider()
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("process-order") as span:
    span.set_attribute("order.id", order_id)
    result = process(order_id)
```

**55. Kiali?**
Service mesh observability dashboard for Istio. Visualizes: service topology, traffic flow, health, distributed traces. Good for understanding microservice interactions.

---

## Interview-Style

**56. Monitoring setup at your organization?**
"We use Prometheus + Grafana for metrics, Loki for logs, Jaeger for traces. kube-prometheus-stack deployed via Helm on AKS. Custom dashboards per service. Alertmanager routes critical alerts to PagerDuty, warnings to Slack. Node exporter on every node via DaemonSet. Applications instrumented with OpenTelemetry SDK."

**57. Service has high latency — diagnose?**
1. Check Grafana dashboards: which endpoint is slow?
2. Check request rate: is it a traffic spike?
3. Distributed tracing: where is time spent? (DB? External API?)
4. Check resource metrics: CPU/memory saturation?
5. Check database: slow queries? Connection pool exhaustion?
6. Check pod logs for errors
7. Check if it correlates with a deployment

**58. Monitoring for new microservice?**
1. **Instrument**: Add Prometheus metrics (request count, duration, errors)
2. **Health endpoints**: `/health` and `/ready`
3. **Structured logging**: JSON format to stdout
4. **Dashboard**: Create Grafana dashboard (RED metrics)
5. **Alerts**: Error rate > 1%, latency p99 > 500ms, pod restarts
6. **Tracing**: Add OpenTelemetry spans
7. **SLOs**: Define availability and latency objectives

**59. 500 alerts/day, mostly noise — fix?**
1. Audit every alert: Is it actionable? Does anyone respond?
2. Delete alerts no one acts on
3. Increase thresholds (too sensitive)
4. Add `for:` duration to avoid transient spikes
5. Group related alerts
6. Use dashboards instead of alerts for informational metrics
7. Implement alert review process (monthly)

**60. Prometheus alerting rules for production web service?**
```yaml
groups:
- name: web-service
  rules:
  - alert: HighErrorRate
    expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.01
    for: 5m
    labels: { severity: critical }
    annotations: { summary: "Error rate > 1%" }

  - alert: HighLatency
    expr: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m])) > 1
    for: 5m
    labels: { severity: warning }
    annotations: { summary: "p99 latency > 1s" }

  - alert: PodCrashLooping
    expr: increase(kube_pod_container_status_restarts_total[1h]) > 3
    labels: { severity: critical }
    annotations: { summary: "Pod {{ $labels.pod }} restarting" }

  - alert: HighMemoryUsage
    expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
    for: 5m
    labels: { severity: warning }
    annotations: { summary: "Memory > 90% of limit" }
```

**61. K8s cluster dashboard — what metrics?**
- **Cluster**: Node count, CPU/memory utilization, pod count
- **Nodes**: CPU%, memory%, disk%, network I/O per node
- **Pods**: Running/pending/failed, restart count, resource usage vs limits
- **Workloads**: Deployment replicas (desired vs available), HPA status
- **Networking**: Request rate, error rate, latency by service
- **Storage**: PV usage, PVC status

**62. On-call rotation and incident management?**
1. Define rotation schedule (weekly, PagerDuty/OpsGenie)
2. Escalation policy: primary → secondary → manager
3. Runbooks for common issues
4. Incident response process: detect → triage → mitigate → resolve → post-mortem
5. Blameless post-mortems
6. Track MTTD (detect), MTTR (resolve)

**63. Runbook? How to create?**
Document with step-by-step instructions for responding to specific alerts:
```
Alert: HighCPUUsage
1. Check which process: top -bn1 | head -20
2. If application: check logs, recent deployments
3. If expected load: scale up (kubectl scale --replicas=5)
4. If unexpected: investigate request patterns
5. Escalation: @platform-team if not resolved in 30 min
```

**64. Monitor CI/CD pipeline health?**
Metrics: build duration, success/failure rate, deployment frequency, lead time, queue time.
Dashboard: pipeline runs per day, failure trends, flaky tests, mean time to recovery.
Alerts: pipeline failure rate > threshold, build duration regression.

**65. Correlate metrics, logs, traces for single request?**
Use **request ID** (correlation ID) across all three:
1. Generate unique request ID at ingress
2. Pass in HTTP headers (`X-Request-ID`)
3. Include in all logs: `{"request_id": "abc123", ...}`
4. Include in trace spans as attribute
5. In Grafana: click metric spike → link to Loki logs filtered by time → link to Jaeger trace by request ID
- Exemplars: Prometheus metric → link directly to trace

---
---

# PART 4: ADVANCED MONITORING — PromQL, OpenTelemetry, Grafana Stack

---

## PromQL Deep Dive

**66. PromQL basics — query types:**

```promql
# ─── INSTANT VECTORS (single value per time series at a point) ───

# Current CPU usage across all pods
container_cpu_usage_seconds_total

# Filter by label
container_cpu_usage_seconds_total{namespace="production"}
container_cpu_usage_seconds_total{pod=~"api-.*"}          # regex match
container_cpu_usage_seconds_total{pod!="api-canary"}      # not equal
http_requests_total{status=~"5.."}                        # all 5xx errors

# ─── RANGE VECTORS (values over time window) ───

# Rate of HTTP requests over last 5 minutes (MOST COMMON)
rate(http_requests_total[5m])

# Rate of errors by service
rate(http_requests_total{status=~"5.."}[5m])

# Increase (total count increase over window)
increase(http_requests_total[1h])
```

```promql
# ─── AGGREGATION OPERATORS ───

# Total requests per service
sum(rate(http_requests_total[5m])) by (service)

# Average response time per endpoint
avg(http_request_duration_seconds) by (handler)

# 95th percentile latency (histogram)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Top 5 pods by CPU
topk(5, rate(container_cpu_usage_seconds_total[5m]))

# Count of unhealthy targets
count(up == 0)
```

```promql
# ─── REAL-WORLD ALERT QUERIES ───

# Error rate > 5%
sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
sum(rate(http_requests_total[5m]))
  > 0.05

# P99 latency > 1 second
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
) > 1.0

# Pod restart rate
rate(kube_pod_container_status_restarts_total[15m]) > 0

# Memory usage > 80% of limit
container_memory_working_set_bytes
  /
container_spec_memory_limit_bytes
  > 0.8

# Disk will fill in 4 hours (prediction)
predict_linear(node_filesystem_avail_bytes[1h], 4*3600) < 0

# Absent metric (target down)
absent(up{job="api-server"})
```

```yaml
# Prometheus alerting rule
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
spec:
  groups:
    - name: app.rules
      rules:
        - alert: HighErrorRate
          expr: |
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)
            > 0.05
          for: 5m                          # Must be true for 5 min
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.service }}"
            description: "Error rate is {{ $value | humanizePercentage }}"

        - alert: PodCrashLooping
          expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Pod {{ $labels.pod }} is crash-looping"
```

---

## OpenTelemetry (OTel)

**67. What is OpenTelemetry? Why is it the future?**

```
OpenTelemetry — Unified Observability Framework:

  ┌──────────────────────────────────────────────────────────────┐
  │                    YOUR APPLICATION                          │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
  │  │  Metrics  │  │  Logs    │  │  Traces  │                  │
  │  └─────┬────┘  └─────┬────┘  └─────┬────┘                  │
  │        └──────────────┼────────────┘                        │
  │                       ▼                                     │
  │              ┌─────────────────┐                            │
  │              │  OTel SDK       │  (auto + manual instrument)│
  │              └────────┬────────┘                            │
  └───────────────────────┼─────────────────────────────────────┘
                          ▼
                 ┌─────────────────┐
                 │  OTel Collector │  (receive, process, export)
                 │  ┌───────────┐  │
                 │  │ Receivers │  │  ← OTLP, Jaeger, Prometheus
                 │  ├───────────┤  │
                 │  │Processors │  │  ← batch, filter, transform
                 │  ├───────────┤  │
                 │  │ Exporters │  │  ← to backends
                 │  └───────────┘  │
                 └────────┬────────┘
              ┌───────────┼───────────┐
              ▼           ▼           ▼
         Prometheus    Jaeger      Loki
         (metrics)    (traces)    (logs)
```

```yaml
# OTel Collector configuration
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
  prometheus:
    config:
      scrape_configs:
        - job_name: 'k8s-pods'
          kubernetes_sd_configs:
            - role: pod

processors:
  batch:
    timeout: 5s
    send_batch_size: 1000
  memory_limiter:
    limit_mib: 512
  attributes:
    actions:
      - key: environment
        value: production
        action: insert

exporters:
  prometheus:
    endpoint: 0.0.0.0:8889
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
  loki:
    endpoint: http://loki:3100/loki/api/v1/push

service:
  pipelines:
    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp/jaeger]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch, attributes]
      exporters: [loki]
```

**Why OTel over vendor-specific SDKs:**
- **Vendor-neutral**: Switch backends without code changes
- **Unified API**: Same SDK for metrics + traces + logs
- **Auto-instrumentation**: Inject without code changes (Java, Python, Node.js)
- **CNCF project**: Industry standard (graduated project)
- **W3C Trace Context**: Standardized trace propagation

---

## Grafana Stack (Loki, Tempo, Mimir)

**68. Modern observability stack:**

```
Grafana LGTM Stack:

  ┌────────────────────────────────────────────────────────┐
  │  L = Loki    (logs)     — Like Prometheus but for logs │
  │  G = Grafana (dashboards) — Unified visualization      │
  │  T = Tempo   (traces)  — Distributed tracing backend   │
  │  M = Mimir   (metrics) — Horizontally scalable Prom    │
  └────────────────────────────────────────────────────────┘

  Why Loki over ELK?
  ├── Doesn't index log content (only labels) → much cheaper
  ├── Uses same label model as Prometheus
  ├── Native Grafana integration
  └── LogQL query language (similar to PromQL)
```

```
# LogQL examples (Loki query language)
{namespace="production", app="api"} |= "error"           # contains "error"
{app="api"} |~ "status=[45].."                            # regex match
{app="api"} | json | status >= 500                        # parse JSON, filter
{app="api"} | json | line_format "{{.method}} {{.path}}"  # reformat

# Aggregate logs into metrics
sum(rate({app="api"} |= "error" [5m])) by (pod)           # error rate
count_over_time({app="api"} |= "timeout" [1h])            # timeout count
```

---

## ServiceMonitor & PodMonitor (Prometheus Operator)

**69. Auto-discover monitoring targets in Kubernetes:**

```yaml
# ServiceMonitor — tell Prometheus to scrape a Service
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-monitor
  labels:
    release: prometheus    # Must match Prometheus selector
spec:
  selector:
    matchLabels:
      app: api-service
  endpoints:
    - port: metrics         # Named port in Service
      path: /metrics
      interval: 15s
  namespaceSelector:
    matchNames: ["production"]

---
# PodMonitor — scrape pods directly (no Service needed)
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: batch-jobs-monitor
spec:
  selector:
    matchLabels:
      app: batch-processor
  podMetricsEndpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

**Key interview answer:**
> "For modern observability, I use the **Grafana LGTM stack**: Loki for logs, Tempo for traces, Mimir/Prometheus for metrics, all unified in Grafana dashboards. I instrument apps with **OpenTelemetry** for vendor-neutral telemetry and use the **OTel Collector** as a pipeline to route data. For Kubernetes, the **Prometheus Operator** with ServiceMonitors auto-discovers targets. I write PromQL for alerting — rate() for request rates, histogram_quantile() for latencies, predict_linear() for capacity planning."
