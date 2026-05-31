> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Deep-dive learning guide (this file) |
| [basics_core.md](basics_core.md) | Core basics questions |
| [os_subprocess_apis_advanced.md](os_subprocess_apis_advanced.md) | OS, subprocess, APIs & advanced |
| [answers.md](answers.md) | All answers |

---

# Python — Deep-Dive Learning Guide (DevOps Focus)

---

## 1. Python Fundamentals for DevOps

### Data Types & Structures

```python
# ─── Strings ───
name = "DevOps"
f"Hello {name}, version {2+1}"   # f-string (3.6+)
"path/to/file".split("/")        # ['path', 'to', 'file']
"/".join(["path", "to", "file"]) # 'path/to/file'
"  spaces  ".strip()             # 'spaces'

# ─── Lists (mutable, ordered) ───
servers = ["web1", "web2", "db1"]
servers.append("web3")           # Add to end
servers.extend(["lb1", "lb2"])   # Add multiple
servers.remove("db1")            # Remove by value
servers.pop(0)                   # Remove by index, returns it
[s for s in servers if s.startswith("web")]  # List comprehension

# ─── Dictionaries (key-value, ordered 3.7+) ───
config = {"host": "localhost", "port": 8080, "debug": True}
config.get("timeout", 30)       # Default if missing (no KeyError)
config.update({"port": 9090})   # Update/add keys
{k: v for k, v in config.items() if isinstance(v, int)}  # Dict comprehension

# ─── Sets (unique, unordered) ───
live = {"web1", "web2", "web3"}
expected = {"web1", "web2", "web3", "web4"}
expected - live                  # {'web4'} — missing servers!

# ─── Tuples (immutable) ───
point = (10, 20)                 # Can't modify after creation
x, y = point                    # Unpacking
```

### Control Flow

```python
# ─── Conditional ───
if status == 200:
    print("OK")
elif status == 404:
    print("Not Found")
else:
    print(f"Error: {status}")

# ─── Ternary ───
env = "prod" if is_production else "dev"

# ─── Loops ───
for server in servers:
    deploy(server)

for i, server in enumerate(servers):   # Index + value
    print(f"{i}: {server}")

for key, value in config.items():      # Dict iteration
    print(f"{key}={value}")

# ─── While (careful: always ensure exit condition!) ───
retries = 3
while retries > 0:
    if health_check():
        break
    retries -= 1
```

---

## 2. Functions & Error Handling

```python
# ─── Functions ───
def deploy(service: str, env: str = "staging", dry_run: bool = False) -> bool:
    """Deploy a service to the specified environment."""
    if dry_run:
        print(f"[DRY RUN] Would deploy {service} to {env}")
        return True
    # ... actual deploy logic
    return True

# ─── *args, **kwargs ───
def run_command(cmd, *args, **kwargs):
    """Run command with variable arguments."""
    timeout = kwargs.get("timeout", 30)
    subprocess.run([cmd, *args], timeout=timeout)

# ─── Lambda (anonymous functions) ───
sorted(servers, key=lambda s: s.split("-")[1])

# ─── Error Handling ───
import subprocess

try:
    result = subprocess.run(["kubectl", "get", "pods"],
                          capture_output=True, text=True, check=True)
    print(result.stdout)
except subprocess.CalledProcessError as e:
    print(f"Command failed with exit code {e.returncode}")
    print(f"stderr: {e.stderr}")
except FileNotFoundError:
    print("kubectl not found in PATH")
except Exception as e:
    print(f"Unexpected error: {type(e).__name__}: {e}")
finally:
    print("Cleanup done")

# ─── Custom Exception ───
class DeploymentError(Exception):
    def __init__(self, service, env, message):
        self.service = service
        self.env = env
        super().__init__(f"Failed to deploy {service} to {env}: {message}")
```

---

## 3. File Operations (Critical for DevOps)

```python
from pathlib import Path
import json
import yaml   # pip install pyyaml

# ─── pathlib (modern, preferred) ───
config_dir = Path("/etc/myapp")
config_file = config_dir / "config.yaml"     # / operator joins paths!

config_file.exists()                          # True/False
config_file.is_file()                         # True/False
config_dir.mkdir(parents=True, exist_ok=True) # mkdir -p
list(config_dir.glob("*.yaml"))               # Find files
config_file.read_text()                       # Read entire file
config_file.write_text("key: value\n")        # Write file

# ─── Context managers (auto-close) ───
with open("servers.txt", "r") as f:
    servers = [line.strip() for line in f if line.strip()]

with open("output.log", "a") as f:     # Append mode
    f.write(f"Deployed at {datetime.now()}\n")

# ─── JSON ───
with open("config.json") as f:
    config = json.load(f)              # Parse JSON → dict

with open("output.json", "w") as f:
    json.dump(config, f, indent=2)     # Dict → JSON file

# ─── YAML ───
with open("deployment.yaml") as f:
    manifest = yaml.safe_load(f)       # Parse YAML → dict

with open("output.yaml", "w") as f:
    yaml.dump(manifest, f, default_flow_style=False)

# ─── INI/Config files ───
import configparser
config = configparser.ConfigParser()
config.read("app.ini")
db_host = config["database"]["host"]
```

