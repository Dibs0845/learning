# LEVEL 0 — Networking Fundamentals

> **Learning Flow:** Concept → Why → How → Real-world example → Visualization → Command → Hands-on exercise → Troubleshooting → Interview questions

---

## Table of Contents

1. [What is a Network?](#1-what-is-a-network)
2. [Why Do Computers Need Networks?](#2-why-do-computers-need-networks)
3. [LAN, WAN, MAN, PAN](#3-lan-wan-man-pan)
4. [Client vs Server](#4-client-vs-server)
5. [Host](#5-host)
6. [Network Interface](#6-network-interface)
7. [MAC Address](#7-mac-address)
8. [IP Address](#8-ip-address)
9. [IPv4](#9-ipv4)
10. [IPv6](#10-ipv6)
11. [Private vs Public IP](#11-private-vs-public-ip)
12. [Loopback, localhost, 127.0.0.1](#12-loopback-localhost-127001)
13. [Network Ports](#13-network-ports)
14. [Protocols](#14-protocols)
15. [Packets](#15-packets)
16. [Frames](#16-frames)
17. [Bandwidth](#17-bandwidth)
18. [Latency](#18-latency)
19. [Throughput](#19-throughput)
20. [Jitter](#20-jitter)
21. [Unicast, Broadcast, Multicast](#21-unicast-broadcast-multicast)

---

## 1. What is a Network?

### Concept

A **network** is a group of two or more devices (computers, phones, servers, printers) connected together so they can **share data and resources**.

### Why?

Without networks, every computer would be an isolated island. You couldn't send an email, browse a website, or share files.

### Real-World Analogy

> 🏘️ Think of a **neighborhood** where houses are connected by roads. The roads let people visit each other, send letters, and share things. A computer network is the same — the "roads" are cables or Wi-Fi signals, and the "houses" are devices.

### Visualization

```
  [Computer A] ----cable/wifi---- [Computer B]
       |                               |
       +----------- [Printer] ---------+
       |
  [Computer C]
```

All these devices form a **network**. They can talk to each other and share the printer.

### How It Connects to Your Tech Stack

| Technology | How it uses networking |
|---|---|
| **Docker** | Containers on the same Docker network can talk to each other |
| **Kubernetes** | Pods communicate over a virtual network inside the cluster |
| **AWS** | EC2 instances in the same VPC form a network |
| **Node.js / Spring Boot** | Your apps listen on the network for incoming requests |

### Command — See Your Network

```bash
# Linux — show your network interfaces
ip addr show

# Windows
ipconfig /all

# Docker — list Docker networks
docker network ls
```

### 🧪 Hands-on Exercise

1. Open a terminal and run `ip addr show` (Linux) or `ipconfig` (Windows).
2. Identify your machine's network connections.
3. Run `docker network ls` — see the default networks Docker creates.

### ❓ Interview Questions

1. **Q:** What is a computer network?
   **A:** A computer network is a set of devices connected together to share data and resources using wired or wireless connections.

2. **Q:** Give a real-world example of networking in DevOps.
   **A:** In Kubernetes, all pods in a cluster communicate over an internal network. Services use ClusterIP networking to route traffic between pods.

---

## 2. Why Do Computers Need Networks?

### Concept

Computers need networks to:

- **Share resources** (files, printers, internet)
- **Communicate** (email, chat, video calls)
- **Access remote services** (websites, APIs, databases)
- **Collaborate** (multiple developers pushing code to Git)
- **Distribute workloads** (microservices, load balancing)

### Real-World Analogy

> 📞 Imagine a world without phones or roads. Every person would be completely isolated. Networks are the "phone lines and highways" of the computing world.

### How It Connects to Your Tech Stack

- **Jenkins** pulls code from GitHub → needs a network
- **Spring Boot** app talks to a **Redis** cache → needs a network
- **React** frontend calls **Node.js** backend API → needs a network
- **Kafka** producers send messages to brokers → needs a network

Without networking, **none of your tools would work together**.

### ❓ Interview Questions

1. **Q:** Why is networking essential in microservices architecture?
   **A:** Microservices are independent services that must communicate over the network (via HTTP/REST, gRPC, or message queues like Kafka/RabbitMQ) to function as a complete application.

---

## 3. LAN, WAN, MAN, PAN

### Concept

Networks are categorized by their **geographic size**:

| Type | Full Form | Range | Example |
|------|-----------|-------|---------|
| **PAN** | Personal Area Network | ~1-10 meters | Bluetooth earphones connected to your phone |
| **LAN** | Local Area Network | A building/office | Office Wi-Fi, home network |
| **MAN** | Metropolitan Area Network | A city | City-wide cable TV network |
| **WAN** | Wide Area Network | Countries/Continents | The Internet itself |

### Real-World Analogy

> 🗺️ Think of it as:
> - **PAN** = Your desk (personal space)
> - **LAN** = Your house/office
> - **MAN** = Your city
> - **WAN** = The entire country/world

### Visualization

```
PAN: [Phone] ---bluetooth--- [Earbuds]

LAN: [PC1] --+
     [PC2] --+-- [Switch/Router] -- [Printer]
     [PC3] --+

MAN: [Office LAN] ---fiber--- [Bank LAN] ---fiber--- [Hospital LAN]
     (all within one city)

WAN: [LAN India] ===undersea cable=== [LAN USA] ===satellite=== [LAN Europe]
```

### How It Connects to Your Tech Stack

| Scenario | Network Type |
|---|---|
| Docker containers on the same host | LAN (virtual) |
| Kubernetes pods across nodes in one data center | LAN |
| AWS VPC across availability zones in a region | LAN/MAN |
| AWS multi-region deployment | WAN |
| Your laptop connecting to production AWS servers | WAN |

### ❓ Interview Questions

1. **Q:** What is the difference between LAN and WAN?
   **A:** LAN covers a small area (building/campus) with high speed and low latency. WAN covers large geographical areas (cities/countries) and typically has higher latency and lower speeds.

2. **Q:** In AWS, is a VPC a LAN or WAN?
   **A:** A VPC acts like a virtual LAN within a region. Multi-region VPC peering would span a WAN.

---

## 4. Client vs Server

### Concept

- **Client**: The device/program that **requests** a service or resource.
- **Server**: The device/program that **provides** a service or resource.

This is the foundation of the **Client-Server model**, which powers almost everything on the internet.

### Why?

Instead of every computer storing all data, we **centralize** services on servers. Clients just connect and ask for what they need.

### Real-World Analogy

> 🍽️ **Restaurant analogy:**
> - **You** (the customer) = Client — you request food
> - **Kitchen** = Server — it prepares and serves food
> - **Waiter** = Network — carries your request and brings back the response

### Visualization

```
[Browser/React App]  ----HTTP Request---->  [Nginx/Node.js/Spring Boot Server]
     (Client)                                        (Server)
                     <---HTTP Response----
```

### How It Connects to Your Tech Stack

| Client | Server |
|--------|--------|
| React (frontend) | Node.js or Spring Boot (backend API) |
| Browser | Nginx (web server) |
| Jenkins agent | Jenkins master |
| `kubectl` CLI | Kubernetes API server |
| Docker CLI | Docker daemon |
| Kafka producer | Kafka broker |
| App code | Redis server |

### Command

```bash
# Start a simple HTTP server (acts as a SERVER)
python3 -m http.server 8080

# Now from another terminal, act as a CLIENT
curl http://localhost:8080
```

### 🧪 Hands-on Exercise

1. Start a simple HTTP server: `python3 -m http.server 8080`
2. Open your browser and go to `http://localhost:8080` — your browser is the **client**, python is the **server**.
3. In another terminal, run `curl http://localhost:8080` — curl is also a **client**.

### ❓ Interview Questions

1. **Q:** Can a machine be both a client and a server?
   **A:** Yes! For example, a Node.js backend is a **server** to the React frontend, but it is a **client** when it calls a Redis database or an external API.

2. **Q:** In Kubernetes, what is the relationship between `kubectl` and the API server?
   **A:** `kubectl` is the client that sends REST requests to the Kubernetes API server.

---

## 5. Host

### Concept

A **host** is any device connected to a network that has an **IP address** and can send or receive data.

Every client is a host. Every server is a host. Your laptop, your EC2 instance, your Docker container — all hosts.

### Real-World Analogy

> 🏠 A host is like a **house with an address** on a street. If it has an address, mail can be delivered to it. If a device has an IP, data can be sent to it.

### How It Connects to Your Tech Stack

- Each **EC2 instance** in AWS is a host
- Each **Docker container** gets its own IP — it's a host on the Docker network
- Each **Kubernetes pod** is a host on the cluster network
- `localhost` literally means "this host" — the machine you're on right now

### Command

```bash
# Show your hostname
hostname

# Show host IP
hostname -I    # Linux

# Resolve a hostname to an IP
nslookup google.com
```

### ❓ Interview Questions

1. **Q:** What is a host in networking?
   **A:** A host is any device on a network that has an IP address and can communicate with other devices. Examples: computers, servers, VMs, containers.

---

## 6. Network Interface

### Concept

A **Network Interface** is the point of connection between a device and the network. It can be:

- **Physical**: Ethernet port (RJ-45), Wi-Fi adapter
- **Virtual**: Docker's `docker0` bridge, loopback `lo`, Kubernetes `veth` interfaces

Each interface has its own **IP address** and **MAC address**.

### Real-World Analogy

> 🚪 A network interface is like a **door** on your house. You might have a front door (Ethernet), a back door (Wi-Fi), and an internal door (loopback). Each door connects you to a different "road" (network).

### Visualization

```
Your Computer
├── eth0    → Wired Ethernet (connects to office LAN)    → IP: 192.168.1.5
├── wlan0   → Wi-Fi adapter  (connects to home Wi-Fi)    → IP: 192.168.0.10
├── lo      → Loopback       (connects to itself)        → IP: 127.0.0.1
├── docker0 → Docker bridge  (connects to containers)    → IP: 172.17.0.1
└── veth123 → Virtual Ethernet (connects to a container) → IP: 172.17.0.2
```

### Command

```bash
# Linux — list all network interfaces
ip link show
ip addr show

# Show only interface names
ls /sys/class/net/

# Docker — inspect the bridge network
docker network inspect bridge

# Windows
ipconfig /all
```

### 🧪 Hands-on Exercise

1. Run `ip addr show` and identify: `lo`, `eth0` (or `ens33`), `wlan0`, `docker0`.
2. Start a Docker container: `docker run -d nginx`
3. Run `ip addr show` again — notice a new `veth*` interface appeared! That's the virtual cable to the container.

### ❓ Interview Questions

1. **Q:** What is `docker0`?
   **A:** `docker0` is a virtual bridge network interface created by Docker. It acts as a gateway for containers on the default bridge network.

2. **Q:** Can a single machine have multiple network interfaces?
   **A:** Yes. Servers commonly have multiple interfaces — e.g., one for public internet traffic and one for private backend communication.

---

## 7. MAC Address

### Concept

A **MAC (Media Access Control) address** is a **unique hardware identifier** permanently assigned to every network interface card (NIC). It operates at **Layer 2 (Data Link Layer)** of the OSI model.

- Format: `AA:BB:CC:DD:EE:FF` (6 bytes, 48 bits)
- It's "burned in" to the hardware by the manufacturer
- Used for **local network communication** (within a LAN)

### Why?

When data travels within a local network, devices use MAC addresses to identify **who is who** on that network segment.

### Real-World Analogy

> 🆔 A MAC address is like your **Aadhaar card / SSN number** — it's unique and permanently assigned to you. No two people have the same one. Similarly, no two network interfaces have the same MAC address.

### Visualization

```
[Computer A: MAC=AA:11:22:33:44:55] ---LAN--- [Computer B: MAC=BB:66:77:88:99:00]
                    |
             [Switch] (uses MAC addresses to forward frames)
                    |
[Computer C: MAC=CC:AB:CD:EF:01:23]
```

The switch maintains a **MAC address table** to know which device is on which port.

### Command

```bash
# Linux — show MAC addresses
ip link show
# Look for "link/ether" — that's the MAC address

# Windows
ipconfig /all
# Look for "Physical Address"

# Show ARP table (IP-to-MAC mappings)
arp -a
```

### 🧪 Hands-on Exercise

1. Run `ip link show` — find the MAC address of your `eth0`/`wlan0` interface.
2. Run `arp -a` — see the IP-to-MAC mappings of devices on your local network.
3. Run a Docker container and check its MAC: `docker inspect <container_id> | grep MacAddress`

### ❓ Interview Questions

1. **Q:** What is the difference between a MAC address and an IP address?
   **A:** MAC address is a hardware-level identifier (Layer 2), permanent and used within a LAN. IP address is a logical identifier (Layer 3), can change, and is used for routing across networks.

2. **Q:** Can MAC addresses be changed?
   **A:** Technically, yes — this is called **MAC spoofing**. Docker and virtual machines regularly assign virtual MAC addresses.

---

## 8. IP Address

### Concept

An **IP (Internet Protocol) address** is a **logical address** assigned to a device on a network. It's used to **identify and locate** devices across networks (not just within a LAN like MAC).

- Think of it as a **postal address** for your device
- Can be **static** (permanent) or **dynamic** (assigned by DHCP)
- Operates at **Layer 3 (Network Layer)**

### Real-World Analogy

> 📬 If MAC address is your Aadhaar/SSN (who you are permanently), then IP address is your **home postal address** (where you currently live). You can move houses (get a new IP), but your Aadhaar stays the same.

### Visualization

```
[Your Laptop]           [Google Server]
IP: 192.168.1.5    →    IP: 142.250.182.14
MAC: AA:BB:CC:..        MAC: XX:YY:ZZ:..

Your router knows: "To reach 142.250.182.14, send the packet out to the internet."
```

### Two Versions

| Version | Bits | Example | Total Addresses |
|---------|------|---------|-----------------|
| **IPv4** | 32-bit | `192.168.1.1` | ~4.3 billion |
| **IPv6** | 128-bit | `2001:0db8::1` | ~340 undecillion |

### Command

```bash
# Show your IP addresses
ip addr show         # Linux
ipconfig             # Windows

# Find the IP of a website
nslookup google.com
dig google.com       # Linux

# Check your public IP
curl ifconfig.me
```

### ❓ Interview Questions

1. **Q:** What happens when two devices have the same IP on a network?
   **A:** An **IP conflict** occurs. Both devices will have intermittent connectivity issues, and the network may generate ARP conflicts.

---

## 9. IPv4

### Concept

**IPv4 (Internet Protocol version 4)** is the most widely used version of IP addressing.

- **32 bits** long, written as **4 octets** separated by dots
- Each octet ranges from **0 to 255**
- Example: `192.168.1.100`

### Structure

```
  192   .   168   .    1    .   100
[octet1] [octet2] [octet3] [octet4]
   8 bits + 8 bits + 8 bits + 8 bits = 32 bits
```

### IPv4 Address Classes (Classical)

| Class | Range | Default Subnet Mask | Use |
|-------|-------|---------------------|-----|
| **A** | 1.0.0.0 – 126.255.255.255 | 255.0.0.0 (/8) | Large networks |
| **B** | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | Medium networks |
| **C** | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | Small networks |
| **D** | 224.0.0.0 – 239.255.255.255 | — | Multicast |
| **E** | 240.0.0.0 – 255.255.255.255 | — | Experimental |

### Real-World Analogy

> 🏷️ An IPv4 address is like a **phone number** — structured, limited in quantity, and used to reach a specific device.

### Command

```bash
# See your IPv4 address
ip -4 addr show

# Ping an IPv4 address
ping 8.8.8.8
```

### ❓ Interview Questions

1. **Q:** Why are we running out of IPv4 addresses?
   **A:** IPv4 has only ~4.3 billion addresses. With billions of devices (phones, IoT, servers), we've exhausted the available pool. Solutions include NAT, CIDR, and migrating to IPv6.

2. **Q:** What subnet does Docker use by default?
   **A:** Docker typically uses `172.17.0.0/16` for its default bridge network.

---

## 10. IPv6

### Concept

**IPv6 (Internet Protocol version 6)** is the successor to IPv4, designed to solve the address shortage.

- **128 bits** long
- Written as **8 groups of 4 hex digits**, separated by colons
- Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Shortened: `2001:db8:85a3::8a2e:370:7334`

### Why IPv6?

- **340 undecillion addresses** (340 × 10³⁶) — enough for every grain of sand on Earth
- Built-in security (IPSec)
- No need for NAT
- Better routing efficiency

### Real-World Analogy

> 📱 IPv4 is like **old 7-digit phone numbers** — we ran out. IPv6 is like switching to **15-digit numbers** — we'll never run out.

### Visualization

```
IPv4:  192.168.1.1                          (32 bits)
IPv6:  2001:0db8:85a3:0000:0000:8a2e:0370:7334  (128 bits)
```

### Command

```bash
# See your IPv6 address
ip -6 addr show

# Ping an IPv6 address
ping6 ::1    # ping localhost via IPv6

# Check if a website supports IPv6
dig AAAA google.com
```

### How It Connects to Your Tech Stack

- **Kubernetes** supports dual-stack (IPv4 + IPv6) networking
- **AWS VPCs** can be configured with IPv6 CIDR blocks
- **Docker** supports IPv6 networks (must be enabled in daemon config)

### ❓ Interview Questions

1. **Q:** What is the IPv6 loopback address?
   **A:** `::1` (equivalent to `127.0.0.1` in IPv4).

2. **Q:** Can IPv4 and IPv6 coexist?
   **A:** Yes, through **dual-stack** (both protocols run simultaneously), **tunneling** (IPv6 packets inside IPv4), or **translation** (NAT64).

---

## 11. Private vs Public IP

### Concept

| Type | Description | Routable on Internet? | Example |
|------|-------------|----------------------|---------|
| **Private IP** | Used within a local network | ❌ No | `192.168.1.5`, `10.0.0.1`, `172.16.0.1` |
| **Public IP** | Unique address on the internet | ✅ Yes | `52.86.123.45` |

### Private IP Ranges (RFC 1918)

| Class | Range | CIDR |
|-------|-------|------|
| A | `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` |
| B | `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` |
| C | `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

### Why?

We don't have enough public IPv4 addresses for every device. So devices inside a network use **private IPs**, and a **router/NAT** translates them to a single public IP when accessing the internet.

### Real-World Analogy

> 🏢 Private IP is like your **room number inside a hotel** (Room 304). Public IP is the **hotel's street address**. The outside world sends mail to the hotel (public IP), and the hotel reception (router/NAT) forwards it to your room (private IP).

### Visualization

```
[Your Laptop: 192.168.1.5]  --+
[Your Phone:  192.168.1.6]  --+-- [Router: Private=192.168.1.1 / Public=49.36.12.89] --- INTERNET
[Smart TV:    192.168.1.7]  --+
```

All devices share the **one public IP** (`49.36.12.89`) via NAT.

### How It Connects to Your Tech Stack

| Technology | IP Type |
|---|---|
| Docker containers (bridge network) | Private IPs (e.g., `172.17.0.2`) |
| Kubernetes pods | Private IPs (cluster-internal) |
| AWS EC2 (private subnet) | Private IP |
| AWS EC2 (public subnet + EIP) | Public IP |
| AWS Load Balancer (ALB/NLB) | Public IP |
| `localhost` / `127.0.0.1` | Loopback (special private) |

### Command

```bash
# See your PRIVATE IP
ip addr show        # Linux
ipconfig            # Windows

# See your PUBLIC IP
curl ifconfig.me
curl ipinfo.io/ip
```

### 🧪 Hands-on Exercise

1. Find your private IP: `ip addr show` — look for `192.168.x.x` or `10.x.x.x`
2. Find your public IP: `curl ifconfig.me`
3. Compare them — they're different! Your router is doing NAT.
4. Check a Docker container's IP: `docker inspect <container> | grep IPAddress` — it'll be a private IP like `172.17.0.2`

### ❓ Interview Questions

1. **Q:** Can two devices on different networks have the same private IP?
   **A:** Yes! `192.168.1.5` can exist in your home network AND in an office network. Private IPs are only unique within their own network.

2. **Q:** How does an AWS EC2 instance in a private subnet access the internet?
   **A:** Through a **NAT Gateway** in a public subnet. The NAT Gateway has a public IP and translates the private IP traffic.

---

## 12. Loopback, localhost, 127.0.0.1

### Concept

| Term | What It Is |
|------|-----------|
| **Loopback** | A virtual network interface that routes traffic **back to the same machine** |
| **localhost** | A hostname that resolves to the loopback IP address |
| **127.0.0.1** | The IPv4 loopback IP address |
| **::1** | The IPv6 loopback IP address |

When you send data to `127.0.0.1` or `localhost`, the data **never leaves your machine**. It goes out of the application, into the network stack, and loops right back.

### Why?

- **Testing**: Run a server and connect to it locally without needing a real network
- **Development**: Your React app at `localhost:3000` calls your Node.js API at `localhost:8080`
- **Security**: Bind services to `127.0.0.1` so they're only accessible locally

### Real-World Analogy

> 💌 Loopback is like **writing a letter to yourself**. You put it in your mailbox, and the postman delivers it right back to you. The letter never goes to anyone else.

### Visualization

```
[Your App] → sends to 127.0.0.1:8080
     ↓
[Network Stack] → "Oh, 127.0.0.1? That's ME!"
     ↓
[Loops back to Your App listening on port 8080]
```

### How It Connects to Your Tech Stack

```bash
# React dev server
npm start
# → http://localhost:3000

# Node.js/Express
node server.js
# → http://localhost:8080

# Spring Boot
mvn spring-boot:run
# → http://localhost:8080

# Redis
redis-cli -h 127.0.0.1 -p 6379

# Jenkins
# → http://localhost:8080
```

### ⚠️ Important Gotcha: Docker and localhost

```bash
# Inside a Docker container, 127.0.0.1 refers to THE CONTAINER, not the host!
# To reach the host from inside a Docker container:
# Use: host.docker.internal (Docker Desktop)
# Or: 172.17.0.1 (Linux, Docker bridge gateway)
```

This is a **very common source of bugs** when containerizing apps.

### Command

```bash
# Ping loopback
ping 127.0.0.1
ping localhost

# See the loopback interface
ip addr show lo

# Check what's running on localhost
ss -tlnp           # Linux
netstat -tlnp       # Linux
netstat -an         # Windows
```

### 🧪 Hands-on Exercise

1. Run `ping 127.0.0.1` — notice the response time is ~0ms (it never leaves your machine).
2. Start a simple server: `python3 -m http.server 9000`
3. Open browser: `http://localhost:9000` — you're using the loopback interface!
4. Try the Docker gotcha: Run `docker run -it alpine ping host.docker.internal`

### ❓ Interview Questions

1. **Q:** If a Spring Boot app is running inside a Docker container and trying to connect to Redis at `localhost:6379`, will it work?
   **A:** ❌ No. `localhost` inside the container refers to the container itself, not the host. You need to use the container name (on a shared Docker network) or `host.docker.internal`.

2. **Q:** What is the full loopback range in IPv4?
   **A:** The entire `127.0.0.0/8` range (127.0.0.1 to 127.255.255.254) is reserved for loopback.

---

## 13. Network Ports

### Concept

A **port** is a **logical number (0–65535)** that identifies a specific **process or service** on a host.

- IP address identifies the **machine**
- Port identifies the **application** on that machine

Think of it this way:

```
IP Address = Building Address
Port       = Apartment Number
```

### Port Ranges

| Range | Name | Use |
|-------|------|-----|
| **0 – 1023** | Well-Known Ports | Reserved for standard services (HTTP, SSH, DNS) |
| **1024 – 49151** | Registered Ports | Used by applications (MySQL, Redis, Kafka) |
| **49152 – 65535** | Dynamic/Ephemeral | Temporary ports for client connections |

### Common Ports You MUST Know

| Port | Service | Your Tech Stack Connection |
|------|---------|---------------------------|
| **22** | SSH | Connecting to EC2 instances, Git over SSH |
| **53** | DNS | Resolving domain names |
| **80** | HTTP | Nginx, Apache serving web content |
| **443** | HTTPS | Secure web traffic, TLS |
| **3000** | — | React dev server |
| **3306** | MySQL | Database |
| **5432** | PostgreSQL | Database |
| **6379** | Redis | Caching, session store |
| **5672** | RabbitMQ | Message queue (AMQP) |
| **8080** | — | Spring Boot, Jenkins, Tomcat |
| **8443** | — | HTTPS alternate |
| **9092** | Kafka | Message streaming |
| **27017** | MongoDB | Database |

### Real-World Analogy

> 🏢 An IP address is like a **mall's address**. A port is like the **shop number** inside the mall.
> - Shop 80 = Web store (HTTP)
> - Shop 443 = Premium store (HTTPS)
> - Shop 22 = Security office (SSH)
> - Shop 6379 = Storage room (Redis)

### Visualization

```
Server: 192.168.1.10
├── Port 22    → SSH daemon
├── Port 80    → Nginx (HTTP)
├── Port 443   → Nginx (HTTPS)
├── Port 3000  → React app
├── Port 8080  → Spring Boot API
├── Port 6379  → Redis
├── Port 9092  → Kafka broker
└── Port 5432  → PostgreSQL
```

### Command

```bash
# See which ports are in use (Linux)
ss -tlnp
netstat -tlnp

# See which ports are in use (Windows)
netstat -an | findstr LISTENING

# Check if a specific port is open on a remote host
nc -zv 192.168.1.10 80          # Linux
Test-NetConnection -Port 80 -ComputerName 192.168.1.10   # PowerShell

# Docker — map host port to container port
docker run -p 8080:80 nginx
# Host port 8080 → Container port 80
```

### ⚠️ Behind the Scenes: Docker Port Mapping

```
docker run -p 3000:80 nginx
```

```
[Browser] → http://localhost:3000
    ↓
[Host Port 3000] → Docker forwards → [Container Port 80] → Nginx
```

This is why in `docker-compose.yml`, you write:
```yaml
ports:
  - "3000:80"   # host:container
```

### 🧪 Hands-on Exercise

1. Run `ss -tlnp` or `netstat -an` to see all listening ports on your machine.
2. Start Nginx in Docker: `docker run -d -p 8080:80 nginx`
3. Access `http://localhost:8080` — your request hits host port 8080, which Docker routes to container port 80.
4. Run `ss -tlnp | grep 8080` — see Docker listening on port 8080.

### ❓ Interview Questions

1. **Q:** What happens if two applications try to use the same port?
   **A:** The second application will get a "port already in use" error (`EADDRINUSE`). Only one process can bind to a specific port on a specific interface at a time.

2. **Q:** What is the significance of `-p 8080:80` in Docker?
   **A:** It maps port 8080 on the host to port 80 inside the container. External traffic to host:8080 is forwarded to the container's port 80.

3. **Q:** Why should you NOT expose port 6379 (Redis) to the public internet?
   **A:** Redis has no authentication by default. Exposing it publicly is a major security risk — anyone could read/modify your data.

---

## 14. Protocols

### Concept

A **protocol** is a **set of rules** that defines how data is formatted, transmitted, and received over a network. Without protocols, devices wouldn't understand each other.

### Real-World Analogy

> 🗣️ A protocol is like a **language**. If you speak Hindi and the other person speaks Japanese, you can't communicate. You both need to agree on a common language (protocol). In networking, both the sender and receiver must follow the same protocol.

### Key Protocols You Must Know

| Protocol | Layer | Port | Purpose | Tech Stack Connection |
|----------|-------|------|---------|----------------------|
| **HTTP** | Application | 80 | Web communication | React, Node.js, Spring Boot, Nginx |
| **HTTPS** | Application | 443 | Secure web communication | All production apps |
| **TCP** | Transport | — | Reliable, ordered delivery | Most internet traffic |
| **UDP** | Transport | — | Fast, unreliable delivery | DNS, video streaming, gaming |
| **IP** | Network | — | Routing packets across networks | Everything |
| **DNS** | Application | 53 | Translates domain names to IPs | Every URL you open |
| **SSH** | Application | 22 | Secure remote access | EC2, Git, server management |
| **FTP/SFTP** | Application | 21/22 | File transfer | Deploying files |
| **SMTP** | Application | 25/587 | Sending email | Notification systems |
| **AMQP** | Application | 5672 | Message queuing | RabbitMQ |
| **ICMP** | Network | — | Diagnostics (ping) | Troubleshooting |
| **ARP** | Data Link | — | Resolves IP → MAC | LAN communication |
| **DHCP** | Application | 67/68 | Auto-assigns IP addresses | Every device on your network |

### Visualization: How Protocols Stack Together

```
When you open https://example.com:

[Browser]
    ↓
[HTTP/HTTPS]  — Application layer protocol (what to say)
    ↓
[TLS/SSL]     — Security layer (encrypt the message)
    ↓
[TCP]         — Transport layer (ensure reliable delivery)
    ↓
[IP]          — Network layer (address and route the packet)
    ↓
[Ethernet]    — Data Link layer (send over the physical wire)
    ↓
[Physical]    — Actual electrical signals / light / radio waves
```

### Command

```bash
# See TCP connections
ss -tn

# See UDP connections
ss -un

# Test DNS protocol
nslookup google.com
dig google.com

# Test HTTP protocol
curl -v http://example.com

# Test ICMP protocol (ping)
ping google.com

# Trace the route (uses ICMP/UDP)
traceroute google.com     # Linux
tracert google.com        # Windows
```

### ❓ Interview Questions

1. **Q:** What is the difference between TCP and UDP?
   **A:** TCP is reliable, ordered, and connection-oriented (uses handshake). UDP is unreliable, unordered, and connectionless (no handshake). TCP is used for web/API traffic. UDP is used for DNS, streaming, and gaming.

2. **Q:** What protocol does `ping` use?
   **A:** ICMP (Internet Control Message Protocol).

3. **Q:** Which protocol does Kafka use?
   **A:** Kafka uses its own binary protocol over TCP on port 9092.

---

## 15. Packets

### Concept

A **packet** is a small unit of data transmitted over a network. When you send a large file or web page, it gets **broken into many small packets**, each one individually routed across the network, and **reassembled at the destination**.

### Why Packets?

- **Efficiency**: Multiple users can share the same network (packet switching)
- **Reliability**: If one packet is lost, only that packet is resent, not the entire file
- **Routing**: Different packets can take different paths to the destination

### Structure of a Packet

```
+------------------+-------------------+------------------+
|    IP Header     |    TCP Header     |    Data/Payload  |
| (src IP, dst IP) | (src port, dst    | (actual content  |
|                  |  port, seq #)     |  you're sending) |
+------------------+-------------------+------------------+
|<---- Header ---->|<---- Header ----->|<---- Payload --->|
```

### Real-World Analogy

> 📦 Imagine you want to send a **large bookshelf** from Delhi to Mumbai. You can't send it as one piece. So you:
> 1. **Disassemble** it into numbered boxes (packets)
> 2. Each box has a **label** (header) with source, destination, and sequence number
> 3. Boxes may travel by **different trucks/routes**
> 4. At Mumbai, boxes are **reassembled** in order

### Command

```bash
# Capture packets (Linux, requires root)
sudo tcpdump -i eth0 -c 10

# Capture packets to/from a specific host
sudo tcpdump -i any host 8.8.8.8

# Count packets to a host
ping -c 5 google.com

# Use Wireshark (GUI tool) for detailed packet analysis
wireshark
```

### 🧪 Hands-on Exercise

1. Open two terminals.
2. Terminal 1: `sudo tcpdump -i any -c 20`
3. Terminal 2: `curl http://example.com`
4. Watch the packets flow in Terminal 1! You'll see the TCP handshake (SYN, SYN-ACK, ACK) and HTTP data.

### ❓ Interview Questions

1. **Q:** What is the maximum size of a packet?
   **A:** The default MTU (Maximum Transmission Unit) for Ethernet is **1500 bytes**. Larger data is fragmented into multiple packets.

2. **Q:** What happens if a packet is lost?
   **A:** With TCP, the sender detects the loss (via missing ACK) and **retransmits** the packet. With UDP, the packet is simply lost — no retransmission.

---

## 16. Frames

### Concept

A **frame** is the data unit at **Layer 2 (Data Link Layer)**. It wraps a packet with **MAC address information** for delivery within a local network segment.

### Packet vs Frame

```
Layer 3 (Network):    [IP Header | TCP Header | Data]  → called a PACKET
Layer 2 (Data Link):  [MAC Header | PACKET | Trailer]  → called a FRAME
```

### Structure of a Frame

```
+----------------+----------------+---------------------------+----------+
| Dest MAC Addr  | Src MAC Addr   | Payload (the IP Packet)   | Checksum |
| (6 bytes)      | (6 bytes)      |                           | (FCS)    |
+----------------+----------------+---------------------------+----------+
```

### Real-World Analogy

> ✉️ A **packet** is like a letter with your name and address (IP). A **frame** is like putting that letter into an **envelope** with the local delivery address (MAC). The envelope (frame) changes at each post office (router), but the letter inside (packet) stays the same.

### Visualization: How a Frame Changes at Each Hop

```
Your PC (192.168.1.5) → Router → Google (142.250.182.14)

Step 1: Your PC creates frame:
  [Dest MAC: Router's MAC] [Src MAC: Your MAC] [Packet: Dest IP=142.250.182.14]

Step 2: Router receives frame, strips it, creates NEW frame:
  [Dest MAC: Next Router's MAC] [Src MAC: Router's MAC] [Packet: same]

→ The PACKET (IP addresses) stays the same. The FRAME (MAC addresses) changes at each hop.
```

### Command

```bash
# See frames being sent/received (need root)
sudo tcpdump -i eth0 -e -c 10
# The -e flag shows Ethernet frame headers (MAC addresses)
```

### ❓ Interview Questions

1. **Q:** What is the difference between a packet and a frame?
   **A:** A packet operates at Layer 3 (has IP addresses). A frame operates at Layer 2 (has MAC addresses). A frame encapsulates a packet for local delivery.

2. **Q:** Do MAC addresses change hop by hop or stay the same?
   **A:** MAC addresses change at each hop (each router creates a new frame). IP addresses remain the same end-to-end.

---

## 17. Bandwidth

### Concept

**Bandwidth** is the **maximum amount of data** that can be transmitted over a network connection in a given time. It's measured in **bits per second (bps)**.

| Unit | Value |
|------|-------|
| Kbps | 1,000 bits/sec |
| Mbps | 1,000,000 bits/sec |
| Gbps | 1,000,000,000 bits/sec |

### Why?

Bandwidth determines how "wide" the road is. More bandwidth = more data can flow at once.

### Real-World Analogy

> 🛣️ Bandwidth is like the **number of lanes on a highway**. A 4-lane highway (high bandwidth) can carry more cars (data) at the same time than a single-lane road (low bandwidth). But bandwidth doesn't tell you how fast the cars are going — that's latency/speed.

### How It Connects to Your Tech Stack

- **AWS EC2** instance types have different network bandwidth (e.g., `t3.micro` = up to 5 Gbps)
- **Kafka/RabbitMQ** throughput depends heavily on available bandwidth
- **Docker image pulls** from registries are limited by bandwidth
- **CI/CD pipelines** (Jenkins) downloading dependencies require bandwidth

### Command

```bash
# Test your bandwidth
speedtest-cli         # Install: pip install speedtest-cli

# Check interface speed
ethtool eth0          # Linux — shows max bandwidth
cat /sys/class/net/eth0/speed   # Speed in Mbps

# Monitor real-time bandwidth usage
iftop                 # Linux (install: apt install iftop)
nload                 # Linux (install: apt install nload)
```

### ❓ Interview Questions

1. **Q:** Is more bandwidth always better?
   **A:** Not necessarily. If latency is high, even high bandwidth won't make real-time communication feel fast. Bandwidth helps with **bulk data transfer**, while latency affects **responsiveness**.

---

## 18. Latency

### Concept

**Latency** is the **time it takes for data to travel from source to destination**. It's measured in **milliseconds (ms)**.

### Types of Latency

| Type | Description |
|------|-------------|
| **Network latency** | Time for a packet to travel across the network |
| **Application latency** | Time for an app to process a request |
| **Round-trip time (RTT)** | Time for data to go + come back |

### Real-World Analogy

> 🏎️ Latency is like the **time it takes for a car to drive from your house to the office**. Even if the highway has 10 lanes (high bandwidth), if the office is 100 km away, it still takes time to get there.

### Why Latency Matters

- **Low latency** (~1-5 ms): Redis cache, local database
- **Medium latency** (~20-100 ms): Same-region API calls
- **High latency** (~200-500 ms): Cross-continent requests
- **Very high latency** (~500+ ms): Satellite links, bad user experience

### How It Connects to Your Tech Stack

| Scenario | Typical Latency |
|----------|----------------|
| `localhost` to `localhost` | < 1 ms |
| Same Docker network | < 1 ms |
| Same AWS AZ | 1-2 ms |
| Cross AZ (same region) | 1-5 ms |
| Cross region (e.g., us-east to eu-west) | 100-200 ms |
| India to US | 200-300 ms |
| Redis cache hit | < 1 ms |
| Database query | 5-50 ms |
| External API call | 50-500 ms |

### Command

```bash
# Measure latency to a host
ping google.com
# Look at the "time=" value — that's RTT

# Measure latency to localhost
ping 127.0.0.1

# Trace latency at each hop
traceroute google.com     # Linux
tracert google.com        # Windows

# Measure HTTP latency
curl -o /dev/null -s -w "Total time: %{time_total}s\n" https://example.com
```

### ❓ Interview Questions

1. **Q:** Why does AWS recommend keeping microservices in the same region?
   **A:** Cross-region communication adds 100-200+ ms latency per request. In a microservices chain (A → B → C → D), latencies multiply, leading to poor user experience.

2. **Q:** How does Redis reduce latency?
   **A:** Redis stores data in-memory (no disk I/O) and is typically deployed close to the application, providing sub-millisecond response times compared to database queries.

---

## 19. Throughput

### Concept

**Throughput** is the **actual amount of data successfully transferred** over a network in a given time. While bandwidth is the **theoretical maximum**, throughput is the **real-world performance**.

```
Throughput ≤ Bandwidth (always)
```

### Why Is Throughput Different from Bandwidth?

Throughput is affected by:
- Packet loss
- Network congestion
- Protocol overhead (headers)
- Latency
- Server processing time

### Real-World Analogy

> 🚰 Bandwidth is the **diameter of a water pipe** (maximum capacity). Throughput is **how much water actually flows** through it. If the pipe is clogged (congestion) or has leaks (packet loss), the actual flow (throughput) is less than the pipe size (bandwidth).

### Visualization

```
Bandwidth:  100 Mbps (maximum capacity)
Congestion: -20 Mbps
Overhead:   -5 Mbps
Packet loss: -10 Mbps
─────────────────────
Throughput:  65 Mbps (actual transfer rate)
```

### Command

```bash
# Measure throughput between two hosts
iperf3 -s                # Start server on host A
iperf3 -c <host-A-IP>    # Connect from host B

# Measure download throughput
wget -O /dev/null http://speedtest.tele2.net/10MB.zip

# Monitor per-process throughput
nethogs       # Linux
```

### ❓ Interview Questions

1. **Q:** If your AWS EC2 bandwidth is 1 Gbps but you're only getting 200 Mbps throughput, what could be wrong?
   **A:** Possible causes: network congestion, TCP window size limits, packet loss, application bottleneck, disk I/O limits, or cross-region transfer with high latency.

---

## 20. Jitter

### Concept

**Jitter** is the **variation in latency** over time. It measures how **consistent** the packet delivery times are.

- Low jitter = packets arrive at consistent intervals ✅
- High jitter = packets arrive at irregular intervals ❌

### Why Does Jitter Matter?

- **Video calls** (Zoom/Teams): High jitter = choppy video, audio glitches
- **Real-time gaming**: High jitter = lag spikes
- **VoIP**: High jitter = garbled voice
- **Streaming** (Netflix): Buffered, so jitter is less impactful

### Real-World Analogy

> 🚌 Imagine a bus that's supposed to arrive **every 10 minutes**:
> - **Low jitter**: Bus arrives at 10, 10, 10, 10 min intervals → reliable
> - **High jitter**: Bus arrives at 5, 15, 3, 20 min intervals → unpredictable

### Visualization

```
Low Jitter:
Packet 1: 20ms
Packet 2: 21ms
Packet 3: 20ms
Packet 4: 22ms
→ Variation: ~1-2ms ✅

High Jitter:
Packet 1: 20ms
Packet 2: 85ms
Packet 3: 12ms
Packet 4: 150ms
→ Variation: ~130ms ❌
```

### Command

```bash
# See jitter in ping output
ping -c 20 google.com
# Look at the final line: "min/avg/max/mdev"
# mdev = mean deviation ≈ jitter

# Example output:
# rtt min/avg/max/mdev = 18.234/20.456/25.789/2.345 ms
#                         ↑                        ↑
#                     min latency              jitter (mdev)
```

### ❓ Interview Questions

1. **Q:** How would you troubleshoot high jitter in a VoIP application?
   **A:** Check for network congestion, ensure QoS (Quality of Service) is configured, check for competing bandwidth-heavy processes, and consider using a dedicated VLAN for voice traffic.

---

## 21. Unicast, Broadcast, Multicast

### Concept

These describe **how data is sent to recipients** on a network:

| Type | Description | Recipients | Example |
|------|-------------|------------|---------|
| **Unicast** | One-to-one | Single device | You sending an HTTP request to google.com |
| **Broadcast** | One-to-all | Every device on the network | ARP request "Who has IP 192.168.1.1?" |
| **Multicast** | One-to-many | A specific group of devices | Live video streaming to subscribers |

### Real-World Analogy

> 📢 
> - **Unicast** = Making a **phone call** — you talk to one specific person
> - **Broadcast** = Using a **loudspeaker in a mall** — everyone hears the announcement
> - **Multicast** = A **WhatsApp group** — only group members receive the message

### Visualization

```
Unicast:
  [Sender] ──────→ [Receiver]

Broadcast:
  [Sender] ──────→ [Device A]
           ──────→ [Device B]
           ──────→ [Device C]
           ──────→ [Device D]
           (ALL devices on the network)

Multicast:
  [Sender] ──────→ [Subscriber A]
           ──────→ [Subscriber C]
           (only subscribed devices)
```

### How It Connects to Your Tech Stack

| Type | Tech Stack Example |
|------|-------------------|
| **Unicast** | HTTP request from React → Node.js API |
| **Unicast** | SSH to an EC2 instance |
| **Broadcast** | ARP resolution on a LAN |
| **Broadcast** | DHCP discover (device asking "Is there a DHCP server?") |
| **Multicast** | Kubernetes service discovery (mDNS) |
| **Multicast** | Docker Swarm node discovery |
| **Multicast** | Live video streaming in a CDN |

### Broadcast Address

- The broadcast address for `192.168.1.0/24` is `192.168.1.255`
- Sending to this address reaches ALL devices in that subnet

### Command

```bash
# See ARP broadcast (watch for "who-has" requests)
sudo tcpdump -i eth0 arp

# Send a broadcast ping (Linux)
ping -b 192.168.1.255

# See multicast groups your machine belongs to
ip maddr show       # Linux
netstat -gn          # Linux
```

### ❓ Interview Questions

1. **Q:** Why is broadcast traffic a problem in large networks?
   **A:** Every device must process every broadcast packet, even if the broadcast isn't relevant to it. This wastes CPU and bandwidth. This is why large networks are divided into **VLANs** — to limit broadcast domains.

2. **Q:** What type of communication does HTTP use?
   **A:** Unicast. HTTP is always a one-to-one communication between client and server.

3. **Q:** In Kubernetes, how do services discover pods?
   **A:** Through the Kubernetes DNS service (CoreDNS) using unicast DNS queries, and `kube-proxy` which manages routing rules.

---

## 📝 Level 0 — Summary Cheat Sheet

| Concept | One-Line Definition |
|---------|-------------------|
| Network | Connected devices sharing resources |
| LAN/WAN | Small area vs large area network |
| Client/Server | Requester vs provider of service |
| Host | Any device with an IP on a network |
| Network Interface | Physical/virtual connection point (eth0, wlan0, docker0) |
| MAC Address | Permanent hardware ID (Layer 2) |
| IP Address | Logical network address (Layer 3) |
| IPv4 | 32-bit address (192.168.1.1) |
| IPv6 | 128-bit address (2001:db8::1) |
| Private IP | Internal, non-routable (10.x, 192.168.x) |
| Public IP | Internet-routable, globally unique |
| Loopback | Traffic loops back to yourself (127.0.0.1) |
| Port | Identifies a service on a host (0-65535) |
| Protocol | Rules for communication (HTTP, TCP, UDP) |
| Packet | Data unit at Layer 3 (has IPs) |
| Frame | Data unit at Layer 2 (has MACs) |
| Bandwidth | Max data capacity of a link |
| Latency | Time for data to travel (delay) |
| Throughput | Actual data transferred |
| Jitter | Variation in latency |
| Unicast | One-to-one communication |
| Broadcast | One-to-all communication |
| Multicast | One-to-group communication |

---

## 🔥 What Happens Behind the Scenes When You Open `https://example.com`?

Now that you know all Level 0 concepts, let's trace the full journey:

```
Step 1: You type https://example.com in the browser

Step 2: DNS Resolution (Protocol: DNS, Port: 53, Unicast)
  → Browser asks: "What is the IP of example.com?"
  → DNS server responds: "93.184.216.34"

Step 3: TCP Connection (Protocol: TCP, Three-Way Handshake)
  → Your machine sends SYN to 93.184.216.34:443
  → Server responds SYN-ACK
  → Your machine sends ACK
  → Connection established! ✅

Step 4: TLS Handshake (Protocol: TLS/SSL, Port: 443)
  → Negotiate encryption (key exchange, certificates)
  → Encrypted tunnel is established 🔒

Step 5: HTTP Request (Protocol: HTTPS over TCP)
  → Browser sends: GET / HTTP/1.1 Host: example.com
  → This request is a PACKET at Layer 3 (has IPs)
  → Wrapped in a FRAME at Layer 2 (has MACs)
  → Travels through your NETWORK INTERFACE (eth0/wlan0)

Step 6: Routing
  → Your router (using the PRIVATE→PUBLIC IP NAT)
  → Sends the packet across the WAN (Internet)
  → Multiple HOPS (each router changes the FRAME but keeps the PACKET)

Step 7: Server receives request
  → Nginx/Apache on the SERVER (HOST) at Port 443
  → Processes the request, generates HTML response

Step 8: Response travels back
  → Same path in reverse
  → Arrives at your browser
  → Browser renders the page 🎉
```

### Networking Concepts Used in This Single Request:

✅ Network, LAN, WAN, Client, Server, Host, Network Interface, MAC Address, IP Address (IPv4), Private/Public IP, Ports (53, 443), Protocols (DNS, TCP, TLS, HTTP), Packets, Frames, Bandwidth, Latency, Unicast

---

> **🎓 Congratulations!** You've completed **Level 0 — Networking Fundamentals**. You now understand the building blocks that every other networking concept is built upon. Next up: **Level 1 — OSI Model, TCP/IP Model, and Deep Dive into Protocols**.
