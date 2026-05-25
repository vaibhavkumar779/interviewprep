# Jenkins — COMPREHENSIVE ANSWERS

---

## Basics & Architecture

**1. What is Jenkins? Why popular?**

```
┌─── Jenkins ─────────────────────────────────────────────────────────┐
│                                                                      │
│  Open-source automation server for CI/CD                            │
│                                                                      │
│  Why popular:                                                       │
│  ✅ 1800+ plugins (integrates with everything)                      │
│  ✅ Pipeline-as-code (Jenkinsfile)                                  │
│  ✅ Free and open-source                                            │
│  ✅ Highly extensible and customizable                               │
│  ✅ Massive community and ecosystem                                  │
│  ✅ Self-hosted (full control)                                      │
│  ✅ Supports any language/platform                                   │
│                                                                      │
│  Used by: Ciena, Netflix, LinkedIn, many enterprises                │
└──────────────────────────────────────────────────────────────────────┘
```

---

**2. Jenkins architecture?**

```
┌─── Jenkins Controller (Master) ─────────────────────────────────────┐
│                                                                      │
│  ┌──────────┐  ┌──────────┐  ┌────────────┐  ┌──────────────────┐ │
│  │ Web UI   │  │ API      │  │ Scheduler  │  │ Plugin Manager   │ │
│  │ (port    │  │ (REST)   │  │ (assigns   │  │ (extends         │ │
│  │  8080)   │  │          │  │  jobs to   │  │  functionality)  │ │
│  │          │  │          │  │  agents)   │  │                  │ │
│  └──────────┘  └──────────┘  └────────────┘  └──────────────────┘ │
│                                                                      │
│  Credentials Store    Job Configs    Build History    SCM Polling   │
│                                                                      │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ Dispatches jobs
              ┌────────────┼────────────────┐
              ▼            ▼                ▼
      ┌───────────┐ ┌───────────┐  ┌───────────────┐
      │ Agent 1   │ │ Agent 2   │  │ Agent 3       │
      │ (Linux)   │ │ (Windows) │  │ (Docker/K8s)  │
      │           │ │           │  │               │
      │ Executor 1│ │ Executor 1│  │ Executor 1    │
      │ Executor 2│ │ Executor 2│  │ Executor 2    │
      │           │ │           │  │ (pods spin up │
      │ Runs build│ │ Runs .NET │  │  per build)   │
      │ jobs      │ │ builds    │  │               │
      └───────────┘ └───────────┘  └───────────────┘
```

Key components:
- **Controller**: Orchestrator — manages UI, scheduling, plugins, credentials
- **Agent**: Worker — executes actual build jobs
- **Executor**: Thread slot on an agent (2 executors = 2 concurrent builds)
- **Node**: Machine (controller or agent)

---

**3. Job vs Pipeline?**

```
Freestyle Job (legacy):              Pipeline (modern):
┌──────────────────────────┐        ┌──────────────────────────┐
│ Configured through UI     │        │ Defined in Jenkinsfile   │
│ Single build action       │        │ Multi-stage workflow     │
│ Limited flexibility       │        │ Version-controlled       │
│ No code review possible   │        │ Reviewable via PR        │
│ Hard to reproduce         │        │ Reproducible             │
│                            │        │ Supports parallel,       │
│ Use when: simple tasks,   │        │ conditions, approvals    │
│ one-off jobs               │        │                          │
│                            │        │ Use: ALWAYS for real CI/CD│
└──────────────────────────┘        └──────────────────────────┘
```

---

**4-5. `$JENKINS_HOME` directory structure:**

```
$JENKINS_HOME (/var/lib/jenkins)
├── config.xml              ← Global Jenkins configuration
├── credentials.xml         ← Encrypted credentials store
├── secrets/                ← Encryption keys
├── plugins/                ← Installed plugins (.jpi/.hpi)
├── jobs/                   ← All job configurations
│   ├── my-pipeline/
│   │   ├── config.xml      ← Job config
│   │   └── builds/         ← Build history
│   │       ├── 1/
│   │       ├── 2/
│   │       └── lastStableBuild → symlink
├── nodes/                  ← Agent configurations
├── workspace/              ← Build workspaces (source code)
├── logs/                   ← Jenkins logs
└── users/                  ← User configs
```

