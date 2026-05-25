# Kubernetes — Deep-Dive Learning Guide

---

## 1. What Is Kubernetes?

Kubernetes (K8s) is a **container orchestration platform** that automates deployment, scaling, and management of containerized applications across a cluster of machines.

```
What K8s does for you:
  ✅ Scheduling    — Places pods on nodes with available resources
  ✅ Self-healing  — Restarts crashed containers, reschedules on dead nodes
  ✅ Scaling       — HPA/VPA auto-scales based on CPU/memory/custom metrics
  ✅ Load balancing — Distributes traffic via Services
  ✅ Rolling updates — Zero-downtime deployments
  ✅ Secret/Config  — Manages config and secrets separately from images
  ✅ Storage        — Dynamically provisions persistent volumes
```

---

## 2. Kubernetes Architecture

```
┌──────────────────── CONTROL PLANE ──────────────────────────────────┐
│                                                                      │
│  ┌─────────────────┐    ┌──────────────────────────────────────┐    │
│  │   kube-apiserver │◄──►│              etcd                    │    │
│  │                  │    │  Distributed key-value store         │    │
│  │  - REST API front│    │  Single source of truth              │    │
│  │  - Authz/Authn   │    │  Stores ALL cluster state            │    │
│  │  - Admission ctrl│    │  Only apiserver talks to etcd        │    │
│  └───────┬──────────┘    └──────────────────────────────────────┘    │
│          │                                                           │
│  ┌───────▼──────────┐    ┌──────────────────────────────────────┐   │
│  │  kube-scheduler  │    │  kube-controller-manager             │   │
│  │                  │    │                                      │   │
│  │  Watches for     │    │  Runs control loops:                 │   │
│  │  unscheduled pods│    │  - ReplicaSet controller             │   │
│  │  Scores nodes:   │    │  - Deployment controller             │   │
│  │  - Resources     │    │  - Node controller                   │   │
│  │  - Affinity      │    │  - Job controller                    │   │
│  │  - Taints/toler. │    │  - ServiceAccount controller         │   │
│  │  Assigns best    │    │  - Endpoint controller               │   │
│  │  node to pod     │    │                                      │   │
│  └──────────────────┘    │  Actual state ──► Desired state      │   │
│                          └──────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  cloud-controller-manager (optional, for cloud providers)    │   │
│  │  - Node controller (detects deleted cloud VMs)               │   │
│  │  - Route controller (cloud network routes)                   │   │
│  │  - Service controller (provisions cloud load balancers)      │   │
│  └──────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
            │
            │  kubelet watches apiserver via HTTP long-poll
            │
┌───────────▼─── WORKER NODE 1 ──────────────────────────────────────┐
│                                                                      │
│  ┌──────────────┐   ┌─────────────┐   ┌──────────────────────────┐ │
│  │   kubelet    │   │ kube-proxy  │   │  Container Runtime       │ │
│  │              │   │             │   │                          │ │
│  │  - Agent on  │   │  - Maintains│   │  containerd or CRI-O    │ │
│  │    each node │   │    iptables/│   │  ┌────────────────────┐ │ │
│  │  - Watches   │   │    IPVS for │   │  │      runc          │ │ │
│  │    apiserver │   │    Services │   │  │  (OCI low-level)   │ │ │
│  │  - Ensures   │   │  - Routes   │   │  └────────────────────┘ │ │
│  │    pods run  │   │    traffic  │   │                          │ │
│  │  - Reports   │   │    to pods  │   │  Talks to kubelet       │ │
│  │    node status│  │             │   │  via CRI (gRPC)         │ │
│  └──────────────┘   └─────────────┘   └──────────────────────────┘ │
│                                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                         │
│  │  Pod A   │  │  Pod B   │  │  Pod C   │                         │
│  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │                         │
│  │ │ ctr1 │ │  │ │ ctr1 │ │  │ │ ctr1 │ │                         │
│  │ │ ctr2 │ │  │ └──────┘ │  │ │ ctr2 │ │                         │
│  │ └──────┘ │  └──────────┘  │ └──────┘ │                         │
│  └──────────┘                └──────────┘                          │
└──────────────────────────────────────────────────────────────────────┘
```

### Control Plane Components

