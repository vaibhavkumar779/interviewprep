# Ciena & The Position — INTERVIEW READY REFERENCE

---

# PART 1: ABOUT CIENA

---

## Company Overview

**1. What is Ciena?**
Ciena Corporation is an American **optical networking systems, services, and software company**. Founded in 1992, headquartered in **Hanover, Maryland, USA**. Listed on NYSE (CIEN), S&P 500 component.

```
Ciena at a Glance:

  Founded:     1992 (by David Huber)
  HQ:          Hanover, Maryland, USA
  CEO:         Gary Smith (since 2001)
  Revenue:     ~$4.8 billion (FY 2025)
  Employees:   ~9,080 (2025)
  NYSE:        CIEN (S&P 500)
  Industry:    Optical Networking / Telecom Equipment / Software

  India Presence:
  ┌──────────────────────────────────────────────┐
  │  Gurgaon campus (opened 2006)                │
  │  ~1,500+ employees (20% of global workforce) │
  │  Focus: R&D + Manufacturing (since 2018)     │
  │  India = fastest growing market globally      │
  └──────────────────────────────────────────────┘

  Key Customers: AT&T, Verizon, Deutsche Telekom, KT Corp,
                 Bharti Airtel, Jio, Vodafone Idea, Sify
```

**2. What does Ciena do?**
Ciena designs, manufactures, and sells networking equipment, software, and services for **telecom operators, cable companies, cloud providers, and governments**. They enable high-capacity data transport over fiber optic networks.

**3. Why is Ciena important?**
Ciena is a **vital player in optical connectivity** — the backbone infrastructure that carries the internet. Every time you stream, make a call, or use cloud services, the data likely travels through Ciena equipment at some point. They power undersea cables, metro networks, and long-haul fiber links globally.

---

## Key Products & Technologies

**4. What are Ciena's main products?**

```
Ciena Product Portfolio:

  ┌─────────────────────────────────────────────────────────────────┐
  │                    CIENA PRODUCTS                                │
  ├──────────────────┬──────────────────┬────────────────────────────┤
  │  HARDWARE        │  SOFTWARE        │  SERVICES                  │
  │                  │                  │                            │
  │  WaveLogic       │  Blue Planet     │  Professional services    │
  │  (coherent       │  (network        │  (design, deploy,        │
  │   optics modem)  │   automation     │   optimize networks)     │
  │                  │   platform)      │                            │
  │  6500 Series     │  MCP             │  Managed services         │
  │  (packet-optical │  (Manage,        │  (24/7 NOC support)      │
  │   platforms)     │   Control, Plan) │                            │
  │                  │                  │                            │
  │  5100/5200       │  Navigator NMS   │  Consulting &             │
  │  (converged      │  (network mgmt)  │  training                │
  │   packet)        │                  │                            │
  │                  │                  │                            │
  │  Routers         │  Analytics &     │                            │
  │  (5164/5166/     │  AI/ML for       │                            │
  │   5168 for 5G)   │  predictive ops  │                            │
  └──────────────────┴──────────────────┴────────────────────────────┘
```

**5. What is WaveLogic?**
Ciena's **coherent optics modem technology** — the "brain" inside optical networking gear. WaveLogic determines how much data you can push through a fiber optic link.
- **WaveLogic 5 Nano/Extreme** — current generation, supports up to **800G** per wavelength
- **WaveLogic 6** — next-gen, pushing toward 1.6T
- WaveLogic is embedded in Ciena's transport platforms and also sold as pluggable modules

**6. What is Blue Planet?**
Ciena's **intelligent network automation software platform**. Uses ML/AI to:
- Automate network provisioning
- Predict and prevent network outages (claims 95% outage prevention)
- Analyze network anomalies in real-time
- Multi-vendor network orchestration

**7. What is MCP (Manage, Control, Plan)?**
Ciena's **domain-level software** for managing optical and packet networks. Provides a single view of the entire network, automated service provisioning, and real-time analytics.

**8. What is Optical Networking (ON)?**
```
Optical Networking — How Data Travels:

  Data Center A                                       Data Center B
  ┌──────────┐                                       ┌──────────┐
  │  Servers  │                                       │  Servers  │
  │  Storage  │                                       │  Storage  │
  └─────┬────┘                                       └─────┬────┘
        │                                                   │
        ▼                                                   ▼
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  Router  │───▶│ Optical  │───▶│ Optical  │───▶│  Router  │
  │          │    │ Mux/DWDM │    │ Amplifier│    │          │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
                        │                │
                   Fiber Optic Cable (100s-1000s km)
                   Multiple wavelengths of light (DWDM)
                   Each wavelength = 100G-800G data

  DWDM = Dense Wavelength Division Multiplexing
  ───► Multiple colors of light travel simultaneously in one fiber
  ───► Each "color" (wavelength) carries independent data
  ───► Ciena equipment: creates, manages, amplifies these signals
```

**Key concepts:**
- **DWDM**: Pack 80-96 wavelengths into a single fiber → massive capacity
- **Coherent optics**: Advanced modulation techniques for long-distance, high-speed data
- **Photonic layer**: The physical light signals in fiber
- **OTN (Optical Transport Network)**: Standards for wrapping data into optical signals
- **ROADM**: Reconfigurable Optical Add-Drop Multiplexer — route wavelengths without converting to electrical

---

## Ciena Culture & Values

**9. What is Ciena's culture like?**
- **"People Power Progress"** — employees drive innovation
- Collaborative, globally distributed teams (US, Canada, India, worldwide)
- Strong engineering culture — the ON team impacts hundreds of developers
- Agile methodology
- Data-driven decision making
- Open to AI tools — Ciena encourages thoughtful AI use (see their recruitment AI policy)

**10. What makes Ciena different from competitors?**
- **Pure-play optical networking** (unlike Nokia/Ericsson which are broader)
- WaveLogic technology leadership — consistently first to hit new speed milestones
- Software-defined networking focus with Blue Planet
- Strong R&D investment (~20% of revenue)
- Acquired Nortel's optical division (2010) — gained deep Ottawa-based talent pool

---

# PART 2: THE POSITION — DevOps Engineer

---

## Role Summary

**11. What is this role about?**

