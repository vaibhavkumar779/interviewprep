# Azure DevOps - PIPELINES, REPOS, BOARDS & ARTIFACTS
## Questions Only - Test Yourself

### Azure Pipelines YAML
1. What is the structure of an Azure Pipeline YAML? (trigger, pool, stages, jobs, steps)
2. What is the difference between a stage, a job, and a step?
3. What are the different trigger types? (CI triggers, PR triggers, scheduled, manual)
4. How do you set up path filters in triggers?
5. What pool types are available? (Microsoft-hosted vs self-hosted agents)
6. How do you define variables? (inline, variable groups, key vault, runtime)
7. What is a variable group? How do you link Azure Key Vault to it?
8. How do you use secrets in Azure Pipelines?
9. What are pipeline templates? How do you create and consume them?
10. What is `extends` template? How is it different from `include`?
11. How do you parameterize templates? (parameters vs variables)
12. How do you set up multi-stage pipelines? (Build → Test → Deploy Staging → Deploy Prod)
13. How do you add manual approval gates between stages?
14. What is a deployment job? How is it different from a regular job?
15. What are environments in Azure DevOps? How do you configure approvals?
16. What is `condition` in a pipeline? Write 3 examples.
17. How do you run stages/jobs in parallel?
18. How do you use `dependsOn` to control stage/job ordering?
19. What is a service connection? Name 5 types.
20. How do you publish and consume pipeline artifacts?
21. What is the difference between pipeline artifacts and build artifacts?
22. How do you cache dependencies in Azure Pipelines? (Cache@2 task)
23. What is `checkout` step? How do you do a shallow checkout?
24. How do you use Docker in Azure Pipelines? (Docker@2 task)
25. How do you deploy to Kubernetes from Azure Pipelines? (KubernetesManifest@0)

### Built-in Tasks
26. What is UsePythonVersion@0? UseDotNet@2? UseNode@1?
27. What is PublishTestResults@2? How do you publish JUnit results?
28. What is PublishCodeCoverageResults@1?
29. What is AzureRmWebAppDeployment@4?
30. How do you run custom scripts? (script, bash, powershell, pwsh)

### Azure Repos
31. What is Azure Repos? Git vs TFVC?
32. How do you set up branch policies?
33. What branch policies can you configure? (reviewers, build validation, status checks, etc.)
34. How do you enforce PR-based workflow?
35. What is a pull request template?

### Azure Boards
36. What is Azure Boards?
37. What work item types exist? (Epic, Feature, User Story, Task, Bug)
38. How do you link work items to PRs and commits?
39. What is a sprint/iteration?

### Azure Artifacts
40. What is Azure Artifacts?
41. What feed types does it support? (npm, NuGet, Maven, Python, Universal)
42. How do you publish and consume packages?
43. What are upstream sources?

### Comparison & Migration
44. How is Azure DevOps different from GitHub Actions?
45. How is Azure Pipelines different from Jenkins?
46. How would you migrate from Jenkins to Azure Pipelines?
47. How would you migrate from Azure DevOps to GitHub Actions?

### Interview-Style
48. Write a complete Azure Pipeline YAML for a .NET application: build, test, Docker build, push to ACR, deploy to AKS staging, approval gate, deploy to AKS prod.
49. Write a pipeline template that standardizes Docker build + push across all teams.
50. How do you implement infrastructure-as-code deployment using Azure Pipelines + Terraform?
51. Your pipeline takes 20 minutes. How do you optimize it?
52. How do you handle secrets for multi-environment deployments? (dev, staging, prod)
53. Describe the Azure DevOps setup at your current organization.
54. A pipeline randomly fails with "agent not available." What do you check?
55. How do you enforce that all code goes through a pipeline before reaching production?
