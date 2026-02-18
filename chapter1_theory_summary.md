# Chapter 1 — Foundation: In-Depth Theory Summary

**Computer Networks: A Systems Approach, 6th Edition**
*Larry L. Peterson & Bruce S. Davie*

> This summary provides a deep, section-by-section walkthrough of the theoretical concepts in Chapter 1. It is intended to build thorough understanding—not just exam prep—of how and why computer networks are designed the way they are.

---

## 1.1 Applications

### 1.1.1 What Distinguishes a Computer Network?

The most important characteristic of a computer network is its **generality**. Unlike single-purpose networks (telephone systems for voice, cable TV for video), computer networks are built from **general-purpose programmable hardware** and can carry many different types of data. Today's computer networks have essentially absorbed the functions that were once performed by those single-use networks.

Understanding a network requires considering multiple stakeholder perspectives:

- **Network designers** — build the hardware and protocols.
- **Application developers** — write software that runs on top of the network.
- **Network operators** — manage and maintain the running network.
- **End users** — interact with network applications.

### 1.1.2 Classes of Applications

The chapter identifies several broad categories of applications, each with different demands on the network:

| Application Class | Characteristics | Examples |
|---|---|---|
| **Web browsing** | Request/reply pattern; a single URL click may trigger dozens of messages (DNS, TCP handshake, HTTP GET, ACKs, teardown) | Fetching a web page |
| **Streaming audio/video** | Continuous consumption; discontinuity (skips, stalls) is unacceptable; one-directional flow | Video on demand, Internet radio |
| **Real-time audio/video** | Tight timing constraints; interactive; bidirectional; delay above ~300 ms RTT becomes unusable | VoIP (Skype), videoconferencing |

**Key insight:** The diversity of applications is what makes network design so challenging. A network must simultaneously support delay-sensitive interactive traffic and bandwidth-hungry bulk transfers.

---

## 1.2 Requirements

This section addresses the fundamental question: *what must a network provide?* The answer is driven by the perspectives of different stakeholders.

### 1.2.1 Stakeholders

Each stakeholder has different priorities:

- **Application programmers** want services like guaranteed delivery, bounded delay, and seamless mobility.
- **Network operators** want easy fault isolation, simple device configuration, and accurate usage accounting.
- **Network designers** want efficient resource utilization, fair allocation, and cost-effective design.

These different requirements form the design space that every network must navigate.

### 1.2.2 Scalable Connectivity

A network must provide **connectivity** among computers, and ideally it should **scale** to support an arbitrarily large number of nodes.

#### Links and Nodes

At the lowest level, a network consists of **nodes** (computers or specialized hardware) connected by **links** (physical media such as copper cable or optical fiber).

- **Point-to-point link** — connects exactly two nodes.
- **Multiple-access link** — shared by more than two nodes (e.g., Wi-Fi, classic Ethernet). These are typically limited in geographic reach and number of nodes, often implementing the "last mile."

#### Switched Networks

Since directly connecting every pair of nodes is impractical, we use **switched networks**, where intermediate forwarding nodes relay data:

- **Circuit-switched networks** — a dedicated circuit is established end-to-end before communication begins (e.g., traditional telephone system).
- **Packet-switched networks** — data is divided into discrete **packets** (or messages) that are individually forwarded through a series of switches. This is the focus of the book.

Packet-switched networks use **store-and-forward**: each switch receives a complete packet, stores it in memory, then forwards it toward the destination. This contrasts with circuit switching, where a dedicated path carries a continuous stream of bits.

#### Internetworks

An **internetwork** (or **internet**, lowercase) is formed when independent networks are interconnected. A node connected to two or more networks is called a **router** or **gateway**, and it forwards messages between networks. The Internet (capital I) is the global TCP/IP internetwork we all use.

**The key recursive insight:** A network can itself be viewed as another kind of network. Internets can be built from internets, allowing arbitrarily large networks to be constructed by interconnecting clouds of smaller networks. This recursive composition was a fundamental innovation of the Internet.

#### Addressing and Routing

For communication, each node needs an **address** — a byte string that uniquely identifies it. When a source wants to send to a destination, it specifies the destination's address. Intermediate switches and routers use this address to **route** the message — the process of systematically forwarding messages toward the destination.

Types of addressing:

| Type | Description |
|------|-------------|
| **Unicast** | Message to a single specific destination |
| **Broadcast** | Message to all nodes on the network |
| **Multicast** | Message to a specific subset of nodes |

