# Controller Assignments (ASSIGN)

Per-patch **ASSIGN 1–8** and related controller workflow from the Roland GR-55 Owner’s Manual (`GR-55_OM.pdf`, MASTER ASSIGN pp. 57–58; Controller Assignments pp. 61–63), extracted via **macOS PDFKit**.

Pedal **FUNCTION** lists (HOLD, TONE SW, PATCH VOLUME, etc.) live in [MASTER.md](./MASTER.md) / [SYSTEM.md](./SYSTEM.md). This document covers the eight free ASSIGN slots and how they interact with PEDAL/GK CTL.

---

## Two layers of control

| Layer | Where | What it does |
|---|---|---|
| **PEDAL/GK CTL** | SYSTEM and/or MASTER | Chooses a **named function** for CTL, EXP, EXP ON, EXP SW, GK S1/S2, GK VOL (e.g. DELAY SW, PATCH VOLUME, MOD CONTROL). |
| **ASSIGN 1–8** | MASTER only | Maps any **TARGET** parameter to a **SOURCE** controller, with min/max, mode, and active range. |

### Global vs per-patch pedals

1. **Same operation for all patches** — In SYSTEM → PEDAL/GK CTL, set the controller to something **other than PATCH SETTING**.
2. **Different per patch** — Set SYSTEM FUNCTION to **PATCH SETTING**, then edit MASTER → PEDAL/GK CTL for that patch.

### Using ASSIGN 1–8

1. Set SYSTEM → PEDAL/GK CTL for the controllers you care about to **PATCH SETTING** (so the patch owns them).
2. Select the patch → EDIT → MASTER → **ASSIGN 1–8**.
3. Save the patch if you want to keep the settings.

---

