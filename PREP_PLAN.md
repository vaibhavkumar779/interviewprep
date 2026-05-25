# Interview Prep Plan — Ciena DevOps Engineer ([DATE])

## Interview Format
- **Written round**: Basic → Advanced questions across all DevOps topics
- **Coding round**: Scripting, K8s manifests, Dockerfiles, Jenkins pipelines, Azure pipelines, DSA
- **Interviewer**: [INTERVIEWER]
- **Date**: Tuesday, [DATE]
- **Time**: [TIME] (1 hour)
- **Zoom**: [MEETING LINK]
- **Position**: [REF] DevOps Engineer
- **Prep window**: ~39 hours (Sun 8 PM → Tue 11 AM)
- **Usable study time**: ~14-15 hours (sleep, meals, breaks excluded)

---

## Priority Matrix (Based on Your Gaps)

| Priority | Topic | Your Level | Action |
|----------|-------|-----------|--------|
| 🔴 CRITICAL | Linux Commands | Weak | Drill top 50 commands |
| 🔴 CRITICAL | Jenkins (Scripted, Shared Libs) | Partial | Focus scripted syntax |
| 🔴 CRITICAL | Yocto/Embedded | Zero | Learn key concepts + Ciena context |
| 🟡 HIGH | Shell Scripting | Moderate | Write 3 scripts from scratch |
| 🟡 HIGH | Git Advanced | Moderate | Rebase, cherry-pick, bisect |
| 🟡 HIGH | Docker + K8s coding | Good | Write manifests from memory |
| 🟢 SOLID | Azure DevOps, Terraform, CI/CD | Good | Skim answers only |
| 🟢 SOLID | Python, Monitoring, Networking | Good | Skim answers only |

---

## 38-Hour Sprint Plan (Sat 8 PM → Mon 11 AM)

### SESSION 1: Sunday Night (8 PM – 12 AM) — 4 hours
**Focus: Critical gaps — Linux + Shell Scripting**

| Time | Task | File |
|------|------|------|
| 8:00–9:30 | Read Linux answers — focus on file ops, grep, awk, sed, find, xargs, permissions, process mgmt | `07_linux/answers.md` (Q1-80) |
| 9:30–10:00 | BREAK — walk, hydrate | |
| 10:00–11:15 | Read Linux answers — networking, systemd, storage, troubleshooting | `07_linux/answers.md` (Q81-162) |
| 11:15–12:00 | Read Shell Scripting answers — conditionals, loops, functions, error handling, real-world | `08_shell_scripting/answers.md` (all 70 Qs) |

**Before bed**: Write down 10 commands you struggled with. Sleep by 12:30 AM.

---

### SESSION 2: Monday Morning (8 AM – 12:30 PM) — 4.5 hours
**Focus: Critical gaps — Jenkins + Git + Yocto**

| Time | Task | File |
|------|------|------|
| 8:00–9:15 | Jenkins answers — Declarative vs Scripted, shared libs, Jenkinsfile patterns, multi-branch | `01_cicd_jenkins/answers.md` |
| 9:15–10:00 | Git answers — rebase, cherry-pick, bisect, stash, reflog, merge strategies | `02_git/answers.md` |
| 10:00–10:15 | BREAK | |
| 10:15–11:15 | Yocto/Embedded — BitBake, recipes, layers, BSP, sstate cache, CI for embedded | `13_go_yocto_embedded/answers.md` (Q16-52) |
| 11:15–12:00 | Yocto interview questions — ramp-up plan, CI pipeline, Ciena context, optical networking | `13_go_yocto_embedded/answers.md` (Q53-70) |
| 12:00–12:30 | Go basics skim (goroutines, channels, cross-compile, why Go for DevOps) | `13_go_yocto_embedded/answers.md` (Q1-15) |

---

### SESSION 3: Monday Afternoon (2 PM – 5:30 PM) — 3.5 hours
**Focus: Docker + K8s + Coding Practice**

| Time | Task | File |
|------|------|------|
| 2:00–2:45 | Docker answers — Dockerfile best practices, multi-stage, networking, compose, security | `04_docker/answers.md` (all 80 Qs) |
| 2:45–3:45 | K8s answers — Deployments, Services, ConfigMaps, Secrets, Networking, RBAC, Helm, debugging | `05_kubernetes/answers.md` (focus on Q1-70) |
| 3:45–4:00 | BREAK | |
| 4:00–5:30 | **CODING DRILL** — write from scratch in blank editor, no reference: | |
| | ✏️ Jenkinsfile (multi-stage, parallel, post actions) | |
| | ✏️ Dockerfile (multi-stage, non-root, best practices) | |
| | ✏️ K8s Deployment + Service + Ingress YAML | |
| | ✏️ Shell script (log parser with grep/awk/sed) | |

---

### SESSION 4: Monday Evening (7 PM – 10:30 PM) — 3.5 hours
**Focus: Remaining topics speed-run + Scenarios**

| Time | Task | File |
|------|------|------|
| 7:00–7:30 | CI/CD General — skim answers | `03_cicd_general/answers.md` |
| 7:30–8:00 | Azure DevOps — skim pipeline YAML, tasks, comparison | `09_azure_devops/answers.md` |
| 8:00–8:20 | Terraform/Ansible — skim state, modules, playbooks | `10_iac_terraform_ansible/answers.md` |
| 8:20–8:40 | Python — skim subprocess, os, requests, OOP | `06_python/answers.md` |
| 8:40–8:50 | BREAK | |
| 8:50–9:10 | Monitoring — Prometheus, Grafana, alerting, PromQL | `11_monitoring_observability/answers.md` |
| 9:10–9:25 | DevSecOps — SAST, SCA, container security, secrets | `12_devsecops/answers.md` |
| 9:25–9:40 | Networking — OSI, DNS, HTTP, TLS basics | `14_networking/answers.md` |
| 9:40–10:30 | **Scenarios & Behavioral** — STAR stories, "tell me about yourself", why Ciena, gap story | `15_scenarios_behavioral/answers.md` |

**Before bed**: Review the 10 commands from last night. Sleep by 11 PM.

---

### SESSION 5: Tuesday Morning (7 AM – 10:30 AM) — 3.5 hours
**Focus: Final review + warm-up**

| Time | Task |
|------|------|
| 7:00–7:30 | Re-read Linux weak spots (the 10 commands you wrote down) |
| 7:30–8:00 | Re-read Yocto key concepts + Ciena optical networking context |
| 8:00–8:30 | Quick coding warm-up: write a Jenkinsfile + Dockerfile from memory |
| 8:30–9:00 | Review Jenkins Scripted pipeline syntax + shared library structure |
| 9:00–9:15 | BREAK — shower, dress, coffee |
| 9:15–9:45 | Re-read Scenarios/Behavioral — your STAR stories, "tell me about yourself" |
| 9:45–10:00 | Review your 3 questions to ask the interviewer |
| 10:00–10:30 | Light skim of any remaining weak topics. **Stop studying by 10:30.** |
| 10:30–11:00 | Relax, deep breaths, join call early |

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

1. "What does a typical day look like for the DevOps engineer on the ON team?"
2. "What build system and CI pipeline does the team currently use?"
3. "How does the team handle testing for embedded firmware?"

---

## Key Stories to Prepare (STAR Method)

1. **CI/CD improvement**: Pipeline optimization, build time reduction
2. **Incident response**: Production issue you debugged and resolved
3. **Automation win**: Manual process you automated
4. **Collaboration**: Working across teams (dev + ops)
5. **Learning challenge**: Quickly ramping up on unfamiliar technology