#### Network Classification by Size

| Acronym | Name | Typical Span |
|---------|------|-------------|
| **SAN** | Storage/System Area Network | Single room |
| **LAN** | Local Area Network | < 1 km |
| **MAN** | Metropolitan Area Network | Tens of km |
| **WAN** | Wide Area Network | Worldwide |

The size of a network has implications for the underlying technology, particularly because of **propagation delay** — the time it takes data to travel from one end to the other.

### 1.2.3 Cost-Effective Resource Sharing

The central question: *How do multiple hosts share the network simultaneously?*

#### Multiplexing

**Multiplexing** is the sharing of a system resource (here, a physical link) among multiple users. Three primary methods exist:

**1. Frequency-Division Multiplexing (FDM)**
- Each flow gets a dedicated frequency band.
- Simple and natural for constant-bandwidth channels (TV, radio).
- **Limitation:** Unused bands sit idle; the maximum number of flows must be known in advance.

**2. Synchronous Time-Division Multiplexing (STDM)**
- Time is divided into fixed quanta; each flow gets a turn in round-robin fashion.
- Best for uniform, constant-rate channels (telephony).
- **Limitation:** Unused time slots go to waste; fixed number of flows.

**3. Statistical Multiplexing (Packet Switching)**
- The physical link is shared over time, but data are transmitted **on demand**, not in predetermined slots.
- If only one flow has data, it can use the full link — no idle waste.
- Data is divided into **packets** of bounded size to prevent any single flow from monopolizing the link.
- Scheduling decisions (which packet to send next) are made independently at each switch on a per-packet basis (e.g., FIFO, round-robin).

**Why packet switching wins for computer networks:** Computer traffic is inherently **bursty** — long idle periods punctuated by short bursts of activity. Statistical multiplexing efficiently handles this pattern by letting active flows use bandwidth that idle flows would waste under FDM or STDM.

#### Congestion

When a switch receives packets faster than it can forward them, it must **buffer** packets in memory. If this persists, the buffer fills and packets must be **dropped**. This state is called **congestion**.

#### Quality of Service (QoS)

A network that attempts to allocate bandwidth to particular flows — guaranteeing minimum rates or maximum delays — is said to support **Quality of Service (QoS)**.

### 1.2.4 Support for Common Services

A network is not just a packet delivery service — it must support meaningful communication between **application processes** distributed across hosts.

#### Channels

We abstract the network as providing **logical channels** between processes. A channel is like a pipe: a sending application puts data in one end, and the network delivers it to the application at the other end.

Different applications need different channel types:

- **Request/reply channels** — guarantee delivery of every message, ensure only one copy is delivered, and may protect privacy/integrity. Used by file transfer, digital libraries.
- **Message stream channels** — may tolerate some message loss but require in-order delivery. May support one-way or two-way traffic, multicast, and varying delay properties. Used by video streaming, conferencing.

The challenge is to find the right set of common channel abstractions that serve the widest variety of applications.

#### Where to Implement Functionality?

There is a design tension between keeping switches simple (treating the network as a "bit pipe" and pushing all intelligence to the end hosts) versus making switches smarter (providing richer services inside the network). This is a recurring theme in network design.

### 1.2.5 Reliable Message Delivery

Networks are imperfect. There are three classes of failure:

**1. Bit Errors**
- Individual bits are corrupted (1 → 0 or vice versa), often in bursts.
- Caused by electromagnetic interference, power surges, etc.
- Rates: ~1 in 10⁶–10⁷ for copper; ~1 in 10¹²–10¹⁴ for optical fiber.
- Detected using error codes (CRC, checksums). May be corrected or may require retransmission.

**2. Packet Loss**
- An entire packet is lost — either due to an uncorrectable bit error (packet discarded) or because a congested switch dropped it.
- A key difficulty: distinguishing between a truly lost packet and one that is merely delayed.

**3. Node/Link Failure**
- A physical link is cut or a computer crashes (software bug, power failure, misconfiguration).
- Packet-switched networks can often **route around** failures, unlike circuit-switched networks.
- A key difficulty: distinguishing between a failed node and a very slow one.

**The semantic gap:** The difference between what applications expect (reliable, ordered, private communication) and what the underlying technology provides (best-effort, lossy, insecure delivery). Filling this gap is the central challenge of network design.

