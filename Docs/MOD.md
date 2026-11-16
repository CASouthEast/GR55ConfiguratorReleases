# MOD (Guitar Effects)

MOD is the monaural-input guitar effects processor used in conjunction with AMP. Extracted from `GR-55_OM.pdf` (pp. 42–45) via macOS PDFKit.

See [Effects_Index.md](./Effects_Index.md) for how overlapping types are split across docs.

**Covered here:** OD/DS, WAH, COMP, LIMITER, OCTAVE, PHASER, FLANGER, TREMOLO, ROTARY, UNI-V, PAN  

**Documented elsewhere:**
- MOD Delay → [DELAY.md](./DELAY.md)
- MOD Chorus → [CHORUS.md](./CHORUS.md)
- MOD EQ → [EQ.md](./EQ.md)

---

## Common MOD parameters

| Group | Parameter | Values | Description / constraints |
|---|---|---|---|
| TYPE | **SWITCH** | OFF, ON | Turns MOD OFF/ON. |
| TYPE | **PAN** | L50–R50 | Pan position. **Valid even if SWITCH is OFF.** |
| TYPE | **EFFECT TYPE** | See MOD Type list | Specifies the MOD type. |
| NS | **SWITCH** | OFF, ON | Noise suppressor on/off. |
| NS | **THRESHOLD** | 0–100 | Response to noise level. **0 switches NS off.** Setting higher than necessary may mute low-volume playing. |
| NS | **RELEASE** | 0–100 | Time from when NS begins until noise level reaches 0. |

### MOD Type list

| # | Type | Documented in |
|---:|---|---|
| 01 | OD/DS | this file |
| 02 | WAH | this file |
| 03 | COMP | this file |
| 04 | LIMITER | this file |
| 05 | OCTAVE | this file |
| 06 | PHASER | this file |
| 07 | FLANGER | this file |
| 08 | TREMOLO | this file |
| 09 | ROTARY | this file |
| 10 | UNI-V | this file |
| 11 | PAN | this file |
| 12 | DELAY | [DELAY.md](./DELAY.md) |
| 13 | CHORUS | [CHORUS.md](./CHORUS.md) |
| 14 | EQ | [EQ.md](./EQ.md) |

---

## 01: OD/DS

| Parameter | Values | Description |
|---|---|---|
| **TYPE** | See OD/DS types below | Selects the overdrive/distortion/boost model. |
| **DRIVE #** | 0–120 | Intensity of the distortion. |
| **TONE #** | -50–+50 | Brightness of the sound. |
| **LEVEL** | 0–100 | Volume level for OD/DS. |

### OD/DS types

| Type | Description |
|---|---|
| **MID BOOST** | Booster with unique midrange character; great for solos. |
| **CLEAN BOOST** | Booster; can also be used alone for clean tones with punch. |
| **TREBLE BOOST** | Booster with bright sound characteristics. |
| **BLUES OD** | BOSS BD-2 crunch; faithfully reproduces picking nuances. |
| **CRUNCH** | Brilliant crunch with amp-like distortion elements. |
| **NATURAL OD** | Natural distortion of a slightly overdriven amp. |
| **OD-1** | BOSS OD-1 — sweet, mild distortion. |
| **T-SCREAM** | Models an Ibanez TS-808. |
| **TURBO OD** | High-gain overdrive of the BOSS OD-2. |
| **WARM OD** | Warm overdrive. |
| **DISTORTION** | Basic, traditional distortion. |
| **MILD DS** | Mild distortion. |
| **MID DS** | Distortion emphasizing the midrange. |
| **RAT** | Models a ProCo RAT. |
| **GUV DS** | Models a Marshall Guv’nor. |
| **DST+** | Models an MXR Distortion Plus. |
| **MODERN DS** | Deep distortion of a large stack-type amp. |
| **SOLID DS** | Distortion with a lot of edge. |
| **STACK** | Fat sound with stack-amp distortion elements. |
| **LOUD** | Distortion ideal for heavy riffs. |
| **METAL ZONE** | BOSS MT-2 — wide range of metal sounds, old style to slash metal. |
| **LEAD** | Smoothness of an overdrive with deep distortion. |
| **60S FUZZ** | Models a Fuzz Face — fat fuzz. |
| **OCT FUZZ** | Models an Ace Tone FUZZ. |
| **MUFF FUZZ** | Models an Electro-Harmonix Big Muff π. |

---

## 02: WAH

| Parameter | Values | Description / constraints |
|---|---|---|
| **MODE #** | MANUAL, T.UP, T.DOWN | MANUAL = expression pedal control; T.UP / T.DOWN = wah matched to picking intensity. |
| **TYPE (*1)** | CRY WAH, VO WAH, FAT WAH, LIGHT WAH, 7STRING WAH, RESO WAH | Wah model (see below). |
| **PEDAL POSITION (*1)** | 0–100 | Wah pedal position; changes when you operate the expression pedal. |
| **SENS (*2)** | 0–100 | Sensitivity of response to the input sound. |
| **FREQUENCY (*2)** | 0–100 | Center frequency of the wah effect. |
| **PEAK (*2)** | 0–100 | Intensity of the wah sound. |
| **LEVEL** | 0–100 | Volume level of the effect. |

