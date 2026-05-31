> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Deep-dive learning guide (this file) |
| [complete.md](complete.md) | Complete question bank |
| [answers.md](answers.md) | All answers |

---

# Networking — Deep-Dive Learning Guide

---

## 1. OSI Model & TCP/IP

```
┌─── OSI Model ───────────────┬─── TCP/IP Model ────────────┐
│                              │                              │
│  7. Application (HTTP, DNS)  │  Application                 │
│  6. Presentation (SSL/TLS)   │  (HTTP, DNS, FTP, SSH, SMTP)│
│  5. Session                  │                              │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤                              │
│  4. Transport (TCP, UDP)     │  Transport (TCP, UDP)        │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤                              │
│  3. Network (IP, ICMP)       │  Internet (IP, ICMP, ARP)   │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤                              │
│  2. Data Link (Ethernet, MAC)│  Network Access              │
│  1. Physical (cables, signals)│ (Ethernet, Wi-Fi)           │
└──────────────────────────────┴──────────────────────────────┘
```

### What Happens When You Type `https://example.com`

```
Step 1: DNS Resolution
  Browser → DNS resolver → Root NS → .com NS → example.com NS
  Returns: 93.184.216.34

Step 2: TCP Three-Way Handshake
  Client ──► SYN ──────────────► Server
  Client ◄── SYN-ACK ◄────────── Server
  Client ──► ACK ──────────────► Server
  Connection established!

Step 3: TLS Handshake (HTTPS)
  Client ──► ClientHello (supported ciphers) ──► Server
  Client ◄── ServerHello + Certificate ◄──────── Server
  Client verifies certificate (trust chain)
  Client ──► Key exchange ──► Server
  Both derive session keys
  Encrypted channel ready!

Step 4: HTTP Request
  GET / HTTP/1.1
  Host: example.com

Step 5: Server responds
  HTTP/1.1 200 OK
  Content-Type: text/html
  (page content)

Step 6: TCP Four-Way Teardown
  Client ──► FIN ──► Server
  Client ◄── ACK ◄── Server
  Client ◄── FIN ◄── Server
  Client ──► ACK ──► Server
```

---

## 2. TCP vs UDP

```
┌─── TCP (Transmission Control Protocol) ────────────────────┐
│  Connection-oriented (3-way handshake first)                │
│  Reliable (ACKs, retransmission, ordering)                  │
│  Flow control (sliding window)                              │
│  Slower but guaranteed delivery                             │
│                                                              │
│  Used by: HTTP, HTTPS, SSH, FTP, SMTP, databases           │
│  "I need every byte to arrive in order"                    │
└──────────────────────────────────────────────────────────────┘

┌─── UDP (User Datagram Protocol) ───────────────────────────┐
│  Connectionless (just send packets)                         │
│  Unreliable (no ACKs, no retransmission)                   │
│  No ordering guarantee                                      │
│  Fast, low overhead                                         │
│                                                              │
│  Used by: DNS, DHCP, VoIP, video streaming, gaming         │
│  "Speed matters more than losing a few packets"            │
└──────────────────────────────────────────────────────────────┘
```

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Required (handshake) | None |
| Reliability | Guaranteed (ACKs) | Best-effort |
| Ordering | Yes | No |
| Speed | Slower | Faster |
| Header size | 20 bytes | 8 bytes |
| Use case | Web, email, file transfer | DNS, streaming, gaming |

---

## 3. IP Addressing & Subnetting

### IPv4 Address

```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0    or /24 (CIDR notation)

  192.168.1.100 = 11000000.10101000.00000001.01100100
  255.255.255.0 = 11111111.11111111.11111111.00000000
                  ├── Network (24 bits) ──┤├ Host(8)┤

  Network:   192.168.1.0     (first address)
  Broadcast: 192.168.1.255   (last address)
  Usable:    192.168.1.1 - 192.168.1.254  (254 hosts)
```

### CIDR Cheat Sheet