---

## 4. subprocess — Running Shell Commands

```python
import subprocess

# ─── Simple command ───
result = subprocess.run(
    ["docker", "ps", "--format", "{{.Names}}"],
    capture_output=True,    # Capture stdout + stderr
    text=True,              # Return strings (not bytes)
    check=True,             # Raise on non-zero exit
    timeout=30              # Kill if too slow
)
print(result.stdout)        # Container names
print(result.returncode)    # 0

# ─── Shell commands (pipes, redirects) ───
result = subprocess.run(
    "kubectl get pods | grep -v Running",
    shell=True,             # SECURITY RISK: only with trusted input!
    capture_output=True, text=True
)

# ─── Streaming output (long-running commands) ───
process = subprocess.Popen(
    ["docker", "build", "-t", "myapp", "."],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True
)
for line in process.stdout:
    print(line, end="")     # Print as it runs
process.wait()
if process.returncode != 0:
    raise Exception("Build failed!")

# ─── Safe command building (NEVER use f-strings with shell=True!) ───
# BAD:  subprocess.run(f"docker rm {user_input}", shell=True)  # INJECTION!
# GOOD: subprocess.run(["docker", "rm", user_input])           # Safe list form
```

---

## 5. HTTP Requests (APIs)

```python
import requests   # pip install requests

# ─── GET ───
response = requests.get(
    "https://api.github.com/repos/kubernetes/kubernetes",
    headers={"Authorization": f"token {os.environ['GITHUB_TOKEN']}"},
    timeout=10
)
response.raise_for_status()    # Raise on 4xx/5xx
data = response.json()         # Parse JSON response
print(data["stargazers_count"])

# ─── POST ───
response = requests.post(
    "https://hooks.slack.com/services/xxx",
    json={"text": "Deployment complete!"},   # Auto JSON-encode
    timeout=10
)

# ─── Session (reuse connections + headers) ───
session = requests.Session()
session.headers.update({"Authorization": f"Bearer {token}"})

for endpoint in ["/pods", "/services", "/deployments"]:
    resp = session.get(f"{api_base}{endpoint}")
    print(resp.json())

# ─── Retry with backoff ───
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retries = Retry(total=3, backoff_factor=1, status_forcelist=[500, 502, 503])
session.mount("https://", HTTPAdapter(max_retries=retries))
```

---

## 6. OOP for DevOps Scripts

```python
from dataclasses import dataclass, field
from typing import Optional

# ─── Dataclass (modern, less boilerplate) ───
@dataclass
class Server:
    hostname: str
    ip: str
    environment: str = "staging"
    tags: list = field(default_factory=list)

    @property
    def fqdn(self) -> str:
        return f"{self.hostname}.{self.environment}.internal"

    def health_check(self) -> bool:
        try:
            resp = requests.get(f"http://{self.ip}:8080/health", timeout=5)
            return resp.status_code == 200
        except requests.RequestException:
            return False

# Usage
server = Server("web1", "10.0.1.5", tags=["frontend", "critical"])
print(server.fqdn)       # web1.staging.internal
print(server.health_check())

# ─── Traditional class ───
class DeploymentManager:
    def __init__(self, cluster: str, namespace: str = "default"):
        self.cluster = cluster
        self.namespace = namespace
        self._client = None     # Lazy init

    def deploy(self, image: str, replicas: int = 3) -> bool:
        """Deploy image to cluster."""
        cmd = ["kubectl", "--context", self.cluster,
               "-n", self.namespace,
               "set", "image", f"deployment/app", f"app={image}"]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0

    def __repr__(self):
        return f"DeploymentManager(cluster={self.cluster!r})"
```

---

## 7. Key Python Modules for DevOps

```
┌─── Standard Library (no install needed) ────────────────────┐
│                                                              │
│  os         → env vars, file ops, process info              │
│  sys        → CLI args, exit, path                          │
│  pathlib    → modern file path handling                      │
│  subprocess → run shell commands                             │
│  json       → parse/generate JSON                            │
│  shutil     → copy/move/delete files/dirs                   │
│  logging    → structured logging                             │
│  argparse   → CLI argument parsing                           │
│  re         → regex pattern matching                         │
│  os.path    → legacy path operations                         │
│  tempfile   → temporary files/directories                    │
│  hashlib    → SHA256, MD5 checksums                          │
│  socket     → network connections, hostname                  │
│  http.server→ quick HTTP server                              │
│  unittest   → testing framework                              │
│  datetime   → date/time operations                           │
│  threading  → parallel execution                             │
│  concurrent.futures → thread/process pools                   │
└──────────────────────────────────────────────────────────────┘

┌─── Third-Party (pip install) ───────────────────────────────┐
│                                                              │
│  requests    → HTTP client                                  │
│  pyyaml      → YAML parsing                                 │
│  boto3       → AWS SDK                                      │
│  azure-*     → Azure SDK                                    │
│  kubernetes  → K8s Python client                            │
│  docker      → Docker SDK                                   │
│  paramiko    → SSH client                                   │
│  click/typer → CLI frameworks                               │
│  jinja2      → Template engine                              │
│  pytest      → Testing framework                            │
│  python-dotenv → Load .env files                            │
└──────────────────────────────────────────────────────────────┘
```

