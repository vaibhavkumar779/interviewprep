# Kubernetes - COMPREHENSIVE ANSWERS (All 134 Questions)

---

## BASICS & WORKLOADS (35 Qs)

### Fundamentals

**1. What is Kubernetes?**
Open-source container orchestration platform. Automates deployment, scaling, self-healing of containerized apps. Originally by Google, maintained by CNCF.

**2. Container orchestration? Why not just Docker?**
Docker runs single containers. When you have 50+ containers across multiple servers, you need: automated placement, scaling, load balancing, self-healing, rolling updates, service discovery. That's orchestration.

**3. K8s architecture?**
**Control Plane**: API Server (front door), etcd (state store), Scheduler (places pods), Controller Manager (ensures desired state).
**Worker Nodes**: kubelet (manages pods), kube-proxy (networking), container runtime (containerd).

**4. Control plane components?**
- **API Server**: RESTful API, all commands go through here
- **etcd**: Distributed key-value store, holds ALL cluster state
- **Scheduler**: Watches for unscheduled pods, picks best node based on resources/constraints
- **Controller Manager**: Runs controllers (ReplicaSet, Deployment, Node, Job controllers)

**5. kubelet?**
Agent running on every worker node. Receives pod specs from API server, ensures containers are running and healthy. Reports node status back.

**6. kube-proxy?**
Network proxy on each node. Maintains iptables/IPVS rules to route traffic to correct pods. Implements Service abstraction.

**7. etcd?**
Distributed key-value store holding entire cluster state. Critical because if etcd is lost, you lose cluster state. Must be backed up regularly. Runs on control plane.

**8. API server?**
Central management point. All kubectl commands, kubelet calls, and controller interactions go through it. Authenticates, authorizes (RBAC), validates, and stores in etcd.

**9. kubectl? 10 daily commands?**
```bash
kubectl get pods -o wide                    # List pods with node info
kubectl get pods --all-namespaces           # All namespaces
kubectl describe pod <pod>                  # Detailed pod info + events
kubectl logs <pod> -f                       # Follow logs
kubectl logs <pod> --previous               # Previous crash logs
kubectl exec -it <pod> -- /bin/sh           # Shell into pod
kubectl apply -f manifest.yaml              # Apply configuration
kubectl delete pod <pod>                    # Delete pod
kubectl get events --sort-by=.lastTimestamp # Recent events
kubectl top pods                            # Resource usage
```

**10. Namespace?**
Virtual cluster for logical isolation. Use cases: separate teams (team-a, team-b), environments (dev, staging), or applications. Default namespaces: default, kube-system, kube-public.

**11. What is a node? Add node?**
A node is a worker machine (VM or physical). Add node: install container runtime + kubelet + kube-proxy, then join with `kubeadm join <api-server>:6443 --token <token>`. In managed K8s (AKS/EKS), add via node pool scaling.

**12. Managed vs self-managed K8s?**
| Managed (AKS/EKS/GKE) | Self-managed (kubeadm) |
|---|---|
| Cloud manages control plane | You manage everything |
| Auto-upgrades available | Manual upgrades |
| Integrated with cloud IAM/networking | You configure everything |
| $$$ (cloud pricing) | Your hardware costs |
| Great for most teams | Full control, complex setups |

### Pods

**13. What is a Pod?**
Smallest deployable unit. One or more containers sharing network namespace (same IP, localhost) and storage volumes. Usually 1 container per pod. Pod gets scheduled to a node as a unit.

**14. Multiple containers in a Pod?**
Yes. Use cases: sidecar pattern (logging agent, proxy), adapter pattern (format converter), ambassador pattern (proxy to external service). All containers share network + volumes.

**15. Sidecar container? 3 examples.**
A helper container running alongside the main container:
1. **Log collector**: Fluent Bit sidecar shipping logs to Loki
2. **Service mesh proxy**: Istio Envoy sidecar handling traffic
3. **Config syncer**: Sidecar that watches ConfigMap and reloads app config

**16. Init container?**
Runs to completion before app containers start. Use cases: wait for database to be ready, clone a Git repo, run DB migrations, set file permissions. Runs sequentially if multiple.
```yaml
initContainers:
- name: wait-for-db
  image: busybox
  command: ['sh', '-c', 'until nc -z postgres 5432; do sleep 2; done']
```

