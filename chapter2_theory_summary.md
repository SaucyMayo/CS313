# Chapter 2 — Direct Links: In-Depth Theory Summary

**Computer Networks: A Systems Approach, 6th Edition**
*Larry L. Peterson & Bruce S. Davie*

> This summary provides a thorough, section-by-section walkthrough of the theoretical concepts in Chapter 2. It covers how raw physical links are transformed into useful building blocks for computer networks by addressing five fundamental problems: encoding, framing, error detection, reliable delivery, and media access control.

---

## Overview: The Five Fundamental Problems

Connecting two nodes with a physical medium is only the first step. Before they can exchange packets and provide **Layer 2 (L2) connectivity**, five problems must be solved:

| Problem | Question Answered |
|---------|-------------------|
| **1. Encoding** | How do we represent bits as signals on the physical medium? |
| **2. Framing** | How does the receiver identify where each frame begins and ends? |
| **3. Error Detection** | How do we detect when bits have been corrupted during transmission? |
| **4. Reliable Delivery** | How do we recover from frames that are lost or corrupted? |
| **5. Media Access Control** | When multiple hosts share a link, who gets to transmit and when? |

These issues are explored in the context of real technologies: point-to-point fiber (SONET), Carrier Sense Multiple Access networks (Ethernet, Wi-Fi), fiber-to-the-home (PON), and cellular (4G/5G).

---

## 2.1 Technology Landscape

### Diversity of Links

The Internet encompasses an enormous variety of link types, each chosen for different economic, geographic, and deployment reasons:

- **Long-distance backbone links** — span hundreds or thousands of km, connecting refrigerator-sized routers. Almost exclusively **optical fiber** using **SONET**.
- **Local Area Networks (LANs)** — inside buildings or campuses. Dominated by **Ethernet** (wired) and **Wi-Fi** (wireless).
- **Last-mile / access links** — connect end users to their ISP. Technologies include:

| Service | Medium | Typical Bandwidth |
|---------|--------|-------------------|
| **DSL** | Copper (twisted pair) | Up to 100 Mbps |
| **G.Fast** | Copper (apartment buildings) | Up to 1 Gbps |
| **PON** | Optical fiber | Up to 10 Gbps |
| **Cellular (4G/5G)** | Radio | 1–5+ Mbps (4G); 10× more (5G) |

- **Mobile / Cellular** — connects phones and mobile devices, with the benefit of maintaining connectivity while moving.

### The Role of Abstraction

Part of the role of a network architecture is to provide a **common abstraction of a link**. Your laptop does not care whether it is connected via Wi-Fi, Ethernet, or cellular — it only cares that it has a link to the Internet. Similarly, a router does not care what type of link connects it to other routers.

Making all these different physical media look "sufficiently alike" requires dealing with real-world physical limitations and is the focus of this chapter.

### The Shannon–Hartley Theorem

The theoretical upper bound on a link's capacity is given by:

```
C = B × log₂(1 + S/N)
```

Where:
- **C** = channel capacity (bps)
- **B** = bandwidth of the channel (Hz)
- **S/N** = signal-to-noise ratio (linear scale)

The SNR is often expressed in decibels: `SNR_dB = 10 × log₁₀(S/N)`

**Example:** A voice-grade phone line with B = 3000 Hz and SNR = 30 dB (S/N = 1000):
```
C = 3000 × log₂(1001) ≈ 30 kbps
```

**Key insight:** There are only two ways to build a high-capacity link — start with a high-bandwidth channel or achieve a high signal-to-noise ratio.

### Electromagnetic Spectrum

Links are classified by the **medium** (copper, fiber, air) and the **frequency** of the electromagnetic waves used:
- Radio waves → wireless links
- Infrared / visible light → optical fiber
- Cellular networks use frequencies from 700 MHz to 2400 MHz (traditional), with new 5G allocations at 6 GHz (mid-spectrum) and above 24 GHz (millimeter wave).

### Encoding vs. Modulation

The problem of encoding binary data onto electromagnetic signals is divided into two layers:
1. **Modulation** (lower layer) — varies frequency, amplitude, or phase to transmit information. We simplify this to "high" and "low" signals.
2. **Encoding** (upper layer) — maps binary data (0s and 1s) onto the distinguishable signals. This is what Section 2.2 addresses.

---

## 2.2 Encoding

Encoding is the process of representing binary data as signals on a physical medium. It is performed by the **network adaptor** (specifically, its signaling component). Signals travel between signaling components over the link; bits flow between network adaptors.

### 2.2.1 NRZ (Non-Return to Zero)

The simplest encoding: map 1 → high signal, 0 → low signal.

**Problems with NRZ:**

1. **Baseline wander** — the receiver keeps a running average of the signal to distinguish high from low. Long runs of consecutive 1s or 0s cause this average to drift, making detection unreliable.

2. **Clock recovery failure** — the sender and receiver must be precisely synchronized. Transitions in the signal (high↔low) provide synchronization points. Long runs without transitions cause **clock drift**, where the receiver's clock gradually loses sync with the sender's.

### 2.2.2 NRZI (Non-Return to Zero Inverted)

