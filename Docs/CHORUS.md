# CHORUS — Dedicated, MOD & MFX

Chorus-family effects appear as a dedicated processor, as a MOD type, and as two MFX algorithms (Hexa-Chorus and Space-D). Extracted from `GR-55_OM.pdf` via macOS PDFKit (dedicated CHORUS p. 53; MOD Chorus p. 44; MFX pp. 48–49).

See [Effects_Index.md](./Effects_Index.md).

---

## Comparison overview

| Aspect | Dedicated CHORUS | MOD Type 13: CHORUS | MFX 10: HEXA-CHORUS | MFX 11: SPACE-D |
|---|---|---|---|---|
| Role | Shared chorus bus (sends from MFX / MOD / BYPASS) | Chorus in AMP→MOD path | Six-phase chorus | Two-phase stereo chorus (transparent) |
| Types | MONO, STEREO, MONO MILD, STEREO MILD | MONO, STEREO 1, STEREO 2, + mild variants | Fixed algorithm | Fixed algorithm |
| Stereo options | MONO vs STEREO (same L/R vs different L/R) | STEREO 1 (different L/R); **STEREO 2** = dry L / wet R | Pan deviation across six voices | PHASE 0–180° spatial spread |
| Mild variants | MONO MILD, STEREO MILD (suppressed highs) | MONO MILD, STEREO 1 MILD, STEREO 2 MILD | — | — |
| Rate / depth | RATE #, DEPTH # | RATE #, DEPTH # | RATE #, DEPTH # + deviations | RATE #, DEPTH # |
| Pre-delay | — | — | 0.0–100 msec + PRE DELAY DEVIATION | 0.0–100 msec |
| Balance | EFFECT LEVEL 0–100 | EFFECT LEVEL 0–100 | BALANCE D/W + LEVEL | BALANCE D/W + LEVEL + 2-band EQ |
| Extra | Send levels | — | DEPTH / PAN DEVIATION | Transparent, “no impression of modulation” |

**When to use which**
- **Dedicated CHORUS** — patch-level chorus after MFX/MOD via sends.
- **MOD Chorus** — chorus only on the guitar/AMP path; unique **STEREO 2** dry/wet split.
- **Hexa-Chorus** — richest multi-voice spread (six phases, pan/depth deviation).
- **Space-D** — subtle transparent thickening without obvious swirl.

---

## Dedicated CHORUS (EFFECT → CHORUS)

| Parameter | Values | Description |
|---|---|---|
| **SWITCH** | OFF, ON | Turns CHORUS OFF/ON. |
| **TYPE** | See types below | Chooses the chorus type. |
| **RATE #** | 0–100, BPM – | Speed of the effect. |
| **DEPTH #** | 0–100 | Richness of the effect. |
| **EFFECT LEVEL** | 0–100 | Volume level of the effect. |
| **MFX SEND** | 0–100 | Level from MFX output into CHORUS. |
| **MOD SEND** | 0–100 | Level from AMP–MOD output into CHORUS. |
| **BYPASS SEND** | 0–100 | Level from bypass channel into CHORUS. |

### Dedicated chorus types

| Type | Description |
|---|---|
| **01: MONO** | Same chorus sound from left and right channels. |
| **02: STEREO** | Stereo chorus with different chorus sounds on L and R. |
| **03: MONO MILD** | Like MONO with more suppressed high end. |
| **04: STEREO MILD** | Like STEREO with more suppressed high end. |

---

## MOD Type 13: CHORUS

| Parameter | Values | Description |
|---|---|---|
| **TYPE** | See types below | Chooses the chorus type. |
| **RATE #** | 0–100, BPM – | Speed of the effect. |
| **DEPTH #** | 0–100 | Richness of the effect. |
| **EFFECT LEVEL** | 0–100 | Volume level of the effect. |

### MOD chorus types

| Type | Description |
|---|---|
| **MONO** | Same sound from left and right. |
| **STEREO 1** | Different chorus sounds on left and right. |
| **STEREO 2** | Spatial synthesis: direct sound left, effect sound right. |
| **MONO MILD** | More suppressed high end than MONO. |
| **STEREO 1 MILD** | More suppressed high end than STEREO 1. |
| **STEREO 2 MILD** | More suppressed high end than STEREO 2. |

Common MOD controls: [MOD.md](./MOD.md).

---

## MFX Type 10: HEXA-CHORUS

Six-phase chorus (six layers) for richness and spatial spread.

| Parameter | Values | Description / constraints |
|---|---|---|
| **PRE DELAY** | 0.0–100 msec | Delay from direct sound until chorus is heard. |
| **RATE #** | 0–100, BPM – | Modulation rate. |
| **DEPTH #** | 0–100 | Modulation depth. |
| **PRE DELAY DEVIATION** | 0–20 | Deviation of when the various chorus sounds appear. |
| **DEPTH DEVIATION** | -20–+20 | Difference in modulation depth between chorus sounds. |
| **PAN DEVIATION** | 0–20 | Stereo spread of each chorus sound. **0** = all center; **20** = each voice 60° apart relative to center. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs chorus (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

---

## MFX Type 11: SPACE-D

Multiple chorus with two-phase stereo modulation. Gives little impression of modulation; produces a transparent chorus. Includes 2-band EQ.

| Parameter | Values | Description |
|---|---|---|
| **PRE DELAY** | 0.0–100 msec | Delay from direct sound until chorus is heard. |
| **RATE #** | 0–100, BPM – | Modulation rate. |
| **DEPTH #** | 0–100 | Modulation depth. |
| **PHASE** | 0–180 deg | Spatial spread of the sound. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs chorus (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

Common MFX controls: [MFX.md](./MFX.md).
