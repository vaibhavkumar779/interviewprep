# DevSecOps — Deep-Dive Learning Guide

---

## 1. What Is DevSecOps?

```
Traditional:   Dev ──► Ops ──► Security (at the end, too late!)
DevSecOps:     Security ──► embedded at EVERY stage

┌─── Shift-Left Security ────────────────────────────────────┐
│                                                             │
│  Find vulnerabilities EARLY when they're cheap to fix      │
│                                                             │
│  Cost to fix bug:                                          │
│    Design phase:   $1                                       │
│    Development:    $10                                      │
│    Testing:        $100                                     │
│    Production:     $10,000+                                │
│                                                             │
│  "Shift left" = move security checks earlier in pipeline   │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Security at Every Pipeline Stage

```
┌─── Code ──────────────────────────────────────────────────┐
│  Pre-commit hooks:                                        │
│    - Secret detection (git-secrets, detect-secrets)       │
│    - Linting (security rules in ESLint, pylint)          │
│    - Commit signing (GPG)                                │
└───────────────────────────────────────────────────────────┘
         │
┌────────▼── Build ─────────────────────────────────────────┐
│  Dependency scanning:                                     │
│    - npm audit, pip-audit, OWASP Dependency-Check         │
│    - Snyk, Dependabot (auto-fix PRs)                     │
│    - License compliance (FOSSA)                           │
│                                                           │
│  SAST (Static Application Security Testing):              │
│    - SonarQube, Semgrep, Checkmarx, Fortify              │
│    - Analyzes source code WITHOUT running it             │
│    - Finds: SQL injection, XSS, hardcoded secrets        │
└───────────────────────────────────────────────────────────┘
         │
┌────────▼── Test ──────────────────────────────────────────┐
│  Container scanning:                                      │
│    - Trivy, Grype, Snyk Container                        │
│    - Scans image layers for CVEs                         │
│    - Dockerfile linting (hadolint)                       │
│                                                           │
│  IaC scanning:                                            │
│    - Checkov, tfsec, Terrascan                           │
│    - Finds: open security groups, unencrypted storage    │
└───────────────────────────────────────────────────────────┘
         │
┌────────▼── Deploy ────────────────────────────────────────┐
│  DAST (Dynamic Application Security Testing):             │
│    - OWASP ZAP, Burp Suite                               │
│    - Tests RUNNING application (like an attacker)        │
│    - Finds: injection, auth issues, misconfig            │
│                                                           │
│  Admission controllers (K8s):                             │
│    - OPA Gatekeeper, Kyverno                             │
│    - Block: privileged containers, no resource limits    │
└───────────────────────────────────────────────────────────┘
         │
