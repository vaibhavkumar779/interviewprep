# Linux - COMPREHENSIVE ANSWERS (All 162 Questions)

---

# PART 1: FILE OPERATIONS & TEXT PROCESSING (67 Qs)

---

## File & Directory Operations

**1. List all files including hidden ones?**
```bash
ls -a          # includes . and ..
ls -A          # hidden files, excludes . and ..
ls -la         # long format + hidden
```

**2. What does `ls -la` show? Explain each column.**
```
-rw-r--r-- 1 vaibhav staff 4096 Jan 15 10:30 file.txt
│          │ │       │     │    │              └── filename
│          │ │       │     │    └── last modified date/time
│          │ │       │     └── file size in bytes
│          │ │       └── group owner
│          │ └── user owner
│          └── hard link count
└── permissions: type(d/-/l) + owner(rwx) + group(rwx) + others(rwx)
```

**3. Create a file? (touch, echo, cat, vim)**
```bash
touch newfile.txt                    # Empty file (updates timestamp if exists)
echo "hello" > newfile.txt           # With content (overwrites)
echo "more" >> newfile.txt           # Append
cat > newfile.txt <<EOF              # Heredoc
line 1
line 2
EOF
vim newfile.txt                      # Editor
```

**4. Create nested directories in one command?**
```bash
mkdir -p /opt/app/config/templates   # -p creates parent dirs as needed
```

**5. Copy a file? A directory recursively?**
```bash
cp source.txt dest.txt               # File
cp -r /src/dir /dest/dir             # Directory recursive
cp -rp /src /dest                    # Preserve permissions + timestamps
```

**6. Move/rename a file?**
```bash
mv old.txt new.txt                   # Rename
mv file.txt /new/path/              # Move
mv /old/dir /new/path/dir           # Move directory
```

**7. Delete a file? A non-empty directory?**
```bash
rm file.txt                          # File
rm -r dirname/                       # Directory recursively
rm -rf dirname/                      # Force, no prompts (DANGEROUS)
rmdir emptydir/                      # Only works if empty
```

**8. Hard link vs soft (symbolic) link?**
| Hard Link | Soft Link (Symlink) |
|---|---|
| Same inode number | Different inode |
| Can't cross filesystems | Can cross filesystems |
| Can't link to directories | Can link to directories |
| Original deleted → link still works | Original deleted → broken link |
| `ln file hardlink` | `ln -s file symlink` |

**9. Create a symbolic link?**
```bash
ln -s /path/to/target /path/to/link
ln -s /opt/app/config.yml /etc/app/config.yml
```

**10. Find a file by name?**
```bash
find / -name "config.yml"            # Exact name (case-sensitive)
find / -iname "config.yml"           # Case-insensitive
find /opt -name "*.log"              # Pattern match
find . -type f -name "*.py"          # Files only
find . -type d -name "logs"          # Directories only
```

**11. Find files modified in the last 24 hours?**
```bash
find /var/log -mtime -1              # Modified within last 1 day
find /var/log -mmin -60              # Modified within last 60 minutes
find /var/log -newer reference.txt   # Newer than reference file
```

**12. Find files larger than 100MB?**
```bash
find / -type f -size +100M
find / -type f -size +100M -exec ls -lh {} \;   # With details
find / -type f -size +1G 2>/dev/null             # Larger than 1GB, suppress errors
```

**13. Find all .log files and delete them?**
```bash
find /var/log -name "*.log" -delete
find /var/log -name "*.log" -exec rm {} \;
find /var/log -name "*.log" -mtime +30 -delete   # Older than 30 days
```

**14. What is `locate`? How different from `find`?**
- `locate`: Searches pre-built database (fast, but may be stale). Update DB with `updatedb`.
- `find`: Searches filesystem in real-time (slower, always current).
```bash
locate config.yml        # Instant, uses database
find / -name config.yml  # Real-time search
```

**15. `which` vs `whereis`?**
```bash
which python3     # Shows path of executable in PATH: /usr/bin/python3
whereis python3   # Shows binary, source, and man page locations
```

---

## Viewing & Editing Files

**16. Difference between cat, less, more, head, tail?**
| Command | Purpose |
|---|---|
| `cat` | Print entire file to stdout (small files) |
| `less` | Page through file (forward + backward, search) |
| `more` | Page forward only (older) |
| `head` | First 10 lines (default) |
| `tail` | Last 10 lines (default) |

**17. View the last 50 lines?**
```bash
tail -50 file.txt
tail -n 50 file.txt
```

**18. Follow a log file in real-time?**
```bash
tail -f /var/log/app.log              # Follow appended data
tail -f /var/log/app.log | grep ERROR # Follow with filtering
tail -F /var/log/app.log              # Follow even if file is rotated
```

**19. What is `wc`? Count lines, words, characters?**
```bash
wc file.txt           # lines  words  chars  filename
wc -l file.txt        # Lines only
wc -w file.txt        # Words only
wc -c file.txt        # Bytes
wc -m file.txt        # Characters
cat file.txt | wc -l  # Count from pipe
```

**20. View binary file in hexadecimal?**
```bash
xxd file.bin          # Hex dump
hexdump -C file.bin   # Hex + ASCII
od -A x -t x1z file.bin  # Octal dump in hex
```

---

## Text Processing (grep, awk, sed)

**21. What is grep? Basic syntax?**
```bash
grep "pattern" file.txt
# grep = Global Regular Expression Print
# Searches for lines matching a pattern
```