- **Rule:** A **1** causes a **transition** (toggle the signal); a **0** keeps the signal unchanged.
- **Solves:** Consecutive 1s — each 1 produces a transition.
- **Does not solve:** Consecutive 0s — no transitions occur.

### 2.2.3 Manchester Encoding

- **Rule:** Transmit the XOR of the NRZ-encoded data and the clock signal. This produces a transition in the **middle** of every bit period:
  - 0 → low-to-high transition
  - 1 → high-to-low transition (or vice versa, by convention)
- **Advantage:** Clock is embedded in the signal — the receiver can synchronize on every single bit.
- **Disadvantage:** The signal transitions **twice as fast** as the data rate. The encoding is only **50% efficient** (the bit rate is half the baud rate).
- **Used by:** Classic 10-Mbps Ethernet.

**Baud rate vs. bit rate:**
- **Baud rate** = the rate of signal changes (transitions per second).
- **Bit rate** = the rate of data bits per second.
- With Manchester: bit rate = baud rate / 2.
- With advanced modulation (e.g., QAM encoding 4+ bits per symbol): bit rate > baud rate.

### 2.2.4 4B/5B Encoding

A compromise between NRZ's efficiency and Manchester's reliability:

- **Idea:** Map every 4 data bits to a carefully chosen 5-bit code.
- **Code selection:** Each 5-bit code has at most 1 leading 0 and at most 2 trailing 0s. This ensures no more than **3 consecutive 0s** when codes are sent back-to-back.
- **Combined with NRZI:** NRZI handles runs of 1s; 4B/5B limits runs of 0s. Together, they ensure sufficient transitions for clock recovery.
- **Efficiency:** 80% (4 useful bits per 5 transmitted).

Since 5 bits can represent 32 codes but only 16 are needed for data, the remaining 16 serve special purposes:
- `11111` = line idle
- `00000` = line dead
- `00100` = halt
- 7 are invalid (violate the 0-run rules)
- 6 are control symbols used by framing protocols

---

## 2.3 Framing

Once we can transmit bits over a link, the next problem is **delineating sequences of bits into complete frames** (messages). The network adaptor on the sending side transmits a frame from the node's memory as a sequence of bits; the receiving adaptor must collect these bits and identify where each frame begins and ends.

### 2.3.1 Byte-Oriented Protocols (PPP)

Byte-oriented protocols view each frame as a collection of **bytes** (characters).

**Two approaches:**

**A. Sentinel-Based Framing**
- Special characters mark frame boundaries (e.g., SYN for start, STX/ETX for data boundaries).
- **Problem:** What if the sentinel character appears in the data?
- **Solution: Character stuffing** — precede any sentinel character in the data with a DLE (Data Link Escape) character. The DLE itself is also escaped (by preceding it with another DLE).

**B. Byte-Counting**
- The frame header includes the **number of bytes** in the frame.
- **Problem:** A transmission error could corrupt the count field, causing the receiver to misidentify the frame boundary (a **framing error**). The receiver must then wait for the next SYN character to resynchronize.

**Point-to-Point Protocol (PPP):**
PPP is the most common modern byte-oriented protocol for carrying IP packets over point-to-point links. Its frame format:

| Field | Size | Purpose |
|-------|------|---------|
| **Flag** | 8 bits | Start/end sentinel: `01111110` |
| **Address** | 8 bits | Default value (unused on point-to-point) |
| **Control** | 8 bits | Default value |
| **Protocol** | 16 bits | Demux key — identifies higher-level protocol (e.g., IP) |
| **Payload** | Up to 1500 bytes | Data (negotiable size) |
| **Checksum** | 16 or 32 bits | CRC for error detection |

PPP uses **Link Control Protocol (LCP)** to negotiate frame parameters. LCP messages are themselves carried in PPP frames (identified by an LCP value in the Protocol field), and they can modify PPP's frame format. LCP also handles link establishment when both sides detect that communication is possible.

### 2.3.2 Bit-Oriented Protocols (HDLC)

Bit-oriented protocols treat the frame as a collection of **bits**, regardless of byte boundaries.

**HDLC (High-Level Data Link Control):**
- Uses the distinguished bit sequence **`01111110`** as both start and end delimiter.
- This same sequence is transmitted during idle periods for clock synchronization.

**Bit Stuffing:**
Since `01111110` could appear anywhere in the data:
- **Sender:** After transmitting 5 consecutive 1s from the data body, **insert a 0**.
- **Receiver:** After seeing 5 consecutive 1s:
  - If the next bit is **0** → it was stuffed; remove it.
  - If the next bit is **1** followed by **0** → end-of-frame marker (`01111110`).
  - If the next bit is **1** followed by **1** → error (7+ consecutive 1s); discard the frame.

**Important property:** Both character stuffing and bit stuffing make the **frame size dependent on the data content**. It is impossible to guarantee fixed-size frames with arbitrary payloads.

### 2.3.3 Clock-Based Framing (SONET)

**SONET (Synchronous Optical Network)** uses a completely different approach:

- **Fixed-size frames** transmitted at precise intervals.
- **No bit stuffing** — frame size does not depend on data content.
- Frame boundaries are identified by a **special bit pattern** in the first 2 bytes. The receiver looks for this pattern appearing consistently every **810 bytes** (the STS-1 frame size: 9 rows × 90 bytes).
- Since no stuffing is used, the pattern may appear in the payload by coincidence. The receiver guards against false positives by requiring the pattern to appear at the correct interval multiple times before concluding it is synchronized.

**STS-1 Frame Structure:**
- 9 rows × 90 bytes = 810 bytes per frame.
- First 3 bytes of each row = overhead; remaining 87 bytes = data.
- Data rate: 51.84 Mbps.
- Each frame is exactly **125 µs** long.

**Clock Recovery:**
- Overhead bytes use NRZ encoding.
- Payload bytes are **scrambled** by XORing with a well-known 127-bit pattern that has many transitions, ensuring sufficient clock recovery.

**Multiplexing:**
- Higher-rate links (STS-N) are integer multiples of STS-1 (51.84 Mbps).
- An STS-N frame interleaves bytes from N STS-1 frames (not blocks), ensuring smooth, evenly paced delivery.
- **STS-Nc (concatenated):** The payloads of N STS-1 frames are linked into a single larger payload. STS-3c provides a single 155.25-Mbps pipe, whereas STS-3 provides three 51.84-Mbps pipes sharing a fiber.

**Payload floating:** The actual payload may not be aligned with frame boundaries — it can float across two frames. An overhead field points to the beginning of the payload. This simplifies clock synchronization across the carrier's network.

---

## 2.4 Error Detection

Bit errors are introduced during transmission (electromagnetic interference, thermal noise, etc.). Error detection adds **redundant bits** to each frame so the receiver can determine if corruption has occurred.

**Design goal:** Maximize the probability of detecting errors while minimizing the number of redundant bits. A 32-bit CRC can protect frames of thousands of bytes, catching the overwhelming majority of errors.

**Two approaches after detection:**
1. **Retransmission** — notify the sender to resend (used when errors are rare).
2. **Error correction** — use an error-correcting code to reconstruct the original data (used when retransmission is costly, e.g., satellite or wireless links). This is called **Forward Error Correction (FEC)**.

### 2.4.1 Internet Checksum Algorithm

The Internet checksum is used by higher-level Internet protocols (IP, TCP, UDP) — not at the link level, but it provides the same conceptual service.

**Algorithm:**
1. Treat the data as a sequence of **16-bit integers**.
2. Add them together using **16-bit ones' complement arithmetic** (carry bits wrap around to the least significant bit).
3. Take the **ones' complement** (invert all bits) of the result.
4. The 16-bit result is the checksum.

**Ones' complement arithmetic:** A negative integer −x is represented by inverting all bits of x. When adding, any carry out of the most significant bit is added back to the result.

**Properties:**
- Only 16 redundant bits for a message of any length — very compact.
- Relatively **weak** error detection. For example, if one 16-bit word is incremented and another is decremented by the same amount, the error is undetected.
- **Easy to implement in software** — the main advantage. This matters because the checksum is the last line of defense in end-to-end protocols; stronger error detection (CRC) handles most errors at the link level.

### 2.4.2 Cyclic Redundancy Check (CRC)

CRC provides much stronger error detection than checksums, using polynomial arithmetic over finite fields.

#### The Core Idea

1. Treat an (n+1)-bit message as a polynomial M(x) of degree n, where each bit is a coefficient. Example: `10011010` → M(x) = x⁷ + x⁴ + x³ + x¹.

2. Sender and receiver agree on a **divisor polynomial** C(x) of degree k. The choice of C(x) determines what types of errors can be detected.

3. The sender computes a **k-bit remainder** (the CRC) and appends it to the message. The resulting transmitted message P(x) is exactly divisible by C(x).

4. The receiver divides the received message by C(x). If the remainder is **zero**, no error is detected. If **nonzero**, an error occurred.

#### Computing the CRC

1. **Multiply** M(x) by x^k (append k zeros to the message).
2. **Divide** the extended message T(x) by C(x) using polynomial long division (XOR-based, no carries — this is modulo-2 arithmetic).
3. The **remainder** R(x) is the CRC (k bits long).
4. **Transmit** M(x) followed by R(x). This is equivalent to T(x) − R(x), which is exactly divisible by C(x).

**Polynomial arithmetic modulo 2:**
- Division is possible when the dividend's degree ≥ the divisor's degree.
- Subtraction (and addition) is **XOR** on corresponding coefficients.
- No carries or borrows.

#### Error Detection Power

If the transmitted message is P(x) and errors introduce E(x), the receiver sees P(x) + E(x). An error goes undetected only if E(x) is divisible by C(x). Careful choice of C(x) makes this extremely unlikely:

| Error Type | Detected if... |
|------------|----------------|
| **All single-bit errors** | C(x) has nonzero x^k and x⁰ terms |
| **All double-bit errors** | C(x) has a factor with at least 3 terms |
| **All odd numbers of errors** | C(x) contains the factor (x + 1) |
| **All burst errors of length ≤ k** | Always detected by a degree-k polynomial |
| **Most burst errors of length > k** | Detected with very high probability |

