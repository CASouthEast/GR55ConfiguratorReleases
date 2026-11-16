# Tone Types (PCM & Modeling)

Tone category / tone-number lists, descriptions, value ranges, and type-dependent constraints extracted from the Roland GR-55 Owner’s Manual (`GR-55_OM.pdf`) via **macOS PDFKit**.

**Source pages**
- Patch / tone overview: pp. 22–23
- Changing tone / PCM categories: p. 23
- PCM parameter list (context): pp. 25–28
- Modeling tone common + 12STR constraints: p. 29
- Modeling **TONE NUMBER** tables: pp. 30–31
- Modeling type-dependent parameters: pp. 31–37

**Note:** Parameters marked with `#` appear on the tone quick-edit screen. Which `#` parameters are shown depends on the selected tone.

Company and product names in modeling descriptions are trademarks of their respective owners; the manual uses them to describe COSM-simulated sounds.

---

## Tone architecture

A GR-55 patch contains:

| Tone | Role |
|---|---|
| **PCM TONE 1** | Synthesizer sound driven by guitar performance data |
| **PCM TONE 2** | Second PCM layer (same architecture as PCM 1) |
| **MODELING TONE** | Modeled sound based on the audio from each guitar string |

Availability also depends on the GK pickup select switch (GK / MIX) and each tone’s **SWITCH**. Even if a tone is available for the current GK switch position, there is no sound if its switch is OFF.

---

## PCM TONE — categories (tone types)

PCM tones are selected by **TONE CATEGORY** + **TONE NUMBER**. The owner’s manual lists categories and how many tones are in each group (not every individual PCM tone name).

### Common PCM parameters related to type selection

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | Turns the tone on/off. OFF = muted. |
| **TONE CATEGORY** | See table below | Selects the category (group) of tones. |
| **TONE NUMBER** | (per category) | Selects the tone number within the category. |
| **LEVEL** | 0–100 | Volume of the tone. |

**MEMO:** If you select the **Drums** tone category, there will be fewer editable parameters than those listed in the manual.

### PCM tone category list (with tone counts)

| Tone category | Number of tones | Tone category | Number of tones |
|---|---:|---|---:|
| Ac.Piano | 16 | Ensemble Strings | 22 |
| Pop Piano | 3 | Orchestral | 4 |
| E.Grand Piano | 2 | Solo Brass | 11 |
| E.Piano1 | 25 | Ensemble Brass | 7 |
| E.Piano2 | 13 | Wind | 7 |
| E.Organ | 32 | Flute | 12 |
| Pipe Organ | 5 | Sax | 7 |
| Reed Organ | 1 | Recorder | 4 |
| Harpsichord | 5 | Vox/Choir | 28 |
| Clav | 8 | Scat | 2 |
| Celesta | 1 | Synth Lead | 123 |
| Accordion | 6 | Synth Brass | 40 |
| Harmonica | 2 | Synth Pad/Strings | 84 |
| Bell | 21 | Synth Bellpad | 17 |
| Mallet | 22 | Synth PolyKey | 45 |
| Ac.Guitar | 18 | Synth FX | 31 |
| E.Guitar | 18 | Synth Seq/Pop | 11 |
| Dist.Guitar | 11 | Pulsating | 32 |
| Ac.Bass | 4 | Beat&Groove | 11 |
| E.Bass | 14 | Hit | 7 |
| Synth Bass | 87 | Sound FX | 37 |
| Plucked/Stroke | 18 | Percussion | 13 |
| Solo Strings | 9 | Drums | 14 |

---

## MODELING TONE — categories

### Category values (depend on Guitar/Bass mode)

| Mode (`GUITAR<->BASS`) | TONE CATEGORY values |
|---|---|
| **GUITAR** | E.GTR, AC, E.BASS, SYNTH |
| **BASS** | E.BASS, SYNTH, E.GTR |

| Parameter | Values | Description / constraints |
|---|---|---|
| **SWITCH** | OFF, ON | Turns the tone on/off. OFF = muted. |
| **TONE CATEGORY** | See above | Selects the category (group) of tones. |
| **TONE NUMBER** | 001– (per category) | Selects the tone. See tables below. |
| **LEVEL** | 0–100 | Volume of the tone. |

### 12STR constraints (type / setup dependent)

**12STR** cannot be used (parameters not shown) with any of the following:

- **MODELING TONE**
  - TONE CATEGORY = **E.BASS** (when set to GUITAR input), or **E.GTR** (when set to BASS input)
  - PITCH SHIFT ≠ 0
  - PITCH FINE ≠ 0