**22. Search recursively in a directory?**
```bash
grep -r "TODO" /path/to/dir/
grep -rn "TODO" .                    # With line numbers
grep -rl "TODO" .                    # Only filenames
```

**23. Search case-insensitively?**
```bash
grep -i "error" /var/log/app.log
```

**24. Show line numbers with matches?**
```bash
grep -n "error" file.txt
# 42:error occurred at line 42
# 87:another error here
```

**25. Count matches?**
```bash
grep -c "error" file.txt             # Count of matching lines
grep -o "error" file.txt | wc -l     # Count of all occurrences
```

**26. Invert the match (exclude lines)?**
```bash
grep -v "DEBUG" file.txt             # Lines NOT containing DEBUG
grep -v "^#" config.txt              # Lines NOT starting with #
grep -v "^$" file.txt                # Remove empty lines
```

**27. Search for multiple patterns?**
```bash
grep -E "error|warning|critical" file.txt
egrep "error|warning|critical" file.txt     # Same
grep -e "error" -e "warning" file.txt       # Alternative
```

**28. Show context lines (before/after)?**
```bash
grep -B3 "ERROR" file.txt            # 3 lines Before
grep -A3 "ERROR" file.txt            # 3 lines After
grep -C3 "ERROR" file.txt            # 3 lines Context (both)
```

**29. Search only in specific file types?**
```bash
grep -r --include="*.py" "import os" .
grep -r --include="*.{yaml,yml}" "image:" .
grep -r --exclude-dir=".git" "TODO" .
```

**30. Difference between grep, egrep, fgrep?**
- `grep`: Basic regex
- `egrep` = `grep -E`: Extended regex (supports `|`, `+`, `?`, `()` without escaping)
- `fgrep` = `grep -F`: Fixed strings (no regex, faster for literal strings)

**31. What is awk? How different from grep?**
- `grep`: Finds lines matching a pattern (filtering)
- `awk`: Processes and transforms data column by column (programming language)
```bash
grep "ERROR" file.txt         # Find lines with ERROR
awk '{print $1, $5}' file.txt # Print 1st and 5th columns
```

**32. Print specific columns with awk?**
```bash
awk '{print $1, $3}' file.txt        # 1st and 3rd columns (space-delimited)
ps aux | awk '{print $1, $2, $11}'   # User, PID, command
```

**33. Change field separator?**
```bash
awk -F: '{print $1, $3}' /etc/passwd    # Colon-separated
awk -F, '{print $1, $2}' data.csv       # Comma-separated
awk -F'\t' '{print $2}' data.tsv        # Tab-separated
```

**34. Add conditions in awk?**
```bash
awk '$3 > 100 {print $1, $3}' file.txt           # Column 3 > 100
awk '/ERROR/ {print $0}' file.txt                 # Lines matching ERROR
awk '$1 == "root" {print $0}' /etc/passwd         # Column 1 equals "root"
awk 'NR >= 10 && NR <= 20' file.txt               # Lines 10-20
```

**35. Calculate sum/average with awk?**
```bash
# Sum of column 3
awk '{sum += $3} END {print "Total:", sum}' data.txt

# Average of column 3
awk '{sum += $3; count++} END {print "Avg:", sum/count}' data.txt
```

**36. Process CSV files with awk?**
```bash
awk -F, 'NR > 1 {print $1, $3}' data.csv       # Skip header, print cols
awk -F, '{sum += $3} END {print sum}' data.csv   # Sum column 3
awk -F, 'NR==1 {print; next} $3 > 100' data.csv # Header + filtered rows
```

**37. Top 5 processes by memory usage?**
```bash
ps aux | awk 'NR>1 {print $4, $11}' | sort -rn | head -5
# OR
ps aux --sort=-%mem | head -6
```

