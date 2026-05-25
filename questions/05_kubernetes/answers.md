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
