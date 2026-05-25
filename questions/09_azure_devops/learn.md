# Azure DevOps — Deep-Dive Learning Guide

---

## 1. Azure DevOps Architecture

```
┌─────────────────── Azure DevOps Organization ───────────────────┐
│  org: https://dev.azure.com/myorg                               │
│                                                                  │
│  ┌──── Project 1 ───────────────────────────────────────────┐   │
│  │                                                           │   │
│  │  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌───────────┐ │   │
│  │  │  Boards │  │  Repos  │  │ Pipelines│  │ Artifacts │ │   │
│  │  │         │  │         │  │          │  │           │ │   │
│  │  │ Work    │  │ Git     │  │ Build    │  │ NuGet     │ │   │
│  │  │ Items   │  │ repos   │  │ Release  │  │ npm       │ │   │
│  │  │ Sprints │  │ PRs     │  │ YAML     │  │ PyPI      │ │   │
│  │  │ Queries │  │ Branches│  │ Agents   │  │ Maven     │ │   │
│  │  └─────────┘  └─────────┘  └──────────┘  └───────────┘ │   │
│  │                                                           │   │
│  │  ┌──────────────────────────────────────────────────┐    │   │
│  │  │  Test Plans (manual + automated test management) │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  └───────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──── Project 2 ──────────────────────────────────────────┐    │
│  │  (same structure, separate data)                         │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Azure Pipelines — YAML

### Complete Pipeline Template

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main, release/*]
    exclude: [feature/experimental]
  paths:
    include: [src/*, Dockerfile]
    exclude: [docs/*, README.md]

pr:
  branches:
    include: [main]
  autoCancel: true        # Cancel running PR builds when new push

pool:
  vmImage: 'ubuntu-latest'    # Microsoft-hosted agent
  # OR
  # name: 'Self-Hosted-Linux'  # Self-hosted agent pool

variables:
  - group: 'Production-Secrets'      # Variable group (from Library)
  - name: imageName
    value: 'myapp'
  - name: isMain
    value: $[eq(variables['Build.SourceBranch'], 'refs/heads/main')]

stages:
  - stage: Build
    displayName: 'Build & Test'
    jobs:
      - job: BuildJob
        displayName: 'Build Docker Image'
        steps:
          - task: Docker@2
            displayName: 'Build Image'
            inputs:
              command: build
              dockerfile: Dockerfile
              tags: |
                $(Build.BuildId)
                latest

          - script: |
              pytest tests/ --junitxml=test-results.xml --cov=src --cov-report=xml
            displayName: 'Run Tests'

          - task: PublishTestResults@2
            inputs:
              testResultsFiles: 'test-results.xml'
              testRunTitle: 'Unit Tests'

          - task: PublishCodeCoverageResults@2
            inputs:
              summaryFileLocation: 'coverage.xml'

  - stage: DeployStaging
    displayName: 'Deploy to Staging'
    dependsOn: Build
    condition: and(succeeded(), eq(variables.isMain, true))
    jobs:
      - deployment: DeployStaging
        displayName: 'Deploy to Staging'
        environment: 'staging'       # Environment with approvals
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  inputs:
                    action: deploy
                    namespace: staging
                    manifests: 'k8s/*.yaml'

  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployStaging
    condition: succeeded()
    jobs:
      - deployment: DeployProd
        displayName: 'Deploy to Production'
        environment: 'production'    # Requires manual approval
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  inputs:
                    action: deploy
                    namespace: production
                    manifests: 'k8s/*.yaml'
```

---

## 3. Pipeline Concepts Deep Dive

### Stages, Jobs, Steps

```
Pipeline
  └── Stage: Build                    (logical boundary)
  │     └── Job: compile              (runs on ONE agent)
  │     │     └── Step: checkout      (individual task)
  │     │     └── Step: npm install
  │     │     └── Step: npm build
  │     └── Job: test                 (parallel with compile if no dependency)
  │           └── Step: npm test
  └── Stage: Deploy                   (depends on Build stage)
        └── Job: deploy-staging
              └── Step: kubectl apply
```