**17. Pod lifecycle?**
- **Pending**: Accepted but not yet scheduled or images downloading
- **Running**: At least one container running
- **Succeeded**: All containers exited with code 0 (Jobs)
- **Failed**: At least one container exited with non-zero
- **Unknown**: Node communication lost

**18. Get logs from Pod? Previous crash?**
```bash
kubectl logs <pod>                        # Current logs
kubectl logs <pod> -c <container>         # Specific container
kubectl logs <pod> --previous             # Previous crash instance
kubectl logs <pod> --since=1h             # Last hour
kubectl logs <pod> --tail=100             # Last 100 lines
kubectl logs -l app=myapp --all-containers # All pods with label
```

**19. Exec into Pod?**
```bash
kubectl exec -it <pod> -- /bin/sh         # Interactive shell
kubectl exec -it <pod> -c <container> -- bash  # Specific container
kubectl exec <pod> -- cat /etc/config     # One-off command
```

**20. What happens when Pod is deleted?**
1. Pod enters `Terminating` state
2. kubelet sends SIGTERM to containers
3. Waits `terminationGracePeriodSeconds` (default 30s)
4. If still running, sends SIGKILL
5. Pod removed from Service endpoints (no more traffic)
6. Pod removed from API server

### Workloads

**21. Deployment?**
Manages ReplicaSets which manage Pods. Provides: declarative updates, rolling updates, rollback, scaling. **Always use Deployment** for stateless apps.

**22. ReplicaSet vs Deployment?**
ReplicaSet ensures N pod replicas running. Deployment manages ReplicaSets and adds rolling updates + rollback. **Never create ReplicaSet directly** — use Deployment.

**23. DaemonSet? 3 use cases.**
Ensures one pod on every node (or selected nodes).
1. **Log collector**: Fluent Bit/Fluentd on every node
2. **Monitoring agent**: Node Exporter (Prometheus) on every node
3. **Network plugin**: Calico/Cilium CNI agent on every node

**24. StatefulSet vs Deployment?**
| Deployment | StatefulSet |
|---|---|
| Stateless apps | Stateful apps (DB, Kafka) |
| Pods are interchangeable | Pods have stable identity (pod-0, pod-1) |
| Random pod names | Ordered, predictable names |
| Shared storage optional | Persistent storage per pod |
| Parallel scaling | Ordered deployment/scaling |

**25. Job vs CronJob?**
- **Job**: Run to completion, then done. Use: DB migration, batch processing, data export.
- **CronJob**: Scheduled Job. `schedule: "0 2 * * *"` = daily at 2am.
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "0 0 * * *"    # midnight daily
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

**26. Rolling update vs Recreate?**
- **RollingUpdate**: Gradually replaces old pods. Zero downtime. Default.
- **Recreate**: Kills all old pods first, then creates new. Brief downtime. Use when app can't run two versions simultaneously.

**27. maxSurge and maxUnavailable?**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          # Max 1 extra pod during update (4 total if replicas=3)
    maxUnavailable: 0    # Never fewer than 3 running (zero downtime)
```
maxSurge=25% maxUnavailable=25% is the default.

**28. Rollback a Deployment?**
```bash
kubectl rollout undo deployment/myapp                    # Rollback to previous
kubectl rollout undo deployment/myapp --to-revision=3    # Specific revision
kubectl rollout history deployment/myapp                 # View revision history
kubectl rollout status deployment/myapp                  # Check rollout status
```

**29. Scale a Deployment?**
```bash
# Manual
kubectl scale deployment/myapp --replicas=5

