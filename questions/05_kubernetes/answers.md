# Kubernetes — COMPREHENSIVE ANSWERS (All 134 Questions)

---

## BASICS & WORKLOADS (35 Qs)

### Fundamentals

**1. What is Kubernetes?**

Open-source container orchestration platform. Automates deployment, scaling, self-healing of containerized apps. Originally by Google (Borg → K8s), maintained by CNCF.

```
What Kubernetes does:
  ┌─────────────────────────────────────────────────────┐
  │  You tell K8s: "I want 3 replicas of my app"       │
  │  K8s ensures: 3 replicas are ALWAYS running         │
  │                                                     │
  │  Pod crashes?    → K8s restarts it automatically    │
  │  Node dies?      → K8s reschedules pods elsewhere   │
  │  Traffic spikes? → K8s scales up (HPA)              │
  │  New version?    → K8s rolls out with zero downtime │
  └─────────────────────────────────────────────────────┘
```

---

**2. Container orchestration? Why not just Docker?**

Docker runs containers on ONE machine. In production you have 50+ containers across multiple servers — you need:

```
Docker alone:                      With Kubernetes:
┌──────────────────────┐          ┌──────────────────────────┐
│ Node 1: docker run   │          │ Automated placement      │
│ Node 2: docker run   │          │ Self-healing             │
│ Node 3: docker run   │          │ Rolling updates          │
│                      │          │ Load balancing           │
│ Manual everything:   │          │ Service discovery (DNS)  │
│ - Placement ❌       │          │ Scaling (HPA/VPA)        │
│ - Health checks ❌   │          │ Secret management        │
│ - Scaling ❌         │          │ Config management        │
│ - Updates ❌         │          │ Storage orchestration    │
│ - Recovery ❌        │          │ All automatic! ✅        │
└──────────────────────┘          └──────────────────────────┘
```

---

**3. K8s architecture?**

```
┌─── Kubernetes Cluster ─────────────────────────────────────────────────┐
│                                                                         │
│  ┌─── Control Plane ──────────────────────────────────────────────┐    │
│  │                                                                 │    │
│  │  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌──────────────┐ │    │
│  │  │ API      │  │ etcd     │  │ Scheduler │  │ Controller   │ │    │
│  │  │ Server   │  │ (state   │  │ (place    │  │ Manager      │ │    │
│  │  │ (front   │  │  store)  │  │  pods on  │  │ (reconcile   │ │    │
│  │  │  door)   │  │          │  │  nodes)   │  │  desired vs  │ │    │
│  │  │          │  │          │  │           │  │  actual)     │ │    │
│  │  └────┬─────┘  └──────────┘  └───────────┘  └──────────────┘ │    │
│  └───────┼─────────────────────────────────────────────────────────┘    │
│          │                                                              │
│  ┌───────▼─── Worker Node 1 ──────┐  ┌─── Worker Node 2 ──────────┐  │
│  │  ┌─────────┐  ┌──────────────┐ │  │  ┌─────────┐  ┌──────────┐ │  │
│  │  │ kubelet │  │ kube-proxy   │ │  │  │ kubelet │  │kube-proxy│ │  │
│  │  └────┬────┘  └──────────────┘ │  │  └────┬────┘  └──────────┘ │  │
│  │  ┌────▼────┐                   │  │  ┌────▼────┐               │  │
│  │  │Container│                   │  │  │Container│               │  │
│  │  │ Runtime │                   │  │  │ Runtime │               │  │
│  │  │(contd)  │                   │  │  │(contd)  │               │  │
│  │  └─────────┘                   │  │  └─────────┘               │  │
│  │  ┌──────┐ ┌──────┐ ┌──────┐  │  │  ┌──────┐ ┌──────┐        │  │
│  │  │Pod A │ │Pod B │ │Pod C │  │  │  │Pod D │ │Pod E │        │  │
│  │  └──────┘ └──────┘ └──────┘  │  │  └──────┘ └──────┘        │  │
│  └────────────────────────────────┘  └────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

---

**4. Control plane components?**

| Component | Role | Analogy |
|-----------|------|---------|
| **API Server** | Front door — ALL requests go through here | Reception desk |
| **etcd** | Distributed key-value store — ALL cluster state | Database |
| **Scheduler** | Watches for unscheduled pods, picks best node | Job placement officer |
| **Controller Manager** | Runs control loops — ensures desired = actual state | Supervisor |

```
How they work together:
  kubectl apply ──► API Server ──► validates ──► stores in etcd
                         │
                    Scheduler watches ──► assigns pod to node
                         │
                    Controller Manager watches ──► ensures replicas match
                         │
                    kubelet on node ──► pulls image, starts container
```

---

**5. kubelet?**

Agent running on **every worker node**. Receives pod specs from API server, ensures containers are running and healthy. Reports node status back. Does NOT manage containers not created by K8s.

---

**6. kube-proxy?**

Network proxy on each node. Maintains iptables/IPVS rules to route traffic to correct pods. Implements the Service abstraction (virtual IP → actual pod IPs).

```
Service IP: 10.96.0.10:80
           │
    kube-proxy creates rules:
           │
    ┌──────▼──────┐
    │  iptables/  │
    │  IPVS rules │
    └──────┬──────┘
     ┌─────┼─────┐
     ▼     ▼     ▼
  Pod A  Pod B  Pod C    (round-robin load balancing)
```

---

**7. etcd?**

Distributed key-value store holding **entire cluster state** — pods, services, secrets, configmaps, RBAC, everything.

```
Critical facts:
  - If etcd is lost → you lose ALL cluster state
  - Must be backed up regularly (etcdctl snapshot save)
  - Runs on control plane (or separate nodes for HA)
  - Uses Raft consensus for distributed consistency
  - Only API Server communicates directly with etcd
```

---

**8. API server?**

Central management point. ALL interactions go through it:

```
kubectl ──────────►│
kubelet ──────────►│ API Server ──► AuthN ──► AuthZ ──► Admission ──► etcd
Controllers ──────►│              (who?)   (allowed?) (mutate/validate)
Dashboard ────────►│
```

---

**9. kubectl? 10 daily commands?**

```bash
kubectl get pods -o wide                    # Pods + node + IP
kubectl get pods --all-namespaces           # ALL namespaces
kubectl describe pod <pod>                  # Detailed info + events
kubectl logs <pod> -f                       # Follow logs
kubectl logs <pod> --previous               # Previous crash logs
kubectl exec -it <pod> -- /bin/sh           # Shell into pod
kubectl apply -f manifest.yaml              # Apply configuration
kubectl delete pod <pod>                    # Delete pod
kubectl get events --sort-by=.lastTimestamp # Recent events
kubectl top pods                            # Resource usage (needs metrics-server)
```

---

**10. Namespace?**

Virtual cluster within a cluster for logical isolation.

```
┌─── Cluster ─────────────────────────────────────┐
│                                                   │
│  ┌─── default ──────┐  ┌─── kube-system ───┐   │
│  │ user workloads   │  │ K8s components    │   │
│  │ (when no ns set) │  │ CoreDNS, kube-    │   │
│  └──────────────────┘  │ proxy, metrics    │   │
│                         └───────────────────┘   │
│  ┌─── team-a ───────┐  ┌─── team-b ────────┐  │
│  │ Team A's apps    │  │ Team B's apps     │  │
│  │ ResourceQuota    │  │ ResourceQuota     │  │
│  │ NetworkPolicy    │  │ NetworkPolicy     │  │
│  └──────────────────┘  └───────────────────┘  │
└───────────────────────────────────────────────────┘
```

Default namespaces: `default`, `kube-system`, `kube-public`, `kube-node-lease`.

---

**11. What is a node? Add node?**

A node is a worker machine (VM or physical). Add:
```bash
# kubeadm (self-managed)
kubeadm join <api-server>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

# Managed K8s (AKS) — scale node pool
az aks nodepool scale --resource-group rg --cluster-name aks --name nodepool1 --node-count 5
```

---

**12. Managed vs self-managed K8s?**

| Aspect | Managed (AKS/EKS/GKE) | Self-managed (kubeadm) |
|--------|------------------------|------------------------|
| Control plane | Cloud manages | You manage |
| Upgrades | Automated / one-click | Manual |
| IAM integration | Built-in | Configure yourself |
| Networking | Cloud CNI | You choose CNI |
| Cost | Cloud pricing | Your hardware |
| Maintenance | Low | High |
| Best for | Most teams | Full control needed |

---

### Pods

**13. What is a Pod?**

Smallest deployable unit in K8s. One or more containers sharing:

```
┌─── Pod ─────────────────────────────────────┐
│                                              │
│  ┌───────────────┐  ┌───────────────┐       │
│  │ App Container │  │ Sidecar       │       │
│  │ (main app)    │  │ (logging/     │       │
│  │               │  │  monitoring)  │       │
│  └───────┬───────┘  └───────┬───────┘       │
│          │                   │               │
│  ┌───────▼───────────────────▼───────┐      │
│  │ Shared Network Namespace          │      │
│  │ (same IP, localhost, same ports)  │      │
│  └───────────────────────────────────┘      │
│  ┌───────────────────────────────────┐      │
│  │ Shared Volumes                    │      │
│  └───────────────────────────────────┘      │
│                                              │
│  Pod IP: 10.244.1.5                         │
└──────────────────────────────────────────────┘
```

Usually **1 container per pod**. Multiple only for tightly coupled helpers (sidecars).

---

**14. Multiple containers in a Pod?**

```
Sidecar Pattern:          Adapter Pattern:         Ambassador Pattern:
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│ App + Log Agent │      │ App + Format    │      │ App + DB Proxy  │
│                 │      │    Converter    │      │                 │
│ App writes logs │      │ App outputs     │      │ App connects to │
│ Agent ships to  │      │ custom format   │      │ localhost:5432  │
│ central logging │      │ Adapter converts│      │ Proxy routes to │
│                 │      │ to standard     │      │ correct DB      │
└─────────────────┘      └─────────────────┘      └─────────────────┘
```

---

**15. Sidecar container? 3 examples.**

1. **Log collector**: Fluent Bit sidecar shipping logs to Loki/ELK
2. **Service mesh proxy**: Istio Envoy sidecar handling mTLS + traffic
3. **Config syncer**: Sidecar that watches ConfigMap changes and reloads app config

---

**16. Init container?**

Runs to completion **before** app containers start. Sequential if multiple.

```yaml
initContainers:
- name: wait-for-db
  image: busybox
  command: ['sh', '-c', 'until nc -z postgres 5432; do sleep 2; done']
- name: run-migrations
  image: myapp:latest
  command: ['python', 'manage.py', 'migrate']
```

Use cases: wait for dependency, clone repo, run DB migrations, set permissions.

---

**17. Pod lifecycle?**

```
                                  ┌─────────┐
  kubectl apply ──► Pending ──► │ Running │ ──► Succeeded (Jobs)
                      │          └────┬────┘        or
                      │               │          Failed (crash)
                      │               │
                   Reasons:        Reasons:      ┌─────────┐
                   - Scheduling    - OOMKilled   │ Unknown │
                   - Image pull    - Error       │(node    │
                   - No resources  - App crash   │ lost)   │
                                                  └─────────┘
```

| Phase | Meaning |
|-------|---------|
| Pending | Accepted but not scheduled, or images downloading |
| Running | At least one container running |
| Succeeded | All containers exited 0 (Jobs/batch) |
| Failed | At least one container exited non-zero |
| Unknown | Node communication lost |

---

**18. Get logs from Pod? Previous crash?**

```bash
kubectl logs <pod>                        # Current logs
kubectl logs <pod> -c <container>         # Specific container in multi-container pod
kubectl logs <pod> --previous             # Previous crashed container instance
kubectl logs <pod> --since=1h             # Last hour
kubectl logs <pod> --tail=100             # Last 100 lines
kubectl logs <pod> -f                     # Follow (stream)
kubectl logs -l app=myapp --all-containers  # All pods with label
```

---

**19. Exec into Pod?**

```bash
kubectl exec -it <pod> -- /bin/sh         # Interactive shell
kubectl exec -it <pod> -c <container> -- bash  # Specific container
kubectl exec <pod> -- cat /etc/config     # One-off command
kubectl exec <pod> -- env                 # Check environment variables
```

---

**20. What happens when Pod is deleted?**

```
kubectl delete pod myapp
  │
  ├─ 1. Pod enters "Terminating" state
  ├─ 2. Removed from Service endpoints (no new traffic)
  ├─ 3. preStop hook runs (if defined)
  ├─ 4. SIGTERM sent to containers
  ├─ 5. Wait terminationGracePeriodSeconds (default 30s)
  ├─ 6. SIGKILL sent (if still running)
  └─ 7. Pod removed from API server + etcd
