# Round 1: AI Coding & Analysis — REA Platform Engineer

> Based on JD: GoLang/Python, Kubernetes, migration automation, developer tooling, CI/CD

---

## SECTION A: Go Basics for Platform Engineering

### Go Fundamentals (Must-Know for Coding Round)

```go
// 1. Basic Go program structure
package main

import (
    "fmt"
    "os"
    "strings"
)

func main() {
    // Variables
    name := "Vaibhav"        // short declaration
    var age int = 28          // explicit type
    var active bool = true    // boolean

    fmt.Printf("Name: %s, Age: %d, Active: %v\n", name, age, active)

    // Slices (dynamic arrays)
    tools := []string{"kubectl", "helm", "terraform"}
    tools = append(tools, "argocd")
    fmt.Println(tools)

    // Maps
    config := map[string]string{
        "env":     "production",
        "region":  "ap-south-1",
        "cluster": "rea-platform",
    }
    for key, value := range config {
        fmt.Printf("%s = %s\n", key, value)
    }

    // Error handling (Go pattern — no try/catch)
    data, err := os.ReadFile("config.yaml")
    if err != nil {
        fmt.Fprintf(os.Stderr, "Error: %v\n", err)
        os.Exit(1)
    }
    fmt.Println(string(data))
}
```

```go
// 2. Structs and Methods
type Pod struct {
    Name      string
    Namespace string
    Status    string
    Restarts  int
}

func (p Pod) IsHealthy() bool {
    return p.Status == "Running" && p.Restarts < 5
}

func (p Pod) String() string {
    return fmt.Sprintf("%s/%s [%s] restarts=%d", p.Namespace, p.Name, p.Status, p.Restarts)
}

// 3. Interfaces
type HealthChecker interface {
    IsHealthy() bool
}

func checkHealth(resources []HealthChecker) {
    for _, r := range resources {
        if !r.IsHealthy() {
            fmt.Printf("UNHEALTHY: %v\n", r)
        }
    }
}
```

```go
// 4. Goroutines and Channels (concurrency — Go's killer feature)
package main

import (
    "fmt"
    "sync"
    "time"
)

// Check health of multiple services concurrently
func checkServiceHealth(services []string) map[string]bool {
    results := make(map[string]bool)
    var mu sync.Mutex
    var wg sync.WaitGroup

    for _, svc := range services {
        wg.Add(1)
        go func(service string) {
            defer wg.Done()
            // Simulate health check
            healthy := pingService(service)
            mu.Lock()
            results[service] = healthy
            mu.Unlock()
        }(svc)
    }
    wg.Wait()
    return results
}

// Channel-based worker pool
func processPodsWorkerPool(pods []string, workers int) {
    jobs := make(chan string, len(pods))
    results := make(chan string, len(pods))

    // Start workers
    for w := 0; w < workers; w++ {
        go func(id int) {
            for pod := range jobs {
                result := fmt.Sprintf("Worker %d processed %s", id, pod)
                results <- result
            }
        }(w)
    }

    // Send jobs
    for _, pod := range pods {
        jobs <- pod
    }
    close(jobs)

    // Collect results
    for i := 0; i < len(pods); i++ {
        fmt.Println(<-results)
    }
}
```

```go
// 5. HTTP Server (common in platform tooling)
package main

import (
    "encoding/json"
    "log"
    "net/http"
)

type HealthResponse struct {
    Status  string `json:"status"`
    Version string `json:"version"`
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    resp := HealthResponse{Status: "ok", Version: "1.0.0"}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}

func main() {
    http.HandleFunc("/healthz", healthHandler)
    log.Println("Starting server on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## SECTION B: Python Platform Tooling (Your Strength)

### Scenario 1: K8s Pod Health Checker CLI Tool

```python
"""Platform tool: Check pod health across namespaces"""
import subprocess
import json
import sys
from typing import List, Dict

def get_pods(namespace: str = "default") -> List[Dict]:
    """Get pods from a K8s namespace"""
    cmd = ["kubectl", "get", "pods", "-n", namespace, "-o", "json"]
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode != 0:
        print(f"Error: {result.stderr}", file=sys.stderr)
        return []
    data = json.loads(result.stdout)
    return data.get("items", [])

