# Azure DevOps - COMPREHENSIVE ANSWERS (All 55 Questions)

---

## Azure Pipelines YAML

**1. Structure of Azure Pipeline YAML?**
```yaml
trigger:
  branches:
    include: [main, develop]
  paths:
    include: [src/*, Dockerfile]

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfig: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: dotnet build
      displayName: 'Build application'
```

**2. Stage vs Job vs Step?**

```
Azure Pipeline Hierarchy:
┌───────────────────────────────────────────────────────────┐
│ Pipeline                                                    │
│                                                             │
│  ┌──────────────────┐   ┌──────────────────┐   ┌─────────────┐  │
│  │ Stage: Build      │─▶│ Stage: Test       │─▶│Stage: Deploy│  │
│  │                    │   │                    │   │             │  │
│  │ ┌──────────────┐ │   │ ┌────┐ ┌─────┐ │   │ ┌─────────┐ │  │
│  │ │ Job: Build    │ │   │ │Job1│ │Job2 │ │   │ │ Deploy  │ │  │
│  │ │              │ │   │ │Unit│ │Integ│ │   │ │ Job     │ │  │
│  │ │ Step: Build │ │   │ │Test│ │Test │ │   │ │         │ │  │
│  │ │ Step: Push  │ │   │ └────┘ └─────┘ │   │ │ Step x3 │ │  │
│  │ └──────────────┘ │   │ (parallel!)       │   │ └─────────┘ │  │
│  └──────────────────┘   └──────────────────┘   └─────────────┘  │
│  (sequential)              (parallel jobs)       (approval)  │
└───────────────────────────────────────────────────────────┘
```

- **Stage**: Logical division (Build, Test, Deploy). Contains jobs. Can have approval gates.
- **Job**: Unit of work running on one agent. Contains steps. Jobs in same stage can run in parallel.
- **Step**: Individual command/task. Runs sequentially within a job.

**3. Trigger types?**
```yaml
# CI trigger (on push)
trigger:
  branches:
    include: [main]

# PR trigger
pr:
  branches:
    include: [main]

# Scheduled
schedules:
- cron: "0 2 * * *"
  displayName: "Nightly build"
  branches:
    include: [main]

# Manual (no trigger)
trigger: none
```

**4. Path filters in triggers?**
```yaml
trigger:
  branches:
    include: [main]
  paths:
    include:
    - src/*
    - Dockerfile
    exclude:
    - docs/*
    - '*.md'
```

**5. Pool types?**
```yaml
# Microsoft-hosted (managed by Azure)
pool:
  vmImage: 'ubuntu-latest'    # Also: windows-latest, macos-latest

# Self-hosted (your own machines)
pool:
  name: 'MyPool'
  demands:
  - docker
  - Agent.OS -equals Linux
```
**Microsoft-hosted**: Pre-configured, no maintenance, fresh each run, limited free minutes.
**Self-hosted**: Full control, persistent, custom tools, no time limits, you maintain.

**6. Define variables?**
```yaml
# Inline
variables:
  buildConfig: 'Release'
  IMAGE_TAG: '$(Build.BuildId)'

# Variable group
variables:
- group: my-variable-group

# Key Vault linked
variables:
- group: keyvault-linked-group

# Runtime (settable at queue time)
parameters:
- name: environment
  type: string
  default: 'staging'
  values: ['dev', 'staging', 'prod']
```

**7. Variable group? Link Azure Key Vault?**
Variable groups store shared variables across pipelines. Create in Library → Variable Groups.
Link Key Vault: Variable Group → "Link secrets from Azure Key Vault" → select vault → authorize.

**8. Use secrets in Azure Pipelines?**
```yaml
# Secret variable (can't echo in logs)
variables:
- name: mySecret
  value: $(secret-from-group)

steps:
- script: |
    echo "Using secret"
    # $MYSECRET is auto-mapped as env var
  env:
    MYSECRET: $(mySecret)
```
Secrets are masked in logs. Can't be passed across stages without explicit mapping.

**9. Pipeline templates?**
```yaml
# templates/build-template.yml
parameters:
- name: project
  type: string
- name: configuration
  type: string
  default: 'Release'

steps:
- script: dotnet build ${{ parameters.project }} -c ${{ parameters.configuration }}
  displayName: Build ${{ parameters.project }}
```
```yaml
# azure-pipelines.yml
steps:
- template: templates/build-template.yml
  parameters:
    project: src/MyApp.csproj
```

