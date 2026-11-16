# AMP & MFX Guitar Amp Simulator

Parameter descriptions, value ranges, and constraints extracted from the Roland GR-55 Owner’s Manual (`GR-55_OM.pdf`), Effect Settings (EFFECT) parameter list.

Part of the effects doc set — see [Effects_Index.md](./Effects_Index.md). Other MFX types: [MFX.md](./MFX.md).

**Source pages**
- AMP: pp. 41–42
- MFX Guitar Amp Simulator: p. 50
- Context: effects overview p. 38; MFX common parameters p. 45

**Note:** Parameters marked with `#` in the manual appear on the effect quick-edit screen (EDIT → EFFECT). Which `#` parameters are shown depends on the selected effect/type.

Company and product names in amp type descriptions are trademarks of their respective owners; the manual uses them to describe COSM-simulated sounds.

---

## Context: where AMP and MFX sit

The GR-55 has seven effect processors: **AMP**, **MFX**, **MOD**, **CHORUS**, **DELAY**, **REVERB**, **EQ**.

In Structure 1 (simplified):
- **AMP** is a preamp/speaker block typically fed by the modeling tone.
- **MFX** is a multi-effect block (one algorithm at a time). Type **14: GUITAR AMP SIMULATOR** is a separate amp simulation available inside MFX, distinct from the dedicated AMP block.

---

## AMP (dedicated amp block)

Turns amp modeling on/off, selects amp type, tone, gain structure, and speaker/mic simulation.

