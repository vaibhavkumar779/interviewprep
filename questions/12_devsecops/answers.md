# DevSecOps & Security - COMPREHENSIVE ANSWERS (All 60 Questions)

---

## Fundamentals

**1. DevSecOps? How different from DevOps?**
DevSecOps = DevOps + Security at every stage. Instead of security being a final gate, it's integrated into CI/CD pipelines. "Shift left" — catch vulnerabilities early.

**2. "Shift left security"?**
Move security checks earlier in the development lifecycle. Instead of finding vulnerabilities in production, find them during coding/building. Earlier = cheaper to fix.

**3. Security pipeline? Where do checks fit?**

```
DevSecOps Pipeline Flow:

  CODE            BUILD           PACKAGE         DEPLOY          RUNTIME
  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ SAST       │   │ SCA        │   │ Container  │   │ IaC scan   │   │ DAST       │
  │ Secret scan│─▶│ Dependency │─▶│ Image scan │─▶│ Compliance │─▶│ WAF/RASP   │
  │ Linting    │   │ scan       │   │ Image sign │   │ check      │   │ Monitoring │
  │ Pre-commit │   │ License    │   │ SBOM       │   │ Approval   │   │ Pen test   │
  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘

  Tools:       Tools:       Tools:         Tools:        Tools:
  SonarQube    Snyk         Trivy          Checkov       OWASP ZAP
  Semgrep      Dependabot   Cosign/Notary  tfsec         Falco
  GitLeaks     npm audit    Grype          OPA/Rego      Tenable
  CodeQL       OWASP DC     Syft (SBOM)    Sentinel      Datadog

  ←──────── Shift Left (cheaper to fix) ───────────────────────────▶
  $100 to fix                                        $10,000 to fix
```

**4. OWASP Top 10? Name 5+.**
1. **Broken Access Control** — unauthorized access to resources
2. **Cryptographic Failures** — weak encryption, exposed secrets
3. **Injection** — SQL injection, command injection
4. **Insecure Design** — missing threat modeling
5. **Security Misconfiguration** — default passwords, open ports
6. **Vulnerable Components** — outdated libraries with CVEs
7. **Authentication Failures** — weak passwords, missing MFA
8. **Software Integrity Failures** — untrusted CI/CD pipeline
9. **Logging Failures** — insufficient monitoring
10. **Server-Side Request Forgery (SSRF)**

**5. Principle of least privilege?**
Grant only the minimum permissions needed for a task. Examples: read-only ServiceAccount for monitoring pods, specific RBAC roles per team, short-lived credentials.

**6. Defense in depth?**
Multiple layers of security controls. If one fails, others still protect.

```
Defense in Depth Layers:

  ┌─────────────────────────────────────────────┐
  │  Physical Security (data centers, locks)     │
  │  ┌───────────────────────────────────────┐  │
  │  │  Network (firewall, WAF, NSG, NACLs)      │  │
  │  │  ┌─────────────────────────────────┐  │  │
  │  │  │  Host (OS hardening, patching)        │  │  │
  │  │  │  ┌───────────────────────────┐  │  │  │
  │  │  │  │  Application (auth, input val) │  │  │  │
  │  │  │  │  ┌─────────────────────┐  │  │  │  │
  │  │  │  │  │  Data (encryption,    │  │  │  │  │
  │  │  │  │  │  access control)     │  │  │  │  │
  │  │  │  │  └─────────────────────┘  │  │  │  │
  │  │  │  └───────────────────────────┘  │  │  │
  │  │  └─────────────────────────────────┘  │  │
  │  └───────────────────────────────────────┘  │
  └─────────────────────────────────────────────┘

Breaching one layer ≠ full compromise
```

Layers: network (firewall) → host (OS hardening) → application (auth) → data (encryption).

**7. Zero trust security?**
"Never trust, always verify." Every request is authenticated and authorized regardless of location. No implicit trust for internal networks. Verify identity, device, and context for every access.

---

## Static Analysis (SAST)

**8. SAST? When runs?**
Static Application Security Testing — analyzes source code for vulnerabilities without running it. Runs during: commit (pre-commit hooks), PR (CI pipeline), build stage.

**9. SonarQube?**
Open-source code quality and security platform. Checks for: bugs, vulnerabilities, code smells, duplications, coverage. Supports 30+ languages. Provides quality gates.

**10. SonarQube vs SonarCloud?**
- **SonarQube**: Self-hosted, on-premise, full control, Community (free) or Enterprise edition.
- **SonarCloud**: SaaS, hosted by Sonar, easy setup, free for open-source.

**11. Code quality vs code security in SonarQube?**
- **Quality**: Code smells, bugs, duplications, maintainability, test coverage.
- **Security**: Vulnerabilities (SQL injection, XSS, hardcoded secrets), security hotspots.