---

**6. Trigger types?**

```
┌─── Build Triggers ──────────────────────────────────────────────────┐
│                                                                      │
│  1. SCM Polling: Jenkins checks Git periodically                    │
│     H/5 * * * *   → every 5 min (H = hash for distribution)        │
│                                                                      │
│  2. Webhook: Git pushes event to Jenkins (preferred — instant)      │
│     GitHub → POST http://jenkins/github-webhook/                    │
│                                                                      │
│  3. Cron Schedule: Time-based                                       │
│     H 2 * * *     → nightly at ~2 AM                               │
│     H 0 * * 1-5   → weekday mornings                               │
│                                                                      │
│  4. Upstream Job: After another job completes                       │
│     build job done → trigger deploy job                             │
│                                                                      │
│  5. Manual: Click "Build Now" or API trigger                        │
│     curl -X POST http://jenkins/job/my-job/build                    │
│                                                                      │
│  6. Gerrit Trigger: On Gerrit patchset-created / change-merged      │
└──────────────────────────────────────────────────────────────────────┘
```

---

**7-8. Workspace? Controller vs Agent?**

```
Workspace: Directory on agent where source code is checked out
  Path: $JENKINS_HOME/workspace/<job-name>/
  Contains: cloned repo + build artifacts
  Cleaned: optionally at start of build (clean workspace)

Controller vs Agent — why NOT build on controller:
  ❌ Security: build scripts run with controller permissions
  ❌ Performance: builds consume controller CPU/memory
  ❌ Stability: bad build can crash controller → affects ALL jobs
  ❌ Scale: single machine can't handle many builds

  ✅ Controller: manage + schedule
  ✅ Agent: execute builds
```

---

**9-10. Executor? Jenkins installation?**

```
Executor = thread slot on a node
  Agent with 2 executors = can run 2 builds simultaneously

  Controller: set executors to 0 (don't run builds on master!)
  Agent: 2-4 executors (depends on CPU/RAM)

Installation:
  1. Java 11+ required
  2. Install: apt install jenkins / brew install jenkins / Docker
  3. docker run -p 8080:8080 jenkins/jenkins:lts
  4. Access http://localhost:8080
  5. Unlock with initial admin password:
     cat /var/lib/jenkins/secrets/initialAdminPassword
  6. Install suggested plugins
  7. Create admin user
```

---

**11. Essential Jenkins plugins (15+)?**

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Jenkinsfile support (core) |
| **Git** | Git SCM integration |
| **Blue Ocean** | Modern UI for pipelines |
| **Docker Pipeline** | Build inside Docker containers |
| **Kubernetes** | Spin up pods as agents |
| **Credentials** | Credential management |
| **Credentials Binding** | Use creds in pipeline steps |
| **JUnit** | Test result publishing |
| **SonarQube Scanner** | Code quality analysis |
| **Slack Notification** | Send alerts to Slack |
| **LDAP / Active Directory** | Authentication |
| **Role Strategy** | RBAC authorization |
| **Gerrit Trigger** | Gerrit code review integration |
| **Pipeline Utility Steps** | Read/write files in pipeline |
| **Timestamper** | Add timestamps to console output |
| **Job Configuration History** | Track config changes |

---

**12. Docker agent in pipeline:**

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.11-slim'
            args '-v /tmp:/tmp'     // Mount volumes
        }
    }
    stages {
        stage('Test') {
            steps {
                sh 'python --version'
                sh 'pip install -r requirements.txt'
                sh 'pytest tests/'
            }
        }
    }
}
```

```
How it works:
  1. Jenkins pulls python:3.11-slim image
  2. Starts container on the agent
  3. Mounts workspace into container
  4. Runs all steps INSIDE the container
  5. Destroys container when done

