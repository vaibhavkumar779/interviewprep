# Linux - FILE OPERATIONS & TEXT PROCESSING
## Questions Only - Test Yourself

### File & Directory Operations
1. How do you list all files including hidden ones?
2. What does `ls -la` show? Explain each column.
3. How do you create a file? (touch, echo, cat, vim)
4. How do you create nested directories in one command?
5. How do you copy a file? A directory recursively?
6. How do you move/rename a file?
7. How do you delete a file? A non-empty directory?
8. What is the difference between hard link and soft (symbolic) link?
9. How do you create a symbolic link?
10. How do you find a file by name? (`find` command)
11. How do you find files modified in the last 24 hours?
12. How do you find files larger than 100MB?
13. How do you find all `.log` files and delete them?
14. What is `locate`? How is it different from `find`?
15. What does `which` do? What does `whereis` do?

### Viewing & Editing Files
16. What is the difference between `cat`, `less`, `more`, `head`, `tail`?
17. How do you view the last 50 lines of a file?
18. How do you follow a log file in real-time? (`tail -f`)
19. What is `wc`? How do you count lines, words, characters?
20. How do you view a binary file in hexadecimal?

### Text Processing (CRITICAL - grep, awk, sed)
21. What is `grep`? Write the basic syntax.
22. How do you search recursively in a directory? (`grep -r`)
23. How do you search case-insensitively? (`grep -i`)
24. How do you show line numbers with grep matches? (`grep -n`)
25. How do you count matches? (`grep -c`)
26. How do you invert the match (exclude lines)? (`grep -v`)
27. How do you search for multiple patterns? (`grep -E "pat1|pat2"`)
28. How do you show 3 lines before and after a match? (`grep -B3 -A3`)
29. How do you search only in specific file types? (`grep --include="*.py"`)
30. What is the difference between `grep`, `egrep`, and `fgrep`?

31. What is `awk`? How is it different from grep?
32. How do you print specific columns with awk? (`awk '{print $1, $3}'`)
33. How do you change the field separator? (`awk -F:`)
34. How do you add conditions in awk? (`awk '$3 > 100'`)
35. How do you calculate sum/average with awk?
36. How do you use awk to process CSV files?
37. Write an awk command to find the top 5 processes by memory usage.
38. Write an awk command to extract IP addresses from an Apache access log.

39. What is `sed`? What does stream editor mean?
40. How do you replace text? (`sed 's/old/new/g'`)
41. What is the difference between `s/old/new/` and `s/old/new/g`?
42. How do you edit a file in-place? (`sed -i`)
43. How do you delete lines matching a pattern? (`sed '/pattern/d'`)
44. How do you print specific line ranges? (`sed -n '10,20p'`)
45. How do you insert a line before/after a pattern?
46. How do you replace text only on specific lines?

### Piping & Redirection
47. What is a pipe (`|`)? How does it work?
48. What is the difference between `>` and `>>`?
49. What is `2>`, `2>&1`, and `&>`?
50. What is `/dev/null`? When would you redirect to it?
51. What is `tee`? How do you write to a file and stdout simultaneously?
52. What is `xargs`? Write 3 examples.
53. Write a one-liner: Find all Python files, search for "TODO", and count total matches.
54. Write a one-liner: Get disk usage of top 10 largest directories.
55. Write a one-liner: Extract unique IP addresses from a log file, sorted by frequency.

### Permissions
56. What does `chmod 755` mean? Break down each digit.
57. What is the difference between `chmod` and `chown`?
58. What is the octal vs symbolic notation for permissions?
59. What is the sticky bit? SUID? SGID?
60. How do you change ownership recursively?
61. What are default permissions for files and directories?
62. What is `umask`? How does it affect file creation?

### Interview-Style
63. Find all log files larger than 1GB, older than 7 days, and compress them.
64. Write a command to find the 10 most frequently accessed URLs from an nginx log.
65. A server is running out of disk space. Walk through your debugging process using Linux commands.
66. Parse a CSV file using only command-line tools and extract the 3rd column.
67. You need to replace a config value across 50 files. How do you do it?
