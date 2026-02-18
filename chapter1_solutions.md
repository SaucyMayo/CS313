# Chapter 1 — Exercise Solutions (6th Edition)

**Computer Networks: A Systems Approach, 6th Edition**
*Larry L. Peterson & Bruce S. Davie*

---

## Exercise 1

> Use anonymous FTP to connect to `ftp.rfc-editor.org` (directory `in-notes`), and retrieve the RFC index. Also retrieve the protocol specifications for TCP, IP, and UDP.

**Answer (Practical Exercise):**

Use a command-line FTP client or browser. The key RFCs are:

| Protocol | RFC Number |
|----------|-----------|
| IP       | RFC 791   |
| TCP      | RFC 793   |
| UDP      | RFC 768   |

Example commands:

```bash
ftp ftp.rfc-editor.org
cd in-notes
get rfc-index.txt
get rfc791.txt    # IP
get rfc793.txt    # TCP
get rfc768.txt    # UDP
```

---

## Exercise 2

> Use a Web search tool to locate useful, general, and noncommercial information about the following topics: MBone, ATM, MPEG, IPv6, and Ethernet.

**Answer (Research Exercise):**

This is a research-based exercise. Suggested starting points:

- **MBone**: The Multicast Backbone — an experimental multicast network layered on the Internet.
- **ATM (Asynchronous Transfer Mode)**: A cell-based switching technique using fixed-size 53-byte cells.
- **MPEG**: Standards for audio/video compression (MPEG-1, MPEG-2, MPEG-4, H.264/265).
- **IPv6**: The next-generation Internet Protocol with 128-bit addresses (RFC 8200).
- **Ethernet**: The dominant LAN technology (IEEE 802.3), from 10 Mbps to 100 Gbps+.

Good sources: Wikipedia, IETF RFCs, IEEE Xplore, textbook companion sites.

---

## Exercise 3

> The Unix utility `whois` can be used to find the domain name corresponding to an organization or vice versa. Experiment with it.

**Answer (Practical Exercise):**

```bash
whois princeton.edu      # Look up domain details
whois princeton          # Search by organization name
```

