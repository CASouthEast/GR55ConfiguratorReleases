# Settings for the Entire GR-55 (SYSTEM)

Global system settings from the Roland GR-55 Owner’s Manual (`GR-55_OM.pdf`, parameter list pp. 74–80; overview pp. 69–73), extracted via **macOS PDFKit**.

**Controller ASSIGN 1–8** (per-patch) → [ASSIGNS.md](./ASSIGNS.md)  
**Per-patch MASTER** (including PEDAL/GK CTL when set to PATCH SETTING) → [MASTER.md](./MASTER.md)

---

## Overview

SYSTEM settings affect the whole unit (GK pickups, global pedals, MIDI/USB, output, tuner, etc.).

**Basic operation**
1. Press **[EDIT]** → PAGE to the **SYSTEM** tab.
2. Edit parameters (many take effect immediately; some need confirming as in the manual).

Related how-to sections in the OM: switching/renaming GK sets (p. 70), GUITAR OUT global vs per-patch (p. 70–71), BACKUP (p. 72), pedal CALIB / FACTORY RESET (p. 73).

---

## GK SETTING

Edits the selected GK SET (1–10). The set used when MASTER **GK SET SELECT** = **SYSTEM** is chosen here; also select which set to edit.

| Parameter | Values | Description / constraints |
|---|---|---|
| **GK SET SELECT** | 1–10 | GK SET for SYSTEM mode / set being edited. |
| **NAME** | (up to 8 characters) | Name of the GK set. |
| **PU TYPE** | See below | Type of GK / piezo pickup (depends on GUITAR↔BASS). |
| **SCALE** | See below | Scale length (guitar or bass ranges). |
| **GK PU POS** | See below | Divided-pickup position (**BASS mode only**). |
| **PU PHASE** | NORMAL, INVERSE | Phase vs normal PU. Start NORMAL; if lows cut, try INVERSE. Easier to hear in MIX. |
| **PU DIRECTION** | NORMAL, REVERSE | Cable from 6th-string side vs 1st-string side. |
| **S1/S2 POSITION** | NORMAL, REVERSE | Swap GK [S1]/[S2] functions. |
| **NORM PU GAIN** | -20–+20 dB | Normal pickup input level. Set GK select switch to **MIX** when adjusting. |
| **PIEZO TONE LOW** | -10–+10 dB | Low EQ for piezo types only (*). |
| **PIEZO TONE HIGH** | -10–+10 dB | High EQ for piezo types only (*). |
| **DISTANCE PU↔BRIDGE** | Guitar 10.0–30.0 mm; Bass 5.5–55.5 mm | Distance to bridge. Ignored for listed piezo types (**). |
| **SENS** | 0–100 | Divided-pickup input sensitivity. |
| **VELOCITY DYNAMICS** | 1–10 | PCM velocity sensitivity; higher = easier high velocities. |
| **PLAY FEEL** | 1–5 | Velocity curve. Lower = more dynamic range; **5** = fixed velocity. |
| **LOW VELOCITY CUT** | 0–10 | Raises threshold to reduce accidental triggers from light touches. |
| **NUANCE DYNAMICS** | 0–10 | Nuance sensitivity; higher = easier Nuance. |
| **NUANCE TRIM** | 0–10 | Nuance threshold; lower if Nuance triggers too often. Aim: meter max on softest touch, little movement when playing normally. |
| **DOWN TUNE SHIFT** | 0 – -5 | Chromatic steps the instrument is down-tuned. |

`(*)` PIEZO TONE LOW/HIGH apply when PU TYPE is PIEZO, PIEZO F, PIEZO G, PIEZO L, or PIEZO R.  
`(**)` DISTANCE ignored for piezo types as listed in the manual (guitar vs bass lists differ slightly).

### PU TYPE

| Mode | Values |
|---|---|
| **GUITAR** | GK-3, GK-2A, PIEZO, PIEZO F (Fishman), PIEZO G (Graph Tech), PIEZO L (L.R. Baggs), PIEZO R (RMC) |
| **BASS** | GK-3B, GK-2B, PIEZO, PIEZO G, PIEZO R |

### SCALE

| Mode | Values |
|---|---|
| **GUITAR** | 500–660 mm, ST (648 mm), LP (628 mm) |
| **BASS** | 710–940 mm, SHORT (760 mm), MEDIUM (812 mm), LONG JB/PB (864 mm), EXTRA LONG (914 mm) |

### GK PU POS (BASS only)

| Value | Description |
|---|---|
| **4STR-1 / 4STR-2 / 4STR-3** | 4-string bass positions. |
| **5STR Lo1 / Lo2** | 5-string Low B–G. |
| **5STR Hi1 / Hi2** | 5-string E–Hi C. |
| **6STR** | 6-string bass. |

---

## OUTPUT SELECT

Global output voicing for the destination amp/system.

| Value | Description |
|---|---|
| **LINE/PHONES** | Headphones, keyboard/bass amp, mixer, or digital recorder. |
| **JC-120** | Guitar input of a Roland JC-120. |
| **SMALL** | Small guitar amp. |
| **COMBO** | Combo guitar amp (amp+speaker) other than JC-120; JC-120 setting may work better on some amps. |
| **STACK** | Stack guitar amp (separate amp/speaker). |
| **JC-120 RETURN** | JC-120 RETURN jack. |
| **COMBO RETURN** | Combo RETURN jack. |
| **STACK RETURN** | Stack RETURN, or guitar power amp + cabinet. |
| **B-AMP WITH TWEETER** | Bass amp with tweeter. |
| **B-AMP NO TWEETER** | Bass amp without tweeter (HF corrected). |

---

## PEDAL/GK CTL (SYSTEM)

