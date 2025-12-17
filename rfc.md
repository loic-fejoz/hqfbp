# Hamradio Quick File Broadcasting Protocol (HQFBP) Specification

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
| 1 | **Chunk-Index** | Unsigned Integer | CONDITIONAL | The sequential number of the current chunk (0-based). **Required** for chunked transfers. |
| 2 | **Src-Callsign** | Text String (UTF-8) | OPTIONAL | Originating station's Callsign-SSID. |
| 3 | **Dst-Callsign** | Text String (UTF-8) | OPTIONAL | Intended recipient (may be a broadcast address like QST-0). |
| 4 | **Content-Format** | Unsigned Integer | OPTIONAL | CoAP Content-Format ID (e.g., 0 for text/plain;charset=UTF-8) [3] . Mutually exclusive with 7. |
| 5 | **Content-Encoding** | Text String or Array of Text Strings (UTF-8) | OPTIONAL | Compression/Encoding applied to the original file. If an Array, encodings MUST be applied in the order listed. |
| 6 | **Content-Digest** | Byte String | OPTIONAL | Hash or Checksum of the **original, uncompressed** file content (e.g., SHA-256, CRC32). |
| 7 | **Content-Type** | Text String (UTF-8) | OPTIONAL | Full HTTP-style MIME type (alternative to 4). Mutually exclusive with 4. |
| 8 | **File-Size** | Unsigned Integer | OPTIONAL | Total size of the original, uncompressed file in bytes. **Required** for chunked transfers. |
| 9 | **Original-Message-Id** | Unsigned Integer | CONDITIONAL | The Message-Id of the first chunk in a file transfer. Used by receivers for grouping. |
| 10 | **Total-Chunks** | Unsigned Integer | CONDITIONAL | The total count of chunks in the File Transfer. **Required** for chunked transfers. |

### 4.1. Header Defaults

If both Content-Format (4) and Content-Type (7) are absent, the content is assumed to be the default: `text/plain;charset=UTF-8` (CoAP Content-Format ID 0).

## 5. File Chunking and Header Merging

When transmitting a file that exceeds the maximum payload size of the underlying link, the file MUST be split into chunks.

### 5.1. Chunk Identification

When chunking is used, the following fields are **MANDATORY**: 1 (Chunk-Index), 10 (Total-Chunks), 8 (File-Size), and 9 (Original-Message-Id).

* **Chunk-Index (Key 1):** MUST be the sequential number of the current chunk (0-based).  
* **Total-Chunks (Key 10):** MUST be the total count of chunks that constitute the complete file.

Example: If Chunk-Index is 3 and Total-Chunks is 10, it indicates this is the 4th chunk out of 10 total chunks.

### 5.2. Header Merging (Reconstruction)

The final, fully decoded header of the reconstructed file MUST be the merge of all distinct header fields received from every chunk.  
**Merging Rules:**

1. **Consistency Check:** The receiver MUST verify that fields common across all chunks in a file transfer (e.g., Src-Callsign, Content-Digest, File-Size) are consistent. If a discrepancy is detected, the File Transfer is considered corrupted and MUST be discarded, or the station transmitting the inconsistent data should be logged for potential transmission issues.  
2. **Merge Logic:** For any field other than the core chunking parameters (0, 1, 9, 10, and 8 for payload length/size), the header of the *first* received chunk (Chunk Index 0\) is provisionally considered the complete header. However, if any subsequent chunk contains an optional header field that was *missing* from the first chunk, that field MUST be included in the final merged header.  
   * *Rationale:* This allows for small changes in metadata (e.g., compression method) to be communicated without excessive repetition, while maintaining the robustness of the core fields. In practice, all critical metadata should be present in Chunk 0.

## 6. Compression and Integrity

### 6.1. Compression (Content-Encoding)

