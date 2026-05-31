> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Prep overview & interview structure (this file) |
| [01_coding_round.md](01_coding_round.md) | Coding round prep |
| [02_fixing_round.md](02_fixing_round.md) | Fixing round prep |
| [03_technical_round.md](03_technical_round.md) | Technical round prep |
| [04_values_round.md](04_values_round.md) | Values round prep |
| [05_go_complete_guide.md](05_go_complete_guide.md) | Go complete guide |
| [06_aws_deepdive.md](06_aws_deepdive.md) | AWS deep dive |
| [07_sli_slo_sla_guide.md](07_sli_slo_sla_guide.md) | SLI/SLO/SLA guide |
| [08_splunk_guide.md](08_splunk_guide.md) | Splunk guide |
| [09_python_practice.md](09_python_practice.md) | Python practice |
| [10_chaos_buildkite_pagerduty.md](10_chaos_buildkite_pagerduty.md) | Chaos, Buildkite & PagerDuty |
| [11_resume_walkthrough.md](11_resume_walkthrough.md) | Resume walkthrough |

---

# REA Group — Senior Engineer Platform | Interview Prep

## Company: REA Group
- **What**: Global leader in online real estate (realestate.com.au, Housing.com India)
- **Founded**: 1995, Melbourne, Australia
- **Scale**: Millions of daily property seekers across 3 continents
- **New India Tech Center**: Cyber City, Gurugram — accelerating global tech delivery
- **Tech Philosophy**: "Day one" startup mindset, continuous innovation

## Role: Senior Engineer - Platform (3-5 years)
- **Location**: Gurugram (Cyber City), Hybrid
- **Team**: Platform/Cloud Engineering

## Interview Rounds
1. **AI Coding & Analysis Round** — JD-based coding/analysis (GoLang/Python, K8s, automation)
2. **Server/Website Deployed Fixing Round** — Debug a live broken service, troubleshoot and fix
3. **Technical Round** — Deep-dive on platform engineering, K8s, cloud, SRE concepts
4. **Values Round** — Cultural fit, REA values, leadership, collaboration

---

## JD vs YOUR SKILL ALIGNMENT

| JD Requirement | Your Level | Gap | Priority |
|---|---|---|---|
| GoLang or Python (developer tooling) | Python strong, Go zero | Learn Go basics for K8s tooling | 🔴 CRITICAL |
| AWS or GCP | AWS moderate (EC2, S3, RDS, IAM) | AWS services deeper (EKS, CloudWatch, Lambda) | 🟡 HIGH |
| Kubernetes (deep) | Strong (AKS, Helm, deployments) | Write K8s operators, CRDs, admission webhooks | 🟡 HIGH |
| CI/CD / Build tools | Strong (Azure DevOps, Jenkins) | Learn Buildkite basics | 🟡 HIGH |
| SLI/SLO/SLA metrics | Conceptual (from ATS prep) | Need practical examples & calculations | 🔴 CRITICAL |
| Chaos Engineering | Conceptual (from ATS prep) | Need hands-on Chaos Mesh/Litmus | 🟡 HIGH |
| Splunk (logs, tracing, monitoring) | Grafana/Prometheus/ELK | Learn Splunk query basics (SPL) | 🟡 HIGH |
| PagerDuty | Not used | Learn setup, escalation policies | 🟢 MODERATE |
| Ruby / Shell scripting | Shell strong, Ruby zero | Ruby awareness only | 🟢 LOW |
| Migration automation | Terraform modules experience | Frame your migration work (secrets, artifacts) | 🟢 SOLID |
| Developer Experience (DevEx) | Jenkins shared libs, pipelines | Frame as platform-as-product | 🟢 SOLID |
| Troubleshooting complex issues | Strong (16 AWS findings, AKS) | Already solid | ✅ DONE |
| Capacity planning & scaling | Cost optimization experience | Frame with HPA/VPA/load testing | 🟢 SOLID |

---

## RESUME TO USE

**➡️ Platform Engineer (ATS Improved)** — `resumes/ats_improved/Vaibhav_Kumar_Platform_Engineer_ATS_Improved.pdf`

**Why**: JD title is "Senior Engineer - Platform", emphasizes developer tooling, Kubernetes platform, DevEx, SLI/SLO — all covered in your Platform Engineer resume. The SRE improved version is a close second if they lean more into reliability.

---

## HOW TO POSITION YOURSELF

