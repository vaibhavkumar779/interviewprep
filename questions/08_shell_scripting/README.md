> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Deep-dive learning guide (this file) |
| [complete.md](complete.md) | Complete question bank |
| [answers.md](answers.md) | All answers |

---

# Shell Scripting (Bash) — Deep-Dive Learning Guide

---

## 1. Script Basics

```bash
#!/bin/bash
# Shebang line — tells OS which interpreter to use
# #!/usr/bin/env bash   ← more portable (finds bash in PATH)

set -euo pipefail       # ALWAYS USE IN PRODUCTION SCRIPTS!
# -e   Exit on any error (non-zero exit code)
# -u   Error on undefined variables (catches typos!)
# -o pipefail   Pipe fails if ANY command in pipe fails (not just last)

# Without set -e:
#   bad_command      ← silently fails, script continues (DANGEROUS!)
# With set -e:
#   bad_command      ← script stops immediately
```

### Running Scripts

```bash
chmod +x script.sh      # Make executable
./script.sh              # Run (uses shebang)
bash script.sh           # Run with bash explicitly
source script.sh         # Run in CURRENT shell (inherits vars/functions)
. script.sh              # Same as source
```

---

## 2. Variables

```bash
# ─── Assignment (NO spaces around =) ───
name="DevOps"                    # String
count=5                          # Number (actually still a string)
readonly PI=3.14                 # Constant (can't change)
unset name                       # Delete variable

# ─── String operations ───
full="${first}_${last}"          # Interpolation (always use braces!)
length=${#name}                  # String length
upper=${name^^}                  # Uppercase (bash 4+)
lower=${name,,}                  # Lowercase (bash 4+)
sub=${name:0:3}                  # Substring (first 3 chars)
replace=${path/old/new}          # Replace first occurrence
replace_all=${path//old/new}     # Replace all occurrences

# ─── Default values ───
host=${DB_HOST:-localhost}       # Use "localhost" if DB_HOST is unset/empty
host=${DB_HOST:=localhost}       # Same + actually SET the variable
port=${DB_PORT:?ERROR: DB_PORT not set}  # Error if unset (great for required vars)

# ─── Arrays ───
servers=("web1" "web2" "db1")   # Declare array
echo "${servers[0]}"             # First element: web1
echo "${servers[@]}"             # All elements
echo "${#servers[@]}"            # Array length: 3
servers+=("web3")                # Append

for server in "${servers[@]}"; do
    echo "Deploying to $server"
done

# ─── Associative Arrays (bash 4+) ───
declare -A config
config[host]="localhost"
config[port]="8080"
echo "${config[host]}"
```

---

## 3. Control Flow

```bash
# ─── If/Else ───
if [[ -f "/etc/nginx/nginx.conf" ]]; then
    echo "Nginx config exists"
elif [[ -d "/etc/nginx" ]]; then
    echo "Directory exists but no config"
else
    echo "Nginx not installed"
fi

# ─── Test operators ───
# Files:
[[ -f file ]]     # File exists (regular file)
[[ -d dir ]]      # Directory exists
[[ -e path ]]     # Exists (file or dir)
[[ -r file ]]     # Readable
[[ -w file ]]     # Writable
[[ -x file ]]     # Executable
[[ -s file ]]     # File exists and non-empty

# Strings:
[[ -z "$var" ]]   # Empty string (zero length)
[[ -n "$var" ]]   # Non-empty string
[[ "$a" == "$b" ]]  # Equal
[[ "$a" != "$b" ]]  # Not equal
[[ "$a" =~ ^[0-9]+$ ]]  # Regex match (numbers only)

# Numbers:
[[ $a -eq $b ]]   # Equal
[[ $a -ne $b ]]   # Not equal
[[ $a -gt $b ]]   # Greater than
[[ $a -lt $b ]]   # Less than
[[ $a -ge $b ]]   # Greater or equal
[[ $a -le $b ]]   # Less or equal

# Logic:
[[ cond1 && cond2 ]]   # AND
[[ cond1 || cond2 ]]   # OR
[[ ! cond ]]            # NOT

# ─── Case (switch) ───
case "$environment" in
    prod|production)
        replicas=5
        ;;
    staging)
        replicas=2
        ;;
    dev|development)
        replicas=1
        ;;
    *)
        echo "Unknown environment: $environment" >&2
        exit 1
        ;;
esac
```