```
Position: DevOps Engineer — Optical Networks (ON) Team
Team:     Fast-growing DevOps team within ON division
Focus:    CI/CD, automation, quality, stability, speed of delivery

  ┌──────────────────────────────────────────────────────────────────┐
  │                    YOUR IMPACT                                   │
  │                                                                  │
  │  Hundreds of ON developers depend on the DevOps infrastructure  │
  │  you build and maintain. You directly affect:                   │
  │                                                                  │
  │  ┌────────────┐   ┌────────────┐   ┌────────────┐              │
  │  │  Quality   │   │ Stability  │   │   Speed    │              │
  │  │ of ON SW   │   │ of builds  │   │ of release │              │
  │  │ releases   │   │ & CI env   │   │  delivery  │              │
  │  └────────────┘   └────────────┘   └────────────┘              │
  │                                                                  │
  │  Broad visibility across entire ON software organization        │
  └──────────────────────────────────────────────────────────────────┘
```

**12. What will you actually do day-to-day?**
1. **Enhance CI pipelines** — improve Jenkins pipelines for ON software builds
2. **Build automation** — scripts (Python, shell) to automate repetitive tasks
3. **Collaborate globally** — work with SW engineers, infra teams, tooling teams across geographies
4. **Integrate tools** — Python, Jenkins, Git, Google Repo, Gerrit into streamlined workflows
5. **Optimize code integration** — make merging, building, testing faster and more reliable
6. **Data-driven improvement** — use metrics to identify bottlenecks and improve system performance
7. **Problem-solve** — triage and resolve DevOps issues (broken builds, flaky tests, infra problems)

---

## JD Requirements — Mapped

**13. Must-haves vs your experience?**

```
JD Requirement Alignment:

  REQUIREMENT                    YOUR LEVEL              STATUS
  ─────────────────────────────────────────────────────────────────
  B.E./M.S. CS/CE/EE           ✅ B.Tech CSE             MATCH
  5+ yrs SW/tools dev in CI    ✅ 5+ yrs DevOps           MATCH
  Strong Git knowledge          ✅ Advanced (studied        MATCH
                                   rebase, cherry-pick,
                                   bisect, hooks, stash)
  Jenkins experience            ✅ Declarative +            MATCH
                                   studying Scripted
  Python experience             ✅ Scripting + REST APIs    MATCH
  Go/Yocto build envs           ⚠️  Awareness level         PREP'D
  Communication skills          ✅ Strong                   MATCH
  Development automation        ✅ CI/CD, IaC, scripts     MATCH
  Agile teams                   ✅ Sprint-based teams      MATCH
  DevOps fundamentals           ✅ Core expertise           MATCH
```

**14. Nice-to-haves vs your experience?**

```
  NICE-TO-HAVE                   YOUR LEVEL              STATUS
  ─────────────────────────────────────────────────────────────────
  Bitbucket, Google Repo,       ✅ Bitbucket used           PARTIAL
  Gerrit                        ⚠️  Gerrit/Repo studied     PREP'D
  Angular/HTML/CSS/JS/Node      ⚠️  Basic exposure          PARTIAL
  REST, SQL/NoSQL               ✅ REST APIs, SQL used      MATCH
  Docker                        ✅ Strong                   MATCH
  Ansible                       ✅ Used in projects         MATCH
  DevSecOps                     ✅ SonarQube, Snyk, Trivy  MATCH
  AI-driven automation          ✅ Using AI tools daily     MATCH
  Large-scale SW dev orgs       ✅ Large enterprise exp     MATCH
  Optical Networking concepts   ⚠️  Awareness level         PREP'D
```

---

## The Tech Stack (What Ciena ON Team Uses)

**15. What is the Ciena ON DevOps tech stack?**

```
Ciena ON DevOps Tech Stack:

  ┌─── Source Control ─────────────────────────────────────────────┐
  │  Git + Gerrit (code review) + Google Repo (multi-repo mgmt)   │
  │  Bitbucket (hosting)                                           │
  └────────────────────────────────────────────────────────────────┘
           │
           ▼
  ┌─── CI/CD ─────────────────────────────────────────────────────┐
  │  Jenkins (primary CI engine)                                   │
  │  - Hundreds of pipelines for ON software builds               │
  │  - Scripted + Declarative pipelines                           │
  │  - Shared libraries for common build patterns                 │
  └────────────────────────────────────────────────────────────────┘
           │
           ▼
  ┌─── Build Systems ─────────────────────────────────────────────┐
  │  Yocto/BitBake — Embedded Linux image builds for ON devices   │
  │  Go — CLI tools, microservices                                │
  │  C/C++ — Core ON firmware/software                            │
  └────────────────────────────────────────────────────────────────┘
           │
           ▼
  ┌─── Automation & Scripting ────────────────────────────────────┐
  │  Python — Build automation, data analysis, REST integration   │
  │  Bash/Shell — System scripts, build helpers                   │
  │  Ansible — Configuration management                           │
  └────────────────────────────────────────────────────────────────┘
           │
           ▼
  ┌─── Containers & Deployment ───────────────────────────────────┐
  │  Docker — Build environments, containerized tools             │
  │  (K8s likely for internal tooling)                            │
  └────────────────────────────────────────────────────────────────┘
           │
           ▼
  ┌─── Web/UI (Nice-to-have) ─────────────────────────────────────┐
  │  Angular + Node.js — Internal DevOps dashboards/portals       │
  │  REST APIs — Integration between tools                        │
  └────────────────────────────────────────────────────────────────┘
```

---

## Why Gerrit + Google Repo? (Ciena Context)

**16. Why does Ciena use Gerrit instead of GitHub/GitLab PRs?**

```
Gerrit Workflow (Ciena Style):

  Developer                    Gerrit                      Repository
  ┌──────────┐               ┌──────────┐               ┌──────────┐
  │ Write    │  git push     │ Code     │  Submit       │ Merged   │
  │ code on  │──────────────▶│ Review   │──────────────▶│ to       │
  │ local    │  refs/for/    │ (Change) │  (after +2    │ branch   │
  │ branch   │  master       │          │   approval)   │          │
  └──────────┘               └──────────┘               └──────────┘
                                  │
                              ┌───┴────┐
                              │ Jenkins │ ← Verify build
                              │ +1/-1  │   (automated)
                              └────────┘

  Why Gerrit for large ON teams:
  ✅ Enforces single-commit reviews (atomic changes)
  ✅ Pre-submit CI verification (Jenkins Verify label)
  ✅ Fine-grained access control per project/branch
  ✅ Scales to hundreds of developers
  ✅ Change-based workflow (not PR-based) = cleaner history
```

