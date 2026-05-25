# CI/CD & DevOps — COMPREHENSIVE ANSWERS

---

## Basics

**1. What is CI/CD?**

```
┌─── CI (Continuous Integration) ─────────────────────────────────────┐
│                                                                      │
│  Developer A ──┐                                                     │
│  Developer B ──┼──► Merge to shared branch ──► Auto Build + Test    │
│  Developer C ──┘    (multiple times/day)       (catch bugs early)    │
│                                                                      │
│  Goal: Code always compiles, tests always pass                      │
└──────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─── CD (Continuous Delivery) ────────────────────────────────────────┐
│                                                                      │
│  Artifact is ALWAYS in a deployable state                           │
│  Manual approval gate before production                             │
│                                                                      │
│  Build ──► Test ──► Stage ──► [Manual Approval] ──► Production     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─── CD (Continuous Deployment) ──────────────────────────────────────┐
│                                                                      │
│  Every passing change goes AUTOMATICALLY to production              │
│  NO manual gate — fully automated end to end                        │
│                                                                      │
│  Build ──► Test ──► Stage ──► Production (automatic!)               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

| Aspect | Continuous Delivery | Continuous Deployment |
|--------|--------------------|-----------------------|
| Manual approval | Yes | No |
| Risk level | Lower (human check) | Higher (must trust tests) |
| Deploy speed | Hours/days | Minutes |
| Common in | Regulated industries | SaaS, web apps |

---

**2. Difference between Continuous Delivery and Continuous Deployment?**

- **Delivery** = artifact is always deployable, but a human decides when to push to production
- **Deployment** = every change that passes all tests goes to production automatically
- Delivery requires strong test suites; Deployment requires **exceptional** test suites

---

**3. What is DevOps? Role, culture, or tools?**

```
┌─── DevOps = Culture + Practices + Tools ────────────────────────────┐
│                                                                      │
│  Traditional:                                                        │
│    Dev Team ──► "Here's code" ──► Wall ──► Ops Team ──► Deploy      │
│    (builds)       (throws over)   🧱      (deploys)                  │
│                                                                      │
│  DevOps:                                                             │
│    Dev + Ops ──► Shared Ownership ──► Automate Everything            │
│    (collaborate)  (you build it,       (CI/CD, IaC, monitoring)      │
│                    you run it)                                        │
│                                                                      │
│  DevOps is primarily a CULTURE that:                                │
│    ✅ Breaks silos between Dev and Ops                               │
│    ✅ Emphasizes automation and feedback loops                       │
│    ✅ Treats infrastructure as code                                  │
│    ✅ Promotes shared accountability                                 │
│                                                                      │
│  "DevOps Engineer" = a job title born from this culture             │
└──────────────────────────────────────────────────────────────────────┘
```

---

**4. Key principles of DevOps?**

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DevOps Principles (CALMS)                        │
│                                                                      │
│  C — Culture:       Collaboration, shared ownership, no blame       │
│  A — Automation:    CI/CD, IaC, testing, monitoring                 │
│  L — Lean:          Eliminate waste, small batches, fast flow        │
│  M — Measurement:   DORA metrics, feedback loops, data-driven       │
│  S — Sharing:       Knowledge sharing, blameless post-mortems       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

Additional principles:
- Everything as Code (infrastructure, config, pipelines)
- Small, frequent releases (not big-bang deploys)
- Continuous improvement (retrospectives, feedback)
- Monitoring and observability from day 1

---

**5. DevOps lifecycle phases?**

```
       ┌────────────────────────────────────────────┐
       │          DevOps Infinity Loop               │
       │                                              │
       │    Plan ──► Code ──► Build ──► Test          │
       │      ▲                            │          │
       │      │                            ▼          │
       │   Monitor ◄── Operate ◄── Deploy ◄── Release│
       │                                              │
       └────────────────────────────────────────────┘

  DEV side (left loop):                OPS side (right loop):
  Plan → Code → Build → Test          Release → Deploy → Operate → Monitor

  Continuous feedback from Monitor → Plan closes the loop
