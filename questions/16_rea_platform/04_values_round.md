# Round 4: Values Round — REA Group

> REA evaluates cultural fit based on their core values.
> Use STAR format (Situation, Task, Action, Result) for behavioral answers.

---

## REA'S CORE VALUES

Based on public information from REA Group:

1. **Customer First** — Everything starts with the customer experience
2. **Own It** — Take ownership and accountability
3. **Dare to be Different** — Innovate, challenge the status quo
4. **Stronger Together** — Collaboration across diverse teams
5. **Keep it Real** — Transparency, honesty, authenticity

---

## BEHAVIORAL QUESTIONS & ANSWERS (STAR Format)

### Q: Tell me about yourself and why you want to join REA.

**A:**
I'm a Platform Engineer with 4.5 years of experience building Kubernetes platforms, CI/CD pipelines, and cloud infrastructure. At SITA, I manage 13 microservices on AKS with full observability and security. Before that at Knoldus/NashTech, I built reusable Jenkins pipelines serving 10+ teams.

**Why REA**: Three things excite me:
1. **Scale** — REA serves millions of property seekers daily. Building platform infrastructure at that scale is exactly the challenge I'm looking for.
2. **Tech culture** — REA's "day one" mindset resonates with me. I run a YouTube channel (DSeDevOps) where I share DevOps content, so I'm genuinely passionate about this space.
3. **India Tech Center** — Being part of building something from scratch at the new Gurugram center is a rare opportunity.

---

### Q: Tell me about a time you took ownership of a difficult problem. (Own It)

**A: Secrets Migration at SITA**
- **Situation**: Our 4 product Helm charts stored application secrets as plaintext values in Git repositories — a critical security risk flagged during audit.
- **Task**: I needed to migrate all secrets to Azure Key Vault without disrupting any running services across 5 environments.
- **Action**: I designed the migration approach using CSI Secret Store Driver, created a rollout plan (dev → QA → preprod → prod), wrote the Helm chart modifications, tested thoroughly in each environment, and coordinated with 3 product teams across time zones for the cutover windows.
- **Result**: Eliminated 100% of plaintext secrets from Git. Zero downtime during migration. This became the standard pattern for all new services.

---

### Q: Tell me about a time you improved developer experience. (Customer First / Platform as Product)

**A: Jenkins Shared Libraries at Knoldus**
- **Situation**: 10+ project teams each had their own Jenkins pipeline code — duplicated, inconsistent, and hard to maintain. Developers spent hours debugging pipeline failures instead of writing features.
- **Task**: Create a standardized, reusable pipeline framework that teams could adopt with minimal effort.
- **Action**: I built Jenkins Shared Libraries with golden-path templates — a single `@Library` import gave teams a complete CI/CD pipeline with build, test, security scanning (SonarQube, Snyk), and deployment stages. I documented it, ran training sessions, and iterated based on team feedback.
- **Result**: 70% reduction in pipeline duplication. Build-to-deploy time reduced by 40%. Teams could onboard a new project in 30 minutes instead of 2 days. This was my first experience treating platform as a product.

---

### Q: Tell me about a time you collaborated with a cross-functional team. (Stronger Together)

**A: Multi-timezone Deployments at SITA**
- **Situation**: SITA's products serve airports globally. Deployments required coordination across India, Europe, and North America teams — product managers, QA, security, and ops.
- **Task**: Orchestrate a complex release of Digital Identity services across 5 environments with different stakeholders owning different components.
- **Action**: I set up a deployment calendar, created runbooks for each environment, established communication channels (Teams war room), and implemented change management process with rollback criteria. During deployments, I served as the release coordinator — running pre-checks, deploying, monitoring, and communicating status in real-time.
- **Result**: Successfully delivered 50+ production deployments with zero customer-impacting incidents. Recognized with 2x Bravo Awards for this work.

---

### Q: Tell me about a time you innovated or challenged the status quo. (Dare to be Different)

**A: JFrog to GitHub Packages Migration at Knoldus**
- **Situation**: We were paying $15K+ annually for JFrog Artifactory licenses. I noticed we already had GitHub Enterprise and weren't using GitHub Packages.
- **Task**: Evaluate whether we could replace JFrog with GitHub Packages and execute the migration.
- **Action**: I built a proof of concept, wrote PowerShell automation to migrate all existing artifacts, set up Azure Pipelines integration, and presented the cost-benefit analysis to management. I also wrote a migration guide so other teams could self-serve.
- **Result**: Eliminated $15K+ in annual licensing costs. The automation I built became the standard migration tool across the organization. This taught me that sometimes the best solution is eliminating a tool, not adding one.

---

### Q: Tell me about a time you had to be transparent about a mistake or challenge. (Keep it Real)

**A: AWS Security Findings**
- **Situation**: During a security audit at SITA, 16 findings were flagged across our AWS Blockchain Sandbox account — IAM overprivileged roles, S3 buckets without encryption, security groups with 0.0.0.0/0.
- **Task**: I was responsible for the account and needed to address all findings.
- **Action**: Instead of downplaying the issues, I proactively documented every finding, created a remediation plan with timelines, and presented it to the security team. I was transparent that some issues existed because we had inherited the account setup and hadn't reviewed it properly. I then remediated all 16 findings systematically — tightened IAM policies, enabled encryption, restricted security groups, configured GuardDuty.
- **Result**: Achieved 100% compliance. The security team appreciated the transparency and proactive approach. I then created a security baseline checklist for all future AWS accounts.

---

### Q: How do you handle disagreements with team members?

**A:**
I focus on **data, not opinions**. When I disagree:
1. I listen fully to understand their perspective
2. I present my reasoning with data/evidence (metrics, benchmarks, documentation)
3. I suggest we try both approaches in a small scope if possible (A/B test, POC)
4. If the team decides differently, I commit fully to the decision

Example: At SITA, a colleague wanted to use Vault for secrets while I advocated for Azure Key Vault with CSI driver (simpler, native to our stack). I presented a comparison (operational overhead, cost, integration effort). The team chose Key Vault, but I documented Vault as a future option if we go multi-cloud.

---

### Q: Where do you see yourself in 3-5 years?

**A:**
I want to grow into a **Staff/Principal Platform Engineer** role:
- **Short term (1-2 years)**: Deepen expertise in Go, AWS at scale, chaos engineering. Become the go-to person for platform reliability.
- **Medium term (3-5 years)**: Lead platform architecture decisions, mentor junior engineers, drive platform strategy. Contribute to open-source platform tooling.
- **Continuous**: Keep sharing knowledge through my YouTube channel and community contributions.

---

### Q: What questions do you have for us?

**Prepare 3-5 thoughtful questions:**

1. "What does the platform team's tech stack look like today? Which parts are you most focused on evolving?"
2. "How do you measure platform success — is it DORA metrics, developer satisfaction surveys, or something else?"
3. "What's the relationship between the India Tech Center and Melbourne teams — how autonomous is the India platform team?"
4. "What's the biggest platform challenge REA is facing right now that this role would help solve?"
5. "How does REA approach chaos engineering and reliability testing in production?"

---

## TIPS FOR VALUES ROUND

- **Be authentic** — REA values "Keep it Real"
- **Show passion** — mention your YouTube channel, open-source PR to Azure Verified Modules
- **Highlight collaboration** — they value "Stronger Together"
- **Show ownership** — every story should show you driving something end-to-end
- **Connect to REA** — tie your answers to how you'd contribute at REA
- **Ask smart questions** — shows genuine interest in the company