## ASSIGN 1–8 parameters

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | Turns that ASSIGN on/off. |
| **TARGET** | See [Assign Target catalog](#assign-target-catalog) | Parameter controlled. |
| **TARGET MIN** | (depends on TARGET) | Minimum of the controlled range. |
| **TARGET MAX** | (depends on TARGET) | Maximum of the controlled range. |
| **SOURCE** | See SOURCE table | Controller that drives the target. |
| **SOURCE MODE** | MOMENT, TOGGLE | How the value changes (see below). |
| **ACT RANGE LO** | 0–126 | Low end of source range that affects the target. Normally leave at **0**. |
| **ACT RANGE HI** | 1–127 | High end. Normally leave at **127**. |

Within the source’s operating range, only the span from ACT RANGE LO to HI controls the target.

### SOURCE

| Value | Description |
|---|---|
| **CTL** | [CTL] pedal |
| **EXP** | Expression pedal |
| **EXP ON** | Expression pedal while EXP switch is on |
| **EXP SW** | Expression pedal switch |
| **INT PDL** | Internal (virtual) pedal — see below |
| **WAVE PDL** | Wave (virtual) pedal — see below |
| **GK S1 / GK S2** | GK pickup [S1]/[S2] |
| **GK VOL** | GK volume knob |
| **CC1–31, CC64–95** | Control change from external MIDI |

### SOURCE MODE

| Value | Behavior |
|---|---|
| **MOMENT** | Normally off (minimum); on (maximum) only while the control is operated. |
| **TOGGLE** | Toggles off/on (min/max) each time the control is operated. |

**Constraint:** For **INT PDL** or **WAVE PDL**, set SOURCE MODE to **MOMENT**.

---

## Virtual expression pedals

Assign a parameter to a virtual pedal so volume/tone can change in real time without a physical EXP move.

### Internal pedal (SOURCE = INT PDL)

Starts on a **trigger**, then moves from heel → toe over **INT TIME** along **INT CURVE**, driving TARGET.

| Parameter | Values | Description |
|---|---|---|
| **INT TRIG** | See triggers below | What starts the internal pedal motion. |
| **INT TIME** | 0–100 | Time from released (heel) to depressed (toe). |
| **INT CURVE** | LINEAR, SLOW RISE, FAST RISE | Shape of the change. |

#### INT TRIG

| Value | Triggered when… |
|---|---|
| **PATCH CHANGE** | You switch patches. |
| **CTL PDL** | You operate [CTL]. |
| **EXP LOW / MID / HIGH** | EXP at minimum / through center / at maximum. |
| **EXP ON LOW / MID / HIGH** | Same, while EXP switch is on. |
| **EXP SW** | You operate the expression pedal switch. |
| **GK S1 / GK S2** | You operate GK [S1] or [S2]. |

`*1` INT TRIG, INT TIME, INT CURVE apply only when SOURCE = **INT PDL**.

### Wave pedal (SOURCE = WAVE PDL)

Cyclically modulates TARGET with a fixed waveform.

| Parameter | Values | Description / constraints |
|---|---|---|
| **WAVE RATE** | 0–100, BPM | Period of one wave cycle. **BPM** follows PATCH TEMPO; if the tempo-based time is too long, it syncs to 1/2 or 1/4. |
| **WAVE FORM** | SAW, TRI, SIN | Wave shape (saw / triangle / sine). |

`*2` WAVE RATE and WAVE FORM apply only when SOURCE = **WAVE PDL**.

---

## Assign Target catalog

Names below match the **Assign Target** picker (as shown in the editor UI). Grouped by tone / effect / patch.

### Values and constraints

This list is **targets only** (what can be assigned). For each target’s meaning, value range, enums, and type-dependent constraints, use the other docs:

| Targets in… | Look up in |
|---|---|
| PCM1 / PCM2 | [Tone_Types.md](./Tone_Types.md) (PCM parameter list in OM; categories/counts) |
| Modeling | [Tone_Types.md](./Tone_Types.md) |
| AMP / MFX Amp Sim | [AMP_and_MFX_GuitarAmpSim.md](./AMP_and_MFX_GuitarAmpSim.md) |
| MOD (except Delay/Chorus/EQ) | [MOD.md](./MOD.md) |
| MFX (except overlapping types) | [MFX.md](./MFX.md) |
| EQ (dedicated / MOD / MFX) | [EQ.md](./EQ.md) |
| Delay (dedicated / MOD / MFX) | [DELAY.md](./DELAY.md) |
| Chorus (dedicated / MOD / MFX) | [CHORUS.md](./CHORUS.md) |
| Reverb | [REVERB.md](./REVERB.md) |
| Patch-level (tempo, GUITAR OUT, alt-tuning, …) | [MASTER.md](./MASTER.md) |
| Index of effect docs | [Effects_Index.md](./Effects_Index.md) |

`TARGET MIN` / `TARGET MAX` use the same units and ranges as the underlying parameter.

---

### PCM1

#### Tone

- PCM1 Tone Switch
- PCM1 Tone Number
- PCM1 Tone Level
- PCM1 Tone Octave
- PCM1 Tone Chromatic
- PCM1 Tone Legato
- PCM1 Tone Lev Velo Sens
- PCM1 Tone Velo Curve Type
- PCM1 Tone Nuance Sw
- PCM1 Tone Pan
- PCM1 Tone String Lev 1 … 6

#### Pitch

- PCM1 Pitch Pitch shift
- PCM1 Pitch Pitch Fine
- PCM1 Pitch Portamento Sw
- PCM1 Pitch Portamento Type
- PCM1 Pitch Portamento Time

#### Filter

- PCM1 Filter Filter Type
- PCM1 Filter CutOff
- PCM1 Filter Resonance
- PCM1 Filter Cutoff Velo Sens
- PCM1 Filter Cutoff Nuance Sens
- PCM1 Filter Cutoff Velo Curve
- PCM1 Filter Cutoff Keyfollow

#### TVF

- PCM1 TVF TVF Env Depth
- PCM1 TVF TVF Attack Time
- PCM1 TVF TVF Decay Time
- PCM1 TVF TVF Sustain level
- PCM1 TVF TVF Release Time
- PCM1 TVF TVF Attack Vel Sens
- PCM1 TVF TVF Attack Nuance Sens

#### TVA

- PCM1 TVA TVA Attack Time
- PCM1 TVA TVA Decay Time
- PCM1 TVA TVA Sustain Level
- PCM1 TVA TVA Release Time
- PCM1 TVA TVA Attack Vel Sens
- PCM1 TVA TVA Attack Nuance Sens
- PCM1 TVA Level Nuance Sens
- PCM1 TVA Release Mode

#### Pitch ENV

- PCM1 Pitch ENV Pitch Env Vel Sens
- PCM1 Pitch ENV Pitch Env Depth
- PCM1 Pitch ENV Attack Time
- PCM1 Pitch ENV Decay Time

#### LFO1 / LFO2

- PCM1 LFO1 LFO1 Rate
- PCM1 LFO1 LFO1 Pitch Depth
- PCM1 LFO1 LFO1 TVF Depth
- PCM1 LFO1 LFO1 TVA Depth
- PCM1 LFO1 LFO1 Pan Depth
- PCM1 LFO2 LFO2 Rate
- PCM1 LFO2 LFO2 Pitch Depth
- PCM1 LFO2 LFO2 TVF Depth
- PCM1 LFO2 LFO2 TVA Depth
- PCM1 LFO2 LFO2 Pan Depth

#### Other

- PCM1 Line Select
- PCM1 Tone1 Bend

---

### PCM2

Same structure as PCM1 (prefix **PCM2**):

- Tone: Switch, Number, Level, Octave, Chromatic, Legato, Lev Velo Sens, Velo Curve Type, Nuance Sw, Pan, String Lev 1–6
- Pitch: Pitch shift, Pitch Fine, Portamento Sw / Type / Time
- Filter: Filter Type, CutOff, Resonance, Cutoff Velo Sens, Cutoff Nuance Sens, Cutoff Velo Curve, Cutoff Keyfollow
- TVF: Env Depth, Attack / Decay / Sustain / Release Time, Attack Vel Sens, Attack Nuance Sens
- TVA: Attack / Decay / Sustain / Release Time, Attack Vel Sens, Attack Nuance Sens, Level Nuance Sens, Release Mode
- Pitch ENV: Pitch Env Vel Sens, Pitch Env Depth, Attack Time, Pitch Decay Time
- LFO1 / LFO2: Rate, Pitch / TVF / TVA / Pan Depth (each)
- Other: Line Select, Tone1 Bend

---

### Modeling tone

#### General / pitch / 12-string

- Modeling Tone Switch
- Modeling Tone Number
- Modeling Tone Level
- Modeling Tone String 1 Level … String 6 Level
- Modeling Pitch Pitch Shift
- Modeling Pitch Pitch Fine
- Modeling 12String Switch
- Modeling 12String Direct Level
- Modeling 12String Shift 1 … 6
- Modeling 12String Fine 1 … 6

#### E.GTR

- Modeling E.GTR Cla-ST,Mod-ST PU Select
- Modeling E.GTR HH,TE,LP,P9,RK,335,L4 PU Select
- Modeling E.GTR LIPS PU Select
- Modeling E.GTR PU Volume
- Modeling E.GTR PU Tone

#### Acoustic / world

- Modeling Acoustic Steel body Type / Body / Tone
- Modeling Acoustic Nylon body Body / Attack / Tone
- Modeling Sitar body PU Select / Sens / Body / Color / Decay / Buzz / Attack Level / Tone
- Modeling Banjo body Attack / Reso / Tone
- Modeling Resonator body Sustain / Resonance / Tone

#### E.Bass

- Modeling E.Bass Jazz PU Volume Rear
- Modeling E.Bass Jazz PU Volume Front
- Modeling E.Bass PU Volume
- Modeling E.Bass PU Tone

#### SYNTH

- **Analog:** Tone Mode, Comp, Filter Cutoff, Filter Reso, ENV MOD Sw / Sens / Attack, Pitch Sw, Pitch A / A Fine / B / B Fine, Duet, Sweep Sw / Rise / Fall, Vibrato Sw / Rate / Depth
- **Wave:** Synth Type, Synth Color
- **Filter Bass:** Filter Cutoff / Reso / Decay, Touch Sens, Color
- **Crystal:** Attack Length, MOD Tune, MOD Depth, Attack Level, Body Level, Sustain
- **Organ:** Feet 16 / 8 / 4, Sustain
- **Brass:** Filter Cutoff / Reso, Touch Sens, Sustain

#### Other modeling

- Modeling Line Select
- Modeling Tone Bend

---

### AMP

- AMP Switch
- AMP Gain
- AMP Level
- AMP Gain Switch
- AMP SOLO Switch
- AMP SOLO Level
- AMP Tone Bass / Middle / Treble / Presence / Bright
- AMP Speaker Type
- AMP Spkr Mic Type
- AMP Spkr Mic Distance
- AMP Spkr Mic Position
- AMP Spkr Mic Level

---

### MOD

#### Common / NS

- MOD Effect Switch
- MOD Pan
- MOD NS Switch
- MOD NS Threshold
- MOD NS Release

#### By MOD type

| Type | Targets |
|---|---|
| **OD/DS** | Type, Drive, Tone, Level |
| **WAH** | Mode, Type, Pedal Position, Sense, Frequency, Peak, Level |
| **COMP** | Sustain, Attack, Level |
| **LIMITER** | Threshold, Release, Level |
| **OCTAVE** | Effect, Direct |
| **PHASER** | Type, Rate, Depth, Resonance, Level |
| **FLANGER** | Rate, Depth, Manual, Resonance, Level |
| **TREMOLO** | Rate, Depth, Wave Shape, Level |
| **ROTARY** | Rate Slow, Rate Fast, Depth, Speed Select, Level |
| **UNI-V** | Rate, Depth, Level |
| **PAN** | Rate, Depth, Wave Shape, Level |
| **DELAY** | Type, Time, Feedback, Effect level |
| **CHORUS** | Type, Rate, Depth, Effect Level |
| **EQ** | Low Cut, Low Gain, Low-Mid Frequency / Q / Gain, High-Mid Frequency / Q / Gain, High Gain, High Cut, Level |

*(Picker labels are prefixed `MOD …`, e.g. `MOD OD/DS Drive`.)*

---

### MFX

#### Common

- MFX Effect Switch
- MFX Pan

#### By MFX type

| Type | Targets |
|---|---|
| **EQ** | Low Frequency / Gain, Mid 1 Frequency / Gain / Q, Mid 2 Frequency / Gain / Q, High Frequency / Gain, Level |
| **FILTER** (Super Filter) | Type, Slope, Cutoff, Resonance, Gain, Modulation Sw / Wave, Rate, Depth, Attack, Level |
| **PHASER** | Mode, Manual, Rate, Depth, Polarity, Resonance, Cross Feedback, Mix, Low Gain, High Gain, Level |
| **STEP PHASER** | Mode, Manual, Rate, Depth, Polarity, Resonance, Cross Feedback, Step rate, Mix, Low Gain, High Gain, Level |
| **RING MOD** | Frequency, Sense, Polarity, Low Gain, High Gain, Balance, Level |
| **TREMOLO** | Mod Wave, Rate, DEPTH, Low Gain, High Gain, Level |
| **AUTO PAN** | Mod Wave, Rate, Depth, Low Gain, High Gain, Level |
| **SLICER** | Pattern, SLICER Rate, Attack, Input Sync Sw / Threshold, Level |
| **VK ROTARY** | Speed, Brake, Woofer Slow/Fast Speed, Woofer Trans Up/Down, Woofer Level, Tweeter Slow/Fast Speed, Tweeter Trans Up/Down, Tweeter level, Spread, Low Gain, High Gain, Level |
| **HEXA-CHORUS** | Pre Delay, Rate, Depth, Pre Dly Deviation, Depth Deviation, Pan Deviation, Balance, Level |
| **SPACE-D** | Pre Delay, Rate, Depth, Phase, Low Gain, High Gain, Balance, Level |
| **FLANGER** | Filter Type, Cutoff Frequency, Pre Delay, Rate, Depth, Phase, Feedback, Low Gain, High Gain, Balance, Level |
| **STEP-FLANGER** | Filter Type, Cutoff Frequency, Pre Delay, Rate, Depth, Phase, Feedback, Step Rate, Low Gain, High Gain, Balance, Level |
| **AMP-SIM** | PreAmp Sw / Type / Volume / Master / Gain / Bass / Middle / Treble / Presence / Bright, Speaker Sw / Type, Speaker Mic Setting / Level, Speaker Direct Level, Pan, Level |
| **COMPRESSOR** | Attack, Threshold, Post Gain, Low Gain, High Gain, Level |
| **LIMITER** | Release, Threshold, Ratio, Post Gain, Low Gain, High Gain, Level |
| **3TAP DELAY** | Delay Left / Right / Center, Center Feedback, HF Damp, Left / Right / Center Level, Low Gain, High Gain, Balance, Level |
| **TIME DELAY** | Delay Time, Acceleration, Feedback, HF Damp, Low Gain, High Gain, Balance, Level |
| **LOFI COMP** | Pre Filter Type, LoFi Type, Post Filter Type / Cutoff, Low Gain, High Gain, Balance, Level |
| **PITCH-SHIFTER** | Coarse, Fine, Delay Time, Feedback, Low Gain, High Gain, Balance, Level |

*(Picker labels are prefixed `MFX …`, e.g. `MFX PHASER Rate`.)*

---

### Dedicated DELAY / REVERB / CHORUS / EQ

#### DELAY

- DELAY Effect Switch
- DELAY Type
- DELAY Time
- DELAY Feedback
- DELAY Effect Level
- DELAY MFX Send
- DELAY MOD Send
- DELAY BYPASS Send

#### REVERB

- REVERB Effect Switch
- REVERB Type
- REVERB Time
- REVERB High Cut
- REVERB Effect Level
- REVERB MFX Send
- REVERB MOD Send
- REVERB BYPASS Send

#### CHORUS

- CHORUS Effect Switch
- CHORUS Type
- CHORUS Rate
- CHORUS Depth
- CHORUS Effect Level
- CHORUS MFX Send
- CHORUS MOD Send
- CHORUS BYPASS Send

#### EQ

- EQ Effect Switch
- EQ Low Cut
- EQ Low Gain
- EQ Low-Mid Frequency / Q / Gain
- EQ High-Mid Frequency / Q / Gain
- EQ High Gain
- EQ High Cut
- EQ Level
- EQ Character

---

### Patch / master / pickup

- NORMAL PICKUP Level
- NORMAL PICKUP Line Select
- ALTERNATE TUNING Switch
- PATCH TEMPO
- GUITAR OUT SOURCE
- PATCH LEVEL

---

## Example setups (from the manual)

### CTL → Delay on/off (all patches)

| Location | Setting |
|---|---|
| SYSTEM CTL FUNCTION | **DELAY SW** |

Raise DELAY **EFFECT LEVEL** first so the switch is audible.

### EXP → Modeling tone volume (one patch)

| Location | Setting |
|---|---|
| SYSTEM EXP FUNCTION | **PATCH SETTING** |
| MASTER EXP → TONE VOLUME | PCM1 OFF, PCM2 OFF, **MODELING ON**, NORMAL PU OFF |

Confirm modeling tone switch is ON.

### CTL → PCM1 bend +1 octave (ASSIGN + INT PDL)

| Location | Setting |
|---|---|
| SYSTEM CTL FUNCTION | **PATCH SETTING** |
| ASSIGN 1 SWITCH | ON |
| TARGET | PCM1 TONE1 BEND (as in OM example) |
| TARGET MIN / MAX | 0 / +12 |
| SOURCE | **INT PDL** |
| SOURCE MODE | **MOMENT** |
| ACT RANGE LO / HI | 0 / 127 |
| INT TRIG | **CTL** |
| INT TIME | 20 (adjust rise time) |
| INT CURVE | LINEAR (or other) |

### CTL → AMP Solo + Gain SW (two ASSIGNs)

| ASSIGN | TARGET | MIN→MAX | SOURCE | MODE |
|---|---|---|---|---|
| 1 | AMP SOLO SW | OFF→ON | CTL | TOGGLE |
| 2 | AMP GAIN SW | LOW→MID | CTL | TOGGLE |

SYSTEM CTL FUNCTION = **PATCH SETTING**.

---

## Important note (saving assigned parameters)

If you edit a parameter that is assigned to a controller and then save the patch, **some parameters are overwritten with the controller’s initial state**, so the edit can appear not to have been saved.

**Avoidance:** Set PEDAL/GK CTL **FUNCTION** to **OFF** and/or set ASSIGN **SWITCH** to **OFF** for the function being assigned while you edit and save that parameter (OM p. 63).

---

## Cross-references

| Topic | Doc |
|---|---|
| MASTER PEDAL/GK CTL function list | [MASTER.md](./MASTER.md) |
| SYSTEM PEDAL/GK CTL + PATCH SETTING + ASSIGN HOLD | [SYSTEM.md](./SYSTEM.md) |
| Assign Target values / constraints | See table under [Assign Target catalog](#assign-target-catalog) |
| Effect docs index | [Effects_Index.md](./Effects_Index.md) |
| Settings index | [Settings_Index.md](./Settings_Index.md) |