**10. `extends` vs `include` templates?**
- **extends**: Pipeline MUST extend from a template. Template controls structure. Used for enforcing security/compliance.
- **include/template**: Reusable snippet. Caller decides where to insert.
```yaml
# extends (enforced by org)
extends:
  template: templates/secure-pipeline.yml
  parameters:
    stages: [...]
```

**11. Parameterize templates?**
```yaml
# Parameters: compile-time, type-checked, support objects/steps
parameters:
- name: env
  type: string
  values: ['dev', 'staging', 'prod']
- name: extraSteps
  type: stepList
  default: []

# Variables: runtime, string only
variables:
  buildConfig: 'Release'
```

**12. Multi-stage pipeline?**
```yaml
stages:
- stage: Build
  jobs:
  - job: BuildApp
    steps:
    - script: dotnet build
    - task: PublishPipelineArtifact@1
      inputs:
        artifactName: drop

- stage: DeployStaging
  dependsOn: Build
  jobs:
  - deployment: DeployStg
    environment: staging
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to staging"

- stage: DeployProd
  dependsOn: DeployStaging
  jobs:
  - deployment: DeployPrd
    environment: production    # Has approval gate
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to prod"
```

**13. Manual approval gates?**
Configure on the **Environment** in Azure DevOps:
Pipelines → Environments → production → Approvals and checks → Add → Approvals → Select approvers.
Also supports: business hours, branch control, required template.

**14. Deployment job vs regular job?**
```yaml
# Regular job
jobs:
- job: BuildJob
  steps: [...]

# Deployment job (tracks deployments to environments)
jobs:
- deployment: DeployWeb
  environment: production        # Linked to environment
  strategy:
    runOnce:                     # Also: rolling, canary
      deploy:
        steps:
        - script: echo "Deploying"
```

```
Deployment Strategies in Azure Pipelines:

  runOnce:                rolling:               canary:
  ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
  │ Deploy ALL    │       │ Deploy 25%   │       │ Deploy 10%   │
  │ at once       │       │ then 50%     │       │ Monitor      │
  │               │       │ then 100%    │       │ Promote/Reject│
  │ Simple        │       │              │       │              │
  │ Risky for     │       │ Gradual      │       │ Safest       │
  │ big changes   │       │ Less risk    │       │ Most complex │
  └──────────────┘       └──────────────┘       └──────────────┘
```

Deployment jobs: support strategies (runOnce/rolling/canary), track history, link to environments with checks.

**15. Environments? Configure approvals?**
Environments represent deployment targets (staging, production). Benefits: deployment history, approval gates, exclusive locks.
Configure: Pipelines → Environments → [env] → Approvals and checks.

**16. `condition` in pipeline? 3 examples.**
```yaml
# 1. Run only on main branch
- stage: Deploy
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))

# 2. Run even if previous stage failed
- job: Cleanup
  condition: always()

# 3. Run only if variable is set
- step:
  condition: ne(variables['skipTests'], 'true')
  script: dotnet test
```

**17. Stages/jobs in parallel?**
```yaml
# Parallel stages (no dependsOn)
stages:
- stage: UnitTests
  jobs: [...]
- stage: IntegrationTests
  jobs: [...]
# Both run in parallel

# Parallel jobs within a stage (default)
jobs:
- job: TestLinux
  pool: { vmImage: 'ubuntu-latest' }
- job: TestWindows
  pool: { vmImage: 'windows-latest' }
```

**18. `dependsOn` for ordering?**
```yaml
stages:
- stage: Build
- stage: Test
  dependsOn: Build
- stage: DeployStaging
  dependsOn: Test
- stage: DeployProd
  dependsOn: DeployStaging

# Fan-out/fan-in
- stage: IntTests
  dependsOn: Build
- stage: PerfTests
  dependsOn: Build
- stage: Deploy
  dependsOn:
  - IntTests
  - PerfTests    # Waits for both
```

**19. Service connection? Name 5.**
Secure link between Azure DevOps and external services:
1. **Azure Resource Manager** — deploy to Azure
2. **Docker Registry** — push/pull images (ACR, Docker Hub)
3. **Kubernetes** — deploy to K8s clusters
4. **GitHub** — access repos
5. **SSH** — connect to remote servers

**20. Publish and consume pipeline artifacts?**
```yaml
# Publish
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: $(Build.ArtifactStagingDirectory)
    artifactName: drop

# Consume (in later stage)
- task: DownloadPipelineArtifact@2
  inputs:
    artifactName: drop
    downloadPath: $(Pipeline.Workspace)/drop
```

