# Hailing Frequency and Modulation Announcement Protocol

This document defines a standard encoding description for announcing communication parameters (frequency, modulation, protocol stack) in amateur radio contexts.
The philosophy is **explicit decomposition**: composite modes (like "Olivia" or "G3RUH") are broken down into their constituent layers (Framing, Scrambling, Modulation, RF).

## Relationship to HQFBP

This document defines the descriptor syntax used in HQFBP announcement 
messages (see [HQFBP RFC](rfc.md) Appendix C). The descriptors are transmitted in 
the `Content-Encoding` field (CBOR key 5) of transparent announcement 
messages, enabling receivers to configure their protocol stack for 
subsequent data reception.

> [!NOTE]
> In the hailing context, `h` encoding simply means HQFBP framing, ie CBOR + data as per the [HQFBP RFC](rfc.md).

## Concept

The announcement string is a comma-separated list of **descriptors**, aka encodings in HQFBP.
The descriptors describe the stack from **Top to Bottom**.
For a valid hail, one should be able to reconstruct the receiving chain by piping these descriptors.

### Discovery and QSO Establishment

#### Passive Discovery (Satellite/Beacon)
1. Satellite/beacon transmits announcement on known frequency
2. Ground stations receive announcement
3. Stations configure receivers based on descriptor string
4. Stations listen for data on announced frequency

#### Active Hailing (Station-to-Station)
1. Station A transmits announcement on hailing channel
2. Station B receives announcement
3. Station B acknowledges (optional, if bidirectional)
4. Both stations QSY to working frequency
5. Data exchange begins


### Basic Syntax

```
layerN,layerN-1,...,layer0
```

## Standard Descriptors

### 1. Application / Protocol Layer (L7-L2)

These tags describe the high-level format, framing, or protocol stack.

| Descriptor | Description |
| :--- | :--- |
| `hqfbp` | HQFBP Protocol |
| `aprs` | Automatic Packet Reporting System |
| `ax.25` | AX.25 Link Layer Framing |
| `ccsds` | CCSDS Space Data Link Protocols (TM/TC/AOS) |
| `ao40` | AO-40 Protocol (FUNcube, etc.) |
| `usp` | U482C / Simple Protocol (GOMspace) |
| `mobitex` | Mobitex Wireless Data |
| `ft8` | WSJT-X FT8 Framing |
| `js8` | JS8Call Framing |
| `olivia` | Olivia Framing |
| `winlink` | Winlink / RMS |
| `pocsag` | POCSAG Paging Protocol |
| `psk31` | Varicode / PSK31 Framing |
| `rtty` | RTTY Baudot Framing |

### 2. Coding, FEC & Scrambling Layer (L1.5)

These tags describe operations on the bit/symbol stream before it hits the modem.
Standard FEC schemes match those defined in the HQFBP RFC.

| Descriptor | Description | Example |
| :--- | :--- | :--- |
| `scr(p, i)` | Additive Scrambler. `p` can be hex/bin (e.g. `0x1A9` or `0b110101001`), or string for table following table. Optional `i` (hex) sets LFSR seed. | `scr(g3ruh)`, `scr(0x1A9, 0xFF)` |
| `asm(w)` | Attached Sync Marker (eg CCSDS is 0x1ACFFC1D). Optional hex sync word `w`. | `asm`, `asm(0x1ACFFC1D)` |
| `rs(n, k)` | Reed-Solomon Error Correction. Block length `n`, data length `k`. | `rs(255, 223)` |
| `ldpc(n, k)` | Low-Density Parity-Check FEC. Block length `n`, data length `k`. | `ldpc(174, 87)` |
| `conv(k, r)` | Convolutional Coding (Constraint `k`, Rate `r`). | `conv(7, 1/2)` |
| `golay` | Golay(24,12) code (often used with ASM). | `golay` |
| `rq(l, m, r)` | RaptorQ (RFC 6330). Source length `l`, MTU `m`, repair `r`. | `rq(1024, 16, 0)` |
| `crc(n)` | `n`-bit Cyclic Redundancy Check. | `crc(16)`, `crc(32)` |
| `manchester` | Manchester encoding. | `manchester` |
| `nrzi` | Non-Return-to-Zero Inverted. | `nrzi` |
| `diff` | Differential Encoding (applied before modulation). | `diff, bpsk(b)` |

