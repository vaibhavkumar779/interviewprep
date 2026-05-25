# Jenkins - ANSWERS

---

## Basics & Architecture

**1.** Jenkins is an open-source automation server primarily used for CI/CD. Popular because: large plugin ecosystem (1800+), pipeline-as-code, extensible, active community, free.

**2.** Architecture: Controller (manages UI, scheduling, plugins, credentials), Agents (execute builds), Executors (threads on agents that run jobs). Controller dispatches jobs to agents based on labels.

**3.** Job = single runnable task (freestyle or pipeline). Pipeline = multi-stage workflow defined in code.

**4.** Freestyle project: UI-configured, single build action. Use when: simple tasks, one-off jobs, teams unfamiliar with Groovy.

**5.** `$JENKINS_HOME` (usually `/var/lib/jenkins`). Contains: `config.xml`, `jobs/`, `plugins/`, `credentials.xml`, `nodes/`, `secrets/`, `workspace/`.

**6.** Triggers: SCM polling (`H/5 * * * *`), webhook (GitHub/Bitbucket), cron schedule, upstream job, manual, API trigger.

**7.** Workspace: directory on the agent where source code is checked out and builds execute. Path: `$JENKINS_HOME/workspace/<job-name>/`.

**8.** Controller = orchestrator (scheduling, UI, API). Agent = worker (runs actual builds). Controller should NOT run builds for security and performance.

**9.** Executor = a thread slot on an agent. 2 executors = 2 concurrent builds on that agent.

**10.** Install: Java 11+, download jenkins.war or use package manager, run `java -jar jenkins.war`, access on port 8080, unlock with initial admin password.

**11-16: Plugins** — Name 10: Pipeline, Git, Blue Ocean, Docker Pipeline, Kubernetes, Credentials, JUnit, SonarQube, Slack Notification, LDAP/AD, Gerrit Trigger.

**17-22: Agents** — Docker agent: `agent { docker { image 'python:3.11' } }` — runs build inside container. K8s agent: uses Kubernetes plugin, spins up pod per build with specified containers.

---

## Pipelines

**1. Declarative skeleton:**
```groovy
pipeline {
    agent any
    stages {
        stage('Name') { steps { echo 'hello' } }
    }
}
```

**2.** `agent any` = run on any available agent. Options: `agent none` (each stage picks), `agent { label 'linux' }`, `agent { docker { image '...' } }`, `agent { kubernetes { yaml '...' } }`.

**3.** `stages` = container for ordered stages. Cannot nest stages directly, but can use `parallel` within a stage.

**4.** `steps` = container for individual commands: `sh`, `bat`, `echo`, `script { }`, `checkout`, `withCredentials`.

**5.** `post` conditions: `always` (always runs), `success`, `failure`, `unstable`, `changed` (status changed from last build), `aborted`, `cleanup`.

**6.** Environment variables: `environment { KEY = 'value'; SECRET = credentials('id') }`

**7.** `when` examples:
```groovy
when { branch 'main' }
when { expression { return params.DEPLOY } }
when { changeset '**/*.java' }
when { environment name: 'ENV', value: 'prod' }
when { allOf { branch 'main'; environment name: 'DEPLOY', value: 'true' } }
```

**8.** Parallel:
```groovy
stage('Tests') {
    parallel {
        stage('Unit') { steps { sh 'test' } }
        stage('Lint') { steps { sh 'lint' } }
    }
}
```

**9.** Parameters:
```groovy
parameters {
    string(name: 'VERSION', defaultValue: '1.0')
    choice(name: 'ENV', choices: ['dev','staging','prod'])
    booleanParam(name: 'FORCE_DEPLOY', defaultValue: false)
}
// Access: params.VERSION, params.ENV
```

**10.** Input (approval gate):
```groovy
stage('Deploy Prod') {
    input { message "Deploy to production?"; ok "Deploy"; submitter "admin" }
    steps { sh 'deploy.sh' }
}
```

**16-20: Scripted Pipeline**
```groovy
node('linux') {
    stage('Build') { sh 'make build' }
    stage('Test')  { sh 'make test' }
}
```
Error handling = `try/catch/finally`. Choose Scripted when: complex conditional logic, dynamic stage generation, calling external APIs mid-pipeline. Mix with Declarative using `script { }` block.

**21-25: Credentials**
```groovy
withCredentials([
    usernamePassword(credentialsId: 'docker-creds',
                     usernameVariable: 'USER',
                     passwordVariable: 'PASS')
]) {
    sh 'docker login -u $USER -p $PASS'
}
```
Never use `echo` with credentials. Jenkins auto-masks but `set -x` in shell can leak.

---

## Shared Libraries & Groovy

**1-5:** Shared libraries = reusable Groovy code in a Git repo loaded into any pipeline. Structure: `vars/` (global functions), `src/` (OOP classes), `resources/` (config files).

**6.** Import: `@Library('my-lib') _` (underscore needed because annotation requires a symbol)

**7.** Version: `@Library('my-lib@v2.0')`, `@Library('my-lib@main')`, `@Library('my-lib@abc123')`

**11-18: Groovy for Jenkins**
```groovy
// Variables
def name = 'Vaibhav'
String typed = 'hello'

// Function
def greet(name) { return "Hello ${name}" }

// Closure
def items = [1, 2, 3]
items.each { println it }

// Maps
def config = [name: 'app', version: '1.0']
config.each { k, v -> println "${k}: ${v}" }

// String: single = literal, double = interpolation
def x = 'no interpolation'
def y = "with ${name} interpolation"
```

**19.** CPS limitation: Jenkins serializes pipeline state at each step (for durability). Some Groovy constructs can't be serialized (closures, iterators). Use `@NonCPS` for methods that don't need serialization.

**21-30: Admin/Troubleshooting**
- Jenkins slow: Check executor count, pending queue, plugin performance, disk I/O, memory/GC, large console logs.
- Backup: `$JENKINS_HOME` (especially jobs/, config.xml, credentials.xml). Use ThinBackup plugin.
- JCasC: Configure Jenkins via YAML files — reproducible, version controlled.
- RBAC: Use Role Strategy plugin or Matrix Authorization. Define global roles + project roles.
- Pipeline worked yesterday, fails today: Plugin update, agent change, dependency update, credential expiry, infrastructure change, Docker image tag `latest` changed.
