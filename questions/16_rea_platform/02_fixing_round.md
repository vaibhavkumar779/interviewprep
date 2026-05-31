# Round 2: Server/Website Deployed Fixing Round — REA Platform

> They deploy a broken service and you troubleshoot + fix it live.
> This tests real-world debugging skills — logs, kubectl, networking, configs.

---

## TROUBLESHOOTING FRAMEWORK (Use This Every Time)

```
1. OBSERVE → What's the symptom? (HTTP 500, pod CrashLoop, timeout)
2. NARROW  → Where in the stack? (DNS, network, app, config, resource)
3. EXAMINE → Read logs, events, describe resources
4. HYPOTHESIZE → What could cause this?
5. FIX → Apply the smallest fix, verify
6. VERIFY → Confirm service is healthy end-to-end
```

---

## SCENARIO 1: Pod in CrashLoopBackOff

### Diagnosis Steps
```bash
# 1. Check pod status
kubectl get pods -n rea-app
# Output: property-api-7d8f9c6b4-x2k9p   0/1   CrashLoopBackOff   5   3m

# 2. Describe pod for events
kubectl describe pod property-api-7d8f9c6b4-x2k9p -n rea-app
# Look for: Events section — OOMKilled? ImagePullBackOff? Liveness probe failed?

# 3. Check logs (current and previous crash)
kubectl logs property-api-7d8f9c6b4-x2k9p -n rea-app
kubectl logs property-api-7d8f9c6b4-x2k9p -n rea-app --previous

# 4. Common causes and fixes:
```

### Common Causes & Fixes

| Symptom in Logs/Events | Root Cause | Fix |
|---|---|---|
| `OOMKilled` | Memory limit too low | Increase `resources.limits.memory` |
| `exec format error` | Wrong image arch (amd64 vs arm64) | Build for correct platform |
| `connection refused localhost:5432` | DB not reachable | Check Service name, namespace, NetworkPolicy |
| `permission denied` | SecurityContext issue | Add `runAsUser`, check RBAC |
| `no such file: /app/config.yaml` | ConfigMap not mounted | Check volumeMounts + volumes |
| `env: DB_PASSWORD not set` | Missing Secret reference | Check secretKeyRef exists |
| Liveness probe failed | App not ready on probe path | Fix probe path/port/initialDelaySeconds |

---

## SCENARIO 2: Service Returns HTTP 502/504

### Diagnosis Steps
```bash
# 1. Check if pods are running
kubectl get pods -n rea-app -l app=property-api

# 2. Check service endpoints (are pods registered?)
kubectl get endpoints property-api -n rea-app
# If empty → selector mismatch between Service and Pod labels

# 3. Check service definition
kubectl get svc property-api -n rea-app -o yaml
# Verify: selector labels match pod labels, targetPort matches containerPort

# 4. Test from within cluster
kubectl run debug --rm -it --image=busybox -- sh
wget -qO- http://property-api.rea-app.svc.cluster.local/healthz

# 5. Check ingress/load balancer
kubectl get ingress -n rea-app
kubectl describe ingress property-api -n rea-app

# 6. Check if app is listening on correct port
kubectl exec -it <pod> -n rea-app -- netstat -tlnp
# or: kubectl exec -it <pod> -n rea-app -- ss -tlnp
```

### Common Causes & Fixes

| Symptom | Root Cause | Fix |
|---|---|---|
| Endpoints empty | Service selector doesn't match pod labels | Fix `spec.selector` in Service |
| 502 from ingress | Backend pods not ready | Check readinessProbe, increase timeout |
| 504 gateway timeout | App taking too long to respond | Check app logs, DB connection, increase timeout |
| Connection refused | App listening on wrong port | Match containerPort with targetPort |
| DNS resolution failed | Service name wrong in config | Use `<svc>.<ns>.svc.cluster.local` |

---

## SCENARIO 3: Deployment Rollout Stuck

### Diagnosis Steps
```bash
# 1. Check rollout status
kubectl rollout status deployment/property-api -n rea-app
# "Waiting for deployment ... rollout to finish: 1 old replicas are pending termination"

# 2. Check replica sets
kubectl get rs -n rea-app | grep property-api
# Look for: new RS with 0 READY

# 3. Check events on new pods
kubectl describe pod <new-pod> -n rea-app
# Events: ImagePullBackOff? Insufficient CPU? Node affinity?

# 4. Common stuck reasons:
# - PodDisruptionBudget blocking old pod termination
kubectl get pdb -n rea-app
# - Insufficient cluster resources
kubectl describe nodes | grep -A5 "Allocated resources"
# - Image doesn't exist
kubectl get events -n rea-app --sort-by=.metadata.creationTimestamp | tail -20
```

