# DevSecOps & Security - COMPLETE
## Questions Only - Test Yourself

### Fundamentals
1. What is DevSecOps? How is it different from DevOps?
2. What does "shift left security" mean?
3. What is the security pipeline? Where do security checks fit in CI/CD?
4. What are the OWASP Top 10? Name at least 5.
5. What is the principle of least privilege?
6. What is defense in depth?
7. What is zero trust security?

### Static Analysis (SAST)
8. What is SAST? When does it run in the pipeline?
9. What is SonarQube? What does it check for?
10. What is the difference between SonarQube and SonarCloud?
11. What is code quality vs code security in SonarQube?
12. What are quality gates in SonarQube?
13. How do you integrate SonarQube with Jenkins? With Azure Pipelines?
14. What is Semgrep? How is it different from SonarQube?
15. What is CodeQL?

### Dependency Scanning (SCA)
16. What is Software Composition Analysis (SCA)?
17. What is Snyk? What types of scanning does it support?
18. What is Mend (WhiteSource)? How does it work?
19. What is Dependabot? How does it create PRs?
20. What is a CVE? What is CVSS score?
21. How do you handle a critical vulnerability in a dependency?
22. What is a Software Bill of Materials (SBOM)? Why is it important?
23. What is SPDX? What is CycloneDX?

### Container Security
24. How do you scan Docker images for vulnerabilities?
25. What is Trivy? How do you integrate it in CI/CD?
26. What is Snyk Container?
27. What are Docker image security best practices? (Name 8)
28. What is a distroless image? Why is it more secure?
29. How do you prevent running containers as root?
30. What is Docker Content Trust?
31. What is image signing? What is Cosign? Notary?

### Secret Management
32. What is secret management? Why not hardcode secrets?
33. What is HashiCorp Vault? How does it work?
34. What is Azure Key Vault? How do you integrate it with pipelines?
35. What is AWS Secrets Manager?
36. How do you detect secrets committed to Git? (git-secrets, trufflehog, gitleaks)
37. How do you rotate secrets? How often?
38. What is a pre-commit hook for secret detection?

### Infrastructure Security
39. What is CIS benchmark? For Docker, K8s, Linux?
40. What is kube-bench? How do you run it?
41. What is network segmentation?
42. How do you implement TLS/SSL in Kubernetes?
43. What is mTLS?
44. What is cert-manager in K8s?
45. How do you audit Kubernetes cluster security?

### Supply Chain Security
46. What is supply chain security?
47. What is SLSA (Supply chain Levels for Software Artifacts)?
48. How do you ensure the integrity of your CI/CD pipeline?
49. What is provenance in software supply chain?
50. What is Sigstore?

### Interview-Style
51. Walk through the security stages in your CI/CD pipeline.
52. A critical CVE is found in your base Docker image. What's your response process?
53. How do you balance security with developer velocity?
54. How do you handle a situation where a security scan blocks a critical release?
55. Design a DevSecOps pipeline from scratch. What tools at each stage?
56. A developer committed a database password to Git. What do you do?
57. How do you enforce security policies across all pipelines in the organization?
58. What compliance frameworks have you worked with? (SOC2, ISO 27001, PCI-DSS)
59. How do you implement role-based access control across your DevOps toolchain?
60. What is your approach to security in a Kubernetes environment?
