# CHANGELOG

## [0.2] - 2026-01-09

### Added
- **Payload-Size** (Key 12): New header field to specify the exact payload size, enabling precise trimming of FEC-added padding.
- **New Encodings**:
    - Reed-Solomon: `rs(n,k)`
    - RaptorQ: `rq(l,m,r)`
    - Convolutional: `conv(k,r)`
    - Scrambling: `scr(poly)`
    - Reassembly Markers: `chunk(s)` and `repeat(k)`
- **Hybrid ARQ Guidance**: Added Section 6.3 describing Best-of-N selection and quality metrics for improved reliability.

### Changed
- **RFC Version**: Bumped to 0.2.
- **Header Merging Rules**: 
    - Clarified that reassembly markers are mandatory for reconstruction.
    - Mandated the use of `Payload-Size` for trimming reconstructed files.
    - Updated exclusion list for header merging to include `Chunk-Id` (Key 9) for consistency checks.

## [0.1] - 2025-12-17

### Added
- Initial protocol specification.
- CBOR-based header mapping (Keys 0-11).
- Support for sequential encodings and the Header Boundary Marker (`h`).
- Basic file chunking and reassembly mechanisms.