**38. Extract IP addresses from Apache access log?**
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
# Explanation: $1 is the IP in Apache common log format
```

**39. What is sed? What does stream editor mean?**
`sed` processes text line-by-line from a stream (file or pipe). It reads, transforms, and outputs without loading entire file into memory.

**40. Replace text with sed?**
```bash
sed 's/old/new/' file.txt             # First occurrence per line
sed 's/old/new/g' file.txt            # All occurrences (global)
echo "hello world" | sed 's/world/earth/'
```

**41. Difference between `s/old/new/` and `s/old/new/g`?**
- Without `g`: Replaces only the **first** occurrence on each line
- With `g` (global): Replaces **all** occurrences on each line

**42. Edit file in-place?**
```bash
sed -i 's/old/new/g' file.txt                    # Linux
sed -i.bak 's/old/new/g' file.txt                # Create backup before edit
```

**43. Delete lines matching a pattern?**
```bash
sed '/pattern/d' file.txt              # Delete lines with pattern
sed '/^#/d' config.txt                 # Delete comment lines
sed '/^$/d' file.txt                   # Delete empty lines
sed '1,5d' file.txt                    # Delete lines 1-5
```

**44. Print specific line ranges?**
```bash
sed -n '10,20p' file.txt               # Print lines 10-20
sed -n '5p' file.txt                   # Print only line 5
sed -n '/START/,/END/p' file.txt       # Between patterns
```

**45. Insert line before/after a pattern?**
```bash
sed '/pattern/i\New line before' file.txt    # Insert before
sed '/pattern/a\New line after' file.txt     # Insert after
```

**46. Replace text only on specific lines?**
```bash
sed '5s/old/new/' file.txt             # Only on line 5
sed '10,20s/old/new/g' file.txt        # Lines 10-20
sed '/section/s/old/new/g' file.txt    # Lines matching "section"
```

---

## Piping & Redirection

**47. What is a pipe? How does it work?**
`|` sends stdout of one command as stdin to the next command. Creates a pipeline.
```bash
cat file.txt | grep "error" | wc -l
# cat outputs file → grep filters errors → wc counts lines
```

**48. Difference between `>` and `>>`?**
```bash
echo "hello" > file.txt     # Overwrite (creates or truncates)
echo "world" >> file.txt    # Append (creates or adds to end)
```

**49. What is `2>`, `2>&1`, and `&>`?**
```bash
command 2> errors.log            # Redirect stderr to file
command > out.log 2>&1           # Redirect both stdout and stderr to same file
command &> all.log               # Same (bash shorthand)
command 2>/dev/null              # Discard stderr
```

**50. What is `/dev/null`? When redirect to it?**
A special file that discards all data written to it ("black hole"). Use to suppress unwanted output:
```bash
command > /dev/null 2>&1         # Suppress all output (cron jobs, background tasks)
find / -name "*.log" 2>/dev/null # Suppress permission denied errors
```

**51. What is `tee`?**
Writes to both stdout AND a file simultaneously:
```bash
command | tee output.log         # Display + save
command | tee -a output.log      # Display + append
echo "msg" | tee file1 file2    # Write to multiple files
```

**52. What is `xargs`? 3 examples.**
Converts stdin into arguments for another command:
```bash
# Example 1: Delete files found by find
find . -name "*.tmp" | xargs rm

# Example 2: Kill processes by name
pgrep -f "myapp" | xargs kill

# Example 3: Download multiple URLs
cat urls.txt | xargs -n1 curl -O

