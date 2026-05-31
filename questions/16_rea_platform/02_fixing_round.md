# Round 2: Server/Website Deployed Fixing Round — REA Platform

> A live website is deployed and broken. You need to debug and fix it.
> This round tests debugging a website running on a web server — Nginx, Apache, app servers (Node.js, Python, Go).
> It covers DNS, SSL/TLS, Linux process management, networking, file permissions, and log analysis.
> **This is NOT a Kubernetes round** — it's about traditional web server and Linux debugging.

---

## MASTER TROUBLESHOOTING FRAMEWORK

**Use this sequence every time. Say it out loud during the interview.**

```
1. SYMPTOM    → What's the user seeing? (blank page, 502, timeout, SSL error)
2. LAYER      → Which layer is broken? (DNS → Network → Webserver → App → Database)
3. VERIFY     → Check each layer bottom-up
4. DIAGNOSE   → Read logs, check configs, test connections
5. FIX        → Apply minimal fix
6. VERIFY     → Confirm end-to-end working
```

### Layer-by-Layer Debugging (Bottom → Up)

```
┌─────────────────────────────────────────┐
│  USER BROWSER                           │
│  "Cannot reach site" / "502" / "Slow"   │
└──────────────┬──────────────────────────┘
               ▼
┌──────────────────────────┐
│  1. DNS RESOLUTION       │  dig, nslookup, host
│     Is domain resolving? │
└──────────────┬───────────┘
               ▼
┌──────────────────────────┐
│  2. NETWORK / FIREWALL   │  ping, telnet, curl, ss, iptables
│     Can we reach the IP? │
└──────────────┬───────────┘
               ▼
┌──────────────────────────┐
│  3. WEBSERVER (Nginx/    │  systemctl, nginx -t, config files
│     Apache) RUNNING?     │
└──────────────┬───────────┘
               ▼
┌──────────────────────────┐
│  4. SSL/TLS CERTIFICATE  │  openssl s_client, certbot
│     Valid & not expired?  │
└──────────────┬───────────┘
               ▼
┌──────────────────────────┐
│  5. APPLICATION SERVER   │  process running? correct port?
│     (Node/Python/Go)     │  logs? crash? OOM?
└──────────────┬───────────┘
               ▼
┌──────────────────────────┐
│  6. DATABASE / BACKEND   │  connection string, auth, running?
│     Can app reach DB?    │
└──────────────┘
```

---

## SCENARIO 1: Website Shows "502 Bad Gateway"

**What 502 means**: The reverse proxy (Nginx) received an invalid/no response from the upstream application server.

### Step-by-Step Diagnosis

```bash
# STEP 1: Is Nginx running?
sudo systemctl status nginx
# If "inactive (dead)" → start it
sudo systemctl start nginx

# STEP 2: Check Nginx error log
sudo tail -50 /var/log/nginx/error.log
# Look for: "connect() failed (111: Connection refused)"
# This means → upstream app is NOT running

# STEP 3: Check Nginx config — what's the upstream?
sudo cat /etc/nginx/sites-enabled/default
# or
sudo cat /etc/nginx/conf.d/app.conf
```

**Typical Nginx reverse proxy config:**
```nginx
server {
    listen 80;
    server_name rea-property.com;

    location / {
        proxy_pass http://127.0.0.1:3000;  # ← App should be on port 3000
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# STEP 4: Is the app actually running on port 3000?
sudo ss -tlnp | grep 3000
# If empty → app is NOT running

# STEP 5: Start the application
# Node.js app:
cd /var/www/app
node server.js &
# or with PM2:
pm2 start server.js
pm2 status

# Python app:
cd /var/www/app
python3 app.py &
# or with gunicorn:
gunicorn --bind 127.0.0.1:3000 app:app

# Go app:
cd /var/www/app
./server &

# STEP 6: Check if the app started successfully
curl -v http://127.0.0.1:3000/
# If 200 OK → app is fine, Nginx should now proxy correctly

# STEP 7: Verify from outside
curl -v http://rea-property.com/
```

### All Common 502 Causes & Fixes

| Cause | How to Identify | Fix |
|---|---|---|
| App not running | `ss -tlnp` shows no listener on upstream port | Start app with PM2/systemd/gunicorn |
| App crashed | Check app logs in `/var/log/app/` or `journalctl -u app` | Fix crash cause, restart |
| Wrong port in Nginx | `proxy_pass` port doesn't match app's listen port | Edit Nginx config, `nginx -s reload` |
| Upstream timeout | Nginx error log: "upstream timed out" | Increase `proxy_read_timeout` or fix slow app |
| Socket permission | App uses Unix socket but Nginx can't access | Fix socket file permissions (`chmod 666 /tmp/app.sock`) |
| App OOM killed | `dmesg | grep -i oom` or `journalctl -k | grep oom` | Increase memory or fix memory leak |
| App listening on 127.0.0.1 only | `ss -tlnp` shows `127.0.0.1:3000` but proxy_pass uses `localhost` | Ensure consistency: both use 127.0.0.1 |

