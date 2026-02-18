# Chapter 2 — Key Concepts & Notes Summary

**Computer Networks: A Systems Approach, 6th Edition**
*Larry L. Peterson & Bruce S. Davie*

> This summary covers the essential concepts from Chapter 2 (Direct Links) that are tested in the exercises. Use it as a study guide for your upcoming test.

---

## 1. Encoding

Encoding is the process of representing binary data as signals on a physical medium.

### 1.1 NRZ (Non-Return to Zero)

- **Rule:** High voltage = 1, Low voltage = 0.
- **Problem:** Long runs of 1s or 0s cause **baseline wander** (the receiver loses track of the average signal level) and **clock recovery** issues (no transitions to synchronize clocks).

### 1.2 NRZI (Non-Return to Zero Inverted)

- **Rule:** A **1** causes a transition (low→high or high→low); a **0** stays the same.
- **Solves:** Long runs of 1s (each 1 causes a transition).
- **Still has problems with:** Long runs of 0s (no transitions).

### 1.3 Manchester Encoding

- **Rule:** Each bit has a transition in the **middle** of its timeslot. 0 = low-to-high; 1 = high-to-low (or vice versa, depending on convention).
- **Advantage:** Clock is embedded — the receiver can synchronize on every bit.
- **Disadvantage:** Requires **twice the bandwidth** of NRZ (signal rate is doubled).
- **Used by:** Classic Ethernet (10 Mbps).

### 1.4 4B/5B Encoding

- **Idea:** Map every 4 data bits to a 5-bit code chosen so that no code has more than one leading 0 and no more than two trailing 0s.
- **Result:** No more than three consecutive 0s appear in the encoded stream.
- **Combined with NRZI:** NRZI handles runs of 1s well; 4B/5B limits runs of 0s. Together, they ensure sufficient transitions for clock recovery.
- **Efficiency:** 80% (4 useful bits per 5 transmitted).

### How to Solve Encoding Problems

1. Write out the bit pattern.
2. For NRZ: draw high for 1, low for 0.
3. For NRZI: start at the given level; toggle on each 1, stay for each 0.
4. For Manchester: draw a mid-bit transition for every bit (low-to-high for 0, high-to-low for 1, or vice versa).
5. For 4B/5B + NRZI: first map each 4-bit group to its 5-bit code (using Table 2.2 in the textbook), then draw the NRZI waveform for the resulting 5-bit stream.

---

## 2. Framing

Framing is how the receiver identifies the start and end of each frame in a stream of bits/bytes.

### 2.1 Bit Stuffing (HDLC-style)

- **Flag pattern:** `01111110` (six 1s between two 0s) marks frame boundaries.
- **Stuffing rule:** After every five consecutive 1s in the data, the sender inserts a **0**.
- **Destuffing:** The receiver removes any 0 that follows five consecutive 1s.
- **Error detection:** If the receiver sees **seven or more** consecutive 1s, an error has occurred.

### 2.2 Byte Stuffing (PPP-style / byte-oriented)

- **Sentinel bytes:** `DLE` (Data Link Escape) and `ETX` (End of Text) are used to delimit frames.
- **Stuffing rule:** If `DLE` appears in the data, insert an extra `DLE` before it. The sequence `DLE ETX` marks the end of a frame.
- **Example:** Data ending in `DLE ETX` is transmitted as `... DLE DLE DLE ETX ETX` (the first DLE is the stuff for the data DLE, the second DLE+ETX is the actual end-of-frame sentinel).

### 2.3 SONET Framing

- SONET uses fixed-size frames transmitted at precise intervals.
- Clock synchronization is maintained through **scrambling** (not bit stuffing).
- Important: The receiver resynchronizes its clock on every 1 bit; long runs of 0s require careful clock accuracy.

### 2.4 Clock Recovery and Accuracy

- If the receiver resyncs on every 1 bit, then during a run of N zero-bits, the clock must be accurate to within ±½ bit over N bits, i.e., accuracy of **1 part in 2N**.

---

## 3. Error Detection

### 3.1 Two-Dimensional (2D) Parity

