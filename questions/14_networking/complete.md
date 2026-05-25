# Networking Fundamentals - COMPLETE
## Questions Only - Test Yourself

### OSI & TCP/IP Model
1. What are the 7 layers of the OSI model? Name each and its function.
2. What are the 4 layers of the TCP/IP model?
3. At which layer does HTTP operate? TCP? IP? Ethernet?
4. What is the difference between L4 and L7 load balancing?
5. What is a socket?

### IP Addressing & Subnetting
6. What is an IP address? IPv4 vs IPv6?
7. What is a subnet mask? What does /24 mean?
8. What is CIDR notation?
9. What are private IP ranges? (10.x, 172.16-31.x, 192.168.x)
10. What is NAT? What types exist? (SNAT, DNAT, PAT)
11. What is a gateway? What is a default gateway?
12. What is a VLAN?
13. What is the difference between a hub, switch, and router?

### DNS
14. What is DNS? How does DNS resolution work step by step?
15. What are the DNS record types? (A, AAAA, CNAME, MX, TXT, NS, SOA, PTR, SRV)
16. What is a DNS zone? Zone file?
17. What is TTL in DNS?
18. What is DNS caching? Where does it happen?
19. What is a DNS round-robin?
20. How do you troubleshoot DNS issues? (dig, nslookup, host)

### HTTP/HTTPS
21. What is HTTP? What is HTTPS?
22. What are HTTP methods? (GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS)
23. What are HTTP status codes? Name 5 from each category (1xx, 2xx, 3xx, 4xx, 5xx).
24. What are HTTP headers? Name 10 common ones.
25. What is the difference between HTTP/1.1, HTTP/2, and HTTP/3?
26. What is a REST API? What are REST principles?
27. What is a webhook? How is it different from polling?

### TLS/SSL
28. What is TLS? What is SSL? Which should you use?
29. How does the TLS handshake work?
30. What is a certificate? What is a CA (Certificate Authority)?
31. What is a self-signed certificate? When would you use one?
32. What is mTLS (mutual TLS)?
33. What is Let's Encrypt? How does it work?
34. How do you troubleshoot TLS certificate issues?

### Load Balancing & Proxies
35. What is a load balancer? Why is it needed?
36. What load balancing algorithms exist? (round-robin, least connections, IP hash, weighted)
37. What is a reverse proxy? (Nginx, HAProxy, Envoy)
38. What is the difference between a load balancer and a reverse proxy?
39. What is health checking in load balancing?
40. What is session persistence/sticky sessions?
41. What is SSL termination?

### Firewalls & Security
42. What is a firewall? Types? (network, host-based, WAF)
43. What is a WAF? How is it different from a regular firewall?
44. What is DDoS? How do you protect against it?
45. What is an ACL (Access Control List)?

### Interview-Style
46. A user reports they can't reach your application. Walk through the network debugging steps.
47. Explain how a request travels from a user's browser to your app running in K8s.
48. How do you set up TLS for a Kubernetes Ingress?
49. Your DNS change isn't propagating. What do you check?
50. What is the difference between TCP and UDP? When would you use each?
51. How do you design a highly available network architecture?
52. What is a CDN? When would you use one?
53. A service is timing out intermittently. How do you diagnose network issues?
54. What is a VPN? Types? (site-to-site, point-to-site)
55. What is a bastion host / jump box? Why use one?
