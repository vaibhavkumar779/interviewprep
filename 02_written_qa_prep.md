# WRITTEN Q&A PREP - Basic to Advanced
# Organized by topic areas from the JD

---

## 1. CI/CD & DEVOPS FUNDAMENTALS

### Basic
**Q: What is CI/CD? Explain the difference between CI, CD (Delivery), and CD (Deployment).**
A: 
- CI (Continuous Integration): Developers merge code to main branch frequently. Each merge triggers automated build + tests. Catches integration bugs early.
- CD (Continuous Delivery): Code is always in a deployable state. After CI passes, artifact is ready to deploy but requires manual approval.
- CD (Continuous Deployment): Every change that passes all stages is automatically deployed to production. No manual gate.

**Q: What is DevOps? Is it a tool or culture?**
A: DevOps is a **culture and set of practices** that bridges Development and Operations. It emphasizes:
- Collaboration between dev and ops teams
- Automation of build, test, deployment
- Continuous feedback loops
- Infrastructure as Code
- Monitoring and observability
Tools enable DevOps but are not DevOps itself.

**Q: What is Infrastructure as Code (IaC)? Tools you've used?**
A: Managing infrastructure through code/config files instead of manual processes.
- Benefits: Version controlled, repeatable, auditable, testable
- Tools I've used: Terraform (provisioning), Ansible (configuration management)
- Terraform = declarative (desired state), Ansible = procedural/declarative hybrid

### Intermediate
**Q: What is a build pipeline vs release pipeline?**
A:
- Build pipeline: Compiles code, runs unit tests, produces artifacts (e.g., Docker images, JARs, binaries)
- Release pipeline: Takes build artifacts and deploys them to environments (dev → staging → prod)
- In modern CI/CD, these are often combined into a single pipeline with stages

**Q: What is blue-green deployment? Canary deployment?**
A:
- Blue-Green: Two identical environments. Blue = current prod, Green = new version. Switch traffic after validation. Instant rollback by switching back.
- Canary: Route small % of traffic (e.g., 5%) to new version. Monitor metrics. Gradually increase if healthy. Safer for large-scale systems.

**Q: What is GitOps?**
A: Using Git as the single source of truth for declarative infrastructure and application config. Changes to infrastructure are made via Git PRs/merges. Tools like ArgoCD or Flux watch the Git repo and sync the cluster state.

### Advanced
**Q: How would you design a CI pipeline for a monorepo with 100+ components?**
A:
- Use path-based triggers (only build what changed)
- Dependency graph to identify affected downstream components
- Parallel builds for independent components
- Shared pipeline templates to reduce duplication
- Caching (dependencies, build artifacts, Docker layers)
- Fan-out/fan-in pattern for testing
- Artifact promotion between stages rather than rebuilding

