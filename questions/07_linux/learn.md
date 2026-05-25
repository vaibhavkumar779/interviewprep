# Linux Administration — Deep-Dive Learning Guide

---

## 1. Linux Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Space                                │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │ User Apps  │  │  Daemons   │  │   Shell    │            │
│  │ (nginx,    │  │ (sshd,     │  │ (bash,     │            │
│  │  python)   │  │  systemd)  │  │  zsh)      │            │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘            │
│        │               │               │                     │
│  ┌─────▼───────────────▼───────────────▼──────┐             │
│  │          GNU C Library (glibc)              │             │
│  │          System Call Interface              │             │
│  └─────────────────────┬──────────────────────┘             │
└────────────────────────┼────────────────────────────────────┘
                         │ system calls (open, read, write, fork, exec)
┌────────────────────────▼────────────────────────────────────┐
│                    Kernel Space                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐ │
│  │ Process  │  │ Memory   │  │ File     │  │ Network    │ │
│  │ Mgmt     │  │ Mgmt     │  │ Systems  │  │ Stack      │ │
│  │ (sched)  │  │ (virtual)│  │ (VFS)    │  │ (TCP/IP)   │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────────┘ │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                Device Drivers                         │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    Hardware                                  │
│  CPU    RAM    Disk    NIC    GPU                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Filesystem Hierarchy

```
/
├── bin/       → Essential user binaries (ls, cp, cat) — symlink to /usr/bin
├── sbin/      → System binaries (fdisk, iptables) — symlink to /usr/sbin
├── etc/       → Configuration files (nginx.conf, fstab, passwd)
├── home/      → User home directories (/home/vaibhav)
├── root/      → Root user's home
├── var/       → Variable data
│   ├── log/   → Log files (syslog, auth.log)
│   ├── lib/   → State data (databases, package info)
│   └── run/   → Runtime data (PID files, sockets)
├── tmp/       → Temporary files (cleared on reboot)
├── usr/       → User programs and data
│   ├── bin/   → User binaries
│   ├── lib/   → Libraries
│   ├── local/ → Locally installed software
│   └── share/ → Shared data (man pages, docs)
├── opt/       → Third-party software (/opt/myapp)
├── proc/      → Virtual filesystem — process info (live kernel data)
├── sys/       → Virtual filesystem — device/kernel info
├── dev/       → Device files (/dev/sda, /dev/null, /dev/tty)
├── mnt/       → Temporary mount points
├── media/     → Removable media mount points
└── boot/      → Kernel, bootloader (vmlinuz, initramfs, grub)
```

---

## 3. File Permissions

```
-rwxr-xr-- 1 vaibhav devops 4096 May 25 10:00 deploy.sh
│├──┤├──┤├──┤  │       │
││   │    │    │       └── Group
││   │    │    └── Owner
││   │    └── Others:  r-- (read only)
││   └── Group:   r-x (read + execute)
│└── Owner:   rwx (read + write + execute)
└── File type: - (file), d (dir), l (link), b (block), c (char)
```

### Numeric Permissions

```
r = 4    w = 2    x = 1

rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

chmod 755 script.sh   →  rwxr-xr-x (owner: all, group+others: read+exec)
chmod 644 config.txt  →  rw-r--r-- (owner: read+write, others: read)
chmod 600 secret.key  →  rw------- (owner only)
chmod 777 file        →  NEVER DO THIS IN PRODUCTION!
```

### Special Permissions

```
SUID (4):  chmod u+s /usr/bin/passwd   →  runs as file owner (not caller)
           -rwsr-xr-x  (s in owner exec position)
           Use case: passwd command needs root to write /etc/shadow

SGID (2):  chmod g+s /shared/dir      →  new files inherit group
           drwxrwsr-x  (s in group exec position)
           Use case: shared team directories

Sticky(1): chmod +t /tmp               →  only owner can delete own files
           drwxrwxrwt  (t in others exec position)
           Use case: /tmp — everyone writes, nobody deletes others' files
```