- **MASTER**
  - ALTERNATE TUNING SWITCH = ON

---

## MODELING TONE NUMBER tables

Tone numbers differ between Guitar mode and Bass mode for the same physical model. A dash (—) means that number is not available in that mode.

### TONE CATEGORY: E.GTR

| GUITAR | BASS | Name | Description |
|---|---|---|---|
| **01** | **16** | CLA-ST / ST | Models a Fender Stratocaster, a guitar with three traditional single-coil pickups. |
| **02** | — | MOD-ST | Models a guitar with three EMG active single-coil pickups. |
| **03** | — | H&H-ST | Models a Stratocaster-type guitar with humbucking pickups. |
| **04** | — | TE | Models a Fender Telecaster, a guitar with two single-coil pickups often used in blues and country music. |
| **05** | **17** | LP | Models a Gibson Les Paul Standard, a guitar with two humbucking pickups often used in rock. |
| **06** | — | P-90 | Models a Gibson Les Paul Junior, a guitar with two single-coil pickups affectionately referred to as “dog ear” or “soap bar” pickups. |
| **07** | — | LIPS | Models a Danelectro 56-U3, a guitar with three pickups with a distinctive silver “lipstick-style” appearance. |
| **08** | — | RICK | Models a Rickenbacker 360, a semi-hollow body guitar with two unique single-coil pickups. |
| **09** | — | 335 | Models a Gibson ES-335 Dot, a well-known semi-acoustic guitar with two humbucking pickups. |
| **10** | — | L4 | Models a Gibson L-4 CES, an acoustic-body guitar suitable for jazz, equipped with two humbucking pickups and strung with flat-wound strings. |

### TONE CATEGORY: E.BASS

| GUITAR | BASS | Name | Description |
|---|---|---|---|
| — | **01** | VINT JB | Models a Fender Jazz Bass made in the 1960s. |
| **16** | **02** | JB | Models a Fender Jazz Bass. |
| — | **03** | VINT PB | Models a Fender Precision Bass made in the early 1960s. |
| **17** | **04** | PB | Models a Fender Precision Bass. |
| — | **05** | M-MAN | Models a Music Man StingRay Bass made in the 1970s. |
| — | **06** | RICK | Models a Rickenbacker 4001. |
| — | **07** | T-BIRD | Models a Gibson Thunderbird. |
| — | **08** | ACTIVE | Models a typical bass equipped with active pickups. |
| — | **09** | VIOLIN | Models a Höfner violin bass. |

### TONE CATEGORY: AC

| GUITAR | BASS | Name | Description |
|---|---|---|---|
| **11** | — | STEEL | Models a steel-string guitar. |
| **12** | — | NYLON | Models a nylon-string guitar. |
| **13** | — | SITAR | Models a Coral electric sitar. The sitar’s distinctive buzz and tonal change are modeled. |
| **14** | — | BANJO | Models a conventional five-string banjo. |
| **15** | — | RESO | Models a Dobro-type resonator guitar. |

### TONE CATEGORY: SYNTH

| GUITAR | BASS | Name | Description |
|---|---|---|---|
| **18** | **10** | ANALOG GR | Classic analog polyphonic guitar (bass) synthesizer. Hexa-distortion, with a hexa-VCO and VCF generating independent pitch-shiftable sawtooth waves for the six strings. Reflects nuances of guitar/bass performance. To change pitch, use **PITCH A/B** (p. 36). |
| **19** | **11** | WAVE SYNTH | Directly processes the string signal from the GK pickup to produce the synthesizer sound. Natural performing feel. |
| **20** | **12** | FILTER BASS | Similar to a bass synth with a filter applied. |
| **21** | **13** | CRYSTAL | Instrument with a metallic resonance. |
| **22** | **14** | ORGAN | Sustained-tone instrument suitable for solo parts or slow songs. As on an organ, balance **FEET 16 / 8 / 4** to create the desired tone. |
| **23** | **15** | BRASS | Clear brass-type sound good for leads. |

---

## Modeling type-dependent parameters

Parameter sets change with the selected modeling tone. Shared **NS** (noise suppressor) for many models refers to the CLA-ST / MOD-ST / ST table.

### Shared NS (CLA-ST, MOD-ST, ST — and referenced by many other models)

| Parameter | Values | Description |
|---|---|---|
| **SWITCH** | OFF, ON | Turns the noise suppressor on/off. |
| **THRESHOLD #** | 0–100 | Adjust according to noise level. Higher if noise is high; lower if low. Keep guitar decay natural. |
| **RELEASE** | 0–100 | Time from when NS begins until volume is fully attenuated. |

---