# -P for parallel execution
find . -name "*.gz" | xargs -P 4 gunzip
```

**53. One-liner: Find Python files, search for TODO, count matches?**
```bash
find . -name "*.py" | xargs grep -c "TODO" | awk -F: '{sum += $2} END {print sum}'
# OR
grep -r --include="*.py" -c "TODO" . | awk -F: '{sum += $2} END {print sum}'
```

**54. One-liner: Disk usage of top 10 largest directories?**
```bash
du -sh /* 2>/dev/null | sort -rh | head -10
# OR for subdirectories:
du -h --max-depth=1 /path | sort -rh | head -10
```

**55. One-liner: Extract unique IPs from log, sorted by frequency?**
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20
```

---

## Permissions

**56. What does `chmod 755` mean?**
```
7 = rwx (owner: read+write+execute = 4+2+1)
5 = r-x (group: read+execute = 4+0+1)
5 = r-x (others: read+execute = 4+0+1)
```
Common values: 755 (executables/directories), 644 (regular files), 600 (private files), 700 (private directories)

**57. `chmod` vs `chown`?**
- `chmod`: Changes **permissions** (who can read/write/execute)
- `chown`: Changes **ownership** (which user/group owns the file)
```bash
chmod 644 file.txt              # Set permissions
chown user:group file.txt       # Change owner and group
```

**58. Octal vs symbolic notation?**
```bash
# Octal
chmod 755 file.txt

# Symbolic
chmod u+rwx,g+rx,o+rx file.txt
chmod u=rwx,g=rx,o=rx file.txt   # Exact set
chmod a+x script.sh              # Add execute for all
chmod o-w file.txt               # Remove write for others
```

**59. Sticky bit? SUID? SGID?**
- **SUID (4xxx)**: File runs as file owner, not executor. `chmod 4755 file` → `-rwsr-xr-x`. Example: `/usr/bin/passwd`
- **SGID (2xxx)**: File runs as group owner; directories → new files inherit group. `chmod 2755 dir`
- **Sticky bit (1xxx)**: Only file owner can delete in directory. `chmod 1777 /tmp` → `drwxrwxrwt`

**60. Change ownership recursively?**
```bash
chown -R appuser:appgroup /opt/app/
```

**61. Default permissions for files and directories?**
- Files: 644 (rw-r--r--)
- Directories: 755 (rwxr-xr-x)
- Determined by umask

**62. What is `umask`? How does it affect file creation?**
```bash
umask              # Show current (typically 0022)
umask 0027         # Set new umask
# Default permissions = max - umask
# Files: 666 - 022 = 644
# Dirs:  777 - 022 = 755
```

---

## Interview-Style

**63. Find log files >1GB, older than 7 days, compress them?**
```bash
find /var/log -name "*.log" -size +1G -mtime +7 -exec gzip {} \;
# Or with nice formatting:
find /var/log -name "*.log" -size +1G -mtime +7 -print0 | xargs -0 gzip
```

**64. Top 10 most frequently accessed URLs from nginx log?**
```bash
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10
# $7 is the URL path in nginx default log format
```

**65. Server running out of disk space - debugging process?**
```bash
# 1. Check overall disk usage
df -h

# 2. Find which partition is full
df -h | grep "9[0-9]%\|100%"

# 3. Find largest directories
du -sh /* 2>/dev/null | sort -rh | head -10

# 4. Drill down into the largest
du -sh /var/* | sort -rh | head -10

# 5. Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -20

# 6. Check for deleted files still held by processes
lsof | grep deleted

# 7. Check inode usage (disk "full" but df shows space)
df -i

# 8. Clean up: old logs, package cache, docker images
journalctl --vacuum-size=500M
docker system prune -f
apt clean / yum clean all
```

**66. Parse CSV and extract 3rd column?**
```bash
awk -F, '{print $3}' data.csv
cut -d',' -f3 data.csv
# Handle quoted commas: use csvtool or Python
```

**67. Replace config value across 50 files?**
```bash
find /etc/app/ -name "*.conf" -exec sed -i 's/old_value/new_value/g' {} \;
# OR with grep to first verify:
grep -rl "old_value" /etc/app/ | xargs sed -i 's/old_value/new_value/g'
```

---

# PART 2: PROCESS MANAGEMENT, NETWORKING, SYSTEMD & STORAGE (95 Qs)

---

## Process Management

**1. What is a process? What is a thread?**
- **Process**: Running instance of a program. Has its own memory space, PID, file descriptors.
- **Thread**: Lightweight unit within a process. Shares memory with other threads in same process. Cheaper to create.

**2. List all running processes?**
```bash
ps aux                    # All processes (BSD syntax)
ps -ef                    # All processes (POSIX syntax)
ps aux --sort=-%mem       # Sorted by memory
```

**3. Each column in `ps aux` output?**
```
USER   PID  %CPU %MEM   VSZ    RSS  TTY  STAT  START  TIME  COMMAND
root   1    0.0  0.1  169532  13248  ?   Ss    Jan01  5:30  /sbin/init
│      │    │    │     │       │     │   │     │      │     └── command
│      │    │    │     │       │     │   │     │      └── CPU time used
│      │    │    │     │       │     │   │     └── start time
│      │    │    │     │       │     │   └── state: S=sleeping, R=running, Z=zombie
│      │    │    │     │       │     └── terminal
│      │    │    │     │       └── Resident Set Size (physical memory in KB)
│      │    │    │     └── Virtual memory size
│      │    │    └── % of physical memory
│      │    └── % of CPU
│      └── Process ID
└── Owner
```

**4. Find a specific process?**
```bash
ps aux | grep nginx
pgrep nginx              # Returns PIDs only
pgrep -a nginx           # PIDs + full command
pidof nginx              # PID of named process
```

**5. `top` vs `htop`?**
- `top`: Built-in, shows real-time process info (CPU, memory, load). Press `P` sort by CPU, `M` by memory, `k` to kill.
- `htop`: Enhanced version with color, mouse support, tree view, easier to use. Install: `apt install htop`

**6. Sort processes by CPU/Memory?**
```bash
ps aux --sort=-%cpu | head -10       # Top 10 by CPU
ps aux --sort=-%mem | head -10       # Top 10 by memory
top -o %CPU                          # In top, sort by CPU
```

**7. Kill a process? What signals?**
```bash
kill PID                  # Send SIGTERM (15) - graceful
kill -9 PID               # Send SIGKILL (9) - force kill
kill -HUP PID             # Send SIGHUP (1) - reload config
killall nginx             # Kill all by name
pkill -f "python app.py"  # Kill by pattern
```

**8. `kill` vs `kill -9` vs `kill -15`?**
- `kill PID` = `kill -15 PID` = SIGTERM: Graceful shutdown, process can clean up
- `kill -9 PID` = SIGKILL: Immediate kill, process cannot catch or ignore it
- Always try SIGTERM first, SIGKILL as last resort

**9. SIGTERM vs SIGKILL vs SIGHUP?**
- **SIGTERM (15)**: "Please terminate." Process can catch, clean up, then exit.
- **SIGKILL (9)**: "Die now." Cannot be caught. Kernel terminates immediately. May leave corrupted state.
- **SIGHUP (1)**: "Hang up." Often used to reload config without restarting (e.g., nginx).

**10. Run process in background?**
```bash
command &                 # Background (tied to terminal)
nohup command &           # Background + survives logout
disown %1                 # Detach running job from terminal
```

**11. What is `nohup`?**
"No hangup." Prevents process from receiving SIGHUP when terminal closes. Output goes to `nohup.out`.
```bash
nohup python long_task.py > output.log 2>&1 &
```

**12. `nohup` vs `screen`/`tmux`?**
- `nohup`: Simple, just keeps process running after logout. Can't reattach.
- `screen`/`tmux`: Terminal multiplexer. Can detach and **reattach** to session. Multiple windows. Interactive.
```bash
tmux new -s mysession     # Create session
# Ctrl+B, D              # Detach
tmux attach -t mysession  # Reattach
```

**13. Zombie process? How to fix?**
A process that has finished but its parent hasn't called `wait()` to collect exit status. Shows as `Z` in `ps`.
```bash
ps aux | grep Z           # Find zombies
# Fix: Kill the parent process (forces init/systemd to reap)
kill -9 <parent_PID>
```

**14. Orphan process?**
A process whose parent has terminated. Adopted by init (PID 1) which will eventually reap it. Not harmful.

**15. Process priority? nice/renice?**
```bash
nice -n 10 command        # Start with lower priority (higher nice = lower priority)
renice -n 5 -p PID        # Change priority of running process
# Nice values: -20 (highest priority) to 19 (lowest)
# Only root can set negative nice values
```

**16. `uptime`? Load average?**
```bash
uptime
# 10:30:01 up 45 days, load average: 1.50, 2.30, 3.10
#                                     1min  5min  15min
```
Load average = average number of processes in runnable + uninterruptible state.

**17. Load average 8.0 on 4-core system - good or bad?**
**Bad.** Load average should ideally be ≤ number of cores. 8.0 on 4 cores means the system is **overloaded** — 4 processes running + 4 waiting. Each process gets ~50% of expected CPU. Investigate with `top` to find CPU-hogging processes.

**18. Find which process is using a specific port?**
```bash
ss -tlnp | grep :8080
netstat -tlnp | grep :8080
lsof -i :8080
fuser 8080/tcp
```

**19. Find which process is using a specific file?**
```bash
lsof /var/log/app.log       # Processes with file open
lsof -c nginx               # Files opened by nginx
lsof -u vaibhav             # Files opened by user
fuser /var/log/app.log       # PIDs using the file
```

**20. `/proc` filesystem?**
Virtual filesystem exposing kernel/process info as files:
```bash
cat /proc/cpuinfo            # CPU information
cat /proc/meminfo            # Memory information
cat /proc/<PID>/status       # Process status
cat /proc/<PID>/cmdline      # Command that started process
cat /proc/<PID>/fd/          # Open file descriptors
ls /proc/<PID>/environ       # Environment variables
```

---

## Systemd & Services

**21. What is systemd? What did it replace?**
Init system and service manager for Linux. Replaced SysVinit and Upstart. Manages services, handles dependencies, parallel startup, logging (journald), and more.

**22. Systemd unit file? Where stored?**
Configuration file for a service/timer/mount. Locations:
- `/usr/lib/systemd/system/` — package-provided units (don't edit)
- `/etc/systemd/system/` — admin customizations (highest priority)
- `/run/systemd/system/` — runtime units

**23. Start, stop, restart, reload a service?**
```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx     # Stop + Start
sudo systemctl reload nginx      # Reload config without stopping
sudo systemctl daemon-reload     # After editing unit files
```

**24. Enable/disable at boot?**
```bash
sudo systemctl enable nginx      # Start at boot
sudo systemctl disable nginx     # Don't start at boot
sudo systemctl enable --now nginx # Enable + start immediately
```

**25. Check service status?**
```bash
systemctl status nginx
# Shows: loaded, active/inactive, PID, memory, recent logs
systemctl is-active nginx        # Just "active" or "inactive"
systemctl is-enabled nginx       # "enabled" or "disabled"
```

**26. View logs for a service?**
```bash
journalctl -u nginx              # All logs for nginx
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx -n 100      # Last 100 lines
journalctl -u nginx -p err      # Only error level
```

**27. Follow logs in real-time?**
```bash
journalctl -u nginx -f           # Follow (like tail -f)
journalctl -f                    # All system logs
```

**28. View logs since specific time?**
```bash
journalctl --since "2024-01-15 10:00:00"
journalctl --since "2 hours ago"
journalctl --since yesterday --until today
```

**29. Systemd unit file for Python app?**
```ini
[Unit]
Description=My Python Web Application
After=network.target
Wants=postgresql.service

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/venv/bin/python app.py
Restart=on-failure
RestartSec=5
Environment=APP_ENV=production
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**30. `restart` vs `reload`?**
- **restart**: Stops the process completely, then starts it. Brief downtime.
- **reload**: Sends SIGHUP (or similar) to re-read config. No downtime. Not all services support it.

**31. Systemd targets?**
Targets replace runlevels. Group of units:
- `multi-user.target`: Multi-user, no GUI (server standard)
- `graphical.target`: Multi-user with GUI
- `rescue.target`: Single-user, minimal services
- `emergency.target`: Emergency shell
```bash
systemctl get-default                    # Current target
systemctl set-default multi-user.target  # Change default
```

**32. Check boot time?**
```bash
systemd-analyze                          # Total boot time
systemd-analyze blame                    # Per-service boot time
systemd-analyze critical-chain           # Critical path
```

**33. Mask a service?**
```bash
sudo systemctl mask nginx      # Prevents starting (even manually)
sudo systemctl unmask nginx    # Allow starting again
```
Use to completely prevent a service from running (e.g., disabling conflicting services).

---

## Cron & Scheduling

**34. What is cron? crontab?**
- `cron`: Daemon that runs scheduled commands
- `crontab`: User's cron table (file with scheduled jobs)
```bash
crontab -e    # Edit your crontab
crontab -l    # List your crontab
crontab -r    # Remove your crontab
```

**35. Cron syntax?**
```
┌────── minute (0-59)
│ ┌──── hour (0-23)
│ │ ┌── day of month (1-31)
│ │ │ ┌ month (1-12)
│ │ │ │ ┌ day of week (0-7, 0/7=Sun)
│ │ │ │ │
* * * * * command
```

**36. Cron job every 5 minutes?**
```
*/5 * * * * /opt/scripts/check_health.sh
```

**37. Cron job at 2 AM every Sunday?**
```
0 2 * * 0 /opt/scripts/weekly_backup.sh
```

**38. Cron job at midnight on 1st of every month?**
```
0 0 1 * * /opt/scripts/monthly_report.sh
```

**39. Where are cron logs stored?**
```bash
/var/log/cron          # RHEL/CentOS
/var/log/syslog        # Debian/Ubuntu (grep CRON)
journalctl -u cron     # systemd
```

**40. crontab vs /etc/cron.d/?**
- `crontab`: Per-user cron tables. Each user manages their own.
- `/etc/cron.d/`: System cron files. Includes username field. Managed by packages/admins.
- `/etc/cron.daily/`, `/etc/cron.hourly/`: Drop scripts here (no syntax needed).

**41. `at` command?**
Schedules a command to run **once** at a specific time (unlike cron which repeats):
```bash
at 2:00 AM tomorrow
> /opt/scripts/migration.sh
> Ctrl+D
atq       # List pending at jobs
atrm 3    # Remove job #3
```

---

## Networking

**42. Check IP address?**
```bash
ip addr show              # Modern (preferred)
ip a                      # Shorthand
ifconfig                  # Legacy (may need net-tools package)
hostname -I               # Just the IPs
```

**43. Check if host is reachable?**
```bash
ping -c 4 google.com      # 4 pings
ping -c 1 -W 3 host       # 1 ping, 3 second timeout
```

**44. Trace network path?**
```bash
traceroute google.com      # Shows each hop
tracepath google.com       # No root needed
mtr google.com             # Combined ping + traceroute (real-time)
```

**45. Check open/listening ports?**
```bash
ss -tlnp                   # TCP listening ports with process
ss -ulnp                   # UDP listening ports
netstat -tulnp              # All protocols (legacy)
# t=TCP, u=UDP, l=listening, n=numeric, p=process
```

**46. `ss` vs `netstat`?**
- `ss`: Modern, faster, more info. Part of iproute2.
- `netstat`: Legacy, from net-tools package. Being deprecated.

**47. HTTP requests from command line?**
```bash
curl https://api.example.com           # GET request
curl -X POST -d '{"key":"val"}' -H "Content-Type: application/json" URL
curl -s URL | jq .                     # Silent + parse JSON
wget URL                               # Download file
wget -q -O- URL                        # Output to stdout
```

**48. Download a file?**
```bash
wget https://example.com/file.tar.gz
curl -O https://example.com/file.tar.gz    # -O saves with original name
curl -o myfile.tar.gz URL                   # -o custom name
```

**49. DNS lookup?**
```bash
dig example.com              # Detailed DNS query
dig example.com A            # A record specifically
dig @8.8.8.8 example.com    # Use specific DNS server
nslookup example.com        # Interactive/simple
host example.com             # Simple output
```

**50. `/etc/hosts` vs `/etc/resolv.conf`?**
- `/etc/hosts`: Local hostname→IP mappings (checked first). Manual DNS override.
- `/etc/resolv.conf`: DNS server configuration (nameserver 8.8.8.8)

**51. `iptables` vs `firewalld` vs `ufw`?**
- `iptables`: Low-level firewall tool. Complex but powerful. Raw rules.
- `firewalld`: Dynamic firewall manager (RHEL/CentOS). Zone-based. `firewall-cmd`.
- `ufw`: Uncomplicated Firewall (Ubuntu). Simple interface over iptables.

**52. Add firewall rule to allow port 443?**
```bash
# ufw
sudo ufw allow 443/tcp