---

## 4. Loops

```bash
# ─── For loop ───
for server in web1 web2 web3; do
    ssh "$server" "systemctl restart nginx"
done

# ─── C-style for ───
for ((i=1; i<=5; i++)); do
    echo "Attempt $i"
done

# ─── Range ───
for i in {1..10}; do
    echo "Item $i"
done

# ─── Loop over files ───
for file in /var/log/*.log; do
    echo "Processing: $file"
    gzip "$file"
done

# ─── While loop ───
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# ─── Read file line by line ───
while IFS= read -r line; do
    echo "Processing: $line"
done < servers.txt

# ─── While with command output ───
kubectl get pods -o name | while read -r pod; do
    echo "Checking $pod"
    kubectl describe "$pod"
done

# ─── Until loop (run until condition is true) ───
until curl -sf http://localhost:8080/health; do
    echo "Waiting for service..."
    sleep 2
done
echo "Service is up!"

# ─── Break / Continue ───
for server in "${servers[@]}"; do
    if ! ping -c1 "$server" &>/dev/null; then
        echo "SKIP: $server unreachable"
        continue    # Skip to next iteration
    fi
    deploy "$server"
done
```

---

## 5. Functions

```bash
# ─── Function definition ───
deploy_service() {
    local service=$1                    # local scope!
    local env=${2:-staging}             # Default value
    local version=${3:?ERROR: version required}  # Required param

    echo "Deploying $service v$version to $env"

    if ! docker pull "registry/$service:$version"; then
        echo "ERROR: Failed to pull image" >&2
        return 1    # Return non-zero for failure
    fi

    docker tag "registry/$service:$version" "$service:latest"
    echo "Deployed successfully"
    return 0
}

# ─── Call function ───
if deploy_service "web-api" "production" "v2.1.0"; then
    echo "Success!"
else
    echo "Deployment failed!"
    exit 1
fi

# ─── Function returning a value (via stdout) ───
get_pod_count() {
    kubectl get pods -n "$1" --no-headers 2>/dev/null | wc -l
}

count=$(get_pod_count "production")
echo "Running pods: $count"

# ─── Error handler function ───
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/deploy_*.tmp
}
trap cleanup EXIT    # Run cleanup on script exit (success or failure!)
trap 'echo "Interrupted!"; exit 1' INT TERM  # Handle Ctrl+C
```

---

## 6. Input/Output & Redirection

```bash
# ─── File descriptors ───
# 0 = stdin    1 = stdout    2 = stderr

command > file           # stdout to file (overwrite)
command >> file          # stdout to file (append)
command 2> errors.log    # stderr to file
command > out.log 2>&1   # Both stdout+stderr to file
command &> combined.log  # Same as above (bash shortcut)
command > /dev/null 2>&1 # Discard ALL output

# ─── Pipes ───
cat access.log | grep "ERROR" | sort | uniq -c | sort -rn | head -10
# Better: avoid useless cat:
grep "ERROR" access.log | sort | uniq -c | sort -rn | head -10

# ─── Here document ───
cat << EOF > /etc/nginx/conf.d/app.conf
server {
    listen 80;
    server_name app.example.com;
    location / {
        proxy_pass http://localhost:8080;
    }
}
EOF

# ─── Here string ───
grep "error" <<< "$log_output"

# ─── Process substitution ───
diff <(kubectl get pods -n staging) <(kubectl get pods -n production)

# ─── Reading user input ───
read -p "Deploy to production? [y/N] " -r answer
if [[ "$answer" =~ ^[Yy]$ ]]; then
    deploy_prod
fi

read -sp "Enter password: " password  # -s = silent (no echo)
```