```

---

**6. What is Infrastructure as Code?**

```
┌─── Manual (Bad) ────────────────┬─── IaC (Good) ──────────────────┐
│                                  │                                  │
│  Click in Azure Portal           │  Write HCL/YAML code            │
│  SSH in, run commands            │  Version control (Git)           │
│  Document in wiki (outdated)     │  Code IS the documentation      │
│  "Who changed the firewall?"     │  PR review + audit trail         │
│  Impossible to reproduce         │  Reproducible in any env         │
│  Snowflake servers               │  Cattle, not pets                │
│                                  │                                  │
│  Tools: none (hands)             │  Tools: Terraform, Ansible,      │
│                                  │  CloudFormation, Pulumi           │
└──────────────────────────────────┴──────────────────────────────────┘
```

---

**7. Configuration Management vs Provisioning?**

```
Provisioning (Day 0):              Configuration (Day 1+):
┌──────────────────────┐          ┌──────────────────────┐
│ CREATE infrastructure │          │ CONFIGURE infra       │
│                        │          │                        │
│ VMs, networks, LBs,   │          │ Install packages,     │
│ K8s clusters, storage  │          │ deploy apps, manage   │
│                        │          │ configs, start services│
│                        │          │                        │
│ Tools: Terraform,      │          │ Tools: Ansible, Chef, │
│ CloudFormation, Pulumi │          │ Puppet, Salt           │
└──────────────────────┘          └──────────────────────┘
         │                                  │
         └──────────┬───────────────────────┘
                    ▼
            Often used TOGETHER:
            Terraform creates VM → Ansible configures it
```

---

**8. Imperative vs Declarative?**

```
Imperative ("how to do it"):       Declarative ("what I want"):
┌──────────────────────────┐      ┌──────────────────────────┐
│ Step 1: Create VM         │      │ "I want 3 VMs with       │
│ Step 2: Assign 4GB RAM    │      │  4GB RAM in East US"     │
│ Step 3: Open port 80      │      │                          │
│ Step 4: Install nginx     │      │ System figures out HOW.  │
│ Step 5: Start nginx       │      │ Already have 2?          │
│                            │      │ → Creates only 1 more.   │
│ Already have 2?            │      │                          │
│ → Script creates 3 MORE!  │      │ Idempotent ✅            │
│   (total = 5 ❌)           │      │                          │
│                            │      │ Tools: Terraform, K8s    │
│ Tools: Shell scripts,      │      │ YAML, CloudFormation     │
│ Ansible ad-hoc              │      │                          │
└──────────────────────────┘      └──────────────────────────┘
```

---

**9. What is a build artifact? 5 examples.**

Output produced by a build process that is deployed or consumed:
1. **Docker image** — containerized application
2. **JAR/WAR file** — Java application
3. **Python wheel (.whl)** — Python package
4. **npm package** — JavaScript library
5. **Compiled binary** — Go binary, C++ executable
6. **Zip/tar archive** — bundled application code

```
Source Code ──► Build Process ──► Artifact ──► Deploy to Environment
                (compile,          (stored in     (dev, staging, prod)
                 test, package)     Artifactory,
                                    ACR, S3)
```

---

**10. What does "shift left" mean?**

```
Traditional (find bugs late, expensive):
  Code ──► Build ──► Test ──► Stage ──► PROD ──► FIND BUGS! 💥
                                                   (costs $$$)

Shift Left (find bugs early, cheap):
  Code ──► FIND BUGS! ──► Build ──► Test ──► Stage ──► PROD ✅
  (pre-commit hooks,      (lint,    (unit,    (integration)
   IDE warnings,           SAST,     SCA)
   code review)            scan)

Cost to fix a bug:
  Design phase:    $1
  Development:     $10
  Testing:         $100
  Production:      $1,000+
```

Shift left applies to: testing, security (DevSecOps), quality, compliance.

---

## Pipeline Concepts (11–20)

**11. Build pipeline — typical stages?**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        BUILD PIPELINE                               │
│                                                                     │
│  ┌──────────┐  ┌─────────┐  ┌──────┐  ┌───────────┐  ┌─────────┐ │
│  │ Checkout  │→ │ Install  │→ │ Lint │→ │ Unit Test │→ │ Build   │ │
│  │ Source    │  │ Deps     │  │      │  │           │  │ Artifact│ │
│  └──────────┘  └─────────┘  └──────┘  └───────────┘  └─────────┘ │
│       │                                                     │       │
│  git clone/                                           Docker build  │
│  checkout                                             or compile    │
│                                                                     │
│  ┌───────────┐  ┌──────────────┐  ┌──────────────┐                │
│  │ SAST Scan │→ │ Publish      │→ │ Notify Team  │                │
│  │ (SonarQube)│  │ Artifact     │  │ (Slack/Teams)│                │
│  └───────────┘  │ (ACR/Nexus)  │  └──────────────┘                │
│                  └──────────────┘                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

**12. Release pipeline vs build pipeline?**

```
Build Pipeline:                     Release Pipeline:
┌──────────────────────────┐       ┌──────────────────────────────────┐
│ Compile + Test + Package  │       │ Deploy artifact to environments  │
│ Output: artifact          │       │ Input: artifact from build       │
│                            │  ──►  │                                  │
│ Triggered by: code push   │       │ Dev → Staging → [Approve] → Prod │
│ Focus: quality             │       │ Focus: delivery                  │
└──────────────────────────┘       └──────────────────────────────────┘