**12. Quality gates?**
Pass/fail criteria for code. Example: "No new critical vulnerabilities, code coverage > 80%, no security hotspots." Pipeline fails if quality gate fails.

**13. Integrate SonarQube with Jenkins? Azure Pipelines?**
```groovy
// Jenkins
stage('SonarQube') {
    withSonarQubeEnv('sonarqube') {
        sh 'sonar-scanner -Dsonar.projectKey=myapp'
    }
    waitForQualityGate abortPipeline: true
}
```
```yaml
# Azure Pipelines
- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'sonarqube-connection'
    projectKey: 'myapp'
- script: dotnet build
- task: SonarQubeAnalyze@5
- task: SonarQubePublish@5
```

**14. Semgrep? vs SonarQube?**
Lightweight, fast SAST tool. Pattern-based rules. Good for: custom rules, CI/CD integration. SonarQube: more comprehensive, quality + security, better dashboards. Semgrep: faster, easier custom rules.

**15. CodeQL?**
GitHub's SAST engine. Queries code like a database. Built into GitHub Actions. Powerful for finding complex vulnerability patterns.

---

## Dependency Scanning (SCA)

**16. Software Composition Analysis (SCA)?**
Scans open-source dependencies for known vulnerabilities (CVEs). Checks: direct + transitive dependencies, license compliance.

**17. Snyk?**
Developer-first security platform. Scans: code (SAST), dependencies (SCA), containers, IaC (Terraform). Auto-fix PRs. Integrates with CI/CD.

**18. Mend (WhiteSource)?**
SCA tool for dependency vulnerability scanning. License compliance. Policy enforcement. Auto-remediation PRs.

**19. Dependabot?**
GitHub's built-in dependency updater. Auto-creates PRs when dependencies have new versions or vulnerabilities. Free with GitHub.

**20. CVE? CVSS?**
- **CVE**: Common Vulnerabilities and Exposures — unique ID for each vulnerability (CVE-2024-12345)
- **CVSS**: Common Vulnerability Scoring System — severity score 0-10. Critical: 9-10, High: 7-8.9, Medium: 4-6.9, Low: 0-3.9.

**21. Handle critical vulnerability in dependency?**
1. Assess impact: Is vulnerable function used? Is it reachable?
2. Check if fix exists: update to patched version
3. If no fix: use alternative library, apply workaround, WAF rule
4. Update immediately in all environments
5. Scan to verify fix
6. Post-mortem: how to detect earlier

**22. SBOM? Why important?**
Software Bill of Materials — complete list of all components/dependencies in software. Required for: compliance (US Executive Order), vulnerability tracking, supply chain security.

**23. SPDX? CycloneDX?**
SBOM formats:
- **SPDX**: Linux Foundation standard. Focuses on licensing.
- **CycloneDX**: OWASP standard. Security-focused. More detail.
```bash
# Generate SBOM with Trivy
trivy sbom --format cyclonedx myimage:latest
```

---

## Container Security

**24. Scan Docker images for vulnerabilities?**
```bash
trivy image myapp:latest            # Trivy (free, comprehensive)
snyk container test myapp:latest    # Snyk
docker scout cves myapp:latest      # Docker Scout
grype myapp:latest                   # Anchore/Grype
```

**25. Trivy? Integrate in CI/CD?**
```yaml
# Azure Pipelines
- script: |
    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
    trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:$(Build.BuildId)
  displayName: 'Scan Docker image'
```
```groovy
// Jenkins
sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:${BUILD_NUMBER}'
```

**26. Snyk Container?**
Scans container images for OS and application dependency vulnerabilities. Provides fix recommendations (upgrade base image, update packages).

**27. Docker image security best practices? (8)**
1. Use minimal base images (alpine, slim, distroless)
2. Run as non-root user
3. Multi-stage builds (minimize final image)
4. Pin image versions (no `latest`)
5. Scan for vulnerabilities in CI
6. Don't store secrets in images
7. Use `.dockerignore`
8. Set `HEALTHCHECK`
9. Use read-only filesystem where possible
10. Drop all capabilities, add only needed ones

**28. Distroless image?**
Google's images with only the application runtime. No shell, no package manager, no utilities. Minimal attack surface.
```dockerfile
FROM gcr.io/distroless/python3-debian11
COPY app.py /app.py
CMD ["app.py"]
```