# firewalld
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

**53. Check active network connections?**
```bash
ss -tn                     # Active TCP connections
ss -s                      # Summary statistics
netstat -an                # All connections (legacy)
```

**54. `tcpdump`? Capture traffic on specific port?**
```bash
sudo tcpdump -i eth0 port 80                    # HTTP traffic
sudo tcpdump -i any port 443 -w capture.pcap    # Save to file
sudo tcpdump -i eth0 host 10.0.0.5             # Specific host
sudo tcpdump -i eth0 -nn port 53               # DNS traffic
```

**55. `nc` (netcat)? 3 use cases.**
```bash
# 1. Test if port is open
nc -zv host 80

# 2. Simple TCP server/client
nc -l 8080                    # Listen on port 8080
nc host 8080                  # Connect to it

# 3. Transfer file
nc -l 9999 > received.txt    # Receiver
nc host 9999 < send.txt      # Sender
```

**56. Check if specific port on remote host is open?**
```bash
nc -zv remotehost 443 -w 3    # netcat with timeout
telnet remotehost 443          # Classic
curl -v telnet://host:443      # Using curl
ss -tn | grep :443             # Local connections to 443
```

**57. TCP vs UDP?**
| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable, ordered delivery | No guarantee |
| Slower (handshake, ACK) | Faster (no overhead) |
| HTTP, SSH, FTP, SMTP | DNS, DHCP, streaming, gaming |