```

---

### Workloads

**21. Deployment?**

Manages ReplicaSets which manage Pods. The **go-to** for stateless apps.

```
Deployment (declarative spec)
    │
    └──► ReplicaSet (ensures N replicas)
              │
              ├──► Pod 1
              ├──► Pod 2
              └──► Pod 3

Provides: rolling updates, rollback, scaling, self-healing
```

---

**22. ReplicaSet vs Deployment?**

| ReplicaSet | Deployment |
|------------|------------|
| Ensures N pod replicas | Manages ReplicaSets |
| No rolling updates | Rolling updates + rollback |
| No revision history | Revision history |
| **Never create directly** | **Always use this** |

---

**23. DaemonSet? 3 use cases.**

Ensures **one pod on every node** (or selected nodes).

```
┌── Node 1 ──┐  ┌── Node 2 ──┐  ┌── Node 3 ──┐
│ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │
│ │DaemonSet│ │  │ │DaemonSet│ │  │ │DaemonSet│ │
│ │  Pod   │ │  │ │  Pod   │ │  │ │  Pod   │ │
│ └────────┘ │  │ └────────┘ │  │ └────────┘ │
└────────────┘  └────────────┘  └────────────┘
New node added? DaemonSet pod auto-created!
```

1. **Log collector**: Fluent Bit/Fluentd on every node
2. **Monitoring agent**: Node Exporter (Prometheus) on every node
3. **Network plugin**: Calico/Cilium CNI agent on every node

---

**24. StatefulSet vs Deployment?**

| Aspect | Deployment | StatefulSet |
|--------|-----------|-------------|
| For | Stateless apps | Stateful apps (DB, Kafka, ZK) |
| Pod names | Random (myapp-x7k2p) | Ordered (myapp-0, myapp-1) |
| Pod identity | Interchangeable | Stable, unique identity |
| Storage | Shared or none | Persistent per pod (PVC template) |
| Scaling | Parallel | Ordered (0→1→2) |
| DNS | Via Service only | Individual: `pod-0.svc.ns.svc.cluster.local` |

---

**25. Job vs CronJob?**

```
Job:      Run once ──► Complete ──► Done
CronJob:  Schedule ──► Create Job ──► Complete ──► Wait ──► Repeat
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "0 0 * * *"          # midnight daily (cron syntax)
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: myapp:latest
            command: ["python", "cleanup.py"]
          restartPolicy: OnFailure
```

---

**26. Rolling update vs Recreate?**

```
RollingUpdate (default):               Recreate:
┌───────────────────────┐              ┌───────────────────────┐
│ v1 v1 v1              │              │ v1 v1 v1              │
│ v1 v1 v2 ← new pod   │              │ (all killed)          │
│ v1 v2 v2              │              │ v2 v2 v2 ← all new   │
│ v2 v2 v2              │              │                       │
│                       │              │                       │
│ Zero downtime ✅      │              │ Brief downtime ❌     │
│ Two versions coexist  │              │ Clean cutover         │
└───────────────────────┘              └───────────────────────┘
```

Use Recreate when app can't run two versions simultaneously (e.g., DB schema incompatibility).

---

**27. maxSurge and maxUnavailable?**

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          # Max extra pods during update
    maxUnavailable: 0    # Never fewer than desired running

# With replicas=3, maxSurge=1, maxUnavailable=0:
#   Step 1: 3 old + 1 new = 4 total (surge)
#   Step 2: 2 old + 2 new = 4 total
#   Step 3: 1 old + 3 new = 4 total
#   Step 4: 0 old + 3 new = 3 total (done)
```

Default: maxSurge=25%, maxUnavailable=25%.

---

**28. Rollback a Deployment?**

```bash
kubectl rollout undo deployment/myapp                    # Previous revision
kubectl rollout undo deployment/myapp --to-revision=3    # Specific revision
kubectl rollout history deployment/myapp                 # View history
kubectl rollout status deployment/myapp                  # Check status
kubectl rollout pause deployment/myapp                   # Pause rollout
kubectl rollout resume deployment/myapp                  # Resume
```

---

**29. Scale a Deployment?**

```bash
# Manual
kubectl scale deployment/myapp --replicas=5

# Auto (HPA — Horizontal Pod Autoscaler)
kubectl autoscale deployment/myapp --min=2 --max=10 --cpu-percent=70
```

---

**30. Horizontal Pod Autoscaler (HPA)?**

```
                    CPU > 70%?
                        │
            ┌───────────▼───────────┐
            │  HPA checks metrics   │ (every 15s)
            │  via metrics-server   │
            └───────────┬───────────┘
                        │
              ┌─────────▼─────────┐
              │ Scale up replicas │
              │ 3 → 5 → 8 → 10   │
              └─────────┬─────────┘
                        │
              CPU < 70% for 5min?
                        │
              ┌─────────▼─────────┐
              │ Scale down         │
              │ 10 → 5 → 3 → 2   │
              └───────────────────┘
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

### Interview-Style (Basics)

**31. Deployment manifest for nginx with 3 replicas:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 5
```

---

**32. Pod in Pending state — 5 reasons?**

```
Pending Pod Diagnosis:
┌──────────────────────────────────────────────────┐
│ 1. Insufficient resources (CPU/memory)           │
│    → kubectl describe pod → "Insufficient cpu"   │
│    → Scale up node pool or reduce requests       │
│                                                   │
│ 2. Node selector/affinity mismatch               │
│    → Pod requires label that no node has          │
│    → Fix nodeSelector or label nodes              │
│                                                   │
│ 3. PVC not bound (storage unavailable)            │
│    → kubectl get pvc → check status               │
│    → Check StorageClass exists                    │
│                                                   │
│ 4. Image pulling (large image / slow registry)    │
│    → kubectl describe pod → "Pulling image"       │
│    → Check image name, registry access            │
│                                                   │
│ 5. Taints preventing scheduling                   │
│    → Node has taint, pod has no toleration         │
│    → Add toleration or remove taint               │
└──────────────────────────────────────────────────┘
```

---

**33. CrashLoopBackOff debugging:**

```
CrashLoopBackOff = container keeps crashing and K8s keeps restarting

Debug Flow:
  kubectl logs <pod> --previous     ← #1: Check WHY it crashed
        │
  kubectl describe pod <pod>        ← #2: Check events (OOMKilled?)
        │
  kubectl get cm,secret -n <ns>    ← #3: Missing config/secret?
        │
  docker run -it <image> sh        ← #4: Test image locally
        │
  Common causes:
    - Application error / exception
    - OOMKilled (memory limit too low)
    - Missing env var or config
    - Wrong CMD / entrypoint
    - Port conflict
    - Permission denied (non-root user)
```

---

**34. Zero-downtime deployments?**

```
5 Requirements for Zero Downtime:
  ┌──────────────────────────────────────────────┐
  │ 1. RollingUpdate + maxUnavailable=0          │
  │ 2. readinessProbe (traffic only when ready)  │
  │ 3. preStop hook: sleep 5 (drain connections) │
  │ 4. terminationGracePeriodSeconds: 30+        │
  │ 5. PodDisruptionBudget (min available)       │
  └──────────────────────────────────────────────┘
```

---

**35. URL → K8s app flow:**

```
User types https://app.example.com
    │
┌───▼────────────────┐
│ DNS Resolution     │ app.example.com → Load Balancer IP
└───┬────────────────┘
    │
┌───▼────────────────┐
│ Cloud Load Balancer│ (Azure LB / AWS ALB)
└───┬────────────────┘
    │
┌───▼────────────────┐
│ Ingress Controller │ (nginx pod in cluster)
│ Matches host/path  │
└───┬────────────────┘
    │
┌───▼────────────────┐
│ Service            │ (ClusterIP — virtual IP)
│ Label selector     │ → selects matching pods
└───┬────────────────┘
    │
┌───▼────────────────┐
│ kube-proxy         │ iptables/IPVS rules
│ Routes to pod IP   │
└───┬────────────────┘
    │
┌───▼────────────────┐
│ Pod                │ Container processes request
└────────────────────┘
```

---

## NETWORKING & SERVICES (34 Qs)

### Service Types

**1. What is a Service?**

Stable networking abstraction over ephemeral pods. Pods come and go — Service provides a stable IP and DNS name.

```
Without Service:                   With Service:
┌────────────────────┐            ┌────────────────────┐
│ Pod IP changes     │            │ Service: 10.96.0.10│
│ every restart!     │            │ DNS: myapp-svc     │
│                    │            │      │              │
│ Client must track  │            │ ┌────▼────┐        │
│ all pod IPs ❌     │            │ │ Pod A   │        │
│                    │            │ │ Pod B   │        │
│                    │            │ │ Pod C   │        │
│                    │            │ └─────────┘        │
│                    │            │ Stable endpoint ✅  │
└────────────────────┘            └────────────────────┘
```

---

**2-8. Service Types:**

```
┌─── ClusterIP (default) ──────────────────────────────────────┐
│  Internal only — accessible within cluster                    │
│  Gets virtual IP from service CIDR (e.g., 10.96.0.10)       │
│  DNS: myapp.namespace.svc.cluster.local                      │
│  Use: inter-service communication                            │
└──────────────────────────────────────────────────────────────┘

┌─── NodePort ─────────────────────────────────────────────────┐
│  Exposes on every node's IP at a static port (30000-32767)   │
│  External access: <NodeIP>:<NodePort>                        │
│  Use: dev/testing, on-prem without cloud LB                  │
└──────────────────────────────────────────────────────────────┘

┌─── LoadBalancer ─────────────────────────────────────────────┐
│  Cloud provider provisions external load balancer            │
│  Gets public IP automatically                                │
│  Use: production external services                           │
└──────────────────────────────────────────────────────────────┘

┌─── ExternalName ─────────────────────────────────────────────┐
│  DNS CNAME record pointing to external service               │
│  No proxy, no port — just DNS alias                          │
│  Use: map external DB to internal DNS name                   │
└──────────────────────────────────────────────────────────────┘
```

| Type | Scope | Port Range | Cloud Required? |
|------|-------|------------|-----------------|
| ClusterIP | Internal | Any | No |
| NodePort | External | 30000-32767 | No |
| LoadBalancer | External | Any | Yes |
| ExternalName | DNS alias | N/A | No |

---

**9. Endpoints object?**

Auto-created for each Service. Contains pod IPs matching the Service's selector. Updated as pods are created/destroyed.

```bash
kubectl get endpoints myapp-svc
# NAME        ENDPOINTS                               AGE
# myapp-svc   10.244.1.5:8080,10.244.2.3:8080        5m
```

If endpoints list is empty → **selector doesn't match any pods** (label mismatch).

---

**10. Headless Service?**

Service with `clusterIP: None`. No load balancing, no virtual IP. DNS returns individual pod IPs.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None          # ← Headless!
  selector:
    app: postgres
  ports:
  - port: 5432
# DNS returns: pod-0.db-headless.ns.svc.cluster.local → 10.244.1.5
#              pod-1.db-headless.ns.svc.cluster.local → 10.244.2.3
```

Used with StatefulSets where clients need specific pods.

---

**11. Service manifests:**

```yaml
# ─── ClusterIP ───
apiVersion: v1
kind: Service
metadata:
  name: api-internal
spec:
  type: ClusterIP
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8080
---
# ─── NodePort ───
apiVersion: v1
kind: Service
metadata:
  name: api-nodeport
spec:
  type: NodePort
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
---
# ─── LoadBalancer ───
apiVersion: v1
kind: Service
metadata:
  name: api-lb
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8080
```

---

### Ingress

**12-16. What is Ingress?**

L7 (HTTP) load balancer. Routes external traffic to internal Services based on host and path.

> **⚠️ NOTE (2026):** The `kubernetes/ingress-nginx` controller was **retired and archived in March 2026**. No further releases or security patches. New projects should use **Traefik**, **AGIC**, or **Gateway API**. See Q20a–20g below for details.

```
Internet ──► Ingress Controller (Traefik / AGIC / etc.)
                    │
             ┌──────┴──────┐
             │  Ingress     │
             │  Rules       │
             └──────┬──────┘
               ┌────┼────┐
               ▼    ▼    ▼
          /api   /web   /auth
          svc    svc    svc