### E.GTR — CLA-ST, MOD-ST, ST

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **PU SEL #** | REAR, R+C, CENTER, C+F, FRONT | Pickup selection (rear / center / front combinations). |
| PU | **VOLUME** | 0–100 | Volume. 0 = no sound. |
| PU | **TONE #** | 0–100 | Tone. Standard value 100; lower = softer. |
| NS | *(see Shared NS)* | | |

### E.GTR — H&H-ST, TE, LP, P-90, RICK, 335, L4

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **PU SEL #** | REAR, R+F, FRONT | Pickup selection. |
| PU | **VOLUME** | 0–100 | Volume. 0 = no sound. |
| PU | **TONE #** | 0–100 | Tone. Standard value 100; lower = softer. |
| NS | *(see Shared NS)* | | |

### E.GTR — LIPS

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **PU SEL #** | REAR, R+C, CENTER, C+F, FRONT, ALL | Includes ALL pickups. |
| PU | **VOLUME** | 0–100 | Volume. 0 = no sound. |
| PU | **TONE #** | 0–100 | Tone. Standard value 100; lower = softer. |
| NS | *(see Shared NS)* | | |

---

### AC — STEEL

| Group | Parameter | Values | Description |
|---|---|---|---|
| BODY | **BODY TYPE #** | MA28, TRP-0, GB45, GB SML, GLD40 | Acoustic guitar body type (see below). |
| BODY | **BODY** | 0–100 | Body resonation. Higher = more body; lower if feedback-prone. |
| BODY | **TONE #** | -50–+50 | Body tone. Standard 0; higher boosts high range. |
| NS | *(see Shared NS)* | | |

**BODY TYPE**

| Value | Description |
|---|---|
| **MA28** | Martin D-28 — older model known for exquisitely balanced sound. |
| **TRP-0** | Martin 000-28 — full low-end resonance and crisp, distinct contour. |
| **GB45** | Gibson J-45 — unique, seasoned tone with good response. |
| **GB SML** | Gibson B-25 — compact body; often used in blues. |
| **GLD40** | Guild D-40 — warm body resonance with delicate string resonance. |

### AC — NYLON

| Group | Parameter | Values | Description |
|---|---|---|---|
| BODY | **BODY #** | 0–100 | Body resonation. Higher = more body; lower if feedback-prone. |
| BODY | **ATTACK** | 0–100 | Attack strength when plucking strongly. Higher = sharper/crisper. |
| BODY | **TONE #** | -50–+50 | Body tone. Standard 0; higher boosts high range. |

### AC — SITAR

| Group | Parameter | Values | Description |
|---|---|---|---|
| | **PU SEL #** | FRONT, R+F, REAR, PIEZO | Pickup selection. |
| | **SENS** | 0–100 | Input sensitivity. |
| BODY | **BODY** | 0–100 | Body resonation. |
| BODY | **COLOR** | 0–100 | Overall tone quality of the sitar. |
| BODY | **DECAY** | 0–100 | Time after attack for the tone to change. |
| BODY | **BUZZ** | 0–100 | Buzz-bridge character. |
| BODY | **ATTACK LEVEL** | 0–100 | Attack volume. |
| BODY | **TONE #** | -50–+50 | Body tone. Standard 0; higher boosts high range. |

### AC — BANJO

| Group | Parameter | Values | Description |
|---|---|---|---|
| BODY | **ATTACK #** | 0–100 | Attack strength when plucking strongly. Higher = sharper/crisper. |
| BODY | **RESO** | 0–100 | Body resonation. Higher = more resonation. |
| BODY | **TONE #** | -50–+50 | Body tone. Standard 0; higher boosts high range. |
| NS | *(see Shared NS)* | | |

### AC — RESO

| Group | Parameter | Values | Description |
|---|---|---|---|
| BODY | **SUSTAIN** | 0–100 | How volume responds to string dynamics; also range/time over which low-level signals are boosted. Larger = longer sustain. |
| BODY | **RESO #** | 0–100 | Body resonation. Higher = more resonation. |
| BODY | **TONE #** | -50–+50 | Body tone. Standard 0; higher boosts high range. |
| NS | *(see Shared NS)* | | |

---

### E.BASS — JB, VINT JB, T-BIRD

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **REAR VOL** | 0–100 | Rear pickup volume. |
| PU | **FRONT VOL** | 0–100 | Front pickup volume. |
| PU | **VOLUME** | 0–100 | Overall bass volume. 0 = no sound. |
| PU | **TONE #** | 0–100 | Tone. |
| NS | *(see Shared NS)* | | |