**58. Well-known ports? Name 10.**
| Port | Service |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 53 | DNS |
| 25 | SMTP |
| 110 | POP3 |
| 143 | IMAP |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |
| 8080 | HTTP alt |

---

## Disk & Storage

**59. Check disk space usage?**
```bash
df -h                      # Human-readable disk usage
df -h /                    # Specific mount point
df -ih                     # Inode usage
```

**60. Check directory size?**
```bash
du -sh /var/log             # Total size of directory
du -sh /var/log/*           # Size of each item
du -h --max-depth=1 /      # One level deep
```

**61. Find largest files on the system?**
```bash
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -20
du -ah / 2>/dev/null | sort -rh | head -20
```

**62. `lsblk`?**
Lists block devices (disks, partitions) in tree format:
```bash
lsblk
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0  100G  0 disk
# ├─sda1   8:1    0    1G  0 part /boot
# └─sda2   8:2    0   99G  0 part /
```

**63. Mount/unmount a filesystem?**
```bash
sudo mount /dev/sdb1 /mnt/data        # Mount
sudo umount /mnt/data                   # Unmount
mount | grep sdb                        # Check mounts
```

**64. `/etc/fstab`?**
Configures automatic mounts at boot:
```
# device          mountpoint  type  options     dump  pass
/dev/sdb1         /data       ext4  defaults    0     2
UUID=abc-123      /backup     xfs   defaults    0     0
```