```

---

**17. TLS/SSL with Ingress?**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret       # cert-manager auto-creates this
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
```

Use **cert-manager** for automatic Let's Encrypt certificate management.

---

**18. Ingress annotations? 5 examples.**

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /              # URL rewriting
  nginx.ingress.kubernetes.io/ssl-redirect: "true"           # Force HTTPS
  nginx.ingress.kubernetes.io/proxy-body-size: "50m"         # Max upload
  nginx.ingress.kubernetes.io/rate-limit: "10"               # Rate limiting
  cert-manager.io/cluster-issuer: letsencrypt-prod           # Auto TLS
```

---

**19. Ingress with TLS and 2 paths:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts: [app.example.com]
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

---

**20. Ingress vs Gateway API?**

| Aspect | Ingress | Gateway API |
|--------|---------|-------------|
| Age | Older, stable | Newer, GA since K8s 1.27 |
| Layer | L7 HTTP only | L4 (TCP/UDP) + L7 |
| RBAC | Single resource | Split: Gateway (infra) + Route (dev) |
| Features | Basic | Traffic splitting, header matching, mirrors |
| Status | Maintenance mode | Active development |

---

### NGINX Ingress Controller Retirement & Alternatives

**20a. What happened to NGINX Ingress Controller?**

```
NGINX Ingress Controller (kubernetes/ingress-nginx) — RETIRED:

  Timeline:
  ┌──────────────────────────────────────────────────────────────────┐
  │  Nov 2025   Retirement announced by Kubernetes project          │
  │  Mar 2026   Repository archived on GitHub (read-only)           │
  │  Post-Mar   No further releases, no bugfixes, NO security       │
  │  2026       patches — even for critical CVEs                    │
  └──────────────────────────────────────────────────────────────────┘

  What this means:
  ✅ Existing deployments still work (not removed from clusters)
  ✅ Helm charts + container images remain available
  ❌ No new features or bug fixes
  ❌ No security vulnerability patches going forward
  ❌ New projects should NOT use ingress-nginx
  ❌ Existing users should plan migration

  Official recommendation:
  "If you are not already using ingress-nginx, you should NOT be
   deploying it. Instead, identify a Gateway API implementation."
```

**Why was it retired?**
- The Ingress API itself is now considered limited — Gateway API is its successor
- NGINX Inc. (F5) shifted focus to their commercial NGINX products
- The open-source community maintainer pool shrank
- Security concerns: the project assumed all Ingress-creating users are cluster admins (unsafe in multi-tenant environments)

---

**20b. What are the alternatives to NGINX Ingress Controller?**

```
Ingress Controller Alternatives (2026):

  ┌─────────────────────────────────────────────────────────────────────┐
  │                                                                     │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐ │
  │  │   Traefik     │  │ Azure AGIC   │  │ Gateway API (standard)  │ │
  │  │              │  │              │  │                          │ │
  │  │ ★ Most       │  │ ★ Azure-     │  │ ★ K8s-native successor  │ │
  │  │   popular    │  │   native     │  │   to Ingress API        │ │
  │  │   replacement│  │   managed    │  │                          │ │
  │  └──────────────┘  └──────────────┘  └──────────────────────────┘ │
  │                                                                     │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐ │
  │  │ Envoy/       │  │ HAProxy      │  │ Kong                    │ │
  │  │ Contour      │  │ Ingress      │  │ Ingress Controller      │ │
  │  │              │  │              │  │                          │ │
  │  │ ★ Envoy-     │  │ ★ HAProxy    │  │ ★ API Gateway +         │ │
  │  │   based,     │  │   based,     │  │   Ingress combined     │ │
  │  │   CNCF       │  │   enterprise │  │                          │ │
  │  └──────────────┘  └──────────────┘  └──────────────────────────┘ │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

---

**20c. What is Traefik and why is everyone moving to it?**

Traefik is a modern, cloud-native reverse proxy and ingress controller. It's the **most popular replacement** for nginx-ingress because:

```
Why Traefik Is Winning:

  1. Gateway API support (native, first-class)
  2. Auto-discovery — watches K8s API, auto-configures routes
  3. Built-in Let's Encrypt (automatic TLS, no cert-manager needed)
  4. Dashboard — real-time traffic visualization out of the box
  5. Middleware system — rate limiting, auth, headers, retry, circuit breaker
  6. Multi-protocol — HTTP, TCP, UDP, gRPC, WebSocket
  7. No reload needed — dynamic config, no NGINX-style reload/restart
  8. Active development + large community (CNCF project)
  9. Easy migration from nginx-ingress (IngressRoute CRD or standard Ingress)
  10. Lightweight — single binary, low resource footprint
```

```yaml
# Traefik IngressRoute (CRD — Traefik-native way):
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: myapp
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`app.example.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: api-svc
          port: 80
      middlewares:
        - name: rate-limit
    - match: Host(`app.example.com`)
      kind: Rule
      services:
        - name: web-svc
          port: 80
  tls:
    certResolver: letsencrypt        # Auto TLS — no cert-manager needed!
```

```yaml
# Traefik Middleware example (rate limiting + auth):
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit
spec:
  rateLimit:
    average: 100
    burst: 200
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: basic-auth
spec:
  basicAuth:
    secret: auth-secret
```

```
Traefik vs NGINX Ingress:

  Feature               NGINX Ingress        Traefik
  ─────────────────────────────────────────────────────────
  Status                RETIRED (Mar 2026)   Active (CNCF)
  Gateway API           ❌ Never added        ✅ Native
  Auto TLS (ACME)       ❌ Needs cert-manager ✅ Built-in
  Dynamic config        ❌ Requires reload     ✅ Hot reload
  Dashboard             ❌ No                  ✅ Built-in
  Middleware CRDs       ❌ Annotations only    ✅ First-class
  TCP/UDP routing       ⚠️  Limited             ✅ Full support
  Multi-protocol        ⚠️  HTTP mainly         ✅ HTTP/TCP/UDP/gRPC
  Config approach       Annotations           CRDs + labels
  Learning curve        Low                   Low-medium
  Community             Declining             Growing
```

---

**20d. What is AGIC (Azure Application Gateway Ingress Controller)?**

AGIC is Azure's **native** ingress controller that uses **Azure Application Gateway** (a cloud L7 load balancer) as the ingress controller instead of running a proxy pod inside the cluster.

```
AGIC Architecture:

  Internet                     Azure Managed
  ┌──────────┐               ┌──────────────────────────┐
  │  Client  │──────────────▶│  Azure Application       │
  │          │               │  Gateway (L7 LB)         │
  └──────────┘               │  - WAF (optional)        │
                             │  - SSL termination       │
                             │  - URL-based routing     │
                             │  - Auto-scaling          │
                             └──────────┬───────────────┘
                                        │
                             ┌──────────▼───────────────┐
                             │  AKS Cluster             │
                             │                          │
                             │  AGIC Controller Pod     │
                             │  (watches Ingress        │
                             │   resources → configures │
                             │   App Gateway via API)   │
                             │                          │
                             │  ┌─────┐  ┌─────┐       │
                             │  │Pod A│  │Pod B│        │
                             │  └─────┘  └─────┘       │
                             └──────────────────────────┘

  Key difference from Traefik/NGINX:
  ┌────────────────────────────────────────────────────────────┐
  │  NGINX/Traefik: proxy runs INSIDE the cluster (pod)       │
  │  AGIC: proxy runs OUTSIDE the cluster (Azure managed)     │
  │        → no proxy pods consuming cluster resources         │
  │        → Azure manages scaling, patching, HA               │
  └────────────────────────────────────────────────────────────┘
```

```yaml
# AGIC Ingress example:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/ssl-redirect: "true"
    appgw.ingress.kubernetes.io/backend-protocol: "http"
    appgw.ingress.kubernetes.io/waf-policy-for-path: "/subscriptions/.../myWafPolicy"
spec:
  tls:
  - hosts: [app.example.com]
    secretName: app-tls
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

---

**20e. Comparison — Which ingress controller to choose?**

```
Decision Matrix:

  Scenario                          Best Choice
  ──────────────────────────────────────────────────────────
  New project, any cloud            Traefik + Gateway API
  AKS (Azure) with WAF needed      AGIC (App Gateway)
  AKS (Azure) without WAF          Traefik on AKS
  Multi-cloud / hybrid             Traefik or Envoy/Contour
  Service mesh already (Istio)     Istio Gateway (built-in)
  API gateway features needed      Kong or Traefik Enterprise
  Existing nginx-ingress           Migrate to Traefik
```

| Feature | Traefik | AGIC (Azure) | Contour (Envoy) | Kong |
|---------|---------|-------------|-----------------|------|
| Gateway API | ✅ Native | ⚠️ Preview | ✅ Native | ✅ Native |
| Cloud-agnostic | ✅ | ❌ Azure only | ✅ | ✅ |
| WAF built-in | ❌ | ✅ (Azure WAF) | ❌ | ✅ (plugin) |
| Auto TLS/ACME | ✅ Built-in | ⚠️ Manual/KV | ✅ cert-manager | ⚠️ Plugin |
| Dashboard | ✅ Built-in | Azure Portal | ❌ | ✅ |
| Runs as pod | ✅ In-cluster | ❌ External LB | ✅ In-cluster | ✅ In-cluster |
| Cost | Free/OSS | App Gateway $$ | Free/OSS | Free/Enterprise |
| Best for | General purpose | Azure-native | Envoy users | API management |

---

**20f. How to migrate from NGINX Ingress to Traefik?**

```
Migration Steps:

  1. Install Traefik alongside nginx-ingress (both can coexist)
     helm install traefik traefik/traefik -n traefik --create-namespace

  2. Test with one service:
     - Change ingressClassName: nginx → ingressClassName: traefik
     - OR use Traefik IngressRoute CRDs for more features
     - Verify routing works

  3. Convert nginx-specific annotations:
     nginx.ingress.kubernetes.io/rewrite-target  →  Traefik middleware (StripPrefix)
     nginx.ingress.kubernetes.io/ssl-redirect    →  Traefik middleware (RedirectScheme)
     nginx.ingress.kubernetes.io/rate-limit      →  Traefik middleware (RateLimit)
     nginx.ingress.kubernetes.io/proxy-body-size →  Traefik middleware (Buffering)

  4. Migrate all Ingress resources one by one

  5. Remove nginx-ingress controller
     helm uninstall ingress-nginx -n ingress-nginx
```

---

**20g. What is Kubernetes Gateway API? (The future)**

Gateway API is the **official successor to the Ingress API** in Kubernetes. GA since K8s 1.27.

```
Gateway API Resource Model:

  ┌─────────────────────────────────────────────────────────┐
  │  GatewayClass        (infra team defines)               │
  │  "Which controller?" — traefik, envoy, istio, etc.     │
  └───────────────┬─────────────────────────────────────────┘
                  │
  ┌───────────────▼─────────────────────────────────────────┐
  │  Gateway              (platform team creates)           │
  │  "Where to listen?"  — ports, TLS config, addresses    │
  │  Like a load balancer instance                         │
  └───────────────┬─────────────────────────────────────────┘
                  │
  ┌───────────────▼─────────────────────────────────────────┐
  │  HTTPRoute / TCPRoute / GRPCRoute   (dev team creates) │
  │  "How to route?"    — host, path, headers, weights     │
  │  Attaches to Gateway                                   │
  └─────────────────────────────────────────────────────────┘

  RBAC Separation (why it's better):
  ┌──────────────┬──────────────────────────────────────────┐
  │ Infra team   │ Creates GatewayClass + Gateway           │
  │ (cluster     │ Controls which IPs, ports, TLS           │
  │  admin)      │                                          │
  ├──────────────┼──────────────────────────────────────────┤
  │ Dev team     │ Creates HTTPRoute only                   │
  │ (namespace   │ Can only route to services in their      │
  │  scoped)     │ namespace — no cluster-wide access       │
  └──────────────┴──────────────────────────────────────────┘
```

