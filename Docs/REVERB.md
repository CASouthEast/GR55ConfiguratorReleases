# REVERB

Dedicated reverb processor. Extracted from `GR-55_OM.pdf` (pp. 52–53) via macOS PDFKit.

See [Effects_Index.md](./Effects_Index.md).

---

## Comparison with MOD / MFX

| Processor | Reverb algorithm? |
|---|---|
| **Dedicated REVERB** | Yes — AMBIENCE, ROOM, HALL 1, HALL 2, PLATE |
| **MOD** | No reverb type |
| **MFX** | No reverb type |

There is no MOD or MFX reverb algorithm. Spatial ambience on those paths is limited to chorus/delay-style effects (see [CHORUS.md](./CHORUS.md), [DELAY.md](./DELAY.md)). Reverb is applied as a shared bus via send levels from MFX, AMP–MOD, and bypass.

---

## Dedicated REVERB (EFFECT → REVERB)

| Parameter | Values | Description |
|---|---|---|
| **SWITCH** | OFF, ON | Turns REVERB OFF/ON. |
| **TYPE** | See types below | Chooses the reverb type. |
| **REVERB TIME #** | 0.1 s–10.0 s | Length (time) of reverberation. |
| **HIGH CUT #** | 700 Hz–11.0 kHz, FLAT | High-cut on the reverb sound. FLAT = no high cut. |
| **EFFECT LEVEL** | 0–100 | Volume level of the effect. |
| **MFX SEND** | 0–100 | Level from MFX output into REVERB. |
| **MOD SEND** | 0–100 | Level from AMP–MOD output into REVERB. |
| **BYPASS SEND** | 0–100 | Level from bypass channel (unprocessed by the effect) into REVERB. |

### Reverb types

| Type | Description |
|---|---|
| **01: AMBIENCE** | Simulates an ambience microphone (off-mic, placed at a distance) used in recording. Emphasizes openness and depth rather than long reverberation. |
| **02: ROOM** | Small-room reverberation; warm. |
| **03: HALL 1** | Concert-hall reverberation; clear and spacious. |
| **04: HALL 2** | Concert-hall reverberation; mild. |
| **05: PLATE** | Plate reverberation (large metal plate). Metallic sound with a distinct upper range. |
