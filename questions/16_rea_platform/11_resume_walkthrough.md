# Resume Walk-Through — Interview Prep

> Every interview starts with "Tell me about yourself" or "Walk me through your resume."
> This file prepares you to answer questions about EVERY bullet point in your resume confidently,
> with depth, context, and follow-up readiness.

---

## TABLE OF CONTENTS

1. [Tell Me About Yourself (2-Minute Pitch)](#1-pitch)
2. [SITA — Terraform & IDP Questions](#2-sita-terraform)
3. [SITA — Secrets Management (Key Vault)](#3-sita-secrets)
4. [SITA — CI/CD Pipelines (Azure DevOps)](#4-sita-cicd)
5. [SITA — Observability Stack](#5-sita-observability)
6. [SITA — AWS Security Remediation](#6-sita-aws)
7. [SITA — Container Image Supply Chain](#7-sita-images)
8. [SITA — Cost Optimization](#8-sita-cost)
9. [Knoldus — Jenkins Shared Libraries](#9-knoldus-jenkins)
10. [Knoldus — JFrog to GitHub Packages Migration](#10-knoldus-jfrog)
11. [Knoldus — DevSecOps & Quality Gates](#11-knoldus-devsecops)
12. [Knoldus — Ansible Configuration Management](#12-knoldus-ansible)
13. [REOMNIFY — Data Engineering](#13-reomnify)
14. [Certifications Deep-Dive](#14-certs)
15. [Achievements & Awards](#15-awards)
16. [Why Are You Looking to Leave / Why This Company?](#16-why-leave)
17. [Tricky Resume Questions & How to Handle](#17-tricky)

---

## 1. TELL ME ABOUT YOURSELF (2-Minute Pitch) <a name="1-pitch"></a>

### The Formula: Present → Past → Future

**Script (adapt per company):**

> "I'm Vaibhav Kumar, a Cloud Infrastructure and DevOps Engineer with 4.5+ years of experience.
>
> Currently, I'm at SITA in Gurugram, where I'm building an Internal Developer Platform on Azure Kubernetes Service. I've designed 11 reusable Terraform modules for our entire infrastructure, migrated all application secrets to Azure Key Vault eliminating plaintext credentials, and deployed a full observability stack with Prometheus, Grafana, and Jaeger that reduced our detection time for issues by 60%.
>
> Before SITA, I spent 3 years at Knoldus — now NashTech Global — where I built Jenkins Shared Library pipelines serving 10+ teams, migrated our artifact management from JFrog to GitHub Packages saving $15K annually, and integrated security scanning into all our CI pipelines.
>
> I'm looking for my next challenge where I can work on platform engineering at scale — building developer-facing infrastructure, improving reliability, and helping engineering teams ship faster and safer."

### Tips
- **Tailor the "future" sentence** to match the JD you're interviewing for
- **Keep it under 2 minutes** — they'll ask follow-ups
- **Don't read your resume** — tell a story arc (growth from DevOps → Platform → SRE thinking)
- **Mention numbers** — they stick ("11 modules", "60% MTTD reduction", "$15K savings")

---

## 2. SITA — TERRAFORM & IDP QUESTIONS <a name="2-sita-terraform"></a>

**Resume Line**: *"Designed and maintained an Internal Developer Platform spanning AKS clusters, VNets, Application Gateways, and PostgreSQL Flexible Servers, codified as 11 reusable Terraform modules with versioned state management across 3 subscription tiers (dev, preprod, production), achieving 95% infrastructure-as-code coverage."*

### Q: "Tell me about the 11 Terraform modules you built."

**Answer:**
"I designed a modular Terraform codebase where each Azure resource type is its own versioned module. The 11 modules cover:

1. **AKS Cluster** — node pools, RBAC, managed identity, CNI networking
2. **VNet + Subnets** — address spaces, NSGs, service endpoints
3. **Application Gateway** — WAF v2, SSL termination, backend pools
4. **PostgreSQL Flexible Server** — HA, backup policies, firewall rules
5. **Key Vault** — access policies, soft delete, purge protection
6. **Container Registry (ACR)** — geo-replication, admin disabled
7. **Storage Account** — blob, queue, lifecycle policies
8. **Log Analytics Workspace** — diagnostic settings, retention
9. **Private DNS Zones** — for private endpoints
10. **Network Security Groups** — rule sets per subnet
11. **Azure Monitor** — action groups, alert rules

Each module has its own Git tag versioning, input validation, and output references. Teams consume them via `source = "git::https://..."` with version pinning."

### Follow-up: "How did you manage Terraform state?"

"We use Azure Storage Account as the backend — one storage account per subscription tier. Each module's state is in a separate container with a unique key. We have state locking via Azure Blob lease. For cross-module references, we use `terraform_remote_state` data sources — for example, the AKS module reads the VNet module's output to get subnet IDs."

### Follow-up: "How did you handle secrets in Terraform?"

"Sensitive variables are never stored in `.tfvars` files. We use Azure Key Vault data sources to pull secrets at plan time, and the Terraform state is encrypted at rest in Azure Storage. We also use `sensitive = true` on output values to prevent them from showing in logs."

### Follow-up: "What's 95% IaC coverage mean? What's the other 5%?"

"The 5% is manual resources that existed before I joined — some legacy DNS records and a few one-off resources in the sandbox account. We have a backlog item to import them into Terraform state, but they're low-priority non-production resources."

---

## 3. SITA — SECRETS MANAGEMENT <a name="3-sita-secrets"></a>

**Resume Line**: *"Migrated credentials from Helm chart values to Azure Key Vault with CSI Secret Store Driver across 4 product lines, eliminating 100% of plaintext secrets."*

### Q: "Walk me through the secrets migration."

**Answer:**
"When I joined, application secrets — database passwords, API keys, certificates — were stored as plaintext in Helm `values.yaml` files committed to Git. This was a major security risk.

**What I did:**
1. **Audited all secrets** across 4 product Helm charts (DI Agents, Proven, Aware, SITA Air) — found 40+ secrets in Git
2. **Created Azure Key Vaults** per environment (dev, QA, preprod, prod) using my Terraform module
3. **Installed CSI Secret Store Driver** on AKS via Helm
4. **Created SecretProviderClass resources** for each namespace mapping Key Vault secrets to pod volumes
5. **Updated Helm charts** to mount secrets as volumes instead of environment variables from values.yaml
6. **Set up Managed Identity** for AKS to authenticate to Key Vault (no service principal passwords)
7. **Removed all secrets from Git history** using `git filter-branch`

**Result:** Zero plaintext secrets in any Git repository. Secret rotation is now done in Key Vault without redeploying."

### Follow-up: "CSI Secret Store Driver vs K8s Secrets — what's the difference?"

"Kubernetes Secrets are base64-encoded (not encrypted) and stored in etcd. Anyone with `kubectl get secret` access can read them. With CSI Driver, secrets live in Key Vault and are mounted as files into the pod at runtime — they never exist as K8s Secret objects. The pod's managed identity authenticates to Key Vault directly. If you need K8s Secrets (for tools that require them), the CSI driver can sync them, but the source of truth remains Key Vault."

---

## 4. SITA — CI/CD PIPELINES <a name="4-sita-cicd"></a>

**Resume Line**: *"Built golden-path CI/CD pipelines in Azure DevOps for 13 microservice repositories... supporting 50+ zero-downtime production releases."*

### Q: "What does a 'golden-path pipeline' mean?"

**Answer:**
"A golden-path pipeline is a standardized, opinionated CI/CD template that teams use by default. Instead of every team writing their own pipeline from scratch, I created shared YAML templates that handle:

1. **Build** — Docker build with layer caching, versioned tags (commit SHA + build number)
2. **Test** — Unit tests, code coverage, fail if coverage drops below threshold
3. **Security Scan** — SonarQube for code quality, Snyk for dependency vulnerabilities
4. **Push** — Push to ACR with both `latest` and versioned tags
5. **Deploy** — Helm upgrade to target namespace with environment-specific values
6. **Smoke Test** — Post-deploy health check (HTTP 200 on /health endpoint)

Teams just reference the template and provide their specific variables (service name, namespace, Helm values path). Onboarding a new microservice went from 2 days of pipeline setup to 2 hours."

### Follow-up: "How did you achieve zero-downtime deployments?"

"Three things:
1. **Rolling update strategy** in K8s — `maxSurge: 1, maxUnavailable: 0` ensures at least the current replica count is always running
2. **Readiness probes** — new pods only receive traffic after they pass health checks
3. **Pre-stop hooks** — pods drain connections gracefully before termination

For critical releases, we do a manual approval gate before production. If smoke tests fail post-deploy, the pipeline triggers an automatic rollback to the previous Helm release."

---

## 5. SITA — OBSERVABILITY STACK <a name="5-sita-observability"></a>

**Resume Line**: *"Deployed Prometheus, Grafana, Jaeger, Kiali on AKS for 13 microservices, reducing MTTD by 60%."*

### Q: "How did you measure the 60% MTTD reduction?"

**Answer:**
"Before the observability stack, issue detection relied on user complaints or manual log checking — average detection time was ~25 minutes. After deploying Prometheus with alerting rules and Grafana dashboards, we started catching issues via automated alerts — CPU spikes, pod restarts, error rate increases — within 5-10 minutes. We tracked this over 3 months of incidents and compared before/after median detection times."

### Q: "What Prometheus metrics do you monitor?"

"I set up monitoring at three levels:

**Infrastructure**: `node_cpu_seconds_total`, `node_memory_MemAvailable_bytes`, `kube_pod_container_status_restarts_total`, `kube_deployment_status_replicas_available`

**Application**: Custom metrics exposed via `/metrics` endpoints — request count, request duration histograms (p50/p95/p99), error rate by status code

**Business**: Queue depth, processing latency, API response times per endpoint

I also created recording rules for SLI calculations — like `rate(http_requests_total{status=~'5..'}[5m]) / rate(http_requests_total[5m])` for error rate SLI."

### Q: "What's the difference between Jaeger and Kiali?"

"**Jaeger** = distributed tracing. It traces a single request as it flows through multiple microservices. You can see exactly which service added latency, where errors occurred, and the full call chain. It answers: 'Why was this specific request slow?'

**Kiali** = service mesh visualization. It shows the topology of your services — which services talk to each other, traffic volume, success rates, and mTLS status. It answers: 'What does our service architecture look like in real-time?' It integrates with Istio's metrics."

---

## 6. SITA — AWS SECURITY REMEDIATION <a name="6-sita-aws"></a>

**Resume Line**: *"Remediated 16 AWS security findings (EC2, RDS, S3, CloudFront, IAM, GuardDuty) within 2 sprints, achieving 100% compliance."*

### Q: "What were the 16 findings? Give examples."

**Answer:**
"The findings came from AWS Security Hub and GuardDuty across the Blockchain Sandbox account. Key ones:

- **EC2**: Instances with public IPs that shouldn't have been public, security groups with 0.0.0.0/0 on SSH (port 22)
- **S3**: Buckets with public access enabled, missing server-side encryption, no versioning
- **RDS**: Database instances publicly accessible, missing encryption at rest, no automated backups enabled
- **IAM**: Overly permissive policies (admin access on service accounts), users without MFA, unused access keys older than 90 days
- **CloudFront**: Distributions without WAF, missing HTTPS-only enforcement
- **GuardDuty**: Flagged unusual API calls from an old Lambda function

I triaged by severity, fixed the critical ones first (public access issues), then worked through the highs and mediums. Each fix was documented in a Confluence runbook."

---

## 7. SITA — CONTAINER IMAGE SUPPLY CHAIN <a name="7-sita-images"></a>

**Resume Line**: *"Automated container image uploads from Nexus/SharePoint to ACR, saving 5+ hours/week."*

### Q: "Why were images on SharePoint?"

"Third-party vendors (Indicio for identity agents, Regula for document reading) delivered container images through different channels — some via Nexus registries, some literally as tar files shared via SharePoint. Engineers were manually downloading, loading, tagging, and pushing to our ACR. It was error-prone and ate 5+ hours weekly.

I automated it: a scheduled Azure Pipeline runs PowerShell scripts that pull from vendor sources, verify image digests for integrity, scan with Trivy for vulnerabilities, re-tag with our naming convention, and push to ACR. Only approved images make it into our cluster's allowed registry list."

---

## 8. SITA — COST OPTIMIZATION <a name="8-sita-cost"></a>

**Resume Line**: *"Reduced monthly Azure spend by 20% ($12K+ annual savings)."*

### Q: "How did you achieve 20% cost reduction?"

**Answer:**
"Three main levers:

1. **Resource cleanup** — Found and deleted orphaned disks, unused public IPs, stopped VMs that were still incurring costs, old AKS node pools that weren't in use. I wrote a PowerShell script that runs weekly to flag resources with zero activity.

2. **Right-sizing** — Analyzed Azure Monitor metrics for CPU/memory utilization. Many VMs and AKS node pools were over-provisioned. Downsized Standard_D4s_v3 nodes to Standard_D2s_v3 where utilization was under 30%. Adjusted pod resource requests/limits to pack more efficiently.

3. **Cost dashboards** — Built weekly cost control reports in Grafana pulling from Azure Cost Management APIs, so the team had visibility. When engineers see costs attributed to their namespace, they start caring about resource efficiency."

---

## 9. KNOLDUS — JENKINS SHARED LIBRARIES <a name="9-knoldus-jenkins"></a>

**Resume Line**: *"Built Jenkins Shared Library pipelines serving 10+ project teams, reducing build-to-deploy cycle time by 40% and pipeline code duplication by 70%."*

### Q: "What was in your Jenkins Shared Library?"

**Answer:**
"The Shared Library had reusable Groovy functions organized as:

```
vars/
├── buildDocker.groovy       # Docker build + tag + push
├── runTests.groovy          # Unit/integration test with coverage
├── securityScan.groovy      # SonarQube + Snyk wrapper
├── deployToK8s.groovy       # kubectl/helm deploy to target env
├── notifySlack.groovy       # Slack notifications with build status
└── promoteBuild.groovy      # Promote artifact from staging to prod
```

Teams would write a simple Jenkinsfile like:
```groovy
@Library('platform-shared-lib') _
pipeline {
    stages {
        stage('Build') { steps { buildDocker(imageName: 'my-service') } }
        stage('Test')  { steps { runTests(type: 'unit') } }
        stage('Deploy'){ steps { deployToK8s(env: 'staging') } }
    }
}
```

Before: each team had 200+ line Jenkinsfiles with copy-pasted logic. After: 10-15 line Jenkinsfiles. When I needed to update the Docker build strategy, I changed it once in the library and all 10+ projects got it automatically."

### Follow-up: "How did you reduce cycle time by 40%?"

"Three changes:
1. **Parallelized stages** — tests and security scans run simultaneously instead of sequentially
2. **Docker layer caching** — used `--cache-from` with previous image to skip unchanged layers
3. **Selective testing** — only run integration tests on `main` branch, unit tests on all branches"

---

## 10. KNOLDUS — JFROG TO GITHUB PACKAGES MIGRATION <a name="10-knoldus-jfrog"></a>

**Resume Line**: *"Migrated artifact management from JFrog Artifactory to GitHub Packages, saving $15K+ annually."*

### Q: "Why migrate away from JFrog?"

"JFrog Artifactory was costing $15K+/year in licensing, and we were only using it for Docker images and npm packages — basic features. GitHub Packages was included in our GitHub Enterprise license at no additional cost and provided the same functionality for our use case.

I built a migration plan:
1. Inventoried all artifacts in JFrog (200+ Docker images, 50+ npm packages)
2. Wrote PowerShell scripts to re-tag and push Docker images to GitHub Container Registry
3. Updated all CI pipeline references from JFrog URLs to GitHub Packages URLs
4. Ran both registries in parallel for 2 weeks to validate
5. Decommissioned JFrog after confirming all consumers were migrated

The key challenge was updating downstream consumers — some Helm charts and docker-compose files had hardcoded JFrog URLs. I used `grep -r` to find every reference across all repositories."

---

## 11. KNOLDUS — DEVSECOPS & QUALITY GATES <a name="11-knoldus-devsecops"></a>

**Resume Line**: *"Integrated SonarQube quality gates, Snyk/Mend dependency scanning, reducing security vulnerabilities reaching production by 90%."*

### Q: "How does your security scanning pipeline work?"

**Answer:**
"Every PR triggers:
1. **SonarQube** — static analysis for code smells, bugs, and security hotspots. Quality gate: no new critical/blocker issues, coverage > 70%
2. **Snyk** — scans `package.json`/`requirements.txt`/`pom.xml` for known CVEs in dependencies. Fails the build on HIGH severity
3. **Mend (WhiteSource)** — license compliance scanning. Flags GPL-licensed dependencies in commercial projects

If any gate fails, the PR cannot be merged. I configured SonarQube's quality gate to be the merge check in Bitbucket.

The 90% reduction: before integration, we'd discover vulnerabilities during quarterly security audits. After, we catch them at PR time — before they ever reach a deployable branch."

---

## 12. KNOLDUS — ANSIBLE <a name="12-knoldus-ansible"></a>

**Resume Line**: *"Ansible-based configuration management across 30+ instances, ensuring environment parity and drift detection."*

### Q: "What did you use Ansible for?"

"Server provisioning and configuration consistency. We had 30+ EC2 instances across dev, staging, and production. Ansible playbooks handled:

- **Base configuration**: NTP, SSH hardening, firewall rules, log rotation
- **Application dependencies**: Java/Node/Python versions, system packages
- **Monitoring agents**: Installing and configuring Prometheus node_exporter, log shippers
- **Environment-specific config**: Database endpoints, API URLs, feature flags

I ran Ansible in 'check mode' weekly to detect configuration drift — if someone manually changed a server, the report would flag it. This caught several instances where engineers had SSH'd in and installed packages manually, which would have caused issues in production."

---

## 13. REOMNIFY — DATA ENGINEERING <a name="13-reomnify"></a>

**Resume Line**: *"Python-based data extraction and ETL pipelines using REST APIs and Selenium, processing 10K+ records daily into PostgreSQL."*

### Q: "Tell me about this role."

"This was my first professional role — a 6-month internship during college. I built data pipelines that:

1. **Extracted** data from real estate websites using Selenium for dynamic pages and REST APIs for structured endpoints
2. **Transformed** the raw data — cleaning, deduplication, normalization
3. **Loaded** into PostgreSQL for the analytics team

It processed about 10K property records daily. This is where I learned Python fundamentals, database operations, and the importance of error handling in data pipelines (websites change their HTML structure constantly).

While it's different from my current DevOps work, it gave me strong Python skills that I still use for automation scripts."

---

## 14. CERTIFICATIONS DEEP-DIVE <a name="14-certs"></a>

### Q: "Tell me about your Google Cloud DevOps Engineer certification."

"It covers CI/CD design, SRE principles (SLI/SLO/SLA, error budgets), incident management, monitoring with Cloud Operations Suite, and infrastructure automation. Even though I primarily work on Azure, the SRE concepts — toil reduction, reliability measurement, blameless postmortems — are cloud-agnostic and directly applicable."

### Q: "Azure Network Engineer Associate — what does that cover?"

"It covers designing and implementing Azure networking — VNets, subnets, NSGs, Azure Firewall, Application Gateway, VPN gateways, ExpressRoute, Private Link, and DNS. In my current role, I use this daily when setting up AKS networking, Application Gateway ingress, and Private DNS zones for internal service communication."

---

## 15. ACHIEVEMENTS & AWARDS <a name="15-awards"></a>

### Q: "Tell me about your SITA Bravo Awards."

"I received two awards in my first 6 months at SITA:

**'Do it Together, Step up for the Customer'** — For a deployment that involved coordinating across 3 time zones (India, UK, Australia). The client needed a production release within 48 hours due to a regulatory deadline. I worked extended hours coordinating with teams, resolved Helm chart conflicts, and delivered the release on time.

**'Do it Together, Dare to Grow'** — For taking ownership of the entire Terraform and IaC initiative when no one else on the team had Terraform experience. I designed the module structure, trained the team, and got us from 0% to 95% IaC coverage in 4 months."

### Q: "Tell me about your open-source contribution."

"I contributed PR #232 to Microsoft's Azure Verified Terraform Modules — specifically the Key Vault module (`terraform-azurerm-avm-res-keyvault-vault`). While using the module in production, I found an issue with the access policy configuration and submitted a fix. It was reviewed and merged by Microsoft's team. It showed me the value of contributing upstream — fixing the module benefits everyone who uses it."

---

## 16. WHY ARE YOU LOOKING TO LEAVE / WHY THIS COMPANY? <a name="16-why-leave"></a>

### Q: "Why are you looking to leave SITA?"

**Answer (honest but professional):**
"SITA has been a great experience — I've built their IDP from scratch and learned a lot. But the platform work is maturing now and I'm looking for a company where platform engineering is a core business differentiator, not just a support function. I want to work in a product-driven engineering culture where I can build developer-facing platforms at larger scale."

### Q: "Why [Company Name]?"

**Adapt per company. Template:**
"[Company] is building [specific product/service] at massive scale, and the platform engineering team directly impacts developer productivity. I'm excited about [specific tech from JD — e.g., 'your Kubernetes platform serving hundreds of developers' or 'your approach to SRE with error budgets']. My experience building IDPs, automating infrastructure, and deploying observability stacks directly maps to what you're doing."

---

## 17. TRICKY RESUME QUESTIONS & HOW TO HANDLE <a name="17-tricky"></a>

### Q: "Your title says 'Infrastructure Engineer' but you're applying for a Platform/DevOps/SRE role?"

"My title at SITA is Infrastructure Engineer, but my actual work is platform engineering — I build the internal developer platform, design golden-path CI/CD workflows, manage the observability stack, and enable other teams to self-serve infrastructure. The work aligns directly with Platform Engineer / SRE responsibilities."

### Q: "You've only been at SITA for ~1.5 years. Why leaving so soon?"

"I've delivered significant impact in that time — 11 Terraform modules, secrets migration, observability stack, 50+ production releases. I'm proud of what I've built, but I've reached a point where the foundational platform work is done and I'm looking for a larger-scale challenge. I'm not leaving because of dissatisfaction — I'm leaving because I'm ready for the next level."

### Q: "I see you were at Knoldus for 3 years — why did you leave?"

"Knoldus was acquired by NashTech Global, and the team structure and focus shifted. I wanted to move closer to production infrastructure work rather than consulting, which led me to SITA where I could own the infrastructure end-to-end."

### Q: "Your first job was Data Engineering at Reomnify — how did you switch to DevOps?"

"During my internship at Reomnify, I built Python pipelines and got interested in how they were deployed and managed. I started learning Docker, CI/CD, and cloud services on my own — built projects, got certified (Google Cloud DevOps Engineer), and joined Knoldus specifically for their DevOps practice. The data engineering background actually helps — I'm comfortable with Python, databases, and understanding data flows through systems."

### Q: "Your B.Tech is from ABESIT — not a top-tier college. How do you compensate?"

"I let my work speak for itself. I've been consistently promoted, earned 2 awards at SITA in my first 6 months, contributed to Microsoft's open-source Terraform modules, and have 4 certifications. I believe in continuous learning — my YouTube channel (@DSeDevOps) and GitHub contributions show that."

### Q: "Why haven't you used Go/Buildkite/Splunk [technology from JD]?"

"I haven't used [tool] professionally yet, but the underlying concepts directly map to what I know. [Then bridge]:
- **Go** → 'I'm proficient in Python for DevOps tooling and actively learning Go for its advantages in the K8s ecosystem.'
- **Buildkite** → 'I've built extensive CI/CD with Jenkins and Azure DevOps. Buildkite's hybrid agent model is conceptually similar.'
- **Splunk** → 'I've built production monitoring with Prometheus, Grafana, and ELK. Splunk's SPL is similar to the query patterns I use.'

The key skill — correlating logs, metrics, and traces to diagnose production issues — transfers regardless of the specific tool."

### Q: "What's your biggest failure?"

"Early at Knoldus, I pushed a Terraform change that accidentally destroyed a staging database because I didn't use `prevent_destroy` lifecycle rules. Data was recoverable from backups, but it cost us 4 hours of downtime. After that, I implemented:
1. `prevent_destroy` on all stateful resources
2. Terraform plan review as a mandatory PR check
3. State file backups before any apply

It taught me that infrastructure automation needs the same review discipline as application code."