**17. Why does Ciena use Google Repo?**

```
Google Repo — Multi-Repository Management:

  ON Software = many Git repos (kernel, drivers, apps, libs, configs)

  manifest.xml defines the "super-project":
  ┌──────────────────────────────────────────────┐
  │  <project name="on-kernel"    path="kernel"  │
  │           revision="v5.10-ciena"/>            │
  │  <project name="on-platform"  path="platform"│
  │           revision="master"/>                 │
  │  <project name="on-apps"      path="apps"    │
  │           revision="release/3.0"/>            │
  │  <project name="meta-ciena"   path="yocto/   │
  │           meta-ciena" revision="master"/>     │
  └──────────────────────────────────────────────┘

  repo init -u <manifest-url>    # Initialize workspace
  repo sync                       # Fetch all repos
  repo forall -c 'git status'    # Run command across all repos
  repo upload                     # Push changes for Gerrit review

  Why Ciena uses it:
  ✅ Manage 50-100+ Git repos as a single workspace
  ✅ Reproducible builds (manifest pins exact versions)
  ✅ Coordinate changes across multiple repos
  ✅ Originally built by Google for Android (similar scale)
```

---

## Why Yocto Matters for Ciena

**18. Why does the ON team use Yocto?**

```
Ciena ON Devices Need Custom Embedded Linux:

  Optical Network Device (e.g., Ciena 6500)
  ┌─────────────────────────────────────────────┐
  │  ┌─────────────────────────────────────┐    │
  │  │  ON Application Software            │    │
  │  │  (control plane, management, SNMP)  │    │
  │  ├─────────────────────────────────────┤    │
  │  │  Libraries (OpenSSL, protobuf, etc) │    │
  │  ├─────────────────────────────────────┤    │
  │  │  Custom Linux Kernel + Drivers      │    │
  │  │  (for specific hardware: FPGA, DSP) │    │
  │  ├─────────────────────────────────────┤    │
  │  │  Bootloader (U-Boot)                │    │
  │  └─────────────────────────────────────┘    │
  │  Hardware: Custom boards, WaveLogic modem   │
  └─────────────────────────────────────────────┘

  Yocto builds ALL of this:
  ┌────────────────────────────────────────────────┐
  │  meta-ciena/          ← Ciena's custom layer   │
  │  ├── recipes-core/    ← ON-specific packages   │
  │  ├── recipes-kernel/  ← Custom kernel config   │
  │  ├── conf/            ← Machine definitions    │
  │  └── classes/         ← Build customizations   │
  │                                                 │
  │  bitbake ciena-on-image  ← Build command       │
  │  Output: flashable image for ON hardware       │
  └────────────────────────────────────────────────┘

  DevOps role with Yocto:
  • CI pipelines that build Yocto images (long builds: 1-4 hrs)
  • sstate-cache management for faster rebuilds
  • Build reproducibility across developer machines
  • Automated image testing and deployment
```

---

# PART 3: INTERVIEW PREPARATION

---

## "Why Ciena?" — Elevator Pitch

**19. Why do you want to join Ciena?**

> "Three reasons:
>
> **1. Impact at scale** — The ON DevOps team serves hundreds of developers. The CI/CD infrastructure I build won't just serve one team — it'll accelerate the entire optical networking software organization. That's the kind of broad impact I look for.
>
> **2. Technical depth** — Ciena's stack is technically challenging — embedded Linux with Yocto, multi-repo management with Google Repo, Gerrit code review at scale, Jenkins pipelines for complex builds. Coming from Azure DevOps and cloud-native CI/CD, I'm excited to deepen my skills in embedded systems DevOps.
>
> **3. Domain significance** — Ciena powers the backbone of the internet. Optical networking is foundational infrastructure — it's not just another SaaS product, it's the physical layer that everything else depends on. Working on software that runs inside these devices feels meaningful."

---

## "Why should we hire you?" — Strengths to Lead With

**20. How to position yourself?**

```
Your Strengths (Lead With):
──────────────────────────────────────────────────────────
✅ CI/CD pipeline design & optimization (Azure DevOps → Jenkins transferable)
✅ Docker + K8s in production environments
✅ Python + shell scripting for automation
✅ Infrastructure as Code (Terraform + Ansible)
✅ DevSecOps practices (SonarQube, Snyk, Trivy)
✅ Monitoring & observability (Prometheus, Grafana)
✅ Working effectively in distributed, Agile teams
✅ Large-scale enterprise experience
✅ Quick learner — demonstrated by deep prep for Yocto/Gerrit/Repo

Your Gaps (Frame Positively):
──────────────────────────────────────────────────────────
⚠️  Gerrit/Google Repo: "I understand the workflow concepts from
    studying the architecture. The code review philosophy is similar
    to what I've done with PRs — just a different tool and workflow."

⚠️  Yocto/Go: "I've built awareness of Yocto's layer system,
    BitBake recipes, and build pipeline. My CI/CD experience means
    I can optimize build caching, parallelization, and
    reproducibility — the DevOps challenges are universal."

⚠️  Jenkins Scripted: "My primary experience is Declarative
    pipelines, but I've studied Scripted pipelines, shared
    libraries, and Groovy DSL. The CI/CD concepts are identical."
```

---

## Key Questions They Might Ask (Ciena-Specific)

**21. How would you optimize a CI pipeline that takes 4 hours to build a Yocto image?**

```
Yocto CI Optimization Strategy:

  Problem: Full Yocto build = 2-4 hours
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  1. sstate-cache (Shared State Cache)                   │
  │     - Cache build artifacts across builds               │
  │     - Only rebuild changed recipes                      │
  │     - Store sstate on shared NFS/S3                     │
  │     - Savings: 80-90% on incremental builds             │
  │                                                          │
  │  2. DL_DIR (Download Directory)                         │
  │     - Pre-cache source tarballs                         │
  │     - Avoid re-downloading on every build               │
  │     - Mirror internally for reliability                 │
  │                                                          │
  │  3. Build parallelization                               │
  │     - BB_NUMBER_THREADS = CPU cores                     │
  │     - PARALLEL_MAKE = "-j$(nproc)"                      │
  │     - Distribute across build agents                    │
  │                                                          │
  │  4. Incremental builds                                  │
  │     - Detect which recipes changed (git diff)           │
  │     - Build only affected packages                      │
  │     - Use buildhistory for change tracking              │
  │                                                          │
  │  5. Pipeline design                                     │
  │     - Separate validation (lint/static) from full build │
  │     - Quick feedback in <10 min for PR validation       │
  │     - Full image build only on merge to main            │
  │     - Nightly full clean builds for verification        │
  │                                                          │
  │  Result: PR feedback in 10-15 min, full build in ~30min │
  └──────────────────────────────────────────────────────────┘
```