**21. Pipeline artifacts vs build artifacts?**
- **Pipeline artifacts**: Newer, faster. Use `PublishPipelineArtifact@1`. Stored in Azure Pipelines.
- **Build artifacts**: Legacy. Use `PublishBuildArtifacts@1`. Stored in Azure DevOps.
**Always use Pipeline artifacts** for new pipelines.

**22. Cache dependencies?**
```yaml
- task: Cache@2
  inputs:
    key: 'pip | "$(Agent.OS)" | requirements.txt'
    path: $(PIP_CACHE_DIR)
    restoreKeys: |
      pip | "$(Agent.OS)"
  displayName: Cache pip packages
```

**23. Checkout step? Shallow checkout?**
```yaml
steps:
- checkout: self
  fetchDepth: 1              # Shallow clone (faster)
  clean: true

# Multiple repos
- checkout: self
- checkout: git://MyProject/OtherRepo@main
```

**24. Docker in Azure Pipelines?**
```yaml
- task: Docker@2
  inputs:
    containerRegistry: 'myACR'
    repository: 'myapp'
    command: 'buildAndPush'
    Dockerfile: '**/Dockerfile'
    tags: |
      $(Build.BuildId)
      latest
```

**25. Deploy to Kubernetes?**
```yaml
- task: KubernetesManifest@0
  inputs:
    action: deploy
    kubernetesServiceConnection: 'myAKS'
    namespace: production
    manifests: |
      k8s/deployment.yaml
      k8s/service.yaml
    containers: |
      myacr.azurecr.io/myapp:$(Build.BuildId)
```

---

## Built-in Tasks

**26. UsePythonVersion, UseDotNet, UseNode?**
```yaml
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.11'

- task: UseDotNet@2
  inputs:
    version: '8.x'

- task: UseNode@1
  inputs:
    version: '20.x'
```

**27. PublishTestResults?**
```yaml
- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: '**/test-results.xml'
    mergeTestResults: true
  condition: always()    # Publish even if tests fail
```

**28. PublishCodeCoverageResults?**
```yaml
- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(System.DefaultWorkingDirectory)/**/coverage.xml'
```

**29. AzureRmWebAppDeployment?**
```yaml
- task: AzureRmWebAppDeployment@4
  inputs:
    ConnectionType: 'AzureRM'
    azureSubscription: 'my-subscription'
    appType: 'webApp'
    WebAppName: 'my-web-app'
    packageForLinux: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

**30. Run custom scripts?**
```yaml
# Bash
- bash: |
    echo "Running bash"
    pip install -r requirements.txt

# PowerShell Core (cross-platform)
- pwsh: |
    Write-Host "PowerShell Core"

# Windows PowerShell
- powershell: |
    Write-Host "Windows PowerShell"

# Script (cross-platform, uses bash on Linux, cmd on Windows)
- script: echo "Hello"
  displayName: 'Run greeting'
```

---

## Azure Repos

**31. Azure Repos? Git vs TFVC?**
- **Git**: Distributed VCS. Full local copy. Branches are cheap. **Standard**.
- **TFVC**: Centralized VCS. Legacy. Only for old projects.

**32. Set up branch policies?**
Project Settings → Repos → Policies → select branch → configure policies.
OR: Repos → Branches → ... → Branch policies.

**33. Branch policies?**
- **Minimum reviewers**: Require N approvals on PRs
- **Build validation**: PR must pass CI build
- **Comment resolution**: All comments must be resolved
- **Work item linking**: PR must link to work item
- **Status checks**: External service approval
- **Merge strategy**: Squash, no fast-forward, etc.

**34. Enforce PR-based workflow?**
Set branch policy on main/develop: require minimum 1 reviewer + build validation + no direct pushes. This forces all changes through PRs.

**35. PR template?**
Create `.azuredevops/pull_request_template.md` in repo root:
```markdown
## Description
## Type of Change
- [ ] Bug fix
- [ ] New feature
## Testing
## Checklist
- [ ] Tests added
- [ ] Documentation updated
```

---

## Azure Boards

**36. Azure Boards?**
Work tracking tool: backlogs, sprints, Kanban boards. Integrates with Repos and Pipelines.

**37. Work item types?**
- **Epic**: Large initiative spanning multiple sprints
- **Feature**: Distinct functionality (child of Epic)
- **User Story**: End-user requirement (child of Feature)
- **Task**: Specific work item (child of Story)
- **Bug**: Defect to fix

**38. Link work items to PRs/commits?**
```bash
# In commit message
git commit -m "Fix login bug AB#1234"