---

## 8. Logging (Not print!)

```python
import logging

# ─── Setup ───
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)
logger = logging.getLogger("deploy")

# ─── Usage ───
logger.debug("Connecting to cluster...")        # Verbose detail
logger.info("Deploying myapp:v2 to production") # Normal operation
logger.warning("Retrying connection (2/3)")     # Potential issue
logger.error("Deployment failed: timeout")      # Something broke
logger.critical("Cluster unreachable!")          # System down

# ─── Structured logging with extra data ───
logger.info("Deploy complete", extra={"service": "web", "version": "v2", "duration": 45})
```

| Level | When to Use |
|-------|------------|
| DEBUG | Detailed diagnostic info (dev only) |
| INFO | Normal operations ("deployed X to Y") |
| WARNING | Something unexpected but handled |
| ERROR | Something failed |
| CRITICAL | System-level failure |

---

## 9. Virtual Environments

```bash
# ─── venv (standard library) ───
python -m venv .venv
source .venv/bin/activate      # Linux/Mac
.venv\Scripts\Activate.ps1     # Windows PowerShell
pip install -r requirements.txt
deactivate

# ─── requirements.txt ───
requests==2.31.0               # Pinned version
pyyaml>=6.0,<7.0               # Range
boto3~=1.28                    # Compatible release (1.28.x)
```

```
Why virtual environments?
  Project A needs requests==2.28
  Project B needs requests==2.31
  Without venv → conflict!
  With venv → each project has own isolated packages
```

---

## 10. Common DevOps Scripts

### Health Check Script

```python
#!/usr/bin/env python3
"""Check health of services and alert on failures."""
import requests
import sys

SERVICES = {
    "API": "https://api.example.com/health",
    "Web": "https://www.example.com",
    "DB": "https://db-proxy.example.com/status",
}

def check_services():
    failures = []
    for name, url in SERVICES.items():
        try:
            resp = requests.get(url, timeout=5)
            if resp.status_code != 200:
                failures.append(f"{name}: HTTP {resp.status_code}")
        except requests.RequestException as e:
            failures.append(f"{name}: {e}")
    return failures

if __name__ == "__main__":
    failures = check_services()
    if failures:
        print("FAILURES:")
        for f in failures:
            print(f"  ✗ {f}")
        sys.exit(1)
    print("All services healthy ✓")
```

### YAML Config Updater

```python
#!/usr/bin/env python3
"""Update image tag in K8s deployment YAML."""
import yaml
import sys

def update_image(yaml_file, new_tag):
    with open(yaml_file) as f:
        manifest = yaml.safe_load(f)

    containers = manifest["spec"]["template"]["spec"]["containers"]
    for container in containers:
        image = container["image"]
        repo = image.rsplit(":", 1)[0]      # Split off old tag
        container["image"] = f"{repo}:{new_tag}"

    with open(yaml_file, "w") as f:
        yaml.dump(manifest, f, default_flow_style=False)

    print(f"Updated {yaml_file} to tag {new_tag}")

if __name__ == "__main__":
    update_image(sys.argv[1], sys.argv[2])
```

---

## 11. Decorators & Context Managers

```python
import functools
import time

# ─── Decorator: retry with backoff ───
def retry(max_retries=3, backoff=2):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    wait = backoff ** attempt
                    print(f"Retry {attempt+1}/{max_retries} in {wait}s: {e}")
                    time.sleep(wait)
        return wrapper
    return decorator

@retry(max_retries=3, backoff=2)
def deploy(service):
    # May fail transiently
    requests.post(f"https://deploy.example.com/{service}")

# ─── Context manager ───
from contextlib import contextmanager

@contextmanager
def timer(label):
    start = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start
        print(f"{label}: {elapsed:.2f}s")

with timer("Deploy"):
    deploy("web-service")
# Output: Deploy: 3.45s
```

---

## 12. Testing with pytest

```python
# test_deploy.py
import pytest

def test_health_check_success(mocker):
    mock_resp = mocker.patch("requests.get")
    mock_resp.return_value.status_code = 200

    assert health_check("web1") is True

def test_health_check_failure(mocker):
    mock_resp = mocker.patch("requests.get")
    mock_resp.side_effect = requests.ConnectionError("refused")

    assert health_check("web1") is False

@pytest.mark.parametrize("input,expected", [
    ("v1.2.3", (1, 2, 3)),
    ("v0.1.0", (0, 1, 0)),
])
def test_parse_version(input, expected):
    assert parse_version(input) == expected

@pytest.fixture
def temp_config(tmp_path):
    config = tmp_path / "config.yaml"
    config.write_text("port: 8080\nenv: test\n")
    return config
```

```bash
pytest                          # Run all tests
pytest -v                       # Verbose
pytest -x                       # Stop on first failure
pytest -k "test_deploy"         # Run matching tests
pytest --cov=src                # Coverage report
```