```yaml
# Gateway API example:
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: traefik
spec:
  controllerName: traefik.io/gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: infra
spec:
  gatewayClassName: traefik
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: wildcard-tls
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: myapp-route
  namespace: app-team
spec:
  parentRefs:
  - name: my-gateway
    namespace: infra
  hostnames: ["app.example.com"]
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-svc
      port: 80
      weight: 90             # Traffic splitting built-in!
    - name: api-svc-canary
      port: 80
      weight: 10
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
```

```
Why Gateway API > Ingress:

  ✅ Role-based: infra team vs dev team separation
  ✅ Multi-protocol: HTTP, TCP, UDP, gRPC, TLS passthrough
  ✅ Traffic splitting: native canary/blue-green by weight
  ✅ Header-based routing: match on headers, query params
  ✅ Cross-namespace references: controlled sharing
  ✅ Portable: works with Traefik, Envoy, Istio, Cilium, etc.
  ✅ Extensible: custom policies via policy attachment
  ❌ Ingress: single resource, HTTP only, annotations = messy
```

---

### NetworkPolicy

**21-27. NetworkPolicy?**

Firewall rules for pods. By default all pods can talk to all pods. NetworkPolicy restricts this.

```
Without NetworkPolicy:              With NetworkPolicy:
┌────────────────────┐              ┌────────────────────┐
│ All pods can       │              │ Frontend → API ✅  │
│ talk to all pods   │              │ Frontend → DB  ❌  │
│ (flat network)     │              │ API → DB       ✅  │
│                    │              │ External → API ❌  │
│ Security risk! ❌  │              │ Least privilege ✅  │
└────────────────────┘              └────────────────────┘
```

---

**28. Deny all ingress to a namespace:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}          # ALL pods in namespace
  policyTypes:
  - Ingress
  # No ingress rules = deny ALL incoming traffic
```

---

**29. CNI plugins for NetworkPolicy?**

| Plugin | NetworkPolicy | Technology | Notes |
|--------|---------------|-----------|-------|
| Calico | ✅ | iptables/eBPF | Most popular |
| Cilium | ✅ | eBPF | Advanced, fastest |
| Weave Net | ✅ | iptables | Simple |
| Flannel | ❌ | VXLAN | No NetworkPolicy! |

---

**30. Debug: user can't reach app, pod running, Service exists?**

```
Debugging Flow:
┌────────────────────────────────────────────────────────┐
│ 1. kubectl get endpoints <svc>                         │
│    Empty? → Label selector mismatch!                   │
│    Check: pod labels vs service selector               │
├────────────────────────────────────────────────────────┤
│ 2. kubectl describe svc <svc>                          │
│    Check Selector field                                │
│    kubectl get pods --show-labels                      │
├────────────────────────────────────────────────────────┤
│ 3. Test from inside cluster:                           │
│    kubectl exec <test-pod> -- curl <svc>:<port>        │
├────────────────────────────────────────────────────────┤
│ 4. Check readiness probe — failing?                    │
│    Pod not ready = removed from endpoints              │
├────────────────────────────────────────────────────────┤
│ 5. Check NetworkPolicy — blocking traffic?             │
├────────────────────────────────────────────────────────┤
│ 6. Check Ingress — wrong host/path?                    │
└────────────────────────────────────────────────────────┘
```

---

**31-34. (Service mesh, L4/L7, inter-service)**

**Service mesh**: Dedicated infrastructure for service-to-service communication. Adds: mTLS, retries, circuit breaking, traffic splitting, observability. Tools: Istio, Linkerd.

**L4 vs L7**: L4 = routes by IP:port (fast, Service LoadBalancer). L7 = routes by HTTP path/headers/cookies (smart, Ingress).

---

## CONFIG, STORAGE, SECURITY, HELM (65 Qs)

### ConfigMap & Secrets

**1-2. ConfigMap?**

Non-confidential configuration data as key-value pairs.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "postgres.prod.svc"
  LOG_LEVEL: "info"
  config.yaml: |
    server:
      port: 8080
      timeout: 30s
```

---

**3. ConfigMap as volume?**

```yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: myapp-config
# Each key becomes a file: /etc/config/DB_HOST, /etc/config/LOG_LEVEL
```

---

**4-6. Secrets?**

```bash
# From literal
kubectl create secret generic db-secret --from-literal=password=mysecret

# From file
kubectl create secret generic tls-secret --from-file=tls.crt --from-file=tls.key

# Declarative (base64 encoded — NOT encrypted!)
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: bXlzZWNyZXQ=    # echo -n "mysecret" | base64
```

```
⚠️ K8s Secrets are base64 encoded, NOT encrypted!
   Anyone with API access can decode them.
   For real security: use external secret managers
   (Vault, Azure Key Vault, AWS Secrets Manager)
```

---

**7. Secret types?**

| Type | Purpose |
|------|---------|
| `Opaque` | Default — arbitrary key-value data |
| `kubernetes.io/dockerconfigjson` | Registry credentials (imagePullSecrets) |
| `kubernetes.io/tls` | TLS certificate + key |
| `kubernetes.io/basic-auth` | Username + password |
| `kubernetes.io/service-account-token` | Auto-generated SA token |

---

**8-9. External secret managers?**

```
┌─── External Secrets Operator ────────────────────────────┐
│                                                           │
│  Azure Key Vault ──► ExternalSecret CR ──► K8s Secret    │
│  AWS Secrets Mgr                                          │
│  HashiCorp Vault     Syncs periodically (refreshInterval)│
│                                                           │
│  Tools:                                                   │
│  - External Secrets Operator (recommended)               │
│  - CSI Secrets Store Driver (mount as volume)            │
│  - Vault Agent Injector (sidecar)                        │
└───────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: azure-keyvault
    kind: ClusterSecretStore
  target:
    name: db-secret
  data:
  - secretKey: password
    remoteRef:
      key: production-db-password
```

---

**10. ConfigMap as env vars in Deployment:**

```yaml
spec:
  containers:
  - name: app
    envFrom:                           # ALL keys as env vars
    - configMapRef:
        name: app-config
    env:                               # Individual keys
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

---

### Storage

**11-12. PV, PVC, Access Modes?**

```
┌─── Storage Architecture ──────────────────────────────────────┐
│                                                                │
│  StorageClass ──► defines HOW to provision                    │
│       │                                                        │
│  PersistentVolumeClaim (PVC) ──► requests storage             │
│       │                                                        │
│  PersistentVolume (PV) ──► actual storage (auto-provisioned)  │
│       │                                                        │
│  Cloud Disk / NFS / local ──► physical storage                │
└────────────────────────────────────────────────────────────────┘
```

| Access Mode | Short | Meaning |
|-------------|-------|---------|
| ReadWriteOnce | RWO | One node mounts read-write (most common) |
| ReadOnlyMany | ROX | Many nodes mount read-only |
| ReadWriteMany | RWX | Many nodes mount read-write (NFS, Azure Files) |

---

**13-14. StorageClass & Reclaim Policy?**

| Reclaim Policy | Behavior |
|----------------|----------|
| `Delete` | PV + cloud disk deleted when PVC deleted (default for dynamic) |
| `Retain` | PV preserved — manual cleanup required |

---

**15-16. emptyDir & hostPath?**

```
emptyDir:                           hostPath:
┌──────────────────────┐           ┌──────────────────────┐
│ Created with pod     │           │ Mounts host dir      │
│ Deleted with pod     │           │ into container       │
│ Temp storage         │           │                      │
│                      │           │ Risks:               │
│ Use: scratch space,  │           │ - Tied to node       │
│ share between        │           │ - Security risk      │
│ containers in pod    │           │ - Not portable       │
│                      │           │                      │
│ Good ✅              │           │ Avoid ❌             │
│                      │           │ (DaemonSets only)    │
└──────────────────────┘           └──────────────────────┘
```

---

**17. PVC in Deployment:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: managed-premium
  resources:
    requests:
      storage: 10Gi
---
spec:
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data
```

---

### Probes

**18-24. Liveness, Readiness, Startup Probes?**

```
┌─── Startup Probe ────────────────────────────────────────────┐
│ "Has the app finished starting up?"                          │
│ Runs first. Until it passes, liveness/readiness don't start. │
│ Use for: slow-starting apps (Java, legacy apps)              │
│ Failure → kill and restart container                         │
└──────────────────────────────────────────────────────────────┘

┌─── Liveness Probe ───────────────────────────────────────────┐
│ "Is the app alive (not deadlocked/hung)?"                    │
│ Failure → kubelet RESTARTS the container                     │
│ Use for: detect deadlocks, hung processes                    │
│ If you're not sure → DON'T add liveness probe               │
└──────────────────────────────────────────────────────────────┘

┌─── Readiness Probe ──────────────────────────────────────────┐
│ "Is the app ready to receive traffic?"                       │
│ Failure → pod removed from Service endpoints (no traffic)    │
│ Use for: warming cache, loading data, dependency checks      │
│ ALWAYS add readiness probe                                   │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# All 3 probes for an HTTP app:
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10            # Allow up to 300s to start

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
  failureThreshold: 3
```

Probe mechanisms: `httpGet`, `tcpSocket`, `exec`.

---

### Resources & QoS

**25-28. Resource requests/limits, QoS classes?**

```
┌─── Resource Model ───────────────────────────────────────────┐
│                                                               │
│  requests: GUARANTEED minimum                                │
│    Scheduler uses this to place pods                         │
│    "My app needs at least this much"                         │
│                                                               │
│  limits: MAXIMUM cap                                         │
│    Container killed (OOMKill) if memory exceeds limit        │
│    Container throttled if CPU exceeds limit                  │
│                                                               │
│  resources:                                                   │
│    requests:                                                  │
│      cpu: 100m        # 10% of a core                        │
│      memory: 128Mi    # 128 MB guaranteed                    │
│    limits:                                                    │
│      cpu: 500m        # Max 50% of a core                    │
│      memory: 512Mi    # OOMKilled if exceeds                 │
└───────────────────────────────────────────────────────────────┘
```

| QoS Class | Condition | Eviction Priority |
|-----------|-----------|-------------------|
| **Guaranteed** | requests = limits (all containers) | Last (highest priority) |
| **Burstable** | requests < limits | Medium |
| **BestEffort** | No requests or limits | First (lowest — never use in prod!) |

---

### Security

**29-38. RBAC, ServiceAccount, Pod Security?**

```
┌─── RBAC Model ───────────────────────────────────────────────┐
│                                                               │
│  Subject ──► RoleBinding ──► Role ──► API Resources          │
│  (User/SA)                   (verbs)  (pods, secrets, etc.)  │
│                                                               │
│  Namespace-scoped:                                            │
│    Role + RoleBinding                                        │
│                                                               │
│  Cluster-scoped:                                              │
│    ClusterRole + ClusterRoleBinding                          │
│                                                               │
│  Verbs: get, list, watch, create, update, patch, delete      │
└───────────────────────────────────────────────────────────────┘
```

**Pod Security Admission** (replaces deprecated PodSecurityPolicy):

| Level | Restrictions |
|-------|-------------|
| Privileged | None — anything goes |
| Baseline | Prevents known privilege escalations |
| Restricted | Best practices — non-root, drop caps, read-only |

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

**Security context for a Pod:**

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
      seccompProfile:
        type: RuntimeDefault
```

---

### Helm

**39-48. Helm?**

Package manager for K8s. A **chart** = bundle of K8s manifests + configuration.

```
┌─── Helm Concepts ────────────────────────────────────────────┐
│                                                               │
│  Chart:     Package of K8s templates + values                │
│  Release:   Installed instance of a chart                    │
│  Repository: Where charts are stored                         │
│  values.yaml: Configuration defaults (override at install)   │
└───────────────────────────────────────────────────────────────┘
```

**Chart structure:**

```
mychart/
├── Chart.yaml          # Metadata (name, version)
├── values.yaml         # Default config values
├── charts/             # Sub-chart dependencies
├── templates/
│   ├── deployment.yaml # Go-templated manifests
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Reusable template functions
│   ├── NOTES.txt       # Post-install message
│   └── tests/
└── .helmignore
```

**Key commands:**

```bash
helm install myapp ./chart -f prod-values.yaml   # Install
helm upgrade myapp ./chart -f prod-values.yaml    # Upgrade
helm rollback myapp 2                              # Rollback to rev 2
helm uninstall myapp                               # Remove
helm list                                          # List releases
helm history myapp                                 # Revision history
helm template myapp ./chart                        # Render locally (debug)
helm lint ./chart                                  # Validate chart
```

**Templating:**

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicas }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
```

