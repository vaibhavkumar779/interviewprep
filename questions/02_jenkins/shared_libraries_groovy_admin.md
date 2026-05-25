# Jenkins - SHARED LIBRARIES & GROOVY
## Questions Only - Test Yourself

### Shared Libraries
1. What are Jenkins Shared Libraries? Why do we need them?
2. What is the directory structure of a shared library? (vars/, src/, resources/)
3. What goes in the `vars/` directory? How are those files used in pipelines?
4. What goes in the `src/` directory?
5. How do you configure a shared library in Jenkins? (Global vs folder-level)
6. How do you import a shared library in a Jenkinsfile? Write the syntax.
7. What does `@Library('my-lib') _` mean? Why the underscore?
8. How do you version shared libraries? (branch, tag, commit)
9. How do you test shared library changes before merging?
10. Write a shared library function that standardizes Docker build + push across all teams.

### Groovy for Jenkins
11. What is Groovy? Why does Jenkins use it?
12. How do you define variables in Groovy? (def, typed)
13. How do you write a function in Groovy?
14. What are closures in Groovy? How are they used in Jenkins pipelines?
15. How do you iterate over a list and a map in Groovy?
16. What is string interpolation in Groovy? Single vs double quotes?
17. How do you handle exceptions in Groovy?
18. What are Groovy's special methods? (.each, .collect, .find, .findAll)
19. What is a CPS (Continuation Passing Style) limitation in Jenkins? Why can't you use certain Groovy features?
20. What is the @NonCPS annotation? When do you need it?

### Administration & Troubleshooting
21. Jenkins is running slow. What are 5 things you check?
22. How do you backup and restore Jenkins?
23. How do you upgrade Jenkins safely?
24. What is Jenkins Configuration as Code (JCasC)?
25. How do you set up RBAC in Jenkins?
26. How do you audit who ran what pipeline and when?
27. A pipeline worked yesterday but fails today with no code changes. What do you check?
28. How do you clean up old builds and workspace to save disk space?
29. How do you set up Jenkins in a Docker container?
30. How do you set up Jenkins HA (high availability)?

### Interview-Style
31. How have you standardized pipelines across multiple teams?
32. Describe a situation where shared libraries saved significant effort.
33. A team complains their Jenkins builds are queued for hours. How do you fix it?
34. How do you handle Jenkins plugin security vulnerabilities?
35. You need to migrate from Jenkins to another CI tool. What's your approach?
