# CI/CD & DevOps - ANSWERS

---

## Basics Answers

**1. What is CI/CD?**
- **CI (Continuous Integration)**: Developers merge code to a shared branch frequently (multiple times/day). Each merge triggers automated build and test. Purpose: catch integration bugs early, keep codebase always buildable.
- **CD (Continuous Delivery)**: After CI passes, the artifact is always in a deployable state. Requires manual approval to deploy to production.
- **CD (Continuous Deployment)**: Every change that passes all automated stages is deployed to production automatically. No manual gate.

**2. Difference between Continuous Delivery and Continuous Deployment?**
- Delivery = manual approval gate before production. Deployment = fully automated to production. Delivery is more common in regulated industries.

**3. What is DevOps? Is it a role, a culture, or a set of tools?**
- DevOps is primarily a **culture and set of practices** that unifies software development (Dev) and IT operations (Ops). It emphasizes collaboration, automation, continuous feedback, and shared responsibility. Tools enable DevOps but aren't DevOps themselves. The role "DevOps Engineer" is a job title that emerged from this culture.

**4. Key principles of DevOps?**
- Collaboration between Dev and Ops
- Automation of everything (build, test, deploy, infra)
- Continuous improvement (feedback loops)
- Infrastructure as Code
- Monitoring and observability
- Shared ownership and accountability
- Small, frequent releases

**5. DevOps lifecycle phases?**
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → (back to Plan)

**6. What is Infrastructure as Code?**
Managing and provisioning infrastructure through machine-readable configuration files instead of manual processes. Benefits: version controlled, repeatable, auditable, testable, reviewable via PRs.

**7. Configuration Management vs Provisioning?**
- **Provisioning**: Creating infrastructure resources (VMs, networks, storage) — Terraform, CloudFormation
- **Configuration Management**: Configuring existing resources (install packages, manage files, services) — Ansible, Chef, Puppet

**8. Imperative vs Declarative?**
- **Imperative**: You specify exact steps to achieve desired state ("create VM, then install nginx, then start service")
- **Declarative**: You specify the desired end state, system figures out steps ("I want 3 nginx servers running")

**9. What is a build artifact? 5 examples.**
Output produced by a build process. Examples: Docker image, JAR/WAR file, Python wheel/sdist, npm package, compiled binary, zip archive.

**10. What does "shift left" mean?**
Moving activities like testing, security scanning, and quality checks earlier in the development lifecycle (leftward on the timeline). Find bugs and vulnerabilities when they're cheapest to fix.

---

**11-20: Pipeline Concepts**

**11.** A build pipeline compiles code, runs tests, produces artifacts. Typical stages: Checkout → Install Dependencies → Lint → Unit Test → Integration Test → Build Artifact → Publish Artifact.

**12.** A release pipeline takes build artifacts and deploys them. It focuses on environment promotion (dev → staging → prod) with approval gates. Modern CI/CD often combines both.

**13.** Pipeline trigger types: SCM webhook (push/PR), schedule/cron, manual/on-demand, pipeline completion (chain trigger), API call.

**14.** Artifact repository stores build outputs for later deployment. Tools: JFrog Artifactory, Azure Artifacts, GitHub Packages, Nexus, Docker Registry.

**15.** Stage = logical grouping (Build, Test, Deploy). Job = unit of work assigned to an agent within a stage. Step = individual command within a job.

**16.** An agent/runner executes pipeline jobs. Don't run on controller because: security risk, performance bottleneck, single point of failure.

**17.** Variables = non-sensitive config (branch name, version). Secrets = sensitive (passwords, tokens) — must be encrypted, masked in logs, accessed via secure mechanisms.

**18.** Pipeline template = reusable pipeline definition. Avoids duplication across repos/teams, enforces standards, single place to update.

**19.** Self-hosted = your own machines (full control, custom tools, persistent). Cloud-hosted = managed by provider (no maintenance, clean environment each run, limited customization).

**20.** If a step fails, the pipeline typically stops (or continues based on `continueOnError`). Handle with: retry logic, post-failure notifications, conditional steps, manual intervention gates.

---

**21-25: Interview-Style**

**21.** *(Prepare your own story — describe real pipeline you built with Azure DevOps)*

**22.** *(Prepare your own story — most complex pipeline problem, how you diagnosed and fixed it)*

**23.** Single pipeline when: tightly coupled components, small project. Multiple pipelines when: independent services, different languages/teams, parallel development needed.

**24.** Metrics: build duration, build success rate, deployment frequency, lead time (commit to prod), change failure rate, MTTR, test coverage, queue time.

**25.** Database changes in CI/CD: Use migration tools (Flyway, Liquibase, EF Migrations). Run migrations as a pipeline step before app deployment. Always make migrations backward-compatible. Have rollback scripts. Test migrations in staging first.

