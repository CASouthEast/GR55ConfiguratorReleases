# GR-55 Effects Documentation Index

Extracted from the Roland GR-55 Owner’s Manual (`GR-55_OM.pdf`, Effect Settings pp. 38–53) via **macOS PDFKit**.

Patch / system settings (MASTER, SYSTEM, ASSIGNS): [Settings_Index.md](./Settings_Index.md)  
SysEx / Data JSON guide: [SysExParsing.md](./SysExParsing.md)

## Processors

| Processor | Doc | Notes |
|---|---|---|
| **AMP** | [AMP_and_MFX_GuitarAmpSim.md](./AMP_and_MFX_GuitarAmpSim.md) | Dedicated AMP + MFX Guitar Amp Simulator comparison |
| **MOD** | [MOD.md](./MOD.md) | MOD types that are not covered in the comparison docs below |
| **MFX** | [MFX.md](./MFX.md) | MFX types that are not covered in AMP / EQ / DELAY / CHORUS docs |
| **DELAY** | [DELAY.md](./DELAY.md) | Dedicated DELAY + MOD Delay + MFX delay types (comparison) |
| **REVERB** | [REVERB.md](./REVERB.md) | Dedicated REVERB only (no MOD/MFX reverb algorithm) |
| **CHORUS** | [CHORUS.md](./CHORUS.md) | Dedicated CHORUS + MOD Chorus + MFX Hexa-Chorus / Space-D (comparison) |
| **EQ** | [EQ.md](./EQ.md) | Dedicated EQ + MOD EQ + MFX EQ (comparison) |

## Where overlapping types live

| Effect family | Locations in the GR-55 | Documented in |
|---|---|---|
| Amp simulation | Dedicated AMP; MFX type 14 | AMP_and_MFX_GuitarAmpSim.md |
| EQ | Dedicated EQ; MOD type 14; MFX type 01 | EQ.md |
| Delay | Dedicated DELAY; MOD type 12; MFX types 17–18 | DELAY.md |
| Chorus | Dedicated CHORUS; MOD type 13; MFX types 10–11 | CHORUS.md |
| Reverb | Dedicated REVERB only | REVERB.md |

Other similar-sounding algorithms (e.g. MOD Phaser vs MFX Phaser, MOD Comp vs MFX Compressor) stay in **MOD.md** / **MFX.md** respectively — they are separate processors, not merged comparison docs.

## Shared manual notes

- Parameters marked `#` appear on the effect quick-edit screen.
- For **RATE** and **DELAY TIME**: choosing **BPM** ties the value to patch tempo (PATCH TEMPO).
- Company/product names in type descriptions are trademarks; used to describe COSM-simulated sounds.