┌────────▼── Runtime ───────────────────────────────────────┐
│  Runtime protection:                                      │
│    - Falco (K8s threat detection)                        │
│    - WAF (Web Application Firewall)                      │
│    - RASP (Runtime Application Self-Protection)          │
│    - Network policies, pod security standards            │
│                                                           │
│  Monitoring:                                              │
│    - Audit logs, access logs                             │
│    - Anomaly detection                                   │
│    - Incident response automation                        │
└───────────────────────────────────────────────────────────┘
```

---

## 3. SAST vs DAST vs SCA vs IAST

| Type | What | When | How | Tools |
|------|------|------|-----|-------|
| **SAST** | Static code analysis | Build time | Scans source code | SonarQube, Semgrep |
| **DAST** | Dynamic testing | Post-deploy | Tests running app | OWASP ZAP, Burp |
| **SCA** | Dependency scanning | Build time | Checks packages for CVEs | Snyk, npm audit |
| **IAST** | Instrumented testing | Test time | Agent inside running app | Contrast, Seeker |

```
SAST:  Looks at CODE (finds hardcoded password in source)
DAST:  Looks at APP (finds SQL injection by sending payloads)
SCA:   Looks at DEPS (finds log4j CVE in your dependencies)
IAST:  Agent IN app (finds vulnerabilities during test execution)
```

---

## 4. OWASP Top 10 (2021)

```
┌─── OWASP Top 10 Web Application Security Risks ───────────┐
│                                                             │
│  1. Broken Access Control                                   │
│     Users can act outside their intended permissions        │
│     Fix: deny by default, validate server-side             │
│                                                             │
│  2. Cryptographic Failures                                  │
│     Weak encryption, plaintext secrets, HTTP not HTTPS     │
│     Fix: TLS everywhere, encrypt at rest, strong algorithms│
│                                                             │
│  3. Injection (SQL, OS, LDAP, XSS)                         │
│     Untrusted data sent to interpreter                     │
│     Fix: parameterized queries, input validation, encoding │
│                                                             │
│  4. Insecure Design                                         │
│     Architecture-level flaws, missing threat modeling      │
│     Fix: threat modeling, secure design patterns           │
│                                                             │
│  5. Security Misconfiguration                               │
│     Default passwords, verbose errors, unnecessary features│
│     Fix: hardened configs, automated config management     │
│                                                             │
│  6. Vulnerable & Outdated Components                        │
│     Using libraries with known CVEs                        │
│     Fix: SCA scanning, auto-update deps, Dependabot        │
│                                                             │
│  7. Authentication Failures                                 │
│     Weak passwords, missing MFA, session issues            │
│     Fix: MFA, strong passwords, session management         │
│                                                             │
│  8. Software & Data Integrity Failures                      │
│     Untrusted CI/CD plugins, unsigned packages             │
│     Fix: verify signatures, secure CI/CD pipeline          │
│                                                             │
│  9. Security Logging & Monitoring Failures                  │
│     Not detecting attacks, no audit trail                   │
│     Fix: centralized logging, alerting, incident response  │
│                                                             │
│  10. Server-Side Request Forgery (SSRF)                     │
│      App fetches remote resource without validating URL    │
│      Fix: whitelist URLs, disable redirects, network seg   │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Secrets Management

```
❌ BAD:
  - Hardcoded in source code
  - In environment variables (visible in process list)
  - In Dockerfiles or docker-compose
  - In git history (even if removed from HEAD!)
  - In CI/CD pipeline logs

✅ GOOD:
  ┌─── Secrets Management Tools ─────────────────────────────┐
  │                                                           │
  │  HashiCorp Vault:                                        │
  │    - Dynamic secrets (short-lived, auto-rotated)         │
  │    - Multiple auth methods (K8s, LDAP, tokens)           │
  │    - Encryption as a service                             │
  │                                                           │
  │  Azure Key Vault:                                         │
  │    - Keys, secrets, certificates                          │
  │    - Managed Identity access (no credentials!)           │
  │    - Audit logging                                        │
  │                                                           │
  │  AWS Secrets Manager / Parameter Store                    │
  │  K8s Secrets + Sealed Secrets / External Secrets Operator│
  │  SOPS (encrypted files in git)                           │
  └───────────────────────────────────────────────────────────┘
```

### Secret Scanning

```bash
# Pre-commit: detect-secrets
pip install detect-secrets
detect-secrets scan > .secrets.baseline
detect-secrets audit .secrets.baseline

# In CI: git-secrets (AWS patterns)
git secrets --install
git secrets --register-aws
git secrets --scan

# Trivy secret scanning
trivy fs --scanners secret .
```

---

## 6. Container Security

```
┌─── Container Security Layers ──────────────────────────────┐
│                                                             │
│  Image Build:                                               │
│    ✅ Minimal base (distroless, alpine, slim)              │
│    ✅ No secrets in image (use runtime injection)          │
│    ✅ Specific tags (not :latest)                          │
│    ✅ Multi-stage builds (no build tools in prod)          │
│    ✅ hadolint for Dockerfile linting                       │
│                                                             │
│  Image Scanning:                                            │
│    ✅ Trivy / Grype in CI pipeline                         │
│    ✅ Block deployment if HIGH/CRITICAL CVEs               │
│    ✅ Scan base images regularly (new CVEs daily!)         │
│                                                             │
│  Runtime:                                                   │
│    ✅ Non-root (USER 1001 in Dockerfile)                   │
│    ✅ Read-only filesystem (--read-only)                   │
│    ✅ Drop capabilities (--cap-drop ALL)                   │
│    ✅ No privileged mode                                   │
│    ✅ Resource limits (CPU, memory)                        │
│    ✅ Network policies (restrict pod-to-pod)               │
│    ✅ Pod Security Standards / Admission controllers       │
│                                                             │
│  Registry:                                                  │
│    ✅ Private registry (ACR, ECR)                          │
│    ✅ Image signing (cosign, Notary)                       │
│    ✅ Content trust (DOCKER_CONTENT_TRUST=1)               │
│    ✅ Vulnerability scanning on push                       │
└─────────────────────────────────────────────────────────────┘
```