#### 2.1. **Complete Polynomial Reference Table**

| Name | Polynomial | Hex | Binary | Standard |
|------|------------|-----|--------|----------|
| `g3ruh` | $1+x^{12}+x^{17}$ | 0x21001 | 100001000000000001 | G3RUH 9600 |
| `ccsds` | $1+x^3+x^5+x^7+x^8$ | 0x1A9 | 110101001 | CCSDS 131.0-B-3 |
| `mobitex` | $1+x^{14}+x^{15}$ | 0xC001 | 1100000000000001 | Mobitex |

### 3. Modulation / Symbol Mapping (L1 Modem)

This defines how bits/symbols are converted to baseband signals (tones, phases).
This is the **intrinsic** nature of the digimode.

| Descriptor | Description | Parameters | Example |
| :--- | :--- | :--- | :--- |
| `afsk(b)` | Audio Frequency Shift Keying. | Baudrate `b` | `afsk(1200)` |
| `fsk(b, d)` | Frequency Shift Keying (Baseband). | Baudrate `b`, Deviation in Hz `d` | `fsk(9600, 4.8k)` |
| `cpfsk(b, d)` | Continuous-Phase FSK. | Baudrate `b`, Deviation in Hz `d` | `cpfsk(4800, 1.2k)` |
| `gfsk(b, d, bt)` | Gaussian FSK (Smoothed CPFSK). | Baudrate `b`, Deviation in Hz `d`, BT product `bt` | `gfsk(9600, 4.8k, 0.5)` |
| `bpsk(b)` | Binary Phase Shift Keying. | Baudrate `b` | `bpsk(31)`, `bpsk(1200)` |
| `diff,bpsk(b)` | Differential BPSK | Baudrate `b` | `diff, bpsk(1000)` |
| `qpsk(b)` | Quadrature PSK. | Baudrate `b` | `qpsk(31)` |
| `diff,qpsk(b)` | Differential QPSK. | Baudrate `b` | `diff, qpsk(1000)` |
| `mfsk(n, r)` | M-ary FSK. | Tones `n`, Symbol Rate `r` | `mfsk(16, 31.25)` |
| `ofdm(n, bw)` | Orthogonal FDM. | Carriers `n`, Bandwidth in Hz `bw` | `ofdm(16, 2000)` |

### 4. RF Transport Layer (L0)

This defines how the baseband signal is put onto the RF carrier.

| Descriptor | Description | Parameters | Example |
| :--- | :--- | :--- | :--- |
| `fm` | Frequency Modulation. | Deviation/BW (optional) | `fm`, `fm(12.5k)` |
| `usb` | Upper Sideband. | - | `usb` |
| `lsb` | Lower Sideband. | - | `lsb` |
| `am` | Amplitude Modulation. | - | `am` |
| `cw` | Continuous Wave. | - | `cw` |

### 5. Physical Parameters (PHY)

| Descriptor | Parameter | Example |
| :--- | :--- | :--- |
| `freq` | Center Frequency (Hz). | `freq(144M8)` |
| `off` | Audio center offset (Hz). | `off(1k5)` |
| `bw` | Bandwidth (Hz). | `bw(25k)` |

---

## Examples of Decomposition

### Classic APRS (VHF)

APRS is just the app. The stack is AX.25, using NRZI, AFSK modulation, over FM.

```
aprs,ax.25,nrzi,afsk(1200),fm(12k5),freq(144M8)
```

### 9600 Baud Packet (G3RUH)