**29. Prevent running containers as root?**
```dockerfile
# In Dockerfile
RUN adduser --system appuser
USER appuser

# In K8s
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

**30. Docker Content Trust?**
Image signing. Ensures images are from trusted publishers and haven't been tampered with.
```bash
export DOCKER_CONTENT_TRUST=1
docker push myapp:v1    # Signs automatically
```

**31. Image signing? Cosign? Notary?**
- **Cosign** (Sigstore): Modern, keyless signing. Integrates with CI/CD.
- **Notary**: Docker's original signing tool (Docker Content Trust).
```bash
cosign sign myregistry/myapp:v1
cosign verify myregistry/myapp:v1
```

---

## Secret Management

**32. Secret management? Why not hardcode?**
Centralized, secure storage for sensitive data. Hardcoded secrets: visible in source code, version history, container layers. Risks: credential theft, compliance violations.

**33. HashiCorp Vault?**
Centralized secret management:
- Dynamic secrets (auto-generated, auto-rotated)
- Encryption as a service
- Lease/revocation
- Access policies
- Audit logging

**34. Azure Key Vault? Integrate with pipelines?**
Azure's secret management service. Store: secrets, certificates, keys.
```yaml
# Azure Pipelines - via variable group
variables:
- group: keyvault-linked-group    # Linked to Key Vault

# Or via task
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'my-sub'
    KeyVaultName: 'my-vault'
    SecretsFilter: 'db-password,api-key'
```

**35. AWS Secrets Manager?**
AWS service for secret storage + automatic rotation. Integrates with RDS for database credential rotation.

**36. Detect secrets in Git?**
```bash
# Pre-commit hook tools
gitleaks detect --source .          # Gitleaks
trufflehog git file://./            # TruffleHog
git-secrets --scan                   # AWS git-secrets

# CI integration
- script: gitleaks detect --exit-code 1
```

**37. Rotate secrets? How often?**
Frequency depends on risk: API keys every 90 days, database passwords every 30-90 days, certificates before expiry. Automate rotation. Use short-lived tokens where possible.

**38. Pre-commit hook for secret detection?**
```yaml
# .pre-commit-config.yaml
repos:
- repo: https://github.com/gitleaks/gitleaks
  rev: v8.18.0
  hooks:
  - id: gitleaks
```
```bash
pip install pre-commit
pre-commit install
```

---

## Infrastructure Security

**39. CIS benchmark?**
Center for Internet Security — hardening guidelines for Docker, K8s, Linux, cloud services. Prescriptive security configuration recommendations.

**40. kube-bench?**
Checks K8s cluster against CIS Kubernetes Benchmark:
```bash
kube-bench run
# Checks: API server config, etcd, scheduler, controller, node, policies
# Provides: PASS/FAIL/WARN per check with remediation steps
```

**41. Network segmentation?**
Dividing network into isolated segments. In K8s: NetworkPolicies, namespaces. In cloud: VPCs, subnets, security groups. Limits blast radius of breach.

**42. TLS/SSL in Kubernetes?**
- Ingress TLS: cert-manager + Let's Encrypt
- Service mesh mTLS: Istio/Linkerd (pod-to-pod encryption)
- etcd encryption: encrypt secrets at rest

**43. mTLS?**
Mutual TLS — both parties verify each other's identity. Service mesh (Istio) auto-configures mTLS between all pods. Prevents unauthorized service communication.

**44. cert-manager?**
K8s add-on that automates TLS certificate management. Integrates with Let's Encrypt, Vault, self-signed. Auto-renews before expiry.

**45. Audit K8s cluster security?**
1. `kube-bench` — CIS benchmark compliance
2. `kubeaudit` — security auditing
3. `kube-hunter` — penetration testing
4. `Polaris` — best practices validation
5. Review RBAC policies
6. Check Pod Security Admission levels
7. Audit logs analysis

---

## Supply Chain Security

**46. Supply chain security?**
Securing the entire software delivery pipeline: source code → build → package → deploy.

```
Supply Chain Attack Surface:

  Developer          Source Code         Build System         Registry          Runtime
  ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐    ┌─────────┐
  │ Malicious │      │ Tampered │      │ Hijacked │      │ Poisoned │    │ Exploit  │
  │ insider   │──▶  │ deps     │──▶  │ CI/CD    │──▶  │ images   │─▶ │ in prod  │
  └─────────┘      └─────────┘      └─────────┘      └─────────┘    └─────────┘
  Protection: Protection:      Protection:      Protection:   Protection:
  MFA, GPG    Snyk/Dependabot  Signed builds    Cosign/Notary Runtime scan
  signed      Lock files       Isolated agents  SBOM          Falco
  commits     SBOM             Audit logs       Admission     Monitoring