### E.BASS — PB, VINT PB

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **VOLUME** | 0–100 | Volume. 0 = no sound. |
| PU | **TONE #** | 0–100 | Tone. |
| NS | *(see Shared NS)* | | |

### E.BASS — M-MAN

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **TREBLE #** | -50–+50 | High frequency tone. |
| PU | **BASS #** | 0–100 | Low frequency tone. |
| PU | **VOLUME** | 0–100 | Volume. 0 = no sound. |
| NS | *(see Shared NS)* | | |

### E.BASS — RICK

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **REAR VOL** | 0–100 | Rear pickup volume. |
| PU | **FRONT VOL** | 0–100 | Front pickup volume. |
| PU | **REAR TONE** | 0–100 | Rear pickup tone. |
| PU | **FRONT TONE** | 0–100 | Front pickup tone. |
| PU | **VOLUME** | 0–100 | Overall bass volume. 0 = no sound. |
| PU | **PU SEL #** | REAR, R+F, FRONT | Pickup selection. |
| NS | *(see Shared NS)* | | |

### E.BASS — ACTIVE

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **REAR VOL** | 0–100 | Rear pickup volume. |
| PU | **FRONT VOL** | 0–100 | Front pickup volume. |
| PU | **TREBLE #** | 0–100 | High frequency tone. |
| PU | **BASS #** | 0–100 | Low frequency tone. |
| PU | **VOLUME** | 0–100 | Overall bass volume. 0 = no sound. |
| NS | *(see Shared NS)* | | |

### E.BASS — VIOLIN

| Group | Parameter | Values | Description |
|---|---|---|---|
| PU | **REAR VOL** | 0–100 | Rear pickup volume. |
| PU | **FRONT VOL** | 0–100 | Front pickup volume. |
| PU | **VOLUME** | 0–100 | Overall bass volume. 0 = no sound. |
| PU | **TREBLE ON #** | OFF, ON | Rear pickup on/off. |
| PU | **BASS ON #** | OFF, ON | Front pickup on/off. |
| PU | **RHYTHM/SOLO #** | RHYTHM, SOLO | RHYTHM = lower volume for backing; SOLO = higher volume for soloing. |
| NS | *(see Shared NS)* | | |

---

### SYNTH — ANALOG GR

| Group | Parameter | Values | Description / constraints |
|---|---|---|---|
| TONE | **MODE #** | VCO, V+D, DIST | VCO only / VCO+distortion / distortion only. |
| TONE | **COMP** | OFF, ON | ON extends hexa-VCO decay (and VCF decay if ENV MOD SW is ON). Does **not** extend hexa-distortion decay. |
| TONE | **FILTER CUTOFF #** | 0–100 | VCF cutoff / brightness. Higher = brighter. |
| TONE | **FILTER RESO #** | 0–100 | VCF resonance. Higher = more distinctive; near-cutoff boost. |
| ENV | **ENV MOD SW** | OFF, ON, INV | Envelope modulator for VCF. ON = wah (high→low); INV = reverse (“oo-ahh”). Extreme/high cutoff settings affect how audible this is. |
| ENV | **ENV MOD SENS** | 0–100 | Input sensitivity of envelope modulation. Higher = more range when plucking softly. |
| ENV | **ENV MOD ATTACK** | 0–100 | Attack time of envelope modulation. Higher = slower. Setting to 0 makes changes easier to hear. |
| PITCH | **PITCH SW** | OFF, A, B | Selects pitch shift for hexa-VCO only (not hexa-distortion). Use MODE VCO or V+D to hear pitch shift. |
| PITCH | **PITCH A/B** | -12–+12 | Semitone shift for A/B. -12 = one octave down; +12 = one octave up. |
| PITCH | **PITCH A/B FINE** | -50–+50 | Fine tune A/B (±½ semitone). |
| PITCH | **DUET** | OFF, ON | Adds a sawtooth at original pitch to hexa-VCO for richness. |
| PITCH | **SWEEP SW** | OFF, ON | Smoothly changes shift when PITCH SW changes. RISE/FALL ignored if OFF. |
| PITCH | **SWEEP RISE** | 0–100 | Time to move to a higher pitch. 0 = instant. |
| PITCH | **SWEEP FALL** | 0–100 | Time to move to a lower pitch. 0 = instant. |
| VIBRATO | **VIBRATO SW** | OFF, ON | Electronic vibrato on hexa-VCO only (not hexa-distortion). |
| VIBRATO | **VIBRATO RATE** | 0–100 | Vibrato rate. Higher = faster. |
| VIBRATO | **VIBRATO DEPTH** | 0–100 | Vibrato depth. 0 = none. |