**22. How would you manage CI for a multi-repo project with Google Repo?**

```
Multi-Repo CI Strategy:

  Trigger:
  ┌─────────────────────────────────────────────────────┐
  │  Gerrit webhook → Jenkins → Which repo changed?    │
  │                                                     │
  │  Option A: Build only the changed repo              │
  │  Option B: Build the full manifest (safe but slow)  │
  │  Option C: Dependency-aware build (smart)           │
  └─────────────────────────────────────────────────────┘

  Smart CI approach:
  1. Maintain dependency graph between repos
  2. On change in repo X → find affected downstream repos
  3. Run targeted builds for affected components
  4. Full integration build on merge to release branch

  Jenkins Pipeline:
  ┌────────────────────────────────────────────────────┐
  │  stage('Init') { repo init + repo sync }           │
  │  stage('Detect') { identify changed recipes/repos }│
  │  stage('Build') { bitbake only affected targets }  │
  │  stage('Test') { run unit + integration tests }    │
  │  stage('Report') { publish results to Gerrit }     │
  └────────────────────────────────────────────────────┘
```

**23. How would you handle a broken CI environment affecting hundreds of developers?**

```
Incident Response for CI Outage:

  1. TRIAGE (first 5 min)
     ├── Is it a build failure or infrastructure issue?
     ├── How many developers are blocked?
     └── Can we roll back to last known good state?

  2. COMMUNICATE (immediately)
     ├── Post in team channel: "CI outage — investigating"
     ├── Set up incident channel if widespread
     └── ETA updates every 15 min

  3. MITIGATE (next 30 min)
     ├── Roll back if recent change caused it
     ├── Spin up backup build agents if capacity issue
     ├── Provide workaround (local builds) if possible
     └── Disable non-critical pipelines to free resources

  4. FIX & VERIFY
     ├── Root cause analysis
     ├── Fix and verify in staging first
     ├── Gradual rollout of fix
     └── Monitor for recurrence

  5. POST-MORTEM
     ├── Blameless retrospective
     ├── Document: timeline, root cause, impact, action items
     ├── Improve: add monitoring, alerts, auto-recovery
     └── Share learnings with broader team
```

**24. How would you use data-driven approaches to improve DevOps? (JD emphasis)**

```
DevOps Metrics & Data-Driven Improvement:

  ┌─── Collect ────────────────────────────────────┐
  │  Build times per pipeline/stage                │
  │  Build success/failure rates                   │
  │  Queue wait times                              │
  │  Test pass rates + flaky test tracking          │
  │  sstate-cache hit rates (Yocto-specific)       │
  │  Developer feedback cycle time                 │
  │  Resource utilization (build agents CPU/memory) │
  └────────────────────────────────────────────────┘
          │
          ▼
  ┌─── Analyze ────────────────────────────────────┐
  │  Identify bottleneck stages                    │
  │  Find most-failing pipelines                   │
  │  Correlate failures to code changes            │
  │  Track improvement trends over time            │
  └────────────────────────────────────────────────┘
          │
          ▼
  ┌─── Act ────────────────────────────────────────┐
  │  Optimize slowest stages                       │
  │  Auto-quarantine flaky tests                   │
  │  Scale agents based on queue depth             │
  │  Set SLOs and alert on regression              │
  └────────────────────────────────────────────────┘
```

---
---

# PART 4: CIENA ROUND 1 — INTERVIEW DEBRIEF & ANSWERS

> **Round 1 Date:** May 2026
> **Status:** Questions I failed to answer + correct answers for next round

---

## QUESTIONS I FAILED TO ANSWER

---

### Q1. How is Python exactly different from C/C++?

```
Python vs C/C++ — Key Differences:

  ┌──────────────────────┬──────────────────────┬──────────────────────┐
  │     ASPECT           │     PYTHON           │     C / C++          │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Type System          │ Dynamically typed    │ Statically typed     │
  │                      │ (type checked at     │ (type checked at     │
  │                      │  runtime)            │  compile time)       │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Execution            │ Interpreted          │ Compiled             │
  │                      │ (bytecode → PVM)     │ (source → machine    │
  │                      │                      │  code directly)      │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Memory Management    │ Automatic (GC +      │ Manual (malloc/free  │
  │                      │  reference counting) │  in C, new/delete    │
  │                      │                      │  in C++)             │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Speed                │ Slower (10-100x)     │ Much faster          │
  │                      │ Interpreted overhead │ Direct machine code  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Pointers             │ No pointers          │ Full pointer support │
  │                      │ Everything is a      │ Direct memory access │
  │                      │ reference to object  │                      │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Syntax               │ Indentation-based    │ Curly braces {}      │
  │                      │ Minimal boilerplate  │ Semicolons required  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ OOP                  │ Fully OOP but also   │ C: Procedural only   │
  │                      │ supports procedural  │ C++: Multi-paradigm  │
  │                      │ & functional         │ (OOP + procedural)   │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Use Cases            │ Scripting, DevOps,   │ OS kernels, drivers, │
  │                      │ automation, ML/AI,   │ embedded systems,    │
  │                      │ web backends         │ game engines,        │
  │                      │                      │ high-perf computing  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Platform             │ Platform-independent │ Platform-dependent   │
  │                      │ (runs on any OS      │ (must recompile for  │
  │                      │  with Python)        │  each target)        │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Error Handling       │ Exceptions           │ C: return codes      │
  │                      │ (try/except)         │ C++: exceptions      │
  │                      │                      │ (try/catch)          │
  └──────────────────────┴──────────────────────┴──────────────────────┘
```

