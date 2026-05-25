# Shell Scripting (Bash) - COMPREHENSIVE ANSWERS (All 70 Questions)

---

## Basics

**1. Shebang line?**
```bash
#!/bin/bash
# First line of script. Tells OS which interpreter to use.
#!/usr/bin/env bash    # More portable (finds bash in PATH)
#!/usr/bin/env python3 # For Python scripts
```

**2. Make a script executable?**
```bash
chmod +x script.sh
# Then run: ./script.sh
```

**3. `./script.sh` vs `bash script.sh` vs `source script.sh`?**

```
./script.sh                   bash script.sh              source script.sh
┌──────────────────────┐    ┌──────────────────────┐  ┌──────────────────────┐
│ Parent shell            │    │ Parent shell            │  │ Current shell          │
│  └─ New child process  │    │  └─ New child process  │  │  (runs in THIS shell) │
│     (subprocess)       │    │     (subprocess)       │  │  Variables PERSIST     │
│  Needs chmod +x        │    │  No chmod needed       │  │  cd CHANGES dir        │
│  Uses shebang line     │    │  Ignores shebang       │  │  Used for .env files   │
└──────────────────────┘    └──────────────────────┘  └──────────────────────┘
```

- `./script.sh`: New subprocess. Needs execute permission. Uses shebang interpreter.
- `bash script.sh`: New subprocess. No execute permission needed. Ignores shebang.
- `source script.sh` (or `. script.sh`): Runs in **current** shell. Changes to variables/directory persist. Used for loading env files.

**4. Define variable? Use it?**
```bash
NAME="Vaibhav"           # No spaces around =
AGE=30
echo "Hello $NAME"        # Use with $
echo "Hello ${NAME}!"     # ${} for clarity/concatenation
echo "Path: ${HOME}/bin"
```

**5. Single quotes vs double quotes?**
```bash
NAME="world"
echo "Hello $NAME"    # Hello world   (double: variables expanded)
echo 'Hello $NAME'    # Hello $NAME   (single: literal, no expansion)
echo "Path: $HOME"    # Path: /home/user
echo 'Path: $HOME'    # Path: $HOME
```

**6. Command substitution?**
```bash
DATE=$(date +%Y-%m-%d)         # Modern (preferred)
DATE=`date +%Y-%m-%d`          # Legacy (backticks)
FILES=$(ls -1 | wc -l)
echo "Today is $DATE, $FILES files found"
```

**7. Exit status of last command?**
```bash
ls /tmp
echo $?    # 0 = success
ls /nonexistent
echo $?    # 2 = error (non-zero = failure)
```

**8. Special variables?**

```
Bash Special Variables Reference:
┌─────────┬────────────────────────────────────────────┐
│ $0      │ Script name                                  │
│ $1..$9  │ Positional arguments (1st through 9th)       │
│ $#      │ Number of arguments                          │
│ $@      │ All args as separate words (★ preferred)      │
│ $*      │ All args as single string                    │
│ $$      │ Current process PID                          │
│ $!      │ PID of last background process               │
│ $?      │ Exit status of last command                  │
└─────────┴────────────────────────────────────────────┘

  ./deploy.sh staging us-east-1
  $0 = ./deploy.sh
  $1 = staging
  $2 = us-east-1
  $# = 2
  $@ = staging us-east-1
```

**9. Read user input?**
```bash
read -p "Enter name: " NAME
echo "Hello, $NAME"

read -sp "Enter password: " PASSWORD    # -s = silent (no echo)
echo
read -t 10 -p "Quick! " ANSWER          # -t = timeout
```

**10. Pass arguments to a script?**
```bash
#!/bin/bash
# Usage: ./script.sh arg1 arg2
echo "Script: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Count: $#"
```

---

## Conditionals

**11. if/elif/else/fi syntax?**
```bash
if [ "$AGE" -ge 18 ]; then
    echo "Adult"
elif [ "$AGE" -ge 13 ]; then
    echo "Teenager"
else
    echo "Child"
fi
```