```bash
# Scan image with Trivy
trivy image myapp:v1

# Scan for HIGH and CRITICAL only
trivy image --severity HIGH,CRITICAL myapp:v1

# Fail CI if vulnerabilities found
trivy image --exit-code 1 --severity CRITICAL myapp:v1

# Scan filesystem (source code)
trivy fs --scanners vuln,secret,config .

# Scan IaC
trivy config ./terraform/
```

---

## 7. Kubernetes Security

```
┌─── K8s Security Layers ───────────────────────────────────┐
│                                                            │
│  Authentication:                                           │
│    - Service accounts, OIDC, certificates                 │
│    - No default service account tokens in pods            │
│                                                            │
│  Authorization (RBAC):                                     │
│    - Least privilege (minimal verbs + resources)          │
│    - No cluster-admin for apps                            │
│    - Namespace isolation                                   │
│                                                            │
│  Admission Control:                                        │
│    - OPA Gatekeeper / Kyverno policies                    │
│    - Pod Security Standards (restricted, baseline)        │
│    - Block: privileged, hostPath, hostNetwork             │
│                                                            │
│  Network:                                                  │
│    - NetworkPolicies (default deny all, allow specific)   │
│    - Service mesh (mTLS between services)                 │
│    - Ingress TLS termination                              │
│                                                            │
│  Runtime:                                                  │
│    - Falco (detects unexpected behavior)                  │
│    - Audit logs enabled                                    │
│    - Seccomp/AppArmor profiles                            │
└────────────────────────────────────────────────────────────┘
```

---

## 8. CI/CD Pipeline Security

```
Supply Chain Attacks:
  SolarWinds, CodeCov, ua-parser-js — attackers compromise
  build pipelines or dependencies to inject malware

Protections:
  ✅ Pin dependency versions (lockfiles: package-lock.json)
  ✅ Verify checksums / signatures
  ✅ Use private artifact feeds (cache upstream packages)
  ✅ Least privilege for CI service accounts
  ✅ Separate build and deploy credentials
  ✅ Immutable build artifacts (content-addressable)
  ✅ SLSA framework (Supply-chain Levels for Software Artifacts)
  ✅ Signed commits and images

  SBOM (Software Bill of Materials):
    List of ALL components in your software
    Tools: syft, Trivy, CycloneDX
    Required by US Executive Order for govt software
```

---

## 9. Security Pipeline Example

```yaml
# Azure DevOps pipeline with security gates
stages:
  - stage: SecurityScan
    jobs:
      - job: SAST
        steps:
          - script: sonar-scanner -Dsonar.projectKey=myapp
            displayName: 'SonarQube SAST Scan'

      - job: DependencyScan
        steps:
          - script: npm audit --audit-level=high
            displayName: 'npm Audit'
          - script: pip-audit
            displayName: 'pip Audit'

      - job: ContainerScan
        steps:
          - script: trivy image --exit-code 1 --severity CRITICAL myapp:$BUILD_ID
            displayName: 'Trivy Container Scan'

      - job: IaCScan
        steps:
          - script: checkov -d terraform/ --hard-fail-on HIGH
            displayName: 'Checkov IaC Scan'

      - job: SecretScan
        steps:
          - script: trivy fs --scanners secret --exit-code 1 .
            displayName: 'Secret Detection'

  - stage: DAST
    dependsOn: Deploy_Staging
    jobs:
      - job: ZAPScan
        steps:
          - script: |
              docker run --rm owasp/zap2docker-stable zap-baseline.py \
                -t https://staging.example.com \
                -r zap-report.html
            displayName: 'OWASP ZAP DAST Scan'
```

---

## 10. Zero Trust Security