```

Prevent: compromised dependencies, tampered builds, malicious packages.

**47. SLSA?**
Supply chain Levels for Software Artifacts — framework by Google. 4 levels of increasing security:
- Level 1: Documentation of build process
- Level 2: Authenticated builds (use CI/CD)
- Level 3: Hardened builds (isolated, non-falsifiable)
- Level 4: Two-person review, hermetic builds

**48. Ensure CI/CD pipeline integrity?**
1. Signed commits (GPG)
2. Protected branches (require reviews)
3. Signed images (Cosign)
4. Immutable build artifacts
5. Audit logs for all pipeline changes
6. Least privilege for pipeline service accounts
7. Pin all tool/dependency versions

**49. Provenance?**
Metadata about how a software artifact was built: source repo, build system, builder identity, build parameters. Proves: where it came from, who built it, what was included.

**50. Sigstore?**
Open-source project for signing, verifying, and protecting software. Tools: Cosign (image signing), Fulcio (certificate authority), Rekor (transparency log). Keyless signing using OIDC identity.

---

## Interview-Style

**51. Security stages in your CI/CD pipeline?**
"1. **Pre-commit**: gitleaks for secret detection, pre-commit hooks
2. **Build**: SonarQube SAST scan, unit tests
3. **Package**: Trivy container image scan, SBOM generation
4. **Test**: DAST scan against staging, dependency scan (Snyk)
5. **Deploy**: Infrastructure scan (tfsec for Terraform), approval gates
6. **Runtime**: Runtime monitoring, periodic pen testing, audit logs"

**52. Critical CVE in base Docker image — response?**
1. **Assess**: Check CVSS score, exploitability, is our app affected?
2. **Update**: Rebuild image with patched base image
3. **Scan**: Verify fix with Trivy scan
4. **Test**: Run full test suite
5. **Deploy**: Push updated image through CI/CD
6. **Communicate**: Notify team, update incident tracker
7. **Prevent**: Add automated scanning to pipeline, set up alerts

**53. Balance security with developer velocity?**
- Automate security checks (don't be manual gate)
- Shift left (catch issues early, cheaper to fix)
- Allow non-blocking warnings for medium issues
- Block only critical/high vulnerabilities
- Provide fix suggestions (Snyk auto-fix PRs)
- Security champions in each team
- Fast feedback loops (scan in PR, not just nightly)

**54. Security scan blocks critical release?**
1. Assess: Is the vulnerability actually exploitable in our context?
2. If false positive: document and add exception
3. If real but low risk: deploy with WAF mitigation, create follow-up ticket
4. If real and high risk: fix it. Communicate delay to stakeholders.
5. Never disable security scanning permanently
6. Post-incident: improve rules to reduce false positives

**55. Design DevSecOps pipeline from scratch?**
```
Pre-commit → gitleaks (secrets), pre-commit hooks
PR stage → SonarQube (SAST), unit tests, Snyk (SCA)
Build → Docker build (multi-stage, non-root)
Scan → Trivy (container), SBOM generation, Cosign (signing)
Deploy to staging → DAST scan, integration tests
Approval gate → Manual approval for production
Deploy to prod → Infrastructure scan, compliance check
Runtime → Monitoring, alerting, periodic pen testing
```

**56. Developer committed database password to Git?**
1. **Immediately**: Rotate the password (change it in the database)
2. **Remove**: Rewrite Git history (`git filter-branch` or BFB Repo-Cleaner)
3. **Scan**: Check for any unauthorized access using the leaked password
4. **Prevent**: Add pre-commit hooks (gitleaks), use Key Vault, educate team
5. **Audit**: Check if secret was used anywhere else

**57. Enforce security policies across all pipelines?**
1. **Required templates**: Use `extends` template that enforces security stages
2. **Branch policies**: Build validation must pass (includes security scan)
3. **Policy-as-code**: OPA/Gatekeeper for K8s, Sentinel for Terraform
4. **Quality gates**: SonarQube gates that block on critical issues
5. **Centralized scanning**: Org-wide security scanning service
6. **Audit**: Regular review of pipeline configurations

**58. Compliance frameworks?**
- **SOC 2**: Service organization controls. Focus: security, availability, processing integrity.
- **ISO 27001**: Information security management system. International standard.
- **PCI-DSS**: Payment Card Industry. Required for handling card data.
- **HIPAA**: Healthcare data protection (US).
- **GDPR**: Data privacy (EU).

**59. RBAC across DevOps toolchain?**
- Azure DevOps: Project-level permissions, team-based access
- K8s: RBAC Roles + RoleBindings per namespace
- Git: Branch policies, code owners
- Registry: Team-level pull/push permissions
- Key Vault: Access policies per application/team
- Principle: least privilege everywhere

**60. Security in K8s environment?**
1. Pod Security Admission (restricted level)
2. RBAC (per-namespace, per-team)
3. NetworkPolicies (deny-all default, allow specific)
4. Image scanning (Trivy in admission webhook)
5. Secrets: External Secrets Operator + Key Vault
6. Non-root containers, read-only filesystems
7. Service mesh mTLS (Istio)
8. Audit logging enabled
9. Regular kube-bench scans
10. Runtime security (Falco)