```
/32  = 1 IP        (single host)
/31  = 2 IPs       (point-to-point link)
/30  = 4 IPs       (2 usable — smallest subnet)
/28  = 16 IPs      (14 usable)
/24  = 256 IPs     (254 usable — "Class C")
/16  = 65,536 IPs  (65,534 usable — "Class B")
/8   = 16.7M IPs   ("Class A")
```

### Private IP Ranges (RFC 1918)

```
10.0.0.0/8          10.0.0.0 - 10.255.255.255      (huge — cloud VNets)
172.16.0.0/12       172.16.0.0 - 172.31.255.255     (medium)
192.168.0.0/16      192.168.0.0 - 192.168.255.255   (home/small networks)

These are NOT routable on the internet.
NAT translates private → public IP for internet access.
```

---

## 4. DNS (Domain Name System)

```
Browser: "What's the IP of app.example.com?"

┌──────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│Client│────►│ Recursive│────►│Root (.com)│────►│Authority.│
│      │     │ Resolver │     │ NS       │     │ NS       │
│      │     │ (ISP or  │     │          │     │(example  │
│      │     │  8.8.8.8)│     │          │     │ .com)    │
│      │◄────│          │◄────│          │◄────│          │
│      │     │ Caches!  │     │          │     │ Returns: │
│      │     │          │     │          │     │93.184.216│
└──────┘     └──────────┘     └──────────┘     └──────────┘
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | Domain → IPv4 | `example.com → 93.184.216.34` |
| **AAAA** | Domain → IPv6 | `example.com → 2606:2800:220:1:...` |
| **CNAME** | Alias to another domain | `www.example.com → example.com` |
| **MX** | Mail server | `example.com → mail.example.com` |
| **NS** | Nameserver delegation | `example.com → ns1.provider.com` |
| **TXT** | Text record (SPF, DKIM, verification) | `v=spf1 include:_spf.google.com` |
| **SRV** | Service discovery (port + host) | `_http._tcp.example.com` |
| **PTR** | Reverse DNS (IP → domain) | `34.216.184.93 → example.com` |

```bash
# DNS lookups
nslookup example.com
dig example.com A              # Detailed A record query
dig example.com MX             # Mail server
dig +short example.com         # Just the IP
host example.com               # Simple lookup
```

---

## 5. HTTP/HTTPS

### HTTP Methods

| Method | Purpose | Idempotent? | Body? |
|--------|---------|-------------|-------|
| GET | Read resource | Yes | No |
| POST | Create resource | No | Yes |
| PUT | Replace resource | Yes | Yes |
| PATCH | Partial update | No | Yes |
| DELETE | Remove resource | Yes | No |
| HEAD | GET without body | Yes | No |
| OPTIONS | Supported methods | Yes | No |

### HTTP Status Codes

```
1xx — Informational
2xx — Success
  200 OK               — request succeeded
  201 Created           — resource created (POST)
  204 No Content        — success, no body (DELETE)
3xx — Redirection
  301 Moved Permanently — use new URL forever
  302 Found             — temporary redirect
  304 Not Modified      — use cached version
4xx — Client Error
  400 Bad Request       — invalid input
  401 Unauthorized      — not authenticated
  403 Forbidden         — authenticated but not allowed
  404 Not Found         — resource doesn't exist
  429 Too Many Requests — rate limited
5xx — Server Error
  500 Internal Server Error — generic server failure
  502 Bad Gateway       — upstream server error
  503 Service Unavailable — server overloaded/maintenance
  504 Gateway Timeout   — upstream didn't respond in time
```

### HTTPS & TLS

```
HTTPS = HTTP + TLS (Transport Layer Security)

TLS provides:
  1. Encryption    — data can't be read in transit
  2. Authentication — server proves identity via certificate
  3. Integrity     — data can't be tampered with

Certificate Chain:
  Root CA (trusted by browsers)
    └── Intermediate CA
        └── Server Certificate (your domain)