- Arrange data in rows and columns; add a parity bit to each row and each column.
- **Detects:** All 1-bit errors, all 2-bit errors, all 3-bit errors.
- **Misses:** Some 4-bit errors (those forming the corners of a rectangle in the data layout).
- **Can correct:** All 1-bit errors (the bad row and column intersect at the corrupted bit).

### 3.2 Internet Checksum

- The Internet checksum is the **16-bit ones' complement sum** of 16-bit words, then complemented.
- **Properties:**
  - Independent of byte order (the sum can be computed in either byte order; only the final result needs byte-swapping).
  - Incrementally updatable: if one byte changes, the checksum can be adjusted without rescanning the entire buffer.
  - Can be computed using 32-bit arithmetic first, then folding the upper and lower 16 bits.
  - Never equals `0xFFFF` unless all bytes are 0 (the spec transmits `0x0000` as `0xFFFF`).

### 3.3 CRC (Cyclic Redundancy Check)

- **Method:** Treat the message as a polynomial M(x). Append r zero bits (where r = degree of the generator polynomial C(x)). Divide M(x)·xʳ by C(x) using **polynomial long division** (XOR-based, no carries). The remainder R(x) is the CRC.
- **Transmitted message:** M(x)·xʳ + R(x) (original message with CRC appended).
- **Receiver:** Divides the received message by C(x). If the remainder is 0, no error is detected.
- **Detection power:** A CRC with generator polynomial of degree r detects:
  - All single-bit errors
  - All double-bit errors (if C(x) has at least three terms)
  - All odd numbers of bit errors (if C(x) has (x+1) as a factor)
  - All burst errors of length ≤ r

### How to Solve CRC Problems

1. Write the message M as a binary string.
2. Append r zeros (r = degree of C(x)).
3. Perform XOR-based long division of the padded message by C.
4. The remainder (r bits) is the CRC.
5. Replace the appended zeros with the CRC to get the transmitted message.
6. To check: divide the received message by C. Remainder = 0 means no detected error.

### 3.4 Table-Driven CRC

- Instead of dividing bit by bit, divide k bits at a time using a lookup table.
- Build the table: for each k-bit pattern p, compute q = (p followed by k zeros) ÷ C, and store C × q.
- During division, use the first k bits of the current remainder to look up the subtraction value.

---

## 4. Reliable Delivery (ARQ)

ARQ (Automatic Repeat reQuest) protocols detect lost or corrupted frames and retransmit them.

### 4.1 Stop-and-Wait

- Sender sends one frame, then waits for an ACK.
- If the ACK doesn't arrive within a **timeout**, the sender retransmits.
- Needs a **1-bit sequence number** (0 or 1) to distinguish retransmissions from new frames.

### 4.2 ACK-Based vs. NACK-Based

- **ACK-based:** Receiver sends positive acknowledgment for each received frame. Sender retransmits on timeout.
- **NACK-based:** Receiver sends negative acknowledgment when it detects a missing frame. Problems:
  - If a packet is lost and the sender is idle, the receiver has no way to detect the loss.
  - At end of transmission, the sender has no confirmation that data arrived.
  - Requires additional timeouts to handle lost NACKs.
- **ACK-based is preferred** because it is simpler and more robust.

### 4.3 Timeouts

- Timeout should be slightly larger than the expected RTT.
- Too small: unnecessary retransmissions (wastes bandwidth).
- Too large: long waits before detecting loss (wastes time).
- Practical timeout ≈ 2 × RTT is common.

---

## 5. Sliding Window Protocol

The sliding window protocol allows multiple frames to be in flight simultaneously, improving link utilization.

### 5.1 Key Variables

| Variable | Meaning |
|----------|---------|
| **SWS** (Send Window Size) | Max frames the sender can have outstanding (unacknowledged) |
| **RWS** (Receive Window Size) | Max frames the receiver is willing to accept out of order |
| **LAR** (Last ACK Received) | Sequence number of the last frame acknowledged |
| **LFS** (Last Frame Sent) | Sequence number of the last frame sent |
| **NFE** (Next Frame Expected) | Next sequence number the receiver expects |
| **MaxSeqNum** | Size of the sequence number space |