#### Standard CRC Polynomials

The most widely used is **CRC-32** (used by Ethernet):
```
CRC-32 = x³² + x²⁶ + x²³ + x²² + x¹⁶ + x¹² + x¹¹ + x¹⁰ + x⁸ + x⁷ + x⁵ + x⁴ + x² + x + 1
```

#### Hardware Implementation

CRC is easily implemented using a **k-bit shift register** and XOR gates. The message bits are shifted in from one end; when all bits (including the appended k zeros) have been processed, the shift register contains the CRC remainder.

#### Correction vs. Detection

- **Error correction** requires more redundant bits but avoids retransmission.
- **Error detection** requires fewer redundant bits but needs retransmission when errors occur.
- **Error correction** is preferred when: (1) errors are frequent (wireless) or (2) retransmission is prohibitively expensive (high-latency satellite links).
- **FEC (Forward Error Correction)** is commonly used in wireless networks like 802.11.

---

## 2.5 Reliable Transmission

Even with error detection, some frames will be corrupted and discarded. A protocol that wants **reliable delivery** must recover from these lost frames.

**Note:** Reliable delivery is a function that may be provided at the link layer, the transport layer, or the application layer. Many modern link technologies omit it entirely. The principles are the same regardless of which layer provides them.

The two fundamental mechanisms are:
- **Acknowledgments (ACKs)** — the receiver sends a small control frame confirming receipt.
- **Timeouts** — the sender retransmits if no ACK arrives within a reasonable time.

This strategy is called **Automatic Repeat reQuest (ARQ)**.

### 2.5.1 Stop-and-Wait

The simplest ARQ algorithm:

1. Sender transmits one frame.
2. Sender waits for an ACK.
3. If ACK arrives before timeout → send next frame.
4. If timeout expires → retransmit the same frame.

**Four scenarios (illustrated by protocol timelines):**
- (a) Normal: ACK arrives before timeout.
- (b) Frame lost: timeout triggers retransmission.
- (c) ACK lost: timeout triggers retransmission; receiver gets a duplicate.
- (d) Timeout too early: sender retransmits; receiver gets a duplicate.

**Duplicate detection:**
To distinguish retransmissions from new frames, the header includes a **1-bit sequence number** (alternating 0, 1, 0, 1, ...). The receiver can identify and discard duplicates while still acknowledging them (in case the previous ACK was lost).

**Performance limitation:**
Stop-and-wait allows only **one outstanding frame** at a time. On a 1.5-Mbps link with 45-ms RTT:
```
Delay × Bandwidth = 67.5 kbits ≈ 8 kB
Max sending rate = 1024 × 8 bits / 0.045 s = 182 kbps (only 1/8 of link capacity)
```

The sender wastes most of the link's capacity waiting for ACKs. To fully utilize the link, we need to have multiple frames in flight simultaneously — this is what the **sliding window** protocol provides.

### 2.5.2 Sliding Window

The sliding window algorithm allows **multiple frames to be outstanding** (unacknowledged) simultaneously, "keeping the pipe full."

#### Sender State

| Variable | Meaning |
|----------|---------|
| **SWS** (Send Window Size) | Maximum outstanding frames allowed |
| **LAR** (Last ACK Received) | Sequence number of most recent acknowledged frame |
| **LFS** (Last Frame Sent) | Sequence number of most recently sent frame |

**Invariant:** `LFS − LAR ≤ SWS`

When an ACK arrives, LAR advances, opening space to send more frames. A timer is associated with each outstanding frame for retransmission.

#### Receiver State

| Variable | Meaning |
|----------|---------|
| **RWS** (Receive Window Size) | Maximum out-of-order frames the receiver will buffer |
| **LFR** (Last Frame Received) | Sequence number of last in-order frame received |
| **LAF** (Largest Acceptable Frame) | `LFR + RWS` |

**Invariant:** `LAF − LFR ≤ RWS`

When a frame arrives:
- If its sequence number is outside `[LFR+1, LAF]` → discard.
- If within the window → accept and buffer.
- The receiver sends a **cumulative ACK** for the highest in-order sequence number received. For example, if frames 7 and 8 arrive but frame 6 is missing, no new ACK is sent until frame 6 arrives, at which point the receiver ACKs frame 8.

#### Finite Sequence Numbers

In practice, sequence numbers are finite (e.g., a 3-bit field gives 8 possible values: 0–7). This means sequence numbers **wrap around**, creating a risk of confusing old frames with new ones.

**The critical rule:**

```
MaxSeqNum ≥ SWS + RWS
```

- If **RWS = 1** (Go-Back-N): `MaxSeqNum ≥ SWS + 1`
- If **RWS = SWS** (Selective Repeat): `MaxSeqNum ≥ 2 × SWS`

**Why this matters:** If the sequence number space is too small, lost ACKs can cause the receiver's window to advance, and when old frames are retransmitted, they are indistinguishable from new frames.

**Intuition:** The sliding window protocol alternates between two halves of the sequence number space, analogous to how stop-and-wait alternates between 0 and 1 — but it *slides* continuously rather than discretely switching.

#### Sizing the Window

To fully utilize a link, the window must cover the delay × bandwidth product:

```
SWS ≥ (Bandwidth × RTT) / FrameSize
```

Minimum bits for the sequence number field = ⌈log₂(MaxSeqNum)⌉

#### Cumulative vs. Selective ACKs

| ACK Type | Description | Pros / Cons |
|----------|-------------|-------------|
| **Cumulative ACK** | Acknowledges all frames up to N | Simple; sender knows everything up to N arrived |
| **Selective ACK (SACK)** | Acknowledges specific individual frames | More information for sender; more complex |
| **Duplicate ACK (DUPACK)** | Re-sends the last cumulative ACK when out-of-order frame arrives | Hints at frame loss; used for fast retransmit (3 DUPACKs → retransmit without waiting for timeout) |

#### Three Roles of the Sliding Window

The sliding window protocol serves three distinct functions, and it is important to distinguish between them (the **separation of concerns** principle):

1. **Reliable delivery** — the core function. Retransmit lost frames.
2. **Ordered delivery** — the receiver buffers out-of-order frames and delivers them in sequence. (Is this needed at the link level, or should it be handled higher up?)
3. **Flow control** — the receiver advertises how many frames it can accept, throttling the sender to prevent buffer overflow. (Again, is this needed at the link level?)

### 2.5.3 Concurrent Logical Channels

The original ARPANET used an alternative to the sliding window: **multiplex several logical channels** onto a single physical link and run **stop-and-wait on each channel independently**.

- Each channel is either busy or idle.
- Each has its own 1-bit sequence number.
- The sender uses the lowest idle channel.
- No ordering is maintained between channels.
- No flow control is implied.

**Example:** 8 logical channels per ground link, requiring 3 bits for channel number + 1 bit for sequence number = 4 bits total header — the same as a sliding window protocol supporting 8 outstanding frames.

---

## 2.6 Multiaccess Networks (Ethernet)

Ethernet is the dominant LAN technology, based on **CSMA/CD** (Carrier Sense, Multiple Access with Collision Detect).

### 2.6.1 Physical Properties

**Classic Ethernet:**
- Coaxial cable segments, up to 500 m each.
- Hosts tap into the cable via **transceivers** connected to **network adaptors**.
- Multiple segments connected by **repeaters** (which regenerate digital signals). Maximum 4 repeaters between any pair of hosts → 2500 m total reach.
- Any signal is **broadcast** over the entire network. All hosts on a segment (and connected segments) are in the same **collision domain**.

**Modern Ethernet:**
- Twisted pair copper (Cat 5/6) or optical fiber.
- Mostly **point-to-point** links connecting hosts to switches.
- Speeds: 100 Mbps, 1 Gbps, 10 Gbps, 40 Gbps, 100 Gbps.
- Uses 4B/5B or 8B/10B encoding (classic Ethernet used Manchester).

### 2.6.2 Access Protocol

#### Frame Format

| Field | Size | Purpose |
|-------|------|---------|
| **Preamble** | 64 bits | Alternating 0s and 1s for receiver synchronization |
| **Destination address** | 48 bits | Recipient's MAC address |
| **Source address** | 48 bits | Sender's MAC address |
| **Packet type** | 16 bits | Demux key — identifies higher-level protocol |
| **Data** | 46–1500 bytes | Payload (padded to minimum 46 bytes) |
| **CRC** | 32 bits | Error detection |

Minimum frame size = **64 bytes** (512 bits) — this is critical for collision detection (explained below).

#### Ethernet Addresses (MAC Addresses)

- 48-bit globally unique address, burned into the adaptor's ROM.
- Human-readable format: six hex bytes separated by colons (e.g., `8:0:2b:e4:b1:2`).
- First 24 bits = manufacturer prefix (assigned by IEEE); remaining 24 bits = unique suffix (assigned by manufacturer).

**Address types:**
- **Unicast** — identifies a single adaptor.
- **Broadcast** (`FF:FF:FF:FF:FF:FF`) — all adaptors accept.
- **Multicast** (first bit = 1, not all 1s) — a subset of adaptors accept.
- **Promiscuous mode** — adaptor accepts all frames (used for monitoring/debugging).

#### The CSMA/CD Transmitter Algorithm

1. **If the line is idle:** Transmit immediately. (Ethernet is **1-persistent**: transmit with probability 1 when idle.)
2. **If the line is busy:** Wait for idle, then transmit immediately. (All adaptors wait 9.6 µs inter-frame gap after each frame.)
3. **If a collision is detected:** Send a **32-bit jamming sequence**, then stop.
4. **Back off:** Wait a random time (exponential backoff), then retry.

#### Why Minimum Frame Size = 512 Bits

The worst-case collision scenario:
- Host A starts transmitting at time *t*.
- A's signal takes one link latency *d* to reach host B at the far end.
- Host B, seeing an idle line, starts transmitting just before A's signal arrives (at time *t + d*).
- The collision is detected by B immediately, but A does not learn of the collision until B's signal propagates back — at time *t + 2d*.

Therefore, **A must still be transmitting at time *t + 2d*** to detect the collision. For a maximally configured Ethernet (2500 m, 4 repeaters), the round-trip delay is **51.2 µs**. At 10 Mbps, this corresponds to **512 bits** — hence the minimum frame size of 64 bytes.

