# SRE — ATS Score Analysis

## Current Score: 93/100

| Criteria | Score | Max | Notes |
|---|---|---|---|
| Format/Parsability | 10 | 10 | Clean DOCX, Calibri, no tables/columns/images |
| Standard Sections | 10 | 10 | All standard sections present |
| Contact Info | 10 | 10 | Complete |
| Keyword Match | 19 | 20 | Missing: SLO, SLA, error budget, postmortem |
| Quantified Impact | 18 | 20 | Strong. Could add: availability %, MTTR |
| Skill-Role Alignment | 18 | 20 | Excellent SRE alignment |
| Consistency | 8 | 10 | PagerDuty in skills but not in experience bullets |

---

## Keywords Missing (High-Impact for ATS)

### Priority 1 — Must Add (SRE-specific vocabulary from Google SRE book)
| Keyword | Why It Matters | Learn? |
|---|---|---|
| SLO (Service Level Objective) | Core SRE concept — in 95% of SRE JDs | ✅ Yes |
| SLA (Service Level Agreement) | Business-facing reliability target | ✅ Yes (SRE book ch. 4) |
| Error Budget | SRE decision framework | ✅ Yes (SRE book ch. 3) |
| Toil Reduction / Toil Automation | SRE mission — in 80% of JDs | ✅ Yes (SRE book ch. 5) |
| Postmortem / Blameless Postmortem | Incident learning process | ✅ Yes |
| MTTR (Mean Time to Recovery) | Companion to MTTD | ⬜ Add metric |

### Priority 2 — Should Add (60%+ of SRE JDs)
| Keyword | Why It Matters | Learn? |
|---|---|---|
| On-call / Incident Response | Core SRE duty | ⬜ Already implied — make explicit |
| Runbook / Playbook | Operational documentation | ✅ Yes |
| Chaos Engineering | Reliability testing (Gremlin, Litmus) | ✅ Yes |
| PagerDuty / OpsGenie | Alerting — currently in skills but unsubstantiated | ⬜ Add experience bullet |
| Capacity Planning | SRE responsibility | ⬜ Already doing — just add the term |
| Istio / Service Mesh | Mentioned Kiali already — add Istio explicitly | ✅ Yes |

### Priority 3 — Nice to Have
| Keyword | Why It Matters | Learn? |
|---|---|---|
| OpenTelemetry (OTel) | Modern observability standard | ✅ Yes |
| Thanos / Cortex | Prometheus at scale | ⬜ Awareness |
| VictoriaMetrics | High-perf metrics | ⬜ Awareness |
| Statuspage | Incident communication | ⬜ Awareness |
| Feature Flags (LaunchDarkly) | Progressive delivery | ⬜ Awareness |

---

## Bullet Point Improvements

### Current → Improved (after learning topics above)

1. **Enhance monitoring bullet** (add SLO/SLA/error budget):
   > Defined SLOs and error budgets for 13 microservices; deployed Prometheus/Grafana with SLI-based dashboards and alerting, reducing MTTD by 60% and MTTR by 45% while maintaining 99.9% availability SLA.

2. **Add toil reduction bullet** (new):
   > Identified and automated 15+ toil-heavy operational tasks (certificate rotations, log cleanup, resource scaling) using Python and PowerShell scripts, reducing toil from 40% to 15% of team capacity.

3. **Add chaos engineering bullet** (new):
   > Implemented chaos engineering practices using Litmus/Chaos Mesh on AKS to validate failure handling — pod kill, network partition, and resource exhaustion scenarios — improving system resilience and reducing production incidents by 30%.

4. **Add incident response bullet** (new):
   > Established on-call rotation and incident response framework with PagerDuty alerting, runbook documentation, and blameless postmortem process, achieving < 15 min response time for P1 incidents.

5. **Add OpenTelemetry bullet** (enhance existing):
   > Standardized observability instrumentation using OpenTelemetry (OTel) across microservices for unified metrics, logs, and traces collection, replacing vendor-specific SDKs and enabling portable observability.

6. **Fix PagerDuty gap** — either add the incident response bullet above (substantiates it) or remove from skills.

---

## Target Score After Improvements: 98/100