### 5.2 Invariants

- **Sender:** LFS − LAR ≤ SWS
- **Receiver:** accepts frames with sequence numbers in [NFE, NFE + RWS − 1]

### 5.3 Sequence Number Space

The critical formula:

```
MaxSeqNum ≥ SWS + RWS
```

- **If RWS = 1** (Go-Back-N): MaxSeqNum ≥ SWS + 1.
- **If RWS = SWS**: MaxSeqNum ≥ 2 × SWS.

**Why this matters:** If the sequence number space is too small, old frames can be confused with new frames after ACKs are lost and the window slides.

### 5.4 Sizing the Window

For full link utilization, the window must cover the **bandwidth × delay product**:

```
SWS ≥ bandwidth × RTT / frame_size
```

**Minimum bits for sequence number** = ⌈log₂(MaxSeqNum)⌉

### 5.5 Window-Based Flow Control

- The receiver can advertise a smaller window in its ACKs to slow down the sender.
- **Problem with delaying ACKs for flow control:** If the receiver delays ACKs until buffer space is free, the sender may time out and retransmit unnecessarily.
- **Better approach:** ACKs carry an explicit window size field that the sender uses to adjust SWS.

### 5.6 Cumulative vs. Selective ACKs

- **Cumulative ACK[N]:** Acknowledges all frames up to and including N.
- **Selective ACK (SACK):** Acknowledges specific individual frames, allowing the sender to retransmit only what's actually missing.
- **Duplicate ACK (DUPACK):** Sent when an out-of-order frame arrives. Receiving multiple DUPACKs is a strong hint that a frame was lost.
- **Fast retransmit:** Retransmit after receiving 3 duplicate ACKs (used by TCP), without waiting for the timeout.

---

## 6. Stop-and-Wait Edge Cases

### 6.1 Sorcerer's Apprentice Bug

If both sender and receiver retransmit their last frame immediately upon receiving a duplicate, a single duplicated frame can cause **infinite duplications** that persist for the entire transmission.

### 6.2 Receiver Timeout

Instead of retransmitting ACKs immediately on receipt of a duplicate data frame, the receiver can use its own timer. This changes the timing diagrams and may introduce additional delays.

---

## 7. Ethernet (IEEE 802.3)

### 7.1 CSMA/CD (Carrier Sense Multiple Access / Collision Detection)

1. **Carrier Sense:** A station listens before transmitting. If the channel is busy, it waits.
2. **Collision Detection:** While transmitting, a station monitors the channel. If it detects a collision, it stops, sends a **jam signal** (48 bits), and backs off.
3. **Key constraint:** The minimum frame size must be large enough that the sender is still transmitting when a collision signal can propagate back. This is why Ethernet has a **minimum frame size of 64 bytes** (512 bits at 10 Mbps, matching the worst-case round-trip propagation delay of ~51.2 µs).

### 7.2 Exponential Backoff

After the kth collision, a station chooses a random delay from {0, 1, ..., 2^k − 1} × slot time (where slot time = 51.2 µs for 10 Mbps Ethernet).

- After 1st collision: choose from {0, 1}
- After 2nd collision: choose from {0, 1, 2, 3}
- After kth collision: choose from {0, ..., 2^k − 1} (capped at k = 10)
- After 16 collisions: give up.

### 7.3 Capture Effect

When two stations compete repeatedly, the one that has been transmitting successfully has a lower collision count and thus a smaller backoff range. It wins subsequent races with increasing probability, effectively "capturing" the channel and starving the other station.

### 7.4 Propagation Delay Budget (Classic Ethernet)

Sources of delay in classic coaxial Ethernet:
- Coaxial cable: propagation at 0.77c
- Link/drop cable: propagation at 0.65c
- Repeaters: ~0.6 µs each
- Transceivers: ~0.2 µs each

The total worst-case round-trip delay determines the minimum packet size.

### 7.5 Repeaters and Signal Decay