Benefits:
  ✅ Consistent build environment
  ✅ No tool installation on agents
  ✅ Different Python versions per job
  ✅ Isolated, reproducible builds
```

---

**13. Kubernetes agent (dynamic pods):**

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: golang
    image: golang:1.21
    command: ['sleep', '3600']
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
                container('golang') {
                    sh 'go build -o myapp'
                    sh 'go test ./...'
                }
            }
        }
        stage('Docker') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:${BUILD_NUMBER} .'
                }
            }
        }
    }
}
```

```
K8s agent workflow:
  1. Jenkins creates Pod in K8s cluster (per build)
  2. Pod has JNLP container (connects to Jenkins) + custom containers
  3. Pipeline steps run in specified container
  4. Pod destroyed after build completes

Benefits: Infinite scale, pay per use, isolated builds
```

---

## Declarative Pipeline — Deep Dive

**14. Full declarative pipeline skeleton:**

```groovy
pipeline {
    agent any                        // Where to run

    options {
        timeout(time: 30, unit: 'MINUTES')
        retry(2)
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'VERSION', defaultValue: '1.0', description: 'Release version')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target')
        booleanParam(name: 'FORCE_DEPLOY', defaultValue: false)
    }

    environment {
        APP_NAME = 'myapp'
        BUILD_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT?.take(7)}"
        DOCKER_CREDS = credentials('docker-hub-creds')    // Injects USR + PSW
    }

    triggers {
        pollSCM('H/5 * * * *')      // Every 5 min
        cron('H 2 * * 1-5')          // Weekday nights
    }

    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            parallel {               // Run in parallel
                stage('Unit') {
                    steps { sh 'make test-unit' }
                }
                stage('Lint') {
                    steps { sh 'make lint' }
                }
                stage('SAST') {
                    steps { sh 'make sast' }
                }
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
                expression { return params.ENV != 'prod' || params.FORCE_DEPLOY }
            }
            input {
                message "Deploy to ${params.ENV}?"
                ok "Deploy"
                submitter "admin,deployers"
            }
            steps {
                sh "deploy.sh ${params.ENV}"
            }
        }
    }

    post {
        always  { junit '**/test-results/*.xml' }
        success { slackSend channel: '#builds', message: "✅ ${APP_NAME} build passed" }
        failure { slackSend channel: '#builds', message: "❌ ${APP_NAME} build FAILED" }
        cleanup { cleanWs() }
    }
}
```

---

**15. `agent` options:**

```groovy
agent any                              // Any available agent
agent none                             // Each stage picks its own
agent { label 'linux' }                // Agent with label 'linux'
agent { label 'linux && docker' }      // Multiple labels
agent { docker { image 'node:20' } }   // Run in Docker container
agent { kubernetes { yaml '...' } }    // K8s pod per build
agent {
    node {
        label 'yocto-builder'
        customWorkspace '/opt/builds/${JOB_NAME}'
    }
}
```

---

**16. `post` conditions:**

```groovy
post {
    always  { /* runs always (cleanup) */ }
    success { /* only on success */ }
    failure { /* only on failure */ }
    unstable { /* test failures but build OK */ }
    changed { /* status changed from last build */ }
    aborted { /* build was cancelled */ }
    cleanup { /* very last step, even after always */ }
}
```

```
Execution order: always → success/failure/unstable → changed → cleanup
Use cases:
  always:  publish test results, archive artifacts
  success: send success notification, tag image
  failure: send alert, create incident
  cleanup: cleanWs() (remove workspace)
```

---

**17. `when` directive — conditional execution:**

```groovy
stage('Deploy Prod') {
    when {
        branch 'main'                                    // Only main branch
    }
}

stage('Deploy Feature') {
    when {
        expression { return params.DEPLOY == true }      // Parameter check
    }
}

stage('Build Changed') {
    when {
        changeset '**/*.java'                            // Only if Java files changed
    }
}

stage('Nightly Only') {
    when {
        triggeredBy 'TimerTrigger'                       // Only cron triggers
    }
}

stage('Complex') {
    when {
        allOf {                                          // AND
            branch 'main'
            environment name: 'DEPLOY', value: 'true'
        }
    }
}

stage('Any Match') {
    when {
        anyOf {                                          // OR
            branch 'main'
            branch 'develop'
        }
    }
}
```

