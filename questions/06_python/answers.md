# Python - COMPREHENSIVE ANSWERS (All 100 Questions)

---

# PART 1: BASICS & CORE (40 Qs)

---

## Data Types & Structures

**1. Basic data types in Python?**
- `int`: 42
- `float`: 3.14
- `str`: "hello"
- `bool`: True/False
- `NoneType`: None
- `complex`: 3+4j
- Collections: `list`, `tuple`, `set`, `dict`

**2. Difference between list, tuple, set, dictionary?**
| Type | Ordered | Mutable | Duplicates | Syntax |
|---|---|---|---|---|
| list | Yes | Yes | Yes | `[1, 2, 3]` |
| tuple | Yes | No | Yes | `(1, 2, 3)` |
| set | No | Yes | No | `{1, 2, 3}` |
| dict | Yes (3.7+) | Yes | Keys: No | `{"a": 1}` |

**3. When tuple instead of list?**
- Fixed data that shouldn't change (coordinates, RGB values, database records)
- Dictionary keys (tuples are hashable, lists aren't)
- Function return values
- Slightly faster and less memory than lists

**4. List comprehension? 3 examples.**
```python
# 1. Squares of numbers
squares = [x**2 for x in range(10)]

# 2. Filter even numbers
evens = [x for x in range(20) if x % 2 == 0]

# 3. Flatten nested list
flat = [item for sublist in [[1,2],[3,4],[5,6]] for item in sublist]
```

**5. Dictionary comprehension?**
```python
# From two lists
keys = ['a', 'b', 'c']
vals = [1, 2, 3]
d = {k: v for k, v in zip(keys, vals)}

# Transform values
word_lengths = {word: len(word) for word in ["hello", "world"]}

# Filter
adults = {name: age for name, age in people.items() if age >= 18}
```

**6. Merge two dictionaries?**
```python
# Python 3.9+ (preferred)
merged = dict1 | dict2

# Python 3.5+
merged = {**dict1, **dict2}

# Older
dict1.update(dict2)  # Modifies dict1 in-place
```

**7. `==` vs `is`?**
- `==`: Checks **value** equality (calls `__eq__`)
- `is`: Checks **identity** (same object in memory, same `id()`)
```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b    # True (same value)
a is b    # False (different objects)
```

**8. Mutable vs immutable types?**
- **Mutable** (can change): `list`, `dict`, `set`, custom objects
- **Immutable** (cannot change): `int`, `float`, `str`, `tuple`, `frozenset`, `bool`
```python
# Immutable: creates new object
s = "hello"
s = s + " world"  # New string object created

# Mutable: modifies in place
lst = [1, 2]
lst.append(3)      # Same object modified
```

**9. Slicing?**
```python
lst = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
lst[2:5]      # [2, 3, 4]           start:stop
lst[:3]       # [0, 1, 2]           first 3
lst[7:]       # [7, 8, 9]           from index 7
lst[-3:]      # [7, 8, 9]           last 3
lst[::2]      # [0, 2, 4, 6, 8]    step of 2
lst[::-1]     # [9, 8, 7, ..., 0]  reverse
lst[1:8:2]    # [1, 3, 5, 7]       start:stop:step
```

**10. Sort list of dicts by specific key?**
```python
users = [{"name": "Bob", "age": 30}, {"name": "Alice", "age": 25}]
sorted_users = sorted(users, key=lambda x: x["age"])
# Or descending:
sorted_users = sorted(users, key=lambda x: x["age"], reverse=True)
```

---

## Functions

**11. `*args` and `**kwargs`?**
```python
def func(*args, **kwargs):
    # args = tuple of positional args
    # kwargs = dict of keyword args
    print(args)    # (1, 2, 3)
    print(kwargs)  # {'name': 'Vaibhav', 'age': 30}

func(1, 2, 3, name="Vaibhav", age=30)
```

**12. Lambda function? When use?**
Anonymous, single-expression function:
```python
square = lambda x: x**2
# Use in: sort, filter, map — when function is short and used once
sorted(data, key=lambda x: x['name'])
list(filter(lambda x: x > 0, numbers))
```

**13. `return` vs `yield`?**
- `return`: Returns a value and **terminates** the function
- `yield`: Returns a value and **pauses** the function (resumes on next call)
```python
def get_numbers():
    yield 1      # Pauses here, returns 1
    yield 2      # Resumes, returns 2
    yield 3      # Resumes, returns 3

for n in get_numbers():
    print(n)     # 1, 2, 3
```

**14. Generator? Why memory-efficient?**
A function using `yield` that produces values lazily (one at a time), not all at once. Only holds one value in memory.
```python
# List: All 1M items in memory at once
big_list = [x**2 for x in range(1_000_000)]  # ~8MB

# Generator: One item at a time
big_gen = (x**2 for x in range(1_000_000))   # ~120 bytes
```

**15. Built-in functions? Name 10.**
`len()`, `range()`, `print()`, `type()`, `isinstance()`, `enumerate()`, `zip()`, `sorted()`, `map()`, `filter()`, `any()`, `all()`, `min()`, `max()`, `sum()`, `abs()`, `round()`, `input()`, `open()`, `int()`, `str()`, `list()`, `dict()`

**16. `map()`, `filter()`, `reduce()`?**
```python
# map: apply function to each element
list(map(str, [1, 2, 3]))           # ['1', '2', '3']

# filter: keep elements where function returns True
list(filter(lambda x: x > 0, [-1, 2, -3, 4]))  # [2, 4]

# reduce: accumulate values (from functools)
from functools import reduce
reduce(lambda a, b: a + b, [1, 2, 3, 4])  # 10
```

**17. Decorator? Write one.**
Function that wraps another function to add behavior:
```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} took {elapsed:.2f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

slow_function()  # "slow_function took 1.00s"
```

**18. Closure?**
A function that remembers variables from its enclosing scope even after that scope has finished:
```python
def make_multiplier(n):
    def multiplier(x):
        return x * n    # 'n' is remembered from outer scope
    return multiplier

double = make_multiplier(2)
double(5)   # 10
triple = make_multiplier(3)
triple(5)   # 15
```

---

## Control Flow & Error Handling

**19. `for` vs `while` loops?**
- `for`: Iterate over a sequence (list, range, dict). Known iteration count.
- `while`: Loop until condition is False. Unknown iteration count.
```python
for item in [1, 2, 3]:    # Iterate over sequence
    print(item)

while not done:            # Loop until condition met
    data = fetch_data()
    done = process(data)
```

**20. `try/except/else/finally`?**
```python
try:
    result = 10 / x
except ZeroDivisionError:
    print("Can't divide by zero")
except Exception as e:
    print(f"Error: {e}")
else:
    print("Success!")      # Runs only if NO exception
finally:
    print("Always runs")   # Runs always (cleanup)
```

**21. Raise custom exception?**
```python
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f"Cannot withdraw {amount} from balance {balance}")

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount
```

**22. `break`, `continue`, `pass`?**
```python
for i in range(10):
    if i == 5:
        break       # Exit loop entirely
    if i % 2 == 0:
        continue    # Skip to next iteration
    pass            # Do nothing (placeholder)
```

**23. `with` statement (context manager)?**
Automatically handles setup/cleanup (e.g., closing files, releasing locks):
```python
# Without with: must remember to close
f = open("file.txt")
data = f.read()
f.close()

# With: auto-closes even on exception
with open("file.txt") as f:
    data = f.read()
# File is automatically closed here
```

---

## String Operations

**24. 5 string methods?**
```python
s = "  Hello, World!  "
s.strip()         # "Hello, World!"   Remove whitespace
s.lower()         # "  hello, world!  "
s.upper()         # "  HELLO, WORLD!  "
s.replace("World", "Python")
s.startswith("  He")  # True
s.split(",")      # ['  Hello', ' World!  ']
s.find("World")   # 9 (index) or -1 if not found
s.count("l")      # 2
s.isdigit()       # False
```

**25. f-string vs `.format()` vs `%`?**
```python
name = "Vaibhav"
age = 30

# f-string (Python 3.6+, preferred)
f"Name: {name}, Age: {age}"

# .format()
"Name: {}, Age: {}".format(name, age)

# % formatting (legacy)
"Name: %s, Age: %d" % (name, age)
```

**26. Split/Join?**
```python
# Split: string → list
"a,b,c".split(",")          # ['a', 'b', 'c']
"hello world".split()       # ['hello', 'world']

# Join: list → string
",".join(['a', 'b', 'c'])   # 'a,b,c'
"\n".join(lines)             # Join with newlines
```

**27. Regex in Python?**
```python
import re

# Search
match = re.search(r'\d+', 'age is 30')
match.group()    # '30'

# Find all
re.findall(r'\d+', 'ages: 25, 30, 35')   # ['25', '30', '35']

# Replace
re.sub(r'\d+', 'X', 'age 30')            # 'age X'

# Match (start of string only)
re.match(r'\d+', '123abc')               # Match object
```

**28. Regex for IP address?**
```python
ip_pattern = r'\b(?:\d{1,3}\.){3}\d{1,3}\b'
ips = re.findall(ip_pattern, "Server at 192.168.1.1 and 10.0.0.5")
# Strict validation:
import ipaddress
try:
    ipaddress.ip_address("192.168.1.1")   # Valid
except ValueError:
    pass
```

**29. Regex for email?**
```python
email_pattern = r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
emails = re.findall(email_pattern, text)
```

---

## File I/O

**30. Read a file?**
```python
# Best practice
with open("file.txt", "r") as f:
    content = f.read()

# Line by line (memory efficient for large files)
with open("file.txt") as f:
    for line in f:
        process(line.strip())
```

**31. `read()`, `readline()`, `readlines()`?**
```python
f.read()          # Entire file as one string
f.readline()      # One line at a time
f.readlines()     # All lines as list of strings
```

**32. Write to a file?**
```python
with open("output.txt", "w") as f:    # 'w' overwrites
    f.write("Hello\n")

with open("output.txt", "a") as f:    # 'a' appends
    f.write("World\n")
```

**33. Read/write JSON?**
```python
import json

# Read
with open("config.json") as f:
    data = json.load(f)

# Write
with open("output.json", "w") as f:
    json.dump(data, f, indent=2)

# String conversion
json_str = json.dumps(data)
data = json.loads(json_str)
```

**34. Read/write CSV?**
```python
import csv

# Read
with open("data.csv") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], row['age'])

# Write
with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "Vaibhav", "age": 30})
```

**35. Handle file encoding issues?**
```python
with open("file.txt", encoding="utf-8") as f:
    content = f.read()

# Handle unknown encoding
with open("file.txt", encoding="utf-8", errors="replace") as f:
    content = f.read()

# Detect encoding
import chardet
with open("file.txt", "rb") as f:
    result = chardet.detect(f.read())
    encoding = result['encoding']
```

---

## Interview-Style

**36. Most frequent element in a list?**
```python
from collections import Counter
def most_frequent(lst):
    return Counter(lst).most_common(1)[0][0]

# Without imports:
def most_frequent(lst):
    return max(set(lst), key=lst.count)
```

**37. Reverse string without built-in reverse?**
```python
def reverse_string(s):
    return s[::-1]

# Manual:
def reverse_string(s):
    result = ""
    for char in s:
        result = char + result
    return result
```

**38. Check palindrome?**
```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]
```

**39. Flatten nested list?**
```python
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

flatten([1, [2, [3, 4]], [5, 6]])  # [1, 2, 3, 4, 5, 6]
```

**40. Read log file, count errors per hour?**
```python
from collections import defaultdict
import re

errors_per_hour = defaultdict(int)
with open("app.log") as f:
    for line in f:
        if "ERROR" in line:
            # Assuming format: 2024-01-15 14:30:00 ERROR message
            match = re.search(r'(\d{4}-\d{2}-\d{2} \d{2}):', line)
            if match:
                hour = match.group(1)
                errors_per_hour[hour] += 1

for hour, count in sorted(errors_per_hour.items()):
    print(f"{hour}: {count} errors")
```

---

# PART 2: OS, SUBPROCESS, APIs & ADVANCED (60 Qs)

---

## os & sys Modules

**1. Get current working directory?**
```python
import os
os.getcwd()

from pathlib import Path
Path.cwd()
```

**2. List files in a directory?**
```python
os.listdir("/path/to/dir")

# pathlib (preferred)
list(Path("/path").iterdir())

# Only files
[f for f in Path("/path").iterdir() if f.is_file()]

# With glob pattern
list(Path("/path").glob("*.py"))
```

**3. Check if file/directory exists?**
```python
os.path.exists("/path/to/file")
os.path.isfile("/path/to/file")
os.path.isdir("/path/to/dir")

# pathlib
Path("/path/to/file").exists()
Path("/path/to/file").is_file()
```

**4. Create directory? Nested?**
```python
os.mkdir("/path/to/dir")           # Single directory
os.makedirs("/path/to/nested/dir", exist_ok=True)  # Nested

Path("/path/to/nested/dir").mkdir(parents=True, exist_ok=True)
```

**5. Delete file? Directory?**
```python
os.remove("file.txt")                # File
os.rmdir("empty_dir")                # Empty directory
import shutil
shutil.rmtree("non_empty_dir")       # Non-empty directory
```

**6. Get environment variables?**
```python
os.environ.get("HOME")               # Returns None if not set
os.environ["HOME"]                    # Raises KeyError if not set
os.getenv("HOME", "/default/path")   # With default
```

**7. Set environment variables?**
```python
os.environ["MY_VAR"] = "value"        # Only for current process + children
# Does NOT persist after script ends
```

**8. Walk directory tree recursively?**
```python
for root, dirs, files in os.walk("/path"):
    for file in files:
        full_path = os.path.join(root, file)
        print(full_path)

# pathlib
for path in Path("/path").rglob("*.py"):
    print(path)
```

**9. File size? Last modified time?**
```python
os.path.getsize("file.txt")                   # Bytes
os.path.getmtime("file.txt")                  # Timestamp

# pathlib
p = Path("file.txt")
p.stat().st_size                               # Bytes
p.stat().st_mtime                              # Timestamp

import datetime
datetime.datetime.fromtimestamp(p.stat().st_mtime)
```

**10. `os.path.join()`? Why not string concatenation?**
```python
# WRONG (breaks on different OS)
path = "/home" + "/" + "user" + "/" + "file.txt"

# RIGHT (handles OS-specific separators)
path = os.path.join("/home", "user", "file.txt")

# pathlib (even better)
path = Path("/home") / "user" / "file.txt"
```

**11. `sys.argv`? Parse CLI arguments?**
```python
import sys
# python script.py arg1 arg2
sys.argv[0]   # "script.py"
sys.argv[1]   # "arg1"

# Better: use argparse
import argparse
parser = argparse.ArgumentParser(description="My tool")
parser.add_argument("--name", required=True, help="Your name")
parser.add_argument("--verbose", action="store_true")
args = parser.parse_args()
print(args.name)
```

**12. `sys.exit()`? Exit codes?**
```python
sys.exit(0)    # Success
sys.exit(1)    # General error
sys.exit(2)    # Command line usage error
# Convention: 0 = success, non-zero = error
```

**13. `pathlib` vs `os.path`?**
```python
# os.path (functional, string-based)
os.path.join("/home", "user")
os.path.exists("/home/user")

# pathlib (object-oriented, preferred in modern Python)
p = Path("/home") / "user"
p.exists()
p.is_file()
p.read_text()
p.write_text("content")
p.suffix      # ".txt"
p.stem        # "filename"
p.parent      # Path("/home")
```

---

## subprocess Module (CRITICAL)

**14. subprocess module? Why not `os.system()`?**
`os.system()` is legacy: no output capture, no error handling, shell injection risk. `subprocess` provides: output capture, return codes, timeout, security.

**15. `subprocess.run()` basic syntax?**
```python
import subprocess
result = subprocess.run(["ls", "-la"], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)
```

**16. Capture stdout and stderr?**
```python
result = subprocess.run(
    ["command", "arg"],
    capture_output=True,   # Capture stdout and stderr
    text=True              # Return as strings (not bytes)
)
print(result.stdout)
print(result.stderr)
```

**17. `capture_output=True`? `text=True`?**
- `capture_output=True`: Shorthand for `stdout=subprocess.PIPE, stderr=subprocess.PIPE`
- `text=True`: Returns strings instead of bytes. Shorthand for `encoding='utf-8'`.

**18. Check if command succeeded? `check=True`?**
```python
# Manual check
result = subprocess.run(["ls", "/nonexistent"], capture_output=True, text=True)
if result.returncode != 0:
    print(f"Error: {result.stderr}")

# Auto-raise exception on failure
try:
    result = subprocess.run(["ls", "/nonexistent"], capture_output=True, text=True, check=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed: {e.stderr}")
```

**19. Pipe output of one command to another?**
```python
# Method 1: Shell pipe (less secure)
result = subprocess.run("ps aux | grep python", shell=True, capture_output=True, text=True)

# Method 2: Proper pipe (more secure)
ps = subprocess.Popen(["ps", "aux"], stdout=subprocess.PIPE)
grep = subprocess.Popen(["grep", "python"], stdin=ps.stdout, stdout=subprocess.PIPE, text=True)
ps.stdout.close()
output = grep.communicate()[0]
```

**20. Set timeout for subprocess?**
```python
try:
    result = subprocess.run(["long_command"], timeout=30, capture_output=True, text=True)
except subprocess.TimeoutExpired:
    print("Command timed out after 30 seconds")
```

**21. Run with elevated privileges?**
```python
# Run command with sudo
result = subprocess.run(["sudo", "systemctl", "restart", "nginx"],
                       capture_output=True, text=True)
```

**22. `subprocess.Popen()` vs `run()`?**
- `run()`: Simple, waits for completion, returns CompletedProcess. **Use by default.**
- `Popen()`: Low-level, non-blocking, for complex scenarios (streaming output, interactive processes, pipes between commands).
```python
# Streaming output line by line
proc = subprocess.Popen(["tail", "-f", "app.log"], stdout=subprocess.PIPE, text=True)
for line in proc.stdout:
    print(line.strip())
```

**23. Run `git status` and parse output?**
```python
import subprocess

result = subprocess.run(["git", "status", "--porcelain"],
                       capture_output=True, text=True, check=True)

modified = []
untracked = []
for line in result.stdout.strip().split("\n"):
    if line.startswith(" M"):
        modified.append(line[3:])
    elif line.startswith("??"):
        untracked.append(line[3:])

print(f"Modified: {modified}")
print(f"Untracked: {untracked}")
```

**24. Run `kubectl get pods` and check for unhealthy?**
```python
import subprocess
import json

result = subprocess.run(
    ["kubectl", "get", "pods", "-o", "json", "-n", "production"],
    capture_output=True, text=True, check=True
)
pods = json.loads(result.stdout)

unhealthy = []
for pod in pods["items"]:
    name = pod["metadata"]["name"]
    phase = pod["status"]["phase"]
    if phase != "Running":
        unhealthy.append(f"{name}: {phase}")
    else:
        for cs in pod["status"].get("containerStatuses", []):
            if not cs["ready"]:
                unhealthy.append(f"{name}: container {cs['name']} not ready")

if unhealthy:
    print("UNHEALTHY PODS:")
    for p in unhealthy:
        print(f"  - {p}")
else:
    print("All pods healthy")
```

**25. Handle CalledProcessError?**
```python
try:
    result = subprocess.run(
        ["kubectl", "apply", "-f", "manifest.yaml"],
        capture_output=True, text=True, check=True
    )
    print(result.stdout)
except subprocess.CalledProcessError as e:
    print(f"Command failed with exit code {e.returncode}")
    print(f"stderr: {e.stderr}")
    print(f"stdout: {e.stdout}")
```

---

## REST APIs & HTTP

**26. requests library?**
Third-party HTTP library. Simple, elegant. `pip install requests`.
```python
import requests
response = requests.get("https://api.example.com/data")
```

**27. GET, POST, PUT, DELETE requests?**
```python
# GET
resp = requests.get("https://api.example.com/users")

# POST
resp = requests.post("https://api.example.com/users", json={"name": "Vaibhav"})

# PUT
resp = requests.put("https://api.example.com/users/1", json={"name": "Updated"})

# DELETE
resp = requests.delete("https://api.example.com/users/1")

# PATCH
resp = requests.patch("https://api.example.com/users/1", json={"age": 31})
```

**28. Headers, query params, request body?**
```python
resp = requests.get(
    "https://api.example.com/search",
    headers={"Authorization": "Bearer TOKEN", "Accept": "application/json"},
    params={"q": "python", "page": 1},     # ?q=python&page=1
)

resp = requests.post(
    "https://api.example.com/data",
    json={"key": "value"},                  # JSON body
    # OR data={"key": "value"}              # Form data
)
```

**29. Handle authentication?**
```python
# Bearer token
headers = {"Authorization": "Bearer YOUR_TOKEN"}
resp = requests.get(url, headers=headers)

# Basic auth
from requests.auth import HTTPBasicAuth
resp = requests.get(url, auth=HTTPBasicAuth("user", "pass"))

# API key in header
headers = {"X-API-Key": "your-api-key"}
resp = requests.get(url, headers=headers)
```

**30. response.json()? response.status_code?**
```python
resp = requests.get(url)
resp.status_code      # 200, 404, 500, etc.
resp.json()           # Parse JSON response body to dict
resp.text             # Raw text response
resp.headers          # Response headers (dict)
resp.ok               # True if status < 400
```

**31. response.raise_for_status()?**
```python
resp = requests.get(url)
resp.raise_for_status()   # Raises HTTPError if status >= 400
# Use it to fail fast on errors instead of checking status manually
```

**32. Handle pagination?**
```python
all_items = []
page = 1
while True:
    resp = requests.get(f"{url}?page={page}&per_page=100")
    data = resp.json()
    if not data:
        break
    all_items.extend(data)
    page += 1
```

**33. Handle rate limiting?**
```python
import time

def api_call_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        resp = requests.get(url)
        if resp.status_code == 429:  # Too Many Requests
            retry_after = int(resp.headers.get("Retry-After", 60))
            time.sleep(retry_after)
            continue
        return resp
    raise Exception("Rate limit exceeded after retries")
```

**34. Upload a file via API?**
```python
with open("report.pdf", "rb") as f:
    resp = requests.post(url, files={"file": f})
```

**35. GitHub API - list repos for user?**
```python
import requests

username = "octocat"
resp = requests.get(f"https://api.github.com/users/{username}/repos")
repos = resp.json()

for repo in repos:
    print(f"{repo['name']}: {repo['html_url']}")
```

**36. Azure DevOps API - get pipeline runs?**
```python
import requests
from requests.auth import HTTPBasicAuth

org = "myorg"
project = "myproject"
pat = "YOUR_PAT"
url = f"https://dev.azure.com/{org}/{project}/_apis/pipelines?api-version=7.0"

resp = requests.get(url, auth=HTTPBasicAuth("", pat))
pipelines = resp.json()["value"]
for p in pipelines:
    print(f"{p['name']} (ID: {p['id']})")
```

---

## OOP

**37. Class? Object?**
```python
class Dog:              # Class: blueprint
    species = "Canine"  # Class attribute

    def __init__(self, name, age):
        self.name = name   # Instance attribute
        self.age = age

    def bark(self):
        return f"{self.name} says Woof!"

rex = Dog("Rex", 5)    # Object: instance of class
```

**38. `__init__`? `self`?**
- `__init__`: Constructor. Called when object is created. Initializes attributes.
- `self`: Reference to the current instance. First parameter of every instance method.

**39. Inheritance?**
```python
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        return "..."

class Dog(Animal):           # Dog inherits from Animal
    def speak(self):         # Override
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"
```

**40. Method overriding?**
Child class redefines a method from parent class. When called on child instance, child's version is used.

**41. Class methods vs static methods vs instance methods?**
```python
class MyClass:
    class_var = 0

    def instance_method(self):        # Access instance (self)
        return self.class_var

    @classmethod
    def class_method(cls):            # Access class (cls), not instance
        cls.class_var += 1

    @staticmethod
    def static_method():             # No access to class or instance
        return "utility function"
```

**42. `__str__` vs `__repr__`?**
```python
class User:
    def __init__(self, name):
        self.name = name

    def __str__(self):           # For end users (print, f-string)
        return f"User: {self.name}"

    def __repr__(self):          # For developers (debugging, repr())
        return f"User('{self.name}')"
```

**43. Dunder/magic methods? Name 5.**
```python
__init__      # Constructor
__str__       # String representation
__repr__      # Developer representation
__len__       # len(obj)
__getitem__   # obj[key]
__eq__        # obj1 == obj2
__lt__        # obj1 < obj2
__enter__/__exit__  # Context manager (with statement)
__iter__/__next__   # Iterator protocol
```

**44. Polymorphism?**
Same interface, different behavior. Multiple classes with same method name but different implementations:
```python
animals = [Dog("Rex"), Cat("Whiskers")]
for animal in animals:
    print(animal.speak())    # Each uses its own speak() method
```

**45. Encapsulation? Private attributes?**
```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance       # Convention: "protected" (single _)
        self.__secret = "hidden"      # Name mangling: "private" (double __)

    @property
    def balance(self):                # Getter
        return self._balance

    @balance.setter
    def balance(self, value):         # Setter with validation
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value
```

---

## Testing

**46. pytest? Write test function?**
```python
# test_math.py
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0

# Run: pytest test_math.py
```

**47. unittest vs pytest?**
- `unittest`: Built-in, class-based, verbose. Java-style (TestCase, setUp, tearDown).
- `pytest`: Third-party, function-based, simpler. Preferred by most. Better output, fixtures, plugins.

**48. Fixtures in pytest?**
```python
import pytest

@pytest.fixture
def sample_data():
    return {"name": "Vaibhav", "age": 30}

def test_name(sample_data):
    assert sample_data["name"] == "Vaibhav"

@pytest.fixture
def db_connection():
    conn = create_connection()
    yield conn             # Provide to test
    conn.close()           # Cleanup after test
```

**49. Mocking? When use?**
Replace real objects with fake ones during testing. Use when: testing code that calls APIs, databases, or external services.
```python
from unittest.mock import patch, MagicMock

@patch("mymodule.requests.get")
def test_api_call(mock_get):
    mock_get.return_value.json.return_value = {"status": "ok"}
    mock_get.return_value.status_code = 200
    result = my_function()
    assert result == "ok"
```

**50. Test function calling external API?**
```python
from unittest.mock import patch

def get_user(user_id):
    resp = requests.get(f"https://api.example.com/users/{user_id}")
    return resp.json()

@patch("mymodule.requests.get")
def test_get_user(mock_get):
    mock_get.return_value.json.return_value = {"id": 1, "name": "Vaibhav"}
    user = get_user(1)
    assert user["name"] == "Vaibhav"
    mock_get.assert_called_once_with("https://api.example.com/users/1")
```

---

## DevOps Scripting Scenarios

**51. Parse Jenkins build logs, extract failure reasons?**
```python
import re

def parse_jenkins_log(log_path):
    failures = []
    with open(log_path) as f:
        for line in f:
            if "FAILED" in line or "ERROR" in line:
                failures.append(line.strip())
            # Check for test failures
            match = re.search(r'Tests run: (\d+), Failures: (\d+)', line)
            if match and int(match.group(2)) > 0:
                failures.append(f"Test failures: {match.group(2)}")
    return failures
```

**52. Check if all services in K8s namespace are healthy?**
```python
import subprocess, json

def check_namespace_health(namespace):
    result = subprocess.run(
        ["kubectl", "get", "pods", "-n", namespace, "-o", "json"],
        capture_output=True, text=True, check=True
    )
    pods = json.loads(result.stdout)["items"]
    unhealthy = []
    for pod in pods:
        name = pod["metadata"]["name"]
        phase = pod["status"]["phase"]
        if phase not in ("Running", "Succeeded"):
            unhealthy.append(f"{name}: {phase}")
            continue
        for cs in pod["status"].get("containerStatuses", []):
            if not cs["ready"]:
                restarts = cs["restartCount"]
                unhealthy.append(f"{name}/{cs['name']}: not ready (restarts: {restarts})")
    return unhealthy

issues = check_namespace_health("production")
if issues:
    print("UNHEALTHY:")
    for i in issues: print(f"  ❌ {i}")
else:
    print("All services healthy ✅")
```

**53. Monitor disk usage, alert if >80%?**
```python
import shutil
import smtplib

def check_disk_usage(path="/", threshold=80):
    usage = shutil.disk_usage(path)
    percent = (usage.used / usage.total) * 100
    if percent > threshold:
        print(f"ALERT: Disk usage at {percent:.1f}% on {path}")
        # Send alert (email, Slack, etc.)
        return True
    print(f"OK: Disk usage at {percent:.1f}%")
    return False

check_disk_usage("/")
```

**54. Rotate AWS access keys older than 90 days?**
```python
import subprocess, json
from datetime import datetime, timezone

result = subprocess.run(
    ["aws", "iam", "list-access-keys", "--user-name", "myuser", "--output", "json"],
    capture_output=True, text=True, check=True
)
keys = json.loads(result.stdout)["AccessKeyMetadata"]

for key in keys:
    created = key["CreateDate"]
    age = (datetime.now(timezone.utc) - datetime.fromisoformat(created.replace("Z", "+00:00"))).days
    if age > 90:
        print(f"Key {key['AccessKeyId']} is {age} days old - rotating")
        # 1. Create new key
        # 2. Update applications
        # 3. Deactivate old key
        # 4. Delete old key after verification
```

**55. Read YAML, modify value, write back?**
```python
import yaml

with open("config.yaml") as f:
    config = yaml.safe_load(f)

config["deployment"]["replicas"] = 5
config["deployment"]["image"]["tag"] = "v2.0"

with open("config.yaml", "w") as f:
    yaml.dump(config, f, default_flow_style=False)
```

**56. Generate deployment report from Azure DevOps API?**
```python
import requests
from requests.auth import HTTPBasicAuth
from datetime import datetime

pat = "YOUR_PAT"
org = "myorg"
project = "myproject"

# Get recent pipeline runs
url = f"https://dev.azure.com/{org}/{project}/_apis/build/builds?api-version=7.0&$top=20"
resp = requests.get(url, auth=HTTPBasicAuth("", pat))
builds = resp.json()["value"]

report = []
for build in builds:
    report.append({
        "pipeline": build["definition"]["name"],
        "status": build["status"],
        "result": build.get("result", "in progress"),
        "started": build.get("startTime", "N/A"),
        "finished": build.get("finishTime", "N/A"),
    })

print(f"Deployment Report - {datetime.now().strftime('%Y-%m-%d')}")
for r in report:
    print(f"  {r['pipeline']}: {r['result']}")
```

**57. Clean up Docker images older than 30 days?**
```python
import subprocess, json
from datetime import datetime, timezone

result = subprocess.run(
    ["docker", "images", "--format", "{{json .}}"],
    capture_output=True, text=True
)

for line in result.stdout.strip().split("\n"):
    image = json.loads(line)
    created = image["CreatedAt"]
    # Parse and check age
    if image["Repository"] != "<none>":
        # Delete dangling images
        subprocess.run(["docker", "image", "prune", "-f", "--filter", "until=720h"],
                      capture_output=True)
```

**58. Validate JSON/YAML config files?**
```python
import json, yaml, sys
from pathlib import Path

def validate_config(filepath):
    path = Path(filepath)
    try:
        content = path.read_text()
        if path.suffix in ('.yaml', '.yml'):
            data = yaml.safe_load(content)
        elif path.suffix == '.json':
            data = json.loads(content)
        else:
            print(f"Unknown format: {path.suffix}")
            return False
        print(f"✅ {filepath} is valid")
        return True
    except (json.JSONDecodeError, yaml.YAMLError) as e:
        print(f"❌ {filepath}: {e}")
        return False

# Validate all config files
for f in Path("configs/").rglob("*.yaml"):
    validate_config(f)
```

**59. Compare two config files and show differences?**
```python
import yaml
from deepdiff import DeepDiff  # pip install deepdiff

with open("config_old.yaml") as f:
    old = yaml.safe_load(f)
with open("config_new.yaml") as f:
    new = yaml.safe_load(f)

diff = DeepDiff(old, new, verbose_level=2)
if diff:
    print("Differences found:")
    for change_type, changes in diff.items():
        print(f"\n{change_type}:")
        for change in changes:
            print(f"  {change}")
else:
    print("Files are identical")
```

**60. Bulk-create Azure DevOps work items from CSV?**
```python
import csv, requests
from requests.auth import HTTPBasicAuth

pat = "YOUR_PAT"
org = "myorg"
project = "myproject"

with open("work_items.csv") as f:
    reader = csv.DictReader(f)
    for row in reader:
        url = f"https://dev.azure.com/{org}/{project}/_apis/wit/workitems/$Task?api-version=7.0"
        body = [
            {"op": "add", "path": "/fields/System.Title", "value": row["title"]},
            {"op": "add", "path": "/fields/System.Description", "value": row["description"]},
            {"op": "add", "path": "/fields/System.AssignedTo", "value": row.get("assignee", "")},
        ]
        resp = requests.post(
            url,
            json=body,
            auth=HTTPBasicAuth("", pat),
            headers={"Content-Type": "application/json-patch+json"}
        )
        if resp.ok:
            print(f"Created: {row['title']} (ID: {resp.json()['id']})")
        else:
            print(f"Failed: {row['title']} - {resp.text}")
```