Repeaters regenerate the signal. They are needed every 500 m not because the signal is unreadable, but because **collision detection** requires the signal to be strong enough to detect a remote transmission while the local station is also transmitting.

### 7.6 Why Ethernet Frames Need a Length Field

Ethernet pads short frames to the minimum size (64 bytes). The length field in the protocol header above Ethernet tells the receiver how much of the frame is actual data vs. padding.

### 7.7 Shared MAC Addresses

Two hosts sharing the same Ethernet address will both receive frames intended for either one. Higher-level protocols will receive interleaved messages from both hosts, causing communication breakdown.

### 7.8 p-Persistent CSMA

Instead of transmitting immediately when the line goes idle, a station transmits with probability p and defers one slot time with probability (1 − p). This reduces collision probability when multiple stations are waiting.

---

## 8. Wireless Networking

### 8.1 Hidden Node Problem

Two nodes A and C are both within range of B but not of each other. A and C cannot detect each other's transmissions, so they may transmit simultaneously and collide at B.

**Solution:** RTS/CTS (Request-to-Send / Clear-to-Send) mechanism in 802.11. A short RTS is sent first; the receiver replies with CTS, which is heard by hidden nodes, who then defer.

### 8.2 Mesh vs. Base Station Topology

- **Base station:** Requires infrastructure (towers, wired backhaul). Vulnerable to disaster.
- **Mesh:** Each node relays for others. Self-organizing. Superior for disaster recovery because adding nodes extends the network without fixed infrastructure.

### 8.3 Bluetooth

- Bluetooth uses a master/slave piconet model (1 master, up to 7 active slaves).
- Multiple Bluetooth masters with separate piconets on the same device could work, as each piconet uses different frequency hopping sequences.

### 8.4 Cell Handoff

When a phone moves into an area where multiple base station cells overlap, the network determines which base station controls the phone based on **signal strength** — typically the base station with the strongest signal takes control.

### 8.5 Sensor Networks

- **Low power:** Sensor nodes run on small batteries and must last months/years. Energy-efficient protocols are critical.
- **GPS impractical:** Too expensive and power-hungry for most sensor nodes. Instead, a few **beacon nodes** know their location (via GPS or manual configuration), and other nodes estimate their position relative to these beacons.

---

## 9. Useful Formulas Quick Reference

| Formula | Use |
|---------|-----|
| Propagation delay = distance / speed | Time for signal to travel the link |
| Transmit delay = frame size / bandwidth | Time to push a frame onto the link |
| RTT ≈ 2 × propagation delay | Round-trip time |
| Bandwidth × delay = bandwidth × RTT | Data "in the pipe" — determines window size |
| MaxSeqNum ≥ SWS + RWS | Minimum sequence number space |
| Sequence bits = ⌈log₂(MaxSeqNum)⌉ | Bits needed for the sequence number field |
| CRC remainder = M·xʳ mod C(x) | CRC calculation |
| Slot time (Ethernet) = 51.2 µs | Base unit for exponential backoff |
| Min frame (Ethernet) = 512 bits | 64 bytes, matches worst-case RTT |

---

## 10. Problem-Solving Strategy for Chapter 2

1. **Encoding problems:** Draw the waveform step by step. Label each bit period. For 4B/5B, do the table lookup first, then draw NRZI.
2. **Framing problems:** Apply stuffing rules mechanically. Count consecutive 1s (for bit stuffing) or look for DLE (for byte stuffing). Mark stuffed bits clearly.
3. **Error detection:** For CRC, set up the long division carefully — XOR only, no borrows. For Internet checksum, work in 16-bit words and remember ones' complement addition (wrap carry back around).
4. **Sliding window:** Draw a timeline. Track LAR, LFS, NFE. Watch for the sequence number wraparound. Use MaxSeqNum ≥ SWS + RWS.
5. **Ethernet:** Calculate propagation delays first. Remember that exponential backoff doubles the range after each collision. The capture effect is about asymmetric collision counts.
6. **Wireless:** Think about what each node can and cannot hear. RTS/CTS helps but doesn't eliminate all problems.
