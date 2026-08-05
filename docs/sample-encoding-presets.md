# Sample Encoding Preset Definitions

This document is part of the [CVBS File Format Specification](index.md). It contains the normative Sample Encoding Preset definitions referenced in the [Sample Encoding Presets](index.md#sample-encoding-presets) section of that specification.

**Naming convention:** Sample Encoding Preset names use only uppercase ASCII letters, digits, and underscores. Names derived from a broadcast standard use the format `CVBS_<ENCODING>`; names for raw hardware captures use the format `RAW_<FORMAT>_<RATE>`.

---

## Preset: `CVBS_U10_4FSC`

**Applicable sources:** Standards-compliant output of `ld-decode`, `vhs-decode`, `cvbs-encode`, and similar tools targeting EBU 3280 or SMPTE ST.0244.

> **Note on SMPTE ST.0244 scope:** SMPTE ST.0244 (the 4fsc digital composite standard, formerly SMPTE 244M) defines the sample rate, word format, sync tip, blanking reference (0 IRE), and 100% white level. It does **not** define black level. The black level — for example the +7.5 IRE NTSC setup pedestal — is an analogue composite signal parameter defined by SMPTE 170M-2004 Table 1 (and treated as optional by ITU-R BT.470-6). Black level for this encoding is determined by the declared Video Standard Preset (see [video-standard-presets](video-standard-presets.md)), not by ST.0244 compliance. Metadata-based black level overrides (e.g. the `black_level` field in the CVBS metadata) are producer-defined values and are independent of ST.0244.

**Word format:** Each sample is stored as a **signed 16-bit little-endian integer**. The unsigned 10-bit value (0–1023) maps directly into the signed 16-bit range such that 10-bit value 0 equals signed 16-bit value 0 and 10-bit value 1023 equals signed 16-bit value 1023.

- **Negative headroom:** Signed 16-bit values below 0 (down to −32768) are available for chroma excursions below the sync tip or other sub-zero signal content.
- **Positive headroom:** Signed 16-bit values above 1023 (up to 32767) are available for signal excursions above the peak level.

This encoding is known as the **cvbs-encode** encoding.

**Amplitude mapping:** The sync tip, blanking, black, white, and peak 10-bit sample values are defined by the declared Video Standard Preset. The sample level tables in [video-standard-presets](video-standard-presets.md) are the normative reference values for this encoding.

For dual-file YC output (`.cvbsy` and `.cvbsc`) under `CVBS_U10_4FSC`, mapping is representation-specific:

- **Luma (`.cvbsy`):** Uses the same 10-bit level definitions as composite output (sync tip, blanking, black, white, and peak), with the same legal/range constraints.
- **Chroma (`.cvbsc`):** Uses a centred 10-bit representation where chroma zero is at 512 (in the 0-1023 domain). Chroma excursion is represented as oscillation around this centre point.

Any integer-domain translation applied by the producer to `CVBS_U10_4FSC` sample values for luma must be applied identically to chroma before storage, preserving the same encoded-domain relationship between Y and C.

**Protected values:** The bottom of the 10-bit range (values 0–3) and the top (values 1020–1023) are reserved (must never appear in conformant output). These exclusions are defined per Video Standard Preset.

---

## Preset: `CVBS_U16_4FSC`

**Applicable sources:** Standards-compliant output of `ld-decode`, `vhs-decode`, `cvbs-encode`, and similar tools targeting EBU 3280 or SMPTE ST.0244, represented as zero-padded 16-bit words.

> **Note on SMPTE ST.0244 scope:** See the equivalent note in `CVBS_U10_4FSC` above. The same applies here: ST.0244 does not define black level; black level is determined by the Video Standard Preset from the analogue composite standard.

**Word format:** Each sample is stored as an **unsigned 16-bit little-endian integer**. The underlying 10-bit value is the same domain as `CVBS_U10_4FSC` (0-1023), encoded by shifting left 6 bits (multiplying by 64): `u16 = value_10bit * 64`.

The six least-significant bits of every stored sample word are always zero. To recover the original 10-bit value, divide by 64 (`value_10bit = u16 / 64`).

**Amplitude mapping:** The sync tip, blanking, black, white, and peak values are exactly those of `CVBS_U10_4FSC`, scaled by ×64 in storage. Equivalently, interpret stored samples by dividing by 64 to return to the normative 10-bit domain, then apply the Video Standard Preset level tables in [video-standard-presets](video-standard-presets.md).

For dual-file YC output (`.cvbsy` and `.cvbsc`) under `CVBS_U16_4FSC`, mapping is representation-specific in the same way as `CVBS_U10_4FSC`, then scaled by ×64 for storage:

- **Luma (`.cvbsy`):** Uses the same 10-bit level definitions as composite output, then stores each sample as value ×64.
- **Chroma (`.cvbsc`):** Uses a centred 10-bit representation where chroma zero is at 512, then stores each sample as value ×64 (chroma zero at 32768 in the unsigned 16-bit domain).

Any integer-domain translation applied by the producer to `CVBS_U16_4FSC` sample values for luma must be applied identically to chroma before storage, preserving the same encoded-domain relationship between Y and C.

**Protected values:** The same protected 10-bit values as `CVBS_U10_4FSC` apply (0-3 and 1020-1023), represented in storage as 0-192 and 65280-65472 respectively in the scaled domain.

---

## Preset: `RAW_S16_28M`

**Applicable sources:** CX Cards and similar hardware capturers operating at approximately 28.6 MHz.

**Word format:** Each sample is stored as a **signed 16-bit little-endian integer**. Values span the full signed 16-bit range (−32768 to +32767) and represent the raw, unscaled ADC output of the capturing hardware.

**Sample rate:** The nominal sample rate for this preset is **28,636,360 Hz** (= 2 × 14,318,180 Hz, twice the NTSC colour subcarrier frequency). The exact rate may differ slightly between hardware units.

**Amplitude mapping:** No standardised mapping between stored values and analogue voltage levels is defined at the preset level. The relationship between the integer range and the analogue signal levels (sync, blanking, black, white) depends on the hardware gain and DC offset settings at the time of capture. Consumers must derive or externally supply this calibration information.

**Signal level compliance:** Signal level compliance is not meaningful for this preset. Signal levels bear no fixed relationship to the reference values defined by the Video Standard Preset.

---

## Preset: `RAW_S16_40M`

**Applicable sources:** DomesdayDuplicator and similar hardware capturers configured for a 40 MHz capture rate.

**Word format:** Each sample is stored as a **signed 16-bit little-endian integer**. Values span the full signed 16-bit range (−32768 to +32767) and represent the raw, unscaled ADC output of the capturing hardware.

**Sample rate:** The nominal sample rate for this preset is **40,000,000 Hz** (40 MHz). 

**Amplitude mapping:** No standardised mapping between stored values and analogue voltage levels is defined at the preset level. Calibration information must be derived or externally supplied by the consumer.

**Signal level compliance:** Signal level compliance is not meaningful for this preset.

**TBC and burst locking:** Files using this preset will normally have the Signal State Preset `NONSTANDARD_RAW`.

---

## Preset: `CVBS_TPG21_4FSC`

**Applicable sources:** Snell & Wilcox TPG21 test pattern generator and compatible equipment producing 10-bit CVBS output files.
**Word format:** Each sample is stored as a **signed 16-bit little-endian integer**. The 10-bit sample value (0–1023) is encoded using a fixed device offset of 508 and a scale factor of 64:

$$
\text{int16} = (\text{value}_{10\text{-bit}} - 508) \times 64
$$

The six least-significant bits of every stored word are always zero; this is a structural consequence of the ×64 scaling, not bit-field padding. To recover the original 10-bit sample value:

$$
\text{value}_{10\text{-bit}} = \frac{\text{int16}}{64} + 508
$$

- **Signed container:** The 16-bit range is −32768–32767. The offset of 508 centres the signal such that 10-bit values above 508 produce non-negative int16 words, and values below 508 produce negative words.
- **Maximum legal encoded value:** The highest legal 10-bit value (1019) is stored as (1019 − 508) × 64 = 32704.
- **Device constant:** The offset 508 is a hardware constant of the Snell & Wilcox TPG21; it is not derived from any video standard and applies identically to PAL and NTSC files.

**Amplitude mapping:** The sync tip, blanking, black, white, and peak 10-bit sample values are defined by the Video Standard Preset (see Section 4.2 of the main specification). The sample level tables in [video-standard-presets](video-standard-presets.md) are the normative reference values, interpreted in the 10-bit domain before applying the signed encoding described above.

For dual-file YC output (`.cvbsy` and `.cvbsc`) under `CVBS_TPG21_4FSC`, interpretation is performed in the 10-bit domain before applying the fixed offset/scale encoding:

- **Luma (`.cvbsy`):** Uses the same 10-bit level definitions as composite output.
- **Chroma (`.cvbsc`):** Uses a centred 10-bit representation where chroma zero is at 512, with excursion around that centre.

Any translation or offset behavior applied to luma in this preset's 10-bit interpretation domain must be applied identically to chroma before converting to stored int16 words.

**Signal level compliance:** Signal level compliance (Section 3.1) applies to this preset. The protected exclusion ranges at the bottom (values 0–3) and top (values 1020–1023) of the 10-bit domain remain reserved. Values 1020–1023 cannot be represented: they would exceed the int16 maximum of 32767 with this encoding. A compliant encoder must clamp to [4, 1019] before encoding.

---

## Preset: `CVBS_S16_4FSC`

**Applicable sources:** CVBS equipment producing signed 16-bit output with blanking centred at zero and a ×32 scale factor.

**Word format:** Each sample is stored as a **signed 16-bit little-endian integer**. The 10-bit sample value (0–1023) is encoded using the Video Standard Preset blanking level as the offset and a scale factor of 32:

$$
\text{int16} = (\text{value}_{10\text{-bit}} - \text{blanking}_{10\text{-bit}}) \times 32
$$

The five least-significant bits of every stored word are always zero; this is a structural consequence of the ×32 scaling, not bit-field padding. To recover the original 10-bit sample value:

$$
\text{value}_{10\text{-bit}} = \frac{\text{int16}}{32} + \text{blanking}_{10\text{-bit}}
$$

- **Signed container:** The 16-bit range is −32768–32767. The blanking level offset centres the signal such that blanking maps to 0 in the int16 domain.
- **Standard-derived offset:** Unlike `CVBS_TPG21_4FSC`, which uses a fixed device constant (508), the offset for this preset is the blanking level defined by the Video Standard Preset and therefore varies by standard (256 for PAL; 240 for NTSC and PAL_M).
- **Encoded level examples (PAL, blanking = 256):** Sync tip (4) → −8064; Blanking (256) → 0; Black (256) → 0 (no setup: PAL black = blanking); White (844) → 18816; Peak (1019) → 24416.
- **Encoded level examples (NTSC/PAL_M, blanking = 240):** Sync tip (16) → −7168; Blanking (240) → 0; Black (282) → 1344 (+7.5 IRE setup, SMPTE 170M-2004 Table 1); White (800) → 17920; Peak (1019) → 24928.

**Amplitude mapping:** The sync tip, blanking, black, white, and peak 10-bit sample values are defined by the Video Standard Preset (see Section 4.2 of the main specification). The sample level tables in [video-standard-presets](video-standard-presets.md) are the normative reference values, interpreted in the 10-bit domain before applying the signed encoding described above.

For dual-file YC output (`.cvbsy` and `.cvbsc`) under `CVBS_S16_4FSC`, interpretation is performed in the 10-bit domain before applying the offset/scale encoding:

- **Luma (`.cvbsy`):** Uses the same 10-bit level definitions as composite output.
- **Chroma (`.cvbsc`):** Uses a centred 10-bit representation where chroma zero is at 512 in the 10-bit domain, with excursion around that centre. In the stored int16 domain, chroma zero maps to (512 − blanking₁₀ᵦᵢₜ) × 32 (for example, 8192 for PAL, 8704 for NTSC and PAL_M).

Any translation or offset behaviour applied to luma in this preset's 10-bit interpretation domain must be applied identically to chroma before converting to stored int16 words.

**Signal level compliance:** Signal level compliance (Section 3.1) applies to this preset. The protected exclusion ranges at the bottom (values 0–3) and top (values 1020–1023) of the 10-bit domain remain reserved. A compliant encoder must clamp to [4, 1019] before encoding.