### Triggers

```yaml
# ─── CI Trigger (push) ───
trigger:
  branches:
    include: [main]
  tags:
    include: ['v*']        # Trigger on version tags

# ─── PR Trigger ───
pr:
  branches:
    include: [main]
  drafts: false            # Don't trigger for draft PRs

# ─── Scheduled ───
schedules:
  - cron: '0 2 * * *'     # Daily at 2 AM UTC
    displayName: 'Nightly Build'
    branches:
      include: [main]
    always: true           # Run even if no changes

# ─── Pipeline Trigger (chain pipelines) ───
resources:
  pipelines:
    - pipeline: upstream
      source: 'Build-Pipeline'
      trigger:
        branches:
          include: [main]
```

### Conditions

```yaml
# Built-in conditions:
condition: succeeded()                                    # Default
condition: failed()                                       # Run on failure
condition: always()                                       # Run regardless
condition: canceled()                                     # Run if canceled

# Custom conditions:
condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
condition: and(succeeded(), ne(variables['Build.Reason'], 'PullRequest'))
condition: or(eq(variables['isRelease'], 'true'), eq(variables['isHotfix'], 'true'))
condition: contains(variables['Build.SourceBranch'], 'release')
```

---

## 4. Templates — Reusable Pipeline Code

```yaml
# templates/build-template.yml
parameters:
  - name: imageName
    type: string
  - name: dockerfile
    type: string
    default: 'Dockerfile'
  - name: runTests
    type: boolean
    default: true

steps:
  - task: Docker@2
    displayName: 'Build ${{ parameters.imageName }}'
    inputs:
      command: build
      dockerfile: ${{ parameters.dockerfile }}
      tags: $(Build.BuildId)

  - ${{ if parameters.runTests }}:
    - script: pytest tests/
      displayName: 'Run Tests'

# ─── Usage in main pipeline ───
# azure-pipelines.yml
stages:
  - stage: Build
    jobs:
      - job: BuildWeb
        steps:
          - template: templates/build-template.yml
            parameters:
              imageName: 'web-app'
              runTests: true
```

### Template Types

```
Step template    → reusable steps within a job
Job template     → reusable job with steps
Stage template   → reusable stage with jobs
Variable template → shared variables
```

---

## 5. Agents

```
┌─── Microsoft-Hosted Agents ──────────────────────────────────┐
│  Managed by Microsoft, fresh VM per build                     │
│  OS: ubuntu-latest, windows-latest, macOS-latest             │
│  Pros: zero maintenance, pre-installed tools                  │
│  Cons: cold start, limited free minutes, can't customize     │
│  Free: 1800 min/month (public), 1 parallel job (private)     │
└───────────────────────────────────────────────────────────────┘

┌─── Self-Hosted Agents ───────────────────────────────────────┐
│  Your own machines (VM, container, on-prem)                   │
│  Pros: faster (cached tools/deps), access to internal network│
│  Cons: you maintain them, security responsibility             │
│  Unlimited parallel jobs, no minute limits                    │
│                                                               │
│  Agent Pools:                                                 │
│    Default     → self-hosted agents                           │
│    Azure Pipelines → Microsoft-hosted                        │
│    Custom      → create your own pools                       │
└───────────────────────────────────────────────────────────────┘
```

```bash
# Install self-hosted agent (Linux)
mkdir agent && cd agent
curl -O https://vstsagentpackage.azureedge.net/agent/3.x/vsts-agent-linux-x64.tar.gz
tar xzf vsts-agent-linux-x64.tar.gz
./config.sh    # Configure with PAT token
./run.sh       # Run interactively
sudo ./svc.sh install && sudo ./svc.sh start  # Run as service
```

---

## 6. Environments & Approvals

