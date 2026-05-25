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
