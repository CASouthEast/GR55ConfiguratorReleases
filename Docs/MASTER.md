# Patch Settings (MASTER)

Per-patch master settings from the Roland GR-55 Owner’s Manual (`GR-55_OM.pdf`, pp. 54–59), extracted via **macOS PDFKit**.

**Assigns are documented separately** → [ASSIGNS.md](./ASSIGNS.md)

See also [SYSTEM.md](./SYSTEM.md) and [Effects_Index.md](./Effects_Index.md).

---

## Overview

MASTER settings belong to the current patch (PEDAL/GK CTL functions, tempo, GK set selection, GUITAR OUT, alternate tuning, V-LINK).

**Basic operation**
1. Select a patch and press **[EDIT]**.
2. Use PAGE buttons to select the **MASTER** tab.
3. Edit parameters; save the patch if you want to keep changes (p. 60).

---

## PEDAL/GK CTL

Assign functions to **[CTL]**, **EXP SW**, **GK S1**, **GK S2**, and to **EXP / EXP ON / GK VOL**.

Whether these per-patch assignments are used depends on SYSTEM → PEDAL/GK CTL: set the controller to **PATCH SETTING** to use MASTER, or another function for a global assignment. See [SYSTEM.md](./SYSTEM.md) and [ASSIGNS.md](./ASSIGNS.md).

### STATUS (CTL, EXP SW only)

| Parameter | Values | Description |
|---|---|---|
| **STATUS** | OFF, ON | Pedal on/off (reflects indicator illumination). |

### FUNCTION — CTL, EXP SW, GK S1, GK S2

| Value | Description / constraints |
|---|---|
| **OFF** | No function assigned. |
| **HOLD** (CTL only) | Hold modes — see HOLD table below. |
| **TAP TEMPO** | Sets tempo to the timing of pedal presses. |
| **TONE SW** | Tone / normal PU on/off for SW OFF and SW ON states — see TONE SW table. |
| **AMP SW** | Switches AMP on/off. |
| **MOD SW** | Switches MOD on/off. |
| **MFX SW** | Switches MFX on/off. |
| **DELAY SW** | Switches DELAY on/off. |
| **REVERB SW** | Switches REVERB on/off. |
| **CHORUS SW** | Switches CHORUS on/off. |
| **AUDIO PLAYER PLAY/STOP** | Starts/stops the audio player. |
| **AUDIO PLAYER SONG INC** | Selects next audio file in USB memory. |
| **AUDIO PLAYER SONG DEC** | Selects previous audio file. |
| **AUDIO PLAYER SW** | Same as panel **[AUDIO PLAYER]** button. |
| **V-LINK SW** | Same as panel **[V-LINK]** button. |
| **LED MOMENT** | Indicator lights while pressed; goes out when released. |
| **LED TOGGLE** | Indicator toggles each press. |

#### HOLD (CTL only)

| Parameter | Values | Description |
|---|---|---|
| **HOLD TYPE** | 1–4 | Hold behavior (see below). |
| **SWITCH MODE** | LATCH, MOMENT | LATCH = toggle each press; MOMENT = hold only while pressed. |
| **PCM TONE 1 / 2** | OFF, ON | OFF = that tone is not held. |

| HOLD TYPE | Behavior |
|---:|---|
| **1** | Notes sounding when pressed are held; new notes while held are also held. Unlike type 4, a new note on the same string replaces the previous one (no gap when jumping frets). |
| **2** | Notes sounding when pressed stay held while pedal is down; notes played after pressing do **not** sound. |
| **3** | Notes sounding when pressed stay held while pedal is down; later notes sound but are **not** held. |
| **4** | Notes sounding when pressed are held; successive notes while held are also held. |

#### TONE SW

Controls on/off for each tone and the normal pickup.

| State | Applies when |
|---|---|
| **SW OFF** | STATUS of CTL/EXP SW is OFF; for GK S1/S2 = default (before press). |
| **SW ON** | STATUS is ON; for GK S1/S2 = after switch pressed once. |

For each state: **PCM TONE 1**, **PCM TONE 2**, **MODELING TONE**, **NORMAL PU** — each **OFF/ON**.

### FUNCTION — EXP, EXP ON, GK VOL

Two different functions can be assigned to the expression pedal: one for when the expression pedal switch is **off** (EXP), one when it is **on** (EXP ON). GK VOL uses the same function list.

| Value | Description / constraints |
|---|---|
| **OFF** | No function. |
| **PATCH VOLUME** | Adjusts patch volume. |
| **TONE VOLUME** | Volume of tones / normal PU — per-source enable below. |
| **PITCH BEND** | Pitch of PCM1, PCM2, modeling — see below. |
| **MODULATION** | Modulation depth for PCM1/PCM2 — see below. |
| **CROSS FADER** | Volume balance between tones — polarity per source. |
| **DELAY LEVEL** | Delay effect level (MIN/MAX). |
| **REVERB LEVEL** | Reverb effect level (MIN/MAX). |
| **CHORUS LEVEL** | Chorus effect level (MIN/MAX). |
| **MOD CONTROL** | Principal MOD-type parameter — see below. |

#### TONE VOLUME

| Sub-parameter | Values | Description |
|---|---|---|
| **PCM TONE 1 / 2** | OFF, ON | OFF = control does not affect that tone. |
| **MODELING TONE** | OFF, ON | Same. |
| **NORMAL PU** | OFF, ON | Same. |

#### PITCH BEND

Changes pitch of PCM1, PCM2, and modeling tone.

| Sub-parameter | Values | Description / constraints |
|---|---|---|
| **DEPTH** | -12–+12 | Max pitch change when pedal fully depressed. |
| **PCM TONE 1 / 2** | OFF, ON | OFF = don’t change that tone’s pitch. |
| **MODELING TONE** | OFF, ON | OFF = don’t change modeling pitch. |

