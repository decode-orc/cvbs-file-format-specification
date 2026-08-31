# Video Standard Preset Definitions

This document is part of the [CVBS File Format Specification](index.md). It contains the normative Video Standard Preset definitions referenced in Section 4.2 of that specification.

> **Note — Line Numbering Convention:** This specification uses **0-based frame line numbering**: frame line 0 is the first line of a frame, and the last line of a frame is numbered N−1 (where N is the total number of lines in that frame). This follows software/coding convention and differs from the 1-based line numbering used in EBU and SMPTE broadcast standards, where lines are numbered starting from 1. Informational blocks that quote line numbers from EBU or SMPTE documents preserve the original 1-indexed numbering from those standards; each such block notes this explicitly. Where the source standard numbers lines sequentially across the full frame, those values are referred to here as **frame line numbers**.

**Naming convention:** Preset names follow the pattern `PRIMARY` or `PRIMARY_SUBSET`, where `PRIMARY` identifies the base standard and `SUBSET` (when present) identifies a regional or technical variant. The underscore `_` separates the primary designator from the subset. Preset names use only uppercase ASCII letters, digits, and underscores; each name defined in this specification is unique. Additional presets may be defined in future revisions or companion documents.

Examples:
- `PAL` — the primary PAL standard
- `NTSC` — the primary NTSC standard
- `PAL_M` — a PAL-family variant using NTSC 525-line/60 Hz timing with PAL colour subcarrier modulation

**External standard references used by these presets:**

- [ITU-R BT.1700-1 (2005)](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/BT-1700-E/BT-1700-E.md), *Characteristics of composite video signals for conventional analogue television systems* — defines conventional analogue composite PAL signal formats, including 625-line PAL and 525-line PAL used by PAL-M, and includes the approved SMPTE 170M-2004 NTSC text in Annex 2.
- [SMPTE 170M-2004](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-170M-2004/SMPTE-170M-2004.md), *Composite Analog Video Signal - NTSC for Studio Applications* — defines the analogue 525-line, 59.94-field NTSC studio composite signal.
- [SMPTE 244M-2003](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-244M-2003/SMPTE-244M-2003.md), *Bit-Parallel Digital Interface for NTSC Composite Video Signals* — defines the 4fsc digital representation, sample phase, line structure, and 10-bit sample levels for NTSC composite video.
- [EBU Tech. 3280-E](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/EBU-Tech-3280-E/EBU-Tech-3280-E.md), *Specification of interfaces for 625-line digital PAL signals* — defines the 625-line digital PAL composite interface, including 4fsc sample levels, sample phase, line structure, and PAL colour-frame relationship.

---

## Stored Frame and Line Origin (Normative)

Stored frames are **0H-aligned**. The horizontal timing reference **0H** is the half-amplitude point of the leading (falling) edge of the line sync pulse. For every stored frame:

- **Horizontal origin:** the first stored sample of a frame is the first sampling instant at or after 0H of the frame's first line. For presets with orthogonal sampling (`NTSC`, `PAL_M`), every stored line follows the same rule: sample 0 of each stored line is the first sampling instant at or after that line's 0H, so the horizontal sync pulse and blanking occupy the **start** of the stored line and the active picture region follows.
- **Vertical origin:** stored frame line 0 corresponds to frame line 1 of the 1-based broadcast frame line convention — the first line of field 1.
- The exact sub-sample position of 0H relative to sample 0 is a consequence of the preset's sampling phase and SC/H relationship; it is not additionally constrained by this specification.

This storage origin intentionally follows the `ld-decode`/`vhs-decode` TBC line convention and **differs from the digital line structure of the 4fsc interface standards** (SMPTE 244M-2003 and EBU Tech. 3280-E), in which the digital line begins with the digital active line and 0H falls late in the digital line — between samples 784 and 785 for NTSC, and midway between samples 957 and 958 on broadcast frame line 1 of PAL frame 1. Informational blocks in this document that quote sample numbers from those standards use the source standard's digital-line coordinates unless explicitly restated in stored coordinates.

---

## Preset: `PAL`

**External standards:**

