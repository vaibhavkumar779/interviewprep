# Kubernetes - NETWORKING & SERVICES
## Questions Only - Test Yourself

### Services
1. What is a Kubernetes Service? Why do you need it?
2. What are the 4 types of Services? (ClusterIP, NodePort, LoadBalancer, ExternalName)
3. When would you use each Service type?
4. What is a ClusterIP service? Can you access it from outside the cluster?
5. What is a NodePort service? What is the port range?
6. What is a LoadBalancer service? How does it work in cloud vs bare-metal?
7. What is an ExternalName service?
8. How does a Service discover Pods? (label selectors)
9. What is the endpoints object in a Service?
10. What is a headless Service? When would you use it?
11. Write a Service manifest for ClusterIP, NodePort, and LoadBalancer types.

### Ingress
12. What is an Ingress? What layer of the OSI model does it operate on?
13. What is an Ingress Controller? Name 3 Ingress Controllers.
14. What is the difference between a Service and an Ingress?
15. How do you set up path-based routing with Ingress?
16. How do you set up host-based routing with Ingress?
17. How do you configure TLS/SSL with Ingress?
18. What are Ingress annotations? Give 5 examples.
19. Write a complete Ingress manifest with TLS and 2 path rules.
20. What is the difference between Ingress and Gateway API?

### DNS & Service Discovery
21. How does DNS work inside Kubernetes?
22. What is CoreDNS?
23. How do you resolve a Service from another namespace?
24. What is the full DNS name of a Service? (svc.namespace.svc.cluster.local)

### Network Policies
25. What is a NetworkPolicy? What does it control?
26. By default, can all Pods communicate with each other?
27. Write a NetworkPolicy that allows only frontend Pods to talk to backend Pods.
28. Write a NetworkPolicy that denies all ingress to a namespace.
29. What CNI plugin is required for NetworkPolicies to work? Name 3.

### Interview-Style
30. A user can't reach your app. The Pod is running, the Service exists. How do you debug?
31. How do you expose a service to the internet in AKS/EKS?
32. What is the difference between L4 and L7 load balancing in K8s context?
33. How do you handle inter-service communication in a microservices architecture on K8s?
34. What is a service mesh? When would you use one? (Istio, Linkerd)
