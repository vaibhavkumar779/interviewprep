# Shell Scripting (Bash) - COMPLETE
## Questions Only - Test Yourself

### Basics
1. What is a shebang line? Write it for bash.
2. How do you make a script executable?
3. What is the difference between `./script.sh`, `bash script.sh`, and `source script.sh`?
4. How do you define a variable in bash? How do you use it?
5. What is the difference between single quotes and double quotes?
6. What is command substitution? (`$(command)` vs backticks)
7. How do you get the exit status of the last command? (`$?`)
8. What are special variables? ($0, $1, $#, $@, $*, $$, $!)
9. How do you read user input? (`read`)
10. How do you pass arguments to a script?

### Conditionals
11. Write the syntax for `if/elif/else/fi`.
12. What is the difference between `[ ]`, `[[ ]]`, and `(( ))`?
13. What are the string comparison operators? (-z, -n, =, !=)
14. What are the integer comparison operators? (-eq, -ne, -lt, -gt, -le, -ge)
15. What are file test operators? (-f, -d, -e, -r, -w, -x, -s)
16. Write a condition to check if a file exists and is readable.
17. Write a condition to check if a string is empty.
18. What is the `case` statement? Write the syntax.
19. Write a case statement for a menu with 4 options.
20. How do you use AND (&&) and OR (||) in conditions?

### Loops
21. Write a `for` loop that iterates over a list of values.
22. Write a `for` loop that iterates over files in a directory.
23. Write a C-style for loop. (`for ((i=0; i<10; i++))`)
24. Write a `while` loop that reads a file line by line.
25. Write a `while` loop that runs until a service is healthy.
26. What is an `until` loop? When would you use it?
27. How do you break out of a loop? Skip an iteration?
28. Write a loop that processes command-line arguments.

### Functions
29. How do you define a function in bash?
30. How do you pass arguments to a function?
31. How do you return a value from a function?
32. What is the scope of variables in bash functions? (global vs local)
33. Write a function that logs messages with timestamp and severity.
34. Write a function that retries a command N times with a delay.

### Error Handling
35. What is `set -e`? Why should you use it?
36. What is `set -u`? What does it prevent?
37. What is `set -o pipefail`? Why is it important?
38. What is `set -x`? When do you use it?
39. What does `set -euo pipefail` do? Why is it best practice?
40. How do you trap signals in bash? (`trap`)
41. Write a trap that cleans up temp files on script exit.
42. How do you handle errors in a pipeline?

### Arrays
43. How do you declare an array in bash?
44. How do you access array elements?
45. How do you get the length of an array?
46. How do you iterate over an array?
47. How do you append to an array?
48. What is an associative array? How do you declare one?

### String Operations
49. How do you get the length of a string in bash?
50. How do you extract a substring?
51. How do you replace text in a variable? (${var/old/new})
52. How do you remove a prefix/suffix from a string? (${var#pattern}, ${var%pattern})
53. How do you convert to upper/lower case?

### Real-World Scripting Scenarios (Interview)
54. Write a script that monitors disk usage and alerts if any partition exceeds 80%.
55. Write a script that checks if a list of services are running and restarts any that are down.
56. Write a script that rotates log files (rename, compress, delete old).
57. Write a script that takes a directory path and finds all duplicate files.
58. Write a script that reads a CSV file and generates an HTML report.
59. Write a script that backs up a directory to a remote server via rsync.
60. Write a script that waits for a URL to become healthy (with timeout).
61. Write a script that parses command-line options using getopts.
62. Write a script that deploys an application: stop service, copy files, run migrations, start service.
63. Write a script that collects system info (CPU, memory, disk, uptime) and outputs JSON.
64. Write a wrapper script for kubectl that adds logging and error handling.
65. Write a script that compares two directories and shows differences.
66. Write a script that validates a YAML or JSON config file.
67. Write a script that cleans up Docker resources (stopped containers, dangling images, unused volumes).
68. Write a script that sends a Slack/Teams notification on pipeline failure.
69. Write a script that auto-scales EC2 instances based on CPU usage.
70. Write an init script or health-check script for a microservice.