**Key interview answer:**
> "Python is **interpreted** and **dynamically typed** — you don't declare types, and code runs through the Python Virtual Machine. C/C++ is **compiled** directly to machine code and **statically typed** — types are checked at compile time. This makes C/C++ much faster but Python much more productive for scripting and automation. The biggest difference for a DevOps role: Python handles memory automatically with garbage collection, while C/C++ requires manual memory management (malloc/free, new/delete). Python also has no pointers — everything is an object reference."

```python
# Python — simple, no types needed
name = "Vaibhav"       # string, no declaration
age = 25                # int, can reassign to any type
items = [1, "two", 3.0] # mixed types in list
```

```c
// C — explicit types, manual memory
char* name = "Vaibhav";          // must declare type
int age = 25;                     // cannot change type
int items[3] = {1, 2, 3};        // homogeneous, fixed size
char* heap = malloc(100);         // manual allocation
free(heap);                       // manual deallocation!
```

---

### Q2. Difference between Tuples and Lists

```
Tuple vs List — Complete Comparison:

  ┌─────────────────┬───────────────────────┬───────────────────────┐
  │   FEATURE       │       LIST            │       TUPLE           │
  ├─────────────────┼───────────────────────┼───────────────────────┤
  │ Syntax          │ [1, 2, 3]             │ (1, 2, 3)             │
  │ Mutable?        │ YES — can add/remove/ │ NO — immutable,       │
  │                 │ change elements       │ cannot change once    │
  │                 │                       │ created               │
  │ Speed           │ Slower                │ Faster (smaller       │
  │                 │                       │ memory footprint)     │
  │ Memory          │ More memory (needs    │ Less memory (fixed    │
  │                 │ extra space for       │ size, no resize       │
  │                 │ dynamic resizing)     │ overhead)             │
  │ Dict Key?       │ NO (unhashable)       │ YES (hashable)        │
  │ Set Element?    │ NO                    │ YES                   │
  │ Thread Safety   │ Not inherently safe   │ Safe (immutable =     │
  │                 │                       │ no race conditions)   │
  │ Use Case        │ Collection that       │ Fixed data like       │
  │                 │ changes: shopping     │ coordinates (x,y),    │
  │                 │ cart, task list,       │ RGB colors, DB rows,  │
  │                 │ build steps           │ function returns      │
  └─────────────────┴───────────────────────┴───────────────────────┘
```

```python
# LIST — mutable
servers = ["web01", "web02", "web03"]
servers.append("web04")      # ✅ Can add
servers[0] = "nginx01"       # ✅ Can modify
servers.remove("web02")      # ✅ Can remove
# servers = ['nginx01', 'web03', 'web04']

# TUPLE — immutable
server_config = ("10.0.0.1", 8080, "production")
# server_config[0] = "10.0.0.2"  # ❌ TypeError!
# server_config.append("new")     # ❌ AttributeError!

ip, port, env = server_config    # ✅ Tuple unpacking works great

# WHY TUPLES MATTER — Can be dictionary keys
location_cache = {
    (40.7128, -74.0060): "New York",   # tuple as key ✅
    (51.5074, -0.1278): "London",
}
# {[40.7128, -74.0060]: "New York"}   # list as key ❌ TypeError

# Memory comparison
import sys
list_size = sys.getsizeof([1, 2, 3, 4, 5])    # 104 bytes
tuple_size = sys.getsizeof((1, 2, 3, 4, 5))   # 80 bytes
# Tuple uses ~23% less memory
```

**Key interview answer:**
> "Lists are **mutable** (can add, remove, modify elements), tuples are **immutable** (fixed once created). Because tuples are immutable, they're **hashable** — so they can be used as dictionary keys and set elements, lists cannot. Tuples are also **faster and use less memory**. I use lists when the collection changes (like a list of build servers), and tuples for fixed data (like a server's IP/port/env config)."

---

### Q3. Can key-value pairs be added to a List? How is a List different from an Array?

**Yes, you can store key-value pairs in a list** — as a list of tuples or dicts:

```python
# Method 1: List of tuples (key-value pairs)
config = [("host", "10.0.0.1"), ("port", 8080), ("env", "prod")]

# Method 2: List of dictionaries
servers = [
    {"name": "web01", "ip": "10.0.0.1"},
    {"name": "web02", "ip": "10.0.0.2"},
]

# Method 3: Dictionary itself (the RIGHT tool for key-value)
config = {"host": "10.0.0.1", "port": 8080, "env": "prod"}
config["region"] = "us-east"  # Add key-value pair
```

**But for actual key-value data, always use a `dict`** — it's O(1) lookup vs O(n) for a list.

```
List vs Array — Key Differences:

  ┌──────────────────┬────────────────────────┬────────────────────────┐
  │   FEATURE        │   Python LIST          │   Array (C/array mod)  │
  ├──────────────────┼────────────────────────┼────────────────────────┤
  │ Data Types       │ MIXED types allowed    │ SINGLE type only       │
  │                  │ [1, "hi", 3.14, True]  │ [1, 2, 3, 4]          │
  ├──────────────────┼────────────────────────┼────────────────────────┤
  │ Size             │ Dynamic (auto-resize)  │ C: Fixed size          │
  │                  │                        │ Python array: Dynamic  │
  ├──────────────────┼────────────────────────┼────────────────────────┤
  │ Memory           │ Stores references to   │ Stores raw values      │
  │                  │ objects (more memory)  │ contiguously (compact) │
  ├──────────────────┼────────────────────────┼────────────────────────┤
  │ Performance      │ Slower for numeric     │ Faster for numeric     │
  │                  │ operations             │ (numpy even faster)    │
  ├──────────────────┼────────────────────────┼────────────────────────┤
  │ Built-in?        │ YES — core type        │ Needs `import array`   │
  │                  │                        │ or `import numpy`      │
  ├──────────────────┼────────────────────────┼────────────────────────┤
  │ Methods          │ append, insert, pop,   │ Fewer methods,         │
  │                  │ sort, reverse, etc.    │ focused on numeric ops │
  └──────────────────┴────────────────────────┴────────────────────────┘
```

```python
# Python LIST — heterogeneous, dynamic
my_list = [1, "hello", 3.14, True, [1,2]]  # mixed types ✅

# Python array module — homogeneous, typed
import array
my_array = array.array('i', [1, 2, 3, 4])  # 'i' = signed int only
# my_array.append("hello")  # ❌ TypeError — int only!

# NumPy array — the real "array" in practice
import numpy as np
np_arr = np.array([1, 2, 3, 4])  # homogeneous, contiguous memory
np_arr * 2  # → [2, 4, 6, 8]  vectorized operation (FAST)
```

