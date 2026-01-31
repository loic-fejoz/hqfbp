# Hailing Frequency and Modulation Announcement Protocol based on HQFBP

**Status:** Draft / Complementary Specification for HQFBP  
**Date:** 2026-01-22

## 1. Introduction & Scope

This document defines the **Standard Descriptor Syntax** used within the **Hamradio Quick File Broadcasting Protocol (HQFBP)**.

Specifically, it defines the values used in the `Content-Encoding` (CBOR Key 5) header field of HQFBP Announcement Messages (as defined in HQFBP RFC Appendix C). These descriptors allow a transmitter to announce a complete, precise, and machine-readable description of the radio protocol stack—from the Application layer down to the Physical RF layer—enabling receivers to automatically configure their demodulation and decoding chains.

While designed for HQFBP, this descriptor syntax is generic and philosophies of **Explicit Decomposition** make it suitable for describing any amateur radio digital mode.

This concept matches the idea of **Cognitive Radio** as thought by Joseph Mitola. In his vision, a cognitive radio is an intelligent system aware of its environment and capable of dynamically adapting its transmission parameters—such as frequency, modulation, and error correction—to optimize communication. By treating the radio protocol stack as a set of explicitly declared, composable blocks rather than a fixed monolithic standard, this descriptor syntax enables receivers to automatically reconfigure their signal processing chains. This moves us closer to the ideal of a radio that can understand and adapt to new modes on the fly, rather than relying on static, pre-programmed configurations.

### 1.1. The Philosophy: Explicit Decomposition

Traditional amateur radio modes are often named as monolithic entities (e.g., "Packet Radio", "FT8", "Olivia"). This specification breaks these composite modes down into their constituent layers:

1.  **Application / Protocol Layer**: The high-level framing (e.g., HQFBP, AX.25).
2.  **Coding / FEC / Scrambling**: Data manipulation before modulation (e.g., Reed-Solomon, Scrambling).
3.  **Modulation**: The intrinsic conversion of bits to symbols (e.g., FSK, BPSK).
4.  **RF Transport**: The carrier method (e.g., FM, USB).
5.  **Physical Parameters**: Center frequency, bandwidth.

## 2. Descriptor Syntax & Registry

The announcement string is a sequential list (JSON/CBOR Array) of descriptors ordered from **Top (Application)** to **Bottom (Physical)**.

### 2.1. Layer 2 to 7: Application & Protocol Framing

Tags describing the high-level format or link-layer protocol.

| Descriptor | Description | Reference |
| :--- | :--- | :--- |
| `h` | HQFBP Protocol | [HQFBP RFC](rfc.md) |
| `aprs` | Automatic Packet Reporting System | APRS Spec 1.0 |
| `ax.25` | AX.25 Link Layer Framing | AX.25 2.2 |
| `ccsds` | CCSDS Space Data Link Protocols | TM/TC/AOS |
| `mobitex` | Mobitex Wireless Data Framing | - |
| `ft8` | WSJT-X FT8 Framing | WSJT-X |
| `js8` | JS8Call Framing | JS8Call |
| `winlink` | Winlink / RMS | - |
| `pocsag` | POCSAG Paging Protocol | - |
| `psk31` | PSK31 / Varicode Framing | - |
| `rtty` | RTTY / Baudot Framing | - |
| `usp` | U482C / Simple Protocol | GOMspace |
| `il2p` | IL2P Link Layer Protocol | - |
| `varac` | VaraC Chat & File Protocol | - |
| `m17` | M17 Digital Radio Protocol | - |
| `npr` | New Packet Radio Protocol | - |
| `dstar` | D-Star Digital Voice & Data | - |


### 2.2. Layer 1.5: Coding, FEC, Scrambling & Framing

Operations on the bit/symbol stream.