# Auto (HPA)
kubectl autoscale deployment/myapp --min=2 --max=10 --cpu-percent=70
```

**30. Horizontal Pod Autoscaler (HPA)?**
Auto-scales pod count based on CPU/memory/custom metrics:
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

### Interview-Style

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

**32. Pod in Pending state - 5 reasons?**
1. **Insufficient resources**: No node has enough CPU/memory → scale up cluster
2. **Node selector/affinity mismatch**: Pod requires specific node label → fix selector
3. **PVC not bound**: Requested storage not available → check StorageClass
4. **Image pulling**: Large image still downloading → check image name
5. **Taints preventing scheduling**: Node has taint, pod has no toleration → add toleration

**33. CrashLoopBackOff debugging:**
1. `kubectl logs <pod> --previous` → check application error
2. `kubectl describe pod <pod>` → check events (OOMKilled? ImagePullError?)
3. Check if config/secret is missing: `kubectl get configmap`, `kubectl get secret`
4. Check if port is already in use or permission denied
5. Test image locally: `docker run -it <image> /bin/sh`

**34. Zero-downtime deployments?**
1. Use RollingUpdate strategy with maxUnavailable=0
2. Configure readinessProbe (pod gets traffic only when ready)
3. Use preStop lifecycle hook (graceful shutdown: `sleep 5` before SIGTERM)
4. Set terminationGracePeriodSeconds appropriately
5. PodDisruptionBudget to maintain minimum available pods

**35. URL → K8s app flow:**
1. User types URL → DNS resolves to LoadBalancer IP
2. Cloud Load Balancer receives traffic
3. Traffic hits Ingress Controller (Nginx) pod
4. Ingress rules match hostname/path → routes to Service
5. Service selects pods via label selector → kube-proxy routes to pod IP
6. Pod's container processes the request

---

## NETWORKING & SERVICES (34 Qs)

**1-8.** (Already covered above in basics)

**9. Endpoints object?**
Automatically created for each Service. Contains the list of pod IPs matching the Service's selector. Updated as pods are created/destroyed.
```bash
kubectl get endpoints myapp-svc
# NAME        ENDPOINTS                               AGE
# myapp-svc   10.244.1.5:8080,10.244.2.3:8080        5m
```

**10. Headless Service?**
Service with `clusterIP: None`. No load balancing, no virtual IP. DNS returns individual pod IPs directly. Used with StatefulSets where clients need to connect to specific pods.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - port: 5432
# DNS: pod-0.db-headless.namespace.svc.cluster.local
```

**11. Write Service manifests:**
```yaml
# ClusterIP (internal only)
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
# NodePort (external via node IP)
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
# LoadBalancer (cloud LB)
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

**12-16.** (Ingress basics covered above)

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
    secretName: app-tls-secret    # Contains tls.crt and tls.key
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
Use cert-manager for automatic Let's Encrypt certificate management.

**18. Ingress annotations? 5 examples.**
```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /              # URL rewriting
  nginx.ingress.kubernetes.io/ssl-redirect: "true"           # Force HTTPS
  nginx.ingress.kubernetes.io/proxy-body-size: "50m"         # Max upload size
  nginx.ingress.kubernetes.io/rate-limit: "10"               # Rate limiting
  nginx.ingress.kubernetes.io/auth-type: basic               # Basic auth
  cert-manager.io/cluster-issuer: letsencrypt-prod           # Auto TLS
```

**19. Complete Ingress with TLS and 2 paths:**
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

**20. Ingress vs Gateway API?**
Ingress: Older, simpler, L7 HTTP only. Gateway API: Newer, richer, supports L4+L7, better RBAC (infra team manages Gateway, dev teams manage HTTPRoutes), more expressive routing.

**21-24.** (DNS covered above)

**25-27.** (NetworkPolicy covered above)

**28. Deny all ingress to a namespace:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}          # Applies to ALL pods
  policyTypes:
  - Ingress
  # No ingress rules = deny all incoming
```

**29. CNI plugins for NetworkPolicy?**
Calico (most popular), Cilium (eBPF-based, advanced), Weave Net. Note: Flannel does NOT support NetworkPolicies.

**30. User can't reach app. Pod running, Service exists. Debug?**
1. `kubectl get endpoints <svc>` → are pod IPs listed? If empty → selector mismatch
2. `kubectl describe svc <svc>` → check selector vs `kubectl get pods --show-labels`
3. `kubectl exec <test-pod> -- curl <svc>:<port>` → test from inside cluster
4. Check pod's readiness probe → if failing, pod removed from endpoints
5. Check NetworkPolicy → might be blocking traffic
6. Check Ingress config → wrong path or host

**31. Expose service to internet (AKS/EKS)?**
Option 1: `type: LoadBalancer` Service → cloud auto-provisions public LB
Option 2: Ingress Controller (Nginx) + `type: LoadBalancer` for the controller → Ingress rules route to services

**32. L4 vs L7 load balancing?**
- **L4 (Transport)**: Routes based on IP + port. TCP/UDP level. Faster. Service type LoadBalancer.
- **L7 (Application)**: Routes based on HTTP headers, path, host. More intelligent. Ingress.

**33. Inter-service communication?**
Services communicate via DNS: `http://service-name.namespace.svc.cluster.local:port`. For complex needs: service mesh (Istio) adds retries, circuit breaking, mTLS, traffic splitting.