---

## 7. Text Processing Tools

### grep — Search text

```bash
grep "error" /var/log/syslog            # Basic search
grep -i "error" log                      # Case insensitive
grep -r "TODO" src/                      # Recursive in directory
grep -c "error" log                      # Count matches
grep -n "error" log                      # Show line numbers
grep -v "debug" log                      # Invert (exclude debug)
grep -l "password" *.conf               # Files containing match
grep -E "error|warning|critical" log    # Extended regex (OR)
grep -A3 -B1 "error" log               # 3 lines After, 1 Before
```

### sed — Stream editor

```bash
sed 's/old/new/' file                    # Replace first per line
sed 's/old/new/g' file                   # Replace ALL per line
sed -i 's/old/new/g' file               # In-place edit (modifies file!)
sed -i.bak 's/old/new/g' file           # In-place with backup
sed '5d' file                            # Delete line 5
sed '/^#/d' file                         # Delete comment lines
sed -n '10,20p' file                     # Print lines 10-20
sed 's/^/PREFIX: /' file                 # Add prefix to every line
```

### awk — Column processing

```bash
awk '{print $1}' file                    # Print first column
awk '{print $1, $3}' file               # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd        # Custom delimiter (:)
awk '$3 > 100 {print $1, $3}' file     # Conditional
awk '{sum += $1} END {print sum}' file  # Sum column
awk 'NR==5' file                         # Line 5 only

# Real example: pods using >500Mi memory
kubectl top pods | awk 'NR>1 && $3 > 500 {print $1, $3 "Mi"}'
```

### cut, sort, uniq, tr, wc

```bash
cut -d: -f1 /etc/passwd                 # First field, : delimiter
sort file                                # Sort alphabetically
sort -n file                             # Sort numerically
sort -rn file                            # Reverse numeric sort
sort -u file                             # Sort + unique (remove dups)
uniq -c                                  # Count occurrences (needs sorted input!)
tr 'a-z' 'A-Z'                          # Lowercase → uppercase
tr -d '\r'                               # Remove carriage returns (Windows → Linux)
wc -l file                               # Line count
wc -w file                               # Word count
```

---

## 8. Special Variables

```bash
$0          # Script name
$1, $2...   # Positional parameters
$#          # Number of arguments
$@          # All arguments (as separate words) — USE THIS
$*          # All arguments (as single string) — rarely needed
$?          # Exit code of last command (0=success)
$$          # Current script's PID
$!          # PID of last background process
$_          # Last argument of previous command
$LINENO     # Current line number in script
$FUNCNAME   # Current function name
```

---

## 9. Exit Codes

```bash
command
echo $?      # 0 = success, non-zero = failure

# ─── Custom exit codes ───
exit 0       # Success
exit 1       # General error
exit 2       # Misuse of command
exit 126     # Permission denied
exit 127     # Command not found
exit 128+N   # Killed by signal N (e.g., 137 = SIGKILL)

# ─── Logical operators ───
command1 && command2    # Run command2 ONLY IF command1 succeeds
command1 || command2    # Run command2 ONLY IF command1 fails
command1 ; command2     # Run both regardless

# ─── Error handling pattern ───
docker build -t myapp . || { echo "Build failed!"; exit 1; }
```

---

## 10. Real DevOps Scripts

### Deployment Script

