# Chapter 1 — Key Concepts & Notes Summary

**Computer Networks: A Systems Approach, 6th Edition**
*Larry L. Peterson & Bruce S. Davie*

> This summary covers the essential concepts from Chapter 1 that are tested in the exercises. Use it as a study guide for your upcoming test.

---

## 1. Network Performance Metrics

These are the fundamental quantities you need to understand and compute.

### 1.1 Bandwidth (Throughput)

- **Definition:** The number of bits per second (bps) that can be transmitted on a link.
- **Units:** bps, Kbps, Mbps, Gbps.
- **Key distinction:** *Bandwidth* is the link capacity; *throughput* is the actual achieved rate (may be less due to overhead, contention, etc.).

### 1.2 Latency (Delay)

Total latency is the sum of three components:

```
Latency = Propagation delay + Transmit delay + Queuing delay
```

| Component | Formula | Depends on |
|-----------|---------|------------|
| **Propagation delay** | distance / speed of signal | Link length and medium |
| **Transmit delay** (serialization) | packet size / bandwidth | Packet size and link speed |
| **Queuing delay** | varies | Network load |

### 1.3 Round-Trip Time (RTT)

- **Definition:** Time for a message to travel from sender to receiver and back.
- RTT ≈ 2 × one-way propagation delay (ignoring transmit time of small control packets).
- Critical for protocols: the sender cannot get feedback faster than 1 RTT.

### 1.4 Delay × Bandwidth Product

```
Delay × Bandwidth = one-way delay × link bandwidth
```

- Represents the **volume of data that fills the pipe** — data in flight between sender and receiver.
- Measured in bits or bytes.
- *Intuition:* Think of the link as a pipe — delay × bandwidth is the pipe's capacity.

### 1.5 Jitter

- **Definition:** Variation in latency from packet to packet.
- Critical for real-time applications (voice, video). High jitter causes choppy audio/video even if average latency is acceptable.

---

## 2. Data Transfer Time Calculations

This is the most common type of exercise. The general formula is:

```
Total time = Handshake + Transmission time + Propagation delay + Waiting time
```

### 2.1 Continuous Sending

```
Time = 2 × RTT (handshake) + File size / Bandwidth + RTT/2 (propagation)
```

### 2.2 Stop-and-Wait

After each packet, the sender waits for an ACK (1 RTT):

```
Time = 2 × RTT + (N-1) × (transmit_per_packet + RTT) + transmit_per_packet + RTT/2
```

where N = number of packets.

### 2.3 Windowed Sending

If the window allows W packets per RTT:

```
Rounds = ⌈N / W⌉
Time = 2 × RTT + (Rounds - 1) × RTT + RTT/2
```

### 2.4 Slow Start (Exponential Growth)

Packets sent per round: 1, 2, 4, 8, ..., 2ⁿ

Cumulative after round n: 2^(n+1) − 1

Find smallest n such that 2^(n+1) − 1 ≥ N, then:

```
Time = 2 × RTT + n × RTT + RTT/2
```

---

## 3. Propagation vs. Transmission Delay

A key conceptual distinction:

| | Propagation Delay | Transmission Delay |
|---|---|---|
| **What it is** | Time for a bit to travel across the link | Time to push all bits of a packet onto the link |
| **Depends on** | Distance and speed of light/signal | Packet size and link bandwidth |
| **Formula** | d / s | L / R |
| **Analogy** | Driving time on a highway | Time for an entire convoy to enter the highway |

**When propagation = transmission:**

Set d/s = L/R and solve for bandwidth R:

```
R = L × s / d
```

where L = packet size (bits), s = propagation speed, d = distance.

---

## 4. Store-and-Forward vs. Cut-Through Switching

### Store-and-Forward

- Switch must receive the **entire packet** before forwarding.
- Latency through one switch = transmit delay + propagation delay (on each link).
- For k switches and (k+1) links:
  ```
  Latency = (k+1) × transmit + (k+1) × propagation + k × processing
  ```

### Cut-Through Switching

- Switch begins forwarding after receiving only the **header** (e.g., first 128–200 bits).
- Only the first bit faces the cut-through delay at each switch.
- Formula:
  ```
  Latency = transmit time + k × cut-through delay + (k+1) × propagation
  ```
  where cut-through delay = header bits / bandwidth.

### Pipelining

When multiple switches can send and receive simultaneously, the **effective bandwidth in steady state** equals the link bandwidth. Only the first packet pays the full pipeline latency.

---

## 5. Addressing

### Key Properties of Network Addresses

1. **Uniqueness** — each address identifies exactly one destination.
2. **Hierarchical** — supports aggregation for routing (e.g., IP prefix).
3. **Fixed-length** — efficient for hardware processing.
4. **Administratively assigned** — allows planned allocation.

### Address Types

| Type | Description | Example |
|------|------------|---------|
| **Unicast** | One sender, one receiver | Standard IP address |
| **Broadcast** | One sender, all receivers | 255.255.255.255 |
| **Multicast** | One sender, a group of receivers | 224.0.0.0/4 |
| **Anycast** | One sender, nearest of multiple receivers | DNS root servers |

---

## 6. Multiplexing

### FDM (Frequency-Division Multiplexing)