**Key interview answer:**
> "Yes, you can store key-value pairs in a list as a list of tuples or dicts, but a `dict` is the right tool for that — O(1) lookup vs O(n). Lists differ from arrays in that Python lists store **references to objects** and allow **mixed types**, while arrays (C-style or `array` module) store **raw values of a single type** contiguously in memory, making them more memory-efficient and faster for numeric operations."

---

### Q4. How to attach a disk to a server (on-prem or cloud)?

```
Disk Attachment — Universal Steps:

  ┌──────────────────────────────────────────────────┐
  │              GENERAL WORKFLOW                     │
  │                                                  │
  │  1. IDENTIFY the disk and the server             │
  │  2. ATTACH (physical or logical)                 │
  │  3. PARTITION the disk (optional)                │
  │  4. FORMAT with a filesystem                     │
  │  5. MOUNT to a directory                         │
  │  6. PERSIST the mount (fstab / cloud config)     │
  └──────────────────────────────────────────────────┘
```

**On-Premises (Physical Server / VMware):**

```bash
# 1. Physically install disk (or add virtual disk in VMware/KVM)
#    VMware: VM Settings → Add Hard Disk → Select datastore

# 2. Detect the new disk
lsblk                    # List block devices
fdisk -l                 # See all disks with details
dmesg | grep sd          # Check kernel messages for new disk
# New disk appears as /dev/sdb (or /dev/nvme1n1 for NVMe)

# 3. Partition the disk
sudo fdisk /dev/sdb
# n → new partition → p → primary → 1 → defaults → w (write)
# OR for disks > 2TB:
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%

# 4. Format with filesystem
sudo mkfs.ext4 /dev/sdb1           # Linux ext4
# OR: sudo mkfs.xfs /dev/sdb1      # XFS (better for large files)

# 5. Mount
sudo mkdir -p /data
sudo mount /dev/sdb1 /data

# 6. Persist in /etc/fstab
sudo blkid /dev/sdb1               # Get UUID
echo "UUID=<uuid> /data ext4 defaults 0 2" | sudo tee -a /etc/fstab
sudo mount -a                       # Verify fstab entry
```

**Cloud — Azure:**

```bash
# Using Azure CLI
# 1. Create a managed disk
az disk create \
  --resource-group myRG \
  --name myDataDisk \
  --size-gb 128 \
  --sku Premium_LRS

# 2. Attach to VM
az vm disk attach \
  --resource-group myRG \
  --vm-name myVM \
  --name myDataDisk

# 3. SSH into VM and format/mount (same Linux steps as above)
ssh user@myVM
lsblk                              # Find new disk (e.g., /dev/sdc)
sudo mkfs.ext4 /dev/sdc
sudo mkdir -p /datadisk
sudo mount /dev/sdc /datadisk
# Add to /etc/fstab for persistence
```

**Cloud — AWS:**

```bash
# 1. Create EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp3

# 2. Attach to instance
aws ec2 attach-volume \
  --volume-id vol-xxx \
  --instance-id i-xxx \
  --device /dev/xvdf

# 3. SSH and format/mount (same Linux steps)
```

**Key interview answer:**
> "The process is the same everywhere: **attach → detect → partition → format → mount → persist**. On-prem, you physically add the disk or add it via hypervisor (VMware/KVM), then use `fdisk`/`parted` to partition, `mkfs` to format, and `mount` + `/etc/fstab` for persistence. In cloud, you create a managed disk (Azure: `az disk create`, AWS: `create-volume`) and attach it via CLI/portal, then SSH in and do the same Linux steps. Always use **UUID in fstab**, not device names, because device names can change on reboot."

---

### Q5. If there is a heap of disks and a set of servers, how will you attach disks to them?

**This is an automation/scale question — they want to know if you can think at scale.**

```
Approach for Bulk Disk Attachment:

  ┌──────────────────────────────────────────────────────────┐
  │                  STRATEGY                                 │
  │                                                          │
  │  1. INVENTORY    — List all servers and disks            │
  │  2. PLAN         — Map which disk → which server        │
  │  3. AUTOMATE     — Script the attachment + formatting   │
  │  4. VALIDATE     — Verify all mounts are working        │
  │  5. PERSIST      — Ensure mounts survive reboot         │
  └──────────────────────────────────────────────────────────┘
```

**Using Ansible (the right answer for interviews):**

```yaml
# inventory.yml
all:
  hosts:
    server1:
      ansible_host: 10.0.0.1
      disks: ["/dev/sdb", "/dev/sdc"]
    server2:
      ansible_host: 10.0.0.2
      disks: ["/dev/sdb"]
    server3:
      ansible_host: 10.0.0.3
      disks: ["/dev/sdb", "/dev/sdc", "/dev/sdd"]

# attach_disks.yml
---
- name: Attach and mount disks to servers
  hosts: all
  become: yes
  tasks:
    - name: Create filesystem on each disk
      filesystem:
        fstype: ext4
        dev: "{{ item }}"
      loop: "{{ disks }}"

    - name: Create mount directories
      file:
        path: "/data/disk{{ idx }}"
        state: directory
      loop: "{{ disks }}"
      loop_control:
        index_var: idx

    - name: Mount disks
      mount:
        path: "/data/disk{{ idx }}"
        src: "{{ item }}"
        fstype: ext4
        state: mounted       # This also adds to fstab
      loop: "{{ disks }}"
      loop_control:
        index_var: idx
```

```bash
# Run across all servers in one command
ansible-playbook -i inventory.yml attach_disks.yml
```

**For Cloud at scale (Azure example):**

```bash
#!/bin/bash
# Bulk attach disks in Azure

SERVERS=("vm1" "vm2" "vm3" "vm4" "vm5")
RG="production-rg"
DISK_SIZE=256
DISK_SKU="Premium_LRS"

for server in "${SERVERS[@]}"; do
    # Create disk
    az disk create \
      --resource-group $RG \
      --name "${server}-datadisk" \
      --size-gb $DISK_SIZE \
      --sku $DISK_SKU

    # Attach to VM
    az vm disk attach \
      --resource-group $RG \
      --vm-name "$server" \
      --name "${server}-datadisk"

    echo "✅ Disk attached to $server"
done

# Then use Ansible to SSH into all VMs and format/mount
ansible-playbook -i azure_inventory format_mount.yml
```

