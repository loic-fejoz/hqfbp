# Hamradio Quick File Broadcasting Protocol (HQFBP)

The **Hamradio Quick File Broadcasting Protocol (HQFBP)** is designed to enable efficient, robust, and asynchronous file and data broadcasting over radio communication links. It is particularly optimized for challenging environments such as satellite downlinks, leveraging **CBOR (RFC 8949)** for compact headers and minimizing overhead.

For the full technical specification, please refer to [rfc.md](./rfc.md).

## 🛰️ FOSM-1 Mission Context

This protocol is not just a theoretical concept; it is one of the experiments that will be run live on an actual satellite payload called **[FOSM-1](https://www.federation-openspacemakers.com/fr/participer/projets/phoenix-infrastructure-informatique-orbi/fosm-1/)**.

FOSM-1 acts as a testbed for open-source space technologies, and HQFBP will demonstrate reliable file broadcasting from orbit to ground stations.

## ⚖️ Comparison with Other Protocols

To help radio amateurs understand where HQFBP fits in the ecosystem, here is how it compares to existing solutions:

| Protocol | Type | HQFBP Comparison |
| :--- | :--- | :--- |
| **FLAMP** | File Broadcasting (FLDIGI) | **Modern Alternative:** Like FLAMP, HQFBP broadcasts files. However, HQFBP uses binary CBOR headers instead of text-based headers, significantly reducing overhead for low-bandwidth links. It also separates metadata from payload more cleanly. |
| **SSDV** | Image Broadcasting | **General Purpose:** SSDV is excellent but specialized for images. HQFBP is content-agnostic; it can transfer images, text, binary binaries, or any other file type with equal efficiency, supporting optional compression and digests. |
| **PACTOR** | Connected Mode / ARQ | **Broadcast (Connectionless):** PACTOR is primarily a connected mode protocol (Auto-Repeat Request), requiring a handshake. HQFBP is a **broadcasting** protocol (FEC/Erasure coding supported), meaning one station transmits and *many* can receive simultaneously without a back-channel. |
| **WINLINK** | Email / Message System | **Transport Layer:** Winlink is a full global email system. HQFBP is a lighter-weight **transport layer** focused on the efficient delivery of a single file or message. HQFBP could theoretically be used as a transport *for* a system like Winlink. |

## 🚀 Key Features

*   **Low Overhead:** CBOR-encoded headers with small integer keys.
*   **Asynchronous:** Designed for broadcast; no handshake required.
*   **Robust:** Supports chunking for large files and robust header reconstruction.
*   **Flexible:** Optional compression (gzip, etc.) and integrity checks (CRC32, SHA256, etc.).

## License

See [LICENSE](./LICENSE) for details.
