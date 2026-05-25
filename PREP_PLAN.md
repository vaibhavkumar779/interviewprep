# Interview Prep Plan — DevOps Engineer

## Interview Format
- **Written round**: Basic → Advanced questions across all DevOps topics
- **Coding round**: Scripting, K8s manifests, Dockerfiles, Jenkins pipelines, Azure pipelines, DSA

---

## Priority Matrix (Based on Your Gaps)

| Priority | Topic | Your Level | Action |
|----------|-------|-----------|--------|
| 🔴 CRITICAL | Linux Commands | Weak | Drill top 50 commands |
| 🔴 CRITICAL | Jenkins (Scripted, Shared Libs) | Partial | Focus scripted syntax |
| 🔴 CRITICAL | Yocto/Embedded | Zero | Learn key concepts |
| 🟡 HIGH | Shell Scripting | Moderate | Write 3 scripts from scratch |
| 🟡 HIGH | Git Advanced | Moderate | Rebase, cherry-pick, bisect |
| 🟡 HIGH | Docker + K8s coding | Good | Write manifests from memory |
| 🟢 SOLID | Azure DevOps, Terraform, CI/CD | Good | Skim answers only |
| 🟢 SOLID | Python, Monitoring, Networking | Good | Skim answers only |

---

## Study Sprint Plan

### SESSION 1: (~4 hours) — Critical Gaps
**Focus: Linux + Shell Scripting**

| Time | Task | File |
|------|------|------|
| 1.5 hr | Read Linux answers — file ops, grep, awk, sed, find, xargs, permissions, process mgmt | `07_linux/answers.md` (Q1-80) |
| 15 min | BREAK | |
| 1.25 hr | Read Linux answers — networking, systemd, storage, troubleshooting | `07_linux/answers.md` (Q81-162) |
| 45 min | Read Shell Scripting answers — conditionals, loops, functions, error handling, real-world | `08_shell_scripting/answers.md` (all 70 Qs) |

**After session**: Write down 10 commands you struggled with.

---

### SESSION 2: (~4.5 hours) — Critical Gaps
**Focus: Jenkins + Git + Yocto**

| Time | Task | File |
|------|------|------|
| 1.25 hr | Jenkins answers — Declarative vs Scripted, shared libs, Jenkinsfile patterns, multi-branch | `02_jenkins/answers.md` |
| 45 min | Git answers — rebase, cherry-pick, bisect, stash, reflog, merge strategies | `03_git/answers.md` |
| 15 min | BREAK | |
| 1 hr | Yocto/Embedded — BitBake, recipes, layers, BSP, sstate cache, CI for embedded | `13_go_yocto_embedded/answers.md` (Q16-52) |
| 45 min | Yocto interview questions — CI pipeline, optical networking context | `13_go_yocto_embedded/answers.md` (Q53-70) |
| 30 min | Go basics skim (goroutines, channels, cross-compile, why Go for DevOps) | `13_go_yocto_embedded/answers.md` (Q1-15) |

---

### SESSION 3: (~3.5 hours) — Docker + K8s + Coding
**Focus: Docker + K8s + Coding Practice**

| Time | Task | File |
|------|------|------|
| 45 min | Docker answers — Dockerfile best practices, multi-stage, networking, compose, security | `04_docker/answers.md` (all 80 Qs) |
| 1 hr | K8s answers — Deployments, Services, ConfigMaps, Secrets, Networking, RBAC, Helm, debugging | `05_kubernetes/answers.md` (focus on Q1-70) |
| 15 min | BREAK | |
| 1.5 hr | **CODING DRILL** — write from scratch in blank editor, no reference: | |
| | ✏️ Jenkinsfile (multi-stage, parallel, post actions) | |
| | ✏️ Dockerfile (multi-stage, non-root, best practices) | |
| | ✏️ K8s Deployment + Service + Ingress YAML | |
| | ✏️ Shell script (log parser with grep/awk/sed) | |

---

### SESSION 4: (~3.5 hours) — Speed Run
**Focus: Remaining topics + Scenarios**

| Time | Task | File |
|------|------|------|
| 30 min | CI/CD General — skim answers | `01_cicd_devops/answers.md` |
| 30 min | Azure DevOps — skim pipeline YAML, tasks, comparison | `09_azure_devops/answers.md` |
| 20 min | Terraform/Ansible — skim state, modules, playbooks | `10_iac_terraform_ansible/answers.md` |
| 20 min | Python — skim subprocess, os, requests, OOP | `06_python/answers.md` |
| 10 min | BREAK | |
| 20 min | Monitoring — Prometheus, Grafana, alerting, PromQL | `11_monitoring_observability/answers.md` |
| 15 min | DevSecOps — SAST, SCA, container security, secrets | `12_devsecops/answers.md` |
| 15 min | Networking — OSI, DNS, HTTP, TLS basics | `14_networking/answers.md` |
| 50 min | **Scenarios & Behavioral** — STAR stories, "tell me about yourself" | `15_scenarios_behavioral/answers.md` |

---

### SESSION 5: (~3.5 hours) — Final Review
**Focus: Final review + warm-up**

| Time | Task |
|------|------|
| 30 min | Re-read Linux weak spots (commands you wrote down) |
| 30 min | Re-read Yocto key concepts + optical networking context |
| 30 min | Quick coding warm-up: write a Jenkinsfile + Dockerfile from memory |
| 30 min | Review Jenkins Scripted pipeline syntax + shared library structure |
| 15 min | BREAK |
| 30 min | Re-read Scenarios/Behavioral — your STAR stories, "tell me about yourself" |
| 15 min | Review your questions to ask the interviewer |
| 30 min | Light skim of any remaining weak topics. **Stop studying.** |

---

## Coding Round Quick Reference

Things to write from memory on interview day:

| Item | Practice Until Automatic |
|------|------------------------|
| Jenkinsfile | Declarative pipeline with stages, parallel, post |
| Dockerfile | Multi-stage, non-root user, .dockerignore |
| K8s Deployment | replicas, image, ports, resources, probes |
| K8s Service | ClusterIP, NodePort, LoadBalancer |
| Shell script | Functions, error handling, grep/awk/sed |
| Azure pipeline | trigger, pool, stages, tasks |
| Terraform | provider, resource, variable, output, module |
| Python | subprocess.run, os.path, requests, json |

---

## Questions to Ask Interviewer

1. "What does a typical day look like for the DevOps engineer on the team?"
2. "What build system and CI pipeline does the team currently use?"
3. "How does the team handle testing for embedded firmware?"
4. "What are the biggest DevOps challenges the team faces today?"
5. "What does the onboarding process look like for a new DevOps engineer?"

---

## Key Stories to Prepare (STAR Method)

1. **CI/CD improvement**: Pipeline optimization, build time reduction
2. **Incident response**: Production issue you debugged and resolved
3. **Automation win**: Manual process you automated
4. **Collaboration**: Working across teams (dev + ops)
5. **Learning challenge**: Quickly ramping up on unfamiliar technology