---

## SCENARIO 2: Website Shows "403 Forbidden"

### Step-by-Step Diagnosis

```bash
# STEP 1: Check Nginx error log (THIS TELLS YOU THE EXACT ISSUE)
sudo tail -20 /var/log/nginx/error.log

# Common error messages:
# "directory index of '/var/www/html/' is forbidden"  → no index file
# "open() '/var/www/html/index.html' failed (13: Permission denied)"  → file permissions
# "client denied by server configuration"  → access restriction in config

# STEP 2: Check file permissions
ls -la /var/www/html/
# Files should be readable by nginx user (www-data on Ubuntu, nginx on RHEL)
# Expected: -rw-r--r-- (644 for files, 755 for directories)

# Who runs Nginx?
grep "user" /etc/nginx/nginx.conf
# Typically: user www-data;  or  user nginx;
```

**FIX: Permission denied**
```bash
sudo chown -R www-data:www-data /var/www/html/
sudo find /var/www/html/ -type d -exec chmod 755 {} \;  # Directories: 755
sudo find /var/www/html/ -type f -exec chmod 644 {} \;  # Files: 644
```

**FIX: Missing index file**
```bash
ls /var/www/html/index.html
# If missing → create one or check if index filename is different
```
```nginx
# In Nginx config — add/fix index directive:
server {
    listen 80;
    root /var/www/html;
    index index.html index.htm index.php;   # ← Must list index files

    location / {
        try_files $uri $uri/ =404;  # ← Try file, then directory, then 404
    }
}
```

**FIX: SELinux blocking (RHEL/CentOS)**
```bash
# Check if SELinux is enforcing
getenforce
# If "Enforcing":

# Option 1: Allow Nginx to read web content
sudo setsebool -P httpd_read_user_content 1
# Option 2: Allow Nginx to connect to network (for proxy_pass)
sudo setsebool -P httpd_can_network_connect 1
# Option 3: Relabel web directory
sudo chcon -R -t httpd_sys_content_t /var/www/html/
# Option 4: Temporary (for testing only)
sudo setenforce 0
```

```bash
# STEP 3: Reload Nginx after config changes
sudo nginx -t          # ALWAYS test config syntax first!
sudo nginx -s reload   # Reload without downtime
```

---

## SCENARIO 3: Website Shows "Connection Timed Out" (Can't Reach Server At All)

### Layer 1: DNS

```bash
# Does the domain resolve to an IP?
dig rea-property.com A +short
nslookup rea-property.com
host rea-property.com

# If "NXDOMAIN" → domain not found in DNS
# Fix: Add A record in DNS provider (Route 53, Cloudflare, GoDaddy)

# Check with different DNS servers:
dig @8.8.8.8 rea-property.com A     # Google DNS
dig @1.1.1.1 rea-property.com A     # Cloudflare DNS

# If works with 8.8.8.8 but not locally → local DNS cache issue
# Flush DNS cache:
sudo systemd-resolve --flush-caches   # systemd-resolved
```

### Layer 2: Network Connectivity

```bash
# Can we reach the server IP?
ping -c 3 <server-ip>
# If "Destination Host Unreachable" → routing issue
# If "100% packet loss" → firewall blocking ICMP (may still work on TCP)

# Can we reach port 80/443?
telnet <server-ip> 80
# Or:
nc -zv <server-ip> 80
curl -v --connect-timeout 5 http://<server-ip>/
```

### Layer 3: Firewall

```bash
# ========= iptables =========
sudo iptables -L -n -v | grep -E "80|443|http|https"
# If no ACCEPT rule:
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 443 -j ACCEPT
# Save rules (or they're lost on reboot):
sudo iptables-save > /etc/iptables.rules

# ========= ufw (Ubuntu) =========
sudo ufw status verbose
# If Status: active but no 80/443 rule:
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
# Or:
sudo ufw allow 'Nginx Full'

# ========= firewalld (RHEL/CentOS) =========
sudo firewall-cmd --list-all
# If http/https not listed:
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# ========= AWS Security Group =========
# Check in AWS Console → EC2 → Security Groups → Inbound Rules
# Must have: Type=HTTP, Port=80, Source=0.0.0.0/0
# Must have: Type=HTTPS, Port=443, Source=0.0.0.0/0
```

### Layer 4: Webserver Listening

```bash
# Is Nginx actually listening on port 80?
sudo ss -tlnp | grep -E ":80|:443"
# If nothing:
sudo systemctl start nginx
sudo systemctl enable nginx   # Enable on boot
```