---

**18. Parallel stages:**

```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            agent { label 'linux' }
            steps { sh 'pytest tests/unit/' }
        }
        stage('Integration Tests') {
            agent { label 'linux' }
            steps { sh 'pytest tests/integration/' }
        }
        stage('Lint') {
            agent { label 'linux' }
            steps { sh 'flake8 src/' }
        }
    }
    // All three run simultaneously on different agents
}
```

```
Without parallel:  Unit(5m) → Integration(10m) → Lint(2m) = 17 min
With parallel:     Unit(5m)                                  = 10 min
                   Integration(10m)   (runs simultaneously)
                   Lint(2m)

Savings: 41% faster
```

---

**19. Parameters:**

```groovy
parameters {
    string(name: 'VERSION', defaultValue: '1.0.0', description: 'App version')
    choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'])
    booleanParam(name: 'RUN_TESTS', defaultValue: true)
    text(name: 'RELEASE_NOTES', defaultValue: '', description: 'Release notes')
    password(name: 'DEPLOY_KEY', description: 'Deployment key')
}

// Access:
steps {
    echo "Deploying ${params.VERSION} to ${params.ENVIRONMENT}"
    script {
        if (params.RUN_TESTS) {
            sh 'make test'
        }
    }
}
```

---

**20. Input (approval gate):**

```groovy
stage('Deploy to Production') {
    input {
        message "Deploy to production?"
        ok "Yes, deploy!"
        submitter "admin,release-managers"       // Only these users can approve
        parameters {
            choice(name: 'CONFIRM', choices: ['yes', 'no'])
        }
    }
    steps {
        sh 'deploy-prod.sh'
    }
}
```

```
Pipeline pauses here:
  ┌──────────────────────────────────────┐
  │  Deploy to production?               │
  │                                      │
  │  [Yes, deploy!]    [Abort]           │
  │                                      │
  │  Waiting for: admin, release-managers│
  └──────────────────────────────────────┘
```

---

## Credentials Management

**21. Credential types:**

| Type | Use Case | Pipeline Access |
|------|----------|-----------------|
| Username + Password | Docker Hub, Git | `credentials('id')` |
| Secret text | API tokens | `credentials('id')` |
| Secret file | Kubeconfig, cert | `credentials('id')` |
| SSH key | Git SSH, server access | `credentials('id')` |
| Certificate | mTLS, code signing | `credentials('id')` |

---

**22. Use credentials in pipeline:**

```groovy
// Method 1: environment block (auto-splits USR + PSW)
environment {
    DOCKER_CREDS = credentials('docker-hub-creds')
    // Creates: DOCKER_CREDS_USR, DOCKER_CREDS_PSW
}
steps {
    sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
}

// Method 2: withCredentials block (more explicit)
steps {
    withCredentials([
        usernamePassword(
            credentialsId: 'docker-hub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )
    ]) {
        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
    }
    // Variables not available outside this block ✅
}

// Method 3: Secret text
withCredentials([string(credentialsId: 'api-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" https://api.example.com'
}

// Method 4: SSH key
withCredentials([sshUserPrivateKey(
    credentialsId: 'ssh-key',
    keyFileVariable: 'SSH_KEY',
    usernameVariable: 'SSH_USER'
)]) {
    sh 'ssh -i $SSH_KEY $SSH_USER@server.example.com "hostname"'
}
```

```
⚠️ Security rules:
  ❌ Never echo credentials: echo $DOCKER_PASS
  ❌ Never use set -x in shell (exposes in trace)
  ❌ Never pass as build parameter
  ✅ Jenkins auto-masks in console output
  ✅ Use withCredentials (scoped access)
  ✅ Store in Jenkins Credentials Store (encrypted)
  ✅ Or better: external vault (HashiCorp Vault plugin)
```