Modern: Often combined into single multi-stage pipeline
```

---

**13. Pipeline trigger types?**

| Trigger | Description | Example |
|---------|-------------|---------|
| **SCM Webhook** | Push or PR event | Push to main → build |
| **Schedule/Cron** | Time-based | Nightly builds at 2 AM |
| **Manual** | Human-initiated | Production deploy |
| **Pipeline completion** | Chained trigger | Build done → deploy |
| **API call** | External system | ChatOps `/deploy prod` |
| **Tag** | Git tag created | `v1.0.0` → release build |

---

**14. Artifact repository?**

Stores build outputs for deployment. **Build once, deploy many** principle.

| Tool | Best For |
|------|----------|
| JFrog Artifactory | Multi-format (Docker, npm, PyPI, Maven) |
| Azure Artifacts | Azure DevOps ecosystem |
| GitHub Packages | GitHub ecosystem |
| Nexus Repository | Self-hosted, open-source |
| Docker Registry/ACR | Container images |

---

**15. Stage vs Job vs Step?**

```
Pipeline
├── Stage: BUILD                    ← Logical grouping (Build, Test, Deploy)
│   ├── Job: compile-app            ← Unit of work on one agent
│   │   ├── Step: checkout code     ← Individual command
│   │   ├── Step: install deps
│   │   ├── Step: run tests
│   │   └── Step: build artifact
│   └── Job: lint-code              ← Can run in PARALLEL with compile
│       ├── Step: run linter
│       └── Step: publish results
│
├── Stage: DEPLOY-STAGING           ← Stages run SEQUENTIALLY by default
│   └── Job: deploy
│       ├── Step: download artifact
│       └── Step: deploy to staging
│
└── Stage: DEPLOY-PROD
    └── Job: deploy
        ├── Step: approval gate
        └── Step: deploy to prod
```

---

**16. Agent/Runner — what & why not on controller?**

```
┌─── Controller (Jenkins Master / Azure DevOps Server) ───┐
│                                                           │
│  Manages: UI, scheduling, plugins, credentials           │
│  Should NOT run builds because:                          │
│    ❌ Security risk (build code runs with controller perms)│
│    ❌ Performance bottleneck                              │
│    ❌ Single point of failure                             │
│                                                           │
│  Dispatches jobs to:                                     │
│    Agent 1 (Linux)  ──► runs build jobs                  │
│    Agent 2 (Windows) ──► runs .NET builds                │
│    Agent 3 (Docker) ──► runs containerized builds        │
└───────────────────────────────────────────────────────────┘

Self-hosted:  Your machines (full control, persistent)
Cloud-hosted: Managed by provider (clean env each run, no maintenance)
```

---

**17. Variables vs Secrets?**

```
Variables (non-sensitive):          Secrets (sensitive):
┌──────────────────────────┐      ┌──────────────────────────┐
│ BUILD_CONFIG=Release      │      │ DB_PASSWORD=S3cr3t!      │
│ ENVIRONMENT=staging       │      │ API_KEY=ak_live_xyz...   │
│ IMAGE_TAG=v1.2.3          │      │ DOCKER_TOKEN=dkr_pat_... │
│                            │      │                          │
│ Visible in logs ✅        │      │ Masked in logs ****      │
│ Stored in YAML/config     │      │ Encrypted at rest         │
│ Can be overridden         │      │ Access-controlled         │
│                            │      │ Stored in Key Vault/     │
│                            │      │ Secrets Manager          │
└──────────────────────────┘      └──────────────────────────┘
```

---

**18. Pipeline template — what & why?**

Reusable pipeline definition shared across repos/teams:
- **DRY**: Write once, use in 50+ repos
- **Standards**: Enforce consistent build/test/deploy stages
- **Maintenance**: Update template → all consumers get fix
- **Governance**: Security scans can't be skipped

```yaml
# Example: template usage
stages:
- template: templates/dotnet-build.yml
  parameters:
    project: src/MyApp.csproj
    environment: staging