---

## SCENARIO 4: SSL/TLS Certificate Errors

**Common browser errors**: NET::ERR_CERT_DATE_INVALID, ERR_CERT_COMMON_NAME_INVALID, ERR_CERT_AUTHORITY_INVALID

### Diagnosis

```bash
# STEP 1: Check certificate details
openssl s_client -connect rea-property.com:443 -servername rea-property.com 2>/dev/null \
    | openssl x509 -noout -dates -subject -issuer -ext subjectAltName

# Key output to check:
# notAfter=Apr  1 00:00:00 2024 GMT     ← Is it expired?
# subject=CN = rea-property.com         ← Does it match domain?
# issuer=O = Let's Encrypt              ← Is it from a trusted CA?
# X509v3 Subject Alternative Name:
#     DNS:rea-property.com, DNS:www.rea-property.com

# STEP 2: Check certificate chain completeness
openssl s_client -connect rea-property.com:443 -servername rea-property.com 2>/dev/null
# Look at "Verify return code:"
# 0 = OK
# 10 = certificate has expired
# 20 = unable to get local issuer certificate (missing intermediate)
# 21 = unable to verify the first certificate (missing chain)

# STEP 3: Check what Nginx is serving
cat /etc/nginx/sites-enabled/default | grep -A5 ssl
```

### SSL Configuration Reference

```nginx
server {
    listen 80;
    server_name rea-property.com www.rea-property.com;
    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rea-property.com www.rea-property.com;

    ssl_certificate     /etc/letsencrypt/live/rea-property.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/rea-property.com/privkey.pem;

    # Modern SSL settings (Mozilla recommended)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Common SSL Fixes

```bash
# FIX 1: Certificate expired → Renew with Let's Encrypt
sudo certbot renew --dry-run     # Test first
sudo certbot renew               # Actually renew
sudo nginx -s reload

# FIX 2: Missing intermediate certificate (chain incomplete)
# Combine your cert + intermediate:
cat domain.crt intermediate.crt > fullchain.crt
# Update Nginx:
# ssl_certificate /path/to/fullchain.crt;

# FIX 3: Certificate doesn't match domain
# Check SANs:
openssl x509 -in /etc/ssl/certs/cert.crt -noout -text | grep -A1 "Subject Alternative Name"
# If domain not listed → get new cert with correct domain:
sudo certbot certonly --nginx -d rea-property.com -d www.rea-property.com

# FIX 4: Certificate file permissions
ls -la /etc/letsencrypt/live/rea-property.com/
# Key file should be readable only by root:
sudo chmod 600 /etc/letsencrypt/live/rea-property.com/privkey.pem

# FIX 5: Self-signed certificate in production
# Browser shows "Not trusted" → need a real CA cert
# Use Let's Encrypt (free):
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d rea-property.com

# STEP 4: Reload after ALL changes
sudo nginx -t
sudo nginx -s reload
```

---

## SCENARIO 5: Website is Extremely Slow

### System Resource Investigation

```bash
# =================== CPU ===================
top -c                           # Press 'P' to sort by CPU
# Look at: %CPU, load average (top-right)
# load average > number of CPU cores = overloaded
nproc                            # How many CPU cores?

# =================== MEMORY ===================
free -h
# If "available" is very low and "Swap used" is high → memory pressure
# Server is swapping to disk = very slow

# What's eating memory?
ps aux --sort=-%mem | head -10

