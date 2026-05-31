# Networking Fundamentals - COMPREHENSIVE ANSWERS (All 55 Questions)

---

## OSI & TCP/IP Model

**1. 7 layers of OSI model?**

```
OSI Model (data flow ↓):
┌───────┬────────────────┬───────────────────────┬──────────────────────┐
│ Layer │ Name            │ Function               │ Protocols/Examples   │
├───────┼────────────────┼───────────────────────┼──────────────────────┤
│   7   │ Application     │ User-facing services    │ HTTP, DNS, SMTP, SSH │
│   6   │ Presentation    │ Encrypt, encode, zip   │ SSL/TLS, JPEG, ASCII │
│   5   │ Session         │ Session management     │ NetBIOS, RPC         │
│   4   │ Transport       │ End-to-end delivery    │ TCP, UDP             │
│   3   │ Network         │ Routing, IP addressing │ IP, ICMP, OSPF       │
│   2   │ Data Link       │ MAC addressing, frames │ Ethernet, ARP        │
│   1   │ Physical        │ Bits on wire/air       │ Cables, Wi-Fi, fiber │
└───────┴────────────────┴───────────────────────┴──────────────────────┘

Mnemonic: "All People Seem To Need Data Processing" (L7→L1)

How data flows:
  Sender:  App data → +segment → +packet → +frame → bits
  Receiver: bits → frame → packet → segment → App data
```

**2. TCP/IP model (4 layers)?**
1. **Application** (OSI 5-7): HTTP, DNS, SSH
2. **Transport** (OSI 4): TCP, UDP
3. **Internet** (OSI 3): IP, ICMP
4. **Network Access** (OSI 1-2): Ethernet, Wi-Fi

**3. Which layer?**
- HTTP: Layer 7 (Application)
- TCP: Layer 4 (Transport)
- IP: Layer 3 (Network)
- Ethernet: Layer 2 (Data Link)

**4. L4 vs L7 load balancing?**
- **L4**: Routes based on IP + port. TCP/UDP level. Fast. No content inspection. Example: AWS NLB.
- **L7**: Routes based on HTTP content (URL path, headers, cookies). Slower but smarter. Example: ALB, Nginx Ingress.

**5. What is a socket?**
Combination of IP address + port number. Uniquely identifies a connection endpoint. Example: `192.168.1.1:8080`. A TCP connection is identified by 4-tuple: source IP:port + dest IP:port.

---

## IP Addressing & Subnetting

**6. IP address? IPv4 vs IPv6?**
- **IPv4**: 32-bit, dotted decimal: `192.168.1.1`. ~4.3 billion addresses.
- **IPv6**: 128-bit, hex: `2001:0db8:85a3::8a2e:0370:7334`. Virtually unlimited.

**7. Subnet mask? /24?**
Defines which portion of IP is network vs host.
`/24` = `255.255.255.0` = first 24 bits are network, last 8 are host = 256 addresses (254 usable).

**8. CIDR notation?**
Classless Inter-Domain Routing. `10.0.0.0/16` = network 10.0.x.x with 65,536 addresses.
Common: `/24` = 256 IPs, `/16` = 65,536 IPs, `/8` = 16 million IPs.

**9. Private IP ranges?**
| Range | CIDR | Addresses |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16M |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1M |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65K |

**10. NAT? Types?**
Network Address Translation — maps private IPs to public IPs.
- **SNAT (Source NAT)**: Changes source IP. Outbound: private→public.
- **DNAT (Destination NAT)**: Changes destination IP. Inbound: public→private.
- **PAT (Port Address Translation)**: Many private IPs share one public IP using different ports. Most common.

**11. Gateway? Default gateway?**
- **Gateway**: Device that connects two different networks.
- **Default gateway**: Router that forwards traffic to destinations outside local network. Usually `x.x.x.1`.

**12. VLAN?**
Virtual LAN — logically segments a physical network. Devices in same VLAN communicate as if on same physical network. Used for: security isolation, traffic management.

**13. Hub vs switch vs router?**
- **Hub**: Broadcasts all traffic to all ports. Layer 1. Obsolete.
- **Switch**: Learns MAC addresses, forwards to specific port. Layer 2. Local network.
- **Router**: Routes between networks using IP addresses. Layer 3. Internet connectivity.

---

## DNS