- [ITU-R BT.1700-1 (2005)](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/BT-1700-E/BT-1700-E.md), Annex 1 Part B: analogue 625-line PAL signal format used for PAL-I-style baseband CVBS characteristics.
- [EBU Tech. 3280-E](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/EBU-Tech-3280-E/EBU-Tech-3280-E.md): digital 625-line PAL composite interface used for the 4fsc sampling structure, sample levels, line timing, and colour-frame relationship.

**Sampling rate:**

| Formula | Frequency |
|---------|-----------|
| 4 × 625 × 25 × (1135/4 + 1/625) | **17,734,475 Hz** (exact) |

**Sample level table (EBU Tech. 3280-E) — applicable when Sample Encoding Preset is `CVBS_U10_4FSC` or `CVBS_U16_4FSC`:**

| Level            | 10-bit Decimal | 10-bit Hex   | Notes                       |
| ---------------- | -------------- | ------------ | --------------------------- |
| Protected min    | 0–3            | 000h–003h    | Must never appear; reserved |
| Sync tip         | 4              | 004h (01.0h) | Minimum legal sample value  |
| Blanking         | 256            | 100h (40.0h) | Zero-signal reference       |
| Black            | 256            | 100h (40.0h) | No pedestal: black = blanking |
| White (100%)     | 844            | 34Ch (D3.0h) | 100% white                  |
| Peak (w/ chroma) | 1019           | 3FBh         | Maximum legal sample value  |
| Protected max    | 1020–1023      | 3FCh–3FFh    | Must never appear; reserved |

**Horizontal line structure:**

| Total samples/line       | Digital active samples | Digital blanking samples |
| ------------------------ | ---------------------- | ------------------------ |
| 1135.0064 (exact average; see note) | 948 (nominal)         | 187.0064 (nominal)       |


**PAL — non-orthogonal 4fsc sampling structure:** At the exact PAL 4fsc sampling rate there are **1135.0064 sample periods per line** and therefore exactly **709,379 samples per frame**. For frame 1 of the PAL sequence at 0 degrees Sc/H, the half-amplitude point of the leading edge of the line sync pulse on broadcast frame line 1 (stored frame line 0) falls midway between samples **957** and **958** in EBU Tech. 3280-E's digital-line coordinates; in this specification's stored coordinates that sync edge immediately precedes the frame's first stored sample (see [Stored Frame and Line Origin](#stored-frame-and-line-origin-normative)). On succeeding lines the sampling structure advances by **0.361 ns per line**, i.e. **4 samples per frame**. Consequently the PAL 4fsc sampling lattice is **non-orthogonal** and repeats at **frame rate** rather than at line rate.

The analogue PAL composite waveform is sampled at **4fsc**, so sampling instants occur at **45 degrees, 135 degrees, 225 degrees, and 315 degrees** relative to the **+U axis**. No synthetic or "additional" samples are required in this native 4fsc representation; that concept is only needed when converting the sampled frame into an auxiliary orthogonal digital line representation.

The analogue horizontal timing reference is **0H**, defined as the midpoint of the leading edge of sync. Any mapping between analogue time and stored digital samples must track the **analogue line period from 0H to 0H** within this non-orthogonal frame-repeating structure, not an imposed line-orthogonal numbering scheme.