# =================== DISK ===================
df -h
# If /var or / is 100% → DISK FULL (very common cause of slowness/crashes)
# Find large files:
du -sh /var/log/*
du -sh /var/www/*

# Quick disk cleanup:
sudo journalctl --vacuum-size=100M     # Clean old system logs
sudo find /var/log -name "*.gz" -delete  # Remove rotated logs
sudo apt autoremove                      # Remove old packages

# =================== DISK I/O ===================
iostat -x 1 3
# Look at: %util column. If >90% → disk I/O bottleneck
# Look at: await (ms per request). High = slow disk
```

### Application-Level Investigation

```bash
# STEP 1: How many connections is the app handling?
sudo ss -tnp | grep :3000 | wc -l

# STEP 2: Check Nginx access log for traffic patterns
sudo tail -200 /var/log/nginx/access.log

# Top IPs (potential DDoS):
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Slow requests (response time in Nginx — requires custom log format):
# If using $request_time in log format:
awk '$NF > 5' /var/log/nginx/access.log   # Requests taking >5 seconds

# STEP 3: Check database for slow queries
# PostgreSQL:
sudo -u postgres psql -c "
SELECT pid, now() - pg_stat_activity.query_start AS duration,
       state, query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - pg_stat_activity.query_start > interval '5 seconds'
ORDER BY duration DESC;"

# MySQL:
mysql -e "SHOW PROCESSLIST;" | grep -v Sleep
mysql -e "SHOW GLOBAL STATUS LIKE 'Slow_queries';"

# STEP 4: Is the app leaking memory? (Node.js)
# Check RSS growth over time:
watch -n 5 'ps aux | grep node'
# If RSS keeps growing → memory leak
```

### Nginx Performance Tuning

```nginx
# /etc/nginx/nginx.conf — Key Performance Settings

# Match worker processes to CPU cores
worker_processes auto;

events {
    worker_connections 2048;        # Connections per worker
    use epoll;                       # Linux-optimized event model
    multi_accept on;                 # Accept multiple connections at once
}

http {
    # Connection optimization
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 100;

    # Compression (huge impact on page speed)
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript
               text/xml application/xml application/xml+rss text/javascript
               image/svg+xml;

    # Static file caching
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Rate limiting (prevents DDoS/abuse)
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://127.0.0.1:3000;
    }

    # Buffer tuning for upstream
    proxy_buffer_size 16k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 64k;
}
```

---

## SCENARIO 6: Website Shows "500 Internal Server Error"

**500 = Application code crashed or threw an unhandled exception.**
The answer is ALWAYS in the application logs, not Nginx logs.

### Step-by-Step

```bash
# STEP 1: Check APPLICATION logs (this is WHERE the answer is)
# ─── Node.js ───
pm2 logs                              # If using PM2
pm2 logs --err --lines 50             # Error logs only
journalctl -u nodeapp -n 100          # If using systemd
cat /var/log/app/error.log             # If custom log path

# ─── Python (Gunicorn/Django/Flask) ───
journalctl -u gunicorn -n 100
cat /var/log/gunicorn/error.log
cat /var/www/app/logs/django.log

# ─── Go ───
journalctl -u goapp -n 100

# STEP 2: Run app manually to see the error directly
cd /var/www/app
# Node.js:
node server.js 2>&1 | head -50
# Python:
python3 app.py 2>&1 | head -50
# Go:
./server 2>&1 | head -50
# The error will print to stdout/stderr — READ IT CAREFULLY

# STEP 3: Common 500 causes
```

### Common 500 Error Causes & Fixes

**A) Missing environment variable**
```bash
# Find what env vars the app expects:
grep -r "process.env\|os.environ\|os.Getenv" /var/www/app/ --include="*.js" --include="*.py" --include="*.go"

# Check what's currently set:
printenv | sort

# Fix: Set missing variables
export DB_HOST=localhost
export DB_PASSWORD=secret123
export API_KEY=abc123

# For systemd services, add to service file:
# Environment=DB_HOST=localhost
# Or: EnvironmentFile=/var/www/app/.env
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

**B) Database connection failed**
```bash
# Test PostgreSQL:
psql -h localhost -U appuser -d appdb -c "SELECT 1;"
# Common errors:
# "connection refused" → PostgreSQL not running: sudo systemctl start postgresql
# "password authentication failed" → wrong credentials in app config
# "database 'appdb' does not exist" → create it: createdb appdb

# Test MySQL:
mysql -h localhost -u appuser -p -e "SELECT 1;"

# Check if database service is running:
sudo systemctl status postgresql
sudo systemctl status mysql
```

**C) Missing dependencies / packages**
```bash
# Node.js:
cd /var/www/app
npm install     # Install missing packages

# Python:
cd /var/www/app
pip3 install -r requirements.txt

# Go:
cd /var/www/app
go mod download
```

**D) File/directory not found**
```bash
# App expects a config file or template:
ls -la /var/www/app/config/
ls -la /var/www/app/templates/
ls -la /var/www/app/public/
# If missing → check if files were deployed properly
```

**E) Syntax error in config file (JSON, YAML)**
```bash
# JSON syntax check:
python3 -m json.tool /var/www/app/config.json
# If "Expecting value" → broken JSON

# YAML syntax check:
python3 -c "import yaml; yaml.safe_load(open('/var/www/app/config.yaml'))"
```

```bash
# STEP 4: After fixing → restart properly
sudo systemctl restart myapp
# Or:
pm2 restart all
# Verify:
curl -v http://localhost:3000/
```

---

## SCENARIO 7: Apache Web Server Debugging

### Basic Apache Commands

```bash
# Status
sudo systemctl status apache2    # Debian/Ubuntu
sudo systemctl status httpd      # RHEL/CentOS

# Test config syntax
sudo apachectl configtest
# Or:
sudo apache2ctl -t

# Restart / Reload
sudo systemctl restart apache2
sudo systemctl reload apache2    # Graceful reload

# Check error log
sudo tail -50 /var/log/apache2/error.log       # Ubuntu
sudo tail -50 /var/log/httpd/error_log          # RHEL

# Check access log
sudo tail -50 /var/log/apache2/access.log
```

### Apache Site Management

```bash
# List enabled sites
ls /etc/apache2/sites-enabled/

# Enable a site
sudo a2ensite mysite.conf
# Disable a site
sudo a2dissite 000-default.conf

# List enabled modules
apache2ctl -M

# Enable required modules
sudo a2enmod rewrite         # URL rewriting
sudo a2enmod proxy           # Reverse proxy
sudo a2enmod proxy_http      # HTTP proxy support
sudo a2enmod ssl             # SSL/TLS
sudo a2enmod headers         # Custom headers
sudo a2enmod expires         # Cache headers

# After enabling modules:
sudo systemctl restart apache2
```

### Apache VirtualHost Configuration

```apache
# /etc/apache2/sites-available/app.conf

# HTTP → HTTPS redirect
<VirtualHost *:80>
    ServerName rea-property.com
    ServerAlias www.rea-property.com
    Redirect permanent / https://rea-property.com/
</VirtualHost>

# HTTPS with reverse proxy
<VirtualHost *:443>
    ServerName rea-property.com
    ServerAlias www.rea-property.com

    SSLEngine on
    SSLCertificateFile    /etc/letsencrypt/live/rea-property.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/rea-property.com/privkey.pem

    # Reverse proxy to app
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    # Security headers
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"

    ErrorLog ${APACHE_LOG_DIR}/app-error.log
    CustomLog ${APACHE_LOG_DIR}/app-access.log combined
</VirtualHost>
```

### Apache-Specific Issues

| Issue | Error Message | Fix |
|---|---|---|
| Module not loaded | "Invalid command 'ProxyPass'" | `sudo a2enmod proxy proxy_http` |
| Port conflict | "could not bind to address 0.0.0.0:80" | `sudo ss -tlnp \| grep :80` → kill conflicting process |
| .htaccess not working | Rewrite rules ignored | `AllowOverride All` in VirtualHost |
| DocumentRoot wrong | 404 for everything | Check `DocumentRoot` path exists |

---

## SCENARIO 8: Process Management with systemd

### Understanding systemd Service Files

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=Property Search Web Application
After=network.target postgresql.service   # Start after network & DB
Wants=postgresql.service                   # Soft dependency on DB

[Service]
Type=simple
User=www-data                              # Run as this user (not root!)
Group=www-data
WorkingDirectory=/var/www/app

# ─── Node.js ───
ExecStart=/usr/bin/node /var/www/app/server.js

# ─── Python (Gunicorn) ───
# ExecStart=/usr/bin/gunicorn --bind 127.0.0.1:3000 --workers 4 app:app

# ─── Go ───
# ExecStart=/var/www/app/server

Restart=always                             # Restart if it crashes
RestartSec=5                               # Wait 5s before restarting
StartLimitBurst=5                          # Max 5 restarts in...
StartLimitIntervalSec=60                   # ...60 seconds

# Environment variables
Environment=NODE_ENV=production
Environment=PORT=3000
Environment=DB_HOST=localhost
Environment=DB_NAME=properties
# Or load from file:
EnvironmentFile=/var/www/app/.env

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

### systemd Commands Cheat Sheet

```bash
# Service lifecycle
sudo systemctl start myapp        # Start
sudo systemctl stop myapp         # Stop
sudo systemctl restart myapp      # Restart
sudo systemctl reload myapp       # Reload config (if supported)
sudo systemctl enable myapp       # Start on boot
sudo systemctl disable myapp      # Don't start on boot

# Diagnostics
sudo systemctl status myapp       # Quick status + last few log lines
sudo systemctl is-active myapp    # Just "active" or "inactive"
sudo systemctl is-failed myapp    # Check if failed

# After editing service file:
sudo systemctl daemon-reload      # REQUIRED after changing .service files!
sudo systemctl restart myapp

# Logs via journalctl
journalctl -u myapp -n 50             # Last 50 lines
journalctl -u myapp -f                # Follow live (like tail -f)
journalctl -u myapp --since "10 min ago"
journalctl -u myapp --since today
journalctl -u myapp -p err            # Only errors
journalctl -u myapp --no-pager        # Don't paginate

# System-wide logs
journalctl -xe                         # Recent logs with explanations
journalctl -k                          # Kernel messages (OOM kills here)
journalctl -k | grep -i oom           # Check for OOM kills
```

### Debugging Service Failures

```bash
# Service won't start? Check these:
sudo systemctl status myapp
# Look at "Active: failed" and the reason

# Common reasons:
# "code=exited, status=1" → App crashed, check logs
journalctl -u myapp -n 50

# "code=exited, status=203/EXEC" → ExecStart binary not found
which node   # Is the binary path correct?
ls -la /var/www/app/server.js  # Does the script exist?

# "code=exited, status=217/USER" → User doesn't exist
id www-data   # Does the user exist?

# "code=exited, status=200/CHDIR" → WorkingDirectory doesn't exist
ls -la /var/www/app/

# "start-limit-hit" → Too many restarts
sudo systemctl reset-failed myapp
sudo systemctl start myapp
```

---

## SCENARIO 9: DNS Troubleshooting Deep-Dive

### DNS Diagnosis Commands

```bash
# ============= Basic Resolution =============
dig rea-property.com A +short          # A record → IP
dig rea-property.com AAAA +short       # IPv6
dig rea-property.com CNAME +short      # CNAME
dig rea-property.com MX +short         # Mail server
dig rea-property.com NS +short         # Nameservers
dig rea-property.com TXT +short        # TXT records (SPF, DKIM)

nslookup rea-property.com             # Simpler alternative
host rea-property.com                  # Simplest

# ============= Check Specific DNS Server =============
dig @8.8.8.8 rea-property.com A       # Google DNS
dig @1.1.1.1 rea-property.com A       # Cloudflare DNS
dig @ns1.example.com rea-property.com A  # Authoritative nameserver

# ============= DNS Propagation =============
dig rea-property.com A +trace          # Full delegation trace
# Shows: root → TLD → authoritative nameserver → answer

# ============= Reverse DNS =============
dig -x 1.2.3.4                         # PTR record (IP → domain)
```

### Common DNS Issues & Fixes

| Issue | Symptom | Fix |
|---|---|---|
| No A record | `NXDOMAIN` | Add A record in DNS provider pointing to server IP |
| Wrong IP in A record | Resolves but can't connect | Update A record to correct IP |
| DNS not propagated | Works from some locations, not others | Wait (TTL), or reduce TTL before changes |
| www not working | `rea-property.com` works, `www.` doesn't | Add CNAME: www → rea-property.com |
| Local override | Server resolves differently | Check `/etc/hosts` for wrong entries |
| Resolver broken | Nothing resolves from the server | Check `/etc/resolv.conf` has valid nameservers |

```bash
# Check local DNS overrides
cat /etc/hosts
# If there's a wrong entry like:
# 10.0.0.99  rea-property.com
# → Remove or fix it

# Check DNS resolver configuration
cat /etc/resolv.conf
# Should have:
# nameserver 8.8.8.8
# nameserver 8.8.4.4
# Or your cloud provider's DNS
```

---

## SCENARIO 10: Port Conflicts & "Address Already In Use"

```bash
# Error when starting app:
# "Error: listen EADDRINUSE: address already in use :::3000"
# "OSError: [Errno 98] Address already in use"

# STEP 1: Find what's using the port
sudo ss -tlnp | grep :3000
# Output: LISTEN  0  128  *:3000  *:*  users:(("node",pid=1234,fd=3))

# Alternative:
sudo lsof -i :3000
# Output: node  1234  www-data  3u  IPv4  12345  TCP *:3000 (LISTEN)

# STEP 2: Decide what to do
# Option A: Kill the existing process (if it's stale/old)
sudo kill -15 1234       # SIGTERM (graceful stop)
sleep 2
sudo kill -9 1234        # SIGKILL (force, if SIGTERM didn't work)

# Option B: Kill by name
sudo pkill -f "node server.js"

# Option C: Change YOUR app's port
# In .env or config: PORT=3001
# Then update Nginx proxy_pass to 3001

# STEP 3: Verify port is free
sudo ss -tlnp | grep :3000
# Should show nothing

# STEP 4: Start your app
cd /var/www/app && node server.js
```

---

## SCENARIO 11: Disk Full (Very Common, Easy to Miss)

```bash
# Check disk usage
df -h
# If any filesystem is at 100% → THINGS WILL BREAK
# /var → logs fill up
# / → system partition full
# /tmp → temp files accumulated

# Find what's using space
du -sh /var/log/*          # Check log sizes
du -sh /var/www/*
du -sh /tmp/*
du -sh /* 2>/dev/null | sort -hr | head -10   # Top 10 largest directories

# Common culprits:
# 1. Log files that were never rotated
ls -lhS /var/log/nginx/*.log
ls -lhS /var/log/syslog*

# 2. Old log archives
find /var/log -name "*.gz" -mtime +30 -delete  # Delete rotated logs >30 days

# 3. Docker (if used)
docker system prune -a -f    # Remove unused images, containers, volumes

# 4. Journal logs
journalctl --disk-usage
sudo journalctl --vacuum-size=100M

# 5. Package cache
sudo apt clean               # Debian/Ubuntu
sudo yum clean all            # RHEL/CentOS

# After freeing space, apps may need restart
sudo systemctl restart nginx
sudo systemctl restart myapp
```

### Set Up Log Rotation (Prevent Recurrence)

```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 640 www-data www-data
    postrotate
        systemctl reload nginx > /dev/null 2>&1 || true
    endscript
}
```

---

## SCENARIO 12: Application Behind Reverse Proxy — Headers & CORS

```bash
# Problem: App works directly (curl localhost:3000) but breaks through Nginx
# Often: redirects to http://, wrong domain, CORS errors

# Check if proxy headers are being passed:
```

```nginx
# CORRECT Nginx proxy config with all necessary headers:
location / {
    proxy_pass http://127.0.0.1:3000;

    # These headers are CRITICAL:
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;   # ← Tells app if HTTPS
    proxy_set_header X-Forwarded-Host $host;

    # WebSocket support (if needed):
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    # Timeouts (for slow APIs):
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

```bash
# CORS error? Add CORS headers in Nginx:
```
```nginx
location /api/ {
    # CORS headers
    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type' always;

    # Handle preflight OPTIONS requests
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type';
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
    }

    proxy_pass http://127.0.0.1:3000;
}
```

---

## ESSENTIAL LINUX COMMANDS CHEAT SHEET

### File & Config Operations
```bash
cat /etc/nginx/nginx.conf           # Full file
head -20 /etc/nginx/nginx.conf      # First 20 lines
tail -50 /var/log/nginx/error.log   # Last 50 lines
tail -f /var/log/nginx/error.log    # Follow live (Ctrl+C to stop)
less /var/log/nginx/error.log       # Paginate (q to quit)

# Search in files
grep -r "error" /var/log/nginx/         # Recursive
grep -i "connection refused" /var/log/  # Case-insensitive
grep -n "proxy_pass" /etc/nginx/conf.d/ # With line numbers
grep -c "502" /var/log/nginx/access.log # Count matches

# Edit files
sudo nano /etc/nginx/sites-enabled/default   # Easy editor
sudo vi /etc/nginx/sites-enabled/default     # Vi editor
# Vi: i=insert, Esc=command, :wq=save, :q!=quit

# Find files
find / -name "nginx.conf" 2>/dev/null
find /var/www -name "*.js" -mmin -30    # Modified in last 30 min
find /var/log -size +100M               # Files larger than 100MB
which node                               # Where is binary?
```

### Network Diagnostics
```bash
# Listening ports
sudo ss -tlnp                        # All TCP listeners with PID
sudo ss -ulnp                        # UDP listeners
sudo netstat -tlnp                   # Alternative (older)

# Test HTTP
curl -v http://localhost:3000/       # Verbose
curl -I http://rea-property.com/    # Headers only
curl -k https://rea-property.com/  # Skip SSL verification
curl -o /dev/null -s -w "HTTP %{http_code} | DNS: %{time_namelookup}s | Connect: %{time_connect}s | TLS: %{time_appconnect}s | Total: %{time_total}s\n" https://rea-property.com/

# Connections
ss -s                                # Socket summary
ss -tn state established | wc -l    # Count established
ss -tn state time-wait | wc -l      # Count TIME_WAIT (high = problem)

# Routing
ip route show
ip addr show                         # All network interfaces
traceroute rea-property.com
```

### Process Management
```bash
ps aux | grep nginx                  # Find process
ps aux | grep -E "node|python|go"   # Find app processes
pgrep -la nginx                      # PID + command

kill -15 <PID>          # SIGTERM (graceful)
kill -9 <PID>           # SIGKILL (force)
pkill -f "node server"  # Kill by command pattern

top -c                   # CPU/memory overview
htop                     # Interactive version
free -h                  # Memory usage
df -h                    # Disk space
uptime                   # Load average + uptime
```

### Log Analysis One-Liners
```bash
# Count HTTP status codes
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn
# Output:  45000 200
#           2500 304
#            150 502
#             30 500

# Top 10 IPs
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Top 10 URLs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# 5xx errors in last 5 minutes
awk -v d="$(date -d '5 minutes ago' '+%d/%b/%Y:%H:%M')" \
    '$4 >= "["d && $9 ~ /^5/' /var/log/nginx/access.log

# Requests per minute
awk '{print $4}' /var/log/nginx/access.log | cut -d: -f1-3 | sort | uniq -c | tail -20

# Errors by type
grep -oP 'upstream [\w ]+' /var/log/nginx/error.log | sort | uniq -c | sort -rn
```

---

## QUICK REFERENCE — SYMPTOM → ACTION

| Symptom | Check 1 | Check 2 | Check 3 |
|---|---|---|---|
| **Can't reach site** | DNS (`dig domain`) | Firewall (`iptables -L`) | Webserver running? (`systemctl status nginx`) |
| **502 Bad Gateway** | App running? (`ss -tlnp`) | Correct port in Nginx? | App logs for crash |
| **403 Forbidden** | File permissions (`ls -la`) | Index file exists? | SELinux (`getenforce`) |
| **500 Internal Error** | Application logs | Missing env vars | Database connection |
| **SSL / Not Secure** | Cert expired? (`openssl`) | Domain match? | Chain complete? |
| **Very slow** | CPU/Memory (`top`, `free -h`) | Disk full? (`df -h`) | Too many connections? (`ss`) |
| **Connection refused** | Port open? (`ss -tlnp`) | App crashed? | Listening on 127.0.0.1 vs 0.0.0.0 |
| **404 Not Found** | Root directory correct? | `try_files` directive | URL rewrite rules |
| **Redirect loop** | HTTP→HTTPS config | `return 301` loop | App-level redirects |
| **CORS error** | CORS headers in Nginx | OPTIONS handling | `Access-Control-Allow-Origin` |

---

## MOCK EXERCISE — Practice on a Linux VM

### Setup (On Ubuntu/EC2)

```bash
# 1. Install Nginx
sudo apt update && sudo apt install -y nginx

# 2. Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# 3. Create a simple app
sudo mkdir -p /var/www/app
cat << 'EOF' | sudo tee /var/www/app/server.js
const http = require('http');
const server = http.createServer((req, res) => {
    if (req.url === '/healthz') {
        res.writeHead(200, {'Content-Type': 'application/json'});
        res.end(JSON.stringify({status: 'ok', time: new Date().toISOString()}));
    } else if (req.url === '/') {
        res.writeHead(200, {'Content-Type': 'text/html'});
        res.end('<h1>REA Property Search</h1><p>Find your dream home</p>');
    } else {
        res.writeHead(404, {'Content-Type': 'text/plain'});
        res.end('Not Found');
    }
});
server.listen(3000, '127.0.0.1', () => console.log('App running on port 3000'));
EOF

# 4. Create systemd service
cat << 'EOF' | sudo tee /etc/systemd/system/propertyapp.service
[Unit]
Description=REA Property Search App
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node /var/www/app/server.js
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now propertyapp

# 5. Configure Nginx reverse proxy
cat << 'EOF' | sudo tee /etc/nginx/sites-available/propertyapp
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
EOF

sudo ln -sf /etc/nginx/sites-available/propertyapp /etc/nginx/sites-enabled/default
sudo nginx -t && sudo nginx -s reload

# 6. Verify it works
curl http://localhost/
```

### Break & Fix Exercises

```bash
# Exercise 1: 502 — Stop the app
sudo systemctl stop propertyapp
# Now: curl http://localhost/ → 502
# Fix it!

# Exercise 2: Wrong port — Change proxy_pass
sudo sed -i 's/3000/4000/' /etc/nginx/sites-available/propertyapp
sudo nginx -s reload
# Now: curl http://localhost/ → 502
# Diagnose and fix!

# Exercise 3: 403 — Remove permissions
sudo chmod 000 /var/www/app/server.js
sudo systemctl restart propertyapp
# Now: App won't start
# Diagnose via journalctl and fix!

# Exercise 4: Port conflict — Start another process on 3000
sudo systemctl stop propertyapp
python3 -c "import http.server; http.server.HTTPServer(('127.0.0.1',3000), http.server.SimpleHTTPRequestHandler).serve_forever()" &
sudo systemctl start propertyapp
# Now: propertyapp fails to start — "Address in use"
# Fix it!

# Exercise 5: Disk full — Fill /var/log
sudo dd if=/dev/zero of=/var/log/bigfile bs=1M count=5000  # Create 5GB file
# Things may start failing
# Diagnose with df -h and fix!
# (Clean up: sudo rm /var/log/bigfile)

# Exercise 6: Firewall block
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
# Now: Can't reach from outside
# Fix it!
# (Reset: sudo iptables -D INPUT -p tcp --dport 80 -j DROP)
```

---

## INTERVIEW COMMUNICATION TIPS

1. **Think out loud**: "I see a 502, so the first thing I'll check is whether the upstream application is running..."
2. **Start from the symptom**: Don't jump to random things. Read the error message first.
3. **Check logs first**: `tail -50 /var/log/nginx/error.log` should be one of your first commands
4. **Verify after fixing**: Always `curl` or check the browser after applying a fix
5. **Explain trade-offs**: "I'm restarting the service, which will cause brief downtime. In production, I'd use a graceful reload instead."
6. **Show you know the stack**: "This 502 tells me Nginx is working fine — it's the upstream that's the problem."
7. **Don't panic**: If you don't know a command, try `man <command>` or `<command> --help`