**Terraform approach (if disks are part of infra-as-code):**

```hcl
variable "servers" {
  default = {
    "web01" = { disk_size = 128 }
    "web02" = { disk_size = 256 }
    "db01"  = { disk_size = 512 }
  }
}

resource "azurerm_managed_disk" "data" {
  for_each             = var.servers
  name                 = "${each.key}-datadisk"
  location             = azurerm_resource_group.rg.location
  resource_group_name  = azurerm_resource_group.rg.name
  storage_account_type = "Premium_LRS"
  create_option        = "Empty"
  disk_size_gb         = each.value.disk_size
}

resource "azurerm_virtual_machine_data_disk_attachment" "data" {
  for_each           = var.servers
  managed_disk_id    = azurerm_managed_disk.data[each.key].id
  virtual_machine_id = azurerm_virtual_machine.vm[each.key].id
  lun                = 1
  caching            = "ReadWrite"
}
```

**Key interview answer:**
> "For bulk operations, I'd never do it manually. I'd use **Ansible** for configuration management — define an inventory of servers and their disk mappings, then create a playbook that partitions, formats, and mounts disks idempotently using the `filesystem` and `mount` modules. For cloud, I'd use **Terraform** to declaratively manage disk resources and attachments as infrastructure-as-code, then Ansible for the OS-level formatting/mounting. The key principles: **automate everything, use idempotent tools, track state, and validate after.**"

---

### Q6. How is memory managed in Python?

```
Python Memory Management — Architecture:

  ┌─────────────────────────────────────────────────────┐
  │                 PYTHON MEMORY MANAGER                │
  │                                                     │
  │  ┌───────────────────────────────────────────────┐  │
  │  │  Layer 3: OBJECT-SPECIFIC ALLOCATORS          │  │
  │  │  int, float, list, dict each have optimized   │  │
  │  │  allocation strategies                        │  │
  │  ├───────────────────────────────────────────────┤  │
  │  │  Layer 2: PYTHON OBJECT ALLOCATOR (pymalloc)  │  │
  │  │  Manages objects ≤ 512 bytes                  │  │
  │  │  Uses ARENAS (256KB) → POOLS (4KB) → BLOCKS  │  │
  │  ├───────────────────────────────────────────────┤  │
  │  │  Layer 1: RAW MEMORY ALLOCATOR                │  │
  │  │  Wraps C's malloc/free for objects > 512B     │  │
  │  ├───────────────────────────────────────────────┤  │
  │  │  Layer 0: OS MEMORY (heap)                    │  │
  │  │  Virtual memory from the operating system     │  │
  │  └───────────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────────┘
```

**Two main mechanisms:**

**1. Reference Counting (Primary):**
```python
# Every object has a reference count
import sys

a = [1, 2, 3]           # refcount = 1
b = a                    # refcount = 2 (b points to same object)
print(sys.getrefcount(a)) # 3 (includes the getrefcount arg itself)

del b                    # refcount drops to 2
# When refcount hits 0 → memory freed IMMEDIATELY
```

**2. Garbage Collector (for circular references):**
```python
import gc

# Circular reference — refcount never reaches 0!
class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()
a.ref = b    # a → b
b.ref = a    # b → a  (CIRCULAR!)

del a
del b
# Refcounts are still 1 each (they reference each other)
# Python's GC detects this cycle and frees both

gc.collect()             # Force garbage collection
gc.get_count()           # Check GC thresholds
gc.get_threshold()       # Default: (700, 10, 10) — 3 generations
```

```
Generational Garbage Collection:

  ┌──────────────────────────────────────────────────┐
  │  Generation 0 (young)    — Collected most often  │
  │  ├── New objects go here                         │
  │  ├── Collected when allocs - deallocs > 700      │
  │  ├── Surviving objects promoted to Gen 1         │
  │                                                  │
  │  Generation 1 (middle)   — Collected less often  │
  │  ├── Objects that survived Gen 0 collection      │
  │  ├── Collected every 10 Gen 0 collections        │
  │  ├── Survivors promoted to Gen 2                 │
  │                                                  │
  │  Generation 2 (old)      — Collected rarely      │
  │  ├── Long-lived objects                          │
  │  ├── Collected every 10 Gen 1 collections        │
  │  └── Full collection — most expensive            │
  └──────────────────────────────────────────────────┘
```

**Key Python memory facts:**
- **Everything is an object** — even `int`, `bool`, `None`
- **Small integers (-5 to 256) are cached** — pre-allocated, reused
- **String interning** — small strings are cached and reused
- **Private heap** — Python manages its own heap, you never use malloc
- **GIL (Global Interpreter Lock)** — only one thread executes Python bytecode at a time, simplifying memory management but limiting true parallelism
- **No manual memory management** — no `malloc`/`free`, no `new`/`delete`

```python
# Prove integer caching
a = 256
b = 256
print(a is b)  # True — same object (cached)

a = 257
b = 257
print(a is b)  # False — different objects (not cached)

# Memory-efficient iteration — generators
# BAD: loads ALL into memory
big_list = [x**2 for x in range(10_000_000)]   # ~80MB RAM

# GOOD: generates one at a time
big_gen = (x**2 for x in range(10_000_000))     # ~0MB RAM
```

**Key interview answer:**
> "Python manages memory automatically using two mechanisms: **reference counting** and a **generational garbage collector**. Every object has a reference count — when it drops to zero, memory is freed immediately. For circular references (A→B→A), the garbage collector runs periodically using a 3-generation scheme (young objects collected most often, old ones rarely). Python uses a **private heap** managed by `pymalloc` for small objects (≤512 bytes), organized in arenas/pools/blocks. Developers never call malloc/free. Python also optimizes memory with **integer caching** (-5 to 256) and **string interning**."

---

## QUESTIONS I ANSWERED SUCCESSFULLY

---

### ✅ Q7. What is Docker and how does it work?

*(Answered correctly in the interview — brief reference below)*

Docker is a **containerization platform** that packages applications with their dependencies into lightweight, portable containers.