Let's Encrypt: Free, automated TLS certificates (90-day renewal)
```

---

## 6. Load Balancing

```
┌─── Load Balancer ──────────────────────────────────────────┐
│                                                             │
│  Client ──► Load Balancer ──► Backend Servers               │
│                  │                                          │
│          ┌───────┼───────┐                                 │
│          │       │       │                                 │
│       Server1 Server2 Server3                              │
│                                                             │
│  L4 (Transport): Routes by IP:port (fast, no content aware)│
│  L7 (Application): Routes by URL, headers, cookies (smart) │
└─────────────────────────────────────────────────────────────┘
```

### Algorithms

| Algorithm | Description | Use Case |
|-----------|------------|----------|
| Round Robin | Sequential rotation | Equal servers |
| Weighted Round Robin | More traffic to stronger servers | Mixed hardware |
| Least Connections | Route to server with fewest connections | Varying request times |
| IP Hash | Same client IP → same server | Session stickiness |
| URL Hash | Same URL → same server | Caching |

### L4 vs L7

| Aspect | L4 (TCP/UDP) | L7 (HTTP/HTTPS) |
|--------|-------------|-----------------|
| Layer | Transport | Application |
| Speed | Faster | Slower (inspects content) |
| SSL termination | No | Yes |
| Content routing | No | Yes (URL, headers, cookies) |
| Examples | AWS NLB, Azure LB | AWS ALB, nginx, HAProxy, Azure App GW |

---

## 7. Firewalls & Security Groups

```
┌─── Firewall Types ─────────────────────────────────────────┐
│                                                             │
│  Stateless: Checks each packet independently               │
│    (AWS NACL, basic iptables)                              │
│    Must allow BOTH inbound AND outbound rules              │
│                                                             │
│  Stateful: Tracks connections                               │
│    (AWS Security Group, Azure NSG, iptables with conntrack)│
│    Allow inbound → return traffic automatically allowed    │
│                                                             │
│  WAF (Web Application Firewall): L7                        │
│    Inspects HTTP content                                    │
│    Blocks: SQL injection, XSS, bot traffic                 │
│    Tools: AWS WAF, Azure WAF, Cloudflare, ModSecurity      │
└─────────────────────────────────────────────────────────────┘
```

### iptables Basics

```bash
# Default policy
iptables -P INPUT DROP         # Drop everything by default
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT      # Allow outbound

# Allow specific
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # HTTPS
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  # Return traffic

# List rules
iptables -L -n -v
```

---

## 8. NAT, VPN, Proxy

### NAT (Network Address Translation)

```
Private Network                    Internet
192.168.1.0/24
                     ┌─────────┐
[192.168.1.5] ─────►│  NAT    │──► [203.0.113.1] ──► Internet
[192.168.1.6] ─────►│  Router │──► [203.0.113.1]
[192.168.1.7] ─────►│         │──► [203.0.113.1]
                     └─────────┘
                     Many private IPs → one public IP
                     Tracks via port numbers (PAT)
```

### VPN (Virtual Private Network)

```
┌─── Site-to-Site VPN ─────────────────────────────────────┐
│                                                           │
│  Office Network ◄═══ Encrypted Tunnel ═══► Cloud VNet    │
│  10.0.0.0/16                                10.1.0.0/16  │
│                                                           │
│  Devices on both networks can communicate as if local    │
└───────────────────────────────────────────────────────────┘

┌─── Client VPN ───────────────────────────────────────────┐
│                                                           │
│  Laptop ◄═══ Encrypted Tunnel ═══► Corporate Network     │
│  (remote worker)                                          │
│  Gets an IP on the corporate network                     │
└───────────────────────────────────────────────────────────┘
```

### Forward vs Reverse Proxy

```
Forward Proxy (client-side):
  Client ──► Proxy ──► Internet
  Client hides behind proxy
  Use: content filtering, caching, bypass geo-blocks