**Q: How do you handle flaky tests in CI?**
A:
- Quarantine known flaky tests (run but don't block)
- Track flaky test metrics over time
- Auto-retry with limit (e.g., retry once)
- Require test owners to fix within SLA
- Run flaky tests in separate pipeline to avoid blocking main CI

---

## 2. JENKINS

### Basic
**Q: What is Jenkins? How does it work?**
A: Jenkins is an open-source automation server for CI/CD. It works by:
- Defining jobs/pipelines
- Triggering builds (SCM poll, webhook, cron, manual)
- Executing steps on agents (master distributes work to agents)
- Reporting results

**Q: What is the difference between Declarative and Scripted pipelines?**
A:
- Declarative: Structured, opinionated syntax with `pipeline { }` block. Easier to read, built-in validation. Has `stages`, `steps`, `post` sections.
- Scripted: Full Groovy DSL with `node { }` block. More flexible but harder to maintain. Uses `try/catch` for error handling.

```groovy
// Declarative
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
    post {
        failure { mail to: 'team@co.com', subject: 'Build failed' }
    }
}

// Scripted
node {
    try {
        stage('Build') {
            sh 'make build'
        }
    } catch (e) {
        mail to: 'team@co.com', subject: 'Build failed'
        throw e
    }
}
```

**Q: What is a Jenkins agent?**
A: A machine that executes builds dispatched by the Jenkins controller.
- Types: Permanent agents (always connected), Cloud agents (spun up on demand - Docker, K8s, EC2)
- The controller should ideally not run builds itself (security + performance)
- Agents connect via SSH, JNLP, or WebSocket

### Intermediate
**Q: What are Jenkins shared libraries?**
A: Reusable Groovy code stored in a Git repo that can be loaded into any pipeline. Used to:
- Standardize pipeline logic across teams
- Avoid copy-pasting pipeline code
- Enforce org policies (security scanning, approval gates)

Structure:
```
(root)
├── vars/          # Global variables/functions callable from pipelines
│   └── myPipeline.groovy
├── src/           # Helper classes (OOP Groovy)
│   └── org/co/Utils.groovy
└── resources/     # Non-Groovy files (configs, templates)
```

Usage in pipeline:
```groovy
@Library('my-shared-lib') _
myPipeline(appName: 'myapp', deployEnv: 'staging')
```

**Q: How do you manage credentials/secrets in Jenkins?**
A:
- Jenkins Credentials Store (built-in, encrypted at rest)
- Types: Username/Password, SSH key, Secret text, Secret file, Certificate
- Access in pipeline: `withCredentials([...]) { }` block
- Never print credentials in logs: Jenkins masks them but be careful with `echo`
- For better security: integrate with HashiCorp Vault or Azure Key Vault

**Q: How do you run stages in parallel in Jenkins?**
A:
```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'make unit-test' }
        }
        stage('Integration Tests') {
            steps { sh 'make integration-test' }
        }
        stage('Lint') {
            steps { sh 'make lint' }
        }
    }
}
```

### Advanced
**Q: How do you optimize Jenkins pipeline performance?**
A:
- Parallel stages for independent tasks
- Docker layer caching
- Dependency caching (node_modules, pip cache, Maven local repo)
- Lightweight checkout (shallow clone: `depth: 1`)
- Stash/unstash for sharing files between stages
- Use agents with SSDs, adequate memory
- Pipeline durability settings (PERFORMANCE_OPTIMIZED for non-critical)
- Limit console log size

**Q: What is Jenkinsfile? Best practices?**
A:
- Pipeline-as-code file stored in the repo root
- Best practices:
  - Keep it declarative when possible
  - Use shared libraries for reusable logic
  - Don't hardcode values, use parameters
  - Use `timeout` and `retry` for resilience
  - Clean workspace after build
  - Use `when` conditions for conditional stages
  - Keep stages focused (single responsibility)

---

## 3. GIT (Strong knowledge required per JD)

### Basic
**Q: What is Git? How is it different from GitHub/Bitbucket/Azure Repos?**
A: Git is a distributed version control system. GitHub/Bitbucket/Azure Repos are hosting platforms that provide Git + collaboration features (PRs, issues, CI/CD).

**Q: What is the difference between merge and rebase?**
A:
- Merge: Creates a merge commit combining two branches. Preserves full history. Non-destructive.
- Rebase: Moves/replays your commits on top of another branch. Creates linear history. Rewrites commit hashes.
- Rule: Never rebase commits that have been pushed to shared branches.

### Intermediate
**Q: What is `git cherry-pick`? When would you use it?** ⚠️ LEARN THIS
A: Applies a specific commit from one branch to another without merging the entire branch.
```bash
git cherry-pick <commit-hash>
```
Use cases:
- Hotfix: Pick a bug fix from develop into a release branch
- Selective backporting: Apply specific features to older versions
- Recover from wrong branch: You committed to wrong branch, cherry-pick to correct one

**Q: What is `git bisect`?** ⚠️ LEARN THIS
A: Binary search through commits to find which commit introduced a bug.
```bash
git bisect start
git bisect bad              # current commit is bad
git bisect good <commit>    # known good commit
# Git checks out middle commit, you test and mark:
git bisect good   # or
git bisect bad
# Repeat until the culprit commit is found
git bisect reset            # go back to original state
```

**Q: What is `git stash`?** ⚠️ LEARN THIS
A: Temporarily saves uncommitted changes so you can switch branches cleanly.
```bash
git stash                  # save changes
git stash list             # see all stashes
git stash pop              # apply latest stash and remove it
git stash apply stash@{2}  # apply specific stash (keep it)
git stash drop stash@{0}   # delete a stash
```

**Q: What are Git hooks?** ⚠️ LEARN THIS
A: Scripts that run automatically at certain Git events. Stored in `.git/hooks/`.
- Client-side: `pre-commit` (lint/format), `commit-msg` (validate message), `pre-push` (run tests)
- Server-side: `pre-receive` (enforce policies), `post-receive` (trigger CI)
- Tools like Husky (JS) or pre-commit (Python) make hook management easier

**Q: What branching strategy do you follow?**
A:
- **GitFlow**: main, develop, feature/*, release/*, hotfix/*. Good for scheduled releases.
- **Trunk-based**: Everyone commits to main (or short-lived branches). Requires feature flags. Good for continuous deployment.
- **GitHub Flow**: main + feature branches + PRs. Simple, works well for web apps.
- At my current company, I've worked with feature branching with PRs and CI validation before merge.

### Advanced
**Q: What is `git reflog`? How do you recover lost commits?**
A: Reflog records every HEAD movement (commits, checkouts, rebases, resets). Even "lost" commits are recoverable.
```bash
git reflog                       # see history of HEAD changes
git checkout <lost-commit-hash>  # recover
git branch recovery-branch       # save it
```

**Q: What are Git submodules? When would you use them?**
A: Submodules embed another Git repo inside your repo at a specific commit.
```bash
git submodule add <url> <path>
git submodule update --init --recursive
```
Use case: Shared libraries, third-party deps. Downside: Complex workflow, easy to get out of sync.
Alternative: Git subtree (copies code into repo, simpler but no upstream tracking).

**Q: What is Gerrit? How is it different from GitHub PRs?** ⚠️ LEARN THIS
A: Gerrit is a code review tool used heavily in large projects (Android, Chromium).
Key differences from GitHub:
- Push to `refs/for/<branch>` instead of creating a PR
- Each commit is reviewed individually (not a branch)
- Uses +1/+2 scoring system (+2 = approved, submit)
- Integrates with Google Repo for multi-repo management
- Rebases are preferred over merge commits
- Changes are amended (same Change-Id) rather than adding new commits

**Q: What is Google Repo?** ⚠️ LEARN THIS
A: A tool built by Google to manage multiple Git repositories as a single project.
- Uses a **manifest XML file** listing all repos, branches, and paths
- `repo init` + `repo sync` to download all repos
- `repo forall -c <command>` to run commands across all repos
- Used in Android (AOSP), Chromium, and embedded systems
- Why Ciena uses it: Optical Network software likely spans many repos (firmware, drivers, apps, tools)

---

## 4. DOCKER

### Basic
**Q: Containers vs VMs?**
A:
- VMs: Full OS per VM, hypervisor, heavy, minutes to start, GB-sized
- Containers: Share host kernel, lightweight, seconds to start, MB-sized
- Containers provide process isolation, not full OS isolation

**Q: What is a Dockerfile? Walk through key instructions.**
A:
- FROM: Base image
- WORKDIR: Set working directory
- COPY/ADD: Copy files (COPY preferred; ADD can extract tars, fetch URLs)
- RUN: Execute commands during build
- ENV: Set environment variables
- EXPOSE: Document port (doesn't actually publish)
- CMD: Default command when container starts
- ENTRYPOINT: Main executable (CMD becomes arguments to ENTRYPOINT)

### Intermediate
**Q: ENTRYPOINT vs CMD?** ⚠️ IMPORTANT
A:
- CMD: Default command, easily overridden at `docker run`
- ENTRYPOINT: Fixed executable, not easily overridden
- Combined: ENTRYPOINT = executable, CMD = default args
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run myimage         → python app.py
# docker run myimage test.py → python test.py
```

**Q: What is a multi-stage build? Why use it?**
A: Multiple FROM statements in one Dockerfile. Each stage can copy artifacts from previous stages.
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (only runtime, no build tools)
FROM node:18-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```
Benefits: Smaller images (no build tools), separate build/runtime concerns, security (fewer packages = fewer vulnerabilities).

**Q: Docker security best practices?**
A:
- Run as non-root user (`USER 1001`)
- Use specific image tags, not `latest`
- Use distroless or slim base images
- Scan images (Snyk, Trivy)
- Don't store secrets in images (use runtime secrets)
- Use `.dockerignore` to exclude sensitive files
- Read-only filesystem where possible

### Advanced
**Q: How do you optimize Docker build time?**
A:
- Order instructions from least to most frequently changing (dependencies before source code)
- Use BuildKit (`DOCKER_BUILDKIT=1`)
- Cache mounts for package managers: `RUN --mount=type=cache,target=/root/.cache pip install -r requirements.txt`
- Multi-stage builds to parallelize independent stages
- Use `.dockerignore` to reduce build context size

---

## 5. KUBERNETES

### Basic
**Q: What is Kubernetes? Why do we need it?**
A: Container orchestration platform. Handles:
- Scheduling containers across nodes
- Self-healing (restart failed containers)
- Scaling (horizontal pod autoscaler)
- Service discovery and load balancing
- Rolling updates and rollbacks
- Secret and config management

**Q: Pod vs Deployment vs Service?**
A:
- Pod: Smallest deployable unit. One or more containers sharing network/storage.
- Deployment: Manages desired state of pods (replicas, update strategy). Creates ReplicaSets.
- Service: Stable network endpoint for pods. Types: ClusterIP (internal), NodePort (external port), LoadBalancer (cloud LB).

### Intermediate
**Q: What are liveness and readiness probes?**
A:
- Liveness: "Is the container alive?" If fails, kubelet restarts the container.
- Readiness: "Is the container ready to serve traffic?" If fails, removed from Service endpoints.
- Startup: For slow-starting apps. Disables liveness/readiness until startup succeeds.
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Q: ConfigMap vs Secret?**
A:
- ConfigMap: Non-sensitive configuration data (key-value pairs, config files)
- Secret: Sensitive data (passwords, tokens). Base64 encoded (NOT encrypted by default!)
- Both can be mounted as files or exposed as env vars
- For real security: use encrypted Secrets (etcd encryption), or external secret managers (Vault, Azure Key Vault)

**Q: What is an Ingress?**
A: Layer 7 (HTTP/HTTPS) load balancer for K8s services.
- Needs an Ingress Controller (nginx, traefik, ALB)
- Provides: path-based routing, host-based routing, TLS termination
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### Advanced
**Q: How do you troubleshoot a CrashLoopBackOff?**
A:
1. `kubectl describe pod <name>` - check Events section for errors
2. `kubectl logs <pod> --previous` - see logs from crashed container
3. Check: Exit code, OOMKilled (memory limits too low), missing config/secrets, liveness probe failing too early
4. `kubectl exec -it <pod> -- /bin/sh` (if container runs long enough)
5. Temporarily set `command: ["sleep", "3600"]` to get a shell for debugging

**Q: Resource requests vs limits?**
A:
- Requests: Guaranteed resources. Scheduler uses this to place pods.
- Limits: Maximum resources. Container is throttled (CPU) or killed (memory OOM) if exceeded.
- Best practice: Set requests = typical usage, limits = peak usage. Never set limits without requests.

---

## 6. PYTHON

### Basic
**Q: Write a Python script to read a file and count word frequency.**
A:
```python
from collections import Counter

with open('file.txt', 'r') as f:
    words = f.read().split()

counter = Counter(words)
for word, count in counter.most_common(10):
    print(f"{word}: {count}")
```

### Intermediate
**Q: How do you run shell commands from Python?** ⚠️ LEARN THIS
A:
```python
import subprocess

# Run and capture output
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)

# With error handling
try:
    result = subprocess.run(['git', 'status'], capture_output=True, text=True, check=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed: {e.stderr}")
```

**Q: How do you make REST API calls in Python?**
A:
```python
import requests

# GET
resp = requests.get('https://api.example.com/data', headers={'Authorization': 'Bearer TOKEN'})
resp.raise_for_status()
data = resp.json()

# POST
resp = requests.post('https://api.example.com/data', json={'key': 'value'})
```

### Advanced
**Q: Write a Python script to monitor a log file for errors and alert.**
A:
```python
import subprocess
import time
import os

def tail_log(filepath, keyword='ERROR'):
    with open(filepath, 'r') as f:
        f.seek(0, os.SEEK_END)  # Go to end of file
        while True:
            line = f.readline()
            if not line:
                time.sleep(0.5)
                continue
            if keyword in line:
                print(f"ALERT: {line.strip()}")
                # Could send email/Slack notification here

tail_log('/var/log/app.log')
```

---

## 7. LINUX COMMANDS ⚠️ CRITICAL GAP - STUDY HARD

### Basic
```bash
# File operations
ls -la          # list with details + hidden files
chmod 755 file  # rwxr-xr-x
chown user:group file
ln -s target link  # symbolic link

# Viewing files
cat file        # entire file
head -20 file   # first 20 lines
tail -f file    # follow file in real-time (great for logs)
less file       # paginated view
wc -l file      # count lines
```

### Intermediate ⚠️ LEARN THESE
```bash
# grep (search text in files)
grep "error" file.log              # basic search
grep -r "TODO" ./src/              # recursive search
grep -i "error" file.log           # case-insensitive
grep -n "error" file.log           # show line numbers
grep -c "error" file.log           # count matches
grep -E "error|warn|fatal" file.log # regex (multiple patterns)
grep -v "debug" file.log           # invert (exclude debug lines)

# awk (text processing)
awk '{print $1, $3}' file          # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd   # custom delimiter
awk '$3 > 100 {print $1}' file    # conditional
df -h | awk '$5 > 80 {print $0}'   # disks over 80% used

# sed (stream editor)
sed 's/old/new/' file              # replace first occurrence per line
sed 's/old/new/g' file             # replace all occurrences
sed -i 's/old/new/g' file         # in-place edit
sed -n '10,20p' file               # print lines 10-20
sed '/pattern/d' file              # delete lines matching pattern

# find
find /var/log -name "*.log" -mtime +7    # files older than 7 days
find . -type f -size +100M               # files over 100MB
find . -name "*.tmp" -exec rm {} \;      # find and delete
find . -name "*.py" | xargs grep "import" # find + search
```

### Process Management ⚠️ LEARN
```bash
ps aux                    # all processes
ps aux | grep nginx       # find specific process
top / htop                # real-time process monitor
kill <PID>                # graceful stop (SIGTERM)
kill -9 <PID>             # force kill (SIGKILL)
nohup ./script.sh &       # run in background, survive logout
jobs                      # list background jobs
systemctl start nginx     # start service
systemctl enable nginx    # start on boot
systemctl status nginx    # check status
journalctl -u nginx -f    # follow logs for a service
```

### Networking ⚠️ LEARN
```bash
curl -v https://api.example.com           # verbose HTTP request
wget https://example.com/file.tar.gz      # download file
ss -tlnp                                  # listening ports (modern netstat)
netstat -tulnp                            # listening ports (legacy)
dig example.com                           # DNS lookup
nslookup example.com                      # DNS lookup (simpler)
ping -c 4 host                            # connectivity check
traceroute host                           # trace network path
ip addr show                              # show IP addresses
```

### Disk & Storage
```bash
df -h              # disk space usage
du -sh /var/log    # directory size
lsblk              # list block devices
mount /dev/sdb1 /mnt  # mount filesystem
```

---

## 8. GO & YOCTO (Awareness Level) ⚠️ LEARN CONCEPTS

### Go (Golang)
- Created by Google, compiled language
- Key features: Fast compilation, built-in concurrency (goroutines), garbage collected, static typing
- Used for: CLI tools (Docker, Kubernetes, Terraform are written in Go), microservices, system tools
- Why Ciena cares: Many DevOps tools are written in Go; may need to build/extend tools
- Basic syntax awareness:
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```
- Build: `go build`, `go run main.go`, `go test`
- Module system: `go mod init`, `go.mod` file
- Cross-compilation: `GOOS=linux GOARCH=amd64 go build` (great for embedded targets)

### Yocto Project ⚠️ IMPORTANT FOR THIS ROLE
- **What**: A build system for creating custom embedded Linux distributions
- **Why Ciena uses it**: Their optical networking hardware runs custom Linux - Yocto builds the OS image
- **Key concepts**:
  - **Recipe (.bb files)**: Instructions to build a single software package
  - **Layer**: Collection of recipes (meta-layers like `meta-ciena`)
  - **BitBake**: The build engine (like Make but for entire Linux distros)
  - **Poky**: Reference distribution (starting point)
  - **Image**: The final output (bootable Linux image for target hardware)
  - **BSP (Board Support Package)**: Hardware-specific layer (kernel, bootloader, drivers)
- **Workflow**: Write/modify recipes → BitBake builds packages → Assemble into image → Flash to device
- **Common commands**:
```bash
source oe-init-build-env    # setup build environment
bitbake core-image-minimal  # build a minimal Linux image
bitbake -c menuconfig virtual/kernel  # configure kernel
```
- **Why this matters**: As DevOps for ON team, you'll likely automate Yocto builds in Jenkins/CI

---

## 9. GERRIT & GOOGLE REPO (Awareness)

### Gerrit Code Review
- Web-based code review tool for Git
- Workflow: Code → Push to `refs/for/main` → Review → Score (+1/+2) → Submit
- Every commit gets a unique Change-Id
- Amend commits instead of adding new ones (different from GitHub)
- Integrates with Jenkins for CI (Gerrit trigger plugin)

### Google Repo
- Manages multiple Git repositories as one project
- Uses a manifest.xml:
```xml
<manifest>
  <remote name="origin" fetch="https://git.example.com" />
  <default remote="origin" revision="main" />
  <project name="firmware" path="src/firmware" />
  <project name="userspace" path="src/userspace" />
  <project name="tools" path="tools" />
</manifest>
```
- Commands: `repo init -u <manifest-url>`, `repo sync`, `repo forall -c 'git status'`

---

## 10. DEVOPS SCENARIO QUESTIONS

**Q: A production deployment failed. Walk through your incident response.**
A:
1. **Assess impact**: What's broken? How many users affected?
2. **Rollback**: If possible, rollback to last known good version immediately
3. **Communicate**: Notify stakeholders (Slack channel, status page)
4. **Diagnose**: Check deployment logs, application logs, monitoring dashboards
5. **Fix**: Identify root cause, apply fix, test in staging
6. **Deploy**: Push fix through normal CI/CD pipeline
7. **Post-mortem**: Blameless RCA, document what happened, add monitoring/tests to prevent recurrence

**Q: How do you ensure CI/CD pipeline security (DevSecOps)?**
A:
- SAST (Static Analysis): SonarQube, Semgrep in build stage
- Dependency scanning: Snyk, Dependabot, Mend
- Container scanning: Trivy, Snyk Container
- Secret detection: git-secrets, trufflehog in pre-commit hooks
- RBAC: Least privilege for pipeline service accounts
- Signed artifacts: Image signing with Cosign/Notary
- Audit logs: Track who deployed what and when

**Q: How do you monitor infrastructure health?**
A:
- Metrics: Prometheus (collection) + Grafana (dashboards)
- Logs: Loki / ELK stack
- Traces: Jaeger / Kiali (service mesh)
- Alerts: PagerDuty / Alertmanager
- Key metrics: CPU, memory, disk, request latency, error rate, deployment frequency
- SLIs/SLOs for reliability targets