```

---

**19. Self-hosted vs cloud-hosted agents?**

| Feature | Self-hosted | Cloud-hosted |
|---------|------------|--------------|
| Control | Full (custom tools, GPU) | Limited (pre-defined images) |
| Maintenance | You manage | Provider manages |
| Environment | Persistent (cache survives) | Clean each run |
| Cost | Your hardware | Per-minute billing |
| Speed | Faster (warm cache) | Slower (cold start) |
| Security | Internal network access | External network |
| Best for | Long builds, special tools, Yocto | Standard web apps |

---

**20. What happens when a pipeline step fails?**

```
Step 1: Checkout ✅
Step 2: Build    ✅
Step 3: Test     ❌ FAILED!
Step 4: Deploy   ⏭️ SKIPPED (by default)
Step 5: Cleanup  ✅ (if condition: always())

Handling strategies:
  continueOnError: true     → next steps still run
  condition: always()       → runs regardless of prior failure
  retry: 3                  → retry failed step up to 3 times
  post { failure { } }      → notify team on failure
  manual intervention gate  → human decides what to do
```

---

## Deployment Strategies (21–30)

**21. Blue-Green deployment?**

```
                     Load Balancer
                          │
              ┌───────────┼───────────┐
              │                       │
      ┌───────▼──────┐       ┌───────▼──────┐
      │  BLUE (v1)    │       │  GREEN (v2)  │
      │  ✅ LIVE       │       │  🔄 Staging  │
      │  serving users │       │  testing...  │
      └───────────────┘       └──────────────┘

After validation, switch traffic:

              ┌───────────┼───────────┐
              │                       │
      ┌───────▼──────┐       ┌───────▼──────┐
      │  BLUE (v1)    │       │  GREEN (v2)  │
      │  ⏸️ Standby   │       │  ✅ LIVE      │
      │  (rollback)   │       │  serving users│
      └───────────────┘       └──────────────┘

Rollback: Just switch LB back to Blue (seconds!)
Downside: Need 2x infrastructure
```

---

**22. Canary deployment?**

```
Phase 1:  5% traffic → v2    95% → v1
          ┌──┐               ┌──────────┐
          │v2│               │    v1    │
          └──┘               └──────────┘
          Monitor: error rate, latency, business metrics

Phase 2:  25% traffic → v2   75% → v1
          ┌───────┐          ┌────────┐
          │  v2   │          │  v1    │
          └───────┘          └────────┘
          If metrics OK → continue

Phase 3:  100% traffic → v2
          ┌──────────────────┐
          │       v2          │
          └──────────────────┘
          Done! v1 scaled down.

If metrics BAD at any phase → route 100% back to v1 (auto-rollback)
```

---

**23. Rolling deployment?**

```
Start:    [v1] [v1] [v1] [v1] [v1]    (5 instances)

Step 1:   [v2] [v1] [v1] [v1] [v1]    Replace 1st
Step 2:   [v2] [v2] [v1] [v1] [v1]    Replace 2nd
Step 3:   [v2] [v2] [v2] [v1] [v1]    Replace 3rd
Step 4:   [v2] [v2] [v2] [v2] [v1]    Replace 4th
Step 5:   [v2] [v2] [v2] [v2] [v2]    Done!

K8s controls:
  maxSurge: 1        → max 1 extra pod during update
  maxUnavailable: 0  → zero downtime (always N pods available)
```

---

**24. A/B Testing vs Canary?**

| Aspect | A/B Testing | Canary |
|--------|-------------|--------|
| Goal | Measure **business** metrics | Measure **technical** health |
| Routing | By user attribute (geo, ID) | By percentage |
| Metrics | Conversion rate, revenue | Error rate, latency |
| Duration | Weeks | Minutes to hours |
| Use case | "Does new checkout flow sell more?" | "Does v2 crash?" |

---

**25. Feature flags?**

```
Code is deployed with flag OFF:
  if (featureFlags.isEnabled("new-checkout")):
      return new_checkout()    ← hidden from users
  else:
      return old_checkout()    ← users see this

When ready: flip flag ON (no deployment needed!)