---

## Shared Libraries

**23. What are shared libraries?**

```
Reusable Groovy code in a Git repo, loaded into any pipeline.
Like "npm packages" for Jenkins pipelines.

Repository structure:
  jenkins-shared-lib/
  ├── vars/                    ← Global functions (most common)
  │   ├── buildApp.groovy      ← Call as: buildApp()
  │   ├── deployToK8s.groovy   ← Call as: deployToK8s()
  │   └── notifySlack.groovy   ← Call as: notifySlack()
  ├── src/                     ← OOP classes (advanced)
  │   └── com/company/
  │       └── Docker.groovy
  ├── resources/               ← Config files, templates
  │   └── config.yaml
  └── README.md
```

---

**24. Example shared library function:**

```groovy
// vars/buildAndPush.groovy
def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "docker build -t ${config.registry}/${config.image}:${config.tag} ."
                }
            }
            stage('Push') {
                steps {
                    withCredentials([usernamePassword(
                        credentialsId: config.credentialsId,
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh "docker login -u $USER -p $PASS ${config.registry}"
                        sh "docker push ${config.registry}/${config.image}:${config.tag}"
                    }
                }
            }
        }
    }
}
```

```groovy
// Jenkinsfile (consumer)
@Library('my-shared-lib') _

buildAndPush(
    registry: 'myacr.azurecr.io',
    image: 'myapp',
    tag: env.BUILD_NUMBER,
    credentialsId: 'acr-creds'
)
```

---

**25. Import and versioning:**

```groovy
@Library('my-shared-lib') _                    // Default branch
@Library('my-shared-lib@main') _               // Specific branch
@Library('my-shared-lib@v2.0') _               // Git tag
@Library('my-shared-lib@abc1234') _            // Specific commit

// Underscore _ is required: annotation needs a symbol to attach to

// Configure in: Manage Jenkins → System → Global Pipeline Libraries
// Source: Git repo URL + credentials
```

---

## Scripted Pipeline

**26. Scripted vs Declarative:**

```
Declarative:                        Scripted:
┌──────────────────────────┐       ┌──────────────────────────┐
│ pipeline {                │       │ node('linux') {           │
│   agent any               │       │   stage('Build') {       │
│   stages {                │       │     sh 'make build'       │
│     stage('Build') {     │       │   }                        │
│       steps {             │       │   try {                    │
│         sh 'make build'   │       │     stage('Test') {       │
│       }                    │       │       sh 'make test'       │
│     }                      │       │     }                      │
│   }                        │       │   } catch (e) {           │
│   post {                   │       │     slackSend "FAILED!"   │
│     failure { notify() } │       │     throw e                │
│   }                        │       │   } finally {             │
│ }                          │       │     cleanWs()              │
│                            │       │   }                        │
│ Structured, opinionated   │       │ }                          │
│ Easier to read             │       │                            │
│ Better for 90% of cases   │       │ Full Groovy power          │
│                            │       │ Dynamic stage generation   │
│                            │       │ Complex logic              │
└──────────────────────────┘       └──────────────────────────┘
```

Choose scripted when: dynamic stage generation, complex conditional logic, calling APIs mid-pipeline, matrix builds.

Mix both with `script { }` block inside Declarative.

---

**27. Error handling in Scripted:**

```groovy
node('linux') {
    try {
        stage('Build') {
            sh 'make build'
        }
        stage('Test') {
            sh 'make test'
        }
        stage('Deploy') {
            sh 'make deploy'
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        slackSend channel: '#alerts', message: "Build failed: ${e.message}"
        throw e    // Re-throw to mark build as failed
    } finally {
        // Always runs (cleanup)
        junit '**/test-results/*.xml'
        cleanWs()
    }
}
```

---

## Groovy Essentials for Jenkins

**28. Groovy syntax cheat sheet:**