The `whois` tool queries domain registration databases and returns information such as the registrant, name servers, registration/expiration dates, and contact details. You can also use the web interface at [whois.icann.org](https://whois.icann.org).

---

## Exercise 4

> Calculate the total time required to transfer a 1000-kB file in the following cases, assuming an RTT of 100 ms, a packet size of 1 kB data, and an initial 2 × RTT of "handshaking" before data are sent.

**Given:**
- File size: 1000 kB = 1000 × 1024 × 8 = 8,192,000 bits
- RTT: 100 ms
- Packet size: 1 kB = 8192 bits
- Number of packets: 1000 kB / 1 kB = 1000 packets
- Handshake: 2 × RTT = 200 ms

> **How to approach these problems:** Break the total time into additive components: *handshake + transmission + propagation + any per-packet waiting*. Draw a timeline if it helps.

### (a) Bandwidth = 1.5 Mbps, continuous sending

The sender pushes all data onto the link without waiting for acknowledgments.

| Component | Calculation | Value |
|-----------|------------|-------|
| Handshake | 2 × 100 ms | 200 ms |
| Transmit time | 8,192,000 bits ÷ 1,500,000 bps | 5461 ms |
| Propagation (last bit) | RTT / 2 | 50 ms |

**Total ≈ 200 + 5461 + 50 = 5711 ms ≈ 5.71 s**

> *Why RTT/2 for propagation?* After the last bit is placed on the wire, it still needs to travel from sender to receiver. That one-way trip is half the round-trip time.

### (b) Bandwidth = 1.5 Mbps, wait one RTT after each packet

After sending each packet, the sender waits for an acknowledgment (one full RTT) before sending the next.

| Component | Calculation | Value |
|-----------|------------|-------|
| Handshake | 2 × RTT | 200 ms |
| Per-packet time | transmit + RTT = 8192/1.5M + 100 ms | ≈ 105.46 ms |
| Packets 1–999 | 999 × 105.46 ms | 105,355 ms |
| Last packet | transmit + RTT/2 = 5.46 + 50 ms | 55.46 ms |

**Total ≈ 200 + 105,355 + 55.46 ≈ 105,611 ms ≈ 105.6 s**

> *Key insight:* Stop-and-wait is extremely slow because the link sits idle during each RTT wait.

### (c) Infinite bandwidth, 20 packets per RTT

Transmit time is zero, but only 20 packets can be sent per RTT (window size = 20).

| Component | Calculation | Value |
|-----------|------------|-------|
| Handshake | 2 × RTT | 200 ms |
| Rounds needed | ⌈1000 / 20⌉ = 50 | — |
| First 49 rounds | 49 × RTT | 4900 ms |
| Last round propagation | RTT / 2 | 50 ms |

**Total = 200 + 4900 + 50 = 5150 ms = 5.15 s**

> *Explanation:* In each RTT window, the sender transmits 20 packets (instantly, since bandwidth is infinite). It then waits one full RTT for ACKs before sending the next batch. The last batch only needs to propagate (half RTT).

### (d) Infinite bandwidth, exponential window growth (1, 2, 4, 8, …)

During RTT *n* (starting from 0), the sender sends 2ⁿ packets.

| RTT # | Packets sent this RTT | Cumulative total |
|-------|----------------------|-----------------|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 4 | 7 |
| 3 | 8 | 15 |
| 4 | 16 | 31 |
| 5 | 32 | 63 |
| 6 | 64 | 127 |
| 7 | 128 | 255 |
| 8 | 256 | 511 |
| 9 | 512 | 1023 ≥ 1000 ✓ |

After **n = 9** sending rounds, cumulative = 2¹⁰ − 1 = 1023 ≥ 1000 packets. The time from the start of round 0 to the start of round 9 is 9 RTTs, then the last batch propagates in RTT/2.

**Total = 2 RTT (handshake) + 9 RTT + 0.5 RTT = 11.5 × 100 = 1150 ms = 1.15 s**

> *Key insight:* Exponential growth (slow start) reaches full utilization surprisingly quickly.

---

## Exercise 5

> Calculate the total time required to transfer a 1.5-MB file, assuming an RTT of 80 ms, a packet size of 1 kB data, and an initial 2 × RTT of "handshaking."

**Given:**
- File size: 1.5 MB = 1.5 × 1024 × 1024 × 8 = 12,582,912 bits
- RTT: 80 ms
- Packet size: 1 kB = 8192 bits
- Number of packets: 1.5 × 1024 = 1536 packets
- Handshake: 2 × 80 = 160 ms

### (a) Bandwidth = 10 Mbps, continuous sending

| Component | Calculation | Value |
|-----------|------------|-------|
| Handshake | 2 × 80 ms | 160 ms |
| Transmit time | 12,582,912 / 10,000,000 | 1258.3 ms |
| Propagation | RTT / 2 | 40 ms |

**Total ≈ 160 + 1258.3 + 40 = 1458.3 ms ≈ 1.46 s**

### (b) Bandwidth = 10 Mbps, wait one RTT after each packet

Per-packet transmit time: 8192 / 10,000,000 = 0.819 ms

| Component | Calculation | Value |
|-----------|------------|-------|
| Handshake | 160 ms | 160 ms |
| Packets 1–1535 | 1535 × (0.819 + 80) | 124,057 ms |
| Last packet | 0.819 + 40 | 40.8 ms |

**Total ≈ 160 + 124,057 + 40.8 ≈ 124,258 ms ≈ 124.3 s**

### (c) Infinite bandwidth, 20 packets per RTT

| Component | Calculation | Value |
|-----------|------------|-------|
| Handshake | 160 ms | 160 ms |
| Rounds needed | ⌈1536 / 20⌉ = 77 | — |
| First 76 rounds | 76 × 80 ms | 6080 ms |
| Last round propagation | RTT / 2 = 40 ms | 40 ms |

**Total = 160 + 6080 + 40 = 6280 ms = 6.28 s**

### (d) Infinite bandwidth, exponential growth

Cumulative after round n: 2^(n+1) − 1

| RTT # | Cumulative |
|-------|-----------|
| 9 | 1023 |
| 10 | 2047 ≥ 1536 ✓ |

Need 11 rounds (0 through 10).

**Total = 2 RTT + 10 RTT + 0.5 RTT = 12.5 × 80 = 1000 ms = 1.0 s**

---

## Exercise 6

> Consider a point-to-point link **2 km** in length. At what bandwidth would propagation delay (speed 2 × 10⁸ m/s) equal transmit delay for 100-byte packets? What about 512-byte packets?

**Method:** Set propagation delay = transmit delay and solve for bandwidth.

Propagation delay = distance / speed = 2000 / (2 × 10⁸) = **10 µs**

**For 100-byte packets:**

transmit delay = packet size / bandwidth = propagation delay

bandwidth = (100 × 8) bits / 10 × 10⁻⁶ s = 800 / 10⁻⁵ = **80 Mbps**

**For 512-byte packets:**

bandwidth = (512 × 8) / 10⁻⁵ = 4096 / 10⁻⁵ = **409.6 Mbps**

> *Interpretation:* Below these bandwidths, transmit delay dominates. Above them, propagation delay dominates. Shorter links make propagation delay less significant, so the crossover bandwidth is higher.

---

## Exercise 7

> Consider a point-to-point link **50 km** in length. At what bandwidth would propagation delay equal transmit delay for 100-byte packets? What about 512-byte packets?

Propagation delay = 50,000 / (2 × 10⁸) = **250 µs**

**For 100-byte packets:**

bandwidth = 800 bits / 250 × 10⁻⁶ s = **3.2 Mbps**

**For 512-byte packets:**

bandwidth = 4096 / 250 × 10⁻⁶ = **16.384 Mbps**

> *Comparison with Exercise 6:* The longer the link, the lower the bandwidth at which propagation delay equals transmit delay. Long links are more "propagation dominated."

---

## Exercise 8

> What properties of postal addresses would be likely to be shared by a network addressing scheme? What differences? What properties of telephone numbering?

**Answer:**

**Shared properties with postal addresses:**
- **Hierarchical structure**: Postal addresses use country → province/state → city → street → number. Network addresses similarly use hierarchical structures (e.g., network → subnet → host) to support efficient routing.
- **Uniqueness**: Each address uniquely identifies a destination.
- **Routing information embedded**: Addresses contain embedded information that guides delivery (postal workers route by region, routers route by network prefix).

**Differences from postal addresses:**
- Postal addresses are **variable-length** and contain **redundant information** (e.g., city name is redundant given the postal code), making them tolerant of minor errors.
- Network addresses are typically **fixed-length** and **compact**, with no built-in redundancy.
- Postal addresses are **human-readable**; network addresses are **numeric**.

**Telephone number properties shared with network addresses:**
- **Hierarchical** (country code → area code → subscriber number, similar to geographic/organizational hierarchy).
- **Fixed-length** (within a country).
- **Administratively assigned** (unlike, say, Ethernet MAC addresses which are factory-assigned).
- Roughly **one-to-one correspondence** with nodes (endpoints).

---

## Exercise 9

> One property of addresses is uniqueness. What other properties might be useful? Can you think of situations where addresses might not be unique?

**Answer:**

**Useful properties of addresses:**

1. **Hierarchical structure** — enables aggregation and efficient routing (routers only need to know about network prefixes, not every individual host).
2. **Location-dependent** (locator property) — the address hints at where the node is, aiding routing decisions.
3. **Fixed length** — simplifies hardware processing and header design.
4. **Administratively assigned** — allows planned allocation and management.

**Situations where addresses may not be unique:**

- **Toll-free phone numbers**: Calling a large retailer may connect you to any one of dozens of phones — all sharing the same "address."
- **Anycast addressing**: In networks, an anycast address is shared by multiple servers; the network routes to the nearest one. Used in DNS root servers and CDNs.
- **Private/NAT addresses**: Many organizations reuse the same private IP ranges (e.g., 192.168.x.x) internally. These are not globally unique.
- **Within a single organization**: If global reachability is not required, non-unique addresses may be acceptable.

---

## Exercise 10

> Give an example of a situation in which multicast addresses might be beneficial.

**Answer:**

**Video/audio conferencing** among multiple geographically dispersed sites is an excellent use case. Without multicast:
- **Unicast** would require a separate copy of the stream from the sender to every receiver, wasting bandwidth.
- **Broadcast** would send traffic to *all* hosts on the network, including those not interested.

**Multicast** sends one copy that is replicated only where the delivery paths diverge, reaching exactly the interested receivers.

Other examples:
- **Live streaming** (e.g., IPTV) — delivering a channel only to subscribed households.
- **Software updates** — distributing the same patch to thousands of machines simultaneously.
- **Stock ticker feeds** — real-time data pushed to subscribing trading terminals.

---

## Exercise 11

> What differences in traffic patterns account for FDM being cost-effective for TV/radio, STDM for voice telephony, yet neither for general-purpose computer networks?

**Answer:**

**STDM (Synchronous Time-Division Multiplexing):**
- Works well for **voice** because voice calls have roughly **constant, uniform bandwidth** needs (64 kbps per call).
- Each timeslot is pre-allocated; unused slots are **wasted**, not available to others.
- Preferred for voice because all channels need the same capacity, and the technology is simple.

**FDM (Frequency-Division Multiplexing):**
- Works well for **TV/radio** because each channel needs sustained bandwidth and receivers only tune to one frequency — the hardware is extremely simple.
- Also supports different channel sizes (e.g., AM vs FM bandwidth).

**Why neither works for computer networks:**
1. **Bursty traffic**: Computer communications alternate between bursts of data and long idle periods. Pre-allocated slots/frequencies waste bandwidth during idle times.
2. **Dynamic connections**: Computer connections are created and torn down rapidly. STDM/FDM require advance allocation of channels, which is too rigid.
3. **Variable bandwidth needs**: Different applications need vastly different bandwidths (a web request vs. a file transfer). Fixed allocation cannot adapt.

**Statistical multiplexing** (packet switching) solves these problems by sharing bandwidth on demand.

---

## Exercise 12

> How "wide" is a bit on a 1-Gbps link? How long is a bit in copper wire, where the speed of propagation is 2.3 × 10⁸ m/s?

**Answer:**

**Bit width in time:**

1 Gbps = 10⁹ bps → each bit occupies **1 / 10⁹ = 1 ns** (one nanosecond)

**Bit length in copper wire:**

length = time × speed = 1 × 10⁻⁹ s × 2.3 × 10⁸ m/s = **0.23 m = 23 cm**

> *Physical intuition:* At 1 Gbps, a single bit stretches 23 cm along the wire. If you could freeze time and look at the wire, you'd see bits spaced 23 cm apart. At 10 Gbps, each bit would be only 2.3 cm long.

---

## Exercise 13

> How long does it take to transmit *x* kB over a *y*-Mbps link? Give your answer as a ratio of *x* and *y*.

**Answer:**

Transmit time = data size / bandwidth

= (x × 1024 × 8 bits) / (y × 10⁶ bps)

= **8192x / (y × 10⁶) seconds = 8.192x / y milliseconds**

> *Example:* 100 kB over 10 Mbps = 8.192 × 100 / 10 = 81.92 ms.

---

## Exercise 14

> Suppose a 100-Mbps point-to-point link is being set up between Earth and a new lunar colony. Distance ≈ 385,000 km, speed of light = 3 × 10⁸ m/s.

### (a) Minimum RTT

RTT = 2 × distance / speed = 2 × 385,000,000 / (3 × 10⁸)

**RTT ≈ 2.567 seconds**

### (b) Delay × bandwidth product

delay × bandwidth = 2.567 s × 100 Mbps = 256.7 Mbit ≈ **32.1 MB**

### (c) Significance

The delay × bandwidth product represents the **maximum amount of data that can be "in flight"** (in the pipe between sender and receiver) at any instant. It is the volume of data the sender can transmit before it is possible to receive any response. For reliable protocols, this determines the **minimum window size** needed to keep the link fully utilized.

### (d) Time to download a 25-MB image

The request travels Earth → Moon (RTT/2), then the image is sent Moon → Earth.

| Component | Calculation | Value |
|-----------|------------|-------|
| Request propagation | RTT / 2 | 1.283 s |
| First image bit to Earth | RTT / 2 | 1.283 s |
| Image transmission | 25 × 8 Mbit / 100 Mbps | 2.0 s |

The image starts arriving at T = RTT = 2.567 s. The last bit leaves the Moon at T = RTT/2 + 2.0 = 3.283 s and arrives at T = RTT/2 + 2.0 + RTT/2 = RTT + 2.0.

**Total = RTT + transmission time = 2.567 + 2.0 ≈ 4.57 s**

---

## Exercise 15

> Suppose a 128-kbps point-to-point link is set up between Earth and a rover on Mars. Distance ≈ 55 Gm (when closest), speed of light = 3 × 10⁸ m/s.

### (a) Minimum RTT

One-way delay = 55 × 10⁹ / (3 × 10⁸) = 183.33 s

**RTT = 2 × 183.33 ≈ 366.67 s ≈ 6.11 minutes**

### (b) Delay × bandwidth product

Using the full RTT:

delay × bandwidth = 366.67 s × 128,000 bps = **46,933,333 bits ≈ 5.58 MB**

> *This means ~5.58 MB of data can be in transit between Earth and Mars at any time.* The one-way delay × bandwidth is ~2.79 MB, representing the data that fills the one-way "pipe."

### (c) How quickly can a 5-Mbit image reach Earth?

| Component | Calculation | Value |
|-----------|------------|-------|
| Transmission time | 5,000,000 / 128,000 | 39.06 s |
| One-way propagation | 55 × 10⁹ / (3 × 10⁸) | 183.33 s |

The rover starts transmitting immediately after taking the picture. The last bit leaves the rover at T = 39.06 s and arrives at Earth at T = 39.06 + 183.33.

**Total ≈ 222.4 s ≈ 3 minutes 42 seconds**

---

## Exercise 16

> For each of the following operations on a remote file server, are they delay-sensitive or bandwidth-sensitive?

**Answer:**

| Operation | Sensitivity | Explanation |
|-----------|------------|-------------|
| **(a) Open a file** | **Delay-sensitive** | Very little data is exchanged (just a request/response). Performance depends on round-trip time. |
| **(b) Read file contents** | **Bandwidth-sensitive** | Large amounts of data must be transferred, especially for large files. |
| **(c) List directory contents** | **Delay-sensitive** | Directory listings are typically small; the wait for the response dominates. |
| **(d) Display file attributes** | **Delay-sensitive** | File metadata (size, date, permissions) is very small — much smaller than the file itself. |

> *Rule of thumb:* If the data transferred is small, performance is **delay-sensitive** (dominated by RTT). If the data is large, performance is **bandwidth-sensitive** (dominated by link speed).

---

## Exercise 17

> Calculate the latency (first bit sent to last bit received) for 10-Mbps Ethernet, packet size 5000 bits, propagation delay 10 µs per link.

**Key formulas:**
- Transmit delay per link: packet size / bandwidth = 5000 / 10⁷ = **500 µs**
- Propagation delay per link: **10 µs**

### (a) One store-and-forward switch

With store-and-forward, the switch must receive the entire packet before retransmitting.

```
Sender ──Link 1──> Switch ──Link 2──> Receiver
       500+10 µs          500+10 µs
```

Latency = 2 × (transmit + propagation) = 2 × (500 + 10) = **1020 µs**

### (b) Three store-and-forward switches (four links)

Latency = 4 × (500 + 10) = **2040 µs**

### (c) One cut-through switch (retransmits after first 200 bits)

With cut-through, the switch begins retransmitting after receiving 200 bits, without waiting for the full packet.

The **last bit** is the bottleneck. Trace its path:
- It leaves the sender at T = 500 µs (transmit time for full 5000-bit packet)
- It propagates through link 1: arrives at switch at T = 500 + 10 = 510 µs
- The switch adds a 200-bit delay: 200 / 10 Mbps = 20 µs → exits switch at T = 530 µs
- It propagates through link 2: arrives at receiver at T = 530 + 10 = **540 µs**

Alternatively: Latency = transmit time + cut-through delay + 2 × propagation = 500 + 20 + 20 = **540 µs**

> *Why is cut-through faster?* The first bit starts propagating on link 2 while the sender is still transmitting the rest of the packet on link 1. We save almost one full transmission time (500 µs → 20 µs switch delay).

---

## Exercise 18

> Calculate the latency for 1-Gbps Ethernet, 5000 bits, 10 µs propagation per link.

**Transmit delay per link:** 5000 / 10⁹ = **5 µs**

### (a) One store-and-forward switch

Latency = 2 × (5 + 10) = **30 µs**

### (b) Three store-and-forward switches (four links)

Latency = 4 × (5 + 10) = **60 µs**

### (c) Three cut-through switches (retransmit after 128 bits)

Cut-through delay per switch: 128 / 10⁹ = 0.128 µs

Latency = transmit time + 3 × cut-through delay + 4 × propagation delay

= 5 + 3 × 0.128 + 4 × 10 = 5 + 0.384 + 40 = **45.4 µs**

> *Note:* At 1 Gbps, the transmit time (5 µs) is much smaller than the propagation delay (10 µs per link). The link is now **propagation-dominated**, and the benefit of cut-through switching is smaller relative to the total latency.

---

## Exercise 19

> Calculate the effective bandwidth for the following cases.

### (a) 10-Mbps Ethernet through three store-and-forward switches (as in Exercise 17(b)). Switches can send on one link while receiving on the other.

Because the switches can pipeline (send and receive simultaneously), once the first packet has traversed the full path, subsequent packets arrive every **500 µs** (one transmit time per link — the bottleneck is the slowest link, which is 10 Mbps).

**Effective bandwidth = 10 Mbps** (in steady state, the pipeline keeps the link fully utilized).

### (b) Same as (a) but sender waits for a 50-byte ACK after each 5000-bit data packet

This is stop-and-wait: send one packet, wait for ACK, then send next.

| Component | Calculation | Value |
|-----------|------------|-------|
| Data packet latency (from 17b) | 4 × (500 + 10) | 2040 µs |
| ACK (400 bits) transmit per link | 400 / 10M | 40 µs |
| ACK latency (4 links back) | 4 × (40 + 10) | 200 µs |
| Total RTT | 2040 + 200 | 2240 µs |

**Effective bandwidth = 5000 bits / 2240 µs ≈ 2.23 Mbps**

> Only ~22% of the 10-Mbps link capacity is utilized due to the stop-and-wait overhead.

### (c) Overnight (12-hour) shipment of 100 compact disks (650 MB each)

Total data = 100 × 650 MB = 65,000 MB = 65 GB

Time = 12 hours = 43,200 seconds

**Effective bandwidth = 65,000 × 10⁶ × 8 bits / 43,200 s ≈ 12.04 Mbps**

Or equivalently: ≈ 1.50 MB/s.

---

## Exercise 20

> Calculate the bandwidth × delay product for the following links. Use one-way delay, measured from first bit sent to first bit received.

### (a) 10-Mbps Ethernet, delay = 10 µs

BDP = 10 × 10⁶ × 10 × 10⁻⁶ = **100 bits = 12.5 bytes**

### (b) 10-Mbps Ethernet with one store-and-forward switch (as in 17(a)), 5000-bit packets, 10 µs per link

With store-and-forward, the first bit is delayed until the entire packet has been received at the switch, then retransmitted.

First-bit one-way delay:
- Packet transmitted on link 1: 500 µs
- Propagation on link 1: 10 µs → packet fully at switch at T = 510 µs
- Switch retransmits first bit at T = 510 µs
- First bit arrives at destination at T = 510 + 10 = **520 µs**

BDP = 10 × 10⁶ × 520 × 10⁻⁶ = **5200 bits = 650 bytes**

### (c) 1.5-Mbps T1 link, transcontinental one-way delay = 50 ms

BDP = 1.5 × 10⁶ × 50 × 10⁻³ = **75,000 bits = 9375 bytes**

### (d) 1.5-Mbps T1 link through geosynchronous satellite (35,900 km altitude)

The signal goes up and back down: one-way distance = 2 × 35,900 km = 71,800 km.

One-way delay = 71,800,000 / (3 × 10⁸) = 0.2393 s ≈ **239.3 ms**

BDP = 1.5 × 10⁶ × 0.2393 = **358,950 bits ≈ 44.9 KB**

---

## Exercise 21

> Hosts A and B connected via a store-and-forward switch S, 10-Mbps links, 20 µs propagation per link. Switch processing delay = 35 µs after fully receiving a packet. Total data: 10,000 bits.

### (a) Single 10,000-bit packet

Transmit time per link = 10,000 / 10⁷ = 1000 µs

| Event | Time (µs) |
|-------|-----------|
| A starts sending | 0 |
| A finishes, packet on wire | 1000 |
| Packet fully arrives at S | 1020 |
| S finishes processing, starts retransmitting | 1055 |
| S finishes sending | 2055 |
| Last bit arrives at B | 2075 |

**Total = 2 × 1000 + 2 × 20 + 35 = 2075 µs**

### (b) Two 5000-bit packets sent back-to-back

Transmit time per packet per link = 5000 / 10⁷ = 500 µs

| Event | Time (µs) |
|-------|-----------|
| T = 0 | A starts sending packet 1 |
| T = 500 | A finishes packet 1, starts packet 2 |
| T = 520 | Packet 1 fully arrives at S |
| T = 555 | S starts forwarding packet 1 |
| T = 1000 | A finishes sending packet 2 |
| T = 1020 | Packet 2 fully arrives at S |
| T = 1055 | S finishes sending packet 1 (555 + 500); also S is ready to send packet 2 (1020 + 35 = 1055) |
| T = 1055 | S starts sending packet 2 |
| T = 1555 | S finishes sending packet 2 |
| T = 1575 | Last bit of packet 2 arrives at B |

**Total = 1575 µs**

> *Why is (b) faster?* By splitting data into two smaller packets, packet 1 can traverse the switch while packet 2 is still being sent on the first link. This **pipelining** saves 500 µs (one packet transmission time).

---

## Exercise 22

> A host has a 1-MB file. Compressing 50% takes 1 second; compressing 60% takes 2 seconds.

### (a) At what bandwidth does each compression option break even?

Without compression: time = 1 MB / bandwidth

With compression: time = CPU time + compressed size / bandwidth

Break-even when they are equal:

**CPU time = (uncompressed − compressed) / bandwidth**

**bandwidth = size reduction / CPU time**

| Option | Reduction | CPU time | Break-even bandwidth |
|--------|----------|----------|---------------------|
| 50% compression | 0.5 MB | 1 s | 0.5 MB/s = **4 Mbps** |
| 60% compression | 0.6 MB | 2 s | 0.3 MB/s = **2.4 Mbps** |

> Below these bandwidths, compression saves time. Above them, the CPU time for compression exceeds the time saved on transmission.

### (b) Why doesn't latency affect the answer?

Latency (propagation delay) is the **same** whether or not the data is compressed — it depends only on the physical distance and speed of signal propagation, not on the size of the data. Therefore, latency cancels out when comparing compressed vs. uncompressed transmission time.

---

## Exercise 23

> Per-packet overhead of 100 bytes. We send 1 million bytes; one byte is corrupted and its entire packet is lost. Give overhead + loss for packet data sizes of 1000, 5000, 10,000, and 20,000 bytes. Which is optimal?

**Method:** Number of packets N = ⌈10⁶ / D⌉. Overhead = 100 × N. Loss = D (one full packet's data is lost; its header is already counted in overhead).

Total = 100 × ⌈10⁶/D⌉ + D

| Data size D | Packets N | Overhead (100×N) | Loss (D) | **Total** |
|------------|----------|-----------------|----------|-----------|
| 1,000 | 1,000 | 100,000 | 1,000 | **101,000** |
| 5,000 | 200 | 20,000 | 5,000 | **25,000** |
| 10,000 | 100 | 10,000 | 10,000 | **20,000** |
| 20,000 | 50 | 5,000 | 20,000 | **25,000** |

**Optimal data size = 10,000 bytes** (minimizes total overhead + loss at 20,000 bytes).

> *Mathematical optimum:* Minimize f(D) = 10⁸/D + D. Setting f′(D) = 0: D² = 10⁸, so D = 10,000. ✓
>
> *Insight:* Small packets → lots of header overhead. Large packets → more data lost when one is corrupted. The optimal size balances these two costs.

---

## Exercise 24

> Transfer an n-byte file over a path with source, destination, 7 links, 5 switches. Links: 2 ms propagation, 4 Mbps. Packets: 24-byte header + 1000-byte payload (1024 B total). Store-and-forward with 1-ms processing per switch. Packets sent continuously. Circuit setup: 1-kB message round-trip, 1-ms delay per switch.

### (a) For what file size is total bytes less for circuits than packets?

**Packets:** Each packet carries 1000 bytes of data but sends 1024 bytes.
- Total bytes on wire = (n / 1000) × 1024 = **1.024n**

**Circuits:** One-time setup cost, then raw data.
- Setup: 1 kB message sent each way = 2 × 1024 = 2048 bytes
- Data: n bytes (no headers)
- Total bytes on wire = **2048 + n**

Circuits use fewer bytes when:

2048 + n < 1.024n → 2048 < 0.024n → **n > 85,333 bytes ≈ 85.3 kB**

> For files larger than ~85 kB (86 packets), circuits send fewer total bytes because the per-packet 24-byte header overhead accumulates.

### (b) For what file size is total latency less for circuits?

**Packet latency** (with pipelining):

Packet transmit time per link: t = 1024 × 8 / 4,000,000 = 2.048 ms

One packet end-to-end: 6 transmissions × 2.048 + 7 × 2 (propagation) + 5 × 1 (processing) = 12.288 + 14 + 5 = 31.288 ms

With pipelining, total = one-packet-latency + (c − 1) × t, where c = n/1000:

**T_packet = 31.288 + (n/1000 − 1) × 2.048 = 29.24 + 0.002048n ms**

**Circuit latency:**

Setup (round trip for 1-kB message):
- Each way: 6 × 2.048 + 7 × 2 + 5 × 1 = 31.288 ms
- Round trip: 62.576 ms

Data transfer (continuous bitstream, no switch delays):
- Transmit: n × 8 / 4,000,000 = 0.002n ms
- Propagation: 7 × 2 = 14 ms

**T_circuit = 62.576 + 14 + 0.002n = 76.576 + 0.002n ms**

Break-even: 29.24 + 0.002048n = 76.576 + 0.002n

0.000048n = 47.336 → **n ≈ 986,167 bytes ≈ 987 kB**

> Circuits achieve lower latency for files larger than ~987 kB because the per-switch store-and-forward delay for every packet accumulates and eventually exceeds the one-time circuit setup cost.

### (c) Sensitivity analysis

| Parameter change | Effect on crossover |
|-----------------|-------------------|
| More switches | Increases the crossover point (packets get slower relative to circuits) |
| Higher bandwidth | Increases the crossover point (both improve, but the difference shrinks) |
| Larger packet payload (relative to header) | Decreases overhead ratio, making packets competitive for larger files |

### (d) Model accuracy

The model ignores:
- **Other traffic** competing for links and switches.
- **Queueing delays** at switches (which would hurt packets more).
- **Memory/state at switches** required for circuits.
- **Fault recovery**: circuits require re-establishment if a switch fails; packets can be rerouted.
- **Complex topologies**: real paths may not be simple source-to-destination chains.

---

## Exercise 25

> Ring topology, 100 Mbps, propagation speed 2 × 10⁸ m/s, 250-byte packets, nodes every 100 m introducing 10 bits of delay.

### Without node delay

Packet size = 250 × 8 = 2000 bits

Transmission time = 2000 / (100 × 10⁶) = **20 µs**

Circumference = time × propagation speed = 20 × 10⁻⁶ × 2 × 10⁸ = **4000 m = 4 km**

### With nodes every 100 m, each adding 10 bits of delay

Without nodes: 2000 bits / 4000 m = **0.5 bits/m** in the cable.

Per 100 m of cable: 50 bits in cable + 10 bits of node delay = **60 bits per 100-m segment**.

For 2000 bits total: 2000 / 60 × 100 = **3333 m ≈ 3.33 km**

> *With node delays, the ring is shorter* because some of the "storage" is in the nodes rather than on the cable.

---

## Exercise 26

> Compare voice traffic and real-time music transmission in terms of bandwidth, delay, and jitter.

**Answer:**

| Property | Voice | Music | Music improvement needed |
|----------|-------|-------|------------------------|
| **Bandwidth** | ~64 kbps | ~128 kbps – 1.4 Mbps (CD quality) | **10–20× more** |
| **Delay (latency)** | < 150 ms for interactive conversation | Seconds acceptable (non-interactive) | **Can be relaxed** |
| **Jitter** | Must be low (< ~30 ms) | Must be low (audible glitches) | **Similar requirement** |
| **Loss tolerance** | An audible error every few seconds is okay | Errors should be ≈100× less frequent | **Much stricter** |

> *Key point:* Music needs much more bandwidth and lower error rates than voice but can tolerate much higher latency (since it is typically one-way streaming, not a two-way conversation).

---

## Exercise 27

> Calculate the bandwidth needed for real-time transmission (no compression).

### (a) Video: 640 × 480, 3 bytes/pixel, 30 fps

Bandwidth = 640 × 480 × 3 × 30 = **27,648,000 bytes/s ≈ 26.4 MB/s ≈ 211 Mbps**

### (b) Video: 160 × 120, 1 byte/pixel, 5 fps

Bandwidth = 160 × 120 × 1 × 5 = **96,000 bytes/s ≈ 93.75 KB/s ≈ 768 kbps**

### (c) CD-ROM audio: 650 MB for 75 minutes

Bandwidth = 650 × 10⁶ / (75 × 60) = **144,444 bytes/s ≈ 141 KB/s ≈ 1.16 Mbps**

### (d) Fax: 8 × 10 inch, 72 dpi, B&W, over 14.4-kbps modem

Total pixels = 8 × 72 × 10 × 72 = 414,720 pixels = 414,720 bits (1 bit per B&W pixel)

Time = 414,720 / 14,400 = **28.8 seconds**

---

## Exercise 28

> Calculate bandwidth for real-time transmission (no compression).

### (a) HDTV: 1920 × 1080, 24 bits/pixel, 30 fps

Bandwidth = 1920 × 1080 × 24 × 30 = **1,492,992,000 bps ≈ 1.49 Gbps**

### (b) POTS voice: 8-bit samples at 8 kHz

Bandwidth = 8 × 8000 = **64,000 bps = 64 kbps**

### (c) GSM mobile voice: 260-bit samples at 50 Hz

Bandwidth = 260 × 50 = **13,000 bps = 13 kbps**

### (d) HDCD audio: 24-bit samples at 88.2 kHz

Per channel: 24 × 88,200 = **2,116,800 bps ≈ 2.12 Mbps**

For stereo (2 channels): **≈ 4.23 Mbps**

> *These show the enormous range* of bandwidth requirements — from 13 kbps (GSM voice) to 1.49 Gbps (uncompressed HDTV). This is why compression is essential in practice.

---

## Exercise 29

> Discuss relative performance needs of the following applications.

| Application | Avg BW | Peak BW | Latency | Jitter | Loss tolerance |
|-------------|--------|---------|---------|--------|---------------|
| **(a) File server** | Moderate | **Very high** | Low-moderate | Not critical | **None** (must retransmit) |
| **(b) Print server** | Low-moderate | Moderate | Not critical | Not critical | None |
| **(c) Digital library** | Moderate | High | Moderate | Not critical | None |
| **(d) Weather monitoring** | Low | Low | Not critical | Not critical | Some loss OK |
| **(e) Voice** | Low (64 kbps) | Low | **< 150 ms** | **Low** | Minor dropouts OK |
| **(f) Video monitoring** | Low-moderate | Moderate | Moderate (seconds OK) | Moderate | Frame loss OK |
| **(g) TV broadcasting** | **Very high** | Very high | Can be hours | Bounded (buffering helps) | Some loss OK |

---

## Exercise 30

> A shared medium M offers hosts round-robin opportunities to transmit one packet; hosts with nothing to send relinquish. How does this differ from STDM?

**Answer:**

In **STDM**, each station is assigned a fixed timeslot regardless of whether it has data to send. Unused timeslots are **wasted** — no other station can use them.

In the **round-robin** scheme, a station with nothing to send immediately yields its turn. The medium quickly moves to the next station that actually has data.

**Network utilization comparison:** Round-robin is **significantly higher** than STDM because bandwidth is not wasted on idle stations. Only stations with data consume network time.

> *STDM is like assigning each person a fixed 1-minute slot in a meeting, even if they have nothing to say. Round-robin skips silent participants.*

---

## Exercise 31

> Simple stop-and-wait file transfer: A sends 1-kB packets to B, waits for ACK. Lost packets are retransmitted.

### (a) Why no sequence numbers are needed if no packets are lost or duplicated?

In stop-and-wait with no losses, the sender sends exactly one packet and waits. When the ACK arrives, it knows the packet was received and sends the next one. There is an implicit ordering: **the Nth ACK received means the Nth packet was delivered.** Since each packet is followed by exactly one ACK, counting is implicit.

### (b) With possible loss (but in-order delivery): is a 1-bit sequence number sufficient?

**Yes**, a 1-bit sequence number (0 or 1, alternating) is sufficient.

The only ambiguity that can arise is: *did the receiver get the packet, or is this a retransmission?* With one bit:
- Sender sends packet with sequence number 0, waits for ACK[0]
- If timeout, retransmit packet 0
- Receiver can distinguish: if it gets sequence 0 when expecting 0 → new packet. If it gets 0 when expecting 1 → duplicate (already ACKed, re-send ACK).

A 2-bit sequence number also works but is more than necessary.

### (c) With out-of-order delivery (packets up to 1 minute late)?

If old packets can arrive up to 1 minute after later packets, the sequence number space must be large enough that a packet from 1 minute ago cannot be confused with a current packet.

Required sequence number range ≥ bandwidth × 1 minute / packet size

This ensures that in any 1-minute window, all packets have distinct sequence numbers.

---

## Exercise 32

> Host A continuously sends timestamps to B. B pairs each received timestamp with its own clock. Give qualitative output for:

### (a) High bandwidth, high latency, low jitter

Readings are frequent (high bandwidth), each delayed by a constant large offset (high latency), and very regular (low jitter):

```
(1000, 1100), (1001, 1101), (1002, 1102), (1003, 1103), (1004, 1104)
```

Latency ≈ 100 units. Readings come every 1 unit, very regular.

### (b) Low bandwidth, high latency, high jitter

Readings are infrequent (low bandwidth), with a large variable delay (high latency + high jitter):

```
(1000, 1100), (1020, 1110), (1040, 1145), (1060, 1180), (1080, 1184)
```

Readings spaced ~20 units (low bandwidth). Latency fluctuates between 90 and 120 (high jitter).

### (c) High bandwidth, low latency, low jitter, occasional lost data

Readings are frequent and regular with small delay, but occasionally one is missing:

```
(1000, 1005), (1001, 1006), (1003, 1008), (1004, 1009), (1005, 1010)
```

Note timestamp 1002 is missing (lost). Latency ≈ 5 units, very consistent.

---

## Exercises 33–36 (Programming Exercises)

These exercises involve building and modifying the `simplex-talk` socket program from the textbook. Key guidance:

### Exercise 33: Server with multiple clients

- Start one server and one client. While the first client is active, start 10 more clients.
- With `MAX_PENDING = 1`: only 1–2 connections are queued (accepted by the OS but not yet processed by the server). Others will eventually timeout.
- When the first client exits, any queued connections are processed in order.

### Exercise 34: Echo server (TCP)

- Modify both client and server to alternate `send()` and `recv()` calls.
- After the client sends a line, it calls `recv()` to read back the echoed line.
- The server reads a line with `recv()`, then sends it back with `send()`.

### Exercise 35: UDP version

- Change `SOCK_STREAM` to `SOCK_DGRAM`.
- Remove `listen()` and `accept()` from server.
- Replace nested loops with a single `recv()` loop on socket `s`.
- **Key difference from TCP:** Two UDP clients can send simultaneously to the server; their messages are interleaved (no connection state prevents this). TCP would require separate connections.

### Exercise 36: TCP options

- Use `man tcp` to explore socket options.
- Key options include `TCP_NODELAY` (disable Nagle), `SO_RCVBUF`/`SO_SNDBUF` (buffer sizes), `TCP_KEEPALIVE`, etc.

---

## Exercises 37–39 (Network Utility Exercises)

### Exercise 37: Using `ping`

```bash
ping www.cs.princeton.edu
ping www.cisco.com
```

Measure RTT at different times of day. Differences are caused by:
- **Network congestion** (more traffic during business hours).
- **Queueing delays** at routers.
- **Geographic distance** (Princeton NJ vs Cisco CA).

### Exercise 38: Using `traceroute`

```bash
traceroute www.cs.princeton.edu
```

Hop count does **not** always correlate well with RTT (a few long-distance hops may have larger delay than many local hops). Geographic distance correlates better with RTT but is not perfect (routing is not always geographically optimal).

### Exercise 39: Internal routing

Use `traceroute` to map routers within your organization. Small organizations may have no internal routers (all hosts on the same subnet); larger ones may have several layers.