Benefits:
  ✅ Deploy code without exposing feature
  ✅ Trunk-based development (no long-lived branches)
  ✅ Kill switch for broken features
  ✅ A/B testing

Tools: LaunchDarkly, Unleash, Flagsmith, ConfigCat
```

---

**26. Immutable infrastructure?**

```
Mutable (bad):                    Immutable (good):
┌──────────────────────┐        ┌──────────────────────┐
│ Server running v1     │        │ Image v1 ──► Server A │
│ SSH in, update to v2  │        │                       │
│ SSH in, patch library │        │ Need v2?              │
│ SSH in, fix config    │        │ Build NEW Image v2    │
│                        │        │ Deploy NEW Server B   │
│ "Configuration drift"  │        │ Destroy Server A      │
│ "Snowflake server"    │        │                       │
│ Can't reproduce!       │        │ Always reproducible!  │
└──────────────────────┘        └──────────────────────┘
```

---

**27. Rollback strategies by deployment type?**

| Strategy | Rollback Method | Speed |
|----------|----------------|-------|
| **Blue-Green** | Switch LB back to old env | Seconds |
| **Canary** | Route 100% back to v1 | Seconds |
| **Rolling** | `kubectl rollout undo` | Minutes |
| **Recreate** | Redeploy old version | Slowest |
| **Feature flag** | Flip flag off | Instant |

---

**28. Dark launch?**

Deploy new feature to production but **don't expose** to users. Test with real production traffic/data behind the scenes. Validate performance and correctness without user impact. Then flip feature flag ON.

---

## Advanced CI/CD (29–35)

**29. GitOps — what and how?**

```
Traditional CI/CD (push-based):
  Developer ──► Pipeline ──► PUSH to cluster
                              (kubectl apply)

GitOps (pull-based):
  Developer ──► PR to Git ──► Merge
                                 │
                    ┌────────────▼──────────────┐
                    │  ArgoCD / Flux (in-cluster)│
                    │  Watches Git repo           │
                    │  PULLS desired state        │
                    │  Syncs cluster to match Git │
                    └─────────────────────────────┘

Benefits:
  ✅ Git = single source of truth
  ✅ Audit trail (Git history)
  ✅ Easy rollback (git revert)
  ✅ Drift detection (cluster ≠ Git → alert)
  ✅ No kubectl access needed for developers
```

---

**30. Trunk-based development?**

```
GitFlow (complex):                 Trunk-based (simple):
  main ────────────────             main ──●──●──●──●──●──●──
  develop ─────────────               ↑  ↑  ↑  ↑  ↑  ↑
  feature/a ───────────               └──┘  └──┘  └──┘
  feature/b ───────────              Short-lived branches
  release/1.0 ─────────              (< 1 day, few commits)
  hotfix/x ────────────
                                    + Feature flags for incomplete work
  Long-lived branches =             + All developers commit to main
  merge conflicts + pain            + Enables continuous deployment
```

---

**31. DORA metrics — measure DevOps performance?**

```
┌─── DORA Metrics ────────────────────────────────────────────────────┐
│                                                                      │
│  ┌─────────────────────┐  ┌─────────────────────────┐              │
│  │ Deployment Frequency │  │ Lead Time for Changes    │              │
│  │ How often deploy?    │  │ Commit → Production?     │              │
│  │                       │  │                           │              │
│  │ Elite: multiple/day  │  │ Elite: < 1 hour          │              │
│  │ High:  weekly        │  │ High:  < 1 week          │              │
│  │ Low:   monthly       │  │ Low:   > 1 month         │              │
│  └─────────────────────┘  └─────────────────────────┘              │
│                                                                      │
│  ┌─────────────────────┐  ┌─────────────────────────┐              │
│  │ Change Failure Rate  │  │ MTTR (Mean Time to       │              │
│  │ % causing incidents? │  │ Restore)                  │              │
│  │                       │  │ How fast to recover?     │              │
│  │ Elite: < 5%          │  │ Elite: < 1 hour          │              │
│  │ High:  < 15%         │  │ High:  < 1 day           │              │
│  │ Low:   > 30%         │  │ Low:   > 1 month         │              │
│  └─────────────────────┘  └─────────────────────────┘              │
│                                                                      │
│  These 4 metrics predict software delivery performance              │
└──────────────────────────────────────────────────────────────────────┘
```

---

**32. SLI / SLO / SLA?**

```
SLI (Service Level Indicator):
  The measurement itself
  "99.5% of requests complete in < 200ms"

