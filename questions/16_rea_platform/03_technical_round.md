# Round 3: Technical Deep-Dive — REA Platform Engineer

> Covers: Kubernetes internals, platform engineering, SLI/SLO/SLA, chaos engineering,
> AWS, observability, capacity planning, developer experience

---

## 1. KUBERNETES DEEP-DIVE

### Q: Explain the Kubernetes architecture. What happens when you run `kubectl apply -f deployment.yaml`?

**A:**
1. `kubectl` sends the manifest to the **API Server** (via REST/HTTPS)
2. **API Server** validates the request (admission controllers, RBAC), persists to **etcd**
3. **Scheduler** watches for unscheduled pods, selects a node based on resource requests, affinity, taints/tolerations
4. **Kubelet** on the selected node receives the pod spec, pulls the image, creates containers via **container runtime** (containerd)
5. **Controller Manager** — the Deployment controller creates a ReplicaSet, the ReplicaSet controller ensures desired replica count
6. **kube-proxy** updates iptables/IPVS rules so the Service can route traffic to the pod

### Q: What is the difference between a Deployment, StatefulSet, and DaemonSet?

**A:**
| | Deployment | StatefulSet | DaemonSet |
|---|---|---|---|
| Use case | Stateless apps | Stateful apps (DBs, queues) | One pod per node (monitoring, logging) |
| Pod identity | Random names | Stable ordinal names (web-0, web-1) | One per node |
| Storage | Shared/no PVC | Per-pod PVC (persistent) | Node-local |
| Scaling | Parallel | Sequential (ordered) | Automatic with nodes |
| Rollout | RollingUpdate | Ordered (one at a time) | RollingUpdate |

### Q: Explain Kubernetes RBAC. How would you restrict a team to only their namespace?

**A:**
```yaml
# Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: team-frontend
  name: team-frontend-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "deployments", "services", "configmaps", "secrets", "jobs"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "create"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: team-frontend
  name: team-frontend-binding
subjects:
- kind: Group
  name: "frontend-developers"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: team-frontend-role
  apiGroup: rbac.authorization.k8s.io
```
Key: Role is namespace-scoped (vs ClusterRole which is cluster-wide). RoleBinding binds the role to users/groups in a specific namespace.

### Q: How do you handle secrets in Kubernetes securely?

**A:**
1. **Never store in Git** — K8s Secrets are base64-encoded, not encrypted
2. **External secret managers**: AWS Secrets Manager, Azure Key Vault, HashiCorp Vault
3. **CSI Secret Store Driver** — mounts external secrets as volumes
4. **Sealed Secrets** — encrypted secrets safe to store in Git (Bitnami)
5. **RBAC** — restrict who can read secrets
6. **Encryption at rest** — configure etcd encryption
7. **Rotate regularly** — automated rotation via external providers

My experience: Migrated secrets from Helm values to Azure Key Vault with CSI driver across 4 product charts.

### Q: Explain Kubernetes networking. How does a request reach a pod?

**A:**
1. **External request** → Ingress Controller (NGINX/ALB) → routes based on host/path rules
2. **Ingress** → **Service** (ClusterIP) → kube-proxy uses iptables/IPVS to select a pod
3. **Pod-to-Pod**: CNI plugin (Calico, Cilium, AWS VPC CNI) — every pod gets a unique IP
4. **Pod-to-Service**: CoreDNS resolves `<svc>.<ns>.svc.cluster.local` → ClusterIP → iptables → pod
5. **Network Policies**: Like firewalls at pod level — allow/deny traffic by labels, namespaces, ports

### Q: What are Admission Controllers? Give examples.

**A:**
Admission controllers intercept API requests after authentication/authorization but before persistence.

Types:
- **Validating**: Reject requests that violate rules (e.g., "no pods without resource limits")
- **Mutating**: Modify requests (e.g., "inject sidecar container")

Examples:
- `LimitRanger` — enforce default resource limits
- `PodSecurityAdmission` — enforce pod security standards
- `OPA Gatekeeper` — custom policy engine (Rego policies)
- `Istio sidecar injector` — mutating webhook that adds Envoy sidecar

---

## 2. SLI/SLO/SLA & RELIABILITY (Critical for REA JD)

### Q: Define SLI, SLO, SLA, and Error Budget with examples.

**A:**
- **SLI (Service Level Indicator)**: A measurable metric of service quality
  - Example: Request latency (p99), error rate, availability
- **SLO (Service Level Objective)**: A target value for an SLI
  - Example: "99.9% of requests return 2xx in < 200ms"
- **SLA (Service Level Agreement)**: A business contract around SLOs with consequences
  - Example: "If availability drops below 99.5%, customer gets 10% credit"
- **Error Budget**: The acceptable amount of unreliability = 100% - SLO
  - Example: 99.9% SLO → 0.1% error budget → **43.2 min/month** of allowed downtime

### Q: How would you implement SLOs for REA's property search service?

