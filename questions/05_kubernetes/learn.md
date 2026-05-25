# Kubernetes - LEARNING MATERIAL

---

## K8s Architecture

```mermaid
graph TD
    subgraph ControlPlane [Control Plane]
        API[API Server<br/>kubectl talks here]
        ETCD[etcd<br/>Key-value store<br/>Cluster state]
        SCHED[Scheduler<br/>Places pods on nodes]
        CM[Controller Manager<br/>Ensures desired state]
    end
    subgraph WorkerNode1 [Worker Node 1]
        KL1[kubelet<br/>Manages pods]
        KP1[kube-proxy<br/>Network routing]
        CR1[Container Runtime<br/>containerd]
        P1[Pod A]
        P2[Pod B]
    end
    subgraph WorkerNode2 [Worker Node 2]
        KL2[kubelet]
        KP2[kube-proxy]
        CR2[Container Runtime]
        P3[Pod C]
    end
    API --> ETCD
    API --> SCHED
    API --> CM
    KL1 --> API
    KL2 --> API
    KL1 --> P1
    KL1 --> P2
    KL2 --> P3
```

## K8s Object Hierarchy

```mermaid
graph TD
    D[Deployment] -->|manages| RS[ReplicaSet]
    RS -->|manages| P1[Pod 1]
    RS -->|manages| P2[Pod 2]
    RS -->|manages| P3[Pod 3]
    SVC[Service] -->|routes to| P1
    SVC -->|routes to| P2
    SVC -->|routes to| P3
    ING[Ingress] -->|routes to| SVC
    CM[ConfigMap] -.->|env vars / files| P1
    SEC[Secret] -.->|credentials| P1
    PVC[PVC] -.->|storage| P1
```

## Service Types

```mermaid
graph LR
    subgraph ClusterIP
        CIP[ClusterIP Service<br/>Internal only<br/>10.96.0.10:80]
    end
    subgraph NodePort
        NP[NodePort Service<br/>Node IP:30080<br/>External access]
    end
    subgraph LoadBalancer
        LB[Cloud Load Balancer<br/>Public IP<br/>Auto-provisioned]
    end

    Internet -->|Public IP| LB
    LB --> NP
    NP --> CIP
    CIP --> Pods
```

| Type | Access | Port | Use Case |
|---|---|---|---|
| `ClusterIP` | Internal only | Any | Default, inter-service |
| `NodePort` | External via node IP | 30000-32767 | Dev/test |
| `LoadBalancer` | External via cloud LB | Any | Production |
| `ExternalName` | DNS CNAME | N/A | External service alias |

## Probes

```mermaid
graph TD
    subgraph Probes
        STARTUP[Startup Probe<br/>Is app started?<br/>Disables others until pass]
        LIVENESS[Liveness Probe<br/>Is app alive?<br/>Fail → restart container]
        READINESS[Readiness Probe<br/>Is app ready for traffic?<br/>Fail → remove from Service]
    end
    STARTUP -->|passes| LIVENESS
    STARTUP -->|passes| READINESS
```

## Complete Manifest Template

```yaml
# deployment.yaml
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
      maxSurge: 1          # 1 extra pod during update
      maxUnavailable: 0    # never have fewer than 3
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: registry/myapp:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: db_host
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: password
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        startupProbe:
          httpGet:
            path: /healthz
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  db_host: "postgres.prod.svc.cluster.local"
---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=    # base64
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
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
            name: myapp-svc
            port:
              number: 80
```

## Troubleshooting Flow

```mermaid
graph TD
    START[Pod Issue] --> CHECK{kubectl describe pod}
    CHECK -->|ImagePullBackOff| IMG[Wrong image name/tag<br/>Registry auth<br/>Network issue]
    CHECK -->|Pending| PEND[No resources<br/>Node selector mismatch<br/>PVC not bound]
    CHECK -->|CrashLoopBackOff| CRASH[Check logs: kubectl logs --previous<br/>OOMKilled: increase memory<br/>App crash: fix code<br/>Missing config: check ConfigMap/Secret]
    CHECK -->|Running but no traffic| TRAFFIC[Check Service selector matches<br/>Check endpoints<br/>Check Ingress rules<br/>Check NetworkPolicy]
```

## Helm Chart Structure
```
mychart/
├── Chart.yaml          # Chart metadata (name, version)
├── values.yaml         # Default values
├── templates/
│   ├── deployment.yaml # {{ .Values.replicas }}
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Template helpers
│   └── NOTES.txt       # Post-install message
└── charts/             # Sub-chart dependencies
```