def check_pod_health(pods: List[Dict]) -> Dict[str, List]:
    """Categorize pods by health status"""
    report = {"healthy": [], "unhealthy": [], "pending": []}
    for pod in pods:
        name = pod["metadata"]["name"]
        phase = pod["status"].get("phase", "Unknown")
        restarts = sum(
            cs.get("restartCount", 0)
            for cs in pod["status"].get("containerStatuses", [])
        )
        if phase == "Running" and restarts < 5:
            report["healthy"].append(name)
        elif phase == "Pending":
            report["pending"].append(name)
        else:
            report["unhealthy"].append({"name": name, "phase": phase, "restarts": restarts})
    return report

def main():
    namespaces = sys.argv[1:] or ["default"]
    for ns in namespaces:
        print(f"\n=== Namespace: {ns} ===")
        pods = get_pods(ns)
        report = check_pod_health(pods)
        print(f"  Healthy: {len(report['healthy'])}")
        print(f"  Unhealthy: {len(report['unhealthy'])}")
        for u in report["unhealthy"]:
            print(f"    ⚠ {u['name']} — {u['phase']} (restarts: {u['restarts']})")
        print(f"  Pending: {len(report['pending'])}")

if __name__ == "__main__":
    main()
```

### Scenario 2: Migration Automation Script

```python
"""Platform tool: Automate K8s resource migration between clusters"""
import subprocess
import json
import yaml
import os
from pathlib import Path

def export_resources(source_context: str, namespace: str, resource_types: list, output_dir: str):
    """Export K8s resources from source cluster"""
    Path(output_dir).mkdir(parents=True, exist_ok=True)
    exported = []

    for rtype in resource_types:
        cmd = [
            "kubectl", "get", rtype, "-n", namespace,
            "--context", source_context, "-o", "json"
        ]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode != 0:
            print(f"Warning: Failed to get {rtype}: {result.stderr}")
            continue

        items = json.loads(result.stdout).get("items", [])
        for item in items:
            name = item["metadata"]["name"]
            # Clean metadata for import
            item["metadata"] = {
                "name": name,
                "namespace": namespace,
                "labels": item["metadata"].get("labels", {}),
                "annotations": {
                    k: v for k, v in item["metadata"].get("annotations", {}).items()
                    if not k.startswith("kubectl.kubernetes.io")
                }
            }
            # Remove status, resourceVersion, uid
            item.pop("status", None)
            item["metadata"].pop("resourceVersion", None)
            item["metadata"].pop("uid", None)

            filename = f"{output_dir}/{rtype}_{name}.yaml"
            with open(filename, "w") as f:
                yaml.dump(item, f, default_flow_style=False)
            exported.append(filename)
            print(f"Exported: {rtype}/{name}")

    return exported

def apply_resources(target_context: str, files: list, dry_run: bool = True):
    """Apply exported resources to target cluster"""
    for f in files:
        cmd = ["kubectl", "apply", "-f", f, "--context", target_context]
        if dry_run:
            cmd.append("--dry-run=client")
        result = subprocess.run(cmd, capture_output=True, text=True)
        status = "✓" if result.returncode == 0 else "✗"
        print(f"  {status} {os.path.basename(f)}: {result.stdout.strip()}")

if __name__ == "__main__":
    resources = ["configmaps", "secrets", "deployments", "services", "ingresses"]
    files = export_resources("old-cluster", "production", resources, "./migration-export")
    print(f"\nExported {len(files)} resources. Apply with --dry-run first:")
    apply_resources("new-cluster", files, dry_run=True)
```

### Scenario 3: Developer Experience — Self-Service Namespace Provisioner

```python
"""Platform tool: Self-service namespace provisioner for developer teams"""
import subprocess
import json
import sys
import yaml

NAMESPACE_TEMPLATE = {
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "name": "",
        "labels": {
            "managed-by": "platform-team",
            "environment": "",
            "team": "",
        }
    }
}

RESOURCE_QUOTA = {
    "apiVersion": "v1",
    "kind": "ResourceQuota",
    "metadata": {"name": "default-quota"},
    "spec": {
        "hard": {
            "requests.cpu": "4",
            "requests.memory": "8Gi",
            "limits.cpu": "8",
            "limits.memory": "16Gi",
            "pods": "20",
        }
    }
}

NETWORK_POLICY = {
    "apiVersion": "networking.k8s.io/v1",
    "kind": "NetworkPolicy",
    "metadata": {"name": "deny-all-ingress"},
    "spec": {
        "podSelector": {},
        "policyTypes": ["Ingress"],
        "ingress": []  # Deny all by default; teams add specific rules
    }
}