"G3RUH" is a scrambler. The modulation is typically GFSK (or filtered FSK) with specific deviation.

```
aprs,ax.25,scr(g3ruh),nrzi,gfsk(9k6, 4k8),fm(25k),freq(437M5)
```

### FT8

FT8 uses LDPC(174,87), `crc(14)`, and 8-FSK modulation.

```
ft8,ldpc(174,87),crc(14),mfsk(8, 6.25),usb,freq(14M074),off(1k5)
```

### RTTY (Classic)

baudot code, 45 baud, 170 Hz shift FSK.
```
rtty,fsk(45, 170),usb,freq(14M08)
```

### PSK31

Varicode encoding, BPSK modulation at 31.25 baud.

```
psk31,varicode,bpsk(31.25),usb,freq(14M07),off(1k)
```

### Olivia (8/250)

Olivia MFSK submode (8 tones, 250 Hz bandwidth). All standard Olivia submodes use 31.25 baud.

```
olivia,mfsk(8, 31.25),usb,freq(14M07),off(1k)
```

### POCSAG (Paging)

POCSAG protocol, usually direct FSK (4.5kHz shift).

```
pocsag,fsk(1k2, 4k5),fm(25k),freq(439M9875)
```

### LoRa APRS (EU)

Standard LoRa-APRS tracker configuration.

```
aprs,ax.25,lora(sf=12, bw=125k, cr=4/5),freq(433M775)
```

### RSID (Reed-Solomon Identification)

The identification signal itself uses 16-tone MFSK.

```
rsid,mfsk(16, 10.766),usb,freq(14M07)
```

### gr-satellites: Generic CCSDS (BPSK)

Common downlink format. CCSDS frames, Reed-Solomon(223, 255), Scrambled, BPSK.

```
ccsds,rs(255,223),scr(ccsds),bpsk(9k6),usb,freq(435M1)
```

### gr-satellites: GOMspace AX100 (ASM+Golay)

Common on CubeSats.
```
ax.25,golay,asm(0x1ACFFC1D),scr(ccsds),fsk(9k6),fm,freq(437M425)
```
*Note: GOMspace often wraps AX.25 in ASM/Golay/Scrambler.*

### gr-satellites: Mobitex

```
mobitex,scr(0xC001, 0x7FFF),fsk(4k8),fm,freq(435M5)
```

### HQFBP High Speed (Example)

Using RS(255,223) FEC and G3RUH scrambling.

```
gzip,h,rs(255,223),scr(g3ruh),gfsk(19k2, 2k4),fm,freq(435M)
```

## Examples of Complete Announcement Exchanges

### Scenario 1: CubeSat Downlink
```
[Satellite FOSM-1 transmits on 435.200 MHz USB]

Announcement Message:
{
  "0": 1001,
  "1": "FOSM-1",
  "4": "application/vnd.hqfbp+cbor",
  "5": ["h", "rs(255,223)", "scr(ccsds)", "gfsk(9k6, 4k8)", "freq(435M2)"]
}

[Ground stations receive announcement, configure decoders]
[Satellite transmits data chunks using announced stack]
```

### Scenario 2: HF Digital File Transfer
```
[Station F4XYZ on 14.074 MHz USB]

Announcement (transmitted once):
{
  "0": 2001,
  "1": "F4XYZ",
  "2": "QST-0",
  "4": "application/vnd.hqfbp+cbor",
  "5": ["hqfbp", "gzip", -1, "rq(1024,256,10)", "bpsk(125)", "usb", "freq(14.074)", "off(1500)"]
}

[Listening stations configure for RaptorQ fountain code reception]
[F4XYZ transmits file chunks with RaptorQ encoding]
```

## Regulatory Compliance

### Station Identification

- Announcement messages MUST include `Src-Callsign` (CBOR key 1)
- Callsign format: `CALL-SSID` (e.g., `F4XYZ-7`)
- Identification interval: per local regulations (typically 10 minutes)