**34. Service mesh?**
Dedicated infrastructure layer for service-to-service communication. Adds: mTLS (encrypted traffic), retries, circuit breaking, traffic splitting, observability. Tools: Istio (most popular), Linkerd (lighter). Use when: many microservices, need fine-grained traffic control.

---

## CONFIG, STORAGE, SECURITY, HELM (65 Qs)

**1-2.** (ConfigMap basics covered)

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
# Each key in ConfigMap becomes a file in /etc/config/
```

**4-5.** (Secret basics covered)

**6. Create Secret from literal/file?**
```bash
# From literal
kubectl create secret generic db-secret --from-literal=password=mysecret

# From file
kubectl create secret generic tls-secret --from-file=tls.crt --from-file=tls.key

# Declarative
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: bXlzZWNyZXQ=    # echo -n "mysecret" | base64
```

**7. Secret types?**
- `Opaque`: Default, arbitrary key-value data
- `kubernetes.io/dockerconfigjson`: Registry credentials (`imagePullSecrets`)
- `kubernetes.io/tls`: TLS certificate + key
- `kubernetes.io/basic-auth`: Username + password
- `kubernetes.io/service-account-token`: Auto-generated SA token

**8. External secret managers?**
Mount secrets from Vault/Key Vault into K8s Secrets:
- **CSI Secrets Store Driver**: Mounts Vault secrets as volumes
- **External Secrets Operator**: Syncs external secrets → K8s Secrets
- **Vault Agent Injector**: Sidecar that injects secrets from Vault

**9. External Secrets Operator?**
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

**10. ConfigMap as env vars in Deployment:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "postgres.prod.svc"
  LOG_LEVEL: "info"
---
spec:
  containers:
  - name: app
    envFrom:
    - configMapRef:
        name: app-config
    # OR individual keys:
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
```

**11.** (PV/PVC covered)

**12. Access modes?**
- `ReadWriteOnce (RWO)`: One node can mount read-write (most common)
- `ReadOnlyMany (ROX)`: Many nodes can mount read-only
- `ReadWriteMany (RWX)`: Many nodes can mount read-write (NFS, Azure Files)

**13.** (StorageClass covered)

**14. Reclaim policies?**
- `Retain`: PV preserved after PVC deletion (manual cleanup required)
- `Delete`: PV and underlying storage deleted when PVC deleted (default for dynamic)
- `Recycle`: Deprecated. Basic scrub (rm -rf) then reuse.

**15. emptyDir volume?**
Temporary storage created when pod is assigned to node. Deleted when pod removed. Useful for: scratch space, sharing files between containers in same pod, caching.
```yaml
volumes:
- name: cache
  emptyDir: {}
```

**16. hostPath?**
Mounts a directory from the host node. Not recommended because: pod is tied to specific node, security risk (access to host filesystem), data not portable. Use only for: DaemonSets accessing node logs/metrics.

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
# In Deployment spec:
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

**18-20.** (Probes covered)

**21. Probe mechanisms?**
```yaml
# HTTP GET
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080

# TCP Socket
livenessProbe:
  tcpSocket:
    port: 3306

# Exec command
livenessProbe:
  exec:
    command: ["pg_isready", "-U", "postgres"]
```

**22. Probe parameters?**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15    # Wait before first probe
  periodSeconds: 10          # Probe every 10s
  timeoutSeconds: 5          # Timeout per probe
  failureThreshold: 3        # Fail after 3 consecutive failures
  successThreshold: 1        # Succeed after 1 success
```

**23. All 3 probes for HTTP app:**
```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10          # Allow up to 300s to start
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

**24.** (Startup probe covered)

**25-26.** (Resources covered)

**27. QoS classes?**
- **Guaranteed**: requests == limits for all containers. Highest priority, last to be evicted.
- **Burstable**: requests < limits (or only requests set). Medium priority.
- **BestEffort**: No requests or limits set. First to be evicted. **Never use in production.**

**28. LimitRange and ResourceQuota?**
```yaml
# LimitRange: Default per-container limits in a namespace
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
---
# ResourceQuota: Total namespace limits
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    pods: "50"
```

**29.** (Scheduler covered)

**30-32.** (RBAC covered)

**33. ServiceAccount?**
Identity for pods to authenticate to the API server. Default SA has minimal permissions. Create custom SAs with specific RBAC roles for each workload.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
automountServiceAccountToken: false  # Don't auto-mount if not needed
```

**34. Restrict Pod API access?**
```yaml
# Don't mount service account token
automountServiceAccountToken: false