| Component | What It Does | Key Details |
|-----------|-------------|-------------|
| **kube-apiserver** | Front door for ALL K8s operations | RESTful, validates requests, persists to etcd |
| **etcd** | Distributed key-value store | Raft consensus, only apiserver reads/writes |
| **kube-scheduler** | Assigns pods to nodes | Scoring: resources, affinity, taints, topology |
| **kube-controller-manager** | Runs reconciliation loops | Drives actual → desired state continuously |
| **cloud-controller-manager** | Cloud provider integration | LBs, volumes, routes (AWS/Azure/GCP) |

### Worker Node Components

| Component | What It Does | Key Details |
|-----------|-------------|-------------|
| **kubelet** | Agent on every node | Watches apiserver, manages pod lifecycle |
| **kube-proxy** | Network routing | iptables or IPVS rules for Service → Pod |
| **Container Runtime** | Runs containers | containerd/CRI-O via CRI interface |

---

## 3. How K8s Replaced Docker

```
Before K8s 1.24 (dockershim era):
  kubelet ──► dockershim ──► dockerd ──► containerd ──► runc
              (adapter)      (full Docker daemon - unnecessary!)

After K8s 1.24 (CRI era):
  kubelet ──► CRI (gRPC) ──► containerd ──► runc
              (standard interface, no Docker needed)
```

**Why?** Docker added unnecessary layers — CLI, API server, image build, Swarm. K8s only needs a runtime. CRI (Container Runtime Interface) is the standard gRPC API that any runtime can implement.

**Your images still work** — they're OCI-compliant. containerd reads them directly.

---

## 4. K8s Object Hierarchy

```
┌──────────────────────────────────────────────────────────────┐
│  Ingress                                                      │
│  (L7 routing: host/path → Service)                           │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Service                                              │    │
│  │  (Stable IP + DNS, load balances to pods)             │    │
│  │  ┌──────────────────────────────────────────────┐    │    │
│  │  │  Deployment                                   │    │    │
│  │  │  (Declarative updates, rollback)              │    │    │
│  │  │  ┌──────────────────────────────────────┐    │    │    │
│  │  │  │  ReplicaSet                          │    │    │    │
│  │  │  │  (Ensures N pods are running)        │    │    │    │
│  │  │  │  ┌────────┐ ┌────────┐ ┌────────┐  │    │    │    │
│  │  │  │  │ Pod 1  │ │ Pod 2  │ │ Pod 3  │  │    │    │    │
│  │  │  │  │┌──────┐│ │┌──────┐│ │┌──────┐│  │    │    │    │
│  │  │  │  ││ ctr  ││ ││ ctr  ││ ││ ctr  ││  │    │    │    │
│  │  │  │  │└──────┘│ │└──────┘│ │└──────┘│  │    │    │    │
│  │  │  │  └────────┘ └────────┘ └────────┘  │    │    │    │
│  │  │  └──────────────────────────────────────┘    │    │    │
│  │  └──────────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘

Supporting Objects:
  ConfigMap ──── env vars / config files ──── mounted into Pods
  Secret ─────── credentials (base64) ─────── mounted into Pods
  PVC ─────────── persistent storage ────────── mounted into Pods
  HPA ─────────── auto-scales Deployment ────── based on metrics
  NetworkPolicy ─ firewall rules ──────────── between Pods
```

---

## 5. Pod — The Smallest Deployable Unit

A Pod is **one or more containers** that share:
- Same network namespace (localhost communication, same IP)
- Same storage volumes
- Same lifecycle (co-scheduled, co-located)

```
┌─────────── Pod (10.244.1.5) ──────────────┐
│                                             │
│  ┌───────────┐    ┌───────────┐            │
│  │ App Container│  │ Sidecar    │           │
│  │ (main app) │    │ (log agent)│           │
│  │ Port 8080  │    │ Port 9090  │           │
│  └─────┬─────┘    └─────┬─────┘           │
│        │                 │                  │
│        └──── localhost ──┘  (shared net ns) │
│                                             │
│  ┌──────────────────────────────────┐      │
│  │  Shared Volume (/var/log)        │      │
│  └──────────────────────────────────┘      │
│                                             │
│  pause container (holds network namespace) │
└─────────────────────────────────────────────┘
```

### Pod Lifecycle

```
Pending ──► Running ──► Succeeded
                │
                └──► Failed
                │
                └──► CrashLoopBackOff (keeps crashing, exponential backoff)
```

### Init Containers