**12. `[ ]` vs `[[ ]]` vs `(( ))`?**
- `[ ]`: POSIX test. Portable. Must quote variables.
- `[[ ]]`: Bash extended test. Pattern matching, regex, no word splitting. **Preferred in bash.**
- `(( ))`: Arithmetic evaluation. No `$` needed for variables.
```bash
[ "$a" = "$b" ]         # POSIX string comparison
[[ $a == $b ]]          # Bash (no quotes needed)
[[ $a =~ ^[0-9]+$ ]]    # Regex match
(( a > b ))             # Arithmetic comparison
```

**13. String comparison operators?**
```bash
[ -z "$str" ]     # True if string is empty (zero length)
[ -n "$str" ]     # True if string is NOT empty
[ "$a" = "$b" ]   # True if strings are equal
[ "$a" != "$b" ]  # True if strings are not equal
[[ "$a" < "$b" ]] # Lexicographic comparison (bash only)
```

**14. Integer comparison operators?**
```bash
[ "$a" -eq "$b" ]   # Equal
[ "$a" -ne "$b" ]   # Not equal
[ "$a" -lt "$b" ]   # Less than
[ "$a" -gt "$b" ]   # Greater than
[ "$a" -le "$b" ]   # Less than or equal
[ "$a" -ge "$b" ]   # Greater than or equal
# OR with (( )):
(( a == b ))
(( a > b ))
```

**15. File test operators?**
```bash
[ -f "$file" ]   # Is a regular file
[ -d "$dir" ]    # Is a directory
[ -e "$path" ]   # Exists (file or dir)
[ -r "$file" ]   # Is readable
[ -w "$file" ]   # Is writable
[ -x "$file" ]   # Is executable
[ -s "$file" ]   # Is non-empty (size > 0)
[ -L "$file" ]   # Is a symlink
```

**16. Check if file exists and is readable?**
```bash
if [ -f "$file" ] && [ -r "$file" ]; then
    echo "File exists and is readable"
fi
# OR
if [[ -f "$file" && -r "$file" ]]; then
    echo "File exists and is readable"
fi
```

**17. Check if string is empty?**
```bash
if [ -z "$STRING" ]; then
    echo "String is empty"
fi

if [ -n "$STRING" ]; then
    echo "String is NOT empty"
fi
```

**18. case statement syntax?**
```bash
case "$option" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

**19. Case statement with 4 options (menu)?**
```bash
#!/bin/bash
echo "1) Check disk usage"
echo "2) Check memory"
echo "3) Check services"
echo "4) Exit"
read -p "Choose option: " choice

case $choice in
    1) df -h ;;
    2) free -h ;;
    3) systemctl list-units --type=service --state=running ;;
    4) exit 0 ;;
    *) echo "Invalid option" ;;
esac
```

**20. AND (&&) and OR (||) in conditions?**
```bash
# AND
[ -f "$file" ] && [ -r "$file" ] && echo "readable file"
[[ -f "$file" && -r "$file" ]]

# OR
[ -f "$file" ] || [ -d "$file" ] || echo "not found"

# Short-circuit
command && echo "success" || echo "failed"
```

---

## Loops

**20b. I/O Redirection & Pipes (Interview Essential):**

```
File Descriptors:
  0 = stdin  (keyboard input)
  1 = stdout (normal output)
  2 = stderr (error output)

Redirection:
  command > file         # stdout → file (overwrite)
  command >> file        # stdout → file (append)
  command 2> file        # stderr → file
  command 2>&1           # stderr → same place as stdout
  command > file 2>&1    # both stdout+stderr → file
  command &> file        # shorthand for above (bash)
  command < file         # file → stdin
  command <<< "string"   # string → stdin (here-string)

Pipe (connects stdout → stdin):
  ┌─────────┐    stdout    ┌─────────┐    stdout    ┌─────────┐
  │ cmd1    │────────────▶│ cmd2    │────────────▶│ cmd3    │
  │ (grep)  │  → stdin     │ (sort)  │  → stdin     │ (uniq)  │
  └─────────┘              └─────────┘              └─────────┘

  cat access.log | grep ERROR | awk '{print $1}' | sort | uniq -c | sort -rn
       │              │             │                │         │
       │              │             │                │         └── sort numerically
       │              │             │                └── count duplicates
       │              │             └── extract first field
       │              └── filter error lines
       └── read file