```
Traditional:  Trust inside network perimeter (castle & moat)
Zero Trust:   Never trust, always verify (every request)

Principles:
  1. Verify explicitly (authenticate + authorize every request)
  2. Least privilege access (minimal permissions, JIT access)
  3. Assume breach (segment network, encrypt everything, monitor)

Implementation in DevOps:
  - mTLS between all services (service mesh: Istio, Linkerd)
  - Short-lived credentials (no permanent passwords)
  - MFA for human access
  - Network segmentation (VNets, security groups, NetworkPolicies)
  - Continuous monitoring and anomaly detection
```

---

## 11. SonarQube Integration Examples

```groovy
// Jenkins Pipeline — SonarQube with Quality Gate
pipeline {
    agent any
    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=myapp \
                          -Dsonar.sources=src/ \
                          -Dsonar.tests=tests/ \
                          -Dsonar.language=python \
                          -Dsonar.python.coverage.reportPaths=coverage.xml
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                    // Pipeline FAILS if quality gate not passed
                }
            }
        }
    }
}
```

```yaml
# Azure DevOps — SonarQube tasks
steps:
  - task: SonarQubePrepare@5
    inputs:
      SonarQube: 'sonar-connection'
      scannerMode: 'CLI'
      configMode: 'manual'
      cliProjectKey: 'myapp'
      cliSources: 'src/'

  - script: dotnet build
    displayName: 'Build'

  - task: SonarQubeAnalyze@5
  - task: SonarQubePublish@5
    inputs:
      pollingTimeoutSec: '300'
```

**Quality Gates** — pass/fail criteria:
```
Condition                        Threshold
──────────────────────────────   ──────────
Coverage on new code             >= 80%
Duplicated lines on new code     <= 3%
Maintainability rating           A
Reliability rating               A
Security rating                  A
Security hotspots reviewed       100%

If ANY condition fails → Quality Gate = FAILED → Pipeline stops
```

---

## 12. CodeQL (GitHub SAST)

```yaml
# .github/workflows/codeql.yml
name: "CodeQL"
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'    # Weekly scan

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        language: ['python', 'javascript', 'go']
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended    # More rules than default
      - uses: github/codeql-action/autobuild@v3
      - uses: github/codeql-action/analyze@v3

# CodeQL finds: SQL injection, XSS, command injection,
# path traversal, insecure deserialization, crypto weaknesses
# Results appear directly in GitHub PR as review comments
```

---

## 13. CVE & CVSS Scoring

```
CVE = Common Vulnerabilities and Exposures
  Unique ID for each known vulnerability: CVE-2024-21626

CVSS = Common Vulnerability Scoring System (0.0 – 10.0)
┌──────────────────────────────────────────────────┐
│  0.0       — None                                │
│  0.1 – 3.9 — Low                                │
│  4.0 – 6.9 — Medium                             │
│  7.0 – 8.9 — High                               │
│  9.0 – 10.0 — Critical                          │
└──────────────────────────────────────────────────┘

CVSS factors:
  Attack Vector (Network > Adjacent > Local > Physical)
  Attack Complexity (Low > High)
  Privileges Required (None > Low > High)
  User Interaction (None > Required)
  Scope (Changed > Unchanged)
  Impact: Confidentiality, Integrity, Availability (High > Low > None)

In CI/CD:
  trivy image --severity CRITICAL,HIGH --exit-code 1 myapp:v1
  # Fail pipeline on CRITICAL/HIGH CVEs only
```

---

## 14. SBOM Formats

```
SBOM = Software Bill of Materials
  Complete inventory of all components, libraries, versions

┌─── SPDX ────────────────────────────────────────────────────┐
│  Linux Foundation / ISO standard                             │
│  Focuses on licensing + components                          │
│  Used by: US government, open source projects               │
│  Tool: syft -o spdx-json myapp:v1                          │
└──────────────────────────────────────────────────────────────┘

┌─── CycloneDX ───────────────────────────────────────────────┐
│  OWASP standard                                              │
│  Focuses on security + vulnerabilities                      │
│  Richer dependency graph, VEX support                       │
│  Tool: syft -o cyclonedx-json myapp:v1                     │
└──────────────────────────────────────────────────────────────┘

Generate SBOM in CI pipeline:
  syft myapp:v1 -o cyclonedx-json > sbom.json
  grype sbom:sbom.json    # Scan SBOM for known vulnerabilities
  # Attach SBOM as pipeline artifact
```