# In PR description
Related to AB#1234
Fixes AB#5678
```
Work items auto-update state when PR is merged (if configured).

**39. Sprint/iteration?**
Time-boxed period (usually 2 weeks) for completing work items. Configure: Project Settings → Boards → Team Configuration → Iterations.

---

## Azure Artifacts

**40. Azure Artifacts?**
Package management service. Host private packages alongside public ones.

**41. Feed types?**
npm, NuGet, Maven, Python (pip), Universal Packages. Each feed can host multiple package types.

**42. Publish and consume packages?**
```yaml
# Publish Python package
- task: TwineAuthenticate@1
  inputs:
    artifactFeed: my-feed
- script: |
    pip install twine
    twine upload -r my-feed dist/*

# Consume (pip.conf or pip install --index-url)
```

**43. Upstream sources?**
Proxy to public registries (npmjs.com, pypi.org, nuget.org). Packages fetched from public registry are cached in your feed. Provides: single source for all packages, caching, vulnerability scanning.

---

## Comparison & Migration

**44. Azure DevOps vs GitHub Actions?**

```
Azure DevOps                            GitHub Actions
┌─────────────────────────┐          ┌─────────────────────────┐
│ Boards (work tracking)  │          │ Issues + Projects       │
│ Repos (Azure Git)       │          │ Repositories (GitHub)   │
│ Pipelines (CI/CD)       │   vs     │ Actions (CI/CD)         │
│ Artifacts (packages)    │          │ Packages                │
│ Test Plans              │          │ (no built-in)           │
└─────────────────────────┘          └─────────────────────────┘

  Pipeline YAML:                Workflow YAML:
  trigger: [main]               on: push: branches: [main]
  stages:                       jobs:
  - stage: Build                  build:
    jobs:                           runs-on: ubuntu-latest
    - job: Build                    steps:
      steps:                        - uses: actions/checkout@v4
      - script: dotnet build        - run: dotnet build
```

| Azure DevOps | GitHub Actions |
|---|---|
| Stages → Jobs → Steps | Workflows → Jobs → Steps |
| YAML or Classic UI | YAML only |
| Variable groups, Library | Secrets, Variables |
| Environments + approvals | Environments + protection rules |
| Azure Boards built-in | GitHub Issues/Projects |
| Better enterprise features | Better open-source community |
| Service connections | Secrets + OIDC |

**45. Azure Pipelines vs Jenkins?**
| Azure Pipelines | Jenkins |
|---|---|
| SaaS (hosted) | Self-hosted |
| YAML pipelines | Jenkinsfile (Groovy) |
| Built-in agents | Must manage agents |
| Integrated with Azure | Plugin ecosystem |
| Less customizable | Highly extensible |
| Easier to start | More complex setup |

**46. Migrate from Jenkins to Azure Pipelines?**
1. Map Jenkinsfile stages to Azure YAML stages
2. Convert Groovy syntax to YAML
3. Replace Jenkins plugins with Azure tasks/scripts
4. Migrate credentials to Azure Key Vault / variable groups
5. Set up service connections for deployments
6. Run both in parallel during migration
7. Validate outputs match

**47. Migrate from Azure DevOps to GitHub Actions?**
1. Map stages/jobs/steps to workflows
2. Convert variable groups to GitHub Secrets/Variables
3. Replace Azure tasks with GitHub Actions (marketplace)
4. Migrate environments and approval gates
5. Move repos to GitHub
6. Update service connections to OIDC

---

## Interview-Style

**48. Complete pipeline: .NET → Docker → AKS?**
```yaml
trigger:
  branches:
    include: [main]

variables:
  acrName: 'myacr'
  imageName: 'myapp'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  jobs:
  - job: BuildAndTest
    pool: { vmImage: 'ubuntu-latest' }
    steps:
    - task: UseDotNet@2
      inputs: { version: '8.x' }
    - script: dotnet build -c Release
    - script: dotnet test --logger trx
    - task: PublishTestResults@2
      inputs: { testResultsFormat: VSTest, testResultsFiles: '**/*.trx' }
    - task: Docker@2
      inputs:
        containerRegistry: 'myACR'
        repository: '$(imageName)'
        command: buildAndPush
        tags: '$(tag)'

