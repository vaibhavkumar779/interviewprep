# Git - WORKFLOWS, STRATEGIES, GERRIT & GOOGLE REPO
## Questions Only - Test Yourself

### Branching Strategies
1. Explain GitFlow. Draw the branch diagram. Name all branch types.
2. Explain Trunk-Based Development. What are its prerequisites?
3. Explain GitHub Flow. How is it simpler than GitFlow?
4. What is GitLab Flow? How does it differ?
5. Which strategy works best for continuous deployment? Why?
6. Which strategy works best for scheduled/versioned releases? Why?
7. What are feature flags? How do they enable trunk-based development?
8. What is a release branch? When do you create one?
9. What is a hotfix branch? Walk through the hotfix workflow.
10. Your team has 5 devs. Which branching strategy do you recommend and why?
11. Your team has 100 devs across 3 timezones. Which strategy and why?

### Code Review
12. What makes a good code review process?
13. What should reviewers look for in a code review?
14. How do you handle disagreements in code reviews?
15. What is the ideal PR size? Why?

### Gerrit (IMPORTANT for Ciena)
16. What is Gerrit? Who uses it and why?
17. How does the Gerrit workflow differ from GitHub PRs?
18. What does `git push origin HEAD:refs/for/main` do in Gerrit?
19. What is a Change-Id in Gerrit? Why is it needed?
20. What is the scoring system in Gerrit? (+1, +2, -1, -2)
21. What happens when you amend a commit in Gerrit? How is it different from GitHub?
22. How does Gerrit integrate with Jenkins for CI?
23. What is the Gerrit trigger plugin in Jenkins?
24. How do you submit (merge) a change in Gerrit?
25. What is "Verified" vs "Code-Review" in Gerrit?

### Google Repo (IMPORTANT for Ciena)
26. What is Google Repo? What problem does it solve?
27. What is a manifest file in Google Repo? What does it contain?
28. Write a sample manifest.xml with 3 repositories.
29. What is `repo init`? What does `-u` flag mean?
30. What is `repo sync`? What does it do?
31. What is `repo forall -c <command>`? Give an example.
32. What is `repo start`? How does it relate to branches?
33. How is Google Repo different from Git submodules?
34. Why would an embedded software team (like Ciena ON) use Google Repo?
35. How do you manage dependencies between repos in a multi-repo setup?

### Interview-Style
36. Your team is using GitFlow but deployments are slow. What do you suggest?
37. How do you migrate from one branching strategy to another with minimal disruption?
38. You've never used Gerrit but the team uses it. How do you ramp up?
39. What are the pros and cons of monorepo vs multi-repo?
40. How do you enforce branch protection rules? Name 5 rules you'd set.