### Strengths to Emphasize
- **Kubernetes platform at scale**: 13 microservices on AKS, Helm charts, multi-env (dev/QA/NFT/demo/preprod)
- **Developer tooling**: Jenkins Shared Libraries serving 10+ teams, golden-path workflows, self-service infra
- **Infrastructure as Code**: 11 Terraform modules, versioned, reusable, cross-team
- **Migration experience**: Secrets migration (Helm → Key Vault), artifact migration (JFrog → GitHub Packages)
- **Observability**: Prometheus, Grafana, Jaeger, Kiali — production monitoring stack
- **Security-first**: SonarQube, Snyk, Mend in CI, 16 AWS findings remediated
- **Cost optimization**: 20% Azure spend reduction through right-sizing

### For Gaps, Frame Positively
- **Go**: "My primary language is Python for DevOps tooling. I'm actively learning Go for its advantages in K8s ecosystem — static binaries, goroutines for concurrent ops, and strong K8s client-go library. I've already explored Go for building CLI tools."
- **AWS (over Azure)**: "I have hands-on AWS experience (EC2, S3, RDS, IAM, GuardDuty) and deep Azure expertise. Cloud concepts are transferable — I've worked multi-cloud and can ramp up on AWS-specific services quickly."
- **Splunk**: "I've built production observability stacks with Prometheus, Grafana, ELK, and Jaeger. Splunk's SPL query language is similar to KQL/LogQL patterns I already use. The core skill — correlating logs, metrics, and traces to diagnose issues — is the same."
- **Buildkite**: "I haven't used Buildkite specifically, but I've built extensive CI/CD in Jenkins and Azure DevOps. Buildkite's agent-based model is conceptually similar to Jenkins agents, and I can pick it up quickly."

---

## STUDY PRIORITY ORDER

| Priority | Topic | Time Needed | Study File | Impact |
|----------|-------|-------------|------------|--------|
| 1 | Go from absolute zero | 5-6 hours | `05_go_complete_guide.md` | Coding round |
| 2 | SLI/SLO/SLA practical (error budgets, burn rates, Prometheus) | 2-3 hours | `07_sli_slo_sla_guide.md` | Technical round |
| 3 | AWS deep-dive (EKS, IAM/IRSA, VPC, CloudWatch) | 3-4 hours | `06_aws_deepdive.md` | Technical round |
| 4 | Splunk SPL from zero (log analysis, dashboards) | 2-3 hours | `08_splunk_guide.md` | Fixing + Technical round |
| 5 | Webserver fixing / debugging (Nginx, Apache, Linux) | 2-3 hours | `02_fixing_round.md` | Fixing round |
| 6 | Python platform coding practice | 2-3 hours | `09_python_practice.md` | Coding round |
| 7 | Chaos Engineering + Buildkite + PagerDuty | 2-3 hours | `10_chaos_buildkite_pagerduty.md` | Technical round |
| 8 | K8s + Platform engineering deep technical | 2 hours | `03_technical_round.md` | Technical round |
| 9 | Go/Python/K8s coding patterns | 1-2 hours | `01_coding_round.md` | Coding round |
| 10 | REA values & behavioral STAR answers | 1 hour | `04_values_round.md` | Values round |

---

## FILE INDEX

| File | Content | Pages |
|------|---------|-------|
| `00_prep_overview.md` | This file — JD analysis, gap matrix, study order | — |
| `01_coding_round.md` | Go basics, Python tooling, K8s manifests, Buildkite pipeline patterns | ~200 lines |
| `02_fixing_round.md` | Webserver debugging: Nginx, Apache, DNS, SSL, systemd, logs, 12 scenarios | ~600 lines |
| `03_technical_round.md` | K8s deep-dive, RBAC, networking, SRE, AWS, observability, capacity planning | ~400 lines |
| `04_values_round.md` | REA values, STAR behavioral answers, cultural fit | ~200 lines |
| `05_go_complete_guide.md` | **Go from absolute zero**: variables, functions, goroutines, HTTP servers, K8s client-go, testing | ~1200 lines |
| `06_aws_deepdive.md` | **AWS mapped from Azure**: IAM/IRSA, VPC, EKS, ECR, CloudWatch, Route 53, S3, Lambda, IaC | ~600 lines |
| `07_sli_slo_sla_guide.md` | **Practical SRE**: SLI definitions, SLO targets, error budget math, burn rate, Prometheus rules, Grafana | ~500 lines |
| `08_splunk_guide.md` | **Splunk from zero**: SPL syntax, stats, timechart, eval, rex, K8s logs in Splunk, dashboards | ~600 lines |
| `09_python_practice.md` | **Python coding practice**: YAML/JSON, log parsing, HTTP APIs, K8s automation, CLI tools, testing | ~600 lines |
| `10_chaos_buildkite_pagerduty.md` | **Three topics**: chaos engineering (Chaos Mesh), Buildkite CI/CD pipelines, PagerDuty incident mgmt | ~700 lines |
| `11_resume_walkthrough.md` | **Resume prep**: "Tell me about yourself", deep-dive on every bullet, tricky questions, why leave/join | ~400 lines |