#### Exponential Backoff

After the *k*th collision, the adaptor randomly selects a delay from {0, 1, ..., 2^min(k,10) − 1} × 51.2 µs:

| Collision # | Range of k | Maximum Delay |
|-------------|-----------|---------------|
| 1st | {0, 1} | 51.2 µs |
| 2nd | {0, 1, 2, 3} | 153.6 µs |
| 3rd | {0, ..., 7} | 358.4 µs |
| *n*th (n ≤ 10) | {0, ..., 2ⁿ−1} | (2ⁿ−1) × 51.2 µs |
| 11th–16th | {0, ..., 1023} | Capped at k=10 |
| After 16 | Give up | Report transmit error |

#### Capture Effect

When two stations compete repeatedly, the one with fewer collisions has a smaller backoff range and is more likely to win subsequent contention — effectively "capturing" the channel and starving the other station.

#### p-Persistent CSMA

A generalization: instead of transmitting with probability 1 when idle (1-persistent), transmit with probability *p* and defer with probability *1−p*. This reduces collision probability when multiple stations are waiting for a busy line to become idle. Ethernet uses the 1-persistent approach, which has been highly effective in practice.

### 2.6.3 Longevity of Ethernet

Ethernet has dominated for over 30 years because of:
1. **Simplicity of administration** — no routing tables, easy to add hosts.
2. **Low cost** — cheap cable/fiber, adaptor-only cost per host.
3. **Incremental deployability** — switch-based and shared-media Ethernets can coexist.
4. **Backward compatibility** — modern gigabit/10G Ethernet is compatible with the original standard.

---

## 2.7 Wireless Networks

Wireless links share many properties with wired links (bit errors, framing, reliability) but introduce unique challenges:
- **Higher error rates** due to unpredictable noise.
- **Power constraints** — mobile devices have limited batteries.
- **Inherently multiaccess** — radio transmissions reach all nearby receivers.
- **Eavesdropping** — signals travel through the air and can be intercepted.

### 2.7.1 Basic Issues

#### Spectrum Sharing

Wireless links share the electromagnetic spectrum, divided by **frequency** and **space**:
- Government agencies (e.g., FCC) allocate frequency bands to specific uses.
- Some bands are **licensed** (cellular); others are **license-exempt** (Wi-Fi, Bluetooth).
- License-exempt devices are power-limited to reduce interference range.

#### Spread Spectrum

Two key techniques for sharing spectrum:

**1. Frequency Hopping**
- Transmit on a random sequence of frequencies, generated by a pseudorandom number generator.
- Receiver uses the same algorithm with the same seed to hop in sync.
- Reduces interference: unlikely two signals share the same frequency for more than an isolated bit.

**2. Direct Sequence**
- Each data bit is XORed with *n* random bits (the **chipping code**), spreading the signal over a frequency band *n* times wider.
- Provides redundancy: if some bits are damaged by interference, the original can be recovered.
- The "spreading" reduces the power per unit bandwidth, making it more resistant to narrowband interference.

#### Network Topologies

**Base Station Model:**
- A stationary base station with a wired backhaul connects to mobile client devices.
- All communication routes through the base station — client devices do not communicate directly.
- Three levels of mobility: (1) no mobility, (2) within range of one base, (3) between bases (handoff).

**Mesh / Ad Hoc Model:**
- All nodes are peers; messages are forwarded hop-by-hop.
- Extends range beyond a single radio.
- Offers fault tolerance through multiple routes.
- Disadvantage: requires more sophisticated (and power-hungry) hardware/software per node.

### 2.7.2 802.11 / Wi-Fi

802.11 is designed for LANs (homes, offices, campuses). Like Ethernet, its challenge is mediating access to a shared medium.

#### Physical Properties

Multiple standards with increasing data rates:

| Standard | Frequency | Max Data Rate | Modulation |
|----------|-----------|---------------|-----------|
| 802.11 (original) | 2.4 GHz | 2 Mbps | FHSS or DSSS |
| 802.11b | 2.4 GHz | 11 Mbps | DSSS variant |
| 802.11a | 5 GHz | 54 Mbps | OFDM |
| 802.11g | 2.4 GHz | 54 Mbps | OFDM |
| 802.11n | 2.4/5 GHz | 150+ Mbps | OFDM + MIMO |
| 802.11ac | 5 GHz | 450+ Mbps | OFDM + MIMO |
| 802.11ax | 2.4/5 GHz | 1+ Gbps | Advanced OFDMA |

- Most products support multiple standards for compatibility.
- All standards support multiple bit rates (e.g., 802.11a: 6, 9, 12, 18, 24, 36, 48, 54 Mbps). Lower rates are more resilient to noise.
- **Rate adaptation** algorithms dynamically choose the optimal bit rate based on observed signal-to-noise ratio.

#### The Hidden Node Problem

Nodes A and C are both within range of B but **not of each other**. If both transmit to B simultaneously, their signals collide at B, but neither A nor C detects the collision.

#### The Exposed Node Problem

