> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Deep-dive learning guide (this file) |
| [basics_architecture.md](basics_architecture.md) | Basics & architecture questions |
| [pipelines.md](pipelines.md) | Pipeline questions |
| [shared_libraries_groovy_admin.md](shared_libraries_groovy_admin.md) | Shared libraries, Groovy & admin |
| [answers.md](answers.md) | All answers |

---

# Jenkins — Deep-Dive Learning Guide

---

## 1. What Is Jenkins?

Jenkins is an **open-source automation server** written in Java that orchestrates CI/CD pipelines. It's plugin-based (1800+ plugins) and can build, test, and deploy virtually anything.

```
┌─── Jenkins in the CI/CD landscape ─────────────────────────────┐
│                                                                 │
│  Code Commit ──► Jenkins ──► Build ──► Test ──► Deploy          │
│                    │                                            │
│                    ├── Freestyle jobs (UI-configured, legacy)   │
│                    ├── Pipeline (Jenkinsfile, code-as-config)   │
│                    └── Multibranch Pipeline (auto per branch)   │
│                                                                 │
│  Alternatives: GitHub Actions, Azure DevOps, GitLab CI,        │
│                CircleCI, Tekton, ArgoCD                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Jenkins Architecture

```
┌──────────────── Jenkins Controller (Master) ──────────────────┐
│                                                                │
│  ┌────────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Web UI         │  │  REST API    │  │  Job Scheduler   │  │
│  │  (dashboard,    │  │  (trigger,   │  │  (cron, SCM poll │  │
│  │   configure,    │  │   status,    │  │   webhook)       │  │
│  │   view logs)    │  │   artifacts) │  │                  │  │
│  └────────────────┘  └──────────────┘  └──────────────────┘  │
│                                                                │
│  ┌────────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Plugin Manager│  │  Credentials │  │  Build Queue     │  │
│  │  (install,     │  │  Store       │  │  (FIFO, labels,  │  │
│  │   update)      │  │  (encrypted) │  │   executors)     │  │
│  └────────────────┘  └──────────────┘  └──────────────────┘  │
│                                                                │
│  Should NOT run builds itself in production!                   │
└───────────┬────────────────────────────────┬──────────────────┘
            │                                │
     ┌──────▼──────┐                  ┌──────▼──────┐
     │   Agent 1   │                  │   Agent 2   │
     │  (Linux)    │                  │  (Windows)  │
     │             │                  │             │
     │  Executors: │                  │  Executors: │
     │  - Build 1  │                  │  - Build 3  │
     │  - Build 2  │                  │             │
     │             │                  │  Labels:    │
     │  Labels:    │                  │  windows,   │
     │  linux,     │                  │  dotnet     │
     │  docker     │                  │             │
     └─────────────┘                  └─────────────┘
```

### Key Concepts

| Concept | Description |
|---------|------------|
| **Controller (Master)** | Manages UI, scheduling, config. Should NOT run builds in prod |
| **Agent (Slave/Node)** | Machine that runs builds. Connected via SSH, JNLP, or cloud |
| **Executor** | A slot on an agent for running one build. 2 executors = 2 parallel builds |
| **Label** | Tag on agent (e.g., `linux`, `docker`). Jobs target agents by label |
| **Workspace** | Directory on agent where job runs. Each build gets its own workspace |
| **Fingerprint** | Hash to track which builds used which artifacts |

### Agent Types

```
┌─── Permanent Agents ─────────────────────────────────────┐
│  Always running, SSH or JNLP connected                    │
│  Good for: dedicated build machines, special hardware     │
└───────────────────────────────────────────────────────────┘