Same controllers as MASTER (**CTL, EXP SW, GK S1, GK S2, EXP, EXP ON, GK VOL**), but at system scope.

### Extra / differing values

| Parameter | Values | Description |
|---|---|---|
| **FUNCTION** | **PATCH SETTING**, plus same function list as MASTER | **PATCH SETTING** = use the current patch’s MASTER PEDAL/GK CTL assignment. Any other value = same function for **all** patches. |
| **ASSIGN HOLD** | OFF, ON | When switching patches: ON = carry expression pedal / GK volume state into the next patch; OFF = do not. |

HOLD types, TONE SW, PATCH VOLUME, PITCH BEND, MODULATION, CROSS FADER, DELAY/REVERB/CHORUS LEVEL, and **MOD CONTROL** match MASTER — see [MASTER.md](./MASTER.md).

Workflow for global vs per-patch pedals: [ASSIGNS.md](./ASSIGNS.md).

---

## MIDI/USB

### GENERAL

| Parameter | Values | Description / constraints |
|---|---|---|
| **PATCH CH** | 1–16 | MIDI channel for patch control. Prog changes on this channel switch patches; received CCs go to ASSIGN SOURCE. |
| **PC RX SWITCH** | OFF, ON | ON = external program changes switch patches (optionally via RX BANK/PC MAP). |
| **PC TX SWITCH** | OFF, ON | ON = transmit program change when switching patches. |
| **MIDI SYNC** | OFF, ON | ON = tempo-sync effects follow external clock. |
| **MIDI CLOCK OUT** | OFF, ON | ON = transmit MIDI clock. **Not transmitted if MIDI SYNC is ON.** |
| **V-LINK TX CH** | 1–16 | MIDI channel for V-LINK devices. |

### GTR-MIDI

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | OFF = no guitar performance data from MIDI OUT. |
| **MODE** | MONO, POLY | MONO = one channel per string (6 channels); POLY = all strings on one channel. |
| **CHROMATIC** | OFF, ON | ON = play external module chromatically. |
| **STRING CH** | 1–11 | Base TX channel. MONO uses six channels starting here; POLY uses this channel only. |
| **DATA THIN** | OFF, ON | ON = thin pitch-bend data to reduce MIDI volume. |
| **CTL PDL CC** | OFF, 1–31, 64–95 | CC for [CTL] pedal; OFF = don’t transmit. |
| **EXP PDL CC** | OFF, 1–31, 64–95 | CC for expression pedal. |
| **EXP PDL BEND RANGE** | -24–+24 | Max pitch bend when EXP sends bend to external module; **0** = don’t send bend. |
| **GK VOL CC** | OFF, 1–31, 64–95 | CC for GK volume. |
| **GK S1 CC / GK S2 CC** | OFF, 1–31, 64–95 | CC for GK [S1]/[S2]. |

| MODE | Notes |
|---|---|
| **MONO** | Different tone/bend per string; needs multitimbral module. |
| **POLY** | Simpler / fewer channels; only one tone for all strings. |

### RX PC MAP

| Parameter | Values | Description |
|---|---|---|
| **MAP SELECT** | FIX, PROG | FIX = fixed patch for each PC; PROG = use BANK/PC map. |
| **BANK** | 0–63 | Bank MSB for the map (LSB fixed at 0). |
| **PC → patch** | [LEAD]/[RHYTHM]/[OTHER]/[USER] **-\* | Patch for each incoming program change / bank. |

---

## USB (under MIDI/USB area)

| Parameter | Values | Description / constraints |
|---|---|---|
| **AUDIO IN LEVEL** | 0–200 | Digital audio from USB (computer). |
| **AUDIO OUT LEVEL** | 0–200 | Digital audio to USB. |
| **DIRECT MONITOR** | OFF, ON | OFF = hear only what the computer passes through; ON = GR-55 sound direct (normal when using GR-55 alone). |

---

## OTHER

| Parameter | Values | Description / constraints |
|---|---|---|
| **GUITAR OUT SOURCE** | PATCH, OFF, NORMAL PU, MODELING, BOTH | **PATCH** = use each patch’s MASTER GUITAR OUT; any other value = that source for **all** patches. |
| **MASTER TUNE** | 435–445 Hz | Reference pitch (factory 440 Hz). |
| **TUNER MUTE** | OFF, ON | ON = mute while tuning (factory ON); OFF = sound while tuning. |
| **AUDIO PLAYER LEVEL** | 0–200 | Audio player volume. |
| **LCD CONTRAST** | 1–10 | Display contrast. |
| **AUTO POWER OFF** | OFF, ON | ON = power off after 10 hours idle (warning ~15 min before); factory ON. |

### BACKUP / INITIALIZE icons

| Icon | Refer to |
|---|---|
| **BACKUP/RESTORE** | Saving/restoring to USB memory (p. 72). |
| **PEDAL CALIBRATION** | Pedal sensitivity CALIB (p. 73). |
| **FACTORY RESET** | Restoring factory settings (p. 73). |
| **GUITAR-BASS SELECT** | GUITAR↔BASS mode (p. 9). |

---

## MASTER vs SYSTEM (quick)

| Topic | MASTER | SYSTEM |
|---|---|---|
| Scope | Current patch | Entire unit |
| Pedal functions | Used when SYSTEM FUNCTION = PATCH SETTING | Global function, or PATCH SETTING |
| GK SET | Which set (SYSTEM / 1–10) | Define sets 1–10 (PU, scale, sens, …) |
| GUITAR OUT | Per-patch source | PATCH vs fixed global source |
| Tempo | PATCH TEMPO | MIDI SYNC / clock |
| Assigns 1–8 | Yes (see ASSIGNS.md) | No (ASSIGN HOLD only) |
