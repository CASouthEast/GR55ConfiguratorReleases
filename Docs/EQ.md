# EQ (Equalizer) — Dedicated, MOD & MFX

Equalizer parameters appear in three places on the GR-55. Extracted from `GR-55_OM.pdf` via macOS PDFKit (dedicated EQ p. 53; MOD EQ p. 45; MFX EQ p. 45).

See [Effects_Index.md](./Effects_Index.md).

---

## Comparison overview

| Aspect | Dedicated EQ | MOD Type 14: EQ | MFX Type 01: EQ |
|---|---|---|---|
| Role in chain | Global patch EQ (effect processor) | Guitar-effects EQ inside AMP→MOD path | Stereo multi-effect algorithm |
| Bands | Low + 2 mids + high, with cuts | Same parametric shape as dedicated | Fixed/select freqs: LOW / MID1 / MID2 / HIGH |
| Low cut | FLAT, 55–800 Hz | FLAT, 55–800 Hz | — (uses LOW FREQ instead) |
| High cut | 700 Hz–11.0 kHz, FLAT | 700 Hz–11.0 kHz, FLAT | — |
| Mid Q | Continuous 0.5–16 | Continuous 0.5–16 | Discrete 0.5 / 1.0 / 2.0 / 4.0 / 8.0 |
| Gain range | ±20 dB (incl. LEVEL) | ±20 dB (incl. LEVEL) | ±15 dB band gains; LEVEL 0–100 |
| Mid freq | 20 Hz–10 kHz (LO-MID / HI-MID) | 20.0 Hz–10.0 kHz | MID1/MID2: 200–8000 Hz |
| Low/high freq | Continuous via cuts + gains | Same | LOW: 200 or 400 Hz; HIGH: 2000 / 4000 / 8000 Hz |
| Extra | **CHARACTER** -3…+3 | — | — |
| Switch | EQ SWITCH | (via MOD SWITCH + type) | (via MFX SWITCH + type) |

**When to use which**
- **Dedicated EQ** — overall patch tone shaping; only place with CHARACTER.
- **MOD EQ** — EQ in the modeling/AMP guitar path (same parametric layout as dedicated, no CHARACTER).
- **MFX EQ** — stereo 4-band with stepped Q and fixed low/high frequency choices; good when MFX is already in the routing.

---

## Dedicated EQ (EFFECT → EQ)

| Parameter | Values | Description |
|---|---|---|
| **EQ SWITCH** | OFF, ON | Turns the EQ OFF/ON. |
| **LOW CUT** | FLAT, 55–800 Hz | Frequency where low-cut begins. FLAT = no low cut. |
| **LOW GAIN #** | -20 dB–+20 dB | Low frequency range tone. |
| **LO-MID FREQ** | 20 Hz–10 kHz | Center frequency for LO-MID GAIN. |
| **LO-MID Q** | 0.5–16 | Width around LO-MID FREQ; higher = narrower. |
| **LO-MID GAIN #** | -20 dB–+20 dB | Low-middle frequency range tone. |
| **HI-MID FREQ** | 20 Hz–10 kHz | Center frequency for HI-MID GAIN. |
| **HI-MID Q** | 0.5–16 | Width around HI-MID FREQ; higher = narrower. |
| **HI-MID GAIN #** | -20 dB–+20 dB | High-middle frequency range tone. |
| **HIGH GAIN #** | -20 dB–+20 dB | High frequency range tone. |
| **HIGH CUT** | 700 Hz–11.0 kHz, FLAT | Frequency where high-cut begins. FLAT = no high cut. |
| **LEVEL** | -20 dB–+20 dB | Overall equalizer volume. |
| **CHARACTER** | -3–0–+3 | Clear definition (+3) vs more mellow (-3). |

---

## MOD Type 14: EQ

Same parametric layout as dedicated EQ (without CHARACTER). Part of the MOD effect type list.

| Parameter | Values | Description |
|---|---|---|
| **LOW CUT** | FLAT, 55–800 Hz | Frequency where low-cut begins. FLAT = no effect. |
| **LOW GAIN #** | -20–+20 dB | Low frequency range tone. |
| **LO-MID FREQ** | 20.0 Hz–10.0 kHz | Center frequency for LO-MID GAIN. |
| **LO-MID Q** | 0.5–16 | Width around LO-MID FREQ; higher = narrower. |
| **LO-MID GAIN #** | -20–+20 dB | Low-middle frequency range tone. |
| **HI-MID FREQ** | 20.0 Hz–10.0 kHz | Center frequency for HI-MID GAIN. |
| **HI-MID Q** | 0.5–16 | Width around HI-MID FREQ; higher = narrower. |
| **HI-MID GAIN #** | -20–+20 dB | High-middle frequency range tone. |
| **HIGH GAIN #** | -20–+20 dB | High frequency range tone. |
| **HIGH CUT** | 700 Hz–11.0 kHz, FLAT | Frequency where high-cut begins. FLAT = no effect. |
| **LEVEL** | -20–+20 dB | Overall equalizer volume. |

Common MOD controls (SWITCH, PAN, NS, EFFECT TYPE) are in [MOD.md](./MOD.md).

---

## MFX Type 01: EQ

Stereo 4-band EQ: low, two mids, high.

| Parameter | Values | Description / constraints |
|---|---|---|
| **LOW FREQ** | 200, 400 Hz | Frequency of the low range. |
| **LOW GAIN #** | -15–+15 dB | Gain of the low range. |
| **MID1 FREQ** | 200–8000 Hz | Frequency of middle range 1. |
| **MID1 GAIN #** | -15–+15 dB | Gain of middle range 1. |
| **MID1 Q** | 0.5, 1.0, 2.0, 4.0, 8.0 | Width of middle range 1; higher Q = narrower. |
| **MID2 FREQ** | 200–8000 Hz | Frequency of middle range 2. |
| **MID2 GAIN #** | -15–+15 dB | Gain of middle range 2. |
| **MID2 Q** | 0.5, 1.0, 2.0, 4.0, 8.0 | Width of middle range 2; higher Q = narrower. |
| **HIGH FREQ** | 2000, 4000, 8000 Hz | Frequency of the high range. |
| **HIGH GAIN #** | -15–+15 dB | Gain of the high range. |
| **LEVEL** | 0–100 | Output volume. |

Common MFX controls (SWITCH, PAN, EFFECT TYPE) are in [MFX.md](./MFX.md).