┌─── Cloud/Dynamic Agents ─────────────────────────────────┐
│  Spun up on demand, destroyed after build                 │
│  Docker agents:    spin up container per build            │
│  K8s agents:       spin up pod per build                  │
│  Cloud agents:     spin up VM per build (AWS, Azure)      │
│  Good for: auto-scaling, clean environments, cost savings │
└───────────────────────────────────────────────────────────┘
```

---

## 3. Jenkinsfile — Pipeline as Code

### Declarative Pipeline (recommended)

```groovy
pipeline {
    agent { label 'linux && docker' }    // Which agent

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        APP_VERSION = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'myregistry.azurecr.io'
        DOCKER_CREDS = credentials('docker-registry-creds')  // username:password
    }

    parameters {
        string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: 'Target env')
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_REGISTRY}/myapp:${APP_VERSION} .'
            }
        }

        stage('Test') {
            when { expression { params.RUN_TESTS } }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'pytest tests/unit/ --junitxml=unit-results.xml'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'pylint src/'
                    }
                }
            }
            post {
                always {
                    junit 'unit-results.xml'
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy image ${DOCKER_REGISTRY}/myapp:${APP_VERSION}'
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    echo ${DOCKER_CREDS_PSW} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_CREDS_USR} --password-stdin
                    docker push ${DOCKER_REGISTRY}/myapp:${APP_VERSION}
                '''
            }
        }

        stage('Deploy to Staging') {
            when { branch 'main' }
            steps {
                sh 'kubectl set image deployment/myapp myapp=${DOCKER_REGISTRY}/myapp:${APP_VERSION}'
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            input {
                message 'Deploy to production?'
                ok 'Yes, deploy!'
            }
            steps {
                sh 'kubectl --context prod set image deployment/myapp myapp=${DOCKER_REGISTRY}/myapp:${APP_VERSION}'
            }
        }
    }

    post {
        success {
            slackSend(channel: '#deploys', message: "✅ Build ${env.BUILD_NUMBER} succeeded")
        }
        failure {
            slackSend(channel: '#deploys', message: "❌ Build ${env.BUILD_NUMBER} failed")
        }
        always {
            cleanWs()    // Clean workspace
        }
    }
}
```

### Scripted Pipeline (older, more flexible)

```groovy
node('linux') {
    try {
        stage('Build') {
            checkout scm
            sh 'make build'
        }
        stage('Test') {
            sh 'make test'
        }
        stage('Deploy') {
            if (env.BRANCH_NAME == 'main') {
                sh 'make deploy'
            }
        }
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        cleanWs()
    }
}
```

### Declarative vs Scripted

| Aspect | Declarative | Scripted |
|--------|-------------|---------|
| Syntax | Structured, opinionated | Full Groovy, flexible |
| Learning curve | Easier | Harder |
| Error handling | `post { failure {} }` | try/catch/finally |
| Recommended? | Yes (Jenkins official) | Legacy, complex cases |

---

## 4. Shared Libraries

Reusable pipeline code shared across teams/repos:

```
jenkins-shared-lib/
├── vars/
│   ├── buildDocker.groovy      # Called as buildDocker() in pipeline
│   ├── deployToK8s.groovy
│   └── notifySlack.groovy
├── src/
│   └── org/mycompany/
│       └── Utils.groovy        # Helper classes
└── resources/
    └── templates/
        └── deploy.yaml         # Template files
```

```groovy
// vars/buildDocker.groovy
def call(Map config) {
    sh "docker build -t ${config.registry}/${config.image}:${config.tag} ."
    sh "docker push ${config.registry}/${config.image}:${config.tag}"
}

// Usage in Jenkinsfile:
@Library('my-shared-lib') _
pipeline {
    stages {
        stage('Build') {
            steps {
                buildDocker(registry: 'acr.io', image: 'myapp', tag: env.BUILD_NUMBER)
            }
        }
    }
}
```

---

## 5. Jenkins Security

```
┌─── Security Layers ────────────────────────────────────────────┐
│                                                                 │
│  Authentication:                                                │
│  - LDAP / Active Directory / SAML / OAuth                      │
│  - Jenkins internal user database (small teams)                │
│                                                                 │
│  Authorization:                                                 │
│  - Matrix-based security (user/group → permission matrix)      │
│  - Role-based strategy plugin (roles: admin, dev, viewer)      │
│  - Project-based (per-job permissions)                         │
│                                                                 │
│  Credentials Management:                                        │
│  - Encrypted credential store (username/password, SSH key,     │
│    secret text, certificate, token)                            │
│  - Scoped: global, folder, or job level                        │
│  - NEVER hardcode secrets in Jenkinsfile!                      │
│                                                                 │
│  Agent Security:                                                │
│  - Agents run in sandboxed environments                        │
│  - Script approval for untrusted Groovy                        │
│  - JNLP agents use encrypted channel                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Jenkins vs Other CI/CD Tools

| Feature | Jenkins | Azure DevOps | GitHub Actions | GitLab CI |
|---------|---------|-------------|----------------|-----------|
| Hosting | Self-hosted | Cloud + self-hosted | Cloud | Cloud + self-hosted |
| Config | Jenkinsfile (Groovy) | YAML | YAML | YAML |
| Plugins | 1800+ | Tasks/Extensions | Actions marketplace | Built-in |
| Scaling | Manual (agents) | Auto (MS-hosted) | Auto (runners) | Auto (runners) |
| Learning curve | Steep | Medium | Easy | Medium |
| Cost | Free (infra cost) | Free tier + paid | Free tier + paid | Free tier + paid |
| Best for | Complex enterprise | Microsoft/Azure shops | GitHub repos | GitLab repos |

---

## 7. Multibranch Pipeline

```
Repository with branches:
  main
  develop
  feature/login
  feature/payment
  hotfix/security-patch

Jenkins Multibranch Pipeline:
  ┌─────────────────────────────────────────────┐
  │  Scans repo for branches with Jenkinsfile   │
  │                                             │
  │  main ──────────► Pipeline (build+deploy)   │
  │  develop ───────► Pipeline (build+test)     │
  │  feature/login ─► Pipeline (build+test)     │
  │  feature/payment► Pipeline (build+test)     │
  │  hotfix/* ──────► Pipeline (build+deploy)   │
  │                                             │
  │  Auto-creates jobs for new branches         │
  │  Auto-deletes jobs for deleted branches     │
  └─────────────────────────────────────────────┘
```

---

## 8. Jenkins Best Practices

```
Pipeline:
  ✅ Pipeline as code (Jenkinsfile in repo, not UI)
  ✅ Declarative over Scripted
  ✅ Shared libraries for reusable logic
  ✅ Parallel stages where possible
  ✅ Fail fast (put quick tests first)
  ✅ Use input{} for manual approval gates

Controller:
  ✅ No builds on controller (agents only)
  ✅ Backup JENKINS_HOME regularly
  ✅ Keep plugins updated
  ✅ Use folders to organize jobs

Agents:
  ✅ Docker/K8s agents for clean, scalable builds
  ✅ Label agents by capability
  ✅ Use ephemeral agents (spin up, build, destroy)

Security:
  ✅ Credentials plugin (never hardcode secrets)
  ✅ RBAC (role-based access)
  ✅ Audit trail plugin
  ✅ Script Security plugin
```

---

## 9. Troubleshooting Jenkins

```
Build stuck in queue?
  → Check: agents online? executors available? label matches?
  → Jenkins → Manage → Nodes → check status

Build fails but works locally?
  → Different environment (PATH, tools, permissions)
  → Check agent OS, installed tools
  → Use Docker agent for reproducible builds

Out of disk space?
  → Build artifacts piling up
  → buildDiscarder(logRotator(numToKeepStr: '10'))
  → Clean old workspaces: cleanWs()
  → docker system prune on agents

Slow builds?
  → Parallel stages
  → Caching (npm cache, pip cache, Docker layer cache)
  → Faster agents (more CPU/RAM)
  → Incremental builds

Plugin conflicts?
  → Test updates in staging Jenkins first
  → Pin critical plugin versions
  → Check Jenkins compatibility matrix
```

---

## 10. Essential Jenkins Plugins

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Core declarative/scripted pipeline support |
| **Git** | Git SCM integration |
| **Docker Pipeline** | Build/run in Docker containers |
| **Kubernetes** | Dynamic K8s pod agents |
| **Credentials Binding** | Inject secrets into builds |
| **Blue Ocean** | Modern UI for pipeline visualization |
| **Job DSL** | Define jobs as Groovy code |
| **Configuration as Code (JCasC)** | YAML-based Jenkins config |
| **Gerrit Trigger** | Trigger builds on Gerrit events |
| **SonarQube Scanner** | Code quality analysis |
| **Warnings Next Gen** | Static analysis result aggregation |
| **Pipeline Utility Steps** | readJSON, readYAML, zip/unzip |
| **Email Extension** | Rich email notifications |
| **Matrix Authorization** | Fine-grained RBAC |
| **Timestamper** | Add timestamps to console output |

---

## 11. Docker & Kubernetes Agents

```groovy
// Docker agent — clean build environment per job
pipeline {
    agent {
        docker {
            image 'python:3.12-slim'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest'
            }
        }
    }
}
```

```groovy
// Kubernetes agent — dynamic pods on K8s cluster
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ['sleep', 'infinity']
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
'''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:${BUILD_NUMBER} .'
                }
            }
        }
    }
}
```

---

## 12. Jenkins Configuration as Code (JCasC)

```yaml
# jenkins.yaml — entire Jenkins config in one file
jenkins:
  systemMessage: "Jenkins configured via JCasC"
  numExecutors: 0             # No builds on controller
  securityRealm:
    ldap:
      configurations:
        - server: ldap.example.com
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions: ["Overall/Administer"]
          - name: "developer"
            permissions: ["Job/Build", "Job/Read"]

  clouds:
    - kubernetes:
        name: "k8s"
        serverUrl: "https://kubernetes.default"
        namespace: "jenkins"
        jenkinsUrl: "http://jenkins.jenkins.svc:8080"
        podTemplates:
          - name: "default"
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: "git-creds"
              username: "jenkins"
              password: "${GIT_PASSWORD}"

