# Git - ADVANCED OPERATIONS
## Questions Only - Test Yourself

### Cherry-Pick
1. What is `git cherry-pick`? What does it do internally?
2. Write the command to cherry-pick commit abc123.
3. How do you cherry-pick a range of commits?
4. What happens if a cherry-pick has conflicts?
5. When would you use cherry-pick instead of merge?
6. What is `git cherry-pick --no-commit`?
7. A hotfix was made on develop. How do you apply ONLY that fix to the release branch?

### Bisect
8. What is `git bisect`? What algorithm does it use?
9. Walk through the complete `git bisect` workflow (start, bad, good, reset).
10. How do you automate git bisect with a test script?
11. You have 1000 commits between the known good and bad. How many steps does bisect need?
12. When would you use bisect in real life?

### Stash
13. What is `git stash`? Where are stashes stored?
14. What is the difference between `git stash pop` and `git stash apply`?
15. How do you stash only specific files?
16. How do you stash untracked files? (`git stash -u`)
17. How do you see the contents of a stash without applying it?
18. How do you create a branch from a stash?
19. What happens to stashes when you switch branches?

### Reset, Revert, Reflog
20. What is the difference between `git reset --soft`, `--mixed`, and `--hard`?
21. What is `git revert`? How is it different from `git reset`?
22. When should you use revert vs reset?
23. What is `git reflog`? How long does it keep entries?
24. How do you recover a commit after `git reset --hard`?
25. How do you recover a deleted branch?
26. You accidentally ran `git reset --hard` and lost 3 days of work. What do you do?

### Hooks
27. What are Git hooks? Where are they stored?
28. Name 5 client-side hooks and when they trigger.
29. Name 3 server-side hooks and when they trigger.
30. Write a pre-commit hook that prevents committing files larger than 10MB.
31. Write a commit-msg hook that enforces "JIRA-123: message" format.
32. What is the pre-push hook? Give a use case.
33. How do you share hooks across a team? (Husky, pre-commit framework)
34. What is the difference between client-side and server-side hooks?

### Submodules & Subtrees
35. What are Git submodules? When would you use them?
36. How do you add a submodule? How do you update it?
37. What are the downsides of submodules?
38. What are Git subtrees? How are they different from submodules?
39. When would you prefer subtree over submodule?

### Advanced Commands
40. What is `git rebase -i` (interactive rebase)? What can you do with it?
41. What is squashing commits? Why and when?
42. What is `git blame`? How do you use it?
43. What is `git log --oneline --graph --all`?
44. What is `git diff --staged`?
45. What is `git clean -fd`?
46. What is `git worktree`? When is it useful?
47. What is `git tag`? Difference between lightweight and annotated tags?
48. How do you sign commits with GPG?

### Interview-Style
49. A production bug appeared. You know it worked 2 weeks ago. How do you find the exact commit?
50. Your team has 50 developers. How do you prevent force-pushes to main?
51. How do you enforce code review before merge?
52. What is your branching strategy and why?
53. How do you handle large binary files in Git? (Git LFS)
