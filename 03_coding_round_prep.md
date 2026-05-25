# CODING ROUND PREP - Practice Scenarios
# Likely: Jenkins pipeline, Dockerfile, K8s manifests, Azure Pipeline, Shell/Python scripts

---

## SCENARIO 1: Jenkins Declarative Pipeline
# Build a Python app, run tests, build Docker image, push to registry, deploy to K8s

```groovy
pipeline {
    agent any

    environment {
        REGISTRY = 'myregistry.azurecr.io'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Lint') {
            steps {
                sh 'flake8 src/'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'pytest tests/ --junitxml=reports/test-results.xml'
            }
            post {
                always {
                    junit 'reports/test-results.xml'
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'snyk test --file=requirements.txt'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
                    sh "docker login ${REGISTRY} -u ${ACR_USER} -p ${ACR_PASS}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh "kubectl set image deployment/myapp myapp=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} --namespace=production"
                sh "kubectl rollout status deployment/myapp --namespace=production --timeout=120s"
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            mail to: 'team@company.com',
                 subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Check: ${env.BUILD_URL}"
        }
        always {
            cleanWs()
        }
    }
}
```

---

## SCENARIO 2: Dockerfile (Multi-stage build for a Python app)

```dockerfile
# Stage 1: Build
FROM python:3.11-slim AS builder

WORKDIR /app

# Install dependencies first (layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Copy source
COPY src/ ./src/

# Stage 2: Production
FROM python:3.11-slim

# Security: run as non-root
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy only installed packages from builder
COPY --from=builder /install /usr/local
COPY --from=builder /app/src ./src/

# Switch to non-root user
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["python"]
CMD ["src/main.py"]
```

---

## SCENARIO 3: Kubernetes Manifests (Complete app deployment)

### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: db_host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db_password
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

### configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production
data:
  db_host: "postgres.production.svc.cluster.local"
  log_level: "info"
  app_port: "8080"
```

### secret.yaml (base64 encoded)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: production
type: Opaque
data:
  db_password: cGFzc3dvcmQxMjM=   # echo -n "password123" | base64
```

### ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

---

## SCENARIO 4: Azure Pipeline YAML (Your strong area - but practice)

```yaml
trigger:
  branches:
    include:
      - main
      - release/*
  paths:
    exclude:
      - '*.md'
      - docs/

pool:
  vmImage: 'ubuntu-latest'

variables:
  imageName: 'myapp'
  registryConnection: 'acr-connection'
  kubernetesConnection: 'aks-connection'

stages:
  - stage: Build
    displayName: 'Build & Test'
    jobs:
      - job: BuildJob
        steps:
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '3.11'

          - script: |
              pip install -r requirements.txt
              pytest tests/ --junitxml=test-results.xml
            displayName: 'Install & Test'

          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results.xml'

          - task: Docker@2
            displayName: 'Build & Push Image'
            inputs:
              containerRegistry: $(registryConnection)
              repository: $(imageName)
              command: 'buildAndPush'
              Dockerfile: 'Dockerfile'
              tags: |
                $(Build.BuildId)
                latest

  - stage: DeployStaging
    displayName: 'Deploy to Staging'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployToStaging
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: $(kubernetesConnection)
                    namespace: 'staging'
                    manifests: 'k8s/*.yaml'
                    containers: '$(registryConnection)/$(imageName):$(Build.BuildId)'

  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployStaging
    condition: succeeded()
    jobs:
      - deployment: DeployToProd
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: $(kubernetesConnection)
                    namespace: 'production'
                    manifests: 'k8s/*.yaml'
                    containers: '$(registryConnection)/$(imageName):$(Build.BuildId)'
```

---

## SCENARIO 5: Shell Script - Log Analyzer