### Ownership

```bash
chown user:group file       # Change owner and group
chown -R user:group dir/    # Recursive
chgrp devops file           # Change group only
```

---

## 4. Process Management

```
┌─── Process States ──────────────────────────────────────────┐
│                                                              │
│  Running (R) ←──→ Sleeping (S/D)                            │
│      │                │                                      │
│      └────────────────┼──► Stopped (T) ← Ctrl+Z / kill -STOP│
│                       │                                      │
│                       └──► Zombie (Z) — finished but parent  │
│                              hasn't collected exit code       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# ─── Viewing processes ───
ps aux                          # All processes, full detail
ps -ef                          # All processes, full format
ps aux --sort=-%mem | head -10  # Top 10 by memory
top / htop                      # Real-time monitor
pstree -p                       # Process tree with PIDs

# ─── Background / Foreground ───
command &                       # Run in background
Ctrl+Z                         # Suspend current process
bg                              # Resume in background
fg                              # Bring to foreground
jobs                            # List background jobs
nohup command &                 # Survives terminal close
disown                          # Detach from terminal

# ─── Signals ───
kill PID                        # SIGTERM (15) — polite shutdown
kill -9 PID                     # SIGKILL (9) — force kill (no cleanup!)
kill -HUP PID                   # SIGHUP (1) — reload config (nginx, sshd)
kill -USR1 PID                  # User-defined signal
killall nginx                   # Kill by name
pkill -f "python deploy"       # Kill by command pattern
```

### Key Signals

| Signal | Number | Default | Use Case |
|--------|--------|---------|----------|
| SIGTERM | 15 | Terminate | Graceful shutdown (default kill) |
| SIGKILL | 9 | Kill | Force kill (cannot be caught!) |
| SIGHUP | 1 | Terminate | Reload config (daemons) |
| SIGINT | 2 | Terminate | Ctrl+C |
| SIGSTOP | 19 | Stop | Pause process (cannot be caught!) |
| SIGCONT | 18 | Continue | Resume paused process |
| SIGUSR1 | 10 | Terminate | Custom: log rotation, debug toggle |

---

## 5. systemd — Service Management

```
┌─── systemd (PID 1) ──────────────────────────────────────────┐
│                                                               │
│  The init system — first process, manages ALL services       │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ sshd     │  │ nginx    │  │ docker   │  │ cron     │    │
│  │ .service │  │ .service │  │ .service │  │ .service │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
│                                                               │
│  Unit types: .service, .socket, .timer, .mount, .target      │
└───────────────────────────────────────────────────────────────┘
```

```bash
# ─── Service management ───
systemctl start nginx          # Start now
systemctl stop nginx           # Stop now
systemctl restart nginx        # Stop + Start
systemctl reload nginx         # Reload config (no downtime)
systemctl status nginx         # Current status + recent logs
systemctl enable nginx         # Start on boot
systemctl disable nginx        # Don't start on boot
systemctl is-active nginx      # Check if running
systemctl is-enabled nginx     # Check if starts on boot

# ─── Listing ───
systemctl list-units --type=service              # Running services
systemctl list-units --type=service --state=failed  # Failed services
systemctl list-unit-files --type=service         # All installed services

# ─── Custom service unit ───
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target
Requires=postgresql.service

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/server --port 8080
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload          # After editing unit files
journalctl -u myapp -f           # Follow logs
journalctl -u myapp --since "1 hour ago"
```

---

## 6. Package Management

| Distro | Package Manager | Commands |
|--------|----------------|----------|
| Ubuntu/Debian | apt | `apt update`, `apt install`, `apt remove` |
| RHEL/CentOS | yum/dnf | `dnf install`, `dnf update`, `dnf remove` |
| Alpine | apk | `apk add`, `apk del`, `apk update` |