### TYPE group

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | Turns the AMP OFF/ON. |
| **TYPE** | See [AMP TYPE](#amp-type) | Specifies the AMP type. |
| **GAIN #** | 0–120 | Adjusts the distortion of the amp. |
| **LEVEL** | 0–100 | Adjusts the overall volume of the preamp. Take care not to raise LEVEL excessively. |
| **GAIN SW** | LOW, MIDDLE, HIGH | Switches the amount of the amp’s distortion in three levels. LOW, MIDDLE, and HIGH provide progressively greater distortion. |
| **SOLO SW** | OFF, ON | Switches to a sound suitable for solos. |
| **SOLO LEVEL** | 0–100 | Adjusts the volume when SOLO SW is ON. |

### TONE group

| Parameter | Values | Description / constraints |
|---|---|---|
| **BASS #** | 0–100 | Adjusts the tone for the low frequency range. |
| **MIDDLE #** | 0–100 | Adjusts the tone for the middle frequency range. |
| **TREBLE #** | 0–100 | Adjusts the tone for the high frequency range. |
| **PRESENCE** | 0–100 | Adjusts the tone for the ultra high frequency range. **When TYPE is** VO DRIVE, VO LEAD, VO CLEAN, MATCH DRIVE, FAT MATCH, or MATCH LEAD, **PRESENCE functions as a high-cut filter.** |
| **BRIGHT** | OFF, ON | See constraints below. |

**BRIGHT constraints**

- Setting is only available when TYPE is one of: BOSS CLEAN, JC-120, JAZZ COMBO, CLEAN TWIN, PRO CRUNCH, TWEED, BOSS CRUNCH, BLUES, STACK CRUNCH, BG LEAD, BG DRIVE, or BG RHYTHM.
- **OFF** — BRIGHT is not used.
- **ON** — BRIGHT is switched on to create a lighter and crisper tone.

### SP/MIC group

| Parameter | Values | Description / constraints |
|---|---|---|
| **SPEAKER TYPE** | See below | Selects the speaker type. |
| **MIC TYPE** | See below | Selects the simulated microphone type. |
| **MIC DISTANCE** | OFF MIC, ON MIC | Simulates the distance between the microphone and speaker. |
| **MIC POSITION** | CENTER, 1–10 | Simulates the microphone position. |
| **MIC LEVEL** | 0–100 | Adjusts the volume of the microphone. |

#### SPEAKER TYPE

| Value | Description |
|---|---|
| **OFF** | Turns off the speaker simulator. |
| **ORIGIN** | The built-in speaker of the amp you selected with AMP TYPE. |
| **1x8"** | An open-back speaker cabinet with one 8-inch speaker. |
| **1x10"** | An open-back speaker cabinet with one 10-inch speaker. |
| **1x12"** | An open-back speaker cabinet with one 12-inch speaker. |
| **2x12"** | An open-back speaker cabinet with two 12-inch speakers. |
| **4x10"** | A closed-back speaker cabinet with four 10-inch speakers. |
| **4x12"** | A closed-back speaker cabinet with four 12-inch speakers. |
| **8x12"** | A double stack of two closed-back cabinets, each with four 12-inch speakers. |

#### MIC TYPE

| Value | Description |
|---|---|
| **DYN57** | Models the Shure SM57, a general-use dynamic microphone used for instruments and vocals. Optimal for use in miking guitar amps. |
| **DYN421** | Models the Sennheiser MD421, a dynamic microphone with extended low end. |
| **CND451** | Models the AKG C451B, a small-diaphragm condenser microphone for use with instruments. |
| **CND87** | Models the Neumann U 87, a condenser microphone with flat response. |
| **FLAT** | Simulates a microphone with perfectly flat response. |

#### MIC DISTANCE

| Value | Description |
|---|---|
| **OFF MIC** | Microphone is placed at a distance from the speaker. |
| **ON MIC** | Microphone is placed so it’s near the speaker. |

#### MIC POSITION

| Value | Description |
|---|---|
| **CENTER** | Microphone is set up so it’s pointed at the center of the speaker’s cone. |
| **1–10** | Microphone is positioned at the specified distance from the center of the speaker’s cone. |

### AMP TYPE

| Value | Description |
|---|---|
| **01: BOSS CLEAN** | A clean sound that is smooth and warm. |
| **02: JC-120** | The sound of the Roland JC-120. |
| **03: JAZZ COMBO** | A sound suited to jazz. |
| **04: FULL RANGE** | A sound with flat response. Good for acoustic guitar. |
| **05: CLEAN TWIN** | Models a Fender Twin Reverb. |
| **06: PRO CRUNCH** | Models a Fender Pro Reverb. |
| **07: TWEED** | Models a Fender Bassman 4 x 10" Combo. |
| **08: DELUXE CRUNCH** | Models a Fender Deluxe Reverb. |
| **09: BOSS CRUNCH** | A crunch sound that faithfully reproduces picking nuances. |
| **10: BLUES** | A sound suited to blues. |
| **11: WILD CRUNCH** | A crunch sound with wild distortion. |
| **12: STACK CRUNCH** | A crunch sound with high gain. |
| **13: VO DRIVE** | Models the drive sound of a VOX AC-30TB. Suited to sixties-style British rock. |
| **14: VO LEAD** | Models the lead sound of the VOX AC-30TB. |
| **15: VO CLEAN** | Models the clean sound of the VOX AC-30TB. |
| **16: MATCH DRIVE** | Models the sound produced using the left input on a Matchless D/C-30, a modern tube amp widely used in styles from blues to rock. |
| **17: FAT MATCH** | Models the sound of a Matchless modified for high gain. |
| **18: MATCH LEAD** | Models the sound produced using the right input on a Matchless D/C-30. |
| **19: BG LEAD** | Models the lead sound of the Mesa/Boogie combo amp, a tube amp that was very popular in the late ‘70s and ‘80s. |
| **20: BG DRIVE** | Models a Mesa/Boogie with Treble Shift Switch on. |
| **21: BG RHYTHM** | Models the rhythm channel of a Mesa/Boogie combo amp. |
| **22: MS1959 I** | Models the sound produced using Input I on a Marshall 1959 Super Lead amp. A trebly sound suited to hard rock. |
| **23: MS1959 I+II** | The sound of connecting Inputs I and II of the Marshall 1959 amp in parallel, creating a sound with a stronger low end than I. |
| **24: MS HIGAIN** | Models the sound of a Marshall modified with a midrange boost. |
| **25: MS SCOOP** | A Marshall sound that’s been tweaked for a metal sound. |
| **26: R-FIER VINTAGE** | Models the sound of the Channel 2 VINTAGE Mode on the Mesa/Boogie DUAL Rectifier. |
| **27: R-FIER MODERN** | Models the sound of the Channel 2 MODERN Mode on the Mesa/Boogie DUAL Rectifier. |
| **28: R-FIER CLEAN** | Models the sound of the Channel 1 CLEAN Mode on the Mesa/Boogie DUAL Rectifier. |
| **29: T-AMP LEAD** | Models AMP3 on a Hughes & Kettner TriAmp. |
| **30: T-AMP CRUNCH** | Models AMP2 on a Hughes & Kettner TriAmp. |
| **31: T-AMP CLEAN** | Models AMP1 on a Hughes & Kettner TriAmp. |
| **32: BOSS DRIVE** | A drive sound producing awesome distortion. |
| **33: SLDN** | Models a Soldano SLO-100, a very popular tube amp in the 1980s. |
| **34: LEAD STACK** | A lead sound with high gain. |
| **35: HEAVY LEAD** | A powerful lead sound featuring extreme distortion. |
| **36: BOSS METAL** | A metal sound suited to heavy riffs. |
| **37: 5150 DRIVE** | Models the lead channel of a Peavey EVH 5150. |
| **38: METAL LEAD** | A lead sound suited to metal. |
| **39: EDGE LEAD** | A sharp sound suited for lead play. |
| **40: BASS CLEAN** | A clean sound that is great for use with bass guitars. |
| **41: BASS CRUNCH** | A crunch sound with natural distortion that sounds great with bass guitars. |
| **42: BASS HIGAIN** | A high-gain sound suitable for use with bass guitars. |

**AMP TYPE–dependent constraints (summary)**

| Constraint | Applies when TYPE is… |
|---|---|
| PRESENCE acts as high-cut filter | VO DRIVE, VO LEAD, VO CLEAN, MATCH DRIVE, FAT MATCH, MATCH LEAD |
| BRIGHT is available | BOSS CLEAN, JC-120, JAZZ COMBO, CLEAN TWIN, PRO CRUNCH, TWEED, BOSS CRUNCH, BLUES, STACK CRUNCH, BG LEAD, BG DRIVE, BG RHYTHM |

---

## MFX — common parameters (all MFX types)

These apply regardless of which MFX algorithm is selected (including Guitar Amp Simulator).

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | Turns MFX OFF/ON. |
| **PAN** | L50–R50 | Adjusts the pan position. **Valid even if SWITCH is OFF.** |
| **EFFECT TYPE** | See MFX Type list | Specifies the MFX type. |

MFX types (01–20): EQ, SUPER FILTER, PHASER, STEP PHASER, RING MODULATOR, TREMOLO, AUTO PAN, SLICER, VK ROTARY, HEXA-CHORUS, SPACE-D, FLANGER, STEP FLANGER, **GUITAR AMP SIMULATOR**, COMPRESSOR, LIMITER, 3TAP PAN DELAY, TIME CTRL DELAY, LOFI COMPRESS, PITCH SHIFTER.

---

## MFX Type 14: GUITAR AMP SIMULATOR

Effect that simulates the sound of a guitar amplifier.

Signal path (manual diagram): stereo in → **Pre Amp** → **Speaker** → pan → stereo out.

### Parameters

| Parameter | Values | Description / constraints |
|---|---|---|
| **PRE AMP SW** | OFF, ON | Turns the amp switch on/off. |
| **PRE AMP TYPE #** | JC-120, CLEAN TWIN, MATCH DRIVE, BG LEAD, MS1959I, MS1959II, MS1959I+II, SLDN LEAD, METAL 5150, METAL LEAD, OD-1, OD-2 TURBO, DISTORTION, FUZZ | Specifies the type of guitar amp. |
| **PRE AMP VOLUME #** | 0–100 | Adjusts the volume and amount of distortion of the amp. |
| **PRE AMP MASTER #** | 0–100 | Adjusts the volume of the entire pre-amp. |
| **PRE AMP GAIN** | LOW, MIDDLE, HIGH | Specifies the amount of pre-amp distortion. |
| **PRE AMP BASS** | 0–100 | Adjust the tone of the bass/mid/treble frequency ranges. |
| **PRE AMP MIDDLE** | 0–100 | Same as above. **Cannot be set if PRE AMP TYPE is MATCH DRIVE.** |
| **PRE AMP TREBLE** | 0–100 | Same as above. |
| **PRE AMP PRESENCE** | 0–100 | Adjusts the tone of the ultra-high frequency range. |
| **PRE AMP BRIGHT** | OFF, ON | Turning this ON produces a sharper and brighter sound. **Available only if PRE AMP TYPE is JC-120, CLEAN TWIN, or BG LEAD.** |
| **SPEAKER SW** | OFF, ON | Determines whether the signal passes through the speaker (ON), or not (OFF). |
| **SPEAKER TYPE #** | See [Speaker type table](#specifications-for-each-speaker-type) | Selects the type of speaker. |
| **MIC SETTING** | 1, 2, 3 | Adjusts the location of the microphone that’s capturing the sound of the speaker. Three steps; the microphone becomes more distant as the value increases. |
| **MIC LEVEL** | 0–100 | Adjusts the volume of the microphone. |
| **DIRECT LEVEL** | 0–100 | Adjusts the volume of the direct sound. |
| **PAN** | L50–R50 | Adjusts the stereo location of the output sound. |
| **LEVEL** | 0–100 | Adjusts the output volume. |

**PRE AMP TYPE–dependent constraints (summary)**

| Constraint | Applies when PRE AMP TYPE is… |
|---|---|
| PRE AMP MIDDLE cannot be set | MATCH DRIVE |
| PRE AMP BRIGHT is available | JC-120, CLEAN TWIN, BG LEAD |

### Specifications for each speaker type

The “Speaker” column indicates the diameter of each speaker unit (in inches) and the number of units.

| Type | Cabinet | Speaker | Microphone |
|---|---|---|---|
| **SMALL 1** | Small open-back enclosure | 10 | Dynamic |
| **SMALL 2** | Small open-back enclosure | 10 | Dynamic |
| **MIDDLE** | Open-back enclosure | 12 x 1 | Dynamic |
| **JC-120** | Open-back enclosure | 12 x 2 | Dynamic |
| **BUILT-IN 1** | Open-back enclosure | 12 x 2 | Dynamic |
| **BUILT-IN 2** | Open-back enclosure | 12 x 2 | Condenser |
| **BUILT-IN 3** | Open-back enclosure | 12 x 2 | Condenser |
| **BUILT-IN 4** | Open-back enclosure | 12 x 2 | Condenser |
| **BUILT-IN 5** | Open-back enclosure | 12 x 2 | Condenser |
| **BG STACK 1** | Sealed enclosure | 12 x 2 | Condenser |
| **BG STACK 2** | Large sealed enclosure | 12 x 2 | Condenser |
| **MS STACK 1** | Large sealed enclosure | 12 x 4 | Condenser |
| **MS STACK 2** | Large sealed enclosure | 12 x 4 | Condenser |
| **METAL STACK** | Large double stack | 12 x 4 | Condenser |
| **2-STACK** | Large double stack | 12 x 4 | Condenser |
| **3-STACK** | Large triple stack | 12 x 4 | Condenser |

---

## AMP vs MFX Guitar Amp Simulator (quick comparison)

| Aspect | Dedicated AMP | MFX Guitar Amp Simulator |
|---|---|---|
| Role | Main amp/speaker block in the effect chain | One of 20 MFX algorithms |
| Amp models | 42 types (BOSS CLEAN … BASS HIGAIN) | 14 types (JC-120 … FUZZ, including OD/DS flavors) |
| Gain range | GAIN 0–120; GAIN SW LOW/MIDDLE/HIGH | VOLUME 0–100; GAIN LOW/MIDDLE/HIGH |
| Solo | SOLO SW / SOLO LEVEL | — |
| Speaker | OFF, ORIGIN, 1x8" … 8x12" | SMALL 1 … 3-STACK (with cabinet/mic specs) |
| Mic model | MIC TYPE, DISTANCE, POSITION, LEVEL | MIC SETTING (1–3), MIC LEVEL, DIRECT LEVEL |
| Bright available on | 12 amp types (see above) | JC-120, CLEAN TWIN, BG LEAD only |
| Special notes | PRESENCE = high-cut on VO/MATCH family | MIDDLE disabled on MATCH DRIVE |