```bash
#!/bin/bash
# Analyze nginx access logs: top IPs, error count, response time stats

LOG_FILE="${1:-/var/log/nginx/access.log}"

if [ ! -f "$LOG_FILE" ]; then
    echo "ERROR: Log file not found: $LOG_FILE"
    exit 1
fi

echo "=== Log Analysis: $LOG_FILE ==="
echo "Total requests: $(wc -l < "$LOG_FILE")"

echo ""
echo "=== Top 10 IPs ==="
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "=== HTTP Status Code Distribution ==="
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn

echo ""
echo "=== 5xx Errors ==="
grep -c ' 5[0-9][0-9] ' "$LOG_FILE"

echo ""
echo "=== Top 10 Requested Paths ==="
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "=== Requests per Hour ==="
awk -F'[' '{print $2}' "$LOG_FILE" | awk -F: '{print $1":"$2}' | sort | uniq -c
```

---

## SCENARIO 6: Python Script - Health Check Monitor

```python
#!/usr/bin/env python3
"""Monitor multiple service endpoints and report status."""

import requests
import sys
import json
from datetime import datetime

ENDPOINTS = [
    {"name": "API", "url": "https://api.example.com/health", "timeout": 5},
    {"name": "Web", "url": "https://www.example.com", "timeout": 5},
    {"name": "DB Proxy", "url": "http://db-proxy:8080/status", "timeout": 3},
]

def check_endpoint(endpoint):
    """Check a single endpoint and return status."""
    try:
        resp = requests.get(endpoint["url"], timeout=endpoint["timeout"])
        return {
            "name": endpoint["name"],
            "status": "UP" if resp.status_code == 200 else "DEGRADED",
            "code": resp.status_code,
            "latency_ms": round(resp.elapsed.total_seconds() * 1000),
        }
    except requests.exceptions.Timeout:
        return {"name": endpoint["name"], "status": "TIMEOUT", "code": None, "latency_ms": None}
    except requests.exceptions.ConnectionError:
        return {"name": endpoint["name"], "status": "DOWN", "code": None, "latency_ms": None}

def main():
    results = [check_endpoint(ep) for ep in ENDPOINTS]
    report = {"timestamp": datetime.now().isoformat(), "checks": results}

    for r in results:
        symbol = "OK" if r["status"] == "UP" else "FAIL"
        latency = f"{r['latency_ms']}ms" if r["latency_ms"] else "N/A"
        print(f"[{symbol}] {r['name']}: {r['status']} (HTTP {r['code']}, {latency})")

    # Exit with error if any service is down
    if any(r["status"] in ("DOWN", "TIMEOUT") for r in results):
        sys.exit(1)

if __name__ == "__main__":
    main()
```

---

## SCENARIO 7: Ansible Playbook (Bonus - on your resume)

```yaml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  vars:
    app_version: "1.2.3"
    app_dir: /opt/myapp

  tasks:
    - name: Install required packages
      apt:
        name:
          - python3
          - python3-pip
          - nginx
        state: present
        update_cache: yes

    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: www-data
        group: www-data

    - name: Deploy application code
      copy:
        src: ./dist/
        dest: "{{ app_dir }}/"
        owner: www-data
      notify: restart app

    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/myapp
      notify: restart nginx

    - name: Enable nginx site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link

  handlers:
    - name: restart app
      systemd:
        name: myapp
        state: restarted

    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
```

---

## SCENARIO 8: Terraform (Bonus - on your resume)

```hcl
# Create an Azure AKS cluster
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "myapp-aks"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "myapp"

  default_node_pool {
    name       = "default"
    node_count = 3
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin = "azure"
    network_policy = "calico"
  }

  tags = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}

output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks.kube_config_raw
  sensitive = true
}
```

---

## DSA Quick Reference (if asked, lowest priority)

### Common patterns for DevOps interviews:
1. **String manipulation**: Parse log lines, extract fields
2. **File processing**: Read CSV/JSON, transform data
3. **Dictionary/HashMap**: Count occurrences, group by key
4. **Sorting**: Sort by timestamp, priority

### Python example - Parse and aggregate logs:
```python
def parse_logs(log_lines):
    """Group error counts by service name from log lines like:
    '2024-01-15 ERROR service-auth: Connection timeout'
    """
    from collections import defaultdict
    errors = defaultdict(int)

    for line in log_lines:
        if 'ERROR' in line:
            parts = line.split()
            service = parts[2].rstrip(':')
            errors[service] += 1

    return dict(sorted(errors.items(), key=lambda x: x[1], reverse=True))
```