- Each channel gets a dedicated frequency band.
- Best for: constant-bandwidth channels (TV/radio).
- Waste: unused frequency bands sit idle.

### STDM (Synchronous Time-Division Multiplexing)

- Each channel gets a fixed timeslot in a repeating frame.
- Best for: uniform, constant-rate channels (voice telephony).
- Waste: unused timeslots sit idle.

### Statistical Multiplexing (Packet Switching)

- Bandwidth is shared on demand — packets from different sources are interleaved.
- Best for: **bursty, variable-rate** traffic (computer networks).
- Advantage: bandwidth from idle sources is available to active sources.

---

## 7. Circuit Switching vs. Packet Switching

| Feature | Circuit Switching | Packet Switching |
|---------|------------------|-----------------|
| **Setup** | Required (connection established first) | None |
| **Overhead** | Setup cost, then zero per-data overhead | Per-packet header overhead |
| **Switch delay** | None (dedicated path) | Store-and-forward at each switch |
| **Resource usage** | Dedicated (wasted if idle) | Shared (efficient for bursty traffic) |
| **Failure recovery** | Must re-establish circuit | Packets can be rerouted |

**When circuits win:** Large files, steady long-duration flows (the setup cost is amortized).

**When packets win:** Short transfers, many concurrent connections, bursty traffic.

---

## 8. Packet Overhead and Optimal Packet Size

Every packet carries a header (overhead bytes). More, smaller packets = more total overhead. Fewer, larger packets = more data lost if one is corrupted.

**Total cost** = header overhead + loss cost:

```
f(D) = H × ⌈FileSize / D⌉ + D
```

Minimized when D = √(H × FileSize).

> *Takeaway:* There is an optimal packet size that balances overhead against vulnerability to loss.

---

## 9. Compression Trade-offs

Compression reduces the amount of data to transmit but costs CPU time:

```
Total time with compression = CPU time + compressed size / bandwidth
Total time without compression = original size / bandwidth
```

Compression is beneficial when:

```
bandwidth < size reduction / CPU time
```

Latency does **not** affect this comparison (it is the same in both cases).

---

## 10. Application Requirements

Different applications have different needs:

| Requirement | Explanation |
|------------|-------------|
| **Bandwidth-sensitive** | Performance limited by link speed (large data transfers) |
| **Delay-sensitive** | Performance limited by RTT (interactive or small-message apps) |
| **Jitter-sensitive** | Needs consistent timing between packets (real-time audio/video) |
| **Loss-tolerant** | Can function with some lost data (streaming media) |
| **Loss-intolerant** | Every bit must arrive correctly (file transfer, database) |

---

## 11. Protocol Basics

### Stop-and-Wait

- Sender sends one frame, waits for ACK.
- Simple but very low utilization on high-latency links.
- Needs at least a **1-bit sequence number** to handle lost packets (distinguish new packets from retransmissions).

### Sliding Window (Preview)

- Sender can have multiple unacknowledged frames in flight.
- Window size determines how many frames can be outstanding.
- Required sequence number space depends on window size and the possibility of out-of-order delivery.

---

## 12. Bandwidth Calculation Quick Reference

| Medium/Application | Calculation | Result |
|---|---|---|
| 640×480 video, 3 B/pixel, 30 fps | 640×480×3×30 | ~211 Mbps |
| 160×120 video, 1 B/pixel, 5 fps | 160×120×1×5 | ~768 kbps |
| CD audio (650 MB / 75 min) | 650M / 4500s | ~1.16 Mbps |
| HDTV (1920×1080, 24 bit, 30 fps) | 1920×1080×3×30 | ~1.49 Gbps |
| POTS voice (8 bit, 8 kHz) | 8×8000 | 64 kbps |
| GSM voice (260 bit, 50 Hz) | 260×50 | 13 kbps |
| HDCD audio (24 bit, 88.2 kHz) | 24×88200 | ~2.12 Mbps/ch |

---

## 13. Useful Unit Conversions

| Quantity | Equivalences |
|----------|-------------|
| 1 byte | 8 bits |
| 1 kB | 1024 bytes = 8192 bits |
| 1 MB | 1,048,576 bytes ≈ 10⁶ bytes |
| 1 GB | ~10⁹ bytes |
| 1 Mbps | 10⁶ bits per second |
| 1 Gbps | 10⁹ bits per second |
| 1 ms | 10⁻³ seconds |
| 1 µs | 10⁻⁶ seconds |
| 1 ns | 10⁻⁹ seconds |

> **Caution:** In networking, "mega" and "giga" usually mean 10⁶ and 10⁹ (SI units), but file sizes often use 2²⁰ and 2³⁰. Be consistent and state your assumptions.

---

## 14. Problem-Solving Strategy

1. **Identify the quantities given**: bandwidth, distance, packet size, RTT, etc.
2. **Draw a timeline** if the problem involves multiple events.
3. **Break the total time into components**: handshake, transmit, propagation, processing, waiting.
4. **Watch out for:**
   - Whether to use one-way delay or RTT.
   - Whether "bandwidth" means bits or bytes.
   - Store-and-forward vs. cut-through.
   - Pipelining effects with multiple switches.
5. **Sanity-check your answer**: Is the total reasonable? Does higher bandwidth give a shorter time?
