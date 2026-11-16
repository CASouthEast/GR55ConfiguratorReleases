# MFX (Multi-Effects)

MFX is the stereo-in multi-effects processor. Extracted from `GR-55_OM.pdf` (pp. 45–52) via macOS PDFKit.

See [Effects_Index.md](./Effects_Index.md) for how overlapping types are split across docs.

**Covered here:** SUPER FILTER, PHASER, STEP PHASER, RING MODULATOR, TREMOLO, AUTO PAN, SLICER, VK ROTARY, FLANGER, STEP FLANGER, COMPRESSOR, LIMITER, LOFI COMPRESS, PITCH SHIFTER  

**Documented elsewhere:**
- MFX EQ → [EQ.md](./EQ.md)
- MFX Hexa-Chorus / Space-D → [CHORUS.md](./CHORUS.md)
- MFX Guitar Amp Simulator → [AMP_and_MFX_GuitarAmpSim.md](./AMP_and_MFX_GuitarAmpSim.md)
- MFX 3TAP PAN DELAY / TIME CTRL DELAY → [DELAY.md](./DELAY.md)

---

## Common MFX parameters

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | Turns MFX OFF/ON. |
| **PAN** | L50–R50 | Pan position. **Valid even if SWITCH is OFF.** |
| **EFFECT TYPE** | See MFX Type list | Specifies the MFX type. |

### MFX Type list

| # | Type | Documented in |
|---:|---|---|
| 01 | EQ | [EQ.md](./EQ.md) |
| 02 | SUPER FILTER | this file |
| 03 | PHASER | this file |
| 04 | STEP PHASER | this file |
| 05 | RING MODULATOR | this file |
| 06 | TREMOLO | this file |
| 07 | AUTO PAN | this file |
| 08 | SLICER | this file |
| 09 | VK ROTARY | this file |
| 10 | HEXA-CHORUS | [CHORUS.md](./CHORUS.md) |
| 11 | SPACE-D | [CHORUS.md](./CHORUS.md) |
| 12 | FLANGER | this file |
| 13 | STEP FLANGER | this file |
| 14 | GUITAR AMP SIMULATOR | [AMP_and_MFX_GuitarAmpSim.md](./AMP_and_MFX_GuitarAmpSim.md) |
| 15 | COMPRESSOR | this file |
| 16 | LIMITER | this file |
| 17 | 3TAP PAN DELAY | [DELAY.md](./DELAY.md) |
| 18 | TIME CTRL DELAY | [DELAY.md](./DELAY.md) |
| 19 | LOFI COMPRESS | this file |
| 20 | PITCH SHIFTER | this file |

---

## 02: SUPER FILTER

Filter with an extremely sharp slope; cutoff can be varied cyclically.

| Parameter | Values | Description / constraints |
|---|---|---|
| **FILTER TYPE #** | LPF, BPF, HPF, NOTCH | Frequencies allowed through the filter. |
| **FILTER SLOPE** | -12 dB, -24 dB, -36 dB | Gentle / steep / extremely steep (attenuation per octave). |
| **FILTER CUTOFF #** | 0–100 | Cutoff frequency; higher = higher cutoff. |
| **FILTER RESONANCE #** | 0–100 | Emphasizes region near cutoff. |
| **FILTER GAIN #** | 0–+12 dB | Boost for filter output. |
| **MODULATION SW** | OFF, ON | On/off for cyclic change. |
| **MODULATION WAVE** | TRI, SQR, SIN, SAW1, SAW2 | How cutoff is modulated. |
| **RATE** | 0–100, BPM – | Modulation rate. |
| **DEPTH** | 0–100 | Modulation depth. |
| **ATTACK** | 0–100 | Speed of cutoff change. Effective if MODULATION WAVE is SQR, SAW1, or SAW2. |
| **LEVEL** | 0–100 | Output volume. |

---

## 03: PHASER

Stereo phaser: phase-shifted sound added to the original and modulated. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **MODE** | 4-STAGE, 8-STAGE, 12-STAGE | Number of stages. |
| **MANUAL #** | 0–100 | Basic frequency from which sound is modulated. |
| **RATE #** | 0–100, BPM – | Modulation rate. |
| **DEPTH #** | 0–100 | Modulation depth. |
| **POLARITY** | INVERSE, SYNCHRO | L/R phase opposite (spreads mono) or same (for stereo source). |
| **RESONANCE #** | 0–100 | Feedback amount. |
| **CROSS FEEDBACK** | -98–+98% | Phaser sound fed back; negative inverts phase. |
| **MIX** | 0–100 | Level of phase-shifted sound. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 04: STEP PHASER