```bash
# ─── Debian/Ubuntu ───
apt update                      # Refresh package list
apt upgrade                     # Upgrade all packages
apt install nginx               # Install
apt remove nginx                # Remove (keep config)
apt purge nginx                 # Remove + config
apt autoremove                  # Remove orphaned deps
dpkg -l | grep nginx            # List installed
apt list --installed            # All installed packages

# ─── RHEL/CentOS ───
dnf check-update                # Check for updates
dnf install nginx               # Install
dnf remove nginx                # Remove
rpm -qa | grep nginx            # List installed
```

---

## 7. Disk & Storage

```bash
# ─── Disk usage ───
df -h                           # Filesystem usage (human-readable)
du -sh /var/log/                # Directory size
du -sh /var/log/* | sort -rh | head -10  # Largest subdirs
ncdu /var/                      # Interactive disk usage (install ncdu)

# ─── Disk partitions ───
lsblk                           # Block devices tree
fdisk -l                        # Partition table
blkid                           # Filesystem UUIDs

# ─── Mount ───
mount /dev/sdb1 /mnt/data       # Mount partition
umount /mnt/data                # Unmount
# Persistent mount → edit /etc/fstab:
# /dev/sdb1  /mnt/data  ext4  defaults  0  2

# ─── LVM (Logical Volume Manager) ───
# Physical Volume → Volume Group → Logical Volume
pvcreate /dev/sdb               # Create PV
vgcreate myvg /dev/sdb          # Create VG from PV
lvcreate -L 10G -n mylv myvg   # Create 10GB LV
mkfs.ext4 /dev/myvg/mylv       # Create filesystem
lvextend -L +5G /dev/myvg/mylv # Extend by 5GB
resize2fs /dev/myvg/mylv       # Resize filesystem to match
```

---

## 8. Networking

```bash
# ─── IP & Interfaces ───
ip addr show                    # All interfaces + IPs
ip route show                   # Routing table
ip link set eth0 up/down        # Enable/disable interface

# ─── DNS ───
cat /etc/resolv.conf            # DNS servers
nslookup google.com             # DNS lookup
dig google.com                  # Detailed DNS query
host google.com                 # Simple DNS lookup

# ─── Connectivity ───
ping -c 4 google.com            # ICMP ping (4 packets)
traceroute google.com           # Path to destination
mtr google.com                  # Combined ping + traceroute (live)
curl -I https://example.com     # HTTP headers only
wget https://example.com/file   # Download file

# ─── Ports & Connections ───
ss -tulnp                       # All listening ports + processes
  # t=TCP, u=UDP, l=listening, n=numeric, p=process
netstat -tulnp                  # Legacy equivalent
lsof -i :8080                   # What's using port 8080

# ─── Firewall (iptables) ───
iptables -L -n -v               # List rules
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # Allow HTTP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # Allow SSH
iptables -A INPUT -j DROP                        # Drop everything else

# ─── Firewall (firewalld — RHEL/CentOS) ───
firewall-cmd --list-all
firewall-cmd --add-service=http --permanent
firewall-cmd --reload
```

---

## 9. SSH

```bash
# ─── Key-based auth (recommended) ───
ssh-keygen -t ed25519 -C "vaibhav@devops"   # Generate key pair
ssh-copy-id user@server                       # Copy pub key to server
ssh user@server                               # Login (no password!)

# ─── SSH config (~/.ssh/config) ───
Host prod-web
    HostName 10.0.1.50
    User deploy
    IdentityFile ~/.ssh/prod_key
    Port 22

ssh prod-web                    # Uses config above

# ─── Tunneling ───
ssh -L 8080:localhost:3000 user@server    # Local port forward
  # Access server's port 3000 via localhost:8080

ssh -R 8080:localhost:3000 user@server    # Remote port forward
  # Server accesses your port 3000 via its localhost:8080

ssh -D 1080 user@server                  # SOCKS proxy

# ─── SCP & rsync ───
scp file.txt user@server:/path/          # Copy file to server
scp -r dir/ user@server:/path/           # Copy directory
rsync -avz --progress dir/ user@server:/path/  # Sync (only changes)
```

---

## 10. Log Management

