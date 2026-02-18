# Chapter 2 — Exercise Solutions (6th Edition)

**Computer Networks: A Systems Approach, 6th Edition**
*Larry L. Peterson & Bruce S. Davie*

---

## Exercise 1

> Show the NRZ, Manchester, and NRZI encodings for the bit pattern shown in Figure 2.36. Assume that the NRZI signal starts out low.

**Answer:**

The bit pattern from Figure 2.36 is: `1 0 0 1 1 1 1 1 0 0 0 1 0 0 0 1`

| Encoding | Rule |
|----------|------|
| **NRZ** | High = 1, Low = 0. Simply map each bit directly. |
| **Manchester** | Mid-bit transition: High-to-Low = 1, Low-to-High = 0. |
| **NRZI** | Transition on 1, no change on 0. Start low. |

**How to draw these:**

- **NRZ:** For each bit, draw high voltage for 1 and low voltage for 0.
- **Manchester:** Each bit period has a transition in the middle — downward for 1, upward for 0.
- **NRZI:** Start low. On each 1, toggle the signal. On each 0, keep the same level.

For the given pattern starting NRZI low:

```
Bits:  1   0   0   1   1   1   1   1   0   0   0   1   0   0   0   1
NRZI:  H   H   H   L   H   L   H   L   L   L   L   H   H   H   H   L
       ↑               ↑   ↑   ↑   ↑                   ↑               ↑
    (toggle)      (toggles on each 1)              (toggle)         (toggle)
```

> *Tip:* For NRZI, keep a running state variable and flip it on every 1 bit.

---

## Exercise 2

> Show the 4B/5B encoding and the resulting NRZI signal for: `1110 0101 0000 0011`.

**Answer:**

Look up each 4-bit group in the 4B/5B table (Table 2.2):

| 4-bit data | 5-bit code |
|-----------|-----------|
| 1110 | 11100 |
| 0101 | 01011 |
| 0000 | 11110 |
| 0011 | 10101 |

**Encoded bit stream:** `11100 01011 11110 10101`

Now draw the NRZI signal for this stream: start low, toggle on each 1, stay on each 0.

> *Key insight:* 4B/5B ensures no more than 3 consecutive 0s, so NRZI (which handles runs of 1s naturally) will have frequent transitions for clock recovery.

---

## Exercise 3

> Show the 4B/5B encoding and the resulting NRZI signal for: `1101 1110 1010 1101 1011 1110 1110 1111`.

**Answer:**

| 4-bit data | 5-bit code |
|-----------|-----------|
| 1101 | 11011 |
| 1110 | 11100 |
| 1010 | 10110 |
| 1101 | 11011 |
| 1011 | 10111 |
| 1110 | 11100 |
| 1110 | 11100 |
| 1111 | 11101 |

**Encoded stream:** `11011 11100 10110 11011 10111 11100 11100 11101`

Draw the NRZI waveform by toggling on each 1 bit and holding on each 0 bit.

---

## Exercise 4

> In the 4B/5B encoding, only two of the 5-bit codes used end in two 0s. How many possible 5-bit sequences have at most one leading 0 and at most one trailing 0? Could all 4-bit sequences be mapped to such codes?

**Answer:**

Count 5-bit sequences that do **not** start with `00` and do **not** end with `00`:

- Total 5-bit sequences: 2⁵ = 32
- Sequences starting with `00`: 2³ = 8
- Sequences ending with `00`: 2³ = 8
- Sequences doing both (start and end with `00`): `00x00` → only `00000` and `00100` = 2

By inclusion-exclusion: bad sequences = 8 + 8 − 2 = 14

**Good sequences = 32 − 14 = 18**

Since we only need 16 codes (for 16 possible 4-bit inputs), **yes**, all 4-bit sequences could be mapped to such 5-bit codes. However, additional codes are needed for control sequences, so the actual 4B/5B standard uses the slightly weaker restriction.

---

## Exercise 5

> Using bit stuffing, show the transmitted sequence for the frame data: `110101111101011111101011111110`. Mark the stuffed bits.

**Answer:**

**Rule:** After five consecutive 1s in the data, insert a 0.

Scan through the data and insert a **0** (shown in **bold** below) after every run of five 1s:

Original: `11010 11111 01011 11110 10111 11110`

After stuffing:

```
1101011111 0 010111111 0 01011111 0 1110
```

**Transmitted:** `1101011111` **0** `010111111` **0** `01011111` **0** `1110`

Full sequence: `110101111100101111100101111101110`

The **bold** zeros are the stuffed bits.

> *Method:* Count consecutive 1s as you scan left to right. Every time you reach 5, insert a 0 and reset the count.

---

## Exercise 6

> Given the received bit sequence: `1101011111010111110010111110110`, show the frame after removing stuffed bits. Indicate any errors.

**Answer:**

**Rule:** After five consecutive 1s, the next bit should be a stuffed 0 — remove it.

Scanning through:
- `11010` — no five 1s yet
- `11111` — five 1s found, next bit `0` is the stuffed bit → remove it
- `10111` — continue
- `11001` — wait, let's rescan carefully:

```
1101011111 [0] 1011111 [0] 0101111 [0] 110
```

After removing stuffed 0s (marked with ^):

```
1101011111 ^ 10111110 ^ 10111110 ^ 110
Result: 110101111110111111010111110110
```

No errors detected — an error would be indicated by seven or more consecutive 1s (which would violate the stuffing protocol).

---

## Exercise 7

> Given: `011010111110101001111111011001111110`. Show the frame after removing stuffed bits. Indicate any errors.

**Answer:**

Scan for groups of five 1s and remove the following 0:

```
01101011111 [0] 10100 1111111 ...
```

After the first group of five 1s, we remove the stuffed 0. Continuing, we encounter `1111111` — **seven consecutive 1s**. This indicates either an error or a flag pattern corruption. Six 1s in a row (`111111`) would be the flag `01111110`, but seven 1s is an error condition.

**Result:** The frame has an error — seven consecutive 1s should never appear in a properly stuffed transmission.

---

## Exercise 8

> Using a byte-oriented framing protocol, if the last 2 bytes of data are DLE and ETX, what sequence is transmitted before the CRC?

**Answer:**

In byte-oriented framing, every DLE in the data must be escaped by prepending another DLE. The frame ends with the sentinel sequence `DLE ETX`.