**Stored frame alignment (normative):** Stored PAL frames are 0H-aligned as defined in [Stored Frame and Line Origin](#stored-frame-and-line-origin-normative): the first stored sample of each frame is the first sampling instant at or after 0H of broadcast frame line 1 (stored frame line 0). Because the PAL 4fsc lattice is non-orthogonal, this origin is defined only at the frame level. It differs from EBU Tech. 3280-E's digital line structure, in which the digital line begins with the digital active line and the frame-1 line-1 sync edge falls midway between samples 957 and 958.

**PAL 4-frame sequence:**

| Frame in sequence | Description |
| ----------------- | ----------- |
| 1                 | Reference frame of the PAL sequence at 0 degrees Sc/H |
| 2                 | Second frame in the PAL sequence |
| 3                 | Third frame in the PAL sequence |
| 4                 | Fourth frame in the PAL sequence; the next frame repeats as frame 1 |

PAL is a **625-line, 25 frames/s** system. The Sc/H relationship repeats every **4 frames**. In this specification, PAL sequence position is described only by the repeating frame numbers **1-4**. Each stored PAL frame is the complete 625-line frame sample sequence within the non-orthogonal 4fsc sample lattice; the frame numbering identifies sequence position only and does **not** imply an orthogonal digital display or storage raster.

**Digital vertical blanking interval:**

> *(Informational — EBU Tech. 3280-E §1.3.2):* For PAL, the digital vertical blanking interval extends (line numbers use EBU Tech. 3280-E's 1-indexed frame line convention, counting sequentially across the full 625-line frame; sample numbers use EBU Tech. 3280-E's digital-line coordinates, in which sample 0 is the start of the digital active line — not this specification's stored coordinates):
>
> - frame line 623 sample 382 to frame line 5 sample 947 (inclusive, wrapping across the frame boundary).
> - frame line 310 sample 948 to frame line 317 sample 947 (inclusive).

**Exact frame size** (normative when the declared Signal State Preset is time-base stable at the standard 4×fsc sample rate — i.e., `STANDARD_TBC_LOCKED` or `STANDARD_TBC_UNLOCKED`; when either condition does not hold, this value is not normative):

| Frames/sample count | Samples | Bytes |
| ------------------- | ------- | ----- |
| PAL frame           | **709,379** | **1,418,758** |

Because the PAL 4fsc sampling structure is non-orthogonal, this specification defines the normative PAL sample count only at the **frame** level. The exact normative constraint is the **frame total** of **709,379 samples** repeating at frame rate.

Bytes = samples × 2 (each sample is one 16-bit little-endian word).

**Colour frame sequence:**

The PAL colour sequence cycles over **4 frames** and then repeats.

> *(Informational — EBU Tech. 3280-E §1.1.1 and Fig. 1):* At 0° Sc/H (the normative PAL sampling phase), the +U axis of the subcarrier is at zero phase relative to the horizontal timing reference point (0H) at the start of frame 1 in the PAL sequence. The colour burst phase rotates through the standard PAL progression across the 4-frame cycle. Any break in the expected Sc/H phase progression between consecutive stored frames indicates a discontinuity in the PAL sequence.

> *(Informational — ld-decode/vhs-decode convention):* PAL TBC output from `ld-decode` conventionally starts at the midpoint of frame 1 in the EBU PAL sequence rather than at its first line. Consumers that care about exact PAL sequence origin should account for that half-frame offset.

---

## Preset: `NTSC`

**External standards:**

- [SMPTE 170M-2004](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-170M-2004/SMPTE-170M-2004.md): analogue NTSC studio composite signal definition for 525-line, 59.94-field operation.
- [SMPTE 244M-2003](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-244M-2003/SMPTE-244M-2003.md): 4fsc digital representation used for NTSC sample levels, line structure, sample phase, and colour-frame relationship.

**Sampling rate:**

| Formula | Frequency |
|---------|-----------|
| 4 × 525 × (30000/1001) × (455/2) | **14,318,181.8… Hz** |

**Sample level table (SMPTE 244M-2003 §4.2.1 / SMPTE 170M-2004 Table 1) — applicable when Sample Encoding Preset is `CVBS_U10_4FSC` or `CVBS_U16_4FSC`:**

| Level            | 10-bit Decimal | 10-bit Hex | Notes                           |
| ---------------- | -------------- | ---------- | ------------------------------- |
| Protected min    | 0–3            | 000h–003h  | Must never appear; reserved     |
| Sync tip         | 16             | 010h       | Minimum legal sample value      |
| Blanking         | 240            | 0F0h       | Zero-signal reference (0 IRE; SMPTE 244M-2003 §4.2.1) |
| Black            | 282            | 11Ah       | +7.5 IRE setup pedestal (SMPTE 170M-2004 Table 1; 240 + 7.5 × 5.6 = 282) |
| White (100%)     | 800            | 320h       | 100% white (SMPTE 244M-2003 §4.2.1) |
| Peak (w/ chroma) | 1019           | 3FBh       | Maximum legal sample value (SMPTE 244M-2003 §4.2.4) |
| Protected max    | 1020–1023      | 3FCh–3FFh  | Must never appear; reserved     |

> **Black level and ST.0244 scope:** SMPTE 244M-2003 (standardised as SMPTE ST 244, sometimes cited as "ST.0244") defines blanking (0 IRE = 240) and white (100 IRE = 800) as its normative signal reference levels. It does **not** independently mandate a picture black level. The +7.5 IRE NTSC setup pedestal shown above comes from SMPTE 170M-2004 Table 1 (and is treated as optional by ITU-R BT.470-6 §2.5.3). Producers that apply no setup (NTSC-J style) will have black at 240 (= blanking). This distinction is recorded via the `black_level` override in the metadata when it differs from the preset default.

**Horizontal line structure:**

| Total samples/line | Digital active samples | Digital blanking samples |
| ------------------ | ---------------------- | ------------------------ |
| 910 (exact)        | 768                    | 142                      |

> *(Informational — SMPTE 244M-2003 §4.1.1):* For NTSC, sampling is orthogonal; all lines carry exactly 910 samples. The half-amplitude point of the leading (falling) horizontal sync edge falls between samples 784 and 785. The digital active line is samples 0–767; the digital horizontal blanking interval is samples 768–909. *(These sample numbers are SMPTE 244M-2003 digital-line coordinates, not this specification's stored coordinates — see below.)*

**Stored line alignment (normative):** Stored NTSC lines are 0H-aligned as defined in [Stored Frame and Line Origin](#stored-frame-and-line-origin-normative): sample 0 of each stored 910-sample line is the first sampling instant at or after that line's 0H, so the horizontal sync pulse and colour burst occupy the start of the stored line. Relative to SMPTE 244M-2003's digital line structure this places 0H between stored samples −1 and 0 rather than between samples 784 and 785; equivalently, SMPTE 244M digital-line sample 0 corresponds to **stored sample 125**, and the digital active line (SMPTE samples 0–767) corresponds to **stored samples 125–892** of the same line.

**NTSC 2-frame sequence:**

| Frame in sequence | Description |
| ----------------- | ----------- |
| A                 | First frame of the NTSC colour sequence |
| B                 | Second frame of the NTSC colour sequence; the next frame repeats as A |

NTSC is a **525-line, 30000/1001 frames/s** system. The SC/H relationship repeats every **2 frames**. In this specification, NTSC sequence position is described only by the alternating frame labels **A** and **B**.

> *(Informational — SMPTE 244M-2003 §4.1, §4.1.2):* SMPTE 244M-2003 describes the same repeating NTSC sequence using a lower-level phase progression. This specification instead normalises that description to a **2-frame** cycle because storage in this format is frame-based. At 0° SC/H, sample 0 of frame line 10 in colour frame A is an I-axis (+123°) sample. *(Line and sample numbers in this block use SMPTE 244M-2003's conventions: 1-indexed frame lines counted sequentially across the full 525-line frame, and digital-line sample coordinates.)* In this specification's stored, 0H-aligned coordinates that reference sample is **sample 125 of stored frame line 9** (0-based) in colour frame A.

**Digital vertical blanking interval:**

> *(Informational — SMPTE 244M-2003 §5.4.1):* For NTSC, the digital vertical blanking interval extends (line numbers use SMPTE 244M-2003's 1-indexed frame line convention, counting sequentially across the full 525-line frame; sample numbers use SMPTE 244M-2003's digital-line coordinates, in which sample 0 is the start of the digital active line — not this specification's stored coordinates):
>
> - frame line 525 sample 768 to frame line 9 sample 767 (inclusive, wrapping across the frame boundary).
> - frame line 263 sample 313 to frame line 272 sample 767 (inclusive).

**Exact frame size** (normative when the declared Signal State Preset is time-base stable at the standard 4×fsc sample rate — i.e., `STANDARD_TBC_LOCKED` or `STANDARD_TBC_UNLOCKED`; when either condition does not hold, this value is not normative):

| Frames/sample count | Samples | Bytes |
| ------------------- | ------- | ----- |
| NTSC frame          | **477,750** | **955,500** |

Derivation:
- **NTSC frame:** 525 × 910 = **477,750 samples**

Bytes = samples × 2 (each sample is one 16-bit little-endian word).

**Colour frame sequence:**

The NTSC colour sequence cycles over **2 frames**, conventionally labelled **A** and **B**, and then repeats.

> *(Informational — SMPTE 244M-2003 §3.2, §4.1.2):* At 0° SC/H (the normative NTSC sampling phase), sample 0 of frame line 10 in colour frame A is an I-axis (+123°) sample — in this specification's stored, 0H-aligned coordinates, **sample 125 of stored frame line 9** (0-based). Comparing the measured burst phase at that reference point against the expected value for colour frame **A** or **B** identifies position within the 2-frame sequence. A phase discontinuity between consecutive stored frames indicates a colour frame sequence break. *(SMPTE line and sample numbers use SMPTE 244M-2003's 1-indexed frame line convention and digital-line sample coordinates.)*

> *(Informational — ld-decode/vhs-decode convention):* NTSC TBC output from `ld-decode` conventionally starts with colour frame **A**.

---

## Preset: `PAL_M`

**External standards:**

- [ITU-R BT.1700-1 (2005)](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/BT-1700-E/BT-1700-E.md), Annex 1 Part B: analogue 525-line PAL signal format used for PAL-M timing, subcarrier frequency relationship, PAL colour modulation, and signal levels.
- [SMPTE 244M-2003](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-244M-2003/SMPTE-244M-2003.md): 525-line 4fsc digital composite coding reference used by this storage preset for the 10-bit sample-value mapping.

PAL-M uses 525-line/60 Hz timing with PAL colour subcarrier modulation, as described by the 525 PAL signal format in ITU-R BT.1700. The subcarrier relationship is `fsc = 909/4 fH`, so native 4fsc sampling gives 909 samples per line. The colour sequence repeats every **4 frames**, longer than NTSC's 2-frame colour sequence, due to the PAL colour encoding.

**Sampling rate:**

| Formula | Frequency |
|---------|-----------|
| 4 × 525 × (30000/1001) × (909/4) | **14,302,448.1… Hz** |

**Sample level table (SMPTE 244M-compatible 10-bit coding levels / SMPTE 170M-2004 Table 1) — applicable when Sample Encoding Preset is `CVBS_U10_4FSC` or `CVBS_U16_4FSC`:**

| Level            | 10-bit Decimal | 10-bit Hex | Notes                           |
| ---------------- | -------------- | ---------- | ------------------------------- |
| Protected min    | 0–3            | 000h–003h  | Must never appear; reserved     |
| Sync tip         | 16             | 010h       | Minimum legal sample value      |
| Blanking         | 240            | 0F0h       | Zero-signal reference (0 IRE)   |
| Black            | 282            | 11Ah       | +7.5 IRE setup pedestal (SMPTE 170M-2004 Table 1; 240 + 7.5 × 5.6 = 282) |
| White (100%)     | 800            | 320h       | 100% white                      |
| Peak (w/ chroma) | 1019           | 3FBh       | Maximum legal sample value      |
| Protected max    | 1020–1023      | 3FCh–3FFh  | Must never appear; reserved     |

> **Black level note:** As with the NTSC preset, the +7.5 IRE setup pedestal comes from SMPTE 170M-2004 Table 1, not from the 4fsc coding standard. PAL-M sources with no setup (black at blanking = 240) should signal this via the `black_level` metadata override.

**Horizontal line structure:**

| Total samples/line | Digital active samples | Digital blanking samples |
| ------------------ | ---------------------- | ------------------------ |
| 909 (exact)        | 768                    | 141                      |

**Stored line alignment (normative):** Stored PAL-M lines are 0H-aligned as defined in [Stored Frame and Line Origin](#stored-frame-and-line-origin-normative): sample 0 of each stored 909-sample line is the first sampling instant at or after that line's 0H, so the horizontal sync pulse and colour burst occupy the start of the stored line.

**PAL-M 4-frame sequence:**

| Frame in sequence | Description |
| ----------------- | ----------- |
| 1                 | Reference frame of the PAL-M sequence |
| 2                 | Second frame in the PAL-M sequence |
| 3                 | Third frame in the PAL-M sequence |
| 4                 | Fourth frame in the PAL-M sequence; the next frame repeats as frame 1 |

PAL-M is a **525-line, 30000/1001 frames/s** system. The SC/H relationship repeats every **4 frames**.

**Exact frame size** (normative when the declared Signal State Preset is time-base stable at the standard 4×fsc sample rate — i.e., `STANDARD_TBC_LOCKED` or `STANDARD_TBC_UNLOCKED`; when either condition does not hold, this value is not normative):

| Frames/sample count | Samples | Bytes |
| ------------------- | ------- | ----- |
| PAL-M frame         | **477,225** | **954,450** |

Derivation:
- **PAL-M frame:** 525 × 909 = **477,225 samples**

Bytes = samples × 2 (each sample is one 16-bit little-endian word).

**Colour frame sequence:**

Despite using 525-line/60 Hz timing, PAL-M uses a **4-frame** colour sequence rather than NTSC's 2-frame sequence, due to the PAL colour subcarrier modulation. Phase verification follows the PAL SC/H approach applied to the 525-line/60 Hz timing structure.

---

## Frame Ordering and Phase Verification

Video Standard Presets are frame-described. Frames are stored sequentially in file order. Position within the colour frame sequence is not encoded in-band; consumers determine sequence position by measuring burst phase and checking that successive frames preserve the expected preset-specific progression.

- **PAL:** verify against the 4-frame progression described in the PAL preset section above.
- **NTSC:** verify against the 2-frame progression described in the NTSC preset section above.
- **PAL_M:** verify against the 4-frame progression described in the PAL_M preset section above.

`ld-decode` and `vhs-decode` frame-sequence conventions are documented in the informational notes in each preset section.

**Frame boundary and length integrity (normative):**

When the declared Signal State Preset is time-base stable at the standard 4×fsc sample rate (`STANDARD_TBC_LOCKED` or `STANDARD_TBC_UNLOCKED`), producers shall preserve frame boundaries and exact frame sample counts throughout the stream: every stored field and frame shall be complete and exact-sized, regardless of whether the content contains sequence discontinuities.

For these presets, the exact frame sample counts are:
- **PAL:** 709,379 samples/frame
- **NTSC:** 477,750 samples/frame
- **PAL_M:** 477,225 samples/frame

**Sequence progression integrity (normative):**

When the [`sequence_continuous`](index.md#sequence_continuous) metadata field is `TRUE`, each stored frame shall preserve the correct preset-specific frame progression, with no skipped frames, duplicated frames, or boundary shifts that would alter sequence alignment, across the entire file.

If a producer cannot guarantee this continuity (for example because source skips, repeated source frames, dropouts in timing recovery, or capture pauses prevent reliable frame-to-frame sequence continuity), the producer shall declare `sequence_continuous = FALSE` (or `NULL` if unknown) rather than downgrading the Signal State Preset; each contiguous run of frames between discontinuities shall still honour the declared presets in full.

---

## Non-Standard Value Signaling

- **Declared non-standard values:** When `has_nonstandard_values = TRUE` in core metadata, consumers must allow non-standard sample values without treating them as automatic compliance errors.
- **Source-specific interpretation:** The metadata flag indicates presence only. The exact type, location, and semantics of non-standard content (for example PAL pilot bursts, NTSC additional bursts, or other anomalies) are source-specific and may be documented in `capture_notes` or a producer extension format.
- **Encoding interaction:** Depending on the declared Sample Encoding Preset and processing path, non-standard values may be preserved directly, quantized, or clipped before storage.
- **DC Offset:** Not required. The signed 16-bit storage format provides sufficient negative headroom below 0 to accommodate chroma excursions without clipping.