```groovy
// Variables
def name = 'Vaibhav'              // Dynamic type
String typed = 'hello'             // Typed
def list = [1, 2, 3]              // List
def map = [name: 'app', ver: '1.0']  // Map

// String interpolation
def msg = "Hello ${name}"          // GString (double quotes)
def raw = 'No ${interpolation}'    // Literal (single quotes)

// Closures (like lambdas)
def items = ['a', 'b', 'c']
items.each { println it }         // 'it' is default parameter
items.each { item -> println item }

// Maps
def config = [env: 'prod', region: 'us']
config.each { key, val -> println "${key}: ${val}" }

// Conditional
def result = (x > 5) ? 'big' : 'small'

// Functions
def greet(name) {
    return "Hello ${name}"
}
```

---

**29. CPS (Continuation Passing Style) limitation:**

```
Jenkins serializes pipeline state at each step for durability.
If Jenkins restarts mid-build → pipeline resumes from last step.

Problem: Some Groovy constructs can't be serialized:
  - Closures with external references
  - Iterators
  - Non-serializable objects

Solution: @NonCPS annotation
  @NonCPS
  def parseJson(text) {
      // This method won't be serialized
      // Can use any Groovy construct freely
      return new JsonSlurper().parseText(text)
  }

⚠️ @NonCPS methods: no Jenkins steps (sh, echo) allowed inside!
```

---

## Admin & Troubleshooting

**30. Jenkins is slow — diagnosis:**

```
┌─── Jenkins Performance Troubleshooting ─────────────────────────────┐
│                                                                      │
│  1. Check executor count: Are builds queuing?                       │
│     → Add more agents or executors                                  │
│                                                                      │
│  2. Check disk I/O: Is $JENKINS_HOME on slow disk?                 │
│     → Move to SSD, cleanup old builds                               │
│                                                                      │
│  3. Check JVM heap: java.lang.OutOfMemoryError?                    │
│     → Increase: -Xmx4g -Xms4g                                     │
│                                                                      │
│  4. Check plugins: Too many? Outdated?                             │
│     → Remove unused, update all                                     │
│                                                                      │
│  5. Check build history: Thousands of old builds?                  │
│     → Set "Discard old builds" policy (keep last 30)               │
│                                                                      │
│  6. Check console logs: Huge logs per build?                       │
│     → Limit output, redirect to file                                │
│                                                                      │
│  7. Check concurrent builds: Too many on one agent?               │
│     → Reduce executors per agent, add more agents                  │
└──────────────────────────────────────────────────────────────────────┘
```

---

**31. Jenkins backup strategy:**

```
What to backup:
  $JENKINS_HOME/
  ├── config.xml              ← Global config (CRITICAL)
  ├── credentials.xml         ← Credentials (CRITICAL)
  ├── secrets/                ← Encryption keys (CRITICAL)
  ├── jobs/*/config.xml       ← Job definitions
  ├── nodes/                  ← Agent configs
  └── users/                  ← User configs

What NOT to backup:
  workspace/    ← Rebuild from source
  plugins/      ← Reinstall from list
  builds/       ← Optional (large, reproducible)

Backup methods:
  1. ThinBackup plugin (scheduled, incremental)
  2. Filesystem snapshot (LVM, ZFS, EBS)
  3. rsync to backup server
  4. Jenkins Configuration as Code (JCasC) → Git
```

---

**32. Jenkins Configuration as Code (JCasC):**

```yaml
# jenkins.yaml — entire Jenkins config in one file
jenkins:
  systemMessage: "Jenkins configured by JCasC"
  numExecutors: 0              # Don't run builds on controller
  securityRealm:
    ldap:
      configurations:
        - server: ldap.company.com
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: admin
            permissions: [Overall/Administer]
  nodes:
    - permanent:
        name: "linux-agent-01"
        launcher:
          ssh:
            host: "10.0.1.10"
            credentialsId: "agent-ssh"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: "docker-creds"
              username: "admin"
              password: "${DOCKER_PASSWORD}"    # From env var

unclassified:
  slackNotifier:
    teamDomain: "mycompany"
    tokenCredentialId: "slack-token"
```