**65. LVM? Components?**
Logical Volume Manager - flexible disk management:
- **PV (Physical Volume)**: Physical disk/partition (`pvcreate /dev/sdb`)
- **VG (Volume Group)**: Pool of PVs (`vgcreate myvg /dev/sdb`)
- **LV (Logical Volume)**: Virtual partition from VG (`lvcreate -L 50G -n mylv myvg`)

**66. Extend a logical volume?**
```bash
lvextend -L +10G /dev/myvg/mylv       # Add 10GB
resize2fs /dev/myvg/mylv               # Resize filesystem (ext4)
xfs_growfs /dev/myvg/mylv              # For XFS
# Or in one command:
lvextend -L +10G -r /dev/myvg/mylv    # -r auto-resizes filesystem
```

**67. Swap? Add swap space?**
Virtual memory on disk when RAM is full:
```bash
# Create swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# Add to /etc/fstab for persistence:
# /swapfile none swap sw 0 0
swapon --show       # Check swap usage
free -h             # Shows swap in memory info
```

**68. Inode? Check inode usage?**
Inode = data structure storing file metadata (permissions, size, location on disk). Each file has one inode.
```bash
df -i               # Inode usage per filesystem
ls -i file.txt      # Show inode number
stat file.txt       # Detailed inode info
```

**69. Disk "full" but `df` shows space? (inode exhaustion)**
When all inodes are used (millions of tiny files), no new files can be created even with disk space available.
```bash
df -i               # Check inode usage — will show 100% IUse%
# Fix: Find and remove excessive small files
find / -xdev -type d | while read dir; do echo "$(ls -1A "$dir" | wc -l) $dir"; done | sort -rn | head -20
```

---

## Users & Groups

**70. Create user? Group?**
```bash
sudo useradd -m -s /bin/bash newuser    # Create user with home dir
sudo passwd newuser                      # Set password
sudo groupadd developers                 # Create group
```

**71. Add user to a group?**
```bash
sudo usermod -aG docker vaibhav         # Add to group (append)
groups vaibhav                            # Check user's groups
id vaibhav                                # Detailed user info
```

**72. `/etc/passwd` vs `/etc/shadow`?**
- `/etc/passwd`: Username, UID, GID, home, shell. Readable by all.
  `vaibhav:x:1000:1000:Vaibhav:/home/vaibhav:/bin/bash`
- `/etc/shadow`: Hashed passwords. Readable only by root.
  `vaibhav:$6$hash...:19000:0:99999:7:::`

**73. Switch users?**
```bash
su username          # Switch user (keeps current environment)
su - username        # Switch user (login shell, fresh environment)
sudo command         # Run single command as root
sudo -u user command # Run as specific user
```

**74. `sudoers`? Edit safely?**
```bash
sudo visudo          # Safely edit /etc/sudoers (syntax check before save)
# Example entry:
# vaibhav ALL=(ALL) NOPASSWD: ALL     # Full sudo without password
# %developers ALL=(ALL) /usr/bin/docker # Group, specific commands only
```

**75. Lock/unlock user account?**
```bash
sudo usermod -L username     # Lock (prepends ! to password hash)
sudo usermod -U username     # Unlock
sudo passwd -l username      # Lock (alternative)
sudo passwd -u username      # Unlock (alternative)
```

**76. `su` vs `sudo`?**
- `su`: Switch to another user entirely (need their password)
- `sudo`: Run single command as root/another user (need YOUR password, controlled by sudoers)
- `sudo` is preferred: logs commands, granular permissions, doesn't need root password

---

## SSH

**77. What is SSH? How does it work?**
Secure Shell — encrypted protocol for remote access. Uses asymmetric encryption for key exchange, then symmetric encryption for data transfer. Default port 22.

**78. Generate SSH key pair?**
```bash
ssh-keygen -t ed25519 -C "vaibhav@email.com"    # Modern, preferred
ssh-keygen -t rsa -b 4096 -C "vaibhav@email.com" # RSA alternative
# Creates: ~/.ssh/id_ed25519 (private) + ~/.ssh/id_ed25519.pub (public)
```

**79. `~/.ssh/authorized_keys`?**
File on remote server containing public keys of users allowed to connect. One key per line.

**80. Copy SSH key to remote server?**
```bash
ssh-copy-id user@remote-host
# OR manually:
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

**81. SSH tunneling? 3 types.**
```bash
# Local forwarding (access remote service via local port)
ssh -L 8080:localhost:80 user@remote
# Access remote port 80 via localhost:8080

# Remote forwarding (expose local service to remote)
ssh -R 9090:localhost:3000 user@remote
# Remote can access your local port 3000 via their 9090

# Dynamic forwarding (SOCKS proxy)
ssh -D 1080 user@remote
# All traffic through localhost:1080 goes via remote
```

**82. SSH config file?**
```
# ~/.ssh/config
Host dev-server
    HostName 10.0.0.5
    User vaibhav
    IdentityFile ~/.ssh/id_ed25519
    Port 22