**14. DNS? Resolution step by step?**
Domain Name System — translates domain names to IP addresses.

```
DNS Resolution Flow:

  Browser           OS              Recursive         Root     TLD(.com)   Authoritative
  Cache             Cache           Resolver          Server   Server      (example.com)
    │                │               │                  │        │           │
    │─▶ check      │               │                  │        │           │
    │  cache?      │               │                  │        │           │
    │  miss!       │               │                  │        │           │
    │───────────▶│               │                  │        │           │
    │             │─▶ /etc/hosts? │                  │        │           │
    │             │  miss!        │                  │        │           │
    │             │────────────▶ Who is .com?    │        │           │
    │             │               │───────────────▶│        │           │
    │             │               │◀─ .com NS server │        │           │
    │             │               │  Who is example?         │           │
    │             │               │──────────────────────▶│           │
    │             │               │◀─ example.com NS server   │           │
    │             │               │  Get A record                       │
    │             │               │───────────────────────────────▶│
    │             │               │◀─── 93.184.216.34 ───────────┘
    │             │◀── cached!     │
    │◀───────────┘               │
    │ 93.184.216.34
```

1. Browser checks cache
2. OS checks `/etc/hosts` and local cache
3. Query to recursive resolver (ISP DNS or 8.8.8.8)
4. Resolver queries root nameserver (`.`)
5. Root refers to TLD nameserver (`.com`)
6. TLD refers to authoritative nameserver (`example.com`)
7. Authoritative returns IP address
8. Resolver caches and returns to client

**15. DNS record types?**
| Record | Purpose | Example |
|---|---|---|
| A | Domain → IPv4 | `example.com → 93.184.216.34` |
| AAAA | Domain → IPv6 | `example.com → 2606:2800:...` |
| CNAME | Alias to another domain | `www → example.com` |
| MX | Mail server | `mail.example.com` |
| TXT | Text data (SPF, DKIM, verification) | `v=spf1 include:...` |
| NS | Nameserver for zone | `ns1.example.com` |
| SOA | Start of Authority (zone metadata) | Serial, refresh, retry |
| PTR | Reverse DNS (IP → domain) | `34.216.184.93 → example.com` |
| SRV | Service location (port + host) | `_sip._tcp.example.com` |

**16. DNS zone? Zone file?**
- **Zone**: Administrative portion of DNS namespace (e.g., everything under `example.com`)
- **Zone file**: Text file containing resource records for the zone

**17. TTL?**
Time To Live — how long DNS resolvers cache a record (in seconds). Low TTL (300s) = faster propagation. High TTL (86400s) = less DNS load.

**18. DNS caching? Where?**
1. Browser cache (~minutes)
2. OS cache (systemd-resolved, nscd)
3. Recursive resolver cache (ISP DNS)
4. Each level caches for TTL duration

**19. DNS round-robin?**
Multiple A records for same domain. DNS returns different IPs in rotation. Basic load balancing.
```
example.com → 1.2.3.4
example.com → 1.2.3.5
example.com → 1.2.3.6
```

**20. Troubleshoot DNS?**
```bash
dig example.com                    # Full DNS query
dig @8.8.8.8 example.com         # Use specific DNS server
dig +trace example.com            # Full resolution path
nslookup example.com              # Simple lookup
host example.com                   # Quick check
cat /etc/resolv.conf              # Check configured DNS servers
systemd-resolve --status          # systemd DNS config
```

---

## HTTP/HTTPS

**21. HTTP vs HTTPS?**
- **HTTP**: Hypertext Transfer Protocol. Port 80. Plaintext. Not secure.
- **HTTPS**: HTTP over TLS. Port 443. Encrypted. Always use HTTPS.

**22. HTTP methods?**
| Method | Purpose | Idempotent |
|---|---|---|
| GET | Retrieve data | Yes |
| POST | Create resource | No |
| PUT | Replace resource | Yes |
| PATCH | Partial update | No |
| DELETE | Remove resource | Yes |
| HEAD | GET without body | Yes |
| OPTIONS | Supported methods (CORS) | Yes |

**23. HTTP status codes?**
| Code | Meaning |
|---|---|
| **1xx** | Informational: 100 Continue, 101 Switching Protocols |
| **2xx** | Success: 200 OK, 201 Created, 204 No Content |
| **3xx** | Redirect: 301 Moved Permanently, 302 Found, 304 Not Modified |
| **4xx** | Client Error: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| **5xx** | Server Error: 500 Internal, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