Benefits: reproducible Jenkins, disaster recovery, review via PR, no click-ops.

---

**33. RBAC (Role-Based Access Control):**

```
Using Role Strategy Plugin:

Global Roles:
┌──────────────┬───────────────────────────────┐
│ Role         │ Permissions                    │
├──────────────┼───────────────────────────────┤
│ admin        │ Everything                     │
│ developer    │ View, Build, Read              │
│ viewer       │ Read only                      │
└──────────────┴───────────────────────────────┘

Project Roles (pattern-based):
┌──────────────┬────────────────┬───────────────┐
│ Role         │ Pattern        │ Permissions    │
├──────────────┼────────────────┼───────────────┤
│ team-alpha   │ alpha-.*       │ Build, Config  │
│ team-beta    │ beta-.*        │ Build, Config  │
│ release-mgr  │ .*-release-.*  │ Build, Deploy  │
└──────────────┴────────────────┴───────────────┘

Best practices:
  - Least privilege
  - Team-based roles
  - No anonymous access
  - Audit access regularly
```

---

**34. Pipeline worked yesterday, fails today — diagnosis:**

```
┌─── Troubleshooting Checklist ───────────────────────────────────────┐
│                                                                      │
│  1. Plugin update?      → Check "Manage Plugins" → Recent updates  │
│  2. Agent changed?      → Agent offline, disk full, Docker update   │
│  3. Dependency update?  → requirements.txt, package.json changed   │
│  4. Credential expired? → Token/password rotated externally        │
│  5. Base image changed? → python:3.11-slim tag updated overnight   │
│  6. External service?   → Registry, API, DNS issue                 │
│  7. Disk space?         → df -h on controller and agents           │
│  8. Git branch?         → Branch deleted or force-pushed            │
│                                                                      │
│  Debugging:                                                         │
│  • Check console output (full log)                                  │
│  • Compare with last successful build diff                          │
│  • Replay pipeline (Blue Ocean) to test changes                    │
│  • Run stage individually                                           │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Jenkins in Ciena Context

**35. Jenkins + Gerrit + Yocto workflow:**

```
┌─── Developer ────────────────────────────────────────────────────────┐
│                                                                      │
│  1. git commit --amend       (update same Change-Id)                │
│  2. git push origin HEAD:refs/for/main                              │
│     └─► Gerrit creates/updates code review                          │
│                                                                      │
│  3. Gerrit Trigger plugin fires Jenkins build                       │
│     └─► "patchset-created" or "change-merged" event                │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─── Jenkins Pipeline ────────────────────────────────────────────────┐
│                                                                      │
│  Stage: Repo Sync                                                   │
│    repo init -u manifest.git -b main                                │
│    repo sync -j8                                                     │
│                                                                      │
│  Stage: Build                                                       │
│    source oe-init-build-env build                                   │
│    bitbake core-image-custom   (uses sstate cache → fast rebuild)   │
│                                                                      │
│  Stage: Test                                                        │
│    runqemu qemuarm64 nographic &                                    │
│    run-tests.sh                                                      │
│                                                                      │
│  Stage: Archive                                                     │
│    archiveArtifacts 'build/tmp/deploy/images/**'                    │
│                                                                      │
│  Stage: Report                                                      │
│    Post build result back to Gerrit: +1 Verified / -1 Verified     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

**36. Jenkins Multibranch Pipeline:**

```
Scans Git repo for Jenkinsfiles in all branches:

  main        → Pipeline from Jenkinsfile in main
  develop     → Pipeline from Jenkinsfile in develop
  feature/xyz → Pipeline from Jenkinsfile in feature/xyz
  PR #42      → Pipeline from Jenkinsfile in PR

  Auto-creates pipeline jobs per branch
  Auto-deletes when branch is deleted

Configuration: New Item → Multibranch Pipeline → Git source
```

---