### 1.2.6 Manageability

Networks must be managed — upgraded, troubleshot, and extended with new features. This is historically a human-intensive activity, but automation is increasingly important.

A fundamental tension exists between **stability** (don't change what works) and **feature velocity** (the rate at which new capabilities are introduced). The explosion of cloud computing has shifted the balance toward faster innovation, making manageability more important than ever.

---

## 1.3 Architecture

A **network architecture** is a blueprint — a set of rules governing the form and content of a protocol graph — that guides the design and implementation of networks.

### 1.3.1 Layering and Protocols

**Abstraction** is the fundamental tool for managing complexity: hide implementation details behind a well-defined interface.

#### Layering

Abstractions in networks are organized into **layers**. Each layer provides a higher level of service, implemented in terms of the services of the layer below:

```
Application programs
       ↕
Process-to-process channels (e.g., reliable delivery)
       ↕
Host-to-host connectivity (hides network topology)
       ↕
Hardware (physical links)
```

**Benefits of layering:**
1. **Decomposition** — breaks the complex problem into manageable pieces.
2. **Modularity** — changes at one layer do not affect other layers.

At any given layer, there may be **multiple alternative abstractions** (e.g., both a request/reply channel and a message stream channel may exist at the same layer).

#### Protocols

The abstract objects that make up the layers are called **protocols**. Each protocol defines two interfaces:

1. **Service interface** — operations available to local objects on the same computer (e.g., "send a message," "fetch a page").
2. **Peer interface** — the form and meaning of messages exchanged with the corresponding protocol on another machine (e.g., the format of an HTTP GET request).

A **protocol specification** defines the abstract interfaces (using prose, pseudocode, state diagrams, packet format diagrams). Concrete **implementations** must adhere to the specification to **interoperate** with each other.

#### Protocol Graphs

The suite of protocols is represented as a **protocol graph**: nodes are protocols, and edges represent a "depends on" relationship. For example:

```
  File App      Video App
       \         /
        RRP    MSP       (process-to-process)
          \   /
           HHP           (host-to-host)
```

Here, RRP (Request/Reply Protocol) and MSP (Message Stream Protocol) both depend on HHP (Host-to-Host Protocol). An application using RRP communicates through the **protocol stack** RRP/HHP.

### 1.3.2 Encapsulation

When a higher-level protocol passes a message to a lower-level protocol, the lower-level protocol attaches its own **header** (and sometimes a **trailer**) containing control information for its peer. The original message becomes the **payload** (or **body**) of the new, larger message.

This process repeats at each layer: HHP encapsulates RRP's message, which already encapsulated the application's data.

At the destination, the process reverses: each layer strips its own header, acts on the control information, and passes the payload up to the next layer. The application at the destination receives exactly the same data the source application sent — all intermediate headers are invisible to it.

**Key principle:** A lower-level protocol does not interpret the payload it receives from a higher-level protocol. It may, however, transform it (e.g., compress or encrypt the entire body, including any higher-level headers).

### 1.3.3 Multiplexing and Demultiplexing

Multiplexing applies not just to physical links but throughout the protocol graph. A single protocol may carry messages from multiple higher-level protocols or applications.

Each protocol includes a **demultiplexing key** (demux key) in its header — an identifier that tells the receiving side which higher-level protocol or application should receive the message.

For example:
- HHP uses a demux key to decide whether to pass a message up to RRP or MSP.
- RRP uses its own demux key to decide which application gets the message.

There is no universal standard for demux keys — different protocols use fields of different sizes (8-bit, 16-bit, 32-bit) and may use one or two fields.

### 1.3.4 The OSI Seven-Layer Model

The ISO's **Open Systems Interconnection (OSI)** model defines seven layers:

| Layer | Name | Function |
|-------|------|----------|
| 7 | **Application** | Application-level protocols (HTTP, FTP, SMTP) |
| 6 | **Presentation** | Data format (integer sizes, byte order, video format) |
| 5 | **Session** | Ties together transport streams for a single application (e.g., audio + video) |
| 4 | **Transport** | Process-to-process channels (messages) |
| 3 | **Network** | Routing among nodes (packets) |
| 2 | **Data Link** | Frames delivered between directly connected nodes (frames) |
| 1 | **Physical** | Raw bit transmission over a link |

- Layers 1–3 are implemented on **all** network nodes (hosts, switches, routers).
- Layers 4–7 typically run only on **end hosts**.
- No OSI-based network runs today, but the **terminology** is universally used.

### 1.3.5 The Internet Architecture

The Internet architecture (also called the **TCP/IP architecture**) evolved from the ARPANET and is simpler than OSI:

```
                  Applications
                (HTTP, FTP, SMTP, ...)
                  /          \
               TCP            UDP          ← Transport
                  \          /
                     IP                    ← Network (Internetworking)
               /     |      \
           NET₁    NET₂    NET₃           ← Link/Subnet (Ethernet, Wi-Fi, ...)
```

#### Key Layers

| Layer | Protocol(s) | Role |
|-------|-------------|------|
| **Application** | HTTP, FTP, SMTP, DNS | End-user services and protocols |
| **Transport** | TCP, UDP | TCP: reliable byte stream; UDP: unreliable datagram delivery |
| **Internet** | IP | Interconnects diverse networks into a single logical internetwork |
| **Link/Subnet** | Ethernet, Wi-Fi, SONET, etc. | Physical link technologies |

#### Three Defining Features of the Internet Architecture

**1. No strict layering.**
Applications can bypass transport protocols and use IP directly, or even access underlying network technologies directly. Programmers are free to define new abstractions at any level.

**2. The hourglass design.**
The protocol graph is wide at the top (many applications), narrow in the middle (one protocol — IP), and wide at the bottom (many link technologies). IP is the **focal point**:

- **Above IP:** Arbitrarily many transport protocols, each offering different channel abstractions.
- **Below IP:** Arbitrarily many network technologies.
- IP provides a **minimal, carefully chosen set of global capabilities** that allows everything above and below it to coexist, share capabilities, and evolve rapidly.

This narrow waist is critical to the Internet's adaptability. It completely separates the problem of host-to-host delivery from the problem of process-to-process communication.

**3. Rough consensus and running code.**
The IETF requires both a protocol specification **and** working implementations before a protocol can be adopted as a standard. This culture ensures that protocols are practically implementable.

> *"We reject kings, presidents, and voting. We believe in rough consensus and running code."* — David Clark

#### Standards Bodies

| Organization | Role | Example Standards |
|---|---|---|
| **IETF** | Primary Internet standards body | TCP, UDP, IP, DNS, BGP |
| **IEEE** | Ethernet and wireless standards | 802.3 (Ethernet), 802.11 (Wi-Fi) |
| **W3C** | Web standards | HTTP, HTML |
| **3GPP** | Cellular standards | 4G LTE, 5G |
| **ITU-T** | Telecommunication standards | H.264, SONET |
| **IANA/ICANN** | Identifier assignment and Internet stewardship | IP address allocation, DNS root |

---

## 1.4 Software

The Internet's success is largely due to the fact that most of its functionality is implemented in **software** running on general-purpose computers. New functionality can be added with "just a small matter of programming," enabling rapid innovation.

### 1.4.1 Application Programming Interface (Sockets)

The **network API** is the interface the operating system provides to its networking subsystem. The most widely used network API is the **socket interface**, originating from Berkeley Unix and now supported on virtually all operating systems.

#### The Socket Abstraction

A **socket** is the point where a local application process attaches to the network.

Key operations (for TCP):

| Operation | Purpose |
|-----------|---------|
| `socket(domain, type, protocol)` | Create a new socket. Returns a handle (file descriptor). |
| `bind(socket, address, addr_len)` | Associate the socket with a local address (IP + port). |
| `listen(socket, backlog)` | Mark the socket as willing to accept connections (server). |
| `accept(socket, address, addr_len)` | Block until a remote host connects; return a new socket for the connection (server). |
| `connect(socket, address, addr_len)` | Establish a connection to a remote host (client). |
| `send(socket, message, msg_len, flags)` | Send data over the connection. |
| `recv(socket, buffer, buf_len, flags)` | Receive data from the connection. |

#### Client/Server Model

- **Server:** Performs a **passive open** — calls `bind`, `listen`, `accept`. It waits for incoming connections on a **well-known port** (e.g., port 80 for HTTP).
- **Client:** Performs an **active open** — calls `connect` with the server's address. The OS assigns an unused local port automatically.

#### Significance

The socket API defines the **demarcation point** between applications and the network implementation. By providing a stable, well-defined interface, it enabled the explosion of Internet applications — from the humble beginnings of email and FTP to today's cloud-based smartphone applications.

### 1.4.2 Example Application

The textbook provides a simple client/server "talk" program demonstrating:

- **Client:** Resolves the server hostname to an IP address, builds an address structure, creates a socket, connects, then enters a loop reading from stdin and sending over the socket.
- **Server:** Builds an address structure for its own port, creates a socket, binds to the local address, listens, then enters a loop accepting connections and printing received text.

This example illustrates the fundamental pattern underlying all networked applications: two processes, a connection, and the exchange of messages.

---

## 1.5 Performance

Network performance is characterized by two fundamental metrics: **bandwidth** and **latency**.

### 1.5.1 Bandwidth and Latency

#### Bandwidth (Throughput)

- **Bandwidth** (or data rate) = the number of bits per second (bps) that can be transmitted on a link.
- Strictly, **bandwidth** refers to the width of a frequency band (measured in Hz), while **data rate** or **throughput** refers to bits per second.
- **Throughput** typically means the *measured, actual* performance, which may be less than the link's rated bandwidth due to overhead, contention, protocol inefficiency, etc.

At the physical level, each bit occupies a certain width of time:
- On a 1-Mbps link, each bit is 1 µs wide.
- On a 2-Mbps link, each bit is 0.5 µs wide.

#### Latency (Delay)

**Latency** is how long it takes a message to travel from one end of a network to the other. It has three components:

```
Latency = Propagation + Transmit + Queue
```

| Component | Formula | Depends on |
|-----------|---------|------------|
| **Propagation delay** | Distance / SpeedOfLight | Link length, medium (3.0×10⁸ m/s in vacuum; 2.3×10⁸ in copper; 2.0×10⁸ in fiber) |
| **Transmit delay** (serialization) | Size / Bandwidth | Packet size, link speed |
| **Queuing delay** | Varies | Network load, switch buffer occupancy |

#### Round-Trip Time (RTT)

RTT is the time for a message to travel from sender to receiver **and back**. It is critical because the sender cannot get feedback faster than one RTT.

Useful approximations:
- Cross-country (US) RTT ≈ **100 ms**
- LAN RTT ≈ **1 ms**

#### When Latency vs. Bandwidth Dominates

| Scenario | Dominant Factor | Why |
|----------|----------------|-----|
| Small message (1 byte) | **Latency** | Transmit time is negligible; total time ≈ RTT |
| Large file (25 MB) over slow link | **Bandwidth** | Transmit time dwarfs propagation delay |
| Large file over fast link | **Latency** may dominate | Even 1 RTT can be significant when the file fits in a single RTT's bandwidth |

#### Units and Conversions

A common source of confusion in networking:

| Context | "Mega" means | "Giga" means |
|---------|-------------|-------------|
| Bandwidth (Mbps, Gbps) | 10⁶ (SI) | 10⁹ (SI) |
| File/memory sizes (MB, GB) | 2²⁰ = 1,048,576 | 2³⁰ |

This means "send a 64-kB message over a 100-Mbps link" implies 64 × 2¹⁰ × 8 bits transmitted at 100 × 10⁶ bits/sec.

### 1.5.2 Delay × Bandwidth Product

The **delay × bandwidth product** represents the maximum amount of data that can be "in flight" (in the pipe) at any instant:

```
Delay × Bandwidth = one-way latency × link bandwidth
```

Think of a link as a hollow pipe:
- **Latency** = length of the pipe.
- **Bandwidth** = diameter of the pipe.
- **Delay × Bandwidth** = volume of the pipe (in bits).

**Example:** A transcontinental link with 50-ms one-way latency and 45-Mbps bandwidth holds:
```
50 × 10⁻³ × 45 × 10⁶ = 2.25 × 10⁶ bits ≈ 280 kB
```

**Why it matters:**
- This is how much data the sender must transmit before the first bit arrives at the receiver.
- For RTT-based protocols, the sender can transmit **RTT × bandwidth** worth of data before receiving any feedback.
- If the sender does not fill the pipe, it is underutilizing the network.
- If the receiver tells the sender to stop, up to one **RTT × bandwidth** worth of data may already be in flight.

| Link Type | Bandwidth | One-Way Distance | RTT | RTT × Bandwidth |
|-----------|-----------|-----------------|-----|----------------|
| Wireless LAN | 54 Mbps | 50 m | 0.33 µs | 18 bits |
| Satellite | 1 Gbps | 35,000 km | 230 ms | 230 Mb |
| Cross-country fiber | 10 Gbps | 4,000 km | 40 ms | 400 Mb |

### 1.5.3 High-Speed Networks

As bandwidth increases, **latency stays constant** — the speed of light does not change. This has profound implications:

**Example — transmitting a 1-MB file:**
- Over a **1-Mbps** link with 100-ms RTT: takes 80 RTTs. The file looks like a **stream**.
- Over a **1-Gbps** link with 100-ms RTT: the file does not even fill one RTT's worth of pipe (delay × bandwidth = 12.5 MB). The file looks like a **single packet**.

At high speeds, a single RTT becomes a significant amount of time relative to the transfer. The difference between 1 RTT and 2 RTTs is a 100% increase, while the difference between 100 RTTs and 101 RTTs is only 1%.

**Effective throughput:**

```
Throughput = TransferSize / TransferTime
TransferTime = RTT + TransferSize / Bandwidth
```

For a 1-MB file on a 1-Gbps link with 100-ms RTT:
```
TransferTime = 100 ms + 8 ms = 108 ms
Effective Throughput = 1 MB / 108 ms = 74.1 Mbps (not 1 Gbps!)
```

**Conclusion:** On high-speed networks, **latency, not bandwidth**, dominates performance. This shifts the design focus toward minimizing RTTs and avoiding retransmissions.

### 1.5.4 Application Performance Needs

Not all applications simply want "as much bandwidth as possible."

#### Bandwidth Requirements

Some applications have a **fixed upper bound** on required bandwidth. For example, a quarter-screen video (352×240 pixels, 24-bit color, 30 fps) needs:
```
352 × 240 × 24 / 8 × 30 = 75 Mbps (uncompressed)
```

With compression, the average may be 2 Mbps, but the **instantaneous rate varies** — some intervals may be higher, some lower. Knowing only the average is insufficient; the network must accommodate **bursts** (characterized by a peak rate sustained for a period of time).

#### Jitter

**Jitter** = variation in latency from packet to packet.

For real-time applications (video, voice), packets are generated at regular intervals (e.g., every 33 ms for 30-fps video). If the network introduces variable delay, packets arrive with uneven spacing:

- Packets that arrive **early** can be buffered until needed.
- Packets that arrive **late** cause quality degradation (choppy video, garbled audio).

**Solution:** If the receiver knows the upper and lower bounds on latency, it can delay playback by enough time to absorb the jitter, using a **playout buffer**. This does not eliminate jitter — it masks it by introducing a constant additional delay.

---

## Perspective: Feature Velocity

The chapter concludes with a discussion of how the networking industry is undergoing a major transformation:

- **Traditional model:** Design a network once, operate it for years. Building and operating are separate activities.
- **Cloud-inspired model:** Continuous design evolution, DevOps-style development and operations, "softwarization" of the network.

**Software-Defined Networks (SDNs)** represent this new approach: moving intelligence into software, using commodity hardware, and applying agile engineering practices to the network internals (not just the applications running on top).

**Two themes of the transformation:**
1. Take advantage of commodity hardware and move all intelligence into software.
2. Adopt agile engineering processes that break down barriers between development and operations.

This "cloudification" is a recurring theme throughout the book and reflects how deeply embedded the Internet is in modern life.

---

## Key Takeaways

1. **Computer networks are general-purpose** — unlike single-use networks, they support an ever-growing range of applications.
2. **Scalability** comes from recursive composition — networks of networks, all using a common addressing and routing scheme.
3. **Statistical multiplexing (packet switching)** is the most efficient way to share links for bursty computer traffic.
4. **Layering and protocols** manage complexity by decomposing the problem and hiding implementation details.
5. **The hourglass architecture** (narrow waist of IP) is the Internet's most important design principle — it separates host-to-host connectivity from application-level services.
6. **Encapsulation** allows each layer to add its own control information without the layers above or below needing to understand it.
7. **Bandwidth and latency** are the two fundamental performance metrics, and their relative importance depends on the application.
8. **The delay × bandwidth product** determines how much data can be in flight and is essential for designing protocols that fully utilize the network.
9. **On high-speed networks, latency dominates** — minimizing round trips matters more than maximizing bandwidth.
10. **Network design is an ongoing process** — the industry is shifting from static designs to continuous, software-driven evolution.