Node B is transmitting to A. Node C hears B's transmission and mistakenly concludes it cannot transmit. But C could safely transmit to node D (which is out of A's range) without interfering with B's transmission to A.

#### CSMA/CA (Collision Avoidance)

802.11 uses **CSMA/CA** instead of Ethernet's CSMA/CD because wireless nodes generally **cannot transmit and receive simultaneously** (the transmitter's power swamps the receiver circuitry).

**Basic mechanism:**
1. Listen before transmitting (Carrier Sense).
2. If idle, transmit the frame.
3. The receiver sends an **explicit ACK** if the frame was received correctly (CRC passed).
4. If no ACK arrives, the sender assumes a collision occurred and retries with exponential backoff.

**RTS-CTS (Ready-to-Send / Clear-to-Send):**
An optional mechanism to address the hidden node problem:
1. Sender transmits a short **RTS** frame to the intended receiver.
2. Receiver replies with a short **CTS** frame (audible to hidden nodes).
3. The CTS tells all nodes within range to defer for the duration specified in the RTS/CTS.
4. After the RTS-CTS exchange, the sender transmits its data frame and waits for an ACK.
5. If two nodes' RTS frames collide, they detect the failure (no CTS received) and back off randomly.

#### Distribution System

Most 802.11 deployments use a **base station topology**, not mesh:
- **Access Points (APs)** are connected to a wired **distribution system** (e.g., Ethernet).
- Each AP operates on a different channel.
- Mobile nodes **associate** with one AP at a time.

**Scanning (AP selection):**
1. Node sends a **Probe** frame.
2. All APs in range reply with a **Probe Response**.
3. Node selects an AP and sends an **Association Request**.
4. AP replies with an **Association Response** (and notifies the old AP via the distribution system).

**Active scanning:** Node initiates by sending Probes.
**Passive scanning:** APs periodically broadcast **Beacon** frames advertising their capabilities; nodes can associate directly.

#### 802.11 Frame Format

The 802.11 frame has **four address fields** (instead of two):

| Field | Meaning |
|-------|---------|
| **Addr1** | Immediate destination |
| **Addr2** | Immediate source |
| **Addr3** | Intermediate destination (entered distribution system) |
| **Addr4** | Original source |

The interpretation depends on the **ToDS** and **FromDS** bits in the Control field. When both are set (frame traveled from wireless → distribution system → wireless), all four addresses are used.

#### Wireless Security

Because wireless signals travel through the air, they can be eavesdropped on and unauthorized users can inject traffic. Wireless networks therefore require access control and encryption mechanisms (e.g., WPA2).

### 2.7.3 Bluetooth (802.15.1)

Bluetooth fills the niche of **very short-range** communication between personal devices (phone to headset, laptop to keyboard).

**Key characteristics:**
- Operates in the license-exempt **2.45 GHz** band.
- Data rate: ~1–3 Mbps.
- Range: ~10 m.
- Very low power consumption (critical for battery-powered devices).
- Classified as a **Personal Area Network (PAN)**.

**Piconet architecture:**
- 1 **master** device + up to 7 **slave** devices.
- All communication is between master and slaves; slaves do not communicate directly.
- Uses **frequency hopping** across 79 channels, each used for 625 µs (the natural time slot).
- Master transmits in odd-numbered slots; slaves respond in even-numbered slots (only when requested by the master — no contention).
- Up to **255 parked** (inactive, low-power) devices in addition to the 7 active slaves.

**Related technologies:**
- **ZigBee (802.15.4)** — even lower power, lower bandwidth, cheaper. Designed for sensor networks where devices must last months/years on small batteries.

---

## 2.8 Access Networks

Access networks connect end users to the Internet via the "last mile." Two key technologies:

### 2.8.1 Passive Optical Network (PON)

PON delivers fiber-based broadband to homes and businesses.

**Architecture:**
- **Point-to-multipoint tree** structure.
- **OLT (Optical Line Terminal)** at the ISP's central office.
- **Passive splitters** fan out the signal — they forward optical signals without storing or processing frames (like optical repeaters).
- **ONUs (Optical Network Units)** at individual homes (up to 1024 per PON).

**Downstream (OLT → ONUs):**
- The OLT broadcasts on a downstream wavelength.
- Every frame reaches every ONU.
- Each ONU checks a unique identifier in the frame and keeps only frames addressed to it.
- **Encryption** prevents ONUs from eavesdropping on each other.

**Upstream (ONUs → OLT):**
- Time-division multiplexed on a separate upstream wavelength.
- The OLT centrally controls access by sending **grants** to individual ONUs, specifying when each can transmit.
- Different ONUs can receive different time shares (implementing different service levels).

**Speeds:**
- **G-PON (Gigabit-PON):** 2.25 Gbps (most widely deployed).
- **XGS-PON (10 Gigabit-PON):** Starting to be deployed.

### 2.8.2 Cellular Networks (4G/5G)

Cellular networks provide mobile broadband access.

#### Architecture