---

## 15. Distroless & Minimal Images

```
┌─── Image Base Options ──────────────────────────────────────┐
│                                                              │
│  ubuntu:22.04    ~78MB   Shell, apt, many packages          │
│  alpine:3.19     ~7MB    Shell, apk, musl libc              │
│  gcr.io/distroless/static  ~2MB   NO shell, NO package mgr │
│  scratch         0MB     Empty (for static Go binaries)     │
│                                                              │
│  Distroless = only app + runtime deps. Nothing else.        │
│  ✅ Smallest attack surface                                 │
│  ✅ Fewer CVEs (nothing to exploit)                         │
│  ❌ No shell → can't exec into container for debugging      │
│  ❌ Harder troubleshooting → use debug variant              │
└──────────────────────────────────────────────────────────────┘

FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o /server .

FROM gcr.io/distroless/static-debian12
COPY --from=builder /server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

---

## 16. Image Signing — Cosign & Notary

```bash
# Cosign (Sigstore project) — sign and verify container images

# Generate key pair
cosign generate-key-pair

# Sign image after build
cosign sign --key cosign.key myregistry/myapp:v1
# Signs with private key, stores signature in registry alongside image

# Verify before deploy
cosign verify --key cosign.pub myregistry/myapp:v1
# Fails if signature invalid → pipeline stops

# Keyless signing (GitHub Actions identity)
cosign sign --yes myregistry/myapp:v1
# Uses OIDC identity from CI provider — no key management!

Pipeline integration:
  Build image → Push → Sign → Deploy only if verified
```

```
Notary v2 (ORAS):
  Similar concept, OCI-native signatures
  Stored as OCI artifacts alongside images
  Used by Azure Container Registry (ACR) with notation CLI
```

---

## 17. CIS Benchmarks & kube-bench

```bash
# kube-bench — checks Kubernetes against CIS Benchmark

# Run on a node:
kube-bench run --targets=master
kube-bench run --targets=node

# Run as K8s Job:
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs job/kube-bench

# Checks include:
#   [PASS] 1.1.1 Ensure API server audit is enabled
#   [FAIL] 1.2.3 Ensure --authorization-mode includes RBAC
#   [WARN] 1.3.1 Ensure controller manager --terminated-pod-gc-threshold is set

# Also: kube-hunter (penetration testing), kubeaudit, Polaris (best practices)
```

---

## 18. SLSA Framework

```
SLSA = Supply-chain Levels for Software Artifacts  (pronounced "salsa")

┌──────────────────────────────────────────────────────────────┐
│  Level 0 — No guarantees                                     │
│  Level 1 — Documentation of build process (provenance)       │
│  Level 2 — Signed provenance from hosted build service       │
│  Level 3 — Hardened build platform (isolated, ephemeral)     │
│                                                              │
│  Provenance = verifiable record of HOW artifact was built    │
│    Who built it? What source? What build steps?              │
│    Was the build environment tampered with?                  │
│                                                              │
│  Sigstore = umbrella project providing:                      │
│    Cosign    — container image signing                       │
│    Rekor     — transparency log (public record of signatures)│
│    Fulcio    — certificate authority for keyless signing     │
│                                                              │
│  GitHub Actions: slsa-github-generator creates SLSA L3       │
│  provenance automatically                                    │
└──────────────────────────────────────────────────────────────┘
```

---

## 19. cert-manager for Kubernetes TLS

```yaml
# Install cert-manager (Helm)
# helm install cert-manager jetstack/cert-manager --set installCRDs=true

# ClusterIssuer — Let's Encrypt
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
      - http01:
          ingress:
            ingressClassName: traefik

# Certificate — auto-issued and renewed
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp-tls
  namespace: production
spec:
  secretName: myapp-tls-secret        # TLS cert stored here
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
    - myapp.example.com
    - api.myapp.example.com
  renewBefore: 360h                    # Renew 15 days before expiry

# cert-manager handles: certificate request → validation →
# issuance → storage → auto-renewal — fully automated TLS!
```