```bash
#!/bin/bash
set -euo pipefail

# ─── Configuration ───
readonly SERVICE=${1:?Usage: $0 <service> <version> [environment]}
readonly VERSION=${2:?Usage: $0 <service> <version> [environment]}
readonly ENV=${3:-staging}
readonly REGISTRY="myregistry.azurecr.io"
readonly IMAGE="${REGISTRY}/${SERVICE}:${VERSION}"
readonly LOG_FILE="/var/log/deploy/${SERVICE}-$(date +%Y%m%d-%H%M%S).log"

# ─── Logging ───
log() { echo "[$(date +%H:%M:%S)] $*" | tee -a "$LOG_FILE"; }
error() { log "ERROR: $*" >&2; }

# ─── Cleanup trap ───
cleanup() {
    local exit_code=$?
    if [[ $exit_code -ne 0 ]]; then
        error "Deployment failed with exit code $exit_code"
        log "Rolling back..."
        kubectl rollout undo deployment/"$SERVICE" -n "$ENV" || true
    fi
}
trap cleanup EXIT

# ─── Main ───
log "Deploying $SERVICE v$VERSION to $ENV"

log "Pulling image..."
docker pull "$IMAGE"

log "Updating deployment..."
kubectl set image deployment/"$SERVICE" "$SERVICE=$IMAGE" -n "$ENV"

log "Waiting for rollout..."
kubectl rollout status deployment/"$SERVICE" -n "$ENV" --timeout=300s

log "Running health check..."
until curl -sf "https://${SERVICE}.${ENV}.internal/health"; do
    sleep 5
done

log "Deployment complete!"
```

### Log Analysis Script

```bash
#!/bin/bash
set -euo pipefail

LOG_FILE=${1:?Usage: $0 <logfile>}

echo "=== Log Analysis: $LOG_FILE ==="
echo ""
echo "Total lines: $(wc -l < "$LOG_FILE")"
echo ""

echo "─── Status Code Distribution ───"
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "─── Top 10 IPs ───"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "─── Errors (5xx) ───"
awk '$9 >= 500 {print $0}' "$LOG_FILE" | tail -20

echo ""
echo "─── Requests per Hour ───"
awk -F'[/:]' '{print $4}' "$LOG_FILE" | sort | uniq -c
```

### Health Check with Retry

```bash
#!/bin/bash
set -euo pipefail

URL=${1:?Usage: $0 <url> [max_retries] [interval]}
MAX_RETRIES=${2:-10}
INTERVAL=${3:-5}

for ((i=1; i<=MAX_RETRIES; i++)); do
    if curl -sf --max-time 5 "$URL" > /dev/null; then
        echo "✓ Service is healthy (attempt $i)"
        exit 0
    fi
    echo "✗ Attempt $i/$MAX_RETRIES failed. Retrying in ${INTERVAL}s..."
    sleep "$INTERVAL"
done

echo "FAILED: Service not healthy after $MAX_RETRIES attempts"
exit 1
```

---

## 11. Debugging

```bash
bash -x script.sh            # Print every command before execution (trace)
set -x                       # Enable trace mode in script
set +x                       # Disable trace mode

# ─── Debug a specific section ───
set -x
problematic_function
set +x

# ─── Dry run pattern ───
DRY_RUN=${DRY_RUN:-false}
run() {
    if [[ "$DRY_RUN" == "true" ]]; then
        echo "[DRY RUN] $*"
    else
        "$@"
    fi
}

run docker push "$IMAGE"
run kubectl apply -f manifest.yaml
```

---

## 12. Best Practices Summary

```
✅ Always: set -euo pipefail
✅ Always: quote variables "$var" (prevents word splitting)
✅ Always: use [[ ]] not [ ] (bash-specific, safer)
✅ Always: use local for function variables
✅ Always: use trap for cleanup
✅ Always: validate inputs at the start
✅ Always: use readonly for constants
✅ Prefer: ${var:-default} over manual checks
✅ Prefer: $() over backticks `` for command substitution
✅ Prefer: [[ "$var" =~ pattern ]] for regex
✅ Prefer: printf over echo for portability
❌ Never: unquoted variables (word splitting bugs!)
❌ Never: eval with user input (code injection!)
❌ Never: parse ls output (use glob or find instead)
```
