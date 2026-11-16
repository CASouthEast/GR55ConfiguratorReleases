# DELAY — Dedicated, MOD & MFX

Delay appears as a dedicated effect processor, as a MOD type, and as two MFX algorithms. Extracted from `GR-55_OM.pdf` via macOS PDFKit (dedicated + REVERB/DELAY sends pp. 52–53; MOD Delay p. 44; MFX delays pp. 51–52).

See [Effects_Index.md](./Effects_Index.md).

---

## Comparison overview

| Aspect | Dedicated DELAY | MOD Type 12: DELAY | MFX 17: 3TAP PAN DELAY | MFX 18: TIME CTRL DELAY |
|---|---|---|---|---|
| Role | Shared delay bus (with send levels from MFX / MOD / BYPASS) | Delay inside AMP→MOD guitar path | Stereo triple-tap delay algorithm | Stereo delay with smooth time changes |
| Types | SINGLE, PAN, REVERSE, ANALOG, TAPE, MODULATE, HICUT | Same + **STEREO** | Fixed algorithm (L / C / R taps) | Fixed algorithm |
| Max time | 1–**3400** msec (or BPM) | 1–**3400** msec (or BPM) | 1–**2600** msec per tap (or BPM) | 1–**1300** msec (or BPM) |
| Feedback | 0–100 (repetitions) | 0–100 | CENTER FEEDBACK **-98–+98%** (phase invert) | FEEDBACK **-98–+98%** |
| Effect level | 0–**120** | 0–**120** (REVERSE = dry/wet balance) | Per-tap LEVEL + BALANCE + LEVEL | BALANCE + LEVEL |
| Extra | Send levels from MFX / MOD / BYPASS | — | HF DAMP, 2-band EQ | **ACCELERATION**, HF DAMP, 2-band EQ |
| Stereo features | PAN type splits L/R time | STEREO = dry L / wet R | Independent L/R/C delays | Dual time-ctrl delays with pan |

**When to use which**
- **Dedicated DELAY** — patch-level delay fed by MFX/MOD/bypass sends; longest time (3400 ms).
- **MOD Delay** — delay only on the modeling/AMP path; unique **STEREO** routing option.
- **MFX 3TAP** — three discrete taps with independent times/levels and HF damp.
- **MFX TIME CTRL** — when you want delay time to slew (pitchy “whoosh”) via ACCELERATION.

---

## Dedicated DELAY (EFFECT → DELAY)

| Parameter | Values | Description |
|---|---|---|
| **SWITCH** | OFF, ON | Turns DELAY OFF/ON. |
| **TYPE** | See types below | Chooses the delay type. |
| **DELAY TIME #** | 1–3400 msec, BPM – | Delay time. |
| **FEEDBACK #** | 0–100 | Number of repetitions. |
| **EFFECT LEVEL** | 0–120 | Volume level of the effect. |
| **MFX SEND** | 0–100 | Level from MFX output into DELAY. |
| **MOD SEND** | 0–100 | Level from AMP–MOD output into DELAY. |
| **BYPASS SEND** | 0–100 | Level from bypass channel (unprocessed by the effect) into DELAY. |

### Dedicated / shared delay types

| Type | Description |
|---|---|
| **01: SINGLE** | Simple monaural delay. |
| **02: PAN** | Tap delay dividing delay time between left and right channels. |
| **03: REVERSE** | Effect of playback in reverse. |
| **04: ANALOG** | Mild analog delay sound. |
| **05: TAPE** | Characteristic wavering of a tape echo. |
| **06: MODULATE** | Delay with a pleasant amount of modulation. |
| **07: HICUT** | Warm delay with high frequencies attenuated. |

---

## MOD Type 12: DELAY

| Parameter | Values | Description / constraints |
|---|---|---|
| **TYPE** | SINGLE, PAN, STEREO, REVERSE, ANALOG, TAPE, MODULATE, HICUT | Delay type (see below). |
| **TIME #** | 1–3400 msec, BPM – | Delay time. |
| **FEEDBACK #** | 0–100 | Number of repetitions. |
| **EFFECT LEVEL** | 0–120 | Effect volume. **When TYPE = REVERSE**, this adjusts the balance of direct and effect sound. |

### MOD delay types

| Type | Description |
|---|---|
| **SINGLE** | Simple monaural delay. |
| **PAN** | Tap delay dividing time between left and right. |
| **STEREO** | Direct sound from left; effect sound from right. *(MOD only — not on dedicated DELAY)* |
| **REVERSE** | Playback in reverse. |
| **ANALOG** | Mild analog delay. |
| **TAPE** | Tape-echo wavering. |
| **MODULATE** | Delay with pleasant modulation. |
| **HICUT** | Warm delay with highs attenuated. |

Common MOD controls: [MOD.md](./MOD.md).

---

## MFX Type 17: 3TAP PAN DELAY

Produces three delay sounds: center, left, and right. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **DELAY LEFT/RIGHT/CENTER #** | 1–2600 msec, BPM – | Time until left, right, and center delayed sounds are heard. |
| **CENTER FEEDBACK #** | -98–+98% | Amount of delay fed back. Negative inverts phase. |
| **HF DAMP** | 200–8000 Hz, BYPASS | Frequency above which feedback is filtered. BYPASS = no high filtering. |
| **LEVEL LEFT/RIGHT/CENTER** | 0–100 | Volume of each delay. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

---

## MFX Type 18: TIME CTRL DELAY

Stereo delay whose delay time can be varied smoothly. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **DELAY TIME #** | 1–1300 msec, BPM – | Time until delay sounds are heard. |
| **ACCELERATION #** | 0–15 | How fast delay time moves to a new setting; rate of change also affects pitch change. |
| **FEEDBACK #** | -98–+98% | Delay fed back; negative inverts phase. |
| **HF DAMP** | 200–8000 Hz, BYPASS | High-frequency damping of feedback; BYPASS = none. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

Common MFX controls: [MFX.md](./MFX.md).
