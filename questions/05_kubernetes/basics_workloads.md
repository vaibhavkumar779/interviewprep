# Kubernetes - BASICS & CORE OBJECTS
## Questions Only - Test Yourself

### Fundamentals
1. What is Kubernetes? What problem does it solve?
2. What is container orchestration? Why can't you just use Docker?
3. Explain the Kubernetes architecture. Draw it. (Control plane, worker nodes)
4. What are the control plane components? (API server, etcd, scheduler, controller manager)
5. What is the kubelet? Where does it run?
6. What is kube-proxy? What does it do?
7. What is etcd? Why is it critical?
8. What is the Kubernetes API server? How do you interact with it?
9. What is kubectl? Name 10 kubectl commands you use daily.
10. What is a namespace? When would you create one?
11. What is a node? How do you add a node to a cluster?
12. What is the difference between a managed K8s (AKS, EKS, GKE) and self-managed?

### Pods
13. What is a Pod? Why not just run containers directly?
14. Can a Pod have multiple containers? When would you do this?
15. What is a sidecar container? Give 3 examples.
16. What is an init container? When is it useful?
17. What is the Pod lifecycle? (Pending, Running, Succeeded, Failed, Unknown)
18. How do you get logs from a Pod? From a previous crashed instance?
19. How do you exec into a Pod?
20. What happens when a Pod is deleted?

### Workloads
21. What is a Deployment? What does it manage?
22. What is a ReplicaSet? How is it different from a Deployment?
23. What is a DaemonSet? Give 3 use cases.
24. What is a StatefulSet? When would you use it instead of Deployment?
25. What is a Job? What is a CronJob?
26. What is the difference between rolling update and recreate strategies?
27. What are maxSurge and maxUnavailable in rolling updates?
28. How do you rollback a Deployment? Write the kubectl command.
29. How do you scale a Deployment? (manual and auto)
30. What is the Horizontal Pod Autoscaler (HPA)? What metrics does it use?

### Interview-Style
31. Write a complete Deployment manifest from scratch for an nginx app with 3 replicas.
32. Your Pod is in Pending state. What are 5 possible reasons?
33. Your Pod keeps restarting (CrashLoopBackOff). Walk through your debugging steps.
34. How do you perform zero-downtime deployments in K8s?
35. Explain the flow: User types URL → request reaches your app in K8s.