Host jump
    HostName bastion.company.com
    User admin

Host internal-server
    HostName 10.0.1.10
    User deploy
    ProxyJump jump          # Jump through bastion
```
Usage: `ssh dev-server`

**83. Transfer files over SSH?**
```bash
scp file.txt user@host:/path/           # Copy to remote
scp user@host:/path/file.txt .          # Copy from remote
scp -r dir/ user@host:/path/            # Copy directory

rsync -avz dir/ user@host:/path/dir/    # Sync (efficient, incremental)
sftp user@host                           # Interactive FTP over SSH
```

**84. `scp` vs `rsync`?**
- `scp`: Simple copy. Copies everything every time. No resume.
- `rsync`: Intelligent sync. Only transfers differences. Can resume. Compression. Much faster for updates.

**85. Keep SSH session alive?**
```bash
# Client-side (~/.ssh/config)
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Server-side (/etc/ssh/sshd_config)
ClientAliveInterval 60
ClientAliveCountMax 3
```

---

## Interview-Style

**86. Server unresponsive, you can SSH in — diagnosis?**
```bash
# 1. Check system load
uptime                     # Load average

# 2. Check CPU usage
top -bn1 | head -20        # Top processes

# 3. Check memory
free -h                    # RAM and swap usage

# 4. Check disk
df -h                      # Disk space

# 5. Check running processes
ps aux --sort=-%cpu | head -10

# 6. Check for OOM kills
dmesg | grep -i "oom\|killed"

# 7. Check system logs
journalctl -p err --since "1 hour ago"

# 8. Check network
ss -s                      # Connection summary
ss -tn | wc -l            # Number of connections
```

**87. CPU at 100% — find and fix?**
```bash
top -bn1 | head -20        # Identify high-CPU process
ps aux --sort=-%cpu | head -5
# Check if it's a legitimate process or runaway
strace -p PID              # What is it doing?
# If runaway: kill PID
# If legitimate but unexpected: check cron, check for infinite loops
# If system-wide: check for fork bomb, malware
```

**88. Memory exhausted — diagnose?**
```bash
free -h                     # Overall memory
ps aux --sort=-%mem | head -10    # Top memory consumers
smem -rs rss                # Per-process memory (accurate)
cat /proc/meminfo           # Detailed memory info
dmesg | grep -i oom         # Check for OOM kills
slabtop                     # Kernel slab allocator (cache)
# Solutions: kill leak, add swap, increase RAM, fix application leak
```

**89. Service won't start — troubleshooting?**
```bash
# 1. Check status and error
systemctl status myservice
journalctl -u myservice -n 50

# 2. Check config syntax
nginx -t                    # (for nginx example)

# 3. Check port conflicts
ss -tlnp | grep :PORT

# 4. Check permissions
ls -la /path/to/binary
ls -la /path/to/config

# 5. Check dependencies
systemctl list-dependencies myservice

# 6. Try running manually
/usr/bin/myservice --config /etc/myservice.conf
```

**90. Find process on port 8080 and kill it?**
```bash
# Method 1
lsof -i :8080
kill $(lsof -t -i :8080)

# Method 2
ss -tlnp | grep :8080
kill <PID from output>

# Method 3
fuser -k 8080/tcp          # Find and kill in one command
```

**91. Cron job: backup DB to S3 every 6 hours?**
```bash
0 */6 * * * /opt/scripts/db_backup.sh >> /var/log/db_backup.log 2>&1
```
Script content:
```bash
#!/bin/bash
set -euo pipefail
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
pg_dump mydb | gzip > /tmp/backup_${TIMESTAMP}.sql.gz
aws s3 cp /tmp/backup_${TIMESTAMP}.sql.gz s3://my-backups/db/
rm /tmp/backup_${TIMESTAMP}.sql.gz
```

**92. Developer can't SSH to server — what to check?**
1. Is the server running? `ping server`
2. Is SSH service running? `systemctl status sshd`
3. Is port 22 open? `ss -tlnp | grep :22`
4. Firewall blocking? `ufw status` / `firewall-cmd --list-all`
5. Check SSH key: does their public key exist in `~/.ssh/authorized_keys`?
6. Check `/etc/ssh/sshd_config`: `PasswordAuthentication`, `AllowUsers`
7. Check `/var/log/auth.log` or `journalctl -u sshd` for errors
8. Network/security group rules (cloud): Is port 22 allowed from their IP?

**93. Replace a string in all config files under /etc/app/?**
```bash
grep -rl "old_string" /etc/app/ | xargs sed -i 's/old_string/new_string/g'
# OR
find /etc/app/ -name "*.conf" -exec sed -i 's/old_string/new_string/g' {} +
```

**94. Linux boot process?**
1. **BIOS/UEFI**: Hardware initialization, finds bootloader
2. **GRUB**: Bootloader loads kernel + initramfs
3. **Kernel**: Initializes hardware, mounts root filesystem
4. **initramfs**: Temporary root filesystem, loads drivers needed for real root
5. **systemd (PID 1)**: Starts services in parallel according to targets
6. **Login**: getty/display manager presents login

**95. OOM Killer? When does it trigger?**
Out of Memory Killer — kernel mechanism that kills processes when system runs out of memory. Selects process based on "badness score" (memory usage, priority). Check with `dmesg | grep -i oom`. Prevent with: proper resource limits, swap, monitoring, K8s resource limits.