Process substitution:
  diff <(sort file1) <(sort file2)   # Compare sorted versions
```

**21. For loop over values?**
```bash
for fruit in apple banana cherry; do
    echo "Fruit: $fruit"
done

for num in {1..10}; do
    echo $num
done

for server in $(cat servers.txt); do
    ping -c 1 $server
done
```

**22. For loop over files in directory?**
```bash
for file in /var/log/*.log; do
    echo "Processing: $file"
    wc -l "$file"
done

for file in $(find . -name "*.yaml"); do
    echo "Validating: $file"
done
```

**23. C-style for loop?**
```bash
for ((i=0; i<10; i++)); do
    echo "Iteration: $i"
done

for ((i=1; i<=100; i+=10)); do
    echo $i
done
```

**24. While loop reading file line by line?**
```bash
while IFS= read -r line; do
    echo "Line: $line"
done < "file.txt"

# Process CSV
while IFS=, read -r name age city; do
    echo "$name is $age years old from $city"
done < "data.csv"
```

**25. While loop until service is healthy?**
```bash
MAX_RETRIES=30
RETRY=0
until curl -sf http://localhost:8080/health > /dev/null; do
    RETRY=$((RETRY + 1))
    if [ $RETRY -ge $MAX_RETRIES ]; then
        echo "Service failed to start after $MAX_RETRIES attempts"
        exit 1
    fi
    echo "Waiting for service... ($RETRY/$MAX_RETRIES)"
    sleep 2
done
echo "Service is healthy!"
```

**26. Until loop?**
```bash
until [ "$status" = "ready" ]; do
    status=$(check_status)
    sleep 1
done
# Opposite of while: runs UNTIL condition is true
```

**27. Break and continue?**
```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue    # Skip iteration 5
    fi
    if [ $i -eq 8 ]; then
        break       # Exit loop at 8
    fi
    echo $i
done
# Output: 1 2 3 4 6 7
```

**28. Loop processing CLI arguments?**
```bash
while [[ $# -gt 0 ]]; do
    case $1 in
        --name) NAME="$2"; shift 2 ;;
        --env)  ENV="$2"; shift 2 ;;
        --dry-run) DRY_RUN=true; shift ;;
        *) echo "Unknown: $1"; exit 1 ;;
    esac
done
```

---

## Functions

**29. Define a function?**
```bash
# Method 1
greet() {
    echo "Hello, $1!"
}

# Method 2
function greet {
    echo "Hello, $1!"
}

greet "Vaibhav"
```

**30. Pass arguments to a function?**
```bash
deploy() {
    local app=$1
    local env=$2
    echo "Deploying $app to $env"
}
deploy "myapp" "production"
```

**31. Return a value?**
```bash
# Method 1: return (exit code, 0-255 only)
is_running() {
    pgrep -x "$1" > /dev/null
    return $?
}
if is_running "nginx"; then echo "running"; fi

# Method 2: echo (for string/complex values)
get_ip() {
    echo $(hostname -I | awk '{print $1}')
}
MY_IP=$(get_ip)
```

**32. Scope of variables? global vs local?**
```bash
GLOBAL_VAR="I'm global"

my_func() {
    local LOCAL_VAR="I'm local"    # Only exists inside function
    GLOBAL_VAR="Modified"           # Modifies global
    echo "$LOCAL_VAR"
}
my_func
echo "$GLOBAL_VAR"    # "Modified"
echo "$LOCAL_VAR"     # Empty (not accessible)
```

**33. Function: log messages with timestamp and severity?**
```bash
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message"
}

log INFO "Application started"
log ERROR "Connection failed"
log WARN "Disk usage at 85%"
# [2024-01-15 10:30:00] [INFO] Application started
```

**34. Function: retry command N times with delay?**
```bash
retry() {
    local max_attempts=$1
    local delay=$2
    shift 2
    local cmd="$@"

    for ((attempt=1; attempt<=max_attempts; attempt++)); do
        echo "Attempt $attempt/$max_attempts: $cmd"
        if eval "$cmd"; then
            echo "Success on attempt $attempt"
            return 0
        fi
        if [ $attempt -lt $max_attempts ]; then
            echo "Retrying in ${delay}s..."
            sleep "$delay"
        fi
    done
    echo "Failed after $max_attempts attempts"
    return 1
}

retry 3 5 curl -sf http://localhost:8080/health
```

---

## Error Handling

**35. `set -e`?**
Exit immediately if any command fails (non-zero exit). Script stops at first error.
```bash
#!/bin/bash
set -e
cp important_file /backup/     # If this fails, script stops
echo "This won't run if cp fails"
```

**36. `set -u`?**
Treat unset variables as error. Prevents bugs from typos.
```bash
set -u
echo $UNDEFINED_VAR    # ERROR: unbound variable (script exits)
```

**37. `set -o pipefail`?**
Makes pipeline return failure if ANY command in pipe fails (not just the last one).
```bash
set -o pipefail
cat nonexistent.txt | grep "pattern" | wc -l
# Without pipefail: returns 0 (wc succeeds)
# With pipefail: returns 1 (cat failed)
```

**38. `set -x`?**
Debug mode. Prints each command before execution. Use while debugging.
```bash
set -x
NAME="Vaibhav"
echo "Hello $NAME"
# + NAME=Vaibhav
# + echo 'Hello Vaibhav'
# Hello Vaibhav
```

**39. `set -euo pipefail`?**
```bash
#!/bin/bash
set -euo pipefail
# -e: Exit on error
# -u: Error on undefined variables
# -o pipefail: Pipe fails if any command fails
# BEST PRACTICE: Always include at top of scripts
```

**40. Trap signals?**
```bash
trap 'echo "Caught SIGINT"' INT       # Ctrl+C
trap 'cleanup' EXIT                     # On script exit
trap 'echo "Error on line $LINENO"' ERR # On any error
```

**41. Trap: clean up temp files on exit?**
```bash
#!/bin/bash
set -euo pipefail

TMPDIR=$(mktemp -d)
trap 'rm -rf "$TMPDIR"' EXIT    # Always clean up, even on error

# Use temp dir safely
cp data.txt "$TMPDIR/"
process "$TMPDIR/data.txt"
# TMPDIR auto-deleted on exit
```

**42. Handle errors in a pipeline?**
```bash
set -o pipefail

# Or check PIPESTATUS array
command1 | command2 | command3
echo "Exit codes: ${PIPESTATUS[0]} ${PIPESTATUS[1]} ${PIPESTATUS[2]}"
```

---

## Arrays

**43. Declare an array?**
```bash
fruits=("apple" "banana" "cherry")
numbers=(1 2 3 4 5)
```

**44. Access elements?**
```bash
echo "${fruits[0]}"      # apple (first element)
echo "${fruits[1]}"      # banana
echo "${fruits[@]}"      # All elements
echo "${fruits[-1]}"     # cherry (last element, bash 4.0+)
```

**45. Length of array?**
```bash
echo "${#fruits[@]}"     # 3
```

**46. Iterate over array?**
```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# With index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done
```

**47. Append to array?**
```bash
fruits+=("dragonfruit")
fruits+=("elderberry" "fig")
```

**48. Associative array?**
```bash
declare -A config
config[host]="localhost"
config[port]="8080"
config[env]="production"

echo "${config[host]}"          # localhost
echo "${!config[@]}"            # All keys
echo "${config[@]}"             # All values

for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done
```

---

## String Operations

**49. Length of a string?**
```bash
str="Hello World"
echo "${#str}"          # 11
```

**50. Extract a substring?**
```bash
str="Hello World"
echo "${str:0:5}"       # Hello  (offset:length)
echo "${str:6}"         # World  (from offset)
echo "${str: -5}"       # World  (last 5 chars, note space before -)
```

**51. Replace text in variable?**
```bash
str="Hello World World"
echo "${str/World/Earth}"      # Hello Earth World  (first match)
echo "${str//World/Earth}"     # Hello Earth Earth  (all matches)
```

**52. Remove prefix/suffix?**
```bash
file="/path/to/script.sh"
echo "${file##*/}"       # script.sh    (remove longest prefix matching */)
echo "${file%.*}"        # /path/to/script  (remove shortest suffix matching .*)
echo "${file%%.*}"       # /path/to/script  (remove longest suffix)
echo "${file#*/}"        # path/to/script.sh (remove shortest prefix)