**A:**
```
SLIs:
1. Availability: % of successful HTTP responses (non-5xx)
2. Latency: p99 response time for search queries
3. Throughput: Requests per second handled without degradation

SLOs:
1. Availability SLO: 99.9% (43 min/month error budget)
2. Latency SLO: p99 < 300ms for search results
3. Throughput SLO: Handle 10K RPS without degradation

Implementation:
- Prometheus recording rules for SLI calculations
- Grafana SLO dashboards showing burn rate
- Multi-window, multi-burn-rate alerting (fast + slow windows)
- Error budget tracking — if budget consumed >50%, freeze non-critical deployments
```

### Q: What is MTTD, MTTA, MTTR, MTBF?

**A:**
| Metric | Full Form | Meaning | Target |
|---|---|---|---|
| MTTD | Mean Time to Detect | Time from incident start to alert firing | < 5 min |
| MTTA | Mean Time to Acknowledge | Time from alert to engineer response | < 15 min |
| MTTR | Mean Time to Recover | Time from incident start to full recovery | < 1 hour |
| MTBF | Mean Time Between Failures | Average uptime between incidents | > 30 days |

---

## 3. CHAOS ENGINEERING

### Q: What is chaos engineering? Why is it important?

**A:**
Chaos engineering is the practice of **intentionally injecting failures** into a system to verify it can withstand real-world disruptions.

**Process:**
1. Define **steady state** (normal metrics: latency, error rate, throughput)
2. Form a **hypothesis** ("if we kill 1 of 3 replicas, latency stays < 300ms")
3. **Inject failure** (pod kill, network delay, CPU stress)
4. **Observe** — does the system behave as expected?
5. **Learn** — fix weaknesses, improve resilience

**Tools:**
- **Chaos Mesh** (K8s-native, CNCF)
- **Litmus Chaos** (K8s-native)
- **Gremlin** (SaaS, enterprise)
- **AWS Fault Injection Simulator** (AWS-native)

**Types of experiments:**
- Pod kill / container restart
- Network latency injection (200ms delay)
- Network partition (isolate a service)
- CPU/memory stress
- Disk I/O failure
- DNS failure

### Q: How would you run a chaos experiment safely in production?

**A:**
1. **Start in staging**, validate the experiment is safe
2. **Blast radius** — target a single pod/service, not entire cluster
3. **Abort conditions** — define automatic rollback triggers (error rate > 5%)
4. **Communicate** — inform the team via Slack/PagerDuty
5. **Run during business hours** when team is available
6. **Monitor in real-time** — Grafana dashboards during experiment
7. **Document findings** — postmortem-style report for each experiment

---

## 4. AWS DEEP-DIVE (REA is multi-cloud, primarily AWS)

### Q: Compare EKS vs self-managed K8s on EC2.

**A:**
| Aspect | EKS | Self-managed |
|---|---|---|
| Control plane | AWS-managed, HA | You manage, you patch |
| Cost | $0.10/hr per cluster + nodes | Nodes only |
| Upgrades | AWS handles control plane | You do everything |
| IAM integration | Native (IRSA, Pod Identity) | Manual setup |
| Networking | VPC CNI (native IPs) | Choose CNI |
| Best for | Production, compliance | Learning, extreme customization |

### Q: What is IRSA (IAM Roles for Service Accounts)?

**A:**
IRSA lets Kubernetes pods assume AWS IAM roles via their Service Account, without managing credentials.

```yaml
# 1. Create IAM role with trust policy for EKS OIDC
# 2. Annotate K8s ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-reader
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/s3-reader-role
# 3. Pod using this SA automatically gets temporary AWS credentials
```
No access keys, no secrets — the most secure way to grant AWS permissions to K8s workloads.

### Q: Explain VPC networking for EKS.

**A:**
- **VPC CNI**: Each pod gets a real VPC IP address (native networking)
- **Subnets**: Nodes in private subnets, load balancers in public subnets
- **Security Groups**: Can be applied at pod level (SGP — Security Groups for Pods)
- **NAT Gateway**: Private subnets use NAT for outbound internet (pulling images)
- **Endpoint**: EKS API endpoint can be public, private, or public+private

### Q: AWS observability tools — CloudWatch, X-Ray, CloudTrail.

**A:**
| Tool | Purpose |
|---|---|
| CloudWatch | Metrics, logs, alarms, dashboards |
| CloudWatch Logs Insights | Query logs with SQL-like syntax |
| X-Ray | Distributed tracing for microservices |
| CloudTrail | API audit trail (who did what) |
| Container Insights | EKS/ECS monitoring (CPU, memory, network per pod/node) |

---

## 5. OBSERVABILITY & MONITORING

### Q: How would you set up observability for a microservices platform?

**A: Three pillars:**
1. **Metrics** — Prometheus + Grafana (or CloudWatch)
   - Infrastructure: CPU, memory, disk, network
   - Application: request rate, error rate, latency (RED method)
   - Business: search queries/sec, listings viewed
2. **Logs** — ELK / Splunk / Loki
   - Structured JSON logs
   - Correlation IDs for request tracing
   - Log levels: ERROR for alerts, INFO for audit, DEBUG for troubleshooting
3. **Traces** — Jaeger / X-Ray / OpenTelemetry
   - Distributed tracing across microservices
   - Identify slow services in request chain
   - Trace sampling (1-10% in production)