**Constraint:** No effect if modeling **12STR SW** is ON (p. 29).

#### MODULATION

Controls modulation depth for PCM1 and PCM2 (effect depends on selected PCM tone).

| Sub-parameter | Values | Description |
|---|---|---|
| **MIN** | 0–100 | Depth when pedal fully released. |
| **MAX** | 0–100 | Depth when pedal fully depressed. |
| **PCM TONE 1 / 2** | OFF, ON | OFF = don’t apply to that tone. |

#### CROSS FADER

| Sub-parameter | Values | Description |
|---|---|---|
| **PCM TONE 1 / 2 POLARITY** | OFF, TOE, HEEL | OFF = no volume change; TOE = louder as you depress; HEEL = louder as you lift. |
| **MODELING TONE POLARITY** | OFF, TOE, HEEL | Same. |
| **NORMAL PU POLARITY** | OFF, TOE, HEEL | Same. |

#### DELAY / REVERB / CHORUS LEVEL

| Sub-parameter | Values | Description |
|---|---|---|
| **MIN** | 0–120 (DELAY), 0–100 (REVERB, CHORUS) | Level when pedal fully released. |
| **MAX** | same ranges | Level when pedal fully depressed. |

#### MOD CONTROL

Controls the principal parameter for each MOD type. Valid if **MOD SWITCH** is ON.

| Sub-parameter | Values | Description |
|---|---|---|
| **MIN / MAX** | (depends on MOD type) | Range of change for the assigned parameter. |

| MOD Type | Controlled parameter | MOD Type | Controlled parameter |
|---|---|---|---|
| OD/DS | DRIVE | TREMOLO | RATE |
| WAH (*1) | PEDAL POSITION | ROTARY | SPEED SELECT |
| COMP | SUSTAIN | UNI-V | RATE |
| LIMITER | THRESHOLD | PAN | RATE |
| OCTAVE | OCTAVE LEVEL | DELAY | EFFECT LEVEL |
| PHASER | RATE | CHORUS | EFFECT LEVEL |
| FLANGER | RATE | EQ | HI-MID FREQ |

`(*1)` Set WAH **MODE** to **MANUAL** (p. 43).

---

## PATCH TEMPO

| Parameter | Values | Description / constraints |
|---|---|---|
| **TEMPO** | 20–250 | Tempo for tempo-synchronized effects. If SYSTEM → MIDI/USB → **MIDI SYNC** is ON, tempo follows external MIDI clock. |

---

## GK SET

| Parameter | Values | Description |
|---|---|---|
| **GK SET SELECT** | SYSTEM, 1–10 | Normally leave at **SYSTEM**. If you swap guitars per patch, choose the GK SET for that guitar. |

GK SET definitions themselves are edited under SYSTEM → GK SETTING → [SYSTEM.md](./SYSTEM.md).

---

## GUITAR OUT

Per-patch signal from the GUITAR OUT jack. Whether this is used depends on SYSTEM → OTHER → **GUITAR OUT SOURCE** (PATCH vs fixed). See [SYSTEM.md](./SYSTEM.md).

| Parameter | Values | Description |
|---|---|---|
| **GUITAR OUT SOURCE** | OFF, NORMAL PU, MODELING, BOTH | OFF = nothing; NORMAL PU / MODELING / BOTH as named. |

---

## ALT-TUNING

| Parameter | Values | Description |
|---|---|---|
| **SWITCH** | OFF, ON | Turns ALT-TUNING on/off. |
| **TYPE** | See table | Alternate tuning preset or USER. |
| **USER SHIFT 1–6** | -24–+24 | Per-string shift when TYPE = USER. |

| TYPE | Description |
|---|---|
| **OPEN-D** | Open strings produce a D chord. |
| **OPEN-E** | Open strings produce an E chord. |
| **OPEN-G** | Open strings produce a G chord. |
| **OPEN-A** | Open strings produce an A chord. |
| **DROP-D** | Drops only the 6th string by one note (D). |
| **D-MODAL** | Drops 6th, 2nd, and 1st by one note (“DADGAD”). |
| **-1 STEP** | Each string one semitone lower. |
| **-2 STEP** | Each string a full step (two frets) lower. |
| **BARITONE** | Each string down a perfect fourth (five frets). |
| **NASHVL** | Raises 6th–3rd by one octave (like 12-string supplements alone). |
| **-1 OCT** | All strings down one octave. |
| **+1 OCT** | All strings up one octave. |
| **USER** | Tuning from USER SHIFT 1–6. |

---

## V-LINK

| Parameter | Values | Description |
|---|---|---|
| **PALETTE** | LAST, 1–32 | Palette on the V-LINK device for this patch. LAST = don’t switch. |
| **CLIP** | LAST, 1–32 | Clip when switching to this patch. LAST = don’t switch. |
| **NOTE CLIP CHANGE** | OFF, 1–4 | How playing switches clips (see below). |
| **EXP / EXP ON / GK VOL** | OFF, COLOR Cb, COLOR Cr, BRIGHT, PLAY SPEED | Controller → image control. |

### NOTE CLIP CHANGE

| Value | Description |
|---:|---|
| **OFF** | Performance does not switch clips. |
| **1** | Clip for the **lowest** sounding note. |
| **2** | Clip for the **highest** sounding note. |
| **3** | Clips output consecutively, regardless of notes. |
| **4** | Clip corresponding to the note. |

### Image controllers (EXP / EXP ON / GK VOL)

| Value | Description |
|---|---|
| **OFF** | No effect. |
| **COLOR Cb** | Hue change. |
| **COLOR Cr** | Hue change. |
| **BRIGHT** | Brightness. |
| **PLAY SPEED** | Motion-video playback speed. |
