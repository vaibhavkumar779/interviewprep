# Kubernetes - CONFIG, STORAGE, SECURITY, HELM & TROUBLESHOOTING
## Questions Only - Test Yourself

### ConfigMaps & Secrets
1. What is a ConfigMap? How do you create one? (imperative and declarative)
2. How do you use a ConfigMap as environment variables?
3. How do you mount a ConfigMap as a file/volume?
4. What is a Secret? How is it different from a ConfigMap?
5. Are Secrets encrypted by default? How do you enable encryption at rest?
6. How do you create a Secret from a literal value? From a file?
7. What are the types of Secrets? (Opaque, docker-registry, tls)
8. How do you use external secret managers with K8s? (Vault, Azure Key Vault, AWS Secrets Manager)
9. What is the External Secrets Operator?
10. Write a ConfigMap and mount it as environment variables in a Deployment.

### Storage
11. What is a PersistentVolume (PV)? What is a PersistentVolumeClaim (PVC)?
12. What are access modes? (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
13. What is a StorageClass? What is dynamic provisioning?
14. What reclaim policies exist? (Retain, Delete, Recycle)
15. What is an emptyDir volume? When is it useful?
16. What is a hostPath volume? Why is it generally not recommended?
17. Write a PVC and mount it in a Deployment.

### Probes
18. What are the 3 types of probes? (liveness, readiness, startup)
19. What happens when a liveness probe fails?
20. What happens when a readiness probe fails?
21. What probe mechanisms exist? (httpGet, tcpSocket, exec)
22. What are initialDelaySeconds, periodSeconds, failureThreshold?
23. Write all 3 probes for an HTTP application.
24. When would you use a startup probe?

### Resource Management
25. What are resource requests and limits?
26. What happens when a container exceeds its CPU limit? Memory limit?
27. What is QoS class? Name all 3. (Guaranteed, Burstable, BestEffort)
28. What is a LimitRange? What is a ResourceQuota?
29. How does the scheduler use resource requests?

### Security & RBAC
30. What is RBAC in Kubernetes?
31. What is a Role vs ClusterRole?
32. What is a RoleBinding vs ClusterRoleBinding?
33. What is a ServiceAccount? Why not use the default one?
34. How do you restrict a Pod from accessing the K8s API?
35. What is a PodSecurityPolicy / Pod Security Admission?
36. What is the principle of least privilege in K8s context?
37. How do you run a Pod as non-root?
38. What is securityContext? Write one that drops all capabilities.

### Helm
39. What is Helm? What problem does it solve?
40. What is a Helm chart? What is its directory structure?
41. What is values.yaml? How do you override values?
42. What is a Helm release?
43. What is `helm install`, `helm upgrade`, `helm rollback`?
44. How do you template a Kubernetes manifest with Helm? ({{ .Values.xxx }})
45. What is a Helm repository?
46. How do you create a custom Helm chart?
47. What is `helm template` vs `helm install`?
48. What is helmfile?

### Troubleshooting
49. Pod is in Pending state - list 5 reasons and how to debug each.
50. Pod is in CrashLoopBackOff - list 5 reasons and how to debug each.
51. Pod is in ImagePullBackOff - list 3 reasons.
52. Pod is Running but not receiving traffic - what do you check?
53. Service is not reachable - walk through the debugging steps.
54. Node is in NotReady state - what do you check?
55. How do you check resource usage of Pods? (kubectl top)
56. How do you check events in a namespace?
57. How do you drain a node for maintenance?
58. How do you cordon/uncordon a node?

### Interview-Style (Write Manifests)
59. Write a complete set of manifests: Deployment + Service + ConfigMap + Secret + Ingress for a web app.
60. Write a StatefulSet for a PostgreSQL database with persistent storage.
61. Write a CronJob that runs a cleanup script every day at midnight.
62. Write a DaemonSet that runs a log collector on every node.
63. Write RBAC rules that give a developer read-only access to Pods and logs in the "dev" namespace.
64. Your team is deploying 15 microservices. How do you organize the K8s manifests?
65. How do you implement GitOps with K8s? (ArgoCD or Flux workflow)