def provision_namespace(team: str, env: str, dry_run: bool = True):
    """Provision a namespace with guardrails"""
    ns_name = f"{team}-{env}"

    # Create namespace
    ns = NAMESPACE_TEMPLATE.copy()
    ns["metadata"]["name"] = ns_name
    ns["metadata"]["labels"]["environment"] = env
    ns["metadata"]["labels"]["team"] = team

    resources = [ns, RESOURCE_QUOTA, NETWORK_POLICY]

    for resource in resources:
        if "metadata" in resource and "namespace" not in resource["metadata"]:
            resource["metadata"]["namespace"] = ns_name

        cmd = ["kubectl", "apply", "-f", "-"]
        if dry_run:
            cmd.append("--dry-run=client")

        result = subprocess.run(
            cmd, input=yaml.dump(resource), capture_output=True, text=True
        )
        kind = resource["kind"]
        if result.returncode == 0:
            print(f"  ✓ {kind}/{resource['metadata']['name']}")
        else:
            print(f"  ✗ {kind}: {result.stderr}")

    print(f"\nNamespace '{ns_name}' provisioned with quota + network policy")

if __name__ == "__main__":
    team = sys.argv[1] if len(sys.argv) > 1 else "frontend"
    env = sys.argv[2] if len(sys.argv) > 2 else "dev"
    provision_namespace(team, env, dry_run=True)
```

---

## SECTION C: Kubernetes Coding Scenarios

### Write a Deployment + Service + HPA from scratch

```yaml
# Scenario: Deploy a microservice with auto-scaling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: property-search-api
  namespace: rea-platform
  labels:
    app: property-search-api
    team: platform
    version: v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: property-search-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: property-search-api
        version: v2
    spec:
      containers:
      - name: api
        image: rea-ecr.amazonaws.com/property-search-api:v2.1.0
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            cpu: 250m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 15
        readinessProbe:
          httpGet:
            path: /readyz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        env:
        - name: ENV
          value: production
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: host
---
apiVersion: v1
kind: Service
metadata:
  name: property-search-api
  namespace: rea-platform
spec:
  selector:
    app: property-search-api
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: property-search-api
  namespace: rea-platform
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: property-search-api
  minReplicas: 3
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 25
        periodSeconds: 60
```

### Write a CronJob for log cleanup

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: log-cleanup
  namespace: rea-platform
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: bitnami/kubectl:latest
            command:
            - /bin/sh
            - -c
            - |
              # Delete completed/failed pods older than 24h
              kubectl get pods -n rea-platform \
                --field-selector=status.phase!=Running \
                -o jsonpath='{.items[*].metadata.name}' | \
                tr ' ' '\n' | \
                xargs -I {} kubectl delete pod {} -n rea-platform
              echo "Cleanup completed at $(date)"
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

---

## SECTION D: CI/CD Pipeline Coding

### Buildkite Pipeline (REA uses Buildkite)

```yaml
# .buildkite/pipeline.yml
steps:
  - label: ":golang: Lint & Test"
    command:
      - go vet ./...
      - go test -v -race -coverprofile=coverage.out ./...
    plugins:
      - docker#v5.0.0:
          image: "golang:1.22"

  - label: ":docker: Build & Push"
    command:
      - docker build -t $ECR_REPO:$BUILDKITE_COMMIT .
      - docker push $ECR_REPO:$BUILDKITE_COMMIT
    plugins:
      - ecr#v2.0.0:
          login: true
          region: ap-southeast-2

  - wait

  - label: ":kubernetes: Deploy to Staging"
    command:
      - helm upgrade --install property-api ./charts/property-api
        --namespace staging
        --set image.tag=$BUILDKITE_COMMIT
        --wait --timeout 5m
    agents:
      queue: "deploy"

  - block: ":rocket: Deploy to Production?"
    branches: "main"

  - label: ":kubernetes: Deploy to Production"
    command:
      - helm upgrade --install property-api ./charts/property-api
        --namespace production
        --set image.tag=$BUILDKITE_COMMIT
        --wait --timeout 10m
    branches: "main"
    agents:
      queue: "deploy-prod"
```

---

## PRACTICE PROBLEMS

1. **Write a Go CLI** that takes a namespace and lists all pods with restarts > 3
2. **Write a Python script** that reads a Helm values.yaml, finds all image tags, and updates them to a new version
3. **Write a K8s manifest** for a StatefulSet with PVCs for a database
4. **Write a Buildkite pipeline** for a multi-service monorepo with path-based triggers
5. **Write a Go HTTP handler** that proxies requests to a backend service with timeout and retry