# Useful patterns:
filename="${filepath##*/}"     # Get filename from path
extension="${filename##*.}"    # Get extension
basename="${filename%.*}"      # Get name without extension
directory="${filepath%/*}"     # Get directory
```

**53. Convert to upper/lower case?**
```bash
str="Hello World"
echo "${str^^}"          # HELLO WORLD  (all upper)
echo "${str,,}"          # hello world  (all lower)
echo "${str^}"           # Hello World  (first char upper)
```

---

## Real-World Scripting Scenarios

**54. Monitor disk usage, alert if >80%?**
```bash
#!/bin/bash
set -euo pipefail

THRESHOLD=80
ALERT_EMAIL="admin@company.com"

df -h --output=pcent,target | tail -n+2 | while read -r usage mount; do
    percent=${usage%\%}
    if [ "$percent" -gt "$THRESHOLD" ]; then
        echo "ALERT: $mount is at ${percent}% usage"
        # Send alert
        echo "Disk $mount at ${percent}%" | mail -s "Disk Alert" "$ALERT_EMAIL"
    fi
done
```

**55. Check services running, restart if down?**
```bash
#!/bin/bash
set -euo pipefail

SERVICES=("nginx" "postgresql" "redis")

for svc in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet "$svc"; then
        echo "$(date): $svc is DOWN. Restarting..."
        sudo systemctl restart "$svc"
        if systemctl is-active --quiet "$svc"; then
            echo "$(date): $svc restarted successfully"
        else
            echo "$(date): CRITICAL - $svc failed to restart!"
        fi
    else
        echo "$(date): $svc is running"
    fi
