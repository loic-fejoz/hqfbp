# Hamradio Quick File Broadcasting Protocol (HQFBP) Specification

## Status of This Memo

| Status | Draft |
| :--- | :--- |
| Version | 0.2 |

## 1. Introduction

The Hamradio Quick File Broadcasting Protocol (HQFBP) is designed to enable efficient, robust, and asynchronous file and data broadcasting over radio communication links, including challenging environments such as satellite downlink.
HQFBP leverages the **Concise Binary Object Representation (CBOR)** (RFC 8949\) for compact and extensible header encoding, optimizing overhead for low-bandwidth digital radio channels. The protocol is inherently delay-tolerant and supports large file transmission via mandatory chunking and header reconstruction mechanisms.

### 1.1. Goals

1. **Low Overhead:** Utilize CBOR's small integer keys for common header fields to minimize transmission size.  
2. **Error Tolerance:** Support asynchronous delivery and reassembly, suitable for intermittent radio links.  
3. **File Broadcasting:** Allow a single transmission to be useful to multiple receiving stations simultaneously.
4. **Extensibility:**
 * Provide a mechanism for future header fields using standard CBOR integer or text keys.
 * support multiple encodings schemes (eg fec, Reed-Solomon, RaptorQ, etc) for experimentations
5. **Do not reinvent the wheel**: Leverage existing concepts and ideas from other protocols.

## 2. Terminology