**Constraints**
- `(*1)` Available when MODE = **MANUAL**.
- `(*2)` Available when MODE = **T.UP** or **T.DOWN**.
- `(*3)` For MANUAL: assign expression pedal; set PEDAL/GK CTL **EXP-FUNCTION** to **MOD CONTROL** (p. 57).

### Wah types

| Type | Description |
|---|---|
| **CRY WAH** | Models CRY BABY wah popular in the ‘70s. |
| **VO WAH** | Models VOX V846. |
| **FAT WAH** | Bold-tone wah. |
| **LIGHT WAH** | Refined sound with no unusual characteristics. |
| **7STRING WAH** | Expanded range for seven-string and baritone guitars. |
| **RESO WAH** | Original effect enhancing analog-synth-filter-style resonances. |

---

## 03: COMP

| Parameter | Values | Description |
|---|---|---|
| **SUSTAIN #** | 0–100 | Sustain for the sound. |
| **ATTACK #** | 0–100 | Attack (onset) of the sound. |
| **LEVEL** | 0–100 | Volume level of the effect. |

---

## 04: LIMITER

| Parameter | Values | Description |
|---|---|---|
| **THRESHOLD #** | 0–100 | When input exceeds this level, limiting is applied. |
| **RELEASE #** | 0–100 | Release time. |
| **LEVEL** | 0–100 | Volume level of the effect. |

---

## 05: OCTAVE

Adds sound one octave lower than the input for a weighty feel. **Play single notes, with other strings muted completely.**

| Parameter | Values | Description |
|---|---|---|
| **OCTAVE LEVEL** | 0–100 | Volume of the octave-lowered sound. |
| **DIRECT LEVEL #** | 0–100 | Volume of the direct sound. |

---

## 06: PHASER

| Parameter | Values | Description |
|---|---|---|
| **TYPE** | 4 STAGE, 8 STAGE, 12 STAGE, BI-PHASE | Number of stages / topology (see below). |
| **RATE #** | 0–100, BPM – | Speed of the effect. |
| **DEPTH #** | 0–100 | Richness of the effect. |
| **RESONANCE #** | 0–100 | Intensity of the effect. |
| **LEVEL** | 0–100 | Volume level of the effect. |

| TYPE | Description |
|---|---|
| **4 STAGE** | Light phaser effect. |
| **8 STAGE** | Most common phaser effect. |
| **12 STAGE** | Deep phase effect. |
| **BI-PHASE** | Two phase-shift circuits in series. |

---

## 07: FLANGER

| Parameter | Values | Description |
|---|---|---|
| **RATE #** | 0–100, BPM – | Speed of the effect. |
| **DEPTH #** | 0–100 | Richness of the effect. |
| **MANUAL #** | 0–100 | Modulation frequency of the flanger. |
| **RESONANCE #** | 0–100 | Intensity of the effect. |
| **LEVEL** | 0–100 | Volume level of the effect. |

---

## 08: TREMOLO

| Parameter | Values | Description |
|---|---|---|
| **RATE #** | 0–100, BPM – | Speed of the effect. |
| **DEPTH #** | 0–100 | Intensity of the effect. |
| **WAVE SHAPE #** | 0–100 | Curve for volume changes; higher = more abrupt. |
| **LEVEL** | 0–100 | Volume level of the effect. |

---

## 09: ROTARY

| Parameter | Values | Description |
|---|---|---|
| **RATE SLOW #** | 0–100, BPM – | Modulation rate when SPEED SELECT = SLOW. |
| **RATE FAST #** | 0–100, BPM – | Modulation rate when SPEED SELECT = FAST. |
| **DEPTH #** | 0–100 | Richness of the effect. |
| **SPEED SELECT #** | SLOW, FAST | Rotation speed of the simulated speaker. |
| **LEVEL** | 0–100 | Volume level of the effect. |

---

## 10: UNI-V

| Parameter | Values | Description |
|---|---|---|
| **RATE #** | 0–100, BPM – | Speed of the effect. |
| **DEPTH #** | 0–100 | Richness of the effect. |
| **LEVEL** | 0–100 | Volume level of the effect. |

---

## 11: PAN

| Parameter | Values | Description |
|---|---|---|
| **RATE #** | 0–100, BPM – | Rate of change in pan position. |
| **DEPTH #** | 0–100 | Intensity of pan-position change. |
| **WAVE SHAPE #** | 0–100 | Curve for pan changes; higher = steeper. |
| **LEVEL** | 0–100 | Volume level of the effect. |