Run **before** app containers, sequentially. Use for setup tasks:

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db 5432; do sleep 2; done']
  - name: migrate
    image: myapp:v1
    command: ['python', 'manage.py', 'migrate']
  containers:
  - name: app
    image: myapp:v1     # Starts only after BOTH init containers succeed
```

---

## 6. Pod Creation Flow (What Actually Happens)

```
Step 1:  kubectl apply -f deploy.yaml
         │
Step 2:  kubectl ──► kube-apiserver (REST: POST /apis/apps/v1/deployments)
         │
Step 3:  apiserver ──► validates YAML, applies admission controllers
         │              writes Deployment object to etcd
         │
Step 4:  Deployment controller (in controller-manager) sees new Deployment
         │              creates ReplicaSet object
         │
Step 5:  ReplicaSet controller sees desired replicas > actual
         │              creates Pod objects (status: Pending)
         │
Step 6:  Scheduler sees unscheduled Pods
         │              scores nodes: resources, affinity, taints, topology
         │              assigns best node → updates Pod.spec.nodeName
         │
Step 7:  kubelet on assigned node watches apiserver
         │              sees new Pod assigned to it
         │
Step 8:  kubelet ──► CRI ──► containerd ──► runc
         │              pulls image, creates container
         │
Step 9:  kubelet runs probes (startup → liveness + readiness)
         │              reports Pod status back to apiserver
         │
Step 10: kube-proxy updates iptables/IPVS rules
                       Service can now route traffic to the Pod
```

---

## 7. Services — Networking Abstraction

A Service provides a **stable IP and DNS name** for a set of Pods (which have ephemeral IPs).

```
┌─────────────── Service (ClusterIP: 10.96.0.10) ──────────────┐
│  selector: app=web                                            │
│  DNS: web-svc.default.svc.cluster.local                      │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ Pod 1    │  │ Pod 2    │  │ Pod 3    │                   │
│  │10.244.1.5│  │10.244.2.3│  │10.244.1.8│                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
│                                                               │
│  Endpoints: [10.244.1.5:8080, 10.244.2.3:8080, 10.244.1.8]  │
└───────────────────────────────────────────────────────────────┘
```

### Service Types

```
Internet ──► LoadBalancer ──► NodePort ──► ClusterIP ──► Pods
             (cloud LB)      (node:30080)  (internal)
```

| Type | Access | Port Range | Use Case |
|------|--------|------------|----------|
| `ClusterIP` | Internal only | Any | Default, inter-service |
| `NodePort` | External via any node IP | 30000-32767 | Dev/test |
| `LoadBalancer` | External via cloud LB | Any | Production |
| `ExternalName` | DNS CNAME alias | N/A | External service reference |

### How kube-proxy works

```
Option 1: iptables mode (default)
  Client ──► Service IP (iptables DNAT) ──► random Pod IP
  - Pure kernel-level, no userspace proxy
  - Random selection, no connection tracking

Option 2: IPVS mode (better for large clusters)
  Client ──► Service IP (IPVS virtual server) ──► Pod IP
  - Real load balancing algorithms (rr, lc, wrr, sh)
  - O(1) lookup vs O(n) iptables chains
  - Better for 1000+ Services
```

---

## 8. Ingress — L7 Routing

> **⚠️ UPDATE (2026):** The `kubernetes/ingress-nginx` controller was **retired and archived in March 2026**. No further releases or security patches. Use **Traefik**, **AGIC (Azure)**, or **Gateway API** instead. See answers.md Q20a–20g for full details.

```
Internet
    │
    ▼
┌──────────────── Ingress Controller (Traefik/AGIC/Envoy) ───────┐
│                                                                 │
│  Rule: host=app.example.com, path=/api  →  api-svc:80         │
│  Rule: host=app.example.com, path=/     →  web-svc:80         │
│  Rule: host=admin.example.com           →  admin-svc:80       │
│  TLS termination: cert from Secret                             │
│                                                                 │
└───────────────────┬────────────────┬───────────────────────────┘
                    │                │
              ┌─────▼─────┐   ┌─────▼─────┐
              │  api-svc  │   │  web-svc  │
              │  (Pods)   │   │  (Pods)   │
              └───────────┘   └───────────┘
```

### Standard Ingress (still works with Traefik)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  ingressClassName: traefik         # ← use traefik instead of nginx
  tls:
  - hosts: [app.example.com]
    secretName: tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service: { name: api-svc, port: { number: 80 } }
      - path: /
        pathType: Prefix
        backend:
          service: { name: web-svc, port: { number: 80 } }
```

