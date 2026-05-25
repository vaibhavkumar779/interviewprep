# Jenkins - PIPELINES (Declarative & Scripted)
## Questions Only - Test Yourself

### Declarative Pipeline
1. What is the basic structure of a Declarative pipeline? Write the skeleton.
2. What does `agent any` mean? What other agent options exist?
3. What is the `stages` block? Can you nest stages?
4. What is the `steps` block? What goes inside it?
5. What is the `post` block? Name all its conditions (always, success, failure, etc.)
6. How do you define environment variables in a Declarative pipeline?
7. How do you use `when` conditions? Write 3 examples.
8. How do you run stages in parallel? Write the syntax.
9. How do you parameterize a Declarative pipeline? (string, choice, boolean params)
10. What is `input` in a pipeline? How do you create an approval gate?
11. What is `options` block? Name 5 options you can set. (timeout, retry, timestamps, etc.)
12. What is `tools` block? How do you specify JDK, Maven, Node versions?
13. How do you trigger other jobs from a pipeline?
14. What is `stash` and `unstash`? When would you use them?
15. How do you archive artifacts in a pipeline?

### Scripted Pipeline
16. What is the basic structure of a Scripted pipeline? Write the skeleton.
17. How is error handling different in Scripted vs Declarative? (try/catch vs post)
18. When would you choose Scripted over Declarative?
19. Can you mix Declarative and Scripted? How? (script {} block)
20. Write a Scripted pipeline that builds, tests, and deploys.

### Credentials & Secrets
21. How do you store credentials in Jenkins?
22. What types of credentials does Jenkins support?
23. How do you use `withCredentials` in a pipeline? Write the syntax.
24. How do you prevent credential leakage in console logs?
25. How do you integrate Jenkins with HashiCorp Vault?

### Jenkinsfile Best Practices
26. Where should the Jenkinsfile live? Why?
27. Name 7 best practices for writing Jenkinsfiles.
28. How do you handle multi-branch pipelines?
29. What is a Multibranch Pipeline job? How does it discover branches?
30. How do you lint/validate a Jenkinsfile before committing?

### Interview-Style (Write Code)
31. Write a Declarative pipeline for a Java app: checkout, build with Maven, run tests, publish JUnit results, build Docker image, push to ECR.
32. Write a pipeline that deploys to staging, waits for manual approval, then deploys to prod.
33. Write a pipeline with parallel test stages: unit, integration, and e2e.
34. Write a pipeline that sends Slack notifications on success and failure.
35. Your Jenkinsfile is 500 lines long. How do you refactor it?