# Deploy: mount jenkins.yaml → set CASC_JENKINS_CONFIG env var
# All config in Git → PR review → merge → Jenkins auto-reloads
```

---

## 13. Gerrit Trigger Plugin (Ciena-relevant)

```groovy
// Triggered by Gerrit patchset-created event
pipeline {
    agent any
    triggers {
        gerrit(
            triggerOnEvents: [patchsetCreated()],
            gerritProjects: [[
                compareType: 'PLAIN',
                pattern: 'myproject',
                branches: [[ compareType: 'ANT', pattern: '**' ]]
            ]]
        )
    }
    stages {
        stage('Verify') {
            steps {
                // Gerrit env vars available:
                // GERRIT_CHANGE_NUMBER, GERRIT_PATCHSET_REVISION
                sh 'make test'
            }
            post {
                success {
                    // Send Verified +1 back to Gerrit
                    gerritReview labels: [Verified: 1]
                }
                failure {
                    gerritReview labels: [Verified: -1]
                }
            }
        }
    }
}

// Gerrit workflow: push → Gerrit → triggers Jenkins →
// Jenkins posts Verified ±1 → reviewer sees result → Code-Review +2 → submit
```

---

## 14. Matrix Builds

```groovy
// Test across multiple OS/language combinations
pipeline {
    agent none
    stages {
        stage('Test Matrix') {
            matrix {
                axes {
                    axis {
                        name 'OS'
                        values 'linux', 'windows'
                    }
                    axis {
                        name 'PYTHON'
                        values '3.10', '3.11', '3.12'
                    }
                }
                excludes {
                    exclude {
                        axis { name 'OS'; values 'windows' }
                        axis { name 'PYTHON'; values '3.10' }
                    }
                }
                stages {
                    stage('Test') {
                        agent { label "${OS}" }
                        steps {
                            sh "python${PYTHON} -m pytest"
                        }
                    }
                }
            }
            // Runs 5 combinations in parallel (6 minus 1 excluded)
        }
    }
}
```
