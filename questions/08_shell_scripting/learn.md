# Shell Scripting - LEARNING MATERIAL

---

## Bash Script Structure

```bash
#!/bin/bash
# ^ Shebang - tells OS which interpreter to use

set -euo pipefail    # THE MOST IMPORTANT LINE
# -e  Exit on any error
# -u  Error on undefined variables
# -o pipefail  Pipe fails if ANY command fails

# Variables
NAME="Vaibhav"
VERSION=${1:-"latest"}          # First arg, default "latest"
TIMESTAMP=$(date +%Y%m%d_%H%M)  # Command substitution

echo "Deploying $NAME version $VERSION at $TIMESTAMP"
```

## Variables & Quoting

```bash
# Assignment (NO spaces around =)
name="hello"
count=42

# Referencing
echo "$name"          # hello
echo "${name}_world"  # hello_world
echo '$name'          # $name (single quotes = literal)

# Command substitution
files=$(ls -la)
today=$(date +%F)

# Arithmetic
count=$((count + 1))
result=$((10 * 5 / 2))

# Arrays
fruits=("apple" "banana" "cherry")
echo "${fruits[0]}"        # apple
echo "${fruits[@]}"        # all elements
echo "${#fruits[@]}"       # length: 3
fruits+=("date")           # append
```

## Conditionals

```bash
# String comparison
if [[ "$env" == "prod" ]]; then
    echo "Production!"
elif [[ "$env" == "staging" ]]; then
    echo "Staging"
else
    echo "Other: $env"
fi

# Numeric comparison
if [[ $count -gt 10 ]]; then echo "More than 10"; fi
# -eq, -ne, -lt, -le, -gt, -ge

# File tests
if [[ -f "/etc/config" ]]; then echo "File exists"; fi
if [[ -d "/var/log" ]]; then echo "Directory exists"; fi
if [[ -x "$script" ]]; then echo "Is executable"; fi
if [[ -z "$var" ]]; then echo "Variable is empty"; fi
if [[ -n "$var" ]]; then echo "Variable is not empty"; fi

# Logical operators
if [[ "$a" == "x" && "$b" == "y" ]]; then echo "Both"; fi
if [[ "$a" == "x" || "$b" == "y" ]]; then echo "Either"; fi
```

## Loops

```bash
# For loop
for server in web1 web2 web3; do
    echo "Deploying to $server"
done

# C-style for
for ((i=0; i<5; i++)); do
    echo "Iteration $i"
done

# Iterate over files
for file in /var/log/*.log; do
    echo "Processing $file"
done

# While loop
while read -r line; do
    echo "Line: $line"
done < input.txt

# While with counter
count=0
while [[ $count -lt 10 ]]; do
    echo "$count"
    ((count++))
done
```

## Functions

```bash
# Function definition
deploy() {
    local env="$1"          # local scope
    local version="${2:-latest}"

    echo "Deploying $version to $env"

    if [[ "$env" == "prod" ]]; then
        return 0   # success
    fi
    return 1       # failure
}

# Call function
deploy "staging" "v1.2.3"
if deploy "prod" "v2.0"; then
    echo "Deploy succeeded"
fi
```

## Text Processing Pipeline

```mermaid
graph LR
    CAT[cat file] --> GREP[grep pattern]
    GREP --> AWK[awk '{print $2}']
    AWK --> SORT[sort]
    SORT --> UNIQ[uniq -c]
    UNIQ --> HEAD[head -10]
```

```bash
# Find top 10 IP addresses in access log
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Extract field from CSV
cut -d',' -f2 data.csv

# Replace text
sed 's/old/new/g' file.txt
sed -i 's/http:/https:/g' config.conf   # in-place edit

# AWK - powerful text processing
awk -F: '{print $1, $3}' /etc/passwd    # print user and UID
awk '$3 > 1000' /etc/passwd             # filter by field value

# grep patterns
grep -r "ERROR" /var/log/           # recursive search
grep -i "warning" app.log           # case-insensitive
grep -c "pattern" file              # count matches
grep -v "DEBUG" app.log             # invert (exclude)
grep -E "error|warning|critical" log # extended regex (OR)
```

## Practical DevOps Scripts

### Health Check Script
```bash
#!/bin/bash
set -euo pipefail

SERVICES=("web:8080" "api:3000" "db:5432")

for svc in "${SERVICES[@]}"; do
    host="${svc%%:*}"     # everything before :
    port="${svc##*:}"     # everything after :

    if nc -z "$host" "$port" 2>/dev/null; then
        echo "[OK] $host:$port"
    else
        echo "[FAIL] $host:$port"
        exit 1
    fi
done
```

### Log Rotation Script
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
MAX_AGE=30  # days

find "$LOG_DIR" -name "*.log" -mtime +$MAX_AGE -exec gzip {} \;
find "$LOG_DIR" -name "*.gz" -mtime +90 -delete

echo "Rotated logs older than $MAX_AGE days"
```