```
┌─── Environment: staging ─────────────────────────────────────┐
│  Checks: none (auto-deploy)                                  │
│  Resources: K8s namespace "staging", VMs                     │
└───────────────────────────────────────────────────────────────┘

┌─── Environment: production ──────────────────────────────────┐
│  Checks:                                                      │
│    ✅ Manual approval (team leads)                           │
│    ✅ Business hours only (Mon-Fri 9am-5pm)                  │
│    ✅ Branch filter (only main)                              │
│    ✅ Required template (must use approved template)         │
│  Resources: K8s namespace "production"                       │
│  Lock: exclusive (one deployment at a time)                  │
└───────────────────────────────────────────────────────────────┘
```

---

## 7. Service Connections

```
Azure DevOps needs credentials to talk to external services:

┌─── Service Connections ──────────────────────────────────────┐
│                                                               │
│  Azure Resource Manager → deploy to Azure (service principal)│
│  Docker Registry        → push/pull images (ACR, Docker Hub) │
│  Kubernetes             → deploy to K8s cluster              │
│  GitHub                 → access GitHub repos                │
│  SSH                    → deploy to VMs                      │
│  Generic                → any REST API                       │
│                                                               │
│  Best practice: use Workload Identity Federation (no secrets)│
└───────────────────────────────────────────────────────────────┘
```

---

## 8. Azure Repos — Git Features

```
Branch Policies (protect main):
  ✅ Require minimum reviewers (1-2)
  ✅ Require linked work items
  ✅ Require build validation (CI must pass)
  ✅ Require comment resolution
  ✅ Enforce merge strategy (squash / semi-linear)
  ✅ Limit merge types (no force push)
  ✅ Automatically include reviewers (CODEOWNERS)

PR Workflow:
  Developer → creates PR → CI runs → reviewers approve → squash merge
```

---

## 9. Azure Artifacts

```
┌─── Feed ─────────────────────────────────────────────────────┐
│  Private package registry inside Azure DevOps                 │
│                                                               │
│  Supported:                                                   │
│    NuGet (.NET)                                               │
│    npm (Node.js)                                              │
│    PyPI (Python)                                              │
│    Maven/Gradle (Java)                                        │
│    Universal Packages (any files)                             │
│                                                               │
│  Upstream Sources:                                            │
│    npmjs.org → Azure Artifacts Feed → your pipeline           │
│    (caches packages, survives npmjs outage)                   │
│                                                               │
│  Retention: keep versions used by builds, delete old ones    │
└───────────────────────────────────────────────────────────────┘
```

---

## 10. Key Azure DevOps CLI / REST API

```bash
# ─── Azure CLI extension ───
az devops configure --defaults organization=https://dev.azure.com/myorg project=MyProject

# Pipelines
az pipelines run --name 'Build-Pipeline'
az pipelines build list --top 5 --status completed

# Repos
az repos pr create --source-branch feature --target-branch main --title "My PR"
az repos pr list --status active

# Work Items
az boards work-item create --type "User Story" --title "Deploy v2"
az boards work-item update --id 1234 --state "Done"

# REST API
curl -u :$PAT "https://dev.azure.com/myorg/myproject/_apis/build/builds?api-version=7.0"
```

---

## 11. Azure DevOps vs GitHub Actions vs Jenkins

| Feature | Azure DevOps | GitHub Actions | Jenkins |
|---------|-------------|----------------|---------|
| Pipelines | YAML | YAML | Groovy/YAML |
| Hosting | Cloud + self-hosted | Cloud + self-hosted | Self-hosted only |
| Built-in boards | ✅ Full ALM | ✅ Projects (basic) | ❌ (need Jira) |
| Artifacts | ✅ Built-in | ✅ Packages | ❌ (need Nexus) |
| Test Plans | ✅ Built-in | ❌ | ❌ |
| Templates | ✅ Extends/includes | ✅ Reusable workflows | ✅ Shared libraries |
| Cost | Free tier + paid | Free tier + paid | Free (infra cost) |
| Best for | Enterprise, Azure shops | GitHub-centric teams | Complex/legacy |