Stereo phaser with gradual stepwise variation. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **MODE** | 4-STAGE, 8-STAGE, 12-STAGE | Number of stages. |
| **MANUAL #** | 0–100 | Basic frequency from which sound is modulated. |
| **RATE #** | 0–100, BPM – | Modulation rate. |
| **DEPTH** | 0–100 | Modulation depth. |
| **POLARITY** | INVERSE, SYNCHRO | L/R phase opposite or same. |
| **RESONANCE #** | 0–100 | Feedback amount. |
| **CROSS FEEDBACK** | -98–+98% | Phaser sound fed back; negative inverts phase. |
| **STEP RATE #** | 0–100, BPM – | Rate of stepwise change. |
| **MIX** | 0–100 | Level of phase-shifted sound. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 05: RING MODULATOR

Amplitude modulation producing bell-like sounds; frequency can follow input volume. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **FREQUENCY #** | 0–127 | Frequency at which modulation is applied. |
| **SENS #** | 0–100 | Amount of frequency modulation applied. |
| **POLARITY** | UP, DOWN | Frequency modulation toward higher or lower frequencies. |
| **LOW GAIN** | -15–+15 dB | Low-frequency gain. |
| **HIGH GAIN** | -15–+15 dB | High-frequency gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

---

## 06: TREMOLO

Cyclically alters the volume. Includes 2-band EQ.

| Parameter | Values | Description |
|---|---|---|
| **MOD WAVE #** | TRI, SQR, SIN, SAW1/2 | How volume is modulated. |
| **RATE #** | 0–100, BPM – | Frequency of the change. |
| **DEPTH #** | 0–100 | Depth of the effect. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 07: AUTO PAN

Cyclically varies the stereo location. Includes 2-band EQ.

| Parameter | Values | Description |
|---|---|---|
| **MOD WAVE** | TRI, SQR, SIN, SAW1/2 | How pan position is varied. |
| **RATE #** | 0–100, BPM – | Frequency of the change. |
| **DEPTH #** | 0–100 | Depth of the effect. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 08: SLICER

Applies successive cuts so the sound appears to be a backing phrase. Especially effective on sustain-type sounds.

| Parameter | Values | Description / constraints |
|---|---|---|
| **PATTERN #** | P01–P20 | Slice pattern used to cut the sound. |
| **RATE #** | 0–100, BPM – | Rate at which the 16-step sequence repeats. |
| **ATTACK #** | 0–100 | Speed of level change between steps. |
| **INPUT SYNC SW** | OFF, ON | ON = input note resumes sequence from step 1. |
| **INPUT SYNC THRESHOLD** | 0–100 | Volume at which an input note is detected. |
| **LEVEL** | 0–100 | Output volume. |

---

## 09: VK ROTARY

Modified rotary-speaker response with further-boosted low end. Same specifications as the Roland VK-7 organ’s built-in rotary speaker. Includes 2-band EQ.

| Parameter | Values | Description |
|---|---|---|
| **SPEED** | SLOW, FAST | Rotational speed of the rotating speaker. |
| **BRAKE** | OFF, ON | ON = rotation gradually stops; OFF = gradually resumes. |
| **WOOFER SLOW SPEED #** | 0–100 | Low-speed woofer rotation. |
| **WOOFER FAST SPEED #** | 0–100 | High-speed woofer rotation. |
| **WOOFER TRANS UP** | 0–100 | Rate woofer speeds up SLOW→FAST. |
| **WOOFER TRANS DOWN** | 0–100 | Rate woofer slows FAST→SLOW. |
| **WOOFER LEVEL** | 0–100 | Woofer volume. |
| **TWEETER SLOW SPEED #** | 0–100 | Same as woofer, for tweeter. |
| **TWEETER FAST SPEED #** | 0–100 | Same as woofer, for tweeter. |
| **TWEETER TRANS UP** | 0–100 | Same as woofer, for tweeter. |
| **TWEETER TRANS DOWN** | 0–100 | Same as woofer, for tweeter. |
| **TWEETER LEVEL** | 0–100 | Tweeter volume. |
| **SPREAD** | 0–10 | How the rotary-speaker sound is spread. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 12: FLANGER