**24. HTTP headers? Name 10.**
`Content-Type`, `Authorization`, `Accept`, `Cache-Control`, `Host`, `User-Agent`, `Cookie`, `Set-Cookie`, `X-Forwarded-For`, `Content-Length`, `Location`, `Access-Control-Allow-Origin`

**25. HTTP/1.1 vs HTTP/2 vs HTTP/3?**
- **HTTP/1.1**: Text-based, one request per connection (head-of-line blocking), keep-alive.
- **HTTP/2**: Binary, multiplexing (multiple requests over one connection), header compression, server push.
- **HTTP/3**: Uses QUIC (UDP-based), faster connection setup, better mobile performance.

**26. REST API? Principles?**
Architectural style for APIs:
1. **Stateless**: No server-side session
2. **Resource-based**: URLs represent resources (`/users/123`)
3. **HTTP methods**: GET/POST/PUT/DELETE for CRUD
4. **Representations**: JSON/XML for data
5. **HATEOAS**: Links in responses for navigation
6. **Uniform interface**: Consistent patterns

**27. Webhook vs polling?**
- **Webhook**: Server pushes notification when event occurs (event-driven, efficient)
- **Polling**: Client repeatedly asks server for updates (simple, wasteful)
Webhook = doorbell (notified when visitor). Polling = looking out window every minute.

---

## TLS/SSL

**28. TLS vs SSL?**
- **SSL**: Secure Sockets Layer. Deprecated (1.0-3.0).
- **TLS**: Transport Layer Security. Current standard (1.2, 1.3). **Always use TLS 1.2+.**

**29. TLS handshake?**

```
TLS 1.2 Handshake:

  Client                                     Server
    │                                           │
    │──── ClientHello ──────────────────────────▶│
    │     (supported ciphers, TLS version,       │
    │      client random)                        │
    │                                           │
    │◀─── ServerHello ──────────────────────────│
    │     (chosen cipher, server random,         │
    │      certificate + public key)             │
    │                                           │
    │     [Client verifies certificate           │
    │      against trusted CAs]                  │
    │                                           │
    │──── Pre-master secret ───────────────────▶│
    │     (encrypted with server's public key)   │
    │                                           │
    │     [Both derive session keys from:        │
    │      client random + server random +       │
    │      pre-master secret]                    │
    │                                           │
    │──── Finished (encrypted) ────────────────▶│
    │◀─── Finished (encrypted) ────────────────│
    │                                           │
    │◀═══ Symmetric encrypted data ═══════════▶│
    │     (AES-256-GCM, fast!)                  │

TLS 1.3: Only 1 round-trip (faster!)
  ClientHello includes key share → server responds with
  key share + encrypted data immediately
```

