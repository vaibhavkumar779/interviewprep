# Splunk & SPL — Complete Guide from Zero

> REA Group uses Splunk for observability. You know ELK and Grafana, but have zero Splunk experience.
> This guide takes you from zero to interview-ready.

---

## TABLE OF CONTENTS

1. [Splunk Architecture & Concepts](#1-architecture)
2. [SPL Basics — Search Processing Language](#2-spl-basics)
3. [Filtering & Field Extraction](#3-filtering)
4. [Stats & Aggregations](#4-stats)
5. [Timechart & Time Operations](#5-timechart)
6. [Eval — Computed Fields](#6-eval)
7. [Rex — Regex Extraction](#7-rex)
8. [Transaction & Correlation](#8-transaction)
9. [Subsearch & Join](#9-subsearch)
10. [Lookup Tables](#10-lookup)
11. [Alerting in Splunk](#11-alerting)
12. [Dashboards](#12-dashboards)
13. [Kubernetes Log Analysis in Splunk](#13-kubernetes)
14. [Real-World SPL Patterns for Platform Engineers](#14-patterns)
15. [Splunk vs ELK/Grafana — Comparison for Interview](#15-comparison)
16. [Interview Questions & Answers](#16-qa)

---

## 1. SPLUNK ARCHITECTURE & CONCEPTS <a name="1-architecture"></a>

### ELK ↔ Splunk Mapping

| ELK/Grafana | Splunk | Purpose |
|---|---|---|
| Elasticsearch | Indexer | Store & index data |
| Logstash/Fluentd | Forwarder (UF/HF) | Collect & ship data |
| Kibana | Search Head / Splunk Web | UI & visualization |
| KQL (Kibana Query Language) | SPL | Query language |
| Index (ES index) | Index (Splunk index) | Data store |
| Index Pattern | Source type | Data format |
| Dashboard | Dashboard | Visualization |
| Saved Search | Saved Search / Report | Reusable queries |
| Watcher (Alerts) | Alerts | Automated monitoring |

### Splunk Components

```
                  ┌───────────────────┐
                  │   Search Head     │  ← User queries here (like Kibana)
                  │   (Splunk Web)    │
                  └────────┬──────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────┴─────┐ ┌───┴────┐ ┌────┴─────┐
        │ Indexer 1  │ │Indexer2│ │ Indexer 3│  ← Stores data (like Elasticsearch nodes)
        └────────────┘ └────────┘ └──────────┘
              ▲            ▲            ▲
              │            │            │
        ┌─────┴────────────┴────────────┴─────┐
        │         Heavy Forwarder (HF)         │  ← Parses & routes (like Logstash)
        └────────────────┬─────────────────────┘
                         ▲
              ┌──────────┼──────────┐
              │          │          │
        ┌─────┴──┐ ┌────┴───┐ ┌───┴─────┐
        │  UF 1  │ │  UF 2  │ │  UF 3   │  ← Universal Forwarders (lightweight agents)
        │(server)│ │(server)│ │(server) │     on every server/container
        └────────┘ └────────┘ └─────────┘
```

### Key Concepts

| Concept | Description | Example |
|---|---|---|
| **Index** | Where data is stored (like a database) | `main`, `web_logs`, `k8s_logs` |
| **Source** | File/stream data came from | `/var/log/nginx/access.log` |
| **Source type** | Format of the data | `nginx:access`, `json`, `syslog` |
| **Host** | Machine that generated the log | `web-server-01`, `pod-abc123` |
| **Event** | A single log entry | One line of a log file |
| **Field** | Key-value pair extracted from events | `status=200`, `method=GET` |
| **_time** | Timestamp of the event | `2024-01-15 10:30:45` |

---

## 2. SPL BASICS <a name="2-spl-basics"></a>

### Basic Search Structure

```
search_terms | command1 | command2 | command3
```

SPL reads left to right, like Unix pipes:
```bash
# Unix:  cat access.log | grep ERROR | sort | head -20
# SPL:   index=web_logs ERROR | sort _time | head 20
```

### First Searches

```spl
# Search everything in the main index (last 24h by default)
index=main

# Search for a keyword
index=web_logs ERROR

# Search for a specific field value
index=web_logs status=500

# Search with multiple conditions (AND is implicit)
index=web_logs status=500 method=GET

# OR condition
index=web_logs status=500 OR status=503

# NOT condition
index=web_logs NOT status=200

# Wildcards
index=web_logs status=5*
index=web_logs path="/api/*"

# Time range (override default 24h)
index=web_logs earliest=-1h         # Last 1 hour
index=web_logs earliest=-7d         # Last 7 days
index=web_logs earliest=-30m latest=-5m  # Between 30 and 5 minutes ago
```

### Essential Commands

```spl
# fields — select specific fields (like SELECT in SQL)
index=web_logs | fields _time, status, path, response_time

# table — display as a table
index=web_logs | table _time, status, path, response_time

# head / tail — first/last N results
index=web_logs | head 20
index=web_logs | tail 10

# sort — order results
index=web_logs | sort -response_time      # Descending (- prefix)
index=web_logs | sort +_time               # Ascending (+ prefix)
index=web_logs | sort -response_time | head 10  # Top 10 slowest

# dedup — remove duplicates
index=web_logs | dedup path | table path

# rename — rename fields
index=web_logs | rename response_time AS latency_ms

# where — filter with expressions
index=web_logs | where response_time > 1000
index=web_logs | where status >= 500 AND method="POST"
index=web_logs | where like(path, "/api/%")
```

---

## 3. FILTERING & FIELD EXTRACTION <a name="3-filtering"></a>

### Search-Time Field Extraction

Splunk auto-extracts fields from structured data (JSON, key=value).
For unstructured data, you extract fields manually.

```spl
# JSON logs are auto-parsed:
# {"timestamp":"2024-01-15T10:30:45Z","status":200,"path":"/api/properties","latency":45}
# Fields: status, path, latency are automatically available

# Key=value logs are auto-parsed:
# status=200 method=GET path=/api/properties latency=45ms
# Fields: status, method, path, latency

# For Apache/Nginx common log format, use the source type
index=web_logs sourcetype=access_combined
```

### Using search and where

```spl
# search — simple matching (in the search pipeline)
index=web_logs | search status=500

# where — complex expressions (supports functions)
index=web_logs | where response_time > 1000 AND status != 200
index=web_logs | where cidrmatch("10.0.0.0/8", client_ip)
index=web_logs | where isnotnull(error_message)
index=web_logs | where len(path) > 100    # Long URLs
index=web_logs | where match(path, "^/api/v[0-9]+/")  # Regex match
```

---

## 4. STATS & AGGREGATIONS <a name="4-stats"></a>

### stats command (like SQL GROUP BY)

```spl
# Count events
index=web_logs | stats count

# Count by field
index=web_logs | stats count by status
# Output:
# status | count
# 200    | 45000
# 301    | 2000
# 404    | 500
# 500    | 50

# Multiple stats
index=web_logs | stats count, avg(response_time), max(response_time), min(response_time) by path

# Count distinct values
index=web_logs | stats dc(client_ip) AS unique_visitors

# Percentiles
index=web_logs | stats p50(response_time) AS p50, p95(response_time) AS p95, p99(response_time) AS p99

# Percentiles by endpoint
index=web_logs | stats p50(response_time) AS p50, p99(response_time) AS p99 by path | sort -p99

# Sum
index=web_logs | stats sum(bytes) AS total_bytes by path | sort -total_bytes

# List and values (see all unique values)
index=web_logs status=500 | stats values(path) AS failed_paths, count by host

# Latest/earliest event per group
index=web_logs | stats latest(_time) AS last_seen, earliest(_time) AS first_seen by host
```

### top and rare commands (shortcuts)

```spl
# Top 10 most common status codes
index=web_logs | top status

# Top 5 most requested paths
index=web_logs | top limit=5 path

# Top paths with 500 errors
index=web_logs status=500 | top limit=10 path

# Rarest (least common) values
index=web_logs | rare status
index=web_logs | rare limit=10 useragent
```

### eventstats (adds stats without collapsing rows)

```spl
# Add average response time as a new field to every event
index=web_logs | eventstats avg(response_time) AS avg_rt
# Then find events above average:
| where response_time > avg_rt * 2
```

---

## 5. TIMECHART & TIME OPERATIONS <a name="5-timechart"></a>

### timechart (time-series aggregation — the most important viz command)

```spl
# Error count over time (auto-bucketed)
index=web_logs status=500 | timechart count

# Error count per 5-minute buckets
index=web_logs status=500 | timechart span=5m count

# Request count by status over time
index=web_logs | timechart span=1h count by status

# Average latency over time
index=web_logs | timechart span=5m avg(response_time)

# P99 latency over time
index=web_logs | timechart span=5m p99(response_time)

# Error rate (%) over time
index=web_logs | timechart span=5m count(eval(status>=500)) AS errors, count AS total
| eval error_rate = round(errors/total * 100, 2)

# Requests per second over time
index=web_logs | timechart span=1m count | eval rps=count/60
```

### bin command (bucket time for stats)

```spl
# Same as timechart but with stats
index=web_logs | bin _time span=5m | stats count by _time, status
```

---

## 6. EVAL — COMPUTED FIELDS <a name="6-eval"></a>

### eval creates new calculated fields

```spl
# Convert milliseconds to seconds
index=web_logs | eval latency_sec = response_time / 1000

# Categorize response codes
index=web_logs | eval status_category = case(
    status < 300, "Success",
    status < 400, "Redirect",
    status < 500, "Client Error",
    status >= 500, "Server Error"
)
| stats count by status_category

# Calculate error rate
index=web_logs | stats count(eval(status>=500)) AS errors, count AS total
| eval error_rate = round(errors / total * 100, 2)
| eval status = if(error_rate > 1, "CRITICAL", "OK")

# String operations
index=web_logs | eval domain = mvindex(split(host, "."), 0)
index=web_logs | eval path_short = substr(path, 1, 50)

# Conditional logic (if)
index=web_logs | eval is_slow = if(response_time > 1000, "slow", "fast")

# Coalesce (first non-null value)
index=web_logs | eval display_name = coalesce(user_name, client_ip, "anonymous")

# Math
index=web_logs | eval bytes_mb = round(bytes / 1048576, 2)

# Time math
index=web_logs | eval age_hours = round((_time - relative_time(now(), "-0h")) / 3600, 1)

# URL parsing
index=web_logs | eval api_version = mvindex(split(path, "/"), 2)
# /api/v2/properties → "v2"
```

---

## 7. REX — REGEX EXTRACTION <a name="7-rex"></a>

### rex extracts fields from raw text using regex

```spl
# Extract IP address from raw log
index=web_logs | rex "(?<client_ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"

# Extract error message from stack trace
index=app_logs | rex "Exception: (?<error_msg>[^\n]+)"

# Extract request duration from log like "completed in 234ms"
index=app_logs | rex "completed in (?<duration>\d+)ms"

# Extract container and pod from Kubernetes log
index=k8s_logs | rex "pod=(?<pod_name>[a-zA-Z0-9-]+)\s+container=(?<container>[a-zA-Z0-9-]+)"

# Multiple captures
index=web_logs | rex "(?<method>GET|POST|PUT|DELETE)\s+(?<path>/\S+)\s+HTTP"

# Replace mode (mask data)
index=web_logs | rex mode=sed "s/password=\S+/password=REDACTED/g"
```

---

## 8. TRANSACTION & CORRELATION <a name="8-transaction"></a>

### transaction groups events into multi-event transactions

```spl
# Group web requests by session ID
index=web_logs | transaction session_id maxspan=30m maxpause=5m
# Creates: duration, eventcount fields

# Find long user sessions
index=web_logs | transaction session_id maxspan=1h
| where duration > 300
| table session_id, duration, eventcount

# Track a request through microservices
index=app_logs | transaction trace_id maxspan=5m
| table trace_id, duration, eventcount, host
| sort -duration

# Find failed transactions
index=app_logs | transaction request_id maxspan=1m
| search "ERROR" OR status=500
| table request_id, duration, _raw
```

---

## 9. SUBSEARCH & JOIN <a name="9-subsearch"></a>

### Subsearch (query within a query)

```spl
# Find all requests from IPs that had 500 errors
index=web_logs [search index=web_logs status=500 | dedup client_ip | fields client_ip]

# Get the top 5 error-producing paths, then find ALL events for those paths
index=web_logs [search index=web_logs status=500 | top limit=5 path | fields path]

# Find hosts with high error rates, then get their metrics
index=metrics host IN [search index=web_logs status=500 | stats count by host | where count > 100 | fields host]
```

### join (like SQL join — use sparingly, it's slow)

```spl
# Join web logs with user database
index=web_logs | join user_id [search index=user_db | fields user_id, user_name, role]

# Alternative: Use lookup tables instead of join (much faster)
```

---

## 10. LOOKUP TABLES <a name="10-lookup"></a>

```spl
# Lookup table = CSV file uploaded to Splunk
# team_owners.csv:
# service,team,oncall_email
# property-api,platform,platform@rea.com
# search-api,search,search@rea.com

# Use lookup to enrich events
index=web_logs | lookup team_owners.csv service AS service_name OUTPUT team, oncall_email

# Auto-lookup (configured globally) adds fields automatically
# Every event with service_name field gets team and oncall_email added
```

---

## 11. ALERTING IN SPLUNK <a name="11-alerting"></a>

### Alert Types

```
1. Scheduled Alert — runs on a cron schedule
   "Every 5 minutes, check if error rate > 5%"

2. Real-time Alert — continuous monitoring
   "Alert immediately when status=500 AND path=/api/checkout"

3. Rolling Window Alert
   "Alert if more than 100 errors in any 15-minute window"
```

### Alert SPL Examples

```spl
# High error rate alert (>5% in last 5 minutes)
index=web_logs earliest=-5m
| stats count(eval(status>=500)) AS errors, count AS total
| eval error_rate = errors/total * 100
| where error_rate > 5

# Latency spike alert (p99 > 2 seconds)
index=web_logs earliest=-5m
| stats p99(response_time) AS p99_latency
| where p99_latency > 2000

# Host not sending logs (dead host detection)
index=web_logs earliest=-15m
| stats latest(_time) AS last_seen by host
| eval minutes_ago = (now() - last_seen) / 60
| where minutes_ago > 10

# Disk usage alert
index=os_metrics metric_name=disk_usage earliest=-5m
| stats latest(disk_used_pct) AS disk_pct by host
| where disk_pct > 90

# Error spike (compared to baseline)
index=web_logs earliest=-1h
| timechart span=5m count(eval(status>=500)) AS errors
| eventstats avg(errors) AS avg_errors
| where errors > avg_errors * 3
```

---

## 12. DASHBOARDS <a name="12-dashboards"></a>

### Dashboard XML (Simple XML)

```xml
<dashboard>
  <label>Property API Health</label>
  <row>
    <panel>
      <title>Error Rate (Last 24h)</title>
      <chart>
        <search>
          <query>
            index=web_logs service=property-api
            | timechart span=5m count(eval(status>=500)) AS errors, count AS total
            | eval error_rate = round(errors/total * 100, 2)
          </query>
          <earliest>-24h@h</earliest>
          <latest>now</latest>
        </search>
        <option name="charting.chart">line</option>
      </chart>
    </panel>
    <panel>
      <title>Response Time P99</title>
      <chart>
        <search>
          <query>
            index=web_logs service=property-api
            | timechart span=5m p99(response_time) AS p99
          </query>
        </search>
        <option name="charting.chart">line</option>
      </chart>
    </panel>
  </row>
</dashboard>
```

---

## 13. KUBERNETES LOG ANALYSIS IN SPLUNK <a name="13-kubernetes"></a>

### Common K8s Log Sources in Splunk

```
index=k8s_logs     → Application container logs
index=k8s_events   → Kubernetes events (pod scheduling, scaling)
index=k8s_metrics   → Kubernetes metrics (CPU, memory)
```

### K8s SPL Patterns

```spl
# All logs from a specific pod
index=k8s_logs pod_name="property-api-*" namespace="production"

# Errors by namespace
index=k8s_logs level=ERROR | stats count by namespace | sort -count

# OOMKilled pods
index=k8s_events reason="OOMKilled" | stats count by pod_name, namespace | sort -count

# CrashLoopBackOff detection
index=k8s_events reason="BackOff" | stats count by pod_name | where count > 5

# Pod restart count
index=k8s_events reason="Started" | stats count by pod_name, namespace | where count > 3 | sort -count

# Container CPU usage over time
index=k8s_metrics metric_name="container_cpu_usage" namespace="production"
| timechart span=5m avg(value) by pod_name

# Memory usage approaching limit
index=k8s_metrics metric_name="container_memory_working_set_bytes" namespace="production"
| eval usage_mb = value / 1048576
| stats latest(usage_mb) AS memory_mb by pod_name
| where memory_mb > 400
| sort -memory_mb

# Deployment rollout events
index=k8s_events reason="ScalingReplicaSet" | table _time, object, message

# Failed scheduling
index=k8s_events reason="FailedScheduling" | stats count by pod_name, message

# Node issues
index=k8s_events source_component="kubelet" level=ERROR | stats count by host, message | sort -count

# Image pull failures
index=k8s_events reason="Failed" message="*ImagePullBackOff*" OR message="*ErrImagePull*"
| stats count by pod_name, message

# Trace a request across microservices
index=k8s_logs trace_id="abc-123-def" | sort _time | table _time, pod_name, level, message
```

---

## 14. REAL-WORLD SPL PATTERNS FOR PLATFORM ENGINEERS <a name="14-patterns"></a>

### Incident Investigation

```spl
# Step 1: What's the error rate right now?
index=web_logs service=property-api earliest=-1h
| timechart span=1m count(eval(status>=500)) AS errors, count AS total
| eval error_rate = round(errors/total * 100, 2)

# Step 2: Which endpoints are failing?
index=web_logs service=property-api status>=500 earliest=-1h
| stats count by path, status | sort -count | head 20

# Step 3: What errors are these pods returning?
index=web_logs service=property-api status>=500 earliest=-30m
| stats count by error_message | sort -count | head 10

# Step 4: Is it specific pods or all pods?
index=web_logs service=property-api status>=500 earliest=-30m
| stats count by host | sort -count

# Step 5: When did it start? (find the inflection point)
index=web_logs service=property-api earliest=-6h
| timechart span=5m count(eval(status>=500)) AS errors
| where errors > 0

# Step 6: Was there a deploy around that time?
index=deploy_logs service=property-api earliest=-6h
| table _time, version, deployer, status

# Step 7: Correlate with infrastructure metrics
index=k8s_metrics namespace=production pod_name="property-api-*" earliest=-2h
| timechart span=5m avg(cpu_usage) AS cpu, avg(memory_usage) AS memory
```

### Capacity Planning

```spl
# Request volume growth over 30 days
index=web_logs service=property-api earliest=-30d
| timechart span=1d count AS daily_requests
| trendline sma5(daily_requests) AS trend

# Peak traffic hours
index=web_logs service=property-api earliest=-7d
| eval hour = strftime(_time, "%H")
| stats count by hour | sort hour

# Traffic by day of week
index=web_logs service=property-api earliest=-30d
| eval day = strftime(_time, "%A")
| stats count by day | sort -count

# Latency trend (is it getting worse?)
index=web_logs service=property-api earliest=-30d
| timechart span=1d p99(response_time) AS p99_latency
| trendline sma7(p99_latency) AS trend
```

### Deploy Safety

```spl
# Pre vs post-deploy comparison
# Before deploy (baseline):
index=web_logs service=property-api earliest=-2h latest=-1h
| stats count AS total, count(eval(status>=500)) AS errors, avg(response_time) AS avg_rt, p99(response_time) AS p99_rt
| eval period="before"

# After deploy:
| append [search index=web_logs service=property-api earliest=-1h
  | stats count AS total, count(eval(status>=500)) AS errors, avg(response_time) AS avg_rt, p99(response_time) AS p99_rt
  | eval period="after"]

# Compare side-by-side
| table period, total, errors, avg_rt, p99_rt
```

### SLO Monitoring in Splunk

```spl
# Current availability SLI (30-day rolling)
index=web_logs service=property-api earliest=-30d
| stats count(eval(status<500)) AS good, count AS total
| eval availability = round(good/total * 100, 4)
| eval slo_target = 99.9
| eval error_budget_pct = round((1 - ((100 - availability) / (100 - slo_target))) * 100, 2)
| eval status = if(availability >= slo_target, "✅ MEETING SLO", "❌ SLO BREACHED")
| table availability, slo_target, error_budget_pct, status
```

---

## 15. SPLUNK vs ELK/GRAFANA — COMPARISON <a name="15-comparison"></a>

| Feature | Splunk | ELK Stack | Grafana/Loki |
|---|---|---|---|
| Query Language | SPL | KQL/Lucene | LogQL |
| Learning Curve | Moderate | Moderate | Easy |
| Cost | Expensive (per GB ingested) | Free (OSS) or Elastic Cloud | Free (OSS) or Grafana Cloud |
| Scalability | Excellent (enterprise) | Good (requires tuning) | Good (labels-based) |
| Real-time | Yes | Yes | Yes |
| Alerting | Built-in, powerful | Watcher/ElastAlert | Grafana Alerts |
| Dashboard | Built-in | Kibana | Grafana |
| Schema | Schema-on-read (no mapping) | Schema-on-write (mappings) | Labels-based |
| Strength | Enterprise, compliance, SIEM | Flexibility, open source | Metrics+Logs correlation |

### When to mention in interview:
"I've worked extensively with ELK and Grafana/Loki for log analysis. Splunk's SPL is similar in concept — pipe-based query language like Logstash queries. The main difference is Splunk's schema-on-read approach vs Elasticsearch's mapping-based schema. I'm confident I can be productive in Splunk quickly because the core concepts — log aggregation, field extraction, stats, timechart — map directly to what I've used in ELK."

---

## 16. INTERVIEW QUESTIONS & ANSWERS <a name="16-qa"></a>

### Q1: "How would you investigate a spike in 500 errors using Splunk?"

**Answer**: "I'd follow a systematic approach:
1. First, quantify the problem: `index=web_logs status>=500 earliest=-1h | timechart span=1m count` — see when it started and the magnitude
2. Narrow down which endpoints: `| stats count by path, status | sort -count` — is it one API or all?
3. Check if it's specific hosts: `| stats count by host | sort -count` — one pod failing or all?
4. Look at the actual errors: `| stats count by error_message | sort -count` — what's the root cause?
5. Correlate with deploys: search the deploy log index for changes around the spike time
6. Check infrastructure: CPU/memory metrics for the affected pods

This systematic narrowing — from broad impact to specific cause — helps me find the root cause in minutes, not hours."

### Q2: "How would you build an SLO dashboard in Splunk?"

**Answer**: "I'd create panels for:
1. **Single value panel** showing current 30-day availability with color coding (green/yellow/red)
2. **Timechart** of error rate over time with the SLO target line
3. **Gauge** showing error budget remaining percentage
4. **Timechart** of latency percentiles (p50, p95, p99) with threshold lines
5. **Table** breaking down SLI by endpoint for drill-down

The key SPL would use rolling 30-day stats: `stats count(eval(status<500)) AS good, count AS total | eval availability = good/total * 100`

I'd also set up scheduled alerts for burn rate — if we're consuming error budget faster than 3x normal rate, create a ticket."

### Q3: "What's the difference between stats and eventstats?"

**Answer**: "`stats` aggregates events into summary rows — it collapses events, so you lose individual event details. If I do `stats count by status`, I get one row per status code.

`eventstats` adds the aggregated values as new fields to EVERY original event without collapsing. So `eventstats avg(response_time) AS avg_rt` adds the average to every event, and I can then filter with `where response_time > avg_rt * 2` to find outliers.

I use `stats` for dashboards and summaries, `eventstats` when I need to compare individual events against aggregates."

### Q4: "How would you analyze Kubernetes logs in Splunk?"

**Answer**: "For K8s in Splunk, I'd typically have:
- Container stdout/stderr forwarded via Fluent Bit or Splunk Connect for Kubernetes
- Kubernetes events in a separate index
- Metrics from Prometheus or directly from kubelet

Common analyses:
- Pod errors: `index=k8s_logs namespace=production level=ERROR | stats count by pod_name | sort -count`
- OOM events: `index=k8s_events reason=OOMKilled | stats count by pod_name`
- CrashLoopBackOff: `index=k8s_events reason=BackOff | stats count by pod_name | where count > 5`
- Request tracing: `index=k8s_logs trace_id=abc123 | sort _time | table _time, pod_name, message` to follow a request across microservices

I'd also set up alerts for critical K8s events like image pull failures, node not ready, and persistent OOM kills."

### Q5: "SPL vs KQL — what are the key differences?"

**Answer**: "Both are pipe-based query languages. Key differences:
- SPL uses `search` as the implicit base command; KQL starts with the index name
- Stats: SPL `stats count by field` vs KQL `| summarize count() by field`
- Time series: SPL `timechart span=5m count` vs KQL `| summarize count() by bin(TimeGenerated, 5m)`
- Field extraction: SPL `rex` vs KQL `parse`
- SPL has `eval` for computed fields; KQL uses `extend`
- SPL's `transaction` for multi-event grouping has no direct KQL equivalent

The concepts are identical though — filtering, aggregation, time-series analysis, field extraction. Switching between them is mainly syntax."