`helm template` vs `helm install`: template renders YAML locally (no cluster). install renders AND applies to cluster.

---

### Troubleshooting

**49-58. Debugging K8s issues:**

```
┌─── K8s Troubleshooting Toolkit ──────────────────────────────┐
│                                                               │
│  Pod Issues:                                                  │
│  kubectl get pods -o wide          # Status, node, restarts  │
│  kubectl describe pod <pod>        # Events (WHY it failed)  │
│  kubectl logs <pod> --previous     # Previous crash logs     │
│  kubectl exec -it <pod> -- sh     # Debug inside container  │
│                                                               │
│  Node Issues:                                                 │
│  kubectl describe node <node>      # Conditions, capacity    │
│  kubectl top nodes                  # Resource usage          │
│  systemctl status kubelet          # On the node itself      │
│                                                               │
│  Network Issues:                                              │
│  kubectl get endpoints <svc>       # Pod IPs in service      │
│  kubectl get svc,ingress           # Service config           │
│  kubectl exec -- curl <svc>:port  # Test from inside cluster│
│                                                               │
│  Events:                                                      │
│  kubectl get events --sort-by=.lastTimestamp                  │
│  kubectl get events --field-selector type=Warning             │
└───────────────────────────────────────────────────────────────┘
```

**Node NotReady?**

```
1. kubectl describe node <node>      → check Conditions
2. SSH to node:
   systemctl status kubelet          → is it running?
   systemctl status containerd       → container runtime OK?
   df -h                             → disk full?
   free -m                           → memory pressure?
   dmesg | grep -i oom               → OOM killer?
3. Can node reach API server?        → network issue?
```

**Drain node for maintenance:**

```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
# Evicts all pods (respects PDB), marks node unschedulable
# After maintenance:
kubectl uncordon <node>

# Just prevent new scheduling (don't evict):
kubectl cordon <node>
```

---

### Interview-Style Manifests

**59. Complete set: Deployment + Service + ConfigMap + Secret + Ingress:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  APP_ENV: production
  LOG_LEVEL: info
---
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myregistry/webapp:v1.0
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: webapp-config
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: DB_PASSWORD
        resources:
          requests: {cpu: 100m, memory: 128Mi}
          limits: {cpu: 500m, memory: 512Mi}
        readinessProbe:
          httpGet: {path: /ready, port: 8080}
        livenessProbe:
          httpGet: {path: /healthz, port: 8080}
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts: [webapp.example.com]
    secretName: webapp-tls
  rules:
  - host: webapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: {name: webapp-svc, port: {number: 80}}
```

---

**60. StatefulSet for PostgreSQL:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pg-secret
              key: password
        volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: pgdata
    spec:
      accessModes: [ReadWriteOnce]
      storageClassName: managed-premium
      resources:
        requests:
          storage: 20Gi
```

---

**61. CronJob for daily cleanup:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-cleanup
spec:
  schedule: "0 0 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: myapp/cleanup:latest
            command: ["/bin/sh", "-c", "python cleanup.py"]
          restartPolicy: OnFailure
```

---

**62. DaemonSet for log collector:**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentbit
spec:
  selector:
    matchLabels:
      app: fluentbit
  template:
    metadata:
      labels:
        app: fluentbit
    spec:
      containers:
      - name: fluentbit
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

---

**63. RBAC — dev read-only in "dev" namespace:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: dev
  name: dev-pod-reader
subjects:
- kind: User
  name: developer@company.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

**64. Organizing manifests for 15 microservices?**

```
Option A: Helm chart per service + shared library chart
Option B: Kustomize with base + overlays per environment

k8s/
├── base/                     # Shared templates
├── services/
│   ├── api-gateway/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   ├── user-service/
│   └── order-service/
└── overlays/
    ├── dev/
    ├── staging/
    └── prod/
```

---

**65. GitOps with ArgoCD?**

```
Developer ──► Git Push ──► PR Review ──► Merge
                                           │
                                    ┌──────▼──────┐
                                    │ ArgoCD      │
                                    │ watches Git │
                                    │ repo        │
                                    └──────┬──────┘
                                           │
                                    ┌──────▼──────┐
                                    │ Detects     │
                                    │ difference  │
                                    │ (drift)     │
                                    └──────┬──────┘
                                           │
                                    ┌──────▼──────┐
                                    │ Syncs       │
                                    │ cluster to  │
                                    │ match Git   │
                                    └─────────────┘

Rollback = git revert → ArgoCD auto-syncs
Git = single source of truth
```

---

# MANIFEST PRACTICE REFERENCE (All Resource Types)

> Quick-copy practice manifests for every K8s resource type. Use `kubectl apply -f <file>` to test.

---

### Pod (standalone — rare in production, good for debugging)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
  labels:
    app: debug
spec:
  containers:
  - name: busybox
    image: busybox:1.36
    command: ["sleep", "3600"]
    resources:
      requests: { cpu: 50m, memory: 64Mi }
      limits: { cpu: 100m, memory: 128Mi }
  restartPolicy: Never
```

---

### ReplicaSet (rarely used directly — Deployments manage these)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.27-alpine
        ports:
        - containerPort: 80
        resources:
          requests: { cpu: 50m, memory: 64Mi }
          limits: { cpu: 200m, memory: 128Mi }
```

```
When to use ReplicaSet directly vs Deployment:
  ReplicaSet: Almost NEVER directly. Only if you need custom update logic.
  Deployment: ALWAYS for stateless apps. It creates ReplicaSets for you.

  Deployment
       │ creates
       ▼
  ReplicaSet (rev 1)  ──► 3 Pods
       │ on update, creates new RS
       ▼
  ReplicaSet (rev 2)  ──► 3 Pods (new)
  ReplicaSet (rev 1)  ──► 0 Pods (scaled down, kept for rollback)
```

---

### Deployment (stateless apps — most common)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  labels:
    app: api-server
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: api-server
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1            # 1 extra pod during update
      maxUnavailable: 0       # 0 downtime
  template:
    metadata:
      labels:
        app: api-server
        version: v2.1.0
    spec:
      serviceAccountName: api-sa
      containers:
      - name: api
        image: myregistry/api-server:v2.1.0
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: api-config
              key: db_host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: api-secret
              key: password
        resources:
          requests: { cpu: 200m, memory: 256Mi }
          limits: { cpu: 1, memory: 512Mi }
        startupProbe:
          httpGet: { path: /healthz, port: 8080 }
          failureThreshold: 30
          periodSeconds: 5
        livenessProbe:
          httpGet: { path: /healthz, port: 8080 }
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet: { path: /ready, port: 8080 }
          periodSeconds: 5
        volumeMounts:
        - name: config-vol
          mountPath: /etc/config
          readOnly: true
      volumes:
      - name: config-vol
        configMap:
          name: api-config
```

---

### StatefulSet (databases, stateful apps — stable identity + storage)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis-headless       # Required — must match headless Service
  replicas: 3
  selector:
    matchLabels:
      app: redis
  updateStrategy:
    type: RollingUpdate
  podManagementPolicy: OrderedReady  # redis-0, redis-1, redis-2 in order
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
          name: redis
        resources:
          requests: { cpu: 100m, memory: 128Mi }
          limits: { cpu: 500m, memory: 256Mi }
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:             # Each pod gets its OWN PVC
  - metadata:
      name: redis-data
    spec:
      accessModes: [ReadWriteOnce]
      storageClassName: standard
      resources:
        requests:
          storage: 5Gi
---
# Headless Service (required for StatefulSet DNS)
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
spec:
  clusterIP: None                    # Headless!
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
```

```
StatefulSet gives:
  ✅ Stable pod names:    redis-0, redis-1, redis-2 (not random)
  ✅ Stable DNS:          redis-0.redis-headless.namespace.svc.cluster.local
  ✅ Ordered startup:     redis-0 → redis-1 → redis-2
  ✅ Ordered shutdown:    redis-2 → redis-1 → redis-0
  ✅ Stable storage:      Each pod keeps its PVC even after restart
```

---

### DaemonSet (one pod per node — logging, monitoring, networking)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true              # Access node network metrics
      tolerations:
      - operator: Exists             # Run on ALL nodes (even tainted ones)
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.8.0
        ports:
        - containerPort: 9100
          hostPort: 9100
        resources:
          requests: { cpu: 50m, memory: 64Mi }
          limits: { cpu: 200m, memory: 128Mi }
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath: { path: /proc }
      - name: sys
        hostPath: { path: /sys }
```

```
DaemonSet use cases:
  ✅ Log collectors:      Fluent Bit, Fluentd
  ✅ Monitoring agents:   Prometheus Node Exporter, Datadog Agent
  ✅ Network plugins:     Calico, Cilium, kube-proxy
  ✅ Storage drivers:     CSI node plugins
  ✅ Security agents:     Falco, Twistlock
```

---

### Job (run-to-completion — one-time tasks)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  backoffLimit: 3                   # Retry up to 3 times on failure
  activeDeadlineSeconds: 300        # Timeout after 5 minutes
  ttlSecondsAfterFinished: 3600    # Auto-delete after 1 hour
  template:
    spec:
      containers:
      - name: migrate
        image: myapp/migrate:v1.0
        command: ["python", "manage.py", "migrate"]
        env:
        - name: DB_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
      restartPolicy: Never          # Never or OnFailure
```

```yaml
# Parallel Job (process 10 items with 3 workers)
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-process
spec:
  completions: 10                   # Total items to process
  parallelism: 3                    # Run 3 pods at a time
  template:
    spec:
      containers:
      - name: worker
        image: myapp/worker:v1.0
      restartPolicy: OnFailure
```

---

### CronJob (scheduled recurring tasks)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"            # 2 AM daily
  concurrencyPolicy: Forbid         # Don't run if previous still running
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  startingDeadlineSeconds: 200      # Miss deadline = skip
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: myapp/backup:v1.0
            command: ["/bin/sh", "-c", "pg_dump $DB_URL > /backup/db.sql"]
            volumeMounts:
            - name: backup-vol
              mountPath: /backup
          volumes:
          - name: backup-vol
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

```
CronJob schedule cheat sheet:
  ┌───── minute (0-59)
  │ ┌───── hour (0-23)
  │ │ ┌───── day of month (1-31)
  │ │ │ ┌───── month (1-12)
  │ │ │ │ ┌───── day of week (0-6, Sun=0)
  │ │ │ │ │
  * * * * *

  "0 * * * *"     = every hour
  "*/15 * * * *"  = every 15 minutes
  "0 0 * * *"     = midnight daily
  "0 2 * * 1"     = 2 AM every Monday
  "0 0 1 * *"     = midnight on 1st of month
```

---

### Service — All Types

```yaml
# ClusterIP (default — internal only)
apiVersion: v1
kind: Service
metadata:
  name: api-svc
spec:
  type: ClusterIP                    # Default
  selector:
    app: api-server
  ports:
  - name: http
    port: 80                         # Service port (what clients use)
    targetPort: 8080                 # Container port
    protocol: TCP
---
# NodePort (expose on every node's IP)
apiVersion: v1
kind: Service
metadata:
  name: api-nodeport
spec:
  type: NodePort
  selector:
    app: api-server
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080                   # 30000-32767 range
---
# LoadBalancer (cloud provider provisions external LB)
apiVersion: v1
kind: Service
metadata:
  name: api-lb
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"  # Internal LB
spec:
  type: LoadBalancer
  selector:
    app: api-server
  ports:
  - port: 80
    targetPort: 8080
---
# Headless Service (for StatefulSet — direct pod DNS)
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None                    # No virtual IP — returns pod IPs directly
  selector:
    app: postgres
  ports:
  - port: 5432
---
# ExternalName (DNS alias to external service)
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.prod.example.com  # CNAME record — no selector
```

```
Service Types Summary:

  Type           Access              Use Case
  ─────────────────────────────────────────────────────────────
  ClusterIP      Internal only       Default, app-to-app
  NodePort       Node IP:30000+      Dev/test, direct access
  LoadBalancer   External LB IP      Production, cloud
  Headless       Pod DNS records     StatefulSet, direct pod access
  ExternalName   DNS CNAME           External service alias
