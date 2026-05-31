# Python Practice Problems — Platform Engineering Focus

> You know Python but need practice. These problems are specific to what
> you'll face in the REA coding round: platform tooling, automation,
> API interaction, log analysis, K8s/YAML manipulation, and HTTP servers.

---

## TABLE OF CONTENTS

1. [YAML/JSON Configuration Management](#1-yaml-json)
2. [Log Parsing & Analysis](#2-log-parsing)
3. [HTTP API Interaction](#3-http-api)
4. [Kubernetes Automation](#4-k8s)
5. [File System & Process Management](#5-filesystem)
6. [Simple HTTP Servers & Health Checks](#6-http-servers)
7. [CLI Tool Building](#7-cli)
8. [Testing & Error Handling](#8-testing)
9. [Data Structures for Platform Problems](#9-data-structures)
10. [Concurrency & Async](#10-concurrency)

---

## 1. YAML/JSON CONFIGURATION MANAGEMENT <a name="1-yaml-json"></a>

### Problem 1.1: Merge Environment Configs
**Task**: Write a function that merges a base config with environment-specific overrides.

```python
import yaml
import json
from copy import deepcopy

def deep_merge(base: dict, override: dict) -> dict:
    """Deep merge override into base. Override wins on conflicts."""
    result = deepcopy(base)
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = deepcopy(value)
    return result

# Test it:
base = {
    "app": {"name": "property-api", "port": 8080, "debug": False},
    "database": {"host": "db.internal", "port": 5432},
    "logging": {"level": "INFO"}
}
staging_override = {
    "app": {"debug": True},
    "database": {"host": "staging-db.internal"},
    "logging": {"level": "DEBUG"}
}

merged = deep_merge(base, staging_override)
assert merged["app"]["name"] == "property-api"  # Kept from base
assert merged["app"]["debug"] == True  # Overridden
assert merged["database"]["host"] == "staging-db.internal"  # Overridden
assert merged["database"]["port"] == 5432  # Kept from base
print("✅ Test passed:", json.dumps(merged, indent=2))
```

### Problem 1.2: Validate Kubernetes YAML
**Task**: Write a validator that checks a K8s Deployment YAML for common issues.

```python
import yaml

def validate_deployment(yaml_content: str) -> list[str]:
    """Validate a K8s Deployment YAML and return list of issues."""
    issues = []
    try:
        doc = yaml.safe_load(yaml_content)
    except yaml.YAMLError as e:
        return [f"Invalid YAML: {e}"]
    
    if not isinstance(doc, dict):
        return ["Document is not a YAML mapping"]
    
    # Check apiVersion
    if "apiVersion" not in doc:
        issues.append("Missing apiVersion")
    
    # Check kind
    kind = doc.get("kind", "")
    if kind != "Deployment":
        issues.append(f"Expected kind 'Deployment', got '{kind}'")
    
    # Check metadata
    metadata = doc.get("metadata", {})
    if not metadata.get("name"):
        issues.append("Missing metadata.name")
    if not metadata.get("namespace"):
        issues.append("Warning: No namespace specified (will use 'default')")
    
    # Check spec
    spec = doc.get("spec", {})
    if not spec:
        issues.append("Missing spec")
        return issues
    
    # Check replicas
    replicas = spec.get("replicas", 1)
    if replicas < 2:
        issues.append(f"Warning: replicas={replicas}. Consider >=2 for HA")
    
    # Check template
    template = spec.get("template", {}).get("spec", {})
    containers = template.get("containers", [])
    
    if not containers:
        issues.append("No containers defined")
        return issues
    
    for i, container in enumerate(containers):
        prefix = f"container[{i}]({container.get('name', 'unnamed')})"
        
        # Check image tag
        image = container.get("image", "")
        if ":latest" in image or ":" not in image:
            issues.append(f"{prefix}: Using 'latest' or no tag — pin a specific version")
        
        # Check resource limits
        resources = container.get("resources", {})
        if not resources.get("limits"):
            issues.append(f"{prefix}: No resource limits — can cause OOM/noisy neighbor")
        if not resources.get("requests"):
            issues.append(f"{prefix}: No resource requests — scheduler can't make good decisions")
        
        # Check liveness/readiness probes
        if not container.get("livenessProbe"):
            issues.append(f"{prefix}: No livenessProbe — K8s won't restart stuck containers")
        if not container.get("readinessProbe"):
            issues.append(f"{prefix}: No readinessProbe — traffic may go to unready pods")
        
        # Check security context
        sec = container.get("securityContext", {})
        if sec.get("runAsRoot", False) or sec.get("privileged", False):
            issues.append(f"{prefix}: Running as root or privileged — security risk")
    
    return issues

# Test:
test_yaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: property-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: property-api
  template:
    metadata:
      labels:
        app: property-api
    spec:
      containers:
      - name: api
        image: property-api:latest
        ports:
        - containerPort: 8080
"""

issues = validate_deployment(test_yaml)
for issue in issues:
    print(f"  ⚠️  {issue}")
# Expected issues: no namespace, replicas=1, latest tag, no resources, no probes
```

### Problem 1.3: Generate K8s Manifests from Template
**Task**: Given a service definition, generate Deployment + Service + Ingress YAML.

```python
import yaml

def generate_k8s_manifests(
    name: str,
    image: str,
    port: int,
    replicas: int = 3,
    namespace: str = "production",
    cpu_request: str = "100m",
    cpu_limit: str = "500m",
    memory_request: str = "128Mi",
    memory_limit: str = "512Mi",
    host: str = None,
) -> str:
    """Generate K8s Deployment + Service + Ingress YAML."""
    
    labels = {"app": name, "team": "platform"}
    
    deployment = {
        "apiVersion": "apps/v1",
        "kind": "Deployment",
        "metadata": {"name": name, "namespace": namespace, "labels": labels},
        "spec": {
            "replicas": replicas,
            "selector": {"matchLabels": {"app": name}},
            "template": {
                "metadata": {"labels": labels},
                "spec": {
                    "containers": [{
                        "name": name,
                        "image": image,
                        "ports": [{"containerPort": port}],
                        "resources": {
                            "requests": {"cpu": cpu_request, "memory": memory_request},
                            "limits": {"cpu": cpu_limit, "memory": memory_limit},
                        },
                        "livenessProbe": {
                            "httpGet": {"path": "/healthz", "port": port},
                            "initialDelaySeconds": 10,
                            "periodSeconds": 10,
                        },
                        "readinessProbe": {
                            "httpGet": {"path": "/readyz", "port": port},
                            "initialDelaySeconds": 5,
                            "periodSeconds": 5,
                        },
                    }],
                },
            },
        },
    }
    
    service = {
        "apiVersion": "v1",
        "kind": "Service",
        "metadata": {"name": name, "namespace": namespace, "labels": labels},
        "spec": {
            "selector": {"app": name},
            "ports": [{"port": 80, "targetPort": port, "protocol": "TCP"}],
            "type": "ClusterIP",
        },
    }
    
    docs = [deployment, service]
    
    if host:
        ingress = {
            "apiVersion": "networking.k8s.io/v1",
            "kind": "Ingress",
            "metadata": {
                "name": f"{name}-ingress",
                "namespace": namespace,
                "annotations": {
                    "kubernetes.io/ingress.class": "alb",
                    "alb.ingress.kubernetes.io/scheme": "internet-facing",
                },
            },
            "spec": {
                "rules": [{
                    "host": host,
                    "http": {
                        "paths": [{
                            "path": "/",
                            "pathType": "Prefix",
                            "backend": {
                                "service": {"name": name, "port": {"number": 80}},
                            },
                        }],
                    },
                }],
            },
        }
        docs.append(ingress)
    
    return "---\n".join(yaml.dump(doc, default_flow_style=False) for doc in docs)

# Test:
manifests = generate_k8s_manifests(
    name="property-api",
    image="123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/property-api:v2.1.0",
    port=8080,
    host="api.rea.com",
)
print(manifests)
```

---

## 2. LOG PARSING & ANALYSIS <a name="2-log-parsing"></a>

### Problem 2.1: Parse Nginx Access Logs
**Task**: Parse nginx access logs and compute request statistics.

```python
import re
from collections import defaultdict, Counter
from datetime import datetime

# Nginx combined log format:
# 10.0.1.5 - - [15/Jan/2024:10:30:45 +0000] "GET /api/properties HTTP/1.1" 200 1234 "https://rea.com" "Mozilla/5.0"

NGINX_LOG_PATTERN = re.compile(
    r'(?P<ip>[\d.]+)\s+-\s+-\s+'
    r'\[(?P<timestamp>[^\]]+)\]\s+'
    r'"(?P<method>\w+)\s+(?P<path>\S+)\s+HTTP/[\d.]+"\s+'
    r'(?P<status>\d+)\s+'
    r'(?P<bytes>\d+)\s+'
    r'"(?P<referer>[^"]*)"\s+'
    r'"(?P<useragent>[^"]*)"'
)

def parse_nginx_logs(log_lines: list[str]) -> dict:
    """Parse nginx logs and return statistics."""
    stats = {
        "total_requests": 0,
        "status_counts": Counter(),
        "top_paths": Counter(),
        "top_ips": Counter(),
        "error_paths": Counter(),
        "total_bytes": 0,
        "methods": Counter(),
        "errors": [],
    }
    
    for line in log_lines:
        match = NGINX_LOG_PATTERN.match(line.strip())
        if not match:
            continue
        
        data = match.groupdict()
        stats["total_requests"] += 1
        stats["status_counts"][data["status"]] += 1
        stats["top_paths"][data["path"]] += 1
        stats["top_ips"][data["ip"]] += 1
        stats["total_bytes"] += int(data["bytes"])
        stats["methods"][data["method"]] += 1
        
        status = int(data["status"])
        if status >= 500:
            stats["error_paths"][data["path"]] += 1
            stats["errors"].append({
                "time": data["timestamp"],
                "path": data["path"],
                "status": status,
                "ip": data["ip"],
            })
    
    # Compute summary
    total = stats["total_requests"]
    errors = sum(v for k, v in stats["status_counts"].items() if int(k) >= 500)
    stats["error_rate"] = round(errors / total * 100, 2) if total > 0 else 0
    stats["bytes_mb"] = round(stats["total_bytes"] / 1048576, 2)
    
    return stats

def print_report(stats: dict):
    """Print a human-readable report."""
    print(f"\n{'='*50}")
    print(f"NGINX LOG ANALYSIS REPORT")
    print(f"{'='*50}")
    print(f"Total Requests: {stats['total_requests']}")
    print(f"Error Rate: {stats['error_rate']}%")
    print(f"Total Data: {stats['bytes_mb']} MB")
    
    print(f"\nStatus Code Distribution:")
    for code, count in stats["status_counts"].most_common():
        pct = round(count / stats["total_requests"] * 100, 1)
        print(f"  {code}: {count} ({pct}%)")
    
    print(f"\nTop 5 Paths:")
    for path, count in stats["top_paths"].most_common(5):
        print(f"  {path}: {count}")
    
    print(f"\nTop Error Paths:")
    for path, count in stats["error_paths"].most_common(5):
        print(f"  {path}: {count}")
    
    print(f"\nTop IPs:")
    for ip, count in stats["top_ips"].most_common(5):
        print(f"  {ip}: {count}")

# Test with sample data:
sample_logs = [
    '10.0.1.5 - - [15/Jan/2024:10:30:45 +0000] "GET /api/properties HTTP/1.1" 200 1234 "-" "Mozilla/5.0"',
    '10.0.1.6 - - [15/Jan/2024:10:30:46 +0000] "GET /api/properties/123 HTTP/1.1" 200 567 "-" "curl/7.68"',
    '10.0.1.5 - - [15/Jan/2024:10:30:47 +0000] "POST /api/search HTTP/1.1" 500 89 "-" "Mozilla/5.0"',
    '10.0.1.7 - - [15/Jan/2024:10:30:48 +0000] "GET /api/properties HTTP/1.1" 200 1234 "-" "Mozilla/5.0"',
    '10.0.1.8 - - [15/Jan/2024:10:30:49 +0000] "GET /healthz HTTP/1.1" 200 2 "-" "kube-probe/1.29"',
    '10.0.1.5 - - [15/Jan/2024:10:30:50 +0000] "GET /api/properties HTTP/1.1" 503 45 "-" "Mozilla/5.0"',
]
stats = parse_nginx_logs(sample_logs)
print_report(stats)
```

### Problem 2.2: Detect Anomalies in Logs
**Task**: Detect error rate spikes by comparing current window to baseline.

```python
from collections import defaultdict
from datetime import datetime, timedelta

def detect_error_spikes(
    events: list[dict],  # [{"timestamp": datetime, "status": int}, ...]
    window_minutes: int = 5,
    spike_threshold: float = 3.0,  # 3x baseline = spike
) -> list[dict]:
    """Detect windows where error rate exceeds baseline by threshold."""
    
    # Bucket events into windows
    if not events:
        return []
    
    min_time = min(e["timestamp"] for e in events)
    max_time = max(e["timestamp"] for e in events)
    window = timedelta(minutes=window_minutes)
    
    buckets = defaultdict(lambda: {"total": 0, "errors": 0})
    
    for event in events:
        bucket_key = min_time + window * int((event["timestamp"] - min_time) / window)
        buckets[bucket_key]["total"] += 1
        if event["status"] >= 500:
            buckets[bucket_key]["errors"] += 1
    
    # Calculate baseline error rate
    total_errors = sum(b["errors"] for b in buckets.values())
    total_requests = sum(b["total"] for b in buckets.values())
    baseline_rate = total_errors / total_requests if total_requests > 0 else 0
    
    # Find spikes
    spikes = []
    for time_bucket, counts in sorted(buckets.items()):
        if counts["total"] == 0:
            continue
        bucket_rate = counts["errors"] / counts["total"]
        if baseline_rate > 0 and bucket_rate > baseline_rate * spike_threshold:
            spikes.append({
                "window_start": time_bucket,
                "error_rate": round(bucket_rate * 100, 2),
                "baseline_rate": round(baseline_rate * 100, 2),
                "multiplier": round(bucket_rate / baseline_rate, 1),
                "errors": counts["errors"],
                "total": counts["total"],
            })
    
    return spikes

# Test
now = datetime.now()
events = []
# Normal traffic (low error rate)
for i in range(100):
    events.append({"timestamp": now - timedelta(minutes=30-i*0.3), "status": 200})
for i in range(2):
    events.append({"timestamp": now - timedelta(minutes=25), "status": 500})
# Spike window
for i in range(20):
    events.append({"timestamp": now - timedelta(minutes=10), "status": 500})
for i in range(30):
    events.append({"timestamp": now - timedelta(minutes=10), "status": 200})

spikes = detect_error_spikes(events)
for spike in spikes:
    print(f"🔥 Spike at {spike['window_start']}: {spike['error_rate']}% error rate ({spike['multiplier']}x baseline)")
```

---

## 3. HTTP API INTERACTION <a name="3-http-api"></a>

### Problem 3.1: Health Check Monitor
**Task**: Build a concurrent health checker for multiple services.

```python
import requests
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass

@dataclass
class HealthResult:
    service: str
    url: str
    status: str  # "healthy", "unhealthy", "timeout", "error"
    status_code: int | None
    response_time_ms: float
    error: str | None = None

def check_health(service: str, url: str, timeout: int = 5) -> HealthResult:
    """Check health of a single service."""
    start = time.time()
    try:
        resp = requests.get(url, timeout=timeout)
        elapsed = (time.time() - start) * 1000
        status = "healthy" if resp.status_code == 200 else "unhealthy"
        return HealthResult(
            service=service, url=url, status=status,
            status_code=resp.status_code, response_time_ms=round(elapsed, 1)
        )
    except requests.exceptions.Timeout:
        elapsed = (time.time() - start) * 1000
        return HealthResult(
            service=service, url=url, status="timeout",
            status_code=None, response_time_ms=round(elapsed, 1),
            error="Request timed out"
        )
    except requests.exceptions.ConnectionError as e:
        elapsed = (time.time() - start) * 1000
        return HealthResult(
            service=service, url=url, status="error",
            status_code=None, response_time_ms=round(elapsed, 1),
            error=str(e)[:100]
        )

def check_all_services(services: dict[str, str], max_workers: int = 10) -> list[HealthResult]:
    """Check all services concurrently."""
    results = []
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = {
            executor.submit(check_health, name, url): name
            for name, url in services.items()
        }
        for future in as_completed(futures):
            results.append(future.result())
    return sorted(results, key=lambda r: r.service)

def print_health_report(results: list[HealthResult]):
    """Print health check results."""
    print(f"\n{'Service':<25} {'Status':<12} {'Code':<6} {'Time (ms)':<10}")
    print("-" * 55)
    for r in results:
        icon = "✅" if r.status == "healthy" else "❌"
        code = str(r.status_code) if r.status_code else "N/A"
        print(f"{icon} {r.service:<23} {r.status:<12} {code:<6} {r.response_time_ms:<10}")
        if r.error:
            print(f"   └── Error: {r.error}")

# Usage:
services = {
    "property-api": "https://httpbin.org/status/200",
    "search-service": "https://httpbin.org/delay/1",
    "auth-service": "https://httpbin.org/status/500",
    "image-service": "https://nonexistent.example.com/health",
}
results = check_all_services(services)
print_health_report(results)
```

### Problem 3.2: REST API Client with Retry
**Task**: Build a resilient API client with exponential backoff.

```python
import requests
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class APIClient:
    """Resilient API client with retry, backoff, and circuit breaker."""
    
    def __init__(self, base_url: str, max_retries: int = 3, timeout: int = 10):
        self.base_url = base_url.rstrip("/")
        self.max_retries = max_retries
        self.timeout = timeout
        self.session = requests.Session()
        self.session.headers.update({"Content-Type": "application/json"})
    
    def _request(self, method: str, path: str, **kwargs) -> requests.Response:
        """Make request with exponential backoff retry."""
        url = f"{self.base_url}/{path.lstrip('/')}"
        kwargs.setdefault("timeout", self.timeout)
        
        last_exception = None
        for attempt in range(self.max_retries + 1):
            try:
                response = self.session.request(method, url, **kwargs)
                
                # Don't retry client errors (4xx), only server errors (5xx)
                if response.status_code < 500:
                    return response
                
                logger.warning(f"Server error {response.status_code} on {method} {path} (attempt {attempt + 1})")
                last_exception = Exception(f"HTTP {response.status_code}")
                
            except (requests.exceptions.ConnectionError, requests.exceptions.Timeout) as e:
                logger.warning(f"Request failed: {e} (attempt {attempt + 1})")
                last_exception = e
            
            if attempt < self.max_retries:
                wait = (2 ** attempt) + (0.1 * attempt)  # Exponential backoff
                logger.info(f"Retrying in {wait:.1f}s...")
                time.sleep(wait)
        
        raise last_exception
    
    def get(self, path: str, **kwargs) -> requests.Response:
        return self._request("GET", path, **kwargs)
    
    def post(self, path: str, **kwargs) -> requests.Response:
        return self._request("POST", path, **kwargs)
    
    def put(self, path: str, **kwargs) -> requests.Response:
        return self._request("PUT", path, **kwargs)
    
    def delete(self, path: str, **kwargs) -> requests.Response:
        return self._request("DELETE", path, **kwargs)

# Usage:
# client = APIClient("https://api.rea-property.com")
# properties = client.get("/api/v1/properties?city=mumbai").json()
```

---

## 4. KUBERNETES AUTOMATION <a name="4-k8s"></a>

### Problem 4.1: Pod Status Monitor
**Task**: Query Kubernetes API to get pod health status.

```python
"""
Kubernetes pod status monitor using the kubernetes Python client.
pip install kubernetes
"""

# Note: This shows the PATTERN. You may not have a cluster to test on.
# But knowing this API is important for the interview.

from kubernetes import client, config

def get_pod_status(namespace: str = "default") -> list[dict]:
    """Get status of all pods in a namespace."""
    # Load kubeconfig (or in-cluster config)
    try:
        config.load_kube_config()
    except:
        config.load_incluster_config()
    
    v1 = client.CoreV1Api()
    pods = v1.list_namespaced_pod(namespace)
    
    results = []
    for pod in pods.items:
        pod_info = {
            "name": pod.metadata.name,
            "namespace": pod.metadata.namespace,
            "phase": pod.status.phase,
            "ready": False,
            "restarts": 0,
            "containers": [],
            "issues": [],
        }
        
        # Check container statuses
        if pod.status.container_statuses:
            for cs in pod.status.container_statuses:
                container = {
                    "name": cs.name,
                    "ready": cs.ready,
                    "restarts": cs.restart_count,
                    "state": "unknown",
                }
                
                if cs.state.running:
                    container["state"] = "running"
                elif cs.state.waiting:
                    container["state"] = f"waiting: {cs.state.waiting.reason}"
                    pod_info["issues"].append(f"{cs.name}: {cs.state.waiting.reason}")
                elif cs.state.terminated:
                    container["state"] = f"terminated: {cs.state.terminated.reason}"
                    pod_info["issues"].append(f"{cs.name}: {cs.state.terminated.reason}")
                
                pod_info["containers"].append(container)
                pod_info["restarts"] += cs.restart_count
            
            pod_info["ready"] = all(cs.ready for cs in pod.status.container_statuses)
        
        results.append(pod_info)
    
    return results

def find_unhealthy_pods(namespace: str = "default") -> list[dict]:
    """Find pods that are not healthy."""
    all_pods = get_pod_status(namespace)
    return [p for p in all_pods if not p["ready"] or p["restarts"] > 5 or p["issues"]]

# Pure Python version (no kubernetes client needed) — uses subprocess
import subprocess
import json

def get_pods_via_kubectl(namespace: str = "default") -> list[dict]:
    """Get pod info using kubectl (no Python client needed)."""
    result = subprocess.run(
        ["kubectl", "get", "pods", "-n", namespace, "-o", "json"],
        capture_output=True, text=True, timeout=30
    )
    if result.returncode != 0:
        raise RuntimeError(f"kubectl failed: {result.stderr}")
    
    data = json.loads(result.stdout)
    pods = []
    for item in data.get("items", []):
        pod = {
            "name": item["metadata"]["name"],
            "namespace": item["metadata"]["namespace"],
            "phase": item["status"].get("phase", "Unknown"),
            "restarts": sum(
                cs.get("restartCount", 0)
                for cs in item["status"].get("containerStatuses", [])
            ),
        }
        pods.append(pod)
    return pods
```

### Problem 4.2: Namespace Resource Report
**Task**: Generate a resource usage report for all namespaces.

```python
import subprocess
import json

def get_namespace_resources() -> list[dict]:
    """Get resource requests/limits summary per namespace."""
    result = subprocess.run(
        ["kubectl", "get", "pods", "--all-namespaces", "-o", "json"],
        capture_output=True, text=True, timeout=30
    )
    if result.returncode != 0:
        raise RuntimeError(f"kubectl failed: {result.stderr}")
    
    data = json.loads(result.stdout)
    ns_resources = {}
    
    for pod in data.get("items", []):
        ns = pod["metadata"]["namespace"]
        if ns not in ns_resources:
            ns_resources[ns] = {
                "pods": 0, "containers": 0,
                "cpu_requests_m": 0, "cpu_limits_m": 0,
                "memory_requests_mi": 0, "memory_limits_mi": 0,
            }
        
        ns_resources[ns]["pods"] += 1
        
        for container in pod["spec"].get("containers", []):
            ns_resources[ns]["containers"] += 1
            resources = container.get("resources", {})
            
            # Parse CPU (e.g., "100m" or "0.5")
            cpu_req = resources.get("requests", {}).get("cpu", "0")
            cpu_lim = resources.get("limits", {}).get("cpu", "0")
            ns_resources[ns]["cpu_requests_m"] += parse_cpu(cpu_req)
            ns_resources[ns]["cpu_limits_m"] += parse_cpu(cpu_lim)
            
            # Parse memory (e.g., "128Mi" or "1Gi")
            mem_req = resources.get("requests", {}).get("memory", "0")
            mem_lim = resources.get("limits", {}).get("memory", "0")
            ns_resources[ns]["memory_requests_mi"] += parse_memory(mem_req)
            ns_resources[ns]["memory_limits_mi"] += parse_memory(mem_lim)
    
    return [{"namespace": ns, **data} for ns, data in sorted(ns_resources.items())]

def parse_cpu(value: str) -> int:
    """Parse CPU value to millicores."""
    if value.endswith("m"):
        return int(value[:-1])
    try:
        return int(float(value) * 1000)
    except ValueError:
        return 0

def parse_memory(value: str) -> int:
    """Parse memory value to MiB."""
    units = {"Ki": 1/1024, "Mi": 1, "Gi": 1024, "Ti": 1048576}
    for unit, multiplier in units.items():
        if value.endswith(unit):
            return int(float(value[:-len(unit)]) * multiplier)
    try:
        return int(int(value) / 1048576)  # Bytes to MiB
    except ValueError:
        return 0
```

---

## 5. FILE SYSTEM & PROCESS MANAGEMENT <a name="5-filesystem"></a>

### Problem 5.1: Certificate Expiry Checker
**Task**: Check SSL certificate expiry for a list of domains.

```python
import ssl
import socket
from datetime import datetime
from concurrent.futures import ThreadPoolExecutor

def check_cert_expiry(hostname: str, port: int = 443) -> dict:
    """Check SSL certificate expiry for a hostname."""
    try:
        context = ssl.create_default_context()
        with socket.create_connection((hostname, port), timeout=5) as sock:
            with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                cert = ssock.getpeercert()
                expiry = datetime.strptime(cert["notAfter"], "%b %d %H:%M:%S %Y %Z")
                days_remaining = (expiry - datetime.utcnow()).days
                return {
                    "hostname": hostname,
                    "expiry": expiry.isoformat(),
                    "days_remaining": days_remaining,
                    "status": "ok" if days_remaining > 30 else ("warning" if days_remaining > 7 else "critical"),
                    "issuer": dict(x[0] for x in cert.get("issuer", [])).get("organizationName", "Unknown"),
                }
    except Exception as e:
        return {
            "hostname": hostname,
            "expiry": None,
            "days_remaining": -1,
            "status": "error",
            "error": str(e)[:100],
        }

def check_all_certs(hostnames: list[str]) -> list[dict]:
    """Check certificates for multiple hostnames concurrently."""
    with ThreadPoolExecutor(max_workers=10) as executor:
        results = list(executor.map(check_cert_expiry, hostnames))
    return sorted(results, key=lambda r: r["days_remaining"])

# Test:
domains = ["google.com", "github.com", "expired.badssl.com"]
results = check_all_certs(domains)
for r in results:
    icon = {"ok": "✅", "warning": "⚠️", "critical": "🔴", "error": "❌"}.get(r["status"], "?")
    print(f"{icon} {r['hostname']}: {r['days_remaining']} days ({r['status']})")
```

### Problem 5.2: Disk Usage Analyzer
**Task**: Find the top space-consuming directories.

```python
import os
from pathlib import Path

def get_dir_sizes(root: str, max_depth: int = 2) -> list[dict]:
    """Get directory sizes up to max_depth."""
    results = []
    root_path = Path(root)
    
    for dirpath, dirnames, filenames in os.walk(root_path):
        depth = len(Path(dirpath).relative_to(root_path).parts)
        if depth > max_depth:
            dirnames.clear()  # Don't recurse deeper
            continue
        
        total_size = 0
        file_count = 0
        for f in filenames:
            fp = os.path.join(dirpath, f)
            try:
                total_size += os.path.getsize(fp)
                file_count += 1
            except (OSError, PermissionError):
                continue
        
        if total_size > 0:
            results.append({
                "path": dirpath,
                "size_bytes": total_size,
                "size_human": human_size(total_size),
                "files": file_count,
            })
    
    return sorted(results, key=lambda x: x["size_bytes"], reverse=True)

def human_size(size_bytes: int) -> str:
    """Convert bytes to human-readable format."""
    for unit in ["B", "KB", "MB", "GB", "TB"]:
        if size_bytes < 1024:
            return f"{size_bytes:.1f} {unit}"
        size_bytes /= 1024
    return f"{size_bytes:.1f} PB"

# Usage:
# top_dirs = get_dir_sizes("/var/log", max_depth=2)[:10]
# for d in top_dirs:
#     print(f"{d['size_human']:>10}  {d['files']:>5} files  {d['path']}")
```

---

## 6. SIMPLE HTTP SERVERS & HEALTH CHECKS <a name="6-http-servers"></a>

### Problem 6.1: Health Check Endpoint Server
**Task**: Build a simple HTTP server with /healthz, /readyz, and /metrics endpoints.

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import time
import threading

# Application state
app_state = {
    "ready": False,
    "start_time": time.time(),
    "request_count": 0,
    "error_count": 0,
}

class HealthHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        app_state["request_count"] += 1
        
        if self.path == "/healthz":
            # Liveness: Is the process alive?
            self.send_json(200, {"status": "ok", "uptime": time.time() - app_state["start_time"]})
        
        elif self.path == "/readyz":
            # Readiness: Is the service ready to accept traffic?
            if app_state["ready"]:
                self.send_json(200, {"status": "ready"})
            else:
                self.send_json(503, {"status": "not ready", "reason": "still initializing"})
        
        elif self.path == "/metrics":
            # Prometheus-style metrics
            metrics = (
                f'# HELP app_requests_total Total requests\n'
                f'# TYPE app_requests_total counter\n'
                f'app_requests_total {app_state["request_count"]}\n'
                f'# HELP app_errors_total Total errors\n'
                f'# TYPE app_errors_total counter\n'
                f'app_errors_total {app_state["error_count"]}\n'
                f'# HELP app_uptime_seconds Uptime\n'
                f'# TYPE app_uptime_seconds gauge\n'
                f'app_uptime_seconds {time.time() - app_state["start_time"]:.0f}\n'
            )
            self.send_response(200)
            self.send_header("Content-Type", "text/plain")
            self.end_headers()
            self.wfile.write(metrics.encode())
        
        else:
            self.send_json(404, {"error": "not found"})
    
    def send_json(self, code: int, data: dict):
        self.send_response(code)
        self.send_header("Content-Type", "application/json")
        self.end_headers()
        self.wfile.write(json.dumps(data).encode())
    
    def log_message(self, format, *args):
        pass  # Suppress default logging

def start_server(port: int = 8080):
    """Start the health check server."""
    # Simulate initialization delay
    def init():
        time.sleep(2)
        app_state["ready"] = True
        print("✅ Service ready")
    
    threading.Thread(target=init, daemon=True).start()
    
    server = HTTPServer(("0.0.0.0", port), HealthHandler)
    print(f"🚀 Server starting on port {port}")
    server.serve_forever()

# if __name__ == "__main__":
#     start_server()
```

---

## 7. CLI TOOL BUILDING <a name="7-cli"></a>

### Problem 7.1: Service Status CLI
**Task**: Build a CLI tool that checks service health and formats output.

```python
import argparse
import sys
import json

def main():
    parser = argparse.ArgumentParser(description="Platform Service Health Checker")
    subparsers = parser.add_subparsers(dest="command")
    
    # 'check' command
    check_parser = subparsers.add_parser("check", help="Check service health")
    check_parser.add_argument("--service", "-s", required=True, help="Service name")
    check_parser.add_argument("--namespace", "-n", default="production", help="K8s namespace")
    check_parser.add_argument("--format", "-f", choices=["text", "json"], default="text")
    
    # 'list' command
    list_parser = subparsers.add_parser("list", help="List all services")
    list_parser.add_argument("--namespace", "-n", default="production")
    
    # 'report' command
    report_parser = subparsers.add_parser("report", help="Generate health report")
    report_parser.add_argument("--output", "-o", default="-", help="Output file (- for stdout)")
    
    args = parser.parse_args()
    
    if args.command == "check":
        result = {"service": args.service, "namespace": args.namespace, "status": "healthy"}
        if args.format == "json":
            print(json.dumps(result, indent=2))
        else:
            print(f"Service: {result['service']}")
            print(f"Namespace: {result['namespace']}")
            print(f"Status: ✅ {result['status']}")
    
    elif args.command == "list":
        services = ["property-api", "search-service", "auth-service"]
        for s in services:
            print(f"  • {s} ({args.namespace})")
    
    elif args.command == "report":
        report = {"total": 5, "healthy": 4, "unhealthy": 1}
        output = json.dumps(report, indent=2)
        if args.output == "-":
            print(output)
        else:
            with open(args.output, "w") as f:
                f.write(output)
            print(f"Report written to {args.output}")
    
    else:
        parser.print_help()

if __name__ == "__main__":
    main()
```

---

## 8. TESTING & ERROR HANDLING <a name="8-testing"></a>

### Problem 8.1: Writing Tests for Platform Code
**Task**: Write tests for the deep_merge function using pytest.

```python
# test_config_merger.py
import pytest

def deep_merge(base: dict, override: dict) -> dict:
    """Deep merge override into base."""
    from copy import deepcopy
    result = deepcopy(base)
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = deepcopy(value)
    return result

class TestDeepMerge:
    def test_simple_merge(self):
        base = {"a": 1, "b": 2}
        override = {"b": 3, "c": 4}
        result = deep_merge(base, override)
        assert result == {"a": 1, "b": 3, "c": 4}
    
    def test_nested_merge(self):
        base = {"app": {"name": "test", "port": 80}}
        override = {"app": {"port": 8080}}
        result = deep_merge(base, override)
        assert result["app"]["name"] == "test"  # Preserved
        assert result["app"]["port"] == 8080  # Overridden
    
    def test_deep_nested_merge(self):
        base = {"a": {"b": {"c": 1, "d": 2}}}
        override = {"a": {"b": {"c": 3}}}
        result = deep_merge(base, override)
        assert result["a"]["b"]["c"] == 3
        assert result["a"]["b"]["d"] == 2
    
    def test_override_dict_with_scalar(self):
        base = {"a": {"nested": True}}
        override = {"a": "flat"}
        result = deep_merge(base, override)
        assert result["a"] == "flat"
    
    def test_empty_override(self):
        base = {"a": 1}
        result = deep_merge(base, {})
        assert result == {"a": 1}
    
    def test_empty_base(self):
        result = deep_merge({}, {"a": 1})
        assert result == {"a": 1}
    
    def test_does_not_mutate_original(self):
        base = {"a": {"b": 1}}
        override = {"a": {"c": 2}}
        result = deep_merge(base, override)
        assert "c" not in base["a"]  # Original unchanged
    
    def test_list_values_replaced_not_merged(self):
        base = {"tags": ["v1", "stable"]}
        override = {"tags": ["v2"]}
        result = deep_merge(base, override)
        assert result["tags"] == ["v2"]  # Lists are replaced, not merged
```

---

## 9. DATA STRUCTURES FOR PLATFORM PROBLEMS <a name="9-data-structures"></a>

### Problem 9.1: Rate Limiter (Sliding Window)

```python
import time
from collections import deque

class RateLimiter:
    """Sliding window rate limiter."""
    
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests: dict[str, deque] = {}  # client_id → timestamps
    
    def allow(self, client_id: str) -> bool:
        """Check if request from client_id is allowed."""
        now = time.time()
        
        if client_id not in self.requests:
            self.requests[client_id] = deque()
        
        window = self.requests[client_id]
        
        # Remove expired entries
        while window and window[0] <= now - self.window_seconds:
            window.popleft()
        
        if len(window) < self.max_requests:
            window.append(now)
            return True
        
        return False

# Test:
limiter = RateLimiter(max_requests=5, window_seconds=10)
for i in range(7):
    allowed = limiter.allow("user-1")
    print(f"Request {i+1}: {'✅ Allowed' if allowed else '❌ Rate limited'}")
```

### Problem 9.2: LRU Cache (for Config/DNS Caching)

```python
from collections import OrderedDict
import time

class TTLCache:
    """LRU cache with TTL (time-to-live)."""
    
    def __init__(self, max_size: int = 100, ttl_seconds: int = 300):
        self.max_size = max_size
        self.ttl = ttl_seconds
        self.cache: OrderedDict[str, tuple] = OrderedDict()  # key → (value, expiry_time)
    
    def get(self, key: str):
        """Get value from cache. Returns None if expired or missing."""
        if key not in self.cache:
            return None
        
        value, expiry = self.cache[key]
        if time.time() > expiry:
            del self.cache[key]
            return None
        
        self.cache.move_to_end(key)  # LRU: mark as recently used
        return value
    
    def set(self, key: str, value):
        """Set value in cache."""
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = (value, time.time() + self.ttl)
        
        if len(self.cache) > self.max_size:
            self.cache.popitem(last=False)  # Remove oldest
    
    def stats(self) -> dict:
        now = time.time()
        valid = sum(1 for _, (_, exp) in self.cache.items() if exp > now)
        return {"size": len(self.cache), "valid": valid, "expired": len(self.cache) - valid}

# Usage:
dns_cache = TTLCache(max_size=1000, ttl_seconds=60)
dns_cache.set("api.rea.com", "10.0.1.50")
print(dns_cache.get("api.rea.com"))  # "10.0.1.50"
```

---

## 10. CONCURRENCY & ASYNC <a name="10-concurrency"></a>

### Problem 10.1: Async HTTP Client

```python
import asyncio
import aiohttp
import time

async def fetch_url(session: aiohttp.ClientSession, url: str) -> dict:
    """Fetch a single URL and return result."""
    start = time.time()
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=5)) as resp:
            body = await resp.text()
            return {
                "url": url,
                "status": resp.status,
                "size": len(body),
                "time_ms": round((time.time() - start) * 1000),
            }
    except Exception as e:
        return {"url": url, "status": 0, "error": str(e)[:50], "time_ms": round((time.time() - start) * 1000)}

async def fetch_all(urls: list[str], max_concurrent: int = 10) -> list[dict]:
    """Fetch multiple URLs concurrently with concurrency limit."""
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def bounded_fetch(session, url):
        async with semaphore:
            return await fetch_url(session, url)
    
    async with aiohttp.ClientSession() as session:
        tasks = [bounded_fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks)

# Usage:
# results = asyncio.run(fetch_all(["https://httpbin.org/get"] * 20, max_concurrent=5))
```

---

## PRACTICE TIPS

1. **Type every solution** — don't just read. Type it, run it, modify it.
2. **Time yourself** — coding round is ~45-60 minutes. Practice solving 2-3 problems in that time.
3. **Explain as you code** — the interviewer watches you think. Narrate your approach.
4. **Start simple, then improve** — get a working solution first, then optimize.
5. **Handle edge cases** — empty input, missing keys, network timeouts.
6. **Know standard library** — `collections` (Counter, defaultdict, deque), `pathlib`, `re`, `json`, `yaml`, `subprocess`, `concurrent.futures`.