```
How Docker Works:

  Dockerfile → docker build → Image → docker run → Container

  ┌──────────────────────────────────────────────────┐
  │               HOST OS (Linux Kernel)              │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
  │  │Container1│  │Container2│  │Container3│       │
  │  │ App + Deps│  │ App + Deps│  │ App + Deps│       │
  │  └──────────┘  └──────────┘  └──────────┘       │
  │  ─────────────────────────────────────────       │
  │  Docker Engine (containerd + runc)               │
  │  ─────────────────────────────────────────       │
  │  Linux Kernel (namespaces + cgroups + UnionFS)   │
  └──────────────────────────────────────────────────┘

  Key tech: namespaces (isolation), cgroups (resource limits),
            UnionFS (layered filesystem)
```
  │  Dashboard: Grafana/ELK visualizations          │
  │  Trends: build time creep, failure patterns     │
  │  Bottlenecks: which stage is slowest?           │
  │  Patterns: failures by day/time/developer/repo  │
  └────────────────────────────────────────────────┘
          │
          ▼
  ┌─── Act ────────────────────────────────────────┐
  │  Slow stage → parallelize or cache             │
  │  Flaky test → quarantine + assign owner        │
  │  Long queue → scale build agents               │
  │  Low cache hit → improve sstate management     │
  │  Recurring failure → automated remediation     │
  └────────────────────────────────────────────────┘
```

---

## Behavioral / Situational

**25. Tell me about yourself (Ciena-tailored version).**

> "I'm a DevOps Engineer with 5+ years of experience in CI/CD pipeline design, infrastructure automation, and developer tooling. At [current company], I work with Azure DevOps pipelines, Docker, Kubernetes, Terraform, and Ansible to deliver software for large-scale enterprise applications.
>
> What excites me about this role at Ciena is the opportunity to work at the intersection of DevOps and embedded systems — optimizing CI for Yocto-based builds, working with Gerrit and Google Repo at scale, and directly impacting the productivity of hundreds of ON developers. I thrive in environments where DevOps isn't just a support function but a force multiplier for engineering velocity."

**26. What do you know about the team you're joining?**

> "The ON DevOps team is a fast-growing group within Ciena's Optical Networks division. The team owns the CI/CD infrastructure, build automation, and developer workflows that support ON software releases. The tech stack includes Jenkins, Git/Gerrit, Google Repo, Python, and Yocto/Go build systems. The team collaborates across geographies — likely Gurgaon, Ottawa, and the US — in an Agile environment. The role has broad visibility because the infrastructure serves hundreds of developers."

**27. How do you handle learning new tools quickly? (Gerrit, Repo, Yocto)**

> "I approach it systematically. For this interview, I've invested significant time understanding Gerrit's change-based review model, Google Repo's manifest system, and Yocto's layer architecture and BitBake build process. I haven't used them in production yet, but I understand the concepts well enough to be productive quickly. My experience with Azure DevOps PRs maps to Gerrit reviews, my multi-repo experience maps to Google Repo, and my CI pipeline optimization experience is directly applicable to Yocto build caching and parallelization."

---

## Ciena Interview Process Notes

**28. What to expect in the interview?**

```
Typical Interview Format:
  ┌──────────────────────────────────────────────────────┐
  │  1. Introduction & "Tell me about yourself" (5 min)  │
  │  2. Technical questions on core skills (30-40 min)   │
  │     - Git workflows (deep)                           │
  │     - Jenkins pipelines                              │
  │     - Linux/shell scripting                          │
  │     - Python                                         │
  │     - Docker                                         │
  │     - DevOps concepts                                │
  │  3. Scenario-based questions (15-20 min)             │
  │     - "How would you handle..."                      │
  │     - CI optimization                                │
  │     - Debugging build failures                       │
  │  4. Gerrit/Repo/Yocto awareness (5-10 min)          │
  │  5. Your questions for them (5-10 min)               │
  └──────────────────────────────────────────────────────┘
```

**29. Questions to ask the interviewer?**

1. "What does a typical CI pipeline look like for an ON software build? How long does a full build take?"
2. "How many repos does the team manage with Google Repo? What's the manifest structure?"
3. "What are the biggest DevOps challenges the ON team faces today?"
4. "How does the team measure DevOps success? What metrics do you track?"
5. "What does the onboarding process look like for a new DevOps engineer?"
6. "Is the team exploring any new tools or practices — like AI-assisted automation or GitOps?"
7. "How do you handle build reproducibility across different developer environments?"

---

## Ciena's AI Policy (For Reference)

**30. Ciena encourages AI use — but authentically.**

```
Ciena's stance on AI in recruitment:

  ✅ ENCOURAGED:
  │  Use AI to research Ciena, industry trends
  │  Use AI to practice and prepare stories
  │  Use AI to polish resume descriptions
  │  Use AI to brainstorm and outline ideas
  │
  ❌ NOT ALLOWED:
  │  No AI during live interviews
  │  Don't copy AI-written answers verbatim
  │  Don't use real-time AI coaching during calls
  │  Don't fabricate experience using AI
  │
  Key quote: "Use AI as a collaborator, not as a creator"
  Key quote: "We want to see how YOU think"
```

**Takeaway**: Be transparent that you used AI to structure your preparation. Show that you understand the concepts — don't just recite answers.

---

## Quick Reference — Optical Networking Vocabulary

**31. Key terms to know (so you don't sound lost if mentioned):**

| Term | Meaning |
|------|---------|
| **DWDM** | Dense Wavelength Division Multiplexing — many wavelengths in one fiber |
| **WaveLogic** | Ciena's coherent optics modem technology |
| **Coherent optics** | Advanced light modulation for high-speed long-distance transmission |
| **ROADM** | Reconfigurable Optical Add-Drop Multiplexer — route light signals |
| **OTN** | Optical Transport Network — standards for optical data framing |
| **400G/800G** | Data rate per wavelength (current gen) |
| **Transponder** | Converts client signals to DWDM wavelengths |
| **Photonic layer** | Physical light signal layer in the network |
| **Packet-optical** | Converged packet switching + optical transport |
| **ON** | Optical Networks — the division you're joining |
| **NMS** | Network Management System |
| **NE** | Network Element — a device in the network (router, switch, amplifier) |
| **SNMP** | Simple Network Management Protocol — monitor network devices |
| **NETCONF/YANG** | Modern network configuration protocols |
| **SDN** | Software-Defined Networking |
| **Control plane** | The "brain" that makes routing/switching decisions |
| **Data plane** | The part that actually moves data packets/light signals |
| **Firmware** | Software embedded in hardware devices |