```

---

### ConfigMap

```yaml
# From literal values
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  MAX_CONNECTIONS: "100"
  # Multi-line config file
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://localhost:8080;
      }
    }
```

```bash
# Create from command line
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-file=nginx.conf=./nginx.conf
```

---

### Secret

```yaml
# Opaque secret (generic key-value)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=              # echo -n "admin" | base64
  password: cGFzc3dvcmQ=          # echo -n "password" | base64
---
# TLS secret
apiVersion: v1
kind: Secret
metadata:
  name: tls-cert
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
---
# Docker registry credentials
apiVersion: v1
kind: Secret
metadata:
  name: registry-cred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

```bash
# Create from command line
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=s3cur3p@ss

# Create TLS secret
kubectl create secret tls tls-cert \
  --cert=./tls.crt --key=./tls.key

# Create registry secret
kubectl create secret docker-registry registry-cred \
  --docker-server=myregistry.azurecr.io \
  --docker-username=user \
  --docker-password=pass
```

---

### PersistentVolume + PersistentVolumeClaim

```yaml
# PV — Cluster-scoped storage resource (admin creates)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteMany                    # RWX — multiple pods can write
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    server: 10.0.0.5
    path: /exports/data
---
# PVC — Namespace-scoped claim (dev creates)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
  - ReadWriteOnce                    # RWO — single node can write
  storageClassName: managed-premium  # Azure managed disk
  resources:
    requests:
      storage: 20Gi
---
# Using PVC in a Pod
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:v1
    volumeMounts:
    - name: data
      mountPath: /app/data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data
```

```
PV/PVC Lifecycle:

  Admin creates PV          Dev creates PVC         PVC binds to PV
  (or StorageClass           (requests size +         (auto or manual)
   provisions dynamically)   access mode)

  ┌────────────────┐        ┌────────────────┐
  │ PV: 50Gi       │◄──────│ PVC: 20Gi      │
  │ RWO            │ bind  │ RWO            │
  │ Available      │       │ Bound          │
  └────────────────┘        └───────┬────────┘
                                    │ mount
                             ┌──────▼──────┐
                             │    Pod       │
                             │ /app/data   │
                             └─────────────┘

  Access Modes:
  RWO (ReadWriteOnce)  — one node reads/writes (most disks)
  ROX (ReadOnlyMany)   — many nodes read
  RWX (ReadWriteMany)  — many nodes read/write (NFS, Azure Files)
```

---

### StorageClass (dynamic provisioning)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: disk.csi.azure.com     # Azure Managed Disk
parameters:
  skuName: Premium_LRS              # Premium SSD
  kind: Managed
reclaimPolicy: Delete               # Delete disk when PVC deleted
volumeBindingMode: WaitForFirstConsumer  # Bind only when pod scheduled
allowVolumeExpansion: true          # Allow resize
```

---

### HorizontalPodAutoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  minReplicas: 2
  maxReplicas: 10
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300   # Wait 5 min before scaling down
      policies:
      - type: Percent
        value: 25                        # Remove max 25% of pods at once
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0      # Scale up immediately
      policies:
      - type: Pods
        value: 4                         # Add max 4 pods at once
        periodSeconds: 60
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70           # Target 70% CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80           # Target 80% memory
```

```bash
# Quick create from CLI
kubectl autoscale deployment api-server --min=2 --max=10 --cpu-percent=70
```

---

### PodDisruptionBudget (PDB)

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  selector:
    matchLabels:
      app: api-server
  minAvailable: 2                    # At least 2 pods must be running
  # OR use: maxUnavailable: 1       # At most 1 pod can be down
```

```
PDB protects against VOLUNTARY disruptions:
  ✅ kubectl drain (node maintenance)
  ✅ Cluster autoscaler scaling down
  ✅ kubectl delete pod (when managed by controller)

  PDB does NOT protect against:
  ❌ Node crash (involuntary)
  ❌ OOM kill
  ❌ Hardware failure
```

---

### ResourceQuota (namespace-level limits)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-alpha
spec:
  hard:
    requests.cpu: "10"               # Total CPU requests in namespace
    requests.memory: 20Gi            # Total memory requests
    limits.cpu: "20"                 # Total CPU limits
    limits.memory: 40Gi              # Total memory limits
    pods: "50"                       # Max 50 pods
    services: "20"                   # Max 20 services
    persistentvolumeclaims: "10"     # Max 10 PVCs
    configmaps: "20"
    secrets: "20"
```

---

### LimitRange (per-pod/container defaults and limits)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-alpha
spec:
  limits:
  - type: Container
    default:                         # Default LIMITS if not specified
      cpu: 500m
      memory: 256Mi
    defaultRequest:                  # Default REQUESTS if not specified
      cpu: 100m
      memory: 128Mi
    max:                             # Maximum any container can request
      cpu: 2
      memory: 2Gi
    min:                             # Minimum any container must request
      cpu: 50m
      memory: 64Mi
  - type: Pod
    max:
      cpu: 4
      memory: 4Gi
```

```
ResourceQuota vs LimitRange:

  ResourceQuota: Total limits for entire namespace
                 "Team gets max 10 CPUs total"
  LimitRange:    Per-pod/container defaults and limits
                 "Each container gets max 2 CPUs"
```

---

### ServiceAccount + RBAC (Role, ClusterRole, Bindings)

```yaml
# ServiceAccount (identity for pods)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-sa
  namespace: production
  annotations:
    # Azure Workload Identity
    azure.workload.identity/client-id: "<managed-identity-client-id>"
---
# Role (namespace-scoped permissions)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployer
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "pods/log", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]                     # Read-only for secrets
---
# RoleBinding (bind Role to ServiceAccount)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployer-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: api-sa
  namespace: production
roleRef:
  kind: Role
  name: deployer
  apiGroup: rbac.authorization.k8s.io
---
# ClusterRole (cluster-wide permissions)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-node-viewer
subjects:
- kind: ServiceAccount
  name: prometheus-sa
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io
```

```
RBAC Model:

  WHO (Subject)         CAN DO WHAT (Role)         WHERE (Binding)
  ──────────────────────────────────────────────────────────────────
  ServiceAccount        Role                       RoleBinding
  User                  (namespace-scoped)         (namespace-scoped)
  Group                 ClusterRole                ClusterRoleBinding
                        (cluster-wide)             (cluster-wide)

  Role + RoleBinding            = permissions in ONE namespace
  ClusterRole + ClusterRoleBinding = permissions across ALL namespaces
  ClusterRole + RoleBinding     = reusable role, applied to ONE namespace
```

---

### NetworkPolicy (firewall rules for pods)

```yaml
# Allow frontend → api, deny everything else to api pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend              # Allow from frontend pods
    - namespaceSelector:
        matchLabels:
          name: monitoring           # Allow from monitoring namespace
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres              # Allow to database
    ports:
    - protocol: TCP
      port: 5432
  - to:                              # Allow DNS resolution
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
# Deny all ingress (default deny)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

---

### Ingress (with Traefik — post NGINX retirement)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  ingressClassName: traefik
  tls:
  - hosts: [app.example.com, api.example.com]
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: { name: web-svc, port: { number: 80 } }
  - host: api.example.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service: { name: api-v1-svc, port: { number: 80 } }
      - path: /v2
        pathType: Prefix
        backend:
          service: { name: api-v2-svc, port: { number: 80 } }
```

---

### Gateway API (HTTPRoute — successor to Ingress)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: traefik
spec:
  controllerName: traefik.io/gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: production-gateway
  namespace: infra
spec:
  gatewayClassName: traefik
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: wildcard-tls
    allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            gateway-access: "true"   # Only labeled namespaces can attach
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-route
  namespace: app-team
spec:
  parentRefs:
  - name: production-gateway
    namespace: infra
  hostnames: ["api.example.com"]
  rules:
  - matches:
    - path: { type: PathPrefix, value: /api }
      headers:
      - name: X-Version
        value: "v2"
    backendRefs:
    - name: api-v2-svc
      port: 80
  - matches:
    - path: { type: PathPrefix, value: /api }
    backendRefs:
    - name: api-v1-svc
      port: 80
      weight: 90                     # 90% traffic
    - name: api-v2-svc
      port: 80
      weight: 10                     # 10% canary
```

---

### Affinity, Tolerations, Node Selector

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gpu-app
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      # Simple node selection
      nodeSelector:
        gpu: "true"

      # Advanced — prefer SSD nodes, require zone
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values: [us-east-1a, us-east-1b]
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: disktype
                operator: In
                values: [ssd]
        # Anti-affinity — spread replicas across nodes
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: gpu-app
              topologyKey: kubernetes.io/hostname

      # Tolerate tainted nodes
      tolerations:
      - key: "gpu"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"

      containers:
      - name: app
        image: myapp:v1
        resources:
          limits:
            nvidia.com/gpu: 1        # Request GPU
```

```
Scheduling Controls Cheat Sheet:

  nodeSelector:     Simple key=value match (must match)
  nodeAffinity:     Advanced rules (required or preferred, operators)
  podAffinity:      Schedule NEAR other pods (same node/zone)
  podAntiAffinity:  Schedule AWAY from other pods (spread out)
  taints:           Node says "keep pods away unless tolerated"
  tolerations:      Pod says "I can handle that taint"

  Taint + Toleration example:
  kubectl taint nodes gpu-node-1 gpu=true:NoSchedule
  → Only pods with matching toleration can schedule on gpu-node-1
```

---

### Pod Security (securityContext)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      securityContext:                # Pod-level
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: myapp:v1
        securityContext:              # Container-level
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
        volumeMounts:
        - name: tmp
          mountPath: /tmp             # Writable tmp since rootfs is read-only
      volumes:
      - name: tmp
        emptyDir: {}
```

---

### Kustomize (base + overlays)

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
- configmap.yaml
commonLabels:
  team: platform
---
# overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
namePrefix: dev-
namespace: dev
patches:
- target:
    kind: Deployment
    name: api-server
  patch: |
    - op: replace
      path: /spec/replicas
      value: 1
    - op: replace
      path: /spec/template/spec/containers/0/resources/requests/cpu
      value: 100m
configMapGenerator:
- name: app-config
  behavior: merge
  literals:
  - APP_ENV=development
  - LOG_LEVEL=debug
---
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
namePrefix: prod-
namespace: production
patches:
- target:
    kind: Deployment
    name: api-server
  patch: |
    - op: replace
      path: /spec/replicas
      value: 5
configMapGenerator:
- name: app-config
  behavior: merge
  literals:
  - APP_ENV=production
  - LOG_LEVEL=warn
```

```bash
# Apply dev overlay
kubectl apply -k overlays/dev/

# Apply prod overlay
kubectl apply -k overlays/prod/

# Preview rendered YAML
kubectl kustomize overlays/prod/
```

```
Kustomize Project Structure:

  k8s/
  ├── base/
  │   ├── kustomization.yaml
  │   ├── deployment.yaml
  │   ├── service.yaml
  │   └── configmap.yaml
  └── overlays/
      ├── dev/
      │   └── kustomization.yaml    (replicas: 1, debug logging)
      ├── staging/
      │   └── kustomization.yaml    (replicas: 2, info logging)
      └── prod/
          └── kustomization.yaml    (replicas: 5, warn logging, PDB)
```

---

### kubectl Cheat Sheet

```bash
# ─── GET ─────────────────────────────────────────────────────
kubectl get pods -A                          # All namespaces
kubectl get pods -o wide                     # IP + node info
kubectl get pods -l app=api                  # By label
kubectl get all -n production                # All resources in ns
kubectl get deploy,svc,ing,cm,secret -n prod # Specific types
kubectl get events --sort-by=.lastTimestamp  # Recent events

# ─── CREATE / APPLY ──────────────────────────────────────────
kubectl apply -f manifest.yaml               # Declarative (preferred)
kubectl create deployment nginx --image=nginx:1.27 --replicas=3
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP

# ─── INSPECT ──────────────────────────────────────────────────
kubectl describe pod <name>                  # Events, conditions
kubectl logs <pod> -c <container> --previous # Previous crash logs
kubectl logs -l app=api --all-containers     # All pods with label
kubectl top pods -n production               # CPU/memory usage
kubectl get pod <name> -o yaml               # Full spec