| Descriptor | Description | Parameters & Examples |
| :--- | :--- | :--- |
| `rs(n, k)` | Reed-Solomon FEC | Block `n`, Data `k`. `rs(255, 223)` |
| `ldpc(n, k)` | LDPC FEC | `ldpc(174, 87)` |
| `rq(l, m, r)` | RaptorQ (RFC 6330) | Source `l`, MTU `m`, Repair `r`. `rq(1024, 16, 0)` |
| `conv(k, r)` | Convolutional Coding | Constraint `k`, Rate `r`. `conv(7, 1/2)` |
| `golay` | Golay(24,12) | `golay` |
| `crc(n)` | Cyclic Redundancy Check | `n` bits. `crc(16)`, `crc(32)`, `crc(14)` |
| `scr(p, [i])` | Additive Scrambler | Poly `p` (hex/name), Init `i` (optional). `scr(g3ruh)`, `scr(ccsds)` |
| `asm(w)` | Attached Sync Marker | `w` (hex) or default CCSDS. `asm(0x1ACFFC1D)` |
| `interleave(d)`| Bit/Symbol Interleaving | Depth `d`. `interleave(16)` |
| `manchester` | Manchester Encoding | - |
| `nrzi` | Non-Return-to-Zero Inverted | - |
| `varicode` | Varicode Encoding | Used in PSK31 |
| `bstuff(n)` | Bit Stuffing | Insert 0 after `n` 1s. `bstuff`, `bstuff(5)` |
| `post_asm(w)` | Trailer Sync Marker | `w` (hex). `post_asm(0x7E)` |

### 2.3. Layer 1: Modulation / Modem

Intrinsic conversion of bits to baseband signals.

**Frequency Shift Keying (FSK)**
| Descriptor | Description | Parameters | Example |
| :--- | :--- | :--- | :--- |
| `afsk(b)` | Audio FSK | Baud `b` | `afsk(1200)` |
| `fsk(b, d)` | FSK (Baseband) | Baud `b`, Deviation `d` | `fsk(9600, 4.8k)` |
| `gfsk(b, d, [bt])`| Gaussian FSK | Baud `b`, Dev `d`, BT (opt) | `gfsk(9k6, 4k8, 0.5)` |
| `mfsk(n, r)` | **M-ary FSK** | Tones `n`, Symbol Rate `r` | `mfsk(16, 31.25)`, `mfsk(8, 6.25)` |
| `cpfsk(b, d)` | Continuous Phase FSK | Baud `b`, Dev `d` | `cpfsk(4800, 1.2k)` |
| `gmsk(b)` | Gaussian MSK | Baud `b` | `gmsk(4800)` |


**Phase Shift Keying (PSK)**
*Note: Differential PSK is represented as `diff, bpsk(...)`.*

| Descriptor | Description | Parameters | Example |
| :--- | :--- | :--- | :--- |
| `bpsk(b)` | Binary PSK | Baud `b` | `bpsk(1200)`, `bpsk(31.25)` |
| `qpsk(b)` | Quadrature PSK | Baud `b` | `qpsk(1000)` |
| `diff` | Differential Encoding | Applied before modulation | `diff, bpsk(31)` (DBPSK) |

**Spread Spectrum & OFDM**
| Descriptor | Description | Parameters | Example |
| :--- | :--- | :--- | :--- |
| `lora(sf, bw, cr)`| LoRa | SF, BW, CR (denominator) | `lora(12, 125k, 5)` |
| `ofdm(n, bw)` | OFDM | Carriers `n`, Total BW `bw` | `ofdm(16, 2000)` |

### 2.4. Layer 0: RF Transport

| Descriptor | Description | Parameters | Example |
| :--- | :--- | :--- | :--- |
| `fm` | Frequency Modulation | Optional BW | `fm(12.5k)` |
| `usb` | Upper Sideband | - | `usb` |
| `lsb` | Lower Sideband | - | `lsb` |
| `am` | Amplitude Modulation| - | `am` |
| `cw` | Continuous Wave | - | `cw` |

### 2.5. Physical Parameters (PHY)

| Descriptor | Description | Unit / Format | Example |
| :--- | :--- | :--- | :--- |
| `freq(f)` | Center Frequency | Hz (Integer preferred) | `freq(435100000)`, `freq(435M1)` |
| `off(o)` | Audio Offset | Hz | `off(1500)` |
| `bw(w)` | Occupied Bandwidth | Hz | `bw(25000)`, `bw(25k)` |

> [!NOTE]
> **Unit Notation**: 'k' = 1000 (kHz/kbps), 'M' = 1,000,000 (MHz).
> Integers are preferred for compact CBOR encoding.

---

## 3. Reference Tables

### 3.1. Standard Polynomials

| Name | Polynomial | Hex | Binary | Standard |
|------|------------|-----|--------|----------|
| `g3ruh` | $1+x^{12}+x^{17}$ | 0x21001 | ...100001000000000001 | Packet 9600 |
| `ccsds` | $1+x^3+x^5+x^7+x^8$ | 0x1A9 | 110101001 | CCSDS |
| `mobitex`| $1+x^{14}+x^{15}$ | 0xC001 | 1100000000000001 | Mobitex |
| `il2p` | $1+x^{14}+x^{15}$ | 0xC001 | 1100000000000001 | IL2P |

