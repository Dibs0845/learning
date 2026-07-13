# AWS Networking End-to-End — A Complete, Easy-to-Understand Guide

> This guide builds up from the ground floor. Every concept uses a plain-English
> analogy first, then the technical detail. Read it top to bottom the first time;
> use it as a reference afterward.

---

## Table of Contents

1. [The Big Picture (Analogy First)](#1-the-big-picture-analogy-first)
2. [IP Addressing & CIDR — The Foundation](#2-ip-addressing--cidr--the-foundation)
3. [VPC — Your Private Data Center in the Cloud](#3-vpc--your-private-data-center-in-the-cloud)
4. [Subnets — Dividing Your VPC](#4-subnets--dividing-your-vpc)
5. [Route Tables — The GPS of Your Network](#5-route-tables--the-gps-of-your-network)
6. [Internet Gateway & NAT — Getting In and Out](#6-internet-gateway--nat--getting-in-and-out)
7. [Security Groups & NACLs — The Firewalls](#7-security-groups--nacls--the-firewalls)
8. [A Complete Request, Traced End-to-End](#8-a-complete-request-traced-end-to-end)
9. [Connecting VPCs and On-Premises](#9-connecting-vpcs-and-on-premises)
10. [DNS with Route 53](#10-dns-with-route-53)
11. [Load Balancing & Content Delivery](#11-load-balancing--content-delivery)
12. [VPC Endpoints — Private Access to AWS Services](#12-vpc-endpoints--private-access-to-aws-services)
13. [A Reference Production Architecture](#13-a-reference-production-architecture)
14. [Common Gotchas & Troubleshooting](#14-common-gotchas--troubleshooting)
15. [Glossary](#15-glossary)

---

## 1. The Big Picture (Analogy First)

Think of AWS networking like **building a private office campus**:

| Real World | AWS Equivalent |
|------------|----------------|
| A plot of land you fence off | **VPC** (Virtual Private Cloud) |
| Buildings on that land | **Subnets** |
| Room numbers / floor plans | **IP addresses / CIDR** |
| The campus road map that says "to reach the exit, go this way" | **Route Tables** |
| The main gate to the public road | **Internet Gateway** |
| A one-way delivery door (staff can order out, but strangers can't walk in) | **NAT Gateway** |
| Security guard at each building door checking a guest list | **Security Group** |
| Border checkpoint at the campus edge checking every vehicle in and out | **Network ACL** |
| The campus phone directory (name → office) | **Route 53 (DNS)** |
| The reception desk that distributes visitors to available staff | **Load Balancer** |
| A private tunnel to your other campus | **VPC Peering / Transit Gateway / VPN** |

Everything else in this guide is just detail on these pieces and how they connect.

```
                            INTERNET
                               │
                        ┌──────┴──────┐
                        │  Internet   │
                        │   Gateway   │
                        └──────┬──────┘
   ╔═══════════════════════════│═══════════════════════════╗
   ║  VPC  10.0.0.0/16         │                            ║
   ║                    ┌──────┴───────┐                    ║
   ║                    │ Public Subnet│  10.0.1.0/24       ║
   ║                    │  [ ALB ]     │                    ║
   ║                    │  [ NAT GW ]  │                    ║
   ║                    └──────┬───────┘                    ║
   ║                           │                            ║
   ║                    ┌──────┴───────┐                    ║
   ║                    │Private Subnet│  10.0.2.0/24       ║
   ║                    │  [ EC2 app ] │                    ║
   ║                    └──────┬───────┘                    ║
   ║                           │                            ║
   ║                    ┌──────┴───────┐                    ║
   ║                    │  DB Subnet   │  10.0.3.0/24       ║
   ║                    │  [ RDS ]     │  (no internet)     ║
   ║                    └──────────────┘                    ║
   ╚═══════════════════════════════════════════════════════╝
```

---

## 2. IP Addressing & CIDR — The Foundation

This is the single most common stumbling block, so we'll go slowly and build it up piece by piece. By the end you'll understand not just *what* CIDR is, but *why* it exists and *why* we split networks into subnets.

### 2.1 What is an IP address? (the house-number analogy)

Every device on a network needs a unique address so data knows where to go — exactly like every house on a street needs a unique house number so the postman knows where to deliver mail.

An IPv4 address is four numbers (each 0–255) separated by dots:

```
10 . 0 . 5 . 23
```

That's the "human-friendly" way to write it. But computers don't think in dots — they think in **bits** (0s and 1s). Each of those four numbers is really **8 bits** (called an *octet*), so a full IP address is **32 bits** total:

```
   10    .    0     .    5     .    23
00001010 . 00000000 . 00000101 . 00010111     ← 32 bits
└──8 bits┘ └──8 bits┘ └──8 bits┘ └──8 bits┘
```

Why does "8 bits" only go up to 255? Because 8 bits can represent 2⁸ = 256 different values, which is `0` through `255`. That's the only reason each number in an IP maxes out at 255.

> **Key idea to hold onto:** an IP address is just a 32-bit number. Everything about CIDR and subnets is about deciding *which bits stay fixed* and *which bits are free to change*.

### 2.2 The problem: an address alone isn't enough

Say you have a company with 500 computers. You *could* give every one a totally random address and keep a giant list of "who is where." But that's a nightmare to manage and route.

Instead, we want to say things like:

> "All computers in the **Mumbai office** have addresses that start with `10.0.5.___`"

Now the last part identifies the individual computer, and the first part identifies the *group* (the office). Routers love this: to send mail to any Mumbai computer, they just look at the first part and forward it toward Mumbai — they don't need to know every single machine.

So every IP address is really split into **two parts**:

```
   NETWORK part   +   HOST part
 (which group?)       (which machine in that group?)
```

The million-dollar question is: **where do we draw the line between the two parts?** That's exactly what CIDR answers.

### 2.3 What is CIDR, and why does it exist?

**CIDR** stands for *Classless Inter-Domain Routing*. Forget the scary name — all it does is draw that dividing line using a simple `/number` suffix.

```
10.0.0.0/16
        └── this "/16" means: the first 16 bits are the NETWORK part.
```

The number after the slash (the **prefix length**) tells you **how many bits from the left are fixed** (the network). Everything after those bits is free to change (the hosts).

Let's see it on the bits for `10.0.0.0/16`:

```
00001010 00000000 | 00000000 00000000
└──── fixed 16 ───┘ └──── free 16 ────┘
   NETWORK  =  10.0    HOST = anything from .0.0 to .255.255
```

So `10.0.0.0/16` describes **every address from `10.0.0.0` to `10.0.255.255`** — the "10.0" part never changes, the last two numbers can be anything.

**Why CIDR was invented (the short history):** the old system ("classes" A/B/C) forced you into fixed sizes — you either got 256 addresses, 65,536, or 16 million, with nothing in between. That wasted enormous numbers of addresses. CIDR (1993) let you draw the line at *any* bit position, so you can carve out a range of *exactly* the size you need. "Classless" literally means "no more rigid classes."

### 2.4 How to read any CIDR block

Two things you can always work out from a CIDR:

**(a) How many addresses does it contain?**

> **addresses = 2^(32 − prefix)**

The logic: 32 total bits, minus the fixed network bits, leaves the free host bits. Each free bit doubles the count.

| CIDR | Fixed (network) bits | Free (host) bits | Total addresses | Think of it as |
|------|------|------|------|------|
| `/8`  | 8  | 24 | 16,777,216 | A whole country |
| `/16` | 16 | 16 | 65,536 | A whole VPC |
| `/24` | 24 | 8  | 256 | One subnet / one street |
| `/28` | 28 | 4  | 16 | A tiny subnet |
| `/32` | 32 | 0  | 1 | One single machine |
| `/0`  | 0  | 32 | 4,294,967,296 | Literally everything |

**(b) What's the range?** The address given is the *start* (network address); the range runs up to the point just before the next block begins.

> **Golden rule of thumb: a bigger slash number = a SMALLER range.**
> `/32` is one address; `/0` is the entire internet. This feels backwards at first — remember: a bigger number means *more bits are locked down*, so *fewer* are left to vary.

```
  smaller number  ◄─────────────────────────►  bigger number
     /8                    /16                       /32
  HUGE range           medium range            ONE address
  16 million            65,536                      1
```

### 2.5 The two CIDR "shortcuts" you'll see everywhere

These two show up constantly in route tables and security groups — memorize them:

- **`0.0.0.0/0`** = "**any address anywhere**" = the entire internet. (Zero bits fixed → everything matches.) When a route table says `0.0.0.0/0 → Internet Gateway`, it means "send anything I don't otherwise recognize out to the internet."
- **`10.0.5.23/32`** = "**this one exact machine**." (All 32 bits fixed → only one address matches.) Used when you want to allow/route to a single specific host, like your office's public IP.

### 2.6 Why do we split a VPC into subnets? (the *why* of subnetting)

You give your whole VPC one big CIDR, say `10.0.0.0/16` (65,536 addresses). Why not just dump every server into that one big pool? Because **subnets buy you four things**:

1. **Placement across Availability Zones.** A subnet lives in exactly *one* AZ (one data-center location). To survive a data-center failure, you split your range so some servers sit in AZ-a and some in AZ-b. Each AZ needs its own subnet.

2. **Public vs. private separation.** As you'll see in §4, a subnet is "public" or "private" based on *its own route table*. Putting your database in a separate subnet with no internet route is how you keep it unreachable from outside. You can't do that if everything shares one subnet.

3. **Blast-radius / security control.** Firewalls at the subnet border (NACLs) and different routing let you contain problems. Web tier, app tier, and database tier each get their own subnet so you can apply different rules to each.

4. **Organization & routing efficiency.** Grouping related machines under a shared prefix keeps routing tables small and your network understandable.

**Subnetting = taking your VPC's big CIDR and slicing it into smaller CIDRs**, one per subnet. The subnets' ranges must fit inside the VPC's range and must **not overlap** each other.

### 2.7 Subnetting worked example (slicing 10.0.0.0/16)

You have a VPC `10.0.0.0/16`. You want to carve it into subnets. A clean, common choice is to make each subnet a `/24` (256 addresses each). Watch how you just change the **third number** to make non-overlapping slices:

```
VPC:  10.0.0.0/16   (10.0.0.0  →  10.0.255.255,  65,536 addresses)
        │
        ├── 10.0.1.0/24   → Public subnet  AZ-a   (10.0.1.0  – 10.0.1.255)
        ├── 10.0.2.0/24   → App subnet     AZ-a   (10.0.2.0  – 10.0.2.255)
        ├── 10.0.3.0/24   → DB subnet      AZ-a   (10.0.3.0  – 10.0.3.255)
        │
        ├── 10.0.11.0/24  → Public subnet  AZ-b   (10.0.11.0 – 10.0.11.255)
        ├── 10.0.12.0/24  → App subnet     AZ-b   (10.0.12.0 – 10.0.12.255)
        └── 10.0.13.0/24  → DB subnet      AZ-b   (10.0.13.0 – 10.0.13.255)
```

Each `/24` is a separate, non-overlapping street within the same city (`10.0`). Notice the third octet (`1`, `2`, `3`, `11`, `12`, `13`) is what keeps them distinct — and there's plenty of room left (`10.0.4.x`, `10.0.20.x`, …) to add more later.

**Why `/24` for subnets?** It's the sweet spot: 256 addresses is plenty for most tiers, and the math is dead easy because the whole third octet is the "subnet number" and the whole fourth octet is the "host number." You *can* use `/25`, `/26`, `/28`, etc. for smaller subnets — the same rules apply, the boundaries just fall on less-obvious numbers.

### 2.8 Private IP ranges — which numbers to use (RFC 1918)

VPCs use *private* address ranges — blocks reserved for internal networks that are **not** routable on the public internet (so many companies can reuse the same `10.x` internally without conflict). Always pick your VPC CIDR from one of these three:

| Private range (CIDR) | Address span | Common use |
|----------------------|--------------|------------|
| `10.0.0.0/8` | `10.0.0.0` – `10.255.255.255` | Big networks; the usual choice for AWS VPCs |
| `172.16.0.0/12` | `172.16.0.0` – `172.31.255.255` | Medium networks |
| `192.168.0.0/16` | `192.168.0.0` – `192.168.255.255` | Home / small office routers |

**Tip:** Pick a range that won't collide with your *other* networks (your office LAN, other VPCs). If two networks you later want to connect both use `10.0.0.0/16`, peering and VPN become painful or impossible — overlapping addresses are ambiguous to routers. Plan your address space up front.

### 2.9 The catch: AWS reserves 5 addresses in every subnet

A `/24` subnet like `10.0.1.0/24` looks like 256 usable IPs — but you actually get only **251**. AWS silently reserves **5 addresses in every subnet** for its own plumbing:

| Address | Reserved for |
|---------|-------------|
| `10.0.1.0` | Network address (the "name" of the subnet itself) |
| `10.0.1.1` | The VPC router |
| `10.0.1.2` | AWS DNS server |
| `10.0.1.3` | Reserved for future use |
| `10.0.1.255` | Broadcast address (AWS doesn't use broadcast, but reserves it anyway) |

**Why this bites people:** it matters most on *small* subnets. A `/28` has 16 addresses but only **11 usable** (16 − 5). A `/29` has 8 addresses but only **3 usable**. So don't size a subnet right at the edge of what you need — leave headroom, and remember the "minus 5" whenever you do the math.

### 2.10 Quick recap

- An IP address is a **32-bit number**, split into a **network part** and a **host part**.
- **CIDR** (`/number`) draws the line: the number is how many left-hand bits are the fixed network part.
- **Bigger slash = smaller range.** `2^(32 − prefix)` gives the address count.
- `0.0.0.0/0` = everything (internet); `x.x.x.x/32` = one host.
- We **subnet** to spread across AZs, separate public from private, contain security blast radius, and keep routing clean.
- Subnets are just smaller, non-overlapping CIDRs carved out of the VPC's CIDR.
- Use RFC 1918 private ranges, avoid overlaps, and remember AWS eats **5 IPs per subnet**.

---

## 3. VPC — Your Private Data Center in the Cloud

We'll build this up the same way as CIDR: *why* it exists first, then the details.

### 3.1 The problem a VPC solves

Imagine AWS as one gigantic shared building full of millions of other companies' servers. If you just launched a server into that shared space, some obvious questions arise:

- How do I keep *my* servers separate from everyone else's?
- How do I stop strangers from reaching my database?
- How do I decide which of my machines face the internet and which stay hidden?
- How do I give my machines predictable addresses so they can find each other?

A **VPC (Virtual Private Cloud)** is AWS's answer. It's your own **private, walled-off network** inside AWS that *you* fully control — your fenced-off plot of land in that giant building. Nothing inside it is reachable from the outside world unless *you* explicitly open a door. Nobody else's traffic wanders in.

> **One-line definition:** A VPC is a logically isolated virtual network where you control the IP address range, the subnets, the routing, and the firewalls.

### 3.2 What "your own network" actually means

When you create a VPC you're handed the controls that, in a physical data center, would require racks of hardware and a networking team:

| You control… | Which means you decide… |
|--------------|-------------------------|
| **The IP range (CIDR)** | The address space of your whole network, e.g. `10.0.0.0/16` (from §2) |
| **Subnets** | How you slice that range into zones (public/private, per-AZ) |
| **Route tables** | Where traffic is allowed to flow (the "roads" — see §5) |
| **Gateways** | Whether/how traffic reaches the internet (IGW, NAT — see §6) |
| **Firewalls** | Who can talk to whom, on which ports (Security Groups, NACLs — see §7) |

Everything else you build — EC2 servers, databases (RDS), load balancers, containers — lives *inside* subnets *within* this VPC. The VPC is the container for your entire application's network.

### 3.3 Key facts about a VPC

- A VPC lives in **exactly one AWS Region** (e.g. `ap-south-1` = Mumbai) but it **spans all the Availability Zones** in that Region. So one VPC can hold servers in multiple data centers within Mumbai.
- You define its address space with a **CIDR block**, e.g. `10.0.0.0/16` (65,536 private addresses to hand out).
- A VPC is **free** — you pay for what runs *inside* it (servers, NAT gateways, data transfer), not for the VPC itself.
- Every AWS account comes with a **default VPC** per Region so you can launch something instantly. But for real workloads you create your own **custom VPC**, because the default one puts everything in *public* subnets — not what you want for production.

### 3.4 Region vs. Availability Zone — and why you must care

This distinction drives almost every design decision, so let's be precise:

- A **Region** is a geographic location — Mumbai (`ap-south-1`), N. Virginia (`us-east-1`), Frankfurt (`eu-central-1`), etc. You pick a Region close to your users (lower latency) and that meets data-residency rules.
- An **Availability Zone (AZ)** is one (or more) physically separate data center *within* a Region — its own building, power, and cooling. AZs in a Region are isolated from each other's failures, yet joined by fast, private, low-latency links.

**Why this matters:** data centers fail — power cuts, floods, hardware faults. If *all* your servers sit in one AZ and that AZ goes down, your whole app goes down. So the golden rule:

> **Spread every tier of your app across at least 2 AZs.** Because a subnet lives in only *one* AZ, "using 2 AZs" in practice means "duplicate your subnets into a second AZ."

```
Region: ap-south-1 (Mumbai)
 ┌─────────────────────────────────────────────────────┐
 │  VPC 10.0.0.0/16                                      │
 │                                                       │
 │  ┌── AZ ap-south-1a ──┐      ┌── AZ ap-south-1b ──┐   │
 │  │  subnet 10.0.1.0/24│      │ subnet 10.0.11.0/24│   │
 │  │  subnet 10.0.2.0/24│      │ subnet 10.0.12.0/24│   │
 │  └────────────────────┘      └────────────────────┘   │
 │     (data center A)             (data center B)       │
 │            └──────── if A dies, B keeps serving ──────┘│
 └─────────────────────────────────────────────────────┘
```

### 3.5 Why do you need an ALB (Load Balancer)? — the "why" behind it

This is where the design comes together. Once you've spread your servers across 2 AZs, a new problem appears — and the **Application Load Balancer (ALB)** is the answer.

**The problem, step by step:**

1. You now run, say, **two app servers** — one in AZ-a, one in AZ-b (for high availability). Good.
2. But a user's browser can only connect to **one address**. Which server's IP do you hand out? If you publish server A's IP and server A dies, everyone is broken — the second server was pointless.
3. Even if both are alive, how do you split visitors *evenly* so one server isn't overloaded while the other sits idle?
4. When traffic spikes and you add a *third* server, how do users find it without you re-publishing addresses?
5. And you *don't* want your app servers exposed to the internet directly anyway (security).

A single fixed server IP can't solve any of this. You need **one stable front door** that sits in front of all your servers and intelligently distributes traffic. That front door is the **load balancer**.

**What the ALB does for you (each point maps to a problem above):**

- **One stable entry point.** Users (and DNS) point at the ALB's single address. Servers can come and go behind it without anyone noticing. *(solves #2, #4)*
- **Spreads the load.** It distributes incoming requests across all healthy servers so no single one is overwhelmed. *(solves #3)*
- **Health checks.** It continuously pings each server; if one becomes unhealthy, the ALB **stops sending it traffic** and routes everyone to the survivors — automatically, within seconds. *(solves #2)*
- **Spans AZs.** A single ALB has nodes in multiple AZs, so it keeps working even if an entire AZ fails. *(solves the HA goal)*
- **Lets your servers stay private.** The ALB lives in the *public* subnets and is the only thing exposed to the internet; your actual app servers live in *private* subnets and only accept traffic *from the ALB*. *(solves #5 — this is the security pattern in §7 and §8)*
- **Smart Layer-7 routing (the "Application" part).** Because an ALB understands HTTP, it can route by URL path or hostname — e.g. send `/api/*` to one group of servers and `/images/*` to another, or `shop.example.com` vs `blog.example.com` to different apps. It can also terminate HTTPS/TLS for you.

```
                    Internet
                       │
                 ┌─────┴─────┐
                 │    ALB    │  ← ONE stable front door, spans AZs,
                 │(public)   │    health-checks every server
                 └──┬─────┬──┘
        health OK?  │     │  health OK?
             ┌──────┘     └──────┐
             ▼                   ▼
      ┌────────────┐      ┌────────────┐
      │ App server │      │ App server │
      │  (AZ-a)    │      │  (AZ-b)    │   ← private, only accept
      │  private   │      │  private   │     traffic FROM the ALB
      └────────────┘      └────────────┘
        If AZ-a dies, the ALB simply sends everyone to AZ-b.
```

> **In one sentence:** an ALB exists because multiple servers across multiple AZs need a *single, always-on, self-healing front door* — and it keeps your real servers safely hidden while it faces the internet.

*(There are other load-balancer types — NLB for raw TCP/UDP performance, GWLB for security appliances — covered in §11. The ALB is the default for web/HTTP apps.)*

---

## 4. Subnets — Dividing Your VPC

A **subnet** is a slice of your VPC's IP range that lives in **one specific AZ**. You put resources into subnets. Subnets are how you separate "public-facing" things from "private, protected" things.

### Public vs. Private subnets

There is **no checkbox** called "public." A subnet is *public* purely because of **how it's routed**:

- **Public subnet** = its route table has a route to an **Internet Gateway**. Resources here *can* have public IPs and talk to the internet directly.
- **Private subnet** = **no** direct route to an Internet Gateway. Resources here are hidden from the internet. They reach out (for updates, API calls) via a **NAT Gateway** that lives in a public subnet.

That's the whole secret: **public vs private is decided by the route table, nothing else.**

### A typical three-tier subnet layout

| Subnet | CIDR | AZ | Purpose | Internet? |
|--------|------|-----|---------|-----------|
| Public A | `10.0.1.0/24` | 1a | Load balancer, NAT GW | Inbound + outbound |
| Public B | `10.0.11.0/24` | 1b | Load balancer, NAT GW | Inbound + outbound |
| App A | `10.0.2.0/24` | 1a | Application servers | Outbound only (via NAT) |
| App B | `10.0.12.0/24` | 1b | Application servers | Outbound only (via NAT) |
| DB A | `10.0.3.0/24` | 1a | Database | None |
| DB B | `10.0.13.0/24` | 1b | Database | None |

Notice the pattern: **each tier duplicated across two AZs** for high availability.

---

## 5. Route Tables — The GPS of Your Network

A **route table** is a set of rules that says: *"For a packet heading to destination X, send it to target Y."* Every subnet is associated with exactly one route table (if you don't attach one, it uses the VPC's **main** route table).

### How a route is evaluated

AWS uses **longest-prefix match** — the *most specific* matching route wins.

Example route table for a **public** subnet:

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | `local` | "Anything inside my VPC — deliver locally." (always present, can't be removed) |
| `0.0.0.0/0` | `igw-abc123` | "Everything else (the internet) — send to the Internet Gateway." |

Example route table for a **private** subnet:

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | `local` | Traffic inside the VPC stays local. |
| `0.0.0.0/0` | `nat-xyz789` | Internet-bound traffic goes to the NAT Gateway (outbound only). |

**Longest-prefix example:** if a packet is destined for `10.0.5.7`, both `10.0.0.0/16` and `0.0.0.0/0` "match," but `/16` is more specific, so it wins and stays local. Good — you never want internal traffic leaving via the internet.

---

## 6. Internet Gateway & NAT — Getting In and Out

### Internet Gateway (IGW)

- One IGW per VPC. It's the **front gate to the public internet**.
- It does two jobs: routes traffic between the VPC and the internet, and performs **1:1 NAT** for instances that have a public IP.
- An instance can talk to the internet **only if all three are true**:
  1. It has a public IP (or Elastic IP).
  2. Its subnet's route table points `0.0.0.0/0` at the IGW.
  3. Security Group + NACL allow the traffic.

### NAT Gateway — outbound-only for private subnets

Your private app servers need to download OS patches, call external APIs, etc. — but you do **not** want the internet initiating connections *to* them. A **NAT (Network Address Translation) Gateway** solves this:

- It lives in a **public** subnet and has an Elastic (public) IP.
- Private instances route `0.0.0.0/0` to the NAT Gateway.
- The NAT GW forwards their requests to the internet using its own public IP, and returns responses.
- **Connections can only start from the inside.** The internet cannot use the NAT GW to reach in. This is the "one-way delivery door."

```
Private EC2 (10.0.2.15)  ──►  NAT GW (public subnet)  ──►  IGW  ──►  Internet
        ▲                                                              │
        └──────────────── response comes back ─────────────────────────┘
  (Internet can NEVER initiate a connection back through the NAT GW)
```

> **IGW vs NAT GW in one line:**
> IGW = two-way door for public-facing resources.
> NAT GW = one-way (outbound) door for private resources.

**Cost note:** NAT Gateways are billed per hour *and* per GB processed. For dev environments people sometimes use a cheaper NAT *instance* or an S3/ECR gateway endpoint to avoid NAT data charges (see §12).

---

## 7. Security Groups & NACLs — The Firewalls

AWS gives you **two layers** of packet filtering. Understanding the difference is essential.

### Security Group (SG) — the bodyguard on the instance

- Operates at the **instance / ENI level** (attached to the resource, not the subnet).
- **Stateful:** if you allow a request *in*, the response is automatically allowed *out* (and vice versa). You don't write return rules.
- **Allow rules only** — you cannot write an explicit "deny." Anything not allowed is implicitly denied.
- Evaluated as a whole — all rules across all attached SGs are considered.
- You can reference **another security group** as a source (e.g. "allow the app SG to reach the DB SG"). This is powerful and avoids hard-coding IPs.

Example SG for a web server:

| Direction | Type | Protocol | Port | Source |
|-----------|------|----------|------|--------|
| Inbound | HTTPS | TCP | 443 | `0.0.0.0/0` |
| Inbound | SSH | TCP | 22 | `203.0.113.10/32` (your office IP) |
| Outbound | All | All | All | `0.0.0.0/0` (default) |

### Network ACL (NACL) — the checkpoint at the subnet border

- Operates at the **subnet level** — every packet entering or leaving the subnet is checked.
- **Stateless:** it does *not* remember connections. You must explicitly allow **both** the inbound request **and** the outbound response (which uses "ephemeral" high ports, typically `1024–65535`).
- Supports **both allow and deny** rules.
- Rules are numbered and evaluated **in order, lowest first**; the first match wins.
- Default NACL allows all traffic; custom NACLs deny all until you add rules.

### Side-by-side

| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| Level | Instance / ENI | Subnet |
| State | **Stateful** (auto return traffic) | **Stateless** (must allow both ways) |
| Rules | Allow only | Allow **and** Deny |
| Evaluation | All rules together | Numbered order, first match wins |
| Default | Deny all inbound, allow all outbound | Default NACL: allow all |
| Typical use | Primary control (use this 95% of the time) | Coarse subnet-wide guardrails, blocking a bad IP |

> **Mental model:** The SG is a bouncer checking each guest at the club door and remembering who's inside so they can leave freely. The NACL is a checkpoint at the neighborhood gate that checks *every* car both entering and leaving, with no memory.

**Practical advice:** Do most of your work with Security Groups. Leave NACLs at their permissive default unless you have a specific reason (compliance, blocking a known-bad CIDR).

---

## 8. A Complete Request, Traced End-to-End

Let's follow a user in a browser loading `https://app.example.com`, which is a web app running on private EC2 instances behind a load balancer. This ties **everything** together.

```
                                          ┌──────────────┐
 1. Browser asks DNS: where is            │   Route 53   │
    app.example.com?  ───────────────────►│    (DNS)     │
                                          └──────┬───────┘
 2. Route 53 returns the ALB's address           │
    ◄────────────────────────────────────────────┘
 3. Browser connects to ALB over HTTPS
        │
        ▼
 ╔══════════════════ VPC 10.0.0.0/16 ══════════════════════╗
 ║                                                          ║
 ║   ┌─────────────── Public Subnet ───────────────┐        ║
 ║   │  Internet Gateway  ──►  [ ALB ]  :443        │        ║
 ║   └────────────────────────┬─────────────────────┘       ║
 ║   4. ALB SG allows 443 in from internet                  ║
 ║                            │                             ║
 ║                            ▼                             ║
 ║   ┌────────────── Private App Subnet ────────────┐       ║
 ║   │   [ EC2 app ]  :8080                          │       ║
 ║   │   5. App SG allows 8080 ONLY from ALB's SG    │       ║
 ║   └────────────────────────┬─────────────────────┘       ║
 ║                            │                             ║
 ║                            ▼                             ║
 ║   ┌────────────── Private DB Subnet ─────────────┐       ║
 ║   │   [ RDS ]  :5432                              │       ║
 ║   │   6. DB SG allows 5432 ONLY from App's SG     │       ║
 ║   └───────────────────────────────────────────────┘      ║
 ╚══════════════════════════════════════════════════════════╝
```

**Step by step:**

1. **DNS lookup.** The browser asks Route 53 (or whatever DNS) for `app.example.com`. Route 53 returns the load balancer's IP/alias.
2. **Reach the ALB.** The browser opens an HTTPS (port 443) connection to the Application Load Balancer, which sits in the **public** subnets. Traffic enters through the **Internet Gateway**.
3. **ALB Security Group** allows inbound 443 from `0.0.0.0/0`. The ALB terminates TLS.
4. **ALB → App.** The ALB forwards the request to a healthy EC2 app instance in a **private** subnet on port 8080. The **app's SG** allows 8080 **only from the ALB's security group** — not from the whole internet. The app instances have no public IP and are unreachable directly.
5. **App → Database.** The app queries RDS on port 5432. The **DB's SG** allows 5432 **only from the app's SG**. The DB subnet has no internet route at all.
6. **Responses flow back** the same path. Because Security Groups are **stateful**, all the return traffic is automatically permitted.
7. **If the app needs the internet** (e.g. to call a third-party API), its outbound `0.0.0.0/0` route points at the **NAT Gateway** in the public subnet — outbound only.

This layered design (public LB → private app → isolated DB, each SG referencing the one above it) is the canonical secure AWS pattern.

---

## 9. Connecting VPCs and On-Premises

Eventually you'll need to connect networks together. Here are your options, easiest to most scalable.

### VPC Peering — a private link between two VPCs

- A **one-to-one** private connection between two VPCs (same or different account/Region).
- Traffic stays on the AWS private network — never touches the internet.
- **Not transitive:** if A peers with B, and B peers with C, then A **cannot** reach C through B. You'd need a direct A–C peering.
- CIDRs **must not overlap.**
- Good for a small number of VPCs. Becomes a mess (n² connections) as you grow.

```
   VPC A ──peer── VPC B ──peer── VPC C
   (A and C CANNOT talk — peering is not transitive)
```

### Transit Gateway (TGW) — the network hub

- A central hub that connects **many** VPCs and on-prem connections in a **hub-and-spoke** model.
- **Transitive:** everything attached to the TGW can (subject to route tables) reach everything else. Solves the peering-mesh explosion.
- The standard choice once you have more than a handful of VPCs.

```
        VPC A     VPC B     VPC C
          │         │         │
          └────┬────┴────┬────┘
             ┌─┴─────────┴─┐
             │  Transit    │──── VPN / Direct Connect ──── On-Prem
             │  Gateway    │
             └─────────────┘
```

### Connecting to your data center (on-premises)

- **Site-to-Site VPN:** an encrypted tunnel over the public internet between your VPC (via a Virtual Private Gateway or TGW) and your on-prem router. Quick to set up, but subject to internet variability.
- **AWS Direct Connect:** a **dedicated physical fiber** link from your data center to AWS. Consistent low latency and high bandwidth, private. Takes longer to provision and costs more. Often paired with a VPN as encrypted backup.

### PrivateLink — expose a single service privately

- Lets you expose (or consume) a **specific service** across VPCs without peering the whole networks, using an **interface endpoint** (an ENI with a private IP in your subnet).
- The connection is one-directional (consumer → service) and doesn't require overlapping-free CIDRs or route sharing. Great for SaaS providers and shared internal services.

**Quick chooser:**

| Need | Use |
|------|-----|
| Connect 2 VPCs, simple | VPC Peering |
| Connect many VPCs + on-prem, at scale | Transit Gateway |
| Encrypted link to your office over internet | Site-to-Site VPN |
| Dedicated, consistent link to your data center | Direct Connect |
| Expose ONE service privately across VPCs | PrivateLink |

---

## 10. DNS with Route 53

**Route 53** is AWS's DNS service — the campus phone directory that turns human names into IP addresses.

### Core record types you'll actually use

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Name → IPv4 address | `app.example.com → 52.1.2.3` |
| **AAAA** | Name → IPv6 address | `app.example.com → 2600:...` |
| **CNAME** | Alias one name to another name | `www → app.example.com` |
| **Alias** | AWS-specific "smart CNAME" that can point to AWS resources (ALB, CloudFront, S3) even at the zone apex, and is **free** to query | `example.com → my-alb-...elb.amazonaws.com` |
| **MX** | Mail servers | for email routing |
| **TXT** | Arbitrary text | domain verification, SPF/DKIM |

> **Alias vs CNAME:** Use an **Alias** record when pointing at an AWS resource — it works at the root domain (`example.com`, where CNAME is illegal) and has no per-query charge.

### Routing policies (Route 53's superpower)

Route 53 can return *different* answers based on rules:

- **Simple** — one static answer.
- **Weighted** — split traffic by percentage (great for canary/blue-green: send 10% to the new version).
- **Latency-based** — send users to the Region with the lowest latency for them.
- **Failover** — return the primary; if a health check fails, return the secondary (disaster recovery).
- **Geolocation / Geoproximity** — answer based on where the user is.

### Private Hosted Zones

A hosted zone can be **private** — resolvable only inside your VPC(s). This lets you use nice internal names (`db.internal.example.com`) that never leak to the public internet.

---

## 11. Load Balancing & Content Delivery

### Elastic Load Balancers — the reception desk

A load balancer accepts incoming traffic and spreads it across multiple healthy targets. It also does **health checks** and stops sending traffic to unhealthy instances.

| Type | Layer | Best for | Key features |
|------|-------|----------|--------------|
| **ALB** (Application) | Layer 7 (HTTP/HTTPS) | Web apps, microservices, containers | Path/host-based routing (`/api` → service A, `/img` → service B), TLS termination, WebSockets |
| **NLB** (Network) | Layer 4 (TCP/UDP) | Extreme performance, static IP, non-HTTP | Millions of req/sec, ultra-low latency, preserves source IP |
| **GWLB** (Gateway) | Layer 3 | Inserting security appliances (firewalls, IDS) | Transparent traffic inspection |

*(The older "Classic Load Balancer" still exists but is legacy — use ALB/NLB.)*

**Key building blocks:**
- **Listener** — the port/protocol the LB listens on (e.g. HTTPS:443).
- **Target group** — the set of backends (EC2, IPs, Lambda, containers) plus their health-check config.
- **Rules** — on an ALB, listener rules decide which target group gets a request based on path, host, headers, etc.

### CloudFront — the global cache (CDN)

**CloudFront** is a Content Delivery Network: it caches your content at **edge locations** worldwide, close to users.

- A user in Sydney gets content from a nearby Sydney edge rather than crossing the ocean to your origin — faster, and it offloads your servers.
- The **origin** can be S3, an ALB, or any HTTP server.
- Provides TLS, DDoS protection (with AWS Shield), and integrates with **AWS WAF** (Web Application Firewall) for filtering malicious requests.
- Use it for static assets, whole web apps, and API acceleration.

```
User (Sydney) ─► CloudFront edge (Sydney) ─cache miss─► Origin (ALB/S3 in Mumbai)
User (Sydney) ─► CloudFront edge (Sydney) ─cache hit──► served instantly from edge
```

---

## 12. VPC Endpoints — Private Access to AWS Services

By default, when your private EC2 instance calls S3 or DynamoDB, that traffic goes **out through the NAT Gateway and across the public internet** (to a public AWS endpoint). That costs NAT data charges and leaves your VPC. **VPC Endpoints** keep that traffic on the AWS private network.

### Two kinds

1. **Gateway Endpoint** — for **S3** and **DynamoDB** only.
   - It's a **route table entry** (a target), not an ENI.
   - **Free.** Just add it and route S3/DynamoDB traffic through it.
   - Best-practice for any private subnet that touches S3 (saves NAT costs).

2. **Interface Endpoint (PrivateLink)** — for **most other AWS services** (SQS, SNS, KMS, Secrets Manager, ECR, etc.) and your own PrivateLink services.
   - Creates an **ENI with a private IP** in your subnet.
   - Your instances talk to the service via that private IP — never leaving the VPC.
   - Billed per hour + per GB, but usually cheaper and more secure than routing via NAT.

> **Why you care:** Endpoints improve security (traffic never touches the internet), reduce cost (no NAT data processing for that traffic), and often reduce latency.

---

## 13. A Reference Production Architecture

Putting it all together — a resilient, secure, two-AZ web application:

```
                              Users
                                │
                          ┌─────┴─────┐
                          │ Route 53  │  (DNS, health-checked)
                          └─────┬─────┘
                                │
                          ┌─────┴─────┐
                          │CloudFront │  (CDN + WAF)
                          └─────┬─────┘
                                │
  ╔══════════════ VPC 10.0.0.0/16 (Region: ap-south-1) ══════════════╗
  ║                                                                   ║
  ║        Internet Gateway ───────────┐                              ║
  ║                                     │                             ║
  ║   ┌── Public Subnet AZ-a ──┐  ┌── Public Subnet AZ-b ──┐          ║
  ║   │   [ ALB node ]         │  │   [ ALB node ]         │          ║
  ║   │   [ NAT GW a ]         │  │   [ NAT GW b ]         │          ║
  ║   └──────────┬─────────────┘  └──────────┬─────────────┘          ║
  ║              │        (ALB spans both AZs)                        ║
  ║   ┌── Private App Subnet a ─┐ ┌── Private App Subnet b ─┐         ║
  ║   │  [ EC2 / ECS tasks ]    │ │  [ EC2 / ECS tasks ]    │         ║
  ║   │   (Auto Scaling Group)  │ │   (Auto Scaling Group)  │         ║
  ║   └──────────┬──────────────┘ └──────────┬──────────────┘         ║
  ║              │                           │                        ║
  ║   ┌── Private DB Subnet a ──┐ ┌── Private DB Subnet b ──┐         ║
  ║   │  [ RDS primary ]        │ │  [ RDS standby (Multi-AZ)]│       ║
  ║   └─────────────────────────┘ └─────────────────────────┘         ║
  ║                                                                   ║
  ║   Gateway Endpoint → S3   (private, free)                         ║
  ║   Interface Endpoints → Secrets Manager, ECR, etc.               ║
  ╚═══════════════════════════════════════════════════════════════════╝
```

**Why this is a good design:**
- **Two AZs** everywhere → survives a data-center failure.
- **Three tiers** (public LB / private app / isolated DB) → defense in depth.
- **SG chaining** (ALB-SG → App-SG → DB-SG) → each layer only accepts traffic from the layer above.
- **NAT GW per AZ** → no cross-AZ dependency or single point of failure for outbound.
- **Auto Scaling Group** → app tier grows/shrinks with load.
- **RDS Multi-AZ** → automatic database failover.
- **VPC Endpoints** → private, cheaper access to AWS services.
- **CloudFront + WAF** → global speed and a security shield at the edge.

---

## 14. Common Gotchas & Troubleshooting

A checklist for "why can't X reach Y?" — walk it in order:

1. **Route table:** Does the source subnet have a route to the destination? (Internet needs `0.0.0.0/0` → IGW/NAT; another VPC needs a peering/TGW route.)
2. **Security Group (destination):** Does the *destination's* SG allow inbound on the right port from the source? Remember SGs are stateful, so you rarely need outbound rules.
3. **Security Group (source):** Default SG allows all outbound, but if you've locked it down, check outbound is allowed.
4. **NACL:** If you use custom NACLs, did you allow **both** the inbound rule **and** the outbound **ephemeral port** (1024–65535) return traffic? This is the #1 stateless-firewall trap.
5. **Public IP:** For direct internet access, does the instance actually have a public/Elastic IP? (Behind an ALB it doesn't need one.)
6. **Overlapping CIDRs:** Peering/VPN will fail or misroute if ranges overlap.
7. **DNS:** Is `enableDnsSupport` and `enableDnsHostnames` on for the VPC? Private hosted zone associated with the right VPC?
8. **NAT direction:** Remember a NAT GW is **outbound only** — nothing on the internet can start a connection inward through it. If you need inbound, you need an IGW + public IP or a load balancer.
9. **Subnet AZ mismatch:** An ALB needs subnets in at least 2 AZs; RDS Multi-AZ needs a subnet group spanning AZs.
10. **IGW attached?** An Internet Gateway must be *attached* to the VPC, and the route must point to it, for public access to work.

**Handy tools:** VPC **Reachability Analyzer** (tells you exactly where a path is blocked), **VPC Flow Logs** (records accepted/rejected traffic per ENI — great for spotting a NACL/SG deny).

---

## 15. Glossary

| Term | Meaning |
|------|---------|
| **VPC** | Virtual Private Cloud — your isolated network in AWS. |
| **CIDR** | Notation for an IP range, e.g. `10.0.0.0/16`. |
| **Subnet** | A slice of a VPC's IP range within one AZ. |
| **AZ** | Availability Zone — an isolated data center within a Region. |
| **Region** | A geographic AWS location containing multiple AZs. |
| **Route Table** | Rules mapping destination CIDRs to targets. |
| **IGW** | Internet Gateway — two-way door to the internet. |
| **NAT GW** | Network Address Translation Gateway — one-way outbound door for private subnets. |
| **SG** | Security Group — stateful, instance-level, allow-only firewall. |
| **NACL** | Network ACL — stateless, subnet-level, allow+deny firewall. |
| **ENI** | Elastic Network Interface — a virtual NIC with a private IP. |
| **Elastic IP** | A static, public IPv4 address you own. |
| **Peering** | Private 1:1 link between two VPCs (non-transitive). |
| **TGW** | Transit Gateway — hub connecting many VPCs/on-prem (transitive). |
| **PrivateLink** | Private access to a single service via an interface endpoint. |
| **VPN** | Encrypted tunnel to on-prem over the internet. |
| **Direct Connect** | Dedicated physical link to AWS. |
| **Route 53** | AWS DNS service. |
| **ALB / NLB / GWLB** | Application / Network / Gateway Load Balancers. |
| **CloudFront** | AWS Content Delivery Network (CDN). |
| **VPC Endpoint** | Private connection from your VPC to an AWS service (Gateway or Interface). |
| **Flow Logs** | Records of traffic to/from network interfaces. |

---

### How to keep learning

1. **Build it once by hand** in the console: a VPC, 2 public + 2 private subnets, an IGW, a NAT GW, route tables, and one EC2 behind an ALB. Nothing cements this like doing it.
2. Then **rebuild it as code** (CloudFormation or Terraform) so it's repeatable.
3. Turn on **VPC Flow Logs** and watch real traffic — deliberately break a Security Group rule and see the REJECT appear.
4. Use the **Reachability Analyzer** whenever something can't connect.

*You now have the full mental model: land (VPC) → buildings (subnets) → maps (route tables) → doors (IGW/NAT) → guards (SG/NACL) → directory (Route 53) → reception (LB) → global cache (CloudFront), plus the ways to connect it all together.*