# ─── DEBUG ────────────────────────────────────────────────────
kubectl exec -it <pod> -- sh                 # Shell into pod
kubectl exec <pod> -- curl http://svc:80     # Test connectivity
kubectl port-forward svc/api-svc 8080:80     # Local access
kubectl debug <pod> --image=busybox -it      # Ephemeral debug container

# ─── EDIT / PATCH ────────────────────────────────────────────
kubectl set image deploy/api api=myapp:v2    # Update image
kubectl scale deploy/api --replicas=5        # Manual scale
kubectl rollout status deploy/api            # Watch rollout
kubectl rollout undo deploy/api              # Rollback
kubectl rollout history deploy/api           # Revision history
kubectl patch deploy api -p '{"spec":{"replicas":5}}'

# ─── DELETE ───────────────────────────────────────────────────
kubectl delete pod <name>                    # Delete pod
kubectl delete pod <name> --force --grace-period=0  # Force kill
kubectl delete -f manifest.yaml              # Delete from file

# ─── CONTEXT / NAMESPACE ─────────────────────────────────────
kubectl config get-contexts                  # List contexts
kubectl config use-context prod-cluster      # Switch cluster
kubectl config set-context --current --namespace=prod  # Default ns

# ─── DRY RUN (generate YAML) ─────────────────────────────────
kubectl create deploy nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl create svc clusterip api --tcp=80:8080 --dry-run=client -o yaml
kubectl run test --image=busybox --dry-run=client -o yaml -- sleep 3600
```

---
---

# PART 6: ADVANCED KUBERNETES — CRDs, Operators, NetworkPolicies, Admission Controllers

---

## Custom Resource Definitions (CRDs)

**90. What is a CRD? Why use it?**

A CRD (Custom Resource Definition) **extends the Kubernetes API** by defining your own resource types beyond built-in ones (Pods, Services, etc.).

```
Why CRDs Matter:

  Built-in K8s resources: Pod, Service, Deployment, ConfigMap, etc.
  But what if you need: Database, Certificate, GitRepository, Pipeline?

  CRDs let you create CUSTOM resources that kubectl treats like native ones:

  kubectl get databases         ← your custom resource!
  kubectl describe certificate  ← your custom resource!
  kubectl apply -f my-app.yaml  ← creates your custom object
```

```yaml
# Example: Define a CRD for a "Database" resource
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.io    # plural.group
spec:
  group: mycompany.io             # API group
  versions:
    - name: v1
      served: true                # Enable this version
      storage: true               # Store objects in this version
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required: ["engine", "version", "storage"]
              properties:
                engine:
                  type: string
                  enum: ["postgres", "mysql", "mongodb"]
                version:
                  type: string
                storage:
                  type: string
                  pattern: '^[0-9]+(Gi|Ti)$'
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 5
                  default: 1
            status:
              type: object
              properties:
                phase:
                  type: string
                endpoint:
                  type: string
      subresources:
        status: {}                # Enable /status subresource
      additionalPrinterColumns:   # Custom kubectl columns
        - name: Engine
          type: string
          jsonPath: .spec.engine
        - name: Version
          type: string
          jsonPath: .spec.version
        - name: Status
          type: string
          jsonPath: .status.phase
        - name: Age
          type: date
          jsonPath: .metadata.creationTimestamp
  scope: Namespaced               # or Cluster
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames: ["db"]            # kubectl get db
    categories: ["all"]           # shows in kubectl get all
```

```yaml
# Now create a custom Database resource
apiVersion: mycompany.io/v1
kind: Database
metadata:
  name: user-db
  namespace: production
spec:
  engine: postgres
  version: "15.4"
  storage: 100Gi
  replicas: 3
```

```bash
# Use it just like built-in resources
kubectl apply -f database.yaml
kubectl get databases            # or: kubectl get db
kubectl describe db user-db
kubectl delete db user-db
kubectl get db -o wide           # Shows custom printer columns

# Output:
# NAME      ENGINE     VERSION   STATUS   AGE
# user-db   postgres   15.4      Ready    5m
```

**Key interview answer:**
> "A CRD extends the Kubernetes API with custom resource types. You define the schema (like a database table definition), and then users can create instances using kubectl just like native resources. CRDs are the foundation for the Operator pattern — the CRD defines the 'what' and a controller implements the 'how'."

---

## Kubernetes Operators

**91. What is an Operator? How is it different from a Controller?**

```
Operator = CRD + Custom Controller + Domain Knowledge

  ┌─────────────────────────────────────────────────────────┐
  │                    OPERATOR PATTERN                      │
  │                                                         │
  │  ┌──────────┐     ┌──────────────────┐                 │
  │  │   CRD    │     │    Controller    │                 │
  │  │ (Schema) │     │ (Reconcile Loop) │                 │
  │  │          │     │                  │                 │
  │  │ defines  │     │ watches CRs and  │                 │
  │  │ desired  │────▶│ makes reality    │                 │
  │  │ state    │     │ match desired    │                 │
  │  │          │     │ state            │                 │
  │  └──────────┘     └────────┬─────────┘                 │
  │                            │                           │
  │                    ┌───────▼────────┐                   │
  │                    │ Domain Logic   │                   │
  │                    │ (backup, scale,│                   │
  │                    │  failover,     │                   │
  │                    │  upgrade DB)   │                   │
  │                    └────────────────┘                   │
  └─────────────────────────────────────────────────────────┘

  Controller: Generic reconcile loop (built-in: Deployment controller)
  Operator: Controller + DOMAIN EXPERTISE encoded in code
            (knows HOW to run a database, not just restart pods)
```

```
Reconciliation Loop (Heart of every Operator):

  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │    ┌───────────┐    Compare    ┌──────────────┐ │
  │    │  Desired  │──────────────▶│   Current    │ │
  │    │  State    │               │   State      │ │
  │    │  (CR spec)│               │  (cluster)   │ │
  │    └───────────┘               └──────────────┘ │
  │         │                            │          │
  │         │         ┌──────────┐       │          │
  │         └────────▶│RECONCILE │◀──────┘          │
  │                   │  (diff & │                  │
  │                   │   act)   │                  │
  │                   └────┬─────┘                  │
  │                        │                        │
  │              ┌─────────▼──────────┐             │
  │              │  Create/Update/    │             │
  │              │  Delete K8s        │             │
  │              │  resources         │             │
  │              └────────────────────┘             │
  │                        │                        │
  │                   Wait / Watch                  │
  │                   (event-driven)                │
  │                        │                        │
  │              Back to Reconcile ◀────────────────│
  └──────────────────────────────────────────────────┘
```

**Popular Operators in production:**

| Operator | What it manages | Why |
|----------|----------------|-----|
| **Prometheus Operator** | Prometheus + Alertmanager + ServiceMonitors | Auto-discovers monitoring targets |
| **cert-manager** | TLS certificates (Let's Encrypt) | Auto-renews certs before expiry |
| **Strimzi** | Apache Kafka clusters | Manages brokers, topics, users |
| **Zalando Postgres Operator** | PostgreSQL HA clusters | Automated failover + backups |
| **ArgoCD** | GitOps deployments | Syncs cluster state from Git |
| **Istio Operator** | Service mesh | Manages proxies, traffic rules |
| **Rook-Ceph** | Distributed storage | Manages Ceph storage on K8s |
| **Crossplane** | Cloud resources (RDS, S3) | Provisions cloud infra from K8s |

**92. How to build an Operator?**

```bash
# Using Operator SDK (most common approach)
# 1. Install
brew install operator-sdk    # or download binary

# 2. Scaffold project
operator-sdk init --domain=mycompany.io --repo=github.com/me/db-operator
operator-sdk create api --group=db --version=v1 --kind=Database --resource --controller

# 3. Project structure
.
├── api/v1/
│   └── database_types.go     # CRD type definitions (your schema)
├── controllers/
│   └── database_controller.go # Reconcile logic
├── config/
│   ├── crd/                   # Generated CRD YAML
│   ├── rbac/                  # RBAC for the operator
│   └── manager/               # Deployment for operator
├── main.go
└── Makefile
```

```go
// api/v1/database_types.go — Define the CRD schema
type DatabaseSpec struct {
    Engine   string `json:"engine"`            // postgres, mysql
    Version  string `json:"version"`           // 15.4
    Storage  string `json:"storage"`           // 100Gi
    Replicas int32  `json:"replicas,omitempty"` // default 1
}

type DatabaseStatus struct {
    Phase    string `json:"phase,omitempty"`    // Pending, Running, Failed
    Endpoint string `json:"endpoint,omitempty"` // connection string
    Ready    bool   `json:"ready,omitempty"`
}

type Database struct {
    metav1.TypeMeta   `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`
    Spec   DatabaseSpec   `json:"spec,omitempty"`
    Status DatabaseStatus `json:"status,omitempty"`
}
```

```go
// controllers/database_controller.go — Reconcile logic
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)

    // 1. Fetch the Database CR
    var db dbv1.Database
    if err := r.Get(ctx, req.NamespacedName, &db); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 2. Create StatefulSet if it doesn't exist
    found := &appsv1.StatefulSet{}
    err := r.Get(ctx, types.NamespacedName{Name: db.Name, Namespace: db.Namespace}, found)
    if err != nil && errors.IsNotFound(err) {
        sts := r.statefulSetForDB(&db)    // Build desired StatefulSet
        if err := r.Create(ctx, sts); err != nil {
            return ctrl.Result{}, err
        }
        log.Info("Created StatefulSet", "name", db.Name)
    }

    // 3. Create Service for the database
    svc := r.serviceForDB(&db)
    if err := r.Create(ctx, svc); err != nil && !errors.IsAlreadyExists(err) {
        return ctrl.Result{}, err
    }

    // 4. Update status
    db.Status.Phase = "Running"
    db.Status.Endpoint = fmt.Sprintf("%s.%s.svc:5432", db.Name, db.Namespace)
    db.Status.Ready = true
    if err := r.Status().Update(ctx, &db); err != nil {
        return ctrl.Result{}, err
    }

    return ctrl.Result{RequeueAfter: 30 * time.Second}, nil // Periodic reconcile
}
```

**Key interview answer:**
> "An Operator is a CRD plus a custom controller that encodes **domain-specific operational knowledge**. A regular controller just watches and reconciles (like the Deployment controller), but an Operator knows *how* to operate complex software — backup a database, failover replicas, upgrade versions safely. I'd build one with the Operator SDK which scaffolds the Go project, CRD types, and reconciliation loop. Real examples: cert-manager for TLS, Prometheus Operator for monitoring, Strimzi for Kafka."

---

## Network Policies

**93. What are NetworkPolicies? How do they work?**

```
NetworkPolicy — Firewall rules for Pod-to-Pod traffic:

  WITHOUT NetworkPolicy (default):
  ┌─────────────────────────────────────────┐
  │  Every pod can talk to every other pod  │
  │  Pod A ←→ Pod B ←→ Pod C ←→ Pod D      │
  │  (fully open — NOT secure!)             │
  └─────────────────────────────────────────┘

  WITH NetworkPolicy:
  ┌─────────────────────────────────────────┐
  │  Only allowed traffic gets through      │
  │  Pod A ──→ Pod B    (allowed ✅)        │
  │  Pod C ──✗ Pod B    (blocked ❌)        │
  │  Pod D ──→ Pod B:80 (allowed ✅)        │
  │  Pod D ──✗ Pod B:22 (blocked ❌)        │
  └─────────────────────────────────────────┘

  IMPORTANT: You need a CNI that supports NetworkPolicies!
  ✅ Calico, Cilium, Weave Net
  ❌ Flannel (no NetworkPolicy support by default)
```

```yaml
# Example 1: Allow only frontend pods to reach backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-frontend
  namespace: production
spec:
  podSelector:                    # WHO is protected
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:                        # WHO can reach backend
    - from:
        - podSelector:
            matchLabels:
              app: frontend
        - namespaceSelector:       # From monitoring namespace too
            matchLabels:
              name: monitoring
      ports:
        - protocol: TCP
          port: 8080
  egress:                          # WHERE can backend connect
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
    - to:                          # Allow DNS
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
```

```yaml
# Example 2: Default deny ALL ingress in a namespace (Zero Trust)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}                  # Applies to ALL pods
  policyTypes:
    - Ingress                      # Block ALL inbound
  # No ingress rules = deny all

# Example 3: Default deny ALL egress (lock everything down)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:                          # Only allow DNS
    - to: []
      ports:
        - protocol: UDP
          port: 53