### 3.2. Common Mode Translation Table

| Common Name | Descriptor Deconstruction |
| :--- | :--- |
| **APRS (1200)** | `aprs, ax.25, crc16, bstuff, asm(0x7E), post_asm(0x7E), nrzi, afsk(1200), fm, freq(144M8)` |
| **Packet 9600** | `ax.25, scr(0x21001), nrzi, gfsk(9k6, 4k8), fm` |
| **FT8** | `ft8, ldpc(174,87), crc(14), mfsk(8, 6.25), usb` |
| **LoRa (EU)** | `lora(12, 125k, 5), freq(433M775)` |
| **PSK31** | `psk31, varicode, diff, bpsk(31.25), usb` |
| **CubeSat (GOM)**| `ax.25, golay, asm(0x1ACFFC1D), scr(ccsds), fsk(9k6), fm` |
| **AX.25 (Expl.)**| `chunk(256), ax.25, crc16, bstuff, asm(0x7E), post_asm(0x7E), nrzi, ...` |
| **FX.25 (Tag 01)**| `ax.25, rs(255,239), asm(0xB74DB7DF8A532F3E), post_asm(0x7E)` |
| **IL2P** | `il2p, rs(255,223), asm(0xF15E48)` |
| **VaraC (HF)** | `varac, h, turbo, ofdm(52, 37.5), bw(2400), usb` |
| **WA4DSY (56k)** | `ax.25, scr(0x21001), nrzi, fsk(56k, 28k), fm` |
| **D-Star DV (Fast)** | `dstar, scr(0x8003), gmsk(4800), fm` |
| **M17 (Text)** | `m17, conv(7, 1/2), interleave, mfsk(4, 4800), fm` |
| **NPR** | `npr, xor(3,1), scr(0x221), mfsk(4, 500k), fm` |




---

## 4. Integration with HQFBP

In HQFBP, these descriptors populate the `Content-Encoding` array.

### 4.1. The Boundary Marker ("h", -1)

The HQFBP stack is divided into two sections by the integer `-1` (or "h"):

1.  **Pre-boundary**: Encodings applied to the **File Payload** (e.g., source compression `gzip`, fountain codes `rq`).
2.  **Post-boundary**: Encodings applied to the **HQFBP Message** itself (e.g., link-level FEC `rs`, scrambling `scr`, modulation).

In the hailing context, it SHOULD be interpreted as the HQFBP framing.

### 4.2. Example 1: Satellite Downlink (Announcement)

**Context**: A satellite beaconing on 435.200 MHz.
**Announcement Message Payload**:

```json
{
  "0": 1001,
  "1": "SAT-1",
  "4": "application/vnd.hqfbp+cbor",
  "5": [
    "h",          // Protocol: HQFBP
    "rs(255,223)",    // FEC: Reed-Solomon
    "scr(ccsds)",     // Scrambler: CCSDS
    "bpsk(9600)",     // Modulation: BPSK 9k6
    "freq(435M2)"     // Tuning: 435.2 MHz
  ]
}
```

**Interpretation**: To decode the *next* message (1001), the ground station must: Tune 435.2MHz → Demod BPSK 9600 → Descramble CCSDS → Decode RS → Parse CBOR.

### 4.3. Example 2: HF File Transfer (Hailing)

**Context**: Reviewing an active hail on 14.074 MHz for a large file transfer.

```json
{
  "0": 2001,
  "1": "CALLSIGN",
  "4": "application/vnd.hqfbp+cbor",
  "5": [
    "gzip",                // Source Compression
    "rq(1024,128,0)",      // Fountain Code (Pre-boundary)
    "h"                    // HQFBP Framing
    "interleave(4)",       // Bit Interleaving
    "conv(7, 1/2)",        // Convolutional Coding
    "ofdm(18, 500)",       // Robust OFDM
    "freq(14M075)"
  ]
}
```

## 5. Operational Recommendations

### 5.1. Suggested Hailing Frequencies

To maximize discoverability, announcements should be made on known coordination channels.

This is not yet part of the specification.

### 5.2. Regulatory Compliance

*   **Callsign**: All HQFBP announcements **MUST** include the `Src-Callsign` (Key 1) to satisfy identification regulations.
*   **Bandwidth**: Announced `bw(),freq()` must comply with the band plan of the target frequency. Also the resulting $(freq-bw, freq+bw)$ be within the authorized band plan.