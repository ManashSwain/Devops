# Networking Foundations

The concepts every DevOps engineer must know before touching IPs, ports, DNS, routing, or cloud networking.

---

## Table of Contents

- [What is a Computer Network?](#what-is-a-computer-network)
- [What is the Internet?](#what-is-the-internet)
- [Internet vs WWW](#internet-vs-www)
- [Client-Server Architecture](#client-server-architecture)
- [Types of Networks](#types-of-networks)
- [Network Topologies](#network-topologies)
- [Summary](#summary)
- [Quick Revision](#quick-revision)

---

# What is a Computer Network?

## Definition

> A **computer network** is a collection of two or more computing devices connected together — by cables or wirelessly — so they can **share data, resources, and services**.

Two devices are "connected" when they can exchange data using a **common set of rules (protocols)**. Without protocols, the hardware link is useless.

## Key Components

| Component | Role | Examples |
|---|---|---|
| **Nodes / Hosts** | Devices that send or receive data | Laptops, servers, phones, printers, IoT devices |
| **Transmission medium** | The path data travels over | Ethernet cable, fiber, Wi-Fi, radio |
| **Network devices** | Connect, direct, and secure traffic | Switch, router, hub, firewall |
| **Protocols** | Rules for how data is formatted and exchanged | TCP, IP, HTTP, DNS, ARP |
| **Services / Resources** | What the network makes available | Files, printers, web pages, databases |

## Why Networks Exist

1. **Resource sharing** — one printer or file server for many users.
2. **Communication** — email, chat, video calls.
3. **Centralized data** — everyone works from the same source of truth.
4. **Scalability** — add more compute (servers, cloud instances) instead of buying one huge machine.
5. **Reliability** — if one node fails, others can continue (especially in mesh designs).

## Simple Diagram

```text
        Node            Medium               Node
   +-----------+    +-------------+    +-----------+
   |  Laptop   |----|  Ethernet   |----|  Server   |
   +-----------+    +-------------+    +-----------+
          \                               /
           \     +---------------------+ /
            \____|  Switch / Router    |/
                 |  (network devices)  |
                 +---------------------+
```

## Mental Model

```text
Network = Nodes + Links + Protocols
          (who)   (how)   (rules)
```

---

# What is the Internet?

## Definition

> The **Internet** is a **global network of networks** — millions of smaller networks (home, office, ISP, cloud, government) interconnected so any connected device can reach any other, using the **TCP/IP protocol suite**.

It is **not** one company, not one cable, and not the same thing as the web. It is the *infrastructure*.

## Characteristics

- **Decentralized** — no single owner; coordinated through standards bodies (IETF, ICANN, IANA).
- **Packet-switched** — data is split into small **packets**, each routed independently and reassembled at the destination.
- **Best-effort** — IP makes no guarantee of delivery, ordering, or timing; TCP adds reliability on top.
- **Layered** — built on the TCP/IP model (Application → Transport → Internet → Link).

## How Devices Reach the Internet

```text
 +--------+    +---------+    +-----------+    +--------------+    +-------------+
 | Home   |    | Home    |    |  Local    |    |  Regional    |    |  Tier-1     |
 | Device |----| Router  |----|  ISP      |----|  / National  |----|  Backbone   |
 +--------+    +---------+    | (ISP)     |    |  ISP         |    |  Provider   |
                              +-----------+    +--------------+    +------+------+
                                                                          |
                                                                          | Fiber
                                                                          |
                                                                    +-----+------+
                                                                    | Tier-1     |
                                                                    | Peer/IXP   |
                                                                    +-----+------+
                                                                          |
                                                                  (Other networks,
                                                                   servers, clouds)
```

- **ISP (Internet Service Provider)** — company that gives you access (Jio, Airtel, Comcast).
- **Tier 1 / backbone** — large carriers that own long-haul fiber and peer with each other.
- **IXP (Internet Exchange Point)** — physical facility where networks directly exchange traffic, cutting cost and latency.
- **The "network of networks"** — each ISP, company, and cloud is an **Autonomous System (AS)** with its own routing policy (BGP between them).

## A Packet's Journey (Simplified)

```text
1. Application creates data        e.g. HTTP request
2. Transport adds TCP segment      ports, ordering, reliability
3. Internet adds IP packet         source IP -> destination IP
4. Link adds frame                 MAC addresses on local link
5. Routed hop-by-hop               router -> ISP -> backbone -> ISP -> destination
6. Destination reverses layers     IP removed, TCP reassembled, app receives data
```

---

# Internet vs WWW

## Definition of WWW

> The **World Wide Web (WWW)** is an **information system** built *on top of* the Internet, where documents and resources are identified by **URLs**, transferred by **HTTP/HTTPS**, and rendered by **browsers**.

Invented by **Tim Berners-Lee (1989)**. It is **one service** among many that run on the Internet.

## The Core Distinction

| | **Internet** | **World Wide Web** |
|---|---|---|
| **What it is** | Physical + logical infrastructure | A service / application running over it |
| **Layer** | Network layer and below | Application layer (HTTP/HTTPS) |
| **Nature** | Hardware, cables, routers, IP | Documents (HTML), hyperlinks, URLs |
| **Invented** | 1960s–70s (ARPANET) | 1989 (Tim Berners-Lee) |
| **Access with** | Any IP-capable device | A browser / HTTP client |
| **Examples** | Email (SMTP), SSH, FTP, VoIP, DNS | Websites, web apps, web APIs |
| **Analogy** | Roads and highways | Shops and buildings along the roads |

## Diagram

```text
        +---------------------------------------------+
        |                 INTERNET                    |
        |     (infrastructure: IP, routers, cables)   |
        |                                             |
        |   +-----------------+   +----------------+  |
        |   |      WWW        |   |  Email (SMTP)  |  |
        |   | HTTP / HTTPS    |   |  SSH, FTP      |  |
        |   | HTML, URLs      |   |  VoIP, DNS     |  |
        |   +--------+--------+   +----------------+  |
        +------------|--------------------------------+
                     |
              +------+------+
              |   Browser   |
              | (Chrome etc)|
              +-------------+
```

> **Key line:** *The web uses the Internet; the Internet does not need the web.* You can ping, SSH, or email without ever opening a browser.

---

# Client-Server Architecture

## Definition

> **Client-Server** is a model where a **client** requests a service and a **server** provides it. The server holds the resources or logic; the client consumes them.

This is the dominant architecture of the web, DNS, email, databases, and most cloud systems.

## Diagram

```text
   CLIENT SIDE                       SERVER SIDE
 +-------------+                 +-------------------+
 |  Browser /  |   Request -->   |                   |
 |  Mobile App |                 |   Web Server      |
 |             |   <-- Response  |   (Nginx/Apache)  |
 | (thin: UI,  |                 |        |          |
 |  rendering) |                 |        v          |
 +-------------+                 |   App Server      |
                                 |   (Node/Python)   |
                                 |        |          |
                                 |        v          |
                                 |   Database        |
                                 +-------------------+
```

## Request/Response Flow

```text
1. Client resolves server name      DNS  -> IP address
2. Client opens connection          TCP 3-way handshake (port 443)
3. Client sends request             GET /index.html
4. Server processes                 app logic, database query
5. Server sends response            HTTP 200 + body
6. Connection closes or is reused   keep-alive
```

## Example Request

```http
GET /api/users HTTP/1.1
Host: example.com
Accept: application/json
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

[{"id": 1, "name": "Alice"}]
```

## Roles and Responsibilities

| **Client** | **Server** |
|---|---|
| Initiates the request | Waits for and handles requests |
| Typically many | Typically few, centralized |
| Thin — UI and logic-lite | Thick — business logic, storage, auth |
| No permanent connection | Long-running, always available |
| Examples: browser, CLI, mobile app | Examples: web server, DB, DNS, API |

## Pros and Cons

**Advantages**
- **Centralized control** — one place for data, backups, updates, security.
- **Scalable** — add more servers or scale vertically.
- **Easier security** — protect one backend, not every device.
- **Thin clients** — cheap devices; heavy lifting on the server.

**Disadvantages**
- **Single point of failure** — if the server dies, clients stop working (mitigated by load balancers/replication).
- **Congestion** — many clients can overload one server.
- **Cost** — servers, power, maintenance, network bandwidth.

## Peer-to-Peer (P2P) — the Contrasting Model

```text
   Client-Server                    Peer-to-Peer
 +---+        +---------+        +---+      +---+
 | C1|------->|         |        | P1|<---->| P2|
 +---+        | Server  |        +---+      +---+
 +---+------->|         |          ^          ^
 | C2|        | (single |          |          |
 +---+        |  point) |        +---+      +---+
 +---+------->|         |        | P3|<---->| P4|
 | C3|        +---------+        +---+      +---+
 +---+

 Each peer is both client and server; no central authority.
 Used in: BitTorrent, blockchain, some VPNs.
```

| **Client-Server** | **Peer-to-Peer** |
|---|---|
| Centralized | Decentralized |
| Easy to manage | Hard to manage |
| Single point of failure | Resilient |
| Simple to secure | Harder to trust peers |

## Where You See It in DevOps

- Browser → **Nginx** (reverse proxy) → **App** (containers) → **PostgreSQL**
- **kubectl / CI runner** → Kubernetes **API server**
- App → **Redis / DNS / S3** (all are services with a client-server contract)
- Client-server is why **ports, load balancers, health checks, and autoscaling** exist.

---

# Types of Networks

Networks are classified mainly by **geographic scope**, and secondarily by ownership and purpose.

## Comparison at a Glance

| Type | Full Name | Scope | Typical Example |
|---|---|---|---|
| **PAN** | Personal Area Network | ~1–10 m | Phone ↔ earbuds, Bluetooth, USB tethering |
| **LAN** | Local Area Network | Building / floor | Home Wi-Fi, office Ethernet |
| **CAN** | Campus Area Network | Multiple buildings | University or corporate campus |
| **MAN** | Metropolitan Area Network | City | ISP metro network, city fiber ring |
| **WAN** | Wide Area Network | Country / global | Internet, MPLS between branches |
| **SAN** | Storage Area Network | Data center | iSCSI / Fibre Channel storage |
| **VPN** | Virtual Private Network | Logical, over any WAN | Remote worker tunnel, site-to-site |

## Scope Diagram

```text
 PAN           LAN              CAN                 MAN                 WAN
 |             |                |                   |                   |
 1-10 m       building         campus              city                country/world
 |             |                |                   |                   |
 phone+       switch +         many LANs           ISP / fiber         Internet,
 earbuds      router           interconnected     backbone rings      global

 |<---------------------------- smaller, faster, private --------------------->|
 |<---------------------------- larger, slower, often public ----------------->|
```

## PAN — Personal Area Network

**Definition:** A network around **one person**, connecting their personal devices.

```text
   +----------+
   |  Phone   |~~~~ Bluetooth ~~~~+----------+
   +----------+                  | Earbuds  |
        |                        +----------+
        | USB
   +----------+
   |  Laptop  |
   +----------+
```

- **Range:** ~1–10 m
- **Media:** Bluetooth, USB, Wi-Fi Direct
- **Used for:** personal device sync, wearables, hotspots
- **Pros:** simple, no infrastructure, low power
- **Cons:** very short range, low throughput, few devices

---

## LAN — Local Area Network

**Definition:** A network covering a **limited area** such as a home, office, or floor, usually owned by a single organization.

```text
                    LAN (single building)
   +--------+   +--------+   +--------+
   |  PC 1  |   |  PC 2  |   | Printer|
   +---+----+   +---+----+   +---+----+
       |            |            |
       +------------+------------+
                    |
              +-----+-----+
              |   Switch  |
              +-----+-----+
                    |
              +-----+-----+
              |  Router   |----> Internet (WAN)
              +-----------+
```

- **Range:** up to a few km (typically a building)
- **Media:** Ethernet, Wi-Fi
- **Speeds:** 100 Mbps – 100 Gbps
- **Ownership:** private
- **Pros:** high speed, low latency, cheap, easy to secure
- **Cons:** limited geographic area

> **DevOps relevance:** most debugging happens on a LAN — subnets, VLANs, ARP, MAC addresses.

---

## CAN — Campus Area Network

**Definition:** A network connecting **multiple LANs** within a defined campus, such as a university or corporate headquarters.

```text
   +---------+     +---------+     +---------+
   | Building|     | Building|     | Building|
   |   A     |     |   B     |     |   C     |
   | (LAN)   |     | (LAN)   |     | (LAN)   |
   +----+----+     +----+----+     +----+----+
        |               |               |
        +---------------+---------------+
                        |
                 +------+------+
                 |  Campus Core|
                 |   Switch    |----> WAN
                 +-------------+
```

- **Range:** a few km
- **Ownership:** usually one organization
- **Pros:** shared services, central management
- **Cons:** more complex than LAN, needs routing

---

## MAN — Metropolitan Area Network

**Definition:** A network spanning a **city or large metropolitan area**, typically operated by an ISP or a large enterprise.

```text
      City-wide fiber ring

         +-------------+
         |   Office 1  |
         +------+------+
                |
   +------------+------------+
   |                         |
+--+---+                 +---+--+
| ISP  |------- fiber ----| ISP  |
| POP1 |                 | POP2 |
+--+---+                 +---+--+
   |                         |
+--+---+                 +---+--+
| Office|                 | Data |
|   2   |                 |Center|
+------+                 +------+
```

- **Range:** ~10–100 km
- **Media:** fiber, microwave
- **Ownership:** ISP / carrier (leased lines)
- **Pros:** connects sites across a city at high speed
- **Cons:** expensive; depends on provider

> **POP** = Point of Presence — a local ISP facility where customers connect.

---

## WAN — Wide Area Network

**Definition:** A network covering a **large geographic area** — multiple cities, countries, or continents — linking LANs and MANs.

```text
   Office (Delhi)                                   Office (London)
   +-----------+                                    +-----------+
   |   LAN     |                                    |    LAN    |
   +-----+-----+                                    +-----+-----+
         |                                                |
   +-----+-----+          +-----------------+       +-----+-----+
   |  Router   |==========|  WAN / Internet |=======|  Router   |
   +-----------+   leased |  (MPLS, VPN,    |       +-----------+
                   line   |   public IP)    |
                          +-----------------+
```

- **Range:** country → global
- **Media:** long-haul fiber, satellite, undersea cables
- **Owner:** ISP / carrier / cloud provider
- **Pros:** connects anywhere, scalable across regions
- **Cons:** higher latency, higher cost, harder to secure — usually needs VPN/encryption
- **The Internet is the largest WAN.**

---

## SAN — Storage Area Network

**Definition:** A **dedicated high-speed network** connecting servers to centralized storage, separate from normal LAN traffic.

```text
   +--------+   +--------+   +--------+
   |Server 1|   |Server 2|   |Server 3|
   +---+----+   +---+----+   +---+----+
       |            |            |
   ====+============+============+====  SAN fabric (Fibre Channel / iSCSI)
                    |
             +------+------+
             | Storage     |
             | Array (RAID)|
             +-------------+
```

- **Used for:** databases, VMs, enterprise storage
- **Pros:** fast, isolated, highly available
- **Cons:** expensive, specialized hardware

---

## VPN — Virtual Private Network

**Definition:** A **logical private network** built on top of a public network (usually the Internet), using encryption and tunneling.

```text
   Remote Laptop                    Public Internet                 Office LAN
 +------------+    encrypted tunnel    ~~~~~~~    +------------+   +----------+
 | VPN Client |=====================>[ Encrypted ]==>| VPN Server |-->| Internal |
 +------------+      (WireGuard,      ~~~~~~~      +------------+   | resources|
                      IPSec, TLS)                                   +----------+
```

- **Types:** remote-access VPN, site-to-site VPN
- **Pros:** secure remote access, cheap compared to leased lines
- **Cons:** adds latency, must be managed and monitored

---

## LAN vs WAN (the interview classic)

| | **LAN** | **WAN** |
|---|---|---|
| **Scope** | Small (building) | Large (country/global) |
| **Ownership** | Private | Usually ISP/carrier |
| **Speed** | Very high | Lower, variable |
| **Latency** | Very low | Higher |
| **Cost** | Low | High |
| **Setup** | Simple | Complex |
| **Security** | Easier | Needs VPN/firewall |

---

# Network Topologies

> **Topology** = the **arrangement** of devices and links in a network, both **physical** (how cables run) and **logical** (how data flows).

Topology answers: *If a link breaks, what still works? How much cable do I need? How easily can I add a device?*

---

## 1. Bus Topology

All devices share **one single backbone cable**.

```text
   Terminator                                                 Terminator
      +                                                            +
      |                                                            |
 +----+----+----+----+----+----+----+----+----+----+----+----+----+
 |         |         |         |         |         |         |    |
+--+--+   +--+--+   +--+--+   +--+--+   +--+--+   +--+--+   +--+--+
| PC1 |   | PC2 |   | PC3 |   | PC4 |   | PC5 |   | PC6 |   | PC7 |
+-----+   +-----+   +-----+   +-----+   +-----+   +-----+   +-----+

           Single backbone (coaxial cable) shared by all nodes
```

- **Data flow:** one node transmits; all others hear it, only the addressed one accepts.
- **Pros:** minimal cable, cheap, simple for small setups.
- **Cons:** **one cable break kills the whole network**; collisions; hard to troubleshoot; poor scalability.

---

## 2. Star Topology

Every device connects to a **central switch/hub**.

```text
                 +-----+
                 | PC1 |
                 +--+--+
                    |
   +-----+      +---+---+      +-----+
   | PC2 |------|       |------| PC3 |
   +-----+      |Switch |      +-----+
                |(hub)  |
   +-----+      |       |      +-----+
   | PC4 |------|       |------| PC5 |
   +-----+      +---+---+      +-----+
                    |
                 +--+--+
                 | PC6 |
                 +-----+
```

- **Data flow:** every packet goes **device → switch → device**; the switch forwards only to the right port.
- **Pros:** easy to add/remove devices; **one cable failure affects only one device**; easy to troubleshoot; the modern standard for LANs.
- **Cons:** **central device is a single point of failure**; more cable than bus.

---

## 3. Ring Topology

Each device connects to **exactly two neighbours**, forming a closed loop.

```text
              +-----+
        +-----| PC1 |-----+
        |     +-----+     |
     +-----+           +-----+
     | PC4 |           | PC2 |
     +-----+           +-----+
        |     +-----+     |
        +-----| PC3 |-----+
              +-----+

     Data travels in one direction around the ring
```

- **Data flow:** token passing or unidirectional frames travel around the ring.
- **Pros:** no collisions (predictable); equal access; simple traffic flow; dual-ring variants survive one break.
- **Cons:** **a break in a single-ring stops the network**; adding/removing a node interrupts service; slower than star at scale.

---

## 4. Mesh Topology

Every device connects to **every other device** (full mesh) or **many others** (partial mesh).

### Full Mesh

```text
                 +-----+
        +--------| PC1 |--------+
        |        +-----+        |
        |        /     \        |
        |       /       \       |
     +-----+---+         +---+-----+
     | PC2 |                   | PC3 |
     +-----+---+         +---+-----+
        |       \       /       |
        |        \     /        |
        |        +-----+        |
        +--------| PC4 |--------+
                 +-----+
```

### Partial Mesh (real-world WAN)

```text
    Site A ---------- Site B
      |  \             /
      |   \           /
      |    \         /
    Site D ---------- Site C
         \            /
          Site E ----
```

- **Links needed:** full mesh with *n* nodes → **n(n−1)/2** links.
- **Pros:** **highest redundancy** — almost any link can fail; direct paths reduce latency.
- **Cons:** **cabling/ports explode with node count**; expensive; complex to manage.

> **DevOps relevance:** cloud regions, BGP peering, and service meshes are mesh-like designs where any path failing still leaves a route.

---

## 5. Tree (Hierarchical) Topology

Groups of star networks connected in a **hierarchy**, like a family tree.

```text
                     +-------------+
                     | Root Switch |
                     +------+------+
                    /              \
          +--------+----+      +---+--------+
          | Branch Sw 1 |      | Branch Sw 2|
          +-----+---+---+      +---+-----+--+
               /     \             /     \
          +---+--+ +--+---+   +--+--+  +--+--+
          | PC1  | | PC2  |   | PC3 |  | PC4 |
          +------+ +------+   +-----+  +-----+
```

- **Structure:** hierarchical — core → distribution → access.
- **Pros:** scalable, organized, easy to segment; matches real enterprise/campus design.
- **Cons:** **the root/upper level is critical** — its failure isolates an entire branch; more cable; hierarchical faults are harder to isolate.

---

## 6. Hybrid Topology

A **combination** of two or more topologies.

```text
      Star cluster 1            Star cluster 2
        (Building A)              (Building B)

          +-- PC1 --+               +-- PC3 --+
          | Switch1 |---------------| Switch2 |
          +-- PC2 --+   Backbone    +-- PC4 --+
                 \                    /
                  \                  /
                   +---- Router -----+
                            |
                    Ring / Mesh core

   = Star LANs joined by a backbone (very common real design)
```

- **Pros:** flexible — take the best of each; scales to large enterprises; this is what **most real networks are**.
- **Cons:** complex design/troubleshooting; requires skilled administration; cost varies.

---

## 7. Point-to-Point Topology

A **dedicated link between exactly two devices**.

```text
   +--------+                              +--------+
   | Router |=========== Link =============| Router |
   +--------+                              +--------+
             (leased line / serial / VPN)
```

- **Used for:** WAN links, dedicated server-to-storage links, VPN tunnels.
- **Pros:** simple, fast, dedicated bandwidth, easy to secure.
- **Cons:** not scalable beyond two endpoints; one link failure disconnects them entirely.

---

## Topology Comparison

| Topology | Cable Cost | Reliability | Scalability | Ease of Troubleshooting | Typical Use |
|---|---|---|---|---|---|
| **Bus** | Lowest | Low — one break kills all | Poor | Hard | Legacy/very small |
| **Star** | Medium | Medium — central device is SPOF | Good | Easy | Home/office LAN |
| **Ring** | Medium | Low–Medium (dual ring better) | Fair | Medium | Legacy token ring, some metro fiber |
| **Mesh** | Highest | **Highest** | Poor (full) / fair (partial) | Hard | WAN cores, cloud, critical links |
| **Tree** | Medium–High | Medium — upper levels critical | **Good** | Medium | Enterprise/campus |
| **Hybrid** | Varies | Varies | **Best** | Hard | **Most real networks** |
| **Point-to-Point** | Low (per link) | Low for that link | Poor | Easy | WAN links, tunnels |

## Choosing a Topology

```text
Small office, low cost?          -> Star
Critical links, zero downtime?   -> Mesh / partial mesh
Large multi-building network?    -> Tree / Hybrid
Two sites only?                  -> Point-to-point
Legacy coax install?             -> Bus (avoid for new builds)
```

---

# Summary

- A **computer network** is devices connected by a medium, agreeing on **protocols**, to **share data and resources**.
- The **Internet** is the **global network of networks** built on **TCP/IP**, using packet switching and ISP/backbone peering. It is infrastructure, not an application.
- The **WWW** is a **service on top of the Internet** — documents identified by URLs and transferred over HTTP/HTTPS and rendered by browsers. *The web needs the Internet; the Internet does not need the web.*
- **Client-server** is the dominant service model: clients request, servers respond with resources or logic. It gives central control and easy scaling at the cost of a possible single point of failure — mitigated by load balancers and replication. **P2P** is the decentralized alternative.
- Networks are classified by **scope**: **PAN** (metres) → **LAN** (building) → **CAN** (campus) → **MAN** (city) → **WAN** (country/global). **SAN** handles storage; **VPN** creates a logical private network over a public one.
- **Topology** is the arrangement of nodes and links: **bus**, **star**, **ring**, **mesh**, **tree**, **hybrid**, **point-to-point**. Most real deployments are **hybrid**; **star** is the default LAN, **mesh** maximizes redundancy, and **tree** is the enterprise hierarchy.

---

# Quick Revision

| Question | One-line answer |
|---|---|
| What is a network? | Two or more devices connected to share data using shared protocols. |
| What is the Internet? | A global network of networks using TCP/IP. |
| Internet vs WWW? | Internet = roads (infrastructure); WWW = shops along the roads (HTTP/URL/HTML). |
| What does a client-server model do? | Client requests; server holds resource/ logic and responds. |
| Which network type covers a city? | MAN. |
| Which is the largest WAN? | The Internet. |
| Which topology is the standard LAN design? | Star. |
| Which topology is most fault-tolerant? | Mesh (especially full mesh). |
| Which topology has a single cable that breaks everything? | Bus. |
| Which topology is the most common in reality? | Hybrid. |
| Formula for full-mesh links? | n(n−1)/2. |
| What is a single point of failure? | A component whose failure takes down the whole system — e.g. the central switch in star. |