### Gateway API (successor to Ingress — preferred for new projects)

```yaml
# Gateway API — the K8s-native future (GA since K8s 1.27)
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: myapp-route
spec:
  parentRefs:
  - name: my-gateway
  hostnames: ["app.example.com"]
  rules:
  - matches:
    - path: { type: PathPrefix, value: /api }
    backendRefs:
    - name: api-svc
      port: 80
  - matches:
    - path: { type: PathPrefix, value: / }
    backendRefs:
    - name: web-svc
      port: 80
```

```
Why Traefik is the top choice (2026):
  ✅ Gateway API native support
  ✅ Built-in Let's Encrypt (no cert-manager needed)
  ✅ Built-in dashboard
  ✅ Dynamic config (no reload/restart)
  ✅ Middleware CRDs (rate limit, auth, headers)
  ✅ Active CNCF project, growing community
```

---

## 9. Probes — Health Checking

```
┌─────────────────────────────────────────────────────────────┐
│  Pod Startup                                                 │
│  ┌──────────────┐                                           │
│  │ Startup Probe│  "Is the app done initializing?"          │
│  │              │  While running: liveness + readiness off  │
│  │              │  Fail → kill + restart container          │
│  └──────┬───────┘                                           │
│         │ passes                                             │
│  ┌──────▼───────┐  ┌───────────────┐                       │
│  │Liveness Probe│  │Readiness Probe│                       │
│  │              │  │               │                        │
│  │"Is app alive │  │"Is app ready  │                        │
│  │ or deadlock?"│  │ for traffic?" │                        │
│  │              │  │               │                        │
│  │Fail → restart│  │Fail → remove  │                        │
│  │  container   │  │ from Service  │                        │
│  └──────────────┘  │ endpoints    │                        │
│                    └───────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

| Probe | Purpose | On Failure |
|-------|---------|------------|
| **Startup** | Slow-starting apps (DB migrations) | Kill + restart |
| **Liveness** | Detect deadlocks/hangs | Kill + restart |
| **Readiness** | App temporarily unavailable | Remove from Service (no traffic) |

```yaml
startupProbe:
  httpGet: { path: /healthz, port: 8080 }
  failureThreshold: 30      # 30 × 10s = 5 min to start
  periodSeconds: 10

livenessProbe:
  httpGet: { path: /healthz, port: 8080 }
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet: { path: /ready, port: 8080 }
  periodSeconds: 5
  failureThreshold: 1
```

---

## 10. Deployments — Rolling Updates & Rollback

```
Rolling Update Strategy:
  maxSurge: 1        (create 1 extra pod during update)
  maxUnavailable: 0  (never have fewer than desired)

  Before:  [v1] [v1] [v1]
  Step 1:  [v1] [v1] [v1] [v2]     ← surge: create v2 pod
  Step 2:  [v1] [v1] [v2]          ← terminate 1 v1 pod
  Step 3:  [v1] [v1] [v2] [v2]     ← surge: create v2 pod
  Step 4:  [v1] [v2] [v2]          ← terminate 1 v1 pod
  Step 5:  [v1] [v2] [v2] [v2]     ← surge: create v2 pod
  Step 6:  [v2] [v2] [v2]          ← done! zero downtime
```

```bash
# Update image
kubectl set image deployment/web web=myapp:v2

# Check rollout status
kubectl rollout status deployment/web

# Rollback to previous
kubectl rollout undo deployment/web

# Rollback to specific revision
kubectl rollout undo deployment/web --to-revision=3

# Pause/resume rollout
kubectl rollout pause deployment/web
kubectl rollout resume deployment/web

# History
kubectl rollout history deployment/web
```

### Deployment Strategies

| Strategy | How | Downtime? | Use Case |
|----------|-----|-----------|----------|
| **RollingUpdate** | Gradual replace old→new | No | Default, most apps |
| **Recreate** | Kill all old, start all new | Yes | DB schema changes |
| **Blue/Green** | Two full deployments, switch Service | No | Need instant rollback |
| **Canary** | Route % traffic to new version | No | Risk-sensitive releases |

---

## 11. ConfigMaps & Secrets

```
┌──────── ConfigMap ────────┐     ┌──────── Secret ─────────┐
│  Non-sensitive config     │     │  Sensitive data          │
│  Stored as plain text     │     │  Stored as base64        │
│                           │     │  (NOT encrypted at rest  │
│  db_host: postgres        │     │   unless you enable it!) │
│  log_level: info          │     │                          │
│                           │     │  password: cGFzc3dvcmQ=  │
└────────┬──────────────────┘     └──────────┬──────────────┘
         │                                    │
         └────── Injected into Pod as ────────┘
                 1. Environment variables
                 2. Mounted files in a volume