1. Client sends "Hello" with supported ciphers
2. Server responds with chosen cipher + certificate
3. Client verifies certificate against trusted CAs
4. Client sends pre-master secret (encrypted with server's public key)
5. Both derive session keys from pre-master secret
6. Symmetric encryption begins
TLS 1.3 reduces this to 1 round-trip.

**30. Certificate? CA?**
- **Certificate**: Contains public key + domain name + issuer info. Proves server identity.
- **CA (Certificate Authority)**: Trusted entity that signs certificates. (Let's Encrypt, DigiCert, Comodo)

**31. Self-signed certificate?**
Certificate signed by yourself (not a CA). Browser shows warning. Use for: development, internal tools, testing. Never for production.

**32. mTLS (mutual TLS)?**
Both client AND server present certificates. Server verifies client identity too. Used for: service-to-service communication (Istio service mesh), zero-trust architectures.

**33. Let's Encrypt?**
Free, automated CA. Uses ACME protocol. Certificates valid 90 days (auto-renew). Use cert-manager in K8s for automation.

**34. Troubleshoot TLS issues?**
```bash
# Check certificate
openssl s_client -connect example.com:443
curl -vI https://example.com            # Verbose output shows TLS details

# Check expiry
echo | openssl s_client -connect host:443 2>/dev/null | openssl x509 -noout -dates

# Common issues: expired cert, wrong hostname, incomplete chain, self-signed
```

---

## Load Balancing & Proxies

**35. Load balancer? Why needed?**
Distributes traffic across multiple servers. Benefits: high availability (if one server dies), scalability (add more servers), performance (no single bottleneck).

**36. Load balancing algorithms?**
- **Round-robin**: Each server in turn. Simple.
- **Least connections**: Route to server with fewest active connections.
- **IP hash**: Same client IP always goes to same server.
- **Weighted**: More traffic to more powerful servers.
- **Random**: Random selection.

**37. Reverse proxy?**
Server that sits in front of backend servers. Forwards client requests to appropriate backend. Examples: Nginx, HAProxy, Envoy. Benefits: SSL termination, caching, compression, security.

**38. Load balancer vs reverse proxy?**
Often the same tool. Load balancer focuses on distributing traffic. Reverse proxy adds: SSL termination, caching, URL rewriting, security. Nginx does both.

**39. Health checking?**
LB periodically checks if backend servers are healthy (HTTP GET /health). Unhealthy servers removed from pool until they recover.

**40. Session persistence/sticky sessions?**
Same client always sent to same backend server. Used when: session data stored in memory (not recommended). Better: externalize sessions (Redis).

**41. SSL termination?**
LB/proxy handles TLS encryption/decryption. Backend servers receive plain HTTP. Benefits: offloads CPU from backends, centralized cert management.

---

## Firewalls & Security

**42. Firewall? Types?**
Controls network traffic based on rules.
- **Network firewall**: Between networks (hardware/virtual)
- **Host-based firewall**: On individual server (iptables, ufw, firewalld)
- **WAF (Web Application Firewall)**: L7, inspects HTTP content (SQL injection, XSS)

**43. WAF vs regular firewall?**
- **Regular firewall**: Filters by IP, port, protocol (L3/L4)
- **WAF**: Inspects HTTP payload (L7). Blocks: SQL injection, XSS, CSRF, malicious bots. Examples: AWS WAF, Cloudflare, ModSecurity.

**44. DDoS? Protection?**
Distributed Denial of Service — overwhelming server with traffic.
Protection: CDN (Cloudflare, AWS CloudFront), rate limiting, WAF, auto-scaling, traffic scrubbing, geo-blocking.

**45. ACL?**
Access Control List — ordered list of permit/deny rules for network traffic. Applied to router/firewall interfaces. Example: "Deny all traffic from 10.0.0.0/8 to port 22."

---

## Interview-Style

**46. User can't reach app — network debugging?**
1. `ping server` → Is the server reachable at network level?
2. `traceroute server` → Where does the path break?
3. `curl -v http://server:port` → Can we reach the service?
4. `dig server` → Is DNS resolving correctly?
5. Check firewall/security groups → Is the port open?
6. Check load balancer health → Are backends healthy?
7. `ss -tlnp` on server → Is the app actually listening?
8. Check application logs → Is the app returning errors?

**47. Request from browser to K8s app?**

```
Full Request Path:

  Browser                  Cloud LB             Ingress Controller
  ┌───────────┐          ┌───────────┐       ┌─────────────┐
  │ 1. DNS     │────────▶│ 2. TLS     │─────▶│ 3. Route     │
  │    resolve │ HTTPS   │    terminate│  HTTP │    by host/  │
  │           │          │            │       │    path      │
  └───────────┘          └───────────┘       └──────┬──────┘
                                                     │
          Service (ClusterIP)      Pod                │
          ┌─────────────┐      ┌──────────┐     │
          │ 4. kube-proxy │────▶│ 5. App    │◀───┘
          │    selects   │ iptab│    handles│
          │    pod IP    │ /IPVS│    request│
          └─────────────┘      └──────────┘

  Response flows back the same path in reverse
```

1. User types URL → browser resolves DNS → gets LoadBalancer IP
2. HTTPS connection to cloud load balancer
3. LB routes to Ingress Controller pod (Nginx)
4. Ingress matches hostname/path rules → routes to Service
5. kube-proxy (iptables/IPVS) selects a pod IP
6. Request reaches pod → container processes request
7. Response travels back the same path

**48. TLS for K8s Ingress?**
Use cert-manager + Let's Encrypt:
1. Install cert-manager
2. Create ClusterIssuer for Let's Encrypt
3. Add `tls` section and `cert-manager.io/cluster-issuer` annotation to Ingress
4. cert-manager auto-provisions and renews certificates

**49. DNS change not propagating?**
1. Check TTL of old record (may need to wait for cache expiry)
2. Check with `dig @8.8.8.8 domain` (bypass local cache)
3. Check with multiple DNS servers
4. Verify change was applied at registrar/DNS provider
5. Check if propagation tool (whatsmydns.net) shows update
6. Flush local DNS cache: `systemd-resolve --flush-caches`

**50. TCP vs UDP? When use each?**

```
TCP 3-Way Handshake:              UDP (no handshake):

  Client         Server            Client         Server
    │               │                │               │
    │── SYN ────▶│                │── data ───▶│
    │               │                │               │  (fire and
    │◀─ SYN+ACK ──│                │── data ───▶│   forget!)
    │               │                │               │
    │── ACK ────▶│                │── data ───▶│
    │               │                │               │
    │◀══ data ═══▶│                No guarantee of delivery,
    │  (reliable)   │                no ordering, no retransmit
```

| TCP | UDP |
|---|---|
| Reliable, ordered | Unreliable, unordered |
| Connection-oriented (3-way handshake) | Connectionless |
| Retransmission on loss | No retransmission |
| Slower | Faster |
| HTTP, SSH, FTP, SMTP, databases | DNS, DHCP, video streaming, gaming, VoIP |

**51. Highly available network architecture?**
- Multiple AZs/regions
- Redundant load balancers (active-active or active-passive)
- Auto-scaling groups
- DNS failover (Route 53 health checks)
- CDN for static content
- Database replication (read replicas, multi-AZ)
- No single points of failure

**52. CDN? When use?**
Content Delivery Network — caches content at edge locations worldwide. Use for: static assets (images, CSS, JS), video streaming, API acceleration. Benefits: lower latency, reduced origin load, DDoS protection. Examples: CloudFront, Cloudflare, Akamai.

**53. Service timing out intermittently — diagnose?**
1. Check if it correlates with load spikes (`top`, `vmstat`)
2. Check network latency: `mtr target`
3. Check connection pool exhaustion (database, HTTP client)
4. Check packet loss: `ping -c 100 host | tail`
5. `tcpdump` to capture actual traffic during timeout
6. Check DNS resolution time: `dig host | grep time`
7. Check if it's specific to one backend (load balancer logs)
8. Check resource limits (CPU, memory, file descriptors)

**54. VPN? Types?**
Virtual Private Network — encrypted tunnel between networks.
- **Site-to-site**: Connects two networks (office ↔ cloud). Always on.
- **Point-to-site**: Individual device connects to network (remote worker → office). On-demand.
- **SSL VPN**: Browser-based VPN access.

**55. Bastion host / jump box?**
Hardened server in public subnet used as gateway to private resources. SSH to bastion, then SSH to internal servers. Security: only bastion has public IP, all access is audited, reduces attack surface.
```bash
ssh -J bastion-user@bastion internal-user@internal-server
# Or ProxyJump in SSH config
```

---
---

# PART 4: ADVANCED NETWORKING — Service Mesh, DNS, Troubleshooting, Cloud Networking

---

## Service Mesh (Istio / Linkerd)

**56. What is a Service Mesh? Why use one?**

```
Service Mesh Architecture:

  WITHOUT Service Mesh:
  ┌──────────┐         ┌──────────┐
  │ Service A │────────▶│ Service B │   Each service handles:
  │ (app code │         │ (app code │   - retries
  │  + retry  │         │  + retry  │   - timeouts
  │  + TLS    │         │  + TLS    │   - auth
  │  + metrics│         │  + metrics│   - tracing
  │  + auth)  │         │  + auth)  │   ALL IN APP CODE!
  └──────────┘         └──────────┘

  WITH Service Mesh (Istio):
  ┌───────────────────┐         ┌───────────────────┐
  │ Pod               │         │ Pod               │
  │ ┌───────┐ ┌─────┐│         │┌─────┐ ┌───────┐ │
  │ │Service│ │Envoy││←───────→││Envoy│ │Service│ │
  │ │   A   │ │proxy││  mTLS   ││proxy│ │   B   │ │
  │ │(clean │ │     ││         ││     │ │(clean │ │
  │ │ code) │ │     ││         ││     │ │ code) │ │
  │ └───────┘ └─────┘│         │└─────┘ └───────┘ │
  └───────────────────┘         └───────────────────┘
         Sidecar handles: retries, TLS, auth, metrics, tracing
         App code stays CLEAN — only business logic

  Control Plane (istiod):
  ┌────────────────────────────────────────┐
  │  Config → distribute to all proxies    │
  │  Certificates → mTLS between services │
  │  Service Discovery → where is what    │
  └────────────────────────────────────────┘
```

```
Service Mesh Features:

  ┌────────────────────┬───────────────────────────────────┐
  │ Feature            │ What it does                      │
  ├────────────────────┼───────────────────────────────────┤
  │ mTLS               │ Encrypt ALL service-to-service    │
  │                    │ traffic automatically             │
  ├────────────────────┼───────────────────────────────────┤
  │ Traffic Management │ Canary deploys, A/B testing,      │
  │                    │ traffic splitting, mirroring      │
  ├────────────────────┼───────────────────────────────────┤
  │ Retries/Timeouts   │ Automatic retry with backoff,     │
  │                    │ circuit breaker                   │
  ├────────────────────┼───────────────────────────────────┤
  │ Observability      │ Distributed tracing, metrics,     │
  │                    │ access logs — zero code changes   │
  ├────────────────────┼───────────────────────────────────┤
  │ Authorization      │ Policy: service A can call B      │
  │                    │ but not C                         │
  ├────────────────────┼───────────────────────────────────┤
  │ Rate Limiting      │ Limit requests per service        │
  └────────────────────┴───────────────────────────────────┘

  Istio vs Linkerd:
  ├── Istio: Feature-rich, complex, Envoy proxy, more config
  └── Linkerd: Simpler, lighter, Rust proxy, easier to operate
```

---

## DNS Deep Dive

**57. DNS resolution flow — what happens when you type a URL?**

```
Full DNS Resolution:

  Browser: "api.example.com"
      │
      ▼
  1. Browser cache (check first)
      │ miss
      ▼
  2. OS cache (/etc/hosts, systemd-resolved)
      │ miss
      ▼
  3. Recursive resolver (ISP or 8.8.8.8/1.1.1.1)
      │ miss
      ▼
  4. Root nameserver (.) → "go ask .com"
      │
      ▼
  5. TLD nameserver (.com) → "go ask ns1.example.com"
      │
      ▼
  6. Authoritative nameserver (example.com)
      → "api.example.com = 93.184.216.34"
      │
      ▼
  7. Response cached at each level (TTL-based)
      │
      ▼
  8. Browser connects to 93.184.216.34

  Record Types:
  A      → Name → IPv4 (api.example.com → 93.184.216.34)
  AAAA   → Name → IPv6
  CNAME  → Name → Another name (alias)
  MX     → Mail exchange servers
  NS     → Nameservers for domain
  TXT    → Text records (SPF, DKIM, verification)
  SRV    → Service location (port + host)
  PTR    → Reverse DNS (IP → name)
```

```bash
# DNS troubleshooting commands
dig api.example.com             # Full query
dig +short api.example.com      # Just the IP
dig @8.8.8.8 api.example.com   # Query specific resolver
dig api.example.com MX          # Mail records
dig +trace api.example.com     # Show full resolution chain
nslookup api.example.com        # Simple lookup
host api.example.com            # Another simple lookup

# Kubernetes DNS
# Service: <svc>.<namespace>.svc.cluster.local
# Pod: <pod-ip-dashed>.<namespace>.pod.cluster.local
kubectl run test --image=busybox --rm -it -- nslookup api-service.production.svc.cluster.local
```

---

## Network Troubleshooting Toolkit

**58. Essential networking commands for interviews:**

```bash
# ─── CONNECTIVITY ─────────────────────────────────
ping 10.0.0.1                    # ICMP connectivity
traceroute 10.0.0.1              # Path to host (shows each hop)
mtr 10.0.0.1                     # Continuous traceroute (best tool)
telnet 10.0.0.1 80               # Test TCP port connectivity
nc -zv 10.0.0.1 80               # Netcat port check
curl -v http://10.0.0.1:80       # HTTP connectivity + headers

# ─── DNS ──────────────────────────────────────────
dig +short api.example.com       # Resolve DNS
nslookup api.example.com         # DNS lookup
cat /etc/resolv.conf             # DNS resolver config

# ─── PORTS & CONNECTIONS ──────────────────────────
ss -tlnp                          # Listening TCP ports (modern)
netstat -tlnp                     # Listening ports (older)
ss -s                             # Connection statistics
lsof -i :8080                    # What process is using port 8080

# ─── TRAFFIC CAPTURE ─────────────────────────────
tcpdump -i eth0 port 80          # Capture HTTP traffic
tcpdump -i any host 10.0.0.1    # All traffic to/from host
tcpdump -w capture.pcap          # Save to file (analyze in Wireshark)

# ─── ROUTING ──────────────────────────────────────
ip route show                    # Routing table
ip route get 10.0.0.1           # How would we reach this IP?
ip addr show                     # All interfaces + IPs
ip neigh show                    # ARP table (MAC addresses)

# ─── BANDWIDTH / PERFORMANCE ─────────────────────
iperf3 -s                        # Start server
iperf3 -c 10.0.0.1              # Test bandwidth to server
curl -o /dev/null -w "time_total: %{time_total}\n" http://api.example.com
```

**59. TCP 3-way handshake and connection states:**

```
TCP 3-Way Handshake:

  Client                    Server
    │                         │
    │──── SYN ───────────────▶│  "I want to connect"
    │                         │
    │◀─── SYN-ACK ───────────│  "OK, I acknowledge"
    │                         │
    │──── ACK ───────────────▶│  "Great, connection open"
    │                         │
    │◀═══ DATA ══════════════▶│  Data transfer
    │                         │
    │──── FIN ───────────────▶│  "I'm done"
    │◀─── ACK ───────────────│
    │◀─── FIN ───────────────│
    │──── ACK ───────────────▶│  Connection closed

  Common states (visible in ss/netstat):
  LISTEN      → Server waiting for connections
  ESTABLISHED → Active connection
  TIME_WAIT   → Connection closing, waiting for late packets
  CLOSE_WAIT  → Remote side closed, app hasn't closed yet (BUG if many)
  SYN_SENT    → Connection attempt in progress
```

---

## Cloud Networking Concepts

**60. VPC/VNET, Subnets, Security Groups, NAT Gateway:**

```
Cloud Network Architecture:

  ┌──── VPC / VNET (10.0.0.0/16) ─────────────────────────────────┐
  │                                                                 │
  │  ┌── Public Subnet (10.0.1.0/24) ──────────────────────────┐  │
  │  │  ┌──────────────┐  ┌──────────────┐                     │  │
  │  │  │ Load Balancer │  │ Bastion Host │                     │  │
  │  │  │ (public IP)   │  │ (public IP)  │                     │  │
  │  │  └──────┬───────┘  └──────────────┘                     │  │
  │  │         │                                                │  │
  │  │   Internet Gateway (IGW) ←→ Internet                     │  │
  │  └─────────┼────────────────────────────────────────────────┘  │
  │            │                                                    │
  │  ┌── Private Subnet (10.0.2.0/24) ─────────────────────────┐  │
  │  │         │                                                │  │
  │  │  ┌──────▼───────┐  ┌──────────────┐  ┌──────────────┐  │  │
  │  │  │ App Server 1 │  │ App Server 2 │  │ App Server 3 │  │  │
  │  │  │ (no public IP)│  │              │  │              │  │  │
  │  │  └──────────────┘  └──────────────┘  └──────────────┘  │  │
  │  │                                                          │  │
  │  │  NAT Gateway → lets private servers reach internet       │  │
  │  │               (outbound only, no inbound)                │  │
  │  └──────────────────────────────────────────────────────────┘  │
  │                                                                 │
  │  ┌── Database Subnet (10.0.3.0/24) ────────────────────────┐  │
  │  │  ┌──────────────┐  ┌──────────────┐                     │  │
  │  │  │  Primary DB  │  │  Replica DB  │  No internet access │  │
  │  │  └──────────────┘  └──────────────┘                     │  │
  │  └──────────────────────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────────────────────┘

  Security Groups (stateful firewall):
  ├── App SG: Inbound from LB on port 8080 only
  ├── DB SG: Inbound from App SG on port 5432 only
  └── Bastion SG: Inbound SSH from your IP only

  NACLs (stateless firewall):
  ├── Subnet-level rules
  ├── Both inbound AND outbound rules needed
  └── Evaluated in order (rule numbers)
```