* **CBOR:** Concise Binary Object Representation (RFC 8949)[1].  
* **HQFBP Message:** A single, independently transmittable protocol data unit (header \+ payload).  
* **Chunk:** A single HQFBP Message when transmitting a file split into multiple parts.  
* **File Transfer:** The complete set of Chunks required to reconstruct the original file.  
* **Receiver:** An amateur radio station capable of receiving and decoding HQFBP Messages.  
* **Callsign-SSID:** A station identifier formatted as a UTF-8 string, optionally including the Secondary Station Identifier (SSID) (e.g., W1AW, W1AW-7).

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
[RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

### 2.1. MIME Type Registration

The media type for the HQFBP Message format is:  
`application/vnd.hqfbp+cbor`  
This type signifies that the content is an HQFBP message, consisting of a CBOR header followed by a binary payload (as defined in Section 3). It can be used when embedding HQFBP messages within other container formats.

## 3. HQFBP Message Structure

An HQFBP Message is composed of two distinct components transmitted sequentially:

1. **CBOR Encoded Header (Required):** A single CBOR map object containing all metadata necessary for the receiver to process the payload.  
2. **Data Payload (Optional):** The binary data content. For chunked file transfers, this contains a segment of the original file.

The total HQFBP Message is encapsulated within the underlying digital mode's frame (e.g., an FX.25 UI frame payload) or directly sent by the underlying physical link.

## 4. HQFBP CBOR Header Definition (Static Key Mapping)

To achieve maximum header compression, HQFBP mandates the use of short integer keys for commonly used fields. Keys 1 through 9 are reserved for single-digit optimization.

| CBOR Key (Integer) | Header Field Name | Data Type | Requirement | Default Value / Notes |
| :---- | :---- | :---- | :---- | :---- |
| 0 | **Message-Id** | Unsigned Integer | MANDATORY | A strictly increasing counter for all messages sent by a single originator. |
| 1 | **Src-Callsign** | Text String (UTF-8) | OPTIONAL | Originating station's Callsign-SSID. |
| 2 | **Dst-Callsign** | Text String (UTF-8) | OPTIONAL | Intended recipient (may be a broadcast address like QST-0). |
| 3 | **Content-Format** | Unsigned Integer | OPTIONAL | CoAP Content-Format ID (e.g., 0 for text/plain;charset=UTF-8) [3] . Mutually exclusive with 5. |
| 4 | **Content-Type** | Text String (UTF-8) | OPTIONAL | Full HTTP-style MIME type (alternative to 4). Mutually exclusive with 4. |
| 5 | **Content-Encoding** | Text String, Integer, or Array | OPTIONAL | Compression/Encoding applied to the original file or chunk. If an Array, encodings MUST be applied in the order listed. Parameterized encodings can be compacted as nested lists. |
| 6 | **Repr-Digest** | Byte String | OPTIONAL | Hash or Checksum of the **original, uncompressed** file content (e.g., SHA-256, CRC32). |
| 7 | **Content-Digest** | Byte String | OPTIONAL | Hash or Checksum of the **encoded/compressed** payload (e.g., SHA-256, CRC32). |
| 8 | **File-Size** | Unsigned Integer | OPTIONAL | Total size of the original, uncompressed file in bytes. **Required** for chunked transfers. |
| 9 | **Chunk-Id** | Unsigned Integer | CONDITIONAL | The sequential number of the current chunk (0-based). **Required** for chunked transfers. |
| 10 | **Original-Message-Id** | Unsigned Integer | CONDITIONAL | The Message-Id of the first chunk in a file transfer. Used by receivers for grouping. |
| 11 | **Total-Chunks** | Unsigned Integer | CONDITIONAL | The total count of chunks in the File Transfer. **Required** for chunked transfers. |
| 12 | **Payload-Size** | Unsigned Integer | OPTIONAL | The exact size of the payload in bytes after all pre-boundary encodings are removed. Used to trim padding added by FEC schemes. |

### 4.1. Header Defaults

If both Content-Format (4) and Content-Type (7) are absent, the content is assumed to be the default: `text/plain;charset=UTF-8` (CoAP Content-Format ID 0).

## 5. File Chunking and Header Merging

When transmitting a file that exceeds the maximum payload size of the underlying link, the file MUST be split into chunks.

### 5.1. Chunk Identification

When chunking is used, the following fields are **MANDATORY**: Chunk-Id, Total-Chunks, and Original-Message-Id.
File-Size SHOULD be transmitted.

* **Chunk-Id:** MUST be the sequential number of the current chunk (0-based).  
* **Total-Chunks:** MUST be the total count of chunks that constitute the complete file.

Example: If Chunk-Id is 3 and Total-Chunks is 10, it indicates this is the 4th chunk out of 10 total chunks.

### 5.2. Header Merging (Reconstruction)

The final, fully decoded header of the reconstructed file MUST be the merge of all distinct header fields received from every chunk, with the exception of the core chunking parameters (0, 9, 10, 11).

**Merging Rules:**

1. **Consistency Check:** The receiver MUST verify that fields common across all chunks in a file transfer (e.g., Src-Callsign, Repr-Digest, File-Size) are consistent. If a discrepancy is detected, the File Transfer is considered corrupted and MUST be discarded.
2. **Mandatory Markers:** Reassembly markers such as `chunk()`, `repeat()`, or `rq()` MUST be present in the `Content-Encoding` array for proper reassembly.
3. **Payload Trimming:** If `Payload-Size` (12) is present in any chunk, it MUST be used to trim the payload of that specific PDU after post-boundary decodings. Trimming is the process of removing padding required at reassembly or transmission time (e.g., to align with FEC block sizes).
4. **Merge Logic:** For any field other than the core chunking parameters, the header of the *first* received chunk (Chunk Index 0) is provisionally considered the complete header. However, if any subsequent chunk contains an optional header field that was *missing* from the first chunk, that field MUST be included in the final merged header.
   * *Rationale:* This allows for small changes in metadata to be communicated without excessive repetition. In practice, all *critical* metadata should be present in Chunk 0.

## 6. Compression and Integrity

### 6.1. Compression (Content-Encoding)

The Content-Encoding field specifies the compression and/or forward error correction (FEC) algorithm applied to the original data *before* and/or *after* chunking.  
The value of Content-Encoding MAY be a single Text String, an Integer (referencing the registry below), or an **Array**. If it is an Array, the encodings MUST be applied sequentially, in the order they appear in the Array. Receivers MUST sequentially undo these encodings in the reverse order.  

To further compact parameterized encodings, they MAY be represented as a nested list where the first element is the integer ID from the registry and subsequent elements are the parameters. For instance, `rs(255, 233)` can be compacted as `[7, 255, 233]`. This allows the `Content-Encoding` field to be a nested list (e.g., `["gzip", [7, 255, 233]]`).

Common values include standard compression schemes like `gzip`, `deflate`, `br`, or `lzma`. This field also supports forward error correction (FEC) schemes relevant to noisy radio environments, such as Fountain Codes (e.g., RaptorQ as per IETF RFC 6330 [[2]](https://datatracker.ietf.org/doc/html/rfc6330)), and other erasure codes like **LDPC** or **Reed-Solomon** (eg `rs(255,233)`).

To minimize overhead, well-known encodings SHOULD use their assigned integer values.

#### 6.1.1. Well-Known Encoding Registry

| ID | Descriptor | Description | Parameters / Notes |
| :--- | :--- | :--- | :--- |
| -1 | h | `hqfbp` Header Boundary Marker | See Section 6.1.2. |
| 0 | identity | No encoding applied | - |
| 1 | gzip | Gzip compression | RFC 1952 |
| 2 | deflate | Deflate compression | RFC 1951 |
| 3 | br | Brotli compression | - |
| 4 | lzma | LZMA compression | - |
| 5 | crc16 | 16-bit CRC | Appended after data |
| 6 | crc32 | 32-bit CRC | Appended after data |
| 7 | rs(n,k) | Reed-Solomon | Block length `n`, data length `k` |
| 8 | rq(l,m,r) | RaptorQ | Source length `l`, MTU `m`, repair `r` |
| 9 | conv(k,r) | Convolutional coding | Constraint length `k`, rate `r` |
| 10 | scr(p,i) | Additive Scrambler | Polynomial mask `p` can be hex/bin (e.g. `0x1A9` or `0b110101001` for CCSDS), optional seed `i` (hex) sets LFSR seed. |
| 11 | chunk(s) | Reassembly Marker | Chunk with size `s` |
| 12 | repeat(k) | Reassembly Marker | Simple repetition `k` times |
| 13 | lora(sf,bw,cr) | LoRa Spread Spectrum | Spread Factor `sf`, Bandwidth in Hz `bw`, Rate `cr` |
| 14 | ldpc(n,k) | LDPC FEC | Block length `n`, data length `k` |
| 15 | golay | Golay(24,12) code | - |
| 16 | manchester | Manchester encoding | - |
| 17 | nrzi | NRZI encoding | Non-Return-to-Zero Inverted |
| 18 | diff | Differential Encoding | - |
| 19 | interleave(d) | Interleaving | Interleaving depth `d` |
| 20 | preamble(p,l) | Preamble sequence | Pattern `p`, length `l` |
| 21 | sync(w) | Sync word | Hex pattern `w` |
| 22 | crc(n) | `n`-bit CRC | e.g. `crc(14)` is used in FT8 |
| 23 | afsk(b) | AFSK Modulation | Baudrate `b` |
| 24 | fsk(b,d) | FSK Modulation | Baudrate `b`, deviation in Hz `d` |
| 25 | cpfsk(b,d) | Continuous-Phase FSK | Baudrate `b`, deviation in Hz `d` |
| 26 | gfsk(b,d,bt) | Gaussian FSK | Baudrate `b`, deviation in Hz `d`, filter BT `bt` |
| 27 | bpsk(b) | BPSK Modulation | Baudrate `b` |
| 28 | qpsk(b) | QPSK Modulation | Baudrate `b` |
| 29 | mfsk(n,r) | M-ary FSK | Tones `n`, symbol rate `r` |
| 30 | ofdm(n,bw) | Orthogonal FDM | Carriers `n`, bandwidth in Hz `bw` |
| 31 | fm(bw) | Frequency Modulation | Optional deviation/bandwidth in Hz `bw` |
| 32 | usb | Upper Sideband | - |
| 33 | lsb | Lower Sideband | - |
| 34 | am | Amplitude Modulation | - |
| 35 | cw | Continuous Wave | - |
| 36 | freq(f) | Center Frequency | Center Frequency in Hz `f` (integer preferred) |
| 37 | off(o) | Audio center offset | Audio center offset in Hz `o` (integer) |
| 38 | bw(bw) | Bandwidth | Occupied bandwidth in Hz `bw` (integer preferred) |
| 39 | hqfbp | HQFBP Protocol | - |
| 40 | aprs | APRS Protocol | - |
| 41 | ax.25 | AX.25 Link Layer | - |
| 42 | ccsds | CCSDS Protocols | - |
| 43 | ao40 | AO-40 Protocol | FUNcube, etc. |
| 44 | usp | USP / Simple Protocol | GOMspace |
| 45 | mobitex | Mobitex Wireless Data | - |
| 46 | ft8 | FT8 Framing | WSJT-X |
| 47 | js8 | JS8Call Framing | - |
| 48 | olivia | Olivia Framing | - |
| 49 | winlink | Winlink / RMS | - |
| 50 | pocsag | POCSAG Paging | - |
| 51 | psk31 | PSK31 Framing | - |
| 52 | rtty | RTTY Baudot Framing | - |
| 53 | varicode | Varicode encoding | Used in PSK31 |
| 54 | asm(w) | Attached Sync Marker | Optional hex sync word `w` |
| 55 | bstuff(n) | Bit Stuffing | Insert 0 after `n` consecutive 1s (default n=5 for HDLC) |
| 56 | post_asm(w) | Post-Attached Sync Marker | Trailer sync word `w` |
| 57 | il2p | IL2P Protocol | IL2P Link Layer Protocol Framing |

> [!NOTE]
> Differential modulations (e.g., DBPSK or DQPSK) are not assigned separate IDs. They MUST be represented by applying `diff` (34) followed by the base modulation:
> - `dbpsk(baud) ≡ [34, [43, baud]]`
> - `dqpsk(baud) ≡ [34, [44, baud]]`

> [!NOTE]
> - `scr(g3ruh)` is $1+x^{12}+x^{17}$.
> - `scr(ccsds)` is $1+x^3+x^5+x^7+x^8$ thus `scr(0x1A9, 0xFF)`.

#### 6.1.3. Unit Notation and Encoding Recommendations

To ensure interoperability and minimize CBOR overhead, the following conventions and recommendations apply:

**Standard Unit Notation (Human Readable Text)**

When representing parameters as text strings (e.g., in `Content-Encoding: ["fsk(9k6, 4k8)"]`), the following suffixes MUST be used:
- `k`: kHz (for frequencies) or kbps/baud (for data rates). Example: `9k6` = 9600.
- `M`: MHz. Example: `144M8` = 144.8 MHz.

The interpretation of these suffixes is determined by the context of the descriptor (e.g., frequency vs. baud rate).

**Compact Encoding Recommendations**

To minimize transmission size, transmitters SHOULD avoid using floating-point numbers in CBOR-encoded descriptors, as CBOR floats (especially double precision) require significantly more bytes (typically 9 bytes) than small integers.

- **Frequencies:** SHOULD be encoded in **Hz** as integers instead of **MHz** as floats.
    - *Example (144.8 MHz):* Use `144800000` or `144M8` instead of `144.8`.
- **Baud rates and Deviations:** SHOULD be encoded as integers.
    - *Example (9600 baud):* Use `9600` or `9k6`.

#### 6.1.2. The Header Boundary Marker (-1)

The integer `-1` (represented as `"h"` in documentation) acts as a structural boundary in the encoding array:

Pre-Boundary: Encodings applied only to the File Payload before HQFBP protocol encapsulation.

Post-Boundary: Encodings applied to the HQFBP Message (Header + Payload).

If -1 is present, the receiver MUST process post-boundary encodings (right-to-left) to reach the CBOR header, then use the header metadata to process pre-boundary encodings.

Example: `[1, -1, 20]` implies:

1. Calculate/Verify `crc32` (20) over the entire packet.
2. Remove `crc32` to reveal the CBOR Header.
3. Apply `gzip` (1) to the payload.

In case of non-transparent encoding, the transmitter MUST:

* either publish publicly the encoding ahead of emission time (eg. well-known satellite downlink),
* or emit a frame containing the header in a preliminary frame whose `Content-Type` is `application/vnd.hqfbp+cbor` so that the decoder know how to decode the chunks.

### 6.2. Integrity (Repr-Digest and Content-Digest)

* **Repr-Digest:** Provides a cryptographic hash or checksum of the **original, uncompressed** file content (Representation Digest). Receivers SHOULD use this digest to verify the integrity of the reconstructed file after decoding.
* **Content-Digest:** Provides a cryptographic hash or checksum of the **encoded/compressed** payload. Receivers MAY use this to verify the integrity of the payload before decoding.

Common digest algorithms include cryptographic hashes (e.g., `sha256`, `sha1`) where the algorithm is usually implied by the length of the byte string. Additionally, simpler checksums for fast verification are supported, such as `crc32` or `crc16`, where the algorithm is also implied by the known digest length.

## 6.3. Hybrid ARQ and Quality Metrics

Receivers SHOULD implement Hybrid ARQ logic to improve reliability in noisy environments. One common approach is **Best-of-N Selection**:

* When multiple copies of the same PDU (identified by Message-Id and Chunk-Id) are received, the receiver SHOULD track a "quality metric" for each.
* Quality can be derived from checksum verification (success/fail) or error correction metrics (e.g., fewer bit flips corrected in Viterbi/RS decoding).
* The version with the highest quality metric SHOULD be stored and used for reassembly.

For iterative codes or symbolic reassembly (like RaptorQ), the receiver SHOULD accumulate enough distinct symbols/packets to satisfy the decoding requirements specified in the `Content-Encoding` parameters.

## 7. Addressing (Callsigns)

The Src-Callsign and Dst-Callsign fields MUST use standard amateur radio callsigns, optionally appended with an SSID (e.g., \-1 to \-15), separated by a hyphen.

* **Broadcasts:** For broadcasts, the Dst-Callsign may be omitted or set to a reserved broadcast address (e.g., QST-0).  
* **Encoding:** All callsigns are encoded as UTF-8 text strings within the CBOR map.

## 8. References

[1]
**RFC 8949 (CBOR):** C. Bormann; P. Hoffman. *Concise Binary Object Representation (CBOR)*. December 2020.  
[2]
**IETF RFC 6330 (RaptorQ):** A. Shokrollahi; S. V. A. B. Hartenstein; J. P. K. Hartenstein. *RaptorQ Forward Error Correction Scheme for Object Delivery*. August 2011.  
[3]
**CoAP Content-Formats:** Defined by the IANA CoAP Content-Formats Registry (RFC 7252, RFC 9176). See https://www.iana.org/assignments/core-parameters/core-parameters.xhtml#content-formats

## Appendix A. Future Extensions (Informative)

### A.1. Related protocols

It is foreseen that HQFBP to be used either as a raw frame but also on top of other protocols like AX.25, FX.25, or even UDP. It should be also a proper companion for Bundle Protocol [RFC9171].

### A.2. CBOR full-extent

It is expected to experiment with CBOR only messages, where the entire data and header are encoded as a single CBOR item. This would allow for more efficient extension and reduce overhead. But also it would bring standard CBOSE signing and verification mechanisms into play.

## Appendix B. CBOR Encoding Example (Informative)

Consider a file (4032 bytes, content type text/markdown, compressed with gzip) split into two chunks.  
**Chunk 1 (Index 0 of 2):**

```json
{  
  "0": 1001,                   // Message-Id: 1001  
  "9": 0,                      // Chunk-Id: 0  
  "1": "FOSM-1",               // Src-Callsign of the satellite  
  "5": 1,                      // Content-Encoding: gzip  
  "4": "text/markdown",        // Content-Type: text/markdown  
  "8": 4032,                   // File-Size (Original)  
  "10": 1001,                  // Original-Message-Id: 1001  
  "11": 2                      // Total-Chunks: 2  
}
```

**Chunk 2 (Index 1 of 2):**

```json
{  
  "0": 1002,                   // Message-Id: 1002  
  "9": 1,                      // Chunk-Id: 1  
  "1": "FOSM-1",               // Src-Callsign  
  "7": "sha-256=:RK/0qy18MlBSVnWgjwz6lZEWjP/lF5HF9bvEF8FabDg=:",           // Repr-Digest: (Hash byte string)  
  "10": 1001,                  // Original-Message-Id: 1001  
  "11": 2                      // Total-Chunks: 2  
}
```

The receiver merges these to get the complete metadata for the file transfer, including both Content-Encoding from Chunk 1 and Repr-Digest from Chunk 2.

## Appendix C. The Bootstrap Sequence for Encapsulated Encodings (Informative)

When a transmission uses "non-transparent" encodings (where the CBOR header itself is wrapped in an encoding like Reed-Solomon or LDPC), a receiver cannot parse the header to discover how to decode the packet.

To resolve this, HQFBP uses a Bootstrap Sequence: a "Transparent Announcement" followed by "Encapsulated Data."

### C.1. Step 1: The Transparent Announcement (Message 1001)

This message is sent using standard transparent CBOR. Its payload contains the specific header metadata required to decode the next message(s).

| Field |	Value |	Role |
|-|-|-|
| Header: Message-Id |	1001 |	The ID of the announcement itself.|
| Header: Content-Type | application/vnd.hqfbp+cbor |	Tells the receiver: "The payload is a header for a future message." |
| Payload (CBOR Map)	| ```{0: 1002, 5: [-1, "ldpc", [7, 255, 233]]}``` |Metadata for the upcoming Message 1002. |

### C.2. Step 2: The Encapsulated File Transfer (Message 1002)

Now that the receiver knows the parameters for ID 1002, the transmitter sends the file in chunks. Each chunk is "wrapped" in the FEC layers.

**Chunk 0 (The first segment):**

* Physical Layer: `[LDPC( RS( CBOR_HEADER + DATA_PART_1 ) )]`
* Header (Visible only after FEC decoding):
```json
    {
      "0": 1002,            // Message-Id matches the Announcement
      "9": 0,               // Chunk-Id (0-based)
      "10": 1002,           // Original-Message-Id
      "11": 2               // Total-Chunks
    }
```

**Chunk 1 (The second segment):**

* Physical Layer: `[LDPC( RS( CBOR_HEADER + DATA_PART_2 ) )]`
* Header:
```json
{
  "0": 1003,            // Unique Message-Id
  "9": 1,               // Chunk-Id 1
  "10": 1002            // Links back to the start of the transfer
}
```