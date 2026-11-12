# Audio Data

This document is part of the [CVBS File Format Specification](index.md). It contains the normative audio storage requirements referenced in the [Audio Data](index.md#audio-data) section of that specification.

---

## Overview

Audio handling in this specification follows [SMPTE 272M-1994](https://github.com/decode-orc/analogue-video-specifications/blob/main/docs/video_formats/SMPTE-272M-1994/SMPTE-272M-1994.md), *Formatting AES/EBU Audio and Auxiliary Data into Digital Video Ancillary Data Space*. SMPTE 272M provides a minimum of two audio channels and a maximum of 16 audio channels, transmitted in **channel pairs** (§1.4, §3.11); its preferred implementation is audio sampled at **48 kHz and clock locked (synchronous) to video** (§1.2), optionally carrying **24-bit** audio in which the four AES auxiliary bits extend the resolution of the 20-bit audio sample (§1.3, §3.10 — level C operation).

This specification adopts that preferred implementation as its **only** permitted audio format:

- Up to **16 audio channels** in **8 channel pairs**, each channel pair stored as a separate stereo WAV file.
- All audio is sampled at **48 kHz**, clock locked (synchronous) to video. Asynchronous audio and other sampling rates are not permitted.
- All audio is **24-bit**. No other resolution is permitted.

A **channel pair** is two digital audio channels, generally derived from the same AES audio source (SMPTE 272M §3.11). Audio channels within the same channel pair have the same sampling rate and the same synchronous status (SMPTE 272M §6.5); in this specification all channel pairs are 48 kHz synchronous. When only one channel of a channel pair is active, both channels are still stored; the inactive channel's audio sample values must be set to all zeros (after SMPTE 272M §6.4).

Only the linearly represented audio sample data is stored. The AES sample validity (V), user data (U), and channel status (C) bits, and the SMPTE 272M packet-level structures (audio data packets, extended data packets, and audio control packets), are not stored; a consumer embedding the audio into a serial digital interface must regenerate them in conformance with SMPTE 272M and ANSI S4.40 (AES 3).

## WAV File Format

Each channel pair must be a **RIFF WAV** file or, where the size of the audio requires it, an **RF64** file (see [RF64 for Long Recordings](#rf64-for-long-recordings)), with the following properties:

- **Container:** RIFF/WAVE with a standard RIFF header (`RIFF` chunk, `WAVE` format identifier, `fmt ` sub-chunk, `data` sub-chunk), or RF64/WAVE as defined below
- **Format tag:** PCM (`0x0001`)
- **Channels:** 2 (one channel pair)
- **Sample rate:** 48000 Hz (the `fmt ` chunk `nSamplesPerSec` field shall contain **48000**)
- **Bit depth:** 24-bit signed integer (twos complement), little-endian

No compression, no extended `fmt ` chunks, and no RIFF variants other than RF64 as described below are permitted. Readers must accept both RIFF and RF64 channel pair files.

### RF64 for Long Recordings

A RIFF WAV file stores its chunk sizes in 32-bit fields, so neither the file nor its `data` chunk can exceed 4 GiB. At 48 kHz, 24-bit, 2 channels a channel pair occupies 288,000 bytes per second and reaches this limit after approximately **4 hours 8 minutes** (about 372,800 `PAL` or `SECAM` frames, or about 446,900 `NTSC` or `PAL_M` frames). Video tape captures of up to six hours are an expected use case (see [Recommended Testing Limits](index.md#recommended-testing-limits-informative)).

A channel pair file whose RIFF chunk size (the total file size minus 8 bytes) would exceed `0xFFFFFFFE` must instead be written as **RF64**, as defined by [EBU Tech 3306](https://tech.ebu.ch/publications/tech3306), *MBWF / RF64: An extended File Format for Audio*:

- The file begins with the `RF64` chunk identifier in place of `RIFF`, followed by the `WAVE` format identifier.
- A `ds64` chunk immediately follows the `WAVE` format identifier. It carries the 64-bit RIFF size, the 64-bit `data` chunk size, and the 64-bit sample count. The sample count must be filled in and must equal the number of sample frames (one sample per channel) in the `data` chunk, which is fixed by the frame count as described in [Synchronous Audio](#synchronous-audio). The `ds64` table length must be 0.
- The 32-bit size fields of the `RF64` chunk and the `data` chunk are set to `0xFFFFFFFF`; readers must take the corresponding sizes from the `ds64` chunk.
- The `fmt ` chunk and the sample data in the `data` chunk are identical to those of a RIFF WAV file.

All channel pair files accompanying a CVBS file must use the same container: if any channel pair file requires RF64, all channel pair files must be written as RF64. Otherwise, a channel pair file that fits within the RIFF limit must be written as RIFF, not RF64, so that files of conventional length remain readable by any WAV reader. RF64 files keep the `.wav` extension and the same file naming as RIFF files.

!!! note "Informative: writing when the final length is unknown"
    EBU Tech 3306 describes how a producer that does not know the final length when it starts writing may reserve a `JUNK` chunk of the size of the `ds64` chunk immediately after the `WAVE` identifier. If the file stays within the RIFF limit, the `JUNK` chunk is left in place and the file is a valid RIFF WAV file; if it exceeds the limit, the `RIFF` identifier is replaced by `RF64` and the `JUNK` chunk is rewritten as the `ds64` chunk. Readers must therefore skip any `JUNK` chunk.

**File naming:** `<basename>_audio_<channel_pair>.wav`

Channel pair number is a single digit `0`–`7`. Channel pair numbers need not be contiguous, but each file's suffix must match its `channel_pair` in the metadata (see the [`audio_channel_pair` table](index.md#audio_channel_pair-table)).

## Channel Assignment

SMPTE 272M numbers audio channels 1 through 16, arranged as channel pairs and combined into audio groups 1 through 4 (§4.3, §6.1). Channel pair *p* of this specification carries SMPTE 272M channels 2*p*+1 and 2*p*+2:

| Channel pair file | SMPTE 272M channels | SMPTE 272M audio group |
|-------------------|---------------------|------------------------|
| `_audio_0.wav`    | 1, 2                | 1                      |
| `_audio_1.wav`    | 3, 4                | 1                      |
| `_audio_2.wav`    | 5, 6                | 2                      |
| `_audio_3.wav`    | 7, 8                | 2                      |
| `_audio_4.wav`    | 9, 10               | 3                      |
| `_audio_5.wav`    | 11, 12              | 3                      |
| `_audio_6.wav`    | 13, 14              | 4                      |
| `_audio_7.wav`    | 15, 16              | 4                      |

Within each WAV file the first interleaved channel carries the odd-numbered SMPTE 272M channel (AES subframe 1) and the second interleaved channel carries the even-numbered channel (AES subframe 2), following the channel ordering of SMPTE 272M §6.2. For a conventional stereo source this places left in the first channel and right in the second.

## Synchronous Audio

Audio is **synchronous** with video as defined by SMPTE 272M §3.15: the audio sampling rate is such that the number of audio samples occurring within an integer number of video frames is itself a constant integer number. The number of video frames required for an integer number of audio samples is the **audio frame sequence** (SMPTE 272M §3.8), and each video frame's position within that sequence, starting at 1, is its **audio frame number** (SMPTE 272M §3.7). For 48 kHz audio the values are fixed for each Video Standard Preset:

| Preset  | Frame rate     | Samples per frame | Audio frame sequence |
|---------|----------------|-------------------|----------------------|
| `PAL`   | 25 fps (exact) | **1920** (exact, constant) | 1 frame |
| `SECAM` | 25 fps (exact) | **1920** (exact, constant) | 1 frame |
| `NTSC`  | 30000⁄1001 fps | **8008⁄5** (1602 or 1601)  | 5 frames (**8008** samples) |
| `PAL_M` | 30000⁄1001 fps | **8008⁄5** (1602 or 1601)  | 5 frames (**8008** samples) |

As required by the note to SMPTE 272M §3.15, the video and audio clocks must be derived from the same source; simple frequency synchronisation could eventually result in a missing or extra sample within the audio frame sequence.

**PAL and SECAM:** 48000 ÷ 25 = **1920** samples per frame exactly. The audio frame sequence is 1 frame; every frame carries 1920 samples.

**NTSC and PAL_M:** 48000 ÷ (30000⁄1001) = **8008⁄5** samples per frame. The audio frame sequence is 5 video frames containing exactly **8008** samples. Following SMPTE 272M §14.3 and Table 1, odd-numbered audio frames (1, 3, 5) carry **1602** samples and even-numbered audio frames (2, 4) carry **1601** samples, with no exceptions at 48 kHz.

The first stored video frame (zero-based frame 0) is **audio frame number 1** of the audio frame sequence; the audio frame number of stored frame *n* is (*n* mod 5) + 1. The per-frame sample counts and cumulative sample offsets within one audio frame sequence are therefore:

| Stored frame (*n* mod 5) | Audio frame number | Samples in frame | Offset within sequence |
|--------------------------|--------------------|------------------|------------------------|
| 0                        | 1                  | 1602             | 0                      |
| 1                        | 2                  | 1601             | 1602                   |
| 2                        | 3                  | 1602             | 3203                   |
| 3                        | 4                  | 1601             | 4805                   |
| 4                        | 5                  | 1602             | 6406                   |

To locate the audio samples for stored frame *n* (zero-based), the sample offset from the start of the `data` chunk is:

> offset(*n*) = 8008 × ⌊*n* ⁄ 5⌋ + offset within sequence for (*n* mod 5)

For a file containing *N* video frames the total audio sample count in the WAV `data` chunk of every channel pair file is offset(*N*) — that is, 1920 × *N* for `PAL` and `SECAM`, and 8008 × ⌊*N* ⁄ 5⌋ plus the partial-sequence offset for `NTSC` and `PAL_M`. All channel pair files accompanying a CVBS file must contain the same number of samples.

## Audio–Video Synchronisation

The first audio sample in each channel pair WAV file is synchronous with the first sample of the first stored video frame. No additional time-offset fields are defined; all channel pairs and the video share the same frame-0 origin. Because all audio is synchronous, no drift accumulates over the length of the recording.