done
```

**56. Log rotation script?**
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
MAX_AGE=30  # days
MAX_FILES=5

for log in "$LOG_DIR"/*.log; do
    [ -f "$log" ] || continue
    # Rotate: rename with timestamp
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    mv "$log" "${log}.${TIMESTAMP}"
    gzip "${log}.${TIMESTAMP}"
    touch "$log"
done

# Delete old compressed logs
find "$LOG_DIR" -name "*.gz" -mtime +$MAX_AGE -delete
```

**57. Find duplicate files?**
```bash
#!/bin/bash
set -euo pipefail

DIR="${1:-.}"
find "$DIR" -type f -exec md5sum {} + | sort | uniq -w 32 -d
# Groups by MD5 hash, shows duplicates
```

**58. Read CSV, generate HTML report?**
```bash
#!/bin/bash
set -euo pipefail

INPUT="data.csv"
OUTPUT="report.html"

cat > "$OUTPUT" << 'HEADER'
<html><head><title>Report</title></head>
<body><table border="1">
HEADER

HEAD=true
while IFS=, read -r col1 col2 col3; do
    if $HEAD; then
        echo "<tr><th>$col1</th><th>$col2</th><th>$col3</th></tr>" >> "$OUTPUT"
        HEAD=false
    else
        echo "<tr><td>$col1</td><td>$col2</td><td>$col3</td></tr>" >> "$OUTPUT"
    fi
done < "$INPUT"

echo "</table></body></html>" >> "$OUTPUT"
echo "Report generated: $OUTPUT"
```

**59. Backup directory to remote via rsync?**
```bash
#!/bin/bash
set -euo pipefail

SRC="/opt/myapp/"
DEST="backup@remote-server:/backups/myapp/"
LOG="/var/log/backup.log"

echo "$(date): Starting backup" >> "$LOG"
rsync -avz --delete \
    --exclude='.git' \
    --exclude='node_modules' \
    --exclude='*.log' \
    "$SRC" "$DEST" >> "$LOG" 2>&1

if [ $? -eq 0 ]; then
    echo "$(date): Backup completed successfully" >> "$LOG"
else
    echo "$(date): Backup FAILED" >> "$LOG"
    exit 1
fi
```