### Fixes
```bash
# Rollback to previous working version
kubectl rollout undo deployment/property-api -n rea-app

# Or fix and re-apply
kubectl set image deployment/property-api api=ecr.aws/property-api:v2.0.1 -n rea-app
```

---

## SCENARIO 4: ConfigMap/Secret Issues

### Diagnosis Steps
```bash
# App crashes with "config file not found" or "env var missing"

# 1. Check if ConfigMap exists
kubectl get configmap app-config -n rea-app
kubectl get configmap app-config -n rea-app -o yaml

# 2. Check if Secret exists
kubectl get secret db-creds -n rea-app
kubectl get secret db-creds -n rea-app -o jsonpath='{.data.password}' | base64 -d

# 3. Check volume mounts in pod spec
kubectl get deployment property-api -n rea-app -o yaml | grep -A20 volumes

# 4. Exec into pod and verify
kubectl exec -it <pod> -n rea-app -- ls -la /etc/config/
kubectl exec -it <pod> -n rea-app -- env | grep DB_
```

### Common Fixes
```yaml
# Fix 1: ConfigMap not mounted
volumes:
- name: config
  configMap:
    name: app-config    # ← Must match ConfigMap name
volumeMounts:
- name: config
  mountPath: /etc/config
  readOnly: true

# Fix 2: Secret env not injected
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-creds    # ← Must match Secret name
      key: password      # ← Must match key in Secret data
```

---

## SCENARIO 5: Network Policy Blocking Traffic

```bash
# App works in dev but not in staging — returns connection timeout

# 1. Check network policies
kubectl get networkpolicy -n rea-staging

# 2. Describe the policy
kubectl describe networkpolicy deny-all -n rea-staging
# If it's deny-all with no ingress rules → traffic blocked

# 3. Fix: Add ingress rule for the service
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-property-api
  namespace: rea-staging
spec:
  podSelector:
    matchLabels:
      app: property-api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: rea-gateway
    ports:
    - protocol: TCP
      port: 8080
```

---

## SCENARIO 6: High Latency / Slow Response

```bash
# 1. Check resource utilization
kubectl top pods -n rea-app
kubectl top nodes

# 2. Check if HPA is maxed out
kubectl get hpa -n rea-app

# 3. Check for CPU/memory throttling
kubectl describe pod <pod> -n rea-app | grep -A5 "Limits"
# If requests.cpu = limits.cpu AND usage is high → throttled

# 4. Check external dependencies
kubectl exec -it <pod> -- curl -w "@curl-format.txt" http://db-service:5432
# Measure DNS, connect, TLS, transfer times

# 5. Check for noisy neighbors on the node
kubectl get pods -o wide | grep <node-name>
```

---

## SPLUNK QUERY BASICS (REA uses Splunk)

```spl
# Basic search
index=rea_platform sourcetype=kubernetes source=property-api
| where status >= 500
| stats count by status, uri

# Error rate over time
index=rea_platform source=property-api
| timechart span=5m count(eval(status>=500)) as errors, count as total
| eval error_rate = round(errors/total*100, 2)

# Slow requests (p99 latency)
index=rea_platform source=property-api
| stats perc99(response_time) as p99 by service
| where p99 > 500

# Pod crash events
index=rea_platform sourcetype=kube:events
| search reason="CrashLoopBackOff" OR reason="OOMKilled"
| stats count by involvedObject.name, reason
| sort -count

# Trace a request by correlation ID
index=rea_platform trace_id="abc123"
| sort _time
| table _time, service, method, uri, status, duration_ms
```

---

## QUICK REFERENCE: kubectl Commands for Debugging

```bash
# Pod diagnostics
kubectl get pods -n <ns> -o wide              # Pod status + node
kubectl describe pod <pod> -n <ns>             # Events, conditions
kubectl logs <pod> -n <ns> -f                  # Stream logs
kubectl logs <pod> -n <ns> --previous          # Previous crash logs
kubectl logs <pod> -n <ns> -c <container>      # Multi-container pod
kubectl exec -it <pod> -n <ns> -- sh           # Shell into pod

# Service diagnostics
kubectl get endpoints <svc> -n <ns>            # Registered backends
kubectl get svc <svc> -n <ns> -o yaml          # Service definition
kubectl port-forward svc/<svc> 8080:80 -n <ns> # Local port forward

# Resource diagnostics
kubectl top pods -n <ns>                       # CPU/memory usage
kubectl top nodes                               # Node utilization
kubectl get events -n <ns> --sort-by='.metadata.creationTimestamp'

# Rollout
kubectl rollout status deployment/<name> -n <ns>
kubectl rollout history deployment/<name> -n <ns>
kubectl rollout undo deployment/<name> -n <ns>
```