Stereo flanger with jet-plane-like metallic resonance and a filter for flanged timbre. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **FILTER TYPE** | OFF, LPF, HPF | No filter / cut above cutoff / cut below cutoff. |
| **CUTOFF FREQ** | 200–8000 Hz | Center frequency when filtering. |
| **PRE DELAY** | 0.0–100 msec | Delay from direct sound until flanger is heard. |
| **RATE #** | 0–100, BPM – | Modulation rate. |
| **DEPTH #** | 0–100 | Modulation depth. |
| **PHASE** | 0–180 deg | Spatial spread. |
| **FEEDBACK #** | -98–+98% | Flanger sound fed back; negative inverts phase. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

---

## 13: STEP FLANGER

Flanger whose pitch changes in steps; step rate can sync to tempo. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **FILTER TYPE** | OFF, LPF, HPF | No filter / cut above cutoff / cut below cutoff. |
| **CUTOFF FREQ** | 200–8000 Hz | Center frequency when filtering. |
| **PRE DELAY** | 0.0–100 msec | Delay until flanger is heard. |
| **RATE #** | 0–100, BPM – | Modulation rate. |
| **DEPTH** | 0–100 | Modulation depth. |
| **PHASE** | 0–180 deg | Spatial spread. |
| **FEEDBACK #** | -98–+98% | Feedback; negative inverts phase. |
| **STEP RATE #** | 0–100, BPM – | Rate (period) of pitch change. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

---

## 15: COMPRESSOR

Flattens high levels and boosts low levels, smoothing volume fluctuations. Includes 2-band EQ.

| Parameter | Values | Description |
|---|---|---|
| **ATTACK #** | 0–100 | Time from exceeding THRESHOLD until compression starts. |
| **THRESHOLD #** | 0–100 | Volume at which compression begins. |
| **POST GAIN #** | 0–+18 dB | Output gain. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 16: LIMITER

Compresses signals that exceed a specified level, preventing distortion. Includes 2-band EQ.

| Parameter | Values | Description |
|---|---|---|
| **RELEASE #** | 0–100 | Time after signal falls below THRESHOLD until compression stops. |
| **THRESHOLD #** | 0–100 | Volume at which compression begins. |
| **RATIO #** | 1.5:1, 2:1, 4:1, 100:1 | Compression ratio. |
| **POST GAIN #** | 0–+18 dB | Output gain. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **LEVEL** | 0–100 | Output volume. |

---

## 19: LOFI COMPRESS

Intentionally degrades sound quality for creative purposes. Path: Compressor → Lo-Fi → 2-band EQ (per side).

| Parameter | Values | Description / constraints |
|---|---|---|
| **PRE FILTER TYPE #** | 1–6 | Type of filter before Lo-Fi. **1** = compressor off; **2–6** = compressor on. |
| **LOFI TYPE #** | 1–9 | Sound-quality degradation; higher = poorer quality. |
| **POST FILTER TYPE** | OFF, LPF, HPF | Filter after Lo-Fi. |
| **POST FILTER CUTOFF** | 200–8000 Hz | Basic frequency of the POST FILTER. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |

---

## 20: PITCH SHIFTER

Stereo pitch shifter. Includes 2-band EQ.

| Parameter | Values | Description / constraints |
|---|---|---|
| **COARSE #** | -24–+12 semi | Pitch in semitone steps. |
| **FINE #** | -100–+100 cent | Pitch in 2-cent steps. |
| **DELAY TIME** | 1–1300 msec, BPM – | Delay from direct sound until pitch-shifted sound. |
| **FEEDBACK** | -98–+98% | Pitch-shifted sound fed back; negative inverts phase. |
| **LOW GAIN** | -15–+15 dB | Low-range gain. |
| **HIGH GAIN** | -15–+15 dB | High-range gain. |
| **BALANCE #** | D100:0W–D0:100W | Direct (D) vs effect (W) balance. |
| **LEVEL** | 0–100 | Output volume. |