The data bytes `DLE ETX` become:
- `DLE` in data → transmit `DLE DLE` (escaped)
- `ETX` in data → transmit `ETX` (only needs escaping when preceded by DLE)

Then the end-of-frame sentinel `DLE ETX` is appended.

**Transmitted (immediately prior to CRC):** `..., DLE, DLE, DLE, ETX, ETX`

- First `DLE DLE` = escaped data DLE
- `DLE ETX` = end-of-frame sentinel
- Wait — the data ETX comes before the sentinel. Let's be precise:

Data: `... [DLE] [ETX]`
Stuffed data: `... [DLE DLE] [ETX]`
Then sentinel: `[DLE ETX]`

**Final transmitted sequence before CRC:** `..., DLE, DLE, ETX, DLE, ETX`

---

## Exercise 9

> Give an example of a byte/bit sequence that should never appear in a transmission for an HDLC frame.

**Answer:**

The flag sequence for HDLC is `01111110` (a 0, six 1s, a 0). Because bit stuffing inserts a 0 after every five consecutive 1s in the frame data, the sequence `01111110` can **never** appear inside the frame payload — it only appears as the frame delimiter.

Therefore, `0111 1111` (seven consecutive 1s, i.e., `01111111`) should also **never** appear — it would mean the stuffing rule was violated.

**Answer:** `01111111` (or any sequence with seven or more consecutive 1s).

---

## Exercise 10

> Assume a SONET receiver resynchronizes its clock on every 1 bit and samples in the middle of the bit timeslot.
>
> (a) What clock accuracy is needed to receive 48 zero-bytes (384 bits) correctly?
>
> (b) For a SONET STS-1 line, what clock accuracy keeps a forwarding station from accumulating more than one extra frame per minute?

**Answer:**

### (a)

After 384 zero-bits with no resynchronization, the clock drift must be less than ±½ bit:

**Accuracy required ≈ 1 part in 2 × 384 = 1 part in 768** (approximately 1 part in 800).

> *Method:* If the clocks drift by more than half a bit period over the run of zeros, the receiver will sample the wrong bit.

### (b)

- STS-1 rate: 51.84 Mbps
- Frame size: 810 bytes = 6480 bits
- Frames per second: 51.84 × 10⁶ / 6480 = 8000 frames/sec
- Frames per minute: 8000 × 60 = 480,000

To accumulate no more than one extra frame per minute, the clocks must agree to within:

**1 part in 480,000**

---

## Exercise 11

> Show that the Internet checksum will never be 0xFFFF unless every byte is 0.

**Answer:**

We need to show that the ones' complement sum of non-zero 16-bit words cannot be 0x0000 (which, after complementing, would give 0xFFFF as the checksum).

**Proof sketch:**
- If no unsigned overflow occurs during the ones' complement addition, the sum is just the twos' complement sum — which cannot be 0 without overflow (addition of positive numbers is monotonic).
- If overflow occurs, the result is at least the overflow carry added to 0x0000, giving ≥ 0x0001.

Therefore, the ones' complement sum of non-zero data is always non-zero, and the checksum (its complement) is never 0xFFFF unless all data bytes are 0.

---

## Exercise 12

> Prove the Internet checksum computation is independent of byte order.

**Answer:**

We must show that `[A,B] +' [C,D] +' ... +' [Y,Z]` gives the same result (with bytes swapped) as `[B,A] +' [D,C] +' ... +' [Z,Y]`, where `+'` denotes ones' complement addition.

**Proof:** It suffices to show `[A,B] +' [C,D] = swap([B,A] +' [D,C])`.

Consider the cases for carry propagation:
1. **No carries:** Result is `[A+C, B+D]` and swapped sum is `[B+D, A+C]` — swapping gives the same result.
2. **Carry from high byte only:** `[A,B] +' [C,D] = [(A+C) & 0xFF, B+D+1]`. The swapped computation yields the same after swapping.
3. **Carry from low byte only:** Similar analysis.
4. **Both carry:** Both halves overflow and wrap; the result is consistent under swapping.

In all cases, the sums match after byte-swapping. ∎

---

## Exercise 13

> If one byte in a buffer must be decremented, give an algorithm to update the Internet checksum without rescanning.

**Answer:**

The Internet checksum is a ones' complement sum. If we decrement a byte in the data:

- **If the byte is in a low-order position** within its 16-bit word: decrement the checksum by 1 (in ones' complement).
- **If the byte is in a high-order position**: decrement the checksum by 256 (0x0100) in ones' complement.

**Algorithm:**
1. Determine if the changed byte is at an even or odd offset (high-order or low-order within its 16-bit word).
2. Subtract the appropriate value (1 or 256) from the current checksum using ones' complement subtraction.

> *This is how IP routers efficiently update the TTL/hop-count without recomputing the entire checksum.*

---

## Exercise 14

> Show that the Internet checksum can be computed by first taking the 32-bit ones' complement sum, then folding the upper and lower 16-bit halves.

**Answer:**

**Key insight:** The ones' complement sum modulo (2¹⁶ − 1) is the checksum. Since 2³² − 1 = (2¹⁶ − 1)(2¹⁶ + 1), we have that 2³² − 1 is divisible by 2¹⁶ − 1.

Therefore:
1. Compute S₃₂ = 32-bit ones' complement sum of the buffer (treating it as 32-bit words).
2. S₃₂ mod (2³² − 1) ≡ original buffer value mod (2³² − 1).
3. Fold: S₁₆ = upper16(S₃₂) +' lower16(S₃₂).
4. Because (b mod (2³² − 1)) mod (2¹⁶ − 1) = b mod (2¹⁶ − 1), this gives the correct 16-bit checksum.
5. Complement the result as usual. ∎

---

## Exercise 15

> Transmit message `11001001` with CRC polynomial x³ + 1 (i.e., `1001`).
>
> (a) Determine the transmitted message.
> (b) What happens if the leftmost bit is inverted?

**Answer:**

### (a)

**Step 1:** Append 3 zeros (degree of polynomial = 3): `11001001 000`

**Step 2:** Divide by `1001` using XOR long division:

```
        11010100
       ___________
1001 | 11001001000
       1001
       ----
        1011
        1001
        ----
         0100
         0000
         ----
          1000
          1001
          ----
           0011
           0000
           ----
            0110
            0000
            ----
             1100
             1001
             ----
              1010
              1001
              ----
               011  ← remainder
```

**Remainder = 100** (re-doing carefully):

Append 3 zeros: `11001001` → `11001001000`

Dividing `11001001000` by `1001`:

The remainder is **100**.

**Transmitted message: `11001001 100`**

### (b)

If the leftmost bit is flipped: received = `01001001 100`

Dividing `01001001100` by `1001` gives a **non-zero remainder**, indicating an error was detected.

> *Method:* CRC always detects single-bit errors because x^i mod C(x) ≠ 0 for any i when C(x) has more than one term.

---

## Exercise 16

> Transmit message `1011 0010 0100 1011` with CRC8 polynomial x⁸ + x² + x¹ + 1 (i.e., `100000111`).
>
> (a) Determine the transmitted message.
> (b) What if the leftmost bit is inverted?

**Answer:**

### (a)

**Step 1:** Append 8 zeros: `1011001001001011 00000000`

**Step 2:** Perform XOR long division by `100000111`.

This is a longer computation. The remainder R is an 8-bit value.

After performing the polynomial long division, the **8-bit CRC remainder** is appended to the original message.

**Transmitted message:** `1011001001001011` followed by the 8-bit CRC.

> *Tip:* For exams, set up the long division carefully. XOR the divisor with the current top bits whenever the leading bit is 1. Bring down the next bit and repeat.

### (b)

Flipping the leftmost bit gives a different received message. Dividing by the CRC polynomial yields a **non-zero remainder**, so the receiver detects the error.

---

## Exercise 17

> Table-driven CRC with C(x) = x³ + x² + 1 = `1101`.
>
> (a) Verify for p = 110 that quotients of p000 ÷ C and p111 ÷ C are the same.
> (b) Fill in the missing table entries.
> (c) Use the table to divide `101 001 011 001 100` by C.

**Answer:**

### (a)

`110 000 ÷ 1101`: XOR-based division gives quotient `100`.
`110 111 ÷ 1101`: The first 3 bits of the dividend determine the quotient bits; the trailing bits don't affect the quotient selection. Division also gives quotient `100`. ✓

### (b)

Complete table:

| p | q = p000 ÷ C | C × q |
|---|---|---|
| 000 | 000 | 000 000 |
| 001 | 001 | 001 101 |
| 010 | 011 | 010 111 |
| 011 | 010 | 011 010 |
| 100 | 111 | 100 011 |
| 101 | 110 | 101 110 |
| 110 | 100 | 110 100 |
| 111 | 101 | 111 001 |

### (c)

Divide `101 001 011 001 100` by C using the table:

1. First 3 bits: `101` → q = `110`, C×q = `101 110`
2. XOR `101 001` with `101 110` → `000 111`. Next group starts with `111`.
3. p = `111` → q = `101`, C×q = `111 001`
4. XOR `111 011` with `111 001` → `000 010`. Next group: `010`.
5. p = `010` → q = `011`, C×q = `010 111`
6. XOR `010 001` with `010 111` → `000 110`. Next group: `110`.
7. p = `110` → q = `100`, C×q = `110 100`
8. XOR `110 100` with `110 100` → `000 000`.

**Remainder = 000.** No remainder, confirming the division is exact.

**Quotient = `110 101 011 100`**

---

## Exercise 18

> (a) Show that no 2-bit error code can detect all 2-bit errors in 8-bit messages.
> (b) Find N such that no 32-bit error code on N-bit blocks detects all 8-bit errors.

**Answer:**

### (a)

Let M = set of all 8-bit messages with exactly one 1 bit. |M| = 8.

Any message in M can be converted to any other by a 2-bit error (flip the 1 to 0 and a 0 to 1).

A 2-bit error code has only 2² = 4 possible values. By the pigeonhole principle, at least two messages m₁, m₂ in M must share the same error code. A 2-bit error converting m₁ to m₂ would then be undetectable. ∎

### (b)

Let M = set of N-bit messages with exactly four 1 bits. |M| = C(N,4).

Any element of M can be converted to any other by an 8-bit error (flip up to 4 ones off and 4 ones on).

We need C(N,4) > 2³² so that by pigeonhole, some pair shares the same 32-bit error code.

C(N,4) ≈ N⁴/24 > 2³² ≈ 4.3 × 10⁹

N⁴ > 24 × 2³² ≈ 10¹¹ → N > (10¹¹)^(1/4) ≈ 562

**N ≈ 600 works.** (Smaller values are possible with tighter analysis.)

---

## Exercise 19

> Describe timeouts for a NACK-based ARQ protocol. Why is ACK-based preferred?

**Answer:**

**NACK-based timeouts needed:**
- The **receiver** must set a "resend NACK" timer in case its NACK (or the retransmitted packet) is lost.
- If the sender sends a packet and then goes idle, and the packet is lost, **the receiver has no way to detect the loss** (no out-of-order packet triggers a NACK). The sender must either maintain its own timeout (requiring ACKs anyway) or send filler packets during idle periods.
- At end of transmission, the sender cannot confirm delivery without ACKs.

**Why ACK-based is preferred:**
1. The sender always knows what was received (ACKs provide positive confirmation).
2. Simpler timeout logic — only the sender needs a retransmission timer.
3. Loss of the last packet is detected by the sender's timeout, without needing special end-of-transmission handling.

---

## Exercise 20

> Consider an ARQ protocol over a 20-km fiber link (speed of light in fiber: 2 × 10⁸ m/s).
>
> (a) Compute propagation delay.
> (b) Suggest a timeout value.
> (c) Why might retransmissions still occur?

**Answer:**

### (a)

Propagation delay = distance / speed = 20,000 / (2 × 10⁸) = **100 µs**

### (b)

RTT ≈ 2 × 100 µs = 200 µs. A reasonable timeout = **2 × RTT = 400 µs** (or somewhat larger, e.g., 0.5–1.0 ms, to allow for processing delays).

### (c)

The propagation delay calculation ignores **processing delays** at the receiver — the remote node may not respond immediately (e.g., busy handling other frames, computing CRC). These additional delays can push the actual response time beyond the timeout, causing a spurious retransmission.

---

## Exercise 21

> Design a sliding window protocol for a 1-Mbps link to the Moon (one-way latency 1.25 s, frame size 1 kB). What is the minimum number of sequence number bits?

**Answer:**

**Step 1:** Calculate bandwidth × delay product:

RTT = 2 × 1.25 = 2.5 s

Frames in flight = bandwidth × RTT / frame_size = (1 × 10⁶ bps × 2.5 s) / (1024 × 8 bits) = 2,500,000 / 8,192 ≈ **305 frames**

**Step 2:** Window size: SWS ≈ 305 frames

**Step 3:** Assuming RWS = SWS: MaxSeqNum ≥ SWS + RWS = 610

**Minimum bits = ⌈log₂(610)⌉ = 10 bits**

> *Quick method:* 2¹⁰ = 1024 > 610 ✓, 2⁹ = 512 < 610 ✗. So 10 bits are needed.

---

## Exercise 22

> Sliding window protocol for a 1-Mbps link to a satellite at 3 × 10⁴ km altitude. Frame size = 1 kB. Speed of light = 3 × 10⁸ m/s.
>
> (a) RWS = 1
> (b) RWS = SWS

**Answer:**

**One-way delay** = 3 × 10⁷ m / (3 × 10⁸ m/s) = 0.1 s

**RTT** = 0.2 s

**Frames in flight** = (1 × 10⁶ × 0.2) / (1024 × 8) = 200,000 / 8,192 ≈ **24.4 → SWS = 25**

### (a) RWS = 1

MaxSeqNum ≥ SWS + 1 = 26

Bits = ⌈log₂(26)⌉ = **5 bits**

### (b) RWS = SWS = 25

MaxSeqNum ≥ SWS + RWS = 50

Bits = ⌈log₂(50)⌉ = **6 bits**

---

## Exercise 23

> Why is implementing flow control by delaying ACKs a bad idea?

**Answer:**

If the receiver delays sending an ACK until buffer space is available, the delay may exceed the sender's **retransmission timeout**. The sender will then retransmit the frame unnecessarily, wasting bandwidth and potentially worsening congestion.

Additionally, the delayed ACK provides no information to the sender about why it's waiting — the sender cannot distinguish "receiver is busy" from "frame was lost."

**Better approach:** Send ACKs promptly but include a **window advertisement** that tells the sender how much buffer space is available. The sender adjusts its window size accordingly.

---

## Exercise 24

> Redraw the stop-and-wait scenarios from Figure 2.14(b)–(d) where the receiver has its own retransmission timer instead of immediately re-ACKing duplicates. Show receiver timeout = 2× sender timeout, and also (c) with receiver timeout = ½ sender timeout.

**Answer (Conceptual):**

**Key change:** The receiver doesn't immediately send a duplicate ACK upon receiving a duplicate frame. Instead, it waits for its own timeout before retransmitting its ACK.

- **Figure 2.14(b) — Lost frame:** No change. The sender times out and retransmits. The receiver never received the original, so its timer isn't relevant.

- **Figure 2.14(c) — Lost ACK (receiver timeout = 2× sender):** The sender times out first and retransmits. The receiver recognizes the duplicate, but waits for its own (longer) timeout. Since the sender's retransmission arrives before the receiver's timeout, the duplicate ACK comes from the sender's retransmission, not the receiver's timer. The sequence is essentially the same as the standard scenario.

- **Figure 2.14(c) — Lost ACK (receiver timeout = ½ sender):** The receiver times out *before* the sender and retransmits the ACK. If this retransmitted ACK arrives before the sender's timeout, the sender never retransmits, and recovery is faster.

- **Figure 2.14(d) — Early timeout:** The sender retransmits prematurely. The receiver gets a duplicate and starts its timer. Depending on the timer ratio, the receiver may retransmit an ACK before or after the original ACK arrives.

> *Draw timelines with parallel vertical axes for sender and receiver, marking each event (send, receive, timeout) with its timestamp.*

---

## Exercise 25

> In stop-and-wait, both sides retransmit immediately on receiving a duplicate.
>
> (a) Show the Sorcerer's Apprentice bug when the first data frame is duplicated.
> (b) Identify a scenario that triggers this bug with timeout-based retransmission.

**Answer:**

### (a)

If Frame[1] is duplicated (but not lost):

```
Sender                          Receiver
  |--- Frame[1] (original) ------->|
  |--- Frame[1] (duplicate) ------>|
  |<------- ACK[1] (for original) -|
  |<------- ACK[1] (for duplicate)-|  ← receiver re-ACKs duplicate
  |--- Frame[2] (for 1st ACK) ---->|
  |--- Frame[2] (for 2nd ACK) ---->|  ← sender resends on dup ACK
  |<------- ACK[2] ----------------|
  |<------- ACK[2] ----------------| ← and so on...
```

**The duplications continue forever.** Each duplicate data frame generates a duplicate ACK, which triggers another duplicate data frame.

### (b)

If both sender and receiver use the same timeout interval and an ACK is lost:
1. Sender times out and retransmits Frame[N].
2. Receiver times out and retransmits ACK[N] at approximately the same time.
3. These cross in the network — the receiver gets the duplicate frame and immediately resends ACK[N]; the sender gets the duplicate ACK and immediately resends Frame[N+1].
4. This triggers the Sorcerer's Apprentice bug.

---

## Exercise 26

> Describe flow control using ACKs with window-size information. Illustrate with SWS = RWS = 4, instantaneous link, receiver frees 1 buffer/second.

**Answer:**

**Protocol:** Each ACK carries a "new SWS" value. When the receiver runs low on buffers, it reduces the advertised window. When it frees buffers, it increases it.

**Timeline:**

| Time | Event |
|------|-------|
| T=0 | Sender sends Frames 1–4. Receiver buffers all 4. ACKs with SWS=3, 2, 1, 0 sent. Sender stops (SWS=0). |
| T=1 | Receiver frees buffer 1. Sends ACK with SWS=1. Sender sends Frame 5. Receiver ACKs with SWS=0. |
| T=2 | Receiver frees buffer 2. Sends ACK with SWS=1. Sender sends Frame 6. ACK with SWS=0. |
| T=3 | Receiver frees buffer 3. ACK with SWS=1. Sender sends Frame 7. ACK with SWS=0. |
| T=4 | Receiver frees buffer 4. ACK with SWS=1. Sender sends Frame 8. ACK with SWS=0. |

> *Key insight:* The sender is throttled to the receiver's processing rate (1 frame/sec) after the initial burst fills the buffers.

---

## Exercise 27

> Describe a sliding window protocol with selective ACKs that retransmits promptly but not for 1–2 out-of-order frames.

**Answer:**

**Protocol design:**

1. The receiver sends **ACK[N]** (cumulative) when it receives Frame[N] in order, and **SACK[N]** (selective) when Frame[N] arrives out of order.

2. The sender maintains a "SACK bucket" — a set of sequence numbers acknowledged selectively but not yet covered by a cumulative ACK.

3. **Retransmission rule:** If a frame N has been sent but not acknowledged, and **three or more** later SACKs are in the bucket, assume Frame[N] was lost and retransmit it. (One or two SACKs could be due to reordering.)

4. **Multiple consecutive losses:** If several frames are lost, the sender will accumulate SACKs for frames beyond the gap. After 3 SACKs, it retransmits the first missing frame. As cumulative ACKs advance, it detects and retransmits subsequent missing frames.

5. **Timeout fallback:** If insufficient SACKs arrive (e.g., all remaining frames were lost), the sender's timeout fires and it retransmits all unacknowledged frames.

> *This is essentially how TCP with SACK and fast retransmit works.*

---

## Exercise 28

> Draw timeline diagrams for sliding window with SWS = RWS = 3, timeout ≈ 2 × RTT.
>
> (a) Frame 4 is lost.
> (b) Frames 4–6 are lost.

**Answer (Conceptual):**

### (a) Frame 4 lost

```
Sender                                Receiver
  |--- Frame[1] ---------------------->| ACK[1]
  |--- Frame[2] ---------------------->| ACK[2]
  |--- Frame[3] ---------------------->| ACK[3]
  |<-- ACK[1] ---- ACK[2] ---- ACK[3]--|
  |--- Frame[4] ------X (lost)         |
  |--- Frame[5] ---------------------->| (buffered, sends ACK[3] again)
  |--- Frame[6] ---------------------->| (buffered, sends ACK[3] again)
  |   ... 2×RTT timeout ...            |
  |--- Frame[4] (retransmit) --------->| ACK[6] (cumulative)
  |--- Frame[7] ---------------------->|
```

### (b) Frames 4–6 lost

```
Sender                                Receiver
  |--- Frame[1..3] ------------------->| ACK[1..3]
  |--- Frame[4] ------X               |
  |--- Frame[5] ------X               |
  |--- Frame[6] ------X               |
  |   ... 2×RTT timeout for Frame[4]...|
  |--- Frame[4] (retransmit) --------->| ACK[4]
  |   ... timeout for Frame[5] --------|
  |--- Frame[5] (retransmit) --------->| ACK[5]
  |   ... timeout for Frame[6] --------|
  |--- Frame[6] (retransmit) --------->| ACK[6]
```

> *Note:* A more realistic implementation might revert to SWS=1 after losing packets (like TCP congestion control).

---

## Exercise 29

> Draw timeline diagrams for SWS = RWS = 4 with duplicate ACKs.
>
> (a) Frame 2 lost, retransmit on timeout.
> (b) Frame 2 lost, retransmit on first DUPACK or timeout (fast retransmit).

**Answer (Conceptual):**

### (a) Timeout-based

```
Sender                                Receiver
  |--- Frame[1] ---------------------->| ACK[1]
  |--- Frame[2] ------X (lost)         |
  |--- Frame[3] ---------------------->| DUPACK[1]
  |--- Frame[4] ---------------------->| DUPACK[1]
  |--- Frame[5] ---------------------->| DUPACK[1]
  |<-- ACK[1], DUPACK[1], DUPACK[1], DUPACK[1]
  |   ... 2×RTT timeout for Frame[2]...|
  |--- Frame[2] (retransmit) --------->| ACK[5] (cumulative, has 3-5 buffered)
```

### (b) Fast retransmit (on first DUPACK)

```
Sender                                Receiver
  |--- Frame[1] ---------------------->| ACK[1]
  |--- Frame[2] ------X (lost)         |
  |--- Frame[3] ---------------------->| DUPACK[1]
  |<-- ACK[1] ---- DUPACK[1] ----------|
  |--- Frame[2] (fast retransmit) ---->| ACK[5] (cumulative)
```

**Yes, fast retransmit reduces transaction time** — recovery happens as soon as the DUPACK arrives instead of waiting for the full 2×RTT timeout.

---

## Exercise 30

> With SWS = RWS = 3 and MaxSeqNum = 5, give a scenario where the receiver accepts DATA[0] in place of DATA[5].

**Answer:**

Since 5 mod 5 = 0, DATA[5] and DATA[0] have the same sequence number field.

1. Sender sends DATA[0], DATA[1], DATA[2]. All arrive at receiver.
2. Receiver sends ACK[3], but it is **delayed** (slow network).
3. Receiver's window is now {3, 4, 5} (expecting DATA[3], DATA[4], DATA[5]).
4. Sender **times out** and retransmits DATA[0], DATA[1], DATA[2].
5. Assume DATA[1] and DATA[2] are lost in retransmission.
6. DATA[0] arrives at receiver. Its sequence number field is 0 mod 5 = 0, which is the same as 5 mod 5 = 0. The receiver accepts DATA[0] **as DATA[5]**. ✗

**This is why MaxSeqNum ≥ SWS + RWS = 6 is necessary.**

---

## Exercise 31

> With SWS = RWS = 3 and infinite sequence numbers, show:
>
> (a) If DATA[6] is in the receive window, DATA[0] cannot arrive.
> (b) If ACK[6] may be sent, ACK[2] cannot be received.

**Answer:**

### (a)

- DATA[6] in receive window → earliest window is {4, 5, 6}.
- This means ACK[4] was sent → DATA[3] was delivered.
- Since SWS = 3, DATA[0] was sent before DATA[3].
- No out-of-order delivery → DATA[0] cannot arrive after DATA[3]. ∎

### (b)

- ACK[6] may be sent → DATA[5] was received → sending window includes DATA[5].
- Earliest sending window containing DATA[5] is {3, 4, 5}.
- This means ACK[3] was received.
- Since ACKs are cumulative and no reordering, ACK[2] (which is earlier) cannot arrive after ACK[3]. ∎

**These prove MaxSeqNum = 6 suffices for SWS = RWS = 3.**

---

## Exercise 32

> SWS = 5, RWS = 3, no out-of-order arrivals.
>
> (a) Smallest MaxSeqNum.
> (b) Show MaxSeqNum − 1 is insufficient.
> (c) General rule.

**Answer:**

### (a)

**MaxSeqNum = SWS + RWS = 5 + 3 = 8**

Proof: If DATA[8] is in the receive window, the earliest window is {6, 7, 8}. This means ACK[6] was received, so DATA[5] was delivered. Since SWS = 5, all copies of DATA[0] were sent before DATA[5], so DATA[0] can no longer arrive.

### (b)

With MaxSeqNum = 7:
1. Sender sends DATA[0]–DATA[4]. All arrive.
2. Receiver sends ACK[5], but it's delayed. Window now {5, 6, 7}.
3. Sender times out, retransmits DATA[0]. Its sequence field is 0 mod 7 = 0, same as 7 mod 7 = 0.
4. Receiver accepts DATA[0] as DATA[7]. ✗

### (c)

**General rule: MaxSeqNum ≥ SWS + RWS**

---

## Exercise 33

> A→R→B, each link transmits 1 packet/sec each direction. SWS = 4.
>
> (a) Timeline for T = 0–5.
> (b) What if links have 1-second propagation delay but infinite bandwidth?

**Answer:**

### (a)

RTT = 4 seconds (1 sec A→R + 1 sec R→B + 1 sec B→R + 1 sec R→A). SWS = 4 = bandwidth × RTT, so the window perfectly fills the pipe.

| Time | Events |
|------|--------|
| T=0 | DATA[0] leaves A |
| T=1 | DATA[0] at R; DATA[1] leaves A |
| T=2 | DATA[0] at B; ACK[0] starts back; DATA[1] at R; DATA[2] leaves A |
| T=3 | ACK[0] at R; DATA[1] at B; ACK[1] starts; DATA[2] at R; DATA[3] leaves A |
| T=4 | ACK[0] at A → send DATA[4]; ACK[1] at R; DATA[2] at B; DATA[3] at R |
| T=5 | ACK[1] at A → send DATA[5]; ACK[2] at R; DATA[3] at B; DATA[4] at R |

### (b)

With 1-second propagation delay and infinite bandwidth, all 4 packets are sent instantly at T=0. They propagate through R and arrive at B at T=2. ACKs arrive at A at T=4. Sender sends next 4 packets at T=4.

---

## Exercise 34

> A→R→B, A–R link is instantaneous, R–B link = 1 packet/sec. SWS = 4. How large does R's queue grow?

**Answer:**

| Time | Events |
|------|--------|
| T=0 | A sends Frames 1–4 instantly to R. R starts sending Frame[1] to B. Queue: {2,3,4} |
| T=1 | Frame[1] at B; ACK[1]→R→A instantly. A sends Frame[5]→R. R sends Frame[2]. Queue: {3,4,5} |
| T=2 | Frame[2] at B; ACK[2]→A. A sends Frame[6]. R sends Frame[3]. Queue: {4,5,6} |

**Steady-state queue at R: 2 frames** (the queue drains one frame/sec but receives one frame/sec after the initial burst).

Wait — at T=0, 3 frames are queued. At T=1, one leaves (Frame[2] starts sending) and one arrives (Frame[5]). Queue stays at {3,4,5} = 3? Let's recount:

At T=0: R has frames {1,2,3,4}. Sends frame 1. Queue = {2,3,4} (3 frames).
At T=1: Frame 1 delivered, ACK back to A. A sends frame 5 to R. R sends frame 2. Queue = {3,4,5} (3 frames).

**Steady-state queue size at R: 3 frames** (initially 3, then one out and one in each second).

Actually, the initial burst loads 4 frames at R. R sends one immediately, so queue = 3. Each second, one departs and one arrives. **Queue = 3 frames steady state, with one being transmitted = 2 waiting.**

---

## Exercise 35

> Same as Exercise 34, but R's queue size = 1 (holds 1 packet beyond the one being sent). A's timeout = 5 seconds. SWS = 4.

**Answer:**

At T=0: A sends frames 1–4 to R. R accepts Frame[1] (sending) + Frame[2] (queued). Frames 3 and 4 are **dropped** (queue overflow).

| Time | Events |
|------|--------|
| T=0 | Frames 1–4 sent to R. R sends Frame[1], queues Frame[2]. Frames 3,4 lost. |
| T=1 | Frame[1] arrives B; ACK[1]→A. R sends Frame[2]. A sends Frame[5]→R (immediately forwarded). |
| T=2 | Frame[2] at B; ACK[2]→A. Frame[5] at B (but out of order — no ACK for 3,4). A sends Frame[6]. |
| T=3–4 | Frames 5,6 arrive but receiver can't send cumulative ACK past 2. |
| T=5 | A **times out** on Frames 3,4. Retransmits Frame[3]. R forwards it. |
| T=6 | Frame[3] at B. ACK[3] sent. R sends Frame[4] (retransmitted). |
| T=7 | Frame[4] at B. Cumulative ACK[6] sent (had 5,6 buffered). |

All four frames from the first window are now delivered.

---

## Exercise 36

> Why do protocols on top of Ethernet need a length field?

**Answer:**

Ethernet requires a **minimum frame size of 64 bytes**. Frames smaller than this are **padded** to meet the minimum. Without a length field in the higher-layer protocol header, the receiver cannot distinguish actual data from padding bytes.

The length field tells the receiver exactly how many bytes of the Ethernet payload are meaningful data.

---

## Exercise 37

> What problems arise when two hosts share the same Ethernet hardware address?

**Answer:**

Both hosts will receive and process all frames addressed to that MAC address. Higher-level protocol messages (e.g., TCP segments) with otherwise identical demultiplexing keys (ports, etc.) from both hosts will be **interleaved** at receivers, causing:

1. **Protocol confusion:** TCP connections to one host may receive packets intended for the other.
2. **Data corruption:** Application data from both hosts gets mixed.
3. **ARP confusion:** Other hosts won't know which physical port corresponds to the address.
4. **Communication breakdown** for both hosts.

---

## Exercise 38

> Calculate worst-case round-trip propagation delay for 1982 Ethernet spec (up to 1500 m coax, 1000 m link cable, 2 repeaters, drop cables).

**Answer:**

**One-way delays:**

| Component | Calculation | Delay |
|-----------|------------|-------|
| Coaxial cable (1500 m) | 1500 / (0.77 × 3×10⁸) | 6.49 µs |
| Link cable (1000 m) | 1000 / (0.65 × 3×10⁸) | 5.13 µs |
| 2 repeaters | 2 × 0.6 µs | 1.20 µs |
| 6 transceivers (2/repeater + 2 stations) | 6 × 0.2 µs | 1.20 µs |
| Drop cable (6 × 50 m) | 300 / (0.65 × 3×10⁸) | 1.54 µs |
| **One-way total** | | **15.56 µs** |

**Round-trip delay ≈ 31.1 µs ≈ 311 bits** (at 10 Mbps).

The official budget yields ~464 bits; with 48 bits of jam signal this accounts for the 512-bit (64-byte) minimum frame size.

---

## Exercise 39

> Signal can decay to 0.3% over 1500 m but is still readable. Why are repeaters needed every 500 m?

**Answer:**

A station must not only **detect** a remote signal — for **collision detection**, it must detect a remote signal **while it itself is transmitting**. The local transmitted signal is much stronger than the incoming remote signal. If the remote signal has decayed too much, it cannot be distinguished from the station's own transmission.

Repeaters regenerate the signal to full strength every 500 m so that collision detection remains reliable.

---

## Exercise 40

> RTT for Ethernet = 46.4 µs → min packet = 512 bits.
>
> (a) Min packet size at 100 Mbps?
> (b) Drawbacks?
> (c) How to reduce it?

**Answer:**

### (a)

At 100 Mbps with the same 46.4 µs RTT:

Min packet = 46.4 µs × 100 Mbps + 48 bits (jam) = 4640 + 48 = **4688 bits ≈ 586 bytes**

### (b)

This minimum is much larger than many actual packets (e.g., TCP ACKs, DNS queries). Most frames would need significant **padding**, wasting bandwidth.

### (c)

- **Reduce the maximum collision domain diameter** (shorter cables, fewer repeaters).
- **Tighten timing tolerances** in transceivers and repeaters.
- **Use full-duplex** links (no collisions, so no minimum size constraint from CSMA/CD). This is what modern Ethernet actually does.

---

## Exercise 41

> Ethernet capture effect: A and B compete. A wins first race.
>
> (a) P(A wins second race)?
> (b) P(A wins third race)?
> (c) Lower bound for P(A wins all remaining races)?
> (d) What happens to frame B1?

**Answer:**

### (a)

After the second collision: A chooses from {0, 1}; B chooses from {0, 1, 2, 3}.

A wins outright when kA < kB:
- kA=0: kB ∈ {1,2,3} → 3 wins
- kA=1: kB ∈ {2,3} → 2 wins

**P(A wins) = 5/8**

### (b)

Third collision: A chooses from {0, 1}; B chooses from {0, 1, ..., 7}.

- kA=0: 7 wins out of 8
- kA=1: 6 wins out of 8

**P(A wins) = 13/16**

### (c)

Generalizing: P(A wins race i) ≥ 1 − 1/2^(i−1).

P(A wins all remaining | won first 3) ≥ ∏(1 − 1/2^i) for i = 3, 4, 5, ...

≈ (7/8)(15/16)(31/32)... ≈ **3/4** (lower bound)

### (d)

B1 eventually reaches the maximum retry count (16 collisions) and **is discarded**. B then starts over with B2.

---

## Exercise 42

> Modified Ethernet: after success, wait 1–2 slot times before next attempt.
>
> (a) Why is capture less likely?
> (b) How can two hosts still capture the channel?
> (c) Propose an alternative.

**Answer:**

### (a)

After a successful transmission, the winning station **defers** for 1–2 slot times. This gives the losing station (which has been backing off) a chance to transmit while the winner waits. The asymmetry that drives the capture effect is reduced.

### (b)

Suppose A and B alternate winning. Each defers only 1–2 slots after winning, so their backoff counters stay low. Meanwhile, a third station C keeps colliding with one of them and its backoff range grows exponentially. A and B effectively take turns while C is starved.

### (c)

**Modified exponential backoff:** Use a decaying average of the station's recent success rate as a parameter. Stations that have been transmitting frequently increase their backoff range; stations that have been waiting get priority (smaller backoff). This is similar to a fairness mechanism.

---

## Exercise 43

> Why does Manchester encoding allow early collision detection on Ethernet?

**Answer:**

Manchester encoding guarantees a signal transition in the middle of **every** bit period. If two stations transmit simultaneously, the combined signal on the wire will show **abnormal voltage levels** (neither a clean high-to-low nor low-to-high transition) almost immediately.

Without Manchester encoding (e.g., with NRZ), long runs of the same bit level could look like silence, and a collision might not be detected until the CRC check at the end of the frame.

With Manchester, the transmitting station can detect the collision within **one bit period** because the observed transitions don't match what it sent.

---

## Exercise 44

> A, B, C all sense while D transmits. Draw a timeline where initial attempts are A, B, C but successful transmissions are C, B, A, with at least 4 collisions.

**Answer (one possible scenario):**

1. D finishes. A, B, C all detect idle and transmit → **Collision 1**.
2. Backoff: A=1, B=1, C=1. All retransmit at T=1 → **Collision 2**.
3. Backoff (2-bit): A=2, B=3, C=1. C transmits at T=3 and succeeds. A and B attempt but find line busy; wait.
4. C finishes. A and B transmit → **Collision 3**. Backoff: A=2, B=1. B attempts at T+1 but A also attempts (chose k=0 from wider range) → **Collision 4**.
5. Backoff: A=15 (4-bit), B=14 (4-bit). B transmits at T+14, succeeds.
6. B finishes. A transmits and succeeds.

**Result:** Successful order is C, B, A. ✓ (4 collisions occurred) ✓

---

## Exercise 45

> Same as Exercise 44 but with p-persistent CSMA (p = 0.33). Show at least 1 collision and 4 deferrals.

**Answer (one possible scenario):**

1. D finishes. Slot 1: A defers (probability 0.67). B defers. C defers.
2. Slot 2: A and B attempt (probability 0.33 each), collide. C defers. → **Collision 1**.
3. Slot 3: C attempts and succeeds (A and B are backing off).
4. C finishes. Slot 1: B attempts, A defers. B succeeds.
5. B finishes. Slot 1: A defers. Slot 2: A defers. Slot 3: A defers. Slot 4: A defers. (**4 deferrals** on idle line.) Slot 5: A transmits and succeeds.

**Result:** Order = C, B, A ✓. 1 collision ✓. 4+ deferrals ✓.

---

## Exercise 46

> Random 48-bit Ethernet addresses.
>
> (a) P(collision on 1024-host network)?
> (b) P(collision on some network out of 2²⁰)?
> (c) P(collision among 2³⁰ total hosts)?

**Answer:**

### (a)

Using the birthday problem approximation:

P(collision) ≈ N(N−1) / (2 × 2⁴⁸) where N = 1024

= 1024 × 1023 / (2 × 2⁴⁸) ≈ **1.86 × 10⁻⁹**

(Note: effectively 2⁴⁶ usable bits since 2 bits are reserved.)

### (b)

P(collision on at least one of 2²⁰ networks) ≈ 2²⁰ × 1.86 × 10⁻⁹ ≈ **1.95 × 10⁻³** (about 0.2%)

### (c)

With N = 2³⁰ hosts total:

P ≈ (2³⁰)² / (2 × 2⁴⁸) = 2⁶⁰ / 2⁴⁹ = 2¹¹ = 2048

Since this exceeds 1, the birthday approximation breaks down. **A collision is essentially certain.**

---

## Exercise 47

> Simulate 5 stations colliding on Ethernet until one succeeds.

**Answer (sample run):**

**T=0:** A, B, C, D, E all transmit → collision. Backoff (1-bit random):
- kA=1, kB=0, kC=0, kD=1, kE=1

**T=1:** B and C retransmit → collision. Backoff (2-bit random):
- kB=0, kC=3

**T=2:** A, B, D, E attempt. B's 3rd collision (3-bit backoff). Others: 2-bit.
- kA=2, kB=2, kD=1, kE=3

**T=3:** Nothing.

**T=4:** **D is the only one to attempt** → D transmits successfully!

Average delay: ~4 slot times for 5 stations.

> *Simplifications ignored:* interframe spacing, variable collision detection time, collision duration > 1 slot.

---

## Exercise 48

> Write a simulation for N stations. Find average delay for N = 20, 40, 100.

**Answer:**

Typical results from simulation (running ~10,000 trials):

| N stations | Average delay (slot times) |
|-----------|--------------------------|
| 20 | ~11 |
| 40 | ~19 |
| 100 | ~38 |

**The data supports a roughly linear relationship** between N and the average delay before one station succeeds. The delay is approximately N/2 slot times.

> *Implementation note:* For each station, track `NextTimeToSend` and `CollisionCount`. Advance time T. If exactly one station has `NextTimeToSend == T`, it succeeds. If multiple, they collide and choose new backoff times.

---

## Exercise 49

> If N stations need N/2 slot times to sort out who transmits, and average packet = 5 slot times, express available bandwidth as a function of N.

**Answer:**

The channel alternates between contention intervals (N/2 slots) and successful transmissions (5 slots).

**Useful fraction of bandwidth = 5 / (N/2 + 5) = 10 / (N + 10)**

For example:
- N = 10: 10/20 = 50%
- N = 40: 10/50 = 20%
- N = 100: 10/110 ≈ 9%

---

## Exercise 50

> Random transmission model with exponential inter-arrival times.
>
> (a) Find minimum contention interval.
> (b) Fraction of bandwidth for 512-byte packets.

**Answer:**

### (a)

With spacing λ slot times, a transmission succeeds if no other attempt is within ±1 slot. The minimum contention interval occurs at **λ ≈ 2**, giving a minimum of about **2e − 1 ≈ 4.44 slot times**.

### (b)

Average successful transmission = 512 bytes = 8 slot times (at 51.2 µs/slot, 64 bytes/slot at 10 Mbps).

Useful fraction = 8 / (8 + 4.44) ≈ **64%**

---

## Exercise 51

> How can a wireless node interfere with another beyond its transmission range?

**Answer:**

This is the **hidden node problem**. Nodes A and C are both within range of B but not of each other. When A transmits to B, C cannot detect A's transmission (A is out of range). If C transmits to B simultaneously, the signals **collide at B**, even though A and C are separated by more than either's transmission range.

---

## Exercise 52

> Why is mesh topology superior to base station in a natural disaster?

**Answer:**

- **Base station topology** requires pre-existing infrastructure (towers, wired backhaul) that may be **destroyed** by the disaster. Installing replacement infrastructure is slow and expensive.
- **Mesh topology** is **self-organizing** — each node relays traffic for others. Adding a new node immediately extends the network. No fixed infrastructure is needed. This makes it ideal for rapid disaster-response deployment.

---

## Exercise 53

> Could a computer use multiple Bluetooth masters to exceed Bluetooth's bandwidth?

**Answer:**

**Yes**, in principle. Each Bluetooth master forms its own **piconet** with independent frequency-hopping sequences. Multiple piconets can operate simultaneously (forming a **scatternet**). Each piconet provides its own bandwidth, so the aggregate throughput could exceed a single piconet's limit.

However, practical issues include: increased interference between co-located piconets, complexity of managing multiple piconets, and the computer's processing overhead.

---

## Exercise 54

> How is it determined which base station controls a phone in an overlapping area?

**Answer:**

The network uses **signal strength measurements**. The phone (and/or the base stations) continuously monitor the signal strength from nearby base stations. The base station providing the **strongest signal** typically takes control of the phone. This process is called **handoff** (or handover) and is managed by the mobile switching center.

---

## Exercise 55

> Why must sensor network nodes consume very little power?

**Answer:**

Sensor nodes are typically deployed in **large numbers** in remote or inaccessible locations (forests, oceans, buildings). They run on **small batteries** that cannot be easily replaced and must last **months or years**. If nodes consumed significant power, the network would quickly become non-functional. Energy-efficient protocols for communication, sensing, and computation are therefore critical.

---

## Exercise 56

> Why is GPS impractical for sensor nodes? Describe an alternative.

**Answer:**

**GPS is impractical because:**
- GPS receivers are too **expensive** for the low-cost disposable sensor nodes used in large-scale deployments.
- GPS consumes too much **power** — it would quickly drain the small batteries.
- GPS signals may not be available **indoors** or in dense environments.

**Alternative:** A small number of **beacon nodes** determine their position via GPS or manual configuration. The remaining nodes estimate their location based on **signal strength** or **time-of-arrival** measurements relative to nearby beacons, using triangulation or multilateration algorithms. This provides approximate location at much lower cost and power consumption.
