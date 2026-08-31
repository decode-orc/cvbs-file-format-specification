# Signal State Preset Definitions

This document is part of the [CVBS File Format Specification](index.md). It contains the normative Signal State Preset definitions referenced in the [Signal State Presets](index.md#signal-state-presets) section of that specification.

**Naming convention:** Signal State Preset names follow the pattern `<RATE>_<TIMING>_<LOCK>`, where `<RATE>` is `STANDARD` (4×fsc) or `NONSTANDARD`, `<TIMING>` is `STABLE` (time-base stable) or `RAW`, and `<LOCK>` is `LOCKED` or `UNLOCKED`. The `STABLE` token denotes a signal that **requires no time-base correction**, whether or not a time-base corrector was ever applied (see [Time-base stable](#time-base-stable) below). The `RAW` state implies unlocked (without stable line timing, subcarrier phase cannot be computed from sample position), so `<RATE>_RAW` presets do not include a `_LOCKED` / `_UNLOCKED` suffix.

A Signal State Preset captures three independent axes of the signal's **sampling and processing state**. All three are properties of the capture and processing chain: they are decided before or during storage and hold uniformly for the entire file.

| Axis | Standard state | Non-standard state |
|---|---|---|
| Sample rate | Exactly 4×fsc for the declared Video Standard Preset | Non-standard (e.g., oversampled at 28.6 MHz or 40 MHz, or line-locked 13.5 MHz) |
| Time-base stable | Yes — fixed, known sample count per line; 0H-referenced fields | No — line lengths vary, timing is raw |
| Phase locked | Yes — content is sampled at standard subcarrier-reference-locked phase points | No — content is not sampled at standard subcarrier-reference-locked phase points |

These axes are independent, with one implication: phase locked requires time-base stable (and therefore `RAW` implies unlocked). In particular, a file can be time-base stable but not phase locked (e.g., standard NTSC `.tbc` output from `ld-decode` or `vhs-decode`: timing is corrected but the subcarrier phase at each field is not anchored to a canonical 0° reference), and a file at a non-standard sample rate can still be time-base stable (oversampled TBC output).

## Axis Definitions (Normative)

### Sample rate

The sample rate is `STANDARD` when it is exactly 4×fsc for the declared Video Standard Preset, and `NONSTANDARD` otherwise. Non-standard rates include oversampled captures (e.g., 28.6 MHz, 40 MHz) and line-locked component-style rates (e.g., 13.5 MHz).

### Time-base stable

A signal is **time-base stable** when it exhibits the following observable properties:

- Every stored line contains a fixed, known number of samples.
- Every stored field is 0H-referenced as defined in [Stored Frame and Line Origin](video-standard-presets.md#stored-frame-and-line-origin-normative).

Time-base stability is defined by these properties of the stored signal, **not by its provenance**. A signal is time-base stable whether that state was achieved by applying a time-base corrector to an unstable analogue source (LaserDisc, analogue VTR), by resampling during software decode, by digital synthesis (`cvbs-encode`, a test pattern generator), or by digitising an inherently stable source (a broadcast feed, a composite digital VTR such as D-2/D-3). Equivalently: a time-base stable signal is one that **requires no time-base correction**.

In the digital domain, time-base stability is what line-locked sampling provides: a fixed integer number of samples per line, anchored to the line timing reference.

### Phase locked

A signal is **phase locked** when the content is sampled at the standard subcarrier-reference-locked phase points defined by the declared Video Standard Preset's Sc/H convention (see [video-standard-presets](video-standard-presets.md)). Equivalently, the subcarrier phase of any sample is deterministically computable from that sample's (field, line, sample) coordinates together with the field's position within the declared colour field sequence.

Notes:

- Phase locked is a **positional** assertion about samples within a field. It does **not** assert that the colour field sequence progresses without breaks from field to field across the file — that is a property of the content, declared separately (see [Continuity is not a preset axis](#continuity-is-not-a-preset-axis)).
- Phase locked requires time-base stability; without a fixed sample-to-line relationship, phase cannot be computed from sample position.
- Line-locked sampling that is not subcarrier-locked (for example 13.5 MHz component-style sampling of a composite signal) is time-base stable but **not** phase locked.
- A signal with no usable subcarrier reference (for example monochrome material) cannot be phase locked and must be declared `UNLOCKED`.
- For PAL, 4×fsc is not an integer multiple of the line frequency in the analogue domain, so a line-locked sampling *clock* cannot simultaneously be subcarrier-locked. Resampled TBC output, however, can satisfy both properties in the stored digital domain; the phase-locked assertion applies to the stored samples under the declared preset's Sc/H convention.

## Continuity is not a Preset Axis

Whether the stored sequence of fields is **continuous** — no missing, repeated, or reordered content, with the colour field sequence progressing according to the declared standard — is a property of the **content**, not of the sampling and processing chain. A single event partway through a capture (a LaserDisc skip, a tape dropout, a paused capture) breaks continuity while every field before and after it remains rate-standard, time-base stable, and phase locked.

Continuity is therefore **not** encoded in the Signal State Preset. It is declared by the [`sequence_continuous`](index.md#sequence_continuous) field of the `cvbs_file` metadata table. A file may be fully phase locked yet discontinuous: each contiguous run of frames honours the declared presets in full, and a consumer must re-establish colour sequence position after each discontinuity. Locating individual discontinuities is the domain of producer extension metadata (see [Producer Extension Metadata](index.md#producer-extension-metadata)).

Producers must not downgrade the declared preset (for example from `LOCKED` to `UNLOCKED`) merely because the content contains discontinuities; the preset describes the processing state, and `sequence_continuous` describes the content.

## What the Preset Governs

The preset governs several aspects of format interpretation:

- **Normative sample-count constraints** as defined by the declared Video Standard Preset in [video-standard-presets](video-standard-presets.md) apply only when the signal is time-base stable and the sample rate is the standard 4×fsc. Depending on the standard, these constraints may be defined per field, per frame, or both. Without time-base stability there is no guarantee of a fixed sample count per line, so consumers must not infer fixed byte sizes from presets alone.
- **Signal level compliance** is only meaningful when the signal is time-base stable and the Sample Encoding Preset is `CVBS_U10_4FSC` or `CVBS_U16_4FSC`. A raw RF capture contains signal levels that bear no relation to the preset's reference sample values.
- **Sample-coordinate anomaly annotations** (for example dropout coordinates) are only stable when lines have a fixed, known length, i.e., when the signal is time-base stable. Such annotations are extension metadata, not part of the core schema.
- **Subcarrier phase analysis:** whether the phase of individual samples can be computed from position is governed by the phase-locked axis of the preset. Whether phase and colour sequence position progress without breaks between fields is governed by the [`sequence_continuous`](index.md#sequence_continuous) metadata field, not by the preset.

---

## Preset: `STANDARD_STABLE_LOCKED`

| Property | Value |
|---|---|
| Sample rate | Standard 4×fsc for the declared Video Standard Preset |
| Time-base stable | Yes |
| Phase locked | Yes |

**Typical source:** `cvbs-encode` synthetic output (time-base stable and phase locked by construction — no time-base corrector involved); broadcast-grade 4×fsc capture; digitised composite digital (D-2/D-3) VTR material; `ld-decode` / `vhs-decode` PAL `.tbc` output when Sc/H locked.

**Normative sample-count constraints:** Apply as defined by the declared Video Standard Preset in [video-standard-presets](video-standard-presets.md).

**Signal level compliance:** Required when Sample Encoding Preset is `CVBS_U10_4FSC` or `CVBS_U16_4FSC`.

**Extension anomaly annotations:** Valid and meaningful if an extension format provides them.

---

## Preset: `STANDARD_STABLE_UNLOCKED`

| Property | Value |
|---|---|
| Sample rate | Standard 4×fsc for the declared Video Standard Preset |
| Time-base stable | Yes |
| Phase locked | No |

**Typical source:** `ld-decode` / `vhs-decode` NTSC `.tbc` output — timing corrected but subcarrier phase not anchored to a canonical reference; monochrome material with stable timing.

**Normative sample-count constraints:** Apply as defined by the declared Video Standard Preset in [video-standard-presets](video-standard-presets.md).

**Signal level compliance:** Required when Sample Encoding Preset is `CVBS_U10_4FSC` or `CVBS_U16_4FSC`.

**Extension anomaly annotations:** Valid and meaningful if an extension format provides them.

---

## Preset: `STANDARD_RAW`

| Property | Value |
|---|---|
| Sample rate | Standard 4×fsc for the declared Video Standard Preset |
| Time-base stable | No |
| Phase locked | No |

**Typical source:** A signal sampled at the standard rate but not time-base stable — e.g., a synchronous ADC capture at the standard 4×fsc rate before any time-base correction.

**Normative sample-count constraints:** Do not apply.

**Signal level compliance:** Not required.

**Extension anomaly annotations:** Per-line/sample coordinates are generally not stable and should not be emitted.

---

## Preset: `NONSTANDARD_STABLE_LOCKED`

| Property | Value |
|---|---|
| Sample rate | Non-standard (e.g., 28.6 MHz, 40 MHz) |
| Time-base stable | Yes |
| Phase locked | Yes |

**Typical source:** Oversampled TBC output (e.g., phase-locked 8×fsc); a burst-locked TBC applied after raw oversampled capture.

**Normative sample-count constraints:** Do not apply at the standard 4×fsc sample counts defined in [video-standard-presets](video-standard-presets.md).

**Signal level compliance:** Not applicable for standard-mapped CVBS amplitude values at non-standard sample rates; consumers must use calibration data to map sample values to analogue levels.

**Extension anomaly annotations:** Valid (sample coordinates are stable because the signal is time-base stable), but coordinates reference samples at the non-standard sample rate.

---

## Preset: `NONSTANDARD_STABLE_UNLOCKED`

| Property | Value |
|---|---|
| Sample rate | Non-standard (e.g., 13.5 MHz, 28.6 MHz, 40 MHz) |
| Time-base stable | Yes |
| Phase locked | No |

**Typical source:** Oversampled TBC output where subcarrier phase is not stabilised; line-locked 13.5 MHz sampling of a composite signal (time-base stable but not subcarrier-locked).

**Normative sample-count constraints:** Do not apply.

**Signal level compliance:** Not applicable.

**Extension anomaly annotations:** Valid for stable sample coordinate references.

---

## Preset: `NONSTANDARD_RAW`

| Property | Value |
|---|---|
| Sample rate | Non-standard (e.g., 28.6 MHz, 40 MHz) |
| Time-base stable | No |
| Phase locked | No |

**Typical source:** DomesdayDuplicator raw RF capture; raw ADC output at any non-standard rate.

**Normative sample-count constraints:** Do not apply.

**Signal level compliance:** Not applicable.

**Extension anomaly annotations:** Per-line/sample coordinates are generally not stable and should not be emitted.

---

## Migration From v1.5.0 to v1.6.0

This section defines the mapping between the v1.5.0 Signal State Preset definitions and the v1.6.0 definitions above, and the accompanying metadata schema change.

### Summary of changes

1. **"TBC applied" axis redefined as "time-base stable", and the `TBC` preset-name token renamed to `STABLE`.** The axis is now defined by observable properties of the stored signal (fixed samples per line, 0H-referenced fields) rather than by whether a time-base corrector was applied. Synthetic, test-pattern, and digitally captured sources that never required correction are now explicitly covered. The new token reads "requires no time-base correction".
2. **"Burst locked" axis redefined as "phase locked".** The axis now asserts only that subcarrier phase is computable from sample position within a field. It no longer implies — and must no longer be used to signal — phase or sequence continuity between fields.
3. **Continuity moved out of the preset.** Whether the stored sequence is free of breaks is now declared by the new [`sequence_continuous`](index.md#sequence_continuous) field of the `cvbs_file` metadata table. This resolves the previously unrepresentable case of a fully phase-locked file containing a source discontinuity (for example a LaserDisc skip), which producers were forced to declare as `UNLOCKED` under v1.5.0 semantics.
4. **Metadata schema version incremented.** The SQLite `PRAGMA user_version` is incremented from **10** (v1.5.0) to **11** (v1.6.0) to reflect the addition of the `sequence_continuous` column.
5. **Frame integrity rules re-scoped.** In [video-standard-presets](video-standard-presets.md#frame-ordering-and-phase-verification), frame boundary and exact frame sample-count guarantees are now tied to the time-base stable axis, while sequence progression guarantees are tied to `sequence_continuous = TRUE` rather than to the `LOCKED` presets.

### Preset name mapping

The `TBC` token is renamed to `STABLE`; the `RATE` and `LOCK` tokens and the two `RAW` preset names are unchanged:

| v1.5.0 preset name | v1.6.0 preset name |
|---|---|
| `STANDARD_TBC_LOCKED` | `STANDARD_STABLE_LOCKED` |
| `STANDARD_TBC_UNLOCKED` | `STANDARD_STABLE_UNLOCKED` |
| `STANDARD_RAW` | `STANDARD_RAW` |
| `NONSTANDARD_TBC_LOCKED` | `NONSTANDARD_STABLE_LOCKED` |
| `NONSTANDARD_TBC_UNLOCKED` | `NONSTANDARD_STABLE_UNLOCKED` |
| `NONSTANDARD_RAW` | `NONSTANDARD_RAW` |

A v1.5.0 preset name must not appear in a v1.6.0 (`user_version = 11`) database, and vice versa; the `signal_state_preset` CHECK constraint in the [metadata schema](index.md#sqlite-metadata-schema) admits only the v1.6.0 names.

### Metadata mapping

When migrating a v1.5.0 metadata database (`user_version = 10`) to v1.6.0 (`user_version = 11`), or when a consumer interprets a v1.5.0 database under v1.6.0 semantics:

| v1.5.0 `signal_state_preset` | v1.6.0 `signal_state_preset` | v1.6.0 `sequence_continuous` |
|---|---|---|
| `STANDARD_TBC_LOCKED` | `STANDARD_STABLE_LOCKED` | `TRUE` |
| `STANDARD_TBC_UNLOCKED` | `STANDARD_STABLE_UNLOCKED` — or `STANDARD_STABLE_LOCKED` after re-analysis (see below) | `NULL` (unknown) unless re-analysed |
| `STANDARD_RAW` | `STANDARD_RAW` | `NULL` unless known from the capture process |
| `NONSTANDARD_TBC_LOCKED` | `NONSTANDARD_STABLE_LOCKED` | `TRUE` |
| `NONSTANDARD_TBC_UNLOCKED` | `NONSTANDARD_STABLE_UNLOCKED` — or `NONSTANDARD_STABLE_LOCKED` after re-analysis (see below) | `NULL` (unknown) unless re-analysed |
| `NONSTANDARD_RAW` | `NONSTANDARD_RAW` | `NULL` unless known from the capture process |

**Rationale:** Under v1.5.0 semantics (as implemented by `ld-decode`), a `LOCKED` preset asserted that subcarrier phase was continuous across the whole file — which is the v1.6.0 combination of phase locked **and** `sequence_continuous = TRUE`. A v1.5.0 `UNLOCKED` preset conflated two distinct cases: a signal that was genuinely not phase locked, and a signal that was phase locked but contained one or more discontinuities. These two cases cannot be distinguished from the v1.5.0 metadata alone, so a mechanical migration must map `UNLOCKED` to `UNLOCKED` with `sequence_continuous = NULL`. A producer that re-analyses the signal and finds each contiguous run to be phase locked may instead reclassify the file as `LOCKED` with `sequence_continuous = FALSE`.

### Consumer guidance

- A database with `user_version = 10` must be interpreted under v1.5.0 semantics; consumers may apply the mapping table above to derive v1.6.0 semantics.
- A database with `user_version = 11` must contain the `sequence_continuous` column and must be interpreted under the definitions in this document.
- Producers writing v1.6.0 metadata must write `user_version = 11` and should populate `sequence_continuous` whenever the continuity of the content is known.
