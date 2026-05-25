# Scenarios & Behavioral - COMPREHENSIVE ANSWERS (All 80 Questions)

---

## Incident Response & Troubleshooting

**1. Production is down. Walk through your complete incident response process.**
1. **Detect & Acknowledge**: Alert fires (PagerDuty/Slack). Acknowledge within 5 min. Join incident bridge.
2. **Communicate**: Post in #incidents channel, update status page. Assign Incident Commander.
3. **Assess Severity**: P1 (full outage)? P2 (degraded)? How many users affected?
4. **Mitigate FIRST** (don't debug first!):
   - Rollback last deployment: `kubectl rollout undo deployment/myapp`
   - Scale up if resource issue: `kubectl scale deployment/myapp --replicas=10`
   - Redirect traffic if region-specific
5. **Diagnose Root Cause** (after mitigation):
   - Check recent changes (deployments, config, infra)
   - Check dashboards (Grafana: latency, errors, CPU/memory)
   - Check logs: `kubectl logs -f deployment/myapp --since=30m`
   - Check dependencies (DB, cache, external APIs)
6. **Fix & Verify**: Deploy hotfix, verify metrics return to normal
7. **Post-mortem** (within 48 hours): Blameless timeline, root cause, action items, update runbooks

**2. Deployment went out, users report 500 errors. First 5 minutes?**
- Minute 0-1: Check deployment status: `kubectl rollout status deployment/myapp`
- Minute 1-2: Check logs: `kubectl logs -l app=myapp --tail=100 --since=5m`
- Minute 2-3: If logs show app crash → immediate rollback: `kubectl rollout undo deployment/myapp`
- Minute 3-4: Verify rollback: check error rate dropping in Grafana
- Minute 4-5: Communicate status to team, begin root cause investigation

**3. CI/CD pipeline broken for 3 days. Nobody can deploy.**
1. **Immediate**: Create a hotfix path (manual deployment procedure as workaround)
2. **Diagnose**: Read error logs, check recent pipeline YAML changes, check agent health, check external dependencies (registry, cloud APIs)
3. **Fix**: Pin dependency versions, fix/replace agent, revert breaking config change
4. **Prevent**: Pipeline health monitoring, alert when red >4 hours, test pipeline changes in branch, maintain "break glass" manual deploy procedure

**4. Response times increased 10x. Diagnosis?**
1. Correlate with recent deployments/config changes
2. Check resources: `kubectl top pods` - CPU/memory exhaustion?
3. Check DB: slow queries? connection pool exhaustion?
4. Check external APIs: increased latency? timeouts?
5. Check app: thread contention? memory leak? GC pauses?
6. Use distributed tracing (Jaeger) to find slow span
7. Compare profiles before vs after

**5. Database connections exhausted.**
1. Check pool settings vs max DB connections
2. Check for leaks: `SELECT * FROM pg_stat_activity;` - idle connections?
3. Check long-running queries blocking connections
4. Check if new service consuming connections
5. **Fix**: Increase pool (short-term), fix leak in code (long-term), add PgBouncer (connection pooler)

**6. Container keeps getting OOM killed.**
1. `kubectl describe pod` → check `limits.memory` and OOMKilled events
2. `kubectl top pod` → actual usage vs limits
3. Root cause: memory leak? processing large files in memory? limit too low?
4. Fix: increase limit (if legitimate), fix memory leak (if bug), add memory profiling

**7. Jenkins master down during a release.**
1. Check what stage release was in; check target environment state manually
2. If partial deployment → decide roll forward or roll back
3. Recover Jenkins: restart, check disk space, JVM heap, restore from backup if corrupted
4. Resume release from failed stage (if pipeline is idempotent)
5. Prevent: Jenkins HA, pipeline checkpoints, external deployment orchestration

**8. Deployment succeeded in staging but fails in production.**
| Category | Differences |
|---|---|
| Config | Different env vars, secrets, feature flags |
| Scale | More traffic, data, concurrent users |
| Data | Different DB schema, volume, edge cases |
| Network | Different firewall rules, DNS, proxy |
| Resources | Different CPU/memory limits |
| Dependencies | Different versions of external services |
| Permissions | Different IAM/RBAC |
**Prevent**: Environment parity via IaC, config validation in pipeline, load testing in staging

**9. Network latency between microservices increased suddenly.**
1. Check recent deployments to either service
2. Check node placement: `kubectl get pods -o wide` — different nodes/zones?
3. Check DNS resolution: `kubectl exec -- time nslookup service-b`
4. Check NetworkPolicies — new policy applied?
5. Check CNI plugin health
6. Check node network with ping/traceroute between nodes
7. Check if sidecar proxy (Istio) was added or misconfigured

**10. "It works on my machine."**
1. Compare systematically: OS, runtime versions, env vars, dependency versions (`pip freeze`, `npm list`), config files
2. Ask: "Can you run it in a container?" → eliminates local differences
3. Check: Does it work in CI? That's the source of truth.
4. Long-term: Containerize dev environment (devcontainers, Docker Compose)

---

## Design Scenarios

**11. CI/CD pipeline for 15 microservices.**
```
Per-service pipeline (shared template):
├── Trigger: PR/push to service directory
├── Build (parallel): Lint + Unit Tests, SAST, Docker build + tag
├── Test: Integration tests (docker compose), container scan (Trivy)
├── Deploy Staging: Auto on main merge, smoke tests
├── Manual approval gate
└── Deploy Production: Canary rollout

Shared infrastructure:
- Pipeline templates (DRY), shared base images
- Centralized registry, monitoring, GitOps with ArgoCD
```

**12. Monitoring and alerting strategy.**
```
Metrics (Prometheus+Grafana): Four Golden Signals per service
Logs (Loki/ELK): Structured JSON via Fluent Bit DaemonSet
Traces (Jaeger+OpenTelemetry): Request flow across services
Alerting tiers:
  P1 (page): Error rate >5%, full outage → PagerDuty
  P2 (Slack): High latency, degraded, disk >85%
  P3 (ticket): Warning trends, cert expiry
```

**13. Disaster recovery for CI/CD.**
```
- Source code: Git distributed + mirrors (RPO=0)
- Pipeline config: Stored as code in repos
- Secrets: Vault with replication
- Artifacts: Registry cross-region replication
- Jenkins: Configuration as Code (JCasC), daily backup (RTO=1hr)
- Agents: Ephemeral from images (reproducible)
- State files: Terraform state in Azure Blob with versioning
```

**14. Secrets management for 50 repos.**
```
Central store: HashiCorp Vault / Azure Key Vault
- Per-service policies (least privilege)
- Automated rotation for DB passwords, API keys
- Audit logging for all access
CI/CD: Variable groups linked to Key Vault
K8s: External Secrets Operator
Rules: No secrets in code (GitLeaks pre-commit), short-lived tokens, audit everything
```

**15. Branching strategy for weekly shipping.**
Trunk-Based Development: main always deployable, short-lived feature branches (<2 days), weekly tag/release, feature flags for incomplete features. GitFlow is too heavy for weekly releases.

**16. Log aggregation for 100 microservices on K8s.**
Apps → stdout/stderr → Fluent Bit DaemonSet → Loki/Elasticsearch → Grafana/Kibana. Structured JSON logging, labels (service, namespace, trace-id). Retention: 7d hot, 30d warm, 90d cold storage.

**17. Self-service environment provisioning.**
Developer portal → API triggers Terraform (provision infra) → ArgoCD deploys services → Slack notification with URL. Auto-cleanup after configurable TTL. Cost control with overnight shutdown.

**18. Embedded Linux firmware pipeline (Yocto).**
Code push → Gerrit review → Jenkins triggers `bitbake custom-image` (sstate-cached) → Archive firmware.wic → QEMU smoke tests → Hardware lab integration tests → Tag + sign release → OTA update server.

**19. Automated rollback for K8s.**
Option 1: `kubectl rollout undo` (native). Option 2: Pipeline deploys → waits 5min → checks error rate → auto-rollback if threshold exceeded. Option 3: Argo Rollouts with canary analysis (auto-promote or rollback based on metrics).

**20. Multi-cloud deployment.**
Abstract with Terraform modules per cloud + same K8s manifests everywhere. Helm charts with cloud-specific values. Challenges: networking between clouds, different managed services, different IAM. Use cloud-agnostic services where possible.

**21. Monolith to microservices - DevOps role?**
Set up per-service CI/CD, containerize extracted services, service mesh for communication, distributed tracing (critical), shared config/secrets management, strangler fig pattern for gradual migration.

**22. Implement GitOps?**
Store all K8s manifests in Git → Install ArgoCD → Create ArgoCD Application pointing to repo → ArgoCD auto-syncs cluster to match Git → PRs become deployment mechanism. Benefits: audit trail, easy rollback (git revert), no kubectl access needed.

**23. Dev onboarding - contribute day 1.**
Pre-provision laptop/accounts. Single setup script or devcontainer. Tested README with "Getting Started". First PR: guided small fix with buddy. Self-service environments.

**24. Team across 3 timezones.**
Async-first (documented decisions), fully automated CI/CD (no human gating), self-service tools, overlapping hours for sync, follow-the-sun on-call, shared dashboards and runbooks.

**25. Compliance-friendly CI/CD.**
PR with required reviewers, branch protection, pipeline logs retained 1yr, artifact signing (cosign), SBOM per build, security scan results stored, approval gates with audit log, RBAC separation, environment isolation.

---

## Architecture & Scale

**26. CI/CD for monorepo (50+ services)?**
Path-based triggers (only build changed), dependency graph (rebuild dependents of shared libs), parallel builds, shared cache, tools like Nx/Bazel/Turborepo, template pipelines parameterized per service.

**27. Optimize 60-minute pipeline.**
1. Profile stages. 2. Parallelize (lint/test/SAST simultaneously). 3. Cache aggressively. 4. Split tests across agents. 5. Incremental builds. 6. Self-hosted pre-warmed agents. 7. Smaller images.

**28. Handle environment drift.**
IaC for everything, same pipeline for all envs (only values differ), drift detection (scheduled Terraform plan), immutable infrastructure, config comparison in CI.

**29. Feature flags.**
Toggle features without deployment. Tools: LaunchDarkly, Unleash, Flagsmith. Use for: canary, A/B testing, kill switch, trunk-based dev. Clean up old flags with expiry dates.

**30. Infrastructure changes across teams.**
RFC process → review → approve → implement. Staged rollout (dev → staging → prod). Always have rollback plan. Maintenance windows for major changes. Communicate timeline in shared channel.

---

## Behavioral (STAR Format)

**31. Improved a CI/CD pipeline.**
> S: Azure DevOps pipeline took 40+ minutes. T: Reduce time. A: Docker layer caching (reorder Dockerfile), parallel test stages, self-hosted agents. R: 40→15 min (62% reduction), deployments 3→8+/day.

**32. Production incident.**
> S: Deployment caused API timeouts for 15% of users. T: Mitigate and fix. A: Checked diff, found config reduced connection pool. Rolled back in 5 min. R: 7 min downtime. Post-mortem led to load testing in staging + config validation.

**33. Automated a manual process.**
> S: 2-hour manual release process. T: Automate. A: Pipeline auto-generates release notes from PR titles, creates tag, builds/pushes images, deploys, runs smoke tests. R: 2hr→15min, zero manual errors, weekly releases.

**34. Learned new technology quickly.**
> S: Joined current company with no Azure DevOps experience. T: Become proficient in month 1. A: Docs, PoC pipelines, pair programming, created shared templates. R: Go-to person in 3 months, created 15+ templates.

**35. Disagreed with team member.**
> S: Dev wanted GitFlow, I wanted trunk-based. T: Reach consensus. A: Created comparison doc, proposed 2-week trial. R: Team adopted trunk-based, merge conflicts reduced 70%.

**36. Worked under pressure.**
> S: Critical CVE, patch needed in 24hrs across all envs. T: Patch 12 services. A: Prioritized public-facing, batch pipeline, staged rollout. R: All patched in 18hrs, zero downtime, documented as runbook.

**37. Collaborated across timezones.**
> S: Teams in India, Europe, US on shared CI/CD. T: Coordinate pipeline changes. A: Documented changes before implementation, async Slack, overlapping-hours meetings, self-service docs. R: Zero disruption, positive feedback.

**38. Automation broke production.**
> S: Pipeline change deployed dev config to prod. T: Fix and prevent. A: Immediate rollback. Added environment validation step, "confirm production" gate, scoped variables properly. R: 8 min downtime, 3 safety checks added.

**39. Most complex problem.**
> S: 30% intermittent build failures on Jenkins agents. T: Find root cause. A: Analyzed patterns → failures during parallel builds → /tmp filling up from Docker layers. Added cleanup, increased partition, monitoring. R: Failure rate 30%→<1%.

**40. Prioritize multiple urgent tasks.**
> 1. Impact (affects production?). 2. Urgency (deadline?). 3. Dependencies (someone blocked?). 4. Delegate (can others handle?). Communicate priorities transparently, update if they shift.

---

## Culture & Motivation

**41. Why Ciena?**
> Backbone of internet infrastructure. Embedded DevOps is growth opportunity. My CI/CD/Docker/K8s skills transfer directly. Want to learn Yocto/Jenkins at scale/hardware workflows.

**42. Why leaving your current company?**
> Growth, not dissatisfaction. Looking for embedded systems challenge. Ciena offers DevOps + embedded Linux combination I want to learn.

**43. Interest in optical networking/embedded?**
> Infrastructure that matters (every cloud service relies on it). Embedded DevOps is different challenge (hours-long builds, hardware testing, firmware shipping). That complexity excites me.

**44. Ciena products knowledge?**
> WaveLogic coherent optical technology, 6500 series packet-optical platform, Blue Planet intelligent automation, adaptive networking approach using analytics. Team works on embedded software for networking devices.

**45. Where in 3-5 years?**
> Senior DevOps engineer architecting end-to-end CI/CD for embedded products. Deep Yocto expertise. Mentoring junior engineers. Contributing to DevOps strategy.

**46. Stay updated?**
> DevOps Weekly newsletter, CNCF blog, KubeCon recordings, hands-on labs for new tools, internal knowledge sharing sessions.

**47. Documentation approach?**
> Close to code (READMs, pipeline comments, ADRs). Tested runbooks. If a new team member can't follow it, it's not good enough.

**48. On-call?**
> Well-defined runbooks, actionable alerts (not noisy), clear escalation paths, post-incident reviews, fair rotation.

**49. Agile experience?**
> Daily standups, sprint planning, retrospectives. Azure Boards for tracking. Link work items to PRs and deployments for traceability.

**50. Questions for interviewer?**
> 1. Typical sprint for DevOps team? 2. Biggest CI/CD challenge? 3. How handle long Yocto build times? 4. Jenkins work vs infra automation split? 5. Onboarding process? 6. Gerrit workflow? 7. Firmware OTA update model?

---

## YOUR Experience Stories

**51. CI/CD at my current company.**
> Azure DevOps YAML pipelines. PR trigger for branches, CI on main. Build → Unit tests → SonarQube SAST + Snyk SCA (parallel) → Docker multi-stage build → ACR push → Helm deploy to AKS (dev auto, staging auto + smoke tests, prod manual approval + rolling update). Shared templates across 10+ services.

**52. JFrog to GitHub Packages migration.**
> S: Consolidating artifact management. T: Migrate npm/NuGet without disruption. A: Migration scripts, parallel publishing for 2 weeks, coordinated with 5 teams. R: Zero downtime, reduced licensing costs, 3-week onboarding.

**53. Terraform and Ansible usage.**
> Terraform: AKS clusters, Azure SQL, App Services, VNets, NSGs. State in Azure Blob with locking. Ansible: VM config — monitoring agents, log forwarding, OS hardening. Pipeline: Terraform provisions → Ansible configures.

**54. Prometheus/Grafana setup.**
> kube-prometheus-stack via Helm on AKS. App metrics (request rate, error rate, p50/p95/p99 latency), infra metrics (CPU, memory, disk). Custom dashboards per team. Alerts: error rate >5% → PagerDuty, CPU >80% → Slack, cert expiry → ticket.

**55. Docker and Kubernetes role.**
> Containerized 8 .NET services (multi-stage Dockerfiles). AKS with Helm charts. Resource requests/limits, probes, HPA, ConfigMaps, Sealed Secrets, Ingress with TLS.

**56. Distributed teams.**
> India + Europe. Async-first Slack, Confluence decisions, overlapping-hours meetings, shared dashboards.

**57. DevSecOps.**
> SonarQube SAST in every PR (quality gate), Snyk dependency scanning (auto-PR for fixes), Trivy container scanning, pre-commit hooks for secret detection.

**58. Pipeline broke.**
> Base image update (python:3.11-slim) broke pip behavior across 5 services. Pinned to specific digest, updated Dockerfiles, added monthly base image update process with testing.

**59. Ansible multi-environment.**
> Inventory groups per env, group_vars/ for env-specific variables, same playbooks. Vault-encrypted secrets per environment. Test in dev before promoting.

**60. Most impactful automation.**
> Automated release process: manual 2hr → 15min pipeline. Auto release notes from PR titles, git tag, Docker build/push, canary deploy, health validation. Bi-weekly → weekly releases, zero manual errors.

---

## Ciena-Specific

**61. Azure DevOps → Jenkins adaptation?**
> Same concepts (stages, triggers, agents). YAML → Jenkinsfile (Groovy). Key to learn: plugin ecosystem, shared libraries, credential management. Approach: convert existing pipeline to Jenkinsfile.

**62. Ramp up on Yocto?**
> Start with Poky: build core-image-minimal for QEMU. Study recipes/layers in Ciena codebase. Pair with experienced member. Analogies: recipe=Dockerfile, sstate-cache=Docker layer cache.

**63. Relevant to 100+ developer CI?**
> Managed CI for 10+ services/5 teams at my current company. Created shared templates. Understand: queue management, agent scaling, flaky tests, cache, standardization.

**64. Improve build reliability?**
> Measure failure rate, categorize (flaky/infra/real), quarantine flaky tests, stable agents with monitoring, pin dependencies, dashboard showing trends.

**65. Azure DevOps → Gerrit transition?**
> Different model: Gerrit reviews individual commits, not PRs. Push to refs/for/branch, +2 review, submit. Change-Id tracking. Underlying Git is same.

**66. Supporting developers with builds?**
> Good docs (searchable, tested), self-service tools, quick response to blockers, root cause over workarounds, knowledge sharing sessions.

**67. Optimize 2+ hour CI builds?**
> sstate-cache (biggest win), incremental builds, parallel BitBake threads, NVMe storage + lots of RAM, distributed compilation, nightly full + on-demand incremental.

**68. Metrics for DevOps impact?**
> DORA metrics: Deployment frequency, Lead time for changes, Change failure rate, MTTR. Plus: Build success rate, Build time, Developer satisfaction.

**69. New automation vs maintenance?**
> 70/30 rule: 70% maintain/improve existing, 30% build new. Track tech debt, allocate sprint capacity. New work driven by biggest pain point.

**70. First 90 days?**
> Month 1: Learn — codebase, build system, pipelines, processes. Shadow team. First small contribution. Month 2: Contribute — own pipeline improvements, fix pain points. Month 3: Impact — propose and implement significant improvement, share knowledge.

---

## Rapid Fire

**71. Docker vs Podman?** Docker needs daemon; Podman daemonless + rootless. Podman more secure. Both OCI-compatible.

**72. Jenkins vs GitHub Actions vs Azure Pipelines?** Jenkins: most flexible, self-hosted. GHA: best for GitHub. Azure: best for Azure ecosystem.

**73. Terraform vs Pulumi vs CloudFormation?** Terraform: multi-cloud, HCL, largest community. Pulumi: real languages. CloudFormation: AWS-only, free.

**74. Ansible vs Chef vs Puppet?** Ansible: agentless (SSH), YAML, easiest. Chef: Ruby, agent-based. Puppet: declarative, agent-based.

**75. Prometheus vs Datadog vs New Relic?** Prometheus: free, K8s-native, self-managed. Datadog/NR: SaaS, expensive, fully managed.

**76. ArgoCD vs Flux?** ArgoCD: better UI, more popular, multi-cluster. Flux: lighter, CNCF graduated, Helm controller.

**77. Monorepo vs Multirepo?** Monorepo: atomic changes, shared tooling. Multirepo: independent deploys, clearer ownership.

**78. GitFlow vs Trunk-Based?** GitFlow: complex, infrequent releases. Trunk-based: simple, continuous delivery. **Trunk-based is industry direction.**

**79. VM vs Container?** VM: full OS isolation, heavier. Container: process isolation, lightweight, faster. Containers for microservices.

**80. Merge vs Rebase?** Merge: preserves history, merge commit. Rebase: linear history, cleaner. Rebase for local branches, merge for main.