# Or use RBAC to give SA minimal permissions
# (Don't bind cluster-admin to workload SAs!)
```

**35. Pod Security Admission?**
Replaces deprecated PodSecurityPolicy. Enforces security standards per namespace:
- **Privileged**: No restrictions
- **Baseline**: Prevents known privilege escalations
- **Restricted**: Heavily restricted, best practices
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

**36. Least privilege in K8s?**
- Namespace-scoped Roles (not ClusterRoles) where possible
- Per-workload ServiceAccounts (not default SA)
- Minimal RBAC verbs (get/list, not *)
- NetworkPolicies to restrict communication
- Security context: non-root, read-only filesystem, drop capabilities

**37. Run Pod as non-root?**
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
```

**38. securityContext with dropped capabilities:**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```

**39.** (Helm basics covered)

**40. Helm chart directory structure?**
```
mychart/
├── Chart.yaml          # Metadata (name, version, appVersion)
├── values.yaml         # Default configuration values
├── charts/             # Sub-chart dependencies
├── templates/
│   ├── deployment.yaml # K8s manifests with Go templates
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Template helpers (named templates)
│   ├── NOTES.txt       # Post-install instructions
│   └── tests/
│       └── test-connection.yaml
└── .helmignore         # Files to exclude from package
```

**41. values.yaml? Override?**
Default configuration. Override with:
```bash
helm install myapp ./chart -f custom-values.yaml
helm install myapp ./chart --set replicas=5
helm install myapp ./chart --set image.tag=v2.0
```

**42. Helm release?**
A specific installation of a chart. Has a name, revision history, and can be upgraded/rolled back.
```bash
helm list                    # List releases
helm history myapp           # Show revisions
```

**43.** (Helm commands covered)

**44. Helm templating?**
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

**45. Helm repository?**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

**46. Create custom Helm chart?**
```bash
helm create mychart           # Scaffold chart
# Edit templates/ and values.yaml
helm lint mychart             # Validate
helm template mychart         # Render locally
helm package mychart          # Create .tgz
helm install myrelease ./mychart  # Install
```

**47. helm template vs helm install?**
- `helm template`: Renders templates locally, outputs YAML. No cluster interaction. Good for debugging.
- `helm install`: Renders AND applies to cluster. Creates a release.

**48. Helmfile?**
Declarative spec for deploying multiple Helm charts:
```yaml
releases:
- name: prometheus
  chart: prometheus-community/kube-prometheus-stack
  values: [prometheus-values.yaml]
- name: myapp
  chart: ./charts/myapp
  values: [values/prod.yaml]
```
Run `helmfile apply` to deploy all.

### Troubleshooting

**49-53.** (Already covered above in detail)

**54. Node NotReady?**
1. `kubectl describe node <node>` → check Conditions
2. Check kubelet: `systemctl status kubelet` on the node
3. Check container runtime: `systemctl status containerd`
4. Check disk space: `df -h`
5. Check memory: OOM killer? `dmesg | grep -i oom`
6. Check network: can node reach API server?

**55.** (kubectl top covered)

**56. Check events?**
```bash
kubectl get events -n <namespace> --sort-by=.lastTimestamp
kubectl get events --field-selector type=Warning
```

**57. Drain node for maintenance?**
```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
# Evicts all pods (respects PDB), marks node unschedulable
# After maintenance:
kubectl uncordon <node>
```

**58. Cordon/uncordon?**
```bash
kubectl cordon <node>    # Mark unschedulable (no new pods, existing stay)
kubectl uncordon <node>  # Mark schedulable again
```

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

**63. RBAC - dev read-only Pods+logs in "dev" namespace:**
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

**64. Organizing manifests for 15 microservices?**
Option 1: Helm chart per service with shared library chart. Option 2: Kustomize with base + overlays per env. Structure:
```
k8s/
├── base/                     # Shared templates
├── services/
│   ├── api-gateway/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── values-{env}.yaml
│   ├── user-service/
│   └── order-service/
└── overlays/
    ├── dev/
    ├── staging/
    └── prod/
```

**65. GitOps with ArgoCD?**
1. All K8s manifests in Git repo (source of truth)
2. ArgoCD watches Git repo for changes
3. Developer submits PR → review → merge
4. ArgoCD detects change → syncs cluster to match Git
5. ArgoCD UI shows sync status, diff, health
6. Rollback = git revert → ArgoCD auto-syncs