**37. Blue Ocean:**

Modern UI for Jenkins pipelines:
- Visual pipeline editor
- Clear stage visualization (pass/fail per stage)
- Better GitHub/Bitbucket integration
- Pipeline run history
- Cleaner log viewing

Access: `/blue/` URL suffix

---

**38. Jenkins pipeline for Docker + K8s deployment:**

```groovy
pipeline {
    agent { label 'linux' }

    environment {
        REGISTRY = 'myacr.azurecr.io'
        IMAGE    = 'myapp'
        TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build & Test') {
            agent { docker { image 'golang:1.21' } }
            steps {
                sh 'go build -o myapp'
                sh 'go test ./... -v'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'acr-creds',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh "docker build -t ${REGISTRY}/${IMAGE}:${TAG} ."
                    sh "docker login -u $ACR_USER -p $ACR_PASS ${REGISTRY}"
                    sh "docker push ${REGISTRY}/${IMAGE}:${TAG}"
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl set image deployment/myapp \
                            myapp=${REGISTRY}/${IMAGE}:${TAG} \
                            --namespace=production
                        kubectl rollout status deployment/myapp -n production --timeout=300s
                    """
                }
            }
        }
    }

    post {
        success { slackSend channel: '#deploys', message: "✅ ${IMAGE}:${TAG} deployed" }
        failure { slackSend channel: '#deploys', message: "❌ ${IMAGE}:${TAG} FAILED" }
    }
}
```

---

**39. Jenkins distributed builds (master-agent architecture):**

```
                    Jenkins Controller
                          │
           ┌──────────────┼──────────────┐
           ▼              ▼              ▼
     ┌──────────┐  ┌──────────┐  ┌──────────────┐
     │ Agent:    │  │ Agent:    │  │ Agent:        │
     │ yocto-01  │  │ yocto-02  │  │ docker-01    │
     │           │  │           │  │               │
     │ Label:    │  │ Label:    │  │ Label:        │
     │ yocto     │  │ yocto     │  │ docker        │
     │           │  │           │  │               │
     │ 32 cores  │  │ 32 cores  │  │ 8 cores       │
     │ 64GB RAM  │  │ 64GB RAM  │  │ 16GB RAM      │
     │ NVMe SSD  │  │ NVMe SSD  │  │               │
     │           │  │           │  │ Runs: Docker  │
     │ Runs:     │  │ Runs:     │  │ builds, tests │
     │ Yocto     │  │ Yocto     │  │               │
     │ bitbake   │  │ bitbake   │  │               │
     └──────────┘  └──────────┘  └──────────────┘

  Agent connection methods:
    - SSH (controller connects to agent)
    - JNLP (agent connects to controller, good for K8s)
    - Docker (ephemeral containers)
```

---

**40. Jenkins vs Azure DevOps comparison:**

```
┌─── Jenkins ──────────────────────┬─── Azure DevOps ──────────────────┐
│                                   │                                    │
│ Self-hosted (you manage)          │ SaaS (Microsoft manages)          │
│ Jenkinsfile (Groovy)              │ YAML pipelines                    │
│ 1800+ plugins                     │ Built-in tasks + marketplace      │
│ Shared Libraries (Groovy)         │ Templates (YAML)                  │
│ Full customization                │ Opinionated, simpler              │
│ Credentials plugin                │ Variable groups + Key Vault       │
│ Gerrit Trigger plugin             │ Azure Repos / GitHub              │
│ Node/label-based agents           │ Pool-based agents                 │
│ Blue Ocean UI                     │ Modern UI built-in                │
│                                   │                                    │
│ Better for:                       │ Better for:                       │
│ - Embedded/Yocto (ecosystem)     │ - Azure cloud deployments         │
│ - Complex custom workflows        │ - Quick start, less maintenance   │
│ - Gerrit integration              │ - Enterprise governance           │
│ - Maximum flexibility             │ - Integrated boards/artifacts    │
└───────────────────────────────────┴────────────────────────────────────┘
```