The Content-Encoding (Key 5\) field specifies the compression and/or forward error correction (FEC) algorithm applied to the original data *before* chunking.  
The value of Key 5 MAY be a single Text String or an **Array of Text Strings**. If it is an Array, the encodings MUST be applied sequentially, in the order they appear in the Array. Receivers MUST sequentially undo these encodings in the reverse order.  
Common values include standard compression schemes like `gzip`, `deflate`, or `lzma`. This field also supports forward error correction (FEC) schemes relevant to noisy radio environments, such as Fountain Codes (e.g., RaptorQ as per IETF RFC 6330 [[2]](https://datatracker.ietf.org/doc/html/rfc6330)), and other erasure codes like **LDPC** or **Reed-Solomon**.

### 6.2. Integrity (Content-Digest)

The Content-Digest (Key 6\) provides a cryptographic hash or checksum of the **original, uncompressed** file content. Receivers SHOULD use this digest to verify the integrity of the reconstructed file.  
Common digest algorithms include cryptographic hashes (e.g., `sha256`, `sha1`) where the algorithm is usually implied by the length of the byte string. Additionally, simpler checksums for fast verification are supported, such as `crc32` or `crc16`, where the algorithm is also implied by the known digest length.

## 7. Addressing (Callsigns)

The Src-Callsign (Key 2\) and Dst-Callsign (Key 3\) fields MUST use standard amateur radio callsigns, optionally appended with an SSID (e.g., \-1 to \-15), separated by a hyphen.

* **Broadcasts:** For broadcasts, the Dst-Callsign may be omitted or set to a reserved broadcast address (e.g., QST-0).  
* **Encoding:** All callsigns are encoded as UTF-8 text strings within the CBOR map.


## 8. References

[1]
**RFC 8949 (CBOR):** C. Bormann; P. Hoffman. *Concise Binary Object Representation (CBOR)*. December 2020.  
[2]
**IETF RFC 6330 (RaptorQ):** A. Shokrollahi; S. V. A. B. Hartenstein; J. P. K. Hartenstein. *RaptorQ Forward Error Correction Scheme for Object Delivery*. August 2011.  
[3]
**CoAP Content-Formats:** Defined by the IANA CoAP Content-Formats Registry.

## Appendix A. Future Extensions (Informative)

### A.1. Related protocols
It is foreseen that HQFBP to be used either as a raw frame but also on top of other protocols like AX.25, FX.25, or even UDP. It should be also a proper companion for Bundle Protocol [RFC9171].

### A.2 Message Checksum

A message checksum field is planned for future versions to ensure data integrity. This will be added as a new key in the CBOR map str. See Repr-Digest vs Content-Digest.

### A.3 CBOR full-extent

It is expected to experiment with CBOR only messages, where the entire data and header are encoded as a single CBOR item. This would allow for more efficient extension and reduce overhead. But also it would bring standard CBOSE signing and verification mechanisms into play.



## Appendix B. CBOR Encoding Example (Informative)

Consider a file (4032 bytes, content type text/markdown, compressed with gzip) split into two chunks.  
**Chunk 1 (Index 0 of 2):**
```json
{  
  "0": 1001,                   // Message-Id: 1001  
  "1": 0,                      // Chunk-Index: 0  
  "2": "FOSM-1",               // Src-Callsign of the satellite  
  "5": "gzip",                 // Content-Encoding: gzip  
  "7": "text/markdown",            // Content-Type: text/markdown  
  "8": 4032,                   // File-Size (Original)  
  "9": 1001,                   // Original-Message-Id: 1001  
  "10": 2                      // Total-Chunks: 2  
}
```

**Chunk 2 (Index 1 of 2):**
```json
{  
  "0": 1002,                   // Message-Id: 1002  
  "1": 1,                      // Chunk-Index: 1  
  "2": "FOSM-1",               // Src-Callsign  
  "6": "sha-256=:RK/0qy18MlBSVnWgjwz6lZEWjP/lF5HF9bvEF8FabDg=:",           // Content-Digest: (Hash byte string)  
  "9": 1000,                   // Original-Message-Id: 1001  
  "10": 2                      // Total-Chunks: 2  
}
```

The receiver merges these to get the complete metadata for the file transfer, including both Content-Encoding from Chunk 1 and Content-Digest from Chunk 2.