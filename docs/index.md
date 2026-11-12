# CVBS File Format Specification

---

## Introduction

This document defines the **CVBS File Format** for use with the `ld-decode`, `vhs-decode` and related projects. **CVBS** (Colour, Video, Blank, and Sync) describes the full class of analogue video signal — both composite (luma and chroma combined into a single signal) and component YC representations carry the same Colour, Video, Blank, and Sync elements and are therefore both CVBS signals.

The format is organised around three independent **preset systems**, each of which captures a distinct dimension of the signal:

- [**Video Standard Presets**](#video-standard-presets): define the timing and structural parameters of the video signal — line counts, field rates, horizontal sample structure, colour field sequence, and the normative sample level tables for standards-compliant signals.
- [**Sample Encoding Presets**](#sample-encoding-presets): define the bit depth, word format, and amplitude mapping used when the sample data was recorded.
- [**Signal State Presets**](#signal-state-presets): define the sampling and processing state of the signal at the time of storage — whether a standard 4×fsc sample rate was used, whether the signal is time-base stable, and whether the sampling is phase-locked to the colour subcarrier.

Every CVBS file is described by one preset from each system. Together they specify how to interpret sample amplitudes, when normative field sizing rules apply, and how reliable phase and level measurements are expected to be.

The format accommodates **non-standard cases** (e.g., LaserDisc PAL pilot bursts and variations such as NTSC-J) through metadata flags rather than structural changes.

---

## File Naming Convention

Each file type uses a distinct extension:

- **Composite CVBS:** `<basename>.cvbs`
- **Dual-File YC CVBS (Luma):** `<basename>.cvbsy`
- **Dual-File YC CVBS (Chroma):** `<basename>.cvbsc`

---

## Video Data Format

### Sample Encoding

The physical encoding and amplitude mapping of samples are defined by the [**Sample Encoding Preset**](#sample-encoding-presets) declared for the file. The main properties governed by the sample encoding preset are:

- **Word format:** the integer width, signedness, and byte order of each stored sample word.
- **Amplitude mapping:** the relationship between the stored integer value and the analogue signal amplitude — specifically, where sync tip, blanking, black, white, and peak levels fall within the integer range.
- **Headroom:** whether negative or positive values beyond the nominal 0–1023 range are meaningful.

The standard encoding preset (`CVBS_U10_4FSC`) represents the normative production output of `ld-decode`, `vhs-decode`, and similar tools. The `CVBS_U16_4FSC` preset represents the same 10-bit domain values stored in an unsigned 16-bit container with 6 zero-padded LSBs (`value << 6`). The raw capture presets represent unscaled ADC output from hardware capturers such as the DomesdayDuplicator. See [sample-encoding-presets](sample-encoding-presets.md) for the full definitions.

This specification supports two CVBS (Colour, Video, Blank, and Sync) storage representations, which differ only in how colour information is carried:

- **Composite CVBS:** Single file containing luma and chroma combined into one signal.
- **Dual-File YC CVBS:** Separate files for luma (Y) and chroma (C), keeping the colour components separated.

The specific sample range — sync tip, blanking, black, white, and peak levels together with any reserved protected values — is defined by the declared [Sample Encoding Preset](#sample-encoding-presets).

For YC (`signal_type='yc'`) files using a Sample Encoding Preset with predefined 10-bit level mapping (for example `CVBS_U10_4FSC`, `CVBS_U16_4FSC`, `CVBS_TPG21_4FSC`, and `CVBS_S16_4FSC`), luma (`.cvbsy`) follows the same level definitions as composite output, while chroma (`.cvbsc`) is represented in a centred 10-bit domain with chroma zero at sample value 512. Any preset-defined integer-domain translation applied to luma must also be applied identically to chroma in that preset's domain.

### File Layout

Video data is stored as a sequence of **frames**, where each frame contains two sequential fields. A **frame** in this context is the smallest navigatable unit in the file and consists of a pair of field samples; this should not be confused with the visual rendering of interlaced video (a single interlaced video frame displayed to the user). No additional framing or headers are included:

```
[Frame 1 Video Data: 2 × Field Data = M bytes]
[Frame 2 Video Data: 2 × Field Data = M bytes]
...
```

The horizontal and vertical origin of each stored frame — 0H-aligned lines, with frame line 0 at the first line of field 1 — is defined normatively in [Stored Frame and Line Origin](video-standard-presets.md#stored-frame-and-line-origin-normative). Note that this differs from the digital line structure of the 4fsc interface standards (SMPTE 244M-2003, EBU Tech. 3280-E), in which the digital line begins with the digital active line.

### Frame Ordering and Sequencing

Frames are stored sequentially in the file with no embedded markers identifying the colour field sequence position or validating correct sequential ordering. In this specification, ordering has two independent dimensions: (1) ordering of frames relative to other frames, and (2) ordering of the two fields inside each frame. **No guarantee exists that either dimension is correct.** A frame may contain the expected pair of fields but with intra-frame field order reversed. Due to the nature of RF capture and physical media characteristics — including disc jumps, scratches, media pauses, physical defects, and frame dropouts — the producer may have no reliable way to verify that captured frame and field ordering maintain proper sequential continuity.

When present, the [`sequence_continuous`](#sequence_continuous) metadata field declares whether the producer asserts that the stored content is one unbroken sequence; when it is `TRUE`, consumers may rely on sequential continuity without verification.

If frame-level reordering, field-level analysis, phase verification, or dropout detection is required, those concerns are the responsibility of the consumer application. The file format itself operates at the frame level as the smallest unit of storage and navigation; field-level manipulation and validation are outside the scope of the format specification.

---

## Video Standard Presets

### Preset System

A **Video Standard Preset** is a named configuration that fully defines all timing and structural parameters for a video standard: sampling rate, line counts, field structure, sample level tables, colour field sequence length, and references to the external standards on which the preset is based. These presets do **not** define the physical sample word format or the processing state of the signal — those are covered by the [Sample Encoding Preset](#sample-encoding-presets) and [Signal State Preset](#signal-state-presets) respectively.

The `preset` field in the `cvbs_file` metadata table (see the [`cvbs_file` table](#cvbs_file-table)) identifies which Video Standard Preset applies to a given CVBS file. Consumers must implement a preset in full; an unrecognised preset name must not be silently interpreted — the consumer must refuse to process the file or report an error.

### Preset Definitions

Full definitions — including naming convention, sampling rates, sample level tables, horizontal and vertical structure, and normative sample-count constraints — are in [video-standard-presets](video-standard-presets.md).

### Non-Standard Value Signaling

Certain captures may contain sample values that fall outside the nominal standard-protected ranges. See [video-standard-presets](video-standard-presets.md#non-standard-value-signaling) for details.

### Field Ordering and Phase Verification

Because frames consist of sequential field pairs and frame ordering is not guaranteed, consumers performing field-level analysis must validate field sequence position, field order, and phase continuity against the declared preset. They may optionally reorder fields and/or frames to establish continuity. The [`sequence_continuous`](#sequence_continuous) metadata field, when present, declares whether the producer asserts continuity across the whole file.

**Note:** The choice to perform these validations and reorderings is entirely optional and application-dependent. The file format itself makes no assertions about frame ordering correctness.

Preset-specific progression rules and `ld-decode`/`vhs-decode` field-ordering conventions are defined in [video-standard-presets](video-standard-presets.md#frame-ordering-and-phase-verification).

---

## Metadata Schema

The metadata file is **optional**. A CVBS file is self-contained as raw sample data and can be processed without a metadata file provided the consumer obtains the necessary parameters (such as video standard, frame count, and colour sequence position) by other means — for example, from user-supplied command-line arguments or application settings.

When present, metadata is stored in a **separate `.meta` file** alongside the video data files.

- **Metadata:** `<basename>.meta`

The metadata file is a **SQLite database** containing the core metadata table defined below.

### SQLite Metadata Schema

```sql
PRAGMA user_version = 12;

CREATE TABLE cvbs_file (
    cvbs_file_id                INTEGER PRIMARY KEY,
    preset                      TEXT    NOT NULL
        CHECK (preset IN ('NTSC', 'PAL', 'PAL_M', 'SECAM')),
    sample_encoding_preset      TEXT    NOT NULL
        CHECK (sample_encoding_preset IN ('CVBS_U10_4FSC', 'CVBS_U16_4FSC', 'RAW_S16_28M', 'RAW_S16_40M', 'CVBS_TPG21_4FSC', 'CVBS_S16_4FSC')),
    signal_state_preset         TEXT    NOT NULL
        CHECK (signal_state_preset IN (
            'STANDARD_STABLE_LOCKED',
            'STANDARD_STABLE_UNLOCKED',
            'STANDARD_RAW',
            'NONSTANDARD_STABLE_LOCKED',
            'NONSTANDARD_STABLE_UNLOCKED',
            'NONSTANDARD_RAW'
        )),
    sequence_continuous         BOOLEAN,
    signal_type                 TEXT    NOT NULL
        CHECK (signal_type IN ('composite', 'yc')),
    decoder                     TEXT    NOT NULL,
    git_branch                  TEXT,
    git_commit                  TEXT,
    number_of_sequential_frames INTEGER
        CHECK (number_of_sequential_frames IS NULL OR number_of_sequential_frames >= 1),
    black_level                 INTEGER,
    has_nonstandard_values      BOOLEAN,
    capture_notes               TEXT
);

CREATE TABLE audio_channel_pair (
    channel_pair                INTEGER PRIMARY KEY
        CHECK (channel_pair BETWEEN 0 AND 7),
    description                 TEXT
);
```

**Schema version 12:** `user_version = 12` adds `'SECAM'` to the `preset` CHECK constraint; the schema is otherwise identical to `user_version = 11`. A `user_version = 11` database remains valid and is interpreted unchanged under version 12 semantics. Producers writing a `SECAM` file must write `user_version = 12`; consumers that do not implement the `SECAM` preset must reject such files as described in [Preset System](#preset-system).

### `cvbs_file` Table

The `cvbs_file` table records file-level metadata. There is one row per CVBS file.

#### `cvbs_file_id`

- **Type:** INTEGER, PRIMARY KEY
- **Nullable:** No
- **Description:** Unique identifier for the CVBS file record within this metadata database.

#### `preset`

- **Type:** TEXT
- **Nullable:** No
- **Range:** Any Video Standard Preset name defined in [video-standard-presets](video-standard-presets.md)
- **Description:** Identifies the Video Standard Preset that applies to this CVBS file. The preset name uniquely determines all timing and structural parameters — sampling rate, line counts, field structure, sample level tables, and colour field sequence length. Consumers must implement the named preset in full; an unrecognised preset name must not be silently interpreted.

#### `sample_encoding_preset`

- **Type:** TEXT
- **Nullable:** No
- **Range:** Any Sample Encoding Preset name defined in [sample-encoding-presets](sample-encoding-presets.md)
- **Description:** Identifies the Sample Encoding Preset that defines the physical word format and amplitude mapping of the sample data.

#### `signal_state_preset`

- **Type:** TEXT
- **Nullable:** No
- **Range:** Any Signal State Preset name defined in [signal-state-presets](signal-state-presets.md)
- **Description:** Identifies the Signal State Preset that describes the sampling and processing state of the signal at the time of storage — whether the sample rate is standard 4×fsc, whether the signal is time-base stable, and whether the sampling is phase-locked to the colour subcarrier. The preset describes the capture/processing chain only; continuity of the stored content is declared separately by [`sequence_continuous`](#sequence_continuous).

#### `sequence_continuous`

- **Type:** BOOLEAN
- **Nullable:** Yes
- **Description:** Declares whether the stored content is a single unbroken sequence. When `TRUE`, the producer asserts that: (1) the stored samples represent one temporally gapless capture, with no missing, repeated, or reordered lines, fields, or frames anywhere in the file; (2) when the declared Signal State Preset is time-base stable, the field/frame sequence advances by exactly one position per stored frame; and (3) when the declared Signal State Preset is phase-locked, the colour field sequence position and subcarrier phase progress according to the declared Video Standard Preset across every frame boundary. When `FALSE`, at least one discontinuity exists (for example a disc skip or capture pause); each contiguous run of frames between discontinuities still honours the declared presets in full, and consumers must re-establish colour sequence position after each discontinuity. `NULL` means continuity is unknown or was not assessed; consumers must not assume continuity. This field is independent of the Signal State Preset: a discontinuity does not affect the preset's structural guarantees (when the preset is time-base stable, every stored field and frame remains complete and exact-sized regardless of this flag), and producers must not downgrade the preset to signal a discontinuity. Locating individual discontinuities is the domain of producer extension metadata (see [Producer Extension Metadata](#producer-extension-metadata)).

#### `signal_type`

- **Type:** TEXT
- **Nullable:** No
- **Range:** `'composite'`, `'yc'`
- **Description:** Declares whether the accompanying CVBS data file is a composite signal (`'composite'`, paired with a `.cvbs` file) or one file of a dual-file YC pair (`'yc'`, paired with a `.cvbsy` or `.cvbsc` file). This allows consumers to determine the project type explicitly without relying on file presence detection.

#### `decoder`

- **Type:** TEXT
- **Nullable:** No
- **Range:** `'ld-decode'`, `'vhs-decode'`, `'cvbs-encode'`, `'cvbs-decode'`, `'other'`
- **Description:** The tool that produced the CVBS file. Used to identify the software source and inform interpretation of any tool-specific behaviours.

#### `git_branch`

- **Type:** TEXT
- **Nullable:** Yes
- **Description:** The Git branch name of the decoder tool at the time of capture. `NULL` if not recorded.

#### `git_commit`

- **Type:** TEXT
- **Nullable:** Yes
- **Description:** The Git commit hash of the decoder tool at the time of capture. `NULL` if not recorded. Enables exact reproduction of the decoding conditions.

#### `number_of_sequential_frames`

- **Type:** INTEGER
- **Nullable:** Yes
- **Range:** ≥ 1, or `NULL`
- **Description:** The total number of frames stored in the accompanying CVBS data file(s), counted sequentially from the first to the last. `NULL` if the frame count is not known at the time of writing — for example, when the file is produced by a raw sampling device that does not segment or count frames during capture. Consumers encountering a `NULL` value must determine the frame count by parsing the file directly.

#### `black_level`

- **Type:** INTEGER
- **Nullable:** Yes
- **Range:** `NULL`, or an INTEGER value interpreted according to the declared presets
- **Description:** Optional override for non-standard black levels. `NULL` means no explicit override is provided and consumers should use the default black level behavior defined by the declared Video Standard Preset and Sample Encoding Preset (see [video-standard-presets](video-standard-presets.md) and [sample-encoding-presets](sample-encoding-presets.md)). A non-`NULL` value provides an explicit black-level override in the integer domain of the declared Sample Encoding Preset. For Sample Encoding Presets where black-level calibration is not defined or not meaningful (for example raw capture presets), this field should be `NULL`.

#### `has_nonstandard_values`

- **Type:** BOOLEAN
- **Nullable:** Yes
- **Description:** Indicates that the producer observed or intentionally generated non-standard sample values relative to the declared Video Standard Preset and Sample Encoding Preset expectations (for example values outside the nominal legal 10-bit domain, including blanking-interval anomalies such as additional bursts). This flag declares presence only; it does not encode the exact signal type or location. Some sample encodings may preserve such values directly while others may clip or quantize them. `NULL` means not applicable or not known.

#### `capture_notes`

- **Type:** TEXT
- **Nullable:** Yes
- **Description:** Free-form human-readable notes about the capture. May include source material description, known non-compliance (e.g., LaserDisc PAL pilot bursts), equipment details, or other contextual information. `NULL` if no notes are recorded.

### `audio_channel_pair` Table

The `audio_channel_pair` table records per-channel-pair audio metadata. There is exactly one row per channel pair file present alongside the CVBS data file (see [Audio Data](#audio-data)); files without audio have no rows.

A **channel pair** is two digital audio channels, generally derived from the same AES audio source, as defined by [SMPTE 272M-1994](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-272M-1994/SMPTE-272M-1994.md) §3.11. All audio properties — sample rate, resolution, and synchronisation — are fixed by this specification (see [Audio Data](#audio-data)), so no per-pair format fields are required or permitted.

#### `channel_pair`

- **Type:** INTEGER, PRIMARY KEY
- **Nullable:** No
- **Range:** 0–7
- **Description:** The channel pair number. Must match the single-digit suffix of the corresponding `<basename>_audio_<channel_pair>.wav` file. Channel pair numbers need not be contiguous, but every row must correspond to an existing channel pair file and vice versa. Channel pair *p* carries SMPTE 272M audio channels 2*p*+1 and 2*p*+2 (see [Channel Assignment](audio-data.md#channel-assignment)).

#### `description`

- **Type:** TEXT
- **Nullable:** Yes
- **Description:** Human-readable channel pair description (for example `Analogue stereo`, `EFM digital audio`, `Commentary`). `NULL` if no description is recorded; consumers should then derive a display name from the channel pair number.

### Producer Extension Metadata

Per-frame, per-field, and per-sample annotations (for example: frame-level dropout flags, dropout maps, confidence scores, or producer-specific segmentation data) are intentionally **not part of the core CVBS metadata schema**.

When a producer needs to store such additional information, it must be written in a **separate, explicitly defined extension format** (for example a sidecar file or extension database with its own schema/versioning document).

The core standard in this document defines only the `cvbs_file` metadata table. Consumers must not assume that any producer-specific extension metadata exists unless that extension format is explicitly declared and available.

A standard frame-based dropout extension is defined in [dropout-extension-format](extensions/dropout-extension-format.md).

---

## Sample Encoding Presets

A **Sample Encoding Preset** defines the physical word format and amplitude mapping applied to every sample in the file: the integer width, signedness, byte order, and the relationship between stored values and analogue signal levels (sync tip, blanking, black, white, peak). The preset name is stored in the `sample_encoding_preset` field of the `cvbs_file` metadata table (see the [`cvbs_file` table](#cvbs_file-table)).

Full definitions: [sample-encoding-presets](sample-encoding-presets.md)

---

## Signal State Presets

A **Signal State Preset** defines the sampling and processing state of the signal at the time of storage along three independent axes: sample rate (standard 4×fsc vs. non-standard), time-base stable (yes vs. no), and phase locked (yes vs. no). The combination governs whether normative sample-count constraints apply, whether signal level compliance is required, and whether the content is sampled at the standard subcarrier-reference-locked phase points. The preset name is stored in the `signal_state_preset` field of the `cvbs_file` metadata table (see the [`cvbs_file` table](#cvbs_file-table)). Continuity of the stored content — whether the sequence is free of breaks such as disc skips — is not a preset axis; it is declared by the [`sequence_continuous`](#sequence_continuous) metadata field.

Full definitions: [signal-state-presets](signal-state-presets.md)

---

## Audio Data

Audio follows the preferred implementation of [SMPTE 272M-1994](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-272M-1994/SMPTE-272M-1994.md): up to **16 audio channels** in **8 channel pairs**, each channel pair stored as a separate 24-bit, 48 kHz stereo WAV file (`<basename>_audio_<channel_pair>.wav`), clock locked (synchronous) to video. Per-channel-pair metadata is recorded in the [`audio_channel_pair` table](#audio_channel_pair-table).

Full definitions: [audio-data](audio-data.md)

---

## Recommended Testing Limits (Informative)

!!! note "Informative"
    This section is informative only. Nothing in it is a conformance requirement; it provides recommended design and testing targets for producer and consumer implementations.

The file format imposes no maximum on file length or frame count. However, the physical media this format serves — LaserDisc and analogue video tape — produce captures of predictable, and substantial, size. Producers and consumers should be designed and tested against these realistic sizes rather than short synthetic samples.

The table below lists expected frame counts and approximate composite file sizes for common source material. Sizes are calculated for Sample Encoding Presets using 16-bit words (for example `CVBS_U10_4FSC`), where an `NTSC` frame is 477,750 samples (955,500 bytes) and a `PAL` frame is 709,379 samples (1,418,758 bytes):

| Source                     | Typical duration         | Frames (`NTSC`) | Frames (`PAL`) | Approx. size (`NTSC` / `PAL`) |
|----------------------------|--------------------------|-----------------|----------------|-------------------------------|
| LaserDisc CAV, one side    | 30 min NTSC / 36 min PAL | 54,000          | 54,000         | ~52 GB / ~77 GB               |
| LaserDisc CLV, one side    | 60 min NTSC / 64 min PAL | ~107,900        | 96,000         | ~103 GB / ~136 GB             |
| VHS SP (T-120 / E-180)     | 2 h NTSC / 3 h PAL       | ~215,800        | 270,000        | ~206 GB / ~383 GB             |
| VHS LP/EP, worst case      | 6 h                      | ~647,400        | 540,000        | ~619 GB / ~766 GB             |

Based on these sizes, the following are recommended:

- **Testing limit:** Producers and consumers should be verified against files of at least **650,000 frames** (approximately 6 hours of NTSC video, ~620 GB of composite sample data) — the practical worst case for the supported source media.
- **64-bit file handling:** At roughly 1 MB per frame, even a few minutes of video exceeds 4 GiB. Implementations should use 64-bit file offsets and sizes throughout, and should process files as streams rather than loading them into memory in their entirety.
- **Storage multipliers:** Dual-file YC storage approximately doubles the sample data size (separate `.cvbsy` and `.cvbsc` files). Each audio channel pair adds approximately 6.2 GB over 6 hours (48 kHz × 24-bit × 2 channels). A channel pair passes the 4 GiB RIFF WAV limit at approximately 4 hours 8 minutes, beyond which it is written as RF64 (see [RF64 for Long Recordings](audio-data.md#rf64-for-long-recordings)).

---