### Q: RED vs USE method for monitoring.

**A:**
| Method | Metrics | Best For |
|---|---|---|
| RED | Rate, Errors, Duration | Request-driven services (APIs) |
| USE | Utilization, Saturation, Errors | Resource-driven (CPU, memory, disk) |

### Q: How do you set up alerting that doesn't cause alert fatigue?

**A:**
1. **Alert on symptoms, not causes** — alert on "error rate > 1%" not "CPU > 80%"
2. **Severity levels**: P1 (page immediately), P2 (page during hours), P3 (ticket)
3. **Runbooks** — every alert links to a runbook with diagnosis steps
4. **SLO-based alerting** — alert on error budget burn rate, not raw thresholds
5. **Deduplication** — group related alerts
6. **Review regularly** — remove alerts nobody acts on

---

## 6. PLATFORM ENGINEERING & DEVELOPER EXPERIENCE

### Q: What is Platform Engineering? How is it different from DevOps?

**A:**
| Aspect | DevOps | Platform Engineering |
|---|---|---|
| Focus | Culture + practices | Product (Internal Developer Platform) |
| Users | Dev + Ops collaboration | Developers are customers |
| Output | CI/CD, automation | Self-service platform |
| Metric | Deployment frequency, MTTR | Developer experience, onboarding time |

Platform Engineering treats **infrastructure as a product**. You build an Internal Developer Platform (IDP) that developers consume via self-service, reducing cognitive load.

### Q: What is an Internal Developer Platform (IDP)? What components does it have?

**A:**
1. **Service Catalog** (Backstage) — discover, own, and document services
2. **Self-service provisioning** — create namespaces, databases, queues via UI/CLI
3. **Golden paths** — standardized CI/CD templates teams use out of the box
4. **Guardrails** — OPA policies, resource quotas, security defaults
5. **Observability** — built-in dashboards per service
6. **Documentation** — auto-generated API docs, runbooks

### Q: How do you measure Developer Experience (DevEx)?

**A:**
- **DORA metrics**: Deployment frequency, lead time, change failure rate, MTTR
- **Developer satisfaction surveys** (quarterly)
- **Onboarding time**: How long for a new dev to ship first feature?
- **Self-service adoption**: % of infra provisioned via platform vs tickets
- **Cognitive load**: How many tools/steps to deploy?
- **Toil reduction**: Engineering time saved per quarter

---

## 7. CAPACITY PLANNING & SCALING

### Q: How do you approach capacity planning for a K8s platform?

**A:**
1. **Current state**: `kubectl top pods/nodes`, Prometheus utilization metrics
2. **Right-size pods**: Use VPA recommendations for request/limit tuning
3. **HPA**: Auto-scale pods on CPU/memory/custom metrics
4. **Cluster autoscaler**: Add/remove nodes based on pending pods
5. **Load test**: Use k6/Locust to simulate peak traffic
6. **Forecast**: Analyze growth trends (30/60/90 day) from Prometheus/Grafana
7. **Buffer**: Maintain 30% headroom for traffic spikes

### Q: HPA vs VPA vs Cluster Autoscaler — when to use which?

**A:**
| Autoscaler | Scales What | When to Use |
|---|---|---|
| HPA | Pod count (horizontal) | Stateless services with variable load |
| VPA | Pod resources (vertical) | Right-sizing requests/limits |
| Cluster Autoscaler | Node count | When HPA creates pods but no node capacity |
| KEDA | Pod count (event-driven) | Queue-based, event-driven workloads |

**Note**: Don't use HPA + VPA on the same metric (CPU) simultaneously — they conflict.

---

## 8. SCENARIO-BASED QUESTIONS

### Q: A developer says "my deployment works in dev but fails in production." How do you troubleshoot?

**A:**
1. **Compare configurations**: diff values files, env vars, secrets between dev and prod
2. **Check resource constraints**: prod may have stricter quotas, network policies
3. **Image tag**: Is it the same image version? Check image digest
4. **RBAC**: Does the service account have required permissions in prod?
5. **Network**: Are dependencies reachable? DNS resolution, firewall rules, network policies
6. **Secrets**: Are all secrets created in prod namespace?
7. **Logs**: Compare pod logs between environments
8. **Resource limits**: Prod may have different CPU/memory limits causing OOMKills

### Q: How would you migrate 50 microservices from one K8s cluster to another with zero downtime?

**A:**
1. **Inventory**: List all resources per namespace (Deployments, Services, ConfigMaps, Secrets, PVCs)
2. **Export & clean**: Export manifests, remove cluster-specific metadata
3. **Parallel run**: Deploy to new cluster alongside old
4. **DNS/traffic**: Use weighted DNS (Route 53) to gradually shift traffic (10% → 50% → 100%)
5. **Validate**: Run smoke tests, compare metrics between old and new
6. **Stateful data**: For databases/queues, set up replication before migration
7. **Rollback plan**: Keep old cluster running for 1-2 weeks as fallback
8. **Decommission**: After validation period, drain and delete old cluster
