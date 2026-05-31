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

| Priority | Topic | Time Needed | Impact |
|----------|-------|-------------|--------|
| 1 | Go basics for K8s tooling | 4-5 hours | Coding round |
| 2 | SLI/SLO/SLA practical scenarios | 2 hours | Technical round |
| 3 | AWS deep-dive (EKS, CloudWatch) | 3 hours | Technical round |
| 4 | Splunk basics (SPL queries) | 2 hours | Fixing round |
| 5 | Chaos Engineering hands-on | 2 hours | Technical round |
| 6 | Buildkite + PagerDuty awareness | 1 hour | Bonus points |
| 7 | REA company research & values | 1 hour | Values round |