```

```
Best Practice — Zero Trust Networking:

  1. Apply default-deny to every namespace
  2. Explicitly allow only needed traffic
  3. Restrict by pod label + namespace + port
  4. Always allow DNS (UDP 53) in egress rules
  5. Test with: kubectl run test --image=busybox -- wget -qO- backend:8080
```

**Key interview answer:**
> "NetworkPolicies are Kubernetes-native firewall rules that control pod-to-pod traffic. By default, all pods can communicate freely — NetworkPolicies restrict this. I apply a **zero-trust model**: default-deny all ingress/egress per namespace, then explicitly allow only needed paths (e.g., frontend→backend:8080, backend→database:5432). They require a CNI that supports them — Calico or Cilium, not Flannel. Always remember to allow DNS (UDP 53) in egress rules or nothing resolves."

---

## Admission Controllers

**94. What are Admission Controllers? Types?**

```
Request Flow Through Kubernetes API:

  kubectl apply ──▶ API Server ──▶ Authentication ──▶ Authorization (RBAC)
                                                           │
                                                           ▼
                                              ┌─── Admission Controllers ───┐
                                              │                             │
                                              │  1. Mutating Webhooks       │
                                              │     (modify the request)    │
                                              │         │                   │
                                              │         ▼                   │
                                              │  2. Schema Validation       │
                                              │         │                   │
                                              │         ▼                   │
                                              │  3. Validating Webhooks     │
                                              │     (accept/reject)         │
                                              │                             │
                                              └──────────────┬──────────────┘
                                                             │
                                                             ▼
                                                        etcd (stored)
```

**Built-in Admission Controllers:**

| Controller | Purpose |
|-----------|---------|
| `NamespaceLifecycle` | Prevents creating objects in terminating namespaces |
| `LimitRanger` | Applies default resource limits from LimitRange |
| `ResourceQuota` | Enforces namespace resource quotas |
| `PodSecurity` | Enforces Pod Security Standards (replaces PSP) |
| `MutatingAdmissionWebhook` | Calls external webhooks to mutate requests |
| `ValidatingAdmissionWebhook` | Calls external webhooks to validate requests |

```yaml
# Example: ValidatingWebhookConfiguration — Block containers running as root
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: no-root-containers
webhooks:
  - name: validate.security.mycompany.io
    clientConfig:
      service:
        name: security-webhook
        namespace: webhook-system
        path: /validate
      caBundle: <base64-ca-cert>
    rules:
      - operations: ["CREATE", "UPDATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    admissionReviewVersions: ["v1"]
    sideEffects: None
    failurePolicy: Fail           # Reject if webhook unavailable

# Example: MutatingWebhookConfiguration — Auto-inject sidecar
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: sidecar-injector
webhooks:
  - name: inject.sidecar.mycompany.io
    clientConfig:
      service:
        name: sidecar-injector
        namespace: webhook-system
        path: /inject
      caBundle: <base64-ca-cert>
    rules:
      - operations: ["CREATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    namespaceSelector:
      matchLabels:
        sidecar-injection: enabled
    admissionReviewVersions: ["v1"]
    sideEffects: None
```

**Real-world uses:**
- **Istio** uses mutating webhook to inject Envoy sidecar into every pod
- **OPA/Gatekeeper** uses validating webhook to enforce policies (no privileged pods, mandatory labels)
- **cert-manager** uses mutating webhook to inject CA bundles

**Key interview answer:**
> "Admission controllers intercept API requests **after authentication and authorization but before persistence to etcd**. There are two types: **mutating** (modify the request — e.g., Istio injecting sidecar containers) and **validating** (accept or reject — e.g., blocking pods without resource limits). Mutating runs first so validating can check the final state. You can write custom webhooks or use tools like OPA Gatekeeper for policy enforcement."

---

## Kubernetes Networking Deep Dive

**95. Kubernetes networking model — how does pod-to-pod communication work?**

```
Kubernetes Networking Requirements (the "4 rules"):

  1. Every Pod gets its own IP address
  2. Pods on same node can communicate without NAT
  3. Pods on different nodes can communicate without NAT
  4. Agents (kubelet, kube-proxy) can communicate with all pods

  ┌──────────────────────── Node 1 ─────────────────────────┐
  │                                                         │
  │   Pod A (10.244.1.2)    Pod B (10.244.1.3)              │
  │   ┌──────────────┐      ┌──────────────┐               │
  │   │  eth0         │      │  eth0         │               │
  │   └──────┬───────┘      └──────┬───────┘               │
  │          │                      │                       │
  │     ┌────▼──────────────────────▼────┐                  │
  │     │       veth pairs → cbr0        │ (bridge)         │
  │     └────────────────┬───────────────┘                  │
  │                      │                                  │
  │                 ┌────▼────┐                             │
  │                 │  eth0   │ (Node IP: 192.168.1.10)     │
  │                 └────┬────┘                             │
  └──────────────────────┼──────────────────────────────────┘
                         │
              ┌──────────▼──────────┐
              │    Network Fabric    │
              │   (physical/overlay) │
              └──────────┬──────────┘
                         │
  ┌──────────────────────┼──────────────────────────────────┐
  │                 ┌────▼────┐                             │
  │                 │  eth0   │ (Node IP: 192.168.1.11)     │
  │                 └────┬────┘                             │
  │     ┌────────────────▼───────────────┐                  │
  │     │       veth pairs → cbr0        │                  │
  │     └────┬──────────────────────┬────┘                  │
  │          │                      │                       │
  │   ┌──────▼───────┐      ┌──────▼───────┐               │
  │   │  eth0         │      │  eth0         │               │
  │   └──────────────┘      └──────────────┘               │
  │   Pod C (10.244.2.2)    Pod D (10.244.2.3)              │
  │                                                         │
  └──────────────────────── Node 2 ─────────────────────────┘
```

**96. CNI Plugins compared:**

```
CNI Plugin Comparison:

  ┌──────────────┬──────────┬──────────┬──────────┬──────────┐
  │   Feature    │  Calico  │  Cilium  │ Flannel  │ Weave    │
  ├──────────────┼──────────┼──────────┼──────────┼──────────┤
  │ NetworkPolicy│  ✅ Full │  ✅ Full │  ❌ No   │  ✅ Full │
  │ Encryption   │  WireGrd │  WireGrd │  ❌ No   │  ✅ Yes  │
  │ Performance  │  High    │  Highest │  Medium  │  Medium  │
  │ eBPF         │  Partial │  ✅ Core │  ❌ No   │  ❌ No   │
  │ L7 Policy    │  ❌      │  ✅ Yes  │  ❌      │  ❌      │
  │ Observability│  Basic   │  Hubble  │  Basic   │  Basic   │
  │ Complexity   │  Medium  │  High    │  Low     │  Low     │
  │ Best For     │  General │  Advanced│  Simple  │  Small   │
  └──────────────┴──────────┴──────────┴──────────┴──────────┘
```

**97. Service types and how kube-proxy works?**

```
Service Types — Traffic Flow:

  ClusterIP (default):
  ┌─────────────────────────────────────────┐
  │  Internal only. Virtual IP.             │
  │  Pod → ClusterIP:port → kube-proxy      │
  │       → iptables/IPVS rules             │
  │       → load-balance to backend pods    │
  └─────────────────────────────────────────┘

  NodePort:
  ┌─────────────────────────────────────────┐
  │  External access via <NodeIP>:<NodePort>│
  │  Range: 30000-32767                     │
  │  Client → Node:30080 → kube-proxy      │
  │         → backend pod                   │
  └─────────────────────────────────────────┘

  LoadBalancer:
  ┌─────────────────────────────────────────┐
  │  Cloud LB → NodePort → Pod             │
  │  Gets external IP from cloud provider  │
  │  Client → Cloud LB:80 → Node:30080    │
  │         → backend pod                   │
  └─────────────────────────────────────────┘

  ExternalName:
  ┌─────────────────────────────────────────┐
  │  CNAME alias to external DNS            │
  │  No proxy, just DNS resolution          │
  │  my-svc.ns.svc → external.example.com  │
  └─────────────────────────────────────────┘

  kube-proxy modes:
  ├── iptables (default): Creates iptables rules for each Service
  │   Random pod selection, no real load balancing
  ├── IPVS: True load balancing (round-robin, least-conn, etc.)
  │   Better performance for large clusters (>1000 services)
  └── eBPF (Cilium): Replaces kube-proxy entirely, highest performance
```

**98. Ingress vs Gateway API?**

```yaml
# Ingress — L7 HTTP routing (older, simpler)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  ingressClassName: nginx
  tls:
    - hosts: [app.example.com]
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

```yaml
# Gateway API — next-gen (more expressive, multi-tenant)
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: main-gateway
spec:
  gatewayClassName: istio
  listeners:
    - name: https
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - name: app-cert
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-route
spec:
  parentRefs:
    - name: main-gateway
  hostnames: ["app.example.com"]
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      backendRefs:
        - name: api-service
          port: 80
          weight: 90            # Traffic splitting!
        - name: api-canary
          port: 80
          weight: 10
```

```
Ingress vs Gateway API:

  ┌───────────────────┬──────────────────┬──────────────────────┐
  │     Feature       │    Ingress       │    Gateway API       │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Status            │ Stable, mature   │ GA (v1.0+), future   │
  │ Protocol          │ HTTP/HTTPS only  │ HTTP, gRPC, TCP, UDP │
  │ Traffic Split     │ Via annotations  │ Native (weights)     │
  │ Multi-tenant      │ No               │ Yes (role-based)     │
  │ Header matching   │ Via annotations  │ Native               │
  │ Portability       │ Annotations vary │ Standardized API     │
  │ Complexity        │ Lower            │ Higher               │
  └───────────────────┴──────────────────┴──────────────────────┘
```

---

## Pod Security Standards (PSS) & Pod Security Admission (PSA)

**99. What replaced PodSecurityPolicies?**

```
Pod Security Standards — 3 levels:

  ┌─────────────────────────────────────────────────────────┐
  │  PRIVILEGED    │ No restrictions. For system components. │
  │                │ kube-system pods, CNI, storage drivers  │
  ├────────────────┼────────────────────────────────────────┤
  │  BASELINE      │ Minimal restrictions. Prevents known   │
  │                │ privilege escalations. Good default.    │
  │                │ No hostNetwork, no privileged, no       │
  │                │ hostPID, no hostIPC                     │
  ├────────────────┼────────────────────────────────────────┤
  │  RESTRICTED    │ Hardened. Best practices.              │
  │                │ Must run as non-root, drop ALL caps,   │
  │                │ read-only rootfs, no privilege escalate │
  │                │ Use for all application workloads.     │
  └────────────────┴────────────────────────────────────────┘
```

```bash
# Apply Pod Security to a namespace
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted

# Modes:
# enforce = reject pods that violate
# warn    = allow but show warning
# audit   = allow but log to audit log
```

```yaml
# Pod that passes RESTRICTED standard
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: myapp:1.0
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        runAsUser: 1000
        capabilities:
          drop: ["ALL"]
      resources:
        limits:
          memory: 256Mi
          cpu: 500m
        requests:
          memory: 128Mi
          cpu: 100m
```

---

## RBAC Deep Dive

**100. RBAC — Role, ClusterRole, RoleBinding, ClusterRoleBinding?**

```yaml
# Role — namespace-scoped permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
  - apiGroups: [""]              # core API group
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]

---
# ClusterRole — cluster-wide (or reusable across namespaces)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["namespaces"]
    verbs: ["get", "list"]
  - apiGroups: ["apiextensions.k8s.io"]  # CRD access
    resources: ["customresourcedefinitions"]
    verbs: ["get", "list"]

---
# RoleBinding — bind Role to user/group/serviceaccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
  - kind: User
    name: vaibhav
    apiGroup: rbac.authorization.k8s.io
  - kind: ServiceAccount
    name: ci-pipeline
    namespace: ci
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRoleBinding — cluster-wide binding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: global-node-viewer
subjects:
  - kind: Group
    name: platform-team
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Check your permissions
kubectl auth can-i create pods --namespace production       # yes/no
kubectl auth can-i '*' '*'                                  # am I admin?
kubectl auth can-i list pods --as=vaibhav                   # impersonate
kubectl auth can-i create deployments --as=system:serviceaccount:ci:ci-pipeline
```