---

## Intermediate & Advanced Answers

**1-10: Deployment Strategies**

**1.** Blue-Green: Two identical environments. Blue = current production. Green = new version deployed and validated. Traffic switch (DNS or load balancer) makes Green the new production. Rollback = switch back to Blue.

**2.** Canary: Deploy new version to small subset of infrastructure. Route 5% of traffic → monitor → gradually increase → 100%. Decision based on error rates, latency, business metrics.

**3.** Rolling: Replace instances one by one. maxSurge = how many extra pods during update. maxUnavailable = how many pods can be down during update.

**4.** A/B testing: Route users based on attributes (geography, user ID). Measures business metrics (conversion). Canary measures technical metrics (errors, latency).

**5.** Feature flags: Toggle features on/off without deployment. Allows trunk-based dev — deploy code with flag off, enable when ready. Tools: LaunchDarkly, Unleash, Flagsmith.

**6.** Immutable infrastructure: Never modify running servers. Instead, build new image with changes, deploy new instances, destroy old ones. Benefits: consistency, reproducibility, no configuration drift.

**7.** Rollback: Blue-Green = instant DNS/LB switch. Canary = route traffic back to v1. Rolling = `kubectl rollout undo`. Recreate = redeploy old version (slowest).

**8.** Dark launch: Deploy new feature to production but don't expose to users. Test with production traffic/data without user impact.

**9.** Canary over blue-green when: you want gradual validation, can't afford 2x infrastructure, need to test with real production traffic patterns.

**10.** Progressive delivery: Umbrella term for canary, feature flags, A/B testing. Gradually expose changes while monitoring.

**11-20: Advanced CI/CD**

**11.** GitOps: Git repo is the single source of truth for infrastructure and app state. Changes via PRs. An agent (ArgoCD/Flux) in the cluster syncs desired state from Git. Differs from traditional CI/CD by using pull-based deployment.

**12.** ArgoCD watches a Git repo for K8s manifests. When changes detected, it syncs the cluster to match. Provides drift detection, auto-sync, rollback.

**13.** Trunk-based: All devs commit to main (or very short-lived branches <1 day). Requires: feature flags, good test coverage, fast CI. Enables continuous deployment.

**14.** Monorepo CI: Path-based triggers (only build changed services), dependency graph analysis, parallel builds, shared pipeline templates, caching.

**15.** Pipeline-as-code: Pipeline definition in a file (Jenkinsfile, azure-pipelines.yml) committed to repo. Benefits: version controlled, reviewed via PR, auditable, reproducible.

**16.** Diamond dependency: A depends on B and C, both depend on D at different versions. Solution: dependency resolution, lock files, monorepo with single version.

**17.** Flaky tests: Quarantine (run but don't block), auto-retry (1-2x max), track flakiness rate, require owners to fix within SLA, separate pipeline for quarantined tests.

**18.** Artifact promotion: Same artifact moves through environments (build once, deploy many). Rebuilding for each environment risks inconsistency.

**19.** Secrets rotation in CI/CD: Use external secret manager (Vault, Key Vault), implement rotation policy, pipeline fetches secrets at runtime (not baked into config), zero-downtime rotation.

**20.** Push-based: CI tool pushes changes to target (Jenkins deploys to K8s). Pull-based: Agent in target pulls from source (ArgoCD pulls from Git). Pull is more secure — no external access needed.

**21-35: Culture, DORA, Scenarios**

**21.** DORA metrics: Deployment Frequency (how often), Lead Time for Changes (commit to prod), Change Failure Rate (% causing incidents), MTTR (time to restore service).

**22.** Blameless post-mortem: Focus on what happened and how to prevent recurrence, not who to blame. Psychological safety encourages reporting and learning.

**23.** SLI = measurement (e.g., 99.5% of requests < 200ms). SLO = target (e.g., 99.9% availability). SLA = contract with consequences (e.g., refund if below 99.5%).

**24.** Error budget = 100% - SLO. If SLO is 99.9%, error budget is 0.1%. If error budget exhausted, freeze deployments until recovered.

**25.** ChatOps: Managing operations through chat (Slack/Teams) with bot integrations. Deploy, rollback, check status via chat commands. Improves visibility and audit trail.

**26.** Value Stream Mapping: Visualize the entire delivery process, identify waste and bottlenecks, measure wait times vs work times.

**27.** Three Ways: 1) Flow (left to right, dev to ops), 2) Feedback (right to left, fast feedback loops), 3) Continual Learning (experimentation, risk-taking).

**28.** Toil: Manual, repetitive, automatable work that scales linearly with service growth. Reduce via automation, self-service, better tooling.

**29-35:** *(Scenario answers should be personalized based on your experience. Use the principles above to structure your responses.)*
