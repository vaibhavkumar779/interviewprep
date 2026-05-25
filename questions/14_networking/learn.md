# Networking - LEARNING MATERIAL

---

## OSI Model

```mermaid
graph TD
    L7[Layer 7: Application<br/>HTTP, DNS, SSH, SMTP] --> L6[Layer 6: Presentation<br/>TLS/SSL, Encryption]
    L6 --> L5[Layer 5: Session<br/>Session management]
    L5 --> L4[Layer 4: Transport<br/>TCP, UDP - Ports]
    L4 --> L3[Layer 3: Network<br/>IP, ICMP - Routing]
    L3 --> L2[Layer 2: Data Link<br/>Ethernet, MAC - Switching]
    L2 --> L1[Layer 1: Physical<br/>Cables, Signals]
```

## TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery, ordering | Best effort |
| Speed | Slower (overhead) | Faster |
| Use cases | HTTP, SSH, FTP, DB | DNS, streaming, VoIP, gaming |

## DNS Resolution

```mermaid
graph LR
    USER[Browser: app.example.com] --> LOCAL[Local DNS Cache]
    LOCAL -->|Miss| RESOLVER[DNS Resolver<br/>ISP / 8.8.8.8]
    RESOLVER -->|Miss| ROOT[Root DNS<br/>. → .com]
    ROOT --> TLD[TLD DNS<br/>.com → example.com]
    TLD --> AUTH[Authoritative DNS<br/>example.com → 93.184.216.34]
    AUTH -->|A Record| RESOLVER
    RESOLVER --> USER
```

### DNS Record Types

| Record | Purpose | Example |
|---|---|---|
| **A** | Domain → IPv4 | `app.example.com → 93.184.216.34` |
| **AAAA** | Domain → IPv6 | `app.example.com → 2606:2800:...` |
| **CNAME** | Alias → another domain | `www.example.com → app.example.com` |
| **MX** | Mail server | `example.com → mail.example.com` |
| **NS** | Nameserver | `example.com → ns1.example.com` |
| **TXT** | Arbitrary text | SPF, DKIM, domain verification |
| **SRV** | Service discovery | `_http._tcp.example.com` |

## HTTP/HTTPS

```mermaid
sequenceDiagram
    Client->>Server: TCP 3-way handshake (SYN, SYN-ACK, ACK)
    Client->>Server: TLS Handshake (if HTTPS)
    Client->>Server: GET /api/users HTTP/1.1
    Server->>Client: 200 OK + JSON body
    Client->>Server: POST /api/users (with body)
    Server->>Client: 201 Created
```

### HTTP Status Codes

| Code | Meaning | Common Scenario |
|---|---|---|
| **200** | OK | Successful GET |
| **201** | Created | Successful POST |
| **301** | Moved Permanently | URL redirect (cached) |
| **302** | Found | Temporary redirect |
| **400** | Bad Request | Invalid input |
| **401** | Unauthorized | Missing/invalid auth |
| **403** | Forbidden | No permission |
| **404** | Not Found | Wrong URL |
| **500** | Internal Server Error | Server bug |
| **502** | Bad Gateway | Upstream server down |
| **503** | Service Unavailable | Server overloaded |
| **504** | Gateway Timeout | Upstream timed out |

## Essential Network Commands

```bash
# DNS lookup
nslookup example.com
dig example.com A
dig +short example.com

# Connectivity
ping -c 4 example.com              # ICMP ping
traceroute example.com              # Route path (tracert on Windows)
curl -v https://api.example.com     # HTTP request with details
curl -I https://example.com         # Headers only

# Ports and connections
ss -tlnp                            # Listening ports (Linux)
netstat -an | grep LISTEN           # Listening ports
nc -zv host 80                      # Test port connectivity

# Network config
ip addr show                        # Show IP addresses
ip route show                       # Show routing table
cat /etc/resolv.conf                # DNS config
```

## Load Balancing

```mermaid
graph TD
    CLIENT[Client] --> LB[Load Balancer]
    LB -->|Round Robin| S1[Server 1]
    LB -->|Least Connections| S2[Server 2]
    LB -->|IP Hash| S3[Server 3]
```

| Algorithm | How | Use Case |
|---|---|---|
| Round Robin | Sequential distribution | Equal servers |
| Least Connections | Send to least busy | Varying request times |
| IP Hash | Same client → same server | Session affinity |
| Weighted | More traffic to stronger servers | Mixed hardware |

## TLS/SSL

```mermaid
sequenceDiagram
    Client->>Server: ClientHello (supported ciphers)
    Server->>Client: ServerHello (chosen cipher) + Certificate
    Client->>Client: Verify certificate with CA
    Client->>Server: Key exchange (encrypted with server's public key)
    Note over Client,Server: Both derive session key
    Client->>Server: Encrypted application data
    Server->>Client: Encrypted response
```

## Proxy vs Reverse Proxy

```mermaid
graph LR
    subgraph ForwardProxy [Forward Proxy]
        C1[Client] --> FP[Proxy<br/>Squid] --> INT[Internet<br/>Servers]
    end
    subgraph ReverseProxy [Reverse Proxy]
        INT2[Internet<br/>Clients] --> RP[Reverse Proxy<br/>Nginx / HAProxy] --> S1[Server 1]
        RP --> S2[Server 2]
    end
```

| Type | Sits in front of | Purpose |
|---|---|---|
| Forward Proxy | Clients | Anonymity, caching, filtering |
| Reverse Proxy | Servers | Load balancing, SSL termination, caching |
