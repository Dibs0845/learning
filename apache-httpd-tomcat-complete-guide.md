# Apache HTTPD & Apache Tomcat: Complete Enterprise Production Guide & Practical Training

**Author/Mentor**: Senior Linux System Administrator, DevOps Architect & Production Troubleshooting Engineer  
**Target Environment**: Ubuntu 22.04 / 24.04 LTS | AWS EC2 / Linux VM | Java 17/21 | Apache HTTPD 2.4 | Apache Tomcat 10/11  

---

## Table of Contents
1. [Curriculum Overview & Training Framework](#curriculum-overview--training-framework)
2. [Module 1: The End-to-End Enterprise Request Lifecycle](#module-1-the-end-to-end-enterprise-request-lifecycle)
   - [1.1 Enterprise Multi-Tier Architecture Diagram](#11-enterprise-multi-tier-architecture-diagram)
   - [1.2 Why Does This Multi-Tier Architecture Exist?](#12-why-does-this-multi-tier-architecture-exist)
   - [1.3 The 10-Stage Request Lifecycle: From Keystroke to Database](#13-the-10-stage-request-lifecycle-from-keystroke-to-database)
   - [1.4 Key Configuration Touchpoints: Apache HTTPD & Tomcat](#14-key-configuration-touchpoints-apache-httpd--tomcat)
   - [1.5 Linux Diagnostic Commands (Verification at Every Layer)](#15-linux-diagnostic-commands-verification-at-every-layer)
   - [1.6 Real-World Log Analysis](#16-real-world-log-analysis)
   - [1.7 Common Failure Modes and Root Causes](#17-common-failure-modes-and-root-causes)
   - [1.8 Enterprise Production Best Practices](#18-enterprise-production-best-practices)
   - [1.9 Module 1 Assessment & Interview Gate](#19-module-1-assessment--interview-gate)
3. [Module 2: Apache HTTPD Deep Dive & MPM Internals](#module-2-apache-httpd-deep-dive--mpm-internals)
   - [2.1 What is Apache HTTPD & How Does a Web Server Work?](#21-what-is-apache-httpd--how-does-a-web-server-work)
   - [2.2 Why Do Multi-Processing Modules (MPMs) Exist?](#22-why-do-multi-processing-modules-mpms-exist)
   - [2.3 The Three MPM Architectures: Prefork vs. Worker vs. Event](#23-the-three-mpm-architectures-prefork-vs-worker-vs-event)
   - [2.4 MPM Sizing Formulas & Production Mathematics](#24-mpm-sizing-formulas--production-mathematics)
   - [2.5 Practical Linux Commands: Inspecting and Managing MPMs](#25-practical-linux-commands-inspecting-and-managing-mpms)
   - [2.6 Real-World Failure: "server reached MaxRequestWorkers setting"](#26-real-world-failure-server-reached-maxrequestworkers-setting)
   - [2.7 Module 2 Assessment & Interview Gate](#27-module-2-assessment--interview-gate)
4. [Module 3: Apache HTTPD Configuration, VirtualHosts & mod_rewrite](#module-3-apache-httpd-configuration-virtualhosts--mod_rewrite)
   - [3.1 Apache HTTPD Configuration Hierarchy & Ubuntu Directory Structure](#31-apache-httpd-configuration-hierarchy--ubuntu-directory-structure)
   - [3.2 Virtual Hosts Architecture: Name-based vs. IP-based](#32-virtual-hosts-architecture-name-based-vs-ip-based)
   - [3.3 Core Directory Containers, Precedence, and Permissions](#33-core-directory-containers-precedence-and-permissions)
   - [3.4 The URL Rewriting Engine (mod_rewrite)](#34-the-url-rewriting-engine-mod_rewrite)
   - [3.5 Production SSL/TLS Hardening & HTTPS Configuration](#35-production-ssltls-hardening--https-configuration)
   - [3.6 Practical Linux Operational Commands](#36-practical-linux-operational-commands)
   - [3.7 Common Production Errors and Root Cause Analysis](#37-common-production-errors-and-root-cause-analysis)
   - [3.8 Module 3 Assessment & Interview Gate](#38-module-3-assessment--interview-gate)
5. [Module 4: Apache Tomcat Architecture & Internals](#module-4-apache-tomcat-architecture--internals)
   - [4.1 What is Apache Tomcat?](#41-what-is-apache-tomcat)
   - [4.2 Tomcat's Internal Component Hierarchy](#42-tomcats-internal-component-hierarchy)
   - [4.3 The Coyote Connector: Threading & I/O Models](#43-the-coyote-connector-threading--io-models)
   - [4.4 Catalina Pipeline & Valve Architecture](#44-catalina-pipeline--valve-architecture)
   - [4.5 Jasper: The JSP Compiler Engine](#45-jasper-the-jsp-compiler-engine)
   - [4.6 Directory Structure & Multi-Instance Architecture (CATALINA_HOME vs CATALINA_BASE)](#46-directory-structure--multi-instance-architecture-catalina_home-vs-catalina_base)
   - [4.7 Hardened Production Systemd Unit (tomcat.service)](#47-hardened-production-systemd-unit-tomcatservice)
   - [4.8 Practical Linux Commands for Tomcat Inspection](#48-practical-linux-commands-for-tomcat-inspection)
   - [4.9 Common Production Failure Modes & Root Causes](#49-common-production-failure-modes--root-causes)
   - [4.10 Module 4 Assessment & Interview Gate](#410-module-4-assessment--interview-gate)
6. [Module 5: Tomcat Production Configuration & JVM Tuning](#module-5-tomcat-production-configuration--jvm-tuning)
   - [5.1 Core Configuration Files Deep Dive](#51-core-configuration-files-deep-dive)
   - [5.2 Sizing Connectors, Thread Pools & Queues](#52-sizing-connectors-thread-pools--queues)
   - [5.3 JNDI Database Connection Pooling (conf/context.xml)](#53-jndi-database-connection-pooling-confcontextxml)
   - [5.4 JVM Memory Layout, Heap Sizing & Garbage Collection Tuning](#54-jvm-memory-layout-heap-sizing--garbage-collection-tuning)
   - [5.5 The Gold-Standard bin/setenv.sh Configuration](#55-the-gold-standard-binsetenvsh-configuration)
   - [5.6 Practical Linux & JVM Commands for Performance Triage](#56-practical-linux--jvm-commands-for-performance-triage)
   - [5.7 Common Production Failure Modes & Root Causes](#57-common-production-failure-modes--root-causes)
   - [5.8 Module 5 Assessment & Interview Gate](#58-module-5-assessment--interview-gate)
7. [Module 6: Enterprise Integration (HTTPD to Tomcat via mod_proxy & AJP)](#module-6-enterprise-integration-httpd-to-tomcat-via-mod_proxy--ajp)
   - [6.1 The Reverse Proxy Integration Architecture](#61-the-reverse-proxy-integration-architecture)
   - [6.2 Communication Protocols: mod_proxy_http vs mod_jk / AJP](#62-communication-protocols-mod_proxy_http-vs-mod_jk--ajp)
   - [6.3 Tuning mod_proxy: Connection Pooling & Reliability Directives](#63-tuning-mod_proxy-connection-pooling--reliability-directives)
   - [6.4 Load Balancing with mod_proxy_balancer](#64-load-balancing-with-mod_proxy_balancer)
   - [6.5 Sticky Sessions (Session Persistence)](#65-sticky-sessions-session-persistence)
   - [6.6 Active Health Checking (mod_proxy_hcheck)](#66-active-health-checking-mod_proxy_hcheck)
   - [6.7 Zero-Downtime Rolling Deployments](#67-zero-downtime-rolling-deployments)
   - [6.8 Complete Production Working Configurations](#68-complete-production-working-configurations)
   - [6.9 Practical Linux Verification Commands](#69-practical-linux-verification-commands)
   - [6.10 Common Production Integration Failures & Root Causes](#610-common-production-integration-failures--root-causes)
   - [6.11 Module 6 Assessment & Interview Gate](#611-module-6-assessment--interview-gate)
8. [Module 7: Linux Production Troubleshooting Playbook](#module-7-linux-production-troubleshooting-playbook)
   - [7.1 The Senior Sysadmin Diagnostic Framework: The USE Method](#71-the-senior-sysadmin-diagnostic-framework-the-use-method)
   - [7.2 The Essential Linux Diagnostic Toolkit](#72-the-essential-linux-diagnostic-toolkit)
   - [7.3 The 20 Production Failure Scenarios & Troubleshooting Playbook](#73-the-20-production-failure-scenarios--troubleshooting-playbook)
   - [7.4 Module 7 Assessment & Interview Gate](#74-module-7-assessment--interview-gate)
9. [Module 8: Security Hardening, Production Monitoring & Hands-on Projects](#module-8-security-hardening-production-monitoring--hands-on-projects)
   - [8.1 Enterprise Security Hardening Checklist](#81-enterprise-security-hardening-checklist)
   - [8.2 Enterprise Production Monitoring & Metrics (Prometheus & Grafana)](#82-enterprise-production-monitoring--metrics-prometheus--grafana)
   - [8.3 Hands-On Production Projects: Step-by-Step Implementation](#83-hands-on-production-projects-step-by-step-implementation)
   - [8.4 Module 8 Assessment & Final Course Capstone](#84-module-8-assessment--final-course-capstone)

---

# Curriculum Overview & Training Framework

Every topic in this curriculum is taught through a rigorous 13-point operational lens:
1. **Basic Concept**: What is it?
2. **Why It Exists**: What business and engineering problem does it solve?
3. **Internal Mechanics**: How does it work down to the OS kernel, process, socket, and memory level?
4. **Architecture & Request Lifecycle**: Data flow and state machines.
5. **Real-World Use Cases**: Enterprise scenarios.
6. **Key Configuration Files & Parameters**: Production syntax.
7. **Practical Linux Commands**: Real-time diagnostic and operational commands.
8. **Verification**: How to validate functionality and prove it is working.
9. **Common Errors & Root Causes**: Why things break in production.
10. **Step-by-Step Troubleshooting**: Diagnostic checklists.
11. **Log Analysis**: Identifying patterns, timing, and errors across logs.
12. **Production Best Practices**: Hardening, sizing, and reliability engineering.
13. **Assessment & Interview Gate**: Technical questions and hands-on scenarios evaluated against senior DevOps hiring standards.

---

# Module 1: The End-to-End Enterprise Request Lifecycle

## 1.1 Enterprise Multi-Tier Architecture Diagram

In an enterprise production deployment, backend application services are isolated within private networks. The web server acts as the ingress controller and reverse proxy inside the DMZ.

```
+-----------------------------------------------------------------------------------------+
|                                    CLIENT LAYER                                         |
|  User Browser (Chrome / Firefox)  ---> Resolves DNS via Recursive Resolvers             |
+--------------------------------------------+--------------------------------------------+
                                             |  HTTPS (Port 443 / TCP)
                                             v
+-----------------------------------------------------------------------------------------+
|                                EDGE / INGRESS LAYER                                     |
|  AWS Route 53 / Anycast DNS  --> AWS ALB / NLB or Hardware Firewall (WAF / DDoS Guard)  |
+--------------------------------------------+--------------------------------------------+
                                             |  Forwarded TCP/HTTPS
                                             v
+-----------------------------------------------------------------------------------------+
|                              WEB TIER: APACHE HTTPD (DMZ)                               |
|  - IP: 10.0.1.10 (Public/DMZ Subnet)                                                    |
|  - TLS/SSL Termination (Certbot / Custom Wildcard CA)                                   |
|  - Serves Static Assets (HTML, CSS, JS, Images) from disk directly via kernel sendfile  |
|  - URL Rewriting / Normalization (mod_rewrite)                                          |
|  - Rate Limiting / Security Filters (mod_security, mod_evasive)                         |
|  - Reverse Proxy & Layer 7 Load Balancer (mod_proxy, mod_proxy_http, mod_proxy_balancer) |
+--------------------------------------------+--------------------------------------------+
                                             |  Internal HTTP (Port 8080) or AJP (Port 8009)
                                             |  Private Subnet (No Public IP)
                                             v
+-----------------------------------------------------------------------------------------+
|                         APPLICATION TIER: APACHE TOMCAT (JVM)                           |
|  - IP: 10.0.2.20 (Private Subnet)                                                       |
|  - Protocol Connector (Coyote): Non-Blocking I/O (NIO/NIO2) Thread Pool                 |
|  - Engine (Catalina): Host -> Context (e.g., /api) -> Pipeline & Valves                 |
|  - Servlet Container: Filters -> DispatcherServlet / Custom Servlets                    |
|  - Business Logic Execution: Spring Services / EJBs                                     |
|  - Database Connection Pool (HikariCP / Tomcat JDBC Pool)                               |
+--------------------------------------------+--------------------------------------------+
                                             |  TCP (Port 5432 - Postgres / 3306 - MySQL)
                                             v
+-----------------------------------------------------------------------------------------+
|                                    DATABASE TIER                                        |
|  PostgreSQL / MySQL Cluster (Primary / Read Replicas)                                   |
+-----------------------------------------------------------------------------------------+
```

---

## 1.2 Why Does This Multi-Tier Architecture Exist?

Why do we avoid exposing Apache Tomcat directly to the public internet?

| Requirement | If Tomcat is Publicly Exposed | How Apache HTTPD Solves It |
| :--- | :--- | :--- |
| **Static Asset Performance** | Tomcat consumes JVM heap memory and Java worker threads parsing HTTP headers for static images, CSS, and JS files. | HTTPD uses the Linux kernel system call `sendfile(2)`, transferring static bytes directly from the page cache to the network socket with **zero-copy** and near-zero CPU/memory overhead. |
| **Security & Attack Surface** | A security flaw in Tomcat or JVM memory exposure risks the core application and database credentials directly. | HTTPD acts as a **bastion shield** in a DMZ. It enforces request size limits, sanitizes HTTP headers, blocks malicious URIs with `mod_security`, and drops slowloris/DDoS attacks before they reach the JVM. |
| **SSL/TLS CPU Overhead** | High-concurrency TLS handshakes (RSA/ECDSA decryption) consume significant JVM CPU cycles and trigger garbage collection pauses. | HTTPD handles **SSL Termination** using native OpenSSL C libraries, utilizing hardware AES-NI instructions. Tomcat receives unencrypted, fast internal HTTP. |
| **Zero-Downtime Deployments** | Restarting Tomcat drops active client TCP connections. | HTTPD's `mod_proxy_balancer` dynamically drains connections from node 1, switches traffic to node 2, and allows rolling upgrades with zero end-user errors. |
| **Process Isolation** | A JVM crash or `java.lang.OutOfMemoryError` takes down the entire site, including static landing pages. | If Tomcat dies, HTTPD remains alive, returning a custom, branded **503 Service Unavailable** page while alerting sysadmins. |

---

## 1.3 The 10-Stage Request Lifecycle: From Keystroke to Database

Let us trace a real-world request:
$$\text{URL: } \mathbf{https://app.example.com/api/v1/orders/1024}$$

```
Browser               DNS                 HTTPD               Tomcat               Database
   |                   |                    |                   |                     |
   |--- 1. Query ----->|                    |                   |                     |
   |<-- 2. IP:Port ----|                    |                   |                     |
   |                                        |                   |                     |
   |--- 3. TCP 3-Way Handshake (SYN/ACK) -->|                   |                     |
   |--- 4. TLS 1.3 Handshake (ECDHE) ------>|                   |                     |
   |--- 5. Encrypted HTTP GET ------------->|                   |                     |
   |                                        |-- 6. mod_proxy -->|                     |
   |                                        |   (Internal HTTP) |                     |
   |                                        |                   |-- 7. Borrow Conn -->|
   |                                        |                   |-- 8. Run SQL ------>|
   |                                        |                   |<- 9. Row Results ---|
   |                                        |<- 10. HTTP 200 ---|                     |
   |<-- 11. Encrypted HTTP 200 (JSON) ------|                   |                     |
```

### Stage 1: Browser Navigation & Client Cache Check
1. The user types the URL and presses `Enter`.
2. The browser checks internal caches in strict order:
   - **Browser HSTS Cache**: Checks if `Strict-Transport-Security` enforces an immediate upgrade of `http://` to `https://`.
   - **Browser DNS Cache**: (e.g., `chrome://net-internals/#dns`).
   - **OS DNS Cache**: Operating system cache (`systemd-resolved` or local hosts file `/etc/hosts`).

### Stage 2: Recursive DNS Resolution
If the IP is not cached:
1. The OS queries the resolver configured in `/etc/resolv.conf` (e.g., `127.0.0.53` or AWS VPC DNS `10.0.0.2`).
2. The resolver traverses the DNS hierarchy:
   - **Root Nameservers** (`.`): Returns `.com` TLD nameservers.
   - **TLD Nameservers** (`.com`): Returns authoritative nameservers for `example.com`.
   - **Authoritative Nameserver**: Responds with the **A Record** (e.g., `203.0.113.50`) or CNAME.
3. The IP `203.0.113.50` is returned to the browser and cached according to its TTL.

### Stage 3: TCP 3-Way Handshake (Layer 4)
The browser opens a client socket on an ephemeral port (e.g., `51234`) to `203.0.113.50:443`:
1. **SYN**: Client sends `[SYN] Seq=0` with TCP options (MSS, Window Scale, SACK).
2. **SYN-ACK**: Web server kernel accepts the connection in its SYN backlog queue and responds with `[SYN, ACK] Seq=0 Ack=1`.
3. **ACK**: Client replies `[ACK] Seq=1 Ack=1`.
*The TCP socket state transitions to `ESTABLISHED`.*

### Stage 4: TLS 1.3 Handshake (Layer 7 Security)
Before any plain text is exchanged, encryption is negotiated:
1. **ClientHello**: Browser sends supported TLS versions (TLS 1.2, 1.3), cipher suites (e.g., `TLS_AES_256_GCM_SHA384`), and crucial extensions:
   - **SNI (Server Name Indication)**: Indicates `app.example.com` so Apache HTTPD can present the correct virtual host SSL certificate.
   - **ALPN (Application-Layer Protocol Negotiation)**: Negotiates `h2` (HTTP/2) or `http/1.1`.
2. **ServerHello + Certificate + Finished**: Apache (via OpenSSL) matches the VirtualHost, returns its public X.509 certificate chain, selects the cipher, and establishes symmetric session keys via Diffie-Hellman Key Exchange (ECDHE).
3. The TLS session is secure and active.

### Stage 5: HTTP Request Submission
The browser encrypts and sends the HTTP payload over the TLS tunnel:
```http
GET /api/v1/orders/1024 HTTP/1.1
Host: app.example.com
User-Agent: Mozilla/5.0 ...
Accept: application/json
Authorization: Bearer eyJhbGciOi...
X-Request-ID: 7a92b2d0-1c3f-4e2b
```

### Stage 6: Apache HTTPD Ingestion & Processing
Apache HTTPD’s Multi-Processing Module (MPM—e.g., **Event MPM**) processes the event:
1. **Kernel to User Space**: The listener thread wakes up via `epoll_wait()`, pulls the file descriptor from the OS accept queue, and assigns it to an available worker thread.
2. **VirtualHost Selection**: HTTPD reads the `Host: app.example.com` header and matches it against configured `<VirtualHost *:443>` blocks.
3. **Security & Rewrite Phase**:
   - `mod_headers` inspects incoming headers.
   - `mod_rewrite` evaluates rules (e.g., validating URL structures or blocking malicious characters).
4. **Proxy Evaluation**:
   HTTPD encounters a directive matching the path:
   ```apache
   ProxyPass /api/ http://10.0.2.20:8080/api/
   ProxyPassReverse /api/ http://10.0.2.20:8080/api/
   ```
5. **Header Enrichment (Crucial for Production)**:
   HTTPD strips untrusted client headers and injects internal reverse proxy headers:
   - `X-Forwarded-For: <Client-Public-IP>`
   - `X-Forwarded-Proto: https`
   - `X-Forwarded-Host: app.example.com`
   - `X-Forwarded-Port: 443`
6. **Backend TCP Connection**: HTTPD opens a connection pool socket to Tomcat on `10.0.2.20:8080` and sends the modified HTTP request across the private network.

### Stage 7: Apache Tomcat Processing
Tomcat receives the request inside the JVM on port `8080`:

```
               [ Tomcat Server (JVM Process) ]
                              |
                     [ Service: Catalina ]
                              |
               [ Coyote Connector (Port 8080, NIO) ]
           Worker Thread assigned from Executor pool
                              |
                 [ Engine: Catalina Engine ]
                              |
                  [ Host: app.example.com ]
                              |
               [ Context: /api (Web Application) ]
                              |
                  [ Valves (Pipeline Chain) ]
             -> RemoteIpValve (reconstructs real IP)
             -> AccessLogValve (logs request)
                              |
               [ Filter Chain (Spring Security) ]
                              |
                  [ DispatcherServlet / Servlet ]
```

1. **Coyote Connector (HTTP/1.1 NIO)**: The internal Poller thread detects readable bytes on the socket connected to Apache HTTPD, extracts a worker thread from its `ThreadPoolExecutor`, and constructs the `HttpServletRequest` and `HttpServletResponse` wrapper objects.
2. **Valve Pipeline**:
   - `RemoteIpValve` reads `X-Forwarded-Proto` and `X-Forwarded-For`. It resets `request.isSecure()` to `true` and `request.getRemoteAddr()` to the client's public IP rather than Apache's internal IP.
3. **Servlet & Framework Execution**:
   - The request hits the application's `Filter` chain (e.g., JWT authentication, rate limiting).
   - Routed to the controller (e.g., `@GetMapping("/orders/{id}")`).

### Stage 8: Database Communication (JDBC & Connection Pool)
1. The Java application needs to query order `1024`. It does **not** create a new TCP connection to the database (which takes 20–50ms).
2. It borrows an already `ESTABLISHED` TCP socket connection from its **Connection Pool** (e.g., HikariCP).
3. The driver issues the SQL statement over TCP (e.g., to PostgreSQL on port 5432):
   ```sql
   SELECT id, customer_id, total, status FROM orders WHERE id = 1024;
   ```
4. The database executes the query against its buffer cache/disk, returns the row result set over the socket, and the application immediately returns the connection handle back to the pool.

### Stage 9: Response Assembly & Reverse Journey
1. The Java business logic serializes the data to JSON:
   ```json
   {"id": 1024, "status": "SHIPPED", "total": 149.50}
   ```
2. Tomcat flushes the HTTP response (Status `200 OK`) back over the private socket to Apache HTTPD.
3. Apache HTTPD receives the response:
   - `ProxyPassReverse` rewrites any redirect headers (`Location: ...`) so Tomcat’s internal IP/port is never leaked to the outside world.
   - `mod_deflate` optionally compresses the JSON with `gzip` or `brotli`.
   - Security headers are appended: `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`.
4. Apache encrypts the response via the established TLS tunnel and transmits it across the public internet to the user's browser.

### Stage 10: Client Rendering
1. The browser receives the TCP packets, decrypts the TLS stream, and parses the HTTP status `200 OK`.
2. The browser JavaScript engine (V8/SpiderMonkey) resolves the API Promise, extracts the JSON object, and renders the updated UI in the DOM.
3. The TCP connection enters an idle `Keep-Alive` state, waiting for subsequent requests to avoid repeating the handshake.

---

## 1.4 Key Configuration Touchpoints: Apache HTTPD & Tomcat

### 1. Apache HTTPD VirtualHost Configuration (`/etc/apache2/sites-available/app.conf`)
```apache
<VirtualHost *:443>
    ServerName app.example.com
    ServerAdmin webmaster@example.com

    # TLS Engine Setup
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/app.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/app.example.com.key
    SSLCertificateChainFile /etc/ssl/certs/intermediate-ca.crt
    SSLProtocol -all +TLSv1.2 +TLSv1.3
    SSLCipherSuite HIGH:!aNULL:!MD5:!3DES:!CAMELLIA:!SRP:!PSK

    # Fast Static Asset Delivery Directly by Apache (Bypassing Tomcat)
    DocumentRoot /var/www/app/static
    <Directory /var/www/app/static>
        Require all granted
        Options -Indexes +FollowSymLinks
        AllowOverride None
    </Directory>

    # Forward Dynamic API Calls to Tomcat Backend
    ProxyPreserveHost On
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"

    ProxyPass /api/ http://10.0.2.20:8080/api/ retry=2 timeout=30
    ProxyPassReverse /api/ http://10.0.2.20:8080/api/

    # Dedicated Logs with Microsecond Timing (%D)
    ErrorLog ${APACHE_LOG_DIR}/app_error.log
    CustomLog ${APACHE_LOG_DIR}/app_access.log "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D"
</VirtualHost>
```

### 2. Tomcat Configuration (`/opt/tomcat/conf/server.xml`)
```xml
<!-- Coyote Connector configured for Reverse Proxy traffic -->
<Connector port="8080" 
           protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           maxThreads="200"
           minSpareThreads="20"
           maxConnections="8192"
           acceptCount="100"
           enableLookups="false"
           URIEncoding="UTF-8" />

<Engine name="Catalina" defaultHost="localhost">
    <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="false">
        
        <!-- Crucial Valve: Rebuilds real client IP and HTTPS status from Apache headers -->
        <Valve className="org.apache.catalina.valves.RemoteIpValve"
               internalProxies="10\.0\.1\.\d+"
               remoteIpHeader="x-forwarded-for"
               protocolHeader="x-forwarded-proto" />

        <Valve className="org.apache.catalina.valves.AccessLogValve" 
               directory="logs"
               prefix="localhost_access_log" 
               suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b %D" />
    </Host>
</Engine>
```

---

## 1.5 Linux Diagnostic Commands (Verification at Every Layer)

### 1. Test DNS Resolution and Latency
```bash
# Query the authoritative DNS server directly with full trace
dig +trace +question app.example.com

# Verify local OS resolution
nslookup app.example.com
```

### 2. Trace the Network Path and Test Port Openings
```bash
# Test TCP handshake to port 443 with netcat (nc)
nc -zv -w 3 203.0.113.50 443

# Check route MTU and hops to eliminate firewall blocks
traceroute -T -p 443 app.example.com
```

### 3. Inspect Full HTTP & TLS Handshake Details
```bash
# Run verbose curl with detailed connection timers
curl -vvv -k --resolve app.example.com:443:127.0.0.1 \
  -H "Host: app.example.com" \
  https://app.example.com/api/v1/orders/1024 \
  -w "\n--- Timers ---\nDNS Lookup: %{time_namelookup}s\nTCP Connect: %{time_connect}s\nTLS Handshake: %{time_appconnect}s\nPre-transfer: %{time_pretransfer}s\nTime to First Byte (TTFB): %{time_starttransfer}s\nTotal Time: %{time_total}s\n"
```

### 4. Inspect Active Listening Ports and Sockets on the Server
```bash
# Show all active listening TCP ports with process names and PIDs
ss -tulpn | grep -E '(:80|:443|:8080|:5432)'

# Count current established socket connections between Apache and Tomcat
ss -tan 'sport = :8080' | grep ESTAB | wc -l
```

### 5. Sniff Internal Packets Between Apache and Tomcat
```bash
# Sniff internal traffic between HTTPD and Tomcat to verify header injection
sudo tcpdump -i any -nn -vv -A 'tcp port 8080 and host 10.0.2.20'
```

---

## 1.6 Real-World Log Analysis

When a request breaks, correlate across logs:

### Apache Access Log (`/var/log/apache2/app_access.log`)
```
203.0.113.4 - - [20/Nov/2026:12:45:01 +0000] "GET /api/v1/orders/1024 HTTP/1.1" 502 452 "-" "Mozilla/5.0" 30014522
```
* **Analysis**: Client `203.0.113.4` received a `502 Bad Gateway`. The last field is `%D`: `30014522` microseconds = **30.01 seconds**. This confirms Apache waited 30 seconds for the backend (Tomcat), reached its `timeout=30`, and terminated the connection.

### Apache Error Log (`/var/log/apache2/app_error.log`)
```
[Fri Nov 20 12:45:01.321045 2026] [proxy_http:error] [pid 14210:tid 140231] (110)Connection timed out: [client 203.0.113.4:52134] AH01102: error reading status line from remote server 10.0.2.20:8080, referer: ...
[Fri Nov 20 12:45:01.321088 2026] [proxy:error] [pid 14210:tid 140231] [client 203.0.113.4:52134] AH00898: Error reading from remote server returned by /api/v1/orders/1024
```
* **Analysis**: `AH01102: error reading status line from remote server`: Apache established the TCP socket to Tomcat, but Tomcat’s thread was blocked (e.g., stuck on a database deadlock or thread pool exhaustion) and failed to respond within 30 seconds.

### Tomcat Access Log (`/opt/tomcat/logs/localhost_access_log.2026-11-20.txt`)
```
203.0.113.4 - - [20/Nov/2026:12:44:31 +0000] "GET /api/v1/orders/1024 HTTP/1.1" 200 125 32100
```
* **Analysis**: Notice that Tomcat finished with `200 OK`, but took `32100` milliseconds (**32.1 seconds**). Because Tomcat took 32 seconds and Apache had a 30-second timeout, the user received an error (502) even though Tomcat eventually finished its work!

---

## 1.7 Common Failure Modes and Root Causes

| HTTP Status | Stage It Breaks | Most Common Production Root Causes |
| :--- | :--- | :--- |
| **Connection Refused / Can't Reach Site** | Stage 3 (TCP) | Apache HTTPD service is dead (`systemctl status apache2`), firewall (`ufw` or AWS Security Group) is blocking port 80/443, or port conflict. |
| **SSL Handshake Failed / Untrusted Cert** | Stage 4 (TLS) | Certificate expired, SNI mismatch (wrong domain on cert), missing Intermediate CA certificate bundle, or unsupported TLS protocol version. |
| **403 Forbidden** | Stage 6 (Apache) | Linux file permissions on `DocumentRoot` are wrong (Apache user `www-data` cannot read the directory), or missing `Require all granted` in Apache config. |
| **502 Bad Gateway** | Stage 6 $\leftrightarrow$ 7 Boundary | Tomcat is completely down/crashed, Tomcat port 8080 is blocked by local firewall, or Tomcat's JVM locked up during Garbage Collection (OOM). |
| **504 Gateway Timeout** | Stage 6 $\leftrightarrow$ 7 Boundary | Tomcat's request processing time exceeded Apache's `timeout` directive (often caused by slow SQL queries or database connection pool exhaustion). |
| **Wrong Client IP in Tomcat Logs** | Stage 7 (Valve) | Missing `RemoteIpValve` in Tomcat or missing `X-Forwarded-For` in Apache HTTPD. Tomcat logs the internal proxy IP (`10.0.1.10`) instead of the real user IP. |

---

## 1.8 Enterprise Production Best Practices

1. **Always Match Reverse Proxy Timeouts**:
   - Ensure Apache's `timeout` directive is slightly larger than Tomcat's internal service timeouts.
   - If Apache times out at 30s and Tomcat times out at 60s, Apache will disconnect, leaving Tomcat executing queries for orphaned clients.
2. **Never Trust Incoming `X-Forwarded-*` Headers from Clients**:
   - Strip client-supplied `X-Forwarded-For` headers at the edge or let Apache overwrite them to prevent IP spoofing attacks.
3. **Always Log Execution Duration (`%D`)**:
   - Always configure `%D` in both Apache (`CustomLog`) and Tomcat (`AccessLogValve`). Microsecond/millisecond timing is the single most valuable metric for isolating whether latency originated in HTTPD, Tomcat, or the Database.
4. **Tune Kernel Socket Backlogs**:
   - On high-traffic Linux hosts, raise `net.core.somaxconn` and `net.ipv4.tcp_max_syn_backlog` so bursts of TCP handshakes are not dropped before reaching Apache or Tomcat.

---

## 1.9 Module 1 Assessment & Interview Gate

Before advancing to **Module 2: Apache HTTPD Deep Dive & MPM Internals**, answer the following questions:

### Conceptual Questions
1. **Explain the difference between TCP connection establishment and the TLS handshake.** Why can't we send the HTTP `Host: app.example.com` header *before* the TLS handshake occurs, and what extension was created to solve this dilemma?
2. In our architecture, why must we configure both `ProxyPass` and `ProxyPassReverse` in Apache HTTPD? What specific problem occurs on the client's browser if `ProxyPassReverse` is omitted when Tomcat issues an HTTP `302 Redirect`?
3. What is the fundamental difference between how Apache HTTPD (using `sendfile`) serves an image vs. how Apache Tomcat serves that same image, from a CPU, memory, and OS syscall perspective?

### Practical Linux Questions
4. You run `curl -I https://app.example.com/api/test` from your laptop and receive `curl: (7) Failed to connect to app.example.com port 443: Connection refused`. You SSH into the Linux server hosting Apache. What **two specific Linux commands** do you run to immediately determine if Apache is listening on port 443 and whether the process is even running?
5. You want to verify whether Apache HTTPD is stripping or correctly injecting the `X-Forwarded-For` header before forwarding the packet to Tomcat on port 8080. You do not want to restart or modify either service. What exact `tcpdump` command would you run on the server to inspect the plain HTTP request headers traveling over port 8080?

### Real-World Production Troubleshooting Scenario
6. **Scenario**: On Black Friday, users report intermittent `HTTP 502 Bad Gateway` errors when checking out.
   - You log in to the Apache HTTPD web tier. The Apache error log displays:
     `[proxy_http:error] [pid 8124:tid 1399] (111)Connection refused: AH01114: HTTP: failed to make connection to backend: 10.0.2.20:8080`
   - You log in to the Tomcat application server (`10.0.2.20`). You check `systemctl status tomcat` and find that the Tomcat process is **active (running)** with high memory usage, but `top` shows CPU usage is bouncing at 99%.
   
   **How do you triage this?** What is the probable root cause, what immediate commands would you execute on the Tomcat server to prove it, and what emergency remediation steps would you take to stabilize the site?

---

# Module 2: Apache HTTPD Deep Dive & MPM Internals

## 2.1 What is Apache HTTPD & How Does a Web Server Work?

At its core, **Apache HTTP Server (HTTPD)** is a high-performance, modular, open-source HTTP/HTTPS server developed in C by the Apache Software Foundation.

### The Role Distinction in Production
- **Web Server (Apache HTTPD, Nginx)**: Optimized for raw network I/O, serving static assets, terminating TLS/SSL, rewriting URLs, and enforcing HTTP protocol security. It does *not* execute Java bytecode or run Spring beans.
- **Application Server (Apache Tomcat, WildFly)**: Contains a runtime container (e.g., Servlet/JSP container) capable of executing dynamic compiled application code, managing application lifecycle, and providing transaction/database access.
- **Reverse Proxy**: A gateway that sits between public clients and private backend services, terminating client connections, rewriting request headers, distributing load, and forwarding requests to upstream application servers.

### How a Web Server Works Internally: From Sockets to Buffers
When Apache HTTPD runs on Linux:
1. **Master / Parent Process (`root`)**: Starts with superuser privileges so it can bind to restricted low ports (`< 1024`, such as 80 and 443), opens log file handles, and reads configuration files.
2. **Child / Worker Processes (`www-data` or `apache`)**: Immediately drops privileges to an unprivileged service account for security.
3. **The OS Kernel Socket Queue**: The Linux kernel maintains two queues for listening sockets:
   - **SYN Queue (`tcp_max_syn_backlog`)**: Holds half-open connections undergoing the 3-way handshake.
   - **Accept Queue (`somaxconn`)**: Holds fully established TCP connections waiting for user space `accept(2)` or `accept4(2)`.
4. **I/O Multiplexing via `epoll(7)`**: Instead of spawning a thread or process just to wait for a network packet, modern HTTPD uses Linux `epoll` system calls. A single listener thread can monitor tens of thousands of active file descriptors simultaneously without consuming CPU cycles while idle.
5. **Bucket Brigades and Filters**: Apache processes HTTP requests using an internal abstraction called **Brigades** (chains) of **Buckets** (memory buffers, file descriptors, or pipe chunks). Modules register as either **Input Filters** (e.g., decompression, SSL decryption) or **Output Filters** (e.g., SSL encryption, gzip compression, header injection), passing buckets down the pipeline without unnecessary buffer copying.

---

## 2.2 Why Do Multi-Processing Modules (MPMs) Exist?

Operating systems handle concurrency differently. Linux offers processes (heavy isolation, separate virtual memory address spaces) and threads (lightweight, shared address space). 

In the early days of the web, serving a request meant forking a process. As traffic scaled to thousands of simultaneous connections (the **C10K Problem**), pure process-based servers suffered catastrophic thrashing and RAM exhaustion.

To solve this across varied operating systems and workloads, Apache HTTPD decoupled its network connection handling architecture into **Multi-Processing Modules (MPMs)**. The MPM dictates how HTTPD binds to network ports, accepts incoming TCP requests, and dispatches them to execution units.

---

## 2.3 The Three MPM Architectures: Prefork vs. Worker vs. Event

```
+----------------------------------------------------------------------------------------------------+
|                                    1. PREFORK MPM (Process-Based)                                  |
|  Master (root) -> Forks N Child Processes (www-data)                                               |
|  Each Process = Exactly 1 Thread = Handles Exactly 1 Connection at a Time                         |
|  Pros: 100% Memory Isolation, Safe for non-thread-safe C modules (e.g., old mod_php)                |
|  Cons: Massive RAM usage (~30MB-50MB per process); Dies under high concurrency or Keep-Alive      |
+----------------------------------------------------------------------------------------------------+

+----------------------------------------------------------------------------------------------------+
|                                2. WORKER MPM (Hybrid Multi-Process / Multi-Threaded)               |
|  Master (root) -> Forks M Child Processes -> Each Process Spawns T Worker Threads                  |
|  Pros: Much lower RAM footprint; High concurrency                                                  |
|  Cons: Idle Keep-Alive connections tie up active worker threads, starving new incoming requests     |
+----------------------------------------------------------------------------------------------------+

+----------------------------------------------------------------------------------------------------+
|                                3. EVENT MPM (Asynchronous Event-Driven / epoll)                    |
|  Master (root) -> Forks M Child Processes -> 1 Dedicated Listener Thread + T Worker Threads        |
|  Listener Thread uses epoll() to monitor Keep-Alive & SSL handshakes.                              |
|  Worker Threads are ONLY dispatched when actual HTTP request bytes arrive to be processed!         |
|  Pros: Ultra-low RAM; Handles 50,000+ persistent connections effortlessly. Production Gold Standard|
+----------------------------------------------------------------------------------------------------+
```

### Detailed Deep-Dive: Event MPM Mechanics (The Modern Standard)
In modern production environments (Ubuntu 22.04/24.04 with HTTP/2 and Tomcat backends), **Event MPM** is mandatory.

Here is how Event MPM processes a persistent HTTP/1.1 or HTTP/2 connection:
1. **Connection Arrival**: A client connects. The listener thread accepts the TCP socket.
2. **TLS Handshake & Request Parsing**: An available worker thread is assigned to read the initial encrypted bytes and parse the HTTP headers.
3. **Keep-Alive Hand-off**: Once the response is sent back to the client, the client socket remains in `Keep-Alive` state. In older Worker MPM, that worker thread had to sit idle, blocked waiting for the client's next click. **In Event MPM, the worker thread immediately detaches and returns to the pool!**
4. **Listener Event Loop**: The listener thread adds the socket's file descriptor to an internal `epoll` set.
5. **Wakeup on Demand**: When the client clicks a new link 10 seconds later, `epoll_wait` triggers in the listener thread, which immediately grabs an available worker thread to process the new request.
*Result: 5,000 idle browser connections consume almost zero worker threads and minimal RAM.*

---

## 2.4 MPM Sizing Formulas & Production Mathematics

Misconfiguring MPM directives is the #1 cause of either server OOM crashes or artificial request queuing. You must calculate these values mathematically based on physical hardware.

### Key Directives Defined
- `ServerLimit`: The maximum hard ceiling of child processes that can ever be configured.
- `StartServers`: Number of child server processes created at startup.
- `ThreadsPerChild`: Number of worker threads created by *each* child process (used in Worker and Event MPM).
- `MaxRequestWorkers`: The absolute maximum number of simultaneous connections that will be processed simultaneously.
  $$\text{MaxRequestWorkers} = \text{ServerLimit} \times \text{ThreadsPerChild}$$
- `MinSpareThreads` / `MaxSpareThreads`: Thresholds for idle worker threads. If idle threads fall below minimum, Apache spawns children; if they exceed maximum, Apache terminates children.
- `MaxConnectionsPerChild`: How many requests an individual child process may serve before dying and recycling its memory (protects against native C memory leaks). In production, set to `5000` or `10000` (never `0` if using third-party modules).

### Sizing Formula for Production Servers (Ubuntu / AWS EC2)
Suppose you have an EC2 instance dedicated to Apache HTTPD:
- **Total Physical RAM**: 8 GB ($8192\text{ MB}$)
- **OS, Kernel & Monitoring Reserve**: 2 GB ($2048\text{ MB}$)
- **Available RAM for Apache**: 6 GB ($6144\text{ MB}$)
- **Average Apache Child Process Memory (RSS)**: Measured via `ps` or `smem` as $\sim 30\text{ MB}$

#### Sizing Event MPM:
Set `ThreadsPerChild = 50` (standard balance between concurrency and context switching).
$$\text{Max Child Processes} = \frac{\text{Available RAM}}{\text{Avg Process Size}} = \frac{6144\text{ MB}}{30\text{ MB}} \approx 204\text{ processes}$$

To keep a safe 25% safety margin:
- Set `ServerLimit = 100`
- Set `ThreadsPerChild = 50`
- Calculate:
  $$\text{MaxRequestWorkers} = 100 \times 50 = \mathbf{5000}\text{ simultaneous concurrent requests}$$

### Production Configuration Block (`/etc/apache2/mods-available/mpm_event.conf`)
```apache
<IfModule mpm_event_module>
    StartServers             4
    MinSpareThreads          100
    MaxSpareThreads          300
    ThreadLimit              64
    ThreadsPerChild          50
    ServerLimit              100
    MaxRequestWorkers        5000
    MaxConnectionsPerChild   10000
</IfModule>
```

---

## 2.5 Practical Linux Commands: Inspecting and Managing MPMs

### 1. Identify Which MPM is Currently Compiled and Running
```bash
# Ubuntu / Debian
apache2ctl -V | grep -i mpm

# RHEL / CentOS / Rocky Linux
httpd -V | grep -i mpm
```
*Expected Output:*
```text
Server MPM:     event
  threaded:     yes (fixed thread count)
    forked:     yes (multi-process)
```

### 2. Inspect All Active Threads Across Apache Processes
```bash
# List all threads (-L) belonging to apache2 processes
ps -eLF | grep apache2
```
*Look at the `NLWP` column (Number of Light-Weight Processes / Threads). In Event MPM with `ThreadsPerChild 50`, you will see each child reporting 52-53 threads (50 workers + listener + housekeeping).*

### 3. Measure Real Memory Usage (RSS vs Shared)
```bash
# Show Apache processes sorted by Resident Set Size (RAM in KB)
ps -eo pid,user,comm,rss,vsz --sort -rss | grep apache2 | awk '{printf "%-8s %-10s %-12s %8.2f MB\n", $1, $2, $3, $4/1024}'
```

### 4. How to Safely Switch MPM on Ubuntu
On Debian/Ubuntu, MPMs are dynamic shared modules (`mod_mpm_*`):
```bash
# Switch from Event to Worker or Prefork (e.g. for troubleshooting)
sudo a2dismod mpm_event
sudo a2enmod mpm_worker
sudo apache2ctl configtest   # ALWAYS test syntax before restart!
sudo systemctl restart apache2
```

---

## 2.6 Real-World Failure: "server reached MaxRequestWorkers setting"

### The Symptom
Users report site unresponsiveness or extreme latency. Some requests time out before receiving headers.

### The Error Log (`/var/log/apache2/error.log`)
```
[Fri Nov 20 14:10:05.112041 2026] [mpm_event:error] [pid 4102:tid 140112] AH00484: server reached MaxRequestWorkers setting, consider raising the MaxRequestWorkers setting
```

### Root Cause Analysis & Mechanics
1. Every single available worker thread in Apache’s thread pool is currently occupied handling a request.
2. New incoming TCP handshakes complete at the OS kernel level, but they sit stalled in the TCP `somaxconn` accept queue because no worker thread is calling `accept()`.
3. If the queue fills up, the Linux kernel drops incoming `SYN` packets or sends `RST`, resulting in `Connection refused` or browser timeouts.

### Step-by-Step Triage
1. Check current connection states:
   ```bash
   ss -ant state established '( sport = :80 or sport = :443 )' | wc -l
   ```
2. Enable `mod_status` to see what worker threads are doing:
   ```apache
   # In /etc/apache2/mods-available/status.conf
   <Location /server-status>
       SetHandler server-status
       Require ip 127.0.0.1 10.0.0.0/8
   </Location>
   ```
3. Query the scoreboard from the command line:
   ```bash
   curl -s http://127.0.0.1/server-status?auto
   ```
   *Look at `Scoreboard` characters:*
   - `_`: Waiting for connection (idle)
   - `S`: Starting up
   - `R`: Reading request
   - `W`: Sending reply (writing)
   - `K`: Keep-Alive (read)
   - `D`: DNS lookup
   - `C`: Closing connection
   - `L`: Logging
   - `G`: Gracefully finishing
   - `I`: Idle cleanup of worker
   - `.`: Open slot with no current process
4. If hundreds of workers are stuck in `W` (Writing/Sending Reply) while talking to Tomcat, the problem is **not** Apache: **Tomcat or the Database is slow**, causing Apache's worker threads to back up!

---

## 2.7 Module 2 Assessment & Interview Gate

Answer the following questions to complete Module 2:

### Conceptual Questions
1. Why is **Event MPM** vastly superior to **Worker MPM** when dealing with clients using HTTP/1.1 `Keep-Alive` connections over high-latency networks (e.g., mobile devices)?
2. What is the danger of setting `MaxConnectionsPerChild 0` in an enterprise production Apache deployment running complex custom modules?
3. How does the Linux kernel `somaxconn` and `tcp_max_syn_backlog` parameter interact with Apache’s `MaxRequestWorkers` setting during a sudden 10,000 req/sec traffic spike?

### Practical Linux Questions
4. You are tasked with tuning an Ubuntu server with 16 GB of RAM, where 4 GB is dedicated to the OS and other agents. Apache Event child processes consume an average of 40 MB RSS each. Write out the exact calculation and the resulting values for `ServerLimit`, `ThreadsPerChild`, and `MaxRequestWorkers` assuming `ThreadsPerChild 25`.
5. You suspect Apache worker processes are leaking memory over time. What Linux command (using `ps` or `awk`) will continuously show you the top 5 memory-consuming Apache PIDs and their exact RSS in megabytes?

### Real-World Production Troubleshooting Scenario
6. **Scenario**: After upgrading an enterprise application, you notice that during peak hours, `top` shows 0% idle CPU and hundreds of `apache2` processes spawned, causing the server to thrash into swap space. The error log is flooded with:
   `AH00484: server reached MaxRequestWorkers setting`
   You check `apache2ctl -V` and discover the server is running **Prefork MPM** with `MaxRequestWorkers 1200`.
   - Explain why the server ran out of memory and started thrashing swap.
   - What is your emergency remediation plan to stabilize the host without rebooting?
   - What is the permanent architectural solution you must implement?

---

# Module 3: Apache HTTPD Configuration, VirtualHosts & mod_rewrite

## 3.1 Apache HTTPD Configuration Hierarchy & Ubuntu Directory Structure

On Ubuntu/Debian, Apache HTTPD does not store all configuration in a single monolithic file. It uses a modular, symlink-driven hierarchy designed for package management and zero-downtime maintenance.

### Directory Structure (`/etc/apache2/`)
```
/etc/apache2/
|-- apache2.conf                # The master configuration file (global directives, MPM inclusion)
|-- ports.conf                  # Defines listening TCP ports (e.g., Listen 80, Listen 443)
|-- envvars                     # Environment variables used by apache2 (APACHE_RUN_USER, LOG_DIR)
|-- mods-available/             # Installed module configurations and load files (.load, .conf)
|-- mods-enabled/               # Symlinks to mods-available/ for ACTIVE modules
|-- sites-available/            # Available VirtualHost configuration files (.conf)
|-- sites-enabled/              # Symlinks to sites-available/ for ACTIVE VirtualHosts
|-- conf-available/             # Global configuration snippets (e.g., security.conf)
`-- conf-enabled/               # Symlinks to conf-available/ for ACTIVE snippets
```

### The Debian/Ubuntu Helper Commands
Never manually create or delete files in `*-enabled` directories. Always use the built-in management utilities:
- **Modules**: `a2enmod <mod_name>` and `a2dismod <mod_name>`
- **Sites (VirtualHosts)**: `a2ensite <site_name>.conf` and `a2dissite <site_name>.conf`
- **Configurations**: `a2enconf <conf_name>` and `a2disconf <conf_name>`

*How they work under the hood:* `a2ensite app.conf` literally executes:
`ln -s /etc/apache2/sites-available/app.conf /etc/apache2/sites-enabled/app.conf`

---

## 3.2 Virtual Hosts Architecture: Name-based vs. IP-based

A **Virtual Host** allows a single Apache HTTPD instance to host dozens or hundreds of distinct websites (e.g., `api.example.com`, `admin.example.com`, `store.example.com`) on a single physical machine.

```
                           Incoming Request: https://api.company.com/v1/users
                                                    |
                                                    v
                                      [ Apache HTTPD Port 443 ]
                                                    |
                     +------------------------------+------------------------------+
                     | Matches IP:Port (0.0.0.0:443)|                              |
                     v                              v                              v
           <VirtualHost *:443>            <VirtualHost *:443>            <VirtualHost *:443>
           ServerName store.company.com   ServerName api.company.com     ServerName admin.company.com
                   |                              |                              |
                [Ignore]                  [EXACT MATCH!]                      [Ignore]
                                                  |
                                                  v
                                      DocumentRoot or ProxyPass
```

### 1. Name-based Virtual Hosting (The Modern Enterprise Standard)
Multiple domains share the **same IP address and port**.
- Apache uses the HTTP request header `Host: api.company.com` (and the TLS `SNI` extension during the handshake) to match against `ServerName` and `ServerAlias`.

### 2. IP-based Virtual Hosting
Each website has a dedicated network interface card (NIC) or secondary IP address:
```apache
<VirtualHost 192.168.1.10:443>
    ServerName payments.example.com
</VirtualHost>

<VirtualHost 192.168.1.20:443>
    ServerName public.example.com
</VirtualHost>
```

### 3. The Default / Fallback VirtualHost Trap
**Rule of Apache Resolution**: When a client requests an unknown domain (or hits the server directly via raw IP `https://203.0.113.50/`), Apache evaluates all loaded VirtualHosts and routes the request to the **first loaded VirtualHost** that matches the port!
- On Ubuntu, files in `sites-enabled/` are evaluated in **alphabetical order**.
- Therefore, `000-default.conf` will catch all unmatched traffic.
- **Production Hardening Practice**: Create a `000-catchall.conf` that immediately returns HTTP `403 Forbidden` or `444 No Response` to drop malicious scanners probing your server by IP:
```apache
# /etc/apache2/sites-available/000-catchall.conf
<VirtualHost *:80>
    ServerName catchall
    ServerAlias *
    <Location />
        Require all denied
    </Location>
</VirtualHost>

<VirtualHost *:443>
    ServerName catchall
    ServerAlias *
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/dummy.crt
    SSLCertificateKeyFile /etc/ssl/private/dummy.key
    <Location />
        Require all denied
    </Location>
</VirtualHost>
```

---

## 3.3 Core Directory Containers, Precedence, and Permissions

Apache controls access using configuration containers. Understanding their **evaluation order** is critical:

### Container Types & Precedence Order
When a request arrives, Apache evaluates configuration containers in this exact sequence:
1. `<Directory>` (standard filesystem paths, evaluated from shortest path to longest path, e.g., `/var` then `/var/www` then `/var/www/app`)
2. `<DirectoryMatch>` (regex filesystem paths)
3. Filesystem `.htaccess` files (if enabled by `AllowOverride`)
4. `<Files>` and `<FilesMatch>` (specific filenames, evaluated concurrently with Directory)
5. `<Location>` and `<LocationMatch>` (evaluated **last**, based on the incoming request URL path)

> **Important Production Warning**: Because `<Location>` is evaluated *after* `<Directory>`, any access restrictions set in `<Directory>` can be accidentally overridden by a loose `<Location>` block!

### The Performance Killer: `AllowOverride` and `.htaccess`
When `AllowOverride All` is set, Apache checks **every single directory in the path** for a `.htaccess` file on **every single incoming HTTP request**.
If a request asks for `/var/www/app/static/images/logo.png`, Apache makes 5 separate disk system calls:
- Checks `/.htaccess`
- Checks `/var/.htaccess`
- Checks `/var/www/.htaccess`
- Checks `/var/www/app/.htaccess`
- Checks `/var/www/app/static/.htaccess`
- Checks `/var/www/app/static/images/.htaccess`

**Production Best Practice**: Always set `AllowOverride None` globally! Put all directives directly in your `<VirtualHost>` configuration files.
```apache
<Directory /var/www/>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

### Access Control Directives (`mod_authz_core`)
- `Require all granted`: Unrestricted public access.
- `Require all denied`: Closes off the directory (returns `403 Forbidden`).
- `Require ip 10.0.0.0/8 192.168.1.0/24`: Restricts access to specific CIDR subnets (ideal for internal management portals like Tomcat Manager).
- `Require not ip 203.0.113.100`: Blacklists a malicious IP.

---

## 3.4 The URL Rewriting Engine (`mod_rewrite`)

`mod_rewrite` is a rule-based rewriting engine that parses and manipulates URLs on the fly using regular expressions.

```
Incoming Request URL
        |
        v
[ RewriteEngine On ]
        |
        +---> [ RewriteCond (Condition 1 True?) ]
                    |
                    v (Yes)
              [ RewriteCond (Condition 2 True?) ]
                    |
                    v (Yes)
              [ RewriteRule (Pattern -> Substitution [FLAGS]) ]
                    |
                    +--> [R=301] -> Send 301 Redirect to Browser
                    +--> [L]     -> Stop processing further rewrite rules
                    +--> [P]     -> Hand over to mod_proxy
                    +--> [END]   -> Terminate rewrite engine completely (no loops)
```

### Anatomy of a Rewrite Rule
```apache
RewriteCond %{VARIABLE_NAME} regex_pattern [FLAGS]
RewriteRule regex_pattern substitution_target [FLAGS]
```

### Common Flags You Must Know for Production
| Flag | Name | Function |
| :--- | :--- | :--- |
| `[L]` | **Last** | Stop processing rules in the current iteration. (Warning: It may loop again on subrequests unless `[END]` is used). |
| `[END]` | **End** | Completely terminates rewriting for this request; prevents internal subrequest loops. |
| `[R=301]` | **Permanent Redirect** | Sends an HTTP `301 Moved Permanently` to the browser, updating search engines and browser caches. |
| `[R=302]` | **Temporary Redirect** | Sends an HTTP `302 Found` to the browser. |
| `[NC]` | **No Case** | Case-insensitive regular expression matching. |
| `[QSA]` | **Query String Append** | Preserves incoming query string parameters (e.g., `?user=john&id=10`) and merges them into the new URL. |
| `[F]` | **Forbidden** | Immediately terminates the request with HTTP `403 Forbidden`. |

### 3 Production Use Cases for `mod_rewrite`

#### Pattern 1: Canonical HTTP to HTTPS Redirection (Port 80 to 443)
```apache
<VirtualHost *:80>
    ServerName app.example.com
    ServerAlias www.app.example.com

    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
</VirtualHost>
```

#### Pattern 2: Enforcing Canonical Domain (Redirect `www` to non-`www`)
```apache
RewriteEngine On
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^ https://%1%{REQUEST_URI} [R=301,L]
```
*(Notice `%1`: In `mod_rewrite`, `%N` references regex capture groups from the preceding `RewriteCond`, whereas `$N` references capture groups from the `RewriteRule`!)*

#### Pattern 3: Single Page Application (SPA / React / Angular) Fallback
If the requested URI does not match a real file or directory on disk, route it to `index.html` so client-side routing works:
```apache
<Directory /var/www/frontend>
    RewriteEngine On
    RewriteBase /
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.html [L]
</Directory>
```

---

## 3.5 Production SSL/TLS Hardening & HTTPS Configuration

Modern production requires disabling outdated SSL/TLS versions, enforcing strong cipher suites, and enabling HTTP Strict Transport Security (HSTS).

### Complete Hardened VirtualHost Configuration
```apache
<VirtualHost *:443>
    ServerName app.example.com
    DocumentRoot /var/www/app/public

    # Enable TLS Engine
    SSLEngine on

    # Certificate Paths
    SSLCertificateFile /etc/ssl/certs/app.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/app.example.com.key
    SSLCertificateChainFile /etc/ssl/certs/intermediate-ca.crt

    # Disable obsolete protocols: Kill SSLv2, SSLv3, TLS 1.0, and TLS 1.1!
    SSLProtocol -all +TLSv1.2 +TLSv1.3

    # Enforce Strong Forward-Secrecy Ciphers
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    SSLHonorCipherOrder off
    SSLSessionTickets off

    # OCSP Stapling (Speeds up SSL handshakes by caching certificate revocation status)
    SSLUseStapling on

    # Enterprise Security Headers (Requires mod_headers)
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
    Header always set X-Frame-Options "DENY"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    # Forwarding to Tomcat API Backend
    ProxyPreserveHost On
    ProxyPass /api/ http://10.0.2.20:8080/api/ retry=2 timeout=30
    ProxyPassReverse /api/ http://10.0.2.20:8080/api/

    ErrorLog ${APACHE_LOG_DIR}/app_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/app_ssl_access.log combined
</VirtualHost>
```

---

## 3.6 Practical Linux Operational Commands

### 1. Test Configuration Syntax
```bash
# Test syntax without touching the running daemon
sudo apache2ctl configtest
# Or
sudo apache2ctl -t
```
*Never run `systemctl restart apache2` before running `configtest`! A typo will crash the running production service.*

### 2. Dump VirtualHost Resolution Mapping
```bash
# View every defined VirtualHost, port, ServerName, config file, and line number
sudo apache2ctl -S
```
*Output sample:*
```text
VirtualHost configuration:
*:80                   app.example.com (/etc/apache2/sites-enabled/app.conf:1)
*:443                  is a NameVirtualHost
         default server catchall (/etc/apache2/sites-enabled/000-catchall.conf:10)
         port 443 namevhost catchall (/etc/apache2/sites-enabled/000-catchall.conf:10)
         port 443 namevhost app.example.com (/etc/apache2/sites-enabled/app.conf:15)
```

### 3. Verify SSL Handshake, SNI, and Expiration Dates from CLI
```bash
# Test TLS connection with explicit SNI
openssl s_client -connect app.example.com:443 -servername app.example.com -tls1_3

# Check certificate expiration date directly from a certificate file
openssl x509 -in /etc/ssl/certs/app.example.com.crt -noout -enddate
```

### 4. Trace HTTP Redirects and Header Responses
```bash
# Follow redirects and show HTTP response codes and headers
curl -IL https://app.example.com
```

---

## 3.7 Common Production Errors and Root Cause Analysis

### Error 1: Infinite Rewrite Loop (`AH00124`)
#### Error Log:
```
[Fri Nov 20 15:30:12.102931 2026] [core:error] [pid 18210:tid 139882] [client 203.0.113.15:49122] AH00124: Request exceeded the limit of 10 internal redirects due to probable configuration error. Use 'LimitInternalRecursion' to increase the limit if necessary. Use 'LogLevel rewrite:trace2' to get detailed debugging.
```
- **Root Cause**: A `RewriteRule` matches its own substitution without a terminal condition or flag. For example:
  `RewriteRule ^(.*)$ /index.php [L]`
  When Apache substitutes `/index.php`, it restarts the rewrite phase internally with the new URI `/index.php`, which matches `^(.*)$` again, recursing until hitting the limit of 10.
- **Fix**: Use `RewriteCond %{REQUEST_URI} !^/index\.php` or use the modern `[END]` flag instead of `[L]`.

### Error 2: Permission Denied (`AH00035` / `403 Forbidden`)
#### Error Log:
```
[Fri Nov 20 15:35:40.401920 2026] [core:error] [pid 18210:tid 139885] (13)Permission denied: [client 203.0.113.15:49128] AH00035: access to /index.html denied (filesystem path '/var/www/app/public/index.html') because search permissions are missing on a component of the path
```
- **Root Cause**: Linux directory traversal requires the **execute (`x`) permission** on *every parent directory* in the path. If `/var/www/app` has permissions `750` owned by `deploy:deploy`, the `www-data` user cannot enter (`cd`) the directory, even if `/var/www/app/public/index.html` is `777`!
- **Fix**: Grant traversal permission to all parent directories:
  ```bash
  sudo chmod o+x /var /var/www /var/www/app /var/www/app/public
  ```

### Error 3: SSL Certificate / Private Key Mismatch (`AH00016`)
#### Error Log:
```
[Fri Nov 20 15:40:02.100912 2026] [ssl:emerg] [pid 19102:tid 14001] AH02565: Certificate and private key app.example.com:443 from /etc/ssl/certs/app.crt and /etc/ssl/private/app.key do not match
```
- **Root Cause**: The public certificate and private key were generated from different cryptographic pairs (e.g., renewed the certificate with a new CSR, but forgot to deploy the matching private key).
- **Verification Command**: Compare the cryptographic MD5 hash of both modulos:
  ```bash
  openssl x509 -noout -modulus -in /etc/ssl/certs/app.crt | openssl md5
  openssl rsa -noout -modulus -in /etc/ssl/private/app.key | openssl md5
  ```
  *If the output hashes do not match 100%, Apache will refuse to start!*

---

## 3.8 Module 3 Assessment & Interview Gate

Answer the following questions to complete Module 3:

### Conceptual Questions
1. If a server receives an HTTPS request for `https://unknown-scanner.com/` (which is pointed at your server's public IP in an attacker's DNS, but not configured in your Apache VirtualHosts), which VirtualHost handles the request, and why?
2. Explain the architectural and disk I/O differences between placing rewrite and authorization directives inside `<VirtualHost>` vs. placing them inside a `.htaccess` file.
3. In `mod_rewrite`, what is the critical difference between the `[L]` flag and the `[END]` flag? In what scenario will `[L]` cause an infinite loop where `[END]` succeeds?

### Practical Linux Questions
4. You made changes to 4 different VirtualHost files in `/etc/apache2/sites-available/`. What exact command sequence do you run to verify there are no syntax errors, verify that no VirtualHost has duplicate `ServerName` collisions, and reload the configuration with **zero dropped active client connections**?
5. An administrator generated a new private key and certificate. Apache fails to start with an SSL error. Write the two shell commands using `openssl` to mathematically verify whether the private key matches the public certificate.

### Real-World Production Troubleshooting Scenario
6. **Scenario**: You deploy a new Vue/React frontend application to `/var/www/frontend`.
   - When users visit the home page `https://app.example.com/`, it loads properly.
   - However, when a user refreshes the page on `https://app.example.com/dashboard` or enters that URL directly into their browser, Apache returns an immediate **HTTP 404 Not Found**.
   - Explain why this occurs from a web server filesystem perspective.
   - Provide the exact Apache configuration block needed to resolve this permanently.

---

# Module 4: Apache Tomcat Architecture & Internals

## 4.1 What is Apache Tomcat?

**Apache Tomcat** is an open-source, pure-Java implementation of the **Jakarta Servlet**, **Jakarta Pages (JSP)**, **Jakarta Expression Language (EL)**, and **Jakarta WebSocket** specifications.

### Technical Classification
- It is **not** a full Java EE / Jakarta EE Application Server (such as WildFly, WebSphere, or WebLogic, which include EJB containers, JMS message brokers, and JTA transaction managers).
- It is a **Servlet Container** (often called a Web Container) that provides an HTTP web server environment in which Java code can run.

### Standalone Tomcat vs. Spring Boot Embedded Tomcat
| Feature | Standalone Tomcat Server | Embedded Tomcat (Spring Boot) |
| :--- | :--- | :--- |
| **Packaging** | Installed directly on OS (`/opt/tomcat`). Deploys `.war` files to `webapps/`. | Packaged inside the application JAR file (`app.jar`) along with all dependencies. |
| **Lifecycle** | Managed as an independent OS systemd service. Multiple WARs can run in one JVM. | Starts and stops with the Java `main()` method: `java -jar app.jar`. 1 app per JVM. |
| **Upgrades** | Tomcat patch can be applied on the OS without recompiling Java source code. | Requires updating `pom.xml`/`build.gradle`, re-running CI/CD, and redeploying the JAR. |
| **Resource Isolation** | Multiple apps share one JVM heap; one app's OutOfMemoryError crashes all apps. | Complete JVM isolation per microservice. Standard in cloud/Kubernetes environments. |

---

## 4.2 Tomcat's Internal Component Hierarchy

Tomcat's architecture follows a strict hierarchical tree defined in `conf/server.xml`. Every component implements the `org.apache.catalina.Lifecycle` interface (with states: `NEW`, `INITIALIZING`, `STARTING`, `STARTED`, `STOPPING`, `STOPPED`, `DESTROYED`).

```
                              [ Server (Port 8005) ]
                                        |
                             +----------+----------+
                             |                     |
                      [ Service: Catalina ]   [ Service: Secondary ]
                             |
         +-------------------+-------------------+
         |                                       |
  [ Connector: Coyote HTTP/1.1 (8080) ]   [ Connector: Coyote AJP (8009) ]
         |                                       |
         +-------------------+-------------------+
                             |
                   [ Engine: Catalina ]
                             |
              +--------------+--------------+
              |                             |
      [ Host: localhost ]          [ Host: api.example.com ]
              |                             |
     +--------+--------+                    +-------+
     |                 |                            |
[ Context: / ]  [ Context: /orders ]        [ Context: /ROOT ]
     |                 |                            |
 [ Wrapper ]      [ Wrapper ]                  [ Wrapper ]
(Servlet: Main)  (Servlet: Spring)            (Servlet: API)
```

### Hierarchy Breakdown:
1. **Server (`org.apache.catalina.Server`)**:
   - The top-level singleton representing the entire Tomcat instance.
   - Listens on an internal shutdown port (default `8005`) for a designated shutdown string (e.g., `SHUTDOWN`).
2. **Service (`org.apache.catalina.Service`)**:
   - An intermediate grouping component that binds **one or more Connectors** to exactly **one Engine**.
3. **Connector (Coyote)**:
   - The network endpoint that accepts incoming TCP connections, parses protocol packets (HTTP/1.1, HTTP/2, AJP), converts them into `HttpServletRequest` / `HttpServletResponse` objects, and hands them off to the Engine.
4. **Engine (`org.apache.catalina.Engine`)**:
   - The top-level request processing container for Catalina. It examines HTTP headers to route the request to the matching child `Host`.
5. **Host (`org.apache.catalina.Host`)**:
   - Represents a Virtual Host (FQDN, e.g., `app.example.com`). Manages an `appBase` directory (e.g., `webapps/`) containing web applications.
6. **Context (`org.apache.catalina.Context`)**:
   - Represents a single deployed **Web Application** (e.g., `/orders` or `/ROOT`). Owns its own isolated ClassLoader, servlet configurations, and session manager.
7. **Wrapper (`org.apache.catalina.Wrapper`)**:
   - The lowest-level container. Represents an **individual Servlet instance** (e.g., Spring's `DispatcherServlet`). Manages the servlet's `init()`, `service()`, and `destroy()` lifecycle.

---

## 4.3 The Coyote Connector: Threading & I/O Models

The **Coyote Connector** is the bridge between the raw network socket and the Java Servlet world.

### Modern Non-Blocking I/O (`Http11NioProtocol`) Mechanics
In Tomcat 9, 10, and 11, the default connector is Java NIO (Non-Blocking I/O):

```
Client TCP Socket (from Apache HTTPD)
           |
           v
    [ Acceptor Thread ]  ---> Calls serverSocket.accept()
           |                  Sets socket to non-blocking
           v
     [ Poller Thread ]   ---> Uses java.nio.channels.Selector (epoll)
           |                  Monitors channels for OP_READ / OP_WRITE
           v (Ready Event)
[ Worker Thread Pool ]   ---> Pulls request from queue
 (ThreadPoolExecutor)         Executes Catalina Pipeline & Servlets
```

1. **Acceptor Thread**: Runs in a continuous loop executing `serverSocketChannel.accept()`. Once a connection is established, it sets the socket to non-blocking mode and hands the channel over to the **Poller**.
2. **Poller Thread**: Runs an internal `java.nio.channels.Selector` (backed by Linux `epoll`). It monitors thousands of registered client sockets without blocking. When data arrives, it creates a `SocketProcessor` task.
3. **Worker Thread Pool (`ThreadPoolExecutor`)**: Picks up the task, parses the HTTP request bytes into `org.apache.coyote.Request`, wraps it in standard `jakarta.servlet.http.HttpServletRequest`, and invokes the Catalina container pipeline.

---

## 4.4 Catalina Pipeline & Valve Architecture

How does a request travel from the `Engine` down to your `Servlet`? Tomcat uses a **Pipeline and Valve** design pattern (similar to Linux shell pipes `|` or the Chain of Responsibility pattern).

Every container (`Engine`, `Host`, `Context`, `Wrapper`) has a dedicated `Pipeline`. A pipeline consists of zero or more intermediate **Valves** and ends with a mandatory **Basic Valve**:

```
Request arrives at Engine Pipeline:
  -> Custom Engine Valve
  -> StandardEngineValve (Basic Valve)
         |
         v
     Hands off to Host Pipeline:
       -> RemoteIpValve (Replaces proxy IP with client IP)
       -> AccessLogValve (Writes to localhost_access_log.txt)
       -> StandardHostValve (Basic Valve)
              |
              v
          Hands off to Context Pipeline:
            -> Authenticator Valve (Basic/Form Auth)
            -> StandardContextValve (Basic Valve)
                   |
                   v
               Hands off to Wrapper Pipeline:
                 -> StandardWrapperValve (Basic Valve)
                        |
                        v
                 [ Filter Chain (Spring Security / CORS) ]
                        |
                        v
                 [ Target Servlet.service() ]
```

### Critical Production Valves in `server.xml`:
1. **`RemoteIpValve`**:
   When Tomcat sits behind Apache HTTPD, all incoming TCP packets originate from Apache's IP (e.g., `10.0.1.10`). Without this valve, `request.getRemoteAddr()` returns `10.0.1.10` instead of the user's real public IP, breaking audit trails, rate limits, and GeoIP detection.
   ```xml
   <Valve className="org.apache.catalina.valves.RemoteIpValve"
          internalProxies="10\.0\.1\.\d+"
          remoteIpHeader="x-forwarded-for"
          protocolHeader="x-forwarded-proto" />
   ```
2. **`ErrorReportValve`**:
   By default, when an unhandled exception occurs, Tomcat prints the full Java exception stack trace and server version on the HTML error page. This is a severe security vulnerability (CWE-209: Information Exposure Through an Error Message).
   ```xml
   <Valve className="org.apache.catalina.valves.ErrorReportValve"
          showReport="false" 
          showServerInfo="false" />
   ```

---

## 4.5 Jasper: The JSP Compiler Engine

If your application uses JavaServer Pages (`.jsp`), **Jasper** compiles them into standard Java servlets on demand:
1. **Translation**: Jasper translates `index.jsp` into a Java source file:
   `$CATALINA_BASE/work/Catalina/localhost/ROOT/org/apache/jsp/index_jsp.java`
2. **Compilation**: Jasper invokes the Java compiler (Eclipse JDT or `javac`) to compile `.java` into `.class` bytecode:
   `index_jsp.class`
3. **Execution**: The compiled class is loaded into the JVM and executed as a normal servlet.

> **Production Best Practice**: Runtime JSP compilation introduces latency on first access and consumes CPU. Pre-compile JSPs during your Maven/Gradle CI/CD build using `jasper-maven-plugin` before packaging the WAR!

---

## 4.6 Directory Structure & Multi-Instance Architecture (`CATALINA_HOME` vs `CATALINA_BASE`)

A standard Tomcat installation contains 7 core directories:

```
/opt/tomcat/
|-- bin/            # Executables & scripts (catalina.sh, setenv.sh, startup.sh)
|-- conf/           # Configuration files (server.xml, web.xml, context.xml)
|-- lib/            # Shared Java libraries (servlet-api.jar, JDBC drivers)
|-- logs/           # Application and container logs (catalina.out, localhost_access_log)
|-- webapps/        # Deployed WAR files and exploded applications
|-- temp/           # JVM scratchpad directory (java.io.tmpdir)
`-- work/           # Jasper compiled JSP servlets and serialized session files
```

### Enterprise Standard: Separating `CATALINA_HOME` and `CATALINA_BASE`
In production, never run multiple Tomcat instances out of a single monolithic directory. Separate the **binaries** from the **runtime state**:

- **`CATALINA_HOME`**: Points to the read-only binaries (`bin/` and `lib/`). Managed by system administrators; shared by all instances.
- **`CATALINA_BASE`**: Points to a specific instance’s configuration and runtime files (`conf/`, `logs/`, `webapps/`, `temp/`, `work/`).

```
                /opt/tomcat-bin/ (CATALINA_HOME)
                |-- bin/
                `-- lib/
                     |
       +-------------+-------------+
       |                           |
       v                           v
/srv/tomcat/instance-1/     /srv/tomcat/instance-2/
(CATALINA_BASE 1)           (CATALINA_BASE 2)
|-- conf/ (Port 8080)       |-- conf/ (Port 8081)
|-- logs/                   |-- logs/
|-- webapps/ (App 1)        |-- webapps/ (App 2)
|-- temp/                   |-- temp/
`-- work/                   `-- work/
```
*Benefits:*
- Upgrading Tomcat is as simple as updating `CATALINA_HOME` symlink; application configs and WARs remain untouched.
- Clean permission boundaries: `CATALINA_HOME` can be read-only (`root`), while `CATALINA_BASE` is owned by the unprivileged `tomcat` service account.

---

## 4.7 Hardened Production Systemd Unit (`tomcat.service`)

Create the dedicated non-root user and group:
```bash
sudo groupadd -r tomcat
sudo useradd -r -g tomcat -d /opt/tomcat -s /sbin/nologin tomcat
```

Create `/etc/systemd/system/tomcat.service`:
```ini
[Unit]
Description=Apache Tomcat Application Server
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh 30 -force

Restart=on-failure
RestartSec=10

# Security Hardening & File Limits
LimitNOFILE=65536
LimitNPROC=4096
PrivateTmp=yes
ProtectSystem=full
ProtectHome=yes
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

Reload and enable:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
```

---

## 4.8 Practical Linux Commands for Tomcat Inspection

### 1. Verify Process Identity, Java Arguments, and PIDs
```bash
# View complete JVM invocation arguments and CATALINA_BASE
ps -ef | grep tomcat
```

### 2. Inspect Active Listening Ports
```bash
# Check Tomcat ports: 8080 (HTTP), 8009 (AJP), and 8005 (Shutdown)
sudo ss -tulpn | grep java
```

### 3. Generate a Live JVM Thread Dump (Diagnose Deadlocks / Freezes)
```bash
# Capture thread dump using jstack from OpenJDK
sudo -u tomcat jstack -l $(pgrep -f "org.apache.catalina.startup.Bootstrap") > /tmp/tomcat_threads.txt

# Inspect thread states
grep -o "java.lang.Thread.State: [A-Z_]*" /tmp/tomcat_threads.txt | sort | uniq -c
```

### 4. Monitor Memory & Garbage Collection in Real Time
```bash
# Show GC statistics every 1000ms (1 second)
sudo -u tomcat jstat -gcutil $(pgrep -f "org.apache.catalina.startup.Bootstrap") 1000
```
*Columns to watch:*
- `S0`, `S1`: Survivor spaces %
- `E`: Eden space %
- `O`: Old generation %
- `FGC`: Full Garbage Collection count (If this number climbs rapidly, you are heading into an OutOfMemory crisis!)

---

## 4.9 Common Production Failure Modes & Root Causes

### 1. `java.lang.NoClassDefFoundError` vs `ClassNotFoundException`
- `ClassNotFoundException`: Occurs when an application explicitly tries to load a class by name (e.g., `Class.forName("com.mysql.cj.jdbc.Driver")`) and the JAR file is completely missing from `WEB-INF/lib` and `$CATALINA_HOME/lib`.
- `NoClassDefFoundError`: The class was present at **compile time**, but at **runtime** either the class itself is missing or a static initializer/dependency of that class failed to load. Often caused by classloader conflicts when the same JAR exists in both `$CATALINA_HOME/lib` and `WEB-INF/lib`.

### 2. Port Already in Use: `BindException: Address already in use`
#### Log in `catalina.out`:
```
SEVERE [main] org.apache.catalina.core.StandardService.initInternal Failed to initialize connector [Connector[HTTP/1.1-8080]]
 org.apache.catalina.LifecycleException: Protocol handler initialization failed
 Caused by: java.net.BindException: Address already in use: 8080
```
- **Root Cause**: A previous Tomcat shutdown command failed or hung, leaving a zombie Java process holding the socket open.
- **Triage**:
  ```bash
  sudo ss -tulpn | grep :8080
  sudo kill -9 <PID>
  ```

### 3. Deployment Failure: WAR Explosion / Context Startup Failure
#### Log in `localhost.YYYY-MM-DD.log`:
```
SEVERE [main] org.apache.catalina.startup.ContextConfig.beforeStart Exception fixing docBase for context [/app]
 java.io.IOException: Unable to create the directory [/opt/tomcat/webapps/app]
```
- **Root Cause**: Filesystem permissions. The `tomcat` OS user does not have write access to the `webapps/` directory, preventing Tomcat from exploding `app.war`.
- **Fix**:
  ```bash
  sudo chown -R tomcat:tomcat /opt/tomcat/webapps /opt/tomcat/work /opt/tomcat/temp /opt/tomcat/logs
  ```

---

## 4.10 Module 4 Assessment & Interview Gate

Answer the following questions to complete Module 4:

### Conceptual Questions
1. Trace the precise path of an HTTP request as it flows through Tomcat's internal containers (`Server`, `Service`, `Connector`, `Engine`, `Host`, `Context`, `Wrapper`). At which component does the transition from raw network protocol to standard Java Servlet API occur?
2. What is the difference between `CATALINA_HOME` and `CATALINA_BASE`? Why does running a production environment with multiple instances sharing `CATALINA_HOME` while having separate `CATALINA_BASE` directories improve operational security and maintainability?
3. What is the function of the `RemoteIpValve` in `server.xml`? What specific application bugs occur if this valve is omitted when Tomcat is placed behind an Apache reverse proxy?

### Practical Linux Questions
4. You run `systemctl stop tomcat`, but when you run `systemctl start tomcat`, the service crashes immediately with `java.net.BindException: Address already in use: 8080`. Write the exact terminal commands to identify the orphaned process holding port 8080 open, terminate it safely, and bring Tomcat back online.
5. In high-security enterprise environments, port `8005` (the Tomcat shutdown port) can be a vector for Denial of Service if bound to `0.0.0.0` or configured with a predictable string. How do you configure `server.xml` to completely disable the network shutdown port?

### Real-World Production Troubleshooting Scenario
6. **Scenario**: A developer deploys a new application WAR (`analytics.war`) to `/opt/tomcat/webapps/`.
   - `systemctl restart tomcat` runs without errors.
   - However, when testing `curl -I http://localhost:8080/analytics`, Tomcat returns **HTTP 404 Not Found**.
   - Other applications on the same Tomcat instance (e.g., `http://localhost:8080/orders`) work fine.
   - What specific log files in `/opt/tomcat/logs/` do you check first?
   - What are the three most common application startup failure root causes that result in this symptom?

---

# Module 5: Tomcat Production Configuration & JVM Tuning

## 5.1 Core Configuration Files Deep Dive

In enterprise environments, Tomcat is configured across 5 distinct configuration files:

```
$CATALINA_BASE/
|-- conf/
|   |-- server.xml            # Master infrastructure: Connectors, Engines, Hosts, Ports
|   |-- context.xml           # JNDI Resources (JDBC Pools) shared across all webapps
|   |-- web.xml               # Global Servlet mappings, MIME types, Default Session Timeout
|   |-- catalina.properties   # ClassLoader definitions, JAR scanning exclusions
|   `-- logging.properties    # Tomcat JULI logging configuration
`-- bin/
    `-- setenv.sh             # Runtime script: JVM memory (-Xms/-Xmx), GC flags, system properties
```

### 1. `conf/server.xml`
Defines the topology of the server. In production, we replace standalone `<Connector>` thread pools with an explicit, shared `<Executor>` component to control thread allocation across multiple connectors.

### 2. `conf/context.xml`
Defines runtime configurations applied to all deployed contexts. This is the ideal location to define enterprise JNDI Database Connection Pools (`Resource`), preventing database credentials from being hardcoded inside application WAR files.

### 3. `conf/web.xml`
The global deployment descriptor. Key configurations include:
- **Default Session Timeout**: `<session-timeout>30</session-timeout>` (in minutes).
- **Default Servlet**: Handles static content if requests bypass Apache HTTPD.
- **Listing Disabling**: Ensures directory browsing is disabled by verifying `listings` is set to `false` for `org.apache.catalina.servlets.DefaultServlet`.

### 4. `conf/catalina.properties` (Massive Startup Optimization)
By default, Tomcat scans every single JAR on the classpath for TLDs and Servlet annotations. This can cause startup to take 3-5 minutes on large enterprise applications.
**Optimization**: Add third-party utility JARs (AWS SDK, Jackson, Commons, Logging) to `tomcat.util.scan.StandardJarScanFilter.jarsToSkip`:
```properties
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=aws-*.jar,jackson-*.jar,commons-*.jar,log4j-*.jar,slf4j-*.jar
```
*Result: Tomcat startup time drops from 180 seconds to under 15 seconds.*

### 5. `bin/setenv.sh`
**Golden Rule of DevOps**: Never edit `bin/catalina.sh` or `bin/startup.sh`. When upgrading Tomcat, those files are overwritten. Always place custom environment variables, JVM memory parameters, and system properties inside `bin/setenv.sh`. Tomcat executes this script automatically if it exists.

---

## 5.2 Sizing Connectors, Thread Pools & Queues

Understanding Tomcat's queuing architecture is critical to avoid artificial connection drops:

```
Incoming TCP Connection (from Apache HTTPD)
                    |
                    v
  [ OS Accept Backlog: acceptCount = 200 ]  ---> If FULL: Kernel drops SYN / resets connection
                    |
                    v
 [ Active TCP Sockets: maxConnections = 10000 ] -> Managed by Poller (epoll)
                    |
                    v
 [ Request Task Queue: maxQueueSize = 500 ]  ---> Holds requests waiting for a free worker
                    |
                    v
[ Worker Thread Pool: maxThreads = 300 ]     ---> Active execution of Servlet code
                    |
                    v
 [ Idle Worker Floor: minSpareThreads = 50 ] ---> Keep-alive floor
```

### The Sizing Directives Explained:
- **`acceptCount`**: The OS TCP backlog queue size for the server socket (`somaxconn`). If all worker threads are busy and the request queue is full, incoming connections wait in this queue. If this fills up, the OS sends `Connection Refused`. (Recommended: `100 - 200`).
- **`maxConnections`**: The maximum number of simultaneous TCP connections that the Coyote NIO Poller can hold open. Because NIO uses `epoll`, idle connections consume almost no CPU. (Recommended: `8192 - 10000`).
- **`maxThreads`**: The maximum number of Java worker threads allocated to execute `Servlet.service()`. Each thread consumes OS stack memory (typically 1 MB via `-Xss1m`). (Recommended: `200 - 400` depending on CPU cores and backend latency).
- **`minSpareThreads`**: The minimum number of worker threads kept alive, waiting for incoming requests. (Recommended: `25 - 50`).

### Production Configuration Example (`conf/server.xml`):
```xml
<!-- Shared High-Performance Executor Pool -->
<Executor name="tomcatThreadPool" 
          namePrefix="catalina-exec-"
          maxThreads="300" 
          minSpareThreads="50" 
          maxQueueSize="200" 
          prestartminSpareThreads="true" />

<!-- Coyote Connector linked to the Executor -->
<Connector executor="tomcatThreadPool"
           port="8080" 
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           connectionTimeout="20000"
           keepAliveTimeout="15000"
           maxKeepAliveRequests="100"
           maxConnections="10000"
           acceptCount="150"
           enableLookups="false"
           URIEncoding="UTF-8" />
```

---

## 5.3 JNDI Database Connection Pooling (`conf/context.xml`)

Applications should borrow database connections from a managed pool rather than opening fresh TCP sockets per query.

```xml
<Context>
    <!-- WatchedResource ensures dynamic reload on web.xml changes -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>

    <!-- Production PostgreSQL Connection Pool -->
    <Resource name="jdbc/AppDB" 
              auth="Container"
              type="javax.sql.DataSource"
              factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
              driverClassName="org.postgresql.Driver"
              url="jdbc:postgresql://10.0.3.50:5432/app_production"
              username="app_user"
              password="SECRET_DB_PASSWORD"
              
              initialSize="10"
              maxTotal="100"
              maxIdle="30"
              minIdle="10"
              maxWaitMillis="10000"
              
              testOnBorrow="true"
              validationQuery="SELECT 1"
              validationInterval="30000"
              
              removeAbandoned="true"
              removeAbandonedTimeout="60"
              logAbandoned="true" />
</Context>
```

### Critical Pool Sizing Directives:
- `maxTotal="100"`: Maximum number of active database sockets that can be opened. Never set this higher than PostgreSQL's `max_connections` divided by the number of Tomcat instances!
- `maxWaitMillis="10000"`: If all 100 connections are checked out, Tomcat waits up to 10 seconds for one to be returned. If none becomes available, it throws `GetConnectionTimeoutException` instead of hanging forever.
- `testOnBorrow="true"` & `validationQuery="SELECT 1"`: Tests the connection before handing it to Java code. If a database firewall or AWS NAT Gateway dropped the idle TCP connection, Tomcat silently discards it and creates a fresh socket, preventing `SocketException: Connection reset by peer`.
- `removeAbandoned="true"` & `removeAbandonedTimeout="60"`: **The ultimate leak protector**. If sloppy application code borrows a connection and forgets to call `connection.close()` in a `finally` block, Tomcat forcibly reclaims the connection after 60 seconds and logs the offending Java stack trace (`logAbandoned="true"`).

---

## 5.4 JVM Memory Layout, Heap Sizing & Garbage Collection Tuning

```
+-------------------------------------------------------------------------------------------------+
|                                     JVM MEMORY ADDRESS SPACE                                    |
|                                                                                                 |
|  +-------------------------------------------------------------------------------------------+  |
|  |                                     HEAP MEMORY (-Xms / -Xmx)                             |  |
|  |  +------------------------------------------------+  +---------------------------------+  |  |
|  |  |                 YOUNG GENERATION               |  |          OLD GENERATION         |  |  |
|  |  |  +---------------+  +--------+  +--------+     |  |          (TENURED)              |  |  |
|  |  |  |  Eden Space   |  |   S0   |  |   S1   |     |  |  Long-lived objects             |  |  |
|  |  |  |  New objects  |  | Survivor Spaces    |     |  |  Session data, cache, pools     |  |  |
|  |  |  +---------------+  +--------+  +--------+     |  +---------------------------------+  |  |
|  |  +------------------------------------------------+                                       |  |
|  +-------------------------------------------------------------------------------------------+  |
|                                                                                                 |
|  +----------------------------------+  +----------------------------------+  +---------------+  |
|  |         METASPACE (Native)       |  |       THREAD STACKS (-Xss)       |  |  CODE CACHE   |  |
|  |  Class metadata, method byte-    |  |  1 MB per thread                 |  |  JIT compiled |  |
|  |  code. (-XX:MaxMetaspaceSize)    |  |  (300 threads = 300 MB)          |  |  native code  |  |
|  +----------------------------------+  +----------------------------------+  +---------------+  |
+-------------------------------------------------------------------------------------------------+
```

### Production Rules for JVM Tuning:
1. **Always Set `-Xms` Equal to `-Xmx`**:
   - If `-Xms2g` and `-Xmx8g`, the JVM will start at 2 GB and continuously request memory from the Linux kernel as load increases. Each allocation triggers a JVM GC pause and OS memory fragmentation. Pre-allocating the full heap at startup eliminates runtime allocation overhead.
2. **Cap the Metaspace**:
   - In Java 8+, PermGen was replaced by **Metaspace**, which allocates from native OS RAM. If left unbounded, dynamic class generation (Spring CGLIB, Hibernate) can leak and exhaust the host's entire physical RAM. Always set `-XX:MaxMetaspaceSize=512m`.
3. **Use G1GC (Garbage-First Collector)**:
   - Standard in modern OpenJDK 17 and 21. Divides the heap into hundreds of small regions. Prioritizes regions containing the most garbage, achieving predictable latency without long "Stop-The-World" pauses.
4. **Crash Recovery & Fast Failover**:
   - `-XX:+ExitOnOutOfMemoryError`: When a JVM throws `OutOfMemoryError`, memory is corrupted and threads hang. **Do not let it linger!** This flag instructs the JVM to terminate immediately. Systemd or your AWS Auto Scaling group will automatically restart it, and upstream Apache HTTPD will failover traffic to surviving nodes.
   - `-XX:+HeapDumpOnOutOfMemoryError`: Automatically writes a complete memory snapshot (`.hprof`) to disk before crashing, enabling developers to find the exact memory leak in Eclipse Memory Analyzer (MAT).

---

## 5.5 The Gold-Standard `bin/setenv.sh` Configuration

Create `/opt/tomcat/bin/setenv.sh` (owned by `tomcat:tomcat`, permissions `750`):

```bash
#!/bin/sh
# ==============================================================================
# Enterprise Production Tomcat JVM Settings (Ubuntu 22.04/24.04 - OpenJDK 17/21)
# ==============================================================================

# 1. Heap Memory Allocation (Example: Server with 8 GB Total RAM, 4 GB for Tomcat)
export CATALINA_OPTS="$CATALINA_OPTS -Xms4096m -Xmx4096m"

# 2. Metaspace & Thread Stack Sizing
export CATALINA_OPTS="$CATALINA_OPTS -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -Xss1m"

# 3. Modern Garbage Collection (G1GC) Tuning
export CATALINA_OPTS="$CATALINA_OPTS -XX:+UseG1GC"
export CATALINA_OPTS="$CATALINA_OPTS -XX:MaxGCPauseMillis=200"
export CATALINA_OPTS="$CATALINA_OPTS -XX:InitiatingHeapOccupancyPercent=45"
export CATALINA_OPTS="$CATALINA_OPTS -XX:G1ReservePercent=15"

# 4. OutOfMemory Crash Handling & Diagnostics
export CATALINA_OPTS="$CATALINA_OPTS -XX:+HeapDumpOnOutOfMemoryError"
export CATALINA_OPTS="$CATALINA_OPTS -XX:HeapDumpPath=/opt/tomcat/logs/heapdump.hprof"
export CATALINA_OPTS="$CATALINA_OPTS -XX:+ExitOnOutOfMemoryError"

# 5. Unified GC Logging (Java 17/21)
export CATALINA_OPTS="$CATALINA_OPTS -Xlog:gc*,gc+phases=debug:file=/opt/tomcat/logs/gc.log:time,uptime,pid:filecount=5,filesize=50M"

# 6. Performance & Security Flags
export CATALINA_OPTS="$CATALINA_OPTS -Djava.awt.headless=true"
export CATALINA_OPTS="$CATALINA_OPTS -Djava.security.egd=file:/dev/./urandom"
export CATALINA_OPTS="$CATALINA_OPTS -Dfile.encoding=UTF-8"
```

---

## 5.6 Practical Linux & JVM Commands for Performance Triage

### 1. View Active JVM Flags of a Running Tomcat Instance
```bash
# Verify active memory limits, GC settings, and JVM options
sudo -u tomcat jcmd $(pgrep -f "org.apache.catalina.startup.Bootstrap") VM.flags
```

### 2. Inspect Real-Time Heap Occupancy Breakdown
```bash
# Display detailed Eden, Survivor, Old Gen, and Metaspace usage
sudo -u tomcat jcmd $(pgrep -f "org.apache.catalina.startup.Bootstrap") GC.heap_info
```

### 3. Identify Top Memory-Consuming Java Classes
```bash
# Print a live class histogram showing which objects occupy the most heap RAM
sudo -u tomcat jcmd $(pgrep -f "org.apache.catalina.startup.Bootstrap") GC.class_histogram | head -n 25
```

### 4. Command-Line GC Log Analysis
```bash
# Extract GC pause durations exceeding 50ms from gc.log
grep -E 'Pause Young|Pause Full' /opt/tomcat/logs/gc.log | awk '{if ($(NF) > 0.050) print $0}'
```

---

## 5.7 Common Production Failure Modes & Root Causes

### 1. `java.lang.OutOfMemoryError: Java heap space`
- **Symptom**: Application stops responding. Apache error log shows `502 Bad Gateway`.
- **Root Cause**: The application allocated objects faster than G1GC could reclaim them, or an unbounded collection (e.g., `static List<UserSession>`) held references permanently.
- **Triage**: Inspect the generated `heapdump.hprof` using Eclipse MAT or VisualVM. Look for the "Leak Suspects" report.

### 2. `java.lang.OutOfMemoryError: Metaspace`
- **Symptom**: Server crashes during a rolling application deployment or hot-reload.
- **Root Cause**: When a web application is redeployed without restarting the JVM, its ClassLoader should be garbage collected. If application threads or JDBC drivers fail to deregister, the old ClassLoader cannot be reclaimed (**ClassLoader Leak**). Repeated redeployments exhaust Metaspace.
- **Remediation**: Always perform a full Tomcat restart (`systemctl restart tomcat`) during production deployments instead of relying on hot-reloading.

### 3. Thread Pool Exhaustion
#### Log in `catalina.out`:
```
SEVERE [catalina-exec-298] org.apache.tomcat.util.threads.ThreadPoolExecutor.logFull
 All threads (300) are currently busy, waiting. Increase maxThreads (300) or check the severity of social calls.
```
- **Root Cause**: Worker threads are blocked waiting for an external downstream service (e.g., slow SQL query or third-party payment API without read timeouts).
- **Triage**: Capture a thread dump via `jstack`. Count how many threads are in `TIMED_WAITING` or `BLOCKED` states and identify which database query or socket call they are waiting on.

---

## 5.8 Module 5 Assessment & Interview Gate

Answer the following questions to complete Module 5:

### Conceptual Questions
1. In `server.xml`, what is the precise relationship between `acceptCount`, `maxConnections`, and `maxThreads`? If all `maxThreads` are occupied and `maxConnections` has reached its ceiling, what happens to the next incoming TCP connection?
2. Why is setting `-Xms` equal to `-Xmx` considered an enterprise best practice in production, and what performance penalty does an application pay if `-Xms` is left at its default low value while `-Xmx` is set to 8 GB?
3. What is the operational purpose of configuring both `-XX:+HeapDumpOnOutOfMemoryError` and `-XX:+ExitOnOutOfMemoryError` simultaneously in `setenv.sh`?

### Practical Linux Questions
4. You are configuring Tomcat on an AWS EC2 instance with 16 GB of RAM dedicated exclusively to Tomcat. How would you size `-Xms`, `-Xmx`, and `-XX:MaxMetaspaceSize` in `setenv.sh`, leaving adequate headroom for OS kernel buffers, thread stacks (`-Xss1m` for 300 threads), and JVM native memory?
5. A production Tomcat server is running slowly. Write the exact terminal command using `jcmd` or `jstat` to verify in real time whether the JVM is experiencing Stop-The-World Full Garbage Collection pauses.

### Real-World Production Troubleshooting Scenario
6. **Scenario**: During a marketing event, users receive intermittent `504 Gateway Timeout` errors.
   - Apache HTTPD access logs report `%D` roundtrip times exceeding 30 seconds for `/api/checkout`.
   - You log in to Tomcat and check `catalina.out`. You discover the following error message repeating:
     `org.apache.tomcat.jdbc.pool.PoolExhaustedException: [catalina-exec-182] Timeout: Pool empty. Unable to fetch a connection in 10000 msec, none available[size:100; busy:100; idle:0; waiting:48]`
   - What is the exact root cause of this failure?
   - What two diagnostic steps would you take immediately to identify *why* all 100 database connections are stuck in `busy` state?
   - What parameter in `context.xml` protects the connection pool from leaking if developers forget to close connections in code?

---

# Module 6: Enterprise Integration (HTTPD to Tomcat via mod_proxy & AJP)

## 6.1 The Reverse Proxy Integration Architecture

In high-availability enterprise environments, Apache HTTPD acts as the front-end reverse proxy and Layer 7 load balancer, distributing traffic across an isolated cluster of backend Apache Tomcat nodes.

```
                              [ PUBLIC CLIENTS ]
                                      |
                                      | HTTPS (Port 443)
                                      v
                             [ Route 53 / Anycast ]
                                      |
                                      v
                        [ AWS ALB / Hardware Firewall ]
                                      |
                                      | HTTPS (Port 443)
                                      v
+-----------------------------------------------------------------------------------------+
|                              WEB TIER: APACHE HTTPD (DMZ)                               |
|  - IP: 10.0.1.10                                                                        |
|  - SSL Termination (OpenSSL native C engine)                                            |
|  - Static Content Delivery (/var/www/app/static) via sendfile                           |
|  - mod_proxy_balancer with bybusyness load balancing                                    |
|  - Active Health Checking (mod_proxy_hcheck)                                            |
|  - Sticky Session Routing (JSESSIONID cookie inspection)                                |
+-----------------------------------------------------------------------------------------+
                                      |
                                      | Internal HTTP (Port 8080)
                                      | Keep-Alive Connection Pool
                                      | Private VPC Subnet
                                      v
+-----------------------------------------------------------------------------------------+
|                                APPLICATION CLUSTER TIER                                 |
|                                                                                         |
|  +----------------------------------+     +----------------------------------+          |
|  |     TOMCAT NODE 1 (Active)       |     |     TOMCAT NODE 2 (Active)       |          |
|  |  - IP: 10.0.2.21:8080            |     |  - IP: 10.0.2.22:8080            |          |
|  |  - jvmRoute: "node1"             |     |  - jvmRoute: "node2"             |          |
|  |  - Sizing: maxThreads=300        |     |  - Sizing: maxThreads=300        |          |
|  +----------------------------------+     +----------------------------------+          |
|                     \                                     /                             |
|                      \                                   /                              |
|                       +-----------------+---------------+                               |
|                                         |                                               |
|                                         v                                               |
|                       +----------------------------------+                              |
|                       |    TOMCAT NODE 3 (Hot Standby)   |                              |
|                       |  - IP: 10.0.2.23:8080            |                              |
|                       |  - jvmRoute: "node3" (status=+H) |                              |
|                       +----------------------------------+                              |
+-----------------------------------------------------------------------------------------+
                                      |
                                      | JDBC Connection Pool (HikariCP / Tomcat Pool)
                                      | TCP Port 5432
                                      v
+-----------------------------------------------------------------------------------------+
|                                    DATABASE TIER                                        |
|  PostgreSQL Primary (10.0.3.50) <--- Streaming Replication ---> Replica (10.0.3.51)     |
+-----------------------------------------------------------------------------------------+
```

---

## 6.2 Communication Protocols: `mod_proxy_http` vs. `mod_jk` / `AJP`

Historically, Apache HTTPD communicated with Tomcat using the **Apache JServ Protocol (AJP)** via the `mod_jk` plugin. In modern cloud and enterprise DevOps, **`mod_proxy_http` is the gold standard**.

### What is AJP (Apache JServ Protocol)?
- A binary packet-based protocol (typically running on port `8009`) designed in the early 2000s when Java's HTTP header parsing was slow.
- AJP translates HTTP headers into 2-byte binary integer codes (e.g., `0xA001` for `Host`, `0xA002` for `Accept-Encoding`), passing pre-parsed credentials and socket attributes directly into the JVM.

### The Security Vulnerability: Ghostcat (CVE-2020-1938)
In February 2020, a critical vulnerability named **Ghostcat** was discovered in the AJP connector.
- **Vulnerability**: By default, Tomcat’s AJP connector listened on `0.0.0.0:8009` with no password authentication. Attackers could send crafted AJP requests to read any file from the Tomcat web application (including `WEB-INF/web.xml` and database passwords) or execute arbitrary code via JSP file inclusion.
- **Tomcat's Fix**: In Tomcat 9.0.31+ and 8.5.51+, AJP is commented out by default. If enabled, it binds strictly to `127.0.0.1`, enforces `secretRequired="true"`, and requires a shared secret password.

### Why `mod_proxy_http` is Preferred in Modern Production:
1. **Zero External Compilation**: `mod_proxy_http` is built into Apache HTTPD (`a2enmod proxy_http`). `mod_jk` requires downloading source code and compiling `mod_jk.so` against Apache development headers (`apxs`).
2. **Observability & Tooling**: `mod_proxy_http` speaks standard HTTP. Sysadmins can inspect packets using standard tools (`tcpdump`, `curl`, Wireshark, AWS VPC Flow Logs). AJP packets require specialized binary decoders.
3. **HTTP/2 & Cloud Native**: `mod_proxy_http` seamlessly bridges modern HTTP/2 frontends with microservice and container architectures.

---

## 6.3 Tuning `mod_proxy`: Connection Pooling & Reliability Directives

A common novice mistake is writing a bare `ProxyPass / http://backend:8080/`. Under high traffic, this opens and closes thousands of ephemeral TCP connections, exhausting Linux sockets and triggering `TIME_WAIT` socket starvation.

### Crucial Worker Parameters Explained:
- `retry=N`: If a backend Tomcat node fails to respond, Apache marks it "broken" and routes traffic to other nodes. `retry=5` tells Apache to wait 5 seconds before trying the broken node again. (Default is a catastrophic 60 seconds!).
- `timeout=N`: How long Apache waits for Tomcat to write response bytes before terminating the connection with `504 Gateway Timeout`. Set to match or exceed application SLA (e.g., `timeout=30`).
- `ping=N`: Configures a 2-second TCP/HTTP handshake probe (`ping=2`). Apache verifies the socket is responsive before streaming the client's request payload, eliminating dropped POST requests.
- `keepalive=On`: Forces Apache to maintain persistent, reusable TCP connection pools to Tomcat, reducing CPU usage and eliminating TCP 3-way handshake latency.
- `acquire=N`: How long Apache waits (in milliseconds) to acquire an idle socket from its proxy connection pool before returning `503 Service Unavailable`.

---

## 6.4 Load Balancing with `mod_proxy_balancer`

`mod_proxy_balancer` provides Layer 7 load balancing across a cluster of Tomcat instances.

### Load Balancing Algorithms (`lbmethod`):
1. **`byrequests` (`lbmethod=byrequests`)**:
   - Classic weighted round-robin. Allocates requests equally or according to weights (`loadfactor=1`, `loadfactor=2`).
2. **`bytraffic` (`lbmethod=bytraffic`)**:
   - Balances traffic based on the total number of bytes transferred. Useful for file download services.
3. **`bybusyness` (`lbmethod=bybusyness`)**:
   - **The Enterprise Standard for Java Backends**. Dynamically routes each new request to the Tomcat worker that currently has the **fewest active, concurrent requests**.
   - If Node 1 is busy executing a heavy database query for 3 seconds, incoming requests are automatically diverted to Node 2!
4. **`heartbeat` (`lbmethod=heartbeat`)**:
   - Reads multicasted cluster metrics to route based on actual server CPU/RAM load.

---

## 6.5 Sticky Sessions (Session Persistence)

Most traditional Java enterprise applications store user login states and shopping carts inside the JVM memory (`HttpSession`).
If User A logs in on **Node 1**, and their next click is routed to **Node 2**, Node 2 does not have their session in RAM, and the user is unexpectedly logged out!

### How Sticky Sessions Work via `JSESSIONID`:
```
1. Client connects -> Apache forwards to Node 1 (via bybusyness)
2. Node 1 creates session and appends its jvmRoute:
   Set-Cookie: JSESSIONID=7A9F1B2C3D4E.node1; Path=/; HttpOnly; Secure
3. Client Browser stores cookie and sends it on subsequent requests
4. Apache mod_proxy_balancer parses the .node1 suffix:
   Extracts route="node1" -> Forwards directly to Node 1!
```

### Configuration Requirements:
- **Tomcat Side (`server.xml`)**: Must define the `jvmRoute` attribute on the `<Engine>`:
  ```xml
  <!-- On Node 1 -->
  <Engine name="Catalina" defaultHost="localhost" jvmRoute="node1">

  <!-- On Node 2 -->
  <Engine name="Catalina" defaultHost="localhost" jvmRoute="node2">
  ```
- **Apache Side (`app.conf`)**:
  ```apache
  ProxyPass / balancer://tomcatcluster/ stickysession=JSESSIONID|jsessionid scolonpathdelimiter=On
  <Proxy balancer://tomcatcluster>
      BalancerMember http://10.0.2.21:8080 route=node1
      BalancerMember http://10.0.2.22:8080 route=node2
  </Proxy>
  ```

---

## 6.6 Active Health Checking (`mod_proxy_hcheck`)

In default Apache setups, health checking is **passive**: Apache only discovers a Tomcat node is dead after a real user request fails (resulting in an ugly error or a 30-second delay for that user).
With **`mod_proxy_hcheck`**, Apache proactively monitors backend health out-of-band:

```apache
# Define what constitutes a healthy response
<Proxy "balancer://tomcatcluster">
    # Active Health Check: Query /health every 5 seconds
    # If 2 consecutive checks pass -> Mark UP
    # If 3 consecutive checks fail -> Mark DOWN
    BalancerMember http://10.0.2.21:8080 route=node1 hcmethod=GET hcuri=/health hcinterval=5 hcpasses=2 hcfails=3
    BalancerMember http://10.0.2.22:8080 route=node2 hcmethod=GET hcuri=/health hcinterval=5 hcpasses=2 hcfails=3
</Proxy>
```

---

## 6.7 Zero-Downtime Rolling Deployments

How do we deploy a new version of our application during peak hours without dropping active customer sessions?

### The 4-Step Rolling Upgrade Playbook:
1. **Drain Node 1 (`status=+D`)**:
   - Access Apache's `balancer-manager` (via web UI or CLI).
   - Set Node 1 to **Drain Mode (`status=+D`)**.
   - *Result:* Node 1 receives **no new sessions**. However, existing users who already have a `JSESSIONID` ending in `.node1` are permitted to finish their checkout transactions!
2. **Deploy on Node 1**:
   - Once active sessions hit zero (or after a 10-minute grace period), stop Tomcat on Node 1: `sudo systemctl stop tomcat`.
   - Deploy the new WAR, run database migrations, and start Tomcat: `sudo systemctl start tomcat`.
   - Test locally: `curl -I http://127.0.0.1:8080/health`.
3. **Re-enable Node 1 (`status=-D`)**:
   - In `balancer-manager`, remove Drain mode. Node 1 is now active and taking traffic on the new version.
4. **Repeat for Node 2**:
   - Drain Node 2, upgrade, verify, and bring it back online.
   - **Zero customer downtime. Zero 502 errors.**

---

## 6.8 Complete Production Working Configurations

### 1. Apache HTTPD VirtualHost with Load Balancer & Health Checks (`/etc/apache2/sites-available/app.conf`)

```apache
# Load Required Modules
# sudo a2enmod proxy proxy_http proxy_balancer lbmethod_bybusyness proxy_hcheck headers ssl

<VirtualHost *:443>
    ServerName app.example.com
    ServerAdmin devops@example.com

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/app.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/app.example.com.key
    SSLCertificateChainFile /etc/ssl/certs/intermediate-ca.crt
    SSLProtocol -all +TLSv1.2 +TLSv1.3

    # Fast Static Asset Serving directly by Apache
    DocumentRoot /var/www/app/static
    <Directory /var/www/app/static>
        Require all granted
        Options -Indexes +FollowSymLinks
        AllowOverride None
    </Directory>

    # Strip Untrusted Client Headers & Inject Trusted Reverse Proxy Headers
    RequestHeader unset X-Forwarded-For
    RequestHeader unset X-Forwarded-Proto
    RequestHeader unset X-Forwarded-Port
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"
    ProxyPreserveHost On

    # Configure the High-Availability Tomcat Cluster
    <Proxy "balancer://tomcatcluster">
        # Primary Active Node 1
        BalancerMember http://10.0.2.21:8080 route=node1 loadfactor=1 \
            retry=5 timeout=30 keepalive=On ping=2 acquire=3000 \
            hcmethod=GET hcuri=/health hcinterval=5 hcpasses=2 hcfails=3

        # Primary Active Node 2
        BalancerMember http://10.0.2.22:8080 route=node2 loadfactor=1 \
            retry=5 timeout=30 keepalive=On ping=2 acquire=3000 \
            hcmethod=GET hcuri=/health hcinterval=5 hcpasses=2 hcfails=3

        # Hot Standby Node 3 (Only receives traffic if BOTH Node 1 and Node 2 are down!)
        BalancerMember http://10.0.2.23:8080 route=node3 status=+H \
            retry=10 timeout=30 keepalive=On ping=2

        # Cluster Settings: bybusyness algorithm + sticky sessions
        ProxySet lbmethod=bybusyness stickysession=JSESSIONID|jsessionid scolonpathdelimiter=On
    </Proxy>

    # Forward Dynamic Application Requests
    ProxyPass /static !
    ProxyPass / balancer://tomcatcluster/
    ProxyPassReverse / balancer://tomcatcluster/

    # Protected Balancer Manager UI (Restricted to Internal DevOps VPN)
    <Location /balancer-manager>
        SetHandler balancer-manager
        Require ip 10.0.0.0/16 127.0.0.1
    </Location>

    # Detailed Microsecond Logging
    ErrorLog ${APACHE_LOG_DIR}/app_error.log
    CustomLog ${APACHE_LOG_DIR}/app_access.log "%h %{BALANCER_WORKER_ROUTE}e %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D"
</VirtualHost>
```

---

## 6.9 Practical Linux Verification Commands

### 1. Test Load Balancing & Route Distribution from CLI
```bash
# Run 10 sequential requests and inspect which Tomcat worker handled each request
for i in {1..10}; do
  curl -s -k https://app.example.com/api/info | grep -o '"worker":"[^"]*"'
done
```

### 2. Verify Sticky Session Cookie Routing
```bash
# Store cookie from first request in cookie jar, then make 5 follow-up calls
curl -k -c /tmp/cookies.txt https://app.example.com/api/login
for i in {1..5}; do
  curl -k -b /tmp/cookies.txt -s https://app.example.com/api/user/profile | grep -o 'node[1-2]'
done
```
*Verification: All 5 follow-up requests must return the exact same node (e.g., `node1`), proving sticky session persistence is functioning!*

### 3. Monitor Active TCP Sockets to Tomcat Nodes
```bash
# Show current established connections to backend Tomcat IP subnets
ss -tan '( dport = :8080 and dst 10.0.2.0/24 )' | grep ESTAB | wc -l
```

---

## 6.10 Common Production Integration Failures & Root Causes

### 1. `503 Service Unavailable: All workers are in error state`
#### Log in `app_error.log`:
```
[Fri Nov 20 16:15:30.123910 2026] [proxy_balancer:emerg] [pid 12901:tid 14012] AH01177: balancer://tomcatcluster: All workers are in error state for route balancer://tomcatcluster
```
- **Root Cause**: Both Node 1 and Node 2 failed health checks or timed out simultaneously. Because `retry=60` was set by default, Apache refuses to send traffic to either node for 60 seconds.
- **Fix**: Set `retry=5` or `retry=2`. Inspect Tomcat logs to see why JVMs froze (DB lock or GC pause).

### 2. Random User Logouts / Broken Sticky Sessions
- **Symptom**: Users complain they are kicked back to the login screen on every 2nd or 3rd click.
- **Root Cause**: The `route` parameter on Apache's `BalancerMember` does not match Tomcat's `jvmRoute` attribute in `server.xml`. (e.g., Apache has `route=node1`, but Tomcat's `server.xml` has `jvmRoute="tomcat-01"`). Apache cannot route the cookie and falls back to round-robin.

---

## 6.11 Module 6 Assessment & Interview Gate

Answer the following questions to complete Module 6:

### Conceptual Questions
1. Why is the `bybusyness` load balancing algorithm mathematically superior to `byrequests` (round-robin) in an enterprise environment where some Java API endpoints execute fast in-memory lookups (5ms) while others execute heavy database reporting queries (2000ms)?
2. Explain the difference between **Session Persistence (Sticky Sessions)** and **Session Replication (Tomcat Clustering via DeltaManager)**. What happens to an active shopping cart when Node 1 suffers a catastrophic power outage under each approach?
3. What was the **Ghostcat (CVE-2020-1938)** vulnerability in the Tomcat AJP connector? If an organization insists on using AJP over `mod_proxy_http`, what three configuration settings in `server.xml` are mandatory to secure it?

### Practical Linux Questions
4. You need to perform a zero-downtime deployment on Tomcat `node1` (`10.0.2.21:8080`). Write the curl command or step-by-step process using `balancer-manager` to put `node1` into Drain mode (`+D`) so active sessions can complete without receiving new users.
5. In your Apache access log format, you want to log exactly which Tomcat node handled each request and how many microseconds the transaction took. What two format strings (`%...`) do you add to `LogFormat` / `CustomLog`?

### Real-World Production Troubleshooting Scenario
6. **Scenario**: Your company deploys a release during business hours.
   - 3 minutes after deployment, customer service reports that 50% of logged-in users are randomly experiencing "Session Expired" errors while browsing.
   - You inspect `curl -i -k https://app.example.com/api/test` and observe:
     `Set-Cookie: JSESSIONID=B98A172CF841920.tomcat_node_01; Path=/; HttpOnly`
   - You inspect `/etc/apache2/sites-available/app.conf` and see:
     ```apache
     <Proxy "balancer://tomcatcluster">
         BalancerMember http://10.0.2.21:8080 route=node1
         BalancerMember http://10.0.2.22:8080 route=node2
         ProxySet stickysession=JSESSIONID
     </Proxy>
     ```
   - What is the exact root cause of the session expiration bug?
   - How do you fix it without restarting Apache or dropping traffic?

---

# Module 7: Linux Production Troubleshooting Playbook

## 7.1 The Senior Sysadmin Diagnostic Framework: The USE Method

In a high-severity production outage (P1/Sev-1), junior engineers often panic and randomly restart services. A **Senior Production Engineer** follows a methodical diagnostic funnel known as the **USE Method** (Utilization, Saturation, Errors):

```
                       [ INCOMING ALERT: SITE DOWN / LATENCY SPIKE ]
                                             |
                                             v
                      +----------------------------------------------+
                      | LEVEL 1: PHYSICAL & OS SYSTEM HEALTH         |
                      | - CPU: Utilization (%usr, %sys) & iowait (%wa)|
                      | - RAM: Free, Available, Buffers/Cached, Swap |
                      | - DISK: Root & Log filesystem % full, Inodes |
                      | - NETWORK: Sockets, Dropped packets, MTU     |
                      +----------------------+-----------------------+
                                             |
                                             v
                      +----------------------------------------------+
                      | LEVEL 2: EDGE & NETWORK LAYER                |
                      | - DNS Resolution (dig, nslookup)             |
                      | - Firewall & Security Groups (ufw, iptables) |
                      | - TCP Port Openings (nc -zv, ss -tulpn)      |
                      +----------------------+-----------------------+
                                             |
                                             v
                      +----------------------------------------------+
                      | LEVEL 3: WEB TIER (APACHE HTTPD)             |
                      | - Process State: systemctl status apache2    |
                      | - Syntax: apache2ctl -t                      |
                      | - MPM Thread Capacity: mod_status Scoreboard |
                      | - Logs: error.log (AH00484, AH01102)         |
                      +----------------------+-----------------------+
                                             |
                                             v
                      +----------------------------------------------+
                      | LEVEL 4: APP TIER (TOMCAT & JVM)             |
                      | - Process State: systemctl status tomcat     |
                      | - Garbage Collection: jstat -gcutil (Full GC)|
                      | - Thread Locks & Deadlocks: jstack           |
                      | - Connector Queue: acceptCount, maxThreads   |
                      | - Logs: catalina.out, localhost.log          |
                      +----------------------+-----------------------+
                                             |
                                             v
                      +----------------------------------------------+
                      | LEVEL 5: PERSISTENCE & DATABASE TIER         |
                      | - Connection Pool Sockets (HikariCP / JNDI)  |
                      | - Slow SQL Queries & Row Locks               |
                      +----------------------------------------------+
```

---

## 7.2 The Essential Linux Diagnostic Toolkit

Before troubleshooting specific failure modes, master these core commands and understand what each does:

| Command | Primary Use Case in Production | Essential Flags & Syntax |
| :--- | :--- | :--- |
| `systemctl` | Manage service states and verify failures | `systemctl status <svc> -l --no-pager` |
| `journalctl` | Query systemd centralized binary journal logs | `journalctl -u apache2 -xe --since "1 hour ago"` |
| `ss` | Modern socket statistics (replaces legacy `netstat`) | `ss -tulpn` (listening ports with PIDs)<br>`ss -tan state established` |
| `lsof` | List open files, sockets, and unlinked file descriptors | `lsof -i :8080` (who owns port)<br>`lsof -p <PID>` (open files of process) |
| `ps` | Process hierarchy and resource allocation | `ps -eo pid,user,comm,pcpu,pmem,rss --sort -rss \| head -n 10` |
| `top` / `htop` | Real-time CPU, thread, and process inspection | `top -H -p <PID>` (inspect threads inside JVM process) |
| `free` | Physical and swap memory inspection | `free -h` (watch `available` memory, NOT just `free`) |
| `df` / `du` | Disk storage and folder consumption | `df -h` (space)<br>`df -i` (inodes)<br>`du -sh /var/log/* \| sort -rh \| head -n 5` |
| `curl` | Raw HTTP/HTTPS probing and response timer dissection | `curl -Iv https://app.com -w "\nTTFB: %{time_starttransfer}s\n"` |
| `nc` (netcat) | Layer 4 TCP port connectivity testing | `nc -zv -w 3 10.0.2.20 8080` |
| `strace` | Trace system calls and signals of a hung process | `sudo strace -p <PID> -f -e trace=network,file` |

---

## 7.3 The 20 Production Failure Scenarios & Troubleshooting Playbook

### Scenario 1: Apache Service Fails to Start
- **Symptoms**: `systemctl start apache2` fails with `Job for apache2.service failed because the control process exited with error code`.
- **Root Causes**: Configuration syntax error, port 80/443 already bound by another daemon (e.g., Nginx), or missing SSL certificate file.
- **Investigation Commands**:
  ```bash
  sudo apache2ctl configtest
  sudo journalctl -u apache2 -n 25 --no-pager
  sudo ss -tulpn | grep -E '(:80|:443)'
  ```
- **Resolution**: Fix syntax typo in `.conf` file, release conflicting port (`kill -9 <PID>`), or correct SSL paths.
- **Prevention**: Always run `apache2ctl configtest` in CI/CD before reloading.

---

### Scenario 2: Tomcat Service Fails to Start
- **Symptoms**: `systemctl start tomcat` reports failure or exits after a few seconds.
- **Root Causes**: Invalid XML syntax in `server.xml`, JVM memory allocation exceeds available OS RAM (`-Xmx`), or `JAVA_HOME` path is wrong.
- **Investigation Commands**:
  ```bash
  tail -n 50 /opt/tomcat/logs/catalina.out
  tail -n 50 /opt/tomcat/logs/catalina.$(date +%Y-%m-%d).log
  grep -i "error" /opt/tomcat/logs/catalina.out
  ```
- **Log Interpretation**:
  `LifecycleException: Protocol handler initialization failed: BindException: Address already in use: 8080` $\to$ Zombie Tomcat process holding socket.
- **Resolution**: Kill orphaned Java PID (`kill -9 $(pgrep -f tomcat)`), validate XML with `xmllint --noout /opt/tomcat/conf/server.xml`.

---

### Scenario 3: Website Completely Inaccessible Externally
- **Symptoms**: Browser hangs with `ERR_CONNECTION_TIMED_OUT`.
- **Root Causes**: AWS Security Group or local firewall (`ufw`, `iptables`) blocking port 80/443, or Route 53 DNS record pointing to wrong IP.
- **Investigation Commands**:
  ```bash
  dig +trace app.example.com
  sudo ufw status verbose
  nc -zv -w 3 <PUBLIC_IP> 443
  sudo iptables -L -n -v
  ```
- **Resolution**: Open port 443 in firewall (`sudo ufw allow 443/tcp`) or adjust AWS VPC Security Group inbound rules.

---

### Scenario 4: Apache Returns `HTTP 502 Bad Gateway`
- **Symptoms**: Users see "502 Bad Gateway" generated by Apache.
- **Root Causes**: Tomcat service crashed, Tomcat port 8080 is blocked by internal firewall, or Tomcat's JVM is stuck in a Stop-The-World GC pause.
- **Investigation Commands**:
  ```bash
  # Check Apache error log
  grep "proxy_http:error" /var/log/apache2/error.log | tail -n 10
  # Check Tomcat reachability from Apache host
  curl -I http://10.0.2.20:8080/health
  # Check if Tomcat JVM is alive
  ps -ef | grep tomcat
  ```
- **Log Signature**:
  `(111)Connection refused: AH01114: HTTP: failed to make connection to backend: 10.0.2.20:8080` $\to$ Tomcat is stopped or port is blocked.
  `AH01102: error reading status line from remote server` $\to$ Tomcat connected, but timed out before returning headers.
- **Resolution**: Start Tomcat service, adjust Tomcat connector timeouts, or remediate JVM heap exhaustion.

---

### Scenario 5: Apache Returns `HTTP 503 Service Unavailable`
- **Symptoms**: Users see "503 Service Unavailable".
- **Root Causes**: All `mod_proxy_balancer` members are marked in error state, or Apache reached `MaxRequestWorkers` limit.
- **Investigation Commands**:
  ```bash
  # Check balancer status
  curl -s http://127.0.0.1/server-status?auto
  # Check error log for balancer errors
  grep "proxy_balancer:emerg" /var/log/apache2/error.log
  ```
- **Log Signature**:
  `AH01177: balancer://tomcatcluster: All workers are in error state for route balancer://tomcatcluster`
- **Resolution**: Verify backend nodes are healthy. Lower the `retry=N` parameter on `BalancerMember` to 5 seconds so Apache rapidly recovers when nodes come back up.

---

### Scenario 6: Website Works Locally (`curl localhost`) But Not Externally
- **Symptoms**: `curl -I http://127.0.0.1` on the server returns `200 OK`, but external browsers cannot connect.
- **Root Causes**: Apache is bound strictly to `127.0.0.1:80` instead of `0.0.0.0:80` (all interfaces), or cloud network ACL/Security Group is dropping external packets.
- **Investigation Commands**:
  ```bash
  # Check socket binding
  sudo ss -tulpn | grep :80
  ```
- **Log Interpretation**:
  If output shows `127.0.0.1:80`, Apache refuses external connections. If `0.0.0.0:80` or `*:80`, Apache is listening publicly, meaning the firewall/routing layer is dropping packets.
- **Resolution**: Update `/etc/apache2/ports.conf` to `Listen 80` (binds to all interfaces).

---

### Scenario 7: Tomcat Running But Application Returns `404 Not Found`
- **Symptoms**: Tomcat default page loads or Tomcat returns 404 with standard header: `The origin server did not find a current representation for the target resource`.
- **Root Causes**: Context path mismatch (e.g., requesting `/api` when WAR is named `ROOT.war` or `app#v1.war`), or WAR file failed to explode due to permission issues.
- **Investigation Commands**:
  ```bash
  ls -la /opt/tomcat/webapps/
  cat /opt/tomcat/logs/localhost.$(date +%Y-%m-%d).log
  ```
- **Resolution**: Fix context path routing, ensure `webapps/` is owned by `tomcat:tomcat`.

---

### Scenario 8: HTTP Works But HTTPS Handshake Fails
- **Symptoms**: `http://app.com` loads, but `https://app.com` throws `SSL_ERROR_RX_RECORD_TOO_LONG` or `Connection refused`.
- **Root Causes**: Apache is not listening on port 443, SSL module is disabled (`mod_ssl`), or VirtualHost on port 443 has `SSLEngine` turned off (sending plain HTTP over port 443!).
- **Investigation Commands**:
  ```bash
  apache2ctl -M | grep ssl
  sudo ss -tulpn | grep :443
  openssl s_client -connect app.example.com:443 -servername app.example.com
  ```
- **Resolution**: Run `sudo a2enmod ssl`, verify `SSLEngine on` is inside `<VirtualHost *:443>`, reload Apache.

---

### Scenario 9: High CPU Usage (Spiking to 100%)
- **Symptoms**: Server becomes sluggish; alert triggers for sustained 95%+ CPU load.
- **Root Causes**:
  - In Tomcat: Continuous Full GC loop (Stop-The-World thrashing) or infinite loop in Java code.
  - In Apache: Prefork MPM process explosion under DDoS.
- **Investigation Commands**:
  ```bash
  top -c
  # Find which thread inside Tomcat is consuming CPU
  top -H -p $(pgrep -f tomcat)
  # Convert top thread PID (decimal) to hex for jstack correlation
  printf "%x\n" <THREAD_PID>
  # Search thread dump for the hex PID (nid)
  jstack <TOMCAT_PID> | grep -A 20 "nid=0x<HEX_PID>"
  ```
- **Resolution**: Optimize or terminate the offending Java thread; scale JVM heap if caused by GC thrashing.

---

### Scenario 10: High Memory Usage & Swap Thrashing
- **Symptoms**: `free -m` shows `available` RAM near zero; `si` (swap in) and `so` (swap out) columns in `vmstat 1` show high continuous disk paging; CPU I/O wait (`%wa`) spikes.
- **Root Causes**: JVM heap + OS buffer requirements exceed physical RAM, or Apache Prefork MPM spawning too many 40MB processes.
- **Investigation Commands**:
  ```bash
  free -h
  vmstat 1 5
  ps -eo pid,user,comm,rss --sort -rss | head -n 10
  ```
- **Resolution**: Reduce Tomcat `-Xmx` in `setenv.sh` or migrate Apache to Event MPM with strictly sized `ServerLimit`.

---

### Scenario 11: Tomcat `OutOfMemoryError: Java heap space`
- **Symptoms**: Tomcat stops responding; `catalina.out` displays `java.lang.OutOfMemoryError: Java heap space`.
- **Root Causes**: Application memory leak (e.g., unbounded caches, session collection leaks), or JVM heap sized too small for concurrent user load.
- **Investigation Commands**:
  ```bash
  # Check if heap dump was generated
  ls -lh /opt/tomcat/logs/*.hprof
  # Analyze class occupancy
  jcmd <PID> GC.class_histogram | head -n 20
  ```
- **Resolution**: Increase `-Xmx` in `setenv.sh` (if hardware permits); download `.hprof` file and inspect in Eclipse MAT.

---

### Scenario 12: File Descriptor Exhaustion (`Too many open files`)
- **Symptoms**: Apache error log shows `(24)Too many open files: AH00056: connect to listener on [::]:443 failed`; Tomcat throws `java.io.IOException: Too many open files`.
- **Root Causes**: Linux user `ulimit -n` is left at default `1024`, exhausting available file handles under high socket concurrency.
- **Investigation Commands**:
  ```bash
  # Check limits of running process
  cat /proc/<PID>/limits | grep "Max open files"
  # Count active open file descriptors of process
  ls /proc/<PID>/fd | wc -l
  ```
- **Resolution**:
  In `/etc/systemd/system/apache2.service` and `tomcat.service`, set:
  ```ini
  LimitNOFILE=65536
  ```
  Run `sudo systemctl daemon-reload && sudo systemctl restart <service>`.

---

### Scenario 13: Port Conflicts (`Address already in use`)
- **Symptoms**: Service fails to bind to port 80, 443, or 8080.
- **Root Causes**: An orphaned instance from an unclean shutdown is still running, or another program (Nginx, Docker) is bound to the port.
- **Investigation Commands**:
  ```bash
  sudo ss -tulpn | grep -E '(:80|:443|:8080)'
  sudo lsof -i :8080
  ```
- **Resolution**: Terminate the rogue process: `sudo kill -9 <PID>`.

---

### Scenario 14: Application Deployment Failure (Context Startup Error)
- **Symptoms**: `app.war` deployed to `webapps/`, but Tomcat returns 404; WAR does not unpack or unpacks with errors.
- **Root Causes**: Missing database environment variables, incorrect Spring application properties, missing JDBC driver, or corrupt WAR file.
- **Investigation Commands**:
  ```bash
  tail -n 100 /opt/tomcat/logs/catalina.$(date +%Y-%m-%d).log
  tail -n 100 /opt/tomcat/logs/localhost.$(date +%Y-%m-%d).log
  ```
- **Log Signature**:
  `org.apache.catalina.core.StandardContext.startInternal One or more listeners failed to start`
- **Resolution**: Read the nested root cause exception in `localhost.log` and provide the missing configuration/dependency.

---

### Scenario 15: Linux Permission Denied Errors (`403 Forbidden`)
- **Symptoms**: Apache returns `403 Forbidden` for static files.
- **Root Causes**: `www-data` user lacks read permissions on files or lacks execute (`+x`) permissions on parent directories.
- **Investigation Commands**:
  ```bash
  namei -l /var/www/app/static/index.html
  ```
  *(`namei -l` prints the exact permissions and ownership of every parent folder down to the file in one single command!)*
- **Resolution**:
  ```bash
  sudo chown -R www-data:www-data /var/www/app
  sudo find /var/www/app -type d -exec chmod 755 {} +
  sudo find /var/www/app -type f -exec chmod 644 {} +
  ```

---

### Scenario 16: Slow API Response / High Latency Triage
- **Symptoms**: Users complain checkout takes 15 seconds.
- **Root Causes**: Database lock contention, unindexed SQL queries, slow external API call, or thread pool queueing.
- **Investigation Commands**:
  ```bash
  # Check Apache microsecond request duration (%D)
  tail -f /var/log/apache2/app_access.log | awk '{print $NF/1000000 " sec - " $7}'
  
  # Check Tomcat request duration
  tail -f /opt/tomcat/logs/localhost_access_log.*.txt
  ```
- **Analysis**:
  - If Apache `%D` is 15s and Tomcat `%D` is 15s $\to$ Delay is inside Tomcat or the Database.
  - If Apache `%D` is 15s and Tomcat `%D` is 50ms $\to$ Delay is between Client and Apache (slow client upload/download network).

---

### Scenario 17: Apache & Tomcat Communication Failure
- **Symptoms**: Intermittent `502` or `504` errors between tiers.
- **Root Causes**: Connection timeout mismatch, TCP keepalive packets dropped by AWS NAT Gateway or stateful firewall, or AJP secret mismatch.
- **Investigation Commands**:
  ```bash
  # Sniff TCP flags between Apache and Tomcat
  sudo tcpdump -i any -nn 'tcp port 8080 and host 10.0.2.20'
  ```
- **Resolution**: Enable `keepalive=On ping=2 retry=5` in Apache `ProxyPass`. Align Apache `timeout` with Tomcat `connectionTimeout`.

---

### Scenario 18: SSL/TLS Certificate Expiration
- **Symptoms**: Browsers display scary red warning: `NET::ERR_CERT_DATE_INVALID` or `SEC_ERROR_EXPIRED_CERTIFICATE`.
- **Root Causes**: Automated renewal cron job (Certbot) failed, or renewed cert was not reloaded into Apache memory.
- **Investigation Commands**:
  ```bash
  # Check certificate on disk
  openssl x509 -in /etc/ssl/certs/app.crt -noout -enddate
  # Check certificate served over network
  echo | openssl s_client -connect app.example.com:443 -servername app.example.com 2>/dev/null | openssl x509 -noout -dates
  ```
- **Resolution**: Renew certificate (`certbot renew`) and execute `systemctl reload apache2` to load the new cert into RAM.

---

### Scenario 19: Disk Space Exhaustion Due to Unrotated Logs
- **Symptoms**: Services crash; system outputs `No space left on device`; bash tab completion fails.
- **Root Causes**: `catalina.out` or Apache access logs grew to 50 GB because log rotation was not configured.
- **Investigation Commands**:
  ```bash
  df -h
  df -i                       # Check for inode exhaustion!
  du -sh /var/log/* | sort -rh | head -n 5
  du -sh /opt/tomcat/logs/* | sort -rh | head -n 5
  ```
- **Emergency Remediation**:
  Never delete an active log file with `rm file.log` while the process is writing to it! (The disk space will NOT be freed until the process is restarted because the file descriptor remains open in kernel memory).
  **Truncate instead**:
  ```bash
  sudo truncate -s 0 /opt/tomcat/logs/catalina.out
  ```
- **Permanent Fix**: Set up `/etc/logrotate.d/tomcat` with `copytruncate` and compression.

---

### Scenario 20: Unexpected Process Kills by Linux OOM Killer
- **Symptoms**: Tomcat or Apache suddenly dies without any shutdown entry in `catalina.out` or `error.log`.
- **Root Causes**: The host ran completely out of physical memory and swap. The Linux kernel invoked the **Out-Of-Memory (OOM) Killer** to protect the operating system from a kernel panic, sending `SIGKILL` (`kill -9`) to the process consuming the most RAM.
- **Investigation Commands**:
  ```bash
  # Check kernel ring buffer for OOM killer invocations
  dmesg -T | grep -i -E 'oom[-_]killer|killed process'
  # Check syslog
  grep -i "killed process" /var/log/syslog
  ```
- **Log Signature**:
  `Out of memory: Killed process 14210 (java) total-vm:8452100kB, anon-rss:4120500kB, file-rss:0kB`
- **Resolution**: Reduce JVM heap size, adjust `vm.swappiness`, or upgrade physical RAM on the server.

---

## 7.4 Module 7 Assessment & Interview Gate

Answer the following questions to complete Module 7:

### Conceptual Questions
1. If a developer accidentally executes `rm /opt/tomcat/logs/catalina.out` on a running production server to free up disk space, why does `df -h` still report 100% disk usage? What Linux command proves the file descriptor is still holding the disk blocks, and how do you recover the space without rebooting?
2. Explain the precise difference between the Linux load average values reported by `top` (`1-min`, `5-min`, `15-min`) when CPU is at 100% due to computing threads vs. when load average is high but CPU idle is at 80% due to disk I/O wait (`%wa`).
3. What is the Linux kernel OOM Killer algorithm? What kernel score (`oom_score`) determines which process gets terminated, and how can a sysadmin protect critical daemons like `sshd` from being killed?

### Practical Linux Questions
4. An Apache reverse proxy is reporting intermittent 502 errors. You suspect network packet drops between Apache and Tomcat. Write the exact `netcat` or `ss` command to test whether Tomcat's port 8080 is reachable from the Apache host with a 2-second timeout.
5. A Java process (`PID: 8421`) is consuming 98% CPU. Write the exact terminal command pipeline to identify the specific thread ID consuming the CPU inside that JVM and convert that thread ID to hexadecimal so you can locate it in a `jstack` thread dump.

### Real-World Production Troubleshooting Scenario
6. **Scenario**: At 02:00 AM, you receive a P1 alert: `app.example.com is DOWN`.
   - You SSH into the Apache web tier: `systemctl status apache2` is **active (running)**.
   - Running `curl -I https://app.example.com` returns `HTTP 502 Bad Gateway` after a 30-second delay.
   - You SSH into the backend Tomcat server (`10.0.2.20`): `systemctl status tomcat` shows the service is **active (running)**.
   - However, when you run `curl -I http://127.0.0.1:8080/health` locally on the Tomcat server, the command hangs indefinitely and never returns.
   - Running `top` on the Tomcat host reveals:
     - CPU: 2% user, 1% system, 0% wa, 97% idle.
     - RAM: 12 GB used of 16 GB, 0% swap used.
   
   **Walk me through your exact diagnostic steps.** What commands do you run right now on the Tomcat server to determine *why* Tomcat is alive but completely unresponsive, what is the most likely root cause given that CPU and RAM are completely idle, and how do you resolve it?

---

# Module 8: Security Hardening, Production Monitoring & Hands-on Projects

## 8.1 Enterprise Security Hardening Checklist

Securing an enterprise web and application tier requires closing vectors across the OS, network, web server, and JVM layers.

```
+-------------------------------------------------------------------------------------------------+
|                                 ENTERPRISE DEFENSE-IN-DEPTH MATRIX                              |
|                                                                                                 |
|  [ LAYER 1: NETWORK & OS ]                                                                      |
|  - Run all services as unprivileged system accounts (www-data, tomcat)                          |
|  - Close all ports externally except 80 and 443 (ufw, AWS Security Groups)                      |
|  - Lock configuration file permissions: chmod 600 context.xml server.xml                        |
|                                                                                                 |
|  [ LAYER 2: APACHE HTTPD ]                                                                      |
|  - Hide banner tokens: ServerTokens Prod, ServerSignature Off                                   |
|  - Disable directory indexes globally: Options -Indexes                                         |
|  - Enforce modern TLS: SSLProtocol -all +TLSv1.2 +TLSv1.3                                       |
|  - Inject Security Headers: HSTS, X-Frame-Options: DENY, X-Content-Type-Options: nosniff        |
|                                                                                                 |
|  [ LAYER 3: APACHE TOMCAT ]                                                                     |
|  - Suppress stack traces: ErrorReportValve showReport="false" showServerInfo="false"            |
|  - Mask Server header: Connector server="Apache"                                                |
|  - Disable/firewall shutdown port: <Server port="-1">                                           |
|  - Restrict Tomcat Manager by CIDR: RemoteAddrValve allow="10\.0\.0\.\d+"                       |
|  - Restrict AJP: secretRequired="true" address="127.0.0.1" (or disable AJP entirely)            |
+-------------------------------------------------------------------------------------------------+
```

### 1. Hide Server Version Banners (Prevent Banner Grabbing)
By default, web servers broadcast their exact software version and OS distribution in HTTP response headers (e.g., `Server: Apache/2.4.52 (Ubuntu) mod_ssl/2.4.52 OpenSSL/3.0.2`), allowing automated vulnerability scanners to target known CVEs.

#### Apache HTTPD (`/etc/apache2/conf-available/security.conf`):
```apache
ServerTokens Prod
ServerSignature Off
TraceEnable Off
```
*`TraceEnable Off` prevents Cross-Site Tracing (XST) attacks by disabling HTTP TRACE requests.*

#### Apache Tomcat (`conf/server.xml`):
```xml
<!-- Mask the Server response header -->
<Connector port="8080" protocol="HTTP/1.1" server="WebServer" ... />

<!-- Suppress JVM Stack Traces on HTTP 500 errors -->
<Valve className="org.apache.catalina.valves.ErrorReportValve"
       showReport="false"
       showServerInfo="false" />
```

### 2. Lock Down File Permissions & Directory Ownership
Never run Tomcat or Apache as `root`. Enforce strict least-privilege permissions:

```bash
# Apache permissions
sudo chown -R root:root /etc/apache2
sudo find /etc/apache2 -type d -exec chmod 755 {} +
sudo find /etc/apache2 -type f -exec chmod 644 {} +
sudo chmod 600 /etc/ssl/private/*

# Tomcat permissions (Binaries owned by root, runtime owned by tomcat)
sudo chown -R root:root /opt/tomcat/bin /opt/tomcat/lib
sudo chown -R tomcat:tomcat /opt/tomcat/conf /opt/tomcat/logs /opt/tomcat/temp /opt/tomcat/webapps /opt/tomcat/work
sudo chmod 750 /opt/tomcat/bin/*.sh
sudo chmod 600 /opt/tomcat/conf/server.xml /opt/tomcat/conf/context.xml /opt/tomcat/conf/tomcat-users.xml
```

### 3. Inject Production HTTP Security Headers
In Apache VirtualHost (`/etc/apache2/sites-available/app.conf`):
```apache
# Requires: sudo a2enmod headers
Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
Header always set X-Frame-Options "DENY"
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Permissions-Policy "geolocation=(), camera=(), microphone=()"
Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;"
```

---

## 8.2 Enterprise Production Monitoring & Metrics (Prometheus & Grafana)

In enterprise operations, you cannot wait for users to report an outage. You must monitor real-time telemetry using **Prometheus** (metrics scraping and time-series storage) and **Grafana** (visualization and alerting).

```
+-----------------------------------------------------------------------------------------+
|                                    TARGET NODES                                         |
|                                                                                         |
|  [ Apache Web Server ]  ---> mod_status ---> [ apache_exporter (Port 9117) ]           |
|                                                                                         |
|  [ Tomcat App Server ]  ---> JMX MBeans ---> [ jmx_prometheus_javaagent (Port 8081) ]   |
|                                                                                         |
|  [ Linux Host OS ]      ---> /proc, /sys ---> [ node_exporter (Port 9100) ]             |
+--------------------------------------------+--------------------------------------------+
                                             |
                                             | Scrapes Metrics HTTP GET every 15s
                                             v
+-----------------------------------------------------------------------------------------+
|                              PROMETHEUS MONITORING SERVER                               |
|  - Scrapes metrics from Exporters                                                       |
|  - Evaluates Alerting Rules                                                             |
|  - Sends PagerDuty / Slack alerts via Alertmanager                                      |
+--------------------------------------------+--------------------------------------------+
                                             |
                                             v
+-----------------------------------------------------------------------------------------+
|                                    GRAFANA DASHBOARDS                                   |
|  - JVM Heap Occupancy & Full GC Count                                                   |
|  - Tomcat Busy Threads vs Max Threads                                                   |
|  - Apache Worker Scoreboard & Req/sec                                                   |
|  - HTTP 5xx Error Rate Spikes                                                           |
+-----------------------------------------------------------------------------------------+
```

### 1. Apache HTTPD Monitoring (`apache_exporter`)
1. Enable `mod_status` in `/etc/apache2/mods-available/status.conf`:
   ```apache
   <Location /server-status>
       SetHandler server-status
       Require ip 127.0.0.1 10.0.0.0/16
   </Location>
   ExtendedStatus On
   ```
2. Run `apache_exporter`:
   ```bash
   ./apache_exporter --scrape_uri="http://127.0.0.1/server-status?auto"
   ```
*Metrics exposed:*
- `apache_workers{state="busy"}`: Number of worker threads currently processing requests.
- `apache_accesses_total`: Total HTTP requests served.
- `apache_scoreboard`: Real-time breakdown of thread states (`_`, `W`, `K`, `R`).

### 2. Tomcat & JVM Monitoring (`jmx_exporter`)
Tomcat exposes its internal health via **Java Management Extensions (JMX)** MBeans.
1. Download Prometheus JMX Java Agent (`jmx_prometheus_javaagent.jar`).
2. Create `/opt/tomcat/conf/jmx_exporter_config.yaml`:
   ```yaml
   startDelaySeconds: 10
   rules:
     - pattern: 'Tomcat<type=ThreadPool, name="(\w+-\w+-\d+)"><>(currentThreadsBusy|currentThreadCount|maxThreads)'
       name: tomcat_threadpool_$2
       labels:
         port: "$1"
     - pattern: 'Tomcat<type=GlobalRequestProcessor, name="(\w+-\w+-\d+)"><>(requestCount|errorCount|processingTime)'
       name: tomcat_requests_$2_total
       labels:
         port: "$1"
   ```
3. Attach agent in `bin/setenv.sh`:
   ```bash
   export CATALINA_OPTS="$CATALINA_OPTS -javaagent:/opt/tomcat/bin/jmx_prometheus_javaagent.jar=8081:/opt/tomcat/conf/jmx_exporter_config.yaml"
   ```
4. Restart Tomcat. Prometheus can now scrape live JVM metrics from `http://10.0.2.20:8081/metrics`.

### Critical Alerting Rules for Production (Prometheus Alertmanager):
- **Tomcat High Thread Utilization**: `tomcat_threadpool_currentThreadsBusy / tomcat_threadpool_maxThreads > 0.85` for 2 minutes $\to$ Triggers Thread Exhaustion Warning.
- **JVM Heap Memory Critical**: `jvm_memory_bytes_used{area="heap"} / jvm_memory_bytes_max{area="heap"} > 0.90` for 3 minutes $\to$ Imminent OutOfMemory Alert.
- **Apache High 5xx Error Spike**: `rate(apache_accesses_total{status=~"5.."}[2m]) > 5` $\to$ Backend Outage P1 Incident.

---

## 8.3 Hands-On Production Projects: Step-by-Step Implementation

### Project 1: Install Apache HTTPD & Host a Static Website
1. Install Apache on Ubuntu:
   ```bash
   sudo apt update && sudo apt install -y apache2
   ```
2. Create static website root:
   ```bash
   sudo mkdir -p /var/www/mycompany/public
   echo "<h1>Enterprise Production Web Portal</h1>" | sudo tee /var/www/mycompany/public/index.html
   sudo chown -R www-data:www-data /var/www/mycompany
   ```
3. Configure VirtualHost `/etc/apache2/sites-available/mycompany.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName mycompany.example.com
       DocumentRoot /var/www/mycompany/public
       <Directory /var/www/mycompany/public>
           Require all granted
           Options -Indexes +FollowSymLinks
           AllowOverride None
       </Directory>
       ErrorLog ${APACHE_LOG_DIR}/mycompany_error.log
       CustomLog ${APACHE_LOG_DIR}/mycompany_access.log combined
   </VirtualHost>
   ```
4. Enable and test:
   ```bash
   sudo a2ensite mycompany.conf
   sudo a2dissite 000-default.conf
   sudo apache2ctl configtest
   sudo systemctl reload apache2
   curl -H "Host: mycompany.example.com" http://127.0.0.1/
   ```

---

### Project 2: Install Apache Tomcat & Deploy a Java WAR Application
1. Install OpenJDK 17 and create dedicated service account:
   ```bash
   sudo apt install -y openjdk-17-jdk
   sudo groupadd -r tomcat
   sudo useradd -r -g tomcat -d /opt/tomcat -s /sbin/nologin tomcat
   ```
2. Download and extract Tomcat 10:
   ```bash
   sudo mkdir -p /opt/tomcat
   wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.20/bin/apache-tomcat-10.1.20.tar.gz -O /tmp/tomcat.tar.gz
   sudo tar -xf /tmp/tomcat.tar.gz -C /opt/tomcat --strip-components=1
   ```
3. Enforce secure permissions:
   ```bash
   sudo chown -R root:root /opt/tomcat/bin /opt/tomcat/lib
   sudo chown -R tomcat:tomcat /opt/tomcat/conf /opt/tomcat/logs /opt/tomcat/temp /opt/tomcat/webapps /opt/tomcat/work
   sudo chmod 750 /opt/tomcat/bin/*.sh
   ```
4. Deploy sample application:
   ```bash
   sudo -u tomcat cp /tmp/sample.war /opt/tomcat/webapps/api.war
   ```
5. Start Tomcat and verify:
   ```bash
   sudo -u tomcat /opt/tomcat/bin/startup.sh
   sleep 5
   curl -I http://127.0.0.1:8080/api/
   ```

---

### Project 3: Configure Apache HTTPD as a Reverse Proxy for Tomcat
1. Enable proxy modules in Apache:
   ```bash
   sudo a2enmod proxy proxy_http headers
   ```
2. Configure proxy directives in `/etc/apache2/sites-available/mycompany.conf`:
   ```apache
   # Forward client headers
   RequestHeader set X-Forwarded-Proto "http"
   ProxyPreserveHost On

   # Proxy /api to Tomcat backend
   ProxyPass /api http://127.0.0.1:8080/api retry=2 timeout=30 keepalive=On
   ProxyPassReverse /api http://127.0.0.1:8080/api
   ```
3. Configure `RemoteIpValve` in Tomcat `/opt/tomcat/conf/server.xml` inside `<Host>`:
   ```xml
   <Valve className="org.apache.catalina.valves.RemoteIpValve"
          internalProxies="127\.0\.0\.1"
          remoteIpHeader="x-forwarded-for"
          protocolHeader="x-forwarded-proto" />
   ```
4. Reload Apache, restart Tomcat, and verify real IP forwarding:
   ```bash
   sudo apache2ctl configtest && sudo systemctl reload apache2
   curl -I http://127.0.0.1/api/
   ```

---

### Project 4: Configure HTTPS with SSL Termination at Apache HTTPD
1. Generate a production self-signed certificate (or use Let's Encrypt / Certbot):
   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout /etc/ssl/private/mycompany.key \
     -out /etc/ssl/certs/mycompany.crt \
     -subj "/CN=mycompany.example.com/O=Enterprise IT"
   sudo chmod 600 /etc/ssl/private/mycompany.key
   ```
2. Enable SSL module: `sudo a2enmod ssl`
3. Configure `<VirtualHost *:443>` in `/etc/apache2/sites-available/mycompany-ssl.conf`:
   ```apache
   <VirtualHost *:443>
       ServerName mycompany.example.com
       DocumentRoot /var/www/mycompany/public

       SSLEngine on
       SSLCertificateFile /etc/ssl/certs/mycompany.crt
       SSLCertificateKeyFile /etc/ssl/private/mycompany.key
       SSLProtocol -all +TLSv1.2 +TLSv1.3

       Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

       RequestHeader set X-Forwarded-Proto "https"
       RequestHeader set X-Forwarded-Port "443"
       ProxyPreserveHost On

       ProxyPass /api http://127.0.0.1:8080/api retry=2 timeout=30
       ProxyPassReverse /api http://127.0.0.1:8080/api
   </VirtualHost>
   ```
4. Enable site and verify TLS:
   ```bash
   sudo a2ensite mycompany-ssl.conf
   sudo apache2ctl configtest && sudo systemctl reload apache2
   curl -kv https://127.0.0.1/api/ --resolve mycompany.example.com:443:127.0.0.1
   ```

---

### Project 5: Configure Apache HTTPD with Two Tomcat Instances & Load Balancing
1. Create two independent `CATALINA_BASE` instances:
   - Instance 1: `/srv/tomcat/node1` (HTTP port `8081`, shutdown port `8006`, `jvmRoute="node1"`)
   - Instance 2: `/srv/tomcat/node2` (HTTP port `8082`, shutdown port `8007`, `jvmRoute="node2"`)
2. Enable load balancer modules:
   ```bash
   sudo a2enmod proxy_balancer lbmethod_bybusyness
   ```
3. Configure `balancer://tomcatcluster` in Apache:
   ```apache
   <Proxy "balancer://tomcatcluster">
       BalancerMember http://127.0.0.1:8081 route=node1 loadfactor=1 retry=5
       BalancerMember http://127.0.0.1:8082 route=node2 loadfactor=1 retry=5
       ProxySet lbmethod=bybusyness stickysession=JSESSIONID
   </Proxy>

   ProxyPass /api balancer://tomcatcluster/api
   ProxyPassReverse /api balancer://tomcatcluster/api
   ```
4. Start both instances and verify traffic distribution across both nodes.

---

### Project 6: Configure Tomcat as a Systemd Service with Production JVM Settings
1. Create `/opt/tomcat/bin/setenv.sh`:
   ```bash
   export CATALINA_OPTS="$CATALINA_OPTS -Xms2048m -Xmx2048m -XX:MaxMetaspaceSize=512m -XX:+UseG1GC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/tomcat/logs/ -XX:+ExitOnOutOfMemoryError"
   ```
2. Create `/etc/systemd/system/tomcat.service` with security limits (`LimitNOFILE=65536`, `PrivateTmp=yes`).
3. Reload daemon and start:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now tomcat
   sudo systemctl status tomcat
   ```

---

### Project 7: Configure Monitoring & Log Rotation
1. Create `/etc/logrotate.d/tomcat`:
   ```text
   /opt/tomcat/logs/catalina.out {
       daily
       rotate 14
       compress
       missingok
       notifempty
       copytruncate
   }
   ```
2. Test rotation dry run:
   ```bash
   sudo logrotate -d /etc/logrotate.d/tomcat
   ```

---

### Project 8: Troubleshoot Deliberately Introduced Production Failures
1. **Lab Exercise 1**: Simulate Tomcat crash (`kill -9 <PID>`) and observe Apache 502 error logs and failover behavior.
2. **Lab Exercise 2**: Introduce a syntax error in `server.xml` and trace the startup failure using `journalctl` and `catalina.out`.
3. **Lab Exercise 3**: Simulate a file permission error on `/var/www` and diagnose using `namei -l`.

---

## 8.4 Module 8 Assessment & Final Course Capstone

Answer the following questions to complete your training:

### Conceptual Questions
1. Why does running Apache HTTPD with `ServerTokens Prod` and `ServerSignature Off` improve production security, and what reconnaissance tool/technique does it obstruct?
2. When configuring Prometheus to monitor Tomcat via JMX, why should the JMX exporter agent run inside the JVM process (`-javaagent`) rather than polling over remote RMI network ports?
3. In a high-availability environment with two Tomcat instances behind Apache `mod_proxy_balancer`, explain the exact steps required to perform a zero-downtime rolling upgrade when database schema migrations must also be applied.

### Practical Linux Questions
4. An auditor demands proof that your Apache server does not support insecure SSL/TLS ciphers. Write the exact `openssl` command to test whether your server accepts an obsolete TLS 1.0 handshake.
5. Write the exact `logrotate` configuration block for `/var/log/apache2/*.log` that rotates logs weekly, keeps 12 weeks of backlog, compresses old logs with gzip, delays compression by one cycle (`delaycompress`), and reloads Apache gracefully without dropping active connections.

### Final Capstone Architecture & Production Troubleshooting Scenario
6. **Scenario**: You are the Lead DevOps Architect for an e-commerce platform processing \$500,000/hour.
   - During Cyber Monday, the website suddenly becomes unresponsive.
   - Apache error logs show:
     `AH01177: balancer://tomcatcluster: All workers are in error state`
     `AH00484: server reached MaxRequestWorkers setting`
   - You log in to the Tomcat cluster (Node 1 and Node 2):
     - Both Tomcat instances have CPU at 99%.
     - `free -m` shows swap memory is 100% full.
     - Database administrators report 200 active PostgreSQL connections, all blocked in `idle in transaction` state.
   
   **Provide your comprehensive Incident Response (IR) plan:**
   1. What is the immediate triage and emergency remediation to restore service to shoppers within 5 minutes?
   2. What is the chronological root cause sequence that caused both tiers (HTTPD and Tomcat) to collapse?
   3. What three architectural changes will you implement post-incident to guarantee this failure mode can never occur again?

---
*Take your time to synthesize your knowledge and solve this Capstone. Once submitted, you will receive a comprehensive senior architectural evaluation covering the entire Apache HTTPD & Tomcat curriculum.*
