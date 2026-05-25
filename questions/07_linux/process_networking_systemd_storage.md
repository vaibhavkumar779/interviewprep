# Linux - PROCESS MANAGEMENT, NETWORKING, SYSTEMD & STORAGE
## Questions Only - Test Yourself

### Process Management
1. What is a process? What is a thread?
2. How do you list all running processes? (`ps aux`)
3. What does each column in `ps aux` output mean?
4. How do you find a specific process? (`ps aux | grep` or `pgrep`)
5. What is `top`? What is `htop`? What do they show?
6. How do you sort processes by CPU usage? Memory usage?
7. How do you kill a process? What signals can you send?
8. What is the difference between `kill`, `kill -9`, `kill -15`?
9. What is SIGTERM vs SIGKILL vs SIGHUP?
10. How do you run a process in the background? (`&`, `nohup`, `disown`)
11. What is `nohup`? When would you use it?
12. What is the difference between `nohup` and `screen`/`tmux`?
13. What is a zombie process? How do you fix it?
14. What is an orphan process?
15. What is process priority? How do you change it? (`nice`, `renice`)
16. What is `uptime`? What does load average mean?
17. Load average is 8.0 on a 4-core system. Is this good or bad? Why?
18. How do you find which process is using a specific port?
19. How do you find which process is using a specific file? (`lsof`)
20. What is `/proc` filesystem? How do you get info about a process from it?

### Systemd & Services
21. What is systemd? What did it replace?
22. What is a systemd unit file? Where are they stored?
23. How do you start, stop, restart, and reload a service?
24. How do you enable/disable a service at boot?
25. How do you check the status of a service?
26. How do you view logs for a specific service? (`journalctl -u`)
27. How do you follow logs in real-time? (`journalctl -f`)
28. How do you view logs since a specific time?
29. Write a systemd unit file for a Python application.
30. What is the difference between `restart` and `reload`?
31. What are systemd targets? (multi-user.target, graphical.target)
32. How do you check boot time? (`systemd-analyze`)
33. How do you mask a service? Why would you?

### Cron & Scheduling
34. What is cron? What is crontab?
35. Write the cron syntax. Explain each field.
36. Write a cron job that runs every 5 minutes.
37. Write a cron job that runs at 2 AM every Sunday.
38. Write a cron job that runs at midnight on the 1st of every month.
39. Where are cron logs stored?
40. What is the difference between crontab and /etc/cron.d/?
41. What is `at` command? How is it different from cron?

### Networking
42. How do you check the IP address of your machine? (`ip addr`, `ifconfig`)
43. How do you check if a host is reachable? (`ping`)
44. How do you trace the network path to a host? (`traceroute`)
45. How do you check open/listening ports? (`ss -tlnp`, `netstat -tulnp`)
46. What is the difference between `ss` and `netstat`?
47. How do you make HTTP requests from the command line? (`curl`, `wget`)
48. How do you download a file? (`wget`, `curl -O`)
49. How do you do DNS lookup? (`dig`, `nslookup`, `host`)
50. What is `/etc/hosts`? What is `/etc/resolv.conf`?
51. What is `iptables`? What is `firewalld`? What is `ufw`?
52. How do you add a firewall rule to allow port 443?
53. How do you check active network connections?
54. What is `tcpdump`? How do you capture traffic on a specific port?
55. What is `nc` (netcat)? Give 3 use cases.
56. How do you check if a specific port on a remote host is open?
57. What is the difference between TCP and UDP?
58. What are well-known ports? Name 10 (SSH, HTTP, HTTPS, DNS, SMTP, etc.)

### Disk & Storage
59. How do you check disk space usage? (`df -h`)
60. How do you check directory size? (`du -sh`)
61. How do you find the largest files on the system?
62. What is `lsblk`? What information does it show?
63. How do you mount/unmount a filesystem?
64. What is `/etc/fstab`? What does it configure?
65. What is LVM? What are its components? (PV, VG, LV)
66. How do you extend a logical volume?
67. What is swap? How do you add swap space?
68. What is inode? How do you check inode usage?
69. How can a disk be "full" when `df` shows space available? (inode exhaustion)

### Users & Groups
70. How do you create a user? A group?
71. How do you add a user to a group?
72. What is the difference between `/etc/passwd` and `/etc/shadow`?
73. How do you switch users? (`su`, `su -`, `sudo`)
74. What is `sudoers`? How do you edit it safely?
75. How do you lock/unlock a user account?
76. What is the difference between `su` and `sudo`?

### SSH
77. What is SSH? How does it work?
78. How do you generate an SSH key pair?
79. What is `~/.ssh/authorized_keys`?
80. How do you copy your SSH key to a remote server?
81. What is SSH tunneling? Give 3 types (local, remote, dynamic).
82. How do you configure SSH config file for shortcuts?
83. How do you transfer files over SSH? (`scp`, `rsync`, `sftp`)
84. What is the difference between `scp` and `rsync`?
85. How do you keep an SSH session alive?

### Interview-Style
86. A server is unresponsive. You can SSH in. Walk through your diagnosis using Linux commands.
87. CPU is at 100%. How do you find and fix the cause?
88. Memory is exhausted. What commands do you run to diagnose?
89. A service won't start. Walk through your troubleshooting steps.
90. You need to find which process is listening on port 8080 and kill it. Write the commands.
91. Write a cron job that backs up a database dump to S3 every 6 hours.
92. A developer says they can't SSH to a server. What do you check?
93. You need to replace a string in all config files under /etc/app/. Write the command.
94. Explain the boot process of a Linux system.
95. What is the OOM Killer? When does it trigger?