### SYNTH — WAVE SYNTH

| Parameter | Values | Description |
|---|---|---|
| **TYPE #** | SAW, SQUARE | SAW = analog-synth lead; SQUARE = analog-synth backing. |
| **COLOR #** | 0–100 | Tone quality. Higher = brighter. |

### SYNTH — FILTER BASS

| Parameter | Values | Description / constraints |
|---|---|---|---|
| **FILTER CUTOFF #** | 0–100 | Cutoff / brightness (hardness). Higher = brighter/harder. |
| **FILTER RESO #** | 0–100 | Resonance / distinctiveness. |
| **FILTER DECAY** | 0–100 | Speed at which the filter stops. Lower = faster. **Effect cannot be obtained if TOUCH SENS is too low.** |
| **TOUCH SENS** | 0–100 | How playing shifts the filter. 0 = filter stays fixed. |
| **COLOR #** | 0–100 | Low-range strength. Higher = stronger lows. |

### SYNTH — CRYSTAL (CRYSTL)

| Parameter | Values | Description |
|---|---|---|
| **ATTACK LENGTH** | 0–100 | Decay time of the attack portion. Smaller = shorter attack. |
| **MOD TUNE** | 0–100 | Tuning of modulation applied to the attack. |
| **MOD DEPTH #** | 0–100 | Depth of attack modulation. Larger = deeper undulations. |
| **ATTACK LEVEL #** | 0–100 | Volume of the attack portion. |
| **BODY LEVEL #** | 0–100 | Volume of the sustained portion. |
| **SUSTAIN** | 0–100 | Range/time over which low-level signals are boosted. Larger = longer sustain. |

### SYNTH — ORGAN

| Parameter | Values | Description |
|---|---|---|
| **FEET16 #** | 0–100 | Long tone at the same pitch as the guitar. |
| **FEET8 #** | 0–100 | Long tone one octave higher than the guitar. |
| **FEET4 #** | 0–100 | Long tone two octaves higher than the guitar. |
| **SUSTAIN** | 0–100 | Range/time over which low-level signals are boosted. Larger = longer sustain. |

### SYNTH — BRASS

| Parameter | Values | Description |
|---|---|---|
| **FILTER CUTOFF #** | 0–100 | Cutoff / brightness (hardness). Higher = brighter/harder. |
| **FILTER RESO #** | 0–100 | Resonance / distinctiveness. |
| **TOUCH SENS #** | 0–100 | How playing shifts the filter. 0 = filter stays fixed. |
| **SUSTAIN** | 0–100 | Range/time over which low-level signals are boosted. Larger = longer sustain. |

---

## Quick reference — modeling tone numbers by mode

### Guitar mode numbering

| # | Category | Tone |
|---:|---|---|
| 01 | E.GTR | CLA-ST |
| 02 | E.GTR | MOD-ST |
| 03 | E.GTR | H&H-ST |
| 04 | E.GTR | TE |
| 05 | E.GTR | LP |
| 06 | E.GTR | P-90 |
| 07 | E.GTR | LIPS |
| 08 | E.GTR | RICK |
| 09 | E.GTR | 335 |
| 10 | E.GTR | L4 |
| 11 | AC | STEEL |
| 12 | AC | NYLON |
| 13 | AC | SITAR |
| 14 | AC | BANJO |
| 15 | AC | RESO |
| 16 | E.BASS | JB |
| 17 | E.BASS | PB |
| 18 | SYNTH | ANALOG GR |
| 19 | SYNTH | WAVE SYNTH |
| 20 | SYNTH | FILTER BASS |
| 21 | SYNTH | CRYSTAL |
| 22 | SYNTH | ORGAN |
| 23 | SYNTH | BRASS |

### Bass mode numbering

| # | Category | Tone |
|---:|---|---|
| 01 | E.BASS | VINT JB |
| 02 | E.BASS | JB |
| 03 | E.BASS | VINT PB |
| 04 | E.BASS | PB |
| 05 | E.BASS | M-MAN |
| 06 | E.BASS | RICK |
| 07 | E.BASS | T-BIRD |
| 08 | E.BASS | ACTIVE |
| 09 | E.BASS | VIOLIN |
| 10 | SYNTH | ANALOG GR |
| 11 | SYNTH | WAVE SYNTH |
| 12 | SYNTH | FILTER BASS |
| 13 | SYNTH | CRYSTAL |
| 14 | SYNTH | ORGAN |
| 15 | SYNTH | BRASS |
| 16 | E.GTR | ST (CLA-ST) |
| 17 | E.GTR | LP |