**60. Wait for URL to become healthy (with timeout)?**
```bash
#!/bin/bash
set -euo pipefail

URL="${1:-http://localhost:8080/health}"
TIMEOUT=${2:-120}
INTERVAL=5
ELAPSED=0

echo "Waiting for $URL to become healthy (timeout: ${TIMEOUT}s)..."

until curl -sf "$URL" > /dev/null 2>&1; do
    ELAPSED=$((ELAPSED + INTERVAL))
    if [ $ELAPSED -ge $TIMEOUT ]; then
        echo "TIMEOUT: $URL not healthy after ${TIMEOUT}s"
        exit 1
    fi
    echo "Not ready yet... (${ELAPSED}s elapsed)"
    sleep $INTERVAL
done

echo "✅ $URL is healthy! (took ${ELAPSED}s)"
```

**61. Parse CLI options with getopts?**
```bash
#!/bin/bash
set -euo pipefail

usage() { echo "Usage: $0 -n NAME -e ENV [-v]" >&2; exit 1; }

VERBOSE=false
while getopts "n:e:vh" opt; do
    case $opt in
        n) NAME="$OPTARG" ;;
        e) ENV="$OPTARG" ;;
        v) VERBOSE=true ;;
        h) usage ;;
        *) usage ;;
    esac
done

[ -z "${NAME:-}" ] && usage
[ -z "${ENV:-}" ] && usage

echo "Deploying $NAME to $ENV (verbose: $VERBOSE)"
```

**62. Deploy script: stop, copy, migrate, start?**
```bash
#!/bin/bash
set -euo pipefail

APP="myapp"
DEPLOY_DIR="/opt/$APP"
BACKUP_DIR="/opt/backups/$APP/$(date +%Y%m%d_%H%M%S)"

log() { echo "[$(date '+%H:%M:%S')] $1"; }

log "Creating backup..."
mkdir -p "$BACKUP_DIR"
cp -r "$DEPLOY_DIR" "$BACKUP_DIR/"

log "Stopping service..."
sudo systemctl stop "$APP"

log "Copying new files..."
cp -r ./dist/* "$DEPLOY_DIR/"

log "Running migrations..."
cd "$DEPLOY_DIR" && python manage.py migrate

log "Starting service..."
sudo systemctl start "$APP"

log "Verifying..."
sleep 5
if systemctl is-active --quiet "$APP"; then
    log "✅ Deployment successful!"
else
    log "❌ Deployment failed! Rolling back..."
    cp -r "$BACKUP_DIR"/* "$DEPLOY_DIR/"
    sudo systemctl start "$APP"
    exit 1
fi
```

**63. Collect system info, output JSON?**
```bash
#!/bin/bash
set -euo pipefail

CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
MEM_TOTAL=$(free -m | awk '/Mem:/{print $2}')
MEM_USED=$(free -m | awk '/Mem:/{print $3}')
DISK=$(df -h / | awk 'NR==2{print $5}')
UPTIME=$(uptime -p)
HOSTNAME=$(hostname)

cat << EOF
{
  "hostname": "$HOSTNAME",
  "cpu_usage_percent": $CPU,
  "memory_total_mb": $MEM_TOTAL,
  "memory_used_mb": $MEM_USED,
  "disk_usage_root": "$DISK",
  "uptime": "$UPTIME",
  "timestamp": "$(date -Iseconds)"
}
EOF
```

**64. Kubectl wrapper with logging and error handling?**
```bash
#!/bin/bash
set -euo pipefail

LOG_FILE="/var/log/kubectl-audit.log"

kube() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local user=$(whoami)
    echo "[$timestamp] [$user] kubectl $*" >> "$LOG_FILE"

    if ! kubectl "$@" 2>&1; then
        echo "[$timestamp] [$user] FAILED: kubectl $*" >> "$LOG_FILE"
        return 1
    fi
}

# Usage:
kube get pods -n production
kube apply -f deployment.yaml
```

**65. Compare two directories, show differences?**
```bash
#!/bin/bash
diff -rq "$1" "$2" | sort
# -r recursive, -q brief (only show which files differ)
# Shows: files only in dir1, only in dir2, files that differ
```