- stage: DeployStaging
  dependsOn: Build
  jobs:
  - deployment: DeployStg
    pool: { vmImage: 'ubuntu-latest' }
    environment: staging
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: deploy
              kubernetesServiceConnection: 'aks-staging'
              namespace: staging
              manifests: k8s/*.yaml
              containers: '$(acrName).azurecr.io/$(imageName):$(tag)'

- stage: DeployProd
  dependsOn: DeployStaging
  jobs:
  - deployment: DeployPrd
    pool: { vmImage: 'ubuntu-latest' }
    environment: production     # Manual approval configured here
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: deploy
              kubernetesServiceConnection: 'aks-prod'
              namespace: production
              manifests: k8s/*.yaml
              containers: '$(acrName).azurecr.io/$(imageName):$(tag)'
```

**49. Docker build+push template?**
```yaml
# templates/docker-build-push.yml
parameters:
- name: containerRegistry
  type: string
- name: repository
  type: string
- name: dockerfile
  type: string
  default: 'Dockerfile'
- name: buildContext
  type: string
  default: '.'

steps:
- task: Docker@2
  displayName: 'Build and Push'
  inputs:
    containerRegistry: ${{ parameters.containerRegistry }}
    repository: ${{ parameters.repository }}
    command: buildAndPush
    Dockerfile: ${{ parameters.dockerfile }}
    buildContext: ${{ parameters.buildContext }}
    tags: |
      $(Build.BuildId)
      latest
```

**50. IaC deployment with Terraform?**
```yaml
stages:
- stage: Plan
  jobs:
  - job: TerraformPlan
    steps:
    - task: TerraformInstaller@0
      inputs: { terraformVersion: '1.5.0' }
    - task: TerraformTaskV4@4
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: 'azure-conn'
        backendAzureRmResourceGroupName: 'tfstate-rg'
        backendAzureRmStorageAccountName: 'tfstateacct'
        backendAzureRmContainerName: 'tfstate'
        backendAzureRmKey: 'prod.tfstate'
    - task: TerraformTaskV4@4
      inputs:
        provider: 'azurerm'
        command: 'plan'
        environmentServiceNameAzureRM: 'azure-conn'

- stage: Apply
  dependsOn: Plan
  jobs:
  - deployment: TerraformApply
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
          - task: TerraformTaskV4@4
            inputs:
              command: 'apply'
              provider: 'azurerm'
              environmentServiceNameAzureRM: 'azure-conn'
```

**51. Pipeline takes 20 min — optimize?**
1. **Caching**: Cache dependencies (npm, pip, NuGet) → `Cache@2` task
2. **Shallow clone**: `fetchDepth: 1`
3. **Parallel jobs**: Split tests across multiple agents
4. **Docker layer caching**: Use cached layers
5. **Skip unnecessary stages**: Use conditions and path filters
6. **Self-hosted agents**: Faster, persistent cache, pre-installed tools
7. **Smaller images**: Multi-stage Docker builds
8. **Incremental builds**: Only build changed components

**52. Secrets for multi-environment deployments?**
- Create separate **variable groups** per environment: `vars-dev`, `vars-staging`, `vars-prod`
- Link each to its own **Key Vault**: `keyvault-dev`, `keyvault-prod`
- Reference appropriate group per stage:
```yaml
- stage: DeployDev
  variables:
  - group: vars-dev
- stage: DeployProd
  variables:
  - group: vars-prod
```

**53. Azure DevOps setup at your organization?**
"We use Azure DevOps for CI/CD with YAML multi-stage pipelines. Code is in Azure Repos with branch policies (2 reviewers, build validation). Pipelines build Docker images, push to ACR, deploy to AKS via KubernetesManifest task. Secrets from Azure Key Vault via variable groups. Infrastructure managed with Terraform in separate pipelines. We use environments with approval gates for production deployments."

**54. Pipeline randomly fails 'agent not available'?**
1. Check agent pool capacity (enough agents?)
2. Check if self-hosted agent is online and idle
3. Check agent demands vs capabilities
4. Check concurrent job limits (free tier: 1 parallel job)
5. Check if agent maintenance is happening
6. Check agent logs on the machine
7. Consider switching to Microsoft-hosted for reliability

**55. Enforce all code goes through pipeline?**
1. **Branch policies** on main: require build validation
2. **No direct pushes**: disable direct push to protected branches
3. **Required reviewers**: minimum 1-2 approvers
4. **Environment approvals**: manual gate before production
5. **Service connection restrictions**: only pipelines can deploy (not individuals)
6. **RBAC**: restrict who can modify pipelines
