# Python - OS, SUBPROCESS, APIS & ADVANCED
## Questions Only - Test Yourself

### os & sys Modules
1. How do you get the current working directory in Python?
2. How do you list files in a directory?
3. How do you check if a file or directory exists?
4. How do you create a directory? Create nested directories?
5. How do you delete a file? A directory?
6. How do you get environment variables?
7. How do you set environment variables from Python?
8. How do you walk a directory tree recursively?
9. How do you get the file size? Last modified time?
10. What is `os.path.join()`? Why use it instead of string concatenation?
11. What is `sys.argv`? How do you parse command-line arguments?
12. What is `sys.exit()`? What exit codes mean?
13. What is the `pathlib` module? How is it different from `os.path`?

### subprocess Module (CRITICAL)
14. What is the `subprocess` module? Why use it instead of `os.system()`?
15. What is `subprocess.run()`? Write the basic syntax.
16. How do you capture stdout and stderr from a subprocess?
17. What is `capture_output=True`? What is `text=True`?
18. How do you check if a command succeeded? What is `check=True`?
19. How do you pipe the output of one command to another?
20. How do you set a timeout for a subprocess?
21. How do you run a command with elevated privileges?
22. What is `subprocess.Popen()`? When would you use it over `run()`?
23. Write a Python script that runs `git status` and parses the output.
24. Write a Python script that runs `kubectl get pods` and checks for unhealthy pods.
25. How do you handle `CalledProcessError`?

### REST APIs & HTTP
26. What is the `requests` library?
27. How do you make GET, POST, PUT, DELETE requests?
28. How do you pass headers, query parameters, and request body?
29. How do you handle authentication? (Bearer token, Basic auth, API key)
30. What is `response.json()`? What is `response.status_code`?
31. What is `response.raise_for_status()`? When should you use it?
32. How do you handle pagination in API responses?
33. How do you handle rate limiting?
34. How do you upload a file via API?
35. Write a script that calls the GitHub API to list all repos for a user.
36. Write a script that calls the Azure DevOps API to get pipeline runs.

### OOP (Object-Oriented Programming)
37. What is a class? What is an object?
38. What is `__init__`? What is `self`?
39. What is inheritance? Write an example.
40. What is method overriding?
41. What are class methods vs static methods vs instance methods?
42. What is `__str__` vs `__repr__`?
43. What are dunder/magic methods? Name 5.
44. What is polymorphism in Python?
45. What is encapsulation? How do you make attributes private?

### Testing
46. What is `pytest`? How do you write a test function?
47. What is `unittest`? How is it different from `pytest`?
48. What are fixtures in pytest?
49. What is mocking? When would you use it?
50. How do you test a function that calls an external API?

### DevOps Scripting Scenarios (Interview)
51. Write a Python script to parse Jenkins build logs and extract failure reasons.
52. Write a Python script to check if all services in a K8s namespace are healthy.
53. Write a Python script that monitors disk usage and sends an alert if above 80%.
54. Write a Python script to rotate AWS access keys older than 90 days.
55. Write a Python script to read a YAML file, modify a value, and write it back.
56. Write a Python script to generate a deployment report from Azure DevOps API.
57. Write a Python script to clean up Docker images older than 30 days.
58. Write a Python script to validate JSON/YAML configuration files.
59. Write a Python script to compare two config files and show differences.
60. Write a Python script to bulk-create Jira/Azure DevOps work items from a CSV.