**66. Validate YAML/JSON config?**
```bash
#!/bin/bash
set -euo pipefail

validate() {
    local file="$1"
    case "${file##*.}" in
        yaml|yml)
            if python3 -c "import yaml; yaml.safe_load(open('$file'))" 2>/dev/null; then
                echo "✅ $file: Valid YAML"
            else
                echo "❌ $file: Invalid YAML"
                return 1
            fi
            ;;
        json)
            if python3 -m json.tool "$file" > /dev/null 2>&1; then
                echo "✅ $file: Valid JSON"
            else
                echo "❌ $file: Invalid JSON"
                return 1
            fi
            ;;
    esac
}

for f in "$@"; do
    validate "$f"
done
```

**67. Clean up Docker resources?**
```bash
#!/bin/bash
set -euo pipefail

echo "Cleaning Docker resources..."

echo "Stopping all running containers..."
docker stop $(docker ps -q) 2>/dev/null || true

echo "Removing stopped containers..."
docker container prune -f

echo "Removing dangling images..."
docker image prune -f

echo "Removing unused volumes..."
docker volume prune -f

echo "Removing unused networks..."
docker network prune -f

echo "Space reclaimed:"
docker system df
```

**68. Send Slack notification on pipeline failure?**
```bash
#!/bin/bash
set -euo pipefail

WEBHOOK_URL="${SLACK_WEBHOOK_URL}"
PIPELINE="${BUILD_PIPELINE_NAME:-unknown}"
BUILD_NUM="${BUILD_BUILDNUMBER:-unknown}"
STATUS="${1:-failed}"

send_slack() {
    local color="danger"
    [ "$STATUS" = "success" ] && color="good"

    curl -sf -X POST "$WEBHOOK_URL" \
        -H "Content-Type: application/json" \
        -d "{
            \"attachments\": [{
                \"color\": \"$color\",
                \"title\": \"Pipeline: $PIPELINE #$BUILD_NUM\",
                \"text\": \"Status: $STATUS\",
                \"footer\": \"$(date)\"
            }]
        }"
}

send_slack
```

**69. Auto-scale EC2 based on CPU?**
```bash
#!/bin/bash
set -euo pipefail

ASG_NAME="my-asg"
HIGH_THRESHOLD=80
LOW_THRESHOLD=20

CPU=$(aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --statistics Average \
    --period 300 \
    --start-time "$(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%S)" \
    --end-time "$(date -u +%Y-%m-%dT%H:%M:%S)" \
    --dimensions Name=AutoScalingGroupName,Value=$ASG_NAME \
    --query 'Datapoints[0].Average' --output text)

CURRENT=$(aws autoscaling describe-auto-scaling-groups \
    --auto-scaling-group-names $ASG_NAME \
    --query 'AutoScalingGroups[0].DesiredCapacity' --output text)

if (( $(echo "$CPU > $HIGH_THRESHOLD" | bc -l) )); then
    NEW=$((CURRENT + 1))
    echo "CPU at ${CPU}%, scaling up to $NEW"
    aws autoscaling set-desired-capacity --auto-scaling-group-name $ASG_NAME --desired-capacity $NEW
elif (( $(echo "$CPU < $LOW_THRESHOLD" | bc -l) )); then
    NEW=$((CURRENT > 1 ? CURRENT - 1 : 1))
    echo "CPU at ${CPU}%, scaling down to $NEW"
    aws autoscaling set-desired-capacity --auto-scaling-group-name $ASG_NAME --desired-capacity $NEW
fi
```

**70. Health-check script for microservice?**
```bash
#!/bin/bash
set -euo pipefail

SERVICE_URL="http://localhost:8080"
SERVICE_NAME="myapp"

check_http() {
    local status=$(curl -sf -o /dev/null -w '%{http_code}' "$SERVICE_URL/health" 2>/dev/null || echo "000")
    echo "$status"
}

check_process() {
    pgrep -x "$SERVICE_NAME" > /dev/null 2>&1
}

# Main health check
HTTP_STATUS=$(check_http)

if [ "$HTTP_STATUS" = "200" ] && check_process; then
    echo "HEALTHY: $SERVICE_NAME (HTTP $HTTP_STATUS)"
    exit 0
elif [ "$HTTP_STATUS" != "200" ]; then
    echo "UNHEALTHY: $SERVICE_NAME (HTTP $HTTP_STATUS)"
    exit 1
else
    echo "UNHEALTHY: $SERVICE_NAME process not found"
    exit 1
fi
```