- **UE (User Equipment)** — the mobile device.
- **BBU (Broadband Base Unit)** — the base station (officially called eNodeB in 4G, gNB in 5G).
- **EPC (Evolved Packet Core)** — the core network at the central office, consisting of:
  - **MME (Mobility Management Entity)** — tracks and manages UE movement.
  - **HSS (Home Subscriber Server)** — subscriber database.
  - **S/PGW (Session/Packet Gateway)** — forwards packets between the RAN and the Internet.
- The geographic area served by a BBU's antenna is a **cell**. Cells overlap, and **handoffs** transfer a UE from one BBU to another based on signal strength.

#### OFDMA (Orthogonal Frequency-Division Multiple Access)

Unlike Wi-Fi's contention-based access, LTE uses **reservation-based** scheduling:

- The radio spectrum is divided into a 2D grid of **Resource Elements (REs)**:
  - **Frequency axis:** 12 subcarrier frequencies, each 15 kHz wide.
  - **Time axis:** one OFDMA symbol duration.
- Resource Elements are grouped into **Physical Resource Blocks (PRBs)** of 7×12 = 84 REs.
- A **scheduler** at the BBU allocates PRBs to different UEs based on:
  - **Channel Quality Indicator (CQI)** — feedback from each UE about signal quality (every 1-ms TTI).
  - The goal: maximize utilization while ensuring each UE gets its required service level.

**Modulation rates:** The number of bits per symbol depends on the scheme:
- 16-QAM → 4 bits/symbol
- 64-QAM → 6 bits/symbol

#### 5G Innovations

5G introduces additional flexibility:

- **Multiple waveforms** optimized for different frequency bands:

| Band | Carrier Frequency | Max Bandwidth | Subcarrier Spacings | Scheduling Interval |
|------|-------------------|---------------|--------------------|--------------------|
| Sub-1 GHz | < 1 GHz | 50 MHz | 15 kHz, 30 kHz | 0.5 ms, 0.25 ms |
| Mid-band | 1–6 GHz | 100 MHz | 15, 30, 60 kHz | 0.5, 0.25, 0.125 ms |
| mmWave | > 24 GHz | 400 MHz | 60, 120 kHz | 0.125 ms |

- The scheduler gains another degree of freedom: it can **dynamically adjust Resource Element sizes** by changing the waveform.
- Sub-1 GHz bands prioritize **range** (mobile broadband, massive IoT).
- Mid-band prioritizes **wider bandwidth** (mobile broadband, mission-critical applications).
- mmWave provides **superwide bandwidth** over short, line-of-sight coverage.

#### CBRS (Citizens Broadband Radio Service)

An unlicensed band at **3.5 GHz** in North America, enabling **private cellular networks** (e.g., on a university campus or factory floor). Three tiers of users share the spectrum: incumbent government users → licensed priority users → general public.

---

## Perspective: Race to the Edge

The access network is undergoing radical transformation:

- Historically built from **proprietary hardware appliances** (OLTs, BNGs, BBUs, EPCs) — expensive, complicated, slow to change.
- Transitioning to **open software on commodity hardware** — the **CORD** (Central Office Re-architectured as a Datacenter) initiative.

**Three competing visions for the network edge:**

1. **Cloud providers** push edge clusters into metro areas, treating the access network as a dumb bit-pipe.
2. **Network operators** rebuild the access network using cloud technology, colocating edge applications in the access infrastructure.
3. **Democratization** — open-source, commodity hardware makes it possible for **anyone** (enterprises, municipalities, rural communities) to deploy their own access-edge cloud.

Enabling factors: commoditized hardware/software, enterprise demand for private 5G, and newly available unlicensed spectrum (CBRS).

---

## Key Takeaways

1. **Five fundamental problems** must be solved to make a raw physical link useful: encoding, framing, error detection, reliable delivery, and media access control.
2. **Encoding** trades off efficiency against clock recovery. NRZ is simple but unreliable for long runs; Manchester guarantees transitions but halves efficiency; 4B/5B + NRZI balances both at 80% efficiency.
3. **Framing** can use sentinels with stuffing (PPP, HDLC) or fixed-size clock-based frames (SONET). Stuffing makes frame size data-dependent.
4. **CRC** provides strong error detection with minimal overhead. The Internet checksum is weaker but easy to implement in software and serves as a last line of defense.
5. **Stop-and-wait** is simple but wastes link capacity on high-latency links. **Sliding window** fills the pipe by allowing multiple outstanding frames.
6. **The sequence number space** must satisfy `MaxSeqNum ≥ SWS + RWS` to prevent ambiguity after sequence number wraparound.
7. **Ethernet's CSMA/CD** uses carrier sense, collision detection, and exponential backoff. The minimum frame size (64 bytes) is determined by the worst-case round-trip propagation delay.
8. **Wi-Fi's CSMA/CA** uses collision *avoidance* (instead of detection) with explicit ACKs and optional RTS/CTS to address the hidden node problem.
9. **PON** uses passive splitters for downstream broadcast and centrally controlled TDM for upstream access.
10. **Cellular (LTE/5G)** uses reservation-based OFDMA scheduling, with 5G introducing flexible waveforms for diverse bands and applications.
11. **Separation of concerns** is a crucial design principle: reliable delivery, ordered delivery, and flow control are distinct functions that should be implemented at the appropriate layer.
