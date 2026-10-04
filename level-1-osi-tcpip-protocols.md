# LEVEL 1 — OSI Model, TCP/IP Model & Deep Dive into Protocols

> **Learning Flow:** Concept → Why → How → Real-world example → Visualization → Command → Hands-on exercise → Troubleshooting → Interview questions

---

## Table of Contents

### Part A — The Models
1. [The OSI Model — 7 Layers Deep](#part-a--the-osi-model)
2. [The TCP/IP Model — 4 Layers](#the-tcpip-model)
3. [OSI ↔ TCP/IP ↔ Protocols Mapping](#osi--tcpip--protocols-mapping)

### Part B — Protocols Deep Dive
4. [Ethernet (Layer 2)](#4-ethernet-layer-2--data-link)
5. [ARP (Layer 2)](#5-arp-layer-2--data-link)
6. [IP (Layer 3)](#6-ip-layer-3--network)
7. [ICMP (Layer 3)](#7-icmp-layer-3--network)
8. [TCP (Layer 4)](#8-tcp-layer-4--transport)
9. [UDP (Layer 4)](#9-udp-layer-4--transport)
10. [DNS (Layer 7)](#10-dns-layer-7--application)
11. [DHCP (Layer 7)](#11-dhcp-layer-7--application)
12. [HTTP (Layer 7)](#12-http-layer-7--application)
13. [HTTPS & TLS (Layer 5-7)](#13-https--tls-layer-5-7)
14. [SSH (Layer 7)](#14-ssh-layer-7--application)
15. [FTP/SFTP (Layer 7)](#15-ftpsftp-layer-7--application)
16. [SMTP (Layer 7)](#16-smtp-layer-7--application)

### Part C — Troubleshooting by Layer
17. [How to Diagnose Which Layer Is Broken](#part-c--troubleshooting-by-layer)
18. [Real Troubleshooting Scenarios](#real-troubleshooting-scenarios)

---

# Part A — The OSI Model

## Why Learn the OSI Model?

The OSI model is NOT something you memorize for an exam and forget. It's a **troubleshooting framework**. When something breaks in networking, you think:

> "Which layer is the problem at?"

That narrows your search from "something is broken somewhere" to a **specific area** with **specific tools**.

### Real-World Analogy

> 📦 Imagine sending a **gift to a friend in another city**. The process has layers:
>
> | Step | What Happens | OSI Layer Analogy |
> |------|-------------|-------------------|
> | 1. You write a letter | Content creation | **Application** (Layer 7) |
> | 2. You translate it to their language | Format/encoding | **Presentation** (Layer 6) |
> | 3. You track the conversation (reply expected) | Session management | **Session** (Layer 5) |
> | 4. Post office ensures delivery & order | Reliable transport | **Transport** (Layer 4) |
> | 5. Post office routes it to the right city | Addressing & routing | **Network** (Layer 3) |
> | 6. Local postman delivers to the door | Local delivery (MAC) | **Data Link** (Layer 2) |
> | 7. Physical truck carries the mail | Wires, cables, signals | **Physical** (Layer 1) |

---

## The 7 Layers — From Bottom to Top

### Visualization: The Full Stack

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 7: APPLICATION    │ HTTP, HTTPS, DNS, SSH, FTP, SMTP    │
│                          │ What the user/app interacts with     │
├─────────────────────────────────────────────────────────────────┤
│  Layer 6: PRESENTATION   │ SSL/TLS encryption, JPEG, JSON,     │
│                          │ Data format, compression, encoding   │
├─────────────────────────────────────────────────────────────────┤
│  Layer 5: SESSION        │ Establish, manage, terminate         │
│                          │ connections (TLS handshake, RPC)     │
├─────────────────────────────────────────────────────────────────┤
│  Layer 4: TRANSPORT      │ TCP (reliable), UDP (fast)           │
│                          │ Ports, segmentation, flow control    │
├─────────────────────────────────────────────────────────────────┤
│  Layer 3: NETWORK        │ IP, ICMP, routing                    │
│                          │ IP addressing, packet forwarding     │
├─────────────────────────────────────────────────────────────────┤
│  Layer 2: DATA LINK      │ Ethernet, ARP, MAC addresses        │
│                          │ Frames, switches, local delivery     │
├─────────────────────────────────────────────────────────────────┤
│  Layer 1: PHYSICAL       │ Cables, Wi-Fi signals, fiber optics  │
│                          │ Bits on the wire, electrical signals │
└─────────────────────────────────────────────────────────────────┘
```

---

### Layer 1: Physical

**What**: The actual **hardware and signals** — electrical pulses, light (fiber optic), radio waves (Wi-Fi).

**Responsible For**:
- Transmitting raw **bits** (0s and 1s) over a medium
- Cable types: Ethernet (Cat5e, Cat6), fiber optic, coaxial
- Connectors: RJ-45, SFP
- Signal: voltage levels, light pulses, radio frequencies
- Speed: 10 Mbps, 100 Mbps, 1 Gbps, 10 Gbps

**Real-World Analogy**:
> 🚛 Layer 1 is the **road itself** — the physical path. It doesn't know about addresses or packages, it just carries things from point A to point B.

**When Layer 1 Breaks**:
- Cable unplugged or damaged
- Wi-Fi signal too weak
- Network card hardware failure
- Wrong cable type (crossover vs straight-through)

**How to Troubleshoot**:
```bash
# Check if the interface is physically UP
ip link show eth0
# Look for "state UP" or "state DOWN"

# Check cable connection (Linux)
ethtool eth0
# Look for "Link detected: yes/no"

# Windows
Get-NetAdapter | Format-Table Name, Status, LinkSpeed
```

**How It Connects to Your Tech Stack**:
- AWS handles Layer 1 for you (that's one reason you use cloud!)
- On-premise servers — you manage cables, switches, and NICs
- If your EC2 instance can't connect → Layer 1 is NOT the issue (AWS manages it)

**Interview Q&A**:

1. **Q:** What layer deals with cables and electrical signals?
   **A:** Layer 1 — Physical layer. It handles the transmission of raw bits over a physical medium.

2. **Q:** If `ip link show` says the interface is DOWN, what layer has the problem?
   **A:** Layer 1 (Physical) or Layer 2 (Data Link). Check the cable, NIC, or driver first.

---

### Layer 2: Data Link

**What**: Responsible for **node-to-node (hop-by-hop) delivery** within a local network using **MAC addresses**.

**Responsible For**:
- **Framing**: Wrapping packets into frames with MAC headers
- **MAC addressing**: Identifying devices on the LAN
- **Error detection**: CRC/checksum in frame trailer
- **Switch operation**: Switches work at Layer 2
- **ARP**: Resolving IP → MAC

**Sub-layers**:
```
Layer 2: Data Link
├── LLC (Logical Link Control) — flow control, error handling
└── MAC (Media Access Control) — physical addressing, frame delivery
```

**Real-World Analogy**:
> 📮 Layer 2 is the **local postman**. He doesn't know about cities or countries (IP/routing). He only knows the houses on his street (MAC addresses in the LAN). He picks up the envelope (frame) and delivers it to the right door.

**When Layer 2 Breaks**:
- Duplicate MAC addresses
- Switch port misconfiguration
- VLAN mismatch
- ARP table poisoning
- Spanning tree loop

**How to Troubleshoot**:
```bash
# Check ARP table (IP → MAC mappings)
arp -a

# Check MAC address of your interface
ip link show eth0

# Watch ARP requests on the network
sudo tcpdump -i eth0 arp

# Check switch MAC table (on a managed switch)
# show mac address-table (Cisco)
```

**How It Connects to Your Tech Stack**:
- **Docker bridge** (`docker0`) is a Layer 2 virtual switch
- **Kubernetes** uses virtual Ethernet pairs (`veth`) — Layer 2 connections
- **AWS Security Groups** operate at Layers 3-4, but **NACLs** can affect Layer 2-like behavior

**Interview Q&A**:

1. **Q:** What is the role of a switch in networking?
   **A:** A switch operates at Layer 2. It uses MAC addresses to forward frames only to the correct port, rather than broadcasting to all ports (which a hub does).

2. **Q:** If two machines on the same LAN can't communicate but are on the same switch, what do you check?
   **A:** VLAN configuration, ARP resolution (`arp -a`), MAC address conflicts, and switch port status.

---

### Layer 3: Network

**What**: Responsible for **routing packets across different networks** using **IP addresses**.

**Responsible For**:
- **Logical addressing**: IP addresses (IPv4, IPv6)
- **Routing**: Finding the best path to the destination
- **Packet forwarding**: Routers work at Layer 3
- **Fragmentation**: Breaking large packets to fit MTU

**Key Protocols**: IP, ICMP, IGMP

**Real-World Analogy**:
> 🗺️ Layer 3 is the **GPS navigation system**. It knows how to route you from Delhi to Mumbai across multiple highways (networks). Each intersection (router) looks at the destination and decides the next turn (next hop).

**When Layer 3 Breaks**:
- Wrong IP address configured
- Subnet mask mismatch
- Routing table missing routes
- Gateway not configured
- Firewall blocking IP traffic

**How to Troubleshoot**:
```bash
# Check your IP and subnet
ip addr show

# Check your default gateway
ip route show
# Look for "default via x.x.x.x"

# Ping the gateway (is Layer 3 working locally?)
ping 192.168.1.1

# Ping an external IP (is routing working?)
ping 8.8.8.8

# Trace the route (see each hop)
traceroute 8.8.8.8     # Linux
tracert 8.8.8.8        # Windows

# Check routing table
ip route show          # Linux
route print            # Windows
```

**How It Connects to Your Tech Stack**:
- **AWS VPC** routing tables = Layer 3 routing
- **Kubernetes** uses `kube-proxy` and CNI plugins for Layer 3 routing between pods
- **Docker** networking uses `iptables` rules for Layer 3 routing
- **Nginx** reverse proxy works at Layer 7 but relies on Layer 3 for connectivity

**Interview Q&A**:

1. **Q:** What is the difference between a switch and a router?
   **A:** A switch operates at Layer 2 (uses MAC addresses, works within a LAN). A router operates at Layer 3 (uses IP addresses, routes between different networks).

2. **Q:** If `ping 8.8.8.8` works but `ping google.com` doesn't, what's the issue?
   **A:** Layer 3 (IP routing) is fine. The problem is at Layer 7 — DNS resolution is failing. Check `/etc/resolv.conf` or DNS server configuration.

---

### Layer 4: Transport

**What**: Responsible for **end-to-end communication** between applications. Handles **reliable/unreliable delivery**, **segmentation**, and **port numbers**.

**Key Protocols**: **TCP** (reliable) and **UDP** (fast)

**Responsible For**:
- **Segmentation**: Breaking data into segments
- **Port numbers**: Multiplexing multiple services on one IP
- **Flow control**: Preventing sender from overwhelming receiver
- **Error recovery**: TCP retransmits lost segments
- **Connection management**: TCP three-way handshake

**TCP vs UDP at a Glance**:

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Guaranteed delivery | Best-effort (no guarantee) |
| Order | Ordered delivery | Unordered |
| Speed | Slower (overhead) | Faster (minimal overhead) |
| Use case | HTTP, SSH, database, Kafka | DNS, video streaming, gaming |
| Header size | 20-60 bytes | 8 bytes |

**Real-World Analogy**:
> 📦 **TCP** is like **registered mail** — you get a tracking number, delivery confirmation, and it's resent if lost.
> **UDP** is like **throwing a paper airplane** — fast, no confirmation, if it crashes, oh well!

**When Layer 4 Breaks**:
- Port blocked by firewall
- Service not listening on the expected port
- TCP connection timeout (SYN sent, no SYN-ACK)
- Too many connections (connection exhaustion)
- Port already in use

**How to Troubleshoot**:
```bash
# Check if a service is listening on a port
ss -tlnp | grep 8080           # Linux
netstat -an | findstr 8080     # Windows

# Test TCP connection to a port
nc -zv 192.168.1.10 8080       # Linux (netcat)
Test-NetConnection -ComputerName 192.168.1.10 -Port 8080   # PowerShell

# See all active TCP connections
ss -tn

# Check for connection states
ss -tn state time-wait | wc -l     # Count TIME_WAIT connections
ss -tn state established | wc -l   # Count ESTABLISHED connections

# Check firewall rules
sudo iptables -L -n             # Linux
Get-NetFirewallRule | Where-Object {$_.Enabled -eq 'True'}   # Windows
```

**How It Connects to Your Tech Stack**:

| Tool | Protocol | Port |
|------|----------|------|
| React dev server | TCP | 3000 |
| Node.js/Express | TCP | 8080 |
| Spring Boot | TCP | 8080 |
| Nginx | TCP | 80, 443 |
| Redis | TCP | 6379 |
| Kafka | TCP | 9092 |
| RabbitMQ | TCP | 5672 |
| DNS lookup | UDP | 53 |
| Docker daemon | TCP | 2375/2376 |

**Interview Q&A**:

1. **Q:** Why does Kafka use TCP instead of UDP?
   **A:** Kafka needs guaranteed, ordered message delivery. TCP provides reliability, sequencing, and flow control — essential for a message streaming platform where losing messages is unacceptable.

2. **Q:** What is a TCP three-way handshake?
   **A:** SYN → SYN-ACK → ACK. The client sends SYN, server responds with SYN-ACK, client confirms with ACK. This establishes a reliable connection before data transfer begins.

3. **Q:** If `ping` works but `curl` to port 8080 doesn't, what layer is the issue?
   **A:** Layer 4 (Transport). The IP connectivity works (ping uses ICMP at Layer 3), but the TCP port 8080 is either blocked by a firewall, or no service is listening on it.

---

### Layer 5: Session

**What**: Manages **sessions** — establishing, maintaining, and terminating connections between applications.

**Responsible For**:
- Opening and closing communication sessions
- Session checkpointing and recovery
- Synchronization of data exchange
- Authentication handshakes

**Real-World Analogy**:
> 📞 Layer 5 is like the **call setup and teardown** in a phone call. Before you talk (data transfer), someone has to dial, the other person picks up, and at the end, someone hangs up. That's session management.

**Examples in Practice**:
- **TLS handshake** — establishing a secure session
- **RPC (Remote Procedure Call)** sessions
- **NetBIOS sessions** (older Windows networking)
- **WebSocket** connection establishment
- **Database connection pooling** (maintaining sessions to PostgreSQL/MySQL)

**How It Connects to Your Tech Stack**:
- **Spring Boot** + **Redis session storage** — HTTP sessions
- **WebSocket connections** in Node.js — long-lived sessions
- **Kafka consumer groups** — session management with brokers
- **Jenkins** — browser sessions with the Jenkins UI
- **kubectl exec** — session into a Kubernetes pod

**Interview Q&A**:

1. **Q:** Give a real-world example of session layer in modern applications.
   **A:** A WebSocket connection between a React frontend and a Node.js backend. The session layer manages the initial handshake (HTTP upgrade), keeps the connection alive, and handles graceful termination.

---

### Layer 6: Presentation

**What**: Handles **data formatting, encoding, encryption, and compression**. It translates data between the application format and the network format.

**Responsible For**:
- **Encryption/Decryption**: SSL/TLS (converting plaintext ↔ ciphertext)
- **Data format translation**: JSON, XML, HTML, JPEG, MP4
- **Character encoding**: UTF-8, ASCII
- **Compression**: gzip, deflate
- **Serialization**: Converting objects to byte streams (and back)

**Real-World Analogy**:
> 🌐 Layer 6 is like a **translator at the UN**. The speaker talks in French (app format), and the translator converts it to English (network format) so everyone can understand. Also handles "sealing the envelope" (encryption).

**Examples in Practice**:
```
Your Spring Boot app returns a Java object
    ↓ Layer 6: Jackson serializes it to JSON
    ↓ Layer 6: gzip compresses it
    ↓ Layer 6: TLS encrypts it
    ↓ Sent over the network
    ↓
React app receives encrypted, compressed JSON
    ↓ Layer 6: TLS decrypts it
    ↓ Layer 6: gzip decompresses it
    ↓ Layer 6: JSON.parse() converts to JS object
    ↓ React renders the data
```

**How It Connects to Your Tech Stack**:

| Tool | Layer 6 Activity |
|------|-----------------|
| **Nginx** | gzip compression, TLS termination |
| **Spring Boot** | Jackson JSON serialization |
| **Node.js** | JSON.stringify/parse, TLS |
| **Kafka** | Avro/Protobuf serialization |
| **Docker Registry** | Image layer compression |
| **Redis** | Data serialization |

**When Layer 6 Breaks**:
- TLS certificate expired → HTTPS fails
- JSON parsing error → API returns 400 Bad Request
- Character encoding mismatch → garbled text
- Content-Type header wrong → browser can't render response
- Compression mismatch → corrupted data

**Interview Q&A**:

1. **Q:** If your API returns `Content-Type: application/json` but the body is actually XML, what layer has the problem?
   **A:** Layer 6 (Presentation). The data format/encoding is incorrect. The server is advertising JSON but sending XML, causing parsing failures.

2. **Q:** Where does TLS encryption happen in the OSI model?
   **A:** Primarily at Layer 6 (Presentation) for encryption/decryption, with session establishment aspects at Layer 5 (Session).

---

### Layer 7: Application

**What**: The layer closest to the **end user**. This is where application-level protocols operate — the protocols your code directly interacts with.

**Responsible For**:
- **HTTP/HTTPS**: Web communication
- **DNS**: Domain name resolution
- **SSH**: Secure remote access
- **SMTP**: Email
- **FTP**: File transfer
- **Application APIs**: REST, GraphQL, gRPC

**Real-World Analogy**:
> 💻 Layer 7 is the **storefront** — it's what you see and interact with. When you type `google.com`, you interact with Layer 7 (HTTP). All the other layers are invisible plumbing behind the wall.

**When Layer 7 Breaks**:
- 404 Not Found — URL path wrong
- 500 Internal Server Error — app crashed
- 502 Bad Gateway — reverse proxy can't reach backend
- 503 Service Unavailable — app overloaded
- DNS resolution failure — can't resolve domain name
- Authentication failure — wrong credentials
- CORS error — browser blocking cross-origin request

**How to Troubleshoot**:
```bash
# Test HTTP
curl -v https://example.com

# Test DNS
nslookup example.com
dig example.com

# Test SSH
ssh -v user@server

# Check HTTP response codes
curl -o /dev/null -s -w "%{http_code}" https://example.com

# Check application logs
docker logs <container>
kubectl logs <pod>
journalctl -u nginx
```

**How It Connects to Your Tech Stack**:

Every tool you use operates at Layer 7:
- **React** → HTTP requests to APIs
- **Node.js/Express** → HTTP server
- **Spring Boot** → HTTP server + REST APIs
- **Nginx** → HTTP reverse proxy, Layer 7 load balancer
- **Jenkins** → HTTP web UI
- **kubectl** → HTTP REST calls to K8s API server
- **Redis CLI** → RESP protocol (application layer)
- **Kafka** → Custom binary protocol (application layer)
- **RabbitMQ** → AMQP protocol (application layer)

**Interview Q&A**:

1. **Q:** What is a Layer 7 load balancer?
   **A:** A load balancer that makes routing decisions based on application-level data — URL path, HTTP headers, cookies. Example: AWS ALB routes `/api/*` to backend and `/static/*` to a CDN.

2. **Q:** What's the difference between a Layer 4 and Layer 7 load balancer?
   **A:** Layer 4 (AWS NLB) routes based on IP + port. Layer 7 (AWS ALB, Nginx) routes based on HTTP content (URL, headers, cookies). Layer 7 is smarter but slightly slower.

---

## The TCP/IP Model

The TCP/IP model is the **practical model** actually used on the internet. It simplifies OSI's 7 layers into **4 layers**.

### The 4 Layers

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 4: APPLICATION     │ HTTP, DNS, SSH, FTP, SMTP, TLS │
│  (OSI Layers 5+6+7)      │ Everything the app touches     │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: TRANSPORT       │ TCP, UDP                       │
│  (OSI Layer 4)            │ End-to-end delivery, ports     │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: INTERNET        │ IP, ICMP, ARP                  │
│  (OSI Layer 3)            │ Routing, addressing            │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: NETWORK ACCESS  │ Ethernet, Wi-Fi, Fiber         │
│  (OSI Layers 1+2)         │ Physical + Data Link combined  │
└─────────────────────────────────────────────────────────────┘
```

### Why Two Models?

| | OSI Model | TCP/IP Model |
|---|---|---|
| **Layers** | 7 | 4 |
| **Purpose** | Theoretical reference | Practical implementation |
| **Created by** | ISO (International Standards Org) | DARPA/US Department of Defense |
| **Used for** | Learning, troubleshooting | Actual internet protocols |
| **Status** | Conceptual framework | What the internet runs on |

> 💡 **Think of it this way**: OSI is the **textbook**. TCP/IP is the **real code**. You learn OSI to understand concepts, but the internet runs on TCP/IP.

---

## OSI ↔ TCP/IP ↔ Protocols Mapping

This is the **master reference table** you'll come back to again and again:

```
┌──────────────────┬──────────────────┬────────────────────────────────────────┐
│    OSI Layer     │   TCP/IP Layer   │          Protocols & Examples          │
├──────────────────┼──────────────────┼────────────────────────────────────────┤
│ 7. Application   │                  │ HTTP, HTTPS, DNS, SSH, FTP, SMTP,     │
│ 6. Presentation  │  4. Application  │ TLS/SSL, JSON, gzip, AMQP,           │
│ 5. Session       │                  │ WebSocket, RESP (Redis), JDBC         │
├──────────────────┼──────────────────┼────────────────────────────────────────┤
│ 4. Transport     │  3. Transport    │ TCP, UDP                              │
├──────────────────┼──────────────────┼────────────────────────────────────────┤
│ 3. Network       │  2. Internet     │ IP (IPv4/IPv6), ICMP, IGMP            │
├──────────────────┼──────────────────┼────────────────────────────────────────┤
│ 2. Data Link     │  1. Network      │ Ethernet, ARP, Wi-Fi (802.11),        │
│ 1. Physical      │     Access       │ Fiber, MAC, PPP                       │
└──────────────────┴──────────────────┴────────────────────────────────────────┘
```

### Data Flow: How a Request Travels Down the Stack

```
YOU: curl https://api.example.com/users

Layer 7 (Application):  HTTP GET /users  Host: api.example.com
Layer 6 (Presentation): TLS encrypts the HTTP request; JSON content type
Layer 5 (Session):      TLS session established (session ID, keys)
Layer 4 (Transport):    TCP segment: Src Port=49152, Dst Port=443, Seq=1
Layer 3 (Network):      IP packet: Src=192.168.1.5, Dst=93.184.216.34
Layer 2 (Data Link):    Ethernet frame: Src MAC=AA:BB:CC:..., Dst MAC=Router MAC
Layer 1 (Physical):     Electrical signals on the wire → 101010110...
```

### Data Flow: How the Response Travels Up the Stack

```
WIRE: → 101010110...

Layer 1 (Physical):     Bits received from wire
Layer 2 (Data Link):    Frame extracted, MAC addresses checked, CRC verified
Layer 3 (Network):      IP packet extracted, destination IP matches us ✅
Layer 4 (Transport):    TCP segment extracted, port 49152 matches our process
Layer 5 (Session):      TLS session validated (session ID matches)
Layer 6 (Presentation): TLS decrypts data, decompresses gzip
Layer 7 (Application):  HTTP 200 OK, JSON body: {"users": [...]}

YOU: See the response! 🎉
```

---

# Part B — Protocols Deep Dive

## 4. Ethernet (Layer 2 — Data Link)

### Concept

**Ethernet** is the most common **LAN technology**. It defines how data is framed and transmitted over wired networks (and is the basis for wireless Wi-Fi standards too).

### What Ethernet Does

- Defines the **frame format** (header + payload + trailer)
- Uses **MAC addresses** for local delivery
- Defines **speed standards**: 10 Mbps, 100 Mbps (Fast Ethernet), 1 Gbps (Gigabit), 10 Gbps, 25 Gbps, 100 Gbps
- **CSMA/CD**: Collision detection (how devices share the wire)

### Ethernet Frame Structure

```
┌────────────┬────────────┬──────────┬───────────────────┬──────────┐
│  Preamble  │  Dest MAC  │ Src MAC  │    Payload        │   FCS    │
│  (8 bytes) │ (6 bytes)  │(6 bytes) │ (46-1500 bytes)   │(4 bytes) │
│            │            │          │ (IP Packet inside) │(checksum)│
└────────────┴────────────┴──────────┴───────────────────┴──────────┘
              ↑                        ↑                    ↑
         Who receives?           The actual data        Error check
```

### How It Connects to Your Tech Stack

- **Docker**: `docker0` bridge = a virtual Ethernet switch
- **Kubernetes**: `veth` pairs = virtual Ethernet cables between pods and the node
- **AWS**: VPC networking is virtualized Ethernet at heart
- When you run `docker run --network bridge nginx`, the container gets a virtual Ethernet interface

### Commands

```bash
# See your Ethernet interface details
ethtool eth0              # Speed, duplex, link status

# See Ethernet frames (with MAC addresses)
sudo tcpdump -i eth0 -e -c 5

# Check interface speed
cat /sys/class/net/eth0/speed
```

### ❓ Interview Questions

1. **Q:** What is the maximum payload size in an Ethernet frame?
   **A:** 1500 bytes (the MTU — Maximum Transmission Unit). This is why IP packets larger than 1500 bytes get fragmented.

2. **Q:** What is a jumbo frame?
   **A:** An Ethernet frame with a payload larger than 1500 bytes (typically 9000 bytes). Used in high-performance environments like storage networks and within AWS VPCs for better throughput.

---

## 5. ARP (Layer 2 — Data Link)

### Concept

**ARP (Address Resolution Protocol)** translates an **IP address → MAC address**. When your machine wants to send data to another device on the same LAN, it knows the IP but needs the MAC address to build the Ethernet frame.

### How ARP Works — Step by Step

```
Your PC wants to reach 192.168.1.1 (the router)

Step 1: PC checks ARP cache — "Do I already know the MAC for 192.168.1.1?"
        → If YES: use the cached MAC address
        → If NO: proceed to Step 2

Step 2: PC sends ARP REQUEST (broadcast to ALL devices):
        "Who has 192.168.1.1? Tell AA:BB:CC:DD:EE:FF (my MAC)"

        ┌──────────┐    BROADCAST     ┌──────────┐
        │  Your PC │ ──────────────→  │ Device A │  ← "Not me"
        │          │ ──────────────→  │ Device B │  ← "Not me"
        │          │ ──────────────→  │ Router   │  ← "That's ME!"
        └──────────┘                  └──────────┘

Step 3: Router sends ARP REPLY (unicast to your PC):
        "192.168.1.1 is at 11:22:33:44:55:66"

Step 4: PC caches this mapping: 192.168.1.1 → 11:22:33:44:55:66
        Now it can build the Ethernet frame!
```

### Real-World Analogy

> 📢 ARP is like walking into a room and shouting: **"Who lives at House #5?"** Everyone hears you (broadcast), but only the person at House #5 responds (unicast): **"That's me! Here's my ID badge (MAC)."**

### Commands

```bash
# View ARP cache (all known IP→MAC mappings)
arp -a                    # Linux/Windows
ip neigh show             # Linux (modern)

# Watch ARP requests in real-time
sudo tcpdump -i eth0 arp

# Clear ARP cache (force fresh resolution)
sudo ip neigh flush all   # Linux

# Add a static ARP entry
sudo arp -s 192.168.1.100 AA:BB:CC:DD:EE:FF
```

### 🧪 Hands-on Exercise

1. Run `arp -a` — see all devices your machine knows about on the LAN.
2. Run `sudo tcpdump -i eth0 arp` in one terminal.
3. In another terminal, ping a new device on your LAN.
4. Watch the ARP request (broadcast) and reply (unicast) in tcpdump!

### ⚠️ Security: ARP Spoofing

- An attacker can send fake ARP replies: "192.168.1.1 is at ATTACKER-MAC"
- All traffic meant for the router goes to the attacker → **Man-in-the-Middle attack**
- Prevention: Static ARP entries, Dynamic ARP Inspection (DAI) on switches

### How It Connects to Your Tech Stack

- **Docker**: Containers on the same bridge network use ARP to find each other
- **Kubernetes**: Pods use ARP within the node's bridge network
- **AWS**: ARP is handled internally in the VPC by AWS (you don't see it)

### ❓ Interview Questions

1. **Q:** Why is ARP needed if we already have IP addresses?
   **A:** Ethernet frames require MAC addresses for local delivery. ARP bridges the gap between Layer 3 (IP) and Layer 2 (MAC). Without ARP, the frame can't be constructed.

2. **Q:** Is ARP used for communication across the internet?
   **A:** No. ARP only works within a local network segment. For cross-network communication, routers handle the forwarding, and each LAN segment has its own ARP process.

---

## 6. IP (Layer 3 — Network)

### Concept

**IP (Internet Protocol)** is the **backbone of the internet**. It provides **logical addressing** (IP addresses) and **routing** — getting packets from source to destination across multiple networks.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Connectionless** | Each packet is routed independently |
| **Unreliable** | No guarantee of delivery (that's TCP's job) |
| **Best-effort** | Tries its best but doesn't retransmit |
| **Fragmentation** | Can split packets if they exceed MTU |

### IP Packet Header (Simplified)

```
┌─────────┬─────────┬─────────┬──────────┬─────────────────┐
│ Version │   TTL   │Protocol │ Source IP│ Destination IP   │
│ (4/6)   │(hops)   │(TCP/UDP)│          │                  │
├─────────┴─────────┴─────────┴──────────┴─────────────────┤
│                    PAYLOAD (data)                         │
└──────────────────────────────────────────────────────────┘
```

**Key Fields**:
- **Version**: IPv4 (4) or IPv6 (6)
- **TTL (Time to Live)**: Max hops before packet is discarded (prevents infinite loops)
- **Protocol**: What's inside — 6=TCP, 17=UDP, 1=ICMP
- **Source/Destination IP**: Where it's from and where it's going

### TTL — The Hop Counter

```
Your PC (TTL=64) → Router 1 (TTL=63) → Router 2 (TTL=62) → ... → Server

If TTL reaches 0, the packet is DROPPED and an ICMP "Time Exceeded" is sent back.
This is how `traceroute` works! It sends packets with TTL=1, 2, 3, ... and collects
the "Time Exceeded" replies from each hop.
```

### Commands

```bash
# See your IP configuration
ip addr show          # Linux
ipconfig              # Windows

# See routing table
ip route show         # Linux
route print           # Windows

# Trace the IP route to a destination
traceroute 8.8.8.8   # Linux
tracert 8.8.8.8      # Windows

# Check default TTL
cat /proc/sys/net/ipv4/ip_default_ttl   # Linux (usually 64)

# View IP headers in packets
sudo tcpdump -i eth0 -v -c 5    # -v shows IP header details
```

### How It Connects to Your Tech Stack

- **AWS VPC**: You define IP ranges (CIDR), subnets, and route tables — all Layer 3
- **Kubernetes**: Every pod gets its own IP address within the cluster CIDR
- **Docker**: Containers get IPs from the Docker subnet (e.g., 172.17.0.0/16)
- **Nginx**: `proxy_pass` directs traffic to backend IPs — Layer 3 routing

### ❓ Interview Questions

1. **Q:** What is TTL and why is it important?
   **A:** TTL (Time to Live) is a counter decremented at each router hop. When it reaches 0, the packet is discarded. This prevents packets from looping infinitely in a misconfigured network.

2. **Q:** Why is IP called "connectionless"?
   **A:** Each IP packet is routed independently. There's no pre-established path or session. Different packets from the same source might take different routes to the destination.

3. **Q:** What happens when a Kubernetes pod sends traffic to a pod on a different node?
   **A:** The CNI plugin (e.g., Calico, Flannel) handles IP routing. The packet is encapsulated (VXLAN or IPIP), routed via the node's IP to the other node, then decapsulated and delivered to the target pod's IP.

---

## 7. ICMP (Layer 3 — Network)

### Concept

**ICMP (Internet Control Message Protocol)** is the **diagnostic and error-reporting protocol** of the internet. It's used by tools like `ping` and `traceroute`.

### What ICMP Does

ICMP doesn't carry application data. It carries **control messages**:

| ICMP Type | Name | Used By |
|-----------|------|---------|
| 0 | Echo Reply | `ping` response |
| 3 | Destination Unreachable | Router can't deliver |
| 4 | Source Quench | "Slow down!" (deprecated) |
| 5 | Redirect | "Use a different route" |
| 8 | Echo Request | `ping` request |
| 11 | Time Exceeded | TTL reached 0 (`traceroute`) |

### How `ping` Works (Behind the Scenes)

```
Your PC                              Google DNS (8.8.8.8)
   │                                       │
   │──── ICMP Echo Request (Type 8) ──────→│
   │                                       │
   │←── ICMP Echo Reply (Type 0) ─────────│
   │                                       │
   RTT = time between request and reply
```

### How `traceroute` Works (Behind the Scenes)

```
Your PC sends packets with increasing TTL:

TTL=1: Packet reaches Router 1 → TTL=0 → Router 1 sends "Time Exceeded" back
TTL=2: Packet reaches Router 2 → TTL=0 → Router 2 sends "Time Exceeded" back
TTL=3: Packet reaches Router 3 → TTL=0 → Router 3 sends "Time Exceeded" back
...
TTL=N: Packet reaches destination → Destination sends "Echo Reply" → Done!

Result: You see every hop along the path! 🗺️
```

### Commands

```bash
# Basic ping
ping -c 4 8.8.8.8           # Linux (4 pings)
ping -n 4 8.8.8.8           # Windows

# Traceroute (see every hop)
traceroute 8.8.8.8          # Linux
tracert 8.8.8.8             # Windows

# Ping with specific TTL
ping -t 3 -c 1 8.8.8.8     # Linux: TTL=3, see which router responds

# Capture ICMP packets
sudo tcpdump -i eth0 icmp
```

### ⚠️ Gotcha: "Ping Works" ≠ "Everything Works"

```
Scenario: ping 8.8.8.8 ✅ works
          curl https://example.com ❌ fails

This means:
  Layer 1 ✅ (physical connection)
  Layer 2 ✅ (local delivery)
  Layer 3 ✅ (IP routing — ICMP works)
  Layer 4 ❓ (maybe TCP port blocked)
  Layer 7 ❓ (maybe DNS failing, maybe app error)

→ ICMP success only proves Layer 1-3 are working!
```

### How It Connects to Your Tech Stack

- **Kubernetes liveness probes**: Can use TCP/HTTP checks (not usually ICMP)
- **AWS Security Groups**: By default, ICMP may be blocked. You must add an inbound rule to allow ping.
- **Docker healthchecks**: Usually HTTP-based, not ICMP
- **Monitoring (Nagios, Prometheus)**: Often use ICMP ping for basic host availability checks

### ❓ Interview Questions

1. **Q:** If you can't ping an AWS EC2 instance, does that mean it's down?
   **A:** Not necessarily. The Security Group might be blocking ICMP traffic. You need to add an inbound rule for ICMP (All ICMP - IPv4) from your IP.

2. **Q:** Why doesn't Kubernetes use ICMP for health checks?
   **A:** ICMP only tests Layer 3 reachability. Kubernetes needs to verify the **application** is healthy (Layer 7), so it uses HTTP GET, TCP connect, or command-based probes instead.

---

## 8. TCP (Layer 4 — Transport)

### Concept

**TCP (Transmission Control Protocol)** is the **reliable transport protocol** of the internet. It guarantees that data arrives **complete, in order, and without errors**.

### The Three-Way Handshake — Deep Dive

```
Client                           Server
  │                                │
  │──── SYN (seq=100) ───────────→│   "Hey, I want to connect"
  │                                │
  │←─── SYN-ACK (seq=300,         │   "OK, I acknowledge. I want to
  │      ack=101) ────────────────│    connect too"
  │                                │
  │──── ACK (ack=301) ───────────→│   "Great, connection established!"
  │                                │
  │     ✅ CONNECTION ESTABLISHED  │
  │                                │
  │──── Data: GET /index.html ───→│
  │←─── Data: <html>...</html> ───│
  │                                │
```

### TCP Connection Termination (Four-Way Teardown)

```
Client                           Server
  │                                │
  │──── FIN ─────────────────────→│   "I'm done sending"
  │←─── ACK ─────────────────────│   "OK, acknowledged"
  │←─── FIN ─────────────────────│   "I'm done too"
  │──── ACK ─────────────────────→│   "OK, connection closed"
  │                                │
  │     ✅ CONNECTION CLOSED       │
```

### TCP Reliability Mechanisms

| Mechanism | What It Does |
|-----------|-------------|
| **Sequence Numbers** | Each byte is numbered, so data can be reassembled in order |
| **Acknowledgments (ACK)** | Receiver confirms which bytes it got |
| **Retransmission** | If ACK not received within timeout, resend the data |
| **Flow Control** | Receiver tells sender its buffer size (window) |
| **Congestion Control** | Sender slows down if network is congested |
| **Checksum** | Detects corrupted data |

### TCP States You Should Know

```
LISTEN      → Server waiting for connections (ss -tlnp shows this)
SYN_SENT    → Client sent SYN, waiting for SYN-ACK
SYN_RECV    → Server received SYN, sent SYN-ACK, waiting for ACK
ESTABLISHED → Connection active, data flowing
FIN_WAIT_1  → Sent FIN, waiting for ACK
FIN_WAIT_2  → Got ACK for FIN, waiting for server's FIN
TIME_WAIT   → Connection closed, waiting 2*MSL before recycling port
CLOSE_WAIT  → Received FIN, waiting for app to close
LAST_ACK    → Sent FIN, waiting for final ACK
CLOSED      → No connection
```

### ⚠️ Common Problem: Too Many TIME_WAIT Connections

```bash
# Check TIME_WAIT count
ss -tn state time-wait | wc -l

# If you see thousands of TIME_WAIT connections:
# → Your app is opening/closing many short-lived connections
# → Solution: Use connection pooling (database, HTTP keep-alive)
```

This is common with:
- **Spring Boot** → PostgreSQL without connection pooling (HikariCP fixes this)
- **Node.js** → HTTP clients without keep-alive
- **Microservices** making many inter-service calls

### Commands

```bash
# See TCP connections
ss -tn                        # All TCP connections
ss -tn state established      # Only established
ss -tlnp                      # Listening ports with process names

# Watch the three-way handshake
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0' -c 10

# Check TCP connection to a port
nc -zv google.com 443         # Linux
Test-NetConnection google.com -Port 443   # PowerShell

# See TCP stats
ss -s                         # Summary of all connections
```

### 🧪 Hands-on Exercise

1. Terminal 1: Start a listener: `nc -l 9999` (or `ncat -l 9999`)
2. Terminal 2: Run tcpdump: `sudo tcpdump -i lo port 9999`
3. Terminal 3: Connect: `nc localhost 9999`
4. Watch the **SYN → SYN-ACK → ACK** in tcpdump!
5. Type messages in Terminal 3, see them appear in Terminal 1.
6. Close with Ctrl+C, watch the **FIN → ACK → FIN → ACK** in tcpdump!

### ❓ Interview Questions

1. **Q:** Why does TCP use a three-way handshake instead of a two-way?
   **A:** Both sides need to confirm they can send AND receive. A two-way handshake only confirms one direction. The third packet (ACK) confirms the client received the server's SYN-ACK, establishing bidirectional communication.

2. **Q:** What is TCP window size?
   **A:** The window size tells the sender how many bytes the receiver can accept before needing an ACK. It's a flow control mechanism to prevent overwhelming the receiver's buffer.

3. **Q:** In a microservices architecture, why is TCP connection pooling important?
   **A:** Without pooling, each request creates a new TCP connection (handshake overhead + TIME_WAIT accumulation). Connection pooling reuses existing connections, reducing latency and port exhaustion. This is why Spring Boot uses HikariCP for database connections.

---

## 9. UDP (Layer 4 — Transport)

### Concept

**UDP (User Datagram Protocol)** is the **fast, lightweight transport protocol**. Unlike TCP, it provides **no guarantees** — no handshake, no retransmission, no ordering.

### UDP Datagram Structure

```
┌──────────────┬──────────────┬──────────┬──────────┐
│  Source Port │  Dest Port   │  Length  │ Checksum  │
│  (2 bytes)   │  (2 bytes)   │(2 bytes) │(2 bytes)  │
├──────────────┴──────────────┴──────────┴──────────┤
│                    Payload/Data                    │
└───────────────────────────────────────────────────┘

Total header: only 8 bytes! (TCP header: 20-60 bytes)
```

### When to Use UDP

| Use Case | Why UDP? |
|----------|---------|
| **DNS lookups** | Small queries, speed matters, can retry if lost |
| **Video streaming** | Losing a frame is OK, buffering is worse |
| **VoIP (Zoom/Teams)** | Real-time, can't wait for retransmission |
| **Gaming** | Low latency critical, lost packet = skip, don't retry |
| **DHCP** | Initial IP assignment, no connection exists yet |
| **NTP** (time sync) | Small packets, speed over reliability |
| **IoT sensors** | Lightweight, resource-constrained devices |

### TCP vs UDP — Visual

```
TCP (Registered Mail):
  "Did you get packet 1?"  → "Yes!" ✅
  "Did you get packet 2?"  → "Yes!" ✅
  "Did you get packet 3?"  → ...silence...
  "Resending packet 3!"    → "Got it!" ✅

UDP (Paper Airplane):
  *throws packet 1* ✈️
  *throws packet 2* ✈️
  *throws packet 3* ✈️   ← this one crashed
  "Oh well, moving on!" 🤷
```

### Commands

```bash
# See UDP listeners
ss -ulnp

# Send a UDP packet
echo "hello" | nc -u -w1 localhost 5000

# Listen for UDP
nc -u -l 5000

# See UDP traffic
sudo tcpdump -i eth0 udp

# DNS uses UDP — watch DNS queries
sudo tcpdump -i eth0 port 53
```

### How It Connects to Your Tech Stack

- **DNS resolution** for every URL your apps use → UDP (port 53)
- **Docker DNS** (internal name resolution between containers) → UDP
- **Kubernetes CoreDNS** → UDP (port 53)
- **DHCP** when your EC2 instance gets its IP → UDP
- **Some logging systems** (syslog) → UDP (port 514)

### ❓ Interview Questions

1. **Q:** Why does DNS use UDP instead of TCP?
   **A:** DNS queries are small (typically fit in one packet) and speed matters. If a response is lost, the client can simply retry. However, DNS **does** fall back to TCP for responses larger than 512 bytes (e.g., zone transfers).

2. **Q:** Can you build a reliable protocol on top of UDP?
   **A:** Yes! **QUIC** (used by HTTP/3) is built on UDP but adds its own reliability, ordering, and encryption. This gives the speed of UDP with the reliability of TCP, plus faster connection setup.

---

## 10. DNS (Layer 7 — Application)

### Concept

**DNS (Domain Name System)** is the **phonebook of the internet**. It translates human-readable domain names (google.com) into IP addresses (142.250.182.14).

### Why DNS?

Humans remember names. Computers use numbers. DNS bridges the gap.

```
You type: https://api.example.com/users
DNS says: api.example.com → 93.184.216.34
Now your browser can connect to 93.184.216.34:443
```

### DNS Resolution — The Full Journey

```
You type: https://www.google.com

Step 1: Browser Cache
  → "Do I already know the IP?" → If YES, use it. If NO ↓

Step 2: OS Cache (Host File)
  → Check /etc/hosts (Linux) or C:\Windows\System32\drivers\etc\hosts (Windows)
  → If found, use it. If NO ↓

Step 3: Resolver (ISP DNS / Corporate DNS / 8.8.8.8)
  → Ask the configured DNS server. If cached, return. If NO ↓

Step 4: Root DNS Server (.)
  → "I don't know google.com, but .com is handled by these servers" ↓

Step 5: TLD DNS Server (.com)
  → "I don't know www.google.com, but google.com is handled by these NS" ↓

Step 6: Authoritative DNS Server (google.com)
  → "www.google.com is 142.250.182.14" ✅

Step 7: Response cached at each level for future queries
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | Domain → IPv4 address | `google.com → 142.250.182.14` |
| **AAAA** | Domain → IPv6 address | `google.com → 2607:f8b0::` |
| **CNAME** | Alias (domain → domain) | `www.example.com → example.com` |
| **MX** | Mail server | `gmail.com → alt1.gmail-smtp-in.l.google.com` |
| **NS** | Name server | `google.com → ns1.google.com` |
| **TXT** | Arbitrary text | SPF, DKIM, domain verification |
| **SRV** | Service location | Used in Kubernetes for service discovery |
| **PTR** | Reverse DNS (IP → domain) | `8.8.8.8 → dns.google` |
| **SOA** | Zone authority | Who is responsible for this zone |

### Commands

```bash
# Basic lookup
nslookup google.com
dig google.com

# Query specific record types
dig google.com A         # IPv4
dig google.com AAAA      # IPv6
dig google.com MX        # Mail servers
dig google.com NS        # Name servers
dig google.com CNAME     # Aliases
dig google.com TXT       # TXT records

# See the full resolution chain
dig +trace google.com

# Use a specific DNS server
dig @8.8.8.8 google.com

# Check DNS cache (Windows)
ipconfig /displaydns

# Flush DNS cache
sudo systemd-resolve --flush-caches    # Linux (systemd)
ipconfig /flushdns                     # Windows

# Check /etc/hosts
cat /etc/hosts
```

### 🧪 Hands-on Exercise

1. Run `dig +trace google.com` — watch the full resolution from root → TLD → authoritative.
2. Add a fake entry to `/etc/hosts`: `127.0.0.1 myapp.local`
3. Run `ping myapp.local` — it resolves to 127.0.0.1! That's how local development domains work.
4. Run `dig +short google.com @8.8.8.8` vs `dig +short google.com @1.1.1.1` — see if different DNS servers return different IPs (CDN load balancing).

### How It Connects to Your Tech Stack

| Tool | DNS Usage |
|------|----------|
| **Docker** | Internal DNS: containers can reach each other by name (`redis`, `postgres`) |
| **Kubernetes** | CoreDNS: `my-service.my-namespace.svc.cluster.local` |
| **AWS** | Route 53: Managed DNS service, A/CNAME records for load balancers |
| **Nginx** | `proxy_pass http://backend-service;` — needs DNS to resolve `backend-service` |
| **Spring Boot** | `spring.datasource.url=jdbc:postgresql://db-host:5432/mydb` — `db-host` resolved by DNS |

### ⚠️ Common DNS Problems

```
Problem: "Could not resolve host"
  → DNS server unreachable, or domain doesn't exist
  → Check: /etc/resolv.conf (Linux), DNS settings

Problem: "Works with IP but not hostname"
  → DNS resolution failing
  → Test: dig <hostname>, nslookup <hostname>

Problem: "Old IP still being used after migration"
  → DNS cache / high TTL
  → Fix: Flush DNS cache, lower TTL before migration
```

### ❓ Interview Questions

1. **Q:** How does DNS work in Kubernetes?
   **A:** Kubernetes runs CoreDNS as a cluster add-on. Each Service gets a DNS record: `<service>.<namespace>.svc.cluster.local`. When Pod A calls `http://my-service`, CoreDNS resolves it to the Service's ClusterIP.

2. **Q:** What is the difference between a CNAME and an A record?
   **A:** An **A record** maps a domain directly to an IP address. A **CNAME** maps a domain to another domain (alias). Example: `www.example.com` CNAME → `example.com` → A → `93.184.216.34`.

3. **Q:** Why should you lower DNS TTL before a migration?
   **A:** If TTL is high (e.g., 24 hours), clients will cache the old IP for up to 24 hours after you change the DNS record. Lowering TTL to 60 seconds before the migration ensures clients pick up the new IP quickly.

---

## 11. DHCP (Layer 7 — Application)

### Concept

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses and network configuration to devices. Without DHCP, you'd have to manually configure every device.

### What DHCP Assigns

| Parameter | Example |
|-----------|---------|
| IP address | `192.168.1.50` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.1.1` |
| DNS server | `8.8.8.8, 8.8.4.4` |
| Lease time | 24 hours |

### DHCP Process — DORA

```
 Client                          DHCP Server
   │                                  │
   │── 1. DISCOVER (broadcast) ─────→│  "Anyone have an IP for me?"
   │                                  │
   │←── 2. OFFER (unicast) ──────────│  "Here, you can use 192.168.1.50"
   │                                  │
   │── 3. REQUEST (broadcast) ──────→│  "I'll take 192.168.1.50, please"
   │                                  │
   │←── 4. ACK (unicast) ───────────│  "Confirmed! It's yours for 24h"
   │                                  │
   │  ✅ IP configured!              │
```

**D**iscover → **O**ffer → **R**equest → **A**ck = **DORA** (easy to remember!)

### Real-World Analogy

> 🏨 DHCP is like checking into a **hotel**:
> 1. **Discover**: "Do you have any rooms?" (broadcast)
> 2. **Offer**: "Room 304 is available" (hotel responds)
> 3. **Request**: "I'll take Room 304" (you confirm)
> 4. **Acknowledge**: "Here's your key. Checkout is in 24 hours" (lease time)

### Commands

```bash
# See your DHCP-assigned info (Linux)
ip addr show
cat /var/lib/dhcp/dhclient.leases   # DHCP lease details

# Release and renew DHCP lease
sudo dhclient -r    # Release
sudo dhclient       # Renew

# Windows
ipconfig /all       # Shows DHCP info
ipconfig /release   # Release lease
ipconfig /renew     # Request new lease
```

### How It Connects to Your Tech Stack

- **Docker**: Containers get IPs from Docker's built-in DHCP-like mechanism (IPAM)
- **Kubernetes**: Pods get IPs from the CNI plugin's IPAM (IP Address Management)
- **AWS EC2**: Instances get private IPs via DHCP from the VPC
- **Home/Office**: Your laptop gets its IP from the Wi-Fi router's DHCP server

### ❓ Interview Questions

1. **Q:** What happens when a DHCP lease expires?
   **A:** The client must renew the lease (usually attempts at 50% and 87.5% of lease time). If renewal fails, the client loses its IP and must restart the DORA process.

2. **Q:** Why is the DHCP Discover a broadcast?
   **A:** The client doesn't know the DHCP server's IP yet (it doesn't have any IP configuration at all), so it broadcasts to `255.255.255.255` on the LAN, hoping a DHCP server will respond.

---

## 12. HTTP (Layer 7 — Application)

### Concept

**HTTP (HyperText Transfer Protocol)** is the foundation of web communication. It defines how **clients request** and **servers respond** with web content.

### HTTP Request Structure

```
GET /api/users HTTP/1.1          ← Method, Path, Version
Host: api.example.com            ← Required header
Accept: application/json         ← I want JSON
Authorization: Bearer <token>    ← Auth credential
Content-Type: application/json   ← Body format (for POST/PUT)
                                  ← Empty line separates headers from body
{"name": "John"}                 ← Body (for POST/PUT)
```

### HTTP Response Structure

```
HTTP/1.1 200 OK                  ← Version, Status Code, Reason
Content-Type: application/json   ← Response format
Content-Length: 42               ← Body size
Cache-Control: max-age=3600      ← Caching instruction
                                  ← Empty line
{"id": 1, "name": "John"}       ← Response body
```

### HTTP Methods

| Method | Purpose | Idempotent? | Safe? | Example |
|--------|---------|-------------|-------|---------|
| **GET** | Retrieve data | ✅ Yes | ✅ Yes | `GET /api/users` |
| **POST** | Create new resource | ❌ No | ❌ No | `POST /api/users` |
| **PUT** | Replace entire resource | ✅ Yes | ❌ No | `PUT /api/users/1` |
| **PATCH** | Partially update | ❌ No | ❌ No | `PATCH /api/users/1` |
| **DELETE** | Remove resource | ✅ Yes | ❌ No | `DELETE /api/users/1` |
| **HEAD** | GET without body | ✅ Yes | ✅ Yes | `HEAD /api/users` |
| **OPTIONS** | Supported methods | ✅ Yes | ✅ Yes | CORS preflight |

### HTTP Status Codes — Groups

| Range | Category | Common Codes |
|-------|----------|-------------|
| **1xx** | Informational | 100 Continue, 101 Switching Protocols (WebSocket) |
| **2xx** | Success ✅ | 200 OK, 201 Created, 204 No Content |
| **3xx** | Redirection 🔄 | 301 Moved Permanently, 302 Found, 304 Not Modified |
| **4xx** | Client Error ❌ | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| **5xx** | Server Error 💥 | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

### HTTP Versions

| Version | Key Feature |
|---------|-------------|
| **HTTP/1.0** | One request per connection |
| **HTTP/1.1** | Keep-alive, pipelining, Host header |
| **HTTP/2** | Multiplexing (multiple requests on one connection), binary, header compression |
| **HTTP/3** | QUIC (UDP-based), even faster connection setup |

### Commands

```bash
# Simple GET request
curl https://api.example.com/users

# Verbose (see headers, TLS handshake, everything!)
curl -v https://api.example.com/users

# POST with JSON body
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}'

# See only response headers
curl -I https://example.com

# See only status code
curl -o /dev/null -s -w "%{http_code}\n" https://example.com

# Follow redirects
curl -L https://example.com

# Send custom headers
curl -H "Authorization: Bearer mytoken" https://api.example.com/users
```

### How It Connects to Your Tech Stack

| Tool | HTTP Role |
|------|----------|
| **React** | Sends HTTP requests (fetch/axios) to backend APIs |
| **Node.js/Express** | HTTP server — handles routes, returns responses |
| **Spring Boot** | HTTP server — REST controllers, `@GetMapping`, `@PostMapping` |
| **Nginx** | HTTP reverse proxy, static file server |
| **Apache** | HTTP server |
| **Jenkins** | HTTP web UI, webhook receivers |
| **Kubernetes API** | Everything is HTTP REST (`kubectl` → HTTP calls to API server) |
| **Docker Registry** | HTTP API for push/pull images |

### ❓ Interview Questions

1. **Q:** What is the difference between 401 and 403?
   **A:** **401 Unauthorized** = "I don't know who you are" (missing or invalid credentials). **403 Forbidden** = "I know who you are, but you don't have permission."

2. **Q:** What is a 502 Bad Gateway?
   **A:** The reverse proxy (Nginx/ALB) could not get a valid response from the upstream server. Common causes: backend crashed, wrong upstream address, backend too slow (timeout).

3. **Q:** What is HTTP keep-alive?
   **A:** HTTP/1.1 feature that reuses the same TCP connection for multiple requests, avoiding the overhead of TCP handshake for each request. Critical for performance in microservices.

---

## 13. HTTPS & TLS (Layer 5-7)

### Concept

**HTTPS** = HTTP + **TLS (Transport Layer Security)**. TLS encrypts the HTTP communication so no one can eavesdrop or tamper with the data.

### Why HTTPS?

Without TLS, HTTP traffic is **plaintext**. Anyone on the same network (coffee shop Wi-Fi, ISP, etc.) can read:
- Your passwords
- Your credit card numbers
- Your API tokens
- Your cookies

### TLS Handshake — Behind the Scenes

```
Client (Browser)                        Server (Nginx)
     │                                       │
     │── 1. ClientHello ───────────────────→│
     │   (supported TLS versions, cipher     │
     │    suites, random number)             │
     │                                       │
     │←── 2. ServerHello ──────────────────│
     │   (chosen TLS version, cipher suite,  │
     │    server's random number)            │
     │                                       │
     │←── 3. Certificate ─────────────────│
     │   (server's SSL certificate with      │
     │    public key, signed by CA)          │
     │                                       │
     │── 4. Key Exchange ──────────────────→│
     │   (client generates pre-master secret,│
     │    encrypts with server's public key) │
     │                                       │
     │   Both derive the SAME session key    │
     │   from the pre-master secret          │
     │                                       │
     │←→ 5. Finished ─────────────────────→│
     │   (both confirm: "I'm using this key")│
     │                                       │
     │   ✅ ENCRYPTED TUNNEL ESTABLISHED 🔒  │
     │                                       │
     │←→ 6. HTTP data (encrypted) ────────→│
     │   GET /api/users (encrypted)          │
     │   200 OK {"users": [...]} (encrypted) │
```

### Certificate Chain of Trust

```
Root CA (trusted by browsers/OS)
  └── Intermediate CA
       └── Your Certificate (*.example.com)
            └── Signed by Intermediate CA
                 └── Which is signed by Root CA
                      └── Which is trusted by the browser ✅
```

### Commands

```bash
# See TLS certificate details
openssl s_client -connect google.com:443
openssl s_client -connect google.com:443 -servername google.com

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# See the full TLS handshake with curl
curl -v https://example.com 2>&1 | grep -E "SSL|TLS|certificate"

# Test TLS versions
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Generate a self-signed certificate (for development)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

### How It Connects to Your Tech Stack

| Tool | TLS Usage |
|------|----------|
| **Nginx** | TLS termination (decrypts HTTPS → forwards HTTP to backend) |
| **AWS ALB** | TLS termination at the load balancer |
| **AWS ACM** | Free managed TLS certificates |
| **Spring Boot** | Can configure `server.ssl.*` for HTTPS |
| **Node.js** | `https.createServer({key, cert}, app)` |
| **Let's Encrypt** | Free TLS certificates (certbot) |
| **Kubernetes** | Ingress controllers handle TLS termination |
| **Docker** | Registry can be configured with TLS |

### Common TLS Architecture

```
[Browser] ──HTTPS──→ [AWS ALB / Nginx (TLS Termination)] ──HTTP──→ [Spring Boot / Node.js]
                         ↑                                              ↑
                  Certificate here                              No TLS needed
                  (decrypt HTTPS)                             (internal network)
```

> 💡 This is called **TLS termination** or **SSL offloading** — the reverse proxy handles encryption so your backend doesn't have to.

### ❓ Interview Questions

1. **Q:** What is TLS termination and why is it important?
   **A:** TLS termination is decrypting HTTPS traffic at the load balancer/reverse proxy, then forwarding plain HTTP to backend services. It reduces CPU load on application servers and centralizes certificate management.

2. **Q:** What happens when a TLS certificate expires?
   **A:** Browsers show a security warning, and most will block the connection. Users see "Your connection is not private." Automated systems (APIs, webhooks) will fail with SSL errors.

3. **Q:** What is the difference between TLS 1.2 and TLS 1.3?
   **A:** TLS 1.3 is faster (1-RTT handshake vs 2-RTT), more secure (removed weak cipher suites), and simpler. TLS 1.3 also supports 0-RTT resumption for even faster subsequent connections.

---

## 14. SSH (Layer 7 — Application)

### Concept

**SSH (Secure Shell)** provides **encrypted remote access** to servers. It replaces insecure protocols like Telnet and rlogin.

### What SSH Does

- **Remote shell access**: Login to servers securely
- **File transfer**: SCP, SFTP
- **Port forwarding/tunneling**: Access remote services through a secure tunnel
- **Key-based authentication**: Passwordless, more secure login

### SSH Authentication Methods

```
Method 1: Password Authentication
  ssh user@server → Enter password → Connected

Method 2: Key-Based Authentication (recommended)
  Your machine has: private key (~/.ssh/id_rsa) — KEEP SECRET
  Server has:       public key  (~/.ssh/authorized_keys)

  ssh user@server →
    Client sends public key fingerprint →
    Server checks authorized_keys →
    Server sends challenge encrypted with public key →
    Client decrypts with private key →
    ✅ Authenticated without a password!
```

### SSH Port Forwarding (Tunneling)

```
Scenario: Redis on a server is only listening on 127.0.0.1:6379.
          You need to access it from your laptop.

Local Port Forwarding:
  ssh -L 6379:localhost:6379 user@server

  Your Laptop               SSH Tunnel               Server
  [redis-cli]               ═══════════>            [Redis]
  localhost:6379  ──SSH──→  server:22  ──→  localhost:6379

Now on your laptop:
  redis-cli -h localhost -p 6379  ← Actually connects to the server's Redis!
```

### Commands

```bash
# Basic SSH connection
ssh user@192.168.1.10
ssh -i ~/.ssh/my-key.pem ec2-user@52.86.123.45    # AWS EC2

# Generate SSH key pair
ssh-keygen -t ed25519 -C "your.email@example.com"
# Creates: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)

# Copy public key to server
ssh-copy-id user@server

# SCP — copy files over SSH
scp file.txt user@server:/tmp/
scp -r folder/ user@server:/home/user/

# SSH tunneling — local port forwarding
ssh -L 5432:localhost:5432 user@db-server
# Now: psql -h localhost -p 5432  → connects to remote PostgreSQL

# SSH tunneling — dynamic SOCKS proxy
ssh -D 8080 user@server
# Now: Configure browser to use localhost:8080 as SOCKS proxy

# SSH config file (~/.ssh/config) — simplify connections
# Host myserver
#   HostName 52.86.123.45
#   User ec2-user
#   IdentityFile ~/.ssh/my-key.pem
# Then just: ssh myserver
```

### How It Connects to Your Tech Stack

| Tool | SSH Usage |
|------|----------|
| **AWS EC2** | Primary access method (`ssh -i key.pem ec2-user@...`) |
| **Git** | `git clone git@github.com:user/repo.git` (SSH protocol) |
| **Jenkins** | SSH agent for connecting to build nodes |
| **Ansible** | Uses SSH to configure remote servers |
| **SCP/SFTP** | Deploy files to servers |
| **kubectl** | `kubectl exec` uses a similar concept (not SSH, but similar) |
| **Docker** | `docker -H ssh://user@server` — remote Docker over SSH |

### ❓ Interview Questions

1. **Q:** Why is key-based SSH authentication more secure than passwords?
   **A:** Keys are cryptographically strong (2048+ bits), immune to brute-force attacks, and the private key never leaves your machine. Passwords can be guessed, intercepted, or phished.

2. **Q:** What is SSH port forwarding used for?
   **A:** Accessing services behind firewalls or on private networks. Example: Accessing a database on a private AWS subnet by tunneling through a bastion host.

---

## 15. FTP/SFTP (Layer 7 — Application)

### Concept

| Protocol | Port | Encrypted? | Use |
|----------|------|-----------|-----|
| **FTP** | 21 (control), 20 (data) | ❌ No | Legacy file transfer |
| **FTPS** | 990 | ✅ Yes (TLS) | FTP over TLS |
| **SFTP** | 22 | ✅ Yes (SSH) | File transfer over SSH |
| **SCP** | 22 | ✅ Yes (SSH) | Simple file copy over SSH |

> ⚠️ **Never use plain FTP** in production. Credentials and data are sent in plaintext. Use SFTP or SCP instead.

### FTP Architecture (Why It's Weird)

```
FTP uses TWO connections:
1. Control connection (port 21) — commands: LIST, RETR, STOR
2. Data connection (port 20) — actual file data

This causes problems with firewalls and NAT!
(SFTP doesn't have this problem — it uses a single SSH connection)
```

### Commands

```bash
# SFTP (interactive)
sftp user@server
sftp> put localfile.txt         # Upload
sftp> get remotefile.txt        # Download
sftp> ls                        # List remote files

# SCP (one-shot copy)
scp file.txt user@server:/path/          # Upload
scp user@server:/path/file.txt ./        # Download
scp -r folder/ user@server:/path/        # Upload directory

# rsync (better for large transfers / incremental sync)
rsync -avz ./project/ user@server:/deploy/project/
```

### How It Connects to Your Tech Stack

- **CI/CD**: Jenkins may deploy artifacts via SCP/SFTP to servers
- **Legacy systems**: Some enterprise systems still use FTP for file exchange
- **AWS S3**: Replaced FTP/SFTP for most file storage needs
- **AWS Transfer Family**: Managed SFTP service backed by S3

### ❓ Interview Questions

1. **Q:** Why is SFTP preferred over FTP?
   **A:** SFTP encrypts both credentials and data (uses SSH). FTP transmits everything in plaintext. SFTP also uses a single connection (port 22), making it firewall-friendly.

---

## 16. SMTP (Layer 7 — Application)

### Concept

**SMTP (Simple Mail Transfer Protocol)** is used to **send emails**. It's the protocol your application uses when it sends notification emails, password reset links, or alerts.

### How Email Works

```
[You compose email]
     │
     ↓
[Your Mail Client / App]
     │ SMTP (port 587)
     ↓
[Your SMTP Server (e.g., Gmail)]
     │ SMTP (port 25)
     ↓
[Recipient's SMTP Server]
     │
     ↓ (stored in mailbox)
     │
[Recipient retrieves email]
     │ IMAP (port 993) or POP3 (port 995)
     ↓
[Recipient's Mail Client]
```

### SMTP Ports

| Port | Use | Encryption |
|------|-----|-----------|
| **25** | Server-to-server relay | None (or STARTTLS) |
| **465** | SMTPS (deprecated) | Implicit TLS |
| **587** | Client submission (recommended) | STARTTLS |

### Commands

```bash
# Test SMTP connection
telnet smtp.gmail.com 587
nc -zv smtp.gmail.com 587

# Send a test email via command line
echo "Test body" | mail -s "Test Subject" user@example.com

# Check MX records (which servers handle email for a domain)
dig example.com MX
nslookup -type=MX example.com
```

### How It Connects to Your Tech Stack

| Tool | SMTP Usage |
|------|-----------|
| **Spring Boot** | `spring.mail.host=smtp.gmail.com` for sending emails |
| **Node.js** | `nodemailer` library for sending emails |
| **Jenkins** | Email notifications on build success/failure |
| **AWS SES** | Amazon Simple Email Service — managed SMTP |
| **Monitoring** | Alert emails from Prometheus/Grafana/Nagios |

### ❓ Interview Questions

1. **Q:** What is the difference between SMTP and IMAP?
   **A:** **SMTP** is for **sending** emails (push). **IMAP** is for **receiving/reading** emails (pull). They serve different parts of the email lifecycle.

2. **Q:** Why does AWS often recommend using SES instead of Gmail SMTP?
   **A:** SES is built for high-volume transactional email, offers better deliverability, integrates with AWS IAM, and doesn't have Gmail's sending limits.

---

# Part C — Troubleshooting by Layer

## How to Diagnose Which Layer Is Broken

Instead of memorizing layers, use this **diagnostic flowchart**:

```
PROBLEM: "I can't reach the website"

Step 1: Is the cable plugged in / Wi-Fi connected?
  └── NO → Layer 1 (Physical) ⚡
  └── YES ↓

Step 2: Does the interface have an IP address? (ip addr show)
  └── NO → Layer 2/3 (DHCP, Data Link) 🔗
  └── YES ↓

Step 3: Can you ping the gateway? (ping 192.168.1.1)
  └── NO → Layer 2/3 (ARP, local routing) 🔗🗺️
  └── YES ↓

Step 4: Can you ping an external IP? (ping 8.8.8.8)
  └── NO → Layer 3 (Routing, firewall blocking ICMP) 🗺️
  └── YES ↓

Step 5: Can you resolve DNS? (nslookup google.com)
  └── NO → Layer 7 (DNS configuration) 📖
  └── YES ↓

Step 6: Can you connect to the port? (nc -zv host 443)
  └── NO → Layer 4 (Firewall blocking port, service not running) 🚪
  └── YES ↓

Step 7: Does the HTTP request succeed? (curl -v https://host)
  └── NO → Layer 5-7 (TLS, HTTP error, application issue) 💻
  └── YES → It works! ✅
```

### The Troubleshooting Commands Cheat Sheet

| Layer | Check | Command |
|-------|-------|---------|
| 1 - Physical | Interface up? | `ip link show`, `ethtool eth0` |
| 2 - Data Link | ARP resolving? | `arp -a`, `tcpdump arp` |
| 3 - Network | IP routing works? | `ping gateway`, `ping 8.8.8.8`, `traceroute` |
| 3 - Network | Route exists? | `ip route show` |
| 4 - Transport | Port open? | `ss -tlnp`, `nc -zv host port`, `telnet host port` |
| 4 - Transport | Firewall? | `iptables -L`, `ufw status` |
| 7 - Application | DNS works? | `nslookup`, `dig` |
| 7 - Application | HTTP works? | `curl -v`, check logs |
| 5/6 - Session/Pres | TLS works? | `openssl s_client -connect host:443` |

---

## Real Troubleshooting Scenarios

### Scenario 1: Ping works but HTTP doesn't

```
$ ping 93.184.216.34
64 bytes from 93.184.216.34: icmp_seq=1 ttl=56 time=85.3 ms ✅

$ curl http://93.184.216.34:8080
curl: (7) Failed to connect to 93.184.216.34 port 8080: Connection refused ❌
```

**Analysis:**
- ✅ Layer 1 (Physical): Working (ping got through)
- ✅ Layer 2 (Data Link): Working (frame was delivered)
- ✅ Layer 3 (Network): Working (IP routing works, ICMP succeeds)
- ❌ **Layer 4 (Transport)**: Port 8080 is either:
  - Blocked by a firewall (AWS Security Group?)
  - No service listening on port 8080

**Fix:**
```bash
# Check if service is listening (on the server)
ss -tlnp | grep 8080

# Check firewall (AWS Security Group, iptables)
sudo iptables -L -n | grep 8080
```

---

### Scenario 2: curl works with IP but not domain name

```
$ curl http://93.184.216.34    ✅ Works!
$ curl http://example.com      ❌ "Could not resolve host: example.com"
```

**Analysis:**
- ✅ Layers 1-4: All working (connection to IP succeeds)
- ❌ **Layer 7 (Application — DNS)**: DNS resolution is failing

**Fix:**
```bash
# Check DNS server
cat /etc/resolv.conf

# Test DNS directly
dig example.com @8.8.8.8

# Temporary fix: add to /etc/hosts
echo "93.184.216.34 example.com" | sudo tee -a /etc/hosts
```

---

### Scenario 3: HTTPS fails but HTTP works

```
$ curl http://example.com      ✅ Works!
$ curl https://example.com     ❌ "SSL certificate problem: certificate has expired"
```

**Analysis:**
- ✅ Layers 1-4: Working
- ✅ Layer 7 HTTP: Working
- ❌ **Layer 6 (Presentation — TLS)**: SSL/TLS certificate has expired

**Fix:**
```bash
# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Renew certificate (Let's Encrypt)
sudo certbot renew
```

---

### Scenario 4: Docker container can't reach another container

```
$ docker exec app curl http://redis:6379
curl: (6) Could not resolve host: redis ❌
```

**Analysis:**
- The containers might be on **different Docker networks**
- Docker's internal DNS only resolves names on the **same network**

**Fix:**
```bash
# Check what networks each container is on
docker inspect app | grep NetworkMode
docker inspect redis | grep NetworkMode

# Create a shared network
docker network create mynet
docker run -d --name redis --network mynet redis
docker run -d --name app --network mynet myapp

# Now: docker exec app curl http://redis:6379 ✅
```

---

### Scenario 5: Spring Boot app can't connect to PostgreSQL in Kubernetes

```
Application error: Connection refused to postgres:5432
```

**Diagnostic Steps:**
```bash
# 1. Is the PostgreSQL pod running?
kubectl get pods -l app=postgres
# → If not running, it's an application/deployment issue

# 2. Is the PostgreSQL service created?
kubectl get svc postgres
# → If no service, create one

# 3. Can you resolve the DNS name from within the pod?
kubectl exec -it app-pod -- nslookup postgres
# → If fails, DNS issue (CoreDNS)

# 4. Can you connect to the port from within the pod?
kubectl exec -it app-pod -- nc -zv postgres 5432
# → If fails, network policy blocking, or service misconfigured

# 5. Are the pods in the same namespace?
# → If different namespaces, use: postgres.other-namespace.svc.cluster.local
```

---

## 📝 Level 1 — Summary Cheat Sheet

### OSI vs TCP/IP — Quick Reference

```
OSI 7: Application    ┐
OSI 6: Presentation   ├── TCP/IP: Application    (HTTP, DNS, SSH, TLS)
OSI 5: Session        ┘
OSI 4: Transport      ─── TCP/IP: Transport      (TCP, UDP)
OSI 3: Network        ─── TCP/IP: Internet       (IP, ICMP)
OSI 2: Data Link      ┐
OSI 1: Physical       ┘── TCP/IP: Network Access  (Ethernet, ARP, Wi-Fi)
```

### Protocol Quick Reference

| Protocol | Layer | Port | Transport | Purpose |
|----------|-------|------|-----------|---------|
| Ethernet | 2 | — | — | LAN framing |
| ARP | 2 | — | — | IP → MAC resolution |
| IP | 3 | — | — | Routing & addressing |
| ICMP | 3 | — | — | Diagnostics (ping) |
| TCP | 4 | — | — | Reliable transport |
| UDP | 4 | — | — | Fast transport |
| DNS | 7 | 53 | UDP/TCP | Name resolution |
| DHCP | 7 | 67/68 | UDP | IP assignment |
| HTTP | 7 | 80 | TCP | Web communication |
| HTTPS | 7 | 443 | TCP | Secure web |
| TLS | 5-6 | — | TCP | Encryption |
| SSH | 7 | 22 | TCP | Secure remote access |
| FTP | 7 | 21 | TCP | File transfer (legacy) |
| SFTP | 7 | 22 | TCP | Secure file transfer |
| SMTP | 7 | 25/587 | TCP | Email sending |

---

> **🎓 Congratulations!** You've completed **Level 1 — OSI & TCP/IP Models with Protocol Deep Dives**. You now understand the layered architecture of networking and can troubleshoot problems by identifying which layer is responsible. Next up: **Level 2 — Subnetting, CIDR, VLSM, NAT, and Routing**.