```

```yaml
# As env vars
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef: { name: app-config, key: db_host }
- name: DB_PASS
  valueFrom:
    secretKeyRef: { name: app-secret, key: password }

# As mounted files
volumeMounts:
- name: config-vol
  mountPath: /etc/config
volumes:
- name: config-vol
  configMap: { name: app-config }
```

---

## 12. Storage — PV, PVC, StorageClass

```
┌──────── StorageClass ──────────┐
│  provisioner: disk.csi.azure   │  (tells K8s HOW to create storage)
│  reclaimPolicy: Retain         │
│  volumeBindingMode: WaitFor... │
└──────────┬─────────────────────┘
           │ dynamically provisions
┌──────────▼─────────────────────┐
│  PersistentVolume (PV)         │  (the actual disk / NFS / cloud volume)
│  capacity: 10Gi                │
│  accessModes: ReadWriteOnce    │
└──────────┬─────────────────────┘
           │ bound to
┌──────────▼─────────────────────┐
│  PersistentVolumeClaim (PVC)   │  (Pod's "request" for storage)
│  requests: 10Gi                │
│  storageClassName: fast-ssd    │
└──────────┬─────────────────────┘
           │ mounted by
┌──────────▼─────────────────────┐
│  Pod                           │
│  volumes:                      │
│  - name: data                  │
│    persistentVolumeClaim:      │
│      claimName: my-pvc         │
└────────────────────────────────┘
```

| Access Mode | Short | Description |
|-------------|-------|------------|
| ReadWriteOnce | RWO | One node reads/writes |
| ReadOnlyMany | ROX | Many nodes read |
| ReadWriteMany | RWX | Many nodes read/write (NFS, Azure Files) |

| Reclaim Policy | What Happens When PVC Deleted |
|----------------|-------------------------------|
| Retain | PV kept (manual cleanup) |
| Delete | PV and underlying storage deleted |
| Recycle | Deprecated — basic rm -rf |

---

## 13. Networking — CNI & Network Policies

### K8s Networking Model (4 requirements)

```
1. Every pod gets its OWN IP address
2. Pods on same node can communicate without NAT
3. Pods on different nodes can communicate without NAT
4. Agents on a node can communicate with all pods on that node
```

### CNI (Container Network Interface)

```
kubelet ──► CNI Plugin ──► configures pod networking

Popular CNI plugins:
  Calico    — L3 networking + NetworkPolicy (BGP or VXLAN)
  Flannel   — Simple L2 overlay (VXLAN), no NetworkPolicy
  Cilium    — eBPF-based, L3/L4/L7 policies, observability
  Weave     — Mesh network, encrypted
  Azure CNI — Azure-native, pods get VNet IPs
```

### NetworkPolicy (Firewall Rules)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-api
spec:
  podSelector:
    matchLabels: { app: db }          # Apply to db pods
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector:
        matchLabels: { app: api }     # Only api pods can talk to db
    ports:
    - port: 5432
```

---

## 14. Scaling

### Horizontal Pod Autoscaler (HPA)

```
              metrics-server
                  │
                  ▼
  HPA checks CPU/memory every 15s
  ┌─────────────────────────────────────────┐
  │  Current: 3 pods, avg CPU = 80%         │
  │  Target:  50% CPU                       │
  │  Desired: ceil(3 × 80/50) = 5 pods     │
  │  Action:  scale up to 5                 │
  └─────────────────────────────────────────┘
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target: { type: Utilization, averageUtilization: 50 }
```

### VPA vs HPA

| | HPA | VPA |
|--|-----|-----|
| Scales | Number of pods | Pod resource requests |
| Direction | Horizontal (more pods) | Vertical (bigger pods) |
| Best for | Stateless apps | Stateful, single-instance |
| Can combine? | Yes, but not on same metric | — |

---

## 15. RBAC — Role-Based Access Control

```
┌─── Who ────┐    ┌─── What ────────┐    ┌─── Where ──────┐
│ Subject     │    │ Role/ClusterRole│    │ Namespace /    │
│             │    │                 │    │ Cluster-wide   │
│ User        │    │ Verbs:          │    │                │
│ Group       │────│ get, list,      │────│ RoleBinding    │
│ ServiceAcct │    │ create, update, │    │ (namespace)    │
│             │    │ delete, watch   │    │                │
│             │    │                 │    │ ClusterRole-   │
│             │    │ Resources:      │    │ Binding        │
│             │    │ pods, services, │    │ (cluster-wide) │
│             │    │ deployments...  │    │                │
└─────────────┘    └─────────────────┘    └────────────────┘
```

```yaml
# Role: can read pods in "production" namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]

# Bind role to user
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: User
  name: developer@example.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 16. Helm — Package Manager

```
mychart/
├── Chart.yaml          # Name, version, description
├── values.yaml         # Default configurable values
├── templates/          # K8s manifests with Go templates
│   ├── deployment.yaml # {{ .Values.replicas }}
│   ├── service.yaml    # {{ .Values.service.type }}
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Reusable template snippets
│   └── NOTES.txt       # Post-install message
├── charts/             # Sub-chart dependencies
└── values-prod.yaml    # Environment-specific overrides
```

```bash
helm install myapp ./mychart                    # Install
helm install myapp ./mychart -f values-prod.yaml # With overrides
helm upgrade myapp ./mychart                     # Upgrade
helm rollback myapp 1                            # Rollback
helm list                                        # List releases
helm uninstall myapp                             # Remove
```

---

## 17. Complete Manifest Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels: { app: myapp }
spec:
  replicas: 3
  selector:
    matchLabels: { app: myapp }
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
  template:
    metadata:
      labels: { app: myapp }
    spec:
      containers:
      - name: myapp
        image: registry/myapp:v1.2.3
        ports: [{ containerPort: 8080 }]
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef: { name: myapp-config, key: db_host }
        - name: DB_PASS
          valueFrom:
            secretKeyRef: { name: myapp-secret, key: password }
        resources:
          requests: { cpu: 100m, memory: 128Mi }
          limits: { cpu: 500m, memory: 512Mi }
        startupProbe:
          httpGet: { path: /healthz, port: 8080 }
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet: { path: /healthz, port: 8080 }
          periodSeconds: 10
        readinessProbe:
          httpGet: { path: /ready, port: 8080 }
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata: { name: myapp-svc }
spec:
  type: ClusterIP
  selector: { app: myapp }
  ports: [{ port: 80, targetPort: 8080 }]
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  ingressClassName: traefik         # traefik (nginx-ingress retired Mar 2026)
  tls: [{ hosts: [app.example.com], secretName: tls-secret }]
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: { name: myapp-svc, port: { number: 80 } }
```

---

## 18. Troubleshooting Flowchart

```
Pod Issue?
  │
  ├─ ImagePullBackOff
  │    → Wrong image name/tag? Registry auth? Network?
  │    → kubectl describe pod → check Events section
  │
  ├─ Pending
  │    → No resources? (kubectl describe → "Insufficient cpu/memory")
  │    → Node selector mismatch? Taint without toleration?
  │    → PVC not bound? (kubectl get pvc)
  │
  ├─ CrashLoopBackOff
  │    → kubectl logs <pod> --previous
  │    → OOMKilled? Increase memory limit
  │    → App crash? Fix code, check config
  │    → Missing env/config? Check ConfigMap/Secret
  │
  ├─ Running but no traffic
  │    → Service selector matches pod labels?
  │    → kubectl get endpoints <svc> (should show pod IPs)
  │    → Ingress rules correct?
  │    → NetworkPolicy blocking?
  │
  └─ General debugging
       → kubectl describe pod/svc/deploy
       → kubectl logs <pod> -f
       → kubectl exec -it <pod> -- sh
       → kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## 19. Key kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide

# Resources
kubectl get pods/svc/deploy/ing -n <ns> -o wide
kubectl describe pod <name>
kubectl logs <pod> -f --previous

# Debugging
kubectl exec -it <pod> -- sh
kubectl port-forward svc/myapp 8080:80
kubectl top pods/nodes

# Apply & manage
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl scale deploy/web --replicas=5
kubectl set image deploy/web web=myapp:v2

# Context
kubectl config get-contexts
kubectl config use-context prod-cluster
kubectl config set-context --current --namespace=prod
```
