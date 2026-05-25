# Azure DevOps - LEARNING MATERIAL

---

## Azure DevOps Services Overview

```mermaid
graph TD
    subgraph AzureDevOps [Azure DevOps]
        BOARDS[Azure Boards<br/>Work Items, Sprints]
        REPOS[Azure Repos<br/>Git Repositories]
        PIPES[Azure Pipelines<br/>CI/CD]
        TESTS[Azure Test Plans<br/>Manual/Automated Testing]
        ARTIFACTS[Azure Artifacts<br/>Package Management]
    end

    DEV[Developer] -->|Plan| BOARDS
    DEV -->|Code| REPOS
    REPOS -->|Trigger| PIPES
    PIPES -->|Store packages| ARTIFACTS
    PIPES -->|Run tests| TESTS
```

## Azure Pipeline YAML Structure

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main, develop]
  paths:
    include: [src/*, tests/*]
    exclude: [docs/*]

pr:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-latest'    # Microsoft-hosted agent
  # OR
  # name: 'self-hosted-pool'  # Self-hosted agent

variables:
  - group: prod-secrets       # Variable group (linked to Key Vault)
  - name: buildConfiguration
    value: 'Release'

stages:
- stage: Build
  displayName: 'Build & Test'
  jobs:
  - job: BuildJob
    steps:
    - checkout: self
      fetchDepth: 0

    - task: UseDotNet@2
      inputs:
        version: '8.x'

    - script: |
        dotnet build --configuration $(buildConfiguration)
        dotnet test --logger trx
      displayName: 'Build and Test'

    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/*.trx'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'

- stage: Deploy
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployProd
    environment: 'production'    # Has approval gates
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'prod-connection'
              appName: 'myapp-prod'
```

## Pipeline Triggers

```mermaid
graph LR
    subgraph Triggers
        CI[CI Trigger<br/>Push to branch]
        PR[PR Trigger<br/>Pull request]
        SCHED[Scheduled<br/>Cron expression]
        PIPE[Pipeline Trigger<br/>Another pipeline completes]
        MAN[Manual<br/>Run button]
    end
    Triggers --> Pipeline
```

## Variables Scope

```mermaid
graph TD
    PIPE_VAR[Pipeline Variables<br/>Defined at top level] --> STAGE_VAR[Stage Variables]
    STAGE_VAR --> JOB_VAR[Job Variables]
    JOB_VAR --> STEP_VAR[Step Variables]

    GROUP[Variable Groups<br/>Shared across pipelines] --> PIPE_VAR
    KV[Azure Key Vault<br/>Secrets] --> GROUP

    PREDEF[Predefined Variables<br/>Build.SourceBranch<br/>System.PullRequest.PullRequestId<br/>Agent.OS] --> STEP_VAR
```

## Templates (DRY)

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
  displayName: 'Build ${{ parameters.project }}'
- script: dotnet test ${{ parameters.project }}
  displayName: 'Test ${{ parameters.project }}'
```

```yaml
# azure-pipelines.yml - using template
stages:
- stage: Build
  jobs:
  - job: BuildAll
    steps:
    - template: templates/build-template.yml
      parameters:
        project: src/MyApp.sln
```

## Environments & Approvals

```mermaid
graph LR
    BUILD[Build Stage] --> DEV[Dev Environment<br/>Auto-deploy]
    DEV --> QA[QA Environment<br/>Auto-deploy]
    QA --> PROD[Production<br/>Manual Approval Required]

    style PROD fill:#FF5722,color:#fff
```

## Self-Hosted vs Microsoft-Hosted Agents

| Aspect | Microsoft-Hosted | Self-Hosted |
|---|---|---|
| Setup | Zero config | Install agent yourself |
| Cost | Free minutes (limited) | Your infrastructure |
| Customization | Limited | Full control |
| Clean state | Fresh VM each run | Persists between runs |
| Network | Public internet | Inside your network |
| Speed | Slower (VM spin-up) | Faster (pre-warmed) |
| Use case | Standard builds | Custom tools, private networks |
