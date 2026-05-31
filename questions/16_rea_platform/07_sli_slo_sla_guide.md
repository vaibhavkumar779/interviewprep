# SLI / SLO / SLA — Complete Practical Guide

> REA Group JD specifically asks for "deep expertise in SLI/SLO/SLA".
> You know the theory. This guide covers the PRACTICAL side —
> calculations, Prometheus rules, Grafana dashboards, error budgets, and real scenarios.

---

## TABLE OF CONTENTS

1. [Definitions — Precise & Clear](#1-definitions)
2. [The SLI → SLO → SLA Pyramid](#2-pyramid)
3. [Choosing SLIs for Different Services](#3-choosing-slis)
4. [Writing SLOs — Real Examples](#4-writing-slos)
5. [Error Budget — Math & Calculations](#5-error-budget)
6. [Burn Rate Alerts](#6-burn-rate)
7. [Prometheus Recording Rules & Alerts](#7-prometheus)
8. [Grafana Dashboard Design](#8-grafana)
9. [Error Budget Policy — What Happens When Budget Exhausted](#9-policy)
10. [SLAs — Business Contracts](#10-sla)
11. [Toil, Reliability, and Feature Velocity](#11-toil)
12. [Interview Questions & Answers](#12-qa)
13. [Real-World Scenarios](#13-scenarios)

---

## 1. DEFINITIONS <a name="1-definitions"></a>

### SLI (Service Level Indicator)
**A measurable metric that quantifies the reliability/performance of a service.**

Think of it as: "What are we measuring?"

```
SLI = (good events / total events) × 100%
```

Examples:
- **Availability SLI**: (successful HTTP requests / total HTTP requests) × 100%
- **Latency SLI**: (requests faster than 300ms / total requests) × 100%
- **Correctness SLI**: (correct responses / total responses) × 100%

### SLO (Service Level Objective)
**A target value for an SLI over a time window.**

Think of it as: "What's our goal?"

```
SLO: 99.9% of requests should return 2xx in a 30-day window
SLO: 95% of requests should complete in under 300ms in a 30-day window
```

### SLA (Service Level Agreement)
**A contract with customers that defines consequences if SLOs are not met.**

Think of it as: "What happens if we fail?"

```
SLA: "99.9% monthly uptime. If breached, customer gets 10% credit."
```

**Key relationship**: SLIs inform SLOs. SLOs are stricter than SLAs.
- Your SLO should be MORE strict than your SLA
- If SLA = 99.9%, your internal SLO might be 99.95%
- This gives you a buffer before breaching the contract

---

## 2. THE SLI → SLO → SLA PYRAMID <a name="2-pyramid"></a>

```
                    ┌─────────┐
                    │   SLA   │  Business contract with customers
                    │  99.9%  │  "We guarantee this or pay penalties"
                    ├─────────┤
                    │   SLO   │  Internal engineering target
                    │  99.95% │  "We aim for this to protect SLA"
                    ├─────────┤
                    │   SLI   │  Actual measurement from monitoring
                    │  99.97% │  "This is what we're actually achieving"
                    └─────────┘

    SLA < SLO ≤ SLI (in a healthy system)
```

---

## 3. CHOOSING SLIs FOR DIFFERENT SERVICES <a name="3-choosing-slis"></a>

### The Four Golden Signals (Google SRE Book)

| Signal | What It Measures | SLI Type |
|---|---|---|
| **Latency** | How long requests take | Request duration |
| **Traffic** | How much demand (not an SLI itself) | Volume indicator |
| **Errors** | Rate of failed requests | Error ratio |
| **Saturation** | How "full" the service is | Resource utilization |

### SLI Selection by Service Type

#### API / Web Service (e.g., REA Property Search API)
```
Primary SLIs:
  1. Availability: % of requests returning non-5xx status codes
  2. Latency: % of requests completing under threshold (e.g., p99 < 500ms)
  3. Correctness: % of search results matching expected format/schema
```

#### Data Pipeline (e.g., Property listing ingestion)
```
Primary SLIs:
  1. Freshness: % of time data is up-to-date (lag < threshold)
  2. Completeness: % of expected records that arrived
  3. Correctness: % of records that pass validation
```

#### Storage System (e.g., Property image storage)
```
Primary SLIs:
  1. Availability: % of successful read/write operations
  2. Latency: % of operations completing under threshold
  3. Durability: (usually covered by cloud provider's SLA)
```

#### Background Job / Worker
```
Primary SLIs:
  1. Success Rate: % of jobs completing successfully
  2. Freshness: Is the queue being processed fast enough?
  3. Throughput: Jobs processed per minute meets expectation
```

### Where to Measure SLIs

```
                    Client                    ← Best (user experience)
                      │
                    CDN / Edge                ← Good
                      │
                    Load Balancer             ← Good (most common)
                      │
                    Application               ← OK
                      │
                    Database                  ← Too deep
```

**Best practice**: Measure as close to the user as possible.
- Load balancer access logs (Nginx, ALB, CloudFront) → best for availability & latency SLIs
- Application metrics (Prometheus) → best for business logic SLIs
- Synthetic monitoring (uptime checks) → supplements real traffic

---

## 4. WRITING SLOs — REAL EXAMPLES <a name="4-writing-slos"></a>

### REA Property Search API SLOs

```yaml
Service: property-search-api
Owner: Platform Team
SLO Window: 30 days (rolling)

SLOs:
  - name: Availability
    description: The proportion of successful HTTP requests
    SLI: count of HTTP requests with status < 500 / count of all HTTP requests
    target: 99.95%
    window: 30 days rolling

  - name: Latency (p50)
    description: Median response time
    SLI: proportion of HTTP requests faster than 200ms
    target: 95%
    window: 30 days rolling

  - name: Latency (p99)
    description: Tail latency
    SLI: proportion of HTTP requests faster than 1000ms
    target: 99%
    window: 30 days rolling
```

### How to Choose Target Numbers

| Target | Downtime/month | Downtime/year | Typical Use |
|---|---|---|---|
| 99% | 7h 18min | 3.65 days | Internal tools, dev environments |
| 99.5% | 3h 39min | 1.83 days | Non-critical services |
| 99.9% | 43min 50s | 8h 46min | Most production APIs |
| 99.95% | 21min 55s | 4h 23min | Critical services |
| 99.99% | 4min 23s | 52min 36s | Core infrastructure (databases) |
| 99.999% | 26s | 5min 15s | Payment processing, extreme reliability |

**Rule of thumb**: Start with 99.9% for most services. Only go higher if business requires it.

---

## 5. ERROR BUDGET — THE MATH <a name="5-error-budget"></a>

### What is an Error Budget?

```
Error Budget = 100% - SLO target = allowed unreliability
```

If SLO = 99.9% availability over 30 days:
```
Error Budget = 100% - 99.9% = 0.1%
Budget in minutes = 30 days × 24 hours × 60 min × 0.001 = 43.2 minutes
```

**You can be "down" for 43.2 minutes per month and still meet your SLO.**

### Error Budget Calculations

#### Calculation 1: Time-based

```
SLO: 99.9% availability (30-day window)

Total minutes in 30 days = 43,200
Error budget = 43,200 × 0.001 = 43.2 minutes

If you've had 20 minutes of downtime so far this month:
Remaining budget = 43.2 - 20 = 23.2 minutes
Budget consumed = 20 / 43.2 = 46.3%
```

#### Calculation 2: Request-based

```
SLO: 99.9% of requests succeed (30-day window)

Total requests this month: 10,000,000
Error budget = 10,000,000 × 0.001 = 10,000 failed requests allowed

If 3,000 requests have failed so far:
Remaining budget = 10,000 - 3,000 = 7,000 requests
Budget consumed = 3,000 / 10,000 = 30%
```

#### Calculation 3: Multi-SLO Budget

```
Service has 2 SLOs:
  1. Availability: 99.9% (10,000 error requests allowed)
  2. Latency p99 < 1s: 99% (100,000 slow requests allowed)

Each SLO has its own error budget. You track them separately.
If EITHER budget is exhausted, you slow down feature work.
```

### Error Budget as a Decision-Making Tool

```
Error Budget Remaining → Action
───────────────────────────────────
> 50% remaining       → Ship features freely, take calculated risks
25-50% remaining      → Proceed with caution, maybe skip risky deploys
10-25% remaining      → Focus on reliability, delay non-critical changes
< 10% remaining       → FREEZE feature releases, all hands on reliability
0% (exhausted)        → Full reliability focus until budget replenishes
```

---

## 6. BURN RATE ALERTS <a name="6-burn-rate"></a>

### Why Not Just Alert on SLO Breach?

If you only alert when SLO is breached (budget = 0%), it's too late.
**Burn rate alerts** tell you when you're consuming budget faster than expected.

### Burn Rate Formula

```
Burn Rate = (actual error rate) / (maximum allowed error rate)

If SLO = 99.9%, allowed error rate = 0.1%
If current error rate = 0.5%:
  Burn Rate = 0.5% / 0.1% = 5x

A burn rate of 5x means you're using error budget 5x faster than expected.
At this rate, your 30-day budget will be exhausted in 6 days (30/5).
```

### Multi-Window Burn Rate Alerts (Google SRE Best Practice)

| Burn Rate | Long Window | Short Window | Budget Consumed | Alert? |
|---|---|---|---|---|
| 14.4x | 1 hour | 5 min | 2% in 1h | PAGE (wake someone up) |
| 6x | 6 hours | 30 min | 5% in 6h | PAGE |
| 3x | 1 day | 2 hours | 10% in 1d | TICKET |
| 1x | 3 days | 6 hours | 10% in 3d | TICKET |

**Two-window approach**: Require BOTH a long and short window to be burning to avoid flapping.

### Prometheus Alert Rule for Burn Rate

```yaml
# Burn rate = 14.4x (consuming 2% budget per hour)
# Alert if both 1-hour AND 5-minute windows show high burn rate
groups:
- name: slo-burn-rate
  rules:
  - alert: HighBurnRate_PropertyAPI
    expr: |
      (
        sum(rate(http_requests_total{service="property-api",code=~"5.."}[1h]))
        /
        sum(rate(http_requests_total{service="property-api"}[1h]))
      ) > (14.4 * 0.001)
      AND
      (
        sum(rate(http_requests_total{service="property-api",code=~"5.."}[5m]))
        /
        sum(rate(http_requests_total{service="property-api"}[5m]))
      ) > (14.4 * 0.001)
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High error budget burn rate for Property API"
      description: "Burning error budget at 14.4x. Will exhaust 30-day budget in ~2 days."
```

---

## 7. PROMETHEUS RECORDING RULES & ALERTS <a name="7-prometheus"></a>

### SLI Recording Rules

Recording rules pre-compute SLI metrics so dashboards and alerts are fast.

```yaml
# prometheus-rules.yaml
groups:
- name: sli-recording-rules
  interval: 30s
  rules:
    # ===== AVAILABILITY SLI =====
    # Total requests per second (by service)
    - record: sli:http_requests:rate5m
      expr: sum by (service) (rate(http_requests_total[5m]))

    # Error requests per second (5xx only)
    - record: sli:http_errors:rate5m
      expr: sum by (service) (rate(http_requests_total{code=~"5.."}[5m]))

    # Availability ratio (1.0 = 100%)
    - record: sli:availability:ratio_rate5m
      expr: |
        1 - (
          sli:http_errors:rate5m
          /
          sli:http_requests:rate5m
        )

    # ===== LATENCY SLI =====
    # Proportion of requests faster than 300ms
    - record: sli:latency_good:rate5m
      expr: |
        sum by (service) (rate(http_request_duration_seconds_bucket{le="0.3"}[5m]))
        /
        sum by (service) (rate(http_request_duration_seconds_count[5m]))

    # P50, P90, P99 latency
    - record: sli:latency_p50:5m
      expr: histogram_quantile(0.5, sum by (service, le) (rate(http_request_duration_seconds_bucket[5m])))

    - record: sli:latency_p99:5m
      expr: histogram_quantile(0.99, sum by (service, le) (rate(http_request_duration_seconds_bucket[5m])))

    # ===== ROLLING SLO WINDOW (30 days) =====
    # Availability over 30 days
    - record: sli:availability:ratio_30d
      expr: |
        1 - (
          sum by (service) (increase(http_requests_total{code=~"5.."}[30d]))
          /
          sum by (service) (increase(http_requests_total[30d]))
        )

    # Error budget remaining (%)
    - record: sli:error_budget_remaining:ratio
      expr: |
        1 - (
          (1 - sli:availability:ratio_30d)
          /
          (1 - 0.999)
        )
      # When this reaches 0, error budget is exhausted
```

### Alerting Rules

```yaml
groups:
- name: slo-alerts
  rules:
    # Page: High burn rate (14.4x over 1h)
    - alert: SLO_HighBurnRate_Page
      expr: |
        (1 - sli:availability:ratio_rate1h{service="property-api"}) > (14.4 * 0.001)
        AND
        (1 - sli:availability:ratio_rate5m{service="property-api"}) > (14.4 * 0.001)
      for: 2m
      labels:
        severity: critical
        slo: availability
      annotations:
        summary: "🔥 Property API burning error budget at 14.4x"
        runbook: "https://wiki.rea.com/runbooks/property-api-slo"
        budget_remaining: '{{ $value | humanize }}'

    # Ticket: Moderate burn rate (3x over 1d)
    - alert: SLO_ModerateBurnRate_Ticket
      expr: |
        (1 - sli:availability:ratio_rate1d{service="property-api"}) > (3 * 0.001)
        AND
        (1 - sli:availability:ratio_rate2h{service="property-api"}) > (3 * 0.001)
      for: 15m
      labels:
        severity: warning
        slo: availability

    # Warning: Error budget < 20%
    - alert: SLO_ErrorBudgetLow
      expr: sli:error_budget_remaining:ratio{service="property-api"} < 0.20
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Property API error budget below 20%"
        budget_pct: '{{ $value | humanizePercentage }}'
```

---

## 8. GRAFANA DASHBOARD DESIGN <a name="8-grafana"></a>

### SLO Dashboard Layout

```
┌─────────────────────────────────────────────────────┐
│ PROPERTY SEARCH API — SLO Dashboard                 │
├─────────────┬─────────────┬─────────────────────────┤
│ Availability│ Error Budget │ Budget Burn Rate         │
│   99.97%    │   70.0%     │   0.8x (healthy)        │
│   ✅ SLO Met │ ✅ 30.4 min │   ◀ Target: < 1.0x     │
│             │  remaining   │                         │
├─────────────┴─────────────┴─────────────────────────┤
│                                                      │
│ Error Budget Consumption Over Time (30 day)          │
│ ████████████░░░░░░░░░░░░░░░░░░░░░░ 30% consumed     │
│                                                      │
├──────────────────────┬───────────────────────────────┤
│ Availability (5m)    │ Request Rate                  │
│ ───────99.9%─────    │ ▂▃▅▆▇█▇▅▃▂                   │
│ ──────actual───      │ 450 req/s                     │
│                      │                               │
├──────────────────────┼───────────────────────────────┤
│ Latency (p50/p99)    │ Error Rate                    │
│ p50: 45ms            │ ▁▁▁▃▁▁▁▁▁▁                   │
│ p99: 320ms           │ 0.03%                         │
│ SLO: <500ms          │                               │
└──────────────────────┴───────────────────────────────┘
```

### Key Grafana Panels

```
Panel 1: Stat — Current Availability (sli:availability:ratio_30d × 100)
Panel 2: Stat — Error Budget Remaining % (sli:error_budget_remaining:ratio × 100)
Panel 3: Gauge — Burn Rate (current / allowed)
Panel 4: Time Series — Error Budget consumption line (should be below 100%)
Panel 5: Time Series — Availability over time with SLO target line
Panel 6: Time Series — Latency percentiles (p50, p90, p95, p99) with SLO threshold
Panel 7: Time Series — Request rate and error rate
Panel 8: Table — SLO status per endpoint
```

### Grafana PromQL for Dashboard Panels

```promql
# Panel 1: Current 30-day availability
sli:availability:ratio_30d{service="property-api"} * 100

# Panel 2: Error budget remaining
sli:error_budget_remaining:ratio{service="property-api"} * 100

# Panel 3: Current burn rate
(1 - sli:availability:ratio_rate1h{service="property-api"}) / (1 - 0.999)

# Panel 4: Error budget consumption over time
(1 - sli:error_budget_remaining:ratio{service="property-api"}) * 100

# Panel 5: Availability with SLO line
sli:availability:ratio_rate5m{service="property-api"} * 100
# Add threshold line at 99.9

# Panel 6: Latency percentiles
sli:latency_p50:5m{service="property-api"} * 1000  # Convert to ms
sli:latency_p99:5m{service="property-api"} * 1000
# Add threshold line at 500ms

# Panel 7: Request rate
sli:http_requests:rate5m{service="property-api"}
# Error rate:
sli:http_errors:rate5m{service="property-api"} / sli:http_requests:rate5m{service="property-api"} * 100
```

---

## 9. ERROR BUDGET POLICY <a name="9-policy"></a>

### What is an Error Budget Policy?

A document that defines what actions teams take based on error budget status.
This is the BRIDGE between SRE and product/engineering teams.

### Example Error Budget Policy (REA Style)

```markdown
# Error Budget Policy — Property Search API

## Budget Status: Green (>50% remaining)
- Feature development proceeds normally
- Standard deploy frequency (multiple per day)
- Acceptable to take calculated risks (new feature rollouts)

## Budget Status: Yellow (20-50% remaining)
- All deploys require approval from on-call engineer
- New features must have rollback plan tested
- Increase monitoring attention
- Schedule reliability improvement tasks

## Budget Status: Orange (5-20% remaining)
- Feature freeze on non-critical changes
- Only bug fixes and reliability improvements deployed
- Postmortem any incident that consumed >5% of budget
- Daily SLO review meetings

## Budget Status: Red (<5% remaining or exhausted)
- FULL feature freeze
- All engineering effort on reliability
- Rollback recent changes that may have contributed
- Leadership escalation
- Daily incident review
- Remains in effect until budget recovers above 20%

## Reset
- Error budget resets at the start of each 30-day rolling window
- No "banking" — unused budget doesn't carry over
```

---

## 10. SLAs — BUSINESS CONTRACTS <a name="10-sla"></a>

### SLA vs SLO

| | SLO | SLA |
|---|---|---|
| Who defines it | Engineering team | Business / Legal |
| Audience | Internal teams | External customers |
| Consequences of breach | Error budget policy kicks in | Financial penalties, contract clauses |
| Target | Typically stricter than SLA | Based on SLO with buffer |
| Format | Internal doc / dashboard | Legal contract |

### SLA Calculation Example

```
SLA: 99.9% monthly uptime

Customer reports:
- Jan: 99.95% → SLA met ✅
- Feb: 99.85% → SLA breached ❌
- Mar: 99.92% → SLA met ✅

Penalty clause: 10% credit for any month below 99.9%
February credit: 10% of monthly bill

Some SLAs use tiered penalties:
- 99.9% - 99.0% → 10% credit
- 99.0% - 95.0% → 25% credit
- < 95.0% → 50% credit
```

### How to Calculate Uptime

```
Method 1: Time-based
Uptime % = (Total minutes - Downtime minutes) / Total minutes × 100

Month = 43,200 minutes
15 minutes downtime:
Uptime = (43200 - 15) / 43200 × 100 = 99.965% ✅

Method 2: Request-based (preferred)
Uptime % = (Successful requests / Total requests) × 100

Total: 10,000,000 requests
Failed: 5,000 requests
Uptime = (10000000 - 5000) / 10000000 × 100 = 99.95% ✅

Method 3: Probe-based
Uptime % = (Successful health checks / Total health checks) × 100
(Used by external monitoring services like Pingdom, UptimeRobot)
```

---

## 11. TOIL, RELIABILITY & FEATURE VELOCITY <a name="11-toil"></a>

### What is Toil?

**Toil** = manual, repetitive, automatable, tactical work that scales linearly with service growth.

Examples:
- Manually restarting crashed pods
- Manually scaling before events
- Manually running deployment scripts
- Responding to alerts that could be auto-remediated
- Manual certificate rotation

**Google's target**: SREs spend ≤50% time on toil, ≥50% on engineering (automation, tooling, reliability improvements).

### The Reliability-Velocity Balance

```
Error Budget = Freedom to Ship

Budget available   → Ship features, innovate, take risks
Budget exhausted   → Fix reliability, pay down tech debt

This creates a healthy tension:
- Product teams WANT to ship features (consume budget)
- SRE teams WANT reliability (preserve budget)
- Error budget is the OBJECTIVE arbitrator
```

---

## 12. INTERVIEW QUESTIONS & ANSWERS <a name="12-qa"></a>

### Q1: "Explain SLIs, SLOs, and SLAs and how they relate."

**Answer**: "SLIs are the metrics we measure — like request success rate or latency percentile. SLOs are targets we set for those metrics — like '99.9% of requests succeed in a 30-day window.' SLAs are business contracts with customers that promise certain SLO levels with penalties for breaches.

The key relationship: SLI is what we measure, SLO is what we target, SLA is what we promise. We always set SLOs stricter than SLAs — if our SLA promises 99.9%, our SLO might be 99.95% — giving us a buffer before breaching the contract."

### Q2: "How do you choose SLIs for a property search service?"

**Answer**: "I'd start with the four golden signals. For a search API:
1. **Availability SLI**: Percentage of requests returning non-5xx codes — this is the most critical because if search is down, users can't find properties.
2. **Latency SLI**: Percentage of requests completing under 500ms — search results need to feel instant.
3. **Correctness SLI** (if applicable): Are search results accurate? This might be a data quality SLI.

I'd measure these at the load balancer level to capture the true user experience, not just application-level health. I'd also set up synthetic monitoring for baseline checks when traffic is low."

### Q3: "What is error budget and how do you use it?"

**Answer**: "Error budget is the complement of the SLO target — if we target 99.9% availability, our error budget is 0.1%, which is about 43 minutes of downtime per month.

We use it as a decision-making tool:
- When budget is healthy (>50%), we ship features freely
- When it's getting low (<20%), we slow down deployments and focus on reliability
- When exhausted, we do a feature freeze until it recovers

This creates an objective, data-driven way to balance reliability and innovation. Product teams can't push for faster shipping when the service is unreliable, and SRE teams can't block all changes when the service is healthy."

### Q4: "How do you set up burn rate alerting?"

**Answer**: "Traditional threshold alerts on SLOs have problems — they either fire too late or too often. Burn rate alerting solves this.

Burn rate measures how fast you're consuming error budget relative to the normal rate. A burn rate of 1x means you'll exactly exhaust your budget by the end of the window. 5x means you'll exhaust it in 6 days instead of 30.

I use multi-window, multi-burn-rate alerts:
- 14.4x burn rate over 1 hour + 5 minutes → page on-call (critical, will exhaust budget in 2 days)
- 6x burn rate over 6 hours + 30 minutes → page (will exhaust in 5 days)
- 3x burn rate over 1 day + 2 hours → create ticket (gradual degradation)

The two-window requirement prevents flapping from short spikes."

### Q5: "You've exhausted your error budget mid-month. What do you do?"

**Answer**: "First, I follow the error budget policy:
1. **Immediate**: Feature freeze — only critical bug fixes and reliability improvements
2. **Investigation**: Postmortem the incidents that consumed the budget — identify root causes
3. **Remediation**: Prioritize fixes — if it was caused by bad deploys, improve CI/CD safety; if infrastructure, address capacity/redundancy
4. **Communication**: Inform stakeholders — product team knows feature work is paused, leadership knows why
5. **Recovery**: Monitor burn rate to confirm we're trending back. Once budget recovers above 20%, gradually resume feature work with extra caution

The error budget policy should be agreed upon in advance, not improvised during an incident."

---

## 13. REAL-WORLD SCENARIOS <a name="13-scenarios"></a>

### Scenario 1: A deploy causes latency spike

```
Situation: You deploy v2.3.1 of property-api. P99 latency jumps from 300ms to 1200ms.
Your latency SLO is: 99% of requests < 500ms.

Action:
1. Check burn rate → it's at 8x (consuming budget 8× normal rate)
2. This will exhaust budget in ~4 days
3. Decision: Rollback to v2.3.0 immediately
4. After rollback, verify latency recovered
5. Investigate v2.3.1 — likely an unoptimized DB query or missing cache
6. Fix the issue, test performance in staging, re-deploy
```

### Scenario 2: Gradual degradation over weeks

```
Situation: Over 3 weeks, availability slowly dropped from 99.98% to 99.92%.
Error budget consumption is at 73%.

Action:
1. Current state = "Yellow" → implement caution measures
2. Look at what changed in the last 3 weeks — new services? traffic growth? infra changes?
3. Check if it's correlated with traffic increase (capacity planning issue)
4. Look at error patterns — are errors concentrated in specific endpoints? time periods?
5. Address root cause (maybe database connection pool exhaustion during peak hours)
6. Implement fix and monitor recovery
```

### Scenario 3: Setting SLOs for a new service

```
Situation: Team is launching a new "property recommendation" service. What SLOs?

Approach:
1. DON'T guess — deploy with monitoring, no SLO for 2-4 weeks
2. Observe actual performance during "SLO-less" period
3. Check natural error rate and latency distribution
4. Set SLO slightly below observed performance (e.g., if p99 is naturally 200ms, set SLO at 500ms)
5. Discuss with product: how critical is this service? Does it block property viewing?
6. If it's a nice-to-have feature, 99.5% availability might be fine
7. If it's on the critical path, aim for 99.9%
8. Start conservative, tighten later
```
