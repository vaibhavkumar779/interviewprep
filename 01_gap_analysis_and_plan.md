# Ciena DevOps Engineer Interview - Prep Plan
## Interview: [DATE], [TIME] | Interviewer: [INTERVIEWER]

---

## YOUR SKILL vs JD ALIGNMENT

| JD Requirement | Your Level | Gap | Priority |
|---|---|---|---|
| Jenkins, CI/CD pipelines | Declarative only | Scripted pipelines, shared libs, Groovy | CRITICAL |
| Strong Git knowledge | Basics + rebase | cherry-pick, bisect, stash, hooks, branching strategies | CRITICAL |
| Python scripting | Basic + REST APIs | subprocess, os, OOP, unit testing | HIGH |
| Go/Yocto build environments | None | Need awareness-level understanding | HIGH |
| Linux fundamentals | Basic navigation only | grep/awk/sed, process mgmt, networking, systemctl | CRITICAL |
| Docker | Good (basic + multi-stage) | entrypoint vs cmd, build args, security practices | LOW |
| Kubernetes | Good (modify templates) | Write from scratch confidently | MEDIUM |
| Azure DevOps | Strong | Already solid | - |
| Gerrit, Google Repo | None | Need awareness-level understanding | MEDIUM |
| Bitbucket | Used | Already covered | - |
| DevSecOps | SonarQube, Snyk | Already covered | - |

---

## 36-HOUR STUDY PLAN

### Phase 1: TODAY (Sat evening, ~6 hours)
1. **Linux commands deep dive** (2 hrs) - grep, awk, sed, find, process mgmt, networking
2. **Git advanced ops** (1.5 hrs) - cherry-pick, bisect, stash, hooks, branching strategies
3. **Jenkins scripted pipelines + shared libraries** (1.5 hrs)
4. **Go/Yocto awareness** (1 hr) - what they are, key concepts, vocabulary

### Phase 2: TOMORROW MORNING (Sun, ~6 hours)
5. **Python subprocess, os, sys** (1 hr)
6. **Practice writing K8s manifests from scratch** (1 hr)
7. **Gerrit/Google Repo concepts** (30 min)
8. **Practice coding scenarios** (2.5 hrs) - Jenkins pipeline, Dockerfile, K8s manifests, shell scripts
9. **Review written Q&A** (1 hr)

### Phase 3: MONDAY MORNING before interview (~2 hours)
10. **Quick review of all notes** (1 hr)
11. **Ciena company research** (30 min) - Optical Networks, products, recent news
12. **Mock dry run** - explain your work at my current company in STAR format (30 min)

---

## ABOUT CIENA (Research this!)
- Ciena is a **networking systems, services, and software company**
- Specializes in **Optical Networking** (fiber optic networking equipment)
- Products: WaveLogic (coherent optics), MCP (Manage, Control, Plan platform)
- The ON (Optical Networks) team builds software for optical networking devices
- **Embedded software** context (interviewer is from embedded SW) - firmware, device OS, build systems
- This is why **Go/Yocto** matters - Yocto is used to build embedded Linux for network devices

---

## HOW TO POSITION YOURSELF

### Strengths to emphasize:
- Azure DevOps + CI/CD pipeline experience (directly transferable to Jenkins)
- Docker + K8s in production
- Infrastructure automation (Terraform, Ansible)
- Monitoring (Prometheus, Grafana) - shows operational maturity
- DevSecOps (SonarQube, Snyk)
- Working in large distributed teams

### For gaps, be honest but frame positively:
- "I haven't used Gerrit/Google Repo directly, but I understand the code review workflow concept and multi-repo management challenges from working with Azure DevOps and Bitbucket"
- "My Jenkins experience is primarily Declarative pipelines, but I'm actively expanding into Scripted pipelines and shared libraries"
- "I haven't worked with Yocto/Go build environments, but I have experience with build systems and am a quick learner in new toolchains"