SLO (Service Level Objective):
  The target you set internally
  "We aim for 99.9% availability per month"

SLA (Service Level Agreement):
  The contract with customers (has consequences!)
  "99.5% uptime guaranteed — or customer gets credit"

Error Budget = 100% - SLO
  If SLO = 99.9% → error budget = 0.1% = 43.8 min/month
  Budget consumed? → freeze features, fix reliability
  Budget available? → safe to push risky changes
```

---

**33. Pipeline-as-Code?**

Pipeline definition stored in a file committed to the repo:

| Tool | File | Language |
|------|------|----------|
| Jenkins | `Jenkinsfile` | Groovy |
| Azure DevOps | `azure-pipelines.yml` | YAML |
| GitHub Actions | `.github/workflows/*.yml` | YAML |
| GitLab CI | `.gitlab-ci.yml` | YAML |
| CircleCI | `.circleci/config.yml` | YAML |

Benefits: version controlled, reviewable via PR, auditable, reproducible, no UI drift.

---

**34. Artifact promotion — build once, deploy many?**

```
❌ Bad: Rebuild for each environment
  Dev build ──► Dev         (artifact A)
  Staging build ──► Staging (artifact B — could be different!)
  Prod build ──► Prod       (artifact C — could be different!)

✅ Good: Build once, promote same artifact
  Build ──► Artifact v1.2.3
              │
              ├──► Deploy to Dev      (same artifact)
              ├──► Deploy to Staging  (same artifact)
              └──► Deploy to Prod     (same artifact)

  Only CONFIG differs between environments, not the artifact!
```

---

**35. Push-based vs Pull-based deployment?**

```
Push-based:                         Pull-based:
┌────────────────────────┐         ┌────────────────────────┐
│ CI tool (Jenkins) pushes│         │ Agent in cluster pulls  │
│ to target environment   │         │ from Git repo           │
│                          │         │                          │
│ CI ──► kubectl apply    │         │ ArgoCD watches Git      │
│                          │         │ Detects change → syncs  │
│                          │         │                          │
│ CI needs cluster creds  │         │ No external access      │
│ Less secure              │         │ More secure             │
│                          │         │                          │
│ Tools: Jenkins, Azure   │         │ Tools: ArgoCD, Flux     │
│ Pipelines (direct)      │         │                          │
└────────────────────────┘         └────────────────────────┘
```

---

## Interview-Style Scenarios (21–35 from intermediate)

**21. Design CI/CD for a team of 10?**

```
┌─── Developer Workflow ──────────────────────────────────────────────┐
│                                                                      │
│  1. Developer creates feature branch from main                      │
│  2. Commits code, pushes branch                                     │
│  3. Opens PR → triggers CI pipeline:                                │
│     ┌───────────────────────────────────────────────────────────┐   │
│     │  Lint → Unit Tests → SAST → Build → Container Scan       │   │
│     │  (all parallel where possible)                             │   │
│     │  Results posted as PR comment                              │   │
│     └───────────────────────────────────────────────────────────┘   │
│  4. PR reviewed by 1-2 peers → approved → merged to main           │
│  5. Main branch → auto-deploy to dev → auto-deploy to staging      │
│  6. Staging validated → manual approval → deploy to production     │
│                                                                      │
│  Key tools:                                                         │
│    Git: branching, PRs               Pipeline: Azure DevOps/Jenkins │
│    Registry: ACR/Docker Hub          Deploy: ArgoCD/Helm            │
│    Monitoring: Prometheus+Grafana    Secrets: Azure Key Vault       │
└──────────────────────────────────────────────────────────────────────┘
```

---

**22. Pipeline metrics to track?**

| Metric | Why It Matters |
|--------|---------------|
| Build duration | Slow → developers lose flow |
| Build success rate | < 90% → investigate |
| Deployment frequency | DORA metric → measures velocity |
| Lead time (commit → prod) | DORA metric → measures efficiency |
| Change failure rate | DORA metric → measures quality |
| MTTR | DORA metric → measures resilience |
| Test coverage | < 80% → insufficient |
| Queue time | High → need more agents |
| Flaky test rate | > 5% → quarantine + fix |

---

**23. Single pipeline vs multiple pipelines?**

| Single Pipeline | Multiple Pipelines |
|-----------------|-------------------|
| Tightly coupled components | Independent microservices |
| Small project | Different languages/teams |
| Same deploy cadence | Different deploy cadences |
| Simpler | More scalable |

---

**24. Database changes in CI/CD?**

```
Migration-based approach:
  V1__create_users.sql
  V2__add_email_column.sql
  V3__create_orders.sql

Pipeline:
  1. Run migrations BEFORE app deploy (Flyway/Liquibase)
  2. Migrations must be backward-compatible
  3. Never drop columns used by current version
  4. Always have rollback scripts
  5. Test migrations in staging first

  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ Migrate  │ →  │ Deploy   │ →  │ Validate │
  │ Database │    │ App v2   │    │ Health   │
  └──────────┘    └──────────┘    └──────────┘
```

---

**25. Flaky tests — how to handle?**

```
Detection:
  Track test results over time → flag tests that pass/fail randomly

Strategy:
  1. Quarantine: Move flaky test to separate suite (runs but doesn't block)
  2. Retry: Allow 1-2 retries (masks problem but unblocks pipeline)
  3. Fix: Assign owner, require fix within SLA (1 sprint)
  4. Track: Dashboard showing flaky test rate trend
  5. Root causes: timing issues, shared state, external dependencies

  Goal: < 2% flaky rate
```

---

## Culture & DevOps Concepts (21–35 from intermediate)

**26. Blameless post-mortem?**

```
After every incident:
  ┌──────────────────────────────────────────────────────────────┐
  │ 1. Timeline: What happened, when?                           │
  │ 2. Root cause: WHY did it happen? (5 Whys)                 │
  │ 3. Impact: Who was affected? For how long?                 │
  │ 4. What went well: Detection, response, communication      │
  │ 5. What to improve: Action items with owners + deadlines   │
  │                                                              │
  │ ❌ "John broke production"                                  │
  │ ✅ "The deployment process allowed untested config to ship" │
  │                                                              │
  │ Blame → people hide mistakes → worse outcomes               │
  │ Blameless → people report openly → system improvements     │
  └──────────────────────────────────────────────────────────────┘
```

---

**27. Three Ways of DevOps?**

```
1st Way — FLOW (left to right):
  Dev → Ops → Customer (fast, small batches, no waste)

2nd Way — FEEDBACK (right to left):
  Customer → Ops → Dev (fast feedback, monitoring, alerts)

3rd Way — CONTINUOUS LEARNING:
  Experimentation, risk-taking, learning from failures
```

---

**28. Toil — what is it, how to reduce?**

```
Toil = manual, repetitive, automatable work that scales linearly

Examples of toil:
  ❌ Manually provisioning VMs for each request
  ❌ SSH-ing into servers to check logs
  ❌ Manually rotating secrets every quarter
  ❌ Copy-pasting pipeline YAML for new services

Reduce via:
  ✅ Self-service portals
  ✅ Automation (IaC, CI/CD)
  ✅ Centralized logging (no SSH)
  ✅ Auto-rotation (Key Vault)
  ✅ Pipeline templates

Google SRE: Keep toil < 50% of team's time
```

---

**29. Value Stream Mapping?**

```
Visualize the entire delivery process:

  Idea → Design → Code → Review → Build → Test → Stage → Prod
  [2d]   [3d]    [2d]   [1d]     [5m]   [30m]  [1d]    [2h]
                          ↑                              ↑
                    Wait time: 1d                  Wait time: 3d

  Total lead time: 10 days
  Actual work time: 5 days
  Wait/waste: 5 days (50%!)

  → Identify bottlenecks and eliminate waste
```

---

**30. ChatOps?**

Managing operations through chat (Slack/Teams) with bot integrations:
```
#deployments channel:
  /deploy myapp staging       → triggers pipeline
  /status myapp production    → shows current version
  /rollback myapp production  → triggers rollback
  /incident create P2         → creates incident

Benefits: visibility, audit trail, knowledge sharing
```

---

**31–35. Scenario answers should be personalized, but key frameworks:**

- **Progressive delivery**: Umbrella term for canary, feature flags, A/B testing — gradually expose changes
- **Error budget**: 100% - SLO — when consumed, freeze features
- **Monorepo CI**: Path-based triggers, dependency graph analysis, parallel builds
- **Diamond dependency**: A depends on B and C, both depend on D at different versions → use lock files, monorepo single version
- **Secrets rotation**: External secret manager, short-lived tokens, automated rotation, zero-downtime rotation plan