```bash
# ─── journalctl (systemd logs) ───
journalctl                              # All logs
journalctl -u nginx                     # Specific service
journalctl -u nginx --since "2 hours ago"
journalctl -f                           # Follow (tail -f equivalent)
journalctl -p err                       # Errors only
journalctl -b                           # Since last boot
journalctl --disk-usage                 # Log storage used

# ─── Traditional logs ───
/var/log/syslog          # System messages (Debian/Ubuntu)
/var/log/messages        # System messages (RHEL/CentOS)
/var/log/auth.log        # Authentication (SSH, sudo)
/var/log/kern.log        # Kernel messages
/var/log/nginx/          # Nginx access + error logs

# ─── Log rotation ───
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser appgroup
    postrotate
        systemctl reload myapp
    endscript
}
```

---

## 11. Users & Groups

```bash
useradd -m -s /bin/bash -G docker,sudo deploy  # Create user
usermod -aG docker deploy                        # Add to group
userdel -r olduser                               # Delete user + home
passwd deploy                                    # Set password

groupadd devops                                  # Create group
id deploy                                        # Show UID, GID, groups
cat /etc/passwd                                  # User list
cat /etc/group                                   # Group list
cat /etc/shadow                                  # Password hashes (root only)

# ─── sudo ───
visudo                           # Edit /etc/sudoers safely
# deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
```

---

## 12. Performance Troubleshooting

```
Symptom → Tool → Diagnosis

Slow system?
  ├── top/htop          → High CPU? Which process?
  ├── free -h           → Memory usage, swap usage
  ├── iostat -x 1       → Disk I/O bottleneck? (%util near 100?)
  ├── vmstat 1          → CPU wait (wa), context switches
  └── dmesg | tail      → Kernel errors, OOM kills

Slow network?
  ├── ping              → Latency, packet loss
  ├── mtr               → Where packets are lost
  ├── ss -s             → Connection count/state
  └── iftop / nethogs   → Bandwidth per connection/process

Disk full?
  ├── df -h             → Which filesystem is full?
  ├── du -sh /* | sort -rh  → Largest directories
  ├── find / -size +100M    → Large files
  └── lsof +L1          → Deleted files still held open (restart service!)

Process issues?
  ├── strace -p PID     → System calls (what is process doing?)
  ├── lsof -p PID       → Open files/sockets
  └── /proc/PID/        → Process details (status, maps, fd/)
```

---

## 13. Cron Jobs

```bash
# ─── Edit crontab ───
crontab -e                      # Edit current user's crontab
crontab -l                      # List crontab

# ─── Format ───
# ┌───────────── minute (0-59)
# │ ┌───────────── hour (0-23)
# │ │ ┌───────────── day of month (1-31)
# │ │ │ ┌───────────── month (1-12)
# │ │ │ │ ┌───────────── day of week (0-7, 0=7=Sunday)
# │ │ │ │ │
# * * * * * command

0 2 * * *   /opt/scripts/backup.sh         # Daily at 2 AM
*/5 * * * * /opt/scripts/health_check.sh   # Every 5 minutes
0 0 * * 0   /opt/scripts/weekly_report.sh  # Sunday midnight
0 9 1 * *   /opt/scripts/monthly.sh        # 1st of month at 9 AM
```

---

## 14. Important One-Liners for Interviews

```bash
# Find and kill process on port 8080
lsof -ti:8080 | xargs kill -9

# Top 10 largest files
find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10

# Monitor log in real-time for errors
tail -f /var/log/syslog | grep -i error

# Count connections by state
ss -ant | awk '{print $1}' | sort | uniq -c | sort -rn

# Disk usage by directory (sorted)
du -sh /* 2>/dev/null | sort -rh

# Find files modified in last 24 hours
find /var/log -mtime -1 -type f

# Check if port is open remotely
nc -zv hostname 443

# Memory usage per process (top 10)
ps aux --sort=-%mem | head -10

# System uptime and load
uptime    # load averages: 1min, 5min, 15min (should be < num CPUs)
```