Reverse Proxy (server-side):
  Internet ──► Proxy ──► Backend Servers
  Servers hide behind proxy
  Use: load balancing, SSL termination, caching, WAF
  Tools: nginx, HAProxy, Traefik, Envoy
```

---

## 9. Container Networking

### Docker Networking

```
Default Bridge:
  Container A (172.17.0.2) ──┐
  Container B (172.17.0.3) ──┤── docker0 bridge ── NAT ── Host
  No DNS resolution between containers!

User-Defined Bridge:
  Container A ──┐
  Container B ──┤── mynet bridge ── NAT ── Host
  DNS resolution by container name! ✅
```

### Kubernetes Networking

```
K8s Network Model:
  1. Every Pod gets its own IP
  2. All Pods can communicate without NAT
  3. Flat network — no port mapping needed

  Pod A (10.244.1.5) ──── CNI Plugin ──── Pod B (10.244.2.3)
        (Node 1)          (Calico/Cilium)       (Node 2)
                          VXLAN/BGP overlay

  Service (10.96.0.10) = stable IP for group of Pods
    kube-proxy creates iptables/IPVS rules to route traffic
```

---

## 10. Network Troubleshooting Commands

```bash
# ─── Connectivity ───
ping -c 4 host                # ICMP echo (is host reachable?)
traceroute host               # Path packets take (where do they stop?)
mtr host                      # Combined ping + traceroute (live)
curl -I https://host          # HTTP headers (is web server running?)
wget https://host/file        # Download file

# ─── DNS ───
nslookup domain               # Basic DNS lookup
dig domain                    # Detailed DNS query
dig +trace domain             # Full DNS resolution path
cat /etc/resolv.conf          # DNS server config

# ─── Ports & Connections ───
ss -tulnp                     # All listening ports with processes
netstat -tulnp                # Legacy equivalent
lsof -i :8080                 # What process is using port 8080
nc -zv host 443               # Test if port is open
telnet host 80                # Test TCP connection

# ─── Network Config ───
ip addr show                  # Interfaces and IPs
ip route show                 # Routing table
ip neigh show                 # ARP table (IP → MAC)

# ─── Packet Capture ───
tcpdump -i eth0 port 80       # Capture HTTP traffic
tcpdump -i any host 10.0.1.5  # Traffic to/from specific host
tcpdump -i eth0 -w capture.pcap  # Save to file (open in Wireshark)

# ─── Bandwidth ───
iftop                         # Live bandwidth by connection
nethogs                       # Bandwidth per process
```

---

## 11. Cloud Networking Concepts

```
┌─── VNet/VPC ────────────────────────────────────────────────┐
│  Virtual Private Cloud — your isolated network in the cloud │
│                                                              │
│  ┌──── Subnet: Public (10.0.1.0/24) ─────────────────┐    │
│  │  Has Internet Gateway (route to 0.0.0.0/0)         │    │
│  │  ┌──────┐  ┌──────┐                                │    │
│  │  │ LB   │  │ NAT  │                                │    │
│  │  │      │  │ GW   │                                │    │
│  │  └──────┘  └──────┘                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──── Subnet: Private (10.0.2.0/24) ────────────────┐    │
│  │  No direct internet access (goes through NAT GW)   │    │
│  │  ┌──────┐  ┌──────┐  ┌──────┐                     │    │
│  │  │App VM│  │App VM│  │App VM│                     │    │
│  │  └──────┘  └──────┘  └──────┘                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──── Subnet: Data (10.0.3.0/24) ───────────────────┐    │
│  │  No internet access at all                          │    │
│  │  ┌──────┐  ┌──────┐                                │    │
│  │  │  DB  │  │Cache │                                │    │
│  │  └──────┘  └──────┘                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  NSG rules control traffic between subnets                  │
│  Route tables define how traffic flows                      │
│  VNet Peering connects VNets (no internet traversal)        │
└──────────────────────────────────────────────────────────────┘
```
